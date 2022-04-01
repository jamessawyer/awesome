常用命令：
```bash
# 下载 并运行
docker run --name my-pg -p 5432:5432 -e POSTGRES_PASSWORD=123456 -d postgres

# 进入bash
docker exec -it my-pg bash

# 登录用户 上面镜像只指定了 POSTGRES_PASSWORD 
# 没有指定 POSTGRES_USER, 则默认用户是 postgres
psql -U postgres

# 或者直接进入psql界面
docker exec -it my-pg psql -U postgres

# 使用psql命令查询
\?
# 创建 test数据库
CREATE DATABASE test;
```
typeorm连接test数据库：
```typescript
import "reflect-metadata"
import { DataSource } from "typeorm"
import { User } from "./entity/User"
import { Photo } from './entity/Photo'
import { PhotoMetadata } from './entity/PhotoMetadata'
import { Author } from "./entity/Author"
import { Album } from './entity/Album'

export const AppDataSource = new DataSource({
    type: "postgres",
    host: "localhost",
    port: 5432,
    username: "postgres",
    password: "123456",
    database: "test",
    synchronize: true,
    logging: false,
    entities: [User, Photo, PhotoMetadata, Author, Album],
    // entities: [__dirname + '/entity/*.ts'], // 直接使用模式匹配
    migrations: [],
    subscribers: [],
})
```
