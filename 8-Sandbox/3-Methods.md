# Métodos

A seção Métodos no `@cmmv/sandbox` permite que você gerencie e defina as operações disponíveis para cada contrato em seu projeto CMMV. Os métodos representam as capacidades funcionais de um contrato, variando de operações CRUD padrão a lógica personalizada adaptada às necessidades da sua aplicação.

Por padrão, se a opção **Gerar Controlador** estiver ativada em um contrato, funções CRUD básicas (Criar, Ler, Atualizar, Excluir) são geradas automaticamente com base na estrutura de dados definida na aba **Campos**. A interface de Métodos se baseia nisso, apresentando esses métodos padrão e oferecendo a capacidade de criar métodos personalizados para expandir a funcionalidade.

## Métodos Padrão

Quando a opção **Gerar Controlador** está ativada, os seguintes métodos CRUD padrão são criados para o contrato:

- **Criar**: Adiciona um novo registro com base nos campos do contrato.
- **Ler**: Recupera um ou mais registros, com suporte para filtragem e paginação.
- **Atualizar**: Modifica um registro existente.
- **Excluir**: Remove um registro (ou o marca como excluído se a exclusão suave estiver habilitada).

Esses métodos são automaticamente conectados à camada de serviço, aproveitando a estrutura definida nos campos do contrato, e estão acessíveis por meio de endpoints REST e GraphQL.

## Métodos Personalizados

Além das operações CRUD padrão, você pode definir **métodos personalizados** para implementar lógica de negócios específica. Embora a declaração desses métodos seja feita dentro do Sandbox, sua lógica real exige implementação manual no código.

### Declaração de Métodos

Quando você adiciona um método personalizado no Sandbox:
- O método é registrado no serviço do contrato com uma implementação de espaço reservado: `throw new Error('Não Implementado')`.
- Um arquivo de serviço público é gerado no diretório `/src`, que você pode editar para substituir a implementação padrão.

### Implementação

Para implementar um método personalizado:
1. Localize o arquivo de serviço gerado em `/src` (por exemplo, `UserService.ts` para um contrato "Usuários").
2. Substitua o método na classe de serviço, que tem acesso a todos os provedores por meio de injeção de dependência no construtor (por exemplo, `Repository`, `DatabaseConnection`).
3. Concentre-se apenas em implementar a lógica correlata, pois a assinatura do método e as dependências são pré-configuradas.

Por exemplo:
```typescript
import { Service } from '@cmmv/core';
import { Repository } from '@cmmv/repository';

import {
    RegisterUserDto, UserResponseDto
} from "@models/user";

@Service()
export class UserService {
    constructor(private repository: Repository) {}

    // Substituir método personalizado
    async registerUser(payload: RegisterUserDto): Promise<UserResponseDto> {
        const user = await this.repository.create('User', payload);
        return { id: user.id, email: user.email };
    }
}
```

### DTOs para Métodos Personalizados

Criar um método personalizado exige a definição de **Objetos de Transferência de Dados (DTOs)** tanto para o payload da requisição quanto para a resposta:
- **DTO de Requisição**: Especifica a estrutura de entrada esperada (por exemplo, `RegisterUserDto` com `email` e `senha`).
- **DTO de Resposta**: Define a estrutura de saída (por exemplo, `UserResponseDto` com `id` e `email`).

Uma vez declarados:
- Esses DTOs são automaticamente adicionados aos modelos do projeto.
- O Sandbox os reconhece para fins de teste.
- Eles ganham suporte passivo para implementação em **REST**, **GraphQL** e **RPC**.

## Uso no Sandbox

Na interface do Sandbox, a seção Métodos oferece:

1. **Lista de Métodos**: Exibe todos os métodos CRUD padrão (se gerados) e quaisquer métodos personalizados que você definiu.
2. **Criação de Método Personalizado**:
   - Adicione um novo método especificando seu nome, DTO de requisição e DTO de resposta.
   - O Sandbox gera o esboço do serviço com um erro "Não Implementado".
3. **Teste**:
   - Teste métodos padrão e personalizados diretamente por meio de endpoints REST ou GraphQL.
   - Use o Sandbox para simular requisições com os DTOs definidos e inspecionar as respostas.
4. **Visibilidade**: Todos os métodos (padrão e personalizados) são visíveis e testáveis, com seus DTOs refletidos na interface.

Por exemplo, após definir um método `registerUser`:
- Teste-o via REST em `POST /users/register` com o DTO de requisição.
- Consulte-o em GraphQL com o resolvedor gerado.
- Verifique se a resposta corresponde ao DTO de resposta definido.

## Benefícios de Integração

- **Camada de Serviço**: Métodos personalizados são pré-esboçados no serviço, reduzindo o tempo de configuração e permitindo que você se concentre na lógica.
- **Injeção de Dependência**: Acesse todos os provedores (por exemplo, banco de dados, autenticação) sem configuração adicional.
- **Suporte Multi-Protocolo**: Os métodos são automaticamente compatíveis com REST, GraphQL e RPC uma vez implementados.
- **Documentação**: Módulos como `@cmmv/openapi` usam declarações de métodos e DTOs para gerar documentação de API precisa.

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
        Embora as declarações de métodos e DTOs sejam suportadas no Sandbox, a implementação da lógica real ainda exige codificação manual. Atualizações futuras visam introduzir construtores visuais de lógica ou geração de código para simplificar esse processo. A geração atual de resolvedores GraphQL para métodos personalizados é básica e pode exigir ajustes manuais para casos de uso complexos.
    </p>
</div>

## Benefícios

- **Eficiência**: Métodos CRUD pré-gerados e esboços de serviço aceleram o desenvolvimento.
- **Flexibilidade**: Métodos personalizados permitem funcionalidades sob medida com mínimo boilerplate.
- **Testabilidade**: A integração com o Sandbox facilita a validação do comportamento dos métodos em vários protocolos.

A seção Métodos no Sandbox aprimora sua capacidade de definir e testar operações de contrato, conectando o design do contrato à implementação prática em sua aplicação CMMV.
