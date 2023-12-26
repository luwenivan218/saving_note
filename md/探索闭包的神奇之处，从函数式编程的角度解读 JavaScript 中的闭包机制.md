> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7315461550558248969)

深入理解 JavaScript 闭包
==================

JavaScript 中的闭包是一个强大而灵活的概念，它在函数式编程中扮演着重要的角色。在本文中，我们将深入探讨闭包的概念，并从函数式编程的视角来理解它的作用。

什么是闭包？
------

闭包是指函数能够访问定义时的词法作用域的能力。当一个函数被定义时，它会记住自己被创建时的作用域，即使在其他地方执行该函数，它仍然能够访问那个作用域中的变量。

```
function createCounter() {
  let count = 0;

  return function() {
    count++;
    return count;
  };
}

const counter = createCounter();
console.log(counter()); // 输出 1
console.log(counter()); // 输出 2


```

在这个例子中，`createCounter` 函数返回了一个内部函数，这个内部函数形成了闭包，可以访问 `createCounter` 中定义的 `count` 变量。

闭包与函数式编程
--------

在函数式编程范式中，闭包常常被用来创建和返回其他函数，从而实现一些功能性的操作。例如，可以通过闭包来实现柯里化和高阶函数等概念。

### 柯里化（Currying）

柯里化是一种将接受多个参数的函数转化为一系列接受单一参数的函数的技术。闭包在这里发挥了重要作用，因为它使得函数能够 “记住” 之前传递的参数。

```
function curry(fn) {
  return function curried(...args) {
    if (args.length >= fn.length) {
      return fn(...args);
    } else {
      return function(...moreArgs) {
        return curried(...args, ...moreArgs);
      };
    }
  };
}

const add = (a, b, c) => a + b + c;
const curriedAdd = curry(add);

console.log(curriedAdd(1)(2)(3)); // 输出 6
console.log(curriedAdd(1, 2)(3)); // 输出 6


```

### 高阶函数（Higher-Order Functions）

闭包也广泛应用于高阶函数的概念。高阶函数是指接受一个或多个函数作为参数，并且 / 或者返回一个新函数的函数。通过使用闭包，可以轻松地创建高阶函数。

```
function multiplyBy(factor) {
  return function(number) {
    return number * factor;
  };
}

const double = multiplyBy(2);
const triple = multiplyBy(3);

console.log(double(4)); // 输出 8
console.log(triple(4)); // 输出 12


```

### 函数式编程的实际应用

在函数式编程中，闭包常常用于创建私有变量和实现不可变性。例如，可以使用闭包来实现一个简单的计数器：

```
function createCounter() {
  let count = 0;

  return {
    increment: function() {
      count++;
    },
    decrement: function() {
      count--;
    },
    getCount: function() {
      return count;
    }
  };
}

const counter = createCounter();
counter.increment();
counter.increment();
console.log(counter.getCount()); // 输出 2


```

在这个例子中，`count` 是一个私有变量，只能通过返回的对象中的方法进行修改。

### 实际应用场景：缓存函数结果

闭包可以用于创建一个能够缓存函数结果的函数。这在某些计算密集型任务中特别有用。

```
function memoize(fn) {
  const cache = {};

  return function(...args) {
    const key = JSON.stringify(args);
    if (cache[key]) {
      console.log('Fetching from cache...');
      return cache[key];
    } else {
      console.log('Calculating and caching result...');
      const result = fn(...args);
      cache[key] = result;
      return result;
    }
  };
}

const expensiveFunction = (x, y) => {
  console.log('Calculating...');
  return x + y;
};

const memoizedFunction = memoize(expensiveFunction);

console.log(memoizedFunction(2, 3)); // 第一次计算
console.log(memoizedFunction(2, 3)); // 从缓存获取


```

在这个例子中，`memoize` 函数使用闭包来记住之前计算的结果，从而避免重复计算。

总结
--

闭包是 JavaScript 函数式编程中的重要概念，它允许函数 “记住” 其定义时的词法作用域。通过闭包，我们可以创建灵活而强大的函数，实现柯里化、高阶函数等函数式编程的核心概念。深入理解闭包有助于更好地利用函数式编程范式，写出更具表达力和可维护性的代码。