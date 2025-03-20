# Middleware

O sistema de middleware no ``@cmmv/server`` oferece flexibilidade e uma integração poderosa para processar requisições e respostas HTTP. Semelhante ao Express, ele fornece *hooks* e pontos de integração que permitem aos desenvolvedores modificar o comportamento das requisições enquanto elas fluem pelo sistema. No entanto, o middleware no ``@cmmv/server`` também introduz novos comportamentos e otimizações, particularmente por meio do uso de *hooks*.

Esse sistema permite a implementação de funções de middleware essenciais como ETag, Body-Parser, Compression, Cookie-Parser, CORS, Helmet e Server-Static, cada uma seguindo um padrão comum. Alguns desses middlewares, como o ETag, são totalmente compatíveis com o Express. Outros, como o Server-Static, possuem implementações personalizadas que introduzem mudanças específicas de comportamento, tornando-os incompatíveis com o Express.

O design central do middleware no ``@cmmv/server`` gira em torno de *hooks*, que são acionados em vários pontos do ciclo de vida da requisição-resposta. Esse design permite um processamento mais preciso e eficiente das requisições.

## Exemplo

O middleware ETag é usado para gerenciar cabeçalhos ETag para fins de cache. Os ETags são um mecanismo para validação de cache e são gerados com base no conteúdo do corpo da resposta. Se o conteúdo não mudou entre as requisições, o servidor pode responder com um status 304 Not Modified, melhorando o desempenho.

Aqui está um exemplo do middleware ETag implementado no ``@cmmv/server``:

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
            throw new TypeError(`Algoritmo ${algorithm} não suportado.`);
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

O sistema de middleware no ``@cmmv/server`` é projetado para oferecer tanto compatibilidade com o Express quanto melhorias que proporcionam maior controle sobre o ciclo de requisição-resposta. Ao aproveitar *hooks* e execução seletiva, o sistema garante que o middleware só seja executado quando necessário, resultando em melhor desempenho.

Embora a maioria dos middlewares seja compatível com o Express, alguns — como o Server-Static — introduzem mudanças específicas de comportamento. Esse sistema é projetado para ser flexível e modular, permitindo que você implemente middlewares personalizados com facilidade, integre middlewares existentes e otimize o desempenho da sua aplicação.

Além disso, todos os middlewares implementados para o ``@cmmv/server`` devem retornar uma promessa, garantindo que se encaixem no ciclo de vida assíncrono do framework. Quando possível, versões compatíveis dos middlewares são fornecidas para uso em aplicações Express.

## Compression

O middleware ``@cmmv/compression`` fornece compressão de respostas HTTP para o seu servidor, suportando codificações gzip, deflate e brotli. A compressão reduz o tamanho do corpo da resposta, o que melhora a velocidade e a eficiência da entrega de conteúdo ao cliente. Ele pode ser usado tanto em ambientes ``@cmmv/server`` quanto Express, com algumas variações na implementação.

Esse middleware se integra perfeitamente ao ciclo de vida da requisição, aplicando compressão às respostas com base no tipo de conteúdo, tamanho e no cabeçalho ``Accept-Encoding`` do cliente. O middleware lida automaticamente com payloads grandes e ajusta dinamicamente a codificação para diferentes tipos de conteúdo.

**Instalação**

Para instalar o middleware ``@cmmv/compression``, execute o seguinte comando:

```bash
$ pnpm add @cmmv/compression
```

**Uso**

Ao usar o ``@cmmv/server``, o middleware de compressão deve ser adicionado como parte da cadeia de middlewares. Aqui está um exemplo de como usá-lo:

```typescript
import compression from '@cmmv/compression';

app.use(compression({
    threshold: 1024,
    cacheEnabled: true
}));
```

O middleware ``@cmmv/compression`` vem com uma variedade de opções para controlar como e quando a compressão é aplicada. Abaixo está uma lista das opções disponíveis:

| Opção         | Tipo     | Padrão                                     | Descrição                                                                                                                                    |
|---------------|----------|--------------------------------------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| `threshold`   | `number` ou `string` | `1024`                            | Especifica o tamanho mínimo da resposta (em bytes) necessário para que a compressão seja aplicada. Respostas menores não são comprimidas. Aceita valores legíveis como '1kb'. |
| `cacheEnabled`| `boolean` | `false`                                     | Habilita o cache de respostas comprimidas para melhorar o desempenho em requisições repetidas. As respostas serão armazenadas em memória pelo tempo especificado. |
| `cacheTimeout`| `number`  | `60000 (1 min)`                             | Define o tempo limite (em milissegundos) após o qual as respostas comprimidas em cache são removidas da memória. Aplicável apenas se `cacheEnabled` for `true`. |
| `algorithm`   | `string`  | `'sha1'`                                    | Especifica o algoritmo de hash usado para gerar um ETag para validação de cache. Valores suportados incluem `sha1`, `md5` e `fnv1a`.      |
| `weak`        | `boolean` | `false`                                     | Indica se o ETag deve ser marcado como "fraco", o que permite uma validação de cache menos rigorosa.                                       |
| `level`       | `number`  | `zlib.constants.Z_DEFAULT_COMPRESSION`      | Especifica o nível de compressão para algoritmos baseados em zlib. Aceita valores entre `0` (sem compressão) e `9` (compressão máxima).   |
| `memLevel`    | `number`  | `8`                                         | Controla o uso de memória para compressão (valores mais altos usam mais memória, mas oferecem melhor compressão). Aceita valores entre `1` e `9`. |
| `strategy`    | `number`  | `zlib.constants.Z_DEFAULT_STRATEGY`         | Especifica a estratégia de compressão a ser usada pelos algoritmos zlib. Inclui `Z_FILTERED`, `Z_HUFFMAN_ONLY` e `Z_RLE`.                 |
| `filter`      | `function`| `shouldCompress`                            | Uma função para determinar se uma resposta deve ser comprimida com base no tipo de conteúdo ou outros fatores.                            |
| `flush`       | `boolean` | `false`                                     | Habilita a liberação manual dos buffers de compressão. Quando ativado, o servidor pode forçar uma liberação quando necessário (exemplo: streaming). |
| `chunkSize`   | `number`  | `16384`                                     | Define o tamanho do pedaço usado para fluxos de compressão. Tamanhos menores resultam em escritas mais frequentes, mas podem impactar o desempenho. |
| `windowBits`  | `number`  | `15`                                        | Especifica o tamanho da janela de compressão (em bits). Um tamanho maior oferece melhor compressão, mas usa mais memória.                  |

## Cookie-Parser

O middleware ``@cmmv/cookie-parser`` é projetado para analisar cookies em requisições HTTP, tanto assinados quanto não assinados, e disponibilizá-los no objeto ``req.cookies``. Ele suporta tanto o ``CMMV`` quanto o Express, proporcionando uma integração perfeita com qualquer um dos frameworks.

**Instalação**

```bash
$ pnpm add @cmmv/cookie-parser
```

**Uso**

No CMMV, o middleware é usado registrando-o com o aplicativo usando *hooks*. Ele analisará automaticamente os cookies nas requisições recebidas e os disponibilizará nos objetos ``req.cookies`` e ``req.signedCookies``.

```typescript
import cmmv from '@cmmv/server';
import cookieParser from '@cmmv/cookie-parser';

const app = cmmv();

app.use(cookieParser({ secret: 'minhaChaveSecreta' }));

app.get('/cookies', (req, res) => {
    res.send({
        cookies: req.cookies,
        signedCookies: req.signedCookies,
    });
});

app.listen({ port: 3000 });
```

| Opção   | Tipo           | Padrão | Descrição                                                                                             |
|---------|----------------|--------|-------------------------------------------------------------------------------------------------------|
| `name`  | `string`       |        | O nome do cookie a ser analisado.                                                                     |
| `secret`| `string`/`string[]` | `[]`   | Uma string ou array de strings usado para assinar e verificar cookies. Garante a integridade de cookies assinados. |
| `decode`| `function`     |        | Uma função de decodificação personalizada para analisar cookies. Se não fornecida, usa-se o padrão `decodeURIComponent`. |
| `path`  | `string`       | `'/'`  | Define o caminho da URL que deve existir para que o cookie seja incluído nas requisições.              |

**Exemplo para Cookies Assinados**

```typescript
app.use(cookieParser({ secret: 'minhaChaveSecreta' }));

app.get('/set-cookie', (req, res) => {
    res.cookie('nome', 'valor', { signed: true });
    res.send('Cookie definido');
});

app.get('/get-cookie', (req, res) => {
    res.json({ signedCookies: req.signedCookies });
});
```

## CORS

O middleware CORS (Cross-Origin Resource Sharing) permite que você habilite o compartilhamento de recursos entre origens cruzadas para suas aplicações ao definir os cabeçalhos HTTP apropriados. Ele é baseado no middleware CORS do Express, mas foi adaptado para suportar *hooks* assíncronos no ambiente do servidor CMMV, tornando-o totalmente compatível com ``@cmmv/server`` enquanto mantém compatibilidade com o Express sempre que possível.

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

| Opção                | Tipo                | Padrão                                           | Descrição                                                                                   |
|----------------------|---------------------|--------------------------------------------------|---------------------------------------------------------------------------------------------|
| `origin`             | `string` ou `function`| `*`                                              | Especifica a origem que tem permissão para acessar o recurso. Pode ser uma string, array ou função. |
| `methods`            | `string` ou `string[]`| `'GET,HEAD,PUT,PATCH,POST,DELETE'`                | Especifica os métodos HTTP permitidos para requisições de origem cruzada.                   |
| `preflightContinue`   | `boolean`            | `false`                                          | Se `true`, o middleware não interromperá as requisições de pré-verificação e as passará ao próximo manipulador. |
| `optionsSuccessStatus`| `number`             | `204`                                            | O código de status enviado para requisições OPTIONS bem-sucedidas (pré-verificação). Alguns navegadores legados usam 204 para sucesso. |
| `credentials`         | `boolean`            | `false`                                          | Se `true`, o cabeçalho `Access-Control-Allow-Credentials` será definido como `true`.         |
| `maxAge`              | `number`             | `0`                                              | Especifica o tempo em segundos que os navegadores podem armazenar respostas de pré-verificação em cache. |
| `headers`             | `string` ou `string[]`| `undefined`                                      | Especifica quais cabeçalhos podem ser enviados em uma requisição.                           |
| `allowedHeaders`      | `string` ou `string[]`| `undefined`                                      | Especifica os cabeçalhos que podem ser usados na requisição real. Padrão é o `Access-Control-Request-Headers` da requisição. |
| `exposedHeaders`      | `string` ou `string[]`| `undefined`                                      | Especifica quais cabeçalhos são expostos ao navegador. Padrão é nenhum.                     |

## ETag

O middleware ETag do CMMV adiciona automaticamente um cabeçalho ``ETag`` às respostas HTTP, auxiliando na validação de cache ao gerar um hash único para cada payload de resposta. Esse middleware é baseado na implementação do ETag do Fastify, mas adaptado para funcionar no framework CMMV. O cabeçalho ETag permite que navegadores e outros clientes determinem se o conteúdo mudou desde a última requisição, reduzindo o uso de banda e melhorando o desempenho.

Esse middleware é projetado para suportar tanto ambientes CMMV quanto Express. Para CMMV, a exportação padrão retorna uma promessa, e para Express, uma função separada ``etag`` está disponível.

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

A interface ``ETagOptions`` fornece várias opções de configuração para personalizar o comportamento do middleware ETag.

| Opção     | Tipo    | Padrão | Descrição                                                                 |
|-----------|---------|--------|---------------------------------------------------------------------------|
| `algorithm` | `string`  | 'sha1' | Especifica o algoritmo de hash usado para gerar o ETag. Valores suportados incluem 'sha1', 'md5' e 'fnv1a'. |
| `weak`    | `boolean` | false  | Se verdadeiro, o middleware gera ETags fracos (prefixados com W/). ETags fracos permitem validação de cache menos rigorosa. |

## Helmet

O middleware ``@cmmv/helmet`` é projetado para aumentar a segurança de aplicações web ao definir vários cabeçalhos HTTP. Ele oferece suporte a políticas de segurança de conteúdo, cabeçalhos de segurança como ``X-Frame-Options``, ``Strict-Transport-Security`` e mais. Esse middleware foi construído com flexibilidade em mente, permitindo que você personalize seu comportamento com base nas necessidades da sua aplicação. Ele segue um padrão de implementação semelhante à popular biblioteca ``helmet`` e inclui integração nativa com o CMMV, mantendo compatibilidade com o Express quando possível.

Esse middleware configura automaticamente cabeçalhos de segurança HTTP para melhor proteção contra vulnerabilidades comuns, como ataques de cross-site scripting (XSS), clickjacking e sniffing de dados.

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
        action: 'deny', // Impede iframes completamente
    }
}));
app.listen({ port: 3000 });
```

O ``HelmetMiddleware`` permite configurar vários cabeçalhos HTTP e políticas de segurança. Abaixo está a lista de opções disponíveis:

| Opção                          | Tipo                | Padrão                      | Descrição                                                                                                  |
|--------------------------------|---------------------|-----------------------------|------------------------------------------------------------------------------------------------------------|
| `contentSecurityPolicy`        | object ou boolean   | Habilitado com políticas padrão | Configura o cabeçalho Content Security Policy (CSP). Pode ser desabilitado com `false` ou personalizado com um objeto. |
| `frameguard`                   | object ou boolean   | `SAMEORIGIN`                | Define o cabeçalho `X-Frame-Options` para prevenir clickjacking.                                           |
| `dnsPrefetchControl`           | boolean             | `true`                      | Controla o cabeçalho `X-DNS-Prefetch-Control` para melhorar a privacidade.                                |
| `expectCt`                     | object ou boolean   | `false`                     | Adiciona o cabeçalho `Expect-CT` para impor requisitos de Transparência de Certificados.                  |
| `hsts`                         | object ou boolean   | `true`                      | Define o cabeçalho `Strict-Transport-Security` para forçar conexões HTTPS por um tempo especificado.      |
| `ieNoOpen`                     | boolean             | `true`                      | Define o cabeçalho `X-Download-Options` para evitar que downloads abram automaticamente no Internet Explorer. |
| `noSniff`                      | boolean             | `true`                      | Define o cabeçalho `X-Content-Type-Options` para impedir que navegadores façam sniffing de MIME da resposta. |
| `xssFilter`                    | boolean             | `true`                      | Define o cabeçalho `X-XSS-Protection` para habilitar o filtro XSS embutido em navegadores modernos.        |
| `referrerPolicy`               | string ou object    | `no-referrer`               | Configura o cabeçalho `Referrer-Policy` para controlar as informações enviadas no cabeçalho `Referer`.     |
| `hidePoweredBy`                | boolean ou object   | `true`                      | Oculta o cabeçalho `X-Powered-By` para evitar vazamento de informações sobre a tecnologia do servidor.     |
| `permittedCrossDomainPolicies` | object ou boolean   | `false`                     | Configura o cabeçalho `X-Permitted-Cross-Domain-Policies`, frequentemente usado por produtos Flash/Adobe.  |

## Server-Static

O middleware ``@cmmv/server-static`` oferece funcionalidades para servir arquivos estáticos a partir de um diretório. Ele é inspirado no ``serve-static`` do Express e no ``fastify-static`` do Fastify. Esse middleware pode ser configurado para servir arquivos de um ou mais diretórios e fornece opções de cache, compressão de arquivos e outras funcionalidades úteis para entrega de conteúdo estático.

Você pode usar a função ``serverStatic`` para servir arquivos estáticos de um diretório:

```typescript
import cmmv, { serverStatic } from '@cmmv/server';

const app = cmmv();
const host = '0.0.0.0';
const port = 3000;

app.use(serverStatic('public'));

app.set('view engine', 'pug');

app.get('/view', function (req, res) {
    res.render('index', { title: 'Oi', message: 'Olá!' });
});

app.listen({ host, port })
.then(server => {
    console.log(
        `Ouvindo em http://${server.address().address}:${server.address().port}`,
    );
})
.catch(err => {
    throw Error(err.message);
});
```

## Todos os Middlewares

Este exemplo configura um servidor usando ``@cmmv/server`` com vários middlewares para lidar eficientemente com requisições HTTP. Ele serve arquivos estáticos da pasta ``public``, habilita CORS para compartilhamento de recursos entre origens, e adiciona cabeçalhos ETag para cache usando o algoritmo ``fnv1a``. O middleware cookie-parser é usado para analisar cookies e preencher ``req.cookies``, enquanto os parsers JSON e URL-encoded processam dados do corpo da requisição. As respostas são comprimidas usando GZIP através do middleware de compressão, e o Helmet adiciona cabeçalhos de segurança, incluindo uma Política de Segurança de Conteúdo personalizada. O servidor define rotas para lidar com requisições GET e POST básicas, renderizar visões, enviar respostas JSON e gerenciar rotas dinâmicas. Ele escuta em ``0.0.0.0:3000``, com a opção de habilitar HTTP/2 e HTTPS se certificados forem fornecidos.

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
    res.render('index', { title: 'Oi', message: 'Olá!' });
});

app.get('/', async (req, res) => {
    res.send('Olá, Mundo');
});

app.get('/json', async (req, res) => {
    res.json({ olá: 'mundo' });
});

app.get('/user/:id', async (req, res) => {
    res.send('Usuário ' + req.params.id);
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
        `Ouvindo em http://${addr.address}:${addr.port}`,
    );
})
.catch(err => {
    throw Error(err.message);
});
```
