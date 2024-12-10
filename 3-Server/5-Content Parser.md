# Parser de Tipo de Conteúdo

Uma das funcionalidades mais poderosas do framework `@cmmv/server` é a sua capacidade de criar e registrar parsers de middleware que lidam com tipos de conteúdo específicos. Diferente de sistemas tradicionais de middleware (como no Express), os middlewares projetados para parsing são executados apenas se o `Content-Type` da requisição corresponder ao tipo específico que você registrou. Isso garante que sua aplicação processe o corpo da requisição de maneira eficiente e somente quando necessário.

Nesta documentação, abordaremos como criar middlewares genéricos para analisar tipos de dados específicos, com foco no uso da função `addContentTypeParser`. Essa função permite registrar parsers personalizados para diferentes tipos de conteúdo.

Em aplicações web, diferentes tipos de conteúdo frequentemente exigem estratégias de parsing distintas. Por exemplo:

- Dados JSON devem ser analisados para objetos JavaScript.
- XML ou CSV podem precisar de tratamento especial para serem analisados corretamente.
- Dados de formulários multipart podem requerer uma estratégia diferente para extrair arquivos e campos.

A função `addContentTypeParser` permite registrar middlewares para lidar com tipos de conteúdo específicos. Isso garante que seu parser seja executado apenas quando o tipo de conteúdo corresponder ao que você definiu.

## Etapas para Criar

**1. Usando `addContentTypeParser` para Registrar o Parser**

Para criar um parser personalizado, você precisa chamar a função `addContentTypeParser` dentro da sua aplicação. Essa função aceita o(s) tipo(s) de conteúdo a ser(em) analisado(s) e a função handler que define o comportamento do parsing.

A função handler deve ser sempre assíncrona, pois geralmente realizará operações de I/O, como leitura do corpo da requisição ou processamento de arquivos.

**2. Definindo a Função Handler Assíncrona**

A função handler é onde ocorre o parsing real. Essa função recebe a requisição (`req`), resposta (`res`) e o payload bruto, que ela processa e anexa ao objeto `req.body`.

Aqui está um exemplo de como criar e registrar um parser personalizado para o tipo de conteúdo `application/json` usando `addContentTypeParser`.

```typescript
const app = cmmv();

// Registra um parser personalizado para 'application/json'
app.addContentTypeParser('application/json', async (req, res, payload) => {
    // Converte o payload bruto (buffer) em uma string
    const bodyString = payload.toString('utf-8');

    // Analisa a string em um objeto JavaScript
    try {
        req.body = JSON.parse(bodyString);
    } catch (err) {
        throw new Error('Payload JSON inválido');
    }
});
```

## Registrando Múltiplos Parsers

Você pode registrar múltiplos parsers para diferentes tipos de conteúdo na sua aplicação. Aqui está um exemplo que registra parsers para os tipos `application/json` e `text/xml`.

```typescript
import xml2js from 'xml2js';

// Registra um parser para JSON
app.addContentTypeParser('application/json', async (req, res, payload) => {
    const jsonData = payload.toString('utf-8');
    try {
        req.body = JSON.parse(jsonData);
    } catch (err) {
        throw new Error('Payload JSON inválido');
    }
});

// Registra um parser para XML
app.addContentTypeParser('text/xml', async (req, res, payload) => {
    const xmlData = payload.toString('utf-8');
    try {
        req.body = await xml2js.parseStringPromise(xmlData);
    } catch (err) {
        throw new Error('Payload XML inválido');
    }
});
```

O método `addContentTypeParser` no `@cmmv/server` fornece uma maneira poderosa de criar middlewares que lidam com tipos de conteúdo específicos, permitindo um parsing eficiente e flexível do corpo da requisição. Ao registrar parsers personalizados que são acionados apenas para seus respectivos tipos de conteúdo, você pode melhorar o desempenho e a segurança da sua aplicação.

Com funções handler assíncronas, seu middleware pode lidar com parsing de dados complexos, como JSON, XML ou CSV, enquanto mantém operações não bloqueantes, tornando o framework ideal para aplicações de servidor de alto desempenho.
