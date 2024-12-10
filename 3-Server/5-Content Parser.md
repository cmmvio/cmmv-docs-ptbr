# Content Type Parser

One of the powerful features of the ``@cmmv/server`` framework is its ability to create and register middleware parsers that handle specific content types. Unlike traditional middleware systems (such as in Express), the middleware designed for parsing is only executed if the request's ``Content-Type`` matches the specific type you have registered. This ensures that your application processes the request body efficiently and only when required.

In this documentation, we will cover how to create generic middleware for parsing specific data types, focusing on the use of the ``addContentTypeParser`` function. This function allows you to register custom parsers for different content types.

In web applications, different content types often require different parsing strategies. For example:

* JSON data should be parsed into JavaScript objects.
* XML or CSV may need special handling to parse correctly.
* Multipart form data may require a different strategy to extract files and fields.

The ``addContentTypeParser`` function lets you register middleware to handle specific content types. This ensures that your parser is executed only when the content type matches what you defined.

## Steps to Create 

**1. Using ``addContentTypeParser`` to Register the Parser**

To create a custom parser, you need to call the ``addContentTypeParser`` function within your application. This function accepts the content type(s) to be parsed and the handler function that defines the parsing behavior.

The handler function should always be asynchronous, as it will typically perform I/O operations like reading the request body or processing files.

**2. Defining the Asynchronous Handler Function**

The handler function is where the actual parsing takes place. This function receives the request (``req``), response (``res``), and the raw body payload, which it processes and attaches to the req.body object.

Here's an example of how to create and register a custom parser for application/json content type using ``addContentTypeParser``.

```typescript
const app = cmmv();

// Register a custom content type parser for 'application/json'
app.addContentTypeParser('application/json', async (req, res, payload) => {
    // Convert the raw payload (buffer) into a string
    const bodyString = payload.toString('utf-8');

    // Parse the string into a JavaScript object
    try {
        req.body = JSON.parse(bodyString);
    } catch (err) {
        throw new Error('Invalid JSON payload');
    }
});
```

## Registering Multiple Parsers

You can register multiple parsers for different content types in your application. Here’s an example that registers parsers for both ``application/json`` and ``text/xml`` content types.

```typescript
import xml2js from 'xml2js';

// Register a JSON parser
app.addContentTypeParser('application/json', async (req, res, payload) => {
    const jsonData = payload.toString('utf-8');
    try {
        req.body = JSON.parse(jsonData);
    } catch (err) {
        throw new Error('Invalid JSON payload');
    }
});

// Register an XML parser
app.addContentTypeParser('text/xml', async (req, res, payload) => {
    const xmlData = payload.toString('utf-8');
    try {
        req.body = await xml2js.parseStringPromise(xmlData);
    } catch (err) {
        throw new Error('Invalid XML payload');
    }
});
```

The ``addContentTypeParser`` method in ``@cmmv/server`` provides a powerful way to create middleware that handles specific content types, allowing for flexible and efficient request body parsing. By registering custom parsers that only trigger for their respective content types, you can improve the performance and security of your application.

With asynchronous handler functions, your middleware can handle complex data parsing, such as JSON, XML, or CSV, while maintaining non-blocking operations, making the framework ideal for high-performance server applications.