# certbot-dns-aliyun

> 解决阿里云 DNS 不能自动为通配符证书续期的问题

## 原理

当我们使用 certbot 申请通配符证书时，我们需要手动添加 TXT 记录。每个 certbot 申请的证书有效期为 3 个月，虽然 certbot 提供了贴心的自动续期命令，但是当我们把自己续期命令配置为定时任务时，我们无法手动添加 TXT 记录。

好在 certbot 提供了一个 hook，可以编写一个 Shell 脚本，让脚本调用 DNS 服务商的 API 接口，动态添加 TXT 记录。

## 安装

1. 安装 aliyun cli 工具

   ```bash
   wget https://aliyuncli.alicdn.com/aliyun-cli-linux-latest-amd64.tgz
   tar xzvf aliyun-cli-linux-latest-amd64.tgz
   sudo cp aliyun /usr/local/bin
   ```

   安装完成后需要配置[凭证信息](https://help.aliyun.com/document_detail/110341.html)

2. 安装 certbot-dns-aliyun 插件

   ```bash
   wget https://cdn.jsdelivr.net/gh/justjavac/certbot-dns-aliyun/alidns.sh
   ```

   将脚本放到某个路径下面，例如 `/home/justjavac/alidns.sh`。

3. 申请证书

   测试是否能正确申请：

   ```bash
   certbot certonly  -d *.example.com --manual --preferred-challenges dns --manual-auth-hook "/home/justjavac/alidns.sh" --manual-cleanup-hook "/home/justjavac/alidns.sh clean" --dry-run
   ```

   正式申请时去掉 `--dry-run` 参数：

   ```bash
   certbot certonly  -d *.example.com --manual --preferred-challenges dns --manual-auth-hook "/home/justjavac/alidns.sh" --manual-cleanup-hook "/home/justjavac/alidns.sh clean"
   ```

4. 证书续期

   ```bash
   certbot renew --manual --preferred-challenges dns --manual-auth-hook "/home/justjavac/alidns.sh" --manual-cleanup-hook "/home/justjavac/alidns.sh clean" --dry-run
   ```

   如果以上命令没有错误，把 `--dry-run` 参数去掉。

5. 自动续期

   添加定时任务 crontab。

   ```bash
   crontab -e
   ```

   输入

   ```txt
   1 1 */1 * * root certbot renew --manual --preferred-challenges dns --manual-auth-hook "/home/justjavac/alidns.sh" --manual-cleanup-hook "/home/justjavac/alidns.sh clean" --deploy-hook "nginx -s reload"
   ```

   上面脚本中的 `--deploy-hook "nginx -s reload"` 表示在续期成功后自动重启 nginx。
