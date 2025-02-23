# Visão Geral (HTTP)

Repositório: [https://github.com/cmmvio/cmmv/tree/main/packages/http](https://github.com/cmmvio/cmmv/tree/main/packages/http)

O framework CMMV apresenta sua própria implementação de servidor padrão, ``@cmmv/server``, que oferece desempenho superior e integração perfeita com o ecossistema geral do CMMV. Este servidor é altamente otimizado e projetado para fornecer suporte integrado para recursos críticos, como compressão, roteamento, manipulação de solicitações, serviço de arquivos estáticos, segurança e gerenciamento de middlewares. Como ``@cmmv/server`` foi desenvolvido como parte essencial do CMMV, ele permite um melhor controle sobre melhorias de recursos, correções de bugs e melhorias de desempenho, tornando-o a opção recomendada para a maioria das aplicações.

O servidor é flexível e compartilha muitas das mesmas APIs e capacidades do [Express](https://expressjs.com/) e do [Fastify](https://fastify.dev/), o que garante uma transição fácil caso você esteja familiarizado com esses frameworks. No entanto, o ``@cmmv/server`` também inclui integração aprimorada com contratos, módulos e serviços do CMMV, proporcionando uma experiência de desenvolvedor mais consistente em diferentes camadas da aplicação.

## Principais Recursos:
* **Suporte HTTP e HTTPS:** O adaptador pode inicializar servidores usando HTTP e HTTPS com base na configuração.
* **Gerenciamento de Middlewares:** Inclui middlewares pré-configurados, como compressão, CORS, Helmet (segurança) e gerenciamento de sessões.
* **Serviço de Arquivos Estáticos:** Serve automaticamente arquivos estáticos do diretório /public.
* **Engine de Visualização:** Suporta renderização de visualizações HTML usando o CMMVRenderer, uma engine de templates personalizada com opções de segurança como CSP.
* **Registro de Controladores:** Registra automaticamente controladores ao escanear o `ControllerRegistry` e mapeia métodos HTTP (GET, POST, PUT, DELETE) para caminhos.
* **Gerenciamento de Sessão e Cabeçalhos de Segurança:** Adiciona gerenciamento de sessão com `express-session` e cabeçalhos de segurança, como Política de Segurança de Conteúdo (CSP), Proteção contra XSS e HSTS.
* **Rastreamento de Solicitações:** Cada solicitação recebe um `requestId` exclusivo para telemetria e monitoramento.
* **Rastreamento de Conexões Abertas:** Acompanha e fecha conexões abertas quando o servidor é encerrado.
* **Manipulação de Erros:** Captura e registra erros durante o processamento de solicitações, fornecendo mensagens detalhadas de erro.

## Servidor Padrão

```typescript
import { Application } from '@cmmv/core';
import { DefaultAdapter, DefaultHTTPModule } from '@cmmv/http';

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [DefaultHTTPModule, ...],
    services: [...],
    contracts: [...],
});
```

## Benchmarks

* [https://github.com/fastify/benchmarks](https://github.com/fastify/benchmarks)
* Máquina: linux x64 | 32 vCPUs | 128.0GB Mem
* Node: v20.17.0
* Execução: Qui, 26 de Nov de 2024 15:23:41 GMT+0000 (Horário Universal Coordenado)
* Método: ``autocannon -c 100 -d 40 -p 10 localhost:3000``

| Framework                | Versão   | Router | Solicitações/s | Latência (ms) | Throughput/Mb |
|--------------------------|----------|--------|----------------|---------------|---------------|
| bare                     | v20.17.0 | ✗      | 88267.6        | 10.87         | 15.74         |
| fastify                  | 5.1.0    | ✓      | 87846.6        | 10.91         | 15.75         |
| cmmv                     | 0.6.2    | ✓      | 79041.6        | 12.16         | 14.17         |
| koa                      | 2.15.3   | ✗      | 76639.6        | 12.54         | 13.67         |
| express                  | 5.0.1    | ✓      | 21549.2        | 45.89         | 3.84          |
| express-with-middlewares | 5.0.1    | ✓      | 18930.4        | 52.30         | 7.04          |

## Express

Além do servidor padrão, o CMMV também suporta Express e Fastify como adaptadores HTTP alternativos, fornecendo flexibilidade para desenvolvedores que preferem ou precisam usar esses frameworks populares. Ambos os adaptadores são totalmente integrados ao ecossistema CMMV e podem ser usados simplesmente alterando o adaptador na configuração da aplicação.

```bash
$ pnpm add @cmmv/express express body-parser cors express-session helmet uuid
```

### Integração

O módulo ``@cmmv/express`` fornece um adaptador HTTP alternativo baseado no Express, permitindo que você use middlewares e recursos do Express de forma transparente com sua aplicação CMMV.

```typescript
import { Application } from '@cmmv/core';
import { ExpressAdapter, ExpressModule } from '@cmmv/express';

Application.create({
    httpAdapter: ExpressAdapter,
    modules: [ExpressModule, ...],
    services: [...],
    contracts: [...],
});
```

O adaptador registra automaticamente todos os controladores do `ControllerRegistry`. Ele mapeia as rotas dos controladores para os métodos HTTP correspondentes (GET, POST, etc.) e processa os middlewares definidos no nível do controlador.

## Fastify

O Adaptador Fastify no CMMV oferece uma alternativa ao Adaptador Express, permitindo manipulação HTTP leve e de alto desempenho usando o framework Fastify. Este adaptador integra middlewares importantes, como compressão, CORS, Helmet para segurança e serviço de arquivos estáticos. Ele registra automaticamente os controladores e gerencia o ciclo de vida das solicitações recebidas. O Adaptador Fastify segue a mesma estrutura do Adaptador Express, suportando gerenciamento de sessões e renderização de conteúdo, enquanto oferece um ambiente mais rápido e otimizado.

```bash
$ pnpm add @cmmv/fastify @fastify/compress @fastify/cors @fastify/helmet @fastify/secure-session @fastify/static @fastify/view
```

### Integração

```typescript
import { Application } from '@cmmv/core';
import { FastifyAdapter, FastifyModule } from '@cmmv/fastify';

Application.create({
    httpAdapter: FastifyAdapter,
    modules: [FastifyModule, ...],
    services: [...],
    contracts: [...],
});
```

## Middlewares

A configuração `httpMiddlewares` permite que você injete middlewares personalizados no adaptador HTTP (como Express ou Fastify) durante a inicialização da aplicação. Isso fornece flexibilidade adicional ao permitir que você aplique qualquer função middleware para lidar com tarefas como logging, validação de solicitações ou verificações de segurança.

Para usar esse recurso, defina um array de funções middleware na propriedade `httpMiddlewares` ao criar a aplicação.

Aqui está um exemplo onde adicionamos o middleware de logging Morgan a uma aplicação CMMV baseada no Express:

```typescript
import { Application } from "@cmmv/core";
import { ExpressAdapter, ExpressModule } from "@cmmv/http";
import { ViewModule } from "@cmmv/view";
import morgan from "morgan";

Application.create({
    httpAdapter: ExpressAdapter,
    httpMiddlewares: [
        morgan('dev'),
    ],
    modules: [
        ExpressModule,
        ViewModule
    ]
});
```

Neste exemplo, o middleware Morgan é usado para registrar solicitações HTTP recebidas no formato 'dev'. Esse middleware é passado para a configuração `httpMiddlewares`, que o aplica automaticamente durante a inicialização da aplicação.

* O array `httpMiddlewares` pode incluir qualquer número de middlewares, e cada um será aplicado na ordem em que são fornecidos.
* Certifique-se de importar qualquer middleware personalizado antes de passá-lo para a configuração.

Essa configuração ajuda a expandir as capacidades de sua aplicação ao permitir o uso de qualquer middleware personalizado ou de terceiros que o adaptador HTTP escolhido (como Express ou Fastify) suporte.
