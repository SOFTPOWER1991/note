本文主要讲解Python中Virtualenv的使用,其中包括如下几个环节：

1. Virtualenv是什么？
2. Virtualenv的出现是为了解决什么问题？
3. 如何安装Virtualenv？
4. 如何使用Virtualenv?
5. virtualenv是创建“独立”的Python运行环境的原理是什么呢？
6. 如何管理电脑上的virtualenv创建的多个虚拟环境？

下面来看看具体的内容。

# 1. Virtualenv是什么？

> Virtualenv 是Python中的虚拟环境管理工具。

那么虚拟环境是什么呢？

> 虚拟环境是程序执行时的独立执行环境，在同一台服务器中可以创建不同的虚拟环境供不同的系统使用，项目之间的运行环境保持独立性而相互不受影响。我们可以在同一台电脑上构建出项目项目A在基于Python2的环境中运行，而项目B可以在基于Python3的环境中运行。

# 2. Virtualenv的出现是为了解决什么问题？

virtualenv为应用提供了隔离的Python运行环境，解决了不同应用间多版本的冲突问题。

例如：

如果我们要同时开发多个应用程序，那这些应用程序都会共用一个Python，就是安装在系统的Python 3。如果应用A需要jinja 2.7，而应用B需要jinja 2.6怎么办？

这种情况下，每个应用可能需要各自拥有一套“独立”的Python运行环境。virtualenv就是用来为一个应用创建一套“隔离”的Python运行环境。

# 3. 如何安装Virtualenv？

使用如下命令进行安装

> $ pip3 install virtualenv

# 4. 如何使用Virtualenv?

假定我们要开发一个新的项目，需要一套独立的Python运行环境，可以这么做：

## 第一步，创建目录：

```
zhanggeng:~$mkdir pythonV
zhanggeng:~$cd pythonV/
zhanggeng:pythonV$
```

## 第二步，创建一个独立的Python运行环境，命名为venv：

> virtualenv --no-site-packages venv

```
zhanggeng:pythonV$virtualenv --no-site-packages venv
Using base prefix '/Library/Frameworks/Python.framework/Versions/3.6'
New python executable in /Users/zhanggeng/pythonV/venv/bin/python3.6
Also creating executable in /Users/zhanggeng/pythonV/venv/bin/python
Installing setuptools, pip, wheel...done.
zhanggeng:pythonV$
zhanggeng:pythonV$ls
venv
zhanggeng:venv$ls
bin			include			lib      pip-selfcheck.json
zhanggeng:venv$
zhanggeng:venv$
```

命令virtualenv就可以创建一个独立的Python运行环境，我们还加上了参数--no-site-packages，这样，已经安装到系统Python环境中的所有第三方包都不会复制过来，这样，我们就得到了一个不带任何第三方包的“干净”的Python运行环境。

## 第三步，进入独立的venv环境目录下：

新建的Python环境被放到当前目录下的venv目录。有了venv这个Python环境，可以用source进入该环境：

```
zhanggeng:pythonV$source venv/bin/activate
(venv) zhanggeng:pythonV$
```
注意到命令提示符变了，有个(venv)前缀，表示当前环境是一个名为venv的Python环境。

## 第四步，安装各种第三方包，并运行python命令：

在venv环境下，用pip安装的包都被安装到venv这个环境下，系统Python环境不受任何影响。也就是说，venv环境是专门针对pythonV这个应用创建的。

```
(venv) zhanggeng:pythonV$pip3 install jinja2
Collecting jinja2
  Using cached Jinja2-2.9.6-py2.py3-none-any.whl
Collecting MarkupSafe>=0.23 (from jinja2)
  Using cached MarkupSafe-1.0.tar.gz
Building wheels for collected packages: MarkupSafe
  Running setup.py bdist_wheel for MarkupSafe ... done
  Stored in directory: /Users/zhanggeng/Library/Caches/pip/wheels/88/a7/30/e39a54a87bcbe25308fa3ca64e8ddc75d9b3e5afa21ee32d57
Successfully built MarkupSafe
Installing collected packages: MarkupSafe, jinja2
Successfully installed MarkupSafe-1.0 jinja2-2.9.6
(venv) zhanggeng:pythonV$python helloworld.py
python: can't open file 'helloworld.py': [Errno 2] No such file or directory
(venv) zhanggeng:pythonV$
```

## 第五步，退出当前venv环境，使用deactivate命令：

通过命令deactivate命令就可以退出当前环境，回到系统环境。现在pip或python均是在系统Python环境下执行。

```
(venv) zhanggeng:pythonV$deactivate
zhanggeng:pythonV$
```

如何删除虚拟环境呢？

> 运行：rm -rf venv/命令即可！

# 5. virtualenv是创建“独立”的Python运行环境的原理是什么呢？

把系统Python复制一份到virtualenv的环境，用命令source venv/bin/activate进入一个virtualenv环境时，virtualenv会修改相关环境变量，让命令python和pip均指向当前的virtualenv环境。

# 6. 如何管理电脑上的virtualenv创建的多个虚拟环境？

使用Virtaulenvwrapper。

##6.1 Virtaulenvwrapper 是什么？

Virtaulenvwrapper是virtualenv的扩展包，用于更方便管理虚拟环境，它可以做：

1. 将所有虚拟环境整合在一个目录下
2. 管理（新增，删除，复制）虚拟环境
3. 切换虚拟环境

相关文档：http://virtualenvwrapper.readthedocs.io/en/latest/install.html

## 6.2 安装Virtualenvwrapper



安装Virtualenvwrapper前需要virtualenv已近安装.使用命令：

> pip3 install virtualenvwrapper


```
zhanggeng:~$pip3 install virtualenvwrapper
Collecting virtualenvwrapper
  Downloading virtualenvwrapper-4.7.2.tar.gz (90kB)
    100% |████████████████████████████████| 92kB 440kB/s 
Requirement already satisfied: virtualenv in /Library/Frameworks/Python.framework/Versions/3.6/lib/python3.6/site-packages (from virtualenvwrapper)
Collecting virtualenv-clone (from virtualenvwrapper)
  Downloading virtualenv-clone-0.2.6.tar.gz
Collecting stevedore (from virtualenvwrapper)
  Downloading stevedore-1.23.0-py2.py3-none-any.whl
Collecting six>=1.9.0 (from stevedore->virtualenvwrapper)
  Downloading six-1.10.0-py2.py3-none-any.whl
Collecting pbr!=2.1.0,>=2.0.0 (from stevedore->virtualenvwrapper)
  Downloading pbr-3.0.1-py2.py3-none-any.whl (99kB)
    100% |████████████████████████████████| 102kB 1.9MB/s 
Installing collected packages: virtualenv-clone, six, pbr, stevedore, virtualenvwrapper
  Running setup.py install for virtualenv-clone ... done
  Running setup.py install for virtualenvwrapper ... done
Successfully installed pbr-3.0.1 six-1.10.0 stevedore-1.23.0 virtualenv-clone-0.2.6 virtualenvwrapper-4.7.2
zhanggeng:~$
```

## 6.3 使用

使用中我碰到了问题：

1. 设置的时候，官方文档这样说的：

```
Add three lines to your shell startup file (.bashrc, .profile, etc.) to set the location where the virtual environments should live, the location of your development project directories, and the location of the script installed with this package:

export WORKON_HOME=$HOME/.virtualenvs
export PROJECT_HOME=$HOME/Devel
source /usr/local/bin/virtualenvwrapper.sh

```

但是我的/usr/local/bin/目录下没有virtualenvwrapper.sh这个文件

在.bash_profile中放入上面几行后，终端会提示：

```
/usr/bin/python: No module named virtualenvwrapper
virtualenvwrapper.sh: There was a problem running the initialization hooks. 

If Python could not import the module virtualenvwrapper.hook_loader,
check that virtualenvwrapper has been installed for
VIRTUALENVWRAPPER_PYTHON=/usr/bin/python and that PATH is
```

然后在StackOverFlow上找，也没弄好！
有哪个小伙伴碰到过这个问题的，可以交流下！









