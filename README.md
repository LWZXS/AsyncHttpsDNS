# AsyncHttpsDNS
DNS Over Https Powered By Asyncio
思路基本和PRCDNS是一样的，其实Google DNS本身就支持EDNS Client IP这个功能，但UDP被污染，TCP的Client IP是基于实际的连接IP，考虑国内网络状况，实际连接IP根本不可能是你本地IP，所以DNS Over Https这种可以自己指定Client IP的才是最好解决办法。

### 为什么要重复造轮子？
1. 学习AsyncIO，尤其是UDP Server，就这么简单。

2. PRCDNS仅仅支持TCP，操蛋的是Dnsmasq不支持TCP Query Only，还得加一层PDNSD才行，这个太浪费了。
虽然考虑到UDP DNS查询的严重劫持情况和UDP本身无连接的脆弱，其实TCP查询是合理的，不过我的想法更简单：只把这玩意部署在路由器上，出去的只有到Google Https的连接。
至于怎么连接到Google Https，你路由器不可能没有SS吧，如果路由器不能翻墙，就用shadowsocks提供的socks5代理，见下方参数。
至于什么样的路由器才能支持Python3和aiohttp，x86软路由当然是王道。


3. PRCDNS没有考虑到一种情况：很多网站其实我们都是走代理的，基于本地IP去做edns client ip查询其实是不合理的。

例如用中国IP去查Google的地址，返回的基本都是台湾的IP，可你的代理如果在日本甚至美国，这个就比较尴尬了，会增加50ms甚至100ms的延迟。

4. DNS Over Https有一个鸡生蛋蛋生鸡的尴尬：如果DNS Over Https是唯一的解析器，现在你想解析域名就要去访问dns.google.com，可是你不知道dns.google.com的地址，于是你向自己查询，然后你自己又去问dns.google.com这个dns.google.com的地址是什么，于是。。。。

5. PRCDNS只解析A记录，这个，如果只作为抗污染辅助手段还是可以的，如果要作为主DNS服务器，这个还是有点不能接受的，毕竟这是残缺的。

### Help:

```
usage: AsyncHttpsDNS [-h] [-p [PORT]] [-i [IP]] [-f [FILE]] [-d [DEBUG]]
                     [-s [SOCKS]]

optional arguments:
  -h, --help            show this help message and exit
  -p [PORT], --port [PORT]
                        Port for async dns server to listen
  -i [IP], --ip [IP]    IP of proxy server to bypass gfw
  -f [FILE], --file [FILE]
                        file that contains blocked domains
  -d [DEBUG], --debug [DEBUG]
                        enable debug logging
  -s [SOCKS], --socks [SOCKS]
                        socks proxy IP:Port in format like: 127.0.0.1:1086
```

1. AsyncHttpsDNS=>直接启动一个在5454端口的DNS服务器，无代理(如果你可以直连dns.google.com的话)，无翻墙域名优化，
2. AsyncHttpsDNS -s 127.0.0.1:1086,同上，但使用127.0.0.1:1086地址上的socks5代理(shadowsocks默认地址)，
3. AsyncHttpsDNS -s 127.0.0.1:1086 -i 45.32.15.77，同上，但请求被墙的域名是使用45.32.15.77作为client ip，对应域名解析的结果为此IP优化，
4. AsyncHttpsDNS -s 127.0.0.1:1086 -i 45.32.15.77 -f mydomains.txt，同上，但使用当前目录的mydomains.txt文件作为被墙域名列表，
5. sudo AsyncHttpsDNS -s 127.0.0.1:1086 -i 45.32.15.77 -f mydomains.txt -p 53,同上,但监听本地53端口。
### 使用注意：

1. 默认监听5454端口，无缓存，配合dnsmasq使用已经足够了，如果要改成53端口，需要有sudo权限。
2. 默认使用114和腾讯DNS解析dns.google.com地址，这个在国内用基本没啥问题，如果你本地的114和腾讯DNS都会被污染，自己改代码，为这个加参数挺无聊的。
3. foreign_domains.txt保存的是国外域名的地址，其实这个就是gfwlist域名，原因不用解释。实际上这个列表不需要这么大，只需要被屏蔽的又有CDN服务器的域名即可。
4. 重点！！使用-i参数指定你的代理服务器IP地址，默认的是日本Vultr地址，不一定合适你。


### 安装
```
pip3 install asynchttpsdns
```


