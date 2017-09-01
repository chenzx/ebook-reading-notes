# Node Cookbook 3rd-Packt Publishing 2017-ReadingNotes.md

## 调试进程
* 用DevTools调试Node
	```
	$ mkdir app
	$ cd app
	$ npm init -y
	$ npm install --save express
	$ node --inspect index.js (6.3.0+，这个调试真的不错啊，酷毙了！)
	```

	How it works... 这段内容很有意思，因为它涉及了v8的debugger协议细节
* 8+ 启动时暂停：`node --inspect-brk index.js`
* 命令行调试：`node debug index.js`
* Enhancing stack trace output
	* 增加栈深度显示：`--stack-trace-limit=21`	
	* 进一步在代码中指定：`Error.stackTraceLimit = Infinity`
* Asynchronous stack traces
	* `$ npm install --save-dev longjohn` 这是什么原理？
* 启用debug logs：
	* `DEBUG=* node index.js`
* 启用core debug logs：
	* 特殊的环境变量：`$ NODE_DEBUG=timer node index.js`
	* NODE_DEBUG可设置为下列模块：http net tls stream module timer cluster child_process fs
	
## 编写模块
## Coordinating I/ O
## Using Streams
## Wielding Web Protocols
## 持久化到数据库
## 使用Web框架
* express
* Hapi
* Koa
## 处理安全
## 优化性能
* HTTP性能基准测试
* Finding bottlenecks with flamegraphs（用火焰图来寻找瓶颈）
* 优化同步函数调用
* 优化异步回调
* Profiling memory
## 构建微服务系统
## Deploying Node.js
## 
