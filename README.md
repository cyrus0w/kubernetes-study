# Kubernetes 入门笔记



```sh
kubernetes-study
| -- docs # 文档
	 | -- Docker.md
	 | -- Linux.md
| -- images
| -- package # 代码包
	 | -- javaproject.zip
	 | -- python-demo.zip
| -- TLS  # 以二进制包方式安装 k8s 所需软件
     | -- etcd
          | -- cfssl_linux-amd64
          | -- cfssl-certinfo_linux-amd64
          | -- cfssljson_linux-amd64
          | -- etcd-v3.4.9-linux-amd64.tar.gz
     | -- k8s
          | -- kube-flannel.yml
          | -- cni-plugins-linux-amd64-v0.8.6.tgz
| -- README.md		# k8s 学习笔记文档
| -- _coverpage.md	# 网页封面文件
| -- index.html		# 网页样式文件
```

![image-20220504095232033](images/image-20220504095232033.png)

## 前置内容

- Linux | [Linux 入门笔记](docs/Linux.md) 
- Docker | [Docker 入门笔记](docs/Docker.md)  

## 目录

- 1  Kubernetes 概念和架构

- 2  从零搭建 Kubernetes 集群

- 3  Kubernetes 核心概念

- 4  搭建集群监控平台系统

- 5  从零搭建高可用 Kubernetess 集群

- 6  在集群环境中部署项目

## 1  Kubernetes 概述和架构

### 1.1 Kubernetes 简介

Kubernetes，首字母 K，尾字母 s，中间 8 个字母，简称 K8s。

### 1.2 Kubernetes 功能

> 目前只需要知道 Kubernetes 有以下 9 个功能，关于这 9 个功能，后面详细介绍。（我也不知道这些是啥玩意，先记住名词再说）

1. 自动装箱

   - 基于容器对应用运行环境的资源配置要求自动部署应用容器

2. 自我修复

   - 当容器失败时，会对容器进行重启

   - 当所部署的 Node 节点有问题时，会对容器进行重新部署和重新调度

   - 当容器未通过监控检查时，会关闭此容器直到容器正常运行时，才会对外提供服务

3. 水平扩展

   - 通过简单的命令、用户 UI 界面或基于 CPU 等资源使用情况，对应用容器进行规模扩大或规模剪裁

   - 当我们有大量的请求来临时，我们可以增加副本数量，从而达到水平扩展的效果

4. 服务发现

   - 用户不需使用额外的服务发现机制，就能够基于 Kubernetes 自身能力实现服务发现和负载均衡

5. 滚动更新
   - 可以根据应用的变化，对应用容器运行的应用，进行一次性或批量式更新

6. 版本回退
   - 可以根据应用部署情况，对应用容器运行的应用，进行历史版本即时回退

7. 密钥和配置管理

   - 在不需要重新构建镜像的情况下，可以部署和更新密钥和应用配置，类似热部署。

8. 存储编排

   - 自动实现存储系统挂载及应用，特别对有状态应用实现数据持久化非常重要

   - 存储系统可以来自于本地目录、网络存储 (NFS、Gluster、Ceph 等）、公共云存储服务

9. 批处理

   - 提供一次性任务，定时任务；满足批量数据处理和分析的场景

### 1.3 Kubernetes 架构组件

**Kuebrnetes 架构图**

> Kubernetes 架构主要包含两部分：Master（主控节点）和 Work node（工作节点）。

---

图 1 

<img src="./images/k8s架构1.png"  >

----

图 2

<img src="./images/k8s架构2.png"  >

-----

图 3

<img src="images/k8s架构3.png"  >

---

**Kubernetes 组件**

- **Master**：主控节点
  - API Server：集群统一入口，以 restful 风格进行操作，同时交给 etcd 存储
    - 提供认证、授权、访问控制、API 注册和发现等机制
  - scheduler：节点的调度，选择 node 节点应用部署
  - controller-manager：处理集群中常规后台任务，一个资源对应一个控制器
  - etcd：存储系统，用于保存集群中的相关数据

- **Worker node**：工作节点
  - Kubelet：master 派到 node 节点代表，管理本机容器
    - 一个集群中每个节点上运行的代理，它保证容器都运行在 Pod 中
    - 负责维护容器的生命周期，同时也负责 Volume(CSI) 和 网络 (CNI) 的管理
  - kube-proxy：提供网络代理，负载均衡等操作
- 容器运行环境【**Container Runtime**】
  - 容器运行环境是负责运行容器的软件
  - Kubernetes 支持多个容器运行环境：Docker、containerd、cri-o、rktlet 以及任何实现 Kubernetes CRI （容器运行环境接口） 的软件。
- fluentd：是一个守护进程，它有助于提升集群层面日志

### 1.4 Kubernetes 核心概念

1. Pod
   - Pod 是 K8s 中最小的单元
   - 一组容器的集合
   - 共享网络【一个 Pod 中的所有容器共享同一网络】
   - 生命周期是短暂的（服务器重启后，就找不到了）

2. Volume
   - 声明在 Pod 容器中可访问的文件目录
   - 可以被挂载到 Pod 中一个或多个容器指定路径下
   - 支持多种后端存储抽象【本地存储、分布式存储、云存储】

3. Controller

   - 确保预期的 pod 副本数量【ReplicaSet】
   - 无状态应用部署【Deployment】
     - 无状态就是指，不需要依赖于网络或者 ip

   - 有状态应用部署【StatefulSet】
     - 有状态需要特定的条件

   - 确保所有的 node 运行同一个 pod 【DaemonSet】

   - 一次性任务和定时任务【Job 和 CronJob】

4. Deployment

   - 定义一组 Pod 副本数目，版本等
   - 通过控制器【Controller】维持 Pod 数目【自动回复失败的 Pod】
   - 通过控制器以指定的策略控制版本【滚动升级、回滚等】

5. Service
   - 定义一组 pod 的访问规则
   - Pod 的负载均衡，提供一个或多个 Pod 的稳定访问地址
   - 支持多种方式【ClusterIP、NodePort、LoadBalancer】

6. Label
   - label：标签，用于对象资源查询，筛选
7. Namespace
   - 命名空间，逻辑隔离
   - 一个集群内部的逻辑隔离机制【鉴权、资源】
   - 每个资源都属于一个 namespace
   - 同一个 namespace 所有资源不能重复
   - 不同 namespace 可以资源名重复

8. API
   - 我们通过 Kubernetes 的 API 来操作整个集群
   - 同时我们可以通过 kubectl 、ui、curl 最终发送 http + json/yaml 方式的请求给 API Server，然后控制整个 K8S 集群，K8S 中所有的资源对象都可以采用 yaml 或 json 格式的文件定义或描述

### 1.5 Kubernetes 工作原理

**Kubernetes 工作原理图**

> 能看懂就看，看不懂就算了，俺也看不懂。

---

<img src="./images/k8s原理1.png">

## 2  从零开始搭建 K8s 集群

### 2.1 基于客户端工具 kubeadm

kubeadm 是官方社区推出的一个用于快速部署 kubernetes 集群的工具。

这个工具能通过两条指令完成一个 kubernetes 集群的部署：

```sh
# 创建一个 Master 节点
kubeadm init

# 将一个 Worker node 节点加入到当前集群中
kubeadm join <Master 节点的 IP 和端口 >
```

#### 2.1.1 安装步骤

使用 kubeadm 方式搭建 Kubernetes 集群主要分为以下几步：

1. 【**环境准备**】准备三台虚拟机，并安装操作系统 CentOS 7.x
2. 【**系统初始化**】对三个刚安装好的操作系统进行初始化操作
3. 【**安装工具**】在三个节点安装 `docker` `kubelet` `kubeadm` `kubectl`
4. 【**集群部署-master**】在 master 节点执行`kubeadm init`命令初始化
5. 【**集群部署-node**】在 node 节点上执行 `kubeadm join`命令，把 node 节点添加到当前集群
6. 【**安装网络插件**】配置 CNI 网络插件，用于节点之间的连通
7. 【**测试集群**】通过拉取一个 nginx 进行测试，能否进行外网测试

####  2.1.2 安装要求

在开始之前，部署 Kubernetes 集群机器需要满足以下几个条件：

- 一台或多台机器，操作系统 CentOS7.x-86_x64
- 硬件配置：2GB 或更多 RAM，2 个 CPU 或更多 CPU，硬盘 30GB 或更多【注意】【注意】【注意】【**master 需要两核**】
- 可以访问外网，需要拉取镜像，如果服务器不能上网，需要提前下载镜像并导入节点
- 禁止 swap 分区

#### 2.1.3  准备环境

> 不会配置环境的可以参考 [Linux 入门笔记 | 虚拟机网络配置](Linux.md)

| 角色       | IP             | 配置  | 操作                                                         |
| ---------- | -------------- | ----- | ------------------------------------------------------------ |
| k8smaster1 | 192.168.60.151 | 2C 2G | `init` `docker` `kubelet` `kubeadm` `kubectl` `kubeadm init` `cni` |
| k8snode1   | 192.168.60.152 | 2C 2G | `init` `docker` `kubelet` `kubeadm` `kubectl` `kubeadm join` |
| k8snode2   | 192.168.60.153 | 2C 2G | `init` `docker` `kubelet` `kubeadm` `kubectl` `kubeadm join` |

#### 2.1.4  系统初始化

【在每台机器上】执行下面的命令：

```sh
# 关闭防火墙
systemctl stop firewalld
# 禁用 firewalld 服务
systemctl disable firewalld

# 关闭 selinux
# 临时关闭【立即生效】告警，不启用，Permissive，查看使用 getenforce 命令
setenforce 0  
# 永久关闭【重启生效】
sed -i 's/SELINUX=enforcing/\SELINUX=disabled/' /etc/selinux/config  

# 关闭 swap
# 临时关闭【立即生效】查看使用 free 命令
swapoff -a 
# 永久关闭【重启生效】
sed -ri 's/.*swap.*/#&/' /etc/fstab

# 在主机名静态查询表中添加 3 台主机
cat >> /etc/hosts << EOF
192.168.60.151 k8smaster
192.168.60.152 k8snode1
192.168.60.153 k8snode2
EOF

# 将桥接的 IPv4 流量传递到 iptables 的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
# 使 k8s 配置生效
sysctl --system  

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com

# 根据规划设置主机名【k8smaster1 节点上操作】
hostnamectl set-hostname ks8master1
# 根据规划设置主机名【k8snode1 节点上操作】
hostnamectl set-hostname k8snode1
# 根据规划设置主机名【k8snode2 节点操作】
hostnamectl set-hostname k8snode2
```

#### 2.1.5  安装组件

【所有节点】需要安装以下组件 ，Kubernetes 默认 CRI（容器运行时）为 Docker，因此先安装 Docker。

- Docker
- kubeadm
- kubelet
- kubectl

**1、安装 Docker**

```sh
# 配置一下 Docker 的 yum 源【阿里云】
cat >/etc/yum.repos.d/docker.repo<<EOF
[docker-ce-edge]
name=Docker CE Edge - \$basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/\$basearch/edge
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
EOF

# 然后 yum 方式安装 docker
yum -y install docker-ce
# 查看 docker 版本
docker --version

# 配置 docker 的镜像源【阿里云】
cat >> /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF

# 启动 docker
systemctl enable docker
systemctl start docker
systemctl status docker
```

**2、安装 kubeadm，kubelet 和 kubectl**

```sh
# 配置 k8s 的 yum 源【阿里云】
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

# 安装 kubelet、kubeadm、kubectl，同时指定版本
yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
# 设置开机自启【这里暂时先不启动 kubelet】
systemctl enable kubelet
```

#### 2.1.6  集群部署【master 节点】

在 `192.168.60.151 ` 上执行【集群初始化命令】，也就是`k8smaster1`节点

```sh
kubeadm init --apiserver-advertise-address=192.168.60.151 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.0 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16
```

> 由于默认拉取镜像地址 k8s.gcr.io 国内无法访问，这里指定阿里云镜像仓库地址，【执行上述命令会比较慢，因为后台其实已经在拉取镜像了】，我们 docker images 命令即可查看已经拉取的镜像。

部署成功后，【系统提示】运行以下命令使用 kubectl

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

执行完成后，我们使用下面命令，查看我们正在运行的节点

```sh
kubectl get nodes
```

#### 2.1.7  集群部署【node 节点】

下面我们需要到 `k8snode1` 和 `k8snode2` 服务器，执行下面的代码向集群添加新节点

执行在 kubeadm init 输出的 kubeadm join 命令：

> 注意，以下的命令是在 k8smaster1 初始化完成后给出的，每个人的都不一样！！！需要复制自己生成的

```sh
kubeadm join 192.168.60.151:6443 --token 8j6ui9.gyr4i156u30y80xf \
    --discovery-token-ca-cert-hash sha256:eda1380256a62d8733f4bddf926f148e57cf9d1a3a58fb45dd6e80768af5a500
```

默认 token 有效期为 24 小时，当过期之后，该 token 就不可用了。这时就需要重新创建 token，操作如下：

```sh
kubeadm token create --print-join-command
```

当我们把两个节点都加入进来后，我们就可以去 `k8smaster1` 节点下 执行下面命令查看情况

```sh
kubectl get nodes
```

#### 2.1.8  部署 CNI 网络插件

上面的状态还是 NotReady，下面我们需要网络插件，来进行联网访问

```sh
# 下载网络插件配置
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
# 添加
kubectl apply -f kube-flannel.yml
# 等一会！
# ......
# 查看状态 【kube-system 是 k8s 中的最小单元】
kubectl get pods -n kube-system
```

运行后的结果为 Ready 状态

【提示】如果上述操作完成后，还存在某个节点处于 NotReady 状态，可以在 Master 将该节点删除

```sh
# 将 k8snode1 节点删除【在 k8smaster1 节点上操作】 
kubectl delete node k8snode1

# 将 k8snode1 节点进行重置【在 k8snode1 节点上操作】
kubeadm reset
# 将 k8snode1 节点加入集群【在 k8snode1 节点上操作】
kubeadm join 192.168.60.151:6443 --token 8j6ui9.gyr4i156u30y80xf     --discovery-token-ca-cert-hash sha256:eda1380256a62d8733f4bddf926f148e57cf9d1a3a58fb45dd6e80768af5a500
```

#### 2.1.9  测试 kubernetes 集群

我们都知道 K8S 是容器化技术，它可以联网去下载镜像，用容器的方式进行启动

在 Kubernetes 集群中创建一个 pod，验证是否正常运行：

```sh
# 下载 nginx 【会联网拉取 nginx 镜像】
kubectl create deployment nginx --image=nginx
# 查看状态
kubectl get pod
```

如果我们出现 Running 状态的时候，表示已经成功运行了

下面我们就需要将端口暴露出去，让其它外界能够访问

```sh
# 暴露端口
kubectl expose deployment nginx --port=80 --type=NodePort
# 查看一下对外的端口
kubectl get pod,svc
```

我这里，已经成功暴露了 80 端口 到 30529 上

我们到我们的宿主机浏览器上，访问如下地址

```sh
http://192.168.60.151:30529/
```

发现我们的 nginx 已经成功启动了

#### 2.1.10  错误汇总

**错误一**

在执行 Kubernetes init 方法的时候，出现这个问题

```
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR NumCPU]: the number of available CPUs 1 is less than the required 2
```

是因为 VMware 设置的核数为 1，而 K8S 需要的最低核数应该是 2，调整核数重启系统即可

**错误二**

我们在给 k8snode1 节点使用 kubernetes join 命令的时候，出现以下错误

```
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR Swap]: running with swap on is not supported. Please disable swap
```

错误原因是我们需要关闭 swap【可能是永久关闭 swap 时没有重启生效】

```sh
# 关闭 swap
# 临时关闭【立即生效】
swapoff -a 
# 永久关闭【重启生效】
sed -ri 's/.*swap.*/#&/' /etc/fstab
```

**错误三**

在给 k8snode1 节点使用 kubernetes join 命令的时候，出现以下错误

```
The HTTP call equal to 'curl -sSL http://localhost:10248/healthz' failed with error: Get http://localhost:10248/healthz: dial tcp [::1]:10248: connect: connection refused
```

解决方法，首先需要到 k8smaster1 节点，创建一个文件

```sh
# 创建文件夹
mkdir /etc/systemd/system/kubelet.service.d

# 创建文件
vim /etc/systemd/system/kubelet.service.d/10-kubeadm.conf

# 添加如下内容
Environment="KUBELET_SYSTEM_PODS_ARGS=--pod-manifest-path=/etc/kubernetes/manifests --allow-privileged=true --fail-swap-on=false"

# 重置
kubeadm reset
```

然后删除刚刚创建的配置目录

```sh
rm -rf $HOME/.kube
```

然后 在 k8smaster1 重新初始化

```sh
kubeadm init --apiserver-advertise-address=92.168.60.151:6443 --image-repository registry.aliyuncs.com/google_containers --kubernetes-version v1.18.0 --service-cidr=10.96.0.0/12  --pod-network-cidr=10.244.0.0/16
```

初始完成后，我们再到 k8snode1 节点，执行 kubeadm join 命令，加入到 k8smaster1【下面这条命令是 k8smaster1 初始化后自动生成的】

```sh
kubeadm join 192.168.60.151:6443 --token c7a7ou.z00fzlb01d76r37s \
    --discovery-token-ca-cert-hash sha256:9c3f3cc3f726c6ff8bdff14e46b1a856e3b8a4cbbe30cab185f6c5ee453aeea5
```

添加完成后，我们使用下面命令，查看节点是否成功添加

```sh
kubectl get nodes
```

**错误四**

我们再执行查看节点的时候， kubectl get nodes 会出现问题

```sh
Unable to connect to the server: x509: certificate signed by unknown authority (possibly because of "crypto/rsa: verification error" while trying to verify candidate authority certificate "kubernetes")
```

这是因为我们之前创建的配置文件还存在，也就是这些配置

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

我们需要做的就是把配置文件删除，然后重新执行一下

```sh
rm -rf $HOME/.kube
```

然后再次创建一下即可

```sh
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

这个问题主要是因为我们在执行 kubeadm reset 的时候，没有把 $HOME/.kube 给移除掉，再次创建时就会出现问题了

**错误五**

安装的时候，出现以下错误

```sh
Another app is currently holding the yum lock; waiting for it to exit...
```

是因为 yum 上锁占用，解决方法

```sh
yum -y install docker-ce
```

**错误六**

在使用下面命令，添加 k8snode1 节点到集群上的时候

```sh
kubeadm join 192.168.60.151:6443 --token jkcz0t.3c40t0bqqz5g8wsb  --discovery-token-ca-cert-hash sha256:bc494eeab6b7bac64c0861da16084504626e5a95ba7ede7b9c2dc7571ca4c9e5
```

然后出现了这个错误

```sh
[root@k8smaster1 ~]# kubeadm join 192.168.60.151:6443 --token jkcz0t.3c40t0bqqz5g8wsb     --discovery-token-ca-cert-hash sha256:bc494eeab6b7bac64c0861da16084504626e5a95ba7ede7b9c2dc7571ca4c9e5
W1117 06:55:11.220907   11230 join.go:346] [preflight] WARNING: JoinControlPane.controlPlane settings will be ignored when control-plane flag is not set.
[preflight] Running pre-flight checks
	[WARNING IsDockerSystemdCheck]: detected "cgroupfs" as the Docker cgroup driver. The recommended driver is "systemd". Please follow the guide at https://kubernetes.io/docs/setup/cri/
error execution phase preflight: [preflight] Some fatal errors occurred:
	[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1
[preflight] If you know what you are doing, you can make a check non-fatal with `--ignore-preflight-errors=...`
To see the stack trace of this error execute with --v=5 or higher
```

出于安全考虑，Linux 系统**默认是禁止数据包转发**的。所谓**转发即当主机拥有多于一块的网卡时，其中一块收到数据包，根据数据包的目的 ip 地址将包发往本机另一网卡，该网卡根据路由表继续发送数据包**。这通常就是路由器所要实现的功能。也就是说 **/proc/sys/net/ipv4/ip_forward** 文件的值不支持转发

- 0：禁止
- 1：转发

所以我们需要将值修改成 1 即可

```sh
echo “1” > /proc/sys/net/ipv4/ip_forward
```

修改完成后，重新执行命令即可

### 2.2  基于二进制方式

参考资料：https://blog.csdn.net/qq_40942490/article/details/114022294

#### 2.2.1 安装步骤

使用二进制包方式搭建 Kubernetes 集群主要分为以下几步：

1. 【**环境准备**】准备三台虚拟机，并安装操作系统 CentOS 7.x
2. 【**系统初始化**】对三个刚安装好的操作系统进行初始化操作
3. 【**部署 etcd 集群**】对三个节点安装 etcd
4. 【**安装 Docker**】对三个节点安装 docker
5. 【**部署 mastber 组件**】在 master 节点上安装`kube-apiserver`、`kube-controller-manager`、`kube-scheduler`
6. 【**部署 node 组件**】在 node 节点上安装`kubelet`、`kube-proxy`
7. 【**安装网络插件**】配置 CNI 网络插件，用于节点之间的连通
8. 【**测试集群**】通过拉取一个 nginx 进行测试，能否进行外网测试

####  2.1.2 安装要求

在开始之前，部署 Kubernetes 集群机器需要满足以下几个条件：

- 一台或多台机器，操作系统 CentOS7.x-86_x64
- 硬件配置：2GB 或更多 RAM，2 个 CPU 或更多 CPU，硬盘 20GB 或更多【注意】【注意】【注意】【**master 需要两核**】
- 可以访问外网，需要拉取镜像，如果服务器不能上网，需要提前下载镜像并导入节点
- 禁止 swap 分区

#### 2.1.3  准备环境

> 不会配置环境的可以参考 [Linux 入门笔记 | 虚拟机 IP 配置]()

| 角色       | IP             | 配置  | 操作                                                         |
| ---------- | -------------- | ----- | ------------------------------------------------------------ |
| k8smaster1 | 192.168.60.151 | 2C 2G | `init` `etcd` `docker` `kube-apiserver` `kube-controller-manager` `kube-scheduler` `cni` |
| k8snode1   | 192.168.60.152 | 2C 2G | `init` `etcd`  `docker` `kubelet` `kube-proxy`               |
| k8snode2   | 192.168.60.153 | 2C 2G | `init`  `etcd` `docker` `kubelet` `kube-proxy`               |

#### 2.2.4  系统初始化

在每台机器上执行下面的命令：

```sh
# 关闭防火墙
systemctl stop firewalld
# 禁用 firewalld 服务
systemctl disable firewalld

# 关闭 selinux
# 临时关闭【立即生效】告警，不启用，Permissive，查看使用 getenforce 命令
setenforce 0  
# 永久关闭【重启生效】
sed -i 's/SELINUX=enforcing/\SELINUX=disabled/' /etc/selinux/config  

# 关闭 swap
# 临时关闭【立即生效】查看使用 free 命令
swapoff -a 
# 永久关闭【重启生效】
sed -ri 's/.*swap.*/#&/' /etc/fstab

# 在主机名静态查询表中添加 3 台主机
cat >> /etc/hosts << EOF
192.168.60.151 k8smaster
192.168.60.152 k8snode1
192.168.60.153 k8snode2
EOF

# 将桥接的 IPv4 流量传递到 iptables 的链
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
# 使 k8s 配置生效
sysctl --system  

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com

# 根据规划设置主机名【k8smaster1 节点上操作】
hostnamectl set-hostname ks8master1
# 根据规划设置主机名【k8snode1 节点上操作】
hostnamectl set-hostname k8snode1
# 根据规划设置主机名【k8snode2 节点操作】
hostnamectl set-hostname k8snode2
```

#### 2.2.5 部署 etcd 集群

Etcd 是一个分布式键值存储系统，Kubernetes 使用 Etcd 进行数据存储，所以先准备一个 Etcd 数据库，为了解决 Etcd 单点故障，应采用集群方式部署，这里使用 3 台组建集群，可容忍一台机器故障，当然也可以使用 5 台组件集群，可以容忍 2 台机器故障。

**1、为 etcd 和 apiserver 自签证书**【k8smaster1 节点操作】

创建工作目录：

```sh
mkdir -p TLS/{etcd,k8s}
cd TLS/etcd/
```

准备 cfssl 证书生成工具：

```sh
# 原地址【下载太慢】 建议迅雷下载
wget https://pkg.cfssl.org/R1.2/cfssl_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssljson_linux-amd64
wget https://pkg.cfssl.org/R1.2/cfssl-certinfo_linux-amd64

# 码云地址【个人上传】
wget https://gitee.com/bbigsun/kubernetes-study/raw/master/TLS/etcd/cfssl_linux-amd64
wget https://gitee.com/bbigsun/kubernetes-study/raw/master/TLS/etcd/cfssljson_linux-amd64
wget https://gitee.com/bbigsun/kubernetes-study/raw/master/TLS/etcd/cfssl-certinfo_linux-amd64

chmod +x cfssl_linux-amd64 cfssljson_linux-amd64 cfssl-certinfo_linux-amd64
mv cfssl_linux-amd64 /usr/local/bin/cfssl
mv cfssljson_linux-amd64 /usr/local/bin/cfssljson
mv cfssl-certinfo_linux-amd64 /usr/local/bin/cfssl-certinfo
```

【使用自签 CA 生成 etcd 证书】

【① 自签 CA】：

```sh
cat > ca-config.json<<EOF
{
    "signing": {
        "default": {
            "expiry": "87600h"
        },
        "profiles": {
            "www": {
                "expiry": "87600h",
                "usages": [
                    "signing",
                    "key encipherment",
                    "server auth",
                    "client auth"
                ]
            }
        }
    }
}
EOF

cat > ca-csr.json<<EOF
{
    "CN": "etcd CA",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "BL": "Beijing"
        }
    ]
}
EOF
```

【② 签发 etcd 证书】：

```sh
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
ls *pem
```

【使用自签 CA 签发 Etcd HTTPS 证书】：

【① 自签 CA】

> 创建证书申请文件：（文件 hosts 字段中 IP 为所有 etcd 节点的集群内部通信 IP，一个都不能少！为了 方便后期扩容可以多写几个预留的 IP）

```sh
cat > server-csr.json << EOF
{
    "CN": "etcd",
    "hosts": [
        "192.168.60.151",
        "192.168.60.152",
        "192.168.60.153"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "name": [
        {
            "C": "CN",
            "L": "Beijing",
            "SL": "Beijing"
        }
    ]
}
EOF
```

【② 签发 etcd https 证书】

```sh
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=www server-csr.json | cfssljson -bare server
ls server*pem
```

**2、部署 etcd**【k8smaster1 节点操作】

从 GitHub 下载二进制文件：

```sh
# 原地址【下载太慢】/ 建议迅雷下载
wget https://github.com/etcd-io/etcd/releases/download/v3.4.9/etcd-v3.4.9-linux-amd64.tar.gz

# 码云地址【个人上传】
wget https://gitee.com/bbigsun/kubernetes-study/raw/master/TLS/etcd/etcd-v3.4.9-linux-amd64.tar.gz
```

安装 etcd：

```sh
mkdir -p /opt/etcd/{bin,cfg,ssl} 
tar -zxvf etcd-v3.4.9-linux-amd64.tar.gz
mv etcd-v3.4.9-linux-amd64/{etcd,etcdctl} /opt/etcd/bin/
cp ~/TLS/etcd/ca*pem ~/TLS/etcd/server*pem /opt/etcd/ssl/
```

创建配置文件：

```sh
cat > /opt/etcd/cfg/etcd.conf << EOF
#[Member]
ETCD_NAME="etcd-1"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.60.151:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.60.151:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.60.151:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.60.151:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.60.151:2380,etcd-2=https://192.168.60.152:2380,etcd-3=https://192.168.60.153:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF

# 名词解释
# ETCD_NAME：节点名称，集群中唯一
# ETCD_DATA_DIR：数据目录
# ETCD_LISTEN_PEER_URLS：集群通信监听地址
# ETCD_LISTEN_CLIENT_URLS：客户端访问监听地址
# ETCD_INITIAL_ADVERTISE_PEER_URLS：集群通告地址
# ETCD_ADVERTISE_CLIENT_URLS：客户端通告地址
# ETCD_INITIAL_CLUSTER：集群节点地址
# ETCD_INITIAL_CLUSTER_TOKEN：集群 Token
# ETCD_INITIAL_CLUSTER_STATE：加入集群的当前状态，new 是新集群，existing 表示加入 已有集群
```

创建 etcd.service：

```sh
cat > /usr/lib/systemd/system/etcd.service << EOF
[Unit]
Description=Etcd Server
After=network.target
After=network-online.target
Wants=network-online.target
[Service]
Type=notify
EnvironmentFile=/opt/etcd/cfg/etcd.conf
ExecStart=/opt/etcd/bin/etcd \
--cert-file=/opt/etcd/ssl/server.pem \
--key-file=/opt/etcd/ssl/server-key.pem \
--peer-cert-file=/opt/etcd/ssl/server.pem \
--peer-key-file=/opt/etcd/ssl/server-key.pem \
--trusted-ca-file=/opt/etcd/ssl/ca.pem \
--peer-trusted-ca-file=/opt/etcd/ssl/ca.pem \
--logger=zap
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
```

【k8smaster1 配置完毕！】

**3、转发 etcd 到 node 节点**【k8smaster1 节点上操作】【需要输入密码，建议密码设置简单一点】

```sh
###### 转发到 k8snode1 ######
scp -r /opt/etcd/ root@192.168.60.152:/opt/
scp -r /usr/lib/systemd/system/etcd.service root@192.168.60.152:/usr/lib/systemd/system/
###### 转发到 k8snode2 ######
scp -r /opt/etcd/ root@192.168.60.153:/opt/
scp -r /usr/lib/systemd/system/etcd.service root@192.168.60.153:/usr/lib/systemd/system/
```

**4、修改 node 节点上 etcd 的配置文件：IP 和名字**【k8snode1 和 k8snode2 节点上操作】

```sh
##### k8sndoe1 上操作 #####
cat > /opt/etcd/cfg/etcd.conf << EOF
#[Member]
ETCD_NAME="etcd-2"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.60.152:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.60.152:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.60.152:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.60.152:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.60.151:2380,etcd-2=https://192.168.60.152:2380,etcd-3=https://192.168.60.153:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF

##### k8sndoe2 上操作 #####
cat > /opt/etcd/cfg/etcd.conf << EOF
#[Member]
ETCD_NAME="etcd-3"
ETCD_DATA_DIR="/var/lib/etcd/default.etcd"
ETCD_LISTEN_PEER_URLS="https://192.168.60.153:2380"
ETCD_LISTEN_CLIENT_URLS="https://192.168.60.153:2379"
#[Clustering]
ETCD_INITIAL_ADVERTISE_PEER_URLS="https://192.168.60.153:2380"
ETCD_ADVERTISE_CLIENT_URLS="https://192.168.60.153:2379"
ETCD_INITIAL_CLUSTER="etcd-1=https://192.168.60.151:2380,etcd-2=https://192.168.60.152:2380,etcd-3=https://192.168.60.153:2380"
ETCD_INITIAL_CLUSTER_TOKEN="etcd-cluster"
ETCD_INITIAL_CLUSTER_STATE="new"
EOF
```

启动并设置开机启动：【k8snode1 和 k8snode2 均需启动】

```sh
systemctl daemon-reload
systemctl start etcd
systemctl enable etcd
```

查看集群状态：

```sh
/opt/etcd/bin/etcdctl --cacert=/opt/etcd/ssl/ca.pem --cert=/opt/etcd/ssl/server.pem --key=/opt/etcd/ssl/server-key.pem --endpoints="https://192.168.60.151:2379,https://192.168.60.152:2379,https://192.168.60.153:2379" endpoint status --write-out=table
```

#### 2.2.6 安装 docker

在所有节点操作。这里采用二进制安装，用 yum 安装也一样 （多台节点安装可以采用键盘工具）

```sh
cd ~/TLS
wget https://download.docker.com/linux/static/stable/x86_64/docker-20.10.3.tgz
tar -zxvf docker-20.10.3.tgz
mv docker/ /usr/bin
```

systemd 管理 docker：

```sh
cat > /usr/lib/systemd/system/docker.service << EOF
[Unit]
Description=Docker Application Container Engine
Documentation=https://docs.docker.com
After=network-online.target firewalld.service
Wants=network-online.target
[Service]
Type=notify
ExecStart=/usr/bin/dockerd
ExecReload=/bin/kill -s HUP $MAINPID
LimitNOFILE=infinity
LimitNPROC=infinity
LimitCORE=infinity
TimeoutStartSec=0
Delegate=yes
KillMode=process
Restart=on-failure
StartLimitBurst=3
StartLimitInterval=60s
[Install]
WantedBy=multi-user.target
EOF
```

配置阿里云加速：

```sh
mkdir /etc/docker
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
```

启动并设置开机启动：

```sh
systemctl daemon-reload
systemctl start docker
systemctl enable docker
systemctl status docker
```

【k8smaster1 节点安装 docker 完毕！转发到 k8snode1 和 k8snode2 节点】【k8smaster1 节点上操作】

```sh
##### 转发到 k8snode1 #####
scp -r /usr/bin/docker/ root@192.168.60.152:/usr/bin/
scp -r /usr/lib/systemd/system/docker.service root@192.168.60.152:/usr/lib/systemd/system/
scp -r /etc/docker/ root@192.168.60.152:/etc/
##### 转发到 k8snode2 #####
scp -r /usr/bin/docker/ root@192.168.60.153:/usr/bin/
scp -r /usr/lib/systemd/system/docker.service root@192.168.60.153:/usr/lib/systemd/system/
scp -r /etc/docker/ root@192.168.60.153:/etc/
```

#### 2.2.7  部署 master 组件

- kube-apiserver
- kuber-controller-manager
- kube-scheduler

**1、安装 kube-apiserver**

【生成 kube-apiserver 证书】

【① 自签证书颁发机构 CA】：

```sh
cd ~/TLS/k8s
```

```sh
cat > ca-config.json << EOF
{
  "signing": {
    "default": {
      "expiry": "87600h"
    },
    "profiles": {
      "kubernetes": {
         "expiry": "87600h",
         "usages": [
            "signing",
            "key encipherment",
            "server auth",
            "client auth"
        ]
      }
    }
  }
}
EOF
```

```sh
cat > ca-csr.json << EOF
{
    "CN": "kubernetes",
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "Beijing",
            "ST": "Beijing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF
```

【② 生成 kube-apiserver 证书】：

```sh
cfssl gencert -initca ca-csr.json | cfssljson -bare ca -
ls *pem
```

【使用自签 CA 签发 kube-apiserver HTTPS 证书】

【① 创建证书申请文件】：

```sh
cat > server-csr.json << EOF
{
    "CN": "kubernetes",
    "hosts": [
      "10.0.0.1",
      "127.0.0.1",
      "192.168.60.151",
      "192.168.60.152",
      "192.168.60.153",
      "kubernetes",
      "kubernetes.default",
      "kubernetes.default.svc",
      "kubernetes.default.svc.cluster",
      "kubernetes.default.svc.cluster.local"
    ],
    "key": {
        "algo": "rsa",
        "size": 2048
    },
    "names": [
        {
            "C": "CN",
            "L": "BeiJing",
            "ST": "BeiJing",
            "O": "k8s",
            "OU": "System"
        }
    ]
}
EOF
```

【② 生成 kube-apiserver https 证书】：

```sh
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes server-csr.json | cfssljson -bare server
ls server*pem
```

【安装 kube-apiserver】

下载二进制包：

```sh
# 下载地址：https://github.com/kubernetes/kubernetes/blob/master/CHANGELOG/CHANGELOG-1.20.md
# kubernetes-server-linux-amd64.tar.gz 包含了 master 和 node 的所有组件
# 这里提供几个下载地址
wget https://storage.googleapis.com/kubernetes-release/release/v1.20.1/kubernetes-server-linux-amd64.tar.gz
wget https://dl.k8s.io/v1.19.0/kubernetes-server-linux-amd64.tar.gz
```

解压二进制包：

```sh
mkdir -p /opt/kubernetes/{bin,cfg,ssl,logs}
tar -zxvf kubernetes-server-linux-amd64.tar.gz
cd kubernetes/server/bin
cp kube-apiserver kube-scheduler kube-controller-manager /opt/kubernetes/bin
cp kubectl /usr/bin/
```

生成 kube-apiserver 配置文件：

```sh
cat > /opt/kubernetes/cfg/kube-apiserver.conf << EOF
KUBE_APISERVER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--etcd-servers=https://192.168.60.151:2379,https://192.168.60.152:2379,https://192.168.60.153:2379 \\
--bind-address=192.168.60.151 \\
--secure-port=6443 \\
--advertise-address=192.168.60.151 \\
--allow-privileged=true \\
--service-cluster-ip-range=10.0.0.0/24 \\
--enable-admission-plugins=NamespaceLifecycle,LimitRanger,ServiceAccount,ResourceQuota,NodeRestriction \\
--authorization-mode=RBAC,Node \\
--enable-bootstrap-token-auth=true \\
--token-auth-file=/opt/kubernetes/cfg/token.csv \\
--service-node-port-range=30000-32767 \\
--kubelet-client-certificate=/opt/kubernetes/ssl/server.pem \\
--kubelet-client-key=/opt/kubernetes/ssl/server-key.pem \\
--tls-cert-file=/opt/kubernetes/ssl/server.pem  \\
--tls-private-key-file=/opt/kubernetes/ssl/server-key.pem \\
--client-ca-file=/opt/kubernetes/ssl/ca.pem \\
--service-account-key-file=/opt/kubernetes/ssl/ca-key.pem \\
--etcd-cafile=/opt/etcd/ssl/ca.pem \\
--etcd-certfile=/opt/etcd/ssl/server.pem \\
--etcd-keyfile=/opt/etcd/ssl/server-key.pem \\
--audit-log-maxage=30 \\
--audit-log-maxbackup=3 \\
--audit-log-maxsize=100 \\
--audit-log-path=/opt/kubernetes/logs/k8s-audit.log"
EOF

# 注：上面两个、\ 第一个是转义符，第二个是换行符，使用转义符是为了使用 EOF 保留换 行符。
# –logtostderr：启用日志
# —v：日志等级
# –log-dir：日志目录
# –etcd-servers：etcd 集群地址
# –bind-address：监听地址
# –secure-port：https 安全端口
# –advertise-address：集群通告地址
# –allow-privileged：启用授权
# –service-cluster-ip-range：Service 虚拟 IP 地址段
# –enable-admission-plugins：准入控制模块
# –authorization-mode：认证授权，启用 RBAC 授权和节点自管理
# –enable-bootstrap-token-auth：启用 TLS bootstrap 机制
# –token-auth-file：bootstrap token 文件
# –service-node-port-range：Service nodeport 类型默认分配端口范围
# –kubelet-client-xxx：apiserver 访问 kubelet 客户端证书
# –tls-xxx-file：apiserver https 证书
# –etcd-xxxfile：连接 Etcd 集群证书
# –audit-log-xxx：审计日志
```

把刚生成的证书拷贝到配置文件中的路径：

```sh
cp ~/TLS/k8s/ca*pem ~/TLS/k8s/server*pem /opt/kubernetes/ssl/
```

创建上述文件配置文件中的 token 文件：

```sh
cat > /opt/kubernetes/cfg/token.csv << EOF
c47ffb939f5ca36231d9e3121a252940,kubelet-bootstrap,10001,"system:node-bootstrapper"
EOF
```

>  格式：token，用户名，UID，用户组 token 也可自行生成替换【建议暂时不要替换，直接 copy 代码就完事了】：

```sh
head -c 16 /dev/urandom | od -An -t x | tr -d ' '
```

systemd 管理 apiserver：

```sh
cat > /usr/lib/systemd/system/kube-apiserver.service << EOF
[Unit]
Description=Kubernetes API Server
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kube-apiserver.conf
ExecStart=/opt/kubernetes/bin/kube-apiserver \$KUBE_APISERVER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

启动并设置开机启动：

```sh
systemctl daemon-reload
systemctl start kube-apiserver
systemctl enable kube-apiserver
systemctl status kube-apiserver 
```

授权 kubelet-bootstrap 用户允许请求证书：

```sh
kubectl create clusterrolebinding kubelet-bootstrap \
--clusterrole=system:node-bootstrapper \
--user=kubelet-bootstrap
```

**2、部署 kube-controller-manager**

```sh
cat > /opt/kubernetes/cfg/kube-controller-manager.conf << EOF
KUBE_CONTROLLER_MANAGER_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--leader-elect=true \\
--master=127.0.0.1:8080 \\
--bind-address=127.0.0.1 \\
--allocate-node-cidrs=true \\
--cluster-cidr=10.244.0.0/16 \\
--service-cluster-ip-range=10.0.0.0/24 \\
--cluster-signing-cert-file=/opt/kubernetes/ssl/ca.pem \\
--cluster-signing-key-file=/opt/kubernetes/ssl/ca-key.pem  \\
--root-ca-file=/opt/kubernetes/ssl/ca.pem \\
--service-account-private-key-file=/opt/kubernetes/ssl/ca-key.pem \\
--experimental-cluster-signing-duration=87600h0m0s"
EOF

# –master：通过本地非安全本地端口 8080 连接 apiserver。
# –leader-elect：当该组件启动多个时，自动选举（HA）
# –cluster-signing-cert-file/–cluster-signing-key-file：自动为 kubelet 颁发证书的 CA，与 apiserver 保持一致
```

systemd 管理 controller-manager：

```sh
cat > /usr/lib/systemd/system/kube-controller-manager.service << EOF
[Unit]
Description=Kubernetes Controller Manager
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kube-controller-manager.conf
ExecStart=/opt/kubernetes/bin/kube-controller-manager \$KUBE_CONTROLLER_MANAGER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

启动并设置开机启动：

```sh
systemctl daemon-reload
systemctl start kube-controller-manager
systemctl enable kube-controller-manager
systemctl status kube-controller-manager
```

**3、部署 kube-scheduler**

```sh
cat > /opt/kubernetes/cfg/kube-scheduler.conf << EOF
KUBE_SCHEDULER_OPTS="--logtostderr=false \
--v=2 \
--log-dir=/opt/kubernetes/logs \
--leader-elect \
--master=127.0.0.1:8080 \
--bind-address=127.0.0.1"
EOF

# 参数说明
# –master：通过本地非安全本地端口 8080 连接 apiserver。
# –leader-elect：当该组件启动多个时，自动选举（HA）
```

```sh
cat > /usr/lib/systemd/system/kube-scheduler.service << EOF
[Unit]
Description=Kubernetes Scheduler
Documentation=https://github.com/kubernetes/kubernetes
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kube-scheduler.conf
ExecStart=/opt/kubernetes/bin/kube-scheduler \$KUBE_SCHEDULER_OPTS
Restart=on-failure
[Install]
WantedBy=multi-user.target
EOF
```

启动并设置开机启动：

```sh
systemctl daemon-reload
systemctl start kube-scheduler
systemctl enable kube-scheduler
systemctl status kube-scheduler
```

**4、查看集群状态**

所有组件都已经启动成功，通过 kubectl 工具查看当前集群组件状态：

```sh
kubectl get cs
```

#### 2.2.8 部署 node 组件

- kubelet
- kube-proxy

**1、安装 kubelet**

```sh
##### k8snode1 节点上操作 #####
mkdir -p /opt/kubernetes/{bin,cfg,ssl,logs}
```

```sh
cat > /opt/kubernetes/cfg/kubelet.conf << EOF
KUBELET_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--hostname-override=m1 \\
--network-plugin=cni \\
--kubeconfig=/opt/kubernetes/cfg/kubelet.kubeconfig \\
--bootstrap-kubeconfig=/opt/kubernetes/cfg/bootstrap.kubeconfig \\
--config=/opt/kubernetes/cfg/kubelet-config.yml \\
--cert-dir=/opt/kubernetes/ssl \\
--pod-infra-container-image=lizhenliang/pause-amd64:3.0"
EOF

# –hostname-override：显示名称，集群中唯一
# –network-plugin：启用 CNI
# –kubeconfig：空路径，会自动生成，后面用于连接 apiserver
# –bootstrap-kubeconfig：首次启动向 apiserver 申请证书
# –config：配置参数文件
# –cert-dir：kubelet 证书生成目录
# –pod-infra-container-image：管理 Pod 网络容器的镜像
```

```sh
cat > /opt/kubernetes/cfg/kubelet-config.yml << EOF
kind: KubeletConfiguration
apiVersion: kubelet.config.k8s.io/v1beta1
address: 0.0.0.0
port: 10250
readOnlyPort: 10255
cgroupDriver: cgroupfs
clusterDNS:
- 10.0.0.2
clusterDomain: cluster.local 
failSwapOn: false
authentication:
  anonymous:
    enabled: false
  webhook:
    cacheTTL: 2m0s
    enabled: true
  x509:
    clientCAFile: /opt/kubernetes/ssl/ca.pem 
authorization:
  mode: Webhook
  webhook:
    cacheAuthorizedTTL: 5m0s
    cacheUnauthorizedTTL: 30s
evictionHard:
  imagefs.available: 15%
  memory.available: 100Mi
  nodefs.available: 10%
  nodefs.inodesFree: 5%
maxOpenFiles: 1000000
maxPods: 110
EOF
```

将 k8smaster1 节点的 bin 文件和证书拷贝到 k8snode1 和 k8snode2 节点上【k8smaster1 节点操作】：

```sh
cd ~/TLS/k8s/kubernetes/server/bin

##### 转发到 k8snode1 #####
scp -r {kubelet,kube-proxy} root@192.168.60.152:/opt/kubernetes/bin/
scp -r /usr/bin/kubectl root@192.168.60.152:/usr/bin/
scp -r /opt/kubernetes/ssl root@192.168.60.152:/opt/kubernetes

##### 转发到 k8snode2 #####
scp -r /opt/kubernetes/ssl root@192.168.60.153:/opt/kubernetes
```

生成 bootstrap.kubeconfig 文件：
```sh
# apiserver IP:PORT
KUBE_APISERVER="https://192.168.60.151:6443" 
# 与 token.csv 里保持一致
TOKEN="c47ffb939f5ca36231d9e3121a252940" 
```

生成 kubelet bootstrap kubeconfig 配置文件：

```sh
kubectl config set-cluster kubernetes \
  --certificate-authority=/opt/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=bootstrap.kubeconfig
kubectl config set-credentials "kubelet-bootstrap" \
  --token=${TOKEN} \
  --kubeconfig=bootstrap.kubeconfig
kubectl config set-context default \
  --cluster=kubernetes \
  --user="kubelet-bootstrap" \
  --kubeconfig=bootstrap.kubeconfig
kubectl config use-context default --kubeconfig=bootstrap.kubeconfig

mv bootstrap.kubeconfig /opt/kubernetes/cfg
```

systemd 管理 kubelet：

```sh
cat > /usr/lib/systemd/system/kubelet.service << EOF
[Unit]
Description=Kubernetes Kubelet
After=docker.service
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kubelet.conf
ExecStart=/opt/kubernetes/bin/kubelet \$KUBELET_OPTS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
```

启动并设置开机启动：

```sh
systemctl daemon-reload
systemctl start kubelet
systemctl enable kubelet
systemctl status kubelet
```

批准 kubelet 证书申请并加入集群【k8smaster1 节点操作】：

```sh
# 查看 kubelet 证书请求
kubectl get csr

###    输出结果
###    NAME                                                   AGE    SIGNERNAME                                    REQUESTOR           CONDITION
###    node-csr-uCEGPOIiDdlLODKts8J658HrFq9CZ--K6M4G7bjhk8A   6m3s   kubernetes.io/kube-apiserver-client-kubelet   kubelet-bootstrap   Pending

# 批准申请
kubectl certificate approve node-csr-uCEGPOIiDdlLODKts8J658HrFq9CZ--K6M4G7bjhk8A

# 查看节点
kubectl get node
```

注：由于网络插件还没有部署，节点会没有准备就绪 NotReady

**2、部署 kube-proxy**

```sh
cat > /opt/kubernetes/cfg/kube-proxy.conf << EOF
KUBE_PROXY_OPTS="--logtostderr=false \\
--v=2 \\
--log-dir=/opt/kubernetes/logs \\
--config=/opt/kubernetes/cfg/kube-proxy-config.yml"
EOF
```

```sh
cat > /opt/kubernetes/cfg/kube-proxy-config.yml << EOF
kind: KubeProxyConfiguration
apiVersion: kubeproxy.config.k8s.io/v1alpha1
bindAddress: 0.0.0.0
metricsBindAddress: 0.0.0.0:10249
clientConnection:
  kubeconfig: /opt/kubernetes/cfg/kube-proxy.kubeconfig
hostnameOverride: m1
clusterCIDR: 10.0.0.0/24
EOF
```

生成 kube-proxy.kubeconfig 文件【**k8smaster1 生成再传到 k8snode1 和 k8snode2**】：

```sh
# 切换工作目录
cd ~/TLS/k8s
```

```sh
# 创建证书请求文件
cat > kube-proxy-csr.json << EOF
{
  "CN": "system:kube-proxy",
  "hosts": [],
  "key": {
    "algo": "rsa",
    "size": 2048
  },
  "names": [
    {
      "C": "CN",
      "L": "BeiJing",
      "ST": "BeiJing",
      "O": "k8s",
      "OU": "System"
    }
  ]
}
EOF
```

```sh
# 生成证书
cfssl gencert -ca=ca.pem -ca-key=ca-key.pem -config=ca-config.json -profile=kubernetes kube-proxy-csr.json | cfssljson -bare kube-proxy
```

```sh
# 生成 kubeconfig 文件
KUBE_APISERVER="https://192.168.60.151:6443"
```

```sh
kubectl config set-cluster kubernetes \
  --certificate-authority=/opt/kubernetes/ssl/ca.pem \
  --embed-certs=true \
  --server=${KUBE_APISERVER} \
  --kubeconfig=kube-proxy.kubeconfig
kubectl config set-credentials kube-proxy \
  --client-certificate=./kube-proxy.pem \
  --client-key=./kube-proxy-key.pem \
  --embed-certs=true \
  --kubeconfig=kube-proxy.kubeconfig
kubectl config set-context default \
  --cluster=kubernetes \
  --user=kube-proxy \
  --kubeconfig=kube-proxy.kubeconfig
kubectl config use-context default --kubeconfig=kube-proxy.kubeconfig

##### 转发到 k8snode1 #####
scp -r kube-proxy.kubeconfig root@192.168.60.152:/opt/kubernetes/cfg/
##### 转发到 k8snode2 #####
scp -r kube-proxy.kubeconfig root@192.168.60.153:/opt/kubernetes/cfg/
```

systemd 管理 kube-proxy：

```sh
cat > /usr/lib/systemd/system/kube-proxy.service << EOF
[Unit]
Description=Kubernetes Proxy
After=network.target
[Service]
EnvironmentFile=/opt/kubernetes/cfg/kube-proxy.conf
ExecStart=/opt/kubernetes/bin/kube-proxy \$KUBE_PROXY_OPTS
Restart=on-failure
LimitNOFILE=65536
[Install]
WantedBy=multi-user.target
EOF
```

启动并设置开机启动：

```sh
systemctl daemon-reload
systemctl start kube-proxy
systemctl enable kube-proxy
systemctl status kube-proxy
```

#### 2.2.9  部署 CNI 网络插件

下载 CNI 网络插件：

```sh
# 原地址
wget https://github.com/containernetworking/plugins/releases/download/v0.8.6/cni-plugins-linux-amd64-v0.8.6.tgz
# 码云地址【个人上传】【推荐】【速度较快】
wget https://gitee.com/bbigsun/kubernetes-study/raw/master/TLS/k8s/cni-plugins-linux-amd64-v0.8.6.tgz
```

安装插件：

```sh
mkdir -p /opt/cni/bin
tar -zxvf cni-plugins-linux-amd64-v0.8.6.tgz -C /opt/cni/bin
```

【k8smaster1 节点操作】：

```sh
wget https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
kubectl apply -f kube-flannel.yml
```

#### 2.2.10  测试 kubernetes 集群

在 Kubernetes 集群中创建一个 pod，验证是否正常运行【master 节点操作】：

```sh
# 下载 nginx 【会联网拉取 nginx 镜像】
kubectl create deployment nginx --image=nginx
# 查看状态
kubectl get pod
```

如果我们出现 Running 状态的时候，表示已经成功运行了

下面我们就需要将端口暴露出去，让其它外界能够访问

```sh
# 暴露端口
kubectl expose deployment nginx --port=80 --type=NodePort
# 查看一下对外的端口
kubectl get pod,svc
```

能够看到，我们已经成功暴露了 80 端口 到 30529 上

我们到我们的宿主机浏览器上，访问如下地址

```sh
http://192.168.177.130:30529/
```

发现我们的 nginx 已经成功启动了

### 2.3  两种方式搭建集群的对比

#### 2.3.1 Kubeadm 方式搭建 K8S 集群

- 安装虚拟机，在虚拟机安装 Linux 操作系统【3 台虚拟机】
- 对操作系统初始化操作
- 所有节点安装 Docker、kubeadm、kubelet、kubectl【包含 master 和 node 节点】
  - 安装 Docker、使用 yum，不指定版本默认安装最新的 Docker 版本
  - 修改 Docker 仓库地址，yum 源地址，改为阿里云地址
  - 安装 kubeadm，kubelet 和 kubectl
    - k8s 已经发布最新的 1.19 版本，可以指定版本安装，不指定安装最新版本
    - `yum install -y kubelet kubeadm kubectl`
- 在 master 节点执行初始化命令操作
  - `kubeadm init`
  - 默认拉取镜像地址 K8s.gcr.io 国内地址，需要使用国内地址
- 安装网络插件 (CNI)
  - `kubectl apply -f kube-flannel.yml`
- 在所有的 node 节点上，使用 join 命令，把 node 添加到 master 节点上
- 测试 kubernetes 集群

#### 2.3.2 二进制方式搭建 K8S 集群

- 安装虚拟机和操作系统，对操作系统进行初始化操作
- 生成 cfssl 自签证书
  - `ca-key.pem`、`ca.pem`
  - `server-key.pem`、`server.pem`
- 部署 Etcd 集群
  - 部署的本质，就是把 etcd 集群交给 systemd 管理
  - 把生成的证书复制过来，启动，设置开机启动
- 安装 Docker
- 部署 master 组件，主要包含以下组件
  - apiserver
  - controller-manager
  - scheduler
  - 交给 systemd 管理，并设置开机启动
  - 如果要安装最新的 1.19 版本，下载二进制文件进行安装
- 部署 node 组件
  - kubelet
  - kube-proxy【需要批准 kubelet 证书申请加入集群】
  - 交给 systemd 管理组件- 组件启动，设置开机启动
- 批准 kubelet 证书申请 并加入集群
- 部署 CNI 网络插件
- 测试 Kubernets 集群【安装 nginx 测试】

## 3  Kubernetes 核心概念

### 3.1 kubernetes 集群命令行工具 kubectl

#### 3.1.1 kubectl 概述

kubectl 是 Kubernetes 集群的命令行工具，通过 kubectl 能够对集群本身进行管理，并能够在集群上进行容器化应用的安装和部署。

#### 3.1.2 kubectl 命令格式

```
kubectl [command] [type] [name] [flags]
```

参数：

- command：指定要对资源执行的操作，例如 create、get、describe、delete

- type：指定资源类型，资源类型是大小写敏感的，开发者能够以单数 、复数 和 缩略的形式

  ```sh
  kubectl get pod pod1
  kubectl get pods pod1
  kubectl get po pod1
  ```

- name：指定资源的名称，名称也是大小写敏感的，如果省略名称，则会显示所有的资源，例如

  ```sh
  kubectl get pods
  ```

- flags：指定可选的参数，例如，可用 -s 或者 -server 参数指定 Kubernetes API server 的地址和端口

#### 3.1.3 kubectl 帮助命令

```sh
# 获取 kubectl 的命令
kubectl --help

# 获取某个命令的介绍和使用
kubectl get --help
kubectl create --help
```

#### 3.1.4 kubectl 基础命令

| 命令    | 介绍                                           |
| ------- | ---------------------------------------------- |
| create  | 通过文件名或标准输入创建资源                   |
| expose  | 将一个资源公开为一个新的 Service                |
| run     | 在集群中运行一个特定的镜像                     |
| set     | 在对象上设置特定的功能                         |
| get     | 显示一个或多个资源                             |
| explain | 文档参考资料                                   |
| edit    | 使用默认的编辑器编辑一个资源                   |
| delete  | 通过文件名，标准输入，资源名称或标签来删除资源 |

#### 3.1.5 kubectl 部署命令

| 命令           | 介绍                                               |
| -------------- | -------------------------------------------------- |
| rollout        | 管理资源的发布                                     |
| rolling-update | 对给定的复制控制器滚动更新                         |
| scale          | 扩容或缩容 Pod 数量，Deployment、ReplicaSet、RC 或 Job |
| autoscale      | 创建一个自动选择扩容或缩容并设置 Pod 数量            |

#### 3.1.6 kubectl 集群管理命令

| 命令         | 介绍                           |
| ------------ | ------------------------------ |
| certificate  | 修改证书资源                   |
| cluster-info | 显示集群信息                   |
| top          | 显示资源 (CPU/M)                |
| cordon       | 标记节点不可调度               |
| uncordon     | 标记节点可被调度               |
| drain        | 驱逐节点上的应用，准备下线维护 |
| taint        | 修改节点 taint 标记              |

#### 3.1.7 kubectl 故障和调试命令

| 命令         | 介绍                                                         |
| ------------ | ------------------------------------------------------------ |
| describe     | 显示特定资源或资源组的详细信息                               |
| logs         | 在一个 Pod 中打印一个容器日志，如果 Pod 只有一个容器，容器名称是可选的 |
| attach       | 附加到一个运行的容器                                         |
| exec         | 执行命令到容器                                               |
| port-forward | 转发一个或多个                                               |
| proxy        | 运行一个 proxy 到 Kubernetes API Server                         |
| cp           | 拷贝文件或目录到容器中                                       |
| auth         | 检查授权                                                     |

#### 3.1.8 kubectl 其它命令

| 命令         | 介绍                                                |
| ------------ | --------------------------------------------------- |
| apply        | 通过文件名或标准输入对资源应用配置                  |
| patch        | 使用补丁修改、更新资源的字段                        |
| replace      | 通过文件名或标准输入替换一个资源                    |
| convert      | 不同的 API 版本之间转换配置文件                       |
| label        | 更新资源上的标签                                    |
| annotate     | 更新资源上的注释                                    |
| completion   | 用于实现 kubectl 工具自动补全                         |
| api-versions | 打印受支持的 API 版本                                 |
| config       | 修改 kubeconfig 文件（用于访问 API，比如配置认证信息） |
| help         | 所有命令帮助                                        |
| plugin       | 运行一个命令行插件                                  |
| version      | 打印客户端和服务版本信息                            |

### 3.2  Kubernetes 集群 YAML 文件详解

参考资料：[YAML 入门教程 | 菜鸟教程](https://www.runoob.com/w3cnote/yaml-intro.html)

#### 3.2.1 YAML 概述

- YAML 文件 : 就是资源清单文件，用于资源编排。
- YAML : 仍是一种标记语言。为了强调这种语言以数据做为中心，而不是以标记语言为重点。
- YAML : 是一个可读性高，用来表达数据序列的格式。

#### 3.2.2 YAML 基本语法

- 使用空格做为缩进
- 缩进的空格数目不重要，只要相同层级的元素左侧对齐即可
- 低版本缩进时不允许使用 Tab 键，只允许使用空格
- 使用#标识注释，从这个字符一直到行尾，都会被解释器忽略
- 使用 --- 表示新的 yaml 文件开始

#### 3.2.3 YAML 数据结构

对象：键值对的集合，又称为映射 (mapping) / 哈希（hashes） / 字典（dictionary）

```
# 对象类型：对象的一组键值对，使用冒号结构表示
name: Tom
age: 18

# yaml 也允许另一种写法，将所有键值对写成一个行内对象
hash: {name: Tom, age: 18}
```

数组：

```
# 数组类型：一组连词线开头的行，构成一个数组
People
- Tom
- Jack

# 数组也可以采用行内表示法
People: [Tom, Jack]
```

####  3.2.4 YAML 组成部分

主要分为了两部分，一个是控制器的定义 和 被控制的对象。

在一个 YAML 文件的控制器定义中，有很多属性名称

| 属性名称   | 介绍       |
| ---------- | ---------- |
| apiVersion | API 版本    |
| kind       | 资源类型   |
| metadata   | 资源元数据 |
| spec       | 资源规格   |
| replicas   | 副本数量   |
| selector   | 标签选择器 |
| template   | Pod 模板    |
| metadata   | Pod 元数据  |
| spec       | Pod 规格    |
| containers | 容器配置   |

#### 3.2.5 YAML 快速编写

一般来说，我们很少自己手写 YAML 文件，因为这里面涉及到了很多内容，我们一般都会借助工具来创建

**1、使用 kubectl create 命令**

这种方式一般用于资源没有部署的时候，我们可以直接创建一个 YAML 配置文件

```sh
# 尝试运行，并不会真正的创建镜像
kubectl create deployment web --image=nginx -o yaml --dry-run
```

或者我们可以输出到一个文件中

```sh
kubectl create deployment web --image=nginx -o yaml --dry-run > hello.yaml
```

然后我们就在文件中直接修改即可

**2、使用 kubectl get 命令导出 yaml 文件**

可以首先查看一个目前已经部署的镜像

```sh
kubectl get deploy
```

然后我们导出 nginx 的配置

```sh
kubectl get deploy nginx -o=yaml --export > nginx.yaml
```

然后会生成一个 `nginx.yaml` 的配置文件

### 3.3 Pod

#### 3.3.1 Pod 概述

**1、Pod 基本概念**

- 最小部署的单元
- Pod 里面是由一个或多个容器组成【一组容器的集合】
- 一个 pod 中的容器是共享网络命名空间
- Pod 是短暂的
- 每个 Pod 包含一个或多个紧密相关的用户业务容器

**2、Pod 存在的意义**

- 创建容器使用 docker，一个 docker 对应一个容器，一个容器运行一个应用进程
- Pod 是多进程设计，运用多个应用程序，也就是一个 Pod 里面有多个容器，而一个容器里面运行一个应用程序
- Pod 的存在是为了亲密性应用
  - 两个应用之间进行交互
  - 网络之间的调用【通过 127.0.0.1 或 socket】
  - 两个应用之间需要频繁调用

**3、k8s 业务类型**

>  Pod 是 K8S 集群中所有业务类型的基础，可以把 Pod 看作运行在 K8S 集群上的小机器人，不同类型的业务就需要不同类型的小机器人去执行。目前 K8S 的业务主要可以分为以下几种

- 长期伺服型：long-running
- 批处理型：batch
- 节点后台支撑型：node-daemon
- 有状态应用型：stateful application

上述的几种类型，分别对应的小机器人控制器为：Deployment、Job、DaemonSet 和 StatefulSet （后面将介绍控制器）

#### 3.3.2  Pod 实现机制

> Pod 主要有以下两大机制：共享网络 和 共享存储。

**1、共享网络**【容器通过 **namespace** 和 **group** 进行隔离】

Pod 中容器通信 过程：

- 同一个 namespace 下
- 在 Pod 中创建一个根容器： `pause 容器`
- 在 Pod 中创建业务容器 【nginx，redis 等】【创建时会添加到 `info 容器` 中】
- 在 `info 容器` 中会独立出 ip 地址，mac 地址，port 等信息，然后实现网络的共享

**2、共享存储**【Pod 持久化数据，专门存储到某个地方中，使用 Volumn 数据卷进行共享存储】

#### 3.3.3  Pod 镜像拉取策略

> 我们以具体实例来说，拉取策略就是 `imagePullPolicy`

拉取策略主要分为了以下几种：

- `IfNotPresent`：默认值，镜像在宿主机上不存在才拉取
- `Always`：每次创建 Pod 都会重新拉取一次镜像
- `Never`：Pod 永远不会主动拉取这个镜像

#### 3.3.4  Pod 资源限制

> 也就是我们 Pod 在进行调度的时候，可以对调度的资源进行限制，例如我们限制 Pod 调度是使用的资源是 2C4G，那么在调度对应的 node 节点时，只会占用对应的资源，对于不满足资源的节点，将不会进行调度。

这里分了两个部分：

- `request`：表示调度所需的资源
- `limits`：表示最大所占用的资源

#### 3.3.5  Pod 重启机制

> 因为 Pod 中包含了很多个容器，假设某个容器出现问题了，那么就会触发 Pod 重启机制

重启策略主要分为以下三种：

- `Always`：当容器终止退出后，总是重启容器，默认策略 【nginx 等，需要不断提供服务】
- `OnFailure`：当容器异常退出（退出状态码非 0）时，才重启容器。
- `Never`：当容器终止退出，从不重启容器 【批量任务】

#### 3.3.6  Pod 健康检查

**1、通过容器检查**

```sh
kubectl get pod
```

**2、通过应用检查**

> 但是有的时候，程序可能出现了 **Java** 堆内存溢出，程序还在运行，但是不能对外提供服务了，这个时候就不能通过容器检查来判断服务是否可用了。需要通过应用检查。

```sh
# 存活检查，如果检查失败，将杀死容器，根据 Pod 的 restartPolicy【重启策略】来操作
livenessProbe

# 就绪检查，如果检查失败，Kubernetes 会把 Pod 从 Service endpoints 中剔除
readinessProbe
```

Probe 支持以下三种检查方式

- `http Get`：发送 HTTP 请求，返回 200 - 400 范围状态码为成功
- `exec`：执行 Shell 命令返回状态码是 0 为成功
- `tcpSocket`：发起 TCP Socket 建立成功

#### 3.3.7 Pod 调度策略

 创建 Pod 流程：

- 首先创建一个 pod，然后创建一个 API Server 和 Etcd【把创建出来的信息存储在 etcd 中】
- 然后创建 Scheduler，监控 API Server 是否有新的 Pod，如果有的话，会通过调度算法，把 pod 调度某个 node 上
- 在 node 节点，会通过 `kubelet -- apiserver `读取 etcd 拿到分配在当前 node 节点上的 pod，然后通过 docker 创建容器

### 3.2  Controller

#### 3.2.1 Controller 内容简介

- 什么是 Controler
- Pod 和 Controller 的关系
- Deployment 控制器应用场景
- yaml 文件字段说明
- Deployment 控制器部署应用
- 升级回滚
- 弹性收缩

#### 3.2.2 Controller 概述

> Controller 是集群上管理和运行容器的对象

- Controller 是实际存在的
- Pod 是虚拟机的

#### 3.2.3 Pod 和 Controller 的关系

Pod 是通过 Controller 实现应用的运维，比如弹性收缩，滚动升级。

Pod 和 Controller 之间是通过 label 标签建立关系，同时 Controller 又被称为控制器工作负载。

- Controller【控制器】【工作负载】`selector: app:nginx`
- Pod【容器】`labels: app:nginx`

#### 3.2.4 Deployment 控制器应用

> Deployment 表示用户对 K8S 集群的一次更新操作。

- Deployment 控制器可以部署无状态应用
- 管理 Pod 和 ReplicaSet
- 部署，滚动升级等功能
- 应用场景：web 服务，微服务

#### 3.2.5 Deployment 部署应用

之前，使用 Deploment 部署应用，代码如下：【缺点：代码不好复用】

```sh
kubectrl create deployment web --image=nginx
```

现在，使用 YAML 文件进行配置：【快速编写 YAML 文件】

```sh
kubectl create deployment web --image=nginx -o yaml --dry-run > nginx.yaml
```

`nginx.yaml`  文件内容如下：【`selector `和 `label` 就是我们 Pod 和 Controller 之间建立关系的桥梁】

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  # Pod
  labels:
    app: web
  name: web
spec:
  replicas: 1
  # Controller
  selector:
    matchLabels:
      app: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      - image: nginx
        name: nginx
        resources: {}
status: {}
```

现在，使用`nginx.yaml`文件创建镜像：

```sh
kubectl apply -f nginx.yaml
```

然后，对外暴露端口：

```sh
kubectl expose deployment web --port=80 --type=NodePort --target-port=80 --name=web1

# 参数说明
# --port：就是我们内部的端口号
# --target-port：就是暴露外面访问的端口号
# --name：名称
# --type：类型
```

同理，导出配置文件：

```sh
kubectl expose deployment web --port=80 --type=NodePort --target-port=80 --name=web1 -o yaml > web1.yaml
```

查看端口：

```sh
kubectl get pods,svc

# 输出结果
NAME                       READY   STATUS    RESTARTS   AGE
pod/web-5dcb957ccc-d89v9   1/1     Running   0          8m35s

NAME                 TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1       <none>        443/TCP        2d5h
service/web1         NodePort    10.111.61.143   <none>        80:30344/TCP   6s
```

然后我们访问对应的 url，即可看到 nginx 了 `http://192.168.60.151:30344/`

<img src="./images/nginx.png" style="zoom:150%;" >

#### 3.2.6 升级回滚和弹性收缩

- 升级： 假设从版本为 1.14 升级到 1.15 ，这就叫应用的升级【升级可以保证服务不中断】
- 回滚：从版本 1.15 变成 1.14，这就叫应用的回滚
- 弹性伸缩：我们根据不同的业务场景，来改变 Pod 的数量对外提供服务，这就是弹性伸缩

**1、创建一个 1.14 版本的 pod**

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  creationTimestamp: null
  labels:
    app: web
  name: web
spec:
  replicas: 1
  selector:
    matchLabels:
      app: web
  strategy: {}
  template:
    metadata:
      creationTimestamp: null
      labels:
        app: web
    spec:
      containers:
      # 修改 nginx 版本 1.14
      - image: nginx:1.14
        name: nginx
        resources: {}
status: {}
```

```sh
kubectl apply -f nginx.yaml
```

**2、应用升级**

```sh
kubectl set image deployment web nginx=nginx:1.15
```

升级过程：

```sh
[root@k8smaster ~]# kubectl set image deployment web nginx=nginx:1.15
deployment.apps/web image updated

# 首先是开始的 nginx 1.14 版本的 Pod 在运行，然后 1.15 版本的在创建
[root@k8smaster ~]# kubectl get pod
NAME                   READY   STATUS              RESTARTS   AGE
web-66bf4959f5-qhzsd   1/1     Running             0          52s
web-bbcf684cb-bbmqv    0/1     ContainerCreating   0          3s

# 然后在 1.15 版本创建完成后，就会暂停 1.14 版本
[root@k8smaster ~]# kubectl get pod
NAME                   READY   STATUS        RESTARTS   AGE
web-66bf4959f5-qhzsd   1/1     Terminating   0          67s
web-bbcf684cb-bbmqv    1/1     Running       0          18s

# 最后把 1.14 版本的 Pod 移除，完成我们的升级
[root@k8smaster ~]# kubectl get pod
NAME                  READY   STATUS    RESTARTS   AGE
web-bbcf684cb-bbmqv   1/1     Running   0          33s
```

> 我们在下载 1.15 版本，容器就处于 ContainerCreating 状态，然后下载完成后，就用 1.15 版本去替换 1.14 版本了，这么做的好处就是：升级可以保证服务不中断

**3、查看升级状态**

```sh
kubectl rollout status deployment web
```

**4、查看历史版本**

```sh
kubectl rollout history deployment web
```

**5、应用回滚**

```sh
# 回滚到上一版本
kubectl rollout undo deployment web

# 回滚到指定版本
kubectl rollout undo deployment web --to-revision=2
```

**6、弹性伸缩**

```sh
# 通过命令创建多个副本
kubectl scale deployment web --replicas=10

# 输出结果，等一会就会全部 Running
[root@k8smaster ~]# kubectl scale deployment web --replicas=10
deployment.apps/web scaled
[root@k8smaster ~]# kubectl get pod
NAME                  READY   STATUS              RESTARTS   AGE
web-bbcf684cb-2f2zl   0/1     ContainerCreating   0          4s
web-bbcf684cb-72pzr   0/1     ContainerCreating   0          4s
web-bbcf684cb-bbmqv   1/1     Running             0          3m9s
web-bbcf684cb-fgpgh   0/1     ContainerCreating   0          4s
web-bbcf684cb-fpk8d   0/1     ContainerCreating   0          4s
web-bbcf684cb-hqp4z   0/1     ContainerCreating   0          4s
web-bbcf684cb-htq2d   0/1     ContainerCreating   0          4s
web-bbcf684cb-lnkwx   0/1     ContainerCreating   0          4s
web-bbcf684cb-vmwb9   0/1     ContainerCreating   0          4s
web-bbcf684cb-vnk5w   0/1     ContainerCreating   0          4s
```

### 3.3  Kubernetes 配置管理

#### 3.3.1 Secret

> Secret 的主要作用就是加密数据

1、Secret 应用场景

​	对 用户名 和 密码 进行加密

2、Secret 三种类型

- `Opaque`：使用 base64 编码存储信息，可以通过 base64 --decode 解码获得原始数据，因此安全性弱。
- `kubernetes.io/dockerconfigjson`：用于存储 docker registry 的认证信息。
- `kubernetes.io/service-account-token`：用于被 serviceaccount 引用。serviceaccout 创建时 Kubernetes 会默认创建对应的 secret。Pod 如果使用了 serviceaccount，对应的 secret 会自动挂载到 Pod 的 /run/secrets/kubernetes.io/serviceaccount 目录中。

3、Secret 创建

（1）命令行方式创建 Secret

```sh
echo -n "admin" > ./username.txt
echo -n "1f1f1f1f1f" > ./password.txt

# 使用 kubectl create secret 命令创建 secret
kubectl create secret generic db-user-pass --from-file=./username.txt --from-file=./password.txt
#  secret/db-user-pass created

# 查看 secret
kubectl get secrets
#  NAME                  TYPE                                  DATA   AGE
#  db-user-pass          Opaque                                2      59s
```

（2）yaml 文件方式创建 Secret

```sh
echo -n 'admin' | base64
#  YWRtaW4=
echo -n '1f1f1f1f1f' | base64
#  MWYxZjFmMWYxZg==

# 创建 secret：创建 yaml 文件
cat > secret.yaml << EOF
apiVersion: v1
kind: Secret
metadata:
  name: mysecret
type: Opaque
data:
  username: YWRtaW4=
  password: MWYxZjFmMWYxZg==
EOF

# 创建 secret：使用 yaml 文件创建 secret
kubectl create -f secret.yaml
#  secret/mysecret created

# 查看 secret
kubectl get secrets | grep mysecret
#  mysecret              Opaque                                2      32s

# 查看 secret 详细信息
kubectl describe secrets mysecret
# 查看 secret yaml 文件
kubectl get secrets mysecret -o yaml
```

4、Secret 使用【两种方式】

- 以 Volume 形式
- 以环境变量形式

（1）将 Secret 挂载到 Volume 中

```sh
cat > mypod1.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: mypod1
spec:
  containers:
  - name: mypod1
    image: redis
    volumeMounts:
    - name: foo
      mountPath: "/etc/foo"
      readOnly: true
  volumes:
  - name: foo
    secret:
      secretName: mysecret
EOF

kubectl create -f mypod1.yaml
#  pod/mypod1 created
kubectl get pods | grep mypod
#  mypod1                1/1     Running   0          48s
kubectl exec -it mypod1 /bin/bash

## 查看密码和用户名
root@mypod1:/data# cd /etc/foo/
root@mypod1:/etc/foo# ls
password  username
root@mypod1:/etc/foo# cat password 
1f1f1f1f1f
root@mypod1:/etc/foo# cat username 
admin
root@mypod1:/etc/foo# 
```

（2）将 Secret 设置为环境变量

```sh
cat > mypod2.yaml << EOF
apiVersion: v1
kind: Pod
metadata:
  name: mypod2
spec:
  containers:
  - name: mypod2
    image: redis
    env:
      - name: SECRET_USERNAME
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: username
      - name: SECRET_PASSWORD
        valueFrom:
          secretKeyRef:
            name: mysecret
            key: password
  restartPolicy: Never
EOF
  
kubectl create -f mypod2.yaml
#  pod/mypod2 created
kubectl get pods | grep mypod
#  mypod1                1/1     Running             0          4m39s
#  mypod2                0/1     ContainerCreating   0          6s
#  等   mypod2    running   之后在进入容器
kubectl exec -it mypod2 /bin/bash

## 查看环境变量
root@mypod2:/data# env | grep -E "USERNAME|PASSWORD"
SECRET_USERNAME=admin
SECRET_PASSWORD=1f1f1f1f1f
```

#### 3.3.2 ConfigMap

> ConfigMap 作用是存储不加密的数据到 etcd 中

1、应用场景

​	配置文件

2、创建

（1）yaml 文件方式创建

```sh
cat > configmap-test01.yaml << EOF
apiVersion: v1
kind: ConfigMap
metadata:
  name: cm-test01
data:
  appconf01: value01
  appconf02: value02
EOF

kubectl create -f configmap-test01.yaml
```

（2）命令行方式创建

> 读取文件方式（也可以是目录）通过`--from-file`参数从文件中读取。可以指定 key 的名称，若不指定，则默认使用文件名为 key。

```sh
cat > test.properties << EOF
key01:value01
key02:value02
conf01: value03
EOF

kubectl create cm cm-test-file --from-file=test.properties
```

3、查询

```sh
# 查看 configmap 列表
kubectl get configmap
# 查看 configmap 详情
kubectl describe configmap cm-test01
kubectl describe configmap cm-test-file
kubectl describe cm cm-test-literal
# 查看 yaml 输出
kubectl get cm cm-test01 -o yaml
kubectl get configmap cm-test-file -o yaml
kubectl get cm cm-test-literal -o yaml
```

4、更新

```sh
# 方式一：edit
kubectl edit cm cm-test01
# 查看更新是否生效
kubectl describe cm cm-test01
# 方式二：apply
kubectl apply -f configmap-test01.yaml
```

5、删除

```sh
# 方式一：通过 yaml 文件删除
kubectl delete -f configmap-test01.yaml
# 方式二：直接删除资源
kubectl delete cm cm-test-file
```

6、使用 【yaml 文件有误，以下四种方式无误】

容器应用对 ConfigMap 的使用主要是两种：

- 通过环境变量获取 ConfigMap 的内容：`spec.env`和`spec.envFrom`
- 通过卷 volume 挂载的方式将 ConfigMap 的内容挂载到容器内部的文件或目录：`spec.volumes`

（1）`spec.env` 【环境变量】

```sh
vim pod-test01.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-pod-test001
spec:
  containers:
  - name: cm-test
    image: tomcat:8
    command: [ "/bin/sh", "-c", "env | grep APP"]
    env:
    - name: APPCONF01 		# 定义环境变量的名称
      valueFrom:	  		# key “appconf01”的值获取
        configMapKeyRef:
          name: cm-test01	# 环境变量的值来自于 configmap cm-test01
          key: appconf01	# configmap 中的配置 key 为 appconf01
    - name: APPCONF02		# 定义环境变量的名称
      valueFrom:			# key “appconf02”的值获取
        configMapKeyRef: 
          name: cm-test01	# 环境变量的值来自于 configmap cm-test01
          key: appconf02	# configmap 中的配置 key 为 appconf02
  restartPolicy: Never		# 重启策略：从不。

kubectl create -f pod-test01.yaml
kubectl get pods | grep cm
kubectl logs cm-pod-test001
```

（2）`spec.envFrom` 【环境变量】

```sh
vim pod-test02.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-pod-test002
spec:
  containers:
  - name: cm-test2
    image: tomcat:8
    command: [ "/bin/sh", "-c", "env"]
    envFrom:
    - configMapRef:
      name: cm-test01	# 根据 ConfigMap cm-test01 资源自动生成环境变量
  restartPolicy: Never

kubectl create -f pod-test02.yaml
kubectl get po
```

（3）指定 items【卷挂载方式】

```sh
vim pod-test03.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-pod-test003
spec:
  containers:
  - name: cm-test3
    image: tomcat:8
    volumeMounts:
    - name: vm-01-1
      mountPath: /conf
  volumes:
  - name: vm-01-1
    configMap:
      name: cm-test-file
      items:
      - key: key-testproperties
        path: test.properties
  restartPolicy: Never

kubectl create -f pod-test03.yaml
kubectl get po
```

（4）不指定 items【卷挂载方式】

```sh
vim pod-test04.yaml
apiVersion: v1
kind: Pod
metadata:
  name: cm-pod-test004
spec:
  containers:
  - name: cm-test4
    image: tomcat:8
    volumeMounts:
    - name: vm-02-2
      mountPath: /conf
  volumes:
  - name: vm-02-2
    configMap:
      name: cm-test-file
  restartPolicy: Never

kubectl create -f pod-test04.yaml
kubectl get po

# 进入容器查看
kubectl exec -it cm-pod-test004 -c cm-test4 -- bash
root@cm-pod-test004:/usr/local/tomcat# ls /conf
```

### 3.4  Kubernetes 集群安全机制

#### 3.4.1 API-SERVER

> Kubernetes api-server 安全访问机制

当我们访问 K8S 集群时，都需要经过 apiserver【 apiserver 做统一协调】，每个请求到达 apiserver 需要经过三个安全关卡：`① 认证` `② 鉴权 ` `③ 准入控制`

- 访问过程中，需要证书、token、或者用户名和密码
- 如果访问 pod 需要 serviceAccount

<img src="./images/api-server.png">

**1、认证**

对外不暴露 8080 端口，只能内部访问，对外使用的端口 6443

客户端身份认证常用方式

- https 证书认证，基于 ca 证书
- http token 认证，通过 token 来识别用户
- http 基本认证，用户名 + 密码认证

**2、鉴权**

基于 RBAC 进行鉴权操作

基于角色访问控制

**3、准入控制**

就是准入控制器的列表，如果列表有请求内容就通过，没有的话 就拒绝

#### 3.4.2 TLS

> Kubernetes 认证方式之客户端证书（TLS）

客户端证书（TLS）认证方式，也叫 HTTPS 双向认证。一般我们访问一个 https 网站，认证是单向的，只有客户端会验证服务端的身份，服务端不会管客户端身份如何。

#### 3.4.3 RBAC 介绍

> Kubernetes 授权方式之 RBAC 
>
> 基于角色的访问控制，为某个角色设置访问内容，然后用户分配该角色后，就拥有该角色的访问权限

k8s 中有默认的几个角色

- role：特定命名空间访问权限
- ClusterRole：所有命名空间的访问权限

角色绑定

- roleBinding：角色绑定到主体
- ClusterRoleBinding：集群角色绑定到主体

主体

- user：用户
- group：用户组
- serviceAccount：服务账号

#### 3.4.4 RBAC 鉴权

1、创建命名空间

```sh
# 查看已经存在的命名空间
kubectl get namespace
# 创建自己的命名空间
kubectl create ns mytest
```

2、命名空间内创建 Pod

> 如果不创建命名空间，Pod 默认在 default

```sh
kubectl run nginx --image=nginx -n mytest
```

3、创建角色

> 通过 rbac-role.yaml 进行创建
>
> tips: 这个角色只对 pod 有 get 和 list 权限

```sh
cat > rbac-role.yaml << EOF
kind: Role
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: mytest
  name: pod-reader
rules:
- apiGroups: [""] # "" indicates the core API group
  resources: ["pods"]
  verbs: ["get", "watch", "list"]
EOF
```

通过 yaml 创建 role

```sh
# 创建
kubectl apply -f rbac-role.yaml
# 查看
kubectl get role -n mytest
```

4、创建角色绑定

通过 rbac-rolebinding.yaml 的方式，来创建我们的角色绑定

```sh
cat > rbac-rolebinding.yaml << EOF
kind: RoleBinding
apiVersion: rbac.authorization.k8s.io/v1
metadata:
  namespace: mytest
  name: read-pods
subjects:
- kind: User
  name: lucy
  apiGroup: rbac.authorization.k8s.io
roleRef: 
  kind: Role
  name: pod-reader
  apiGroup: rbac.authorization.k8s.io
EOF

kubectl apply -f rbac-rolebinding.yaml
kubectl get role,rolebinding -n mytest

#  NAME                                        CREATED AT
#  role.rbac.authorization.k8s.io/pod-reader   2022-01-04T03:05:57Z
#  NAME                                              ROLE              AGE
#  rolebinding.rbac.authorization.k8s.io/read-pods   Role/pod-reader   35s
```

## 4  搭建集群监控平台系统

>  关于监控平台的使用，整理了一份文档：[Prometheus](docs/Prometheus.md) （学会 k8s 之后再来看）

### 4.1 监控指标

一个好的监控系统主要监控以下内容：

1. 集群监控
   - 节点资源利用率
   - 节点数
   - 运行 Pods
2. Pod 监控
   - 容器指标
   - 应用程序【程序占用多少 CPU、内存】

### 4.2 监控平台

- prometheus【监控】
  - 定时搜索被监控服务的状态
  - 开源
  - 监控、报警、数据库
  - 以 HTTP 协议周期性抓取被监控组件状态
  - 不需要复杂的集成过程，使用 http 接口接入即可
- Grafana【展示】
  - 开源的数据分析和可视化工具
  - 支持多种数据源

**监控平台架构图**

<img src="./images/监控架构.png">

### 4.3 部署 Pormetheus

#### 4.3.1 node-exporter

`vim node-exporter.yaml`

```yaml
---
apiVersion: apps/v1
kind: DaemonSet
metadata:
  name: node-exporter
  namespace: kube-system
  labels:
    k8s-app: node-exporter
spec:
  selector:
    matchLabels:
      k8s-app: node-exporter
  template:
    metadata:
      labels:
        k8s-app: node-exporter
    spec:
      containers:
      - image: prom/node-exporter
        name: node-exporter
        ports:
        - containerPort: 9100
          protocol: TCP
          name: http
---
apiVersion: v1
kind: Service
metadata:
  labels:
    k8s-app: node-exporter
  name: node-exporter
  namespace: kube-system
spec:
  ports:
  - name: http
    port: 9100
    nodePort: 31672
    protocol: TCP
  type: NodePort
  selector:
    k8s-app: node-exporter
```

#### 4.3.2 rbac

`vim rbac-setup.yaml`

```yaml
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: prometheus
rules:
- apiGroups: [""]
  resources:
  - nodes
  - nodes/proxy
  - services
  - endpoints
  - pods
  verbs: ["get", "list", "watch"]
- apiGroups:
  - extensions
  resources:
  - ingresses
  verbs: ["get", "list", "watch"]
- nonResourceURLs: ["/metrics"]
  verbs: ["get"]
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: prometheus
  namespace: kube-system
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: prometheus
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: prometheus
subjects:
- kind: ServiceAccount
  name: prometheus
  namespace: kube-system
```

#### 4.3.3 ConfigMap

`vim configmap.yaml`

```yaml
apiVersion: v1
kind: ConfigMap
metadata:
  name: prometheus-config
  namespace: kube-system
data:
  prometheus.yml: |
    global:
      scrape_interval:     15s
      evaluation_interval: 15s
    scrape_configs:

    - job_name: 'kubernetes-apiservers'
      kubernetes_sd_configs:
      - role: endpoints
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - source_labels: [__meta_kubernetes_namespace, __meta_kubernetes_service_name, __meta_kubernetes_endpoint_port_name]
        action: keep
        regex: default;kubernetes;https

    - job_name: 'kubernetes-nodes'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics

    - job_name: 'kubernetes-cadvisor'
      kubernetes_sd_configs:
      - role: node
      scheme: https
      tls_config:
        ca_file: /var/run/secrets/kubernetes.io/serviceaccount/ca.crt
      bearer_token_file: /var/run/secrets/kubernetes.io/serviceaccount/token
      relabel_configs:
      - action: labelmap
        regex: __meta_kubernetes_node_label_(.+)
      - target_label: __address__
        replacement: kubernetes.default.svc:443
      - source_labels: [__meta_kubernetes_node_name]
        regex: (.+)
        target_label: __metrics_path__
        replacement: /api/v1/nodes/${1}/proxy/metrics/cadvisor

    - job_name: 'kubernetes-service-endpoints'
      kubernetes_sd_configs:
      - role: endpoints
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_scheme]
        action: replace
        target_label: __scheme__
        regex: (https?)
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_service_annotation_prometheus_io_port]
        action: replace
        target_label: __address__
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        action: replace
        target_label: kubernetes_name

    - job_name: 'kubernetes-services'
      kubernetes_sd_configs:
      - role: service
      metrics_path: /probe
      params:
        module: [http_2xx]
      relabel_configs:
      - source_labels: [__meta_kubernetes_service_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__address__]
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_service_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_service_name]
        target_label: kubernetes_name

    - job_name: 'kubernetes-ingresses'
      kubernetes_sd_configs:
      - role: ingress
      relabel_configs:
      - source_labels: [__meta_kubernetes_ingress_annotation_prometheus_io_probe]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_ingress_scheme,__address__,__meta_kubernetes_ingress_path]
        regex: (.+);(.+);(.+)
        replacement: ${1}://${2}${3}
        target_label: __param_target
      - target_label: __address__
        replacement: blackbox-exporter.example.com:9115
      - source_labels: [__param_target]
        target_label: instance
      - action: labelmap
        regex: __meta_kubernetes_ingress_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_ingress_name]
        target_label: kubernetes_name

    - job_name: 'kubernetes-pods'
      kubernetes_sd_configs:
      - role: pod
      relabel_configs:
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_scrape]
        action: keep
        regex: true
      - source_labels: [__meta_kubernetes_pod_annotation_prometheus_io_path]
        action: replace
        target_label: __metrics_path__
        regex: (.+)
      - source_labels: [__address__, __meta_kubernetes_pod_annotation_prometheus_io_port]
        action: replace
        regex: ([^:]+)(?::\d+)?;(\d+)
        replacement: $1:$2
        target_label: __address__
      - action: labelmap
        regex: __meta_kubernetes_pod_label_(.+)
      - source_labels: [__meta_kubernetes_namespace]
        action: replace
        target_label: kubernetes_namespace
      - source_labels: [__meta_kubernetes_pod_name]
        action: replace
        target_label: kubernetes_pod_name
```

#### 4.3.4 Deployment

`vim prometheus.deploy.yaml`

```yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  labels:
    name: prometheus-deployment
  name: prometheus
  namespace: kube-system
spec:
  replicas: 1
  selector:
    matchLabels:
      app: prometheus
  template:
    metadata:
      labels:
        app: prometheus
    spec:
      containers:
      - image: prom/prometheus:v2.0.0
        name: prometheus
        command:
        - "/bin/prometheus"
        args:
        - "--config.file=/etc/prometheus/prometheus.yml"
        - "--storage.tsdb.path=/prometheus"
        - "--storage.tsdb.retention=24h"
        ports:
        - containerPort: 9090
          protocol: TCP
        volumeMounts:
        - mountPath: "/prometheus"
          name: data
        - mountPath: "/etc/prometheus"
          name: config-volume
        resources:
          requests:
            cpu: 100m
            memory: 100Mi
          limits:
            cpu: 500m
            memory: 2500Mi
      serviceAccountName: prometheus    
      volumes:
      - name: data
        emptyDir: {}
      - name: config-volume
        configMap:
          name: prometheus-config  
```

#### 4.3.5 Service

`vim prometheus.svc.yaml`

```yaml
---
kind: Service
apiVersion: v1
metadata:
  labels:
    app: prometheus
  name: prometheus
  namespace: kube-system
spec:
  type: NodePort
  ports:
  - port: 9090
    targetPort: 9090
    nodePort: 30003
  selector:
    app: prometheus
```

#### 4.3.6 Create

```sh
kubectl create -f node-exporter.yaml
kubectl create -f rbac-setup.yaml
kubectl create -f configmap.yaml
kubectl create -f prometheus.deploy.yml
kubectl create -f prometheus.svc.yml
```

#### 4.3.7 Get

```sh
kubectl get pod,svc -n kube-system | grep prometheus

#  pod/prometheus-7486bf7f4b-nv2t5         1/1     Running   0          20m
#  service/prometheus      NodePort    10.102.173.197   <none>        9090:30003/TCP           20m

# 浏览器访问：[ip:port] 192.168.60.151:30003
```

<img src="./images/prometheus.png">

### 4.4 部署 Grafana

#### 4.4.1 Deployment

`vim grafana-deploy.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: grafana-core
  namespace: kube-system
  labels:
    app: grafana
    component: core
spec:
  replicas: 1
  selector:
    matchLabels:
      app: grafana
      component: core
  template:
    metadata:
      labels:
        app: grafana
        component: core
    spec:
      containers:
      - image: grafana/grafana:4.2.0
        name: grafana-core
        imagePullPolicy: IfNotPresent
        # env:
        resources:
          # keep request = limit to keep this container in guaranteed class
          limits:
            cpu: 100m
            memory: 100Mi
          requests:
            cpu: 100m
            memory: 100Mi
        env:
          # The following env variables set up basic auth twith the default admin user and admin password.
          - name: GF_AUTH_BASIC_ENABLED
            value: "true"
          - name: GF_AUTH_ANONYMOUS_ENABLED
            value: "false"
          # - name: GF_AUTH_ANONYMOUS_ORG_ROLE
          #   value: Admin
          # does not really work, because of template variables in exported dashboards:
          # - name: GF_DASHBOARDS_JSON_ENABLED
          #   value: "true"
        readinessProbe:
          httpGet:
            path: /login
            port: 3000
          # initialDelaySeconds: 30
          # timeoutSeconds: 1
        volumeMounts:
        - name: grafana-persistent-storage
          mountPath: /var
      volumes:
      - name: grafana-persistent-storage
        emptyDir: {}
```

#### 4.4.2 Service

`vim grafana-svc.yaml`

```yaml
apiVersion: v1
kind: Service
metadata:
  name: grafana
  namespace: kube-system
  labels:
    app: grafana
    component: core
spec:
  type: NodePort
  ports:
    - port: 3000
  selector:
    app: grafana
    component: core
```

#### 4.4.3 Runing

`vim grafana-ing.yaml`

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
   name: grafana
   namespace: kube-system
spec:
   rules:
   - host: k8s.grafana
     http:
       paths:
       - path: /
         backend:
          serviceName: grafana
          servicePort: 3000
```

#### 4.4.4 Creat

```sh
kubectl create -f grafana-deploy.yaml
kubectl create -f grafana-svc.yaml
kubectl create -f grafana-ing.yaml
```

#### 4.4.5 Get

```sh
kubectl get pod,svc -n kube-system | grep grafana
#  pod/grafana-core-768b6bf79c-lmq9z       1/1     Running   0          35m
#  service/grafana         NodePort    10.99.106.133    <none>        3000:32389/TCP           35m

#  浏览器访问：[ip:port] http://192.168.60.151:32389/
#  用户名/密码：admin/admin
```

**登录**

<img src="./images/grafana.login.png" style="zoom:80%;" >

**添加数据源 Prometheus**

<img src="./images/grafana.adddata.png" style="zoom: 80%;" >

**展示**

<img src="./images/grafana.dashboard1.png">

<img src="./images/grafana.dashboard2.png">

<img src="./images/grafana.dashboard3.png">

## 5  从零搭建高可用 Kubernetes 集群

> 之前我们搭建的集群，只有一个 master 节点，当 master 节点宕机的时候，通过 node 节点将无法继续访问，而 master 主要是管理作用，所以整个集群将无法提供服务。

### 5.1 高可用集群架构

- 在 node 节点和 master 节点之间，需要一个 LoadBalancer 组件
  - 【作用 1】负载
  - 【作用 2】检查 master 节点的状态
- 对外需要一个统一的 VIP
  - 【作用 1】虚拟 ip 对外进行访问

<img src="./images/k8s高可用.png">

### 5.2 高可用集群技术细节

- keepalived：配置虚拟 ip，检查节点的状态
- haproxy：负载均衡服务【类似于 nginx】
- apiserver
- controller
- manager
- scheduler

### 5.3 高可用集群搭建

> 我们采用 2 个 master 节点，一个 node 节点来搭建高可用集群。

#### 5.3.1 安装步骤

使用二进制包方式搭建 Kubernetes 集群主要分为以下几步：

1. 【**环境准备**】准备四台虚拟机，并安装操作系统 CentOS 7.x
2. 【**系统初始化**】对四个刚安装好的操作系统进行初始化操作
3. 【**安装 docker、kubectl、kubeadm、kubectl**】对四个节点进行安装
4. 【**配置高可用 VIP**】对 master 节点安装`keepalived`和`haproxy`
5. 【**部署 master 组件**】在 master 节点上安装`kube-apiserver`、`kube-controller-manager`、`kube-scheduler`
6. 【**安装网络插件**】配置 CNI 网络插件，用于节点之间的连通
7. 【**测试集群**】通过拉取一个 nginx 进行测试，能否进行外网测试

####  5.3.2 安装要求

在开始之前，部署 Kubernetes 集群机器需要满足以下几个条件：

- 一台或多台机器，操作系统 CentOS7.x-86_x64
- 硬件配置：2GB 或更多 RAM，2 个 CPU 或更多 CPU，硬盘 30GB 或更多**【注意】【注意】【注意】【master 需要两核】**
- 可以访问外网，需要拉取镜像，如果服务器不能上网，需要提前下载镜像并导入节点
- 禁止 swap 分区

#### 5.3.3 准备环境

| 角色            | IP             | 配置    | 步骤                                                         |
| --------------- | -------------- | ------- | ------------------------------------------------------------ |
| k8sLoadBalancer | 192.168.60.150 | 2CPU 1G | `init` `docker` `kubectl` `kubeadm` `kubectl`                |
| k8smaster1      | 192.168.60.151 | 2CPU 2G | `init` `docker` `kubectl` `kubeadm` `kubectl` `keepalived` `haproxy` |
| k8smaster2      | 192.168.60.152 | 2CPU 2G | `init` `docker` `kubectl` `kubeadm` `kubectl`  `keepalived` `haproxy` |
| k8snode1        | 192.168.60.153 | 2CPU 2G | `init` `docker` `kubectl` `kubeadm` `kubectl`                |

#### 5.3.4 系统初始化

```sh
# 关闭防火墙
systemctl stop firewalld
# 禁用 firewalld 服务
systemctl disable firewalld

# 关闭 selinux
# 临时关闭【立即生效】告警，不启用，Permissive，查看使用 getenforce 命令
setenforce 0  
# 永久关闭【重启生效】
sed -i 's/SELINUX=enforcing/\SELINUX=disabled/' /etc/selinux/config  

# 关闭 swap
# 临时关闭【立即生效】查看使用 free 命令
swapoff -a 
# 永久关闭【重启生效】
sed -ri 's/.*swap.*/#&/' /etc/fstab

# 在主机名静态查询表中添加 4 台主机
cat >> /etc/hosts << EOF
192.168.60.150 k8sLoadBalancer
192.168.60.151 k8smaster1
192.168.60.152 k8smaster2
192.168.60.153 k8snode1
EOF

# 将桥接的 IPv4 流量传递到 iptables 的链【3 个节点上都执行】
cat > /etc/sysctl.d/k8s.conf << EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

# 生效
sysctl --system  

# 时间同步
yum install ntpdate -y
ntpdate time.windows.com

# 根据规划设置主机名【k8sLoadBalancer 节点上操作】
hostnamectl set-hostname k8sLoadBalancer
# 根据规划设置主机名【k8smaster1 节点上操作】
hostnamectl set-hostname ks8master1
# 根据规划设置主机名【k8smaster2 节点上操作】
hostnamectl set-hostname k8smaster2
# 根据规划设置主机名【k8snode1 节点操作】
hostnamectl set-hostname k8snode1
```

#### 5.3.5 安装 docker、kubelet、kubeadm、kubectl

> 所有节点安装 docker/kubelet/kubeadm/kubectl，Kubernetes 默认 CRI（容器运行时）为 docker，因此先安装 docker

1、安装 docker

（1）首先配置一下 docker 的阿里 yum 源

```sh
cat >/etc/yum.repos.d/docker.repo<<EOF
[docker-ce-edge]
name=Docker CE Edge - \$basearch
baseurl=https://mirrors.aliyun.com/docker-ce/linux/centos/7/\$basearch/edge
enabled=1
gpgcheck=1
gpgkey=https://mirrors.aliyun.com/docker-ce/linux/centos/gpg
EOF
```

（2）然后 yum 方式安装 docker

```sh
# yum 安装
yum -y install docker-ce

# 查看 docker 版本
docker --version  
```

（3）配置 docker 的镜像源【阿里云】

```sh
cat >> /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"]
}
EOF
```

（4）然后启动 docker

```sh
systemctl start docker
systemctl enable docker
systemctl status docker
```

2、安装 kubeadm，kubelet 和 kubectl

（1）配置 kubernetes 阿里云 yum 源

```sh
cat > /etc/yum.repos.d/kubernetes.repo << EOF
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF
```

（2）yum 方式安装，由于版本更新频繁，这里指定版本号部署

```sh
# 查看版本
yum list kubeadm --showduplicates

# 安装 kubelet、kubeadm、kubectl，同时指定版本
yum install -y kubelet-1.18.0 kubeadm-1.18.0 kubectl-1.18.0
# 设置开机启动【这里先不启动】
systemctl enable kubelet
```

#### 5.3.6 配置高可用 VIP【haproxy+keepalived】

【k8smaster1 + k8smaster2 上操作】

1、安装 haproxy + keepalived

> 我们需要在所有的 master 节点【k8smaster1 和 k8smaster2】上部署 haproxy + keepAlive

```sh
yum install -y haproxy keepalived
```

2、配置 haproxy

> 所有`master`节点的`haproxy`配置相同，haproxy 的配置文件是`/etc/haproxy/haproxy.cfg`
>
> 配置中声明了后端代理的两个 master 节点服务器，指定了 haproxy 运行的端口为 16443 等，因此 16443 端口为集群的入口

```sh
cat > /etc/haproxy/haproxy.cfg << EOF
#---------------------------------------------------------------------
# Global settings
#---------------------------------------------------------------------
global
    # to have these messages end up in /var/log/haproxy.log you will
    # need to:
    # 1) configure syslog to accept network log events.  This is done
    #    by adding the '-r' option to the SYSLOGD_OPTIONS in
    #    /etc/sysconfig/syslog
    # 2) configure local2 events to go to the /var/log/haproxy.log
    #   file. A line like the following can be added to
    #   /etc/sysconfig/syslog
    #
    #    local2.*                       /var/log/haproxy.log
    #
    log         127.0.0.1 local2
    
    chroot      /var/lib/haproxy
    pidfile     /var/run/haproxy.pid
    maxconn     4000
    user        haproxy
    group       haproxy
    daemon 
       
    # turn on stats unix socket
    stats socket /var/lib/haproxy/stats
#---------------------------------------------------------------------
# common defaults that all the 'listen' and 'backend' sections will
# use if not designated in their block
#---------------------------------------------------------------------  
defaults
    mode                    http
    log                     global
    option                  httplog
    option                  dontlognull
    option http-server-close
    option forwardfor       except 127.0.0.0/8
    option                  redispatch
    retries                 3
    timeout http-request    10s
    timeout queue           1m
    timeout connect         10s
    timeout client          1m
    timeout server          1m
    timeout http-keep-alive 10s
    timeout check           10s
    maxconn                 3000
#---------------------------------------------------------------------
# kubernetes apiserver frontend which proxys to the backends
#--------------------------------------------------------------------- 
frontend kubernetes-apiserver
    mode                 tcp
    bind                 *:16443
    option               tcplog
    default_backend      kubernetes-apiserver    
#---------------------------------------------------------------------
# round robin balancing between the various backends
#---------------------------------------------------------------------
backend kubernetes-apiserver
    mode        tcp
    balance     roundrobin
    server      k8smaster1   192.168.60.151:6443 check
    server      k8smaster2   192.168.60.152:6443 check
#---------------------------------------------------------------------
# collection haproxy statistics message
#---------------------------------------------------------------------
listen stats
    bind                 *:10080
    stats auth           admin:awesomePassword
    stats refresh        5s
    stats realm          HAProxy\ Statistics
    stats uri            /admin?stats
EOF
```

3、配置 keepalived

>`keepalived`中使用`track_script`机制来配置脚本进行探测`kubernetes`的`master`节点是否宕机，并以此切换节点实现高可用。

（1）`k8smaster1`节点的`keepalived`配置文件如下所示，配置文件所在的位置`/etc/keepalived/keepalived.cfg`。

```sh
cat > /etc/keepalived/keepalived.conf <<EOF 
! Configuration File for keepalived

global_defs {
   router_id k8s
}

vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 3
    weight -2
    fall 10
    rise 2
}

vrrp_instance VI_1 {
    state MASTER 
    interface ens33 
    mcast_src_ip 192.168.60.151
    virtual_router_id 51
    priority 100
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass ceb1b3ec013d66163d6ab
    }
    virtual_ipaddress {
        192.168.60.150
    }
    track_script {
        check_haproxy
    }
}
EOF
```

需要注意几点（前两点记得修改）：

- `mcast_src_ip`：配置多播源地址，此地址是当前主机的 ip 地址。
- `priority`：`keepalived`根据此项参数的大小仲裁`master`节点。我们这里让 master 节点为`kubernetes`提供服务，其他两个节点暂时为备用节点。因此`k8smaster1`节点设置为`100`，`k8smaster2`节点设置为`99`。
- `state`：我们将`k8smaster1`节点的`state`字段设置为`MASTER`，其他节点字段修改为`BACKUP`。
- 上面的集群检查功能是关闭的，等到集群建立完成后再开启。

（2）配置 k8smaster2 节点

```sh
cat > /etc/keepalived/keepalived.conf <<EOF 
! Configuration File for keepalived

global_defs {
   router_id k8s
}

vrrp_script check_haproxy {
    script "killall -0 haproxy"
    interval 3
    weight -2
    fall 10
    rise 2
}

vrrp_instance VI_1 {
    state BACKUP 
    interface ens33 
    mcast_src_ip 192.168.60.152
    virtual_router_id 51
    priority 99
    advert_int 1
    authentication {
        auth_type PASS
        auth_pass ceb1b3ec013d66163d6ab
    }
    virtual_ipaddress {
        192.168.60.150
    }
    track_script {
        check_haproxy
    }
}
EOF
```

4、启动和检查 【k8smaster1 和 k8smaster2 均要启动】

```sh
# 启动 haproxy
systemctl start haproxy
systemctl enable haproxy
systemctl status haproxy

# 启动 keepalived
systemctl start keepalived.service
systemctl enable keepalived.service
systemctl status keepalived.service

# 启动后查看 master 网卡信息
ip a s ens33

# 检查是否可以 ping 通
ping 192.168.60.150

# 如果出错
#      初始化一下！！！并重新启动！！！
systemctl stop firewalld
setenforce 0  
swapoff -a 
```

#### 5.3.7 部署 Kubernetes Master 组件

【k8smaster1 + k8smaster2 + k8snode1 上操作】

1、`k8smaster1` 节点

（1）初始化操作

```sh
# 导出初始化配置文件，然后修改配置，再进行初始化
kubeadm config print init-defaults > kubeadm-init.yaml

# 这里直接写入配置，并初始化
cat > kubeadm-init.yaml << EOF
apiVersion: kubeadm.k8s.io/v1beta2
bootstrapTokens:
- groups:
  - system:bootstrappers:kubeadm:default-node-token
  token: abcdef.0123456789abcdef
  ttl: 24h0m0s
  usages:
  - signing
  - authentication
kind: InitConfiguration
localAPIEndpoint:
  advertiseAddress: 192.168.60.150 # k8sLoadBalancer ip
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: k8sloadbalancer
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer: # 添加两行配置
  certSANs:
  - "192.168.60.150" # k8sLoadBalancer ip 即 VIP 的地址
  timeoutForControlPlane: 4m0s
apiVersion: kubeadm.k8s.io/v1beta2
certificatesDir: /etc/kubernetes/pki
clusterName: kubernetes
controllerManager: {}
dns:
  type: CoreDNS
etcd:
  local:
    dataDir: /var/lib/etcd
imageRepository: registry.cn-hangzhou.aliyuncs.com/google_containers   # 阿里云的镜像站点
controlPlaneEndpoint: "192.168.60.150:16443"  # VIP 的地址和端口
kind: ClusterConfiguration
kubernetesVersion: v1.18.0
networking:
  dnsDomain: cluster.local
  serviceSubnet: 10.96.0.0/12
  podSubnet: 10.244.0.0/16        # 添加 pod 网段
scheduler: {}
EOF

# 直接 kubeadm init 初始化，中间会拉取镜像，速度较慢，分为两步来做
# （1）提前拉取镜像
kubeadm config images pull --config kubeadm-init.yaml
# （2）初始化
kubeadm init --config kubeadm-init.yaml --upload-certs

####### 初始化结果 ########
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
  https://kubernetes.io/docs/concepts/cluster-administration/addons/

You can now join any number of the control-plane node running the following command on each as root:

  kubeadm join 192.168.60.150:16443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:68d59df77d9109c44a60a8ca4e7f0932d8cd270c5d0a8adc83c9a1a7d72de73a \
    --control-plane --certificate-key b84d54cf9015ef8252e38d68ae96be4b7e41fc9380d8dc2b9ac9ae916b0e9cda

Please note that the certificate-key gives access to cluster sensitive data, keep it secret!
As a safeguard, uploaded-certs will be deleted in two hours; If necessary, you can use
"kubeadm init phase upload-certs --upload-certs" to reload certs afterward.

Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.168.60.150:16443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:68d59df77d9109c44a60a8ca4e7f0932d8cd270c5d0a8adc83c9a1a7d72de73a 
```

（2）按照提示信息，执行下方命令

```sh
# 执行下方命令
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

# 查看节点
kubectl get nodes
# 查看 pod
kubectl get pods -n kube-system

## 输出结果
[root@ks8master1 ~]# kubectl get nodes
NAME              STATUS     ROLES    AGE     VERSION
k8sloadbalancer   NotReady   master   3m58s   v1.18.0
[root@ks8master1 ~]# kubectl get pods -n kube-system
NAME                                      READY   STATUS    RESTARTS   AGE
coredns-546565776c-skjzz                  0/1     Pending   0          3m50s
coredns-546565776c-xm8wf                  0/1     Pending   0          3m50s
etcd-k8sloadbalancer                      1/1     Running   0          4m5s
kube-apiserver-k8sloadbalancer            1/1     Running   0          4m5s
kube-controller-manager-k8sloadbalancer   1/1     Running   0          4m5s
kube-proxy-gbjmm                          1/1     Running   0          3m48s
kube-scheduler-k8sloadbalancer            1/1     Running   0          4m5s
```

2、`k8smaster2` 节点

按照`k8smaster1`提示信息，将`k8smaster2`加入集群

```sh
# k8smaster2 加入集群
kubeadm join 192.168.60.150:16443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:68d59df77d9109c44a60a8ca4e7f0932d8cd270c5d0a8adc83c9a1a7d72de73a \
    --control-plane --certificate-key b84d54cf9015ef8252e38d68ae96be4b7e41fc9380d8dc2b9ac9ae916b0e9cda
 

# 查看集群状态
kubectl get cs
# 查看 pod
kubectl get pods -n kube-system
```

3、`k8snode1` 节点

按照`k8smaster1`提示信息，将`k8snode1`加入集群

```sh
kubeadm join 192.168.60.150:16443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:68d59df77d9109c44a60a8ca4e7f0932d8cd270c5d0a8adc83c9a1a7d72de73a 
```

#### 5.3.8 安装集群网络

从官方地址获取到 flannel 的 yaml，在 k8smaster1 上执行

```sh
# 下载 yaml 文件
wget -c https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml

# 安装 flannel 网络
kubectl apply -f kube-flannel.yml 

# 检查
kubectl get pods -n kube-system

## 可以看到 kube-flannel 正在安装
[root@ks8master1 ~]# kubectl get pods -n kube-system
NAME                                      READY   STATUS     RESTARTS   AGE
coredns-546565776c-skjzz                  0/1     Pending    0          21m
coredns-546565776c-xm8wf                  0/1     Pending    0          21m
etcd-k8sloadbalancer                      1/1     Running    2          21m
etcd-k8smaster2                           1/1     Running    0          7m58s
kube-apiserver-k8sloadbalancer            1/1     Running    3          21m
kube-apiserver-k8smaster2                 1/1     Running    1          7m58s
kube-controller-manager-k8sloadbalancer   1/1     Running    2          21m
kube-controller-manager-k8smaster2        1/1     Running    1          7m58s
kube-flannel-ds-cv84g                     0/1     Init:1/2   0          91s
kube-flannel-ds-j9mbn                     0/1     Init:1/2   0          91s
kube-flannel-ds-qplqm                     0/1     Init:1/2   0          91s
kube-proxy-gbjmm                          1/1     Running    0          21m
kube-proxy-qqdl5                          1/1     Running    0          13m
kube-proxy-s8bvq                          1/1     Running    0          6m27s
kube-scheduler-k8sloadbalancer            1/1     Running    2          21m
kube-scheduler-k8smaster2                 1/1     Running    1          7m58s
```

#### 5.3.9 测试 kubernetes 集群

在 Kubernetes 集群中创建一个 pod，验证是否正常运行：

```sh
# 创建 nginx deployment
kubectl create deployment nginx --image=nginx
# 暴露端口
kubectl expose deployment nginx --port=80 --type=NodePort
# 查看状态
kubectl get pod,svc

## [ip:port]
# 浏览器访问：
# 192.168.60.151:32594
# 192.168.60.152:32594
# 192.168.60.153:32594
[root@ks8master1 ~]# kubectl get pod,svc
NAME                 TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
service/kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP        24m
service/nginx        NodePort    10.109.174.226   <none>        80:32594/TCP   8s
```

然后我们通过任何一个节点，都能够访问我们的 nginx 页面。

## 6  在集群环境中部署项目

> 在 Kubernetes 集群中部署 Java 项目

### 6.1 容器交付流程

- 开发代码阶段
  - 编写代码
  - 编写 Dockerfile【打镜像做准备】
- 持续交付/集成
  - 代码编译打包
  - 制作镜像
  - 上传镜像仓库
- 应用部署
  - 环境准备
  - Pod
  - Service
  - Ingress
- 运维
  - 监控
  - 故障排查
  - 应用升级

### 6.2 k8s 部署 java 项目流程

- 制作镜像【Dockerfile】
- 上传到镜像仓库【Dockerhub、阿里云、网易】
- 控制器部署镜像【Deployment】
- 对外暴露应用【Service、Ingress】
- 运维【监控、升级】

### 6.3 k8s 部署 Java 项目

#### 6.3.1 制作 Jar 包

> 这里已经制作好了一个 jar 包，直接下载即可

```text
javaproject/
└── demojenkins
    ├── demojenkins.iml
    ├── Dockerfile
    ├── HELP.md
    ├── mvnw
    ├── mvnw.cmd
    ├── pom.xml
    ├── src
    │   ├── main
    │   └── test
    ## 使用 java 和 Maven 进行打包
    ## 使用 java （springboot）进行打包
    └── target	# jar 包
        ├── classes
        ├── generated-sources
        ├── generated-test-sources
        ├── maven-archiver
        ├── maven-status
        ├── surefire-reports
        ├── demojenkins.jar  ## jar 包

```

#### 6.3.2 制作镜像

> 这里已经写好了 Dockerfile

```dockerfile
FROM openjdk:8-jdk-alpine
VOLUME /tmp
ADD ./target/demojenkins.jar demojenkins.jar
ENTRYPOINT ["java","-jar","/demojenkins.jar", "&"]
```

打包镜像：

```sh
cd javaproject/demojenkins/
docker build -t java-demo-01:latest .
```

测试镜像：

```sh
docker run -d -p 8111:8111 java-demo-01:latest -t 
```

访问：`ip:8111/user`

<img src="images/javademo.png">

上传到镜像仓库：（本地仓库）

```sh
## 搭建私人仓库
mkdir -p /data/myregistry
docker pull registry:latest
docker run -d -p 5000:5000 --name my_registry --restart=always -v /data/myregistry:/var/lib/registry registry:latest

## 更改 docker 配置文件（在需要连接到私有仓库的机器上全部都执行一遍）
## 在 k8smaster k8snode1 k8snode2 上均执行一遍
cat > /etc/docker/daemon.json << EOF
{
  "registry-mirrors": ["https://b9pmyelo.mirror.aliyuncs.com"],
  "insecure-registries": ["192.168.60.151:5000"]
}
EOF
## 重启 docker，重启 registry（如果停止了的话）
systemctl restart docker  # 3 台机器上执行
docker start my_registry  # 主节点上执行（因为私人仓库在主节点上）
```

访问：`ip:5000/v2/_catalog`查看本地仓库镜像

<img src="images/dockerregistry.png">

测试本地私有仓库：

```sh
docker tag java-demo-01 192.168.60.151:5000/test/java-demo-01:v1
docker push 192.168.60.151:5000/test/java-demo-01:v1
```

访问：`ip:5000/v2/_catalog`查看本地仓库镜像

<img src="images/javadocker2.png">

在 node 节点上测试：

```sh
docker pull 192.168.60.151:5000/test/java-demo-01:v1
```

#### 6.3.4 部署项目

```sh
kubectl create deployment java01 --image=192.168.60.151:5000/test/java-demo-01:v1 --dry-run -o yaml > java01.yaml
kubectl create -f java01.yaml
kubectl get pod -o wide
kubectl expose deployment java01 --port=8111 --target-port=8111 --type=NodePort

## 查看暴露的端口
[root@k8smaster test]# kubectl get svc
NAME         TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)          AGE
java01       NodePort    10.109.195.123   <none>        8111:31954/TCP   15s
kubernetes   ClusterIP   10.96.0.1        <none>        443/TCP          6d

```

浏览器访问：`ip:31954`

<img src="images/k8sjava.png">

其实不一定非要部署 jar 包

部署其他服务也是一样的，比如部署 nginx，将 index.html 换掉。（学会这个，其他的都会了）

总结一下要点：

- 制作项目的 docker 镜像
- 将镜像上传到 docker 仓库
- 使用 k8s 部署项目

完结。......

恭喜你！完成了第一阶段的学习。





## 下一阶段学习



传送门...... （考完更新！）

- [CKA 认证]()
- [CKS 认证]()

