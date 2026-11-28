# 初次搭建梯子记录


## 01 购买 VPS

购买 [raksmart](https://cn.raksmart.com/) 的最便宜的一档2美元一月，支持支付宝付款。
选择公钥登录。然后配置安全组，记得放行443端口。
重置一下密码。


## 02 安装 xray-core

之前设置后会给你下载一个公钥文件 `id_rsa.pem`，保存到 `~/.ssh/` 下就行，然后配置 ssh 方便连到 VPS 上。
`~/.ssh/config` 配置：

```yaml
# ========================
# raksmart
# ========================
Host rks
    HostName 173.148.27.45
    Port 60321
    User root
    IdentityFile ~/.ssh/id_rsa.pem
```

接着打开 ternimal 输入 `ssh rks` 就能直接登上去。

进入 VPS 的终端，安装 `xray`, 一键安装脚本：
```bash
curl -fsSL https://raw.githubusercontent.com/leeyorke/xray-onexray-reality/refs/heads/main/install.sh | bash
```
安装完成后命令行会打印出一个二维码和 `vless://` 链接。直接用 [v2rayNG](https://github.com/2dust/v2rayNG) 扫描就能用。
如果要在 flclash、clash party 里用, 就需要转换一下订阅，实测直接导入是不能用的，原因是 xray 最新版本升级了 reality 身份验证机制。flclash、clash party 这些客户端可能没有及时支持。

## 03 测试连通性

浏览器打开 [tcp.ping.pe](https://tcp.ping.pe/), 输入 ip 及端口号，如 `173.148.27.45:8443`, 如全部为绿，则证明你的 VPS 无问题。

## FAQ

**connect error: REALITY authentication failed 如何解决?**

可能是 Public Key（公钥）、Short ID、SNI（Server Name）、UUID 以及 Flow（如 xtls-rprx-vision）与服务端不完全一致。
也可能是客户端与服务端版本不兼容导致认证失败：服务端升级 Xray-core 后（例如从 v26.6.27 升级到部分新版本），可能与客户端或 Mihomo 内核出现兼容问题，所以需要 Xray-core 降级。

命令：

```bash
bash -c "$(curl -L https://raw.githubusercontent.com/XTLS/Xray-install/main/install-release.sh)" @ install --version 26.6.27
```

重启 `systemctl restart xray`, 查看版本 `xray version`。

**REALITY localAddr: 173.148.27.45:8443 is using X25519MLKEM768 for TLS' communication: false**

is using X25519MLKEM768 for TLS' communication: false 意味着 Xray 服务端在 REALITY 配置中强制开启了后量子密钥交换（X25519MLKEM768），或者你的节点本身已成功连通到了 Xray 服务端（端口与防火墙没问题），但因为客户端与服务端在 后量子加密（ML-KEM-768） 协商上不一致导致握手被拒。通过 vless:// 链接导入时，FlClash 和 Clash Party 默认往往不会自动添加 support-x25519mlkem768: true，导致 Mihomo 默认以传统 X25519 发起握手，所以 clash 配置改为以下:

```yaml
proxies:
  - name: "你的节点"
    type: vless
    server: 173.148.27.45
    port: 8443
    uuid: 你的UUID
    network: tcp
    tls: true
    udp: true
    flow: xtls-rprx-vision
    servername: 你的SNI域名
    client-fingerprint: chrome
    reality-opts:
      public-key: 你的公钥
      short-id: "你的ShortID"
      support-x25519mlkem768: true
```
完整最简配置模板：[clash-template-from-vless.yaml](https://raw.githubusercontent.com/leeyorke/rules/refs/heads/main/template/clash-template-from-vless.yaml)

**如何加强伪装性**

1. 改走443端口，高位端口容易被封。
2. 屏蔽回国流量。具体操作：
  ```bash
  python3 -c 'import json; p="/usr/local/etc/xray/config.json"; d=json.load(open(p)); d.setdefault("outbounds",[]); (any(o.get("tag")=="block" for o in d["outbounds"]) or d["outbounds"].append({"protocol":"blackhole","tag":"block"})); r=d.setdefault("routing",{}); r["domainStrategy"]="IPIfNonMatch"; rules=r.setdefault("rules",[]); block_rules=[{"type":"field","ip":["geoip:cn"],"outboundTag":"block"},{"type":"field","domain":["geosite:cn"],"outboundTag":"block"}]; r["rules"]=[x for x in block_rules if x not in rules]+rules; json.dump(d,open(p,"w"),indent=2,ensure_ascii=False)'

  python3 -m json.tool /usr/local/etc/xray/config.json > /dev/null
  ```
  无报错输出说明 JSON 格式完整有效。xray -test -c /usr/local/etc/xray/config.json 检查配置合法性，再执行 systemctl restart xray。验证方式： xray -test 提示 Configuration OK，且 systemctl status xray 显示 active (running) 即生效。

**高峰期速度慢怎么优化**

把 TCP 拥塞控制算法改成 bbr。
```bash
echo "net.ipv4.tcp_congestion_control=bbr" >> /etc/sysctl.conf
echo "net.core.default_qdisc=fq" >> /etc/sysctl.conf
sysctl -p
sysctl net.ipv4.tcp_congestion_control
```
如果还是 cubic，说明内核不支持（BBR 需 Linux 4.9+），运行 uname -r 确认。

BBR 和 cubic 的优势与劣势

BBR（基于带宽/时延测量）

优势：抗随机丢包强（丢包率 1%~5% 基本不掉速），高延迟/高带宽链路吞吐高
劣势：早期版本对低延迟局域网（RTT<20ms）可能有更高重传；抢占带宽较激进，与 CUBIC 混跑时可能挤占对方；需内核 4.9+

CUBIC（基于丢包）

优势：行为温和、公平性好、与旧系统兼容，局域网表现稳定；几乎所有系统默认支持
劣势：丢包即降速，对随机丢包敏感；高 BDP 链路（如跨国、卫星）吞吐量明显低于 BBR
一句话：链路好、丢包少时两者差不多；链路差（跨洋、无线网络）选 BBR。
