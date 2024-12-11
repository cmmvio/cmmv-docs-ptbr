# Estilos

O novo recurso de estilização no ``@cmmv/view`` permite que os desenvolvedores definam estilos específicos para temas que podem ser alternados dinamicamente no frontend. O sistema mapeia arquivos JSON do diretório `/public/styles`, que contêm as regras de estilo, tornando-os acessíveis no frontend para facilitar o gerenciamento de temas.

Para criar um estilo personalizado, defina um arquivo `.style.json` em `/public/styles`. Cada par chave-valor representa uma classe de estilo, e você pode criar múltiplos temas adicionando um sufixo (por exemplo, `.dark`) às chaves.

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

## Acessando

No seu HTML, você pode acessar os estilos definidos referenciando o arquivo de estilo e as chaves. O objeto de estilos é automaticamente disponibilizado, e as classes específicas do tema são aplicadas com base no tema atual.

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

## Alternando

O sistema permite alternar entre temas (por exemplo, ``default``, ``dark``). Quando um tema é alterado, todas as classes relevantes com o sufixo (por exemplo, ``.dark``) são aplicadas automaticamente.

```javascript
toggleTheme() {
    this.$style.switch(
        (this.$style.theme === "default") ? 
        "dark" : "default"
    );
}
```

<br/>

* **Estrutura do Arquivo JSON:** O JSON define classes de estilo para diferentes componentes. Classes específicas de tema recebem sufixos como ``.dark``, ``.light``, etc.
* **Acesso no Frontend:** Os estilos são acessados através de ``styles.[arquivo].[chave]`` no frontend.
* **Alternância Dinâmica:** A troca de tema é feita programaticamente, e o sistema lida com a substituição de classes de estilo com base no tema ativo.
* **Sem Suporte a Subíndices:** Subíndices aninhados em JSON não são suportados, o que significa que cada par chave-valor deve ser uma entrada plana.

## Gerenciamento

A seleção de tema no ``@cmmv/view`` é gerenciada automaticamente pelo framework, salvando a preferência do usuário no ``localStorage`` e recuperando-a ao carregar a página. Isso permite que o sistema mantenha uma estilização consistente com base na escolha anterior do usuário, sem intervenção manual.

Você pode verificar o tema atual diretamente no seu template usando ``$style.theme``. Por exemplo, para integrar com um componente como DocSearch que requer configuração de tema, você pode atualizar a tag HTML com o atributo ``data-theme``:

```html
<!DOCTYPE html>
<html lang="en" :data-theme='$style.theme' scope>
    <head>
        <headers/>
    </head>
</html>
```

Isso garante que o tema correto seja aplicado em componentes como barras de pesquisa e outros elementos que necessitem de reconhecimento de tema.

Esta implementação de temas dinâmicos no ``@cmmv/view`` oferece uma abordagem simplificada para gerenciar múltiplos temas sem necessidade de plugins adicionais, extensões do Tailwind ou regras CSS complexas. Ao utilizar arquivos `.style.json`, os desenvolvedores podem facilmente definir estilos específicos para temas, reduzindo a quantidade de código e garantindo alternância consistente de temas. Embora seja possível alcançar resultados semelhantes usando variáveis CSS, este método simplifica o processo, eliminando a necessidade de lógica personalizada na view ou gerenciamento CSS para alternância de tema, tornando-o intuitivo e eficiente.

* **Suporte para Estilos Aninhados:** Introduzir suporte para índices JSON aninhados poderia permitir estruturas de estilo mais complexas e melhor organização.
* **Integração com Variáveis CSS:** Uma opção para integrar com variáveis CSS pode complementar o sistema de temas, oferecendo controle mais refinado sobre a personalização dinâmica de temas.
* **Pré-carregamento de Temas:** Permitir o pré-carregamento de temas com base nas preferências do sistema (por exemplo, modo escuro baseado nas configurações do sistema operacional) para melhorar a experiência do usuário.
* **Animações Avançadas:** Adicionar transições embutidas para alternância de temas poderia melhorar a experiência do usuário ao alternar entre modos claro e escuro.

## Componente

Além do sistema de estilos discutido anteriormente, o CMMV introduz a propriedade ``$style`` dentro de componentes, permitindo que estilos com escopo sejam acessados diretamente.

Esse recurso permite que os desenvolvedores definam e referenciem estilos dentro do modelo de dados do componente, facilitando o gerenciamento de estilos dentro dos templates.

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

Isso permite um gerenciamento de estilo mais intuitivo, tornando os estilos acessíveis diretamente dentro do contexto do componente.
