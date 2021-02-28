# JS数据类型——概念

## 介绍一下js的数据类型有哪些，值是如何存储的

JS共有8种数据类型，分为基本数据类型和引用数据类型。

- 基本数据类型：string, number, boolean, undefined, null, symbol, bigint
- 引用数据类型：object，包含普通对象-Object，数组对象-Array，正则对象-RegExp，日期对象-Date，数学函数-Math，函数对象-Function


存储方式：

基本数据类型存储在栈中，

而引用数据类型的地址存储在栈中，通过地址寻找堆中的实体。

看起来没有错误，但实际上是有问题的。可以考虑一下闭包的情况，如果变量存在栈中，那函数调用完`栈顶空间销毁`，闭包变量不就没了吗？

其实还是需要补充一句:

> 闭包变量是存在堆内存中的。
>

## null是对象吗？为什么？

虽然typeof null会返回object，但是这是由于历史遗留问题，000开头代表对象，而null为全0，从而也会被判定为对象。

## null 和 undefined 的区别？

首先 Undefined 和 Null 都是基本数据类型，这两个基本数据类型分别都只有一个值，就是 undefined 和 null。
undefined 代表的含义是未定义，
null 代表的含义是空对象。一般变量声明了但还没有定义的时候会返回 undefined，null主要用于赋值给一些可能会返回对象的变量，作为初始化。当我们使用双等 号对两种类型的值进行比较时会返回 true，使用三个等号时会返回 false。

## 0.1+0.2为什么不等于0.3？

0.1和0.2在转换成二进制后会无限循环，由于标准位数的限制后面多余的位数会被截掉，此时就已经出现了精度的损失，相加后因浮点数小数位的限制而截断的二进制数字在转换为十进制就会变成0.30000000000000004。

主要是因为JavaScript同样采用IEEE754标准，在64位中存储一个数字的有效数字形式

![img](https://pics5.baidu.com/feed/8326cffc1e178a823aa0c40fa760d388a877e86e.png?token=9f3f1313b9a709e27837a51737e41f07&s=E6F82072C178512008D5D0480300C0F2)

第0位表示符号位，0表示整数1表示负数，第1~11位存储指数部分，第12~63位存小数部分；

由于二进制的有效数字总是表示为`1.xxx...`这样的形式，尾数部分在规约形式下的第一位默认为1，故存储时第一位省略不写，尾数部分存储有效数字小数点后的xxx...，最长52位，因此，JavaScript提供的有效数字最长为53个二进制位（尾数部分52位+被省略的1位）

由于需要对求和结果规格化(用有效数字表示)，右规导致低位丢失，此时需对丢失的低位进行舍入操作，遵循IEEE754舍入规则，会有精度损失

## 如何理解bigint？

**什么是BigInt?**

> BigInt是一种新的数据类型，用于当整数值大于Number数据类型支持的范围时。这种数据类型允许我们安全地对`大整数`执行算术操作，表示高分辨率的时间戳，使用大整数id，等等，而不需要使用库。

**为什么需要BigInt?**

在JS中，所有的数字都以双精度64位浮点格式表示，那这会带来什么问题呢？

这导致JS中的Number无法精确表示非常大的整数，它会将非常大的整数四舍五入，确切地说，JS中的Number类型只能安全地表示-9007199254740991(-(2^53-1))和9007199254740991（(2^53-1)），任何超出此范围的整数值都可能失去精度。

```javascript
console.log(999999999999999);  //=>10000000000000000
```

同时也会有一定的安全性问题:

```javascript
9007199254740992 === 9007199254740993;    // → true 居然是true!
```

**如何创建并使用BigInt？**

要创建BigInt，只需要在数字末尾追加n即可。

```javascript
console.log( 9007199254740995n );    // → 9007199254740995n	
console.log( 9007199254740995 );     // → 9007199254740996
```

另一种创建BigInt的方法是用BigInt()构造函数、

```javascript
BigInt("9007199254740995");    // → 9007199254740995n
```

简单使用如下:

```javascript
10n + 20n;    // → 30n	
10n - 20n;    // → -10n	
+10n;         // → TypeError: Cannot convert a BigInt value to a number	
-10n;         // → -10n	
10n * 20n;    // → 200n	
20n / 10n;    // → 2n	
23n % 10n;    // → 3n	
10n ** 3n;    // → 1000n	

const x = 10n;	
++x;          // → 11n	
--x;          // → 9n
console.log(typeof x);   //"bigint"
```

**值得警惕的点**

1. BigInt不支持一元加号运算符, 这可能是某些程序可能依赖于 + 始终生成 Number 的不变量，或者抛出异常。另外，更改 + 的行为也会破坏 asm.js代码。
2. 因为隐式类型转换可能丢失信息，所以不允许在bigint和 Number 之间进行混合操作。当混合使用大整数和浮点数时，结果值可能无法由BigInt或Number精确表示。

```javascript
10 + 10n;    // → TypeError
```

1. 不能将BigInt传递给Web api和内置的 JS 函数，这些函数需要一个 Number 类型的数字。尝试这样做会报TypeError错误。

```javascript
Math.max(2n, 4n, 6n);    // → TypeError
```

1. 当 Boolean 类型与 BigInt 类型相遇时，BigInt的处理方式与Number类似，换句话说，只要不是0n，BigInt就被视为truthy的值。

```javascript
if(0n){//条件判断为false

}
if(3n){//条件为true

}
```

1. 元素都为BigInt的数组可以进行sort。
2. BigInt可以正常地进行位运算，如|、&、<<、>>和^

# JS数据类型——检测

## typeof 是否能正确判断类型？

对于原始类型来说，除了 null 都可以调用typeof显示正确的类型。

```javascript
typeof 1 // 'number'
typeof NaN // 'number'
typeof '1' // 'string'
typeof undefined // 'undefined'
typeof true // 'boolean'
typeof Symbol() // 'symbol'
```

但对于引用数据类型，除了函数之外，都会显示"object"。

```javascript
typeof [] // 'object'
typeof {} // 'object'
typeof console.log // 'function'
```

## Instanceof

如果我们想判断一个对象的正确类型，这时候可以考虑使用 `instanceof`，因为内部机制是通过原型链来判断的，在后面的章节中我们也会自己去实现一个 `instanceof`。

```js
const Person = function() {}
const p1 = new Person()
p1 instanceof Person // true

var str1 = 'hello world'
str1 instanceof String // false

var str2 = new String('hello world')
str2 instanceof String // true

```

对于原始类型来说，你想直接通过 `instanceof` 来判断类型是不行的，当然我们还是有办法让 `instanceof` 判断原始类型的。

```js
class PrimitiveString {
  static [Symbol.hasInstance](x) {
    return typeof x === 'string'
  }
}
console.log('hello world' instanceof PrimitiveString) // true
```

`Symbol.hasInstance` 其实就是一个能让我们自定义 `instanceof` 行为的东西，以上代码等同于 `typeof 'hello world' === 'string'`，所以结果自然是 `true` 了。这其实也侧面反映了一个问题， `instanceof` 也不是百分之百可信的。

## 手写instanceof

核心：原型链的向上查找。

```js
function myInstanceof(left, right) {
    let prototype = right.prototype
    left = left.__proto__
    while (true) {
      if (left === null || left === undefined)
        return false
      if (prototype === left)
        return true
      left = left.__proto__
    }
  }
  console.log(myInstanceof('123',String));
  console.log(myInstanceof(2,String));
  console.log(myInstanceof(2,Number));
```

## Object.prototype.toString.call()

使用 Object 对象的原型方法 toString ，使用 call 进行狸猫换太子，借用Object的 toString 方法

```javascript
var a = Object.prototype.toString;
 
console.log(a.call(2));
console.log(a.call(true));
console.log(a.call('str'));
console.log(a.call([]));
console.log(a.call(function(){}));
console.log(a.call({}));
console.log(a.call(undefined));
console.log(a.call(null));
```

## 如何判断空数组

arr.length == 0

arr == false

## 如何判断空对象

```
JSON.stringify({}) == "{}"
```

```
Object.keys(data).length === 0
```

# JS数据类型——转换

首先我们要知道，在 JS 中类型转换只有三种情况，分别是：

- 转换为布尔值
- 转换为数字
- 转换为字符串

![img](https://user-gold-cdn.xitu.io/2020/5/28/1725b947653323df?imageslim)

## [] == ![]结果是什么？为什么？

解析:

== 中，左右两边都需要转换为数字然后进行比较。

[]转换为数字为0。

![] 首先是转换为布尔值，由于[]作为一个引用类型转换为布尔值为true,

因此![]为false，进而在转换成数字，变为0。

0 == 0 ， 结果为true

## JS中类型转换有哪几种？

JS中，类型转换只有三种：

- 转换成数字
- 转换成布尔值
- 转换成字符串

转换具体规则如下:

> 注意"Boolean 转字符串"这行结果指的是 true 转字符串的例子

![img](https://user-gold-cdn.xitu.io/2019/10/20/16de9512eaf1158a?imageslim)

##  == 和 ===有什么区别？

===叫做严格相等，是指：左右两边不仅值要相等，类型也要相等，例如'1'===1的结果是false，因为一边是string，另一边是number。

==不像===那样严格，对于一般情况，只要值相等，就返回true，但==还涉及一些类型转换，它的转换规则如下：

- 两边的类型是否相同，相同的话就比较值的大小，例如1==2，返回false
- 判断的是否是null和undefined，是的话就返回true
- 判断的类型是否是String和Number，是的话，把String类型转换成Number，再进行比较
- 判断其中一方是否是Boolean，是的话就把Boolean转换成Number，再进行比较
- 如果其中一方为Object，且另一方为String、Number或者Symbol，会将Object转换成字符串，再进行比较

# 作用域

## 什么是作用域？

 作用域是定义变量的区域，它有一套访问变量的规则，这套规则来管理浏览器引擎如何在当前作用域以及嵌套的作用域中根据变量（标识符）进行变量查找。

## 什么是作用域链？

作用域链的作用是保证对执行环境有权访问的所有变量和函数的有序访问，通过作用域链，我们可以访问到外层环境的变量和函数。

作用域链的本质上是一个指向变量对象的指针列表。变量对象是一个包含了执行环境中所有变量和函数的对象。作用域链的前端始终都是当前执行上下文的变量对象。全局执行上下文的变量对象（也就是全局对象）始终是作用域链的最后一个对象。

## 执行上下文

执行上下文有且只有三类，全局执行上下文，函数上下文，与eval上下文；由于eval一般不会使用，这里不做讨论。

**1.全局执行上下文**

全局执行上下文只有一个，在客户端中一般由浏览器创建，也就是我们熟知的window对象，我们能通过this直接访问到它。

![img](https://img2018.cnblogs.com/blog/1213309/201908/1213309-20190831171325789-1160629712.png)

全局对象window上预定义了大量的方法和属性，我们在全局环境的任意处都能直接访问这些属性方法，同时window对象还是var声明的全局变量的载体。我们通过var创建的全局对象，都可以通过window直接访问。

![img](https://img2018.cnblogs.com/blog/1213309/201908/1213309-20190831172214034-866054334.png)

**2.函数执行上下文**

函数执行上下文可存在无数个，每当一个函数被调用时都会创建一个函数上下文；需要注意的是，同一个函数被多次调用，都会创建一个新的上下文。

说到这你是否会想，上下文种类不同，而且创建的数量还这么多，它们之间的关系是怎么样的，又是谁来管理这些上下文呢，这就不得不说说执行上下文栈了。

**3.执行上下文栈(执行栈)**

执行上下文栈(下文简称执行栈)也叫调用栈，执行栈用于存储代码执行期间创建的所有上下文，具有LIFO（Last In First Out后进先出，也就是先进后出）的特性。

JS代码首次运行，都会先创建一个全局执行上下文并压入到执行栈中，之后每当有函数被调用，都会创建一个新的函数执行上下文并压入栈内；由于执行栈LIFO的特性，所以可以理解为，JS代码执行完毕前在执行栈底部永远有个全局执行上下文。

```javascript
function f1() {
    f2();
    console.log(1);
};

function f2() {
    f3();
    console.log(2);
};

function f3() {
    console.log(3);
};

f1();//3 2 1
```

我们通过执行栈与上下文的关系来解释上述代码的执行过程，为了方便理解，我们假象执行栈是一个数组，在代码执行初期一定会创建全局执行上下文并压入栈，因此过程大致如下：

```javascript
//代码执行前创建全局执行上下文
ECStack = [globalContext];
// f1调用
ECStack.push('f1 functionContext');
// f1又调用了f2，f2执行完毕之前无法console 1
ECStack.push('f2 functionContext');
// f2又调用了f3，f3执行完毕之前无法console 2
ECStack.push('f3 functionContext');
// f3执行完毕，输出3并出栈
ECStack.pop();
// f2执行完毕，输出2并出栈
ECStack.pop();
// f1执行完毕，输出1并出栈
ECStack.pop();
// 此时执行栈中只剩下一个全局执行上下文
```

那么到这里，我们解释了执行栈与执行上下文的存储规则；还记得我在前文提到代码执行前JS引擎会做准备创建执行上下文吗，具体怎么创建呢，我们接着说。

 **4.执行上下文创建阶段**

执行上下文创建分为创建阶段与执行阶段两个阶段，较为难理解应该是创建阶段，我们先说创建阶段。

JS执行上下文的创建阶段主要负责三件事：确定this---创建词法环境组件（LexicalEnvironment）---创建变量环境组件（VariableEnvironment），两者区别在于词法环境存放函数声明与const let声明的变量，而变量环境只存储var声明的变量。

这里我就直接借鉴了他人翻译资料的伪代码，来表示这个创建过程：

```javascript
ExecutionContext = {  
    // 确定this的值
    ThisBinding = <this value>,
    // 创建词法环境组件
    LexicalEnvironment = {},
    // 创建变量环境组件
    VariableEnvironment = {},
};
```

# this

this 是很多人会混淆的概念，但是其实它一点都不难，只是网上很多文章把简单的东西说复杂了。在这一小节中，你一定会彻底明白 this 这个概念的。

```javascript
function foo() {
  console.log(this.a)
}
var a = 1
foo()

const obj = {
  a: 2,
  foo: foo
}
obj.foo()

const c = new foo()
```

接下来我们一个个分析上面几个场景

- 对于直接调用 `foo` 来说，不管 `foo` 函数被放在了什么地方，`this` 一定是 `window`
- 对于 `obj.foo()` 来说，我们只需要记住，谁调用了函数，谁就是 `this`，所以在这个场景下 `foo` 函数中的 `this` 就是 `obj` 对象
- 对于 `new` 的方式来说，`this` 被永远绑定在了 `c` 上面，不会被任何方式改变 `this`

箭头函数的this

```javascript
function a() {
  return () => {
    return () => {
      console.log(this)
    }
  }
}
console.log(a()()())
```

首先箭头函数其实是没有 `this` 的，箭头函数中的 `this` 只取决包裹箭头函数的第一个普通函数的 `this`。在这个例子中，因为包裹箭头函数的第一个普通函数是 `a`，所以此时的 `this` 是 `window`。另外对箭头函数使用 `bind` 这类函数是无效的。

最后种情况也就是 `bind` 这些改变上下文的 API 了，对于这些函数来说，`this` 取决于第一个参数，如果第一个参数为空，那么就是 `window`。

那么说到 `bind`，不知道大家是否考虑过，如果对一个函数进行多次 `bind`，那么上下文会是什么呢？

```javascript
let a = {}
let fn = function () { console.log(this) }
fn.bind().bind(a)() // => ?
```

其实我们可以把上述代码转换成另一种形式

```javascript
// fn.bind().bind(a) 等于
let fn2 = function fn1() {
  return function() {
    return fn.apply()
  }.apply(a)
}
fn2()
```

可以从上述代码中发现，不管我们给函数 `bind` 几次，`fn` 中的 `this` 永远由第一次 `bind` 决定，所以结果永远是 `window`。

```javascript
let a = { name: 'yck' }
function foo() {
  console.log(this.name)
}
foo.bind(a)() // => 'yck'
```

以上就是 `this` 的规则了，但是可能会发生多个规则同时出现的情况，这时候不同的规则之间会根据优先级最高的来决定 `this` 最终指向哪里。

**例题：**

```javascript
var a = {
  name: "bytedance",
  func: function () {
    console.log(this.name);
  },
};
a.func(); // bytedance
var fun1 = a.func;
fun1(); // undefined
a.func.call({ name: "toutiao" }); //toutiao
```

首先，`new` 的方式优先级最高，接下来是 `bind` 这些函数，然后是 `obj.foo()` 这种调用方式，最后是 `foo` 这种调用方式，同时，箭头函数的 `this` 一旦被绑定，就不会再被任何方式所改变。

![img](https://user-gold-cdn.xitu.io/2018/11/15/16717eaf3383aae8?imageslim)

# 手写call、apply、bind

1.call、apply与bind都用于改变this绑定，但call、apply在改变this指向的同时还会执行函数，而bind在改变this后是返回一个全新的boundFcuntion绑定函数，这也是为什么上方例子中bind后还加了一对括号 ()的原因。

2.bind属于硬绑定，返回的 boundFunction 的 this 指向无法再次通过bind、apply或 call 修改；call与apply的绑定只适用当前调用，调用完就没了，下次要用还得再次绑。

3.call与apply功能完全相同，唯一不同的是call方法传递函数调用形参是以散列形式，而apply方法的形参是一个数组。在传参的情况下，call的性能要高于apply，因为apply在执行时还要多一步解析数组。

## 手写call

首先从以下几点来考虑如何实现这几个函数

- 不传入第一个参数，那么上下文默认为 `window`
- 改变了 `this` 指向，让新的对象可以执行该函数，并能接受参数

```js
Function.prototype.myCall = function(context) {
  if(typeof this !== 'function'){
    throw new TypeError('error');
  }
  context = context || window;
  context.fn = this;
  const args = [...arguments].slice(1);
  const result = context.fn(...args);
  delete context.fn;
  return result;
}
```

以下是对实现的分析：

- 首先 `context` 为可选参数，如果不传的话默认上下文为 `window`
- 接下来给 `context` 创建一个 `fn` 属性，并将值设置为需要调用的函数
- 因为 `call` 可以传入多个参数作为调用函数的参数，所以需要将参数剥离出来
- 然后调用函数并将对象上的函数删除

## 手写apply

```js
Function.prototype.myApply = function(context) {
	if(typeof this !== 'function'){
    throw new TypeError('error');
  }
  context = context || window;
  context.fn = this;
  let result;
  if(arguments[1]){
    result = context.fn(...arguments[1]);
  }else{
    reslut = context.fn();
  }
  delete context.fn;
  return result;
}
```

## 手写bind

`bind` 的实现对比其他两个函数略微地复杂了一点，因为 `bind` 需要返回一个函数，需要判断一些边界问题，以下是 `bind` 的实现

```javascript
Function.prototype.myBind = function(context) {
  if(typeof this !== 'function'){
    throw new TypeError('error');
  }
  const _this = this;
  const args = [...arguments].slice(1)
  return function F() {
    if(this instanceof F) {
      return new _this(...args,...arguments);
    }
    return _this.apply(context,args.concat(...arguments));
  }
}
```

以下是对实现的分析：

- 前几步和之前的实现差不多，就不赘述了
- `bind` 返回了一个函数，对于函数来说有两种方式调用，一种是直接调用，一种是通过 `new` 的方式，我们先来说直接调用的方式
- 对于直接调用来说，这里选择了 `apply` 的方式实现，但是对于参数需要注意以下情况：因为 `bind` 可以实现类似这样的代码 `f.bind(obj, 1)(2)`，所以我们需要将两边的参数拼接起来，于是就有了这样的实现 `args.concat(...arguments)`
- 最后来说通过 `new` 的方式，在之前的章节中我们学习过如何判断 `this`，对于 `new` 的情况来说，不会被任何方式改变 `this`，所以对于这种情况我们需要忽略传入的 `this`

# new

## new的原理

在调用 `new` 的过程中会发生以上四件事情：

1. 新生成了一个对象
2. 链接到原型
3. 绑定 this
4. 返回新对象

## 手写new

```js
function myNew() {
  let obj = {};
  let Con = [].shift.call(arguments);
  obj.__proto__ = Con.prototype;
  let result = Con.apply(obj, arguments);
  return result instanceof Object? result : obj;
}
```

- 创建一个空对象
- 获取构造函数
- 设置空对象的原型
- 绑定 `this` 并执行构造函数
- 确保返回值为对象

# 闭包

## 什么是闭包？

闭包的定义其实很简单：函数 A 内部有一个函数 B，函数 B 可以访问到函数 A 中的变量，那么函数 B 就是闭包。

> 红宝书(p178)上对于闭包的定义：闭包是指有权访问另外一个函数作用域中的变量的函数，

> MDN 对闭包的定义为：闭包是指那些能够访问自由变量的函数。 （其中自由变量，指在函数中使用的，但既不是函数参数arguments也不是函数的局部变量的变量，其实就是另外一个函数作用域中的变量。）

```js
function A() {
  let a = 1
  window.B = function () {
      console.log(a)
  }
}
A()
B() // 1
```

很多人对于闭包的解释可能是函数嵌套了函数，然后返回一个函数。其实这个解释是不完整的，就比如我上面这个例子就可以反驳这个观点。

在 JS 中，闭包存在的意义就是让我们可以间接访问函数内部的变量。

## 循环中使用闭包解决 `var` 定义函数的问题

```js
for (var i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i)
  }, i * 1000)
}
```

首先因为 `setTimeout` 是个异步函数，所以会先把循环全部执行完毕，这时候 `i` 就是 6 了，所以会输出一堆 6。

解决办法有三种，第一种是使用闭包的方式

```javascript
for (var i = 1; i <=5; i++) {
	(function(j){
    setTimeout(function timer() {
      console.log(j)
    },j * 1000)
  })(i);
}
```

在上述代码中，我们首先使用了立即执行函数将 `i` 传入函数内部，这个时候值就被固定在了参数 `j` 上面不会改变，当下次执行 `timer` 这个闭包的时候，就可以使用外部函数的变量 `j`，从而达到目的。

第二种就是使用 `setTimeout` 的第三个参数，这个参数会被当成 `timer` 函数的参数传入。

```javascript
for (var i = 1; i <= 5; i++) {
  setTimeout(
    function timer(j) {
      console.log(j)
    },
    i * 1000,
    i
  )
}
```

第三种就是使用 `let` 定义 `i` 了来解决问题了，这个也是最为推荐的方式

```javascript
for (let i = 1; i <= 5; i++) {
  setTimeout(function timer() {
    console.log(i)
  }, i * 1000)
}
```

## 闭包的使用场景？

在定时器、事件监听、Ajax请求、跨窗口通信、Web Workers或者任何异步中，只要使用了回调函数，实际上就是在使用闭包。

以下的闭包保存的仅仅是window和当前作用域。

```javascript
// 定时器
setTimeout(function timeHandler(){
  console.log('111');
}，100)

// 事件监听
$('#app').click(function(){
  console.log('DOM Listener');
})
```

IIFE(立即执行函数表达式)创建闭包, 保存了`全局作用域window`和`当前函数的作用域`，因此可以全局的变量。

```javascript
var a = 2;
(function IIFE(){
  // 输出2
  console.log(a);
})();
```

使用闭包实现一个方法，第一次调用返回1，第二次返回2，第三次3，以此类推。

```javascript
(function add(){
	var counter = 0;
    return function(){
        return counter +=1
    }
})();
```

# JavaScript原型

## 原型

对于obj来说，我们可以通过`__proto__`找到它的原型。

在JavaScript中，每当定义一个函数数据类型(普通函数、类)时候，都会天生自带一个prototype属性，这个属性指向函数的原型对象。

当函数经过new调用时，这个函数就成为了构造函数，返回一个全新的实例对象，这个实例对象有一个`__proto__`属性，指向构造函数的原型对象。

![img](https://user-gold-cdn.xitu.io/2019/10/20/16de955a81892535?imageslim)

## 原型链

JavaScript对象通过`__proto__` 指向父类对象，直到指向Object对象为止，这样就形成了一个原型指向的链条, 即原型链。

![img](https://user-gold-cdn.xitu.io/2019/10/20/16de955ca89f6091?imageslim)

![img](https://user-gold-cdn.xitu.io/2019/11/6/16e3fec6c1fb209e?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)

## 获取原型的方法

- `p.__proto__`
- `p.constructor.prototype`
- `Object.getPrototypeOf(p)`

## 查找对象的属性

对象的 hasOwnProperty() 来检查对象自身中是否含有该属性

对in运算符的介绍：**如果指定的属性在指定的对象或其原型链中，则`in` 运算符返回`true`。**

# JavaScript继承

## 原型继承

```javascript
function Parent1() {
  this.name = "parent1";
  this.play = [1,2,3];
}
function Child1() {
  this.type = "child1";
}
Child1.prototype = new Parent1();
console.log(new Child1());
```

看似没有问题，父类的方法和属性都能够访问，但实际上有一个潜在的不足。举个例子：

```js
var s1 = new Child1();
var s2 = new Child1();
s1.play.push(4);
console.log(s1.play, s2.play);
```

![image-20200706100448935](/Users/chenzhimin/Library/Application Support/typora-user-images/image-20200706100448935.png)

明明我只改变了s1的play属性，为什么s2也跟着变了呢？很简单，因为两个实例使用的是同一个原型对象。

## 构造继承(借用call)

```js
function Parent2() {
  this.name = "parent2"
}
function Child2() {
  Parent2.call(this);
  this.type = "child2";
}
console.log(new Child2());
```

这样写的时候子类虽然能够拿到父类的属性值，但是问题是父类原型对象中一旦存在方法那么子类无法继承。那么引出下面的方法。

## 组合继承（上两种方法的结合）

```javascript
function Parent3() {
  this.name = "parent3";
  this.play = [1,2,3];
}
function Child3() {
  Parent3.call(this);
  this.type = "child3";
}
Child3.prototype = new Parent3;
var s3 = new Child3();
var s4 = new Child3();
s3.play.push(4);
console.log(s3.play, s4.play);
```

那就是Parent3的构造函数会多执行了一次（Child3.prototype = new Parent3();）。

## 寄生组合继承

```javascript
function Parent4() {
	this.name = "parent4";
  this.play = [1,2,3];
}
function Child4() {
  Parent4.call(this);
  this.type = "child4";
}
Child4.prototype = Object.create(Parent4.prototype, {
  constructor: {
    value: Child,
    enumerable: false,
    writable: true,
    configurable: true
  }
})

const child = new Child4();
```

## ES6 class继承

```javascript
class Parent {
  constructor(value) {
    this.val = value
  }
  getValue() {
    console.log(this.val)
  }
}
class Child extends Parent {
  constructor(value) {
    super(value)
  }
}
let child = new Child(1)
child.getValue() // 1
child instanceof Parent // true
```

`class` 实现继承的核心在于使用 `extends` 表明继承自哪个父类，并且在子类构造函数中必须调用 `super`，因为这段代码可以看成 `Parent.call(this, value)`。

当然了，之前也说了在 JS 中并不存在类，`class` 的本质就是函数。

# JavaScript事件

## 三种事件模型

### DOM0级事件模型

又称为原始事件模型，在该模型中，事件不会传播，即没有事件流的概念。事件绑定监听函数比较简单, 有两种方式:

```html
<input type="button" onclick="fun()">
```

```javascript
var btn = document.getElementById('.btn');
btn.onclick = fun;
btn.onclick = null;
```

### IE事件模型

事件处理阶段(target phase)。事件到达目标元素, 触发目标元素的监听函数。

事件冒泡阶段(bubbling phase)。事件从目标元素冒泡到document, 依次检查经过的节点是否绑定了事件监听函数，如果有则执行。

事件绑定监听函数的方式如下:

```javascript
attachEvent(eventType, handler)
```

事件移除监听函数的方式如下:

```javascript
detachEvent(eventType, handler)
```

参数说明：

- eventType指定事件类型，记得加on
- handler是事件处理函数

具体例子：

```javascript
var btn = document.getElementById('.btn');
btn.attachEvent(‘onclick’, showMessage);
btn.detachEvent(‘onclick’, showMessage);
```

### DOM2级事件模型

在该事件模型中，一次事件共有三个过程:

- 事件捕获阶段(capturing phase)。事件从document一直向下传播到目标元素, 依次检查经过的节点是否绑定了事件监听函数，如果有则执行。

- 事件处理阶段(target phase)。事件到达目标元素, 触发目标元素的监听函数。

- 事件冒泡阶段(bubbling phase)。事件从目标元素冒泡到document, 依次检查经过的节点是否绑定了事件监听函数，如果有则执行。


事件绑定监听函数的方式如下:

```javascript
addEventListener(eventType, handler, useCapture)
```

事件移除监听函数的方式如下：

```javascript
removeEventListener(eventType, handler, useCapture)
```

具体例子：

```javascript
var btn = document.getElementById('.btn');
btn.addEventListener(‘click’, showMessage, false);
btn.removeEventListener(‘click’, showMessage, false);
```

参数说明:

- eventType指定事件类型(不要加on)

- handler是事件处理函数

- useCapture是一个boolean用于指定是否在捕获阶段进行处理，一般设置为false与IE浏览器保持一致。


## 事件委托（代理）

**事件委托** 本质上是利用了浏览器事件冒泡的机制。因为事件在冒泡过程中会上传到父节点，并且父节点可以通过事件对象获取到 目标节点，因此可以把子节点的监听函数定义在父节点上，由父节点的监听函数统一处理多个子元素的事件，这种方式称为事件代理。

使用事件代理我们可以不必要为每一个子元素都绑定一个监听事件，这样减少了内存上的消耗。并且使用事件代理我们还可以实现事件的动态绑定，比如说新增了一个子节点，我们并不需要单独地为它添加一个监听事件，它所发生的事件会交给父元素中的监听函数来处理。

## 事件传播

当**事件**发生在DOM元素上时，该事件并不完全发生在那个元素上。在“当事件发生在DOM元素上时，该事件并不完全发生在那个元素上。

事件传播有三个阶段：

1. 捕获阶段–事件从 window 开始，然后向下到每个元素，直到到达目标元素事件或event.target。
2. 目标阶段–事件已达到目标元素。
3. 冒泡阶段–事件从目标元素冒泡，然后上升到每个元素，直到到达 window。

## 事件捕获

当事件发生在 DOM 元素上时，该事件并不完全发生在那个元素上。在捕获阶段，事件从window开始，一直到触发事件的元素。`window----> document----> html----> body ---->目标元素`

## 事件冒泡

事件冒泡刚好与事件捕获相反，`当前元素---->body ----> html---->document ---->window`。当事件发生在DOM元素上时，该事件并不完全发生在那个元素上。在冒泡阶段，事件冒泡，或者事件发生在它的父代，祖父母，祖父母的父代，直到到达window为止。

## 如果对一个容器同时绑定捕获和冒泡事件，先触发哪个？为什么？

当同一个容器绑定两个事件，遵循先捕获，后冒泡 ，有一个前提是 点击的事件先执行

https://blog.csdn.net/qq_32766999/article/details/101195839

## 常见的基础事件

![image-20200706112458476](/Users/chenzhimin/Library/Application Support/typora-user-images/image-20200706112458476.png)

## React合成事件

React并不是将click事件绑在该div的真实DOM上，而是在document处监听所有支持的事件，当事件发生并冒泡至document处时，React将事件内容封装并交由真正的处理函数运行。

![img](https://user-gold-cdn.xitu.io/2017/10/9/8792eeae6dc6011274986acf42a76b15?imageslim)

# DOM操作

## 创建新节点

```javascript
createDocumentFragment()    //创建一个DOM片段
createElement()   //创建一个具体的元素
createTextNode()   //创建一个文本节点
```

## 添加、移除、替换、插入

```javascript
appendChild(node)
removeChild(node)
replaceChild(new,old)
insertBefore(new,old)
```

## 查找

```javascript
getElementById();
getElementsByName();
getElementsByTagName();
getElementsByClassName();
querySelector();
querySelectorAll();
```

getElementById()和querySelector()的区别主要前者是动态合集，而后者获取的是静态合集（即使新增加了元素，但是获取的仍然是原先的）

## 属性操作

```javascript
getAttribute(key);
setAttribute(key, value);
hasAttribute(key);
removeAttribute(key);
```

## 大量插入dom节点的办法

我们经常使用javascript来操作DOM元素，比如使用appendChild()方法。
每次调用该方法时，浏览器都会重新渲染页面。如果大量的更新DOM节点，则会非常消耗性能，影响用户体验.
javascript提供了一个文档碎片DocumentFragment的机制。
如果将文档中的节点添加到文档片段中，就会从文档树中移除该节点。

把所有要构造的节点都放在文档片段中执行，这样可以不影响文档树，也就不会造成页面渲染。
当节点都构造完成后，再将文档片段对象添加到页面中，这时所有的节点都会一次性渲染出来，
这样就能减少浏览器负担，提高页面渲染速度。

```html
<ul id="root"></ul>
<script>
var root = document.getElementById('root')
var fragment = document.createDocumentFragment()
for(let i = 0; i < 1000; i++){
	let li = document.createElement('li')
	li.innerHTML = '我是li标签'
    fragment.appendChild(li)
}
root.appendChild(fragment);
</script>
```

# JS数组

## 数组的常用方法

- push
- pop
- shift
- unshift
- slice
- splice
- join
- concat
- map
- filter
- some
- every
- forEach

## js数组如果内部元素个数已经到达了一开始设定的长度再向其中加入元素会怎么样？会报错吗？

不会报错，可以继续添加。

## 数组乱序

1. 利用的数组的原生sort方法进行随机排序

```javascript
function randomSort(arr) {
  return arr.sort(function(a,b){
    return Math.random() > 0.5 ? 1: -1;
  })
}
```

2. 洗牌原理：从数组的最后位置开始，然后从前面随机一个位置，对这两个数进行交换！直到循环完毕

```javascript
function shuffleSort(arr) {
  let len = arr.length;
  let i = len-1;
  while(i>=0) {
    let index = Math.floor(Math.random()*i);
    let temp = arr[i];
    arr[i] = arr[index];
    arr[index] = temp;
    i--;
  }
  return arr;
}
```

## 判断数组中是否包含某个值

**方法一：array.indexOf**

> 此方法判断数组中是否存在某个值，如果存在，则返回数组元素的下标，否则返回-1。

```javascript
var arr=[1,2,3,4];
var index=arr.indexOf(3);
console.log(index);
```

**方法二：array.includes(searcElement[,fromIndex])**

> 此方法判断数组中是否存在某个值，如果存在返回true，否则返回false

```javascript
var arr=[1,2,3,4];
if(arr.includes(3))
    console.log("存在");
else
    console.log("不存在");
```

**方法三：array.find(callback[,thisArg])**

> 返回数组中满足条件的**第一个元素的值**，如果没有，返回undefined

```javascript
var arr=[1,2,3,4];
var result = arr.find(item =>{
    return item > 3
});
console.log(result);
```

**方法四：array.findeIndex(callback[,thisArg])**

> 返回数组中满足条件的第一个元素的下标，如果没有找到，返回`-1`]

```javascript
var arr=[1,2,3,4];
var result = arr.findIndex(item =>{
    return item > 3
});
console.log(result);
```

当然，for循环当然是没有问题的，这里讨论的是数组方法，就不再展开了。

## 数组扁平化

对于前端项目开发过程中，偶尔会出现层叠数据结构的数组，我们需要将多层级数组转化为一级数组（即提取嵌套数组元素最终合并为一个数组），使其内容合并且展开。那么该如何去实现呢？

需求:多维数组=>一维数组

```javascript
let ary = [1, [2, [3, [4, 5]]], 6];// -> [1, 2, 3, 4, 5, 6]
let str = JSON.stringify(ary);
```

**1. 调用ES6中的flat方法**

```javascript
ary = ary.flat(Infinity);
```

**2. replace + split**

```javascript
ary = str.replace(/(\[|\])/g, '').split(',')
```

**3. replace + JSON.parse**

```javascript
str = str.replace(/(\[|\])/g, '');
str = '[' + str + ']';
ary = JSON.parse(str);
```

**4. 普通递归**

```javascript
let result = [];
let fn = function(ary) {
  for(let i = 0; i < ary.length; i++) {
    let item = ary[i];
    if (Array.isArray(ary[i])){
      fn(item);
    } else {
      result.push(item);
    }
  }
}
```

**5. 利用reduce函数迭代**

```javascript
function flatten(ary) {
    return ary.reduce((pre, cur) => {
        return pre.concat(Array.isArray(cur) ? flatten(cur) : cur);
    }, []);
}
let ary = [1, 2, [3, 4], [5, [6, 7]]]
console.log(flatten(ary))
```

**6：扩展运算符**

```javascript
//只要有一个元素有数组，那么循环继续
while (ary.some(Array.isArray)) {
  ary = [].concat(...ary);
}
```

## 数组去重

**1. 遍历数组使用indexOf去重**

```javascript
arr.forEach(item=>{
    if(hash.indexOf(item) == '-1'){
        hash.push(item);
    }    
})
console.log(hash);    //[23, 44, 5, 2, 1, 7, 8]
```

**2.sort排序后遍历过滤数组**

```javascript
var hash = [arr[0]];
arr.forEach((item,index)=>{
    if(item != hash[hash.length-1]){
        hash.push(item)
    }
})
```

**3.ES6实现**

```javascript
function unique(arr) {
return Array.from(new Set(arr))
}
let arr = [1,'true', true,null, NaN,'NaN',0, 'a',{},1,'true', true,null, NaN,'NaN',0, 'a',{}]
console.log(unique(arr))

[...new Set(arr)]
```

**4.利用for嵌套for，然后splice去重**

```javascript
function unique(arr) {
for(let i=0; i<arr.length;i++){
	for(let j=i+1; j<arr.length; j++){
		if(arr[i] == arr[j]){ //第一个等同于第二个，splice方法删除第二个
			arr.splice(j,1);
    		j--;
    	}
    }
  }
	return arr
}
let arr = [1,'true', true,null, NaN,'NaN',0, 'a',{},1,'true', true,null, NaN,'NaN',0, 'a',{}]
console.log(unique(arr))
```

**5.利用hasOwnProperty**

```javascript
function unique(arr) {
var obj = {}
return arr.filter(function(item, index,arr){
return obj.hasOwnProperty(typeof item + item)? false: (obj[typeof item+item] = true);
});
}
let arr = [1,1, 4,5,5, 4,'NaN',0, 'a',{},1,'true', true,null, NaN,'NaN',0, 'a',{}]
console.log(unique(arr))
```

**6.利用filter**

```javascript
function unique(arr) {
	return arr.filter(function(item, index,arr){
		return arr.indexOf(item, 0) === index;
	});
}
let arr = [1,1, 4,5,5, 4,'NaN',0, 'a',{},1,'true', true,null, NaN,'NaN',0, 'a',{}]
console.log(unique(arr))
```

## 数组中的高阶函数

### 高阶函数的定义

概念非常简单，如下:

> `一个函数`就可以接收另一个函数作为参数或者返回值为一个函数，`这种函数`就称之为高阶函数。

### map

- 参数:接受两个参数，一个是回调函数，一个是回调函数的this值(可选)。

其中，回调函数被默认传入三个值，依次为当前元素、当前索引、整个数组。

- 创建一个新数组，其结果是该数组中的每个元素都调用一个提供的函数后返回的结果
- 对原来的数组没有影响

```javascript
let nums = [1, 2, 3];
let obj = {val: 5};
let newNums = nums.map(function(item,index,array) {
  return item + index + array[index] + this.val; 
  //对第一个元素，1 + 0 + 1 + 5 = 7
  //对第二个元素，2 + 1 + 2 + 5 = 10
  //对第三个元素，3 + 2 + 3 + 5 = 13
}, obj);
console.log(newNums);//[7, 10, 13]
```

当然，后面的参数都是可选的 ，不用的话可以省略。

### reduce

- 参数: 接收两个参数，一个为回调函数，另一个为初始值。回调函数中四个默认参数，依次为积累值、当前值、当前索引和整个数组。

```javascript
let nums = [1, 2, 3];
// 多个数的加和
let newNums = nums.reduce(function(preSum,curVal,currentIndex,array) {
  return preSum + curVal; 
}, 0);
console.log(newNums);//6
```

不传默认值会怎样？

不传默认值会自动以第一个元素为初始值，然后从第二个元素开始依次累计。

### filter

参数: 一个函数参数。这个函数接受一个默认参数，就是当前元素。这个作为参数的函数返回值为一个布尔类型，决定元素是否保留。

filter方法返回值为一个新的数组，这个数组里面包含参数里面所有被保留的项。

```javascript
let nums = [1, 2, 3];
// 保留奇数项
let oddNums = nums.filter(item => item % 2);
console.log(oddNums);
```

### sort

参数: 一个用于比较的函数，它有两个默认参数，分别是代表比较的两个元素。

举个例子:

```javascript
let nums = [2, 3, 1];
//两个比较的元素分别为a, b
nums.sort(function(a, b) {
  if(a > b) return 1;
  else if(a < b) return -1;
  else if(a == b) return 0;
})
```

当比较函数返回值大于0，则 a 在 b 的后面，即a的下标应该比b大。

反之，则 a 在 b 的后面，即 a 的下标比 b 小。

整个过程就完成了一次升序的排列。

当然还有一个需要注意的情况，就是比较函数不传的时候，是如何进行排序的？

答案是将数字转换为字符串，然后根据字母unicode值进行升序排序，也就是根据字符串的比较规则进行升序排序。

# JS字符串

## 手写indexOf

```js
String.prototype.myIndexof = function(searchElement, fromIndex) {
  var len = this.length;
  if(fromIndex === null) {
    fromIndex = 0;
  }
  for(let i = fromIndex; i < len; i++) {
    if(searchElement === this[i]) {
      return i;
    }
  }
  return -1;
}
```

## 返回字符串的位置

查找字符串“abcoefhkhcldhcodoslc”中所有o出现的位置及次数

```javascript
var str = "abcoefhkhcldhcodoslc";
var index = str.indexOf('o');
var num = 0;
while(index !== -1) {
  console.log(index);
  num++;
  index = str.indexOf('o',index+1);
}
console.log("o出现的次数是：", num);
```

判断一个字符串“abcoefhkhcldhcodoslc”中出现次数最多的字符，并统计其次数。

```javascript
var str = "abcoefhkhcldhcodoslc";
var o = {}
for(let i = 0; i < str.length; i++) {
  var chars = str.charAt(i);
  if(o[chars]){
    o[chars]++;
  } else {
    o[chars] = 1;
  }
}
console.log(o);

var max = 0;
var ch = '';
for (var k in o) {
  if(o[k] > max) {
    max = o[k];
    ch = k;
  }
}
console.log(max);
console.log('最多的字符是'+ch);
```

# 常用正则表达式

```javascript
//（1）匹配 16 进制颜色值
var color = /#([0-9a-fA-F]{6}|[0-9a-fA-F]{3})/g;

//（2）匹配日期，如 yyyy-mm-dd 格式
var date = /^[0-9]{4}-(0[1-9]|1[0-2])-(0[1-9]|[12][0-9]|3[01])$/;

//（3）匹配 qq 号
var qq = /^[1-9][0-9]{4,10}$/g;

//（4）手机号码正则
var phone = /^1[34578]\d{9}$/g;

//（5）用户名正则
var username = /^[a-zA-Z\$][a-zA-Z0-9_\$]{4,16}$/;

//（6）Email正则
var email = /^([A-Za-z0-9_\-\.])+\@([A-Za-z0-9_\-\.])+\.([A-Za-z]{2,4})$/;

//（7）身份证号（18位）正则
var cP = /^[1-9]\d{5}(18|19|([23]\d))\d{2}((0[1-9])|(10|11|12))(([0-2][1-9])|10|20|30|31)\d{3}[0-9Xx]$/;

//（8）URL正则
var urlP= /^((https?|ftp|file):\/\/)?([\da-z\.-]+)\.([a-z\.]{2,6})([\/\w \.-]*)*\/?$/;

// (9)ipv4地址正则
var ipP = /^(?:(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)\.){3}(?:25[0-5]|2[0-4][0-9]|[01]?[0-9][0-9]?)$/;

// (10)//车牌号正则
var cPattern = /^[京津沪渝冀豫云辽黑湘皖鲁新苏浙赣鄂桂甘晋蒙陕吉闽贵粤青藏川宁琼使领A-Z]{1}[A-Z]{1}[A-Z0-9]{4}[A-Z0-9挂学警港澳]{1}$/;

// (11)强密码(必须包含大小写字母和数字的组合，不能使用特殊字符，长度在8-10之间)：var pwd = /^(?=.\d)(?=.[a-z])(?=.[A-Z]).{8,10}$/

```

# 深浅拷贝

## 什么是拷贝？

首先来直观的感受一下什么是拷贝。

```javascript
let arr = [1, 2, 3];
let newArr = arr;
newArr[0] = 100;

console.log(arr);//[100, 2, 3]
```

这是直接赋值的情况，不涉及任何拷贝。当改变newArr的时候，由于是同一个引用，arr指向的值也跟着改变。

## 浅拷贝

对内存地址的复制，让目标对象指针和源对象指向同一片内存空间。注意：当内存销毁的时候，只想对象的指针，必须重新定义，才能够使用。

### Object.assign

```js
let obj = { name: 'orange', age: 18 };
const obj2 = Object.assign({}, obj, {name: 'sss'});
console.log(obj2);//{ name: 'sss', age: 18 }
```

### 扩展运算符

```javascript
let arr = [1, 2, 3];
let newArr = [...arr];//跟arr.slice()是一样的效果
```

### concat浅拷贝数组

```javascript
let arr = [1, 2, 3];
let newArr = arr.concat();
newArr[1] = 100;
console.log(arr);//[ 1, 2, 3 ]
```

### slice浅拷贝

```javascript
let arr = [1, 2, {val: 4}];
let newArr = arr.slice();
newArr[2].val = 1000;

console.log(arr);//[ 1, 2, { val: 1000 } ]
```

## 深拷贝

深拷贝是指，拷贝对象的具体内容，二内存地址是自主分配的，拷贝结束之后俩个对象虽然存的值是一样的，但是内存地址不一样，俩个对象页互相不影响，互不干涉。

通常浅拷贝就能解决大部分问题了，但是当我们遇到如下情况就可能需要使用到深拷贝了。

```javascript
let a = {
  age: 1,
  jobs: {
    first: 'FE'
  }
}
let b = { ...a }
a.jobs.first = 'native'
console.log(b.jobs.first) // native
```

浅拷贝只解决了第一层的问题，如果接下去的值中还有对象的话，那么就又回到最开始的话题了，两者享有相同的地址。要解决这个问题，我们就得使用深拷贝了。

`JSON.parse(JSON.stringify());`

```javascript
let a = {
  age: 1,
  jobs: {
    first: 'FE'
  }
}
let b = JSON.parse(JSON.stringify(a))
a.jobs.first = 'native'
console.log(b.jobs.first) // FE
```

但是该方法也是有局限性的：

- 会忽略 `undefined`
- 会忽略 `symbol`
- 不能序列化函数
- 不能解决循环引用的对象

### 简易版深拷贝

```javascript
const deepClone = (target) => {
  if (typeof target === 'object' && target !== null) {
    const cloneTarget = Array.isArray(target) ? []: {};
    for (let prop in target) {
      if (target.hasOwnProperty(prop)) {
          cloneTarget[prop] = deepClone(target[prop]);
      }
    }
    return cloneTarget;
  } else {
    return target;
  }
}

let obj = {
  a: [1, 2, 3],
  b: {
    c: 2,
    d: 3
  }
}
let newObj = deepClone(obj)
newObj.b.c = 1
console.log(obj.b.c) // 2
```

#### 深复制 深度优先遍历

```javascript
let DFSdeepClone = (obj, visitedArr = []) => {
  let _obj = {}
  if (isTypeOf(obj, 'array') || isTypeOf(obj, 'object')) {
    let index = visitedArr.indexOf(obj)
    _obj = isTypeOf(obj, 'array') ? [] : {}
    if (~index) { // 判断环状数据
      _obj = visitedArr[index]
    } else {
      visitedArr.push(obj)
      for (let item in obj) {
        _obj[item] = DFSdeepClone(obj[item], visitedArr)
      }
    }
  } else if (isTypeOf(obj, 'function')) {
    _obj = eval('(' + obj.toString() + ')');
  } else {
    _obj = obj
  }
  return _obj
}
```

#### 广度优先遍历

```javascript
let BFSdeepClone = (obj) => {
    let origin = [obj],
      copyObj = {},
      copy = [copyObj]
      // 去除环状数据
    let visitedQueue = [],
      visitedCopyQueue = []
    while (origin.length > 0) {
      let items = origin.shift(),
        _obj = copy.shift()
      visitedQueue.push(items)
      if (isTypeOf(items, 'object') || isTypeOf(items, 'array')) {
        for (let item in items) {
          let val = items[item]
          if (isTypeOf(val, 'object')) {
            let index = visitedQueue.indexOf(val)
            if (!~index) {
              _obj[item] = {}
                //下次while循环使用给空对象提供数据
              origin.push(val)
                // 推入引用对象
              copy.push(_obj[item])
            } else {
              _obj[item] = visitedCopyQueue[index]
              visitedQueue.push(_obj)
            }
          } else if (isTypeOf(val, 'array')) {
            // 数组类型在这里创建了一个空数组
            _obj[item] = []
            origin.push(val)
            copy.push(_obj[item])
          } else if (isTypeOf(val, 'function')) {
            _obj[item] = eval('(' + val.toString() + ')');
          } else {
            _obj[item] = val
          }
        }
        // 将已经处理过的对象数据推入数组 给环状数据使用
        visitedCopyQueue.push(_obj)
      } else if (isTypeOf(items, 'function')) {
        copyObj = eval('(' + items.toString() + ')');
      } else {
        copyObj = obj
      }
    }
  return copyObj
}
```

# JS异步编程

## 并发（concurrency）和并行（parallelism）区别

> 涉及面试题：并发与并行的区别？

异步和这小节的知识点其实并不是一个概念，但是这两个名词确实是很多人都常会混淆的知识点。其实混淆的原因可能只是两个名词在中文上的相似，在英文上来说完全是不同的单词。

并发是宏观概念，我分别有任务 A 和任务 B，在一段时间内通过任务间的切换完成了这两个任务，这种情况就可以称之为并发。

并行是微观概念，假设 CPU 中存在两个核心，那么我就可以同时完成任务 A、B。同时完成多个任务的情况就可以称之为并行。

## 回调函数（Callback）

> 涉及面试题：什么是回调函数？回调函数有什么缺点？如何解决回调地狱问题？

回调函数应该是大家经常使用到的，以下代码就是一个回调函数的例子：

```javascript
ajax(url, () => {
    // 处理逻辑
})
```

但是回调函数有一个致命的弱点，就是容易写出回调地狱（Callback hell）。假设多个请求存在依赖性，你可能就会写出如下代码：

```javascript
ajax(url, () => {
    // 处理逻辑
    ajax(url1, () => {
        // 处理逻辑
        ajax(url2, () => {
            // 处理逻辑
        })
    })
})
```

以上代码看起来不利于阅读和维护，当然，你可能会想说解决这个问题还不简单，把函数分开来写不就得了

```javascript
function firstAjax() {
  ajax(url1, () => {
    // 处理逻辑
    secondAjax()
  })
}
function secondAjax() {
  ajax(url2, () => {
    // 处理逻辑
  })
}
ajax(url, () => {
  // 处理逻辑
  firstAjax()
})
```

以上的代码虽然看上去利于阅读了，但是还是没有解决根本问题。

回调地狱的根本问题就是：

1. 嵌套函数存在耦合性，一旦有所改动，就会牵一发而动全身
2. 嵌套函数一多，就很难处理错误

当然，回调函数还存在着别的几个缺点，比如不能使用 `try catch` 捕获错误，不能直接 `return`。在接下来的几小节中，我们将来学习通过别的技术解决这些问题。

## Generator

> 涉及面试题：你理解的 Generator 是什么？

`Generator` 算是 ES6 中难理解的概念之一了，`Generator` 最大的特点就是可以控制函数的执行。在这一小节中我们不会去讲什么是 `Generator`，而是把重点放在 `Generator` 的一些容易困惑的地方。

```javascript
function *foo(x) {
  let y = 2 * (yield (x + 1))
  let z = yield (y / 3)
  return (x + y + z)
}
let it = foo(5)
console.log(it.next())   // => {value: 6, done: false}
console.log(it.next(12)) // => {value: 8, done: false}
console.log(it.next(13)) // => {value: 42, done: true}
```

你也许会疑惑为什么会产生与你预想不同的值，接下来就让我为你逐行代码分析原因

- 首先 `Generator` 函数调用和普通函数不同，它会返回一个迭代器
- 当执行第一次 `next` 时，传参会被忽略，并且函数暂停在 `yield (x + 1)` 处，所以返回 `5 + 1 = 6`
- 当执行第二次 `next` 时，传入的参数等于上一个 `yield` 的返回值，如果你不传参，`yield` 永远返回 `undefined`。此时 `let y = 2 * 12`，所以第二个 `yield` 等于 `2 * 12 / 3 = 8`
- 当执行第三次 `next` 时，传入的参数会传递给 `z`，所以 `z = 13, x = 5, y = 24`，相加等于 `42`

`Generator` 函数一般见到的不多，其实也于他有点绕有关系，并且一般会配合 co 库去使用。当然，我们可以通过 `Generator` 函数解决回调地狱的问题，可以把之前的回调地狱例子改写为如下代码：

```javascript
function *fetch() {
    yield ajax(url, () => {})
    yield ajax(url1, () => {})
    yield ajax(url2, () => {})
}
let it = fetch()
let result1 = it.next()
let result2 = it.next()
let result3 = it.next()
```

## Promise

> 涉及面试题：Promise 的特点是什么，分别有什么优缺点？什么是 Promise 链？Promise 构造函数执行和 then 函数执行有什么区别？

`Promise` 翻译过来就是承诺的意思，这个承诺会在未来有一个确切的答复，并且该承诺有三种状态，分别是：

1. 等待中（pending）
2. 完成了 （resolved）
3. 拒绝了（rejected）

这个承诺一旦从等待状态变成为其他状态就永远不能更改状态了，也就是说一旦状态变为 resolved 后，就不能再次改变

```javascript
new Promise((resolve, reject) => {
  resolve('success')
  // 无效
  reject('reject')
})
```

当我们在构造 `Promise` 的时候，构造函数内部的代码是立即执行的

```
new Promise((resolve, reject) => {
  console.log('new Promise')
  resolve('success')
})
console.log('finifsh')
// new Promise -> finifsh
```

`Promise` 实现了链式调用，也就是说每次调用 `then` 之后返回的都是一个 `Promise`，并且是一个全新的 `Promise`，原因也是因为状态不可变。如果你在 `then` 中 使用了 `return`，那么 `return` 的值会被 `Promise.resolve()` 包装

```javascript
Promise.resolve(1)
  .then(res => {
    console.log(res) // => 1
    return 2 // 包装成 Promise.resolve(2)
  })
  .then(res => {
    console.log(res) // => 2
  })
```

当然了，`Promise` 也很好地解决了回调地狱的问题，可以把之前的回调地狱例子改写为如下代码：

```javascript
ajax(url)
  .then(res => {
      console.log(res)
      return ajax(url1)
  }).then(res => {
      console.log(res)
      return ajax(url2)
  }).then(res => console.log(res))
```

前面都是在讲述 `Promise` 的一些优点和特点，其实它也是存在一些缺点的，比如无法取消 `Promise`，错误需要通过回调函数捕获。

## async 及 await

> 涉及面试题：async 及 await 的特点，它们的优点和缺点分别是什么？await 原理是什么？

一个函数如果加上 `async` ，那么该函数就会返回一个 `Promise`

```javascript
async function test() {
  return "1"
}
console.log(test()) // -> Promise {<resolved>: "1"}
```

`async` 就是将函数返回值使用 `Promise.resolve()` 包裹了下，和 `then` 中处理返回值一样，并且 `await` 只能配套 `async` 使用

```javascript
async function test() {
  let value = await sleep()
}
```

`async` 和 `await` 可以说是异步终极解决方案了，相比直接使用 `Promise` 来说，优势在于处理 `then` 的调用链，能够更清晰准确的写出代码，毕竟写一大堆 `then` 也很恶心，并且也能优雅地解决回调地狱问题。当然也存在一些缺点，因为 `await` 将异步代码改造成了同步代码，如果多个异步代码没有依赖性却使用了 `await` 会导致性能上的降低。

```javascript
async function test() {
  // 以下代码没有依赖性的话，完全可以使用 Promise.all 的方式
  // 如果有依赖性的话，其实就是解决回调地狱的例子了
  await fetch(url)
  await fetch(url1)
  await fetch(url2)
}
```

下面来看一个使用 `await` 的例子：

```javascript
let a = 0
let b = async () => {
  a = a + await 10
  console.log('2', a) // -> '2' 10
}
b()
a++
console.log('1', a) // -> '1' 1
```

对于以上代码你可能会有疑惑，让我来解释下原因

- 首先函数 `b` 先执行，在执行到 `await 10` 之前变量 `a` 还是 0，因为 `await` 内部实现了 `generator` ，`generator` 会保留堆栈中东西，所以这时候 `a = 0` 被保存了下来
- 因为 `await` 是异步操作，后来的表达式不返回 `Promise` 的话，就会包装成 `Promise.reslove(返回值)`，然后会去执行函数外的同步代码
- 同步代码执行完毕后开始执行异步代码，将保存下来的值拿出来使用，这时候 `a = 0 + 10`

上述解释中提到了 `await` 内部实现了 `generator`，其实 `await` 就是 `generator` 加上 `Promise` 的语法糖，且内部实现了自动执行 `generator`。如果你熟悉 co 的话，其实自己就可以实现这样的语法糖。

## 常用定时器函数

> 涉及面试题：setTimeout、setInterval、requestAnimationFrame 各有什么特点？

异步编程当然少不了定时器了，常见的定时器函数有 `setTimeout`、`setInterval`、`requestAnimationFrame`。我们先来讲讲最常用的`setTimeout`，很多人认为 `setTimeout` 是延时多久，那就应该是多久后执行。

其实这个观点是错误的，因为 JS 是单线程执行的，如果前面的代码影响了性能，就会导致 `setTimeout` 不会按期执行。当然了，我们可以通过代码去修正 `setTimeout`，从而使定时器相对准确

```javascript
let period = 60 * 1000 * 60 * 2
let startTime = new Date().getTime()
let count = 0
let end = new Date().getTime() + period
let interval = 1000
let currentInterval = interval

function loop() {
  count++
  // 代码执行所消耗的时间
  let offset = new Date().getTime() - (startTime + count * interval);
  let diff = end - new Date().getTime()
  let h = Math.floor(diff / (60 * 1000 * 60))
  let hdiff = diff % (60 * 1000 * 60)
  let m = Math.floor(hdiff / (60 * 1000))
  let mdiff = hdiff % (60 * 1000)
  let s = mdiff / (1000)
  let sCeil = Math.ceil(s)
  let sFloor = Math.floor(s)
  // 得到下一次循环所消耗的时间
  currentInterval = interval - offset 
  console.log('时：'+h, '分：'+m, '毫秒：'+s, '秒向上取整：'+sCeil, '代码执行时间：'+offset, '下次循环间隔'+currentInterval) 

  setTimeout(loop, currentInterval)
}

setTimeout(loop, currentInterval)
```

接下来我们来看 `setInterval`，其实这个函数作用和 `setTimeout` 基本一致，只是该函数是每隔一段时间执行一次回调函数。

通常来说不建议使用 `setInterval`。第一，它和 `setTimeout` 一样，不能保证在预期的时间执行任务。第二，它存在执行累积的问题，请看以下伪代码

```javascript
function demo() {
  setInterval(function(){
    console.log(2)
  },1000)
  sleep(2000)
}
demo()
```

以上代码在浏览器环境中，如果定时器执行过程中出现了耗时操作，多个回调函数会在耗时操作结束以后同时执行，这样可能就会带来性能上的问题。

如果你有循环定时器的需求，其实完全可以通过 `requestAnimationFrame` 来实现

```javascript
function setInterval(callback, interval) {
  let timer
  const now = Date.now
  let startTime = now()
  let endTime = startTime
  const loop = () => {
    timer = window.requestAnimationFrame(loop)
    endTime = now()
    if (endTime - startTime >= interval) {
      startTime = endTime = now()
      callback(timer)
    }
  }
  timer = window.requestAnimationFrame(loop)
  return timer
}

let a = 0
setInterval(timer => {
  console.log(1)
  a++
  if (a === 3) cancelAnimationFrame(timer)
}, 1000)
```

首先 `requestAnimationFrame` 自带函数节流功能，基本可以保证在 16.6 毫秒内只执行一次（不掉帧的情况下），并且该函数的延时效果是精确的，没有其他定时器时间不准的问题，当然你也可以通过该函数来实现 `setTimeout`。

## sleep

```javascript
function sleep (time) {
  return new Promise((resolve) => setTimeout(resolve, time));
}
 
// 用法
sleep(500).then(() => {
    // 这里写sleep之后需要去做的事情
})
```

```javascript
function sleep (time) {
  return new Promise((resolve) => setTimeout(resolve, time));
}
 
(async function() {
  console.log('Do some thing, ' + new Date());
  await sleep(3000);
  console.log('Do other things, ' + new Date());
})();
```



# promise

## promise如何解决回掉地狱问题

**问题**

首先，什么是回调地狱:

1. 多层嵌套的问题。
2. 每种任务的处理结果存在两种可能性（成功或失败），那么需要在每种任务执行结束后分别处理这两种可能性。

这两种问题在回调函数时代尤为突出。Promise 的诞生就是为了解决这两个问题。

**解决方法**

Promise 利用了三大技术手段来解决`回调地狱`:

- **回调函数延迟绑定**。
- **返回值穿透**。
- **错误冒泡**。

首先来举个例子:

```javascript
let readFilePromise = (filename) => {
  fs.readFile(filename, (err, data) => {
    if(err) {
      reject(err);
    }else {
      resolve(data);
    }
  })
}
readFilePromise('1.json').then(data => {
  return readFilePromise('2.json')
});
```

看到没有，回调函数不是直接声明的，而是在通过后面的 then 方法传入的，即延迟传入。这就是`回调函数延迟绑定`。

然后我们做以下微调:

```javascript
let x = readFilePromise('1.json').then(data => {
  return readFilePromise('2.json')//这是返回的Promise
});
x.then(/* 内部逻辑省略 */)
```

我们会根据 then 中回调函数的传入值创建不同类型的Promise, 然后把返回的 Promise 穿透到外层, 以供后续的调用。这里的 x 指的就是内部返回的 Promise，然后在 x 后面可以依次完成链式调用。

这便是`返回值穿透`的效果。

这两种技术一起作用便可以将深层的嵌套回调写成下面的形式:

```javascript
readFilePromise('1.json').then(data => {
    return readFilePromise('2.json');
}).then(data => {
    return readFilePromise('3.json');
}).then(data => {
    return readFilePromise('4.json');
});
```

这样就显得清爽了许多，更重要的是，它更符合人的线性思维模式，开发体验也更好。

两种技术结合产生了`链式调用`的效果。

这解决的是多层嵌套的问题，那另一个问题，即每次任务执行结束后`分别处理成功和失败`的情况怎么解决的呢？

Promise 采用了`错误冒泡`的方式。其实很简单理解，我们来看看效果:

```javascript
readFilePromise('1.json').then(data => {
    return readFilePromise('2.json');
}).then(data => {
    return readFilePromise('3.json');
}).then(data => {
    return readFilePromise('4.json');
}).catch(err => {
  // xxx
})
```

这样前面产生的错误会一直向后传递，被 catch 接收到，就不用频繁地检查错误了。

**解决效果**

- 1. 实现链式调用，解决多层嵌套问题
- 1. 实现错误冒泡后一站式处理，解决每次任务中判断错误、增加代码混乱度的问题

## promise引入微任务的原因

Promise 中的执行函数是同步进行的，但是里面存在着异步操作，在异步操作结束后会调用 resolve 方法，或者中途遇到错误调用 reject 方法，这两者都是作为微任务进入到 EventLoop 中。但是你有没有想过，Promise 为什么要引入微任务的方式来进行回调操作？

**解决方式**

回到问题本身，其实就是如何处理回调的问题。总结起来有三种方式:

1. 使用同步回调，直到异步任务进行完，再进行后面的任务。
2. 使用异步回调，将回调函数放在进行`宏任务队列`的队尾。
3. 使用异步回调，将回调函数放到`当前宏任务中`的最后面。

**优劣对比**

第一种方式显然不可取，因为同步的问题非常明显，会让整个脚本阻塞住，当前任务等待，后面的任务都无法得到执行，而这部分`等待的时间`是可以拿来完成其他事情的，导致 CPU 的利用率非常低，而且还有另外一个致命的问题，就是无法实现`延迟绑定`的效果。

如果采用第二种方式，那么执行回调(resolve/reject)的时机应该是在前面`所有的宏任务`完成之后，倘若现在的任务队列非常长，那么回调迟迟得不到执行，造成`应用卡顿`。

为了解决上述方案的问题，另外也考虑到`延迟绑定`的需求，Promise 采取第三种方式, 即`引入微任务`, 即把 resolve(reject) 回调的执行放在当前宏任务的末尾。

这样，利用`微任务`解决了两大痛点:

- 1. 采用**异步回调**替代同步回调解决了浪费 CPU 性能的问题。
- 1. 放到**当前宏任务最后**执行，解决了回调执行的实时性问题。

好，Promise 的基本实现思想已经讲清楚了，相信大家已经知道了它`为什么这么设计`，接下来就让我们一步步弄清楚它内部到底是`怎么设计的`。

## 手写promise

第一版promise

```javascript
// 首先定义三种状态
const PENDING = "pending";
const FULFILLED = "fulfilled";
const REJECTED = "rejected";

function myPromise(executor) {
    let self  = this; //缓存当前promise实例
    self.value = null;
    self.error = null;
    self.status = PENDING;
    self.onFulfilled = null; //成功的回调函数
    self.onRejected = null; //失败的回调函数

    const resolve = (value) => {
        if(self.status !== PENDING) return;
        setTimeout(()=> {
            self.status = FULFILLED;
            self.value = value;
            self.onFulfilled(self.value)
        })
    }

    const reject = (error) => {
        if(self.status !== PENDING) return;
        setTimeout(() => {
            self.status = REJECTED;
            self.error = error;
            self.onRejected(self.error);
        })
    }

    executor(resolve, reject)
    
    myPromise.prototype.then = function(onFulfilled, onRejected) {
        if(this.status === PENDING) {
            this.onFulfilled = onFulfilled;
            this.onRejected = onRejected;
        }else if(this.status === FULFILLED) {
            onFulfilled(this.value)
        }else{
            onRejected(this.error)
        }
        return this;
    }
}
```

**A+版本promise**

https://juejin.im/post/5c88e427f265da2d8d6a1c84#heading-0

```javascript
const PENDING = 'pending';
const FULFILLED = 'fulfilled';
const REJECTED = 'rejected';
function Promise(executor) {
    let self = this;
    self.status = PENDING;
    self.onFulfilled = [];//成功的回调
    self.onRejected = []; //失败的回调
    //PromiseA+ 2.1
    function resolve(value) {
        if (self.status === PENDING) {
            self.status = FULFILLED;
            self.value = value;
            self.onFulfilled.forEach(fn => fn());//PromiseA+ 2.2.6.1
        }
    }

    function reject(reason) {
        if (self.status === PENDING) {
            self.status = REJECTED;
            self.reason = reason;
            self.onRejected.forEach(fn => fn());//PromiseA+ 2.2.6.2
        }
    }

    try {
        executor(resolve, reject);
    } catch (e) {
        reject(e);
    }
}

Promise.prototype.then = function (onFulfilled, onRejected) {
    //PromiseA+ 2.2.1 / PromiseA+ 2.2.5 / PromiseA+ 2.2.7.3 / PromiseA+ 2.2.7.4
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : reason => { throw reason };
    let self = this;
    //PromiseA+ 2.2.7
    let promise2 = new Promise((resolve, reject) => {
        if (self.status === FULFILLED) {
            //PromiseA+ 2.2.2
            //PromiseA+ 2.2.4 --- setTimeout
            setTimeout(() => {
                try {
                    //PromiseA+ 2.2.7.1
                    let x = onFulfilled(self.value);
                    resolvePromise(promise2, x, resolve, reject);
                } catch (e) {
                    //PromiseA+ 2.2.7.2
                    reject(e);
                }
            });
        } else if (self.status === REJECTED) {
            //PromiseA+ 2.2.3
            setTimeout(() => {
                try {
                    let x = onRejected(self.reason);
                    resolvePromise(promise2, x, resolve, reject);
                } catch (e) {
                    reject(e);
                }
            });
        } else if (self.status === PENDING) {
            self.onFulfilled.push(() => {
                setTimeout(() => {
                    try {
                        let x = onFulfilled(self.value);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                });
            });
            self.onRejected.push(() => {
                setTimeout(() => {
                    try {
                        let x = onRejected(self.reason);
                        resolvePromise(promise2, x, resolve, reject);
                    } catch (e) {
                        reject(e);
                    }
                });
            });
        }
    });
    return promise2;
}
```

## promise小例子

```javascript
const promise = new Promise((resolve, reject) => {
    setTimeout(reject,1000,'我是value值');
})

promise.then((value) => {
    console.log('resolve:' + value)
}).catch((value) => {
    console.log('reject:'+ value)
})
```

```javascript
class Promise{
  constructor(executor){
    this.state = 'pending';
    this.value = undefined;
    this.reason = undefined;
    this.onResolvedCallbacks = [];
    this.onRejectedCallbacks = [];
    let resolve = value => {
      if (this.state === 'pending') {
        this.state = 'fulfilled';
        this.value = value;
        this.onResolvedCallbacks.forEach(fn=>fn());
      }
    };
    let reject = reason => {
      if (this.state === 'pending') {
        this.state = 'rejected';
        this.reason = reason;
        this.onRejectedCallbacks.forEach(fn=>fn());
      }
    };
    try{
      executor(resolve, reject);
    } catch (err) {
      reject(err);
    }
  }
  then(onFulfilled,onRejected) {
    onFulfilled = typeof onFulfilled === 'function' ? onFulfilled : value => value;
    onRejected = typeof onRejected === 'function' ? onRejected : err => { throw err };
    let promise2 = new Promise((resolve, reject) => {
      if (this.state === 'fulfilled') {
        setTimeout(() => {
          try {
            let x = onFulfilled(this.value);
            resolvePromise(promise2, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        }, 0);
      };
      if (this.state === 'rejected') {
        setTimeout(() => {
          try {
            let x = onRejected(this.reason);
            resolvePromise(promise2, x, resolve, reject);
          } catch (e) {
            reject(e);
          }
        }, 0);
      };
      if (this.state === 'pending') {
        this.onResolvedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onFulfilled(this.value);
              resolvePromise(promise2, x, resolve, reject);
            } catch (e) {
              reject(e);
            }
          }, 0);
        });
        this.onRejectedCallbacks.push(() => {
          setTimeout(() => {
            try {
              let x = onRejected(this.reason);
              resolvePromise(promise2, x, resolve, reject);
            } catch (e) {
              reject(e);
            }
          }, 0)
        });
      };
    });
    return promise2;
  }
  catch(fn){
    return this.then(null,fn);
  }
}
function resolvePromise(promise2, x, resolve, reject){
  if(x === promise2){
    return reject(new TypeError('Chaining cycle detected for promise'));
  }
  let called;
  if (x != null && (typeof x === 'object' || typeof x === 'function')) {
    try {
      let then = x.then;
      if (typeof then === 'function') { 
        then.call(x, y => {
          if(called)return;
          called = true;
          resolvePromise(promise2, y, resolve, reject);
        }, err => {
          if(called)return;
          called = true;
          reject(err);
        })
      } else {
        resolve(x);
      }
    } catch (e) {
      if(called)return;
      called = true;
      reject(e); 
    }
  } else {
    resolve(x);
  }
}
//resolve方法
Promise.resolve = function(val){
  return new Promise((resolve,reject)=>{
    resolve(val)
  });
}
//reject方法
Promise.reject = function(val){
  return new Promise((resolve,reject)=>{
    reject(val)
  });
}
//race方法 
Promise.race = function(promises){
  return new Promise((resolve,reject)=>{
    for(let i=0;i<promises.length;i++){
      promises[i].then(resolve,reject)
    };
  })
}
//all方法(获取所有的promise，都执行then，把结果放到数组，一起返回)
Promise.all = function(promises){
  let arr = [];
  let i = 0;
  function processData(index,data){
    arr[index] = data;
    i++;
    if(i == promises.length){
      resolve(arr);
    };
  };
  return new Promise((resolve,reject)=>{
    for(let i=0;i<promises.length;i++){
      promises[i].then(data=>{
        processData(i,data);
      },reject);
    };
  });
}

```

# 函数柯里化

柯里化（Currying）是函数式编程的一个很重要的概念，将使用多个参数的一个函数转换成一系列使用一个参数的函数

主要有三个作用：1. 参数复用**；**2. 提前返回；3. 延迟计算/运行

```javascript
var curry = function (fn) {
    var args = [].slice.call(arguments, 1);
    return function() {
        var newArgs = args.concat([].slice.call(arguments));
        return fn.apply(this, newArgs);
    };
};
```

# JS运行机制

## js单线程

JavaScript的单线程，与它的用途有关。作为浏览器脚本语言，JavaScript的主要用途是与用户互动，以及操作DOM。这决定了它只能是单线程，否则会带来很复杂的同步问题。比如，假定JavaScript同时有两个线程，一个线程在某个DOM节点上添加内容，另一个线程删除了这个节点，这时浏览器应该以哪个线程为准？
所以，为了避免复杂性，从一诞生，JavaScript就是单线程，这已经成了这门语言的核心特征，将来也不会改变。

## js事件循环

js代码执行过程中会有很多任务，这些任务总的分成两类：

- 同步任务
- 异步任务

当我们打开网站时，网页的渲染过程就是一大堆同步任务，比如页面骨架和页面元素的渲染。而像加载图片音乐之类占用资源大耗时久的任务，就是异步任务。

# JS延迟加载

JS 的作用在于**修改**，它帮助我们修改网页的方方面面：内容、样式以及它如何响应用户交互。这“方方面面”的修改，本质上都是对 DOM 和 CSSDOM 进行修改。因此 JS 的执行会阻止 CSSOM，在我们不作显式声明的情况下，它也会阻塞 DOM。

我们通过一个🌰来理解一下这个机制：

```html
<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>JS阻塞测试</title>
  <style>
    #container {
      background-color: yellow;
      width: 100px;
      height: 100px;
    }
  </style>
  <script>
    // 尝试获取container元素
    var container = document.getElementById("container")
    console.log('container', container)
  </script>
</head>
<body>
  <div id="container"></div>
  <script>
    // 尝试获取container元素
    var container = document.getElementById("container")
    console.log('container', container)
    // 输出container元素此刻的背景色
    console.log('container bgColor', getComputedStyle(container).backgroundColor)
  </script>
  <style>
    #container {
      background-color: blue;
    }
  </style>
</body>
</html>
```

三个 console 的结果分别为：

![img](https://user-gold-cdn.xitu.io/2018/9/28/166203a2d62212c9?imageslim)

注：本例仅使用了内联 JS 做测试。感兴趣的同学可以把这部分 JS 当做外部文件引入看看效果——它们的表现一致。

第一次尝试获取 id 为 container 的 DOM 失败，这说明 JS 执行时阻塞了 DOM，后续的 DOM 无法构建；第二次才成功，这说明脚本块只能找到在它前面构建好的元素。这两者结合起来，“阻塞 DOM”得到了验证。再看第三个 console，尝试获取 CSS 样式，获取到的是在 JS 代码执行前的背景色（yellow），而非后续设定的新样式（blue），说明 CSSOM 也被阻塞了。那么在阻塞的背后，到底发生了什么呢？

我们前面说过，**JS 引擎是独立于渲染引擎存在的**。我们的 JS 代码在文档的何处插入，就在何处执行。当 HTML 解析器遇到一个 script 标签时，它会暂停渲染过程，将控制权交给 JS 引擎。JS 引擎对内联的 JS 代码会直接执行，对外部 JS 文件还要先获取到脚本、再进行执行。等 JS 引擎运行完毕，浏览器又会把控制权还给渲染引擎，继续 CSSOM 和 DOM 的构建。 因此与其说是 JS 把 CSS 和 HTML 阻塞了，不如说是 JS 引擎抢走了渲染引擎的控制权。

现在理解了阻塞的表现与原理，我们开始思考一个问题。浏览器之所以让 JS 阻塞其它的活动，是因为它不知道 JS 会做什么改变，担心如果不阻止后续的操作，会造成混乱。但是我们是写 JS 的人，我们知道 JS 会做什么改变。假如我们可以确认一个 JS 文件的执行时机并不一定非要是此时此刻，我们就可以通过对它使用 defer 和 async 来避免不必要的阻塞，这里我们就引出了外部 JS 的三种加载方式。

### JS的三种加载方式

- 正常模式：

  ```
  <script src="index.js"></script>
  ```

这种情况下 JS 会阻塞浏览器，浏览器必须等待 index.js 加载和执行完毕才能去做其它事情。

- async 模式：

  ```
  <script async src="index.js"></script>
  ```

async 模式下，JS 不会阻塞浏览器做任何其它的事情。它的加载是异步的，当它加载结束，JS 脚本会**立即执行**。

- defer 模式：

  ```
  <script defer src="index.js"></script>
  ```

defer 模式下，JS 的加载是异步的，执行是**被推迟的**。等整个文档解析完成、DOMContentLoaded 事件即将被触发时，被标记了 defer 的 JS 文件才会开始依次执行。

从应用的角度来说，一般当我们的脚本与 DOM 元素和其它脚本之间的依赖关系不强时，我们会选用 async；当脚本依赖于 DOM 元素和其它脚本的执行结果时，我们会选用 defer。

通过审时度势地向 script 标签添加 async/defer，我们就可以告诉浏览器在等待脚本可用期间不阻止其它的工作，这样可以显著提升性能。

js 的加载、解析和执行会阻塞页面的渲染过程，因此我们希望 js 脚本能够尽可能的延迟加载，提高页面的渲染速度。

1. 将 js 脚本放在文档的底部，来使 js 脚本尽可能的在最后来加载执行。
2. 给 js 脚本添加 defer属性，这个属性会让脚本的加载与文档的解析同步解析，然后在文档解析完成后再执行这个脚本文件，这样的话就能使页面的渲染不被阻塞。多个设置了 defer 属性的脚本按规范来说最后是顺序执行的，但是在一些浏览器中可能不是这样。
3. 给 js 脚本添加 async属性，这个属性会使脚本异步加载，不会阻塞页面的解析过程，但是当脚本加载完成后立即执行 js脚本，这个时候如果文档没有解析完成的话同样会阻塞。多个 async 属性的脚本的执行顺序是不可预测的，一般不会按照代码的顺序依次执行。
4. 动态创建 DOM 标签的方式，我们可以对文档的加载事件进行监听，当文档加载完成后再动态的创建 script 标签来引入 js 脚本。

![preview](https://segmentfault.com/img/bVWhRl?w=801&h=814/view)

# 如何实现私有变量（考察闭包、proxy特性）

# 函数的arguments为什么不是数组？如何转化成数组？

因为arguments本身并不能调用数组方法，它是一个另外一种对象类型，只不过属性从0开始排，依次为0，1，2...最后还有callee和length属性。我们也把这样的对象称为类数组。

常见的类数组还有：

- 1. 用getElementsByTagName/ClassName()获得的HTMLCollection
- 1. 用querySelector获得的nodeList

那这导致很多数组的方法就不能用了，必要时需要我们将它们转换成数组，有哪些方法呢？

### 1. Array.prototype.slice.call()

```javascript
function sum(a, b) {
  let args = Array.prototype.slice.call(arguments);
  console.log(args.reduce((sum, cur) => sum + cur));//args可以调用数组原生的方法啦
}
sum(1, 2);//3
```

### 2.	Array.from()

```javascript
function sum(a, b) {
  let args = Array.from(arguments);
  console.log(args.reduce((sum, cur) => sum + cur));//args可以调用数组原生的方法啦
}
sum(1, 2);//3
```

这种方法也可以用来转换Set和Map哦！

### 3. ES6展开运算符

```javascript
function sum(a, b) {
  let args = [...arguments];
  console.log(args.reduce((sum, cur) => sum + cur));//args可以调用数组原生的方法啦
}
sum(1, 2);//3
```

### 4. 利用concat+apply

```javascript
function sum(a, b) {
  let args = Array.prototype.concat.apply([], arguments);//apply方法会把第二个参数展开
  console.log(args.reduce((sum, cur) => sum + cur));//args可以调用数组原生的方法啦
}
sum(1, 2);//3
```

当然，最原始的方法就是再创建一个数组，用for循环把类数组的每个属性值放在里面，过于简单，就不浪费篇幅了。

# 模块化

模块化是指把一个复杂的系统分解到多个模块以方便编码。

很久以前，开发网页要通过命名空间的方式来组织代码，例如 jQuery 库把它的API都放在了 `window.$` 下，在加载完 jQuery 后其他模块再通过 `window.$` 去使用 jQuery。 这样做有很多问题，其中包括：

- 命名空间冲突，两个库可能会使用同一个名称，例如 [Zepto](http://zeptojs.com/) 也被放在 `window.$` 下；
- 无法合理地管理项目的依赖和版本；
- 无法方便地控制依赖的加载顺序。

当项目变大时这种方式将变得难以维护，需要用模块化的思想来组织代码。

### CommonJS

[CommonJS](http://www.commonjs.org/) 是一种使用广泛的 JavaScript 模块化规范，核心思想是通过 `require` 方法来同步地加载依赖的其他模块，通过 `module.exports` 导出需要暴露的接口。 CommonJS 规范的流行得益于 Node.js 采用了这种方式，后来这种方式被引入到了网页开发中。

采用 CommonJS 导入及导出时的代码如下：

```js
// 导入
const moduleA = require('./moduleA');

// 导出
module.exports = moduleA.someFunc;
```

CommonJS 的优点在于：

- 代码可复用于 Node.js 环境下并运行，例如做同构应用；
- 通过 NPM 发布的很多第三方模块都采用了 CommonJS 规范。

CommonJS 的缺点在于这样的代码无法直接运行在浏览器环境下，必须通过工具转换成标准的 ES5。

> CommonJS 还可以细分为 CommonJS1 和 CommonJS2，区别在于 CommonJS1 只能通过 `exports.XX = XX` 的方式导出，CommonJS2 在 CommonJS1 的基础上加入了 `module.exports = XX` 的导出方式。 CommonJS 通常指 CommonJS2。

### AMD

[AMD](https://en.wikipedia.org/wiki/Asynchronous_module_definition) 也是一种 JavaScript 模块化规范，与 CommonJS 最大的不同在于它采用异步的方式去加载依赖的模块。 AMD 规范主要是为了解决针对浏览器环境的模块化问题，最具代表性的实现是 [requirejs](http://requirejs.org/)。

采用 AMD 导入及导出时的代码如下：

```js
// 定义一个模块
define('module', ['dep'], function(dep) {
  return exports;
});

// 导入和使用
require(['module'], function(module) {
});
```

AMD 的优点在于：

- 可在不转换代码的情况下直接在浏览器中运行；
- 可异步加载依赖；
- 可并行加载多个依赖；
- 代码可运行在浏览器环境和 Node.js 环境下。

AMD 的缺点在于JavaScript 运行环境没有原生支持 AMD，需要先导入实现了 AMD 的库后才能正常使用。

### ES6 模块化

ES6 模块化是国际标准化组织 ECMA 提出的 JavaScript 模块化规范，它在语言的层面上实现了模块化。浏览器厂商和 Node.js 都宣布要原生支持该规范。它将逐渐取代 CommonJS 和 AMD 规范，成为浏览器和服务器通用的模块解决方案。

采用 ES6 模块化导入及导出时的代码如下：

```js
// 导入
import { readFile } from 'fs';
import React from 'react';
// 导出
export function hello() {};
export default {
  // ...
};
```

ES6模块虽然是终极模块化方案，但它的缺点在于目前无法直接运行在大部分 JavaScript 运行环境下，必须通过工具转换成标准的 ES5 后才能正常运行。

### 样式文件中的模块化

除了 JavaScript 开始模块化改造，前端开发里的样式文件也支持模块化。 以 SCSS 为例，把一些常用的样式片段放进一个通用的文件里，再在另一个文件里通过 `@import` 语句去导入和使用这些样式片段。

```scss
// util.scss 文件

// 定义样式片段
@mixin center {
  // 水平竖直居中
  position: absolute;
  left: 50%;
  top: 50%;
  transform: translate(-50%,-50%);
}

// main.scss 文件

// 导入和使用 util.scss 中定义的样式片段
@import "util";
#box{
  @include center;
}
```

# JS构造函数的几种方式

## 函数声明

```javascript
function sum1(num1,num2){
   return num1+num2;
}
sum1(10,20);
```

## 函数表达式

```javascript
var sum2 = function(num1,num2){
   return num1+num2;
}
sum(10,20);
```

**两者的区别：**解析器会先读取函数声明，并使其在执行任何代码之前可以访问；而函数表达式则必须等到解析器执行到它所在的代码行才会真正被解释执行。

自执行函数严格来说也叫函数表达式，它主要用于创建一个新的作用域，在此作用域内声明的变量，不会和其它作用域内的变量冲突或混淆，大多是以匿名函数方式存在，且立即自动执行。

## 函数构造法

```javascript
var sum3=new Function('n1','n2','return n1+n2');
console.log(sum3(2,3));//5
```

从技术角度讲，这是一个函数表达式。一般不推荐用这种方法定义函数，因为这种语法会导致解析两次代码（第一次是解析常规ECMAScript代码，第二次是解析传入构造函数中的字符串），从而影响性能。

# 箭头函数和普通函数的区别

**（1）箭头函数是匿名函数，不能作为构造函数，不能使用new。**

```javascript
let a = () => { console.log(111)} 
a()

let fn = new a()
VM325:1 Uncaught TypeError: a is not a constructor
    at <anonymous>:1:10
```

**（2）箭头函数不绑定arguments，取而代之用rest参数...解决**

```javascript
function A(a){ console.log(arguments)}
A(2,'sdas','asda')
Arguments(3) [2, "sdas", "asda", callee: ƒ, Symbol(Symbol.iterator): ƒ]


let B = (b)=>{
  console.log(arguments);
}
B(2,92,32,32);   // Uncaught ReferenceError: arguments is not defined


let C = (...c) => {
  console.log(c);
}
C(3,82,32,11323);  // [3, 82, 32, 11323]
```

**（3）this的作用域不同，箭头函数不绑定this，会捕获其所在的上下文的this值，作为自己的this值。**

```javascript
var obj = {
  a: 10,
  b: () => {
    console.log(this.a); // undefined
    console.log(this); // Window {postMessage: ƒ, blur: ƒ, focus: ƒ, close: ƒ, frames: Window, …}
  },
  c: function() {
    console.log(this.a); // 10
    console.log(this); // {a: 10, b: ƒ, c: ƒ}
  }
}
obj.b(); 
obj.c();
```

**（4）箭头函数没有原型属性**

```javascript
var a = ()=>{
  return 1;
}

function b(){
  return 2;
}

console.log(a.prototype);  // undefined
console.log(b.prototype);   // {constructor: ƒ}
```

**（5）箭头函数不能当做Generator函数,不能使用yield关键字**

# for of和for in

**一句话概括：for in是遍历（object）键名，for of是遍历（array）键值。**

## for...in

`for...in` 循环只遍历可枚举属性（包括它的原型链上的可枚举属性）。像 `Array`和`Object`使用内置构造函数所创建的对象都会继承自`Object.prototype`和`String.prototype`的不可枚举属性，例如 [`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String) 的 [`indexOf()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/String/indexOf) 方法或 [`Object`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object)的[`toString()`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Object/toString)方法。循环将遍历对象本身的所有可枚举属性，以及对象从其构造函数原型中继承的属性（更接近原型链中对象的属性覆盖原型属性）。

```javascript
var obj = {a:1, b:2, c:3};
    
for (let key in obj) {
  console.log(key);
}

// a
// b
// c
```

## for...of

**`for...of`语句**在[可迭代对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/iterable)（包括[`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Array)，[`Map`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Map)，[`Set`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/Set)，[`String`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/String)，[`TypedArray`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Global_Objects/TypedArray)，[arguments](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions_and_function_scope/arguments) 对象等等）上创建一个迭代循环，调用自定义迭代钩子，并为每个不同属性的值执行语句

```javascript
const array1 = ['a', 'b', 'c'];

for (const val of array1) {
  console.log(val);
}

// a
// b
// c
```

**for of不可以遍历普通对象**，想要遍历对象的属性，可以用for in循环, 或内建的Object.keys()方法

## for...of与for...in的区别

无论是`for...in`还是`for...of`语句都是迭代一些东西。它们之间的主要区别在于它们的迭代方式。

[`for...in`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Statements/for...in)语句以任意顺序迭代对象的[可枚举属性](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Enumerability_and_ownership_of_properties)。

`for...of` 语句遍历[可迭代对象](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Guide/Iterators_and_Generators#Iterables)定义要迭代的数据。

以下示例显示了与[`Array`](https://developer.mozilla.org/zh-CN/docs/Web/JavaScript/Reference/Array)一起使用时，`for...of`循环和`for...in`循环之间的区别。

```javascript
Object.prototype.objCustom = function() {}; 
Array.prototype.arrCustom = function() {};

let iterable = [3, 5, 7];
iterable.foo = 'hello';

for (let i in iterable) {
  console.log(i); // 0, 1, 2, "foo", "arrCustom", "objCustom"
}

for (let i in iterable) {
  if (iterable.hasOwnProperty(i)) {
    console.log(i); // 0, 1, 2, "foo"
  }
}

for (let i of iterable) {
  console.log(i); // logs 3, 5, 7
}
```

# JS编译过程

作为新生代 ES 语法编译器，Babel 在前端工具链中占据了非常重要的地位，它严格按照 ECMA-262 语言规范，实现对最新语法的解析，而无需等待浏览器升级来提供对新特性的支持。Babel 内部所使用的语法解析器是 Babylon，抽象语法树（简写为 AST）的结点类型定义则参考了 Mozilla JS 引擎 SpiderMonkey，并对其进行扩展增强，且支持对 Flow、JSX、TypeScript 语法的解析。它所使用的 Babylon 实现了编译器中两个部分，词法分析和语法分析。

**词法分析**

词法分析是处理源程序的第一部分，主要任务是逐个扫描输入字符，转换为词法单元（Token）序列，传递给语法分析器进行语法分析。Token 是一个不可分割的最小单元。例如 var 这三个字符，它只能作为一个整体，语义上不能再被分解，因此它是一个 Token。每个 Token 对象都有能够被单独识别的类型属性和其它附加属性（操作符优先级、行列号等）。在 Babylon 词法分析器里，每个关键字是一个 Token ，每个标识符是一个 Token，每个操作符是一个 Token，每个标点符号也都是一个 Token。除此之外，还会过滤掉源程序中的注释和空白字符（换行符、空格、制表符等）。

对于 Token 的匹配规则，可以根据正则表达式来描述。举个例子，要匹配一个 Number 类型的 Token，可以检测是否以 [0-9] 开头，接着循环或递归扫描紧连的后续字符，且需要特别留意 0b、0o、0x 开头的非十进制数值、科学计数法 e 或 E、小数点等特殊字符，指针不断后移直至不满足匹配规则或者到达行末尾。最后生成一个 Number 类型的 Token，附带值、文件位置等属性，并加入到 Token 序列中，继续下一轮扫描。

一个简单的 Number 类型状态转换如图 2 所示：

![img](https://pic2.zhimg.com/80/v2-dd0fcef2961b3252d7b60ece4cace226_1440w.jpg)图2 Number 类型状态转换示意图

当然除了 Babylon 手写词法分析器之外，这个过程还可以采用有穷自动机（DFA/NFA）的方式实现，通过词法分析器生成器，把输入程序（模式匹配规则）自动转换成一个词法分析器，这里不展开阐述。

**语法分析**

语法分析是词法分析的下一步，主要任务是扫描来自词法分析器产生的 Token 序列，根据文法和结点类型定义构造出一棵 AST，传递给编译器前端余下部分。文法描述了程序设计语言的构造规则，用于指导整个语法分析的过程。它由四个部分组成，一组终结符号（也称 Token）、一组非终结符号、一组产生式和一个开始符号。例如，函数声明语句的产生式表示形式如图 3 所示：

![img](https://pic1.zhimg.com/80/v2-137c71f94287060f5f729649fd9b6715_1440w.jpg)图3 函数声明语句的产生式

根据文法，语法分析器将 Token 逐个读入，不断替换文法产生式体的非终结符号，直至全部将非终结符号替换为终结符号，这个过程被称为推导。推导又分为两种方式，最左推导和最右推导。如果总是优先替换产生式体最左侧的非终结符号，被称为最左推导，如果总是优先替换产生式体最右侧的非终结符号，被称为最右推导。

语法分析器按照工作方式来划分，分为自顶向下分析法和自底向上分析法。自顶向下分析法要求通过最左推导从顶部 ( 根结点 ) 开始构造 AST，常用的分析器有递归下降语法分析器、 LL 语法分析器。而自底向上分析法要求通过最右推导从底部 ( 叶子结点 ) 开始构造 AST，常用的分析器有 LR 语法分析器、SLR 语法分析器、LALR 语法分析器。这两种分析方式在 Babylon 中都有所实践。

首先是自顶向下分析法，例如变量声明语句：

```js
var foo = "bar";
```



经由词法分析器处理后，会生成 Token 序列：

```js
Token('var')
Token('foo')
Token('=')
Token('"bar"')
Token(';')
```



由 LL(1) 语法分析器进行递归下降分析，每次向前查看一个输入 Token，来决定该用哪种产生式展开。对于变量声明语句的 FIRST 集合（推导结果的首个 Token 集合），只需检查输入 Token 为 Token('var')、Token('let')、Token('const') 三者其中之一，那么就使用该产生式展开。首先构造 AST 最顶层结点 VariableDeclaration，把 Token('var') 的值加入到该结点属性中， 接着逐个读入其余 Token，根据产生式的非终结符号从左到右的顺序，依次构造它的子结点，不断递归下降分析，直至所有 Token 读入完毕。最后生成的一棵 AST 如图 4 所示：

![img](https://pic3.zhimg.com/80/v2-4f393465dc37aa83ba4bc0ccf7f18814_1440w.jpg)图4 自顶向下分析法产生的 AST 树



另一种是自底向上分析法，例如成员表达式语句：

```js
foo.bar.baz.qux
```



我们都知道这条语句等价于：

```js
((foo.bar).baz).qux
```



而不是：

```js
foo.(bar.(baz.qux))
```



原因就在于它所设计的文法是左递归的，而 LL 语法分析器是无法做到解析左递归的文法，这时候只能使用 LR 语法分析器的方式，自底向上地构造 AST。LR 语法分析器的核心是移入 - 归约分析技术，通过维护一个栈，由下一个输入 Token 来决定是把它移入栈中还是将栈顶的部分符号进行归约（把产生式体替换为产生式头），先构造子结点，再构造父结点，直至栈中所有符号全部归约。最后生成的一棵 AST 如图 5 所示：

![img](https://pic3.zhimg.com/80/v2-fa73035935c2dbf442d8edcab24b6d75_1440w.jpg)图5 自底向上分析法产生的 AST 树



此外，由 Babylon 构建的完整的 AST 还拥有特殊顶层结点 File 和 Program，它们描述了文件的基本信息、模块类型等等。



**生成代码**



工业级别的语言编译器，通常还会有语义分析阶段，检查程序上下文是否和语言所定义的语义一致，比如类型检查，作用域检查，另一个则是生成中间代码，比如三地址代码，用地址和指令来线性描述程序。但由于 Babel 的定位仅仅是对 ES 语法的转换，这一部分工作可以交给 JS 解释器引擎来处理。而 Babel 最为特色的部分是它的插件机制，针对不同的浏览器版本环境，调用不同的 Babel 插件。通过访问者模式（一种设计模式）的接口定义，对 AST 进行一遍深度优先遍历，对指定的匹配到的结点进行修改、删除、新增、移位，使原先的 AST 转换为另一棵经过修改的 AST。



一个访问者模式的接口定义如下：

```js
visitor: {
  Identifier(path) {
    enter() {
      //遍历AST进入Identifier结点时执行
      ... 
    },
    exit() {
      //遍历AST离开Identifier结点时执行
      ...
    }
  },
  ...
}
```



最后一个阶段则是生成目标代码，从 AST 的根结点出发，递归下降遍历，对每个结点都调用一个相关函数，执行语义动作，不断打印代码片段，最终生成目标代码，即经过 babel 编译后的代码。



## 模板引擎



再讲到模板引擎，最早诞生于服务端动态页面的开发，如 JSP、PHP、ASP 等模板引擎，自 Node.js 快速发展以后，前端界又产出了非常多的轮子，包括 EJS、Handlebars、Pug （前身为 Jade）、Mustache 等等，数不胜数。模板引擎技术使得结合数据渲染视图变得更加灵活，给逻辑的抽象带来了更多的可能性，数据与内容互不依赖。模板引擎的实现方式有很多种，比较简单的模板引擎，直接利用字符串替换、拼接的方式实现，比较复杂的模板引擎，例如 Pug，则会有比较完整的词法分析和语法分析过程，将模板预编译成 JS 代码再去动态执行。



例如模板语句：

```html
h1 hello #{name}
```



经由 Pug 解析器生成的 AST 如图 6 所示：

![img](https://pic2.zhimg.com/80/v2-ff766a1121973c3430fee76d6973ee87_1440w.jpg)图6 由 Pug 解析器生成的 AST



生成器生成的目标代码为（伪代码）：

```js
'<h1>' + 'hello' + name + '<h1>'
```

运行时再调用 new Function 来动态执行代码：

```js
var compiledFn = new Function('local', `
  with (local) {
    return '<h1>' + 'hello' + name + '<h1>'; 
  }
`)

compiledFn({
  name: 'world'
})
```

最后输出 HTML 语句：

```html
<h1>hello world</h1>
```

整个过程由两部分组成，预编译阶段和运行时阶段。当然一个好的模板引擎还会考虑功能、性能与安全兼备，上面的`with`语句是要避免的，还要引入缓存机制，XSS 防范机制，以及更加强大、友好、易于使用的语法糖。

另外值得一提的是以 Angular、React、Vue 为代表的前端 MVVM 框架，无一不引入了模板编译技术。Vue 作为渐进式的前端解决方案，受到众多开发者们的青睐，它对视图的渲染提供了渲染函数和模板两种方式。使用渲染函数需要调用核心 API 来构建 Virtual DOM 类型，过程相对复杂，编码量非常大，一旦 DOM 层次嵌套过深，就会造成代码难以掌控和维护的局面。为了应对这种复杂性，另一种方式则是编写基于 HTML 的模板，并加入 Vue 特有的标签、指令、插值等语法，由编译器来进行从模板到渲染函数的编译和优化，相对前者更优雅、便捷、易于编码。



## CSS 预处理器

前端布局方式从刀耕火种的纯 CSS 年代演进到以 Sass、Less、Stylus 为代表的预处理语言，赋予了 CSS 可编程的能力，定义变量，函数，表达式计算、模块化等特性，极大地提升了开发人员的生产效率。这些都是编译技术所带来的变化。同样，编译器对原样式代码进行词法分析，产生 Token 序列。接着，语法分析，生成中间表示，一棵符合定义的 AST。同时，还会为每个程序块建立一个符号表来记录变量的名字，属性，为代码生成阶段的变量作用域分析提供帮助。最后，递归下降访问 AST，生成能够在浏览器环境中直接执行的 CSS 代码。



以预处理器 Stylus 语法为例：

```css
foo = 14px

body
  font-size foo
```



编译生成的 AST 为图 7 所示：

![img](https://pic3.zhimg.com/80/v2-a2391f9eab04dc84a7d5985310776158_1440w.jpg)图7 由 Stylus 解析器生成的 AST



最后生成的目标代码为：

```css
body {
  font-size: 14px;
}
```



看似简单容易的代码转换背后，编译器为我们做了许多语法层面的处理，给 CSS 带来了从未有过的强大的扩展能力，以及底层对编译速度的持续优化，让 CSS 的编写方式更加简洁高效，易于维护和管理。