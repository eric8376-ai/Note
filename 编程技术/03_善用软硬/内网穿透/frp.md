

### 流程
7000：是C/S的通信端口
DNS（*.oa）-》Server（114.55.248.111：8001）-》Client（web1.oa：8088,web2.oa:3000）

### 客户端frpc.ini
[common]
server_addr = 114.55.248.111
server_port = 7000
token = 12345678
[web1]
type = http
local_port = 8088  
custom_domains = superset.oa.xmsrv.tech 

[web2]
type = http
local_port = 3000  
custom_domains = metabase.oa.xmsrv.tech 

### 服务端配置frps.ini
[common]
bind_port = 7000
authentication_method = token

token = 12345678
**vhost_http_port = 8001    入口端口**
vhost_https_port =8002

tls_only = false
tls_cert_file =./server.crt
tls_key_file =./server.key

dashboard_addr = 0.0.0.0
dashboard_port = 7500
dashboard_user = admin
dashboard_pwd = admin


