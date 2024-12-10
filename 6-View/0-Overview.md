# View

The ``@cmmv/view`` module in CMMV is a custom view engine designed to optimize SEO and performance by incorporating server-side rendering (SSR) with seamless integration into modern frontend frameworks. Built on top of EJS (Embedded JavaScript), it serves as a middleware for Express and Fastify, processing views in real-time and injecting pre-loaded data into the HTML before it reaches the browser. This approach enables search engines to index content that has already been processed, while still offering flexibility to use popular frontend frameworks like Vue.js, React, or Angular for additional client-side interaction.

Traditional client-side rendering (CSR) frameworks, such as Vue.js and React, generate content dynamically in the browser. While these frameworks provide rich interactivity, they can negatively impact SEO due to the delayed rendering of content—search engines may not fully index dynamic content that relies on JavaScript execution.

By incorporating SSR through ``@cmmv/view``, the critical content of your web application is rendered on the server, allowing it to be immediately visible to both users and search engines. This significantly enhances the SEO of your site because the content is presented in the initial HTML response, allowing for better crawlability and faster page loads.

## Key Features of @cmmv/view

1. **Server-Side Rendering (SSR)**
The ``@cmmv/view`` module processes views and injects dynamic data server-side before the HTML is sent to the browser. This means that any service calls, database queries, or API requests required to generate content are completed on the server, so the user and search engines receive fully rendered HTML.

2. **Optimized JavaScript Bundle**
In addition to rendering views, the ``@cmmv/view`` module creates an optimized JavaScript bundle that includes the core libraries required by the application, such as ``@cmmv/reactivity`` (a component that handles real-time updates via RPC). This bundle ensures that the frontend is prepared for fast and efficient interaction once it reaches the client.

3. **Seamless Integration with JavaScript Frameworks**
While ``@cmmv/view`` focuses on SSR, it doesn't prevent the use of modern frontend frameworks like Vue.js, React, or Angular. It provides a way to hydrate the server-rendered HTML with these frameworks, so developers can benefit from both SSR and client-side interactivity. In future versions, there will be streamlined support for these frameworks, allowing them to work effortlessly with the existing CMMV infrastructure.

4. **Pre-Loaded Data for SEO**
One of the most significant SEO advantages of ``@cmmv/view`` is its ability to inject pre-loaded data directly into the HTML template. This data is often gathered from services such as APIs, databases, or repositories, making it available for search engine crawlers before any JavaScript needs to be executed. This feature ensures that essential content is available upfront, improving SEO rankings, as search engines can see the full context of the page without relying on client-side scripts.

5. **Dynamic JavaScript Inclusion**
In addition to handling SSR, ``@cmmv/view`` dynamically includes any additional JavaScript files necessary for the page. Whether it’s a custom script or a framework like React, these files are bundled and served in a single request, reducing network overhead and speeding up initial load times.

6. **Custom Directives for SSR**
The ``@cmmv/view`` module also supports custom directives that process HTML and data on the server-side. These directives handle pre-loading data, managing conditional content, looping over datasets, and handling i18n translations—all of which are rendered as static HTML, further improving SEO.

## SEO

**Improved Crawlability**
When search engines crawl your site, they rely on HTML to extract key information for indexing. In a typical CSR application, much of the content is loaded asynchronously via JavaScript, which search engines may not execute fully, or at all. This results in poor visibility for crawlers, reducing the discoverability of critical content.

With ``@cmmv/view``, the content is already pre-rendered by the time it reaches the browser. The server completes the processing of all service requests, API calls, and dynamic data queries, delivering fully rendered HTML. As a result, search engines can immediately identify the core content of your pages, leading to higher rankings in search results.

**Faster Page Load Times**
Since ``@cmmv/view`` processes data on the server and sends it along with the initial HTML, users experience faster page loads. The time spent rendering content in the browser is minimized, which not only improves the user experience but also enhances the page's SEO performance.

**JavaScript Overhead Reduction**
Client-side rendering frameworks often involve large JavaScript bundles and significant runtime execution. With ``@cmmv/view``, much of the work is offloaded to the server, reducing the amount of JavaScript required to process the page on the client-side. This results in reduced network load, quicker time-to-interactive (TTI), and a smoother overall experience for the user.

**Pre-Loaded Data for SEO and UX**
By pre-fetching data and injecting it into the HTML, ``@cmmv/view`` ensures that users and search engines see the most relevant content immediately. Whether it’s product information, blog content, or news articles, everything is available as soon as the page loads, improving both SEO and user engagement.

## Exemple

```html
<div 
    class="product-page" 
    s:product="services.product.getProductById(productId)" 
    scope
>
    <h1>{{ product.name }}</h1>
    <p>{{ product.description }}</p>

    <div class="price">{{ product.price | currency }}</div>

    <div class="availability">
        {{ product.inStock ? 'In Stock' : 'Out of Stock' }}
    </div>

    <button 
        s-i18n="addToCart"
        class="add-to-cart" 
        @click="addToCart(product.id)" 
    ></button>
</div>

<script s-setup>
export default {
    layout: "default",
    
    data() {
        return {
            product: null
        }
    },
    
    methods: {
        addToCart(productId) {
            ...
        }
    }
}
</script>
```

By using ``@cmmv/view``, developers can leverage the advantages of SSR to significantly boost their SEO performance and reduce load times. By pre-loading content, optimizing JavaScript bundles, and offering seamless integration with modern frontend frameworks, ``@cmmv/view`` provides a comprehensive solution for building scalable, SEO-friendly web applications.

The ability to serve pre-rendered content, while maintaining the flexibility of client-side interactivity, ensures that CMMV applications can deliver the best of both worlds: high SEO performance and rich, responsive user experiences.

## Settings

The ``@cmmv/view`` module in CMMV allows for flexible customization through the ``.cmmv.config.js`` file. Below are the available configurations that you can use to fine-tune the behavior of the view engine, internationalization (i18n), meta tags for SEO, security headers, and JavaScript resources.

```typescript
module.exports = {
    ...

    i18n: {
        localeFiles: "./src/locale",
        default: "en"
    },

    view: {
        extractInlineScript: true,
        minifyHTML: true
    },

    head: {
        title: "CMMV",
        htmlAttrs: {
            lang: "pt-br"
        },
        meta: [
            { charset: 'utf-8' },
            { name: 'viewport', content: 'width=device-width' },
            ...
        ],
        link: [
            { rel: 'icon', href: 'assets/favicon/favicon.ico' }, 
            ...
        ]
    },

    headers: {
        "Content-Security-Policy": [
            "default-src 'self'",
            "script-src 'self' 'unsafe-eval'",
            "style-src 'self' 'unsafe-inline'",
            "font-src 'self'"
        ],
        ...
    },

    scripts: [
        { type: "text/javascript", src: '/assets/bundle.min.js' }
    ]
}
```

## I18n Configuration

The i18n configuration allows you to manage internationalization (i18n) in your CMMV application by defining the directory for locale files and setting a default language.

```typescript
i18n: {
    localeFiles: "./src/locale", 
    default: "en" 
}
```

**localeFiles:** Directory where the translation files for different languages are stored.
**default:** The default language to be used when no specific locale is selected.

## View Configuration

The view configuration lets you control how the HTML output is processed. It includes options for extracting inline scripts and minifying HTML for optimized performance.

```typescript
view: {
    extractInlineScript: true, 
    minifyHTML: true  
}
```

## Head Configuration

The head configuration controls the ``<head>`` section of your HTML document, allowing you to define meta tags, attributes for the ``<html>`` tag, and links such as favicons.

```typescript
head: {
    title: "CMMV",  
    htmlAttrs: { lang: "pt-br" },
    meta: [
        { charset: 'utf-8' },
        { name: 'viewport', content: 'width=device-width' }
        ...
    ],
    link: [
        { rel: 'icon', href: 'assets/favicon/favicon.ico' }
        ...
    ]
}
```

## Headers Configuration

The ``headers`` configuration allows you to define HTTP headers such as ``Content-Security-Policy`` (CSP), which help secure your application.

```typescript
headers: {
    "Content-Security-Policy": [
        "default-src 'self'", 
        "script-src 'self' 'unsafe-eval'",  
        "style-src 'self' 'unsafe-inline'",  
        "font-src 'self'" 
    ]
}
```

### Scripts Configuration

The ``scripts`` configuration specifies external or internal JavaScript files that should be included in the HTML output.

```typescript
scripts: [
    { type: "text/javascript", src: '/assets/bundle.min.js', defer: "defer" }
]
```