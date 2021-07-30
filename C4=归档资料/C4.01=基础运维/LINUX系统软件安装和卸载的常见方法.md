# LINUX系统软件安装和卸载的常见方法

linux系统分很多种简单介绍几种常用的：

## 1、centos/redhat：

**安装：**

rpm安装，如果有依赖，很闹心，如果使用--nodeps不检查依赖，会有问题。

```bash
#rpm -ivh <XXX.rpm> #不检查依赖的话，添加 --nodeps
```

yum安装，自动解决依赖，推荐。

```bash
#yum -y install openssh-server #通过yum自动解决依赖 -y 自动确认安装
```

源码安装，由于centos及redhat系统出于稳定性考虑，很多软件版本都很低，需要使用源码安装：

```bash
#tar xf XXXX.tar
#cd XXXX
#./configure
#make && make install
```

 

**卸载：**

rpm卸载，同样需要考虑依赖，可使用--nodeps

```bash
#rpm -e XXXX #xXX 可以使用rpm -qa 来获得
```

使用yum卸载，需要注意，会将依赖的卸载导致莫名的问题，需要注意。

```bash
#yum remove XXXX
```

源码包卸载：

直接删除源码包

进入源码包，`make uninstall`

 

## 2、ubuntu系统

ubuntu系统软件较新，而且比较全，几乎想要的都可能使用apt-get来安装

安装：

使用dpkg安装，类似于rpm安装 是debian系统的软件包管理工具

```
#dpkg -i <XXXX.deb>
```

apt-get包管理工具：

```
#apt-get install openssh-server #类似centos的yum
```

这里延伸介绍一下ubuntu/debian系统下的解决依赖神器：

aptitude包管理工具：

aptitude包管理工具要比ubuntu原生自带的apt-get 要好用，比如在卸载软件时，会卸载的更彻底

```
$sudo aptitude install gcc-c++
```

 

卸载：

使用dpkg：

```
#dpkg -r <XXXX.deb>
```

使用apt-get：

```
#apt-get remove <XXXX>
#apt-get autoremove <XXX>
```

使用aptitude:

```
#aptitude remove <XXX>
```

javascript:void(0))