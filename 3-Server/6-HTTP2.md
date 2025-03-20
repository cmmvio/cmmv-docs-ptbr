# HTTP/2

O framework ``@cmmv/server`` oferece suporte nativo ao HTTP/2, que proporciona melhorias significativas de desempenho em relação ao HTTP/1.1 ao permitir fluxos multiplexados, compressão de cabeçalhos e *server push*. Esses recursos são ideais para aplicações que necessitam de tempos de carregamento de página mais rápidos e melhor utilização de recursos, especialmente em aplicações web modernas.

O HTTP/2 é particularmente útil para reduzir a latência, melhorar o desempenho de carregamento de páginas e aumentar a eficiência geral da comunicação entre cliente e servidor.

### Habilitando o HTTP/2 no ``@cmmv/server``

Para habilitar o HTTP/2, você precisa configurar o servidor com a opção ``http2`` definida como ``true``. Além disso, será necessário fornecer certificados SSL, pois o HTTP/2 requer HTTPS para suporte em navegadores.

Aqui está um exemplo de como implementar o suporte ao HTTP/2 no ``@cmmv/server``:

**Exemplo com HTTP/2**

```typescript
import { readFileSync } from 'node:fs';
import cmmv from '@cmmv/server';

const app = cmmv({
    http2: true, // Habilita o HTTP/2
    https: {
        key: readFileSync('./cert/private-key.pem'),
        cert: readFileSync('./cert/certificate.pem'),
        passphrase: '1234'
    }
});

const host = '0.0.0.0';
const port = 3000;

app.get('/', (req, res) => {
    res.send('Olá, Mundo HTTP/2!');
});

app.listen({ host, port })
.then(server => {
    console.log(
        `Servidor HTTP/2 rodando em https://${server.address().address}:${server.address().port}`
    );
})
.catch(err => {
    throw new Error(err.message);
});
```

A [negociação ALPN](https://datatracker.ietf.org/doc/html/rfc7301) permite suporte tanto para HTTPS quanto para HTTP/2 no mesmo socket. Os objetos ``req`` e ``res`` do núcleo do Node podem ser tanto HTTP/1 quanto HTTP/2:

```typescript
const app = cmmv({
    http2: true, // Habilita o HTTP/2
    https: {
        allowHTTP1: true, // Suporte de fallback para HTTP/1
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

Para testar o HTTP/2 localmente com o ``@cmmv/server``, você precisa de um certificado SSL, pois os navegadores exigem HTTPS para suportar o HTTP/2. Veja como gerar um certificado autoassinado para seu ambiente localhost.

### Passos para Criar um Certificado SSL Autoassinado

Você pode usar o OpenSSL, uma ferramenta gratuita e de código aberto, para gerar os arquivos SSL necessários (chave privada e certificado) para desenvolvimento local. Siga os passos abaixo:

**1. Instalar o OpenSSL**

Se você ainda não tem o OpenSSL instalado, pode baixá-lo e instalá-lo a partir [daqui](https://www.openssl.org/) ou por meio de um gerenciador de pacotes como ``brew`` no macOS ou ``apt-get`` no Linux.

Para macOS:

```bash
brew install openssl
```

Para Linux (Ubuntu/Debian):

```bash
sudo apt-get install openssl
```

**2. Gerar uma Chave Privada**

Primeiro, você precisa criar uma chave privada que será usada para assinar o certificado SSL.

```bash
openssl genpkey -algorithm RSA -out private-key.pem -aes256
```

**3. Criar um Certificado Autoassinado**

Agora que você tem uma chave privada, pode criar o certificado SSL.

```bash
openssl req -new -x509 -key private-key.pem -out certificate.pem -days 365
```

Esse comando gerará um certificado autoassinado chamado ``certificate.pem`` válido por 365 dias. O parâmetro ``-subj`` fornece as informações necessárias do certificado, como o Nome Comum (``CN``), que deve ser ``localhost``.

**4. Ignorar Alertas do Navegador (Opcional)**

Como o certificado é autoassinado, a maioria dos navegadores exibirá um aviso de segurança. Para contornar isso:

* **Chrome:** Navegue até ``chrome://flags/#allow-insecure-localhost`` e habilite a opção que permite localhost inseguro.
* **Firefox:** Clique em "Avançado" e depois em "Adicionar Exceção" quando o aviso aparecer.

**5. Usar o Certificado no ``@cmmv/server``**

Agora você pode usar os arquivos gerados ``private-key.pem`` e ``certificate.pem`` na configuração do seu ``@cmmv/server`` como mostrado abaixo:

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
    res.send('Olá, HTTP/2 com HTTPS!');
});

app.listen({ host: '0.0.0.0', port: 3000 })
.then(server => {
    console.log(`Servidor HTTP/2 rodando em https://localhost:3000`);
})
.catch(err => {
    console.error('Erro ao iniciar o servidor:', err);
});
```

Agora, seu servidor local rodará com HTTP/2 e criptografia SSL!

## Simples ou Inseguro

Se você está construindo microsserviços, pode conectar-se ao HTTP/2 em texto simples; no entanto, isso não é suportado por navegadores.

```typescript
import { readFileSync } from 'node:fs';
import cmmv from '@cmmv/server';

const app = cmmv({ http2: true });

app.get('/', (req, res) => {
    res.send('Olá, HTTP/2 sem HTTPS!');
});

app.listen({ host: '0.0.0.0', port: 3000 })
.then(server => {
    console.log(`Servidor HTTP/2 rodando em http://localhost:3000`);
})
.catch(err => {
    console.error('Erro ao iniciar o servidor:', err);
});
```

Você pode testar seu novo servidor com:

```bash
$ npx h2url http://localhost:3000
```
