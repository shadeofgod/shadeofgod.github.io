---
layout: post
title:  "Closure for dummies"
image: ''
date:   2016-10-15 22:22:31
tags:
- javascript
description: ''
categories:
- Javascript
serie: learn

---

originally posted on <a href="http://stackoverflow.com/questions/111102/how-do-javascript-closures-work">stackoverflow</a> by Morris. 

这是一篇在stackoverflow上关于Javascript闭包的解释，因为大部分中文或者英文官方资料都写的都非常晦涩难懂，在这里尽量用中文化的习惯翻译并解释，不同于英文的部分加入的是自己的理解，欢迎对错误的地方进行指正！希望本文让自己能有更深刻的理解，也可以帮到初学者。以下并附上英文部分的原文。

This page explains closures so that a programmer can understand them — using working JavaScript code. It is not for gurus or functional programmers.

以下通过Javascript代码为大家简单解释一下闭包的概念，大牛可以选择跳过。

Closures are *not hard* to understand once the core concept is grokked. However, they are impossible to understand by reading any academic papers or academically oriented information about them!

只要核心的概念理解了以后，闭包其实就很好懂了，但是在学术性质比较浓的专业文档中读起来却会感觉非常难以理解。

This article is intended for programmers with some programming experience in a mainstream language, and who can read the following JavaScript function:

此文主要是为一些有基础编程经验的，了解一门主流编程语言，并且可以看懂以下简单Javascript代码的程序员所写的：

```javascript
function sayHello(name) {
    var text = 'Hello ' + name;
    var say = function() { console.log(text); }
    say();
}
```

## An Example of a Closure 一个闭包的例子

Two one sentence summaries:

- a closure is one way of supporting [first-class functions](https://en.wikipedia.org/wiki/First-class_function); it is an expression that can reference variables within its scope (when it was first declared), assigned to a variable, passed as an argument to a function, or returned as a function result. Or
- a closure is a stack frame which is allocated when a function starts its execution, and *not freed* after the function returns (as if a 'stack frame' were allocated on the heap rather than the stack!).

两句话总结：

- 闭包是一种支持头等函数的方式；是一种可以在变量初次声明时的作用域内引用变量，赋值给另外一个变量，作为参数传递给一个函数或者作为一个函数的返回值的表达方式。
- 闭包是当一个函数开始执行时分配到堆栈中的一系列数据，并且在函数返回时并不会被释放。

（注：对以上的翻译并不满意，建议阅读英文资料，但看不懂也没有关系，可以继续看下面的例子。）

The following code returns a reference to a function:

下面的代码返回了一个函数引用值：

```javascript
function sayHello2(name) {
    var text = 'Hello ' + name; // Local variable
    var say = function() { console.log(text); }
    return say;
}
var say2 = sayHello2('Bob');
say2(); // logs "Hello Bob"
```

Most JavaScript programmers will understand how a reference to a function is returned to a variable (`say2`) in the above code. If you don't, then you need to before you can learn closures. A C programmer would think of the function as returning a pointer to a function, and that the variables`say` and `say2` were each a pointer to a function.

大部分JavaScript开发者都会明白上面的代码中函数返回给变量`say2`的是函数的引用值，即`say2`=`say`,而不是`say()`,没有带括号意味着函数并没有被调用，而C语言的开发者会理解为函数返回的是一个函数的指针（即指向的某一个函数，而非立刻调用某函数），而`say`和`say2`两者都是同一个函数的指针。

There is a critical difference between a C pointer to a function and a JavaScript reference to a function. In JavaScript, you can think of a function reference variable as having both a pointer to a function *as well* as a hidden pointer to a closure.

而C语言中的函数指针和JavaScript中的函数引用值有一个很关键的区别。你可以理解为在JS中，一个函数的引用变量既包含了一个函数的指针，也包含了一个指向闭包的隐藏指针。

The above code has a closure because the anonymous function `function() { console.log(text); }` is declared *inside* another function, `sayHello2()` in this example. In JavaScript, if you use the `function` keyword inside another function, you are creating a closure.

上面的代码中含有闭包是因为 `function() { console.log(text); }` 这个匿名函数在一个函数内部被声明，并且在外部被使用，在JavaScript如果你在一个函数内部使用了`function` 关键字来定义一个新的函数，你其实就是创建了一个闭包。

In C and most other common languages, *after* a function returns, all the local variables are no longer accessible because the stack-frame is destroyed.

在c语言和大部分主流语言中，在一个函数结束返回了值以后，所有的内部局部变量都再也无法访问，因为堆栈中的数据已经被释放摧毁了。

In JavaScript, if you declare a function within another function, then the local variables can remain accessible after returning from the function you called. This is demonstrated above, because we call the function `say2()` after we have returned from `sayHello2()`. Notice that the code that we call references the variable `text`, which was a *local variable* of the function `sayHello2()`.

在js中，如果你在一个函数内部定义了一个新的函数，那么内部的局部变量在通过你返回的这个新的函数被调用的时候还是可以被访问的。上面的例子里面，因为我们从`sayHello2()`返回了在它内部定义的函数给`say2`，这样在调用`say2()`的时候，实际上变量`text`作为 `sayHello2()`的内部的局部变量还是可以被我们访问到。

```javascript
function() { console.log(text); } // Output of say2.toString();
```

Looking at the output of `say2.toString()`, we can see that the code refers to the variable `text`. The anonymous function can reference `text` which holds the value `'Hello Bob'` because the local variables of `sayHello2()` are kept in a closure.

你可以看到把变量`say2`转换为字符串的结果，其实的代码就需要使用到变量`text`。这个匿名函数中的变量`text`其实就保存了`Hello Bob`这样一个值，因为这个函数 `sayHello2()`的内部的局部变量处于一个闭包之中。

The magic is that in JavaScript a function reference also has a secret reference to the closure it was created in — similar to how delegates are a method pointer plus a secret reference to an object.

神奇的地方在于，js中一个函数的引用值也包含了闭包被创建的时候的秘密引用值，就好像其他编程语言中的委托，是一个方法的指针，也是一个对象的秘密引用值。

## More examples 更多例子

For some reason, closures seem really hard to understand when you read about them, but when you see some examples you can click to how they work (it took me a while). I recommend working through the examples carefully until you understand how they work. If you start using closures without fully understanding how they work, you would soon create some very weird bugs!

闭包通常在你读这些解释文字的时候会感到非常难以理解，但是结合具体例子的话就好多了。强烈建议你详细阅读下面的例子并且搞清楚他们到底是如何工作的。如果你没有彻底搞清楚闭包的机制，在使用的时候非常容易造成一些奇怪的bug！

### Example 3

This example shows that the local variables are not copied — they are kept by reference. It is kind of like keeping a stack-frame in memory when the outer function exits!

下面的例子展示了，局部变量并不是被复制的，而是直接引用的，就好像在函数执行完成后数据仍然被保存在堆栈中。

```javascript
function say667() {
    // Local variable that ends up within closure
    var num = 42;
    var say = function() { console.log(num); }
    num++;
    return say;
}
var sayNumber = say667();
sayNumber(); // logs 43
```

### Example 4

All three global functions have a common reference to the *same* closure because they are all declared within a single call to `setupSomeGlobals()`.

所有的三个全局函数都有一个指向同一个闭包的相同引用值，因为他们都是通过 `setupSomeGlobals()`这同一个函数被声明的。

```javascript
var gLogNumber, gIncreaseNumber, gSetNumber;
function setupSomeGlobals() {
    // Local variable that ends up within closure
    var num = 42;
    // Store some references to functions as global variables
    gLogNumber = function() { console.log(num); }
    gIncreaseNumber = function() { num++; }
    gSetNumber = function(x) { num = x; }
}

setupSomeGlobals();
gIncreaseNumber();
gLogNumber(); // 43
gSetNumber(5);
gLogNumber(); // 5

var oldLog = gLogNumber;

setupSomeGlobals();
gLogNumber(); // 42

oldLog() // 5
```

The three functions have shared access to the same closure — the local variables of `setupSomeGlobals()` when the three functions were defined.

当这三个函数被定义时都共享了同一个闭包 - 即函数 `setupSomeGlobals()` 内部的局部变量。

Note that in the above example, if you call `setupSomeGlobals()` again, then a new closure (stack-frame!) is created. The old `gLogNumber`, `gIncreaseNumber`, `gSetNumber` variables are overwritten with *new* functions that have the new closure. (In JavaScript, whenever you declare a function inside another function, the inside function(s) is/are recreated again *each* time the outside function is called.)

注意到上面的例子里，如果你再次调用 `setupSomeGlobals()`，一个新的闭包又形成了，旧的`gLogNumber`, `gIncreaseNumber`, `gSetNumber` 这些变量都被新的值覆盖了。（在js中，当你在一个函数内部声明了另外一个函数，那么当外部这个函数被调用的时候，内部的函数总会被重新创建。）

### Example 5

This one is a real gotcha for many people, so you need to understand it. Be very careful if you are defining a function within a loop: the local variables from the closure do not act as you might first think.

这一个例子让很多人茅塞顿开，所以你一定要理解它！要注意当你在一个循环中定义一个函数的时候，闭包里的局部变量并不会像你想的那样运作。

```javascript
function buildList(list) {
    var result = [];
    for (var i = 0; i < list.length; i++) {
        var item = 'item' + i;
        result.push( function() {console.log(item + ' ' + list[i])} );
    }
    return result;
}

function testList() {
    var fnlist = buildList([1,2,3]);
    // Using j only to help prevent confusion -- could use i.
    for (var j = 0; j < fnlist.length; j++) {
        fnlist[j]();
    }
}

 testList() //logs "item2 undefined" 3 times
```

The line `result.push( function() {console.log(item + ' ' + list[i])}` adds a reference to an anonymous function three times to the result array. If you are not so familiar with anonymous functions think of it like:

 `result.push( function() {console.log(item + ' ' + list[i])}` 这一行把里面这个匿名函数的引用值三次添加到了结果的数组里。如果你对匿名函数不是很熟悉，可以这么想：

```javascript
pointer = function() {console.log(item + ' ' + list[i])};
result.push(pointer);
```

Note that when you run the example, `"item2 undefined"` is alerted three times! This is because just like previous examples, there is only one closure for the local variables for `buildList`. When the anonymous functions are called on the line `fnlist[j]()`; they all use the same single closure, and they use the current value for `i` and `item` within that one closure (where `i` has a value of `3` because the loop had completed, and `item` has a value of `'item2'`). Note we are indexing from 0 hence `item` has a value of `item2`. And the i++ will increment `i` to the value `3`.

注意当你运行这个例子的时候，出来的结果是打印三次 `"item2 undefined"` 。这是因为同之前的例子一样，对于 `buildList`中的局部变量，他们都处于唯一的闭包中。当匿名函数在 `fnlist[j]()`这里被调用的时候，他们都是用同一个唯一的闭包，使用同一个目前的`i`和`item`的值，而循环已经结束，此时的`i`的值为3，而`item`的值为`item2`。注意，`i`从0开始计算，循环结束的时候通过`i++`变成了3，而没有满足循环条件再次进入循环，因此`item`的值还是为`item2`。

### Example 6

This example shows that the closure contains any local variables that were declared inside the outer function before it exited. Note that the variable `alice` is actually declared after the anonymous function. The anonymous function is declared first; and when that function is called it can access the `alice` variable because `alice` is in the same scope (JavaScript does [variable hoisting](http://stackoverflow.com/a/3725763/1269037)). Also `sayAlice()()` just directly calls the function reference returned from `sayAlice()`— it is exactly the same as what was done previously, but without the temporary variable.

这个例子展示了闭包包含了在外部函数结束前声明的所有局部变量。注意，变量`alice`实际上是在匿名函数之后声明的，匿名函数首先被定义，当这个函数被调用时它依然可以访问到`alice`这个变量，因为`alice`在同样的作用域内。同样的，`sayAlice()()`直接通过`sayAlice()`返回的函数引用值调用了指向的函数，而省略了我们之前先把函数引用值保存在一个临时的变量这一步骤。

```javascript
function sayAlice() {
    var say = function() { console.log(alice); }
    // Local variable that ends up within closure
    var alice = 'Hello Alice';
    return say;
}
sayAlice()();// logs "Hello Alice"
```

Tricky: note also that the `say` variable is also inside the closure, and could be accessed by any other function that might be declared within `sayAlice()`, or it could be accessed recursively within the inside function.

注意，变量`say`同样也在闭包之中，并且可以被其他可能在`sayAlice()`内部的函数所访问，意味着可以在内部通过递归函数来访问。

### Example 7

This final example shows that each call creates a separate closure for the local variables. There is *not* a single closure per function declaration. There is a closure for *each call* to a function.

最后一个例子展示了每次调用都为局部变量创造了一个分开的闭包。并不是每次函数声明都有一个单独的闭包，而是函数调用的时候！

```javascript
function newClosure(someNum, someRef) {
    // Local variables that end up within closure
    var num = someNum;
    var anArray = [1,2,3];
    var ref = someRef;
    return function(x) {
        num += x;
        anArray.push(num);
        console.log('num: ' + num +
            '\nanArray ' + anArray.toString() +
            '\nref.someVar ' + ref.someVar);
      }
}
obj = {someVar: 4};
fn1 = newClosure(4, obj);
fn2 = newClosure(5, obj);
fn1(1); // num: 5; anArray: 1,2,3,5; ref.someVar: 4;
fn2(1); // num: 6; anArray: 1,2,3,6; ref.someVar: 4;
obj.someVar++;
fn1(2); // num: 7; anArray: 1,2,3,5,7; ref.someVar: 5;
fn2(2); // num: 8; anArray: 1,2,3,6,8; ref.someVar: 5;
```

## Summary 总结

If everything seems completely unclear then the best thing to do is to play with the examples. Reading an explanation is much harder than understanding examples. My explanations of closures and stack-frames, etc. are not technically correct — they are gross simplifications intended to help understanding. Once the basic idea is grokked, you can pick up the details later.

如果完全不懂的话最好从具体的例子入手，只是阅读一些解释的文字通常很难理解。我的关于闭包和堆栈的解释实际上并非绝对准确，他们只是为了简单的帮助你理解，在最基础的概念弄懂以后，你可以多查阅学习一些具体的细节。

## Final points: 最后几点：

- Whenever you use `function` inside another function, a closure is used.

  当你在一个函数内部定义一个函数的时候，你就使用了闭包。

- Whenever you use `eval()` inside a function, a closure is used. The text you `eval` can reference local variables of the function, and within `eval` you can even create new local variables by using `eval('var foo = …')`

  不管你什么时候在函数内部使用`eval()`函数，你就使用了闭包。你可以`eval`来引用函数的局部变量，也可以通过在`eval`的参数内来创建新的局部变量。

- When you use `new Function(…)` (the [Function constructor](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Function)) inside a function, it does not create a closure. (The new function cannot reference the local variables of the outer function.)

  当你在函数内部使用构造函数的时候，并不会创建闭包。

- A closure in JavaScript is like keeping a copy of all the local variables, just as they were when a function exited.

  js中的闭包就好像在函数执行完毕后依然保留着所有局部变量。

- It is probably best to think that a closure is always created just on entry to a function, and the local variables are added to that closure.

  最好总是在函数刚进入的时候就考虑闭包的建立并且把所有局部变量加入到闭包中。

- A new set of local variables is kept every time a function with a closure is called (given that the function contains a function declaration inside it, and a reference to that inside function is either returned or an external reference is kept for it in some way).

  在每次函数被调用的时候，闭包中的局部变量的值都会重新调整。（前提是这个函数内部包含了函数声明，并且这个内部函数的引用值作为返回值或者保存到了外部的引用。）

- Two functions might look like they have the same source text, but have completely different behaviour because of their 'hidden' closure. I don't think JavaScript code can actually find out if a function reference has a closure or not.

  两个函数可能看上去他们有同样的来源，但是由于隐藏的闭包所以有着完全不同的行为。

- If you are trying to do any dynamic source code modifications (for example: `myFunction = Function(myFunction.toString().replace(/Hello/,'Hola'));`), it won't work if `myFunction` is a closure (of course, you would never even think of doing source code string substitution at runtime, but...).

- It is possible to get function declarations within function declarations within functions — and you can get closures at more than one level.

  在函数内部的函数声明中再次声明新的函数也是可能的，而且你可以超过一个层级的闭包。

- I think normally a closure is the term for both the function along with the variables that are captured. Note that I do not use that definition in this article!

- I suspect that closures in JavaScript differ from those normally found in functional languages.

## Links

另附阮一峰老师关于闭包的解释，写的非常好！<a href="http://www.ruanyifeng.com/blog/2009/08/learning_javascript_closures.html">学习Javascript闭包（Closure）</a>。
