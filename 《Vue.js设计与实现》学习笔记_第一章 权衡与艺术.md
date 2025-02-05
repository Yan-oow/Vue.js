@[TOC](《Vue.js设计与实现》学习笔记_第一章 权衡与艺术)
### 1.命令式&声明式的差异
命令式更加关注过程，声明式更加关注结果。命令式在理论上可以做到极致优化，但是用户要承受巨大的心智负担；而声明式能够有效减轻用户的心智负担，但是性能上有一定的牺牲，框架设计者要想办法尽量使性能损耗最小化。

命令式框架：

```java
	const div = document.querySelector（'#app'）// 荻取div
	div.innerText = 'hello world' // 设置文本内容
	div.addEventListener（'cltck'，（）=>｛ alert（'ok'）］）//绑定点击事件
```
声明式框架：

```java
	<div @click="() => alert('ok')">hello world</div>
```

### 2.虚拟 DOM的性能
声明式的更新性能消耗=找出差异的性能消耗＋ 直接修改的性能消耗。虚拟 DOM的意义就在于使找出差异的性能消耗最小化。用原生 JavaScript 操作 DOM的方法（如 document.createElement）、虚拟 DOM和innerHTML 三者操作页面的性能，不可以简单地下定论，这与页面大小、变更部分的大小都有关系，除此之外，与创建页面还是更新页面也有关系，选择哪种更新策略，需要我们结合心智负担、可维护性等因素综合考虑。

