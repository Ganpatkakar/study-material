# Node.js Study Material

## 1. What is Node.js?

Node.js is a JavaScript runtime built on Chrome's V8 engine. It allows developers to run JavaScript on the server side and build fast, scalable network applications.

### Why use Node.js?
- Uses JavaScript across frontend and backend
- Non-blocking, event-driven architecture
- Great for APIs, real-time systems, and microservices
- Large ecosystem via npm

---

## 2. Prerequisites

Before learning Node.js, you should be comfortable with:
- JavaScript basics: variables, functions, loops, arrays, objects
- ES6+ features: let/const, arrow functions, template literals, destructuring, modules
- DOM knowledge is not required for Node.js, but understanding JavaScript fundamentals is essential

---

## 3. Setup

### Install Node.js
- Download from: https://nodejs.org/
- Verify installation:

```bash
node -v
npm -v
```

### Create a project

```bash
mkdir my-node-app
cd my-node-app
npm init -y
```

---

## 4. Core JavaScript Concepts to Revise

### Variables and data types
```javascript
const name = 'Alice';
let age = 25;
var isStudent = true;
```

### Functions
```javascript
function greet(name) {
  return `Hello, ${name}!`;
}

const greetArrow = (name) => `Hello, ${name}!`;
```

### Arrays and objects
```javascript
const fruits = ['apple', 'banana', 'orange'];
const user = { name: 'Bob', age: 30 };
```

### Destructuring
```javascript
const { name, age } = user;
const [firstFruit] = fruits;
```

### Modules
```javascript
// math.js
export function add(a, b) {
  return a + b;
}

// app.js
import { add } from './math.js';
console.log(add(2, 3));
```

---

## 5. Node.js Fundamentals

### REPL
Node.js has a built-in REPL (Read-Eval-Print-Loop):

```bash
node
```

### Running a script
```bash
node app.js
```

### Global objects
Common global objects in Node.js include:
- `console`
- `process`
- `Buffer`
- `setTimeout`, `setInterval`
- `require` (CommonJS)

### CommonJS vs ES Modules
Node.js supports both:

#### CommonJS
```javascript
const fs = require('fs');
```

#### ES Modules
```javascript
import fs from 'fs';
```

To use ES modules in Node.js, often add this in `package.json`:

```json
{
  "type": "module"
}
```

---

## 6. Built-in Modules

Node.js includes many built-in modules:
- `fs` - file system operations
- `path` - working with file paths
- `http` - creating HTTP servers
- `os` - operating system details
- `events` - event emitter
- `stream` - streaming data
- `url` - URL parsing

### Example: read a file
```javascript
const fs = require('fs');

fs.readFile('example.txt', 'utf8', (err, data) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(data);
});
```

---

## 7. Asynchronous Programming

One of the most important concepts in Node.js is asynchronous behavior.

### Callback example
```javascript
setTimeout(() => {
  console.log('This runs after 1 second');
}, 1000);
```

### Promise example
```javascript
const fetchData = () => {
  return new Promise((resolve, reject) => {
    setTimeout(() => resolve('Data received'), 1000);
  });
};

fetchData().then((data) => console.log(data));
```

### Async/Await example
```javascript
const run = async () => {
  const data = await fetchData();
  console.log(data);
};

run();
```

### Important concept
Node.js is designed around the event loop, which helps it handle many requests efficiently without blocking.

---

## 8. HTTP Server

### Basic HTTP server
```javascript
const http = require('http');

const server = http.createServer((req, res) => {
  res.statusCode = 200;
  res.setHeader('Content-Type', 'text/plain');
  res.end('Hello from Node.js!');
});

server.listen(3000, () => {
  console.log('Server running on http://localhost:3000');
});
```

### What to learn here
- HTTP methods: GET, POST, PUT, DELETE
- Request and response objects
- Routing
- Status codes
- Middleware

---

## 9. Express.js

Express.js is the most popular Node.js framework for building web applications and APIs.

### Install Express
```bash
npm install express
```

### Example app
```javascript
const express = require('express');
const app = express();

app.get('/', (req, res) => {
  res.send('Welcome to Express!');
});

app.listen(3000, () => {
  console.log('App running on port 3000');
});
```

### Express topics
- Routing
- Middleware
- Request/response handling
- Express validators
- API design
- REST patterns

---

## 10. Working with Databases

Node.js commonly connects to databases such as:
- MongoDB
- PostgreSQL
- MySQL
- SQLite

### Common ORMs / query tools
- Mongoose (MongoDB)
- Prisma
- Sequelize
- Knex

### Typical flow
- Connect to database
- Define schema/model
- CRUD operations
- Error handling
- Validation

---

## 11. Authentication and Security

Important Node.js application topics:
- JWT authentication
- Password hashing (bcrypt)
- Input validation
- CORS
- Helmet
- Rate limiting
- Environment variables

### Example using environment variables
```javascript
require('dotenv').config();

const port = process.env.PORT || 3000;
console.log(port);
```

---

## 12. Package Management with npm

npm is Node.js's package manager.

### Install a package
```bash
npm install lodash
```

### Save as dev dependency
```bash
npm install -D nodemon
```

### Run scripts from package.json
```json
{
  "scripts": {
    "dev": "node app.js",
    "start": "node server.js"
  }
}
```

---

## 13. Testing in Node.js

You should learn:
- Unit testing
- Integration testing
- API testing

### Popular tools
- Jest
- Mocha
- Chai
- Supertest

### Example Jest test
```javascript
test('adds numbers correctly', () => {
  expect(2 + 2).toBe(4);
});
```

---

## 14. Debugging and Error Handling

Learn to:
- Use `console.log` and breakpoints
- Read stack traces
- Handle errors with try/catch
- Use `process.env.NODE_ENV`
- Log structured messages in production

### Example
```javascript
try {
  // risky code
} catch (error) {
  console.error('Something went wrong:', error.message);
}
```

---

## 15. Deployment and Production

When moving from development to production, learn:
- Environment variables
- Process managers (`pm2`)
- Docker basics
- Reverse proxies (Nginx)
- Hosting platforms: Render, Vercel, Railway, Heroku, AWS, Azure
- CI/CD basics

---

## 16. Project Ideas for Practice

### Beginner projects
- To-do app API
- Weather app
- Blog API
- URL shortener
- Notes application

### Intermediate projects
- Chat application with Socket.IO
- Authentication system
- Ecommerce backend
- Real-time dashboard
- File upload service

### Advanced projects
- Microservice architecture
- Event-driven system
- Social media backend
- Job processing queue system

---

## 17. Recommended Roadmap

### Week 1: JavaScript basics
- Variables, functions, arrays, objects
- ES6+ syntax
- Scope and closures

### Week 2: Node.js basics
- Node runtime
- Modules
- File system
- npm

### Week 3: Asynchronous JavaScript
- callbacks
- promises
- async/await
- event loop

### Week 4: HTTP and Express
- HTTP server creation
- REST API design
- Express routes and middleware

### Week 5: Database integration
- MongoDB or PostgreSQL
- CRUD APIs
- ORM usage

### Week 6: Security and testing
- Auth
- Validation
- JWT
- Testing

### Week 7: Deployment
- Environment setup
- Hosting
- Debugging
- Production optimization

---

## 18. Useful Resources

### Documentation
- Node.js official docs: https://nodejs.org/docs/
- npm docs: https://docs.npmjs.com/
- Express docs: https://expressjs.com/

### Learning platforms
- MDN JavaScript: https://developer.mozilla.org/en-US/docs/Web/JavaScript
- freeCodeCamp: https://www.freecodecamp.org/
- W3Schools: https://www.w3schools.com/
- NodeSchool: https://nodeschool.io/

### Practice tools
- GitHub
- Postman
- VS Code
- MongoDB Atlas / PostgreSQL setup

---

## 19. Core Interview Topics

Prepare for questions on:
- Event loop
- Callback vs Promise vs Async/Await
- Node.js modules
- Streams
- Middleware in Express
- REST API design
- Authentication and authorization
- Database connections
- Error handling
- Performance optimization

---

## 20. Final Tips

- Practice by building small projects instead of only reading theory
- Learn debugging early
- Focus on clean API design
- Read documentation regularly
- Keep a GitHub repository with your practice projects
- Build one full-stack project to connect frontend and backend knowledge

---

## 21. Quick Start Template

```bash
mkdir node-api
cd node-api
npm init -y
npm install express
```

```javascript
const express = require('express');
const app = express();

app.get('/health', (req, res) => {
  res.json({ status: 'ok' });
});

app.listen(3000, () => {
  console.log('Server running on port 3000');
});
```

Run:
```bash
node app.js
```

This is a simple starting point for learning Node.js backend development.

---

## Summary

Node.js is a powerful runtime for building fast, scalable, and modern backend applications using JavaScript. Start with JavaScript fundamentals, move through async programming and HTTP, then practice with Express and database-backed projects.

A strong learning approach is:
1. Understand JavaScript well
2. Learn Node basics
3. Build simple APIs
4. Add databases and authentication
5. Deploy and iterate

This path will help you become confident in backend development with Node.js.
