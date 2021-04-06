#  你不知道的JavaScript 上——作用域和闭包202004

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

如你所见，这个普通的对象属性对应的属性描述符（也被称为“数据描述符”，因为它只保存一个数据值）可不仅仅只是一个 2 。它海包含另外三个特性：writable、enumerable、configurable。

在创建普通属性时属性描述符会使用默认值，我们也可以使用 Object.defineProperty(..) 来添加一个新属性或者修改一个已有属性（如果它是 configurable）并对特性进行设置。

> configurable 修改成 false 是单向操作。
>
> 有一个小小的例外：即使属性是 configurable: false，我们还是可以把 writable 的状态由 true 改为 false，但是无法由 false 改为 true。
>
> 除了无法修改，configurable: false 还会禁止删除这个属性（delete 语句）

enumerable 描述符控制的是属性是否会出现在对象的属性枚举中，比如说 for .. in 循环。

#### 不变性

1. 对象常量

   结合 writable: false 和 configurable: false 就可以创建一个真正的常量属性（不可修改、重定义或者删除）；

2. 禁止扩展

   如果你想禁止一个对象添加新属性并且保留已有属性，可以使用 Object.preventExtensions(..)

3. 密封

    Object.seal(..) 会创建一个“密封”的对象，这个方法实际上会在一个现有对象上调用 Object.preventExtensions(..) 并把所有现有属性标记为 configurable: false。

   所以，密封之后不仅不能添加新属性，也不能重新配置或者删除任何现有属性（虽然可以修改属性的值）。

4. 冻结

    Object.freeze(..) 会创建一个冻结对象，这个方法实际上会在一个现有对象上调用 Object.seal(..) 并把所有“数据访问”属性标记为 writable: false，这样就无法修改它们的值。

   这个方法是你可以应用在对象上的级别最高的不可变性，它会禁止对于对象本身及其任意直接函数的修改（不过就像我们之前说过的，这个对象引用的其他对象是不受影响的）。

   你可以**“深度冻结”**一个对象，具体方法为，首先在这个对象上调用 Object.freeze(..) ，然后遍历它引用的所有对象并在这些对象上调用  Object.freeze(..)。但是一定要小心，因为这样做有可能会在无意中冻结其他（共享）对象。

#### [[Get]]

```js
var myObject = {
	a: 2
};
myObject.a; // 2
```

myObject.a 是一次属性访问，但是这条语句并不仅仅是在 myObject 中查找名字为 a 的属性，虽然看起来好像是这样。

在语言规范中，myObject.a 在 myObject 上实际上是实现了 [[Get]] 操作（有点像函数调用：[[Get]]\() )。对象默认的内置 [[Get]] 操作首先在对象中查找是否有名称相同的属性，如果找到就会返回找到这个属性的值。

然而，如果没有找到名称相同的属性，按照 [[Get]] 算法的定义会执行另外一种非常重要的行为（遍历可能存在的 [[Prototype]]链，也就是原型链）。

#### [[Put]]

既然有可以获取属性值的 [[Get]] 操作，就一定有对应的 [[Put]] 操作。

[[Put]] 被触发时，实际的行为取决于许多因素，包括对象中是否已经存在这个属性（这是最重要的因素）。

如果已经存在这个属性，[[Put]] 算法大致会检查下面这些内容。

1. 属性是否是访问描述符？如果是并且存在 setter 就调用 setter。
2. 属性的数据描述符中 writable 是否是 false ？如果是，在非严格模式下静默失败，在严格模式下抛出 TypeError 异常。
3. 如果都不是，将该值设置为属性的值。

如果对象中不存在这个属性，[[Put]] 操作会更加复杂。

#### Getter 和 Setter

对象默认的 [[Put]] 和 [[Get]] 操作分别可以控制属性值的设置和获取。

在 ES5 中可以使用 getter 和 setter 部分改写默认操作，但是只能应用在单个属性上，无法应用在整个对象上。getter 是一个隐藏函数，会在获取属性值时调用。setter 也是一个隐藏函数，会在设置属性值时调用。

当你给一个属性定义 getter 、setter 或者两者都有时，这个属性会被定义为“访问描述符”（和“数据描述符”相对）。对于访问描述符来说，JavaScript 会忽略它们的 value 和 writable 特性，取而代之的是关心 set 和 get （还有 configurable 和 enumerable）特性。

```js
var myObject = {
	// 给 a 定义一个 getter
    get a() {
		return 2;
    }
};
Object.defineProperty(
	myObject, // 目标对象
    "b", // 属性名
    { // 描述符
		// 给 b 设置一个 getter
        get: function() { return this.a * 2 },
        
        // 确保 b 会出现在对象的属性列表中
        enumerable: true
    }
);
myObject.a; // 2
myObject.b; // 4
```

不管是对象文字语法中的 get a() { .. } ,还是 defineProperty(..) 中的显式定义，二者都会在对象中创建一个不包含值得属性，对于这个属性的访问会自动调用一个隐藏函数，它的返回值会被当作属性访问的返回值。

```js
var myObject = {
	// 给 a 定义一个 getter
    get a() {
		return 2;
    }
};
myObject.a =3；
myObject.a； // 2
```

由于我们只定义了 a 的 getter ，所以对 a 的值进行设置时 set 操作会忽略复制操作，不会抛出错误。

通常来说 getter 和 setter 是承兑出现的（只定义一个的话通常会产生意料之外的行为）：

```js
var myObject = {
	// 给 a 定义一个 getter
    get a() {
		return this._a_;
    }，
    set a(val) {
		this._a_ = val * 2;
    }
};
myObject.a =2；
myObject.a； // 4
```

#### 存在性

如 myObject.a 的属性访问返回值可能是 undefined，但是这个值有可能是属性中存储的 undefined，也可能是因为属性不存在所以返回 undefined。

区分方法：

```js
var myObject = {
    a: 2
};
("a" in myObject); // true
("b" in myObject); // false

myObject.hasOwnProperty("a"); // true
myObject.hasOwnProperty("b"); // false
```

in 操作符会检查属性是否在对象及其 [[Prototype]] 原型链中。相比之下， hasOwnProperty(..) 只会检查属性是否在 myObject 对象中，不会检查 [[Prototype]] 链。

所有的普通对象都可以通过对于 Object.prototype 的委托来访问 hasOwnProperty(..)，但是有的对象可能没有连接到 Object.prototype （通过 Object.create(null) 来创建）。在这种情况下，形如 myObject.hasOwnProperty(..) 就会失败。

这时可以使用一种更加强硬的方法来进行判断：Object.prototype.hasOwnProperty.call(myObject, "a")，它借用基础的 hasOwnProperty(..) 方法并把它显式绑定到 myObject 上。

> 看起来 in 操作符可以检查容器内是否有某个值，但是它实际上检查的是某个属性名是否存在。对于数组来说这个区别非常重要， 4 in [2, 4, 6] 的结果并不是 true ，因为 [2, 4, 6] 这个数组中包含的属性名是 0，1，2，没有4。

##### 枚举

“可枚举”就相当于“可以出现在对象属性的遍历中”。因此可以使用 for..in 循环来判断。

> 在数组上应用 for..in 循环有时会产生出人意料的结果，因为这种枚举不仅会包含所有数值索引，还会包含所有可枚举属性。最好只在对象上应用 for..in 循环。

也可以通过另一种方式来区分属性是否可枚举：

```js
var myObject = {};
Object.defineProperty(
	myObject,
    "a",
    // 让 a 像普通属性一样可以枚举
    { enumerrable: true, value: 2 }
);
Object.defineProperty(
	myObject,
    "b",
    // 让 b 不可枚举
    { enumerrable: false, value: 3 }
);

myObject.propertyIsEnumerable("a"); // true
myObject.propertyIsEnumerable("b"); // false

Object.keys(myObject); // ["a"]
Object.getOwnPropertyNames( myObject ); // ["a", "b"]
```

propertyIsEnumerable(..) 会检查给定的属性名是否直接存在于对象中（而不是在原型链上）并且满足 enumerable: true。

Object.keys(..) 会返回一个数组，包含所有可枚举属性，Object.getOwnPropertyNames(..) 会返回一个数组，包含所有属性，无论它们是否可枚举。

**in 和 hasOwnProperty(..) 的去区别在于是否查找 [[Prototype]] 链，然而 Object.keys(..) 和 Object.getOwnPropertyNames(..) 都只会查找对象直接包含的属性。**

（目前）并没有内置的方法可以获取 in 操作符使用的属性列表（对象本身的属性及 [[Prototype]] 链 中的所有属性）。不过你可以递归遍历某个对象的 [[Prototype]] 链并保存每一层中使用 Object.keys(..) 得到的属性列表——只包含可枚举属性。

### 遍历

ES5 中增加了一些数组的辅助迭代器，包括 forEach(..), every(..) 和 some(..)。每种辅助函数都可以接受一个回调函数并把它应用到数组的每个元素上，，唯一的区别就是它们对于回调函数返回值的处理方式不同。

forEach(..) 会遍历数组中的所有值并忽略回调函数的返回值。every(..) 会一直运行直到回调函数返回 false（或者“假”值），some(..) 会一直运行知道回调函数返回 true（或者“真”值）。

every(..) 和 some(..) 中特殊的返回值和普通 for 循环中的 break 语句类似，它们会提前终止遍历。

使用 for..in 遍历对象是无法直接获取属性值的，因为它实际上遍历的是对象中的所有可枚举属性，你需要手动获取属性值。

> 遍历数组下标时采用的是数字顺序（ for 循环或者其他迭代器），但是遍历对象属性时的顺序是不确定的，在不同的 JS 引擎中可能不一样。因此，在不同的环境中需要保证一致性时，一定不要相信任何观察到的顺序，它们是不可靠的。

ES6 增加了一种用来遍历数组的 for..of 循环语法（如果对象本身定义了迭代器的话也可以遍历对象）

for..of 循环首先会向被访问对象请求一个迭代器对象，然后通过调用迭代器对象的 next() 方法来遍历所有返回值。

数组有内置的 @@iterator，因此 for..of 可以直接应用在数组上。

普通的对象没有内置的 @@iterator，所以无法自动完成 for.of 遍历。不过我们可以给任何想遍历的对象定义 @@iterator。

对于用户定义的对象来说，结合 for..of 循环和自定义迭代器可以组成非常强大的对象操作工具。



## 混合对象“类”

类的设计模式：实例化( instantiation )、继承（ inheritance ）和（相对）多态（ polymorphism ）。

这些概念实际上无法直接对应到 JavaScript 的对象机制，因此我们会介绍许多 JavaScript 开发者所使用的解决方法（比如混入，mixin ）

#### 混入

在继承或者实例化时，JavaScript 的对象机制并不会自动执行复制行为。简单来说，JavaScript 中只有对象，并不存在可以被实例化的“类”。一个对象并不会被复制到其他对象，它们会被关联起来。

由于在其他语言中类表现出来的都是复制行为，因此 JavaScript 开发者也想出了一个方法来模拟类的复制行为，这个方法就是混入。混入有两种类型：显式和隐式。

JavaScript 中的函数无法（用标准、可靠的方法）真正地复制，所以你只能复制对共享函数对象的引用（函数就是对象）。如果你修改了共享的函数对象，比如添加了一个属性，那 Vehicle 和 Car 都会受到影响。



混入模式（无论显式还是隐式）可以用来模拟类的复制行为，但是通常会产生丑陋并且脆弱的语法，比如显示伪堕胎（ OtherObj,methodName.call(this, ...)），这会让代码更加难懂并且难以维护。

此外，显示混入实际上无法完全模拟类的复制行为，因为对象（和函数！别忘了函数也是对象）只能复制引用，无法复制被引用的对象或者函数本身。忽视这一点会导致许多问题。

总地来说，在 JavaScript 中模拟类是得不偿失的，虽然能解决当前的问题，但是可能会埋下更多的隐患。



## 原型

### [[Prototype]]

JavaScript 中的对象有一个特殊的 [[Prototype]] 内置属性，其实就是对于其他对象的引用。几乎所有的对象在创建时 [[Prototype]] 属性都会被赋予一个非空的值。

对象的 [[Prototype]] 链接可以为空，不过比较少见。

对于默认的 [[Get]] 操作来说，如果无法在对象本身找到需要的属性，就会继续访问对象的 [[Prototype]] 链：

```js
var anotherObject = {
    a: 2
};
// 创建一个关联到 anotherObject 的对象
var myObject = Object.create( anotherObject );
myObject.a; // 2
```

显然 myObject.a 并不存在，但是尽管如此，属性访问仍然成功地（在 anotherObject 中）找到了值 2。

但是，如果 anotherObject 中也找不到 a 并且 [[Prototype]] 链不为空的话，就会继续查找下去。

这个过程会持续到找到匹配的属性名或者查找完整条 [[Prototype]] 链。如果是后者的话， [[Get]] 操作的返回值是 undefined。

使用 for..in 遍历对象时原理和查找 [[Prototype]] 链类似，任何可以通过原型链访问到（并且是 enumerable）的属性都会被枚举。使用 in 操作符来检查属性在对象中是否存在时，同样会查找对象的整条原型链（无论属性是否可枚举）。

因此，当你通过各种语法进行属性查找时都会查找 [[Prototype]] 链，直到找到属性或者查找完整条原型链。

#### Object.prototype

到哪里才是 [[Prototype]] 的“尽头”呢？

所有普通的 [[Prototype]] 链最终都会指向内置的 Object.prototype。由于所有的“普通”（内置，不是特定主机的扩展）对象都“源于”（或者说把 [[Prototype]] 链的顶端设置为）这个 Object.prototype 对象，所以它包含 JavaScript 中许多通用的功能。（如 .toString() , .valueOf(), hasOwnProperty(..), .isPrototypeOf(..) 。

#### 属性设置和屏蔽

给一个对象设置属性并不仅仅是添加一个新属性或者修改已有的属性值。完整的过程：

```js
myObject.foo = "bar";
```

如果 myObject 对象中包含名为 foo 的普通数据访问属性，这条赋值语句只会修改已有的属性值。

**如果 foo 不是直接存在于 myObject 中， [[Prototype]] 链就会被遍历，类似 [[Get]] 操作。**如果原型链上找不到 foo，foo 就会被直接添加到 myObject 上。

然而，如果 foo 存在于原型链上层，赋值语句 myObject.foo = "bar" 的行为就会有些不同（而且可能很出人意料）。

如果属性名 foo 既出现在 myObject 中也出现在 myObject 的  [[Prototype]] 链上层，那么就会发生**屏蔽**。myObject 中包含的 foo 属性会屏蔽原型链上层的所有 foo 属性，因为 **myObject.foo 总是会选择原型链中最底层的 foo 属性**。

屏蔽比我们想象中复杂。下面我们分析一下如果 foo 不直接存在于myObject 中而是存在于原型链上层时 myObject.foo = "bar" 会出现的三种情况。

1. 如果在  [[Prototype]] 链上层存在名为 foo 的普通数据访问属性并且没有被标记为只读（writable: false),那就会直接在 myObject 中添加一个名为 foo 的新属性，它是**屏蔽属性**。
2. 如果在  [[Prototype]] 链上层存在 foo ，但是它被标记为只读（writable: false），那么无法修改已有属性或者在 myObject 上创建屏蔽属性。如果运行在严格模式下，代码会抛出一个错误。否则，这条赋值语句会被忽略。总之，不会发生屏蔽。
3. 如果在  [[Prototype]] 链上层存在 foo 并且它是一个 setter ，那就一定会调用这个 setter。foo 不会被添加到（或者说屏蔽于） myObject ，也不会重新定义 foo 这个setter。

大多数开发者都认为如果向  [[Prototype]] 链上层已经存在的属性（ [[Put]] ）赋值，就一定会触发屏蔽，但是如你所见，三种情况中只有第一种事这样的。

如果你希望在第二种和第三种情况下也屏蔽 foo，那就不能使用 **= 操作符**来赋值，而是使用 Object.defineProperty(..) 来向 myObject 添加 foo。

> 第二种情况可能是最令人意外的，只读属性会阻止  [[Prototype]] 链下层隐式创建（屏蔽）同名属性。这样做主要是为了模拟类属性的继承。你可以把原型链上层的 foo 看作是父类种的属性，它会被 myObject 继承（复制），这样一来 myObject 中的 foo 属性也是只读，所以无法创建。但是一定要注意，实际上并不会发生类似的继承复制。这看起来有点奇怪，myObject 对象竟然会因为其他对象中有一个只读 foo 就不能包含 foo 属性。更奇怪的是，这个限制只存在于 = 赋值中，使用 Object.defineProperty(..) 并不会受到影响。

有些情况下会隐式产生屏蔽，一定要当心。思考下面的代码：

```js
var anotherObject = {
    a: 2
};
var myObject = Object.create( anotherObject );

anotherObject.a; // 2
myObject.a; // 2

anotherObject.hasOwnProperty("a"); // true
myObject.hasOwnProperty("a"); // false

myObject.a++; // 隐式屏蔽！

anotherObject.a; // 2
myObject.a; // 3

myObject.hasOwnProperty("a"); // true
```

尽管 myObject.a++ 看起来应该（通过委托）查找并增加 anotherObject.a 属性，但是别忘了 ++ 操作相当于 myObject.a = myObject.a +1。因此 ++ 操作首先会通过  [[Prototype]] 查找属性 a 并从  anotherObject.a 获取当前属性值 2，然后给这个值加 1，接着用 [[Put]] 将值 3 赋给 myObject中新建的屏蔽属性 a。

修改委托属性时一定要小心。如果想让 anotherObject.a 的值增加，唯一的办法是 anotherObject.a++。

### “类”

JavaScript 和面向类的语言不同，它并没有类来作为对象的抽象模式或者说蓝图。JavaScript 中只有对象。

JavaScript 才是真正应该被称为“面向对象”的语言，因为它是少有的可以不通过类，直接创建对象的语言。

所有的函数都会默认拥有一个名为 prototype 的共有并且不可枚举的属性，它会指向另一个对象：

```js
function Foo() {
    // ...
}
Foo.prototype; // { }
```

这个对象通常被称为 Foo 的**原型**，因为我们通过名为 Foo.prototype  的属性引用来访问它。

Foo 的**原型**，这个对象是在调用 new Foo() 时创建的，最后会被关联到这个“Foo 点 prototype” 对象上。

```js
function Foo() {
    // ...
}
var a = new Foo();
Object.getPrototypeOf( a ) === Foo.protoytpe; // true
```

调用 new Foo() 时会创建 a ，其中的一步就是给 a 一个内部的  [[Prototype]] 链接，关联到 Foo.prototype 指向的那个对象。

在面向类的语言中，类可以被复制多次，就像用模具制作东西一样。

但是在 JavaScript 中，并没有类似的复制机制。你不能创建一个类的多个实例，只能创建多个对象，它们 [[Prototype]] 关联的是同一个对象。但是在默认情况下并不会进行复制，因此这些对象之间并不会完全失去联系，它们是互相关联的。

new Foo( ) 会生成一个新对象（我们称之为 a ）,这个新对象的内部链接 [[Prototype]] 关联的是 Foo.prototype 对象。

最后我们得到了两个对象，它们之间互相关联，就是这样。我们并没有初始化一个类，实际上我们并没有从“类”中复制任何行为到一个对象中，只是让两个对象互相关联。

实际上，绝大多数 JavaScript 开发者不知道的秘密是， new Foo() 这个函数调用实际上并没有直接创建关联，这个关联只是一个意外的副作用。new Foo() 只是间接完成了我们的目标：一个关联到其他对象的新对象。

继承意味着复制操作，JavaScript （默认）并不会复制对象属性。相反，JavaScript 会在两个对象之间创建一个关联，这样一个对象就可以通过委托访问另一个对象的函数和属性。委托这个属于可以更加准确地描述 JavaScript 中对象地关联机制。

```js
function Foo() {
    // ...
}
Foo.protoytpe.constructor === Foo; // true

var a = new Foo();
a.constructor === Foo; // true
```

 Foo.protoytpe 默认（在代码中第一行声明时！）有一个公有并且不可枚举的属性 .constructor，这个属性引用的是对象关联的函数（本例中是 Foo）。此外，我们可以看到通过“构造函数”调用 new Foo() 创建的对象也有一个 .constructor 属性，指向“创建这个对象的函数”。

> 实际上 a 本身并没有 .constructor 属性。而且，虽然 a.constructor 确实指向 Foo 函数，但是这个属性并不是表示 a 由 Foo “构造”。

上一段代码很容易让人认为 Foo 是一个构造函数，因为我们使用 new 来调用它并且看到它“构造”了一个对象。

实际上，Foo 和你程序中的其他函数没有任何区别。函数本身并不是构造函数，然而，当你在普通函数调用前面加上 new 关键字之后，就会把这个函数调用变成一个**“构造函数调用”**。实际上， new 会劫持所有普通函数并对构造对象的形式来调用它。

换句话说，在 JavaScript 中对于“构造函数”最准确的解释是，所有带 new 的函数调用。

函数不是构造函数，但是当且仅当使用 new 时，函数调用会变成“构造函数调用”。



模仿类的行为：

```js
function Foo(name) {
	this.name = name;
}
Foo.prototype.maName = function() {
	return this.name;
};
var a = new Foo("a");
var b = new Foo("b");

a.myName(); // "a"
b.myName(); // "b"
```

在这段代码中，看起来似乎创建 a 和 b 时会把 Foo.prototype 对象复制到这两个对象中，然而事实并不是这样。

在创建的过程中，a 和 b 的内部 [[Prototype]] 都会关联到 Foo.prototype 上。当 a 和 b 中无法找到 myName 时，它会（通过委托）在 Foo.prototype 上找到。

Foo.prototype 的 .constructor 属性只是 Foo 函数在声明时的默认属性。如果你创建了一个新对象并替换了函数默认的 .prototype 对象引用，那么新对象并不会自动获得 .constructor 属性。

```js
function Foo() { /*..*/ }
Foo.prototype = { /*..*/ }; // 创建一个新原型对象

var a1 = new Foo();
a.constructor === Foo; // false
a.constructor === Object; // true
```

原因：a1 并没有 .constructor 属性，所以它会委托 [[Prototype]] 链上的 Foo.prototype。但是这个对象也没有 .constructor 属性（默认情况是有的），所以它会继续委托，这次会委托给委托链顶端的 Object.prototype。这个对象有  .constructor 属性，指向内置的 Object(..) 函数。

“ constructor 并不表示被构造”。

 .constructor 并不是一个不可变属性。它是不可枚举的，但是它的值是可写的（可以被修改）。此外，你可以给任意 [[Prototype]] 链中的任意对象添加一个名为  constructor 的属性或者对其进行修改，你可以任意对其赋值。

a1.constructor 是一个非常不可靠并且不安全的引用。通常来说要尽量避免使用这些引用。

### （原型）继承

原型风格的代码：

```js
function Foo(name) {
    this.name = name;
}
Foo.prototype.maName = function() {
	return this.name;
};
function Bar(name,label) {
	Foo.call(this, name);
    this.label = label;
}
// 我们创建了一个新的 Bar.prototype 对象并关联到 Foo.prototype
Bar.prototype = Object.create(Foo.prototype);

// 注意！现在没有 Bar.prototype.constructor 了
// 如果你需要这个属性的话可能需要手动修复一下它
Bar.prototype.myLabel = function() {
	return this.label;    
};
var a = new Bar("a", "obj a");
a.myName(); // "a"
a.myLabel(); // "obj a"
```

这段代码的核心部分就是语句 Bar.prototype = Object.create(Foo.prototype)。调用 Object.create(..) 会凭空创建一个“新”对象并把对象内部的 [[Prototype]] 关联到你指定的对象（本例中是 Foo.prototype ）。

换句话说，这条语句的意思是：“创建一个新的 Bar.prototype 对象并把它关联到 Foo.prototype ”。

声明 function Bar( ) { .. } 时，和其他函数一样，Bar 会有一个 .prototype 关联到默认的对象，但是这个对象并不是我们想要的 Foo.prototype。 因此我们创建了一个新对象并把它关联到我们希望的对象上，直接把原始的关联对象抛弃掉。

注意，下面这两种方式是常见的错误做法，实际上它们都存在一些问题：

```js
// 和你想要的机制不一样！
Bar.prototype = Foo.prototype;
// 基本上满足你的需求，但是可能会产生一些副作用 
Bar.prototype = new Foo();
```

Bar.prototype = Foo.prototype 并不会创建一个关联到 Bar.prototype 的新对象，它只是让 Bar.prototype 直接引用 Foo.prototype 对象。因此当你执行类似 Bar.prototype.myLabel = ... 的赋值语句时会直接修改 Foo.prototype 对象本身。显然这不是你想要的结果，否则你根本不需要 Bar 对象，直接使用 Foo 就可以了，这样代码也会更简单一些。

Bar.prototype = new Foo() 的确会创建一个关联到 Bar.prototype 的新对象。但是它使用了 Foo(..) 的“构造函数调用”，如果函数 Foo 有一些副作用（比如写日志、修改状态、注册到其他对象、给 this 添加数据属性，等等）的话，就会影响到 Bar( ) 的“后代”，后果不堪设想。

因此，要创建一个合适的关联对象，我们必须使用 **Object.create(..)** 而不是使用具有副作用的 Foo(..)。这样做唯一的缺点就是需要创建一个新对象然后把旧对象抛弃掉，不能直接修改已有的默认对象。

如果能有一个标准并且可靠的方法来修改对象的 [[Prototype]] 关联就好了。在 ES6 之前，我们只能通过设置 \.\_\_proto\__ 属性来实现，但是这个方法并不是标准并且无法兼容所有浏览器。ES6 添加了辅助函数 Object.setPrototypeOf(..) ，可以用标准并且可靠的方法来修改关联。

```js
// ES6 之前需要抛弃默认的 Bar.prototype
Bar.prototype = Object.create(Foo.prototype);

// ES6 开始可以直接修改现有的 Bar.prototype
Object.setPrototypeOf(Bar.prototype, Foo.prototype);
```

如果忽略掉 Object.create(..) 方法带来的轻微性能损失（抛弃的对象需要进行垃圾回收），它实际上比 ES6 及其之后的方法更短而且可读性更高。

#### 检查“类”关系

对于下面的代码

```js
function Foo() {
    // ...
}
Foo.prototype.blah = ...;
var a = new Foo();
```

如何通过内省找出 a 的“祖先”（委托关联）呢？

**第一种方法**是站在“类”的角度来判断“

```js
a instanceof Foo; // true
```

instanceof 操作符的左操作数是一个普通的对象，右操作数是一个函数。 instanceof 回答的问题是：在 a 的整条 [[Prototype]] 链中是否右指向 Foo.prototype 的对象？

不过这个方法只能处理对象（a）和函数（带 .prototype 引用的 Foo）之间的关系。如果你想判断两个对象（比如 a 和 b）之间是否通过 [[Prototype]] 链关联，只用 instanceof 无法实现。

> 如果使用内置的 .bind(..) 函数来生成一个硬绑定函数的话，该函数是没有 .prototype 属性的。在这样的函数上使用 instanceof 的话，目标函数的 .prototype 会代替硬绑定函数的 .prototype。
>
> 通常我们不会在“构造函数调用”中使用硬绑定函数，不过如果你这么做的话，实际上相当于直接调用目标函数。同理，在硬绑定函数上使用 instanceof 也相当于直接在目标函数上使用 instanceof。

**第二种判断**  [[Prototype]] 反射的方法，它更加简洁:

```js
Foo.prototype.isPrototypeOf(a); // true
```

注意，在本例中，我们实际上并不关心（甚至不需要）Foo，我们只需要一个可以用来判断的对象（本例中是 Foo.prototype）就行。isPrototypeOf(..) 回答的问题是：在 a 的整条 [[Prototype]] 链中是否出现过 Foo.prototype ？

我们只需要两个对象就可以判断它们之间的关系。举例来说：

```js
// 非常简单：b 是否出现在 c 的 [[Prototype]] 链中？
b.isPrototypeOf(a)
```

注意，这个方法并不需要使用函数（“类”），它直接使用 b 和 c 之间的对象引用来判断它们的关系。

我们也可以直接获取一个对象的 [[Prototype]] 链。在 ES5 中，标准的方法是：

```js
Object.getPrototypeOf(a);
```

可以验证一下，这个对象引用是否和我们想的一样：

```js
Object.getPrototypeOf(a) === Foo.prototype; // true
```

这个奇怪的 \.\_\_proto\__ （在ES6之前并不是标准！）属性“神奇地”引用了内部的 [[Prototype]] 对象，如果你想直接查找（甚至可以通过  \.\_\_proto\_\.\_\_proto\_...来遍历）原型链的话，这个方法非常有用。

\.\_\_proto\__ 实际上并不存在于你正在使用的对象中（本例中是 a ）。实际上，它和其他的常用函数（.toString( )、.isPrototypeOf(..)，等等）一样，存在于内置的Object.prototype 中。（它们是不可枚举的）

此外，\.\_\_proto\__ 看起来很像一个属性，但是实际上它更像一个 getter/setter。

\.\_\_proto\__ 的实现大致上是这样的：

```js
Object.defineProperty( Object.prototype, "__proto__",{
   get: function() {
		return Object.getPrototypeOf(this);
   }, 
   set: function(o) {
		// ES6 中的 setPrototypeOf(..)
         Object.setPrototypeOf(this, o);
         return o;
   }
});
```

因此，访问（获取值）a.\_\_proto\__ 时，实际上时调用了 a.\_\_proto\__( ) （调用 getter 函数）。

### 对象关联

现在我们知道了， [[Prototype]] 机制就是存在于对象中的一个内部链接，它会引用其他对象。

通常来说，这个链接的作用时：如果在对象上没有找到需要的属性或者方法引用，引擎就会继续在 [[Prototype]] 关联的对象上进行查找。同理，如果在后者中也没有找到需要的引用就会继续查找它的 [[Prototype]] ，以此类推。这一系列对象的链接被称为**“原型链”**。

#### 创建关联

Object.create(..) 会创建一个新对象（bar）并把它关联到我们指定的对象（foo），这样我们就可以充分发挥 [[Prototype]] 机制的威力（委托）并且避免不必要的麻烦（比如使用 new 的构造函数调用会生成 .prototype 和 .constructor 引用）。

> Object.create(null) 会创建一个拥有空（或者说 nul） [[Prototype]] 链接的对象，这个对象无法进行委托。由于这个对象没有原型链，所以 instanceof 操作符无法进行判断，因此总是会返回 false。这些特殊的空 [[Prototype]] 对象通常被称作“字典”，它们完全不会受到原型链的干扰，因此非常适合用来存储数据。

**Object.create(..) 的 polyfill 代码**，实现了部分功能。

```js
// Object.create(..) 是 ES5 中新增的函数
if(!Object.create) {
	Object.create = function(o) {
		function F(){}
         F.prototype = o;
         return new F();
    };
}
```

这段 polyfill 代码使用了一个一次性函数 F，我们通过改写它的 .prototype 属性使其指向想要关联的对象，然后再使用 new F( ) 来构造一个新对象进行关联。

Object.create(..) 的第二个参数制定了需要添加到新对象中的属性名以及这些属性的属性描述符。因为 ES5 之前的版本无法模拟属性操作符，所以 polyfill 代码无法实现这个附加功能。

通常来说不会使用 Object.create(..) 的附加功能。

### 小结

**如果要访问对象中并不存在的一个属性，[[Get]] 操作就会查找对象内部 [[Prototype]] 关联的对象。这个关联关系实际上定义了一条“原型链”（有点像嵌套的作用域链），再查找属性时会对它进行遍历。**

所有普通对象都有内置的 Object.prototype，指向原型链的顶端（比如说全局作用域），如果在原型链中找不到指定的属性就会停止。toString( )、valueOf( ) 和其他一些通用的功能都存在于 Object.prototype 对象上，因此语言中所有的对象都可以使用它们。

关联两个对象最常用的方法就是使用 new 关键词进行函数调用，在调用的 4 个步骤中会创建一个关联其他对象的新对象。

使用 new 调用函数时会把新对象的 .prototype 属性关联到“其他对象”。带 new 的函数调用通常被称为“**构造函数调用**”，尽管它们实际上和传统面向类语言中的类构造函数不一样。

虽然这些 JavaScript 机制和传统面向类语言中的“类初始化”和“类继承”很相似，但是 **JavaScript 中的机制有一个核心区别，那就是不会进行复制，对象之间时通过内部的 [[Prototype]] 链关联的。**

出于各种原因，以“继承”结尾的术语（包括“原型继承”）和其他面向对象的术语都无法帮助你理解 JavaScript 的真实机制（不仅仅时限制我们的思维模式）。

相比之下，“委托”是一个更合适的术语，因为对象之间的关系不是复制而是**委托**。

## 行为委托

[[Prototype]] 机制就是指对象中的一个内部链接引用另一个对象。

如果在第一个对象上没有找到需要的属性或者方法引用，引擎就会继续在 [[Prototype]] 关联的对象上进行查找。同理，如果在后者中也没有找到需要的引用就会继续查找它的 [[Prototype]] ，以此类推。这一系列对象的链接，被称为“**原型链**”。

换句话说，JavaScript 中这个机制的本质就是**对象之间的关联关系。**

### 面向委托的设计

假设我们需要在软件中建模一些类似的任务（“XYZ”、“ABC”等）

#### 类理论

如果使用类，那设计方法可能是这样的：定义一个通用父（基）类，可以将其命名为 Task，在 Task 类中定义所有任务都有的行为。接着定义子类 XYZ 和 ABC，它们都继承自 Task 并且会添加一些特殊的行为来处理对应的任务。

非常重要的是，类设计模式鼓励你在及城市使用方法重写（和多态），比如说在 XYZ 任务中重写 Task 中定义的一些通用方法，甚至在添加新行为时通过 super 掉头这个方法的原始版本。你会发现许多行为可以先“抽象”到父类然后再用子类进行特殊化（重写）。

#### 委托理论

现在我们试着来使用**委托行为**而不是类来思考同样的问题。

首先你会定义一个名为 Task 的**对象**，它会包含所有任务都可以使用（写作使用，读作委托）的具体行为。接着，对于每个任务（“XYZ”、“ABC”等）你都会定义一个对象来存储对应的数据和行为。你会把特定的任务对象都关联到 Task **功能对象**是，让它们在需要的时候可以进行**委托**。

