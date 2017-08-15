# Creating Web Animations

# Basics
## CSS Animations
## CSS Transitions
## CSS Timing Functions

# 应用
## Animating Your Links to Life
* 不是所有的CSS属性都可以设置动画
    * text-decoration --> border-bottom
## Simple Text Fade and Scale Animation
* 定义关键帧
```
@keyframes fadeAndScale {
from {
    opacity: 0;
        transform: scale(.9, .9);
    }
    to {
        opacity: 1;
        transform: scale(1, 1);
    }
}
```
* 定义动画
```
h1 {
    animation-duration: .3s;
    animation-name: fadeAndScale;
    animation-timing-function: cubic-bezier(.71,.55,.62,1.57);
}
```
* 定义转换
```
a {
    background-color: #A6D2FF;
    transition: background-color .2s ease-out;
}
a:hover {
    background-color: #EEE;
}
```
* cubic-bezier曲线在线可视化编辑: http://cubic-bezier.com/#.71,.55,.62,1.57

## Creating a Smooth Sliding Menu
* 菜单从左边slide in的效果：
```
#theMenu {
    position: fixed;
    left: 0;
    top: 0;
    transform: translate3d(-100vw, 0, 0);
    transition: transform .3s cubic-bezier(0, .52, 0, 1); /* transition定义在初始样式上？*/
    width: 100vw;
    height: 100vh;
}
#theMenu.visible {
    transform: translate3d(0vw, 0, 0);
}
```
* 疑问：在transition动画进行中的时候，应该是无法响应用户事件输入吧？
## Scroll-Activated Animations
* transition同时作用到transform和opacity，因此是`all`：
```
#myList li {
    padding-left: 7px;
    margin-bottom: 15px;
    transition: all .2s ease-in-out;
    transform: translate3d(0px, 30px, 0);
    opacity: 0;
}
#myList li.active {
    transform: translate3d(0px, 0, 0);
    opacity: 1;
}
```
* 检测元素是否可见
```
function isPartiallyVisible(el) {
    var elementBoundary = el.getBoundingClientRect();

    var top = elementBoundary.top;
    var bottom = elementBoundary.bottom;
    var height = elementBoundary.height;

    return ((top + height >= 0) &&
        (height + window.innerHeight >= bottom));
    //如果是“完全可见”，则：
    //return ((top >= 0) && (bottom <= window.innerHeight));
}
```
* W3C新技术
    * Passive event listeners
    * IntersectionObserver（检测元素是否可见）
## The iOS Icon Wobble/Jiggle
* 使用2组不同的rotate（及transform-origin）：
```
@keyframes keyframes1 {
    0% {
        transform: rotate(-1deg);
        animation-timing-function: ease-in;
    }
    50% {
        transform: rotate(1.5deg);
        animation-timing-function: ease-out;
    }
}
@keyframes keyframes2 {
    0% {
        transform: rotate(1deg);
        animation-timing-function: ease-in;
    }
    50% {
        transform: rotate(-1.5deg);
        animation-timing-function: ease-out;
    }
}
#main .icon:nth-child(2n) {
    animation-name: keyframes1;
    animation-iteration-count: infinite;
    transform-origin: 50% 10%;
}
#main .icon:nth-child(2n-1) {
    animation-name: keyframes2;
    animation-iteration-count: infinite;
    animation-direction: alternate;
    transform-origin: 30% 5%;
}
```

同时对每个icon应用不同的delay和duration：

```
<img class="icon" style="animation-delay: -.2s; animation-duration: .22s" .../>
```
* 缺点：CSS中没有随机性控制（引入一个random()函数？）
## Parallax Scrolling
* 对position:fixed;及z-index:-3;的背景元素进行translate-3d变换？
```
#greenPentagon {
    background-image: url("http://bit.ly/greenPentagon");
    background-repeat: no-repeat;
    background-position: 5% top;
    background-size: 50%;
    position: fixed;
    top: 0;
    width: 100vw;
    height: 100vh;
    z-index: -3;
    opacity: .75;
}
```
* 获取scroll位置
```
window.addEventListener("DOMContentLoaded", scrollLoop, false);
var xScrollPosition;
var yScrollPosition;
function scrollLoop() {
    xScrollPosition = window.scrollX;
    yScrollPosition = window.scrollY;
    .... 这里还少一些更新CSS样式的代码（我说怎么回事呢）
    requestAnimationFrame(scrollLoop); //把状态监测函数作为callback放到rAF里执行？
}
```
It ensures we call our  scrollLoop function every time our screen is ready to update—no slower, no faster. （？）
## Sprite Sheet Animations Using Only CSS
* keyframe定义的是初始状态？
```
@keyframes sprite {
    100% {
        background-position: -7224px;
    }
}
```
* 关联animation：
```
#spriteContainer {
    width: 300px;
    height: 300px;
    display: block;
    background-image: url("images/sprites_final.png");
    animation: sprite .3s ease-in infinite;
}
```
* 定制的easing曲线实现jump：steps
```
    animation: sprite .3s steps(24) infinite;
```
备注：感觉这比gif动画有意思一点？steps jump动画每2个相邻step之间是不是无法平滑过渡的？如果是那样的话，我觉得不如用SVG动画更好？
## Creating a Sweet Content Slider
* 每个content使用float，而直接wrapper容器上定义transform
* ！wrapper容器外部再套一层contentContainer，用于clip
```
#contentContainer {
    width: 550px;
    height: 350px;
    border: 5px solid black;
    overflow: hidden;
}
```
* navLinks
```
<div id="navLinks">
    <ul>
        <li class="itemLinks" data-pos="0px"></li>
        <li class="itemLinks" data-pos="-550px"></li>
        <li class="itemLinks" data-pos="-1100px"></li>
        <li class="itemLinks" data-pos="-1650px"></li>
    </ul>
</div>
```
感觉为了实现一个动画，需要指定这些中间值，是比较繁琐的地方。。。

JS响应link元素的点击：

```
// Handle changing the slider position as well as ensure
// the correct link is highlighted as being active
function changePosition(link) {
    var position = link.getAttribute("data-pos");
    var translateValue = "translate3d(" + position + ", 0px, 0)";
    wrapper.style.transform = translateValue;
    link.classList.add("active");
}
```

* 最后，再wrapper上添加transition：
```
#wrapper {
    width: 2200px;
    transform: translate3d(0, 0, 0);
    transition: transform .5s ease-in-out;
}
```

读完这本书后半部分的example，感觉CSS动画（Web动画）也不是那么难。不过，问题是：假如复杂动画需要组合多个transform以及不同的transition衔接控制呢？更复杂的3d变换？？
