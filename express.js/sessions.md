# Express.js — Sessions (`express-session`)

Sessions solve a problem plain cookies don't handle well: keeping meaningful data tied to a visitor **without** putting that data itself inside the cookie. This note explains how `express-session` works, walks through your setup line by line, flags two real configuration bugs, and adds extra examples (logging out, session stores, regeneration) your code doesn't show yet.

---

## 1. What a Session Actually Is

In the cookies note, you saw that a cookie can hold data directly — the browser stores it and sends it back on every request. A **session** flips that around:

- The **real data** (`{ visited: true, userId: 4, ... }`) lives **on the server**, in memory or a database.
- The cookie the browser holds only contains a **random ID** pointing at that server-side record — nothing meaningful, nothing an attacker could tamper with usefully.

```
Browser cookie:  connect.sid = s%3AeH8f2k9...   ← just an ID, signed
                                    │
                                    ▼
Server-side store:  { "eH8f2k9...": { visited: true, cookie: {...} } }
                     ← the actual session DATA lives here, not in the cookie
```

This is why sessions are the standard choice for things like "is this user logged in" — the sensitive decision-making data never leaves the server.

---

## 2. Setting Up `express-session`

```javascript
app.use(
  session({
    secret: "secret wow",
    saveUninitialized: "false",
    resave: "false",
    Cookie: {
      maxAge: 60000 * 60,
    },
  }),
);
```

Going through each option:

- **`secret`** — a string used to sign the session-ID cookie, so the server can detect if a client tampered with it. Should be long and random, ideally pulled from an environment variable rather than hardcoded (`"secret wow"` is fine for practicing, not for anything real).
- **`resave`** — whether to save the session back to the store on every request, even if nothing changed. Usually set to `false` to avoid unnecessary writes.
- **`saveUninitialized`** — whether to create and store a session for visitors who never actually put anything in `req.session` (e.g. a bot just hitting your homepage). Usually set to `false` to avoid storing empty, useless sessions.
- **`cookie`** — options for the cookie that carries the session ID (same shape as `res.cookie()`'s third argument from the cookies note — `maxAge`, `httpOnly`, `secure`, etc.).

> ⚠️ **Bug 1 — `"false"` is a string, and strings are truthy.**
> 
> ```javascript
> saveUninitialized: "false",
> resave: "false",
> ```
> 
> In JavaScript, **every non-empty string is truthy**, including the string `"false"`. So both of these options are currently being evaluated as `true` — the exact opposite of what the code is trying to say. This silently defeats the whole point of setting them.
> 
> **Fix — use the actual boolean, no quotes:**
> 
> ```javascript
> saveUninitialized: false,
> resave: false,
> ```
> 
> This is a very common JS trap, not specific to sessions — always double check that `true`/`false` in config objects aren't accidentally strings.

> ⚠️ **Bug 2 — `Cookie` (capital C) is not a recognized option; it must be `cookie`.**
> 
> `express-session` looks specifically for a lowercase `cookie` key. Because this code capitalizes it, the entire block:
> 
> ```javascript
> Cookie: {
>   maxAge: 60000 * 60,
> },
> ```
> 
> is silently ignored — JavaScript object keys are case-sensitive, so `Cookie` and `cookie` are two completely different keys as far as the library is concerned. The session falls back to its **default** cookie settings (which just makes it a session cookie — gone when the browser closes) instead of the intended 1-hour (`60000 * 60` ms) lifetime.

**Corrected setup:**

```javascript
app.use(
  session({
    secret: "secret wow",
    saveUninitialized: false,
    resave: false,
    cookie: {
      maxAge: 60000 * 60, // 1 hour, in milliseconds
    },
  }),
);
```

---

## 3. Reading and Writing Session Data — `req.session`

```javascript
app.get("/", usingmiddleware, (req, res) => {
  res.cookie("password", "123456", { maxAge: 6000 * 60 });
  console.log(req.session);
  console.log(req.session.id);
  req.session.visited = true;
  res.send("wow hi !");
});
```

- `req.session` — an object the middleware attaches to every request. You can read from it and write to it like a plain object.
- `req.session.visited = true;` — this is all it takes to **store** data in the session. Behind the scenes, `express-session` takes care of saving this back into the store (memory, by default) and keeping the browser's cookie in sync.
- `req.session.id` — the current session's unique identifier. This works because `express-session` also mirrors it as a property directly on the session object. You'll also commonly see it accessed as **`req.sessionID`** (no dot, capital ID) — both refer to the exact same value; use whichever reads more clearly to you.

**What your code doesn't show: actually reading it back.** Right now, `visited` gets set on every visit to `/`, but nothing ever checks it. Here's the missing half of the picture:

```javascript
app.get("/", (req, res) => {
  if (req.session.visited) {
    res.send("Welcome back!");
  } else {
    req.session.visited = true;
    res.send("Hi, first time seeing you!");
  }
});
```

Because the session persists across requests (as long as the browser sends the same session cookie back), a second visit from the same browser will hit the `if` branch instead of the `else`.

---

## 4. Extra: Session Stores (the Part Your Code Is Missing Entirely)

Your `session(...)` config never sets a `store` option. That means `express-session` quietly falls back to its built-in **`MemoryStore`** — sessions are kept in a plain JavaScript object in your server's RAM.

This is fine for learning, but has real limitations worth knowing:

- **All sessions vanish** the moment the server restarts.
- **It leaks memory over time** — the library's own documentation states it explicitly, and it will print a warning to your console about not being fit for production.
- **It doesn't work if you ever run more than one server instance** (e.g. behind a load balancer), since each instance would have its own separate memory and no shared view of sessions.

For anything beyond learning/demos, you'd swap in a real store, e.g. Redis:

```javascript
import RedisStore from "connect-redis";
import { createClient } from "redis";

const redisClient = createClient();
await redisClient.connect();

app.use(
  session({
    store: new RedisStore({ client: redisClient }),
    secret: "secret wow",
    resave: false,
    saveUninitialized: false,
    cookie: { maxAge: 60000 * 60 },
  }),
);
```

Same `session()` API, same `req.session` usage in your routes — only the storage location changes.

---

## 5. Extra: Logging Out — `req.session.destroy()`

Nothing in your code removes a session. The standard "log out" pattern:

```javascript
app.post("/logout", (req, res) => {
  req.session.destroy((err) => {
    if (err) return res.status(500).send("Could not log out");
    res.clearCookie("connect.sid"); // remove the now-invalid cookie too
    res.send("Logged out");
  });
});
```

`destroy()` removes the session's data from the store entirely (not just the cookie), and takes a callback since it involves I/O (talking to the store).

---

## 6. Extra: Regenerating a Session (Security Best Practice After Login)

A subtle but important security pattern: when a user successfully logs in, you should generate a **brand-new** session ID rather than reusing whatever anonymous session ID they already had (this prevents a class of attack called _session fixation_).

```javascript
app.post("/login", (req, res) => {
  // ...verify credentials first...
  req.session.regenerate((err) => {
    if (err) return res.status(500).send("Login failed");
    req.session.userId = 4; // now attach the real user's data to the fresh session
    res.send("Logged in");
  });
});
```

---

## 7. Extra: `cookie-parser` vs. `express-session` — You Don't Need Both for the Same Job

Your code uses both packages:

```javascript
app.use(session({ ... }));
app.use(cookieparse());
```

It's worth being clear that these serve **different purposes** here, and one isn't required for the other to function:

- `express-session` manages **its own** cookie (default name `connect.sid`) internally — parsing it, verifying its signature, and syncing it — all on its own. It does **not** rely on `cookie-parser` to do this.
- `cookie-parser` in your code is there for the **separate**, manually-set `"password"` cookie from the earlier cookies lesson, so `req.cookies` works for that unrelated cookie.

They can coexist fine, as your code does, but if you ever see a tutorial claim you _must_ install `cookie-parser` for sessions to work — that hasn't been true in modern `express-session` versions.

---

## 8. Summary: `session()` Options

|Option|Type|Purpose|
|---|---|---|
|`secret`|string|Signs the session-ID cookie so tampering can be detected.|
|`resave`|boolean|Save the session back to the store on every request, even if unchanged. Usually `false`.|
|`saveUninitialized`|boolean|Store a session even if nothing was ever written to it. Usually `false`.|
|`cookie`|object|Options for the session-ID cookie itself (`maxAge`, `httpOnly`, `secure`, `sameSite`, ...). Lowercase — this is easy to typo.|
|`store`|Store instance|Where session data actually lives. Defaults to in-memory (not production-safe) — swap in Redis/Mongo/etc. for real apps.|

---

## 9. Bugs Found (Recap)

1. **`resave: "false"` and `saveUninitialized: "false"` are strings, and any non-empty string is truthy in JS.** Both settings are currently behaving as `true`, the opposite of the evident intent. Use the bare booleans `false` instead.
2. **`Cookie: { maxAge: ... }` uses the wrong casing.** `express-session` only recognizes lowercase `cookie`; as written, the custom 1-hour `maxAge` is silently ignored and the session falls back to default cookie behavior.
3. **No `store` is configured**, so sessions rely on the default `MemoryStore` — fine for learning, but explicitly not meant for production (data loss on restart, memory growth, doesn't scale across multiple server instances).
4. **Nothing ever reads `req.session.visited` back**, so there's currently no observable difference in behavior between a first-time and returning visitor, even though the data is being stored correctly.