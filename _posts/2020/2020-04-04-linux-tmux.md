---
layout: post
title: "Linux 下终端复用利器 tmux"
date: "2020-04-04 20:00"
category: Linux
tags: Linux tmux
author: jiangliheng
---
* content
{:toc}

tmux 是一个终端复用器（terminal multiplexer）。



# 简介
> tmux 是一个终端复用器类自由软件，功能类似 GNU Screen，但使用 BSD 许可发布。用户可以通过 tmux 在一个终端内管理多个分离的会话，窗口及面板，对于同时使用多个命令行，或多个任务时非常方便。     --- 维基百科

# 基本概念

tmux 的三个重要概念：**会话（session）**，**窗口（window）**，**窗格（pane）**。

一个**会话（session）** 可以有多个**窗口（window）**，一个**窗口（window）**又可以有多个**窗格（pane）**。

# 安装

```bash
# macOS
$ brew install tmux

# centos
$ yum install tmux
```

# 前缀键

tmux 的快捷键都要通过前缀键才可以使用。默认的前缀键是```Ctrl+b```，即先按下```Ctrl+b```进入快捷键模式，再按快捷键才会生效。

举例：分离会话的快捷键是```Ctrl+b d```。用法是，在 tmux 窗口下，先按下```Ctrl+b```，再按下```d```，就会分离会话，进入正常命令行模式。

# 会话管理

- ```tmux```：创建一个无名称的会话
- ```tmux new -s daodaotest```：创建名为 daodaotest 的会话
- ```tmux new -s daodaotest -d```：在后台创建名为 daodaotest 的会话
- ```tmux detach```：分离会话
- ```tmux ls```：显示会话列表
- ```tmux a```：接入最后一次会话
- ```tmux a -t daodaotest```：接入 daodaotest 会话
- ```tmux kill-session```：关闭最后一次会话
- ```tmux kill-session -t 0```：使用会话编号杀死会话
- ```tmux kill-session -t daodaotest```：使用会话名称杀死会话
- ```tmux kill-session -a -t daodaotest```：关闭除 daodaotest 外的所有会话
- ```tmux kill-server```：关闭所有会话
- ```tmux switch -t 0```：使用会话编号切换会话
- ```tmux switch -t daodaotest```：使用会话名称切换会话
- ```tmux rename-session -t daodaotest daodaotest2```：重命名会话名称
- ```exit``` 或 ```Ctrl+d```：退出会话

**会话快捷键**
- ```Ctrl+b s```：列出会话，可进行切换
- ```Ctrl+b d```：分离当前会话
- ```Ctrl+b $```：重命名当前会话

# 窗口管理

- ```tmux new-window```：新建一个新窗口
- ```tmux new-window -n daodaotest```：新建一个 daodaotest 名称的新窗口
- ```tmux select-window -t 0~9```：切换到指定编号的窗口
- ```tmux select-window -t daodaotest```：切换到 daodaotest 的窗口
- ```tmux rename-window daodaotest2```：重命名当前窗口为：daodaotest2

**窗口快捷键**
- ```Ctrl+b c```：新建一个新窗口
- ```Ctrl+b ,```：重命名当前窗口
- ```Ctrl+b w```：列出所有窗口，可进行切换
- ```Ctrl+b n```：进入下一个窗口
- ```Ctrl+b p```：进入上一个窗口
- ```Ctrl+b l```：进入之前操作的窗口
- ```Ctrl+b 0~9```：选择编号0~9对应的窗口
- ```Ctrl+b .```：修改当前窗口索引编号
- ```Ctrl+b '```：切换至指定编号（可大于9）的窗口
- ```Ctrl+b f```：根据显示的内容搜索窗格
- ```Ctrl+b &```：关闭当前窗口

# 窗格管理

- ```tmux sp -h```：水平方向创建窗格
- ```tmux sp```：垂直方向创建窗格
- ```tmux select-pane -U ```：光标切换到上方窗格
- ```tmux select-pane -D ```：光标切换到下方窗格
- ```tmux select-pane -L ```：光标切换到左边窗格
- ```tmux select-pane -R ```：光标切换到右边窗格
- ```tmux swap-pane -U ```：当前窗格上移
- ```tmux swap-pane -D ```：当前窗格下移

**窗格快捷键**
- ```Ctrl+b %```：水平方向创建窗格
- ```Ctrl+b "```：垂直方向创建窗格
- ```Ctrl+b Up|Down|Left|Right```：根据箭头方向切换窗格
- ```Ctrl+b q```：显示窗格编号
- ```Ctrl+b o```：顺时针切换窗格
- ```Ctrl+b }```：与下一个窗格交换位置
- ```Ctrl+b {```：与上一个窗格交换位置
- ```Ctrl+b x```：关闭当前窗格
- ```Ctrl+b space(空格键)```：重新排列当前窗口下的所有窗格
- ```Ctrl+b !```：将当前窗格置于新窗口
- ```Ctrl+b Ctrl+o```：逆时针旋转当前窗口的窗格
- ```Ctrl+b t```：在当前窗格显示时间
- ```Ctrl+b z```：放大当前窗格(再次按下将还原)
- ```Ctrl+b i```：显示当前窗格信息

# 使用场景

## 后台运行程序

在做自动化部署脚本时，远程执行目标服务器 ```xStart.sh``` 脚本，来后台启动 java 应用。脚本如下：

```bash
# 启动应用
start() {
  ......
  if [ "X$pid" = "X" ]; then
      # 关闭之前终端
      tmux kill-session -t $SYSTEM_NAME-$PORT
      # 创建终端
      tmux new -s $SYSTEM_NAME-$PORT -d
      # 终端启动服务
      tmux send -t $SYSTEM_NAME-$PORT "cd $PIDDIR;nohup java $JAVA_OPTS -jar $SYSTEM_NAME*.jar $SPRING_ACTIVE --server.port=$PORT $EXT_OPTS >/dev/null 2>&1 &" ENTER
  else
      echo "$SYSTEM_NAME is already running."
      exit 1
  fi
  ......
}
```

## 大量数据迁移

在迁移 maven 私服 nexus 时，nexus 数据大概有 13G 左右，在使用```scp```进行迁移，终端超时关闭后就会中断。操作命令如下：

```bash
# 创建 tmux 会话
$ tmux new -s nexus-scp

# 在 tmxu 会话中，执行 scp 命令
$ scp -R nexus@x.x.x.x:~/nexus .

# 分离会话，后台执行即可
$ Ctrl+b d
```

## 快速恢复工作现场

日常工作中，每天都要查询某些日志或者监控某些指标时，可以通过```tmux```保存现场，下次快速恢复工作现场，提高工作效率。

**nginx 操作现场**
![-w1233](/assets/images/tmux/15860081927732.jpg)

> 微信公众号：daodaotest
