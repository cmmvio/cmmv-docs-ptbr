# Middleware

O sistema de middleware em `@cmmv/server` oferece flexibilidade e integração poderosa para processar requisições e respostas HTTP. Similar ao Express, ele fornece hooks e pontos de integração que permitem aos desenvolvedores modificar o comportamento das requisições enquanto elas fluem pelo sistema. No entanto, o middleware no `@cmmv/server` também introduz novos comportamentos e otimizações, particularmente por meio do uso de hooks.

Este sistema permite a implementação de funções de middleware essenciais, como ETag, Body-Parser, Compression, Cookie-Parser, CORS, Helmet e Server-Static, cada uma seguindo um padrão comum. Alguns desses middlewares, como o ETag, são totalmente compatíveis com o Express. Outros, como o Server-Static, possuem implementações personalizadas que introduzem mudanças específicas de comportamento, tornando-os incompatíveis com o Express.

O design central do middleware em `@cmmv/server` gira em torno de hooks, que são acionados em vários pontos do ciclo de vida da requisição-resposta. Esse design permite um processamento mais preciso e eficiente das requisições.

## Exemplo

O middleware ETag é usado para gerenciar cabeçalhos ETag para fins de cache. ETags são um mecanismo para validação de cache e são gerados com base no conteúdo do corpo da resposta. Se o conteúdo não mudou entre as requisições, o servidor pode responder com um status 304 Not Modified, melhorando o desempenho.

Aqui está um exemplo do middleware ETag implementado no `@cmmv/server`:

```typescript
export class EtagMiddleware {
    public middlewareName: string = 'etag';

    protected options: ETagOptions;

    constructor(options?: ETagOptions) {
        this.options = {
            algorithm: options?.algorithm || 'sha1',
            weak: Boolean(options?.weak === true),
        };
    }

    async process(req, res, next) {
        if (req.app && typeof req.app.addHook == 'function')
            req.app.addHook('onSend', this.onCall.bind(this));
        else this.onCall.call(this, req, res, res.body, next);
    }

    onCall(req, res, payload, done) {
        const hash = this.buildHashFn(this.options.algorithm, this.options.weak);
        let etag = res.getHeader('etag');
        let newPayload;

        if (!etag) {
            if (!(typeof payload === 'string' || payload instanceof Buffer)) {
                done(null, newPayload);
                return;
            }

            etag = hash(payload);
            res.set('etag', etag);
        }

        if (
            req.headers['if-none-match'] === etag ||
            req.headers['if-none-match'] === 'W/' + etag ||
            'W/' + req.headers['if-none-match'] === etag
        ) {
            res.code(304);
            newPayload = '';
        }

        done(null, newPayload);
    }

    buildHashFn(algorithm = 'sha1', weak = false) {
        this.validateAlgorithm(algorithm);

        const prefix = weak ? 'W/"' : '"';

        if (algorithm === 'fnv1a')
            return payload => prefix + fnv1a(payload).toString(36) + '"';

        return payload =>
            prefix +
            createHash(algorithm).update(payload).digest('base64') +
            '"';
    }

    validateAlgorithm(algorithm) {
        if (algorithm === 'fnv1a') return true;

        try {
            createHash(algorithm);
        } catch (e) {
            throw new TypeError(`Algorithm \${algorithm} not supported.`);
        }
    }
}

export default async function (options?: ETagOptions) {
    const middleware = new EtagMiddleware(options);
    return (req, res, next) => middleware.process(req, res, next);
}

export const etag = function (options?: ETagOptions) {
    const middleware = new EtagMiddleware(options);
    return (req, res, next) => middleware.process(req, res, next);
};
```

O sistema de middleware em `@cmmv/server` foi projetado para oferecer compatibilidade com o Express e melhorias que fornecem melhor controle sobre o ciclo de vida de requisição-resposta. Ao aproveitar hooks e execução seletiva, o sistema garante que o middleware seja executado apenas quando necessário, levando a um melhor desempenho.

Embora a maioria dos middlewares seja compatível com o Express, alguns, como o Server-Static, introduzem mudanças de comportamento específicas. Este sistema é projetado para ser flexível e modular, permitindo que você implemente middlewares personalizados facilmente, integre middlewares existentes e otimize o desempenho da sua aplicação.

Adicionalmente, todos os middlewares implementados para `@cmmv/server` devem retornar uma promise, garantindo que eles se encaixem no ciclo de vida assíncrono do framework. Sempre que possível, versões compatíveis dos middlewares são fornecidas para uso em aplicações Express.

## Compression

O middleware `@cmmv/compression` fornece compressão de resposta HTTP para seu servidor, suportando codificações gzip, deflate e brotli. A compressão reduz o tamanho do corpo da resposta, melhorando a velocidade e eficiência na entrega de conteúdo ao cliente. Ele pode ser usado tanto no `@cmmv/server` quanto em ambientes Express, com algumas variações na implementação.

Este middleware se integra perfeitamente ao ciclo de vida da requisição, aplicando compressão às respostas com base no tipo de conteúdo, tamanho e no cabeçalho `Accept-Encoding` do cliente. Ele lida automaticamente com grandes payloads e ajusta dinamicamente a codificação para diferentes tipos de conteúdo.

**Instalação**

Para instalar o middleware `@cmmv/compression`, execute o seguinte comando:

```bash
$ pnpm add @cmmv/compression
```

**Uso**

Ao usar o `@cmmv/server`, o middleware de compressão deve ser adicionado como parte da cadeia de middlewares. Aqui está um exemplo de como usá-lo:

```typescript
import compression from '@cmmv/compression';

app.use(compression({ 
    threshold: 1024, 
    cacheEnabled: true 
}));
```

O middleware `@cmmv/compression` vem com uma série de opções para controlar como e quando a compressão é aplicada. Abaixo está uma lista de opções disponíveis:

| Opção         | Tipo                | Padrão                                     | Descrição                                                                                                                                                         |
|---------------|---------------------|-------------------------------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------|
| `threshold`  | `number` ou `string` | `1024`                                    | Especifica o tamanho mínimo da resposta (em bytes) necessário para que a compressão seja aplicada. Respostas menores não são comprimidas.                       |
| `cacheEnabled` | `boolean`         | `false`                                   | Habilita o cache de respostas comprimidas para melhorar o desempenho em requisições repetidas. Respostas serão armazenadas na memória pelo tempo especificado.   |
| `algorithm`   | `string`          | `'sha1'`                                  | Especifica o algoritmo de hashing usado para gerar um ETag para validação de cache. Valores suportados incluem `sha1`, `md5` e `fnv1a`.                      |
| `weak`        | `boolean` | `false`                                     | Indica se o ETag deve ser marcado como "fraco", o que significa que fornece validação de cache menos rigorosa.                                          |
| `level`       | `number`  | `zlib.constants.Z_DEFAULT_COMPRESSION`       | Especifica o nível de compressão para algoritmos baseados no zlib. Aceita um valor entre `0` (sem compressão) e `9` (compressão máxima).               |
| `memLevel`    | `number`  | `8`                                         | Controla o uso de memória para compressão (valores mais altos usam mais memória, mas fornecem melhor compressão). Aceita valores entre `1` e `9`.      |
| `strategy`    | `number`  | `zlib.constants.Z_DEFAULT_STRATEGY`          | Especifica a estratégia de compressão a ser usada pelos algoritmos zlib. Estratégias comuns incluem `Z_FILTERED`, `Z_HUFFMAN_ONLY` e `Z_RLE`.          |
| `filter`      | `function`| `shouldCompress`                            | Uma função para determinar se uma resposta deve ou não ser comprimida com base no tipo de conteúdo ou outros fatores.                                      |
| `flush`       | `boolean` | `false`                                     | Habilita a limpeza manual dos buffers de compressão. Quando ativado, o servidor pode forçar uma limpeza quando necessário (por exemplo, ao transmitir respostas). |
| `chunkSize`   | `number`  | `16384`                                     | Define o tamanho do bloco usado para fluxos de compressão. Tamanhos menores resultam em gravações mais frequentes, mas podem impactar o desempenho.         |
| `windowBits`  | `number`  | `15`                                        | Especifica o tamanho da janela de compressão (em bits). Uma janela maior oferece melhor compressão, mas usa mais memória.                                 |

## Cookie-Parser

O middleware `@cmmv/cookie-parser` foi projetado para analisar cookies em requisições HTTP, tanto assinados quanto não assinados, e torná-los disponíveis no objeto `req.cookies`. Ele suporta tanto `CMMV` quanto Express, proporcionando integração perfeita com qualquer um dos frameworks.

**Instalação**

```bash
$ pnpm add @cmmv/cookie-parser
```

**Uso**

No `CMMV`, o middleware é usado registrando-o no aplicativo por meio de hooks. Ele analisará automaticamente os cookies nas requisições recebidas e os disponibilizará nos objetos `req.cookies` e `req.signedCookies`.

```typescript
import cmmv from '@cmmv/server';
import cookieParser from '@cmmv/cookie-parser';

const app = cmmv();

app.use(cookieParser({ secret: 'mySecretKey' }));

app.get('/cookies', (req, res) => {
    res.send({
        cookies: req.cookies,
        signedCookies: req.signedCookies,
    });
});

app.listen({ port: 3000 });
```

| Opção   | Tipo                | Padrão | Descrição                                                                                              |
|---------|---------------------|--------|------------------------------------------------------------------------------------------------------|
| `name`  | `string`          |        | O nome do cookie a ser analisado.                                                                    |
| `secret`| `string`/`string[]` | `[]`  | Uma string ou array de strings usadas para assinar e verificar cookies. Isso garante a integridade dos cookies assinados. |
| `decode`| `function`       |        | Uma função personalizada para decodificar cookies. Se não fornecida, será usado o padrão `decodeURIComponent`. |
| `path`  | `string`          | `'/'`| Define o caminho da URL que deve existir para que o cookie seja incluído nas requisições.             |

**Exemplo para Cookies Assinados**

```typescript
app.use(cookieParser({ secret: 'mySecretKey' }));

app.get('/set-cookie', (req, res) => {
    res.cookie('name', 'value', { signed: true });
    res.send('Cookie set');
});

app.get('/get-cookie', (req, res) => {
    res.json({ signedCookies: req.signedCookies });
});
```

## Cors

O middleware CORS (Cross-Origin Resource Sharing) permite que você habilite o compartilhamento de recursos entre origens diferentes para suas aplicações, configurando os cabeçalhos HTTP apropriados. Ele é baseado no middleware CORS do Express, mas foi adaptado para suportar hooks assíncronos no ambiente `CMMV`, tornando-o totalmente compatível com `@cmmv/server` enquanto mantém compatibilidade com o Express sempre que possível.

**Instalação**

```bash
$ pnpm add @cmmv/cors
```

**Uso**

```typescript
import cmmv from '@cmmv/server';
import cors from '@cmmv/cors';

const app = cmmv();
app.use(cors());
app.listen({ port: 3000 });
```

| Opção               | Tipo                  | Padrão                    | Descrição                                                                                  |
|----------------------|-----------------------|---------------------------|------------------------------------------------------------------------------------------|
| `origin`            | `string` ou `function` | `*`                     | Especifica a origem que pode acessar o recurso. Pode ser uma string, array ou função.    |
| `methods`           | `string` ou `string[]`| `'GET,HEAD,PUT,PATCH,POST,DELETE'` | Especifica os métodos HTTP permitidos para requisições de origem cruzada.               |
| `credentials`       | `boolean`           | `false`                 | Se `true`, o cabeçalho `Access-Control-Allow-Credentials` será definido como `true`. |
| `allowedHeaders`    | `string` ou `string[]`| `undefined`             | Especifica os cabeçalhos que podem ser usados na requisição real.                        |

## Etag

## Helmet

O middleware `@cmmv/helmet` é projetado para melhorar a segurança de aplicativos web, configurando vários cabeçalhos HTTP. Ele oferece suporte a políticas de segurança de conteúdo (Content Security Policies), cabeçalhos de segurança como `X-Frame-Options`, `Strict-Transport-Security` e outros. Este middleware é desenvolvido com flexibilidade, permitindo que você personalize seu comportamento com base nos requisitos do seu aplicativo. Ele segue um padrão de implementação semelhante ao popular `helmet` e inclui integração nativa com o CMMV, mantendo a compatibilidade com o Express sempre que possível.

Este middleware configura automaticamente cabeçalhos de segurança HTTP para proteger melhor contra vulnerabilidades comuns, como ataques de cross-site scripting (XSS), clickjacking e interceptação de dados.

**Instalação**

```bash
$ pnpm add @cmmv/helmet
```

**Uso**

```typescript
import cmmv from '@cmmv/server';
import helmet from '@cmmv/helmet';

const app = cmmv();
app.use(helmet({
    contentSecurityPolicy: {
        directives: {
        defaultSrc: ["'self'"],
        scriptSrc: ["'self'", "https://trustedscripts.com"],
        },
    },
    hsts: {
        maxAge: 31536000, // Força HTTPS por um ano
    },
    frameguard: {
        action: 'deny', // Desabilita iframes completamente
    }
}));
app.listen({ port: 3000 });
```

O `HelmetMiddleware` permite configurar vários cabeçalhos HTTP e políticas de segurança. Abaixo está a lista de opções disponíveis:

| Opção                          | Tipo                | Padrão                       | Descrição                                                                                                   |
|--------------------------------|---------------------|------------------------------|-------------------------------------------------------------------------------------------------------------|
| `contentSecurityPolicy`      | objeto ou booleano  | Habilitado com políticas padrão | Configura o cabeçalho Content Security Policy (CSP). Pode ser desativado definindo como `false` ou personalizado fornecendo um objeto. |
| `frameguard`                 | objeto ou booleano  | `SAMEORIGIN`               | Configura o cabeçalho `X-Frame-Options` para evitar clickjacking.                                         |
| `dnsPrefetchControl`         | booleano            | `true`                     | Controla o cabeçalho `X-DNS-Prefetch-Control` para melhorar a privacidade.                                |
| `expectCt`                   | objeto ou booleano  | `false`                    | Adiciona o cabeçalho `Expect-CT` para reforçar os requisitos de transparência de certificado.             |
| `hsts`                       | objeto ou booleano  | `true`                     | Configura o cabeçalho `Strict-Transport-Security` para forçar conexões HTTPS por um tempo especificado.    |
| `ieNoOpen`                   | booleano            | `true`                     | Configura o cabeçalho `X-Download-Options` para evitar que downloads sejam abertos automaticamente no Internet Explorer. |
| `noSniff`                    | booleano            | `true`                     | Configura o cabeçalho `X-Content-Type-Options` para evitar que navegadores façam MIME-sniffing da resposta. |
| `xssFilter`                  | booleano            | `true`                     | Configura o cabeçalho `X-XSS-Protection` para habilitar o filtro XSS embutido na maioria dos navegadores modernos. |
| `referrerPolicy`             | string ou objeto    | `no-referrer`              | Configura o cabeçalho `Referrer-Policy` para controlar as informações enviadas no cabeçalho `Referer`.  |
| `hidePoweredBy`              | booleano ou objeto  | `true`                     | Oculta o cabeçalho `X-Powered-By` para evitar vazamento de informações sobre a tecnologia do servidor.    |
| `permittedCrossDomainPolicies`| objeto ou booleano | `false`                    | Configura o cabeçalho `X-Permitted-Cross-Domain-Policies`, geralmente usado por produtos Flash/Adobe.     |


O middleware CMMV ETag adiciona automaticamente um cabeçalho `ETag` às respostas HTTP, ajudando na validação do cache ao gerar um hash único para cada payload de resposta. Este middleware é baseado na implementação ETag do Fastify, mas adaptado para funcionar dentro do framework CMMV. O cabeçalho ETag permite que navegadores e outros clientes determinem se o conteúdo mudou desde a última solicitação, reduzindo o uso de largura de banda e melhorando o desempenho.

Este middleware foi projetado para suportar ambientes CMMV e Express. Para CMMV, a exportação padrão retorna uma promise, enquanto para Express, uma função `etag` separada está disponível.

**Instalação**

```bash
$ pnpm add @cmmv/etag
```

**Uso**

```typescript
import cmmv from '@cmmv/server';
import etag from '@cmmv/etag';

const app = cmmv();
app.use(etag());
app.listen({ port: 3000 });
```

A interface `ETagOptions` fornece várias opções de configuração para personalizar o comportamento do middleware ETag.

| Opção     | Tipo     | Padrão | Descrição                                                                 |
|-----------|----------|--------|---------------------------------------------------------------------------|
| algorithm | string   | 'sha1' | Especifica o algoritmo de hashing a ser usado para gerar o ETag. Valores suportados incluem 'sha1', 'md5' e 'fnv1a'. |
| weak      | boolean  | false  | Se `true`, o middleware gera ETags fracos (prefixados com W/). ETags fracos permitem uma validação de cache menos rigorosa. |

## Server-Static

O middleware `@cmmv/server-static` fornece funcionalidade para servir arquivos estáticos de um diretório. Ele é inspirado no `serve-static` do Express e no `fastify-static` do Fastify. Este middleware pode ser configurado para servir arquivos de um ou mais diretórios e fornece cache, compressão de arquivos e outras opções úteis para servir conteúdo estático.

Você pode usar a função `serveStatic` para servir arquivos estáticos de um diretório:

```typescript
import cmmv, { serverStatic } from '@cmmv/server';

const app = cmmv();
const host = '0.0.0.0';
const port = 3000;

app.use(serverStatic('public'));

app.set('view engine', 'pug');

app.get('/view', function (req, res) {
    res.render('index', { title: 'Hey', message: 'Hello there!' });
});

app.listen({ host, port })
.then(server => {
    console.log(
        `Listen on http://\${server.address().address}:\${server.address().port}`,
    );
})
.catch(err => {
    throw Error(err.message);
});
```

## All Middlwares

Este exemplo configura um servidor usando `@cmmv/server` com vários middlewares para lidar de forma eficiente com solicitações HTTP. Ele serve arquivos estáticos da pasta `public`, habilita CORS para compartilhamento de recursos entre origens diferentes e adiciona cabeçalhos ETag para cache usando o algoritmo `fnv1a`. O middleware de análise de cookies é usado para interpretar cookies e preencher `req.cookies`, enquanto os analisadores JSON e URL codificados processam os dados do corpo das requisições. As respostas são compactadas usando GZIP por meio do middleware de compressão, e o Helmet adiciona cabeçalhos de segurança, incluindo uma Política de Segurança de Conteúdo personalizada. O servidor define rotas para lidar com requisições GET e POST básicas, renderizar visualizações, enviar respostas JSON e gerenciar rotas dinâmicas. Ele escuta na porta `0.0.0.0:3000`, com uma opção para habilitar HTTP/2 e HTTPS se os certificados forem fornecidos.

```typescript
//import { readFileSync } from "node:fs";

import cmmv, { json, urlencoded, serverStatic } from '@cmmv/server';
import etag from '@cmmv/etag';
import cors from '@cmmv/cors';
import cookieParser from '@cmmv/cookie-parser';
import compression from '@cmmv/compression';
import helmet from '@cmmv/helmet';

const app = cmmv({
    /*http2: true,
    https: {
        key: readFileSync("./cert/private-key.pem"),
        cert: readFileSync("./cert/certificate.pem"),
        passphrase: "1234"
    }*/
});

const host = '0.0.0.0';
const port = 3000;

app.use(serverStatic('public'));
app.use(cors());
app.use(etag({ algorithm: 'fnv1a' }));
app.use(cookieParser());
app.use(json({ limit: '50mb' }));
app.use(urlencoded({ limit: '50mb', extended: true }));
app.use(compression({ level: 6 }));
app.use(
    helmet({
        contentSecurityPolicy: {
            useDefaults: false,
            directives: {
                defaultSrc: ["'self'"],
                scriptSrc: ["'self'", 'example.com'],
                objectSrc: ["'none'"],
                upgradeInsecureRequests: [],
            },
        },
    }),
);

app.set('view engine', 'pug');

app.get('/view', function (req, res) {
    res.render('index', { title: 'Hey', message: 'Hello there!' });
});

app.get('/', async (req, res) => {
    res.send('Hello World');
});

app.get('/json', async (req, res) => {
    res.json({ hello: 'world' });
});

app.get('/user/:id', async (req, res) => {
    res.send('User ' + req.params.id);
});

app.get('/users', async (req, res) => {
    res.json(req.query);
});

app.post('/test', async (req, res) => {
    console.log(req.body);
    res.send('ok');
});

app.listen({ host, port })
.then(server => {
    const addr = server.address();
    console.log(
        `Listen on http://${addr.address}:${addr.port}`,
    );
})
.catch(err => {
    throw Error(err.message);
});
```