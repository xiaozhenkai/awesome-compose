## docker-compose-mysql-cluster

项目结构：

```
.
├── docker-compose.yaml
└── mysql
    ├── master
    │   ├── data
    │   └── my.cnf
    └── slave
        ├── data
        └── my.cnf

```

[_docker-compose.yml_](docker-compose.yml)

```
version: '3'
services:
  mysql-master:
    image: mysql:5.7
    container_name: mysql-master
    environment:
      - MYSQL_ROOT_PASSWORD=root
    ports:
      - "3307:3306"
    volumes:
      - "./mysql/master/my.cnf:/etc/my.cnf"
      - "./mysql/master/data:/var/lib/mysql"
    links:
      - mysql-slave

  mysql-slave:
    image: mysql:5.7
    container_name: mysql-slave
    environment:
      - MYSQL_ROOT_PASSWORD=root
    ports:
      - "3308:3306"
    volumes:
      - "./mysql/slave/my.cnf:/etc/my.cnf"
      - "./mysql/slave/data:/var/lib/mysql"

```

这个compose文件定义了两个数据库，版本是5.7，一个是主库，另一个是从库，主库映射到主机的3307端口，从库映射到主机的3308端口，数据目录分别在`./mysql/master/data`和`./mysql/slave/data`

## 使用 docker-compose 部署

```
$ docker-compose up -d
Creating network "docker-compose-mysql-cluster_default" with the default driver
Creating mysql-slave ... done
Creating mysql-master ... done

```

## 预期的结果

列出容器，正常有会有两个运行容器，端口映射如下：

```
$ docker ps
CONTAINER ID   IMAGE       COMMAND                  CREATED         STATUS         PORTS                                                  NAMES
6c9fa730f8ac   mysql:5.7   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   33060/tcp, 0.0.0.0:3307->3306/tcp, :::3307->3306/tcp   mysql-master
e7aadc854cc4   mysql:5.7   "docker-entrypoint.s…"   2 minutes ago   Up 2 minutes   33060/tcp, 0.0.0.0:3308->3306/tcp, :::3308->3306/tcp   mysql-slave

```





停止并移除容器。 如果要擦除所有数据，请使用 `-v` 参数删除卷(volumes)。

```
$ docker-compose down -v
```

