## 创建密钥

###  生成新的 SSH 密钥（如果没有密钥）

如果你的系统中没有 SSH 密钥，可以按照以下步骤生成一个：

```bash
ssh-keygen -t rsa -b 4096 -C "your_email@example.com"
```

按照提示保存密钥（通常选择默认路径 `~/.ssh/id_rsa`），并为密钥设置一个 passphrase。

###  将公钥添加到 GitHub 账户

你需要将公钥 `id_rsa.pub` 添加到 GitHub 上。首先，复制公钥内容：

```bash
cat ~/.ssh/id_rsa.pub
```

将输出的内容复制到 GitHub 上：

1. 登录到 GitHub。
2. 进入 **Settings**。
3. 在左侧栏选择 **SSH and GPG keys**。
4. 点击 **New SSH key**，粘贴公钥并保存。

###  配置 SSH 客户端

确认你的 SSH 客户端正确地使用了你的密钥。编辑或创建 `~/.ssh/config` 文件，并添加以下内容：

```bash
Host github.com
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_rsa
```

###  测试 SSH 连接

你可以通过以下命令来测试 SSH 连接是否正常：

```bash
ssh -T git@github.com
```

如果一切正常，你应该会看到类似于以下的提示：

```
Hi XXX! You've successfully authenticated, but GitHub does not provide shell access.
```

###  确保 Git 使用 SSH URL

检查你的 Git 仓库 URL 是否是 SSH 格式。使用以下命令查看仓库的远程 URL：

```bash
git remote -v
```

显示：

```
origin  git@github.com:1ujin/LeetCode.git (fetch)
origin  git@github.com:1ujin/LeetCode.git (push)
```

确保显示的是以 `git@github.com:` 开头的 URL，而不是 `https://`。如果不是，可以使用以下命令将其更改为 SSH URL：

```bash
git remote set-url origin git@github.com:username/repository.git
```

###  再次尝试 `git pull`

完成以上步骤后，再次尝试运行 `git pull`，看看问题是否解决：

```bash
git pull
```

### 将私钥文件加入 SSH agent

查看私钥文件

```shell
cat ~/.ssh/id_rsa
```

启动 SSH agent

```shell
ssh-agent -s
```

添加私钥

```shell
ssh-add ~/.ssh/id_rsa
```

