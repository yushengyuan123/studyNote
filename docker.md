# 容器和镜像的关系

镜像包含了我们代码所运行的环境，例如gcc的编译环境或者node环境等等

而容器是根据我们的镜像通过docker run命令进行创建，创建出来的容器就是具有了镜像的环境

可以理解为容器就是我们镜像的实例

## dockerfile

dockerfile的作用就是定制我们的镜像，使用docker build命令会根据我们的dockerfile文件创建处一个新的镜像，不是创建容器！！！！！！所以dockerfile的作用就是帮助我们定制镜像的

build完之后，我们就可以run创建容器了


