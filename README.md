# Express-Login-Middleware

# Express Login Middleware

Express middleware দিয়ে login-এর আগে email + password check।

## Full Flow

```text
POST /login
    ↓
loginMiddleware
    ↓
Email check
    ↓
Password check
    ↓
Correct
    ↓
next()
    ↓
login controller
```

Wrong email/password → `login` controller execute হবে না।

---

## Project Structure

```text
project/
│
├── controllers/
│   └── auth.controllers.js
│
├── middleware/
│   └── auth.middleware.js
│
├── routes/
│   └── auth.routes.js
│
└── index.js
```

---

## 1. Middleware

### `middleware/auth.middleware.js`

```js
const loginMiddleware = (req, res, next) => {
    const { email, password } = req.body;

    if (email !== "admin@gmail.com") {
        return res.status(401).json({
            message: "Invalid email"
        });
    }

    if (password !== "123456") {
        return res.status(401).json({
            message: "Invalid password"
        });
    }

    next();
};

module.exports = loginMiddleware;
```

### কাজ

`req.body` থেকে email + password নেয়:

```js
const { email, password } = req.body;
```

Email check:

```js
if (email !== "admin@gmail.com") {
    return res.status(401).json({
        message: "Invalid email"
    });
}
```

Password check:

```js
if (password !== "123456") {
    return res.status(401).json({
        message: "Invalid password"
    });
}
```

দুটো correct হলে:

```js
next();
```

`next()` → পরের controller-এ request পাঠায়।

---

# 2. Router

### `routes/auth.routes.js`

```js
const express = require("express");

const {
    register,
    login,
    logout,
} = require("../../controllers/auth.controllers");

const loginMiddleware = require("../../middleware/auth.middleware");

const router = express.Router();

router.post("/register", register);

router.post("/login", loginMiddleware, login);

router.get("/logout", logout);

module.exports = router;
```

Main line:

```js
router.post("/login", loginMiddleware, login);
```

এখানে:

```text
/login
  ↓
loginMiddleware
  ↓
login
```

আগে middleware execute হবে।

Middleware `next()` দিলে → `login` execute হবে।

---

# 3. Controller

### `controllers/auth.controllers.js`

```js
const login = (req, res) => {
    res.json({
        message: "Login successful"
    });
};

module.exports = {
    login
};
```

Middleware থেকে `next()` আসলে `login()` execute হবে।

---

# 4. Main Server

### `index.js`

```js
const express = require("express");

const app = express();

app.use(express.json());

const authRouter = require("./routes/auth.routes");

app.use("/auth", authRouter);

app.listen(5000, () => {
    console.log("Server running on port 5000");
});
```

---

# 5. API

Server:

```text
http://localhost:5000
```

Login:

```text
POST /auth/login
```

Full URL:

```text
http://localhost:5000/auth/login
```

---

# 6. Correct Login

Request:

```json
{
    "email": "admin@gmail.com",
    "password": "123456"
}
```

Flow:

```text
POST /auth/login
        ↓
loginMiddleware
        ↓
email correct
        ↓
password correct
        ↓
next()
        ↓
login()
        ↓
Login successful
```

Response:

```json
{
    "message": "Login successful"
}
```

---

# 7. Wrong Email

Request:

```json
{
    "email": "user@gmail.com",
    "password": "123456"
}
```

Flow:

```text
POST /auth/login
        ↓
loginMiddleware
        ↓
email wrong
        ↓
401
        ↓
STOP
```

Response:

```json
{
    "message": "Invalid email"
}
```

`login()` execute হবে না।

---

# 8. Wrong Password

Request:

```json
{
    "email": "admin@gmail.com",
    "password": "wrong123"
}
```

Flow:

```text
POST /auth/login
        ↓
loginMiddleware
        ↓
email correct
        ↓
password wrong
        ↓
401
        ↓
STOP
```

Response:

```json
{
    "message": "Invalid password"
}
```

`login()` execute হবে না।

---

# 9. Middleware Basic Structure

General middleware:

```js
const middleware = (req, res, next) => {

    // check

    next();
};

module.exports = middleware;
```

Route:

```js
router.post("/login", middleware, controller);
```

Flow:

```text
Request
   ↓
Middleware
   ↓
Check
   ↓
next()
   ↓
Controller
   ↓
Response
```

`next()` না দিলে → request পরের controller-এ যাবে না।

---

# 10. Multiple Middleware

একাধিক middleware-ও use করা যায়:

```js
router.post(
    "/login",
    middleware1,
    middleware2,
    login
);
```

Flow:

```text
Request
   ↓
middleware1
   ↓
next()
   ↓
middleware2
   ↓
next()
   ↓
login
```

যেকোনো middleware থেকে response দিলে → flow stop।

---

# 11. Important Note

এই example শুধু middleware শেখার জন্য।

Hardcoded credential:

```js
"admin@gmail.com"
"123456"
```

Production auth-এ use করা উচিত না।

Real project flow:

```text
Login
  ↓
DB
  ↓
Email find
  ↓
Password verify
  ↓
JWT / Session
  ↓
Login success
```
