# Primeiros Passos

CMMV (Contract Model Model View) é uma revolução no desenvolvimento de aplicações web, quebrando paradigmas e redefinindo como criamos, mantemos e escalamos projetos digitais. Inspirado pelas melhores práticas e conceitos inovadores, o CMMV integra o poder dos contratos para gerar automaticamente estruturas robustas e seguras, eliminando a complexidade do código manual e proporcionando uma experiência de desenvolvimento sem precedentes.

Imagine uma plataforma onde a definição de contratos em TypeScript se torna o coração da sua aplicação, gerando automaticamente APIs, controladores, entidades ORM e até mesmo comunicação via RPC binário, tudo com desempenho otimizado e integração perfeita com as tecnologias mais modernas. Com o CMMV, você não só acelera o desenvolvimento, mas também garante a qualidade e a consistência do código, reduzindo drasticamente erros e retrabalho.

Além disso, o CMMV oferece uma interface reativa e leve, baseada no Vue 3, mas com a capacidade de suportar outros frameworks como React e Angular, sempre com foco em desempenho e SEO. No CMMV, o frontend não é apenas uma camada de apresentação, mas uma parte integral e dinâmica da sua aplicação, sincronizada em tempo real com o backend.

Seja você um desenvolvedor experiente ou um iniciante em programação, o CMMV capacita todos a construir sistemas poderosos, escaláveis e modernos, eliminando barreiras técnicas e colocando a criatividade e inovação no centro da jornada de desenvolvimento. É mais do que um framework; é uma nova maneira de pensar e construir o futuro das aplicações web.

## Pré-requisitos

Para executar o CMMV, será necessário ter o `Node.js (versão >= 18.0)` instalado no seu sistema operacional.

## Configuração com CLI

O CMMV agora oferece uma CLI (Interface de Linha de Comando) para simplificar o processo de instalação e configurar rapidamente o seu projeto com as opções desejadas.

Para inicializar um novo projeto, utilize o seguinte comando:

```bash
$ pnpm dlx @cmmv/cli@latest init <nome-do-projeto>
```

Esse comando irá guiá-lo por um processo de configuração interativo, perguntando sobre as suas configurações preferidas, como ativar Vite, RPC, caching, tipo de repositório e configuração de view (ex.: Vue 3 ou Reatividade). Ele criará automaticamente os arquivos e pastas necessários, configurará dependências e ajustará o projeto.

## Configuração Manual (Legado)

Se você preferir configurar o projeto manualmente, ainda é possível instalar os módulos necessários individualmente:

```bash
$ pnpm add @cmmv/core @cmmv/http @cmmv/view rxjs reflect-metadata class-validator class-transformer fast-json-stringify
```

## Estrutura

O CMMV é um projeto que se inspira em algumas das melhores práticas e tecnologias do desenvolvimento moderno, trazendo referências sólidas para criar um framework poderoso e flexível. No coração do CMMV estão os contratos, que seguem um modelo semelhante ao TypeORM e Protobuf, mas vão além, oferecendo uma gama completa de configurações que permitem a geração automática de toda a base estrutural da aplicação. Esses contratos são a peça fundamental do projeto, pois a partir deles é possível criar desde entidades de banco de dados até APIs e controladores.

Escrito em TypeScript, o CMMV adota uma estrutura de backend que se assemelha ao NestJS, mas com ajustes que tornam a injeção de dependências e o controle de módulos mais intuitivos e adaptados às necessidades do desenvolvimento moderno. A ideia é oferecer um sistema modular, onde os desenvolvedores têm controle total sobre as camadas da aplicação, permitindo que a arquitetura se adapte às demandas específicas de cada projeto.

No frontend, o CMMV incorpora um controlador básico de data binding inspirado no Vue 3, mas simplificado e extremamente leve. Essa camada reativa pode ser facilmente substituída por qualquer outro framework, como React, Angular ou até mesmo o próprio Vue, garantindo que a aplicação mantenha flexibilidade e desempenho, sem sacrificar a simplicidade.

O projeto é fortemente influenciado pelos princípios SOLID, aplicando conceitos bem definidos de responsabilidade, mas sem seguir rigidamente todas as regras. O objetivo é manter a arquitetura limpa e organizada, mas com a flexibilidade necessária para atender às realidades práticas do desenvolvimento. Além disso, conceitos de TDD (Desenvolvimento Orientado por Testes) são aplicados para facilitar a manutenção e garantir a qualidade do código, mas o sistema é completamente modular e opcional, permitindo que cada desenvolvedor escolha o que melhor se adapta ao seu fluxo de trabalho.

Em resumo, o CMMV é mais do que um conjunto de ferramentas e frameworks; é uma abordagem integrada ao desenvolvimento, onde os contratos estão no núcleo de tudo. Com uma base sólida, o CMMV dá aos desenvolvedores a liberdade de construir aplicações modernas e escaláveis, sem comprometer a simplicidade e a flexibilidade essenciais para a inovação contínua.

## Fluxo da Aplicação

Todo fluxo da aplicação começa no arquivo `src/index.ts`, que é responsável por iniciar o servidor HTTP e seus adaptadores.

```typescript
import { Application } from "@cmmv/core";
import { DefaultAdapter, DefaultHTTPModule } from "@cmmv/http";
import { ProtobufModule } from "@cmmv/protobuf";
import { WSModule, WSAdapter } from "@cmmv/ws";
import { ViewModule } from "@cmmv/view";
import { RepositoryModule, Repository } from "@cmmv/repository";
import { ApplicationModule } from "./app.module";

Application.create({
    httpAdapter: DefaultAdapter,    
    wsAdapter: WSAdapter,
    modules: [
        DefaultHTTPModule,
        ProtobufModule,
        WSModule,
        ViewModule,
        RepositoryModule
    ],
    services: [Repository],
    contracts: [...]
});
```

É possível realizar configurações através do arquivo `.cmmv.config.js` sem precisar alterar o código-fonte da aplicação. No futuro, forneceremos outros módulos de servidor HTTP.

## Diretórios Padrão

Os diretórios padrão que acompanham o projeto são os seguintes:

```
.
└── public/
    ├── assets/
    ├── templates/
    └── views/
└── src/
    ├── contracts/
    ├── controllers/
    ├── entities/
    ├── models/
    ├── services/
    ├── app.module.ts
    └── index.ts
```

**Explicação dos Diretórios:**

- **public/**: Contém todos os recursos estáticos da aplicação, como CSS, JavaScript e imagens.
- **src/**: Diretório principal do código-fonte, onde a aplicação é organizada de forma modular para facilitar o desenvolvimento e a manutenção.

## Geração Automática

O grande diferencial do CMMV é sua capacidade de gerar código automaticamente com base nos contratos definidos.

- **Controladores, Entidades, Modelos e Serviços**: Gerados automaticamente com base nos contratos.
- **Suporte a RPC com Protobuf**: Permite comunicação em tempo real via WebSocket.

## Arquivos Estáticos

Arquivos estáticos são copiados automaticamente do diretório `/public` e ficam disponíveis para acesso público.

## Modo Dev

Inicie a aplicação em modo de desenvolvimento:

```bash
$ pnpm dev
```

Com isso, a aplicação recarrega automaticamente ao detectar alterações nos arquivos.
