## 路由Routing

​	客户端把请求发送给web服务器,web服务器在把请求发给Flask程序实例,程序实例需要知道对每个URL请求运行哪些代码, 所以保存一个URL到Python函数的映射关系.

​	处理URL和函数之间关系的程序称为路由.URL是去除掉域名剩下带的部分,URL总是从`/`开始

​	在FLask中有装饰器route()将一个函数绑定到一个URL. 

​	

```python
@app.route('/') 
def index():
    return 'Index Page'

@app.route('/hello')
def hello():
    return 'Hello, World'

#上面两种定义方法和下面等同

def index():
    return 'Index Page'
app.add_url_rule('/')

def hello():
    return 'Hello, World'
app.add_url_rule('/hello')
#以下是route源码
    def route(self, rule, **options):
        """ 这个装饰器用来给给定的URL规则注册一个视图函数,它的用法和add_url_rule()一样"""
        def decorator(f):
            endpoint = options.pop('endpoint', None)
            self.add_url_rule(rule, endpoint, f, **options) #实际上route调用了add_url_rule()方法
            return f
        return decorator

    #add_url_rule源码
	    def add_url_rule(self, rule, endpoint=None, view_func=None, **options):
        """Connects a URL rule.  Works exactly like the :meth:`route`
        decorator.  If a view_func is provided it will be registered with the
        endpoint.

        Basically this example::

            @app.route('/')
            def index():
                pass

        Is equivalent to the following::

            def index():
                pass
            app.add_url_rule('/', 'index', index)

        If the view_func is not provided you will need to connect the endpoint
        to a view function like so::

            app.view_functions['index'] = index

        Internally :meth:`route` invokes :meth:`add_url_rule` so if you want
        to customize the behavior via subclassing you only need to change
        this method.

        For more information refer to :ref:`url-route-registrations`.

        .. versionchanged:: 0.2
           `view_func` parameter added.

        .. versionchanged:: 0.6
           ``OPTIONS`` is added automatically as method.

        :param rule: the URL rule as string
        :param endpoint: the endpoint for the registered URL rule.  Flask
                         itself assumes the name of the view function as
                         endpoint
        :param view_func: the function to call when serving a request to the
                          provided endpoint
        :param options: the options to be forwarded to the underlying
                        :class:`~werkzeug.routing.Rule` object.  A change
                        to Werkzeug is handling of method options.  methods
                        is a list of methods this rule should be limited
                        to (``GET``, ``POST`` etc.).  By default a rule
                        just listens for ``GET`` (and implicitly ``HEAD``).
                        Starting with Flask 0.6, ``OPTIONS`` is implicitly
                        added and handled by the standard request handling.
        """
        if endpoint is None:
            endpoint = _endpoint_from_view_func(view_func)
        options['endpoint'] = endpoint
        methods = options.pop('methods', None)

        # if the methods are not given and the view_func object knows its
        # methods we can use that instead.  If neither exists, we go with
        # a tuple of only ``GET`` as default.
        if methods is None:
            methods = getattr(view_func, 'methods', None) or ('GET',)
        if isinstance(methods, string_types):
            raise TypeError('Allowed methods have to be iterables of strings, '
                            'for example: @app.route(..., methods=["POST"])')
        methods = set(item.upper() for item in methods)

        # Methods that should always be added
        required_methods = set(getattr(view_func, 'required_methods', ()))

        # starting with Flask 0.8 the view_func object can disable and
        # force-enable the automatic options handling.
        provide_automatic_options = getattr(view_func,
            'provide_automatic_options', None)

        if provide_automatic_options is None:
            if 'OPTIONS' not in methods:
                provide_automatic_options = True
                required_methods.add('OPTIONS')
            else:
                provide_automatic_options = False

        # Add the required methods now.
        methods |= required_methods

        rule = self.url_rule_class(rule, methods=methods, **options)
        rule.provide_automatic_options = provide_automatic_options

        self.url_map.add(rule)
        if view_func is not None:
            old_func = self.view_functions.get(endpoint)
            if old_func is not None and old_func != view_func:
                raise AssertionError('View function mapping is overwriting an '
                                     'existing endpoint function: %s' % endpoint)
            self.view_functions[endpoint] = view_func
            
```

#### 动态路由 Variable Rules

​	除了指定固定的URL,我们还可以创建动态的URL,在URL中添加`<variable_name>`,这样的部分作为关键字参数传递给函数,可选的一个转换器可以被用来指定一个规则用`<converter:variable_name >`。这里有一些很好的例子:

```python
@app.route('/user/<username>')
def show_user_profile(username):
    return 'User %s' % username

@app.route('/post/<int:post_id>')
def show_post(post_id):
    return 'Post %d' % post_id
```

##### 动态ULR变量类型 

| string | accepts any text without a slash (the default)    默认接受任何文本类型不包括/ |
| :----: | ---------------------------------------- |
|  int   | accepts integers  接受一个整数                 |
| float  | like `int` but for floating point values    浮点数 |
|  path  | like the default but also accepts slashes  接受一个路径 |
|  any   | matches one of the items provided   指定范围中的任意一个 |
|  uuid  | accepts UUID strings   UUID字符串唯一识别码 用python  `from uuid import uuid4  uuid4()生成` |

 ##### 唯一路径/重定向动作

​	FLask的URL规则基于Werkzeug的路由模块,然而这个模块是基于Apache和早期HTTP服务器创建一个完美和唯一的URL.

```python
@app.route('/projects/')
	def projects():
	return 'The project page'

@app.route('/about')
def about():
	return 'The about page'

```

​	虽然已上面两个看起来时很相似,不同之处在于定义URL时尾部的斜线.在第一个例子中，项目端点的规范URL有一个尾部斜杠。从这个意义上说，它类似于文件系统上的文件夹。在没有尾部斜杠的情况下访问它将导致Flask重定向到带有尾部斜杠的规范URL。

​	然而，在第二种情况下，URL的定义是没有拖尾的斜杠，更像是unix类系统上的文件的路径名。使用尾斜杠访问URL将产生一个404“Not Found”错误。

​	这种行为允许相对url继续工作，即使省略了后面的斜杠，这与Apache和其他服务器的工作方式是一致的。另外，唯一URL有助于搜索引擎避免两次索引相同的页面。

#### 构造URL

​	为一个特殊的函数构造一个URL可以是用`url_for()` 函数, 这个函数接受的第一个参数是:视图函数的名字(或者端点名,即在定义路由规则时提供的的第二个参数,flask默认端点名为视图函数名); 第二个参数时一个关键字参数(关键字参数如果和URL变量名相同则作为变量的值传入,如果URL中没有提供这个参数,则url_for()中的关键字参数URL的请求参数).

```python
>>> from flask import Flask, url_for
>>> app = Flask(__name__)
>>> @app.route('/')
... def index(): pass
...
>>> @app.route('/login')
... def login(): pass
...
>>> @app.route('/user/<username>')
... def profile(username): pass
...
>>> with app.test_request_context():
...  print url_for('index')
...  print url_for('login')
...  print url_for('login', next='/')
...  print url_for('profile', username='John Doe')
...
/
/login
/login?next=/
/user/John%20Doe  #这里的%20时URL中的空格
```

​	这里使用了`test_request_context()`方法, 解释如下:这个方法告诉FLask运行正在处理一个请求,即便我们在Python shell中进行交互.

​	一下几个原因告诉你为什么在模板中不适用硬编码而是使用反转函数`url_for()` 方法构造一个URL:

 * 反转比硬编码URL更具有描述性,更重要的是,这个方法可以更方便的更变URL而不需要记住这些url的使用地方
 * 构造URL会处理特殊字符和Unicode数据,所以不用自己去处理这些问题;
 * 如果你的应用方在不在URL的根目录,比方说在/myapplication,只要使用`-url_for()` 依然可以获取正确的URL

#### HTTP请求方法

​	HHTP知道不同的URL请求方法,默认的,一个路由应答`GET ` 请求方式,但是在路由`route()` 装饰器中提供`methods`参数改变请求方式.

```python
from flask import request

@app.route('/login', methods=['GET', 'POST'])
def login():
    if request.method == 'POST':
        do_the_login()
    else:
        show_the_login_form()
```

