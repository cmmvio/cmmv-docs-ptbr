# Elastic

The ``@cmmv/elastic`` module provides a seamless integration with [Elastic](https://www.elastic.co/pt/) for managing indices, handling documents, and executing search queries. This module is designed to be scalable and efficient, integrating seamlessly with the ``@cmmv`` framework to support modern data-driven applications.

## Features

* **Index Management:** Simplifies creating, deleting, checking, and managing Elasticsearch indices.
* **Document Operations:** Perform CRUD operations on Elasticsearch documents.
* **Search and Query Execution:** Execute advanced search queries with support for scoring and filtering.
* **Alias and Rollover Support:** Manage aliases for flexible index routing and perform rollovers for scalable data storage.
* **Integration with CMMV Framework:** Built-in support for @cmmv contracts and models.
* **Error Logging and Handling:** Ensures robust error management and debugging.

## Installation

To install the ``@cmmv/elastic`` module:

```bash
$ pnpm add @cmmv/elastic @elastic/elasticsearch
```

## Configuration

The ``@cmmv/elastic`` module requires an Elasticsearch configuration, which can be defined in the ``.cmmv.config.js`` file:

```javascript
import * as fs from "node:fs";

module.exports = {
    env: process.env.NODE_ENV,

    elastic: {
        node: 'http://localhost:9200', // Elasticsearch node URL
        //cloud: { id: '<cloud-id>' }, // Optional for Elastic Cloud
        /*
        tls: {
            ca: fs.readFileSync('./http_ca.crt'),
            rejectUnauthorized: false
        },
        auth: {
            bearer: process.env.ELASTIC_BEARER || "",
            apiKey: process.env.ELASTIC_APIKEY || "",
            username: process.env.ELASTIC_USERNAME || "",
            password: process.env.ELASTIC_PASSWORD || ""
        }
        */
    }
};
```

## Setting Up the Application

In your ``index.ts``, include the ``ElasticModule`` and its services for seamless integration with the application:

```typescript
import { Application } from "@cmmv/core";
import { DefaultAdapter, DefaultHTTPModule } from "@cmmv/http";
import { ElasticModule, ElasticService } from "@cmmv/elastic";

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [
        DefaultHTTPModule,
        ElasticModule,
    ],
    services: [ElasticService],
});
```

## Usage

### Creating an Index

Allows you to create a new Elasticsearch index with configurable settings like the number of shards and replicas. Use this to structure your data storage.


```typescript
await ElasticService.createIndex('my-index', {
    number_of_shards: 1,
    number_of_replicas: 0,
});
```

### Checking Index Existence

Verifies whether a specific index exists in Elasticsearch. Useful for conditional operations to avoid duplicate index creation.

```typescript
const exists = await ElasticService.checkIndexExists('my-index');
console.log(`Index exists: ${exists}`);
```

### Deleting an Index

Deletes an existing index from Elasticsearch. Use this to remove outdated or unnecessary indices.

```typescript
await ElasticService.deleteIndex('my-index');
```

### Inserting Documents

Adds a new document to a specified index. Ideal for storing and indexing structured data for retrieval and analysis.

```typescript
await ElasticService.insertDocument('my-index', { 
    id: 1, 
    name: 'Test Document' 
});
```

### Updating Documents

Updates an existing document in a specified index. Use this to modify stored data without creating duplicates.

```typescript
await ElasticService.updateDocument('my-index', 'document-id', { 
    name: 'Updated Name' 
});
```

### Searching Documents

Performs a search query on a specified index. Use this to retrieve documents matching specific criteria.

```typescript
const results = await ElasticService.searchDocuments('my-index', {
    query: { match: { name: 'Test Document' } },
});
console.log(results);
```

### Alias Management

Creates or checks the existence of an alias for an index. Aliases are useful for abstracting access to indices, enabling flexible data management.

```typescript
await ElasticService.createAlias('my-index');
const aliasExists = await ElasticService.checkAliasExists('my-alias');
```

### Rollover Index

Performs a rollover operation on an alias, creating a new index when specified conditions are met (e.g., max documents). Useful for managing large-scale data storage.

```typescript
await ElasticService.performRollover('my-alias', { max_docs: 1000 });
```