Python 2和Python 3之间存在着较大的差异，并且，由于各种原因导致了Python 2和Python 3的长期共存。在实际工作过程中，我们可能会同时用到Python 2和Python 3，因此，也需要经常在Python 2和Python 3之间进行来回切换。此外，如果你是喜欢尝鲜的人，那么，你很有可能在Python的新版本出来的时候，立即下载Python的最新版本，试验Python的最新特性。

在Python世界里，除了需要对Python的版本进行管理以外，还需要对不同的软件包进行管理。大部分情况下，对于开源的库我们使用最新版本即可。但是，有时候可能需要对相同的Python版本，在不同的项目中使用不同版本的软件包。

在这篇文章中，我们将介绍两个工具，即pyenv和virtualenv。前者用于管理不同的Python版本，后者用于管理不同的工作环境。有了这两个工具，Python相关的版本问题将不再是问题。

## 1 使用pyenv管理不同的Python版本

安装不同的Python版本并不是一件容易的事情，在不同的Python版本之间来回切换更加困难，而且，多版本并存非常容易互相干扰。因此，我们需要一个名为pyenv的工具。pyenv是一个Python版本管理工具，它能够进行全局的Python版本切换，也可以为单个项目提供对应的Python版本。使用pyenv以后，可以在服务器上安装多个不同的Python版本，也可以安装不同的Python实现。不同Python版本之间的切换也非常简单。接下来我们就一起看一下pyenv的安装和使用。

pyenv/pyenv

我们直接从github上clone项目到本地，然后，分别执行以下命令进行安装即可：

```
$ git clone https://github.com/yyuu/pyenv.git ~/.pyenv
Cloning into '/home/lmx/.pyenv'...
remote: Counting objects: 14458, done.
remote: Compressing objects: 100% (8/8), done.
Receiving objects: 100% (14458/14458), 2.58 MiB | 541 KiB/s, done.
remote: Total 14458 (delta 1), reused 0 (delta 0), pack-reused 14449
Resolving deltas: 100% (9938/9938), done.

$ echo 'export PYENV_ROOT="$HOME/.pyenv"' >> ~/.bash_profile
$ echo 'export PATH="$PYENV_ROOT/bin:$PATH"' >> ~/.bash_profile
$ echo 'eval "$(pyenv init -)"' >> ~/.bash_profile
```

安装完成以后，需要重新载入配置文件，或者退出以后重新登录，以使~/.bash_profile中的配置生效。笔者一般选择使用source命令重新载入配置文件，如下所示：

```
$ source ~/.bash_profile
```

至此，pyenv就安装完成了，我们可以通过下面的命令验证pyenv是否正确安装以及获取pyenv的帮助信息：

```
$ pyenv --help
Usage: pyenv <command> [<args>]

Some useful pyenv commands are:
   commands    List all available pyenv commands
   local       Set or show the local application-specific Python version
   global      Set or show the global Python version
   shell       Set or show the shell-specific Python version
   install     Install a Python version using python-build
   uninstall   Uninstall a specific Python version
   rehash      Rehash pyenv shims (run this after installing executables)
   version     Show the current Python version and its origin
   versions    List all Python versions available to pyenv
   which       Display the full path to an executable
   whence      List all Python versions that contain the given executable

See `pyenv help <command>' for information on a specific command.
For full documentation, see: https://github.com/yyuu/pyenv#readme
```

**使用pyenv**

我们通过pyenv的install命令，可以查看pyenv当前支持哪些Python版本，如下所示：

```
pyenv  install --list
Available versions:
  3.6.0
  3.6-dev
  3.7-dev
  ...
```

由于pyenv可以安装的Python版本列表非常长，所以，这里进行了省略。读者可以在自己电脑上安装pyenv，然后执行pyenv install --list命令。可以看到，pyenv不但可以安装不同的Python版本，而且还可以安装不同的Python实现，也可以安装最新版本的Python用以学习。

使用pyenv安装不同的Python版本：

```
pyenv install -v 3.6.0
pyenv install -v 2.7.13
```

查看当前系统中包含的Python版本：

```
$ pyenv versions
* system (set by /home/lmx/.pyenv/version)
  2.7.13
  3.6.0
```

由于我们安装了2个Python版本，加上我们系统自身的Python，当前系统中存在3个不同的Python版本。其中，输出结果前面的"*"表示当前正在使用的版本。我们也可以通过pyenv global选择不同的Python版本，如下所示：

```
$ pyenv global 3.6.0
$ pyenv versions
  system
  2.7.13
* 3.6.0 (set by /home/lmx/.pyenv/version)

$ python
Python 3.6.0 (default, Feb  8 2017, 15:53:33)
[GCC 4.7.2] on linux
Type "help", "copyright", "credits" or "license" for more information.
>>> exit()
$ pyenv global 2.7.13
$ python
Python 2.7.13 (default, Feb  8 2017, 16:03:42)
[GCC 4.7.2] on linux2
Type "help", "copyright", "credits" or "license" for more information.
```

使用pyenv以后，可以快速地切换Python的版本。切换Python版本以后，与版本相关的依赖也会一起切换。因此，我们不用担心不同的版本在系统中是否会相互干扰。例如，切换Python版本以后，相应的pip也会跟着切换，所以不用担心自己用pip版本和Python版本不匹配的问题，如下所示：

```
$ pyenv global 3.6.0
$ pip --version
pip 9.0.1 from /home/lmx/.pyenv/versions/3.6.0/lib/python3.6/site-packages (python 3.6)
$ pyenv global 2.7.13
$ pip --version
pip 9.0.1 from /home/lmx/.pyenv/versions/2.7.13/lib/python2.7/site-packages (python 2.7)
```

如果想要删除Python版本，则使用uninstall命令。如下所示：

```
pyenv uninstall 2.7.10
```

### 2 使用virtualenv管理不同的项目

virtualenv本身是一个独立的项目，用以隔离不同项目的工作环境。例如，用户lmx希望在项目A中使用Flask 0.8这个版本，与此同时，又想在项目B中使用Flask 0.9这个版本。如果我们全局安装Flask，必然无法满足用户的需求。这个时候，我们就可以使用virtualenv。

读者需要注意pyenv和virtualenv的区别。pyenv用以管理不同的Python版本，例如，你的系统工作时使用Python 2.7.13，学习时使用Python 3.6.0。virtualenv用以隔离项目的工作环境，例如，项目A和项目B都是使用Python 2.7.13，但是，项目A需要使用Flask 0.8版本，项目B需要使用Flask 0.9版本。我们只要组合pyenv和virtualenv这两个工具，就能够构造Python和第三方库的任意版本组合，拥有了很好的灵活性，也避免了项目之间的相互干扰。

virtualenv本身是一个独立的工具，用户可以不使用pyenv单独使用virtualenv。但是，如果你使用了pyenv，就需要安装pyenv-virtualenv插件而不是virtualenv软件来使用virtualenv的功能。

pyenv/pyenv-virtualenv

安装和使用pyenv-virtualenv插件如下所示：

```shell
$ git clone https://github.com/yyuu/pyenv-virtualenv.git $(pyenv root)/plugins/pyenv-virtualenv
Cloning into '/home/lmx/.pyenv/plugins/pyenv-virtualenv'...
remote: Counting objects: 1860, done.
remote: Total 1860 (delta 0), reused 0 (delta 0), pack-reused 1860
Receiving objects: 100% (1860/1860), 530.62 KiB | 213 KiB/s, done.
Resolving deltas: 100% (1274/1274), done.
$ echo 'eval "$(pyenv virtualenv-init -)"' >> ~/.bash_profile
```

与安装pyenv类似，安装完成以后需要重新载入配置文件，或者退出用户再登录，以使得配置文件生效：

```shell
$ source  ~/.bash_profile
$ pyenv help virtualenv
Usage: pyenv virtualenv [-f|--force] [VIRTUALENV_OPTIONS] [version] <virtualenv-name>
       pyenv virtualenv --version
       pyenv virtualenv --help

  -f/--force       Install even if the version appears to be installed already
```

**pyenv-virtualenv使用**

有了pyenv-virtualenv以后，我们可以为同一个Python解释器，创建多个不同的"工作环境"。例如，我们新建两个工作环境：

```shell
$ pyenv virtualenv 2.7.13 first_project
$ pyenv virtualenv 2.7.13 second_project
```

可以使用virtualenvs子命令查看工作环境：

```
$ pyenv virtualenvs
  2.7.13/envs/first_project (created from /home/lmx/.pyenv/versions/2.7.13)
  2.7.13/envs/second_project (created from /home/lmx/.pyenv/versions/2.7.13)
  first_project (created from /home/lmx/.pyenv/versions/2.7.13)
  second_project (created from /home/lmx/.pyenv/versions/2.7.13)
```

创建完工作环境以后，可以通过activate和deactivate子命令进入或退出一个工作环境。进入工作环境以后，左边的提示符会显示你当前所在的工作环境，以免因为环境太多导致误操作。

```
$ pyenv activate first_project
(first_project) $ pip install flask==0.8
(first_project) $ pyenv deactivate
```

接下来，我们看一下在不同的工作环境安装不同的Flask版本：

```
$ pyenv activate first_project
(first_project) $ pip install flask==0.8
(first_project) $ pyenv deactivate
$ pyenv activate second_project
(second_project) $ pip install flask==0.9
```

如果想要删除虚拟环境，则使用：

```
pyenv virtualenv-delete first_project
```

使用pyenv和python-virtualenv插件，我们就能够自由地在不同的版本之间进行切换，相比管理Python版本，不但节省了时间，也避免了工作过程中的相互干扰。