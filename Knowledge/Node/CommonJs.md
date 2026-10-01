> це система модулів, яка була створена для JavaScript поза браузером. Вона стала стандартом де-факто для **Node.js** з моменту його появи у 2009 році.

До CommonJS JavaScript не мав вбудованої модульної системи. Весь код у браузері жив у глобальному scope, що призводило до конфліктів імен та хаосу. CommonJS вирішив цю проблему для серверного JS.

---

## Як це працює

### Основний механізм

CommonJS базується на трьох ключових елементах:

1. **`require()`** — функція для імпорту модуля
2. **`module.exports`** — об'єкт, через який модуль експортує свій публічний API
3. **`exports`** — скорочення (alias) для `module.exports`

### Приклад

```js
// math.js — експорт
const add = (a, b) => a + b;
const subtract = (a, b) => a - b;

module.exports = { add, subtract };
```

```js
// app.js — імпорт
const math = require('./math');

console.log(math.add(2, 3));      // 5
console.log(math.subtract(5, 2)); // 3
```

### Альтернативний спосіб експорту

```js
// Можна додавати властивості до exports напряму
exports.add = (a, b) => a + b;
exports.subtract = (a, b) => a - b;
```

> ⚠️ **УВАГА:** Не можна перезаписувати `exports` повністю!
> ```js
> // ❌ НЕ працює — ви розриваєте зв'язок з module.exports
> exports = { add, subtract };
> 
> // ✅ Працює — перезаписуємо саме module.exports
> module.exports = { add, subtract };
> ```

---

## Як Node.js завантажує модулі (під капотом)

Коли ви пишете `require('./math')`, Node.js виконує наступне:

### 1. Резолюція шляху (Resolution)

Node.js визначає, який файл завантажити:
- `'./math'` → відносний шлях → шукає `./math.js`, `./math/index.js`
- `'express'` → імʼя пакету → шукає в `node_modules/`

Порядок пошуку файлу:
1. Точний файл: `math.js`
2. З розширенням: `math.js`, `math.json`, `math.node`
3. Як директорія: `math/index.js`

### 2. Обгортка (Wrapper)

Node.js обгортає код модуля у функцію:

```js
(function(exports, require, module, __filename, __dirname) {
    // ваш код модуля тут
});
```

Саме тому:
- Кожен модуль має свій ізольований scope
- `__filename` і `__dirname` доступні в кожному файлі
- `exports` та `module` передаються як параметри

### 3. Виконання

Функція-обгортка виконується, код модуля запускається, і `module.exports` повертається як результат `require()`.

### 4. Кешування

**Модуль виконується тільки ОДИН раз.** Після першого `require()` результат кешується:

```js
// counter.js
let count = 0;
module.exports = { increment: () => ++count, getCount: () => count };
```

```js
// a.js
const counter = require('./counter');
counter.increment(); // count = 1

// b.js
const counter = require('./counter');
console.log(counter.getCount()); // 1 — той самий інстанс!
```

---

## Характеристики CommonJS

| Властивість | Опис |
|---|---|
| **Синхронне завантаження** | `require()` блокує виконання, поки модуль не завантажиться |
| **Динамічний** | `require()` можна викликати будь-де: в if, в циклі, в функції |
| **Runtime evaluation** | Залежності визначаються під час виконання, не до нього |
| **Значення копіюються** | При імпорті ви отримуєте копію (snapshot) значення на момент експорту |
| **Кешування** | Модуль виконується один раз, результат кешується |

---

## Динамічність require()

Одна з головних особливостей — `require()` це звичайна функція, яку можна використовувати де завгодно:

```js
// Умовний імпорт
if (process.env.NODE_ENV === 'production') {
    const logger = require('./prodLogger');
} else {
    const logger = require('./devLogger');
}

// Динамічний шлях
const plugin = require(`./plugins/${pluginName}`);

// В циклі
['module1', 'module2'].forEach(name => {
    const mod = require(`./${name}`);
});
```

Це зручно, але робить **статичний аналіз неможливим** — bundler не може знати заздалегідь, які модулі будуть завантажені.

---

## Копіювання значень (не живі звʼязки)

CommonJS експортує **копію примітивного значення**, а не посилання на нього:

```js
// counter.js
let count = 0;
const increment = () => { count++; };
module.exports = { count, increment };
```

```js
// main.js
const { count, increment } = require('./counter');
console.log(count);  // 0
increment();
console.log(count);  // 0 — все ще 0! Це копія, а не live binding
```

Щоб обійти це, повертають функцію-геттер:

```js
module.exports = { getCount: () => count, increment };
```

---

## Циклічні залежності

CommonJS дозволяє циклічні залежності, але з обмеженнями:

```js
// a.js
console.log('a починається');
exports.done = false;
const b = require('./b'); // переходимо в b.js
console.log('в a.js, b.done =', b.done);
exports.done = true;
console.log('a закінчується');
```

```js
// b.js
console.log('b починається');
exports.done = false;
const a = require('./a'); // отримуємо НЕПОВНИЙ експорт a (done = false)
console.log('в b.js, a.done =', a.done); // false — бо a ще не закінчив виконання
exports.done = true;
console.log('b закінчується');
```

Вивід:
```
a починається
b починається
в b.js, a.done = false
b закінчується
в a.js, b.done = true
a закінчується
```

---

## Як позначити файл як CommonJS

- Розширення `.cjs` — завжди CommonJS
- Розширення `.js` — CommonJS, **якщо** в найближчому `package.json` **немає** `"type": "module"`
- В `package.json`: `"type": "commonjs"` (або відсутність поля `type`)

---

## Переваги

- Простий і зрозумілий синтаксис
- Динамічність — можна завантажувати модулі умовно
- Величезна екосистема (npm побудований на CJS)
- Кешування підвищує продуктивність

## Недоліки

- Синхронне завантаження — не підходить для браузера (мережа повільна)
- Неможливий статичний аналіз → ускладнює tree-shaking
- `module.exports` / `exports` — заплутаність для новачків
- Копії значень замість live bindings
- Не є офіційним стандартом мови JavaScript

---

## Зв'язок з іншими нотатками

- [[ECMAScript Modules]] — сучасна альтернатива, стандарт мови
