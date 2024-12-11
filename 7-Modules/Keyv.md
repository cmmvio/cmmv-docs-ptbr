# Keyv

O módulo ``@cmmv/keyv`` integra-se com o pacote [Keyv](https://keyv.org/) para fornecer um serviço flexível de armazenamento chave-valor. Este módulo permite que os desenvolvedores integrem soluções de armazenamento como Redis, Memcached, MongoDB e mais, em suas aplicações, com configuração mínima. Diferentemente do módulo ``@cmmv/cache``, que foca principalmente em cachear rotas e respostas de gateways, o ``@cmmv/keyv`` é destinado à gestão de estado em microsserviços e múltiplas aplicações que compartilham o mesmo armazenamento. O módulo inclui suporte nativo para namespaces e compactação para reduzir a latência em ``get`` e apresenta parsing customizado de JSON para manipulação de dados de forma mais eficiente em implementações futuras.

## Instalação

Para instalar o módulo ``@cmmv/keyv``, use o seguinte comando:

```bash 
$ pnpm add @cmmv/keyv keyv @keyv/compress-gzip
$ pnpm add -D @types/keyv
```

Se você planeja usar Redis, Memcached ou MongoDB como seu armazenamento, será necessário instalar o driver apropriado:

Armazenamentos Chave-Valor e Drivers Suportados

**Redis:** [Github](https://github.com/jaredwray/keyv/tree/main/packages/redis)

```bash
$ pnpm add @keyv/redis
```

**Memcached:** [Github](https://github.com/jaredwray/keyv/tree/main/packages/memcache)

```bash
$ pnpm add @keyv/memcache
```

**MongoDB:** [Github](https://github.com/jaredwray/keyv/tree/main/packages/mongo)

```bash
$ pnpm add @keyv/mongo
```

**PostgreSQL:** [Github](https://github.com/jaredwray/keyv/tree/main/packages/postgres)

```bash
$ pnpm add @keyv/postgres
```

**SQLite:** [Github](https://github.com/jaredwray/keyv/tree/main/packages/sqlite)

```bash
$ pnpm add @keyv/sqlite
```

Para uma lista completa de armazenamentos suportados, visite [Keyv on npm](https://www.npmjs.com/package/keyv).

## Configuração

O módulo ``@cmmv/keyv`` oferece configuração fácil por meio do sistema CMMV. Abaixo está um exemplo de como configurar um armazenamento Redis para uso com ``@cmmv/keyv``:

```javascript
module.exports = {
    // Outras configurações

    keyv: {
        uri: 'redis://localhost:6379',
        options: {
            namespace: 'cmmv',
            ttl: 600,
            adapter: 'redis',
            ...
        },
    },
}
```

| Opção         | Tipo                     | Obrigatório | Descrição                                                                 | Padrão               |
|---------------|--------------------------|-------------|---------------------------------------------------------------------------|-----------------------|
| namespace     | String                   | N           | Namespace para a instância atual.                                        | 'keyv'               |
| ttl           | Number                   | N           | TTL padrão. Pode ser sobrescrito especificando um TTL em ``.set()``. | undefined            |
| compression   | @keyv/compress-gzip      | N           | Pacote de compactação a ser usado. Veja Compression para mais detalhes.  | undefined            |
| serialize     | Function                 | N           | Uma função de serialização customizada.                                  | JSONB.stringify      |
| deserialize   | Function                 | N           | Uma função de desserialização customizada.                               | JSONB.parse          |
| store         | Instância de adaptador   | N           | A instância do adaptador de armazenamento usada pelo Keyv.               | new Map()            |
| adapter       | String                   | N           | Especifica um adaptador a ser usado, como 'redis' ou 'mongodb'.          | undefined            |

## Serviço Keyv

O módulo ``@cmmv/keyv`` inclui um serviço que pode ser opcionalmente injetado em controladores, gateways ou outros serviços para acesso direto ao armazenamento chave-valor. Diferentemente do módulo ``@cmmv/cache``, o ``@cmmv/keyv`` não fornece decoradores para cache automático, pois seu caso de uso foca mais na gestão de estado em microsserviços e aplicações distribuídas.

Aqui está um exemplo de como usar o ``KeyvService`` em um controlador para interagir com o armazenamento chave-valor:

```typescript
import { Controller, Get, Post, Body, Request, Param } from '@cmmv/http';
import { KeyvService } from '@cmmv/keyv';

@Controller('state')
export class StateController {
  constructor(private readonly keyvService: KeyvService) {}

  // GET valor por chave
  @Get(':key')
  async getValue(@Param('key') key: string, @Request() req): Promise<any> {
    const value = await this.keyvService.get(key);
    return value ? JSON.parse(value) : { message: 'Chave não encontrada' };
  }

  // POST (Definir valor para uma chave)
  @Post()
  async setValue(@Body() data: { key: string; value: any }, @Request() req): Promise<any> {
    const stringifiedValue = JSON.stringify(data.value);
    await this.keyvService.set(data.key, stringifiedValue);
    return { message: 'Valor definido com sucesso' };
  }
}
```

### Métodos do Serviço Keyv

<br/>

* **get(key: string): Promise<any>:** Recupera um valor pela chave.
* **set(key: string, value: any, ttl?: number): Promise<void>:** Armazena um valor pela chave com TTL opcional.
* **delete(key: string): Promise<boolean>:** Exclui um valor pela chave.
* **clear(): Promise<boolean>:** Exclui todos os valores.

## Parsing Customizado de JSON

O ``@cmmv/keyv`` inclui suporte para parsing customizado de JSON, que será importante para suporte futuro ao pacote [fast-json-stringify](https://www.npmjs.com/package/fast-json-stringify). Isso permitirá uma serialização e desserialização mais rápida e eficiente de objetos no armazenamento chave-valor.

A futura integração com o ``fast-json-stringify`` trará melhorias significativas de desempenho para manipulação de JSON no armazenamento chave-valor. Com esse recurso, o ``@cmmv/keyv`` suportará parsing de JSON otimizado, permitindo operações de leitura e gravação mais rápidas, especialmente para estruturas de dados complexas. Isso será um aprimoramento essencial para aplicações que exigem gestão de estado de alto desempenho em microsserviços.

## Uso em Microsserviços

O módulo ``@cmmv/keyv`` é projetado para suportar aplicações distribuídas e microsserviços. Ao aproveitar armazenamentos chave-valor compartilhados como Redis ou Memcached, as aplicações podem gerenciar estados compartilhados entre diferentes serviços, melhorando a consistência e reduzindo a latência. Isso o torna uma ótima escolha para cenários onde várias aplicações ou serviços precisam interagir com os mesmos dados com estado.

O módulo ``@cmmv/keyv`` fornece uma solução flexível e poderosa de armazenamento chave-valor para aplicações baseadas em CMMV. Ele é particularmente útil para gerenciar o estado da aplicação em microsserviços, graças ao suporte para namespaces, compactação e parsing customizado de JSON. Embora não inclua decoradores de cache automático como o ``@cmmv/cache``, oferece uma solução ideal para cenários onde o estado compartilhado é crucial, como ambientes de múltiplas aplicações ou ecossistemas de microsserviços.
