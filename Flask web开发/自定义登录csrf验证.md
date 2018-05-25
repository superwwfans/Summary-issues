自定义csrf_token

```python
def gen_token():
    token = str(uuid4())   # 生成令牌
    session['csrf_token'] = token   # 设置session令牌
    return token
  
app.jinja_env.globals.update({'gen_token': gen_token})  # 生成令牌的函数设置到jinja环境变量

@app.before_request
def check_token():   # 验证令牌  表单令牌和session令牌是否相同
    if request.method == "POST":
        form_token = request.form.get('csrf_token', '')
        if form_token != session.get('csrf_token'):
            abort(403, 'you are hacker')
            
            
注意:当form和session中同时获取不到返回值相同的情况
```

使用AJAX提交表单, 后台使用flask-wtf  csrfproject 时

 第一 : 路由使用装饰器去除csrf_token

```python
@csrf.exempt
@app.route("/json_submit", methods=["POST"])
def submit_handler():
    # a = request.get_json(force=True)
    app.logger.log("json_submit")
    return {}
```

第二:也可以在AJAX POST请求中使用CSRF令牌。

​	要将令牌包含到AJAX请求中，将令牌插入到某个位置的页面中;在< meta >头或生成的JavaScript中，然后设置x - csrftoken头。使用jQuery时，使用ajaxSetup钩子。

```
<meta name="csrf-token" content="{{ csrf_token() }}">
```

```javascript
var csrftoken = $('meta[name=csrf-token]').attr('content')

$.ajaxSetup({
    beforeSend: function(xhr, settings) {
        if (!/^(GET|HEAD|OPTIONS|TRACE)$/i.test(settings.type)) {
            xhr.setRequestHeader("X-CSRFToken", csrftoken)
        }
    }
})
```

