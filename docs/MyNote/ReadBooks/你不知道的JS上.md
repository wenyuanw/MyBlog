#  你不知道的JavaScript 上——作用域和闭包

## 作用域是什么

程序编译的三个过程：

1. 分词/语法分析：这个过程会将由字符组成的字符串分解成（对编程语言来说）有意义的代码块，这些代码块被称为词法单元（token）。
2. 解析/语法分析：这个过程是将词法单元流（数组）转换成一个由元素逐级嵌套所组成的代表了程序语法结构的树。这个树被称为“抽象语法树”（Abstract Syntax Tree，AST）。
3. 代码生成：将 AST 转换为可执行代码的过程被称为代码生成。



作用域是一套规则，用于确定在何处以及如何查找变量（标识符）。

如果有查找的目的是对变量进行复制，那么就会使用 LHS 查询；

如果目的是获取变量的值，就会使用 RHS 查询。



LHS 和 RHS 查询都会在当前执行作用域中开始，如果有需要（也就是说它们没有找到所需的标识符），就会向上级作用域继续查找目标标识符，这样每次上升一级作用域（一层楼），最后抵达全局作用域（顶层），无论找到或没找到都将停止。



不成功的 RHS 引用会导致抛出 **ReferenceError** 异常。不成功的 LHS 引用会导致自动隐式地创建一个全局变量（非严格模式下），该变量使用 LHS 引用地目标作为标识符，或者抛出 ReferenceError 异常（严格模式下）。



## 词法作用域

JavaScript 采用的作用域模型是**“词法作用域”**模型。与之相对的是“动态作用域”。

简单地说，词法作用域就是定义在词法阶段的作用域。



作用域查找会在找到第一个匹配的标识符时停止。

在多层的嵌套作用域中可以定义同名的标识符，这叫作“遮蔽效应”（内部的标识符"遮蔽"了外部的标识符）。



全局对象会自动成为全局对象（比如浏览器中的 window 对象的属性，因此可以不直接通过全局对象的词法名称，而是间接地通过对全局对象属性的引用来对其进行访问。

如：window.a

通过这种技术可以访问那些被同盟变量所遮蔽的全局变量。但非全局的变量如果被遮蔽了，无论如何都无法被访问到。



欺骗词法：

eval(...) 函数可以接受一个字符串为参数，并将其中的内容视为好像在书写时就存在于程序中这个位置的代码。

```js
function foo(str, a) {
	eval(str); // 欺骗
	console.log(a, b);
}
var b = 2;
foo("var b = 3;", 1); // 1,3
```

在程序中动态生成代码的使用场景非常罕见，因为它所带来的好处无法抵消性能上的损失。



with 关键字通常被当作重复引用同一个对象中的多个属性的快捷方式，可以不需要重复引用对象本身。

with 可以将一个没有或有多个属性的对象处理为一个完全隔离的词法作用域，因此这个对象的属性也会被处理为定义在这个作用域中的词法标识符。



eval(...) 函数如果接受了含有一个或多个声明的代码，就会修改其所处的词法作用域，而 with 声明实际上是根据你传递给它的对象凭空创建了一个全新的词法作用域。



eval(...) 和 with 会在运行时修改或创建新的作用域，以此来欺骗其他在书写时定义的词法作用域。



## 函数作用域和块作用域

**函数作用域**的含义是指，属于这个函数的全部变量都可以在整个函数的范围内使用及复用（事实上嵌套的作用域中也可以使用）。



函数声明和函数表达式之间最重要的区别是它们的名称标识符将会绑定在何处。

函数是 JavaScript 中最常见的作用域单元。

单函数不是唯一的作用域单元。块作用域指的是变量喝函数不仅可以属于所处的作用域，也可以属于某个代码块（通常指 { .. } 内部）。

从 ES3 开始，try/catch 结构在 catch 分句中具有块作用域。



## 提升

包括变量和函数在内的所有声明都会在任何代码被执行前首先被处理。



当你看到 var a = 2；时，可能会认为这是一个声明。但 JS 实际上会将其看成两个声明：var a; 和 a = 2;。第一个定义声明是在**编译阶段**进行的。第二个赋值声明会被**留在原地**等待**执行阶段**。

第一个是编译阶段的任务，第二个则是执行阶段的任务。



**只有声明本身会被提升，而赋值或其他运行逻辑会留在原地。**

每个作用域都会进行提升操作。

```js
foo();

function foo() {
	console.log(a); // undefine
	var a = 2;
}
// foo 函数的声明 被提升了，因此第一行的调用可以正常执行
// 这段代码实际上被理解为一下形式：
function foo() {
	var a;
    console.log(a);
    a = 2;
}
foo();
```

**函数声明会被提升，但是函数表达式却不会被提升。**

```js
foo(); // 不是 ReferenceError，而是 TypeError！

var foo = function bar() {
	// ...
}
// 这段程序中的表里标识符 foo() 被提升并分配给所在作用域（这里是全局作用域），因此 foo() 不会导致 
// ReferenceError。但是 foo 此时并没有赋值（如果它是一个函数声明而不是函数表达式，那么就会赋值）
// foo() 由于对 undefined 值进行函数调用而导致非法操作，因此抛出 TypeError 异常。
```

即使是具名的函数表达式，名称标识符在赋值之前也无法在所在作用域中使用：

```js
foo(); // TypeError
bar(); // ReferenceErrors

var foo = function bar() {
	// ...
}

这个代码片段经过提升后，实际被理解为以下形式：
var foo;

foo(); // TypeError
bar(); // ReferenceErrors

var foo = function() {
	var bar = ...self...
    // ...
}
```

### 函数优先

函数声明和变量声明都会被提升。但是一个值得注意的细节（这个细节可以出现在有多个“重复”声明的代码中）是**函数会首先被提升，然后才是变量。**

```js
// 考虑以下代码：
foo(); // 1

var foo;

function foo() {
	console.log(1);
}

foo = function {
	console.log(2);
}
// 会输出 1 而不是 2 ！这个代码片段会被引擎理解为如下形式：
function foo() {
	console.log(1);
}

foo(); // 1

foo = function {
	console.log(2);
}
// 注意，var foo 尽管出现在 function foo() ... 的声明之前，但它是重复的声明（因此被忽略了），因为函数声明会被提升到普通变量之前。
// 尽管重复的 var 声明会被忽略掉，但出现在后面的函数声明还是可以覆盖前面的。

foo(); // 3

foo = function {
	console.log(1);
}

var foo = function {
	console.log(2);
}

foo = function {
	console.log(3);
}
```

一个普通块内部的函数声明通常会被提升到所在作用域的顶部，这个过程不会像下面的代码暗示的那样可以被条件判断所控制：

```js
foo(); // b
var a = true;
if(a){
	function foo() { console.log("a") ;}
} else {
	function foo() { console.log("b") ;}
}
```

因此应该尽可能避免在块内部声明函数。



**声明本身会被提升，而包括函数表达式的赋值在内的赋值操作并不会提升。**



## 作用域闭包

> 闭包是基于词法作用域书写代码时所产生的自然结果，你甚至不需要为了利用它们而有意识地创建闭包。闭包地创建和使用在你的代码中随处可见。你缺少地是根据你自己的意愿来识别、拥抱和影响闭包的思维环境。



**定义：当函数可以记住并访问所在的词法作用域时，就产生了闭包，即使函数是在当前词法作用域之外执行。**



下面我们来看一段代码，清晰地展示了闭包：

```js
function foo() {
	var a = 2;
	
	function bar() {
		console.log( a );
	}
	
	return bar;
}

var baz = foo();
baz(); // 2 —— 这就是闭包的效果
```

函数 bar( ) 的词法作用域能够访问 foo( ) 的内部作用域。然后我们将 bar( ) 函数本身当作一个值类型进行传递。在这个例子中，我们**将 bar 所引用的函数对象本身当作返回值。** 

在 foo( ) 执行后，其返回值（也就是内部的 bar( ) 函数）赋值给变量 baz 并调用 baz( ) ，实际上知识通过不同的标识符引用调用了内部的函数 bar( )。

bar( ) 显然可以被正常执行。但是在这个例子中，它在自己定义的词法作用域以外的地方执行。

在 foo( ) 执行后，通常会期待 foo( ) 的整个内部作用域都被销毁，因为我们知道引擎有垃圾回收器用来释放不在使用的内存空间。由于看上去 foo( ) 的内容不会再被使用，所以很自然地会考虑对其进行回收。

而闭包的“神奇”之处正式可以阻止这件事情地发送。事实上内部作用域依然存在，因此没有被回收。

因为 **bar() 本身在使用这个作用域。**

由于 bar( ) 的声明位置，它拥有涵盖 foo( ) 内部作用域的闭包，使得该作用域能够一直存活，以供 bar( ) 在之后任何时间进行调用。

**bar( ) 依然持有对该作用域的引用，而这个引用就叫做闭包。**

因此，在几微秒之后变量 baz 被实际调用（调用内部函数 bar ），不出意料它可以访问定义域时的词法作用域，因此它也可以如预期般访问变量 a 。

这个函数在定义时地词法作用域以外地地方被调用。**闭包使得函数可以继续访问定义时的词法作用域。**



当然，**无论使用何种方式对函数类型的值进行传递，当函数在别处被调用时都可以观察到闭包。**

```js
function foo() {
	var a = 2;
	
	function baz() {
		console.log( a ); // 2
	}
	
	bar(baz);
}

function bar(fn) {
	fn(); // 这就是闭包！
}
```

把内部函数 baz 传递给 bar ，当调用这个内部函数时（现在叫做 fn ），它涵盖的 foo( ) 内部作用域的闭包就可以观察到了，因为它能够访问 a。



传递函数也可以是间接的：

```js
var fn;

function foo() {
	var a = 2;
    function baz() {
		console.log( a );
    }
    fn = baz; // 将 baz 分配给全局变量
}

function bar() {
	fn(); // 这就是闭包！
}

foo();
bar(); // 2
```

无论通过任何手段将内部函数传递到所在的词法作用域以外，它都会持有对原始定义作用域的引用，无论在何处执行这个函数都会使用闭包。



一个常见的闭包：

```js
function wait(message) {
	setTimeout( function timer() {
		console.log(message);
    }, 1000);
}
wait("Hello, closure!");
```

将一个内部函数（名为 timer ）传递给 setTimeout(..)。timer 具有涵盖 wait(..) 作用域的闭包，因此还保有对变量 message 的引用。



本质上无论何时何地，如果将函数（访问它们各自的词法作用域）当作第一级的值类型并到处传递，你就会看到闭包在这些函数中的应用。在定时器、事件监听器、Ajax 请求、跨窗口通信、Web Worker 或者任何其他的异步（或者同步）任务中，只要使用了回调函数，实际上就是在使用闭包。



### 循环和闭包

要说明闭包，for 循环时最常见的例子。

```js
for (var i = 1; i <= 5; i++){
	setTimeout( function timer() {
		console.log( i );
	}, i*1000);
}
```

正常情况下，我们对这段代码行为的预期是分别输出数字 1~5，每秒一次，每次一个。

但实际上，这段代码在运行时会以每秒一次的频率输出五次 6。

**延迟函数的回调会在循环结束时才执行。**

事实上，当定时器运行时即使每个迭代中执行的是 setTimeout(.., 0)，所有的回调函数依然实在循环结束后才会被执行，因此会每次输出一个6出来。

> 上述代码的缺陷是我们试图假设循环中的每个迭代在运行时都会给自己“捕获”一个 i 的副本。但是根据作用域的工作原理，实际情况是尽管循环中的五个函数是在各个迭代中分别定义的，但是它们都被封闭在一个共享的全局作用域中，因此实际上只有一个 i。



IIFE 会通过声明并立即执行一个函数来创建作用域。所以以下代码能解决问题吗？

```js
for (var i = 1; i <= 5; i++){
	(function() {
        setTimeout(function timer() {
			console.log(i);
        }, i*1000);
    })();
}
```

答案是不行的。

的确每个延迟函数都会将 IIFE 在每次迭代中创建的作用域封闭起来。

如果作用域是空的，那么仅仅将它们进行封闭式不够的。仔细看一下，我们的 IIFE 知识一个什么都没有的空作用域。它需要包含一点实质内容才能为我们所用。

它需要有自己的变量，用来在每个迭代中储存 i 的值。



解决代码：

```js
for (var i = 1; i <= 5; i++){
	(function(j) {
        setTimeout(function timer() {
			console.log(j);
        }, j*1000);
    })(i);
}
```

**在迭代内使用 IIFE 会为每个迭代都生成一个新的作用域，使得延迟函数的回调可以将新的作用域封闭在每个迭代内部，每个迭代中都会含有一个具有正确值的变量供我们访问。**



我们使用 IIFE 在每次迭代时都创建一个新的作用域。换句话说，每次迭代我们都需要一个块作用域。

**let** 声明，可以用来劫持块作用域，并且在这个块作用域中声明一个变量。

本质上这是将一个块转换成一个可以被关闭的作用域。因此下面这段代码可以正常运行。

```js
for (var i=1; i<=5; i++) {
	let j=i; // 闭包的块作用域
    setTimeout(function timer() {
			console.log(j);
    }, j*1000);
}
```

for 循环头部的 let 声明还会有一个特殊的行为。这个行为指出变量在循环过程中不止被声明一次，每次迭代都会声明。随后的每个迭代都会使用上一个迭代结束时的值来初始化这个变量。

```js
for (let i=1; i<=5; i++) {
    setTimeout(function timer() {
			console.log(i);
    }, i*1000);
}
```

这就是**块作用域+闭包**的效果。



**当函数可以记住并访问所在的词法作用域，即使函数是在当前词法作用域之外执行，这时就产生了闭包。**



## 关于this

> this 关键字是 JavaScript 中最复杂的机制之一。它是一个很特别的关键字，被自动定义再所有函数的作用域中。但是即使是非常有经验的 JavaScript 开发者也很难说清它到底指向什么。

为什么要用 this ？

**this 提供了一种更优雅的方式来隐式“传递”一个对象引用，因此可以将 API 设计得更加简洁并且易于复用。**

this 是在运行时进行绑定得，并不是在编写时绑定，它的上下文取决于函数调用时的各种条件。this 的绑定和函数声明的位置没有任何关系，只取决于函数的调用方式。

当一个函数被调用时，会创建一个活动记录（有时候也称为执行上下文）。这个记录会包含函数在哪里被调用（调用栈）、函数的调用方法、传入的参数等信息。this 就是记录的其中一个属性，会在函数执行的过程中用到。

this 既不指向函数自身，也不指向函数的词法作用域。

**this 实际上是在函数被调用时发生的绑定，它指向什么完全取决于函数在哪里被调用。**



### this 的绑定规则：

1. **默认绑定**

2. **隐式绑定**

   一个最常见的 this 绑定问题就是被隐式绑定的函数会丢失绑定对象，也就是说它会应用默认绑定，从而把 this 绑定到全局对象或者 undefined 上，取决于是否是严格模式。

   **回调函数丢失 this 绑定是非常常见的。**

3. **显式绑定** call apply

   显式绑定也无法解决丢失绑定的问题。

   **硬绑定**：显式绑定的一个变种，可以解决丢失绑定的问题。

   ```js
   function foo() {
   	console.log( this.a );
   }
   var obj = {
   	a:2
   };
   var bar = function() {
   	foo.call(obj);
   };
   bar(); // 2
   // 硬绑定的 bar 不可能再修改它的 this
   bar.call(window); // 2
   /*
   我们创建了函数 bar(),并在它的内部手动调用了 foo.call(obj),因此强制把 foo 的 this 绑定到了 obj。无论之后如何调用函数 bar，它总会手动在 obj 上调用 foo。这种绑定是一种显示的强制绑定，因此我们称之为硬绑定。
   */
   ```

   硬绑定的典型应用场景就是创建一个包裹函数，传入所有的参数并返回接收到的所有值。

   ```js
   function foo(something) {
   	console.log( this.a, something);
       return this.a + something;
   }
   var obj = {
   	a: 2
   };
   var bar = function() {
   	return foo.apply(obj, arguments);
   };
   var b = bar( 3 );  // 2 3
   console.log( b );  // 5
   ```

   另一种使用方法是创建一个可以重复使用的辅助函数**（apply 实现  bind）**：

   ```js
   function foo(something) {
   	console.log( this.a, something);
       return this.a + something;
   }
   // 简单的辅助绑定函数
   function bind(fn, obj) {
   	return function() {
   		return fn.apply(obj, arguments);
       };
   }
   var obj = {
   	a: 2
   };
   var bar = bind(foo, obj);
   var b = bar( 3 );  // 2 3
   console.log( b );  // 5
   ```

   由于硬绑定时一种非常常用的模式，所以在 ES5 中提供了内置的方法 Function.prototype.bind，它的用法如下：

   ```js
   function foo(something) {
   	console.log( this.a, something);
       return this.a + something;
   }
   var obj = {
   	a: 2
   };
   var bar = foo.bind(obj);
   var b = bar( 3 );  // 2 3
   console.log( b );  // 5
   ```

   

4. **new 绑定**

   在 JavaScript 中，构造函数只是一些使用 new 操作符时被调用的函数。它们并不会属于某个类，也不会实例化一个类。实际上，它们甚至都不能说是一种特殊的函数类型，它们只是被 new 操作符调用的普通函数而已。

   包括内置对象函数（比如 Number(..)）在内的所有函数都可以i用 new 来调用，这种函数调用被称为构造函数调用。这里有一个重要但是非常细微的区别：实际上并不存在所谓的“构造函数”，只有对于函数的“构造调用”。

   **使用 new 来调用函数，或者说发生构造函数调用时，会自动执行下面的操作**：

   1. 创建（或者说构造）一个全新的对象。
   2. 这个新对象会被执行 [[ 原型 ]] 连接。
   3. 这个新函数会绑定到函数调用的 this。
   4. 如果函数没有返回其他对象，那么 new 表达式中的函数调用会自动返回这个新对象。

   ```js
   function foo(a) {
   	this.a = a;
   }
   var bar = new foo(2);
   console.log(bar.a); // 2
   ```

   使用 new 来调用 foo(..) 时，我们会构造一个新对象并把它绑定到 foo(..) 调用中的 this 上。



### 判断 this的顺序：

1. 函数是否在 new 中调用（new 绑定）？如果是的话 this 绑定的是新创建的对象。
2. 函数是否通过call、apply （显式绑定）或者硬绑定调用？如果是的话，this 绑定的是指定的对象。
3. 函数是否在某个上下文对象中调用（隐式绑定）？如果是的话，this 绑定的是那个上下文对象。
4. 如果都不是的话，使用默认绑定。如果在严格模式下，就绑定到 undefined，否则绑定到全局对象。

> 不过也会有例外。



### 绑定例外

如果你把 null 或者 undefined 作为 this 的绑定对象传入 call、apply 或者 bind，这些值在调用时会被忽略，实际应用的是默认绑定规则。

#### 被忽略的 this

一种常见的作法是使用 apply(..) 来“展开”一个数组，并当作参数传入一个函数。类似地，bind(..) 可以对参数进行柯里化（预先设置一些参数）

```js
function foo(a,b) {
	console.log("a:" + a + ",b:" + b);
}

// 把数组“展开”成参数
foo.apply(null, [2,3]); // a:2, b:3

// 使用 bind(..) 进行柯里化
var bar = foo.bind(null, 2);
bar(3): // a:2, b:3
```

这两种方法都需要传入一个参数当作 this 的绑定对象，所以用 null 可能是一个不错的选择。

> 在 ES6 中，可以用 **... 操作符**代替 apply(..) 来“展开”数组，**foo(...[1, 2])** 和 **foo(1, 2)** 是一样的，这样可以避免不必要的 this 绑定。
>
> 在 ES6 中没有柯里化的相关语法，因此还需要使用 bind(..)。

然而使用 null 来忽略 this 绑定可能产生一些副作用。如果某个函数确实使用了 this （比如第三方库中的一个函数），那默认绑定规则会把 this 绑定到全局对象（在浏览器中这个对象是 window），中而将导致不可预计的后果（比如修改全局对象）。

显而易见，这种方式可能会导致许多难以分析和追踪的 bug。

#### 更安全的this

一种“更安全”的做法是传入一个特殊的对象，把 this 绑定到这个对象不会对你的程序产生任何副作用。就像网络（以及军队）一样，我们可以创建一个“DMZ”（demilitarized zone，非军事区）对象——**它就是一个空的非委托的对象**。

在 JavaScript 中创建一个空对象最简单的方法都是 Object.create(null)。Object.create(null) 和 {} 很小，但是并不会创建 Object.prototype 这个委托，所以它比 {} “更空”。

#### 间接引用

另一个需要注意的是，你有可能（有意或者无意地）创建一个函数的“间接引用”，在这种情况下，调用这个函数会应用默认绑定规则。

间接引用最容易在赋值时发生：

```js
function foo() {
	console.log(this.a);
}
var a = 2;
var o = { a: 3, foo: foo };
var p = { a: 4 };
o.foo(); // 3
(p.foo = o.foo)(); // 2
```

**赋值表达式 p.foo = o.foo 的返回值是目标函数的引用，因此调用位置是 foo() 而不是 p.foo() 或者 o.foo()。**根据之前说过的，这里会应用默认绑定。

#### 软绑定

可以给默认绑定指定一个全局对象和 undefined 以外的值，可以实现和硬绑定相同的效果，同时保留隐式绑定或者显示绑定修改 this 的能力。



### this 词法——箭头函数

**箭头函数**不使用 this 的四种标准规则，而是根据外层（函数或者全局）作用域来决定 this。

让我们来看看箭头函数的词法作用域：

```js
function foo() {
	// 返回一个箭头函数
    return (a) => {
		// this 继承自 foo()
        console.log(this.a);
    }
}
var obj1 = {
	a: 2
};
var obj2 = {
	a: 3
};
var bar = foo.call(obj1)；
bar.call( obj2 ); // 2,不是3！
```

foo() 内部创建的箭头函数会捕获调用时 foo() 的 this。由于 foo() 的 this 绑定到 obj1，bar（引用箭头函数）的 this 也会绑定到 obj1，**箭头函数的绑定无法被修改。（ new 也不行！）**

箭头函数可以像 bind(..) 一样确保函数的 this 被绑定到指定对象，此外，其重要性还体现在它用更常见的语法作用域取代了传统的 this 机制。



### 小结：

如果要判断一个运行宏函数的 this 绑定，就需要找到这个函数的**直接调用位置**。找到之后就可以顺序应用下面的斯特规则来判断 this 的绑定对象。

1. 由 new 调用？绑定到新创建的对象。
2. 由 call 或者 apply （或者 bind）调用？绑定到指定的对象。
3. 由上下文对象调用？绑定到那个上下文对象。
4. 默认：在严格模式下绑定到 undefined ，否则绑定到全局对象。

一定要注意，有些调用可能在无意中使用默认绑定规则。如果想“更安全”地忽略 this 绑定，你可以使用一个 DMZ 对象，比如：**Ø = Object.create(null)**，以保护全局对象。

ES6 中的箭头函数并不会使用四条标准的绑定规则，而是根据当前的词法作用域来决定this，具体来说，**箭头函数会继承外层函数调用的 this 绑定**（无论 this 绑定到什么）。这其实和 ES6 之前代码中的 self = this 机制一样。



## 对象

对象可以通过两种形式定义：声明(文字)形式和构造形式。

注意，简单基本类型本身并不是对象。



**null** 有时会被当作一种对象类型，但是这其实只是语言本身的一个 bug，即**对 null 执行 typeof null 时会返回字符串 “object"**。实际上，null 本身是基本类型。

有一种常见的错误说法是” JavaScript 中万物皆是对象“，这显然是错误的。

> **原理：**
>
> 不同的对象在底层都表示为二进制，在 JavaScript 中二进制前三位都为 0 的话会被判断为 object 类型，null 的二进制表示是全 0，自然前三位也是 0，所以执行 typeof 时会返回 ”object“。



函数就是对象的一个子类型（从技术角度来说就是“可调用的对象”）。JavaScript 中的函数是”一等公民“，因为它们本质上和普通的对象一样（只是可以调用），所以可以像操作其他对象一样操作函数（比如当作另一个函数的参数）。



### 内置对象

```
String、Number、Boolean、Object、Function、Array、Date、RegExp、Error
```

这些内置对象从表现形式来说很像其他语言中的类型（type）或者类（class），比如 Java 中的 String 类。

但是在 JavaScript 中，它们实际上只是一些内置函数。这些内置函数可以当作构造函数来使用，从而可以创建一个对应子类型的新对象。举例来说：

```js
var strPrimitive = "I am a string";
typeof strPrimitive; // "string"
strPrimitive instanceof String; // false

var strObject = new String("I am a string")；
typeof strObject; // "object"
strObject instanceof String; // true

// 检查 sub-type 对象
Object.prototype.toString.call(strObject); // [object String]
```

原始值 strPrimitive =  "I am a string" 并不是一个对象，它只是一个字面量，并且是一个不可变的值。如果要在这个字面量上执行一些操作，比如获取长度、访问其中某个字符等等，那需要将其转换为 String 对象。

在必要时语言会自动把字符串字面量转换成一个 String 对象，也就是说你并不需要显式创建一个对象。

```js
var strPrimitive = "I am a string";
console.log(strPrimitive.length); // 13
console.log(strPrimitive.charArt(3)); // "m"
```

使用以上两种方法，我们都可以直接在字符串字面量上访问属性或者方法，之所以可以这样做，是因为搜索引擎自动把字面量转换成 String 对象，所以可以访问属性和方法。

同样的事也会发生在数值字面量上，如果使用类似 42.359.toFixed(2) 的方法，引擎会把 42 转换成 new Number(42)。对于布尔字面量来说也是如此。

**null 和 undefined 没有对应的构造形式，它们只有文字形式。相反，Date 只有构造，没有文字形式。**

对于 Object、Array、Function 和 RegExp 来说，无论使用文字形式还是构造形式，它们都是对象，不是字面量。

Error 对象很少在代码中显式创建，一般是在抛出异常时被自动创建。



### 内容

对象的内容是由一些存储在特定命名位置的（任意类型的）值组成的，我们称之为属性。

需要强调的一点是，当我们说”内容“时，似乎在暗示这些值实际上被存储在对象内部，但是这只是它的表现形式。在引擎内部，这些值的存储方式时多种多样的，一般并不会存在对象容易内部，存储在对象容器内部的是这些属性的名称，它们就像指针（从技术角度来说是就是引用）一样，指向这些值真正的存储位置。

```js
var myObject = {
	a: 2
};
myObject.a; // 2
myObject["a"]; // 2
```

如果需要访问 myObject 中 a 位置上的值，我们需要使用 `.` 操作符或者 `[]`操作符。.a 语法通常被称为”属性访问“，["a"] 语法通常被称为“键访问”。

这两种语法的主要区别在于 **. 操作符要求属性名满足标识符的命名规范**，而 [".."] 语法 可以接收任意 UTF-8/Unicode 字符串作为属性名。举例来说，如果要引用名称为 ”Super-Fun!“ 的属性，那就必须使用 [”Super-Fun!“] 语法访问，因为 Super-Fun! 并不是一个有效的标识符属性名。

**在对象中，属性名永远都是字符串。**如果你使用 string（字面量）以外的其他值作为属性名，那么它首先会被转换为一个字符串。即使是数字也不例外。

#### 可计算属性名

ES6 中增加了可计算属性名，可以在文字形式中使用 [] 包裹一个表达式来当作属性名：

```js
var prefix = "foo";
var myObject = {
	[prefix + "bar"]: "hello",
    [prefix + "baz"]: "world"
};
myObject["foobar"]; // hello
myObject["foobaz"]; // world
```

#### 属性与方法

无论返回值是什么类型，每次访问对象的属性就是属性访问。如果属性访问返回的是一个函数，那它也并不是一个“方法”。属性访问返回的函数和其他函数没有任何区别（除了可能发生的隐式绑定 this ）

举例来说：

```js
function foo() {
	console.log("foo");
}
var someFoo = foo; // 对 foo 的变量引用
var myObject = {
	someFoo: foo
};
foo; // function foo(){..}
someFoo;  // function foo(){..}
myObject.someFoo; // function foo(){..}
```

someFoo 和 myObject.someFoo 只是对于同一个函数的不同引用，并不能说明这个函数是特别的或者“属于”某个对象。如果 foo() 定义时在内部有一个 this 引用，那这两个函数引用的唯一区别就是 myObject.someFoo 中的 this 会被隐式绑定到一个对象。无论哪种引用形式都不能称之为“方法”。

#### 数组

数组也是对象，所以虽然每个下标都是整数，你仍可以给数组添加属性：

```js
var myArray = ["foo",42,"bar"];
myArray.baz = "baz";
myArray.length; // 3
myArray.baz; // "baz"
```

可以看到虽然添加了命名属性（无论是通过 . 语法还是 [] 语法），数组的 length 值并未发生变化。

注意：如果你试图向数组添加一个属性，但是属性名“看起来”像一个数组，那么它会变成一个数指下标（因此会修改数组的内容而不是添加一个属性）：

```js
var myArray = ["foo",42,"bar"];
myArray["3"] = "baz";
myArray.length; // 4
myArray[3]; // "baz"
```

#### 复制对象

如何复制一个对象是一个常见问题。

举例来说，思考以下这个对象：

```js
function anotherFunction() { /*..*/ }
var antherObject = {
	c = true
};
var anthorArray = [];
var myObject = {
    a: 2,
    b: antherObject, // 引用，不是副本！
    c: antherArray, // 另一个引用
    d: anotherFunction
};
antherArray.push(antherObject, myObject);
```

如何准确地表示 myObject 的复制呢？

首先，我们应该判断它是浅复制还是深复制。

对于**浅拷贝**来说，复制出的新对象中 a 的值会复制旧对象中 a 的值，也就是 2，但是新对象中 b、c、d 三个属性其实只是三个引用，它们和就对象中的 b、c、d 引用的对象是一样的。

对于**深拷贝**来说，除了复制 myObject 以外还会复制 antherObject 和 antherArray。这时问题就来了，antherArray 引用了 antherObject 和 myObject，所以又需要复制 myObject ，这样就**会由于循环引用而导致死循环**。

解决方法：

对于 JSON 安全（也就是说可以被序列化为一个 JSON 字符串并且可以根据这个字符串解析出一个结构和值完全一样的对象）的对象来说，有一种巧妙的复制方法：

```js
var myObj = JSON.parse( JSON.stringify( someObj ) );
```

当然这种方法需要保证对象是 JSON 安全的，所以只适用于部分情况。



相比深拷贝，浅拷贝非常移动并且问题要少得多，所以 ES6 定义了 Object.assign(..) 方法来实现浅拷贝。

Object.assign(..) 方法的第一个参数是目标对象，之后还可以跟一个或多个源对象。它会遍历一个或多个源对象的所有可枚举（enumerable）的自有键（owned key）并把它们复制（使用 = 操作符赋值）到目标对象，最后返回目标对象。

```js
var newObj = Object.assign( {}, myObject );

newObj.a; // 2
newObj.b == antherObject; // true
newObj.c == antherArray; // true
newObj.d == anotherFunction; // true
```

> 但是需要注意一点，由于 Object.assign(..) 就是使用 = 操作符来复制，所以源对象属性的一些特性（比如 writable）不会被复制到目标对象。

#### 属性描述符

在 ES5 之前，JavaScript 语言本身并没有提供可以直接检测属性特性的方法，比如判断属性是否是只读。

但是从 ES5 开始，所有的属性都具备了属性描述符。

```js
var myObject = {
	a: 2
};
Object.getOwnPropertyDescriptor( myObject, "a");
/*
{
	value: 2,
	writable: true,
	enumerable: true,
	configurable: true
}
*/
```

