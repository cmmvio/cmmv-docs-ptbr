# Componentes

<div style="
    background-color: #FEF3C7;
    border-left: 4px solid #F59E0B;
    color: #92400E;
    padding: 1rem;
    border-radius: 0.375rem;
    margin: 1.5rem 0;
    font-size: 12px;
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

O framework CMMV apresenta uma maneira simples, mas poderosa, de integrar componentes renderizados no servidor (SSR), utilizando uma sintaxe semelhante ao Vue.js. Com o módulo ``@cmmv/view``, os componentes podem ser criados no diretório ``/public`` e importados dinamicamente para serem usados nas views.

Este documento descreve o processo completo de configuração de componentes SSR usando CMMV, com exemplos de como estruturar templates, estilos, scripts e integração com as views.

## Exemplo

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
        console.log("Componente montado");
    }
}
</script>
```

A sintaxe do componente é baseada no Vue.js, suportando métodos de ciclo de vida (``created``, ``mounted``), dados reativos (``data``), ``props`` e ``methods``. O componente também pode usar estilos com escopo com capacidades SSR.

Para um exemplo completo, veja [CMMV Reactivity Samples](https://github.com/cmmvio/cmmv-reactivity/blob/main/samples/componentTemplate.cmmv).

## Dados

Nos componentes CMMV, ``data`` é um recurso chave usado para definir estados reativos dentro do componente. A função ``data`` retorna um objeto que contém propriedades reativas, permitindo que o componente atualize dinamicamente o DOM quando os dados mudam.

```html
<template>
    <div>{{ message }}</div>
</template>

<script>
export default {
    data() {
        return {
            message: "Olá, Mundo!"
        };
    }
};
</script>
```

Quando as propriedades definidas no ``data`` são modificadas, o DOM é automaticamente atualizado para refletir essas mudanças. Isso proporciona uma experiência de usuário reativa e contínua, semelhante ao Vue.js.

## Propriedades (Props)

Os componentes podem aceitar ``props`` para torná-los reutilizáveis e flexíveis:

```html
<div scope>
    <ComponentTemplate ref="comp" :count="test"></ComponentTemplate>

    <hr/>

    <div>
        Raiz: {{ test }}<br/>
        Contexto do Componente: {{ $refs.comp.count }}
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

``/samples/componentTemplate.cmmv`` [Github](https://github.com/cmmvio/cmmv-reactivity/blob/main/samples/componentTemplate.cmmv)

```html
<template>
    <div>{{ count }}</div>

    <button @click="addCount()" class="btnAdd">Adicionar</button>
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

## Métodos

Nos componentes CMMV, métodos são essenciais para lidar com interações do usuário e realizar lógica dinâmica. Os métodos são definidos no bloco ``script`` de um componente e podem ser usados para uma variedade de finalidades, como atualizar dados, fazer requisições HTTP e acionar outras ações de componentes.

Para definir um método em um componente CMMV, inclua uma propriedade ``methods`` na exportação padrão. Os métodos podem ser chamados diretamente no template usando bindings de eventos (por exemplo, ``@click``).

```html
<template>
  <div>
    <button @click="incrementCount">Adicionar</button>
    <p>Contagem: {{ count }}</p>
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

## Criado (Created)

O hook de ciclo de vida ``created`` no CMMV é executado após a instância do componente ser criada, mas antes de ser montada no DOM. Este hook é útil para realizar lógicas como busca de dados, inicialização de variáveis ou disparo de certas ações antes do template ser renderizado.

```typescript
export default {
  data() {
    return {
      message: "Mensagem Inicial"
    };
  },
  created() {
    console.log("O componente foi criado!");
    this.fetchData();
  },
  methods: {
    fetchData() {
      this.message = "Mensagem Atualizada!";
    }
  }
}
```

O hook ``created`` fornece um ponto de entrada inicial para executar lógica antes do template ser renderizado, tornando-o útil para inicializar dados ou estados em um componente.

## Montado (Mounted)

O hook de ciclo de vida ``mounted`` é executado após o componente ter sido inserido no DOM. Este hook é ideal para ações que exigem interação direta com o DOM renderizado ou para iniciar processos que dependem da presença do componente no documento.

```typescript
export default {
  data() {
    return {
      count: 0
    };
  },
  mounted() {
    console.log("O componente foi montado!");
    this.initializeCounter();
  },
  methods: {
    initializeCounter() {
      // Interagir com o DOM ou iniciar operações
      setInterval(() => {
        this.count++;
      }, 1000);
    }
  }
}
```

O hook ``mounted`` é essencial quando você precisa garantir que seu componente esteja totalmente carregado no DOM antes de interagir com ele, tornando-o útil para manipulações no DOM, configuração de ouvintes de eventos ou inicialização de componentes que dependem de bibliotecas ou serviços de terceiros.

# Slot

Os slots no CMMV permitem passar conteúdo personalizado do escopo pai para o componente filho. Eles podem ser atualizados dinamicamente usando dados do componente pai.

```html
<ComponentTemplate ref="comp" :count="test">
    <template c-slot="{ count }">
        Valor do componente: {{ count }}
    </template>
</ComponentTemplate>
```

Neste exemplo, a diretiva ``c-slot`` é usada para passar o conteúdo do slot, tornando os dados ``count`` do pai disponíveis dentro do ``ComponentTemplate``. O escopo pai pode alterar dinamicamente o conteúdo do slot, refletindo os valores atualizados dentro do componente filho.

O slot no componente é renderizado assim:

```html
<slot :count="count"></slot>
```

**Renderização**

```html
<div ref="comp" count="123">
  Valor do componente: 123
</div>
```

* **Slots Nomeados:** Slots podem ter nomes para melhor organização.
* **Reatividade:** Slots atualizam-se reativamente com mudanças nos dados do pai.
* **Slots Escopados:** Slots escopados fornecem a capacidade de passar dados do componente filho para o pai para renderizar conteúdo personalizado.
