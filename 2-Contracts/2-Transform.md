# Transformação

No CMMV, você pode **transformar** dados dentro dos contratos usando o parâmetro `transform` no decorador de campo. O sistema utiliza a biblioteca `class-transformer` ([NPM](https://www.npmjs.com/package/class-transformer)) para lidar com essas transformações. Isso é particularmente útil para converter dados de entrada no formato apropriado para classes de entidade e modelo, necessárias para interações com repositórios, entre outras funcionalidades.

* **`exclude:`** Remove o campo durante a serialização ou desserialização.
* **`toClassOnly:`** Garante que a transformação seja aplicada apenas ao converter para uma instância de classe (útil para integridade de dados).
* **`transform:`** Permite lógica de transformação personalizada, como criptografar, aplicar hash, adicionar metadados ou converter datas.

Essas transformações possibilitam o tratamento eficiente de formatos de dados complexos, como aplicar hash em senhas, gerar timestamps e converter ou formatar dados de entrada/saída. Veja abaixo alguns exemplos:

```typescript
import * as crypto from 'crypto';
import { AbstractContract, Contract, ContractField } from '@cmmv/core';

@Contract({
    controllerName: 'User',
    protoPath: 'src/protos/user.proto',
    protoPackage: 'user',
})
export class UsersContract extends AbstractContract {
    @ContractField({
        protoType: 'string',
        validations: ["IsString"],
        // Exemplo de transformação simples
        transform: ({ value }) => value.toUpperCase(),  
    })
    name: string;

    @ContractField({
        protoType: 'string',
        // Este campo será excluído na saída
        exclude: false,  
        // Será processado apenas ao converter para uma classe
        toClassOnly: true,  
        transform: ({ value }) => 
            crypto.createHash('sha256')
            .update(value).digest('hex'),  // Aplica hash ao valor
    })
    password: string;
}
```

<br/>

* **Segurança de Dados:** Você pode facilmente criptografar ou aplicar hash a campos sensíveis, como senhas.
* **Formatação de Dados:** Formate automaticamente datas, strings ou números antes de armazená-los em um repositório.
* **Flexibilidade:** Você tem controle total sobre como os dados de entrada são transformados, garantindo compatibilidade com serviços de backend e bancos de dados.

Essas transformações asseguram que o contrato e a estrutura de dados permaneçam flexíveis, ao mesmo tempo que permitem controle sobre como os dados são processados durante o ciclo de vida de requisição-resposta.

## Dados de Entrada

Suponha que você tenha o seguinte objeto JavaScript como dados de entrada:

```json
{
    "name": "john doe",
    "password": "mypassword"
}
```

Veja o que acontece quando `plainToClass(UsersContract, input)` é aplicado:

```typescript
{
    // Transformado para letras maiúsculas, conforme especificado
    name: "JOHN DOE",
    // Não incluído na saída devido às opções `exclude` e `toClassOnly`   
    password: undefined 
}
```

<br/>

* **Transformação do Nome:**
  * O campo `name` foi convertido para letras maiúsculas devido à função `transform` aplicada: `value.toUpperCase()`.
* **Transformação da Senha:**
  * O campo `password` foi aplicado hash com SHA-256, mas foi excluído da saída devido às opções `exclude: true` e `toClassOnly: true`. Isso significa que a senha é aplicada hash ao converter para uma instância de classe, mas não é incluída na saída serializada.

**Exemplo Sem Excluir**

Se removermos a opção `exclude: true` para `password`, ela ainda será aplicada hash, mas será incluída na saída:

```typescript
{
    // Transformado para letras maiúsculas
    name: "JOHN DOE",   
    // Senha com hash
    password: "5e884898da28047151d0e56f..."  
}
```

Nesse caso, a função `transform` é aplicada e a senha recebe hash, mas não é mais excluída da saída. A opção `toClassOnly: true` garante que a transformação seja aplicada apenas ao converter para uma instância de classe.
