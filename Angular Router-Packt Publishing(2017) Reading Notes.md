Angular Router-Packt Publishing(2017) Web Draft
Reading Notes

# 路由是做什么的？
wait，怎么感觉这本书不是讲AngularJS 1.x的？？

# Overview
真的是Ng2的代码，见鬼。而且为什么不能聪pdf种复制？
 
# URLs

# URL匹配
／outbox／234/(key:value)
* 圆括号：outlets？

# Redirects
前端路由也要来“重定向”？

# 路由状态
* RouterStateSnapshot
* ActivatedRoute

# Links和Navigation
ng2把url根据／分解？url也要结构化地处理？

# 懒惰加载
me：用ng2写一个像feedly那样复杂的应用？？

# Guards
## 4种类型的guards：
* canLoad
* canActivate
* canActivateChild
* canDeactivate

# 事件
## 启用跟踪

```
@NgModule({
	import: [RouteModule.forRoot(routes, {enableTracing: true})]
})
class MailModule{
}
platformBrowserDynamic().bootstrapModule(MailModule);
```

## 监听事件

```
class MailAppCmp {
	constructor(r: Router) {
		r.events.subscribe(e => {...})
	}
}
```
这里的代码风格让我想到了RxJS...

# 测试

# 配置
