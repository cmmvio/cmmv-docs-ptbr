# Hooks

O sistema de hooks foi projetado para seguir um ciclo de vida semelhante ao do [Fastify](https://fastify.dev/docs/latest/Reference/Hooks/), mas com hooks anexados à aplicação. Hooks são funções executadas em pontos específicos no ciclo de vida de uma requisição, permitindo comportamentos personalizados durante o manuseio de uma requisição. Esse sistema possibilita interceptar e manipular requisições, respostas ou erros em pontos-chave do ciclo de vida.

Cada hook é anexado à aplicação e pode ser definido usando o método `addHook()`. A ordem em que os hooks são chamados depende do tipo de hook. Abaixo, abordaremos três hooks específicos: `onRequest`, `preParsing` e `onRequestAbort`.

## onRequest

O hook `onRequest` é executado logo no início do ciclo de vida da requisição, antes de qualquer análise da requisição. Este é o local ideal para realizar operações como autenticação, registro de logs ou configuração de dados necessários da requisição.

**Exemplo:**

```typescript
app.addHook('onRequest', (req, res, done) => {
    done()
});
```

Ou `async/await`:

```typescript
app.addHook('onRequest', async (req, res) => {
    await asyncMethod()
});
```

## preParsing

O hook `preParsing` é chamado após o hook `onRequest`, mas antes que o corpo da requisição ou os parâmetros de consulta sejam analisados. É ideal para casos onde é necessário manipular ou validar o fluxo bruto da requisição antes de analisá-lo (por exemplo, descompressão ou criptografia personalizada).

**Exemplo:**

```typescript
app.addHook('preParsing', async (req, res) => {
    console.log('A requisição está prestes a ser analisada');
    // Modificar ou inspecionar o fluxo bruto da requisição aqui
});
```

## onRequestAbort

O hook `onRequestAbort` é acionado quando o cliente aborta a requisição, geralmente devido a um timeout ou ao cliente fechar a conexão antes que o servidor envie a resposta. Isso é útil para operações de limpeza, como cancelar consultas ao banco de dados ou registrar requisições abortadas.

**Exemplo:**

```typescript
app.addHook('onRequestAbort', async (req) => {
    console.log(`Requisição abortada: \${req.id}`);
    // Realizar tarefas de limpeza aqui
});
```

## onError

O hook `onError` é acionado sempre que ocorre um erro durante o processamento de uma requisição. Este hook oferece uma oportunidade para registrar o erro, transformá-lo em uma resposta mais significativa ou lidar com diferentes tipos de erros de forma personalizada.

**Exemplo:**

```typescript
app.addHook('onError', async (error, req, res, done) => {
    console.error('Ocorreu um erro:', error);
    res.status(500).send({ message: 'Ocorreu um erro interno' });
    done();
});
```

## onSend

O hook `onSend` é acionado pouco antes de a resposta ser enviada ao cliente, após a requisição ter sido processada. Isso é útil para modificar o corpo da resposta, cabeçalhos ou realizar verificações finais antes de enviar a resposta.

**Exemplo:**

```typescript
app.addHook('onSend', async (req, res, payload) => {
    console.log('A resposta está prestes a ser enviada');
    // Modificar o payload ou os cabeçalhos
    res.setHeader('X-Custom-Header', 'ValorPersonalizado');
    return payload;  // O payload modificado ou original a ser enviado
});
```

## onResponse

O hook `onResponse` é executado após a resposta ter sido completamente enviada ao cliente. Isso é útil para registrar informações de requisição e resposta, realizar tarefas de limpeza ou acionar análises.

**Exemplo:**

```typescript
app.addHook('onResponse', async (req, res) => {
    console.log(`Resposta enviada para a requisição \${req.id}`);
    // Realizar tarefas pós-resposta, como registrar logs
});
```

## onTimeout

O hook `onTimeout` é chamado quando a requisição excede o tempo permitido para processamento, resultando em um timeout. Isso pode ocorrer quando o servidor demora muito para responder. O hook é útil para registrar eventos de timeout ou acionar comportamentos alternativos quando ocorrências de timeout acontecem.

**Exemplo:**

```typescript
app.addHook('onTimeout', async (req, res) => {
    console.log(`Requisição com timeout: \${req.id}`);
    res.status(408).send({ message: 'Tempo limite da requisição' });
});
```

## onListen

O hook `onListen` é acionado quando o servidor começa a escutar requisições na porta e host especificados. Ele fornece uma oportunidade para realizar tarefas como registrar que o servidor foi iniciado ou executar quaisquer operações pós-inicialização.

**Exemplo:**

```typescript
app.addHook('onListen', async (server) => {
    console.log(`O servidor está agora escutando na porta \${server.address().port}`);
});
```

## onReady

O hook `onReady` é chamado quando a aplicação está totalmente inicializada e pronta para lidar com requisições. Isso é útil para realizar quaisquer operações que precisam ocorrer uma vez que o servidor esteja completamente preparado, como pré-carregar dados, verificar a saúde do sistema ou registrar um evento de prontidão.

**Exemplo:**

```typescript
app.addHook('onReady', async () => {
    console.log('A aplicação está pronta para aceitar requisições');
});
```

## onClose

O hook `onClose` é acionado quando o servidor está prestes a ser fechado ou encerrado. Isso é útil para tarefas de limpeza, como fechar conexões com bancos de dados, liberar recursos ou registrar eventos de desligamento.

**Exemplo:**

```typescript
app.addHook('onClose', async (server) => {
    console.log('O servidor está sendo encerrado');
    // Realizar tarefas de limpeza
});
```