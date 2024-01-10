# 用 clusterissuer 自动申请 SSL 证书

- clusterissuer 不支持 Aliyun DNS，因此首先参考 [TrueNAS使用ACME自动添加续期泛域名证书](https://www.truenasscale.com/2021/12/10/126.html) 把域名迁到 cloudflare
- 参考 [clusterissuer Setup Guide](https://truecharts.org/charts/enterprise/clusterissuer/how-to#prerequisites)

无意外的话就能自动签发证书了，在其他 TrueCharts App 里填这个 cert 即可