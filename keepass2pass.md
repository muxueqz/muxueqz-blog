Date: 2019-09-13 11:34

Tags: 系统运维

Title: 迁移密码管理器始末，从KeePass到Pass

Slug: keepass2pass

## 前言
我以前是用KeePass来管理密码的，包括浏览器密码、Git密码，后来嫌KeePass在Linux上有些重（KeePassXC也只是好了一些，好吧，我承认是我斤斤计较），
后来看到有一些新的密码管理器，比如[pass](https://passwordstore.org)，一个用shell实现的密码管理器，非常符合我的需求，
所以呢，最近就把KeePass中的数据迁移到了[pass](https://passwordstore.org)中，
迁移起来也很简单，在KeePassXC中导出CSV格式的数据，再通过[pass-import](https://github.com/roddhjav/pass-import)导入即可：
```bash
pass import keepassxc-csv file.csv
```

## 浏览器密码管理
接下来，我们让浏览器可以通过Pass来自动登陆，
在FireFox上可以用[passff](https://github.com/passff/passff)这个扩展，而在Chrome呢，可以用[browserpass](https://github.com/browserpass/browserpass-extension)

## Git密码管理
最后，作为一个开发者，经常使用Git，而公司的Git是使用HTTPS协议的，
虽然可以将用户名密码保存在gitconfig中自动登陆，但这样未免太不安全，
以前用KeePass时是用[git-credential-keepasshttp](https://github.com/muxueqz/git-credential-keepasshttp.git)来自动获取用户名密码登陆的，
那迁移到`pass`后用什么呢？
答案是[git-credential-pass](https://github.com/muxueqz/git-credential-pass)！

那么，怎么使用呢？

```bash
# 首先，获取代码
git clone https://github.com/muxueqz/git-credential-pass
# 赋予执行权限
chmod +x git-credential-pass/git-credential-pass.py
# 然后把脚本放到可执行的命令中
cp -v git-credential-pass/git-credential-pass.py /usr/bin/
# [可选]配置映射（可以映射到名字不同的用户密码）
cp -v git-credential-pass/git-credential-pass.ini ~/.config/
# 配置git使用这个脚本
git config --global credential.helper git-credential-pass.py
```

大功告成！

现在使用`git pull/git push`就不再提示输入用户名密码啦！
