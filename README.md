# 适用于 privoxy 的 gfwlist.pac（socks5 代理）

> 在线获取 gfwlist.txt 并将其转换为 privoxy.action，以达 gfwlist.pac 效果

## macOS 版本

- 由于 macOS 的命令行参数与 Linux 命令行参数部分不一致，macOS 用户请使用 `gfwlist2privoxy-mac-zsh` 代替 `gfwlist2privoxy`。使用 `zsh` 能得到更好的提示效果。

## 用法

- 启动你的 socks5 代理（或确保它现在为可用状态），我自己使用的是 [ss-local](https://www.zfl9.com/ss-local.html)，监听在 1080 端口；
- `curl -skL https://raw.github.com/zfl9/gfwlist2privoxy/master/gfwlist2privoxy -o gfwlist2privoxy`
- `bash gfwlist2privoxy '127.0.0.1:1080'`，请注意将 "127.0.0.1:1080" 替换为你的 socks5 代理地址；
- `cp -af gfwlist.action /etc/privoxy/`，将 gfwlist.action 拷贝到 privoxy 目录；
- `echo 'actionsfile gfwlist.action' >> /etc/privoxy/config`，启用 gfwlist.action 动作文件；
- `systemctl restart privoxy.service`，重启 privoxy 服务；
- `systemctl -l status privoxy.service`，检查 privoxy 运行状态；
- `proxy=http://127.0.0.1:8118; export http_proxy=$proxy https_proxy=$proxy no_proxy="localhost, 127.0.0.1, ::1"`
- 当然，如果你不想自己转换，也可以直接使用提供的 gfwlist.action 成品，它也是使用脚本转换的。

## 测试

- 访问未墙网站：`curl -skL www.baidu.com`，走直连，使用 tcpdump 抓包可验证；
- 访问被墙网站：`curl -skL www.google.com`，走代理，实现了 gfwlist PAC 效果；
- 查看当前 IP：`curl -skL ip.chinaz.com/getip.aspx`，按道理来说，会显示本机 IP；

## adblockplus 规则

- 通配符：`*`任意长度字符，并且默认假设在字符串两边存在`*`，即`ad`与`*ad*`是一样的；
- 边界符：`|`位于模式边界，用于取消默认的边界`*`通配符，如`|https://www.google.com/`；
- 协议符：`||`位于模式开头，用于匹配`http://*.`、`https://*.`协议字符串及子域名部分；
- 分隔符：`^`url 分隔符，如`http://192.168.1.1:8080/?a=1&b=2`的`//`、`:`、`/`、`?`、`=`、`&`及末尾；
- 正则符：`/regex/`使用 Unix 路径分隔符包住正则表达式（Perl 正则）即可（应避免正则的使用，性能较低）；
- 例外符：`@@`开头的模式，表示定义一个例外规则，如`@@||google.com^`、`@@/https?:\/\/.*\.google\..*/`；
- 注释符：`!`开头的行均为注释行，会被 adblockplus 忽略，当然还有特殊注释，此处略过。

## privoxy.action 规则

- scheme 部分：因为 privoxy 已经默认假设存在`http://`、`https://`，因此不能再定义协议字符串；
- host 部分：使用 globbing 模式，即`*`任意长度字符、`?`任意单个字符、`[set]`集合、`[^set]`集合（取反）；
- host 部分是可以指定端口号的，和平时指定端口号一样，比如：`www.zfl9.com:8989`、`192.168.1.1:8080`；
- host 部分~~若为 IP 地址，则不能使用通配符~~，通配符适用于 IP 地址；如果为 IPv6 地址，需用`<>`括起来；
- uri 部分：使用 regex 模式（扩展正则，POSIX 1003.2），因此不支持非贪婪匹配等 Perl 正则特性。

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

## 关于

- 2018-01-27 09:18:34 CST
- Otokaze（zfl9.com@gmail.com）
