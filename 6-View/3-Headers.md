# Headers

In CMMV, you can configure response headers to enhance security, define metadata, or customize the behavior of your web application. This can be achieved in two ways: using the ``.cmmv.config.js`` file located in the root of your project or by adding specific configurations directly into the template files for more granular control.

The ``.cmmv.config.js`` file serves as the global configuration file for your CMMV project. You can set default headers that will apply to all responses from your application by defining the headers object in this file.

For example, to configure a Content-Security-Policy and other security-related headers, your ``.cmmv.config.js`` might look like this:

```typescript
module.exports = {
    headers: {
        "Content-Security-Policy": [
            "default-src 'self'",
            "script-src 'self' 'unsafe-eval'",
            "style-src 'self' 'unsafe-inline'",
            "font-src 'self'",
            "connect-src 'self'",
            "img-src 'self' data:"
        ],
        "X-Frame-Options": "DENY",
        "X-Content-Type-Options": "nosniff",
        "Referrer-Policy": "no-referrer",
        "Strict-Transport-Security": "max-age=31536000; ..."
    }
}
```

For more specific configurations, such as adding metadata or setting headers that only apply to a particular route, you can define these directly in your Vue-style template files.

In the template's ``<script>`` block, under head, you can configure meta tags, link tags, and other headers:

```html
<script s-setup>
export default {
    layout: "default",

    head: {
        meta: [
            { name: "description", content: "CMMV Todolist sample" },
            { name: "keywords", content: "cmmv, contract model, websocket" }
        ],
        link: [
            { rel: "stylesheet", href: "/assets/styles/todo.css" },
            { rel: "canonical", href: "https://cmmv.io" },
        ],
        script: [
            { src: "https://cdn.jsdelivr.net/npm/some-lib.js", async: true }
        ]
    }

    ...
}
</script>
```

* **meta:** Adds or overrides meta tags in the page's head, including those for description, keywords, and Content-Security-Policy.
* **link:** Used to include external resources like stylesheets or canonical URLs for SEO.
* **script:** Enables you to include additional scripts with custom attributes like async or defer.

By configuring headers both globally and at the template level, you have full control over the behavior, security, and SEO optimization of your CMMV application.

# HTTP Module

The ``@cmmv/http`` module provides an automatic way to manage and optimize the headers that are sent with HTTP responses, reducing the size of unnecessary headers based on the request type (e.g., ``GET``, ``POST``, ``PUT``, or ``DELETE`` requests). This built-in functionality ensures that only relevant headers are included, minimizing overhead and improving performance.

```javascript
if (req.method === 'GET') {
    res.setHeader(
        'Strict-Transport-Security',
        'max-age=15552000; includeSubDomains',
    );
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'SAMEORIGIN');
    res.setHeader('X-XSS-Protection', '0');
}
```

* **Strict-Transport-Security:** Ensures that the browser only communicates with the server over HTTPS for the next 180 days (15552000 seconds) and applies this policy to all subdomains.

* **X-Content-Type-Options:** Prevents browsers from interpreting files as a different MIME type, which can help prevent attacks.

* **X-Frame-Options:** Protects the site from clickjacking attacks by only allowing it to be displayed in a frame of the same origin.

* **X-XSS-Protection:** Disables XSS filtering to avoid potential browser-level issues.

POST, PUT, DELETE Requests: For these types of requests, the module removes specific headers that are not necessary for write operations, ensuring smaller and more efficient responses.

```javascript
if (['POST', 'PUT', 'DELETE'].includes(req.method)) {
    res.removeHeader('X-DNS-Prefetch-Control');
    res.removeHeader('X-Download-Options');
    res.removeHeader('X-Permitted-Cross-Domain-Policies');
    res.removeHeader('Strict-Transport-Security');
    res.removeHeader('Content-Security-Policy');
    res.removeHeader('Cross-Origin-Opener-Policy');
    res.removeHeader('Cross-Origin-Resource-Policy');
    res.removeHeader('Origin-Agent-Cluster');
    res.removeHeader('Referrer-Policy');
}
```

* **X-DNS-Prefetch-Control:** Reduces DNS prefetching, which is not needed in POST requests.
* **X-Download-Options:** Typically used for file downloads; unnecessary for most write operations.
* **X-Permitted-Cross-Domain-Policies:** Restricts which domain policies are sent, but not required for write operations.
* **Strict-Transport-Security:** Removed in write requests to allow more flexibility in transport for sensitive operations.
* **Content-Security-Policy:** Temporarily disabled for certain write requests.
* **Cross-Origin-Opener-Policy:** Removed to allow more flexibility when interacting with cross-origin content.
* **Cross-Origin-Resource-Policy:** Ensures more open policies for resource sharing during POST, PUT, or DELETE requests.
* **Origin-Agent-Cluster:** Removes restrictions on the agent's ability to share resources across origins.
* **Referrer-Policy:** Not needed for write operations, removed to reduce header size.

The ``@cmmv/http`` module intelligently manages headers to optimize performance, especially for write operations (``POST``, ``PUT``, ``DELETE``). For read-only operations (``GET``), it ensures strong security by setting appropriate headers. This approach reduces the amount of unnecessary data sent with each request, while still adhering to best practices in web security.

# Additional Modules

The ``@cmmv/http`` module integrates several essential middleware components to further enhance security, session management, and overall performance. These include ``cors``, ``helmet``, ``session``, and ``compression``, which modify headers and handle certain aspects of HTTP communication.

## CORS

Controls access to resources from different origins by setting appropriate CORS (Cross-Origin Resource Sharing) headers.

**Effect on Headers:**

* **Access-Control-Allow-Origin:** Specifies which origins can access the server.
* **Access-Control-Allow-Methods:** Lists allowed HTTP methods (e.g., GET, POST).
* **Access-Control-Allow-Headers:** Indicates which headers can be used during the actual request.
* **Access-Control-Allow-Credentials:** Indicates whether the response to the request can be exposed to the client.

## Helmet

Helmet helps secure the application by setting various HTTP headers that protect against common attacks.

**Effect on Headers:**

* **Content-Security-Policy:** Restricts the sources from which the browser can load resources.
* **X-DNS-Prefetch-Control:** Disables DNS prefetching to reduce privacy leakage.
* **X-Frame-Options:** Protects against clickjacking by controlling whether the page can be embedded in a frame.
* **X-Permitted-Cross-Domain-Policies:** Controls Adobe Flash and PDF cross-domain policies.
* **Strict-Transport-Security:** Forces secure (HTTPS) connections.
* **X-Download-Options:** Prevents browsers from downloading files from your website without authorization.
* **Referrer-Policy:** Controls how much referrer information is included with requests.

## Session

Manages user sessions via cookies, storing user authentication state and other session data.

* **Set-Cookie:** Sets the session cookie with attributes like HttpOnly, Secure, SameSite, etc.

## Compression

Compresses the HTTP responses using Gzip or Brotli to reduce the size of the response body, speeding up the transmission.

**Effect on Headers:**

* **Content-Encoding:** Specifies the type of compression used (e.g., gzip, br).
* **Vary:** Instructs the browser to vary the response based on the Accept-Encoding header, which allows the server to return different content based on the client's compression capabilities.

The ``.cmmv.config.js`` file in the root of the project allows developers to define and customize security policies, session management, CORS settings, and compression options. These configurations will automatically modify the response headers to meet security and performance requirements.

Example ``.cmmv.config.js`` Configuration:
```javascript
module.exports = {
    headers: {
        "Content-Security-Policy": [
            "default-src 'self'",
            "script-src 'self' 'unsafe-eval'",
            "style-src 'self' 'unsafe-inline'",
            "font-src 'self'",
            "connect-src 'self'",
            "img-src 'self' data:"
        ]
    },
    cors: {
        origin: "https://example.com",
        methods: ["GET", "POST"],
        credentials: true
    },
    helmet: {
        contentSecurityPolicy: {
            directives: {
                defaultSrc: ["'self'"],
                scriptSrc: ["'self'", "https://cdn.jsdelivr.net"],
                styleSrc: ["'self'", "'unsafe-inline'"]
            }
        }
    },
    session: {
        secret: "my-secret-key",
        resave: false,
        saveUninitialized: false,
        cookie: {
            secure: true,
            httpOnly: true,
            sameSite: 'lax'
        }
    },
    compression: {
        level: 6,
        threshold: 1024
    }
};
```

### Header Customization

The headers can also be set or modified directly within the template setup using the ``head`` property. This provides flexibility to add meta tags, canonical links, and other SEO-related settings dynamically per page.

```html
<script s-setup>
export default {
    layout: "default",

    head: {
        meta: [
            { name: "description", content: "CMMV Todolist sample" },
            { name: "keywords", content: "cmmv, contract model, websocket" }
        ],
        link: [
            { rel: "stylesheet", href: "/assets/styles/todo.css" },
            { rel: "canonical", href: "https://cmmv.io" },
        ]
    },
}
</script>
```

The ``@cmmv/http`` module automates essential optimizations to HTTP headers, ensuring security and performance through compression, session management, and protection against common web attacks via middleware like ``cors``, ``helmet``, and ``compression``. This is supplemented by custom configurations defined in the ``.cmmv.config.js`` file and template-specific header controls, making the CMMV platform a secure and efficient foundation for web applications.