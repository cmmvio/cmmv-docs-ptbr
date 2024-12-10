# Serialização

A serialização e desserialização em um projeto que utiliza `class-transformer` são fundamentais para converter objetos JavaScript em instâncias de classes específicas e vice-versa. No exemplo abaixo, a classe `Task`, gerada automaticamente pelo `@cmmv/core`, pode ser serializada e validada, garantindo que a aplicação processe os dados de forma segura e com a tipagem correta.

O objetivo da serialização é transformar os dados em instâncias de classes apropriadas, garantindo que os objetos manipulados possuam as propriedades e tipos corretos. Utilizamos o pacote `class-transformer` para serializar e desserializar objetos e `class-validator` para validar os dados.

Aqui está a implementação de um modelo `Task` que representa uma tarefa e utiliza `class-validator` e `class-transformer` para garantir a serialização e validação adequadas dos campos:

```typescript
// Gerado automaticamente pelo CMMV

import { IsString, IsNotEmpty, IsBoolean } from 'class-validator';
import { plainToClass, classToPlain } from 'class-transformer';

export interface ITask {
    id?: any;
    label: string;
    checked: boolean;
    removed: boolean;
}

export class Task implements ITask {
    id?: any;

    @IsString({ message: 'Label inválido' })
    @IsNotEmpty({ message: 'Label inválido' })
    label: string;

    @IsBoolean({ message: 'Tipo de checked inválido' })
    checked: boolean = false;

    @IsBoolean({ message: 'Tipo de removed inválido' })
    removed: boolean = false;

    constructor(partial: Partial<Task>) {
        Object.assign(this, partial);
    }
}
```

<br/>

* **`plainToInstance:`** Converte objetos simples (como JSON) em instâncias de classes, transformando um objeto JSON recebido em uma instância da classe `Task`.
* **`instanceToPlain:`** Converte uma instância de classe de volta em um objeto simples para serialização ou envio em uma resposta HTTP.

Aqui está como converter um objeto simples que representa uma tarefa em uma instância da classe `Task` usando `plainToInstance`:

## Desserialização

```typescript
import { plainToInstance } from 'class-transformer';

// Objeto simples
const plainTask = {
    label: 'Concluir relatório',
    checked: true,
    removed: false,
};

// Converte para uma instância da classe Task
const taskInstance = plainToInstance(Task, plainTask);

console.log(taskInstance); // Instância da classe Task
```

## Serialização 

Quando a aplicação precisa enviar dados para um banco de dados ou uma API, é possível serializar a instância da classe de volta para um objeto simples usando `instanceToPlain`.

```typescript
import { instanceToPlain } from 'class-transformer';

const taskInstance = new Task({ label: 'Concluir relatório', checked: true });

// Converte para um objeto simples
const plainTask = instanceToPlain(taskInstance);

console.log(plainTask); // Objeto simples
```

## Validação de Dados

Use `validate` do `class-validator` para verificar se os dados são válidos:

```typescript
import { validate } from 'class-validator';

const taskInstance = new Task({ label: '', checked: 'true' });

validate(taskInstance).then(errors => {
    if (errors.length > 0) {
        console.log('Validação falhou. Erros: ', errors);
    } else {
        console.log('Validação bem-sucedida');
    }
});
```

Ao utilizar `class-transformer` e `class-validator`, você garante que as instâncias de classes sejam devidamente validadas e facilmente convertidas entre objetos simples e instâncias de classe. Isso fornece uma camada robusta de validação e simplifica o gerenciamento de dados, especialmente para APIs e bancos de dados.
