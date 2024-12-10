# Hooks

The hook system in this server framework is designed to follow a similar lifecycle as that of [Fastify](https://fastify.dev/docs/latest/Reference/Hooks/) but with hooks attached to the application. Hooks are functions that are executed at specific points in the request lifecycle, allowing for customizable behavior during the handling of a request. This system enables you to intercept and manipulate requests, responses, or errors at key points during the lifecycle.

Each hook is attached to the application and can be defined using the ``addHook()`` method. The order in which hooks are called depends on the hook type. Below, we'll cover three specific hooks: ``onRequest``, ``preParsing``, and ``onRequestAbort``.

## onRequest

The ``onRequest`` hook is executed right at the beginning of the request lifecycle, before any parsing of the request. This is the ideal place to perform operations such as authentication, logging, or setting up required request data.

**Example:**

```typescript
app.addHook('onRequest', (req, res, done) => {
    done()
});
```

Or ``async/await``:

```typescript
app.addHook('onRequest', async (req, res) => {
    await asyncMethod()
});
```

## preParsing

The ``preParsing`` hook is called after the ``onRequest`` hook, but before the request body or query parameters are parsed. It is ideal for cases where you need to manipulate or validate the raw request stream before parsing it (e.g., custom decompression or encryption).

**Example:**

```typescript
app.addHook('preParsing', async (req, res) => {
    console.log('Request is about to be parsed');
    // Modify or inspect the raw request stream here
});
```

## onRequestAbort

The ``onRequestAbort`` hook is triggered when the client aborts the request, typically due to a timeout or the client closing the connection before the server sends the response. This is useful for cleanup operations, such as canceling database queries or logging aborted requests.

**Example:**

```typescript
app.addHook('onRequestAbort', async (req) => {
    console.log(`Request aborted: ${req.id}`);
    // Perform any cleanup tasks here
});
```

## onError

The ``onError`` hook is triggered whenever an error occurs during the processing of a request. This hook provides an opportunity to log the error, transform it into a more meaningful response, or handle different types of errors in a custom way.

**Example:**

```typescript
app.addHook('onError', async (error, req, res, done) => {
    console.error('Error occurred:', error);
    res.status(500).send({ message: 'An internal error occurred' });
    done();
});
```

## onSend

The ``onSend`` hook is triggered just before the response is sent to the client, after the request has been processed. This is useful for modifying the response body, headers, or performing final checks before sending the response.

**Example:**

```typescript
app.addHook('onSend', async (req, res, payload) => {
    console.log('Response is about to be sent');
    // Modify the payload or headers
    res.setHeader('X-Custom-Header', 'CustomValue');
    return payload;  // The modified or original payload to send
});
```

## onResponse

The ``onResponse`` hook is executed after the response has been fully sent to the client. This is useful for logging request and response information, performing cleanup tasks, or triggering analytics.

**Example:**

```typescript
app.addHook('onResponse', async (req, res) => {
    console.log(`Response sent for request ${req.id}`);
    // Perform any post-response tasks such as logging
});
```

## onTimeout

The ``onTimeout`` hook is called when the request times out due to exceeding the allowed time for processing. This can happen when the server takes too long to respond. The hook is useful for logging timeout events or triggering alternative behavior when timeouts occur.

**Example:**

```typescript
app.addHook('onTimeout', async (req, res) => {
    console.log(`Request timed out: ${req.id}`);
    res.status(408).send({ message: 'Request Timeout' });
});
```

## onListen

The ``onListen`` hook is triggered when the server starts listening for requests on the specified port and host. It provides an opportunity to perform tasks like logging that the server has started or performing any post-startup operations.

**Example:**

```typescript
app.addHook('onListen', async (server) => {
    console.log(`Server is now listening on ${server.address().port}`);
});
```

## onReady

The ``onReady`` hook is called when the application is fully initialized and ready to handle incoming requests. This is useful for performing any operations that need to occur once the server is completely prepared, such as preloading data, checking system health, or logging a readiness event.

**Example:**

```typescript
app.addHook('onReady', async () => {
    console.log('Application is ready to accept requests');
});
```

## onClose

The ``onClose`` hook is triggered when the server is about to close or shut down. This is useful for cleanup tasks, such as closing database connections, releasing resources, or logging shutdown events.

**Example:**

```typescript
app.addHook('onClose', async (server) => {
    console.log('Server is shutting down');
    // Perform cleanup tasks
});
```