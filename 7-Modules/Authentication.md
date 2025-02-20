# Autenticação

O módulo ``@cmmv/auth`` fornece um conjunto de funcionalidades para lidar com autenticação em sua aplicação. Ele suporta autenticação baseada em HTTP e WebSocket e pode ser facilmente integrado a qualquer aplicação baseada em ``@cmmv``.

## Instalação

Para instalar o módulo ``@cmmv/auth``, execute o seguinte comando:

```typescript
$ pnpm add @cmmv/auth
```

## Integração

Após instalar, você pode integrar o módulo ``@cmmv/auth`` em sua aplicação conforme mostrado abaixo. Este exemplo demonstra a configuração básica de uma aplicação CMMV que inclui o módulo ``@cmmv/auth`` para lidar com autenticação.

```typescript
require('dotenv').config();

import { Application } from '@cmmv/core';
import { ExpressAdapter, ExpressModule } from '@cmmv/http';
import { ProtobufModule } from '@cmmv/protobuf';
import { WSModule, WSAdapter } from '@cmmv/ws';
import { ViewModule } from '@cmmv/view';
...
import { AuthModule } from '@cmmv/auth';

Application.create({
    httpAdapter: ExpressAdapter,
    wsAdapter: WSAdapter,
    modules: [
        ExpressModule,
        ProtobufModule,
        WSModule,
        ViewModule,
        RepositoryModule,
        AuthModule,  // Adicione o AuthModule
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

O arquivo ``.cmmv.config.cjs`` é o arquivo de configuração central da sua aplicação CMMV, permitindo configurar diferentes definições relacionadas ao servidor, autenticação e outros módulos. Abaixo está uma explicação detalhada das opções de configuração relevantes para o módulo ``@cmmv/auth``:

```javascript
module.exports = {
    env: process.env.NODE_ENV,

    server: {
        session: {
            enabled: true,  // Habilitar suporte a sessão
            options: {
                // Nome do cookie de sessão
                sessionCookieName: "cmmv-session", 
                // Chave secreta para assinar o ID do cookie de sessão
                secret: process.env.SESSION_SECRET, 
                // Impede que a sessão seja salva novamente no armazenamento
                resave: false, 
                // Força salvar a sessão, mesmo que não esteja inicializada
                saveUninitialized: false, 
                cookie: { 
                    // Garante que o navegador envie o cookie apenas via HTTPS
                    secure: true,  
                    // Tempo máximo (em milissegundos) para o cookie de sessão
                    maxAge: 60000  
                }
            }
        }
    },

    // Configurações de autenticação para @cmmv/auth
    auth: {
        // Habilitar registro local (e-mail/senha)
        localRegister: true,  
        // Habilitar login local (e-mail/senha)
        localLogin: true,     
        // Chave secreta para assinar tokens JWT
        jwtSecret: process.env.JWT_SECRET || "secret", 
        // Chave secreta para assinar o token de atualização JWT
        jwtSecretRefresh: process.env.JWT_SECRET_REFRESH,  
        // Nome do cookie do token de atualização
        refreshCookieName: "refreshToken",   
        // Tempo de expiração do token em segundos (1 hora)
        expiresIn: 60 * 60,  
        qrCode: {} //Veja o tópico do QR-Code 
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

O módulo ``@cmmv/auth`` é responsável por gerar os arquivos necessários para autenticação em uma aplicação baseada no CMMV. Isso inclui arquivos ``.proto``, definições TypeScript, implementações de serviços e controladores. Caso o módulo ``@cmmv/ws`` esteja instalado, ele também gerará o gateway WebSocket para lidar com eventos de autenticação. Abaixo estão os principais componentes gerados por este módulo:

**``.generated/protos/auth/user.proto``**

Este arquivo ``.proto`` define as mensagens ``User``, ``LoginRequest``, ``LoginResponse``, ``RegisterRequest``, ``RegisterResponse`` e o serviço ``AuthService`` com dois métodos RPC: Login e Register.

```proto
/**                                                                               
    **********************************************
    This script was generated automatically by CMMV.
    It is recommended not to modify this file manually, 
    as it may be overwritten by the application.
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

Este arquivo fornece interfaces TypeScript que espelham as definições ``.proto``, facilitando o trabalho com as estruturas de dados na sua aplicação.

```typescript
/**                                                                               
    **********************************************
    This script was generated automatically by CMMV.
    It is recommended not to modify this file manually, 
    as it may be overwritten by the application.
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

## Decorador @Auth

O decorador ``@Auth`` é projetado para aplicar autenticação e autorização em rotas específicas dentro de uma aplicação. Ele fornece uma funcionalidade middleware que verifica se um usuário está autenticado, validando um JWT (JSON Web Token) e, opcionalmente, verificando se o usuário possui as funções necessárias para acessar a rota.

* **Localização do Token:** O middleware verifica primeiro o token JWT nos cookies da requisição (usando um nome configurável) ou no cabeçalho ``Authorization``. Se nenhum token for encontrado, retorna uma resposta ``401 Unauthorized``.
* **Validação do Token:** Se um token for encontrado, ele é verificado usando o segredo armazenado na configuração (``auth.jwtSecret``). Caso o token seja inválido ou expirado, o middleware retorna uma resposta ``401 Unauthorized``.
* **Validação de Funções:** Se as funções (``roles``) forem especificadas, o middleware verifica se o payload do token inclui as funções necessárias. Caso contrário, retorna uma resposta ``403 Forbidden``.
* **Informações do Usuário:** Após a validação bem-sucedida, o token decodificado (geralmente contendo informações do usuário, como ID e nome de usuário) é anexado ao objeto ``req.user`` para uso posterior no manipulador de rota.

```typescript
import { Controller, Get, Req } from '@cmmv/http';
import { Auth } from '@cmmv/auth';

@Controller('user')
export class UserController {
    // Protege a rota, verifica se o JWT é válido
    @Get('profile')
    @Auth()  
    async getProfile(@Req() req) {
        return req.user;  // Acessa as informações do usuário autenticado
    }

    // Protege a rota, verifica se o JWT é válido e se o usuário é admin
    @Get('admin')
    @Auth(['admin'])  
    async getAdminDashboard(@Req() req) {
        return `Bem-vindo(a), admin \${req.user.username}`;
    }
}
```

<br/>

* **``roles?:``** (Opcional): Um array de funções que o usuário deve possuir para acessar a rota. Se nenhuma função for fornecida, apenas a autenticação é necessária (ou seja, um JWT válido). Caso contrário, o usuário deve possuir pelo menos uma das funções especificadas.

```typescript
@Auth(['admin', 'moderator']) 
```

**Dependências de Configuração**

* **Segredo do JWT:** O decorador usa o segredo especificado na configuração (``auth.jwtSecret``) para verificar o token.
* **Nome do Cookie de Sessão:** Por padrão, o token é verificado em um cookie com o nome especificado em ``server.session.options.sessionCookieName``. Se nenhum token for encontrado no cookie, ele verifica o cabeçalho ``Authorization``.

O decorador adiciona o middleware de autenticação e autorização aos metadados da rota. Este middleware é processado antes do manipulador da rota ser executado, garantindo que usuários não autorizados sejam bloqueados antes que qualquer lógica de negócio seja alcançada.

# User Registration

A partir da versão `0.8.10`, o módulo `@cmmv/auth` foi redesenhado para incluir controladores, serviços e gateways dentro do próprio módulo, eliminando a necessidade de geração de código. Apenas adicionar o módulo à sua aplicação já ativa todas as funcionalidades de autenticação automaticamente.

O registro de usuários e o login local devem ser explicitamente ativados no arquivo de configuração (`.cmmv.config.cjs`) utilizando as opções `auth.localRegister` e `auth.localLogin`. Caso esses parâmetros estejam desativados, apenas a autenticação via provedores externos estará disponível.

Endpoint: `/auth/register`
Method: `POST`
Payload Format:

```typescript
{
    username: string;
    password: string;
}
```

Registro no modo de administrador

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
Method: `POST`
Payload Format:

```typescript
{ 
    username: string;
    password: string;
    token?: string;
    opt?: string; 
}
```
<br/>

* `token:` Necessário caso a validação do reCAPTCHA esteja ativada (`auth.recaptcha.required = true`).
* `opt:` Um código de segurança gerado pelo Google Authenticator ou um serviço similar, caso a Autenticação de Dois Fatores (2FA) esteja ativada para o usuário.

Response Format:

```typescript
{
    "status": 200,
    "processingTime": 13,
    "result": {
        "success": true,
        "token": "eyJhbGciOiJIUzI1NiI...",
        "refreshToken": "eyJhbGciOiJIUzI1...",
        "message": "Login successful"
        "actions": "opt-validate"
    }
}
```

### Token de acesso

O campo `token` contém o token de autenticação, que é válido por 15 minutos.

* Este token deve ser incluído no cabeçalho `Authorization` (`Authorization: Bearer <token>`) para todas as solicitações de API que exigem autenticação.
* Se o token expirar, um novo deve ser obtido usando o token de atualização.

### Token de atualização

O `refreshToken` é usado para renovar o token de acesso sem exigir que o usuário faça login novamente.

Para atualizar um token expirado, envie uma solicitação para:

Endpoint: `/auth/refresh`
Método: `POST`
Cabeçalhos: `Authorization: Bearer <expired_access_token>`

Normalmente, os navegadores enviam automaticamente o token de atualização por meio de cookies. No entanto, se os cookies não forem usados ​​devido às configurações do aplicativo, o token de atualização pode ser enviado manualmente por meio do cabeçalho `RefreshToken`:

Headers (método alternativo):

```text
Authorization: Bearer <expired_access_token>
RefreshToken: <refresh_token>
```

Se o token de atualização for válido e tiver sido criado pelo mesmo dispositivo que realizou o login, a API retornará um novo token de acesso com validade de 15 minutos.

Se você tentar usar o token de acesso ou o token de atualização de uma ferramenta como o Postman e, em seguida, tentar usar o token atualizado em um navegador ou outro dispositivo, a solicitação será negada.

O sistema de autenticação vincula a sessão ao dispositivo e considera o User-Agent da solicitação, garantindo segurança aprimorada contra roubo de token e sequestro de sessão.

## reCAPTCHA

Se a validação do reCAPTCHA estiver ativada (`auth.recaptcha.required`), a API do Google reCAPTCHA será utilizada para verificação.

A chave `secret` deve ser armazenada no arquivo `.env` e configurada na opção `auth.recaptcha.secret`:

```env
RECAPTCHA_SECRET=your-secret-key
```

Mais detalhes e a geração de chaves podem ser encontrados em: [Google reCAPTCHA Admin](https://www.google.com/recaptcha/admin/create)

## Dois Fatores (2FA)

Se um usuário tiver a Autenticação de Dois Fatores (2FA) (`optSecret`) ativada, será necessário fornecer um código de verificação gerado por um aplicativo autenticador (por exemplo, Google Authenticator) durante o login.

Para tornar o 2FA obrigatório para todos os usuários, ative a opção `auth.optSecret.required` no arquivo de configuração.

### QR Code

Se a resposta do login indicar que o registro do 2FA é necessário, você pode solicitar um QR code para escaneamento via um aplicativo autenticador utilizando o seguinte endpoint:

Endpoint: `/auth/opt-qrcode`
Método: `GET`
Cabeçalhos: `Authorization: Bearer <login_token>`
Resposta: Uma imagem QR code codificada em Base64, que pode ser exibida diretamente em uma tag IMG.

### Activating 2FA for a User

Para ativar a Autenticação de Dois Fatores (2FA) para um usuário:

Endpoint: `/auth/opt-enable`
Método: `POST`
Cabeçalhos: `Authorization: Bearer <login_token>`
Payload:

```typescript
{
    "secret": "user-generated-code", // Código gerado pelo aplicativo autenticador
    "token": "optional-recaptcha-token" // Código do reCAPTCHA (se obrigatório)
}
```

Se a ativação for bem-sucedida, nos próximos logins será solicitado um código de autenticação para sessões sem um dispositivo registrado. Se o dispositivo já estiver validado, o código não será necessário.

### Validating 2FA for a New Session

Para validar uma nova sessão usando 2FA:

Endpoint: `/auth/opt-validate`
Method: `POST`
Headers: `Authorization: Bearer <login_token>`
Payload:

```typescript
{
    "secret": "user-generated-code", // Código gerado pelo aplicativo autenticador
    "token": "optional-recaptcha-token" // Código do reCAPTCHA (se obrigatório)
}
```

### Disabling 2FA for a User

Se um usuário quiser desabilitar a autenticação de dois fatores, ele pode fazer uma solicitação para:

Endpoint: `/auth/opt-remove`
Method: `DELETE`
Headers: `Authorization: Bearer <login_token>`

## Recomendações de segurança

O módulo de autenticação é projetado como um componente fundamental para o CMMV IAM System, que será desenvolvido em breve. Se você escolher gerenciar o controle de sessão de forma independente, recomendamos fortemente seguir estas melhores práticas de segurança:

### Transmissão Segura (HTTPS)

* Sempre use HTTPS para solicitações de registro e login.
* Isso previne ataques Man-in-the-Middle (MitM), garantindo que credenciais e tokens sejam criptografados durante a transmissão.

### Implementar proteção reCAPTCHA

* Obtenha chaves reCAPTCHA gratuitas e registre-as no sistema.
* Habilite a validação reCAPTCHA para login e registro configurando
* Isso ajuda a mitigar ataques de força bruta automatizados.

### Gerenciamento seguro de chaves JWT

Use chaves separadas e fortemente geradas para `jwtSecret` e `jwtSecretRefresh`

```text
JWT_SECRET=secure_random_key_1
JWT_SECRET_REFRESH=secure_random_key_2
```

* NÃO use a mesma chave para autenticação e tokens de atualização.
* Armazene essas chaves com segurança no arquivo `.env` e garanta chaves diferentes para ambientes de desenvolvimento e produção.

### Aplicar autenticação de dois fatores (2FA)

* Recomendamos fortemente tornar o 2FA obrigatório para todos os usuários
* Isso adiciona uma camada extra de segurança, impedindo acesso não autorizado mesmo se uma senha for comprometida.
* Os usuários precisarão verificar os logins com o Google Authenticator ou um aplicativo semelhante.

### Integração futura do sistema IAM

* Assim que o CMMV IAM System for lançado, é altamente recomendável mudar para seu serviço de autenticação centralizado.
* Você poderá gerar chaves de API de aplicativo a partir do [CMMV.io](https://cmmv.io), garantindo uma solução IAM totalmente gerenciada, segura e escalável.

Ao implementar essas medidas de segurança, seu sistema de autenticação será significativamente mais resiliente a ataques e alinhado com as melhores práticas do setor.

### Use políticas de senhas fortes

* Imponha um comprimento mínimo de senha (por exemplo, pelo menos 12 caracteres).
* Exija uma combinação de letras maiúsculas, minúsculas, números e caracteres especiais.
* Evite o uso de senhas comuns verificando em um banco de dados de senhas violadas (por exemplo, Have I Been Pwned).
* Use limitação de taxa para evitar ataques de força bruta em tentativas de login.

### Implementar bloqueio de conta

Se várias tentativas de login com falha ocorrerem em um curto período de tempo:

* Bloqueie temporariamente a conta.
* Exija verificação do reCAPTCHA antes de permitir novas tentativas de login.
* Envie uma notificação de segurança para o e-mail do usuário.