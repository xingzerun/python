# 第1章 安装
## 1. 虚拟环境的搭建
### 1.1 安装虚拟环境
检查是否安装了virtualenv:  

	$virtualenv --version
如果出错，怎需要安装该工具：  
Ubuntu版本：  

	$sudo apt-get install python-Virtualenv
Mac OSX系统:
	$sudo easy_install virtualenv
Windows或者其他没有官方virtualenv包的操作系统：

	$python ez_setup.py
	$easy_install virtualenv
上述命令必须以管理员权限执行。  
### 1.2 新建flasky文件夹
现在新建一个文件夹，用来保存示例代码（从Github库中获取）。

	$ git clone https://github.com/miguelgrinberg/flask.git
	$ cd flasky
	$ git checkout la
### 1.3 创建并使用Python虚拟环境
该命令只有一个必需的参数，即虚拟环境的名字。

	$ virtualenv venv
flasky文件夹中就有一个名为venv的子文件夹，它保存一个全新的虚拟环境，其中有一个私有的Python解释器。在使用这个虚拟环境之前，需要先将其“激活”。如果使用bash命令行（Linux和Mac OS X用户），可以通过如下命令激活这个虚拟环境：  

	$ source venv/bin/activate
如果使用Windows:

	$ venv\Scripts\activate
激活虚拟环境，Python解释器的路径就被添加进PATH，但是这种改变不是永久性的，它只会影响当前的命令行会话。  
当虚拟环境中的工作完成，可以输入deactivate回到全局Python解释器中。
## 2. 使用pip安装Python包
### 在虚拟环境中安装Flask：
	
	(venv) $ pip install flask
测试Flask是否正确安装：

	(venv) $ python
	>>> import flask
	>>> 

# 第２章　程序的基本结构
## 2.1 初始化
所有的Flask程序都必须创建一个程序实例。Web服务器使用一种名为Web服务器网关接口（WSGI）的协议，把接收自客户端的所有请求都转交给这个对象处理。程序实例是Flask类的对象，经常使用下述代码创建：

	from flask import Flask
	app = Flask(__name__)
Flask类的构造函数只有一个必须指定的参数，即程序主模块或包的名字。大多数程序中，Python的__name__变量就是所需的值。
## 2.2 路由和视图函数
客户端（例如Web浏览器）把请求发送给Web服务器，Web服务器再把请求发送给Flask程序实例。程序实例需要知道对每个URL请求运行哪些代码，所以保存了一个URL到Python函数的映射关系。处理URL和函数之间关系的程序称为**路由**。
在Flask中定义路由的最简方式是使用程序实例提供的app.route修饰器，把修饰的函数注册为路由。如下：

	@app.route('/')
	def index():
		return '<h1>Hello World!</h1>'
前例把index()函数注册为程序根地址的处理程序。如果部署程序的服务器域名为www.example.com，在浏览器中访问http://www.example.com后，会触发服务器执行index()函数。这个函数的返回值称为响应，是客户端接收到的内容。如果客户端是Web浏览器，响应就是显示给用户查看的文档。index()被称为视图函数（view function）。视图函数返回的响应可以是包含HTML的简单字符串，也可以是复杂表单。  
**注意：  
在Python代码中嵌入响应字符串会导致代码难以维护，此处这么做只是为了介绍响应的概念。**  
### 构造URL

	@app.route('/user/<name>')
	def user(name):
		return '<h1>Hello, %s!</h1>' % name
尖括号的内容就是动态部分，任何能匹配静态部分的URL都会映射到这个路由上。  
路由中的动态部分默认使用字符串，不过可使用类型定义。例如，路由/user/<int:id>只会匹配动态片段id为整数的URL。Flask还支持在路由中使用int、float和path类型。path类型也是字符串，但不把斜线视作分隔符，而将其当作动态片段的一部分。
## 2.3 启动服务器
程序实例用run方法启动Flask集成的开发Web服务器：  

	if __name__ == '__main__':
	    app.run(debug=True)  

__name__ == '__main__'是Python的惯常用法，在这里确保直接执行这个脚本时才启动开发Web服务器。如果由其他脚本引入，程序假定父级脚本会启动不同的服务器，因此不会执行app.run()。  
服务器启动后，会进入轮询，等待并处理请求。轮询会一直运行，直到程序停止，比如按Ctrl-C。  
启动调试模式会带来一些便利，比如说激活调试器和重载程序。想要启动调试模式，我们把debug参数设为True。
## 2.4 一个完成的程序
新建hello.py文件:

	from flask import Flask
	app = Flask(__name__)
	
	@app.route('/')
	def index():
		return '<h1>Hello World!</h1>'
	
	if __name__ == '__main__':
	    app.run(debug=True)
想要运行这个程序，请确保激活了创建的虚拟环境，并在其中安装了Flask。现在打开Web浏览器，输入http://127.0.0.1:5000/。
启动程序:

	(venv) $ python hello.py
如果输入其他地址，程序会向浏览器返回404。  
添加一个动态路由，新建hello.py文件：
	from flask import Flask
	app = Flask(__name__)
	
	@app.route('/')
	def index():
		return '<h1>Hello World!</h1>'
	
	@app.route('/user/<name>')
	def user(name):
		return '<h1>Hello, %s!</h1>' % name
	
	if __name__ == '__main__':
	    app.run(debug=True)
访问http://127.0.0.1:5000/user/Dave。程序会显示一个使用name动态参数生成的欢迎消息。
## 2.5 请求-响应循环
介绍Flask框架的设计理念
### 2.5.1 程序和请求上下文
为了避免大量可有可无的参数把视图弄得一团糟，Flask使用上下文临时把某些对象变为全局可访问。有了上下文，就可以写出下面的视图函数：

	from flask import Flask
	
	@app.route('/')
	def index():
		user_agent = request.headers.get('User_Agent')
		return '<p>Your browser is %s</p>' % user_agent
在这个视图函数中我们如何把request当作全局变量。事实上，request不可能是全局变量。试想，在多线程服务器中，多个线程同时处理不同客户端发送的不同请求时，每个线程看到的request对象必然不同。Flask使用上下文让特定的变量在一个线程中全局可访问，与此同时却不会干扰其他线程。    
**注意：  
线程是可单独管理的最小指令集。进程经常使用多个活动线程，有时还会共享内存或文件句柄等资源。多线程Web服务器会创建一个线程池，再从线程池中选择一个线程用于处理接收到的请求。**  
Flask有两种上下文：程序上下文和请求上下文。表2-1列出了这两种上下文提供的变量。
![](http://i.imgur.com/HE9Ydvc.png)
下面这个Python shell会话演示了程序上下文的使用方法：

	>>> from hello import app
	>>> from flask import current_app
	>>> current_app.name
	...
	>>> app_ctx = app.app_context()
	>>> app_ctx.push()
	>>> current_app.name
	'hello'
	>>> app_ctx.pop()
在这个例子中，没激活程序上下文之前就调用current_app.name会导致错误，但推送完上下文之后就可以调用了。注意，在程序实例上调用app.app_context()可获得一个程序上下文。
### 2.5.2 请求调度
程序收到客户端发来的请求时，要找到处理该请求的视图函数。Flask会在程序的URL映射中查找请求的URL。URL映射是URL和视图函数之间的对应关系。  
要想查看Flask程序中URL映射是什么样子，我们可以在Python shell中检查hello.py生产的映射。测试之前，确保激活了虚拟环境：  

	(venv_ $ python
	>>> from hello import app
	>>> app.url_map
/和/user/<name>路由在程序中使用app.route修饰器定义。/static/<filename>路由是Flask添加的特殊路由，用于访问静态文件。  
URL映射中的HEAD和Options、GET是请求方法，由路由进行处理。Flask为每个路由都指定了请求方法，这样不同的请求方法发送到相同的URL上时，会使用不同的视图函数进行处理。HEAD和OPTIONS方法由Flask自动处理，因此可以这么说，在这个程序中，URL映射中的3个路由都使用GET方法。
