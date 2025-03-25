# Throttler

Repositório: [https://github.com/cmmvio/cmmv/tree/main/packages/throttler](https://github.com/cmmvio/cmmv/tree/main/packages/throttler)

O módulo `@cmmv/throttler` é uma solução de limitação de taxa poderosa e flexível projetada para funcionar perfeitamente dentro do framework CMMV. Ele oferece capacidades de limitação de taxa para endpoints REST, GraphQL e RPC, garantindo que sua aplicação permaneça protegida contra abusos, solicitações excessivas ou ataques de negação de serviço. Construído com desempenho e integração em mente, este módulo aproveita o ecossistema CMMV para oferecer uma experiência consistente e fácil de usar para desenvolvedores.

O módulo throttler integra-se nativamente com o `@cmmv/server` e outros componentes do CMMV, permitindo que você defina regras de limitação de taxa globalmente ou por endpoint. Ele é altamente configurável, leve e otimizado para aplicações de alto tráfego.

## Instalação

Para adicionar o módulo `@cmmv/throttler` ao seu projeto CMMV, use o seguinte comando:

```bash
$ pnpm add @cmmv/throttler
```

## Uso

Para habilitar a limitação de taxa em sua aplicação CMMV, importe o `ThrottlerModule` e inclua-o na lista de módulos da sua aplicação. Abaixo está um exemplo de como configurá-lo:

```typescript
import { Application } from '@cmmv/core';
import { DefaultAdapter, DefaultHTTPModule } from '@cmmv/http';
import { ThrottlerModule } from '@cmmv/throttler';

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [
        DefaultHTTPModule,
        ThrottlerModule,
        // Outros módulos...
    ],
    providers: [
        // Seus provedores...
    ],
});
```

Uma vez adicionado, o `ThrottlerModule` aplicará automaticamente a limitação de taxa com base na configuração padrão ou em quaisquer configurações personalizadas que você fornecer.

## Configuração

Você pode personalizar o comportamento do módulo `@cmmv/throttler` adicionando um objeto de configuração no seu arquivo `.cmmv.config.cjs`. O módulo usa um sistema de configuração baseado em esquema fornecido pelo `@cmmv/core`.

```javascript
require('dotenv').config();

module.exports = {
    ...

    throttler: {
        limit: 100,
        ttl: 60,
        gcInterval: 60
    },

    ...
};
```
<br/>

- **`limit`**: Define o número máximo de solicitações permitidas dentro da janela de tempo especificada (`ttl`). Aceita um valor de string (por exemplo, `'100'`) para permitir flexibilidade na configuração dinâmica.
- **`ttl`**: A duração (em segundos) da janela de limitação de taxa. Após esse tempo decorrer, o contador de solicitações é reiniciado.
- **`gcInterval`**: O intervalo (em segundos) no qual o throttler limpa as entradas de limite de taxa expiradas para otimizar o uso de memória.

## Principais Recursos

- **Suporte a REST, GraphQL e RPC**: Aplica limitação de taxa em todos os principais paradigmas de manipulação de solicitações do CMMV.
- **Limitação Global e Por Endpoint**: Configure o throttling globalmente ou substitua-o para rotas ou controladores específicos.
- **Leve e Performático**: Otimizado para sobrecarga mínima, mesmo sob grandes volumes de solicitações.
- **Personalizável**: Ajuste as regras de limitação de taxa por meio do arquivo de configuração ou programaticamente.
- **Integração Perfeita**: Funciona imediatamente com o `@cmmv/server` e outros módulos do CMMV.
- **Rastreamento de Solicitações**: Utiliza o `requestId` do `@cmmv/server` para monitoramento de solicitações limitadas.