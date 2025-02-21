# View

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
        A partir da versão <strong>0.8.3</strong>, o módulo <strong>@cmmv/view</strong> foi totalmente desacoplado do sistema HTTP, tornando-se um módulo opcional para renderização do lado do servidor (SSR).
Ele continua a funcionar com o servidor padrão e o Express, mas agora deve ser explicitamente habilitado nas configurações e instalado separadamente.
    </p>
    <p>
        Essa alteração permite maior flexibilidade, permitindo que os projetos escolham se desejam incluir o módulo de visualização com base em suas necessidades específicas, reduzindo dependências e melhorando a modularidade.
    </p>
</div>

O módulo `@cmmv/view` no CMMV é um motor de visualização personalizado projetado para otimizar SEO e desempenho, incorporando renderização no lado do servidor (SSR) com integração perfeita a frameworks frontend modernos. Construído sobre o EJS (Embedded JavaScript), ele funciona como um middleware para Express e Fastify, processando visualizações em tempo real e injetando dados pré-carregados no HTML antes que ele chegue ao navegador. Essa abordagem permite que os mecanismos de busca indexem conteúdo já processado, enquanto ainda oferece flexibilidade para usar frameworks frontend como Vue.js, React ou Angular para interatividade no lado do cliente.

Os frameworks tradicionais de renderização no lado do cliente (CSR), como Vue.js e React, geram conteúdo dinamicamente no navegador. Embora esses frameworks proporcionem interatividade rica, podem impactar negativamente o SEO devido ao atraso na renderização do conteúdo—os mecanismos de busca podem não indexar totalmente o conteúdo dinâmico que depende da execução de JavaScript.

Ao incorporar SSR com o `@cmmv/view`, o conteúdo crítico da sua aplicação web é renderizado no servidor, permitindo que ele seja imediatamente visível tanto para usuários quanto para mecanismos de busca. Isso melhora significativamente o SEO do seu site, pois o conteúdo é apresentado na resposta inicial do HTML, facilitando a indexação e proporcionando carregamentos mais rápidos.

## Principais Recursos

### 1. **Renderização no Lado do Servidor (SSR)**
O módulo `@cmmv/view` processa visualizações e injeta dados dinâmicos no servidor antes que o HTML seja enviado ao navegador. Isso significa que qualquer chamada de serviço, consulta ao banco de dados ou solicitação à API necessária para gerar o conteúdo é completada no servidor, permitindo que o usuário e os mecanismos de busca recebam um HTML totalmente renderizado.

### 2. **Bundle de JavaScript Otimizado**
Além de renderizar visualizações, o módulo `@cmmv/view` cria um bundle de JavaScript otimizado que inclui as bibliotecas principais necessárias para a aplicação, como `@cmmv/reactivity` (um componente que gerencia atualizações em tempo real via RPC). Esse bundle garante que o frontend esteja preparado para interações rápidas e eficientes assim que atinge o cliente.

### 3. **Integração com Frameworks JavaScript**
Embora `@cmmv/view` foque em SSR, ele não impede o uso de frameworks modernos como Vue.js, React ou Angular. Ele fornece uma maneira de hidratar o HTML renderizado no servidor com esses frameworks, permitindo que os desenvolvedores se beneficiem tanto de SSR quanto da interatividade no lado do cliente.

### 4. **Dados Pré-Carregados para SEO**
Uma das maiores vantagens para SEO é a capacidade de injetar dados diretamente no template HTML. Esses dados são frequentemente obtidos de serviços como APIs, bancos de dados ou repositórios, tornando-os disponíveis para os crawlers antes que qualquer JavaScript seja executado. Isso garante que o conteúdo essencial esteja disponível imediatamente, melhorando os rankings nos mecanismos de busca.

### 5. **Inclusão Dinâmica de JavaScript**
Além de lidar com SSR, `@cmmv/view` inclui dinamicamente qualquer arquivo JavaScript adicional necessário para a página. Seja um script personalizado ou um framework como React, esses arquivos são empacotados e servidos em uma única solicitação, reduzindo a sobrecarga da rede e acelerando os tempos de carregamento iniciais.

### 6. **Diretivas Personalizadas para SSR**
O módulo suporta diretivas personalizadas que processam HTML e dados no lado do servidor. Essas diretivas gerenciam pré-carregamento de dados, conteúdo condicional, loops sobre conjuntos de dados e traduções i18n—todos renderizados como HTML estático, melhorando ainda mais o SEO.

## SEO

### Melhor Crawlability
Com `@cmmv/view`, o conteúdo já está pré-renderizado ao atingir o navegador. O servidor processa todas as solicitações de serviço, chamadas à API e consultas dinâmicas de dados, entregando um HTML totalmente renderizado. Isso permite que os mecanismos de busca identifiquem imediatamente o conteúdo principal das páginas, levando a classificações mais altas nos resultados de pesquisa.

### Carregamentos de Página Mais Rápidos
Como `@cmmv/view` processa os dados no servidor e os envia com o HTML inicial, os usuários experimentam carregamentos de página mais rápidos. O tempo gasto renderizando o conteúdo no navegador é minimizado, melhorando tanto a experiência do usuário quanto o desempenho SEO da página.

### Redução de Sobrecarga de JavaScript
Frameworks de renderização no lado do cliente frequentemente envolvem grandes bundles de JavaScript e execução significativa no runtime. Com `@cmmv/view`, grande parte do trabalho é transferida para o servidor, reduzindo a quantidade de JavaScript necessária para processar a página no lado do cliente.

# Instalação

Instale o pacote ``@cmmv/view`` via npm:

```bash
$ pnpm add @cmmv/view
```

## Exemplo

```html
<div
    class="product-page"
    s:product="services.product.getProductById(productId)"
    scope
>
    <h1>{{ product.name }}</h1>
    <p>{{ product.description }}</p>

    <div class="price">{{ product.price | currency }}</div>

    <div class="availability">
        {{ product.inStock ? 'In Stock' : 'Out of Stock' }}
    </div>

    <button
        s-i18n="addToCart"
        class="add-to-cart"
        @click="addToCart(product.id)"
    ></button>
</div>

<script s-setup>
export default {
    layout: "default",

    data() {
        return {
            product: null
        }
    },

    methods: {
        addToCart(productId) {
            ...
        }
    }
}
</script>
```

Com o `@cmmv/view`, os desenvolvedores podem aproveitar as vantagens do SSR para impulsionar significativamente o desempenho SEO e reduzir os tempos de carregamento. Com conteúdo pré-carregado, bundles de JavaScript otimizados e integração perfeita com frameworks frontend modernos, `@cmmv/view` oferece uma solução abrangente para construir aplicações web escaláveis e otimizadas para SEO.

## Configurações

O módulo `@cmmv/view` permite personalização flexível através do arquivo `.cmmv.config.cjs`. Abaixo estão as configurações disponíveis para ajustar o comportamento do motor de visualização, internacionalização (i18n), metadados para SEO, cabeçalhos de segurança e recursos JavaScript.

| Configuração       | Descrição                                                                                          | Exemplo                                                   |
|-------------------|--------------------------------------------------------------------------------------------------|-----------------------------------------------------------|
| `server.publicDirs` | Define os diretórios públicos que serão servidos estaticamente pelo servidor.                     | `["public", "public/views"]`                              |
| `server.render`    | Especifica o módulo de renderização SSR (ex: `@cmmv/view`). Deve ser instalado separadamente.       | `"@cmmv/view"`                                            |
| `i18n.localeFiles` | Caminho para os arquivos de localização (traduções) utilizados no projeto.                         | `"./src/locale"`                                          |
| `i18n.default`     | Define o idioma padrão para a aplicação.                                                           | `"en"`                                                    |
| `view.extractInlineScript` | Se ativado, extrai scripts inline para arquivos separados para melhorar o desempenho.       | `true`                                                    |
| `view.minifyHTML`  | Minifica o HTML renderizado para melhorar a performance e reduzir o tamanho da resposta.           | `true`                                                    |
| `view.scriptsTimestamp` | Adiciona timestamps aos scripts para evitar problemas de cache.                               | `false`                                                   |
| `head.title`       | Define o título da aplicação que aparecerá na aba do navegador.                                   | `"CMMV"`                                                  |
| `head.htmlAttrs`   | Define atributos globais para a tag `<html>`, como idioma.                                         | `{ lang: "pt-br" }`                                       |
| `head.meta`        | Lista de metadados a serem inseridos na página, como charset e viewport.                           | `[{ charset: "utf-8" }, { name: "viewport", content: "width=device-width" }]` |
| `head.link`        | Define links externos, como favicon e fontes.                                                      | `[{ rel: "icon", href: "assets/favicon/favicon.ico" }]`   |
| `headers`          | Define os cabeçalhos HTTP para segurança e controle de políticas de conteúdo.                     | `{ "Content-Security-Policy": ["default-src 'self'"] }`   |
| `scripts`          | Lista de scripts que serão carregados na aplicação, como bundles de JavaScript externos.           | `[{ type: "text/javascript", src: "/assets/bundle.min.js" }]` |

### Configurações Disponíveis:

```typescript
module.exports = {
    ...

    server: {
        ...
        publicDirs: ["public", 'public/views'],
        render: "@cmmv/view",
    },

    i18n: {
        localeFiles: "./src/locale",
        default: "en"
    },

    view: {
        extractInlineScript: true,
        minifyHTML: true,
        scriptsTimestamp: false
    },

    head: {
        title: "CMMV",
        htmlAttrs: {
            lang: "pt-br"
        },
        meta: [
            { charset: 'utf-8' },
            { name: 'viewport', content: 'width=device-width' },
            ...
        ],
        link: [
            { rel: 'icon', href: 'assets/favicon/favicon.ico' },
            ...
        ]
    },

    headers: {
        "Content-Security-Policy": [
            "default-src 'self'",
            "script-src 'self' 'unsafe-eval'",
            "style-src 'self' 'unsafe-inline'",
            "font-src 'self'"
        ],
        ...
    },

    scripts: [
        { type: "text/javascript", src: '/assets/bundle.min.js', defer: "defer" }
    ]
}
```

### Configuração de i18n

Gerencie internacionalização (i18n) na sua aplicação CMMV definindo o diretório de arquivos de localização e configurando um idioma padrão.

```typescript
i18n: {
    localeFiles: "./src/locale",
    default: "en"
}
```

<br/>

* **localeFiles:** Diretório onde os arquivos de tradução para diferentes idiomas são armazenados.
* **default:** O idioma padrão usado quando nenhum outro idioma específico é selecionado.

### Configuração de Head

Controle a seção `<head>` do seu documento HTML, definindo tags meta, atributos para a tag `<html>`, e links como favicons.

```typescript
head: {
    title: "CMMV",
    htmlAttrs: { lang: "pt-br" },
    meta: [
        { charset: 'utf-8' },
        { name: 'viewport', content: 'width=device-width' },
        ...
    ],
    link: [
        { rel: 'icon', href: 'assets/favicon/favicon.ico' },
        ...
    ]
}
```

<br/>

* **title:** O título padrão da página.
* **htmlAttrs:** Atributos para a tag `<html>`, como o idioma.
* **meta:** Metatags personalizadas.
* **link:** Links externos como favicons ou folhas de estilo.

### Configuração de Headers

Defina cabeçalhos HTTP como `Content-Security-Policy` (CSP) para proteger sua aplicação.

```typescript
headers: {
    "Content-Security-Policy": [
        "default-src 'self'",
        "script-src 'self' 'unsafe-eval'",
        "style-src 'self' 'unsafe-inline'",
        "font-src 'self'"
    ],
    ...
}
```

### Configuração de Scripts

Especifique arquivos JavaScript externos ou internos que devem ser incluídos na saída HTML.

```typescript
scripts: [
    { type: "text/javascript", src: '/assets/bundle.min.js', defer: "defer" }
]
```

<br/>

* **type:** Define o tipo do arquivo (geralmente `text/javascript`).
* **src:** O caminho do arquivo JavaScript.
* **defer:** Adia a execução do script até que o HTML tenha sido totalmente analisado.

## Conclusão

O módulo `@cmmv/view` oferece uma solução completa para otimizar aplicações web, combinando a eficiência da renderização no lado do servidor (SSR) com a flexibilidade de frameworks modernos. Suas funcionalidades, como injeção de dados pré-carregados, bundles de JavaScript otimizados e suporte a internacionalização, tornam o desenvolvimento mais ágil e o desempenho superior.

Com configurações personalizáveis, o `@cmmv/view` capacita os desenvolvedores a criar aplicações escaláveis e amigáveis para SEO, mantendo uma experiência de usuário rica e interativa.
