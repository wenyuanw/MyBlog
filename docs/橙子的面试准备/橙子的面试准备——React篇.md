# 对react的理解（react运行机制）

React 是一个 View 层的框架，用来渲染视图，它主要做几件事情：

组件化利用 props 形成单向的数据流根据 state 的变化来更新 view利用虚拟 DOM 来提升渲染性能

# react与原生js

1.组件化：其中以react的组件化最为彻底，甚至可以到函数级别的原子组件，高度的组件化可以使我们的工程易于维护，易于组合扩展；

2.天然分层：jQuery时代的代码大部分情况下是面条代码，耦合严重，现代框架不管是MVC、MVP还是MVVM模式都可以帮我们进行分层，代码解耦更易于读写；

3.生态：现代主流框架都自带生态，不管是数据流管理架构还是UI库都有成熟的解决方案；

4：开发效率：现在前端框架都默认自动更新DOM，而非我们手动操作，解放了开发者的手动DOM成本，提高开发效率，从根本上解决了UI与状体同步问题。

# react调和过程

调和的目的：Reconciliation 的目的是在用户无感知的情况下将数据的更新体现到UI上。

触发调和的过程： 

在React 中有以下几种操作会触发调和过程： 

- ReactDom.render()函数 和ReactNativeRenderer.render()函数
- setState()函数
- forceUpdate()函数的调用
- componentWillMount 和componentWillReceiveProp 中直接修改了state（地址）
- hooks 中的useReducer 和 useState 返回的钩子函数

调和过程涉及的数据结构：

接触React时，首先接触的是virtualDom这个词，会好奇什么是virtualDom。开端的引文已经解释，virtualDom只是一种编程理念，在react中若一定要将他与某种数据相关联，那应该是ReactElement 和fibe，fibe更合适。

# class类组件和函数式组件区别

**1. Extends React.Component 定义的组件**

React.Component是以ES6的形式来创建react的组件的，是React目前极为推荐的创建有状态组件的方式，最终会取代React.createClass形式

```react
class Demo extends React.Component {
    constructor(props) {
        super(props);
        this.state = {
            text: props.initialValue || 'placeholder'
        };
        // 手动绑定this
        this.handleChange = this.handleChange.bind(this);
    }
    handleChange(event) {
        this.setState({
            text: event.target.value
        });
    }
    render() {
        return (
            <div>
                <input value={this.state.text} onChange={this.handleChange}/>
            </div>
        );
    }
}
Demo.propTypes = {
    initialValue: React.PropTypes.string
}
```

**2. 纯函数式定义的无状态组件**

纯函数组件的特点:

- 组件不会被实例化,整体渲染性能得到提升
- 组件不能访问this对象
- 组件无法访问生命周期的方法
- 无状态组件只能访问输入的props,无副作用

```react
function DemoComponent(props) {
  return <div>Hello {props.name}</div>
}
ReactDOM.render(<DemoComponent name="每日一题" />, mountNode) 
```

**使用场景**

以类形式创建的组件不用多说，该怎么用还怎么用, 这里说一纯函数组件, 纯函数组件被鼓励在大型项目中尽可能以简单的写法来分割原本庞大的组件，未来React也会这种面向无状态组件在譬如无意义的检查和内存分配领域进行一系列优化，所以只要有可能，尽量使用无状态组件

**总结**

- react中有三种创建组件的形式
- 纯函数不会被实例化，不能访问this，没有生命周期
- 尽可能的使用纯函数拆分复杂型组件

# pureComponent

除了为你提供了一个具有浅比较的`shouldComponentUpdate`方法，`PureComponent`和`Component`基本上完全相同。当`props`或者`state`改变时，`PureComponent`将对`props`和`state`进行浅比较。另一方面，Component不会比较当前和下个状态的`props`和`state`。因此，每当`shouldComponentUpdate`被调用时，组件默认的会重新渲染。

当组件更新时，如果组件的 props 和 state 都没发生改变， render 方法就不会触发，省去 Virtual DOM 的生成和比对过程，达到提升性能的目的。具体就是 React 自动帮我们做了一层浅比较：

```react
if (this._compositeType === CompositeTypes.PureClass) {
    shouldUpdate = !shallowEqual(prevProps, nextProps) || !shallowEqual(inst.state, nextState);
}
```

而 shallowEqual 又做了什么呢？会比较 Object.keys(state | props) 的长度是否一致，每一个 key 是否两者都有，并且是否是一个引用，也就是只比较了第一层的值，确实很浅，所以深层的嵌套数据是对比不出来的。

大部分情况下，你可以使用React.PureComponent而不必写你自己的shouldComponentUpdate，它只做一个浅比较。但是由于浅比较会忽略属性或状态**突变**的情况，此时你不能使用它。

`PureComponent` 真正起作用的，只是在一些纯展示组件上，复杂组件使用的话`shallowEqual` 那一关基本就过不了。另外在使用的过程中为了确保能够正确的渲染，记得 `props` 和 `state` 不能使用同一个引用哦。

# 组件中传值

**关于react组件之间的传值**            

```
1.首先是父子组件最基础的传值(举一个todulist的例子)
```

- 父传子  采用props的格式

  我们拿这个datas

  ![preview](https://segmentfault.com/img/bVbwoi7?w=686&h=203/view)

  在父组件里引入的子组件上进行传值

  ![preview](https://segmentfault.com/img/bVbwojB?w=609&h=44/view)

  子组件中采用this.props.xx的方式获取
  ![preview](https://segmentfault.com/img/bVbwokB?w=734&h=352/view)

  效果：
  ![preview](https://segmentfault.com/img/bVbwokD?w=710&h=119/view)

- 子传父  通过在父组件引入的子组件中传递一个函数并传参，子组件去触发这个函数更改参数完成数据更新

![preview](https://segmentfault.com/img/bVbwok0?w=459&h=100/view)

![preview](https://segmentfault.com/img/bVbwok1?w=530&h=38/view)

![preview](https://segmentfault.com/img/bVbwolj?w=818&h=493/view)

```
这就行了！
查看删除效果 这样我们就把煮饺子干掉了
```

![preview](https://segmentfault.com/img/bVbwolS?w=420&h=144/view)

2.接下来介绍一款插件，pubsub 这个可以采用发布订阅的方式实现组件间的传值                        

```
 这里拿一个github搜索用户的小实例
 
 下载引入pubsub并在其中一个组件中发布
```

![preview](https://segmentfault.com/img/bVbwoqR?w=851&h=472/view)

​                        

```
前者是键名，后者是键值
在另一个组件中通过pubsub.subscribe订阅
```

![preview](https://segmentfault.com/img/bVbwoqY?w=665&h=390/view)

​                        

```
拿到数据就可以做后续一系列操作  我这里是一个页面发送搜索内容 一个页面收到搜索内容并通过axios请求搜索内容找到当前github用户
```

效果：搜索github用户

3.通过redux或者react-redux的方式进行各个页面的数据传递，就不用我说了吧？

4.通过上下文的形式做组件传值
这里以react hooks配合函数组件做例子
从react中引入

![preview](https://segmentfault.com/img/bVbwor5?w=742&h=36/view)
创建上下文
![preview](https://segmentfault.com/img/bVbwor6?w=289&h=30/view)
在父组件引入子组件的地方采用你刚才创建的上下文.Provider 通过value=传递数据
![preview](https://segmentfault.com/img/bVbwor9?w=945&h=266/view)

最后  在你子组件里用就好啦
![图片描述](https://segmentfault.com/img/bVbwoso?w=788&h=344)

# React key的作用

在交叉对比中，当新节点跟旧节点`头尾交叉对比`没有结果时，会根据新节点的key去对比旧节点数组中的key，从而找到相应旧节点（这里对应的是一个key => index 的map映射）。如果没找到就认为是一个新增节点。而如果没有key，那么就会采用遍历查找的方式去找到对应的旧节点。一种一个map映射，另一种是遍历查找。相比而言。map映射的速度更快。

主要是为了提升diff【同级比较】的效率。自己想一下自己要实现前后列表的diff，如果对列表的每一项增加一个key，即唯一索引，那就可以很清楚的知道两个列表谁少了谁没变。而如果不加key的话，就只能一个个对比了。

# 执行setState后经过了哪些生命周期，发生了什么

- React会将当前传入的参数对象与组件当前的状态合并,然后触发调和过程,在调和的过程中,React会以相对高效的方式根据新的状态构建React元素树并且重新渲染整个UI界面.
- React得到的元素树之后,React会自动计算出新的树与老的树的节点的差异,然后根据差异对界面进行最小化的渲染,在React的差异算法中,React能够精确的知道在哪些位置发生看改变以及应该如何去改变,这样就保证了UI是按需更新的而不是重新渲染整个界面.

setState的调用会引起React的更新生命周期的4个函数执行。

shouldComponentUpdate
componentWillUpdate
render
componentDidUpdate

当shouldComponentUpdate执行时，返回true，进行下一步，this.state没有被更新
返回false，停止，更新this.state

当componentWillUpdate被调用时，this.state也没有被更新

直到render被调用时候，this.state才被更新。

总之，直到下一次render函数调用(或者下一次shouldComponentUpdate返回false时)才能得到更新后的this.state
因此获取更新后的状态可以有3种方法：

# 高阶组件HOC，render props以及hooks的对比及用处

## HOC

### 创建 HOC 的方式

学习 HOC 我们只需要记住以下 2 点定义:

1. 创建一个函数, 该函数接收一个组件作为输入*除了组件, 还可以传递其他的参数*
2. 基于该组件返回了一个不同的组件.

代码如下:

```javascript
function withSubscription(WrappedComponent, selectData) {
  return class extends React.Component {
    constructor(props) {
      super(props);
      this.state = {
        data: selectData(DataSource, props)
      };
    }
    // 一些通用的逻辑处理
    render() {
      // ... 并使用新数据渲染被包装的组件!
      return <WrappedComponent data={this.state.data} {...this.props} />;
    }
  };
}
```

### HOC 的优点

- 不会影响内层组件的状态, 降低了耦合度

### HOC 的缺点

- 固定的 props 可能会被覆盖.

```javascript
<MyComponent
  x="a"
  y="b"
/>
```

这 2 个值,如果跟高阶组件的值相同, 那么 x,y 都会被来自高阶组件的值覆盖.

```javascript
// 如果 withMouse 和 withPage 使用了同样的 props, 比如 x , y.
// 则会有冲突.
export default withMouse(withPage(MyComponent));
```

- 它无法清晰地标识数据的来源

withMouse(MyComponent) 它不会告诉你组件中包含了哪些 props , 增加了调试和修复代码的时间.

## Render props

1. 接收一个外部传递进来的 props 属性
2. 将内部的 state 作为参数传递给调用组件的 props 属性方法.

代码如下:

```javascript
class Mouse extends React.Component {
  constructor(props) {
    super(props);
    this.state = { x: 0, y: 0 };
  }

  render() {
    return (
      <div style={{ height: '100%' }}>
        // 使用 render props 属性来确定要渲染的内容
        {this.props.render(this.state)}
      </div>
    );
  }
}
// 调用方式:
<Mouse render={mouse => (
  <p>鼠标的位置是 {mouse.x}，{mouse.y}</p>
)}/>
```

### 缺点

- 无法在 return 语句外访问数据

它不允许在 return 语句之外使用它的数据. 比如上面的例子, 不能在 useEffect 钩子或组件中的任何其他地方使用 x 和 y 值, 只能在 return 语句中访问.

- 嵌套

它很容易导致嵌套地狱. 如下代码:

```
const MyComponent = () => {
  return (
    <Mouse>
      {({ x, y }) => (
        <Page>
          {({ x: pageX, y: pageY }) => (
            <Connection>
              {({ api }) => {
                // yikes
              }}
            </Connection>
          )}
        </Page>
      )}
    </Mouse>
  )
};
```

基于上面的两种方式,都有他们的缺点,那我们来到最香的解决方案: hook

## Hooks

我们来看一个例子:

```
const { x, y } = useMouse();
const { x: pageX, y: pageY } = usePage();

useEffect(() => {

}, [pageX, pageY]);
```

从上面的代码可以看出, hook 有以下的特性. 它解决了上面 hoc 和 render props 的缺点.

- hook 可以重命名

如果 2 个 hook 暴露的参数一样,我们可以简单地进行重命名.

- hook 会清晰地标注来源

从上面的例子可以简单地看到, x 和 y 来源于 useMouse. 下面的 x, y 来源于 usePage.

- hook 可以让你在 return 之外使用数据
- hook 不会嵌套
- 简单易懂, 对比 hoc 和 render props 两种方式, 它非常直观, 也更容易理解.

# 虚拟DOM是什么？

在 React 中，React 会先将代码转换成一个 JS 对象，然后再将这个 JS 对象转换成真正的 DOM。这个 JS 对象就是所谓的虚拟 DOM。
它可以让我们无须关注 DOM 操作， 只需要开心地编写数据,状态即可。

# React diff原理，如何从O(n^3)变成O(n)

## 为什么是 O(n^3) ?

从一棵树转化为另外一棵树，直观的方式是用动态规划，通过这种记忆化搜索减少时间复杂度。由于树是一种递归的数据结构，因此最简单的树的比较算法是递归处理。确切地说，树的最小距离编辑算法的时间复杂度是 O(n^2m(1+logmn)), 我们假设 m 与 n 同阶， 就会变成 O(n^3)。

简单的来讲, react 它只比较同一层, 一旦不一样, 就删除. 这样子每一个节点只会比较一次, 所以算法就变成了 O(n).

对于同一层的一组子节点. 他们有可能顺序发生变化, 但是内容没有变化. react 根据 key 值来进行区分, 一旦 key 值相同, 就直接返回之前的组件, 不重新创建.

这也是为什么渲染数组的时候, 没有加 key 值或者出现重复key值会出现一些奇奇怪怪的 bug . 

除了 key , 还提供了选择性子树渲染。开发人员可以重写 shouldComponentUpdate 提高 diff 的性能。

# jsx的原理

```javascript
<div>Hello ConardLi</div>
```

实际上, babel 帮我们将这个语法转换成

```javascript
React.createElement('div', null, `Hello ConardLi`)
```

## 自定义组件必须大写的原因.

babel 在编译的过程中会判断 JSX 组件的首字母, 如果是小写, 则为原生 DOM 标签, 就编译成字符串. 如果是大写, 则认为是自定义组件. 编译成对象.

为什么以下代码会报错?

```
return (<a></a><a></a>)
```

同样的, 因为我们是按照 React.createElement() 来创建组件, 所以只能有一个根节点. 如果你想要使用 2 个平行的节点, 可以用 <></> 来包裹. <></> 会被编译成 <React.Fragment/>. 

babel 转译如下:![img](https://mmbiz.qpic.cn/mmbiz_png/ZWVxrQ7G0WTCibmkfzlhDlvdn6c5jWz184Gp7P7045dzc6GnVapgZ1J4pTadshKYfHexrE37nOuR06tibOXY5BAw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

# setState什么时候是同步，什么时候是异步？

**`setState` 只在合成事件和钩子函数中是“异步”的，在原生事件和 `setTimeout` 中都是同步的。**

**`setState`的“异步”并不是说内部由异步代码实现，其实本身执行的过程和代码都是同步的，只是合成事件和钩子函数的调用顺序在更新之前，导致在合成事件和钩子函数中没法立马拿到更新后的值，形式了所谓的“异步”，当然可以通过第二个参数 setState(partialState, callback) 中的callback拿到更新后的结果。**

**`setState` 的批量更新优化也是建立在“异步”（合成事件、钩子函数）之上的，在原生事件和setTimeout 中不会批量更新，在“异步”中如果对同一个值进行多次 `setState` ， `setState` 的批量更新策略会对其进行覆盖，取最后一次的执行，如果是同时 `setState` 多个不同的值，在更新时会对其进行合并批量更新。**

https://juejin.im/post/6844903636749778958#heading-1

这里的“异步”不是说异步代码实现. 而是说 react 会先收集变更,然后再进行统一的更新.

setState 在原生事件和 setTimeout 中都是同步的. 在合成事件和钩子函数中是异步的.

在 setState 中, 会根据一个 isBatchingUpdates 判断是直接更新还是稍后更新, 它的默认值是 false. 但是 React 在调用事件处理函数之前会先调用 batchedUpdates 这个函数, batchedUpdates 函数 会将 isBatchingUpdates 设置为 true. 因此, 由 react 控制的事件处理过程, 就变成了异步(批量更新).

# React如何实现自己的事件机制？

我们先看看 冒泡捕获 的经典图:

![img](https://mmbiz.qpic.cn/mmbiz_png/ZWVxrQ7G0WTCibmkfzlhDlvdn6c5jWz189abwwrY4A2Bu7lOpM59YYicYuMQaJXthWUbcLWUSDvSBLc9a82r3YZQ/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

在组件挂载的阶段, 根据组件生命的 react 事件, 给 document 添加事件 addEventListener, 并添加统一的事件处理函数 dispatchEvent.

将所有的事件和事件类型以及 react 组件进行关联, 将这个关系保存在一个 map 里. 当事件触发的时候, 首先生成合成事件, 根据组件 id 和事件类型找到对应的事件函数, 模拟捕获流程, 然后依次触发对应的函数.

> 如果原生事件使用 stopPropagation 阻止了冒泡, 那么合成事件也被阻止了.

## React 事件机制跟原生事件有什么区别

1. React 的事件使用驼峰命名, 跟原生的全部小写做区分.
2. 不能通过 return false 来阻止默认行为, 必须明确调用 preventDefault 去阻止浏览器的默认响应.

# 聊一聊fiber架构

背景: 由于浏览器它将 GUI 描绘，时间器处理，事件处理，JS 执行，远程资源加载统统放在一起。如果执行 js 的更新， 占用了太久的进程就会导致浏览器的动画没办法执行，或者 input 响应比较慢。

react fiber 使用了 2 个核心解决思想:

- 让渲染有优先级
- 可中断

React Fiber 将虚拟 DOM 的更新过程划分两个阶段，reconciler 调和阶段与 commit 阶段. 看下图:![img](https://mmbiz.qpic.cn/mmbiz_png/ZWVxrQ7G0WTCibmkfzlhDlvdn6c5jWz18qyibQibjeGHFxZPryu9g0zNfmz28ChL70e4icNDd5lleMDsLG4Y5GC6Og/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)一次更新过程会分为很多个分片完成, 所以可能一个任务还没有执行完, 就被另一个优先级更高的更新过程打断, 这时候, 低优先级的工作就完全作废, 然后等待机会重头到来.

**调度的过程**

requestIdleCallback![img](https://mmbiz.qpic.cn/mmbiz_png/ZWVxrQ7G0WTCibmkfzlhDlvdn6c5jWz18XyMXlAHKhLZwegElw13MZMHWV7ghhZKrCjeqqkOVI66kPau5UaKonw/640?wx_fmt=png&wxfrom=5&wx_lazy=1&wx_co=1)

首先 react 会根据任务的优先级去分配各自的过期时间 expriationTime . requestIdleCallback 在每一帧的多余时间(黄色的区域)调用. 调用 channel.port1.onmessage , 先去判断当前时间是否小于下一帧时间, 如果小于则代表我们有空余时间去执行任务, 如果大于就去执行过期任务,如果任务没过期. 这个任务就被丢到下一帧执行了.

# React事件中为什么要绑定this或者箭头函数，它们有什么区别？

事实上, 这并不算是 react 的问题, 而是 this 的问题. 但是也是 react 中经常出现的问题. 因此也讲一下

```
<button type="button" onClick={this.handleClick}>Click Me</button>
```

这里的 this . 当事件被触发且调用时, 因为 this 是在运行中进行绑定的.his 的值会回退到默认绑定，即值为 undefined，这是因为类声明和原型方法是以严格模式运行。

我们可以使用 bind 绑定到组件实例上. 而不用担心它的上下文.

因为箭头函数中的 this 指向的是定义时的 this，而不是执行时的 this. 所以箭头函数同样也可以解决.

# redux

应用中所有的 state 都以一个对象树的形式储存在一个单一的 *store* 中。 惟一改变 state 的办法是触发 *action*，一个描述发生什么的对象。 为了描述 action 如何改变 state 树，你需要编写 *reducers*。

### Redux 设计和使用的三项原则

1. 首先 store 要求必须是唯一的
2. 只有 store 能够改变自己的内容

有的小伙伴可能会疑惑，明明是 reducer 对数据进行了整理，其实 reducer 只是将原有数据和新的 action 进行了整理，最终还是需要把新的 state 返回给 store , store 拿到 reducer 的数据，再对自己的数据进行更新

3、reducer 必须是纯函数

纯函数指的是，给定固定的输入，就一定会有固定的输出，而且不会有任何副作用，如果一个函数里边有 ajax 等异步操作,或者与日期相关的操作之后，他都不是一个纯函数，副作用是指对传入的参数进行修改

### Redux 中核心的 API

1. **createStore** 可以帮助创建 store
2. **store.dispatch** 帮助派发 action , action 会传递给 store
3. **store.getState** 这个方法可以帮助获取 store 里边所有的数据内容
4. **store.subscrible** 方法可以让让我们订阅 store 的改变，只要 store 发生改变， store.subscrible 这个函数接收的这个回调函数就会被执行

# React生命周期函数

## 16.3之前的生命周期函数

初始化阶段：constructor

挂载阶段（mounting）：

- componentWillMount  在组件挂载到DOM前调用，且只会被调用一次，在这边调用this.setState不会引起组件重新渲染，也可以把写在这边的内容提前到constructor()中，所以项目中很少用。
- render  在组件挂载到DOM前调用，且只会被调用一次，在这边调用this.setState不会引起组件重新渲染，也可以把写在这边的内容提前到constructor()中，所以项目中很少用。
- componentDidMount   组件挂载到DOM后调用，且只会被调用一次

更新阶段（update）：在讲述此阶段前需要先明确下react组件更新机制。setState引起的state更新或父组件重新render引起的props更新，更新后的state和props相对之前无论是否有变化，都将引起子组件的重新render。

- componentWillReceiveProps(nextProps)

此方法只调用于props引起的组件更新过程中，响应 Props 变化之后进行更新的唯一方式，参数nextProps是父组件传给当前组件的新props。但父组件render方法的调用不能保证重传给当前组件的props是有变化的，所以在此方法中根据nextProps和this.props来查明重传的props是否改变，以及如果改变了要执行啥，比如根据新的props调用this.setState出发当前组件的重新render

- shouldComponentUpdate(nextProps, nextState)

此方法通过比较nextProps，nextState及当前组件的this.props，this.state，返回true时当前组件将继续执行更新过程，返回false则当前组件更新停止，以此可用来减少组件的不必要渲染，优化组件性能。

ps：这边也可以看出，就算componentWillReceiveProps()中执行了this.setState，更新了state，但在render前（如shouldComponentUpdate，componentWillUpdate），this.state依然指向更新前的state，不然nextState及当前组件的this.state的对比就一直是true了。

如果shouldComponentUpdate 返回false，那就一定不用rerender(重新渲染 )这个组件了，组件的React elements(React 元素) 也不用去比对。 但是如果shouldComponentUpdate 返回true，会进行组件的React elements比对，如果相同，则不用rerender这个组件，如果不同，会调用render函数进行rerender。

- componentWillUpdate(nextProps, nextState)

此方法在调用render方法前执行，在这边可执行一些组件更新发生前的工作，一般较少用。

- render

render方法在上文讲过，这边只是重新调用。

- componentDidUpdate(prevProps, prevState)

此方法在组件更新后被调用，可以操作组件更新的DOM，prevProps和prevState这两个参数指的是组件更新前的props和state

卸载阶段（Unmount）：

componentWillUnmount

## 16.3之后的生命周期函数

### 变更缘由

原来（React v16.0前）的生命周期在React v16推出的[Fiber](https://links.jianshu.com/go?to=https%3A%2F%2Fzhuanlan.zhihu.com%2Fp%2F26027085)之后就不合适了，因为如果要开启async rendering，在render函数之前的所有函数，都有可能被执行多次。

原来（React v16.0前）的生命周期有哪些是在render前执行的呢？

- componentWillMount
- componentWillReceiveProps
- shouldComponentUpdate
- componentWillUpdate

如果开发者开了async rendering，而且又在以上这些render前执行的生命周期方法做AJAX请求的话，那AJAX将被无谓地多次调用。。。明显不是我们期望的结果。而且在componentWillMount里发起AJAX，不管多快得到结果也赶不上首次render，而且componentWillMount在服务器端渲染也会被调用到（当然，也许这是预期的结果），这样的IO操作放在componentDidMount里更合适。

禁止不能用比劝导开发者不要这样用的效果更好，所以除了shouldComponentUpdate，其他在render函数之前的所有函数（componentWillMount，componentWillReceiveProps，componentWillUpdate）都被getDerivedStateFromProps替代。

也就是用一个静态函数getDerivedStateFromProps来取代被deprecate的几个生命周期函数，就是强制开发者在render之前只做无副作用的操作，而且能做的操作局限在根据props和state决定新的state

React v16.0刚推出的时候，是增加了一个componentDidCatch生命周期函数，这只是一个增量式修改，完全不影响原有生命周期函数；但是，到了React v16.3，大改动来了，引入了两个新的生命周期函数： `getDerivedStateFromProps`，`getSnapshotBeforeUpdate`

![img](https://upload-images.jianshu.io/upload_images/5287253-19b835e6e7802233.png?imageMogr2/auto-orient/strip|imageView2/2/w/1200/format/webp)

# React-Router

## 哈希路由

hash路由一个明显的标志是带有#,我们主要是通过监听url中的hash变化来进行路由跳转。
hash的优势就是兼容性更好,在老版IE中都有运行,问题在于url中一直存在#不够美观,而且hash路由更像是Hack而非标准,相信随着发展更加标准化的History API会逐步蚕食掉hash路由的市场。

#### 1.1 初始化class

我们用`Class`关键字初始化一个路由.

```javascript
class Routers {
  constructor() {
    // 以键值对的形式储存路由
    this.routes = {};
    // 当前路由的URL
    this.currentUrl = '';
  }
}
```

#### 1.2 实现路由hash储存与执行

在初始化完毕后我们需要思考两个问题:

1. 将路由的hash以及对应的callback函数储存
2. 触发路由hash变化后,执行对应的callback函数

```javascript
class Routers {
  constructor() {
    this.routes = {};
    this.currentUrl = '';
  }
  // 将path路径与对应的callback函数储存
  route(path, callback) {
    this.routes[path] = callback || function() {};
  }
  // 刷新
  refresh() {
    // 获取当前URL中的hash路径
    this.currentUrl = location.hash.slice(1) || '/';
    // 执行当前hash路径的callback函数
    this.routes[this.currentUrl]();
  }
}
```

#### 1.3 监听对应事件

那么我们只需要在实例化`Class`的时候监听上面的事件即可.

```javascript
class Routers {
  constructor() {
    this.routes = {};
    this.currentUrl = '';
    this.refresh = this.refresh.bind(this);
    window.addEventListener('load', this.refresh, false);
    window.addEventListener('hashchange', this.refresh, false);
  }

  route(path, callback) {
    this.routes[path] = callback || function() {};
  }

  refresh() {
    this.currentUrl = location.hash.slice(1) || '/';
    this.routes[this.currentUrl]();
  }
}

```

对应效果如下:

![img](https://user-gold-cdn.xitu.io/2018/4/7/1629f6b9cb508a43?imageslim)



#### 2.1 实现后退功能

我们在需要创建一个数组`history`来储存过往的hash路由例如`/blue`,并且创建一个指针`currentIndex`来随着*后退*和*前进*功能移动来指向不同的hash路由。

```javascript
class Routers {
  constructor() {
    // 储存hash与callback键值对
    this.routes = {};
    // 当前hash
    this.currentUrl = '';
    // 记录出现过的hash
    this.history = [];
    // 作为指针,默认指向this.history的末尾,根据后退前进指向history中不同的hash
    this.currentIndex = this.history.length - 1;
    this.refresh = this.refresh.bind(this);
    this.backOff = this.backOff.bind(this);
    window.addEventListener('load', this.refresh, false);
    window.addEventListener('hashchange', this.refresh, false);
  }

  route(path, callback) {
    this.routes[path] = callback || function() {};
  }

  refresh() {
    this.currentUrl = location.hash.slice(1) || '/';
    // 将当前hash路由推入数组储存
    this.history.push(this.currentUrl);
    // 指针向前移动
    this.currentIndex++;
    this.routes[this.currentUrl]();
  }
  // 后退功能
  backOff() {
    // 如果指针小于0的话就不存在对应hash路由了,因此锁定指针为0即可
    this.currentIndex <= 0
      ? (this.currentIndex = 0)
      : (this.currentIndex = this.currentIndex - 1);
    // 随着后退,location.hash也应该随之变化
    location.hash = `#${this.history[this.currentIndex]}`;
    // 执行指针目前指向hash路由对应的callback
    this.routes[this.history[this.currentIndex]]();
  }
}
```

我们看起来实现的不错,可是出现了Bug,在后退的时候我们往往需要点击两下。

点击查看Bug示例 [hash router](https://codepen.io/xiaomuzhu/pen/mxQBod/) by 寻找海蓝 ([@xiaomuzhu](https://codepen.io/xiaomuzhu)) on [CodePen](https://codepen.io).

问题在于,我们每次在后退都会执行相应的callback,这会触发`refresh()`执行,因此每次我们后退,`history`中都会被`push`新的路由hash,`currentIndex`也会向前移动,这显然不是我们想要的。

```javascript
  refresh() {
    this.currentUrl = location.hash.slice(1) || '/';
    // 将当前hash路由推入数组储存
    this.history.push(this.currentUrl);
    // 指针向前移动
    this.currentIndex++;
    this.routes[this.currentUrl]();
  }
```

如图所示,我们每次点击后退,对应的指针位置和数组被打印出来

![img](https://user-gold-cdn.xitu.io/2018/4/7/162a01b9e9d6f502?imageView2/0/w/1280/h/960/format/webp/ignore-error/1)



#### 2.2 完整实现hash Router

我们必须做一个判断,如果是后退的话,我们只需要执行回调函数,不需要添加数组和移动指针。

```javascript
class Routers {
  constructor() {
    // 储存hash与callback键值对
    this.routes = {};
    // 当前hash
    this.currentUrl = '';
    // 记录出现过的hash
    this.history = [];
    // 作为指针,默认指向this.history的末尾,根据后退前进指向history中不同的hash
    this.currentIndex = this.history.length - 1;
    this.refresh = this.refresh.bind(this);
    this.backOff = this.backOff.bind(this);
    // 默认不是后退操作
    this.isBack = false;
    window.addEventListener('load', this.refresh, false);
    window.addEventListener('hashchange', this.refresh, false);
  }

  route(path, callback) {
    this.routes[path] = callback || function() {};
  }

  refresh() {
    this.currentUrl = location.hash.slice(1) || '/';
    if (!this.isBack) {
      // 如果不是后退操作,且当前指针小于数组总长度,直接截取指针之前的部分储存下来
      // 此操作来避免当点击后退按钮之后,再进行正常跳转,指针会停留在原地,而数组添加新hash路由
      // 避免再次造成指针的不匹配,我们直接截取指针之前的数组
      // 此操作同时与浏览器自带后退功能的行为保持一致
      if (this.currentIndex < this.history.length - 1)
        this.history = this.history.slice(0, this.currentIndex + 1);
      this.history.push(this.currentUrl);
      this.currentIndex++;
    }
    this.routes[this.currentUrl]();
    console.log('指针:', this.currentIndex, 'history:', this.history);
    this.isBack = false;
  }
  // 后退功能
  backOff() {
    // 后退操作设置为true
    this.isBack = true;
    this.currentIndex <= 0
      ? (this.currentIndex = 0)
      : (this.currentIndex = this.currentIndex - 1);
    location.hash = `#${this.history[this.currentIndex]}`;
    this.routes[this.history[this.currentIndex]]();
  }
}
```

查看完整示例 [Hash Router](https://codepen.io/xiaomuzhu/pen/VXVrxa/) by 寻找海蓝 ([@xiaomuzhu](https://codepen.io/xiaomuzhu)) on [CodePen](https://codepen.io).

前进的部分就不实现了,思路我们已经讲得比较清楚了,可以看出来,hash路由这种方式确实有点繁琐,所以HTML5标准提供了History API供我们使用。

## history路由

我们只简单看一下常用的API

```
window.history.back();       // 后退
window.history.forward();    // 前进
window.history.go(-3);       // 后退三个页面
```

`history.pushState`用于在浏览历史中添加历史记录,但是并不触发跳转,此方法接受三个参数，依次为：

> `state`:一个与指定网址相关的状态对象，`popstate`事件触发时，该对象会传入回调函数。如果不需要这个对象，此处可以填`null`。
>  `title`：新页面的标题，但是所有浏览器目前都忽略这个值，因此这里可以填`null`。
>  `url`：新的网址，必须与当前页面处在同一个域。浏览器的地址栏将显示这个网址。

`history.replaceState`方法的参数与`pushState`方法一模一样，区别是它修改浏览历史中当前纪录,而非添加记录,同样不触发跳转。

`popstate`事件,每当同一个文档的浏览历史（即history对象）出现变化时，就会触发popstate事件。

需要注意的是，仅仅调用`pushState`方法或`replaceState`方法 ，并不会触发该事件，只有用户点击浏览器倒退按钮和前进按钮，或者使用 JavaScript 调用`back`、`forward`、`go`方法时才会触发。

另外，该事件只针对同一个文档，如果浏览历史的切换，导致加载不同的文档，该事件也不会触发。

### hash 路由

```javascript
class Router {
  constructor() {
    // 储存 hash 与 callback 键值对
    this.routes = {};
    // 当前 hash
    this.currentUrl = '';
    // 记录出现过的 hash
    this.history = [];
    // 作为指针,默认指向 this.history 的末尾,根据后退前进指向 history 中不同的 hash
    this.currentIndex = this.history.length - 1;
    this.backIndex = this.history.length - 1
    this.refresh = this.refresh.bind(this);
    this.backOff = this.backOff.bind(this);
    // 默认不是后退操作
    this.isBack = false;
    window.addEventListener('load', this.refresh, false);
    window.addEventListener('hashchange', this.refresh, false);
  }

  route(path, callback) {
    this.routes[path] = callback || function() {};
  }

  refresh() {
    console.log('refresh')
    this.currentUrl = location.hash.slice(1) || '/';
    this.history.push(this.currentUrl);
    this.currentIndex++;
    if (!this.isBack) {
      this.backIndex = this.currentIndex
    }
    this.routes[this.currentUrl]();
    console.log('指针:', this.currentIndex, 'history:', this.history);
    this.isBack = false;
  }
  // 后退功能
  backOff() {
    // 后退操作设置为true
    console.log(this.currentIndex)
    console.log(this.backIndex)
    this.isBack = true;
    this.backIndex <= 0 ?
      (this.backIndex = 0) :
      (this.backIndex = this.backIndex - 1);
    location.hash = `#${this.history[this.backIndex]}`;
  }
}
```

完整实现 [hash-router](https://codepen.io/fi3ework/pen/ejOGOJ?editors=1111)，参考 [hash router](https://juejin.im/post/6844903589123457031) 。

其实知道了路由的原理，想要实现一个 hash 路由并不困难，比较需要注意的是 backOff 的实现，包括  [hash router](https://link.juejin.im/?target=https%3A%2F%2Fcodepen.io%2Fxiaomuzhu%2Fpen%2FmxQBod%2F) 中对 backOff 的实现也是有 bug 的，浏览器的回退会触发 `hashChange` 所以会在 `history` 中 push 一个新的路径，也就是每一步都将被记录。所以需要一个 `backIndex` 来作为返回的 index 的标识，在点击新的 URL 的时候再将 backIndex 回归为 `this.currentIndex`。

### 基于 history 的路由实现

```javascript
class Routers {
  constructor() {
    this.routes = {};
    // 在初始化时监听popstate事件
    this._bindPopState();
  }
  // 初始化路由
  init(path) {
    history.replaceState({path: path}, null, path);
    this.routes[path] && this.routes[path]();
  }
  // 将路径和对应回调函数加入hashMap储存
  route(path, callback) {
    this.routes[path] = callback || function() {};
  }

  // 触发路由对应回调
  go(path) {
    history.pushState({path: path}, null, path);
    this.routes[path] && this.routes[path]();
  }
  // 后退
  backOff(){
    history.back()
  }
  // 监听popstate事件
  _bindPopState() {
    window.addEventListener('popstate', e => {
      const path = e.state && e.state.path;
      this.routes[path] && this.routes[path]();
    });
  }
}
```

参考 [H5 Router](https://link.juejin.im/?target=https%3A%2F%2Fcodepen.io%2Fxiaomuzhu%2Fpen%2FQmJorQ%2F)

相比 hash 路由，h5 路由不再需要有些丑陋去的去修改 `window.location` 了，取而代之使用 `history.pushState` 来完成对 `window.location` 的操作，使用 `window.addEventListener('popstate', callback)` 来对前进/后退进行监听，至于后退则可以直接使用 `window.history.back()` 或者  `window.history.go(-1)` 来直接实现，由于浏览器的 history 控制了前进/后退的逻辑，所以实现简单了很多。

# React性能优化

**shouldComponentUpdate和PureComponent**

尽量将复杂类型数据（ArrayList）所关联的视图单独拆成PureComonent有助于提高渲染性能，比如表单、文本域和复杂列表在同一个 render() 中，表单域的输入字段改变会频繁地触发 setState() 从而导致 组件 重新 render()。而用于渲染复杂列表的数据其实并没有变化，但由于重新触发 render()，列表还是会重新渲染。

