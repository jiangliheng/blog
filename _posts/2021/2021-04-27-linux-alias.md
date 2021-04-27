---
layout: post
title: "Linux/Unix 常用的 15 类别名（alias）"
date: "2021-04-27 18:40"
category: Linux
tags: Linux Unix alias ops
author: jiangliheng
---
* content
{:toc}



# 背景

最近在整理 Linux 运维基线，整理记录下常用的 ```alias``` 设置。

# alias

```alias``` 命令用于设置指令的别名。用于简化较长的命令。

## 语法
```bash
alias [别名]=[指令名称]

示例：alias ls='ls --color=auto'
```

## 用法

```bash
# 列出所有别名
$ alias
alias ll='ls -l --color=auto'
alias ls='ls --color=auto'
alias mv='mv -i'
alias rm='rm -i'
......

# 设置别名，仅当前终端窗口生效，重启消失
$ alias ll='ls -l --color=auto'

# 查看别名设置
$ alias ll
alias ll='ls -l --color=auto'
或
$ which ll
alias ll='ls -l --color=auto'
	/usr/bin/ls

# 删除别名
$ unalias ll
# 查看删除的别名
$ alias ll
-bash: alias: ll: 未找到
```

## 永久生效

- ```系统级```别名设置，推荐在 ```/etc/profile.d``` 目录下创建 ```alias.sh```，```source /etc/profile.d/alias.sh``` 生效即可。不推荐:/etc/bashrc，/etc/profile。
- ```用户级```别名设置，可添加到```~/.bashrc```或```~/.bash_profile```中，```source ~/.bashrc``` 生效。

# 常用 alias

建议在 ```/etc/profile.d``` 目录下创建 ```alias.sh```。

```bash
## 1 - ls
# 带颜色设置
alias ls='ls --color=auto'
# 长格式输出
alias ll='ls -l --color=auto'
# 显示隐藏文件
alias l.='ls -ld .* --color=auto'
# 长格式显示所有文件，按照时间倒序并显示每个文件的容量
alias lh='ls -alths --color=auto'

## 2 - cd
# 避免日常手误
alias cd..='cd ..'
# 退出当前目录
alias ..='cd ..'
alias ...='cd ../../..'
alias ....='cd ../../../..'
alias .....='cd ../../../..'
alias .2='cd ../..'
alias .3='cd ../../..'
alias .4='cd ../../../..'
alias .5='cd ../../../../..'

## 3 - grep
# 带颜色设置
alias grep='grep --color=auto'
alias egrep='egrep --color=auto'
alias fgrep='fgrep --color=auto'

## 4 - bc
# 使用的标准数学库
alias bc='bc -l'

## 5 - mkdir
# 创建级联目录并打印
alias mkdir='mkdir -pv'

## 6 - diff
#  colordiff 替代 diff 命令，前提：yum install -y colordiff
alias diff='colordiff'

## 7 - date
alias now='date "+%Y-%m-%d %H:%M:%S.%s"'
# 获取秒和毫秒的时间戳，时间戳转换为时间：date "+%Y-%m-%d %H:%M:%S" -d @1619503315
alias timestamp='now; echo s: $(date +"%s"); echo ms: $(echo `expr \`date +%s%N\` / 1000000`)'

## 8 - vim
alias vi=vim
alias svi='sudo vi'

## 9 - 查看端口
alias ports='netstat -tulanp'

## 10 - 危险命令安全设置,
alias mv='mv -i'
alias cp='cp -i'
alias ln='ln -i'
# 不对根目录进行递归操作
alias rm='rm -i --preserve-root'
alias chown='chown --preserve-root'
alias chmod='chmod --preserve-root'
alias chgrp='chgrp --preserve-root'

## 11 - yum update
alias update='yum update'
alias updatey='yum -y update'

## 12 - 磁盘、内存、CPU、进程监控
alias psg='ps -ef | grep '
# 仅显示当前用户的进程
alias psme='ps -ef | grep $USER --color=always '

# 磁盘
alias du1='du -h -d 1'
alias du2='du -h -d 2'
alias du3='du -h -d 3'

# 内存信息
alias meminfo='free -h -l -t'
# cpu信息
alias cpuinfo='lscpu'

# 获取占用内存的进程排名
alias psmem='ps auxf | sort -nr -k 4'
alias psmem10='ps auxf | sort -nr -k 4 | head -10'

# 获取占用 cpu 的进程排名
alias pscpu='ps auxf | sort -nr -k 3'
alias pscpu10='ps auxf | sort -nr -k 3 | head -10'

# 磁盘、内存、端口情况
alias dfn='df -h; free -h -l -t; netstat -tulanp'

## 13 - 短命令
alias h='history'
alias j='jobs -l'

## 14 - git
alias g='git'
alias gr='git rm -rf'
alias gs='git status'
alias ga='g add'
alias gc='git commit -m'
alias gp='git push origin master'
alias gl='git pull origin master'

## 15 - other
alias ping="time ping"
alias nocomment='grep -Ev "^(#|$)"'
alias tf='tail -f '
```

> 微信公众号：daodaotest
