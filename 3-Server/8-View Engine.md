# View Engine

In web applications, the view engine is responsible for rendering templates and generating HTML pages. In a typical setup, the view engine takes dynamic data and renders it into a static HTML page that is sent to the client. The ``res.render`` method in the server allows you to render a template using the view engine you configure.

In this documentation, we’ll demonstrate how to set up and use a view engine, specifically with the popular Pug template engine. However, it’s possible to configure different view engines depending on your needs.

In the following example, we configure the Pug template engine to render ``.pug`` files from the ``/views`` directory. Here’s how you can set it up:

```typescript
// Import the necessary modules
import cmmv from '@cmmv/server';
import { json, urlencoded } from '@cmmv/server';
import { readFileSync } from 'fs';
import path from 'path';

// Initialize the app
const app = cmmv();

// Set the view engine to Pug
app.set('view engine', 'pug');

// Specify the directory where views are located
app.set('views', path.join(__dirname, 'views'));

// Middleware to parse JSON and URL-encoded data
app.use(json({ limit: '50mb' }));
app.use(urlencoded({ extended: true }));

// Example route to render a Pug template
app.get('/view', function (req, res) {
  res.render('index', { title: 'Hello', message: 'Welcome to the app!' });
});

// Start the server
const host = '0.0.0.0';
const port = 3000;

app.listen({ host, port }).then((server) => {
  console.log(`Server running at http://${host}:${port}`);
}).catch((err) => {
  console.error(err);
});
```

The default views directory is ``/views``, but this can be customized as shown in the example. The Pug template engine expects files with the ``.pug`` extension inside this directory.

For example, the ``index.pug`` file in the ``/views`` directory could look like this:

```pug
doctype html
html
  head
    title= title
  body
    h1= message
```

By setting up a view engine, you can dynamically generate HTML pages based on templates and data. The example provided shows how to configure the Pug template engine and use it to render HTML from Pug templates stored in a specific directory. This allows you to easily manage dynamic content and display it to the client in your web application.

In CMMV, you can configure various view engines supported by Express, such as [EJS](https://ejs.co/), [Mustache](https://mustache.github.io/), and others. Furthermore, you can also create custom implementations to suit your needs. For instance, in CMMV, there is native support for SSR (Server-Side Rendering) using the custom view engine @cmmv/view. Below is an example implementation that demonstrates how to configure and use the CMMV custom view engine.

## Custom View Engine

This implementation configures a custom view engine that processes ``.html`` files for SSR (Server-Side Rendering). It uses CMMV’s own view engine, which provides flexibility in handling dynamic content with custom headers and nonce generation for security policies.

```typescript
import cmmv from '@cmmv/server';
import { v4 as uuidv4 } from 'uuid';
import path from 'path';
import fs from 'fs';
import Config from './config'; // Assume you have a Config service for app configuration

const publicDir = path.join(process.cwd(), 'public');
const app = cmmv();

// Setting up the view engine to process .html files using the custom engine
app.set('views', publicDir);
app.set('view engine', 'html');
app.engine('html', (filePath, options, callback) => {
    app.render.renderFile(
        filePath,
        options,
        { nonce: options.nonce || '' },
        callback,
    );
});

// Adding a hook to manage requests and handle headers with security policies
app.addHook('onRequest', (req, res, next) => {
    req.requestId = uuidv4();
    res.locals = {};
    res.locals.nonce = uuidv4().substring(0, 8);

    // Set custom security headers
    const customHeaders = Config.get('headers') || {};
    for (const headerName in customHeaders) {
        let headerValue = customHeaders[headerName];

        if (Array.isArray(headerValue)) {
            headerValue = headerValue
                .map(value => {
                    if (headerName === 'Content-Security-Policy')
                        return value.indexOf('style-src') == -1
                            ? `${value} 'nonce-${res.locals.nonce}'`
                            : value;
                    return value;
                })
                .join('; ');
        } else if (typeof headerValue === 'string') {
            if (headerName === 'Content-Security-Policy')
                headerValue =
                    headerValue.indexOf('style-src') == -1
                        ? `${headerValue} 'nonce-${res.locals.nonce}'`
                        : headerValue;
        }

        res.setHeader(headerName, headerValue);
    }

    // Serve HTML files from the /public/views directory
    const publicDir = path.join(process.cwd(), 'public/views');
    const requestPath = req.path === '/' ? 'index' : req.path.substring(1);
    const possiblePaths = [
        path.join(publicDir, `${requestPath}.html`),
        path.join(publicDir, requestPath, 'index.html'),
        path.join(publicDir, `${requestPath}`),
        path.join(publicDir, requestPath, 'index.html'),
    ];

    // Check if any of the possible paths exists
    let fileFound = false;
    for (const filePath of possiblePaths) {
        if (fs.existsSync(filePath)) {
            fileFound = true;
            const config = Config.getAll();

            return res.render(filePath, {
                nonce: res.locals.nonce,
                requestId: req.requestId,
                config,
            });
        }
    }

    if (!fileFound) res.code(404).send('Page not found');

    next();
});

// Start the server
const host = '0.0.0.0';
const port = 3000;

app.listen({ host, port }).then(server => {
    console.log(`Server running at http://${host}:${port}`);
});
```

The CMMV framework provides flexibility by supporting any view engine compatible with Express, such as EJS, Mustache, and more. Additionally, you can implement custom view engines, like the ``@cmmv/view`` engine used in the example above, to enhance server-side rendering capabilities and optimize performance with customized behavior.