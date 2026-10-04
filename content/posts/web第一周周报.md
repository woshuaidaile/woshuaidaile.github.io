+++
date = '2026-09-29T17:17:00+08:00'
draft = false
title = 'web第一周周报'
tags = ['web']
categories = ['http与前端三件套']
+++

<!-- TOC -->

- [谷歌浏览器](#谷歌浏览器)
- [AI Agent](#ai-agent)
- [markdown](#markdown)
- [Burp Suite](#burp-suite)
- [HTTP协议基础](#http协议基础)
- [Hackbar](#hackbar)
- [登录页面](#登录页面)

<!-- /TOC -->

## 谷歌浏览器

![谷歌浏览器](/images/谷歌浏览器.png)

## AI Agent

我用的DeepSeek的模型+DeepSeek的Agent外壳，操作上我先装了个包装脚本，把 dsh 这个短命令绑定到 npx @deepseek-ai/dsh。以后我只要敲dsh web，它自己会去网上下代码、然后在本地打开个网页，最终浏览器打开那个地址就是AI界面

## markdown

我博客就是用它写的

## Burp Suite

我自己装了一个社区版的bp，然后在浏览器上装了bp的代理并且给bp装了证书，可以对https的网站进行简单的抓包，只是不太敢操作，怕犯法，所以就在虚拟机的靶场里抓了一下。接着尝试了借助Proxifier对微信小程序进行抓包（大体流程就是配置代理服务器、设置代理服务器、修改代理规则、添加并选择适用于微信小程序的代理规则），这里抓微信小程序是因为我在网上了解到对大部分的基础漏洞发现都是在小程序被发现的。

![APP抓包示意图](/images/小程序抓包示意图.png)

然后我理解到的就是BP与微信小程序之间需要一个桥梁，而这个桥梁就是Proxifier；最后尝试对APP进行抓包，我通过雷电模拟器用安卓9系统成功在bp上抓到了包

![APP抓包示意图](/images/APP抓包示意图.png)

## HTTP协议基础

1、GET与POST：GET用于向服务器获取数据，有长度限制，安全系数较低；POST用于向服务器提交数据，长度无限制，安全系数较高

2、请求头：（1）UA：大部分APP只有把模拟器改为原生手机的UA才能伪装成正常用户访问
          （2）Referer：用于向浏览器介绍请求标识的来源，类似于介绍信

3、鉴权相关字段：
|项目|Session|JWT
|----|----|----|
| 数据存放位置 | 服务器 | 客户端（Token字符串内）
| 身份凭证 |SessionID（Cookie）| JWT字符串（一般在Header）
| 服务器存储压力 |大，需要保存会话 | 小，不存会话
| 能否立刻注销 | 可以，服务端删除session记录 | 很难，只能等待过期
| 篡改 | 改SessionID没用，服务端校验 | 改payload内容，签名校验失败就拒绝
||||

注：Cookie本身不会鉴权，它只是搬运数据，作为Session与JWT的载体；而Cookie在作为请求头时出现在请求头的中间区域，专门用来携带身份凭证的其中一个字段。总结一下的话就是Cookie 把自己的“编号”写在“请求头（备注栏）”上，交由服务器进行“鉴权（查账本）”，从而让服务器认出“你是谁”，并决定“让不让你进”。（总结来自豆女士）

4、前端的理解：前端是用户可以直接看到和交互的网站、web应用、APP、桌面程序等，与之对应的是后端，后端负责保存和管理数据。HTML（超文本标记语言）用于向网页添加内容，比如按钮图片和文字等；CSS（叠层样式表）用来美化网页，它通过调整颜色大小间距让页面更好看；Javascript让网页变得可交互。用比喻的话就是前端相当于餐厅，后端相当于服务员。HTML相当于餐厅的位置和菜单，CSS相当于餐厅的装修设计，Javascript相当于餐厅的服务员。同时我了解到的还有Javascript框架， Bundler打包器-Webpack，Transpiler转译器-TypeScript， CSS预处理器-Sass， CSS框架-Bootstrap以及前端通过XML HTTP请求发送消息到后端URL，但是后面这些没有深入学习，只是先了解了一下定义

## Hackbar

通过 https://www.bilibili.com/video/BV1VEa1zSEXH/?share_source=copy_web&vd_source=ad98be3eb81340f9b0b75f484aa38277 进行了简单学习，额这里就没有详细说明，但是个人认为基本应该有了一定的了解

## 登录页面

我用deepseek v4 flash写了一个登录页面

![AI做的登录界面](/images/AI做的登录界面.png)

同时我让他写了一个教学笔记（我一并上传了），但是我觉得有点难理解，目前正在努力学习