# css（cascading style sheet）层叠样式表书写习惯

在给出 CSS 选择器方面的优化建议之前，先告诉大家一个小知识：CSS 引擎查找样式表，对每条规则都按从右到左的顺序去匹配。 看如下规则：

```
#myList  li {}
```

这样的写法其实很常见。大家平时习惯了从左到右阅读的文字阅读方式，会本能地以为浏览器也是从左到右匹配 CSS 选择器的，因此会推测这个选择器并不会费多少力气：#myList 是一个 id 选择器，它对应的元素只有一个，查找起来应该很快。定位到了 myList 元素，等于是缩小了范围后再去查找它后代中的 li 元素，没毛病。

事实上，**CSS 选择符是从右到左进行匹配的**。我们这个看似“没毛病”的选择器，实际开销相当高：浏览器必须遍历页面上每个 li 元素，并且每次都要去确认这个 li 元素的父元素 id 是不是 myList，你说坑不坑！

说到坑，不知道大家还记不记得这个经典的通配符：

```
* {}
```

入门 CSS 的时候，不少同学拿通配符清除默认样式（我曾经也是通配符用户的一员）。但这个家伙很恐怖，它会匹配所有元素，所以浏览器必须去遍历每一个元素！大家低头看看自己页面里的元素个数，是不是心凉了——这得计算多少次呀！

这样一看，一个小小的 CSS 选择器，也有不少的门道！好的 CSS 选择器书写习惯，可以为我们带来非常可观的性能提升。根据上面的分析，我们至少可以总结出如下性能提升的方案：

- 避免使用通配符，只对需要用到的元素进行选择。

- 关注可以通过继承实现的属性，避免重复匹配重复定义。

- 少用标签选择器。如果可以，用类选择器替代，举个🌰：

  错误示范：

  ```
  #myList li{}
  ```

  课代表：

  ```
  .myList_li {}
  ```

- 不要画蛇添足，id 和 class 选择器不应该被多余的标签选择器拖后腿。举个🌰：

  错误示范

  ```
  .myList#title
  ```

  课代表

  ```
  #title
  ```

- 减少嵌套。后代选择器的开销是最高的，因此我们应该尽量将选择器的深度降到最低（最高不要超过三层），尽可能使用类来关联每一个标签元素。

搞定了 CSS 选择器，万里长征才刚刚开始的第一步。但现在你已经理解了浏览器的工作过程，接下来的征程对你来说并不再是什么难题~

# css 实现一个自适应搜索框

要求输入框部分宽度自适应，搜索按钮宽度固定

个人感觉可以类比左右两栏布局，右边固定，左边自适应，代码之后补充。

```html

```

# 水平垂直居中

![img](https://user-gold-cdn.xitu.io/2019/10/13/16dc2f70c2c796c0?imageslim)

# position相关属性

## static

默认值。没有定位，元素出现在正常的流中（忽略 top, bottom, left, right 或者 z-index 声明）。

## relative

相对于原来位置移动，元素仍然在文档流中，不影响其他元素的布局

## absolute

元素会脱离文档流，如果设置偏移量，会影响其他元素的位置定位。

在父元素没有设置相对定位或绝对定位的情况下，元素相对于根元素定位（即html元素）（是父元素没有）。

父元素设置了相对定位或绝对定位，元素会相对于离自己最近的设置了相对或绝对定位的父元素进行定位（或者说离自己最近的不是static的父元素进行定位，因为元素默认是static）

## fixed

 生成绝对定位的元素，相对于浏览器窗口进行定位。元素的位置通过 "left", "top", "right" 以及 "bottom" 属性进行规定。

## sticky

***定义***
粘性定位可以被认为是相对定位（relative）和固定定位（fixed）的混合。元素在跨越特定阈值前为相对定位，之后为固定定位。
也就是说sticky会让元素在页面滚动时如同在正常流中（relative定位效果），但当滚动到特定位置时就会固定在屏幕上如同 fixed ，这个特定位置就是指定 的top, right, bottom 或 left 四个阈值其中之一
***作用域***
设置了sticky定位的元素相对于第一个定位不为static的父级元素的位置，sticky的作用区域也是在该父级元素的内。也就是说粘性布局的效果只在该父元素内表现出来。**这一点与fixed布局有区别**。

当页面刚开始滚动时item元素并无特殊的表现（相当于relative定位），当滚动到特殊位置（top: 0）时出现粘性定位效果（相当于fixed定位），继续滚动页面发现item元素并不是一直定位在顶部，**当其父元素不在视窗内时item元素失去粘性效果。这一点与fixed的表现不同。fixed定位元素是相对于整个视窗定位**。

# 浮动布局

当元素浮动以后可以向左或向右移动，直到它的外边缘碰到包含它的框或者另外一个浮动元素的边框为止。元素浮动以后会脱离正常的文档流，所以文档的普通流中的框就变现的好像浮动元素不存在一样。

## 优点

这样做的优点就是在图文混排的时候可以很好的使文字环绕在图片周围。另外当元素浮动了起来之后，它有着块级元素的一些性质例如可以设置宽高等，但它与inline-block还是有一些区别的，第一个就是关于横向排序的时候，float可以设置方向而inline-block方向是固定的；还有一个就是inline-block在使用时有时会有空白间隙的问题

## 缺点

最明显的缺点就是浮动元素一旦脱离了文档流，就无法撑起父元素，会造成父级元素高度塌陷。

## 清除浮动

1. 添加额外标签

```html
<div class="parent">
    //添加额外标签并且添加clear属性
    <div style="clear:both"></div>
    //也可以加一个br标签
</div>
```

2. 父级添加overflow属性，或者设置高度

```html
<div class="parent" style="overflow:hidden">//auto 也可以
    //将父元素的overflow设置为hidden
    <div class="f"></div>
</div>
```

3. 建立伪类选择器清除浮动（推荐）

```css
//在css中添加:after伪元素
.parent:after{
    /* 设置添加子元素的内容是空 */
      content: '';  
      /* 设置添加子元素为块级元素 */
      display: block;
      /* 设置添加的子元素的高度0 */
      height: 0;
      /* 设置添加子元素看不见 */
      visibility: hidden;
      /* 设置clear：both */
      clear: both;
}
<div class="parent">
    <div class="f"></div>
</div>
```

# 使用display:inline-block会产生什么问题？解决方法？

**问题复现**

问题: 两个display：inline-block元素放到一起会产生一段空白。

如代码:

```javascript
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
      .container {
        width: 800px;
        height: 200px;
      }

      .left {
        font-size: 14px;
        background: red;
        display: inline-block;
        width: 100px;
        height: 100px;
      }

      .right {
        font-size: 14px;
        background: blue;
        display: inline-block;
        width: 100px;
        height: 100px;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <div class="left">
        左
      </div>
      <div class="right">
        右
      </div>
    </div>
  </body>
</html>
```

效果如下:

![img](https://user-gold-cdn.xitu.io/2019/10/13/16dc2f7d81886473?imageslim)

**产生空白的原因**

元素被当成行内元素排版的时候，元素之间的空白符（空格、回车换行等）都会被浏览器处理，根据CSS中white-space属性的处理方式（默认是normal，合并多余空白），原来`HTML代码中的回车换行被转成一个空白符`，在字体不为0的情况下，空白符占据一定宽度，所以inline-block的元素之间就出现了空隙。

**解决办法**

1. 将子元素标签的结束符和下一个标签的开始符写在同一行或把所有子标签写在同一行

```html
<div class="container">
  <div class="left">
      左
  </div><div class="right">
      右
  </div>
</div>
```

2. 父元素中设置font-size: 0，在子元素上重置正确的font-size

```css
.container{
  width:800px;
  height:200px;
  font-size: 0;
}
```

3. 为子元素设置float:left

```css
.left{
  float: left;
  font-size: 14px;
  background: red;
  display: inline-block;
  width: 100px;
  height: 100px;
}
//right是同理
```

# div垂直居中，左右10px，高度始终为宽度一半

> 问题描述: 实现一个div垂直居中, 其距离屏幕左右两边各10px, 其高度始终是宽度的50%。同时div中有一个文字A，文字需要水平垂直居中。

**思路一：利用height:0; padding-bottom: 50%;**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
      *{
        margin: 0;
        padding: 0;
      }
      html, body {
        height: 100%;
        width: 100%;
      }
      .outer_wrapper {
        margin: 0 10px;
        height: 100%;
        /* flex布局让块垂直居中 */
        display: flex;
        align-items: center;
      }
      .inner_wrapper{
        background: red;
        position: relative;
        width: 100%;
        height: 0;
        padding-bottom: 50%;
      }
      .box{
        position: absolute;
        width: 100%;
        height: 100%;
        display: flex;
        justify-content: center;
        align-items: center;
        font-size: 20px;
      }
    </style>
  </head>
  <body>
    <div class="outer_wrapper">
      <div class="inner_wrapper">
        <div class="box">A</div>
      </div>
    </div>
  </body>
</html>
```

强调两点:

1. padding-bottom究竟是相对于谁的？

答案是相对于`父元素的width值`。

那么对于这个out_wrapper的用意就很好理解了。 CSS呈流式布局，div默认宽度填满，即100%大小，给out_wrapper设置margin: 0 10px;相当于让左右分别减少了10px。

1. 父元素相对定位，那绝对定位下的子元素宽高若设为百分比，是相对谁而言的？

相对于父元素的(content + padding)值, 注意不含border

> 延伸：如果子元素不是绝对定位，那宽高设为百分比是相对于父元素的宽高，标准盒模型下是content, IE盒模型是content+padding+border。

**思路二: 利用calc和vw**

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
      * {
        padding: 0;
        margin: 0;
      }

      html,
      body {
        width: 100%;
        height: 100%;
      }

      .wrapper {
        position: relative;
        width: 100%;
        height: 100%;
      }

      .box {
        margin-left: 10px;
        /* vw是视口的宽度， 1vw代表1%的视口宽度 */
        width: calc(100vw - 20px);
        /* 宽度的一半 */
        height: calc(50vw - 10px);
        position: absolute;
        background: red;
        /* 下面两行让块垂直居中 */
        top: 50%;
        transform: translateY(-50%);
        display: flex;
        align-items: center;
        justify-content: center;
        font-size: 20px;
      }
    </style>
  </head>
  <body>
    <div class="wrapper">
      <div class="box">A</div>
    </div>
  </body>
</html>
```

效果如下:



![img](https://user-gold-cdn.xitu.io/2019/10/13/16dc2f82defd2829?imageslim)



# CSS如何进行品字布局

## 第一种

```javascript
<!doctype html>
<html>

<head>
  <meta charset="utf-8">
  <title>品字布局</title>
  <style>
    * {
      margin: 0;
      padding: 0;
    }
    body {
      overflow: hidden;
    }
    div {
      margin: auto 0;
      width: 100px;
      height: 100px;
      background: red;
      font-size: 40px;
      line-height: 100px;
      color: #fff;
      text-align: center;
    }

    .div1 {
      margin: 100px auto 0;
    }

    .div2 {
      margin-left: 50%;
      background: green;
      float: left;
      transform: translateX(-100%);
    }

    .div3 {
      background: blue;
      float: left;
      transform: translateX(-100%);
    }
  </style>
</head>

<body>
  <div class="div1">1</div>
  <div class="div2">2</div>
  <div class="div3">3</div>
</body>

</html>
```

效果:

![img](https://user-gold-cdn.xitu.io/2019/10/13/16dc2f88cc9974e6?imageslim)

## 第二种(全屏版)

```html
<!doctype html>
<html>
  <head>
    <meta charset="utf-8">
    <title>品字布局</title>
    <style>
      * {
        margin: 0;
        padding: 0;
      }

      div {
        width: 100%;
        height: 100px;
        background: red;
        font-size: 40px;
        line-height: 100px;
        color: #fff;
        text-align: center;
      }

      .div1 {
        margin: 0 auto 0;
      }

      .div2 {
        background: green;
        float: left;
        width: 50%;
      }

      .div3 {
        background: blue;
        float: left;
        width: 50%;
      }
    </style>
  </head>

  <body>
    <div class="div1">1</div>
    <div class="div2">2</div>
    <div class="div3">3</div>
  </body>
</html>
```

效果:

![img](https://user-gold-cdn.xitu.io/2019/10/13/16dc2f8ecad5c2f6?imageslim)

# flex布局常见属性

## 容器的属性

- flex-direction: row | row-reverse | column | column-reverse;
- flex-wrap: nowrap | wrap | wrap-reverse;
- flex-flow:  <flex-direction> || <flex-wrap>;
- justify-content: flex-start | flex-end | center | space-between | space-around;
- align-items: flex-start | flex-end | center | baseline | stretch;
- align-content: flex-start | flex-end | center | space-between | space-around | stretch;

## 项目的属性

- `order`属性定义项目的排列顺序。数值越小，排列越靠前，默认为0。

- `flex-grow`属性定义项目的放大比例，默认为`0`，即如果存在剩余空间，也不放大。

- `flex-shrink`属性定义了项目的缩小比例，默认为1，即如果空间不足，该项目将缩小。

- `flex-basis`属性定义了在分配多余空间之前，项目占据的主轴空间（main size）。浏览器根据这个属性，计算主轴是否有多余空间。它的默认值为`auto`，即项目的本来大小。

- `flex`是`flex-grow`, `flex-shrink` 和 `flex-basis`的简写，默认值为`0 1 auto`。后两个属性可选。该属性有两个快捷值：`auto` (`1 1 auto`) 和 none (`0 0 auto`)。

  建议优先使用这个属性，而不是单独写三个分离的属性，因为浏览器会推算相关值。

- `align-self`属性允许单个项目有与其他项目不一样的对齐方式，可覆盖`align-items`属性。默认值为`auto`，表示继承父元素的`align-items`属性，如果没有父元素，则等同于`stretch`。

# 两栏布局（左边宽度固定，右边自适应）

## 双inline-block方案

```html
<div class="wrapper" id="wrapper">
  <div class="left">
    左边固定宽度，高度不固定 </br> </br></br></br>高度有可能会很小，也可能很大。
  </div>
  <div class="right">
    这里的内容可能比左侧高，也可能比左侧低。宽度需要自适应。</br>
    基本的样式是，两个div相距20px, 左侧div宽 120px
  </div>
</div>
```

```css
.wrapper {
    padding: 15px 20px;
    border: 1px dashed #ff6c60;
  	box-sizing: content-box;
  	font-size: 0;
}
.left {
    width: 120px;
    border: 5px solid #ddd;
  	display: inline-block;
  	vertical-align: top;
  	font-size: 14px;
  	box-sizing: border-box;
}
.right {
    margin-left: 20px;
    border: 5px solid #ddd;
  	display: inline-block;
  	vertical-align: top;
  	font-size: 14px;
  	box-sizing: border-box;
  	width: calc(100% - 140px);
}
```

这种方法是通过`width: calc(100% - 140px)`来动态计算右侧盒子的宽度。需要知道右侧盒子距离左边的距离，以及左侧盒子具体的宽度(content+padding+border)，以此计算父容器宽度的`100%`需要减去的数值。同时，还需要知道右侧盒子的宽度是否包含`border`的宽度。
在这里，为了简单的计算右侧盒子准确的宽度，设置了子元素的`box-sizing:border-box;`以及父元素的`box-sizing: content-box;`。
同时，作为两个`inline-block`的盒子，必须设置`vertical-align`来使其顶端对齐。
另外，为了**准确地应用**计算出来的宽度，需要消除`div`之间的空格，需要通过设置父容器的`font-size: 0;`,或者用注释消除`html`中的空格等方法。
**缺点:**

- 需要知道左侧盒子的宽度，两个盒子的距离，还要设置各个元素的`box-sizing`
- 需要消除空格字符的影响
- 需要设置`vertical-align: top`满足顶端对齐。

## 双float方案

```html
<div class="wrapper" id="wrapper">
  <div class="left">
    左边固定宽度，高度不固定 </br> </br></br></br>高度有可能会很小，也可能很大。
  </div>
  <div class="right">
    这里的内容可能比左侧高，也可能比左侧低。宽度需要自适应。</br>
    基本的样式是，两个div相距20px, 左侧div宽 120px
  </div>
</div>
```

```css
.wrapper {
    padding: 15px 20px;
    border: 1px dashed #ff6c60;
  	overflow: hidden;
  	box-sizing: content-box;
}
.left {
    width: 120px;
    border: 5px solid #ddd;
  	float: left;
  	vertical-align: top;
  	font-size: 14px;
  	box-sizing: border-box;
}
.right {
    margin-left: 20px;
    border: 5px solid #ddd;
  	float: left;
  	vertical-align: top;
  	font-size: 14px;
  	box-sizing: border-box;
  	width: calc(100% - 140px);
}
```

本方案和双`float`方案原理相同，都是通过动态计算宽度来实现自适应。但是，由于浮动的`block`元素在有空间的情况下会[依次紧贴，排列在一行](https://www.w3.org/TR/CSS21/visuren.html#bfc-next-to-float)，所以无需设置`display: inline-block;`，自然也就少了顶端对齐，空格字符占空间等问题。

> A floated box is shifted to the left or right until its outer edge touches the containing block edge or the outer edge of another float.

不过由于应用了浮动，父元素需要清除浮动。
**缺点:**

- 需要知道左侧盒子的宽度，两个盒子的距离，还要设置各个元素的`box-sizing`。
- 父元素需要清除浮动。

## float+margin-left方案

```html
<div class="wrapper" id="wrapper">
  <div class="left">
    左边固定宽度，高度不固定 </br> </br></br></br>高度有可能会很小，也可能很大。
  </div>
  <div class="right">
    这里的内容可能比左侧高，也可能比左侧低。宽度需要自适应。</br>
    基本的样式是，两个div相距20px, 左侧div宽 120px
  </div>
</div>
```

```css
.wrapper {
    padding: 15px 20px;
    border: 1px dashed #ff6c60;
  	overflow: hidden;
}
.left {
    width: 120px;
    border: 5px solid #ddd;
  	float: left;
}
.right {
    margin-left: 20px;
    border: 5px solid #ddd;
		margin-left: 150px;
}
```

上面两种方案都是利用了CSS的`calc()`函数来计算宽度值。下面两种方案则是利用了`block`级别的元素盒子的宽度具有**填满父容器，并随着父容器的宽度自适应**的**流动特性**。
但是`block`级别的元素都是独占一行的，所以要想办法让两个`block`排列到一起。
我们知道，`block`级别的元素会认为浮动的元素不存在，但是`inline`级别的元素能识别到浮动的元素。这样，`block`级别的元素就可以和浮动的元素同处一行了。
为了让右侧盒子和左侧盒子保持距离，需要为左侧盒子留出足够的距离。这个距离的大小为左侧盒子的宽度以及两个盒子之间的距离之和。然后将该值设置为右侧盒子的`margin-left`。
**缺点：**

- 需要清除浮动
- 需要计算右侧盒子的`margin-left`

## absolute+margin-left方案

```html
<div class="wrapper" id="wrapper">
  <div class="left">
    左边固定宽度，高度不固定 </br> </br></br></br>高度有可能会很小，也可能很大。
  </div>
  <div class="right">
    这里的内容可能比左侧高，也可能比左侧低。宽度需要自适应。</br>
    基本的样式是，两个div相距20px, 左侧div宽 120px
  </div>
</div>
```

```css
.wrapper {
    padding: 15px 20px;
    border: 1px dashed #ff6c60;
  	overflow: hidden;
  	position: relative;
}
.left {
    width: 120px;
    border: 5px solid #ddd;
  	position: absolute;
}
.right {
    margin-left: 20px;
    border: 5px solid #ddd;
		margin-left: 150px;
}
```

**缺点:**

- 使用了绝对定位，若是用在某个div中，需要更改父容器的`position`。
- 没有清除浮动的方法，若左侧盒子高于右侧盒子，就会超出父容器的高度。因此只能通过设置父容器的`min-height`来放置这种情况。

## float+BFC方案

```html
<div class="wrapper" id="wrapper">
        <div class="left">
          左边固定宽度，高度不固定 </br> </br></br></br>高度有可能会很小，也可能很大。
        </div>
        <div class="right">
          这里的内容可能比左侧高，也可能比左侧低。宽度需要自适应。</br>
          基本的样式是，两个div相距20px, 左侧div宽 120px
        </div>
</div>
```

```css
.wrapper {
    padding: 15px 20px;
    border: 1px dashed #ff6c60;
    overflow: auto;
}
.left {
    width: 120px;
    border: 5px solid #ddd;
  	float: left;
    margin-right: 20px;
}
.right {
    margin-left: 20px;
    border: 5px solid #ddd;
    margin-left: 0;
    overflow: auto;
}
```

这个方案同样是利用了左侧浮动，但是右侧盒子通过`overflow: auto;`形成了BFC，因此右侧盒子不会与浮动的元素重叠。

> an element in the normal flow that establishes a new block formatting context (such as an element with 'overflow' other than 'visible') must not overlap the margin box of any floats in the same block formatting context as the element itself。

这种情况下，只需要为左侧的浮动盒子设置`margin-right`，就可以实现两个盒子的距离了。而右侧盒子是`block`级别的，所以宽度能实现自适应。
**缺点:**

- 父元素需要清除浮动

## flex方案

```html
<div class="wrapper" id="wrapper">
        <div class="left">
          左边固定宽度，高度不固定 </br> </br></br></br>高度有可能会很小，也可能很大。
        </div>
        <div class="right">
          这里的内容可能比左侧高，也可能比左侧低。宽度需要自适应。</br>
          基本的样式是，两个div相距20px, 左侧div宽 120px
        </div>
</div>
```

```css
.wrapper {
    padding: 15px 20px;
    border: 1px dashed #ff6c60;
    display: flex;
  	align-items: flex-start;
}
.left {
    width: 120px;
    border: 5px solid #ddd;
    flex: 0 0 auto;
}
.right {
    margin-left: 20px;
    border: 5px solid #ddd;
    flex: 1 1 auto;
}
```

flex可以说是最好的方案了，代码少，使用简单。有朝一日，大家都改用现代浏览器，就可以使用了。
需要注意的是，flex容器的一个默认属性值:align-items: stretch;。这个属性导致了列等高的效果。
为了让两个盒子高度自动，需要设置: align-items: flex-start;

## grid方案

```html
<div class="wrapper" id="wrapper">
        <div class="left">
          左边固定宽度，高度不固定 </br> </br></br></br>高度有可能会很小，也可能很大。
        </div>
        <div class="right">
          这里的内容可能比左侧高，也可能比左侧低。宽度需要自适应。</br>
          基本的样式是，两个div相距20px, 左侧div宽 120px
        </div>
</div>
```

```css
.wrapper {
    padding: 15px 20px;
    border: 1px dashed #ff6c60;
    display: grid;
  	align-items: start;
  	grid-template-columns: 120px 1fr;
}
.left {
    width: 120px;
    border: 5px solid #ddd;
    box-sizing: border-box;
    grid-column: 1;
}
.right {
    margin-left: 20px;
    border: 5px solid #ddd;
    box-sizing: border-box;
    grid-column: 2;
}
```

**注意:**

- `grid`布局也有列等高的默认效果。需要设置: ` align-items: start;`。
- `grid`布局还有一个值得注意的小地方和`flex`不同:在使用`margin-left`的时候，`grid`布局默认是`box-sizing`设置的盒宽度之间的位置。而`flex`则是使用两个div的`border`或者`padding`外侧之间的距离。

# 三栏布局（左右宽度固定，中间自适应）

## 绝对定位法

```html
<div class="outer">
        <div class="left"></div>
        <div class="right"></div>
        <div class="center"></div>    
</div>
```

```css
.outer {
  position: relative;
}
.left {
  position: absolute;
  left: 0;
  width: 200px;
  height: 200px;
  background-color: antiquewhite;
}
.right {
  position: absolute;
  right: 0;
  width: 200px;
  height: 200px;
  background-color: antiquewhite;
}
.center {
  margin: 2 210px;
  background-color: pink;
  height: 200px;
}
```

## float法

```css
.left {
float: left;
width: 200px;
height: 200px;
background-color: antiquewhite;
}
.right {
float: right;
width: 200px;
height: 200px;
background-color: antiquewhite;
}
.center {
margin: 2 210px;
background-color: pink;
height: 200px;
}
```

## 圣杯布局

比较特殊的三栏布局，同样也是两边固定宽度，中间自适应，唯一区别是dom结构必须是先写中间列部分，这样实现中间列可以优先加载。

```css
  .container {
    padding-left: 220px;//为左右栏腾出空间
    padding-right: 220px;
  }
  .left {
    float: left;
    width: 200px;
    height: 400px;
    background: red;
    margin-left: -100%;
    position: relative;
    left: -220px;
  }
  .center {
    float: left;
    width: 100%;
    height: 500px;
    background: yellow;
  }
  .right {
    float: left;
    width: 200px;
    height: 400px;
    background: blue;
    margin-left: -200px;
    position: relative;
    right: -220px;
  }
```

```html
  <article class="container">
    <div class="center">
      <h2>圣杯布局</h2>
    </div>
    <div class="left"></div>
    <div class="right"></div>
  </article>

```

- 三个部分都设定为左浮动，**否则左右两边内容上不去，就不可能与中间列同一行**。然后设置center的宽度为100%(**实现中间列内容自适应**)，此时，left和right部分会跳到下一行

![img](https://user-gold-cdn.xitu.io/2018/10/18/16682cae82722a6a?imageslim)

- 通过设置margin-left为负值让left和right部分回到与center部分同一行

![img](https://user-gold-cdn.xitu.io/2018/10/18/16682c1d72a1ea68?imageslim)

- 通过设置父容器的padding-left和padding-right，让左右两边留出间隙。

![img](https://user-gold-cdn.xitu.io/2018/10/18/16682c473f605745?imageslim)

- 通过设置相对定位，让left和right部分移动到两边。

![img](https://user-gold-cdn.xitu.io/2018/10/17/16682bf3615502c2?imageslim)

 缺点

- center部分的最小宽度不能小于left部分的宽度，否则会left部分掉到下一行
- 如果其中一列内容高度拉长(如下图)，其他两列的背景并不会自动填充。

## 双飞翼布局

```css
.container {
  min-width: 600px;//确保中间内容可以显示出来，两倍left宽+right宽
}
.left {
  float: left;
  width: 200px;
  height: 400px;
  background: red;
  margin-left: -100%;
}
.center {
  float: left;
  width: 100%;
  height: 500px;
  background: yellow;
}
.center .inner {
  margin: 0 200px; //新增部分
}
.right {
  float: left;
  width: 200px;
  height: 400px;
  background: blue;
  margin-left: -200px;
}
```

```html
<article class="container">
  <div class="center">
    <div class="inner">双飞翼布局</div>
  </div>
  <div class="left"></div>
  <div class="right"></div>
</article>
```

## flex布局

```html
<div id = "box">
        <div id = "left_box"></div>
        <div id = "center_box"></div>
        <div id = "right_box"></div>
</div>
```

```css
#box{width:100%;display: flex; height: 100px;margin: 10px;}
#left_box,#right_box{width: 200px;height: 100px; margin: 10px; background-color: lightpink}
#center_box{ flex:1; height: 100px;margin: 10px; background-color: lightgreen}
```

# 多列等高

HTML结构代码如下所示：

```html
  <ul class="Article">
    <li class="js-article-item">
      <p>
      一家将客户利益置于首位的经纪商，
      为客户提供专业的交易工具一家将客户利益置于首位的经纪商，
      为客户提供专业的交易工具一家将客户利益置于首位的经纪商，
      为客户提供专业的交易工具一家将客户利益置于首位的经纪商，为客户提供专业的交易工具
      </p>
    </li>
    <li class="js-article-item">
      <p>一家将客户利益置于首位的经纪商，为客户提供专业的交易工具
      一家将客户利益置于首位的经纪商，为客户提供专业的交易工具</p>
    </li>
    <li class="js-article-item">
      <p>一家将客户利益置于首位的经纪商</p>
    </li>
  </ul>
```

**1、使用负margin-bottom和正padding-bottom对冲实现**

```css
.Article {
  overflow: hidden;
}

.Article>li {
  float: left;
  margin: 0 10px -9999px 0;
  padding-bottom: 9999px;
  background: #4577dc;
  width: 200px;
  color: #fff;
}

.Article>li>p {	
  padding: 10px;
}
```

元素设置的padding-bottom尽可能大一些，并且需要设置一样大小的margin-bottom负值去抵消padding-bottom撑大的区域，正负一抵消，对于页面布局不会有影响。另外的话还需要设置父元素overflow：hidden把子元素多出来的色块背景隐藏掉。

**2、模仿table布局**

```css
.Article {
  display: table;
  width: 100%;
  table-layout: fixed;
}

.Article>li {
  display: table-cell;
  width: 200px;
  border-left: solid 5px currentColor;
  border-right: solid 5px currentColor;
  color: #fff;
  background: #4577dc;
}

.Article>li>p {
  padding: 10px;
}
```

table元素中的table-cell元素默认就是等高的。

**3、flex布局**

```css
.Article {
  display: flex;
}

.Article>li {
  flex: 1;
  border-left: solid 5px currentColor;
  border-right: solid 5px currentColor;
  color: #fff;
  background: #4577dc;
}

.Article>li>p {
  padding: 10px;
}
```

flex中的伸缩项目默认为拉伸为父元素的高度，同样可以实现等高效果。在pc端兼容性不是很好，在ie9以及ie9以下不支持。

# BFC

## 定义

浮动元素和绝对定位元素，非块级盒子的块级容器（例如 inline-blocks, table-cells, 和 table-captions），以及overflow值不为"visiable"的块级盒子，都会为他们的内容创建新的BFC（Block Fromatting Context， 即块级格式上下文）。

## 触发条件

一个HTML元素要创建BFC，则满足下列的任意一个或多个条件即可： 下列方式会创建块格式化上下文：

- 根元素()
- 浮动元素（元素的 float 不是 none）
- 绝对定位元素（元素的 position 为 absolute 或 fixed）
- 行内块元素（元素的 display 为 inline-block）
- 表格单元格（元素的 display为 table-cell，HTML表格单元格默认为该值）
- 表格标题（元素的 display 为 table-caption，HTML表格标题默认为该值）
- 匿名表格单元格元素（元素的 display为 table、table-row、 table-row-group、table-header-group、table-footer-group（分别是HTML table、row、tbody、thead、tfoot的默认属性）或 inline-table）
- overflow 值不为 visible 的块元素 -弹性元素（display为 flex 或 inline-flex元素的直接子元素）
- 网格元素（display为 grid 或 inline-grid 元素的直接子元素） 等等。

## BFC渲染规则

（1）BFC垂直方向边距重叠

（2）BFC的区域不会与浮动元素的box重叠

（3）BFC是一个独立的容器，外面的元素不会影响里面的元素

（4）计算BFC高度的时候浮动元素也会参与计算

## 应用场景

**1. 防止浮动导致父元素高度塌陷**

现有如下页面代码:

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <meta http-equiv="X-UA-Compatible" content="ie=edge">
    <title>Document</title>
    <style>
      .container {
        border: 10px solid red;
      }
      .inner {
        background: #08BDEB;
        height: 100px;
        width: 100px;
      }
    </style>
  </head>
  <body>
    <div class="container">
      <div class="inner"></div>
    </div>
  </body>
</html>
```

![img](https://user-gold-cdn.xitu.io/2019/10/13/16dc2dc952e4179b?imageslim)

接下来将inner元素设为浮动:

```css
  .inner {
    float: left;
    background: #08BDEB;
    height: 100px;
    width: 100px;
  }
```

会产生这样的塌陷效果：

![img](https://user-gold-cdn.xitu.io/2019/10/13/16dc2de4ca399fc2?imageslim)



但如果我们对父元素设置BFC后, 这样的问题就解决了:

```css
.container {
    border: 10px solid red;
    overflow: hidden;
}
```

同时这也是清除浮动的一种方式。

**2. 避免外边距折叠**

两个块同一个BFC会造成外边距折叠，但如果对这两个块分别设置BFC，那么边距重叠的问题就不存在了。

现有代码如下:

```html
<!DOCTYPE html>
<html lang="en">

<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <meta http-equiv="X-UA-Compatible" content="ie=edge">
  <title>Document</title>
  <style>
    .container {
      background-color: green;
      overflow: hidden;
    }

    .inner {
      background-color: lightblue;
      margin: 10px 0;
    }
  </style>
</head>

<body>
  <div class="container">
    <div class="inner">1</div>
    <div class="inner">2</div>
    <div class="inner">3</div>
  </div>
</body>

</html>
```

![img](https://user-gold-cdn.xitu.io/2019/10/13/16dc2e49602d3a99?imageslim)

此时三个元素的上下间隔都是10px, 因为三个元素同属于一个BFC。 现在我们做如下操作:

```html
  <div class="container">
    <div class="inner">1</div>
    <div class="bfc">
      <div class="inner">2</div>
    </div>
    <div class="inner">3</div>
  </div>
```

style增加:

```css
.bfc{
    overflow: hidden;
}
```

效果如下:

![img](https://user-gold-cdn.xitu.io/2019/10/13/16dc2eb0f144f2c2?imageslim)

可以明显地看到间隔变大了，而且是原来的两倍，符合我们的预期。

# CSS实现圆形、扇形、三角形、梯形

## 圆形

`border-radius:50%`

## 扇形

```css
.sector {
	border-radius: 80px 0 0 0;
	width: 80px;
	height: 80px;
	background: pink;
}
```

```html
<>
  <style>
    #demo {
      position: relative;
      width: 50px;
      height: 50px;
      overflow: hidden;
    }
    #circle {
      position: absolute;
      width: 100px;
      height: 100px;
      background-color: black;
      border-radius: 50%;
    }
  </style>
  <body>
    <div id="demo">
      <div id="circle"></div>
    </div>
  </body>
</>
```

## 三角形

```css
.triangle {
  height: 0;
  width: 0;
  border: 40px solid transparent;
  border-bottom: 40px solid pink;
}
```

## 梯形

```css
.tixing {
            height: 0;
            width: 400px;
            border-top: 40px solid black;
            border-left: 40px solid transparent;
            border-right: 40px solid transparent;
        }
```

# 响应式布局

响应式布局就是实现不同屏幕分辨率的终端上浏览网页的不同展示方式。通过响应式设计能使网站在手机和平板电脑上有更好的浏览阅读体验。换句话说就是一个网站能够兼容多个终端，而不是为了每一个终端做一个特定的版本。

## 媒体查询

@media screen and (min-width: 600px) and (max-width:100px)

# 盒模型

## 标准盒模型

标准（W3C）盒子模型：`width` = 内容宽度`（content） + border + padding + margin`

![img](https://user-gold-cdn.xitu.io/2020/5/30/172633c783abc1eb?imageslim)

Box-sizing: content-box;

## IE盒模型

低版本IE盒子模型： `width` = 内容宽度`（content + border + padding）+ margin`

![img](https://user-gold-cdn.xitu.io/2020/5/30/17263443113eb879?imageslim)

box-sizing: border-box;

# CSS3新特性

https://juejin.im/post/5a0c184c51882531926e4294

新增各种CSS选择器	（:not(.input)：所有class不是“input”的节点）
圆角		（border-radius:8px）
多列布局	（multi-columnlayout）
阴影和反射	（Shadow\Reflect）
文字特效		（text-shadow）
文字渲染		（Text-decoration）
线性渐变		（gradient）
旋转			（transform）
缩放，定位，倾斜，动画，多背景
例如：transform:\scale(0.85,0.90)\translate(0px,-30px)\skew(-9deg,0deg)\Animation:

## CSS3动画

| [@keyframes](https://www.w3school.com.cn/cssref/pr_keyframes.asp) | 规定动画。                                               | 3    |
| ------------------------------------------------------------ | -------------------------------------------------------- | ---- |
| [animation](https://www.w3school.com.cn/cssref/pr_animation.asp) | 所有动画属性的简写属性，除了 animation-play-state 属性。 | 3    |
| [animation-name](https://www.w3school.com.cn/cssref/pr_animation-name.asp) | 规定 @keyframes 动画的名称。                             | 3    |
| [animation-duration](https://www.w3school.com.cn/cssref/pr_animation-duration.asp) | 规定动画完成一个周期所花费的秒或毫秒。默认是 0。         | 3    |
| [animation-timing-function](https://www.w3school.com.cn/cssref/pr_animation-timing-function.asp) | 规定动画的速度曲线。默认是 "ease"。                      | 3    |
| [animation-delay](https://www.w3school.com.cn/cssref/pr_animation-delay.asp) | 规定动画何时开始。默认是 0。                             | 3    |
| [animation-iteration-count](https://www.w3school.com.cn/cssref/pr_animation-iteration-count.asp) | 规定动画被播放的次数。默认是 1。                         | 3    |
| [animation-direction](https://www.w3school.com.cn/cssref/pr_animation-direction.asp) | 规定动画是否在下一周期逆向地播放。默认是 "normal"。      | 3    |
| [animation-play-state](https://www.w3school.com.cn/cssref/pr_animation-play-state.asp) | 规定动画是否正在运行或暂停。默认是 "running"。           | 3    |
| [animation-fill-mode](https://www.w3school.com.cn/cssref/pr_animation-fill-mode.asp) | 规定对象动画时间之外的状态。                             | 3    |

# 实现九宫格布局（float、grid、flex）

https://www.cnblogs.com/padding1015/p/9566443.html

1、css3选择器nth-child()

```html
<ul class="lists">
    <li class="list list1">1</li>
    <li class="list list2">2</li>
    <li class="list list3">3</li>
    <li class="list list4">4</li>
    <li class="list list5">5</li>
    <li class="list list6">6</li>
    <li class="list list7">7</li>
    <li class="list list8">8</li>
    <li class="list list9">9</li>
  </ul>
```

```css
 ul,li{
      list-style: none;
      overflow: hidden;
    }
    ul{
      width: 620px;
    }
    li.list{
      float: left;
      width: 200px;
      height: 200px;
      margin-right: 10px;
      margin-bottom: 10px;
      background: #eee;
    }
    li:nth-child(3n){
      margin-right: 0;
    }

```

2、grid布局

```html
<div class="wrapper">
	<div class="list list1">
		1
	</div>
	<div class="list list2">
		2
	</div>
	<div class="list list3">
		3
	</div>
	<div class="list list4">
		4
	</div>
	<div class="list list5">
		5
	</div>
	<div class="list list6">
		6
	</div>
	<div class="list list7">
		7
	</div>
	<div class="list list8">
		8
	</div>
	<div class="list list9">
		9
	</div>
</div>
```

```css
.wrapper{
     display: grid;
     grid-template-columns: 100px 100px 100px;
     grid-template-rows: 100px 100px 100px;
}
 .list{
     background: #eee;
}
 .list:nth-child(odd){
     background: #999;
}   
```

3、flex布局

```html
<!DOCTYPE html>
<html>
<style>
.block {
    padding-top: 30%;
    margin-top: 3%;
    border-radius: 10%;
    background-color: orange;
    width: 30%;
}
.container-flex2  {
    display: flex;
    flex-wrap: wrap;
    justify-content: space-around;
}
</style>
<body>
   <div class="container-flex2">
        <div class="block"></div>
        <div class="block"></div>
        <div class="block"></div>
        <div class="block"></div>
        <div class="block"></div>
        <div class="block"></div>
        <div class="block"></div>
        <div class="block"></div>
        <div class="block"></div>
    </div>
</body>
</html>
```

# 移动端

## 1px问题

1. 利用 css 中的`transfrom：scaleY(0.5)`
2. 设置 媒体查询根据不同 DPR 缩放

```less
.border(
    @borderWidth: 1px; 
    @borderStyle: solid; 
    @borderColor: @lignt-gray-color; 
    @borderRadius: 0) {
    position: relative;
    &:before {
        content: '';
        position: absolute;
        width: 98%;
        height: 98%;
        top: 0;
        left: 0;
        transform-origin: left top;
        -webkit-transform-origin: left top;
        box-sizing: border-box;
        pointer-events: none;
    }
    @media (-webkit-min-device-pixel-ratio: 2) {
        &:before {
            width: 200%;
            height: 200%;
            -webkit-transform: scale(.5);
        }
    }
    @media (-webkit-min-device-pixel-ratio: 2.5) {
        &:before {
            width: 250%;
            height: 250%;
            -webkit-transform: scale(.4);
        }
    }
    @media (-webkit-min-device-pixel-ratio: 2.75) {
        &:before {
            width: 275%;
            height: 275%;
            -webkit-transform: scale(1 / 2.75);
        }
    }
    @media (-webkit-min-device-pixel-ratio: 3) {
        &:before {
            width: 300%;
            height: 300%;
            transform: scale(1 / 3);
            -webkit-transform: scale(1 / 3);
        }
    }
    .border-radius(@borderRadius);
    &:before {
        border-width: @borderWidth;
        border-style: @borderStyle;
        border-color: @borderColor;
    }
}

.border-all(
	@borderWidth: 1px; 
	@borderStyle: solid; 
	@borderColor: @lignt-gray-color; 
	@borderRadius: 0) {
    .border(@borderWidth; @borderStyle; @borderColor; @borderRadius);
}
```



# CSS预编译

less

less基本语法

@

&

嵌套语法

# CSS选择器

选择器：1.id选择器（ # myid）
2.类选择器（.myclassname）
3.标签选择器（div, h1, p）
4.相邻选择器（h1 + p）
5.子选择器（ul > li）
6.后代选择器（li a）
7.通配符选择器（ * ）
8.属性选择器（a[rel = “external”]）
9.伪类选择器（a: hover, li: nth – child）

# CSS可继承的属性

1、字体系列属性

　　font-family：字体系列

　　font-weight：字体的粗细

　　font-size：字体的大小

　　font-style：字体的风格

2、文本系列属性

　　text-indent：文本缩进

　　text-align：文本水平对齐

　　line-height：行高

　　word-spacing：单词之间的间距

　　letter-spacing：中文或者字母之间的间距

　　text-transform：控制文本大小写（就是uppercase、lowercase、capitalize这三个）

　　color：文本颜色

3、元素可见性：

　　visibility：控制元素显示隐藏

4、列表布局属性：

　　list-style：列表风格，包括list-style-type、list-style-image等

5、光标属性：

　　cursor：光标显示为何种形态

# CSS 优先级算法如何计算？

判断优先级时，首先我们会判断一条属性声明是否有权重，也就是是否在声明后面加上了!important。一条声明如果加上了权重，
那么它的优先级就是最高的，前提是它之后不再出现相同权重的声明。如果权重相同，我们则需要去比较匹配规则的特殊性。

一条匹配规则一般由多个选择器组成，一条规则的特殊性由组成它的选择器的特殊性累加而成。选择器的特殊性可以分为四个等级，
第一个等级是行内样式，为1000，第二个等级是id选择器，为0100，第三个等级是类选择器、伪类选择器和属性选择器，为0010，
第四个等级是元素选择器和伪元素选择器，为0001。规则中每出现一个选择器，就将它的特殊性进行叠加，这个叠加只限于对应的等
级的叠加，不会产生进位。选择器特殊性值的比较是从左向右排序的，也就是说以1开头的特殊性值比所有以0开头的特殊性值要大。
比如说特殊性值为1000的的规则优先级就要比特殊性值为0999的规则高。如果两个规则的特殊性值相等的时候，那么就会根据它们引
入的顺序，后出现的规则的优先级最高。

# css实现顶部固定

```html
<!DOCTYPE html>
<html lang="zh">

<head>
    <meta charset="UTF-8">
    <title>顶部固定，中部自适应</title>
    <style type="text/css">
        * {
            margin: 0;
            padding: 0;
        }
        
        .hd {
            width: 100%;
            height: 50px;
            padding: 15px 0;
            text-align: center;
            color: #fff;
            background: #f00;
        }
        
        .bd {
            position: absolute;
            top: 50px;
            /* 往下移顶部的距离 */
            bottom: 0;
            /* 底部贴边 */
            overflow: auto;
            /* 溢出的部分自适应 */
            /* 不设置高度 */
            width: 100%;
            padding: 15px 0;
            text-align: center;
            color: #fff;
            background: #00f;
        }
    </style>
</head>

<body>
    <header class="hd">
        我是固定头部
    </header>
    <div class="bd">
        我是中部部分
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
        <p>超出的部分滚动</p>
    </div>
</body>

</html>
```

# SVG和Canvas的区别

- **Canvas：** 依赖分辨率；不支持事件处理器；弱的文本渲染能力；能够以 .png 或 .jpg 格式保存结果图像；最适合图像密集型的游戏，其中的许多对象会被频繁重绘
- **SVG：** 不依赖分辨率；支持事件处理器；最适合带有大型渲染区域的应用程序（比如谷歌地图）；复杂度高会减慢渲染速度（任何过度使用 DOM 的应用都不快）；不适合游戏应用

# 宽高比4:3自适应

- **垂直方向的padding**: 在css中，padding-top或padding-bottom的百分比值是根据容器的width来计算的。

```
.wrap{
       position: relative;
       height: 0;  //容器的height设置为0
       width: 100%;
       padding-top: 75%;  //100%*3/4
}
.wrap > *{
       position: absolute;//容器的内容的所有元素absolute,然子元素内容都将被padding挤出容器
       left: 0;
       top: 0;
       width: 100%;
       height: 100%;
}
```

- **padding & calc()**: 跟第一种方法原理相同

```
padding-top: calc(100%*9/16);
```

- **padding & 伪元素**
- **视窗单位**： 浏览器100vw表示浏览器的视窗宽度

```
	width:100vw;
	height:calc(100vw*3/4)
```

# 如何用 css 或 js 实现多行文本溢出省略效果，考虑兼容性？

- 这样写，就算在不是webkit内核的浏览器，也可以 **优雅降级**（高度=行高*行数（webkit-line-clamp）

```html
<!-- html代码 -->

<p class="single">该文的主题思想即对自由境界的向往。朱自清当时虽置身在污浊黑暗的旧中国，但他的心灵世界则是一片澄澈明净，他的精神依然昂奋向上。朱自清把他健康高尚的审美情趣，把他对美好事物的无限热爱，将他对人生理想的不懈追求熔铸到文章中去。熔铸到诗一样美丽的语言中去。从而使整篇文章洋溢着浓浓的诗意，产生了经久不衰的艺术魅力<p>

/*css代码*/
/*单行*/
.single{
    width: 800px;
    text-overflow: ellipsis;
    white-space: nowrap;
    overflow: hidden;    
}

/*多行*/
.single{
  	width: 800px;
    overflow: hidden;
    text-overflow: ellipsis;
    display: -webkit-box;
    -webkit-line-clamp: 2;
    -webkit-box-orient:vertical;
     flex-direction: column;
}
```

- js实现： 通过scrollHeight与clientHeight判断是否溢出

```
const p = document.querySelector('p')
let words = p.innerHTML.split(/(?<=[\u4e00-\u9fa5])|(?<=\w*?\b)/g)
while (p.scrollHeight > p.clientHeight) {
  words.pop()
  p.innerHTML = words.join('') + '...'
}

```

# animation属性

| *[animation-name](https://www.w3school.com.cn/cssref/pr_animation-name.asp)* | 规定需要绑定到选择器的 keyframe 名称。。 |
| ------------------------------------------------------------ | ---------------------------------------- |
| *[animation-duration](https://www.w3school.com.cn/cssref/pr_animation-duration.asp)* | 规定完成动画所花费的时间，以秒或毫秒计。 |
| *[animation-timing-function](https://www.w3school.com.cn/cssref/pr_animation-timing-function.asp)* | 规定动画的速度曲线。                     |
| *[animation-delay](https://www.w3school.com.cn/cssref/pr_animation-delay.asp)* | 规定在动画开始之前的延迟。               |
| *[animation-iteration-count](https://www.w3school.com.cn/cssref/pr_animation-iteration-count.asp)* | 规定动画应该播放的次数。                 |
| *[animation-direction](https://www.w3school.com.cn/cssref/pr_animation-direction.asp)* | 规定是否应该轮流反向播放动画。           |

# 伪类和伪元素？

**伪类可以理解为是一种状态，而伪元素则代表一些实实在在存在的元素，但是它们都是抽象刻画的，游离于标准文档之外。**

伪类存在的意义是为了通过选择器，格式化dom树以外的信息（:visited,:link），以及**不能被常规CSS选择器获取到的信息**（比如说获取第一个子元素，常规css选择器不行,可以用：first-child）。

伪类常用的有`first-child、last-child、nth-child、first-of-type`(父元素第一个特定的子元素)、`last-of-type、nth-of-type、lang、focus`、lvha（a标签四个）

伪元素可以**创建一些文档语言无法创建的虚拟元素**，比如文档语言没有一种机制可以描述元素内容第一个字母或者第一行，但是伪元素可以`::first-letter,::first-line`。同时伪元素还可以创建文档中不存在的内容比如说`::after,::before`。

伪元素主要有：

```
::after,::before,::first-letter,::first-line,::selection
```

# Rem,em和px的区别

px是绝对长度单位。

em是相对长度单位，继承父级元素的字体大小，所有字体都是相对于父元素大小的

rem也是相对长度单位，但它是相对于根元素（html),不会像em造成混乱。

**其实rem布局的本质是等比缩放，一般是基于宽度**，试想一下如果UE图能够等比缩放，那该多么美好啊

rem的计算原理：
试想把屏幕平均分成10份，那么每一份就是1/10，我们选择每一份的大小是1rem，那么一个5*(1/10)的大小就占屏幕大小的50%，这就是rem

我们设计稿的宽度是750px，那么对于设计稿上宽度为600px的一个元素，它的rem值是怎么计算呢？
上面我们把屏幕分成了10份，屏幕宽度就是10rem
设计稿上600px元素在设计稿上占的比例是 600/750，将这个比例应用到屏幕上，屏幕的宽度是10rem，那么，这个元素在屏幕上占用的rem就是 600/750*10rem
所以，公式就是 设计稿元素尺寸/设计稿宽度\*以rem为单位的屏幕宽度