mysql压力测试及优化
##########################################################################################################################################
:date: 2010-09-27 12:41:51
:category: 系统运维
:slug: mysql_innodb

环境
''''

硬件环境：

+------------------+---------------------+----------------------+------------------------------------------+
| 型号             | CPU                 | 内存                 | 网卡                                     |
+------------------+---------------------+----------------------+------------------------------------------+
| Dell PowerEdge   | 双路四核            | 8G ECC               | Broadcom Corporation                     |
|  R410            |  Xeon E5506 2.13G   |  DDR3 1333 MHZ内存   |  NetXtreme II BCM5716 Gigabit Ethernet   |
+------------------+---------------------+----------------------+------------------------------------------+

 软件环境：

+--------------------+------------------------------------------------------+
| 操作系统版本       | DB                                                   |
+--------------------+------------------------------------------------------+
| Debian GNU/Linux   | mysql Ver 14.14 Distrib 5.1.47,                      |
|  Lenny 5.0 amd64   |  for debian-linux-gnu (x86\_64) using readline 5.2   |
+--------------------+------------------------------------------------------+

经测试， innodb\_flush\_log\_at\_trx\_commit影响较大，
 innodb\_flush\_log\_at\_trx\_commit = 1(默认)时的测试情况：

::

    Benchmark
    Running for engine innodb
    Average number of seconds to run all queries: 43.718 seconds
    Minimum number of seconds to run all queries: 43.718 seconds
    Maximum number of seconds to run all queries: 43.718 seconds
    Number of clients running queries: 2000
    Average number of queries per client: 10


    User time 0.10, System time 0.19
    Maximum resident set size 0, Integral resident set size 0
    Non-physical pagefaults 15784, Physical pagefaults 0, Swaps 0
    Blocks in 0 out 0, Messages in 0 out 0, Signals 0
    Voluntary context switches 63228, Involuntary context switches 1499

innodb\_flush\_log\_at\_trx\_commit = 0(改动后)的测试情况：

::

    Benchmark
    Running for engine innodb
    Average number of seconds to run all queries: 4.469 seconds
    Minimum number of seconds to run all queries: 4.469 seconds
    Maximum number of seconds to run all queries: 4.469 seconds
    Number of clients running queries: 2000
    Average number of queries per client: 10


    User time 0.18, System time 0.19
    Maximum resident set size 0, Integral resident set size 0
    Non-physical pagefaults 15480, Physical pagefaults 5, Swaps 0
    Blocks in 736 out 0, Messages in 0 out 0, Signals 0
    Voluntary context switches 61984, Involuntary context switches 2991

