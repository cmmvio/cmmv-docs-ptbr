# Singleton

No CMMV, o padrão tradicional de injeção de dependência é substituído por registros singleton, que simplificam o gerenciamento de serviços globais. Em muitas aplicações, módulos frequentemente compartilham serviços como cache, filas ou conexões de banco de dados. Em vez de declarar e injetar esses serviços em vários módulos, um singleton garante que apenas uma instância de um serviço seja criada e compartilhada por toda a aplicação.

Essa abordagem oferece várias vantagens:

* Simplifica o gerenciamento de serviços ao eliminar a necessidade de injeção explícita em módulos.
* Reduz a complexidade ao evitar dependências circulares.
* Otimiza o desempenho ao garantir que apenas uma instância de cada serviço exista, minimizando a sobrecarga.

## Exemplo

```typescript
import { Singleton } from "@cmmv/core";

export class Scope extends Singleton {
    private data: Map<string, any> = new Map();

    public static set(name: string, data: any): boolean {
        const scope = Scope.getInstance();

        if (!scope.data.has(name)) {
            scope.data.set(name, data);
            return true;
        }

        return false;
    }

    public static has(name: string): boolean {
        const scope = Scope.getInstance();
        return scope.data.has(name);
    }

    public static get<T = any>(name: string): T | null {
        const scope = Scope.getInstance();
        return scope.data.has(name) ? (scope.data.get(name) as T) : null;
    }

    public static clear(name: string): void {
        const scope = Scope.getInstance();
        scope.data.delete(name);
    }

    public static addToArray<T = any>(name: string, value: T): boolean {
        const scope = Scope.getInstance();
        const array = scope.data.get(name) || [];

        if (Array.isArray(array)) {
            array.push(value);
            scope.data.set(name, array);
            return true;
        }

        return false;
    }

    public static removeFromArray<T = any>(name: string, value: T): boolean {
        const scope = Scope.getInstance();
        const array = scope.data.get(name);

        if (Array.isArray(array)) {
            const index = array.indexOf(value);

            if (index > -1) {
                array.splice(index, 1);
                scope.data.set(name, array);
                return true;
            }
        }

        return false;
    }

    public static getArray<T = any>(name: string): T[] | null {
        const scope = Scope.getInstance();
        const array = scope.data.get(name);

        if (Array.isArray(array))
            return array as T[];

        return null;
    }

    public static getArrayFromIndex<T = any>(name: string, index: number): T | null {
        const scope = Scope.getInstance();
        const array = scope.data.get(name);

        if (Array.isArray(array) && array.length >= index)
            return array[index] as T;

        return null;
    }
}
```

## Vantagens

O uso de singletons oferece várias vantagens sobre sistemas tradicionais de injeção de dependência, especialmente para aplicações maiores com serviços complexos e múltiplos módulos:

* **Otimização de Desempenho:** Como os singletons são instanciados apenas uma vez, eles reduzem a sobrecarga associada à criação de múltiplas instâncias de serviços. Isso é particularmente benéfico ao lidar com serviços de alta frequência, como conexões de banco de dados ou gerenciadores de cache.

* **Arquitetura Simplificada:** Em vez de gerenciar árvores de dependência complexas e importações em cada módulo, os registros singleton permitem acesso direto aos serviços, simplificando o design do módulo e reduzindo a necessidade de código boilerplate.

* **Evita Dependências Circulares:** Dependências circulares ocorrem quando dois serviços dependem um do outro, fazendo com que sistemas de injeção de dependência falhem ou se tornem excessivamente complexos. Singletons evitam esse problema por serem autocontidos e globalmente acessíveis.

* **Facilidade de Acesso:** Serviços como configuração, cache e logging podem ser tornados globalmente acessíveis sem a necessidade de serem injetados explicitamente em cada módulo ou serviço que os utiliza.

* **Escalabilidade:** À medida que a aplicação cresce, o uso de singletons simplifica o gerenciamento de serviços. Diferentemente dos sistemas tradicionais de injeção de dependência que exigem o gerenciamento cuidadoso de importações e provedores globais, os singletons oferecem uma solução direta para compartilhar serviços entre diferentes módulos.

## Melhores Práticas

* **Use Singletons para Serviços Globais:** Utilize registros singleton para serviços que precisam ser compartilhados entre vários módulos, como configuração, logging, cache e conexões de banco de dados.
* **Minimize o Uso Excessivo:** Embora os singletons simplifiquem o gerenciamento de serviços, usá-los em excesso pode dificultar testes e depuração. Use-os com moderação para serviços verdadeiramente globais.
* **Evite Mutabilidade de Estado:** Se possível, mantenha os serviços singleton sem estado ou gerencie cuidadosamente seu estado para evitar efeitos colaterais indesejados.
* **Testes:** Ao testar serviços singleton, lembre-se de que eles mantêm seu estado ao longo do ciclo de vida da aplicação. Redefina ou simule instâncias singleton conforme necessário durante os testes.
