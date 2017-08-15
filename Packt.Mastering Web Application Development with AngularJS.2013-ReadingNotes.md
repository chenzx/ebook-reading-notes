# Angular Zen
* Batarang

  Batarang is a Chrome developer tool extension for inspecting the AngularJS web applications. Batarang is very handy for visualizing and examining the runtime characteristics of AngularJS applications. We are going to use it extensively in this book to peek under the hood of a running application. Batarang can be installed from the Chrome's Web Store (AngularJS Batarang) as any other Chrome extension.

* Sublime 2插件：
https://github.com/angular-ui/AngularJS-sublime-package

* ng-app ng-init

* 2-way binding：`<input type="text" ng-model="name">`

## 控制器
```
<div ng-controller="HelloCtrl">
...
var HelloCtrl = function ($scope) {
   ... //初始化scope对象
}
```

## scope

```
<li ng-repeat="country in countries">
```
注意这里ng-repeat指令为每次迭代的country变量创建一个单独的scope！（仔细体会这里的精髓～）

scope的读访问类似于JS的prototype，相当直觉（但是前提是没有名字shadowing）。写访问就不是了：

```
<input type="text" ng-model="$parent.name">
```
避免使用$parent，这使得表达式依赖于DOM树结构。

scope上的事件传播：

* $emit 向上
* $broadcast

```
$scope.$on('$locationChangeSuccess', function(event, newUrl, oldUrl){
    ...
})
```
3个$emit事件：

1. $includeContentRequested
2. $includeContentLoaded
3. $viewContentLoaded

7个$broadcast事件：

1. $locationChangeStart
2. $locationChangeSuccess
3. $routeUpdate
4. $routeChangeStart
5. $routeChangeSuccess
6. $routeChangeError
7. $destroy

## DI

注册服务

* $provide
* $injector

DI管理的对象类型

* value
* myModule.service('notificationsService', NotificationsService);
* factory
    * 对象可以引用内部私有状态
* constant 常量
* 最一般的：provider
    * $get
    * 对象可以拥有额外的属性/方法

模块生命周期

* 配置阶段

```
myMod.config(function(notificationsServiceProvider){ //<-- DI风格;
    notificationsServiceProvider.setMaxLen(5);
});
```

* 运行阶段

```
angular.module('upTimeApp', []).run(function($rootScope) {
    $rootScope.appStarted = new Date();
});
```

* 模块依赖于其他模块
```
    angular.module('application', ['notifications', 'archive'])
```
A service defined in one of the application's modules is visible to all the other modules. （平面的名字空间）

** redefine to override（更靠近根的优先）

## Testacular(spectacular test runner)
## A sneak peek into the future
* Object.observe
  就为了把Angular的性能提升20%？幸好后来被废弃了（内核开发者发现此API容易被滥用）

# 构建和测试
* Sample应用：https://github.com/angular-app/angular-app
* 项目模板：https://github.com/angular/angular-seed
* Anatomy of a Jasmine test
    * describe
    * beforeEach
        ```
        beforeEach(module('archive'));
        beforeEach(*inject*(function (_notificationsArchive_) { //inject返回了一个callback函数？
            notificationsArchive = _notificationsArchive_;
        }));
        ```
    * it
    * expect
* Mock objects and asynchronous code testing
    * $timeout.flush()
* 端到端测试
    * Karma runner tips and tricks（有点意思）
        * xdescribe xit
        * ddescribe
        * iit

# 与后端服务器通信
## XHR & JSONP by $http
* MongoLab
    * https://api.mongolab.com/api/1/databases/[DB-name]/collections/[collection-name]/[item-id]?apiKey=[secret-key]
* JSONP：angular.callbacks._k
* CORS
    * OPTIONS
* Server-side proxies

## promise with $q
* $q.defer():
    * ~.promise.then(succCb, errCb);
        * then可注册多次
    * ~.resolve(value)
* 最后需要 $rootScope.$digest()
* 异步调用链
    We can return a new promise from an error callback. The returned promise will be part of the resolution chain, and the final consumer won't even notice that something went wrong. （但是如果这个error callback是在一个很长的异步调用链中间时？）
    * return ... .then(...) 或 $q.reject(...)
* $q.all
    * 怎么没有race方法？
* $q.when：包装普通value为promise
* $q在AngularJS中集成
    * promise可作为普通value在model中使用：
    ```
        $scope.name = $timeout(function () {
            return "World";
        }, 2000);
    ```
    * 但是返回promise的函数调用不能直接用在{{...}}模板中

## REST通信with $resource
略

# 显示和格式化数据
* 嵌入html代码：`<p ng-bind-html-unsafe="msg"></p>`
    * 或 ng-bind-html ，需引入ngSanitize
* 条件显示： ng-show / ng-hide , ng-switch-* , ng-if and ng-include
    * ng-switch/if 会添加/删除DOM元素，并创建新scope
* ng-include：动态包含内容，`<div ng-include="'header.tpl.html'"></div>`
* ng-repeat
    ```
    <li ng-repeat="(name, value) in user">
        Property {{$index}} with {{name}} has value {{value}}
    </li>
    ```
    ng-repeat默认会对name属性在输出前进行排序？
    
    ```
    <tbody ng-repeat="user in users" ng-click="selectUser(user)" ng-switch on="isSelected(user)">
    ```
    注意这里user的scope
    
    * 可在ng-repeat作用的元素上定义controller，以显式使用ng-repeat创建的scope
* 事件
    * `<li ... ng-click="logPosition(item, $event)" `
* AngularJS (1.2.x)：ng-repeat不再需要应用到单独的容器元素（而可以是一组元素，fragment）
    ```
    <li ng-repeat-start="item in items">
        <strong>{{item.name}}</strong>
    </li>
    <li ng-repeat-end>{{item.description}}</li>
    ```
* IE不允许动态修改input元素的type属性
    * `<input type="{{myinput.type}}" ng-model="myobject[myinput.model]">`
        * 绕过：`<ng-include src="'input'+myinput.type+'.html'"></ng-include>`
* 过滤器
    略（就一个日期格式化、及数字编号的宽度对齐可能项目中会用到）
    * $
    ```
        ng-repeat="item in filteredBacklog = (backlog | filter:{$: criteria, done: false})"
    ```
    * DI中访问
        * `var limitToFilter = $filter('limitTo');`

# 创建高级表单
暂略（表单不是当前关注的内容）

# 组织导航（前端路由）
## $location服务
* $anchorScroll
    `$anchorScrollProvider.disableAutoScrolling();`
* 不需要#
    `$locationProvider.html5Mode(true);` 但还是需要服务器端配置重定向？
* Structuring pages around routes
    * navbar: `<a href="#/admin/users/list">List users</a>`
    * ng-include：`<div class="container-fluid" ng-include="selectedRoute.templateUrl"> ... </div>`

    ```
    $scope.$watch(function () {
        return $location.path();
    }, function (newPath) {
        $scope.selectedRoute = routes[newPath] || defaultRoute; //ng-include指令将响应此model的变化
    });
    ```

## $route服务
1.2：ngRoute服务被分离到单独的angular-route.js文件。

```
angular.module('routing_basics', [])
    .config(function($routeProvider) {
        $routeProvider
            .when('/admin/users/list', {templateUrl: 'tpls/users/list.html'}) //可以指定controller属性;
            .when('/admin/users/new', {templateUrl: 'tpls/users/new.html'})
            .when('/admin/users/:id', {templateUrl: 'tpls/users/edit.html'})
            .otherwise({redirectTo: '/admin/users/list'});
    })
```

疑问：

0. 动态url path看起来可以做到，注意到url path支持:id占位符这种形式
1. templateUrl可以改用template吗？其内容是替换整个body还是可以指定容器根元素？ng-view指令指定？？
    ```
    <div class="container-fluid" ng-view>
        <!-- Route-dependent content goes here -->
    </div>
    ```
2. config只能执行一次？

* 避免route改变时UI闪烁
    * 尽快显示markup html，并在有数据后再次update —— 这回造成闪烁
    * 确保route改变前，所有后端请求已完成
        * resolve：枚举controller的所有异步依赖项
            * key是注入到controller的变量，对应的value是获取它的函数，可以返回一个promise

* 阻止route改变
    * resolve：function返回一个rejected promise。缺陷：地址栏的新url无法被重置回之前的？
    ```
    If the route's navigation is canceled the browser's address bar won't be reverted and will still read  /users/edit/1234 , even if UI will be still reflecting, content of the  /users/list route.
    ```

* 默认route的局限
    * 更强大的`ui-router` https://github.com/angular-ui/ui-router
    * ng-view只能定义一个矩形区域（hole）—— 但是可以改变hole的位置吧？
    * 不支持route嵌套（url path）
        * 对于使用了iframe tree的Web应用可能比较重要（比如feedly？不对不对，feedly没有使用iframe元素）

* Routing-specific patterns, tips, and tricks
    * I see：`<a ng-href="/admin/users/{{user.$id()}}">Edit user</a>`
    * 链接到外部资源：target="_self"
    * 每个module可config自己的$routeProvider（那么假如module可以动态添加的话，route也可以了？？）
    * 封装 $routeProvider 服务，提供定制的provider，以减少代码冗余
        * TODO
    
# App安全
* $templateCache
* 防止XSS
    * 注意对html格式的动态嵌入内容进行转义即可
* JSON注入（防止JSON代码可执行）
    * $http：有意在response之前注入 ")]}',\n"
* 防止XSRF
    * $http要求服务器端在session cookie中设置一个XSRF-TOKEN
* Adding client-side security support
  略
* 用定制的securityInterceptor服务来处理401响应：略
    * 不过，值得借鉴：可用于实现在会话超时失效的情况下自动重新登录
* route resolve中使用定制的authorization服务
    略

# 定制指令
* 内建指令：https://github.com/angular/angular.js/tree/master/src/ng/directive/

```
  The compile stage is mostly an optimization. It is possible to do almost all the work in the linking function (
  except for a few advanced things like access to the transclusion function). If you consider the case of a 
  repeated directive (inside ng-repeat), the compile function of the directive is called only once, but the 
  linking function is called on every iteration of the repeater, every time the data changes.
```

* 测试

```
element = linkingFn(scope); //实际上，原来模板中对应的DOM元素也一直挂载在document中？
```

```
scope.$digest(); //当测试中使用了$watch、$observe、$q时需要;
```

* A directive definition is an object which ...

* 设置button元素样式（但是button不已经是HTML标准元素吗）

```
myModule.directive('button', function() {
    return {
        restrict: 'E',
        compile: function(element, attributes) {
            element.addClass('btn');
            if ( attributes.size ) {
                element.addClass('btn-' + attributes.size);
            }
        }
    };
});
```
指令不依赖于scope数据：只在compile中处理即可。

* 编写一个分页指令
    * Isolated scope：child scope的prototype断开与parent scope的链接，但是$parent仍然可以引用到？
    * There are three types of interface we can specify between the element's attributes and the isolated scope: 
        interpolate (@), data bind (=), and expression (&).
    * @等价于：
    ```
    attrs.$observe('attribute1', function(value) {
        isolatedScope.isolated1 = value;
    });
    attrs.$$observers['attribute1'].$$scope = parentScope;
    ```
    * =相当于2个$watch：(实际实现要更复杂点)
    ```
    var parentGet = $parse(attrs['attribute2']);
    var parentSet = parentGet.assign;
    parentScope.$watch(parentGet, function(value) {
        isolatedScope.isolated2 = value;
    });
    isolatedScope.$watch('isolated2', function(value) {
        parentSet(parentScope, value);
    });
    ```
    * &提供了一个回调表达式：
    ```
    parentGet = $parse(attrs['attribute3']);
    scope.isolated3 = function(locals) {
        return parentGet(parentScope, locals);
    };
    ```
    * &的用法示例：
    ```
    scope: { ...,
        onSelectPage: '&'
    },
    ...
    scope.onSelectPage({ page: page }); //map传参就相当于scope上的属性一样;
        //==> 使用：on-select-page="selectPageHandler(page)"
    ```
* 编写定制校验指令（需要引用同一级别DOM元素上的scope关联数据）
    * require的用法：
    ```
    require: '^?ngModel',
    link: function(scope, element, attrs, ngModelController) { ... }
    ```
    * ngModelController暴露了下列属性：
        * $parsers
        * $formatters
        * $setValidity(validationErrorKey, isValid)
        * $valid
        * $error
    * link实现：（注意这里参数被重命名了，前后不太一致！）
    ```
    function validateEqual(myValue) {
        var valid = (myValue === scope.$eval(attrs.validateEquals));
        ngModelCtrl.$setValidity('equal', valid);
        return valid ? myValue : undefined;
    }
    ngModelCtrl.$parsers.push(validateEqual);
    ngModelCtrl.$formatters.push(validateEqual);
    scope.$watch(attrs.validateEquals, function() {
        ngModelCtrl.$setViewValue(ngModelCtrl.$viewValue);
    });
    ```
* 创建一个异步校验指令
    * TDD：`<input ng-model="user.email" unique-email>`
    * `ngModelCtrl.$parsers.push(function (viewValue) {...略}`

* 封装jQuery datepicker：(这个例子有点复杂)
    * link实现：
    ```
    ...
    var updateModel = function () {
        scope.$apply(function () {
             var date = element.datepicker("getDate");
             element.datepicker("setDate", element.val());
             ngModelCtrl.$setViewValue(date);
        });
    };
    ...
    ngModelCtrl.$render = function () {
        element.datepicker("setDate", ngModelCtrl.$viewValue);
    };
    ```

# 高级指令
## Using transclusion
* 当元素在DOM树上移动位置到新的scope时，仍然能够"bring the original scope with us"
    ```
    Transclusion is necessary whenever a directive is replacing its original contents with new elements but wants to use the original contents somewhere in the new elements.
    ```
* ng-repeat不同寻常，因为它clone自身元素；更常见的是用于`templated widget`
    1. 创建一个定制的alert指令：
        * 看起来它就是一个scope数据的xml封装，这让人想到最新的W3C规范 template 元素，以及Web Components...
        ```
        <alert type="alert.type" close="closeAlert($index)" ng-repeat="alert in alerts">
            {{alert.msg}}
        </alert>
        ```
        * 这里的type属性映射到指令template的isolate scope？
            * 但transcluded scope（即alert.msg）仍然从原来的parent scope继承...
    2. alert指令的实现
    ```
    myModule.directive('alert', function () {
        return {
            restrict:'E',
            replace: true,
            transclude: true,
            template:
                '<div class="alert alert-{{type}}">' +
                    '<button type="button" class="close" ng-click="close()">&times;</button>' +
                    '<div ng-transclude></div>' +
                '</div>',
            scope: { type:'=', close:'&' }
        };
    });
    ```
    3. 理解`replace: true`
        * 属性将从<alert>拷贝到template的div元素上
        * 如果指定了template但未指定replace，则template内容会append到指令元素（<alert>）
    4. 理解`transclude`属性
        * true：transclude的是指令元素的children
        * element：transclude的是整个指令元素（对应于ng-repeat的例子）
    5. 理解transclusion的scope
        * 指令template中的表达式无法访问指令元素所在的parent scope？？？

## Creating and working with transclusion functions
* `transclude: true`：

```
    var elementsToTransclude = directiveElement.contents();
    directiveElement.html('');
    var transcludeFunction = $compile(elementsToTransclude);
```

* clone when transcluding：

```
var clone = linkingFn(scope, function callback(clone) {
    element.append(clone);
});
```

* 在指令中访问transclusion函数：

```
myModule.directive('myDirective', function() {
    return {
        transclude: true,
        compile: function(element, attrs, transcludeFn) { ... },
        controller: function($scope, $transclude) { ... },
    };
});
```

* 访问transclusion函数所在的scope：
```
compile: function(element, attrs, transcludeFn) {
    return function postLink(scope, element, attrs, controller) {
        var newScope = scope.$parent.$new();
        element.find('p').first().append(transcludeFn(newScope));
    };
}
```

* 改在controller中访问：注入的$transclude已绑定scope！

```
controller: function($scope, $element, $transclude) {
    $element.find('p').first().append($transclude());
}
```

* 创建一个if指令，使用transclusion函数（而不是ng-transclude）
    略

* `transclude: 'element'`情况下priority属性的处理
    * The `ng-repeat` directive has `transclude: 'element'` and `priority: 1000`, 
        * so generally all attributes that appear on the ng-repeat element are transcluded to appear on the cloned repeated elements.

## Understanding directive controllers
* 指令控制器 vs ng-controller
    * TODO
* 注入特殊依赖：
    1. $element
    2. $attrs
    3. $transclude
* 创建一个基于指令控制器的分页指令

```
controller: ['$scope, '$element', '$attrs',
    function($scope, $element, $attrs) {
        ...
    }]
```

* vs link函数

```
If an element contains multiple directives then for that element:
    • A scope is created, if necessary
    • Each directive's directive controller is instantiated
    • Each directive's pre-link function is called
    • Any child elements are linked
    • Each directive's post-link function is called
```

link函数可通过require注入控制器参数，而指令控制器不能注入其他的指令控制器。（并非实现上不可能吧？）

* 用法示例：accordion

```
myModule.controller('AccordionController', ['$scope', '$attrs',
    function ($scope, $attrs) {
        ...
    }
    ]);

myModule.directive('accordion', function () {
    return {
        restrict:'E',
        controller:'AccordionController',
            //accordion指令引入的controller可被其子指令通过require注入到子指令的link;
        link: function(scope, element, attrs) {
            element.addClass('accordion');
        }
    };
})

myModule.directive('accordionGroup', function() {
    return {
        require:'^accordion',
        //下略
 ```

## 手工控制compile过程
* TDD：

```
<field type="email" ng-model="user.email" required >
    <label>Email</label>
    <validator key="required">$fieldLabel is required</validator>
    <validator key="email">Please enter a valid email</validator>
</field>
```

* 定制compile实现：

```
priority: 100, //在ng-model之前执行;
terminal: true,
compile: function(element, attrs) {
  ...
  var validationMgs = getValidationValidationMessages(element);
  var labelContent = getLabelContent(element);
  element.html('');
  return function postLink(scope, element, attrs) {
    var template = attrs.template || 'input.html';
    loadTemplate(template).then(function(templateElement) {
    ... 
    });
  };
}
```

* 需要自己处理`{{}}`插值：$interpolate服务
  略

* 动态加载template

```
function loadTemplate(template) {
    return $http.get(template, {cache:$templateCache})
        .then(function(response) {
            return angular.element(response.data);
        }, function(response) {
            throw new Error('Template not found: ' + template);
        }
    );
}
```

... faint，下面还有一堆代码 ... skip

# 国际化和本地化
* `<span>{{'greetings.hello' | i18n}}, {{name}}!</span>` 用了一个过滤器？
* `i18n key='greetings.hello'></i18n>`
    * 不再需要附加的$watch（提高了性能？）
* Translating partials during the build-time
    * Grunt.js模版？`<%= greeting.hello %>`

# 编写健壮的Web应用
* 理解AngularJS的inner workings
    * `scope.$watch(watchExpression, modelChangeCallback)`
    * scope.$apply
        * Enter the $digest loop
        ```
        AngularJS makes sure that all the model values are calculated and "stable" before giving control back to the DOM rendering context. This way UI is repainted in one single batch, instead of being constantly redrawn in response to individual model value changes.
        ```
        * Model stability
            * dirty-checking算法：watchExpression至少会被求值2次！
            * 每次都必须从$rootScope开始，求值所有$watch
* 性能优化（边界）

```
AngularJS, as any other well-engineered library, was constructed within a frame of certain boundary conditions, and those are best described by Misko Hevery, father of AngularJS (http://stackoverflow.com/a/9693933/1418796):
Humans are:
    slow：小于50ms但延迟感知不到
    limited（信息过载的问题）
So the real question is this: can you do 2000 comparisons in 50 ms even on slow browsers?
```

* 性能测量
Batarang allows us to easily pinpoint the slowest watch expressions（这个要试一试）

* Avoid DOM access in the watch-expression
    * CSS渲染属性的访问会触发reflow？
    ```
    The entire AngularJS philosophy is based on the fact that the source of truth is the model. It is the model that drives declarative UI. But by observing DOM properties, we are turning things upside-down!
    ```

* 不要写 `{{myComplexComputation()}}`

* Don't watch for invisible

* call scope.$digest if 确切知道哪个scope受到model修改的影响

* Remove unused watches
 
    ```
    var watchUnregisterFn = $scope.$watch(...)
    watchUnregisterFn(); //remove this watch;
    ```

* Entering the $digest loop less frequently
    * `$timeout(update, 1000, false);`
    * 每次mouse移动(travels over)都会触发$digest loop：
        * `<div ng-class='{active: isActive}' ng-mouseenter ='isActive=true' ng- mouseleave='isActive=false'>Some content</div>`
        * ? 考虑使用定制指令，以响应事件，修改DOM... how?

* Avoid deep-watching whenever possible（可能导致大的内存消耗）
    * $scope.$watch(expr, cb, true); //<-- 把这里的expr改成function类型，返回序列化后的string

# 打包和部署
* Preloading templates
    1. `<script type="text/ng-template" id="tpls/users/list.html"> ... </scriopt>`
    2. 使用$templateCache服务
* Optimizing the landing page
    * ng-cloak 将元素设置为display:none; 直到`{{}}`里的model数据准备好
    * 将`{{name}}`改写成`<span ng-bind="name"></span>`
* 与AMD
    * `ng-app` => `angular.bootstrap`, This way you can control the timing of AngularJS kickstarting the application.
