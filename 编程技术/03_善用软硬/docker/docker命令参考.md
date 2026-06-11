docker run [OPTIONS] IMAGE [COMMAND] [ARG...]

常用参数说明：
- **`-d`**: 后台运行容器并返回容器 ID。
- **`-it`**: 交互式运行容器，分配一个伪终端。
- **`--name`**: 给容器指定一个名称。
- **`-p`**: 端口映射，格式为 `host_port:container_port`。
- **`-v`**: 挂载卷，格式为 `host_dir:container_dir`。
- **`--rm`**: 容器停止后自动删除容器。
- **`--env` 或 `-e`**: 设置环境变量。
- **`--network`**: 指定容器的网络模式。
- **`--restart`**: 容器的重启策略（如 `no`、`on-failure`、`always`、`unless-stopped`）。
- **`-u`**: 指定用户。

清理无用和镜像

docker image prune


-- 背景
docker-compose up -d


--- 如果想先拉取镜像，再手动启动
docker-compose pull    # 只拉取，不启动
docker-compose up -d   # 启动

docker-compose pull web
--  拉取多个服务
docker-compose pull web mysql


docker-compose -f compose.yaml up -d web mysql

docker-compose -f docker-compose-db.yml up -d mysql8