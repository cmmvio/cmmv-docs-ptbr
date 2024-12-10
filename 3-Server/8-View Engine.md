## View Engine

No desenvolvimento de aplicações web, o motor de visualização (view engine) é responsável por renderizar templates e gerar páginas HTML. O método `res.render` permite renderizar um template usando o motor de visualização configurado.

### Configurando o Motor de Visualização com Pug

O exemplo a seguir demonstra como configurar o Pug como motor de visualização para renderizar arquivos `.pug` localizados no diretório `/views`:

```typescript
// Importando os módulos necessários
import cmmv from '@cmmv/server';
import { json, urlencoded } from '@cmmv/server';
import { readFileSync } from 'fs';
import path from 'path';

// Inicializando o app
const app = cmmv();

// Configurando o motor de visualização como Pug
app.set('view engine', 'pug');

// Especificando o diretório onde os templates estão localizados
app.set('views', path.join(__dirname, 'views'));

// Middleware para parsear dados JSON e URL-encoded
app.use(json({ limit: '50mb' }));
app.use(urlencoded({ extended: true }));

// Rota de exemplo para renderizar um template Pug
app.get('/view', function (req, res) {
  res.render('index', { title: 'Hello', message: 'Welcome to the app!' });
});

// Iniciando o servidor
const host = '0.0.0.0';
const port = 3000;

app.listen({ host, port }).then((server) => {
  console.log(`Server running at http://\${host}:\${port}`);
}).catch((err) => {
  console.error(err);
});
```

O diretório padrão de views é `/views`, mas isso pode ser personalizado. O arquivo `index.pug` no diretório `/views` poderia ser assim:

```pug
doctype html
html
  head
    title= title
  body
    h1= message
```

### Configuração de Motor de Visualização Personalizado

Esta implementação configura um motor de visualização customizado para processar arquivos `.html`, suportando renderização no lado do servidor (SSR) com políticas de segurança customizadas.

```typescript
import cmmv from '@cmmv/server';
import { v4 as uuidv4 } from 'uuid';
import path from 'path';
import fs from 'fs';
import Config from './config'; // Assume que existe um serviço Config para configurações do app

const publicDir = path.join(process.cwd(), 'public');
const app = cmmv();

// Configurando o motor de visualização para processar arquivos .html
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

// Adicionando um hook para gerenciar requisições e cabeçalhos de segurança
app.addHook('onRequest', (req, res, next) => {
    req.requestId = uuidv4();
    res.locals = {};
    res.locals.nonce = uuidv4().substring(0, 8);

    // Configurando cabeçalhos de segurança personalizados
    const customHeaders = Config.get('headers') || {};
    for (const headerName in customHeaders) {
        let headerValue = customHeaders[headerName];

        if (Array.isArray(headerValue)) {
            headerValue = headerValue
                .map(value => {
                    if (headerName === 'Content-Security-Policy')
                        return value.indexOf('style-src') == -1
                            ? `\${value} 'nonce-\${res.locals.nonce}'`
                            : value;
                    return value;
                })
                .join('; ');
        } else if (typeof headerValue === 'string') {
            if (headerName === 'Content-Security-Policy')
                headerValue =
                    headerValue.indexOf('style-src') == -1
                        ? `\${headerValue} 'nonce-\${res.locals.nonce}'`
                        : headerValue;
        }

        res.setHeader(headerName, headerValue);
    }

    // Servindo arquivos HTML do diretório /public/views
    const publicDir = path.join(process.cwd(), 'public/views');
    const requestPath = req.path === '/' ? 'index' : req.path.substring(1);
    const possiblePaths = [
        path.join(publicDir, `\${requestPath}.html`),
        path.join(publicDir, requestPath, 'index.html'),
        path.join(publicDir, `\${requestPath}`),
        path.join(publicDir, requestPath, 'index.html'),
    ];

    // Verificando se algum dos caminhos existe
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

    if (!fileFound) res.code(404).send('Page not found');

    next();
});

// Iniciando o servidor
const host = '0.0.0.0';
const port = 3000;

app.listen({ host, port }).then(server => {
    console.log(`Server running at http://\${host}:\${port}`);
});
```

O framework CMMV oferece flexibilidade para suportar qualquer motor de visualização compatível com Express, como EJS, Mustache, entre outros. Também permite a criação de motores de visualização personalizados, como o motor `@cmmv/view`, utilizado no exemplo acima.
