# Server

The ``@cmmv/server`` is a minimalistic server written in TypeScript, designed to maintain the same structure and methods as [Express](https://expressjs.com/), while addressing performance issues and introducing new features for efficient delivery of components and static files.

This project incorporates code from [Express](https://github.com/expressjs/express), [Koa](https://github.com/koajs/koa), and [Fastify](https://github.com/fastify/fastify), but has been completely rewritten in TypeScript with a focus on modernity and performance. Additionally, it integrates features from Vite to optimize the delivery of components and assets, ensuring a faster and more agile experience for modern applications.

Due to the complexity of the project, it has been separated into another repository [cmmv-server monorepo](https://github.com/andrehrferreira/cmmv-server), which contains multiple packages. In addition to the core server, several modules have been implemented, including:

* body-parser
* compression
* cookie-parser
* cors
* etag
* helmet
* server-static

Below, we will discuss each module in more detail.

Currently, the project is in a testing phase and therefore not recommended for production use.

## Features

* Fully rewritten in **TypeScript**.
* **Dynamic property definitions** (like ``Object.defineProperty``) were removed to avoid serious performance issues.
* **Fastify-inspired hook system** for flexible request lifecycle management.
* Support for **HTTP/2**.
* Compression support for **Brotli**, **gzip**, and **deflate**.
* **ETag** implementation using **FNV-1a**.
* Built-in support for cryptographic algorithms such as **MD5**, **SHA-1**, and others from the ``crypto`` module.
* Superior performance compared to **Koa**, **Hapi**, and **Express**, and ongoing optimization to achieve performance similar to **Fastify**.
* Full compatibility with all methods provided by **Express.js**.

## Benchmarks

* [https://github.com/fastify/benchmarks](https://github.com/fastify/benchmarks)
* Machine: linux x64 | 32 vCPUs | 128.0GB Mem
* Node: v20.17.0
* Run: Thu Nov 26 2024 15:23:41 GMT+0000 (Coordinated Universal Time)
* Method: ``autocannon -c 100 -d 40 -p 10 localhost:3000``

| Framework                | Version  | Router | Requests/s | Latency (ms) | Throughput/Mb |
|--------------------------|----------|--------|------------|--------------|---------------|
| bare                     | v20.17.0 | ✗      | 88267.6    | 10.87        | 15.74         |
| fastify                  | 5.1.0    | ✓      | 87846.6    | 10.91        | 15.75         |
| polka                    | 0.5.2    | ✓      | 87234.8    | 10.99        | 15.56         |
| connect                  | 3.7.0    | ✗      | 86129.8    | 11.13        | 15.36         |
| connect-router           | 1.3.8    | ✓      | 85804.8    | 11.16        | 15.30         |
| server-base              | 7.1.32   | ✗      | 85724.8    | 11.18        | 15.29         |
| rayo                     | 1.4.6    | ✓      | 85504.6    | 11.21        | 15.25         |
| server-base-router       | 7.1.32   | ✓      | 84189.0    | 11.39        | 15.01         |
| micro                    | 10.0.1   | ✗      | 81955.2    | 11.70        | 14.62         |
| micro-route              | 2.5.0    | ✓      | 81153.6    | 11.82        | 14.47         |
| cmmv                     | 0.6.2    | ✓      | 79041.6    | 12.16        | 14.17         |
| koa                      | 2.15.3   | ✗      | 76639.6    | 12.54        | 13.67         |
| polkadot                 | 1.0.0    | ✗      | 72702.4    | 13.25        | 12.96         |
| koa-isomorphic-router    | 1.0.1    | ✓      | 72588.4    | 13.28        | 12.95         |
| hono                     | 4.6.12   | ✓      | 72410.8    | 13.31        | 12.91         |
| take-five                | 2.0.0    | ✓      | 71261.2    | 13.54        | 25.62         |
| 0http                    | 3.5.3    | ✓      | 71047.6    | 13.58        | 12.67         |
| restana                  | 4.9.9    | ✓      | 68919.6    | 14.01        | 12.29         |
| koa-router               | 12.0.1   | ✓      | 67593.6    | 14.31        | 12.05         |
| h3-router                | 1.13.0   | ✓      | 66985.2    | 14.44        | 11.95         |
| microrouter              | 3.1.3    | ✓      | 62076.0    | 15.61        | 11.07         |
| h3                       | 1.13.0   | ✗      | 60265.6    | 16.10        | 10.75         |
| hapi                     | 21.3.12  | ✓      | 58199.2    | 16.68        | 10.38         |
| restify                  | 11.1.0   | ✓      | 57493.6    | 16.89        | 10.36         |
| fastify-big-json         | 5.1.0    | ✓      | 21931.2    | 45.09        | 252.32        |
| express                  | 5.0.1    | ✓      | 21549.2    | 45.89        | 3.84          |
| express-with-middlewares | 5.0.1    | ✓      | 18930.4    | 52.30        | 7.04          |
| trpc-router              | 10.45.2  | ✓      | N/A        | N/A          | N/A           |

## Installation

Install the ``@cmmv/server`` package via npm:

```bash
$ pnpm add @cmmv/server
```

## Quick Start

Below is a simple example of how to create a new CMMV application:

```typescript
import cmmv, { json, urlencoded, serverStatic } from '@cmmv/server';

const app = cmmv({
    /*http2: true,
    https: {
        key: readFileSync("./cert/private-key.pem"),
        cert: readFileSync("./cert/certificate.pem"),
        passphrase: "1234"
    }*/
});

app.use(serverStatic('public'));
app.use(json({ limit: '50mb' }));
app.use(urlencoded({ limit: '50mb', extended: true }));

app.get('/', async (req, res) => {
    res.send('Hello World');
});

app.listen({ host: "0.0.0.0", port: 3000 })
.then(server => {
    const addr = server.address();
    console.log(
        `Listen on http://${addr.address}:${addr.port}`,
    );
})
.catch(err => {
    throw Error(err.message);
});
```

## Application

The Application class is central to the ``@cmmv/server`` and provides a flexible foundation for managing HTTP/HTTP2 server instances, routing, middleware, and error handling. It is built on top of Node.js’s ``EventEmitter``, and integrates features such as query parsing, content-type parsers, and HTTP method handlers.

This class is responsible for managing the lifecycle of HTTP requests, routing them to the correct handlers, and applying middlewares and hooks (such as pre-parsing, on-request, and error handling). The class also supports customizable settings and error handling mechanisms. Here’s a breakdown of its features:


***Features:***

* **HTTP/HTTP2 Support:** The ``Application`` class can create either HTTP or HTTP2 servers, supporting both regular and secure requests.
* **Routing:** Offers full routing capabilities for different HTTP methods (``GET``, ``POST``, etc.) via the ``Router`` class.
* **Middleware:** Supports chaining middlewares via the ``use()`` method, including handling arrays of middlewares.
* **Error Handling:** Custom error handling through ``setErrorHandler`` that allows defining specific error-handling logic.
* **View Rendering:** Uses the ``View`` class for rendering templates. It supports custom view engines that can be registered via ``app.engine()``.
* **Hooks:** Manages lifecycle hooks like ``onRequest``, ``preParsing``, ``onError``, and more, which can be used to customize request handling.
* **Configuration Options:** Configurable server settings like timeouts, body limits, and HTTP2 support.

| Option                    | Description                                                                                                                                                           | Type                | Default        |
|----------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------|----------------|
| `http2`                    | Enables HTTP2 support. When `true`, the server instance will use HTTP2 for communication.                                                                              | `boolean`           | `false`        |
| `https`                    | Configuration for HTTPS (including keys and certificates). Required for secure HTTP2.                                                                                  | `object`            | `undefined`    |
| `connectionTimeout`        | The time (in milliseconds) the server will wait before closing idle connections.                                                                                       | `number`            | `0`            |
| `keepAliveTimeout`         | The time (in milliseconds) the server will wait for a keep-alive connection before closing it.                                                                          | `number`            | `72000`        |
| `maxRequestsPerSocket`     | The maximum number of requests allowed per socket.                                                                                                                     | `number`            | `0`            |
| `requestTimeout`           | The time (in milliseconds) the server will wait for the request to complete before timing out.                                                                          | `number`            | `0`            |
| `bodyLimit`                | The maximum size (in bytes) for the request body.                                                                                                                      | `number`            | `1048576`      |
| `maxHeaderSize`            | The maximum size (in bytes) for the request headers.                                                                                                                   | `number`            | `16384`        |
| `insecureHTTPParser`       | Enables the use of an insecure HTTP parser that accepts non-standard HTTP requests.                                                                                     | `boolean`           | `false`        |
| `joinDuplicateHeaders`     | Combines duplicate HTTP headers into a single header if set to `true`.                                                                                                 | `boolean`           | `false`        |
| `querystringParser`        | Custom query string parsing function. Must be a function if provided.                                                                                                  | `function`          | `undefined`    |
| `serverFactory`            | A custom function for creating the server instance, useful for advanced configurations like clustering.                                                                | `function`          | `undefined`    |

This table lists the configuration options available when creating an instance of the ``Application`` class, which can be used to fine-tune its behavior for different environments and use cases.

```typescript
import cmmv from '@cmmv/server';

const app = new cmmv({
    http2: true,
    https: {
        key: fs.readFileSync('./key.pem'),
        cert: fs.readFileSync('./cert.pem')
    },
    requestTimeout: 30000
});

app.listen({ port: 8080, host: '0.0.0.0' });
```

This example creates a secure HTTP2 server using the ``Application`` class with custom request timeouts and listens on port 8080. The flexibility of the ``Application`` class allows for dynamic middleware registration, custom hooks, and full control over server behavior.

## Router

The ``Router`` class is responsible for defining, registering, and handling routes in the ``@cmmv/server``. It uses the [``find-my-way``](https://github.com/delvedor/find-my-way) module to manage HTTP method routing, and it provides an abstraction for setting up routes using various HTTP methods (GET, POST, PUT, DELETE, etc.).

By utilizing ``find-my-way``, the ``Router`` class ensures efficient routing, supporting features such as route param handling, middleware stacking, and dynamic route resolution. This router supports all major HTTP methods and allows flexible route definitions.

Features of the ``Router``:
* **Route Management:** Allows registering routes for all HTTP methods, including custom methods like ``BIND``, ``MKCOL``, and more.
* **Middleware Support:** Supports middleware stacking for each route, allowing the addition of multiple handlers per route.
* **Parameter Handlers:** Provides a mechanism to handle parameters within routes via the ``param()`` method, similar to how Express handles route parameters.
* **Dynamic Path Handling:** Efficiently resolves dynamic paths and can apply multiple middlewares to the same route.
* **Error Handling:** Includes mechanisms to ensure valid route handlers are defined, with error handling for missing or invalid handlers.
* **Compatibility:** While it provides an API similar to Express.js for route definitions, it does not depend on Express and operates independently.

The ``find-my-way`` library is a high-performance HTTP router that matches requests to registered routes. It provides features such as:

* **Dynamic Routing:** Supports dynamic path segments and parameters (e.g., ``/users/:id``).
* **Multiple Methods:** Routes can be registered for any HTTP method (GET, POST, etc.).
* **Case Sensitivity:** Routes are case-insensitive by default but can be configured otherwise.
* **Performance:** Designed for efficiency, it supports fast lookup times for large numbers of routes.
* **Middleware Stack:** Allows attaching an array of middleware functions to a single route.

The Router class provides methods for each HTTP method and allows route definitions. Below is a table summarizing the methods and their functionality:

| Method       | Description                                                           | Example Usage                                |
|--------------|-----------------------------------------------------------------------|----------------------------------------------|
| `acl()`      | Registers a route for the ACL method.                                 | `router.acl('/resources', handler)`          |
| `bind()`     | Registers a route for the BIND method.                                | `router.bind('/binding', handler)`           |
| `checkout()` | Registers a route for the CHECKOUT method.                            | `router.checkout('/checkout', handler)`      |
| `connect()`  | Registers a route for the CONNECT method.                             | `router.connect('/connect', handler)`        |
| `copy()`     | Registers a route for the COPY method.                                | `router.copy('/copy', handler)`              |
| `delete()`   | Registers a route for the DELETE method.                              | `router.delete('/resource/:id', handler)`    |
| `get()`      | Registers a route for the GET method. Also registers HEAD by default.  | `router.get('/users', handler)`              |
| `head()`     | Registers a route for the HEAD method.                                | `router.head('/headers', handler)`           |
| `link()`     | Registers a route for the LINK method.                                | `router.link('/link', handler)`              |
| `lock()`     | Registers a route for the LOCK method.                                | `router.lock('/lock', handler)`              |
| `m-search()` | Registers a route for the M-SEARCH method.                            | `router['m-search']('/search', handler)`     |
| `merge()`    | Registers a route for the MERGE method.                               | `router.merge('/merge', handler)`            |
| `mkactivity()`| Registers a route for the MKACTIVITY method.                         | `router.mkactivity('/activity', handler)`    |
| `mkcalendar()`| Registers a route for the MKCALENDAR method.                         | `router.mkcalendar('/calendar', handler)`    |
| `mkcol()`    | Registers a route for the MKCOL method.                               | `router.mkcol('/col', handler)`              |
| `move()`     | Registers a route for the MOVE method.                                | `router.move('/move', handler)`              |
| `notify()`   | Registers a route for the NOTIFY method.                              | `router.notify('/notify', handler)`          |
| `options()`  | Registers a route for the OPTIONS method.                             | `router.options('/options', handler)`        |
| `patch()`    | Registers a route for the PATCH method.                               | `router.patch('/update', handler)`           |
| `post()`     | Registers a route for the POST method.                                | `router.post('/submit', handler)`            |
| `propfind()` | Registers a route for the PROPFIND method.                            | `router.propfind('/find', handler)`          |
| `proppatch()`| Registers a route for the PROPPATCH method.                           | `router.proppatch('/patch', handler)`        |
| `purge()`    | Registers a route for the PURGE method.                               | `router.purge('/purge', handler)`            |
| `put()`      | Registers a route for the PUT method.                                 | `router.put('/resource', handler)`           |
| `rebind()`   | Registers a route for the REBIND method.                              | `router.rebind('/rebind', handler)`          |
| `report()`   | Registers a route for the REPORT method.                              | `router.report('/report', handler)`          |
| `search()`   | Registers a route for the SEARCH method.                              | `router.search('/search', handler)`          |
| `source()`   | Registers a route for the SOURCE method.                              | `router.source('/source', handler)`          |
| `subscribe()`| Registers a route for the SUBSCRIBE method.                           | `router.subscribe('/subscribe', handler)`    |
| `trace()`    | Registers a route for the TRACE method.                               | `router.trace('/trace', handler)`            |
| `unbind()`   | Registers a route for the UNBIND method.                              | `router.unbind('/unbind', handler)`          |
| `unlink()`   | Registers a route for the UNLINK method.                              | `router.unlink('/unlink', handler)`          |
| `unlock()`   | Registers a route for the UNLOCK method.                              | `router.unlock('/unlock', handler)`          |
| `unsubscribe()`| Registers a route for the UNSUBSCRIBE method.                       | `router.unsubscribe('/unsubscribe', handler)`|

Example 

```typescript
// Initialize router
const router = new Router();

// Register GET route
router.get('/users', (req, res) => {
  res.send('List of users');
});

// Register POST route
router.post('/users', (req, res) => {
  res.send('User created');
});

// Register PUT route
router.put('/users/:id', (req, res) => {
  res.send(`User ${req.params.id} updated`);
});

// Register DELETE route
router.delete('/users/:id', (req, res) => {
  res.send(`User ${req.params.id} deleted`);
});
```

In this example, the ``Router`` class is used to define routes for handling different HTTP methods such as GET, POST, PUT, and DELETE. Each route accepts a request (``req``) and a response (``res``), allowing handlers to manage incoming requests and send appropriate responses.

Exemple in Application 

```typescript
import cmmv from '@cmmv/server';

const app = cmmv();
const host = '0.0.0.0';
const port = 3000;

app.set('view engine', 'pug');

app.get('/view', function (req, res) {
    res.render('index', { title: 'Hey', message: 'Hello there!' });
});

app.get('/', async (req, res) => {
    res.send('Hello World');
});

app.get('/json', async (req, res) => {
    res.json({ hello: 'world' });
});

app.get('/user/:id', async (req, res) => {
    res.send('User ' + req.params.id);
});

app.get('/users', async (req, res) => {
    res.json(req.query);
});

app.post('/test', async (req, res) => {
    console.log(req.body);
    res.send('ok');
});

app.listen({ host, port });
```

## Static

The ``@cmmv/server-static`` middleware is used to serve static files from a specified directory or directories. It is designed to handle requests for static content such as HTML, CSS, JavaScript, images, or any other assets. The middleware offers several configuration options for file serving behavior, including caching, compression, custom headers, and more.

Once the middleware is initialized, it listens for requests matching the specified prefix and serves files from the root directory (or multiple directories) as configured.

| Option          | Description                                                                                                                                                                                   | Type                     | Default         |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------|-----------------|
| `root`          | Specifies the root directory (or directories) from which static files will be served.                                                                                                         | `string` or `string[]`    | `undefined` (Required) |
| `prefix`        | URL prefix for serving static files. Files will be served from `prefix + path` (e.g., `/static/myfile.js`).                                                                                   | `string`                  | `'/'`           |
| `maxAge`        | Sets the Cache-Control max-age directive in milliseconds. Determines how long files should be cached by the client.                                                                            | `number`                  | `0`             |
| `cacheControl`  | Enables or disables the Cache-Control header. When enabled, cache headers will be automatically set based on the `maxAge` option.                                                              | `boolean`                 | `true`          |
| `dotfiles`      | Defines how to handle dotfiles (files starting with `.`). Options are `'allow'`, `'deny'`, or `'ignore'`.                                                                                      | `'allow'`, `'deny'`, `'ignore'` | `'allow'`     |
| `serverDotfiles`| If enabled, the server will serve dotfiles from the root directory.                                                                                                                            | `boolean`                 | `false`         |
| `index`         | Specifies the default file to serve when a directory is requested. Can be a string (e.g., `'index.html'`) or `false` to disable index file serving.                                             | `string` or `boolean`     | `'index.html'`  |
| `fallthrough`   | When `true`, allows requests to fall through to the next middleware if a file is not found. If `false`, a 404 error will be returned.                                                          | `boolean`                 | `true`          |
| `redirect`      | Redirects requests to directories that don't end with a `/` by adding a trailing slash.                                                                                                        | `boolean`                 | `true`          |
| `immutable`     | When set to `true`, serves files with the `Cache-Control: immutable` directive, indicating that the files will never change.                                                                   | `boolean`                 | `false`         |
| `lastModified`  | Enables or disables the Last-Modified header. When enabled, it sets the Last-Modified header based on the file's last modified time.                                                           | `boolean`                 | `true`          |
| `etag`          | Enables or disables the generation of ETag headers, which can be used for cache validation.                                                                                                    | `boolean`                 | `true`          |
| `extensions`    | File extensions to try when a file is not found (e.g., serving `file.html` when the request is for `file`). Can be an array of strings.                                                        | `string[]` or `boolean`   | `false`         |
| `acceptRanges`  | Enables support for HTTP range requests, useful for media streaming.                                                                                                                           | `boolean`                 | `true`          |
| `preCompressed` | If `true`, the middleware will attempt to serve pre-compressed files (e.g., `.br` or `.gz` files) if available.                                                                                | `boolean`                 | `false`         |
| `allowedPath`   | A function that can be used to filter or restrict access to specific paths. The function receives `pathname`, `root`, and `req` as parameters and returns `true` to allow the path or `false` to reject it. | `Function` | `undefined` |
| `setHeaders`    | A custom function to set headers for the response. It receives `res`, `path`, and `stat` as parameters and can modify headers before the response is sent.                                     | `Function`                | `null`          |

This middleware is not compatible with Express because its internal workings differ significantly from Express’s static file server. One major change is the removal of directory listing support. Instead, the middleware verifies existing files and creates routes at the application’s startup, resulting in improved efficiency.

It leverages [``@fastify/send``](https://github.com/fastify/send) and [``@fastify/accept-negotiator``](https://github.com/fastify/accept-negotiator) to deliver files efficiently. Additionally, it supports add-ons like ``@cmmv/etag`` and ``@cmmv/compression`` for enhanced static file delivery, offering cache control through the use of ``lastModified`` and ``ETag`` headers. These features provide a more optimized and performant static file server compared to traditional approaches.

## JSON

The ``@cmmv/body-parser`` middleware in cmmv-server serves as a built-in function designed to handle incoming requests with JSON payloads. Similar to Express, this middleware parses the request body and attaches the parsed data to the ``req.body`` object, making it easily accessible for further processing.

This middleware specifically parses JSON data, and only processes requests where the ``Content-Type: application/json`` or ``application/vnd.api+json`` header matches the specified type option. It supports any Unicode encoding and can automatically inflate compressed requests using gzip or deflate.

After the middleware runs, the parsed JSON data is available in ``req.body``. If there’s no body to parse, the ``Content-Type`` doesn’t match, or an error occurs, req.body will be an empty object (``{}``).

Since ``req.body`` is populated from user-provided input, it is important to note that all properties and values in the object are untrusted. They should be carefully validated before being used. For instance, attempting to access ``req.body.foo.toString()`` could result in errors if ``foo`` is undefined, not a string, or if the ``toString`` method has been overridden or replaced by malicious input.

The following table outlines the options available for configuring this middleware, offering customization over its behavior to suit various application needs.

| Property  | Description                                                                                                                                                                                                                                                                                                                                                                             | Type     | Default          |
|-----------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|------------------|
| `inflate` | Enables or disables handling deflated (compressed) bodies. When disabled, deflated bodies are rejected.                                                                                                                                                                                                                                           | Boolean  | `true`           |
| `limit`   | Controls the maximum request body size. If this is a number, the value specifies the number of bytes; if it is a string, the value is passed to the `bytes` library for parsing.                                                                                                                                                                   | Mixed    | `"100kb"`        |
| `reviver` | The `reviver` option is passed directly to `JSON.parse` as the second argument. It allows custom transformation of parsed JSON data. More details can be found in the [MDN documentation on JSON.parse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse#using_the_reviver_parameter).               | Function | `null`           |
| `strict`  | Enables or disables only accepting arrays and objects as valid JSON. When disabled, the middleware will accept anything that `JSON.parse` accepts.                                                                                                                                                                                                | Boolean  | `true`           |
| `type`    | Determines the media type the middleware will parse. This can be a string, array of strings, or a function. If not a function, the value is passed directly to the `type-is` library and can be an extension (e.g., `json`), a MIME type (e.g., `application/json`), or a wildcard MIME type (e.g., `*/json`). If a function, it is called as `fn(req)` and parses when it returns `true`. | Mixed    | `"application/json"` |
| `verify`  | If provided, this function is called as `verify(req, res, buf, encoding)`, where `buf` is the raw request body. The parsing can be aborted by throwing an error from within this function.                                                                                                                                                        | Function | `undefined`      |

## Raw

This middleware processes incoming request payloads and converts them into a Buffer, leveraging functionality inspired by ``body-parser``. It is designed specifically to handle bodies as Buffers and only processes requests where the ``Content-Type: application/octet-stream`` or ``application/vnd+octets`` header matches the defined type option.

The parser is capable of handling any Unicode encoding and supports automatic decompression for gzip and deflate encoded requests.

After the middleware runs, a Buffer containing the parsed data is assigned to the ``req.body`` property. If no body is found, the ``Content-Type`` doesn't match, or an error occurs, ``req.body`` will either default to an empty object ({}) or remain unchanged if another parser has already processed it.

Since ``req.body`` is populated based on user input, it is crucial to validate any properties and values before using them. For example, calling ``req.body.toString()`` might lead to errors if ``req.body`` has been altered by multiple parsers. It is highly recommended to verify that ``req.body`` is a Buffer before performing any buffer-specific operations.

The following table provides an overview of the optional configuration options:

| Property  | Description                                                                                                                                                                                                                                                                                                              | Type     | Default                    |
|-----------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|----------------------------|
| `inflate` | Enables or disables handling deflated (compressed) bodies. When disabled, deflated bodies are rejected.                                                                                                                                                                                                                  | Boolean  | `true`                     |
| `limit`   | Controls the maximum request body size. If this is a number, it specifies the number of bytes. If it is a string, the value is passed to the `bytes` library for parsing.                                                                                                                                               | Mixed    | `"100kb"`                  |
| `type`    | Determines the media type the middleware will parse. This can be a string, array of strings, or a function. If it’s not a function, it’s passed to the `type-is` library and can be an extension (e.g., `bin`), a MIME type (e.g., `application/octet-stream`), or a wildcard MIME type (e.g., `application/*`).         | Mixed    | `"application/octet-stream"` |
| `verify`  | A function called as `verify(req, res, buf, encoding)` where `buf` is the raw request body and `encoding` is its encoding. Throwing an error from this function aborts the parsing process.                                                                                                                               | Function | `undefined`                |

## Text

This middleware processes incoming request payloads by converting them into a string, utilizing functionality from the ``body-parser`` module.

It provides middleware that parses all bodies as strings, only processing requests where the ``Content-Type: text/plain`` header aligns with the specified type. It handles various Unicode encodings and supports automatic decompression of gzip and deflate encodings.

After the middleware runs, a new string representation of the parsed data is assigned to the ``req.body`` object. If there is no body to parse, the Content-Type does not match, or an error occurs, ``req.body`` will default to an empty object (``{}``).

Since the structure of req.body is based on user input, it is important to treat it as untrusted. Each property and value must be validated before use. For instance, attempting to call ``req.body.trim()`` could lead to errors if req.body is not a string. Therefore, it's advisable to ensure that ``req.body`` is a string before using string methods.

The following table outlines the available configuration options:

| Property        | Description                                                                                                                                                                                                                                                   | Type     | Default       |
|-----------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|---------------|
| `defaultCharset` | Specifies the default character set for the text content when the charset is not included in the `Content-Type` header of the request.                                                                                                                        | String   | `"utf-8"`     |
| `inflate`       | Controls whether or not compressed (deflated) request bodies should be handled. If disabled, compressed bodies will be rejected.                                                                                                                               | Boolean  | `true`        |
| `limit`         | Sets the maximum allowed size for the request body. When provided as a number, the value is treated as the number of bytes. If provided as a string, it is parsed using the `bytes` library.                                                                    | Mixed    | `"100kb"`     |
| `type`          | Defines the media type the middleware should parse. This option can be a string, an array of strings, or a function. If not a function, the value is passed to the `type-is` library and can be an extension (e.g., `txt`), a MIME type (e.g., `text/plain`), or a wildcard pattern (e.g., `text/*`). If a function, it is called as `fn(req)` and the request is parsed if it returns a truthy value. | Mixed    | `"text/plain"` |
| `verify`        | If provided, this function is invoked as `verify(req, res, buf, encoding)`, where `buf` is the raw request body in Buffer form, and `encoding` is the encoding of the request. Parsing can be stopped by throwing an error within this function.                  | Function | `undefined`   |

## Urlencoded

This middleware is designed to parse incoming requests that contain urlencoded payloads. It is based on the functionality provided by body-parser.

The middleware only processes urlencoded bodies, specifically looking for requests where the ``Content-Type: application/x-www-form-urlencoded`` header matches the specified type option. This parser accepts bodies encoded in UTF-8 and supports automatic decompression for requests compressed with gzip or deflate.

After the middleware processes a request, it populates the req.body object with the parsed data. If no body is found, the Content-Type doesn't match, or an error occurs, ``req.body`` will be set to an empty object ({}). The object will contain key-value pairs, where values can either be strings or arrays (when extended: false), or any data type (when extended: true).

Since ``req.body`` is populated based on user input, it is important to validate all properties and values before using them. For example, calling ``req.body.foo.toString()`` might cause errors if foo is not a string, or if it is undefined. Therefore, it is advisable to check the type of ``req.body`` values before performing any operations on them.

The following table outlines the available configuration options for the middleware:

| Property         | Description                                                                                                                                                                                                                                           | Type     | Default                           |
|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|-----------------------------------|
| `extended`       | Allows you to choose between parsing URL-encoded data with the `querystring` library (when `false`) or the `qs` library (when `true`). The "extended" syntax allows for more complex objects and arrays to be encoded into URL-encoded format.         | Boolean  | `true`                            |
| `inflate`        | Enables or disables support for handling compressed (deflated) request bodies. If disabled, compressed bodies will be rejected.                                                                                                                        | Boolean  | `true`                            |
| `limit`          | Sets the maximum allowed size for the request body. If provided as a number, it specifies the number of bytes. If provided as a string, it will be parsed by the `bytes` library.                                                                      | Mixed    | `"100kb"`                         |
| `parameterLimit` | Controls the maximum number of parameters that can be present in the URL-encoded data. If the request exceeds this limit, an error will be thrown.                                                                                                      | Number   | `1000`                            |
| `type`           | Defines which media types the middleware will process. This can be specified as a string, array of strings, or a function. If not a function, this value is passed to the `type-is` library and can be an extension (e.g., `urlencoded`), a MIME type (e.g., `application/x-www-form-urlencoded`), or a wildcard pattern. | Mixed    | `"application/x-www-form-urlencoded"` |
| `verify`         | A function that is invoked as `verify(req, res, buf, encoding)`, where `buf` is the raw request body in Buffer form. Throwing an error from this function will stop the parsing process.                                                                | Function | `undefined`                       |
