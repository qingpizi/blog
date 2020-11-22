# 【转】免费 HTTPS 证书 Let's Encrypt 安装教程



### [\#](https://nfangxu.com/linux/free-https-certificate-let-s-encrypt-installation-tutorial.html#%E7%AC%AC%E9%9B%B6%E6%AD%A5%EF%BC%9A%E6%9C%AC%E6%96%87%E4%BD%BF%E7%94%A8%E7%8E%AF%E5%A2%83)第零步：本文使用环境 <a id="&#x7B2C;&#x96F6;&#x6B65;&#xFF1A;&#x672C;&#x6587;&#x4F7F;&#x7528;&#x73AF;&#x5883;"></a>

* 阿里云服务器
* ubuntu 16.04
* nginx

### [\#](https://nfangxu.com/linux/free-https-certificate-let-s-encrypt-installation-tutorial.html#%E7%AC%AC%E4%B8%80%E6%AD%A5%EF%BC%9A%E5%88%9B%E5%BB%BA-let-s-encrypt-%E8%B4%A6%E5%8F%B7)第一步：创建 Let's Encrypt 账号 <a id="&#x7B2C;&#x4E00;&#x6B65;&#xFF1A;&#x521B;&#x5EFA;-let-s-encrypt-&#x8D26;&#x53F7;"></a>

> Let's Encrypt使用一个私钥来进行账号的创建与登陆，因此我们需要使用openssl创建一个account.key。

```text
openssl genrsa 4096 > account.key
```

> 如果你已经有一个Let's Encrypt key的话，那么只需要做一次转换，因为Let's Encrpt 的客户端生成的key是JWK格式，而acm-tiny使用的是PEM格式。转换key需要使用一个脚本

```text
# 下载脚本
wget -O - "https://gist.githubusercontent.com/JonLundy/f25c99ee0770e19dc595/raw/6035c1c8938fae85810de6aad1ecf6e2db663e26/conv.py" > conv.py

# 把private key 拷贝到你的工作目录
cp /etc/letsencrypt/accounts/acme-v01.api.letsencrypt.org/directory/<id>/private_key.json private_key.json

# 创建一个DER编码的private key
openssl asn1parse -noout -out private_key.der -genconf <(python conv.py private_key.json)

# 转换成PEM格式
openssl rsa -in private_key.der -inform der > account.key
```

### [\#](https://nfangxu.com/linux/free-https-certificate-let-s-encrypt-installation-tutorial.html#%E7%AC%AC%E4%BA%8C%E6%AD%A5%EF%BC%9A%E5%88%9B%E5%BB%BA%E5%9F%9F%E5%90%8D%E7%9A%84csr%EF%BC%88certificate-signing-request%EF%BC%89)第二步：创建域名的CSR（CERTIFICATE SIGNING REQUEST） <a id="&#x7B2C;&#x4E8C;&#x6B65;&#xFF1A;&#x521B;&#x5EFA;&#x57DF;&#x540D;&#x7684;csr&#xFF08;certificate-signing-request&#xFF09;"></a>

> Let's Encrypt 使用的ACME协议需要一个CSR文件，可以使用它来重新申请HTTPS证书，接下来我们就可以创建域名CSR，在创建CSR之前，我们需要给我们的域名创建一个私钥（这个和上面的账户私钥无关）。

```text
openssl genrsa 4096 > domain.key #创建普通域名私钥 
```

> 接下来，使用你的域名私钥创建CSR文件，这一步里面是可以增加最多100个需要加密的域名的，替换下面的foofish.net即可（注意，稍后会说到，每个域名都会涉及到验证）

```text
# 单个域名
openssl req -new -sha256 -key domain.key -subj "/CN=nfangxu.com" > domain.csr

# 多个域名(如果你有多个域名，比如：www.foofish.net和foofish.net，使用这种方式)
openssl req -new -sha256 -key domain.key \
-subj "/" -reqexts SAN \
-config <(cat /etc/ssl/openssl.cnf \
<(printf "[SAN]\nsubjectAltName=DNS:nfangxu.com,DNS:www.nfangxu.com")) \
> domain.csr
```

> **执行这一步时，需要指定 openssl.cnf 文件，一般这个文件在你的 openssl 安装目录底下。**

### [\#](https://nfangxu.com/linux/free-https-certificate-let-s-encrypt-installation-tutorial.html#%E7%AC%AC%E4%B8%89%E6%AD%A5%EF%BC%9A%E9%85%8D%E7%BD%AE%E5%9F%9F%E5%90%8D%E9%AA%8C%E8%AF%81)第三步：配置域名验证 <a id="&#x7B2C;&#x4E09;&#x6B65;&#xFF1A;&#x914D;&#x7F6E;&#x57DF;&#x540D;&#x9A8C;&#x8BC1;"></a>

> CA 在签发 DV（Domain Validation）证书时，需要验证域名所有权。传统 CA 的验证方式一般是往 admin@foofish.net 发验证邮件，而 Let's Encrypt 是在你的服务器上生成一个随机验证文件，再通过创建 CSR 时指定的域名访问，如果可以访问则表明你对这个域名有控制权。 首先创建用于存放验证文件的目录，例如：

```text
mkdir -p /path/to/your/web/root/.well-known/acme-challenge/
```

> 然后配置一个 HTTP 服务，以 Nginx 为例：\(**注意：这里的端口是80，不是443**）

```text
server {
    listen 80;

    server_name nfangxu.com www.nfangxu.com;

    root /path/to/your/web/root/;
    index index.php;

    location ^~ /.well-known/acme-challenge/ {
            try_files $uri =404;
    }

    # 后期加的, 用于 http 向 https 的跳转
    location / {
            rewrite ^(.*)$ https://nfangxu.com/$1 permanent;
    }
}

```

> **这个验证服务以后更新证书还要用到，需要一直保留。**

### [\#](https://nfangxu.com/linux/free-https-certificate-let-s-encrypt-installation-tutorial.html#%E7%AC%AC%E5%9B%9B%E6%AD%A5%EF%BC%9A%E8%8E%B7%E5%8F%96%E7%BD%91%E7%AB%99%E8%AF%81%E4%B9%A6)第四步：获取网站证书 <a id="&#x7B2C;&#x56DB;&#x6B65;&#xFF1A;&#x83B7;&#x53D6;&#x7F51;&#x7AD9;&#x8BC1;&#x4E66;"></a>

> 先把 acme-tiny 脚本保存到之前的 ssl 目录：

```text
wget https://raw.githubusercontent.com/diafygi/acme-tiny/master/acme_tiny.py
```

> 指定账户私钥、CSR 以及验证目录，执行脚本：

```text
python acme_tiny.py \ 
--account-key ./account.key \ 
--csr ./domain.csr \ 
--acme-dir /path/to/your/web/root/.well-known/acme-challenge/ \ 
> ./signed.crt
```

> 如果一切正常，当前目录下就会生成一个 signed.crt，这就是申请好的证书文件。

### [\#](https://nfangxu.com/linux/free-https-certificate-let-s-encrypt-installation-tutorial.html#%E7%AC%AC%E4%BA%94%E6%AD%A5%EF%BC%9A%E5%AE%89%E8%A3%85%E8%AF%81%E4%B9%A6)第五步：安装证书 <a id="&#x7B2C;&#x4E94;&#x6B65;&#xFF1A;&#x5B89;&#x88C5;&#x8BC1;&#x4E66;"></a>

> 证书生成后，就可以把它配置在web 服务器上了，需要注意的是，Nginx需要追加一个Let's Encrypt的中间证书，在 Nginx 配置中，需要把中间证书和网站证书合在一起：

```text
# 获取证书
wget -O - https://letsencrypt.org/certs/lets-encrypt-x1-cross-signed.pem > intermediate.pem

# 合并证书
cat signed.crt intermediate.pem > chained.pem
```

> 最终，修改 Nginx 中有关证书的配置并 reload 服务即可：

```text
server {
    listen 443;

    # ssl 配置
    ssl on;
    ssl_certificate /path/to/chained.pem;
    ssl_certificate_key /path/to/domain.key;
    ssl_session_timeout 5m;
    ssl_protocols TLSv1 TLSv1.1 TLSv1.2;
    ssl_ciphers ECDHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES128-GCM-SHA256:DHE-RSA-AES256-GCM-SHA384:ECDHE-RSA-AES256-SHA384:ECDHE-RSA-AES128-SHA256:ECDHE-RSA-AES256-SHA:ECDHE-RSA-AES128-SHA:DHE-RSA-AES256-SHA:DHE-RSA-AES128-SHA;
    ssl_session_cache shared:SSL:50m;
    ssl_prefer_server_ciphers on;
    # END

    server_name nfangxu.com www.nfangxu.com;

    root /path/to/your/web/root/;
    index index.php index.html index.htm index.nginx-debian.html;

    location / {
            try_files $uri $uri/ /index.php;
    }

    location ~ \.php$ {
            include snippets/fastcgi-php.conf;
            fastcgi_pass unix:/run/php/php7.2-fpm.sock;
    }

    location ~ /\.ht {
            deny all;
    }
}

```

### [\#](https://nfangxu.com/linux/free-https-certificate-let-s-encrypt-installation-tutorial.html#%E7%AC%AC%E5%85%AD%E6%AD%A5%EF%BC%9A%E5%AE%9A%E6%9C%9F%E6%9B%B4%E6%96%B0)第六步：定期更新 <a id="&#x7B2C;&#x516D;&#x6B65;&#xFF1A;&#x5B9A;&#x671F;&#x66F4;&#x65B0;"></a>

> Let’s Encrypt 签发的证书只有90天有效期，但可以通过脚本定期更新。你可以创建了一个自动更新脚本 `cert_refresh.sh` ，内容如下：

```text
#!/bin/bash

# 创建CSR
python /path/to/acme_tiny.py --account-key /path/to/account.key --csr /path/to/domain.csr --acme-dir /path/to/your/web/root/.well-known/acme-challenge/ > /tmp/signed.crt || exit

# 获取
wget -O - https://letsencrypt.org/certs/lets-encrypt-x1-cross-signed.pem > /tmp/intermediate.pem

# 合并
cat /tmp/signed.crt /tmp/intermediate.pem > /path/to/chained.pem

# 重新载入 nginx 配置文件
service nginx restart
```

> 修改crontab配置，加入以下内容：

```text
# 每个月执行一次
0 0 1 * * /path/to/cert_refresh.sh 2>> /var/log/acme_tiny.log
```

> 大功告成，访问下自己的HTTPS网站是否正常，不出意外的话，网站已经正式启用HTTPS了 !

来源：[https://nfangxu.com/linux/free-https-certificate-let-s-encrypt-installation-tutorial.html](https://nfangxu.com/linux/free-https-certificate-let-s-encrypt-installation-tutorial.html)

