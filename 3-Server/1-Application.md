# Application

The ``Application`` class is the core component that manages the server's lifecycle, routes, and middleware. It acts as the entry point for configuring routes, applying middleware, and handling HTTP requests and responses. This class is designed to be flexible, allowing developers to define various route-handling mechanisms and middleware functions that enhance the server's capabilities.

The ``Application`` class in this system is heavily inspired by frameworks like Express, but it is designed to improve performance and add support for modern HTTP features like HTTP2. It also integrates additional middleware and optimizations such as compression and caching mechanisms.

## app.addRoute()

The ``addRoute`` method allows the application to register specific routes with corresponding handlers. This method takes a configuration object as its parameter, which includes the HTTP method, the route path, and the handler function. The addRoute method ensures that each route is mapped properly within the internal router and can handle requests accordingly.

Exemple:

```typescript
app.addRoute({
  method: 'GET',
  url: '/users',
  handler: (req, res) => {
    res.json({ users: ['John', 'Jane'] });
  }
});
```

This example registers a GET route at the /users URL, with a handler that responds with a JSON object.

## app.use()

The ``use`` method is used to apply middleware functions to the application. Middleware functions are executed in the order they are registered, allowing for various operations such as request validation, logging, and response modifications.

``use`` can also handle sub-applications or routers, making it a powerful tool for structuring large applications into smaller components.

Exemple:

```typescript
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // Pass control to the next middleware/route
});
```

In this example, every incoming request is logged before passing control to the next middleware or route handler.

## app.render()

The ``render`` method is responsible for rendering views in the application. It allows dynamic HTML templates to be rendered with data passed in by the application. This method typically works with template engines like Pug, which are registered via the ``app.set()`` method.

Exemple:

```typescript
app.render('email', function (err, html) {
  // ...
})

app.render('email', { name: 'Tobi' }, function (err, html) {
  // ...
})
```

## app.route()

The ``route`` method provides a mechanism to define a route handler for a specific HTTP method and path. Unlike ``addRoute``, which directly registers the handler, ``route`` returns an object where you can chain methods for configuring the route further.

Exemple:

```typescript
const userRoute = app.route('GET', '/user/:id');

userRoute.stack.push((req, res) => {
  res.send(`User ID: ${req.params.id}`);
});
```

This example creates a ``GET`` route for ``/user/:id`` and registers a handler that responds with the user ID provided in the URL. The ``route`` method is especially useful when you need more control over how the route is handled, allowing for greater flexibility.

## app.engine()

The ``engine`` method is used to register a template engine with the application for rendering dynamic views. Template engines allow HTML files to be dynamically populated with data before being served to the client.

**Parameters:**

* ``ext:`` The file extension for which the engine should be registered. It can be a string representing the extension (e.g., ``'ejs'`` or ``'pug'``).
* ``fn:`` The callback function responsible for rendering the template. This function typically accepts the file path, options, and a callback to render the view.

**Example:**

```typescript
app.engine('pug', require('pug').__express);
app.set('view engine', 'pug');
```

## app.set()

The ``set`` method is used to configure application settings. It can be used to either assign a value to a setting or retrieve the current value of a setting.

**Parameters:**

* **setting:** The name of the setting (e.g., ``'view engine'``, ``'env'``, ``'etag'``).
* **val:** The value to assign to the setting.

**Example:**

```typescript
app.set('env', 'production');
app.set('view engine', 'pug');
```

In this example, the environment is set to ``'production'``, and the view engine is set to ``'pug'``.

**Special Settings:**

* ``'etag':`` Configures the ETag generation mechanism for caching.
* ``'query parser':`` Sets how the query string should be parsed.
* ``'trust proxy':`` Determines if the app should trust reverse proxies.

## app.param()

The ``param`` method is used to define route parameter preprocessing. This is useful for middleware that performs actions on specific route parameters before the request is processed.

**Parameters:**

* ``name:`` The name of the route parameter, or an array of parameter names.
* ``fn:`` A callback function to handle the route parameter. The callback receives ``(req, res, next, value)``.

**Example:**

```typescript
app.param('id', (req, res, next, id) => {
  console.log(`Received ID: ${id}`);
  next();
});
```

This example logs the ``id`` parameter whenever a route that includes ``:id`` is hit.

## app.path()

The ``path`` method returns the application's absolute pathname. If the application is mounted at a specific path (e.g., ``/admin``), it will return the full mount path, including any parent application mounts.

**Example:**

```typescript
const admin = express();
app.use('/admin', admin);
console.log(admin.path());  // Outputs: '/admin'
```

## app.enabled()

The ``enabled`` method checks if a specific setting is enabled (i.e., truthy). It returns ``true`` if the setting is enabled, and ``false`` if it is not.

**Example:**

```typescript
app.set('view cache', true);
console.log(app.enabled('view cache'));  // Outputs: true
```

## app.disabled()

The ``disabled`` method checks if a specific setting is disabled (i.e., falsy). It returns ``true`` if the setting is disabled, and ``false`` if it is not.

**Example:**

```typescript
app.set('view cache', false);
console.log(app.disabled('view cache'));  // Outputs: true
```

## app.enable()

The ``enable`` method enables a specific setting, making it truthy.

**Example:**

```typescript
app.enable('view cache');
console.log(app.enabled('view cache'));  // Outputs: true
```

## app.disable()

The ``disable`` method disables a specific setting, making it falsy.

**Example:**

```typescript
app.disable('view cache');
console.log(app.disabled('view cache'));  // Outputs: true
```

## app.setErrorHandler()

The ``setErrorHandler`` method sets a global error handler for the application. This handler will be invoked whenever an error occurs during request processing. The default error handler will be overridden by the one provided in this method.

**Example:**

```typescript
app.setErrorHandler((error, req, res) => {
  console.error('Error occurred:', error);
  res.status(500).send({ error: 'Internal Server Error' });
});
```

## app.addHook()

The ``addHook`` method allows you to attach hooks to specific lifecycle events during the request-response cycle. You can add hooks for events such as ``onSend``, ``preSerialization``, ``onError``, and more.

**Example:**

```typescript
app.addHook('onSend', async (request, response, payload) => {
  console.log('Response being sent for URL:', request.url);
});
```

This example adds a hook for the onSend event that logs a message just before the response is sent to the client.

## app.addContentTypeParser()

The ``addContentTypeParser`` method registers a custom content-type parser for handling different types of request bodies. It allows the application to parse specific content types such as XML, YAML, etc.

**Example:**

```typescript
app.addContentTypeParser('application/xml', (req, res, body, done) => {
  const parsed = parseXml(body); // Custom XML parsing logic
  done(null, parsed);
});
```

This example registers a custom parser for application/xml content type that converts XML into a usable JavaScript object.

## app.contentTypeParser()

The ``contentTypeParser`` method is used internally to process incoming request bodies based on the registered content-type parsers. It finds the parser for the given content type and applies it to the request body.

**Example:**

```typescript
app.contentTypeParser('application/json', (req, res, next) => {
  console.log(req.body); // Process parsed JSON body
  next();
}, req, res);
```

This example processes requests with a content type of application/json and logs the parsed body.

## app.METHOD

The ``app.METHOD()`` functions in the framework allow you to define routes based on specific HTTP methods, such as ``GET``, ``PUT``, ``POST``, and ``DELETE``. Each HTTP method corresponds to a method call on the application object, for example, ``app.get()``, ``app.post()``, ``app.put()``, etc. These methods handle incoming requests by routing them to the appropriate callback based on the HTTP method and URL.

While the documentation focuses primarily on the most common methods (``GET``, ``POST``, ``PUT``, ``DELETE``), other HTTP methods like ``PATCH``, ``OPTIONS``, and ``HEAD`` are supported in the same way. You can define routes using any standard HTTP method by using the corresponding function on the app object.

For HTTP methods that are not valid JavaScript variable names (e.g., ``M-SEARCH``), you can use bracket notation to define the route. For example:

```typescript
app['m-search']('/', function(req, res) {
    res.send('M-SEARCH request');
});
```

Additionally, when defining a route with ``app.get()``, the framework automatically handles ``HEAD`` requests for the same path, unless you explicitly define ``app.head()`` for that path.

The ``app.all()`` function is special in that it routes requests of all HTTP methods to a specific path. This is not derived from any one HTTP method, making it useful for middleware or handling any type of request at a given route.

For more detailed information on routing and the various HTTP methods supported, see the routing guide in the documentation.

<table style="border: 0px; background: none">
<tbody><tr>
<td style="background: none; border: 0px;">
<ul>
<li><code>checkout</code></li>
<li><code>copy</code></li>
<li><code>delete</code></li>
<li><code>get</code></li>
<li><code>head</code></li>
<li><code>lock</code></li>
<li><code>merge</code></li>
<li><code>mkactivity</code></li>
</ul>
</td>
<td style="background: none; border: 0px;">
<ul>
<li><code>mkcol</code></li>
<li><code>move</code></li>
<li><code>m-search</code></li>
<li><code>notify</code></li>
<li><code>options</code></li>
<li><code>patch</code></li>
<li><code>post</code></li>
</ul>
</td>
<td style="background: none; border: 0px;">
<ul>
<li><code>purge</code></li>
<li><code>put</code></li>
<li><code>report</code></li>
<li><code>search</code></li>
<li><code>subscribe</code></li>
<li><code>trace</code></li>
<li><code>unlock</code></li>
<li><code>unsubscribe</code></li>
</ul>
</td>
</tr>
</tbody></table>

## app.listen

The ``listen()`` method starts the server on the specified host and port, returning a Promise that resolves when the server successfully starts listening for incoming connections, or rejects if an error occurs.

**Example:**

```typescript
const listenOptions = { host: '0.0.0.0', port: 3000 };

app.listen(listenOptions)
.then(server => {
    const attr = server.address();
    console.log(`Server is listening at ${attr.address}:${attr.port}`);
})
.catch(err => {
    console.error('Failed to start the server:', err);
});
```