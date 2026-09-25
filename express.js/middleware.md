# Express.js — Middleware

Middleware is arguably the single most important concept in Express. Almost everything you've already seen — `json()`, route handlers, even the multi-argument routes — _is_ middleware under the hood. This note explains what it is, walks through every middleware pattern in your code, points out one important bug, and adds a few extra patterns your code doesn't demonstrate yet (error handling, routers, third-party middleware).

---

## 1. What Is Middleware?

A middleware is just a function with this exact shape:

```javascript
(req, res, next) => {
  // do something with req/res
  next(); // pass control to the next function in line
}
```

Every route handler you've written so far (`(req, res) => {...}`) is technically middleware too — it's just usually the _last_ one in the chain, so it doesn't need to call `next()`.

A middleware function can do one of three things:

1. **Run some code**, then call `next()` to hand off control to whatever comes next.
2. **End the response** itself (`res.send()`, `res.sendStatus()`, etc.) and _not_ call `next()` — stopping the chain right there.
3. **Pass an error** to `next(err)`, which skips straight to error-handling middleware (see §6).

If a middleware does neither #2 nor #3, and forgets to call `next()`, **the request hangs forever** — the client just waits with no response, no error, nothing. This is one of the most common beginner bugs.

---

## 2. Global Middleware — `app.use()`

```javascript
app.use(json());
```

`app.use(fn)` registers `fn` to run on **every incoming request**, regardless of path or HTTP method (as long as it's declared _before_ the routes you want it to affect — order matters, see §7).

Your code also defines this one but leaves it commented out:

```javascript
const usingmiddleware = (req, res, next) => {
  console.log(`${req.method} - ${req.url}`);
  next();
};
// using middleware globaly
//app.use(usingmiddleware);
```

If you uncommented `app.use(usingmiddleware)`, **every single request** to any route would first print something like:

```
GET - /api/users
POST - /api/users
```

This is a classic **logging middleware** pattern — very common in real apps (it's essentially a tiny version of the popular `morgan` package, see §8).

Right now, since that line is commented out, `usingmiddleware` only runs where it's explicitly attached — which is the next pattern.

---

## 3. Route-Specific Middleware

```javascript
app.get("/", usingmiddleware, (req, res) => {
  res.send("wow hi !");
});
```

Instead of applying `usingmiddleware` globally, you can pass it as an extra argument to a _single_ route. Express runs arguments **left to right**:

```
request → usingmiddleware → route handler → response
```

So only `GET /` gets logged this way — no other route does, since `usingmiddleware` isn't attached anywhere else.

---

## 4. Chaining Multiple Middleware in One Route

```javascript
app.get(
  "/api/users",
  (req, res, next) => {
    console.log("BASE URL");
    next();
  },
  (req, res) => {
    res.send(mockuser);
  },
);
```

You can pass as many middleware functions as you want to a single route — Express just runs them in the order given, as long as each one calls `next()`. Here:

1. First function logs `"BASE URL"` and calls `next()`.
2. Second function (the "real" handler) sends the response.

This is functionally identical to using `usingmiddleware` for this route too — it's just written inline instead of as a named, reusable function.

---

## 5. Custom Validation Middleware — `finduserbyid`

```javascript
const finduserbyid = (req, res, next) => {
  const {
    params: { id },
  } = req;
  const typedid = parseInt(id);
  if (isNaN(typedid)) return res.status(400).send("not a valid id");
  const finduserid = mockuser.findIndex((user) => {
    return user.id === typedid;
  });
  if (finduserid === -1) return res.sendStatus(404);
  request.finduserid = finduserid;
  next();
};
```

This is the most useful middleware pattern here: **pulling repeated validation logic out of your route handlers.** Notice that your `PUT`, `PATCH`, and `DELETE` routes all previously repeated the exact same four lines (parse id → check `isNaN` → `findIndex` → check `-1`). `finduserbyid` exists to do that _once_, so routes can reuse it instead of duplicating the logic.

How it's meant to work:

1. Parse and validate `id` from the URL, same as before.
2. If invalid → respond `400` and stop (no `next()` call, so the chain ends here).
3. If not found → respond `404` and stop.
4. If found → **attach the result onto the request** so the next function in the chain can use it, then call `next()` to continue.

It's currently wired up on `PUT`:

```javascript
app.put("/api/users/:id", finduserbyid, (req, res) => {
  const { body, finduserid } = req;
  mockuser[finduserid] = { id: mockuser[finduserid].id, ...body };
  return res.sendStatus(200);
});
```

`finduserbyid` runs first, does the lookup, and the actual `PUT` handler just trusts that `req.finduserid` is already a valid index — no need to re-validate.

> ⚠️ **Bug: `request.finduserid` should be `req.finduserid`.**
> 
> Look at the import line:
> 
> ```javascript
> import express, { json, request } from "express";
> ```
> 
> `request` here is **not** "the current request" — it's a low-level object Express itself uses internally as the _prototype_ that every real `req` object is built from. It is shared across your entire application, not per-request.
> 
> Writing `request.finduserid = finduserid` mutates that shared prototype object instead of the request that's currently being handled. In a simple, single-request-at-a-time demo this can _appear_ to work by coincidence (because `req` inherits from `request` on the prototype chain), but it's a serious anti-pattern:
> 
> - If two requests come in close together (very common — Node handles requests concurrently), one request's `finduserid` can leak into another request that's still being processed, corrupting the wrong user's data.
> - It never gets cleaned up between requests, since it's sitting on a shared object, not a fresh one created per request.
> 
> **Fix — always attach custom data to the local `req` parameter, never to the imported `request`:**
> 
> ```javascript
> const finduserbyid = (req, res, next) => {
>   const typedid = parseInt(req.params.id);
>   if (isNaN(typedid)) return res.status(400).send("not a valid id");
>   const finduserid = mockuser.findIndex((user) => user.id === typedid);
>   if (finduserid === -1) return res.sendStatus(404);
>   req.finduserid = finduserid; // ✅ per-request, not shared
>   next();
> };
> ```
> 
> As a rule of thumb: **never import `request` or `response` from `"express"` and use them as if they were `req`/`res`.** They exist for advanced prototype-extension use cases, not everyday route logic.

### Reusing it properly

Once fixed, the same middleware could clean up `PATCH` and `DELETE` too, which currently duplicate the lookup logic manually:

```javascript
app.patch("/api/users/:id", finduserbyid, (req, res) => {
  mockuser[req.finduserid] = { ...mockuser[req.finduserid], ...req.body };
  return res.sendStatus(200);
});

app.delete("/api/users/:id", finduserbyid, (req, res) => {
  mockuser.splice(req.finduserid, 1);
  return res.sendStatus(200);
});
```

This is the real payoff of middleware: write the validation once, reuse it on every route that needs it.

---

## 6. `next()` in Detail

- **`next()`** with no arguments → moves on to the next middleware/handler in the chain.
- **`next(err)`** with an argument → Express treats it as an error and **skips every remaining normal middleware**, jumping straight to the nearest **error-handling middleware** (see below). Your code doesn't do this anywhere, but it's essential to know.
- **Not calling `next()` at all** (and not sending a response) → the request hangs until the client times out. This is a very common bug when someone adds an `if` check but forgets an `else` branch:

```javascript
// ❌ Bug: if the id is valid, neither res.send() nor next() ever runs
const badMiddleware = (req, res, next) => {
  if (isNaN(parseInt(req.params.id))) {
    res.status(400).send("bad id");
  }
  // missing: next() here for the valid case!
};
```

---

## 7. Extra Example: Error-Handling Middleware

Your code has no error-handling middleware, but it's a core part of Express you'll want to know. It looks like a normal middleware **except it takes four arguments instead of three** — Express specifically recognizes the four-argument signature as an error handler:

```javascript
// A normal route that might throw
app.get("/api/risky/:id", (req, res, next) => {
  try {
    const typedid = parseInt(req.params.id);
    if (isNaN(typedid)) throw new Error("Invalid ID format");
    res.send({ id: typedid });
  } catch (err) {
    next(err); // hand the error off instead of crashing the server
  }
});

// Error-handling middleware — must be defined LAST, after all routes
app.use((err, req, res, next) => {
  console.error(err.message);
  res.status(500).send({ error: err.message });
});
```

Key rules:

- Error handlers must be registered **after** every other `app.use()`/route in the file.
- Express identifies it purely by **argument count** (`err, req, res, next` — exactly 4), not by name.
- You can have multiple error handlers for different error types, same as normal middleware chains.

---

## 8. Extra Example: Router-Level Middleware

For bigger apps, you don't attach everything to `app` directly — you split routes into an `express.Router()` and scope middleware to just that router:

```javascript
import { Router } from "express";
const userRouter = Router();

// Runs only for routes defined on userRouter, not the whole app
userRouter.use((req, res, next) => {
  console.log("User router hit:", req.method, req.url);
  next();
});

userRouter.get("/:id", finduserbyid, (req, res) => {
  res.send(mockuser[req.finduserid]);
});

app.use("/api/users", userRouter); // mount it under a base path
```

This keeps middleware scoped to a feature area instead of running for the entire app.

---

## 9. Extra Example: Third-Party Middleware

Most Express "plugins" you'll install from npm are just middleware functions someone else wrote. A few extremely common ones:

|Package|What it does|
|---|---|
|`express.json()`|Built-in — parses JSON request bodies (you're already using this)|
|`cors`|Adds the headers needed to allow requests from other origins/domains|
|`morgan`|Automatic, more detailed request logging (like your `usingmiddleware`, but configurable)|
|`helmet`|Sets various security-related HTTP headers|

They're all used the exact same way:

```javascript
import cors from "cors";
app.use(cors());
```

---

## 10. Middleware Execution Order

Order matters — Express runs everything **top to bottom, in the order it was registered**, and only for routes/middleware declared _after_ a matching `app.use()`.

```
Incoming Request
      │
      ▼
app.use(json())              ← runs for every route below this line
      │
      ▼
app.use(usingmiddleware)     ← (if uncommented) also runs for every route below
      │
      ▼
app.get("/api/users/:id", finduserbyid, handler)
      │                         │
      │                         ▼
      │                    finduserbyid runs first
      │                         │
      │                    calls next() if valid
      │                         ▼
      └───────────────►   route handler runs, sends response
```

If a middleware sends a response and does **not** call `next()`, everything below it in that chain simply never runs.

---

## 11. Types of Middleware — Summary

|Type|Example from your code / added examples|Scope|
|---|---|---|
|**Application-level**|`app.use(json())`, `app.use(usingmiddleware)`|Every route (declared after it)|
|**Route-level**|`usingmiddleware` passed into `app.get("/", usingmiddleware, ...)`|Just that one route|
|**Chained (multiple per route)**|The two functions in `app.get("/api/users", fn1, fn2)`|Just that one route, run in order|
|**Built-in**|`express.json()`|Wherever you `app.use()` it|
|**Router-level**|`userRouter.use(...)` (§8)|Only routes on that router|
|**Third-party**|`cors()`, `morgan()` (§9)|Wherever you `app.use()` it|
|**Error-handling**|`(err, req, res, next) => {...}` (§7)|Triggered by `next(err)`|

---

## 12. Bugs Found (Recap)

1. **`request.finduserid = finduserid;` mutates the wrong object.** `request` is the imported Express prototype object, not the current request — it should be `req.finduserid = finduserid;`. As written, this risks leaking state between concurrent requests.
2. **`finduserbyid` is only used on `PUT`.** `PATCH` and `DELETE` still duplicate the same id-lookup logic manually, even though the whole point of writing `finduserbyid` was to avoid that repetition.
3. **`usingmiddleware`'s global registration is commented out**, so right now it only logs for the `/` route — worth being intentional about whether you want it global or route-specific.