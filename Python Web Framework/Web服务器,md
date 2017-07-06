# 1.WSGI接口
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
