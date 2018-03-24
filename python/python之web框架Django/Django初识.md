# MTV模型

Django的MTV分别代表：

       Model(模型)：负责业务对象与数据库的对象(ORM)

       Template(模版)：负责如何把页面展示给用户

       View(视图)：负责业务逻辑，并在适当的时候调用Model和Template

       此外，Django还有一个urls分发器，它的作用是将一个个URL的页面请求
       分发给不同的view处理，view再调用相应的Model和Template
       
# Django基本命令

## 1、下载Django：

```
pip3 install django
```

## 2、创建一个django project

```python 
django-admin.py startproject mysite
```

当前目录下会生成mysite的工程，目录结构如下：

```
mysite
├── manage.py
└── mysite
     ├── __init__.py
     ├── settings.py
     ├── urls.py
     └── wsgi.py
```

manage.py ----- Django项目里面的工具，通过它可以调用django shell和数据库等。

settings.py ---- 包含了项目的默认设置，包括数据库信息，调试标志
以及其他一些工作的变量。

urls.py ----- 负责把URL模式映射到应用程序。

## 3、在mysite目录下创建应用

```
python manage.py startapp blog
```

## 4、启动django项目

```
python manage.py runserver 8080
```

## 5、同步更改数据库表或字段

```
    python manage.py syncdb
     
    注意：Django 1.7.1 及以上的版本需要用以下命令
    python manage.py makemigrations
    python manage.py migrate
    
```

这种方法可以创建表，当你在models.py中新增了类时，运行它就可以自动在数据库中创建表了，不用手动创建。

## 6、清空数据库

```
python manage.py flush
```
 此命令会询问是 yes 还是 no, 选择 yes 会把数据全部清空掉，只留下空表。
 
##  7、创建超级管理员

```
    python manage.py createsuperuser
     
    # 按照提示输入用户名和对应的密码就好了邮箱可以留空，用户名和密码必填
     
    # 修改 用户密码可以用：
    python manage.py changepassword username
```

## 8、Django 项目环境终端

```
python manage.py shell
```

这个命令和 直接运行 python 进入 shell 的区别是：你可以在这个 shell
里面调用当前项目的 models.py 中的 API，对于操作数据的测试非常方便。

## 9、Django 项目环境终端
```
ython manage.py dbshell
```
Django 会自动进入在settings.py中设置的数据库，如果是 MySQL 或 postgreSQL,会要求输入数据库用户密码。

在这个终端可以执行数据库的SQL语句。如果您对SQL比较熟悉，可能喜欢这种方式。

## 10、更多命令

```
python manage.py
```
 查看所有的命令，忘记子名称的时候特别有用。

## 11 静态文件配置
```
概述：

     静态文件交由Web服务器处理，Django本身不处理静态文件。
     简单的处理逻辑如下(以nginx为例)：

              URI请求-----> 按照Web服务器里面的配置规则先处理，以nginx为例，
              主要求配置在nginx.
                             conf里的location

                         |---------->如果是静态文件，则由nginx直接处理

                         |---------->如果不是则交由Django处理，
                         Django根据urls.py里面的规则进行匹配

    以上是部署到Web服务器后的处理方式，为了便于开发，
    Django提供了在开发环境的对静态文件的处理机制，方法是这样：
```

static配置：

STATIC主要指的是如css,js,images这样文件：

```
STATIC_URL = '/static/'      # 别名
STATICFILES_DIRS = (
            os.path.join(BASE_DIR,"static"),  #实际名 ,即实际文件夹的名字
        )

'''

注意点1:
 django对引用名和实际名进行映射,引用时,只能按照引用名来,不能按实际名去找
        <script src="/statics/jquery-3.1.1.js"></script>
        ------error－－－－－不能直接用，必须用STATIC_URL = '/static/':
        <script src="/static/jquery-3.1.1.js"></script>

注意点2:
 STATICFILES_DIRS = (
    ("app01",os.path.join(BASE_DIR, "app01/statics")),
        )

 <script src="/static/app01/jquery.js"></script>
```

# 视图层之路由配置系统(views)

URL配置(URLconf)就像Django 所支撑网站的目录。它的本质是URL与要为该URL调用的视图函数之间的映射表；
你就是以这种方式告诉Django，对于这个URL调用这段代码，对于那个URL调用
那段代码。


```
    
    urlpatterns = [
         url(正则表达式, views视图函数，参数，别名),
]


参数说明：

    一个正则表达式字符串
    一个可调用对象，通常为一个视图函数或一个指定视图函数路径的字符串
    可选的要传递给视图函数的默认参数（字典形式）
    一个可选的name参数

```

## 1 URLconf的正则字符串参数

### 1.1 简单配置
```
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^articles/2003/$', views.special_case_2003),
    url(r'^articles/([0-9]{4})/$', views.year_archive),
    url(r'^articles/([0-9]{4})/([0-9]{2})/$', views.month_archive),
    url(r'^articles/([0-9]{4})/([0-9]{2})/([0-9]+)/$', views.article_detail),
]
```
```
    NOTE:
    1 一旦匹配成功则不再继续
    2 若要从URL 中捕获一个值，只需要在它周围放置一对圆括号。
    3 不需要添加一个前导的反斜杠，因为每个URL 都有。例如，应该是^articles 而不是 ^/articles。
    4 每个正则表达式前面的'r' 是可选的但是建议加上。

一些请求的例子：

    /articles/2005/3/ 不匹配任何URL 模式，因为列表中的第三个模式要求月份应该是两个数字。
    /articles/2003/ 将匹配列表中的第一个模式不是第二个，因为模式按顺序匹配，第一个会首先测试是否匹配。
    /articles/2005/03/ 请求将匹配列表中的第三个模式。Django 将调用函数
                       views.month_archive(request, '2005', '03')。
```
```
#设置项是否开启URL访问地址后面不为/跳转至带有/的路径
APPEND_SLASH=True
```

### 1.2 有名分组(named group)

上面的示例使用简单的、没有命名的正则表达式组（通过圆括号）来捕获URL
中的值并以位置 参数传递给视图。在更高级的用法中，可以使用命名的
正则表达式组来捕获URL 中的值并以关键字 参数传递给视图。

在Python 正则表达式中，命名正则表达式组的语法是(?P<name>pattern)，
其中name 是组的名称，pattern 是要匹配的模式。

下面是以上URLconf 使用命名组的重写：

```
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^articles/2003/$', views.special_case_2003),
    url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
    url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/$', views.month_archive),
    url(r'^articles/(?P<year>[0-9]{4})/(?P<month>[0-9]{2})/(?P<day>[0-9]{2})/$', views.article_detail),
]
```
这个实现与前面的示例完全相同，只有一个细微的差别：捕获的值作为关键字
参数而不是位置参数传递给视图函数。例如：
```
    /articles/2005/03/    
    请求将调用views.month_archive(request, year='2005', month='03')函数
    /articles/2003/03/03/ 
    请求将调用函数views.article_detail(request, year='2003', month='03', day='03')。
```

在实际应用中，这意味你的URLconf 会更加明晰且不容易产生参数顺序问题的
错误 —— 你可以在你的视图函数定义中重新安排参数的顺序。当然，这些好处
是以简洁为代价；有些开发人员认为命名组语法丑陋而繁琐。

### 1.3 URLconf 在什么上查找

URLconf 在请求的URL 上查找，将它当做一个普通的Python 字符串。不包括GET和POST参数以及域名。

例如，http://www.example.com/myapp/ 请求中，URLconf 将查找myapp/。

在http://www.example.com/myapp/?page=3 请求中，URLconf 仍将查找myapp/。

URLconf 不检查请求的方法。换句话讲，所有的请求方法 —— 同一个URL的POST、GET、HEAD等等 —— 都将路由到相同的函数。

### 1.4 捕获的参数永远是字符串

每个捕获的参数都作为一个普通的Python 字符串传递给视图，无论正则表达式使用的是什么匹配方式。例如，
下面这行URLconf 中：

```
url(r'^articles/(?P<year>[0-9]{4})/$', views.year_archive),
views.year_archive() 的year 参数将是一个字符串
```
### 1.5 指定视图参数的默认值

有一个方便的小技巧是指定视图参数的默认值。 下面是一个URLconf 和视图的示例：

```
# URLconf
from django.conf.urls import url

from . import views

urlpatterns = [
    url(r'^blog/$', views.page),
    url(r'^blog/page(?P<num>[0-9]+)/$', views.page),
]

# View (in blog/views.py)
def page(request, num="1"):

```
在上面的例子中，两个URL模式指向同一个视图views.page —— 但是第一个模式不会从URL 中捕获任何值。如果第一个模式匹配，page() 函数将使用num参数的默认值"1"。如果第二个模式匹配，page() 将使用正则表达式捕获的num 值。

### 1.6 Including other URLconfs

```
#At any point, your urlpatterns can “include” other URLconf modules. This
#essentially “roots” a set of URLs below other ones.

#For example, here’s an excerpt of the URLconf for the Django website itself.
#It includes a number of other URLconfs:


from django.conf.urls import include, url

urlpatterns = [
   url(r'^admin/', admin.site.urls),
   url(r'^blog/', include('blog.urls')),
]
```
## 2 传递额外的选项给视图函数(了解)

URLconfs 具有一个钩子，让你传递一个Python 字典作为额外的参数传递给视图函数。

django.conf.urls.url() 函数可以接收一个可选的第三个参数，它是一个字典，表示想要传递给视图函数的额外关键字参数。

例如：
```
from django.conf.urls import url
from . import views

urlpatterns = [
    url(r'^blog/(?P<year>[0-9]{4})/$', views.year_archive, {'foo': 'bar'}),
]
```
在这个例子中，对于/blog/2005/请求，Django 将调用views.year_archive(request, year='2005', foo='bar')。

这个技术在Syndication 框架中使用，来传递元数据和选项给视图。

## 3 URL 的反向解析

在使用Django 项目时，一个常见的需求是获得URL 的最终形式，以用于嵌入到生成的内容中（视图中和显示给用户的URL等）或者用于处理服务器端的导航（重定向等）。

人们强烈希望不要硬编码这些URL（费力、不可扩展且容易产生错误）或者设计一种与URLconf 毫不相关的专门的URL 生成机制，因为这样容易导致一定程度上产生过期的URL。

换句话讲，需要的是一个DRY 机制。除了其它有点，它还允许设计的URL 可以自动更新而不用遍历项目的源代码来搜索并替换过期的URL。

获取一个URL 最开始想到的信息是处理它视图的标识（例如名字），查找正确的URL 的其它必要的信息有视图参数的类型（位置参数、关键字参数）和值。

Django 提供一个办法是让URL 映射是URL 设计唯一的地方。你填充你的URLconf，然后可以双向使用它：

根据用户/浏览器发起的URL 请求，它调用正确的Django 视图，并从URL 中提取它的参数需要的值。
根据Django 视图的标识和将要传递给它的参数的值，获取与之关联的URL。
第一种方式是我们在前面的章节中一直讨论的用法。第二种方式叫做反向解析URL、反向URL 匹配、反向URL 查询或者简单的URL 反查。

在需要URL 的地方，对于不同层级，Django 提供不同的工具用于URL 反查：

在模板中：使用url 模板标签。
在Python 代码中：使用django.core.urlresolvers.reverse() 函数。
在更高层的与处理Django 模型实例相关的代码中：使用get_absolute_url() 方法。
例子：

考虑下面的URLconf：

```
from django.conf.urls import url

from . import views

urlpatterns = [
    #...
    url(r'^articles/([0-9]{4})/$', views.year_archive, name='news-year-archive'),
    #...
]
```
根据这里的设计，某一年nnnn对应的归档的URL是/articles/nnnn/。

你可以在模板的代码中使用下面的方法获得它们：

```
<a href="{% url 'news-year-archive' 2012 %}">2012 Archive</a>

<ul>
{% for yearvar in year_list %}
<li><a href="{% url 'news-year-archive' yearvar %}">{{ yearvar }} Archive</a></li>
{% endfor %}
</ul>
```
在Python 代码中，这样使用：

```
from django.core.urlresolvers import reverse
from django.http import HttpResponseRedirect

def redirect_to_year(request):
    # ...
    year = 2006
    # ...
    return HttpResponseRedirect(reverse('news-year-archive', args=(year,)))
```
如果出于某种原因决定按年归档文章发布的URL应该调整一下，那么你将只需要
修改URLconf 中的内容。

在某些场景中，一个视图是通用的，所以在URL 和视图之间存在多对一的关系。对于这些情况，当反查URL 时，只有视图的名字还不够。

## 4 名称空间（Namespace）

命名空间（英语：Namespace）是表示标识符的可见范围。一个标识符可在多
个命名空间中定义，它在不同命名空间中的含义是互不相干的。这样，在一
个新的命名空间中可定义任何标识符，它们不会与任何已有的标识符发生冲突，
因为已有的定义都处于其它命名空间中。

由于name没有作用域，Django在反解URL时，会在项目全局顺序搜索，当查找
到第一个name指定URL时，立即返回
我们在开发项目时，会经常使用name属性反解出URL，当不小心在不同的app的
urls中定义相同的name时，可能会导致URL反解错误，为了避免这种事情发生，
引入了命名空间。

project.urls:
```
urlpatterns = [
    url(r'^admin/', admin.site.urls),
    url(r'^app01/', include("app01.urls",namespace="app01")),
    url(r'^app02/', include("app02.urls",namespace="app02")),
]
```
app01.urls:
```
urlpatterns = [
    url(r'^index/', index,name="index"),
]
```
app02.urls:
```
urlpatterns = [
    url(r'^index/', index,name="index"),
]
```
app01.views 
```
from django.core.urlresolvers import reverse

def index(request):

    return  HttpResponse(reverse("app01:index"))
```
app02.views
```python
from django.core.urlresolvers import reverse

def index(request):

    return  HttpResponse(reverse("app02:index"))
```