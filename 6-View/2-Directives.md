# Directives

Preloading data that will be sent directly in HTML is a crucial factor for SEO (Search Engine Optimization). When a search engine analyzes a page, it searches for content that is already rendered in HTML. By ensuring that the essential data of the page is already loaded in HTML, rendering time is reduced and the content is immediately accessible for indexing, improving visibility in search results.

CMMV (Contract-Model-Model-View) was designed with this need in mind, optimizing content delivery in a way that prioritizes SEO. The system emulates the traditional MVC (Model-View-Controller) model, widely used by well-known frameworks such as Ruby on Rails, Laravel, or Spring. This is done through SSR (Server-Side Rendering) directives, which allow data to be processed on the server and inserted directly into HTML, without the need for another application or overloading the frontend with complex frameworks.

By using SSR directives such as ``s-data``, CMMV ensures that dynamic data is sent in the HTML itself, without relying on AJAX calls or asynchronous loading via JavaScript, which can delay the display of critical content and negatively impact page performance. This makes content instantly available to search engines, improving load time (TTFB) and maximizing the chances of good indexing.

This model not only ensures faster loading, but also improves the end-user experience. The user receives a complete page almost instantly, without the need to wait for additional loading or content updates. By centralizing data processing in the backend, CMMV reduces complexity and frontend overhead, eliminating the need for additional configuration or running multiple applications to render and serve dynamic content.

In short, CMMV is a modern solution that reuses the best of the traditional MVC model, adapting it to the current scenario of SEO and performance optimization, with a focus on simplicity and efficiency.

## s-data

The ``s-data`` directive allows you to directly render server-side data into your HTML templates. This is particularly useful for embedding server-side variables in your HTML without needing additional frontend frameworks or JavaScript.

How it Works
You declare a variable in the server-side controller and pass it to the view. The ``s-data`` directive is responsible for injecting this variable directly into the corresponding HTML element during server-side rendering (SSR). This ensures that the content is pre-rendered, optimizing the page for performance and SEO.

View (HTML Template)
```html
<div s-data="datetime"></div>
```

In this example, the ``s-data`` directive is used on a ``<div>`` element to render the ``datetime`` variable, which will be provided by the server.

Controller
```typescript
res.render("template", { datetime: new Date().getTime() });
```

In the controller, you pass the datetime variable (the current timestamp) as part of the data rendered into the template.

Result (Rendered HTML)
```html
<div>1725568913552</div>
```

## s-attr

The ``s-attr`` directive in CMMV allows you to dynamically assign attributes to HTML elements from server-rendered data, similar to how the ``s-data`` directive works. However, instead of filling in the content of a tag, it adds or updates an attribute of the element. This is particularly useful for adding security attributes like ``nonce`` in ``<script>`` or ``<link>`` tags, often required by strict Content Security Policies (CSP).

View (HTML Template)
```html
<div s-attr="ref">Content</div>
```

Controller
```typescript
res.render("template", { ref: "ABC" });
```

Result (Rendered HTML)
```html
<div ref="ABC">Content</div>
```

In this case, the ``s-attr="ref"`` directive tells the server to create a ref attribute on the ``<div>`` element with the value "ABC", which is passed from the server-side controller.

## s-i18n

The ``s-i18n`` directive is the native internationalization module of the CMMV system. It allows developers to manage and display multilingual content by utilizing locale files. These locale files are stored in the ``/src/locale`` directory and should be in JSON format.

CMMV provides the ability to define a default language, and it allows for dynamic language switching based on the user session. This is ideal for websites and applications that need to cater to multiple languages.

The locale files contain key-value pairs, where the key represents the string identifier and the value is the localized string. For example:

``/src/locale/en.json``
```json
{
    "welcome": "Welcome to CMMV",
    "description": "CMMV makes development faster and easier."
}
```

``/src/locale/pt-br.json``
```json
{
    "welcome": "Bem-vindo ao CMMV",
    "description": "O CMMV torna o desenvolvimento mais rápido e fácil."
}
```

You can use the ``s-i18n`` directive in your view templates to render localized strings based on the current language.

View (HTML Template)
```html
<div s-i18n="welcome"></div>
<p s-i18n="description"></p>
```

Result (For English Locale)
```html
<div>Welcome to CMMV</div>
<p>CMMV makes development faster and easier.</p>
```

**Configuration**

In the ``.cmmv.config.js`` file, you can configure the i18n settings to define the locale file path and the default language for your application:

```typescript
module.exports = {
    i18n: {
        localeFiles: "./src/locale",  // Path to the locale files
        default: "en"  // Default language
    },
    // Other configurations...
}
```

## s-if

The ``s-if`` directive is used to conditionally render a block of content based on the evaluation of an expression. If the provided expression evaluates to true, the content within the ``s-if`` directive will be rendered. Otherwise, the content will not be displayed.

```html
<s-if exp="todolist.length > 0">
    <div>Total Record Loaded SSR: {{ todolist.length }}</div>
    <s-else>
        <div>No records were loaded via SSR</div>
    </s-else>
</s-if> 
```

* ``exp:`` The boolean expression to evaluate. The expression can use variables and logical operators to determine if the block should be displayed.

While the ``s-if`` directive provides a powerful way to conditionally render content, it should be used with caution due to its potential impact on client-side updates. The content inside an s-if block is removed from the DOM if the expression evaluates to false, and this removal is not automatically updated or re-rendered on the client side after the initial server-side rendering (SSR).

**Recommendations:**
* **Use Case:** It is recommended to use s-if primarily for scenarios where server-side conditions determine the content that should be included in the initial HTML. For instance, you might use ``s-if`` to load and display data that is available at the time of server rendering, such as user authentication data or initial configuration settings.

* **Avoid Frequent Client-Side Updates:** Since ``s-if`` removes the content from the DOM based on the initial evaluation, it is not suitable for dynamic client-side content that might change frequently or be updated after the initial load. For content that may need to be dynamically updated based on client-side interactions or data changes, consider using other methods such as client-side rendering techniques or reactive frameworks.

By adhering to these recommendations, you can ensure that your use of the ``s-if`` directive enhances both the performance and maintainability of your application without unintended side effects.

## s-for

The ``s-for`` directive is designed to handle server-side rendering (SSR) of lists and ensure that dynamic data is pre-rendered for improved SEO. This directive, used in conjunction with the ``c-for`` directive, allows you to define how data should be rendered on the server and how it should be updated on the client side. It supports pre-rendering content that will be displayed immediately to users and search engines, while also allowing for client-side updates.

```html
<s-for
    c-show="condition"
    c-for="(item, key) in collection"
    render-tag="tag-name"
>
    <!-- Content to render -->
</s-for>
```

**How It Works**

* The ``s-for`` directive pre-renders the list items on the server based on the data provided. This includes evaluating expressions and injecting static content directly into the HTML.
* The directive ensures that content defined within the ``s-for`` block, including variables rendered using ``{{}}``, ``c-text``, ``c-html``, and ``:``, is pre-rendered.
* On the client side, the ``c-for`` directive will take over and handle dynamic updates. This allows the list to be reactive and updated as needed.

**Input Template:**
```html
<s-for 
    c-show="todolist"
    c-for="(item, key) in todolist"
    class="todo-item"
    render-tag="div"
>
    <div class="todo-item-content">
        <input 
            type="checkbox" 
            c-model="item.checked" 
            @change="UpdateTaskRequest(item)"
        ></input>

        <label 
            :class="{'todo-item-checked': item.checked}"
        >{{ item.label }}</label>
    </div>
    
    <button 
        class="todo-btn-remove"
        s-i18n="remove" 
        @click="DeleteTaskRequest(item.id)"
    ></button>
</s-for> 
```

**Final Rendered Output:**
```html
<div c-if="!loaded && !todolist">
    <div c-show="todolist" class="todo-item">
        <div class="todo-item-content">
            <input 
                type="checkbox" 
                c-model="item.checked" 
                @change="UpdateTaskRequest(1)"
            >
            <label class="todo-item-checked">Task 1</label>
        </div>
        <button 
            class="todo-btn-remove" 
            @click="DeleteTaskRequest(1)"
        >Remove</button>
    </div>
</div>
<div c-else>
    <div 
        c-show="todolist" 
        c-for="(item, key) in todolist" 
        class="todo-item"
    >
        <div class="todo-item-content">
            <input 
                type="checkbox" 
                c-model="item.checked" 
                @change="UpdateTaskRequest(item)"
            >
            <label 
                :class="{'todo-item-checked': item.checked}"
            >{{ item.label }}</label>
        </div>
        <button 
            class="todo-btn-remove" 
            @click="DeleteTaskRequest(item.id)"
        >Remove</button>
    </div>
</div>
```

<br/>

* **Pre-Rendering:** The ``s-for`` directive is crucial for SEO as it pre-renders list items with actual data, making the content immediately available for indexing by search engines.
* **Client-Side Update:** On the client side, the ``c-for`` directive ensures that the content is reactive and can be updated dynamically.
* **Use with Caution:** As ``s-for`` performs server-side rendering, ensure that the list and its contents are suitable for static pre-rendering. Avoid using it for highly dynamic data that changes frequently on the client side.
By combining ``s-for`` with ``c-for``, you can leverage server-side rendering for SEO benefits while maintaining dynamic client-side updates.

## Include 

The ``include`` directive allows the insertion of components or templates from other files into the main layout. This feature is useful for building modular pages, enabling the division of the layout into small, reusable blocks, which optimizes both loading performance and code maintenance.

### How It Works

The ``include`` directive is responsible for preloading components on the server side before sending the page to the client. These components are treated as templates, and their inclusion can occur in a cascading manner, meaning that a template can include other templates. However, this behavior is influenced by the caching strategy, ensuring optimal performance by preventing repeated loading of already processed components.

```html
<div>
    <!-- Include the navbar -->
    <!-- include('public/views/docs/navbar') -->
    
    <!-- Include the footer -->
    <!-- include('public/views/docs/footer') -->
</div>
```

When using the ``include`` directive, it’s important to consider how data or behavior for each template will be managed. The directive is designed to pre-process components on the server side, so any setup logic needs to be carefully planned. The recommended approach is to execute the setup logic in the main template and avoid cascading setups in included templates. This prevents unnecessary multiple setup executions that could negatively impact performance.

To optimize template injection and improve performance, the include directive uses a caching strategy. This means that after the initial loading of a template, its content is cached, and subsequent inclusions will use the already-loaded template. However, if the content needs to be dynamically updated, it’s recommended that the setup be managed in the main template rather than relying on individual setups in each included template. This ensures that changes are centralized and easily managed.

**Complete Example**

```html
<div id="app">
    <!-- include('public/views/docs/navbar') -->

    <div class="content">
        <p>Welcome to our site!</p>
    </div>

    <!-- include('public/views/docs/footer') -->
</div>

<script>
export default {
    data() {
        return {
            navbarState: {},
            contentData: {}
        };
    },
    mounted() {
        this.loadState();
    },
    methods: {
        loadState() {
            this.navbarState = JSON.parse(
                localStorage.getItem('navbarState')
            ) || {};

            this.contentData = { 
                message: 'Dynamically loaded content.' 
            };
        }
    }
};
</script>
```

## Call Services

In this implementation, you have a combination of server-side directives for data preloading and the use of the ``include`` directive to incorporate reusable components, such as the navbar and other layout elements, into the page. While this pattern works efficiently, it currently lacks support for component-specific setups as seen in frameworks like Nuxt.js. Let's break down how this approach works and the potential for future enhancements.

**Server-Side Directive (s:docs)**

In the main HTML file (index.hhtml), you are using a directive (s:docs="docs") to bind server-preloaded data (in this case, docs) to the client-side rendering. This allows the initial data, such as the navigation structure (docs.navbar), to be fetched server-side and passed to the client before the page is rendered.

```html
<div id="app" s:docs="docs" c-cloak>
```

The ``docs`` object is populated on the server, likely from a backend data source or file, and it is injected into the template. This enables the use of this data immediately upon loading without additional client-side API calls, improving the initial loading performance.

**Using the include Directive for Templates**
You are including reusable components, such as the navbar, breadcrumbs, and footer, with the ``include`` directive. These templates are preloaded on the server, enhancing modularity and maintainability:

```html
<!-- include('public/views/docs/navbar') -->
<!-- include('public/views/docs/breadcrumb'); -->
<!-- include('public/views/docs/footer'); -->
```

By loading these components server-side, you avoid multiple HTTP requests to fetch the template separately and enable the server to handle the logic related to content inclusion. However, as you noted, this approach currently processes only the main setup logic, and any additional setup logic within included templates (like navbar.html) is not supported.

### Client-Side State Updates

Once the data and templates are preloaded on the server, the client-side JavaScript is responsible for handling state updates. For example, in your main component, you load the saved state of the navbar from localStorage and provide methods to toggle the state:

```javascript
export default {
    data() {
        return { navbar: [] }
    },

    async mounted() {
        this.loadState();
    },

    methods: {
        loadState() {
            this.navbar = JSON.parse(
                localStorage.getItem('navbarState')
            ) || {};
            
            return this.navbar;
        },

        saveState(state) {
            localStorage.setItem('navbarState', JSON.stringify(state));
        },

        toggle(isOpened, itemName) {
            isOpened = !isOpened;
            const currentState = this.loadState();
            currentState[itemName] = isOpened;
            this.saveState(currentState);
            return isOpened;
        }
    }
};
```

This ensures that interactions on the page, such as expanding or collapsing navbar sections, persist between sessions, as the state is stored in localStorage and reloaded when the page is mounted.

### The Limitation

As noted in the provided comment, there is no current support for individual setup logic within included templates (like ``navbar.html``). This differs from frameworks like Nuxt.js, where each component can have its own setup function. In your current implementation, adding a ``script setup`` block inside an included template would cause issues, as only the setup logic from the main template (``index.html``) is processed.

For example, adding a ``setup`` block in the ``navbar.html`` would not work as expected:

```html
<!-- navbar.html -->
<script s-setup>
export default {
    data() {
        return { isOpened: false };
    },
    methods: {
        toggleNavbar() {
            this.isOpened = !this.isOpened;
        }
    }
};
</script>
```

This would break the application because only the first setup is processed, and additional setups are ignored.

### Future Enhancements

The idea of allowing component-specific setups, similar to Nuxt.js, is a potential enhancement that could improve the flexibility of the system. However, this needs to be handled carefully to avoid performance issues like infinite loading loops. When each component has its own setup logic, the application might end up in a continuous loop of re-rendering if not properly managed.

To safely implement component-specific setup:

* **Careful Dependency Management:** Ensure that each component’s setup logic is independent and doesn't trigger unnecessary re-renders of the parent component.
* **Lazy Loading:** Consider lazy loading for templates or components that don’t need to be preloaded on the server, reducing the initial load and avoiding potential issues with large applications.
* **Optimized State Sharing:** Provide mechanisms to share state across components without requiring full re-execution of setup logic when components are dynamically included.

Here’s how individual setups could work in the future, following a model like Nuxt.js:

```html
<!-- index.html -->
<div id="app" s:docs="docs" c-cloak>
    <div>
        <!-- include('public/views/docs/navbar') -->
    </div>
</div>
```

<br/>

```html
<!-- navbar.html -->
<script s-setup>
export default {
    data() {
        return { isOpened: false };
    },
    methods: {
        toggleNavbar() {
            this.isOpened = !this.isOpened;
        }
    }
};
</script>

<ul>
    <li @click="toggleNavbar">Menu Item</li>
</ul>
```
In this future scenario, each component or template could handle its own state and methods without conflicting with the main template setup.

## Full implementation

Below is an example of implementation of practically all the directives that were used to create the system in this documentation.

``/public/templates/default.html``
```html
<!DOCTYPE html>
<html>
    <head>
        <headers/>
    </head>
    <body scope> 
        <slot/>
        <scripts/>
    </body>
</html>
```

``/public/views/docs/index.html``
```html
<div id="app" s:docs="docs" c-cloak>
    <nav class="navbar bg-neutral-800 h-16 top-0 w-full fixed flex z-50 shadow-lg">
        <div class="max-w-8xl mx-auto flex container">
            <div class="w-60">
                <a href="/" title="UCS.js">
                    <img src="/assets/logo-min-invert.png" class="mt-6 ml-4" />
                </a>
            </div>

            <div class="justify-between w-full text">
                <div class="relative text-right mt-3 hover:cursor-pointer group bg-neutral-800 float-right rounded-lg border border-black">
                    <div class="absolute text-white z-40 top-2 left-18" style="left: 12px; color: #ccd0d5;">
                        <i class="fa-solid fa-search"></i>
                    </div>
                    <div>
                        <input 
                        type="text" 
                            class="p-1.5 pl-10 text-white bg-transparent" 
                            placeholder="Search" 
                        />
                    </div>
                </div>
            </div>

            <div class="justify-between align-middle text-center mr-2 text-white flex">
                <a href="https://github.com/andrehrferreira/cmmv" title="Github" target="_blank" class="text-2xl p-2 mt-2 hover:text-neutral-300">
                    <i class="fa-brands fa-github"></i>
                </a>

                <!--<a href="https://discord.gg/XtUH9sJP" title="Discord" target="_blank" class="text-2xl p-2 mt-2 hover:text-neutral-300">
                    <i class="fa-brands fa-discord"></i>
                </a>-->
            </div>
        </div>
    </nav>

    <div class="max-w-8xl mx-auto flex container">
        <div 
            class="w-60 fixed mt-20 z-40 overflow-auto" 
            style="height: calc(100% - 84px); background-color: #2e3035" 
            c-cloak
        >
            <!-- include('public/views/docs/navbar') -->
        </div>

        <div class="mt-20 ml-64 text-justify relative">
            <div 
                class="lg:pl-[19.5rem] m-4 p-4 px-20 max-w-3x1 mx-auto xl:max-w-none xl:ml-0 xl:mr-[15.5rem] xl:pr-16"
                :class="{'w-full': docs.anchors.length < 4}"
            >
                <!-- include('public/views/docs/breadcrumb'); -->
                <!-- include('public/views/docs/anchors'); -->

                <div class="max-w-screen-lg relative text-white mb-20 context-html">
                    <div c-html="docs.index">{ docs.index }</div>

                    <div class="absolute top-0 right-0">
                        <a 
                            :href="`https://github.com/andrehrferreira/docs.cmmv.io/tree/main${docs.link?.replace('.html', '.md')}?plain=1`" 
                            target="_blank" 
                            title="Suggest change"
                        >
                            <i class="fa-solid fa-pen-to-square fa-lg"></i>
                        </a>
                    </div>
                </div>
            </div>

            <!-- include('public/views/docs/footer'); -->
        </div>
    </div>
</div>

<script s-attr="nonce">
    function updateCurrent(){
        const scrollPosition = window.scrollY;

        document.querySelectorAll('.current').forEach(el => el.classList.remove('current'));

        let repoint = false;
        document.querySelectorAll('#anchors li').forEach((item, index) => {
            const target = document.querySelector(item.querySelector('a').getAttribute('href'));
            
            if (target?.offsetTop >= scrollPosition && !repoint) {
                repoint = true;
                item.classList.add('current');
            }
            else if(!target) {
                repoint = true;
                item.classList.add('current');
            }
        });
    }

    window.addEventListener('scroll', updateCurrent);

    window.addEventListener('DOMContentLoaded', () => {
        document.querySelectorAll('a[href^="#"]').forEach((link, index) => {
            link.addEventListener('click', function(event) {
                event.preventDefault();
                const href = this.getAttribute('href');
                
                document.querySelectorAll('.current').forEach(el => el.classList.remove('current'));
                
                this.parentElement.classList.add('current');
                
                window.scrollTo({
                    top: document.querySelector(href).offsetTop,
                    behavior: 'smooth'
                });

                window.location.hash = href;
            });
        });

        updateCurrent();
    });
</script>

<script s-setup>
export default {
    layout: "default",

    data(){
        return { navbar: [] }
    },

    async mounted() {
        this.loadState();
    },

    methods: {
        loadState(){
            this.navbar = JSON.parse(localStorage.getItem('navbarState')) || {};
            return this.navbar;
        },

        saveState(state) {
            localStorage.setItem('navbarState', JSON.stringify(state));
        },

        toggle(isOpened, itemName) {
            isOpened = !isOpened;

            const currentState = this.loadState();
            currentState[itemName] = isOpened;
            this.saveState(currentState);

            return isOpened;
        }
    }
}
</script>
```

``/public/views/docs/navbar.html``
```html
<ul class="p-4 select-none top-16" c-cloak c-show="docs">
    <li c-for="(item, key) in docs.navbar">
        <div 
            c-show="item"
            class="flex hover:text-blue-700 itemRoot text-white" 
            :id="item?.name.replace(/\s/,'_')" 
            :data-opened="false" 
            @click.stop="navbar[item?.name.replace(/\s/, `_`)] = toggle(navbar[item?.name.replace(/\s/, `_`)], item.name?.replace(/\s/, `_`))"
        >
            <div class="flex flex-1 font-bold text-md cursor-pointer navbar-item">
                <h3 c-if="item && item?.isDir" class="text-white">{{ item?.name }}</h3>
                <span c-else class="text-white">{{ item?.name }}</span>
            </div>

            <div class="justify-between cursor-pointer" c-if="item?.isDir">
                <i class="fa-solid fa-angle-down" c-show="!navbar[item?.name.replace(/\s/, `_`)]"></i>
                <i class="fa-solid fa-angle-up" c-show="navbar[item?.name.replace(/\s/, `_`)]"></i>
            </div>
        </div>

        <ul 
            c-if="item && item.children && item.children.length > 0"
            :id="`${item?.name.replace(/\s/, `_`)}_contents`" 
            class="p-4 py-1 text-md mb-4"
            :style="(navbar[item?.name.replace(/\s/, `_`)]) ? '' : 'display: none;'"
        >
            <li c-for="(child) in item.children">
                <div class="hover:text-gray-800 text-white text-base p-1" style="font-size: 12px">
                    <a :href="child.uri" class="text-base">{{ child.name }}</a>
                </div>
            </li>
        </ul>
    </li>
</ul>
```

``/src/docs.controller.ts``
```typescript
import * as fs from 'fs';
import * as path from "path";

import { 
    Controller, Get, Param, 
    Response, ServiceRegistry 
} from '@cmmv/http';

import { DocsService } from './docs.service';

const index = require("../docs/index.json");

@Controller("docs")
export class DocsController {
    constructor(private docsService: DocsService){}

	@Get()
	async indexHandler(@Response() res) {		
		return res.render("views/docs/index", {
			docs: await this.docsService.getDocsStrutucture(),
			services: ServiceRegistry.getServicesArr()
		});
	}

	@Get(":item")
	async getDocHandler(
        @Param("item") item: string, 
        @Response() res
    ) {
		if(index[item])
			this.getDoc(index[item], res)
		else
			res.status(404).end();
	}

	@Get(":dir/:item")
	async getDocSubdirHandler(
        @Param("dir") dir: string, 
        @Param("item") item: string, 
        @Response() res
    ) {
		const fullPath = `${dir}/${item}`;

		if(index[fullPath])
			this.getDoc(index[fullPath], res)
		else
			res.status(404).end();
	}

	async getDoc(docFilename: string, @Response() res) {
		const file = path.resolve(docFilename);
		const data = await this.docsService.getDocsStrutucture(file);

		return res.render("views/docs/index", {
			docs: data,
			services: ServiceRegistry.getServicesArr()
		});
	}
}
```

To have full access to the code, access [Github](https://github.com/andrehrferreira/docs.cmmv.io)
