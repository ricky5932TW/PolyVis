# JavaScript 考點筆記（給 C/Python 工程師）

> 策略：只看跟 C/Python 不一樣的雷區，基本語法直接跳過。

---

## 1.1 型別系統的奇怪行為

### == vs ===

- **永遠用 `===`（嚴格相等）**，`==` 會做 type coercion，結果不直覺。
- `===` 不轉型，型別不同直接回傳 `false`。

```js
"0" == 0    // true（"0" 被轉成數字）
"0" === 0   // false（型別不同）
```

### null vs undefined

| 概念 | C | Python | JS |
| --- | --- | --- | --- |
| 空值 | NULL pointer | None | `null`（明確設為空） |
| 未宣告/未賦值 | 無 | 無 | `undefined`（系統自動） |

```js
null == undefined   // true（== 特例，這兩個互相寬鬆相等）
null === undefined  // false（型別不同）
```

記法：**`null` 是你給的空，`undefined` 是 JS 給的空。**

### truthy / falsy

**Falsy 值（只有這六個）：**

```text
false, 0, "", null, undefined, NaN
```

**容易踩的雷——跟 Python 相反：**

| 值 | Python | JS |
| --- | --- | --- |
| `[]` | falsy | **truthy** |
| `{}` | falsy | **truthy** |
| `0` | falsy | falsy（相同） |
| `""` | falsy | falsy（相同） |

```js
if ([])  console.log("truthy"); // 會印出！Python 工程師陷阱
if ({})  console.log("truthy"); // 會印出！
```

---

### 驗收題 1.1：預測輸出再跑 console

```js
console.log([] == false);        // true  — [] 轉成 "" 再轉 0，false 轉 0，0==0
console.log([] == ![]);          // true  — ![] 是 false，所以等同上一行
console.log(null == undefined);  // true  — 特例，唯一互相寬鬆相等的組合
console.log(null === undefined); // false — 型別不同
console.log("0" == 0);           // true  — "0" 被 coerce 成數字 0
console.log("0" === 0);          // false — 型別不同（string vs number）
```

**考試答對關鍵：** `==` 遇到 boolean 會先把 boolean 轉數字，遇到 string/number 會把 string 轉數字。

---

## 1.2 Closure + 函式是一等公民

### Closure 運作原理

內層函式**捕捉（capture）外層函式的變數參考**，即使外層函式已執行完畢，變數仍然存活。

```js
function outer() {
  let count = 0;          // 被 closure 捕捉的私有狀態
  return function() {     // 回傳內層函式
    count++;              // 記住並修改外層變數
    return count;
  };
}

const counter = outer();
counter(); // 1
counter(); // 2  — outer() 早就結束了，但 count 還活著
```

- C：function pointer 做不到這件事（沒有捕捉環境）
- Python：有 closure 但較少作為設計核心；JS 整個生態靠 closure

### 高階函式（Higher-Order Function）

函式可以當**參數**傳入，也可以當**回傳值**。

```js
[1, 2, 3].map(x => x * 2);                      // [2, 4, 6]
[1, 2, 3].filter(x => x > 1);                   // [2, 3]
[1, 2, 3].reduce((acc, x) => acc + x, 0);       // 6
```

### 非同步演化史

理解為什麼語法長這樣：`Callback Hell → Promise → async/await`

#### 1. Callback Hell（舊時代，看懂就好）

```js
getUser(id, function(user) {
  getOrders(user, function(orders) {
    getDetails(orders[0], function(detail) {
      // 巢狀越來越深，難以維護
    });
  });
});
```

#### 2. Promise（中間過渡）

```js
getUser(id)
  .then(user => getOrders(user))
  .then(orders => getDetails(orders[0]))
  .then(detail => console.log(detail))
  .catch(err => console.error(err));
```

#### 3. async/await（現代寫法，主要用這個）

```js
async function fetchDetail(id) {
  try {
    const user   = await getUser(id);
    const orders = await getOrders(user);
    const detail = await getDetails(orders[0]);
    return detail;
  } catch (err) {
    console.error(err);
  }
}
```

`async` 函式永遠回傳 Promise；`await` 只能在 `async` 函式內使用。

---

### 驗收題 1.2：實作 createCounter（不用全域變數）

```js
function createCounter() {
  let count = 0;
  return function() {
    return ++count;
  };
}

const counter = createCounter();
console.log(counter()); // 1
console.log(counter()); // 2
console.log(counter()); // 3
```

**考點：** `count` 活在 closure 裡，不是全域變數，多個 counter 彼此獨立。

```js
const a = createCounter();
const b = createCounter();
a(); // 1
a(); // 2
b(); // 1  — b 有自己的 count，不受 a 影響
```

---

## 1.3 `this` 的行為

### 核心規則

JS 的 `this` 不是 C++ / Python 的 `self`——它**不看函式在哪裡定義，而是看怎麼呼叫**（普通函式）或**在哪裡定義**（箭頭函式）。

#### 普通函式：動態綁定（看呼叫者）

```js
function greet() {
  console.log(this.name);
}

const obj = { name: "Mold", greet };
obj.greet();       // "Mold"  — this 是 obj
const fn = obj.greet;
fn();              // undefined（嚴格模式）或 window.name — this 是全域
```

規則：**點左邊是誰，`this` 就是誰；沒有點，`this` 是全域（或 undefined in strict mode）。**

#### 箭頭函式：語法綁定（看定義位置）

箭頭函式沒有自己的 `this`，繼承外層作用域的 `this`，定義時就固定了。

```js
const obj = {
  name: "Mold",
  regularFn: function() { console.log(this.name); },
  arrowFn: () => console.log(this.name)   // this 繼承自模組/全域，不是 obj
};

obj.regularFn();   // "Mold"      — this 是 obj（動態綁定）
obj.arrowFn();     // undefined   — this 是外層全域，沒有 name
```

#### React class component 的 `.bind(this)` 原因

```js
class Button extends React.Component {
  handleClick() {
    console.log(this.state); // 不 bind 的話，this 是 undefined
  }
  render() {
    // onClick 呼叫時沒有 dot，this 丟失
    return <button onClick={this.handleClick.bind(this)}>click</button>;
  }
}
```

Function component 用箭頭函式或 hooks，`this` 問題完全消失。

---

### 驗收題 1.3：預測輸出

```js
const obj = {
  name: "Mold",
  regularFn: function() { console.log(this.name); },
  arrowFn: () => console.log(this.name)
};

obj.regularFn();        // "Mold"      — obj 呼叫，this = obj
obj.arrowFn();          // undefined   — 箭頭函式，this = 外層（全域無 name）
const fn = obj.regularFn;
fn();                   // undefined   — 沒有 dot，this = 全域
```

---

## 1.4 async/await 與 event loop

### 核心概念：單線程 + event loop

JS 是**單線程**，靠 event loop 模擬高併發：

```text
同步程式碼（call stack）
    ↓ 清空後
microtask queue（Promise .then / await）
    ↓ 清空後
macrotask queue（setTimeout / setInterval / I/O）
```

**關鍵順序：** 同步 → microtask → macrotask

#### macrotask vs microtask

| 類型 | 例子 | 優先度 |
| --- | --- | --- |
| macrotask | `setTimeout`, `setInterval`, I/O 回呼 | 低 |
| microtask | `Promise.then`, `await` 後的程式碼 | 高 |

#### await 是 Promise .then 的語法糖

```js
// 這兩段等價
async function a() {
  const result = await fetch(url);
  console.log(result);
}

function b() {
  fetch(url).then(result => {
    console.log(result);
  });
}
```

#### Top-level await

只能在 ES module（`.mjs` 或 `type: "module"`）中使用，普通 script 不行。

```js
// module 內可以
const data = await fetch(url).then(r => r.json());
```

---

### 驗收題 1.4：預測輸出順序

```js
console.log(1);
setTimeout(() => console.log(2), 0);
Promise.resolve().then(() => console.log(3));
console.log(4);
```

### 答案：1 → 4 → 3 → 2

| 步驟 | 動作 | 原因 |
| --- | --- | --- |
| 1 | `1` | 同步，直接執行 |
| 2 | `4` | 同步，直接執行 |
| 3 | `3` | call stack 清空，microtask（Promise）先跑 |
| 4 | `2` | microtask 清空，macrotask（setTimeout）才跑 |

能解釋「microtask 優先於 macrotask」就過關。

---

## 1.5 Module 系統（ESM）

### named export vs default export

```js
// --- foo.js ---
export const PI = 3.14;           // named export
export function add(a, b) { ... } // named export
export default class Foo { ... }  // default export（一個檔案只能一個）

// --- main.js ---
import Foo from "./foo";              // default export — 名字可自取
import { PI, add } from "./foo";     // named export — 名字必須對
import Foo, { PI } from "./foo";     // 兩種一起
```

**快速判斷：**

- `import x from "..."` → default export
- `import { x } from "..."` → named export

### CommonJS 不用管

React / Next.js 全用 ESM。`require()` 和 `module.exports` 是 Node.js 舊格式，看懂就好，不用寫。

---

## 快速記憶卡

| 雷 | 記法 |
| --- | --- |
| 空 array/object 是 truthy | JS 跟 Python 反過來 |
| `null == undefined` 為 true | 唯一的例外，背起來 |
| 永遠用 `===` | `==` 碰到就當錯的看 |
| closure 讓變數「活下來」 | 函式帶著它的出生環境 |
| `async` fn 永遠回傳 Promise | 就算你 return 一個數字也會被包成 Promise |
