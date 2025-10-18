# Ethers.js: Начало работы

Это краткое руководство по **Ethers.js**, которое охватывает основные операции для взаимодействия с блокчейном Ethereum. 🚀

***

## Установка

Вы можете добавить Ethers.js в свой проект, используя NPM, или подключить его напрямую в браузере.

### NPM
Установите пакет через терминал:
```bash
npm install ethers
````

Затем импортируйте необходимые компоненты в ваш код:

```javascript
// Импортировать всю библиотеку
import { ethers } from "ethers";

// Импортировать только определённые функции
import { BrowserProvider, parseUnits } from "ethers";
```

### Браузер

Подключите библиотеку через CDN, добавив следующий тег в ваш HTML-файл:

```html
<script type="module">
  import { ethers } from "[https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js](https://cdnjs.cloudflare.com/ajax/libs/ethers/6.7.0/ethers.min.js)";

  // Ваш код здесь...
</script>
```

-----

## Ключевые понятия

  * **Provider**: Ваш "мост" к блокчейну в режиме **только для чтения**. Позволяет получать данные, такие как баланс счёта, информация о блоках или транзакциях.
  * **Signer**: Объект, который может **подписывать транзакции** и сообщения от имени аккаунта. Необходим для любых операций, изменяющих состояние блокчейна (например, отправка ETH).
  * **Contract**: JavaScript-объект, представляющий конкретный смарт-контракт в блокчейне. Через него вы можете вызывать функции контракта.
  * **Transaction**: Операция, отправленная в блокчейн для изменения его состояния (например, перевод токенов).

-----

## Подключение к Ethereum

### 🦊 Через MetaMask (и другие браузерные кошельки)

Это самый простой способ для веб-приложений. Код определяет, установлен ли MetaMask, и подключается к нему.

```javascript
let provider;
let signer = null;

if (window.ethereum == null) {
    // Если MetaMask не найден, используем провайдера по умолчанию (только чтение)
    console.log("MetaMask не установлен!");
    provider = ethers.getDefaultProvider();
} else {
    // Подключаемся к провайдеру MetaMask
    provider = new ethers.BrowserProvider(window.ethereum);
    
    // Запрашиваем доступ к аккаунту для подписи транзакций
    signer = await provider.getSigner();
}
```

### 🖥️ Через JSON-RPC (собственный узел или Infura)

Если у вас есть URL-адрес узла Ethereum (например, от Infura, Alchemy или вашего собственного Geth/Hardhat), вы можете подключиться напрямую.

```javascript
// Подключение к вашему узлу
const provider = new ethers.JsonRpcProvider("http://localhost:8545"); // Замените на ваш URL

// Получаем Signer'a от узла (если узел управляет аккаунтами)
const signer = await provider.getSigner();
```

-----

## Основные операции

### Чтение данных из блокчейна

Используйте **Provider** для получения информации.

```javascript
// Получить номер последнего блока
const blockNumber = await provider.getBlockNumber();
console.log("Последний блок:", blockNumber);

// Получить баланс аккаунта (результат в wei)
const balanceInWei = await provider.getBalance("ethers.eth");

// Конвертировать wei в ether для удобного отображения
const balanceInEth = ethers.formatEther(balanceInWei);
console.log("Баланс:", balanceInEth, "ETH");
```

### Отправка транзакций

Для отправки транзакций нужен **Signer**.

```javascript
// Создаем объект транзакции
const tx = await signer.sendTransaction({
  to: "ethers.eth",
  value: ethers.parseEther("0.1") // Конвертируем 0.1 ETH в wei
});

console.log("Хэш транзакции:", tx.hash);

// Ожидаем подтверждения транзакции в блокчейне
const receipt = await tx.wait();
console.log("Транзакция подтверждена в блоке:", receipt.blockNumber);
```

-----

## Взаимодействие со смарт-контрактами

Для работы с контрактом вам нужен его **адрес** и **ABI** (Application Binary Interface).

### Создание экземпляра контракта

ABI — это, по сути, "меню" функций контракта.

```javascript
const daiAddress = "dai.tokens.ethers.eth"; // Адрес контракта DAI

// Упрощенный ABI для ERC-20 токена
const daiAbi = [
  "function symbol() view returns (string)",
  "function balanceOf(address) view returns (uint)",
  "function transfer(address to, uint amount)",
  "event Transfer(address indexed from, address indexed to, uint amount)"
];

// Создаем объект контракта
const daiContract = new ethers.Contract(daiAddress, daiAbi, provider);
```

### Чтение данных из контракта

Для `view` функций (только чтение) достаточно **Provider**.

```javascript
const symbol = await daiContract.symbol();
const balance = await daiContract.balanceOf("ethers.eth");

console.log(`Символ токена: ${symbol}`);
console.log(`Баланс: ${ethers.formatUnits(balance, 18)} ${symbol}`);
```

### Изменение состояния контракта

Для отправки транзакций (например, `transfer`) нужно подключить контракт к **Signer**.

```javascript
// Подключаем Signer к нашему объекту контракта
const contractWithSigner = daiContract.connect(signer);

// Количество токенов для отправки (1 DAI с 18 знаками)
const amount = ethers.parseUnits("1.0", 18);

// Вызываем функцию transfer, которая отправит транзакцию
const tx = await contractWithSigner.transfer("адрес_получателя", amount);
await tx.wait(); // Ждем подтверждения
```

### Прослушивание событий

Вы можете в реальном времени отслеживать события, которые испускает контракт.

```javascript
daiContract.on("Transfer", (from, to, amount, event) => {
    console.log(`Перевод ${ethers.formatEther(amount)} от ${from} к ${to}`);
});
```

-----

## Подпись сообщений

Вы можете использовать **Signer** для подписи любого текстового сообщения. Это часто применяется для аутентификации пользователя без отправки транзакции.

```javascript
const message = "Войти на сайт example.com";

// Подписываем сообщение
const signature = await signer.signMessage(message);
console.log("Подпись:", signature);

// Проверяем, кто подписал сообщение
const recoveredAddress = ethers.verifyMessage(message, signature);
console.log("Адрес подписавшего:", recoveredAddress); // Должен совпадать с signer.address
```

```
```