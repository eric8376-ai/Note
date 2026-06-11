# 概览
Shadowsocks、V2Ray和Clash是三种主流的网络代理工具，但定位和功能各有侧重。Shadowsocks出现较早，是一个简洁高效的SOCKS5代理，专注于加密传输以绕过网络封锁，其优势在于轻量快速、配置简单，但协议特征明显，在现代深度检测环境下较易被识别。

V2Ray则是一个功能强大的代理平台，其核心优势在于强大的抗封锁能力与高度的可定制性。它独创了VMESS协议，并支持与WebSocket、HTTP/2等常见协议融合，将代理流量伪装成普通网页访问，从而有效对抗流量审查。它内置了灵活的路由功能，可以满足复杂的代理需求。

与前两者不同，Clash本质上并非一个独立的代理协议，而是一个智能的代理客户端。它本身不创造新协议，但可以兼容管理Shadowsocks、V2Ray、Trojan等多种协议的节点。它的核心竞争力在于极其精细的规则分流引擎，允许用户根据域名、地理位置、IP等规则，让不同流量自动选择直连、代理或阻塞，非常适合管理多个服务器节点或订阅机场服务。

总而言之，三者的核心区别在于：Shadowsocks是轻量协议，V2Ray是抗封锁平台，而Clash是智能客户端。在实际应用中，它们常被组合使用，例如用V2Ray或Shadowsocks搭建服务器提供代理能力，再用Clash作为前端客户端进行便捷的分流和管理。
## Shadowsocks

[Shadowsocks完全指南：从入门到精通（2025最新版） - Cursor IDE 博客](https://www.cursor-ide.com/blog/shadowsocks-guide-2025)

https://shadowsocks.org/
## V2Ray
手机端
https://en.v2rayng.org/
## FlClash

[Releases · chen08209/FlClash](https://github.com/chen08209/FlClash/releases)
https://clash.info/flclash/
## clash-verge-rev
https://github.com/clash-verge-rev/clash-verge-rev

|客户端|Windows|macOS|iOS|Android|教程|
|---|---|---|---|---|---|
|Clash for Android||||✔|[使用教程](https://clash.info/clash-for-android/)|
|Clash for Windows|✔|✔|||[使用教程](https://clash.info/clash-for-windows/)|
|Clash Meta For Android||||✔|[使用教程](https://clash.info/clash-meta-for-android/)|
|Clash Mi|✔|✔|✔|✔|[使用教程](https://clash.info/clash-mi/)|
|Clash Verge|✔|✔|||[使用教程](https://clash.info/clash-verge/)|
|ClashX||✔|||[使用教程](https://clash.info/clashx/)|
|ClashX Meta||✔|||[使用教程](https://clash.info/clashx-meta/)|
|FlClash|✔|✔||✔|[使用教程](https://clash.info/flclash/)|
|Karing|✔|✔|✔|✔|[使用教程](https://clash.info/karing/)|
|Clash Party|✔|✔|||[使用教程](https://clash.info/clash-party/)|
|Stash||✔|✔||[使用教程](https://clash.info/stash/)|
|Hiddify|✔|✔|✔|✔|[使用教程](https://clash.info/hiddify/)|
|Mihomo Party|✔|✔|||[使用教程](https://clash.info/mihomo-party/)|
|Clash Nyanpasu|✔|✔|||[使用教程](https://clash.info/clash-nyanpasu/)|
# VPN服务商
## Just My Socks 购买教程

[https://justmysocks1.net](https://justmysocks1.net/)

https://justmysocks3.net/

[https://justmysocks3.net/members/viewinvoice.php?id=5718740](https://justmysocks3.net/members/viewinvoice.php?id=5718740)

https://www.jiongjun.cc/banwagong/698.html

8376eric@163.com

lz8376

500GB/mo on 2.5 Gbps | 5 devices   50 

## 杨帆云

扬帆云.com/yangfanhome.com

eric_83@qq.com
lz13606935895
# 其他各种机场
https://clash.info/jichangtuijian/


## VPN设置
### PAC
为什么大家喜欢用PAC模式？
PAC模式在“省心”和“高效”之间找到了一个很好的平衡，这也是它被广泛使用的原因：

智能分流，兼顾速度与访问范围：它能让你流畅地访问国内网站（直连），同时也能访问需要代理的国外或特定网站。全局模式虽然也能访问所有网站，但会导致国内网站变慢。

节省代理流量和资源：只有被规则匹配的流量才会经过代理服务器，这可以节省代理服务器的流量和带宽，对于按流量计费的代理服务尤其有用。

免去手动切换的麻烦：配置好一次后，它就全自动运行了。你不需要在访问不同网站时，频繁地去系统设置里勾选或取消代理。

灵活的定制能力：如果觉得某个网站没有被智能分流（比如一个国外小众网站无法访问），你可以手动编辑PAC文件，把它的地址加入“走代理”的规则中。

?? 如何配置PAC模式？
配置PAC模式通常有两种场景：

使用代理客户端软件：这是最常见的个人使用场景。像Clash、V2RayN等软件通常都会内置PAC模式选项。你只需要在软件界面中，将代理模式从“全局”切换到“PAC”或“规则模式”即可，软件会自动管理和更新PAC规则。

在操作系统或浏览器中手动配置：如果你有自己的PAC文件地址（比如公司提供的），可以在系统网络设置里找到“自动代理配置”或类似选项，然后在“自动配置脚本”或“使用自动配置脚本”的输入框中，填上PAC文件的URL（网络地址）或本地文件路径即可

## 具体的应用
### git
verge
git config --global https.proxy http://127.0.0.1:7897
git config --global http.proxy http://127.0.0.1:7897

git config --global --unset https.proxy
git config --global --unset http.proxy