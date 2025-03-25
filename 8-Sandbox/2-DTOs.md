# DTOs

Além de definir campos para tabelas de banco de dados e operações CRUD, os contratos também suportam a definição de **Objetos de Transferência de Dados (DTOs)**. DTOs são estruturas de dados especializadas usadas para comunicação com serviços, como rotas de API que exigem payloads personalizados não diretamente vinculados à entidade completa do contrato.

- **Exemplo**: O contrato "Usuários" pode definir uma entidade completa com campos como `id`, `nome`, `email` e `senha`, mas para rotas como registro ou login, você pode definir DTOs com um subconjunto de campos (por exemplo, `email` e `senha` para login) adaptados a essas operações específicas.
- **Métodos Personalizados**: DTOs representam as estruturas de dados esperadas para implementar métodos personalizados em controladores, gateways ou resolvedores.

### DTOs no Sandbox

No Sandbox, os DTOs aprimoram a experiência de configuração ao:

- **Configuração Automática de Requisição/Resposta**: O Sandbox compreende essas estruturas de dados e as utiliza para configurar automaticamente os payloads de requisição e resposta para testes e validação.
- **Integração com Módulos**: Módulos como `@cmmv/openapi` aproveitam os DTOs declarados em controladores para gerar documentação de API precisa, garantindo consistência entre as definições do contrato e os endpoints expostos.

Para definir DTOs, você pode estender a configuração do contrato dentro da interface do Sandbox, especificando a estrutura e associando-a a rotas ou métodos específicos.

## Uso no Sandbox

Na interface do Sandbox:

1. **Menu à Esquerda**: Navegue por todos os contratos no seu projeto e módulos instalados.
   - Contratos editáveis (específicos do projeto) permitem configuração completa de campos, relacionamentos e DTOs.
   - Contratos somente leitura (de módulos) exibem sua estrutura e detalhes de implementação.
2. **Configuração de Campos**:
   - Adicione, edite ou remova campos para definir a estrutura de dados do contrato.
   - Configure relacionamentos vinculando a outros contratos.
   - Defina validações para garantir a consistência dos dados.
3. **Configuração de DTOs**:
   - Defina DTOs personalizados para rotas de API ou métodos de serviço específicos.
   - Teste payloads de requisição/resposta baseados nesses DTOs diretamente no Sandbox.
4. **Pré-visualização e Validação**: Use o Sandbox para testar como os campos, relacionamentos e DTOs do seu contrato se traduzem em APIs REST, consultas GraphQL e layouts de formulário, com suporte automático à documentação quando combinado com OpenAPI.
