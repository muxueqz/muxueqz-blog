Title: Heka
Date: 2015-05-30 13:20
Tags: heka
Category: 系统运维
Slug: heka
Author: muxueqz
Summary: heka是一个用Golang编写的日志收集服务

# Heka
+ 日志收集服务
+ 流式
+ Mozilla出品
+ Golang编写
+ 轻量却功能强大
+ 灵活易用
+ Go/Lua扩展

# Heka用户
+ Mozilla(FireFox)
+ Disqus(最流行的评论服务)

# Heka架构
## Inputs
用于接收数据，有多种输入方式，如：

- 文件
- TCP
- 消息队列如AMQP
- 系统命令输出
## Splitters
用于将Input收到的数据进行拆分，比如按换行符
## Decoders
解析Input/Split之后的数据

- Nginx/Apache Access Log
- GeoIP
- syslog
- MultiDecoder，把多个Decoder整合起来一起用
- 还有很多等等，以及很方便的Lua扩展
## Filters
消息处理引擎，可以进行过滤、计算、聚合
## Encoders
在输出到Output前将数据处理成Output支持的数据格式
- 有许多常用插件，以及很方便的Lua扩展
## Outputs
可以同时多个输出落地，如：

- ElasticSeasrch
- TCP
- AMQP
- HTTP
- 还有很多等等，以及很方便的Lua扩展

# 日志收集架构
## 1.至简
heka logstreamer Input-> Output到最终落地
## 2.性能扩展
heka logstreamer Input-> TCP Output -> 汇总层heka TCP Input -> Output到最终落地
## 3.性能扩展(使用队列)
heka logstreamer Input-> AMQP(Kafka) Output -> 汇总层heka AMQP(Kafka) Input -> Output到最终落地

# 性能测试
环境：4 CPU虚拟机，CentOS 6.5
syslog协议+文件输出，可达到**34K/s**
