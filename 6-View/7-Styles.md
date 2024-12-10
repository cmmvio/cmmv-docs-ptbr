# Styles

The new styling feature in ``@cmmv/view`` allows developers to define theme-specific styles that can be dynamically switched on the frontend. The system maps JSON files from the /public/styles directory, which contain the style rules, and makes them accessible in the frontend for easy theme management.

To create a custom style, define a ``.style.json`` file in ``/public/styles``. Each key-value pair represents a style class, and you can create multiple themes by appending a suffix (e.g., .dark) to the keys.

``docs.style.json``

```json
{
    "app": "bg-gray-200",
    "app.dark": "bg-gray-900",
    "title": "ml-2 text-lg text-slate-800 font-semibold",
    "title.dark": "ml-2 text-lg text-gray-200 font-semibold",
    "mainText": "text-slate-800 relative text-white mb-20 context-html",
    "mainText.dark": "text-gray-200 relative text-white mb-20 context-html",
    "sideMenu": "w-60 fixed z-40 overflow-auto text-slate-800 ...",
    "sideMenu.dark": "w-60 fixed z-40 overflow-auto text-white ..."
}
```

## Accessing

In your HTML, you can access the defined styles by referencing the style file and keys. The styles object is automatically made available, and the theme-specific classes are applied based on the current theme.

```html
<div class="w-60">
    <a 
        href="/" 
        title="CMMV - Contract Model Model View Framework" 
        class="text-white ml-4 flex items-center"
    >
        <img 
            src="/assets/favicon/favicon-32x32.png" 
            alt="CMMV Logo" height="32" width="32"
        >
        <span :class="$style.docs.title">CMMV</span>
    </a>
</div>
```

## Switching

The system allows switching between themes (e.g., ``default``, ``dark``). When a theme is changed, all relevant classes with the suffix (e.g., ``.dark``) are applied automatically.

```javascript
toggleTheme() {
    this.$style.switch(
        (this.$style.theme === "default") ? 
        "dark" : "default"
    );
}
```

<br/>

* **JSON File Structure:** The JSON defines style classes for different components. Theme-specific classes are suffixed with ``.dark``, ``.light``, etc.
* **Frontend Access:** Styles are accessed through ``styles.[filename].[key]`` in the frontend.
* **Dynamic Switching:** Theme switching is done programmatically, and the system handles the replacement of style classes based on the active theme.
* **No Subindices Support:** Nested JSON subindices are not supported, meaning that each key-value pair must be a flat entry.

## Management

The theme selection in ``@cmmv/view`` is handled automatically by the framework, saving the user's preference in ``localStorage`` and retrieving it upon page load. This enables the system to maintain consistent styling based on the user's previous choice, without manual intervention.

You can check the current theme directly in your template using ``$style.theme``. For example, to integrate with a component like DocSearch that requires a theme setting, you can update the HTML tag with the ``data-theme`` attribute:

```html
<!DOCTYPE html>
<html lang="en" :data-theme='$style.theme' scope>
    <head>
        <headers/>
    </head>
</html>
```

This ensures that the correct theme is applied across components like search bars and other elements that require theme awareness.

This implementation of dynamic themes in ``@cmmv/view`` provides a streamlined approach to managing multiple themes without needing additional plugins, Tailwind extensions, or complex CSS rules. By utilizing ``.style.json`` files, developers can easily define theme-specific styles, reducing the amount of code and ensuring consistent theme switching. While it’s possible to achieve similar results using CSS variables, this method simplifies the process, eliminating the need for custom view logic or CSS management for theme switching, making it intuitive and efficient.

* **Support for Nested Styles:** Introducing support for nested JSON indices could allow for more complex style structures and better organization.
* **CSS Variable Integration:** An option to integrate with CSS variables can complement the theme system, offering finer control over dynamic theming.
* **Theme Preloading:** Allow theme preloading based on system preferences (e.g., dark mode based on OS settings) to enhance user experience.
* **Advanced Animations:** Adding built-in transitions for theme switching could improve UX when toggling between light and dark modes.

## Component 

In addition to the previously discussed style system, CMMV introduces the ``$style`` property within components, allowing scoped styles to be accessed directly.

This feature enables developers to define and reference styles within the component’s data model, facilitating style management within templates.

```html
<template>
    <div :class="$style.themeSwitch.container">{{ test }}</div>
    <button @click="test++">Add</button>
</template>

<script>
export default {
    data(){
        return {
            test: 123
        }
    }
}
</script>
```

This allows more intuitive style handling, making styles accessible directly within the component context.