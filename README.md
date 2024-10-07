# 在ipv6小鸡上手搓realm，实现端口转发

演示机为Ubuntu 22.0，演示环境为ipv6全通，ipv4反向墙

## 安装realm

wget -O realm.tar.gz https://github.com/zhboner/realm/releases/download/v2.4.5/realm-x86_64-unknown-linux-gnu.tar.gz

tar -xvf realm.tar.gz

chmod +x realm

由于github没有ipv4，可以在 https://github.akams.cn/ 寻找代理，举例
https://gh.llkk.cc/https://github.com/zhboner/realm/releases/download/v2.4.5/realm-x86_64-unknown-linux-gnu.tar.gz
https://github.moeyy.xyz/https://github.com/zhboner/realm/releases/download/v2.4.5/realm-x86_64-unknown-linux-gnu.tar.gz

## realm配置文件

在此使用nano进行编辑

nano基本用法：在输入完成之后，ctrl+o保存文件，然后ctrl+x返回，下文同

nano config.toml

在 config.toml 文件中粘贴内容

[log]
level = "warn"
output = "/root/realm.log"
 
[[endpoints]]
listen = "[::]:11111"
remote = "[2606:4700:9a95::5de7:d53]:443"
 
[[endpoints]]
listen = "[::]:22222"
remote = "[2a06:98c1:3121::f36:2c1e]:443"

[[endpoints]]
listen = "[::]:33333"
remote = "[2606:4700:e2::132:41be]:443"

[[endpoints]]
listen = "0.0.0.0:44444"
remote = "162.159.140.120:443"


listen代表监听本机，remote代表落地机
注：listen中的0.0.0.0或::不要改，在IPv6中，表示所有地址的特定地址是::  未指定地址::在IPv6中用于表示没有特定地址的情况，类似于IPv4中的0.0.0.0

11111,22222，33333端口可以随便改，本文使用cloudflare的ipv6，v4地址作为演示，请改成你自己的地址，被别人扫到偷流量后果自负！！！

## realm自启动


因为手动每次运行很麻烦，我们需要创建 Linux 的服务项来实现自启动转发通道。
举例创建 service 服务项，首先使用 nano 编辑服务项内容：

nano /etc/systemd/system/realm.service

如果你的 realm 主程序和配置文件都和我一样在 /root 目录里的话，直接使用下面的内容即可

[Unit]
Description=realm
After=network-online.target
Wants=network-online.target systemd-networkd-wait-online.service
 
[Service]
Type=simple
User=root
Restart=on-failure
RestartSec=5s
DynamicUser=true
ExecStart=/root/realm -c /root/config.toml
 
[Install]
WantedBy=multi-user.target


然后保存文件就可以了


## 收尾

systemctl daemon-reload

systemctl enable realm && systemctl start realm


查看realm状态
systemctl status realm

## 参考文章

https://ooly.cc/archives/linux/396
https://zhucaidan.xyz/2022/09/570/


