# Express.js — POST, PUT, PATCH, DELETE

This continues from the previous notes and focuses only on the **mutation routes** (the ones that create, replace, update, or remove data) plus the middleware needed to make them work.

---

## 1. Body-Parsing Middleware

```javascript
import express, { json } from "express";
const app = express();
app.use(json());
```

- `{ json }` is a **named import** — Express exposes `express.json()` as a built-in middleware, and this destructures it directly from the package.
- `app.use(json())` — registers that middleware **globally**, meaning it runs on _every_ incoming request before your route handlers.
- What it does: reads the raw request body, and if the `Content-Type` header is `application/json`, parses it into a JS object and attaches it to `req.body`.

> ⚠️ **Why this matters:** without `app.use(json())`, `req.body` would be `undefined` in every route below, since Express does **not** parse bodies by default.

---

## 2. POST — Create

```javascript
app.post("/api/users", (req, res) => {
  res.send(req.body);
});
```

- `app.post(path, handler)` — registers a route that only responds to `POST` requests on that path.
- `req.body` — the parsed JSON payload the client sent (thanks to the `json()` middleware above). For example, sending:
    
    ```json
    { "name": "sara", "age": "25" }
    ```
    
    makes that object available as `req.body`.
- This handler just **echoes the body back** — it doesn't actually push anything into `mockuser`.

> ⚠️ **Incomplete, not just a demo shortcut:** a real "create" endpoint should generate a new `id`, push the new object into `mockuser`, and return the created resource (usually with status `201 Created`). As written, it looks like it creates a user but doesn't persist anything:
> 
> ```javascript
> app.post("/api/users", (req, res) => {
>   const newUser = { id: mockuser.length + 1, ...req.body };
>   mockuser.push(newUser);
>   return res.status(201).send(newUser);
> });
> ```

---

## 3. PUT — Full Replace

```javascript
app.put("/api/users/:id", (req, res) => {
  const {
    body,
    params: { id },
  } = req;
  const typedid = parseInt(id);
  if (isNaN(typedid)) return res.status(400).send("not a valid id");
  const finduserid = mockuser.findIndex((user) => {
    return user.id === typedid;
  });
  if (finduserid === -1) return res.sendStatus(404);
  mockuser[finduserid] = { id: typedid, ...body };
  return res.sendStatus(200);
});
```

Step by step:

1. Destructure `body` (parsed JSON payload) and `id` (route param) straight out of `req`.
2. Convert `id` to a number the same way as before, and reject non-numeric ids with `400`.
3. `findIndex(...)` — like `find()`, but returns the **index** of the match instead of the object itself (`-1` if nothing matches). You need the index here because you're about to overwrite that array slot.
4. If no match, `404`.
5. `mockuser[finduserid] = { id: typedid, ...body }` — **replaces the entire object** at that index with a brand-new one built from the URL's `id` plus whatever fields were in the request body.

This is the defining trait of **PUT**: it's meant to replace the _whole_ resource. If the client only sends `{ name: "new name" }`, the resulting user loses its `age` field entirely, because nothing is merged with the old data — it's fully overwritten.

> ⚠️ **Bug — id can be overridden by the body:** `{ id: typedid, ...body }` spreads `body` _after_ `id`, so if the client's JSON payload happens to include its own `id` field (e.g. `{ "id": 999, "name": "x" }`), the spread overwrites `typedid` and the user's ID silently changes. Fix by spreading first, then forcing the id last:
> 
> ```javascript
> mockuser[finduserid] = { ...body, id: typedid };
> ```

---

## 4. PATCH — Partial Update

```javascript
app.patch("/api/users/:id", (req, res) => {
  const {
    body,
    params: { id },
  } = req;
  const typedid = parseInt(id);
  if (isNaN(typedid)) return res.status(400).send("not a valid id");
  const finduserid = mockuser.findIndex((user) => {
    return user.id === typedid;
  });
  if (finduserid === -1) return res.sendStatus(404);
  mockuser[finduserid] = { ...mockuser[finduserid], ...body };
  return res.sendStatus(200);
});
```

This is nearly identical to `PUT`, with one key difference on the update line:

```javascript
mockuser[finduserid] = { ...mockuser[finduserid], ...body };
```

- It spreads the **existing user first**, then spreads `body` on top. Any field present in `body` overwrites the old value; any field _not_ mentioned in `body` is kept as-is.
- This is the defining trait of **PATCH**: it's a _partial_ update. Sending `{ name: "new name" }` only changes `name` — `age` and `id` stay untouched (mostly — see the bug below).

> ⚠️ **Same id-override bug as PUT:** since `body` is spread last, a client sending `{ "id": 999 }` in a PATCH request would still overwrite the record's real id. Same fix applies:
> 
> ```javascript
> mockuser[finduserid] = { ...mockuser[finduserid], ...body, id: typedid };
> ```

---

## 5. DELETE — Remove

```javascript
app.delete("/api/users/:id", (req, res) => {
  const {
    params: { id },
  } = req;
  const typedid = parseInt(id);
  if (isNaN(typedid)) return res.status(400).send("not a valid id");
  const finduserid = mockuser.findIndex((user) => {
    return user.id === typedid;
  });
  if (finduserid === -1) return res.sendStatus(404);
  mockuser.splice(finduserid);
  return res.sendStatus(200);
});
```

Same lookup pattern as `PUT`/`PATCH`, but instead of reassigning the slot, it removes it with `Array.prototype.splice()`.

> ⚠️ **Bug — this deletes far more than intended:** `splice(start)` **without a second argument removes every element from `start` to the end of the array**, not just one item. So deleting user `id: 2` (index `1`) doesn't just remove that user — it wipes out every user after it too.
> 
> **Fix — always pass a delete count:**
> 
> ```javascript
> mockuser.splice(finduserid, 1); // remove exactly 1 element at that index
> ```
> 
> This is one of the most common `splice()` mistakes in JavaScript, so it's worth committing to memory: **`splice(index, count)`** — both arguments matter.

---

## 6. Quick Reference: The Four Methods

|Method|Purpose|Needs `req.body`?|Needs `:id`?|Typical success status|
|---|---|---|---|---|
|`POST`|Create a new resource|Yes|No (server assigns id)|`201 Created`|
|`PUT`|Replace a resource entirely|Yes|Yes|`200 OK` (or `204 No Content`)|
|`PATCH`|Update part of a resource|Yes|Yes|`200 OK`|
|`DELETE`|Remove a resource|No|Yes|`200 OK` or `204 No Content`|

**PUT vs. PATCH in one sentence:** PUT says _"here is the complete new version of this resource"_; PATCH says _"here are the specific fields to change."_

---

## 7. Bugs Found (Recap)

1. **POST** doesn't add the new user to `mockuser` — it only reflects the body back, so nothing is actually created.
2. **PUT** and **PATCH** both spread `body` _after_ setting `id`, so a client-supplied `id` in the request body silently overwrites the real id. Spread `body` first, then force `id` last.
3. **DELETE** uses `mockuser.splice(finduserid)` with no delete count, which removes that user _and every user after it_ in the array, instead of just one. Should be `mockuser.splice(finduserid, 1)`.
4. None of the mutation routes send back the updated/created resource in the response — only a bare status code — which makes it harder for a client to confirm what actually changed.