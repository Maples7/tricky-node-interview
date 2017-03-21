# tricky-node-interview
[Chinese Repo] A bunch of tricky questions I've met while preparing job interview of Node.js back-end developer.

## 前言
这个 Repo 记录我在准备 Node.js Web 后端开发面试过程中遇到的一些记得一提的问题。     
问题会按照考查的知识模块进行分类且每个分类基本按照难度递增和关联度进行排序。       
一些问题的完整答案可能需要扎实的编程基础才能理解，我会列出可以进一步参考的资料。    
最后，我也无法保证每一个回答都能说明得正确、清晰完整且切中要害，如有任何问题和争议，欢迎提 issue 或 PR 指出。       

## JavaScript 语言

### Q：JavaScript 中基本类型（原始类型）有哪些？               
A：6 种基本类型：Boolean、Null、Undefined、Number、String、Symbol（ES6）。其他全部都是 Object（引用类型）。值得一提的是 `typeof` 的一些特殊情形：

1. `typeof Null === 'object'`；
2. `typeof [function] === 'function'`，但实际上 JS 没有 Function 这种类型；

这题一般都是引出后续的其他题。

---

### Q：`10 > undefined` 是 `true` 还是 `false`？         
A：`false`。
这一题则是明显的不等比较时的类型转换规则，如下：

1. 如果两个操作值都是数值，则进行数值比较
2. 如果两个操作值都是字符串，则比较字符串对应的字符编码值
3. 如果只有一个操作值是数值，则将另一个操作值转换为数值，进行数值比较
4. 如果一个操作数是对象，则调用 `valueOf()` 方法（如果对象没有 `valueOf()` 方法则调用 `toString()` 方法），得到的结果按照前面的规则执行比较
5. 如果一个操作值是布尔值，则将其转换为数值，再进行比较
7. `NaN` 是非常特殊的值，它不和任何类型的值相等，包括它自己，同时它与任何类型的值比较大小时都返回`false`

首先应用规则 3，`Number(undefined)` 为 `NaN`，然后应用规则 7。

---

### Q：`[1] == [1]` 是 `true` 还是 `false`？    
A：`false`。    
这里涉及到两个点：`==` 与 `===` 的区别 以及 JavaScript 的类型系统。     
`==` 与 `===` 的区别在于 `===` 在执行比较运算前不会发生隐式的类型转换，而 `==` 运算发生的隐式类型转换遵循以下规则：

1. 如果一个操作值为布尔值，则在比较之前先将其转换为数值
2. 如果一个操作值为字符串，另一个操作值为数值，则通过 `Number()` 函数将字符串转换为数值
3. 如果一个操作值是对象，另一个不是，则调用对象的 `valueOf()` 方法，得到的结果按照前面的规则进行比较
4. `null` 与 `undefined` 是相等的
5. 如果一个操作值为 `NaN`，则相等比较返回 `false`
6. 如果两个操作值都是对象，则比较它们是不是指向同一个对象

可见，这个转换规则还是比较复杂的，这也是为什么编程实践中一律推荐使用 `===` 的原因。`[1] == [1]` 显然应用到了规则 6。这题 tricky 的地方在于容易被 `==` 的类型转换所误导。

**扩展**：
1. [JS Comparison Table](https://dorey.github.io/JavaScript-Equality-Table/)
2. [ECMA-262 11.8 Relational Operators](http://www.ecma-international.org/ecma-262/5.1/#sec-11.8)

---

### Q: JavaScript 中对象的引用传递是怎样的？               
A: 首先题干有问题，JavaScript 中只有「传递引用」而没有「引用传递」。

「引用传递」实际上是 C++ 中存在的概念：`&a = b` 相当于给变量 `b` 起了一个别名 `a`，「引用」与 「指针」最大的区别在于**一旦引用被初始化，就不能改变引用的关系**。

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


