## gfw.action 配合 ss-local 使用，达到 gfwlist PAC 效果
- 在线获取gfwlist.txt, 转换为privoxy可用的格式

## 用法
- `wget https://raw.github.com/zfl9/gfwlist2privoxy/master/gfwlist2privoxy && bash gfwlist2privoxy`
- 输入ss-local的监听地址和端口号; 如：`127.0.0.1:1080`
- `cp -af gfw.action /etc/privoxy/`，拷贝配置文件到 privoxy 目录
- `echo 'actionsfile gfw.action' >> /etc/privoxy/config`，修改 privoxy 主配置文件
- `systemctl restart privoxy`，重启 privoxy 进程，注意查看运行状态
- 运行shadowsocks ss-local
- 设置shell proxy环境变量
- `export http_proxy=http://127.0.0.1:8118 https_proxy=http://127.0.0.1:8118`
- 这里也提供了经脚本转换的 gfw.action 成品，可直接使用

## 测试
- 访问大陆网站，`curl -sL www.baidu.com`，走直连，使用 tcpdump 抓包可验证
- 访问Google，`curl -sL www.google.com`，走代理，实现了我们想要的 gfwlist PAC 效果
- 查看当前IP，`curl -sL ip.chinaz.com/getip.aspx`，输出本地IP说明配置无误

## ss-local 教程
[ss-local + privoxy 实现 shell 终端代理](https://www.zfl9.com/ss-local.html)

- 2017-07-21 by Otokaze