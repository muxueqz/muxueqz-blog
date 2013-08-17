cron快速手册
##########################################################################################################################################
:date: 2009-06-15 12:13:09
:category: 系统运维
:slug: crontab

cron是一个UNIX工具,使用cron后台进程使得任务能够以后台方式在特定时间自动执行。这些任务通常叫做cron
jobs. crontab是一个包括在特定时间要运行的cron记录的

1.Crontab限制:

如果你的名字出现在/usr/lib/cron/cron.allow，你可以执行crontab.如果那个文件不存在,而你的名字又没有在/usr
/lib/cron/cron.deny中出现,你也可以使用crontab.如果只有cron.deny存在,而且是空文件，那么所有用户都可以使用
crontab.如果两个文件都不存在，那么只有root用户可以使用crontab.all/deny文件每行一个用户名。

2.crontab命令：

 首先需要指定一个编辑器来打开crontab文件，通常使用vi.

:: 

 crontab -e：编辑你的crontab文件或者当它不存在时创建一个。
 crontab -l：显示你的crontab文件。
 crontab -r：删除你的crontab文件。

3.crontab文件语法：

:: 

 \* \* \* \* \* command to be executed
 - - - - -
 \| \| \| \| \|
 \| \| \| \| +—– day of week (0 - 6) (Sunday=0)
 \| \| \| +——- month (1 - 12)
 \| \| +——— day of month (1 - 31)
 \| +———– hour (0 - 23)
 +————- min (0 - 59)

上面value域中的\*表示所有该列括号中的合法值。value列可以是一个\*，也可以是使用逗号隔开的一组元素。所有列的元素要么是上述范围的的一个数字，要么是被分割符(-)分开的上述范围内的两个数字(表示一个左右闭合域)。
日期的指定可以在两个域中：month day和week day.如果两个在一条记录中都指定，那么他们是两个记录的累计效果。

4.crontab示例
 下面crontab文件中的一列在18:30从/home/someuser/tmp中移除临时文件：

::

 30 18 \* \* \* rm /home/someuser/tmp/\*
 如下所示改变参数会使得该命令按照不同的时间计划执行：
 min
 hour
 day/month
 month
 day/week
 Execution time
 30
 0
 1
 1,6,12
 \*
 1 月 ,6 月和 12 月的 1 日的 00:30 分执行
 0
 20
 \*
 10
 1-5
 10 月份的每个工作日 ( 周一至周五 ) 的 20: 点执行


 0
 0
 1,10,15
 \*
 \*
 #每个月的 1,10,15 日午夜执行

 5,10
 0
 10
 \*
 1

 每个星期一以及每个月的 10 号，在 00:05 与 00:10 执行

5.Crontab环境

 cron从用户的HOME目录，使用/usr/bin/sh来调用命令。
 cron为每个shell提供一个缺省的环境，定义如下:

::

 HOME=user’s-home-directory
 LOGNAME=user’s-login-id
 PATH=/usr/bin:/usr/sbin:.
 SHELL=/usr/bin/sh

如果用户希望他们的.profile被执行，则必须显式的在crontab的命令脚本中执行,或者在一个单独的脚本中，这个单独的脚本被crontab命令脚本调用。

6.Disable Email

在命令行下输出到屏幕上的信息在使用crontab时会写入到mail中，如果不需要，将下面命令放到crontab的cron
job line的末尾。

备注：在UNIX上使用>/dev/null运行一个程序时可以屏蔽掉程序向stdout的输出，然而如果程序也有向stderr的输出时，仍然会将向stderr输出的信息显示到屏幕上。常用的一种屏蔽stderr的方法是使用管道，把stderr也输出到stdout，而stdout输出到
/dev/null中。一般有如下写法：proc >/dev/null 2>&1.这种方法仅适用于sh环境下，如果在其他shell环境，例如csh，就会出现“Ambiguous output redirect.”这样的错误。幸好crontab的默认shell正是sh.

7.产生日志文件

 与6的原理一样，将其重定向到文件即可：
 >/home/someuser/cronlogs/a.log 2>&1
