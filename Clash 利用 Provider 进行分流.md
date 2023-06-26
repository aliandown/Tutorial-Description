proxy-providers: 提供节点
rule-providers: 提供(在线)规则集
proxy-groups: 策略组，对 proxy-providers 提供的节点或手动添加的节点进行分组管理
rules: 规则，对 rule-providers 提供的规则集或手动添加的规则进行管理，设置由哪个策略组进行访问
---------
如：RULE-SET,telegramcidr,PROXY

表示：当访问的域名或 IP 在 telegramcidr 规则集中时，使用 PROXY 策略组进行访问

配置

1.proxy-providers

proxy-providers:
  <name>:  # 代理集名称
    type: http | file  # 代理集类型：http(clash 订阅链接) | file(本地文件)
    path: ./profiles/proxies/<name>.yaml  # 代理集存放路径，可以为绝对路径，相对路径相对于 clash home 目录
    url: <url>  # 代理集链接（若 type 为 http）
    filter: ''  # 节点过滤器
    interval: 3600  # 代理集更新间隔（若 type 为 http）
    health-check:  # 用于对代理集所包含的节点设置自动延迟测试
      enable: true
      interval: 300  # 间隔时间
      url: http://www.gstatic.com/generate_204  # 测试地址
      
<name>: 代理集名称，可以随意命名，但是不能重复
type: 代理集类型，可以是 http 或 file
path: 代理集存放路径
url: 代理集链接，只有当 type 为 http 时才需要填写
filter: 节点过滤器，可以筛选出符合条件的节点。用 | 分隔
ABC | DEF: 指筛选出节点名称中包含 ABC 或 DEF 的节点
interval: 代理集更新间隔，只有当 type 为 http 时才需要填写
health-check: 用于对代理集所包含的节点设置自动延迟测试
enable: 是否开启自动延迟测试
interval: 自动延迟测试间隔时间
url: 自动延迟测试地址

2.proxy-groups

proxy-groups:
  - name: PROXY
    type: url-test
    interval: 3600
    url: http://www.gstatic.com/generate_204
    use:
      - <name>
    proxies:
      - <name>

name: 策略组名称，可以随意命名，但是不能重复
PROXY: 必须有一个 PROXY 策略组，因为 PROXY 为默认的策略组
type: 策略组类型
url-test: 通过测试地址测试节点延迟，选择延迟最低的节点
select: 手动选择节点
interval: 自动执行节点延迟测试的间隔，只有当 type 为 url-test 时才需要填写
url: 节点延迟测试地址，只有当 type 为 url-test 时才需要填写
use: 引用 proxy-provider 中的代理集，填写对应的代理集名称
proxies: 可以在此手动引用单个节点配置或其它策略组
DIRECT: 若使用 openclash 工具，需要添加一个 DIRECT 策略组，否则可能无法正常下载订阅链接中的配置文件
默认策略组： DIRECT、REJECT、PROXY

3.rule-providers

rule-providers:
  <name>:  # 规则集名称
    type: http | file  # 规则集文件类型：http(在线规则集) | file(本地文件)
    behavior: # 规则集类型classical/ipcidr/domain 
    path: ./profiles/rules/<name>.yaml  # 规则集存放路径
    url: <url>  # 规则集链接（若 type 为 http）
    interval: 86400  # 规则集更新间隔（若 type 为 http）
<name>: 规则集名称，可以随意命名，但是不能重复
type: 规则集文件类型，可以是 http 或 file
behavior: 规则集行为，可以是 classical | ipcidr | domain (参考示例：Example of a rule-provider file)
path: 规则集存放路径
url: 规则集链接，只有当 type 为 http 时才需要填写
interval: 规则集更新间隔（如 86400 秒指每 10 天更新一次规则集），只有当 type 为 http 时才需要填写

4.rules

使用上面的rule-provider引入的规则集，或者手动添加的规则，设置由哪个策略组进行访问。

白名单模式（推荐）
即没有命中规则的域名或 IP 都走代理。适用于线路稳定、快速、不缺流量的情况。

1
2
3
4
5
6
7
8
9
10
11
12
13
14
15
16
17
COPY
rules:
  - RULE-SET,applications,DIRECT
  - DOMAIN,clash.razord.top,DIRECT
  - DOMAIN,yacd.haishan.me,DIRECT
  - RULE-SET,private,DIRECT
  - RULE-SET,reject,REJECT
  - RULE-SET,icloud,DIRECT
  - RULE-SET,apple,DIRECT
  - RULE-SET,google,DIRECT
  - RULE-SET,proxy,PROXY
  - RULE-SET,direct,DIRECT
  - RULE-SET,lancidr,DIRECT
  - RULE-SET,cncidr,DIRECT
  - RULE-SET,telegramcidr,PROXY
  - GEOIP,LAN,DIRECT
  - GEOIP,CN,DIRECT
  - MATCH,PROXY

黑名单模式
即没有命中规则的域名或 IP 都不走代理。适用于线路不稳定、速度慢、流量有限的情况。

1
2
3
4
5
6
7
8
9
10
11
COPY
rules:
  - RULE-SET,applications,DIRECT
  - DOMAIN,clash.razord.top,DIRECT
  - DOMAIN,yacd.haishan.me,DIRECT
  - RULE-SET,private,DIRECT
  - RULE-SET,reject,REJECT
  - RULE-SET,tld-not-cn,PROXY
  - RULE-SET,gfw,PROXY
  - RULE-SET,greatfire,PROXY
  - RULE-SET,telegramcidr,PROXY
  - MATCH,DIRECT
配置文件示例
查看配置文件范例
在 proxy-groups 组中我添加了一个名为 AUTO 的代理组，用于为 PROXY 组自动选择代理节点，这样可以避免首次访问需要代理的网站偶尔被直连的问题，且能实现 PROXY 组可以手动切换节点，也可以自动选择节点。

注意事项
由于需要下载的文件较多，因此第一次使用可能会因为超时而报错失败，多重试几次就好了。
对于 base64 编码的配置文件，无法直接使用 proxy-providers 导入，需要使用 Clash 链接转换工具将其转换后再导入。
如：https://clash.back2me.cn/
题外话
我使用的 Clash For Windows 在使用上面的方案时，明明填的存放路径都是 profiles 目录，但是他们都存放到了 providers 目录，并且文件名都是对应的 md5。
      
