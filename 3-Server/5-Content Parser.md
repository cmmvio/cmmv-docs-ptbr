# Parser de Tipo de Conteúdo

Uma das funcionalidades poderosas do framework ``@cmmv/server`` é sua capacidade de criar e registrar middlewares de análise que lidam com tipos de conteúdo específicos. Diferente de sistemas de middleware tradicionais (como no Express), o middleware projetado para análise é executado apenas se o ``Content-Type`` da requisição corresponder ao tipo específico que você registrou. Isso garante que sua aplicação processe o corpo da requisição de forma eficiente e somente quando necessário.

Nesta documentação, abordaremos como criar middlewares genéricos para análise de tipos de dados específicos, focando no uso da função ``addContentTypeParser``. Essa função permite que você registre parsers personalizados para diferentes tipos de conteúdo.

Em aplicações web, diferentes tipos de conteúdo frequentemente exigem estratégias de análise distintas. Por exemplo:

* Dados JSON devem ser analisados em objetos JavaScript.
* XML ou CSV podem precisar de tratamento especial para serem analisados corretamente.
* Dados de formulário multipart podem requerer uma estratégia diferente para extrair arquivos e campos.

A função ``addContentTypeParser`` permite que você registre middlewares para lidar com tipos de conteúdo específicos, garantindo que seu parser seja executado apenas quando o tipo de conteúdo corresponder ao que você definiu.

## Passos para Criar

**1. Usando ``addContentTypeParser`` para Registrar o Parser**

Para criar um parser personalizado, você precisa chamar a função ``addContentTypeParser`` dentro da sua aplicação. Essa função aceita o(s) tipo(s) de conteúdo a ser(em) analisado(s) e a função manipuladora que define o comportamento de análise.

A função manipuladora deve ser sempre assíncrona, pois geralmente realizará operações de entrada/saída, como ler o corpo da requisição ou processar arquivos.

**2. Definindo a Função Manipuladora Assíncrona**

A função manipuladora é onde a análise propriamente dita ocorre. Essa função recebe a requisição (``req``), a resposta (``res``) e o payload bruto do corpo, que ela processa e anexa ao objeto ``req.body``.

Aqui está um exemplo de como criar e registrar um parser personalizado para o tipo de conteúdo ``application/json`` usando ``addContentTypeParser``.

```typescript
const app = cmmv();

// Registra um parser de tipo de conteúdo personalizado para 'application/json'
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

Você pode registrar múltiplos parsers para diferentes tipos de conteúdo na sua aplicação. Aqui está um exemplo que registra parsers para os tipos de conteúdo ``application/json`` e ``text/xml``.

```typescript
import xml2js from 'xml2js';

// Registra um parser JSON
app.addContentTypeParser('application/json', async (req, res, payload) => {
    const jsonData = payload.toString('utf-8');
    try {
        req.body = JSON.parse(jsonData);
    } catch (err) {
        throw new Error('Payload JSON inválido');
    }
});

// Registra um parser XML
app.addContentTypeParser('text/xml', async (req, res, payload) => {
    const xmlData = payload.toString('utf-8');
    try {
        req.body = await xml2js.parseStringPromise(xmlData);
    } catch (err) {
        throw new Error('Payload XML inválido');
    }
});
```

O método ``addContentTypeParser`` no ``@cmmv/server`` oferece uma maneira poderosa de criar middlewares que lidam com tipos de conteúdo específicos, permitindo uma análise eficiente e flexível do corpo da requisição. Ao registrar parsers personalizados que só são acionados para seus respectivos tipos de conteúdo, você pode melhorar o desempenho e a segurança da sua aplicação.

Com funções manipuladoras assíncronas, seu middleware pode lidar com análises complexas de dados, como JSON, XML ou CSV, mantendo operações não bloqueantes, tornando o framework ideal para aplicações de servidor de alto desempenho.
