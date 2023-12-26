> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7316202800664559626)

前言
--

在现代前端开发中，编写优雅的 JavaScript 代码是一项至关重要的技能。优雅的代码不仅使得程序更易读懂，还有助于提高代码的可维护性和扩展性。以下是一些关键的实践和技巧，帮助你编写更为优雅的 JavaScript 代码。

命名清晰且一致
-------

良好的命名是代码可读性的关键。确保变量、函数和类的命名具有描述性，能够清晰地表达其用途。同时，保持一致性，采用相似的命名规范，有助于形成统一的代码风格。

```
// 不好的例子
let x = 'John'; // x 是什么？
let a = function(){}; // a 是什么？

// 好的例子
let userName = 'John';
let greetUser = function(){};


```

使用常量和枚举
-------

对于不变的值，使用常量来代替硬编码的值，提高代码的可维护性。对于一组相关的常量，可以考虑使用枚举。

```
// 不好的例子
function calculateArea(radius) {
  return 3.14 * radius * radius;
}

// 好的例子
const PI = 3.14;

function calculateArea(radius) {
  return PI * radius * radius;
}


```

利用解构赋值
------

解构赋值是一种简洁而强大的语法，能够快速提取对象或数组中的值，使代码更为紧凑。

```
// 不好的例子
function getFullName(user) {
  return user.firstName + ' ' + user.lastName;
}

// 好的例子
function getFullName({ firstName, lastName }) {
  return `${firstName} ${lastName}`;
}


```

使用箭头函数
------

箭头函数不仅语法简洁，还能更好地处理 `this` 关键字，使代码更为清晰。

```
// 不好的例子
setTimeout(function() {
  console.log('Delayed message');
}, 1000);

// 好的例子
setTimeout(() => {
  console.log('Delayed message');
}, 1000);


```

模块化代码
-----

将代码分割为小的、独立的模块，有助于提高代码的可维护性。每个模块应专注于完成一个特定的任务，降低代码的复杂性。

```
// 不好的例子
function complexTask() {
  // 一大块复杂的代码
}

// 好的例子
import { handleSubtask1, handleSubtask2 } from './helpers';

function complexTask() {
  handleSubtask1();
  handleSubtask2();
}


```

使用模板字符串
-------

模板字符串是一种更灵活、更易读的字符串拼接方式，可以轻松嵌入变量和表达式。

```
// 不好的例子
const greeting = 'Hello, ' + name + '!';

// 好的例子
const greeting = `Hello, ${name}!`;


```

避免全局变量
------

全局变量容易导致命名冲突和代码耦合。尽量将变量限制在局部作用域，避免使用全局变量。

```
// 不好的例子
let globalVar = 10;

function addGlobalVar(num) {
  return num + globalVar;
}

// 好的例子
function addLocalVar(num) {
  let localVar = 10;
  return num + localVar;
}


```

异步操作使用 async/await
------------------

使用 `async/await` 简化异步代码，使其看起来更像同步代码，提高可读性。

```
// 不好的例子
function fetchData() {
  return fetch('https://api.example.com/data')
    .then(response => response.json())
    .then(data => console.log(data))
    .catch(error => console.error(error));
}

// 好的例子
async function fetchData() {
  try {
    const response = await fetch('https://api.example.com/data');
    const data = await response.json();
    console.log(data);
  } catch (error) {
    console.error(error);
  }
}


```

函数参数的默认值与解构
-----------

函数参数的默认值和解构可以在函数签名中提供更多灵活性。这不仅使得函数的调用更清晰，还能避免不必要的空值检查。

```
// 不好的例子
function greet(user) {
  const name = user.name || 'Guest';
  console.log(`Hello, ${name}!`);
}

// 好的例子
function greet({ name = 'Guest' }) {
  console.log(`Hello, ${name}!`);
}


```

使用 Map 替代 Object
----------------

在一些特定场景下，使用 `Map` 对象而不是普通对象可能更为合适。`Map` 提供了更灵活的键值对存储，而且键可以是任意数据类型。

```
// 不好的例子
const userRoles = {};
userRoles['John'] = 'admin';

// 好的例子
const userRoles = new Map();
userRoles.set('John', 'admin');


```

使用 Set 创建唯一值集合
--------------

`Set` 是一种用于存储唯一值的集合。使用 `Set` 可以轻松处理数组中的重复值问题，并且提供了一些集合操作的方法。

```
// 不好的例子
const uniqueNumbers = array.filter((value, index, self) => self.indexOf(value) === index);

// 好的例子
const uniqueNumbers = new Set(array);


```

简化条件语句
------

在一些简单的条件判断中，使用三元运算符能够提高代码的简洁度。但请注意，过度使用三元运算符可能会降低代码的可读性。

```
// 不好的例子
const result = (x > 0) ? 'positive' : 'non-positive';

// 好的例子
const result = x > 0 ? 'positive' : 'non-positive';


```

使用箭头函数进行隐式返回
------------

对于单行函数体，使用箭头函数可以省略 `return` 关键字，使得代码更为简洁。

```
// 不好的例子
const add = function(a, b) {
  return a + b;
};

// 好的例子
const add = (a, b) => a + b;


```

利用 Rest 和 Spread 操作符
--------------------

Rest 操作符（`...`）可以用于将剩余的参数收集成一个数组，而 Spread 操作符也是 `...`，但是用于展开数组或对象。

```
// 不好的例子
function sum(a, b, c) {
  return a + b + c;
}

// 好的例子
function sum(...numbers) {
  return numbers.reduce((acc, num) => acc + num, 0);
}

const arr1 = [1, 2, 3];
const arr2 = [...arr1, 4, 5];


```

使用 Object.keys() 遍历对象
---------------------

`Object.keys()` 方法可以获取对象所有可枚举属性的数组，方便遍历对象的键。

```
const user = {
  name: 'John',
  age: 30,
  role: 'admin',
};

// 不好的例子
for (let key in user) {
  console.log(key, user[key]);
}

// 好的例子
Object.keys(user).forEach(key => {
  console.log(key, user[key]);
});


```

利用 Array 方法操作数组
---------------

数组方法（如 `map`、`filter`、`reduce` 等）是处理数组的强大工具，能够使代码更为简洁和高效。

```
const numbers = [1, 2, 3, 4, 5];

// 不好的例子
let sum = 0;
for (let i = 0; i < numbers.length; i++) {
  sum += numbers[i];
}

// 好的例子
const sum = numbers.reduce((acc, num) => acc + num, 0);


```

使用 Array.from 处理类数组对象
---------------------

`Array.from` 可以将类数组对象（如 NodeList）或可迭代对象转换为真正的数组。

```
// 不好的例子
const nodeList = document.querySelectorAll('.items');
const array = [];
for (let i = 0; i < nodeList.length; i++) {
  array.push(nodeList[i]);
}

// 好的例子
const nodeList = document.querySelectorAll('.items');
const array = Array.from(nodeList);


```

使用 IntersectionObserver 监听元素可见性
-------------------------------

`IntersectionObserver` 可以用于监听元素是否进入视口，避免不必要的性能消耗。

```
const observer = new IntersectionObserver(entries => {
  entries.forEach(entry => {
    if (entry.isIntersecting) {
      console.log('Element is now visible!');
    }
  });
});

const targetElement = document.getElementById('myElement');
observer.observe(targetElement);


```

使用 Intl 处理国际化
-------------

`Intl` 对象提供了对国际化的支持，可以轻松处理日期、时间、数字和货币的格式化。

```
const number = 123456.789;

console.log(new Intl.NumberFormat('en-US').format(number)); // 输出: 123,456.789
console.log(new Intl.NumberFormat('de-DE').format(number)); // 输出: 123.456,789


```

使用 `Object.freeze()` 冻结对象
-------------------------

`Object.freeze()` 方法可以冻结对象，防止对其进行修改。

```
const person = { name: 'John', age: 30 };

// 冻结对象
Object.freeze(person);

// 尝试修改对象属性（无效操作）
person.age = 31;
console.log(person.age); // 输出: 30


```

使用 `Array.from()` 生成数字序列
------------------------

`Array.from()` 可以用于生成数字序列。

```
// 生成数字序列
const numberSequence = Array.from({ length: 5 }, (_, index) => index + 1);
console.log(numberSequence); // 输出: [1, 2, 3, 4, 5]


```

使用 `Array.prototype.fill()` 填充数组
--------------------------------

`fill()` 方法用于将数组的所有元素更改为静态值。

```
const newArray = new Array(5).fill(0);

console.log(newArray); // 输出: [0, 0, 0, 0, 0]


```

结语
--

编写优雅的 JavaScript 代码是一项不断提升的技能，需要在实践中不断学习和积累经验。通过保持代码的清晰、简洁和一致性，你可以提高代码的可读性、可维护性，并使得你的代码更为优雅。在不断进步的前端领域中，不断追求编码技巧的提升，将使你成为更为高效的开发者。