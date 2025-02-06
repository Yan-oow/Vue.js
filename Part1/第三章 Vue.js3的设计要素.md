@[TOC](《Vue.js设计与实现》学习笔记_第三章 Vue.js3的设计要素)
### 1. 声明式的描述UI
前端页面涉及内容：

> DOM 元素：例如是 dtv 标签还是a标签。
属性：如a 标签的 href 属性，再如id、class 等通用属性。
事件：如click、keydown 等。
元素的层级结构：DOM树的层级结构，既有子节点，又有父节点。

Vue.js3的解决方案：

> 使用与 HTML 标签一致的方式来描述DOM元素，例如描述一个dtv 标签时可以使用 < div >< /div >;
使用与HTML标签一致的方式来描述属性，例如 < div id="app" >< /div >；
使用：或v-bind 来描述动态绑定的属性，例如 < div :id="dynamicId" >< /div >；
使用@或v-on 来描述事件，例如点击事件 < dtv @click="handler" >< /div >；
使用与 HTMIL 标签一致的方式来描述层级结构，例如一个具有 span 子节点的 dtv 标签
‹ div > < span >< /span>< /div>.

Vue.js 是一个声明式的框架。声明式的好处在于，它直接描述结果，用户不需要关注过程。Vue.js 采用模板的方式来描述UI，但它同样支持使用虚拟 DOM 来描述UI。虚拟 DOM 要比模板更加灵活，但模板要比虚拟DOM更加直观。

> tag用来描述标签名称，所以 tag：'div'描述的就是一个 < div >标签。
props 是一个对象，用来描述< div- 标签的属性、事件等内容。可以看到，我们希望给div绑定一个点击事件。
children 用来描述标签的子节点。在上面的代码中，children 是一个字符串值，意思是div 标签有一个文本子节点：< div>click me< /div>
```javascript
	//JavaScript
	const title = {
	    // 标签名称
	    tag：'div'，/ 标签属性
	    props: {
	        onClick: ()=>alert('hello')
	    },
	    // 子节点
	    children: 'click me'
	//对应到 Vuejs 模版
	ch1 @click="handler"><span></span></h1>
```
### 2. 初识渲染器
渲染器的作用是，把虚拟 DOM 对象渲染为真实DOM元素，如图1所示。它的工作原理是，递归地遍历虚拟 DOM 对象，并调用原生DOM API 来完成真实DOM的创建。這染器的精髓在于后续的更新，它会通过Diff算法找出变更点，并且只会更新需要更新的内容。
![在这里插入图片描述](https://i-blog.csdnimg.cn/direct/095b011d189c4509abd88525eefb7c13.jpeg#pic_center =x113)
<center>图1 渲染器的作用</center>

> 创建元素：把 vnode.tag 作力标签名称来创建 DOM元素。
为元素添加属性和事件：遍历vnode.props 对象，如果key 以on字符开头，说明它是一个事件，把字符 on 截取掉后再调用 toLowerCase函数将事件名称小写化，最终得到合法的事件名称，例如 onclick 会变成 click，最后调用 addEventListener 绑定事件处理函数。
处理 children：如果 children 是一个数组，就递归地调用 renderer 继续渲染，注意，此时我们要把刚刚创建的元素作为挂载点（父节点）；如果 children 是字符串，则使用createTextNode 函数创建一个文本节点，并将其添加到新创建的元素内。
### 3. 组件的本质
组件是一组虚拟DOM元素的封装，它可以是一个返回虚拟DOM的函数，也可以是一个对象，但这个对象下必须要有一个函数用来产出组件要渲染的虚拟 DOM。谊染器在渲染组件时，会先获取组件要渲染的内容，即执行组件的渲染函数并得到其返回值，我们称之次 subtree，最后再递归地调用渲染器将 subtree 渲染出来即可。

```javascript
	function renderer (vnode, container) {
	    if (typeof vnode. tag === 'string') { 
	        // 说明 vnode 描述的是标签元素
	        mountElement(vnode, container)
	    } else if (typeof vnode.tag === 'function') { 
	        // 说明 vnode 描述的是组件
	        mountComponent (vnode, container)
	    }
	}
```

> 这里的 renderer 函数接收如下两个参数。
vnode：虚拟DOM对象。
container：一个真实DOM元素，作为挂载点，谊染器会把虚拟 DOM渲染到该挂载点下。
如果 vnode.tag 的类型是字符串，说明它描述的是普通标签元素，此时调用 mountElenent 函数完成谊染；如果 vnode.tag 的类型是函数，则说明它描述的是组件，此时调用 mountComponent函数完成渲染。其中 mountElement 函数与上文中 renderer 函数的内容一致：

```javascript
	function mountElement(vnode, container) {
	    // 使用 vnode.tag 作为标签名称创建 DOM 元素
	    const el = document.createElement(vnode. tag)
	    // 遍历 vnode.Props，将属性、事件添加到DOM 元素
	    for (const key in vnode.props) {
	        if (/^on/.test(key))｛
	            // 如果key 以字符串on 开头，说明它是事件
	            el.addEventListener (
	                key.substr(2).tolowerCase（），//事件名称 onclick-..>click
	                vnode.props［key］ // 事件处理函数
	            )
	        }
	    }
	
	    // 处理 children
	    if (typeof vnode.children === 'string')｛
	        // 如果 children 是字符串，说明它是元素的文本子节点
	        el. appendChild(document.createTextNode(vnode.children))
	    } else if (Array. isArray(vnode.children)){
	        // 递归地调用 renderer函数渲染子节点，使用当前元素 el作为挂载点
	        vnode.children.forEach(child => renderer(chiLd,eL))
	    }
	    // 将元素添加到挂载点下
	    container.appendChild(el)
	}
```
### 4. 模版的工作原理
Vue.js 的模板会被一个叫作编译器的程序编译为渲染函数。

< template>标签里的内容就是模板内容，编译器会把模板内容编译成渲染函数并添加到< script>标签块的组件对象上。所以，无论是使用模板还是直接手写渲染函数，对于一个组件来说，它要渲染的内容最终都是通过渲染函数产生的，然后渲染器再把渲染函数返回的虚拟 DOM渲染为真实 DOM，这就是模板的工作原理，也是 Vuejs 渲染页面的流程。
### 5. Vue.js是由各个模块组成的有机整体
如前所述，组件的实现依赖于渲染器，模板的编译依赖于编译器，并且编译后生成的代码是根据渲染器和虚拟 DOM 的设计决定的。编译器、渲染器都是Vue.js 的核心组成部分，它们共同构成一个有机的整体，不同模块之间互相配合，进一步提升框架性能。
