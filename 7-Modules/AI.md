# AI

O módulo **@cmmv/ai** fornece suporte total para **Retrieval-Augmented Generation (RAG)** e, no futuro, **Retrieval-Augmented Synthesis (RAS)** para **LLMs**. Ele permite a compreensão e geração de código, suportando:

- **Embeddings**: Integração com modelos de **Hugging Face**, **LLaMA** e mais.
- **Vector Databases**: Suporte nativo para **Qdrant**, **Neo4j** e outros.
- **LLM Interfaces**: Compatibilidade com **Gemini**, **LLaMA**, **ChatGPT** e mais.
- **LangChain Community**: Construído em `@langchain/community` para alavancar utilitários de IA estáveis.

## Instalar Python

Antes de instalar o HuggingFace CLI, certifique-se de que **Python** esteja instalado no seu sistema.

Execute o seguinte comando para instalar o Python no Ubuntu:

```sh
$ sudo apt update && sudo apt install python3 python3-pip -y
```

Para outros sistemas operacionais, consulte a [página de download oficial do Python](https://www.python.org/downloads/).

## Instalar HuggingFace CLI

Depois que o Python estiver instalado, instale o Hugging Face CLI usando **pip**:

```sh
$ pip3 install -U "huggingface_hub[cli]"
```

## CLI seja reconhecida

Se o seu terminal não reconhecer `huggingface-cli`, adicione `~/.local/bin` ao seu **PATH** do sistema:

```sh
$ echo 'export PATH="$HOME/.local/bin:$PATH"' >> ~/.bashrc
$ source ~/.bashrc
```

Execute o seguinte comando para verificar a instalação:

```sh
$ huggingface-cli --help
```

Se o comando funcionar, a instalação foi bem-sucedida! 🎉

## Autentique com HuggingFace

Para acessar e baixar modelos, você precisa autenticar.

Execute:

```sh
$ huggingface-cli login
```

Você será solicitado a inserir seu **token de acesso do Hugging Face**.
Gere um em: [Tokens do Hugging Face](https://huggingface.co/settings/tokens)
Certifique-se de que o token tenha permissões de **LEITURA**.

## Baixando modelos

Para baixar um modelo, use o seguinte comando:

```sh
$ huggingface-cli download meta-llama/CodeLlama-7B-Python-hf --local-dir ./models/CodeLlama-7B
```

Isso fará o download do modelo **CodeLlama 7B Python** no diretório `./models/CodeLlama-7B`.

Para **CMMV**, defina o caminho do modelo em `.cmmv.config.cjs`:

```js
huggingface: {
    token: process.env.HUGGINGFACE_HUB_TOKEN,
    localModelPath: './models',
    allowRemoteModels: false
},
search: {
    embeddingTopk: 10,
    codeModel: "./models/CodeLlama-7B",
    codeMaxTokens: 512,
    textModel: "google-bert/bert-base-uncased",
    textMaxTokens: 4000,
    baseCodeQuestion: ""
}
```

Agora seu ambiente está configurado para usar modelos Hugging Face com **CMMV**! 🚀

## **Configuração**

Dada a complexidade deste módulo, várias configurações são necessárias, especialmente para definir quais modelos usar em cada estágio. Abaixo está a **configuração base** com todas as propriedades disponíveis atualmente.

### **Exemplo de configuração (`.cmmv.config.cjs`)**
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
            chunkSize: 500,
            chunkOverlap: 100,
            patterns: [
                '../cmmv/**/*.ts',
                '../cmmv/src/**/*.ts',
                '../cmmv/packages/**/*.ts',
                '../cmmv-*/**/*.ts',
                '../cmmv-*/src/*.ts',
                '../cmmv-*/src/**/*.ts',
                '../cmmv-*/packages/**/*.ts',
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
            provider: "qdrant",
            qdrant: {
                url: 'http://localhost:6333',
                collection: 'embeddings'
            }
        },
        search: {
            embeddingTopk: 10,
            useCodeModel: true,
            codeModel: "Xenova/codegen-350M-mono",
            codeMaxTokens: 328,
            textModel: "google-bert/bert-base-uncased",
            textMaxTokens: 4000,
            baseCodeQuestion: ""
        }
    }
};
```

## Convertendo modelos

Alguns **LLMs (Large Language Models)** não são nativamente compatíveis com todas as estruturas de inferência. Um exemplo importante é o **Gemma do Google**, que não é diretamente suportado por muitas ferramentas. Para usar esses modelos de forma eficiente, você precisa **convertê-los para o formato ONNX**.

ONNX (**Open Neural Network Exchange**) é um formato aberto que otimiza modelos para inferência eficiente em várias plataformas. Muitas estruturas de inferência, como **ONNX Runtime**, **TensorRT** e **OpenVINO**, oferecem suporte ao ONNX para uma implantação mais rápida e escalável.

Antes de converter, instale os pacotes necessários:

```sh
$ pip3 install -U "optimum[exporters]" onnx onnxruntime
```

Para converter **o modelo Gemma 2B do Google**, execute:

```sh
$ python3 -m optimal.exporters.onnx --model google/gemma-2b ./models/gemma-2b-onnx
```

## Tokenizer

A classe Tokenizer é responsável por processar e indexar arquivos no pipeline RAG. Ela fornece várias configurações, incluindo:

* Padrões de pesquisa de arquivo (`patterns`): define padrões glob para pesquisar arquivos.
* Exclusões e arquivos ignorados (`ignore`, `exclude`): filtra arquivos desnecessários da indexação.
* Modelo de incorporação (`model`): seleciona o modelo Hugging Face para gerar incorporações.
* Tamanho do índice (`indexSize`): ajusta as dimensões de incorporação com base no modelo selecionado.
* Fragmentação (`chunkSize`, `chunkOverlap`): controla a segmentação de dados para melhor recuperação.
* Geração de metadados KeyBERT (`useKeyBERT`): extrai palavras-chave para aprimorar pesquisas de vetores.

Para outros modelos que exigem autenticação (por exemplo, `LLaMA`), forneça a chave de API em embraceface.token.

### Exemplo de uso:

```typescript
import {Aplicativo, Gancho, TipoDeGanchos} de '@cmmv/core';
import {AIModule} de '@cmmv/ai';

classe TokenizerSample {
    @Hook(HooksType.onInitialize)
    async start() {
        const { Tokenizer } = await import('@cmmv/ai');
        const tokenizer = new Tokenizer();
        tokenizer.start();
    }
}

Aplicativo.exec({
    módulos: [ModuleAIM],
    serviços: [SampleTokenizer]
});
```
<br/>

1. **Verifica diretórios de projeto** com base na configuração `patterns`.
2. **Analisa arquivos TypeScript/JavaScript/Markdown**, extraindo **funções, classes, enums, interfaces, constantes e decoradores**.
3. **Gera embeddings** usando modelos Hugging Face.
4. **Armazena o conjunto de dados** em um arquivo binário `.bin`.

## Usando KeyBERT

KeyBERT é um recurso opcional que aprimora a indexação extraindo palavras-chave relevantes. Ele ajuda a refinar os resultados da pesquisa em **FAISS** ou bancos de dados vetoriais, melhorando a precisão das **consultas LLM**.

Ao contrário de **TF-IDF**, **YAKE!** ou **RAKE**, que dependem de métodos estatísticos, **KeyBERT** aproveita **embeddings BERT** para gerar palavras-chave mais significativas. Isso resulta em melhor filtragem de pesquisa, levando a **respostas baseadas em LLM** mais precisas.

Se o KeyBERT **não estiver habilitado**, o método de extração de palavra-chave padrão será **TF-IDF**, que pode não ser tão preciso, mas é significativamente mais rápido.

Antes de usar o KeyBERT, certifique-se de ter o **Python 3** instalado. Em seguida, instale o KeyBERT usando **pip**:

```bash
$ pip install keybert
```

Uma vez instalado, o KeyBERT será usado **durante a tokenização** para gerar **palavras-chave de filtragem**. Essas palavras-chave melhoram a classificação do conteúdo indexado, tornando os resultados de pesquisa baseados em vetores mais relevantes.

Se você preferir **processamento mais rápido**, você pode desabilitar o KeyBERT, e o sistema retornará ao **TF-IDF**.

Para habilitar o **KeyBERT**, atualize seu arquivo `.cmmv.config.cjs`:

```javascript
module.exports = {
    ai: {
        tokenizer: {
            useKeyBERT: true // Set to false to use TF-IDF instead
        }
    }
};
```

Com **KeyBERT habilitado**, a filtragem de pesquisa se torna mais **contextual**, levando a **respostas LLM mais precisas**.

Para mais detalhes sobre KeyBERT, visite: [Documentação KeyBERT](https://github.com/MaartenGr/KeyBERT).

## Dataset - FAISS & Vector Storage

A classe **Dataset** gerencia **armazenamento vetorizado** para recuperação rápida.

* Salva embeddings em **formato binário** (`.bin`).
* Busca baseada em **FAISS** na memória.
* Suporte futuro para **Neo4j, Qdrant, Pinecone**.

```typescript
const dataset = new Dataset();
dataset.save(); // Salva o conjunto de dados em formato binário
dataset.load(); // Carrega o conjunto de dados na memória
```

Para armazenar e pesquisar **embeddings** de forma eficiente, `@cmmv/ai` suporta **Qdrant, Milvus e Neo4j**.

Para executar esses bancos de dados localmente, use os seguintes
**comandos do Docker**:

### **🔹 Qdrant**
```bash
$ docker run -p 6333:6333 --name qdrant-server qdrant/qdrant
```
<br/>
- Executa um servidor **Qdrant** na porta `6333`.
- API disponível em `http://localhost:6333`.

### **🔹 Milvus**
```bash
$ docker run -p 19530:19530 --name milvus-server milvusdb/milvus
```
<br/>
- Executa **Milvus** na porta `19530`.
- Requer **Python/Node SDK** para interação.

### **🔹 Neo4j**
```bash
$ docker run --publish=7474:7474 --publish=7687:7687 --volume=$HOME/neo4j/data:/data --name neo4j-server neo4j
```
<br/>
- Executa **Neo4j** nas portas `7474` (HTTP) e `7687` (Bolt).
- Os dados são armazenados persistentemente em `$HOME/neo4j/data`.

# Integração futura

O próximo passo é integrar **modelos pré-treinados** para **entendimento e geração de código** usando o conjunto de dados tokenizado.

- [x] Tokenização de **funções, classes, interfaces, decoradores**.
- [x] **Busca de vetores baseada em FAISS** para recuperação na memória.
- [x] Integração com KeyBert para geração de palavras-chave.
- [ ] Externalizar servidor HTTP para comunicação com APIs
- [ ] Integração com **Qdrant, Milvus, Neo4j**.
- [ ] Usar **DeepSeek Code** para **geração de código com tecnologia LLM**.
- [ ] Integrar APIs LLM externas como ChatGPT, Gemini, etc.
