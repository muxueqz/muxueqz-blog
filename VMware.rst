VMware for Linux键盘失灵临时解决办法
##########################################################################################################################################
:date: 2009-08-01 11:33:33
:category: 系统运维
:slug: VMware

::

 VMware Workstation for Linux，用着用着经常会发现，回到Host系统后有时Ctrl、Alt等按键失效。
 目前只能临时解决一下，运行setxkbmap一下，重新加载键盘布局。
 注意：如果你加载过.Xmodmap，则需要在运行setxkbmap后重新xmodemap .Xmodmap
