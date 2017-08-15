# 在2013年设计Web应用

从一个controller开始写代码，还是从一个UI片段开始？（CSS布局仍然不能那么快速）

    ```
    function WidgetController ($scope) {
        $scope.tweets = [];//loaded from JSON data
    }
    ```

# 指令的必要性
指令的特性：
* 声明式
* 数据驱动
* conversational
  消息系统：$scope $element $emit/$broadcast

# 解构指令
* 关联模式：EACM
    * restrict
* 配置选项（一个指令就是所有这些选项作为key的JSON对象）
    * priority
    * terminal
    * template 用字符串来描述复杂的DOM模板不太方便，但是引入外部url的话似乎有额外的IO开销？
    * templateUrl
    * replace
    * compile: function (element, attributes, transclude) {},
        * 想要同时使用自定义compile和link特性：compile返回一个{pre: CompileFn, post: LinkFn}
    * link: function ($scope, $element, $attrs) {},
    * scope
        * 一个DOM元素只能有一个scope => 如果多个指令都设置了scope:true，则它们共享同一个scope
        * 只有一个指令能够请求isolate scope（？）
    * controller
        * 参考下文的ngModelDirective
    * require
        * 2个合起来用。共享指定指令的controller实例给当前指令（通过link的参数注入）
            * 当前指令和require的指令都是作用在同一个DOM元素上？
            *  ^ngModel 意味着向上遍历查找
        * $scope.$apply( ... ): 推迟DOM更新到下一次digest loop？
    * transclude

# Compile vs Link
## ng-repeat
以ng-repeat为例，其link做的事情是：调用$scope.$watch，每次model变化时，重新设置整个$element子树（代码示例里function嵌套太多了，很不适应...）

  ```
  compile: function (element, attrs, linker) {
    return function ($scope, $element, $attrs) {
        $scope.$watch(function ($internalScope) {
            $element.html('');
            var values = … 
            values.forEach(function (data, index) {
                $internalScope.element = data; 
                linker($internalScope, function (clone) {
                    $element.append(clone);
                });
            });
        });
    };
  }
  ```

## ng-switch
ng-switch-when：

* 通过transclude: 'element',请求linker的一个实例，
* 通过require: '^ngSwitch'注入父ng-switch指令的controller
* link逻辑：ctrl.cases['!' + attrs.ngSwitchWhen] = linker; 往上收集linker实例，也就是对应的DOM子树

26个不同的AngularUI指令中，只有3个用到了compile（自定义linking）

# Keeping it Clean with Scope
## scope = false
跟父指令共享同一个scope
## scope = true
对父scope可以读但不能修改，子scope从父scope原型继承（如果是这样的话，那为什么不能修改？）
## scope = {} / isolate scope（细粒度控制）
key是父scope中的变量名字，而value对应子scope中的变量。value之前指定额外的前缀：

* @ 只读，父scope的修改会被持续监控反映到子scope，直到子scope修改（写）其值为止
* = 2-way binding（这导致指令之间强耦合了吗）
* & 内部（方法绑定？推荐改用控制器）
    * wrapper函数？

这个地方让我想起了C++ 11的lambda的参数捕获语法。这里的语法不是特别的直觉，没有Angular 2里面的输入/输出概念更容易理解。

口号：结构化！模块化！

# 控制器
FormController：啥也不干，只是关联/注册了4个函数：$addControl $removeControl $setValidaity $setDirty

ngModelController（对应ng-model指令？）
  ```
  var parentForm = $element.inheritedData('$formController') || nullFormCtrl;
  ```
  ...
  ngModelDirective:
 
* 通过 require: ['ngModel', '^form'], 引入2个controller，注入为link的参数
* controller: NgModelController,
* element.bind('$destroy', function() { formCtrl.$removeControl(modelCtrl); });
    * 何时需要监听$destroy事件？

```
<input type="text" ng-model="timeOfDay" time-picker />
```

timePicker指令:

* ... require : '?ngModel', ...
* 这里的代码示例似乎引入一个问题：如何引入jQuery UI插件，与AngularJS指令共存？
* p51 这说了半天，setTimeout不带超时参数到底是什么目的？jQuery与AngularJS之间有竞争？
    * 作者的意思是不是在link函数里推迟控件的初始化？
* ngModel.$render = function (val) { //将value从angular传递到jQuery插件？

# Transclusion
For Angular that answer is transclusion. My unofficial interpretation of the word is translated-inclusion. What transclusion does is offer a way to create a widget with an isolate scope, which we as good modular developers always do, but then tunnel back out into the parent scope to parse the original content. 

注意这里的movieInfo指令：

```
<div movie-info="movie">
     <p>Hi, I'm {{name}}, and I'm going to see {{movie}} with
   {{friendCount}} friends</p>
</div>

template: "...{{name}} ... <div ng-transclude></div> ...
```

name: 名字冲突？

## `transclude`指的是template用占位元素包含指令实际装饰的DOM子树？

* template内的binding在`指令scope`
* 而move-info指令属性标记的binding对应其`parent scope`

感觉可以把前者称为静态模板，后者称为动态模板...

## 操作转换嵌入的内容

```
controller : function ($scope, $element, $attrs, $transclude) {
    $transclude(function (clone) {
        $element.append(clone.find(".title")); ...
    })
}
```

2种用法：
  always remember that transclusion is your friend when you need to interact with the content internal to your directive. Use the standard `ng-transclude` directive when you want the content unaltered, and `controller plus $transclude` if you need to manipulate it first.

# 端到端测试
## Karma
```
describe('My Tested Directive', function () {
     ... //setup code
     var directiveTpl = '<div player-widget="playerList"></div>';
     it('should create player widget element', function () {
       var $scope = $rootScope.$new();
       var $element = $compile(directiveTpl)($scope);
       expect($element.html()).toContain('class="player-widget"');
}); });
```

## ng-scenario

  ```
  browser().navigateTo('../../app/index.html');
  ```

vs Jasmine: `expect` here requires a future, not a value.

# 单元测试
显式地把异常情况在data model上建模？

```
{{p.name}} <span class="team" ng-show="p.team">({{p.team}})</span>
```

# Put it All Together
## DOM操纵插件注意事项

* 识别`初始化`和`更新`
* 评估：显式告知变化，or 自行检测
* 配置watchers

为此，实现controller和link：前者作为参数被注入到后者？

```
link : function ($scope, $element, $attrs, ctrl) {
    ...
    $scope.$watch('elements', function (newValue, oldValue) {
        if (newValue)
            initOrUpdate();
    }
}
```

## Jasmime异步测试？
I've introduced a few new Jasmine testing methods here and if you're not accustomed to asynchronous testing within Jasmine, you likely haven't seen `runs` and `waitsFor` blocks before.

...

It turns out that even though Masonry was accurately positioning our elements, it was operating before all the element content had been compiled, and thus it miscalculated the proper height to apply to our grid（最好用setTimeout将DOM操纵代码包装起来）

作者个人网站：www.mrvdot.com （Still there？）
