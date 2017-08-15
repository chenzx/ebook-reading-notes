# AngularJS UI Development-Packt Publishing 2014
[toc]

## 配置环境
## AngularUI：Intro & Utils
* ui-mask指令
* ng-style
* jQuery Passthrough
    * 没有详细解释背后的实现机制！
## AngularUI：Extended
### Embedding Google Maps
* # bower install angular-ui-map
* 等Google maps加载完成后再初始化angular：
    ```
    function onGoogleReady() {
        angular.bootstrap(document.getElementById("myApp"), ['myApp']);
    }
    ```
* 添加marker
    ```
    $scope.addMarker = function($event, $params) {
        $scope.myMarkers.push(new google.maps.Marker({
            map: $scope.myMap,
            position: $params[0].latLng
        }));
    };
    $scope.eventBinding = {'map-click': 'addMarker($event, $params)'};
    ```
### 用bower管理应用依赖（略）
### calendar组件
* 日期格式化: http://momentjs.com
## ng-grid
* https://angular-ui.github.io/ng-grid 网址已失效
* 这个例子里作者展示了控件怎么用（声明式地），但是并没有讲解ng-grid怎么实现的？？
## Learning Animation
* 1.2+  $animate服务
    * no magic
    ```
    For CSS-based animations, the $animate service object
    parses the transitions/animations associated with an element defined in those CSS
    classes. The service then extracts the animation details, such as transition-property,
    transition-duration, transition-delay, and so on, that delay DOM updates till the
    animation is completed. So, this is how animation works and it's no magic.
    ```
    * Staggering animations
        * .ng-EVENT-stagger
        ```
        The  ngAnimate service looks for the ng-EVENT-stagger CSS class associated with the element and extracts
        the animation/transition details to perform the staggering effect.（怎么根据class提取CSS细节？）
        ```
* angular-animate: 动态添加class
    * .ng-enter ng-enter.ng-enter-active .ng-leave .ng-leave.ng-leave-active
        * *-active代表CSS动画结束后的最终状态？
    * 但是怎么定义对应的CSS恐怕还是有点难度...
* easing函数
    * bezier-curve() in CSS？
* Using LESS（允许CSS变量）
    * `<link rel="stylesheet/less" type="text/css" href="css/main.less">` 这样也行？
    * 这并没有解决本质问题：easing曲线如何可视化编辑？？
* Using animate.css（略）
* JavaScript-defined animations
    * enter/leave
    * done回调
## Using Charts and Data-driven Graphics
* [NVD3](http://nvd3.org/) is a library of reusable chart components for d3.js. 
## CSS Frameworks
* The evolution of responsive design
    * https://www.smashingmagazine.com/2009/06/fixed-vs-fluid-vs-elastic-layout-whats-the-right-one-for-you/
    * Mobile First, Luke Wroblewski, Ingram Publishing
* @media 媒体查询
* Twitter bootstrap
* foudation
    * http://blog.teamtreehouse.com/use-bootstrap-or-foundation
## AngularUI Bootstrap
## 定制Bootstrap
* tabset模板：多个transclude？但另外一个是特殊的tab-content-transclude
```
<div>
    <ul class="navnav-{{type || 'tabs'}}" ng-class="{'nav-stacked':vertical, 'nav-justified': justified}" ng-transclude></ul>
    <div class="tab-content">
        <div class="tab-pane" ng-repeat="tab in tabs" ng-class="{active: tab.active}" tab-content-transclude="tab">
        </div>
    </div>
</div>
```
    * 似乎没怎么讲清楚~
## Mobile Development Using AngularJS and Bootstrap
* data-target="#addBookmark"
* angular-route：略
* angular-touch
    * `<div ... ng-swipe-left="paginate('forward')" ng-swipe-right="paginate('backward')">`
* Improving initial page load
