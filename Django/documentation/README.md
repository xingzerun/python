#注意事项
![](http://i.imgur.com/6NS53gr.png)
![](http://i.imgur.com/XpZxLaD.png)
对应链接：https://docs.djangoproject.com/en/1.11/topics/install/

Django：  
文档主页：https://docs.djangoproject.com/en/1.11/  
文档内容：https://docs.djangoproject.com/en/1.11/contents/  
下面为文档内容中的链接，按目录由上到下  
入门项目：https://docs.djangoproject.com/en/1.11/intro/  
概述：https://docs.djangoproject.com/en/1.11/intro/overview/









## 安装Django

	# pip install django
验证一下:

	#python
	>>>import django
	>>>django.VERSION
	>>>'版本信息'
## 建立项目
在进行Django开发之前需要先用django-admin建立Django项目：

	#django-admin startproject djangosite(站点名称）
第一次建立Django项目是在Ubuntu上，使用的是Python2的环境。出现了如下错误：
![](http://i.imgur.com/XD7lT8s.png)
然后听话的执行提示的命令后，又出现：
![](http://i.imgur.com/4WRJPOW.png)
解决方法：

	sudo apt-get install python-django
再次建立项目：
![](http://i.imgur.com/dXBVYYW.png)
