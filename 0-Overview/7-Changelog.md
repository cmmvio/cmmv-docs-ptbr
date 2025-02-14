# Registro de Alterações

## Versão 0.8.8
24 de Janeiro de 2025

### Correções Críticas

- **Correções na Geração de Entidades em `@cmmv/repository`:**  
  Resolvido problemas relacionados à geração de entidades, abordando especificamente casos que envolvem objetos serializados para garantir definições de esquema corretas e integridade dos dados.

- **Tratamento de Transformação com `toPlain`:**  
  Introduzido o parâmetro `toPlain` para reverter transformações aplicadas via `transform` ao serializar objetos para transmissão via RPC ou REST, garantindo uma representação precisa dos dados.

### Adições

- **Suporte a Índices Personalizados em Contratos:**  
  Adicionada a capacidade de definir índices personalizados diretamente no contrato usando o parâmetro `index`, permitindo índices complexos de múltiplos campos com opções avançadas do TypeORM, como `unique`, `spatial`, `fulltext`, e mais.

- **Indexação Estendida em `@cmmv/repository`:**  
  Atualizado o módulo de repositório para suportar e aplicar corretamente as definições de índices personalizados dos contratos, melhorando o desempenho das consultas e a organização dos dados.

- **Novo Decorador `@ContractMessage`:**  
  Introduzido o decorador `@ContractMessage`, permitindo que os desenvolvedores definam mensagens estruturadas de dados nos contratos, alinhando-se aos conceitos do Protobuf e padronizando DTOs para o tratamento de requisições e respostas.

- **Novo Decorador `@ContractService`:**  
  Adicionado o decorador `@ContractService` para facilitar a definição de funções personalizadas dentro dos contratos, fornecendo uma abordagem estruturada para gerenciar fluxos de requisição/resposta personalizados.

- **Geração Automática de DTOs e Interfaces:**  
  Implementada a geração automática de DTOs e interfaces TypeScript com base nas mensagens dos contratos, simplificando o desenvolvimento e garantindo consistência entre os serviços.

- **Geração de Boilerplate para Funções Personalizadas:**  
  O sistema agora gera código boilerplate para a implementação de funções personalizadas em controladores e gateways, facilitando a integração e reduzindo o esforço manual.

### Melhorias

- **Aprimoramento do Tratamento de Dados em RPC e REST:**  
  A introdução de `toPlain` melhora a serialização de dados, proporcionando flexibilidade no tratamento de transformações e garantindo compatibilidade com diferentes camadas de transporte.

- **Lógica de Indexação Otimizada no Repositório:**  
  Melhorada a lógica de aplicação de índices personalizados no repositório, aumentando o desempenho e reduzindo a sobrecarga potencial em consultas ao banco de dados.

- **Estruturas Padronizadas de Requisição e Resposta:**  
  Com a introdução dos decoradores `@ContractMessage` e `@ContractService`, as aplicações agora possuem uma maneira consistente e sustentável de definir estruturas de requisição e resposta.

### Atualizações

- **Revisão da Documentação:**  
  A documentação foi atualizada para refletir os novos recursos, incluindo exemplos e diretrizes para o uso dos decoradores `@ContractMessage`, `@ContractService` e opções de indexação personalizada.

## Versão 0.8.7
22 de Janeiro de 2025

### Correções Críticas

- **Correções na Identificação de Rotas no `@cmmv/server`**:  
  Resolvido problemas relacionados à identificação incorreta de rotas, garantindo roteamento preciso das requisições e maior confiabilidade.

- **Correção no Middleware Helmet**:  
  Ajustadas inconsistências na configuração do middleware de segurança Helmet, aumentando a segurança geral da aplicação.

- **Correções no Ciclo de Vida da Requisição no `@cmmv/http`**:  
  Problemas no ciclo de vida das requisições foram corrigidos para garantir um fluxo de execução adequado e reduzir possíveis travamentos.

### Adições

- **Pontos de Retorno para Cabeçalhos no `@cmmv/server`**:  
  Introduzidos pontos adicionais de retorno para o processamento de cabeçalhos, evitando operações desnecessárias quando as respostas já foram enviadas.

- **Função de Compilação no `@cmmv/core`**:  
  Adicionada a nova função `compile`, permitindo que os desenvolvedores processem módulos e gerem arquivos transpilados sem precisar executar a aplicação.

### Melhorias

- **Melhorias de Desempenho no `@cmmv/server`**:  
  Componentes do servidor foram otimizados, resultando em tempos de resposta mais rápidos e maior taxa de transferência.

- **Atualização dos Benchmarks**:  
  Os resultados de benchmark foram atualizados para refletir as melhorias de desempenho e otimizações em toda a estrutura do servidor.

- **Correções no Tratamento de Requisições no `@cmmv/http`**:  
  Melhoradas as importações de middlewares e o processamento de requisições para uma maior estabilidade e extensibilidade.

- **`scriptsTimestamp` e Controle de Cache no `@cmmv/view`**:  
  - Adicionada a configuração `scriptsTimestamp` para permitir o gerenciamento de scripts com base em marcação de tempo.
  - Integrado o tempo de modificação do script (mtime) nas requisições de consulta para ignorar o cabeçalho de cache e garantir recursos atualizados.

### Remoções

- **Remoção Completa do Vite do `@cmmv/http`**:  
  O Vite foi totalmente removido do módulo HTTP para simplificar as dependências e aumentar a flexibilidade.

- **Desacoplamento do `@cmmv/view` do `@cmmv/http`**:  
  O módulo `@cmmv/view` agora está totalmente desacoplado, tornando-se um módulo opcional para renderização do lado do servidor, oferecendo mais modularidade.

### Recomendações

- **Atualize para a Versão Mais Recente do Servidor**:  
  Para aproveitar as melhorias de desempenho e correções, atualize para a versão `0.8.7` do `@cmmv/server`.

- **Refatore as Integrações com Vue**:  
  Com o desacoplamento do `@cmmv/view`, ajuste seu projeto caso esteja utilizando Vue para garantir compatibilidade.

## Versão 0.8.0 
10 de Dezembro de 2024

### Correções Críticas

- **Loop Infinito em Rotas Inexistentes**:  
  Resolvido um problema crítico onde rotas inexistentes causavam um loop infinito de requisições, melhorando a estabilidade do servidor.

### Adições

- **Grupos e Acesso Root em Modo Dev**:  
  - Adicionado suporte para grupos de usuários, permitindo controle mais granular sobre permissões e funções.
  - Introduzido suporte ao acesso root no modo de desenvolvimento, simplificando o processo de depuração e testes.

- **Parâmetro Root**:  
  Adicionado o novo recurso `rootParam` para melhorar a flexibilidade e simplificar configurações em operações de nível root.

### Melhorias

- **Atualização do Servidor para a Versão 0.7.1**:  
  Atualizado o núcleo do servidor para a versão `0.7.1`, trazendo otimizações e melhor compatibilidade com os módulos mais recentes.

- **Atualizações no Service**:  
  Melhorias na camada de serviço com melhor tratamento de erros, processamento aprimorado de requisições e suporte para configurações dinâmicas de nível root.

- **Módulo `@cmmv/auth`**:  
  Corrigido o tratamento incorreto do retorno HTTP 401 para garantir um fluxo de autenticação adequado e mensagens de erro precisas.

### Remoções

- **Vue do `@cmmv/view`**:  
  O Vue foi completamente removido do `@cmmv/view`. Agora, os desenvolvedores devem usar o módulo dedicado `@cmmv/vue` para todas as integrações relacionadas ao Vue.

### Recomendações

- **Use o `@cmmv/vue` para Integrações com Vue**:  
  Transicione para o módulo `@cmmv/vue` para suporte completo ao Vue, incluindo mixins e composables para RPC.

- **Atualize para o Servidor 0.7.1**:  
  Certifique-se de atualizar sua aplicação para a versão mais recente do servidor para aproveitar as novas correções e recursos.

## Versão 0.7.5
10 de Dezembro de 2024

### Adições

- **Módulo `@cmmv/testing`:**  
  Introduzido um novo módulo para simplificar o teste de aplicações CMMV, fornecendo utilitários para simular módulos, serviços e contratos.

- **Validador de Configuração:**  
  Adicionado um sistema de validação para configurações da aplicação, garantindo consistência e correção em todos os ambientes.

- **Esquema para Configurações:**  
  Introduzidas definições de esquema para validar e estruturar configurações de forma padronizada.

### Melhorias

- **Refatoração dos Módulos Centrais:**  
  Simplificados os módulos centrais para melhorar a manutenibilidade, reduzindo a complexidade enquanto mantém todas as funcionalidades essenciais.

- **Paginação, Ordenação e Classificação:**  
  Adicionado suporte nativo para paginação, ordenação e classificação em rotas GET e gateways, simplificando a recuperação de dados e o design da API.

- **Limpeza de Dependências:**  
  Removidas dependências não utilizadas e funções legadas, reduzindo o impacto do framework e melhorando a eficiência geral.

- **Adaptadores HTTP Modulares:**  
  Os módulos Express e Fastify, anteriormente integrados ao `@cmmv/http`, foram desacoplados em módulos independentes:  
  - `@cmmv/express`  
  - `@cmmv/fastify`  
  Essa mudança reduz o impacto das dependências no núcleo, permitindo que os desenvolvedores escolham e instalem apenas o adaptador HTTP necessário.

### Remoções

- **Integração com Vue e TailwindCSS:**  
  Vue e TailwindCSS foram removidos do módulo central. Use o Vite para gerenciamento de ativos frontend e o módulo `@cmmv/vue` para integrações específicas do Vue.

### Recomendações

- **Integração com Vite:**  
  Utilize o Vite para fluxos de trabalho modernos de desenvolvimento frontend, incluindo Vue e TailwindCSS.

- **Módulo `@cmmv/vue`:**  
  Utilize o módulo `@cmmv/vue` para gerar mixins RPC e composables para uma integração perfeita com o Vue.

## Versão 0.6.0
26 de Novembro de 2024

### Novos Recursos

- **Módulo Inspector:**  
  Adicionado o módulo `@cmmv/inspector` para depuração e monitoramento de aplicações CMMV. O módulo fornece informações em tempo de execução, perfis de desempenho e análise de gargalos críticos.
  
### Melhorias

- **Otimização de Desempenho:**  
  Usando insights do perfil do módulo Inspector, funções críticas do lado do servidor foram otimizadas. Isso resultou em um aumento de desempenho de uma média de 71.1k para 79k requisições por segundo em testes de benchmark, uma melhoria de aproximadamente 8-10% nos tempos de resposta.
- **Atualização de Documentação:**  
  Documentação atualizada para incluir o novo módulo Inspector e seu uso no ecossistema.
- **Revisão de Benchmark:**  
  Benchmarks refeitos e documentados com o servidor otimizado, refletindo as métricas de desempenho aprimoradas.

### Alterações

- **Remoção do `Fast-JSON-Stringify` dos Exemplos:**  
  Removido `fast-json-stringify` dos exemplos, já que o esquema precisa ser processado durante o início da aplicação. Esse recurso será revisado e potencialmente otimizado em versões futuras.

### Notas para Versões Futuras

- Planeja-se integrar o processamento de esquema durante o início da aplicação para melhorar a usabilidade do `fast-json-stringify` sem degradação de desempenho.

## Versão 0.5.5 
16 de Novembro de 2024

### Novos Recursos

- **Integração com Middleware Vite:**  
  Adicionado Vite como middleware para servir arquivos estáticos durante o desenvolvimento, melhorando a integração com Vue 3 e a experiência de desenvolvimento.

- **Mixins RPC para Vue 3 e Nuxt:**  
  Introduzidos mixins RPC gerados a partir de contratos CMMV para integração perfeita com Vue 3 e Nuxt.

- **Implementação Completa de RPC:**  
  Implementado suporte completo a RPC para Vue e Nuxt, permitindo comunicação dinâmica cliente-servidor baseada em contratos.

- **Suporte a Importação de Vue 3 + TailwindCSS:**  
  Aprimorado o módulo `view` para suportar importações de Vue 3 e TailwindCSS para fluxos de trabalho modernos de desenvolvimento frontend.

### Melhorias

- **Fast-JSON-Stringify:**  
  Substituída a serialização JSON padrão por `fast-json-stringify`, melhorando significativamente o desempenho da serialização.

- **Mudança para Vitest:**  
  Migrado de Mocha para Vitest para testes mais rápidos, modernos e com melhor integração ao TypeScript.

- **Exemplo de Aplicativo de Lista de Tarefas:**  
  Adicionado um exemplo funcional de lista de tarefas mostrando Vue 3, Vite e mixins RPC gerados pelo CMMV.

### Correções de Bugs

- **Correções no Repositório MongoDB:**  
  Resolvidos bugs críticos no módulo `repository` para MongoDB, garantindo interações estáveis com o banco de dados.

- **Atualizações de Dependências Obsoletas:**  
  Atualizadas dependências obsoletas em todo o framework, melhorando a compatibilidade e removendo avisos de segurança.

- **Aprimoramentos no Módulo View:**  
  Corrigidos problemas com importações estáticas no módulo `view` para melhor manipulação de ativos Vue 3 e TailwindCSS.

### Atualizações de Dependências
- Dependências atualizadas em todo o framework, garantindo compatibilidade com as versões mais recentes do ecossistema e ferramentas.

## Versão 0.5.0 
5 de Novembro de 2024

### Adicionado

- **Suporte ao Vue 3:**  
  Implementação completa do Vue 3, configurável via `.cmmv.config.cjs`, permitindo o uso de sintaxe do Vue 3. Configurações de SSR integradas para desempenho otimizado.

- **Carregamento Dinâmico de Layouts e Views:**  
  Adicionado suporte ao carregamento dinâmico de layouts e views, incluindo integração com Vite para servir eficientemente arquivos `.vue`.

- **Vinculação de Dados e Configuração de Setup:**  
  Introduzido `s-setup` para configurar scripts e dados reativos em views, com suporte a hooks de ciclo de vida como `mounted` e `created`.

- **Novos Hooks de Ciclo de Vida:**  
  Adicionados hooks `mounted` e `created` em scripts de configuração de views, proporcionando flexibilidade para executar código quando a view for carregada.

- **Diretiva Include:**  
  Uma nova diretiva para importar e reutilizar componentes de view modularmente em diferentes páginas.

- **Serviço de Telemetria:**  
  Adicionada a classe `Telemetry` para monitorar desempenho e capturar métricas de tempo de execução em todo o sistema.

- **Injeção de Métodos:**  
  Funções RPC agora são automaticamente injetadas como métodos em componentes Vue, simplificando chamadas de funções remotas no frontend.

### Alterado

- **Renderização de Views Aprimorada:**  
  Sistema de renderização de views otimizado com minificação de HTML e extração de scripts inline para melhorar o desempenho.

- **Manipulação de Erros Aprimorada:**  
  Melhorada a manipulação de erros para garantir o registro adequado de falhas com mensagens de erro mais informativas.

## Versão 0.4.0
15 de Outubro de 2024

### Adicionado

- **Suporte a RPC com Protobuf:**  
  Integração para Protobuf, permitindo comunicação binária via WebSocket, aprimorando as capacidades em tempo real.

- **Adaptador WebSocket:**  
  Adicionado suporte a WebSocket com um adaptador dedicado para comunicação em tempo real, incluindo geração de RPC baseada em contratos.

- **Módulo de Cache:**  
  Integrado o módulo de cache (`@cmmv/cache`) para armazenamento de dados de alto desempenho, com suporte a Redis e opções em memória.

- **Módulo de Repositório:**  
  Adicionado suporte para gerenciamento de banco de dados baseado em repositórios com integração para TypeORM e MongoDB.

- **Geração Automática Baseada em Contratos:**  
  Controladores, entidades e serviços agora são gerados automaticamente a partir de contratos TypeScript, reduzindo código repetitivo.

- **Configuração Dinâmica:**  
  Suporte adicionado para configuração dinâmica usando `.cmmv.config.cjs`, permitindo ajustes em tempo de execução sem modificar o código-fonte.

### Alterado

- **CLI Melhorada:**  
  CLI aprimorada para simplificar a inicialização de projetos, com configurações personalizáveis para RPC, cache e repositórios.

- **Estrutura de Diretórios Refinada:**  
  Estrutura de projetos revisada para melhor organização, separando ativos públicos, views e templates.

- **Integração de Módulos Aprimorada:**  
  Integração refinada de módulos como `@cmmv/repository` e `@cmmv/view` para garantir compatibilidade perfeita com componentes gerados automaticamente.

### Corrigido

- Resolvemos problemas de validação de contratos durante a inicialização da aplicação.
- Corrigidos bugs de sincronização de banco de dados para configurações SQLite e MongoDB.

## Versão 0.3.5 
28 de Setembro de 2024

### Recursos

- **Gerador de Contratos:**  
  Automatiza a criação de contratos entre servidor e cliente, garantindo interfaces de API consistentes e reduzindo esforços manuais.

- **Suporte Aprimorado ao WebSocket:**  
  Integração de WebSocket para comunicações RPC em tempo real usando Protobuf, otimizando a eficiência e confiabilidade da transmissão de dados.

### Melhorias

- **Otimização de Desempenho:**  
  Melhorias significativas no tempo de processamento para geração e execução de contratos, reduzindo a latência em cenários de alta carga.

- **Design Modular:**  
  Reestruturado o framework principal para suportar melhor isolamento de módulos, permitindo extensões ou modificações mais fáceis sem impactar o sistema como um todo.

- **Aprimoramentos de Escalabilidade:**  
  Ajustes finos no sistema principal para lidar com maior tráfego e cargas de dados maiores com impacto mínimo no desempenho.

### Correções de Bugs

- **Correções de Lint:**  
  Solucionados todos os problemas de linting em todo o código-base para garantir um código limpo e fácil de manter.

- **Correções de Testes:**  
  Resolvidos diversos problemas para melhorar a confiabilidade dos testes unitários e de integração.

- **Correção de Importação de WebSocket:**  
  Corrigidos problemas de importação no módulo de WebSocket para garantir inicialização e funcionalidade adequadas.

- **Configuração do CircleCI:**  
  Integrado e corrigido problemas no CircleCI para pipelines contínuos de integração e implantação.

- **Correções Diversas:**  
  Várias pequenas correções de bugs e melhorias no código em todo o framework.

### Documentação e Exemplos

- **Configuração Avançada de Contratos:**  
  Documentação atualizada para configuração avançada de contratos e geração de APIs, fornecendo exemplos mais claros e detalhados para os desenvolvedores.

### Testes e Qualidade

- **Cobertura de Testes Expandida:**  
  Adicionados testes unitários para módulos críticos, melhorando a estabilidade e confiabilidade geral do framework.

- **Testes de Estresse:**  
  Simulados ambientes de tráfego intenso para validar a robustez da comunicação em tempo real e garantir escalabilidade sob carga.
