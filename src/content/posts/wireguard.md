---
title: WireGuard
published: 2025-11-20
pinned: false
description: A simple example of a Markdown blog post.
image: "images/wireguard.png"
tags: [WireGuard]
category: WireGuard
licenseName: "Unlicensed"
author: 御剑乘风
sourceLink: "https://github.com/h-lyf/Mizuki/blob/master/src/content/posts/wireguard.md"
draft: false
date: 2025-11-20
pubDate: 2025-11-20
---

WireGuard® 是一种极其简单但快速且现代的 VPN，它利用了最先进的[加密技术](https://www.wireguard.com/protocol/)。它的目标是比 IPsec [更快](https://www.wireguard.com/performance/)、[更简单](https://www.wireguard.com/quickstart/)、更精简、更有用，同时避免了巨大的麻烦。它旨在比 OpenVPN 性能好得多。WireGuard 旨在作为通用 VPN，适用于嵌入式接口和超级计算机，适用于许多不同的情况。最初为 Linux 内核发布，现在它是跨平台的（Windows、macOS、BSD、iOS、Android），并且可以广泛部署。它目前正处于积极开发中，但即使如此，它可能被认为是行业内最安全、最容易使用和最简单的 VPN 解决方案。

# 配置
## 服务器端
1. 安装 WireGuard  
    ```bash  
    apt update && apt install wireguard -y
    ```  
2. 配置服务器  
    - 生成密钥对  
    ```bash
    wg genkey | tee private.key | wg pubkey > public.key
    ```   
    - 创建配置文件
    ```bash  
    vim /etc/wireguard/wg0.conf
    ```
    ```bash  
    [Interface]
    PrivateKey = <服务器私钥>
    Address = 10.0.1.1/24
    ListenPort = 51820

    PostUp = sysctl -w net.ipv4.ip_forward=1;
    PostDown = sysctl -w net.ipv4.ip_forward=0;

    [Peer]
    PublicKey = <客户端1的公钥>
    AllowedIPs = 10.0.1.2/32

    [Peer]
    PublicKey = <客户端2的公钥>
    AllowedIPs = 10.0.1.3/32
    ```  
    > PostUp = sysctl -w net.ipv4.ip_forward=1;     # 启动 WireGuard 接口时开启 IP 转发  
    > PostDown = sysctl -w net.ipv4.ip_forward=0;   # 关闭 WireGuard 接口时停止 IP 转发  
3. 启动 WireGuard  
    ```bash  
    systemctl start wg-quick@wg0
    ```  
    ```bash  
    systemctl enable wg-quick@wg0
    ```  
4. 检查状态  
    ```bash
    wg show
    ```  
5. 防火墙配置
    - ufw  
    ```bash
    ufw allow 51820/udp
    ufw allow in on wg0 to 10.0.1.0/24
    ufw allow out on wg0 to 10.0.1.0/24
    ```  
    - iptables
    ```bash
    iptables -A INPUT -p udp --dport 51820 -j ACCEPT
    iptables -A INPUT -i wg0 -s 10.0.1.0/24 -j ACCEPT
    iptables -A OUTPUT -o wg0 -d 10.0.1.0/24 -j ACCEPT
    ```  
## 客户端  
1. 客户端配置  
    ```bash  
    [Interface]
    PrivateKey = <客户端的私钥>
    Address = 10.0.1.2/24

    [Peer]
    PublicKey = <服务器的公钥>
    Endpoint = <服务器公网IP>:51820
    AllowedIPs = 10.0.1.0/24
    PersistentKeepalive = 10
    ```  
    > PersistentKeepalive = 10      # 客户端会向服务器发送一个 keep-alive 数据包时间间隔  
2. 验证  
    - 服务端连接状态  
    ```bash
    sudo wg show
    ```  
    - 客户端互通  
    * 客户端 1 Ping 客户端 2  
    ```bash
    ping 10.0.1.3
    ```  
    * 客户端 2 Ping 客户端 1  
    ```bash
    ping 10.0.1.2
    ```  
# 进阶配置  
1. 公网访问内网服务  
    - **服务端**  
    ```bash
    iptables -t nat -A PREROUTING -p tcp -d <服务器公网IP> --dport <服务器端口> -j DNAT --to-destination <客户端IP>:<客户端端口>
    ```  
    **如果服务器是弹性公网 IP,服务器公网 IP 可能需要改成服务商的私网 IP,通过查看 IP**  
    ```bash
    ip addr
    ```  
    **IP 和服务器公网 IP 一致用公网 IP,否则使用刚查到的私网 IP**  
    * 删除规则  
    ```bash
    iptables -t nat -D PREROUTING -p tcp -d <服务器公网IP> --dport <服务器端口> -j DNAT --to-destination <客户端IP>:<客户端端口>
    ```  
2. 客户端通过服务端访问外网  
    - **服务端**  
    ```bash
    iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    ```  
    - **客户端**
    ```bash
    [Interface]
    PrivateKey = <客户端的私钥>
    Address = 10.0.1.2/24

    [Peer]
    PublicKey = <服务器的公钥>
    Endpoint = <服务器公网IP>:51820
    AllowedIPs = 0.0.0.0/0
    PersistentKeepalive = 10
    ```  
    - 删除规则  
    ```bash
    iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE
    ```  
    **AllowedIPs = 0.0.0.0/0 -- 通过 WireGuard 隧道发送和接收任何目的地的 IP 流量,根据情况进行修改**  
3. 保存规则  
    - 通过 iptables-persistent 保存  
    ```bash
    apt install iptables-persistent -y
    netfilter-persistent save
    ```  
    - 直接写入服务端配置
    ```bash  
    [Interface]
    PrivateKey = <服务器私钥>
    Address = 10.0.1.1/24
    ListenPort = 51820

    PostUp = sysctl -w net.ipv4.ip_forward=1;iptables -t nat -A PREROUTING -p tcp -d <服务器公网IP> --dport <服务器端口> -j DNAT --to-destination <客户端IP>:<客户端端口>;iptables -t nat -A POSTROUTING -o eth0 -j MASQUERADE
    PostDown = sysctl -w net.ipv4.ip_forward=0;iptables -t nat -D PREROUTING -p tcp -d <服务器公网IP> --dport <服务器端口> -j DNAT --to-destination <客户端IP>:<客户端端口>; iptables -t nat -D POSTROUTING -o eth0 -j MASQUERADE

    [Peer]
    PublicKey = <客户端1的公钥>
    AllowedIPs = 10.0.1.2/32

    [Peer]
    PublicKey = <客户端2的公钥>
    AllowedIPs = 10.0.1.3/32
    ```  