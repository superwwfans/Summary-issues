​	flask中有时要对一个路由进行保护, 只有登录的用户才能访问这个路由, 在不行视图函数的本省结构上 , 使用装饰器很大的方便我们实现这个问题, 并且还可以复用.

​	路由保护是当用户访问被保护的路由是会重定向到登录页面, 登录成功后返回访问被保护的路由.

​	首先在请求到达之前判断用户登录状态,从session中回去用户唯一标识, 再在数据中查询, 得到结果放在g中,

```python
@app.before_request
def get_user_name():
    uid = session.get('uid', None)
    user = User.query.filter_by(id=uid).first()
    g.user = user
```

​	自定义装饰器, 判断g中是否有用户标识, 没有则重定向到登录页面,此时将被保护路由的端点做为url查询字符串,使用`request.endpoint` 获取被保护路由的端点;有则执行被保护的路由视图函数.

```python
from flask import redirect, g, url_for, request
from functools import wraps

def login_before_request(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        if g.user:
            return func(*args, **kwargs)
        return redirect(url_for('login', next=request.endpoint))
    return wrapper

```

在重定向到登录页面是 获取到url查询字符串(被保护路由的端点), 将其渲染到登录模板的表单中, 在提交表单时从表单中获取这个端点,使用url_for()构造出url重定向到保护的路由.

```python
@app.route('/login/', methods=["GET", "POst"])
def login():  # 处理用户登录
    form = LoginForm()
    next_endpoint = request.args.get('next', 'index')
    print(next_endpoint, )
    if form.validate_on_submit():
        user = User.query.filter_by(email=form.email.data).first()
        if user and user.check_password(form.password.data):  # 验证邮箱,用户,密码一致
            session['uid'] = user.id
            if form.remember:
                session.permanent = True
            return redirect(request.form.get('next_endpoint'))
        flash('用户名或密码不正确!')
        return render_template('login.html', form=form)
    return render_template('login.html', form=form, next_endpoint=next_endpoint)

```

当被保护的路由带转换器时, 视图函数要传一个必备参数.  请求url应为这样`<http://127.0.0.1/confirm/eyJleHAiOjE1MDgzMzM4MjEsImFsZyI6IkhTMjU2IiwiaWF0IjoxNTA4MzMwMjIxfQ.eyJjb25maXJtIjoiODdmMDhkMjQtYjM2MS00MGZjLTgwZTItMTA4YWZmNTZjOGQ1In0.XiIhdGJmCILBA1ghv5ZVpKEz_RmmjU9_f3TJIlLyj4M/>` ,

```python
@app.route('/confirm/<token>/')
@login_before_request
def confirm(token):   # 确认注册用户
    user = User.query.filter_by(id=session.get('uid')).first()
    if user.confirmed:
        return redirect(url_for('index'))
    if user.confirm(token):
        print('bbbbb')
        flash('你的账户已经确认,谢谢!')
    else:
        flash('你的确认链接已经过期或不可用!!!')
    return redirect(url_for('index'))

```

 然而url后面的参数要传到视图函数中, 装饰器应是这样的:

```python

from flask import redirect, g, url_for, request
from functools import wraps

def login_before_request(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        if g.user:
            return func(*args, **kwargs)
        if kwargs:
            token = kwargs['token']
            return redirect(url_for('login', next=request.endpoint, token=token))
        return redirect(url_for('login', next=request.url))
    return wrapper
  
注意:
  *args 可以接受一个任意长度参数, **kwargs 可以接受一个字典
  所以wrapper 可以接受任意参数, 那么就可以给被装饰的函数传任意参数
  
```

此处涉及装饰器的几个知识:

​	函数也是对象，它有`__name__`等属性，但你去看经过decorator装饰之后的函数，它们的`__name__`已经从原来的`'confirm'`变成了`'wrapper'`：

​	因为返回的那个`wrapper()`函数名字就是`'wrapper'`，所以，需要把原始函数的`__name__`等属性复制到`wrapper()`函数中，否则，有些依赖函数签名的代码执行就会出错。

​	不需要编写`wrapper.__name__ = func.__name__`这样的代码，Python内置的`functools.wraps`就是干这个事的，所以，一个完整的decorator的写法如下：

```python
from flask import redirect, g, url_for, request
from functools import wraps

def login_before_request(func):
    @wraps(func)
    def wrapper(*args, **kwargs):
        if g.user:
            return func(*args, **kwargs)
        return redirect(url_for('login', next=request.endpoint))
    return wrapper
  
  注意: 如果不加@wraps(func) request.endpoint 获取的结果始终是wrapper
```

