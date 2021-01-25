## Padavan设置DDNS访问路由器管理界面并实现远程桌面控制本地电脑

### 环境

- 中国移动宽带
- 斐讯K2路由器刷的Padavan固件
- 本地Win10笔记本

记得很久以前用的长城宽带，打电话给客服要到了固定的ipv4，然后ddns用的花生壳还是3322。
现在换成移动宽带，也想弄弄，打电话结果被告知移动不提供固定ip，后来网上查到现在很多isp都提供ipv6，那就相当于固定的了。

### 步骤
#### 让路由器代替移动光猫进行拨号  
超密登录光猫先备份默认的拨号设置（我这边是2_INTERNET_R_VID_1101），然后删掉这个，新建一个ipv4/ipv6的桥接设置。此时光猫就无法拨号，无法上网了。
然后登录路由器去WAN里面设置成pppoe拨号上网
#### 找DDNS服务商  
免费并支持ipv6的不好找，而且很多最后可以解析并回传但域名根本无法访问，如最开始我用的dynv6：  
最开始是**直接在WAN-DDNS里面设置**，但是路由器总是报错，跟这个帖子的error message一样  
https://www.right.com.cn/forum/thread-350896-1-1.html  
首先说明：我在DYNV6网站里设置了我的主机IPV6地址后是可以在外网连接主机的。我如下图在padavan里设置了，但日志报错：  
Nov  4 13:13:11 inadyn[989]: Inadyn version 1.99.15 -- Dynamic DNS update client.  
Nov  4 13:13:11 inadyn[989]: Failed resolving hostname XXXXX.dynv6.net: Name or service not known  
Nov  4 13:13:11 inadyn[989]: Checking for IP# change, connecting to checkip.dyndns.org (216.146.43.70:80)  
Nov  4 13:13:11 inadyn[989]: Network error while waiting for reply: Connection reset by peer  
Nov  4 13:13:11 inadyn[989]: Will retry again after 5 min...  
但比这个情况更差，他外网可以连，我ping不通更连不了，有点像https://www.right.com.cn/forum/thread-4059214-1-1.html  
接下来只能放弃直接设置，换到在脚本里面设置：  
自定义设置-脚本-自定义脚本0(功能配置)  
dynv6官方文档里用的一种用shell脚本:  
https://dynv6.com/docs/apis  
Use our update script  
We provide a small script that updates your address at dynv6. You can run it with:  
token=自己的token字符串 ./dynv6.sh 自己名字.dynv6.net
这里没设置ssh的需要操作下：  
打开ssh服务，ssh登录到路由器，我一直用的bitvise ssh（打个广告，本人强推这个，tftp很好用。可惜这个软件只有windows版本）。把dynv6.sh传到路由器，这里得传到/etc/storage，所以对应的命令是
```
token=自己的token字符串 /etc/storage/dynv6.sh 自己名字.dynv6.net
```
然后在脚本最后加入即可  
<img src="https://github.com/chinasoul/chinasoul.github.io/blob/main/ddns_pics/WeChat%20Screenshot_20210123223415.png" width="600"/><br/>
重启路由器后就可以在dynv6网站看到自己的ip获取到了  
<img src="https://github.com/chinasoul/chinasoul.github.io/blob/main/ddns_pics/WeChat%20Screenshot_20210123223506.png" width="600"/><br/>
当然可以在crontab里定时执行解析  
<img src="https://github.com/chinasoul/chinasoul.github.io/blob/main/ddns_pics/WeChat%20Screenshot_20210123223736.png" width="600"/><br/>
不过可惜的是，在外网无法ping通也无法在浏览器访问这个xxx.dynv6.net，换成ip就没问题，是不是被抢了。。
后来又试了afraid ddns  
http://freedns.afraid.org/dynamic/
ping的时候会显示出正确的ip不过还是不通  
<img src="https://github.com/chinasoul/chinasoul.github.io/blob/main/ddns_pics/WeChat%20Screenshot_20210123224216.png" width="600"/><br/>

最后duckdns可以
官方文档有参数  
差不多这样就可以https://www.duckdns.org/update?domains=xxxxxxx.duckdns.org&token=8e44xxxx-xxxx-xxxx&ipv6=2409:8xxxxxx
这里要借助dynv6提供的脚本里面有获取ipv6的方法：  
```
address=$(ip -6 addr list scope global $device | grep -v " fd" | sed -n 's/.*inet6 \([0-9a-f:]\+\).*/\1/p' | head -n 1)
......
# address with netmask
current=$address/$netmask #这里current已经存了获取到的ipv6
......
# send addresses to dynv6
$bin "http://dynv6.com/api/update?hostname=$hostname&ipv6=$current&token=$token"
#这里注释掉ipv4因为不需要$bin "http://ipv4.dynv6.com/api/update?hostname=$hostname&ipv4=auto&token=$token"
#为了方便，我直接hardcode自己的域名和token
$bin "https://www.duckdns.org/update?domains=xxxxxx.duckdns.org&token=8e44xxxx-xxx-xxx-xx767&ipv6=$current"
```
这样dynv6和duckdns都能获取了，当然现在可以删掉dynv6了，自己甚至可以改写dynv6.sh让它仅仅适用在duckdns或者直接用curl集成到自定义脚本中然后删掉这个脚本。  
<img src="https://github.com/chinasoul/chinasoul.github.io/blob/main/ddns_pics/WeChat%20Image_20210123230425.jpg" width="200"/><br/>
<img src="https://github.com/chinasoul/chinasoul.github.io/blob/main/ddns_pics/WeChat%20Screenshot_20210123225256.png" width="600"/><br/>

#### 远程控制局域网的电脑桌面  
一直在用vncviewer，但是这个不支持ipv6的地址，网上说可以[ipv6]:[port]但我试过不行  
这里使用微软官方的RD Client  
在这之前有个问题，你怎么获取到电脑的ipv6并ddns解析到域名呢？
还是之前的思路，通过某种方式获取内网电脑的ip，然后在dynv6的脚本里curl
```
laptop_address=$(ip -6 neigh | sed -n 's/\([0-9a-f:]\{26,\}\).*xx:xx:xx:xx:91:ae.*/\1/p')
```
增加一行获取laptop_addr
为什么这么写？  
<img src="https://github.com/chinasoul/chinasoul.github.io/blob/main/ddns_pics/WeChat%20Screenshot_20210123230935.png" width="600"/><br/>
用这个你可以看到局域网内有很多设备，你的笔记本手机等等
```
ip -6 neigh | sed 's/\([0-9a-f:].*\).*/\1/p'
```
通过{26,}限定返回具有完整ipv6的设备，通过.*MAC地址.*来限定返回你的笔记本  
```
laptop_current=$laptop_address/$netmask
......
$bin "https://www.duckdns.org/update?domains=xxxxx.duckdns.org&token=8e4xxxxxx767&ipv6=$laptop_current"
```
用类似的方法就给笔记本也做了ddns  

附上duckdns参数
```
You can update your domain(s) with a single HTTPS get to DuckDNS

https://www.duckdns.org/update?domains={YOURVALUE}&token={YOURVALUE}[&ip={YOURVALUE}][&ipv6={YOURVALUE}][&verbose=true][&clear=true]
The domain can be a single domain - or a comma separated list of domains.
The domain does not need to include the .duckdns.org part of your domain, just the subname.
If you do not specify the IP address, then it will be detected - this only works for IPv4 addresses
You can put either an IPv4 or an IPv6 address in the ip parameter
If you want to update BOTH of your IPv4 and IPv6 records at once, then you can use the optional parameter ipv6
to clear both your records use the optional parameter clear=true
A normal good response is

OK
A normal bad response is

KO
```
如果发现解析出了Ipv4地址（显然这个不应该，因为我们并没有固定的IPV4地址，解析出来会造成无法访问）加上&clear=true清理下
