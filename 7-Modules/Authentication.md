# Autenticação

O módulo ``@cmmv/auth`` fornece um conjunto de funcionalidades para lidar com autenticação em sua aplicação. Ele suporta autenticação baseada em HTTP e WebSocket e pode ser facilmente integrado a qualquer aplicação baseada em ``@cmmv``.

## Instalação

Para instalar o módulo ``@cmmv/auth``, execute o seguinte comando:

```typescript
$ pnpm add @cmmv/auth jsonwebtoken
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

O arquivo ``.cmmv.config.js`` é o arquivo de configuração central da sua aplicação CMMV, permitindo configurar diferentes definições relacionadas ao servidor, autenticação e outros módulos. Abaixo está uma explicação detalhada das opções de configuração relevantes para o módulo ``@cmmv/auth``:

```javascript
module.exports = {
    env: process.env.NODE_ENV,

    server: {
        session: {
            enabled: true,  // Habilitar suporte a sessão
            options: {
                // Nome do cookie de sessão
                sessionCookieName: 
                    process.env.SESSION_COOKIENAME || "cmmv-session", 
                // Chave secreta para assinar o ID do cookie de sessão
                secret: process.env.SESSION_SECRET || "secret", 
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
        // Tempo de expiração do token em segundos (1 hora)
        expiresIn: 60 * 60,  
        
        google: {
            // ID do cliente OAuth do Google
            clientID: process.env.GOOGLE_CLIENT_ID,  
            // Segredo do cliente OAuth do Google
            clientSecret: process.env.GOOGLE_CLIENT_SECRET,  
            // URL para redirecionamento após login no Google
            callbackURL: "http://localhost:3000/auth/google/callback"  
        }
    },
};
```

## Gerados

O módulo ``@cmmv/auth`` é responsável por gerar os arquivos necessários para autenticação em uma aplicação baseada no CMMV. Isso inclui arquivos ``.proto``, definições TypeScript, implementações de serviços e controladores. Caso o módulo ``@cmmv/ws`` esteja instalado, ele também gerará o gateway WebSocket para lidar com eventos de autenticação. Abaixo estão os principais componentes gerados por este módulo:

**``src/protos/auth.proto``**

Este arquivo ``.proto`` define as mensagens ``User``, ``LoginRequest``, ``LoginResponse``, ``RegisterRequest``, ``RegisterResponse`` e o serviço ``AuthService`` com dois métodos RPC: Login e Register.

```proto
// Gerado automaticamente pelo CMMV

syntax = "proto3";
package auth;

message User {
   string username = 1;
   string password = 2;
   string googleId = 3;
   string groups = 4;
}

message LoginRequest {
    string username = 1;
    string password = 2;
}

message LoginResponse {
    bool success = 1;
    string token = 2;
    string message = 3;
}

message RegisterRequest {
    string username = 1;
    string email = 2;
    string password = 3;
}

message RegisterResponse {
    bool success = 1;
    string message = 3;
}
          
service AuthService {
    rpc Login (LoginRequest) returns (LoginResponse);
    rpc Register (RegisterRequest) returns (RegisterResponse);
}
```

<br/>

**``src/protos/auth.d.ts``**

Este arquivo fornece interfaces TypeScript que espelham as definições ``.proto``, facilitando o trabalho com as estruturas de dados na sua aplicação.

```typescript
// Gerado automaticamente pelo CMMV

export namespace User {
    export type username = string;
    export type password = string;
    export type googleId = string;
    export type groups = string;
}

export interface LoginRequest {
    username: string;
    password: string;
}

export interface LoginResponse {
    success: boolean;
    token: string;
    message: string;
}

export interface RegisterRequest {
    username: string;
    email: string;
    password: string;
}

export interface RegisterResponse {
    success: boolean;
    message: string;
}
```

<br/>

**``src/services/auth.services.ts``**

O serviço lida com a lógica central de autenticação do usuário, como login e registro. Ele utiliza `class-validator` para validação e `class-transformer` para transformar os dados recebidos em um objeto `User`.

```typescript
// Gerado automaticamente pelo CMMV
    
import * as jwt from 'jsonwebtoken';
import { validate } from 'class-validator';
import { plainToClass } from 'class-transformer';

import { 
    Telemetry, Service, 
    AbstractService, Config
} from "@cmmv/core";

import { Repository } from '@cmmv/repository';

import { User, IUser } from '../models/user.model';

import { 
    LoginRequest, LoginResponse, 
    RegisterRequest, RegisterResponse 
} from '../protos/auth';

import { UserEntity } from '../entities/user.entity';

@Service("auth")
export class AuthService extends AbstractService {

    public async login(
        payload: LoginRequest, 
        req?: any, res?: any, 
        session?: any
    ): Promise<{ result: LoginResponse, user: any }> {
        Telemetry.start('AuthService::login', req?.requestId);

        const jwtToken = Config.get<string>("auth.jwtSecret");
        const expiresIn = Config.get<number>("auth.expiresIn", 60 * 60);
        const sessionEnabled = Config.get<boolean>(
            "server.session.enabled", true
        );
        const cookieName = Config.get<string>(
            "server.session.options.sessionCookieName", 
            "token"
        );
        const cookieTTL = Config.get<number>(
            "server.session.options.cookie.maxAge", 
            24 * 60 * 60 * 100
        );
        const cookieSecure = Config.get<boolean>(
            "server.session.options.cookie.secure", 
            process.env.NODE_ENV !== 'dev'
        );

        const userValidation = plainToClass(User, payload, { 
            exposeUnsetFields: true,
            enableImplicitConversion: true
        }); 

        const user = await Repository.findBy(UserEntity, userValidation);

        if (!user) {
            return { result: { 
                success: false, token: "", 
                message: "Credenciais inválidas" 
            }, user: null  };
        }
            
        const token = jwt.sign({ 
            id: user.id,
            username: payload.username 
        }, jwtToken, { expiresIn });

        res.cookie(cookieName, `Bearer \${token}`, {
            httpOnly: true,
            secure: cookieSecure,
            sameSite: 'strict',
            maxAge: cookieTTL
        });

        if(sessionEnabled){
            session.user = {
                username: payload.username,
                token: token,
            };
    
            session.save();
        }
        
        Telemetry.end('AuthService::login', req?.requestId);        
        return { result: { 
            success: true, token, 
            message: "Login realizado com sucesso" 
        }, user };
    }

    public async register(
        payload: RegisterRequest, req?: any
    ): Promise<RegisterResponse> {
        Telemetry.start('AuthService::register', req?.requestId);
        const jwtToken = Config.get("auth.jwtSecret");

        const newUser = plainToClass(User, payload, { 
            exposeUnsetFields: true,
            enableImplicitConversion: true
        }); 

        const errors = await validate(newUser, { 
            skipMissingProperties: true 
        });
        
        if (errors.length > 0) {
            console.error(errors);
            Telemetry.end('AuthService::register', req?.requestId);
            return { 
                success: false, 
                message: JSON.stringify(errors[0].constraints) 
            };
        } 
        else {    
            try{
                const result = await Repository.insert<UserEntity>(
                    UserEntity, newUser
                );

                Telemetry.end('AuthService::register', req?.requestId);

                return (result) ? 
                    { 
                        success: true, 
                        message: "Usuário registrado com sucesso!" 
                    } : 
                    { 
                        success: false, 
                        message: "Erro ao tentar registrar novo usuário" 
                    };
            }   
            catch(e){
                console.error(e);
                Telemetry.end('AuthService::register', req?.requestId);
                return { success: false, message: e.message };
            }                                                    
        }
    }
}
```

<br/>

**``src/controllers/auth.controller.ts``**

Este controlador é responsável por lidar com requisições HTTP relacionadas à autenticação. Ele inclui endpoints para login, registro e obtenção das informações do usuário atual.

```typescript
// Gerado automaticamente pelo CMMV
    
import { Config } from "@cmmv/core";
import { Auth } from "@cmmv/auth";

import { 
    Controller, Post, Body, Req, 
    Res, Get, Session
} from "@cmmv/http";

import { AuthService } from '../services/auth.service';

import { 
    LoginRequest, LoginResponse, 
    RegisterRequest, RegisterResponse 
} from '../protos/auth';

@Controller("auth")
export class AuthController {
    constructor(private readonly authService: AuthService) {}

    @Get("user")
    @Auth()  // Usa o decorator Auth para autorização
    async user(@Req() req) {
        return req.user;
    }

    @Post("login")
    async login(
        @Body() payload: LoginRequest, 
        @Req() req, @Res() res, @Session() session
    ): Promise<LoginResponse> {
        const { result } = await this.authService.login(
            payload, req, res, session
        );

        return result;
    }

    @Post("register")
    async register(
        @Body() payload: RegisterRequest
    ): Promise<RegisterResponse> {
        return await this.authService.register(payload);
    }
}
```

<br/>

**``src/gateways/auth.gateway.ts``**

Quando o módulo ``@cmmv/ws`` está instalado, um gateway WebSocket é gerado para lidar com eventos de autenticação em tempo real.

```typescript
// Gerado automaticamente pelo CMMV

import { Rpc, Message, Data, Socket, RpcUtils } from "@cmmv/ws";
import { AuthService } from '../services/auth.service';

import { 
    LoginRequest,
    RegisterRequest  
} from "../protos/auth";

@Rpc("auth")
export class AuthGateway {
    constructor(private readonly authService: AuthService) {}

    @Message("LoginRequest")
    async login(@Data() data: LoginRequest, @Socket() socket) {
        try {
            const { result } = await this.authService.login(data);
            const response = await RpcUtils.pack(
                "auth", "LoginResponse", result
            );

            if (response)
                socket.send(response);            
        } catch (e) {
            return null;
        }
    }

    @Message("RegisterRequest")
    async register(@Data() data: RegisterRequest, @Socket() socket) {
        try {
            const result = await this.authService.register(data);
            const response = await RpcUtils.pack(
                "auth", "RegisterResponse", result
            );

            if (response)
                socket.send(response);            
        } catch (e) {
            return null;
        }
    }
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
