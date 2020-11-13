---
layout: post
title: "Mac 下使用 Python+Selenium 实现西瓜视频自动上传及草稿发布"
date: "2020-11-12 20:30"
category: Selenium
tags: macOS Python Selenium
author: jiangliheng
---
* content
{:toc}



# 背景

研究下 Python+Selenium 自动化测试框架，简单实现 Mac 下自动化批量上传视频西瓜视频并发布，分享给需要的同学（未做过多的异常处理）。

# 脚本实现

1. 首先通过手工手机号登录，保存西瓜视频网站的 cookie 文件
2. 之后加载 cookie 内容，使用脚本批量上传视频，保存到草稿（也可自动发布，为了二次编辑，如修改封面）
3. 最后通过遍历视频草稿列表，来进行草稿视频发布

PS: 同一天上传或发布视频太多时，会被西瓜视频限流。

## 安装依赖

```bash
# 安装依赖
$ pip install selenium PyUserInput pyperclip

# 安装 chromedriver
$ brew install chromedriver
```

## 脚本内容

```python
#!/usr/bin/python
# -*- coding: utf-8 -*-
import time
import json
import os
import shutil
import sys

from selenium import webdriver
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver import ActionChains
from pykeyboard import PyKeyboard
from pymouse import PyMouse
import pyperclip


class XiGua:
    """
    Mac 西瓜视频自动上传视频及发布草稿
    """

    def __init__(self):
        """
        初始化，打开浏览器
        """
        self.driver = webdriver.Chrome()

    def save_cookies(self, cookies_file_name):
        """
        保存 cookies

        cookies_file_name: cookies 文件名称
        """
        # 预留 20 秒，来进行手工登录
        time.sleep(20)
        # 登录成功后，保存 cookies 文件
        with open(cookies_file_name, 'w') as cookies_file:
            cookies_file.write(json.dumps(self.driver.get_cookies()))

    def load_cookies(self, cookies_file_name):
        """
        加载 cookie

        cookies_file_name: cookies 文件名称
        """
        # 加载 cookies 文件
        with open(cookies_file_name, 'r') as cookies_file:
            cookies_list = json.load(cookies_file)

            for cookie in cookies_list:

                if 'expiry' in cookie:
                    del cookie['expiry']
                self.driver.add_cookie(cookie)

        # 加载 cookie 后，刷新页面生效
        self.driver.refresh()

    def is_exist_element_by_xpath(self, xpath):
        """
        判断元素是否存在
        """
        flag = True
        try:
            self.driver.find_element_by_xpath(xpath)
            return flag
        except Exception as e:
            flag = False
            print("xpath: [%s] 的元素不存在，错误：%s" % xpath, e)
            return flag

    def upload_video(self, video_file_path):
        """
        上传视频

        video_file_path: 上传视频路径
        """
        # 打开上传视频页面
        self.driver.get("https://studio.ixigua.com/upload?from=post_article")
        # 点击上传
        self.driver.find_element_by_class_name("byte-upload-trigger-drag").click()
        time.sleep(5)

        # 选择视频文件
        k = PyKeyboard()
        m = PyMouse()
        # 打开
        k.press_keys(['Command', 'Shift', 'G'])
        x_dim, y_dim = m.screen_size()
        k.press_keys(['Shift'])
        m.click(x_dim // 2, y_dim // 2, 1)
        # 复制视频文件路径
        pyperclip.copy(video_file_path)
        # 粘贴
        k.press_keys(['Command', 'V'])
        time.sleep(2)
        k.press_key('Return')
        time.sleep(2)
        k.press_key('Return')
        time.sleep(2)

        # 设置转载选项
        self.driver.find_element_by_xpath(
            '//*[@id="js-video-list-content"]/div/div[2]/div[4]/div[2]/div/div/label[2]/span/span').click()
        time.sleep(1)
        # 同步到抖音
        # self.driver.find_element_by_class_name("byte-checkbox-mask").click()

        # 循环判断视频上传成功，不成功等待10秒后，再次判断，直到成功
        while '上传成功' not in self.driver.find_element_by_xpath(
                '//*[@id="js-video-list-content"]/div/div[1]/div[1]/div[2]/div[2]').text:
            print("循环等待视频上传成功，等待10秒")
            time.sleep(10)

        # 设置视频封面
        self.driver.find_element_by_class_name("m-xigua-upload").click()
        print('点击-上传封面')
        time.sleep(5)
        try:
            reload = self.driver.find_element_by_xpath('/html/body/div[3]/div/div[2]/div/div[1]/div/div/div/div[2]')
            # 视频封面解析失败处理，循环刷新
            if reload != '':
                print('视频封面解析失败处理，开始循环刷新')
                while XiGua.is_exist_element_by_xpath(self,
                                                      '/html/body/div[3]/div/div[2]/div/div[1]/div/div/div/div[2]'):
                    # 点击循环
                    self.driver.find_element_by_xpath(
                        '/html/body/div[3]/div/div[2]/div/div[1]/div/div/div/div[2]').click()
                    print('刷新失败后，等待5秒，再次刷新')
                    time.sleep(5)
                # 选择第一个图片
                img = self.driver.find_element_by_xpath('/html/body/div[3]/div/div[2]/div/div[1]/div/div/div[1]/img')
                img.click()
        except Exception as e:
            print('封面解析正常，无需刷新')
            pass

        # 下一步
        cover_next_element = WebDriverWait(self.driver, 30).until(
            lambda x: x.find_element_by_xpath(
                '/html/body/div[3]/div/div[2]/div/div[2]/div')
        )
        cover_next_element.click()
        print('点击-封面下一步')

        try:
            # 完成裁剪
            cover_crop_element = WebDriverWait(self.driver, 30).until(
                lambda x: x.find_element_by_xpath(
                    '//*[@id="tc-ie-base-content"]/div[2]/div[2]/div[2]/div/div[2]/div/div/div[2]')
            )
            if cover_crop_element != '':
                cover_crop_element.click()
                print('点击-封面完成裁剪')
            else:
                print('封面无需裁剪')
        except Exception as e:
            print('裁剪封面出现异常：%s' % e)
            pass

        time.sleep(5)
        # 确定
        self.driver.find_element_by_xpath('//*[@id="tc-ie-base-content"]/div[2]/div[2]/div[3]/div[3]/button[2]').click()
        print('点击-封面确定')
        time.sleep(1)
        # 再次确定
        self.driver.find_element_by_xpath('/html/body/div[4]/div/div[2]/div/div[2]/button[2]').click()
        print('点击-封面再次确定')
        time.sleep(5)

        # 存草稿
        draft_element = WebDriverWait(self.driver, 30).until(
            lambda x: x.find_element_by_xpath('//*[@id="js-submit-draft-0"]/button')
        )
        action = ActionChains(self.driver)
        print('点击-保存草稿')
        # 移动滚动条到底部
        js = "window.scrollTo(0,document.body.scrollHeight)"
        self.driver.execute_script(js)
        # 移动到 存草稿 按钮点击
        action.move_to_element(draft_element).click().perform()

    def close(self):
        """
        关闭浏览器
        """
        self.driver.close()

    def batch_upload(self, videos_dir_path):
        """
        批量上传视频

        videos_dir_path: 上传视频存储路径
        """
        files = os.listdir(videos_dir_path)
        # 降序排序上传，草稿发布时，视频序号则为顺序
        files.sort(reverse=True)
        # 批量上传视频
        for file in files:
            if os.path.splitext(file)[1] == '.mp4':
                full_file_path = os.path.join(videos_dir_path, os.path.splitext(file)[0])
                print("==开始上传视频：%s" % full_file_path)
                self.upload_video(full_file_path)

                src = os.path.join(videos_dir_path, file)
                dst = os.path.join(videos_dir_path, 'bak', file)
                # 发布完成后，移到到备份目录
                shutil.move(src, dst)

    def videos_release(self):
        """
        草稿视频发布
        """
        self.driver.get("https://studio.ixigua.com/content")
        time.sleep(2)
        # 点击草稿导航
        draft_navigation_element = WebDriverWait(self.driver, 30).until(
            lambda x: x.find_element_by_xpath('//*[@id="app"]/div/section/div/div[1]/ul/li[3]')
        )
        draft_navigation_element.click()
        print('点击-草稿导航')
        time.sleep(2)
        # 草稿列表
        draft_elements = self.driver.find_elements_by_class_name('content-card__title ')
        # 草稿列表为空，则退出
        if len(draft_elements) == 0:
            print("草稿列表为空")
            XiGua.close(self)
            sys.exit()

        # 循环发布草稿，每次都发布第一个
        for i in range(1, 99999):
            # 草稿列表为空，退出
            if draft_elements == '':
                print('草稿发布完成，总共：%s' % str(i))
                XiGua.close(self)
                sys.exit()
            print('当前发布数量 %s， 发布视频: %s' % (str(i), draft_elements[0].text))
            # 发布草稿第一个视频
            draft_elements[0].click()
            time.sleep(3)

            # 立即发布
            element2 = WebDriverWait(self.driver, 30).until(
                lambda x: x.find_element_by_xpath('//button[contains(text(), "发布")]')
            )
            element2.click()
            print('点击-视频发布')

            # 判断是否发布失败，如标题超长
            try:
                # 错误处理
                if XiGua.is_exist_element_by_xpath(self, '/html/body/div[3]/div/div/div/span'):
                    print('发布出现错误，退出，请检查错误，如标题超长等')
                    sys.exit()
            except Exception as e:
                print('草稿发布异常：%s' % e)
                pass

            # 处理封面分辨率低提示
            try:
                # 封面分辨率低
                cover_cancel_element = self.driver.find_element_by_xpath('//div[contains(text(), "取消")]')
                print('封面分辨率低处理,直接取消')
                # 错误处理
                if cover_cancel_element != '':
                    print('取消封面分辨率低')
                    cover_cancel_element.click()
                    # 立即发布
                    cover_publish_element = WebDriverWait(self.driver, 30).until(
                        lambda x: x.find_element_by_xpath('//button[contains(text(), "发布")]')
                    )
                    cover_publish_element.click()
            except Exception as e:
                print('封面分辨率低出现异常：%s' % e)
                pass

            # 点击草稿
            draft_publish_element = WebDriverWait(self.driver, 30).until(
                lambda x: x.find_element_by_xpath('//*[@id="app"]/div/section/div/div[1]/ul/li[3]')
            )
            draft_publish_element.click()
            time.sleep(2)
            print('重新获取草稿列表')
            draft_elements = self.driver.find_elements_by_class_name('content-card__title ')
            print(draft_elements)

    def xigua_videos_release(self, base_url, cookies_file_path):
        """
        西瓜视频发布草稿

        base_url: 西瓜视频网站
        cookies_file_path: 西瓜视频 cookies 文件路径
        """
        self.driver.get(base_url)
        # 加载 cookies
        XiGua.load_cookies(self, cookies_file_path)
        # 草稿发布视频
        XiGua.videos_release(self)
        # 关闭浏览器
        XiGua.close(self)

    def xigua_batch_upload(self, base_url, cookies_file_path, videos_dir_path):
        """
        西瓜视频批量发布视频

        base_url: 西瓜视频网站
        cookies_file_path: 西瓜视频 cookies 文件路径
        videos_dir_path: 上传视频存储路径
        """
        self.driver.get(base_url)
        XiGua.load_cookies(self, cookies_file_path)
        XiGua.batch_upload(self, videos_dir_path)
        XiGua.close(self)

    def xigua_save_cookies(self, base_url, cookies_file_path):
        """
        保存网站 cookie

        base_url: 网站地址
        cookies_file_path: 网站 cookies 文件路径
        """
        self.driver.get(base_url)
        # 保存 cookies
        XiGua.save_cookies(self, cookies_file_path)
        XiGua.close(self)


if __name__ == '__main__':
    xi_gua = XiGua()
    # 西瓜视频
    base_url = 'https://www.ixigua.com/'
    xigua_cookies = '/tmp/xigua_update_video/xigua_cookies.txt'
    videos_dir_path = '/tmp/rm'
    ## 1. 保存 cookie
    # xi_gua.xigua_save_cookies(base_url, 'xigua_cookies.txt')

    ## 2. 批量上传
    xi_gua.xigua_batch_upload(base_url, xigua_cookies, videos_dir_path)

    ## 3. 批量发布草稿
    # xi_gua.xigua_videos_release(base_url, xigua_cookies)
```

> 微信公众号：daodaotest
