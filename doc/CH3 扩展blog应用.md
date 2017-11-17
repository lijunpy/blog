# CH3 扩展blog应用

前面一章我们学习了基本的表单及如何在项目中集成第三方应用。本章的要点为：

- 创建自定义模板标签和过滤器
- 添加网站的sitemap和文章的feeds
- 使用Solr和Haystack创建搜索引擎

##创建自定义模板标签和过滤器

Django提供许多内置标签（比如{% if %}、{% block%}等），我们已经在前面开发的模板中使用了一些。从下面的链接中我们可以找到Django所有的内置标签和过滤器：https://docs.djangoproject.com/en/1.11/ref/templates/builtins/。

Django也允许我们创建自己的自定义标签来实现自定义动作。如果我们需要实现Django内置标签无法覆盖的功能时可以创建自定义标签。

###创建自定义模板标签

Django提供一下帮助函数来帮助我们快速创建自定义模板标签：

- simple_tag: 处理数据并返回字符串

- inclusion_tag: 处理数据并返回渲染的模板

- assignment_tag:  处理数据返回变量（由于simple_tag具备了assignment_tag的功能，Django1.9及之后的版本弃用该方法）

模板标签必须位于Django应用内。

在blog应用的根目录下创建名为templatetags的目录，并在新建目录下创建名为`__init__.py`的空文件，然后在于该空文件相同的目录下创建名为`blog_tags.py`的文件。目录及文件创建完成后的blog应用文件结构是这样的：

![blog_side_constructure](/Users/apple/profile/django_by_example/doc/project_1/figures/CH3/blog_side_constructure.png)

文件的名称很重要，你将在模板中使用这个名称加载自定义模板标签。

我们从创建一个simple_tag开始，该标签获取blog发布的文章数量。编辑blog_tags.py文件，添加以下代码：

```python
from django import template

register = template.Library()

from ..models import Post


@register.simple_tag
def total_posts():
    return Post.published.count()
```

我们已经创建了一个返回发布文章数量的模板标签。每个模板标签模块都需要包括一个名为register的变量，这个变量是template.Library的实例，它用于将自定义模板标签或过滤器注册到django的模板库。然后我们定义了一个名为total_posts的python函数并对其使用`@register.simple_tag`语法糖来将对该标签进行注册。Django将使用该函数的名称作为标签名称。如果需要使用其它名称作为标签名称，可以将`@register.simple_tag`更改为`@register.simple_tag(name='my_tag')`。

> 注意：
>
> 添加完模板标签后需要重启django的开发服务器才能使用新的自定义模板标签和过滤器。
>

在使用自定义模板标签之前，需要先使用`{% load %}`标签对其进行加载。打开blog/base.html模板，并在开始的位置添加`{% load blog_tags %}`来加载模板标签模块，然后通过添加`{% total_posts %}`来使用创建的标签表示所有文章，html最终看起来是这样的：

```html
{% load blog_tags %}
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
    <p>This is my blog. I've written {% total_posts %} posts so far.</p>
</div>
</body>
</html>
```

我们需要重启开发服务器来将新的文件添加到项目，按Ctrl+C停止开发服务器并运行以下命令启动开发服务器：

```python
python manage.py runserver
```

在浏览器中打开http://127.0.0.1:8000/blog/，我们可以在右侧边栏看到所有已发布的文章信息(由于查询用的是published管理器，要想拿到正确结果需要在admin网站或者python manage.py shell将文章的状态改为published)。

![blog_sp_tag](/Users/apple/profile/django_by_example/doc/project_1/figures/CH3/blog_sp_tag.png)



自定义标签的强大之处在于你可以处理任何数据并将其添加到任意模板，我们可以通过Queryset或者处理任何数据在模板中进行展示。

现在，我们将创建另一个用来在blog边栏展示最新文章的标签。这次我们将使用inclusion标签。我们可以使用inclusion标签对内容进行渲染。编辑blog_tags.py文件并添加以下代码：

```python
@register.inclusion_tag('blog/post/latest_posts.html')
def show_latest_posts(count=5):
    latest_posts = Post.published.order_by('-publish')[:count]
    return {'latest_posts': latest_posts}
```

在这段代码中，我们使用`register.inclusion_tag`注册了模板标签并制定返回数据渲染所用的模板`blog/post/latest_posts.html`。我们的模板标签可以输入可选的count参数，其默认值为5。我们使用这个变量来限制查询`Post.published.order_by('-publish')`结果的数量。注意，这个函数返回变量字典而不是单个值。Inclusion标签需要返回值的字典用来作为特定模板的内容。我们刚刚创建的模板可以用于传入可选的文章数量并这样进行显示`{% show_latest_posts 3 %}`。

现在，在templates/blog/post创建一个名为latest_posts.html模板文件并添加以下代码：

```html
<ul>
    {% for post in latest_posts %}
        <li>
            <a href="{{ post.get_absolute_url }}">{{ post.title }}</a>
        </li>
    {% endfor %}
</ul>
```

这里，我们使用latest_posts展示了无序的文章列表并通过标签返回。现在，编辑blog/base.html模板并添加新的模板标签来展示最新的3篇文章。sidebar <div>应该是这样的：

```html
<div id="sidebar">
    <h2>My blog</h2>
    <p>This is my blog. I've written {% total_posts %} posts so far.</p>
    <h3>Latests posts</h3>
    {% show_latest_posts 3 %}
</div>
```

通过传入要展示的文章数量来调用模板标签并且已经模板已经通过给定的内容进行了渲染。

现在回到浏览器刷新一下页面，边栏现在是这样的了：



![blog_sp_in_tag](/Users/apple/profile/django_by_example/doc/project_1/figures/CH3/blog_sp_in_tag.png)

最终，由于assignment_tag已废弃，我们将再次使用simple_tag创建一个assignment_tag功能的标签，该功能是将返回的值保存在一个变量中。我们将创建一个标签来展示评论最多的文章。编辑blog_tags.py文件，并添加以下代码：

```python
from django.db.models import Count


@register.simple_tag
def get_most_commented_posts(count=5):
    return Post.published.annotate(total_comments=Count('comments')).order_by(
        '-total_comments')[:count]
```

Queryset通过annotate()来使用Count聚合函数进行聚合查询。我们创建了一个通过total_comments字段保存每篇文章评论数量的queryset并通过total_comments字段来对查询结果进行排序。我们也提供了一个可选的参数count来限制返回对象的数量。

除了Count，Django还提供Avg、Max、Min和Sum等聚合函数。我们可以通过阅读官网资料对聚合函数进行更多了解：https://docs.djangoproject.com/en/1.11/topics/db/aggregation/。

编辑blog/base.html模板并在sidebar <div>元素下添加以下代码：

```html
    <h3>Most commented posts</h3>
    {% get_most_commented_posts as most_commented_posts %}
    <ul>
        {% for post in most_commented_posts %}
        <li>
            <a href="{{ post.get_absolute_url }}">{{ post.title }}</a>
        </li>
        {% endfor %}
    </ul>
```

这里最大的不同是使用了`{% template_tag as variable %}`。在我们的代码中，我们使用了`{% get_most_commented_posts as most_commented_posts %}`。这样我们可以将标签结果保存到一个新的名为most_commented_posts的变量中，然后我们使用无序列表对其进行展示。

现在，再次刷新浏览器，我们将看到这样的页面：

![blog_3_tag](/Users/apple/profile/django_by_example/doc/project_1/figures/CH3/blog_3_tag.png)

我们可以阅读官方文档了解更多关于模板标签的内容：https://docs.djangoproject.com/en/1.11/howto/custom-template-tags/。

### 创建自定义模板过滤器

Django内置许多末班过滤器来允许我们更改模板中的变量。它们是具备一个或者两个输入（第一个是要操作的对象，第二个是可选参数）的python函数。它们返回一个可以被其他过滤器处理的值。一个过滤器看起来是这样的{{ variable|my_filter}}或者可以传递参数的{{ variable|my_filter: 'foo' }}。我们可以对一个变量应用任意数量的过滤器{{ variable|filter1|filter2 }}，它们中的每一个将使用前一个过滤器的输出作为输入。

我们将创建一个自定义过滤器来允许我们的blog文章可以使用markdown语法然后在模板中将文章内容转换为HTML。markdown是一个简单易用的文本格式语法并且可以转换为HTML。我们可以了解它们的基本格式：https://daringfireball.net/projects/markdown/basics。

首先使用以下命令通过pip安装Python markdown模块：

```python
sudo pip install Markdown
```

然后编辑blog_tags文件，添加以下代码：

```python

from django.utils.safestring import mark_safe
import markdown


@register.filter(name='markdown')
def markdown_format(text):
    return mark_safe(markdown.markdown(text))
```

我们采用与模板标签一样的方式对模板过滤器进行注册。为避免我们的函数名与markdown模块混淆，我们将函数名称命名为markdown_format并将过滤器命名为markdown以便在模板中这样使用{{ variable|markdown }}。Django不对过滤器生成的HTML代码进行转义。因此，我们使用django提供的mark_safe函数对结果进行转义以保证其可以在模板中进行渲染。默认情况下，Django不信任任何HTML代码并在将其输出之前进行转义。唯一的例外是变量被标记为safe。这个特性可以预防Django生成存在潜在危险的HTML并且允许用户在知道代码将返回安全HTML时不进行转义。

现在，在post/list.html和post/detail.html中的{%extends%}之后加载模板标签：

```
{% load blog_tags %}
```

在post/detail.html中使用{{ post.body|markdown }}代替{{ post.body|linebreaks }}。

然后，在post/list.html中，使用{{ post.body|markdown|truncatechars_html:30 }}代替{{ post.body|truncatewords:30|linebreaks }}。

truncatechars_html过滤器将截取一定数量字符的字符串，以避免未关闭的HTML标签。

现在，打开浏览器的http://127.0.0.1:8000/admin/blog/post/add/页面并添加以下内容的文章：

```markdown
This is a post formatted with markdown

*This is emphasized* and **this is more emphasized**.

Here is a list:

- One
- Two
- Three

And a [link to the Django website](https://www.djangoproject.com/)
```

![add_markdown](/Users/apple/profile/django_by_example/doc/project_1/figures/CH3/add_markdown.png)

打开浏览器，我们将看到这篇文章如何渲染，我们应该看到这样的效果：

![markdown_page](/Users/apple/profile/django_by_example/doc/project_1/figures/CH3/markdown_page.png)



你可以看到，自定义模板过滤器可以有效的自定义格式，可以通过以下链接了解更多内容：https://docs.djangoproject.com/en/1.11/howto/custom-template-tags/。

## 创建sitemap

Django内置sitemap框架，可以允许我们为网站动态创建sitemaps。sitemap是告诉搜索引擎你网站上的页面、它们的相关系及更新频率的XML文件。sitemap可以帮助crawlers检索网站的内容。

Django sitemap框架依赖于django.contrib.sites（它允许用户在自己网站上使用特定网址的对象）。这满足了使用一个Django项目运行多个网站的需求。安装sitemap框架，需要在项目中同时激活sites和sitemap两个应用。编辑项目的settings.py文件，将django.contrib.site和django.contrib.sitemaps放到INSTALLED_APPS中，然后为site ID定义一个新的设置：

```python
SITE_ID = 1

# Application definition

INSTALLED_APPS = ['django.contrib.admin', 
                  'django.contrib.auth',
                  'django.contrib.contenttypes',
                  'django.contrib.sessions',
                  'django.contrib.messages',
                  'django.contrib.staticfiles',
                  'blog', 
                  'taggit',
                  'django.contrib.sites',
                  'django.contrib.sitemaps']
```

现在，运行以下命令来为Django的sites应用同步数据库：

```
python manage.py migrate
```

我们将看到这样的结果：

```
Operations to perform:
  Apply all migrations: admin, auth, blog, contenttypes, sessions, sites, taggit
Running migrations:
  Applying sites.0001_initial... OK
  Applying sites.0002_alter_domain_unique... OK
```

sites应用的模型现在与数据库同步了。现在，我们在blog应用的目录下新建一个名为sitemaps.py的文件，打开文件并添加以下代码：

```python 
from django.contrib.sitemaps import Sitemap

from .models import Post


class PostSitemap(Sitemap):
    changefreq = 'weekly'
    priority = 0.9

    def items(self):
        return Post.published.all()

    def lastmod(self, obj):
        return obj.publish
```

我们通过继承sitemap模块的Sitemap类新建自定义sitemap。changefreq和priority属性表示文章页面更新频率和他们与您的网站的相关性（最大值为1）。items方法为sitemap返回对象的query set。默认情况下，django调用模型的get_absolute_url()方法来获取其URL，我们在第一章中为Post模型实现了get_absolute_url()方法。如果希望为每篇文章都指定URL，那么你可以为sitemap类添加location方法。lastmod方法获得item()返回的每个对象并返回这篇文章的发布时间。changefreq和priority属性可以为方法或者属性。关于sitemap的更多内容详见：https://docs.djangoproject.com/en/1.11/ref/contrib/sitemaps/。

最后，我们只需要添加我们的sitemap URL，编辑项目的urls.py文件，并修改成这样：

```python 

from django.conf.urls import url, include
from django.contrib import admin
from django.contrib.sitemaps.views import sitemap

from blog.sitemaps import PostSitemap

sitemaps = {'post': PostSitemap, }

urlpatterns = [url(r'^admin/', admin.site.urls),  
               url(r'^blog/',include('blog.urls',namespace='blog',
                                     app_name='blog')),
               url(r'^sitemap\.xml$', sitemap, {'sitemaps': sitemaps},
                   name='django.contrib.sitemaps.views.sitemap')]
```

现在，我们导入了需要的模块并定义了一个sitemaps字典。我们定义一个与sitemap.xml匹配并使用sitemap视图的URL，sitemaps字典传入sitamap视图，现在在浏览器中打开http://127.0.0.1:8000/sitemap.xml，我们将看到这样的XML：

![sitemaps](/Users/apple/profile/django_by_example/doc/project_1/figures/CH3/sitemaps.png)：



每篇文章的url通过调用它的get_absolute_url()获得。lastmod属性对应我们在sitemap中指定的文章的publish日期字段，changefreq和priority属性也是从PostSitemap中获得的。你可以看到创建url时使用的域名为example.com。这个域名来自于数据库中存储的一个Site对象，example.com是我们同步数据库时自动添加的默认对象。现在在浏览器中打开http://127.0.0.1:8000/admin/sites/site/，你将看到：

![site_domain](/Users/apple/profile/django_by_example/doc/project_1/figures/CH3/site_domain.png)

这是sites框架的域名列表视图。这里你可以设置sites框架使用的域名或主机。为了生成存在于本地环境的URL，将域名更改为127.0.0.0:8000：

![domain_to_local](/Users/apple/profile/django_by_example/doc/project_1/figures/CH3/domain_to_local.png)



为了便于开发，我们将域名指定为本地。在生产环境中，我们需要使用自己的域名。

### 为blog文章创建feeds

Django内置同步feed框架用于动态生成RSS或Atom feeds，这个框架的使用方法与sitemap框架类似。

在blog应用目录下创建一个名为feeds.py的文件，并添加以下代码：

```python
from django.contrib.syndication.views import Feed
from django.template.defaultfilters import truncatewords

from .models import Post


class LatestPostsFeed(Feed):
    title = 'My blog'
    link = '/blog'
    description = 'New posts of my blog'

    def items(self):
        return Post.published.all()[：5]

    def item_title(self, item):
        return item.title

    def item_description(self, item):
        return truncatewords(item.body, 30)
```

首先，我们创建LatestPostFeed类，它是syndication框架的Feed类的子类。其中的title、link和description属性分别对应RSS元素的<title>、<link>和<discription>。

items方法获取feed包含的对象。在这个feed中我们只得到了最新的5篇文章。item_title和item_description方法获得items返回的每个对象并为每个对象返回名称和描述。我们使用内置的模板过滤器truncateword截取文章的前30个词作为描述。

现在，编辑blog应用的urls.py文件，导入新建的LatestPostFeed,并为其创建url模式。

```python
from django.conf.urls import url

from . import views
from .feeds import LatestPostsFeed

urlpatterns = [url(r'^$', views.post_list, name='post_list'),  # post views
    # url(r'^$', views.PostListView.as_view(), name='post_list'),
    url(r'^tag/(?P<tag_slug>[-\w]+)/$', views.post_list,
        name='post_list_by_tag'),
    url(r'(?P<year>\d{4})/(?P<month>\d{2})/(?P<day>\d{2})/(?P<slug>[-\w]+)/$',
        views.post_detail, name='post_detail'),
    url(r'(?P<post_id>\d+)/share/$', views.post_share, name='post_share'),
    url(r'^feed/$', LatestPostsFeed(), name='post_feed'), ]
```

在浏览器中打开http://127.0.0.1:8000/blog/feed/，我们应该看到最新五篇文章的RSS feed：

![feeds](/Users/apple/profile/django_by_example/doc/project_1/figures/CH3/feed_link.png)

如果这一个RSS客户端打开同样的URL，你将看到一个用户友好的feed。

最后一步是为blog边栏添加feed订阅链接。打开blog/base.html模板并在sidebar <div>的所有文章数量下面一行添加以下代码：

```html
<p><a href="{% url 'blog:post_feed' %}">Subscribe to my RSS feed</a></p>
```

在浏览器中打开http://127.0.0.1:8000/blog/，我们在边栏中可以看到转向blog feed的新链接：

![feed_link](/Users/apple/profile/django_by_example/doc/project_1/figures/CH3/feed_link.png)





##使用Slor和Haystack添加搜索引擎

现在，我们将为博客添加搜索功能。Django ORM允许我们使用icontains过滤器实现不区分大小写的查询。例如，我们可以使用下面的查询找到正本包含framework这个词的文章：

```python
Post.objects.filter(body__icontains='framework')
```

然而，如果需要更加强大的搜索函数，我们需要使用一个合适的搜索引擎。我们将使用Solr结合Django来创建blog的搜索引擎。Slor是一个很流行的开源搜索框架，它可以提供全文搜索、动态聚类以及其它高级搜索特性。

为了在项目中集成Solr，我们将使用Haystack。Haystack是一个Django应用，它可以作为许多搜索引擎的抽象层。它提供一个与Queryset类似的搜索API。我们从安装并配置Solr和Haystack开始。

### 安装Solr

我们需要Java运行环境V1.7级以上版本来安装Solr。可以使用java -version命令来测试你的java版本。输出可能有所变化，但是至少需要安装1.7版本。

如果没有安装Java或者版本低于要求，可以从以下网址下载JDK：http://www.oracle.com/technetwork/java/javase/downloads/index.html。

```co&#39;m&#39;m
java version "9.0.1"
Java(TM) SE Runtime Environment (build 9.0.1+11)
Java HotSpot(TM) 64-Bit Server VM (build 9.0.1+11, mixed mode)
```

检查完Java版本后，从http://archive.apache.org/dist/lucene/solr/下载Solr版本4.10.4。解压下载的文件并转向Slor安装目录的example目录(cd color-4.10.4/example/)。这个目录包含一个可用的Solr配置。从这个目录下使用以下命令来使用内置的Jetty web服务器运行Solr：

```java
java -jar start.jar
```

打开浏览器并输入以下网址，将看到下面的内容：

![solr_init](figures/CH3/solr_init.png)这是solr的管理平台。这个平台静态显示用法并允许我们管理自己的搜索backend，检查索引数据及实现查询。

### 创建Solr内核

Solr允许隔离内核中的实例。每个Solr内核是一个Lucene实例，该实例有一个Slor配置、一个数据事务和其它使用需要的配置。Solr允许我们创建并管理内核。示例配置包含一个名为collection1的内核，如果点击Core Admin目录按钮将看到这个内核的信息，如下图所示：

![solr_core](figures/CH3/solr_core.png)



我们将为blog应用创建一个内核。首先，我们需要为内核创建一个文件结构，在color-4.10.4/example/solr目录中创建一个新的名为blog的目录，然后在该文件夹中创建下面的空文件及目录：

![solr_structure](figures/CH3/solr_structure.png)



向solrconfig中添加下面的XML代码：

```xml
<?xml version="1.0" encoding="utf-8" ?>
<config>
  <luceneMatchVersion>LUCENE_36</luceneMatchVersion>
  <requestHandler name="/select" class="solr.StandardRequestHandler" default="true" />
  <requestHandler name="/update" class="solr.UpdateRequestHandler" />
  <requestHandler name="/admin" class="solr.admin.AdminHandlers" />
  <requestHandler name="/admin/ping" class="solr.PingRequestHandler">
    <lst name="invariants">
      <str name="qt">search</str>
      <str name="q">*:*</str>
    </lst>
  </requestHandler>
</config>
```

这是一个最小的Solr配置，编辑schema.xml文件并添加以下XML代码：

```xml
<?xml version="1.0" ?>
<schema name="default" version="1.5">
</schema>
```



这是一个空的schema。这个schema定义了需要在搜索引擎索引的字段及其数据类型。稍后我们将使用一个自定义schema。

现在，点击Core Amdin 目录按钮然后点击Core Core按钮。你将看到一个为内核指定信息的表单：

![solr_add_core](figures/CH3/solr_add_core.png)

使用以下数据填充表单：

- *name*: blog
- *instanceDir*: blog
- *dataDir* :data
- *config* :solrconfig.xml
- *schema* :schema.xml

name字段为这个内核的名称，instanceDir字段为内核目录，dataDir为索引数据的目录，config字段为Solr XML配置文件的名称，schema字段为Solr XML数据schema文件的名称。

现在，点击Add Core按钮，如果你看到以下信息，那么内核已经成功添加到Solr中：

![solr_added_core](figures/CH3/solr_added_core.png)



### 安装Haystack

在Django中使用Solr需要使用Haystack。通过匹配安装Haystack：

```
sudo pip install django-haystack
```

Haystack可以与几个搜索引擎交互。为了使用Solr引擎，我们需要安装pysolr模块。运行下面的命令进行安装：

```
sudo pip install pysolr
```

安装完django-haustack和pysolr后，我们需要在项目中激活Haystack，打开settings.py文件并将haystack添加到INSTALLED_APPS中：

```python

INSTALLED_APPS = ['django.contrib.admin',
                  'django.contrib.auth',
                  'django.contrib.contenttypes',
                  'django.contrib.sessions',
                  'django.contrib.messages',
                  'django.contrib.staticfiles',
                  'blog',
                  'taggit',
                  'django.contrib.sites',
                  'django.contrib.sitemaps',
                  'haystack']


```

我们需要为haystack定义搜索引擎backends。可以通过添加HAYSTACK_CONNECTIONS配置实现，向settings.py文件中添加以下代码：

```python
# search engine setting
HAYSTACK_CONNECTIONS = {
    'default': {
        'ENGINE': 'haystack.backends.solr_backend.SolrEngine',
        'URL': 'http://127.0.0.1:8983/solr/blog'
    },
}
```

注意，URL执行blog内核，Haystack现在已经可以使用Solr了。

### 创建索引

现在，我们需要注册希望保存到搜索引擎中的模型。Haystack可以通过在应用下创建一个名为search_indexes.py的文件并在该文件中注册模型实现注册。在blog应用下创建名为search_indexes.py的文件，并添加以下代码：

```python
from haystack import indexes

from .models import Post


class PostIndex(indexes.SearchIndex, indexes.Indexable):
    text = indexes.CharField(document=True, use_template=True)
    publish = indexes.DateTimeField(model_attr='publish')

    def get_model(self):
        return Post

    def index_queryset(self, using=None):
        return self.get_model().published.all()
```

这是一个Post模型的自定义SearchIndex。通过这个Index，我们告诉Haystack这个模型中的哪些数据要在搜索引擎中创建索引。这个index是indexes.SearchIndex和indexes.Indexable的子类。每个SearchIndex都要求它的其中一个字段具备document=True属性设置，比较方便的方法是将具备该属性的字段命名为text，这一字段为第一搜索字段。通过使用use_template=True的属性设置，我们告诉Haystack搜索引擎索引使用的这个字段将通过数据渲染为文档。我们通过使用model_attr属性将publish字段对应到Post模型中的publish字段。这个字段将索引的Post对象的publish字段的内容进行索引。这样的一个额外字段将有助于为索引创建额外的过滤器。get_model()方法将返回保存到索引中的文档模型。index_queryset()方法返回用于索引的queryset。注意，这里只包含状态为published的文章。

现在，在blog/templates/下创建search/indexes/blog/post_text.txt文件，并添加以下代码:

```html
{{ object.title }}
{{ object.tags.all|join:',' }}
{{ object.body }}
```

这是索引的txt字段的文档模板的默认路径。Haystack使用应用名称和模型名称来动态创建路径。每次我们对一个对象进行索引时，Haystack将基于这个模板创建一个文档然后再Solr搜索引擎创建文档索引。

现在，我们需要一个自定义索引索引，我们需要创建一个合适的Solr schema。Solr配置基于XML，因此我们需要为将要检索的数据创建一个XML schema。幸运的是，Haystack提供一种基于我们的搜索索引动态生成schema的方法。打开terminal并运行以下命令：

```python 
python manage.py build_solr_schema	
```

> 注意：
>
> 这一步报错，原因在于haystack/management/commands/build_solr_schema.py文件夹下build_template函数调用bulid_context函数，build_context函数应该返回一个字典，但是返回了一个Context对象，去掉build_context 中return只返回Context()中()中的内容即可。
>
> 这个错误，从haystack的git文件clone，对应文件已经修正，但是pip卸载再次安装，仍然没有修改，没搞清楚为什么git和pip存在区别。
>
> 暂时拷贝书中的输出到测试下一步是否可用。

将看到一个XML输出。如果看一下生成的XML文件的下部，我们将看到Haystack为PostIndex自动生成的内容：

```html
<field name="text" type="text_en" indexed="true" stored="true" multiValued="false" />
<field name="publish" type="date" indexed="true" stored="true" multiValued="false" />
```

这个XML是为Solr创建索引数据的schema。将整个XML输出(从最初的<?xml version="1.0" ?>到最终的</schema>)拷贝到Solr应用example/solr/blog/conf/schema.xml文件中。全部xml内容为：

```xml
<?xml version="1.0" ?>
<!--
 Licensed to the Apache Software Foundation (ASF) under one or more
 contributor license agreements.  See the NOTICE file distributed with
 this work for additional information regarding copyright ownership.
 The ASF licenses this file to You under the Apache License, Version 2.0
 (the "License"); you may not use this file except in compliance with
 the License.  You may obtain a copy of the License at

     http://www.apache.org/licenses/LICENSE-2.0

 Unless required by applicable law or agreed to in writing, software
 distributed under the License is distributed on an "AS IS" BASIS,
 WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
 See the License for the specific language governing permissions and
 limitations under the License.
-->

<schema name="default" version="1.5">
  <types>
    <fieldtype name="string"  class="solr.StrField" sortMissingLast="true" omitNorms="true"/>
    <fieldType name="boolean" class="solr.BoolField" sortMissingLast="true" omitNorms="true"/>
    <fieldtype name="binary" class="solr.BinaryField"/>

    <!-- Numeric field types that manipulate the value into
         a string value that isn't human-readable in its internal form,
         but with a lexicographic ordering the same as the numeric ordering,
         so that range queries work correctly. -->
    <fieldType name="int" class="solr.TrieIntField" precisionStep="0" omitNorms="true" sortMissingLast="true" positionIncrementGap="0"/>
    <fieldType name="float" class="solr.TrieFloatField" precisionStep="0" omitNorms="true" sortMissingLast="true" positionIncrementGap="0"/>
    <fieldType name="long" class="solr.TrieLongField" precisionStep="0" omitNorms="true" sortMissingLast="true" positionIncrementGap="0"/>
    <fieldType name="double" class="solr.TrieDoubleField" precisionStep="0" omitNorms="true" sortMissingLast="true" positionIncrementGap="0"/>
    <fieldType name="sint" class="solr.SortableIntField" sortMissingLast="true" omitNorms="true"/>
    <fieldType name="slong" class="solr.SortableLongField" sortMissingLast="true" omitNorms="true"/>
    <fieldType name="sfloat" class="solr.SortableFloatField" sortMissingLast="true" omitNorms="true"/>
    <fieldType name="sdouble" class="solr.SortableDoubleField" sortMissingLast="true" omitNorms="true"/>

    <fieldType name="tint" class="solr.TrieIntField" precisionStep="8" omitNorms="true" positionIncrementGap="0"/>
    <fieldType name="tfloat" class="solr.TrieFloatField" precisionStep="8" omitNorms="true" positionIncrementGap="0"/>
    <fieldType name="tlong" class="solr.TrieLongField" precisionStep="8" omitNorms="true" positionIncrementGap="0"/>
    <fieldType name="tdouble" class="solr.TrieDoubleField" precisionStep="8" omitNorms="true" positionIncrementGap="0"/>

    <fieldType name="date" class="solr.TrieDateField" omitNorms="true" precisionStep="0" positionIncrementGap="0"/>
    <!-- A Trie based date field for faster date range queries and date faceting. -->
    <fieldType name="tdate" class="solr.TrieDateField" omitNorms="true" precisionStep="6" positionIncrementGap="0"/>

    <fieldType name="point" class="solr.PointType" dimension="2" subFieldSuffix="_d"/>
    <fieldType name="location" class="solr.LatLonType" subFieldSuffix="_coordinate"/>
    <fieldtype name="geohash" class="solr.GeoHashField"/>

    <fieldType name="text_general" class="solr.TextField" positionIncrementGap="100">
      <analyzer type="index">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" enablePositionIncrements="true" />
        <!-- in this example, we will only use synonyms at query time
        <filter class="solr.SynonymFilterFactory" synonyms="index_synonyms.txt" ignoreCase="true" expand="false"/>
        -->
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory" ignoreCase="true" words="stopwords.txt" enablePositionIncrements="true" />
        <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
    </fieldType>

    <fieldType name="text_en" class="solr.TextField" positionIncrementGap="100">
      <analyzer type="index">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.StopFilterFactory"
                ignoreCase="true"
                words="lang/stopwords_en.txt"
                enablePositionIncrements="true"
                />
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.EnglishPossessiveFilterFactory"/>
        <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
        <!-- Optionally you may want to use this less aggressive stemmer instead of PorterStemFilterFactory:
          <filter class="solr.EnglishMinimalStemFilterFactory"/>
        -->
        <filter class="solr.PorterStemFilterFactory"/>
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.StandardTokenizerFactory"/>
        <filter class="solr.SynonymFilterFactory" synonyms="synonyms.txt" ignoreCase="true" expand="true"/>
        <filter class="solr.StopFilterFactory"
                ignoreCase="true"
                words="lang/stopwords_en.txt"
                enablePositionIncrements="true"
                />
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.EnglishPossessiveFilterFactory"/>
        <filter class="solr.KeywordMarkerFilterFactory" protected="protwords.txt"/>
        <!-- Optionally you may want to use this less aggressive stemmer instead of PorterStemFilterFactory:
          <filter class="solr.EnglishMinimalStemFilterFactory"/>
        -->
        <filter class="solr.PorterStemFilterFactory"/>
      </analyzer>
    </fieldType>

    <fieldType name="text_ws" class="solr.TextField" positionIncrementGap="100">
      <analyzer>
        <tokenizer class="solr.WhitespaceTokenizerFactory"/>
      </analyzer>
    </fieldType>

    <fieldType name="ngram" class="solr.TextField" >
      <analyzer type="index">
        <tokenizer class="solr.KeywordTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
        <filter class="solr.NGramFilterFactory" minGramSize="3" maxGramSize="15" />
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.KeywordTokenizerFactory"/>
        <filter class="solr.LowerCaseFilterFactory"/>
      </analyzer>
    </fieldType>

    <fieldType name="edge_ngram" class="solr.TextField" positionIncrementGap="1">
      <analyzer type="index">
        <tokenizer class="solr.WhitespaceTokenizerFactory" />
        <filter class="solr.LowerCaseFilterFactory" />
        <filter class="solr.WordDelimiterFilterFactory" generateWordParts="1" generateNumberParts="1" catenateWords="0" catenateNumbers="0" catenateAll="0" splitOnCaseChange="1"/>
        <filter class="solr.EdgeNGramFilterFactory" minGramSize="2" maxGramSize="15" side="front" />
      </analyzer>
      <analyzer type="query">
        <tokenizer class="solr.WhitespaceTokenizerFactory" />
        <filter class="solr.LowerCaseFilterFactory" />
        <filter class="solr.WordDelimiterFilterFactory" generateWordParts="1" generateNumberParts="1" catenateWords="0" catenateNumbers="0" catenateAll="0" splitOnCaseChange="1"/>
      </analyzer>
    </fieldType>
  </types>

  <fields>
    <!-- general -->
    <field name="id" type="string" indexed="true" stored="true" multiValued="false" required="true"/>
    <field name="django_ct" type="string" indexed="true" stored="true" multiValued="false"/>
    <field name="django_id" type="string" indexed="true" stored="true" multiValued="false"/>
    <field name="_version_" type="long" indexed="true" stored ="true"/>

    <dynamicField name="*_i"  type="int"    indexed="true"  stored="true"/>
    <dynamicField name="*_s"  type="string"  indexed="true"  stored="true"/>
    <dynamicField name="*_l"  type="long"   indexed="true"  stored="true"/>
    <dynamicField name="*_t"  type="text_en"    indexed="true"  stored="true"/>
    <dynamicField name="*_b"  type="boolean" indexed="true"  stored="true"/>
    <dynamicField name="*_f"  type="float"  indexed="true"  stored="true"/>
    <dynamicField name="*_d"  type="double" indexed="true"  stored="true"/>
    <dynamicField name="*_dt" type="date" indexed="true" stored="true"/>
    <dynamicField name="*_p" type="location" indexed="true" stored="true"/>
    <dynamicField name="*_coordinate"  type="tdouble" indexed="true"  stored="false"/>


    <field name="publish" type="date" indexed="true" stored="true" multiValued="false" />

    <field name="text" type="text_en" indexed="true" stored="true" multiValued="false" />

  </fields>

  <!-- field to use to determine and enforce document uniqueness. -->
  <uniqueKey>id</uniqueKey>

  <!-- field for the QueryParser to use when an explicit fieldname is absent -->
  <defaultSearchField>text</defaultSearchField>

  <!-- SolrQueryParser configuration: defaultOperator="AND|OR" -->
  <solrQueryParser defaultOperator="AND"/>
</schema>
```

在浏览器中打开http://127.0.0.1:8983/solr/点击core Admin目录按钮，然后点击blog，然后点击reload:

![solr_reload](figures/CH3/solr_reload.png)

我们重新载入了内核以便更新schema.xml的更改。当内核结束reload时，新的schema可以对索引新的数据。

### 索引数据

我们为Solr创建blog中的文章索引。打开teminal运行以下命令：

```
python manage.py rebuild_index
```

你将看到以下警告信息：

```
WARNING: This will irreparably remove EVERYTHING from your search index in connection 'default'.
Your choices after this are to restore from backups or rebuild via the `rebuild_index` command.
Are you sure you wish to continue? [y/N] 
```

输入y，Haystack将清空搜索索引并填入所有发布的文章。你应该看到这样的输出：

```
Removing all documents from your index because you said so.
All documents removed.
Indexing 5 posts
```

在浏览器中打开http://127.0.0.1:8983/solr/#/blog，在Statistics下你将看到文件索引数量：

![solr_index](figures/CH3/solr_index.png)



现在，在浏览器中打开http://127.0.0.1:8983/solr/#/blog/query，这是Solr提供的一个query接口，点击Excute query按钮。默认搜索返回所有内核中的文件索引。你将看到一个JSON格式的输出，输出内容是这种样子的：

```JSON
{
  "responseHeader": {
    "status": 0,
    "QTime": 13
  },
  "response": {
    "numFound": 5,
    "start": 0,
    "docs": [
      {
        "django_ct": "blog.post",
        "text": "Meet Django\njazz,music\nDjango is a high-level Python Web framework that encourages rapid development and clean, pragmatic design. Built by experienced developers, it takes care of much of the hassle of Web development, so you can focus on writing your app without needing to reinvent the wheel. It’s free and open source.",
        "django_id": "1",
        "id": "blog.post.1",
        "publish": "2017-10-25T01:19:22Z"
      },
      {
        "django_ct": "blog.post",
        "text": "New title\njazz\nPost body",
        "django_id": "2",
        "id": "blog.post.2",
        "publish": "2017-10-25T02:13:00Z"
      },
      {
        "django_ct": "blog.post",
        "text": "a post for taggit test\njazz,other,third\nthis is for tags test",
        "django_id": "3",
        "id": "blog.post.3",
        "publish": "2017-11-09T03:25:52Z"
      },
      {
        "django_ct": "blog.post",
        "text": "a post for similar test\njazz\nused for queryset annotate test",
        "django_id": "4",
        "id": "blog.post.4",
        "publish": "2017-11-09T03:27:14Z"
      },
      {
        "django_ct": "blog.post",
        "text": "MarkDown post\nmarkdown\nThis is a post formatted with markdown\n--------------------------------------\n\n*This is emphasized* and **this is more emphasized**.\n\nHere is a list:\n\n* One\n* Two\n* Three\n\nAnd a [link to the Django website](https://www.djangoproject.com/)",
        "django_id": "5",
        "id": "blog.post.5",
        "publish": "2017-11-10T03:20:49Z"
      }
    ]
  }
}

```

这是搜索索引为每篇文章保存的数据。text字段包含用标题、标签和正本（用\分隔)，正如我们在search_indexes.py中定义的内容。

我们已经使用python manage.py rebuild_index来移除了index的所有内容并对文章重新进行索引。我们可以使用python manage.py update_index在不移除所有对象的情况下更新索引。而且，可以通过设置参数 —age=<num_hours>来更新更少的对象。我们可以通过它来更新Solr中的索引。

### 新建一个搜索页面

现在，我们新建一个允许用户搜索文章的自定义视图。首先，我们需要一个搜索表单。编辑blog应用下的forms.py并添加以下form：

```python
class SearchForm(forms.Form):
    query = forms.CharField()
```

用户通过query字段输入搜索内容。编辑blog应用的views.py文件并添加以下代码：

```python
from .forms import SearchForm
from haystack.query import SearchQuerySet


def post_search(request):
    form = SearchForm()
    if 'query' in request.GET:
        form = SearchForm(request.GET)
        if form.is_valid():
            cd = form.cleaned_data
            results = SearchQuerySet().models(Post).filter(
                content=cd['query']).load_all()
            # count total results
            total_results = results.count()
            return render(request, 'blog/post/search.html',
                          {'form': form, 'cd': cd, 'results': results,
                           'total_results': total_results})
    else:
        return render(request,'blog/post/search.html',{'form':form})
      
```

在这个视图中，我们首先需要初始化刚刚创建的SearchForm。我们将使用GET方法提交表单以便于URL可以包含query参数。我们查看request.GET字典中是否包含‘query’来判断表单是否提交。如果表单已经提交，我们使用GET中的数据对其进行初始化并检查输入的数据是否有效，如果表单有效，则使用SearchQuerySet（已包含给定的查询）对索引的文章对象实现一次查询。load_all()方法从数据库加载所有相关文章对象。我们使用这个方法从数据库中获取数据以避免对结果进行遍历时每个对象都要访问数据库。我们将结果的数量保存到total_results变量中并将其作为context的一个变量来渲染模板。

search视图已经实现了，现在，我们需要创建一个模板来暂时表单及用户搜索结果，在templates/blog/post/目录下新建一个名为search.html的文件并添加以下代码：

```python
{% extends "blog/base.html" %}

{% block title %}Search{% endblock %}

{% block content %}
    {% if "query" in request.GET %}
        <h1>Posts containing "{{ cd.query }}"</h1>
        <h3>Found {{ total_results }} result{{ total_results|pluralize }}</h3>
        {% for result in results %}
            {% with post=result.object %}
                <h4><a href="{{ post.get_absolute_url }}">{{ post.title }}</a></h4>
                {{ post.body|truncatewords:5 }}
            {% endwith %}
        {% empty %}
            <p>There are no results for your query.</p>
        {% endfor %}
        <p><a href="{% url 'blog:post_search' %}">Search again</a></p>
    {% else %}
        <h1>Search for posts</h1>
        <form action="." method="get">
            {{ form.as_p }}
            <input type="submit" value="Search">
        </form>
    {% endif %}
{% endblock %}
```

在Search视图中，我们通过query参数来判断表单是否提交，在表单提交之前，我们展示表单及一个确定按钮，在表单提交之后，我们展示查询结果，查询总数及查询结果列表。每个rusult为一个Solr返回并通过Haystack打包的文档。我们需要使用result.object来访问查询结果对应的文章对象。

最后，编辑blog应用的urls.py并添加以下URL模式：

```python
url(r'^search/$', views.post_search, name='post_search'),
```

现在，在浏览器中打开http://127.0.0.1:8000/blog/search/，我们将看到如下结果:



![solr_search](figures/CH3/solr_search.png)

现在，在搜索框中输入搜索内容(这里输入了‘django')并点击Search按钮，我们将看这样的结果：

![solr_search_result](figures/CH3/solr_search_result.png)

现在，你的项目内置了一个强大的搜索引擎，但是Solr和Haystack可以实现更多的功能。Haystack包含搜索视图、表单及高级函数。可以通过以下链接了解HayStack更多内容：http://django-haystack.readthedocs.io/en/latest/。

Solr可以通过自定义schema来实现各种需求。你可以在创建索引或者搜索时结合分析器、标记器及token过滤器为网站内容提供更精确的搜索。可以通过以下链接了解更多可能：https://wiki.apache.org/solr/AnalyzersTokenizersTokenFilters。

## 总结

本章我们学习了如何自定义Django模板标签和过滤器，我们为搜索引擎爬取网站创建了sitemap并为订阅创建了RSS feed。我们还通过Solr和Haystack为blog添加了搜索引擎。

在下一章中，我们将学习使用Django权限框架实现社交网站、创建自定义用户配置文件并实现社交认证。