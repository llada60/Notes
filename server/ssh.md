# ssh

[TOC]

local files `~/.ssh/config`

## Security with SSH keys [Useful!!!]

When you normally connect to the cluster, you need to type your password each time. SSH keys provide a more secure alternative where you create a pair of keys (like a sophisticated lock and key system):

1. A private key (stays on your computer)
2. A public key (goes on the cluster)

### **Setting Up SSH Key Authentication**

Here’s how to create an SSH key:

1. If you don’t already have one, generate a new SSH key pair:

   ```powershell
   ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_serverA
   ```

   > - `ssh-keygen`：这是生成 SSH 密钥对的命令。密钥对由 **私钥** (存放在你本地机器上) 和 **公钥** (上传到远程服务器/GitHub/GitLab等) 组成。
   > - `-t rsa`：指定密钥类型为 **RSA**（常见的加密算法）。
   > - `-b 4096`：指定密钥长度为 4096 位，比默认的 2048 位更安全。
   > - `-C "your_email"`：给密钥加一个注释，通常写邮箱地址，方便你以后识别这把密钥属于哪个账号。
   > - `-f file_name`：生成指定文件名的密钥，不然会覆盖默认的`id_rsa`
   >
   > 执行后，会在 `~/.ssh/` 目录下生成：
   >
   > - 私钥：`id_rsa`（千万别泄露）
   > - 公钥：`id_rsa.pub`（可以安全上传到服务器、GitHub 等）

2. （Optional) Start the SSH agent and add your key:

    `~/.ssh/id_rsa` 设置passphrase (safer)，每次登录ssh会提示输入密码。此时可以启动`ssh-agent`	

   ```powershell
   eval "$(ssh-agent -s)"  ssh-add ~/.ssh/id_rsa_serverA
   ```

   它会将解锁后的私钥缓存到内存里，之后`ssh`登陆则不需要输入密码。

3. Copy your public key to the cluster:

   ```powershell
   ssh-copy-id -i ~/.ssh/id_rsa_serverA.pub login@gpu-gw.enst.fr
   ```

Now, you should be able to connect without needing a password.

> **Note**: if the ~/ permission is 777, the server assumes other users could have modified files in my home directoy, and SSH will refuses to use the public key to access without password.

## Simplifying Connection (Optional)

You can create an SSH config file to avoid typing the full SSH command every time.

1. Open the SSH config file:

   `vi ~/.ssh/config`

2. Add the following content:

   `Host cluster                      *# This is your shortcut*       HostName gpu-gw.enst.fr      *# The actual server address*      User login                   *# Replace with your username*      IdentityFile ~/.ssh/id_rsa   *# Path to your SSH key*`

This allows you to connect to the cluster with a simple command:

```
ssh cluster
```