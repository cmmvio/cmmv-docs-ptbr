# Tailwind CSS

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
        O suporte para integração com <strong>TailwindCSS</strong> foi removido do módulo principal na versão <strong>0.7.5</strong>. 
        Agora é recomendado usar o <strong>Vite</strong> para gerenciar os assets do frontend e realizar o bundling. Siga os passos abaixo 
        para configurar o TailwindCSS com Vite no seu projeto CMMV.
    </p>
</div>

Siga estas etapas para integrar o [TailwindCSS](https://tailwindcss.com/) ao seu projeto CMMV usando o Vite para fluxos de trabalho modernos e otimizados de desenvolvimento:

## Instale o TailwindCSS

Execute o seguinte comando para adicionar o TailwindCSS como dependência de desenvolvimento:

```bash
pnpm add -D tailwindcss postcss autoprefixer
```

## Inicialize o TailwindCSS

Gere os arquivos de configuração do TailwindCSS executando:

```bash
npx tailwindcss init
```

Isso criará um arquivo `tailwind.config.js` na raiz do seu projeto.

## Configure o TailwindCSS

Atualize o arquivo `tailwind.config.js` para incluir os caminhos para seus componentes Vue ou outros arquivos de frontend:

```javascript
/** @type {import('tailwindcss').Config} */
module.exports = {
    content: [
        './index.html',
        './src/**/*.{vue,js,ts,jsx,tsx}',
    ],
    darkMode: 'class', // Habilitar dark mode baseado em classes
    theme: {
        extend: {},
    },
    plugins: [],
};
```

## Crie o Arquivo CSS de Entrada

Crie um arquivo `src/tailwind.css` no seu projeto e inclua as diretivas do TailwindCSS:

```css
@tailwind base;
@tailwind components;
@tailwind utilities;
```

## Configure o Vite

Modifique seu `vite.config.js` para garantir que o TailwindCSS seja processado corretamente:

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

## Use o TailwindCSS no Seu Projeto

Certifique-se de importar o arquivo CSS de entrada (`tailwind.css`) na sua aplicação. Você pode fazer isso importando-o no seu `main.js`:

```javascript
import { createApp } from 'vue';
import App from './App.vue';
import './tailwind.css'; // Importa o TailwindCSS

createApp(App).mount('#app');
```

## Template HTML

Certifique-se de que seu `index.html` está configurado para incluir o elemento `#app`, onde o Vue será montado:

```html
<!DOCTYPE html>
<html lang="pt-BR">
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

## Recomendações

- Use o módulo `@cmmv/vue` para gerar mixins e composables RPC para integração com o Vue.
- Configure o Vite como sua principal ferramenta de build para gerenciar fluxos de trabalho modernos de frontend.
- Siga as melhores práticas do TailwindCSS para criar estilos reutilizáveis e escaláveis.

Essa configuração garante uma separação clara de responsabilidades, permitindo que o CMMV se concentre em operações de backend enquanto o Vite gerencia os assets do frontend. 
