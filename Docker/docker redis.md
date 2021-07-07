redis docker:

```bash
# 下载redis image
docker pull redis

# 使用redis运行一个容器 容器名叫 rdb
docker run -d --name rdb -p 6379:6379 redis

# 查看容器
docker ps

# 进入命令行莫属
docker exec -it rdb redis-cli
```

## redis 常用命令

```bash
SET james 23

# 设置KEY过期时间
# https://redis.io/commands/expire
EXPIRE james 10

GET james

# 删除某个key
# https://redis.io/commands/del
DEL james

# 显示所有的key
# https://redis.io/commands/keys
KEYS *

# 显示KEY的过期时间
# https://redis.io/commands/ttl
TTL james
```

