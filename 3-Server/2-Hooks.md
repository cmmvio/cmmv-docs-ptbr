# Hooks

O sistema de hooks neste framework de servidor é projetado para seguir um ciclo de vida semelhante ao do [Fastify](https://fastify.dev/docs/latest/Reference/Hooks/), mas com hooks vinculados à aplicação. Hooks são funções executadas em pontos específicos do ciclo de vida da requisição, permitindo comportamento personalizável durante o processamento de uma requisição. Esse sistema possibilita interceptar e manipular requisições, respostas ou erros em momentos-chave do ciclo de vida.

Cada hook é anexado à aplicação e pode ser definido usando o método ``addHook()``. A ordem em que os hooks são chamados depende do tipo de hook. Abaixo, abordaremos três hooks específicos: ``onRequest``, ``preParsing`` e ``onRequestAbort``.

## onRequest

O hook ``onRequest`` é executado logo no início do ciclo de vida da requisição, antes de qualquer análise da requisição. Este é o local ideal para realizar operações como autenticação, logging ou configuração de dados necessários para a requisição.

**Exemplo:**

```typescript
app.addHook('onRequest', (req, res, done) => {
    done()
});
```

Ou com ``async/await``:

```typescript
app.addHook('onRequest', async (req, res) => {
    await asyncMethod()
});
```

## preParsing

O hook ``preParsing`` é chamado após o hook ``onRequest``, mas antes que o corpo da requisição ou os parâmetros de consulta sejam analisados. É ideal para casos em que você precisa manipular ou validar o fluxo bruto da requisição antes de analisá-lo (por exemplo, descompressão personalizada ou criptografia).

**Exemplo:**

```typescript
app.addHook('preParsing', async (req, res) => {
    console.log('A requisição está prestes a ser analisada');
    // Modifique ou inspecione o fluxo bruto da requisição aqui
});
```

## onRequestAbort

O hook ``onRequestAbort`` é acionado quando o cliente aborta a requisição, geralmente devido a um timeout ou ao cliente fechar a conexão antes que o servidor envie a resposta. É útil para operações de limpeza, como cancelar consultas ao banco de dados ou registrar requisições abortadas.

**Exemplo:**

```typescript
app.addHook('onRequestAbort', async (req) => {
    console.log(`Requisição abortada: ${req.id}`);
    // Realize tarefas de limpeza aqui
});
```

## onError

O hook ``onError`` é acionado sempre que ocorre um erro durante o processamento de uma requisição. Esse hook oferece a oportunidade de registrar o erro, transformá-lo em uma resposta mais significativa ou lidar com diferentes tipos de erros de maneira personalizada.

**Exemplo:**

```typescript
app.addHook('onError', async (error, req, res, done) => {
    console.error('Erro ocorrido:', error);
    res.status(500).send({ mensagem: 'Ocorreu um erro interno' });
    done();
});
```

## onSend

O hook ``onSend`` é acionado logo antes que a resposta seja enviada ao cliente, após o processamento da requisição. É útil para modificar o corpo da resposta, cabeçalhos ou realizar verificações finais antes de enviar a resposta.

**Exemplo:**

```typescript
app.addHook('onSend', async (req, res, payload) => {
    console.log('A resposta está prestes a ser enviada');
    // Modifique o payload ou cabeçalhos
    res.setHeader('X-Custom-Header', 'ValorPersonalizado');
    return payload;  // O payload modificado ou original a ser enviado
});
```

## onResponse

O hook ``onResponse`` é executado após a resposta ter sido completamente enviada ao cliente. É útil para registrar informações da requisição e resposta, realizar tarefas de limpeza ou acionar análises.

**Exemplo:**

```typescript
app.addHook('onResponse', async (req, res) => {
    console.log(`Resposta enviada para a requisição ${req.id}`);
    // Realize tarefas pós-resposta, como logging
});
```

## onTimeout

O hook ``onTimeout`` é chamado quando a requisição atinge o tempo limite devido a ultrapassar o tempo permitido para processamento. Isso pode acontecer quando o servidor demora muito para responder. O hook é útil para registrar eventos de timeout ou acionar comportamentos alternativos quando ocorrem timeouts.

**Exemplo:**

```typescript
app.addHook('onTimeout', async (req, res) => {
    console.log(`Requisição atingiu o tempo limite: ${req.id}`);
    res.status(408).send({ mensagem: 'Tempo Limite da Requisição' });
});
```

## onListen

O hook ``onListen`` é acionado quando o servidor começa a ouvir requisições na porta e host especificados. Ele oferece uma oportunidade para realizar tarefas como registrar que o servidor foi iniciado ou executar operações pós-inicialização.

**Exemplo:**

```typescript
app.addHook('onListen', async (server) => {
    console.log(`O servidor está agora ouvindo na porta ${server.address().port}`);
});
```

## onReady

O hook ``onReady`` é chamado quando a aplicação está totalmente inicializada e pronta para lidar com requisições recebidas. É útil para realizar operações que precisam ocorrer quando o servidor estiver completamente preparado, como pré-carregar dados, verificar a saúde do sistema ou registrar um evento de prontidão.

**Exemplo:**

```typescript
app.addHook('onReady', async () => {
    console.log('A aplicação está pronta para aceitar requisições');
});
```

## onClose

O hook ``onClose`` é acionado quando o servidor está prestes a fechar ou encerrar. É útil para tarefas de limpeza, como fechar conexões com o banco de dados, liberar recursos ou registrar eventos de desligamento.

**Exemplo:**

```typescript
app.addHook('onClose', async (server) => {
    console.log('O servidor está sendo desligado');
    // Realize tarefas de limpeza
});
```
