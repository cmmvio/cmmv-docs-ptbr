# Sandbox

O módulo `@cmmv/sandbox` é um sistema de suporte visual introduzido na versão 0.9.11 do CMMV. Ele foi projetado para simplificar o gerenciamento de contratos e demonstrar todo o potencial de uma abordagem *contract-first* em sua aplicação. Com o Sandbox, você pode configurar o sistema, criar e editar contratos, validar implementações REST e GraphQL, criar backups da aplicação, acessar logs, instalar módulos do CMMV e muito mais. Além disso, o módulo inclui uma versão inicial de um *form builder* que permite personalizar telas de formulário derivadas de contratos para uso em seus dashboards.

O Sandbox tem como objetivo otimizar a administração de contratos enquanto demonstra as capacidades abrangentes do CMMV por meio de uma interface prática e acessível. Para usá-lo, basta instalá-lo em seu projeto e incluí-lo na configuração da sua aplicação.

<div class="relative overflow-hidden pt-2 mb-4">
    <div class="container mx-auto px-4">
        <div class="max-w-5xl mx-auto">
            <img src="/assets/publish-example-dark.png" alt="Visão Geral do Framework CMMV" class="rounded-xl shadow-2xl ring-1 ring-white/20 w-full">
            <div class="relative" aria-hidden="true">
                <div class="absolute -inset-x-20 bottom-0 bg-gradient-to-t from-neutral-900 pt-[7%]"></div>
            </div>
        </div>
    </div>
</div>

## Instalação

Para instalar o `@cmmv/sandbox` e suas dependências necessárias, execute:

```bash
$ pnpm add @cmmv/sandbox @cmmv/repository @cmmv/http @cmmv/graphql @cmmv/auth
```

O Sandbox depende dos módulos `@cmmv/repository`, `@cmmv/http`, `@cmmv/graphql` e `@cmmv/auth` para funcionar corretamente. Certifique-se de que todos estejam incluídos em seu projeto.

## Configuração

Para habilitar o suporte ao Sandbox em sua aplicação CMMV, adicione o `SandboxModule` à configuração da sua aplicação:

```typescript
import { Application } from '@cmmv/core';
import { DefaultAdapter, DefaultHTTPModule } from '@cmmv/http';
import { RepositoryModule, Repository } from '@cmmv/repository';
import { AuthModule } from '@cmmv/auth';
import { GraphQLModule } from '@cmmv/graphql';
import { SandboxModule } from '@cmmv/sandbox';

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [
        DefaultHTTPModule,
        RepositoryModule,
        AuthModule,
        GraphQLModule,
        SandboxModule,
        // Outros módulos, se necessário
    ],
    providers: [Repository],
});
```

Após a configuração, o Sandbox estará disponível em sua aplicação, fornecendo acesso às suas funcionalidades por meio de uma interface visual integrada.

## Funcionalidades

O módulo `@cmmv/sandbox` oferece as seguintes funcionalidades principais:

- **Configuração do Sistema**: Ajuste as configurações gerais da aplicação diretamente pela interface.
- **Gerenciamento de Contratos**: Crie, edite e visualize contratos de forma intuitiva.
- **Validação de Implementação**: Teste e valide endpoints REST e GraphQL gerados a partir de contratos.
- **Backups**: Crie backups completos da aplicação para segurança e recuperação.
- **Logs**: Acesse logs detalhados do sistema para monitoramento e depuração.
- **Instalação de Módulos**: Instale módulos adicionais do CMMV diretamente pelo Sandbox.
- **Form Builder**: Personalize formulários derivados de contratos para uso em dashboards (versão inicial).

O Sandbox integra-se ao ecossistema CMMV, aproveitando os módulos existentes para fornecer uma experiência unificada e eficiente.

## Uso

Após a instalação e configuração, inicie sua aplicação CMMV. Acesse a rota do Sandbox (padrão: `/sandbox`) em seu navegador para explorar a interface visual. A partir daí, você pode:

1. Configurar o sistema e visualizar contratos existentes.
2. Criar ou editar contratos e testar suas implementações REST e GraphQL.
3. Usar o *form builder* para personalizar formulários baseados em contratos.
4. Gerenciar backups e revisar logs diretamente na interface.

O Sandbox é projetado para ser uma ferramenta *all-in-one* para desenvolvedores que usam o CMMV, oferecendo uma maneira prática de gerenciar e expandir suas aplicações.