# tricky-node-interview
[Chinese Repo] A bunch of tricky questions I've met while preparing for job interview of Node.js back-end developer.

## 前言
这个 Repo 记录我在准备 Node.js Web 后端开发面试过程中遇到的一些记得一提的问题。     
问题会按照考查的知识模块进行分类且每个分类下的问题基本按照难度递增和关联度进行排序。           
一些问题的完整答案可能需要扎实的编程基础才能理解，同时我会列出可以进一步参考的资料。        
最后，我无法保证每一个回答都能说明得正确、清晰完整且切中要害，如有任何问题和争议，欢迎提 issue 或 PR 指出。       

## JavaScript 语言

### Q：JavaScript 中基本类型（原始类型）有哪些？               
A：6 种基本类型：Boolean、Null、Undefined、Number、String、Symbol（ES6）。其他全部都是 Object（引用类型）。值得一提的是 `typeof` 的一些特殊情形：

1. `typeof Null === 'object'`；
2. `typeof [function] === 'function'`，但实际上 JS 没有 Function 这种类型；

这题一般都是引出后续其他与 JavaScript 类型系统相关的题。

---

### Q：`10 > undefined` 是 `true` 还是 `false`？         
A：`false`。
这一题是明显的不等比较时的类型转换规则，如下：

1. 如果两个操作值都是数值，则进行数值比较
2. 如果两个操作值都是字符串，则比较字符串对应的字符编码值
3. 如果只有一个操作值是数值，则将另一个操作值转换为数值，进行数值比较
4. 如果一个操作数是对象，则调用 `valueOf()` 方法（如果对象没有 `valueOf()` 方法则调用 `toString()` 方法），得到的结果按照前面的规则执行比较
5. 如果一个操作值是布尔值，则将其转换为数值，再进行比较
6. `NaN` 是非常特殊的值，它不和任何类型的值相等，包括它自己，同时它与任何类型的值比较大小时都返回`false`

首先应用规则 3，`Number(undefined)` 为 `NaN`，然后应用规则 6。

---

### Q：`[1] == [1]` 是 `true` 还是 `false`？    
A：`false`。    
这里涉及到两个点：`==` 与 `===` 的区别 以及 JavaScript 的类型系统。     
`==` 与 `===` 的区别在于 `===` 在执行比较运算前不会发生隐式的类型转换，而 `==` 运算发生的隐式类型转换遵循以下规则：

1. 如果一个操作值为布尔值，则在比较之前先将其转换为数值
2. 如果一个操作值为字符串，另一个操作值为数值，则通过 `Number()` 函数将字符串转换为数值
3. 如果一个操作值是对象，另一个不是，则调用对象的 `valueOf()` 方法，得到的结果按照前面的规则进行比较
4. **`null` 与 `undefined` 是相等的**
5. 如果一个操作值为 `NaN`，则相等比较返回 `false`
6. 如果两个操作值都是对象，则比较它们是不是指向同一个对象

可见，这个转换规则还是比较复杂的，这也是为什么编程实践中一律推荐使用 `===` 的原因。`[1] == [1]` 显然应用到了规则 6。这题 tricky 的地方在于容易被 `==` 的类型转换所误导。

**扩展**：
1. [JS Comparison Table](https://dorey.github.io/JavaScript-Equality-Table/)
2. [ECMA-262 11.8 Relational Operators](http://www.ecma-international.org/ecma-262/5.1/#sec-11.8)

---

### Q：JavaScript 中对象的引用传递是怎样的？               
A：首先题干有问题，JavaScript 中只有「传递引用」而没有「引用传递」。

「引用传递」实际上是 C++ 中存在的概念：`&a = b` 相当于给变量 `b` 起了一个别名 `a`，「引用」与「指针」最大的区别在于**一旦引用被初始化，就不能改变引用的关系**。

实际上，JavaScript 中所有函数的参数**都是按值来传递**的（有时也称 JS 的「传递引用」为「共享传递」—— [Call by sharing](https://en.wikipedia.org/wiki/Evaluation_strategy#Call_by_sharing)），但引用类型变量的值只是一个对原对象的引用，所以造成了「在同一个对象上」进行操作的效果（类似于 C++ 中的「指针传递」）。但是，这个引用类型变量的值是可以改变的，也就是可以指向其他的对象（而 C++ 的引用是不能改变引用关系的）。

看一个例子：
```js
function changeStuff(a, b, c)
{
  a = a * 10;
  b.item = "changed";
  c = {item: "changed"};
}

var num = 10;
var obj1 = {item: "unchanged"};
var obj2 = {item: "unchanged"};

changeStuff(num, obj1, obj2);

console.log(num);       // 10
console.log(obj1.item); // changed
console.log(obj2.item); // unchanged
```
由于函数内对 `obj1` 是对对象的内部进行操作，所以还是一直指向的是同一个对象，而 `obj2` 在函数中直接把 `c` 这个变量的值引用到了另一个完全不同的对象，所以对原对象（`obj2` 所引用到的对象）没有任何影响。但如果这里 `obj2` 是按照 C++ 的「引用传递」传递给函数内的 `c` 的话，则外部的 `obj2` 也会指向函数内 `c` 指向的对象，因为 `c` 完全就是 `obj2` 的一个别名。

**扩展**：
1. [Is JavaScript a pass-by-reference or pass-by-value language?](http://stackoverflow.com/questions/518000/is-javascript-a-pass-by-reference-or-pass-by-value-language)
2. [javascript传递参数如果是object的话，是按值传递还是按引用传递？](https://www.zhihu.com/question/27114726)

---

### Q：Promise.then() 的回调为什么一定是异步执行的 ？               
A：因为异步回调才能避免代码执行顺序上的 Race Condition。
考虑下面这一段代码：
```js
promise.then(function(){ 
  if (trueOrFalse) { 
    // 同步执行 
    foo(); 
  } else { 
    // 异步执行 (如：使用第三方库)
     setTimeout(function(){ 
        foo(); 
     }) 
  } 
}); 

bar();
```
如果 .then() 的回调是同步执行的，那上段代码中的 `foo()` 与 `bar()` 的执行顺序取决于 `trueOrFalse` 这个变量的值（`true` 则 `foo()` 先执行，`false` 则 `bar()` 先执行），而如果 .then() 的回调是异步执行的话，`bar()` 一定会先执行，这样可以保证代码执行顺序上的一致性。

**扩展**：
1. [Promise then中回调为什么是异步执行？](https://www.zhihu.com/question/57071244)
2. [专栏: Promise只能进行异步操作？](https://wohugb.gitbooks.io/promise/content/usage/async.html)

---
---

## Node.js

### Q：microtask 与 macrotask 有什么区别？                
A：在 Node.js 的一轮 Event-Loop 中，macrotask 只会执行现有的存在于 macrotask queue 中的任务，至于在此过程中新产生的 macrotask，会放在下一轮事件循环的 macrotask queue 中。而对于 microtask，在这一轮事件循环中新产生的 microtask，同样会被追加到这一轮 microtask queue 尾部，直到整个 microtask queue 被清空。

对于 Node.js 的几个异步函数，`setTimeout`、`setImmediate` 和 `setInterval` 产生的是 macrotask，`process.nextTick` 产生的是 microtask。

所以务必避免这样的代码：
```js
function cb() {
  process.nextTick(cb);
}
cb();
```
在 `process.nextTick` 的回掉中递归调用自己，这跟直接把线程阻塞无异。

另外，由于 V8 的实现中 Promise 的异步回调产生的是 microtask，所以目前在 Node 中 Promise.then() 的回调会比 setTimeout() 的回调先执行。

**扩展**：
1. [Difference between microtask and macrotask within an event loop context](http://stackoverflow.com/questions/25915634/difference-between-microtask-and-macrotask-within-an-event-loop-context)
2. [The Node.js Event Loop, Timers, and process.nextTick()](https://nodejs.org/en/docs/guides/event-loop-timers-and-nexttick/)

---
---

## NPM

### Q：[`npm shrinkwrap`](https://docs.npmjs.com/cli/shrinkwrap) 有什么优点和缺点？
A：优点不言自明，生成的 lockfile 可以保证每次安装到的依赖包都会有完全一致的版本和依赖树，所以可以很大程度的避免开发环境没问题但部署到正式环境就挂掉的问题。

缺点就是 lockfile 使得包管理工具对于依赖包的管理完全丧失了其[语义化版本](https://docs.npmjs.com/getting-started/semantic-versioning)的能力，同时也阉割了生成更加扁平的依赖树的机会（相同的其他条件下，扁平的依赖树可以使得整个依赖下载速度更快、占用更小空间），比如项目依赖 A 和 B，而 B 又依赖 A，如果一个版本的 A 可以解决问题那么依赖树中只需要那一个版本的 A 即可（语义化版本提供了这样的空间），而如果 lockfile 完全把版本号给锁死，那可以用一个版本的 A 来生成依赖树的可能就非常小了，也就是依赖树的生成不够灵活了。

所以 Node.js 社区的负责人之一 [ashley williams](https://medium.com/@ag_dubs) 的确说的没错：lockfiles 对应用好，对库不好。

另外 `npm shrinkwrap` 没 [yarn](https://yarnpkg.com/) 好的地方在于需要用户手动执行命令，这样很容易在引入了新的依赖包之后忘记执行导致 lockfile 与实际情况不符，而 yarn 对于 lockfile 都是自动生成和与实际情况同步的。

**扩展**
1. [An introduction to how JavaScript package managers work](https://medium.freecodecamp.com/javascript-package-managers-101-9afd926add0a)
2. [使用 npm shrinkwrap 来管理项目依赖](http://tech.meituan.com/npm-shrinkwrap.html)
3. [语义化版本 2.0.0](http://semver.org/lang/zh-CN/)

---
