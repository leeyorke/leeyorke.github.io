# nanobot配置obsidian mcp server(V2)


## 方案优化点

V1方案: [nanobot配置obsidian mcp server](../2026-06-23/) 不太合理。
因为通过执行命令进行宿主机监听 `172.17.121.1`，在加上mcp server本身监听，实际上是监听了两个地址，一个 windows
本地回环地址，一个windows的虚拟网关地址。那我直接监听后者不就行了吗，好在 obsidian mcp server 可以修改 bindhost，
我直接改成后者，obsidian一启动就可以暴漏 27123 端口给 wsl。省得我再去敲一次命令。


## 知识点

1. WSL2 的 localhost 不是 Windows 的 localhost。WSL2 可以访问 Windows 的服务，但不是通过 127.0.0.1，而是通过 Windows 的虚拟网关 IP（172.17..）。这个 IP 不是 Windows 的真实 IP，是为了给 WSL 访问windows宿主机的服务用的。即：在 windows 本机，
它的 IP 是 192.168..，在 WSL2 中，它的 IP 就是 172.17..，在 WSL2 中执行以下命令就能获取 Windows 的虚拟网关 IP
    ```
    # 方法1（推荐，最常用）
    cat /etc/resolv.conf | grep nameserver | awk '{print $2}'

    # 方法2
    ip route show | grep default | awk '{print $3}'
    ```
    如果要获取 WSL 的真实 IP，可以在 windows 里使用 `wsl hostname -I`, 在 WSL2 里可以使用 `ip a` 或者其他命令，这两个结果是一样的。
2. Clash Party的允许局域网连接。先说作用，这个是为了在同一局域网内，你可以通过该电脑代理上网。作用就是把7897
端口暴露在0.0.0.0/0，即监听局域网内所有地址，这些地址背后的设备如果设置了 `http://{该电脑的IP}:7897` 就都可以访问该电脑的7897端口。那么在 WSL2 中将 172.17.. 赋给环境变量 `http_proxy` 或 `https_proxy`, 那么命令行的所有工具比如 `curl` 的流量就会都由 Windows 宿主机的7897端口代理，就可以访问外网了。
3. 在 WSL2 中 运行的 nanobot，要访问 Windows 的7897、27123端口, 就必须配置代理为 `https://172.17.*.*:27123`。之前copilot告诉我 WSL2 可以直接通过 localhost 访问 windows 的 localhost, 害得我改了好多代码和配置，**这两个其实不一样的！**。
4. `curl` 命令可以测试网络连通性，可以使用 `curl -vk ttps://172.17.*.*:27123`，v=verbos(显示详细调试信息), k=insecure(忽略证书错误)。

## 最后成果

1. obsidian mcp 可以通过自身设置随意更改bindhost（自身具有，缺点是但是只能改一个，如果后续要增加，可以通过命令监听，这个问题不大）。
2. nanobot 源码[变动](https://github.com/leeyorke/nanobot/commit/dd13ef6a7dccf4ecb259d0b0454a2a0befb74cee)了下，使得不管是obsidian打开/关闭, 都不影响 MCP 连接, 如果 MCP 服务存活，就连接，否则就抛出警告跳过连接，不影响 gateway 得运行。代码未改之前，在 obsidian 关闭情况下，会抛出异常直接退出进程，除非你删掉MCP配置。
