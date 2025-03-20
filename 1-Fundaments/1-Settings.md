# Configurações

A configuração do sistema para um projeto CMMV é gerenciada por meio de um arquivo `.cmmv.config.cjs` no diretório raiz. Este arquivo define várias configurações, como:

* **Configurações do Servidor:** Configura o host e a porta para a aplicação.
* **I18n:** Gerencia a internacionalização com arquivos de localização e idiomas padrão.
* **Repositório:** Define o tipo de banco de dados (por exemplo, SQLite) e detalhes de conexão.
* **Configuração de Cabeçalho:** Define atributos HTML, meta tags e links para o frontend da aplicação.
* **Cabeçalhos:** Políticas de segurança como Content-Security-Policy.
* **Scripts:** Especifica arquivos JavaScript a serem carregados.

Módulos podem estender essa configuração com base nas necessidades da aplicação.

```typescript
require('dotenv').config();

module.exports = {
    server: {
        host: "0.0.0.0",
        port: process.env.PORT || 3000
    },

    i18n: {
        localeFiles: "./src/locale",
        default: "en"
    },

    head: {
        title: "Documentação | CMMV - Um framework Node.js minimalista",
        htmlAttrs: {
            lang: "pt-br"
        },
        meta: [
            { charset: 'utf-8' },
            { "http-equiv": "content-type", content: "text/html; charset=UTF-8" },
            { name: 'viewport', content: 'width=device-width, initial-scale=1' },
            { name: "robots", content: "noodp" }
        ],
        link: [
            { rel: 'icon', href: '/assets/favicon/favicon.ico' },
            { rel: "dns-prefetch", href: "https://docs.cmmv.io" },
            { rel: "preconnect", href: "https://docs.cmmv.io" },
            { rel: "stylesheet", href: "/assets/docs.css" }
        ]
    },

    headers: {
        "Content-Security-Policy": [
            "default-src 'self'",
            "script-src 'self' 'unsafe-eval' 'unsafe-inline'",
            "style-src 'self' 'unsafe-inline'",
            "font-src 'self'"
        ]
    },

    scripts: [
        { type: "text/javascript", src: '/assets/bundle.min.js'},
    ]
};
```

O renderizador HTTP padrão no CMMV suporta nativamente internacionalização, configuração de SEO e cabeçalhos personalizados focados em segurança. Esses recursos permitem que os desenvolvedores gerenciem conteúdo localizado, melhorem a otimização para motores de busca e apliquem políticas de segurança como Content-Security-Policy diretamente na configuração. Configurações adicionais para modelos e visualizações também podem ser adicionadas posteriormente, proporcionando flexibilidade para personalizar páginas. Instruções mais detalhadas podem ser encontradas na documentação "View", que explica como implementar essas configurações dentro do sistema de modelos.

## Atribuir

O método `Config.assign()` permite que você modifique ou substitua dinamicamente a configuração do sistema em tempo de execução. Este método recebe um objeto que mescla ou atualiza a configuração existente. Abaixo está um exemplo de como a configuração pode ser aplicada usando `Config.assign()`:

```typescript
import { Config } from '@cmmv/core';

Config.assign({
    server: {
        host: process.env.HOST || '127.0.0.1',
        port: process.env.PORT || 4000,
    },
    i18n: {
        localeFiles: './locales',
        default: 'en',
    },
    // Configurações adicionais...
});
```

Com `Config.assign()`, você pode modificar de forma flexível configurações como a configuração do servidor, internacionalização e mais, sem codificá-las diretamente no arquivo `.cmmv.config.cjs`.

## API

As configurações do sistema em uma aplicação baseada em CMMV podem ser acessadas programaticamente por meio da classe Config do módulo `@cmmv/core`. Isso permite que os desenvolvedores recuperem, modifiquem ou excluam valores de configuração dinamicamente ao longo da aplicação. Por exemplo, as configurações de host e porta do servidor, configurações de banco de dados ou opções de internacionalização podem ser acessadas e alteradas chamando métodos como Config.get(), Config.set() e Config.has(). Esses recursos permitem uma abordagem flexível e centralizada para o gerenciamento das configurações do sistema.

```typescript
import { Config } from "@cmmv/core";

// Obter
const serverPort = Config.get<number>('server.port');
console.log(`Servidor rodando na porta: \${serverPort}`);

// Definir
Config.set('server.host', '127.0.0.1');

// Excluir
Config.delete('repository.logging');

// Obter tudo
const allConfig = Config.getAll();
console.log(allConfig);

// Atribuir
Config.assign({
    server: { port: 4000 },
    repository: { type: 'sqlite' }
});
```

Cada módulo no sistema CMMV possui seu próprio conjunto de configurações que podem ser adicionadas ao arquivo de configuração central (`.cmmv.config.cjs`). Isso permite que os desenvolvedores personalizem o comportamento e os recursos dos módulos que estão utilizando. Como diferentes módulos podem exigir configurações específicas, é essencial revisar cuidadosamente a documentação de cada módulo para entender as opções de configuração disponíveis e como elas se integram ao sistema principal. Isso garante uma configuração otimizada e o uso de funcionalidades avançadas como RPC, autenticação, cache e mais.

## Validação de Esquema

O CMMV introduz um sistema de validação baseado em esquema para garantir que todas as configurações definidas em `.cmmv.config.cjs` sigam a estrutura e os tipos esperados. Essa validação oferece um tratamento robusto de erros e garante que sua aplicação opere com configurações corretamente definidas.

Cada módulo pode definir seu próprio esquema usando a interface ConfigSchema. Este esquema especifica os campos obrigatórios, seus tipos, valores padrão e propriedades aninhadas quando aplicável.

Aqui está um exemplo de esquema para uma configuração de módulo `auth`, demonstrando propriedades planas e aninhadas:

```typescript
import { ConfigSchema } from '@cmmv/core';

export const AuthConfig: ConfigSchema = {
    auth: {
        localRegister: {
            required: true,
            type: 'boolean',
            default: true,
        },
        localLogin: {
            required: true,
            type: 'boolean',
            default: true,
        },
        jwtSecret: {
            required: true,
            type: 'string',
            default: 'secret',
        },
        expiresIn: {
            required: true,
            type: 'number',
            default: 3600,
        },
        google: {
            required: false,
            type: 'object',
            default: {},
            properties: {
                clientID: {
                    required: true,
                    type: 'string',
                    default: '',
                },
                clientSecret: {
                    required: true,
                    type: 'string',
                    default: '',
                },
                callbackURL: {
                    required: true,
                    type: 'string',
                    default: 'http://localhost:3000/auth/google/callback',
                },
            },
        },
    },
};
```

O CMMV valida automaticamente a configuração carregada contra os esquemas definidos em tempo de execução usando o método `Config.validateConfigs()`. Isso garante que todos os campos obrigatórios estejam presentes, que os tipos correspondam e que as propriedades aninhadas sigam suas especificações.

### Tipos Suportados

O esquema suporta os seguintes tipos:

* string
* number
* boolean
* object (incluindo propriedades aninhadas)
* array
* any (pula a validação de tipo)

### Como a Validação Funciona
<br/>

* **Carregar Configurações:** O método `Config.loadConfig()` carrega o arquivo `.cmmv.config.cjs`.
* **Validar Contra Esquemas:** Cada módulo registra seu esquema, e o método `Config.validateConfigs()` garante que todas as configurações correspondam aos seus respectivos esquemas.
* **Lançar Erros para Configurações Inválidas:** Se algum valor de configuração não corresponder ao esquema, um erro é lançado com detalhes sobre a chave inválida e o tipo esperado.

Se uma configuração não atender aos requisitos do esquema, o CMMV lançará um erro como:

```typescript
Error: Configuração "auth.google.clientID" espera o tipo "string" mas recebeu "undefined".
```

## Ambiente

A configuração `env` permite que você controle o ambiente da aplicação (por exemplo, `development`, `production`). Essa configuração geralmente vem de variáveis de ambiente usando `dotenv` [NPM](https://www.npmjs.com/package/dotenv) para gerenciar valores sensíveis de forma segura.

```typescript
env: process.env.NODE_ENV, // Exemplo: 'development' ou 'production'
```

## Configuração do Servidor

As configurações do servidor controlam vários aspectos de como a aplicação é hospedada e seu comportamento em termos de segurança, desempenho e gerenciamento de sessões. A configuração suporta variáveis de ambiente para flexibilidade em diferentes ambientes de implantação.

```typescript
server: {
    host: process.env.HOST || '0.0.0.0',  // Host padrão
    port: process.env.PORT || 3000,       // Porta padrão
    poweredBy: false,                     // Remove o cabeçalho 'X-Powered-By'
    removePolicyHeaders: false,           // Opção para remover cabeçalhos adicionados pelo Helmet
    cors: true,                           // Habilita o Compartilhamento de Recursos de Origem Cruzada (CORS)

    // Configurações de compressão
    compress: {
        enabled: true,
        options: {
            level: 6,  // Nível de compressão (0-9)
        },
    },

    // Cabeçalhos de segurança com Helmet.js
    helmet: {
        enabled: true,
        options: {
            contentSecurityPolicy: false, // CSP pode ser personalizado
        },
    },

    // Gerenciamento de sessão
    session: {
        enabled: true,  // Habilita sessões
        options: {
            sessionCookieName: process.env.SESSION_COOKIENAME || 'cmmv-session',
            secret: process.env.SESSION_SECRET || 'secret',  // Segredo da sessão
            resave: false,    // Impede o resalvamento de dados da sessão
            saveUninitialized: false, // Não salva sessões não inicializadas
            cookie: {
                secure: true,  // Garante cookie seguro (somente por HTTPS)
                maxAge: 60000, // Tempo de expiração do cookie em ms
            },
        },
    },
},
```

Esta seção inclui controle detalhado sobre CORS, gerenciamento de sessões, compressão e recursos de segurança como Helmet. Você também pode configurar diferentes ambientes usando `dotenv`.

## Internacionalização

O sistema i18n (Internacionalização) fornece uma maneira de gerenciar arquivos de localização e definir o idioma padrão para a aplicação. Isso ajuda na tradução e localização de conteúdo.

```typescript
i18n: {
    localeFiles: './src/locale',  // Caminho para os arquivos de localização
    default: 'en',                // Idioma padrão (Inglês)
},
```

## RPC

A configuração `rpc` permite que Chamadas de Procedimento Remoto (RPC) sejam usadas na aplicação. O RPC melhora o desempenho e a modularidade do sistema ao possibilitar a comunicação por meio de contratos predefinidos.

```typescript
rpc: {
    enabled: true,           // Habilita o sistema RPC
    preLoadContracts: true,  // Pré-carrega contratos RPC para eficiência
},
```

## Visualização

As configurações de visualização controlam como o HTML é renderizado, incluindo opções para minificar HTML e lidar com scripts inline. Essas configurações podem ser ajustadas para otimizar a entrega de conteúdo frontend.

```typescript
view: {
    extractInlineScript: false, // Desabilita a extração de scripts inline para melhor controle
    minifyHTML: true,           // Habilita a minificação de HTML para desempenho
},
```

## Repositório

As configurações de `repository` definem o tipo de banco de dados, caminho e opções para sincronização e registro. Por padrão, o SQLite é usado, mas outros bancos de dados podem ser configurados conforme necessário.

```typescript
repository: {
    type: 'sqlite',                // Tipo de banco de dados padrão
    database: './database.sqlite',  // Caminho para o arquivo de banco de dados SQLite
    synchronize: true,              // Sincroniza o esquema do banco de dados
    logging: false,                 // Desabilita o registro para desempenho
},
```

## Cache

A configuração de cache usa o Redis por padrão para armazenar dados em cache, melhorando a velocidade e o desempenho da aplicação.

```typescript
cache: {
    store: '@tirke/node-cache-manager-ioredis',  // Armazenamento de cache Redis
    getter: 'ioRedisStore',                      // Obtentor de armazenamento de cache
    host: 'localhost',                           // Host Redis
    port: 6379,                                  // Porta Redis
    ttl: 600,                                    // Tempo de vida do cache (em segundos)
},
```

## Autenticação

Esta seção lida com a autenticação do usuário, incluindo login/registro local e integração com OAuth de terceiros (por exemplo, Google). A configuração permite uma configuração flexível de mecanismos de autenticação.

```typescript
auth: {
    localRegister: true,  // Habilita o registro de usuário local
    localLogin: true,     // Habilita o login de usuário local
    jwtSecret: process.env.JWT_SECRET || 'secret',  // Segredo JWT para assinatura de token
    expiresIn: 60 * 60,   // Tempo de expiração do token JWT (em segundos)

    // Configuração OAuth do Google
    google: {
        clientID: process.env.GOOGLE_CLIENT_ID,               // ID do cliente Google
        clientSecret: process.env.GOOGLE_CLIENT_SECRET,       // Segredo do cliente Google
        callbackURL: 'http://localhost:3000/auth/google/callback',  // URL de retorno OAuth
    },
},
```

## Cabeçalho

A configuração `head` gerencia atributos HTML, meta tags e elementos de link, que são essenciais para SEO e renderização adequada da aplicação.

```typescript
head: {
    title: 'CMMV',  // Título padrão
    htmlAttrs: {
        lang: 'pt-br',  // Idioma padrão
    },
    meta: [
        { charset: 'utf-8' },
        {
            'http-equiv': 'content-type',
            content: 'text/html; charset=UTF-8',
        },
        {
            name: 'viewport',
            content: 'width=device-width, initial-scale=1',
        },
    ],
    link: [{ rel: 'icon', href: 'assets/favicon/favicon.ico' }],
},
```

## Cabeçalhos

```typescript
headers: {
    'Content-Security-Policy': [
        "default-src 'self'",
        "script-src 'self' 'unsafe-eval' 'unsafe-hashes'",
        "style-src 'self' 'unsafe-inline'",
        "font-src 'self'",
        "connect-src 'self'",
    ],
},
```

## Scripts

A configuração `scripts` define os arquivos JavaScript a serem carregados na aplicação, que podem ser personalizados com base nas necessidades do projeto.

```typescript
scripts: [
    {
        type: 'text/javascript',
        src: '/assets/bundle.min.js',  // Pacote JavaScript principal
        defer: 'defer',  // Carrega o script após o HTML ser analisado
    },
],
```

Isso conclui a documentação detalhada das configurações. O método assign na Config permite que você atualize ou substitua dinamicamente essas configurações, proporcionando flexibilidade em diferentes ambientes e módulos.
