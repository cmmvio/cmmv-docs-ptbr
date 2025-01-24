# Reactivity

<div style="
    background-color: #FEF3C7; 
    border-left: 4px solid #F59E0B; 
    color: #92400E; 
    padding: 1rem; 
    border-radius: 0.375rem; 
    margin: 1.5rem 0;
">
    <p style="font-weight: bold; margin-bottom: 0.5rem;">Aviso</p>
    <p>
        A partir da versão <strong>0.7.5</strong>, o módulo <strong>@cmmv/reactivity</strong> está sendo substituído pela integração nativa do <strong>Vue 3</strong> com <strong>Vite</strong>, que será a pipeline recomendada para versões futuras. 
        Embora o <strong>@cmmv/reactivity</strong> continue sendo suportado, nenhuma funcionalidade adicional será adicionada ao script além das já existentes.
    </p>
    <p>
        Além disso, o módulo <strong>@cmmv/view</strong> terá suporte ao <strong>Angular</strong> e <strong>React</strong> em versões futuras, oferecendo uma solução mais versátil para renderização no lado do servidor e integração com o frontend.
    </p>
</div>

O módulo `@cmmv/reactivity` foi criado com base nas ideias centrais do [Petite Vue](https://github.com/vuejs/petite-vue), uma versão simplificada do Vue.js. O Petite Vue, desenvolvido por Evan You, criador do Vue.js, oferece uma alternativa leve ao framework completo do Vue, fornecendo apenas os recursos essenciais de reatividade e templating necessários para projetos menores ou mais simples. O Petite Vue tem apenas `6KB` e implementa diretivas básicas, tornando-se uma excelente base para construir sistemas reativos mínimos.

Inspirado pela filosofia de design do Petite Vue, o `@cmmv/reactivity` integra essa abordagem reativa leve ao framework CMMV. Com isso, ele oferece aos desenvolvedores a capacidade de usar binding bidirecional de dados e atualizações eficientes de UI com overhead mínimo. O núcleo reativo do Petite Vue foi adaptado para se integrar perfeitamente ao ecossistema CMMV, permitindo integração fácil com renderização no lado do servidor, WebSockets, Protobuf e outros módulos principais do CMMV.

Com essa combinação, o `@cmmv/reactivity` proporciona um equilíbrio ideal entre simplicidade e desempenho, sendo perfeito para construir interfaces de usuário dinâmicas enquanto mantém o tamanho do bundle geral mínimo. Ele é adequado para desenvolvedores que preferem um sistema reativo inspirado no Vue, mas sem a complexidade de um framework completo.

Para mais informações sobre o Petite Vue, visite o [repositório oficial do Petite Vue](https://github.com/vuejs/petite-vue).

## Limitações

O objetivo principal do framework CMMV é construir aplicações de alto desempenho com forte foco em velocidade, eficiência e otimização para SEO. Para alcançar isso, recomendamos fortemente o uso do `@cmmv/reactivity` como o framework frontend em vez de incorporar frameworks mais pesados como Vue, React ou Angular. O `@cmmv/reactivity` é projetado para lidar com a maioria dos desafios comuns enfrentados em aplicações web, fornecendo recursos essenciais de reatividade em um pacote leve que minimiza o impacto nos tempos de carregamento e no desempenho da aplicação.

Ao escolher o `@cmmv/reactivity`, você garante que sua aplicação permaneça rápida e otimizada tanto para os usuários quanto para os mecanismos de busca. A adição de frameworks maiores, como Vue, React ou Angular, introduz camadas adicionais de JavaScript que podem aumentar o tempo de carregamento inicial da página, prejudicando a pontuação do PageSpeed e o desempenho geral. Isso vai contra os princípios centrais do CMMV, que visam reduzir overheads desnecessários.

Se você identificar que determinados recursos críticos estão faltando no `@cmmv/reactivity`, incentivamos você a enviar um pull request para o repositório. Estamos abertos a sugestões e contribuições que melhorem o framework, mantendo-o alinhado aos objetivos de desempenho do CMMV.

Em casos extremos em que frameworks mais pesados como Vue, React ou Angular sejam necessários, eventualmente forneceremos suporte nativo para a integração desses frameworks. No entanto, aconselhamos fortemente contra essa abordagem devido aos compromissos de desempenho envolvidos. Qualquer implementação desses frameworks será por sua conta e risco, e não ofereceremos suporte oficial para tais configurações—nosso foco permanecerá exclusivamente na otimização do `@cmmv/reactivity` para proporcionar a melhor experiência do usuário e resultados de SEO possíveis.

## Instalação

Para instalar o pacote `@cmmv/reactivity`, basta executar o seguinte comando:

```bash
$ pnpm add @cmmv/reactivity
```

Alternativamente, você pode visitar o [repositório no GitHub](https://github.com/cmmvio/cmmv-reactivity) para mais detalhes.

O módulo `@cmmv/view` já integra reatividade nativamente, então não precisa ser instalado separadamente.

## Uso

Abaixo está a documentação de todas as diretivas suportadas pelo `@cmmv/reactivity` com base nos exemplos fornecidos no repositório do CMMV Reactivity no GitHub.

## c-model 

Vincula o valor de um elemento de entrada ao modelo de dados da aplicação e permite o binding bidirecional. Isso é útil para atualizar dinamicamente a UI conforme os usuários inserem dados.

```html
<input c-model="username">
<p>Usuário: {{ username }}</p>
```

[Exemplo](https://github.com/cmmvio/cmmv-reactivity/blob/main/samples/model.html): 
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
  <h2>Entrada de Texto</h2>
  {{ text }}
  <input v-model.trim="text" />

  <h2>Área de Texto</h2>
  {{ text }}
  <textarea v-model.trim="text"></textarea>

  <h2>Checkbox</h2>
  <input type="checkbox" id="checkbox" v-model="checked" />
  <label for="checkbox">{{ checked }}</label>

  <h2>Checkbox com Array</h2>
  <label><input type="checkbox" v-model="arr" value="one" /> um</label>
  <label><input type="checkbox" v-model="arr" value="two" /> dois</label>
  <label
    ><input type="checkbox" v-model="arr" :value="123" /> número real</label
  >
  <div>{{ arr }}</div>

  <h2>Checkbox com true-value / false-value</h2>
  <input
    type="checkbox"
    v-model="checkToggle"
    :true-value="trueValue"
    :false-value="falseValue"
  />
  <div>{{ checkToggle }}</div>

  <h2>Radio</h2>
  <label><input type="radio" v-model="radioSelected" value="one" /> um</label>
  <label><input type="radio" v-model="radioSelected" value="two" /> dois</label>
  <label
    ><input type="radio" v-model="radioSelected" value="three" /> três</label
  >
  <div>{{ radioSelected }}</div>

  <h2>Seleção</h2>
  <select v-model="selected" @change="console.log(selected, $event.target.value)">
    <option>um</option>
    <option>dois</option>
    <option>três</option>
  </select>
  <div>{{ selected }}</div>
</div>
```

Neste exemplo, o valor do input é vinculado ao campo username no modelo de dados. Alterações no input serão refletidas na propriedade username.

## c-show

Controla a visibilidade de elementos com base em uma condição. Se a condição for avaliada como falsa, o elemento será ocultado.

```html
<p c-show="isLoggedIn">Bem-vindo de volta, usuário!</p>
```

## c-if

Renderiza condicionalmente um elemento apenas se a expressão especificada for verdadeira. Diferentemente de `c-show`, esta diretiva remove o elemento do DOM se a condição for falsa.

```html
<p c-if="showMessage">Esta mensagem será exibida apenas se showMessage for verdadeiro.</p>
```

[Exemplo](https://github.com/cmmvio/cmmv-reactivity/blob/main/samples/if.html):
```html
<script type="module">
  import { createApp } from '../src'
  createApp().mount('#app')
</script>

<div id="app" scope="{ open: true, elseOpen: true }">
  <button @click="open = !open">Alternar</button>
  <button @click="elseOpen = !elseOpen">Alternar Else</button>
  <div c-if="open">Ok</div>
  <div c-else-if="elseOpen">Else If</div>
  <template c-else>Else</template>
</div>
```

## c-for

Faz um loop em um array ou objeto e repete o elemento associado para cada item.

```html
<ul>
  <li c-for="item in items">{{ item }}</li>
</ul>
```

[Exemplo](https://github.com/cmmvio/cmmv-reactivity/blob/main/samples/for.html): 
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
        this.list.push({ id: ++id, text: 'novo item' });
      },
      splice() {
        this.list.splice(1, 0, { id: ++id, text: 'novo item' })
      }
    }).mount('#app')
</script>
  
<div id="app" scope>
    <button @click="add">Adicionar</button>
    <button @click="list.reverse()">Reverter</button>
    <button @click="list.pop()">Remover último</button>
    <button @click="splice">Adicionar no meio</button>
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

Anexa um listener de eventos a um elemento. Comumente usado para lidar com eventos de clique, envios de formulário ou outras interações do usuário.

```html
<button c-on:click="incrementCounter">Clique aqui!</button>
<button @click="incrementCounter">Clique aqui!</button>
```

[Exemplo](https://github.com/cmmvio/cmmv-reactivity/blob/main/samples/on.html): 
```html
<script type="module">
  import { createApp } from '../src'
  createApp().mount('#app')
</script>

<div id="app">
	<input
		type="text"
		@keyup.x="alert('yo')"
		placeholder="digite x para testar o modificador de tecla"
	/>
	<form>
		<button type="submit" @click.prevent.stop>Enviar (prevenido)</button>
	</form>
	<button @click.right="alert('clique direito')">Clique direito</button>
	<button @click.middle="alert('clique meio')">Clique botão do meio</button>
	<button @click.once="alert('clicado')">Clique uma vez</button>
</div>
```

## c-bind

Vincula dinamicamente um atributo a uma expressão. Comumente usado para modificar atributos como `src`, `href` ou `class`.

```html
<img :src="imageSource">
```

[Exemplo](https://github.com/cmmvio/cmmv-reactivity/blob/main/samples/bind.html):
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
    <div :id="id">Binding simples - deve ser verde</div>

    <div class="static" :class="classes">
        Binding de classe - deve ser vermelho e negrito
    </div>

    <div style="font-weight: bold" :style="style">
        Binding de estilo - deve ser azul e negrito
    </div>

    <div c-bind="obj">Binding de objeto - deve ser laranja</div>
</div>
```

## c-text

Define dinamicamente o conteúdo de texto de um elemento.

```html
<div scope="{ count: 1 }">
	<p c-text="count"></p>
	<button @click="count++">Aumentar</button>
</div>
```

## c-html

Insere conteúdo HTML bruto dentro de um elemento.

```html
<div c-html="htmlContent"></div>
```

## c-class

Vincula dinamicamente uma ou mais classes CSS a um elemento.

```html
<div c-class="{ active: isActive, disabled: isDisabled }"></div>
```

Neste exemplo, se `isActive` for verdadeiro, a classe `active` será adicionada. Se `isDisabled` for verdadeiro, a classe `disabled` será adicionada.

## c-once

Renderiza o conteúdo uma única vez e não reage a mudanças posteriores nos dados.

```html
<script type="module">
  import { createApp } from '../src'
  createApp().mount()
</script>

<div scope="{ count: 5 }">
    {{ count }}
    <div c-once>
      <h2>Uma vez</h2>
      {{ count }}
      <span c-text="count"></span>
      <span v-for="i in count">{{ i }}</span>
    </div>
    <span c-text="count"></span>
    <button @click="count++">++</button>
</div>
```

## ref

No `@cmmv/reactivity`, o sistema `ref` permite referenciar facilmente elementos do DOM dentro do template e manipulá-los na lógica JavaScript. Esse conceito é crucial para gerenciar elementos DOM diretamente quando as ligações reativas não são suficientes.

Benefícios chave do uso de `ref`:
- **Acesso direto ao DOM:** Permite interagir diretamente com elementos DOM, necessário para operações que não podem ser puramente reativas.
- **Atribuição dinâmica de ref:** Altere dinamicamente o elemento alvo atribuído a um `ref` em tempo de execução.
- **Refs escopados:** Suporte a escopos aninhados, permitindo que `refs` sejam definidos dentro de um contexto específico sem afetar referências globais.

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
	<p>Acessando elemento root: id é {{ $refs.root.id }}</p>

	<input ref="input" />

	<span v-show="show" :ref="dynamicRef">Span com ref dinâmico</span>

	<p>dynamicRef é {{ dynamicRef }}</p>

	<button @click="dynamicRef = dynamicRef === 'x' ? 'y' : 'x'">
		Alterar dynamicRef
	</button>

	<button @click="show = !show">Alternar</button>

	<div scope>
		<p ref="x">Ref de escopo aninhado</p>
		<button
		@click="console.log({ x: $refs.x, y: $refs.y, input: $refs.input })"
		>
		Log refs do escopo aninhado
		</button>
	</div>
</div>
```

## Components

O `@cmmv/reactivity` permite criar componentes leves e reativos sem o overhead de frameworks tradicionais. O exemplo a seguir demonstra como definir e usar um componente simples com `createApp`, `reactive` e templates.

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

### Configuração do Componente

- **Setup do Componente:** A função `MyComp` retorna os dados e propriedades computadas do componente.
- **Template:** O `$template` aponta para o HTML que renderizará o componente.

### Reatividade

- `count` é uma propriedade reativa. Alterar seu valor atualiza automaticamente a UI.
- `plusOne` é uma propriedade computada que reage a `count` e exibe `count + 1`.

### Montagem

O `createApp` registra o componente e o monta dentro da tag `div scope="MyComp()"`, que define onde o componente aparecerá.

---

Com isso, o `@cmmv/reactivity` se consolida como uma solução eficiente para desenvolver interfaces dinâmicas, leves e reativas, alinhadas aos objetivos de alto desempenho do CMMV.
