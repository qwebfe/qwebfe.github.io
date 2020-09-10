---
layout: post
title: 记Mousetarck的一些技术尝试
date: 2019-08-15
categories: article
theme: jekyll-theme-leap-day
---

![mousetrack_img](https://p5.ssl.qhimg.com/t01771478e667255d55.png)

### 需求背景

为了统计分析用户页面行为，我们经常需要收集鼠标在页面上的各类操作，如页面中按钮和链接的点击和统计较准确的访客页面停留时间。而本文主要记录在解决`分别记录鼠标在页面各个区域的停留时间总和`这一问题的技术方案的尝试。

简化问题描述：将浏览器的可视窗口平均分为2 × 2的4份， 如图。分别记录鼠标在各个区域的停留时间总和，当然，不能影响页面的正常功能的使用。

![页面区域示意](https://p5.ssl.qhimg.com/t01752acf18ecbe6aa7.png)

当鼠标进入某块区域时开始计时， 移出时结束计时。那问题就在于如何判断鼠标的所在区域。

于是， 自然就想到了通过鼠标的坐标来判断所处区域 。



### Solution 1

监听鼠标的mousemove事件， 获取鼠标的坐标(x, y),  根据它来判断鼠标当前处在哪个区域，然后累加当前区域的记录时间。

此外， 需要对该事件的触发频率进行优化，避免触发频率过快，造成不必要的计算开销。

```javascript
// 主要代码
const mouseTrack = () => {
  fromEvent(document, 'mousemove')
    .pipe(debounceTime(sampleTime))
    .subscribe((e: MouseEvent) => {  
      	const { clientX, clientY } = e
        let currentPart : number = 	// 计算所在区域
          Math.floor(clientX / unitWidth) + 
          n * Math.floor(clientY / unitHeight);
 
    	const now = new Date().getTime();
    	try {
        	  record[lastPart].duration += now - tick;
    	} catch(err) {}
      
        tick = now;
        lastPart = currentPart;
        console.log(record.map(v => v.duration));
     })
}
````

问题似乎就这样愉快的被解决了！

但是经过一番测试之后， 发现了一些不足的地方。

+ 页面加载后， 若鼠标一直不动， 则无法触发事件，进而无法判断所在区域（疑无解）
+ 当用户以较快的速度移动，有时导致统计的时间和所在区域对应错误（统计的准确性有待提高）
+ 实际需要统计的区域肯定不是简单均分页面的4块区域， 实际判断鼠标所在区域的计算会更复杂

对于mousemove这类触发频繁的事件， 在其他如drag， window resize， scroll的场景下，可以使用函数节流、防抖等操作优化执行频率，且无明显副作用。但在此需求下，如果时间间隔过大，统计的准确性明显下降，如果时间间隔太小甚至不对触发频率进行限制， 而增加的性能开销也不是我们想要的。

难道，就秉持中庸之道， 取个中间值？

后期也了解到通过分析搜集海量的这种统计数据， 清洗极端数据， 一通分析之后，也可以得到较准确的结果。单纯的收集鼠标在某个区域的时间意义不大， 主要还是通过海量的数据搜集分析， 进而了解用户在页面上行为、喜好、热点区域等。

道理是都懂，但是作为一个优秀的程序员， 总是想在自己的一亩三分地写出更好的代码，能不能提供比较准确的数据，又不会产生较大的性能开销？



### Solution 2

上述方式需要频繁的触发事件， 主要是为了获取鼠标当前所在区域， 那我就在统计区域上加个透明div, 当div的`mouseenter`事件触发时，开始计时， 触发`mouseleave`时， 结束计时。加上后，发现由于div的遮盖， 页面中的各类按钮，链接都无法触发。这肯定不行， 又查了一下事件穿透， get了[pointer-events](https://www.w3.org/TR/SVG/interact.html#PointerEventsProperty)属性.

`pointer-events`可以禁用鼠标事件， 允许事件穿透，常用的属性值(auto / none), 其他属性值只适用于svg元素

愉快的加上该属性， 发现透明div的mouseenter/mouseleave事件也被禁止，无法触发了......

稳住心态，思索了一下

![pointer-evnets](https://p3.ssl.qhimg.com/t012d3f90a084507415.png)

```
鼠标首次移动：
	记录开始时间
	当前所在区域 pointer-events: none;
某一区域触发mouseenter事件：
	该区域变成 pointer-events: none;
	原来区域 pointer-events: all;
	记录原区域的停留时间
	记录进入新区域的时间
```



鼠标当前所在的透明的div区域处于事件穿透状态， 其他区域被div正常覆盖，当鼠标移入其他区域时， 移入区域变成事件穿透状态(页面的链接、按钮等功能正常)， 原来区域还原。



通过这种方式，可以较好的解决上面的问题

+ 不用mousemove这种频繁触发的事件，节省了额外的开销

+ 只要鼠标的移动速度在浏览器的捕获范围内， 它在不同区域之间的切换都能很好的触发事件， 大大提供了统计数据的准确性

不足：

+ pointer-events的兼容性， 兼容到 IE11 (硬伤)

还有一点瑕疵，就是鼠标在不同区域移动时， 需要操作div的pointer-events属性在all和none之间切换， 且在非鼠标所在区域，透明div是遮挡页面鼠标的各种事件的。能不能不让它遮挡呢？



### Solution 3

其实，是看中了pointer-events的`stroke`属性值， 利用它， 可以使得在元素**内部**事件穿透， 在元素**边框**触发事件，极好的满足了我的诉求,  由于该属性只在SVG下生效，于是将透明div换成了SVG元素。

![pointer-events-border](https://p0.ssl.qhimg.com/t01d62897a2fba37bbe.png)

实际测试一番后发现，要以较慢的速度移动鼠标，才会触发SVG边框的事件，稍快移动鼠标， 边框就无法捕捉事件。由于边框的宽度有限， 移动太快， 就超出了浏览器的捕捉极限。之后又试着增大了边框的宽度， 但效果变化不是很明显。

感觉有点奇怪， 于是新建一个测试页面，监听一个边框为1px的div，在我的手速极限内， `mouseenter`事件正常触发。但是在svg里面为什么触发比较艰难呢？

是浏览器的极限就是这样，还是我写的SVG有问题？一直没有解决这个问题，欢迎大家指教 。

```javascript
const drawsvg = function() {
    const svgns = "http://www.w3.org/2000/svg"; 
    let svg = document.createElementNS(svgns, "svg"); 
    svg.setAttribute(); // other attribute
    svg.setAttribute("style", "pointer-events: none;");

    let polygon = document.createElementNS(svgns, "polygon");
    polygon.setAttribute(); // other attribute
    polygon.setAttribute("style", "pointer-events: stroke;");
    
    svg.appendChild(polygon);
    document.body.appendChild(svg);
}
```



### 后记

本篇文章主要记录在解决**记录鼠标在页面各个区域的停留时间总和**这一需求时的实现方式的尝试和解决相应问题的进一步尝试。现阶段，选择的是方案二和方案一组合使用，在满足pointer-events属性的兼容性情况下，使用方案二， 退之， 则使用第一种方案。

*如文章有任何bug，欢迎指出*！
