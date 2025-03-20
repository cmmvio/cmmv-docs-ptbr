# IA

Repositório: [https://github.com/cmmvio/cmmv-ai](https://github.com/cmmvio/cmmv-ai)

O módulo **@cmmv/ai** oferece suporte completo para **Geração Aumentada por Recuperação (RAG)** e, no futuro, **Síntese Aumentada por Recuperação (RAS)** para **LLMs**. Ele permite a compreensão e geração de código, suportando:

* **Tokenização e Mapeamento de Código** – Extrai tokens estruturados de arquivos TypeScript/JavaScript.
* **Criação de Conjunto de Dados RAG** – Gera conjuntos de dados binários para busca vetorial.
* **Busca Vetorial com FAISS e Bancos de Dados Vetoriais** – Suporta **Qdrant, Milvus, Neo4j**.
* **Integração com Hugging Face** – Usa `transformers` para embeddings.
* **Modelos de Embedding Personalizados** – Suporta `WhereIsAI/UAE-Large-V1`, `MiniLM`, `CodeLlama`, `DeepSeek`, entre outros.
* **Integração com Bancos de Dados** – Suporta `Elasticsearch`, `Pinecone`, `Qdrant`, `PGVector`, entre outros.
* **Integração com LLM** – Suporta `OpenAI`, `Hugging Face`, `Ollama`, `DeepSeek`, `Groq`, `Gemini`, entre outros.

## Instalar Python

Antes de instalar a CLI do Hugging Face, certifique-se de que o **Python** está instalado no seu sistema.

Execute o seguinte comando para instalar o Python no Ubuntu:

```sh
$ sudo apt update && sudo apt install python3 python3-pip -y
```

Para outros sistemas operacionais, consulte a página oficial de [download do Python](https://www.python.org/downloads/).

## Instalar a CLI do Hugging Face

Com o Python instalado, instale a CLI do Hugging Face usando **pip**:

```sh
$ pip3 install -U "huggingface_hub[cli]"
```

## Garantir que a CLI seja Reconhecida

Se o terminal não reconhecer `huggingface-cli`, adicione `~/.local/bin` ao seu **PATH** do sistema:

```sh
$ echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
$ source ~/.bashrc
```

Execute o seguinte comando para verificar a instalação:

```sh
$ huggingface-cli --help
```

Se o comando funcionar, a instalação foi bem-sucedida! 🎉

## Autenticar com Hugging Face

Para acessar e baixar modelos, você precisa se autenticar.

Execute:

```sh
$ huggingface-cli login
```

Você será solicitado a inserir seu **token de acesso do Hugging Face**.
Gere um em: [Tokens do Hugging Face](https://huggingface.co/settings/tokens)
Certifique-se de que o token tenha permissões de **LEITURA**.

## Baixar Modelos

Para baixar um modelo, use o seguinte comando:

```sh
$ huggingface-cli download meta-llama/CodeLlama-7B-Python-hf --local-dir ./models/CodeLlama-7B
```

Isso baixará o modelo **CodeLlama 7B Python** para o diretório `./models/CodeLlama-7B`.

Para o **CMMV**, defina o caminho do modelo no arquivo `.cmmv.config.cjs`:

```js
huggingface: {
    token: process.env.HUGGINGFACE_HUB_TOKEN,
    localModelPath: './models',
    allowRemoteModels: true
},
llm: {
    provider: "google",
    embeddingTopk: 10,
    modelName: "gemini-1.5-pro",
    textMaxTokens: 2048,
    apiKey: process.env.GOOGLE_API_KEY,
    language: 'pt-br'
}
```

Agora seu ambiente está configurado para usar modelos do Hugging Face com o **CMMV**! 🚀

## Configuração

Dada a complexidade deste módulo, várias configurações são necessárias, especialmente para definir quais modelos usar em cada etapa. Abaixo está a **configuração base** com todas as propriedades disponíveis atualmente.

### **Exemplo de Configuração (`.cmmv.config.cjs`)**
<br/>

```javascript
require('dotenv').config();

module.exports = {
    env: process.env.NODE_ENV,

    ai: {
        huggingface: {
            token: process.env.HUGGINGFACE_HUB_TOKEN,
            localModelPath: './models',
            allowRemoteModels: true
        },
        tokenizer: {
            provider: "huggingface",
            model: "sentence-transformers/distilbert-base-nli-mean-tokens",
            indexSize: 768,
            useKeyBERT: false,
            chunkSize: 1000,
            chunkOverlap: 0,
            patterns: [
                //'../cmmv/**/*.ts',
                //'../cmmv/src/**/*.ts',
                //'../cmmv/packages/**/*.ts',
                //'../cmmv-*/**/*.ts',
                //'../cmmv-*/src/*.ts',
                //'../cmmv-*/src/**/*.ts',
                //'../cmmv-*/packages/**/*.ts',
                '../cmmv-*/**/*.md',
                '../cmmv-docs/docs/en/**/*.md'
            ],
            output: "./samples/data.bin",
            ignore: [
                "node_modules", "*.d.ts", "*.cjs",
                "*.spec.ts", "*.test.ts", "/tools/gulp/"
            ],
            exclude: [
                "cmmv-formbuilder", "cmmv-ui",
                "cmmv-language-tools", "cmmv-vue",
                "cmmv-reactivity", "cmmv-vite-plugin",
                "eslint.config.ts", "vitest.config.ts",
                "auto-imports.d.ts", ".d.ts", ".cjs",
                ".spec.ts", ".test.ts", "/tools/gulp/",
                "node_modules"
            ]
        },
        vector: {
            provider: "neo4j",
            qdrant: {
                url: 'http://localhost:6333',
                collection: 'embeddings'
            },
            neo4j: {
                url: "bolt://localhost:7687",
                username: process.env.NEO4J_USERNAME,
                password: process.env.NEO4J_PASSWORD,
                indexName: "vector",
                keywordIndexName: "keyword",
                nodeLabel: "Chunk",
                embeddingNodeProperty: "embedding"
            }
        },
        llm: {
            provider: "google",
            embeddingTopk: 10,
            model: "gemini-1.5-pro",
            textMaxTokens: 2048,
            apiKey: process.env.GOOGLE_API_KEY,
            language: 'pt-br'
        }
    }
};
```

| **Caminho**                                   | **Descrição**                                      | **Valor Padrão / Exemplo**                         |
|-----------------------------------------------|--------------------------------------------------|------------------------------------------------|
| `ai.huggingface.token`                        | Token da API para o Hugging Face Hub             | `process.env.HUGGINGFACE_HUB_TOKEN`            |
| `ai.huggingface.localModelPath`               | Caminho para modelos locais                      | `./models`                                     |
| `ai.huggingface.allowRemoteModels`            | Permite baixar modelos do Hugging Face Hub       | `true`                                         |
| `ai.tokenizer.provider`                       | Provedor de tokenização                          | `"huggingface"`                                |
| `ai.tokenizer.model`                          | Modelo de tokenização                            | `"sentence-transformers/distilbert-base-nli-mean-tokens"` |
| `ai.tokenizer.indexSize`                      | Tamanho do índice de embeddings                  | `768`                                          |
| `ai.tokenizer.useKeyBERT`                     | Habilita o KeyBERT para extração de palavras-chave | `false`                                        |
| `ai.tokenizer.chunkSize`                      | Tamanho dos pedaços de texto para processamento  | `1000`                                         |
| `ai.tokenizer.chunkOverlap`                   | Sobreposição entre pedaços de texto              | `0`                                            |
| `ai.tokenizer.patterns`                       | Padrões de busca de arquivos para tokenização    | `['../cmmv-*/**/*.md', '../cmmv-docs/docs/en/**/*.md']` |
| `ai.tokenizer.output`                         | Arquivo de saída para dados tokenizados          | `"./samples/data.bin"`                         |
| `ai.tokenizer.ignore`                         | Padrões de arquivos a ignorar                    | `["node_modules", "*.d.ts", "*.cjs", "*.spec.ts", "*.test.ts", "/tools/gulp/"]` |
| `ai.tokenizer.exclude`                        | Arquivos e diretórios a excluir                  | `["cmmv-formbuilder", "cmmv-ui", "cmmv-language-tools", "cmmv-vue", "cmmv-reactivity", "cmmv-vite-plugin", "eslint.config.ts", "vitest.config.ts", "auto-imports.d.ts", ".d.ts", ".cjs", ".spec.ts", ".test.ts", "/tools/gulp/", "node_modules"]` |
| `ai.vector.provider`                          | Provedor de armazenamento vetorial               | `"neo4j"`                                      |
| `ai.vector.qdrant.url`                        | URL do serviço Qdrant                            | `"http://localhost:6333"`                      |
| `ai.vector.qdrant.collection`                 | Nome da coleção para Qdrant                      | `"embeddings"`                                 |
| `ai.vector.neo4j.url`                         | URL do banco de dados Neo4j                      | `"bolt://localhost:7687"`                      |
| `ai.vector.neo4j.username`                    | Nome de usuário do Neo4j                         | `process.env.NEO4J_USERNAME`                   |
| `ai.vector.neo4j.password`                    | Senha do Neo4j                                   | `process.env.NEO4J_PASSWORD`                   |
| `ai.vector.neo4j.indexName`                   | Nome do índice para armazenamento vetorial       | `"vector"`                                     |
| `ai.vector.neo4j.keywordIndexName`            | Nome do índice para busca por palavras-chave     | `"keyword"`                                    |
| `ai.vector.neo4j.nodeLabel`                   | Rótulo para nós vetorizados                      | `"Chunk"`                                      |
| `ai.vector.neo4j.embeddingNodeProperty`       | Propriedade que armazena embeddings vetoriais    | `"embedding"`                                  |
| `ai.llm.provider`                             | Provedor de LLM                                  | `"google"`                                     |
| `ai.llm.embeddingTopk`                        | Número de resultados top-k para embeddings       | `10`                                           |
| `ai.llm.model`                                | Nome do modelo LLM                               | `"gemini-1.5-pro"`                             |
| `ai.llm.textMaxTokens`                        | Máximo de tokens por requisição                  | `2048`                                         |
| `ai.llm.apiKey`                               | Chave de API para o provedor de LLM              | `process.env.GOOGLE_API_KEY`                   |
| `ai.llm.language`                             | Idioma padrão                                    | `"pt-br"`                                      |

## Conversão de Modelos

Alguns **LLMs (Modelos de Linguagem de Grande Escala)** não são nativamente compatíveis com todos os frameworks de inferência. Um exemplo importante é o **Gemma da Google**, que não é diretamente suportado por muitas ferramentas. Para usar esses modelos de forma eficiente, você precisa **convertê-los para o formato ONNX**.

ONNX (**Open Neural Network Exchange**) é um formato aberto que otimiza modelos para inferência eficiente em múltiplas plataformas. Muitos frameworks de inferência, como **ONNX Runtime**, **TensorRT** e **OpenVINO**, suportam ONNX para uma implantação mais rápida e escalável.

Antes de converter, instale os pacotes necessários:

```sh
$ pip3 install -U "optimum[exporters]" onnx onnxruntime
```

Para converter o modelo **Gemma 2B da Google**, execute:

```sh
$ python3 -m optimum.exporters.onnx --model google/gemma-2b ./models/gemma-2b-onnx
```

## Tokenizador

A classe **Tokenizer** é responsável por processar e indexar arquivos no pipeline RAG. Ela oferece várias configurações, incluindo:

* **Padrões de Busca de Arquivos (`patterns`)**: Define padrões glob para buscar arquivos.
* **Exclusões e Arquivos Ignorados (`ignore`, `exclude`)**: Filtra arquivos desnecessários da indexação.
* **Modelo de Embeddings (`model`)**: Seleciona o modelo do Hugging Face para gerar embeddings.
* **Tamanho do Índice (`indexSize`)**: Ajusta as dimensões do embedding com base no modelo selecionado.
* **Fragmentação (`chunkSize`, `chunkOverlap`)**: Controla a segmentação de dados para melhor recuperação.
* **Geração de Metadados com KeyBERT (`useKeyBERT`)**: Extrai palavras-chave para aprimorar buscas vetoriais.

Para outros modelos que requerem autenticação (por exemplo, `LLaMA`), forneça a chave de API em `huggingface.token`.

### Exemplo de Uso:
<br/>

```typescript
import { Application, Hook, HooksType } from '@cmmv/core';

class TokenizerSample {
    @Hook(HooksType.onInitialize)
    async start() {
        const { Tokenizer } = await import('@cmmv/ai');
        const tokenizer = new Tokenizer();
        tokenizer.start();
    }
}

Application.exec({
    services: [TokenizerSample],
});
```
<br/>

1. **Escaneia os diretórios do projeto** com base na configuração `patterns`.
2. **Analisa arquivos TypeScript/JavaScript/Markdown**, extraindo **funções, classes, enums, interfaces, constantes e decoradores**.
3. **Gera embeddings** usando modelos do Hugging Face.
4. **Armazena o conjunto de dados** em um arquivo binário `.bin`.

## Modelos de Embedding Comuns

| **Embedding**   | **Modelo Padrão**                         | **Requer Chave de API** |
|----------------|-----------------------------------------|-----------------------|
| Bedrock       | amazon.titan-embed-text-v1             | Sim                   |
| Cohere        | embed-english-v3.0                     | Não                   |
| DeepInfra     | -                                       | Sim                   |
| Doubao        | -                                       | Sim                   |
| Fireworks     | nomic-ai/nomic-embed-text-v1.5         | Sim                   |
| HuggingFace   | Xenova/all-MiniLM-L6-v2                | Não                   |
| LlamaCpp      | - (requer arquivo de modelo local)     | Não                   |
| OpenAI        | text-embedding-3-large                 | Sim                   |
| Pinecone      | multilingual-e5-large                  | Não                   |
| Tongyi        | -                                       | Sim                   |
| Watsonx       | -                                       | Sim                   |
| Jina          | jina-clip-v2                            | Sim                   |
| MiniMax       | embo-01                                | Não                   |
| Premai        | -                                       | Não                   |
| Hunyuan       | -                                       | Sim                   |
| TensorFlow    | -                                       | Não                   |
| TogetherAI    | togethercomputer/m2-bert-80M-8k-retrieval | Sim                   |
| Voyage        | voyage-01                               | Sim                   |
| ZhipuAI       | embedding-2                            | Sim                   |

* *[https://huggingface.co/models?pipeline_tag=feature-extraction&library=transformers.js&sort=downloads](https://huggingface.co/models?pipeline_tag=feature-extraction&library=transformers.js&sort=downloads)*
* *[https://v03.api.js.langchain.com/index.html](https://v03.api.js.langchain.com/index.html)*

## Usando KeyBERT

O **KeyBERT** é um recurso opcional que aprimora a indexação extraindo palavras-chave relevantes. Ele ajuda a refinar os resultados de busca em **FAISS** ou bancos de dados vetoriais, melhorando a precisão das **consultas LLM**.

Diferente de **TF-IDF**, **YAKE!** ou **RAKE**, que dependem de métodos estatísticos, o **KeyBERT** utiliza **embeddings BERT** para gerar palavras-chave mais significativas. Isso resulta em uma filtragem de busca melhor, levando a **respostas baseadas em LLM mais precisas**.

Se o KeyBERT **não estiver habilitado**, o método padrão de extração de palavras-chave será **TF-IDF**, que pode não ser tão preciso, mas é significativamente mais rápido.

Antes de usar o KeyBERT, certifique-se de ter o **Python 3** instalado. Em seguida, instale o KeyBERT usando **pip**:

```bash
$ pip install keybert
```

Uma vez instalado, o KeyBERT será usado **durante a tokenização** para gerar **palavras-chave de filtragem**. Essas palavras-chave melhoram a classificação do conteúdo indexado, tornando os resultados da busca baseada em vetores mais relevantes.

Se você prefere um **processamento mais rápido**, pode desabilitar o KeyBERT, e o sistema voltará a usar **TF-IDF**.

Para habilitar o **KeyBERT**, atualize seu arquivo `.cmmv.config.cjs`:

```javascript
module.exports = {
    ai: {
        tokenizer: {
            useKeyBERT: true // Defina como false para usar TF-IDF
        }
    }
};
```

Com o **KeyBERT habilitado**, a filtragem de busca se torna mais **consciente do contexto**, levando a **respostas LLM mais precisas**.

Para mais detalhes sobre o KeyBERT, visite: [Documentação do KeyBERT](https://github.com/MaartenGr/KeyBERT).

## Conjunto de Dados

A classe **Dataset** gerencia o **armazenamento vetorizado** para recuperação rápida.

* Salva embeddings em **formato binário** (`.bin`).
* Busca em memória baseada em **FAISS**.
* Suporte para **Neo4j, Elasticsearch, PgVector, Qdrant**.

```typescript
const dataset = new Dataset();
dataset.save(); // Salva o conjunto de dados em formato binário
dataset.load(); // Carrega o conjunto de dados na memória
```

Para armazenar e buscar **embeddings** de forma eficiente com `@cmmv/ai`.

| Banco de Dados | Código Aberto | Suporte Node.js | Backend de Armazenamento | Busca por Similaridade               |
|---------------|--------------|----------------|-------------------------|------------------------------------|
| **Qdrant**    | ✅ Sim       | ✅ Sim (`@qdrant/js-client-rest`) | Disco/Memória | Cosseno, Euclidiana, Produto Escalar |
| **Milvus**    | ✅ Sim       | ✅ Sim (`@zilliz/milvus2-sdk-node`) | Disco/Memória | IVF_FLAT, HNSW, PQ                |
| **Neo4j**     | ✅ Sim (Comunidade) | ✅ Sim (`neo4j-driver`) | Banco de Dados de Grafos | Busca vetorial baseada em Cypher   |
| **Elasticsearch** | ✅ Sim     | ✅ Sim (`@elastic/elasticsearch`) | Disco | k-NN, Vizinhos Mais Próximos Aproximados (ANN) |
| **PGVector**  | ✅ Sim       | ✅ Sim (`pg`) | PostgreSQL | Cosseno, Euclidiana, Produto Interno |

Para executar esses bancos de dados localmente, use os seguintes
**comandos Docker**:

### **🔹 Qdrant**
```bash
$ docker run -p 6333:6333 --name qdrant-server qdrant/qdrant
```
<br/>

* Executa um servidor **Qdrant** na porta `6333`.
* API disponível em `http://localhost:6333`.

### **🔹 Milvus**
```bash
$ docker run -p 19530:19530 --name milvus-server milvusdb/milvus
```
<br/>

* Executa **Milvus** na porta `19530`.
* Requer **SDK Python/Node** para interação.

### **🔹 Neo4j**
```bash
$ docker run --publish=7474:7474 --publish=7687:7687 --volume=$HOME/neo4j/data:/data --name neo4j-server neo4j
```
<br/>

* Executa **Neo4j** nas portas `7474` (HTTP) e `7687` (Bolt).
* Os dados são armazenados persistentemente em `$HOME/neo4j/data`.

### **🔹 PGVector**
```bash
docker run --name pgvector-db -e POSTGRES_USER=admin -e POSTGRES_PASSWORD=admin -e POSTGRES_DB=vector_db -p 5432:5432 -d ankane/pgvector
```
* Executa **PostgreSQL** com PGVector na porta `5432`.
* Banco de dados padrão é `vector_db` com usuário `admin` e senha `admin`.

### **🔹 Elasticsearch**
```bash
docker run -d --name elasticsearch -p 9200:9200 -e "discovery.type=single-node" docker.elastic.co/elasticsearch/elasticsearch:8.5.1
```
* Executa **Elasticsearch** na porta `9200`.
* Modo de nó único habilitado para uso local.

## LLMs

O módulo @cmmv/ai inclui suporte para múltiplos LLMs (Modelos de Linguagem de Grande Escala), permitindo integração flexível com diferentes provedores. Atualmente, os seguintes modelos são suportados:

* **DeepSeek** – Otimizado para programação e pesquisa técnica.
* **Gemini (Google)** – Um LLM multimodal com capacidades avançadas de raciocínio.
* **Hugging Face** – Compatível com modelos de código aberto como CodeLlama, MiniLM, DeepSeek e outros.
* **OpenAI (ChatGPT)** – Integração com modelos como GPT-4 e GPT-3.5.
* **Ollama (Facebook)** – Execução de modelos locais para aplicações focadas em privacidade.
* **Groq (X)** – Inferência de alta velocidade com modelos LLama-3, Mixtral e Gemma.

| **Provedor LLM**  | **Modelo Padrão**                      | **Requer Chave de API** |
|-------------------|--------------------------------------|-----------------------|
| **AI21 Labs**     | `j1-jumbo`, `j1-large`               | Sim                   |
| **Aleph Alpha**   | `luminous-base`, `luminous-extended` | Sim                   |
| **Anthropic**     | `claude-3-haiku-20240307`           | Sim                   |
| **AWS Bedrock**   | Vários modelos (Claude, Mistral, etc.) | Sim             |
| **Cohere**        | `command-xlarge-nightly`, `command-medium` | Sim        |
| **DeepInfra**     | Vários modelos                      | Sim                   |
| **DeepSeek**      | `deepseek-ai/deepseek-coder-7b`     | Não                   |
| **Fireworks**     | Vários modelos                      | Sim                   |
| **Google Gemini** | `gemini-1.5-pro`                   | Sim                   |
| **Google Vertex AI** | `text-bison@001`                   | Sim                   |
| **Groq**          | `llama3-8b`, `mixtral`              | Sim                   |
| **Hugging Face**  | `code-llama`, `MiniLM`, etc.        | Não                   |
| **Mistral AI**    | `mistral-7b`, `mixtral`             | Sim                   |
| **Ollama**        | `llama3`, `mistral`, `gemma`        | Não (execução local)  |
| **OpenAI**        | `gpt-4`, `gpt-3.5`                  | Sim                   |
| **Together AI**   | `GPT-JT-6B-v1`                      | Sim                   |
| **Vertex AI**     | `text-bison@001`                    | Sim                   |

A interface de busca é acessível por meio da classe `Search`, que realiza busca semântica usando embeddings e gera respostas conscientes do contexto.

*[https://v03.api.js.langchain.com/index.html](https://v03.api.js.langchain.com/index.html)*

### Configuração

As configurações do LLM podem ser ajustadas no arquivo `.cmmv.config.cjs`. Abaixo está um exemplo de configuração para usar o modelo Gemini 1.5 Pro da Google:

```javascript
module.exports = {
    ai: {
        llm: {
            provider: "google",  // Opções: "openai", "deepseek", "huggingface", "gemini", "ollama", "groq"
            model: "gemini-1.5-pro", // Modelo padrão para o provedor selecionado
            embeddingTopk: 10, // Número de resultados top-k usados para recuperação de contexto
            textMaxTokens: 2048, // Máximo de tokens por resposta
            apiKey: process.env.GOOGLE_API_KEY, // Chave de API para o provedor selecionado (se necessário)
            language: 'pt-br' // Idioma padrão da resposta
        }
    }
}
```
<br/>

| **Caminho**          | **Descrição**                              | **Valor Padrão / Exemplo**                          |
|-------------------|--------------------------------------------|----------------------------------------------------|
| `llm.provider`   | Provedor de LLM a ser usado                | `"google"` (`"openai"`, `"ollama"`, `"huggingface"`, `"groq"`) |
| `llm.model`      | Modelo LLM usado para respostas            | `"gemini-1.5-pro"` (`"gpt-4"`, `"deepseek-coder-7b"`) |
| `llm.embeddingTopk` | Número de embeddings relevantes a recuperar | `10`                                               |
| `llm.textMaxTokens` | Máximo de tokens por requisição           | `2048`                                             |
| `llm.apiKey`     | Chave de API para acessar o provedor de LLM | `process.env.GOOGLE_API_KEY` (se necessário)       |
| `llm.language`   | Idioma padrão para respostas               | `"pt-br"` (`"en"`, `"es"`, etc.)                   |

### Consultas de Busca

O exemplo a seguir demonstra como usar o LLM para gerar respostas com base em dados vetoriais recuperados.

```typescript
import { Application, Hook, HooksType } from '@cmmv/core';

import {
    PromptTemplate,
    RunnableSequence,
    RunnablePassthrough,
    StringOutputParser,
    Embedding,
    Dataset,
    Search,
} from '@cmmv/ai';

class SearchSample {
    @Hook(HooksType.onInitialize)
    async start() {
        const question = 'Como criar um controlador CMMV?';

        const search = new Search();
        await search.initialize();

        const finalResult = await search.invoke(question);
        console.log(`Resposta do LLM: `, finalResult.content);
    }
}

Application.exec({
    services: [SearchSample],
})
```

### Como Funciona
<br/>

* **Recuperação de Busca**: O sistema de busca recupera os embeddings mais relevantes do banco de dados vetorial.
* **Injeção de Contexto**: Os dados recuperados são inseridos no prompt.
* **Processamento do LLM**: O modelo configurado gera uma resposta com base no contexto fornecido.
* **Saída em JSON**: O resultado é estruturado em formato JSON para integração contínua.

Essa configuração permite respostas precisas e orientadas por contexto, mantendo a flexibilidade na seleção e configuração do modelo. 🚀
