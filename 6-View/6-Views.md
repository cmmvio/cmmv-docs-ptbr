# Views

As views no CMMV são arquivos HTML localizados no diretório `/public/views/`, geralmente com a extensão .html. Elas podem ser renderizadas por controladores ou acessadas diretamente com base no caminho da URL. Por exemplo, uma requisição para `http://localhost:3000/docs` será automaticamente mapeada para `/public/views/docs/index.html`. O arquivo `index.html` também pode incluir outras views por meio da diretiva `include`, permitindo componentes de view modulares.

Aqui está um exemplo de estrutura de uma view usando a seção `docs` da aplicação.

**`docs/index.html`** [Código](https://github.com/cmmvio/docs.cmmv.io/blob/main/public/views/docs/index.html)

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
                        alt="CMMV Logo" 
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
                           title="Sugira alterações"
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

**`docs/navbar.html`** [Código](https://github.com/cmmvio/docs.cmmv.io/blob/main/public/views/docs/navbar.html)
```html
<ul class="p-4 select-none top-16" c-cloak c-show="docs">
    <s-for c-for="(item, key) in docs.navbar" render-tag="li">
        <div c-show="item"
            class="flex hover:text-blue-700 itemRoot text-white"
            :id="item?.name.replace(/\s/,'_')" 
            :data-opened="false"
            @click.stop="navbar[item?.name.replace(/\s/, `_`)] = toggle(...">
            <div class="flex flex-1 font-bold ...">
                <h3 
                    c-if="item && item?.isDir" 
                    class="text-white"
                >{{ item?.name }}</h3>
                <span c-else class="text-white">{{ item?.name }}</span>
            </div>
            <div class="justify-between cursor-pointer" c-if="item?.isDir">
                <i :class="navbar[item?.name...."></i>
            </div>
        </div>
        <ul c-if="item && item.children && item.children.length > 0"
            :id="`${item?.name.replace(/\s/, `_`)}_contents`"
            class="p-4 py-1 text-md mb-4"
            :style="(navbar[item?.name.replace(/\s/, `_`)]) ? '' : ...">
            <li c-for="(child) in item.children">
                <div class="hover:text-..." style="font-size: 12px">
                    <a :href="child.uri" class="text-base">{{ child.name }}</a>
                </div>
            </li>
        </ul>
    </s-for>
</ul>
```

As views são mapeadas com base no caminho da URL. Quando uma requisição é feita para um caminho específico como /docs, o sistema automaticamente procura por `/public/views/docs/index.html`. Se encontrado, é renderizado; caso contrário, um erro 404 é retornado.

## Diretiva Include

A diretiva `include` permite a inclusão de outros arquivos de view, possibilitando modularidade nas views. Por exemplo, `docs/index.html` inclui `docs/navbar.html` usando:

```html
<!-- include('public/views/docs/navbar') -->
```

Essa abordagem permite reutilizar componentes como cabeçalhos, rodapés e barras laterais em diferentes views.

## Configuração e Binding de Dados

Os componentes das views podem ser controlados usando a diretiva `s-setup`. No entanto, no momento, apenas o `index.html` (ou view raiz) pode usar a tag `s-setup` para configurar scripts, metatags ou outras configurações. Qualquer configuração adicionada em subcomponentes será ignorada. Isso significa que dados ou configurações precisam ser passados da view de nível superior.

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
}
</script>
```

CMMV introduz um conceito de scripts de configuração nas views, semelhante ao que você encontra em frameworks como [Vue.js](https://vuejs.org/) e [Nuxt.js](https://nuxt.com/). Esse conceito permite configurar cabeçalhos dinamicamente, dados estruturados e habilita o binding de dados ao frontend. Além disso, os scripts de configuração fornecem hooks de ciclo de vida, como `mounted` e `created`, que são executados quando o frontend é carregado.

## Layout Dinâmico 

A propriedade `layout` permite especificar o layout que a view usará. Neste exemplo, o layout é definido como `"default"`, o que significa que a view herdará e será renderizada dentro de um layout base, geralmente definido em `/public/templates`.

```javascript
layout: "default"
```

## Hooks de Ciclo de Vida

Os scripts de configuração do CMMV fornecem hooks de ciclo de vida semelhantes ao Vue.js. Esses hooks permitem controlar a execução do código em diferentes estágios do ciclo de vida do componente:

* **mounted:** Executa quando a view está totalmente montada no DOM. Tipicamente usado para tarefas como manipulação do DOM ou requisições de API.
* **created:** Pode ser usado para executar código assim que a view é criada, antes de ser montada no DOM.

```javascript
async mounted() {
    this.loadState();
}
```

## Propriedade Data

Você pode definir a função `data()` para retornar um objeto que contém dados reativos, que podem ser vinculados à view. Esses dados serão automaticamente atualizados quando modificados.

```javascript
data() {
    return { navbar: [] }
}
```

Neste exemplo, `navbar` é inicializado como um array vazio e é posteriormente preenchido usando o método `loadState()`.

## Métodos

Os scripts de configuração permitem a inclusão de métodos que estão acessíveis no escopo da view. Esses métodos são incorporados ao contexto do framework e podem ser usados dentro do template ou como manipuladores de eventos para interações da interface.

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

## Cabeçalhos

O script de configuração também pode ser usado para configurar dinamicamente cabeçalhos e scripts para a view. Usando propriedades como `head`, você pode definir metatags, links (por exemplo, para folhas de estilo) e outros elementos dinamicamente:

```javascript
head: {
    meta: [
        { name: "description", content: "Exemplo de Todolist CMMV" },
        { name: "keywords", content: "cmmv, contract model, websocket" }
    ],
    link: [
        { rel: "stylesheet", href: "/assets/styles/todo.css" },
        { rel: "canonical", href: "https://cmmv.io" },
    ]
}
```

## Binding de Dados

A configuração fornece dados e métodos que são diretamente acessíveis para binding de dados no frontend. Isso permite uma interação perfeita com os componentes da interface e atualizações dinâmicas. Por exemplo:

```html
<div c-html="docs.index" s-data="docs.index"></div>
```

Isso vincula o conteúdo de `docs.index` ao HTML, permitindo uma renderização dinâmica com base no estado de `docs.index`.
