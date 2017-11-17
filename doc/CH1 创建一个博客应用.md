#CH1 创建一个博客应用

你将通过这本书学习如何创建用于生产的完整Django项目。如果你还没有安装django，那么你将在本章第一节学习如何安装。这章包括如何使用Django创建一个简单的博客应用。本章的目的在于介绍框架如何工作，不同部分如何相互作用及如何通过基本功能创建django项目。你将在不过分关注细节的情况下创建一个完整的项目。框架各个部分的细节将在后续章节进行介绍。

本章涉及以下内容：

- 安装Django并创建第一个项目

- 设计并迁移模型

- 为模型创建一个administration网站

- 使用Queryset和manage工作

- 创建视图、模板和URLs

- 为list视图添加Pagination

- 使用Django基本类视图

##安装Django

如果你已经安装了Django，那么可以跳过这几节到创建第一个项目一节。作为Python库Django可用于任何python环境。如果你还没有安装django，下面是安装Django的快速向导。

Django可用于Python2.7和Python3。本书的例子使用Python3。如果你使用LInux或Mac OS X，python很可能已经预先安装好了。可以通过在teminal中输入python来确定是否安装了python。如果你看到下面的输出，那么python已经安装好了。

```
“Python 3.5.0 (v3.5.0:374f501f4567, Sep 12 2015, 11:00:19) 
[GCC 4.2.1 (Apple Inc. build 5666) (dot 3)] on darwin
Type "help", "copyright", "credits" or "license" for more information.
>>>”

摘录来自: “Django By Example”。 iBooks. 
```

如果你安装的python版本低于3.0，或者没有安装python，可以从http://www.python.org/download/ 下载Python3.5并进行安装。

由于使用Python3.0，你不需要安装数据库，python内置SQLite数据库，SQLite是一个可用于Django开发的轻量级的数据库，如果希望在生产环境部署应用，那么应该使用PostGreSQL、MySQL或者Oracle。可以从“https://docs.djangoproject.com/en/1.8/topics/install/#database-installation.”学习使用这些数据库。

## 创建第一个项目

我们第一个项目将完成一个博客网站。Django提供了创建项目文件结构的命令，在shell中运行以下命令：

```python
django-admin startproject mysite	
```

这个命令将创建一个名为mysite的Django项目。让我们来看一下项目的结构：

![structure](/Users/apple/dev/Django/Django_by_example/figures/structure.png)

这些文件的作用是：

- manage.py：与项目进行交互的命令行工具。它是django-admin.py工具的轻量级封装，我们不需要编辑这个文件。
- mysite/：项目目录包含以下文件：
  - __init__.py: 告诉python将mysite文件夹当做一个模块进行处理的空白文件；
  - settings.py：设置及配置项目。包括初始化缺省设置；
  - urls.py：保存URL patterns的地方。这里的每个url都与一个视图相匹配。
    - wigs.py: 以WSGI应用运行项目的配置。

settings.py文件默认使用SQLite数据库及Django中间件。我们需要为项目创建数据库表。

打开shell，跳转到项目的根目录并运行以下命令：

```python
cd mysite
python manage.py migrate
```

将会输出以下内容：

```python
Operations to perform:
  Apply all migrations: admin, auth, contenttypes, sessions
Running migrations:
  Applying contenttypes.0001_initial... OK
  Applying auth.0001_initial... OK
  Applying admin.0001_initial... OK
  Applying admin.0002_logentry_remove_auto_add... OK
  Applying contenttypes.0002_remove_content_type_name... OK
  Applying auth.0002_alter_permission_name_max_length... OK
  Applying auth.0003_alter_user_email_max_length... OK
  Applying auth.0004_alter_user_username_opts... OK
  Applying auth.0005_alter_user_last_login_null... OK
  Applying auth.0006_require_contenttypes_0002... OK
  Applying auth.0007_alter_validators_add_error_messages... OK
  Applying auth.0008_alter_user_username_max_length... OK
  Applying sessions.0001_initial... OK
```

数据库中添加了项目初始化需要的数据库表，你可以对migrate命令进行一些了解。

### 运行开发服务器

Django内置一个轻量级的web服务器来运行代码，从而节约配置生产服务器的时间。Django服务器运行期间会自动监测代码的变化并自动重载代码。然后它无法识别向项目中添加文件的操作，这种情况下需要手动重启服务器。

打开shell并跳转到项目的根目录，运行以下命令来启动开发服务器：

```python
python manage.py runserver
```

你见看到以下输出：

```python
Performing system checks...

System check identified no issues (0 silenced).
October 24, 2017 - 04:48:13
Django version 1.11.6, using settings 'mysite.settings'
Starting development server at http://127.0.0.1:8000/
Quit the server with CONTROL-C.
```

现在在浏览器中打开URL：http://127.0.0.1:8000/，你将看到以下页面：

![first_start](/Users/apple/dev/Django/Django_by_example/figures/first_start.png)

可以引导django开发服务器在自定义主机和端口运行，或者告诉它载入其他的setting文件，例如，可以这样运行manage.py命令：

```python
python manage.py runserver 127.0.0.1:8001 --settings=mysite.settings
```

这样可以手动适用需要多个配置的多个环境。

需要牢记的是，开发服务器只适用于开发，但是并不适合生产。为了部署生产环境，应该使用Apache、Gunicorn、uWSGI等web服务器运行项目。使用不同web服务器部署django的详细文档连接为：https://docs.djangoproject.com/en/1.11/howto/deployment/wsgi/。

此外，可下载的第十三章可以为你的Django项目生成生产环境。

### 项目设置

我们打开setting.py 文件来看一下项目配置。Django在这个文件中进行了几项配置，但这只是可用配置的其中一部分，可以从以下连接查看所有可用配置及其默认值：https://docs.djangoproject.com/en/1.11/ref/settings/。

下面的设置需要格外注意：

- DEBUG：打开/关闭项目调试模式的布尔值。如果设置为True，当捕捉到异常时Django将显示详细的错误信息。项目部署到生产环境时，要将其设为False。不要在生产环境中打开DEBUG，否则将可能暴露项目的敏感信息。
- ALLOW_HOST: 打开DEBUG模式或者运行测试时不使用该设置。一旦将网站迁移到生产环境并且将DEBUG设置为False，则需要添加域名/主机到该设置来使用Django网站。
- INSTALLED_APPS:该设置告诉Django网站的哪些应用处于激活状态。默认情况下，Django包含以下应用：
  - django.contrib.admin:  administration网站；
  - django.contrib.auth: 权限框架；
  - django.contrib.contenttypes: 内容框架；
  - django.contrib.sessions: 会话框架；
  - django.contrib.messages:消息框架；
  - django.contrib.staticfiles: 静态文件管理框架；
- MIDDLEWARE_CLASSSES：需要执行的中间件tuple。
- ROOT_URLCONF ：项目定义的root URL  pattern。
- DATABASES：项目使用的所有数据库设置的字典。这里必须有一个default数据库。默认配置使用SQLite3数据库。
- LANGUAGE_CODE: 定义默认Django网站的默认语言代码。

不要担心不理解这些内容。在下面的章节将会熟悉这些设置。

### 项目和应用

本书将会不断的涉及项目和应用这两个术语。在Django中，项目是指具有一些设置的Django安装；应用是一组模型、视图、模板和URLs。应用与框架交互来提供一些特定功能，它可以在不同项目中重用。我们把项目当作网站，它可以包括几个应用，如博客、wiki、论坛等，这些应用也可以用在其它项目中。

### 创建一个应用

现在让我们来创建第一个Django应用。我们将从创建一个博客应用开始。在terminal中跳转到项目的根目录并运行以下命令：

```python
python manage.py startapp blog
```

这将创建应用的基本结构，看起来是这样的：

![first_app](/Users/apple/dev/Django/Django_by_example/figures/first_app.png)

这些文件包括：

- admin.py：该文件用于将模型注册到administration网站。

- migrations: 这个目录将包含应用的数据库迁移记录。它允许Django追踪模型变化并进行相应的数据库同步。

- models.py： 应用的数据库模型，所有的Django应用都要有一个models.py文件，但是这个文件可以是空的。

- tests.py：该文件用于添加引用的测试程序。

- views.py：该文件实现应用逻辑。每个view接收一个HTTP请求，对其进行处理并返回一个响应。

  ​

##设计blog数据模式

我们将从为blog定义数据模型开始。每个模型都是django.db.models.Model的子类，它的每个属性都表示数据库表的一个字段。Django将为models.py中的每个模型创建一个数据库表。当你创建一个模型后，Django将提供一个API以便于数据查询。

首先，我们将定义一个Post模型，将以下代码添加到blog应用的models.py文件中：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.contrib.auth.models import User
from django.db import models
from django.utils import timezone


# Create your models here.


class Post(models.Model):
    STATUS_CHOICES = (('draft', 'Draft'), ('published', 'Published'),)
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250, unique_for_date='publish')
    author = models.ForeignKey(User, related_name='blog_posts')
    body = models.TextField()
    publish = models.DateTimeField(default=timezone.now)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    status = models.CharField(max_length=10, choices=STATUS_CHOICES,
                              default='draft')

    class Meta:
        ordering = ('-publish',)

    def __str__(self):
        return self.title
```

这是blog文章的基本模型，我们来看下这个模型的字段：

- title：文章标题字段。这个字段是CharField，在SQL数据库中将转化为VARCHAR列。
- slug：用于URLs的字段，slug是只包含字母、数字、下划线、连字符的短标签。我们将使用slug字段为blog文章创建漂亮、SEO友好的URLs。我们为该字段添加了unique_for_date参数，这样我们可以使用date和slug为文章创建标签，Django不允许相同日期的多篇文章使用相同的slug。
- author：这个字段是一个外键（ForeignKey）。这个字段定义了一个多对一关系。我们告诉Django每篇文章有一个作者而一个作者可以写多篇文章。Django将在数据库中使用相关模型的id创建一个外键。在这里，我们使用Django权限系统的User模型。我们使用related_name属性为反向关系（从User到Post）指定了一个名称，后面我们将针对这一点学习更多知识。
- body：这个字段是文章的正文。这个字段是TextField，在SQL数据库中将转化为Text列。
- publish：表示文章发表日期，我们使用Django的timezone now方法作为默认值。该方法表示时区敏感的datetime.now。
- create：表示文章创建日期，由于这里我们使用auto_now_add，该字段将在创建对象时自动添加。
- updated：表示文章最后一次更新日期，由于这里使用auto_now，当我们保存对象时将会自动更新该字段。
- status：表示文章状态的字段。我们使用choices参数，因此，该字段只能被设置为给定的choices中的一个。

我们可以看到，Django可以使用不同类型的字段来对模型进行定义。可以从以下链接找到所有字段：https://docs.djangoproject.com/en/1.11/ref/models/fields/。

模型内部的Meta类包含metadata。通过它告诉Django当查询数据库按照publish降序的顺序对查询结果进行排序。这里通过-前缀表示降序。

`__str__`方法是对象默认的表示方法。Django将在很多地方用到它，比如administration网站。

> 注意：
>
> Python3默认所有字符串都是unicode编码，因此我们使用`__str__`方法，如果你使用Pyhton2.X，使用`__unicode__`方法代替`__str__`方法。

由于我们将处理时间，我们将安装pytz模块。这个模块为python提供timezone定义而且SQLite需要用它来处理时间。打开shell并使用以下命令安装pytz：

```python
pip install pytz
```

Django支持时区敏感的datetimes。你可以通过在项目的settings.py中设置USE_TZ的值来激活/关闭时区。当使用startproject创建一个项目时该设置将被设为True。

### 激活你的应用

为了让Django追踪我们的应用并为应用的模型创建数据库表，我们需要激活该应用。我们通过编辑项目的settings.py文件在INSTALLED_APPS中添加blog来激活应用。看起来是这样的：

```python
INSTALLED_APPS = [
    'django.contrib.admin',
    'django.contrib.auth',
    'django.contrib.contenttypes',
    'django.contrib.sessions',
    'django.contrib.messages',
    'django.contrib.staticfiles',
    'blog',
]
```

现在Django知道我们应用已经激活而且可以引入它的模型了。

###创建并实现migrations

我们在数据库中为模型创建一个数据表。Django内置迁移系统来追踪模型的变化并该这些更新同步到数据库中。migrate将为INSTALL_APPS中所有应用实现迁移，它对当前模型及数据库进行同步。

首先，我们需要为我们刚刚创建的新波形创建migration，从项目的根目录输入以下命令：

```python 
python manage.py makemigrations blog 
```

你将看到以下输出：

```python
Migrations for 'blog':
  blog/migrations/0001_initial.py
    - Create model Post
```

Django 刚刚在blog应用的migration目录下创建了一个名为0001_initial.py的文件。你可以打开文件查看以下migration。

让我们查看一下Django为我们的模型创建数据库表所要执行的SQL代码。sqlmigrate根据migration的名字返回SQL但并不执行。运行以下命令：

```python
 python manage.py sqlmigrate blog 0001
```

输出是这样的：

```SQL
BEGIN;
--
-- Create model Post
--
CREATE TABLE "blog_post" ("id" integer NOT NULL PRIMARY KEY AUTOINCREMENT, "title" varchar(250) NOT NULL, "slug" varchar(250) NOT NULL, "body" text NOT NULL, "publish" datetime NOT NULL, "created" datetime NOT NULL, "updated" datetime NOT NULL, "status" varchar(10) NOT NULL, "author_id" integer NOT NULL REFERENCES "auth_user" ("id"));
CREATE INDEX "blog_post_slug_b95473f2" ON "blog_post" ("slug");
CREATE INDEX "blog_post_author_id_dd7a8485" ON "blog_post" ("author_id");
COMMIT;
```

具体的输出会因使用的数据库稍有变化。上面是SQLite的输出。你可以看到，Django使用应用名称的小写形式和数据库表名称的小写形式组成数据库表名称(blog_post)，你也可以在Meta类中使用db_table属性指定数据库表名称。Django自动为每个模型创建一个主键字段，你也可以通过为模型中某个字段设置primary_key=True来设置主键。

我们来为模型同步数据库，在项目根目录运行以下命令：

```python
python manage.py migrate
```

将会看到以下输出：

```
Operations to perform:
  Apply all migrations: admin, auth, blog, contenttypes, sessions
Running migrations:
  Applying blog.0001_initial... OK
```

我们刚为INSTALL_APPS中列出的实现了migrations。migration之后模型与数据库一致。（打开数据库，将会发现名为blog_post的数据库表）。

如果需要为已经存在的模型添加、删除或修改字段，或者添加新模型，将需要使用makemigrations命令创建新的migration，migration将允许Django追踪模型变化，然后需要运行migrate命令来同步数据库。

## 为模型创建administration网站

现在，我们已经定义了Post模型，我们将创建一个简单的administration网站来管理blog文章。Django内置非常有用的模型编辑接口。该接口(Django  admin site)通过阅读模型metadata动态创建模型编辑接口，你还可以对模型的展示形式进行编辑。

由于Django自动将django.contrib.admin放到项目setting.py的INSTALLED_APPS中了，因此我们无需再次添加。

### 创建superuser

首先，我们需要创建一个管理admin网站的用户，跳转到项目根目录，运行以下命令：

```python
python manage.py createsuperuser
```

你将看到如下输出，根据提示逐步输入你的用户名、邮箱及密码：

```python
Username (leave blank to use 'apple'): admin
Email address: lijun@osscloud.net
Password: 
Password (again): 
Superuser created successfully.
```

### Django admin网站

现在，通过`python manage.py runserver`运行开发服务器，在浏览器中打开http://127.0.0.1:8000/admin，将会看到admin网站的登录页面，如下图所示：

![admin login](/Users/apple/dev/Django/Django_by_example/figures/admin login.png)

使用上一步骤创建的用户登录，你将看到admin网站的索引页面如下图所示：

![first_login](/Users/apple/dev/Django/Django_by_example/figures/first_login.png)

这里你看到的Group和User模型是Django权限框架(Django.contrib.auth)的模型。如果点击Users，你见看到刚刚创建的用户。blog应用的Post模型与User模型有一个关系。记住，它是一个有author字段定义的关系。

### 将模型添加到admin网站

让我们将blog模型添加到admin网站。向blog应用的admin.py文件写入以下代码：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.contrib import admin
from .models import Post
# Register your models here.

admin.site.register(Post)
```

现在在浏览器中重新载入admin网站，你应该在网站中看到如下图所示的Post模型：

![add_post_model](/Users/apple/dev/Django/Django_by_example/figures/add_post_model.png)

向Django admin网站注册一个模型后，可以得到一个接口，通过这个接口我们可以很容易的列出模型对象以及创建、修改、删除模型对象。

点击Posts右侧的Add连接可以添加新的文章。你可以看到Django为我们刚刚创建的模型自动生成的表单，如下图所示：

![edit_post](/Users/apple/dev/Django/Django_by_example/figures/edit_post.png)

Django为每种类型的字段使用不同的表单组件。即使DateTimeField这样的复杂字段也通过接口（如JavaScript date picker)进行展示。

填写表单并点击save按钮，你将被重定向到文章列表页面，这里你将看到刚刚成功添加的信息，如下图所示：

![add_post](/Users/apple/dev/Django/Django_by_example/figures/add_post.png)

### 自定义模型展示方式

现在，我们来看看如何自动以admin网站。编辑blog应用的admin.py文件，将其更改为：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.contrib import admin

from .models import Post


# Register your models here.
class PostAdmin(admin.ModelAdmin):
    list_display = ('title', 'slug', 'author', 'publish', 'status')



admin.site.register(Post, PostAdmin)
```

我们告诉Django admin网站我们使用继承ModelAdmin的自定义类实现模型注册。在这个勒种，我们可以包括模型在admin网站的展示及交互方式。这里的list_display属性允许我们设置admin对象列表页面展示的字段。

让我们使用如下代码设置更多自定义选项：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.contrib import admin

from .models import Post


# Register your models here.
class PostAdmin(admin.ModelAdmin):
    list_display = ('title', 'slug', 'author', 'publish', 'status')
    list_filter = ('status', 'created', 'publish', 'author')
    search_fields = ('title', 'body')
    prepopulated_fields = {'slug': ('title',)}
    raw_id_fields = ('author',)
    date_hierarchy = 'publish'
    ordering = ['status', 'publish']


admin.site.register(Post, PostAdmin)
```

回到浏览器重新载入文章列表页面，你将看到如下页面：



你将看到文章列表页面展示的字段是list_display设置的字段。现在列表页面包括一个右侧边栏来允许按照list_filters属性设置的字段对列表进行过滤。页面还多了一个搜索条，这是因为我们定义了search_field属性。搜索条下方是一个日期的面包屑导航菜单，这是有date_hierarchy属性定义的。我们还可以看到文章默认通过Status和Publish字段排序，我们可以使用ordering属性定义默认排序字段。

现在点击Add Post连接，也将看到一些变化。当输入新文章的标题时会自动生成slug。我们使用prepopulated_field告诉Django输入title时自动生成slug。而且，现在author字段用lookup组件展示，当用户数量很多时，这种操作从从下拉框中寻找更容易，如下图所示。

![raw_id_fields](/Users/apple/dev/Django/Django_by_example/figures/raw_id_fields.png)

点击图中的放大镜，将会出现搜索页面，如下图所示：

![raw_id_fields_lookup](/Users/apple/dev/Django/Django_by_example/figures/raw_id_fields_lookup.png)

我们已经使用很少的代码自定义了模型在admin网站的展示形式，后续章节我们会涉及更多自定义及扩张admin网站的方法。

## 使用Queryset和manager

现在我们有了一个可以管理blog内容的admin网站了，现在要学习如何从数据库中获取信息并与其进行交互。Django内置一个让我们很容易创建、获取、更新和删除对象的数据库API-Django Object-Relational-Mapper(ORM)，它可以与MySQL、PostGreSQL、SQLite及Oracle兼容。我们可以通过编辑项目settings.py中的DATABASES设置定义项目所要使用的数据库。Django可以同时与多个数据库工作。

一旦创建了数据模型，Django提供API与其进行交互，有关数据模型的官方文档连接为：https://docs.djangoproject.com/en/1.11/ref/models/。

### 创建对象

打开terminal在项目根目录运行以下命令:

```python
python manage.py shell
```

然后输入以下信息：

```python
In [1]: from django.contrib.auth.models import User

In [2]: from blog.models import Post

In [3]: user = User.objects.get(username='admin')

In [4]: post = Post(title='One more post',slug='one-more-post',body='Post body',author=user)

In [5]: post.save()

In [6]: Post.objects.create(title='another post',slug='another-post',body='Post body',author=user)
Out[6]: <Post: another post>

```

我们来分析下这些代码，首先，我们获取名为admin的用户：

```python

In [3]: user = User.objects.get(username='admin')
```

get()方法允许我们从数据库中获取单个对象。注意，这个方法期望查询结果为一个结果。如果数据库没有返回结果，这个方法将引发DoesNotExist异常，如果数据库返回的结果多于一个，这个方法将引发MultipleObjectsReturened异常。这两个异常都是模型类的属性。

然后，我们使用title，slug，body和author创建了一个Post对象，其中author的值为上一步获取的user：

```python
In [4]: post = Post(title='One more post',slug='one-more-post',body='Post body',author=user)
```

注意：这一步并没有影响数据库。

最后，我们将Post对象保存到数据库：

```python
In [5]: post.save()
```

这个操作实现了一个Insert SQL。我们已经看到了如何创建对象并保存到数据库。我们也可以使用create()方法在数据库中创建对象：

```python
In [6]: Post.objects.create(title='another post',slug='another-post',body='Post body',author=user)
Out[6]: <Post: another post>
```

现在再次查看admin网站的Post列表，我们会发现下面三条记录：

![queryset_1](/Users/apple/dev/Django/Django_by_example/figures/queryset_1.png)

### 更新对象

现在，更改post的标题：

```python

In [1]: from blog.models import Post

In [2]: post = Post.objects.get(pk=2)

In [3]: post
Out[3]: <Post: One more post>

In [4]: post.title='New title'

In [5]: post.save()

In [6]: post
Out[6]: <Post: New title>
```

首先，我们获取要更改的文章：

```python
In [2]: post = Post.objects.get(pk=2)
```

这里的pk是值post的主键，由于主键唯一，get()方法不会抛出异常。

然后，更改标题：

```python
In [4]: post.title='New title'
```

这时，也不会影响数据库。

最后保存到数据库：

```python
In [5]: post.save()
```

当然，也可以直接使用update()函数直接对数据库中的记录进行更改。

### 获取对象

Django ORM基于QuerySet，Queryset是数据库中的对象集合，我们可以使用几个filter限制Queryset。我们已经知道如何使用get()获取一个对象。每个Django模型至少有一个manager，默认的manager名为objects。我们使用模型的manager获取Queryset对象。我们可以使用objects的all()方法从一个数据库表中获取所有对象：

```python
In [9]: all_post = Post.objects.all()
```

注意，这个Queryset并没有被执行。Django的Queryset是惰性的。这个行为使得Queryset非常高效。如果我们不将QuerySet设置为一个变量，而是直接写到Python shell中，QuerySet将被执行，因为这样需要输出结果：

```python
In [10]: Post.objects.all()
Out[10]: <QuerySet [<Post: another post>, <Post: New title>, <Post: Meet Django>]>
```

### 使用filter()方法

我们可以使用filter()方法过滤QuerySet，比如我们可以获取所有2015年发布的文章：

```python
In [12]: Post.objects.filter(publish__year=2015)
Out[12]: <QuerySet []>

```

我们也可以使用多个字段进行过滤，比如，我们要获取用户admin在2015年发布的文章：

```python
In [13]: Post.objects.filter(publish__year=2015,author__username='admin')
Out[13]: <QuerySet []>
```

这与多级filter效果一样：

```python
In [14]: Post.objects.filter(publish__year=2015).filter(author__username='admin')
Out[14]: <QuerySet []>
```

注意，我们使用双下划线来获取字段变量，比如`publish__year`，还可以用双下划线获取字段对应的模型，比如`author__username`。

### 使用exclude()

我们可以使用manager的exclude()方法排除某些结果，比如，我们要查询2015年发布的标题不以why开头的对象：

```python
In [16]: Post.objects.filter(publish__year=2015).exclude(title__startswith='Why')
Out[16]: <QuerySet []>
```

### 使用order_by()

我们可以使用manager的order_by()函数根据不同字段进行排序，比如，通过标题排序：

```python
In [17]: Post.objects.order_by('title')
Out[17]: <QuerySet [<Post: Meet Django>, <Post: New title>, <Post: another post>]>
```

上述查询执行的顺序排序，倒序排序需要在字段前加上'-'，比如：

```python
In [18]: Post.objects.order_by('-title')
Out[18]: <QuerySet [<Post: another post>, <Post: New title>, <Post: Meet Django>]>
```

### 删除对象

如果要删除对象，我们可以这样实现：

```python
In [19]: post = Post.objects.get(pk=3)

In [20]: post.delete()
Out[20]: (1, {u'blog.Post': 1})
```

### 何时执行QuerySet

我们可以对QuerySet进行任何filter，在QuerySet被执行之前并不会影响数据库。只有下面的情况会触发QuerySet执行：

- 对其进行第一次迭代时
- 对其进行切片时，比如Post.objects.all()[:3]
- 对其进行pickle或者缓存
- 对其使用repr()或者len()函数
- 对其使用list()或values()函数
- 使用bool()、or、and或者if操作QuerySet

### 创建模型Manager

正如我们上一节提到的，objects是所有模型的默认manager，它将获取数据库中的所有数据。我们也可以为模型自定义manager，我们将创建一个获取所有处于published状态文章的manager。

我们可以采用两种方法为模型添加manager：1. 添加额外的manager方法，2. 修改最初的manager的Queryset。第一种方法看起来像Post.objects.my_manager()，第二种看起来像Post.my_manager.all()。我们的manager允许我们通过Post.published获取文章。

编辑blog应用models.py文件来添加自定义manager：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.contrib.auth.models import User
from django.db import models
from django.utils import timezone


# Create your models here.

class PublishedManager(models.Manager):
    def get_queryset(self):
        return super(PublishedManager, self).get_queryset().filter(
            status='published')


class Post(models.Model):
    STATUS_CHOICES = (('draft', 'Draft'), ('published', 'Published'),)
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250, unique_for_date='publish')
    author = models.ForeignKey(User, related_name='blog_posts')
    body = models.TextField()
    publish = models.DateTimeField(default=timezone.now)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    status = models.CharField(max_length=10, choices=STATUS_CHOICES,
                              default='draft')

    objects = models.Manager()  # default manager
    published = PublishedManager()  # custom manager
    
    class Meta:
        ordering = ('-publish',)

    def __str__(self):
        return self.title
```

get_queryset是获取QuerySet的方法，自定义的manger增加了filter(status='published')限制。我们已经定义了了自定义manager并将其添加到Post模型中，现在我们用它来实现查询，比如，我们需要查询所有标题以’Who'开头的已发表的文章：

```python
In [4]: Post.published.filter(title__startswith='Who')
Out[4]: <QuerySet []>
```

## 建立list和detail视图

现在我们已经有了一些使用ORM的知识，可以为blog创建视图了。Django视图是一个python函数，用来接收web请求并返回web响应，视图内部实现获取期望响应的所有逻辑。

首先，我们将创建应用视图，然后为每个视图定义URL，最后我们创建HTML模板来渲染视图生成的数据。每个视图都将渲染一个模板并返回一个渲染好的HTTP响应。

###创建list和detail视图

我们先来创建一个视图来展示文章列表，编辑blog应用的views.py文件：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.shortcuts import render

from .models import Post


# Create your views here.
def post_list(request):
    posts = Post.published.all()
    return render(request, 'blog/post/list.html', {'posts': posts})

```

我们刚刚创建了第一个Django视图，post_list视图只有request对象一个输入。请注意：所有视图都需要输入request。在这个视图中，我们获取所有状态为published的文章（通过前面的published manager获得）。

最后，我们使用Django提供的render渲染模板。这个函数的输入包括request，模板地址和模板渲染需要的数据，它返回一个渲染后的HttpResponse对象。render()将request的context考虑在内，因此，给定的模板可以获得模板context处理器设置的任意变量。模板context处理器只是用来为context设置变量。我们将在第三章（扩展模板应用）学习如何使用它们。

我们创建第二个视图来展示一篇文章。向views.py中增加下面的函数：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.shortcuts import render, get_object_or_404

from .models import Post


# Create your views here.
def post_list(request):
    posts = Post.objects.all()
    return render(request, 'blog/post/list.html', {'posts': posts})


def post_detail(request, year, month, day, slug):
    post = get_object_or_404(Post, slug=slug, status='draft',
                             publish__year=year, publish__month=month,
                             publish__day=day)
    return render(request, 'blog/post/detail.html', {'post': post})
```

这是文章详细视图，这个视图的输入包括year、month、day、slug来获取某天slug为slug的文章。注意，我们在创建Post模型时向slug字段添加了unique_for_date参数，这样，我们可以确定满足某天某个slug的文章是唯一的，我们通过get_object_or_404()来获取需要的文章，这个函数获得满足输入参数的对象，如果没找到相应对象则HTTP 404。最后，我们使用render()函数来使用模板渲染数据。

### 为视图添加URL模式

URL模式由一个python正则表达式、一个视图和一个项目范围内唯一的名称组成。Django遍历每个URL模式然后停在第一个匹配的URL。然后，Django导入与URL模式对应的视图并执行，传入一个HttpRequset类的实例、关键词或位置参数。

如果你之前没有处理过正则表达式，你可能需要了解一下：https://docs.python.org/3/howto/regex.html。

从blog应用的目录下创建一个urls.py文件，并添加下面的代码：

```python
from django.conf.urls import url

from . import views

urlpatterns = [  # post views
    url(r'^$', views.post_list, name='post_list'),
    url(r'^(?P<year>\d{4})/(?P<month>\d{2})/(?P<day>\d{2})/(?P<slug>[-\w]+)/$',
        views.post_detail, name='post_detail'), ]
```

第一个URL模式并不输入任何参数并匹配post_list视图。第二个模式输入四个参数并匹配post_detail视图。让我们看一下URL模式的正则表达式：

- year：需要四位数字；
- month：需要两位数字；
- days：需要两位数字；
- slug：可以由单词和连字符组成。

注意：为每个app创建一个urls.py文件以便于其它项目重用你的应用。

现在需要将blog应用的URL模式放到项目的主URL模式中。编辑项目mysite目录下的urls.py：

```python
"""mysite URL Configuration

The `urlpatterns` list routes URLs to views. For more information please see:
    https://docs.djangoproject.com/en/1.11/topics/http/urls/
Examples:
Function views
    1. Add an import:  from my_app import views
    2. Add a URL to urlpatterns:  url(r'^$', views.home, name='home')
Class-based views
    1. Add an import:  from other_app.views import Home
    2. Add a URL to urlpatterns:  url(r'^$', Home.as_view(), name='home')
Including another URLconf
    1. Import the include() function: from django.conf.urls import url, include
    2. Add a URL to urlpatterns:  url(r'^blog/', include('blog.urls'))
"""
from django.conf.urls import url, include
from django.contrib import admin

urlpatterns = [url(r'^admin/', admin.site.urls),
    url(r'^blog/', include('blog.urls', namespace='blog', app_name='blog')), ]

```

这样我们就可以告诉Django将blog中urls.py中定义的url放到blog/下。我们给它一个名字叫blog，这样可以很容易找到这组URLs。

### 为模型规范URLs

我们可以使用前一节定义的post_detail URL来为Post对象创建规范URL。Django可以通过为模型添加get_absolute_url()来得到对象的规范URL。在这个方法里，我们使用reverse()方法实现通过名称和参数创建URL。编辑models.py文件并添加以下代码：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.contrib.auth.models import User
from django.core.urlresolvers import reverse
from django.db import models
from django.utils import timezone


# Create your models here.

class PublishedManager(models.Manager):
    def get_queryset(self):
        return super(PublishedManager, self).get_queryset().filter(
            status='published')


class Post(models.Model):
    STATUS_CHOICES = (('draft', 'Draft'), ('published', 'Published'),)
    title = models.CharField(max_length=250)
    slug = models.SlugField(max_length=250, unique_for_date='publish')
    author = models.ForeignKey(User, related_name='blog_posts')
    body = models.TextField()
    publish = models.DateTimeField(default=timezone.now)
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    status = models.CharField(max_length=10, choices=STATUS_CHOICES,
                              default='draft')

    objects = models.Manager()  # default manager
    published = PublishedManager()  # custom manager

    class Meta:
        ordering = ('-publish',)

    def __str__(self):
        return self.title

    def get_absolute_url(self):
        return reverse('blog:post_detail',
                       args=[self.publish.strftime('%Y'),
                             self.publish.strftime('%m'),
                             self.publish.strftime('%d'), self.slug])
```

注意，我们使用strftime()函数来创建URL。我们将在模板中使用get_absolute_url()方法。

> ***官方相关材料：https://docs.djangoproject.com/en/1.11/ref/models/instances/#hash***

## 为视图添加模板

我们已经为应用创建了视图和URL模式。现在需要添加模板以友好的方式来展示文章了。

在blog应用下创建以下目录和文件：

![template](/Users/apple/dev/Django/Django_by_example/figures/template.png)

上图是模板的文件结构。base.html文件将包含网站的HTML结构并将内容划分为主内容和边栏。list.html和detail.html文件将继承base.html文件来分别渲染文章列表视图和细节视图。

Django的模板语言允许我们指定如何展示数据。它基于模板标签（格式为{% tag %}）、变量（格式为{{ variable }}）、过滤器（格式为{{ variable|filter }}。可以从官网查看所有内置标签和过滤器：https://docs.djangoproject.com/en/1.11/ref/templates/builtins/。

在base.html中添加以下代码：

```html
{% load static %}
<!DOCTYPE html>
<html>
<head>
    <title>{% block title %}{% endblock %}</title>
    <link href="{% static "css/blog.css" %}" rel="stylesheet">
</head>
<body>
<div id="content">
    {% block content %}
    {% endblock %}
</div>
<div id="sidebar">
    <h2>My blog</h2>
    <p>This is my blog.</p>
</div>
</body>
</html>
```

{% load static %}告诉Django加载django.contrib.staticfile应用提供的模板标签和过滤器。加载完后，可以在整个模板范围内使用{% static %}标签，通过这个标签我们可以include本例中位于blog应用static文件夹下的blog.css文件。它将复制这个目录到你项目中的相同位置来使用已有的静态文件。

我们可以看大这里有两个{% block %}标签。这些告诉Django要在该区域定义一个block。继承这个模板的模板可以使用内容填充这个block。我们定义了一个名为title的block和一个名为content的block。

我们来编辑post/list.html文件：

```python
{% extends "blog/base.html" %}

{% block title %}My Blog{% endblock %}

{% block content %}
    <h1>My Blog</h1>
    {% for post in posts %}
        <h2>
            <a href="{{ post.get_absolute_url }}">
                {{ post.title }}
            </a>
        </h2>
        <p class="date">
            Published {{ post.publish }} by {{ post.author }}
        </p>
        {{ post.body|truncatewords:30|linebreaks }}
    {% endfor %}
{% endblock %}
```

我们使用{% extends %}模板标签来继承blog/base.html。然后我们来填充title和content block。我们遍历所有文章依次展示它们的标题、日期、作者、内容，并且每个标题附带指向文章页面的连接。对于文章的内容我们使用了两个模板过滤器：truncatewords截取指定长度的内容，linebreaks将输出转换为HTML断行格式。后续还可以添加任意过滤器应用到之前的输出。

打开shell，在blog应用根目录运行以下代码：

```python
python manage.py runserver
```

在浏览器中打开http://http://127.0.0.1:8000/blog/，我们将看到以下内容：

![html_list](/Users/apple/dev/Django/Django_by_example/figures/html_list.png)

如果将视图中post_list获取post的manager换为published，即将views.py中的post_list函数修改为：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.shortcuts import render, get_object_or_404

from .models import Post


# Create your views here.
def post_list(request):
    posts = Post.published.all()
    print posts
    return render(request, 'blog/post/list.html', {'posts': posts})
```

则将看到以下页面：

![html_published](/Users/apple/dev/Django/Django_by_example/figures/html_published.png)

这是由于我们之前输入的两篇文章时没有设置status的值，Django将其设为默认值draft，而published要求status的值为published，数据库没有查询到符合条件的值。我们可以通过shell或者admin修改这两个对象的status值。

现在，我们来编辑post/detail.html文件：

```python
{% extends "blog/base.html" %}

{% block title %}{{ post.title }}{% endblock %}

{% block content %}
    <h1>{{ post.title }}</h1>
    <p class="date">
        Published {{ post.publish }} by {{ post.author }}
    </p>
    {{ post.body|linebreaks }}
{% endblock %}
```

现在我们回到浏览器，点击任意一篇文章的标题，将看到以下页面：

![html_post](/Users/apple/dev/Django/Django_by_example/figures/html_post.png)

它的url为:http://127.0.0.1:8000/blog/2017/10/25/meet-django/。

## 添加分页

当我们添加一定数量的文章后会发现需要对文章进行分页。Django内置pagination类来帮助我们非常容易的实现分页。

将blog应用的views.py的post_list函数修改为：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from django.shortcuts import render, get_object_or_404

from .models import Post


def post_list(request):
    object_list = Post.objects.all()
    paginator = Paginator(object_list, 3)  # 3 posts in each page
    page = request.GET.get('page')
    try:
        posts = paginator.page(page)
    except PageNotAnInteger:
        # If page is not an integer deliver the first page
        posts = paginator.page(1)
    except EmptyPage:
        # If page is out of range deliver last page of results
        posts = paginator.page(paginator.num_pages)
    return render(request, 'blog/post/list.html',
                  {'page': page, 'posts': posts})
```

pagination是这样工作的:

1. 使用文章列表的queryset和每页想要显示的文章数量对Paginator进行实例化；
2. 从request的GET参数中获取请求的page；
3. 从Paginator的page()方法获取要展示的文章列表的queryset，设为posts；
4. 如果步骤2中得到page不是整数，则取第一页的文章进行展示，如果得到的page大于最大分页数，那么取最后一页的文章进行展示。
5. 使用模板渲染步骤2中的page和步骤3中的posts，返回httpResponse。

然后，我们需要创建一个展示分页的模板，这个模板可以用于任意需要分页的模板。在blog应用的templates\blog文件夹下的创建一个名为pagination.html的文件，并添加以下代码：

```python
<div class="pagination">
  <span class="step-links">
    {% if page.has_previous %}
        <a href="?page={{ page.previous_page_number }}">Previous</a>
    {% endif %}
      <span class="current">
      Page {{ page.number }} of {{ page.paginator.num_pages }}.
    </span>
      {% if page.has_next %}
          <a href="?page={{ page.next_page_number }}">Next</a>
      {% endif %}
  </span>
</div>
```

分页模板期望一个Page对象来渲染前一页链接、后一页链接、展示当前页及总页码。让我们回到blog/post/list.html模板将pagination.html模板放到{% content %}的底部：

```python
{% extends "blog/base.html" %}

{% block title %}My Blog{% endblock %}

{% block content %}
    <h1>My Blog</h1>
    {% for post in posts %}
        <h2>
            <a href="{{ post.get_absolute_url }}">
                {{ post.title }}
            </a>
        </h2>
        <p class="date">
            Published {{ post.publish }} by {{ post.author }}
        </p>
        {{ post.body|truncatewords:30|linebreaks }}
    {% endfor %}
    {% include "blog/pagination.html" with page=posts %}
{% endblock %}
```

我们传入模板的Page对象名为posts，现在我们已经将分页模板放到了文章列表模板。我们可以采用这个方法来复用分页模板。

现在，打开http://http://127.0.0.1:8000/blog/，我们会发现底部多了页码信息:

![pagination_3](/Users/apple/dev/Django/Django_by_example/figures/pagination_3.png)由于我们只有两篇文章，视图中设置的每页显示3篇文章，因此，我们只有1页，而且没有前一页、后一页链接。

将views.py中post_list函数中分页初始化改为：

```python
paginator = Paginator(object_list, 1) 
```

我们看到的页面将会变为:

![pagination_1_1](/Users/apple/dev/Django/Django_by_example/figures/pagination_1_1.png)

点击Next，我们将会看到：

![pagination_1_3](/Users/apple/dev/Django/Django_by_example/figures/pagination_1_3.png)

而且url会变为http://127.0.0.1:8000/blog/?page=2，点击previous会回到前一个页面，但是前一个页面的url由http://127.0.0.1:8000/blog/变为http://127.0.0.1:8000/blog/?page=1。

## 使用基类视图

视图是一个输入web请求输出web响应的函数，我们也可以使用类函数定义视图。Django提供基础试图类，这些视图类已经解决了HTTP方法匹配和其它功能。下面是创建视图的一个替代方法。

我们将使用Django的基本类视图ListView来修改post_list视图。这个类视图允许我们列出各种类型的对象。

编辑blog应用的views.py文件并添加以下代码：

```python
from django.views.generic import ListView


class PostListView(ListView):
    queryset = Post.objects.all()
    context_object_name = 'posts'
    paginate_by = 3
    template_name = 'blog/post/list_classview.html'
```

这个类视图与前面的post_list视图功能类似。我们告诉ListView：

- 使用queryset代替获取所有对象，我们也可以指定model=Post，这样Django将为我们创建Post.objects.all()。

- 使用内容变量posts表示查询结果，如果不指定context_object_name，默认变量名为object_list。

- 将查询结果按照每页3项进行分页显示。

- 使用自定义模板渲染页面。如果我们不设置模板，ListView将使用blog/post_list.html作为默认模板。

现在打开blog应用的urls.py文件，注释掉原来的post_list URL并为PostListView添加URL：

```python
from django.conf.urls import url

from . import views

urlpatterns = [  
    # url(r'^$', views.post_list, name='post_list'),# post views
    url(r'^$', views.PostListView.as_view(), name='post_list'),
    url(r'^(?P<year>\d{4})/(?P<month>\d{2})/(?P<day>\d{2})/(?P<slug>[-\w]+)/$',
        views.post_detail, name='post_detail'), ]
```

为了保证分页正常工作，我们需要向模板传入正确的page对象，Django ListView的page对象名为page_obj，因此，我们在templates/blog/post/下创建list_classview.html，将同一文件夹下list.html的内容拷贝过来，然后将context最后一句更改为：

```html
    {% include "blog/pagination.html" with page=page_obj %}
```

整个html为:

```python
{% extends "blog/base.html" %}

{% block title %}My Blog{% endblock %}

{% block content %}
    <h1>My Blog</h1>
    {% for post in posts %}
        <h2>
            <a href="{{ post.get_absolute_url }}">
                {{ post.title }}
            </a>
        </h2>
        <p class="date">
            Published {{ post.publish }} by {{ post.author }}
        </p>
        {{ post.body|truncatewords:30|linebreaks }}
    {% endfor %}
    {% include "blog/pagination.html" with page=page_obj %}
{% endblock %}
```

打开浏览器http://127.0.0.1:8000/blog/我们会发现PostListView与post_view的http响应一致。这是基类视图的简单例子，我们将在第十章（Building an e-Learning Platform）及后续章节继续学习。

## 总结

这一章，我们已经通过创建一个简单地blog应用了解了Django web框架。我们已经设计了数据模型并为项目应用了migrations。我们也创建了视图、模板和blog的URL，包括分页。

下一章，我们将为blog应用添加评论系统、标签功能、并允许用户通过e-mail分享文章。