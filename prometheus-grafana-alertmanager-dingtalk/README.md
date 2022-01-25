## Compose 实例
### Prometheus & Grafana & alertmanager & prometheus-webhook-dingtalk

项目结构:
```
.
├── alertmanager
│   └── alertmanager.yml         #alergmanager的配置文件
├── docker-compose.yml
├── grafana
│   └── datasource.yml
├── output.jpg
├── prometheus
│   ├── prometheus.yml           #prometheus的配置文件
│   └── rules
│       ├── hoststats.alert.yml  #告警规则
│       └── memory.yml           #告警规则
├── prometheus-webhook-dingtalk
│   ├── config.yml               #prometheus-webhook-dingtalk的配置文件
│   └── dingtalk
│       └── template.tmpl        #告警模板
└── README.md

```

docker内部的通讯直接用docker容器的名称来表示，比如alertmanager.xml里需要配置webhook的地址，使用的地址是

```
http://prometheus-webhook-dingtalk:8060/dingtalk/webhook/send
```

这里的`prometheus-webhook-dingtalk`就是容器名称，其他几个容器的名称也可以通过命令`docker ps`进行查看最后一列的`NAMES`就是容器名称，容器内部可以直接通过名称进行交互。

[_docker-compose.yml_](docker-compose.yml)

```
version: "3.7"
services:
  prometheus:
    image: prom/prometheus
    ...
    ports:
      - 9090:9090
  grafana:
    image: grafana/grafana
    ...
  prometheus-webhook-dingtalk:
    image: timonwong/prometheus-webhook-dingtalk
    ...
  alertmanager:
    image: quay.io/prometheus/alertmanager
    ...
    ports:
      - 3000:3000
```
这个 compose 文件定义了一个包含四个服务的堆栈 `prometheus` ,`grafana`,`alertmanager`and `prometheus-webhook-dingtalk`.
部署堆栈时，docker-compose 将每个服务的默认端口映射到主机上的等效端口，以便更轻松地检查每个服务的 Web 界面。
确保服务器上的 **9090** 端口 **3000** 端口、**8060** 端口 和 **9093** 端口没有被使用，可用下面的命令确认:

```
ss -ntulp|grep -E '9090|3000|8060|9093'
或
netstat -ntulp|grep -E '9090|3000|8060|9093'
```



## 使用 docker-compose 启动

```
$ docker-compose up -d
Creating network "prometheus-grafana_default" with the default driver
Creating volume "prometheus-grafana_prom_data" with default driver
...
Creating grafana                     ... done
Creating prometheus-webhook-dingtalk ... done
Creating alertmanager                ... done
Creating prometheus                  ... done

Attaching to prometheus, grafana, alertmanager, prometheus-webhook-dingtalk

```

## 预期输出结果

列出的容器必须显示**四个**正在运行的容器和端口映射，如下：

```
$ docker ps
CONTAINER ID   IMAGE                                  COMMAND                  CREATED         STATUS         PORTS                    NAMES
6091a20315de   prom/prometheus                        "/bin/prometheus --c…"   6 minutes ago   Up 6 minutes   0.0.0.0:9090->9090/tcp   prometheus
101aebe33bee   quay.io/prometheus/alertmanager        "/bin/alertmanager -…"   6 minutes ago   Up 6 minutes   0.0.0.0:9093->9093/tcp   alertmanager
a32127cde56f   timonwong/prometheus-webhook-dingtalk  "/bin/prometheus-web…"   6 minutes ago   Up 6 minutes   0.0.0.0:8060->8060/tcp   prometheus-webhook-dingtalk
6636c57c2e2c   grafana/grafana                        "/run.sh"                6 minutes ago   Up 6 minutes   0.0.0.0:3000->3000/tcp   grafana
```

在 Web 浏览器中打开页面 `http://IP:3000`，把这里的**IP**替换成部署的服务器IP，并使用 compose 文件中指定的登录凭据访问 Grafana。默认用`admin`和`grafana`作为用户名和密码，grafana 已经使用 prometheus 作为默认数据源进行了配置。

![page](output.jpg)

在浏览器中打开 `http://IP:9090` 可以直接访问 prometheus 的 Web 界面。

停止并移除容器。 如果要擦除所有数据，请使用 `-v` 删除卷：
```
$ docker-compose down -v
```



### 技巧

修改Prometheus配置后可以直接用下面命令重新加载配置

```
curl -X POST http://127.0.0.1:9090/-/reload
```



要使用**钉钉机器人**告警需要修改配置`prometheus-webhook-dingtalk/config.yml`

```
  webhook:
    url: https://oapi.dingtalk.com/robot/send?access_token=xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx
    # secret for signature
    secret: xxxxxxxxxxxxxxx
```

把`url`和`secret` 改成钉钉自定义机器人里的`Webhook地址`和安全设置里的`加签`里的密钥。
