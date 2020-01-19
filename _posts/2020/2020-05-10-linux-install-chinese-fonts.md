---
layout: post
title: "Centos7 使用 Ansible 批量安装中文字体"
date: "2020-05-10 20:00"
category: Linux
tags: Linux Ansible fontconfig
author: jiangliheng
---
* content
{:toc}




# 需求背景

Centos7 下 Java 生成图片水印时中文乱码，原因是没有安装中文字体。

# 安装中文字体

以下是基于 Centos7 手动安装中文字体的详细步骤。当测试或者生产环境服务器比较多的时候，建议使用自动化运维工具。

```bash
# 安装字体库
$ yum -y install fontconfig

# 查看是否有中文字体
$ fc-list :lang=zh

# 创建中文字体目录
$ mkdir /usr/share/fonts/chinese

# 在 windows 的 C:\Windows\Fonts 目录下找到相应的字体 copy 到 chinese 目录下，这里以 宋体 为例
$ scp simsun.ttc simsunb.ttf root@xxxxx:/usr/share/fonts/chinese

# 查看是否有中文字体
$ fc-list :lang=zh
/usr/share/fonts/chinese/simsun.ttc: SimSun,宋体:style=Regular,常规
/usr/share/fonts/chinese/simsun.ttc: NSimSun,新宋体:style=Regular,常规
```

# Ansible 批量安装

通常测试或者生产环境服务器比较多，下面记录如何使用 Ansbile 来批量安装中文字体。

```bash
# ansbile playbook 执行
$ ansible-playbook fonts.yml

# 验证所有服务器是否生效
$ ansible all -m shell -a "fc-list :lang=zh"
sever01 | SUCCESS | rc=0 >>
/usr/share/fonts/chinese/simsun.ttc: SimSun,宋体:style=Regular,常规
/usr/share/fonts/chinese/simsun.ttc: NSimSun,新宋体:style=Regular,常规
sever02 | SUCCESS | rc=0 >>
/usr/share/fonts/chinese/simsun.ttc: SimSun,宋体:style=Regular,常规
/usr/share/fonts/chinese/simsun.ttc: NSimSun,新宋体:style=Regular,常规
......
```

fonts.yml 内容：
```yml
---
- name: Install Chinese Fonts.
  hosts: all
  remote_user: root
  become: yes
  become_method: sudo
  become_user: root
  roles:
    - fonts
```

ansible playbook 目录结构（删除了无用目录）：
```bash
$ tree roles/fonts
roles/fonts
├── files
│   ├── simsun.ttc
│   └── simsunb.ttf
└── tasks
    └── main.yml

2 directories, 3 files
```

task/main.yml 内容：
```yml
---
# tasks file for fonts

- name: install fontconfig.
  yum:
    name: "{{ item }}"
    state: installed
  with_items:
    - fontconfig
  ignore_errors: true

- name: mkdir /usr/share/fonts/chinese.
  file:
    path: /usr/share/fonts/chinese
    state: directory
    mode: 0755

- name: Copy fonts to agent.
  copy:
    src: "{{ item }}"
    dest: /usr/share/fonts/chinese
  with_items:
    - simsun.ttc
    - simsunb.ttf
```

> 微信公众号：daodaotest
