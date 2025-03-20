# Email

Repositório: [https://github.com/cmmvio/cmmv-email](https://github.com/cmmvio/cmmv-email)

O módulo `@cmmv/email` fornece uma interface unificada para enviar emails em aplicações baseadas em CMMV, utilizando o [nodemailer](https://nodemailer.com/) para transporte de email e integração opcional com AWS SES.

## Recursos

* **Suporte a SMTP e AWS SES:** Envie emails via SMTP padrão ou Amazon SES.

* **Integração Perfeita:** Funciona com o ecossistema CMMV.

* **Opções de Transporte:** Opções de transporte personalizáveis via configuração.

* **Suporte a Registro de Logs:** Ative ou desative o registro de logs de email para fins de depuração.

## Instalação

Instale o pacote `@cmmv/email` via npm:

```bash
$ pnpm add @cmmv/email
```

## Configuração

O módulo `@cmmv/email` requer configurações para integração com SMTP ou AWS SES. Essas configurações podem ser definidas no arquivo `.cmmv.config.cjs`:

```javascript
module.exports = {
    email: {
        host: process.env.EMAIL_HOST || "smtp.mailtrap.io",
        port: process.env.EMAIL_PORT || 587,
        secure: false,  // Defina como true para SSL
        auth: {
            user: process.env.EMAIL_USER,
            pass: process.env.EMAIL_PASS,
        },
        logger: true,  // Habilita o registro de logs
        debug: true,   // Habilita o modo de depuração
        SES: {
            region: process.env.AWS_REGION,
            accessKeyId: process.env.AWS_ACCESS_KEY_ID,
            secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
        }
    }
};
```

## Configurando a Aplicação

No seu arquivo ``index.ts``, inclua o ``EmailModule`` na aplicação:

```typescript
import { Application } from "@cmmv/core";
import { DefaultAdapter, DefaultHTTPModule } from "@cmmv/http";
import { EmailModule } from "@cmmv/email";

Application.create({
    httpAdapter: DefaultAdapter,
    modules: [
        DefaultHTTPModule,
        EmailModule,
    ],
});
```

## Usando o EmailService

O `EmailService` pode ser integrado ao módulo `@cmmv/queue` para processamento assíncrono de emails.

```typescript
import { Channel, Consume, QueueMessage } from "@cmmv/queue";
import { EmailService } from "@cmmv/email";

@Channel("email-queue")
export class EmailConsumer {
    constructor(private readonly emailService: EmailService) {}

    @Consume("send-email")
    public async onSendEmail(@QueueMessage() message) {
        await this.emailService.send(
            message.from,
            message.to,
            message.subject,
            message.html,
            message.options
        );
        console.log("Email enviado com sucesso");
    }
}
```

## Integração com AWS SES

Se as configurações do AWS SES forem fornecidas na configuração, o módulo usará automaticamente o AWS SES como provedor de email.

```javascript
SES: {
    region: "us-east-1",
    accessKeyId: "SUA_CHAVE_DE_ACESSO_AWS",
    secretAccessKey: "SUA_CHAVE_SECRETA_AWS"
}
```
