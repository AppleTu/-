# JsonLayout log4j2 json格式输出日志

如果日志输出时，想改变日志的输出形式为Json格式，可以在**log4j2.xml**中使用JsonLayout标签，使日志输出格式为Json格式。

前提需要Jackson的包，保证项目中包含jackson的依赖

然后在log4j2.xml中在需要日志输出的地方，添加

```
<JsonLayout/>
```

例如需要在控制台输出格式为Json格式则：

```xml
 <!--这个输出控制台的配置-->
        <Console name="Console" target="SYSTEM_OUT">  
            <JsonLayout/>
        </Console>  
```

如果需要在日志文件输出，则为：

```xml
         <RollingFile name="syswareLog" fileName="${LOG_HOME}/logs/sysware.log"
                     filePattern="${LOG_HOME}/logs/$${date:yyyy-MM}/sysware-%d{yyyy-MM-dd}-%i.log">
            <JsonLayout/>
            <Policies>
                <TimeBasedTriggeringPolicy/>
                <SizeBasedTriggeringPolicy size="100000 kb"/>
            </Policies>
            <!-- DefaultRolloverStrategy属性如不设置，则默认为最多同一文件夹下7个文件，这里设置了20 -->
            <DefaultRolloverStrategy max="100"/>
        </RollingFile>
```



设置完成之后，日志输出格式如下：

```json
{
  "timeMillis" : 1547108632526,
  "thread" : "main",
  "level" : "WARN",
  "loggerName" : "com.ctrip.framework.apollo.internals.DefaultMetaServerProvider",
  "message" : "Could not find meta server address, because it is not available in neither (1) JVM system property 'apollo.meta', (2) OS env variable 'APOLLO_META' (3) property 'apollo.meta' from server.properties nor (4) property 'apollo.meta' from app.properties",
  "endOfBatch" : false,
  "loggerFqcn" : "org.apache.logging.slf4j.Log4jLogger",
  "threadId" : 1,
  "threadPriority" : 5
}
```

但是这个日志是格式化后的json,有同事提出需求，想让一个json对象输出为一行，这样日志读取的时候，可以以一个对象为单位读取，而不是一行。

继续查看JsonLayout 源码，既然他提供了转换为json,那是不是也可以提供格式化json规则。

org.apache.logging.log4j.core.layout.JsonLayout 。 继承自 AbstractJacksonLayout，在AbstractJacksonLayout 中有下面几个属性介绍。我们下面主要介绍这几个属性的含义及使用方法。

```java
    protected final String eventEol;
    protected final ObjectWriter objectWriter;
    protected final boolean compact;
    protected final boolean complete;
```

 

> **compact** 设置是否紧凑输出，默认是false,如果设置为true,则不是用行位和缩进，大概意思是说，输出把所有的日志输出为一行。
> 　　　　　原文（ If "true", does not use end-of-lines and indentation, defaults to "false".）
> **complete** 设置是否完成，默认是false， 如果设置为true,则在日志开始和结束会包含页眉和页脚，和逗号。
> 　　　　原文（If "true", includes the JSON header and footer, and comma between records.）
> **eventEol** 如果为“true”，则在每个日志事件后强制执行EOL（即使compact是“true”），默认是false,即使在紧凑模式下，也会生效，大概意思就是说，
> 　　　　  即使是在紧凑模式下，如果evetEol设置为true ，也会在每个log event(log.info().log.error(),log.debug()等等)之后强制换行输出。
> objectWriter 暂不清楚
>
> 通过上面的属性介绍，根据需要，只需将`compact` ,`evetEol` 设置成true，即可将每行日志输出成一行json。



# 参照 Log4j 官网的说明：

[https://logging.apache.org/log4j/2.x/log4j-core/apidocs/org/apache/logging/log4j/core/layout/JsonLayout.html](https://logging.apache.org/log4j/2.x/log4j-core/apidocs/org/apache/logging/log4j/core/layout/JsonLayout.html)

> @Deprecated
> public static JsonLayout createLayout(Configuration config,
> boolean locationInfo,
> boolean properties,
> boolean propertiesAsList,
> boolean complete,
> boolean compact,
> boolean eventEol,
> String headerPattern,
> String footerPattern,
> Charset charset,
> boolean includeStacktrace)
> Deprecated. Use newBuilder() instead
> Creates a JSON Layout.
> Parameters:
> config - The plugin configuration.
> locationInfo - If “true”, includes the location information in the generated JSON.
> properties - If “true”, includes the thread context map in the generated JSON.
> propertiesAsList - If true, the thread context map is included as a list of map entry objects, where each entry has a “key” attribute (whose value is the key) and a “value” attribute (whose value is the value). Defaults to false, in which case the thread context map is included as a simple map of key-value pairs.
> complete - If “true”, includes the JSON header and footer, and comma between records.
> compact - If “true”, does not use end-of-lines and indentation, defaults to “false”.
> eventEol - If “true”, forces an EOL after each log event (even if compact is “true”), defaults to “false”. This allows one even per line, even in compact mode.
> headerPattern - The header pattern, defaults to “[” if null.
> footerPattern - The header pattern, defaults to “]” if null.
> charset - The character set to use, if null, uses “UTF-8”.
> includeStacktrace - If “true”, includes the stacktrace of any Throwable in the generated JSON, defaults to “true”.
> Returns:
> A JSON Layout.

参数说明部分翻译过来是：

> config - 插件配置。
> locationInfo - 如果为“true”，则在生成的 JSON 中包含位置信息。
> properties - 如果为“true”，则在生成的 JSON 中包含线程上下文映射。
> propertiesAsList - 如果为 true，则将线程上下文映射包括为映射条目对象的列表，其中每个条目具有“key”属性（其值为键）和“value”属性（其值为值）。默认为 false，在这种情况下，线程上下文映射包含为键值对的简单映射。
> complete - 如果为“true”，则包括 JSON 页眉和页脚，以及记录之间的逗号。
> compact - 如果为“true”，则不使用行尾和缩进，默认为“false”。
> eventEol - 如果为“true”，则在每个日志事件后强制执行 EOL（即使compact为“true”），默认为“false”。即使在紧凑模式下，这也允许一行甚至每一行。
> headerPattern- 标题模式，默认为"[“如果是 null。
> footerPattern- 标题模式，默认为”]"如果是 null。
> charset- 要使用的字符集if null，使用“UTF-8”。
> includeStacktrace - 如果为“true”，则包括生成的 JSON 中任何Throwable的堆栈跟踪，默认为“true”。

满足每条日志输出为一行 json 的需求我们该这样设置：

```xml
<JsonLayout compact="true" locationInfo="true" complete="false" eventEol="true"/>
```

完整的 log4j2.xml 配置如下：

```xml
<?xml version="1.0" encoding="UTF-8"?>
<Configuration status="INFO">
    <!-- 日志文件目录和压缩文件目录配置 -->
    <Appenders>
        <Console name="Console" target="SYSTEM_OUT">
            <!--控制台只输出level及以上级别的信息（onMatch），其他的直接拒绝（onMismatch）-->

            <ThresholdFilter level="info" onMatch="ACCEPT" onMismatch="DENY"/>
            <PatternLayout
                    pattern="%highlight{%d{yyyy.MM.dd 'at' HH:mm:ss z} %-5level %class{36} %M() @%L - %msg%n}{FATAL=Bright Red, ERROR=Bright Magenta, WARN=Bright Yellow, INFO=Bright Green, DEBUG=Bright Cyan, TRACE=Bright White}"/>
        </Console>

        <!--这个会打印出所有的信息，每次大小超过size，则这size大小的日志会自动存入按年份-月份建立的文件夹下面并进行压缩，作为存档-->
        <RollingFile name="RollingInfoFile" fileName="log/app.log"
                     filePattern="logs/info/$${date:yyyy-MM}/app-%d{MM-dd-yyyy}-%i.log.gz">
            <JsonLayout compact="true" locationInfo="true" complete="false" eventEol="true" />
            <SizeBasedTriggeringPolicy size="20MB"/>
        </RollingFile>

        <!-- error 日志 -->
        <RollingFile name="RollingErrorFile" fileName="log/error.log"
                     filePattern="logs/error/$${date:yyyy-MM}/error-%d{MM-dd-yyyy}-%i.log.gz">
            <JsonLayout compact="true" locationInfo="true" complete="false" eventEol="true"/>
            <ThresholdFilter level="error" onMatch="ACCEPT" onMismatch="DENY"/>
            <SizeBasedTriggeringPolicy size="20MB"/>
        </RollingFile>
    </Appenders>

    <!-- 全局配置，默认所有的Logger都继承此配置 -->
    <Loggers>
        <Root level="INFO">
            <AppenderRef ref="RollingInfoFile"/>
            <AppenderRef ref="RollingErrorFile"/>
            <AppenderRef ref="Console"/>
        </Root>
    </Loggers>
</Configuration>

```

