# Motor de Visão

Em aplicações web, o motor de visão é responsável por renderizar templates e gerar páginas HTML. Em uma configuração típica, o motor de visão pega dados dinâmicos e os renderiza em uma página HTML estática que é enviada ao cliente. O método ``res.render`` no servidor permite que você renderize um template usando o motor de visão que você configurar.

Nesta documentação, demonstraremos como configurar e usar um motor de visão, especificamente com o popular motor de templates Pug. No entanto, é possível configurar diferentes motores de visão dependendo das suas necessidades.

No exemplo a seguir, configuramos o motor de templates Pug para renderizar arquivos ``.pug`` a partir do diretório ``/views``. Veja como você pode configurá-lo:

```typescript
// Importa os módulos necessários
import cmmv from '@cmmv/server';
import { json, urlencoded } from '@cmmv/server';
import { readFileSync } from 'fs';
import path from 'path';

// Inicializa o aplicativo
const app = cmmv();

// Define o motor de visão como Pug
app.set('view engine', 'pug');

// Especifica o diretório onde os templates estão localizados
app.set('views', path.join(__dirname, 'views'));

// Middleware para analisar dados JSON e URL-encoded
app.use(json({ limit: '50mb' }));
app.use(urlencoded({ extended: true }));

// Rota de exemplo para renderizar um template Pug
app.get('/view', function (req, res) {
  res.render('index', { title: 'Olá', message: 'Bem-vindo ao aplicativo!' });
});

// Inicia o servidor
const host = '0.0.0.0';
const port = 3000;

app.listen({ host, port }).then((server) => {
  console.log(`Servidor rodando em http://${host}:${port}`);
}).catch((err) => {
  console.error(err);
});
```

O diretório padrão para visões é ``/views``, mas isso pode ser personalizado como mostrado no exemplo. O motor de templates Pug espera arquivos com a extensão ``.pug`` dentro desse diretório.

Por exemplo, o arquivo ``index.pug`` no diretório ``/views`` poderia ser assim:

```pug
doctype html
html
  head
    title= title
  body
    h1= message
```

Ao configurar um motor de visão, você pode gerar dinamicamente páginas HTML com base em templates e dados. O exemplo fornecido mostra como configurar o motor de templates Pug e usá-lo para renderizar HTML a partir de templates Pug armazenados em um diretório específico. Isso permite gerenciar facilmente conteúdo dinâmico e exibi-lo ao cliente em sua aplicação web.

No CMMV, você pode configurar vários motores de visão suportados pelo Express, como [EJS](https://ejs.co/), [Mustache](https://mustache.github.io/), entre outros. Além disso, também é possível criar implementações personalizadas para atender às suas necessidades. Por exemplo, no CMMV, há suporte nativo para SSR (Renderização no Lado do Servidor) usando o motor de visão personalizado ``@cmmv/view``. Abaixo está um exemplo de implementação que demonstra como configurar e usar o motor de visão personalizado do CMMV.

## Motor de Visão Personalizado

Esta implementação configura um motor de visão personalizado que processa arquivos ``.html`` para SSR (Renderização no Lado do Servidor). Ele utiliza o próprio motor de visão do CMMV, que oferece flexibilidade no manuseio de conteúdo dinâmico com cabeçalhos personalizados e geração de *nonce* para políticas de segurança.

```typescript
import cmmv from '@cmmv/server';
import { v4 as uuidv4 } from 'uuid';
import path from 'path';
import fs from 'fs';
import Config from './config'; // Assume que você tem um serviço Config para configuração do aplicativo

const publicDir = path.join(process.cwd(), 'public');
const app = cmmv();

// Configurando o motor de visão para processar arquivos .html usando o motor personalizado
app.set('views', publicDir);
app.set('view engine', 'html');
app.engine('html', (filePath, options, callback) => {
    app.render.renderFile(
        filePath,
        options,
        { nonce: options.nonce || '' },
        callback,
    );
});

// Adicionando um hook para gerenciar requisições e lidar com cabeçalhos com políticas de segurança
app.addHook('onRequest', (req, res, next) => {
    req.requestId = uuidv4();
    res.locals = {};
    res.locals.nonce = uuidv4().substring(0, 8);

    // Definir cabeçalhos de segurança personalizados
    const customHeaders = Config.get('headers') || {};
    for (const headerName in customHeaders) {
        let headerValue = customHeaders[headerName];

        if (Array.isArray(headerValue)) {
            headerValue = headerValue
                .map(value => {
                    if (headerName === 'Content-Security-Policy')
                        return value.indexOf('style-src') == -1
                            ? `${value} 'nonce-${res.locals.nonce}'`
                            : value;
                    return value;
                })
                .join('; ');
        } else if (typeof headerValue === 'string') {
            if (headerName === 'Content-Security-Policy')
                headerValue =
                    headerValue.indexOf('style-src') == -1
                        ? `${headerValue} 'nonce-${res.locals.nonce}'`
                        : headerValue;
        }

        res.setHeader(headerName, headerValue);
    }

    // Servir arquivos HTML do diretório /public/views
    const publicDir = path.join(process.cwd(), 'public/views');
    const requestPath = req.path === '/' ? 'index' : req.path.substring(1);
    const possiblePaths = [
        path.join(publicDir, `${requestPath}.html`),
        path.join(publicDir, requestPath, 'index.html'),
        path.join(publicDir, `${requestPath}`),
        path.join(publicDir, requestPath, 'index.html'),
    ];

    // Verificar se algum dos caminhos possíveis existe
    let fileFound = false;
    for (const filePath of possiblePaths) {
        if (fs.existsSync(filePath)) {
            fileFound = true;
            const config = Config.getAll();

            return res.render(filePath, {
                nonce: res.locals.nonce,
                requestId: req.requestId,
                config,
            });
        }
    }

    if (!fileFound) res.code(404).send('Página não encontrada');

    next();
});

// Iniciar o servidor
const host = '0.0.0.0';
const port = 3000;

app.listen({ host, port }).then(server => {
    console.log(`Servidor rodando em http://${host}:${port}`);
});
```

O framework CMMV oferece flexibilidade ao suportar qualquer motor de visão compatível com o Express, como EJS, Mustache e outros. Além disso, você pode implementar motores de visão personalizados, como o ``@cmmv/view`` usado no exemplo acima, para aprimorar as capacidades de renderização no lado do servidor e otimizar o desempenho com comportamentos personalizados.
