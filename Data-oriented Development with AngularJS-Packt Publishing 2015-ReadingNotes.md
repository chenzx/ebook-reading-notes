# Data-oriented Development with AngularJS-Packt Publishing 2015

## AngularJS Rationale and Data Binding
skip

## Working with Data
skip

## Custom Controls
* &attr
```
app.directive("myEmployee", function () {
    return {
        restrict: 'E',
        scope : {
            'click': '&onClick'
        },
        templateUrl: 'employee.tpl.html'
    };
});
```

onClick的定义：（对应这里的on-click属性）
```
<my-employee on-click="buttonClick(message)">
</my-employee>
```

employee.tpl.html:
```
<input type="button" ng-click="click({message: 'This msg comes from the directive'})" value="Click me!" />
```

注意，这里指令template里面为Isolate scope。

* Transclusion
    * 为何必须是指令template去wrap DOM内容呢？
    ```
    app.directive("address", function () {
        return {
            require: '^myEmployee',
            restrict: 'E',
            scope: {
                type: '@'
            },
            transclude: true,
            link: function (scope, element, attrs, myEmployeeCtrl) {
                console.log(myEmployeeCtrl.getName() + ' ' + scope.type);
            },
            template: '<div ng-transclude style="background-color:powderblue"></div>'
        };
    });
    ```

## Firebase
skip

* Three-way data binding ?

## AngularFire
skip

## Applied Angular and AngularFire

## Appendix A: Yeoman
* 生成器？RoR？

## Appendix B: Git and Git Flow
skip


