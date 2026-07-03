---
title: "Charlse抓包教程"
date: 2026-07-02T21:15:09+08:00
lastmod:
draft: false
tags: ["Charlse", "抓包"]
categories: ["技术教程"]
description: "Charlse抓包教程"
---

## 环境准备

- win10 笔记本（有线网连接）
- 安卓手机
- Charlse

## 步骤

1. 打开电脑移动热点，cmd 输入 `ipconfig` 查看 `无线局域网适配器 本地连接* 2` 的 `IP` 地址。一般是 `192.168.137.1`。记住，后面要用到。
2. 打开 **Charlse**， 选择 proxy > SSL Proxing Settings。勾选Enable SSL Proxing，include 下 add location, 域名，端口都为*。添加后勾选，依次点击OK。
3. 再选择 proxy > Proxy Settings。http proxy port 设置为8888，Surpport HTTP/2，Enable 等都勾上。点击OK。
4. 再点击 proxy > Access Control Settings。IP Range 设置为：0.0.0.0/0, add 后点击OK。
5. 用手机连接电脑热点。进入 `WLAN` 详情>高级选项>代理。主机名为第一步的 `IP` 地址，端口为8888，保存。
6. 电脑防火墙，允许应用通过防火墙进行通信，勾选 Charlse。
7. 打开网络适配器>更改适配器选项，找到以太网，属性>共享。勾选允许其他用户连接。家庭网络连接选项选择第一步的网络，即: 无线局域网适配器 本地连接* 2。
8. 打开手机浏览器，访问 `chls.pro/ssl`。进去后会弹出证书下载链接。下载后如果是 `.pem`则改后缀为 `.crt`。
9. 安装证书。回到手机设置 -> 搜索“CA证书” -> 从存储设备安装 -> 选择这个刚刚改名后的 charles.crt 文件安装。
10. 回到电脑，进入charlse, 勾选小红点 **Start Recording** 开始录制。
11. 手机浏览器访问任意网站或APP，即可捕获所有 http 请求。


## FAQ

Q. **为什么老是跳出该网站的证书不被信任？**

A. 用对用户自主安装的 CA 证书放得比较宽的浏览器，只要系统装了证书就不会再弹窗。

Q. **手机上不了网，电脑也上不了。**

A. 确保没有开其他代理软件，或者严格按上述步骤在配置一次。

## 其他常见问题排查

| 现象                        | 原因                               | 解决办法                                         |
| ------------------------- | -------------------------------- | -------------------------------------------- |
| 手机打不开网页                   | IP 填错                            | 确认 `ipconfig` 中热点网卡 IP，通常是 `192.168.137.1`   |
| Charles 没有任何请求            | 手机代理未生效                          | 检查 WiFi 代理是否为手动，Host/Port 是否正确               |
| Charles 未弹出授权             | Access Control 或防火墙拦截            | 放宽 Access Control，并允许 Charles 通过 Windows 防火墙 |
| 只能抓 HTTP，HTTPS 全是 CONNECT | 未安装 CA 证书或未开启 SSL Proxying       | 安装 Charles 根证书，并在 Charles 中启用 SSL Proxying   |
| 浏览器能抓，App 抓不到             | App 启用了证书固定（Certificate Pinning） | 需要关闭 App 的证书固定、使用调试版，或借助 Frida、Magisk 等工具绕过  |
