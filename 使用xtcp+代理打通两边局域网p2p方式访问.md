![test](images/使用xtcp+代理打通两边局域网p2p方式访问/test-1692692391463-2.png)

## 最终效果

==实现C内网所有设备借助c1内网代理访问B内网所有服务器==

## 配置公网服务端A

### frps

配置`frps.ini`

```
[common]
# 绑定frp穿透使用的端口
bind_port = 7000
# 使用token认证
authentication_method = token
token = xxxx
```

`./frps -c frps.ini`启动

### 配置service自启(可选)

`/etc/systemd/system/frps.service`

```
[Unit]
# 服务名称，可自定义
Description = frp server
After = network.target syslog.target
Wants = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart = /root/frp/frps -c /root/frp/frps.ini

[Install]
WantedBy = multi-user.target
```

`systemctl start frps.service`启动服务

`systemctl enable frps`设置自启

## 配置内网服务端B

### danted(socks5代理)

> `sudo apt update`
>
> `sudo apt install dante-server`
>
> `cp /etc/danted.conf /etc/danted.conf.bk`
>
> `vim /etc/danted.conf`  修改为以下内容
>
> `service danted restart`
>
> `systemctl enable danted`  自动启动

```
logoutput: stderr
user.privileged: root
user.unprivileged: nobody
internal: 0.0.0.0 port=7891
# external修改为对外网卡名称或者ip
external: xx.xx.xx.xx
socksmethod: none
clientmethod: none
client pass {
	from: 0.0.0.0/0 to: 0.0.0.0/0
}
socks pass {
	from: 0.0.0.0/0 to: 0.0.0.0/0
}
```

==修改external==

### frpc

配置`frpc.ini`

```
[common]
# 公网ip
server_addr = xx.xx.xx.xx
server_port = 7000
token = xxxx

[test_p2p]
type = xtcp
sk = yyyy
local_ip = 127.0.0.1
local_port = 7891
```

`./frpc -c frpc.ini`启动

### 配置service自启(可选)

`/etc/systemd/system/frpc.service`

```
[Unit]
# 服务名称，可自定义
Description = frp client
After = network.target syslog.target
Wants = network.target
# After = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart = /home/user/software/frp/frpc -c /home/user/software/frp/frpc.ini
Restart= always
RestartSec = 60s

[Install]
WantedBy = multi-user.target
```

`systemctl start frpc.service`启动服务

`systemctl enable frpc`设置自启

## 配置内网主机端C

### frpc

`frpc.ini`

```
[common]
server_addr = xx.xx.xx.xx
server_port = 7000
token = xxxx

[test_p2p_visitor]
type = xtcp
# xtcp 的访问者
role = visitor
# 要访问的 xtcp 代理的名字
server_name = test_p2p
sk = yyyy
# 将远程端口映射为本地对应的端口
bind_addr = 0.0.0.0
bind_port = 9998
# 当需要自动保持隧道打开时，设置为 true
keep_tunnel_open = true
# 每小时尝试打开隧道的次数
max_retries_an_hour = 8
# 重试打开隧道的最小间隔时间，单位: 秒	
min_retry_interval = 90	
```

`./frpc -c frpc.ini`启动

### 配置service自启(可选)

`/etc/systemd/system/frpc.service`

```
[Unit]
# 服务名称，可自定义
Description = frp client
After = network.target syslog.target
Wants = network.target
# After = network.target

[Service]
Type = simple
# 启动frps的命令，需修改为您的frps的安装路径
ExecStart = /home/user/software/frp/frpc -c /home/user/software/frp/frpc.ini
Restart= always
RestartSec = 60s

[Install]
WantedBy = multi-user.target
```

`systemctl start frpc.service`启动服务

`systemctl enable frpc`设置自启

## 内网C所有主机访问内网B使用示例

> 借助c1主机访问所有内网B中设备

### ssh

> `vim ~/.ssh/config`  输入以下内容

```
Host Bx
  HostName 想要访问的内网Bx主机的内网ip
  user ssh连接用户名
  Port 22
  ProxyCommand=nc -x 内网C1主机ip:9998  %h %p
```

`ssh Bx`  就能ssh连接了

### ubuntu文件管理器(nautilus)

> 同ssh
>
> 然后在`Other Locations -> Connect to Server`中输入`ssh://Bx`访问

### remmina

> 使用Bx主机内网ip创建一个会话
>
> 然后`vim ~/.local/share/remmina/对应会话名称.remmina`最下方添加

```
proxy_hostname=内网C1主机ip
proxy_type=socks5
proxy_port=9998
```

### 其他软件

> 查找软件使用socks5代理方式，或者直接设置系统代理来使用
