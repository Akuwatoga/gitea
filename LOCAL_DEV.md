# 本地开发说明

这个 fork 已经支持本地基于 SQLite 的开发环境，适合你先在本地快速验证，再推送当前分支触发 ECS 自动构建和部署。

## 本地配置文件

本地使用的配置文件是：

```bash
/Users/akuwatoga/Dev/personal/gitea/custom/conf/app.ini
```

这个配置已经改成了 SQLite，并且会跳过安装页。

## 为什么安装页里看不到 SQLite

SQLite 不是只靠安装页开关控制的，而是要求 `gitea` 二进制本身在编译时带上 SQLite 的 tag。

如果你当前运行的是不带 SQLite 支持的二进制，那么安装页里可能只会显示 MySQL / PostgreSQL，并且即使你手动写 SQLite 配置，启动时也会失败。

## 本地编译 SQLite 版本二进制

本地开发时请使用下面这条命令重新编译：

```bash
cd /Users/akuwatoga/Dev/personal/gitea
GOPROXY=https://goproxy.cn,direct CGO_ENABLED=1 go build -tags 'sqlite sqlite_unlock_notify' -o gitea .
```

注意：

- 普通的 `make build` 可能会把当前二进制重新编译成不带 SQLite 支持的版本。
- 如果你发现本地 SQLite 又不能用了，就重新执行上面的编译命令。

## 启动本地服务

前台启动：

```bash
cd /Users/akuwatoga/Dev/personal/gitea
./gitea web --config /Users/akuwatoga/Dev/personal/gitea/custom/conf/app.ini --port 3000 --install-port 3000
```

后台启动：

```bash
cd /Users/akuwatoga/Dev/personal/gitea
nohup ./gitea web --config /Users/akuwatoga/Dev/personal/gitea/custom/conf/app.ini --port 3000 --install-port 3000 > /tmp/gitea-local.log 2>&1 &
```

启动后访问：

```bash
http://127.0.0.1:3000
```

## 停止本地服务

如果你已经知道进程 PID：

```bash
kill <PID>
```

如果你想先查 PID：

```bash
lsof -nP -iTCP:3000 -sTCP:LISTEN
```

## 本地管理员账号

当前已经创建好的本地管理员账号是：

```text
用户名: localadmin
密码: LocalPass123!
```

如果后面你还想再创建一个管理员账号，可以执行：

```bash
cd /Users/akuwatoga/Dev/personal/gitea
./gitea admin user create --config /Users/akuwatoga/Dev/personal/gitea/custom/conf/app.ini --admin --username <用户名> --password '<密码>' --email <邮箱> --must-change-password=false
```

## 验证本地服务是否启动成功

可以执行：

```bash
curl -I http://127.0.0.1:3000
```

如果启动正常，应该看到：

```text
HTTP/1.1 200 OK
```

## 推荐工作流

1. 如果有需要，先重新编译 SQLite 版本的本地二进制。
2. 启动本地 Gitea，监听 `127.0.0.1:3000`。
3. 在本地验证你的改动。
4. 确认没问题后推送当前分支。
5. 让服务器负责 build 和自动部署。
