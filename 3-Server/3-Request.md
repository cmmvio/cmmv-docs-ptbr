# Requisição (Request)

O objeto `req` representa a requisição HTTP recebida e fornece acesso a várias propriedades e métodos, como a string de consulta (query string), parâmetros, corpo da requisição, cabeçalhos HTTP, entre outros. Nesta documentação, assim como em convenções comuns, este objeto é referido como `req` (enquanto o objeto de resposta HTTP é `res`). No entanto, os nomes reais usados para esses objetos podem ser definidos pelos parâmetros na função callback em que você está trabalhando.

**Exemplo:**
```typescript
app.get('/user/:id', function (req, res) {
  res.send('Usuário: ' + req.params.id);
});
```

Alternativamente, você pode nomear os parâmetros de forma diferente, como neste exemplo:

```typescript
app.get('/user/:id', function (request, response) {
  response.send('Usuário: ' + request.params.id);
});
```

O objeto `req` estende o objeto `http.IncomingMessage` do Node.js, o que significa que ele herda todas as propriedades e métodos integrados do objeto de requisição do Node. Além disso, o objeto `req` inclui várias funcionalidades adicionais específicas para o manuseio de requisições HTTP em sua aplicação, tornando-o mais poderoso e fácil de usar.

## Propriedades

## req.query

Retorna a string de consulta analisada como um objeto. A string de consulta é automaticamente analisada usando o parser de consulta (por exemplo, a biblioteca `qs`).

```typescript
// URL: /search?name=John&age=30
console.log(req.query);  // { name: 'John', age: '30' }
```

## req.querystring

Retorna a string de consulta bruta da URL.

```typescript
// URL: /search?name=John&age=30
console.log(req.querystring);  // "name=John&age=30"
```

## req.search

Retorna a string de busca, que é a string de consulta prefixada com um ponto de interrogação (`?`).

```typescript
// URL: /search?name=John&age=30
console.log(req.search);  // "?name=John&age=30"
```

## req.socket

Retorna o objeto de socket da requisição, que representa a conexão subjacente.

## req.protocol

Retorna o protocolo usado na requisição (`http` ou `https`). Se a aplicação estiver atrás de um proxy, também verificará o cabeçalho `X-Forwarded-Proto` se a configuração `trust proxy` estiver habilitada.

```typescript
console.log(req.protocol);  // "https"
```

## req.headers

Retorna os cabeçalhos da requisição como um objeto.

```typescript
console.log(req.headers['content-type']);  // "application/json"
```

## req.url

Retorna a URL da requisição.

```typescript
console.log(req.url);  // "/search?name=John&age=30"
```

## req.origin

Retorna a origem da requisição, incluindo o protocolo e o host.

```typescript
console.log(req.origin);  // "https://example.com"
```

## req.href

Retorna a URL completa da requisição, incluindo a origem e a URL original.

```typescript
console.log(req.href);  // "https://example.com/search?name=John&age=30"
```

## req.secure

Retorna `true` se a conexão for segura (HTTPS).

## req.method

Retorna o método HTTP da requisição (por exemplo, `GET`, `POST`, etc.).

```typescript
console.log(req.method);  // "GET"
```

## req.path

Retorna o caminho da URL da requisição sem a string de consulta.

```typescript
// URL: /search?name=John
console.log(req.path);  // "/search"
```

## req.host

Retorna o nome do host da requisição, considerando o cabeçalho `X-Forwarded-Host` se a configuração `trust proxy` estiver habilitada.

```typescript
console.log(req.host);  // "example.com"
```

## req.hostname

Retorna o nome do host do cabeçalho `Host`, excluindo o número da porta.

```typescript
console.log(req.hostname);  // "example.com"
```

## req.URL

Retorna um objeto URL analisado da API WHATWG URL.

## req.fresh

Retorna `true` se a requisição for considerada "nova", ou seja, a resposta não mudou desde a última requisição (com base nos cabeçalhos `Last-Modified` e/ou `ETag`).

## req.stale

Retorna `true` se a requisição for considerada "antiga", ou seja, o recurso mudou desde a última requisição.

## req.idempotent

Retorna `true` se o método da requisição for idempotente (`GET`, `HEAD`, `PUT`, `DELETE`, etc.).

## req.ip

Retorna o endereço IP remoto da requisição, considerando a configuração `trust proxy`.

## req.ips

Retorna um array de endereços IP, com o IP do cliente em primeiro lugar, seguido pelos endereços de proxy (quando `trust proxy` está habilitado).

## req.length

Retorna o valor do cabeçalho `Content-Length` como um número, se presente.

## req.subdomains

Retorna um array de subdomínios do hostname da requisição. Os subdomínios são determinados pela configuração `subdomain offset`.

## req.xhr

Retorna `true` se a requisição foi feita usando um XMLHttpRequest (ou seja, uma requisição Ajax).

## req.cookies

Retorna os cookies enviados pelo cliente.

## req.signedCookies

Retorna os cookies assinados enviados pelo cliente.

## Métodos

## req.header()

Alias para `req.get()`. Retorna o valor do campo de cabeçalho da requisição especificado.

```typescript
req.header('Content-Type');  // "application/json"
```

## req.get()

Retorna o valor do campo de cabeçalho da requisição especificado.

```typescript
req.get('Content-Type');  // "application/json"
```

## req.accepts()

Verifica se os tipos MIME especificados são aceitáveis com base no cabeçalho `Accept`. Retorna a melhor correspondência ou `undefined`.

```typescript
req.accepts('html');  // "html"
```

## req.acceptsEncodings()

Verifica se as codificações especificadas são aceitáveis com base no cabeçalho `Accept-Encoding`.

```typescript
req.acceptsEncodings('gzip');  // "gzip"
```

## req.acceptsCharsets()

Verifica se os conjuntos de caracteres especificados são aceitáveis com base no cabeçalho `Accept-Charset`.

```typescript
req.acceptsCharsets('utf-8');  // "utf-8"
```

## req.acceptsLanguages()

Verifica se os idiomas especificados são aceitáveis com base no cabeçalho `Accept-Language`.

```typescript
req.acceptsLanguages('en');  // "en"
```

## req.range()

Analisa o cabeçalho Range e retorna um array de intervalos, ou `-1` se não for satisfatório, ou `-2` se for inválido.

```typescript
req.range(1000);  // [{ start: 0, end: 999 }]
```

## req\.is()

Verifica se o `Content-Type` da requisição corresponde ao tipo MIME especificado.

```typescript
req.is('json');  // "json"
```