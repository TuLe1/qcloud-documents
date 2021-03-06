<style rel="stylesheet">
table th:nth-of-type(1){
width:200px;
}</style>
<style rel="stylesheet">
table th:nth-of-type(2){
width:200px;
}</style>
<style rel="stylesheet">
table th:nth-of-type(3){
width:200px;
}</style>
<style rel="stylesheet">
table th:nth-of-type(4){
width:200px;
}</style>
<style rel="stylesheet">
table tr:hover {
background: #efefef; 
</style>
## 专线 NAT
腾讯云专线 NAT（Network Address Translation，网络地址转换）是混合云连接时将内网 IP 转换成其他 IP 的一种方式，包含 IP 转换和 IP 端口转换两种形式。使用专线 NAT 可以满足以下两种需求：
- 专线两端 IP 冲突。
- 需要屏蔽本端或对端的实际网段，不暴露给对方。

## IP 转换
IP 转换指将原 IP 转换为新的 IP 实现网络互访，分为**本端 IP 转换**和**对端 IP 转换**两种。IP 转换不区分源、目的访问，新 IP 既可以主动访问对端，也可以被对端主动访问。

### 本端 IP 转换
指私有网络内原 IP 映射为新 IP，并以新 IP 身份与用户 IDC 互访。
本端 IP 转换中，原 IP 指私有网络内 IP，映射 IP **不可以** 位于私有网络 CIDR 范围内。
<div style="text-align:center">
![](//mccdn.qcloud.com/static/img/89b1432e10828610a7287f1c5a150ec9/image.png)

</div>
<p style="text-align:center">图1：本端 IP 转换</p>
**示例：**
私有网络内 IP A： `192.168.0.3` 映射为IP B： `10.100.0.3` ，则 IP A 对专线对端的主动访问网络包源 IP 将自动修改为 `10.100.0.3`，所有专线对端返回的 `10.100.0.3` 的网络包将自动指向 IP A `192.168.0.3`。

**注意：**
- 您可以配置多条本端 IP 转换规则，并为每条本端 IP 转换规则配置网络 ACL，网络 ACL 支持源端口、目的 IP、目的端口配置。网络地址转换规则仅对符合 ACL 限制的网络请求生效。
- 本端 IP 转换不限制网络请求的方向，可以是私有网络主动访问专线对端，也支持专线对端主动访问私有网络。

### 对端 IP 转换
指用户 IDC 内原 IP 映射为新 IP，并以新 IP 身份与私有网络内 IP 互访。对端 IP 转换中，原 IP 指用户 IDC 的 IP，映射 IP ** 不可以** 位于私有网络 CIDR 范围内。
<div style="text-align:center">
![](//mccdn.qcloud.com/static/img/a3cbf4c2a9b0c3ac3664b2a20d1768fd/image.png)

</div>
<p style="text-align:center">图1：对端 IP 转换</p>
** 示例：**
专线对端 IP D： `10.0.0.3` 映射为 IP C： `172.16.0.3`，则 `10.0.0.3` 主动访问私有网络的网络包源 IP 将自动修改为 IP C `172.16.0.3`，所有私有网络返回给 IP C `172.16.0.3`的网络包将自动指向专线对端 IP D `10.0.0.3`。
<div style="text-align:center">
![](//mccdn.qcloud.com/static/img/cfdd11f873c0db9a0027088e955e46b7/image.png)

</div>
<p style="text-align:center">图3：对端 IP 示例</p>
** 注意：**
- 本端 IP 转换和对端 IP 转换映射 IP 不可以重叠，否则网络访问会出现异常。
- 当用户 IDC IP 与私有网络内 IP 冲突时，需要 **同时** 配置本端 IP 转换和对端 IP 转换，仅配置对端 IP 转换无法解决两边 IP 冲突的问题。
- 配置本端、对端 IP 转换后，专线网关仅会将转换后 IP 的路由下发至专线对端，因而本端、对端 IP 转换的原 IP 将无法 ```ping``` 通专线对端。但专线网关无法代替专业的网络防火墙，如果您需要高级的网络防护，请在私有网络内配置安全组和网络 ACL 策略，同时在您的 IDC 机房部署专业的物理网络防火墙设备。


## IP 端口转换
IP 端口转换指将源 IP 端口映射为新 IP 端口，并以新 IP 端口实现网络互访。IP 端口包含**本端源 IP 端口转换**、**本端目的 IP 端口转换**两种。
IP 端口转换强调方向性，源 IP 端口转换指主动外访，目的 IP 端口转换指被对端主动访问。

### 本端源 IP 端口转换
指私有网络内 IP 通过专线网关主动外访时，以指定 IP 池内的随机 IP 的随机端口访问专线对端的用户 IDC：
<div style="text-align:center">
![](//mccdn.qcloud.com/static/img/4c4131dadd5b8c120f1f16c31cca46df/image.png)

</div>
<p style="text-align:center">图4：本端源 IP 转换</p>
** 示例：**

私有网络 C 网段为 `172.16.0.0/16`, 通过专线连接第三方银行 A 和 B，其中银行 A 对端网段为`10.0.0.0/28`，要求对接网段为 `192.168.0.0/28`；银行 B 对端网段为 `10.1.0.0/28`，要求对接网段为 `192.168.1.0/28`。则可以按照下面配置两条本端源 IP 端口转换：

- 地址池 A： `192.168.0.1-192.168.0.15`
- ACL 规则 A：源 IP `172.16.0.0/16` 目的 IP `10.0.0.0/28` 目的端口 `ALL`
- 地址池B： `192.168.1.1-192.168.1.15`
- ACL 规则 B：源 IP `172.16.0.0/16` 目的 IP `10.1.0.0/28` 目的端口 `ALL`

则私有网络内主动访问 A、B 的网络请求会根据 ACL 规则 A、B 分别转换为对应的地址池的随机端口访问对应的专线通道。

** 注意：**
- 本端源 IP 端口转换仅支持私有网络端主动发起的网络访问请求，如果专线对端需要主动访问私有网络内的 IP 端口，需要额外配置本端目的 IP 端口转换配置。
- 本端源 IP 端口转换私有网络主动发起的网络请求为有状态连接，因而不用考虑网络回包的问题。
- 本端源 IP 端口转换支持配置 ACL 规则，只有符合 ACL 规则的网络出访问才会匹配地址池转发规则。通过为地址池配置不同的 ACL 规则，您可以灵活配置多个第三方接入时的网络地址转换规则。

### 本端目的 IP 端口转换
指专线对端主动访问私有网络的一种方法，通过将私有网络内指定 IP 的指定端口映射为新的 IP 和端口，专线对端则只可以通过访问映射后 IP 端口来与私有网络内指定 IP 端口通信，其他 IP 端口则不对专线对端暴露：
<div style="text-align:center">
![](//mccdn.qcloud.com/static/img/f5569c546acb766908cf99d7c7e15798/image.png)

</div>
<p style="text-align:center">图5：本端目的 IP 转换</p>
** 示例：**
私有网络 C 的网段为 `172.16.0.0/16`，只希望开放若干端口给专线对端主动访问，则可以按照下面方案进行配置：
- 映射 A：原 IP 端口 `172.16.0.1:80` 映射 IP 端口 `10.0.0.1:80`
- 映射 B：原 IP 端口 `172.16.0.0:8080` 映射 IP 端口 `10.0.0.1:8080`

则专线对端可以主动访问 `10.0.0.1:80`、`10.0.0.1:8080`端口，实现对私有网络内 `172.16.0.1:80`、`172.16.0.0:8080`两个端口的主动访问。

** 注意：**

- 配置本端源、目的 IP 端口转换后，专线网关仅会将转换后 IP 端口路由下发至专线对端，因而会本端 IP 端口将无法主动发起请求或被动接受请求。但专线网关无法代替专业的网络防火墙，如果您需要高级的网络防护，请在私有网络内配置安全组和网络 ACL 策略，同时在您的 IDC 机房部署专业的物理网络防火墙设备。
- 当同时配置 IP 转换和 IP 端口转换时，优先匹配 IP 转换，IP 转换如果没有匹配中才会继续匹配IP 端口转换。
- 本端目的 IP 端口转换不支持 ACL 规则适配，因而 IP 端口转换规则将对专线网关所连接的所有专线通道生效。
- 本端目的 IP 端口转换仅对专线通道对端主动访问私有网络生效，如果私有网络需要主动访问专线对端，可以配置本端源 IP 端口转换。
- 本端目的 IP 端口转换的网络请求为有状态连接，无需考虑网络回包的问题。
- 本端目的 IP 端口转换中，您可以填入单个 IP 或一组连续的 IP 作为目的地址转换 IP 池。
- 本端目的 IP 端口转换中，您指定的原 IP 端口为私有网络内的 IP，映射 IP 端口为对端主动访问的转换后的 IP 端口，您在做端口映射规则时还需指定网络协议是 TCP 还是 UDP。



## 使用约束
关于**专线接入**您需要了解：
- 新建专线网关时，IP 转换和IP 端口转换内容默认为空，此时 IP 转换与 IP 端口转换均不生效。
- 专线通道支持 BPG 路由和静态路由两种路由方式。

关于**IP 转换**您需要了解：
- IP 地址池不可以在专线网关所在私有网络的 CIDR 范围内。
- 多个 IP 地址池的 ACL 规则不可以重叠，否则会导致网络地址转换冲突。
- 多个 IP 地址池之间 IP 不可以重叠。
- IP 地址池仅支持单 IP 或连续 IP，且连续 IP 的 /24 网段需保持一致，即支持 `192.168.0.1~192.168.0.6`，不支持 `192.168.0.1~192.168.1.2`。
- 地址池不支持广播地址（255.255.255.255）、D 类地址（224.0.0.0~239.255.255.255）、E 类地址（240.0.0.0~255.255.255.254）。
- 本端源 IP 端口转换最大支持 100 个 IP 地址池，每个地址池支持最大 20 条 ACL 规则（如果有需求可以发起工单提高配额）
- 当您需要从 IP 转换切换为 IP 端口转换时，请清空原 IP 转换规则，刷新页面后即可编辑 IP 端口转换规则。

关于**IP 端口转换**您需要了解：
- 原 IP 必须在专线网关所在私有网络 CIDR 范围之内
- 原 IP 端口唯一，即私有网络内同一 IP 端口只能唯一映射为一个 IP 端口
- 映射 IP 端口不可以在私有网络 CIDR 范围之内
- 映射 IP 端口不可以重复，即不存在一个 IP 端口映射多个私有网络 IP 端口
- 原 IP 和映射 IP 不支持广播地址（255.255.255.255）、D 类地址（224.0.0.0~239.255.255.255）、E 类地址（240.0.0.0~255.255.255.254）
- 本端目的 IP 端口转换最大支持 100 个 IP 端口映射（如果有需求可以发起工单提高配额）
- 同时配置 IP 转换和 IP 端口转换时，如果同时命中优先匹配 IP 转换。

| 资源 | 限制 | 说明 |
|------|:-----:|:-----:|
| 物理专线 / 用户 | 10 个 | |
| 专线通道 / 物理专线 | 10 个 | |
| 专线网关 （支持 NAT）/ 私有网络 | 1 个 | |
| 专线网关 （不支持 NAT）/ 私有网络 | 1 个 | |
| 本端 IP 转换 / 专线网关 | 100 条 | 可申请调高配额|
| 对端 IP 转换 / 专线网关 | 100 条 | 可申请调高配额|
| 本端源 IP 端口转换 IP 数 / 专线网关 | 20 个 | 可申请调高配额|
| 本端目的 IP 端口转换 / 专线网关 | 100 条 | 可申请调高配额|