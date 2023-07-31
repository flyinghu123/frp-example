![framework](https://github.com/flyinghu123/frp-example/assets/48802940/e03fe0d8-b251-445b-a7da-91907818af78)

通过frp实现内网client访问另外一个内网服务器

## 管理员

- 1）配置公网服务端frps
- 2）配置内网服务端frpc

### Ubuntu

#### 配置公网服务端frps

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

##### 配置service自启(可选)

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

#### 配置内网服务端frpc

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
local_port = 7890
```

`./frpc -c frpc.ini`启动

##### 配置service自启(可选)

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
RestartSec=60

[Install]
WantedBy = multi-user.target
```

`systemctl start frpc.service`启动服务

`systemctl enable frpc`设置自启

## 使用者

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

配置完成后与当前client相同内网的都能通过当前client的ip:9998访问另外一个内网服务器7890服务

##### 配置service自启(可选)

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
RestartSec=60

[Install]
WantedBy = multi-user.target
```

`systemctl start frpc.service`启动服务

`systemctl enable frpc`设置自启
