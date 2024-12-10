# HTTP2

The ``@cmmv/server`` framework natively supports HTTP/2, which offers significant performance improvements over HTTP/1.1 by allowing multiplexed streams, header compression, and server push. These features are ideal for applications that need faster page load times and better resource utilization, especially in modern web applications.

HTTP/2 is particularly useful for reducing latency, improving page load performance, and enhancing the overall efficiency of the client-server communication.

Enabling HTTP/2 in @cmmv/server
To enable HTTP/2, you simply need to configure the server with the http2 option set to true. Additionally, you will need to provide SSL certificates since HTTP/2 requires HTTPS for browser support.

Here's an example of how to implement HTTP/2 support in @cmmv/server:

**Example with HTTP/2**

```typescript
import { readFileSync } from 'node:fs';
import cmmv from '@cmmv/server';

const app = cmmv({
    http2: true, // Enable HTTP/2
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
        `HTTP/2 server running on https://${server.address().address}:${server.address().port}`
    );
})
.catch(err => {
    throw new Error(err.message);
});
```

[ALPN negotiation](https://datatracker.ietf.org/doc/html/rfc7301) allows support for both HTTPS and HTTP/2 over the same socket. Node core req and res objects can be either HTTP/1 or HTTP/2:

```typescript
const app = cmmv({
    http2: true, // Enable HTTP/2
    https: {
        allowHTTP1: true, // fallback support for HTTP1
        key: readFileSync('./cert/private-key.pem'),  
        cert: readFileSync('./cert/certificate.pem'), 
        passphrase: '1234'                           
    }
});
```

You can test your new server with:

```bash
$ npx h2url https://localhost:3000
```

## Self-Signed SSL Certificate

To test HTTP/2 locally with ``@cmmv/server``, you need an SSL certificate because browsers require HTTPS to support HTTP/2. Here's how to generate a self-signed certificate for your localhost environment.

Steps to Create a Self-Signed SSL Certificate
You can use OpenSSL, which is a free and open-source tool, to generate the necessary SSL files (private key and certificate) for local development. Follow the steps below:

**1. Install OpenSSL**
If you don't already have OpenSSL installed, you can download and install it from here or through a package manager like ``brew`` on macOS or ``apt-get`` on Linux.

For macOS:

```bash
brew install openssl
```

For Linux (Ubuntu/Debian):

```bash
sudo apt-get install openssl
```

**2. Generate a Private Key**

First, you need to create a private key that will be used to sign the SSL certificate.

```bash
openssl genpkey -algorithm RSA -out private-key.pem -aes256
```

**3. Create a Self-Signed Certificate**

Now that you have a private key, you can create the SSL certificate.

```bash
openssl req -new -x509 -key private-key.pem -out certificate.pem -days 365
```

This command will generate a self-signed certificate named ``certificate.pem`` that's valid for 365 days. The ``-subj`` parameter provides the necessary certificate information, such as the Common Name (``CN``), which should be ``localhost``.

**4. Bypass Browser Warnings (Optional)**

Since the certificate is self-signed, most browsers will display a security warning. To bypass this:

* **Chrome:** Navigate to ``chrome://flags/#allow-insecure-localhost`` and enable the flag that allows insecure localhost.
* **Firefox:** Click "Advanced" and then "Add Exception" when the warning appears.

**5. Use the Certificate in ``@cmmv/server``**

Now, you can use the generated ``private-key.pem`` and certificate.pem files in your ``@cmmv/server`` configuration as shown below:

```typescript
import { readFileSync } from 'node:fs';
import cmmv from '@cmmv/server';

const app = cmmv({
    http2: true,
    https: {
        key: readFileSync('./cert/private-key.pem'),
        cert: readFileSync('./cert/certificate.pem'),
        passphrase: 'your-passphrase'
    }
});

app.get('/', (req, res) => {
    res.send('Hello, HTTP/2 with HTTPS!');
});

app.listen({ host: '0.0.0.0', port: 3000 })
.then(server => {
    console.log(`HTTP/2 server running on https://localhost:3000`);
})
.catch(err => {
    console.error('Error starting the server:', err);
});
```

Now your local server will run with HTTP/2 and SSL encryption!

## Plain or insecure

If you are building microservices, you can connect to HTTP2 in plain text, however, this is not supported by browsers.

```typescript
import { readFileSync } from 'node:fs';
import cmmv from '@cmmv/server';

const app = cmmv({ http2: true });

app.get('/', (req, res) => {
    res.send('Hello, HTTP/2 without HTTPS!');
});

app.listen({ host: '0.0.0.0', port: 3000 })
.then(server => {
    console.log(`HTTP/2 server running on https://localhost:3000`);
})
.catch(err => {
    console.error('Error starting the server:', err);
});
```

You can test your new server with:

```bash
$ npx h2url http://localhost:3000
```