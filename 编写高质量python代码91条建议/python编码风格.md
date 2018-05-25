####建议1： 编写 Pythonic 代码

- 要避免劣化代码，比如不合适的变量命名等

  - 避免只用大小写来区分不同的对象

  - 避免使用容易引起混淆的名称，包括：

    - 重复使用已经存在于上下文中的变量名来表示不同的类型
    - 误用了内建名称来表示其他含义的名称而使之在当前命名空间被屏蔽
    - 没有构建新的数据类型的情况下使用类似于 `element、list、dict` 等作为变量名
    - 使用 o（字母 O 小写的形式，容易与数值 0 混淆）、1（字母 L 小写的形式，容易与数字 1 混淆）等作为变量名

  - 不要害怕过长的变量名，比如 `person_info` 比 `pi` 的可读性要强得多。

- 深入认识 Python 有助于编写 Pythonic 代码

  - 全面掌握 Python 提供给我们的所有特性，包括语言特性和库特性。其中最好的学习方式应该是通读官方手册中的 `Language Reference` 和 `Library Reference`。
  - 一方面 Python 语言推荐使用大量的惯用法来完成任务；另一方面，语言的进化又趋于更好地支持惯用法。
  - 深入学习业界公认的比较 Pythonic 的代码，比如 `Flask、gevent 和 requests`等


#### 建议 2：在代码中适当添加注释

​	Python 中有 3 种形式的代码注释：块注释、行注释以及文档注释（docstring）。这 3 种形式的惯用法大概有如下几种：

- 使用块或者行注释的时候仅注释那些复杂的操作、算法，还有可能别人难以理解的技巧或者不够一目了然的代码
- 注释和代码隔开一定的距离，同时在块注释之后最好多留几行空白再写代码。
- 给外部可访问的函数和方法添加文档注释。注释要清楚地描述方法的功能，并对参数、返回值以及可能发生的异常进行说明，使得外部调用它的人员仅仅看 docstring 就能正确使用。（使用 `""" """` 进行注释）
- 推荐在头文件中包含 copyright 申明、模块描述等，如有必要，可以考虑加入作者信息以及变更记录
- 需要避免的注释：
  - 代码即注释（不写注释）
  - 注释与代码重复
  - 利用注释语法快速删除代码。对于不再需要的代码，应该将其删除，而不是将其注释掉。即使担心以后还会用到，版本控制工具也可以让你轻松找回被删除的代码。
  - 代码不断更新而注释却没有更新
  - 注释比代码本身还复杂繁琐
  - 将别处的注释和代码一起拷贝过来，但上下文的变更导致注释与代码不同步
  - 将注释当作自己的娱乐空间从而留下个性特征

#### 建议 3：通过适当添加空行使代码布局更为优雅、合理

Python 代码布局有一些基本规则可以遵循：

- 在一组代码表达完一个完整的思路之后，应该用空白行进行间隔。如每个函数之间，导入声明、变量赋值等。通俗点讲就是不要在一段代码中说明几件事。推荐在函数定义或者类定义之间空两行，在类定义与第一个方法之间，或者需要进行语义分割的地方空一行。
- 尽量保持上下文语义的易理解性
- 避免过长的代码行，每行最好不要超过 80 个字符。超过的部分可以用圆括号、方括号和花括号等进行行连接，并且保持行连接的元素垂直对齐。
- 不要为了保持水平对齐而使用多余的空格，其实使阅读者尽可能容易地理解代码所要表达的意义更重要。
- 空格的使用要能够在需要强调的时候警示读者，在疏松关系的实体间起到分隔作用，而在具有紧密关系的时候不要使用空格。具体细节如下：
  - 二元运算符（赋值、比较、布尔运算的左右两边应该有空格）
  - 逗号和分号前不要使用空格
  - 函数名和左括号之间、序列索引操作时序列名和 [] 之间不需要空格，函数的默认参数两侧不需要空格
  - 强调前面的操作符的时候使用空格

#### 建议4：编写函数的几个原则

​	函数能够带来最大化的代码重用和最小化的代码冗余。精心设计的函数不仅可以提高程序的健壮性，还可以增强可读性、减少维护成本。

一般来说函数设计有以下基本原则可以参考：

- 原则 1：函数设计要尽量短小，嵌套层次不宜过深。最好能控制在 3 层以内。
- 原则 2：函数申明应该做到合理、简单、易于使用。参数个数不宜太多。
- 原则 3：函数参数设计应该考虑向下兼容。比如相同功能的函数不同版本的实现，唯一不同的是在更高级的版本中添加了参数导致程序中函数调用的接口发生了改变。这并不是最佳设计，更好的方法是通过加入默认参数来避免这种退化，做到向下兼容。
- 原则 4：一个函数只做一件事，尽量保证函数语句粒度的一致性。
- 原则 5：不要在函数中定义可变对象作为默认值
- 原则 6：使用异常替换返回错误
- 原则 7：保证通过单元测试

#### 建议 5：将常量集中到一个文件

​	Python 的内建命名空间是支持一小部分常量的，例如 True、False、None 等，只是 Python 没有提供定义常量的直接方式而已。

在 Python 中应该如何使用常量？一般来说有以下两种方式：

- 通过命名风格来提醒使用者该变量代表的意义为常量，如常量名所有字母大写，用下划线连接各个单词

- 通过自定义的类实现常量功能。这要求符合”命名全部为大写“和”值一旦绑定便不可再修改“这两个条件。下面是一种较为常见的解决办法，它通过对常量对应的值进行修改时或者命名不符合规范时抛出异常来满足以上常量的两个条件。

  ```python
  class _const:
      class ConstError(TypeError):
          pass
      class ConstCaseError(ConstError):
          pass

      def __setattr__(self, name, value):
          if self.__dict__.has_key(name):
              raise self.ConstError, "Can't change const.{}".format(name)
          if not name.isupper():
              raise self.ConstCaseError, "const name {} is not all uppercase".format(name)
          self.__dict__[name] = value

  import sys
  sys.modules[__name__] = _const()
  ```

  如果上面的代码对应的模块名为 const，使用时候只需要 import const，便可以直接定义常量了，如以下代码：

  ```python
  import const
  const.COMPANY = "IBM"
  ```

- 无论采用哪一种方式来实现常量，都提倡将常量集中到一个文件中，因为这样有利于维护，一旦需要修改常量的值，可以集中统一而不是逐个文件去检查。采用第二种方式实现的常量可以这么做：将存放常量的文件命名为 `constant.py` ，并在其中定义一系列的常量。

  ```python
  class _const:
      [...]

  import sys
  sys.modules[__name__] = _const()
  import const
  const.MY_CONSTANT = 1
  const.MY_SECOND_CONSTANT = 2
  ```

  当在其他模块中引用这些常量时，按照如下方式进行即可：

  ```python
  from constant import const
  print(const.MY_SECOND_CONSTANT)
  ```

  **最后**，也可以尝试利用工具达到事半功倍的效果。比如风格检查程序 PEP8。其实一开始 PEP 8 是一篇关于 Python 编码风格的指南，它提出了保持代码一致性的细节要求。它至少包括了对代码布局、注释、命名规范等方面的要求，在代码中遵循这些原则，有利于编写 Pythonic 的代码。

  - 应用程序 PEP 8 可以用来进行检测，它是使用 Python 开发的，安装的方法：`pip install -U pep8`

  - 然后即可简单地用它检测一下自己的代码：`pep8 --first optparse.py`，如果嫌报表不够细致，可以考虑使用 `--show-source` 参数让 `PEP8` 显示每一个错误和警告对应的代码。

  - 比如，对代码的换行，不好的风格如下：

    ```
    if foo == "blah": do_blah_thing()
    do_one(); do_two(); do_three()
    ```

    而 Pythonic 的风格则是这样的：

    ```
    if foo == "blah":
        do_blah_thing()
    do_one()
    do_two()
    do_three()
    ```

  - `PEP8` 程序，甚至还可以给出“正确”的写法，除了针对某一个源代码文件以外，还可以直接检测一个项目的质量，并通过直观的报表给出报告。

  - `PEP8` 有优秀的插件架构，可以方便地实现特定风格的检测；它生成的报告易于处理，可以很方便地与编辑器集成。

  - `PEP8` 不是唯一的编程规范。有些公司制定的编程规范也非常有参考意义，比如 `Google Python Style Guide`。同样，`PEP8` 也不是唯一的风格检测程序，类似的应用还有 `Pychecker、Pylint、Pyflakes` 等。其中 `Pychecker` 是 `Google Python Style Guide` 推荐的工具；`Pylint` 因可以非常方便地通过编辑配置文件实现公司或团队的风格检测而受到许多人的青睐；`Pyflakes` 则因为易于集成到 vim 中，所以使用的人也非常多。

  ​

  #### End：看看Flask中的代码风格

  ```python
    def run(self, host=None, port=None, debug=None, **options):
          """Runs the application on a local development server.

          Do not use ``run()`` in a production setting. It is not intended to
          meet security and performance requirements for a production server.
          Instead, see :ref:`deployment` for WSGI server recommendations.

          If the :attr:`debug` flag is set the server will automatically reload
          for code changes and show a debugger in case an exception happened.

          If you want to run the application in debug mode, but disable the
          code execution on the interactive debugger, you can pass
          ``use_evalex=False`` as parameter.  This will keep the debugger's
          traceback screen active, but disable code execution.

          It is not recommended to use this function for development with
          automatic reloading as this is badly supported.  Instead you should
          be using the :command:`flask` command line script's ``run`` support.

          .. admonition:: Keep in Mind

             Flask will suppress any server error with a generic error page
             unless it is in debug mode.  As such to enable just the
             interactive debugger without the code reloading, you have to
             invoke :meth:`run` with ``debug=True`` and ``use_reloader=False``.
             Setting ``use_debugger`` to ``True`` without being in debug mode
             won't catch any exceptions because there won't be any to
             catch.

          .. versionchanged:: 0.10
             The default port is now picked from the ``SERVER_NAME`` variable.

          :param host: the hostname to listen on. Set this to ``'0.0.0.0'`` to
                       have the server available externally as well. Defaults to
                       ``'127.0.0.1'``.
          :param port: the port of the webserver. Defaults to ``5000`` or the
                       port defined in the ``SERVER_NAME`` config variable if
                       present.
          :param debug: if given, enable or disable debug mode.
                        See :attr:`debug`.
          :param options: the options to be forwarded to the underlying
                          Werkzeug server.  See
                          :func:`werkzeug.serving.run_simple` for more
                          information.
          """
          from werkzeug.serving import run_simple
          if host is None:
              host = '127.0.0.1'
          if port is None:
              server_name = self.config['SERVER_NAME']
              if server_name and ':' in server_name:
                  port = int(server_name.rsplit(':', 1)[1])
              else:
                  port = 5000
          if debug is not None:
              self.debug = bool(debug)
          options.setdefault('use_reloader', self.debug)
          options.setdefault('use_debugger', self.debug)
          try:
              run_simple(host, port, self, **options)
          finally:
              # reset the first request information if the development server
              # reset normally.  This makes it possible to restart the server
              # without reloader and that stuff from an interactive shell.
              self._got_first_request = False
  ```

  ​