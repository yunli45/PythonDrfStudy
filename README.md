# PythonDrfStudy
学习DRF，先自学后边结合微信微信小程序使用

[python | DRF 框架知识总览 - MR_黄Python之路 - 博客园 (cnblogs.com)](https://www.cnblogs.com/huangjiangyong/p/14088982.html)

[Django的CBV与FBV - Yuan先生 - 博客园 (cnblogs.com)](https://www.cnblogs.com/yuanchenqi/articles/8715364.html)

还有一个完整的教程：

[Django: 路由与视图 / 简介 - 汇智网 (hubwiz.com)](http://cw.hubwiz.com/card/c/562efe441bc20c980538e801/1/1/1/) 

## 简述

​		Django 有两种开发模式，分别为 FBV 何 CBV,**FBV（function base views）** 就是在视图里使用函数处理请求。**CBV（class base views）** 就是在视图里使用类处理请求。

​		Python是一个面向对象的编程语言，如果只用函数来开发，有很多面向对象的优点就错失了（继承、封装、多态）。所以Django在后来加入了Class-Based-View。可以让我们用类写View。这样做的优点主要下面两种：

1. 提高了代码的复用性，可以使用面向对象的技术，比如Mixin（多继承）
2. 可以用不同的函数针对不同的HTTP方法处理，而不是通过很多if判断，提高代码可读性

## 环境搭建

​	 

1. 语言环境： **Anaconda  4.10.3**  、**python 3.7**

2. 编程IDE：**Pycharm 2021**

3. 数据库：mysql 8.01

4. 需要安装的库：

   |             包名             | 版本号 |                    作用                     |
   | :--------------------------: | :----: | :-----------------------------------------: |
   |            Django            | 3.0.7  |               创建项后端项目                |
   | djangorestframework （框架） | 3.11.0 | 前后端分离，快速编写基于Django的RESTful API |
   |     django-rest-swagger      | 2.2.0  |            为drf生成api接口文档             |
   |            json5             | 0.9.5  |              用于返回json数据               |
   |            PyJWT             | 1.7.1  |                   JWT认证                   |
   |     django-pyodbc-azure      |        |   链接MysqlServer 数据库（本次没有使用）    |
   |           pymysql            | 0.9.3  |               链接mysql数据库               |

 至于使用 django-pyodbc-azure   链接 MysqlServer  可以参考我的另一篇文章： [Django3.0 中使用django-pyodbc-azure 和 pyodbc 链接sql server 2008数据库，操作已有的数据库 和表](https://blog.csdn.net/dadaowuque/article/details/105375110)



## CBV  FBV   Mixin 的使用

### 1、FBV

如果我们要写一个处理GET方法的view，用函数写的话是下面这样。

```python
from django.http import HttpResponse
  
def my_view(request):
     if request.method == 'GET':
            return HttpResponse('OK')
```

### 2、CBV

​	在Django中，实现视图的基类是*View*，我们在应用开发中，通常应该基于*View* 来创建自己的视图类：

如果用class-based view写的话，就是下面这样



```PYTHON
from django.http import HttpResponse
from django.views import View
  
class MyView(View):
      def get(self, request):
            return HttpResponse('OK')
```

​		Django的url是将一个请求分配给可调用的函数的，而不是一个class。针对这个问题，class-based view提供了一个 **as_view()** 静态方法（也就是类方法），调用这个方法，会创建一个类的实例，然后通过实例调用`dispatch()`方法，`dispatch()`方法会根据request的method的不同调用相应的方法来处理request（如`get() `,` post()`等）。到这里，这些方法和function-based view差不多了，要接收request，得到一个response返回。如果方法没有定义，会抛出HttpResponseNotAllowed异常。



基类*View*封装了类方法*as_view()*来获得一个类的实例，因此我们的路由表 应当这样写：

在url中，就这么写：

```PYTHON
# urls.py
from django.conf.urls import url
from myapp.views import MyView
  
urlpatterns = [
     url(r'^index/$', MyView.as_view()),
]
```

类的属性可以通过两种方法设置，第一种是常见的Python的方法，可以被子类覆盖。

[![复制代码](README.assets/copycode.gif)](javascript:void(0);)

```python
from django.http import HttpResponse
from django.views import View
  
class GreetingView(View):
    name = "yuan"
    def get(self, request):
         return HttpResponse(self.name)
  
# You can override that in a subclass
  
class MorningGreetingView(GreetingView):
    name= "alex"
```

[![复制代码](README.assets/copycode.gif)](javascript:void(0);)

第二种方法，你也可以在url中指定类的属性：

在url中设置类的属性Python

```python
urlpatterns = [
   url(r'^index/$', GreetingView.as_view(name="egon")),
]
```

### 3、使用Mixin 

​		cbv的实现原理通过看django的源码就很容易明白，大体就是由url路由到这个cbv之后，通过cbv内部的dispatch方法进行分发，将get请求分发给cbv.get方法处理，将post请求分发给cbv.post方法处理，其他方法类似。怎么利用多态呢？cbv里引入了mixin的概念。Mixin就是写好了的一些基础类，然后通过不同的Mixin组合成为最终想要的类。

​		所以，理解cbv的基础是，理解Mixin。Django中使用Mixin来重用代码，一个View Class可以继承多个Mixin，但是只能继承一个View（包括View的子类），推荐把View写在最右边，多个Mixin写在左边。

​		除了*继承*之外，*Mixin*是Django视图类中实现代码级复用的另一种主要模式。

#### 3.1 什么是Mixin？

根据[WIKI](https://en.wikipedia.org/wiki/Mixin) 的定义，Mixin是一种编程理念，用来将代码*注入*到一个*类*里，使被注入的类具备Mixin所 实现的功能。当需要在多个类之间复用代码时，将这部分代码抽出来定义一个新的类，就构成 一个*Mixin*。Mixin通常作为父类，使继承类具备了它实现的功能。

Got it？

三个要点：

1. 引入Mixin概念的目的，是实现类之间的代码复用（功能复用）
2. 一个Mixin就是一个语法上的类，但是这个类本身不一定需要有明确的语义，它仅仅是可复 用代码（功能特性）的堆积。Mixin存在的目的*仅仅*是为了以可复用的方式*充实*那些真 正具有语义的类的功能
3. 其他类通过继承Mixin类，来获得Minxi实现的功能
