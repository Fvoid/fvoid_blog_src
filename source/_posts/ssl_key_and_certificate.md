# SSL Key and Certificate Background Knowledge

## Openssl 生成方式

1. 随机生成ca.key
2. 基于ca.key 生成 ca.crt
3. 随机生成server.key
4. 创建生成 Certificate Signing Request (CSR) 的conf 文件， `csr.conf`
5. 基于server.key 和 csr.conf 生成server.csr
6. 基于csr.conf 使用 ca.crt 和 ca.key 对 server.csr 进行签名生成 server.crt

