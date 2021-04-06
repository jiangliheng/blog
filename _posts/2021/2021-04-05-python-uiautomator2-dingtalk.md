---
layout: post
title: "python + uiautomator2 实现钉钉工单自动批量提交"
date: "2021-04-05 18:30"
category: APP自动化
tags: Python uiautomator2 钉钉
author: jiangliheng
---
* content
{:toc}



# 背景

每个月初，团队负责人需要提交整个团队的上个月绩效评价以及本月的绩效设定，在钉钉上选择员工和Excel 附件提交员工个人审批。

随着团队人员的增加，人工提交耗时耗力，我偶尔还提错，故写个简单的 APP 自动化脚本实现。

懒使人进步~

# 人工提交流程

员工绩效设定与员工绩效评价流程一致，仅考核周期和附件不同。

1. 打开钉钉，依次选择```工作台```-```OA 审批```-```员工绩效设定（评价）```
2. 选择员工姓名、考核周期（月份）、员工绩效设定（评价）表、审核人（与员工姓名一致），提交即可
3. 员工查看，签字确认

**工单截图**
![dingtalk-workflow](/assets/images/app-test/dingtalk-workflow.jpg)


# 自动化实现

人工提交工单是在电脑上操作的，所以附件都从电脑本地选取的。

为了 APP 自动化脚本实现，Excel 附件改为从钉钉私人云盘获取。

存放路径设计为：绩效文件/202104/202104-张三-员工绩效设定.xlsx

具体实现流程见脚本及注释。

## 脚本

```python
# !/usr/bin/env python
# -*- coding:utf-8 -*-

__author__ = 'jiangliheng'

from time import sleep
import uiautomator2 as u2
import logging
import datetime as datetime
import dateutil.relativedelta as relativedelta

# logging 配置
logging.basicConfig(level=logging.INFO, format='%(asctime)s - %(filename)s:%(lineno)d - %(levelname)s: %(message)s')
logger = logging.getLogger(__name__)

# 连接设备
d = u2.connect()


def convert_month(month):
    '''
    MM数字月份转换为中文月份

    :param month: 数字月份 MM
    :return: str 中文月份
    '''
    months = ["一月", "二月", "三月", "四月", "五月", "六月", "七月", "八月", "九月", "十月", "十一月", "十二月"]
    return str(months[int(month)-1])


def submit_order(order_type, date, employee_name):
    '''
    提交员工绩效设定（评价）工单

    前提：
    1. 打开钉钉，进入主页面即可；
    2. 私人云盘按照 "绩效文件/YYYYMM" 目录存放员工绩效设定（评价）文件。

    :param order_type: 工单类型 1-员工绩效设定；2-员工绩效评价
    :param date: 提交日期 eg：202104
    :param employee_name: 员工名称
    :return:
    '''
    # 截取数字月份转换为汉字月份
    month = convert_month(date[4:6])
    if order_type == 1:
        type_message = '员工绩效设定'
    else:
        type_message = '员工绩效评价'

    # 打开钉钉主页面，点击 工作台
    logger.info('点击工作台')
    # 解决默认在工作台页面时，先点击第一个下标按钮
    d(resourceId="com.alibaba.android.rimet:id/home_bottom_tab_icon").click()
    d.xpath('//*[@resource-id="com.alibaba.android.rimet:id/home_app_recycler_view"]/android.widget.RelativeLayout[3]').click()

    logger.info('点击 OA 审批')
    d(text='OA审批').click()

    logger.info('点击' + type_message)
    # 当前页面没有时，上滑后选择
    if not d(text=type_message).exists:
        d.swipe(200, 1000, 200, 100)
    d(text=type_message).click()

    logging.info('选择员工姓名')
    select_name = d(text='员工姓名').down(className='android.view.View').get_text()
    # 判断选中的员工是不是需要提交的员工
    if select_name != employee_name:
        # 非空，选中的不是要提交的员工
        if select_name != '请选择':
            d(text='员工姓名').down(className='android.view.View').click()
        # 默认没有任何选中，请选择
        elif select_name == '请选择':
            d(text='请选择').click()
        # 输入员工姓名搜索
        d(resourceId="com.alibaba.android.rimet:id/view_search").click()
        d(resourceId="android:id/search_src_text").click()
        sleep(1)
        d(resourceId="android:id/search_src_text").set_text(employee_name)
        # 选中
        d(resourceId="com.alibaba.android.rimet:id/tv_avatar").click()
        sleep(1)

    logging.info('选择考核周期')
    month_text = d(text='考核周期').down(className='android.view.View').get_text()
    logging.info(month_text)
    # 默认空，直接选择考核周期
    if month_text == '请选择考核周期':
        d(text='请选择考核周期').click()
        sleep(1)
        # 大于7月时，需要下滑后选择
        if int(date[5:6]) > 7:
            d.swipe(600, 2000, 600, 1000)
        d(text=month).click()

    logging.info('选择审批人')
    # 员工绩效设定时，移除默认选中的人，重新选中
    if order_type == 1:
        d(text='移除', className='android.widget.Button').click()
        d.xpath('//*[@resource-id="MF_APP"]/android.view.View[2]/android.view.View[2]/android.view.View[7]/android.view.View[1]/android.widget.Button[1]/android.widget.Image[1]').click()
    else:
        # 删除存在的审核人
        d(text='移除', className='android.widget.Button').click()
        # 滑动后选择审核人
        d.swipe(600, 2000, 600, 1500)
        d.xpath('//*[@resource-id="MF_APP"]/android.view.View[2]/android.view.View[2]/android.view.View[9]/android.view.View[1]/android.widget.Button[1]/android.widget.Image[1]').click()

    sleep(1)
    # 搜索员工姓名，并选择
    d(resourceId="com.alibaba.android.rimet:id/view_search").click()
    d(resourceId="android:id/search_src_text").click()
    d(resourceId="android:id/search_src_text").set_text(employee_name)
    sleep(2)
    d(resourceId="com.alibaba.android.rimet:id/checkbox").click()
    d(resourceId="com.alibaba.android.rimet:id/btn_ok").click()
    sleep(1)

    logging.info('选择员工绩效文件')
    d(text='添加', className='android.widget.Button').click()
    # 选择云盘方式
    d(text='云盘').click()
    sleep(2)
    # 私人盘
    d(text='私人盘').click()
    # 绩效文件
    d(text='绩效文件').click()
    # 选择月份
    d(text=date).click()
    sleep(1)
    # 选择员工对应文件
    d(textContains=employee_name).click()
    # 点击确定
    d(text='确定', className='android.view.View').click()
    # 提交工单
    logging.info('提交' + name + str(date) + "的" + type_message)
    d(text='提交', className='android.widget.Button').click()
    sleep(5)

    # 退出标识
    flag = True
    # 一直退出，直到无退出按钮
    while flag:
        try:
            d(resourceId='com.alibaba.android.rimet:id/img_back').click()
        except Exception as e:
            flag = False
    # 关闭当前流程，回到钉钉主页面
    d(resourceId='com.alibaba.android.rimet:id/close_icon').click()


if __name__ == '__main__':
    # 1 员工绩效设定 2 员工绩效评价
    order_type = 1

    # 1 员工绩效设定，月份为当月
    if order_type == 1:
        date = datetime.datetime.now().strftime("%Y%m")
    else:
        # 2 员工绩效评价，月份为上个月
        date = (datetime.datetime.now() + relativedelta.relativedelta(months=-1)).strftime("%Y%m")
    # 需要提交所有员工
    employee_names = ["张三", "李四"]
    # 记录总数
    count = 1
    for name in employee_names:
        logging.info("======== " + str(count) + " : " + date + " : " + name)
        # 提交流程
        submit_order(order_type, date, name)
        # 计数
        count = count + 1

    logging.info('上传完成:' + str(count - 1))
```

> 微信公众号：daodaotest
