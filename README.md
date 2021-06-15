# pintos

## 前提背景

本篇文章只是我自己一点点对实验的拙见，具体详细还应该看同学大佬的详细介绍

本文参考:

[大佬轩宝博客](https://www.dengzexuan.top/)

[初始代码（未通过实验版本）](https://github.com/ThierrySans/CSCC69-Pintos)

[通过案例的参考代码](https://github.com/NicoleMayer/pintos_project2)

[自己的代码（其实都是根据大佬们的完整代码en嫖的）](https://github.com/Zhangkaiwen2001/pintos)

下面开始我们的实验吧



## 环境配置

因为我看到有很多同伴都不知道环境应该如何配置

下面提供一个不用自己配置直接无脑使用的方法——docker

docker可以使用别人搭建好的环境，然后我们直接使用即可

[docker的下载安装可见此链接](https://yeasy.gitbook.io/docker_practice/install/)

安装好后可以直接在pintos文件夹内打开终端，输入命令

```
docker run --rm --name pintos -it -v "$(pwd):/pintos" thierrysans/pintos bash
```

即可进入由thierrysans教授提供的ubuntu环境，然后就可以愉快的跑代码啦

![成功进入ubuntu环境的界面](../pintos/1.png)

图为成功进入ubuntu环境的界面

