# Templates

O CMMV oferece um sistema de templates flexível, permitindo configurar templates de página mestre para visões modulares. Essa configuração ajuda a criar layouts reutilizáveis em toda a sua aplicação, simplificando a estrutura e garantindo consistência em visões específicas.

Um template de página mestre define o layout HTML base das suas visões e é armazenado no diretório `/public/templates`. Ele serve como uma fundação onde conteúdos dinâmicos, cabeçalhos e scripts são injetados. A estrutura segue um formato consistente, como este:

```html
<!DOCTYPE html>
<html lang="pt-BR" data-theme='dark' class="dark">
    <head>
        <headers/>
    </head>
    <body scope> 
        <slot/>
        <scripts/>
    </body>
</html>
```

## Elementos Chave

* **Tag `<headers/>`:**

    * Essa tag é usada para injetar todos os cabeçalhos processados pelo módulo `@cmmv/view`, que podem incluir metadados, folhas de estilo e outros elementos configurados dentro de suas visões ou globalmente.
    * Esses cabeçalhos vêm do seu arquivo `.cmmv.config.js` ou de configurações personalizadas definidas em cada visão.

* **Tag `<slot/>`:**

    * O elemento `<slot/>` atua como um espaço reservado para o conteúdo principal da sua visão.
    * Ao renderizar uma visão, o conteúdo dessa visão é dinamicamente injetado no slot.

* **Tag `<scripts/>`:**

    * Essa tag lida com a inclusão de arquivos JavaScript configurados no seu arquivo `.cmmv.config.js` ou diretamente dentro da própria visão.
    * Isso garante que todos os scripts necessários no lado do cliente sejam incluídos na renderização final da página.

* **Outras Tags Personalizadas:**

    * Quaisquer tags ou atributos adicionais definidos no seu template serão preservados durante o processo de renderização.
    * Certifique-se de que todos os recursos externos (links, scripts) incluam o atributo `nonce="{ nonce }"` ou `s-attr="nonce"` para segurança, conforme exigido pelas configurações de Content Security Policy (CSP) no CMMV.

Todos os templates de página mestre são armazenados no diretório `/public/templates`. Aqui está um exemplo de estrutura de diretórios possível:

```bash
/public
    /templates
        /default.html
        /admin.html
        /dashboard.html
```

## Definindo uma Visão

Na sua visão, você pode configurar qual template mestre usar. Isso é feito na seção `s-setup` da visão. Aqui está um exemplo de configuração de visão que usa um layout personalizado e injeta scripts:

```html
<script s-setup>
export default {
    layout: "admin",  // Referência ao arquivo /public/templates/admin.html

    head: {
        meta: [
            { name: "description", content: "Painel Administrativo" },
            { name: "keywords", content: "admin, cmmv, painel" }
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

O arquivo `.cmmv.config.js` permite gerenciar globalmente JavaScript e folhas de estilo que devem ser incluídos em suas visões. Esses serão injetados nas tags `<scripts/>` e `<headers/>` dos seus templates mestre.

**Exemplo de `.cmmv.config.js:`**

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

Após configurar uma visão com um template mestre, a página renderizada pode se parecer com isto:

```html
<!DOCTYPE html>
<html lang="pt-BR" data-theme='dark' class="dark">
    <head>
        <meta name="description" content="Painel Administrativo">
        <meta name="keywords" content="admin, cmmv, painel">
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

No CMMV, você pode modularizar suas views definindo templates de página mestre localizados em `/public/templates`. Esses templates lidam com a injeção de cabeçalhos, conteúdo dinâmico e scripts por meio das tags `<headers/>`, `<slot/>` e `<scripts/>`, respectivamente. Garantindo que protocolos de segurança sejam mantidos com atributos como `nonce="{ nonce }"`, sua aplicação permanece segura enquanto serve assets de maneira eficiente. Por meio dessa abordagem, você pode criar layouts reutilizáveis e manter a consistência em diferentes seções da sua aplicação, aumentando tanto a velocidade de desenvolvimento quanto a manutenção.
