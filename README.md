# GPG 快速入门指南

**GnuPG** 同时支持对称密钥加密和公钥加密：

1. **对称密钥加密**：加密和解密使用同一个密钥。如果通信双方要使用对称加密算法通信，必须事先协商密钥。双方协商好密钥和加密算法后，发送者用此密钥加密一个文档，发送给接收者，接受者也使用这个密钥对其进行解密。对称密钥加密机制的主要问题是密钥交换。如果有 `n` 个人需要相互进行通信，那么总共需要 `n(n-1)/2` 个密钥。
2. **公钥加密**: 此加密机制需要生成公钥和私钥。私钥不应该与任何人共享，但是公钥需要被共享给任何需要接受你发送的加密消息的人。文档使用公钥进行加密，加密过的文档使用私钥进行解密。和对称密钥加密机制相对的，`n` 个人相互通信只需要 `n` 个密钥对。

更详细的讨论在 [GNU 隐私手册](https://www.gnupg.org/gph/en/manual.html) 中。

下文中我们主要讨论公钥加密机制，在最后一节中对对称密钥加密机制进行简要介绍。

## GPG 配置文件

**GnuPG** 及其相关的帮助工具默认情况下在 `~/.gnupg` 下查找配置文件，此目录可通过 `--homedir` 参数进行配置。默认情况下此目录权限设为 `700` ，它包含的文件的权限是 `600` 。保护 `~/.gnupg` 及其备份的安全**非常重要**。

常用的几个配置文件有：

- `gpg.conf`: `gpg` 的配置文件。这里可能包含适用于 `gpg` 的一些长选项。在第一次运行 `gpg` 时会生成一个简单的配置。
- `gpg-agent.conf`: `gpg-agent` 的配置文件。`gpg-agent` 被 `gpg` 用于请求并缓存密钥的口令。
- `dirmngr.conf`: `dirmngr` 的配置文件。`dirmngr` 负责访问 OpenPGP 密钥服务器，同时下载并管理证书撤销列表。在第一次运行 `gpg` 时会生成一个简单的配置。

[github.com:bfrg/gpg-guide](https://github.com/bfrg/gpg-guide) 中有几个配置文件的样本。

## 创建新密钥

首先创建一个新的密钥对：

```bash
$ gpg --full-gen-key
```

用户会被提示回答几个问题：

1. 密钥对：默认值是 `RSA and RSA`，意味着有一个主密钥用于签名，一个子密钥用于加密。
2. 密钥长度：通常来说，2048 位的密钥足够了。
3. 过期时间：一般将过期时间设在 1 年后。参见 [编辑密钥](#编辑密钥) 了解如何修改密钥过期时间。
4. 姓名及电子邮件地址。
5. 注释：为此密钥的用途添加说明。
6. 口令。

## 密钥管理

列出所有公钥：

```bash
$ gpg --list-keys
$ gpg --list-sigs   # 列出签名
$ gpg --fingerprint # 列出指纹
```

列出私钥：

```bash
$ gpg --list-secret-keys
```

删除一个公钥：

```bash
$ gpg --delete-key <key-id>
```

删除一个私钥：

```bash
$ gpg --delete-secret-key <key-id>
```

**注**：`<key-id>` 用于标示密钥，可能是密钥持有者的姓名、电子邮件地址；密钥的指纹；8位十六进制的ID；其他类似的东西。查看 `man gpg` 中的 HOW TO SPECIFY A USER ID 一节以了解更多细节。

## 备份私钥

强烈建议对私钥进行备份，并将备份存放在独立、安全的介质上。

备份私钥：

```bash
$ gpg --export-secret-keys --output private-key.asc --armor <key-id>
```

如果不通过 `--output <file>` 选项指定输出文件， **GnuPG** 会把导出的密钥写入到标准输出流。

导入一个私钥的备份：

```bash
$ gpg --allow-secret-key-import --import private-key.asc
```

## 编辑密钥

运行这个命令打开编辑密钥的菜单：

```bash
$ gpg --edit-key <key-id>
```

常用命令：

- `help`: 显示所有命令
- `passwd`: 修改口令
- `clean`: 清理不可用（被撤销或过期）的密钥
- `revkey`: 撤销密钥
- `addkey`: 向此密钥添加子密钥
- `expire`: 修改密钥的过期时间
- `adduid`: 向此密钥添加一个电子邮件地址

### 示例：重新启用一个已过期的密钥

密钥的过期时间可在任何时候被修改，即使它已经过期。

```bash
$ gpg --edit-key <key-id>
# gpg> key 1 (如果你想要更新子密钥，默认情况下选择主密钥)
# gpg> expire
#  (根据提示信息进行相应操作)
# gpg> save
```

### 示例：添加 UID

可以为密钥添加额外的电子邮件地址。

```bash
$ gpg --edit-key <key-id>
# gpg> adduid
#   Real name: 路人甲
#   Email address: new_email@address.com
#   ... 输入口令以解锁私钥 ...
# gpg> save
```

如果密钥有多个 UID ，我们可以选择一个主要的 UID ：

```bash
$ gpg --edit-key <key-id>
# gpg> uid 2
# gpg> primary
#   ... 输入口令以解锁密钥 ...
# gpg> save
```

## 公钥的导出和导入

把公钥导出到一个 7 位 ASCII 编码的文件：

```bash
$ gpg --armor --output some-public.key --export <key-id>
```

如果不指定 `<key-id>` ，所有密钥都会被导出。

公钥可以被任意分享，比如发送给你的朋友、在网站上发布、发布到公共的密钥服务器。

为其他人加密文件或验证他们的签名，我们需要对方的公钥。

导入某个人的公钥：

```bash
$ gpg --import some-public.key
```

## 撤销密钥

在有些情况下（比如私钥被泄露、UID被修改、忘记口令等），你需要告诉其他人之前的公钥不应再被使用。建议在生成新密钥后立刻生成一个撤销证书：

```bash
$ gpg --output revoke.asc --gen-revoke <key-id>
```

把 `revoke.asc` 文件保存在安全的地方。它可以被用于在私钥不再安全后撤销一个密钥。

要撤销一个密钥，导入撤销证书：

```bash
$ gpg --import revoke.asc
```

如果使用了密钥服务器，同时在服务器上更新信息：

```bash
$ gpg --keyserver subkeys.pgp.net --send <key-id>
```

注：当 `~/.gnupg/dirmngr.conf` 中指定了密钥服务器时， `--keyserver` 选项不是必须的。

## 加密和解密

导入公钥后，我们可以为这个接收者加密文件或消息：

```bash
$ gpg [--output <outfile>] --recipient <key-id> --encrypt <some-file>
```

默认情况下加密过的文件会被添加 `.gpg` 后缀。您也可以通过添加 `--option <outfile>` 选项指定输出文件。

如果本地有多个密钥，可以通过 `--local-user <key-id>` 选项指定将哪个密钥用于此次加密或解密。

解密一个用自己的公钥加密的文件：

```bash
$ gpg --output somefile.txt --decrypt somefile.txt.gpg
```

`gpg` 会提示输入口令，解密文件，并将解密后的数据写入到 `--output` 参数指定的文件中。

更多选项：

- `--armor, -a` 用 ASCII 文本方式输出
- `--hidden-recipient <user-id>, -R <user-id>` 将接受者 Key ID 信息放在加密过的消息中，在消息中隐藏接收者信息，防止流量分析。
- `--no-emit-version` 不在 ASCII 方式的输出中显示版本号。

## 签发并检查签名

为防止其他人声称为您，最好对每个加密后的文档进行签名。如果加密后的文档被修改，签名检查会失败。

用自己的密钥签名：

```bash
$ gpg --sign <file> --output <file.sig>
```

注意签名后的文件被压缩了，以二进制格式被输出，人类不可读。要创建一个明文的签名，执行：

```bash
$ gpg --clearsign <file> --output <file.sig>
```

这导致文档被 ASCII 方式的签名包起来，但不修改文档本身。明文签名的内容部分可被读取。只有在验证签名时才需要用到 **OpenPGP** 。

上面方法的缺点是接收者必须修改文件才能得到被签名的原始文件（因为签名是文件的一部分）。你也可以把签名写入到单独的文件中：

```bash
$ gpg --armor --output <file.sig> --detach-sign <file>
```

强烈推荐在对二进制文件签名时使用这种方式。

签名并加密一个文件：

```bash
$ gpg --sign <file> [--armor] --encrypt --recipient <user-id> [--local-user <key-id>]
```

在密文被签名的情况下，解密文件时会自动验证签名：

```bash
$ gpg --output <file> --decrypt <file.gpg>
```

如果只是要验证签名，用 `--verify` 选项：

```bash
$ gpg --verify <pgp-file/sig-file>
```

此命令需要事先导入签名者的公钥。

要验证 `archive.tar.gz` 及其独立的签名 `archive.tar.gz.asc` ，我们可以在下载签名者公钥后哦验证这个签名：

```bash
$ gpg --verify archive.tar.gz.asc archive.tar.gz
```

## 密钥服务器

把公钥发送到服务器，让其他人可以获取这个密钥：

```bash
$ gpg --keyserver <keyserver-name> --send-keys <key-id>
```

获取密钥信息：

```bash
$ gpg --search-keys <key-id> --keyserver <keyserver-name>
```

从密钥服务器导入密钥：

```bash
$ gpg --recv-keys <key-id>
```

建议使用[SKS 密钥服务器池](https://sks-keyservers.net/overview-of-pools.php#pool_hkps)。和密钥服务器的通信使用 HKPS 协议。

如果要在需要时自动从密钥服务器下载密钥，在 `~/.gnupg/gpg.conf` 中添加下面一行：

```conf
keyserver-options autokey-retrieve
```

要了解如何设立密钥服务器，参见 [GPG 最佳实践](https://riseup.net/en/gpg-best-practices)。注意从 GnuPG 2.1 起部分选项被移动到 `dirmngr` 中，需要在 `~/.gnupg/dirmngr.conf` 中配置。

## 对称密钥加密

可以使用对称加密算法对文档进行加密。默认的加密算法是 AES-128 ，也可以通过 `--cipher-algo` 选项指定要使用的加密算法。

用对称密钥加密文件：

```bash
$ gpg --symmetric <file>
```

会将加密后的文件写入到添加了 `.gpg` 后缀的文件中。

解密文件：

```bash
$ gpg file.txt.gpg [--output file.txt]
```

用户会被提示输入口令以解密文件。

## 参考资料及扩展阅读

* [GNU 隐私手册][gnu-handbook]
* [Arch Linux wiki][arch-gpg]
* [OpenPGP 最佳实践][best-practices]
* [LinuxCrypto][linux-crypto]
* [Linux 命令行加密工具][howtoforge]
* [GPG 快速入门][madboa]
* [Ana 的博客][ana]

[gnu-handbook]: https://www.gnupg.org/gph/en/manual.html
[arch-gpg]: https://wiki.archlinux.org/index.php/GnuPG
[sks-pool]: https://sks-keyservers.net/overview-of-pools.php#pool_hkps
[best-practices]: https://riseup.net/en/gpg-best-practices
[linux-crypto]: https://sanctum.geek.nz/arabesque/series/linux-crypto
[howtoforge]: https://www.howtoforge.com/tutorial/linux-commandline-encryption-tools
[madboa]: https://www.madboa.com/geek/gpg-quickstart
[ana]: https://ekaia.org/blog/2009/05/10/creating-new-gpgkey
