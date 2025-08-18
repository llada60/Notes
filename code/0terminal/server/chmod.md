`chmod 777 .`： 允许所有人读写进入（完全开放）

**给某个用户组权限**：需要sudo权限

```powershell
# 把目录归属改为 group=developers
chown -R root:developers /path/to/dir

# 设置组用户可读写
chmod -R 770 /path/to/dir
```