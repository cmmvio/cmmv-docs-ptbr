# Contratos

A seção Contratos no `@cmmv/sandbox` oferece uma interface poderosa para gerenciar e inspecionar contratos dentro do seu projeto CMMV. Os contratos são centrais para a filosofia *contract-first* do CMMV, definindo a estrutura, o comportamento e os relacionamentos dos dados e APIs da sua aplicação.

No menu à esquerda do Sandbox, você encontrará uma lista de todos os contratos existentes no seu projeto, bem como contratos de módulos instalados. Contratos específicos do projeto são totalmente editáveis, enquanto os contratos fornecidos por módulos estão disponíveis no modo **somente leitura**. Isso permite que você verifique sua implementação sem modificar seu comportamento.

## Configuração de Contratos

Os contratos no CMMV oferecem uma ampla gama de opções de configuração, permitindo um controle refinado sobre seu comportamento e integração no sistema. Isso inclui:

- **Rota do Controlador**: Define a rota base para os endpoints REST gerados a partir do contrato.
- **Opções de RPC**: Configura as definições de Chamada de Procedimento Remoto para os métodos do contrato.
- **Subdiretórios e Namespaces**: Organiza os contratos em subdiretórios lógicos ou namespaces para uma melhor estrutura.
- **Dados Específicos de Banco de Dados**: Especifica detalhes relacionados ao banco de dados, como:
  - Nome da tabela ou coleção.
  - Suporte a exclusão suave (por exemplo, marcar registros como excluídos sem removê-los).
  - Outras configurações específicas do banco de dados.

Essas configurações permitem que os contratos se adaptem a vários casos de uso, desde a geração de APIs até interações com o banco de dados.

## Aba Campos

A primeira aba disponível na interface do contrato é **Campos**, onde você pode definir a estrutura central do contrato. Isso inclui:

- **Definições de Campos**: Estabelece a estrutura básica dos campos suportados (por exemplo, nome, tipo, restrições).
- **Relacionamentos**: Estabelece conexões entre contratos (por exemplo, um-para-muitos, muitos-para-um).
- **Validações**: Aplica regras usando a biblioteca `class-validator` para garantir a integridade dos dados.

Os campos definidos aqui servem como base para gerar interfaces e modelos genéricos em todo o sistema. Eles são automaticamente equipados com validações do `class-validator`, e futuras atualizações integrarão transformações de dados via `class-transformer`.

### Relacionamentos em Contratos

Uma característica chave dos contratos do CMMV é a abstração de relacionamentos, que é gerenciada independentemente do banco de dados subjacente. Essa abstração vem antes de qualquer mapeamento direto de relacionamento no banco de dados, tornando-a particularmente útil para bancos de dados como MongoDB, onde relacionamentos nativos não são predefinidos.

- **Abstração de Relacionamento**: Define como os contratos se relacionam entre si (por exemplo, um contrato "Usuário" vinculado a um contrato "Postagem").
- **Carregamento de Dados**: O CMMV gerencia esses relacionamentos e garante que os dados dependentes sejam carregados corretamente em:
  - **Respostas da API**: Endpoints REST retornam dados relacionados conforme especificado.
  - **Resolvedores GraphQL**: Resolvedores gerados automaticamente buscam e resolvem dados relacionados.
  - **Form Builder**: Formulários refletem relacionamentos para edição e visualização contínuas.

Isso significa que você pode criar uma abstração completa de relacionamento mesmo ao usar MongoDB, com o CMMV lidando com a lógica para buscar e apresentar dados dependentes de forma precisa.

## Uso no Sandbox

Na interface do Sandbox:

1. **Menu à Esquerda**: Navegue por todos os contratos no seu projeto e módulos instalados.
   - Contratos editáveis (específicos do projeto) permitem configuração completa.
   - Contratos somente leitura (de módulos) exibem sua estrutura e detalhes de implementação.
2. **Configuração de Campos**:
   - Adicione, edite ou remova campos para definir a estrutura de dados do contrato.
   - Configure relacionamentos vinculando a outros contratos.
   - Defina validações para garantir a consistência dos dados.
3. **Pré-visualização e Validação**: Use o Sandbox para testar como os campos e relacionamentos do seu contrato se traduzem em APIs REST, consultas GraphQL e layouts de formulário.

<div style="
    background-color: #DBEAFE;
    border-left: 4px solid #3B82F6;
    color: #1E40AF;
    padding: 1rem;
    border-radius: 0.375rem;
    margin: 1.5rem 0;
    font-size: 12px;
">
    <p style="font-weight: bold; margin-bottom: 0.5rem;">Aviso Importante</p>
    <p>
        A abstração de relacionamento atualmente suporta configurações básicas (por exemplo, um-para-muitos, muitos-para-um). Recursos avançados como exclusões em cascata ou junções complexas estão planejados para versões futuras. Da mesma forma, a integração do <strong>class-transformer</strong> para transformação de dados ainda não está disponível, mas aprimorará o processamento de campos em versões futuras.
    </p>
</div>

## Benefícios

- **Flexibilidade**: Defina contratos adaptados às necessidades da sua aplicação, independentemente do backend do banco de dados.
- **Consistência**: Garanta estruturas de dados e validações uniformes em REST, GraphQL e formulários.
- **Escalabilidade**: Use namespaces e subdiretórios para organizar contratos à medida que seu projeto cresce.

A seção Contratos no Sandbox capacita você a aproveitar ao máximo as capacidades *contract-first* do CMMV, fornecendo uma maneira visual e interativa de projetar, gerenciar e validar os componentes centrais da sua aplicação.
