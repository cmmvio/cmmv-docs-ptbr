# Serialização

A serialização e desserialização em um projeto usando o ``class-transformer`` são fundamentais para converter objetos JavaScript em instâncias de classes específicas e vice-versa. No exemplo acima, a classe ``Task``, gerada automaticamente pelo ``@cmmv/core``, pode ser serializada e validada, garantindo que a aplicação processe os dados de forma segura e com tipagem correta.

O objetivo da serialização é transformar os dados em instâncias de classes apropriadas, assegurando que os objetos manipulados tenham as propriedades e tipos corretos. Utilizamos o pacote ``class-transformer`` para serializar e desserializar objetos e o ``class-validator`` para validar os dados.

Aqui está uma implementação de um modelo ``Task`` que representa uma tarefa e utiliza o ``class-validator`` e o ``class-transformer`` para garantir a serialização e validação adequadas dos campos:

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

    @IsString({ message: 'Rótulo inválido' })
    @IsNotEmpty({ message: 'Rótulo inválido' })
    label: string;

    @IsBoolean({ message: 'Tipo de "checked" inválido' })
    checked: boolean = false;

    @IsBoolean({ message: 'Tipo de "removed" inválido' })
    removed: boolean = false;

    constructor(partial: Partial<Task>) {
        Object.assign(this, partial);
    }
}
```

<br/>

* **``plainToInstance:``** Converte objetos simples (como JSON) em instâncias de classe, transformando um objeto JSON recebido em uma instância da classe ``Task``.
* **``instanceToPlain:``** Converte uma instância de classe de volta em um objeto simples para serialização ou envio em uma resposta HTTP.

Aqui está como convertemos um objeto simples que representa uma tarefa em uma instância da classe ``Task`` usando ``plainToInstance``:

## Desserialização

```typescript
import { plainToInstance } from 'class-transformer';

// Objeto simples
const plainTask = {
    label: 'Completar relatório',
    checked: true,
    removed: false,
};

// Converte para uma instância da classe Task
const taskInstance = plainToInstance(Task, plainTask);

console.log(taskInstance); // Instância de Task
```

## Serialização

Quando a aplicação precisa enviar dados para um banco de dados ou uma API, ela pode serializar a instância da classe de volta em um objeto simples usando ``instanceToPlain``.

```typescript
import { instanceToPlain } from 'class-transformer';

const taskInstance = new Task({ label: 'Completar relatório', checked: true });

// Converte para um objeto simples
const plainTask = instanceToPlain(taskInstance);

console.log(plainTask); // Objeto simples
```

## Validação de Dados

Use o método `validate` do ``class-validator`` para verificar se os dados são válidos:

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

Ao usar o ``class-transformer`` e o ``class-validator``, você garante que as instâncias de classe sejam devidamente validadas e facilmente convertidas entre objetos simples e instâncias de classe. Isso proporciona uma forte camada de validação e simplifica o manuseio de dados, especialmente para APIs e bancos de dados.
