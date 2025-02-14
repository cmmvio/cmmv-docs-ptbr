# Vue 3

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
        O suporte para <strong>Vue</strong> e <strong>TailwindCSS</strong> foi removido do módulo principal na versão <strong>0.7.5</strong>. 
        Agora é recomendado usar <strong>Vite</strong> para gerenciar os assets do frontend e realizar o bundling. 
        Além disso, você pode usar o módulo <strong>@cmmv/vue</strong> para transpilação dos mixins e composables necessários para a integração com o Vue.
    </p>
</div>

O módulo `@cmmv/vue` fornece integração completa com o [Vue 3](https://vuejs.org/) para a construção de interfaces ricas e reativas em suas aplicações. Este módulo permite aproveitar o ecossistema do Vue enquanto se integra profundamente às capacidades de comunicação REST ou RPC do CMMV. A responsabilidade pela geração e fornecimento dos composables para o Vue 3 é totalmente gerenciada pelo módulo `@cmmv/vue`, promovendo o uso de ferramentas modernas de frontend como o Vite para gerenciar assets e comunicação eficiente com o backend.

## Funcionalidades

- **Integração com Vue 3:** Suporte completo para Vue 3, incluindo seu sistema reativo, composables e sintaxe de templates.
- **API Composable RPC:** Geração automática de composables para comunicação simplificada com o backend do CMMV via REST ou RPC usando o hook `useRPC`.
- **Arquitetura Frontend-First:** Incentiva o uso de ferramentas modernas como o Vite para gerenciar assets e fluxos de desenvolvimento.
- **Comunicação Flexível:** Use o CMMV como um proxy para facilitar a comunicação REST ou RPC em aplicações Vue.

## Instalação

Para instalar o módulo `@cmmv/vue`:

```bash
$ pnpm add @cmmv/vue vue
```

Certifique-se de ter o `vite` instalado para o gerenciamento de assets:

```bash
$ pnpm add -D vite
```

## Configuração

Atualize seu arquivo `.cmmv.config.cjs` para incluir o módulo `@cmmv/vue`. Remova quaisquer configurações diretas do Vue 3 ou TailwindCSS da seção `view`, pois essas responsabilidades agora pertencem ao gerenciamento de frontend.

```javascript
module.exports = {
    env: process.env.NODE_ENV,

    vue: {
        composableEndpoint: '/assets/rpc-composable.min.js', // Endpoint para os composables gerados
    },
};
```

## Configurando sua Aplicação

Inclua o `VueModule` na configuração da sua aplicação. O módulo cuidará da geração dos composables para comunicação com o backend.

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

O módulo `@cmmv/vue` também suporta Vue 2 através do uso de mixins, permitindo integração suave com os serviços de backend do CMMV. Para aplicações que ainda usam Vue 2, os mixins são gerados automaticamente e acessíveis via o arquivo `rpc-mixins.min.js`.

### Usando Mixins no Vue 2

Para integrar com o Vue 2, importe dinamicamente os mixins e inclua-os na instância do Vue.

**Template HTML**

```html
<!DOCTYPE html>
<html lang="pt-BR">
    <head>
        <title>Integração Vue 2 RPC</title>
        <script src="https://cdn.jsdelivr.net/npm/vue@2/dist/vue.min.js"></script>
    </head>
    <body>
        <div id="app">
            <h1>{{ example }}</h1>
            <button @click="fetchData">Buscar Dados</button>
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
            example: "Olá, Vue 2 com RPC!",
        },
        methods: {
            async fetchData() {
                const data = await this.GetAllTaskRequest();
                console.log("Dados buscados:", data);
            },
        },
        mounted() {
            console.log("Aplicação montada com exemplo:", this.example);
        },
    });
};

initApp();
```

- **Mixins:** O Vue 2 usa mixins em vez de composables para injetar métodos RPC nos componentes.
- **Hooks de Ciclo de Vida:** O Vue 2 utiliza hooks baseados em instâncias como `mounted` para gerenciar inicialização e atualizações de estado.
- **Dados Reativos:** Os dados são gerenciados pela função `data` e vinculados ao DOM através do sistema de reatividade do Vue 2.

### Recomendações para Usuários do Vue 2

- Aproveite os mixins para integração simplificada com os serviços de backend do CMMV.
- Use o arquivo `rpc-mixins.min.js` para simplificar a comunicação com o backend sem precisar configurar RPC manualmente.
- Considere migrar para o Vue 3 para aproveitar os recursos mais recentes, incluindo composables e reatividade aprimorada.

## Vue 3

O módulo `@cmmv/vue` gera um arquivo `rpc-composable.min.js` que pode ser usado em sua aplicação Vue para integrar chamadas RPC ao backend através do composable `useRPC`.

### Exemplo de Aplicação Vue 3

Instale o Vue:

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
            const example = ref("Olá, Vue com RPC!");

            const fetchData = async () => {
                const data = await rpc.GetAllTaskRequest();
                console.log("Dados buscados:", data);
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

**Template HTML**

```html
<!DOCTYPE html>
<html lang="pt-BR">
    <head>
        <title>Integração Vue RPC</title>
        <script src="https://unpkg.com/vue@3.5.12/dist/vue.global.prod.js"></script>
    </head>
    <body>
        <div id="app">
            <h1>{{ example }}</h1>
            <button @click="fetchData">Buscar Dados</button>
        </div>
        <script type="module" src="/main.js"></script>
    </body>
</html>
```

## Nuxt

Para aplicações Nuxt, importe dinamicamente os composables em seus plugins no lado do cliente.

### Instalando o Nuxt

```bash
npx nuxi init nuxt-rpc
cd nuxt-rpc
pnpm install
```

### Adicionando Plugin de Composables CMMV

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

### Usando Composables em uma Página

**pages/index.vue**

```html
<template>
    <div>
        <h1>{{ example }}</h1>
        <button @click="fetchData">Buscar Dados</button>
    </div>
</template>

<script setup>
import { useNuxtApp } from '#app';

const { useRPC } = useNuxtApp();
const rpc = useRPC();

const example = ref("Olá, Nuxt com RPC!");

const fetchData = async () => {
    const data = await rpc.GetAllTaskRequest();
    console.log("Dados buscados:", data);
};
</script>
```

## Recomendações

- Use `vite` para builds de desenvolvimento e produção.
- Utilize os composables `@cmmv/vue` para comunicação eficiente com os serviços de backend do CMMV.
- Siga as boas práticas de desenvolvimento Vue 3, aproveitando a reatividade e os composables para simplificar a arquitetura de sua aplicação.
