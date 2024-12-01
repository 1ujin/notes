## 暂存区回退工作区

`git restore --staged` 和 `git reset --mixed` 都能将文件从暂存区撤回到工作区，但它们的作用范围和应用场景有所不同。以下是它们的详细区别：

### 1. `git restore --staged`
- **作用**：仅将指定的文件从暂存区撤回到工作区，即“取消暂存”。
- **影响范围**：只影响指定的文件，不会影响其他文件的暂存状态。
- **适用场景**：当你只想“取消暂存”某个文件或一些文件，而不影响其他文件的暂存状态时。
- **用法示例**：
  ```bash
  git restore --staged <文件名>
  ```

- **示例效果**：
  假设你有三个文件 `file1`, `file2`, 和 `file3` 被暂存了，现在你只想把 `file2` 撤回到工作区，那么：
  ```bash
  git restore --staged file2
  ```
  这样 `file2` 将从暂存区移回工作区，而 `file1` 和 `file3` 的状态不变。

### 2. `git reset --mixed`
- **作用**：将指定的提交（默认为最新提交）撤销到工作区，同时将所有变更从暂存区移到工作区，使其处于“未暂存”状态。
- **影响范围**：影响到整个提交的所有文件（通常是最近一次提交的所有内容）。
- **适用场景**：当你希望撤销最近一次提交，并将所有更改返回到工作区（通常用于修改上次提交的内容）。
- **用法示例**：
  ```bash
  git reset --mixed HEAD^
  ```

- **示例效果**：
  假设你刚刚提交了一次，包含了多个文件的变更，现在你想要撤销这次提交并继续修改所有文件，那么：
  ```bash
  git reset --mixed HEAD^
  ```
  这样，最新的提交会被撤销，所有更改会从暂存区撤回到工作区，并处于“未暂存”状态。

### 总结
- **`git restore --staged`**：只影响暂存区中的指定文件，适合对部分文件执行“取消暂存”操作。
- **`git reset --mixed`**：影响整个提交，将最近的提交内容全部撤回到工作区，并取消暂存状态，适合撤销整个提交的内容。



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