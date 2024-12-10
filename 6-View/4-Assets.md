# Assets

In CMMV, it is recommended that all static files, such as JavaScript libraries, CSS, fonts, and images, are served through a Content Delivery Network (CDN). CDNs are optimized for fast and efficient delivery of assets, reducing latency and improving the user experience. However, if you choose to serve these files directly from your application, they must be placed in the ``/public`` directory. The ``@cmmv/http`` module automatically looks for and serves static files from this directory.

If you choose not to use a CDN, place all your static files under the ``/public`` directory. This is the directory where @cmmv/http will automatically serve static files like:

## Bundle 

By default, the application will generate complementary files, resulting in a final bundle that is necessary if you are using RPC and frontend reactivity. This bundle will be created as ``/assets/bundle.min.js`` and must be included in your HTML or template files to ensure the frontend functions correctly.

**Example ``.cmmv.config.js`` Configuration for Assets:**

```javascript
module.exports = {
    ...

    scripts: [
        { type: "text/javascript", src: "/assets/bundle.min.js", defer: "defer" },
        ...
    ]
};
```

If you are serving assets locally, ensure your ``/public`` directory is structured as follows:

```bash
/public
    /assets
        /styles.css
        /app.min.js
        /bundle.min.js
    /images
        /logo.png
    /fonts
        /custom-font.woff2
    /favicon.ico
    /robots.txt
    /sitemap.xml
    /service-worker.js
```

When using these assets in your HTML or templates, you can include the JavaScript bundle and any other static files like this:

```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>CMMV App</title>

    <!-- Include Styles -->
    <link rel="stylesheet" href="/assets/styles.css">
    
    <!-- Favicon -->
    <link rel="icon" href="/favicon.ico" type="image/x-icon">
</head>
<body>
    <!-- App Content -->

    <!-- Include the Bundle -->
    <script src="/assets/bundle.min.js"></script>
</body>
</html>
```

In CMMV, managing assets is streamlined by recommending the use of CDNs for static files. However, if serving these assets directly from the application, placing them in the ``/public`` directory will allow ``@cmmv/http`` to handle them automatically. Additionally, the application will generate the necessary frontend bundles for handling reactivity and RPC features, available as ``/assets/bundle.min.js``. All static files are easily accessible from this directory, ensuring a seamless integration with your application.