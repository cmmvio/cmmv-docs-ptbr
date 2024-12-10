# Views

Views in CMMV are HTML files located in the ``/public/views/`` directory, typically with the .html extension. They can either be rendered by controllers or accessed directly based on the URL path. For example, a request to ``http://localhost:3000/docs`` will automatically map to ``/public/views/docs/index.html``. The ``index.html`` file can also include other views through the include directive, allowing for modular view components.

Here is an example of a view structure using the ``docs`` section of the application.

**``docs/index.html``** [Code](https://github.com/andrehrferreira/docs.cmmv.io/blob/main/public/views/docs/index.html)

```html
<div id="app" s:docs="docs" c-cloak>
    <nav 
        class="navbar bg-neutral-800 h-16 top-0 w-full fixed ..."
    >
        <div class="max-w-8xl mx-auto flex container items-center">
            <button 
                id="menu-toggle" 
                class="text-white text-2xl p-2 lg:hidden ml-2"
            >
                <i class="fa-solid fa-bars"></i>
            </button>

            <div class="w-60">
                <a 
                    href="/" 
                    title="CMMV - Contract Model Model View Framework" 
                    class="text-white ml-4 flex items-center"
                >
                    <img 
                        src="/assets/favicon/favicon-32x32.png" 
                        alt="CMMV Logo" 
                        height="32" 
                        width="32" 
                        class="w-[32px] h-auto"
                    >
                    <span class="ml-2 text-lg font-semibold">CMMV</span>
                </a>
            </div>

            <div class="justify-between w-full text mr-2">
                <div class="relative text-right ...">
                    <div id="docsearch" class="dark"></div>
                </div>
            </div>

            <div class="justify-between align-middle ...">
                <a 
                    href="https://github.com/andrehrferreira/cmmv" 
                    title="Github" 
                    target="_blank" 
                    class="text-2xl p-2 hover:text-neutral-300"
                >
                    <i class="fa-brands fa-github"></i>
                </a>
            </div>
        </div>
    </nav>

    <div class="flex flex-wrap mx-auto">
        <div 
            id="sidebar-menu" 
            class="w-60 fixed z-40 overflow-auto leftbar ..." 
            c-cloak
        >
            <!-- include('public/views/docs/navbar') -->
        </div>
        <div class="mt-20 ml-64 text-justify relative">
             <!-- include('public/views/docs/anchors'); -->

            <div 
                class="lg:pl-[19.5rem] m-4 p-4 px-20 max-w-3x1 mx-auto ..."
                :class="{'w-full': docs.anchors.length < 3}"
            >
                <div class="relative text-white mb-20 context-html">
                    <div c-html="docs.index" s-data="docs.index"></div>

                    <div class="absolute top-0 right-0">
                        <a :href="`https://github.com/...`" 
                           target="_blank" 
                           title="Suggest change"
                        >
                            <i class="fa-solid fa-pen-to-square fa-lg"></i>
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

**``docs/navbar.html``** [Code](https://github.com/andrehrferreira/docs.cmmv.io/blob/main/public/views/docs/navbar.html)
```html
<ul class="p-4 select-none top-16" c-cloak c-show="docs">
    <s-for c-for="(item, key) in docs.navbar" render-tag="li">
        <div c-show="item"
            class="flex hover:text-blue-700 itemRoot text-white"
            :id="item?.name.replace(/\s/,'_')" 
            :data-opened="false"
            @click.stop="navbar[item?.name.replace(/\s/, `_`)] = toggle(...">
            <div class="flex flex-1 font-bold ...">
                <h3 
                    c-if="item && item?.isDir" 
                    class="text-white"
                >{{ item?.name }}</h3>
                <span c-else class="text-white">{{ item?.name }}</span>
            </div>
            <div class="justify-between cursor-pointer" c-if="item?.isDir">
                <i :class="navbar[item?.name...."></i>
            </div>
        </div>
        <ul c-if="item && item.children && item.children.length > 0"
            :id="`${item?.name.replace(/\s/, `_`)}_contents`"
            class="p-4 py-1 text-md mb-4"
            :style="(navbar[item?.name.replace(/\s/, `_`)]) ? '' : ...">
            <li c-for="(child) in item.children">
                <div class="hover:text-..." style="font-size: 12px">
                    <a :href="child.uri" class="text-base">{{ child.name }}</a>
                </div>
            </li>
        </ul>
    </s-for>
</ul>
```

Views are mapped based on the URL path. When a request is made to a specific path like /docs, the system automatically looks for ``/public/views/docs/index.html``. If found, it is rendered; otherwise, a 404 error is returned.

## Include Directive

The ``include`` directive allows the inclusion of other view files, enabling modularity in views. For example, ``docs/index.html`` includes ``docs/navbar.html`` using:

```html
<!-- include('public/views/docs/navbar') -->
```

This approach lets you reuse components like headers, footers, and sidebars across different views.

## Setup and Data Binding

View components can be controlled using the ``s-setup`` directive. However, at the moment, only the ``index.html`` (or root view) can use the ``s-setup`` tag for setting up scripts, meta tags, or other configurations. Any setups added in subcomponents are ignored. This means that data or configurations must be passed from the top-level view.

**Exemple:**

```javascript
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
}
</script>
```

CMMV introduces a concept of setup scripts in the views, similar to what you may find in frameworks like [Vue.js](https://vuejs.org/) and [Nuxt.js](https://nuxt.com/). This concept allows dynamic configuration of headers, structured data, and enables binding data to the frontend. Additionally, setup scripts provide lifecycle hooks, such as ``mounted`` and ``created``, which are executed when the frontend is loaded.

## Dynamic Layout 

The ``layout`` property allows you to specify the layout that the view will use. In this example, the layout is set to ``"default"``, meaning the view will inherit and render within a base layout, often defined in ``/public/templates``.

```javascript
layout: "default"
```

## Lifecycle Hooks

CMMV's setup scripts provide lifecycle hooks similar to Vue.js. These lifecycle hooks allow you to control code execution at different stages of the component's lifecycle:

* **mounted:** Runs when the view is fully mounted on the DOM. Typically used for tasks like DOM manipulation or API requests.
* **created:** Can be used to execute code as soon as the view is created, before it is mounted to the DOM.

```javascript
async mounted() {
    this.loadState();
}
```

## Data Property

You can define the ``data()`` function to return an object that holds reactive data, which can be bound to the view. This data will be automatically updated when modified.

```javascript
data() {
    return { navbar: [] }
}
```

In this example, navbar is initialized as an empty array and is later populated using the ``loadState()`` method.

## Methods

Setup scripts allow the inclusion of methods that are accessible in the view's scope. These methods are incorporated into the framework’s context and can be used within the template or as event handlers for UI interactions.

```javascript
methods: {
    loadState() {
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
```

## Headers

The setup script can also be used to dynamically configure headers and scripts for the view. By using properties like head, you can define meta tags, links (e.g., for stylesheets), and other elements dynamically:

```javascript
head: {
    meta: [
        { name: "description", content: "CMMV Todolist sample" },
        { name: "keywords", content: "cmmv, contract model, websocket" }
    ],
    link: [
        { rel: "stylesheet", href: "/assets/styles/todo.css" },
        { rel: "canonical", href: "https://cmmv.io" },
    ]
}
```

## Data Binding

The setup provides data and methods that are directly accessible for data binding on the frontend. This allows seamless interaction with UI components and dynamic updates. For example:

```html
<div c-html="docs.index" s-data="docs.index"></div>
```

This binds the ``docs.index`` content to the HTML, enabling dynamic rendering based on the state of docs.index.