解决s5与opera 11的兼容性问题
##########################################################################################################################################
:date: 2011-06-16 03:20:57
:category: 系统运维
:slug: s5-opera-fix

关于s5的介绍：
英文：\ `wiki`_
中文：\ `http://server.linjunhalida.com/blog/article/rst2s5/S5/`_
摘述：
为什么要用S5?

作为乐于分享的开发人员, 总是免不了想向其他人介绍一些好玩的东西.
这个时候, 我们需要写幻灯片, 用展示自己要讲的东西. 其他人往往使用PPT,
PPT是很好, 但是它是二进制专有格式的, 无法以文本方式编辑,
也无法进行版本管控, 离开了PPT程序也无法打开, 无法满足我们的需求.
那么我们应该用什么工具来做幻灯片呢? 在wikipedia上面有 slide工具列表
来介绍现有的幻灯片工具, 我选择的是 S5 .

那么S5有什么好处呢?

纯文本

html

纯文本, 直观, 清爽, 方便编写以及后期的维护. 文件本身其实是html格式的,
利用其他javascript, css之类的设置弄成slide的效果. 本身方便分发以及发布,
再也不用超级大的PPT工具啦!

``sed -i s/"('Opera') > -1"/"('Opera') > 9"/g ui/default/slides.js``

即可

.. _wiki: http://en.wikipedia.org/wiki/S5_(file_format)
.. _`http://server.linjunhalida.com/blog/article/rst2s5/S5/`: http://server.linjunhalida.com/blog/article/rst2s5/S5/
