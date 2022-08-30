# Harbor

## 一、相关要求

### 1.环境

```
Component	Version
Postgresql	13.3.0
Redis	6.0.13
Beego	1.9.0
Chartmuseum	0.9.0
Docker/distribution	2.7.1
Docker/notary	0.6.1
Helm	2.9.1
Swagger-ui	3.22.1
```

### 2.硬件要求

| Resource | Minimum | Recommended |
| :------- | :------ | :---------- |
| CPU      | 2 CPU   | 4 CPU       |
| Mem      | 4 GB    | 8 GB        |
| Disk     | 40 GB   | 160 GB      |

### 3.软件要求

| Software       | Version                       | Description                                                  |
| :------------- | :---------------------------- | :----------------------------------------------------------- |
| Docker engine  | Version 17.06.0-ce+ or higher | For installation instructions, see [Docker Engine documentation](https://docs.docker.com/engine/installation/) |
| Docker Compose | Version 1.18.0 or higher      | For installation instructions, see [Docker Compose documentation](https://docs.docker.com/compose/install/) |
| Openssl        | Latest is preferred           | Used to generate certificate and keys for Harbor             |

### 4.网络端口

| Port | Protocol | Description                                                  |
| :--- | :------- | :----------------------------------------------------------- |
| 443  | HTTPS    | Harbor portal and core API accept HTTPS requests on this port. You can change this port in the configuration file. |
| 4443 | HTTPS    | Connections to the Docker Content Trust service for Harbor. Only required if Notary is enabled. You can change this port in the configuration file. |
| 80   | HTTP     | Harbor portal and core API accept HTTP requests on this port. You can change this port in the configuration file. |



------

## 二、下载安装

1.到Habor的发布页面

```
https://github.com/goharbor/harbor/releases
```

2.下载你需要安装的版本（online or offline）

3.可选下载相应*.asc 文件以验证包是否为正版。

该`*.asc`文件是一个 OpenPGP 密钥文件。执行以下步骤以验证下载的捆绑包是否为正版。

- 获取文件的公钥`*.asc`。

```
gpg --keyserver hkps://keyserver.ubuntu.com --receive-keys 644FF454C0B4115C
```

你应该看到消息` public key "Harbor-sign (The key for signing Harbor build) <jiangd@vmware.com>" imported`

- 通过运行以下命令之一来验证软件包是否为正版。

  - 在线安装程序	

    ```
    gpg -v --keyserver hkps://keyserver.ubuntu.com --verify harbor-online-installer-version.tgz.asc		
    ```

    

  - 离线安装程序

    ```
    gpg -v --keyserver hkps://keyserver.ubuntu.com --verify harbor-offline-installer-version.tgz.asc
    ```

该`gpg`命令验证捆绑包的签名是否与`*.asc`密钥文件的签名匹配。您应该看到签名正确的确认。

```
gpg: armor header: Version: GnuPG v1
gpg: assuming signed data in 'harbor-online-installer-v2.0.2.tgz'
gpg: Signature made Tue Jul 28 09:49:20 2020 UTC
gpg:                using RSA key 644FF454C0B4115C
gpg: using pgp trust model
gpg: Good signature from "Harbor-sign (The key for signing Harbor build) <jiangd@vmware.com>" [unknown]
gpg: WARNING: This key is not certified with a trusted signature!
gpg:          There is no indication that the signature belongs to the owner.
Primary key fingerprint: 7722 D168 DAEC 4578 06C9  6FF9 644F F454 C0B4 115C
gpg: binary signature, digest algorithm SHA1, key algorithm rsa4096
```

4.解压安装包

在线安装包：

```
tar xzvf harbor-online-installer-version.tgz 
```

离线安装包：

```
tar xzvf harbor-offline-installer-version.tgz
```



------

## 三、配置harbor.yml

进入`harbor`目录，并拷贝复制一份harbor.yml.example 为harbor.yml

```
cp harbor.yml.example  harbor.yml
```

编辑harbor.yml

```
# harbor.yml中按需修改或添加如下内容
## 修改hostname
hostname: harbor.test.com   #可以跟domain一样
## 修改admin密码
harbor_admin_password: xxxxxxx
## 暂时关闭https
#https:
  # https port for harbor, default is 443
  #  port: 443
  # The path of cert and key files for nginx
  # certificate: /your/certificate/path   #为证书路径，一般为/data/harbor/harbor-cert
  # private_key: /your/private/key/path   #为证书路径
## 添加禁止用户自注册
self_registration: off
## 设置只有管理员可以创建项目（可选）
# project_creation_restriction: adminonly
## 指定数据目录，若无则手动创建该目录
data_volume: /data/harbor      #需要修改
```

修改完`harbor.yml`后，执行以下命令。

```
./prepare
./install.sh
```

此时即可通过IP地址访问harbor了，注意：此时为HTTP访问，若需要HTTPS，则需要配置相关TLS证书。

## 四、配置Harbor的HTTPS访问

默认情况下，Harbor 不附带证书。可以在没有安全性的情况下部署 Harbor，以便您可以通过 HTTP 连接到它。但是，只有在没有连接到外部 Internet 的气隙测试或开发环境中才能使用 HTTP。在非气隙环境中使用 HTTP 会使您面临中间人攻击。在生产环境中，始终使用 HTTPS。如果您启用 Content Trust with Notary 以正确签署所有图像，则必须使用 HTTPS。

要配置 HTTPS，您必须创建 SSL 证书。您可以使用由受信任的第三方 CA 签名的证书，也可以使用自签名证书。本节介绍如何使用 [OpenSSL](https://www.openssl.org/)创建 CA，以及如何使用您的 CA 签署服务器证书和客户端证书。您可以使用其他 CA 提供程序，例如 [Let's Encrypt](https://letsencrypt.org/)。

下面的过程假设您的 Harbor 注册中心的主机名是`yourdomain.com`，并且它的 DNS 记录指向您正在运行 Harbor 的主机。

## 生成证书颁发机构证书

在生产环境中，您应该从 CA 获取证书。在测试或开发环境中，您可以生成自己的 CA。要生成 CA 证书，请运行以下命令。

1.生成 CA 证书私钥。

```sh
openssl genrsa -out ca.key 4096
```

2.生成 CA 证书。

调整`-subj`选项中的值以反映您的组织。如果您使用 FQDN 连接您的 Harbor 主机，则必须将其指定为公用名 ( `CN`) 属性。

```sh
openssl req -x509 -new -nodes -sha512 -days 3650 \
 -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=yourdomain.com" \
 -key ca.key \
 -out ca.crt
```

## 生成服务器证书

证书通常包含一个`.crt`文件和一个`.key`文件，例如，`yourdomain.com.crt`和`yourdomain.com.key`.

1.生成私钥。

```sh
openssl genrsa -out yourdomain.com.key 4096
```

2.生成证书签名请求 (CSR)。

调整`-subj`选项中的值以反映您的组织。如果您使用 FQDN 连接您的 Harbor 主机，则必须将其指定为公用名称 ( `CN`) 属性并在密钥和 CSR 文件名中使用它。

```
openssl req -sha512 -new \
    -subj "/C=CN/ST=Beijing/L=Beijing/O=example/OU=Personal/CN=yourdomain.com" \
    -key yourdomain.com.key \
    -out yourdomain.com.csr
```

3.生成 x509 v3 扩展文件。

无论您是使用 FQDN 还是 IP 地址连接到您的 Harbor 主机，您都必须创建此文件，以便为您的 Harbor 主机生成符合主题备用名称 (SAN) 和 x509 v3 的证书扩展要求。替换`DNS`条目以反映您的域。

```
cat > v3.ext <<-EOF
authorityKeyIdentifier=keyid,issuer
basicConstraints=CA:FALSE
keyUsage = digitalSignature, nonRepudiation, keyEncipherment, dataEncipherment
extendedKeyUsage = serverAuth
subjectAltName = @alt_names

[alt_names]
DNS.1=yourdomain.com
DNS.2=yourdomain
DNS.3=hostname
EOF
```

4.使用该`v3.ext`文件为您的 Harbor 主机生成证书。

将`yourdomain.com`CRS 和 CRT 文件名中的 替换为 Harbor 主机名。

```
openssl x509 -req -sha512 -days 3650 \
    -extfile v3.ext \
    -CA ca.crt -CAkey ca.key -CAcreateserial \
    -in yourdomain.com.csr \
    -out yourdomain.com.crt
```

## 向 Harbor 和 Docker 提供证书

生成 、 和 文件后`ca.crt`，`yourdomain.com.crt`您`yourdomain.com.key`必须将它们提供给 Harbor 和 Docker，并重新配置 Harbor 以使用它们。

1.将服务器证书和密钥复制到 Harbor 主机上的 certficates 文件夹中。

```
cp yourdomain.com.crt /data/cert/
cp yourdomain.com.key /data/cert/
```

2.转换`yourdomain.com.crt`为`yourdomain.com.cert`, 供 Docker 使用。

Docker 守护进程将`.crt`文件解释为 CA 证书，将`.cert`文件解释为客户端证书。

```
openssl x509 -inform PEM -in yourdomain.com.crt -out yourdomain.com.cert
```

3.将服务器证书、密钥和 CA 文件复制到 Harbor 主机上的 Docker 证书文件夹中。您必须先创建适当的文件夹。本文采用的路径为：`/data/harbor/harbor-cert.com`

```
cp yourdomain.com.cert /etc/docker/certs.d/yourdomain.com/
cp yourdomain.com.key /etc/docker/certs.d/yourdomain.com/
cp ca.crt /etc/docker/certs.d/yourdomain.com/
```

如果您将默认`nginx`端口 443 映射到不同的端口，请创建文件夹`/etc/docker/certs.d/yourdomain.com:port`或`/etc/docker/certs.d/harbor_IP:port`.

4.重启 Docker 引擎。

```
systemctl restart docker
```

## 部署或重新配置 Harbor

如果您已经使用 HTTP 部署了 Harbor，并希望将其重新配置为使用 HTTPS，请执行以下步骤。

1. 运行`prepare`脚本以启用 HTTPS。

   Harbor 使用`nginx`实例作为所有服务的反向代理。您使用`prepare`脚本配置`nginx`为使用 HTTPS。位于Harbor 安装程序包中，与脚本`prepare`处于同一级别。`install.sh`

   ```sh
   ./prepare
   ```

2. 如果 Harbor 正在运行，请停止并删除现有实例。

   您的图像数据保留在文件系统中，因此不会丢失任何数据。

   ```sh
   docker-compose down -v
   ```

3. 重启港口：

   ```sh
   docker-compose up -d
   ```

## 验证 HTTPS 连接

为 Harbor 设置 HTTPS 后，您可以通过执行以下步骤来验证 HTTPS 连接。

- 打开浏览器并输入[https://yourdomain.com](https://yourdomain.com/)。它应该显示 Harbor 界面。

  某些浏览器可能会显示一条警告，指出证书颁发机构 (CA) 未知。当使用不是来自受信任的第三方 CA 的自签名 CA 时，会发生这种情况。您可以将 CA 导入浏览器以删除警告。

- 在运行 Docker 守护程序的机器上，检查`/etc/docker/daemon.json`文件以确保`-insecure-registry`未为[https://yourdomain.com](https://yourdomain.com/)设置该选项。

- 从 Docker 客户端登录到 Harbor。

  ```sh
  docker login yourdomain.com
  ```

  如果您已将`nginx`443 端口映射到其他端口，请在`login`命令中添加该端口。

  ```sh
  docker login yourdomain.com:port
  ```



------

## harbor 版本升级：

https://goharbor.io/docs/2.4.0/administration/upgrade/

1.登录到 Harbor 主机，如果它仍在运行，请停止现有的 Harbor 实例。

```
cd harbor
docker-compose down
```

2.备份 Harbor 的当前文件，以便在必要时回滚到当前版本。

```
cp -a  harbor /my_backup_dir/harbor
```

3.[从https://github.com/goharbor/harbor/releases](https://github.com/goharbor/harbor/releases)获取最新的 Harbor 发布包 并解压。

4.在升级 Harbor 之前，请执行迁移。迁移工具位于以 docker 映像形式提供的港口准备工具中。您可以从 docker hub 拉取图像。在以下命令中：

```
docker pull goharbor/prepare:[tag]
```

或者，如果您使用的是脱机安装程序包，则可以从脱机安装程序包中包含的映像 tarball 加载它。在以下命令中将 [tag] 替换为新的 Harbor 版本，例如 v1.10.0：

```
tar zxf <offline package>
docker image load -i harbor/harbor.[version].tar.gz
```

5.复制`/path/to/old/harbor.yml`到`harbor.yml`并升级它。

```
docker run -it --rm -v /:/hostfs goharbor/prepare:[tag] migrate -i ${path to harbor.yml}
```

6.在该`./harbor`目录中，运行`./install.sh`脚本以安装新的 Harbor 实例。

```
harbor/common/config/shared/trust-certificates  会缺CA证书
```

