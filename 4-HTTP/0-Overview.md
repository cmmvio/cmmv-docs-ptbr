# Visão Geral

Repositório: [https://github.com/cmmvio/cmmv/tree/main/packages/http](https://github.com/cmmvio/cmmv/tree/main/packages/http)

O framework CMMV introduz sua própria implementação de servidor padrão, o ``@cmmv/server``, que oferece desempenho superior e integração perfeita com o ecossistema geral do CMMV. Este servidor é altamente otimizado e projetado para fornecer suporte embutido a recursos críticos, como compressão, roteamento, manipulação de requisições, entrega de arquivos estáticos, segurança e gerenciamento de middlewares. Como o ``@cmmv/server`` é desenvolvido como uma parte central do CMMV, ele permite maior controle sobre melhorias de recursos, correções de bugs e aprimoramentos de desempenho, tornando-o a opção recomendada para a maioria das aplicações.

O servidor é flexível e compartilha muitas das mesmas APIs e capacidades do [Express](https://expressjs.com/) e do [Fastify](https://fastify.dev/), o que garante uma transição fácil se você já está familiarizado com esses frameworks. No entanto, o ``@cmmv/server`` também inclui integração aprimorada com os contratos, módulos e serviços do CMMV, proporcionando uma experiência de desenvolvimento mais consistente em diferentes camadas da aplicação.

Recursos Principais:
* **Suporte a HTTP e HTTPS:** O adaptador pode inicializar servidores usando tanto HTTP quanto HTTPS com base na configuração.
* **Gerenciamento de Middleware:** Inclui middlewares pré-configurados como compressão, CORS, Helmet (segurança) e gerenciamento de sessões.
* **Entrega de Arquivos Estáticos:** Serve automaticamente arquivos estáticos do diretório ``/public``.
* **Motor de Visão:** Suporta renderização de visões HTML usando o CMMVRenderer, um motor de templates personalizado com opções de segurança como CSP.
* **Registro de Controladores:** Registra automaticamente controladores ao escanear o ``ControllerRegistry`` e mapear métodos HTTP (GET, POST, PUT, DELETE) para caminhos.
* **Sessões e Cabeçalhos de Segurança:** Adiciona gerenciamento de sessões usando ``express-session`` e cabeçalhos de segurança como Política de Segurança de Conteúdo, Proteção contra XSS e HSTS.
* **Rastreamento de Requisições:** Cada requisição recebe um ``requestId`` único para telemetria e monitoramento.
* **Rastreamento de Conexões Abertas:** Rastreia e fecha conexões abertas quando o servidor é encerrado.
* **Tratamento de Erros:** Captura e registra erros durante o processamento de requisições, fornecendo mensagens de erro detalhadas.

## Servidor Padrão

<br/>

\`\`\`typescript
import { Application } from '@cmmv/core';
import { DefaultAdapter, DefaultHTTPModule } from '@cmmv/http';

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [DefaultHTTPModule, ...],
    services: [...],
    contracts: [...],
});
\`\`\`
