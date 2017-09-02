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
* npm config set init.author.name "< name here >"
	* npm init（package.json文件里似乎没有依赖字段？）
	* npm version 1.0.0
* 安装依赖：
	* npm install --save hsl-to-rgb-for-reals
	* --save-dev
* npm run lint
* 安装全局依赖不需要sudo（实质是修改了prefix路径）：`sudo chown -R $(whoami) $(npm config get prefix)/{lib/node_modules,bin,share}`
	* 或：`npm config set prefix ~/ npm-global`（需要添加到PATH中）
* node -p "require('./')( 0, 100, 100)" （-p选项有点像Perl里的-e）
* 测试：
	* tap 代码覆盖？
* 发布模块
	* npm login
	* npm publish --access = public
* 模块安全
	* npm i -g auditjs
	* prepublish（略）
	* 发布到IPFS：npm install -g stay-cli
* 使用私有仓库（npm官方也是使用的这个实现吗？还是说仅仅是mock的api？）
	* npm install -g sinopia && sinopia &> /dev/null &
	* npm set registry http://localhost:4873

## Coordinating I/O
* setInterval(() => process.stdout.write('.'), 10).unref()
* 监视目录：chokidar

## Using Streams
* stream事件
	* data
	* end（输入流）--> finish（输出流）
	* close，error（不保证一定有触发）
	* pause／resume（都是针对ReadableStream的！pull模型）
* 使用pipe
	* ！产品环境中不要使用，推荐`pump`（见鬼）
	* backpressure
	* content.pipe( socket, {end: false})
		* content.on('end', ()=>{socket.end()})
	* 验证：同一input可以pipe到多个output？？？
* pipe缺少error处理？？？！不早说，tnnd
	* npm install --save pump
	* pump(input, output, (error)={...})
	* 避免自己手写的boilerplate代码：
		* stream.pipe(res); res.on('close', ()=>{stream.destroy()})
* pumpify：忽略pipeline中间的transform流，只展示开始的input和最终的output？
* 创建自己的transform流：through2 ？
	* 推荐使用readable-stream，而不是core stream，以屏蔽不同Node版本的差异？
* 对象流*
* from2、to2（见鬼）
* stream.Readable/Writable
* flow control：
	* readable流的_read方法依赖于push的驱动，怎么以异步方式连续push多次呢？
	* 包from2 + 嵌套setTimeout？？？
* duplex流：const stream = duplexify(ws, rs)
* Decoupling I/O（？？？）
* By default, writable streams have a high watermark of 16,384 bytes (16 KB). 如果超过了仍继续写入就会内存泄露。
	* pipe控制的write每次写不会超过16KB，但直接write可以超过：
		* `rs.on(' data', (chunk) = > ws.write( chunk))`
		* 这种情况下，需要检查write的返回值，true代表可以继续写，false说明需要暂停，直到drain事件触发：
			```
			rs.resume())
			ws.once('drain', () = > rs.resume())
			```

## Wielding Web Protocols
* 上传文件via PUT和xhr2: 。。。
* `const buffer = Buffer.allocUnsafe(size); ... chunk.copy(buffer, index)`

## 持久化到数据库
* Postgres jsonb类型？
* $ npm install --save hiredis（比纯JS的包redis更快？）
* LevelDB？跟SQLite属于嵌入式环境使用吧

## 使用Web框架
* express
* Hapi
* Koa（v2: 8+）
```
const router = require('koa-router')()
router.get('/', async function (ctx, next) {
	await next()
	const { title } = ctx.state
	ctx.body = `...`
}, async (ctx) = > ctx.state = {...});
```

## 处理安全
* express：用helmet增强http响应头部*
* 防御XSS：url请求参数中用JS嵌入特殊的token（？似乎不是根本的解决方法？）
	* 协议handler类型的XSS（javascript:...）
* 防御CSRF：cookie sameSite选项？

## 优化性能
* HTTP性能基准测试
	* autocannon：POST请求
* Finding bottlenecks with flamegraphs（用火焰图来寻找瓶颈）
	* 0x ？ --pref-basic-prof 使用HTML+D3.js生成图表？
* 优化同步函数调用
	* node --trace-opt --trace-deopt app.js
* 优化异步回调
* Profiling memory

## 构建微服务系统
* fuge？？？
* 服务发现机制：dns？
	* Consul.io
	* etcd
	* zookeeper
	* P2P协议，如Raft

## Deploying Node.js
* kops: 管理多个k8s集群？？？
