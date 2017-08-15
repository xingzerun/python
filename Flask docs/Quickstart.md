## 一个最小的应用  

	from flask import Flask
	app = Flask(__name__)
	
	@app.route('/')
	def hello_world():
	    return 'Hello World!'
	
	if __name__ == '__main__':
	    app.run()
**外不可访问的服务器:**  
app.run(host='0.0.0.0')  
这会让操作系统监听所有公网IP。

## 调试模式
有两种方法：  
一种是直接在应用对象上设置：

	app.debug = True
	app.run()
另一种是作为run方法的一个参数传入：

	app.run(debug=True)
**注意:**  
尽管交互式调试器在允许 fork 的环境中无法正常使用（也即在生产服务器上正常使用几乎是不可能的），但它依然允许执行任意代码。这使它成为一个巨大的安全隐患，因此它 **绝对不能用于生产环境**。
## 路由

	@app.route('/')
	def index():
	    return 'Index Page'
	
	@app.route('/hello')
	def hello():
	    return 'Hello World'
但是，不仅如此！你可以构造含有动态部分的 URL，也可以在一个函数上附着多个规则。  
### 变量规则

	@app.route('/user/<username>')
	def show_user_profile(username):
	    # show the user profile for that user
	    return 'User %s' % username
	
	@app.route('/post/<int:post_id>')
	def show_post(post_id):
	    # show the post with the given id, the id is an integer
	    return 'Post %d' % post_id
转换器有下面几种：    
![](http://i.imgur.com/Ao5oJk2.png)  
**唯一URL/重定向行为：**  
Flask 的 URL 规则基于 Werkzeug 的路由模块。这个模块背后的思想是基于 Apache 以及更早的 HTTP 服务器主张的先例，保证优雅且唯一的 URL。  

	@app.route('/projects/')
	def projects():
	    return 'The project page'
	
	@app.route('/about')
	def about():
	    return 'The about page'
虽然它们看起来着实相似，但它们结尾斜线的使用在 URL 定义 中不同。 第一种情况中，指向 projects 的规范 URL 尾端有一个斜线。这种感觉很像在文件系统中的文件夹。访问一个结尾不带斜线的 URL 会被 Flask 重定向到带斜线的规范 URL 去。  
然而，第二种情况的 URL 结尾不带斜线，类似 UNIX-like 系统下的文件的路径名。访问结尾带斜线的 URL 会产生一个 404 “Not Found” 错误。  
这个行为使得在遗忘尾斜线时，允许关联的 URL 接任工作，与 Apache 和其它的服务器的行为并无二异。此外，也保证了 URL 的唯一，有助于避免搜索引擎索引同一个页面两次。
### 构造URL
	from flask import Flask, url_for 
	app = Flask(__name__)
	@app.route('/')
	def index():pass
	
	@app.route('/login')
	def login():pass
	
	@app.route('/user/<username>')
	def profile(username):pass
	
	with app.test_request_context():
		print(url_for('index'))
		print(url_for('login'))
		print(url_for('login', next='/'))
		print(url_for('profile', username='John Doe'))
为什么要构建URL而非在模板中硬编码？  
1. 反向构建通常比硬编码的描述性更好。更重要的是，它允许你一次性修改URL，而不是到外边找边改。
2. URL构建会转义特殊字符的Unicode数据，免去你很多麻烦。
3. 如果你的应用不位于URL的根路径（比如，在/myapplication下，而不是/），url_for()会妥善处理这个问题。
### HTTP方法
默认情况下，路由只回应GET请求，但是通过route()装饰器提供methods参数可以更改这个行为。
	
	@app.route('/login',methods=['GET','POST'])
	def login():
		if request.method == 'POST':
			do_the_login()
		else:
			show_the_login_from()
常见的HTTP方法：

* GET
* HEAD
* POST
* PUT
* DELETE
* OPTIONS
## 静态文件
动态web应用也需要静态文件，通常是CSS和JavaScript文件的存放位置。理想情况下，已经配置web服务器来提供他们，但是在开发中，Flask也可以做到。需要在包中或模块旁边创建一个名为static的文件夹，在应用中使用/static即可访问。  
给静态文件生成URL，使用特殊的'static'端点名：

	url_for('static',filename='style.css')
这个文件应该存储在文件系统上的static/style.css
## 模板渲染
在Python里生成HTML十分无趣，而且相当繁琐，因为需要自行对HTML做转义来保证应用安全。因此，Flask自动配置了Jinja2模板引擎。  
你可以使用render_template()方法来渲染模板。需要做的事：将模板名和想作为关键字的参数传入模板的变量。
	
	from flask import render_template

	@app.route('/hello/')
	@app.route('/hello/<name>')
	def hello(name=None):
		return render_template('hello.html', name=name)
Flask会在templates文件夹里寻找模板。所以，如果你的应用是这个模板，这个文件夹在模板的旁边；如果它是一个包，那么这个文件夹在你的包里面：  
**情况1：模板：**

	/application.py
	/templates
		/hello.html
**情况2：包**

	/application
		/__init_.py
		/templates
			/hello.html

这是一个模板实例：

	<!doctype html>
	<title>Hello from Flask</title>
	{% if name %}
		<h1>Hello {{ name }}!</h1>
	{% else %}
		<h1>Hello World!</h1>
	{% endif %}
在模板里，你可以访问request、session和g对象（它允许你按需存储信息，可以查看g对象的文档和在Flask中使用SQLLite3的文档获取更多信息），以及get_flashed_messages()函数。  
自定义功能默认是开启的，所以如果name包含HTML，它将会被自动转义。如果你能信任一个变量，并且你知道它是安全的（例如一个模块吧Wiki标记转换为HTML），你可以用Markup类或|safe过滤器在模板中把它标记为安全的。  
这是一个Markup类如何使用的简单介绍：
![](http://i.imgur.com/VrJlCVd.png)
## 访问请求数据
对于Web应用，与客户端发送给服务器的数据交互至关重要。在Flask中由全局的request对象来提供这些信息，如果你有一定的Python经验，你会好奇，为什么这个对象是全局的，为什么Flask还能保证线程安全。答案是环境作用域：
### 环境局部变量
Flask中的某些对象是全局对象，但却不是通常的那种。这些对象实际上是特定环境的局部对象的代理。虽然很拗口，但实际上很容易理解。  
处理线程的环境：一个请求传入，Web服务器决定生成一个新线程（或者别的什么东西，只要这个底层的对象可以胜任并发系统，而不仅仅是线程）。当Flask开始它内部的请求处理时，它认定当前线程是活动的环境，并绑定当前的应用和WSGI环境到那个环境上（线程）。它的实现很巧妙，能保证一个应用调用另一个应用时不会出现问题。  
所以，除非需要做类似单元测试的东西，否则基本可以完全无视它。你会发现依赖一段请求对象的代码，因没有请求对象无法正常运行。解决方案是，自行创建一个请求对象并且把它绑定到环境中。单元测试的最简单的解决方案是：用test_request_context()环境管理器。结合with声明，绑定一个测试请求，这样你才能与之交互。下面是一个例子：

	from flask import request
	
	with app.test_request_context('/hello', method='POST'):
		# now you can do something with the request until the 
		# end of the with block, such as basic assertions:
		assert request.path == '/hello'
		assert request.method == 'POST'
另一种可能是：传递整个WSGI环境给request_context()方法：

	from flask import request
	
	with app.request_context(environ):
		assert request.method == 'POST'
### 请求对象
导入：  

	from flask import request
当前请求的HTTP方法可通过method属性来访问。通过attr：~flask.request.from属性来访问表单数据（POST或PUT请求提交的数据）。这里有一个用到上面提到的那两个属性的完整实例：	

	from flask import request
	
	@app.route('/login', methods=['POST', 'GET'])
	def login():
		error = None
		if request.method == 'POST':
			if valid_login(request.form['username'],
							request.form['password']):
				return log_the_user_in(request.form['username'])
			else:
				error = 'Invalid username/password'
		# the code below is executed if the request method
		# was GET or the credentials were invalid
		return render_template('login.html', error=error)
当访问的form属性中不存在的键会抛出一个KeyError异常。你可以像捕获标准的KeyError一样来捕获它。如果你不愿意这么做，它会显示一个HTTP 400 Bad Request错误页面。所以，多数情况下你并不需要干预这个行为。  
通过args属性来访问URL中提交的参数（？key=value）：

	searchword = request.args.get('q', '')
推荐使用get来访问URL参数或捕获KeyError，因为用户可能会修改URL，向他们展现一个400 bad request页面会影响用户体验。
（欲获取请求对象的完整方法和属性清单，请参阅request的文档）
### 文件上传
在HTML表单中设置enctype="multipart/form-data"属性，就可以很容易的用Flask处理文件上传，否则你的浏览器将根本不提交文件。
已上传的文件存储在内存或是文件系统上的临时位置。你可以通过请求对象的files属性访问那些文件。每个上传的文件都会存储在那个字典里。它表现得如同一个标准的Python file对象，但它还有一个save()方法来允许你在服务器的文件系统上保存它。  
这里有一个它是如何工作的例子：

	from flask import request
	
	@app.route('/upload', methods=['GET', 'POST'])
	def upload_file():
		if request.method == 'POST':
			f = request.files['the_file']
			f.save('/var/www/uploads/uploaded_file.txt')
		...	
通过访问filename属性，获取上传前文件在客户端的文件名。但是这个值是可以伪造的。如果想要使用客户端的文件名来在服务器上存储文件，把它传递个Werkzeug提供的secure_filename()函数:

	from flask import request
	from werkzeug import secure_filename
	
	@app.route('/upload', methods=['GET','POST'])
	def upload_file():
		if request.method == 'POST':
			f = requst.files['the_file']
			f.save('/var/www/uploads/' + secure_filename(f.filename))
		...
### Cookies
* 通过Cookies属性来访问cookies。
* 通过响应对象的set_cookie方法设置cookies
* 请求对象的cookies属性是一个客户端提交的所有cookies字典
* 如果你想使用会话，请不要直接使用cookies，参考**会话**一节
* 在Flask中，已经在cookies上增加了一些安全细节

读取cookies:

	from flask import request
	
	@app.route('/')
	def index():
		username = request.cookies.get('username')
		# use cookies.get(key) instead of cookies[key] to not get a
		# KeyError if the cookies is missing
存储cookies:

	from flask import make_response
	
	@app.route('/')
	def index():
		resp = make_response(render_template(...))
		resp.set_cookie('username','the username')
		return resp
cookies是设置在响应对象上。由于通常只是从视图函数返回字符串，Flask会将其转换为响应对象。如果你显示地想要这么做，你可以使用make_response()函数然后修改它。  
想要在响应对象不存在的时候设置cookie，这在利用延迟请求回调模式时是可行的。  
可以参阅**关于响应**
