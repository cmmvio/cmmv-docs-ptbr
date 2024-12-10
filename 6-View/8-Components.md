# Components

The CMMV framework introduces a simple yet powerful way to integrate server-side rendered (SSR) components, leveraging a syntax similar to Vue.js. With the ``@cmmv/view`` module, components can be created in the ``/public`` directory and dynamically imported and used in views.

This document outlines the full process of setting up SSR components using CMMV, with examples of how to structure templates, styles, scripts, and integration with views.

## Example

```html
<div scope>
  <ComponentTeste ref="componentTest" name="Test"></ComponentTeste>
</div>

<script s-setup>
import ComponentTeste from "@components/component.cmmv";

export default {
    layout: "default",

    components: { ComponentTeste },

    data() { 
        return { 
            test: 123 
        }
    },

    methods: { 
        addTask() { /*...*/ } 
    }
}
</script>
```

``/public/components/component.cmmv``

```html
<template>
  <div>{{ test }}</div>
  <button @click="test++">Add</button>
</template>

<script>
export default {
    data() { 
        return { 
            test: 123 
        }
    },

    mounted() { 
        console.log("Component mounted"); 
    }
}
</script>
```

The component syntax is based on Vue.js, supporting lifecycle methods (``created``, ``mounted``), reactive ``data``, ``props``, and ``methods``. The component can also use scoped styles with SSR capabilities.

For a complete example, see [CMMV Reactivity Samples](https://github.com/andrehrferreira/cmmv-reactivity/blob/main/samples/componentTemplate.cmmv).

## Data 

In CMMV components, ``data`` is a key feature used to define reactive state within the component. The ``data`` function returns an object that holds reactive properties, allowing the component to dynamically update the DOM when the data changes.

```html
<template>
    <div>{{ message }}</div>
</template>

<script>
export default {
    data() {
        return {
            message: "Hello, World!"
        };
    }
};
</script>
```

<br/>

When properties defined in data are modified, the DOM updates to reflect those changes automatically. This provides a seamless, reactive user experience similar to Vue.js.

## Props

Components can accept ``props`` to make them reusable and flexible:

```html
<div scope>
    <ComponentTemplate ref="comp" :count="test"></ComponentTemplate>

    <hr/>
    
    <div>
        Root: {{ test }}<br/>
        Component Ctx: {{ $refs.comp.count }}
    </div>
</div>

<script type="module">
import { createApp } from '../src';
import ComponentTemplate from './componentTemplate.cmmv';

createApp({
    components: { ComponentTemplate },

    data(){
        return {
            test: 123
        }
    }
}).mount();
</script>
```

<br/>

``/samples/componentTemplate.cmmv`` [Github](https://github.com/andrehrferreira/cmmv-reactivity/blob/main/samples/componentTemplate.cmmv)

```html
<template>
    <div>{{ count }}</div>

    <button @click="addCount()" class="btnAdd">Add</button>
</template>

<style scoped>
.btnAdd{
    border: 1px solid #CCC;
}
</style>

<script>
export default {
    props: {
        count: {
            type: Number,
            defaultValue: 0
        }
    },

    data(){
        return {
            teste: ""
        }
    },

    created(){
        //console.log("created")
    },

    mounted(){
        //console.log("mounted")
    },

    methods: {
        addCount(){
            this.count++;
            this.emit("count", this.count)
        }
    }
}
</script>
```

<br/>

## Methods

In CMMV components, methods are essential for handling user interactions and performing dynamic logic. Methods are defined in the ``script`` block of a component and can be used for a wide variety of purposes, such as updating data, making HTTP requests, and triggering other component actions.

To define a method in a CMMV component, you include a ``methods`` property in the default export. Methods can be directly called from the template using event bindings (e.g., ``@click``).

```html
<template>
  <div>
    <button @click="incrementCount">Add</button>
    <p>Count: {{ count }}</p>
  </div>
</template>

<script>
export default {
  data() {
    return {
      count: 0
    };
  },
  methods: {
    incrementCount() {
      this.count++;
    }
  }
};
</script>
```

## Created

The ``created`` lifecycle hook in CMMV is executed after the component's instance is created but before it is mounted to the DOM. This is a useful hook for performing logic such as data fetching, initializing variables, or triggering certain actions before the template is rendered.

```typescript
export default {
  data() {
    return {
      message: "Initial Message"
    };
  },
  created() {
    console.log("Component has been created!");
    this.fetchData();
  },
  methods: {
    fetchData() {
      this.message = "Updated Message!";
    }
  }
}
```

The ``created`` hook provides an early entry point to execute logic before the template is rendered, making it useful for initializing data or state in a component.

## Mounted

The ``mounted`` lifecycle hook is executed after the component has been inserted into the DOM. This hook is ideal for actions that require direct interaction with the rendered DOM or for starting processes that rely on the component being present in the document.

```typescript
export default {
  data() {
    return {
      count: 0
    };
  },
  mounted() {
    console.log("Component has been mounted!");
    this.initializeCounter();
  },
  methods: {
    initializeCounter() {
      // Interact with DOM or start operations
      setInterval(() => {
        this.count++;
      }, 1000);
    }
  }
}
```

The ``mounted`` hook is essential when you need to ensure that your component is fully loaded into the DOM before interacting with it, making it useful for DOM manipulations, setting up event listeners, or initializing components that rely on third-party libraries or services.

# Slot

Slots in CMMV allow you to pass custom content from the parent scope into the child component. They can be dynamically updated using data from the parent component.

```html
<ComponentTemplate ref="comp" :count="test">
    <template c-slot="{ count }">
        Component value: {{ count }}
    </template>
</ComponentTemplate>
```

In this example, the ``c-slot`` directive is used to pass the slot content, making the parent’s ``count`` data available inside the ``ComponentTemplate``. The parent scope can dynamically change the slot content, reflecting the updated values inside the child component.

The slot in the component is rendered like this:

```html
<slot :count="count"></slot>
```

**Rendering**

```html
<div ref="comp" count="123">
  Component value: 123
</div>
```

<br/>

* **Named Slots:** Slots can be given names for better organization.
* **Reactivity:** Slots update reactively with changes in parent data.
* **Scoped Slots:** Scoped slots provide the ability to pass data from the child component to the parent for rendering custom content.