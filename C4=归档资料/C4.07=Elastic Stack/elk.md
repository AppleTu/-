# 1. elasticsearch

```bash
rpm -ivh elasticsearch-7.8.0-x86_64.rpm
rpm -ivh filebeat-7.8.0-x86_64.rpm
rpm -ivh kibana-7.8.0-x86_64.rpm
```



# 2. filebeat

官方文档：[https://www.elastic.co/guide/en/beats/filebeat/7.13/configuration-filebeat-options.html](https://www.elastic.co/guide/en/beats/filebeat/7.13/configuration-filebeat-options.html)

```
[root@ZJHZ-YUNMAS-API01 filebeat]# find / -name filebeat
/etc/filebeat
/etc/rc.d/init.d/filebeat
/usr/share/filebeat
/usr/share/filebeat/bin/filebeat
/usr/bin/filebeat
```



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
output.elasticsearch:            # output 到 elasticsearch 
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

### 2.4.1 方式一：修改 fielbeat.yml 的 inputs

```bash
fielbeat.inputs:
- type: log
  enable: true
  paths: /opt/tomcat/logs/*.log
  include_lines: ['^ERR','WARN','']  # 只收集错误、警告、sshd的日志
```

### 2.4.1 方式二：修改 fielbeat.yml 的 processors

```bash
processors:
  # 将自带的一些字段可以去除
  # - add_host_metadata:
  #     when.not.contains.tags: forwarded
  # - add_cloud_metadata: ~
  # - add_docker_metadata: ~
  # - add_kubernetes_metadata: ~
  # 在这里可以设置要去除的字段
  - drop_fields:
      # when: 可以设置去除的条件
      #   condition
      fields: ["log","host","input","agent","ecs"]
      ignore_missing: false
```

### 2.4.1 方式二：修改 fielbeat.yml 的 output

```bash
output.elasticsearch:
  hosts:["192.168.1.101:9200"]
  
  # 按照 json 格式输出
  codec.json:
    pretty: true
    escape_html: false
    
  # 按照字符串格式只输出message
  codec.format:
    string: '%{[message]}'
```



## 2.5 更改索引名称

### 2.5.1 方式一：修改file beat

```bash
fielbeat.inputs:
- type: log
  enable: true
  paths: /opt/tomcat/logs/*.log
  
output.elasticsearch:
  hosts: ["192.168.1.101:9200"]
  index: tomcat-%{agent.version}-%{yyyy.MM.dd}  # 自定义索引名称，日产1T以上，按天；日产1T一下，可按月。
  
setup.ilm.enabled: false      # 6.8以上必须加上这，定义所应关联的模板mign
setup.template:
  name: "api-tomcat"          # 定义模板名称,索引关联的模板名称
  pattern: "api-tomcat-*"     # 定义模板的匹配索引名称
  overwrite: true             # 是否覆盖现有的模板
  enabled: false              # 禁用模板加载
setup.template.settings:      # 定义索引分片数和副本
  index.number_of_shards: 1   # 分片数
  index.number_of_replicas: 1 # 副本数
```

> 要使设定的模板生效，需要：
>
> 1. 删除 old 模板
> 2. 删除 old 索引
>
> 3. 重启 filebeat
> 4. kibaban 重新添加索引模式

### 2.5.2 方式二：es 上修改模板

使用 cerebro  或  kibana 工具直接修改模板

在 “setting” 节下，添加 shards 分片数量，replicas 数量：

```bash
"number_of_routing_shards": "30",
"number_of_shards": "10",
"number_of_replicas": "1"
```

1. 修改 test 模板；
2. 删除模板关联的索引；
3. 删除 file beat 自行制定的分片数和副本数



## 2.6 filebeat 自定义字段等

### 2.6.1 



## 2.7 filebeat 实战 - 收集 tocmat 日志

需求：

1.tomcat 日志

2.应用 日志

3.json 格式

```bash
filebeat.inputs:
- type: log
  enabled: true
  paths:
    - /opt/tomcat/logs/*.log    # 日志路径
  json.keys_under_root: true    # 可以让字段位于根节点，默认为false
  json.overwrite_keys: true     # 覆盖默认的key，使用自定义json格式的key
  json.add_error_key: true      # 将解析错误的消息记录储存在error.message字段中
  json.message_key: message     # 用来合并多行json日志使用的，如果配置该项还需要配置multiline的设置
  
output.elasticsearch:            # output 到 elasticsearch 
  hosts:["192.168.1.101:9200"]   # 指定 elasticsearch 的节点IP地址
```





# 3 常见问题



~~~bash
1. 找到registry文件的位置，如果没有单独配置那么文件路径为`/var/lib/filebeat/registry`，不在也没关心，可以直接find命令查找
```
# find / -name registry
/var/lib/filebeat/registry
```

2. 关闭filebeat --> 删掉registry文件 --> 启动filebeat
```
/etc/init.d/filebeat stop &&\
rm -r /var/lib/filebeat/registry &&\
/etc/init.d/filebeat start
```
 
参考文档：https://mp.weixin.qq.com/s/H5QiZ10KSdUXFW5LdH79wA
~~~







