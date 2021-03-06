# Docker

什么是容器

容器技术已经成为应用程序封装和交付的核心技术

容器技术的核心有以下几个内核技术组成:

- Cgroups(Control Groups)-资源管理
- NameSpace-进程隔离
- SELinux安全

由于是在物理机上实施隔离,启动一个容器,可以像启动一个进程一样快速

Docker是完整的一套容器管理系统

Docker提供了一组命令,让用户更加方便直接地使用容器技术,而不需要过多关心底层内核技术

Docker优点
• 相比于传统的虚拟化技术,容器更加简洁高效
• 传统虚拟机需要给每个VM安装操作系统
• 容器使用的共享公共库和程序

Docker缺点
• 容器的隔离性没有虚拟化强
• 共用Linux内核,安全性有先天缺陷
• SELinux难以驾驭
• 监控容器和容器排错是挑战



安装前准备

- 需要64位操作系统
- 至少RHEL7以上的版本
- 关闭防火墙(不是必须)

```shell
# 添加阿里云镜像
curl -o /etc/yum.repos.d/CentOS-Base.repo http://mirrors.aliyun.com/repo/Centos-7.repo
# 添加docker国内镜像
curl -o /etc/yum.repos.d/Docker.repo http://mirrors.aliyun.com/docker-ce/linux/centos/docker-ce.repo
# 缓存
yum makecache fast
# 必要安装
yum install -y yum-utils device-mapper-persistent-data lvm2
# 安装docker社区版
yum -y install docker-ce
systemctl restart docker
systemctl enable docker
# 验证
docker version
# 查看镜像
docker images

# 
yum-config-manager --add-repo=https://cbs.centos.org/repos/paas7-crio-311-candidate/x86_64/os/

# 安装CRI-O
modprobe overlay
modprobe br_netfilter

# Setup required sysctl params, these persist across reboots.
cat > /etc/sysctl.d/99-kubernetes-cri.conf <<EOF
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF

sysctl --system

# Install prerequisites
yum-config-manager --add-repo=https://cbs.centos.org/repos/paas7-crio-311-candidate/x86_64/os/

# Install CRI-O
yum install --nogpgcheck cri-o

systemctl start crio

# 安装 Containerd

### Add docker repository
yum-config-manager \
    --add-repo \
    https://download.docker.com/linux/centos/docker-ce.repo

## Install containerd
yum update && yum install containerd.io

# Configure containerd
mkdir -p /etc/containerd
containerd config default > /etc/containerd/config.toml

# Restart containerd
systemctl restart containerd

# systemd
# 要使用systemdcgroup的驱动程序，设置plugins.cri.systemd_cgroup = true在/etc/containerd/config.toml。使用kubeadm时，手动配置kubelet的 cgroup驱动程序
```

Docker hub镜像仓库

https://hub.docker.com

Docker官方提供公共镜像的仓库(Registry)

## Docker 国内镜像

```shell
vim /etc/docker/daemon.json
{
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}
```



## 镜像常用命令

```shell
docker search 关键字 # 查找镜像

# 下载镜像(从镜像仓库中下载镜像)
# docker pull [OPTIONS] NAME[:TAG|@DIGEST]
docker pull docker.io/busybox

# 上传镜像(上传镜像到仓库)
# docker push [OPTIONS] NAME[:TAG]
docker push docker.io/busybox

# 导出镜像(将本地镜像导出为tar文件),标签默认为latest
docker save docker.io/busybox:latest -o busybox.tar

# 导入镜像(通过tar包文件导入镜像)
docker load -i nginx.tar

# 开启另一个终端(查看容器列表)
docker ps # -a 全部 -aq 全部的id

# 查看镜像制作历史
docker history

# 查看属性信息
docker inspect

# 删除本地镜像
docker rmi

# 修改镜像名称和标签
docker tag
```



## 容器常用命令

```shell
# 创建一个全新的容器并启动
# docker 子命令 参数 镜像名称[：标签] [容器内启动命令]
docker run -it docker.io/centos:latest /bin/bash

# ctrl+p+q 保持容器运行退出

# i 交互 t 终端 d 后台运行

# 关闭容器
docker stop

# 启动容器
docker start

# 重启容器
docker restart

# /进入容器
docker attach 容器ID # 进入上帝进程，不推荐
docker exec -it 容器ID 容器中的命令 # 新开进程进入

# 查看容器底层信息
docker inspect

# 查看容器进程列表
docker top

# 删除容器
docker rm
```



## 自定义镜像

修改容器至需要的配置后执行命令

```shell
docker commit 容器ID 镜像名称:标签 # 生成新的镜像
```



Dockerfile语法格式，关键字大写
– FROM:基础镜像
– MAINTAINER:镜像创建者信息
– EXPOSE:开放的端口
– ENV:设置变量
– ADD:复制文件到镜像
– RUN:制作镜像时执行的命令,可以有多个
– WORKDIR:定义容器默认工作目录
– CMD:容器启动时执行的命令,仅可以有一条CMD

```shell
# 新建空目录，并进入
vim Dockerfile # 注意大小写
FROM    docker.io/centos:latest
RUN     rm -f /etc/yum.repos.d/*.repo
ADD     local.repo /etc/yum.repos.d/local.repo
RUN     yum -y install httpd
RUN     yum clean all
WORKDIR /var/www/html/
RUN     echo "hello world" > index.html
ENV     EnvironmentFile=/etc/sysconfig/httpd
EXPOSE  80
CMD     ["/usr/sbin/httpd", "-DFOREGROUND"]

vim local.repo # 创建yum源文件

# 生成新镜像
# docker build -t 镜像名:标签 Dockerfile所在目录
docker build -t qlstos:0.2 .
```

CMD 格式

CMD ["/bin/ls", "-l", "-a"] # 注意空格



## 自定义镜像仓库

```shell
# 安装私有仓库(服务端)
yum install -y docker-distribution
systemctl enable docker-distribution
systemctl start docker-distribution
# 仓库配置文件及数据存储路径
/etc/docker-distribution/registry/config.yml
/var/lib/registry
```

客户端配置

```shell
# 修改配置文件
vim /etc/sysconfig/docker
# 允许非加密方式访问仓库
INSECURE_REGISTRY='--insecure-registry IP:5000'
# docker 仓库地址
ADD_REGISTRY='--add-registry IP:5000'

systemctl restart docker
```

为镜像创建标签

```shell
# 这里的地址要写 宿主机 的 IP 地址或主机名，可以省略
docker tag 镜像:标签 IP:5000/镜像:标签
```

上传镜像

```shell
# 上传镜像的标签内包含地址和端口号
docker push IP:5000/镜像:标签
```

远程启动容器

```shell
docker run -it IP:5000/镜像:标签
```

查看私有镜像仓库中的 镜像名称

```shell
curl IP:5000/v2/_catalog
```

查看某一仓库的标签

```shell
curl IP:5000/v2/<imagename>/tags/list/
```

进入registry 的配置文件,进入容器查看

```shell
cat /etc/docker/registry/config.yml
```



## 持久化存储

卷的概念
• docker容器不保持任何数据
• 重要数据请使用外部卷存储(数据持久化)
• 容器可以挂载真实机目录或共享存储为卷

主机卷的映射
• 将真实机目录挂载到容器中提供持久化存储
– 目录不存在就自动创建
– 目录存在就直接覆盖掉

```shell
docker run -v /data:/data -it docker.io/centos /bin/bash # -v 本地目录：容器内目录
```



共享存储基本概念
• 一台共享存储服务器可以提供给所有Docker主机使用
• 共享存储服务器(NAS、SAN、DAS等)
• 如:
– 使用NFS创建共享存储服务器
– 客户端挂载NFS共享,并最终映射到容器中

使用共享存储的案例
• NFS 服务器共享目录

```shell
yum -y install nfs-utils
vim /etc/exports
systemctl start nfs
```

• Docker主机
– mount 挂载共享
– 运行容器时,使用-v选项映射磁盘到容器中



## Docker网络

```shell
# 查看默认Docker创建的网络模型
docker network list

# 新建Docker网络模型
docker network create --subnet=10.10.10.0/24 docker1

# 启动容器,使用刚刚创建的自定义网桥
docker run --network=bridge|host|none ... ...
docker run --network=docker1 -itd docker.io/myos
```

客户端访问容器内的资源

• 默认容器可以访问外网
• 但外部网络的主机不可以访问容器内的资源
• 容器的特征是可以把宿主机变成对应的服务
– 我们可以使用 -p 参数把容器端口和宿主机端口绑定
– -p 宿主机端口:容器端口
– 例如 把宿主机变成 httpd

```shell
docker run -itd -p 80:80 docker.io/myos:httpd     # -p 真机:容器
```

• Docker主机
– mount 挂载共享
– 运行容器时,使用-v选项映射磁盘到容器中



## Docker compose

### 安装 Docker Compose

- 安装 Docker Compose 可以通过下面命令自动下载适应版本的 Compose，并为安装脚本添加执行权限

```
sudo curl -L https://github.com/docker/compose/releases/download/1.21.2/docker-compose-$(uname -s)-$(uname -m) -o /usr/local/bin/docker-compose
sudo chmod +x /usr/local/bin/docker-compose
```

- 查看安装是否成功

```
docker-compose -v
```

### 快速入门

- 打包项目，获得 jar 包 docker-demo-0.0.1-SNAPSHOT.jar

```
mvn clean package
```

- 在 jar 包所在路径创建 Dockerfile 文件，添加以下内容

```
FROM java:8
VOLUME /tmp
ADD docker-demo-0.0.1-SNAPSHOT.jar app.jar
RUN bash -c 'touch /app.jar'
EXPOSE 9000
ENTRYPOINT ["java","-Djava.security.egd=file:/dev/./urandom","-jar","app.jar"]
```

- 在 jar 包所在路径创建文件 docker-compose.yml，添加以下内容

```
version: '2' # 表示该 Docker-Compose 文件使用的是 Version 2 file
services:
  docker-demo:  # 指定服务名称
    build: .  # 指定 Dockerfile 所在路径
    ports:    # 指定端口映射
      - "9000:8761"
```

- 在 docker-compose.yml 所在路径下执行该命令 Compose 就会自动构建镜像并使用镜像启动容器

```
docker-compose up
docker-compose up -d  // 后台启动并运行容器
```

- 访问 <http://localhost:9000/hello> 即可访问微服务接口

### 工程、服务、容器

- **Docker Compose 将所管理的容器分为三层，分别是工程（project）、服务（service）、容器（container）**
- **Docker Compose 运行目录下的所有文件（docker-compose.yml）组成一个工程,一个工程包含多个服务，每个服务中定义了容器运行的镜像、参数、依赖，一个服务可包括多个容器实例**

### Docker Compose 常用命令与配置

### 常见命令

- **ps**：列出所有运行容器

```
docker-compose ps
```

- **logs**：查看服务日志输出

```
docker-compose logs
```

- **port**：打印绑定的公共端口，下面命令可以输出 eureka 服务 8761 端口所绑定的公共端口

```
docker-compose port eureka 8761
```

- **build**：构建或者重新构建服务

```
docker-compose build
```

- **start**：启动指定服务已存在的容器

```
docker-compose start eureka
```

- **stop**：停止已运行的服务的容器

```
docker-compose stop eureka
```

- **rm**：删除指定服务的容器

```
docker-compose rm eureka
```

- **up**：构建、启动容器

```
docker-compose up
```

- **kill**：通过发送 SIGKILL 信号来停止指定服务的容器

```
docker-compose kill eureka
```

- **pull**：下载服务镜像
- **scale**：设置指定服务运气容器的个数，以 service=num 形式指定

```
docker-compose scale user=3 movie=3
```

- **run**：在一个服务上执行一个命令

```
docker-compose run web bash
```

### docker-compose.yml 属性

- **version**：指定 docker-compose.yml 文件的写法格式
- **services**：多个容器集合
- **build**：配置构建时，Compose 会利用它自动构建镜像，该值可以是一个路径，也可以是一个对象，用于指定 Dockerfile 参数

```yaml
build: ./dir
---------------
build:
    context: ./dir
    dockerfile: Dockerfile
    args:
        buildno: 1
```

- **command**：覆盖容器启动后默认执行的命令

```yaml
command: bundle exec thin -p 3000
----------------------------------
command: [bundle,exec,thin,-p,3000]
```

- **dns**：配置 dns 服务器，可以是一个值或列表

```yaml
dns: 8.8.8.8
------------
dns:
    - 8.8.8.8
    - 9.9.9.9
```

- **dns_search**：配置 DNS 搜索域，可以是一个值或列表

```yaml
dns_search: example.com
------------------------
dns_search:
    - dc1.example.com
    - dc2.example.com
```

- **environment**：环境变量配置，可以用数组或字典两种方式

```yaml
environment:
    RACK_ENV: development
    SHOW: 'ture'
-------------------------
environment:
    - RACK_ENV=development
    - SHOW=ture
```

- **env_file**：从文件中获取环境变量，可以指定一个文件路径或路径列表，其优先级低于 environment 指定的环境变量

```yaml
env_file: .env
---------------
env_file:
    - ./common.env
```

- **expose**：暴露端口，只将端口暴露给连接的服务，而不暴露给主机

```yaml
expose:
    - "3000"
    - "8000"
```

- **image**：指定服务所使用的镜像

```yaml
image: java
```

- **network_mode**：设置网络模式

```yaml
network_mode: "bridge"
network_mode: "host"
network_mode: "none"
network_mode: "service:[service name]"
network_mode: "container:[container name/id]"
```

- **ports**：对外暴露的端口定义，和 expose 对应

```yaml
ports:   # 暴露端口信息  - "宿主机端口:容器暴露端口"
- "8763:8763"
- "8763:8763"
```

- **links**：将指定容器连接到当前连接，可以设置别名，避免ip方式导致的容器重启动态改变的无法连接情况

```yaml
links:    # 指定服务名称:别名 
    - docker-compose-eureka-server:compose-eureka
```

- **volumes**：卷挂载路径

```yaml
volumes:
  - /lib
  - /var
```

- **logs**：日志输出信息

```yaml
--no-color          单色输出，不显示其他颜.
-f, --follow        跟踪日志输出，就是可以实时查看日志
-t, --timestamps    显示时间戳
--tail              从日志的结尾显示，--tail=200
```

### Docker Compose 其它

### 更新容器

- 当服务的配置发生更改时，可使用 docker-compose up 命令更新配置
- 此时，Compose 会删除旧容器并创建新容器，新容器会以不同的 IP 地址加入网络，名称保持不变，任何指向旧容起的连接都会被关闭，重新找到新容器并连接上去

### links

- 服务之间可以使用服务名称相互访问，links 允许定义一个别名，从而使用该别名访问其它服务

```yaml
version: '2'
services:
    web:
        build: .
        links:
            - "db:database"
    db:
        image: postgres
```

- 这样 Web 服务就可以使用 db 或 database 作为 hostname 访问 db 服务了

### 官方示例

```yaml
# 
version: "3.7"
services:

  redis:
    image: redis:alpine
    ports:
      - "6379"
    networks:
      - frontend
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  db:
    image: postgres:9.4
    volumes:
      - db-data:/var/lib/postgresql/data
    networks:
      - backend
    deploy:
      placement:
        constraints: [node.role == manager]

  vote:
    image: dockersamples/examplevotingapp_vote:before
    ports:
      - "5000:80"
    networks:
      - frontend
    depends_on:
      - redis
    deploy:
      replicas: 2
      update_config:
        parallelism: 2
      restart_policy:
        condition: on-failure

  result:
    image: dockersamples/examplevotingapp_result:before
    ports:
      - "5001:80"
    networks:
      - backend
    depends_on:
      - db
    deploy:
      replicas: 1
      update_config:
        parallelism: 2
        delay: 10s
      restart_policy:
        condition: on-failure

  worker:
    image: dockersamples/examplevotingapp_worker
    networks:
      - frontend
      - backend
    deploy:
      mode: replicated
      replicas: 1
      labels: [APP=VOTING]
      restart_policy:
        condition: on-failure
        delay: 10s
        max_attempts: 3
        window: 120s
      placement:
        constraints: [node.role == manager]

  visualizer:
    image: dockersamples/visualizer:stable
    ports:
      - "8080:8080"
    stop_grace_period: 1m30s
    volumes:
      - "/var/run/docker.sock:/var/run/docker.sock"
    deploy:
      placement:
        constraints: [node.role == manager]

networks:
  frontend:
  backend:

volumes:
  db-data:
```



## Docker vagrant cluster

```bash
yum -y install '/root/下载/vagrant_2.2.5_x86_64.rpm'
git clone https://github.com/pires/kubernetes-vagrant-coreos-cluster
cd kubernetes-vagrant-coreos-cluster/
vagrant up
```

# Kubernetes

## minikube

```shell
# 安装minikube
curl -Lo minikube http://kubernetes.oss-cn-hangzhou.aliyuncs.com/minikube/releases/v1.2.0/minikube-linux-amd64 && chmod +x minikube && sudo mv minikube /usr/local/bin/
```



- `kubeadm`: 用来初始化集群的指令。
- `kubelet`: 在集群中的每个节点上用来启动 pod 和 container 等。
- `kubectl`: 用来与集群通信的命令行工具。

```shell
# 集群所有机器配置阿里云安装kubelet kubeadm kubectl
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF

sysctl --system

cat > /etc/sysconfig/modules/ipvs.modules <<EOF
#!/bin/bash
modprobe -- ip_vs
modprobe -- ip_vs_rr
modprobe -- ip_vs_wrr
modprobe -- ip_vs_sh
modprobe -- nf_conntrack_ipv4
EOF
chmod 755 /etc/sysconfig/modules/ipvs.modules && bash /etc/sysconfig/modules/ipvs.modules && lsmod | grep -e ip_vs -e nf_conntrack_ipv4
#　上面脚本创建了的/etc/sysconfig/modules/ipvs.modules文件，保证在节点重启后能自动加载所需模块。 使用lsmod | grep -e ip_vs -e nf_conntrack_ipv4命令查看是否已经正确加载所需的内核模块。

cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=https://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64/
enabled=1
gpgcheck=1
repo_gpgcheck=1
gpgkey=https://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg https://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
EOF

setenforce 0
yum install -y kubelet kubeadm kubectl
systemctl enable kubelet && systemctl start kubelet
```

## kubeadm

```shell
# 修改docker配置
vim /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "registry-mirrors": ["https://docker.mirrors.ustc.edu.cn/"]
}

# 配置master节点
mkdir -p /usr/local/docker/kubernetes
cd /usr/local/docker/kubernetes
# 集群初始化方法一,查看默认配置,生成配置文件,修改
kubeadm config print init-defaults --kubeconfig ClusterConfiguration > kubeadm.yaml

vim kubeadm.yaml
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
# 设置为master的ip
  advertiseAddress: 192.169.1.10
  bindPort: 6443
nodeRegistration:
  criSocket: /var/run/dockershim.sock
  name: master
  taints:
  - effect: NoSchedule
    key: node-role.kubernetes.io/master
---
apiServer:
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
# 修改为阿里云镜像url
imageRepository: registry.aliyun.com/google_containers
kind: ClusterConfiguration
# 注意版本,需要与 kubeadm 版本一致
kubernetesVersion: v1.15.0
networking:
  dnsDomain: cluster.local
  # 网络插件默认的网段
  # podSubnet: "192.168.0.0/16"
  serviceSubnet: 10.96.0.0/12
scheduler: {}

# 查看需要拉取的镜像
kubeadm config images list --config kubeadm.yaml
# 拉取镜像
kubeadm config images pull --config kubeadm.yaml
# 查看下载的镜像
docker images

# 集群初始化方法一,可以不拉取,软件会自动拉取镜像
kubeadm init --config kubeadm.yaml --ignore-preflight-errors=Swap 
# 集群初始化方法二,不修改配置文件
kubeadm init --image-repository registry.cn-hangzhou.aliyuncs.com/google_containers --pod-network-cidr=192.168.0.0/16 --kubernetes-version=v1.15.0 --apiserver-advertise-address=192.169.1.10


# 必须看到如下提示,才是安装初始化
Your Kubernetes control-plane has initialized successfully!

To start using your cluster, you need to run the following as a regular user:

  mkdir -p $HOME/.kube
  sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
  sudo chown $(id -u):$(id -g) $HOME/.kube/config
Then you can join any number of worker nodes by running the following on each as root:

kubeadm join 192.169.1.10:6443 --token abcdef.0123456789abcdef \
    --discovery-token-ca-cert-hash sha256:74c84483674b84c9db8137a73145e4c4f4de0c743879df273e47b89d9c0e1397
# 按照提示执行命令才能使用集群,以上最后一段命令用于slave加入集群
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
# 执行命令后,验证集群
kubectl get nodes
NAME     STATUS   ROLES    AGE   VERSION
master   Ready    master   12m   v1.15.0
# 停止集群
kubeadm reset
```

配置网络

每个群集只能安装一个Pod网络。

```shell
# master安装 flannel 网络插件
kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/62e44c867a2846fefb68bd5f178daf4da3095ccb/Documentation/kube-flannel.yml
# 查看集群网络状态,直到所有网络ready
watch kubectl get pods --all-namespaces
```

```shell
# 配置slave,修改好主机名后运行master节点上的提示命令
kubeadm join 192.169.1.10:6443 --token abcdef.0123456789abcdef --discovery-token-ca-cert-hash sha256:74c84483674b84c9db8137a73145e4c4f4de0c743879df273e47b89d9c0e1397

# 在master验证
kubectl get nodes

# 删除slave node
# 在slave 执行
kubeadm reset
# 在master 执行;替换 nodename
kubectl delete nodes nodename
```

配置检查

```shell
# 查看健康检查
kubectl get cs
NAME                 STATUS    MESSAGE             ERROR
scheduler            Healthy   ok                  
controller-manager   Healthy   ok                  
etcd-0               Healthy   {"health":"true"} 
# 查看集群信息
kubectl	cluster-info
Kubernetes master is running at https://192.169.1.10:6443
KubeDNS is running at https://192.169.1.10:6443/api/v1/namespaces/kube-system/services/kube-dns:dns/proxy

To further debug and diagnose cluster problems, use 'kubectl cluster-info dump'.

# 检查 node
kubectl get nodes
NAME     STATUS   ROLES    AGE     VERSION
master   Ready    master   3h30m   v1.15.0
slave1   Ready    <none>   164m    v1.15.0
slave2   Ready    <none>   142m    v1.15.0
```

运行第一个服务

```shell
kubectl run nginx --image=nginx --replicas=2 --port=80
# 查看已经部署的服务
kubectl get deployment
# 设置映射,让用户可以访问
kubectl expose deployment nginx --port=80 --type=LoadBalancer
# 查看开启的映射服务
kubectl get svc
# 查看映射服务详情
kubectl describe service nginx
# 删除deployment
kubectl delete deployment nginx
# 删除 service
kubectl delete service nginx
```

## yaml文件

创建资源的方法:

  apiserver 仅接收 JSON 格式的资源定义;

  aml 格式提供配置清单,apiserver 可自动将其转为json格式,然后再提交;

大部分资源的配置清单:

```yaml
apiVersion: group/version
    # $ kubectl api-versions
kind: 资源类别
matedata: 元数据
    name:
    namespace:
    labels:
    annotations:
      每个资源的引用PATH
        /api/GROUP/VERSION/namespaces/NAMESPACE/TYPE/NAME
spec: 期望的状态,disired state
status: 当前的状态,current state,本字段由kubernetes集群维护
```

更多key讲解

```yaml
imagePullPolicy: <string>
    Always,Never,IfNotPresent
    修改镜像中的默认应用:
    command,args
        https://kubernetes.io/docs/tasks/inject-data-application/define-command-argument-container/
标签:
    key=value
       key: 字母 数字 _ - . 五种 长度63字符
       value: 可以为空,长度63字符,只能字母数字开头,结尾
     
标签选择器:
    等值关系: =,==,!=
    集合关系:
        KEY in(VALUE1,VALUE2,...)
        KEY not in(VALUE1,VALUE2,...)
        KEY
        !KEY
许多资源支持内嵌字段定义其使用的标签选择器:
    matchlabel: 直接给定键值
    matchExpressions: 基于给定的表达式来定义使用标签选择器,{key:"KEY",operator:"OPEARTOR",value:[VALUE1,VALUE2,...]}
        操作符:
            In,NotIn: values字段的值必须为非空列表;
            Exists,NotExists: values字段的值必须为空列表;
                
nodeSelector: <map[string]string> 节点标签选择器
nodeName: <string>
annotations: 
   与label不同的地方在于,它不能用于挑选资源对象,仅用于为对象提供"元数据"。没有字符数量限制
Pod生命周期:
    状态: Pending(挂起),Running,Failed,Succeeded,Unknown
    创建Pod: 
    Pod生命周期中的重要行为:
        初始化容器
        容器探测:
            liveness
            readliness
restartPolicy:
    Always,OnFailure,Never.Default to Always.
    
探针类型有三种:
    ExecAction,TCPSockAction,HTTPGetAction
Pod控制器:
    ReplicaSet: 用户期望数,标签选择器,模板,简写rs,在任何给定时间维护一组稳定的副本Pod
    Deployment: 管理使用多个ReplicaSet
    DaemonSet: 能确保其创建的Pod在集群中的每一台（或指定）Node上都运行一个副本。
    Job: 临时任务
    CronJob: 
    StatefulSet:
    
Service:
    使用CoreDNS 1.11+
    node network,pod network,cluster network
    virtual IP
    工作在4层,有调度功能
    工作模式:
        userspace: 1.1-
        iptables: 1.10-
        ipvs: 1.11+
    类型:
        ExternalName:
        ClusterIP: 在群集内部公开的IP服务。选择此值使服务只能从群集中访问。
        NodePort: 在每个节点上公开的IP服务。可以在群集外部访问
        LoadBalancer: 使用云提供商的负载均衡器在外部公开服务
        NOdePort: client -> NodeIP: NodePort -> ClusterIP: ServicePort -> PodIP: containerPort
    资源记录:
        SVC_NAME.NS_NAME.DOMAIN.LTD.
        reids.default.svc.cluster.local.

Ingress and Ingress Controller:
    利用DaemonSet在指定Pod调度外部访问,工作在7层
    Ingress配置提供外部可访问的URL、负载均衡、SSL、基于名称的虚拟主机等。用户通过POST Ingress资源到API server的方式来请求ingress。 Ingress controller负责实现Ingress，通常使用负载平衡器，它还可以配置边界路由和其他前端，这有助于以HA方式处理流量。
    
存储卷:
    emptyDir: 临时目录
    hostPath: 主机目录
    ...
    pvc绑定pc
    
statefulset: 有状态应用副本集
    在具有以下特点时使用StatefulSets:
        稳定且唯一的网络标志符;
        稳定且持久的存储;
        有序,平滑的部署,扩展,删除和终止;
        有序的滚动更新;
    三个组件: headless service,Statefulset,volumeClaimTemplate
    名称: pod_name.service.ns_name.svc.cluster.local

客户端-->API server:
    user: username,uid
    group:
    extra:
    
    API
    Request path: 所有的资源数据 可以通过url获得
    HTTP request verb: get,post,put,delete
    API request verb: get,list,create,update,patch,watch,proxy,redirect,delete,deletecollection
    Object URL: /apis/<GROUP>/<VERSION>/namespaces/<NAMESPACE_NAME>/<KIND>[/OBJECT_ID]/
    Resource:
    Subresource:
    Namespace
    API group
    
    授权插件: Node,ABAC,RBAC(Role-based AC),Webhook

kubernetes网络通信:
    (1)容器间通信: 同一Pod内的多个容器间通信,lo
    (2)Pod通信: Pod IP <-> Pod IP
    (3)Pod与Service通信: Pod IP <-> Cluster IP
    (4)Service与集群外部客户端通信:
    
CNI:
    flannel
    calico
    canel
    kube-router
    ...
    解决方案:
        虚拟网桥
        多路复用: MacVLAN
        硬件交换: SR-IOV
    kubelet, /etc/cni/net.d/
    flannel:
        支持多种后端:
            VxLAN:
                (1) vxlan
                (2) Directrouting
            host-gw: Host Gateway
            UDP:
        flannel 的网络配置参数:
            NetWork: flannel 使用的CIDR格式的网络地址,用于为Pod配置网络功能;
                10.244.0.0/16 ->
            SubnetLen: 把NetWork切分子网供各节点使用,使用多长的掩码进行切分,默认24位;
            SubnetMin: 起始子网 如:10.244.10.0/24
            SubnetMax: 结尾子网 如:10.244.100.0/24
            Backend: vxlan,host-gw,udp
                vxlan: 
Scheduler 提供的调度流程分三步:
    预选策略(predicate) 遍历nodelist，选择出符合要求的候选节点，Kubernetes内置了多种预选规则供用户选择。
    优选策略(priority) 在选择出符合要求的候选节点中，采用优选规则计算出每个节点的积分，最后选择得分最高的。
    选定(select) 如果最高得分有好几个节点，select就会从中随机选择一个节点。
调度器:
    预选策略:
        CheckNodeCondition:
        GerneralPredicates:
            HostName: 检查Pod对象是否定义了pod.spec.hostname,
            PodFitsHostPorts: pod.spec.containers.ports.hostPort
            MatchNodeSelector: pod.spec.nodeSelector
            PodFitsResources: 检查Pod的资源需求是否能被节点所满足.
        NoDiskConflict: 检查Pod依赖的存储卷是否能满足需求
        POdToleratesNodeTaints: 检查Pod上的spec.tolerations可容忍的污点是否完全包含节点上的污点;

资源限制:
    requests: 需求,最低保障;
    limits: 限制,硬限制;
    
    CPU:
        1颗逻辑CPU=1000 millicores
        500m = 0.5CPU
    Mem: E,P,T,G,M,K,Ei,Pi
    QoS: # 资源类别
        Guranteed: 每个容器同时设置CPU和内存的requests和limits.并且相等,最高优先级别.
        Burstable: 至少有一个容器设置CPU或内存资源的requests属性.
        BestEffort: 没有任何一个容器设置了requests或limits属性,最低优先级别.
    
    HeapSter: Heapster是容器集群监控和性能分析工具，天然的支持Kubernetes和CoreOS。
              Kubernetes有个出名的监控agent—cAdvisor。在每个kubernetes Node上都会运行cAdvisor，它会收集本机以及容器的监控数据(cpu,memory,filesystem,network,uptime)。在较新的版本中，K8S已经将cAdvisor功能集成到kubelet组件中。每个Node节点可以直接进行web访问。

资源指标API: metrics-server
自定义指标API: prometheus,k8s-prometheus-adapter

新一代指标架构:
   核心指标流水线: 由kubelet,metrics-server以及由APIserver提供的api组成;CPU累积使用率,内存实时使用率,Pod的资源占用率以及容器的磁盘占用率;
   监控流水线: 用于从系统收集各种指标数据并提供终端用户,存储系统以及HPA(Horizontal Pod Autoscaling，即pod的水平自动扩展),包含核心指标与非核心指标,非核心指标本身不能被k8s解析.
   metrics-server: 第三方
           
Helm: 类似yum,管理k8s源,可以快速部署.工作在集群之外,是客户端.
    组件:
      chart(配置,相当于rpm,部署后叫release),tiller(服务端,集群内)
      Repository: Charts仓库
      Release: 部署于集群上的实例
      
      Chart -- config --> Release
      
      程序架构:
          helm: 客户端,管理本地的Chart仓库,管理Chart,与Tiller服务器交互,发送Chart,实例安装,查询,卸载等操作.
          Tiller: 服务端,接收helm发来的Charts与config,合并生成release.
      release管理:
          install
          delete
          upgrade/rollbcak
          list
          history: release的历史信息;
          status: 获得release的状态信息;
      
      chart管理:
          create
          fetch
          get
          inspect
          package
          verify
```

### 创建第一个demo

```yaml
vim demo.yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-demo
    namespace: default
    labels:
        app: myapp
        tier: frontend
spec:
    containers:
        - name: myapp
          image: ikubernetes/myapp:v1
        - name: busybox
          image: busybox:latest
          command:    # 指定CMD       
              - "/bin/sh"
              - "-c"
              - "sleep 5"

kubectl create -f demo.yaml
```

### 改进demo.yaml

```yaml
vim demo.yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-demo
    namespace: default
    labels:
        app: myapp
        tier: frontend
    annotations:
        qlst.top/created-by: "Archangel" # 新增元数据
spec:
    containers:
        - name: myapp
          image: ikubernetes/myapp:v1
          ports:     # 新增
              - name: http
                containerPort: 80
              - name: https
                containerPort: 443
        - name: busybox
          image: busybox:latest
          imagePullPolicy: IfNotPresent  # 新增
          command:     
              - "/bin/sh"
              - "-c"
              - "sleep 5"
    nodeSelector:    # 新增,指定只在符合标签条件的node运行
        disktype: ssd

kubectl create -f demo.yaml
```

### liveness-exec.yaml

```yaml
vim liveness-exec.yaml
apiVersion: v1
kind: Pod
metadata:
    name: liveness-exec-demo
    namespace: default
spec:
    containers:
    - name: liveness-exec-container
      images: busy-box:latest
      imagePullPolicy: IfNotPresent
      command: ["bin/sh","-c","touch /tmp/healthy;sleep 30;rm -rf /tmp/healthy;sleep 60;"]
      livenessProbe:
          exec:
              command: ["test","-e","/tmp/healthy"]
          initialDelaySecond: 1
          periodSeconds: 3

kubectl create -f liveness-exec.yaml
```

### liveness-httpget.yaml

```yaml
vim liveness-httpget.yaml
apiVersion: v1
kind: Pod
metadata:
    name: liveness-httpget-demo
    namespace: default
spec:
    containers:
    - name: liveness-httpget-container
      images: ikubernetes/myapp:v1
      imagePullPolicy: IfNotPresent
      ports:
          - name: http
            containerPort: 80
      livenessProbe:
          httpGet:
              port: http
              path: /index.html
          initialDelaySecond: 1
          periodSeconds: 3

kubectl create -f liveness-httpget.yaml
```

### readiness-httpget.yaml

```yaml
vim readiness-httpget.yaml
apiVersion: v1
kind: Pod
metadata:
    name: readiness-httpget-demo
    namespace: default
spec:
    containers:
    - name: readiness-httpget-container
      images: ikubernetes/myapp:v1
      imagePullPolicy: IfNotPresent
      ports:
          - name: http
            containerPort: 80
      livenessProbe:
          httpGet:
              port: http
              path: /index.html
          initialDelaySecond: 1
          periodSeconds: 3

kubectl create -f readiness-httpget.yaml
```

### poststart-pod.yaml

```yaml
vim poststart-pod.yaml
apiVersion: v1
kind: Pod
metadata:
    name: poststart-pod
    namespace: default
spec:
    containers:
    - name: busybox-httpd
      images: busybox:latest
      imagePullPolicy: IfNotPresent
      lifecycle:  
          postStart: # 启动过程中执行
              exec:
                  command: ["bin/sh","-c","mkdir -p /data/web/html;echo 'Hello world' >> /data/web/html/index.html"]
      command: ["bin/sh","-c","sleep 30"] # 启动时执行此命令

kubectl create -f poststart-pod.yaml
```

### rs-demo.yaml

```yaml
apiVersion: apps/v1
kind: ReplicaSet
metadata:
    name: myapp
    namespace: default
spec:
    replicas: 2
    selector:
        matchLabels:
            app: myapp
            release: canary
    template:
        metadata:
            name: myapp-pod
            labels:
                app: myapp
                release: canary
        spec:
            containers:
                - name: myapp-container
                  image: ikubernetes/myapp:v1
                  ports:
                      - name: http
                        containerPort: 80
```

### deploy-demo.yaml

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
    name: redis
    namespace: default
spec:
    replicas: 1
    selector:
        matchLabels:
            app: redis
            roll: logsto
    template:
        matchLabels:
            labels:
                app: redis
                roll: logsto
        spec:
            containers:
                - name: redis
                  image: redis:4.0
                  ports:
                      - name: redis
                        containerPort: 6379
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: myapp-deploy
    namespace: default
spec:
    replicas: 2
    selector:
        matchLabels:
            app: myapp
            release: canary
    template:
        metadata:
            name: myapp-pod
            labels:
                app: myapp
                release: canary
        spec:
            containers:
                - name: myapp-container
                  image: ikubernetes/myapp:v1
                  ports:
                      - name: http
                        containerPort: 80
```

### redis-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
    name: redis
    namespace: default
spec:
    selector:   # 与redis标签对应
        app: redis
        roll: logsto 
    clusterIP: 10.97.97.97
    type: ClusterIP
    ports:
        - port: 6379
          targetPort: 6379
```

### myapp-svc.yaml

```yaml
apiVersion: v1
kind: Service
metadata:
    name: myapp
    namespace: default
spec:
    selector:   # 与应用标签对应
        app: myapp
        release: canary 
    clusterIP: 10.99.99.99 # 不指定自动分配
    sessionAffinity: ClientIP # 指定相同IP转发至相同Pod,默认None
    type: NodePort
    ports:
        - port: 80
          targetPort: 80
          nodePort: 30080 # 不指定自动分配
```

### myapp-svc-headless.yaml

有时您不需要负载平衡和单个服务IP。在这种情况下，您可以通过显式指定`"None"`群集IP（`.spec.clusterIP`）来创建所谓的“无头”服务。会显示所有podip

```yaml
apiVersion: v1
kind: Service
metadata:
    name: myapp-svc
    namespace: default
spec:
    selector:   # 与应用标签对应
        app: myapp
        release: canary 
    clusterIP: None # 指定无头服务
    sessionAffinity: ClientIP # 指定相同IP转发至相同Pod,默认None
    type: NodePort
    ports:
        - port: 80
          targetPort: 80
          nodePort: 30080 # 不指定自动分配
```



## kubectl

```shell
# kubectl 另一种安装方式
curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl

chmod +x ./kubectl

mv ./kubectl /usr/local/bin/kubectl

# Test to ensure the version you installed is up-to-date:
kubectl version
 
# kubectl 命令行补全
kubectl completion  -h
```

## Pod

### explain pods

查看pod如何定义

```shell
$ kubectl explain pods
$ kubectl explain pods.metadata
```

### create pod

create pod through yml file

```shell
$ kubectl create -f nginx_busybox.yml
pod "nginx-busybox" created
```

### get pod list

```shell
$ kubectl get pods
NAME            READY     STATUS    RESTARTS   AGE
nginx-busybox   2/2       Running   0          57s

$ kubectl get pods -l app # 只显示key为app的pod
```

### get pod detail

we use use describe or get pod with `-o` option to display more information about this pod

```shell
$ kubectl describe pod nginx-busybox
$ kubectl get pods nginx-busybox -o wide
NAME            READY     STATUS    RESTARTS   AGE       IP           NODE
nginx-busybox   2/2       Running   0          3m        172.17.0.4   minikube
```

here `-o` we can use `wide`, `json`, `yaml`, etc.

### get into pod(container)

```shell
$ kubectl exec nginx-busybox -c nginx -it sh
Defaulting container name to nginx.
Use 'kubectl describe pod/nginx-busybox -n default' to see all of the containers in this pod.
```

### logs

```shell
$ kubectl logs nginx-busybox nginx # 查看访问日志
```

### delete pod

```shell
$ kubectl delete -f nginx_busybox.yml
pod "nginx-busybox" deleted
```

## Label

```shell
$ kubectl label pods nginx-busybox release=canary # 添加标签
$ kubectl label pods nginx-busybox release=stable --overwrite # 修改已有标签
```



## Namespace

### get all namesapce

```shell
$ kubectl get namespace
NAME          STATUS    AGE
default       Active    1d
kube-public   Active    1d
kube-system   Active    1d
```

### create namespace

```shell
% kubectl create namespace demo
namespace "demo" created
```

### delete namespace

Please makesure there are no resources within this namespace you want to delete.

```shell
$ kubectl delete namespace demo
namespace "demo" deleted
```

### create pod with namespace

```shell
$ more nginx_namespace.yml
apiVersion: v1
kind: Pod
metadata:
  name: nginx
  namespace: demo
spec:
  containers:
  - name: nginx
    image: nginx
    ports:
    - containerPort: 80
$ 
$ kubectl create -f nginx_namespace.yml
Error from server (NotFound): error when creating "nginx_namespace.yml": namespaces "demo" not found
$ kubectl create namespace demo
namespace "demo" created
$ kubectl create -f nginx_namespace.yml
pod "nginx" created
```

### get pod with namespace

```shell
$ kubectl get pod --namespace demo
NAME      READY     STATUS    RESTARTS   AGE
nginx     1/1       Running   0          6m
```

## create our own context

```shell
$ kubectl config set-context demo --user=minikube --cluster=minikube --namespace=demo
Context "demo" created.
$ kubectl config get-contexts
CURRENT   NAME       CLUSTER    AUTHINFO   NAMESPACE
          demo       minikub    minikube   demo
*         minikube   minikube   minikube
$ kubectl config use-context demo
Switched to context "demo".
```

Now we create a pod without namespace will in demo namespace by default.

## Deployment

自动管理pods

### Create a Deployment

```shell
$ kubectl create -f nginx_depolyment.yml
deployment.apps "nginx-deployment" created
```

### Get deployment

Get deployment and pod information

```shell
$ kubectl get deployment
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE
nginx-deployment   2         2         2            2           12s
$ kubectl get deployment -o wide
NAME               DESIRED   CURRENT   UP-TO-DATE   AVAILABLE   AGE       CONTAINERS   IMAGES        SELECTOR
nginx-deployment   2         2         2            2           40s       nginx        nginx:1.7.9   app=nginx
$ kubectl get pod -l app=nginx
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-75675f5897-bwz4j   1/1       Running   0          1m
nginx-deployment-75675f5897-z626w   1/1       Running   0          1m
```

also can use describe

```shell
$ kubectl describe deployment nginx-deployment
Name:                   nginx-deployment
Namespace:              default
CreationTimestamp:      Tue, 26 Jun 2018 23:15:17 +0800
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision=1
Selector:               app=nginx
Replicas:               2 desired | 2 updated | 2 total | 2 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.7.9
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-75675f5897 (2/2 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  3m    deployment-controller  Scaled up replica set nginx-deployment-75675f5897 to 2
```

### keep desired pod state

```shell
$ kubectl get pod -l app=nginx
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-75675f5897-bwz4j   1/1       Running   0          3m
nginx-deployment-75675f5897-z626w   1/1       Running   0          3m
$ kubectl delete pod nginx-deployment-75675f5897-bwz4j
pod "nginx-deployment-75675f5897-bwz4j" deleted
$ kubectl get pod -l app=nginx
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-75675f5897-6fdvh   1/1       Running   0          <invalid>
nginx-deployment-75675f5897-z626w   1/1       Running   0          4m
$ kubectl get pod -l app=nginx
NAME                                READY     STATUS    RESTARTS   AGE
nginx-deployment-75675f5897-6fdvh   1/1       Running   0          0s
nginx-deployment-75675f5897-z626w   1/1       Running   0          4m
```

### Updating the deployment

```shell
apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
kind: Deployment
metadata:
  name: nginx-deployment
spec:
  selector:
    matchLabels:
      app: nginx
  replicas: 2
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: nginx
        image: nginx:1.8 # Update the version of nginx from 1.7.9 to 1.8
        ports:
        - containerPort: 80
        
$ kubectl apply -f nginx_depolyment_update.yml
depolyment "nginx-deployment" updated
```

### Scaling the application by increasing the replica count

```shell
$ kubectl apply -f nginx_depolyment_scale.yml
depolyment "nginx-deployment" updated
$ kubectl get pods -l app=nginx
NAME                               READY     STATUS    RESTARTS   AGE
nginx-deployment-148880595-4zdqq   1/1       Running   0          25s
nginx-deployment-148880595-6zgi1   1/1       Running   0          25s
nginx-deployment-148880595-fxcez   1/1       Running   0          2m
nginx-deployment-148880595-rwovn   1/1       Running   0          2m
```

### Delete a Deployment

```shell
$ kubectl delete deployment nginx-deployment
depolyment "nginx-deployment" deleted
```

## Replicaset

### create deployment

```
$ kubectl apply -f nginx_deployment.yml
deployment.apps "nginx-deployment-test" created
```

describe and get the relicaset event

```
$ kubectl describe deployment nginx-deployment-test
Name:                   nginx-deployment-test
Namespace:              default
CreationTimestamp:      Wed, 27 Jun 2018 00:51:10 +0800
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision=1
                        kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"nginx-deployment-test","namespace":"default"},"spec":{"replicas":4,"se...
Selector:               app=nginx
Replicas:               4 desired | 4 updated | 4 total | 4 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.7.9
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Available      True    MinimumReplicasAvailable
  Progressing    True    NewReplicaSetAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-test-75675f5897 (4/4 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  8h    deployment-controller  Scaled up replica set nginx-deployment-test-75675f5897 to 4
```

### scale and check the replicaset event

```
$ kubectl scale --current-replicas=4 --replicas=6 deployment/nginx-deployment-test
deployment.extensions "nginx-deployment-test" scaled
$ kubectl describe deployment nginx-deployment-test
Name:                   nginx-deployment-test
Namespace:              default
CreationTimestamp:      Wed, 27 Jun 2018 00:51:10 +0800
Labels:                 <none>
Annotations:            deployment.kubernetes.io/revision=1
                        kubectl.kubernetes.io/last-applied-configuration={"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"nginx-deployment-test","namespace":"default"},"spec":{"replicas":4,"se...
Selector:               app=nginx
Replicas:               6 desired | 6 updated | 6 total | 6 available | 0 unavailable
StrategyType:           RollingUpdate
MinReadySeconds:        0
RollingUpdateStrategy:  25% max unavailable, 25% max surge
Pod Template:
  Labels:  app=nginx
  Containers:
   nginx:
    Image:        nginx:1.7.9
    Port:         80/TCP
    Host Port:    0/TCP
    Environment:  <none>
    Mounts:       <none>
  Volumes:        <none>
Conditions:
  Type           Status  Reason
  ----           ------  ------
  Progressing    True    NewReplicaSetAvailable
  Available      True    MinimumReplicasAvailable
OldReplicaSets:  <none>
NewReplicaSet:   nginx-deployment-test-75675f5897 (6/6 replicas created)
Events:
  Type    Reason             Age   From                   Message
  ----    ------             ----  ----                   -------
  Normal  ScalingReplicaSet  8h    deployment-controller  Scaled up replica set nginx-deployment-test-75675f5897 to 4
  Normal  ScalingReplicaSet  8h    deployment-controller  Scaled up replica set nginx-deployment-test-75675f5897 to 6
```

### check rollout status

```
kubectl rollout status deployment nginx-deployment-test
```

rollout history

```
$ kubectl rollout history deployment nginx-deployment-test
deployments "nginx-deployment-test"
REVISION  CHANGE-CAUSE
1         <none>
2         <none>
$ kubectl rollout history deployment nginx-deployment-test --revision 2
deployments "nginx-deployment-test" with revision #2
Pod Template:
  Labels:	app=nginx
	pod-template-hash=703038527
  Containers:
   nginx:
    Image:	nginx:1.9.1
    Port:	80/TCP
    Host Port:	0/TCP
    Environment:	<none>
    Mounts:	<none>
  Volumes:	<none>
```

### rollout undo

```
$ kubectl rollout undo deployment nginx-deployment-test
deployment.apps "nginx-deployment-test"
```

## Kubernetes Service ClusterIP

Kubernetes的service有三种类型：ClusterIP，NodePort，LoadBalancer，今天我们来看看ClusterIP。

### 创建Deployment

首先我们先创建一个Deployment，这个Deployment是一个Python实现的HTTP服务，请求这个Web Server的时候，会发回给我们这个server的hostname（如果是container，那就是container的hostname）。

这个Deployment有四个Replica。

```shell
$ more deployment_python_http.yml
apiVersion:  apps/v1
kind: Deployment
metadata:
  name: service-test
spec:
  replicas: 4
  selector:
    matchLabels:
      app: service_test_pod
  template:
    metadata:
      labels:
        app: service_test_pod
    spec:
      containers:
      - name: simple-http
        image: python:2.7
        imagePullPolicy: IfNotPresent
        command: ["/bin/bash"]
        args: ["-c", "echo \"<p>Hello from $(hostname)</p>\" > index.html; python -m SimpleHTTPServer 8080"]
        ports:
        - name: http
          containerPort: 8080
$ kubectl create -f deployment_python_http.yml
deployment.apps "service-test" created
```

创建完我们看到pod是这样的；

```shell
$ kubectl get pod -o wide
NAME                            READY     STATUS    RESTARTS   AGE       IP          NODE
service-test-54b5b4b547-8l9s4   1/1       Running   0          1m        10.36.0.0   ks8-node2
service-test-54b5b4b547-c2t85   1/1       Running   0          1m        10.36.0.1   ks8-node2
service-test-54b5b4b547-nxn9z   1/1       Running   0          1m        10.40.0.1   k8s-node1
service-test-54b5b4b547-vlpff   1/1       Running   0          1m        10.40.0.0   k8s-node1
```

这四个pod IP我们都可以在k8s cluster任意一个节点上访问,每一个都会返回自己的container name。

```shell
$ curl 10.36.0.0:8080
<p>Hello from service-test-54b5b4b547-8l9s4</p>
```

### 创建Service

通过kubectl expose给刚才这个deployment创建一个service，端口绑定为8088.

```shell
kubectl expose deployment service-test --port 8088 --target-port=8080
service "service-test" exposed
```

这样，就给我们生成了一个类型为ClusterIP的service，这个service有一个Cluster IP，其实就一个VIP。

```shell
kubectl get service -o wide
NAME           TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE       SELECTOR
kubernetes     ClusterIP   10.96.0.1       <none>        443/TCP    1d        <none>
service-test   ClusterIP   10.101.90.210   <none>        8088/TCP   11s       app=service_test_pod
```

首先我们可以通过这个Cluster IP加端口8088访问我们的deployment。

```shell
$ for i in `seq 4`; do curl 10.101.90.210:8088; done
<p>Hello from service-test-54b5b4b547-nxn9z</p>
<p>Hello from service-test-54b5b4b547-vlpff</p>
<p>Hello from service-test-54b5b4b547-vlpff</p>
<p>Hello from service-test-54b5b4b547-nxn9z</p>
```

并且我们发现，这个VIP实现了负载均衡，每次返回的hostname不同。

### VIP和负载均衡的实现

为什么我们访问VIP就能访问我们的四个pod，并且还做了负载均衡呢？下面我们就看一下，

其实呢，我们刚才创建这个deployment，service的时候，k8s集群下面的几个部件参与了相关的工作。

- apiserver kubectl命令向apiserver发送创建service的命令，apiserver接收到请求以后将数据存储到etcd中。
- kube-proxy kubernetes的每个节点中都有一个叫做kube-proxy的进程，这个进程负责感知service，pod的变化，并将变化的信息写入本地的iptables中。
- iptables 使用NAT等技术将virtualIP的流量转至endpoint中。

那么IPtable到底是如何转发我们的流量的呢，我们到任意一台k8s节点运行 `sudo iptables -L -v -n -t nat`

首先找到我们的ClusterIP 10.101.90.210,发现他在一个iptables chain中

```shell
Chain KUBE-SERVICES (2 references)
 pkts bytes target     prot opt in     out     source               destination
    0     0 KUBE-MARK-MASQ  tcp  --  *      *      !172.100.0.0/16       10.101.90.210        /* default/service-test: cluster IP */ tcp dpt:8088
    0     0 KUBE-SVC-LY73ZDGF4KGO4YFJ  tcp  --  *      *       0.0.0.0/0            10.101.90.210        /* default/service-test: cluster IP */ tcp dpt:8088
```

这个chain类似一个链条，那么访问10.101.90.210:8088的流量到底怎么被转发了呢，我们需要看一下 `KUBE-SVC-LY73ZDGF4KGO4YFJ` 这个Chain， 找到这个chain

```shell
Chain KUBE-SVC-LY73ZDGF4KGO4YFJ (1 references)
 pkts bytes target     prot opt in     out     source               destination
    0     0 KUBE-SEP-YOQWVZZ4NQDNEBVN  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/service-test: */ statistic mode random probability 0.25000000000
    0     0 KUBE-SEP-WHOFXZ2VQXEUUKVO  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/service-test: */ statistic mode random probability 0.33332999982
    0     0 KUBE-SEP-3TBKTCTGJZ27RFOH  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/service-test: */ statistic mode random probability 0.50000000000
    0     0 KUBE-SEP-6LDVTIHDBDOU3D3G  all  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/service-test: */
```

这个Chain比较有意思，过来的流量它按照随机的概率，分别以0.25000000000， 0.33332999982，0.50000000000的概率，转发到这三个Chain，这三个Chain其实就是我们的pod，那为啥有一个pod不会被转发呢，这个应该是负载均衡的配置，最大几个负载均衡的问题。

我们随便拿出一个

```shell
Chain KUBE-SEP-WHOFXZ2VQXEUUKVO (1 references)
 pkts bytes target     prot opt in     out     source               destination
    0     0 KUBE-MARK-MASQ  all  --  *      *       10.36.0.1            0.0.0.0/0            /* default/service-test: */
    0     0 DNAT       tcp  --  *      *       0.0.0.0/0            0.0.0.0/0            /* default/service-test: */ tcp to:10.36.0.1:8080
```

好的，那经过这么一转发，我们的流量就可以转发到正确的pod上了，不知道大家明白没有。

## Liveness and Readiness Probes

### Define a liveness command

Many applications running for long periods of time eventually transition to broken states, and cannot recover except by being restarted. Kubernetes provides liveness probes to detect and remedy such situations.

In this exercise, you create a Pod that runs a Container based on the `k8s.gcr.io/busybox` image. Here is the configuration file for the Pod:

| [`pods/probe/exec-liveness.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/pods/probe/exec-liveness.yaml) ![Copy pods/probe/exec-liveness.yaml to clipboard](https://d33wubrfki0l68.cloudfront.net/951ae1fcc65e28202164b32c13fa7ae04fab4a0b/b77dc/images/copycode.svg) |
| ------------------------------------------------------------ |
| `apiVersion: v1 kind: Pod metadata:   labels:     test: liveness   name: liveness-exec spec:   containers:   - name: liveness     image: k8s.gcr.io/busybox     args:     - /bin/sh     - -c     - touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600     livenessProbe:       exec:         command:         - cat         - /tmp/healthy       initialDelaySeconds: 5       periodSeconds: 5 ` |

In the configuration file, you can see that the Pod has a single Container. The `periodSeconds` field specifies that the kubelet should perform a liveness probe every 5 seconds. The `initialDelaySeconds` field tells the kubelet that it should wait 5 second before performing the first probe. To perform a probe, the kubelet executes the command `cat /tmp/healthy`in the Container. If the command succeeds, it returns 0, and the kubelet considers the Container to be alive and healthy. If the command returns a non-zero value, the kubelet kills the Container and restarts it.

When the Container starts, it executes this command:

```shell
/bin/sh -c "touch /tmp/healthy; sleep 30; rm -rf /tmp/healthy; sleep 600"
```

For the first 30 seconds of the Container’s life, there is a `/tmp/healthy` file. So during the first 30 seconds, the command `cat /tmp/healthy` returns a success code. After 30 seconds, `cat /tmp/healthy` returns a failure code.

Create the Pod:

```shell
kubectl apply -f https://k8s.io/examples/pods/probe/exec-liveness.yaml
```

Within 30 seconds, view the Pod events:

```shell
kubectl describe pod liveness-exec
```

The output indicates that no liveness probes have failed yet:

```shell
FirstSeen    LastSeen    Count   From            SubobjectPath           Type        Reason      Message
--------- --------    -----   ----            -------------           --------    ------      -------
24s       24s     1   {default-scheduler }                    Normal      Scheduled   Successfully assigned liveness-exec to worker0
23s       23s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Pulling     pulling image "k8s.gcr.io/busybox"
23s       23s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Pulled      Successfully pulled image "k8s.gcr.io/busybox"
23s       23s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Created     Created container with docker id 86849c15382e; Security:[seccomp=unconfined]
23s       23s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Started     Started container with docker id 86849c15382e
```

After 35 seconds, view the Pod events again:

```shell
kubectl describe pod liveness-exec
```

At the bottom of the output, there are messages indicating that the liveness probes have failed, and the containers have been killed and recreated.

```shell
FirstSeen LastSeen    Count   From            SubobjectPath           Type        Reason      Message
--------- --------    -----   ----            -------------           --------    ------      -------
37s       37s     1   {default-scheduler }                    Normal      Scheduled   Successfully assigned liveness-exec to worker0
36s       36s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Pulling     pulling image "k8s.gcr.io/busybox"
36s       36s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Pulled      Successfully pulled image "k8s.gcr.io/busybox"
36s       36s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Created     Created container with docker id 86849c15382e; Security:[seccomp=unconfined]
36s       36s     1   {kubelet worker0}   spec.containers{liveness}   Normal      Started     Started container with docker id 86849c15382e
2s        2s      1   {kubelet worker0}   spec.containers{liveness}   Warning     Unhealthy   Liveness probe failed: cat: can't open '/tmp/healthy': No such file or directory
```

Wait another 30 seconds, and verify that the Container has been restarted:

```shell
kubectl get pod liveness-exec
```

The output shows that `RESTARTS` has been incremented:

```shell
NAME            READY     STATUS    RESTARTS   AGE
liveness-exec   1/1       Running   1          1m
```

### Define a liveness HTTP request

Another kind of liveness probe uses an HTTP GET request. Here is the configuration file for a Pod that runs a container based on the `k8s.gcr.io/liveness` image.

| [`pods/probe/http-liveness.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/pods/probe/http-liveness.yaml) ![Copy pods/probe/http-liveness.yaml to clipboard](https://d33wubrfki0l68.cloudfront.net/951ae1fcc65e28202164b32c13fa7ae04fab4a0b/b77dc/images/copycode.svg) |
| ------------------------------------------------------------ |
| `apiVersion: v1 kind: Pod metadata:   labels:     test: liveness   name: liveness-http spec:   containers:   - name: liveness     image: k8s.gcr.io/liveness     args:     - /server     livenessProbe:       httpGet:         path: /healthz         port: 8080         httpHeaders:         - name: Custom-Header           value: Awesome       initialDelaySeconds: 3       periodSeconds: 3 ` |

In the configuration file, you can see that the Pod has a single Container. The `periodSeconds` field specifies that the kubelet should perform a liveness probe every 3 seconds. The `initialDelaySeconds` field tells the kubelet that it should wait 3 seconds before performing the first probe. To perform a probe, the kubelet sends an HTTP GET request to the server that is running in the Container and listening on port 8080. If the handler for the server’s `/healthz` path returns a success code, the kubelet considers the Container to be alive and healthy. If the handler returns a failure code, the kubelet kills the Container and restarts it.

Any code greater than or equal to 200 and less than 400 indicates success. Any other code indicates failure.

You can see the source code for the server in [server.go](https://github.com/kubernetes/kubernetes/blob/master/test/images/agnhost/liveness/server.go).

For the first 10 seconds that the Container is alive, the `/healthz` handler returns a status of 200. After that, the handler returns a status of 500.

```go
http.HandleFunc("/healthz", func(w http.ResponseWriter, r *http.Request) {
    duration := time.Now().Sub(started)
    if duration.Seconds() > 10 {
        w.WriteHeader(500)
        w.Write([]byte(fmt.Sprintf("error: %v", duration.Seconds())))
    } else {
        w.WriteHeader(200)
        w.Write([]byte("ok"))
    }
})
```

The kubelet starts performing health checks 3 seconds after the Container starts. So the first couple of health checks will succeed. But after 10 seconds, the health checks will fail, and the kubelet will kill and restart the Container.

To try the HTTP liveness check, create a Pod:

```shell
kubectl apply -f https://k8s.io/examples/pods/probe/http-liveness.yaml
```

After 10 seconds, view Pod events to verify that liveness probes have failed and the Container has been restarted:

```shell
kubectl describe pod liveness-http
```

In releases prior to v1.13 (including v1.13), if the environment variable `http_proxy` (or `HTTP_PROXY`) is set on the node where a pod is running, the HTTP liveness probe uses that proxy. In releases after v1.13, local HTTP proxy environment variable settings do not affect the HTTP liveness probe.

### Define a TCP liveness probe

A third type of liveness probe uses a TCP Socket. With this configuration, the kubelet will attempt to open a socket to your container on the specified port. If it can establish a connection, the container is considered healthy, if it can’t it is considered a failure.

| [`pods/probe/tcp-liveness-readiness.yaml`](https://raw.githubusercontent.com/kubernetes/website/master/content/en/examples/pods/probe/tcp-liveness-readiness.yaml) ![Copy pods/probe/tcp-liveness-readiness.yaml to clipboard](https://d33wubrfki0l68.cloudfront.net/951ae1fcc65e28202164b32c13fa7ae04fab4a0b/b77dc/images/copycode.svg) |
| ------------------------------------------------------------ |
| `apiVersion: v1 kind: Pod metadata:   name: goproxy   labels:     app: goproxy spec:   containers:   - name: goproxy     image: k8s.gcr.io/goproxy:0.1     ports:     - containerPort: 8080     readinessProbe:       tcpSocket:         port: 8080       initialDelaySeconds: 5       periodSeconds: 10     livenessProbe:       tcpSocket:         port: 8080       initialDelaySeconds: 15       periodSeconds: 20 ` |

As you can see, configuration for a TCP check is quite similar to an HTTP check. This example uses both readiness and liveness probes. The kubelet will send the first readiness probe 5 seconds after the container starts. This will attempt to connect to the `goproxy` container on port 8080. If the probe succeeds, the pod will be marked as ready. The kubelet will continue to run this check every 10 seconds.

In addition to the readiness probe, this configuration includes a liveness probe. The kubelet will run the first liveness probe 15 seconds after the container starts. Just like the readiness probe, this will attempt to connect to the `goproxy` container on port 8080. If the liveness probe fails, the container will be restarted.

To try the TCP liveness check, create a Pod:

```shell
kubectl apply -f https://k8s.io/examples/pods/probe/tcp-liveness-readiness.yaml
```

After 15 seconds, view Pod events to verify that liveness probes:

```shell
kubectl describe pod goproxy
```

### Use a named port

You can use a named [ContainerPort](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#containerport-v1-core) for HTTP or TCP liveness checks:

```yaml
ports:
- name: liveness-port
  containerPort: 8080
  hostPort: 8080

livenessProbe:
  httpGet:
    path: /healthz
    port: liveness-port
```

### Define readiness probes

Sometimes, applications are temporarily unable to serve traffic. For example, an application might need to load large data or configuration files during startup, or depend on external services after startup. In such cases, you don’t want to kill the application, but you don’t want to send it requests either. Kubernetes provides readiness probes to detect and mitigate these situations. A pod with containers reporting that they are not ready does not receive traffic through Kubernetes Services.

> **Note:** Readiness probes runs on the container during its whole lifecycle.

Readiness probes are configured similarly to liveness probes. The only difference is that you use the `readinessProbe` field instead of the `livenessProbe` field.

```yaml
readinessProbe:
  exec:
    command:
    - cat
    - /tmp/healthy
  initialDelaySeconds: 5
  periodSeconds: 5
```

Configuration for HTTP and TCP readiness probes also remains identical to liveness probes.

Readiness and liveness probes can be used in parallel for the same container. Using both can ensure that traffic does not reach a container that is not ready for it, and that containers are restarted when they fail.

### Configure Probes

[Probes](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#probe-v1-core) have a number of fields that you can use to more precisely control the behavior of liveness and readiness checks:

- `initialDelaySeconds`: Number of seconds after the container has started before liveness or readiness probes are initiated.
- `periodSeconds`: How often (in seconds) to perform the probe. Default to 10 seconds. Minimum value is 1.
- `timeoutSeconds`: Number of seconds after which the probe times out. Defaults to 1 second. Minimum value is 1.
- `successThreshold`: Minimum consecutive successes for the probe to be considered successful after having failed. Defaults to 1. Must be 1 for liveness. Minimum value is 1.
- `failureThreshold`: When a Pod starts and the probe fails, Kubernetes will try `failureThreshold` times before giving up. Giving up in case of liveness probe means restarting the Pod. In case of readiness probe the Pod will be marked Unready. Defaults to 3. Minimum value is 1.

[HTTP probes](https://kubernetes.io/docs/reference/generated/kubernetes-api/v1.15/#httpgetaction-v1-core) have additional fields that can be set on `httpGet`:

- `host`: Host name to connect to, defaults to the pod IP. You probably want to set “Host” in httpHeaders instead.
- `scheme`: Scheme to use for connecting to the host (HTTP or HTTPS). Defaults to HTTP.
- `path`: Path to access on the HTTP server.
- `httpHeaders`: Custom headers to set in the request. HTTP allows repeated headers.
- `port`: Name or number of the port to access on the container. Number must be in the range 1 to 65535.

For an HTTP probe, the kubelet sends an HTTP request to the specified path and port to perform the check. The kubelet sends the probe to the pod’s IP address, unless the address is overridden by the optional `host` field in `httpGet`. If `scheme` field is set to `HTTPS`, the kubelet sends an HTTPS request skipping the certificate verification. In most scenarios, you do not want to set the `host` field. Here’s one scenario where you would set it. Suppose the Container listens on 127.0.0.1 and the Pod’s `hostNetwork` field is true. Then `host`, under `httpGet`, should be set to 127.0.0.1. If your pod relies on virtual hosts, which is probably the more common case, you should not use `host`, but rather set the `Host` header in `httpHeaders`.

For a probe, the kubelet makes the probe connection at the node, not in the pod, which means that you can not use a service name in the `host` parameter since the kubelet is unable to resolve it.



## Ingress

使用 ingress-nginx 作为 ingress-controller

```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/mandatory.yaml

# 裸机(非云及mac等),可以接收外部访问
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/static/provider/baremetal/service-nodeport.yaml
# 验证安装
kubectl get pods --all-namespaces -l app.kubernetes.io/name=ingress-nginx --watch
# 查看开放的端口
kubectl get svc -n ingress-nginx
```

编写 deplayment与service yaml

```yaml
vim deploy.yaml
apiVersion: v1
kind: Service
metadata:
   name: myapp-sv
   namespace: default
spec:
    selector:
        app: myapp
        release: canary
    ports:
        - name: http
          port: 80
          targetPort: 80
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: myapp-deploy
    namespace: default
spec:
    replicas: 3
    selector:
        matchLabels:
            app: myapp
            release: canary
    template:
        metadata:
            name: myapp-pod
            labels:
                app: myapp
                release: canary
        spec:
            containers:
                - name: myapp-container
                  image: ikubernetes/myapp:v1
                  ports:
                      - name: http
                        containerPort: 80
```

编写 ingress yaml

```yaml
vim ingress-myapp.yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
    name: ingress-myapp
    namespace: default
    annotations:
        kubernetes.io/ingress.class: "nginx"
spec:
    rules:
        - host: myapp.qlst.top
          http:
              paths:
                  - path:
                    backend:
                           serviceName: myapp-sv
                           servicePort: 80
```

启动yaml

```shell
kubectl apply -f deploy.yaml     
kubectl apply -f ingress-myapp.yaml
kubectl get ingress
```

编写 tomcat-deplayment与service yaml

```yaml
vim tomcat-deploy.yaml
apiVersion: v1
kind: Service
metadata:
   name: tomcat-sv
   namespace: default
spec:
    selector:
        app: tomcat
        release: canary
    ports:
        - name: http
          port: 8080
          targetPort: 8080
        - name: ajp
          port: 8009
          targetPort: 8009
---
apiVersion: apps/v1
kind: Deployment
metadata:
    name: tomcat-deploy
    namespace: default
spec:
    replicas: 3
    selector:
        matchLabels:
            app: tomcat
            release: canary
    template:
        metadata:
            name: tomcat-pod
            labels:
                app: tomcat
                release: canary
        spec:
            containers:
                - name: tomcat-container
                  image: tomcat:v1
                  ports:
                      - name: http
                        containerPort: 8080
                      - name: ajp
                        containerPort: 8009
```

编写 ingress-tomcat yaml

```yaml
vim ingress-tomcat.yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
    name: ingress-tomcat
    namespace: default
    annotations:
        kubernetes.io/ingress.class: "nginx"
spec:
    rules:
        - host: tomcat.qlst.top
          http:
              paths:
                  - path:
                    backend:
                           serviceName: tomcat-sv
                           servicePort: 8080
```

创建https

```yaml
kubectl create secret tls tomcat-ingress-secret --cert=tls.cry --key=tls.key

vim ingress-tomcat.yaml
apiVersion: networking.k8s.io/v1beta1
kind: Ingress
metadata:
    name: ingress-tomcat
    namespace: default
    annotations:
        kubernetes.io/ingress.class: "nginx"
spec:
    tls:
        - host: tomcat.qlst.top
          secretName: tomcat-ingress-secret
    rules:
        - host: tomcat.qlst.top
          http:
              paths:
                  - path:
                    backend:
                           serviceName: tomcat-sv
                           servicePort: 8080
```

## Volume

创建emptyDir

```yaml
vim demo.yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-demo
    namespace: default
    labels:
        app: myapp
        tier: frontend
    annotations:
        qlst.top/created-by: "Archangel"
spec:
    containers:
        - name: myapp
          image: ikubernetes/myapp:v1
          imagePullPolicy: IfNotPresent
          ports:
              - name: http
                containerPort: 80
          volumeMounts:
              - name: html
                mountPath: /usr/share/nginx/html
        - name: busybox
          image: busybox:latest
          imagePullPolicy: IfNotPresent
          volumeMounts:
              - name: html
                mountPath: /data/
          command: ["/bin/sh"]
          args: ["-c","while true; do echo $(date) >> /data/index.html; sleep 2; done"]
    volumes:
        - name: html
          emptyDir: {}
```

创建hostPath

```yaml
vim host-vol.yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-host-demo
    namespace: default
    annotations:
        qlst.top/created-by: "Archangel"
spec:
    containers:
        - name: myapp
          image: ikubernetes/myapp:v1
          imagePullPolicy: IfNotPresent
          ports:
              - name: http
                containerPort: 80
          volumeMounts:
              - name: html
                mountPath: /usr/share/nginx/html
    volumes:
        - name: html
          hostPath:
              path: /data/pod/volume1
              type: DirectoryOrCreate
```

创建 nfs

```yaml
vim nfs-vol.yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-nfs-demo
    namespace: default
    annotations:
        qlst.top/created-by: "Archangel"
spec:
    containers:
        - name: myapp
          image: ikubernetes/myapp:v1
          imagePullPolicy: IfNotPresent
          ports:
              - name: http
                containerPort: 80
          volumeMounts:
              - name: html
                mountPath: /usr/share/nginx/html
    volumes:
        - name: html
          nfs:
              path: /data/volumes
              server: nfsIP
```

## PV/PVC/StorageClass

创建 PV

```yaml
vim pv-demo.yaml
apiVersion: v1
Kind: PersistentVolume
    name: pv001
    labels:
        name: pv001
spec:
    nfs:
        path: /data/volumes/v1
        server: nfsIP
    accessModes: ["ReadWriteMany","ReadWriteOnce"]
    capacity:
        storage: 2Gi
---
vim pv-demo.yaml
apiVersion: v1
Kind: PersistentVolume
    name: pv002
    labels:
        name: pv002
spec:
    nfs:
        path: /data/volumes/v2
        server: nfsIP
    accessModes: ["ReadWriteOnce"]
    capacity:
        storage: 5Gi
---
vim pv-demo.yaml
apiVersion: v1
Kind: PersistentVolume
    name: pv003
    labels:
        name: pv003
spec:
    nfs:
        path: /data/volumes/v3
        server: nfsIP
    accessModes: ["ReadWriteMany","ReadWriteOnce"]
    capacity:
        storage: 10Gi
```

创建 PVC

```yaml
vim pvc-demo.yaml
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
    name: pod-pvc-demo
    namespace: default
    annotations:
        qlst.top/created-by: "Archangel"
spec:
     accessModes: ["ReadWriteMany","ReadWriteOnce"]
     resource:
         requests:
             storage: 6Gi
---
apiVersion: v1
kind: Pod
metadata:
    name: pod-pvc-vol-demo
    namespace: default
    annotations:
        qlst.top/created-by: "Archangel"
spec:
    containers:
        - name: myapp
          image: ikubernetes/myapp:v1
          imagePullPolicy: IfNotPresent
          ports:
              - name: http
                containerPort: 80
          volumeMounts:
              - name: html
                mountPath: /usr/share/nginx/html
    volumes:
        - name: html
          persistentVolumeClaim:
              claimName: pod-pvc-demo
```

## secret

```shell
kubectl create secret generic mysql-root-password --from-literal=passwd=Mysql@11..q
```

### 创建 yaml

```yaml
vim pod-secret.yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-secret-demo
    namespace: default
    labels:
        app: myapp
        tier: frontend
spec:
    containers:
        - name: myapp
          image: ikubernetes/myapp:v1
          ports:
              - name: http
                containerPort: 80
          env:   # 在启动时设置,不会自动更新,密码会明文显示在环境变量
              - name: MYSQL_ROOT_PASSWORD
                valueFrom:
                    secretKeyRef:
                        name: mysql-root-password
                        key: passwd
```



## configmap

键值数据,可以被pod使用.用于全局配置,可以通过volume实现实时更新配置

### create configmap from literal

```shell
$ kubectl create configmap nginx-config --from-literal=nginx_port=80 --from-literal=server_name=myapp.qlst.top
```

or use yaml file to do the same thing:

```shell
$ kubectl apply -f configmap_from-literal.yml
apiVersion: v1
kind: ConfigMap
metadata:
  name: nginx-config
  namespace: default
data:
  nginx_port: 80
  server_name: "myapp.qlst.top"
  
$ kubectl get configmap
NAME       DATA      AGE
nginx-config   2         5s
```

### create configmap from file

create configmap `config-2` from CLI

```shell
$ kubectl create configmap config-2 --from-file=./nginx.conf
configmap "config-2" created
$ kubectl get configmap
NAME            DATA      AGE
nginx-config    2         1m
config-2        1         4s
```

### using configmap by env

```
$ kubectl apply -f configmap_pod_env.yml
```

and go to pod container to check the env

```
$ kubectl exec busybox-1 -it sh
# echo $HOST
1.1.1.1
# echo $PORT
3000
```

### using configmap by volume

```
$ kubectl apply -f configmap_pod_volume.yml
```

and go to pod container to check the config file.

```
$ kubectl exec busybox-2 -it sh
# cd /etc/config
# ls
nginx.conf
```

### 创建 yaml

```yaml
vim pod-configmap.yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-cm-demo
    namespace: default
    labels:
        app: myapp
        tier: frontend
spec:
    containers:
        - name: myapp
          image: ikubernetes/myapp:v1
          ports:
              - name: http
                containerPort: 80
          env:   # 在启动时设置,不会自动更新
              - name: NGINX_SERVER_PORT
                valueFrom:
                    configMapKeyRef:
                        name: nginx-config
                        key: nginx_port
              - name: NGINX_SERVER_PORT
                valueFrom:
                    configMapKeyRef:
                        name: nginx-config
                        key: server_name
```

### 创建 yaml2

```yaml
vim pod-configmap2.yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-cm-demo
    namespace: default
    labels:
        app: myapp
        tier: frontend
spec:
    containers:
        - name: myapp
          image: ikubernetes/myapp:v1
          ports:
              - name: http
                containerPort: 80
          volumeMounts:
              - name: nginxconf
                mountPath: /etc/nginx/config.d/
                readOnly: true
    volumes:   # 可以自动更新
        - name: nginxconf
          configMap:
              name: nginx-config
```

## StatefulSets

```yaml
apiVersion: v1
kind: Service
metadata:
  name: nginx
  labels:
    app: nginx
spec:
  ports:
  - port: 80
    name: web
  clusterIP: None
  selector:
    app: nginx
---
apiVersion: apps/v1beta1
kind: StatefulSet
metadata:
  name: web
spec:
  serviceName: "nginx"
  replicas: 3
  template:
    metadata:
      labels:
        app: nginx
    spec:
      terminationGracePeriodSeconds: 10
      containers:
      - name: nginx
        image: gcr.io/google_containers/nginx-slim:0.8
        ports:
        - containerPort: 80
          name: web
        volumeMounts:
        - name: www
          mountPath: /usr/share/nginx/html
  volumeClaimTemplates:
  - metadata:
      name: www
    spec:
      accessModes: [ "ReadWriteOnce" ]
      storageClassName: my-storage-class
      resources:
        requests:
          storage: 1Gi
```

## 认证与授权

认证检测 -> 授权检测 -> 准入控制

### ServiceAccount

每个namespace下有一个名为default的默认的ServiceAccount对象，这个ServiceAccount里有一个名为Tokens的可以作为Volume一样被Mount到Pod里的Secret，当Pod启动时这个Secret会被自动Mount到Pod的指定目录下，用来协助完成Pod中的进程访问API Server时的身份鉴权过程。

```shell
# 创建 ServiceAccount 账户 admin
kubectl create serviceaccount admin
```

创建使用制定ServiceAccount的Pod

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-sa-demo
    namespace: default
    labels:
        app: myapp
        tier: frontend
spec:
    containers:
        - name: myapp
          image: ikubernetes/myapp:v1
          ports:
              - name: http
                containerPort: 80
    serviceAccountName: admin
```

### role,rolebinding,clusterrole

```shell
# 创建pods-reader角色 对pods 有get list 权限,--dry-run 不实际执行
kubectl create role pods-reader --verb=get,list --resource=pods --dry-run
# 显示配置
kubectl create role pods-reader --verb=get,list --resource=pods --dry-run -o yaml

# 显示已经创建的role
kubectl get role

# 创建rolebinding
kubectl create rolebinding binding-reader-pods --role=pods-reader --user=<用户名>

# 创建 clusterrole
kubectl create clusterrole cluster-reader --verb=get,list --resource=pods --dry-run
```

## Dashboard

```shell
# github:https://github.com/kubernetes/dashboard
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v1.10.1/src/deploy/recommended/kubernetes-dashboard.yaml

# 创建登录用户,根据https://github.com/kubernetes/dashboard/wiki/Creating-sample-user
```

## 网络插件 flannel

```shell
# 查看 flannel 名称
kubectl get configmap -n kube-system
# 查看 flannel 详细信息
kubectl get configmap kube-flanne-cfg -o yaml -n kube-system
```

## 网络插件 Calico

## 调度器,预选策略,优选函数

## 资源限制

```yaml
apiVersion: v1
kind: Pod
metadata:
    name: pod-limit-demo
    namespace: default
    labels:
        app: myapp
        tier: frontend
spec:
    containers:
        - name: myapp
          image: ikubernetes/myapp:v1
          ports:
              - name: http
                containerPort: 80
          resources:
              requests:
                  cpu: "200m"
                  memory: "128Mi"
              limits:
                  cpu: "500m"
                  memory: "256Mi"
```

## HeapSter InfluxDB Grafana 1.11+已经废弃

```shell
# 部署 InfluxDB 
# https://github.com/kubernetes-retired/heapster
wget https://raw.githubusercontent.com/kubernetes-retired/heapster/master/deploy/kube-config/influxdb/influxdb.yaml

vim influxdb.yaml
apiVersion: apps/v1 # 修改
kind: Deployment
metadata:
  name: monitoring-influxdb
  namespace: kube-system
spec:
  replicas: 1
  selector: # 添加
    matchLabels:
      task: monitoring
      k8s-app: influxdb
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: influxdb
    spec:
      containers:
      - name: influxdb
        image: k8s.gcr.io/heapster-influxdb-amd64:v1.5.2
        volumeMounts:
        - mountPath: /data
          name: influxdb-storage
      volumes:
      - name: influxdb-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: monitoring-influxdb
  name: monitoring-influxdb
  namespace: kube-system
spec:
  ports:
  - port: 8086
    targetPort: 8086
  selector:
    k8s-app: influxdb
    
    
wget https://raw.githubusercontent.com/kubernetes-retired/heapster/master/deploy/kube-config/influxdb/heapster.yaml

vim heapster.yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  name: heapster
  namespace: kube-system
---
apiVersion: apps/v1 # 修改
kind: Deployment
metadata:
  name: heapster
  namespace: kube-system
spec:
  replicas: 1
  selector: # 添加
    matchLabels:
      task: monitoring
      k8s-app: heapster
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: heapster
    spec:
      serviceAccountName: heapster
      containers:
      - name: heapster
        image: k8s.gcr.io/heapster-amd64:v1.5.4
        imagePullPolicy: IfNotPresent
        command:
        - /heapster
        - --source=kubernetes:https://kubernetes.default
        - --sink=influxdb:http://monitoring-influxdb.kube-system.svc:8086
---
apiVersion: v1
kind: Service
metadata:
  labels:
    task: monitoring
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: Heapster
  name: heapster
  namespace: kube-system
spec:
  ports:
  - port: 80
    targetPort: 8082
  type: NodePort # 添加
  selector:
    k8s-app: heapster
    
wget https://raw.githubusercontent.com/kubernetes-retired/heapster/master/deploy/kube-config/influxdb/grafana.yaml

vim grafana.yaml
apiVersion: apps/v1 # 修改
kind: Deployment
metadata:
  name: monitoring-grafana
  namespace: kube-system
spec:
  replicas: 1
  selector: # 添加
    matchLabels:
      task: monitoring
      k8s-app: grafana
  template:
    metadata:
      labels:
        task: monitoring
        k8s-app: grafana
    spec:
      containers:
      - name: grafana
        image: k8s.gcr.io/heapster-grafana-amd64:v5.0.4
        ports:
        - containerPort: 3000
          protocol: TCP
        volumeMounts:
        - mountPath: /etc/ssl/certs
          name: ca-certificates
          readOnly: true
        - mountPath: /var
          name: grafana-storage
        env:
        - name: INFLUXDB_HOST
          value: monitoring-influxdb
        - name: GF_SERVER_HTTP_PORT
          value: "3000"
          # The following env variables are required to make Grafana accessible via
          # the kubernetes api-server proxy. On production clusters, we recommend
          # removing these env variables, setup auth for grafana, and expose the grafana
          # service using a LoadBalancer or a public IP.
        - name: GF_AUTH_BASIC_ENABLED
          value: "false"
        - name: GF_AUTH_ANONYMOUS_ENABLED
          value: "true"
        - name: GF_AUTH_ANONYMOUS_ORG_ROLE
          value: Admin
        - name: GF_SERVER_ROOT_URL
          # If you're only using the API Server proxy, set this value instead:
          # value: /api/v1/namespaces/kube-system/services/monitoring-grafana/proxy
          value: /
      volumes:
      - name: ca-certificates
        hostPath:
          path: /etc/ssl/certs
      - name: grafana-storage
        emptyDir: {}
---
apiVersion: v1
kind: Service
metadata:
  labels:
    # For use as a Cluster add-on (https://github.com/kubernetes/kubernetes/tree/master/cluster/addons)
    # If you are NOT using this as an addon, you should comment out this line.
    kubernetes.io/cluster-service: 'true'
    kubernetes.io/name: monitoring-grafana
  name: monitoring-grafana
  namespace: kube-system
spec:
  # In a production setup, we recommend accessing Grafana through an external Loadbalancer
  # or through a public IP.
  # type: LoadBalancer
  # You could also use NodePort to expose the service at a randomly-generated port
  # type: NodePort
  ports:
  - port: 80
    targetPort: 3000
  selector:
    k8s-app: grafana
  type: NodePort
```

## metrics-server

```shell
# https://github.com/kubernetes/kubernetes/tree/master/cluster/addons/metrics-server
git clone https://github.com/kubernetes/kubernetes.git
# 或者
wget https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/metrics-server/auth-delegator.yaml
wget https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/metrics-server/auth-reader.yaml
wget https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/metrics-server/metrics-apiservice.yaml
wget https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/metrics-server/metrics-server-deployment.yaml
wget https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/metrics-server/metrics-server-service.yaml
wget https://raw.githubusercontent.com/kubernetes/kubernetes/master/cluster/addons/metrics-server/resource-reader.yaml


cd metrics-server/
kubectl create -f .

kubectl get svc -n kube-system

kubectl get pods -n kube-system

# 安装 prometheus(监控报警系统)
# 安装 prome
```

## Helm

[helm_github](https://github.com/helm/helm)  需要go语言环境 [helm_home](https://helm.sh)

```shell
# 安装helm
wget https://get.helm.sh/helm-v2.14.2-linux-amd64.tar.gz
tar -xvf helm-v2.14.2-linux-amd64.tar.gz
cd linux-amd64
mv helm /usr/bin

# 创建 chart 名称为myapp
helm create myapp

# 查看目录结构
tree myapp/

cd myapp
# 编辑发布信息
vim Chart.yaml
apiVersion: v1
appVersion: "1.0"
description: A Helm chart for kubernetes
name: myapp
version: 0.1.0
mantianer:
  - name: QLST
    email: qlst@qlst.top
    url: http://qlst.top/
    
# 编辑配置文件
vim values.yaml

#　分析文件是否有错
helm lint ../myapp

cd ..
# 打包
helm package myapp/

# 打开仓库服务
helm serve # 监听8879

helm search myapp

# 部署服务
helm install --name myapp3 local/myapp

helm status myapp3

# 清除服务,释放名字
helm delete --purge myapp3
```

### 部署EFK

```shell
# 根据教程 下载,安装 elasticsearch https://hub.kubeapps.com/charts/stable/elasticsearch

# 安装 fluentd
编辑域名

# 安装 kibana
```



## kubernetes 高可用集群

- 满足kubeadm对主人的最低要求的三台机器
- 满足kubeadm对工人最低要求的三台机器
- 群集中所有计算机之间的完全网络连接（公共或专用网络）
- 所有机器都有sudo权限
- 从一个设备到系统中所有节点的SSH访问
- kubeadm并kubelet安装在所有机器上。kubectl是可选的。

### 第一个master节点的步骤

```shell
cd /usr/local/docker/kubernetes/
vim kubeadm-config.yaml
apiVersion: kubeadm.k8s.io/v1beta2
kind: ClusterConfiguration
kubernetesVersion: stable
controlPlaneEndpoint: "LOAD_BALANCER_DNS:LOAD_BALANCER_PORT"
```

- `kubernetesVersion`应该设置为Kubernetes版本使用。这个例子使用`stable`。
- `controlPlaneEndpoint` 应匹配负载均衡器的地址或DNS和端口。
- 建议kubeadm，kubelet，kubectl和Kubernetes的版本匹配。