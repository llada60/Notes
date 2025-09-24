## access to private github

生成key：

```bash
ssh-keygen -t rsa -b 4096 -f ~/.ssh/id_rsa_chatgarment
```

启动ssh-agent并加载key：

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa_chatgarment
```

在github仓库里 Settings --> Deploy Keys --> Add deploy key

```bash
cat ~/.ssh/id_rsa_chatgarment.pub
```

将pub key上传上去，测试访问：

```bash
ssh -T git@github.com
```

配置成功则显示 `Hi <your-username>! You've successfully authenticated, but GitHub does not provide shell access.`





每次启动新的terminal：

```shell
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_rsa_chatgarment

//check
ssh -T git@github.com
```

