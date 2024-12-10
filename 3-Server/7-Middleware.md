# Middleware

The middleware system in ``@cmmv/server`` offers flexibility and powerful integration for processing HTTP requests and responses. Similar to Express, it provides hooks and integration points that allow developers to modify the behavior of requests as they flow through the system. However, the middleware in ``@cmmv/server`` also introduces new behaviors and optimizations, particularly through its use of hooks.

This system allows for the implementation of essential middleware functions such as ETag, Body-Parser, Compression, Cookie-Parser, CORS, Helmet, and Server-Static, each of which follows a common pattern. Some of these middlewares, like ETag, are fully compatible with Express. Others, such as Server-Static, have custom implementations that introduce specific behavior changes, making them incompatible with Express.

The core design of middleware in ``@cmmv/server`` revolves around hooks, which are triggered at various points in the request-response lifecycle. This design allows for more precise and efficient processing of requests.

## Sample

ETag middleware is used to manage ETag headers for caching purposes. ETags are a mechanism for cache validation and are generated based on the content of the response body. If the content hasn't changed between requests, the server can respond with a 304 Not Modified status, improving performance.

Here’s an example of the ETag middleware implemented in ``@cmmv/server``:

```typescript
export class EtagMiddleware {
    public middlewareName: string = 'etag';

    protected options: ETagOptions;

    constructor(options?: ETagOptions) {
        this.options = {
            algorithm: options?.algorithm || 'sha1',
            weak: Boolean(options?.weak === true),
        };
    }

    async process(req, res, next) {
        if (req.app && typeof req.app.addHook == 'function')
            req.app.addHook('onSend', this.onCall.bind(this));
        else this.onCall.call(this, req, res, res.body, next);
    }

    onCall(req, res, payload, done) {
        const hash = this.buildHashFn(this.options.algorithm, this.options.weak);
        let etag = res.getHeader('etag');
        let newPayload;

        if (!etag) {
            if (!(typeof payload === 'string' || payload instanceof Buffer)) {
                done(null, newPayload);
                return;
            }

            etag = hash(payload);
            res.set('etag', etag);
        }

        if (
            req.headers['if-none-match'] === etag ||
            req.headers['if-none-match'] === 'W/' + etag ||
            'W/' + req.headers['if-none-match'] === etag
        ) {
            res.code(304);
            newPayload = '';
        }

        done(null, newPayload);
    }

    buildHashFn(algorithm = 'sha1', weak = false) {
        this.validateAlgorithm(algorithm);

        const prefix = weak ? 'W/"' : '"';

        if (algorithm === 'fnv1a')
            return payload => prefix + fnv1a(payload).toString(36) + '"';

        return payload =>
            prefix +
            createHash(algorithm).update(payload).digest('base64') +
            '"';
    }

    validateAlgorithm(algorithm) {
        if (algorithm === 'fnv1a') return true;

        try {
            createHash(algorithm);
        } catch (e) {
            throw new TypeError(`Algorithm ${algorithm} not supported.`);
        }
    }
}

export default async function (options?: ETagOptions) {
    const middleware = new EtagMiddleware(options);
    return (req, res, next) => middleware.process(req, res, next);
}

export const etag = function (options?: ETagOptions) {
    const middleware = new EtagMiddleware(options);
    return (req, res, next) => middleware.process(req, res, next);
};
```

The middleware system in ``@cmmv/server`` is designed to offer both compatibility with Express and enhancements that provide better control over the request-response cycle. By leveraging hooks and selective execution, the system ensures that middleware only runs when necessary, leading to better performance.

While most middleware is compatible with Express, some—like Server-Static—introduce specific behavior changes. This system is designed to be flexible and modular, allowing you to implement custom middleware easily, integrate existing middleware, and optimize your application's performance.

Additionally, all middlewares implemented for ``@cmmv/server`` must return a promise, ensuring they fit into the asynchronous lifecycle of the framework. When possible, compatible versions of middleware are provided for use in Express applications.

## Compression

The ``@cmmv/compression`` middleware provides HTTP response compression for your server, supporting gzip, deflate, and brotli encodings. Compression reduces the size of the response body, which improves the speed and efficiency of content delivery to the client. It can be used in both ``@cmmv/server`` and Express environments with some variations in implementation.

This middleware integrates seamlessly with the request lifecycle, applying compression to responses based on content type, size, and the client’s Accept-Encoding header. The middleware automatically handles large payloads and dynamically adjusts encoding for different content types.

**Installation**

To install the ``@cmmv/compression`` middleware, run the following command:

```bash
$ pnpm add @cmmv/compression
```

**Usage**

When using ``@cmmv/server``, the compression middleware must be added as part of the middleware chain. Here’s an example of how to use it:

```typescript
import compression from '@cmmv/compression';

app.use(compression({ 
    threshold: 1024, 
    cacheEnabled: true 
}));
```

The ``@cmmv/compression`` middleware comes with a range of options to control how and when compression is applied. Below is a list of available options:

| Option        | Type      | Default                                     | Description                                                                                                                                                       |
|---------------|-----------|---------------------------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `threshold`   | `number` or `string` | `1024`                                       | Specifies the minimum response size (in bytes) required for compression to be applied. Smaller responses are not compressed. Accepts human-readable string values such as '1kb'. |
| `cacheEnabled`| `boolean` | `false`                                     | Enables caching of compressed responses to improve performance on repeated requests. Responses will be stored in memory for the specified duration.                 |
| `cacheTimeout`| `number`  | `60000 (1 min)`                             | Sets the timeout (in milliseconds) after which cached compressed responses are purged from the memory cache. Only applicable if `cacheEnabled` is `true`.          |
| `algorithm`   | `string`  | `'sha1'`                                    | Specifies the hashing algorithm used to generate an ETag for caching validation. Supported values include `sha1`, `md5`, and `fnv1a`.                              |
| `weak`        | `boolean` | `false`                                     | Indicates whether the ETag should be marked as "weak", meaning it provides less strict validation for cache revalidation.                                           |
| `level`       | `number`  | `zlib.constants.Z_DEFAULT_COMPRESSION`       | Specifies the compression level for zlib-based algorithms. Accepts a value between `0` (no compression) and `9` (maximum compression).                             |
| `memLevel`    | `number`  | `8`                                         | Controls the memory usage for compression (higher values use more memory but provide better compression). Accepts values between `1` and `9`.                      |
| `strategy`    | `number`  | `zlib.constants.Z_DEFAULT_STRATEGY`          | Specifies the compression strategy to be used by zlib algorithms. Common strategies include `Z_FILTERED`, `Z_HUFFMAN_ONLY`, and `Z_RLE`.                           |
| `filter`      | `function`| `shouldCompress`                            | A function to determine whether or not a response should be compressed based on its content type or other factors.                                                 |
| `flush`       | `boolean` | `false`                                     | Enables the manual flushing of compression buffers. When enabled, the server can force a flush when necessary (for example, when streaming responses).              |
| `chunkSize`   | `number`  | `16384`                                     | Defines the chunk size used for compression streams. Smaller chunk sizes result in more frequent writes but can impact performance.                                 |
| `windowBits`  | `number`  | `15`                                        | Specifies the size of the compression window (in bits). A larger window size provides better compression but uses more memory.                                     |

## Cookie-Parser

The ``@cmmv/cookie-parser`` middleware is designed to parse cookies in HTTP requests, both signed and unsigned, and make them available in the ``req.cookies`` object. It supports both ``CMMV`` and Express, providing seamless integration with either framework.

**Installation**

```bash
$ pnpm add @cmmv/cookie-parser
```

**Usage**

In CMMV, the middleware is used by registering it with the app using hooks. It will automatically parse cookies on incoming requests and make them available in the ``req.cookies`` and ``req.signedCookies`` objects.

```typescript
import cmmv from '@cmmv/server';
import cookieParser from '@cmmv/cookie-parser';

const app = cmmv();

app.use(cookieParser({ secret: 'mySecretKey' }));

app.get('/cookies', (req, res) => {
    res.send({
        cookies: req.cookies,
        signedCookies: req.signedCookies,
    });
});

app.listen({ port: 3000 });
```

| Option  | Type           | Default | Description                                                                                              |
|---------|----------------|---------|----------------------------------------------------------------------------------------------------------|
| `name`  | `string`       |         | The name of the cookie to parse.                                                                         |
| `secret`| `string`/`string[]` | `[]`     | A string or array of strings used to sign and verify cookies. This ensures the integrity of signed cookies.|
| `decode`| `function`     |         | A custom decoder function for parsing cookies. If not provided, the default `decodeURIComponent` is used. |
| `path`  | `string`       | `'/'`   | Defines the URL path that must exist for the cookie to be included in requests.                           |

**Example for Signed Cookies**

```typescript
app.use(cookieParser({ secret: 'mySecretKey' }));

app.get('/set-cookie', (req, res) => {
    res.cookie('name', 'value', { signed: true });
    res.send('Cookie set');
});

app.get('/get-cookie', (req, res) => {
    res.json({ signedCookies: req.signedCookies });
});
```

## Cors

The CORS (Cross-Origin Resource Sharing) middleware allows you to enable cross-origin resource sharing for your applications by setting appropriate HTTP headers. It is based on the Express CORS middleware but has been adapted to support asynchronous hooks for the CMMV server environment, making it fully compatible with @cmmv/server while maintaining compatibility with Express wherever possible.

**Installation**

```bash
$ pnpm add @cmmv/cors
```

**Usage**

```typescript
import cmmv from '@cmmv/server';
import cors from '@cmmv/cors';

const app = cmmv();
app.use(cors());
app.listen({ port: 3000 });
```

| Option               | Type                 | Default                                          | Description                                                                                 |
|----------------------|----------------------|--------------------------------------------------|---------------------------------------------------------------------------------------------|
| `origin`             | `string` or `function`| `*`                                              | Specifies the origin that is allowed to access the resource. Can be a string, array, or function. |
| `methods`            | `string` or `string[]`| `'GET,HEAD,PUT,PATCH,POST,DELETE'`                | Specifies the HTTP methods allowed for cross-origin requests.                               |
| `preflightContinue`   | `boolean`            | `false`                                          | If `true`, the middleware will not short-circuit preflight requests and will pass them to the next handler. |
| `optionsSuccessStatus`| `number`             | `204`                                            | The status code sent for successful OPTIONS requests (preflight). Some legacy browsers use 204 for success. |
| `credentials`         | `boolean`            | `false`                                          | If `true`, the Access-Control-Allow-Credentials header will be set to `true`.                |
| `maxAge`              | `number`             | `0`                                              | Specifies the time in seconds that browsers are allowed to cache preflight responses.        |
| `headers`             | `string` or `string[]`| `undefined`                                      | Specifies which headers are allowed to be sent in a request.                                |
| `allowedHeaders`      | `string` or `string[]`| `undefined`                                      | Specifies the headers that can be used in the actual request. Defaults to the request's `Access-Control-Request-Headers`. |
| `exposedHeaders`      | `string` or `string[]`| `undefined`                                      | Specifies which headers are exposed to the browser. Defaults to none.                       |

## Etag

The CMMV ETag middleware automatically adds an ``ETag`` header to HTTP responses, helping with cache validation by generating a unique hash for each response payload. This middleware is based on the Fastify ETag implementation but adapted to work within the CMMV framework. The ETag header allows browsers and other clients to determine whether the content has changed since the last request, reducing bandwidth usage and improving performance.

This middleware is designed to support both CMMV and Express environments. For CMMV, the default export returns a promise, and for Express, a separate etag function is available.

**Installation**

```bash
$ pnpm add @cmmv/etag
```

**Usage**

```typescript
import cmmv from '@cmmv/server';
import etag from '@cmmv/etag';

const app = cmmv();
app.use(etag());
app.listen({ port: 3000 });
```

The ``ETagOptions`` interface provides several configuration options to customize the behavior of the ETag middleware.

| Option    | Type    | Default | Description                                                                 |
|-----------|---------|---------|-----------------------------------------------------------------------------|
| algorithm | string  | 'sha1'  | Specifies the hashing algorithm to use for generating the ETag. Supported values include 'sha1', 'md5', and 'fnv1a'. |
| weak      | boolean | false   | If true, the middleware generates weak ETags (prefixed with W/). Weak ETags allow for more lenient cache validation. |

## Helmet

The ``@cmmv/helmet`` middleware is designed to enhance the security of web applications by setting various HTTP headers. It provides support for content security policies, security headers like ``X-Frame-Options``, ``Strict-Transport-Security``, and more. This middleware is built with flexibility in mind, allowing you to customize its behavior based on your application's requirements. It follows a similar implementation pattern to the popular ``helmet`` library and includes native integration with CMMV while maintaining compatibility with Express when possible.

This middleware automatically configures HTTP security headers for better protection against common vulnerabilities such as cross-site scripting (XSS) attacks, clickjacking, and data sniffing.

**Installation**

```bash
$ pnpm add @cmmv/helmet
```

**Usage**

```typescript
import cmmv from '@cmmv/server';
import helmet from '@cmmv/helmet';

const app = cmmv();
app.use(helmet({
    contentSecurityPolicy: {
        directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "https://trustedscripts.com"],
        },
    },
    hsts: {
        maxAge: 31536000, // Force HTTPS for one year
    },
    frameguard: {
        action: 'deny', // Disallow iframes entirely
    }
}));
app.listen({ port: 3000 });
```

The ``HelmetMiddleware`` allows for configuring various HTTP headers and security policies. Below is the list of available options:

| Option                         | Type                | Default                      | Description                                                                                                  |
|---------------------------------|---------------------|------------------------------|--------------------------------------------------------------------------------------------------------------|
| `contentSecurityPolicy`         | object or boolean   | Enabled with default policies | Configures the Content Security Policy (CSP) header. Can be disabled by setting it to `false` or customized by providing an object. |
| `frameguard`                    | object or boolean   | `SAMEORIGIN`                 | Sets the `X-Frame-Options` header to prevent clickjacking.                                                    |
| `dnsPrefetchControl`            | boolean             | `true`                       | Controls the `X-DNS-Prefetch-Control` header to improve privacy.                                              |
| `expectCt`                      | object or boolean   | `false`                      | Adds the `Expect-CT` header to enforce Certificate Transparency requirements.                                 |
| `hsts`                          | object or boolean   | `true`                       | Sets the `Strict-Transport-Security` header to force HTTPS connections for a specified time.                  |
| `ieNoOpen`                      | boolean             | `true`                       | Sets the `X-Download-Options` header to prevent file downloads from opening automatically in Internet Explorer. |
| `noSniff`                       | boolean             | `true`                       | Sets the `X-Content-Type-Options` header to prevent browsers from MIME-sniffing the response.                 |
| `xssFilter`                     | boolean             | `true`                       | Sets the `X-XSS-Protection` header to enable the XSS filter built into most modern web browsers.              |
| `referrerPolicy`                | string or object    | `no-referrer`                | Configures the `Referrer-Policy` header to control the information sent in the `Referer` header.              |
| `hidePoweredBy`                 | boolean or object   | `true`                       | Hides the `X-Powered-By` header to avoid leaking information about the server technology.                     |
| `permittedCrossDomainPolicies`  | object or boolean   | `false`                      | Configures the `X-Permitted-Cross-Domain-Policies` header, often used by Flash/Adobe products.                |

## Server-Static

The `@cmmv/server-static` middleware provides functionality to serve static files from a directory. It is inspired by `serve-static` from Express and Fastify's `fastify-static`. This middleware can be configured to serve files from one or more directories and provides caching, file compression, and other useful options for serving static content.

You can use the ``serveStatic`` function to serve static files from a directory:

```typescript
import cmmv, { serverStatic } from '@cmmv/server';

const app = cmmv();
const host = '0.0.0.0';
const port = 3000;

app.use(serverStatic('public'));

app.set('view engine', 'pug');

app.get('/view', function (req, res) {
    res.render('index', { title: 'Hey', message: 'Hello there!' });
});

app.listen({ host, port })
.then(server => {
    console.log(
        `Listen on http://${server.address().address}:${server.address().port}`,
    );
})
.catch(err => {
    throw Error(err.message);
});
```

## All Middlwares

This example sets up a server using ``@cmmv/server`` with various middlewares to efficiently handle HTTP requests. It serves static files from the ``public`` folder, enables CORS for cross-origin resource sharing, and adds ETag headers for caching using the ``fnv1a`` algorithm. The cookie parser middleware is used to parse cookies and populate ``req.cookies``, while JSON and URL-encoded parsers process request body data. Responses are compressed using GZIP via the compression middleware, and Helmet adds security headers, including a custom Content Security Policy. The server defines routes to handle basic GET and POST requests, render views, send JSON responses, and handle dynamic routes. It listens on ``0.0.0.0:3000``, with an option to enable HTTP/2 and HTTPS if certificates are provided.

```typescript
//import { readFileSync } from "node:fs";

import cmmv, { json, urlencoded, serverStatic } from '@cmmv/server';
import etag from '@cmmv/etag';
import cors from '@cmmv/cors';
import cookieParser from '@cmmv/cookie-parser';
import compression from '@cmmv/compression';
import helmet from '@cmmv/helmet';

const app = cmmv({
    /*http2: true,
    https: {
        key: readFileSync("./cert/private-key.pem"),
        cert: readFileSync("./cert/certificate.pem"),
        passphrase: "1234"
    }*/
});

const host = '0.0.0.0';
const port = 3000;

app.use(serverStatic('public'));
app.use(cors());
app.use(etag({ algorithm: 'fnv1a' }));
app.use(cookieParser());
app.use(json({ limit: '50mb' }));
app.use(urlencoded({ limit: '50mb', extended: true }));
app.use(compression({ level: 6 }));
app.use(
    helmet({
        contentSecurityPolicy: {
            useDefaults: false,
            directives: {
                defaultSrc: ["'self'"],
                scriptSrc: ["'self'", 'example.com'],
                objectSrc: ["'none'"],
                upgradeInsecureRequests: [],
            },
        },
    }),
);

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

app.listen({ host, port })
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