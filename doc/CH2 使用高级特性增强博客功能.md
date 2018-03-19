#CH2 使用高级特性增强博客功能

前面一章，我们建立了一个简单的blog应用。现在，我们使用高级特性对这个blog进行完善，我们将增加通过e-mail分享文章、添加评论、添加文章标签、获取相似文章等。这一章我们将学到以下主题：

- 使用Django发送e-mails；
- 创建forms并在视图中处理它们；
- 根据模型创建forms；
- 集成第三方应用；
- 创建复杂的QuerySets

## 通过e-mail分享文章

首先，我们允许用户通过发送邮件的方式分享文章。想一下我们如何通过上一章学到views、URLs和templates实现这个功能。如果要允许用户通过e-mail发送邮件，我们需要：

- 为用户创建一个form来填写它们的名字、e-mail，接收文章的e-mail和评论（可选）；

- 在views.py文件中创建一个视图来处理post的数据并发送e-mail；

- 在urls.py文件中为新建立的视图添加URL。

- 创建一个模板来展示表单。


### 使用Django创建form

form即为表单，下面表述中form与表单意义相同。

我们从创建分享文章的form开始。Django内置form框架帮助我们非常方便的创建form。form矿建允许我们定义form的字段、指定它们展示的方式、输入数据的验证方式。Django form矿建还提供灵活的方法来渲染form和处理数据。

Django提供两个基准类来创建forms:

Form：帮助我们创建标准forms；

ModelForm：帮助我们创建增加或者修改模型实例的forms。

首先，在blog应用的根目录新建一个名为forms.py的文件，并添加以下代码：

```python
from django import forms


class EmailPostForm(forms.Form):
    name = forms.CharField(max_length=25)
    email = forms.EmailField()
    to = forms.EmailField()
    comments = forms.CharField(required=False, widget=forms.Textarea)

```

这是你的第一个django表单（form）。我们来看一下通过集成Form类创建的form。我们使用不同的字段对输入进行验证。

> 注意：
>
> Forms可以放在Django项目的任何位置，为了方便起见，我们将其放在每个项目的forms.py文件中。
>

name字段是一个CharField。这种类型的字段渲染一个`<input type="text">`的HTML元素。每一个字段都对应一个小组件，这个小组件决定HTML如何展示该字段。默认的组件可以通过设置widget属性进行覆盖。在comments字段中，我们使用Textarea组件将其展示为一个`<textarea>`HTML元素来代替默认的`<input>`元素。

字段验证还依赖字段类型。例如，email和to字段为EmailField，两个字段都需要有效地e-mail地址，否则字段验证将引发forms.ValidationError异常并且form无法通过验证。form验证还会考虑其他参数：我们定义了一个最大长度为25的name字段并将comment字段设置为required=False。form验证时会将这些都考虑在内。这个form中使用的字段类型只是django表单字段的一小部分，所有的表单字段可以参考：https://docs.djangoproject.com/en/1.11/ref/forms/fields/。

### 在视图中处理表单

我们需要创建了一个新的视图来处理表单并在它成功提交时发送e-mail。编辑blog应用的views.py写入以下代码：

```python
from .forms import EmailPostForm


def post_share(request, post_id):
    # Retrieve post by id
    post = get_object_or_404(Post, id=post_id, status='published')

    if request.method == 'POST':
        # Form was submitted
        form = EmailPostForm(request.POST)
        if form.is_valid():
            # Form fields passed validation
            cd = form.cleaned_data
            # ... send email
    else:
        form = EmailPostForm()
    return render(request, 'blog/post/share.html', {'post': post, 'form': form})

```

该表单将实现以下功能：

我们定义了post_share视图，该视图输入request对象和post_id作为参数。

我们使用get_object_or_404()通过id获取文章并且要求文章状态为published。

展示初始表单和处理提交的数据使用相同的视图。我们使用request方法对其进行区分，如果request方法为GET，我们将展示一个空的表单；如果request方法为POST，那么表单将被提交并且需要处理。因此我们使用request.method="POST"来区分这两种情况。

下面是展示和处理表单的过程：

1. 当使用GET请求视图时，我们创建一个新的form类在模板中展示空的表单：

   `form=EmailPostForm()`

2. 用户填写表单并通过POST提交，然后，我们在POST部分使用提交的数据创建了一个表单类视图：

    ```python
     if request.method == 'POST':
           # Form was submitted
           form = EmailPostForm(request.POST)
    ```

3. 然后，我们使用is_valid方法对提交的数据进行验证，这个方法对表单中的数据进行验证，如果数据均为有效数据，则会返回Ture，否则会返回False。如果验证为False我们可以通过访问form.errors看到错误列表：

4. 如果表单没有通过验证，我们将使用提交的数据再次渲染表单，并且在模板中显示验证错误。

5. 如果表单通过验证，我们通过form.cleaned_data获取数据，这个属性为表单字段名和值的属性。

    > 注意：
    >
    > 如果表单字段没有验证，cleaned_data将值包含通过验证的字段。


现在我们需要学习如何使用Django发送邮件了。

### 使用Django发送邮件

使用Django发送邮件非常简单。首先，我们需要一个本地SMTP服务器或者在项目setting.py中添加以下设置来配置一个外部SMTP服务器：

EMAIL_HOST: SMTP服务器主机。默认为localhost；

EMAIL_PORT: SMTP服务器端口。默认为25；

EMAIL_HOST_USER: the SMTP 服务器的用户名；

EMAIL_HOST_PASSWORD:  SMTP 服务器的密码；

EMAIL_USE_TLS: 是否使用TLS安全连接；

EMAIL_USE_SSL: 是否使用隐式TLS安全连接。

如果没有本地SMTP服务器，可以使用e-mail提供者的SMTP服务器。下面的简单配置是使用hotmail账户通过hotmail服务器发送e-mail的配置(https://outlook.live.com/owa/?path=/options/popandimap)：

```python
EMAIL_HOST = 'smtp.gmail.com'
EMAIL_HOST_USER = 'your_account@gmail.com'
EMAIL_HOST_PASSWORD = 'your_password'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
```

> 注意：
>
> 配置中，EMAIL_USE_TLS和EMAIL_USE_SSL都默认设置为False，需要配置其中一个为True，但是不能两个都设置为True。一般端口587对应TLS，端口465对应SSL（加强TSL）。

在teminal的项目根目录输入命令：python manage.py shell打开Python shell并发送邮件：

```python
from django.core.mail import send_mail
>>> send_mail('Django mail', 'This e-mail was sent with Django.', 'your_account@gmail.com', ['your_account@gmail.com'], fail_silently=False) 
```

send_mail()输入主题、消息、发送者、接受者列表作为参数，通过设置fail_silently=False可以在邮件没有正确发送的时候引发异常。如果输出为1，那么邮件就正常发送了。如果采用setting.py中设置google web服务器发送邮件，需要正常访问以下网址：https://www.google.com/settings/security/lesssecureapps。

####发送邮件测试

国内无法访问https://www.google.com/settings/security/lesssecureapps。因此，测试了hotmail邮箱和163邮箱：

测试环境：python2.7，Django1.11

#####hotmail邮箱

下面的简单配置是使用hotmail账户通过hotmail服务器发送e-mail的配置(https://outlook.live.com/owa/?path=/options/popandimap)：

```python
EMAIL_HOST = 'smtp-mail.outlook.com'
EMAIL_HOST_USER = 'your_account@hotmail.com'
EMAIL_HOST_PASSWORD = 'your_password'
EMAIL_PORT = 587
EMAIL_USE_TLS = True
```

在teminal的项目根目录输入命令：python manage.py shell打开Python shell并发送邮件：

```python
from django.core.mail import send_mail
>>> send_mail('Django mail', 'This e-mail was sent with Django.', 'your_account@hotmail.com', ['your_account@163.com'], fail_silently=False) 
```

如果send_mail第三个参数your_account@hotmail.com与EMAIL_HOST_USER一致，邮件正常发送。但是如果接收邮件只有阿里云企业邮箱，该邮箱没有返回，需要在程序中增加时间限制，否则邮件发送正常但是程序会长时间等待接收反馈信息而无法执行其他命令。

如果send_mail第三个参数与EMAIL_HOST_USER不一致，会引发SMTPDataError 异常，无法发送邮件。

#####163邮箱

```python
EMAIL_HOST = 'smtp.163.com'
EMAIL_HOST_USER = 'your_account@163.com'
EMAIL_HOST_PASSWORD = 'your_auth_code'  #邮箱的授权码而非密码
EMAIL_PORT = 465
EMAIL_USE_SSL = True
```

在teminal的项目根目录输入命令：python manage.py shell打开Python shell并发送邮件：

```python
from django.core.mail import send_mail
>>> send_mail('Django mail', 'This e-mail was sent with Django.', 'your_account@hotmail.com', ['your_account@163.com'], fail_silently=False) 
```

注意，这里的password不是邮箱密码，而是在[邮箱]-[设置]-[POP3/SMTP/IMAP]中设置的授权码。

如果send_mail第三个参数与EMAIL_HOST_USER一致，引发SMTPDataError: (554, 'DT:SPM 163 smtp11,D8CowADnvQK5UwFa6C2ZAw--.46120S2 1510036409,please see http://mail.163.com/help/help_spam_16.htm?ip=120.194.143.53&hostid=smtp11&time=1510036409')异常，http://mail.163.com/help/help_spam_16.htm?ip=120.194.143.53&hostid=smtp11&time=1510036409的对应内容为该邮件被视为垃圾邮件，更改主题与内容后仍无效。需要进一步了解163判断垃圾邮件的依据。

如果send_mail第三个参数为hotmail邮箱，邮件无法发送，引发SMTPSenderRefused: (553, 'Mail from must equal authorized user')异常。即发送信息邮箱必须与授权邮箱一致。

#### 发送邮件测试总结

1. 测试环境：python2.7+Django1.11。
2. 尽量使用hotmail邮件作为settings中的邮箱；
3. send_mail中的发送邮箱最好与settings中的授权邮箱一致。

现在，将发送邮件功能添加到视图中，将post_share视图更改为：

```python
from .forms import EmailPostForm
from django.core.mail import send_mail


def post_share(request, post_id):
    # Retrieve post by id
    post = get_object_or_404(Post, id=post_id, status='published')
    sent = False
    if request.method == 'POST':
        # Form was submitted
        form = EmailPostForm(request.POST)
        if form.is_valid():
            # Form fields passed validation
            cd = form.cleaned_data
            post_url = request.build_absolute_uri(post.get_absolute_url())
            subject = '{}({})recommends you read "{}"'.format(cd['name'],
                                                              cd['email'],
                                                              post.title)
            message = 'read"{}" at {} \n\n\'s comments:{}'.format(post.title,
                                                                  post_url,
                                                                  cd['name'],
                                                                  cd[
                                                                      'comments'])
            send_mail(subject, message, cd['email'], [cd['to']])
            sent = True
            # ... send email
    else:
        form = EmailPostForm()
    return render(request, 'blog/post/share.html',
                  {'post': post, 'form': form, 'sent': sent})
```

这里，我们定义了一个变量sent，当邮件发送成功时，sent设为True。后续我们会在模板中使用该变量，当表单正确提交且邮件发送成功时显示成功信息。这里使用get_absolute_url()方法获取在邮件中用到的文章的链接，我们将该函数作为request.build_absolute_uri()的输入来构建包含http的完整URL。我们使用验证的表单的cleaned data作为邮件的主题和内容，然后将邮件发送到表单中to一栏中添加的地址中。

现在视图完成了，我们为其添加URL，打开blog应用下的urls.py文件，添加以下内容：

```python
urlpatterns = [
    # ...
    url(r'^(?P<post_id>\d+)/share/$', views.post_share,
        name='post_share'),
]
```

### 在模板中渲染表单

创建完表单，完成视图并添加URL后，我们还需要为这个视图添加模板。在blog/templates/blog/post目录下创建名为share.html的文件，添加以下内容：

```HTML
{% extends "blog/base.html" %}

{% block title %}Share a post{% endblock %}

{% block content %}
    {% if sent %}
        <h1>E-mail successfully sent</h1>
        <p>
            "{{ post.title }}" was successfully sent to {{ form.to }}.
        </p>
    {% else %}
        <h1>Share "{{ post.title }}" by e-mail</h1>
        <form action="." method="post">
            {{ form.as_p }}
            {% csrf_token %}
            <input type="submit" value="Send e-mail">
        </form>
    {% endif %}
{% endblock %}
```

这是展示表单的模板，当发送邮件成功时该模板则会显示成功信息。我们创建了通过POST提交的表单：

```HTML
<form action="." method="post">
```

然后，我们包含了form实例，我们通过as_p方法告诉Django将字段渲染为HTML的p元素（我们还可以通过as_table方法将字段渲染为table或者通过as_ul方法将字段渲染为ul）。如果我们要渲染每个字段，我们可以对每个字段进行迭代：

```HTML
 {%  for field in form %}
    <div>
      {{ field.errors }}
      {{ field.label_tag }}{{ field.field }}
    </div>
 {% endfor %}
```

模板标签中的{% csrf_token %}模板标签引入一个自动生成避免CSRF袭击所用token的隐藏字段。关于CSRF可以从以下网站获取更多信息：“https://en.wikipedia.org/wiki/Cross-site_request_forgery。该字段将会自动生成一个隐藏输入：

```HTML
<input type='hidden' name='csrfmiddlewaretoken' value='4WZ53yXqRomGrnH1xFeaXlVGeqPQzgJdGAE3D9ZoWpY9qknlUFLNyRMgFqATIRea' />
```

> 注意：
>
> 默认，Django为所有POST请求检查CSRFtoken。所以我们需要通过POST提交的表单添加csrf_token。



编辑blog/post/detail.html模板并在{{ post.body|linebreaks }}后添加分享链接：

```HTML
<p>
 <a href="{% url "blog:post_share" post.id %}">
    Share this post
 </a>
</p>
```

我们通过Django的url模板标签动态生成URL。我们使用了命名空间为blog名称为post_share的URL，我们将post.id作为参数传入绝对URL中。

现在，在teminal中项目根目录下运行：

```python
    python manage.py runserver 
```

在浏览器中打开http://127.0.0.1:8000/blog/，点击任意文章标题到文章详细内容页面，在正文后面，你可以看到以下链接：

![post_share_link](../../../../.Trash/post_share_form.png)

点击Share this post，我们将跳到分享邮件分享页面：

![post_share_form](/Users/apple/profile/django_by_example/blog/doc/figures/CH2/post_share_form.png)

表单的css位于static/css/blog.css文件汇总。当我们点击send e-mail按钮时，表单将被提交并验证。如果所有字段都通过验证，且邮件正常发送，我们将得到以下页面：

![post_shared](/Users/apple/profile/django_by_example/blog/doc/figures/CH2/post_shared.png)

如果输入包含无效数据，根据浏览器的不同，可能点击send e-mail显示错误提示从而无法跳转，也可能跳转后显示错误提示，重新填写表单（通过验证的字段无需重复填写）。

## 创建评论系统

现在我们为博客创建一个评论系统，用户可以在这里对文章进行评论。我们需要做的工作包括：

- 创建一个保存评论的模型；
- 创建一个提交评论并验证输入的表单；
- 创建一个处理表单并将新评论保存到数据库中的视图；
- 编辑文章详细内容模板来展示评论列表及添加新评论的表单。

首先，我们创建一个保存评论的模型。打开blog应用的models.py文件并添加以下代码：

```python
class Comment(models.Model):
    post = models.ForeignKey(Post, related_name='comments')
    name = models.CharField(max_length=80)
    email = models.EmailField()
    body = models.TextField()
    created = models.DateTimeField(auto_now_add=True)
    updated = models.DateTimeField(auto_now=True)
    active = models.BooleanField(default=True)

    class Meta:
        ordering = ('created',)

    def __str__(self):
        return 'Comment by {} on {}'.format(self.name, self.post)
```

这是我们的Comment模型。它包含与文章建立联系的外键post，这种在Comment中定义的多对一关系是指一个评论必须针对某篇文章但是一篇文章可以有多个评论，这里的related_name属性用于通过文章查看评论，定义完成后，我们可以通过comment.post获得文章并通过post.comments.all()获得文章的所有评论。如果没有定义related_name属性，django将使用模型名称的小写形式加`_set`后缀(即comment_set)获得。

我们通过官网可以了解更多关于多对一关系的内容：https://docs.djangoproject.com/en/1.11/topics/db/examples/many_to_one/。

这里添加了active自动来手动屏蔽某些评论。我们使用create字段对文章的评论进行默认排序。

我们刚刚创建的Comment模型需要同步到数据库，在terminal的项目的根目录下运行以下命令：

```python
python manage.py makemigrations blog
```

我们将看到以下输出：

```
Migrations for 'blog':
  blog/migrations/0002_comment.py
    - Create model Comment
```

Django在blog应用的migrations文件夹下创建了`0002_comment.py`文件。现在需要运行以下指令：

```
python manage.py migrate
```

我们将看到以下输出：

```
Operations to perform:
  Apply all migrations: admin, auth, blog, contenttypes, sessions
Running migrations:
  Applying blog.0002_comment... OK
```

现在，数据库中多了一个名为blog_comment的数据库表。

现在，我们可以把新模型添加到admin网站上，打开blog应用的admin.py文件并添加以下代码：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.contrib import admin

from .models import Post, Comment


# Register your models here.
class PostAdmin(admin.ModelAdmin):
    list_display = ('title', 'slug', 'author', 'publish', 'status')
    list_filter = ('status', 'created', 'publish', 'author')
    search_fields = ('title', 'body')
    prepopulated_fields = {'slug': ('title',)}
    raw_id_fields = ('author',)
    date_hierarchy = 'publish'
    ordering = ['status', 'publish']


class CommentAdmin(admin.ModelAdmin):
    list_display = ('name', 'email', 'post', 'created', 'active')
    list_filter = ('active', 'created', 'updated')
    search_fields = ('name', 'email', 'body')


admin.site.register(Post, PostAdmin)
admin.site.register(Comment, CommentAdmin)
```

运行python manage.py runserver，在浏览器中打开http://127.0.0.1:8000/admin/，可以看到评论出现在admin网站中。现在我们可以通过admin网站对comment实例进行管理。

![comment_admin](/Users/apple/profile/django_by_example/blog/doc/figures/CH2/comment_admin.png)

### 从模型中创建表单

我们还需要创建一个form来允许用户对文章进行评论。Django有两种类型的表单：Form和ModelForm。前面一节我们使用了Form来允许用户通过邮件分享文章。本节的这种情况则需要使用ModelForm，因为我们创建的Form与Comment模型息息相关。编辑blog应用的forms.py文件并添加以下内容：

```python
from .models import Comment


class CommentForm(forms.ModelForm):
    class Meta:
        model = Comment
        fields = ('name', 'email', 'body')
```

从模型创建表单只需在表单的Meta类中申明使用哪个模型即可。每个模型字段类型都有对应的表单字段类型，表单验证会使用我们定义的模型字段类型。默认情况下，Django为模型中的所有字段创建一个表单字段。然而，我们也可以在表单的Meta类中使用fields来告诉框架希望表单包括的字段。对于CommentForm而言，我们只在表单中使用name、email和body字段。

###在视图中处理ModelForms

我们使用文章视图对表单进行实例化，并对其进行处理以保证其简洁。编辑blog应用的views.py文件，编辑后的views.py文件为：

```python
# -*- coding: utf-8 -*-
from __future__ import unicode_literals

from django.core.mail import send_mail
from django.core.paginator import Paginator, EmptyPage, PageNotAnInteger
from django.shortcuts import render, get_object_or_404
from django.views.generic import ListView

from .forms import EmailPostForm, CommentForm
from .models import Post


class PostListView(ListView):
    queryset = Post.objects.all()
    context_object_name = 'posts'
    paginate_by = 3
    template_name = 'blog/post/list.html'


def post_list(request):
    object_list = Post.objects.all()
    paginator = Paginator(object_list, 1)  # 1 posts in each page
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


def post_detail(request, year, month, day, slug):
    post = get_object_or_404(Post, slug=slug, status='draft',
                             publish__year=year, publish__month=month,
                             publish__day=day)
    # list of active comments for this post
    comments = post.comments.filter(active=True)
    if request.method == "POST":
        # a comment was posted
        comment_form = CommentForm(data=request.POST)
        if comment_form.is_valid():
            # create comment object but don't save to database yet
            new_comment = comment_form.save(commit=False)
            # assign the current post to the comment
            new_comment.post = post
            # save the comment to the database
            new_comment.save()
    else:
        comment_form = CommentForm()
    return render(request, 'blog/post/detail.html',
                  {'post': post, 'comments': comments,
                   'comment_form': comment_form})


# from mysite.settings import EMAIL_HOST_USER

def post_share(request, post_id):
    # Retrieve post by id
    post = get_object_or_404(Post, id=post_id)
    sent = False
    if request.method == 'POST':
        # Form was submitted
        form = EmailPostForm(request.POST)
        if form.is_valid():
            # Form fields passed validation
            cd = form.cleaned_data
            post_url = request.build_absolute_uri(post.get_absolute_url())
            subject = '{}({})recommends you read "{}"'.format(cd['name'],
                                                              cd['email'],
                                                              post.title)
            message = 'read"{}" at {} \n\n\'s comments:{}'.format(post.title,
                                                                  post_url,
                                                                  cd['name'],
                                                                  cd[
                                                                      'comments'])
            send_mail(subject, message, cd['email'], [cd['to']])
            sent = True
            # ... send email
    else:
        form = EmailPostForm()
    return render(request, 'blog/post/share.html',
                  {'post': post, 'form': form, 'sent': sent})

```

让我们看下我们为视图添加的内容。我们使用post_detial视图来展示文章和它的评论。我们添加了一个queryset来获取这篇文章所有处于激活状态的评论：

```python
comments = post.comments.filter(active=True)
```

这个queryset以post对象开始，comments是我们在Comment模型中的post多对一关系字段中使用related_name定义的反向查询名称。

我们还使用这个视图来实现用户添加新评论的功能。因此，如果request请求方法为GET我们使用`comment_form=CommentForm()`创建一个表单实例。如果request请求方法为POST，我们使用提交的数据实例化表单并通过表单的is_valid方法验证提交的数据。如果表单数据无效，我们使用验证错误渲染模板。如果表单数据有效，我们实现以下功能：

1. 我们通过调用save方法创建了一个新的Comment对象：

```python
new_comment = comment_form.save(commit=False)
```

`save()`方法创建了一个表单对应模型的实例并将其保存到数据库。如果使用`commit=False`调用它，那么只创建模型实例，不会将实例保存到数据库，这样我们就可以在保存之前在不影响表单的情况下手动更改模型实例。注意，只有ModelForm具有save方法，Form没有该方法。

2. 我们将当前文章绑定到刚刚创建的comment上：

   ```python
   new_comment.post = post
   ```

   通过这一步，我们指定新的评论属于这篇文章。

3. 最后，我们将新的评论保存到数据库中：

   ```python
   new_comment.save()
   ```

现在，我们的视图可以展示并且处理新的评论了。

### 为文章模板添加评论

我们已经实现了管理文章评论的功能。现在，我们需要对post_detail.html模型进行修改以实现以下功能：

- 展示文章的评论总数；
- 展示评论列表；
- 展示用户添加新评论用的表单；

首先，我们添加总评论数。打开blog_detail.html模板并在content block下添加以下代码：

```html
    {% with comments.count as total_comments %}
    <h2>
        {{ total_comments }} comment{{ total_comments|pluralize }}
    </h2>
    {% endwith %}
```

我们在模板中使用Django ORM执行query set `comments.count()`。注意，django模板语言不使用括号调用函数。`{% with %}`标签允许我们将值绑定到一个新的变量中，在endwith之前的内容都可以是使用这个新的变量。

> 注意：
>
> `{% with %}`模板标签可以避免多次查询数据库或者访问expensive方法。

我们使用pluralize模板过滤器来为comment添加复数形式，pluralize依赖于total_comments的值，如果total_comments的值不为1，则{{ total_comments|pluralize }}为's'否则为''，即可能渲染为0 comments、1comment、N comments。django提供许多的模板标签和过滤器来帮我们展示信息，我们将在第三章详细讨论模板过滤器。

现在，我们添加评论列表。在上面的代码后面添加以下代码：

```html
    {% for comment in comments %}
        <div class="comment">
            <p class="info">
                Comment {{ forloop.counter }} by {{ comment.name }}{{ comment.created }}
            </p>
                {{ comment.body|linebreaks }}
        </div>
    {% empty %}
        <p>There are no comments yet.</p>
    {% endfor %}
```

我们使用`{% for %}`标签对comments进行循环。如果comments为空，我们将展示默认信息来告诉用户暂时没有评价。我们使用`{{ forloop.counter}}`对获得comment的索引。然后我们展示提交评论的用户的姓名、时间及提交的评论内容。

最后，我们需要渲染表单来展示表单成功提交时显示的信息。在前面代码后面添加以下代码：

```html
    {% if new_comment %}
        <h2> Your comment has been added.</h2>
    {% else %}
        <h2>Add a new comment</h2>
        <form action="." method="post">
            {{ comment_form.as_p }}
            {% csrf_token %}
            <p><input type="submit" value="Add comment"></p>
        </form>
    {% endif %}
```

代码很清晰，如果存在new_comment对象，我们展示评论成功提交信息；否则，我们使用`<p>`元素渲染表单的每个字段并为表单的POST请求添加csrf token。在浏览器中打开http://127.0.0.1:8000/blog/点击任意一篇文章到文章页面，我们将看到下面的内容：

![add_comment_html](/Users/apple/profile/django_by_example/blog/doc/figures/CH2/add_comment_html.png)

使用表单添加一些评论，在文章下可以看到按创建时间排序的评论：

![comment_show](/Users/apple/profile/django_by_example/blog/doc/figures/CH2/comment_show.png)

在浏览器中打开http://127.0.0.1:8000/admin/blog/comment/ 将看到评论列表。点击其中的一个进行编辑，点击Active按钮并保存，我们将再次看到评论列表，看起来是这样的：

![comment_manage](/Users/apple/profile/django_by_example/blog/doc/figures/CH2/comment_manage.png)

如果回到文章页面，我们将发现刚刚编辑过的评论不见了，计算得到的评论数也减少了一条。这是由于我们在获取评论时过滤了active为False的评论。

## 添加标签功能

完成评论系统后我们开始对文章添加标签。我们通过在项目中引入Django的第三方库来实现标签功能。django-taggit是一个可复用的应用，它的主要功能为提供一个Tag模型和一个易于为任何模型添加标签的管理器，可以从https://github.com/alex/django-taggit找到它的源代码。

首先，我们需要通过pip安装django-taggit，在terminal中运行以下命令：

```python
pip install django-taggit
```

然后打开项目的settings.py文件并将taggit添加到INSTALLED_APPS中：

```python
INSTALLED_APPS = ['django.contrib.admin', 'django.contrib.auth',
                  'django.contrib.contenttypes', 'django.contrib.sessions',
                  'django.contrib.messages', 'django.contrib.staticfiles',
                  'blog', 'taggit']
```

打开blog应用的models.py文件，为Post模型添加django-taggit提供的TaggableManager管理器：

```python
from taggit.managers import TaggableManager


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
    tags = TaggableManager()
```

tags管理器允许我们为文章对象添加、删除标签或者获取文章的标签。

打开Terminal，在项目的根目录下执行以下命令来将tags字段同步到数据库：

```python
python manage.py makemigrations
```

我们会看到以下输出：

```python
Migrations for 'blog':
  blog/migrations/0003_post_tags.py
    - Add field tags to post
```

然后执行以下命令：

```python
python manage.py migrate
```

我们会看到以下输出：

```python
Operations to perform:
  Apply all migrations: admin, auth, blog, contenttypes, sessions, taggit
Running migrations:
  Applying taggit.0001_initial... OK
  Applying taggit.0002_auto_20150616_2121... OK
  Applying blog.0003_post_tags... OK
```

现在数据库中包含django-taggit模型了，打开terminal并运行`python manage.py shell`学习如何使用tags管理器。首先，我们选择一篇文章（id=1的文章）：

```python
In [2]: from blog.models import Post

In [3]: post = Post.objects.get(id=1)
```

然后为文章添加一些标签，然后获取这篇文章的标签以确定标签是否添加成功：

```python
In [4]: post.tags.add('music','jazz','django')

In [5]: post.tags.all()
Out[5]: <QuerySet [<Tag: jazz>, <Tag: music>, <Tag: django>]>
```

最后，删除一个标签，并再次查看文章的所有标签：

```python
In [6]: post.tags.remove('django')

In [7]: post.tags.all()
Out[7]: <QuerySet [<Tag: jazz>, <Tag: music>]>
```

运行python manage.py runserver来运行开发服务器。在浏览器中打开http://127.0.0.1:8000/admin/taggit/tag，你将看到admin网站上taggit应用的Tag对象列表：

![tag_admin](/Users/apple/profile/django_by_example/blog/doc/figures/CH2/tag_admin.png)

跳转到http://127.0.0.1:8000/admin/blog/post/，点击任意一个文章对象，你将看到文章对象现在包含一个这样的Tags字段：



![tag_field](/Users/apple/profile/django_by_example/blog/doc/figures/CH2/tag_field.png)

现在，我们编辑blog 文章来展示tags。打开blog/post/list.html模板并在文章标题下添加以下代码：

```html
        <p class="tags">Tags:{{ post.tags.all|join:"," }}</p>
```

模板过滤器join与python的join函数功能类似，都是用来连接给定的字符串的。打开http://127.0.0.1:8000/blog/，我们可以看到文章标题下添加了tag：



![tag_post](/Users/apple/profile/django_by_example/blog/doc/figures/CH2/tag_post.png)

现在我们来编辑post_list视图以允许用户找到特定标签的所有文章。打开blog应用的views.py文件，将post_list函数更改为：

```python
from taggit.models import Tag


def post_list(request, tag_slug=None):
    object_list = Post.objects.all()
    tag = None
    if tag_slug:
        tag = get_object_or_404(Tag, slug=tag_slug)
        object_list = object_list.filter(tags__in=[tag])

    paginator = Paginator(object_list, 1)  # 1 posts in each page
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

这个视图时这样工作的：

1. 视图输入可选的tag_slug参数，该参数的默认值为None，这个参数来自于URL。
2. 在视图中，我们创建了初始queryset来获得所有发布的文章，如果URL传入了有效地tag_slug，我们使用get_object_or_404获得满足slug要求的Tag对象（Tag模型中的slug应该是唯一的）；
3. 根据步骤2中得到的tag对文章列表进行过滤。

Queryset是惰性的。只有当我们执行循环时Queryset才会真正执行文章查询。

最后更改post_list的render()将tag传到template中。

```python
from taggit.models import Tag


def post_list(request, tag_slug=None):
    object_list = Post.objects.all()
    tag = None
    if tag_slug:
        tag = get_object_or_404(Tag, slug=tag_slug)
        object_list = object_list.filter(tags__in=[tag])

    paginator = Paginator(object_list, 1)  # 1 posts in each page
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
                  {'page': page, 'posts': posts, 'tag': tag})
```

打开应用的urls.py文件，注释掉PostListView的url并去掉post_list的url的注释，现在url是这样的：

```python
from django.conf.urls import url

from . import views
urlpatterns = [
    url(r'^$', views.post_list, name='post_list'),# post views
    # url(r'^$', views.PostListView.as_view(), name='post_list'),
    url(r'(?P<year>\d{4})/(?P<month>\d{2})/(?P<day>\d{2})/(?P<slug>[-\w]+)/$',
        views.post_detail, name='post_detail'),
    url(r'(?P<post_id>\d+)/share/$', views.post_share, name='post_share')]
```

添加以下url来实现通过tag获得post列表：

```python
url(r'^tag/(?P<tag_slug>[-\w]+)/$', views.post_list, name='post_list_by_tag'),
```

我们可以看到，两个url都指向相同的视图，但是我们为它们使用了不同的name。第一个名为post_list，它不需要传入任何参数，第二个名为post_list_by_tag，需要传入tag_slug参数。

由于我们使用的是post_list视图，编辑blog/post/list.html：

```html
{% extends "blog/base.html" %}

{% block title %}My Blog{% endblock %}

{% block content %}
    <h1>My Blog</h1>
    {% if tag %}
        <h2>Posts tagged with "{{ tag.name }}"</h2>
    {% endif %}
    
    {% for post in posts %}
        <h2>
            <a href="{{ post.get_absolute_url }}">
                {{ post.title }}
            </a>
        </h2>
        <p class="tags">Tags:{{ post.tags.all|join:"," }}</p>
        <p class="date">
            Published {{ post.publish }} by {{ post.author }}
        </p>
        {{ post.body|truncatewords:30|linebreaks }}
    {% endfor %}
    {% include "blog/pagination.html" with page=posts %}
{% endblock %}
```

如果用户访问blog，他将看到所有文章列表，如果他通过某个特定tag进行过滤，他将看到`<h2>Posts tagged with "{{ tag.name }}"</h2>`，现在更改文章标题下的tags，即list.html改为：

```html
{% extends "blog/base.html" %}

{% block title %}My Blog{% endblock %}

{% block content %}
    <h1>My Blog</h1>
    {% if tag %}
        <h2>Posts tagged with "{{ tag.name }}"</h2>
    {% endif %}

    {% for post in posts %}
        <h2>
            <a href="{{ post.get_absolute_url }}">
                {{ post.title }}
            </a>
        </h2>
        <p class="tags">Tags:
            {% for tag in post.tags.all %}
                <a href="{% url 'blog:post_list_by_tag' tag.slug %}">
                    {{ tag.name }}
                </a>
                {% if not forloop.last %},{% endif %}
            {% endfor %}
        </p>
        <p class="date">
            Published {{ post.publish }} by {{ post.author }}
        </p>
        {{ post.body|truncatewords:30|linebreaks }}
    {% endfor %}
    {% include "blog/pagination.html" with page=posts %}
{% endblock %}
```

现在我们以自定义链接的形式展示文章的标签，链接通过{% url 'blog:post_list_by_tag' tag.slug %}实现，然后通过逗号连接标签。

在浏览器中打开http://127.0.0.1:8000/blog/并点击任意标签，我们将看到：

![tag_list](/Users/apple/profile/django_by_example/blog/doc/figures/CH2/tag_list.png)

## 查找同类文章

现在，我们已经为blog文章添加了标签，现在我们可以通过它实现很多好玩的功能。通过标签，我们可以很好的对文章进行分类。相同主题的文章可能会有几个共同的标签，我们将创建一个函数来展示拥有共同标签的文章。这样，当用户阅读文章时，我们将建议他们阅读其他相关文章。

为了获取某一文章的同类文章，我们需要:

- 获取当前文章的所有标签；
- 获得所有具备这些标签的文章；
- 从得到的列表中排除当前文章以避免推荐同一文章；
- 通过与该文章共用标签的数量从多到少对文章进行排序；
- 对于共用标签数目相同的文章，首先推荐最新的文章；
- 控制我们要推荐文章的总数。

这些步骤可以转化为post_detail视图中的一个复杂的queryset。打开blog应用的views.py文件，并在文件的最上面添加以下内容：

```python
form django.db.models import Count
```

这是django ORM的聚合函数Count，这个函数运行我们实现聚合计数。然后再post_detail函数的render之前添加以下代码：

```python
# list of similar posts
    post_tags_ids = post.tags.values_list('id', flat=True)
    similar_posts = Post.published.filter(tags__in=post_tags_ids).exclude(
        id=post.id)
    similar_posts = similar_posts.annotate(same_tags=Count('tags')).order_by(
        '-same_tags', '-publish')[:4]
```

通过执行以下命令可以打印similar_post执行的SQL语句:

```python
print(similar_posts.query)
```

对应的SQL语句为:

```SQL
SELECT "blog_post"."id", "blog_post"."title", "blog_post"."slug", "blog_post"."author_id", "blog_post"."body", "blog_post"."publish", "blog_post"."created", "blog_post"."updated", "blog_post"."status", COUNT("taggit_taggeditem"."tag_id") AS "same_tags" 

FROM "blog_post" INNER JOIN "taggit_taggeditem" ON ("blog_post"."id" = "taggit_taggeditem"."object_id" AND ("taggit_taggeditem"."content_type_id" = 7)) 

WHERE ("taggit_taggeditem"."tag_id" IN (SELECT DISTINCT U0."id" AS Col1 FROM "taggit_tag" U0 INNER JOIN "taggit_taggeditem" U1 ON (U0."id" = U1."tag_id") WHERE (U1."object_id" = 1 AND U1."content_type_id" = 7)) AND NOT ("blog_post"."id" = 1)) GROUP BY "blog_post"."id", "blog_post"."title", "blog_post"."slug", "blog_post"."author_id", "blog_post"."body", "blog_post"."publish", "blog_post"."created", "blog_post"."updated", "blog_post"."status" 
ORDER BY "same_tags" DESC, "blog_post"."publish" DESC LIMIT 4
```

这些代码的功能为:

1. 获取当前文章所有tag的id，queryset的values_list()返回指定字段的元组列表，我们为其传入flat=True以直接获得字段值列表（而不是元组列表）[1,2,3…]；
2. 获得除当前文章以外的所有具备这些标签的文章；
3. 通过聚合函数Count得到一个计算字段same_tags，该字段为包含相同标签的数量；
4. 按照共有标签数量的降序及相同标签时发布时间的降序对文章进行排序，然后截取前4篇文章。

将similar_posts对象添加到下面的render()函数中:

```python
render(request, 'blog/post/detail.html',
                  {'post': post, 'comments': comments,
                   'comment_form': comment_form,
                   'similar_posts':similar_posts})
```

现在，编辑blog/post/detail.py模板并在文章评论之前添加以下代码：

```python
    <h2>Similar posts</h2>
    {% for post in similar_posts %}
        <p>
            <a href="{{ post.get_absolute_url }}">{{ post.title }}</a>
        </p>
    {% empty %}
        <p>There are no similar posts yet.</p>
    {% endfor %}
```

我们也可以按照文章列表添加标签的方式为文章添加标签，即整个html最终为：

```python
{% extends "blog/base.html" %}

{% block title %}{{ post.title }}{% endblock %}

{% block content %}
    <h1>{{ post.title }}</h1>
    <p class="date">
        Published {{ post.publish }} by {{ post.author }}
    </p>
    <p class="tags">Tags:
            {% for tag in post.tags.all %}
                <a href="{% url 'blog:post_list_by_tag' tag.slug %}">
                    {{ tag.name }}
                </a>
                {% if not forloop.last %},{% endif %}
            {% endfor %}
        </p>
    {{ post.body|linebreaks }}
    <p>
        <a href="{% url "blog:post_share" post.id %}">
            Share this post
        </a>
    </p>
    <h2>Similar posts</h2>
    {% for post in similar_posts %}
        <p>
            <a href="{{ post.get_absolute_url }}">{{ post.title }}</a>
        </p>
    {% empty %}
        <p>There are no similar posts yet.</p>
    {% endfor %}


    {% with comments.count as total_comments %}
        <h2>
            {{ total_comments }} comment{{ total_comments|pluralize }}
        </h2>
    {% endwith %}

    {% for comment in comments %}
        <div class="comment">
            <p class="info">
                Comment {{ forloop.counter }}
                by {{ comment.name }} {{ comment.created }}
            </p>
            {{ comment.body|linebreaks }}
        </div>
    {% empty %}
        <p>There are no comments yet.</p>
    {% endfor %}

    {% if new_comment %}
        <h2> Your comment has been added.</h2>
    {% else %}
        <h2>Add a new comment</h2>
        <form action="." method="post">
            {{ comment_form.as_p }}
            {% csrf_token %}
            <p><input type="submit" value="Add comment"></p>
        </form>
    {% endif %}
{% endblock %}
```

现在的文章页面看起来是这样的：

![similar](/Users/apple/profile/django_by_example/blog/doc/figures/CH2/similar.png)

你可以成功的向你的用户推荐相似文章了，django-taggit也包含一个similar_object()的管理器来帮助我们获取具有相同标签的文章列表，我们可以在下面链接中找到django-taggit的所有管理器：http://django-taggit.readthedocs.io/en/latest/api.html。

## 总结

在这一章，我们学习如何使用forms和modelforms。我们实现了通过邮件分享文章和文章评论系统。我们为文章添加了标签、集成了可复用的应用并创建了复杂的queryset来查找同类文章。

在下一章中，我们将学习如何创建自定义标签及过滤器。还将实现一个自定义sitemap、为blog文章供稿以及为应用集成一个高级搜索引擎。