# 前言

因为在之前做实验的时候配置环境花了很多时间，而且网上的教程也良莠不齐，需要自己一一踩坑和鉴别，因此粗略地对遇到的问题和可行的方法进行一个总结。

-by Zhouxiang Fang 2022/7/14

# 教程

## 一、准备

1.PyCharm Professional（可以通过学生身份申请Professional版，要注意Community版本不支持远程连接服务器）

2.服务器地址、账号、密码、端口

## 二、PyCharm连接服务器

首先需要用其他软件，如Xshell等连接服务器，并在服务器上提前创建好项目所在的文件夹。

接着介绍如何用PyCharm连接服务器。首先如图所示打开`Configuration`。

![image-20220713131438758](pics\image-20220713131438758.png)

打开后如图所示，类别选则SFTP

![image-20220713131637742](pics\image-20220713131637742.png)

点击左上角`+`号，新建server

![image-20220713131744407](pics\image-20220713131744407.png)

接着配置新的`SSH configuration`，点击那三个小点即可

![image-20220713132307797](pics\image-20220713132307797.png)

如图填写地址、账户、密码、端口(port)等信息，点击test connection可测试连接是否成功

![image-20220713132359271](pics\image-20220713132359271.png)

完成上一步后，建议本地文件夹和远程服务器文件夹的映射。`Local path`即为本地项目文件夹，`Deployment path`即为远程服务器项目文件夹（实际上是`Root path`加上`Deployment path`才是真正的远程服务器项目文件夹，但前者填`/`即可）

![image-20220713132616574](pics\image-20220713132616574.png)

![image-20220713132750230](pics\image-20220713132750230.png)

到此为止，已经可以用PyCharm连接服务器了。可以找到PyCharm下方的Terminal，新建命令行。

![image-20220713133005326](pics\image-20220713133005326.png)

需要注意的是，在本地修改的代码需要及时上传到服务器进行更新。可以通过下图的方式手动上传/下载/更新文件（从服务器上下载文件或者上传文件到服务器)。`ctrl+s`在保存的时候也会自动更新远程服务器上的文件（但似乎不支持上传）。

![image-20220713143339702](pics\image-20220713143339702.png)

我们还需要为项目配置理解器（python interpreter），即选择某一个python环境去解析我们的项目。这一步需要创建完相应的虚拟环境后才能完成，此处暂且不表。

## 三、配置miniconda

miniconda用于管理虚拟环境，下载完相应文件（例如`Miniconda3-py38_4.12.0-Linux-x86_64.sh`）后可以打开linux命令行，按照如下指令安装

```bash
sh Miniconda3-py38_4.12.0-Linux-x86_64.sh
```

安装完后，可以看到当前目录下多出一个文件夹`miniconda3`，说明安装成功

## 四、创建虚拟环境

安装完`miniconda`后，可以按照如下指令创建新环境

```bash
conda create -n envs_name python==3.8
```

其中，`envs_name`用于指定新环境的环境名，`python==3.8`即为指定`python`版本为3.8

命令行会询问，一路按`y`即可完成环境创建。

如果一直卡在`Collecting package metadata`，无法完成创建，则大概率是网路连接不佳，需要用国内的镜像源替换默认的源。

首先打开conda的配置文件

```bash
vim ~/.condarc
```

接着将该文件的内容进行替换。

```
channels:
  - defaults
show_channel_urls: true
default_channels:
  - http://mirrors.aliyun.com/anaconda/pkgs/main
  - http://mirrors.aliyun.com/anaconda/pkgs/r
  - http://mirrors.aliyun.com/anaconda/pkgs/msys2
custom_channels:
  conda-forge: http://mirrors.aliyun.com/anaconda/cloud
  msys2: http://mirrors.aliyun.com/anaconda/cloud
  bioconda: http://mirrors.aliyun.com/anaconda/cloud
  menpo: http://mirrors.aliyun.com/anaconda/cloud
  pytorch: http://mirrors.aliyun.com/anaconda/cloud
  simpleitk: http://mirrors.aliyun.com/anaconda/cloud
```

这里用的是阿里源，如果效果不佳也可以替换成其他源（例如清华、北大等）。替换完成后执行`conda clean -i`用于清除index，保证用的是新的源下载。

重新执行`conda create -n envs_name python==3.8`就能完成环境创建。

激活环境执行如下命令

```bash
conda activate envs_name
```

![image-20220713134257226](pics\image-20220713134257226.png)

此后不论是进行库的下载还是jupyter notebook的配置，都需要在该虚拟环境中执行。

### 配置PyCharm Interpreter

现在我们已经完成了虚拟环境的创建，可以为我们的项目配置解释器了。

如图所示打开`Settings`

![image-20220713134454763](pics\image-20220713134454763.png)

点击添加Python Interpreter

![image-20220713134545937](pics\image-20220713134545937.png)

进入SSH Interpreter，选择之前写好的SSH配置文件。

![image-20220713134642070](pics\image-20220713134642070.png)

选择解释器并设置文件夹映射关系。Interpreter可以按照`/home/your_name/miniconda3/envs/envs_name/bin/python`找到。其中，`your_name`为你所在的文件夹，`envs_name`为创建的虚拟环境名字。`Sync Folder`和之前一样，即为本地项目文件夹到远程服务器文件夹的映射。

![image-20220713134858151](pics\image-20220713134858151.png)

完成后能够看到该python解释器所安装的一些包，点击apply即完成了解释器的配置。

![image-20220713135302195](pics\image-20220713135302195.png)

## 五、远程连接jupyter notebook

经过如上配置，我们已经可以通过PyCharm和远程服务器交互。但有时候我们也想使用jupyter notebook调试代码。这里介绍如何用本地的浏览器打开在服务器上启动的jupyter notebook。

1.首先进入虚拟环境。

```bash
conda activate envs_name
```

2.下载jupyter notebook，如果已经下载好就可以跳过这一步。

```
conda install jupyter notebook
```

3.生成配置文件。如果提示配置文件已经存在，那么可以选择`n`，这样就不会生成新的配置文件来覆盖原有文件。

```bash
jupyter notebook --generate-config
```

4.之后生成登录密码（如果不需要密码，则在提示你输入密码时按`Enter`跳过即可）

```bash
(hotnote) fangzhouxiang@T640:~/miniconda3/envs/hotnote/lib$ ipython
In [1]: from notebook.auth import passwd

In [2]: passwd()
Enter password: 
Verify password: 
Out[2]: 'argon2:$argon2id$v=19$m=10240,t=10,p=8$yL2vtbVjyop9AdDGDeLnDw$bzU0OdVrw/IbLEceqNwt3ySrcRXHHefBRVhQY7mpkNU'
exit()
```

简单介绍一下用到的所有指令。`ipython`用于启动jupyter notebook的python环境，`from notebook.auth import passwd`导入所需库，`passwd()`用于生成密码,`exit()`用于退出当前python环境。

5.之后，修改jupyter notebook配置文件

```bash
vim ~/.jupyter/jupyter_notebook_config.py
```

将如下代码复制到文件中

```
c.NotebookApp.ip = 'xxx'
c.NotebookApp.password = u'xxxxxx'
c.NotebookApp.open_browser = False
c.NotebookApp.port = 7777
```

![image-20220713140745024](pics\image-20220713140745024.png)

其中，`ip`设为服务器地址即可，`password`即为步骤4中生成的密码。保存文件后退出。

6.最后，在命令行启动

`jupyter notebook`

将地址粘贴到本地的浏览器即可运行。(在这里即为http://10.214.243.17:9999/)

## 六、配置jupyter notebook补全插件

jupyter notebook的便利之处在于可以分块执行代码，并且保存之前运行的结果。但本身的补全功能比较麻烦，需要按tab键才会提示补全。这里介绍如何安装nbextensions插件支持**自动**补全功能。

安装之前要先关闭jupyter notebook

1.安装nbextensions   

```bash
pip install jupyter_contrib_nbextensions 
```

2.安装nbextensions_configurator

```bash
pip install jupyter_nbextensions_configurator 
```

3.打开jupyter notebook。打开Nbextensions，勾选Hinterland即可

![image-20220713141658251](pics\image-20220713141658251.png)

需要注意的是，如果Jupyter notebook不是pip安装，而是**conda**自带的话（例如环境为conda的base），则有可能需要改用conda的相关指令才能安装扩展，如下所示。

```
conda install -c conda-forge jupyter_contrib_nbextensions
conda install -c conda-forge jupyter_nbextensions_configurator
```

