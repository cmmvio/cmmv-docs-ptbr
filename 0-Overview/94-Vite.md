# Vite

The CMMV Vite Plugin allows Vite to seamlessly handle ``.cmmv`` files in the same way it processes Vue Single File Components (SFCs). With this plugin, developers can write and use ``.cmmv`` files directly in their projects, leveraging Vite’s fast development environment.

* **GitHub:** [https://github.com/andrehrferreira/vite-plugin-cmmv](https://github.com/andrehrferreira/vite-plugin-cmmv)

## Installation

To use the CMMV Vite Plugin, you can install it via npm:

```bash
$ pnpm add -D @cmmv/plugin-vite vite @vitejs/plugin-vue vue
```

## Configuration

To configure Vite to recognize and process ``.cmmv`` files, follow the simple steps below.

### Step 1: Add the plugin to your Vite configuration

In your ``vite.config.js``, import the plugin and add it to the ``plugins`` array:

```javascript
// vite.config.js
import cmmvPlugin from '@cmmv/plugin-vite';

export default {
  plugins: [cmmvPlugin()],
};
```

This is all you need to configure Vite to interpret ``.cmmv`` files!

### Example Usage

Here's an example ``.cmmv`` file structure. It works similarly to Vue’s Single File Component format:

```html
<template>
	<div>{{ message }}</div>
</template>

<script>
export default {
	data() {
		return {
			message: 'Hello CMMV!'
		};
	}
};
</script>

<style>
div {
	color: blue;
}
</style>
```

The ``.cmmv`` file above consists of three sections: ``template``, ``script``, and ``style``, which are processed in the same way as Vue components.

### Example Project

For a working example of how to use the **CMMV Vite Plugin**, check out the [CMMV Reactivity Project](https://github.com/andrehrferreira/cmmv-reactivity). This project demonstrates how to set up and use the plugin in a real-world scenario, showcasing the power and simplicity of integrating CMMV files into your Vite-powered projects.