---
title: dockerdb
tags:
  - devops
createTime: 2025/06/04 00:37:27
permalink: /article/6n83630c/
---

# 使用 Docker Compose 部署 Postgres 数据库

以下是一个适用于本地开发环境的 Postgres 数据库 docker-compose 配置示例：

```yaml
version: '3.8'
services:
  postgres:
    image: postgres:15
    container_name: postgres_db
    restart: always
    environment:
      POSTGRES_USER: postgres
      POSTGRES_PASSWORD: postgres
      POSTGRES_DB: mydb
    ports:
      - "5432:5432"
    volumes:
      - ./pgdata:/var/lib/postgresql/data
    healthcheck:
      test: ["CMD-SHELL", "pg_isready -U postgres"]
      interval: 10s
      timeout: 5s
      retries: 5
```

## 使用说明

1. 将上述内容保存为 `docker-compose-postgres.yml` 文件。
2. 在该目录下运行：
   ```sh
   docker compose -f docker-compose-postgres.yml up -d
   ```
3. 默认数据库名、用户名和密码均为 `postgres`，数据库名为 `mydb`，可根据需要修改。
4. 数据将持久化到本地 `./pgdata` 目录。
5. 通过 `localhost:5432` 连接数据库。

如需自定义配置，可修改 `environment` 或 `volumes` 等参数。

