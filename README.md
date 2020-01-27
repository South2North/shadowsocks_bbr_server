# CentOS7搭建shadowsocks配置BBR

## 1. 安装pip
在控制台执行以下命令安装pip：
```sh
# 先升级一些库，不然下一步会报错：
# curl: (35) Peer reports incompatible or unsupported protocol version.
yum update -y nss curl libcurl
# 安装pip
curl "https://bootstrap.pypa.io/get-pip.py" -o "get-pip.py"
python get-pip.py
```

## 2. 安装并配置shadowsocks
- 在控制台执行以下命令安装shadowsocks：
```sh
# 升级pip
pip install --upgrade pip
pip install shadowsocks
```
- 安装完成后，需要创建shadowsocks的配置文件`/etc/shadowsocks.json`，使用vi编辑：
```sh
vi /etc/shadowsocks.json
```
**如果需要配置多个端口**，内容如下：
```sh
{
    "server": "0.0.0.0",
    "local_address": "127.0.0.1",
    "local_port": 1080,
    "port_password": {
        "8080": "填写密码",
        "8081": "填写密码"
    },
    "timeout": 600,
    "method": "aes-256-cfb"
}
```
若仅配置单个端口，则使用以下配置：
```sh
{
    "server": "0.0.0.0",  
    "server_port": 8080,  
    "password": "填写密码",  
    "method": "aes-256-cfb"
}
```
- 配置自启动

编辑shadowsocks服务的启动脚本文件：
```sh
vi /etc/systemd/system/shadowsocks.service
```
内容如下：
```sh
[Unit]
Description=Shadowsocks

[Service]
TimeoutStartSec=0
ExecStart=/usr/bin/ssserver -c /etc/shadowsocks.json

[Install]
WantedBy=multi-user.target
```

- 执行以下命令启动 shadowsocks 服务：
```sh
systemctl enable shadowsocks
systemctl start shadowsocks
```
执行：
```sh
systemctl status shadowsocks -l
```
如果服务启动成功，则控制台显示的信息应该类似这样：
```sh
shadowsocks.service - Shadowsocks
   Loaded: loaded (/etc/systemd/system/shadowsocks.service; enabled)
   Active: active (running) since 三 2019-06-05 03:29:25 EDT; 14s ago
 Main PID: 10182 (ssserver)
   CGroup: /system.slice/shadowsocks.service
           └─10182 /usr/bin/python /usr/bin/ssserver -c /etc/shadowsocks.json

6月 05 03:29:25 localhost.localdomain systemd[1]: Started Shadowsocks.
6月 05 03:29:26 localhost.localdomain ssserver[10182]: INFO: loading config from /etc/shadowsocks.json
6月 05 03:29:26 localhost.localdomain ssserver[10182]: 2019-06-05 03:29:26 INFO     loading libcrypto from libcrypto.so.10
6月 05 03:29:26 localhost.localdomain ssserver[10182]: 2019-06-05 03:29:26 INFO     starting server at 0.0.0.0:8080

```

- *如果端口挂了，或者需要重新配置json文件：*
```sh
# 停止ss服务
systemctl stop shadowsocks
# 更改json文件
vi /etc/shadowsocks.json
# 重新启动服务
systemctl start shadowsocks
```
*如果ip挂了的话，就只能删除服务器重建了*

## 3. 配置BBR
TCP BBR是谷歌出品的TCP拥塞控制算法。BBR目的是要尽量跑满带宽，并且尽量不要有排队的情况。BBR可以起到单边加速TCP连接的效果。
执行以下命令（**第1个命令现在执行到NetworkManager会断开，不是很清楚咋回事，不输这句貌似也没什么影响**）
```sh
# yum -y update
wget --no-check-certificate https://github.com/teddysun/across/raw/master/bbr.sh && chmod +x bbr.sh && ./bbr.sh
```
执行完之后确认重启，过一段时间应该就能成功使用了。可以再通过ssh连上server确认：
```sh
sysctl net.ipv4.tcp_available_congestion_control
# 输出结果中有bbr：
# net.ipv4.tcp_available_congestion_control = bbr cubic reno
# 如果没有，再输入一次：
./bbr.sh
# 以下表示设置成功：
# Info: Your kernel version is greater than 4.9, directly setting TCP BBR...
# Info: Setting TCP BBR completed...

```

## 4. Ubuntu安装shadowsocks客户端（可选）
```sh
sudo add-apt-repository ppa:hzwhuang/ss-qt5
sudo apt-get update
sudo apt-get install shadowsocks-qt5
```

## TODO
- 更新一键安装ss并配置bbr的脚本

## 参考
- https://blog.51cto.com/zero01/2064660
- https://www.cnblogs.com/zzugyl/p/10277012.html
