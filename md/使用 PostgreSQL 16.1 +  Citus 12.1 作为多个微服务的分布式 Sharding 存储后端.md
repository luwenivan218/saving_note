> 本文由 [简悦 SimpRead](http://ksria.com/simpread/) 转码， 原文地址 [juejin.cn](https://juejin.cn/post/7316382839221256192)

![](https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/d6a8352abe774a28a9e9a347db9da804~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=1200&h=570&s=29190&e=png&b=c7d7ea)

在本教程中，我们将使用 `PostgreSQL 16.1` + `Citus 12.1` 作为多个微服务的存储后端，演示此类集群的样例设置和基本操作。

Citus 12.1 实验环境设置
-----------------

### Docker 快速启动 Citus 分布式集群

#### docker-compose.yml

```
version: "3"

services:
  master:
    container_name: "${COMPOSE_PROJECT_NAME:-citus}_master"
    image: "citusdata/citus:12.1.1"
    ports: ["${COORDINATOR_EXTERNAL_PORT:-5432}:5432"]
    labels: ["com.citusdata.role=Master"]
    environment: &AUTH
      POSTGRES_USER: "${POSTGRES_USER:-postgres}"
      POSTGRES_PASSWORD: "${POSTGRES_PASSWORD:-citus}"
      PGUSER: "${POSTGRES_USER:-postgres}"
      PGPASSWORD: "${POSTGRES_PASSWORD:-citus}"
      POSTGRES_HOST_AUTH_METHOD: "${POSTGRES_HOST_AUTH_METHOD:-trust}"
  worker:
    image: "citusdata/citus:12.1.1"
    labels: ["com.citusdata.role=Worker"]
    depends_on: [manager]
    environment: *AUTH
    command: "/wait-for-manager.sh"
    volumes:
      - healthcheck-volume:/healthcheck
  manager:
    container_name: "${COMPOSE_PROJECT_NAME:-citus}_manager"
    image: "citusdata/membership-manager:0.3.0"
    volumes:
      - "${DOCKER_SOCK:-/var/run/docker.sock}:/var/run/docker.sock"
      - healthcheck-volume:/healthcheck
    depends_on: [master]
    environment: *AUTH
volumes:
  healthcheck-volume:



```

#### 启动拥有 3 个 worker 的集群

```
docker-compose -p citus up --scale worker=3


```

![](https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f38a14196209463d9fdd5fca0702d99f~tplv-k3u1fbpfcp-jj-mark:3024:0:0:0:q75.awebp#?w=2238&h=998&s=328414&e=png&b=010101)

#### 查看 worker 节点

```
docker exec -it citus_master psql -U postgres
SELECT master_get_active_worker_nodes();


```

```
 master_get_active_worker_nodes
--------------------------------
 (citus-worker-3,5432)
 (citus-worker-1,5432)
 (citus-worker-2,5432)
(3 rows)


```

微服务的 Citus 存储后端实战
-----------------

### Citus 官方示例源码

[citus-example-microservices](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcitusdata%2Fcitus-example-microservices "https://github.com/citusdata/citus-example-microservices")

*   [github.com/citusdata/c…](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcitusdata%2Fcitus-example-microservices "https://github.com/citusdata/citus-example-microservices")

在我们的示例中，我们将使用三个服务:

*   `user` service
*   `time` service
*   `ping` service

### Distributed schemas(分布式模式)

分布式模式可以在 Citus 集群中重新定位。系统可以在可用节点之间将它们作为一个整体重新平衡，从而允许有效地共享资源，而无需手动分配。

根据设计，微服务拥有自己的存储层，我们不会对它们将创建和存储的表和数据的类型做任何假设。但是，我们将为每个服务提供一个 `schema`，并假设它们将使用不同的 `ROLE` 连接到数据库。当用户连接时，他们的角色名称将放在 `search_path` 的开头，因此，如果 `role` 与 `schema` 名称匹配，则不需要更改任何应用程序来设置正确的 `search_path`。

### 连接到 `Citus` 协调器

```
docker exec -it citus_master psql -U postgres


```

### 为服务创建数据库角色与密码

```
CREATE USER user_service;
CREATE USER time_service;
CREATE USER ping_service;


```

在 Citus 中，模式有两种分布方式:

手动调用 _citus_schema_distribute(schema_name)_ 函数:

```
CREATE SCHEMA AUTHORIZATION user_service;
CREATE SCHEMA AUTHORIZATION time_service;
CREATE SCHEMA AUTHORIZATION ping_service;

SELECT citus_schema_distribute('user_service');
SELECT citus_schema_distribute('time_service');
SELECT citus_schema_distribute('ping_service');


```

此方法还允许将现有的常规模式转换为分布式模式。

> 只能分发不包含分布式表和引用表的模式。

另一种方法是启用 _citus.enable_schema_based_sharding_ 配置变量:

```
SET citus.enable_schema_based_sharding TO ON;

CREATE SCHEMA AUTHORIZATION user_service;
CREATE SCHEMA AUTHORIZATION time_service;
CREATE SCHEMA AUTHORIZATION ping_service;


```

这个变量可以在当前会话中修改，也可以在 `postgresql.conf` 中永久修改。将参数设置为 `ON` 时，默认情况下将分发所有创建的模式。

你可以列出当前分布的模式:

```
select * from citus_schemas;


```

```
 schema_name  | colocation_id | schema_size | schema_owner
--------------+---------------+-------------+--------------
 user_service |             1 | 0 bytes     | user_service
 time_service |             2 | 0 bytes     | time_service
 ping_service |             3 | 0 bytes     | ping_service
(3 rows)


```

### 创建表

现在需要为每个微服务连接到 Citus 协调器。可以使用 `\c` 命令在现有的 _psql_ 实例中切换用户。

```
\c postgres user_service


```

```
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    email VARCHAR(255) NOT NULL
);
\d


```

```
                   List of relations
    Schema    |     Name      |   Type   |    Owner
--------------+---------------+----------+--------------
 public       | citus_schemas | view     | postgres
 public       | citus_tables  | view     | postgres
 user_service | users         | table    | user_service
 user_service | users_id_seq  | sequence | user_service


```

```
\c postgres time_service


```

```
CREATE TABLE query_details (
    id SERIAL PRIMARY KEY,
    ip_address INET NOT NULL,
    query_time TIMESTAMP NOT NULL
);


```

```
\c postgres ping_service


```

```
CREATE TABLE ping_results (
    id SERIAL PRIMARY KEY,
    host VARCHAR(255) NOT NULL,
    result TEXT NOT NULL
);


```

### 配置服务

[citus-example-microservices](https://link.juejin.cn?target=https%3A%2F%2Fgithub.com%2Fcitusdata%2Fcitus-example-microservices "https://github.com/citusdata/citus-example-microservices") 存储库包含 `ping`、`time` 和 `user` 服务。它们都有一个我们要运行的 `app.py`。

```
$ tree
.
├── LICENSE
├── README.md
├── ping
│   ├── app.py
│   ├── ping.sql
│   └── requirements.txt
├── time
│   ├── app.py
│   ├── requirements.txt
│   └── time.sql
└── user
    ├── app.py
    ├── requirements.txt
    └── user.sql


```

为了保障快速测试 `Demo`，这里我为大家提供已经构建好的示例服务的 `docker` 镜像，大家直接用就好。

在构建镜像之前，我已经针对每个服务的 `app.py` 做了一定的修改，如下：

1.  `user/app.py`

```
db_config = {
    'host': 'citus_master',
    'database': 'postgres',
    'user': 'user_service',
    'port': 5432
}
...
app.run(host="0.0.0.0", port=5000)


```

2.  `time/app.py`

```
db_config = {
    'host': 'citus_master',
    'database': 'postgres',
    'user': 'time_service',
    'port': 5432
}
...
app.run(host="0.0.0.0", port=5000)


```

3.  `ping/app.py`

```
db_config = {
    'host': 'citus_master',
    'database': 'postgres',
    'user': 'ping_service',
    'port': 5432
}
...
app.run(host="0.0.0.0", port=5000)


```

当然，你也可以切换到每个 `app` 目录，直接在各自的 `python` 环境中运行它们。

### Docker 快速启动 `user` 服务

```
docker run -d --name usersvc -p 6000:5000 \
         --network=citus_default \
         registry.cn-heyuan.aliyuncs.com/hacker-linner/citus-microsvc-user:1.0.0


```

### Docker 快速启动 `time` 服务

```
docker run -d --name timesvc -p 6001:5000 \
         --network=citus_default \
         registry.cn-heyuan.aliyuncs.com/hacker-linner/citus-microsvc-time:1.0.0


```

### Docker 快速启动 `ping` 服务

```
docker run -d --name pingsvc -p 6002:5000 \
         --network=citus_default \
         registry.cn-heyuan.aliyuncs.com/hacker-linner/citus-microsvc-ping:1.0.0


```

### 创建一些用户

```
curl -X POST -H "Content-Type: application/json" -d '[
  {"name": "John Doe", "email": "john@example.com"},
  {"name": "Jane Smith", "email": "jane@example.com"},
  {"name": "Mike Johnson", "email": "mike@example.com"},
  {"name": "Emily Davis", "email": "emily@example.com"},
  {"name": "David Wilson", "email": "david@example.com"},
  {"name": "Sarah Thompson", "email": "sarah@example.com"},
  {"name": "Alex Miller", "email": "alex@example.com"},
  {"name": "Olivia Anderson", "email": "olivia@example.com"},
  {"name": "Daniel Martin", "email": "daniel@example.com"},
  {"name": "Sophia White", "email": "sophia@example.com"}
]' http://localhost:6000/users


```

```
{"message":"Users created successfully","user_ids":[1,2,3,4,5,6,7,8,9,10]}


```

### 列出已创建的用户

```
curl http://localhost:6000/users


```

### 获取当前时间

```
curl http://localhost:6001/current_time


```

```
{"current_time":"2023-12-25 06:19:28","ip_address":"192.168.65.1"}


```

对 `baidu.com` 执行 `ping` 命令:

```
curl -X POST -H "Content-Type: application/json" -d '{"host": "baidu.com"}' http://localhost:6002/ping


```

```
{"host":"baidu.com","result":"PING baidu.com (110.242.68.66): 56 data bytes\n64 bytes from 110.242.68.66: icmp_seq=0 ttl=63 time=56.996 ms\n64 bytes from 110.242.68.66: icmp_seq=1 ttl=63 time=84.375 ms\n64 bytes from 110.242.68.66: icmp_seq=2 ttl=63 time=99.899 ms\n64 bytes from 110.242.68.66: icmp_seq=3 ttl=63 time=130.946 ms\n--- baidu.com ping statistics ---\n4 packets transmitted, 4 packets received, 0% packet loss\nround-trip min/avg/max/stddev = 56.996/93.054/130.946/26.731 ms\n"}


```

### 探索数据库

现在我们调用了一些 `API` 函数，数据已经存储，我们可以检查 `citus_schemas` 是否反映了我们所期望的:

```
docker exec -it citus_master psql -U postgres
select * from citus_schemas;


```

```
 schema_name  | colocation_id | schema_size | schema_owner 
--------------+---------------+-------------+--------------
 user_service |             1 | 32 kB       | user_service
 time_service |             2 | 32 kB       | time_service
 ping_service |             3 | 32 kB       | ping_service
(3 rows)


```

当我们创建模式时，我们没有告诉 `citus` 在哪些 `worker` 机器上创建 `schema`。它已经自动为我们完成了这些。我们可以通过下面的查询查看每个 `schema` 所在的位置:

```
select nodename,nodeport, table_name, pg_size_pretty(sum(shard_size))
from citus_shards
group by nodename,nodeport, table_name;


```

```
    nodename    | nodeport |         table_name         | pg_size_pretty
----------------+----------+----------------------------+----------------
 citus-worker-1 |     5432 | ping_service.ping_results  | 32 kB
 citus-worker-2 |     5432 | user_service.users         | 32 kB
 citus-worker-3 |     5432 | time_service.query_details | 32 kB
(3 rows)


```

OK! 我们可以看到，各个微服务的后端存储已经默认被分配到不同的 `worker` 机器节点了。