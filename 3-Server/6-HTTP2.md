# HTTP2

O framework `@cmmv/server` suporta nativamente HTTP/2, oferecendo melhorias significativas de desempenho em relação ao HTTP/1.1, permitindo streams multiplexados, compressão de cabeçalhos e server push. Esses recursos são ideais para aplicações que precisam de tempos de carregamento mais rápidos e melhor utilização de recursos, especialmente em aplicações web modernas.

O HTTP/2 é particularmente útil para reduzir a latência, melhorar o desempenho de carregamento de páginas e aumentar a eficiência geral da comunicação cliente-servidor.

## Habilitando HTTP/2

Para habilitar o HTTP/2, basta configurar o servidor com a opção `http2` definida como `true`. Além disso, será necessário fornecer certificados SSL, pois o HTTP/2 exige HTTPS para suporte em navegadores.

### Exemplo com HTTP/2

```typescript
import { readFileSync } from 'node:fs';
import cmmv from '@cmmv/server';

const app = cmmv({
    http2: true, // Habilita HTTP/2
    https: {
        key: readFileSync('./cert/private-key.pem'),  
        cert: readFileSync('./cert/certificate.pem'), 
        passphrase: '1234'                           
    }
});

const host = '0.0.0.0';
const port = 3000;

app.get('/', (req, res) => {
    res.send('Hello HTTP/2 World!');
});

app.listen({ host, port })
.then(server => {
    console.log(
        `Servidor HTTP/2 rodando em https://\${server.address().address}:\${server.address().port}`
    );
})
.catch(err => {
    throw new Error(err.message);
});
```

A negociação [ALPN](https://datatracker.ietf.org/doc/html/rfc7301) permite suporte para HTTPS e HTTP/2 no mesmo socket. Os objetos `req` e `res` do Node podem ser tanto HTTP/1 quanto HTTP/2:

```typescript
const app = cmmv({
    http2: true, // Habilita HTTP/2
    https: {
        allowHTTP1: true, // Suporte fallback para HTTP1
        key: readFileSync('./cert/private-key.pem'),  
        cert: readFileSync('./cert/certificate.pem'), 
        passphrase: '1234'                           
    }
});
```

Você pode testar seu novo servidor com:

```bash
$ npx h2url https://localhost:3000
```

## Certificado SSL Autoassinado

Para testar HTTP/2 localmente com `@cmmv/server`, é necessário um certificado SSL, já que navegadores exigem HTTPS para suporte ao HTTP/2. Veja como gerar um certificado autoassinado para o seu ambiente localhost.

### Passos para Criar um Certificado SSL Autoassinado

Você pode usar o OpenSSL, uma ferramenta gratuita e open-source, para gerar os arquivos SSL necessários (chave privada e certificado) para desenvolvimento local. Siga os passos abaixo:

#### **1. Instale o OpenSSL**

Caso não tenha o OpenSSL instalado, você pode baixá-lo aqui ou instalá-lo via gerenciador de pacotes como `brew` no macOS ou `apt-get` no Linux.

No macOS:

```bash
brew install openssl
```

No Linux (Ubuntu/Debian):

```bash
sudo apt-get install openssl
```

#### **2. Gere uma Chave Privada**

Primeiro, crie uma chave privada que será usada para assinar o certificado SSL.

```bash
openssl genpkey -algorithm RSA -out private-key.pem -aes256
```

#### **3. Crie um Certificado Autoassinado**

Agora que você tem a chave privada, pode criar o certificado SSL.

```bash
openssl req -new -x509 -key private-key.pem -out certificate.pem -days 365
```

Esse comando gerará um certificado autoassinado chamado `certificate.pem`, válido por 365 dias.

#### **4. Ignorar Avisos do Navegador (Opcional)**

Como o certificado é autoassinado, a maioria dos navegadores exibirá um aviso de segurança. Para ignorar isso:

- **Chrome:** Navegue para `chrome://flags/#allow-insecure-localhost` e habilite a flag que permite localhost inseguro.
- **Firefox:** Clique em "Avançado" e depois em "Adicionar Exceção" quando o aviso aparecer.

#### **5. Use o Certificado no `@cmmv/server`**

Agora, você pode usar os arquivos gerados `private-key.pem` e `certificate.pem` na configuração do `@cmmv/server`, conforme mostrado abaixo:

```typescript
import { readFileSync } from 'node:fs';
import cmmv from '@cmmv/server';

const app = cmmv({
    http2: true,
    https: {
        key: readFileSync('./cert/private-key.pem'),
        cert: readFileSync('./cert/certificate.pem'),
        passphrase: 'sua-senha'
    }
});

app.get('/', (req, res) => {
    res.send('Hello, HTTP/2 with HTTPS!');
});

app.listen({ host: '0.0.0.0', port: 3000 })
.then(server => {
    console.log('Servidor HTTP/2 rodando em https://localhost:3000');
})
.catch(err => {
    console.error('Erro ao iniciar o servidor:', err);
});
```

Agora seu servidor local estará rodando com HTTP/2 e criptografia SSL!

## HTTP2 em Texto Simples ou Inseguro

Se você estiver criando microsserviços, pode se conectar ao HTTP2 em texto simples. No entanto, isso não é suportado por navegadores.

```typescript
import cmmv from '@cmmv/server';

const app = cmmv({ http2: true });

app.get('/', (req, res) => {
    res.send('Hello, HTTP/2 without HTTPS!');
});

app.listen({ host: '0.0.0.0', port: 3000 })
.then(server => {
    console.log('Servidor HTTP/2 rodando em http://localhost:3000');
})
.catch(err => {
    console.error('Erro ao iniciar o servidor:', err);
});
```

Você pode testar seu novo servidor com:

```bash
$ npx h2url http://localhost:3000
```