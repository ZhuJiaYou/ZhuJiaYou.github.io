---
title: Linux SS代理全教程
date: 2019-05-19 09:07:04
tags:
  - Linux
  - SS
  - Deepin
  - 终端代理
  - PAC
  - 开机自启
categories: Linux 技术
---
## 一 引言
1. Linux下可完美配置SS，包括PAC模式，终端代理，开机自启等；
2. 不过要想完美使用尚需一番配置；
3. 这里针对Debian系Linux，具体以Deepin为例。
<!--more-->


## 二 安装配置 SS Server
要使用SS，首先要有SS Server，可以购买SS Server账号，或者自行搭建。这里以自行搭建为例。
1. 注册云计算机器租赁提供商DigitalOcean账号，使用如下链接可享用免费额度(需绑定信用卡)：
https://m.do.co/c/7794f02c9f32
2. 在DO上租赁一台国外服务器(Droplet)，最便宜的5美元一月即可。具体可自行搜索解决，很简单。
3. 安装SS Server：
```
sudo apt update
sudo apt install python-pip
pip install git+https://github.com/??/??.git@master  # ??处具体内容自己补充
```
4. 配置SS Server：
```
sudo touch /etc/??.json
sudo echo '{"server":"::", "server_port":8388, "local_address": "127.0.0.1", "local_port":1080, "password":"222265", "timeout":300, "method":"chacha20", "fast_open": false }' > /etc/??.json
```
5. 启动SS Server：
`ssserver -c /etc/??.json -d start`


## 三 SS Client 安装及基础配置
1. 安装SS Client(命令行版本)：
```
sudo apt install python-pip
pip install ??
```
2. 配置SS Client：
`vim ~/cfgs/??.json`  # 可自行选择创建位置
写入如下内容：
```json
{
    "server":Your Server IP,  # SS Server IP，字符串形式
    "server_port":8388,
    "local_address":"127.0.0.1",
    "local_port":1080,
    "password":Your Server Passwd,  # 字符串形式
    "timeout":300,
    "method":"chacha20"
}
```
3. 启动SS Client：
`sslocal -c ~/cfgs/??.json &`
4. 设置Deepin系统代理：
（1）打开系统设置==>网络==>系统代理==>手动，在socks代理处输入本地地址和本地端口号，如图，然后点击确定保存：
![系统代理](/images/blog2/sys.png)
（2）如此可配置Deepin实现系统代理，但是不完美，没有PAC模式等。下面进行讲解。


## 四 浏览器PAC模式代理
这里需要先生成PAC规则文件，后设置浏览器。
1. 安装genpac：
`pip install genpac`
更新：
`pip install --upgrade genpac`
或从github更新开发版本：
`pip install --upgrade https://github.com/JinnLynn/genpac/archive/master.zip`
2. 生成/更新PAC规则文件：
```
cd ~/cfgs/
sudo genpac --proxy="SOCKS5 127.0.0.1:1080" --gfwlist-proxy="SOCKS5 127.0.0.1:1080" -o autoproxy.pac --gfwlist-url="https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt"`
```
3. 设置PAC代理，：
打开系统设置==>网络==>系统代理==>自动，在配置URL处输入如下PAC文件地址，如图，然后点击确定保存：
`file:///home/user/cfgs/autoproxy.pac`
![PACFile](/images/blog2/PACFile.png)
4. 安装SwitchyOmega插件，可到谷歌、360或者Firedox等浏览器插件商店安装，很简单，可自行搜索解决。
5. 配置SwitchyOmega，在插件“选项”中操作。
（1）设置代理地址
可以左边选择新建情景模式==>选择代理服务器(命名自设，如“SS”，其他默认)；之后创建==>代理协议选择为SOCKS5 、地址为 127.0.0.1、端口为自己设置的SS端口(默认为1080)，点击应用选项保存。
![so1](/images/blog2/so1.png)
（2）设置自动切换
①点击左侧自动切换(Auto switch)==>“规则列表匹配请求”后面选择新建的SS==>“默认情景模式”为直接连接==>应用选项保存。
②“规则列表格式”设置为AutoProxy ==>填入[地址](https://raw.githubusercontent.com/gfwlist/gfwlist/master/gfwlist.txt)==>立即更新情景模式。
③点击应用选项保存。
![so2](/images/blog2/so2.png)
（3）待应用之后，可以点击浏览器右上角SwitchyOmega图标，选择自动切换，浏览器可PAC模式代理。


## 五 终端代理
上面虽然装好了??，但它是shell 里执行的命令，发起的网络请求现在只支持http代理。privoxy可以把http请求转发给socks5。
1. 安装privoxy：
`sudo apt install privoxy`
2. 配置privoxy：
`sudo vim /etc/privoxy/config`
先搜索关键字 listen-address 找到`listen-address 127.0.0.1:8118`这一句，保证这一句没有注释，8118就是将来http代理要输入的端口；然后搜索 forward-socks5t, 将 `#forward-socks5t / 127.0.0.1:1080 .`此句前面的注释去掉, 意思是转发流量到本地的1080端口, 而1080端口正是 ss 监听的端口。
3. 应用privoxy配置：
`sudo privoxy --user privoxy /etc/privoxy/config`
4. 转发配置：
`vim ~/.bashrc`
写入：
```
export http_proxy=http://127.0.0.1:8118
export https_proxy=http://127.0.0.1:8118
export ftp_proxy=http://127.0.0.1:8118
```
使.bashrc生效：
`source ~/.bashrc`
5. 在终端中测试是否可代理：
`curl www.google.com`
或者：
`wget www.google.com`


## 六 开机自启
Deepin 没有 /etc/rc.local 文件，但可以自己新建此文件并用其实现** 的开机自启。
1. 安装supervisor管理sslocal启动：
`sudo apt install supervisor`
2. 配置supervisor：
`sudo vim /etc/supervisor/supervisord.conf`
写入：
```
[program:??]
command=sslocal -c /home/zjy/cfgs/??.json
autostart=true
autorestart=true
user=root
log_stderr=true
logfile=/var/log/??.log
```
3. 拷贝一份命令文件sslocal文件到/bin下：
`sudo cp /usr/bin/sslocal /bin  # 这里的 sslocal 文件路径不同的系统，路径可能不同，可以使用 find命令查找`
4. 检查supervisor是否起作用：
保存并关掉之前的终端，再重新打开终端输入：
`sudo service supervisor restart`
并打开浏览器访问检查 sslocal 是否在运行。
当然你也可以使用命令：
`ps -aux | grep sslocal`
5. 创建/etc/rc.local文件：
`sudo vim /etc/rc.local`
写入：
```
/bin/sh -e
# 在 exit 0 之前添加你的开机启动程序

service supervisor start

exit 0
```
5. 重启机器，再次查看是否代理成功：
`ps -aux | grep sslocal`

