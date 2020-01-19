---
layout: post
title: "发送钉钉消息 Shell 脚本"
date: "2020-06-10 8:30"
category: ops-scripts
tags: Linux Shell dingalk send-ding.sh
author: jiangliheng
---
* content
{:toc}



# 需求背景

生产环境定时监控凌晨跑批生成文件，并获取业务汇总信息发送到运维钉钉群。

主要原因还是懒得半夜监控~

# 变更记录

* Version 0.0.1 2020/06/08
    * 发送钉钉消息，支持 text，markdown 两种类型消息

# 选项

```
sh send-ding.sh [options] <value> ...

    * -a <value>                 钉钉机器人 Webhook 地址的 access_token
    * -t <value>                 消息类型：text，markdown
    & -T <value>                 title，首屏会话透出的展示内容；消息类型（-t）为：markdown 时
    * -c <value>                 消息内容，content或者text
    & -m <value>                 被@人的手机号（在 content 里添加@人的手机号），多个参数用逗号隔开；如：138xxxx6666，182xxxx8888；与是否@所有人（-A）互斥，仅能选择一种方式
    & -A                         是否@所有人，即 isAtAll 参数设置为 ture；与被@人手机号（-m）互斥，仅能选择一种方式
      -v, --version              版本信息
      --help                     帮助信息

    * 表示必输，& 表示条件必输，其余为可选
```

# 示例

```
1. 发送 text 消息类型，并@指定人
sh send-ding.sh -a xxx -t text -c "我就是我, 是不一样的烟火" -m "138xxxx6666,182xxxx8888"

2. 发送 markdown 消息类型，并@所有人
sh send-ding.sh -a xxx -t markdown -T "markdown 测试标题" -c "# 我就是我, 是不一样的烟火" -A
```

# 使用场景

## 定时监控跑批结果文件生成，发送汇总信息

由于跑批任务大概在凌晨 2:15 分左右完成，故设置 2:20 开始检测，每 30 分钟（可调整）钉钉告警一次未获取到，之后一直检测，直到检测到文件生成。

crontab 定时任务设置
```bash
$ crontab -e
20 2 * * * sh /ops-scripts/checkPreSettle.sh
```

定时监控文件并反馈汇总信息脚本 checkPreSettle.sh：
```bash
#!/bin/bash
# 定时检查预对账文件，并输出汇总信息

# 昨天日期
DAY=$(date -d yesterday +%Y%m%d)
# 文件路径
PRE_SETTLE_PATH="/data/${DAY}"
# 预对账文件
PRE_SETTLE="${PRE_SETTLE_PATH}/${DAY}_pre_settle.csv"
# 预对账 md5 文件
PRE_SETTLE_MD5="${PRE_SETTLE_PATH}/${DAY}_pre_settle.md5"

# 检测次数，即 60*30/60秒=30分钟
COUNT=60
# 间隔扫描时间
SLEEP_TIME=30
# HOSTNAME
HOSTNAME=${HOSTNAME}
# IP
IP=$(ifconfig -a|grep inet|grep -v 127.0.0.1|grep -v inet6|awk '{print $2}')

# send-ding.sh
SEND_DING_SH="/ops-scripts/send-ding.sh"

# 开始时间
START_TIME=$(date "+%Y-%m-%d %H:%M:%S")

# ACCESS_TOKEN
ACCESS_TOKEN=xxxxxxx
# 钉钉消息标题
TILE="预对账汇总信息\n"

# 计数
i=1
# 循环检查文件是否存在，SLEEP_TIME 秒检查一次，总共检查 COUNT*SLEEP_TIME 秒
while [ $i -le ${COUNT} ];do
	if [ -f "${PRE_SETTLE}" ] && [ -f "${PRE_SETTLE}" ]; then
		amd5=$(md5sum "${PRE_SETTLE}" | awk '{print $1}')
		xmd5=$(cat "${PRE_SETTLE_MD5}")
		if [ "X${amd5}" = "X${xmd5}" ]; then
			# 获取文件最后一行
			result=$(sed -n '$p' "${PRE_SETTLE}")
			# 发送钉钉
			sh "${SEND_DING_SH}" -a "${ACCESS_TOKEN}" -t markdown -T "${TILE}" -c "# 【SUCCESS】预对账汇总信息(条数，结算总金额)：${result}\n #### **HOSTNAME**：${HOSTNAME} \n#### **IP**：${IP} \n#### **文件路径**：${PRE_SETTLE}\n  #### **当前时间**：$(date "+%Y-%m-%d %H:%M:%S")\n" -A
			exit 0
		fi
	fi

	# 继续计数
	(( i++ ))
	# 间隔 SLEEP_TIME 秒
	sleep ${SLEEP_TIME}
done

# 检测 SLEEP_TIME * COUNT 秒后，仍然没有检查到，则告警
sh "${SEND_DING_SH}" -a "${ACCESS_TOKEN}" -t markdown -T "${TILE}" -c "# 【FAIL】经过 ${COUNT}*${SLEEP_TIME} 秒后，仍然未检查到预对账文件：${PRE_SETTLE}\n #### **HOSTNAME**：${HOSTNAME} \n#### **IP**：${IP} \n #### **开始检查时间**：${START_TIME}\n #### **结束检查时间**：$(date "+%Y-%m-%d %H:%M:%S")\n ## **说明**：会持续检查（间隔时间：${COUNT}*${SLEEP_TIME} 秒），直到检查到预对账文件生成！！！" -A

# 继续调起
sh "$0"
```

# 脚本

```bash
#!/bin/bash
#================================================================
# HEADER
#================================================================
#    Filename         send-ding.sh
#    Revision         0.0.1
#    Date             2020/06/08
#    Author           jiangliheng
#    Email            jiang_liheng@163.com
#    Website          https://jiangliheng.github.io/
#    Description      发送钉钉消息
#    Copyright        Copyright (c) jiangliheng
#    License          GNU General Public License
#
#================================================================
#
#  Version 0.0.1 2020/06/08
#     发送钉钉消息，支持 text，markdown 两种类型消息
#
#================================================================
#%名称(NAME)
#%       ${SCRIPT_NAME} - 发送钉钉消息
#%
#%概要(SYNOPSIS)
#%       sh ${SCRIPT_NAME} [options] <value> ...
#%
#%描述(DESCRIPTION)
#%       发送钉钉消息
#%
#%选项(OPTIONS)
#%     * -a <value>                 钉钉机器人 Webhook 地址的 access_token
#%     * -t <value>                 消息类型：text，markdown
#%     & -T <value>                 title，首屏会话透出的展示内容；消息类型（-t）为：markdown 时
#%     * -c <value>                 消息内容，content或者text
#%     & -m <value>                 被@人的手机号（在 content 里添加@人的手机号），多个参数用逗号隔开；如：138xxxx6666，182xxxx8888；与是否@所有人（-A）互斥，仅能选择一种方式
#%     & -A                         是否@所有人，即 isAtAll 参数设置为 ture；与被@人手机号（-m）互斥，仅能选择一种方式
#%       -v, --version              版本信息
#%       --help                     帮助信息
#%
#%     * 表示必输，& 表示条件必输，其余为可选
#%
#%示例(EXAMPLES)
#%
#%       1. 发送 text 消息类型，并@指定人
#%       sh ${SCRIPT_NAME} -a xxx -t text -c "我就是我, 是不一样的烟火" -m "138xxxx6666,182xxxx8888"
#%
#%       2. 发送 markdown 消息类型，并@所有人
#%       sh ${SCRIPT_NAME} -a xxx -t markdown -T "markdown 测试标题" -c "# 我就是我, 是不一样的烟火" -A
#%
#================================================================
# END_OF_HEADER
#================================================================

# header 总行数
SCRIPT_HEADSIZE=$(head -200 "${0}" |grep -n "^# END_OF_HEADER" | cut -f1 -d:)
# 脚本名称
SCRIPT_NAME="$(basename "${0}")"
# 版本
VERSION="0.0.1"

# usage
function usage() {
  head -"${SCRIPT_HEADSIZE:-99}" "${0}" \
  | grep -e "^#%" \
  | sed -e "s/^#%//g" -e "s/\${SCRIPT_NAME}/${SCRIPT_NAME}/g" -e "s/\${VERSION}/${VERSION}/g"
}

# 发送 ding 消息
function sendDingMessage() {
  curl -s "${1}" -H 'Content-Type: application/json' -d "${2}"
}

# 检查参数输入合法性
function checkParameters() {
  # -a，-t，-c 参数必输校验
  if [ -z "${ACCESS_TOKEN}" ] || [ -z "${MSG_TYPE}" ] || [ -z "${CONTENT}" ]
  then
    printf "Parameter [-a,-t,-c] is required!\n"
    exit 1
  fi

  # -t 为：markdown 时，检验参数 -T 必输
  if [ "X${MSG_TYPE}" = "Xmarkdown" ] && [ -z "${TITLE}" ]
  then
    printf "When [-t] is 'markdown', you must enter the parameter [-T]!\n"
    exit 1
  fi

  # -A 和 -m 互斥，仅能选择一种方式
  if [ "X${IS_AT_ALL}" = "Xtrue" ] && [ -n "${MOBILES}" ]
  then
    printf "Only one of the parameters [-A] and [-m] can be entered!\n"
    exit 1
  fi
}

# markdown 消息内容
function markdownMessage() {
  # 标题
  title=${1}
  # 消息内容
  text=${2}
  # @ 方式
  at=${3}

  # 判断是@所有人，还是指定人
  if [ -z "${at}" ]; then
    atJson=""
  elif [ "X${at}" = "Xtrue" ]; then
    atJson='"at": {
        "isAtAll": true }'
  else
    # 判断是否多个手机号
    result=$(echo "${at}" | grep ",")

    # N 个手机号
    if [ "X${result}" != "X" ]; then
      # 转换为手机号数组
      mobileArray=(${at//,/ })
      # 循环遍历数组，组织 json 格式字符串
      for mobile in "${mobileArray[@]}"
      do
         mobiles="${mobile}",${mobiles}
         # @ 指定人
         atMobiles="@${mobile}",${atMobiles}
      done

    # 1 个手机号
    else
      mobiles="${at}"
      # @ 指定人
      atMobiles="@${at}"
    fi

    # @ json内容
    atJson='"at": {
        "atMobiles": [
            '${mobiles/%,/}'
        ]
    }'

    # 内容信息添加 @指定人
    text="${text}\n${atMobiles/%,/}"
  fi

  message='{
       "msgtype": "markdown",
       "markdown": {
           "title":"'${title}'",
           "text": "'${text}'"},
        '${atJson}'
   }'

   echo "${message}"
}

# text 消息内容
function textMessage() {
  # 消息内容
  text=${1}
  # @ 方式
  at=${2}

  # 判断是@所有人，还是指定人
  if [ -z "${at}" ]; then
    atJson=""
  elif [ "X${at}" = "Xtrue" ]; then
    atJson='"at": {
        "isAtAll": true }'
  else
    # 判断是否多个手机号
    result=$(echo "${at}" | grep ",")

    # N 个手机号
    if [ "X${result}" != "X" ]; then
      # 转换为手机号数组
      mobileArray=(${at//,/ })
      # 循环遍历数组，组织 json 格式字符串
      for mobile in "${mobileArray[@]}"
      do
         mobiles="${mobile}",${mobiles}
         # @ 指定人
         atMobiles="@${mobile}",${atMobiles}
      done

    # 1 个手机号
    else
      mobiles="${at}"
      # @ 指定人
      atMobiles="@${at}"
    fi

    # @ json内容
    atJson='"at": {
        "atMobiles": [
            '${mobiles/%,/}'
        ]
    }'

    # 内容信息添加 @指定人
    text="${text}\n${atMobiles/%,/}"
  fi

  message='{
       "msgtype": "text",
       "text": {
           "content": "'${text}'"},
        '${atJson}'
   }'

   echo "${message}"
}

# 主方法
function main() {

  # 检查参数输入合法性
  checkParameters

  # 判断发送消息类型
  case ${MSG_TYPE} in
    markdown)
      # 判断 @ 方式
      if [ -n "${MOBILES}" ]; then
        DING_MESSAGE=$(markdownMessage "${TITLE}" "${CONTENT}" "${MOBILES}")
      elif [ -n "${IS_AT_ALL}" ]; then
        DING_MESSAGE=$(markdownMessage "${TITLE}" "${CONTENT}" "${IS_AT_ALL}")
      else
        DING_MESSAGE=$(markdownMessage "${TITLE}" "${CONTENT}")
      fi
      ;;
    text)
      if [ -n "${MOBILES}" ]; then
        DING_MESSAGE=$(textMessage "${CONTENT}" "${MOBILES}")
      elif [ -n "${IS_AT_ALL}" ]; then
        DING_MESSAGE=$(textMessage "${CONTENT}" "${IS_AT_ALL}")
      else
        DING_MESSAGE=$(textMessage "${CONTENT}")
      fi
      ;;
    *)
      printf "Unsupported message type, currently only [text, markdown] are supported!"
      exit 1
      ;;
  esac

  sendDingMessage "${DING_URL}" "${DING_MESSAGE}"
}

# 判断参数个数
if [ $# -eq 0 ];
then
  usage
  exit 1
fi

# getopt 命令行参数
if ! ARGS=$(getopt -o vAa:t:T:c:m: --long help,version -n "${SCRIPT_NAME}" -- "$@")
then
  # 无效选项，则退出
  exit 1
fi

# 命令行参数格式化
eval set -- "${ARGS}"

while [ -n "$1" ]
do
  case "$1" in
    -a)
      # Webhook access_token
      ACCESS_TOKEN=$2
      # 钉钉机器人 url 地址
      DING_URL="https://oapi.dingtalk.com/robot/send?access_token=${ACCESS_TOKEN}"
      shift 2
      ;;

    -t)
      MSG_TYPE=$2
      shift 2
      ;;

    -T)
      TITLE=$2
      shift 2
      ;;

    -c)
      CONTENT=$2
      shift 2
      ;;

    -m)
      MOBILES=$2
      shift 2
      ;;

    -A)
      IS_AT_ALL=true
      shift 2
      ;;

    -v|--version)
      printf "%s version %s\n" "${SCRIPT_NAME}" "${VERSION}"
      exit 1
      ;;

    --help)
      usage
      exit 1
      ;;

    --)
      shift
      break
      ;;

    *)
      printf "%s is not an option!" "$1"
      exit 1
      ;;

  esac
done

main
```

> 微信公众号：daodaotest
