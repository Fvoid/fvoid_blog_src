---
title: microk8s scripts analysis
date: 2021-01-10 01:16:10
tags: microk8s
---

# 1. cluster

## 1.1 add_token.py

- 判断是否传入token
  
  - 若否，随机生成16 byte的hex

- 写入`credentials/cluster_token.txt`

# 2. wrappers
