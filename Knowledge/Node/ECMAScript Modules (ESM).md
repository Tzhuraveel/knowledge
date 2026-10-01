> це **офіційна стандартизована** система модулів JavaScript, визначена в специфікації мови (починаючи з ES2015/ES6). На відміну від CommonJS, яка є рішенням спільноти для Node.js, ESM — це частина самої мови.

ESM підтримується:
- Усіма сучасними браузерами (Chrome, Firefox, Safari, Edge)
- Node.js (починаючи з v12 з прапорцем, стабільно з v14+)
- Bundler-ами (Webpack, Rollup, Vite, esbuild)

---

## Як це працює

### Основний синтаксис

**Експорт:**

```js
// math.js

// Іменований експорт (named export)
export const add = (a, b) => a + b;
export const subtract = (a, b) => a - b;

// Або згрупований експорт
const multiply = (a, b) => a * b;
const divide = (a, b) => a / b;
export { multiply, divide };
```

**Імпорт:**

```js
// app.js

// Іменований імпорт
import { add, subtract } from './math.js';

// Імпорт з перейменуванням (alias)
import { add as sum } from './math.js';

// Імпорт усього як namespace object
import * as math from './math.js';
math.add(2, 3);

// Імпорт default export
import Calculator from './Calculator.js';
```

### Default Export

Кожен модуль може мати **один** default export:

```js
// Calculator.js
export default class Calculator {
    add(a, b) { return a + b; }
    subtract(a, b) { return a - b; }
}
```

```js
// app.js — імʼя при імпорті може бути будь-яке
import Calc from './Calculator.js';
import MyCalculator from './Calculator.js'; // теж працює
```

### Комбінування named + default

```js
// utils.js
export const VERSION = '1.0.0';
export const helpers = { /* ... */ };
export default function main() { /* ... */ }
```

```js
// app.js
import main, { VERSION, helpers } from './utils.js';
```

---

## Три фази завантаження модуля

ESM завантажує модулі в три чіткі фази (на відміну від CommonJS, де все відбувається разом):

### 1. Parsing (Розбір)

Движок читає файл і **статично аналізує** всі `import`/`export` — ще до виконання коду. На цьому етапі будується граф залежностей (dependency graph).

```js
// Движок бачить це ДО виконання будь-якого коду:
import { foo } from './a.js';    // → потрібно завантажити a.js
import { bar } from './b.js';    // → потрібно завантажити b.js
```

### 2. Instantiation (Інстанціювання)

Для кожного модуля створюється **module record**. Імпорти та експорти зв'язуються між собою через **live bindings** (живі посилання) — але ще без значень.

### 3. Evaluation (Виконання)

Код модулів виконується у правильному порядку (від листків дерева залежностей до кореня), і live bindings отримують свої значення.

---

## Ключові особливості

### Статичний аналіз

`import` і `export` **мають бути на верхньому рівні** модуля. Їх не можна помістити в `if`, цикл або функцію:

```js
// ❌ Syntax Error
if (condition) {
    import { foo } from './foo.js';
}

// ❌ Syntax Error
function loadModule() {
    export const x = 5;
}
```

Це обмеження — головна перевага ESM. Завдяки ньому:
- Bundler-и можуть робити **tree-shaking** (видаляти невикористаний код)
- IDE можуть давати **автодоповнення** та рефакторинг
- Помилки виявляються **до виконання** коду

### Live Bindings (Живі зв'язки)

На відміну від CommonJS, ESM експортує **живе посилання**, а не копію:

```js
// counter.js
export let count = 0;
export const increment = () => { count++; };
```

```js
// main.js
import { count, increment } from './counter.js';

console.log(count);  // 0
increment();
console.log(count);  // 1 ← значення оновилось! Це live binding!
```

> У CommonJS `count` залишився б `0`, бо це копія. В ESM ви завжди бачите актуальне значення.

**Але!** Імпортовані binding-и є **read-only** — ви не можете їх змінити з боку імпортера:

```js
import { count } from './counter.js';
count = 5; // ❌ TypeError: Assignment to constant variable
```

Тільки модуль-експортер може змінювати свої значення.

### Асинхронне завантаження

ESM завантажується **асинхронно**. У браузері це означає, що модулі не блокують рендеринг:

```html
<!-- Завантажується асинхронно, виконується після парсингу HTML -->
<script type="module" src="./app.js"></script>

<!-- Звичайний скрипт блокує парсинг -->
<script src="./old-app.js"></script>
```

`<script type="module">` автоматично має поведінку `defer`.

---

## Динамічний імпорт — import()

Хоча статичний `import` не можна використовувати умовно, є **динамічний `import()`**:

```js
// Повертає Promise
const module = await import('./heavy-module.js');
module.doSomething();

// Умовне завантаження
if (user.isAdmin) {
    const { AdminPanel } = await import('./admin.js');
}

// Завантаження за динамічним шляхом
const lang = await import(`./locales/${language}.js`);
```

`import()` — це:
- Функція (а не statement), тому її можна використовувати де завгодно
- Повертає **Promise**, що резолвиться в module namespace object
- Корисна для **code splitting** та **lazy loading**

---

## Strict Mode за замовчуванням

ESM модулі **завжди** виконуються в strict mode. Не потрібно писати `'use strict'`:

```js
// В ESM модулі це автоматично strict mode
x = 5;          // ❌ ReferenceError (в non-strict було б глобальна змінна)
delete Object.prototype; // ❌ TypeError
```

---

## Top-level await

В ESM модулях можна використовувати `await` на верхньому рівні (без обгортки в async функцію):

```js
// config.js
const response = await fetch('/api/config');
export const config = await response.json();
```

```js
// app.js
import { config } from './config.js';
// config вже завантажений, бо модуль "чекав" на нього
```

Модулі, що імпортують такий модуль, автоматично чекають на завершення його виконання.

---

## this на верхньому рівні

В ESM `this` на верхньому рівні дорівнює `undefined` (не `global`/`window`):

```js
// ESM
console.log(this); // undefined

// CommonJS
console.log(this); // module.exports (об'єкт)
```

---

## Циклічні залежності в ESM

ESM обробляє циклічні залежності краще за CommonJS завдяки live bindings:

```js
// a.js
import { b } from './b.js';
export const a = 'a value';
console.log('a.js:', b); // 'b value' — працює!

// b.js
import { a } from './a.js';
export const b = 'b value';
console.log('b.js:', a); // може бути undefined якщо a ще не ініціалізовано
```

Завдяки фазі instantiation, binding-и створюються заздалегідь, але значення заповнюються тільки під час evaluation. Тому порядок має значення.

---

## Як позначити файл як ESM

- Розширення **`.mjs`** — завжди ESM
- Розширення `.js` — ESM, **якщо** в найближчому `package.json` є `"type": "module"`
- В `package.json`:
  ```json
  {
      "type": "module"
  }
  ```
- В HTML: `<script type="module" src="..."></script>`

---

## Відмінності від CommonJS

| Характеристика | CommonJS | ESM |
|---|---|---|
| **Синтаксис** | `require()` / `module.exports` | `import` / `export` |
| **Завантаження** | Синхронне | Асинхронне |
| **Аналіз** | Runtime (динамічний) | Статичний (до виконання) |
| **Bindings** | Копія значення | Live binding (посилання) |
| **Умовний імпорт** | `require()` в if — ОК | Тільки через `import()` |
| **this** | `module.exports` | `undefined` |
| **strict mode** | Не за замовчуванням | Завжди strict |
| **Tree-shaking** | ❌ Неможливий | ✅ Можливий |
| **Top-level await** | ❌ Ні | ✅ Так |
| **Браузер** | ❌ Потребує bundler | ✅ Нативна підтримка |
| **Стандарт** | Де-факто (Node.js) | Офіційний (ECMAScript) |
| **Розширення** | `.cjs` | `.mjs` |
| **package.json** | `"type": "commonjs"` | `"type": "module"` |

---

## Інтероп (сумісність між ESM і CJS)

### Імпорт CJS в ESM

```js
// ESM файл може імпортувати CJS модуль
import pkg from './cjs-module.cjs';        // default import = module.exports
import { named } from './cjs-module.cjs';  // може не працювати для всіх модулів
```

Node.js обгортає `module.exports` CJS модуля як default export.

### Імпорт ESM в CJS

```js
// CJS файл НЕ може використовувати статичний import
// Потрібно використовувати динамічний import()
const esmModule = await import('./esm-module.mjs');

// Або через .then()
import('./esm-module.mjs').then(module => {
    module.doSomething();
});
```

> ⚠️ `require()` не може завантажити ESM модуль — це завжди помилка.

---

## Переваги ESM

- Офіційний стандарт мови — працює скрізь однаково
- Tree-shaking — менший розмір бандлу
- Статичний аналіз — кращі інструменти розробки
- Live bindings — завжди актуальні значення
- Асинхронне завантаження — не блокує UI
- Top-level await — зручна асинхронність
- Нативна підтримка в браузерах — без bundler-а для розробки

## Недоліки ESM

- Складніший перехід з CJS екосистеми
- Потрібно вказувати розширення файлу при імпорті в Node.js
- `__filename` та `__dirname` не доступні (потрібно `import.meta.url`)
- Не всі npm пакети мають ESM версію
- JSON імпорт потребує assertion: `import data from './data.json' assert { type: 'json' }`

---

## Заміна __filename і __dirname в ESM

```js
import { fileURLToPath } from 'url';
import { dirname } from 'path';

const __filename = fileURLToPath(import.meta.url);
const __dirname = dirname(__filename);
```

Або в новіших версіях Node.js (v21.2+):
```js
const __dirname = import.meta.dirname;
const __filename = import.meta.filename;
```

---

## Зв'язок з іншими нотатками

- [[CommonJs]] — попередня система модулів для Node.js
