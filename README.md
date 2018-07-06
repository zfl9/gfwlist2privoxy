# 适用于 privoxy 的 gfwlist.pac（socks5 代理）
- 由于 macOS 的命令行参数与 Linux 命令行参数部分不一致，macOS 用户请使用 `gfwlist2privoxy-mac-zsh` 代替 `gfwlist2privoxy`。使用 `zsh` 能得到更好的提示效果。-- [c-rainstorm](https://github.com/c-rainstorm)
- 现使用 Perl5 替代 sed、grep 等正则处理工具，应该不存在所谓参数不一致问题了，请先尝试 `gfwlist2privoxy`。-- [zfl9](https://github.com/zfl9)

## 脚本依赖
- base64
- curl (https)
- Perl5 v5.10.0+

## 脚本用法
- 运行 socks5 代理，比如 [ss-local](https://www.zfl9.com/ss-local.html)，监听端口随意（如 1080 端口）
- `curl -4sSkL https://raw.github.com/zfl9/gfwlist2privoxy/master/gfwlist2privoxy -O`
- `sh gfwlist2privoxy 127.0.0.1:1080`，请注意将 `127.0.0.1:1080` 替换为你的 socks5 代理地址
- `cp -af gfwlist.action /etc/privoxy/`，将 gfwlist.action 拷贝到 privoxy 配置目录
- `echo 'actionsfile gfwlist.action' >>/etc/privoxy/config`，应用 gfwlist.action 配置
- `systemctl restart privoxy.service`，重启 privoxy 服务
- `systemctl -l status privoxy.service`，检查 privoxy 运行状态
- 设置环境变量（建议添加到 bashrc、zshrc 等文件中，方便使用），注意将 8118（默认）替换为 privoxy 的端口号
- `proxy=http://127.0.0.1:8118; export http_proxy=$proxy https_proxy=$proxy no_proxy="localhost, 127.0.0.1, ::1"`
- 如果你没有运行 gfwlist2privoxy 脚本的条件，也可以从 http://main.zfl9.com/gfwlist.action 下载（6 小时更新一次）

## 代理测试
- 访问墙内网站：`curl -4sSkL https://www.baidu.com`，走直连，tcpdump 抓包可验证
- 访问墙外网站：`curl -4sSkL https://www.google.com`，走代理，实现了 gfwlist PAC 效果
- 查看当前 IP：`curl -4sSkL http://ip.chinaz.com/getip.aspx`，按道理来说，会显示本机 IP

## Adblock Plus 规则
- 注释符：`!`开头的行均为注释行，会被 Adblock Plus 忽略，当然还有特殊注释，此处略过
- 通配符：`*`匹配任意长度字符，并且默认假设字符串两边存在`*`，即`ad`与`*ad*`是一样的
- 边界符：`|`位于模式首尾，用于取消首尾默认的`*`通配符，比如`|https://www.google.com/ad`
- 子域符：`||`位于模式头部，用于匹配`http://*.`、`https://*.`协议字符串及子域部分（含当前域名）
- 分隔符：`^`匹配 url 分隔符，如`http://192.168.1.1:8080/?a=1&b=2`中的红色部分以及地址结尾（见下图）
- 正则符：`/regex/`使用 Unix 路径分隔符括住正则表达式（Perl 正则）（应避免大量正则的使用，性能较低）
- 例外符：`@@`开头的模式，表示定义一个例外规则，如`@@||google.com^`、`@@/https?:\/\/.*\.google\..*/`

![Adblock Plus 中的 `^` 模式字符](http://main.zfl9.com/images/abp-url-split-rule.png)

## privoxy.action 规则
- 协议部分：privoxy 已经默认假设存在`http://`、`https://`，因此不能再定义协议字符串
- host 部分：使用 glob 模式，即`*`任意长度字符、`?`任意单个字符、`[set]`集合、`[^set]`集合（取反）
- host 部分可以指定端口号，语法和平时指定端口号一样，比如：`www.zfl9.com:8989`、`192.168.1.1:8080`
- host 部分除了可以是域名外，还可以是 IPv4、IPv6 地址，如果为 IPv6 地址，需用`<>`括起来（取消歧义）
- uri 部分：使用 regex 模式（POSIX 扩展正则表达式，ERE，POSIX 1003.2），不支持非贪婪匹配等高级特性

host 模式例子：

- `/`，匹配所有 URL 请求，这是一个特殊的模式；
- `:8080`，匹配所有目的端口为 8080 的请求；
- `192.168.1.1`，匹配目的主机 192.168.1.1 的所有请求；
- `<2001:db8::1>`，匹配目的主机 2001:db8::1 的所有请求；
- `www.zfl9.com`，匹配 www.zfl9.com 下的所有请求；
- `www.zfl9.com/`，匹配 www.zfl9.com 下的所有请求，末尾的 / 可以省略；
- `www.`，匹配所有以 www. 开头的域名下的所有请求（包括 www）；
- `.google.com`，匹配所有以 .google.com 结尾的域名下的所有请求（包括 google.com）；
- `.google.`，匹配所有包含 .google. 内容的域名下的所有请求（包括 google）；

典型的 privoxy.action：

``` bash
### 定义别名，可包含除空格、TAB、=、{} 外的任意字符
{{alias}}
# 代理(socks5)
socks5 = +forward-override{forward-socks5 127.0.0.1:1080 .}
# 直连
direct = +forward-override{forward .}

### 位于后面的规则优先于前面的规则，因此例外规则必须写在后面，否则无法生效
# 所有网站走代理 (一般情况写在前面)
{socks5}
/

# 以下网站走直连 (特殊情况写在后面)
{direct}
.ip.cn
.chinaz.com
```

更多 privoxy 用法请参考：[privoxy - 用户手册](https://www.privoxy.org/user-manual/)；
