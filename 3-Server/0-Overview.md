# Servidor

O `@cmmv/server` é um servidor minimalista escrito em TypeScript, projetado para manter a mesma estrutura e métodos do [Express](https://expressjs.com/), enquanto resolve problemas de desempenho e introduz novos recursos para a entrega eficiente de componentes e arquivos estáticos.

Este projeto incorpora código do [Express](https://github.com/expressjs/express), [Koa](https://github.com/koajs/koa) e [Fastify](https://github.com/fastify/fastify), mas foi completamente reescrito em TypeScript com foco na modernidade e desempenho. Além disso, integra recursos do Vite para otimizar a entrega de componentes e ativos, garantindo uma experiência mais ágil para aplicativos modernos.

Devido à complexidade do projeto, ele foi separado em outro repositório [cmmv-server monorepo](https://github.com/cmmvio/cmmv-server), que contém múltiplos pacotes. Além do servidor principal, vários módulos foram implementados, incluindo:

* body-parser
* compression
* cookie-parser
* cors
* etag
* helmet
* server-static

Abaixo, discutiremos cada módulo em mais detalhes.

Atualmente, o projeto está em fase de testes e, portanto, não é recomendado para uso em produção.

## Recursos

* Reescrito totalmente em **TypeScript**.
* Definições dinâmicas de propriedade (como `Object.defineProperty`) foram removidas para evitar problemas graves de desempenho.
* Sistema de hooks inspirado no **Fastify** para um gerenciamento flexível do ciclo de vida das requisições.
* Suporte para **HTTP/2**.
* Suporte a compressão para **Brotli**, **gzip** e **deflate**.
* Implementação de **ETag** usando **FNV-1a**.
* Suporte embutido para algoritmos criptográficos, como **MD5**, **SHA-1** e outros do módulo `crypto`.
* Desempenho superior em comparação com **Koa**, **Hapi** e **Express**, com otimizações contínuas para alcançar desempenho semelhante ao **Fastify**.
* Total compatibilidade com todos os métodos fornecidos pelo **Express.js**.

## Benchmarks

* [https://github.com/fastify/benchmarks](https://github.com/fastify/benchmarks)
* Máquina: linux x64 | 32 vCPUs | 128.0GB Mem
* Node: v20.17.0
* Execução: Qui Nov 26 2024 15:23:41 GMT+0000 (Horário Universal Coordenado)
* Método: `autocannon -c 100 -d 40 -p 10 localhost:3000`

| Framework                | Version  | Router | Requests/s | Latency (ms) | Throughput/Mb |
|--------------------------|----------|--------|------------|--------------|---------------|
| bare                     | v20.17.0 | ✗      | 88267.6    | 10.87        | 15.74         |
| fastify                  | 5.1.0    | ✓      | 87846.6    | 10.91        | 15.75         |
| polka                    | 0.5.2    | ✓      | 87234.8    | 10.99        | 15.56         |
| connect                  | 3.7.0    | ✗      | 86129.8    | 11.13        | 15.36         |
| connect-router           | 1.3.8    | ✓      | 85804.8    | 11.16        | 15.30         |
| server-base              | 7.1.32   | ✗      | 85724.8    | 11.18        | 15.29         |
| rayo                     | 1.4.6    | ✓      | 85504.6    | 11.21        | 15.25         |
| server-base-router       | 7.1.32   | ✓      | 84189.0    | 11.39        | 15.01         |
| micro                    | 10.0.1   | ✗      | 81955.2    | 11.70        | 14.62         |
| micro-route              | 2.5.0    | ✓      | 81153.6    | 11.82        | 14.47         |
| cmmv                     | 0.6.2    | ✓      | 79041.6    | 12.16        | 14.17         |
| koa                      | 2.15.3   | ✗      | 76639.6    | 12.54        | 13.67         |
| polkadot                 | 1.0.0    | ✗      | 72702.4    | 13.25        | 12.96         |
| koa-isomorphic-router    | 1.0.1    | ✓      | 72588.4    | 13.28        | 12.95         |
| hono                     | 4.6.12   | ✓      | 72410.8    | 13.31        | 12.91         |
| take-five                | 2.0.0    | ✓      | 71261.2    | 13.54        | 25.62         |
| 0http                    | 3.5.3    | ✓      | 71047.6    | 13.58        | 12.67         |
| restana                  | 4.9.9    | ✓      | 68919.6    | 14.01        | 12.29         |
| koa-router               | 12.0.1   | ✓      | 67593.6    | 14.31        | 12.05         |
| h3-router                | 1.13.0   | ✓      | 66985.2    | 14.44        | 11.95         |
| microrouter              | 3.1.3    | ✓      | 62076.0    | 15.61        | 11.07         |
| h3                       | 1.13.0   | ✗      | 60265.6    | 16.10        | 10.75         |
| hapi                     | 21.3.12  | ✓      | 58199.2    | 16.68        | 10.38         |
| restify                  | 11.1.0   | ✓      | 57493.6    | 16.89        | 10.36         |
| fastify-big-json         | 5.1.0    | ✓      | 21931.2    | 45.09        | 252.32        |
| express                  | 5.0.1    | ✓      | 21549.2    | 45.89        | 3.84          |
| express-with-middlewares | 5.0.1    | ✓      | 18930.4    | 52.30        | 7.04          |
| trpc-router              | 10.45.2  | ✓      | N/A        | N/A          | N/A           |

## Instalação

Instale o pacote `@cmmv/server` via npm:

```bash
$ pnpm add @cmmv/server
```

## Início Rápido

Abaixo está um exemplo simples de como criar uma nova aplicação CMMV:

```typescript
import cmmv, { json, urlencoded, serverStatic } from '@cmmv/server';

const app = cmmv();

app.use(serverStatic('public'));
app.use(json({ limit: '50mb' }));
app.use(urlencoded({ limit: '50mb', extended: true }));

app.get('/', async (req, res) => {
    res.send('Hello World');
});

app.listen({ host: "0.0.0.0", port: 3000 })
.then(server => {
    const addr = server.address();
    console.log(
        `Listen on http://\${addr.address}:\${addr.port}`,
    );
})
.catch(err => {
    throw Error(err.message);
});
```

## Aplicação

A classe Application é central para o `@cmmv/server` e fornece uma base flexível para gerenciar instâncias de servidor HTTP/HTTP2, rotas, middlewares e tratamento de erros. 

**Recursos:**
* Suporte a HTTP/HTTP2.
* Roteamento com métodos HTTP (GET, POST, DELETE, etc.).
* Middleware via `use()`.
* Hooks personalizáveis para request lifecycle.
* Configurações ajustáveis como timeouts, limites de tamanho de corpo, etc.

| Opção                     | Descrição                                                                                                                                                           | Tipo                | Padrão         |
|---------------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------|----------------|
| `http2`                  | Habilita o suporte a HTTP2. Quando `true`, a instância do servidor usará HTTP2 para comunicação.                                                                    | `boolean`           | `false`        |
| `https`                  | Configuração para HTTPS (incluindo chaves e certificados). Necessário para HTTP2 seguro.                                                                            | `object`            | `undefined`    |
| `connectionTimeout`      | O tempo (em milissegundos) que o servidor esperará antes de fechar conexões inativas.                                                                               | `number`            | `0`            |
| `keepAliveTimeout`       | O tempo (em milissegundos) que o servidor esperará por uma conexão keep-alive antes de fechá-la.                                                                    | `number`            | `72000`        |
| `maxRequestsPerSocket`   | O número máximo de requisições permitidas por socket.                                                                                                               | `number`            | `0`            |
| `requestTimeout`         | O tempo (em milissegundos) que o servidor esperará para a conclusão da requisição antes de expirar.                                                                 | `number`            | `0`            |
| `bodyLimit`              | O tamanho máximo (em bytes) permitido para o corpo da requisição.                                                                                                   | `number`            | `1048576`      |
| `maxHeaderSize`          | O tamanho máximo (em bytes) permitido para os cabeçalhos da requisição.                                                                                            | `number`            | `16384`        |
| `insecureHTTPParser`     | Habilita o uso de um analisador HTTP inseguro que aceita requisições HTTP não padronizadas.                                                                          | `boolean`           | `false`        |
| `joinDuplicateHeaders`   | Combina cabeçalhos HTTP duplicados em um único cabeçalho se configurado como `true`.                                                                                 | `boolean`           | `false`        |
| `querystringParser`      | Função personalizada para análise de query strings. Deve ser uma função, se fornecida.                                                                               | `function`          | `undefined`    |
| `serverFactory`          | Uma função personalizada para criar a instância do servidor, útil para configurações avançadas como clustering.                                                     | `function`          | `undefined`    |

Esta tabela lista as opções de configuração disponíveis ao criar uma instância da classe ``Application``, que podem ser usadas para ajustar seu comportamento para diferentes ambientes e casos de uso.

```typescript
import cmmv from '@cmmv/server';

const app = new cmmv({
    http2: true,
    https: {
        key: fs.readFileSync('./key.pem'),
        cert: fs.readFileSync('./cert.pem')
    },
    requestTimeout: 30000
});

app.listen({ port: 8080, host: '0.0.0.0' });
```

Este exemplo cria um servidor HTTP2 seguro usando a classe Application com tempos limite personalizados para requisições e ouve na porta 8080. A flexibilidade da classe Application permite o registro dinâmico de middlewares, hooks personalizados e controle total sobre o comportamento do servidor.

## Roteador

A classe `Router` é responsável por definir, registrar e gerenciar rotas no `@cmmv/server`. Ela utiliza o módulo [`find-my-way`](https://github.com/delvedor/find-my-way) para lidar com o roteamento de métodos HTTP de maneira eficiente.

**Recursos do Roteador**:
* **Gerenciamento de Rotas:** Permite registrar rotas para todos os métodos HTTP, incluindo métodos personalizados como `BIND`, `MKCOL` e outros.
* **Suporte a Middleware:** Suporta empilhamento de middlewares para cada rota, permitindo a adição de vários manipuladores por rota.
* **Manipulação de Parâmetros:** Fornece um mecanismo para lidar com parâmetros dentro das rotas via o método `param()`, semelhante à forma como o Express lida com parâmetros de rota.
* **Manipulação de Caminhos Dinâmicos:** Resolve caminhos dinâmicos de maneira eficiente e pode aplicar múltiplos middlewares à mesma rota.
* **Tratamento de Erros:** Inclui mecanismos para garantir que manipuladores de rota válidos sejam definidos, com tratamento de erros para manipuladores ausentes ou inválidos.
* **Compatibilidade:** Embora forneça uma API semelhante ao Express.js para definições de rotas, não depende do Express e opera de forma independente.

A biblioteca `find-my-way` é um roteador HTTP de alto desempenho que combina solicitações a rotas registradas. Ela fornece recursos como:

* **Roteamento Dinâmico:** Suporta segmentos de caminho dinâmicos e parâmetros (ex.: `/users/:id`).
* **Vários Métodos:** As rotas podem ser registradas para qualquer método HTTP (GET, POST, etc.).
* **Sensibilidade a Maiúsculas e Minúsculas:** As rotas são insensíveis a maiúsculas e minúsculas por padrão, mas podem ser configuradas de outra forma.
* **Desempenho:** Projetado para eficiência, suporta tempos de busca rápidos para um grande número de rotas.
* **Empilhamento de Middleware:** Permite anexar uma matriz de funções de middleware a uma única rota.

A classe Router fornece métodos para cada método HTTP e permite definições de rota. Abaixo está uma tabela resumindo os métodos e sua funcionalidade:

| Método       | Descrição                                                            | Exemplo de Uso                             |
|--------------|----------------------------------------------------------------------|--------------------------------------------|
| `acl()`      | Registra uma rota para o método ACL.                                 | `router.acl('/resources', handler)`        |
| `bind()`     | Registra uma rota para o método BIND.                                | `router.bind('/binding', handler)`         |
| `checkout()` | Registra uma rota para o método CHECKOUT.                            | `router.checkout('/checkout', handler)`   |
| `connect()`  | Registra uma rota para o método CONNECT.                             | `router.connect('/connect', handler)`     |
| `copy()`     | Registra uma rota para o método COPY.                                | `router.copy('/copy', handler)`           |
| `delete()`   | Registra uma rota para o método DELETE.                              | `router.delete('/resource/:id', handler)` |
| `get()`      | Registra uma rota para o método GET. Também registra HEAD por padrão. | `router.get('/users', handler)`           |
| `head()`     | Registra uma rota para o método HEAD.                                | `router.head('/headers', handler)`        |
| `link()`     | Registra uma rota para o método LINK.                                | `router.link('/link', handler)`           |
| `lock()`     | Registra uma rota para o método LOCK.                                | `router.lock('/lock', handler)`           |
| `m-search()` | Registra uma rota para o método M-SEARCH.                            | `router['m-search']('/search', handler)`  |
| `merge()`    | Registra uma rota para o método MERGE.                               | `router.merge('/merge', handler)`         |
| `mkactivity()`| Registra uma rota para o método MKACTIVITY.                         | `router.mkactivity('/activity', handler)` |
| `mkcalendar()`| Registra uma rota para o método MKCALENDAR.                         | `router.mkcalendar('/calendar', handler)` |
| `mkcol()`    | Registra uma rota para o método MKCOL.                               | `router.mkcol('/col', handler)`           |
| `move()`     | Registra uma rota para o método MOVE.                                | `router.move('/move', handler)`           |
| `notify()`   | Registra uma rota para o método NOTIFY.                              | `router.notify('/notify', handler)`       |
| `options()`  | Registra uma rota para o método OPTIONS.                             | `router.options('/options', handler)`     |
| `patch()`    | Registra uma rota para o método PATCH.                               | `router.patch('/update', handler)`        |
| `post()`     | Registra uma rota para o método POST.                                | `router.post('/submit', handler)`         |
| `propfind()` | Registra uma rota para o método PROPFIND.                            | `router.propfind('/find', handler)`       |
| `proppatch()`| Registra uma rota para o método PROPPATCH.                           | `router.proppatch('/patch', handler)`     |
| `purge()`    | Registra uma rota para o método PURGE.                               | `router.purge('/purge', handler)`         |
| `put()`      | Registra uma rota para o método PUT.                                 | `router.put('/resource', handler)`        |
| `rebind()`   | Registra uma rota para o método REBIND.                              | `router.rebind('/rebind', handler)`       |
| `report()`   | Registra uma rota para o método REPORT.                              | `router.report('/report', handler)`       |
| `search()`   | Registra uma rota para o método SEARCH.                              | `router.search('/search', handler)`       |
| `source()`   | Registra uma rota para o método SOURCE.                              | `router.source('/source', handler)`       |
| `subscribe()`| Registra uma rota para o método SUBSCRIBE.                           | `router.subscribe('/subscribe', handler)` |
| `trace()`    | Registra uma rota para o método TRACE.                               | `router.trace('/trace', handler)`         |
| `unbind()`   | Registra uma rota para o método UNBIND.                              | `router.unbind('/unbind', handler)`       |
| `unlink()`   | Registra uma rota para o método UNLINK.                              | `router.unlink('/unlink', handler)`       |
| `unlock()`   | Registra uma rota para o método UNLOCK.                              | `router.unlock('/unlock', handler)`       |
| `unsubscribe()`| Registra uma rota para o método UNSUBSCRIBE.                       | `router.unsubscribe('/unsubscribe', handler)`|

Exemplo

```typescript
// Inicializa o roteador
const router = new Router();

// Registra rota GET
router.get('/users', (req, res) => {
  res.send('Lista de usuários');
});

// Registra rota POST
router.post('/users', (req, res) => {
  res.send('Usuário criado');
});

// Registra rota PUT
router.put('/users/:id', (req, res) => {
  res.send(`Usuário \${req.params.id} atualizado`);
});

// Registra rota DELETE
router.delete('/users/:id', (req, res) => {
  res.send(`Usuário \${req.params.id} deletado`);
});
```

Neste exemplo, a classe `Router` é usada para definir rotas que lidam com diferentes métodos HTTP, como GET, POST, PUT e DELETE. Cada rota aceita uma solicitação (`req`) e uma resposta (`res`), permitindo que os manipuladores gerenciem as solicitações recebidas e enviem respostas apropriadas.

Exemplo com Aplicação

```typescript
import cmmv from '@cmmv/server';

const app = cmmv();
const host = '0.0.0.0';
const port = 3000;

app.set('view engine', 'pug');

app.get('/view', function (req, res) {
    res.render('index', { title: 'Oi', message: 'Olá Mundo!' });
});

app.get('/', async (req, res) => {
    res.send('Olá Mundo');
});

app.get('/json', async (req, res) => {
    res.json({ hello: 'mundo' });
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

app.listen({ host, port });
```

## Estático

O middleware `@cmmv/server-static` é usado para servir arquivos estáticos de um diretório ou diretórios especificados. Ele foi projetado para lidar com solicitações de conteúdo estático, como HTML, CSS, JavaScript, imagens ou quaisquer outros assets. O middleware oferece várias opções de configuração para o comportamento de entrega de arquivos, incluindo cache, compressão, cabeçalhos personalizados e mais.

Após a inicialização do middleware, ele escuta solicitações que correspondem ao prefixo especificado e serve arquivos do diretório raiz (ou múltiplos diretórios) conforme configurado.

| Opção          | Descrição                                                                                                                                                                                   | Tipo                     | Padrão         |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|--------------------------|-----------------|
| `root`          | Especifica o diretório raiz (ou diretórios) de onde os arquivos estáticos serão servidos.                                                                                                         | `string` ou `string[]`    | `undefined` (Obrigatório) |
| `prefix`        | Prefixo da URL para servir arquivos estáticos. Os arquivos serão servidos de `prefix + path` (por exemplo, `/static/myfile.js`).                                                                                   | `string`                  | `'/'`           |
| `maxAge`        | Define a diretiva Cache-Control max-age em milissegundos. Determina por quanto tempo os arquivos devem ser armazenados em cache pelo cliente.                                                                            | `number`                  | `0`             |
| `cacheControl`  | Ativa ou desativa o cabeçalho Cache-Control. Quando ativado, cabeçalhos de cache serão configurados automaticamente com base na opção `maxAge`.                                                              | `boolean`                 | `true`          |
| `dotfiles`      | Define como lidar com arquivos ocultos (arquivos que começam com `.`). As opções são `'allow'`, `'deny'` ou `'ignore'`.                                                                                      | `'allow'`, `'deny'`, `'ignore'` | `'allow'`     |
| `serverDotfiles`| Se ativado, o servidor irá servir arquivos ocultos do diretório raiz.                                                                                                                            | `boolean`                 | `false`         |
| `index`         | Especifica o arquivo padrão a ser servido quando um diretório é solicitado. Pode ser uma string (por exemplo, `'index.html'`) ou `false` para desativar o serviço de arquivos de índice.                                             | `string` ou `boolean`     | `'index.html'`  |
| `fallthrough`   | Quando `true`, permite que solicitações passem para o próximo middleware se um arquivo não for encontrado. Se `false`, será retornado um erro 404.                                                          | `boolean`                 | `true`          |
| `redirect`      | Redireciona solicitações para diretórios que não terminam com `/` adicionando uma barra final.                                                                                                        | `boolean`                 | `true`          |
| `immutable`     | Quando configurado como `true`, serve arquivos com a diretiva `Cache-Control: immutable`, indicando que os arquivos nunca mudarão.                                                                   | `boolean`                 | `false`         |
| `lastModified`  | Ativa ou desativa o cabeçalho Last-Modified. Quando ativado, define o cabeçalho Last-Modified com base no tempo de última modificação do arquivo.                                                           | `boolean`                 | `true`          |
| `etag`          | Ativa ou desativa a geração de cabeçalhos ETag, que podem ser usados para validação de cache.                                                                                                    | `boolean`                 | `true`          |
| `extensions`    | Extensões de arquivo para tentar quando um arquivo não for encontrado (por exemplo, servir `file.html` quando a solicitação for para `file`). Pode ser um array de strings.                                                        | `string[]` ou `boolean`   | `false`         |
| `acceptRanges`  | Ativa suporte para solicitações de intervalo HTTP, útil para streaming de mídia.                                                                                                                           | `boolean`                 | `true`          |
| `preCompressed` | Se `true`, o middleware tentará servir arquivos pré-comprimidos (por exemplo, arquivos `.br` ou `.gz`) se disponíveis.                                                                                | `boolean`                 | `false`         |
| `allowedPath`   | Uma função que pode ser usada para filtrar ou restringir o acesso a caminhos específicos. A função recebe `pathname`, `root` e `req` como parâmetros e retorna `true` para permitir o caminho ou `false` para rejeitá-lo. | `Function` | `undefined` |
| `setHeaders`    | Uma função personalizada para definir cabeçalhos para a resposta. Ela recebe `res`, `path` e `stat` como parâmetros e pode modificar os cabeçalhos antes que a resposta seja enviada.                                     | `Function`                | `null`          |

Este middleware não é compatível com Express porque seu funcionamento interno difere significativamente do servidor de arquivos estáticos do Express. Uma grande mudança é a remoção do suporte a listagem de diretórios. Em vez disso, o middleware verifica arquivos existentes e cria rotas na inicialização do aplicativo, resultando em uma eficiência aprimorada.

Ele aproveita `[@fastify/send](https://github.com/fastify/send)` e `[@fastify/accept-negotiator](https://github.com/fastify/accept-negotiator)` para entregar arquivos de forma eficiente. Além disso, suporta complementos como `@cmmv/etag` e `@cmmv/compression` para uma entrega aprimorada de arquivos estáticos, oferecendo controle de cache por meio do uso de cabeçalhos `lastModified` e `ETag`. Esses recursos fornecem um servidor de arquivos estáticos mais otimizado e com melhor desempenho em comparação com abordagens tradicionais.

## JSON

O middleware ``@cmmv/body-parser`` no cmmv-server serve como uma função embutida projetada para lidar com solicitações de entrada com payloads JSON. Semelhante ao Express, este middleware analisa o corpo da solicitação e anexa os dados analisados ao objeto ``req.body``, tornando-o facilmente acessível para processamento adicional.

Este middleware processa especificamente dados JSON e somente processa solicitações onde o cabeçalho ``Content-Type: application/json`` ou ``application/vnd.api+json`` corresponde à opção de tipo especificada. Ele suporta qualquer codificação Unicode e pode inflar automaticamente solicitações compactadas usando gzip ou deflate.

Após a execução do middleware, os dados JSON analisados estarão disponíveis em ``req.body``. Se não houver um corpo para analisar, o ``Content-Type`` não corresponder, ou ocorrer um erro, ``req.body`` será um objeto vazio (``{}``).

Como ``req.body`` é preenchido a partir de entrada fornecida pelo usuário, é importante observar que todas as propriedades e valores no objeto são não confiáveis. Eles devem ser cuidadosamente validados antes de serem usados. Por exemplo, tentar acessar ``req.body.foo.toString()`` pode resultar em erros se ``foo`` for indefinido, não for uma string ou se o método ``toString`` tiver sido sobrescrito ou substituído por entrada maliciosa.

A tabela a seguir descreve as opções disponíveis para configurar este middleware, oferecendo personalização sobre seu comportamento para atender às necessidades de várias aplicações.

| Propriedade  | Descrição                                                                                                                                                                                                                                                                                                                                                                             | Tipo     | Padrão          |
|--------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|------------------|
| `inflate`    | Ativa ou desativa o tratamento de corpos compactados (deflated). Quando desativado, corpos compactados são rejeitados.                                                                                                                                                                                                                                           | Booleano  | `true`           |
| `limit`      | Controla o tamanho máximo do corpo da solicitação. Se for um número, o valor especifica o número de bytes; se for uma string, o valor é passado para a biblioteca `bytes` para análise.                                                                                                                                                                   | Misto    | `"100kb"`        |
| `reviver`    | A opção `reviver` é passada diretamente para `JSON.parse` como o segundo argumento. Ela permite a transformação personalizada de dados JSON analisados. Mais detalhes podem ser encontrados na [documentação MDN sobre JSON.parse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse#using_the_reviver_parameter).               | Função   | `null`           |
| `strict`     | Ativa ou desativa a aceitação de apenas arrays e objetos como JSON válido. Quando desativado, o middleware aceitará qualquer coisa que `JSON.parse` aceitar.                                                                                                                                                                                                | Booleano  | `true`           |
| `type`       | Determina o tipo de mídia que o middleware analisará. Pode ser uma string, um array de strings ou uma função. Se não for uma função, o valor é passado diretamente para a biblioteca `type-is` e pode ser uma extensão (por exemplo, `json`), um tipo MIME (por exemplo, `application/json`) ou um tipo MIME curinga (por exemplo, `*/json`). Se for uma função, ela será chamada como `fn(req)` e analisará quando retornar `true`. | Misto    | `"application/json"` |
| `verify`     | Se fornecida, esta função é chamada como `verify(req, res, buf, encoding)`, onde `buf` é o corpo bruto da solicitação. O processo de análise pode ser interrompido lançando um erro a partir desta função.                                                                                                                                                        | Função   | `undefined`      |

## Raw

Este middleware processa payloads de solicitação de entrada e os converte em um Buffer, aproveitando a funcionalidade inspirada no ``body-parser``. Ele é projetado especificamente para lidar com corpos como Buffers e somente processa solicitações onde o cabeçalho ``Content-Type: application/octet-stream`` ou ``application/vnd+octets`` corresponde à opção de tipo definida.

O analisador é capaz de lidar com qualquer codificação Unicode e suporta descompressão automática para solicitações codificadas com gzip e deflate.

Após a execução do middleware, um Buffer contendo os dados analisados é atribuído à propriedade ``req.body``. Se nenhum corpo for encontrado, o ``Content-Type`` não corresponder, ou ocorrer um erro, ``req.body`` será por padrão um objeto vazio (``{}``) ou permanecerá inalterado se outro analisador já o tiver processado.

Como ``req.body`` é preenchido com base na entrada do usuário, é crucial validar quaisquer propriedades e valores antes de usá-los. Por exemplo, chamar ``req.body.toString()`` pode levar a erros se ``req.body`` tiver sido alterado por vários analisadores. É altamente recomendado verificar se ``req.body`` é um Buffer antes de realizar quaisquer operações específicas de Buffer.

A tabela a seguir fornece uma visão geral das opções de configuração opcionais:

| Propriedade  | Descrição                                                                                                                                                                                                                                                                                                              | Tipo     | Padrão                    |
|--------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|----------------------------|
| `inflate`    | Ativa ou desativa o tratamento de corpos compactados (deflated). Quando desativado, corpos compactados são rejeitados.                                                                                                                                                                                                                  | Booleano  | `true`                     |
| `limit`      | Controla o tamanho máximo do corpo da solicitação. Se for um número, especifica o número de bytes. Se for uma string, o valor é passado para a biblioteca `bytes` para análise.                                                                                                                                               | Misto    | `"100kb"`                  |
| `type`       | Determina o tipo de mídia que o middleware analisará. Pode ser uma string, um array de strings ou uma função. Se não for uma função, é passada para a biblioteca `type-is` e pode ser uma extensão (por exemplo, `bin`), um tipo MIME (por exemplo, `application/octet-stream`) ou um padrão curinga (por exemplo, `application/*`).         | Misto    | `"application/octet-stream"` |
| `verify`     | Uma função chamada como `verify(req, res, buf, encoding)` onde `buf` é o corpo bruto da solicitação e `encoding` é sua codificação. Lançar um erro a partir desta função interrompe o processo de análise.                                                                                                                               | Função   | `undefined`                |

## Text

Este middleware processa payloads de solicitações de entrada, convertendo-os em uma string, utilizando a funcionalidade do módulo ``body-parser``.

Ele fornece um middleware que analisa todos os corpos como strings, processando apenas solicitações onde o cabeçalho ``Content-Type: text/plain`` corresponda ao tipo especificado. Ele lida com várias codificações Unicode e suporta descompressão automática de solicitações compactadas com gzip e deflate.

Após a execução do middleware, uma nova representação em string dos dados analisados é atribuída ao objeto ``req.body``. Se não houver corpo para analisar, o ``Content-Type`` não corresponder ou ocorrer um erro, ``req.body`` será por padrão um objeto vazio (``{}``).

Como a estrutura de ``req.body`` é baseada na entrada do usuário, é importante tratá-la como não confiável. Cada propriedade e valor deve ser validado antes do uso. Por exemplo, tentar chamar ``req.body.trim()`` pode levar a erros se ``req.body`` não for uma string. Portanto, é recomendável garantir que ``req.body`` seja uma string antes de usar métodos de string.

A tabela a seguir descreve as opções de configuração disponíveis:

| Propriedade        | Descrição                                                                                                                                                                                                                                                   | Tipo     | Padrão       |
|--------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|--------------|
| `defaultCharset`   | Especifica o conjunto de caracteres padrão para o conteúdo de texto quando o charset não está incluído no cabeçalho `Content-Type` da solicitação.                                                                                                            | String   | `"utf-8"`    |
| `inflate`          | Controla se os corpos de solicitações compactados (deflated) devem ser processados. Se desativado, corpos compactados serão rejeitados.                                                                                                                       | Booleano | `true`       |
| `limit`            | Define o tamanho máximo permitido para o corpo da solicitação. Quando fornecido como um número, o valor é tratado como o número de bytes. Se fornecido como uma string, ele é analisado usando a biblioteca `bytes`.                                           | Misto    | `"100kb"`    |
| `type`             | Define o tipo de mídia que o middleware deve analisar. Esta opção pode ser uma string, um array de strings ou uma função. Se não for uma função, o valor é passado para a biblioteca `type-is` e pode ser uma extensão (por exemplo, `txt`), um tipo MIME (por exemplo, `text/plain`), ou um padrão curinga (por exemplo, `text/*`). Se for uma função, ela será chamada como `fn(req)` e a solicitação será analisada se retornar um valor verdadeiro. | Misto    | `"text/plain"` |
| `verify`           | Se fornecida, esta função é invocada como `verify(req, res, buf, encoding)`, onde `buf` é o corpo bruto da solicitação em formato Buffer, e `encoding` é a codificação da solicitação. O processo de análise pode ser interrompido lançando um erro dentro desta função.                              | Função   | `undefined`  |

## Urlencoded

Este middleware é projetado para analisar solicitações de entrada que contêm payloads codificados como URL. Ele é baseado na funcionalidade fornecida pelo body-parser.

O middleware processa apenas corpos codificados como URL, especificamente procurando solicitações onde o cabeçalho ``Content-Type: application/x-www-form-urlencoded`` corresponda à opção de tipo especificada. Este analisador aceita corpos codificados em UTF-8 e suporta descompressão automática para solicitações compactadas com gzip ou deflate.

Após o processamento da solicitação pelo middleware, ele popula o objeto ``req.body`` com os dados analisados. Se nenhum corpo for encontrado, o ``Content-Type`` não corresponder ou ocorrer um erro, ``req.body`` será definido como um objeto vazio (``{}``). O objeto conterá pares de chave-valor, onde os valores podem ser strings ou arrays (quando `extended: false`), ou qualquer tipo de dado (quando `extended: true`).

Como ``req.body`` é preenchido com base na entrada do usuário, é importante validar todas as propriedades e valores antes de usá-los. Por exemplo, chamar ``req.body.foo.toString()`` pode causar erros se `foo` não for uma string ou se estiver indefinido. Portanto, é aconselhável verificar o tipo dos valores de ``req.body`` antes de realizar quaisquer operações sobre eles.

A tabela a seguir descreve as opções de configuração disponíveis para o middleware:

| Propriedade         | Descrição                                                                                                                                                                                                                                           | Tipo     | Padrão                           |
|---------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|-----------------------------------|
| `extended`          | Permite escolher entre analisar dados codificados como URL com a biblioteca `querystring` (quando `false`) ou com a biblioteca `qs` (quando `true`). A sintaxe "extended" permite objetos e arrays mais complexos serem codificados no formato URL.   | Booleano | `true`                            |
| `inflate`           | Ativa ou desativa o suporte para lidar com corpos de solicitações compactados (deflated). Se desativado, corpos compactados serão rejeitados.                                                                                                         | Booleano | `true`                            |
| `limit`             | Define o tamanho máximo permitido para o corpo da solicitação. Se fornecido como um número, especifica o número de bytes. Se fornecido como uma string, será analisado pela biblioteca `bytes`.                                                       | Misto    | `"100kb"`                         |
| `parameterLimit`    | Controla o número máximo de parâmetros que podem estar presentes nos dados codificados como URL. Se a solicitação exceder esse limite, um erro será lançado.                                                                                           | Número   | `1000`                            |
| `type`              | Define quais tipos de mídia o middleware processará. Isso pode ser especificado como uma string, um array de strings ou uma função. Se não for uma função, este valor é passado para a biblioteca `type-is` e pode ser uma extensão (por exemplo, `urlencoded`), um tipo MIME (por exemplo, `application/x-www-form-urlencoded`), ou um padrão curinga. | Misto    | `"application/x-www-form-urlencoded"` |
| `verify`            | Uma função que é invocada como `verify(req, res, buf, encoding)`, onde `buf` é o corpo bruto da solicitação em formato Buffer. Lançar um erro a partir desta função interromperá o processo de análise.                                                | Função   | `undefined`                       |
