# 🖋📚 Sample Projects and Code Snippets 📚🖋
Diving into practical examples can sometimes be the most enlightening way to truly grasp a concept. While we've explored Middleware-Oriented Architecture (MOA) at a theoretical level, seeing it in action is where the real magic happens. In this section, we'll journey through some sample projects and bite-sized code snippets, tailored to provide hands-on insight into how MOA can be elegantly integrated into your applications. Whether you're a novice just getting your feet wet or a seasoned developer looking for implementation nuances, this section promises insights for all. Let's roll up our sleeves and delve into the world of MOA, one example at a time!

## 📖🔐 Basic Authenticated Contact List API 🔐📖

Creating an authenticated contact list is a common requirement in many web applications. By using MOA with Express.js, we can simplify and modularize this task. Here's a step-by-step guide on setting up such an API.

Setup
Before we dive into code, ensure you've the necessary packages installed:

```bash
npm install express body-parser sqlite3 jsonwebtoken
```

**Middleware: Authentication**

Let's start by creating a simple authentication middleware:

```javascript
const jwt = require('jsonwebtoken');

function authenticate(req, res, next) {
    const token = req.headers['authorization'];

    if (!token) {
        return res.status(401).send('Access Denied. No token provided.');
    }

    try {
        const decoded = jwt.verify(token, 'your-secret-key');
        req.user = decoded;
        next();
    } catch (err) {
        res.status(400).send('Invalid token.');
    }
}
```

**Middleware: Fetch Contacts**

Assuming we have a SQLite database with a contacts table:

```javascript
const sqlite3 = require('sqlite3').verbose();
const db = new sqlite3.Database('./contacts.db');

function fetchContacts(req, res, next) {
    db.all("SELECT * FROM contacts WHERE userId = ?", [req.user.id], (err, rows) => {
        if (err) {
            return next(err);
        }
        
        req.contacts = rows;
        next();
    });
}
```

**Setting up Routes**

Using the authentication and fetchContacts middlewares:

```javascript
const express = require('express');
const app = express();

app.get('/contacts', authenticate, fetchContacts, (req, res) => {
    res.json(req.contacts);
});

app.listen(3000, () => {
    console.log('Server started on http://localhost:3000');
});
```

This simple API setup ensures that only authenticated users can fetch their contacts. By chaining the authenticate and fetchContacts middlewares, we've streamlined the request process in a modular and readable manner.

## 🔒🔄 Enhancements for Transactional Consistency 🔄🔒

Transactional consistency ensures that a sequence of operations either completes entirely or doesn't execute at all. In web applications, especially those that interact with databases, it's crucial to manage transactions to maintain data integrity.

By using MOA in our Express.js application, we can introduce middlewares to handle transactions elegantly. Here's how:

Setup
To handle database transactions, make sure you have SQLite and related modules set up:

```bash
npm install express body-parser sqlite3
```

**Middleware: Start Transaction**

Initiate a transaction before any database interaction:

```javascript
function startTransaction(req, res, next) {
    req.db = new sqlite3.Database('./contacts.db');
    req.db.serialize(() => {
        req.db.run("BEGIN TRANSACTION");
        next();
    });
}
```

**Middleware: Commit Transaction**

Upon successful execution of subsequent middlewares, commit the changes:

```javascript
function commitTransaction(req, res, next) {
    req.db.run("COMMIT", () => {
        req.db.close();
        next();
    });
}
```

**Middleware: Rollback Transaction**

In case of errors in any of the middlewares, roll back the transaction to maintain data consistency:

```javascript
function rollbackTransaction(err, req, res, next) {
    req.db.run("ROLLBACK", () => {
        req.db.close();
        // Pass the error to the next middleware (like an error handler)
        next(err);
    });
}
```

**Setting up Routes**

Integrate these middlewares into your routes:

```javascript
const express = require('express');
const app = express();

app.post('/addContact', startTransaction, authenticate, addContact, commitTransaction, (req, res) => {
    res.json({ message: 'Contact added successfully!' });
});

// Error handling middleware
app.use(rollbackTransaction);
app.use((err, req, res, next) => {
    res.status(500).json({ message: 'Something went wrong!' });
});

app.listen(3000, () => {
    console.log('Server started on http://localhost:3000');
});
```

**Key Points**

- Start the transaction at the beginning of the middleware chain.

- If everything goes smoothly, commit the transaction.

- If any middleware encounters an error, the rollback middleware ensures that changes are not committed, maintaining the database's integrity.

- The error is then passed on to the general error handling middleware.

- This design ensures that our application can perform multiple operations in an atomic manner, safeguarding the integrity and consistency of our data. By modularizing the transaction logic into separate middlewares, we can easily incorporate transactional consistency across various routes.

## 🔗🧩 Code Snippets for Common Patterns 🧩🔗
In the process of developing applications using MOA (Middleware-Oriented Architecture), certain patterns emerge that are frequently utilized. Below, we've gathered some of the most common patterns, providing reusable code snippets that you can integrate into your projects.

**Authentication Middleware**

Ensuring the authenticity of the request sender:

```javascript
function authenticate(req, res, next) {
    const token = req.headers['authorization'];

    // Mock token verification
    if (token !== 'Bearer valid_token') {
        return res.status(401).json({ message: 'Unauthorized' });
    }

    next();
}
```

**Logging Middleware**

Capture details about the incoming request:

```javascript
function logger(req, res, next) {
    console.log(`[${new Date().toISOString()}] ${req.method} ${req.path}`);
    next();
}
```

**Rate Limiting Middleware**

Restrict the number of requests in a given time period:

```javascript
const rateLimiter = (() => {
    const requests = {};

    return (req, res, next) => {
        const ip = req.ip;
        if (!requests[ip]) {
            requests[ip] = 1;
        } else if (requests[ip] < 5) {
            requests[ip]++;
        } else {
            return res.status(429).json({ message: 'Too many requests' });
        }
        next();
    };
})();
```

**Data Validation Middleware**

Ensure the request body conforms to specific rules:

```javascript
function validateContact(req, res, next) {
    const { name, email } = req.body;

    if (!name || !email) {
        return res.status(400).json({ message: 'Name and email are required.' });
    }

    next();
}
```

**Caching Middleware**

Cache the result of certain operations to improve performance:

```javascript
const cache = {};

function cachingMiddleware(req, res, next) {
    const key = `${req.method}_${req.path}`;
    if (cache[key]) {
        return res.json(cache[key]);
    }

    // Capture original send method
    const originalSend = res.send;
    res.send = (data) => {
        cache[key] = data;
        originalSend.call(res, data);
    };

    next();
}
```

**CORS Handling Middleware**

Manage Cross-Origin Resource Sharing settings for your API:

```javascript
function handleCORS(req, res, next) {
    res.header("Access-Control-Allow-Origin", "*");
    res.header("Access-Control-Allow-Headers", "Origin, X-Requested-With, Content-Type, Accept, Authorization");
    next();
}
```

These are just a few common patterns and their respective middleware implementations. MOA, by design, encourages the creation of modular and reusable components, ensuring that over time, as your application grows, your code remains clean, organized, and efficient.
