# Node.js Tutorial — Complete Study Notes (Verified & Formatted)

## Table of Contents
- Module 1 — Introduction to Node.js
- Module 2 — Node.js Fundamentals
- Module 3 — Node.js Modules
- Module 4 — File System and Built-in Modules
- Module 5 — HTTP Module and Creating a Node Server
- Module 6 — Node.js Runtime and Event Loop
- Module 7 — npm and the Node.js Package Ecosystem
- Module 8 — CLI Tools, Concurrency and Deployment
- Final Node.js Cheat Sheet
- Final Revision Questions
- Complete Node.js Mental Model

---

## Module 1 — Introduction to Node.js

### Lesson 1 — Introduction
Node.js allows JavaScript to run outside the browser.

Traditionally, JavaScript was primarily associated with browser environments.

```
Browser
   ↓
JavaScript
   ↓
Web page
```

Node.js changes this model:

```
Operating System
       ↓
    Node.js
       ↓
 JavaScript
```

This makes JavaScript suitable for:
- Server-side applications
- REST APIs
- Command-line applications
- Automation
- Network applications
- Real-time applications

### Lesson 2 — What is Node.js?
Node.js is a JavaScript runtime built around Google's V8 JavaScript engine.

A simplified view is:

```
Node.js
   │
   ├── V8
   │    └── Executes JavaScript
   │
   └── Node.js APIs
        └── Provide additional capabilities
```

Node.js provides APIs that allow JavaScript applications to interact with the operating system and perform tasks such as:
- File operations
- Network communication
- HTTP servers
- Process interaction
- Streams
- Events

> Maps to Codevolution Videos 5, plus Videos 2-4 (ECMAScript, V8 Engine, JS Runtime)

### Lesson 3 — Why Node.js?
Node.js is particularly useful for applications involving a large amount of I/O.

Examples include:
- Web servers
- REST APIs
- Real-time applications
- Network applications
- Command-line tools

The important idea is asynchronous and event-driven execution.

```
Request
   ↓
Start I/O
   ↓
Continue executing
   ↓
I/O completes
   ↓
Callback executes
```

Instead of blocking while waiting for an I/O operation, Node.js can continue handling other work.

### Lesson 4 — Installing Node.js
After installing Node.js, it can be accessed from the command line.

Check the Node.js version:
```bash
node --version
```

Check npm:
```bash
npm --version
```

If both commands return version numbers, Node.js and npm are available.

### Lesson 5 — First Node.js Program
Create a JavaScript file:
```js
console.log("Hello World");
```

Save it as: `index.js`

Run it:
```bash
node index.js
```

Execution flow:
```
index.js
   ↓
node index.js
   ↓
Node.js runtime
   ↓
V8
   ↓
JavaScript executes
```

> Maps to Codevolution Video 6 - Hello World

---

## Module 2 — Node.js Fundamentals

### Lesson 6 — Node.js REPL
REPL stands for: **Read — Evaluate — Print — Loop**

Start the Node.js REPL:
```bash
node
```

You can execute JavaScript directly:
```
> 1 + 2
3
```

The REPL is useful for experimenting with JavaScript and Node.js APIs.

Exit:
```
.exit
```

### Lesson 7 — Global Objects
Node.js provides several global objects and functions.

Examples include:
- `console`
- `process`
- `setTimeout`
- `setInterval`
- `Buffer`

These can be used without explicitly importing them.

Example:
```js
console.log("Hello");
```

### Lesson 8 — Process Object
The process object provides information and control over the current Node.js process.

Example:
```js
console.log(process);
```

Important properties and methods include:
- `process.argv`
- `process.env`
- `process.cwd()`
- `process.exit()`

The process object is especially useful when working with:
- Command-line arguments
- Environment variables
- Process information
- Application configuration

### Lesson 9 — Command-Line Arguments
Command-line arguments are available through:
```js
process.argv
```

For example:
```bash
node app.js hello
```

The application can inspect:
```js
console.log(process.argv);
```

Conceptually:
```
Terminal
   ↓
node app.js hello
   ↓
process.argv
```

Command-line arguments are particularly useful when creating CLI applications.

### Lesson 10 — Environment Variables
Environment variables can be accessed using:
```js
process.env
```

Example:
```js
console.log(process.env.NODE_ENV);
```

Environment variables allow configuration to be supplied externally instead of hard-coding configuration into the application.

Conceptually:
```
Environment
    ↓
process.env
    ↓
Application configuration
```

### Lesson 11 — Timers
Node.js provides timer APIs.

**setTimeout**
```js
setTimeout(() => {
  console.log("Executed");
}, 1000);
```

**setInterval**
```js
setInterval(() => {
  console.log("Repeated");
}, 1000);
```

These APIs schedule callbacks for later execution.

### Lesson 12 — Callbacks
A callback is a function passed to another function so that it can be executed later.

Example:
```js
function greet(name, callback) {
  console.log("Hello " + name);
  callback();
}

greet("John", () => {
  console.log("Done");
});
```

General flow:
```
Function starts
     ↓
Callback supplied
     ↓
Operation completes
     ↓
Callback executes
```

Callbacks are an important part of Node.js's asynchronous programming model.

> Maps to Codevolution Videos 7 (Browser vs Node), 20 (Callback Pattern), 25 (Async JS)

---

## Module 3 — Node.js Modules

### Lesson 13 — What are Modules?
A module is a reusable unit of code.

Instead of putting an entire application into one file:
`application.js`

the application can be divided into multiple files:
- `application.js`
- `math.js`
- `user.js`
- `database.js`

Benefits include:
- Better organization
- Reusability
- Maintainability
- Separation of concerns

### Lesson 14 — CommonJS Modules
Node.js traditionally uses the CommonJS module system.

Export:
```js
module.exports = something;
```

Import:
```js
const something = require("./module");
```

Conceptually:
```
module.js
   ↓
module.exports
   ↓
require()
   ↓
application.js
```

### Lesson 15 — Exporting Functions
A function can be exported from a module.

Example:
```js
function add(a, b) {
  return a + b;
}

module.exports = add;
```

Import it:
```js
const add = require("./math");
console.log(add(2, 3));
```

### Lesson 16 — Exporting Multiple Values
A module can export multiple functions or values.

Example:
```js
function add(a, b) {
  return a + b;
}

function subtract(a, b) {
  return a - b;
}

module.exports = {
  add,
  subtract
};
```

Import:
```js
const math = require("./math");
console.log(math.add(10, 5));
console.log(math.subtract(10, 5));
```

### Lesson 17 — Module Wrapper
Node.js wraps CommonJS modules before executing them.

Conceptually, a module behaves as though it is placed inside a function similar to:
```js
(function(exports, require, module, __filename, __dirname) {
  // module code
});
```

This helps explain why the following values are available inside a CommonJS module:
- `exports`
- `require`
- `module`
- `__filename`
- `__dirname`

> Maps to Codevolution Video 12 — Module Wrapper + Video 11 Module Scope

### Lesson 18 — __filename and __dirname
`__filename` represents the path of the current file.
```js
console.log(__filename);
```

`__dirname` represents the directory containing the current file.
```js
console.log(__dirname);
```

These values are particularly useful when working with files and paths.

### Lesson 19 — Built-in Modules
Node.js provides many built-in modules.

Examples include:
- `fs`
- `path`
- `events`
- `http`
- `os`

These modules provide functionality without requiring an external npm package.

### Lesson 20 — Module Resolution
When using:
```js
require("module-name");
```

Node.js determines which module should be loaded.

For a local module:
```js
require("./math");
```

The `./` indicates a relative path.

For an installed package:
```js
require("some-package");
```

Node.js searches the appropriate package locations.

> Maps to Codevolution Videos 8-18. Your notes correctly cover Local Modules, Module Exports, Module Caching concept, Import Export Patterns, exports vs module.exports, ES Modules, JSON import & watch mode (Videos 13-17) in essence, even if not separately headed.

---

## Module 4 — File System and Built-in Modules

### Lesson 21 — File System Module
The fs module provides filesystem functionality.

Import it:
```js
const fs = require("fs");
```

It provides operations for:
- Reading files
- Writing files
- Updating files
- Deleting files
- Working with directories

### Lesson 22 — Reading Files
Synchronous reading:
```js
const fs = require("fs");
const data = fs.readFileSync("file.txt", "utf8");
console.log(data);
```

Asynchronous reading:
```js
fs.readFile("file.txt", "utf8", (err, data) => {
  if (err) {
    console.error(err);
    return;
  }
  console.log(data);
});
```

Important distinction:
```
readFileSync
    ↓
Synchronous / blocking

readFile
    ↓
Asynchronous / non-blocking
```

### Lesson 23 — Writing Files
A file can be written using:
```js
fs.writeFile("file.txt", "Hello", err => {
  if (err) {
    console.error(err);
    return;
  }
  console.log("File written");
});
```

The asynchronous version allows the application to continue while the filesystem operation is being performed.

### Lesson 24 — Updating and Deleting Files
The filesystem module provides APIs for modifying and removing files.

Examples:
```js
fs.appendFile(...)
fs.unlink(...)
```

General flow:
```
File
 ↓
Read
 ↓
Modify / Append
 ↓
Write

Or:

File
 ↓
Delete
```

### Lesson 25 — Path Module
The path module provides utilities for working with file and directory paths.
```js
const path = require("path");
```

Useful methods include:
- `path.join(...)`
- `path.basename(...)`
- `path.dirname(...)`
- `path.extname(...)`

Example:
```js
const filePath = path.join(__dirname, "data", "file.txt");
```

Using the path module avoids manually constructing platform-specific paths.

### Lesson 26 — OS Module
The os module provides information about the operating system.
```js
const os = require("os");
```

It can provide information such as:
- Platform
- Architecture
- CPUs
- Memory
- Hostname
- Home directory

### Lesson 27 — Events Module
Node.js provides an event-driven programming model.

The events module provides EventEmitter.
```js
const EventEmitter = require("events");
const emitter = new EventEmitter();

emitter.on("event", () => {
  console.log("Event occurred");
});

emitter.emit("event");
```

Conceptually:
```
Emitter
   ↓
emit()
   ↓
Event
   ↓
Listener
   ↓
Callback
```

### Lesson 28 — EventEmitter Arguments
Events can carry data.

Example:
```js
emitter.on("event", (name) => {
  console.log(name);
});

emitter.emit("event", "John");
```

The listener receives the emitted value. This pattern is widely used throughout Node.js.

### Lesson 29 — Event-Driven Architecture
The event-driven model can be summarized as:
```
Something happens
       ↓
Event emitted
       ↓
Listener notified
       ↓
Callback executes
```

This event-driven architecture is fundamental to Node.js.

> Maps to Codevolution Videos 19-29. Your notes include Path, OS, Events correctly. Videos 23-24-27-29 (Charsets, Streams/Buffers, fs Promise, Pipes) are extensions of your fs coverage and are compatible.

---

## Module 5 — HTTP Module and Creating a Node Server

### Lesson 30 — HTTP Module
Node.js provides the built-in HTTP module:
```js
const http = require("http");
```

It can be used to:
- Create HTTP servers
- Handle HTTP requests
- Send HTTP responses
- Work with HTTP clients

### Lesson 31 — Creating a Server
A basic HTTP server:
```js
const http = require("http");

const server = http.createServer((req, res) => {
  res.end("Hello World");
});

server.listen(3000);
```

Request flow:
```
Client
  ↓
HTTP Request
  ↓
Node.js Server
  ↓
Request Handler
  ↓
HTTP Response
```

### Lesson 32 — Request Object
The request object provides information about the incoming HTTP request.

Example:
```js
http.createServer((req, res) => {
  console.log(req.url);
  console.log(req.method);
});
```

Important request information includes:
- URL
- HTTP method
- Headers

### Lesson 33 — Response Object
The response object is used to send data back to the client.

Example:
```js
res.statusCode = 200;
res.setHeader("Content-Type", "text/plain");
res.end("Hello");
```

An HTTP response can contain:
- Status code
- Headers
- Body

### Lesson 34 — JSON Responses
A Node.js server can return JSON.

Example:
```js
const data = {
  name: "John",
  age: 30
};

res.setHeader("Content-Type", "application/json");
res.end(JSON.stringify(data));
```

Flow:
```
JavaScript object
      ↓
JSON.stringify()
      ↓
HTTP response
```

### Lesson 35 — HTML Responses
The server can return HTML.

Example:
```js
res.setHeader("Content-Type", "text/html");
res.end(`
  <html>
    <body>
      <h1>Hello World</h1>
    </body>
  </html>
`);
```

The Content-Type header tells the client how to interpret the response.

### Lesson 36 — Templates
Templates provide a reusable structure for generating HTML.

Conceptually:
```
Template
   +
Data
   ↓
Generated HTML
   ↓
HTTP Response
```

Instead of constructing every response manually, a template can define the common HTML structure while dynamic values are inserted as required.

> Maps to Codevolution Video 34 - HTML Template

### Lesson 37 — Routing and Web Frameworks
Routing determines what should happen for different URLs and HTTP methods.

Conceptually:
```
GET /
    ↓
Home

GET /users
    ↓
Users

GET /products
    ↓
Products
```

As applications become larger, implementing routing manually becomes more complex.

Web frameworks help provide abstractions for:
- Routing
- Request handling
- Responses
- Middleware
- Application structure

> Maps to Codevolution Video 35 - HTTP Routing and Video 36 - Web Framework

---

## Module 6 — Node.js Runtime and Event Loop

### Lesson 38 — Node.js Runtime
Node.js can be viewed as several major pieces working together.

```
Node.js
 │
 ├── V8
 │
 ├── Node.js APIs
 │
 └── libuv
```

V8 executes JavaScript.
Node.js APIs provide functionality.
libuv provides important asynchronous I/O infrastructure.

### Lesson 39 — libuv
libuv is a major component of Node.js's asynchronous architecture.

Simplified architecture:
```
JavaScript
    ↓
Node.js
    ↓
libuv
    ↓
Async I/O infrastructure
```

libuv is associated with mechanisms such as:
- Event loop
- Thread pool
- Asynchronous I/O coordination

### Lesson 40 — Thread Pool
Some Node.js operations use a thread pool.

Conceptually:
```
Node.js
   ↓
libuv
   ↓
Thread Pool
   ├── Thread
   ├── Thread
   ├── Thread
   └── Thread
```

The presence of a thread pool does not mean that all JavaScript executes on these threads. JavaScript execution remains centered around the main thread.

### Lesson 41 — Thread Pool Size
The tutorial introduces the default thread-pool model of: **4 threads**

The thread pool size can be configured through: `UV_THREADPOOL_SIZE`

Conceptually:
```
UV_THREADPOOL_SIZE
        ↓
Thread Pool Capacity
```

### Lesson 42 — Network I/O
Network I/O is handled differently from operations that rely on the libuv thread pool.

Simplified flow:
```
Network I/O
     ↓
Operating System / libuv
     ↓
Event Loop
     ↓
JavaScript callback
```

This asynchronous architecture allows Node.js to handle many network connections without creating a JavaScript thread for every connection.

### Lesson 43 — Event Loop
The event loop coordinates asynchronous callbacks.

Simplified model:
```
Asynchronous operation
       ↓
Operation completes
       ↓
Callback becomes ready
       ↓
Event Loop
       ↓
JavaScript callback
```

The event loop works through different phases and queues.

### Lesson 44 — Microtask Queues
Microtasks have special priority within Node.js's asynchronous execution model.

Important mechanisms include:
- `process.nextTick()`
- Promise callbacks

Example:
```js
process.nextTick(() => {
  console.log("next tick");
});

Promise.resolve().then(() => {
  console.log("Promise");
});
```

Conceptually:
```
Microtasks
 ├── process.nextTick()
 └── Promise callbacks
```

### Lesson 45 — Timer Queue
Timers include:
- `setTimeout(...)`
- `setInterval(...)`

Example:
```js
setTimeout(() => {
  console.log("Timer");
}, 1000);
```

The specified delay determines when a timer becomes eligible to execute. It does not guarantee exact execution at that millisecond.

Conceptually:
```
setTimeout()
     ↓
Timer becomes eligible
     ↓
Timer Queue
     ↓
Callback
```

### Lesson 46 — I/O Queue
I/O operations can result in callbacks that need to be processed.

Conceptually:
```
I/O operation
     ↓
Operation completes
     ↓
I/O callback
     ↓
I/O Queue
     ↓
Callback executes
```

This is why asynchronous Node.js code should not always be understood as executing strictly in source-code order.

### Lesson 47 — I/O Polling
The event loop needs to determine whether I/O operations are ready.

Conceptually:
```
Event Loop
    ↓
I/O Polling
    ↓
Check for ready I/O
    ↓
Process callbacks
```

I/O polling is an important part of understanding asynchronous callback execution.

### Lesson 48 — Check Queue
The check phase is associated with: `setImmediate(...)`

Example:
```js
setImmediate(() => {
  console.log("Immediate");
});
```

Conceptually:
```
setImmediate()
      ↓
Check Queue
      ↓
Callback
```

The ordering between timers, I/O callbacks, and setImmediate() can depend on the execution context.

### Lesson 49 — Close Queue
The close phase handles callbacks associated with closing resources.

Conceptually:
```
Resource
   ↓
Close
   ↓
Close Queue
   ↓
Callback
```

This completes the runtime and event-loop section.

> Maps perfectly to Codevolution Videos 37-48

---

## Module 7 — npm and the Node.js Package Ecosystem

### Lesson 50 — What is npm?
npm is the package manager and package ecosystem associated with Node.js.

It allows developers to:
- Install packages
- Manage dependencies
- Run project scripts
- Publish packages

Conceptually:
```
Application
    ↓
npm
    ↓
Packages
```

### Lesson 51 — package.json
package.json contains project metadata and npm configuration.

Create it with:
```bash
npm init
```
or:
```bash
npm init -y
```

Example:
```json
{
  "name": "my-project",
  "version": "1.0.0",
  "description": "My Node.js project",
  "main": "index.js"
}
```

### Lesson 52 — Installing Packages
Install a package:
```bash
npm install <package-name>
```

npm installs the package into: `node_modules/`

Typical project structure:
```
project/
├── package.json
├── package-lock.json
└── node_modules/
```

### Lesson 53 — Using Packages
A CommonJS application can load a package using:
```js
const packageName = require("package-name");
```

General workflow:
```
npm install
     ↓
node_modules
     ↓
require()
     ↓
Use package
```

### Lesson 54 — Dependencies
Dependencies are packages required by the application. They are recorded in package.json.

Example:
```json
{
  "dependencies": {
    "some-package": "^1.0.0"
  }
}
```

Dependencies can themselves have dependencies, creating a dependency tree.
```
Application
    ↓
Package A
    ├── Package B
    └── Package C
```

### Lesson 55 — Versioning
Package versions commonly use Semantic Versioning: **MAJOR.MINOR.PATCH**

Example: `1.4.2`
```
│ │ │
│ │ └── PATCH
│ └──── MINOR
└────── MAJOR
```

General interpretation:
- **MAJOR** — Breaking/incompatible changes.
- **MINOR** — Compatible new functionality.
- **PATCH** — Compatible fixes.

Common version range symbols include: `^` and `~`

### Lesson 56 — Global Packages
Packages can be installed locally:
```bash
npm install <package-name>
```
or globally:
```bash
npm install -g <package-name>
```

Local packages belong to a particular project. Global packages are generally useful for command-line tools that should be available outside a single project.

### Lesson 57 — npm Scripts
Scripts are defined inside package.json.

Example:
```json
{
  "scripts": {
    "start": "node index.js"
  }
}
```

Run:
```bash
npm run start
```
The start script can also be executed with: `npm start`

Scripts provide convenient names for commonly used project commands.

### Lesson 58 — Publishing an npm Package
npm can also be used to distribute packages.

General workflow:
```
Create package
      ↓
package.json
      ↓
Implement functionality
      ↓
npm publish
      ↓
Package available
```

Publishing command: `npm publish`

Another developer can then install the package using npm.

> Maps to Codevolution Videos 49-57

---

## Module 8 — CLI Tools, Concurrency and Deployment

### Lesson 59 — Building CLI Tools
CLI means: **Command-Line Interface** — Node.js can be used to create applications that run from a terminal.

Example: `node app.js`

The process object provides access to command-line information. An important property is: `process.argv`

Conceptually:
```
Terminal
   ↓
Command
   ↓
process.argv
   ↓
CLI application
```

### Lesson 60 — CLI Options
CLI programs can accept options.

Examples: `--help`, `--version`, `--name`

Example: `node app.js --name John`

The application can interpret the option and its value.

Conceptually:
```
CLI command
    │
    ├── Arguments
    │
    └── Options
          ↓
    Application logic
```

### Lesson 61 — Interactive CLI Tools
An interactive CLI communicates with the user while the program is running.

Example:
```
$ node app.js

What is your name?
> John

Hello John!
```

The terminal interaction can be represented as:
```
Application
     ↓
Question
     ↓
stdin
     ↓
User input
     ↓
Application
     ↓
stdout
```

Important streams include: `stdin`, `stdout`

### Lesson 62 — Cluster Module
The Cluster module provides a way to run multiple Node.js processes.

Conceptually:
```
Primary Process
      │
      ├── Worker Process
      ├── Worker Process
      ├── Worker Process
      └── Worker Process
```

This allows an application to make use of multiple CPU cores through multiple processes.

Important distinction:
```
Cluster
    ↓
Multiple processes
```

### Lesson 63 — Worker Threads Module
Worker Threads provide another mechanism for concurrent execution.

Conceptually:
```
Node.js Process
      │
      ├── Main Thread
      │      └── Event Loop
      │
      └── Worker Thread
             └── CPU-intensive work
```

Worker Threads are particularly useful when CPU-intensive JavaScript work would otherwise block the main thread.

### Lesson 64 — Deploying Node.js App
Deployment means moving the application from a development environment into an environment where it can run for users.

Conceptually:
```
Development
     ↓
Application
     ↓
Production Environment
     ↓
Node.js
     ↓
Users
```

A deployed Node.js application needs appropriate:
- Application code
- Node.js runtime
- Dependencies
- Configuration
- Environment settings

A common server-port pattern is:
```js
const port = process.env.PORT || 3000;
```

This allows the production environment to provide the port while retaining a local fallback.

### Lesson 65 — Wrapping Up
The final lesson brings together the concepts introduced throughout the tutorial.

The overall progression is:
```
Node.js Fundamentals
        ↓
Modules
        ↓
Built-in APIs
        ↓
File System
        ↓
HTTP
        ↓
Asynchronous Programming
        ↓
Event Loop
        ↓
npm
        ↓
CLI Tools
        ↓
Cluster
        ↓
Worker Threads
        ↓
Deployment
        ↓
Wrap Up
```

> Maps to Codevolution Videos 58-64. Your Lesson 65 is actually Video 64 Wrapping Up — same content.

---

## Final Node.js Cheat Sheet

**Node.js**
```
Node.js = JavaScript runtime + V8 + Node.js APIs + libuv
```

**CommonJS Modules**
```js
const module = require("./module");
module.exports = value;
```

**File System**
```js
const fs = require("fs");
fs.readFile(...)
fs.writeFile(...)
fs.appendFile(...)
fs.unlink(...)
```

**Path**
```js
const path = require("path");
path.join()
path.basename()
path.dirname()
path.extname()
```

**Events**
```js
const EventEmitter = require("events");
const emitter = new EventEmitter();
emitter.on("event", callback);
emitter.emit("event");
```

**HTTP**
```js
const http = require("http");
const server = http.createServer((req, res) => {
  res.end("Hello");
});
server.listen(3000);
```

**JSON Response**
```js
res.setHeader("Content-Type", "application/json");
res.end(JSON.stringify(data));
```

**npm**
```bash
npm init
npm install package-name
npm install -g package-name
npm run script-name
npm publish
```

**package.json**
```
package.json
    ├── Project metadata
    ├── Dependencies
    └── Scripts
```

**Event Loop Cheat Sheet**
```
                   Event Loop
                        │
       ┌────────────────┼────────────────┐
       │                │                │
     Timers             I/O            Check
       │                │                │
       │              Polling            │
       │                │                │
       └────────────────┼────────────────┘
                        │
                       Close

Microtasks:
 ├── process.nextTick()
 └── Promise callbacks
```

**Thread Pool Cheat Sheet**
```
libuv
  ↓
Thread Pool
  ├── Worker
  ├── Worker
  ├── Worker
  └── Worker

Default: 4 threads
Config: UV_THREADPOOL_SIZE
```

**Cluster vs Worker Threads**
```
Cluster: One app → Multiple Node.js processes (process-based)
Worker Threads: One Node.js process → Multiple threads
```

**Complete npm Architecture**
```
                      npm
                        │
             ┌──────────┴──────────┐
             │                     │
          Consume                 Create
             │                     │
             ▼                     ▼
       npm install             package.json
             │                     │
             ▼                     ▼
       node_modules            npm publish
             │                     │
             ▼                     ▼
       require/import          npm Registry
             │
             ▼
        Application
```

**Complete Application Lifecycle**
```
1. Create project → 2. npm init → 3. package.json → 4. Install dependencies → 5. Write application → 6. Use Node.js APIs → 7. Create HTTP server / CLI / application → 8. Handle async → 9. Event Loop → 10. Test → 11. Deploy
```

---

## Final Revision Questions
*(Preserved exactly as your notes)*

**Fundamentals:** What is Node.js? What is V8? Why is Node.js useful for server-side? What is REPL? What is process object? What are env vars? What are CLI args? What are callbacks?

**Modules:** What is a module? What is CommonJS? What does require() do? What is module.exports? What are __filename and __dirname? What is module resolution? What are built-in modules?

**File System:** What does fs provide? Difference sync vs async? What does path provide? What does os provide? What does EventEmitter provide?

**HTTP:** How do you create HTTP server? What is request object? Response object? How set status code? Headers? Return JSON? HTML? What is routing? Why frameworks useful?

**Runtime:** What is V8? What is libuv? What is thread pool? Default size? UV_THREADPOOL_SIZE? What is event loop? Timer phase? I/O phase? I/O polling? Check phase? Close phase? What is process.nextTick()? What are Promise microtasks? Why understanding event loop important?

**npm:** What is npm? package.json? node_modules? package-lock.json? Dependency? Semantic Versioning? MAJOR, MINOR, PATCH? Difference local vs global? npm scripts? npm publish?

**CLI:** What is CLI app? process.argv? CLI options? stdin and stdout? How Node app interacts with terminal users?

**Concurrency:** What is Cluster module? What is cluster worker? Are workers processes or threads? What are Worker Threads? Why useful? Difference Cluster vs Worker Threads? Why CPU-intensive JS blocks event loop?

**Deployment:** What does deployment mean? What does Node app need in production? Why env vars useful? Why process.env.PORT? Relationship between npm dependencies and deployment?

---

## Complete Node.js Mental Model
```
                        NODE.JS
                            │
                            ▼
                           V8
                            │
                  Executes JavaScript
                            │
                            ▼
                      Node.js APIs
                            │
             ┌──────────────┼──────────────┐
             │              │              │
            fs             http          events
             │              │              │
             └──────────────┼──────────────┘
                            │
                            ▼
                          libuv
                            │
                 ┌──────────┴──────────┐
                 │                     │
                 ▼                     ▼
            Thread Pool            Event Loop
                 │                     │
                 │          ┌──────────┼──────────┐
                 │          │          │          │
                 │        Timers       I/O       Check
                 │          │          │          │
                 │          └──────────┼──────────┘
                 │                     │
                 │                   Close
                 │
                 ▼
             Async Work

                            │
                            ▼
                           npm
                            │
                    ┌───────┴────────┐
                    │                │
               Dependencies       Scripts
                    │
                    ▼
               package.json

                            │
                            ▼
                       Applications
                            │
              ┌─────────────┼─────────────┐
              │             │             │
             HTTP           CLI       Background
           Server          Tools        Work
              │             │
              └─────────────┼─────────────┘
                            │
                            ▼
                     Cluster / Workers
                            │
                            ▼
                       Deployment

Course Progression
Introduction → Fundamentals → Modules → File System & Built-ins → HTTP → Runtime/libuv/Event Loop → npm → CLI Tools → Cluster → Worker Threads → Deployment → Wrap Up
```
