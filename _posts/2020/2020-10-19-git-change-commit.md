---
layout: post
title: "Git 修改已提交 commit 的信息"
date: "2020-10-19 18:00"
category: Git
tags: Git
author: jiangliheng
---
* content
{:toc}



# 背景

由于 Github 和公司 Git 使用账号不一样，偶尔没注意，提交出错后就需要修改 commit 信息。

# 修改最后一次提交 commit 的信息

```bash
# 修改最后一次提交的 commit 信息
$ git commit --amend --message="modify message by daodaotest" --author="jiangliheng <jiang_liheng@163.com>"

# 仅修改 message 信息
$ git commit --amend --message="modify message by daodaotest"

# 仅修改 author 信息
$ git commit --amend --author="jiangliheng <jiang_liheng@163.com>"
```

# 修改历史 commit 的信息

操作步骤：
- ```git rebase -i <commit id>``` 列出 commit 列表
- 找到需要修改的 commit 记录，把 ```pick``` 修改为 ```edit``` 或 ```e```，```:wq``` 保存退出
- 修改 commit 的具体信息```git commit --amend```，保存并继续下一条```git rebase --continue```，直到全部完成
- 中间也可跳过或退出```git rebase (--skip | --abort)```

```bash
# 列出 rebase 的 commit 列表，不包含 <commit id>
$ git rebase -i <commit id>
# 最近 3 条
$ git rebase -i HEAD~3
# 本地仓库没 push 到远程仓库的 commit 信息
$ git rebase -i

# vi 下，找到需要修改的 commit 记录，```pick``` 修改为 ```edit``` 或 ```e```，```:wq``` 保存退出
# 重复执行如下命令直到完成
$ git commit --amend --message="modify message by daodaotest" --author="jiangliheng <jiang_liheng@163.com>"
$ git rebase --continue

# 中间也可跳过或退出 rebase 模式
$ git rebase --skip
$ git rebase --abort
```

# 批量修改历史 commit 信息

创建批量脚本```changeCommit.sh```：

```bash
$ cat changeCommit.sh
#!/bin/sh

git filter-branch --env-filter '

# 之前的邮箱
OLD_EMAIL="jiangliheng@126.com"
# 修改后的用户名
CORRECT_NAME="jiangliheng"
# 修改后的邮箱
CORRECT_EMAIL="jiangliheng@163.com"

if [ "$GIT_COMMITTER_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_COMMITTER_NAME="$CORRECT_NAME"
    export GIT_COMMITTER_EMAIL="$CORRECT_EMAIL"
fi
if [ "$GIT_AUTHOR_EMAIL" = "$OLD_EMAIL" ]
then
    export GIT_AUTHOR_NAME="$CORRECT_NAME"
    export GIT_AUTHOR_EMAIL="$CORRECT_EMAIL"
fi
' --tag-name-filter cat -- --branches --tags
```

执行脚本成功后，强制推送到远程服务器：
```bash
$ git push --force --tags origin 'refs/heads/*'
```

> 微信公众号：daodaotest
