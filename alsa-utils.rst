Alsa-utils使用方法
##########################################################################################################################################
:date: 2009-07-12 14:54:44
:category: 系统运维
:tags: alsa,sound
 
注: 前面大部分为转载 http://www.turbolinux.com.cn中的Wiki
  
一.alsa-utils介绍
  
ALSA是kernel中的一个声音驱动程序.它包括alsa核心和其他声卡的驱动.
alsa-utils是alsa的一个工具包,里面包含有声卡测试和音频编辑的工具.
二.alsa-utils的安装
1.RPM包方式
  
Turbolinux 10.5,11版本已经包含有alsa-utils的rpm包,你可以直接安装:
# rpm -ivh alsa-utils-xxx.rpm
2.源码包方式
 
下载地址:
http://www.alsa-project.org/main/index.php/Download
 
源码包安装方法:

::

 # tar zxvf alsa-utils-1.0.6.tar.gz
 # cd alsa-utils-1.0.6
 # ./configure
 # make install


三.alsa-utils工具的使用
  
 alsa-utils包含的工具有:
 alsactl, aconnect, alsamixer, amidi, amixer, aplay, aplaymidi, arecord, arecordmidi, aseqnet, iecset, speaker-test
 1.alsactl的使用
  

alsactl用来对alsa声卡驱动进行一些高级的设置.系统中装有多个声卡,它也可以支持.
 有时在音量控制面板无法调整的选项,可以使用alsactl来实现.

alsactl可以将指定声卡的驱动程序设置信息保存到配置文件.或从配置文件中恢复指定
 声卡的驱动程序的设置信息.
 alsactl格式:

::
  
        alsactl [options] [store\|restore] <card # or id>
  
 选项:
  
        -h, --help
 打印帮助信息
  
        -f, --file
 指定使用的配置文件,默认为/etc/asound.state.
               Select   the   configuration   file  to  use.  The  default  is
               /etc/asound.state
  
  
        -F, --force
 与恢复命令一起使用.表示最大限度的恢复设置值.
  
  
        -d, --debug
 调试模式,输出更多细节信息.
  
        -v, --version
 打印alsactl版本号.
  
 文件:
 /etc/asound.state(或使用-f指定的文件)保存有声卡所有混合器的设置信息.
  
 示例:
 # rm /etc/asound.state -f
 # alsactl store
 2.aconnect的使用
  

aconnect是ALSA音序器的连接管理器.用来连接或断开ALSA音序器上的端口.端口是
 可以随意定义的.
 如,使用aconnect可以连接到任何由aseqview建立的设备端口.
  
 命令格式:

::

        aconnect [-d] [-options] sender receiver
        aconnect -i\|-o [-options]
        aconnect -x
  
 选项:
 连接管理
        -d, --disconnect
 断开连接.
  
        -e, --exclusive
 使用独占模式连接端口.发送和接收端口将不能再与其他端口相连.
  
        -r, --real queue
 将时间包的时间戳,转换为真实时间队列的当前值.
  
 显示端口
        -i, --input
 显示存在的输入端口.
  
        -o, --output
 显示存在的输出端口.
  
        -l, --list
 显示当前的连接状态.
  
 删除连接
        -x, --removeall
 删除所有连接.
  
 示例:
 连接端口64:0到65:0:
            % aconnect 64:0 65:0

这个连接是单向的,所有到发送端口64:0的数据,将被重定向到接收65:0端口.如果有另一个端口65:1,
 也使用64:0作为发送端口,则数据会同时发送到2个接收端口.
 端口连接时,使用:

::

            % aconnect -d 64:0 65:0
  
 地址也可以使用客户端的名字来代替:
            % aconnect External:0 Emu8000:1
  
 使用-i打印出输入端口信息.-o打印出输出端口信息.
            % aconnect -i
            client 0: ’System’ [type=kernel]
                0 ’Timer           ’
                1 ’Announce        ’
            client 64: ’External MIDI-0’ [type=kernel]
                0 ’MIDI 0-0        ’
  
 可以使用-x选项来清除所有的连接.
            % aconnect -x
 3.alsamixer的使用
  
 alsamixer是一个终端界面的声卡音量调节器.如图:
 命令格式:
        alsamixer [options]
  
 选项:
        -h, -help
 显示帮助信息.
  
        -c <card number or idenfication>
 指定需要设置的声卡.默认为0.
  
        -D <device identification>
 选择需要控制的调节器.
  
        -g
 设置界面颜色.
  
        -s
 最小化界面窗口.
  
 快捷键:
 进入alsamixer界面后,可以使用下面快捷键进行控制:
  
    常规控制:
 左右箭头或n,p 用来选择通道.
  
 上下箭头或+,- 同时调整选定通道的左右声道的音量.
  
 B,= 设置左右声道音量相同.
  
 M 静音当前通道.<,>分别对左,右声道静音.
  
 空格
 选择录音源.在选定的通道上按"空格",可以标记此通道为录音源.此操作仅限
 输入设备.插入键或";",删除键或"'"分别选定左右通道.
  
 L 刷新屏幕.
  
    快捷设置
        PageUp 增大5格音量.
  
        PageDown 减小5格音量.
        End 设置音量为0.
  
 分别调整左,右或整个通道的音量.
 Q,W,E 增大 左,右,通道 的音量.
  
 Z,X,C 减小 左,右,通道 的音量.
  
 alt-q,ESC 退出.
 4.amidi的使用
  
 amidi的作用是对ALSA的RawMIDI端口进行读写.
 amidi是一个命令行工具,允许你以独占模式向MIDI设备读/写数据.
 命令格式:
        amidi options
  
 选项:
 -h,-V,-l,-L 用于显示信息.
 -s,-r,-S,-d 用于发送/接收数据.
  
        -h, --help
 打印帮助信息.
  
        -V, --version
 打印版本号.
  
        -l, --list-devices
 打印所有硬件MIDI端口的列表.
  
        -L, --list-rawmidis
 打印所有RawMIDI定义.
  
        -p, --port=name
 设置要使用的ALSA RawMIDI端口.若不指定,则使用声卡0的端口0.
  
        -s, --send=filename
 发送指定文件的内容到MIDI端口.文件中必须包含raw MIDI命令(.syx,.mid文件).
  
        -r, --receive=filename
 将MIDI端口接收的数据写入指定文件.
  
        -S, --send-hex="..."
 发送十六进制字节到MIDI端口.
  
        -d, --dump
 从MIDI端口接收数据,然后以十六进制形式打印出来.
  
        -t, --timeout=秒
 指定超时,当端口无数据输出达到超时时长时,将停止接收数据.
  
 示例:
        amidi -p hw:0 -s my\_settings.syx
 发送my\_settings.syx终端MIDI命令到端口 hw:0.
  
        amidi -S ’
 发送XG复位到默认端口.
  
        amidi -p virtual -d
 建立一个虚拟RawMIDI端口,然后发送所有数据到这个端口.
 5.amixer的使用
  
 amixer是命令行的ALSA声卡驱动调节器工具.
 amixer用来在命令行控制ALSA的调节器,并且支持多声卡.
 amixer不加参数时,将打印默认声卡的设置信息.
  
 命令格式:
        amixer [-c card] [cmd]
  
 命令:
        help   显示语法帮助.
  
        info   显示调节器设备的信息.
  
        scontrols 显示调节器器的完整列表 .
  
        scontents 显示包含详细信息的调节器的完整列表.
  
        set or sset <SCONTROL> <PARAMETER> ...
 设置调节器信息.
  
        get or sget <SCONTROL>
 显示调节器的信息.
  
        controls 显示声卡控制器的信息.
        contents 显示完整的声卡控制器信息.
  
        cset <CONTROL> <PARAMETER> ...
 设置声卡控制器信息.
  
        cget <CONTROL> 显示声卡控制器的信息.
  
 选项:
        [-c card]
 选择指定的声卡.
  
        [-D device]
 选择需要控制的设备名.默认是 default.
  
        -h     Help
 显示帮助信息.
  
        -q
 安静模式.不输出设置结果.
  
 示例:
       # amixer -c 1 sset Line,0 80%,40% unmute cap
 设置第2块声卡的"line"的左声道音量为80%,右声道为40%,取消静音,并设置
 它为声音源.
  
       # amixer -c 2 cset numid=34 40%
 设置第34个声卡元素为40%.
 6.arecord,aplay的使用
  
 arecord,aplay是命令行的ALSA声卡驱动的录音和播放工具.
 arecord是命令行ALSA声卡驱动的录音程序.支持多种文件格式和多个声卡.
 aplay是命令行播放工具,支持多种文件格式.
  
 命令格式:
        arecord [flags] [filename]
        aplay [flags] [filename [filename]] ...
  
 选项:
        -h, --help
              帮助.
  
        --version
               打印版本信息.
  
        -l, --list-devices
               列出全部声卡和数字音频设备.
  
        -L, --list-pcms
               列出全部PCM定义.
  
        -D, --device=NAME
 指定PCM设备名称.
  
        -q --quiet
 安静模式.
  
        -t, --file-type TYPE
 文件类型(voc,wav,raw或au).
  
        -c, --channels=#
 设置通道号.
  
        -f --format=FORMAT
 设置格式.格式包括:S8  U8  S16\_LE  S16\_BE  U16\_LE
               U16\_BE  S24\_LE S24\_BE U24\_LE U24\_BE S32\_LE S32\_BE U32\_LE U32\_BE
               FLOAT\_LE  FLOAT\_BE  FLOAT64\_LE  FLOAT64\_BE   IEC958\_SUBFRAME\_LE
               IEC958\_SUBFRAME\_BE MU\_LAW A\_LAW IMA\_ADPCM MPEG GSM
  
        -r, --rate=#<Hz>
 设置频率.
  
        -d, --duration=#
 设置持续时间,单位为秒.
  
        -s, --sleep-min=#
 设置最小休眠时间.
  
        -M, --mmap
 mmap流.
  
        -N, --nonblock
 设置为非块模式.
  
        -B, --buffer-time=#
 缓冲持续时长.单位为微妙.
  
        -v, --verbose
 显示PCM结构和设置.
  
        -I, --separate-channels
 设置为每个通道一个单独文件.
  
 示例:
        aplay -c 1 -t raw -r 22050 -f mu\_law foobar
 播放raw文件foobar.以22050Hz,单声道,8位,mu\_law格式.
  
        arecord -d 10 -f cd -t wav -D copy foobar.wav
 以CD质量录制foobar.wav文件10秒钟.使用PCM的"copy".
 7.aplaymidi的使用
  
 aplaymidi用来播放标准的MIDI文件.
 aplaymidi是一个命令行工具,可以在一个或多个ALSA端口上播放MIDI
 文件.
  
 命令格式:
        aplaymidi -p client:port[,...] [-d delay] midifile ...
  
 选项:
        -h, --help
               输出帮助信息.
  
        -V, --version
               输出版本信息.
  
        -l, --list
               输出可以使用的输出端口列表.
  
        -p, --port=client:port,...
 设置端口.
  
        -d, --delay=seconds
 设置MIDI文件结束后,等待时长.
 8.arecordmidi的使用
  
 arecordmidi用于录制标准的MIDI文件.
 arecordmidi可以从一个或多个ALSA端口上,录制一个标准MIDI文件.
  
 命令格式:
        arecordmidi -p client:port[,...] [options] midifile
  
 选项:
        -h,--help
               打印帮助信息.
  
        -V,--version
               打印版本号.
  
        -l,--list
               打印可以使用的输入端口.
  
        -p,--port=client:port,...
 设置端口.
  
        -b,--bpm=beats
 设置MIDI文件的速率,默认为120 BPM.
  
        -f,--fps=frames
 设置帧率.
  
        -s,--split-channels
 设置每个通道将录制成一个单独的MIDI文件.
  
        -d,--dump
 在标准输出上,以文本形式显示接受到的事件信息
 9.aseqnet的使用
  
 aseqnet是ALSA调节器的网络连接工具.
 aseqnet是ALSA调节器的客户端程序,可以从网络上发送和接收事件数据包.
 网络上有主机A,主机B.A为服务器端,B为客户端.ALSA调节器系统必须同事运行
 在两个服务器上.然后建立服务器端口:
 hostA% aseqnet
  sequencer opened: 128:0
  
 在HostB上执行:
            hostB% aseqnet hostA
            sequencer opened: 132:0
  
 现在所有发送到HostA:128:0的数据将被传送到HostB:132:0上,反之亦然.
  
 命令格式:
        aseqnet [remotehost]
  
 选项:
        -p port
 指定TCP端口号或服务名.
  
        -s addr
 设置指定地址用于读操作.
  
        -d addr
 设置指定地址用于写操作.
        -v
 详细输出模式.
 10.iecset的使用
  
 设置或输出IEC958状态位.
 iecset是个小工具,通过ALSA的API,设置或输出IEC958(或称S/PDIF)状态位信息.
 直接运行iecset将输出当前IEC958的状态信息. 命令格式:
        iecset [options] [cmd arg...]
  
 选项:
        -D device
 设置需要打开的设备名.
  
        -c card
 设置需要打开的网卡名.
  
        -x
 输出AESx字节格式的状态信息.
  
        -i
 从标准输入读取命令信息,每行一个命令.
  
 命令:
        professional <bool>
 专业模式(true)或用户模式(false).
  
        audio <bool>
 音频模式(true).
  
        rate <int>
 采样频率,单位Hz.
  
        emphasis <int>
 设置加强值.0 = none, 1 = 50/15us, 2 = CCITT.
  
        lock <bool>
 速率锁.
  
        sbits <int>
 采样位:2 = 20bit, 4 = 24bit, 6 = undefined.
  
        wordlength <int>
 设置字长:0  =  No,  2 = 22-18 bit, 4 = 23-19 bit, 5 = 24-20
               bit, 6 = 20-16 bit.
  
        category <int>
 分类:值从0到0x7f.
  
        copyright <bool>
 设置是否包含版权.
  
        original <boo>
 原始标记:
  
 示例:
 输出当前IEC958信息.
 $ iecset
 Mode: consumer
 Data: audio
 Rate: 44100 Hz
 Copyright: permitted
 Emphasis: none
 Category: general
 Original: 1st generation
 Clock: 1000 ppm
  
 显示当前第1块声卡的IEC958状态位.
 $ iecset -Dhw:0
 Mode: consumer
 Data: non-audio
 Rate: 44100 Hz
 Copyright: permitted
 Emphasis: none
 Category: general
 Original: 1st generation
 Clock: 1000 ppm
  
 设置当前为用户模式,并打开"非音频"位.
 $ iecset pro off audio off
 Mode: consumer
 Data: non-audio
 Rate: 44100 Hz
 Copyright: permitted
 Emphasis: none
 Category: general
 Original: 1st generation
 Clock: 1000 ppm
 11.speaker-test的使用
  
 speaker-test是一个针对 ALSA驱动的声音测试工具.
 speaker-test可以分别对左右声道进行单独的测试.
  
 命令格式:
        speaker-test [-options]
  
 选项:
        -c \| --channels NUM
 设置通道数目.
  
        -D \| --device NAME
 设置使用的PCM设备名.
  
        -f \| --frequency FREQ
 设置声音频率.
  
        --help
 输出帮助信息.
  
        -b \| --buffer TIME
 设置缓冲区时长.0为使用最大的缓冲区大小.
  
        -p \| --period TIME
 设置节拍为多少微秒.
  
        -r \| --rate RATE
 设置音频率.
  
        -t \| --test pink\|sine\|wav
               -t pink 表示测试时使用噪声.
               -t sine 表示测试时使用音频信号声.
               -t wav 表示测试时使用WAV文件.
  
        -l \| --nloops COUNT
 设置测试循环的次数.
  
        -w \| --wavfile
 设置测试时播放的wav文件.
  
        -W \| --wavdir
 设置一个包含wav文件的目录.默认为/usr/share/sounds/alsa.
  
 示例:
 在一个音频接口上进行立体声测试
 #  speaker-test -Dplug:front -c2
  
 在两个音频接口上进行4声道测试.
 #  speaker-test -Dplug:surround40 -c4
  
 在立体声接口上进行5.1声道测试.
 # speaker-test -Dplug:surround51 -c6
  
 测试低音扬声器.
 # speaker-test -Dplug:surround51 -c6 -s1 -f75
  
 简要使用方法：
 降低百分之三的音量
 amixer -q set Master 3%-
 增加百分之三的音量，并取消静音
 amixer -q set Master unmute 3%+
 静音
  
 amixer -q set Master mute

 取消静音

 amixer -q set Master unmute
  
  
