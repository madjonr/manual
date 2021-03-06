---
refcn: chapter_02/01_overview
refen: configuration/overview
---

# 配置文件格式

V2Ray 的配置文件形式如下，客户端和服务器通用一种形式，只是实际的配置不一样。

```javascript
{
  "log": {},
  "api": {},
  "dns": {},
  "stats": {},
  "routing": {},
  "policy": {},
  "reverse": {},
  "inbound": {},
  "outbound": {},
  "inboundDetour": [],
  "outboundDetour": [],
  "transport": {}
}
```

其中：

* `log`: 日志配置，见下文；
* `api`: 内置的远程控置 API，详见[远程控制配置](api.md)；
* `dns`: 内置的 DNS 服务器，若此项不存在，则默认使用本机的 DNS 设置。详见[DNS 配置](04_dns.md)；
* `routing`: [路由配置](03_routing.md)；
* `policy`: 本地策略可进行一些权限相关的配置，详见[本地策略](policy.md)；
* `inbound`: 入站连接配置，见下文；
* `outbound`: 出站连接配置，见下文；
* `inboundDetour`: 额外的入站连接配置，见下文；
* `outboundDetour`: 额外的出站连接配置，见下文；
* `transport`: 用于配置 V2Ray 如何与其它服务器建立和使用网络连接。详见[底层传输配置](05_transport.md)；
* `stats`: 当此项存在时，开启[统计信息](stats.md)。
* `reverse`: [反向代理](reverse.md)配置。

## 日志配置 {#log}

```javascript
{
  "access": "文件地址",
  "error": "文件地址",
  "loglevel": "warning"
}
```

其中：

* `access`: 访问日志的文件地址，其值可以是：
  * 一个合法的文件地址，如`"/tmp/v2ray/_access.log"`（Linux）或者`"C:\\Temp\\v2ray\\_access.log"`（Windows）；
  * 或者留空表示不记录访问日志，并将日志输出至 stdout。
* `error`: 错误日志的文件地址，其值可以是：
  * 一个合法的文件地址，如`"/tmp/v2ray/_error.log"`（Linux）或者`"C:\\Temp\\v2ray\\_error.log"`（Windows）；
  * 或者留空表示不记录错误日志，并将日志输出至 stdout。
* `loglevel`: 错误日志的级别，可选的值为`"debug"`、`"info"`、`"warning"`、`"error"` 和 `"none"`：
  * 其中`"debug"`记录的数据最多，`"error"`记录的最少；
  * `"none"`表示不记录任何内容；
  * 默认值为`"warning"`。

日志信息分级:

* `debug`: 只有开发人员能看懂的信息
* `info`: V2Ray 在运行时的状态，不影响正常使用
* `warning`: V2Ray 遇到了一些问题，通常是外部问题，不影响 V2Ray 的正常运行，但有可能影响用户的体验
* `error`: V2Ray 遇到了无法正常运行的问题，需要立即解决

## 主入站连接配置 {#inbound}

入站连接用于接收从客户端（浏览器或上一级代理服务器）发来的数据，可用的协议请见[协议列表](02_protocols.md)。

```javascript
{
  "port": 1080,
  "listen": "127.0.0.1",
  "protocol": "协议名称",
  "settings": {},
  "streamSettings": {},
  "tag": "标识",
  "domainOverride": ["http", "tls"],
  "sniffing": {
    "enabled": false,
    "destOverride": ["http", "tls"]
  }
}
```

其中：

* `port`: 端口。接受的格式如下:
  * 整型数值: 实际的端口号。
  * 环境变量: 以`"env:"`开头，后面是一个环境变量的名称，如`"env:PORT"`。V2Ray 会以字符串形式解析这个环境变量。
  * 字符串: 一个数值类型的字符串，如`"1234"`。
* `listen`: 监听地址，只允许 IP 地址，默认值为`"0.0.0.0"`。
* `protocol`: 连接协议名称，可选的值见[协议列表](02_protocols.md)。
* `settings`: 具体的配置内容，视协议不同而不同。
* `streamSettings`: [底层传输配置](05_transport.md#分连接配置)。
* `tag`: 此入站连接的标识，用于在其它的配置中定位此连接。属性值必须在所有 tag 中唯一。
* `domainOverride`: 识别相应协议的流量，并根据流量内容重置所请求的目标。
  * 接受一个字符串数组，默认值为空。
  * 可选值为 `"http"` 和 `"tls"`。
  * (V2Ray 3.32+) 已弃用。请使用 `sniffing`。当指定了 `domainOverride` 而没有指定 `sniffing` 时，强制开启 `sniffing`。
* `sniffing` (V2Ray 3.32+): 尝试探测流量类型。
  * `enabled`: 是否开启流量探测。
  * `destOverride`: 当流量为指定类型时，按其中包括的目标地址重置当前连接的目标。
    * 可选值为 `"http"` 和 `"tls"`。

## 主出站连接配置 {#outbound}

主出站连接用于向远程网站或下一级代理服务器发送数据，可用的协议请见[协议列表](02_protocols.md)。

```javascript
{
  "sendThrough": "0.0.0.0",
  "protocol": "协议名称",
  "settings": {},
  "tag": "标识",
  "streamSettings": {},
  "proxySettings": {
    "tag": "another-outbound-tag"
  },
  "mux": {}
}
```

其中：

* `sendThrough`: 用于发送数据的 IP 地址，当主机有多个 IP 地址时有效，默认值为`"0.0.0.0"`。
* `protocol`: 连接协议名称，可选的值见[协议列表](02_protocols.md)。
* `settings`: 具体的配置内容，视协议不同而不同。
* `tag`: 此出站连接的标识，用于在其它的配置中定位此连接。属性值必须在所有 tag 中唯一。
* `streamSettings`: [底层传输配置](05_transport.md#分连接配置)。
* `proxySettings`: 出站代理配置。当出站代理生效时，此出站协议的`streamSettings`将不起作用。
  * `tag`: 当指定另一个出站协议的标识时，此出站协议发出的数据，将被转发至所指定的出站协议发出。
* `mux`: [Mux 配置](mux.md)。

## 额外的入站连接配置 {#inbound-detour}

此项是一个数组，可包含多个连接配置，每一个配置形如：

```javascript
{
  "protocol": "协议名称",
  "port": "端口",
  "tag": "标识",
  "listen": "127.0.0.1",
  "allocate": {
    "strategy": "always",
    "refresh": 5,
    "concurrency": 3
  },
  "settings": {},
  "streamSettings": {},
  "domainOverride": ["http", "tls"],
  "sniffing": {
    "enabled": false,
    "destOverride": ["http", "tls"]
  }
}
```

其中：

* `protocol`: 连接协议名称，可选的值见[协议列表](02_protocols.md)。
* `port`: 端口。接受的格式如下:
  * 整型数值: 实际的端口号。
  * 环境变量 (V2Ray 3.23+): 以`"env:"`开头，后面是一个环境变量的名称，如`"env:PORT"`。V2Ray 会以字符串形式解析这个环境变量。
  * 字符串: 可以是一个数值类型的字符串，如`"1234"`；或者一个数值范围，如`"5-10"`表示端口 5 到端口 10 这 6 个端口。
* `tag`: 此入站连接的标识，用于在其它的配置中定位此连接。属性值必须在所有 tag 中唯一。
* `listen`: 监听地址，只允许 IP 地址，默认值为`"0.0.0.0"`。
* `allocate`: 分配设置：
  * `strategy`: 分配策略，可选的值有`"always"`和`"random"`两个。`"always"`表示总是分配所有已指定的端口，port 是指定了多少个端口，V2Ray 就会监听这些端口。random 表示随机开放端口，每隔 refresh 分钟在 port 范围中随机选取 concurrency 个端口来监听。
  * `refresh`: 随机端口刷新间隔，单位为分钟。最小值为`2`，建议值为`5`。这个属性仅当 strategy = random 时有效。
  * `concurrency`: 随机端口数量。最小值为`1`，最大值为 port 范围的三分之一。建议值为`3`。
* `settings`: 具体的配置内容，视协议不同而不同。
* `streamSettings`: [底层传输配置](05_transport.md#分连接配置)。
* `domainOverride`: 识别相应协议的流量，并根据流量内容重置所请求的目标。
  * 接受一个字符串数组，默认值为空。
  * 可选值为 `"http"` 和 `"tls"`。
  * (V2Ray 3.32+) 已弃用。请使用 `sniffing`。当指定了 `domainOverride` 而没有指定 `sniffing` 时，强制开启 `sniffing`。
* `sniffing` (V2Ray 3.32+): 尝试探测流量类型。
  * `enabled`: 是否开启流量探测。
  * `destOverride`: 当流量为指定类型时，按其中包括的目标地址重置当前连接的目标。
    * 可选值为 `"http"` 和 `"tls"`。

### 额外的出站连接配置 {#outbound-detour}

此项是一个数组，可包含多个连接配置，每一个配置形如：

```javascript
{
  "protocol": "协议名称",
  "sendThrough": "0.0.0.0",
  "tag": "标识",
  "settings": {},
  "streamSettings": {},
  "proxySettings": {
    "tag": "another-outbound-tag"
  },
  "mux": {}
}
```

其中：

* `protocol`: 连接协议名称，可选的值见[协议列表](02_protocols.md)；
* `sendThrough`: 用于发送数据的 IP 地址，当主机有多个 IP 地址时有效，默认值为`"0.0.0.0"`。
* `tag`: 当前的配置标识，当路由选择了此标识后，数据包会由此连接发出；
* `settings`: 具体的配置内容，视协议不同而不同。
* `streamSettings`: [底层传输配置](05_transport.md#分连接配置)。
* `proxySettings`: 出站代理配置。当出站代理生效时，此出站协议的`streamSettings`将不起作用。
  * `tag`: 当指定另一个出站协议的标识时，此出站协议发出的数据，将被转发至所指定的出站协议发出。
* `mux`: [Mux 配置](mux.md)。
