# 函数式编程简介

函数式编程时一种范式，我们能够以此创建依赖输入就可以完成自身逻辑的函数。这保证了当函数呗多次调用时仍返回相同的结果。函数不会改变任何外部环境的变量，这将产生可缓存、可测试的代码库。

所有的函数对于相同的输入都将返回相同的值。函数的这一属性被称为**引用透明性**。

**并发代码**和**缓存**。

函数式编程主张**声明式编程**和编写**抽象**的代码。



# JavaScript 函数基础

babel 是一个转换编译器，能够把 ES6 代码转换为有效的 ES5 代码。

var  在一个函数内全局地定义变量，不管它定义在哪个块。



# 高阶函数

**高阶函数**（Higher-Order Function）是接收函数作为参数并且/或者返回函数作为输出的函数。

```js
const forEach = (array, fn) => {
    let i;
    for(i=0; i < array.length; i++)
        fn(array[i]);
}
```

函数也可作为 JavaScript 的一种数据类型。当一门语言允许函数作为任何其他数据类型使用时，函数被称为**一等公民**。也就是说，函数可被赋值给变量，作为参数传递，也可被其他函数返回。

```js
// 如果参数是函数，tellType 就执行它
var tellType = (arg) => {
	if(typeof arg === "function")
        arg();
    else
        ocnsole.log("The passed data is " + arg);
}
```

遍历对象的函数：

```js
const forEachObject = (obj, fn) => {
    for (var property in obj) {
        if (obj.hasOwnProperty(property)) {
            // 以 key 和 value 作为参数调用 fn
            fn(property, obj[property])
        }
    }
}
```

以抽象的方式实现对流程控制的处理：

```js
const unless = (predicate, fn) => {
    if(!predicate)
        fn()
}

// 查找一个列表中的偶数
forEach([1,2,3,4,5,6,7],(number) => {
    unless((number % 2), () => {
		console.log(number, " is even")
    })
})
```

