# TLS certificate application

## Certbot first

```bash
# 第1次请求必须使用该脚本，
# 配置DNSTXT记录，目的是域名所有权验证，letsencrypt保证域名是归属当前所有人的
# Install certbot
sudo bash install-certbot.sh
# 手动请求tls证书, 配置DNSTXT记录
sudo bash certbot-manual.sh example.com
# 手动验证dns的txt记录内容
nslookup -q=txt _acme-challenge.example.com
# 查看证书有效期
sudo openssl x509 -noout -dates -in /etc/letsencrypt/live/example.com/fullchain.pem

# 测试续签(need --manual-auth-hook auth-hook.sh)
sudo certbot renew --dry-run

```

## Certbot second

每个 dns 厂商的添加 dns 的 txt 记录不同，导致申请方式不同

```bash
# 第2次第3次第n次申请证书使用该脚本
# Install alicdn and alidns script
sudo bash aliyun/install-aliyuncli-alidns.sh
# 配置本地AK，需要从aliyun官网拿到AK
sudo bash aliyun/conf-aliyun.sh [YourAccessKeyId] [YourAccessKeySecret]

# 手动申请证书,root证书和泛域名证书不能混用
sudo bash aliyun/certbot-manual-aliyun.sh example.com root --dry-run
sudo bash aliyun/certbot-manual-aliyun.sh example.com wildcard --dry-run

# 自动更新证书
sudo bash aliyun/certbot-renew-aliyun.sh --dry-run
# aliyun自动续签
crontab -e
1 1 */1 * * root certbot renew --manual --preferred-challenges dns --manual-auth-hook "alidns" --manual-cleanup-hook "alidns clean" --deploy-hook "nginx -s reload"

```

# Other

## 自签证书

```bash
# 生成私钥
openssl genpkey -algorithm RSA -out cert.key -pkeyopt rsa_keygen_bits:2048
# 生成自签名证书，有效期为 3650 天（约 10 年）
openssl req -new -x509 -key cert.key -out cert.crt -days 3650 -subj "/C=CN/ST=State/L=City/O=Organization/OU=Unit/CN=localhost"
# 一些容器内可能不信任自签证书，加到信任列表即可
cp /etc/ssl/cert.crt /usr/local/share/ca-certificates/cert.crt
update-ca-certificates
curl --cert /etc/ssl/cert.crt --key /etc/ssl/cert.key https://localhost:443/api
```
