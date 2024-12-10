# Keyv

The ``@cmmv/keyv`` module integrates with the [Keyv](https://keyv.org/) package to provide a flexible key-value store service. This module allows developers to integrate key-value storage solutions like Redis, Memcached, MongoDB, and more into their applications with minimal configuration. Unlike the ``@cmmv/cache`` module, which is primarily focused on caching routes and gateway responses, ``@cmmv/keyv`` is intended for state management across microservices and multiple applications that share the same store. The module includes native support for namespaces and compression to reduce ``get`` latency and features custom JSON parsing for more efficient data handling in future implementations.

## Installation

To install the ``@cmmv/keyv`` module, use the following command:

```bash 
$ pnpm add @cmmv/keyv keyv @keyv/compress-gzip
$ pnpm add -D @types/keyv
```

If you plan to use Redis, Memcached, or MongoDB as your store, you'll need to install the appropriate driver package for your store:

Supported Key-Value Stores and Drivers

**Redis:** [Github](https://github.com/jaredwray/keyv/tree/main/packages/redis)

```bash
$ pnpm add @keyv/redis
```

**Memcached:** [Github](https://github.com/jaredwray/keyv/tree/main/packages/memcache)

```bash
$ pnpm add @keyv/memcache
```

**MongoDB:** [Github](https://github.com/jaredwray/keyv/tree/main/packages/mongo)

```bash
$ pnpm add @keyv/mongo
```

**PostgreSQL:** [Github](https://github.com/jaredwray/keyv/tree/main/packages/postgres)

```bash
$ pnpm add @keyv/postgres
```

**SQLite:** [Github](https://github.com/jaredwray/keyv/tree/main/packages/sqlite)

```bash
$ pnpm add @keyv/sqlite
```

For a full list of supported stores, visit [Keyv on npm](https://www.npmjs.com/package/keyv).

## Configuration

The ``@cmmv/keyv`` module offers easy configuration through the CMMV system. Below is an example of how to set up a Redis store for use with ``@cmmv/keyv``:

```javascript
module.exports = {
    // Other configurations

    keyv: {
        uri: 'redis://localhost:6379',
        options: {
            namespace: 'cmmv',
            ttl: 600,
            adapter: 'redis',
            ...
        },
    },
}
```

| Option       | Type     | Required | Description                                                                  | Default               |
|--------------|----------|----------|------------------------------------------------------------------------------|-----------------------|
| namespace    | String   | N        | Namespace for the current instance.                                           | 'keyv'                |
| ttl          | Number   | N        | Default TTL. Can be overridden by specifying a TTL on `.set()`.               | undefined             |
| compression  | @keyv/compress- | N  | Compression package to use. See Compression for more details.                | undefined             |
| serialize    | Function | N        | A custom serialization function.                                              | JSONB.stringify       |
| deserialize  | Function | N        | A custom deserialization function.                                            | JSONB.parse           |
| store        | Storage adapter instance | N | The storage adapter instance to be used by Keyv.                              | new Map()             |
| adapter      | String   | N        | Specify an adapter to use, e.g., 'redis' or 'mongodb'.                        | undefined             |

## Keyv Service

The ``@cmmv/keyv`` module includes a service that can be optionally injected into controllers, gateways, or other services for direct access to the key-value store. Unlike the ``@cmmv/cache`` module, ``@cmmv/keyv`` does not provide decorators for automatic caching because its use case focuses more on state management across microservices and distributed applications.

Here’s an example of how you can use the ``KeyvService`` in a controller to interact with the key-value store:

```typescript
import { Controller, Get, Post, Body, Request, Param } from '@cmmv/http';
import { KeyvService } from '@cmmv/keyv';

@Controller('state')
export class StateController {
  constructor(private readonly keyvService: KeyvService) {}

  // GET value by key
  @Get(':key')
  async getValue(@Param('key') key: string, @Request() req): Promise<any> {
    const value = await this.keyvService.get(key);
    return value ? JSON.parse(value) : { message: 'Key not found' };
  }

  // POST (Set value for key)
  @Post()
  async setValue(@Body() data: { key: string; value: any }, @Request() req): Promise<any> {
    const stringifiedValue = JSON.stringify(data.value);
    await this.keyvService.set(data.key, stringifiedValue);
    return { message: 'Value set successfully' };
  }
}
```

### Keyv Service Methods

<br/>

* **get(key: string): Promise<any>:** Retrieves a value by key.
* **set(key: string, value: any, ttl?: number): Promise<void>:** Stores a value by key with an optional TTL.
* **delete(key: string): Promise<boolean>:** Deletes a value by key.
* **clear(): Promise<boolean>:** Deletes all values.

## Custom JSON Parsing

``@cmmv/keyv`` includes support for custom JSON parsing, which will be important for future support of the [fast-json-stringify](https://www.npmjs.com/package/fast-json-stringify) package. This will allow for faster and more efficient serialization and deserialization of objects in the key-value store.

The future integration of ``fast-json-stringify`` will provide significant performance improvements for JSON handling in the key-value store. With this feature, ``@cmmv/keyv`` will support optimized JSON parsing, enabling faster read and write operations, particularly for complex data structures. This will be a key enhancement for applications that require high-performance state management across microservices.

## Usage Across Microservices

The ``@cmmv/keyv`` module is designed to support distributed applications and microservices. By leveraging shared key-value stores like Redis or Memcached, applications can manage shared state across different services, improving consistency and reducing latency. This makes it a great choice for scenarios where multiple applications or services need to interact with the same stateful data.

The ``@cmmv/keyv`` module provides a flexible and powerful key-value storage solution for CMMV-based applications. It is particularly useful for managing application state across microservices, thanks to its support for namespaces, compression, and custom JSON parsing. While it doesn't include automatic caching decorators like ``@cmmv/cache``, it offers an ideal solution for scenarios where shared state is crucial, such as multi-application environments or microservice ecosystems.