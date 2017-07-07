# 实战演练1.WSGI接口
WSGI是将Python服务器端程序连接到Web服务器的通用协议。  
WSGI的全程为Web Server Gateway Interface,也可称作Python Web Server Gateway Interface。  
接口有两个：一个是与Web服务器(Nginx/Apache...)的接口，另一个是与服务器端程序的接口。WSGI Server与Web服务器的接口包括uwsgi、fast、cgi等，服务器端程序的开发者需要关注WSGI与服务器程序的接口。  
WSGI的服务器程序接口非常简单，以下为一个例子，该文件保存为webapp.py:  
	
	def application(env, start_response):
	    start_response('200 OK', [('Content-Type','text/html')])
	    return [b"Hello World"]
该代码只定义了一个函数application，所有来自Web服务器的HTTP请求都会由WSGI服务器转换为对该函数的调用。与该服务器端程序相对应的是下面的WSGI Servier程序：  
	
	# 引入Python的WSGI包
	from wsgiref.simple_server import make_server
	# 引入服务器端程序端的代码
	from webapp import application
	
	#实例化一个监听 8080端口的服务器
	server = make_server('', 8080 ,application)
	# 开始监听HTTP请求：
	server.serve_forever()
将该WSGI Server的程序保存为wsgi_server.py,通过下面的命令即可启动一个Web服务器，该服务器对所有请求都返回Hell World页面：  

	#python wsgi_server.py
**注意：** 虽然WSGI的设计目标是连接标准的Web服务器(Nginx、Apache等)与服务器端程序，但是WSGI Server本身也可以作为Web服务器运行。由于性能方面的原因，该服务器一般只做测试使用，不能用于正式运行。 
# 实战演练2.Linux+Nginx+uWSGI配置  
Nginx是由俄罗斯工程师开发的一个高性能HTTP和反向代码服务器。其以运行稳定、配置简单、资源消耗低而闻名。  
Nginx是Python在Linux环境下的首选Web服务器之一。
## 1.安装Nginx
在Ubuntu Linux中安装：  

	#sudo apt-get install nginx
启动Nginx服务器：  

	# sudo service nginx start
停止Nginx服务器：  
	
	# sudo service nginx stop	
查看Nginx服务器：的状态：
	
	# sudo service nginx status
重启Nginx服务器： 
	
	# sudo service nginx restart
## 2.Nginx配置文件
Nginx安装完成后默认启动方式，在开发调试的过程中可能需要调整Nginx的运行参数，这些运行参数通过全局配置文件(nginx.conf)和站点配置文件(sites-enabled/*)进行设置。  
每个Nginx服务器中可以运行多个Web站点，每个站点的配置通过站点配置文件设置。每个站点应该以一个单独的配置文件存放在/etc/nginx/sites-enabled目录中，默认站点配置文件名为/etc/nginx/sites-enabled/default。
## 3.安装uWSGI及配置
uWSGI是WSGI在Linux中的一种实现，这样开发者就无须自己编写WSGI Server了。  
安装uWSGI：

	#pip install uwsgi
安装完成后即可运行uwsgi命令启动WSGI服务器，uwsgi命令通过启动参数的方式配置可选的运行方式。比如，如下命令可以运行uWSGI，用于加载之前编写的服务器端程序webapp.py：  
	
	#uwsgi --http：9090 --wsgi-file webapp.py
注意：--http当您使用前端网络服务器或您正在做某种形式的基准测试时，请勿使用--http-socket。继续阅读quickstart了解原因。  
**由于一直报错，所以进行不下去了。真的烦！**




uWSGI Github:   https://github.com/unbit/uwsgi  
uWSGI 文档：    https://uwsgi-docs.readthedocs.io/en/latest/index.html
uWSGI 中文文档：http://uwsgi-docs-zh.readthedocs.io/zh_CN/latest/index.html
