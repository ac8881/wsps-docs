# etcd 部署文档

## 安装和配置 etcd 3.5.14 集群

## 前提条件

确保你有三个服务器（或虚拟机），我们将称它们为 `node1`、`node2` 和 `node3`。假设它们的 IP
地址分别为 `192.168.1.1`，`192.168.1.2` 和 `192.168.1.3`。

## 第一步：在每个节点上安装 etcd

在每个节点上执行以下命令来安装 etcd：

1. **下载 etcd**

```Bash
wget https://github.com/etcd-io/etcd/releases/download/v3.5.14/etcd-v3.5.14-linux-amd64.tar.gz
```

2.**移动到合适的目录**

```Bash
tar -xvf etcd-v3.5.14-linux-amd64.tar.gz
sudo mv etcd-v3.5.14-linux-amd64/etcd* /usr/local/bin/
```

3.**验证安装版本**

```Bash
etcd --version
```

## 第二步：配置 etcd 集群 {id="etcd_1"}

在每个节点上创建一个 etcd 配置文件。我们将使用 `/etc/etcd/etcd.conf.yml` 作为配置文件路径。

### node1 的配置文件

在 `node1` 上创建或编辑 `/etc/etcd/etcd.conf.yml` 文件，内容如下：

```yaml
name: 'node1'
data-dir: '/var/lib/etcd'
initial-advertise-peer-urls: 'http://192.168.1.1:2380'
listen-peer-urls: 'http://192.168.1.1:2380'
advertise-client-urls: 'http://192.168.1.1:2379'
listen-client-urls: 'http://192.168.1.1:2379,http://127.0.0.1:2379'
initial-cluster: 'node1=http://192.168.1.1:2380,node2=http://192.168.1.2:2380,node3=http://192.168.1.3:2380'
initial-cluster-state: 'new'
initial-cluster-token: 'etcd-cluster'
```

在 `node2` 上创建或编辑 `/etc/etcd/etcd.conf.yml` 文件，内容如下：

```yaml
name: 'node2'
data-dir: '/var/lib/etcd'
initial-advertise-peer-urls: 'http://192.168.1.2:2380'
listen-peer-urls: 'http://192.168.1.2:2380'
advertise-client-urls: 'http://192.168.1.2:2379'
listen-client-urls: 'http://192.168.1.2:2379,http://127.0.0.1:2379'
initial-cluster: 'node1=http://192.168.1.1:2380,node2=http://192.168.1.2:2380,node3=http://192.168.1.3:2380'
initial-cluster-state: 'new'
initial-cluster-token: 'etcd-cluster'
```

在 `node3` 上创建或编辑 `/etc/etcd/etcd.conf.yml` 文件，内容如下：

```yaml
name: 'node3'
data-dir: '/var/lib/etcd'
initial-advertise-peer-urls: 'http://192.168.1.3:2380'
listen-peer-urls: 'http://192.168.1.3:2380'
advertise-client-urls: 'http://192.168.1.3:2379'
listen-client-urls: 'http://192.168.1.3:2379,http://127.0.0.1:2379'
initial-cluster: 'node1=http://192.168.1.1:2380,node2=http://192.168.1.2:2380,node3=http://192.168.1.3:2380'
initial-cluster-state: 'new'
initial-cluster-token: 'etcd-cluster'
```

## 第三步 启动ETCD {id="etcd_2"}

直接启动命令：

```Bash
sudo etcd --config-file /etc/etcd/etcd.conf.yml
```

或者创建一个 systemd 服务来管理 etcd：

1. 创建 systemd 服务文件

在每个节点上创建一个 /etc/systemd/system/etcd.service 文件，内容如下：

```Ini
[Unit]
Description=etcd key-value store
Documentation=https://github.com/coreos/etcd
After=network.target

[Service]
User=root
Type=notify
ExecStart=/usr/local/bin/etcd --config-file=/etc/etcd/etcd.conf.yml
Restart=always
RestartSec=5
LimitNOFILE=40000

[Install]
WantedBy=multi-user.target
```

2.启动并启用etcd服务

```Bash
sudo systemctl daemon-reload
sudo systemctl start etcd
sudo systemctl enable etcd
```

## 第四步：验证 etcd 集群 {id="etcd_3"}

```Bash
etcdctl --endpoints=http://192.168.1.1:2379,http://192.168.1.2:2379,http://192.168.1.3:2379 endpoint health
```

将正常看到每个节点的健康状态