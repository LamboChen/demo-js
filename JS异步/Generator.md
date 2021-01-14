[TOC]

# 1. 什么是 Generator 函数

在 JavaScript 中，一个函数一旦开始执行，就会运行到最后或遇到return时结束，运行期间不会有其它代码能够打断它，也不能从外部再传入值到函数体内

而 Generator 函数（生成器）的出现使得打破函数的完整运行成为了可能，其语法行为与传统函数完全不同

Generator 函数是 ES6 提供的一种异步编程解决方案，形式上也是一个普通函数，但有几个显著的特征：
 
- function 关键字与函数名之间有一个星号 "*" （推荐紧挨着 function 关键字）
- 函数体内使用 yield 表达式，定义不同的内部状态 （可以有多个 yield ）
- 直接调用 Generator 函数并不会执行，也不会返回运行结果，而是返回一个遍历器对象（Iterator Object）
- 依次调用遍历器对象的next方法，遍历 Generator 函数内部的每一个状态

代码示例：

```js
{
  function* gen() {
    yield 'hello'
    yield 'world'
    return 'ending'
  }

  let it = gen()

  it.next()   // {value: "hello", done: false}
  it.next()   // {value: "world", done: false}
  it.next()   // {value: "ending", done: true}
  it.next()   // {value: undefined, done: true}
}
```

第四次调用 next 方法时，由于函数已经遍历运行完毕，不再有其它状态，因此返回 {value: undefined, done: true}。如果继续调用 next 方法，返回的也都是这个值

# 2. yield、yield*、return、iterator.next()、for...of

### a. yield

- yield 表达式只能用在 Generator 函数里面，用在其它地方都会报错
- yield 表达式如果用在另一个表达式中，必须放在圆括号里面
- yield 表达式用作参数或放在赋值表达式的右边，可以不加括号
- yield 表达式和 return 语句的区别 (yield 表示停顿、return 表示终止)

### b. yield*

如果在 Generator 函数里面调用另一个 Generator 函数，默认情况下是没有效果的, yield* 表达式用来在一个 Generator 函数里面 执行 另一个 Generator 函数

```js
{
  function* foo() {
    yield 'aaa'
    yield 'bbb'
  }

  function* bar() {
    yield* foo()      // 在bar函数中 **执行** foo函数
    yield 'ccc'
    yield 'ddd'
  }

  let iterator = bar()

  for(let value of iterator) {
    console.log(value)
  }

  // aaa
  // bbb
  // ccc
  // ddd
}
```

### c. return

方法返回，对应 done 属性为  true

### d. iterator.next()

用于遍历 Generator 返回值 - 迭代器对象

next() 参数：yield 表达式本身没有返回值，或者说总是返回 undefined。next 方法可以带一个参数，该参数就会被当作上一个 yield 表达式的返回值

```js
{
  function* gen(x) {
    let y = 2 * (yield (x + 1))   // 注意：yield 表达式如果用在另一个表达式中，必须放在圆括号里面
    let z = yield (y / 3)
    return x + y + z
  }

  let it = gen(5)

  console.log(it.next())  // 正常的运算应该是先执行圆括号内的计算，再去乘以2，由于圆括号内被 yield 返回 5 + 1 的结果并暂停，所以返回{value: 6, done: false}
  console.log(it.next(9))  // 上次是在圆括号内部暂停的，所以第二次调用 next方法应该从圆括号里面开始，就变成了 let y = 2 * (9)，y被赋值为18，所以第二次返回的应该是 18/3的结果 {value: 6, done: false}
  console.log(it.next(2))  // 参数2被赋值给了 z，最终 x + y + z = 5 + 18 + 2 = 25，返回 {value: 25, done: true}
}
```
Generator 函数从暂停状态到恢复运行，它的上下文状态（context）是不变的。通过 next 方法的参数，就有办法在 Generator 函数开始运行之后，继续向函数体内部注入值。

也就是说，可以在 Generator 函数运行的不同阶段，从外部向内部注入不同的值，从而调整函数行为。


Tips:

Generator 函数返回的遍历器对象，还有一个 return 方法，可以返回给定的值(若没有提供参数，则返回值的value属性为 undefined)，并且**终结**遍历 Generator 函数

```js
{
  function* gen() {
    yield 1
    yield 2
    yield 3
  }

  let it = gen()

  it.next()             // {value: 1, done: false}
  it.return('ending')   // {value: "ending", done: true}
  it.next()             // {value: undefined, done: true}
}
```

### e. for...of

由于 Generator 函数运行时生成的是一个 Iterator 对象，因此，可以直接使用 for...of 循环遍历，且此时无需再调用 next() 方法
 
这里需要注意，一旦 next() 方法的返回对象的 done 属性为 true，for...of 循环就会终止，且不包含该返回对象

```js
{
  function* gen() {
    yield 1
    yield 2
    yield 3
    yield 4
    return 5
  }
  
  for(let item of gen()) {
    console.log(item)
  }
  
  // 1 2 3 4
}
```

# 3. Promise、Generator、asynv/await

### a. Promise 

Promise 对象是一个代理对象（代理一个值），被代理的值在 Promise 对象创建时可能是未知的。它允许你为异步操作的成功和失败分别绑定相应的处理方法（handlers）。 这让异步方法可以像同步方法那样返回值，但并不是立即返回最终执行结果，而是一个能代表未来出现的结果的 Promise 对象 

### b. Generator

function* 这种声明方式(function关键字后跟一个星号）会定义一个生成器函数(generator function)，它返回一个 Generator 对象。生成器函数在执行时能暂停，后面又能从暂停处继续执行。

```js
function getApi(params) {
  return new Promise((resolve) => {
    // 模拟ajax
    setTimeout(() => {
      resolve('api result: ' + params)
    }, 1000)
  })
}

function* gen(stage0) {
  console.log(stage0)
  let stage1 = yield getApi('startParams')
  console.log('stage1', stage1)
  let stage2 = yield getApi(stage1)
  console.log('stage2', stage2)
  let stage3 = yield getApi(stage2)
  console.log('stage3', stage3)
  return 'all Done!!'
}

function run(generator, v) {
  let { value, done } = generator.next(v)
  if (!done) {
    value.then((res) => {
      run(generator, res)
    })
  } else {
    console.log(value)
  }
}

run(gen('start'))
```

### c. async/await

ES 2017 新语法，Generator + Promise 的语法糖。

```js
function getApi(params) {
  return new Promise((resolve) => {
    // 模拟ajax
    setTimeout(() => {
      resolve('api result: ' + params)
    }, 1000)
  })
}

async function getAllApi() {
  let stage1 = await getApi('startParams')
  console.log('stage1', stage1)
  let stage2 = await getApi(stage1)
  console.log('stage2', stage2)
  let stage3 = await getApi(stage2)
  console.log('stage3', stage3)
  return 'all Done!!'
}

getAllApi()
```

Reference 
* [一次搞懂 Generator 函数](https://www.cnblogs.com/rogerwu/p/10764046.html)
* [generator，promise 与 async/await 的关系](https://segmentfault.com/a/1190000022270916)
* [更強大的非同步模式：從Generator到Async Await](https://medium.com/@xyz030206/%E6%9B%B4%E5%BC%B7%E5%A4%A7%E7%9A%84%E9%9D%9E%E5%90%8C%E6%AD%A5%E6%A8%A1%E5%BC%8F-%E5%BE%9Egenerator%E5%88%B0async-await-7235f24d00eb)