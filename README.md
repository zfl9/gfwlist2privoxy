## 适用于 privoxy 的 gfwlist.pac（socks5 代理）
> 在线获取 gfwlist.txt 并将其转换为 privoxy.action 文件，以达到 gfwlist.pac 效果

## 用法
- 启动你的 socks5 代理（或确保它现在为可用状态），我自己使用的是 ss-local，监听在 1080 端口；
- `curl -skL https://raw.github.com/zfl9/gfwlist2privoxy/master/gfwlist2privoxy -o gfwlist2privoxy`
- `bash gfwlist2privoxy '127.0.0.1:1080'`，请注意将 "127.0.0.1:1080" 替换为你的 socks5 代理地址；
- `cp -af gfwlist.action /etc/privoxy/`，将 gfwlist.action 拷贝到 privoxy 目录；
- `echo 'actionsfile gfwlist.action' >> /etc/privoxy/config`，启用 gfwlist.action 动作文件；
- `service privoxy restart`，（sysvinit）重启 privoxy 服务；
- `systemctl restart privoxy.service`，（systemd）重启 privoxy 服务；
- `export http_proxy=http://127.0.0.1:8118 https_proxy=http://127.0.0.1:8118`，设置环境变量；
- 当然，如果你不想自己转换，也可以直接使用提供的 gfwlist.action 成品，它也是使用脚本转换的。

## 测试
- 访问未墙网站：`curl -skL www.baidu.com`，走直连，使用 tcpdump 抓包可验证；
- 访问被墙网站：`curl -skL www.google.com`，走代理，实现了 gfwlist PAC 效果；
- 查看当前 IP：`curl -skL ip.chinaz.com/getip.aspx`，按道理来说，会显示本机 IP；

## ss-local 相关
如果你还不知道什么是 socks5 代理，你可以前往 - [ss-local + privoxy 代理](https://www.zfl9.com/ss-local.html)。

- 2017-11-27 19:48:03 CST
- Otokaze（j1002238565@gmail.com）
