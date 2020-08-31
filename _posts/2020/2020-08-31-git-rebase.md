---
layout: post
title: "Git 合并多个 commit，保持历史简洁"
date: "2020-08-31 21:00"
category: Git
tags: Git rebase
author: jiangliheng
---
* content
{:toc}



# 背景

开发过程中，本地通常会有无数次 commit ，可以合并“相同功能”的多个 commit，以保持历史的简洁。

# git rebase

```bash
# 从HEAD版本开始往过去数3个版本
$ git rebase -i HEAD~3

# 合并指定版本号（不包含此版本）
$ git rebase -i [commitid]
```

说明：
- ```-i（--interactive）```：弹出交互式的界面进行编辑合并
- ```[commitid]```：要合并多个版本之前的版本号，注意：```[commitid]``` 本身不参与合并

指令解释（交互编辑时使用）：
- p, pick = use commit
- r, reword = use commit, but edit the commit message
- e, edit = use commit, but stop for amending
- s, squash = use commit, but meld into previous commit
- f, fixup = like "squash", but discard this commit's log message
- x, exec = run command (the rest of the line) using shell
- d, drop = remove commit

# 合并步骤

1. 查看 log 记录，使用```git rebase -i```选择要合并的 commit
2. 编辑要合并的版本信息，保存提交，多条合并会出现多次（可能会出现冲突）
3. 修改注释信息后，保存提交，多条合并会出现多次
4. 推送远程仓库或合并到主干分支

## 查看 log

```bash
$ git log --oneline
291e427 update website
8c8f3f4 update website
1693a6f update clear-logs.sh version
3759b84 update clear-logs.sh
fc36a2a add links
1d795e6 fix && update clear-logs.sh 0.0.2
9536dab add dingtalk script
3a51aaa fix shellcheck problem
2db6ad3 add clear logs scripts
e57b0e6 fix && add batch del
17cb931 fix && add batch del
cf7e875 add redis script
fe4bbcb Initial commit
```

## 编辑要合并版本

```
# 指定要合并版本号，cf7e875 不参与合并，进入 vi 编辑器
$ git rebase -i cf7e875
pick 17cb931 fix && add batch del
pick e57b0e6 fix && add batch del
pick 2db6ad3 add clear logs scripts
pick 3a51aaa fix shellcheck problem
pick 9536dab add dingtalk script
pick 1d795e6 fix && update clear-logs.sh 0.0.2
pick fc36a2a add links
pick 3759b84 update clear-logs.sh
pick 1693a6f update clear-logs.sh version
pick 8c8f3f4 update website

# Rebase cf7e875..291e427 onto cf7e875 (10 commands)
#
# Commands:
# p, pick = use commit
# r, reword = use commit, but edit the commit message
# e, edit = use commit, but stop for amending
# s, squash = use commit, but meld into previous commit
# f, fixup = like "squash", but discard this commit's log message
# x, exec = run command (the rest of the line) using shell
# d, drop = remove commit
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

将 commit 内容编辑如下：
```bash
# 把 e57b0e6 合并到 17cb931，不保留注释；把 1693a6f 合并到 3759b84
pick 17cb931 fix && add batch del
f e57b0e6 fix && add batch del
pick 2db6ad3 add clear logs scripts
pick 3a51aaa fix shellcheck problem
pick 9536dab add dingtalk script
pick 1d795e6 fix && update clear-logs.sh 0.0.2
pick fc36a2a add links
pick 3759b84 update clear-logs.sh
s 1693a6f update clear-logs.sh version
pick 8c8f3f4 update website
```

然后```:wq```保存退出后是注释界面：
```
# This is a combination of 2 commits.
# This is the 1st commit message:

update clear-logs.sh

# This is the commit message #2:

update clear-logs.sh version

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Tue Jul 28 18:25:57 2020 +0800
#
# interactive rebase in progress; onto cf7e875
# Last commands done (9 commands done):
#    pick 3759b84 update clear-logs.sh
#    s 1693a6f update clear-logs.sh version
# Next command to do (1 remaining command):
#    pick 8c8f3f4 update website
# You are currently editing a commit while rebasing branch 'master' on 'cf7e875'.
#
# Changes to be committed:
#	modified:   logs/README.md
#	modified:   logs/clear-logs.sh
#
```

编辑注释信息，保存退出```:wq```即可完成 commit 的合并：
```bash
update clear-logs.sh
```

## 查看合并后的 log

```bash
$ git log --oneline
47e7751 update website
4c2316c update clear-logs.sh
73f082e add links
56adcf2 fix && update clear-logs.sh 0.0.2
ebf3786 add dingtalk script
6e81ea7 fix shellcheck problem
64ca58f add clear logs scripts
9327def fix && add batch del
cf7e875 add redis script
fe4bbcb Initial commit
```

## 推送远程

```bash
# 由于我是一个人开发，故强制推送到远程仓库
$ git push --force origin master
```

# 冲突解决

在 ```git rebase``` 过程中，可能会存在冲突，此时就需要解决冲突。

错误提示信息： ```git rebase -i resumeerror: could not apply ...```。

```bash
# 查看冲突
$ git status

# 解决冲突之后，本地提交
$ git add .

# rebase 继续
$ git rebase --continue
```

> 微信公众号：daodaotest
