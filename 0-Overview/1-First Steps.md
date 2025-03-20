# Primeiros Passos

CMMV (Contract Model Model View) é uma revolução no desenvolvimento de aplicações web, quebrando paradigmas e redefinindo a maneira como criamos, mantemos e escalamos projetos digitais. Inspirado pelas melhores práticas e conceitos inovadores, o CMMV integra o poder dos contratos para gerar automaticamente estruturas robustas e seguras, eliminando a complexidade do código manual e proporcionando uma experiência de desenvolvimento sem precedentes.

Imagine uma plataforma onde a definição de contratos em TypeScript se torna o coração da sua aplicação, gerando automaticamente APIs, controladores, entidades ORM e até mesmo comunicação via RPC binário, tudo com desempenho otimizado e integração perfeita com as tecnologias mais modernas. Com o CMMV, você não apenas acelera o desenvolvimento, mas também garante a qualidade e a consistência do seu código, reduzindo drasticamente erros e retrabalho.

Além disso, o CMMV oferece uma interface reativa e leve, baseada no Vue 3, mas com capacidade de suportar outros frameworks como React e Angular, sempre focando em desempenho e SEO. Com o CMMV, o frontend não é apenas uma camada de apresentação, mas uma parte integral e dinâmica da sua aplicação, sincronizada em tempo real com o backend.

Seja você um desenvolvedor experiente ou um iniciante na programação, o CMMV permite que qualquer pessoa construa sistemas poderosos, escaláveis e modernos, eliminando barreiras técnicas e permitindo que a criatividade e a inovação estejam no centro da sua jornada de desenvolvimento. Mais do que um framework, é uma nova forma de pensar e construir o futuro das aplicações web.

## Pré-requisitos

Para executar o CMMV, será necessário ter o `Node.js (versão >= 20.0)` instalado no seu sistema operacional.

## Configuração com CLI

O CMMV agora fornece uma CLI (Interface de Linha de Comando) para simplificar o processo de instalação e configurar rapidamente seu projeto com as configurações desejadas.

Para inicializar um novo projeto, utilize o seguinte comando:

```bash
$ pnpm dlx @cmmv/cli@latest init nome-do-projeto
```

Esse comando guiará você por um processo de configuração interativo, perguntando sobre suas preferências, como habilitação do Vite, RPC, cache, tipo de repositório e configuração de visualização (por exemplo, Vue 3 ou Reactivity). Ele criará automaticamente os arquivos e pastas necessários, configurará as dependências e ajustará o projeto.

## Configuração Manual (Legacy Setup)

Se preferir configurar o projeto manualmente, ainda é possível instalar os módulos necessários individualmente:

```bash
$ pnpm add @cmmv/core @cmmv/http reflect-metadata
```

## Estrutura

O CMMV é um projeto que se inspira nas melhores práticas e tecnologias do desenvolvimento moderno, trazendo referências sólidas para criar um framework poderoso e flexível. No coração do CMMV estão os contratos, que seguem um modelo semelhante ao TypeORM e Protobuf, mas vão além, oferecendo um conjunto completo de configurações que permitem a geração automática de toda a estrutura base da aplicação.

Escrito em TypeScript, o CMMV adota uma estrutura de backend semelhante ao NestJS, mas com ajustes que tornam a injeção de dependências e o controle de módulos mais intuitivos e adaptados às necessidades do desenvolvimento moderno. A ideia é oferecer um sistema modular, onde os desenvolvedores tenham controle total sobre as camadas da aplicação, permitindo que a arquitetura se adapte às demandas específicas de cada projeto.

No frontend, o CMMV incorpora um controlador básico de data binding inspirado no Vue 3, mas simplificado e extremamente leve. Essa camada reativa pode ser facilmente substituída por qualquer outro framework, garantindo flexibilidade e desempenho sem comprometer a simplicidade.

## Estrutura de Diretórios

```
.
└── .generated/
    ├── controllers/
    ├── gateways/
    ├── protos/
    ├── services/
    └── app.module.ts
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
    └── main.ts
```

### Explicação da Estrutura

- **public**: Contém todos os recursos estáticos da aplicação, como CSS, JavaScript e imagens.
- **src**: O núcleo da aplicação, organizado de maneira modular para facilitar o desenvolvimento.
  - **contracts**: Diretório onde os contratos são definidos e a partir dos quais são gerados automaticamente os controladores, entidades, modelos e serviços.
  - **controllers**: Responsáveis por lidar com requisições HTTP e retornar respostas apropriadas.
  - **entities**: Caso o módulo `repository` esteja presente, armazena as entidades ORM geradas.
  - **models**: Definem a estrutura dos dados manipulados pela aplicação.
  - **services**: Encapsulam a lógica de negócios e interagem com os módulos da aplicação.
  - **index.ts**: Ponto de entrada da aplicação, onde o servidor é inicializado.

## Exemplo de Código

```typescript
import { Application } from "@cmmv/core";
import { DefaultAdapter, DefaultHTTPModule } from "@cmmv/http";
import { ProtobufModule } from "@cmmv/protobuf";
import { WSModule, WSAdapter } from "@cmmv/ws";
import { RepositoryModule, Repository } from "@cmmv/repository";
import { ApplicationModule } from "./app.module";

Application.create({
    httpAdapter: DefaultAdapter,
    wsAdapter: WSAdapter,
    modules: [
        DefaultHTTPModule,
        ProtobufModule,
        WSModule,
        RepositoryModule
    ],
    services: [Repository],
    contracts: [...]
});
```

## Geração Automática

O maior diferencial do CMMV é sua capacidade de **gerar automaticamente** código com base nos contratos. Ao iniciar a aplicação, o sistema verifica o diretório `contracts` e, dependendo dos módulos instalados, gera automaticamente:

- **Controladores, Entidades, Modelos e Serviços**: Criados com base nos contratos.
- **Suporte RPC com Protobuf**: Se o módulo `protobuf` estiver presente, arquivos `.proto` serão gerados automaticamente.
- **Modularidade do Servidor HTTP**: Suporte a Express e Fastify, com possibilidade de novos módulos no futuro.

## Arquivos Estáticos

A recomendação para arquivos estáticos é utilizar a integração de CDN disponível no editor, pois, ao construir a aplicação, todos os arquivos estáticos serão automaticamente carregados e atualizados na CDN. No entanto, se desejar manter a disponibilidade de arquivos servidos pela aplicação, o sistema está configurado para mapear o diretório `/public`. Se este diretório existir, seus arquivos serão copiados automaticamente na exportação e implantação, tornando-se acessíveis ao público sem controle de acesso.

## Modo de Desenvolvimento

Para iniciar a aplicação em modo de desenvolvimento, utilize o comando:

```bash
$ pnpm dev
```

Isso iniciará a aplicação em modo de observação, onde qualquer alteração nos arquivos recarregará automaticamente o servidor, garantindo um ambiente ágil para o desenvolvimento.
