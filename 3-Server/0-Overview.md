# Servidor

Repositório: [https://github.com/cmmvio/cmmv-server](https://github.com/cmmvio/cmmv-server)

O ``@cmmv/server`` é um servidor minimalista escrito em TypeScript, projetado para manter a mesma estrutura e métodos do [Express](https://expressjs.com/), ao mesmo tempo que aborda problemas de desempenho e introduz novos recursos para entrega eficiente de componentes e arquivos estáticos.

Este projeto incorpora código do [Express](https://github.com/expressjs/express), [Koa](https://github.com/koajs/koa) e [Fastify](https://github.com/fastify/fastify), mas foi completamente reescrito em TypeScript com foco em modernidade e desempenho. Além disso, integra funcionalidades do Vite para otimizar a entrega de componentes e ativos, garantindo uma experiência mais rápida e ágil para aplicações modernas.

Devido à complexidade do projeto, ele foi separado em outro repositório [cmmv-server monorepo](https://github.com/cmmvio/cmmv-server), que contém vários pacotes. Além do servidor principal, diversos módulos foram implementados, incluindo:

* body-parser ([https://github.com/cmmvio/cmmv-server/tree/main/packages/body-parser](https://github.com/cmmvio/cmmv-server/tree/main/packages/body-parser))
* compression ([https://github.com/cmmvio/cmmv-server/tree/main/packages/compression](https://github.com/cmmvio/cmmv-server/tree/main/packages/compression))
* cookie-parser ([https://github.com/cmmvio/cmmv-server/tree/main/packages/cookie-parser](https://github.com/cmmvio/cmmv-server/tree/main/packages/cookie-parser))
* cors ([https://github.com/cmmvio/cmmv-server/tree/main/packages/cors](https://github.com/cmmvio/cmmv-server/tree/main/packages/cors))
* etag ([https://github.com/cmmvio/cmmv-server/tree/main/packages/etag](https://github.com/cmmvio/cmmv-server/tree/main/packages/etag))
* helmet ([https://github.com/cmmvio/cmmv-server/tree/main/packages/helmet](https://github.com/cmmvio/cmmv-server/tree/main/packages/helmet))
* server-static ([https://github.com/cmmvio/cmmv-server/tree/main/packages/server-static](https://github.com/cmmvio/cmmv-server/tree/main/packages/server-static))

Abaixo, discutiremos cada módulo em mais detalhes.

Atualmente, o projeto está em fase de testes e, portanto, não é recomendado para uso em produção.

## Recursos

* Totalmente reescrito em **TypeScript**.
* **Definições dinâmicas de propriedades** (como ``Object.defineProperty``) foram removidas para evitar sérios problemas de desempenho.
* Sistema de **hooks inspirado no Fastify** para gerenciamento flexível do ciclo de vida das requisições.
* Suporte a **HTTP/2**.
* Suporte a compressão para **Brotli**, **gzip** e **deflate**.
* Implementação de **ETag** usando **FNV-1a**.
* Suporte embutido para algoritmos criptográficos como **MD5**, **SHA-1** e outros do módulo ``crypto``.
* Desempenho superior ao **Koa**, **Hapi** e **Express**, com otimização contínua para alcançar desempenho semelhante ao **Fastify**.
* Compatibilidade total com todos os métodos fornecidos pelo **Express.js**.

## Benchmarks

* [https://github.com/fastify/benchmarks](https://github.com/fastify/benchmarks)
* Machine: linux x64 | 32 vCPUs | 128.0GB Mem
* Node: v20.17.0
* Run: Sun Mar 16 2025 14:51:12 GMT+0000 (Coordinated Universal Time)
* Method: ``autocannon -c 100 -d 40 -p 10 localhost:3000``

|                          | Version  | Router | Requests/s | Latency (ms) | Throughput/Mb |
|--------------------------|----------|--------|------------|--------------|---------------|
| bare                     | v20.17.0 | ✗      | 51166.4    | 19.19        | 9.13          |
| cmmv                     | 0.9.4    | ✓      | 46879.2    | 21.03        | 8.40          |
| fastify                  | 5.2.1    | ✓      | 46488.0    | 21.19        | 8.33          |
| h3                       | 1.15.1   | ✗      | 34626.2    | 28.37        | 6.18          |
| restify                  | 11.1.0   | ✓      | 34020.6    | 28.88        | 6.07          |
| koa                      | 2.16.0   | ✗      | 31031.0    | 31.72        | 5.53          |
| express                  | 5.0.1    | ✓      | 12913.6    | 76.87        | 2.30          |

## Instalação

Instale o pacote ``@cmmv/server`` via npm:

```bash
$ pnpm add @cmmv/server
```

## Início Rápido

Abaixo está um exemplo simples de como criar uma nova aplicação CMMV:

```typescript
import cmmv, { json, urlencoded, serverStatic } from '@cmmv/server';

const app = cmmv({
    /*http2: true,
    https: {
        key: readFileSync("./cert/private-key.pem"),
        cert: readFileSync("./cert/certificate.pem"),
        passphrase: "1234"
    }*/
});

app.use(serverStatic('public'));
app.use(json({ limit: '50mb' }));
app.use(urlencoded({ limit: '50mb', extended: true }));

app.get('/', async (req, res) => {
    res.send('Olá, Mundo');
});

app.listen({ host: "0.0.0.0", port: 3000 })
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

## Aplicação

A classe ``Application`` é central no ``@cmmv/server`` e fornece uma base flexível para gerenciar instâncias de servidores HTTP/HTTP2, roteamento, middlewares e tratamento de erros. Ela é construída sobre o ``EventEmitter`` do Node.js e integra funcionalidades como análise de consultas, parsers de tipo de conteúdo e manipuladores de métodos HTTP.

Essa classe é responsável por gerenciar o ciclo de vida das requisições HTTP, direcionando-as aos manipuladores corretos e aplicando middlewares e hooks (como pré-análise, on-request e tratamento de erros). A classe também suporta configurações personalizáveis e mecanismos de tratamento de erros. Aqui está uma visão geral de suas funcionalidades:

***Funcionalidades:***

* **Suporte a HTTP/HTTP2:** A classe ``Application`` pode criar servidores HTTP ou HTTP2, suportando tanto requisições normais quanto seguras.
* **Roteamento:** Oferece capacidades completas de roteamento para diferentes métodos HTTP (``GET``, ``POST``, etc.) por meio da classe ``Router``.
* **Middleware:** Suporta o encadeamento de middlewares via método ``use()``, incluindo o tratamento de arrays de middlewares.
* **Tratamento de Erros:** Tratamento de erros personalizado por meio de ``setErrorHandler`` que permite definir lógica específica para erros.
* **Renderização de Visões:** Usa a classe ``View`` para renderizar templates, suportando motores de visão personalizados registrados via ``app.engine()``.
* **Hooks:** Gerencia hooks do ciclo de vida como ``onRequest``, ``preParsing``, ``onError`` e mais, que podem ser usados para personalizar o tratamento de requisições.
* **Opções de Configuração:** Configurações ajustáveis do servidor como tempos limite, limites de corpo e suporte a HTTP2.

| Opção                    | Descrição                                                                                                                                                           | Tipo                | Padrão         |
|--------------------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------|---------------------|----------------|
| `http2`                  | Habilita o suporte a HTTP2. Quando `true`, a instância do servidor usará HTTP2 para comunicação.                                                                     | `boolean`           | `false`        |
| `https`                  | Configuração para HTTPS (incluindo chaves e certificados). Necessário para HTTP2 seguro.                                                                             | `object`            | `undefined`    |
| `connectionTimeout`      | O tempo (em milissegundos) que o servidor aguardará antes de fechar conexões ociosas.                                                                                | `number`            | `0`            |
| `keepAliveTimeout`       | O tempo (em milissegundos) que o servidor aguardará por uma conexão keep-alive antes de fechá-la.                                                                    | `number`            | `72000`        |
| `maxRequestsPerSocket`   | O número máximo de requisições permitidas por socket.                                                                                                               | `number`            | `0`            |
| `requestTimeout`         | O tempo (em milissegundos) que o servidor aguardará para que a requisição seja concluída antes de expirar.                                                           | `number`            | `0`            |
| `bodyLimit`              | O tamanho máximo (em bytes) para o corpo da requisição.                                                                                                             | `number`            | `1048576`      |
| `maxHeaderSize`          | O tamanho máximo (em bytes) para os cabeçalhos da requisição.                                                                                                       | `number`            | `16384`        |
| `insecureHTTPParser`     | Habilita o uso de um parser HTTP inseguro que aceita requisições HTTP não padrão.                                                                                   | `boolean`           | `false`        |
| `joinDuplicateHeaders`   | Combina cabeçalhos HTTP duplicados em um único cabeçalho se definido como `true`.                                                                                   | `boolean`           | `false`        |
| `querystringParser`      | Função personalizada para análise de query string. Deve ser uma função se fornecida.                                                                                | `function`          | `undefined`    |
| `serverFactory`          | Uma função personalizada para criar a instância do servidor, útil para configurações avançadas como clustering.                                                     | `function`          | `undefined`    |

Essa tabela lista as opções de configuração disponíveis ao criar uma instância da classe ``Application``, que podem ser usadas para ajustar seu comportamento para diferentes ambientes e casos de uso.

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

Este exemplo cria um servidor HTTP2 seguro usando a classe ``Application`` com tempos limite de requisição personalizados e escuta na porta 8080. A flexibilidade da classe ``Application`` permite registro dinâmico de middlewares, hooks personalizados e controle total sobre o comportamento do servidor.

## Roteador

A classe ``Router`` é responsável por definir, registrar e gerenciar rotas no ``@cmmv/server``. Ela utiliza o módulo [``find-my-way``](https://github.com/delvedor/find-my-way) para gerenciar o roteamento de métodos HTTP, oferecendo uma abstração para configurar rotas usando vários métodos HTTP (GET, POST, PUT, DELETE, etc.).

Ao utilizar o ``find-my-way``, a classe ``Router`` garante um roteamento eficiente, suportando recursos como manipulação de parâmetros de rota, empilhamento de middlewares e resolução dinâmica de rotas. Esse roteador suporta todos os principais métodos HTTP e permite definições flexíveis de rotas.

Funcionalidades do ``Router``:
* **Gerenciamento de Rotas:** Permite registrar rotas para todos os métodos HTTP, incluindo métodos personalizados como ``BIND``, ``MKCOL`` e mais.
* **Suporte a Middleware:** Suporta empilhamento de middlewares para cada rota, permitindo a adição de múltiplos manipuladores por rota.
* **Manipuladores de Parâmetros:** Fornece um mecanismo para lidar com parâmetros dentro de rotas via método ``param()``, semelhante ao manejo de parâmetros de rota no Express.
* **Manipulação Dinâmica de Caminhos:** Resolve caminhos dinâmicos de forma eficiente e pode aplicar múltiplos middlewares à mesma rota.
* **Tratamento de Erros:** Inclui mecanismos para garantir que manipuladores de rota válidos sejam definidos, com tratamento de erros para manipuladores ausentes ou inválidos.
* **Compatibilidade:** Embora forneça uma API semelhante ao Express.js para definições de rota, não depende do Express e opera de forma independente.

A biblioteca ``find-my-way`` é um roteador HTTP de alto desempenho que associa requisições às rotas registradas. Ela oferece recursos como:

* **Roteamento Dinâmico:** Suporta segmentos de caminho dinâmicos e parâmetros (ex.: ``/users/:id``).
* **Múltiplos Métodos:** Rotas podem ser registradas para qualquer método HTTP (GET, POST, etc.).
* **Sensibilidade a Maiúsculas:** Rotas são insensíveis a maiúsculas por padrão, mas podem ser configuradas de outra forma.
* **Desempenho:** Projetada para eficiência, suporta tempos de busca rápidos para grandes quantidades de rotas.
* **Pilha de Middleware:** Permite anexar um array de funções middleware a uma única rota.

A classe ``Router`` fornece métodos para cada método HTTP e permite definições de rotas. Abaixo está uma tabela resumindo os métodos e suas funcionalidades:

| Método          | Descrição                                                             | Exemplo de Uso                              |
|-----------------|-----------------------------------------------------------------------|---------------------------------------------|
| `acl()`         | Registra uma rota para o método ACL.                                  | `router.acl('/resources', handler)`         |
| `bind()`        | Registra uma rota para o método BIND.                                 | `router.bind('/binding', handler)`          |
| `checkout()`    | Registra uma rota para o método CHECKOUT.                             | `router.checkout('/checkout', handler)`     |
| `connect()`     | Registra uma rota para o método CONNECT.                              | `router.connect('/connect', handler)`       |
| `copy()`        | Registra uma rota para o método COPY.                                 | `router.copy('/copy', handler)`             |
| `delete()`      | Registra uma rota para o método DELETE.                               | `router.delete('/resource/:id', handler)`   |
| `get()`         | Registra uma rota para o método GET. Registra HEAD por padrão também. | `router.get('/users', handler)`             |
| `head()`        | Registra uma rota para o método HEAD.                                 | `router.head('/headers', handler)`          |
| `link()`        | Registra uma rota para o método LINK.                                 | `router.link('/link', handler)`             |
| `lock()`        | Registra uma rota para o método LOCK.                                 | `router.lock('/lock', handler)`             |
| `m-search()`    | Registra uma rota para o método M-SEARCH.                             | `router['m-search']('/search', handler)`    |
| `merge()`       | Registra uma rota para o método MERGE.                                | `router.merge('/merge', handler)`           |
| `mkactivity()`  | Registra uma rota para o método MKACTIVITY.                           | `router.mkactivity('/activity', handler)`   |
| `mkcalendar()`  | Registra uma rota para o método MKCALENDAR.                           | `router.mkcalendar('/calendar', handler)`   |
| `mkcol()`       | Registra uma rota para o método MKCOL.                                | `router.mkcol('/col', handler)`             |
| `move()`        | Registra uma rota para o método MOVE.                                 | `router.move('/move', handler)`             |
| `notify()`      | Registra uma rota para o método NOTIFY.                               | `router.notify('/notify', handler)`         |
| `options()`     | Registra uma rota para o método OPTIONS.                              | `router.options('/options', handler)`       |
| `patch()`       | Registra uma rota para o método PATCH.                                | `router.patch('/update', handler)`          |
| `post()`        | Registra uma rota para o método POST.                                 | `router.post('/submit', handler)`           |
| `propfind()`    | Registra uma rota para o método PROPFIND.                             | `router.propfind('/find', handler)`         |
| `proppatch()`   | Registra uma rota para o método PROPPATCH.                            | `router.proppatch('/patch', handler)`       |
| `purge()`       | Registra uma rota para o método PURGE.                                | `router.purge('/purge', handler)`           |
| `put()`         | Registra uma rota para o método PUT.                                  | `router.put('/resource', handler)`          |
| `rebind()`      | Registra uma rota para o método REBIND.                               | `router.rebind('/rebind', handler)`         |
| `report()`      | Registra uma rota para o método REPORT.                               | `router.report('/report', handler)`         |
| `search()`      | Registra uma rota para o método SEARCH.                               | `router.search('/search', handler)`         |
| `source()`      | Registra uma rota para o método SOURCE.                               | `router.source('/source', handler)`         |
| `subscribe()`   | Registra uma rota para o método SUBSCRIBE.                            | `router.subscribe('/subscribe', handler)`   |
| `trace()`       | Registra uma rota para o método TRACE.                                | `router.trace('/trace', handler)`           |
| `unbind()`      | Registra uma rota para o método UNBIND.                               | `router.unbind('/unbind', handler)`         |
| `unlink()`      | Registra uma rota para o método UNLINK.                               | `router.unlink('/unlink', handler)`         |
| `unlock()`      | Registra uma rota para o método UNLOCK.                               | `router.unlock('/unlock', handler)`         |
| `unsubscribe()` | Registra uma rota para o método UNSUBSCRIBE.                          | `router.unsubscribe('/unsubscribe', handler)`|

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
  res.send(`Usuário ${req.params.id} atualizado`);
});

// Registra rota DELETE
router.delete('/users/:id', (req, res) => {
  res.send(`Usuário ${req.params.id} deletado`);
});
```

Nesse exemplo, a classe ``Router`` é usada para definir rotas para manipular diferentes métodos HTTP, como GET, POST, PUT e DELETE. Cada rota aceita uma requisição (``req``) e uma resposta (``res``), permitindo que os manipuladores gerenciem as requisições recebidas e enviem respostas apropriadas.

Exemplo em Aplicação

```typescript
import cmmv from '@cmmv/server';

const app = cmmv();
const host = '0.0.0.0';
const port = 3000;

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

app.listen({ host, port });
```

## Estático

O middleware ``@cmmv/server-static`` é usado para servir arquivos estáticos a partir de um diretório ou diretórios especificados. Ele é projetado para lidar com requisições de conteúdo estático, como HTML, CSS, JavaScript, imagens ou quaisquer outros ativos. O middleware oferece várias opções de configuração para o comportamento de entrega de arquivos, incluindo cache, compressão, cabeçalhos personalizados e mais.

Uma vez inicializado, o middleware escuta requisições que correspondem ao prefixo especificado e serve arquivos a partir do diretório raiz (ou múltiplos diretórios) conforme configurado.

| Opção           | Descrição                                                                                                                                                                                   | Tipo                     | Padrão         |
|-----------------|-----------------------------------------------------------------------------------------------------------------------------|--------------------------|----------------|
| `root`          | Especifica o diretório raiz (ou diretórios) de onde os arquivos estáticos serão servidos.                                                                                                   | `string` ou `string[]`   | `undefined` (Obrigatório) |
| `prefix`        | Prefixo da URL para servir arquivos estáticos. Os arquivos serão servidos a partir de `prefix + caminho` (ex.: `/static/myfile.js`).                                                        | `string`                 | `'/'`          |
| `maxAge`        | Define a diretiva max-age do Cache-Control em milissegundos. Determina por quanto tempo os arquivos devem ser armazenados em cache pelo cliente.                                            | `number`                 | `0`            |
| `cacheControl`  | Habilita ou desabilita o cabeçalho Cache-Control. Quando habilitado, os cabeçalhos de cache serão definidos automaticamente com base na opção `maxAge`.                                     | `boolean`                | `true`         |
| `dotfiles`      | Define como lidar com dotfiles (arquivos começando com `.`). As opções são `'allow'`, `'deny'` ou `'ignore'`.                                                                               | `'allow'`, `'deny'`, `'ignore'` | `'allow'`     |
| `serverDotfiles`| Se habilitado, o servidor servirá dotfiles a partir do diretório raiz.                                                                                                                      | `boolean`                | `false`        |
| `index`         | Especifica o arquivo padrão a ser servido quando um diretório é requisitado. Pode ser uma string (ex.: `'index.html'`) ou `false` para desabilitar a entrega de arquivo de índice.          | `string` ou `boolean`    | `'index.html'` |
| `fallthrough`   | Quando `true`, permite que as requisições passem para o próximo middleware se um arquivo não for encontrado. Se `false`, um erro 404 será retornado.                                       | `boolean`                | `true`         |
| `redirect`      | Redireciona requisições para diretórios que não terminam com `/` adicionando uma barra final.                                                                                               | `boolean`                | `true`         |
| `immutable`     | Quando definido como `true`, serve arquivos com a diretiva `Cache-Control: immutable`, indicando que os arquivos nunca mudarão.                                                             | `boolean`                | `false`        |
| `lastModified`  | Habilita ou desabilita o cabeçalho Last-Modified. Quando habilitado, define o cabeçalho Last-Modified com base no horário da última modificação do arquivo.                                 | `boolean`                | `true`         |
| `etag`          | Habilita ou desabilita a geração de cabeçalhos ETag, que podem ser usados para validação de cache.                                                                                          | `boolean`                | `true`         |
| `extensions`    | Extensões de arquivo a tentar quando um arquivo não é encontrado (ex.: servir `file.html` quando a requisição é para `file`). Pode ser um array de strings.                                 | `string[]` ou `boolean`  | `false`        |
| `acceptRanges`  | Habilita o suporte para requisições de intervalo HTTP, útil para streaming de mídia.                                                                                                       | `boolean`                | `true`         |
| `preCompressed` | Se `true`, o middleware tentará servir arquivos pré-comprimidos (ex.: arquivos `.br` ou `.gz`) se disponíveis.                                                                              | `boolean`                | `false`        |
| `allowedPath`   | Uma função que pode ser usada para filtrar ou restringir acesso a caminhos específicos. Recebe `pathname`, `root` e `req` como parâmetros e retorna `true` para permitir ou `false` para rejeitar. | `Function`               | `undefined`    |
| `setHeaders`    | Uma função personalizada para definir cabeçalhos para a resposta. Recebe `res`, `path` e `stat` como parâmetros e pode modificar cabeçalhos antes do envio da resposta.                      | `Function`               | `null`         |

Este middleware não é compatível com o Express porque seu funcionamento interno difere significativamente do servidor de arquivos estáticos do Express. Uma mudança importante é a remoção do suporte a listagem de diretórios. Em vez disso, o middleware verifica arquivos existentes e cria rotas na inicialização da aplicação, resultando em maior eficiência.

Ele utiliza [``@fastify/send``](https://github.com/fastify/send) e [``@fastify/accept-negotiator``](https://github.com/fastify/accept-negotiator) para entregar arquivos de forma eficiente. Além disso, suporta complementos como ``@cmmv/etag`` e ``@cmmv/compression`` para entrega aprimorada de arquivos estáticos, oferecendo controle de cache por meio do uso de cabeçalhos ``lastModified`` e ``ETag``. Esses recursos proporcionam um servidor de arquivos estáticos mais otimizado e performático em comparação com abordagens tradicionais.

## JSON

O middleware ``@cmmv/body-parser`` no cmmv-server atua como uma função embutida projetada para lidar com requisições recebidas com payloads JSON. Semelhante ao Express, este middleware analisa o corpo da requisição e anexa os dados analisados ao objeto ``req.body``, tornando-os facilmente acessíveis para processamento adicional.

Este middleware analisa especificamente dados JSON e processa apenas requisições onde o cabeçalho ``Content-Type: application/json`` ou ``application/vnd.api+json`` corresponde à opção de tipo especificada. Ele suporta qualquer codificação Unicode e pode descomprimir automaticamente requisições comprimidas usando gzip ou deflate.

Após a execução do middleware, os dados JSON analisados ficam disponíveis em ``req.body``. Se não houver corpo para analisar, o ``Content-Type`` não corresponder ou ocorrer um erro, ``req.body`` será um objeto vazio (``{}``).

Como ``req.body`` é preenchido a partir de entrada fornecida pelo usuário, é importante notar que todas as propriedades e valores no objeto não são confiáveis. Eles devem ser cuidadosamente validados antes de serem usados. Por exemplo, tentar acessar ``req.body.foo.toString()`` pode resultar em erros se ``foo`` for indefinido, não for uma string ou se o método ``toString`` tiver sido sobrescrito ou substituído por entrada maliciosa.

A tabela a seguir descreve as opções disponíveis para configurar este middleware, oferecendo personalização sobre seu comportamento para atender a várias necessidades da aplicação.

| Propriedade | Descrição                                                                                                                                                                                                                                                                                                                                                                             | Tipo     | Padrão           |
|-------------|---------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|------------------|
| `inflate`   | Habilita ou desabilita o manuseio de corpos descomprimidos (comprimidos). Quando desabilitado, corpos descomprimidos são rejeitados.                                                                                                                                                                                                   | Boolean  | `true`           |
| `limit`     | Controla o tamanho máximo do corpo da requisição. Se for um número, o valor especifica o número de bytes; se for uma string, o valor é passado para a biblioteca `bytes` para análise.                                                                                                                                                           | Misto    | `"100kb"`        |
| `reviver`   | A opção `reviver` é passada diretamente para `JSON.parse` como segundo argumento. Permite transformação personalizada dos dados JSON analisados. Mais detalhes podem ser encontrados na [documentação do MDN sobre JSON.parse](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/JSON/parse#using_the_reviver_parameter). | Função   | `null`           |
| `strict`    | Habilita ou desabilita a aceitação apenas de arrays e objetos como JSON válido. Quando desabilitado, o middleware aceitará qualquer coisa que `JSON.parse` aceite.                                                                                                                                                                              | Boolean  | `true`           |
| `type`      | Determina o tipo de mídia que o middleware analisará. Pode ser uma string, array de strings ou uma função. Se não for uma função, o valor é passado diretamente para a biblioteca `type-is` e pode ser uma extensão (ex.: `json`), um tipo MIME (ex.: `application/json`) ou um tipo MIME curinga (ex.: `*/json`). Se for uma função, é chamada como `fn(req)` e analisa quando retorna `true`. | Misto    | `"application/json"` |
| `verify`    | Se fornecida, esta função é chamada como `verify(req, res, buf, encoding)`, onde `buf` é o corpo bruto da requisição. A análise pode ser abortada lançando um erro dentro desta função.                                                                                                                                                          | Função   | `undefined`      |

## Raw

Este middleware processa payloads de requisições recebidas e os converte em um Buffer, aproveitando funcionalidades inspiradas no ``body-parser``. Ele é projetado especificamente para lidar com corpos como Buffers e processa apenas requisições onde o cabeçalho ``Content-Type: application/octet-stream`` ou ``application/vnd+octets`` corresponde à opção de tipo definida.

O parser é capaz de lidar com qualquer codificação Unicode e suporta descompressão automática para requisições codificadas com gzip e deflate.

Após a execução do middleware, um Buffer contendo os dados analisados é atribuído à propriedade ``req.body``. Se não houver corpo, o ``Content-Type`` não corresponder ou ocorrer um erro, ``req.body`` será padrão como um objeto vazio (``{}``) ou permanecerá inalterado se outro parser já o tiver processado.

Como ``req.body`` é preenchido com base em entrada do usuário, é crucial validar quaisquer propriedades e valores antes de usá-los. Por exemplo, chamar ``req.body.toString()`` pode levar a erros se ``req.body`` tiver sido alterado por múltiplos parsers. Recomenda-se verificar que ``req.body`` é um Buffer antes de realizar operações específicas de buffer.

A tabela a seguir fornece uma visão geral das opções de configuração opcionais:

| Propriedade | Descrição                                                                                                                                                                                                                                                                                                              | Tipo     | Padrão                      |
|-------------|------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|-----------------------------|
| `inflate`   | Habilita ou desabilita o manuseio de corpos descomprimidos (comprimidos). Quando desabilitado, corpos descomprimidos são rejeitados.                                                                                                                                                                                           | Boolean  | `true`                      |
| `limit`     | Controla o tamanho máximo do corpo da requisição. Se for um número, especifica o número de bytes. Se for uma string, o valor é passado para a biblioteca `bytes` para análise.                                                                                                                                                 | Misto    | `"100kb"`                   |
| `type`      | Determina o tipo de mídia que o middleware analisará. Pode ser uma string, array de strings ou uma função. Se não for uma função, é passado para a biblioteca `type-is` e pode ser uma extensão (ex.: `bin`), um tipo MIME (ex.: `application/octet-stream`) ou um tipo MIME curinga (ex.: `application/*`). | Misto    | `"application/octet-stream"` |
| `verify`    | Uma função chamada como `verify(req, res, buf, encoding)` onde `buf` é o corpo bruto da requisição e `encoding` é sua codificação. Lançar um erro nesta função aborta o processo de análise.                                                                                                                                   | Função   | `undefined`                 |

## Texto

Este middleware processa payloads de requisições recebidas convertendo-os em uma string, utilizando funcionalidades do módulo ``body-parser``.

Ele fornece middleware que analisa todos os corpos como strings, processando apenas requisições onde o cabeçalho ``Content-Type: text/plain`` está alinhado com o tipo especificado. Ele lida com várias codificações Unicode e suporta descompressão automática de codificações gzip e deflate.

Após a execução do middleware, uma nova representação em string dos dados analisados é atribuída ao objeto ``req.body``. Se não houver corpo para analisar, o Content-Type não corresponder ou ocorrer um erro, ``req.body`` será padrão como um objeto vazio (``{}``).

Como a estrutura de ``req.body`` é baseada em entrada do usuário, é importante tratá-la como não confiável. Cada propriedade e valor deve ser validado antes do uso. Por exemplo, tentar chamar ``req.body.trim()`` pode levar a erros se ``req.body`` não for uma string. Portanto, é aconselhável garantir que ``req.body`` seja uma string antes de usar métodos de string.

A tabela a seguir descreve as opções de configuração disponíveis:

| Propriedade      | Descrição                                                                                                                                                                                                                                                   | Tipo     | Padrão        |
|------------------|-------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|---------------|
| `defaultCharset` | Especifica o conjunto de caracteres padrão para o conteúdo de texto quando o charset não está incluído no cabeçalho `Content-Type` da requisição.                                                                                                           | String   | `"utf-8"`     |
| `inflate`        | Controla se os corpos de requisição comprimidos (descomprimidos) devem ser tratados ou não. Se desabilitado, corpos comprimidos serão rejeitados.                                                                                                           | Boolean  | `true`        |
| `limit`          | Define o tamanho máximo permitido para o corpo da requisição. Quando fornecido como número, o valor é tratado como o número de bytes. Se fornecido como string, é analisado usando a biblioteca `bytes`.                                                     | Misto    | `"100kb"`     |
| `type`           | Define o tipo de mídia que o middleware deve analisar. Esta opção pode ser uma string, um array de strings ou uma função. Se não for uma função, o valor é passado para a biblioteca `type-is` e pode ser uma extensão (ex.: `txt`), um tipo MIME (ex.: `text/plain`) ou um padrão curinga (ex.: `text/*`). Se for uma função, é chamada como `fn(req)` e a requisição é analisada se retornar um valor verdadeiro. | Misto    | `"text/plain"` |
| `verify`         | Se fornecida, esta função é invocada como `verify(req, res, buf, encoding)`, onde `buf` é o corpo bruto da requisição em formato Buffer, e `encoding` é a codificação da requisição. A análise pode ser interrompida lançando um erro dentro desta função.       | Função   | `undefined`   |

## Urlencoded

Este middleware é projetado para analisar requisições recebidas que contêm payloads codificados em URL. Ele é baseado na funcionalidade fornecida pelo body-parser.

O middleware processa apenas corpos codificados em URL, procurando especificamente por requisições onde o cabeçalho ``Content-Type: application/x-www-form-urlencoded`` corresponde à opção de tipo especificada. Este parser aceita corpos codificados em UTF-8 e suporta descompressão automática para requisições comprimidas com gzip ou deflate.

Após o middleware processar uma requisição, ele preenche o objeto ``req.body`` com os dados analisados. Se não houver corpo, o Content-Type não corresponder ou ocorrer um erro, ``req.body`` será definido como um objeto vazio (``{}``). O objeto conterá pares chave-valor, onde os valores podem ser strings ou arrays (quando extended: false) ou qualquer tipo de dado (quando extended: true).

Como ``req.body`` é preenchido com base em entrada do usuário, é importante validar todas as propriedades e valores antes de usá-los. Por exemplo, chamar ``req.body.foo.toString()`` pode causar erros se `foo` não for uma string ou estiver indefinido. Portanto, é aconselhável verificar o tipo dos valores de ``req.body`` antes de realizar quaisquer operações sobre eles.

A tabela a seguir descreve as opções de configuração disponíveis para o middleware:

| Propriedade      | Descrição                                                                                                                                                                                                                                           | Tipo     | Padrão                            |
|------------------|-----------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|----------|-----------------------------------|
| `extended`       | Permite escolher entre analisar dados codificados em URL com a biblioteca `querystring` (quando `false`) ou a biblioteca `qs` (quando `true`). A sintaxe "extended" permite que objetos e arrays mais complexos sejam codificados no formato URL. | Boolean  | `true`                            |
| `inflate`        | Habilita ou desabilita o suporte para lidar com corpos de requisição comprimidos (descomprimidos). Se desabilitado, corpos comprimidos serão rejeitados.                                                                                             | Boolean  | `true`                            |
| `limit`          | Define o tamanho máximo permitido para o corpo da requisição. Se fornecido como número, especifica o número de bytes. Se fornecido como string, será analisado pela biblioteca `bytes`.                                                               | Misto    | `"100kb"`                         |
| `parameterLimit` | Controla o número máximo de parâmetros que podem estar presentes nos dados codificados em URL. Se a requisição exceder esse limite, um erro será lançado.                                                                                            | Number   | `1000`                            |
| `type`           | Define quais tipos de mídia o middleware processará. Pode ser especificado como uma string, array de strings ou uma função. Se não for uma função, esse valor é passado para a biblioteca `type-is` e pode ser uma extensão (ex.: `urlencoded`), um tipo MIME (ex.: `application/x-www-form-urlencoded`) ou um padrão curinga. | Misto    | `"application/x-www-form-urlencoded"` |
| `verify`         | Uma função que é invocada como `verify(req, res, buf, encoding)`, onde `buf` é o corpo bruto da requisição em formato Buffer. Lançar um erro nesta função interromperá o processo de análise.                                                        | Função   | `undefined`                       |
