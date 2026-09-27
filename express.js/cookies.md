# Express.js — Cookies

Cookies are small pieces of data the server asks the browser to store and send back on every future request. This note walks through the two cookie operations your code performs, flags one important security issue, and adds extra patterns your code doesn't demonstrate yet (deleting cookies, signed cookies, and the full option list).

---

## 1. Setup: `cookie-parser`

```javascript
import cookieparse from "cookie-parser";
app.use(cookieparse());
```

Express does **not** parse cookies on its own. The raw `Cookie` header on an incoming request looks like a single string:

```
Cookie: password=123456; theme=dark
```

`cookie-parser` is middleware that reads that header and turns it into a friendly object for you, attached as `req.cookies`. Without this middleware, `req.cookies` would simply be `undefined` everywhere in your app.

> 📝 Small style note: `cookieparse` is just the name you chose for the import — nothing wrong with it, but you'll see it conventionally named `cookieParser` in tutorials/docs. Doesn't affect behavior at all.

---

## 2. Reading Cookies — `req.cookies`

```javascript
app.get(
  "/api/users",
  (req, res, next) => {
    console.log("BASE URL");
    next();
  },
  (req, res) => {
    console.log(req.cookies);
    res.send(mockuser);
  },
);
```

Once `cookie-parser` has run (globally, via `app.use`), every route handler downstream can read `req.cookies` as a plain object:

```javascript
console.log(req.cookies);
// { password: '123456' }   ← whatever the client's browser sent back
```

If the client's browser previously received the cookie set in the `/` route (see below), then any later request from that same browser to _any_ route on this server automatically includes it — that's the whole point of cookies. You don't have to do anything special to "get" it into the request; the browser attaches it for you.

---

## 3. Setting Cookies — `res.cookie()`

```javascript
app.get("/", usingmiddleware, (req, res) => {
  res.cookie("password", "123456", { maxAge: 6000 * 60 });
  res.send("wow hi !");
});
```

`res.cookie(name, value, options)` tells the browser, via a `Set-Cookie` response header, to store a cookie. Breaking this call down:

- `"password"` — the cookie's name.
- `"123456"` — the cookie's value.
- `{ maxAge: 6000 * 60 }` — an options object. `maxAge` is in **milliseconds**, so `6000 * 60 = 360000 ms = 360 seconds = 6 minutes`. After that time passes, the browser automatically discards the cookie.

> ⚠️ **Important ordering rule:** any `res.cookie()` call must happen **before** `res.send()`/`res.json()`/etc. Cookies are sent as HTTP headers, and once a response starts being sent, headers can no longer be added. Your code does this correctly (`res.cookie()` then `res.send()`), but it's a common bug to watch for.

---

## 4. ⚠️ Security Issue: Never Store Real Secrets in a Plain Cookie

```javascript
res.cookie("password", "123456", { maxAge: 6000 * 60 });
```

This line has several serious problems if it were pointing at a real password rather than demo data:

1. **No `httpOnly` flag.** By default, cookies are readable by client-side JavaScript via `document.cookie`. That means any XSS vulnerability anywhere on your site (a malicious script injected into the page) can simply read this cookie and steal it. Setting `httpOnly: true` prevents JavaScript from reading it — only the browser and server can access it.
2. **No `secure` flag.** Without `secure: true`, the cookie is sent even over plain, unencrypted HTTP, meaning anyone snooping on the network (e.g. public Wi-Fi) can read it in transit.
3. **Not signed.** There's no way to detect if this cookie was tampered with on the client side before being sent back (see §6).
4. **Storing a password at all.** Cookies (and localStorage) should never hold raw passwords — not even a demo one. The standard pattern is: verify the password once at login, then issue a random **session ID** or **token** (like a JWT), and store _that_ in the cookie instead. The actual password is never touched again after the login request.

**A safer version of the same idea:**

```javascript
res.cookie("sessionId", generateRandomSessionId(), {
  maxAge: 6000 * 60,
  httpOnly: true,   // JS on the page can't read it
  secure: true,     // only sent over HTTPS
  sameSite: "strict", // see §5 — helps prevent CSRF
});
```

---

## 5. Extra: The Full Set of Cookie Options

Your code only used `maxAge`. Here's what else `res.cookie()` accepts:

|Option|What it does|
|---|---|
|`maxAge`|Lifetime in **milliseconds**, counted from now.|
|`expires`|An exact `Date` object for when the cookie should die, instead of a relative duration. (If you set both, `maxAge` wins.)|
|`httpOnly`|If `true`, blocks client-side JavaScript from reading the cookie — critical for anything sensitive.|
|`secure`|If `true`, the cookie is only sent over HTTPS connections.|
|`sameSite`|`"strict"`, `"lax"`, or `"none"` — controls whether the cookie is sent on cross-site requests, which helps prevent CSRF attacks.|
|`signed`|If `true`, the cookie is cryptographically signed (see §6) so tampering can be detected.|
|`path`|Restricts which URL paths the cookie is sent to (defaults to the whole site, `"/"`).|
|`domain`|Restricts which domain/subdomains receive the cookie.|

Example using several at once:

```javascript
res.cookie("theme", "dark", {
  maxAge: 1000 * 60 * 60 * 24 * 7, // 7 days
  httpOnly: false,                  // fine here — not sensitive data
  sameSite: "lax",
});
```

**`maxAge` vs `expires`** — same goal, different style:

```javascript
res.cookie("a", "1", { maxAge: 60000 });                       // expires in 1 minute from now
res.cookie("b", "2", { expires: new Date(Date.now() + 60000) }); // same result, explicit Date
```

If you set neither, the cookie becomes a **session cookie** — it disappears when the browser itself is closed, not after a fixed time.

---

## 6. Extra: Signed Cookies

Your code never uses signing, but it's the right tool given you're storing something meant to be sensitive. A signed cookie has a cryptographic signature attached, so if a user edits the cookie's value in their browser (trivial to do with dev tools), the server can detect it and reject the tampered value.

```javascript
// 1. Give cookie-parser a secret when you set it up
app.use(cookieparse("some-long-random-secret-string"));

// 2. Mark the cookie as signed when you set it
app.get("/", (req, res) => {
  res.cookie("sessionId", "abc123", { signed: true, httpOnly: true });
  res.send("logged in");
});

// 3. Read it from req.signedCookies, not req.cookies
app.get("/api/users", (req, res) => {
  console.log(req.signedCookies.sessionId); // "abc123" if untampered
  // if someone tampered with it client-side, this becomes `false` instead
});
```

Unsigned cookies you set (like `password` in your code) still show up in `req.cookies`, never in `req.signedCookies` — the two are kept separate.

---

## 7. Extra: Deleting a Cookie

Your code never removes the cookie it sets (e.g. on logout). The correct way is `res.clearCookie()`, not trying to set `maxAge: 0` manually:

```javascript
app.get("/logout", (req, res) => {
  res.clearCookie("password");
  res.send("logged out");
});
```

> 📝 One gotcha: if you set a cookie with specific `path`/`domain` options, you must pass those **same** options to `clearCookie()`, or the browser won't consider it the same cookie and won't remove it.

---

## 8. Extra: Cookies vs. Sessions (Just Enough to Know the Difference)

Your code stores actual data (`"123456"`) directly inside the cookie. There's a very common alternative pattern worth knowing about, usually via the `express-session` package:

- **Cookie-only approach (what your code does):** all the data lives in the cookie itself, sent back and forth on every request.
- **Session approach:** the server stores the real data (like "this session belongs to user #4") in its own memory/database, and the cookie only holds a random, meaningless session ID pointing to that server-side record. This keeps sensitive data off the client entirely.

Not something your current code needs to change to, just useful context for why you'll see `req.session` in other tutorials instead of raw cookies.

---

## 9. Bugs Found (Recap)

1. **Storing `"password"` as a literal cookie value with no protection at all** — no `httpOnly`, no `secure`, no `signed`. If this were real user data instead of a demo, it would be trivially readable by any injected script and by anyone intercepting unencrypted traffic.
2. **No way to remove the cookie later** — there's no `/logout`-style route using `res.clearCookie()`, so once set, the cookie just sits there until its 6-minute `maxAge` runs out.