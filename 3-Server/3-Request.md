# Request

The ``req`` object represents the incoming HTTP request and provides access to various properties and methods, such as the request's query string, parameters, body, HTTP headers, and more. In this documentation, as well as by common convention, this object is referred to as req (with the HTTP response object being res). However, the actual names used for these objects can be defined by the parameters in the callback function you're working within.

**Example:**
```typescript
app.get('/user/:id', function (req, res) {
  res.send('User: ' + req.params.id);
});
```

Alternatively, you could name the parameters differently, like this:

```typescript
app.get('/user/:id', function (request, response) {
  response.send('User: ' + request.params.id);
});
```

The ``req`` object extends Node.js’s core ``http.IncomingMessage`` object, which means it inherits all the built-in properties and methods of Node's request object. In addition, the ``req`` object includes a number of added functionalities specific to HTTP request handling in your application, making it more powerful and easier to work with.

## Properties

## req.query

Returns the parsed query string as an object. The query string is automatically parsed using the query parser (e.g., qs library).

```typescript
// URL: /search?name=John&age=30
console.log(req.query);  // { name: 'John', age: '30' }
```

## req.querystring

Returns the raw query string from the URL.

```typescript
// URL: /search?name=John&age=30
console.log(req.querystring);  // "name=John&age=30"
```

## req.search

Returns the search string, which is the query string prefixed with a question mark (``?``).

```typescript
// URL: /search?name=John&age=30
console.log(req.search);  // "?name=John&age=30"
```

## req.socket

Returns the request socket object, which represents the underlying connection.

## req.protocol

Returns the protocol used in the request (``http`` or ``https``). If the application is behind a proxy, it will also check the ``X-Forwarded-Proto`` header if the ``trust proxy`` setting is enabled.

```typescript
console.log(req.protocol);  // "https"
```

## req.headers

Returns the request headers as an object.

```typescript
console.log(req.headers['content-type']);  // "application/json"
```

## req.url

Returns the request URL.

```typescript
console.log(req.url);  // "/search?name=John&age=30"
```

## req.origin

Returns the origin of the request, including the protocol and host.

```typescript
console.log(req.origin);  // "https://example.com"
```

## req.href

Returns the full request URL, including the origin and original URL.

```typescript
console.log(req.href);  // "https://example.com/search?name=John&age=30"
```

## req.secure

Returns the HTTP method of the request (e.g., ``GET``, ``POST``, etc.).

```typescript
console.log(req.method);  // "GET"
```

## req.path

Returns the path of the request URL without the query string.

```typescript
// URL: /search?name=John
console.log(req.path);  // "/search"
```

## req.host

Returns the host name from the request, considering the ``X-Forwarded-Host`` header if the ``trust proxy`` setting is enabled.

```typescript
console.log(req.host);  // "example.com"
```

## req.hostname

Returns the hostname from the ``Host`` header, excluding the port number.

```typescript
console.log(req.hostname);  // "example.com"
```

## req.URL

Returns a parsed URL object from the WHATWG URL API.

## req.fresh

Returns ``true`` if the request is considered "fresh", meaning the response has not changed since the last request (based on ``Last-Modified`` and/or ``ETag`` headers).

## req.stale

Returns ``true`` if the request is considered "stale", meaning the resource has changed since the last request.

## req.idempotent

Returns ``true`` if the request method is idempotent (``GET``, ``HEAD``, ``PUT``, ``DELETE``, etc.).

## req.ip

Returns the remote IP address of the request, considering the ``trust proxy`` setting.

## req.ips

Returns an array of IP addresses, with the client IP first, followed by any proxy addresses (when ``trust proxy`` is enabled).

## req.length

Returns the ``Content-Length`` header value as a number if present.

## req.subdomains

Returns an array of subdomains from the request's hostname. The subdomains are determined by the ``subdomain offset`` setting.

## req.xhr

Returns ``true`` if the request was made using an XMLHttpRequest (i.e., an Ajax request).

## req.cookies

Returns the cookies sent by the client.

## req.signedCookies

Returns the signed cookies sent by the client.

## Methods

## req.header()

Alias for ``req.get()``. Returns the value of the specified request header field.

```typescript
req.header('Content-Type');  // "application/json"
```

## req.get()

Returns the value of the specified request header field.

```typescript
req.get('Content-Type');  // "application/json"
```

## req.accepts()

Checks if the specified MIME types are acceptable based on the Accept header. Returns the best match or ``undefined``.

```typescript
req.accepts('html');  // "html"
```

## req.acceptsEncodings()

Checks if the specified encodings are acceptable based on the ``Accept-Encoding`` header.

```typescript
req.acceptsEncodings('gzip');  // "gzip"
```

## req.acceptsCharsets()

Checks if the specified charsets are acceptable based on the ``Accept-Charset`` header.

```typescript
req.acceptsCharsets('utf-8');  // "utf-8"
```

## req.acceptsLanguages()

Checks if the specified languages are acceptable based on the ``Accept-Language`` header.

```typescript
req.acceptsLanguages('en');  // "en"
```

## req.range()

Parses the Range header and returns an array of ranges, or ``-1`` if unsatisfiable, or ``-2`` if invalid.

```typescript
req.range(1000);  // [{ start: 0, end: 999 }]
```

## req\.is()

Checks if the request's ``Content-Type`` matches the specified MIME type.

```typescript
req.is('json');  // "json"
```