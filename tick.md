Title: 新兴监控平台InfluxDB（TICK）初体验
Date: 2019-08-08 18:20
Tags: DevOps, 系统运维
Slug: tick
Author: muxueqz
Summary: 体验InfluxDB(TICK)

# 介绍
近两年技术领域变化真快，新工具层出不穷，InfluxDB作为一款流行的时序数据库，本来就在监控领域应用较广，
现在它又推出了一系列工具来组成完整的监控系统：
* Telegraf：采集数据的Agent，支持超多种常见的监控类型，如Nginx/Redis/SNAP以及Windows的服务，以及通过`Exec Input`来实现自定义扩展
* InfluxDB：时序数据库，用于存储采集到的数据
* Chronograf：Web Dashboard，展示采集到的数据图表，高度自定义，且默认带了许多非常好的Dashboard模板，还可以通过chronograf在Web上配置kapacitor的报警规则，非常方便
* Kapacitor：数据处理器，用于判断是否报警

**友情提示：本文在PC浏览器上访问效果最佳**


## 下载
```bash
wget https://dl.influxdata.com/chronograf/releases/chronograf-1.7.12_linux_amd64.tar.gz
wget https://dl.influxdata.com/kapacitor/releases/kapacitor-1.5.3_linux_amd64.tar.gz
wget https://dl.influxdata.com/telegraf/releases/telegraf-1.11.4_linux_amd64.tar.gz
wget https://dl.influxdata.com/influxdb/releases/influxdb-1.7.7_linux_amd64.tar.gz
```

## 安装
```bash
# 安装InfluxDB时序数据库
tar xvf influxdb-1.7.7_linux_amd64.tar.gz
cd influxdb-1.7.7-1/
./usr/bin/influxd &> influxd.log &
cd ..
# 安装telegraf监控采集Agent
tar xvf telegraf-1.11.4_linux_amd64.tar.gz
rsync -av --progress telegraf/ /
telegraf &> telegraf.log &

# 安装kapacitor报警器
tar xvf kapacitor-1.5.3_linux_amd64.tar.gz
rsync  -av --progress kapacitor-1.5.3-1/ /
useradd -m kapacitor
chown -R kapacitor /var/run/kapacitor/
kapacitord &> kapacitord.log &

# 安装chronograf Dashboard
tar xvf chronograf-1.7.12_linux_amd64.tar.gz
rsync -av --progress chronograf-1.7.12-1/ /
useradd -m chronograf
chown -R chronograf  /var/log/chronograf/
chronograf  &> chronograf.log &
```


## 体验
现在，打开 http://localhost:8888 即可看到chronograf的管理界面了


# Dashboard效果
## 报警列表
![image](https://user-images.githubusercontent.com/730639/62693655-8033ec00-ba05-11e9-9485-b48bd03275b7.png)

## InfluxDB状态
![image](https://user-images.githubusercontent.com/730639/62693783-b1acb780-ba05-11e9-90c8-0480872b7c60.png)

## 系统状态
![image](https://user-images.githubusercontent.com/730639/62693832-cab56880-ba05-11e9-9265-ae2f607277b1.png)

# 总结
TICK作为新兴之秀，
* 一扫上古监控系统的陈旧气息，界面漂亮且定制性强
* 又比其它新秀如Prometheus架构更合理（Prometheus那么多exporter管理起来就是个灾难）
* 也比另一个流行的监控系统——OpenFalcon更简单清晰，OpenFalcon那么多组件，理解起来真是头大

所以还是非常值得关注的，试用的过程中非常顺利，真是非常棒的体验。
