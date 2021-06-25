# 8.一些使用配置问题

## docker pull x509:certificate signed by unknown authority <a id="articleContentId"></a>

```text
domain=xxx
port=443

mkdir -p /etc/docker/certs.d/${domain}
openssl s_client -showcerts -connect ${domain}:${port} < /dev/null | \
  sed -ne '/-BEGIN CERTIFICATE-/,/-END CERTIFICATE-/p' > /etc/docker/certs.d/${domain}/ca.crt
  
```

