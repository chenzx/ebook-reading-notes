Flask_ Building Python Web Services-Packt Publishing(2017)-ReadingNotes.md / Learning Path

# Flask By Example
## Hello World!
```
“from flask import Flask

app = Flask(__name__)

@app.route("/")
def index():
    return "Hello, World!"

if __name__ == '__main__':
    app.run(port=5000, debug=True)”
```

* 部署到VPS
	* hello.wsgi
	```
	import sys
	sys.path.insert(0, "/var/www/firstapp")
	from hello import app as application
	```
	* /etc/apache2/sites-available／hello.conf
	```
	<VirtualHost *>
	    ServerName example.com

	    WSGIScriptAlias / /var/www/firstapp/hello.wsgi
	    WSGIDaemonProcess hello
	    <Directory /var/www/firstapp>
	       WSGIProcessGroup hello
	       WSGIApplicationGroup %{GLOBAL}
	        Order deny,allow
	        Allow from all
	    </Directory>
	</VirtualHost>
	```
	* 启用
	```
	sudo a2dissite 000-default.conf
	sudo a2ensite hello.conf
	sudo service apache2 reload
	```

## Getting Started with Our Headlines Project
* `pip install --user feedparser`
```
	feed = feedparser.parse(BBC_FEED)
  	first_article = feed['entries'][0]
```
* url路由参数
```
@app.route("/<publication>")
def get_news(publication="bbc"):
	...
```

## 使用模板（Jinja2）
```
from flask import render_template

...
return render_template("home.html", title=first_article.get("title"),summary=first_article.get("summary"))
	#使用命名参数？-- 也可以传递一个python对象
```

带循环的Jinja模板：

```
<body>
    <h1>Headlines</h1>
    {% for article in articles %}
        <b>{{article.title}}</b><br />
        <i>{{article.published}}</i><br />
        <p>{{article.summary}}</p>
        <hr />
    {% endfor %}
</body>
```

## 处理用户输入
* url查询参数： `query = request.args.get("publication")`
	* 这里request是隐式变量？还是说由`@app.route("/")`装饰器注入的？

## POST参数
* Change `request.args.get` to `request.form.get` (mdzz，这里API风格就不一致了)
* `@app.route("/", methods=['GET', 'POST'])`
* [天气API](https://home.openweathermap.org/users/sign_in) JSON解析：
```
	data = urllib2.urlopen(url).read()
	parsed = json.loads(data)
```
* 汇率？ `CURRENCY_URL = "https://openexchangerates.org//api/latest.json?app_id=<your-api-key-here>`

## 提高用户体验
* cookies
	* 这里的API用法太繁琐了，简直就是C语言的风格...
* 添加CSS：外部、内部、inline

## Building an Interactive Crime Map
* import pymysql
	* 这个MySQL查询API也非常原始，不知道性能与PHP或JDBC的相比如何？
	```
	def add_input(self, data):
	    connection = self.connect()
	    try:
		    # The following introduces a deliberate security flaw. See section on SQL injection below
		    query = "INSERT INTO crimes (description) VALUES ('{}');".format(data)
		    with connection.cursor() as cursor:
        		cursor.execute(query)
        		connection.commit()
    	finally:
      		connection.close()
	```
	* 防御SQL注入攻击: query中使用%s作为占位符，同时带参数的cursor.execute(query, data)

## 集成Google Maps
略

## 校验用户输入
* Validating versus sanitizing

## Building a Waiter Caller App
* 使用Bootstrap
	* 所谓的`class="col-md-4”`
* 处理用户登录

```
from flask.ext.login import LoginManager
from flask.ext.login import login_required

app = Flask(__name__)
login_manager = LoginManager(app)

@login_required
...
```

下略

## Template Inheritance and WTForms
* CSRF

## Using MongoDB

这就完了？Flask这个库感觉不昨的

# Flask Framework Cookbook
## Flask配置
## Templating with Jinja2（现在不太喜欢这种后端模板了！）
* 完全可以使用前端框架+前端模板编译，再加上SSR（服务器端渲染）

## Data Modeling in Flask
* $ pip install flask-sqlalchemy
```
class Product(db.Model):
    id = db.Column(db.Integer, primary_key=True)
    name = db.Column(db.String(255))
    price = db.Column(db.Float)
```

就语法上看，Spring Data比sqlalchemy好不到哪里去，而且Spring本身还需要编译。（不过Python就算没有JIT，其性能不应该是高并发情况下的瓶颈吧？）

不过ORM的优点是能够生成代码：

```
@catalog.route('/product/<id>')
def product(id):
    product = Product.query.get_or_404(id)
    return 'Product - %s, $%s' % (product.name, product.price)
```

* Database migration using Alembic and Flask-Migrate
	* 数据库迁移的概念都是来自于Ruby on Rails吧？

## Views
## Webforms with WTForms
* CSRF保护

```
app.config['WTF_CSRF_SECRET_KEY'] = 'random key for form”

<form method="POST" action="/some-action-like-create-product">
    {{ form.csrf_token }}
</form>
```

## Authenticating in Flask
## RESTful API Building
## Internationalization and Localization
## Debugging, Error Handling, and Testing
* pdb调试: `import pdb; pdb.set_trace()`
* Using mocking to avoid real API access

## Deployment and Post Deployment
* Apache
* uWSGI and nginx
* GUnicorn and Supervisor
* Tornado

```
from tornado.wsgi import WSGIContainer
from tornado.httpserver import HTTPServer
from tornado.ioloop import IOLoop
from my_app import app

http_server = HTTPServer(WSGIContainer(app))
http_server.listen(5000)
IOLoop.instance().start()
```
* Fabric
* S3文件上传
	* $ pip install boto
* Heroku
* AWS Elastic Beanstalk
* Pingdom应用监控
* New Relic APM

## Other Tips and Tricks
*  `Celery` is a task queue for Python.

# Mastering Flask
## Creating Models with SQLAlchemy
* Relationships between models
	* 这让我想起以前配置Hibernate的经历了，矬～
* SQLAlchemy sessions：handler of transactions
## Creating Controllers with Blueprints
“In Flask, a blueprint is a method of extending an existing Flask app.” ？
## Advanced Application Structure
## Securing Your App
## Using NoSQL
* CRUD（mongodb）
* Relationships in NoSQL
```
class Post(mongo.Document):
    title = mongo.StringField(required=True)
    text = mongo.StringField()
    publish_date = mongo.DateTimeField(
        default=datetime.datetime.now()
    )
    user = mongo.ReferenceField(User)
    comments = mongo.ListField(
        mongo.EmbeddedDocumentField(Comment)
    )
```
## Building RESTful APIs
* 怎么又搞一套请求处理框架？？

## Creating Asynchronous Tasks with Celery
## Useful Flask Extensions
* Flask Assets：资源文件的优化（NodeJS里面一般交给build系统处理）
## Building Your Own Extension
## Testing Flask Apps
## Deploying Flask Apps
* Gevent（现在不应该使用asyncio了吗）

