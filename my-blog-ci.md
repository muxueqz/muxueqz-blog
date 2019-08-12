Title: 静态博客持续集成大法
Date: 2019-08-09 22:20
Tags: DevOps, 系统运维, nim
Slug: my-blog-ci
Author: muxueqz

# 前言
因为最近自己造了[静态博客的轮子](/a-small-static-site-generator.html) ， 之前看到可以用CI来自动build页面并部署，一直想试试，这次就顺便试试。

本来一开始选择的是开源界流行的`travis-ci`，不过在尝试过程中忽然不再build，我怀疑是超出一定的次数限制，

因此转向[Drone Cloud](https://cloud.drone.io/)。

# 那么，通过Drone CI自动部署需要几步？
配置pipeline编排:
1. 下载静态博客构建代码，这里采用我自己写的[kun](https://github.com/muxueqz/kun)
1. 下载代码，我把它放在了 https://github.com/muxueqz/muxueqz-blog.git
1. 编译静态博客构建工具
1. 构建静态页面
1. 通过git push部署到git仓库，我用的是国内的coding.net，需要注意的是：
    * SSH密钥通过Drone的secret来存取，而不暴露在公开的git仓库中（因为`.drone.yml`是在公开的git仓库中）
    * 需要在coding.net的仓库中添加Deploy Key并允许推送
    * 因为我的HTML是托管在coding.net上，所以不能使用Drone的Github Pages插件，找到一个`drone-git-push`，但试了半天都用不了，看Drone的bash插件写起来很简单，索性自己写一个。


# 编写Drone配置文件
在文章源码仓库（对我来说是https://github.com/muxueqz/muxueqz-blog.git）中增加`.drone.yml`文件，内容如下：

```yaml
---
kind: pipeline
name: default

clone:
  disable: true

steps:
- name: clone
  image: docker:git
  commands:
  # 下载静态博客构建代码
  - git clone https://github.com/muxueqz/kun.git
  # 下载代码
  - git clone https://github.com/muxueqz/muxueqz-blog.git

- name: build
  image: nimlang/nim
  commands:
  # 编译静态博客构建工具
  - nimble install -y nwt markdown
  - cd kun
  - nim c -d:release build.nim
  - rm -rf srcs
  - mv -v ../muxueqz-blog/ srcs
  - mkdir -pv public/tags/
  # 构建静态页面
  - ./build

# 通过git push部署到git仓库，我用的是国内的coding.net
- name: git_push
  image: muxueqz/drone-git-push-by-bash
  settings:
    branch: master
    remote: git@git.coding.net:muxueqz/muxueqz.git
    force: true
    commit: true
    path: ./kun/public
    ssh_key:
      from_secret: GIT_PUSH_SSH_KEY
```

## 安装Drone客户端
```bash
wget https://github.com/drone/drone-cli/releases/download/v1.1.0/drone_linux_amd64.tar.gz
tar xvf drone_linux_amd64.tar.gz
cp -v drone /usr/bin/
```

## 配置Drone Secret
```bash
# 将ssh私钥添加到Drone的secret配置中
drone secret add --repository muxueqz/muxueqz-blog --name GIT_PUSH_SSH_KEY --data @/tmp/iam_logs/pages_id_rsa
drone sign muxueqz/muxueqz-blog  --save
```

# 编写Drone Git Push插件
其实非常简单，ENTRYPOINT脚本这样写就可以了：
```bash
#!/bin/sh

mkdir -pv $HOME/.ssh
echo "${PLUGIN_SSH_KEY}" > $HOME/.ssh/id_rsa
chmod 400 $HOME/.ssh/id_rsa
cd ${PLUGIN_PATH}

export GIT_SSH_COMMAND='ssh -o StrictHostKeyChecking=no'
git config --global user.name $DRONE_COMMIT_AUTHOR
git config --global user.email $DRONE_COMMIT_AUTHOR_EMAIL
git init
git add .
[ -z $PLUGIN_COMMIT_MESSAGE ] && PLUGIN_COMMIT_MESSAGE='push'
[ $PLUGIN_FORCE == "true" ] && PLUGIN_FORCE='--force'
git commit . -m "${PLUGIN_COMMIT_MESSAGE}"
git remote add origin ${PLUGIN_REMOTE}
git push --set-upstream origin master ${PLUGIN_FORCE}
```

完整Docker容器在这里：[drone-git-push-by-bash](https://github.com/muxueqz/drone-git-push-by-bash)


# CI Dashboard效果

![image](https://user-images.githubusercontent.com/730639/62789429-78ef0a00-bafb-11e9-949a-cef436e77ef4.png)

# 总结
* Drone作为基于Docker的CI/CD平台，和Docker结合得非常好，可以直接调用Docker容器中的命令，用起来很爽。
* 基于bash的自定义插件写起来也很方便，分分钟就写好了，以CI的角度，似乎都没有必要用Go写插件

本文通过Drone CI构建部署而成。
