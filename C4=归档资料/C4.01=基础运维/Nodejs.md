# Node.js

# 1. 下载文件

下载地址：[https://nodejs.org/zh-cn/download/](https://nodejs.org/zh-cn/download/)

# 2. windows 下安装配置

## 2.1 配置 npm 安装目录

.zip包安装，直接解压，然后`重命名为node`（方便以后升级，直接覆盖就行，不用纠结目录名上的版本号）

我这里设置解压安装目录为`C:\node`，在根目录下创建两个文件夹：

`node_global`（npm全局安装位置）

`node_cache`（npm 缓存路径）

## 2.2 修改环境变量

```bash
# 添加系统变量，上移置顶
Path：C:\nodejs
NODE_PATH:C:\nodejs\node_global\node_modules  
# 添加用户变量
Path：C:\nodejs\node_global

# 修改默认配置
npm config set prefix "C:\nodejs\node_global"
npm config set cache "C:\nodejs\node_cache"

```

## 2.3 修改镜像源

```bash
# 查看镜像源
npm config list
# 修改默认镜像源
npm set registry https://registry.npm.taobao.org/
# 删除镜像源淘宝地址则回到默认镜像地址：
npm config rm registry

# nrm是专门用来管理和快速切换私人配置的registry
node
# 全局安装
npm install nrm -g

# 用nrm ls命令查看默认配置，带*号即为当前使用的配置, 默认自带了好几个
nrm ls

# 查看当前使用的是哪个源
nrm current

# 切换源
nrm use taobao

# 添加源
nrm add 原名称 源地址
nrm add taobao https://registry.npm.taobao.org

# 删除源
nrm del taobao

# 测试源的网速
nrm test taobao

```

## 2.4 更新 Node 版本和 npm 版本

```bash
# 查看node 版本, 有点老，想更新(直接删了老的，把新的安装一遍也挺快)
node -v

# 安装node版本管理工具’n’
npm install n -g

# 清楚node缓存
npm cache clean -f

# 升级npm的版本
npm install npm@latest -g    # 升级到最新版
npm install npm -g			 # 升级到最新版
npm install npm@6.4.1 -g     # 升级到指定版

# 升级 node
n stable

# 查看node的地址配置环境变量
where node  
# or
which node
```

# 3. linux 下安装配置

```bash
# 创建安装目录
sudo mkdir -p /usr/local/nodejs
# 解压到指定目录
sudo tar -xJvf node-v14.17.3-linux-x64.tar.xz -C /usr/local/nodejs
# 测试
cd /usr/local/nodejs/bin
./node -v

# 修改默认配置
cd /usr/local/nodejs
mkdir node_global
mkdir node_cache
npm config set prefix "/usr/local/nodejs/node_global"
npm config set cache "/usr/local/nodejs/node_cache"

# 软连接
sudo ln -s /usr/local/nodejs/bin/node /usr/local/bin/node
sudo ln -s /usr/local/nodejs/bin/npm /usr/local/bin/npm
sudo ln -s /usr/local/nodejs/bin/npx /usr/local/bin/npx

# 环境变量
vi /etc/profile
export NODE_HOME=/usr/local/nodejs
export PATH=$PATH:$NODE_HOME/bin
export NODE_PATH=$NODE_HOME/node_modules
source /etc/profile   # 使环境变量生效

# 任意目录下测试
node -v
npm -v
npx -v

# 觉得npm慢的时候，可以
# 设置 淘宝镜像源
npm config set registry https://registry.npm.taobao.org
# 查看 使用的 镜像源
npm config get registry

# 安装 淘宝镜像源 cnpm
npm install -g cnpm --registry=https://registry.npm.taobao.org
sudo ln -s /usr/local/nodejs/node_global/cnpm /usr/local/bin/cnpm

```

