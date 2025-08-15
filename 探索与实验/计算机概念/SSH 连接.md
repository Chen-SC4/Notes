# 如何建立 SSH 连接

这里以连接到远程仓库 `Gitee` 为例，介绍如何建立 `SSH` 连接。

## 1. 生成公钥和私钥

使用算法创建 `SSH ` 密钥对，可以使用 `RSA` 或 `ED25519` 算法。

```bash
ssh-keygen -t ed25519 -C "some-string@example.com" -f ~/.ssh/gitee_ed25519
```

- `-t` 指定算法为 `Ed25519`
- `-C` 添加注释，一般为邮箱
- `-f` 指定保存路径

### 1.1 Passphrase 密钥短语

在生成密钥对时，会遇到输入 `Passphrase` 的提示，这是为了增加密钥的安全性。如果不想输入 `Passphrase`，可以直接按回车。

`Passphrase` 可以看作是私钥的密码，如果私钥泄露，黑客还需要获得 `Passphrase` 才能使用私钥。

不过 `Passsphrase`  也有一个缺点，就是每次连接都需要输入 `Passphrase`。

在低风险的场景下，可以选择不需要 `Passphrase`。

### 1.2 配置 config 文件

在完成密钥对的生成之后，需要创建 `ssh` 的配置文件 `config`，配置文件的路径在 `~/.ssh/config`。

```plaintext
Host gitee.com
    HostName gitee.com
    User git
    IdentityFile ~/.ssh/gitee_ed25519
    IdentitiesOnly yes
```

|      配置项       |                 作用                 |          值           |
| :------------: | :--------------------------------: | :------------------: |
|      Host      |              别名（可自定义）              |      gitee.com       |
|    HostName    |               真实主机地址               |      gitee.com       |
|      User      |          连接用户（固定为 `git`）           |         git          |
|  IdentityFile  |               私钥文件路径               | ~/.ssh/gitee_ed25519 |
| IdentitiesOnly | 强制使用指定的密钥 `IdentityFile`，如果没有则连接失败 |         yes          |

## 2. 将公钥交给被连接方

将生成的公钥 `.ssh/gitee_ed25519.pub` 文件中的内容复制到 `Gitee` 中。

## 3. 测试连接

使用 `ssh -T git@gitee.com` 测试连接。

### 3.1 主机身份验证

在初次测试连接时，可能会有如下提示：

```plaintext
The authenticity of host 'gitee.com (180.76.198.77)' can't be established. ED25519 key fingerprint is SHA256:+ULzij2u99B9eWYFTw1Q4ErYG/aepHLbu96PAUCoV88. This key is not known by any other names. Are you sure you want to continue connecting (yes/no/[fingerprint])?
```

这是因为这是电脑与主机的第一次连接，所以需要确认上面给出的 `fingerprint` 是否与 `Gitee` 官方给出的一致。

如果一致，则输入 `yes`，电脑就会把主机的公钥保存到 `~/.ssh/known_hosts` 文件中，下次连接时就不会再提示。
