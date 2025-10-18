# Утилиты для хеширования (Hashing Utilities)

В этом руководстве описаны функции для выполнения общих задач, связанных с хешированием в Ethereum, включая подпись сообщений, работу с ENS и кодирование данных в стиле Solidity.

***

## Основные функции хеширования

Эти функции используются для общих криптографических операций.

### **`id()`**
Просто вычисляет хеш **keccak256** от строки в формате UTF-8. Идеально подходит для создания уникальных идентификаторов.

```javascript
import { id } from "ethers";

// Получить 32-байтный идентификатор из строки
const topic = id("hello world");
// '0x47173285a8d7341e5e972fc677286384f802f8ef42a5ec5f03bbfa254cb01fad'
````

### **`hashMessage()`**

Вычисляет хеш сообщения в соответствии со стандартом **EIP-191** (personal-sign). Этот метод добавляет к сообщению префикс `\x19Ethereum Signed Message:\n` и длину сообщения перед хешированием.

**Важно:** Хеширование строки и байтов даёт разный результат\!

```javascript
import { hashMessage, getBytes } from "ethers";

// Хеширование строки "Hello World"
hashMessage("Hello World");
// '0xa1de988600a42c4b4ab089b619297c17d53cffae5d5120d82d8a92d0bb3b78f2'

// Хеширование СТРОКИ "0x4243" (6 символов)
hashMessage("0x4243");
// '0x6d91b221f765224b256762dcba32d62209cf78e9bebb0a1b758ca26c76db3af4'

// Хеширование БАЙТОВ 0x4243 (2 байта)
hashMessage(getBytes("0x4243"));
// '0x0d3abc18ec299cf9b42ba439ac6f7e3e6ec9f5c048943704e30fc2d9c7981438'
```

-----

## Хеширование в стиле Solidity

Эти функции имитируют поведение `abi.encodePacked()` в Solidity, что крайне важно для проверки подписей или вычислений, которые производятся в смарт-контрактах.

### **`solidityPackedKeccak256()`**

Объединяет значения разных типов и вычисляет от них хеш **keccak256**. Это аналог `keccak256(abi.encodePacked(type1, type2, ...))` в Solidity.

```javascript
import { solidityPackedKeccak256 } from "ethers";

const addr = "0x8ba1f109551bd432803012645ac136ddd64dba72";
const value = 45;

const packedHash = solidityPackedKeccak256(
    ["address", "uint"],
    [addr, value]
);
// '0x9465ddbc845149cfc7046bee85c30fd1b52b4f87d9c03ca8a0bd046868763030'
```

Также доступны функции **`solidityPacked()`** (только объединяет данные без хеширования) и **`solidityPackedSha256()`** (использует SHA256).

-----

## Проверка подписей ✅

Эти функции позволяют проверить, кто подписал сообщение, и восстановить адрес подписанта.

### **`verifyMessage()`**

Восстанавливает адрес из подписи, созданной с помощью `personal_sign`.

```javascript
import { verifyMessage } from "ethers";

const message = "Hello World";
// Подпись, полученная от signer.signMessage(message)
const signature = "0x..."; 

const recoveredAddress = verifyMessage(message, signature);
console.log("Сообщение подписал:", recoveredAddress);
```

### **`verifyTypedData()`**

Восстанавливает адрес из подписи типизированных данных (EIP-712).

-----

## Подпись типизированных данных (EIP-712)

**EIP-712** — это стандарт для подписи структурированных данных, которые отображаются пользователю в читаемом формате, а не в виде непонятной hex-строки.

### Основные компоненты:

  * **`TypedDataDomain`**: Описывает контекст подписи (название DApp, версия, `chainId`, адрес контракта).
  * **`types`**: Описывает структуру самих данных, которые подписываются (например, `struct Mail { string from; string to; }`).

### **`TypedDataEncoder`**

Это класс-помощник для кодирования и хеширования данных по стандарту EIP-712.

**Основной метод: `TypedDataEncoder.hash()`**
Он вычисляет итоговый хеш, который затем должен подписать пользователь.

```javascript
import { TypedDataEncoder } from "ethers";

// 1. Описываем домен (контекст)
const domain = {
    name: 'My DApp',
    version: '1',
    chainId: 1,
    verifyingContract: '0xCcCCccccCCCCcCCCCCCcCcCccCcCCCcCcccccccC'
};

// 2. Описываем структуру данных
const types = {
    Person: [
        { name: 'name', type: 'string' },
        { name: 'wallet', type: 'address' }
    ],
    Mail: [
        { name: 'from', type: 'Person' },
        { name: 'to', type: 'Person' },
        { name: 'contents', type: 'string' }
    ]
};

// 3. Создаем объект с данными
const value = {
    from: { name: 'Alice', wallet: '0x...' },
    to: { name: 'Bob', wallet: '0x...' },
    contents: 'Hello!'
};

// 4. Получаем хеш для подписи
const digest = TypedDataEncoder.hash(domain, types, value);
// '0x...' - этот хеш нужно передать signer.sign(digest) или signer._signTypedData(...)
```

-----

## Функции для работы с ENS

  * **`ensNormalize(name)`**: Нормализует ENS-имя (например, приводит к нижнему регистру).
  * **`isValidName(name)`**: Проверяет, является ли имя валидным ENS-именем.
  * **`namehash(name)`**: Вычисляет `namehash` для ENS-имени, который используется для поиска резолверов в реестре ENS.

<!-- end list -->

```
```