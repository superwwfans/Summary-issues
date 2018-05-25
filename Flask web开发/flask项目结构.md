#### 项目结构

​	尽力实现可扩展, 易维护, 高内聚低耦合, 代码分层MVC

    └── myprotject
      ├── app_berfore_request.py  # 请求前执行的函数
      ├── conifg.py # 总的配置文件
      ├── production_config.py  # 生产环境的配置文件
      ├── register_blueprints_list.py # 放置所有的蓝图,组成一个列表
      ├── requirements.txt # 项目依赖的python安装包名称和版本号
      ├── create_app.py # 创建实例一个app
      ├── develop_config.py #开发环境的配置
      ├── ext.py # 将一些可能出现循环导入的模块放在此模块中
      ├── flask-project.py 
      ├── manage.py # 命令行启动
      ├── libs # 库文件, 中间层的封装  模型类相关的增删查改, 将控制层和模型层分隔
      │   ├── account
      │   │   ├── account_auth_libs.py
      │   │   └── __init__.py
      │   ├── admin
      │   │   └── __init__.py
      │   └── __init__.py
      
      ├── models  # 放置模型类
      │   ├── account
      │   │   ├── account_model.py
      │   │   ├── account_user_model.py
      │   │   └── __init__.py
      │   ├── adimin
      │   │   ├── admin_user_model.py
      │   │   └── __init__.py
      │   └── __init__.py
      ├── static  # 静态文件
      │   ├── css
      │   │   ├── bootstrap.css
      │   │   ├── bootstrap.min.css
      │   │   └── bootstrap-theme.css
      │   ├── fonts
      │   │   ├── glyphicons-halflings-regular.eot
      │   │   ├── glyphicons-halflings-regular.svg
      │   │   ├── glyphicons-halflings-regular.ttf
      │   │   ├── glyphicons-halflings-regular.woff
      │   │   └── glyphicons-halflings-regular.woff2
      │   ├── imge
      │   │   ├── Blog.ico
      │   │   ├── logo2.png
      │   │   └── logo.png
      │   ├── jquery-3.2.1.js
      │   ├── js
      │   │   ├── bootstrap.js
      │   │   ├── bootstrap.min.js
      │   │   └── npm.js
      │   └── mystyle.css
      ├── templates   # 模板文件
      │   ├── account
      │   │   ├── login.html
      │   │   └── register.html
      │   ├── base.html
      │   └── index.html
      ├── utls  # 不依赖项目中的任何模块的小工具, 比如生成验证码
      │   ├── __init__.py
      │   └── send_mail.py
      └── views  # 控制层 处理逻辑和路由映射
          ├── account
          │   ├── account_auth_views.py
          │   ├── account_views.py
          │   └── __init__.py
          ├── admin
          │   └── __init__.py
          └── __init__.py