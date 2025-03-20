# Aplicação

A classe ``Application`` é o componente central que gerencia o ciclo de vida do servidor, rotas e middlewares. Ela atua como o ponto de entrada para configurar rotas, aplicar middlewares e lidar com requisições e respostas HTTP. Essa classe foi projetada para ser flexível, permitindo que os desenvolvedores definam diversos mecanismos de manipulação de rotas e funções de middleware que ampliam as capacidades do servidor.

A classe ``Application`` neste sistema é fortemente inspirada em frameworks como o Express, mas foi projetada para melhorar o desempenho e adicionar suporte a recursos HTTP modernos, como o HTTP2. Ela também integra middlewares adicionais e otimizações, como mecanismos de compressão e cache.

## app.addRoute()

O método ``addRoute`` permite que a aplicação registre rotas específicas com seus respectivos manipuladores. Esse método recebe um objeto de configuração como parâmetro, que inclui o método HTTP, o caminho da rota e a função manipuladora. O método ``addRoute`` assegura que cada rota seja mapeada corretamente no roteador interno e possa lidar com requisições adequadamente.

Exemplo:

```typescript
app.addRoute({
  method: 'GET',
  url: '/users',
  handler: (req, res) => {
    res.json({ users: ['João', 'Joana'] });
  }
});
```

Esse exemplo registra uma rota GET na URL ``/users`` com um manipulador que responde com um objeto JSON.

## app.use()

O método ``use`` é usado para aplicar funções de middleware à aplicação. As funções de middleware são executadas na ordem em que são registradas, permitindo diversas operações como validação de requisições, logging e modificações de resposta.

O ``use`` também pode lidar com subaplicações ou roteadores, tornando-o uma ferramenta poderosa para estruturar grandes aplicações em componentes menores.

Exemplo:

```typescript
app.use((req, res, next) => {
  console.log(`${req.method} ${req.url}`);
  next(); // Passa o controle para o próximo middleware/rota
});
```

Nesse exemplo, cada requisição recebida é registrada no log antes de passar o controle para o próximo middleware ou manipulador de rota.

## app.render()

O método ``render`` é responsável por renderizar visões na aplicação. Ele permite que templates HTML dinâmicos sejam renderizados com dados fornecidos pela aplicação. Esse método geralmente funciona com motores de template como o Pug, que são registrados via método ``app.set()``.

Exemplo:

```typescript
app.render('email', function (err, html) {
  // ...
})

app.render('email', { name: 'Tobi' }, function (err, html) {
  // ...
})
```

## app.route()

O método ``route`` fornece um mecanismo para definir um manipulador de rota para um método HTTP e caminho específicos. Diferente do ``addRoute``, que registra diretamente o manipulador, o ``route`` retorna um objeto onde você pode encadear métodos para configurar a rota ainda mais.

Exemplo:

```typescript
const userRoute = app.route('GET', '/user/:id');

userRoute.stack.push((req, res) => {
  res.send(`ID do Usuário: ${req.params.id}`);
});
```

Esse exemplo cria uma rota ``GET`` para ``/user/:id`` e registra um manipulador que responde com o ID do usuário fornecido na URL. O método ``route`` é especialmente útil quando você precisa de mais controle sobre como a rota é manipulada, oferecendo maior flexibilidade.

## app.engine()

O método ``engine`` é usado para registrar um motor de template com a aplicação para renderizar visões dinâmicas. Motores de template permitem que arquivos HTML sejam preenchidos dinamicamente com dados antes de serem servidos ao cliente.

**Parâmetros:**

* ``ext:`` A extensão de arquivo para a qual o motor deve ser registrado. Pode ser uma string representando a extensão (ex.: ``'ejs'`` ou ``'pug'``).
* ``fn:`` A função de callback responsável por renderizar o template. Essa função geralmente aceita o caminho do arquivo, opções e um callback para renderizar a visão.

**Exemplo:**

```typescript
app.engine('pug', require('pug').__express);
app.set('view engine', 'pug');
```

## app.set()

O método ``set`` é usado para configurar as definições da aplicação. Ele pode ser usado para atribuir um valor a uma configuração ou recuperar o valor atual de uma configuração.

**Parâmetros:**

* **setting:** O nome da configuração (ex.: ``'view engine'``, ``'env'``, ``'etag'``).
* **val:** O valor a ser atribuído à configuração.

**Exemplo:**

```typescript
app.set('env', 'produção');
app.set('view engine', 'pug');
```

Nesse exemplo, o ambiente é definido como ``'produção'`` e o motor de visão é configurado como ``'pug'``.

**Configurações Especiais:**

* ``'etag':`` Configura o mecanismo de geração de ETag para cache.
* ``'query parser':`` Define como a string de consulta deve ser analisada.
* ``'trust proxy':`` Determina se a aplicação deve confiar em proxies reversos.

## app.param()

O método ``param`` é usado para definir o pré-processamento de parâmetros de rota. Isso é útil para middlewares que executam ações em parâmetros de rota específicos antes que a requisição seja processada.

**Parâmetros:**

* ``name:`` O nome do parâmetro da rota ou um array de nomes de parâmetros.
* ``fn:`` Uma função de callback para lidar com o parâmetro da rota. O callback recebe ``(req, res, next, value)``.

**Exemplo:**

```typescript
app.param('id', (req, res, next, id) => {
  console.log(`ID Recebido: ${id}`);
  next();
});
```

Esse exemplo registra no log o parâmetro ``id`` sempre que uma rota que inclui ``:id`` é acessada.

## app.path()

O método ``path`` retorna o caminho absoluto da aplicação. Se a aplicação estiver montada em um caminho específico (ex.: ``/admin``), ele retornará o caminho completo de montagem, incluindo quaisquer montagens de aplicações pai.

**Exemplo:**

```typescript
const admin = express();
app.use('/admin', admin);
console.log(admin.path());  // Saída: '/admin'
```

## app.enabled()

O método ``enabled`` verifica se uma configuração específica está habilitada (ou seja, verdadeira). Retorna ``true`` se a configuração estiver habilitada e ``false`` se não estiver.

**Exemplo:**

```typescript
app.set('view cache', true);
console.log(app.enabled('view cache'));  // Saída: true
```

## app.disabled()

O método ``disabled`` verifica se uma configuração específica está desabilitada (ou seja, falsa). Retorna ``true`` se a configuração estiver desabilitada e ``false`` se não estiver.

**Exemplo:**

```typescript
app.set('view cache', false);
console.log(app.disabled('view cache'));  // Saída: true
```

## app.enable()

O método ``enable`` habilita uma configuração específica, tornando-a verdadeira.

**Exemplo:**

```typescript
app.enable('view cache');
console.log(app.enabled('view cache'));  // Saída: true
```

## app.disable()

O método ``disable`` desabilita uma configuração específica, tornando-a falsa.

**Exemplo:**

```typescript
app.disable('view cache');
console.log(app.disabled('view cache'));  // Saída: true
```

## app.setErrorHandler()

O método ``setErrorHandler`` define um manipulador de erros global para a aplicação. Esse manipulador será invocado sempre que ocorrer um erro durante o processamento de uma requisição. O manipulador de erros padrão será substituído pelo fornecido neste método.

**Exemplo:**

```typescript
app.setErrorHandler((error, req, res) => {
  console.error('Erro ocorrido:', error);
  res.status(500).send({ error: 'Erro Interno do Servidor' });
});
```

## app.addHook()

O método ``addHook`` permite que você anexe hooks a eventos específicos do ciclo de vida durante o ciclo de requisição-resposta. Você pode adicionar hooks para eventos como ``onSend``, ``preSerialization``, ``onError`` e mais.

**Exemplo:**

```typescript
app.addHook('onSend', async (request, response, payload) => {
  console.log('Resposta sendo enviada para a URL:', request.url);
});
```

Esse exemplo adiciona um hook para o evento ``onSend`` que registra uma mensagem logo antes da resposta ser enviada ao cliente.

## app.addContentTypeParser()

O método ``addContentTypeParser`` registra um parser de tipo de conteúdo personalizado para lidar com diferentes tipos de corpos de requisição. Ele permite que a aplicação analise tipos de conteúdo específicos como XML, YAML, etc.

**Exemplo:**

```typescript
app.addContentTypeParser('application/xml', (req, res, body, done) => {
  const parsed = parseXml(body); // Lógica personalizada de análise de XML
  done(null, parsed);
});
```

Esse exemplo registra um parser personalizado para o tipo de conteúdo ``application/xml`` que converte XML em um objeto JavaScript utilizável.

## app.contentTypeParser()

O método ``contentTypeParser`` é usado internamente para processar corpos de requisições recebidas com base nos parsers de tipo de conteúdo registrados. Ele encontra o parser para o tipo de conteúdo fornecido e o aplica ao corpo da requisição.

**Exemplo:**

```typescript
app.contentTypeParser('application/json', (req, res, next) => {
  console.log(req.body); // Processa o corpo JSON analisado
  next();
}, req, res);
```

Esse exemplo processa requisições com tipo de conteúdo ``application/json`` e registra o corpo analisado no log.

## app.METHOD

As funções ``app.METHOD()`` no framework permitem que você defina rotas com base em métodos HTTP específicos, como ``GET``, ``PUT``, ``POST`` e ``DELETE``. Cada método HTTP corresponde a uma chamada de método no objeto da aplicação, por exemplo, ``app.get()``, ``app.post()``, ``app.put()``, etc. Esses métodos lidam com requisições recebidas, roteando-as para o callback apropriado com base no método HTTP e na URL.

Embora a documentação foque principalmente nos métodos mais comuns (``GET``, ``POST``, ``PUT``, ``DELETE``), outros métodos HTTP como ``PATCH``, ``OPTIONS`` e ``HEAD`` são suportados da mesma forma. Você pode definir rotas usando qualquer método HTTP padrão utilizando a função correspondente no objeto da aplicação.

Para métodos HTTP que não são nomes válidos de variáveis JavaScript (ex.: ``M-SEARCH``), você pode usar a notação de colchetes para definir a rota. Por exemplo:

```typescript
app['m-search']('/', function(req, res) {
    res.send('Requisição M-SEARCH');
});
```

Além disso, ao definir uma rota com ``app.get()``, o framework automaticamente lida com requisições ``HEAD`` para o mesmo caminho, a menos que você defina explicitamente ``app.head()`` para aquele caminho.

A função ``app.all()`` é especial, pois roteia requisições de todos os métodos HTTP para um caminho específico. Isso não deriva de um único método HTTP, tornando-a útil para middlewares ou para lidar com qualquer tipo de requisição em uma rota específica.

Para informações mais detalhadas sobre roteamento e os diversos métodos HTTP suportados, consulte o guia de roteamento na documentação.

<table style="border: 0px; background: none">
<tbody><tr>
<td style="background: none; border: 0px;">
<ul>
<li><code>checkout</code></li>
<li><code>copy</code></li>
<li><code>delete</code></li>
<li><code>get</code></li>
<li><code>head</code></li>
<li><code>lock</code></li>
<li><code>merge</code></li>
<li><code>mkactivity</code></li>
</ul>
</td>
<td style="background: none; border: 0px;">
<ul>
<li><code>mkcol</code></li>
<li><code>move</code></li>
<li><code>m-search</code></li>
<li><code>notify</code></li>
<li><code>options</code></li>
<li><code>patch</code></li>
<li><code>post</code></li>
</ul>
</td>
<td style="background: none; border: 0px;">
<ul>
<li><code>purge</code></li>
<li><code>put</code></li>
<li><code>report</code></li>
<li><code>search</code></li>
<li><code>subscribe</code></li>
<li><code>trace</code></li>
<li><code>unlock</code></li>
<li><code>unsubscribe</code></li>
</ul>
</td>
</tr>
</tbody></table>

## app.listen

O método ``listen()`` inicia o servidor no host e porta especificados, retornando uma Promessa que é resolvida quando o servidor começa a ouvir conexões recebidas com sucesso ou rejeitada se ocorrer um erro.

**Exemplo:**

```typescript
const listenOptions = { host: '0.0.0.0', port: 3000 };

app.listen(listenOptions)
.then(server => {
    const attr = server.address();
    console.log(`Servidor está ouvindo em ${attr.address}:${attr.port}`);
})
.catch(err => {
    console.error('Falha ao iniciar o servidor:', err);
});
```
