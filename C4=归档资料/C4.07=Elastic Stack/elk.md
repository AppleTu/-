# 1. elasticsearch

![image-20210716153142983](D:/Project/gitbook/%E6%B8%A3%E6%B8%A3%E8%BF%90%E7%BB%B4%E5%8F%B2/C4=%E5%BD%92%E6%A1%A3%E8%B5%84%E6%96%99/C4.07=Elastic%20Stack/elk.assets/image-20210716153142983-1626420709740.png)

# 2. filebeat

## 2.1 output.console 通过终端获取数据

`vi /etc/filebeat/filebeat.yml`

```bash
filebeat.inputs:  # 写入数据
- type: log       # 日志格式 log | stdin | redis | udp | docker
  enabled: true   # 开启 input
  paths:          # 写入数据的路径文件
    - /opt/tomcat/logs/*.log  # 日志路径
output.console:   # 输出收集的数据
  pretty: true    # 格式化输出
  enable: true    # 是否开启 output
```

```bash
./filebeat -e -c filebeat.yml   # 启动 filebeat
```



## 2.2 output.elasticsearch

`vi  /etc/filebeat/filebeat.yml`

```bash
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /opt/tomcat/logs/*.log
output.elasticsearch:       # output 到 elasticsearch 
  hosts:["192.168.1.101:9200"]   # 指定 elasticsearch 的节点IP地址
```

```bash
./filebeat -e -c filebeat.yml   # 启动 filebeat
```



## 2.3 Module 内置模块采集Nginx

filebeat 内置模块目录  `/usr/share/filebeat/module`

filebeat 开启 module 的配置文件目录 `/etc/filebeat/modules.d`

### 2.3.1 nginx 模块启动和停止

```bash
./filebeat module enable nginx   # 安装 nginx 模块
./filebeat module disable nginx  # 卸载 nginx 模块
./filebeat module list           # 查看模块启动列表，是否启动成功
```

### 2.3.2 配置 Module - Nginx 配置文件

`vi /etc/filebeat/modules.d/nginx.yml`

```bash
- module: nginx
  access:
    enable: true
    var.paths: ["/usr/local/nginx/logs/access.log*"]  # 重点：别忘了加个 *
    
  error:
    enabled: true
    var.paths: ["/usr/local/nginx/logs/error.log*"]  # 重点：别忘了加个 *
    
```

### 2.3.3 配置 filebeat

`vi /etc/filebeat/filebeat.yml`

```bash
filebeat.inputs:

output.elasticsearch:
  hosts: ["192.168.1.101:9200"]    # output 到 elasticsearch

filebeat.config.modules:           # 指定内置 Module 模块 
  path: ${path.config}/modules.d/nginx.yml  # nginx.yml 文件路径
  reload.enable: false             # 是否重新加载，暂时 false
```

```bash
./filebeat -e -c filebeat.yml   # 启动 filebeat
```

### 2.3.4  Module - Nginx - 日志仪表板

`vim /etc/filebeat/filebeat.yml`

```bash
filebeat.inputs:

output.elasticsearch:
  hosts: ["192.168.1.101:9200"]

filebeat.config.modules:
  path: ${path.config}/modules.d/nginx.yml
  reload.enable: false
  
setup.kibana:
  host: "192.168.1.101:5601"       # Kibana 仪表板
```

```bash
./filebeat -c filebeat.yml setup  # 安装仪表板 
./filebeat -e -c filebeat.yml     # 启动 filebeat
```

在 Kibana -dashboard 中搜 Nginx，点击 Filebeat 仪表板即可查看

## 2.4 减少无用的日志信息

## 2.5 更改索引名称

### 2.5.1 修改file beat

```bash
fielbeat.inputs:
- type: log
  enable: true
  paths: /opt/tomcat/logs/*.log
  
output.elasticsearch:
  hosts: ["192.168.1.101:9200"]
  index: tomcat-%{agent.version}-%{yyyy.MM.dd}  # 自定义索引名称，日产1T以上，按天；日产1T一下，可按月。
  
setup.ilm.enabled: false
setup.template:
  name: "api-tomcat"        # 定义模板名称,索引关联的模板名称
  pattern: "api-tomcat-*"   # 定义模板的匹配索引名称
setup.template.overwrite: true  # 是否覆盖现有的模板
setup.template.enabled: false   # 禁用模板加载
setup.template.settings:   # 定义索引分片数和副本
  index.number_of_shards: 1
  index.number_of_replicas: 1
```

### 2.5.2 删除 elasticsearch 索引

### 2.5.3 kibana 重新添加索引名称



