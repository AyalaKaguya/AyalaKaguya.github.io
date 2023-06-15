+++
title = "Pytorch学习笔记 - 安装Anaconda&PyTorch"
date = "2022-04-16T20:44:17+08:00"
author = "AyalaKaguya"
summary = '摘要：如何安装Anaconda以及PyTorch呢？附带一些常用指令和排错。'
tags = [
    "PyTorch",
    "Anaconda",
    "机器学习",
    "笔记"
]
categories = [
    "机器学习",
]
keywords = [
    "Pytorch教程",
    "机器学习",
    "Anaconda"
]
+++

> ❗ 注意
>
> 这篇文章有部分尚未完成。

 ## 先看网址

1. PyTorch [https://pytorch.org/](https://pytorch.org/)
2. Anaconda [https://www.anaconda.com/](https://www.anaconda.com/)
3. PyCharm （可选） [https://www.jetbrains.com/pycharm/](https://www.jetbrains.com/pycharm/)


本站不提供镜像

## 安装思路

Anaconda是一个Python发行版，使用Anaconda做环境分隔可以解决很多环境干扰问题（不搞base环境的话基本上没啥大问题），我们将在Anaconda创建一个虚拟环境，然后在这个虚拟环境上安装PyTorch，并测试这个PyTorch环境。

## 安装步骤

### step 1 下载并安装Anaconda

打开Anaconda[网站](https://www.anaconda.com/)，并点击“Download”，
可能会下的有点慢，可以选择使用迅雷加速

![大图](/images/torchle01/20220416210426.png "Anaconda网站")

 - 下载完成后双击安装包安装

![中图](/images/torchle01/20220416211225.png "Anaconda安装包")

-----

 - 接下来一路Next->I Agree->Next->Next

![中图](/images/torchle01/20220416211518.png "Anaconda安装1")
![中图](/images/torchle01/20220416211524.png "Anaconda安装2")
![中图](/images/torchle01/20220416211531.png "Anaconda安装3")
▲ 应该没人用多用户吧（比如跑在服务器上
![中图](/images/torchle01/20220416211542.png "Anaconda安装4")

-----

接下来这一步有两个选项，不够我在这里给个小建议，当然你们也可以根据实际情况自己选择：
 - 当你电脑里没有Python：建议全选
 - 当你电脑里有python,但版本号与选项2给出的版本不一致：建议全选
 - 当你电脑里有Python，但版本号与选项2给出的版本一致：建议只选第一项
 - 当然你可以都不选(建议电脑里面没有预先安装的Python)，在后期自己添加环境变量（详见教程ep 1，请记住上一步的安装位置）

查看python版本号,在这里只看前两位：
```
python -V                           # Python 3.9.7
```

![中图](/images/torchle01/20220416211557.png "Anaconda安装5")

-----

选好之后点击Install,然后就是Next->Next->Finish

![中图](/images/torchle01/20220416213304.png "Anaconda安装6")

接下来弹出一个PyCharm的广告，虽然我自己用的VS Code，不过PyCharm社区版免费，也挺好用的，何乐而不为呢？
![中图](/images/torchle01/20220416213511.png "Anaconda安装7")

接下来两个勾勾都取消然后就可以Finish啦。
![中图](/images/torchle01/20220416213519.png "Anaconda安装8")

### step 2 创建并配置conda环境

当Anaconda被正确安装之后，首先打开你的命令行软件（CMD或者PowerShell），我们需要执行`conda init`命令来应用环境。

```S
$ conda init
```

接下来我们创建一个名为torch的环境，使用Python3.9

```S
$ conda create -n torch python=3.9
```

使用这条指令创建的环境将只包含一些基础且必须的包（比如pip等），如果你需要额外安装其他的东西（比如numpy或者jupyter等），你需要在命令的末尾添加`anaconda`，这里我们并不需要这么做。
在执行过程中可能需要联网下载一些文件，速度比较慢的话请耐心等待。

接着我们激活这个环境，当显示以下图片即意味着conda环境创建成功了
```S
$ conda activate torch
```

![中图](/images/torchle01/20220416223632.png "Anaconda安装13")

> 常用conda指令见ep 3

### step 3 安装PyTorch

> #### 写作规划 (该部分尚未完成)
>
> 1. 区分显卡类型
> 
> 2. N卡找cuda版本号
> 
> 3. 在PyTorch官网寻找合适的版本获取指令
> 
> 4. 在指定环境执行指令

### step 4 在VSCode测试PyTorch的安装情况

> #### 写作规划 (该部分尚未完成)
>
> 1. 设置python解释器
> 
> 2. 代码
> 
> 3. 调试执行
> 
> 4. 结果

### ep 1 手动设置Anaconda

> ❗ 注意
>
> 该部分不是新手教程，在执行这一步骤前，你需要知道你的Anaconda安装位置

假如我的Anaconda安装在`C:\ProgramData\Anaconda3`下，这个时候我们按`win`键，然后直接输入并搜索"编辑系统环境变量"，点击控制面板项。
![中图](/images/torchle01/20220416215540.png "Anaconda安装9")

让后我们点击右下角的"环境变量"选项
![中图](/images/torchle01/20220416215823.png "Anaconda安装10")

我们在"系统变量"中找到"Path"变量，点击编辑
![中图](/images/torchle01/20220416220157.png "Anaconda安装11")

这个时候我们添加以下三项：
```
C:\ProgramData\Anaconda3                # 这是你刚才记下的位置
C:\ProgramData\Anaconda3\Scripts        # 这是你刚才记下的位置的一个子目录
C:\ProgramData\Anaconda3\Library\bin    # 注意改变前面的位置
```
![中图](/images/torchle01/20220416220433.png "Anaconda安装12")

最后我们打开一个命令行窗口，输入以下命令测试conda是否被正确安装：
```S
$ conda --version                        # conda 4.12.0
```

> 常用conda指令见ep 3

### ep 2 pip换源

pip安装包比较慢想换源？

这里我安利一个快捷换源方法：`pqi`

打开你的命令行终端，输入以下指令即可快捷切换至tuna源：

```S
$ pip install -i https://pypi.tuna.tsinghua.edu.cn/simple/ pqi
$ pqi use tuna
```

当然pqi的用法还可以在研究研究....

### ep 3 常用的conda命令

1. 基本指令

 - conda list           查看当前环境安装了哪些包。
 - conda info --env     查看当前都有哪些虚拟环境
 - conda update conda   更新conda
 - conda --version      查询conda版本
 - conda -h             查询conda命令帮助

2. 创建或删除环境

```S
$ conda create -n <env_name> python=X.X                  # 创建python版本为X.X、名字为<env_name>的虚拟环境
$ conda create -n <env_name> python=X.X anaconda         # 创建python版本为X.X、名字为<env_name>的虚拟环境，附带常用包
$ conda remove -n <env_name> --all                       # 删除名为<env_name>的环境及环境下所有的包
```

3. 激活或退出环境

```S
$ conda activate <env_name>             # 激活名为<env_name>的环境
$ conda deactivate                      # 退出到base环境
```

4. 包管理

```S
$ conda install -n <env_name> <package_name>        # 在<env_name>环境内安装名为<package_name>的包
$ conda remove --name <env_name> <package_name>     # 卸载在<env_name>环境内名为<package_name>的包
```