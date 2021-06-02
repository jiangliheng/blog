---
layout: post
title: "Centos7 安装 Python3 及配置国内源、虚拟环境"
date: "2021-06-02 20:00"
category: Python
tags: Python virtualenv virtualenvwrapper
author: jiangliheng
---
* content
{:toc}



# 安装

```bash
# 安装 python3
$ sudo yum install -y epel-release
$ sudo yum install -y python3

# 升级 pip 为最新版本
$ sudo pip3 install pip -U
```

# 设置国内镜像源

```bash
# 查看当前源地址
$ pip config list | grep global.index-url

# 设置 pip 为清华源
$ pip config set global.index-url https://pypi.tuna.tsinghua.edu.cn/simple
Writing to /root/.config/pip/pip.conf

# 确认源地址
$ pip config list | grep global.index-url
global.index-url='https://pypi.tuna.tsinghua.edu.cn/simple'
```

**常用国内源：**
- 清华：https://pypi.tuna.tsinghua.edu.cn/simple
- 阿里云：https://mirrors.aliyun.com/pypi/simple
- 豆瓣：https://pypi.doubanio.com/simple

# pip 临时指定源

```bash
# 临时使用 https 源
$ pip install -i https://pypi.tuna.tsinghua.edu.cn/simple pip -U

# 临时使用 http 源，需要加 --trusted-host xxxx，信任不安全源
$ pip install -i http://pypi.doubanio.com/simple pip -U --trusted-host pypi.doubanio.com
```

# virtualenv 虚拟环境

```bash
# 安装 virtualenv
$ pip install virtualenv

# 创建名称为 env3 虚拟环境
$ python3 -m venv env3

# 设置环境变量
$ echo 'source ~/env3/bin/activate' >> ~/.bashrc

# 激活虚拟环境
$ source ~/env3/bin/activate

# 升级虚拟环境的 pip
$ pip install pip -U

# 关闭虚拟环境
$ deactivate
```

# virtualenvwrapper 管理虚拟环境

```bash
# 安装 virtualenvwrapper
$ pip install virtualenvwrapper

# 创建虚拟环境目录
$ mkdir -p ~/.envs

# 设置环境变量
$ cat << EOF >> ~/.bashrc
VIRTUALENVWRAPPER_PYTHON=/usr/bin/python3
export WORKON_HOME=~/.envs
source /usr/local/bin/virtualenvwrapper.sh
EOF

# 生效环境变量
$ source ~/.bashrc

# 创建虚拟环境
$ mkvirtualenv env1
$ mkvirtualenv env2

# 查看虚拟环境
$ lsvirtualenv
或
$ workon

# 切换虚拟环境
$ workon env1

# 退出虚拟环境
$ deactivate

# 删除虚拟环境，删除前需要先 deactivate
$ rmvirtualenv env1 env2
```

> 微信公众号：daodaotest
