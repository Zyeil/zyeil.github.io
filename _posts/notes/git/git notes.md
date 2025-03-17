由于涉及到多个账户管理，原本的「Github Desktop」看起来似乎有点不太够用，于是详细地学习「git」似乎成了一个可行之道。

## 不借助「Github Desktop」进行部署

clone remote repository to local.

```bash
# 后面加上 https 或者 SSH
git clone https://github.com/Usename/repository.github.io
```

现在可以对文件进行编辑。

使用这些命令将修改同步到 remote repository.

```bash
git add .
git commit -m "注释所修改的内容"
git push origin master/main # 总之就是 branch 的名字
```

## 在一台设备上管理多个账号

以两个账号为例，现在把这两个账号记作 User1 and User2.

### 为每个账号生成单独的「SSH」密匙

为「User1」生成密匙：

```bash
ssh-keygen -t ed25519 -C "User1的登录邮箱"
```

>注意看「Bash」在运行时显示的密匙存储地址。

Copy the pub key of **User1** and add it on the Web page.

路径：Settings > SSH and GPG keys > New SSH key

### 配置「SSH」文件

编辑`~/.ssh/config`文件（没有则创建）
```bash
nano ~/.ssh/config
```

add the context:
```bash
# User1 账户
Host github-user1
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_user1

# User2 账户
Host github-user2
  HostName github.com
  User git
  IdentityFile ~/.ssh/id_ed25519_user2
```

#### 「nano」编辑器

- `Ctrl + 0` to save the document;
- `Enter` to ensure the name of the document;
- `Ctrl + X` to exit the editor.

```bash
# used to check if the document is correct
cat ~/.ssh/config
```

#### 将「Github」的公匙添加到`~/.ssh/known_hosts`

```bash
ssh-keyscan -t ed25519 github.com >> ~/.ssh/known_hosts
```

#### 测试连接

```bash
# test the connection of User1
ssh -T git@github-user1

# test the connection of User2
ssh -T git@github-user2
```

if you see:
```bash
Host key verification failed.
```

then you should add the pub key to the file: `~/.ssh/known_hosts`
```bash
ssh-keyscan -t ed25519 github.com >> ~/.ssh/known_hosts
```

Then test the connection again!
```bash
ssh -T git@github-user1
```

### 克隆仓库时使用不同的「SSH」配置

启动「SSH」代理
```bash
eval "$(ssh-agent -s)"
```

添加「SSH」密匙
```bash
# 定位到对应密匙
ssh-add ~/.ssh/id_ed25519
```

```bash
git clone "从「Github」网页获取的「SSH」"
```

### 在每个仓库中配置独立的 Git 用户信息

将「bash」定位到对应仓库，然后
```bash
git config user.name "User1"
git config user.email "correspongding email"
```

确认局部的「Git」配置
```bash
git config --local user.name
git config --local user.email
```

手动加载「SSH Key」
```bash
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_ed25519
```

### 修改某个仓库使用的提交账号

```bash
# 查看此仓库使用的账号
git config user.name
git config user.email

# 修改
git config user.name "name"
git config user.email "correspoding email"
```

### 修改历史提交的用户名

```bash
git rebase -i HEAD~n
```

然后修改对应的提交记录

```bash
git rebase --continue
```

## SSH 认证问题解决

### 检查远程仓库地址

```bash
git remote -v
```

### 确保 SSH 正确

```bash
ssh -T git@github.com
```

### 添加 SSH Key
```bash
# 确保 SSH agent is running
eval $(ssh-agent -s)

# 添加 SSH Key
ssh-add ~/.ssh/id_ed25519
```
### 让 SSH 代理自动加载密匙

```bash
# 打开nano配置文件
nano ~/.bashrc

# 添加
eval $(ssh-agent -s)
ssh-add ~/.ssh/id_rsa_frederic

# 保存
source ~/.bashrc
```