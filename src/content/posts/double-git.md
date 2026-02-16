---
title: 同一电脑，双开Github?
published: 2026-02-16
description: '同一电脑里推送两个github账号上的更改'
image: '/cover6.webp'
tags: ['git','经验杂谈','折腾','github']
category: '经验杂谈'
draft: false 
lang: 'zh_CN'
---
## 同一电脑，双开Github?

现在是北京时间23：50，在还有10分钟跨年的除夕夜里，我————正在改博客（（（  
从晚上6点一直弄到现在，中间遇到了一个很有意思的小问题。遂记录下来。  
大家除夕快乐哇φ(゜▽゜*)♪

### 一、 生成多组 SSH 密钥对

GitHub 不允许在多个账号上使用同一个 SSH 公钥(当遇到多个账号使用同一个公钥时，GitHub会选择第一个配置这个公钥的账号作为推送时使用的身份)，因此必须为每个账号生成独立的密钥。

1. **为账号 A 生成密钥**：
```bash
ssh-keygen -t rsa -C "email_A@example.com"
# 在提示输入文件名时，指定一个唯一的名称，如：id_rsa_github_main

```

2. **为账号 B 生成密钥**：
```bash
ssh-keygen -t rsa -C "email_B@example.com"
# 指定名称如：id_rsa_github_second

```

3. **将密钥添加到系统中**（防止失效）：
```bash
ssh-add ~/.ssh/id_rsa_github_main
ssh-add ~/.ssh/id_rsa_github_second

```

4. **复制公钥到Github**
```bash
cat ~/.ssh/id_rsa_github_main.pub
```
>公钥的内容为一串长字符串，字符串的末尾为你输入的密钥备注  
之后访问 Github 的用户的Settings界面，并来到 SSH and GPG keys 面板  

点击 New SSH key ，之后将前面输出的公钥内容粘贴到 输入框中：


### 二、 配置 SSH Config 文件

通过修改 `~/.ssh/config` 文件（若不存在则新建），为不同的账号设置“别名（Host）”，这是区分账号的核心。

在文件中输入以下内容：

```text
# 账号 A (主账号)
Host github.com
    HostName github.com
    User git
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_github_main

# 账号 B (小号/博客账号)
Host github_second
    HostName github.com
    User git
    PreferredAuthentications publickey
    IdentityFile ~/.ssh/id_rsa_github_second

```

* **注意**：`Host` 后面的名字是自定义的别名，后续克隆和推送代码时会用到。

### 三、 在不同项目中使用对应的账号

配置好 SSH 后，关键在于如何让 Git 知道某个项目该用哪个账号。

1. **修改远程仓库地址**：
如果你从小号（账号 B）克隆了一个博客仓库，默认地址是 `git@github.com:user_b/blog.git`。你需要将 `github.com` 修改为你设置的**别名** `github_second`：
```bash
# 如果是新项目
git clone git@github_second:user_b/blog.git

# 如果是已有项目，修改 remote url
git remote set-url origin git@github_second:user_b/blog.git

```


2. **单独配置项目用户信息**：
:::note
为了保证博客提交记录显示的作者信息正确，不要使用全局配置（`--global`）  
需要在**对应项目目录**下进行局部配置：
:::

```bash
git config user.name "Your_Name_B"
git config user.email "email_B@example.com"

```


### 四、 验证配置

使用以下命令测试连接是否指向了正确的账号：

* 测试主账号：`ssh -T git@github.com`
* 测试小号：`ssh -T git@github_second`

如果看到 `Hi [用户名]! You've successfully authenticated...` 且用户名对应正确，说明配置成功。
