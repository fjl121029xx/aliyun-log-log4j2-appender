# Log4j2 Appender

## Aliyun Log Log4j2 Appender
Log4j2是由log4j的升级版本。通过使用Log4j2，您可以控制日志信息输送的目的地是控制台、文件、GUI 组件、甚至是套接口服务器、NT 的事件记录器、UNIX Syslog 守护进程等；您也可以控制每一条日志的输出格式；通过定义每一条日志信息的级别，您能够更加细致地控制日志的生成过程。最令人感兴趣的就是，这些可以通过一个配置文件来灵活地进行配置，而不需要修改应用的代码。

通过Aliyun Log Log4j2 Appender，您可以控制日志的输出目的地为阿里云日志服务。需要注意的是，Aliyun Log Log4j2 Appender不支持设置日志的输出格式，写到日志服务中的日志的样式如下：
```
level: ERROR
location:  com.aliyun.openservices.log.log4j2.example.Log4j2AppenderExample.main(Log4j2AppenderExample.java:16)
message: error log
thread: main
time: 2018-01-02T03:15+0000
```
其中：
+ level 是日志级别。
+ location 是日志打印语句的代码位置。
+ message 是日志内容。
+ thread 是线程名称。
+ time 是日志打印时间。


## 功能优势
+ 客户端日志不落盘，即数据生产后直接通过网络发往服务端。
+ 对于已经使用Log4j2记录日志的应用，只需要简单修改配置文件就可以将日志传输到日志服务。
+ 异步高吞吐，Aliyun Log Log4j2 Appender 会将用户的日志merge之后异步发送，提高网络I/O效率。


## 版本支持
* log-loghub-producer 0.1.8
* protobuf-java 2.5.0

> 该版本主要适配于Log4J 2.X以上版本，以下版本请参考[aliyun-log-log4j-appender](https://github.com/aliyun/aliyun-log-log4j-appender)


## 配置步骤

### 1. maven 工程中引入依赖

```
<dependency>
    <groupId>com.google.protobuf</groupId>
    <artifactId>protobuf-java</artifactId>
    <version>2.5.0</version>
</dependency>
<dependency>
    <groupId>com.aliyun.openservices</groupId>
    <artifactId>aliyun-log-log4j2-appender</artifactId>
    <version>0.1.1</version>
</dependency>
```

### 2. 修改配置文件

以xml型配置文件`log4j2.xml`为例（不存在则在项目根目录创建），配置Loghub相关的appender与 Logger，例如：
```
    <Appenders>
        <Loghub name="Loghub"
                projectName="your project"
                logstore="your logstore"
                endpoint="your project endpoint"
                accessKeyId="your accesskey id"
                accessKey="your accesskey"
                stsToken="your ststoken"
                packageTimeoutInMS="3000"
                logsCountPerPackage="4096"
                logsBytesPerPackage="3145728"
                memPoolSizeInByte="104857600"
                retryTimes="3"
                maxIOThreadSizeInPool="8"
                topic="your topic"
                timeFormat="yyyy-MM-dd'T'HH:mmZ"
                timeZone="UTC"
                ignoreExceptions="true">
            <PatternLayout pattern="%d{HH:mm:ss.SSS} [%t] %-5level %logger{36} - %msg%n"/>
        </Loghub>
    </Appenders>
    <Loggers>
        <Root level="warn">
            <AppenderRef ref="Loghub"/>
        </Root>
    </Loggers>
```
其中：
level 是日志记录的优先级，优先级从高到低分别是 ERROR、WARN、INFO、DEBUG。通过在这里定义的级别，您可以控制应用程序中相应级别的日志信息的开关。比如在这里定义了 WARN 级别，则应用程序中所有 INFO、DEBUG
级别的日志信息将不被打印出来。

## 参数说明

Aliyun Log Log4j2 Appender 可供配置的属性（参数）如下，其中注释为必选参数的是必须填写的，可选参数在不填写的情况下，使用默认值。

```
#日志服务的 project 名，必选参数
projectName = [your project]
#日志服务的 logstore 名，必选参数
logstore = [your logstore]
#日志服务的 HTTP 地址，必选参数
endpoint = [your project endpoint]
#用户身份标识，必选参数
accessKeyId = [your accesskey id]
accessKey = [your accesskey]

#被缓存起来的日志的发送超时时间，如果缓存超时，则会被立即发送，单位是毫秒，默认值为3000，最小值为10，可选参数
packageTimeoutInMS = 3000
#每个缓存的日志包中包含日志数量的最大值，不能超过 4096，可选参数
logsCountPerPackage = 4096
#每个缓存的日志包的大小的上限，不能超过 5MB，单位是字节，可选参数
logsBytesPerPackage = 5242880
#Appender 实例可以使用的内存的上限，单位是字节，默认是 100MB，可选参数
memPoolSizeInByte = 1048576000
#指定I/O线程池最大线程数量，主要用于发送数据到日志服务，默认是8，可选参数
maxIOThreadSizeInPool = 8
#指定发送失败时重试的次数，如果超过该值，会把失败信息记录到log4j的StatusLogger里，默认是3，可选参数
retryTimes = 3

#指定日志主题
topic = [your topic]

#输出到日志服务的时间格式，使用 Java 中 SimpleDateFormat 格式化时间，默认是 ISO8601，可选参数
timeFormat = yyyy-MM-dd'T'HH:mmZ
timeZone  UTC
```
参阅：https://help.aliyun.com/document_detail/43758.html

## 使用实例
项目中提供了一个名为`com.aliyun.openservices.log.log4j2.Log4j2AppenderExample`的实例，它会加载resources目录下的`log4j2.xml`文件进行log4j2配置。

**log4j2.xml样例说明**
+ 配置了三个appender：loghubAppender1、loghubAppender2、STDOUT。
+ loghubAppender1：将日志输出到project=test-proj，logstore=store1。输出WARN、ERROR级别的日志。
+ loghubAppender2：将日志输出到project=test-proj，logstore=store2。只输出INFO级别的日志。
+ STDOUT：将日志输出到控制台。由于没有对日志级别进行过滤，会输出root中配置的日志级及以上的所有日志。

[Log4j2AppenderExample.java](/src/main/java/com/aliyun/openservices/log/log4j2/example/Log4j2AppenderExample.java)

[log4j2.xml](/src/main/resources/log4j2.xml)