# Assets

No CMMV, é recomendado que todos os arquivos estáticos, como bibliotecas JavaScript, CSS, fontes e imagens, sejam servidos por meio de uma Content Delivery Network (CDN). As CDNs são otimizadas para uma entrega rápida e eficiente de assets, reduzindo a latência e melhorando a experiência do usuário. No entanto, se você optar por servir esses arquivos diretamente de sua aplicação, eles devem ser colocados no diretório `/public`. O módulo `@cmmv/http` procura automaticamente e serve arquivos estáticos desse diretório.

Se você optar por não usar uma CDN, coloque todos os seus arquivos estáticos no diretório `/public`. Este é o diretório onde o `@cmmv/http` servirá automaticamente arquivos estáticos, como:

## Bundle 

Por padrão, a aplicação gerará arquivos complementares, resultando em um bundle final que é necessário se você estiver utilizando RPC e reatividade no frontend. Este bundle será criado como `/assets/bundle.min.js` e deve ser incluído em seus arquivos HTML ou templates para garantir que o frontend funcione corretamente.

**Exemplo de Configuração do `.cmmv.config.js` para Assets:**

```javascript
module.exports = {
    ...

    scripts: [
        { type: "text/javascript", src: "/assets/bundle.min.js", defer: "defer" },
        ...
    ]
};
```

Se você estiver servindo assets localmente, certifique-se de que o seu diretório `/public` esteja estruturado da seguinte forma:

```bash
/public
    /assets
        /styles.css
        /app.min.js
        /bundle.min.js
    /images
        /logo.png
    /fonts
        /custom-font.woff2
    /favicon.ico
    /robots.txt
    /sitemap.xml
    /service-worker.js
```

Ao utilizar esses assets em seus arquivos HTML ou templates, você pode incluir o bundle JavaScript e quaisquer outros arquivos estáticos assim:

```html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Aplicação CMMV</title>

    <!-- Incluir Estilos -->
    <link rel="stylesheet" href="/assets/styles.css">
    
    <!-- Favicon -->
    <link rel="icon" href="/favicon.ico" type="image/x-icon">
</head>
<body>
    <!-- Conteúdo da Aplicação -->

    <!-- Incluir o Bundle -->
    <script src="/assets/bundle.min.js"></script>
</body>
</html>
```

No CMMV, o gerenciamento de assets é simplificado ao recomendar o uso de CDNs para arquivos estáticos. No entanto, se você estiver servindo esses assets diretamente da aplicação, colocá-los no diretório `/public` permitirá que o `@cmmv/http` os manipule automaticamente. Além disso, a aplicação gerará os bundles necessários para lidar com as funcionalidades de reatividade e RPC no frontend, disponíveis como `/assets/bundle.min.js`. Todos os arquivos estáticos ficam facilmente acessíveis a partir desse diretório, garantindo uma integração perfeita com sua aplicação.
