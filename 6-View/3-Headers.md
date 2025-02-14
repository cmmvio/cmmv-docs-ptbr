# Cabeçalhos

No CMMV, você pode configurar os cabeçalhos de resposta para aumentar a segurança, definir metadados ou personalizar o comportamento do seu aplicativo web. Isso pode ser feito de duas maneiras: usando o arquivo `.cmmv.config.cjs` localizado na raiz do seu projeto ou adicionando configurações específicas diretamente nos arquivos de template para controle mais granular.

O arquivo `.cmmv.config.cjs` serve como o arquivo de configuração global para o seu projeto CMMV. Você pode definir cabeçalhos padrão que serão aplicados a todas as respostas da sua aplicação definindo o objeto de cabeçalhos neste arquivo.

Por exemplo, para configurar uma Content-Security-Policy e outros cabeçalhos relacionados à segurança, seu `.cmmv.config.cjs` pode ser assim:

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

No bloco `<script>` do template, em `head`, você pode configurar metatags, links e outros cabeçalhos:

```html
<script s-setup>
export default {
    layout: "default",

    head: {
        meta: [
            { name: "description", content: "Exemplo de Todolist no CMMV" },
            { name: "keywords", content: "cmmv, contract model, websocket" }
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

* **meta:** Adiciona ou sobrescreve metatags no `head` da página, incluindo aquelas para descrição, palavras-chave e Content-Security-Policy.
* **link:** Usado para incluir recursos externos, como folhas de estilo ou URLs canônicas para SEO.
* **script:** Permite incluir scripts adicionais com atributos personalizados, como `async` ou `defer`.

Ao configurar cabeçalhos globalmente e no nível do template, você tem controle total sobre o comportamento, segurança e otimização de SEO do seu aplicativo CMMV.

# Módulo HTTP

O módulo `@cmmv/http` fornece uma maneira automática de gerenciar e otimizar os cabeçalhos enviados com as respostas HTTP, reduzindo o tamanho de cabeçalhos desnecessários com base no tipo de solicitação (por exemplo, `GET`, `POST`, `PUT` ou `DELETE`). Essa funcionalidade embutida garante que apenas os cabeçalhos relevantes sejam incluídos, minimizando a sobrecarga e melhorando o desempenho.

```javascript
if (req.method === 'GET') {
    res.setHeader(
        'Strict-Transport-Security',
        'max-age=15552000; includeSubDomains',
    );
    res.setHeader('X-Content-Type-Options', 'nosniff');
    res.setHeader('X-Frame-Options', 'SAMEORIGIN');
    res.setHeader('X-XSS-Protection', '0');
}
```

* **Strict-Transport-Security:** Garante que o navegador só se comunique com o servidor por HTTPS nos próximos 180 dias (15552000 segundos) e aplica essa política a todos os subdomínios.

* **X-Content-Type-Options:** Impede que os navegadores interpretem arquivos como um tipo MIME diferente, ajudando a prevenir ataques.

* **X-Frame-Options:** Protege o site contra ataques de clickjacking, permitindo que ele seja exibido em um frame apenas da mesma origem.

* **X-XSS-Protection:** Desativa o filtro de XSS para evitar possíveis problemas no nível do navegador.

### Solicitações POST, PUT, DELETE: 
Para esses tipos de solicitações, o módulo remove cabeçalhos específicos que não são necessários para operações de escrita, garantindo respostas menores e mais eficientes.

```javascript
if (['POST', 'PUT', 'DELETE'].includes(req.method)) {
    res.removeHeader('X-DNS-Prefetch-Control');
    res.removeHeader('X-Download-Options');
    res.removeHeader('X-Permitted-Cross-Domain-Policies');
    res.removeHeader('Strict-Transport-Security');
    res.removeHeader('Content-Security-Policy');
    res.removeHeader('Cross-Origin-Opener-Policy');
    res.removeHeader('Cross-Origin-Resource-Policy');
    res.removeHeader('Origin-Agent-Cluster');
    res.removeHeader('Referrer-Policy');
}
```

* **X-DNS-Prefetch-Control:** Reduz o pré-busca de DNS, desnecessário em solicitações POST.
* **X-Download-Options:** Normalmente usado para downloads de arquivos; desnecessário para operações de escrita.
* **X-Permitted-Cross-Domain-Policies:** Restringe quais políticas de domínio são enviadas, mas não é necessário para operações de escrita.
* **Strict-Transport-Security:** Removido em solicitações de escrita para permitir mais flexibilidade no transporte para operações sensíveis.
* **Content-Security-Policy:** Temporariamente desativado para certas solicitações de escrita.
* **Cross-Origin-Opener-Policy:** Removido para permitir mais flexibilidade ao interagir com conteúdo de origem cruzada.
* **Cross-Origin-Resource-Policy:** Garante políticas mais abertas para compartilhamento de recursos durante POST, PUT ou DELETE.
* **Origin-Agent-Cluster:** Remove restrições na capacidade do agente de compartilhar recursos entre origens.
* **Referrer-Policy:** Não é necessário para operações de escrita, removido para reduzir o tamanho do cabeçalho.

O módulo `@cmmv/http` gerencia inteligentemente os cabeçalhos para otimizar o desempenho, especialmente para operações de escrita (`POST`, `PUT`, `DELETE`). Para operações somente leitura (`GET`), garante forte segurança configurando cabeçalhos apropriados. Essa abordagem reduz a quantidade de dados desnecessários enviados com cada solicitação, ao mesmo tempo que adere às melhores práticas de segurança na web.

# Módulos Adicionais

O módulo `@cmmv/http` integra vários componentes de middleware essenciais para melhorar a segurança, o gerenciamento de sessões e o desempenho geral. Esses incluem `cors`, `helmet`, `session` e `compression`, que modificam os cabeçalhos e lidam com certos aspectos da comunicação HTTP.

## CORS

Controla o acesso a recursos de diferentes origens configurando os cabeçalhos apropriados de CORS (Cross-Origin Resource Sharing).

**Efeito nos Cabeçalhos:**

* **Access-Control-Allow-Origin:** Especifica quais origens podem acessar o servidor.
* **Access-Control-Allow-Methods:** Lista os métodos HTTP permitidos (por exemplo, GET, POST).
* **Access-Control-Allow-Headers:** Indica quais cabeçalhos podem ser usados durante a solicitação real.
* **Access-Control-Allow-Credentials:** Indica se a resposta à solicitação pode ser exposta ao cliente.

## Helmet

Helmet ajuda a proteger a aplicação configurando vários cabeçalhos HTTP que protegem contra ataques comuns.

**Efeito nos Cabeçalhos:**

* **Content-Security-Policy:** Restringe as fontes de onde o navegador pode carregar recursos.
* **X-DNS-Prefetch-Control:** Desativa o pré-busca de DNS para reduzir vazamentos de privacidade.
* **X-Frame-Options:** Protege contra clickjacking controlando se a página pode ser embutida em um frame.
* **X-Permitted-Cross-Domain-Policies:** Controla políticas de domínio cruzado para Adobe Flash e PDF.
* **Strict-Transport-Security:** Força conexões seguras (HTTPS).
* **X-Download-Options:** Evita que navegadores baixem arquivos do seu site sem autorização.
* **Referrer-Policy:** Controla quanta informação de referência é incluída nas solicitações.

## Session

Gerencia sessões de usuário via cookies, armazenando o estado de autenticação do usuário e outros dados de sessão.

* **Set-Cookie:** Define o cookie de sessão com atributos como HttpOnly, Secure, SameSite, etc.

## Compression

Comprime as respostas HTTP usando Gzip ou Brotli para reduzir o tamanho do corpo da resposta, acelerando a transmissão.

**Efeito nos Cabeçalhos:**

* **Content-Encoding:** Especifica o tipo de compressão usada (por exemplo, gzip, br).
* **Vary:** Instrui o navegador a variar a resposta com base no cabeçalho Accept-Encoding, permitindo que o servidor retorne conteúdo diferente com base nas capacidades de compressão do cliente.

O arquivo `.cmmv.config.cjs` na raiz do projeto permite que os desenvolvedores definam e personalizem políticas de segurança, gerenciamento de sessões, configurações de CORS e opções de compressão. Essas configurações modificam automaticamente os cabeçalhos de resposta para atender aos requisitos de segurança e desempenho.

### Personalização de Cabeçalhos

Os cabeçalhos também podem ser configurados ou modificados diretamente dentro do setup do template usando a propriedade `head`. Isso oferece flexibilidade para adicionar metatags, links canônicos e outras configurações relacionadas ao SEO dinamicamente por página.

```html
<script s-setup>
export default {
    layout: "default",

    head: {
        meta: [
            { name: "description", content: "Exemplo de Todolist no CMMV" },
            { name: "keywords", content: "cmmv, contract model, websocket" }
        ],
        link: [
            { rel: "stylesheet", href: "/assets/styles/todo.css" },
            { rel: "canonical", href: "https://cmmv.io" },
        ]
    },
}
</script>
```

O módulo `@cmmv/http` automatiza otimizações essenciais para os cabeçalhos HTTP, garantindo segurança e desempenho por meio de compressão, gerenciamento de sessões e proteção contra ataques comuns na web via middleware como `cors`, `helmet` e `compression`. Isso é complementado por configurações personalizadas definidas no arquivo `.cmmv.config.cjs` e controles de cabeçalho específicos por template, tornando a plataforma CMMV uma base segura e eficiente para aplicativos web.
