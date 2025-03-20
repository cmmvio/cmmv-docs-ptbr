# SSR (Renderização no Lado do Servidor)

<div style="
    background-color: #DBEAFE;
    border-left: 4px solid #3B82F6;
    color: #1E40AF;
    padding: 1rem;
    border-radius: 0.375rem;
    margin: 1.5rem 0;
    font-size: 12px;
">
    <p style="font-weight: bold; margin-bottom: 0.5rem;">Aviso Importante</p>
    <p>
        As diretivas de SSR (Renderização no Lado do Servidor) estão disponíveis exclusivamente para implementações que utilizam o <strong>@cmmv/http</strong> como modelo de renderização.
        Para aplicações em <strong>Vue</strong> servidas por <strong>Vite</strong> ou arquivos estáticos, os templates não passam pelas funções de SSR, e as diretivas não serão processadas durante a construção ou execução.
    </p>
</div>

# Diretivas

Pré-carregar dados que serão enviados diretamente no HTML é um fator crucial para SEO (Otimização para Motores de Busca). Quando um motor de busca analisa uma página, ele procura conteúdo que já esteja renderizado no HTML. Garantir que os dados essenciais da página estejam pré-carregados no HTML reduz o tempo de renderização e torna o conteúdo imediatamente acessível para indexação, melhorando a visibilidade nos resultados de busca.

O CMMV (Contract-Model-Model-View) foi projetado com essa necessidade em mente, otimizando a entrega de conteúdo de forma a priorizar o SEO. O sistema emula o modelo tradicional MVC (Model-View-Controller), amplamente utilizado por frameworks conhecidos como Ruby on Rails, Laravel ou Spring. Isso é feito por meio de diretivas de SSR (Renderização no Lado do Servidor), que permitem que os dados sejam processados no servidor e inseridos diretamente no HTML, sem a necessidade de outra aplicação ou sobrecarga do frontend com frameworks complexos.

Ao usar diretivas SSR como ``s-data``, o CMMV garante que dados dinâmicos sejam enviados no próprio HTML, sem depender de chamadas AJAX ou carregamento assíncrono via JavaScript, o que pode atrasar a exibição de conteúdo crítico e impactar negativamente o desempenho da página. Isso torna o conteúdo instantaneamente disponível para motores de busca, melhorando o tempo de carregamento (TTFB) e maximizando as chances de uma boa indexação.

Esse modelo não apenas assegura um carregamento mais rápido, mas também melhora a experiência do usuário final. O usuário recebe uma página completa quase instantaneamente, sem a necessidade de esperar por carregamentos adicionais ou atualizações de conteúdo. Ao centralizar o processamento de dados no backend, o CMMV reduz a complexidade e a sobrecarga no frontend, eliminando a necessidade de configurações adicionais ou execução de múltiplas aplicações para renderizar e servir conteúdo dinâmico.

Em resumo, o CMMV é uma solução moderna que reutiliza o melhor do modelo MVC tradicional, adaptando-o ao cenário atual de otimização para SEO e desempenho, com foco em simplicidade e eficiência.

### s-data

A diretiva ``s-data`` permite renderizar dados do lado do servidor diretamente nos seus templates HTML. Isso é particularmente útil para incorporar variáveis do servidor no HTML sem a necessidade de frameworks adicionais no frontend ou JavaScript.

**Como Funciona**
Você declara uma variável no controlador do lado do servidor e a passa para a visão. A diretiva ``s-data`` é responsável por injetar essa variável diretamente no elemento HTML correspondente durante a renderização no lado do servidor (SSR). Isso garante que o conteúdo seja pré-renderizado, otimizando a página para desempenho e SEO.

**Visão (Template HTML)**
```html
<div s-data="datetime"></div>
```

Neste exemplo, a diretiva ``s-data`` é usada em um elemento ``<div>`` para renderizar a variável ``datetime``, que será fornecida pelo servidor.

**Controlador**
```typescript
res.render("template", { datetime: new Date().getTime() });
```

No controlador, você passa a variável ``datetime`` (o timestamp atual) como parte dos dados renderizados no template.

**Resultado (HTML Renderizado)**
```html
<div>1725568913552</div>
```

### s-attr

A diretiva ``s-attr`` no CMMV permite atribuir atributos a elementos HTML dinamicamente a partir de dados renderizados no servidor, semelhante ao funcionamento da diretiva ``s-data``. No entanto, em vez de preencher o conteúdo de uma tag, ela adiciona ou atualiza um atributo do elemento. Isso é particularmente útil para adicionar atributos de segurança como ``nonce`` em tags ``<script>`` ou ``<link>``, frequentemente exigidos por Políticas de Segurança de Conteúdo (CSP) rigorosas.

**Visão (Template HTML)**
```html
<div s-attr="ref">Conteúdo</div>
```

**Controlador**
```typescript
res.render("template", { ref: "ABC" });
```

**Resultado (HTML Renderizado)**
```html
<div ref="ABC">Conteúdo</div>
```

Neste caso, a diretiva ``s-attr="ref"`` instrui o servidor a criar um atributo ``ref`` no elemento ``<div>`` com o valor "ABC", que é passado pelo controlador do lado do servidor.

### s-i18n

A diretiva ``s-i18n`` é o módulo nativo de internacionalização do sistema CMMV. Ela permite que os desenvolvedores gerenciem e exibam conteúdo multilíngue utilizando arquivos de localidade. Esses arquivos de localidade são armazenados no diretório ``/src/locale`` e devem estar no formato JSON.

O CMMV oferece a capacidade de definir um idioma padrão e permite a troca dinâmica de idioma com base na sessão do usuário. Isso é ideal para sites e aplicações que precisam atender a múltiplos idiomas.

Os arquivos de localidade contêm pares chave-valor, onde a chave representa o identificador da string e o valor é a string localizada. Por exemplo:

**``/src/locale/en.json``**
```json
{
    "welcome": "Bem-vindo ao CMMV",
    "description": "O CMMV torna o desenvolvimento mais rápido e fácil."
}
```

**``/src/locale/pt-br.json``**
```json
{
    "welcome": "Welcome to CMMV",
    "description": "CMMV makes development faster and easier."
}
```

Você pode usar a diretiva ``s-i18n`` nos seus templates de visão para renderizar strings localizadas com base no idioma atual.

**Visão (Template HTML)**
```html
<div s-i18n="welcome"></div>
<p s-i18n="description"></p>
```

**Resultado (Para Localidade em Inglês)**
```html
<div>Welcome to CMMV</div>
<p>CMMV makes development faster and easier.</p>
```

**Configuração**

No arquivo ``.cmmv.config.cjs``, você pode configurar as opções de i18n para definir o caminho dos arquivos de localidade e o idioma padrão da sua aplicação:

```typescript
module.exports = {
    i18n: {
        localeFiles: "./src/locale",  // Caminho para os arquivos de localidade
        default: "en"  // Idioma padrão
    },
    // Outras configurações...
}
```

### s-if

A diretiva ``s-if`` é usada para renderizar condicionalmente um bloco de conteúdo com base na avaliação de uma expressão. Se a expressão fornecida for avaliada como verdadeira, o conteúdo dentro da diretiva ``s-if`` será renderizado. Caso contrário, o conteúdo não será exibido.

```html
<s-if exp="todolist.length > 0">
    <div>Total de Registros Carregados SSR: {{ todolist.length }}</div>
    <s-else>
        <div>Nenhum registro foi carregado via SSR</div>
    </s-else>
</s-if>
```

* ``exp:`` A expressão booleana a ser avaliada. A expressão pode usar variáveis e operadores lógicos para determinar se o bloco deve ser exibido.

Embora a diretiva ``s-if`` ofereça uma maneira poderosa de renderizar conteúdo condicionalmente, ela deve ser usada com cautela devido ao seu impacto potencial em atualizações do lado do cliente. O conteúdo dentro de um bloco ``s-if`` é removido do DOM se a expressão for avaliada como falsa, e essa remoção não é automaticamente atualizada ou re-renderizada no lado do cliente após a renderização inicial no lado do servidor (SSR).

**Recomendações:**
* **Caso de Uso:** É recomendado usar ``s-if`` principalmente para cenários onde condições do lado do servidor determinam o conteúdo que deve ser incluído no HTML inicial. Por exemplo, você pode usar ``s-if`` para carregar e exibir dados disponíveis no momento da renderização no servidor, como dados de autenticação do usuário ou configurações iniciais.

* **Evitar Atualizações Frequentes no Lado do Cliente:** Como ``s-if`` remove o conteúdo do DOM com base na avaliação inicial, ele não é adequado para conteúdo dinâmico do lado do cliente que pode mudar frequentemente ou ser atualizado após o carregamento inicial. Para conteúdo que pode precisar ser atualizado dinamicamente com base em interações do lado do cliente ou mudanças de dados, considere usar outros métodos, como técnicas de renderização do lado do cliente ou frameworks reativos.

Seguindo essas recomendações, você pode garantir que o uso da diretiva ``s-if`` melhore tanto o desempenho quanto a manutenibilidade da sua aplicação sem efeitos colaterais indesejados.

### s-for

A diretiva ``s-for`` foi projetada para lidar com a renderização no lado do servidor (SSR) de listas e garantir que dados dinâmicos sejam pré-renderizados para melhorar o SEO. Esta diretiva, usada em conjunto com a diretiva ``c-for``, permite definir como os dados devem ser renderizados no servidor e como devem ser atualizados no lado do cliente. Ela suporta a pré-renderização de conteúdo que será exibido imediatamente para usuários e motores de busca, enquanto também permite atualizações no lado do cliente.

```html
<s-for
    c-show="condicao"
    c-for="(item, key) in colecao"
    render-tag="nome-da-tag"
>
    <!-- Conteúdo a ser renderizado -->
</s-for>
```

**Como Funciona**

* A diretiva ``s-for`` pré-renderiza os itens da lista no servidor com base nos dados fornecidos. Isso inclui avaliar expressões e injetar conteúdo estático diretamente no HTML.
* A diretiva garante que o conteúdo definido dentro do bloco ``s-for``, incluindo variáveis renderizadas usando ``{{}}``, ``c-text``, ``c-html`` e ``:``, seja pré-renderizado.
* No lado do cliente, a diretiva ``c-for`` assume o controle e lida com atualizações dinâmicas. Isso permite que a lista seja reativa e atualizada conforme necessário.

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
            <label class="todo-item-checked">Tarefa 1</label>
        </div>
        <button
            class="todo-btn-remove"
            @click="DeleteTaskRequest(1)"
        >Remover</button>
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
        >Remover</button>
    </div>
</div>
```

<br/>

* **Pré-renderização:** A diretiva ``s-for`` é crucial para SEO, pois pré-renderiza itens da lista com dados reais, tornando o conteúdo imediatamente disponível para indexação por motores de busca.
* **Atualização no Lado do Cliente:** No lado do cliente, a diretiva ``c-for`` garante que o conteúdo seja reativo e possa ser atualizado dinamicamente.
* **Uso com Cautela:** Como ``s-for`` realiza renderização no lado do servidor, certifique-se de que a lista e seu conteúdo sejam adequados para pré-renderização estática. Evite usá-la para dados altamente dinâmicos que mudam frequentemente no lado do cliente.
Ao combinar ``s-for`` com ``c-for``, você pode aproveitar a renderização no lado do servidor para benefícios de SEO enquanto mantém atualizações dinâmicas no lado do cliente.

## Include

A diretiva ``include`` permite a inserção de componentes ou templates de outros arquivos no layout principal. Esse recurso é útil para construir páginas modulares, possibilitando a divisão do layout em blocos pequenos e reutilizáveis, o que otimiza tanto o desempenho de carregamento quanto a manutenção do código.

### Como Funciona

A diretiva ``include`` é responsável por pré-carregar componentes no lado do servidor antes de enviar a página ao cliente. Esses componentes são tratados como templates, e sua inclusão pode ocorrer de forma em cascata, ou seja, um template pode incluir outros templates. No entanto, esse comportamento é influenciado pela estratégia de cache, garantindo desempenho otimizado ao evitar o carregamento repetido de componentes já processados.

```html
<div>
    <!-- Inclui a barra de navegação -->
    <!-- include('public/views/docs/navbar') -->

    <!-- Inclui o rodapé -->
    <!-- include('public/views/docs/footer') -->
</div>
```

Ao usar a diretiva ``include``, é importante considerar como os dados ou comportamentos de cada template serão gerenciados. A diretiva é projetada para pré-processar componentes no lado do servidor, então qualquer lógica de configuração precisa ser cuidadosamente planejada. A abordagem recomendada é executar a lógica de configuração no template principal e evitar configurações em cascata nos templates incluídos. Isso previne execuções desnecessárias de múltiplas configurações que poderiam impactar negativamente o desempenho.

Para otimizar a injeção de templates e melhorar o desempenho, a diretiva ``include`` utiliza uma estratégia de cache. Isso significa que, após o carregamento inicial de um template, seu conteúdo é armazenado em cache, e inclusões subsequentes usarão o template já carregado. No entanto, se o conteúdo precisar ser atualizado dinamicamente, é recomendado que a configuração seja gerenciada no template principal em vez de depender de configurações individuais em cada template incluído. Isso garante que as alterações sejam centralizadas e facilmente gerenciáveis.

**Exemplo Completo**

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

## Chamada de Serviços

Nesta implementação, você tem uma combinação de diretivas do lado do servidor para pré-carregamento de dados e o uso da diretiva ``include`` para incorporar componentes reutilizáveis, como a barra de navegação e outros elementos de layout, na página. Embora esse padrão funcione de forma eficiente, atualmente ele não suporta configurações específicas de componentes como visto em frameworks como Nuxt.js. Vamos detalhar como essa abordagem funciona e o potencial para melhorias futuras.

**Diretiva do Lado do Servidor (s:docs)**

No arquivo HTML principal (index.hhtml), você está usando uma diretiva (s:docs="docs") para vincular dados pré-carregados no servidor (neste caso, docs) à renderização do lado do cliente. Isso permite que os dados iniciais, como a estrutura de navegação (docs.navbar), sejam buscados no lado do servidor e passados ao cliente antes da renderização da página.

```html
<div id="app" s:docs="docs" c-cloak>
```

O objeto ``docs`` é preenchido no servidor, provavelmente a partir de uma fonte de dados backend ou arquivo, e é injetado no template. Isso permite o uso desses dados imediatamente ao carregar, sem chamadas adicionais de API do lado do cliente, melhorando o desempenho de carregamento inicial.

**Usando a Diretiva include para Templates**
Você está incluindo componentes reutilizáveis, como a barra de navegação, migalhas de pão e rodapé, com a diretiva ``include``. Esses templates são pré-carregados no lado do servidor, aprimorando a modularidade e a manutenibilidade:

```html
<!-- include('public/views/docs/navbar') -->
<!-- include('public/views/docs/breadcrumb'); -->
<!-- include('public/views/docs/footer'); -->
```

Ao carregar esses componentes no lado do servidor, você evita múltiplas requisições HTTP para buscar os templates separadamente e permite que o servidor gerencie a lógica relacionada à inclusão de conteúdo. No entanto, como você observou, essa abordagem processa atualmente apenas a lógica de configuração principal, e qualquer lógica de configuração adicional dentro dos templates incluídos (como navbar.html) não é suportada.

### Atualizações de Estado no Lado do Cliente

Uma vez que os dados e templates são pré-carregados no servidor, o JavaScript do lado do cliente é responsável por gerenciar atualizações de estado. Por exemplo, no seu componente principal, você carrega o estado salvo da barra de navegação do ``localStorage`` e fornece métodos para alternar o estado:

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

Isso garante que interações na página, como expandir ou recolher seções da barra de navegação, persistam entre sessões, pois o estado é armazenado no ``localStorage`` e recarregado quando a página é montada.

### A Limitação

Conforme mencionado no comentário fornecido, atualmente não há suporte para lógicas de configuração individuais dentro dos templates incluídos (como ``navbar.html``). Isso difere de frameworks como Nuxt.js, onde cada componente pode ter sua própria função de configuração. Na sua implementação atual, adicionar um bloco ``script setup`` dentro de um template incluído causaria problemas, pois apenas a lógica de configuração do template principal (``index.html``) é processada.

Por exemplo, adicionar um bloco ``setup`` no ``navbar.html`` não funcionaria como esperado:

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

Isso quebraria a aplicação porque apenas a primeira configuração é processada, e configurações adicionais são ignoradas.

### Melhorias Futuras

A ideia de permitir configurações específicas por componente, semelhante ao Nuxt.js, é uma possível melhoria que poderia aumentar a flexibilidade do sistema. No entanto, isso precisa ser tratado com cuidado para evitar problemas de desempenho, como loops de carregamento infinitos. Quando cada componente tem sua própria lógica de configuração, a aplicação pode acabar em um ciclo contínuo de re-renderização se não for gerenciada adequadamente.

Para implementar configurações específicas por componente de forma segura:

* **Gerenciamento Cuidadoso de Dependências:** Garanta que a lógica de configuração de cada componente seja independente e não provoque re-renderizações desnecessárias do componente pai.
* **Carregamento Preguiçoso:** Considere o carregamento preguiçoso para templates ou componentes que não precisam ser pré-carregados no servidor, reduzindo a carga inicial e evitando problemas potenciais em aplicações grandes.
* **Compartilhamento de Estado Otimizado:** Forneça mecanismos para compartilhar estado entre componentes sem exigir a re-execução completa da lógica de configuração quando os componentes são incluídos dinamicamente.

Aqui está como configurações individuais poderiam funcionar no futuro, seguindo um modelo como o do Nuxt.js:

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
    <li @click="toggleNavbar">Item do Menu</li>
</ul>
```

Nesse cenário futuro, cada componente ou template poderia gerenciar seu próprio estado e métodos sem conflitos com a configuração do template principal.

## Implementação

Abaixo está um exemplo de implementação de praticamente todas as diretivas que foram usadas para criar o sistema nesta documentação.

**``/public/templates/default.html``**
```html
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

**``/public/views/docs/index.html``**
```html
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
                            placeholder="Pesquisar"
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
                            title="Sugerir alteração"
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

**``/public/views/docs/navbar.html``**
```html
<ul class="p-4 select-none top-16" c-cloak c-show="docs">
    <li c-for="(item, key) in docs.navbar">
        <div
            c-show="item"
            class="flex hover:text-blue-700 itemRoot text-white"
            :id="item?.name.replace(/\s/,'_')"
            :data-opened="false"
            @click.stop="navbar[item?.name.replace(/\s/, `_`)] = toggle(navbar[item?.name.replace(/\s/, `_`)], item.name?.replace(/\s/, `_`))"
        >
            <div class="flex flex-1 font-bold text-md cursor-pointer navbar-item">
                <h3 c-if="item && item?.isDir" class="text-white">{{ item?.name }}</h3>
                <span c-else class="text-white">{{ item?.name }}</span>
            </div>

            <div class="justify-between cursor-pointer" c-if="item?.isDir">
                <i class="fa-solid fa-angle-down" c-show="!navbar[item?.name.replace(/\s/, `_`)]"></i>
                <i class="fa-solid fa-angle-up" c-show="navbar[item?.name.replace(/\s/, `_`)]"></i>
            </div>
        </div>

        <ul
            c-if="item && item.children && item.children.length > 0"
            :id="`${item?.name.replace(/\s/, `_`)}_contents`"
            class="p-4 py-1 text-md mb-4"
            :style="(navbar[item?.name.replace(/\s/, `_`)]) ? '' : 'display: none;'"
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

**``/src/docs.controller.ts``**
```typescript
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
		const fullPath = `${dir}/${item}`;

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

Para ter acesso completo ao código, acesse [Github](https://github.com/cmmvio/docs.cmmv.io)

## Ativos

No CMMV, é recomendado que todos os arquivos estáticos, como bibliotecas JavaScript, CSS, fontes e imagens, sejam servidos por meio de uma Rede de Distribuição de Conteúdo (CDN). As CDNs são otimizadas para entrega rápida e eficiente de ativos, reduzindo a latência e melhorando a experiência do usuário. No entanto, se você optar por servir esses arquivos diretamente da sua aplicação, eles devem ser colocados no diretório ``/public``. O módulo ``@cmmv/http`` procura e serve automaticamente arquivos estáticos deste diretório.

Se você escolher não usar uma CDN, coloque todos os seus arquivos estáticos no diretório ``/public``. Este é o diretório onde o ``@cmmv/http`` servirá automaticamente arquivos estáticos como:

### Pacote

Por padrão, a aplicação gerará arquivos complementares, resultando em um pacote final necessário se você estiver usando RPC e reatividade no frontend. Esse pacote será criado como ``/assets/bundle.min.js`` e deve ser incluído nos seus arquivos HTML ou templates para garantir que o frontend funcione corretamente.

**Exemplo de Configuração ``.cmmv.config.cjs`` para Ativos:**

```javascript
module.exports = {
    scripts: [
        { type: "text/javascript", src: "/assets/bundle.min.js", defer: "defer" },
        ...
    ]
};
```

Se você estiver servindo ativos localmente, certifique-se de que seu diretório ``/public`` esteja estruturado da seguinte forma:

```bash
/public
    /assets
        /styles.css
        /app.min.js
        /bundle.min.js
    /images
        /logo.png
    /fonts
        /custom-font.woff2
    /favicon.ico
    /robots.txt
    /sitemap.xml
    /service-worker.js
```

Ao usar esses ativos no seu HTML ou templates, você pode incluir o pacote JavaScript e outros arquivos estáticos assim:

```html
<!DOCTYPE html>
<html lang="pt-br">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aplicação CMMV</title>

    <!-- Incluir Estilos -->
    <link rel="stylesheet" href="/assets/styles.css">

    <!-- Favicon -->
    <link rel="icon" href="/favicon.ico" type="image/x-icon">
</head>
<body>
    <!-- Conteúdo da Aplicação -->

    <!-- Incluir o Pacote -->
    <script src="/assets/bundle.min.js"></script>
</body>
</html>
```

No CMMV, o gerenciamento de ativos é simplificado ao recomendar o uso de CDNs para arquivos estáticos. No entanto, se você optar por servi-los diretamente da aplicação, colocá-los no diretório ``/public`` permitirá que o ``@cmmv/http`` os gerencie automaticamente. Além disso, a aplicação gerará os pacotes necessários para o frontend para lidar com reatividade e recursos RPC, disponíveis como ``/assets/bundle.min.js``. Todos os arquivos estáticos são facilmente acessíveis a partir deste diretório, garantindo uma integração perfeita com sua aplicação.

## Estilos

O novo recurso de estilização no ``@cmmv/view`` permite que os desenvolvedores definam estilos específicos de temas que podem ser alternados dinamicamente no frontend. O sistema mapeia arquivos JSON do diretório ``/public/styles``, que contêm as regras de estilo, e os torna acessíveis no frontend para facilitar o gerenciamento de temas.

Para criar um estilo personalizado, defina um arquivo ``.style.json`` em ``/public/styles``. Cada par chave-valor representa uma classe de estilo, e você pode criar múltiplos temas anexando um sufixo (ex.: .dark) às chaves.

**``docs.style.json``**

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

### Acesso

No seu HTML, você pode acessar os estilos definidos referenciando o arquivo de estilo e as chaves. O objeto de estilos é disponibilizado automaticamente, e as classes específicas do tema são aplicadas com base no tema atual.

```html
<div class="w-60">
    <a
        href="/"
        title="CMMV - Contract Model Model View Framework"
        class="text-white ml-4 flex items-center"
    >
        <img
            src="/assets/favicon/favicon-32x32.png"
            alt="Logo CMMV" height="32" width="32"
        >
        <span :class="$style.docs.title">CMMV</span>
    </a>
</div>
```

### Troca

O sistema permite alternar entre temas (ex.: ``default``, ``dark``). Quando um tema é alterado, todas as classes relevantes com o sufixo (ex.: ``.dark``) são aplicadas automaticamente.

```javascript
toggleTheme() {
    this.$style.switch(
        (this.$style.theme === "default") ?
        "dark" : "default"
    );
}
```

<br/>

* **Estrutura do Arquivo JSON:** O JSON define classes de estilo para diferentes componentes. Classes específicas do tema são sufixadas com ``.dark``, ``.light``, etc.
* **Acesso no Frontend:** Os estilos são acessados por meio de ``styles.[nome-do-arquivo].[chave]`` no frontend.
* **Troca Dinâmica:** A troca de temas é feita programaticamente, e o sistema lida com a substituição das classes de estilo com base no tema ativo.
* **Sem Suporte a Subíndices:** Subíndices aninhados em JSON não são suportados, o que significa que cada par chave-valor deve ser uma entrada plana.

### Gerenciamento

A seleção de temas no ``@cmmv/view`` é gerenciada automaticamente pelo framework, salvando a preferência do usuário no ``localStorage`` e recuperando-a ao carregar a página. Isso permite que o sistema mantenha uma estilização consistente com base na escolha anterior do usuário, sem intervenção manual.

Você pode verificar o tema atual diretamente no seu template usando ``$style.theme``. Por exemplo, para integrar com um componente como DocSearch que requer uma configuração de tema, você pode atualizar a tag HTML com o atributo ``data-theme``:

```html
<!DOCTYPE html>
<html lang="pt-br" :data-theme='$style.theme' scope>
    <head>
        <headers/>
    </head>
</html>
```

Isso garante que o tema correto seja aplicado em componentes como barras de pesquisa e outros elementos que requerem reconhecimento de tema.

Essa implementação de temas dinâmicos no ``@cmmv/view`` oferece uma abordagem simplificada para gerenciar múltiplos temas sem a necessidade de plugins adicionais, extensões do Tailwind ou regras CSS complexas. Ao utilizar arquivos ``.style.json``, os desenvolvedores podem definir facilmente estilos específicos de temas, reduzindo a quantidade de código e garantindo uma troca de temas consistente. Embora seja possível alcançar resultados semelhantes usando variáveis CSS, esse método simplifica o processo, eliminando a necessidade de lógica de visão personalizada ou gerenciamento de CSS para troca de temas, tornando-o intuitivo e eficiente.

* **Suporte a Estilos Aninhados:** Introduzir suporte a subíndices JSON aninhados poderia permitir estruturas de estilo mais complexas e uma melhor organização.
* **Integração com Variáveis CSS:** Uma opção para integrar com variáveis CSS pode complementar o sistema de temas, oferecendo controle mais fino sobre a tematização dinâmica.
* **Pré-carregamento de Tema:** Permitir o pré-carregamento de temas com base nas preferências do sistema (ex.: modo escuro baseado nas configurações do SO) para melhorar a experiência do usuário.
* **Animações Avançadas:** Adicionar transições integradas para troca de temas poderia melhorar a UX ao alternar entre os modos claro e escuro.

### Componente

Além do sistema de estilos discutido anteriormente, o CMMV introduz a propriedade ``$style`` dentro dos componentes, permitindo que estilos com escopo sejam acessados diretamente.

Esse recurso permite que os desenvolvedores definam e referenciem estilos dentro do modelo de dados do componente, facilitando o gerenciamento de estilos nos templates.

```html
<template>
    <div :class="$style.themeSwitch.container">{{ test }}</div>
    <button @click="test++">Adicionar</button>
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

Isso permite um manuseio de estilos mais intuitivo, tornando os estilos acessíveis diretamente no contexto do componente.

## Cabeçalhos

No CMMV, você pode configurar cabeçalhos de resposta para aumentar a segurança, definir metadados ou personalizar o comportamento da sua aplicação web. Isso pode ser feito de duas maneiras: usando o arquivo ``.cmmv.config.cjs`` localizado na raiz do seu projeto ou adicionando configurações específicas diretamente nos arquivos de template para um controle mais granular.

O arquivo ``.cmmv.config.cjs`` serve como o arquivo de configuração global do seu projeto CMMV. Você pode definir cabeçalhos padrão que se aplicarão a todas as respostas da sua aplicação configurando o objeto ``headers`` neste arquivo.

Por exemplo, para configurar uma Política de Segurança de Conteúdo e outros cabeçalhos relacionados à segurança, seu ``.cmmv.config.cjs`` pode parecer assim:

```typescript
module.exports = {
    headers: {
        "Content-Security-Policy": [
            "default-src 'self'",
            "script-src 'self' 'unsafe-eval'",
            "style-src 'self' 'unsafe-inline'",
            "font-src 'self'",
            "connect-src 'self'",
            "img-src 'self' data:"
        ],
        "X-Frame-Options": "DENY",
        "X-Content-Type-Options": "nosniff",
        "Referrer-Policy": "no-referrer",
        "Strict-Transport-Security": "max-age=31536000; ..."
    }
}
```

Para configurações mais específicas, como adicionar metadados ou definir cabeçalhos que se aplicam apenas a uma rota específica, você pode defini-los diretamente nos arquivos de template no estilo Vue.

No bloco ``<script>`` do template, sob ``head``, você pode configurar meta tags, tags de link e outros cabeçalhos:

```html
<script s-setup>
export default {
    layout: "default",

    head: {
        meta: [
            { name: "description", content: "Amostra de Lista de Tarefas CMMV" },
            { name: "keywords", content: "cmmv, modelo de contrato, websocket" }
        ],
        link: [
            { rel: "stylesheet", href: "/assets/styles/todo.css" },
            { rel: "canonical", href: "https://cmmv.io" },
        ],
        script: [
            { src: "https://cdn.jsdelivr.net/npm/some-lib.js", async: true }
        ]
    }

    ...
}
</script>
```

* **meta:** Adiciona ou sobrescreve meta tags no cabeçalho da página, incluindo aquelas para descrição, palavras-chave e Política de Segurança de Conteúdo.
* **link:** Usado para incluir recursos externos como folhas de estilo ou URLs canônicas para SEO.
* **script:** Permite incluir scripts adicionais com atributos personalizados como ``async`` ou ``defer``.

Ao configurar cabeçalhos tanto globalmente quanto no nível do template, você tem controle total sobre o comportamento, segurança e otimização de SEO da sua aplicação CMMV.

### Personalização de Cabeçalhos

Os cabeçalhos também podem ser definidos ou modificados diretamente na configuração do template usando a propriedade ``head``. Isso oferece flexibilidade para adicionar meta tags, links canônicos e outras configurações relacionadas a SEO dinamicamente por página.

```html
<script s-setup>
export default {
    layout: "default",

    head: {
        meta: [
            { name: "description", content: "Amostra de Lista de Tarefas CMMV" },
            { name: "keywords", content: "cmmv, modelo de contrato, websocket" }
        ],
        link: [
            { rel: "stylesheet", href: "/assets/styles/todo.css" },
            { rel: "canonical", href: "https://cmmv.io" },
        ]
    },
}
</script>
```

O módulo ``@cmmv/http`` automatiza otimizações essenciais nos cabeçalhos HTTP, garantindo segurança e desempenho por meio de compressão, gerenciamento de sessões e proteção contra ataques web comuns via middlewares como ``cors``, ``helmet`` e ``compression``. Isso é complementado por configurações personalizadas definidas no arquivo ``.cmmv.config.cjs`` e controles de cabeçalhos específicos de templates, tornando a plataforma CMMV uma base segura e eficiente para aplicações web.

## Visões

As visões no CMMV são arquivos HTML localizados no diretório ``/public/views/``, geralmente com a extensão ``.html``. Elas podem ser renderizadas por controladores ou acessadas diretamente com base no caminho da URL. Por exemplo, uma requisição para ``http://localhost:3000/docs`` será automaticamente mapeada para ``/public/views/docs/index.html``. O arquivo ``index.html`` também pode incluir outras visões por meio da diretiva ``include``, permitindo componentes de visão modulares.

Aqui está um exemplo da estrutura de visão usando a seção ``docs`` da aplicação.

**``docs/index.html``** [Código](https://github.com/cmmvio/docs.cmmv.io/blob/main/public/views/docs/index.html)

```html
<div id="app" s:docs="docs" c-cloak>
    <nav
        class="navbar bg-neutral-800 h-16 top-0 w-full fixed ..."
    >
        <div class="max-w-8xl mx-auto flex container items-center">
            <button
                id="menu-toggle"
                class="text-white text-2xl p-2 lg:hidden ml-2"
            >
                <i class="fa-solid fa-bars"></i>
            </button>

            <div class="w-60">
                <a
                    href="/"
                    title="CMMV - Contract Model Model View Framework"
                    class="text-white ml-4 flex items-center"
                >
                    <img
                        src="/assets/favicon/favicon-32x32.png"
                        alt="Logo CMMV"
                        height="32"
                        width="32"
                        class="w-[32px] h-auto"
                    >
                    <span class="ml-2 text-lg font-semibold">CMMV</span>
                </a>
            </div>

            <div class="justify-between w-full text mr-2">
                <div class="relative text-right ...">
                    <div id="docsearch" class="dark"></div>
                </div>
            </div>

            <div class="justify-between align-middle ...">
                <a
                    href="https://github.com/cmmvio/cmmv"
                    title="Github"
                    target="_blank"
                    class="text-2xl p-2 hover:text-neutral-300"
                >
                    <i class="fa-brands fa-github"></i>
                </a>
            </div>
        </div>
    </nav>

    <div class="flex flex-wrap mx-auto">
        <div
            id="sidebar-menu"
            class="w-60 fixed z-40 overflow-auto leftbar ..."
            c-cloak
        >
            <!-- include('public/views/docs/navbar') -->
        </div>
        <div class="mt-20 ml-64 text-justify relative">
            <!-- include('public/views/docs/anchors'); -->

            <div
                class="lg:pl-[19.5rem] m-4 p-4 px-20 max-w-3x1 mx-auto ..."
                :class="{'w-full': docs.anchors.length < 3}"
            >
                <div class="relative text-white mb-20 context-html">
                    <div c-html="docs.index" s-data="docs.index"></div>

                    <div class="absolute top-0 right-0">
                        <a :href="`https://github.com/...`"
                           target="_blank"
                           title="Sugerir alteração"
                        >
                            <i class="fa-solid fa-pen-to-square fa-lg"></i>
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</div>
```

As visões são mapeadas com base no caminho da URL. Quando uma requisição é feita para um caminho específico como ``/docs``, o sistema automaticamente procura por ``/public/views/docs/index.html``. Se encontrado, ele é renderizado; caso contrário, um erro 404 é retornado.

### Diretiva Include

A diretiva ``include`` permite a inclusão de outros arquivos de visão, possibilitando modularidade nas visões. Por exemplo, ``docs/index.html`` inclui ``docs/navbar.html`` usando:

```html
<!-- include('public/views/docs/navbar') -->
```

Essa abordagem permite reutilizar componentes como cabeçalhos, rodapés e barras laterais em diferentes visões.

### Configuração e Vinculação de Dados

Componentes de visão podem ser controlados usando a diretiva ``s-setup``. No momento, apenas o ``index.html`` (ou visão raiz) pode usar a tag ``s-setup`` para configurar scripts, meta tags ou outras configurações. Configurações adicionadas em subcomponentes são ignoradas. Isso significa que dados ou configurações devem ser passados a partir da visão de nível superior.

**Exemplo:**

```javascript
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

O CMMV introduz um conceito de scripts de configuração nas visões, semelhante ao que você encontra em frameworks como [Vue.js](https://vuejs.org/) e [Nuxt.js](https://nuxt.com/). Esse conceito permite a configuração dinâmica de cabeçalhos, dados estruturados e possibilita a vinculação de dados ao frontend. Além disso, os scripts de configuração fornecem *hooks* de ciclo de vida, como ``mounted`` e ``created``, que são executados quando o frontend é carregado.

### Layout Dinâmico

A propriedade ``layout`` permite especificar o layout que a visão usará. Neste exemplo, o layout é definido como ``"default"``, o que significa que a visão será herdada e renderizada dentro de um layout base, frequentemente definido em ``/public/templates``.

```javascript
layout: "default"
```

### *Hooks* de Ciclo de Vida

Os scripts de configuração do CMMV fornecem *hooks* de ciclo de vida semelhantes ao Vue.js. Esses *hooks* permitem controlar a execução de código em diferentes estágios do ciclo de vida do componente:

* **mounted:** Executado quando a visão é completamente montada no DOM. Normalmente usado para tarefas como manipulação de DOM ou requisições de API.
* **created:** Pode ser usado para executar código assim que a visão é criada, antes de ser montada no DOM.

```javascript
async mounted() {
    this.loadState();
}
```

### Propriedade Data

Você pode definir a função ``data()`` para retornar um objeto que contém dados reativos, que podem ser vinculados à visão. Esses dados serão atualizados automaticamente quando modificados.

```javascript
data() {
    return { navbar: [] }
}
```

Neste exemplo, ``navbar`` é inicializado como um array vazio e posteriormente preenchido usando o método ``loadState()``.

### Métodos

Os scripts de configuração permitem a inclusão de métodos que são acessíveis no escopo da visão. Esses métodos são incorporados ao contexto do framework e podem ser usados dentro do template ou como manipuladores de eventos para interações na interface do usuário.

```javascript
methods: {
    loadState() {
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
```

### Cabeçalhos

O script de configuração também pode ser usado para configurar dinamicamente cabeçalhos e scripts para a visão. Usando propriedades como ``head``, você pode definir meta tags, links (por exemplo, para folhas de estilo) e outros elementos dinamicamente:

```javascript
head: {
    meta: [
        { name: "description", content: "Amostra de Lista de Tarefas CMMV" },
        { name: "keywords", content: "cmmv, modelo de contrato, websocket" }
    ],
    link: [
        { rel: "stylesheet", href: "/assets/styles/todo.css" },
        { rel: "canonical", href: "https://cmmv.io" },
    ]
}
```

### Vinculação de Dados

A configuração fornece dados e métodos que são diretamente acessíveis para vinculação de dados no frontend. Isso permite uma interação contínua com componentes da interface do usuário e atualizações dinâmicas. Por exemplo:

```html
<div c-html="docs.index" s-data="docs.index"></div>
```

Isso vincula o conteúdo de ``docs.index`` ao HTML, permitindo a renderização dinâmica com base no estado de ``docs.index``.

## Templates

O CMMV oferece um sistema de templates flexível, permitindo configurar templates de página mestra para visões modulares. Essa configuração ajuda a criar layouts reutilizáveis em toda a aplicação, simplificando a estrutura e garantindo consistência em visões específicas.

Um template de página mestra define o layout HTML base das suas visões e é armazenado no diretório ``/public/templates``. Ele serve como uma base onde conteúdo dinâmico, cabeçalhos e scripts são injetados. A estrutura segue um formato consistente que se parece com isso:

```html
<!DOCTYPE html>
<html lang="pt-br" data-theme='dark' class="dark">
    <head>
        <headers/>
    </head>
    <body scope>
        <slot/>
        <scripts/>
    </body>
</html>
```

### Elementos Chave

* **Tag ``<headers/>``:**

* Esta tag é usada para injetar todos os cabeçalhos processados do módulo ``@cmmv/view``, que podem incluir metadados, folhas de estilo e outros elementos configurados nas suas visões ou globalmente.
* Esses cabeçalhos vêm do seu ``.cmmv.config.cjs`` ou de configurações personalizadas definidas em cada visão.

* **Tag ``<slot/>``:**

* O elemento ``<slot/>`` atua como um espaço reservado para o conteúdo principal da sua visão.
* Ao renderizar uma visão, o conteúdo dessa visão é injetado dinamicamente no slot.

* **Tag ``<scripts/>``:**

* Esta tag gerencia a inclusão de arquivos JavaScript configurados no seu ``.cmmv.config.cjs`` ou diretamente na visão.
* Isso garante que todos os scripts necessários do lado do cliente sejam incluídos na renderização final da página.

* **Outras Tags Personalizadas:**

* Quaisquer tags ou atributos adicionais que você definir no seu template serão preservados durante o processo de renderização.
* Certifique-se de que todos os recursos externos (links, scripts) incluam o atributo ``nonce="{ nonce }"`` ou ``s-attr="nonce"`` para segurança, conforme exigido pelas configurações de Política de Segurança de Conteúdo (CSP) no CMMV.

Todos os templates de página mestra são armazenados no diretório ``/public/templates``. Aqui está um exemplo de uma possível estrutura de diretórios:

```bash
/public
    /templates
        /default.html
        /admin.html
        /dashboard.html
```

### Definindo uma Visão

Na sua visão, você pode configurar qual template mestre usar. Isso é feito na seção ``s-setup`` da visão. Aqui está um exemplo de configuração de visão que usa um layout personalizado e injeta scripts:

```html
<script s-setup>
export default {
    layout: "admin",  // Referência ao arquivo /public/templates/admin.html

    head: {
        meta: [
            { name: "description", content: "Painel Administrativo" },
            { name: "keywords", content: "admin, cmmv, dashboard" }
        ],
        link: [
            { rel: "stylesheet", href: "/assets/styles/admin.css" },
            { rel: "canonical", href: "https://admin.cmmv.io" }
        ]
    },

    scripts: [
        { src: "/assets/js/admin-dashboard.js", async: true }
    ]
}
</script>
```

O arquivo ``.cmmv.config.cjs`` permite gerenciar globalmente JavaScript e folhas de estilo que devem ser incluídos nas suas visões. Eles serão injetados nas tags ``<scripts/>`` e ``<headers/>`` dos seus templates mestres.

**Exemplo ``.cmmv.config.cjs:``**

```javascript
module.exports = {
    headers: {
        "Content-Security-Policy": [
            "default-src 'self'",
            "script-src 'self' 'unsafe-eval'",
            "style-src 'self' 'unsafe-inline'",
            "font-src 'self'",
            "connect-src 'self'",
            "img-src 'self' data:"
        ]
    },

    assets: {
        scripts: [
            { src: "/assets/js/main.js", async: true },
            { src: "/assets/js/extra.js", defer: true }
        ],
        styles: [
            { rel: "stylesheet", href: "/assets/css/main.css" }
        ]
    }
};
```

Após configurar uma visão com um template mestre, a página renderizada pode parecer assim:

```html
<!DOCTYPE html>
<html lang="pt-br" data-theme='dark' class="dark">
    <head>
        <meta name="description" content="Painel Administrativo">
        <meta name="keywords" content="admin, cmmv, dashboard">
        <link
            rel="stylesheet"
            href="/assets/styles/admin.css"
            nonce="a1b2c3"
        />
        <link rel="canonical" href="https://admin.cmmv.io" />
    </head>
    <body scope>
        <!-- Conteúdo principal da visão -->
        <div id="dashboard">
            <h1>Bem-vindo ao Painel Administrativo</h1>
        </div>

        <!-- Scripts -->
        <script
            src="/assets/js/admin-dashboard.js"
            async
            nonce="a1b2c3"
        ></script>
    </body>
</html>
```

No CMMV, você pode modularizar suas visões definindo templates de página mestra localizados em ``/public/templates``. Esses templates gerenciam a injeção de cabeçalhos, conteúdo dinâmico e scripts por meio das tags ``<headers/>``, ``<slot/>`` e ``<scripts/>``, respectivamente. Ao garantir que os protocolos de segurança sejam mantidos com atributos como ``nonce="{ nonce }"``, sua aplicação permanece segura enquanto serve ativos de forma eficiente. Por meio dessa abordagem, você pode criar layouts reutilizáveis e manter consistência em diferentes seções da sua aplicação, aumentando a velocidade de desenvolvimento e a manutenibilidade.
