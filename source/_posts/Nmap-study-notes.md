---
title: Nmap study notes
date: 2019-10-07 12:28:26
categories:
- StudyNotes
tags:
- web
- network
- security
---

最近公司的项目被客户的白帽黑客找出一堆安全漏洞，为此我们在较劲脑汁的想解决方案的同时，也佩服着那位一个人找出如此多漏洞并给我们开了一大堆issue的黑客。本人从来没有接触过网络安全方面的知识。在黑客开的issue里面经常提到一个叫nmap的东西。于是便查了一下，发现nmap是一个网络嗅探(network sniffing)的工具。可以用来检测服务器开放的端口及对应端口提供的服务，以及可能的弱点。感觉这个工具实在是有点强大，于是便找了些资料，学习一下，也对我了解网络有所帮助。

# Nmap
## Install
[nmap website](https://nmap.org/)
### windows
下载后，双击exe文件安装即可
### 其他
查看官网
## 命令
### 列举远程机器开放的端口
```
nmap scanme.nmap.org

```
nmap 进行探测之前要把域名通过DNS 服务器解析为IP地址，我们可以使用指定的DNS服务器进行解析。使用`--dns-servers`。
```
nmap --dns-servers 8.8.8.8 {hostname}

```
这里使用了google的dns服务器

使用`-pn`来停止探测之前的ICMP请求，已达到不触发防火墙的安全机制。
```
nmap -Pn {hostname}

```
探测特定端口：
```
nmap -p 100-30000 {hostname}

```
端口状态：
- open 开放状态
- closed 关闭状态
- filtered 过滤无法收到返回的probe
- unfiltered 收到返回的probe，但是无法确认

### 识别服务指纹
```
nmap -sV {hostIP}

```
侵略性探测
```
nmap -A -v -T4 {ip}

```
```
nmap -sC -sV -O {ip}

```
### 主机发现
**CIDR**： classless inter-domain routing 无类别域间路由
172.16.1.1/24 表示在 172.168.1.1-172.16.1.255之间所有的主机ip地址
```
nmap -sP {CIDR}

```
```
nmap -sn {CIDR}

```
输出结果xml：
```
nmap -sn CIDR -oX test.xml

```
### 端口检测
```
nmap -p{port1, port2} {host}

```
范围扫描
```
nmap -p1-100 {host}

```
所有端口
```
nmap -p- {host}

```
指定端口的协议
```
nmap -p T:22,U:53 {host}

```
通过协议名来扫描端口
```
nmap -p smtp {host}

```
wild card
```
nmap -p s* {host}

```
### NSE 介绍
nmap script engine 
```
nmap --script {scriptname} 

```
```
nmap --script http-header {host}

```
漏洞检测
```
nmap -sV --script vuln {host}

```
NSE 更新：
```
nmap -script-updatedb

```
### 指定网卡进行探测
输出网卡信息
```
nmap --iflist

```
```
nmap -e interface CIDR

```
```
nmap -e eth0 {ip}

```
## Other
#### ndiff
compare two nmap xml file

```
 ndiff file1 file2

```
#### zenmap
可视化nmap


