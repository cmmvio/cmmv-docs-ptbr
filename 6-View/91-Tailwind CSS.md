# Tailwind CSS

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
        Support for <code>TailwindCSS</code> integration was removed from the core module in version <strong>0.7.5</strong>. 
        It is now recommended to use <code>Vite</code> for managing frontend assets and bundling. Follow the steps below 
        to configure TailwindCSS with Vite in your CMMV project.
    </p>
</div>

Follow these steps to integrate [TailwindCSS](https://tailwindcss.com/) into your CMMV project using Vite for modern and optimized development workflows:

## Install TailwindCSS

Run the following command to add TailwindCSS as a development dependency:

```bash
pnpm add -D tailwindcss postcss autoprefixer
```

## Initialize TailwindCSS

Generate the TailwindCSS configuration files by running:

```bash
npx tailwindcss init
```

This will create a `tailwind.config.js` file at the root of your project.

## Configure TailwindCSS

Update your `tailwind.config.js` file to include the paths for your Vue or other frontend components:

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
    content: [
        './index.html',
        './src/**/*.{vue,js,ts,jsx,tsx}',
    ],
    darkMode: 'class', // Enable class-based dark mode
    theme: {
        extend: {},
    },
    plugins: [],
};
```

## Create Entry CSS File

Create a `src/tailwind.css` file in your project and include the TailwindCSS directives:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Configure Vite

Modify your `vite.config.js` to ensure that TailwindCSS is processed correctly:

```javascript
import { defineConfig } from 'vite';
import vue from '@vitejs/plugin-vue';
import path from 'path';

// https://vitejs.dev/config/
export default defineConfig({
    plugins: [vue()],
    css: {
        postcss: {
            plugins: [
                require('tailwindcss'),
                require('autoprefixer'),
            ],
        },
    },
    resolve: {
        alias: {
            '@': path.resolve(__dirname, './src'),
        },
    },
});
```

## Use TailwindCSS in Your Project

Ensure the entry CSS file (`tailwind.css`) is imported into your application. You can do this by importing it in your `main.js`:

```javascript
import { createApp } from 'vue';
import App from './App.vue';
import './tailwind.css'; // Import TailwindCSS

createApp(App).mount('#app');
```

## HTML Template

Ensure your `index.html` is configured to include the `#app` element where Vue will mount:

```html
<!DOCTYPE html>
<html lang="en">
    <head>
        <meta charset="UTF-8" />
        <meta name="viewport" content="width=device-width, initial-scale=1.0" />
        <title>CMMV + TailwindCSS</title>
    </head>
    <body class="bg-gray-50 text-gray-800 dark:bg-gray-900 dark:text-gray-100">
        <div id="app"></div>
        <script type="module" src="/src/main.js"></script>
    </body>
</html>
```

## Recommendations

- Use the `@cmmv/vue` module for generating RPC mixins and composables for Vue integration.
- Configure Vite as your primary build tool to handle modern frontend workflows.
- Follow TailwindCSS best practices for creating reusable and scalable styles.

This setup ensures a clean separation of concerns, allowing CMMV to focus on backend operations while Vite handles the frontend assets. 