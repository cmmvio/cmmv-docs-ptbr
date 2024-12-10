# Vue 3

<div style="
    background-color: #FEF3C7; 
    border-left: 4px solid #F59E0B; 
    color: #92400E; 
    padding: 1rem; 
    border-radius: 0.375rem; 
    margin: 1.5rem 0;
">
    <p style="font-weight: bold; margin-bottom: 0.5rem;">Notice</p>
    <p>
        The support for <code>Vue</code> and <code>TailwindCSS</code> was removed from the core module in version <strong>0.7.5</strong>. 
        It is now recommended to use <code>Vite</code> for handling frontend assets and bundling. 
        Additionally, you can use the <code>@cmmv/vue</code> module to transpile the RPC mixins and composables required for Vue integration.
    </p>
</div>

The `@cmmv/vue` module provides seamless integration with [Vue 3](https://vuejs.org/) for building rich, reactive interfaces in your applications. This module enables you to leverage Vue's ecosystem while integrating deeply with CMMV's REST or RPC communication capabilities. The responsibility for generating and serving Vue 3 composables is fully handled by the `@cmmv/vue` module, promoting the use of modern frontend tooling like Vite for asset handling and efficient communication with the backend.

## Features

- **Vue 3 Integration:** Full support for Vue 3, including its reactive system, composables, and templating syntax.
- **Composable RPC API:** Automatically generate composables for seamless communication with CMMV backend via REST or RPC using the `useRPC` hook.
- **Frontend-First Architecture:** Encourages modern tooling like Vite to handle assets and development workflows.
- **Flexible Communication:** Use CMMV as a proxy to facilitate REST or RPC communication in Vue applications.

## Installation

To install the `@cmmv/vue` module:

```bash
$ pnpm add @cmmv/vue vue
```

Ensure you have `vite` installed for asset management:

```bash
$ pnpm add -D vite
```

## Configuration

Update your `.cmmv.config.js` to include the `@cmmv/vue` module. Remove any direct Vue 3 or TailwindCSS configurations from the `view` section, as those responsibilities now lie with the frontend tooling.

```javascript
module.exports = {
    env: process.env.NODE_ENV,

    vue: {
        composableEndpoint: '/assets/rpc-composable.min.js', // Endpoint for generated composables
    },
};
```

## Setting Up Your Application

Include the `VueModule` in your application configuration. The module will handle generating composables for communication with the backend.

```typescript
import { Application } from '@cmmv/core';
import { DefaultAdapter, DefaultHTTPModule } from '@cmmv/http';
import { VueModule } from '@cmmv/vue';

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [
        DefaultHTTPModule,
        VueModule,
    ],
});
```

## Vue 2

The `@cmmv/vue` module also supports Vue 2 through the use of mixins, enabling smooth integration with CMMV's backend services. For applications still using Vue 2, mixins are automatically generated and accessible via the `rpc-mixins.min.js` file.

### Using Mixins in Vue 2

To integrate with Vue 2, dynamically import the mixins and include them in your Vue instance.

**HTML Template**

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Vue 2 RPC Integration</title>
        <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.min.js"></script>
    </head>
    <body>
        <div id="app">
            <h1>{{ example }}</h1>
            <button @click="fetchData">Fetch Data</button>
        </div>
        <script type="module" src="/main.js"></script>
    </body>
</html>
```

**main.js**

```javascript
import Vue from 'vue';

const initApp = async () => {
    const { default: CMMVMixin } = 
        await import('/assets/rpc-mixins.min.js');

    new Vue({
        el: '#app',
        mixins: [CMMVMixin],
        data: {
            example: "Hello, Vue 2 with RPC!",
        },
        methods: {
            async fetchData() {
                const data = await this.GetAllTaskRequest();
                console.log("Fetched data:", data);
            },
        },
        mounted() {
            console.log("App mounted with example:", this.example);
        },
    });
};

initApp();
```

- **Mixins**: Vue 2 relies on mixins instead of composables for injecting RPC methods into components.
- **Lifecycle Hooks**: Vue 2 uses `mounted` and other instance-based lifecycle hooks to manage initialization and state updates.
- **Reactive Data**: Data is managed through the `data` function and is bound to the DOM through Vue 2's reactivity system.

### Recommendations for Vue 2 Users
<br/>

- Leverage mixins for seamless integration with CMMV backend services.
- Use `rpc-mixins.min.js` to simplify backend communication without requiring manual RPC configuration.
- Consider migrating to Vue 3 for access to the latest features, including composables and improved reactivity.

## Vue 3

The `@cmmv/vue` module generates an `rpc-composable.min.js` file that can be used in your Vue application to integrate backend RPC calls via the `useRPC` composable.

### Example Vue 3 Application

Install Vue:

```bash
$ pnpm add vue
```

**main.js**

```javascript
import { createApp } from 'vue';

const initApp = async () => {
    const { useRPC } = 
        await import('/assets/rpc-composable.min.js');
    
    const app = createApp({
        setup() {
            const rpc = useRPC();
            const example = ref("Hello, Vue with RPC!");

            const fetchData = async () => {
                const data = await rpc.GetAllTaskRequest();
                console.log("Fetched data:", data);
            };

            return {
                example,
                fetchData,
            };
        },
    });

    app.mount('#app');
};

initApp();
```

**HTML Template**

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <title>Vue RPC Integration</title>
        <script src="https://unpkg.com/vue@3.5.12/dist/vue.global.prod.js"></script>
    </head>
    <body>
        <div id="app">
            <h1>{{ example }}</h1>
            <button @click="fetchData">Fetch Data</button>
        </div>
        <script type="module" src="/main.js"></script>
    </body>
</html>
```

## Nuxt

For Nuxt applications, dynamically import the composables into your client-side plugins.

### Installing Nuxt

```bash
npx nuxi init nuxt-rpc
cd nuxt-rpc
pnpm install
```

### Adding CMMV Composables Plugin

**plugins/cmmv-composables.client.ts**

```javascript
export default defineNuxtPlugin(async () => {
    const { useRPC } = 
        await import('http://localhost:3000/assets/rpc-composable.min.js');

    return {
        provide: {
            useRPC,
        },
    };
});
```

### Using Composables in a Page

**pages/index.vue**

```html
<template>
    <div>
        <h1>{{ example }}</h1>
        <button @click="fetchData">Fetch Data</button>
    </div>
</template>

<script setup>
import { useNuxtApp } from '#app';

const { useRPC } = useNuxtApp();
const rpc = useRPC();

const example = ref("Hello, Nuxt with RPC!");

const fetchData = async () => {
    const data = await rpc.GetAllTaskRequest();
    console.log("Fetched data:", data);
};
</script>
```

## Recommendations

- Use `vite` for development and production builds.
- Utilize `@cmmv/vue` composables for efficient communication with CMMV backend services.
- Follow best practices for Vue 3 development, leveraging reactivity and composables to simplify your application architecture.
