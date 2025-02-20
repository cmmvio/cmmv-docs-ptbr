# Email

O módulo `@cmmv/email` fornece uma interface unificada para o envio de e-mails em aplicações baseadas no CMMV, utilizando [nodemailer](https://nodemailer.com/) para transporte de e-mails e integração opcional com AWS SES.

## Recursos

* **Suporte a SMTP e AWS SES:** Envie e-mails via SMTP padrão ou Amazon SES.

* **Integração Transparente:** Funciona perfeitamente com o ecossistema CMMV.

* **Opções de Transporte:** Opções de transporte personalizáveis via configuração.

* **Suporte a Logs:** Ative o registro de logs de e-mails para fins de depuração.

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
        logger: true,  // Ativar logs
        debug: true,   // Ativar modo de depuração
        SES: {
            region: process.env.AWS_REGION,
            accessKeyId: process.env.AWS_ACCESS_KEY_ID,
            secretAccessKey: process.env.AWS_SECRET_ACCESS_KEY
        }
    }
};
```

## Configurando a Aplicação

No seu arquivo `index.ts`, inclua o `EmailModule` na aplicação:

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

O `EmailService` pode ser integrado com o módulo `@cmmv/queue` para processamento assíncrono de e-mails.

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

Se as configurações do AWS SES forem fornecidas na configuração, o módulo usará automaticamente o AWS SES como provedor de e-mail.

```javascript
SES: {
    region: "us-east-1",
    accessKeyId: "YOUR_AWS_ACCESS_KEY",
    secretAccessKey: "YOUR_AWS_SECRET_KEY"
}
```