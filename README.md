## gfw.action 配合 ss-local 使用，达到 Windows|Mac|Android|IOS 使用PAC模式的效果
- 在线获取gfwlist.txt, 转换为privoxy可用的格式

## 用法
- `wget https://raw.github.com/zfl9/gfwlist2privoxy/master/gfwlist2privoxy && bash gfwlist2privoxy`
- 输入ss-local的监听地址和端口号; 如：`127.0.0.1:1080`
- `cp -af gfw.action /etc/privoxy/`
- `echo 'actionsfile gfw.action' >> /etc/privoxy/config`
- `systemctl restart privoxy`
- 运行shadowsocks ss-local
- 设置shell http/https/ftp_proxy环境变量
- `export http_proxy=http://127.0.0.1:8118 https_proxy=http://127.0.0.1:8118 ftp_proxy=http://127.0.0.1:8118`
- 如果你不想自己转换, 这里也有现成的gfw.action, 也是这个脚本转换的, 你可以直接放在privoxy目录下, 直接用

## 测试
- `curl ip.cn` 不走代理，直连，显示本机IP
- `curl www.google.com` 302重定向
- `curl -sL https://www.google.com` privoxy -> ss-local -> ss-server -> target_host，成功代理

## 更多请戳 https://www.zfl9.com/ss-local.html

- 2017-07-21 by Otokaze
