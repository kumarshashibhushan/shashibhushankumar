## POST /users/register

Description
---------

Register a new user. Creates a user record, hashes the password, and returns a JSON Web Token plus the created user object (password is not returned).

Base route (from `user.routes.js`): `/register` — typically mounted under `/users`, resulting in `/users/register`.

Request
-------

Headers
- `Content-Type: application/json`

Body (JSON)
- `fullname` (object)
  - `firstname` (string) — required, minimum 3 characters
  - `lastname` (string) — optional, minimum 3 characters if provided
- `email` (string) — required, must be a valid email address, minimum 5 characters
- `password` (string) — required, minimum 6 characters

Example request body

```
{
  "fullname": {
    "firstname": "John",
    "lastname": "Doe"
  },
  "email": "john.doe@example.com",
  "password": "s3cret123"
}
```

Validation rules (from `user.routes.js` and `user.model.js`)
- `email`: must be a valid email (express-validator `.isEmail()`)
- `fullname.firstname`: required, minimum 3 characters (`.isLength({ min: 3 })`)
- `password`: required, minimum 6 characters (`.isLength({ min: 6 })`)

Responses
---------

- 201 Created
  - Description: User registered successfully.
  - Body: `{ token: <jwt>, user: <user object> }`
  - Notes: `user` will not include the `password` field (the schema sets `select: false`).

- 400 Bad Request
  - Description: Validation errors (missing/invalid fields) or user already exists.
  - Body examples:
    - Validation errors: `{ errors: [ { msg, param, location, ... } ] }`
    - User exists: `{ message: 'User already exist' }`

- 500 Internal Server Error
  - Description: Unexpected server error while creating the user.

Implementation details / contract
-------------------------------

- Input: JSON body with `fullname`, `email`, `password` as described above.
- Output: On success, HTTP 201 with a JWT and the created user object.
- Error modes: validation failures (400), duplicate email (400), server/database issues (500).

Edge cases
----------

- Missing `fullname` or `fullname.firstname` — results in a 400 validation error.
- Email already in use — controller returns 400 with a message.
- Password length too short — validation error (400).

Notes
-----

- The route handler in `controllers/user.controller.js` hashes the password via `userModel.hashPassword` before calling `userService.createUser`.
- The service `createUser` does basic presence checks and calls `userModel.create`. If you want stronger validation, consider moving more checks into the service or model-level validators.

Contact / Further work
----------------------

If you want, I can also add a short automated test for this endpoint and a Postman collection example.
