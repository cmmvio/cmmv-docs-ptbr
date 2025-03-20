# Resolvers

O módulo `@cmmv/graphql` gera automaticamente resolvers GraphQL com base nos contratos do sistema. Isso permite que os desenvolvedores se concentrem em definir contratos enquanto o sistema cuida da geração dos tipos e resolvers necessários.

Além disso, o `@cmmv/graphql` integra-se perfeitamente ao `@cmmv/auth`, habilitando mecanismos de autenticação e autorização dentro dos resolvers. No entanto, os desenvolvedores também podem definir **resolvers personalizados** e adicioná-los manualmente à configuração do módulo.

## Geração Automática de Resolvers

Quando um contrato é criado no CMMV, o sistema gera automaticamente um resolver correspondente. Abaixo está um exemplo de um resolver **gerado automaticamente** para a entidade `User`:

```typescript
/**
    **********************************************
    Este script foi gerado automaticamente pelo CMMV.
    Não é recomendado modificar este arquivo manualmente,
    pois ele pode ser sobrescrito pela aplicação.
    **********************************************
**/

import {
    Resolver, Query, Mutation,
    Authorized, Arg, Args,
    ID, Int, Float, Ctx,
    Field, ArgsType, ObjectType,
    PaginationArgs, GraphQLContext,
    PaginationResponse
} from "@cmmv/graphql";

import {
    IUser, User
} from "@models/auth/user.model";

import {
   UserService
} from "@services/auth/user.service";

@ArgsType()
class CreateUserInput {
    @Field(() => String, { nullable: false })
    username: String;

    @Field(() => String, { nullable: false })
    password: String;

    @Field(() => String, { nullable: true, defaultValue: "'{}'" })
    profile?: String;
}

@ObjectType()
class PaginationUserReturn {
    @Field(() => Int, { description: "Total de registros disponíveis.", nullable: false })
    count!: number;

    @Field(() => [User], { description: "Lista de registros de usuários.", nullable: false })
    data?: User[];

    @Field(() => PaginationResponse, { description: "Metadados de paginação.", nullable: false })
    pagination!: PaginationResponse;
}

@Resolver(of => User)
export class UserResolverGenerated {
    private readonly userservice: UserService;

    constructor() {
        this.userservice = new UserService();
    }

    @Query(returns => PaginationUserReturn)
    @Authorized({ rootOnly: true })
    async userFind(@Args() queries: PaginationArgs, @Ctx() ctx: GraphQLContext) {
        return await this.userservice.getAll(queries, ctx.req);
    }

    @Query(returns => User)
    @Authorized({ rootOnly: true })
    async userById(@Arg("id") id: string) {
        return (await this.userservice.getById(id)).data;
    }

    @Mutation(returns => User)
    @Authorized({ rootOnly: true })
    async createUser(@Args() createUserData: CreateUserInput): Promise<User> {
        return (await this.userservice.insert(createUserData as Partial<User>)).data;
    }

    @Mutation(returns => Boolean)
    @Authorized({ rootOnly: true })
    async deleteUser(@Arg('id') id: string): Promise<boolean> {
        return (await this.userservice.delete(id)).success;
    }
}
```
<br/>

* **Criação Automática de Resolvers Baseada em Contratos:** Quando um contrato é definido no CMMV, o sistema gera automaticamente resolvers com base em sua estrutura.
* **Suporte a Paginação:** O CMMV fornece suporte integrado à paginação por meio de `PaginationArgs` e `PaginationResponse`.
* **Suporte ao Contexto GraphQL:** Lógicas personalizadas de autenticação e autorização podem ser aplicadas acessando o `GraphQLContext`.
* **Autorização Integrada:** Usa `@Authorized({ rootOnly: true })` para restringir o acesso a usuários autenticados.

## Resolvers Personalizados

Embora o CMMV gere automaticamente a maioria dos resolvers necessários, os desenvolvedores podem criar resolvers personalizados quando necessário e adicioná-los ao sistema.

Para registrar um resolver personalizado, modifique a configuração do `Module`:

```typescript
import { Module } from '@cmmv/core';

import { MyConfig } from './my.config';
import { MyService } from './my.service';
import { MyController } from './my.controller';
import { MyTranspile } from './my.transpiler';
import { MyResolver } from './my.resolver';

export const MyModule = new Module('mymodule', {
    configs: [MyConfig],
    controllers: [MyController],
    providers: [MyService],
    transpilers: [MyTranspile],
    resolvers: [MyResolver],
});
```
<br/>

* Os desenvolvedores podem definir seus próprios resolvers de consulta e mutação para expandir a funcionalidade existente.
* Os resolvers podem ser injetados dinamicamente, permitindo uma expansão modular.
* A autorização é totalmente personalizável, integrando-se ao `@cmmv/auth`.

Os resolvers no `@cmmv/graphql` são gerados automaticamente a partir de contratos, garantindo uma integração suave com GraphQL enquanto impõem um esquema consistente.
Para casos de uso avançados, os desenvolvedores podem definir resolvers personalizados e incluí-los na configuração dos Módulos.

Atualizações futuras introduzirão a integração do middleware Apollo com `@cmmv/http`, permitindo a coexistência perfeita entre GraphQL e API REST.

## Decoradores TypeGraphQL Embutidos

O módulo `@cmmv/graphql` integra-se completamente ao `type-graphql`, incorporando seus principais decoradores e funções.
Isso significa que **você não precisa instalar ou importar manualmente o `type-graphql` no seu projeto** — tudo está disponível diretamente no `@cmmv/graphql`.

Essa abordagem garante uma **configuração simplificada**, melhora a **experiência do desenvolvedor** e proporciona uma **melhor compatibilidade** dentro do ecossistema CMMV.

🔗 **Documentação Oficial do TypeGraphQL**: [https://typegraphql.com/docs/introduction.html](https://typegraphql.com/docs/introduction.html)

A tabela a seguir lista todos os decoradores e funções embutidos do `type-graphql`, junto com suas finalidades:

| Decorador/Função   | Descrição                                                                 |
|---------------------|---------------------------------------------------------------------------|
| `@ObjectType()`    | Define um tipo de objeto GraphQL, usado para definir esquemas.            |
| `@Field()`         | Declara um campo dentro de um tipo de objeto, tipo de entrada ou classe de argumentos. |
| `@Resolver()`      | Marca uma classe como um resolver GraphQL para um tipo de objeto específico. |
| `@Query()`         | Define um campo de consulta GraphQL que busca dados.                      |
| `@Mutation()`      | Define um campo de mutação GraphQL para modificar dados.                  |
| `@Authorized()`    | Restringe o acesso a uma consulta/mutação com base em regras de autenticação. |
| `@InputType()`     | Define um tipo de entrada GraphQL, usado principalmente para passar argumentos estruturados. |
| `@Arg()`           | Declara um argumento para uma consulta ou mutação.                        |
| `@Args()`          | Agrupa múltiplos argumentos em um único objeto.                           |
| `@ArgsType()`      | Define uma classe que encapsula múltiplos argumentos como um objeto.      |
| `ArgOptions`       | Opções de configuração para o decorador `@Arg()`, como valores nulos e padrão. |
| `buildSchema()`    | Gera assincronamente um esquema GraphQL a partir de resolvers.            |
| `buildSchemaSync()`| Gera sincronicamente um esquema GraphQL a partir de resolvers.            |
| `@Ctx()`           | Fornece acesso ao contexto de execução GraphQL, útil para autenticação.   |
| `ID`               | Define um campo de tipo `ID` no esquema GraphQL, geralmente usado para identificadores únicos. |
| `Int`              | Declara um tipo inteiro no esquema GraphQL.                               |
| `Float`            | Declara um tipo de ponto flutuante no esquema GraphQL.                    |

### Uso em `@cmmv/graphql`

Como todos os decoradores e utilitários necessários estão **pré-empacotados** no `@cmmv/graphql`, você pode **usá-los diretamente** nos seus resolvers sem precisar importar nada do `type-graphql`.

```typescript
import {
    Resolver, Query, ObjectType,
    Field
} from "@cmmv/graphql";

@ObjectType()
class User {
    @Field() id: string;
    @Field() username: string;
}

@Resolver()
class UserResolver {
    @Query(() => User)
    getUser(): User {
        return { id: "1", username: "cmmv-user" };
    }
}
```
<br/>

* Sem dependências extras: O `type-graphql` está embutido no `@cmmv/graphql`, então você não precisa instalá-lo separadamente.
* Importações simplificadas: Tudo o que você precisa está disponível em `@cmmv/graphql`, reduzindo a bagunça.
* Totalmente compatível: Funciona perfeitamente com `@cmmv/auth` e outros módulos.
