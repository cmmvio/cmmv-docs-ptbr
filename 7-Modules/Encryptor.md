# Criptografia

Repositório: [https://github.com/cmmvio/cmmv-encryptor](https://github.com/cmmvio/cmmv-encryptor)

O módulo ``@cmmv/encryptor`` oferece um conjunto de utilitários criptográficos para aplicações baseadas em CMMV. Ele fornece métodos robustos para criptografia, descriptografia, geração de assinaturas digitais e gerenciamento de chaves usando criptografia de curva elíptica (``ECC``) com ``secp256k1``, ``AES-256-GCM`` para criptografia e BIP32/BIP39 para gerenciamento de carteiras. Este módulo foi projetado para proteger a transmissão e o armazenamento de dados, garantindo confidencialidade, integridade e autenticidade de informações sensíveis.

## Instalação

Para instalar o módulo ``@cmmv/encryptor``, use o seguinte comando:

```bash
$ pnpm add @cmmv/encryptor
```

### Recursos

<br/>

* **Criptografia Baseada em ECC:** Suporta criptografia de curva elíptica (secp256k1) para gerenciamento seguro de chaves e criptografia.
* **Criptografia AES-256-GCM:** Fornece criptografia usando AES-256-GCM para garantir confidencialidade e autenticidade dos dados.
* **Assinaturas Digitais:** Assina e verifica mensagens e objetos usando chaves privadas, garantindo autenticidade.
* **Gerenciamento de Carteiras BIP32/BIP39:** Gera frases mnemônicas, deriva chaves privadas/públicas e gerencia chaves de acordo com os padrões BIP32 e BIP39.

## Exemplos

### 1. Geração de Chaves

Gere um novo par de chaves privada-pública usando Wallet:

```typescript
import { Wallet } from '@cmmv/encryptor';

const mnemonic = Wallet.generateMnenomic();
const privateKey = Wallet.toPrivate(mnemonic);
const publicKey = Wallet.privateToPublic(privateKey);

console.log(`Mnemônico: \${mnemonic}`);
console.log(`Chave Privada: \${privateKey}`);
console.log(`Chave Pública: \${publicKey}`);
```

### 2. Assinatura de Objetos

Assine um objeto com uma chave privada e verifique sua assinatura:

```typescript
import { Signer } from '@cmmv/encryptor';

const privateKey = 'sua-chave-privada-em-hex';
const transaction = { amount: 100, recipient: 'endereço' };
const { objectHash, signature } = Signer.signObject(privateKey, transaction);

console.log(`Hash: \${objectHash}`);
console.log(`Assinatura: \${signature}`);

const publicKey = Wallet.privateToPublic(privateKey);
const isValid = Signer.verifySignature(objectHash, signature, publicKey);

console.log(`É válido: \${isValid}`);
```

### 3. Criptografia de Dados

Cripte e descriptografe um payload de string usando ECC e ``AES-256-GCM``:

```typescript
import { Encryptor } from '@cmmv/encryptor';

// Chaves pública e privada do destinatário
const recipientPublicKey = 'sua-chave-pública-do-destinatário-em-hex';
const recipientPrivateKey = 'sua-chave-privada-do-destinatário-em-hex';

// Criptografar um payload
const payload = "Mensagem confidencial";
const encryptedData = Encryptor.encryptPayload(recipientPublicKey, payload);

console.log('Dados Criptografados:', encryptedData);

// Descriptografar o payload
const decryptedPayload = Encryptor.decryptPayload(
    recipientPrivateKey,
    {
        encrypted: encryptedData.payload,
        iv: encryptedData.iv,
        authTag: encryptedData.authTag,
    },
    encryptedData.ephemeralPublicKey
);

console.log(`Payload Descriptografado: \${decryptedPayload}`);
```

## Carteiras

A classe ``Wallet`` fornece métodos para lidar com operações criptográficas relacionadas à geração de carteiras, derivação de chaves e geração de endereços usando os padrões BIP32 e BIP39. Ela suporta geração de mnemônicos, derivação de chaves privadas e públicas e conversão para o Formato de Importação de Carteira (WIF). Esta classe foi projetada para funcionar com várias redes blockchain, como Bitcoin e Ethereum.

Para usar a classe ``Wallet``, certifique-se de importá-la:

```typescript
import { Wallet } from '@cmmv/encryptor';
```

### ``getEntropyForWordCount(size: number)``

Retorna o tamanho da entropia em bits com base na contagem de palavras de uma frase mnemônica.

```typescript
const entropy = Wallet.getEntropyForWordCount(12); // Retorna 128 para 12 palavras
```

### ``generateMnenomic(size: number = 24, wordlists: string[] = bip39.wordlists.english)``

Gera uma frase mnemônica usando uma contagem de palavras e uma lista de palavras especificadas.

```typescript
const mnemonic = Wallet.generateMnenomic(12);
```

### ``entropyToMnemonic(entropy: Buffer | string, wordlists: string[] = bip39.wordlists.english)``

Converte um valor de entropia dado em uma frase mnemônica.

```typescript
const mnemonic = Wallet.entropyToMnemonic('000102030405060708090a0b0c0d0e0f');
```

### ``randomByteMnemonic(wordlists: string[] = bip39.wordlists.english)``

Gera uma frase mnemônica usando 32 bytes de entropia aleatória.

```typescript
const mnemonic = Wallet.randomByteMnemonic();
```

### ``getSeed(mnemonic: string)``

Converte uma frase mnemônica em uma semente em formato hexadecimal.

```typescript
const seed = Wallet.getSeed(mnemonic);
```

### ``getSeedBuffer(mnemonic: string)``

Converte uma frase mnemônica em uma semente como um Buffer.

```typescript
const seedBuffer = Wallet.getSeedBuffer(mnemonic);
```

### ``toPrivate(mnemonic: string, passphrase: string = '')``

Deriva a chave privada raiz a partir de uma frase mnemônica e uma senha opcional.

```typescript
const privateKey = Wallet.toPrivate(mnemonic, 'minhaSenha');
```

### ``createDerivationPath(bip: number = 44, coinType: number = 0, account: number = 0, change: number = 0, addressIndex: number = 0)``

Cria um caminho de derivação BIP.

```typescript
const path = Wallet.createDerivationPath();
```

### ``toDerivatationPrivateKey(mnemonic: string, derivationPath: string = "m/44'/0'/0'/0/0", passphrase: string = '')``

Deriva a chave privada usando uma frase mnemônica e um caminho de derivação específico.

```typescript
const privateKey = Wallet.toDerivatationPrivateKey(mnemonic, "m/44'/0'/0'/0/0");
```

### ``toRootKey(mnemonic: string, passphrase: string = '', network?: Network)``

Deriva a chave raiz no formato Base58.

```typescript
const rootKey = Wallet.toRootKey(mnemonic);
```

### ``toPublic(mnemonic: string, derivationPath: string = "m/44'/0'/0'/0/0", passphrase: string = '')``

Deriva a chave pública a partir de uma frase mnemônica e um caminho de derivação.

```typescript
const publicKey = Wallet.toPublic(mnemonic);
```

### ``privateToPublic(privateKey: string | Uint8Array)``

Deriva a chave pública a partir de uma chave privada.

```typescript
const publicKey = Wallet.privateToPublic(privateKey);
```

### ``bip32ToPublic(bip32Key: any)``

Obtém a chave pública diretamente de uma chave BIP32.

```typescript
const publicKey = Wallet.bip32ToPublic(bip32Key);
```

### ``privateKeyToWIF(privateKey: string, compressed: boolean = true)``

Converte uma chave privada para o Formato de Importação de Carteira (WIF).

```typescript
const wif = Wallet.privateKeyToWIF(privateKey);
```

### ``wifToPrivateKey(wif: string)``

Converte uma chave WIF em uma chave privada.

```typescript
const { privateKey, compressed } = Wallet.wifToPrivateKey(wif);
```

### ``privateKeyToAddress(privateKey: string | Uint8Array | undefined)``

Gera um endereço público a partir de uma chave privada.

```typescript
const address = Wallet.privateKeyToAddress(privateKey);
```

### ``publicKeyToAddress(publicKey: string)``

Gera um endereço público a partir de uma chave pública.

```typescript
const address = Wallet.publicKeyToAddress(publicKey);
```

## Assinador

A classe ``Signer`` fornece métodos para assinatura e verificação criptográfica de objetos e strings usando criptografia de curva elíptica (``secp256k1``). Esta classe é essencial para lidar com assinaturas digitais e verificar autenticidade em aplicações de blockchain e criptomoedas. Ela utiliza ``tiny-secp256k1`` para operações criptográficas e suporta algoritmos de hash como ``sha256`` e ``sha3-256``.

Para usar a classe ``Signer``, certifique-se de importá-la:

```typescript
import { Signer } from '@cmmv/encryptor';
```

Esta classe oferece métodos para assinar, verificar e recuperar chaves públicas com base em mensagens e objetos assinados. Abaixo está uma explicação detalhada de cada método, como usá-los e exemplos de uso.

### ``signObject(privateKeyHex: string, object: any, algorithm: string = "sha3-256")``

Assina um objeto usando a chave privada fornecida e retorna o hash do objeto e a assinatura.

```typescript
const signature = Signer.signObject(privateKeyHex, { key: "value" });
console.log(signature); // { objectHash: '...', signature: '...' }
```

### ``verifySignature(objectHash: string, signatureHex: string, publicKeyHex: string | Uint8Array | undefined)``

Verifica uma assinatura em relação ao hash de um objeto usando a chave pública fornecida.

```typescript
const isValid = Signer.verifySignature(objectHash, signature, publicKeyHex);
console.log(isValid); // true ou false
```

### ``recoverPublicKey(objectHash: string, signatureHex: string, recoveryId: 0 | 1 = 1)``

Recupera a chave pública a partir de uma assinatura e hash de objeto fornecidos.

```typescript
const publicKey = Signer.recoverPublicKey(objectHash, signatureHex, 1);
console.log(publicKey); // Chave pública comprimida em formato hex
```

### ``signString(privateKeyHex: string, message: string, algorithm: string = "sha256")``

Assina uma string ao fazer o hash dela com o algoritmo especificado e retorna a assinatura junto com o hash.

```typescript
const signedMessage = Signer.signString(privateKeyHex, "mensagem para assinar");
console.log(signedMessage); // 'hash:signature'
```

### ``verifyHashSignature(signature: string, publicKeyHex: string)``

Verifica se a assinatura corresponde ao hash e à chave pública fornecidos.

```typescript
const isValid = Signer.verifyHashSignature(signedMessage, publicKeyHex);
console.log(isValid); // true ou false
```

### ``recoverPublicKeyFromHash(signature: string, recoveryId: 0 | 1 = 1)``

Recupera a chave pública a partir de uma assinatura e hash fornecidos.

```typescript
const publicKey = Signer.recoverPublicKeyFromHash(signedMessage);
console.log(publicKey); // Chave pública comprimida em formato hex
```

### ``signedBy(signature: string, publicKeyHex: string)``

Verifica se a mensagem foi assinada pela chave pública fornecida ao verificar ambos os IDs de recuperação.

```typescript
const isSignedBy = Signer.signedBy(signedMessage, publicKeyHex);
console.log(isSignedBy); // true ou false
```

A classe ``Signer`` fornece métodos essenciais para operações de assinatura digital, tornando-a adequada para uso em aplicações de blockchain e criptomoedas onde verificar a autenticidade de transações e mensagens é crítico.

## Criptografador

A classe ``Encryptor`` fornece métodos para criptografar e descriptografar dados usando criptografia de curva elíptica (``secp256k1``) e AES-256-GCM para criptografia. Ela é projetada para transmissão segura de dados usando o protocolo Diffie-Hellman de Curva Elíptica (ECDH) para derivar chaves compartilhadas para criptografia simétrica. Esta classe é essencial para garantir a confidencialidade e autenticidade de informações sensíveis trocadas entre partes.

Para usar a classe ``Encryptor``, importe-a da seguinte forma:

```typescript
import { Encryptor } from '@cmmv/encryptor';
```

### ``encryptPayload(recipientPublicKeyHex: string, payload: string)``

Criptografa um payload usando a chave pública do destinatário. Este método gera um par de chaves efêmeras e deriva um segredo compartilhado usando ECDH. O payload é criptografado usando AES-256-GCM com a chave compartilhada derivada, garantindo confidencialidade e autenticidade.

```typescript
const encryptedData = Encryptor.encryptPayload(recipientPublicKey, "Olá, Mundo!");
console.log('Dados Criptografados:', encryptedData);
// {
//     payload: '0x...',
//     iv: '0x...',
//     authTag: '0x...',
//     ephemeralPublicKey: '0x...'
// }
```

### ``decryptPayload(recipientPrivateKeyHex: string, encryptedData: { encrypted: string, iv: string, authTag: string }, ephemeralPublicKeyHex: string): string``

Descriptografa um payload criptografado usando a chave privada do destinatário. Este método deriva o segredo compartilhado usando a chave privada do destinatário e a chave pública efêmera do remetente. A chave compartilhada é então usada para descriptografar o payload com AES-256-GCM.

```typescript
const decryptedPayload = Encryptor.decryptPayload(
    recipientPrivateKey,
    { encrypted: '0x...', iv: '0x...', authTag: '0x...' },
    ephemeralPublicKey
);
console.log(`Payload Descriptografado: \${decryptedPayload}`); // "Olá, Mundo!"
```

A classe ``Encryptor`` oferece um mecanismo eficiente e seguro para criptografar e descriptografar dados usando criptografia de curva elíptica combinada com AES-256-GCM. Isso garante que tanto a confidencialidade quanto a integridade dos dados sejam mantidas durante a transmissão.
