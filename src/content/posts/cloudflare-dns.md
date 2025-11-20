---
title: Cloudflare DNS
published: 2025-11-20
pinned: false
description: A simple example of a Markdown blog post.
image: "src/assets/images/3200x3200.webp"
tags: [Cloudflare, DNS]
category: Cloudflare
licenseName: "Unlicensed"
author: 御剑乘风
sourceLink: "https://github.com/h-lyf/Mizuki/blob/master/src/content/posts/cloudflare-dns.md"
draft: false
date: 2025-11-20
pubDate: 2025-11-20
---

# 前期准备

## 获取域的区域ID

![Snipaste_2024-09-20_08-20-06.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/78d9c530f363110fb9aceb29df91ff466e71f6e2.png)

## 获取Cloudflare API密钥

![Snipaste_2024-09-20_08-24-46.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/23afbf2c5ce92ff31716269fb663a4a2dd65e4e5.png)
![Snipaste_2024-09-20_08-27-38.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/fc54423e7d613fc5a03752c324f027516e474e1c.png)
![Snipaste_2024-09-20_08-28-30.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/a9f9aa2aca085efd670f677fb16ab8cf5645d767.png)
![Snipaste_2024-09-20_08-30-16.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/65de192291bc078792e65fbe3d69998f014714c6.png)
![Snipaste_2024-09-20_08-31-15.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/d250a6e11566d6db246c1f1bd78f1d6466f6e3be.png)
![Snipaste_2024-09-20_08-32-17.png](https://pub-d169c15f75c9477fb4d500a8cf7ff73b.r2.dev/images/e4bd52df011d9749c299ea91988c11a87879c32c.png)

## 安装软件

```bash
sudo apt-get install curl jq -y
```

# Cloudflare DNS 解析脚本

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
IP_ADDRESS：需要解析的IP；  
RECORD_TYPE：DNS记录类型，IPv4为A，IPv6为AAAA;  
