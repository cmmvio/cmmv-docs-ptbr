# Resposta

O objeto ``res`` representa a resposta HTTP em uma aplicação do lado do servidor e é responsável por gerenciar e enviar respostas de volta ao cliente. Este objeto permite definir códigos de status, cabeçalhos, cookies e mais. É uma versão aprimorada do objeto padrão do Node.js ``ServerResponse``, oferecendo recursos adicionais para maior controle sobre a resposta. O objeto ``res`` é comumente usado para enviar vários tipos de respostas — como HTML, JSON ou arquivos — ao cliente. Por convenção, o objeto ``res`` refere-se à resposta (e ``req`` à requisição), mas seu nome pode variar dependendo dos parâmetros usados na função de callback.

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

O objeto ``res`` suporta todos os métodos e propriedades embutidos do ``ServerResponse`` do Node.js, mas também oferece capacidades adicionais, facilitando o gerenciamento do ciclo de vida HTTP em aplicações modernas.

## Propriedades

### res.socket

Retorna o socket da requisição.

```typescript
console.log(res.socket);
```

### res.status

Recupera o código de status HTTP da resposta.

```typescript
console.log(res.status); // Exibe o código de status
```

### res.now

Retorna o timestamp atual em milissegundos.

```typescript
console.log(res.now); // Exibe o timestamp atual
```

### res.elapsedTime

Retorna o tempo decorrido em milissegundos desde o início da resposta.

```typescript
console.log(res.elapsedTime); // Exibe o tempo decorrido
```

### res.sent

Verifica se a resposta já foi enviada.

```typescript
if (res.sent) {
  console.log('Resposta já enviada');
}
```

### res.headerSent

Retorna ``true`` se os cabeçalhos já foram enviados.

### res.writable

Retorna ``true`` se a resposta ainda é gravável (não finalizada).

### res.message (get/set)

Recupera a mensagem de status HTTP da resposta.

### res.body (get/set)

Retorna ou define o corpo da resposta. Pode aceitar diferentes tipos, como string, Buffer, Object ou Stream.

### res.length (get/set)

Retorna o comprimento do conteúdo da resposta. / Define o cabeçalho ``Content-Length`` como ``n``.

### res.status (get/set)

Define o código de status HTTP para a resposta.

```typescript
res.status = 404;
```

### res.lastModified (get/set)

Define o cabeçalho ``Last-Modified`` usando uma string ou uma ``Date``.

```typescript
res.lastModified = new Date();
```

## Métodos

### res.has(field)

Verifica se um cabeçalho específico está presente na resposta.

```typescript
if (res.has('Content-Type')) {
  console.log('Cabeçalho Content-Type está definido');
}
```

### res.remove(field)

Remove um cabeçalho específico da resposta.

```typescript
res.remove('Content-Type');
```

### res.hijack()

Marca a resposta como sequestrada.

### res.code(code)

Define o código de status HTTP e retorna a resposta para encadeamento.

```typescript
res.code(200);
```

### res.links(links)

Define o campo de cabeçalho ``Link`` com o objeto ``links`` fornecido.

```typescript
res.links({
  next: 'http://example.com/page2',
  last: 'http://example.com/page5'
});
```

### res.header(field, val)

Define um campo de cabeçalho com um valor específico.

```typescript
res.header('Content-Type', 'application/json');
```

### res.get(field)

Recupera o valor de um campo de cabeçalho específico.

```typescript
console.log(res.get('Content-Type'));
```

### res.type(type)

Define o ``Content-Type`` com base no tipo fornecido.

```typescript
res.type('json');
```

### res.sendStatus(statusCode)

Envia o código de status HTTP com a mensagem padrão.

```typescript
res.sendStatus(404);
```

### res.send(payload)

Envia a resposta com o payload fornecido.

```typescript
res.send({ mensagem: 'Olá, Mundo' });
```

### res.json(obj)

Envia uma resposta JSON.

```typescript
res.json({ usuário: 'João' });
```

### res.jsonp(obj)

Envia uma resposta JSONP com suporte a callback.

### res.sendFile(path, opt, cb)

A função ``res.sendFile()`` é usada para transferir o arquivo localizado em um determinado ``path`` para o cliente. Ela define automaticamente o cabeçalho de resposta ``Content-Type`` com base na extensão do arquivo e aciona um callback quando a transferência do arquivo é concluída ou se ocorre um erro. Esse método é particularmente útil para servir arquivos estáticos ou entregar arquivos dinamicamente em resposta a requisições.

Aqui está uma visão geral de como o ``res.sendFile()`` funciona, junto com opções adicionais que podem ser usadas para controlar o comportamento de entrega de arquivos.

| Opção          | Tipo           | Padrão      | Descrição                                                                                                                                    |
|----------------|----------------|-------------|----------------------------------------------------------------------------------------------------------------------------------------------|
| `maxAge`       | number/string  | `0`         | Especifica a propriedade `max-age` do `Cache-Control` em milissegundos ou uma string (convertida por `ms`). Controla o tempo de cache do cliente. |
| `root`         | string         | `undefined` | O diretório raiz a partir do qual os caminhos relativos dos arquivos são resolvidos. Sem `root`, o `path` deve ser um caminho absoluto.      |
| `headers`      | object         | `{}`        | Objeto contendo cabeçalhos personalizados que serão adicionados à resposta quando o arquivo for servido.                                     |
| `dotfiles`     | string         | `'ignore'`  | Determina como lidar com dotfiles (arquivos ou diretórios começando com ponto, como `.gitignore`). Pode ser `'allow'`, `'deny'` ou `'ignore'`. |
| `etag`         | boolean        | `true`      | Define se deve gerar um cabeçalho `ETag` para o arquivo.                                                                                     |
| `lastModified` | boolean        | `true`      | Define se deve configurar o cabeçalho `Last-Modified` para o arquivo.                                                                        |
| `cacheControl` | boolean        | `true`      | Habilita ou desabilita a configuração do cabeçalho `Cache-Control` na resposta.                                                              |
| `acceptRanges` | boolean        | `true`      | Permite conteúdo parcial (via cabeçalho `Range`). Útil para streaming de vídeo e áudio.                                                      |
| `immutable`    | boolean        | `false`     | Quando definido como `true`, adiciona `Cache-Control: immutable` à resposta, indicando que o arquivo nunca mudará e pode ser armazenado indefinidamente. |

### res.download(path, filename, opt, cb)

O método ``res.download()`` é usado para transferir o arquivo localizado no ``path`` especificado como um anexo, solicitando ao cliente que faça o download do arquivo. Você pode opcionalmente fornecer um nome de arquivo diferente para o arquivo baixado e uma função de callback para lidar com quaisquer erros que possam ocorrer durante a transferência.

Esse método também aceita um objeto ``options`` semelhante ao usado com ``res.sendFile()``. O cabeçalho ``Content-Disposition`` é automaticamente definido para sinalizar que o arquivo deve ser baixado como um anexo, sobrescrevendo quaisquer cabeçalhos ``Content-Disposition`` previamente definidos.

Internamente, esse método usa ``res.sendFile()`` para gerenciar a transferência do arquivo.

### res.end(payload, encoding, cb)

Finaliza o processo de resposta.

### res.format(obj)

Responde à requisição chamando o callback apropriado do objeto ``obj`` com base no cabeçalho ``Accept``.

```typescript
res.format({
  'text/plain': () => res.send('Texto puro'),
  'application/json': () => res.json({ mensagem: 'JSON' })
});
```

### res.vary(field)

Adiciona o campo fornecido ao cabeçalho ``Vary``.

```typescript
res.vary('Accept-Encoding');
```

### res.attachment(filename)

Define o cabeçalho ``Content-Disposition`` como anexo e define o nome do arquivo.

### res.append(field, val)

Adiciona um valor a um cabeçalho específico.

```typescript
res.append('Link', ['<http://localhost/>', '<http://localhost:3000/>']);
res.append('Set-Cookie', 'foo=bar; Path=/; HttpOnly');
res.append('Warning', '199 Miscellaneous warning');
```

### res.location(url)

Define o cabeçalho ``Location`` para a URL fornecida.

```typescript
res.location('/foo/bar');
res.location('http://example.com');
res.location('../login');
```

### res.redirect(url, alt)

Realiza um redirecionamento 302 para a URL especificada.

```typescript
this.redirect('back');
this.redirect('back', '/index.html');
this.redirect('/login');
this.redirect('http://google.com');
```

### res.clearCookie(name, options)

Limpa o cookie com o nome fornecido.

```typescript
res.clearCookie('session_id');
```

### res.cookie(name, value, options)

O método ``res.cookie()`` é usado para definir um cookie com um nome e valor especificados. Ele permite definir opções adicionais para controlar o comportamento do cookie, como tempo de expiração, bandeiras de segurança e mais. O método é encadeável, permitindo chamá-lo várias vezes em uma única resposta.

```typescript
// Define um cookie que expira em 15 minutos
res.cookie('rememberme', '1', {
    expires: new Date(Date.now() + 900000),
    httpOnly: true
});

// Define um cookie com max-age de 15 minutos (em milissegundos)
res.cookie('rememberme', '1', {
    maxAge: 900000,
    httpOnly: true
});
```

| Opção      | Tipo            | Padrão      | Descrição                                                                                                    |
|------------|-----------------|-------------|--------------------------------------------------------------------------------------------------------------|
| `maxAge`   | number          | `undefined` | Especifica o `max-age` do cookie em milissegundos. Se definido, substitui a opção `expires`.                  |
| `expires`  | Date            | `undefined` | Define a data de expiração do cookie. Use `maxAge` como alternativa para definir o tempo de expiração relativo. |
| `path`     | string          | `'/'`       | Define o caminho da URL para o qual o cookie é válido. Padrão é o caminho raiz `/`.                          |
| `domain`   | string          | `undefined` | Especifica o domínio para o qual o cookie é válido.                                                          |
| `secure`   | boolean         | `false`     | Marca o cookie como seguro, garantindo que seja enviado apenas por HTTPS.                                    |
| `httpOnly` | boolean         | `false`     | Marca o cookie como HTTP-only, significando que ele não pode ser acessado via JavaScript no navegador.         |
| `signed`   | boolean         | `false`     | Indica se o cookie deve ser assinado. Requer um segredo definido em `cookieParser()`.                         |
| `sameSite` | boolean/string  | `false`     | Restringe como o cookie é enviado entre sites. Pode ser `'strict'`, `'lax'` ou `true` (que define como `'strict'`). |

### res.render(view, opt, cb)

Renderiza um template de visão com as opções fornecidas e uma função de callback opcional. Se um callback for fornecido, ele é executado assim que a visão for renderizada, e nenhuma resposta automática é enviada. Se nenhum callback for fornecido, o método envia uma resposta padrão com código de status ``200`` e tipo de conteúdo ``text/html``.

**Exemplo:**

```typescript
app.get('/about', (req, res) => {
    res.render('about', { título: 'Sobre Nós', empresa: 'MinhaEmpresa' });
});
```

**Exemplo com Callback:**

```typescript
app.get('/custom-view', (req, res) => {
    res.render('custom', { título: 'Página Personalizada' }, (err, html) => {
        if (err)
            return res.status(500).send('Erro ao renderizar a página.');

        res.send(html);
    });
});
```

Nesse exemplo, uma função de callback personalizada é fornecida. Se ocorrer um erro durante a renderização, o callback lida com isso enviando um código de status ``500`` e uma mensagem de erro ao cliente.

## Compatibilidade de Resposta

### res.trailer(key, fn)

Define um cabeçalho de trailer com a chave e função fornecidas.

### res.flushHeaders()

Libera os cabeçalhos da resposta.

### res.getHeaderNames()

Retorna um array com os nomes dos cabeçalhos atualmente definidos na resposta.

### res.getHeaders()

Retorna o objeto de cabeçalhos da resposta.

### res.hasHeader(name)

Verifica se um cabeçalho específico está presente na resposta.

### res.removeHeader(name)

Remove um cabeçalho específico da resposta.

### res.setTimeout(msecs, cb)

Define um tempo limite para a resposta.

### res.uncork()

Desobstrui o fluxo de resposta.

### res.writeEarlyHints(hints, callback)

Escreve HTTP/1.1 103 Early Hints com o objeto de dicas fornecido.
