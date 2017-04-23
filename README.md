## gfw.action 配合 shadowsocks 使用，达到 Windows|Mac|Android|IOS 使用PAC模式的效果
- 在线获取gfwlist.txt, 转换为privoxy可用的格式

## 用法
- `wget https://raw.github.com/zfl9/gfwlist2privoxy/master/gfwlist2privoxy && bash gfwlist2privoxy`
- `cp -af gfw.action /etc/privoxy/`
- `echo 'actionsfile gfw.action' >> /etc/privoxy/config`
- `systemctl restart privoxy`
- 运行shadowsocks sslocal
- 设置term代理
- `export http_proxy=http://127.0.0.1:8118 https_proxy=http://127.0.0.1:8118 ftp_proxy=http://127.0.0.1:8118`

## 测试
- `curl ip.cn` 不走代理，显示本机IP
- `curl www.google.com` 302重定向
- `curl -L https://www.google.com` 成功科学上网!

## 更多请戳 https://www.zfl9.com/sslocal+privoxy.html

- 2017-04-23 16:04:48
