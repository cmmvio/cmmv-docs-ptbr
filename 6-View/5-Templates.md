# Templates

CMMV provides a flexible templating system, allowing you to configure master page templates for modular views. This setup helps create reusable layouts across your application, streamlining the structure and ensuring consistency in specific views.

A master page template defines the base HTML layout of your views and is stored in the ``/public/templates`` directory. It serves as a foundation where dynamic content, headers, and scripts are injected. The structure follows a consistent format that looks like this:

```html
<!DOCTYPE html>
<html lang="en" data-theme='dark' class="dark">
    <head>
        <headers/>
    </head>
    <body scope> 
        <slot/>
        <scripts/>
    </body>
</html>
```

## Key Elements

* **``<headers/>`` Tag:**

* This tag is used to inject all processed headers from the ``@cmmv/view`` module, which may include metadata, stylesheets, and other elements configured within your views or globally.
* These headers come from your ``.cmmv.config.js`` or from custom settings defined in each view.

* **``<slot/>`` Tag:**

* The ``<slot/>`` element acts as a placeholder for the main content of your view.
* When rendering a view, the content of that view is dynamically injected into the slot.

* **``<scripts/>`` Tag:**

This tag handles the inclusion of JavaScript files configured in your ``.cmmv.config.js`` or directly within the view itself.
This ensures that all required client-side scripts are included in the final rendering of the page.

* **Other Custom Tags:**

Any additional tags or attributes you define in your template will be preserved during the rendering process.
Ensure that all external resources (links, scripts) include the ``nonce="{ nonce }"`` attribute or ``s-attr="nonce"`` for security, as required by the Content Security Policy (CSP) settings in CMMV.

All master page templates are stored in the ``/public/templates`` directory. Here's an example of a possible directory structure:

```bash
/public
    /templates
        /default.html
        /admin.html
        /dashboard.html
```

## Defining a View

In your view, you can configure which master template to use. This is done in the ``s-setup`` section of the view. Here's an example of a view configuration that uses a custom layout and injects scripts:

```html
<script s-setup>
export default {
    layout: "admin",  // Reference to the /public/templates/admin.html file

    head: {
        meta: [
            { name: "description", content: "Admin Panel" },
            { name: "keywords", content: "admin, cmmv, dashboard" }
        ],
        link: [
            { rel: "stylesheet", href: "/assets/styles/admin.css" },
            { rel: "canonical", href: "https://admin.cmmv.io" }
        ]
    },

    scripts: [
        { src: "/assets/js/admin-dashboard.js", async: true }
    ]
}
</script>
```

The ``.cmmv.config.js`` file allows you to manage global JavaScript and stylesheets that should be included in your views. These will be injected into the ``<scripts/>`` and ``<headers/>`` tags of your master templates.

**Example ``.cmmv.config.js:``**

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

    assets: {
        scripts: [
            { src: "/assets/js/main.js", async: true },
            { src: "/assets/js/extra.js", defer: true }
        ],
        styles: [
            { rel: "stylesheet", href: "/assets/css/main.css" }
        ]
    }
};
```

After setting up a view with a master template, the rendered page might look like this:

```html
<!DOCTYPE html>
<html lang="en" data-theme='dark' class="dark">
    <head>
        <meta name="description" content="Admin Panel">
        <meta name="keywords" content="admin, cmmv, dashboard">
        <link 
            rel="stylesheet" 
            href="/assets/styles/admin.css" 
            nonce="a1b2c3"
        />
        <link rel="canonical" href="https://admin.cmmv.io" />
    </head>
    <body scope> 
        <!-- Main content of the view -->
        <div id="dashboard">
            <h1>Welcome to the Admin Dashboard</h1>
        </div>

        <!-- Scripts -->
        <script 
            src="/assets/js/admin-dashboard.js" 
            async 
            nonce="a1b2c3"
        ></script>
    </body>
</html>
```

In CMMV, you can modularize your views by defining master page templates located in ``/public/templates``. These templates handle the injection of headers, dynamic content, and scripts through the ``<headers/>``, ``<slot/>``, and ``<scripts/>`` tags, respectively. By ensuring that security protocols are maintained with attributes like ``nonce="{ nonce }"``, your application remains secure while serving assets efficiently. Through this approach, you can create reusable layouts and maintain consistency across different sections of your application, enhancing both development speed and maintainability.