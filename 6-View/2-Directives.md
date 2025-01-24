# Diretrizes

<div style="
    background-color: #DBEAFE; 
    border-left: 4px solid #3B82F6; 
    color: #1E40AF; 
    padding: 1rem; 
    border-radius: 0.375rem; 
    margin: 1.5rem 0;
">
    <p style="font-weight: bold; margin-bottom: 0.5rem;">Aviso Importante</p>
    <p>
        As diretivas de SSR (renderização do lado do servidor) estão disponíveis exclusivamente para implementações que utilizam o <strong>@cmmv/view</strong> como template de renderização. 
        Para aplicações em <strong>Vue</strong> servidas pelo <strong>Vite</strong> ou arquivos estáticos, os templates não passam pelas funções de SSR, e as diretivas não serão processadas durante o build ou em tempo de execução.
    </p>
</div>

Pré-carregar dados que serão enviados diretamente no HTML é um fator crucial para SEO (Search Engine Optimization). Quando um mecanismo de busca analisa uma página, ele procura por conteúdo que já esteja renderizado no HTML. Ao garantir que os dados essenciais da página já estejam carregados no HTML, o tempo de renderização é reduzido e o conteúdo fica imediatamente acessível para indexação, melhorando a visibilidade nos resultados de busca.

O CMMV (Contract-Model-Model-View) foi projetado com essa necessidade em mente, otimizando a entrega de conteúdo de uma maneira que prioriza SEO. O sistema emula o modelo tradicional MVC (Model-View-Controller), amplamente utilizado por frameworks conhecidos como Ruby on Rails, Laravel ou Spring. Isso é feito por meio de diretrizes de renderização do lado do servidor (SSR - Server-Side Rendering), que permitem que os dados sejam processados no servidor e inseridos diretamente no HTML, sem a necessidade de outra aplicação ou de sobrecarregar o frontend com frameworks complexos.

Usando diretrizes de SSR como `s-data`, o CMMV garante que os dados dinâmicos sejam enviados no próprio HTML, sem depender de chamadas AJAX ou carregamento assíncrono via JavaScript, o que pode atrasar a exibição de conteúdo crítico e impactar negativamente o desempenho da página. Isso torna o conteúdo instantaneamente acessível para mecanismos de busca, melhorando o tempo de carregamento (TTFB) e maximizando as chances de boa indexação.

Esse modelo não apenas garante carregamento mais rápido, mas também melhora a experiência do usuário final. O usuário recebe uma página completa quase instantaneamente, sem a necessidade de esperar por carregamentos adicionais ou atualizações de conteúdo. Ao centralizar o processamento de dados no backend, o CMMV reduz a complexidade e a sobrecarga do frontend, eliminando a necessidade de configurações adicionais ou de executar várias aplicações para renderizar e servir conteúdo dinâmico.

Em resumo, o CMMV é uma solução moderna que reutiliza o melhor do modelo MVC tradicional, adaptando-o ao cenário atual de otimização para SEO e desempenho, com foco na simplicidade e eficiência.

## s-data

A diretiva `s-data` permite renderizar diretamente dados do lado do servidor em seus templates HTML. Isso é particularmente útil para inserir variáveis do servidor diretamente no HTML, sem precisar de frameworks adicionais ou JavaScript no frontend.

### Como funciona

Você declara uma variável no controlador do lado do servidor e a passa para a visualização. A diretiva `s-data` é responsável por injetar essa variável diretamente no elemento HTML correspondente durante a renderização do lado do servidor (SSR). Isso garante que o conteúdo seja pré-renderizado, otimizando o desempenho da página e o SEO.

#### Visualização (Template HTML)
```html
<div s-data="datetime"></div>
```

Nesse exemplo, a diretiva `s-data` é usada em um elemento `<div>` para renderizar a variável `datetime`, que será fornecida pelo servidor.

#### Controlador
```typescript
res.render("template", { datetime: new Date().getTime() });
```

No controlador, você passa a variável `datetime` (o timestamp atual) como parte dos dados renderizados no template.

#### Resultado (HTML Renderizado)
```html
<div>1725568913552</div>
```

## s-attr

A diretiva `s-attr` no CMMV permite atribuir dinamicamente atributos a elementos HTML a partir de dados renderizados no servidor, de forma semelhante à diretiva `s-data`. No entanto, em vez de preencher o conteúdo de uma tag, ela adiciona ou atualiza um atributo do elemento. Isso é particularmente útil para adicionar atributos de segurança como `nonce` em tags `<script>` ou `<link>`, frequentemente exigidos por Políticas de Segurança de Conteúdo (CSP).

Visualização (Template HTML)
```html
<div s-attr="ref">Conteúdo</div>
```

Controlador
```typescript
res.render("template", { ref: "ABC" });
```

Resultado (HTML Renderizado)
```html
<div ref="ABC">Conteúdo</div>
```

Nesse caso, a diretiva `s-attr="ref"` instrui o servidor a criar um atributo `ref` no elemento `<div>` com o valor "ABC", que é passado pelo controlador do lado do servidor.

## s-i18n

A diretiva `s-i18n` é o módulo de internacionalização nativo do sistema CMMV. Ela permite que os desenvolvedores gerenciem e exibam conteúdo multilíngue utilizando arquivos de localidade. Esses arquivos de localidade são armazenados no diretório `/src/locale` e devem estar no formato JSON.

O CMMV permite definir um idioma padrão e possibilita a troca dinâmica de idioma com base na sessão do usuário. Isso é ideal para sites e aplicações que precisam atender a vários idiomas.

Os arquivos de localidade contêm pares de chave-valor, onde a chave representa o identificador da string e o valor é a string localizada. Por exemplo:

`/src/locale/en.json`
```json
{
    "welcome": "Welcome to CMMV",
    "description": "CMMV makes development faster and easier."
}
```

`/src/locale/pt-br.json`
```json
{
    "welcome": "Bem-vindo ao CMMV",
    "description": "O CMMV torna o desenvolvimento mais rápido e fácil."
}
```

Você pode usar a diretiva `s-i18n` em seus templates de visualização para renderizar strings localizadas com base no idioma atual.

Visualização (Template HTML)
```html
<div s-i18n="welcome"></div>
<p s-i18n="description"></p>
```

Resultado (Para o idioma inglês)
```html
<div>Welcome to CMMV</div>
<p>CMMV makes development faster and easier.</p>
```

**Configuração**

No arquivo `.cmmv.config.js`, você pode configurar as definições de i18n para definir o caminho do arquivo de localidade e o idioma padrão para sua aplicação:

```typescript
module.exports = {
    i18n: {
        localeFiles: "./src/locale",  // Caminho para os arquivos de localidade
        default: "en"  // Idioma padrão
    },
    // Outras configurações...
}
```

## s-if

A diretiva `s-if` é usada para renderizar condicionalmente um bloco de conteúdo com base na avaliação de uma expressão. Se a expressão fornecida for avaliada como verdadeira, o conteúdo dentro da diretiva `s-if` será renderizado. Caso contrário, o conteúdo não será exibido.

```html
<s-if exp="todolist.length > 0">
    <div>Total de Registros Carregados SSR: {{ todolist.length }}</div>
    <s-else>
        <div>Nenhum registro foi carregado via SSR</div>
    </s-else>
</s-if> 
```

* `exp:` A expressão booleana a ser avaliada. A expressão pode usar variáveis e operadores lógicos para determinar se o bloco deve ser exibido.

Embora a diretiva `s-if` forneça uma maneira poderosa de renderizar condicionalmente conteúdo, ela deve ser usada com cautela devido ao seu impacto potencial em atualizações no lado do cliente. O conteúdo dentro de um bloco `s-if` é removido do DOM se a expressão for avaliada como falsa, e essa remoção não é automaticamente atualizada ou re-renderizada no lado do cliente após a renderização inicial no servidor (SSR).

**Recomendações:**
* **Caso de Uso:** Recomenda-se usar `s-if` principalmente para cenários onde as condições do lado do servidor determinam o conteúdo que deve ser incluído no HTML inicial. Por exemplo, você pode usar `s-if` para carregar e exibir dados que estão disponíveis no momento da renderização do servidor, como dados de autenticação do usuário ou configurações iniciais.

* **Evitar Atualizações Frequentes no Cliente:** Como `s-if` remove o conteúdo do DOM com base na avaliação inicial, não é adequado para conteúdo dinâmico no lado do cliente que pode mudar frequentemente ou ser atualizado após o carregamento inicial. Para conteúdo que pode precisar ser atualizado dinamicamente com base em interações no lado do cliente ou mudanças de dados, considere usar outros métodos, como técnicas de renderização do lado do cliente ou frameworks reativos.

Seguindo essas recomendações, você pode garantir que o uso da diretiva `s-if` melhore tanto o desempenho quanto a manutenibilidade de sua aplicação sem efeitos colaterais indesejados.

## s-for

A diretiva `s-for` é projetada para lidar com a renderização no lado do servidor (SSR) de listas e garantir que os dados dinâmicos sejam pré-renderizados para melhorar o SEO. Essa diretiva, usada em conjunto com a diretiva `c-for`, permite definir como os dados devem ser renderizados no servidor e como devem ser atualizados no cliente. Ela suporta a pré-renderização de conteúdo que será exibido imediatamente para os usuários e motores de busca, permitindo também atualizações no lado do cliente.

```html
<s-for
    c-show="condition"
    c-for="(item, key) in collection"
    render-tag="tag-name"
>
    <!-- Conteúdo a ser renderizado -->
</s-for>
```

**Como Funciona**

* A diretiva `s-for` pré-renderiza os itens da lista no servidor com base nos dados fornecidos. Isso inclui avaliar expressões e injetar conteúdo estático diretamente no HTML.
* A diretiva garante que o conteúdo definido dentro do bloco `s-for`, incluindo variáveis renderizadas usando `{{}}`, `c-text`, `c-html` e `:`, seja pré-renderizado.
* No lado do cliente, a diretiva `c-for` assume o controle e lida com atualizações dinâmicas. Isso permite que a lista seja reativa e atualizada conforme necessário.

**Template de Entrada:**
```html
<s-for 
    c-show="todolist"
    c-for="(item, key) in todolist"
    class="todo-item"
    render-tag="div"
>
    <div class="todo-item-content">
        <input 
            type="checkbox" 
            c-model="item.checked" 
            @change="UpdateTaskRequest(item)"
        ></input>

        <label 
            :class="{'todo-item-checked': item.checked}"
        >{{ item.label }}</label>
    </div>
    
    <button 
        class="todo-btn-remove"
        s-i18n="remove" 
        @click="DeleteTaskRequest(item.id)"
    ></button>
</s-for> 
```

**Saída Renderizada Final:**
```html
<div c-if="!loaded && !todolist">
    <div c-show="todolist" class="todo-item">
        <div class="todo-item-content">
            <input 
                type="checkbox" 
                c-model="item.checked" 
                @change="UpdateTaskRequest(1)"
            >
            <label class="todo-item-checked">Task 1</label>
        </div>
        <button 
            class="todo-btn-remove" 
            @click="DeleteTaskRequest(1)"
        >Remove</button>
    </div>
</div>
<div c-else>
    <div 
        c-show="todolist" 
        c-for="(item, key) in todolist" 
        class="todo-item"
    >
        <div class="todo-item-content">
            <input 
                type="checkbox" 
                c-model="item.checked" 
                @change="UpdateTaskRequest(item)"
            >
            <label 
                :class="{'todo-item-checked': item.checked}"
            >{{ item.label }}</label>
        </div>
        <button 
            class="todo-btn-remove" 
            @click="DeleteTaskRequest(item.id)"
        >Remove</button>
    </div>
</div>
```

* **Pré-Renderização:** A diretiva `s-for` é crucial para SEO, pois pré-renderiza itens de lista com dados reais, tornando o conteúdo imediatamente disponível para indexação pelos motores de busca.
* **Atualização no Lado do Cliente:** No lado do cliente, a diretiva `c-for` garante que o conteúdo seja reativo e possa ser atualizado dinamicamente.
* **Uso com Cautela:** Como `s-for` realiza renderização no lado do servidor, garanta que a lista e seus conteúdos sejam adequados para pré-renderização estática. Evite usá-la para dados altamente dinâmicos que mudam frequentemente no lado do cliente.

Ao combinar `s-for` com `c-for`, você pode aproveitar os benefícios de renderização no servidor para SEO, mantendo atualizações dinâmicas no lado do cliente.

## Include 

A diretiva `include` permite a inserção de componentes ou templates de outros arquivos no layout principal. Esse recurso é útil para construir páginas modulares, permitindo a divisão do layout em pequenos blocos reutilizáveis, o que otimiza tanto o desempenho de carregamento quanto a manutenção do código.

### Como Funciona

A diretiva `include` é responsável por pré-carregar componentes no lado do servidor antes de enviar a página ao cliente. Esses componentes são tratados como templates, e sua inclusão pode ocorrer de maneira em cascata, o que significa que um template pode incluir outros templates. No entanto, esse comportamento é influenciado pela estratégia de cache, garantindo um desempenho ideal ao prevenir o carregamento repetido de componentes já processados.

```html
<div>
    <!-- Inclui o navbar -->
    <!-- include('public/views/docs/navbar') -->
    
    <!-- Inclui o footer -->
    <!-- include('public/views/docs/footer') -->
</div>
```

Ao usar a diretiva `include`, é importante considerar como os dados ou o comportamento de cada template serão gerenciados. A diretiva é projetada para pré-processar componentes no lado do servidor, portanto, qualquer lógica de configuração precisa ser planejada cuidadosamente. A abordagem recomendada é executar a lógica de configuração no template principal e evitar configurações em cascata nos templates incluídos. Isso previne execuções múltiplas desnecessárias que poderiam impactar negativamente o desempenho.

### Exemplo Completo

```html
<div id="app">
    <!-- include('public/views/docs/navbar') -->

    <div class="content">
        <p>Bem-vindo ao nosso site!</p>
    </div>

    <!-- include('public/views/docs/footer') -->
</div>

<script>
export default {
    data() {
        return {
            navbarState: {},
            contentData: {}
        };
    },
    mounted() {
        this.loadState();
    },
    methods: {
        loadState() {
            this.navbarState = JSON.parse(
                localStorage.getItem('navbarState')
            ) || {};

            this.contentData = { 
                message: 'Conteúdo carregado dinamicamente.' 
            };
        }
    }
};
</script>
```

## Serviços de Chamada

Nesta implementação, você tem uma combinação de diretivas no lado do servidor para pré-carregamento de dados e o uso da diretiva `include` para incorporar componentes reutilizáveis, como a barra de navegação e outros elementos de layout, na página. Embora esse padrão funcione de maneira eficiente, atualmente ele não suporta configurações específicas de componentes, como visto em frameworks como Nuxt.js. Vamos detalhar como essa abordagem funciona e o potencial para melhorias futuras.

**Diretiva no Lado do Servidor (`s:docs`)**

No arquivo HTML principal (`index.html`), você está usando uma diretiva (`s:docs="docs"`) para vincular dados pré-carregados no servidor (neste caso, `docs`) ao renderizador no lado do cliente. Isso permite que os dados iniciais, como a estrutura de navegação (`docs.navbar`), sejam buscados no servidor e passados ao cliente antes que a página seja renderizada.

```html
<div id="app" s:docs="docs" c-cloak>
```

O objeto `docs` é populado no servidor, provavelmente de uma fonte de dados backend ou arquivo, e injetado no template. Isso permite o uso desses dados imediatamente após o carregamento, sem chamadas adicionais de API no lado do cliente, melhorando o desempenho do carregamento inicial.

**Usando a Diretiva Include para Templates**

Você está incluindo componentes reutilizáveis, como a barra de navegação, breadcrumb e rodapé, com a diretiva `include`. Esses templates são pré-carregados no servidor, melhorando a modularidade e a manutenção:

```html
<!-- include('public/views/docs/navbar') -->
<!-- include('public/views/docs/breadcrumb'); -->
<!-- include('public/views/docs/footer'); -->
```

Ao carregar esses componentes no lado do servidor, você evita múltiplas solicitações HTTP para buscar os templates separadamente e permite que o servidor lide com a lógica relacionada à inclusão de conteúdo. No entanto, como observado, essa abordagem atualmente processa apenas a lógica de configuração principal, e qualquer lógica adicional dentro dos templates incluídos (como `navbar.html`) não é suportada.

### Atualizações de Estado no Lado do Cliente

Depois que os dados e templates são pré-carregados no servidor, o JavaScript no lado do cliente é responsável por lidar com as atualizações de estado. Por exemplo, no seu componente principal, você carrega o estado salvo da barra de navegação do `localStorage` e fornece métodos para alternar o estado:

```javascript
export default {
    data() {
        return { navbar: [] }
    },

    async mounted() {
        this.loadState();
    },

    methods: {
        loadState() {
            this.navbar = JSON.parse(
                localStorage.getItem('navbarState')
            ) || {};
            
            return this.navbar;
        },

        saveState(state) {
            localStorage.setItem('navbarState', JSON.stringify(state));
        },

        toggle(isOpened, itemName) {
            isOpened = !isOpened;
            const currentState = this.loadState();
            currentState[itemName] = isOpened;
            this.saveState(currentState);
            return isOpened;
        }
    }
};
```

Isso garante que interações na página, como expandir ou recolher seções da barra de navegação, sejam persistentes entre sessões, já que o estado é armazenado no `localStorage` e recarregado quando a página é montada.

### A Limitação

Como observado no comentário fornecido, atualmente não há suporte para lógica de configuração individual dentro de templates incluídos (como `navbar.html`). Isso difere de frameworks como Nuxt.js, onde cada componente pode ter sua própria função `setup`. Na implementação atual, adicionar um bloco `script setup` dentro de um template incluído causaria problemas, pois apenas a lógica de configuração do template principal (`index.html`) é processada.

Por exemplo, adicionar um bloco `setup` no `navbar.html` não funcionaria como esperado:

```html
<!-- navbar.html -->
<script s-setup>
export default {
    data() {
        return { isOpened: false };
    },
    methods: {
        toggleNavbar() {
            this.isOpened = !this.isOpened;
        }
    }
};
</script>
```

Isso quebraria a aplicação porque apenas a primeira configuração seria processada, e configurações adicionais seriam ignoradas.

### Melhorias Futuras

A ideia de permitir configurações específicas de componentes, semelhante ao Nuxt.js, é uma melhoria potencial que poderia aumentar a flexibilidade do sistema. No entanto, isso precisa ser tratado com cuidado para evitar problemas de desempenho, como loops infinitos de carregamento. Quando cada componente tem sua própria lógica de configuração, a aplicação pode acabar em um loop contínuo de re-renderização, se não for gerenciada corretamente.

Para implementar configurações específicas de componentes de maneira segura:

* **Gerenciamento Cuidadoso de Dependências:** Garanta que a lógica de configuração de cada componente seja independente e não acione re-renderizações desnecessárias do componente pai.
* **Carregamento Sob Demanda:** Considere o carregamento sob demanda para templates ou componentes que não precisam ser pré-carregados no servidor, reduzindo o carregamento inicial e evitando possíveis problemas em aplicações grandes.
* **Compartilhamento de Estado Otimizado:** Forneça mecanismos para compartilhar estado entre componentes sem exigir a reexecução completa da lógica de configuração quando componentes são incluídos dinamicamente.

Aqui está como configurações específicas de componentes poderiam funcionar no futuro, seguindo um modelo semelhante ao Nuxt.js:

```html
<!-- index.html -->
<div id="app" s:docs="docs" c-cloak>
    <div>
        <!-- include('public/views/docs/navbar') -->
    </div>
</div>
```

<br/>

```html
<!-- navbar.html -->
<script s-setup>
export default {
    data() {
        return { isOpened: false };
    },
    methods: {
        toggleNavbar() {
            this.isOpened = !this.isOpened;
        }
    }
};
</script>

<ul>
    <li @click="toggleNavbar">Menu Item</li>
</ul>
```

Nesse cenário futuro, cada componente ou template poderia gerenciar seu próprio estado e métodos sem conflito com a configuração do template principal.

## Implementação Completa

Abaixo está um exemplo de implementação de praticamente todas as diretivas que foram usadas para criar o sistema nesta documentação.

```html
<!-- /public/templates/default.html -->
<!DOCTYPE html>
<html>
    <head>
        <headers/>
    </head>
    <body scope> 
        <slot/>
        <scripts/>
    </body>
</html>
```

<br/>

```html
<!-- /public/views/docs/index.html -->
<div id="app" s:docs="docs" c-cloak>
    <nav class="navbar bg-neutral-800 h-16 top-0 w-full fixed flex z-50 shadow-lg">
        <div class="max-w-8xl mx-auto flex container">
            <div class="w-60">
                <a href="/" title="UCS.js">
                    <img src="/assets/logo-min-invert.png" class="mt-6 ml-4" />
                </a>
            </div>

            <div class="justify-between w-full text">
                <div class="relative text-right mt-3 hover:cursor-pointer group bg-neutral-800 float-right rounded-lg border border-black">
                    <div class="absolute text-white z-40 top-2 left-18" style="left: 12px; color: #ccd0d5;">
                        <i class="fa-solid fa-search"></i>
                    </div>
                    <div>
                        <input 
                            type="text" 
                            class="p-1.5 pl-10 text-white bg-transparent" 
                            placeholder="Search" 
                        />
                    </div>
                </div>
            </div>

            <div class="justify-between align-middle text-center mr-2 text-white flex">
                <a href="https://github.com/cmmvio/cmmv" title="Github" target="_blank" class="text-2xl p-2 mt-2 hover:text-neutral-300">
                    <i class="fa-brands fa-github"></i>
                </a>
            </div>
        </div>
    </nav>

    <div class="max-w-8xl mx-auto flex container">
        <div 
            class="w-60 fixed mt-20 z-40 overflow-auto" 
            style="height: calc(100% - 84px); background-color: #2e3035" 
            c-cloak
        >
            <!-- include('public/views/docs/navbar') -->
        </div>

        <div class="mt-20 ml-64 text-justify relative">
            <div 
                class="lg:pl-[19.5rem] m-4 p-4 px-20 max-w-3x1 mx-auto xl:max-w-none xl:ml-0 xl:mr-[15.5rem] xl:pr-16"
                :class="{'w-full': docs.anchors.length < 4}"
            >
                <!-- include('public/views/docs/breadcrumb'); -->
                <!-- include('public/views/docs/anchors'); -->

                <div class="max-w-screen-lg relative text-white mb-20 context-html">
                    <div c-html="docs.index">{ docs.index }</div>

                    <div class="absolute top-0 right-0">
                        <a 
                            :href="`https://github.com/cmmvio/docs.cmmv.io/tree/main${docs.link?.replace('.html', '.md')}?plain=1`" 
                            target="_blank" 
                            title="Sugerir mudança"
                        >
                            <i class="fa-solid fa-pen-to-square fa-lg"></i>
                        </a>
                    </div>
                </div>
            </div>

            <!-- include('public/views/docs/footer'); -->
        </div>
    </div>
</div>

<script s-attr="nonce">
    function updateCurrent(){
        const scrollPosition = window.scrollY;

        document.querySelectorAll('.current').forEach(el => el.classList.remove('current'));

        let repoint = false;
        document.querySelectorAll('#anchors li').forEach((item, index) => {
            const target = document.querySelector(item.querySelector('a').getAttribute('href'));
            
            if (target?.offsetTop >= scrollPosition && !repoint) {
                repoint = true;
                item.classList.add('current');
            }
            else if(!target) {
                repoint = true;
                item.classList.add('current');
            }
        });
    }

    window.addEventListener('scroll', updateCurrent);

    window.addEventListener('DOMContentLoaded', () => {
        document.querySelectorAll('a[href^="#"]').forEach((link, index) => {
            link.addEventListener('click', function(event) {
                event.preventDefault();
                const href = this.getAttribute('href');
                
                document.querySelectorAll('.current').forEach(el => el.classList.remove('current'));
                
                this.parentElement.classList.add('current');
                
                window.scrollTo({
                    top: document.querySelector(href).offsetTop,
                    behavior: 'smooth'
                });

                window.location.hash = href;
            });
        });

        updateCurrent();
    });
</script>

<script s-setup>
export default {
    layout: "default",

    data(){
        return { navbar: [] }
    },

    async mounted() {
        this.loadState();
    },

    methods: {
        loadState(){
            this.navbar = JSON.parse(localStorage.getItem('navbarState')) || {};
            return this.navbar;
        },

        saveState(state) {
            localStorage.setItem('navbarState', JSON.stringify(state));
        },

        toggle(isOpened, itemName) {
            isOpened = !isOpened;

            const currentState = this.loadState();
            currentState[itemName] = isOpened;
            this.saveState(currentState);

            return isOpened;
        }
    }
}
</script>
```

<br/>

```html
<!-- /public/views/docs/navbar.html -->
<ul class="p-4 select-none top-16" c-cloak c-show="docs">
    <li c-for="(item, key) in docs.navbar">
        <div 
            c-show="item"
            class="flex hover:text-blue-700 itemRoot text-white" 
            :id="item?.name.replace(/\\s/,'_')" 
            :data-opened="false" 
            @click.stop="navbar[item?.name.replace(/\\s/, `_`)] = toggle(navbar[item?.name.replace(/\\s/, `_`)], item.name?.replace(/\\s/, `_`))"
        >
            <div class="flex flex-1 font-bold text-md cursor-pointer navbar-item">
                <h3 c-if="item && item?.isDir" class="text-white">{{ item?.name }}</h3>
                <span c-else class="text-white">{{ item?.name }}</span>
            </div>

            <div class="justify-between cursor-pointer" c-if="item?.isDir">
                <i class="fa-solid fa-angle-down" c-show="!navbar[item?.name.replace(/\\s/, `_`)]"></i>
                <i class="fa-solid fa-angle-up" c-show="navbar[item?.name.replace(/\\s/, `_`)]"></i>
            </div>
        </div>

        <ul 
            c-if="item && item.children && item.children.length > 0"
            :id="`\${item?.name.replace(/\\s/, `_`)}_contents`" 
            class="p-4 py-1 text-md mb-4"
            :style="(navbar[item?.name.replace(/\\s/, `_`)]) ? '' : 'display: none;'"
        >
            <li c-for="(child) in item.children">
                <div class="hover:text-gray-800 text-white text-base p-1" style="font-size: 12px">
                    <a :href="child.uri" class="text-base">{{ child.name }}</a>
                </div>
            </li>
        </ul>
    </li>
</ul>
```

<br/>

```typescript
// /src/docs.controller.ts
import * as fs from 'fs';
import * as path from "path";

import { 
    Controller, Get, Param, 
    Response, ServiceRegistry 
} from '@cmmv/http';

import { DocsService } from './docs.service';

const index = require("../docs/index.json");

@Controller("docs")
export class DocsController {
    constructor(private docsService: DocsService){}

	@Get()
	async indexHandler(@Response() res) {		
		return res.render("views/docs/index", {
			docs: await this.docsService.getDocsStrutucture(),
			services: ServiceRegistry.getServicesArr()
		});
	}

	@Get(":item")
	async getDocHandler(
        @Param("item") item: string, 
        @Response() res
    ) {
		if(index[item])
			this.getDoc(index[item], res)
		else
			res.status(404).end();
	}

	@Get(":dir/:item")
	async getDocSubdirHandler(
        @Param("dir") dir: string, 
        @Param("item") item: string, 
        @Response() res
    ) {
		const fullPath = `\${dir}/\${item}`;

		if(index[fullPath])
			this.getDoc(index[fullPath], res)
		else
			res.status(404).end();
	}

	async getDoc(docFilename: string, @Response() res) {
		const file = path.resolve(docFilename);
		const data = await this.docsService.getDocsStrutucture(file);

		return res.render("views/docs/index", {
			docs: data,
			services: ServiceRegistry.getServicesArr()
		});
	}
}
```

Para acessar o código completo, visite [Github](https://github.com/cmmvio/docs.cmmv.io).
