# SVG Essentials, 2nd

## 开始
## 页面中使用svg
## 坐标系
### Preserving Aspect Ratio
## 基本形状
## 文档结构
## 变换坐标系统
## 路径
* Quadratic Bézier Curves：开始点、结束点、控制点
* Cubic Bézier Curves：在一个segment里同时具有peak和valley，2个控制点

## 模式与梯度
* gradientTransform 与 patternTransform（变换的是fill单元，而不是被fill的对象）

## 文本
* `<tspan>`

## 裁剪与蒙版
* `<clipPath>`
* An SVG mask, on the other hand, transfers its transparency to the object it masks. 

## Filters
## SVG动画
* SMIL3

```
<rect x="10" y="10" width="200" height="20" stroke="black" fill="none">
    <animate
        attributeName="width"
        attributeType="XML"
        from="200" to="20"
        begin="0s" dur="5s"
        fill="freeze" />
</rect>
```

* 时间：有了小时部分就是绝对时间？仅有分钟和秒代表相对时间？
* 引用另一个`<animate>`元素的属性:

```
<circle cx="120" cy="60" r="10" style="fill: #9f9; stroke: gray;">
    <animate id="c2" attributeName="r" attributeType="XML"
        begin="c1.begin+1.25s" dur="4s" from="10" to="30" fill="freeze"/>
</circle>
```

* 重复动画
    * fill="remove" 默认
    * repeatCount/repeatDur属性
        * 值indefinite

* keyTimes属性：分割各个transition所占的duration
* calcMode属性：
    * paced
    * linear
    * discrete
    * spline
* 非连续可变的属性：`<set>`元素

```
<text text-anchor="middle" x="60" y="60" style="visibility: hidden;">
    <set attributeName="visibility" attributeType="CSS"
        to="visible" begin="4.5s" dur="1s" fill="freeze"/>
    All gone!
    </text>
```

* `<animateTransform>`
    * The `<animate>` element doesn’t work with rotate, translate, scale, or skew 
      because they’re all “wrapped up” inside the `transform` attribute.（animate元素不支持变换属性）
    * 例：

    ```
    <g transform="translate(100,60)">
        <rect x="-10" y="-10" width="20" height="20" style="fill: #ff9; stroke: black;">
            <animateTransform attributeType="XML"
            attributeName="transform" type="scale"
            from="1" to="4 2"
            begin="0s" dur="4s" fill="freeze"/>
        </rect>
    </g>
    ```
    * 如果需要组合多个变换，使用`additive`属性（replace/sum）
        * 靠，这规范太繁琐了
* `<animateMotion>`：使对象沿任意路径运动（移动的是坐标系？）
```
<g>
    <rect x="0" y="0" width="30" height="30" style="fill: #ccc;"/>
    <circle cx="30" cy="30" r="15" style="fill: #cfc; stroke: green;"/>
    <animateMotion from="0,0" to="60,30" dur="4s" fill="freeze"/>
</g>
```
```
<path d="M-10,-3 L10,-3 L0,-25z" style="fill: yellow; stroke: red;">
    <animateMotion
    path="M50,125 C 100,25 150,225, 200, 125"
    dur="6s" fill="freeze"/>
</path>
```
注意这里animateMotion元素使用使用2套类型机制。
    * rotate:auto 使对象x轴始终与路径的切线平行
* Specifying Key Points and Times for Motion
    * keyPoints、keyTimes
* Animating SVG with CSS（略）
## 添加交互
* animation-play-state: paused/running; ？？？
## 使用SVG DOM
## 生成SVG
