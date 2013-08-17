Apt的gpg错误
##########################################################################################################################################
:date: 2009-09-29 10:50:40
:category: 系统运维
:slug: apt-gpg-error

问题描述：
 W: GPG签名验证错误： http://www.debian-multimedia.org sid Release:
由于没有公钥，下列签名无法进行验证： NO\_PUBKEY 07DC563D1F41B907
 对于错误提示：
 引用:
 正在读取软件包列表... 完成
 W: GPG签名验证错误： http://deb.opera.com unstable Release:
由于没有公钥，下列签名无法进行验证： NO\_PUBKEY 033431536A423791
 W: 您可能需要运行 apt-get update 来解决这些问题

需要把上面两行命令中的“4F6C1E86”替换成“NO\_PUBKEY”后面的字串的最后8位，也即：
 $ gpg --keyserver subkeys.pgp.net --recv 6A423791
 $ gpg --export --armor 6A423791 \| sudo apt-key add -
 总结： 除此之外，可能还有软件源的问题，实在不行就换个源试试。
