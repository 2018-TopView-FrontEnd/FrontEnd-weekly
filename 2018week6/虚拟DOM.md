# 虚拟DOM
## 虚拟DOM（Virtual DOM）
### 1. 什么是真实的DOM？
> * DOM代表了Document Object Model
* DOM是一种抽象化的结构性文本
* DOM的作用只是需要能够直接获取东西
对于Web Developers来说，这种结构性的文本就是`HTML code`，并且 `DOM`被简单的称为`HTML DOM`。`HTML code`中的elements`就变成了`DOM`中的`nodes`。

 HTML是一种文本，那么DOM就是HTML在内存中的表现形式。

 HTML DOM提供了能够过去和修改Node的API，例如：`getElementById`和`removeChild`。
我们通常使用`JavaScript`来操作DOM对象。

### 2. DOM操作中的问题
> * HTML DOM使用的是树结构，通过树结构我们可以轻松遍历树，但是！**容易操作并不代表操作起来很快**

比方说，典型的`jQuery`事件处理程序需要这样子：
1. 查找对事件感兴趣的每一个节点
2. 必要时更新

其中有两个问题：
1. 难管理
想象一下，你必须调整一个事件处理程序。如果你失去了背景，你必须潜水真正深入到代码，甚至知道发生了什么事。
2. 耗时长、错误风险大
它的效率很低。

所以就出现了虚拟DOM!

###3. Virtual DOM
> Virtual DOM技术并不是React发明的，但是React确实是使他发扬光大。

Virtual DOM是HTML DOM的抽象。它是轻量级和脱离浏览器的具体实施细节。因为DOM本身已经是一个抽象的、虚拟的DOM，事实上，Virtual DOM是一个抽象的抽象。

* 实际上，Virtual DOM包含：
    * Javascript DOM模型树（VTree），类似文档节点树（DOM）
    * DOM模型树转节点树方法（VTree -> DOM）
    * 两个DOM模型树的差异算法（diff(VTree, VTree) -> PatchObject）
    * 根据差异操作节点方法（patch(DOMNode, PatchObject) -> DOMNode）
    
* VTree
基本结构模型如下
```
{ 
  // tag的名字 
  tagName: 'p', 
  // 节点包含属性 
  properties: { style: { color: '#fff' } }, 
  // 子节点 
  children: [], 
  // 该节点的唯一表示，后面会讲有啥用
  key: 1
}

```

React是这样创建这种树状结构的

```
React.createElement('h2', {id:myId}, msg)
```


* VTree -> DOM

```
function create(vds, parent) { 
  // 首先看看是不是数组，如果不是数组统一成数组 
  !Array.isArray(vds) && (vds = [vds]); 
  // 如果没有父元素则创建个fragment来当父元素 
  parent = parent || document.createDocumentFragment(); var node; 
  // 遍历所有VNode 
  vds.forEach(function (vd) { 
      // 如果VNode是文字节点 
      if (isText(vd)) { 
        // 创建文字节点 
        node = document.createTextNode(vd.text); 
        // 否则是元素
      } else { 
        // 创建元素 
        node = document.createElement(vd.tag); 
      } 
      // 将元素塞入父容器 
      parent.appendChild(node); 
      // 看看有没有子VNode，有孩子则处理孩子
      VNode vd.children && vd.children.length && create(vd.children, node); 
      // 看看有没有属性，有则处理属性 
      vd.properties && setProps({ style: {} }, vd.properties, node); }); 
      return parent;}

```

上面的看看就好，现在到了最重要的一帕！

* diff(VTree, VTree) -> PatchObject
差异算法是Virtual DOM的核心。

> * Virtual DOM主要思想就是模拟DOM的树状结构，在内存中创建保存映射DOM信息的节点数据，在由于交互等因素需要视图更新时，先通过对节点数据进行diff后得到差异结果后，再一次性对DOM进行批量更新操作。
* 这就好比在内存中创建了一个平行世界，浏览器中DOM树的每一个节点与属性数据都在这个平行世界中存在着另一个版本的虚拟DOM树，所有复杂曲折的更新逻辑都在平行世界中的VirtualDOM处理完成，只将最终的更新结果发送给浏览器中的DOM树执行，这样就避免了冗余琐碎的DOM树操作负担，进而有效提高了性能。

* diff算法主要策略

###### 1. 分层对比

两棵树如果完全比较时间复杂度是O(n^3)，但React的Diff算法的时间复杂度是O(n)。

要实现这么低的时间复杂度，意味着只能平层地比较两棵树的节点，放弃了深度遍历。这样做，似乎牺牲了一定的精确性来换取速度，但考虑到现实中前端页面通常也不会跨层级移动DOM元素，所以这样做是最优的。

![此处输入图片的描述][1]

* 平行的diff层有以下四种情况：
    * REPLACE：节点类型变了。例如下图中的P变成了h3。直接将旧节点卸载（componentWillUnmount）并装载新节点（componentWillMount）就行了。
    * PROPS：节点类型一样，仅仅属性或属性值变了。此时不会触发节点的卸载（componentWillUnmount）和装载（componentWillMount）动作。而是执行节点更新（shouldComponentUpdate到componentDidUpdate的一系列方法）。
    * TEXT：文本变。文本对也是一个Text Node，也比较简单，直接修改文字内容就行了。
    * REORDER：移动，增加，删除子节点。
    
![分层对比][2]

* REORDER拿出来讲一下
![此处输入图片的描述][3]

在中间插入一个节点，程序员写代码很简单：$(B).after(F)。但如何高效地插入呢？简单粗暴的做法是：卸载C，装载F，卸载D，装载C，卸载E，装载D，装载E。

![此处输入图片的描述][4]

涉及到移动，增加，删除子节点的操作时，用上面那种简单粗暴的做法来更新。虽然程序运行不会有错，但效率太低。所以就出现了`key`。

###### 2. 基于key来匹配
React中，数组或枚举类型定义一个key会报错
![报错][5]
如果我们在JSX里为数组或枚举型元素增加上key后，React就能根据key，直接找到具体的位置进行操作，效率比较高。
![此处输入图片的描述][6]

![此处输入图片的描述][7]

###### 3. 基于自定义元素做优化
无论VirtualDOM中的节点数据对应的是一个原生的DOM节点还是vue或者react中的一个组件，不同类型的节点所具有的子树节点之间结构往往差异明显，因此对不同类型的节点的子树进行diff的投入成本与产出比将会很高昂。为了提升diff效率，VirtualDOM只对相同类型的同一个节点进行diff，当新旧节点发生了类型的改变时，则并不进行子树的比较，直接创建新类型的VirtualDOM，替换旧节点。

![此处输入图片的描述][8]

* patch(DOMNode, PatchObject) -> DOMNode
    * 虚拟DOM有了，Diff也有了，现在就可以将Diff应用到真实DOM上了。
    * 由于diff操作已经找出两个VTree不同的地方，只要根据计算出来的结果，我们就可以对DOM的进行差异渲染。

### 3. 虚拟DOM与真实DOM的区别
> * 虚拟DOM比真实DOM轻
* 虚拟DOM比真实DOM快

* 首先来看虚拟DOM的轻在哪里体现？
上面介绍虚拟DOM的时候讲到了它的两个特点：

    * 脱离浏览器的具体实施细节（上面已经介绍了）
    * 轻量级(看test02的比较)
    
    ```
    const vDom2 = <h3 id={myId}>{msg}</h3>
    const test1Div = document.getElementById('test1')
    ```
    
    > 目前，浏览器中存在四种形态的对象：
    1. 超轻量 Object.create(nulll)
    2. 轻量 一般的对象 {}
    3. 重量 带有访问器属性的对象, avalon或vue的VM对象
    4. 超重量 各种节点或window对象

有了虚拟DOM，我们是使用够轻量的对象代替超重对象作为直接操作 主体，减少对超重对象的操作！虚拟DOM的结构是很轻量，最多不超过10个属性，并且其继承层级不超过2层。而DOM节点有70＋个属性，继承层级有6，7层（文本节点6层，元素节点7层）.访问一个属性，可能会追溯几重原型链。

* 虚拟DOM真的比真实DOM快吗？
[小测试来一个][9]

这样看来似乎虚拟DOM并不比真实DOM快。
[网上都说操作真实 DOM 慢，但测试结果却比 React 更快，为什么？][10]

其中前端大佬尤雨溪回答：
React 的基本思维模式是每次有变动就整个重新渲染整个应用。如果没有 Virtual DOM，简单来想就是直接重置 innerHTML。很多人都没有意识到，在一个大型列表所有数据都变了的情况下，重置 innerHTML 其实是一个还算合理的操作... 真正的问题是在 “全部重新渲染” 的思维模式下，即使只有一行数据变了，它也需要重置整个 innerHTML，这时候显然就有大量的浪费。
我们可以比较一下 innerHTML vs. Virtual DOM 的重绘性能消耗：

* innerHTML:  render html string O(template size) + 重新创建所有 DOM 元素 O(DOM size)
* Virtual DOM: render Virtual DOM + diff O(template size) + 必要的 DOM 更新 O(DOM change)

Virtual DOM render + diff 显然比渲染 html 字符串要慢，但是！它依然是纯 js 层面的计算，比起后面的 DOM 操作来说，依然便宜了太多。可以看到，innerHTML 的总计算量不管是 js 计算还是 DOM 操作都是和整个界面的大小相关，但 Virtual DOM 的计算量里面，只有 js 计算和界面大小相关，DOM 操作是和数据的变动量相关的。前面说了，和 DOM 操作比起来，js 计算是极其便宜的。这才是为什么要有 Virtual DOM：它保证了 1）不管你的数据变化多少，每次重绘的性能都可以接受；2) 你依然可以用类似 innerHTML 的思路去写你的应用。



  [1]: https://upload-images.jianshu.io/upload_images/850323-42e4319f8cd51fb9.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/377/format/webp
  [2]: https://upload-images.jianshu.io/upload_images/1959053-fd068c191a95ea82.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp
  [3]: https://upload-images.jianshu.io/upload_images/1959053-b592d77d1cc244e1.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp
  [4]: https://upload-images.jianshu.io/upload_images/1959053-b13f0c68b7cc7c43.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp
  [5]: https://upload-images.jianshu.io/upload_images/1959053-e5ca945bf041e1f4.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp
  [6]: https://upload-images.jianshu.io/upload_images/1959053-17cf74f6fdd45468.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1000/format/webp
  [7]: https://upload-images.jianshu.io/upload_images/850323-934b7cb6bc6e498c.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/452/format/webp
  [8]: https://upload-images.jianshu.io/upload_images/850323-c1211269c31cc981.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/349/format/webp
  [9]: http://chrisharrington.github.io/demos/performance/
  [10]: https://www.zhihu.com/question/31809713