<p align="center">
  <img src="assets/logo_CMMV2_icon.png" width="300" alt="CMMV Logo" />
</p>
<p align="center">Contract-Model-Model-View (CMMV) <br/> Building scalable and modular applications using contracts.</p>
<div class="flex flex-container">
    <a href="https://www.npmjs.com/package/@cmmv/core">
        <img src="https://img.shields.io/npm/v/@cmmv/core.svg" alt="NPM Version" />
    </a>
    <a href="https://github.com/cmmvio/cmmv/blob/main/LICENSE">
        <img src="https://img.shields.io/npm/l/@cmmv/core.svg" alt="Package License" />
    </a>
    <a href="https://dl.circleci.com/status-badge/redirect/circleci/QyJWAYrZ9JTfN1eubSDo5u/7gdwcdqbMYfbYYX4hhoNhc/tree/main" target="_blank">
        <img src="https://dl.circleci.com/status-badge/img/circleci/QyJWAYrZ9JTfN1eubSDo5u/7gdwcdqbMYfbYYX4hhoNhc/tree/main.svg" alt="CircleCI" />
    </a>
</div>

<br/>

# Introdução

CMMV (Contract-Model-Model-View) é um framework minimalista projetado para simplificar o desenvolvimento de aplicações escaláveis e modulares utilizando TypeScript. Combinando o poder dos contratos com uma arquitetura modular, o CMMV permite que os desenvolvedores definam toda a estrutura de suas aplicações, desde entidades ORM até controladores REST e endpoints WebSocket, de maneira clara e fácil de manter.

O projeto é dividido em três componentes principais:

1. **Sistema Core**: O núcleo do CMMV, escrito em TypeScript e executado no Node.js, é responsável por gerenciar contratos, gerar código, integrar com bancos de dados e lidar com operações do lado do servidor. Este componente realiza desde o parsing e geração de código até a integração com serviços externos, como servidores em nuvem e bancos de dados.

2. **Backend**: O backend do CMMV é inspirado no [NestJS](https://nestjs.com/) em termos de estrutura e organização, utilizando um formato semelhante de decoradores, serviços, controladores e outros padrões arquiteturais. Isso proporciona uma experiência de desenvolvimento familiar para aqueles que já utilizam o ``NestJS``, facilitando a criação de aplicações escaláveis e organizadas com uma abordagem modular e orientada a objetos. Algumas implementações são bastante diferentes, especialmente no que diz respeito ao contexto e à necessidade de injeção de dependência.

3. **Frontend**: O CMMV utiliza seu próprio sistema de reatividade, inspirado pelo [Vue 3](https://vuejs.org/) como framework base, com suporte já disponível para Vue 3. Em breve, será disponibilizado suporte para outros frameworks, como React e Angular. Apesar dessa expansão, o uso desses frameworks não é recomendado para casos em que desempenho máximo e otimização de SEO sejam prioritários. Essa abordagem mantém o processo de criação, gerenciamento e implantação de aplicações simples e oferece um desempenho otimizado.

O CMMV é inspirado por uma ampla gama de tecnologias e conceitos, mesclando ideias do desenvolvimento de jogos (por exemplo, Blueprints do Unreal Engine), arquiteturas baseadas em componentes (por exemplo, Delphi) e práticas modernas de desenvolvimento web. Ele desafia os paradigmas tradicionais de desenvolvimento web e de aplicações, buscando tornar a criação de sistemas complexos o mais intuitiva possível.

Este projeto reflete mais de 20 anos de experiência em diferentes linguagens e frameworks, com influências de Delphi, Unity, Unreal, C#, C++, JavaScript, Node.js, TypeScript e VSCode.

Esperamos que você ache o CMMV tão empolgante e poderoso quanto nós. Ele é o culminar de quase uma década de trabalho e paixão por simplificar e aprimorar o processo de desenvolvimento.

## Por que o CMMV?

Com mais de 20 anos de experiência na indústria de tecnologia como programador, desenvolvi diversos sistemas e projetos utilizados por milhões de usuários. Em 2020, eu estava trabalhando no maior projeto da minha carreira, que foi construído com o seguinte stack: backend utilizando Node.js, TypeScript, NestJS, Nuxt.js, Redis, MongoDB, Elasticsearch e RabbitMQ; frontend com Vue.js e Tailwind CSS; testes com Mocha, ESLint e GitLab CI; infraestrutura gerenciada com Kubernetes e Nginx; e inteligência de negócios impulsionada por Grafana, Kafka e IndexDB, juntamente com um aplicativo móvel baseado em Flutter. Vamos explorar os problemas que enfrentamos nesse ambiente.

Primeiro, a API do NestJS, à medida que mais controladores e gateways foram adicionados, tornou-se cada vez mais difícil de gerenciar e desacelerou significativamente, principalmente devido à injeção de dependências. O gerenciamento de módulos se transformou em um pesadelo burocrático. Embora a aplicação final tivesse um bom desempenho, o desenvolvimento tornou-se trabalhoso. Além disso, a renderização SSR (server-side rendering) do Nuxt exigia integração, o que frequentemente causava problemas devido a políticas de CORS. A comunicação entre aplicações ocorria via HTTP, e, embora o NestJS suportasse RPC, o Nuxt.js exigia proxies personalizados para implementar WebSockets. O uso de Protobuf no frontend apresentava desafios adicionais, e a geração de controladores RPC com Protoc resultava em uma base de código inchada, tornando a aplicação ainda mais pesada.

Segundo, o Nuxt.js, embora capaz de SSR, ainda dependia de proxies com APIs para carregar dados, pois rodava em uma implementação baseada em Vite. A geração de páginas estáticas em escala, com milhares de páginas, tornou-se uma tarefa inatingível. Usando SSR padrão, com proxy API e comunicação HTTP+JSON, o TTFB (Tempo para o Primeiro Byte) aumentava significativamente, dificultando a otimização. Apenas entregando conteúdo diretamente por meio de uma CDN conseguimos mitigar esses atrasos, mas mesmo assim, o carregamento da página era lento demais para um SEO ideal. Além disso, o Nuxt.js gerava inúmeros pacotes JavaScript e arquivos de dados para complementar o binding de dados do frontend, aumentando o tempo de carregamento da página e tornando a otimização de SEO uma luta constante.

Embora pareça contraintuitivo combinar API e SSR na mesma aplicação devido a processos concorrentes, considere que é muito mais simples criar um único balanceador de carga que sirva tanto o frontend quanto o backend. Com integração direta, o SSR elimina a latência na obtenção de dados, o que é excelente para reduzir o tempo de carregamento da página e gerar páginas estáticas. Quando bem integrado ao cache Redis e consultas eficientes ao banco de dados, o SSR pode ser quase tão rápido quanto páginas estáticas pré-renderizadas. Além disso, essa arquitetura simplifica integrações como internacionalização, dados estruturados e sitemaps, tornando-os muito mais fáceis de gerenciar e servir.

Finalmente, a redução no tempo de carregamento da página, que também é considerada pelos motores de busca, é crucial. Embora o Vue seja uma ferramenta excelente, a componentização no frontend apresenta desafios de desempenho devido ao possível acoplamento profundo entre os componentes. Mesmo com o gerenciamento de estado, problemas críticos relacionados à reatividade dos elementos podem levar a fluxos de atualização em cascata, que podem congelar o aplicativo ou causar loops infinitos. Evitar esses problemas requer um sólido entendimento do framework subjacente.

Considerando todos esses fatores, criei o CMMV (Contract-Model-Model-View). Inicialmente, o objetivo era desenvolver uma solução completa que atendesse às minhas necessidades de desenvolvimento, mantendo a sintaxe familiar do NestJS, Vue, etc., mas resolvendo os problemas que me atormentaram ao longo dos anos e dificultaram o desenvolvimento.

O CMMV não foi criado para competir ou substituir nenhuma das ferramentas mencionadas. Todas as tecnologias mencionadas são de excelente qualidade e altamente recomendadas. Elas possuem grandes comunidades e são adequadas para a maioria dos projetos. No entanto, eu enfrentei desafios únicos nos meus projetos, particularmente ao lidar com altos volumes de tráfego e a necessidade do melhor desempenho possível em SEO. Apesar de todos os esforços com meu stack anterior, os resultados ainda não eram satisfatórios para minhas necessidades específicas. O CMMV nasceu dessa necessidade de atender a esses requisitos excepcionais.

André Ferreira (CEO)
