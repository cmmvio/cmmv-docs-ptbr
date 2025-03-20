# Autenticação

Repositório: [https://github.com/cmmvio/cmmv/tree/main/packages/auth](https://github.com/cmmvio/cmmv/tree/main/packages/auth)

O módulo ``@cmmv/auth`` oferece um conjunto de funcionalidades para gerenciar autenticação em sua aplicação. Ele suporta autenticação baseada em HTTP e WebSocket e pode ser facilmente integrado a qualquer aplicação baseada em ``@cmmv``.

## Instalação

Para instalar o módulo ``@cmmv/auth``, execute o seguinte comando:

```typescript
$ pnpm add @cmmv/auth
```

## Integração

Após a instalação, você pode integrar o módulo ``@cmmv/auth`` em sua aplicação conforme mostrado abaixo. Este exemplo demonstra a configuração básica de uma aplicação CMMV que inclui o módulo ``@cmmv/auth`` para lidar com autenticação.

```typescript
require('dotenv').config();

import { Application } from '@cmmv/core';
import { DefaultAdapter, DefaultHTTPModule } from '@cmmv/http';
import { ProtobufModule } from '@cmmv/protobuf';
import { WSModule, WSAdapter } from '@cmmv/ws';
...
import { AuthModule } from '@cmmv/auth';

Application.create({
    httpAdapter: DefaultAdapter,
    wsAdapter: WSAdapter,
    modules: [
        DefaultHTTPModule,
        ProtobufModule,
        WSModule,
        RepositoryModule,
        AuthModule,  // Adiciona o AuthModule
    ],
    services: [
        // Adicione seus serviços personalizados aqui
    ],
    contracts: [
        // Adicione seus contratos aqui
    ],
});
```

## Configuração

O arquivo ``.cmmv.config.cjs`` é o arquivo de configuração central da sua aplicação CMMV, permitindo que você defina diferentes configurações relacionadas ao servidor, autenticação e outros módulos. Abaixo está uma explicação detalhada das opções de configuração relevantes para o módulo ``@cmmv/auth``:

```javascript
module.exports = {
    env: process.env.NODE_ENV,

    server: {
        session: {
            enabled: true,  // Habilita suporte a sessões
            options: {
                // Nome do cookie da sessão
                sessionCookieName: "cmmv-session",
                // Segredo para assinar o cookie de ID da sessão
                secret: process.env.SESSION_SECRET,
                // Impede que a sessão seja salva novamente no armazenamento de sessões
                resave: false,
                // Força a gravação de uma sessão mesmo que não inicializada
                saveUninitialized: false,
                cookie: {
                    // Garante que o navegador envie o cookie apenas por HTTPS
                    secure: true,
                    // Tempo máximo de vida (em milissegundos) do cookie da sessão
                    maxAge: 60000
                }
            }
        }
    },

    // Configurações de autenticação para @cmmv/auth
    auth: {
        // Habilita registro local (email/senha)
        localRegister: true,
        // Habilita login local (email/senha)
        localLogin: true,
        // Chave secreta para assinar tokens JWT
        jwtSecret: process.env.JWT_SECRET || "secret",
        // Chave secreta para assinar token de atualização JWT
        jwtSecretRefresh: process.env.JWT_SECRET_REFRESH,
        // Nome do cookie do token de atualização
        refreshCookieName: "refreshToken",
        // Tempo de expiração do token em segundos (1 hora)
        expiresIn: 60 * 60,
        qrCode: {}, // Veja o tópico QR Code
        optSecret: {
            issuer: "CMMV",
            algorithm: "sha512"
        },
        recaptcha: {
            required: true,
            secret: process.env.RECAPTCHA_SECRET
        }
    },
};
```

## Gerados

O módulo ``@cmmv/auth`` é responsável por gerar os arquivos necessários para autenticação em uma aplicação baseada em CMMV. Isso inclui buffers de protocolo (arquivos proto), definições TypeScript, implementações de serviços e controladores. Se o módulo ``@cmmv/ws`` estiver instalado, ele também gerará o gateway WebSocket para lidar com eventos de autenticação. Abaixo estão os principais componentes gerados por este módulo:

**``.generated/protos/auth/user.proto``**

Este arquivo ``.proto`` define as mensagens ``User``, ``LoginRequest``, ``LoginResponse``, ``RegisterRequest``, ``RegisterResponse`` e o serviço ``AuthService`` com dois métodos RPC, Login e Register.

```proto
/**
    **********************************************
    Este script foi gerado automaticamente pelo CMMV.
    Não é recomendado modificar este arquivo manualmente,
    pois ele pode ser sobrescrito pela aplicação.
    **********************************************
**/

syntax = "proto3";
package auth;

import "./groups.proto";

message User {
   string username = 1;
   string password = 2;
   string provider = 3;
   repeated Groups groups = 4;
   repeated string roles = 5;
   bool root = 6;
   bool blocked = 7;
   bool validated = 8;
   bool verifyEmail = 9;
   int32 verifyEmailCode = 10;
   bool verifySMS = 11;
   int32 verifySMSCode = 12;
   string optSecret = 13;
   bool optSecretVerify = 14;
}

message UserList {
    repeated User items = 1;
}

message AddUserRequest {
    User item = 1;
}

message AddUserResponse {
    string id = 1;
    User item = 2;
}

message UpdateUserRequest {
    string id = 1;
    User item = 2;
}

message UpdateUserResponse {
    bool success = 1;
    int32 affected = 2;
}

message DeleteUserRequest {
    string id = 1;
}

message DeleteUserResponse {
    bool success = 1;
    int32 affected = 2;
}

message GetAllUserRequest {}

message GetAllUserResponse {
    UserList items = 1;
}

message LoginRequest {
   string username = 1;
   string password = 2;
}

message LoginResponse {
   bool success = 1;
   optional string token = 2;
   optional string message = 3;
}

message RegisterRequest {
   string username = 1;
   string email = 2;
   string password = 3;
}

message RegisterResponse {
   bool success = 1;
   optional string message = 2;
}

service UserService {
   rpc AddUser (AddUserRequest) returns (AddUserResponse);
   rpc UpdateUser (UpdateUserRequest) returns (UpdateUserResponse);
   rpc DeleteUser (DeleteUserRequest) returns (DeleteUserResponse);
   rpc GetAllUser (GetAllUserRequest) returns (GetAllUserResponse);
   rpc Login (LoginRequest) returns (LoginResponse);
   rpc Register (RegisterRequest) returns (RegisterResponse);
}
```
<br/>

**``.generated/protos/auth/auth.d.ts``**

Este arquivo fornece interfaces TypeScript que refletem as definições do ``.proto``, facilitando o trabalho com as estruturas de dados em sua aplicação.

```typescript
/**
    **********************************************
    Este script foi gerado automaticamente pelo CMMV.
    Não é recomendado modificar este arquivo manualmente,
    pois ele pode ser sobrescrito pela aplicação.
    **********************************************
**/

import { Groups } from "./groups.d";

export namespace User {
    export type username = string;
    export type password = string;
    export type provider = string;
    export type groups = Groups;
    export type roles = string;
    export type root = boolean;
    export type blocked = boolean;
    export type validated = boolean;
    export type verifyEmail = boolean;
    export type verifyEmailCode = number;
    export type verifySMS = boolean;
    export type verifySMSCode = number;
    export type optSecret = string;
    export type optSecretVerify = boolean;
}

export interface AddUserRequest {
    item: User;
}

export interface AddUserResponse {
    item: User;
}

export interface UpdateUserRequest {
    id: string;
    item: User;
}

export interface UpdateUserResponse {
    success: boolean;
    affected: number;
}

export interface DeleteUserRequest {
    id: string;
}

export interface DeleteUserResponse {
    success: boolean;
    affected: number;
}

export interface GetAllUserRequest {}

export interface GetAllUserResponse {
    items: User[];
}
```
<br/>

## Decorador

O decorador ``@Auth`` foi projetado para impor autenticação e autorização em rotas específicas dentro de uma aplicação. Ele fornece funcionalidade de middleware que verifica se um usuário está autenticado ao validar um JWT (JSON Web Token) e, opcionalmente, valida se o usuário possui os papéis necessários para acessar a rota.

Esse decorador é usado em conjunto com manipuladores de rotas para proteger endpoints específicos. Ele valida a existência de um token JWT e, se fornecido, verifica se o payload do token inclui os papéis necessários para acessar a rota.

* **Localização do Token:** O middleware verifica primeiro o token JWT nos cookies da requisição (usando um nome de cookie configurável) ou no cabeçalho ``Authorization``. Se nenhum token for encontrado, retorna uma resposta ``401 Unauthorized``.
* **Verificação do Token:** Se um token for encontrado, ele verifica o token usando o segredo armazenado na configuração (``auth.jwtSecret``). Se o token for inválido ou expirado, o middleware retorna uma resposta ``401 Unauthorized``.
* **Validação de Papéis:** Se papéis forem especificados, o middleware verifica se o payload do token inclui os papéis necessários. Se o usuário não possuir os papéis apropriados, o middleware retorna uma resposta ``403 Forbidden``.
* **Informações do Usuário:** Após validação bem-sucedida, o token decodificado (que normalmente inclui informações do usuário como ID, nome de usuário e papéis) é anexado ao objeto ``req.user`` para uso posterior no manipulador de rota.

```typescript
import { Controller, Get, Req } from '@cmmv/http';
import { Auth } from '@cmmv/auth';

@Controller('user')
export class UserController {
    // Protege a rota, verifica JWT válido
    @Get('profile')
    @Auth()
    async getProfile(@Req() req) {
        return req.user;  // Acessa as informações do usuário autenticado
    }

    // Protege a rota, verifica JWT válido e papel admin
    @Get('admin')
    @Auth(['admin'])
    async getAdminDashboard(@Req() req) {
        return `Bem-vindo, admin \${req.user.username}`;
    }
}
```
<br/>

* **roles?:** (Opcional) Um array de papéis que o usuário deve possuir para acessar a rota. Se nenhum papel for fornecido, apenas autenticação é necessária (ou seja, um JWT válido). Se papéis forem especificados, o usuário deve possuir pelo menos um dos papéis fornecidos.

```typescript
@Auth(['admin', 'moderator'])
```

**Dependências de Configuração**

* **Segredo JWT:** O decorador usa o segredo especificado na configuração (``auth.jwtSecret``) para verificar o token.
* **Nome do Cookie da Sessão:** Por padrão, o token é verificado em um cookie com o nome especificado em ``server.session.options.sessionCookieName``. Se nenhum token for encontrado no cookie, ele verifica o cabeçalho ``Authorization``.

O decorador adiciona o middleware de autenticação e autorização aos metadados da rota. Esse middleware é processado antes da execução do manipulador de rota, garantindo que usuários não autorizados sejam bloqueados antes que qualquer lógica de negócios seja alcançada.

## Registro de Usuário

A partir da versão `0.8.10`, o módulo `@cmmv/auth` foi redesenhado para incluir controladores, serviços e gateways dentro do próprio módulo, eliminando a necessidade de geração de código. Basta adicionar o módulo à sua aplicação para habilitar automaticamente todas as funcionalidades de autenticação.

O registro de usuários e o login local devem ser explicitamente habilitados no arquivo de configuração (`.cmmv.config.cjs`) usando as opções `auth.localRegister` e `auth.localLogin`. Se esses parâmetros estiverem desabilitados, apenas a autenticação via provedores externos estará disponível.

Endpoint: `/auth/register`
Método: `POST`
Formato do Payload:

```typescript
{
    username: string;
    password: string;
}
```

Registro em modo administrador

```typescript
{
    username: string;
    password: string;
    provider?: string;
    groups?: object | string | string[] | ObjectId;
    roles?: string[];
    root: boolean;
    blocked: boolean;
    validated: boolean;
    verifyEmail: boolean;
    verifyEmailCode?: number;
    verifySMS: boolean;
    verifySMSCode?: number;
    optSecret?: string;
    optSecretVerify: boolean;
    profile?: string;
}
```

## Login

Endpoint: `/auth/login`
Método: `POST`
Formato do Payload:

```typescript
{
    username: string;
    password: string;
    token?: string;
    opt?: string;
}
```
<br/>

* `token:` Obrigatório se a validação reCAPTCHA estiver habilitada (`auth.recaptcha.required = true`).
* `opt:` Um código de segurança gerado por Google Authenticator ou serviço similar, se a Autenticação de Dois Fatores (2FA) estiver habilitada para o usuário.

Formato da Resposta:

```typescript
{
    "status": 200,
    "processingTime": 13,
    "result": {
        "success": true,
        "token": "eyJhbGciOiJIUzI1NiI...",
        "refreshToken": "eyJhbGciOiJIUzI1...",
        "message": "Login bem-sucedido",
        "actions": "opt-validate"
    }
}
```

### Token de Acesso

O campo `token` contém o token de autenticação, válido por 15 minutos.

* Esse token deve ser incluído no cabeçalho `Authorization` (`Authorization: Bearer <token>`) para todas as requisições de API que exigem autenticação.
* Se o token expirar, um novo deve ser obtido usando o token de atualização.

### Token de Atualização

O `refreshToken` é usado para renovar o token de acesso sem exigir que o usuário faça login novamente.

Para atualizar um token expirado, envie uma requisição para:

Endpoint: `/auth/refresh`
Método: `POST`
Cabeçalhos: `Authorization: Bearer <expired_access_token>`

Normalmente, os navegadores enviam automaticamente o token de atualização via cookies. No entanto, se os cookies não forem usados devido às configurações da aplicação, o token de atualização pode ser enviado manualmente via cabeçalho `RefreshToken`:

Cabeçalhos (método alternativo):

```text
Authorization: Bearer <expired_access_token>
RefreshToken: <refresh_token>
```

Se o token de atualização for válido e tiver sido criado pelo mesmo dispositivo que realizou o login, a API retornará um novo token de acesso com validade de 15 minutos.

Se você tentar usar o token de acesso ou o token de atualização em uma ferramenta como o Postman e depois tentar usar o token atualizado em um navegador ou outro dispositivo, a requisição será negada.

O sistema de autenticação vincula a sessão ao dispositivo e considera o User-Agent da requisição, garantindo maior segurança contra roubo de tokens e sequestro de sessão.

## reCAPTCHA

Se a validação reCAPTCHA estiver habilitada (`auth.recaptcha.required`), a API do Google reCAPTCHA será usada para verificação.

A chave `secret` deve ser armazenada no arquivo `.env` e definida em `auth.recaptcha.secret`:

```env
RECAPTCHA_SECRET=sua-chave-secreta
```

Mais detalhes e geração de chaves podem ser encontrados em: [Google reCAPTCHA Admin](https://www.google.com/recaptcha/admin/create)

## Autenticação de Dois Fatores (2FA)

Se um usuário tiver a Autenticação de Dois Fatores (2FA) (`optSecret`) habilitada, ele deverá fornecer um código de verificação gerado por um aplicativo autenticador (por exemplo, Google Authenticator) durante o login.

Para tornar a 2FA obrigatória para todos os usuários, habilite `auth.optSecret.required` no arquivo de configuração.

### QR Code

Se a resposta de login indicar que o registro de 2FA é necessário, você pode solicitar um QR code para escaneamento por um aplicativo de autenticação usando o seguinte endpoint:

Endpoint: `/auth/opt-qrcode`
Método: `GET`
Cabeçalhos: `Authorization: Bearer <login_token>`
Resposta: Uma imagem de QR code codificada em Base64, que pode ser exibida diretamente em uma tag IMG.

### Ativando 2FA para um Usuário

Para habilitar a Autenticação de Dois Fatores para um usuário:

Endpoint: `/auth/opt-enable`
Método: `POST`
Cabeçalhos: `Authorization: Bearer <login_token>`
Payload:

```typescript
{
    "secret": "código-gerado-pelo-usuário", // Código gerado pelo aplicativo autenticador
    "token": "token-recaptcha-opcional" // Obrigatório se reCAPTCHA estiver habilitado
}
```

Se a ativação for bem-sucedida, um código de verificação será necessário para logins futuros em dispositivos não registrados. Se o dispositivo já estiver validado, o código não será necessário.

### Validando 2FA para uma Nova Sessão

Para validar uma nova sessão usando 2FA:

Endpoint: `/auth/opt-validate`
Método: `POST`
Cabeçalhos: `Authorization: Bearer <login_token>`
Payload:

```typescript
{
    "secret": "código-gerado-pelo-usuário", // Código gerado pelo aplicativo autenticador
    "token": "token-recaptcha-opcional" // Obrigatório se reCAPTCHA estiver habilitado
}
```

### Desativando 2FA para um Usuário

Se um usuário deseja desativar a Autenticação de Dois Fatores, ele pode fazer uma requisição para:

Endpoint: `/auth/opt-remove`
Método: `DELETE`
Cabeçalhos: `Authorization: Bearer <login_token>`

## Recomendações de Segurança

O módulo de autenticação foi projetado como um componente fundamental para o Sistema IAM do CMMV, que será desenvolvido em breve. Se você optar por gerenciar o controle de sessão de forma independente, recomendamos fortemente seguir estas melhores práticas de segurança:

### Transmissão Segura (HTTPS)
<br/>

* Sempre use HTTPS para requisições de registro e login.
* Isso previne ataques Man-in-the-Middle (MitM), garantindo que credenciais e tokens sejam criptografados durante a transmissão.

### Implementar Proteção reCAPTCHA
<br/>

* Obtenha chaves reCAPTCHA gratuitas e registre-as no sistema.
* Habilite a validação reCAPTCHA para login e registro configurando
* Isso ajuda a mitigar ataques de força bruta automatizados.

### Gerenciamento Seguro de Chaves JWT

Use chaves separadas e fortemente geradas para `jwtSecret` e `jwtSecretRefresh`

```text
JWT_SECRET=chave_segura_aleatoria_1
JWT_SECRET_REFRESH=chave_segura_aleatoria_2
```
<br/>

* NÃO use a mesma chave para autenticação e tokens de atualização.
* Armazene essas chaves com segurança no arquivo `.env` e garanta chaves diferentes para ambientes de desenvolvimento e produção.

### Impor Autenticação de Dois Fatores (2FA)
<br/>

* Recomende fortemente tornar a 2FA obrigatória para todos os usuários
* Isso adiciona uma camada extra de segurança, prevenindo acesso não autorizado mesmo que uma senha seja comprometida.
* Os usuários precisarão verificar logins com Google Authenticator ou um aplicativo similar.

### Integração Futura com o Sistema IAM
<br/>

* Quando o Sistema IAM do CMMV for lançado, é altamente recomendado migrar para seu serviço centralizado de autenticação.
* Você poderá gerar chaves de API da aplicação a partir de [CMMV.io](https://cmmv.io), garantindo uma solução IAM segura, escalável e totalmente gerenciada.

Ao implementar essas medidas de segurança, seu sistema de autenticação será significativamente mais resiliente a ataques e estará alinhado às melhores práticas da indústria.

### Use Políticas de Senha Fortes
<br/>

* Imponha um comprimento mínimo de senha (por exemplo, pelo menos 12 caracteres).
* Exija uma combinação de letras maiúsculas, minúsculas, números e caracteres especiais.
* Impeça o uso de senhas comuns verificando contra um banco de dados de senhas comprometidas (por exemplo, Have I Been Pwned).
* Use limitação de taxa para prevenir ataques de força bruta em tentativas de login.

### Implementar Bloqueio de Conta

Se ocorrerem múltiplas tentativas de login falhas em um curto período:

* Bloqueie temporariamente a conta.
* Exija verificação reCAPTCHA antes de permitir novas tentativas de login.
* Envie uma notificação de segurança ao e-mail do usuário.
