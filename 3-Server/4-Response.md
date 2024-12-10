# Response

The ``res`` object represents the HTTP response in a server-side application and is responsible for managing and sending responses back to the client. This object allows you to set status codes, headers, cookies, and more. It is an enhanced version of the standard Node.js ``ServerResponse`` object, providing additional features for greater control over the response.  ``res`` object is commonly used to send various types of responses—such as HTML, JSON, or files—to the client. By convention, the ``res`` object refers to the response (and req refers to the request), but its name can be different depending on the parameters used in your callback function.

**Example:**

```typescript
app.get('/user/:id', function (req, res) {
  res.send('user ' + req.params.id);
});
```

You can also name the parameters differently, like this:

```typescript
app.get('/user/:id', function (request, response) {
  response.send('user ' + request.params.id);
});
```

The res object not only supports all built-in methods and properties of Node’s core ``ServerResponse`` but also offers additional capabilities, making it easier to manage the HTTP lifecycle in modern applications.

## Properties

### res.socket

Returns the request socket.

```typescript
console.log(res.socket);
```

### res.status

Retrieves the HTTP status code of the response.

```typescript
console.log(res.status); // Outputs the status code
```

### res.now

Returns the current timestamp in milliseconds.

```typescript
console.log(res.now); // Outputs the current timestamp
```

### res.elapsedTime

Returns the elapsed time in milliseconds since the response started.

```typescript
console.log(res.elapsedTime); // Outputs the time elapsed
```

### res.sent

Checks if the response has already been sent.

```typescript
if (res.sent) {
  console.log('Response already sent');
}
```

### res.headerSent

Returns ``true`` if the headers have already been sent.

### res.writable

Returns ``true`` if the response is still writable (not ended).

### res.message (get/set)

Retrieves the HTTP status message of the response.

### res.body (get/set)

Returns/set the response body, Can accept different types such as string, Buffer, Object, Stream.

### res.length (get/set)

Returns the length of the response content. / Sets the Content-Length header to n.

### res.status (get/set)

Sets the HTTP status code for the response.

```typescript
res.status = 404;
```

### res.lastModified (get/set)

Sets the ``Last-Modified`` header using a string or a ``Date``.

```typescript
res.lastModified = new Date();
```

## Methods

## res.has(field)

Checks if a specific header is present in the response.

```typescript
if (res.has('Content-Type')) {
  console.log('Content-Type header is set');
}
```

## res.remove(field)

Removes a specific header from the response.

```typescript
res.remove('Content-Type');
```

## res.hijack()

Marks the response as hijacked.

## res.code(code)

Sets the HTTP status code and returns the response for chaining.

```typescript
res.code(200);
```

## res.links(links)

Sets the ``Link`` header field with the given ``links`` object.

```typescript
res.links({
  next: 'http://example.com/page2',
  last: 'http://example.com/page5'
});
```

## res.header(field, val)

Sets a header field to a specific value.

```typescript
res.header('Content-Type', 'application/json');
```

## res.get(field)

Retrieves the value of a specific header.

```typescript
console.log(res.get('Content-Type'));
```

## res.type(type)

Sets the ``Content-Type`` based on the provided type.

```typescript
res.type('json');
```

## res.sendStatus(statusCode)

Sends the HTTP status code with the standard message.

```typescript
res.sendStatus(404);
```

## res.send(payload)

Sends the response with the provided payload.

```typescript
res.send({ message: 'Hello World' });
```

## res.json(obj)

Sends a JSON response.

```typescript
res.json({ user: 'John' });
```

## res.jsonp(obj)

Sends a JSONP response with callback support.

## res.sendFile(path, opt, cb)

The ``res.sendFile()`` function is used to transfer the file located at a given ``path`` to the client. It automatically sets the ``Content-Type`` response header based on the file extension and triggers a callback when the file transfer is complete or if an error occurs. This method is particularly useful for serving static files or dynamically serving files in response to requests.

Here’s an overview of how ``res.sendFile()`` works, along with additional options that can be used to control file serving behavior.

| Option          | Type           | Default     | Description                                                                                                                                     |
|-----------------|----------------|-------------|-------------------------------------------------------------------------------------------------------------------------------------------------|
| `maxAge`        | number/string  | `0`         | Specifies the `Cache-Control` max-age property in milliseconds or a string (converted by `ms`). Used to set how long the file is cached by clients. |
| `root`          | string         | `undefined` | The root directory from which relative file paths are resolved. Without `root`, the `path` must be an absolute path.                               |
| `headers`       | object         | `{}`        | Object containing custom headers that will be added to the response when the file is served.                                                     |
| `dotfiles`      | string         | `'ignore'`  | Determines how dotfiles (files or directories starting with a dot, like `.gitignore`) are handled. Can be set to `'allow'`, `'deny'`, or `'ignore'`. |
| `etag`          | boolean        | `true`      | Whether to generate an `ETag` header for the file.                                                                                               |
| `lastModified`  | boolean        | `true`      | Whether to set the `Last-Modified` header for the file.                                                                                          |
| `cacheControl`  | boolean        | `true`      | Enables or disables setting the `Cache-Control` header in the response.                                                                          |
| `acceptRanges`  | boolean        | `true`      | Allows partial content (via the `Range` header). This enables clients to request parts of the file, especially for video and audio streaming.     |
| `immutable`     | boolean        | `false`     | When set to `true`, adds `Cache-Control: immutable` to the response, indicating the file will never change and can be cached indefinitely.       |

## res.download(path, filename, opt, cb)

The ``res.download()`` method is used to transfer the file located at the specified ``path`` as an attachment. This prompts the client to download the file. You can optionally provide a different filename for the downloaded file, and a callback function to handle any errors that may occur during the transfer.

This method also accepts an ``options`` object similar to the one used with ``res.sendFile()``. The ``Content-Disposition`` header is automatically set to signal that the file should be downloaded as an attachment, overriding any previously set ``Content-Disposition`` headers.

Internally, this method uses ``res.sendFile()`` to handle the file transfer.

## res.end(payload, encoding, cb)

Ends the response process.

## res.format(obj)

Responds to the request by calling the appropriate callback from the object ``obj`` based on the ``Accept`` header.

```typescript
res.format({
  'text/plain': () => res.send('Plain text'),
  'application/json': () => res.json({ message: 'JSON' })
});
```

## res.vary(field)

Adds the given field to the ``Vary`` header.

```typescript
res.vary('Accept-Encoding');
```

## res.attachment(filename)

Sets the ``Content-Disposition`` header to attachment and sets the file name.

## res.append(field, val)

Appends a value to a specific header.

```typescript
res.append('Link', ['<http://localhost/>', '<http://localhost:3000/>']);
res.append('Set-Cookie', 'foo=bar; Path=/; HttpOnly');
res.append('Warning', '199 Miscellaneous warning');
```

## res.location(url)

Sets the ``Location`` header to the given URL.

```typescript
res.location('/foo/bar').;
res.location('http://example.com');
res.location('../login');
```

## res.redirect(url, alt)

Performs a 302 redirect to the specified URL.

```typescript
this.redirect('back');
this.redirect('back', '/index.html');
this.redirect('/login');
this.redirect('http://google.com');
```

## res.clearCookie(name, options)

Clears the cookie with the given name.

```typescript
res.clearCookie('session_id');
```

## res.cookie(name, value, options)

The ``res.cookie()`` method is used to set a cookie with a specified name and value. It allows you to define additional options to control the behavior of the cookie, such as its expiration time, security flags, and more. The method is chainable, meaning you can call it multiple times in a single response.

```typescript
// Set a cookie that expires in 15 minutes
res.cookie('rememberme', '1', { 
    expires: new Date(Date.now() + 900000), 
    httpOnly: true 
});

// Set a cookie with max-age of 15 minutes (in milliseconds)
res.cookie('rememberme', '1', { 
    maxAge: 900000, 
    httpOnly: true 
});
```

| Option     | Type            | Default     | Description                                                                                                    |
|------------|-----------------|-------------|----------------------------------------------------------------------------------------------------------------|
| `maxAge`   | number           | `undefined` | Specifies the cookie's `max-age` in milliseconds. If set, it overrides the `expires` option.                   |
| `expires`  | Date             | `undefined` | Sets the cookie's expiration date. Use `maxAge` as an alternative for setting relative expiration time.         |
| `path`     | string           | `'/'`       | Defines the URL path for which the cookie is valid. Defaults to the root path `/`.                             |
| `domain`   | string           | `undefined` | Specifies the domain for which the cookie is valid.                                                            |
| `secure`   | boolean          | `false`     | Marks the cookie as secure, ensuring it is only sent over HTTPS.                                                |
| `httpOnly` | boolean          | `false`     | Marks the cookie as HTTP-only, meaning it can't be accessed via JavaScript in the browser.                       |
| `signed`   | boolean          | `false`     | Indicates whether the cookie should be signed. Requires a secret to be set in `cookieParser()`.                  |
| `sameSite` | boolean/string   | `false`     | Restricts how the cookie is sent across sites. Can be `'strict'`, `'lax'`, or `true` (which sets to `'strict'`). |

## res.render(view, opt, cb)

Renders a view template with the provided options and an optional callback function. If a callback is provided, it is executed once the view has been rendered, and no automatic response is sent. If no callback is provided, the method sends a default response with a status code of ``200`` and content type ``text/html``.

**Example:**

```typescript
app.get('/about', (req, res) => {
    res.render('about', { title: 'About Us', company: 'MyCompany' });
});
```

**Callback Example:**

```typescript
app.get('/custom-view', (req, res) => {
    res.render('custom', { title: 'Custom Page' }, (err, html) => {
        if (err) 
            return res.status(500).send('Error rendering the page.');
        
        res.send(html);
    });
});
```

In this example, a custom callback function is provided. If an error occurs during rendering, the callback handles it by sending a ``500`` status code and error message to the client.

## Response Compatibility 

### res.trailer(key, fn)

Sets a trailer header with the provided key and function.

### res.flushHeaders()

Flushes the response headers.

### res.getHeaderNames()

Returns an array of the names of the headers currently set in the response.

### res.getHeaders()

Returns the response headers object.

### res.hasHeader(name)

Checks if a specific header is present in the response.

### res.removeHeader(name)

Removes a specific header from the response.

### res.setTimeout(msecs, cb)

Sets a timeout for the response.

### res.uncork()

Uncorks the response stream.

### res.writeEarlyHints(hints, callback)

Writes HTTP/1.1 103 Early Hints with the provided hints object.