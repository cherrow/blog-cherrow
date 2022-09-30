---
title: JavaScript简洁之道
toc: true
categories:
- 工程实践
  tags:
- 代码规范
  abbrlink: 5772e7ea
  date: 2022-09-25 15:41:58
---

《Clean Code》这本书我在之前的博客中也有提到，不得不说它对我现在良好的编码风格有着极大的影响，该书虽然是以 Java 为模板来编写的，但其中大多数的思想与范式可以适用于任何语言。这篇博客中的原文（原文：[clean-code-javascript](https://github.com/ryanmcdermott/clean-code-javascript)，翻译版：[JavaScript 风格指南](https://github.com/alivebao/clean-code-js)）就是在借鉴了《Clean Code》之后为 JavaScript 设计编写的，我在原文中挑了一些比较接地气、很快就能上手应用的规则出来，并在每一条中都添加了一些自己在实际项目中的理解，便于大家参考并落地实践。

## 变量

### 使用有意义，可读性好的变量名

**反例**:

```javascript
var yyyymmdstr = moment().format('YYYY/MM/DD');
```

**正例**:

```javascript
var yearMonthDay = moment().format('YYYY/MM/DD');
```

**我的理解**：

项目中看到太多毫无意义、可读性差的变量名了，现在我随便打开一个项目就能看到两个：`list`、`upinfo`，这种命名对于理解代码上下文、捋清业务逻辑没有任何好处，对于有意义的命名，**名字取得再长也是值得的**，如：`showReceiveCardDialog`、`activeCardPack`。

### 使用 ES6 的 const 定义常量
反例中使用"var"定义的"常量"是可变的。

在声明一个常量时，该常量在整个程序中都应该是不可变的。

**反例**:

```javascript
var FIRST_US_PRESIDENT = "George Washington";
```

**正例**:
```javascript
const FIRST_US_PRESIDENT = "George Washington";
```

**我的理解**：

可变性这里不再赘述，想补充一点的是，我们应当尽量避免在代码中出现 Magic number（魔术数），对于魔术数，应当使用 const 来定义成一个名字有业务含义的常量，便于提高代码的可维护性与可读性。如：

```javascript
// 不推荐这样写，这里的 525600 即魔术数，525600 是什么?
for (var i = 0; i < 525600; i++) {
  runCronJob();
}

// 以下为推荐写法
const MINUTES_IN_A_YEAR = 525600;
for (let i = 0; i < MINUTES_IN_A_YEAR; i++) {
  runCronJob();
}
```

### 使用说明变量(即有意义的变量名)
**反例**:
```javascript
const cityStateRegex = /^(.+)[,\\s]+(.+?)\s*(\d{5})?$/;
saveCityState(cityStateRegex.match(cityStateRegex)[1], cityStateRegex.match(cityStateRegex)[2]);
```

**正例**:

```javascript
const ADDRESS = 'One Infinite Loop, Cupertino 95014';
const cityStateRegex = /^(.+)[,\\s]+(.+?)\s*(\d{5})?$/;
const match = ADDRESS.match(cityStateRegex)
const city = match[1];
const state = match[2];
saveCityState(city, state);
```

**我的理解**：

这也是项目中特别常见的问题，在 `if` 的条件判断语句中尤其突出。经常能看到一个 `if` 判断里包含了一大堆逻辑判断，与、或、取反混合运算甚至一行都写不下，认真看半天还没能看懂这个条件是什么意思，如果我们把这个**逻辑判断抽离出来**，用 const 定义一个有含义的变量，那么代码的可读性将会大大提升，如：

```javascript
// 不推荐
if (file.open(fileName, "w") !== null && (...) || (...)) {
    ...
}

// 推荐写法
const existed = (file.open(fileName, "w") !== null) && (...) || (...);
if (existed) {
    ...
}
```

## 函数

### 函数参数 (理想情况下应不超过 2 个)
限制函数参数数量很有必要，这么做使得在测试函数时更加轻松。过多的参数将导致难以采用有效的测试用例对函数的各个参数进行测试。

应避免三个以上参数的函数。通常情况下，参数超过两个意味着函数功能过于复杂，这时需要重新优化你的函数。当确实需要多个参数时，大多情况下可以考虑这些参数**封装成一个对象**。

JS 定义对象非常方便，当需要多个参数时，可以**使用一个对象进行替代**。

**反例**:

```javascript
function createMenu(title, body, buttonText, cancellable) {
  ...
}
```

**正例**:
```javascript
const menuConfig = {
  title: 'Foo',
  body: 'Bar',
  buttonText: 'Baz',
  cancellable: true
}
function createMenu(menuConfig) {
  ...
}
```

**我的理解**：

建议直接作为项目规范，超过 2 个参数需要定义为对象进行传参。

### 避免副作用
当函数产生了除了“接受一个值并返回一个结果”之外的行为时，称该函数产生了副作用。比如在函数中更新dom、修改全局变量等就是产生了副作用。

**反例**:

```javascript
// 如果有其他函数也需要用到 name 变量，那么这时 name 已经不再是原来的样子了
var name = 'Ryan McDermott';
function splitIntoFirstAndLastName() {
  name = name.split(' ');
}
splitIntoFirstAndLastName();
console.log(name); // ['Ryan', 'McDermott'];
```

**正例**:
```javascript
// 将 name 作为参数传进去，并返回一个新的变量，以保证不会对输入以及函数之外的变量有任何副作用
function splitIntoFirstAndLastName(name) {
  return name.split(' ');
}
var name = 'Ryan McDermott'
var newName = splitIntoFirstAndLastName(name);
console.log(name); // 'Ryan McDermott';
console.log(newName); // ['Ryan', 'McDermott'];
```

**我的理解**：

这里要说的就是**尽量使用和编写纯函数**，纯函数即不会产生副作用的函数，这样能最大程度避免莫名其妙的 bug 产生。

### 采用函数式编程
函数式的编程具有更干净且便于测试的特点。尽可能的使用这种风格吧。

**反例**:
```javascript
const programmerOutput = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  }, {
    name: 'Suzie Q',
    linesOfCode: 1500
  }, {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  }, {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
];
const totalOutput = 0;
for (let i = 0; i < programmerOutput.length; i++) {
  totalOutput += programmerOutput[i].linesOfCode;
}
```

**正例**:
```javascript
const programmerOutput = [
  {
    name: 'Uncle Bobby',
    linesOfCode: 500
  }, {
    name: 'Suzie Q',
    linesOfCode: 1500
  }, {
    name: 'Jimmy Gosling',
    linesOfCode: 150
  }, {
    name: 'Gracie Hopper',
    linesOfCode: 1000
  }
];
const totalOutput = programmerOutput
  .map((programmer) => programmer.linesOfCode)
  .reduce((acc, linesOfCode) => acc + linesOfCode, 0);
```

**我的理解**：

不得不说，函数式编程特别方便与简洁，可项目中依然有使用上古方法一堆 `for` 循环做命令式编程的。对于数组，请务必尽情地使用 `filter`, `map`, `reduce`, `find`, `sort` 吧！

### 封装判断条件

**反例**:
```javascript
if (fsm.state === 'fetching' && isEmpty(listNode)) {
  /// ...
}
```

**正例**:
```javascript
function shouldShowSpinner(fsm, listNode) {
  return fsm.state === 'fetching' && isEmpty(listNode);
}
if (shouldShowSpinner(fsmInstance, listNodeInstance)) {
  // ...
}
```

**我的理解**：

这一点和[变量](#使用说明变量(即有意义的变量名))一节中提到的其实是一个事情，至于抽成变量还是抽成方法，就看需不需要传参了。

## **注释**
### 只对存在一定业务逻辑复杂性的代码进行注释
注释并不是必须的，好的代码能够让人一目了然，不用过多无谓的注释。

**反例**:

```javascript
function hashIt(data) {
  // The hash
  var hash = 0;
  // Length of string
  var length = data.length;
  // Loop through every character in data
  for (var i = 0; i < length; i++) {
    // Get character code.
    var char = data.charCodeAt(i);
    // Make the hash
    hash = ((hash << 5) - hash) + char;
    // Convert to 32-bit integer
    hash = hash & hash;
  }
}
```

**正例**:
```javascript
function hashIt(data) {
  var hash = 0;
  var length = data.length;
  for (var i = 0; i < length; i++) {
    var char = data.charCodeAt(i);
    hash = ((hash << 5) - hash) + char;
    // Convert to 32-bit integer
    hash = hash & hash;
  }
}
```

**我的理解**：

大家可以去看看自己的项目，代码里面充斥着多少毫无意义、丑陋不堪的注释，很多开发同学喜欢每个函数甚至每个变量都写上注释，建议把这些注释全删掉，**重新给你的变量和函数取一个通俗易懂的名字**吧，如：

```javascript
// 用户信息
const info = {}
// 获取用户信息
function getInfo() {
}

// 下面这样是不是清爽多了？
const user = {}
function getUser() {
}

```

### 不要在代码库中遗留被注释掉的代码
版本控制的存在是有原因的。让旧代码存在于你的 history 里吧。

**反例**:
```javascript
doStuff();
// doOtherStuff();
// doSomeMoreStuff();
// doSoMuchStuff();
```

**正例**:
```javascript
doStuff();
```

**我的理解**：

这也是个大问题，所有被注释的代码似乎都有着同一个使命：“以后说不定用得上呢”。我一贯坚持当前不用的代码就直接删掉，坚决不提交到 git 上去，现在 git 这么方便，还怕要用的代码找不回来吗？注释的代码只会让你的项目越来越臃肿越来越难看，看到这里，快去把你项目中的注释删掉吧。


