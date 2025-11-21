---
title: Cloudflare DNS
published: 2025-11-20
pinned: false
description: Cloudflare DNS 解析自动化脚本。
image: "images/cloudflare.webp"
tags: [Cloudflare, DNS]
category: Cloudflare
licenseName: "Unlicensed"
author: 御剑乘风
sourceLink: "https://github.com/h-lyf/Mizuki/blob/master/src/content/posts/cloudflare-dns.md"
draft: false
date: 2025-11-20
pubDate: 2025-11-20
---

Cloudflare 是一家全球领先的“连接云”（Connectivity Cloud）公司，成立于2009年，总部位于美国旧金山。截至2025年，Cloudflare 的网络已覆盖全球超过330个城市，代理了互联网上约20%的网页流量，为数百万网站、应用和API提供性能优化、安全防护和开发者工具。Cloudflare 的使命是“构建一个更好的互联网”，它通过一个统一的平台，帮助用户解决现代互联网面临的性能、安全、隐私和弹性问题。

# 介绍
## Cloudflare 的主要功能和优势
Cloudflare 提供了一系列集成服务，即使免费计划也能享受到企业级防护。核心功能包括：
- 全球CDN（内容分发网络）：将静态内容（如图片、JS、CSS）缓存到全球边缘服务器，显著加速网站加载速度，减少延迟。
- DDoS 防护：无计量、无限制的DDoS攻击缓解能力（容量是史上最大攻击的数十倍），无需改代码即可防护。
- Zero Trust 安全：包括SASE/SSE架构、WAF（Web应用防火墙）、Bot管理、API防护等，防止入侵、数据泄露和恶意机器人。
- DNS 服务：权威DNS托管、公用解析器（1.1.1.1），速度极快且注重隐私。
- 开发者工具：Workers（Serverless计算）、R2（对象存储）、Pages（静态站点托管）等，支持全栈开发。
- 其他：免费SSL证书、Argo智能路由、WARP（类似VPN的隐私工具）、图像优化（Polish）、视频流（Stream）等。

Cloudflare 的最大优势在于集成性强、免费门槛低、全球Anycast网络（查询自动路由到最近节点）和一键启用。即使是个人博客，也能免费获得原本只有大企业才有的防护和加速能力。

## Cloudflare DNS 详解
Cloudflare DNS 是 Cloudflare 最核心的服务之一，分成两个主要部分：
1. **公用DNS解析器（Public DNS Resolver）——1.1.1.1**
    - 这是 Cloudflare 在2018年推出的免费公共DNS服务，地址简单易记：1.1.1.1（IPv4）和 2606:4700:4700::1111（IPv6）。
    - 速度最快：独立测试机构 DNSPerf 显示，截至2025年，1.1.1.1 仍是全球响应时间最快的公共DNS（欧洲平均约6-11ms）。
    - 隐私优先：不记录用户查询IP、不出售数据，查询日志仅保留24小时用于调试，随后永久删除。支持 DNS over HTTPS (DoH) 和 DNS over TLS (DoT)，防止ISP窥探浏览记录。
    - 安全特性：自动DNSSEC验证、恶意域名过滤（可选择1.1.1.2过滤成人内容，1.1.1.3过滤恶意+成人内容）、配合WARP App可实现全设备加密流量。
    - 适用于任何人：手机、电脑、路由器一键切换即可使用，比默认ISP DNS更快、更私密。
2. **权威DNS托管（Authoritative DNS）**
    - 当你把域名NS指向 Cloudflare（如 ns1.cloudflare.com）后，Cloudflare 成为你域名的权威DNS提供商。
    - 核心优势：
        - 极致速度：全球330+城市Anycast网络，平均查询时间11ms，记录更新几乎瞬间生效（无传统DNS的漫长传播）。
        - 高可用：100% uptime SLA（付费计划），内置无限DDoS防护。
        - 一键DNSSEC：防止DNS劫持和中间人攻击。
        - 高级特性：CNAME Flattening（根域名支持CNAME）、扁平化解析、地理路由、负载均衡记录（付费）、DNS防火墙等。
        - 完全免费：即使免费计划，也支持无限DNS查询、无限记录数（部分高级功能需付费）。
    - 与其他服务的无缝集成：开启“橙色云”代理后，流量经过Cloudflare CDN + WAF，获得加速+安全双重加持。

# 过程
## 前期准备
1. 获取域的区域ID

![Snipaste_2024-09-20_08-20-06.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/78d9c530f363110fb9aceb29df91ff466e71f6e2.png)

2. 获取Cloudflare API密钥

![Snipaste_2024-09-20_08-24-46.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/23afbf2c5ce92ff31716269fb663a4a2dd65e4e5.png)
![Snipaste_2024-09-20_08-27-38.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/fc54423e7d613fc5a03752c324f027516e474e1c.png)
![Snipaste_2024-09-20_08-28-30.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/a9f9aa2aca085efd670f677fb16ab8cf5645d767.png)
![Snipaste_2024-09-20_08-30-16.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/65de192291bc078792e65fbe3d69998f014714c6.png)
![Snipaste_2024-09-20_08-31-15.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/d250a6e11566d6db246c1f1bd78f1d6466f6e3be.png)
![Snipaste_2024-09-20_08-32-17.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/e4bd52df011d9749c299ea91988c11a87879c32c.png)

3. 安装软件

```bash
sudo apt-get install curl jq -y
```

## 配置脚本

```bash
#!/bin/bash

# Cloudflare管理域名解析记录的脚本

# 获取Cloudflare API密钥
API_TOKEN="XXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXXX"

# 获取要管理的域的区域ID
ZONE_ID="xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"

# 要管理的域名
DOMAIN_NAME="xxx.com"

# 要添加或更新的DNS记录类型
RECORD_TYPE="AAAA"

# 指定要更新的IP地址
IP_ADDRESS=`ip -6 addr | grep "inet6.*global" | grep -v "deprecated" | awk '{print $2}' | awk -F"/" '{print $1}' | sed -n '1,1p'`
echo "获取 IPv6 : $IP_ADDRESS"

# 设置DNS解析记录文件
DNS_RECORD_JSON="$HOME/cfdns/dns_record.json"
UPDATE_JSON="$HOME/cfdns/update_record.json"
ADD_JSON="$HOME/cfdns/add_record.json"

# 获取现有的DNS记录，找到匹配的记录
curl -X GET "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records?type=$RECORD_TYPE&name=$DOMAIN_NAME" \
     -H "Authorization: Bearer $API_TOKEN" \
     -H "Content-Type: application/json" \
     -s -o "$DNS_RECORD_JSON"

# 提取现有的记录ID
RECORD_ID=$(jq -r '.result[0].id' "$DNS_RECORD_JSON")

if [[ -n "$RECORD_ID" && "$RECORD_ID" != "null" ]]; then
  echo "找到现有的DNS记录，正在更新..."

  # 使用现有的记录ID直接更新记录
  curl -X PUT "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records/$RECORD_ID" \
       -H "Authorization: Bearer $API_TOKEN" \
       -H "Content-Type: application/json" \
       -d '{
             "type": "'"$RECORD_TYPE"'",
             "name": "'"$DOMAIN_NAME"'",
             "content": "'"$IP_ADDRESS"'",
             "ttl": 0,
             "proxied": false
           }' \
       -s -o "$UPDATE_JSON"

  # 检查更新记录的结果
  UPDATE_SUCCESS=$(jq -r '.success' "$UPDATE_JSON")

  if [ "$UPDATE_SUCCESS" = "true" ]; then
    echo "IP地址 $IP_ADDRESS 的DNS解析记录更新成功。"
  else
    echo "IP地址 $IP_ADDRESS 的DNS解析记录更新失败。"
    jq -r '.errors[] | .message' "$UPDATE_JSON"
  fi

else
  echo "未找到匹配的DNS记录，正在添加新记录..."

  # 添加新的DNS记录
  curl -X POST "https://api.cloudflare.com/client/v4/zones/$ZONE_ID/dns_records" \
       -H "Authorization: Bearer $API_TOKEN" \
       -H "Content-Type: application/json" \
       -d '{
             "type": "'"$RECORD_TYPE"'",
             "name": "'"$DOMAIN_NAME"'",
             "content": "'"$IP_ADDRESS"'",
             "ttl": 0,
             "proxied": false
           }' \
       -s -o "$ADD_JSON"

  # 检查添加记录的结果
  ADD_SUCCESS=$(jq -r '.success' "$ADD_JSON")

  if [ "$ADD_SUCCESS" = "true" ]; then
    echo "IP地址 $IP_ADDRESS 的DNS解析记录添加成功。"
  else
    echo "IP地址 $IP_ADDRESS 的DNS解析记录添加失败。"
    jq -r '.errors[] | .message' "$ADD_JSON"
  fi
fi
```

> RECORD_TYPE：DNS记录类型，IPv4为A，IPv6为AAAA;  
