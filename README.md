<div align="center">
<img src="https://raw.githubusercontent.com/cooderl/wewe-rss/main/assets/logo.png" width="80" alt="预览"/>

<h1 align="center">WeWe RSS</h1>

更优雅的微信公众号订阅方式。

![主界面](https://raw.githubusercontent.com/cooderl/wewe-rss/main/assets/preview1.png)

</div>

## 功能

- [x]  支持微信公众号订阅（基于微信读书）
- [x]  后台自动定时更新内容
- [x]  微信公众号RSS生成（支持`.atom`\.`rss`\.`json`格式)
- [x]  支持全文内容输出，让阅读无障碍

## 部署

### 一键部署（待完善添加模板）

你可以通过以下平台一键部署，只需填写本项目的URL即可。

[Zeabur](https://zeabur.com/)

[Railway](https://railway.app/)

### Docker Compose 部署

可参考 [docker-compose.yml](https://github.com/cooderl/wewe-rss/blob/main/docker-compose.yml) 和 [docker-compose.sqlite.yml](https://github.com/cooderl/wewe-rss/blob/main/docker-compose.sqlite.yml)

### Docker 命令启动

#### Sqlite

```sh
docker run -d \
  --name wewe-rss \
  -p 4000:4000 \
  -e DATABASE_URL=file:../data/wewe-rss.db \
  -e DATABASE_TYPE=sqlite \
  -e AUTH_CODE=123567 \
  -e FEED_MODE=fulltext \
  -e MAX_REQUEST_PER_MINUTE=60 \
  -e SERVER_ORIGIN_URL=http://localhost:4000 \
  -v $(pwd)/data:/app/data \
  cooderl/wewe-rss-sqlite:latest
```

#### Mysql

1. 创建docker网络

```sh
docker network create wewe-rss
```

2. 启动 MySQL 数据库

```sh
docker run -d \
  --name db \
  -e MYSQL_ROOT_PASSWORD=123456 \
  -e TZ='Asia/Shanghai' \
  -e MYSQL_DATABASE='wewe-rss' \
  -v db_data:/var/lib/mysql \
  --network wewe-rss \
  mysql:latest --default-authentication-plugin=mysql_native_password
```

3. 启动 Server

```sh
docker run -d \
  --name wewe-rss \
  -p 4000:4000 \
  -e DATABASE_URL='mysql://root:123456@db:3306/wewe-rss?schema=public&connect_timeout=30&pool_timeout=30&socket_timeout=30' \
  -e AUTH_CODE=123567 \
  -e FEED_MODE=fulltext \
  -e MAX_REQUEST_PER_MINUTE=60 \
  -e SERVER_ORIGIN_URL="http://localhost:4000" \
  --network wewe-rss \
  cooderl/wewe-rss:latest

```

外网访问时，需将`SERVER_ORIGIN_URL`设置为服务器的公网 IP 或者域名地址

[Nginx配置参考](https://raw.githubusercontent.com/cooderl/wewe-rss/main/assets/nginx.example.conf)

### 本地部署

如果你想本地部署，请使用 `pnpm install &&  pnpm run -r build && pnpm run start:server` 命令(可以配合 pm2 来守护进程，防止被杀死)。

## 使用方式

1. 进入账号管理，点击添加账号，微信扫码登录微信读书账号。
<img width="400" src="./assets/preview2.png"/>

1. 进入公众号源，点击添加，通过提交微信公众号分享链接，订阅微信公众号。
  **（添加频率过高容易被封控，等24小时解封）**
<img width="400" src="./assets/preview3.png"/>

## 环境变量

- `DATABASE_URL` （必填项）数据库地址，例如 `mysql://root:123456@127.0.0.1:3306/wewe-rss`。

- `DATABASE_TYPE` 数据库类型，使用 `sqlite` 需要填写 `sqlite`。

- `AUTH_CODE` （必填项）服务端接口请求授权码，(`/feeds`路径不需要)。

- `SERVER_ORIGIN_URL` （必填项）服务端访问地址，用于RSS生成时使用。

- `MAX_REQUEST_PER_MINUTE` 每分钟最大请求次数，默认 60。

- `FEED_MODE` 输出模式，可选值 `fulltext`（RSS全文模式会使接口响应会变慢，占用更多内存）。

- `CRON_EXPRESSION` 定时更新订阅源Cron表达式，默认为 `35 5,17 * * *`。

## 本地开发

1. 安装 nodejs 18 和 pnpm；
2. 修改环境变量`cp ./apps/web/.env.local.example ./apps/web/.env`和`cp ./apps/server/.env.local.example ./apps/server/.env`
3. 执行 `pnpm install && pnpm dev` 即可。⚠️ 注意：此命令仅用于本地开发，不要用于部署！
4. 前端访问 `http://localhost:5173` ，后端访问 `http://localhost:4000`

## 风险声明

为了确保本项目的持久运行，某些接口请求将通过`weread.111965.xyz`进行转发。请放心，该转发服务不会保存任何数据。

## License
[MIT](https://raw.githubusercontent.com/cooderl/wewe-rss/main/LICENSE) @cooderl