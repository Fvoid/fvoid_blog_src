---
title: microk8s resource analysis
date: 2021-01-09 09:05:53
tags: microk8s
---

# 1. wrappers

## microk8s-start.wrapper 分析

- 判断`${SNAP_DATA}/var/lock/clustered.lock`是否存在

- 若存在，提示使用`snap start microk8s`

- 使用`snap start microk8s --enable`

- --enable: 当机子重启时，snap服务会重新启动microk8s

- 判断是否存在`${SNAP_DATA}/var/lock/stopped.lock`

- 若存在，移除该文件。则说明api server在运行

- 执行`wait_for_node`

- 输出`kubectl get node`的结果

## microk8s-add-node.wrapper

- [shell set review](./echo+shell-special-variables.md)

- `set -eu`: `-e` 使得脚本只要发生错误，就终止执行; `-u` 执行脚本的时候，如果遇到不存在的变量，Bash 名默认忽略它。

- 若当前存在`${SNAP_DATA}/var/lock/clustered.lock`，则不执行并提示在master执行

- 获取ca.crt的subject
  
  + 若subject为127.0.0.1，报错

- 若不存在文件`$SNAP_DATA/credentials/cluster-tokens.txt`,创建一个

- 若存在microk8s group，将`$SNAP_DATA/credentials/cluster-tokens.txt`的group设为microk8s
  
  + 对于& 1 更准确的说应该是文件描述符 1,而1标识标准输出，stdout。
  + 对于2 ，表示标准错误，stderr。
  + 2>&1 的意思就是将标准错误重定向到标准输出。这里标准输出已经重定向到了 /dev/null。那么标准错误也会输出到/dev/null

- 生成token--跑[add_token.py](./microk8s-scripts-analysis.md)

- 保存当前IFS（Internal Field Separator）

```shell
$ echo $IFS

$ echo "$IFS" | od -b
0000000 040 011 012 012
0000004

"040"是空格，"011"是Tab，"012"是换行符"\n" 
```

- 获取token
- 还原IFS
- 若`${SNAP_DATA}"/args/cluster-agent`中存在"25000", 设port为cluster-agent中的第二个字段
- 输出join node的结果

## microk8s-join-node.wrapper

# 2. action/

## 2.1 common/utils.sh

**get_opt_in_config()**

- 在`/var/snap/microk8s/current/args`路径下获取

----

**restart_service()**

- 重启本机daemon-XXXX service

- 查看是否存在文件`${SNAP_DATA}/var/lock/ha-cluster`

- 若存在，执行`distributed_op.py`

- 查看是否存在`${SNAP_DATA}/credentials/callback-tokens.txt`

- 若存在，执行`distributed_op.py`

- （callback token为join另一node时所带来的）

----

**get_defualt_ip()**

```bash
get_default_ip() {

# Get the IP of the default interface

local DEFAULT_INTERFACE="$($SNAP/bin/netstat -rn | $SNAP/bin/grep '^0.0.0.0' | $SNAP/usr/bin/gawk '{print $NF}' | head -1)"

# Get default interface's ip

local IP_ADDR="$($SNAP/sbin/ip -o -4 addr list "$DEFAULT_INTERFACE" | $SNAP/usr/bin/gawk '{print $4}' | $SNAP/usr/bin/cut -d/ -f1 | head -1)"

if [[ -z "$IP_ADDR" ]]> []> []

then

echo "none"

else

echo "${IP_ADDR}"

fi

}
```

`netstat -rn`会返回一系列数据组成的表格

```sh
Kernel IP routing table

Destination Gateway Genmask Flags MSS Window irtt Iface

0.0.0.0 192.168.1.2 0.0.0.0 UG 0 0 0 ens33

169.254.0.0 0.0.0.0 255.255.0.0 U 0 0 0 ens33

192.168.1.0 0.0.0.0 255.255.255.0 U 0 0 0 ens33
```

`gawk '{print $NF}' `会返回表格的最后一列。

这是`$nf `的详细描述

```
NF is a predefined variable whose value is the number of fields in the current record. awk automatically updates the value of NF each time it reads a record. No matter how many fields there are, the last field in a record can be represented by $NF . So, $NF is the same as $7 , which is ' example.
```

----

**produce_certs()**

[SSL Key and Certificate Background](./ssl_key_and_certificate.md)

- 基于`/snap/microk8s/current/etc/ssl/openssl.cnf`来生成key, 存在`$SNAP_DATA/certs/`

- `serviceaccounnt.key`

- `ca.key`

- `server.key`:

- `front-proxy-ca.key`：用于apiserver的aggredated layer，（aggregated apiserver）

- `front-proxy-client.key`：用于apiserver的aggredated layer，（aggregated apiserver）

- 从`csr.conf.rendered` 更新数据到 `csr.conf`

- 基于`ca.key` 生成10年的`ca.crt`

- 基于`front-proxy-ca.key` 生成**10**年的`front-proxy-ca.crt`

- 执行**render_csr_conf()**

- 获取csr.conf,从csr.conf.template中添加额外的ip，从$(get_ips)中获取。

- 执行**gen_server_cert()**

- openssl 使用OPENSSL_CONF 环境变量来获取cnf文件。

- 基于`server.key`, `csr.conf`生成`server.csr`

- 使用`server.csr` 和 `ca.crt`, `ca.key`的签名生成**1年的**`server.crt` **###**

- 执行**gen_proxy_client_cert()**

- 同上生成**1年**的`front-proxy-client.crt` **###**

----

**drain_node()**

- safely evict all of your pods from a node before you perform maintenance on the node

**uncordon_node()**

- mark node as schedulable

----

**init_cluster()**

- 添加本机IP到`${SNAP_DATA}/var/kubernetes/backend/init.yaml`

- 将csr-dqlite.conf中的DNS填为本机hostname，IP填为127.0.0.1

- 基于csr-dqlite.conf生成cluster.key和**10年**的cluster.crt

---

**update_configs()**

- 从`know_tokens.csv`提取token到

- admin token -> `client.config`

- controller token-> `controller.config`

- scheduler token -> `scheduler.config`

- node token -> `kubelet.config`

- kube-proxy token -> `proxy.config`

- 执行stop

- 执行start

## 2.2Reference

1. [snap environment variables](

2. https://snapcraft.io/docs/environment-variables)