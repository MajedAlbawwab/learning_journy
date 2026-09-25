# Express.js Basics — Code Walkthrough

This is a breakdown of a small Express.js API that serves mock user data, with routes for a homepage, listing users, fetching one user by URL parameter, and filtering users by query parameter.

---

## 1. Setup

```javascript
import express from "express";
const app = express();
const PORT = 3000;
```

- `import express from "express"` — loads the Express framework (ES module syntax, so the project's `package.json` needs `"type": "module"`).
- `express()` — creates an Express **application instance**. This `app` object is what you attach routes and middleware to.
- `PORT` — just a constant holding the port number the server will listen on.

---

## 2. Mock Data

```javascript
const mockuser = [
  { id: 1, name: "majed", age: "20" },
  { id: 2, name: "majed1o", age: "201" },
  { id: 3, name: "majed2", age: "202" },
  { id: 4, name: "majed3o", age: "203" },
];
```

A hardcoded array acting as a fake "database" for the demo. Note `age` is stored as a **string**, not a number — fine for a demo, but worth knowing since it can bite you later if you try to do math on it.

---

## 3. Basic Routes

```javascript
app.get("/", (req, res) => {
  res.send("wow hi !");
});

app.get("/api/users", (req, res) => {
  res.send(mockuser);
});
```

- `app.get(path, handler)` — registers a route that responds to `GET` requests on `path`.
- The handler always gets `(req, res)`:
    - `req` = the incoming request (params, query, headers, body, etc.)
    - `res` = the response you build and send back
- `res.send(...)` — sends data back to the client. Express is smart about the `Content-Type`:
    - A string → sent as plain text/HTML.
    - An object/array (like `mockuser`) → automatically sent as **JSON**.

So visiting `/` returns `"wow hi !"`, and visiting `/api/users` returns the whole `mockuser` array as JSON.

---

## 4. Handling URL Params (`/api/users/:id`)

```javascript
app.get("/api/users/:id", (req, res) => {
  const typedid = parseInt(req.params.id);

  if (isNaN(typedid)) {
    return res.statusCode.send("not invalid numper");
  }

  const finduser = mockuser.find((user) => user.id === typedid);
  if (!finduser) return res.sendStatus(404);

  return res.send(finduser);
});
```

**What `:id` means:** the colon marks a **route parameter**. Anything in that position of the URL gets captured into `req.params.id` as a **string**. Example: visiting `/api/users/3` gives you `req.params.id === "3"`.

Step by step:

1. `parseInt(req.params.id)` — converts the string param into a number, since `mockuser[i].id` is a number and `"3" === 3` would be `false` in JS.
2. `isNaN(typedid)` — checks if the conversion failed (e.g. someone visits `/api/users/banana`). If so, it should return a `400 Bad Request` with an error message.
3. `mockuser.find(...)` — searches the array for a user whose `id` matches. Returns `undefined` if nothing matches.
4. `if (!finduser) return res.sendStatus(404)` — if no user was found, send a `404 Not Found` status with no custom body (`sendStatus` sends the status code _and_ its default text, e.g. `"Not Found"`).
5. Otherwise, send back the found user as JSON.

> ⚠️ **Bug here:** `res.statusCode.send(...)` is invalid. `res.statusCode` is just a _number property_, not a function — it has no `.send()` method. This line will crash with `TypeError: res.statusCode.send is not a function`.
> 
> **Fix:**
> 
> ```javascript
> if (isNaN(typedid)) {
>   return res.status(400).send("Not a valid number");
> }
> ```
> 
> `res.status(code)` sets the HTTP status and returns `res` so you can chain `.send()` off it.

---

## 5. Handling Query Params (`/api/test`)

```javascript
// the URL IS /api/test?filter=name&&?value=o
app.get("/api/test", (req, res) => {
  const {
    query: { filter, value },
  } = req;

  if (!filter && !value) return res.send(mockuser);

  if (filter && value)
    return res.send(mockuser.filter((user) => user[filter].includes(value)));
});
```

**Query params** are the `?key=value&key2=value2` part of a URL. Express parses them automatically into `req.query` as an object.

- `const { query: { filter, value } } = req;` — this is **object destructuring**. It's equivalent to:
    
    ```javascript
    const filter = req.query.filter;const value = req.query.value;
    ```
    
- If **neither** `filter` nor `value` is provided → return the full unfiltered list.
- If **both** are provided → filter `mockuser` down to entries where `user[filter]` (dynamic property access, e.g. `user["name"]`) contains `value` as a substring, using `String.prototype.includes()`.

**Example URL:** `/api/test?filter=name&value=o` → returns every user whose `name` contains `"o"`.

> ⚠️ **Bug in the comment/URL example:** `?filter=name&&?value=o` is not valid query string syntax. Query params are joined with a single `&`, not `&&`, and there's no second `?`. It should be:
> 
> ```
> /api/test?filter=name&value=o
> ```

> ⚠️ **Logic gap:** there's no `else` case for when only _one_ of `filter`/`value` is given (e.g. `?filter=name` alone). In that situation, neither `if` branch runs, so Express hangs waiting for a response until the request times out. Also, if `filter` refers to a property that doesn't exist on `user` (e.g. `?filter=email&value=x`), `user[filter]` is `undefined`, and calling `.includes()` on it throws a `TypeError`. A safer version:
> 
> ```javascript
> app.get("/api/test", (req, res) => {
>   const { filter, value } = req.query;
> 
>   if (!filter || !value) return res.send(mockuser);
> 
>   const validFields = ["name", "age"]; // whitelist to avoid crashes
>   if (!validFields.includes(filter)) {
>     return res.status(400).send("Invalid filter field");
>   }
> 
>   return res.send(
>     mockuser.filter((user) => String(user[filter]).includes(value))
>   );
> });
> ```

---

## 6. Starting the Server

```javascript
app.listen(PORT, () => {
  console.log("woooow");
});
```

- `app.listen(port, callback)` — starts the HTTP server, binding it to `PORT` (3000).
- The callback runs once the server successfully starts listening — commonly used to log a "server running" message.

---

## 7. Quick Reference: `req.params` vs `req.query`

|Feature|`req.params`|`req.query`|
|---|---|---|
|Comes from|Route path segments (`/users/:id`)|The `?key=value` part of the URL|
|Defined by|The route pattern itself|Whatever the client appends|
|Example URL|`/api/users/3`|`/api/test?filter=name&value=o`|
|Access|`req.params.id` → `"3"`|`req.query.filter` → `"name"`|
|Type|Always a string|Always a string (or array/object for repeated/nested keys)|

---

## 8. Summary of Routes

|Method|Path|Purpose|
|---|---|---|
|GET|`/`|Sanity-check route, returns plain text|
|GET|`/api/users`|Returns all mock users|
|GET|`/api/users/:id`|Returns one user by numeric ID|
|GET|`/api/test`|Filters users by a `field=value` substring match|

---

## 9. Bugs Found (Recap)

1. `res.statusCode.send(...)` → should be `res.status(400).send(...)`.
2. Malformed example query string in the comment (`&&?` → should just be `&`).
3. No handling for the case where only one of `filter`/`value` is passed — request never gets a response.
4. `user[filter]` isn't validated, so an unknown field name causes a runtime crash on `.includes()`.
5. `parseInt(req.params.id)` has no radix argument (`parseInt(x, 10)` is the safer habit, though not a bug here since IDs are plain digits).