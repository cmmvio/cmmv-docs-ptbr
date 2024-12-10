# Overview

The CMMV framework introduces its own default server implementation, ``@cmmv/server``, which offers superior performance and seamless integration with the overall CMMV ecosystem. This server is highly optimized and designed to provide built-in support for critical features such as compression, routing, request handling, static file serving, security, and middleware management. Because ``@cmmv/server`` is developed as a core part of CMMV, it allows for better control over feature enhancements, bug fixes, and performance improvements, making it the recommended option for most applications.

The server is flexible and shares many of the same APIs and capabilities as [Express](https://expressjs.com/) and [Fastify](https://fastify.dev/), which ensures an easy transition if you're familiar with those frameworks. However, ``@cmmv/server`` also includes enhanced integration with CMMV’s contracts, modules, and services, providing a more consistent developer experience across different layers of the application.

Key Features:
* **HTTP and HTTPS Support:** The adapter can initialize servers using both HTTP and HTTPS based on configuration.
* **Middleware Management:** Preconfigured middlewares such as compression, CORS, Helmet (security), and session handling are included.
* **Static File Serving:** Automatically serves static files from the /public directory.
* **View Engine:** Supports rendering HTML views using CMMVRenderer, a custom template engine with security options like CSP.
* **Controller Registration:** Automatically registers controllers by scanning the ControllerRegistry and mapping HTTP methods (GET, POST, PUT, DELETE) to paths.
* **Session and Security Headers:** Adds session management using express-session and security headers like Content Security Policy, XSS Protection, and HSTS.
* **Request Tracking:** Each request is assigned a unique requestId for telemetry and monitoring.
* **Open Connection Tracking:** Tracks and closes open connections when the server shuts down.
* **Error Handling:** Captures and logs errors during request processing, providing detailed error messages.

## Default Server

```typescript
import { Application } from '@cmmv/core';
import { DefaultAdapter, DefaultHTTPModule } from '@cmmv/http';

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [DefaultHTTPModule, ...],
    services: [...],
    contracts: [...],
});
```

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
| cmmv                     | 0.6.2    | ✓      | 79041.6    | 12.16        | 14.17         |
| koa                      | 2.15.3   | ✗      | 76639.6    | 12.54        | 13.67         |
| express                  | 5.0.1    | ✓      | 21549.2    | 45.89        | 3.84          |
| express-with-middlewares | 5.0.1    | ✓      | 18930.4    | 52.30        | 7.04          |

## Express

In addition to the default server, CMMV also supports Express and Fastify as alternative HTTP adapters, providing flexibility for developers who prefer or need to use these popular frameworks. Both Express and Fastify adapters are fully integrated into the CMMV ecosystem and can be used by simply switching the adapter in the application configuration.

```bash
$ pnpm add @cmmv/express express body-parser cors express-session helmet uuid
```

### Integration

The ``@cmmv/express`` module provides an alternative HTTP adapter based on Express, allowing you to use Express middleware and features seamlessly with your CMMV application.

```typescript
import { Application } from '@cmmv/core';
import { ExpressAdapter, ExpressModule } from '@cmmv/express';

Application.create({
    httpAdapter: ExpressAdapter,
    modules: [ExpressModule, ...],
    services: [...],
    contracts: [...],
});
```

The adapter registers all controllers automatically from the ControllerRegistry. It matches the controller’s routes to the corresponding HTTP methods (GET, POST, etc.), and processes middleware defined at the controller level.

## Fastify

The Fastify Adapter in CMMV provides an alternative to the Express Adapter, allowing for lightweight, high-performance HTTP handling using Fastify's framework. This adapter integrates key middleware like compression, CORS, helmet for security, and static file serving. It automatically registers controllers and manages the lifecycle of incoming requests. The Fastify Adapter follows the same structure as the Express Adapter, supporting session management and content rendering, while offering a faster, more optimized environment.

```bash
$ pnpm add @cmmv/fastify @fastify/compress @fastify/cors @fastify/helmet @fastify/secure-session @fastify/static @fastify/view
```

### Integration

```typescript
import { Application } from '@cmmv/core';
import { FastifyAdapter, FastifyModule } from '@cmmv/fastify';

Application.create({
    httpAdapter: FastifyAdapter,
    modules: [FastifyModule, ...],
    services: [...],
    contracts: [...],
});
```

## Middlewares

The httpMiddlewares configuration allows you to inject custom middleware into the HTTP adapter (such as Express or Fastify) during the application's initialization. This provides additional flexibility by letting you apply any middleware functions to handle tasks like logging, request validation, or security checks.

To use this feature, define an array of middleware functions in the httpMiddlewares property when creating the application.

Here’s an example where we add the morgan logging middleware to an Express-based CMMV application:

```typescript
import { Application } from "@cmmv/core";
import { ExpressAdapter, ExpressModule } from "@cmmv/http";
import { ViewModule } from "@cmmv/view";
import morgan from "morgan";

Application.create({
    httpAdapter: ExpressAdapter,
    httpMiddlewares: [
        morgan('dev'), 
    ],
    modules: [ 
        ExpressModule,
        ViewModule
    ]
});
```

In this example, the morgan middleware is used to log incoming HTTP requests in the 'dev' format. This middleware is passed into the httpMiddlewares configuration, which applies it automatically during the application's initialization.

* The httpMiddlewares array can include any number of middlewares, and each will be applied in the order they are provided.
* Make sure to import any custom middleware before passing it into the configuration.

This configuration helps expand the capabilities of your application by allowing you to use any custom or third-party middleware that your chosen HTTP adapter (like Express or Fastify) supports.