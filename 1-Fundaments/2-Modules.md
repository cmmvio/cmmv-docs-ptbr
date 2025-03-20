# Módulos

O sistema de módulos do CMMV compartilha semelhanças com o NestJS, fornecendo uma abordagem modular para o desenvolvimento de aplicações. No entanto, ele difere em um aspecto chave: não há controle de contexto de dependência. Isso significa que os serviços não são injetados ou gerenciados por um contêiner centralizado de injeção de dependência. Em vez disso, qualquer provedor (serviço, utilitário, etc.) pode ser criado independentemente e adicionado aos módulos onde for necessário. Para serviços compartilhados em todo o sistema, como as classes Repository ou Config, é recomendado usar registros singleton, tornando esses serviços globalmente acessíveis sem a necessidade de instanciação repetida.

## Principais Características

* **Sem Controle de Contexto de Dependência:** Os serviços não são injetados automaticamente. Em vez disso, eles podem ser criados e indexados dentro do sistema de módulos.
* **Registros Singleton:** Serviços compartilhados podem ser registrados como singletons para evitar duplicação e torná-los acessíveis em todo o sistema.
* **Composição de Módulos:** Os módulos no CMMV são flexíveis, permitindo que você defina controladores, serviços (provedores), transpilers, contratos e submódulos dentro de um único módulo.
* **Geração Automática de Módulos:** O sistema gera o módulo principal (por exemplo, `app.module.ts`) com base em contratos, controladores e gateways definidos dentro da aplicação.

## Exemplo

Aqui está um exemplo de um módulo completo com todas as propriedades possíveis no CMMV:

```typescript
import { Module } from '@cmmv/core';
import { DocsController } from './docs.controller';
import { DocsService } from './docs.service';
import { DocsTranspile } from './docs.transpile';
import { DocsContract } from './docs.contract';
import { SubModule } from './submodule';

export let DocsModule = new Module({
    controllers: [DocsController],
    providers: [DocsService],
    transpilers: [DocsTranspile],
    submodules: [SubModule],
    contracts: [DocsContract]
});
```

<br/>

* **controllers:** Uma matriz de controladores que lidam com requisições HTTP e retornam respostas. Eles gerenciam as rotas da aplicação e são responsáveis por interagir com os serviços.
* **providers:** Uma lista de serviços ou classes que contêm a lógica de negócios do seu módulo. Esses provedores são instanciados manualmente e indexados quando necessário.
* **transpilers:** São responsáveis por gerar arquivos necessários, como entidades de banco de dados, definições Protobuf ou outros artefatos de código baseados em contratos.
* **submodules:** Permite dividir sua aplicação em módulos menores e autocontidos que podem ser aninhados dentro de outros módulos.
* **contracts:** Define os contratos associados a este módulo. Os contratos definem a estrutura de dados e o comportamento das entidades, e os transpilers geram os arquivos apropriados (por exemplo, modelos de banco de dados, endpoints de API).

## Registros Singleton

Em vez de injetar serviços compartilhados em vários módulos, registros singleton podem ser usados para serviços como `Repository` ou `Config`, que são destinados a serem acessíveis globalmente. Isso reduz a necessidade de injeção de dependência e garante que esses serviços sejam instanciados apenas uma vez e compartilhados por todo o sistema.

## Aplicação

Sempre que contratos forem definidos, o sistema criará automaticamente um módulo em `/src/app.module.ts`. Este módulo incluirá todos os contratos, controladores e gateways gerados automaticamente pela aplicação. Aqui está um exemplo de como o arquivo pode ser gerado:

```typescript
// Gerado automaticamente pelo CMMV

import { Module } from '@cmmv/core';
import { TaskController } from './controllers/task.controller';
import { TaskService } from './services/task.service';
import { TaskGateway } from './gateways/task.gateway';

export let ApplicationModule = new Module({
    controllers: [TaskController],
    providers: [TaskService, TaskGateway]
});
```

É importante notar que o arquivo `/src/app.module.ts` é gerado automaticamente pelo sistema e não deve ser modificado manualmente, pois será recriado toda vez que a aplicação for iniciada. Quaisquer alterações feitas diretamente neste arquivo serão sobrescritas. Se você precisar adicionar ou modificar módulos, serviços ou controladores, é recomendado fazê-lo em arquivos separados e registrá-los adequadamente dentro da aplicação para garantir que suas modificações sejam preservadas.

## Design Modular

O CMMV segue um design modular, permitindo que você divida sua aplicação em partes reutilizáveis e manuteníveis. Cada módulo pode se concentrar em um domínio ou funcionalidade específica, o que melhora a separação de preocupações, a manutenibilidade e a escalabilidade.

* **Estrutura Orientada a Funcionalidades:** Agrupe controladores, serviços e outros componentes relacionados à mesma funcionalidade juntos em um módulo.
* **Reusabilidade:** Módulos podem ser reutilizados em diferentes partes da aplicação, e submódulos podem ajudar a dividir funcionalidades complexas em unidades mais simples.
* **Transpilers para Automação:** Transpilers automatizam tarefas repetitivas, como gerar arquivos baseados em contratos, minimizando a codificação manual e garantindo consistência.

## Melhores Práticas

* **Use Registros Singleton:** Para serviços compartilhados (por exemplo, configuração, repositório), use registros singleton para evitar instanciação repetida e simplificar o acesso.
* **Defina Módulos para Cada Funcionalidade:** Organize sua aplicação definindo um módulo separado para cada funcionalidade ou domínio, facilitando o gerenciamento e a escalabilidade.
* **Contratos e Serviços Automáticos:** Deixe o framework CMMV lidar com a geração de serviços, controladores e gateways baseados em contratos para reduzir o código boilerplate manual.
* **Evite Modificar Arquivos Gerados Automaticamente:** Arquivos como `app.module.ts` são gerados automaticamente e não devem ser editados manualmente. Sempre faça suas modificações em módulos ou serviços separados.

O sistema de módulos do CMMV oferece uma maneira flexível e poderosa de estruturar sua aplicação, permitindo o manejo eficiente de controladores, serviços e transpilers. Ao aproveitar recursos como registros singleton e geração automática de arquivos por meio de contratos, o CMMV ajuda a simplificar o processo de desenvolvimento, reduzir a configuração manual e garantir que a aplicação permaneça escalável e manutenível.
