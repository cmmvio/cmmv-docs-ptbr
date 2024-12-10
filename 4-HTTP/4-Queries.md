# Queries

The ``findAll`` function in the repository layer has been enhanced to support dynamic query filtering and is compatible with both SQL and MongoDB databases. The caching system provided by ``@cmmv/cache`` has also been updated to support multiple filters, improving flexibility and performance.

Fetches all records of a specified entity, applying filters, pagination, and sorting dynamically based on the provided ``queries``. This method is compatible with both SQL and MongoDB, adapting its behavior based on the database type.

| **Parameter**      | **Type**   | **Description**                                                                                  | **Default** |
|---------------------|------------|--------------------------------------------------------------------------------------------------|-------------|
| `limit`            | *(number)* | Specifies the maximum number of results to return.                                              | `10`        |
| `offset`           | *(number)* | Specifies the number of results to skip.                                                        | `0`         |
| `sortBy`           | *(string)* | Specifies the field by which to sort the results.                                               | `'id'`      |
| `sort`             | *(string)* | Specifies the sort order. Possible values are `'asc'` and `'desc'`.                             | `'asc'`     |
| `search`           | *(string)* | Performs a case-insensitive search on the specified field.                                       | -           |
| `searchField`      | *(string)* | Specifies the field to perform the search operation on.                                          | -           |
| **Additional Filters** | *(varied)* | Any additional query parameters are dynamically applied as filters based on the entity's field names. | -           |

1. SQL Query Handling:

    * Converts ``search`` and ``searchField`` into ``LIKE`` conditions.
    * Dynamically builds the ``WHERE`` clause based on ``filters``.
    * Uses ``LIMIT``, ``OFFSET``, and ``ORDER BY`` for pagination and sorting.

2. MongoDB Query Handling:

    * Uses ``$regex`` for case-insensitive searches.
    * Dynamically applies filters as part of the MongoDB ``find`` query.
    * Supports ``skip``, ``limit``, and sorting using MongoDB-compatible syntax.

### Example 

<br/>

```typescript
const tasks = await Repository.findAll(TaskEntity, {
    limit: 20,
    offset: 0,
    sortBy: 'createdAt',
    sort: 'desc',
    search: 'John',
    searchField: 'name',
    status: 'active',
}); 
```

## Caching 

If the @cmmv/cache module is configured in the system, the caching mechanism dynamically adjusts the cache key to include query parameters such as search, ensuring that filtered results are cached under unique keys. This allows the system to cache and retrieve results efficiently, even when queries involve dynamic filters like limit, offset, or searchField.

When a controller method is decorated with @Cache, the following steps occur:

1. Base Cache Key:
The base key provided in the decorator (e.g., ``"task:getAll"``) is used.

2. Query Parameters:
If the request includes query parameters such as ``search`` or ``searchField``, these are appended to the cache key. For example:

* Request:
``GET /api/tasks?search=John&searchField=name&limit=10``
* Cache Key:
``task:getAll:search=John&searchField=name&limit=10``

### Example 

```typescript
@Get()
@Cache("task:getAll", { ttl: 300, compress: true, schema: TaskFastSchema })
async getAll(@Queries() queries: any, @Request() req) {
    Telemetry.start('TaskController::GetAll', req.requestId);
    const result = await this.taskservice.getAll(queries, req);
    Telemetry.end('TaskController::GetAll', req.requestId);
    return result;
}
```

## Example Workflow

Client sends a GET request with query parameters:

```
GET /api/tasks?search=John&searchField=name&limit=5
```

**Controller**

```typescript
@Get()
@Cache("task:getAll", { ttl: 300, compress: true, schema: TaskFastSchema })
async getAll(@Queries() queries: any, @Request() req) {
    return await this.taskservice.getAll(queries, req);
}
```

**SQL**

```sql
SELECT * FROM tasks WHERE LOWER(name) LIKE LOWER('%John%') LIMIT 5 OFFSET 0;
```

**MongoDB**

```javascript
{ name: { $regex: /John/i } }
```

**Cache Key**

```
task:getAll:search=John&searchField=name&limit=5
```
