# Configurações

A configuração do sistema para um projeto CMMV é gerenciada através do arquivo `.cmmv.config.cjs` na raiz do diretório. Este arquivo define várias configurações, como:

* **Configurações do Servidor:** Configura o host e a porta da aplicação.
* **I18n:** Gerencia internacionalização com arquivos de idioma e linguagens padrão.
* **Repositório:** Define o tipo de banco de dados (ex.: SQLite) e os detalhes de conexão.
* **Configuração de Head:** Define atributos HTML, meta tags e links para o frontend da aplicação.
* **Headers:** Políticas de segurança como Content-Security-Policy.
* **Scripts:** Especifica arquivos JavaScript a serem carregados.

Os módulos podem estender essa configuração com base nas necessidades da aplicação.

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
        title: "Documentação | CMMV - Um framework minimalista para Node.js",
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

O renderizador HTTP padrão no CMMV suporta nativamente internacionalização, configuração de SEO e headers personalizados com foco em segurança. Esses recursos permitem que os desenvolvedores gerenciem conteúdos localizados, melhorem a otimização para motores de busca e apliquem políticas de segurança como Content-Security-Policy diretamente na configuração. Configurações adicionais para templates e visualizações também podem ser adicionadas, proporcionando flexibilidade para personalizar as páginas. Instruções mais detalhadas podem ser encontradas na documentação "View", que explica como implementar essas configurações no sistema de templates.

## Assign

O método `Config.assign()` permite modificar ou sobrescrever dinamicamente a configuração do sistema em tempo de execução. Esse método aceita um objeto que mescla ou atualiza a configuração existente. Abaixo está um exemplo de como a configuração pode ser aplicada usando `Config.assign()`:

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

Com `Config.assign()`, você pode modificar configurações como a configuração do servidor, internacionalização e mais, sem precisar codificar diretamente no arquivo `.cmmv.config.cjs`.

## API

As configurações do sistema em uma aplicação baseada no CMMV podem ser acessadas programaticamente através da classe Config do módulo `@cmmv/core`. Isso permite que os desenvolvedores recuperem, modifiquem ou excluam dinamicamente valores de configuração ao longo da aplicação. Por exemplo, as configurações de host e porta do servidor, configurações de banco de dados ou opções de internacionalização podem ser acessadas e alteradas chamando métodos como `Config.get()`, `Config.set()` e `Config.has()`. Esses recursos possibilitam uma abordagem centralizada e flexível para gerenciar as configurações do sistema.

```typescript
import { Config } from "@cmmv/core";

// Obter
const serverPort = Config.get<number>('server.port');
console.log(`Servidor rodando na porta: \${serverPort}`);

// Definir
Config.set('server.host', '127.0.0.1');

// Excluir
Config.delete('repository.logging');

// Obter Tudo
const allConfig = Config.getAll();
console.log(allConfig);

// Assign
Config.assign({
    server: { port: 4000 },
    repository: { type: 'sqlite' }
});
```

Cada módulo no sistema CMMV possui seu próprio conjunto de configurações que podem ser adicionadas ao arquivo de configuração central (`.cmmv.config.cjs`). Isso permite que os desenvolvedores personalizem o comportamento e os recursos dos módulos que estão utilizando. Como diferentes módulos podem exigir configurações específicas, é essencial revisar a documentação de cada módulo para entender as opções de configuração disponíveis e como elas se integram ao sistema principal. Isso garante a configuração ideal e o uso de funcionalidades avançadas, como RPC, autenticação, cache, entre outras.

## Validação de Schema

O CMMV introduz um sistema de validação baseado em schema para garantir que todas as configurações definidas no `.cmmv.config.cjs` estejam de acordo com a estrutura e os tipos esperados. Essa validação oferece um manuseio robusto de erros e garante que sua aplicação opere com configurações corretamente definidas.

Cada módulo pode definir seu próprio schema usando a interface `ConfigSchema`. Esse schema especifica os campos necessários, seus tipos, valores padrão e propriedades aninhadas, quando aplicável.

Aqui está um exemplo de schema para a configuração de um módulo `auth`, demonstrando propriedades simples e aninhadas:

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

O CMMV valida automaticamente a configuração carregada contra os schemas definidos em tempo de execução usando o método `Config.validateConfigs()`. Isso garante que todos os campos necessários estejam presentes, os tipos correspondam e as propriedades aninhadas sigam suas especificações.

### Tipos Suportados

O schema suporta os seguintes tipos:

- string
- number
- boolean
- object (incluindo propriedades aninhadas)
- array
- any (pula validação de tipo)

### Como a Validação Funciona

1. **Carregar Configurações:** O método `Config.loadConfig()` carrega o arquivo `.cmmv.config.cjs`.
2. **Validar Contra Schemas:** Cada módulo registra seu schema, e o método `Config.validateConfigs()` garante que todas as configurações correspondam aos seus respectivos schemas.
3. **Erros para Configurações Inválidas:** Se algum valor de configuração não corresponder ao schema, um erro será lançado com detalhes sobre a chave inválida e o tipo esperado.

Se uma configuração não atender aos requisitos do schema, o CMMV lançará um erro como:

```typescript
Error: Configuração "auth.google.clientID" espera o tipo "string" mas recebeu "undefined".
```

## Ambiente

A configuração `env` permite controlar o ambiente da aplicação (ex.: `development`, `production`). Essa configuração geralmente vem de variáveis de ambiente usando `dotenv` [NPM](https://www.npmjs.com/package/dotenv) para gerenciar valores sensíveis de forma segura.

```typescript
env: process.env.NODE_ENV, // Exemplo: 'development' ou 'production'
```

## Configuração do Servidor

As configurações do servidor controlam vários aspectos de como a aplicação é hospedada e seu comportamento em termos de segurança, desempenho e gerenciamento de sessões. A configuração suporta variáveis de ambiente para flexibilidade em diferentes ambientes de implantação.

```typescript
server: {
    host: process.env.HOST || '0.0.0.0',  // Host padrão
    port: process.env.PORT || 3000,       // Porta padrão
    poweredBy: false,                     // Remove o header 'X-Powered-By'
    removePolicyHeaders: false,           // Opção para remover headers adicionados pelo Helmet
    cors: true,                           // Habilita Cross-Origin Resource Sharing (CORS)
    
    // Configurações de compressão
    compress: {
        enabled: true,
        options: {
            level: 6,  // Nível de compressão (0-9)
        },
    },
    
    // Headers de segurança com Helmet.js
    helmet: {
        enabled: true,
        options: {
            contentSecurityPolicy: false, // CSP pode ser personalizado
        },
    },

    // Gerenciamento de sessões
    session: {
        enabled: true,  // Habilita sessões
        options: {
            sessionCookieName: process.env.SESSION_COOKIENAME || 'cmmv-session',
            secret: process.env.SESSION_SECRET || 'secret',  // Chave secreta para sessão
            resave: false,    // Previne o re-save de dados da sessão
            saveUninitialized: false, // Não salva sessões não inicializadas
            cookie: {
                secure: true,  // Garante cookie seguro (apenas HTTPS)
                maxAge: 60000, // Tempo de expiração do cookie em ms
            },
        },
    },
},
```

Essa seção inclui controle detalhado sobre CORS, gerenciamento de sessões, compressão e recursos de segurança como Helmet. Você também pode configurar diferentes ambientes usando `dotenv`.

## Internacionalização

O sistema i18n (Internacionalização) fornece uma maneira de gerenciar arquivos de localização e definir o idioma padrão da aplicação. Isso facilita a tradução e localização de conteúdo.

```typescript
i18n: {
    localeFiles: './src/locale',  // Caminho para os arquivos de localização
    default: 'en',               // Idioma padrão (Inglês)
},
```

## RPC

A configuração `rpc` permite que chamadas de procedimento remoto (RPC) sejam usadas na aplicação. O RPC melhora o desempenho e a modularidade do sistema ao habilitar a comunicação via contratos pré-definidos.

```typescript
rpc: {
    enabled: true,           // Habilita o sistema RPC
    preLoadContracts: true,  // Pré-carrega contratos RPC para eficiência
},
```

## View

As configurações de `view` controlam como o HTML é renderizado, incluindo opções para minificar HTML e lidar com scripts embutidos. Essas configurações podem ser ajustadas para otimizar a entrega do conteúdo frontend.

```typescript
view: {
    extractInlineScript: false, // Desabilita extração de scripts inline para maior controle
    minifyHTML: true,           // Habilita minificação de HTML para desempenho
},
```

## Repositório

As configurações de `repository` definem o tipo de banco de dados, caminho e opções para sincronização e registro de logs. Por padrão, o SQLite é usado, mas outros bancos de dados podem ser configurados conforme necessário.

```typescript
repository: {
    type: 'sqlite',                // Tipo de banco de dados padrão
    database: './database.sqlite', // Caminho para o arquivo do banco SQLite
    synchronize: true,             // Sincroniza o esquema do banco
    logging: false,                // Desabilita logs para desempenho
},
```

## Cache

A configuração de cache utiliza Redis por padrão para armazenar dados em cache, melhorando a velocidade e o desempenho da aplicação.

```typescript
cache: {
    store: '@tirke/node-cache-manager-ioredis',  // Store de cache Redis
    getter: 'ioRedisStore',                      // Getter do store de cache
    host: 'localhost',                           // Host do Redis
    port: 6379,                                  // Porta do Redis
    ttl: 600,                                    // Tempo de vida do cache (em segundos)
},
```

## Autenticação

Esta seção gerencia a autenticação de usuários, incluindo login/registro local e integração OAuth de terceiros (ex.: Google). A configuração permite a configuração flexível dos mecanismos de autenticação.

```typescript
auth: {
    localRegister: true,  // Habilita registro de usuários localmente
    localLogin: true,     // Habilita login de usuários localmente
    jwtSecret: process.env.JWT_SECRET || 'secret',  // Chave JWT para assinatura de tokens
    expiresIn: 60 * 60,   // Tempo de expiração do token JWT (em segundos)
    
    // Configuração OAuth do Google
    google: {
        clientID: process.env.GOOGLE_CLIENT_ID,               // ID do cliente Google
        clientSecret: process.env.GOOGLE_CLIENT_SECRET,       // Chave secreta do cliente Google
        callbackURL: 'http://localhost:3000/auth/google/callback',  // URL de callback OAuth
    },
},
```

## Head

A configuração `head` gerencia atributos HTML, meta tags e elementos `link`, que são essenciais para SEO e renderização adequada da aplicação.

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

## Headers

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
        src: '/assets/bundle.min.js',  // Bundle principal de JavaScript
        defer: 'defer',  // Carrega o script após o HTML ser analisado
    },
],
```

Com isso, concluímos a documentação detalhada das configurações. O método `assign` na classe Config permite atualizar ou substituir dinamicamente essas configurações, proporcionando flexibilidade em diferentes ambientes e módulos.