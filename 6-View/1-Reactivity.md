# Reactivity

The `@cmmv/reactivity` module was created based on the core ideas of [Petite Vue](https://github.com/vuejs/petite-vue), a simplified version of Vue.js. Petite Vue, developed by Evan You, the creator of Vue.js, offers a lightweight alternative to the full Vue framework, providing only the essential reactivity and templating features needed for smaller or simpler projects. Petite Vue is just ``6KB`` in size and implements basic directives, making it an excellent foundation for building minimal reactive systems.

Inspired by Petite Vue's design philosophy, `@cmmv/reactivity` integrates this lightweight, reactive approach into the CMMV framework. By doing so, it offers developers the ability to use two-way data binding and efficient UI updates with minimal overhead. The core reactivity engine from Petite Vue has been adapted to fit seamlessly into the CMMV ecosystem, allowing for easy integration with server-side rendering, WebSockets, Protobuf, and other key modules of CMMV.

With this combination, `@cmmv/reactivity` provides an optimal balance of simplicity and performance, making it ideal for building dynamic user interfaces while keeping the overall bundle size minimal. It is well-suited for developers who prefer a lightweight, Vue-inspired reactivity system without the complexity of a full-featured framework.

For more information on Petite Vue, you can visit the official [Petite Vue repository](https://github.com/vuejs/petite-vue).

## Limitations

The primary goal of the CMMV framework is to build high-performance applications with a strong focus on speed, efficiency, and SEO optimization. To achieve this, we highly recommend using ``@cmmv/reactivity`` as the frontend framework rather than incorporating heavier frameworks like Vue, React, or Angular. @cmmv/reactivity is designed to handle most of the common challenges faced in web applications, providing essential reactivity features in a lightweight package that minimizes the impact on your application's load times and performance.

By choosing ``@cmmv/reactivity``, you can ensure that your application remains fast and optimized for both users and search engines. Adding larger frameworks such as Vue, React, or Angular introduces additional layers of JavaScript that can increase the page's initial load time, potentially harming your PageSpeed score and overall performance. This goes against the core principles of CMMV, which aims to reduce unnecessary overhead.

If you find that certain critical features are missing from ``@cmmv/reactivity``, we encourage you to submit a pull request to the repository. We're open to suggestions and contributions that improve the framework while keeping it aligned with CMMV's performance goals.

In extreme cases where heavier frameworks like Vue, React, or Angular are necessary, we will eventually provide native support for their integration. However, we strongly advise against this approach due to the performance trade-offs. Any implementation of these frameworks will be at your own risk, and we will not offer official support for such setups—our focus will remain exclusively on optimizing ``@cmmv/reactivity`` for the best possible user experience and SEO results.

## Installation

To install the ``@cmmv/reactivity`` package, simply run the following command:

```bash
$ pnpm add @cmmv/reactivity
```

Alternatively, you can visit the [GitHub repository](https://github.com/andrehrferreira/cmmv-reactivity) for more details.

The ``@cmmv/view`` module already integrates reactivity natively, so it does not need to be installed separately.

## Usage

Below is the documentation for all directives supported by ``@cmmv/reactivity`` based on the samples provided in the CMMV Reactivity GitHub repository.

## c-model 

Binds the value of an input element to the application's data and enables two-way data binding. This is useful for dynamically updating the UI as users input data.

```html
<input c-model="username">
<p>Username: {{ username }}</p>
```

[Sample](https://github.com/andrehrferreira/cmmv-reactivity/blob/main/samples/model.html) : 
```html
<script type="module">
  import { createApp } from '../src'

  createApp().mount('#app')
</script>

<div
  id="app"
  scope="{
    text: 'hello',
    checked: true,
    checkToggle: { a: 1 },
    trueValue: { a: 1 },
    falseValue: { a: 2 },
    arr: ['one'],
    radioSelected: 'two',
    selected: 'two'
  }"
>
  <pre>{{ $data }}</pre>
  <h2>Text Input</h2>
  {{ text }}
  <input v-model.trim="text" />

  <h2>TextArea</h2>
  {{ text }}
  <textarea v-model.trim="text"></textarea>

  <h2>Checkbox</h2>
  <input type="checkbox" id="checkbox" v-model="checked" />
  <label for="checkbox">{{ checked }}</label>

  <h2>Checkbox w/ Array</h2>
  <label><input type="checkbox" v-model="arr" value="one" /> one</label>
  <label><input type="checkbox" v-model="arr" value="two" /> two</label>
  <label
    ><input type="checkbox" v-model="arr" :value="123" /> actual number</label
  >
  <div>{{ arr }}</div>

  <h2>Checkbox w/ true-value / false-value</h2>
  <input
    type="checkbox"
    v-model="checkToggle"
    :true-value="trueValue"
    :false-value="falseValue"
  />
  <div>{{ checkToggle }}</div>

  <h2>Radio</h2>
  <label><input type="radio" v-model="radioSelected" value="one" /> one</label>
  <label><input type="radio" v-model="radioSelected" value="two" /> two</label>
  <label
    ><input type="radio" v-model="radioSelected" value="three" /> three</label
  >
  <div>{{ radioSelected }}</div>

  <h2>Select</h2>
  <select v-model="selected" @change="console.log(selected, $event.target.value)">
    <option>one</option>
    <option>two</option>
    <option>three</option>
  </select>
  <div>{{ selected }}</div>
</div>
```

In this example, the input's value is bound to the username field in the data model. Changes in the input will be reflected in the username property.

## c-show

Controls the visibility of elements based on a condition. If the condition evaluates to false, the element will be hidden.

```html
<p c-show="isLoggedIn">Welcome back, user!</p>
```

## c-if

Conditionally renders an element only if the specified expression is true. Unlike c-show, this directive removes the element from the DOM if the condition is false.

```html
<p c-if="showMessage">This message is shown only if showMessage is true.</p>
```

[Sample](https://github.com/andrehrferreira/cmmv-reactivity/blob/main/samples/if.html):
```html
<script type="module">
  import { createApp } from '../src'
  createApp().mount('#app')
</script>

<div id="app" scope="{ open: true, elseOpen: true }">
  <button @click="open = !open">toggle</button>
  <button @click="elseOpen = !elseOpen">toggle else</button>
  <div c-if="open">ok</div>
  <div c-else-if="elseOpen">else if</div>
  <template c-else>else</template>
</div>
```

## c-for

Loops through an array or object and repeats the associated element for each item.

```html
<ul>
  <li c-for="item in items">{{ item }}</li>
</ul>
```

[Sample](https://github.com/andrehrferreira/cmmv-reactivity/blob/main/samples/for.html): 
```html
<script type="module">
    import { createApp } from '../src'
  
    let id = 4
    createApp({
      list: [
        { id: 1, text: 'bar' },
        { id: 2, text: 'boo' },
        { id: 3, text: 'baz' },
        { id: 4, text: 'bazz' }
      ],
      add() {
        this.list.push({ id: ++id, text: 'new item' });
      },
      splice() {
        this.list.splice(1, 0, { id: ++id, text: 'new item' })
      }
    }).mount('#app')
</script>
  
<div id="app" scope>
    <button @click="add">add</button>
    <button @click="list.reverse()">reverse</button>
    <button @click="list.pop()">pop</button>
    <button @click="splice">splice</button>
    <ul>
      <li c-for="({ id, text }, index) in list" :key="id">
        <div>{{ index }} {{ { id, text } }}</div>
      </li>
    </ul>
  
    <ul>
      <li c-for="item of list" :key="item.id">
        <input c-model="item.text" />
      </li>
    </ul>
</div>
```

## c-on

Attaches an event listener to an element. Commonly used for handling click events, form submissions, or other user interactions.

```html
<button c-on:click="incrementCounter">Click me!</button>
<button @click="incrementCounter">Click me!</button>
```

[Sample](https://github.com/andrehrferreira/cmmv-reactivity/blob/main/samples/on.html): 
```html
<script type="module">
  import { createApp } from '../src'
  createApp().mount('#app')
</script>

<div id="app">
	<input
		type="text"
		@keyup.x="alert('yo')"
		placeholder="type x to test key modifier"
	/>
	<form>
		<button type="submit" @click.prevent.stop>submit (prevented)</button>
	</form>
	<button @click.right="alert('clicked')">right click</button>
	<button @click.middle="alert('clicked')">middle click</button>
	<button @click.once="alert('clicked')">click once</button>
</div>
```

## c-bind

Dynamically binds an attribute to an expression. This is often used to modify attributes like src, href, or class.

```html
<img :src="imageSource">
```

[Sample](https://github.com/andrehrferreira/cmmv-reactivity/blob/main/samples/bind.html):
```html
<style>
    #green {
        color: green;
    }

    .red {
        color: red;
    }

    .orange {
        color: orange;
    }

    .static {
        font-weight: bold;
    }
</style>
  
<script type="module">
    import { createApp, reactive } from '../src'

    const data = (window.data = reactive({
        id: 'green',
        classes: ['foo', { red: true }],
        style: { color: 'blue' },
        obj: { class: 'orange' }
    }))

    createApp(data).mount()
</script>
  
<div scope>
    <div :id="id">simple binding - this should be green</div>

    <div class="static" :class="classes">
        class binding - this should be red and bold
    </div>

    <div style="font-weight: bold" :style="style">
        style binding - this should be blue and bold
    </div>

    <div c-bind="obj">object binding - this should be orange</div>
</div>
```

## c-text

Dynamically sets the text content of an element.

```html
<div scope="{ count: 1 }">
	<p c-text="count"></p>
	<button @click="count++">increase</button>
</div>
```

## c-html

Inserts raw HTML content into an element.

```html
<div c-html="htmlContent"></div>
```

## c-class

Dynamically binds one or more CSS classes to an element.

```html
<div c-class="{ active: isActive, disabled: isDisabled }"></div>
```

In this example, if isActive is true, the active class is added, and if isDisabled is true, the disabled class is added.

## c-once

```html
<script type="module">
  import { createApp } from '../src'
  createApp().mount()
</script>

<div scope="{ count: 5 }">
    {{ count }}
    <div c-once>
      <h2>Once</h2>
      {{ count }}
      <span c-text="count"></span>
      <span v-for="i in count">{{ i }}</span>
    </div>
    <span c-text="count"></span>
    <button @click="count++">++</button>
</div>
```

## ref

In the @cmmv/reactivity framework, the ref system allows you to easily reference DOM elements within the template and manipulate them in your JavaScript logic. This concept is crucial for managing DOM elements directly when reactive bindings alone are not enough. By associating a ref with an element, you can interact with the DOM directly from the reactive context of your application.

Key Benefits of Using ref:
Direct DOM Access: It allows you to interact directly with DOM elements, which is sometimes necessary for operations that cannot be purely reactive.
Dynamic Ref Assignment: You can dynamically change the target element assigned to a ref at runtime.
Scoped Refs: The system supports nested scopes, meaning you can define refs within a specific context without affecting the global references.

```html
<script type="module">
  import { createApp, reactive } from '../src'
  createApp().mount()
</script>

<div
  id="root"
  ref="root"
  scope="{ dynamicRef: 'x', show: true }"
  c-effect="console.log({ x: $refs.x, y: $refs.y, input: $refs.input })"
>
	<p>Accessing root el: id is {{ $refs.root.id }}</p>

	<input ref="input" />

	<span v-show="show" :ref="dynamicRef">Span with dynamic ref</span>

	<p>dynamicRef is {{ dynamicRef }}</p>

	<button @click="dynamicRef = dynamicRef === 'x' ? 'y' : 'x'">
		change dynamicRef
	</button>

	<button @click="show = !show">toggle</button>

	<div scope>
		<p ref="x">nested scope ref</p>
		<button
		@click="console.log({ x: $refs.x, y: $refs.y, input: $refs.input })"
		>
		log nested scope refs
		</button>
	</div>
</div>
```

* **Static Ref Assignment:** The ref attribute binds an element to a key in the $refs object. In the example above, the element ``input ref="input"`` is now accessible via $refs.input in your JavaScript code.

* **Accessing Ref Values:** The c-effect directive is used to log the values of the refs whenever the component is re-rendered or when its state changes. For example, c-effect="console.log({ x: $refs.x, y: $refs.y, input: $refs.input })" logs the current values of x, y, and input.

* **Dynamic Ref Assignment:** You can dynamically change which element is referenced by using a reactive variable. For example, the <span> element uses :ref="dynamicRef", which means the ref value can change based on the value of dynamicRef. The button with the text "change dynamicRef" toggles the reference between x and y.

* **Scoped Refs:** The <div scope> element defines a nested scope, which means the ref="x" inside it refers to a local element. The $refs object will adjust to the scope context, providing an isolated reference within that scope.

* **Toggling Visibility:** The c-show="show" directive toggles the visibility of the <span>, but the ref still holds the reference to the element, even when it is hidden.

## Components

The @cmmv/reactivity framework allows you to create lightweight and reactive components in your application without the overhead of traditional frameworks. The example provided demonstrates how you can define and use a simple component using createApp, reactive, and templates.

```html
<script type="module">
    import { createApp, reactive } from '../src'
  
    function MyComp() {
      return {
        $template: '#comp',
        count: 0,
        get plusOne() {
          return this.count + 1
        }
      }
    }
  
    createApp({ MyComp }).mount()
</script>

<template id="comp">
    {{ count }} {{ plusOne }}
    <button @click="count++">++</button>
</template>

<div scope="MyComp()"></div>
```

**Component Setup:**

* The ``MyComp`` function returns the component's data and computed properties.
* The ``$template`` points to the HTML template that will render this component.

**Reactive Data:**

* ``count`` is a reactive property. Changing its value updates the UI automatically.
* ``plusOne`` is a computed property that reacts to count and displays count + 1.

**Template:**

* The template uses ``{{ count }}`` and ``{{ plusOne }}`` to display the reactive data.
* Clicking the button increments count and updates the displayed value.

**Mounting:**

The ``createApp`` function registers the component and mounts it inside the ``div scope="MyComp()"`` tag, which defines where the component will appear.