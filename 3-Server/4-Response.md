# Resposta (Response)

O objeto `res` representa a resposta HTTP em uma aplicação no servidor e é responsável por gerenciar e enviar respostas de volta ao cliente. Este objeto permite definir códigos de status, cabeçalhos, cookies, entre outros. É uma versão aprimorada do objeto padrão `ServerResponse` do Node.js, oferecendo recursos adicionais para maior controle sobre a resposta. O objeto `res` é comumente usado para enviar vários tipos de respostas—como HTML, JSON ou arquivos—ao cliente. Por convenção, o objeto `res` refere-se à resposta (e `req` refere-se à requisição), mas seu nome pode ser diferente dependendo dos parâmetros usados na função de callback.

**Exemplo:**

```typescript
app.get('/user/:id', function (req, res) {
  res.send('usuário ' + req.params.id);
});
```

Você também pode nomear os parâmetros de forma diferente, como neste exemplo:

```typescript
app.get('/user/:id', function (request, response) {
  response.send('usuário ' + request.params.id);
});
```

O objeto `res` não apenas suporta todos os métodos e propriedades integrados do `ServerResponse` do Node, mas também oferece capacidades adicionais, facilitando o gerenciamento do ciclo de vida HTTP em aplicações modernas.

## Propriedades

## res.socket

Retorna o socket da requisição.

```typescript
console.log(res.socket);
```

## res.status

Recupera o código de status HTTP da resposta.

```typescript
console.log(res.status); // Exibe o código de status
```

## res.now

Retorna o timestamp atual em milissegundos.

```typescript
console.log(res.now); // Exibe o timestamp atual
```

## res.elapsedTime

Retorna o tempo decorrido em milissegundos desde que a resposta começou.

```typescript
console.log(res.elapsedTime); // Exibe o tempo decorrido
```

## res.sent

Verifica se a resposta já foi enviada.

```typescript
if (res.sent) {
  console.log('Resposta já enviada');
}
```

## res.headerSent

Retorna `true` se os cabeçalhos já foram enviados.

## res.writable

Retorna `true` se a resposta ainda pode ser gravada (não finalizada).

## res.message (get/set)

Recupera ou define a mensagem de status HTTP da resposta.

## res.body (get/set)

Recupera ou define o corpo da resposta. Pode aceitar diferentes tipos, como `string`, `Buffer`, `Object` ou `Stream`.

## res.length (get/set)

Recupera o tamanho do conteúdo da resposta ou define o cabeçalho `Content-Length`.

## res.status (get/set)

Define o código de status HTTP para a resposta.

```typescript
res.status = 404;
```

## res.lastModified (get/set)

Define o cabeçalho `Last-Modified` usando uma `string` ou um objeto `Date`.

```typescript
res.lastModified = new Date();
```

## Métodos

## res.has(field)

Verifica se um cabeçalho específico está presente na resposta.

```typescript
if (res.has('Content-Type')) {
  console.log('O cabeçalho Content-Type está definido');
}
```

## res.remove(field)

Remove um cabeçalho específico da resposta.

```typescript
res.remove('Content-Type');
```

## res.hijack()

Marca a resposta como sequestrada (`hijacked`).

## res.code(code)

Define o código de status HTTP e retorna a resposta para encadeamento (`chaining`).

```typescript
res.code(200);
```

## res.links(links)

Define o campo de cabeçalho `Link` com o objeto `links` fornecido.

```typescript
res.links({
  next: 'http://example.com/page2',
  last: 'http://example.com/page5'
});
```

## res.header(field, val)

Define um campo de cabeçalho com um valor específico.

```typescript
res.header('Content-Type', 'application/json');
```

## res.get(field)

Recupera o valor de um cabeçalho específico.

```typescript
console.log(res.get('Content-Type'));
```

## res.type(type)

Define o tipo de conteúdo (`Content-Type`) com base no tipo fornecido.

```typescript
res.type('json');
```

## res.sendStatus(statusCode)

Envia o código de status HTTP com a mensagem padrão.

```typescript
res.sendStatus(404);
```

## res.send(payload)

Envia a resposta com o payload fornecido.

```typescript
res.send({ message: 'Olá Mundo' });
```

## res.json(obj)

Envia uma resposta no formato JSON.

```typescript
res.json({ user: 'João' });
```

## res.jsonp(obj)

Envia uma resposta JSONP com suporte a callback.

## res.sendFile(path, opt, cb)

A função `res.sendFile()` é usada para transferir o arquivo localizado no caminho (`path`) especificado para o cliente. Ela configura automaticamente o cabeçalho `Content-Type` com base na extensão do arquivo e dispara um callback quando a transferência do arquivo é concluída ou ocorre um erro.

Veja algumas opções suportadas:

| Opção          | Tipo            | Padrão      | Descrição                                                                                                     |
|----------------|-----------------|-------------|---------------------------------------------------------------------------------------------------------------|
| `maxAge`      | número/string   | `0`        | Especifica o tempo de cache no cliente em milissegundos ou string.                                            |
| `root`        | string          | `undefined`| Define o diretório raiz para resolução de caminhos relativos.                                                 |
| `headers`     | objeto          | `{}`       | Cabeçalhos personalizados que serão adicionados na resposta.                                                  |
| `dotfiles`    | string          | `ignore`   | Determina como lidar com arquivos ocultos (ex.: `.gitignore`). Pode ser `allow`, `deny`, ou `ignore`. |
| `etag`        | boolean         | `true`     | Gera um cabeçalho `ETag` para o arquivo.                                                                     |
| `lastModified`| boolean         | `true`     | Configura o cabeçalho `Last-Modified` para o arquivo.                                                        |

## res.end(payload, encoding, cb)

Finaliza o processo de resposta.

## res.render(view, opt, cb)

Renderiza um template de visualização com as opções fornecidas e, opcionalmente, um callback.

```typescript
app.get('/sobre', (req, res) => {
    res.render('sobre', { titulo: 'Sobre Nós', empresa: 'MinhaEmpresa' });
});
```

Este método é ideal para aplicações que usam engines de template como Pug ou EJS.

## Compatibilidade de Resposta

## res.append(field, val)

Adiciona um valor a um cabeçalho específico.

```typescript
res.append('Link', ['<http://localhost/>', '<http://localhost:3000/>']);
```

## res.redirect(url, alt)

Realiza um redirecionamento (302) para a URL especificada.

```typescript
res.redirect('/login');
```

## res.clearCookie(name, options)

Remove o cookie com o nome especificado.

```typescript
res.clearCookie('session_id');
```

## res.cookie(name, value, options)

Define um cookie com nome e valor especificados, podendo incluir opções adicionais, como tempo de expiração, flags de segurança e mais.

```typescript
res.cookie('lembrar', '1', { 
    maxAge: 900000, 
    httpOnly: true 
});
```

Essas propriedades e métodos tornam o `res` um objeto poderoso para gerenciar respostas HTTP em aplicações modernas.

| Opção       | Tipo            | Padrão      | Descrição                                                                                                     |
|-------------|-----------------|-------------|---------------------------------------------------------------------------------------------------------------|
| `maxAge`   | número          | `undefined` | Especifica o `max-age` do cookie em milissegundos. Se definido, substitui a opção `expires`.                 |
| `expires`  | Date             | `undefined` | Define a data de expiração do cookie. Use `maxAge` como alternativa para definir o tempo de expiração relativo. |
| `path`     | string           | `'/'`       | Define o caminho URL no qual o cookie é válido. O padrão é o caminho raiz `/`.                                |
| `domain`   | string           | `undefined` | Especifica o domínio no qual o cookie é válido.                                                                |
| `secure`   | boolean          | `false`     | Marca o cookie como seguro, garantindo que seja enviado apenas via HTTPS.                                       |
| `httpOnly` | boolean          | `false`     | Marca o cookie como HTTP-only, o que significa que ele não pode ser acessado via JavaScript no navegador.        |
| `signed`   | boolean          | `false`     | Indica se o cookie deve ser assinado. Requer um segredo definido no `cookieParser()`.                          |
| `sameSite` | boolean/string   | `false`     | Restringe como o cookie é enviado entre sites. Pode ser `'strict'`, `'lax'` ou `true` (o que define como `'strict'`). |

## res.render(view, opt, cb)

Renderiza um modelo de visualização com as opções fornecidas e uma função de callback opcional. Se um callback for fornecido, ele será executado assim que a visualização for renderizada, e nenhuma resposta automática será enviada. Se nenhum callback for fornecido, o método envia uma resposta padrão com o status `200` e o tipo de conteúdo `text/html`.

**Exemplo:**

```typescript
app.get('/sobre', (req, res) => {
    res.render('sobre', { title: 'Sobre Nós', company: 'MinhaEmpresa' });
});
```

**Exemplo com Callback:**

```typescript
app.get('/visualizacao-personalizada', (req, res) => {
    res.render('personalizado', { title: 'Página Personalizada' }, (err, html) => {
        if (err) 
            return res.status(500).send('Erro ao renderizar a página.');
        
        res.send(html);
    });
});
```

Neste exemplo, uma função de callback personalizada é fornecida. Se ocorrer um erro durante a renderização, o callback lida com ele enviando um status `500` e uma mensagem de erro ao cliente.

## Compatibilidade de Resposta

## res.trailer(key, fn)

Define um cabeçalho de trailer com a chave e função fornecidas.

```typescript
res.trailer('X-Custom-Trailer', () => 'Valor do Trailer');
```

### res.flushHeaders()

Envia imediatamente os cabeçalhos acumulados para o cliente, sem esperar que o corpo da resposta seja enviado.

```typescript
res.flushHeaders();
```

### res.getHeaderNames()

Retorna um array com os nomes dos cabeçalhos atualmente definidos na resposta.

```typescript
console.log(res.getHeaderNames());
```

### res.getHeaders()

Retorna um objeto contendo todos os cabeçalhos definidos na resposta.

```typescript
console.log(res.getHeaders());
```

### res.hasHeader(name)

Verifica se um cabeçalho específico está presente na resposta.

```typescript
if (res.hasHeader('Content-Type')) {
    console.log('O cabeçalho Content-Type está definido');
}
```

### res.removeHeader(name)

Remove um cabeçalho específico da resposta.

```typescript
res.removeHeader('Content-Type');
```

### res.setTimeout(msecs, cb)

Define um tempo limite para a resposta. O callback `cb` será executado se o tempo limite for alcançado.

```typescript
res.setTimeout(5000, () => {
    console.log('A resposta atingiu o tempo limite');
});
```

### res.uncork()

Desbloqueia o fluxo de escrita na resposta, enviando qualquer conteúdo armazenado em buffer.

### res.writeEarlyHints(hints, callback)

Escreve cabeçalhos HTTP/1.1 103 Early Hints, que podem ser usados para sugerir ao navegador que carregue recursos antes de receber a resposta final.

**Exemplo:**

```typescript
res.writeEarlyHints({
    'Link': '</style.css>; rel=preload; as=style'
}, () => {
    console.log('Hints enviados ao cliente');
});
```

### res.trailer(key, fn)

Define um cabeçalho de trailer com a chave e função fornecidas.

```typescript
res.trailer('X-Custom-Trailer', () => 'Valor do Trailer');
```

Esses métodos adicionais fornecem maior flexibilidade no controle de respostas HTTP, permitindo otimizações e personalizações no comportamento do servidor. O objeto `res` é projetado para atender a uma ampla gama de casos de uso em aplicações modernas, garantindo compatibilidade com os padrões HTTP e melhores práticas.
