#### 基本设置1234

在所有节点上执行以下操作：

1. **关闭 swap 分区**：

   ```
   sudo swapoff -a
   sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
   ```

2. **确保时间同步**： 安装 `ntp` 或 `chrony`，确保各个节点时间一致。

3. **配置防火墙（可选）**： 如果启用了防火墙，请确保以下端口打开：

   **TCP 9500-9504：Longhorn 数据复制**

   **TCP 9808：Longhorn 控制平面**

   - **Master 节点**：6443, 2379-2380, 10250, 10251, 10252
   - **Worker 节点**：10250, 30000-32767


![image-20240911113230402](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240911113230402.png)

```bash
$ tar Cxzvf /usr/local containerd-1.7.22-linux-amd64.tar.gz
bin/
bin/containerd-shim-runc-v2
bin/containerd-shim
bin/ctr
bin/containerd-shim-runc-v1
bin/containerd
bin/containerd-stress
```



into `/usr/local/lib/systemd/system/containerd.service`

touch containerd.service

```
# Copyright The containerd Authors.
#
# Licensed under the Apache License, Version 2.0 (the "License");
# you may not use this file except in compliance with the License.
# You may obtain a copy of the License at
#
#     http://www.apache.org/licenses/LICENSE-2.0
#
# Unless required by applicable law or agreed to in writing, software
# distributed under the License is distributed on an "AS IS" BASIS,
# WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
# See the License for the specific language governing permissions and
# limitations under the License.

[Unit]
Description=containerd container runtime
Documentation=https://containerd.io
After=network.target local-fs.target

[Service]
ExecStartPre=-/sbin/modprobe overlay
ExecStart=/usr/local/bin/containerd

Type=notify
Delegate=yes
KillMode=process
Restart=always
RestartSec=5

# Having non-zero Limit*s causes performance problems due to accounting overhead
# in the kernel. We recommend using cgroups to do container-local accounting.
LimitNPROC=infinity
LimitCORE=infinity

# Comment TasksMax if your systemd version does not supports it.
# Only systemd 226 and above support this version.
TasksMax=infinity
OOMScoreAdjust=-999

[Install]
WantedBy=multi-user.target
```

and run the following commands:

```bash
systemctl daemon-reload
systemctl enable --now containerd
```

![image-20240911113257249](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240911113257249.png)

```bash
install -m 755 runc.amd64 /usr/local/sbin/runc
```

![image-20240911141406403](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240911141406403.png)

```
$ mkdir -p /opt/cni/bin
$ tar Cxzvf /opt/cni/bin cni-plugins-linux-amd64-v1.5.1.tgz
```



[root@master ~]# cd /etc
[root@master etc]# mkdir containerd
[root@master etc]# cd containerd/
[root@master containerd]# ls
[root@master containerd]# touch config.toml
[root@master containerd]# containerd config default > /etc/containerd/config.toml

**vi config.toml**

**SystemdCgroup = true**

[root@master containerd]# **sudo systemctl restart containerd**



https://github.com/kubernetes-sigs/cri-tools/releases/download/v1.30.0/crictl-v1.30.0-linux-amd64.tar.gz

![image-20240911145143247](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240911145143247.png)

```
tar Cxzvf /usr/local/bin crictl-v1.30.0-linux-amd64.tar.gz
```

安装kubeadm & kubelet

**install kubeadm -> kubectl -> kubeadm init -> network addon....**

![image-20240911161940493](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240911161940493.png)

### 在服务器上处理文件

#### 1 复制二进制文件到系统路径

在服务器上，将 `kubeadm` 和 `kubelet` 二进制文件复制到系统路径，例如 `/usr/local/bin`：

```
sudo cp /path/to/server/destination/kubeadm /usr/local/bin/
sudo cp /path/to/server/destination/kubelet /usr/local/bin/
sudo chmod +x /usr/local/bin/kubeadm /usr/local/bin/kubelet
```

#### 2 配置 `kubelet` 服务

将 `kubelet.service` 复制到 `/usr/lib/systemd/system/` 目录：

```
sudo cp /path/to/server/destination/kubelet.service /usr/lib/systemd/system/
```

修改 `kubelet.service` 文件中的路径，将 `/usr/bin` 替换为您实际的二进制文件路径（通常是 `/usr/local/bin`）：

```
sudo sed -i 's:/usr/bin:/usr/local/bin:g' /usr/lib/systemd/system/kubelet.service
```

#### 3 创建 `kubelet.service.d` 目录并复制配置文件

创建 `kubelet.service.d` 目录并将 `10-kubeadm.conf` 复制进去：

```
sudo mkdir -p /usr/lib/systemd/system/kubelet.service.d
sudo cp /path/to/server/destination/10-kubeadm.conf /usr/lib/systemd/system/kubelet.service.d/
```

同样地，修改 `10-kubeadm.conf` 文件中的路径：

```
sudo sed -i 's:/usr/bin:/usr/local/bin:g' /usr/lib/systemd/system/kubelet.service.d/10-kubeadm.conf
```

#### 4 启动并启用 `kubelet` 服务

执行以下命令以启动 `kubelet` 并设置其开机自启：

```
sudo systemctl daemon-reload
sudo systemctl enable kubelet
sudo systemctl start kubelet
```



安装kubectl

![image-20240911172202910](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240911172202910.png)

```bash
sudo install -o root -g root -m 0755 kubectl /usr/local/bin/kubectl
```

```bash
export KUBECONFIG=/etc/kubernetes/admin.conf
```



```bash
echo 'export KUBECONFIG=/etc/kubernetes/admin.conf' >> ~/.bashrc
source ~/.bashrc
```

初始化

```bash
kubeadm init --pod-network-cidr=192.168.0.0/16
```



Once a Pod network has been installed, you can confirm that it is working by checking that the CoreDNS Pod is `Running` in the output of `kubectl get pods --all-namespaces`. And once the CoreDNS Pod is up and running, you can continue by joining your nodes



### 安装longhorn

检查安装依赖

```bash
./longhornctl-linux-amd64 check preflight
```

三节点安装

```bash
  yum --setopt=tsflags=noscripts install iscsi-initiator-utils
  echo "InitiatorName=$(/sbin/iscsi-iname)" > /etc/iscsi/initiatorname.iscsi
  systemctl enable iscsid
  systemctl start iscsid
  sudo yum install -y nfs-utils
```

```bash
kubectl apply -f longhorn.yaml
```

查看安装进展

```shell
kubectl get pods \
--namespace longhorn-system \
--watch
```

查看服务
```
kubectl get svc -n longhorn-system
```

编辑文件

```
kubectl edit svc longhorn-frontend -n longhorn-system
```

这将打开一个编辑器（默认是 vi），找到以下内容：

```
spec:
  type: ClusterIP
```

将其修改为：

```
spec:
  type: NodePort

```
再次查看服务，找到nodePort端口号

```bash
[root@master ~]# kubectl get svc -n longhorn-system
NAME                          TYPE        CLUSTER-IP       EXTERNAL-IP   PORT(S)        AGE
longhorn-admission-webhook    ClusterIP   10.96.148.119    <none>        9502/TCP       23m
longhorn-backend              ClusterIP   10.96.32.112     <none>        9500/TCP       23m
longhorn-conversion-webhook   ClusterIP   10.97.78.169     <none>        9501/TCP       23m
longhorn-frontend             NodePort    10.100.149.234   <none>        80:32261/TCP   23m
longhorn-recovery-backend     ClusterIP   10.109.207.110   <none>        9503/TCP       23m
```

访问 http://ip:32261

### 安装portainer

```bash
kubectl apply -n portainer -f portainer.yaml
```

### 查看portainer - svc
```bash
[root@master ~]# kubectl get svc -n portainer
NAME        TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                                         AGE
portainer   NodePort   10.109.142.35   <none>        9000:30777/TCP,9443:30779/TCP,30776:30776/TCP   4m26s
```

访问 http://ip:30777

Kubernetes 在进行初始化时发现了系统环境中的一些问题，导致初始化无法继续。这些问题需要解决才能成功初始化集群。以下是每个问题的详细说明及解决方法：

### 1. **socat not found in system path** （警告）：

- **问题**：您的系统没有安装 `socat`。尽管这是一个警告，不是致命错误，但 `socat` 是一个依赖工具，它用于转发连接，因此最好安装它。

- 解决方案

  ： 使用以下命令安装 socat

  ```
  yum install -y socat  # CentOS 或 RHEL 系统
  ```

### 2. **[ERROR FileExisting-conntrack]: conntrack not found in system path**（致命错误）：

- **问题**：系统中没有安装 `conntrack`，这是 Kubernetes 用于跟踪网络连接状态的工具，必须安装它才能继续。

- 解决方案

  ： 使用以下命令安装 conntrack

  ```
  yum install -y conntrack  # CentOS 或 RHEL 系统
  ```

### 3. **[ERROR FileContent--proc-sys-net-ipv4-ip_forward]: /proc/sys/net/ipv4/ip_forward contents are not set to 1**（致命错误）：

- **问题**：Kubernetes 需要开启 `ip_forward` 功能，即允许内核转发 IP 包，但系统当前的设置没有启用这一功能。

- 解决方案

  ： 执行以下命令启用 ip_forward

  ```
  echo "1" > /proc/sys/net/ipv4/ip_forward
  ```

  或者修改系统配置文件 

  ```
  /etc/sysctl.conf
  ```

  确保以下行存在：

  ```
  net.ipv4.ip_forward = 1
  ```

  然后运行以下命令使更改生效：

  ```
  sysctl -p
  ```

在初始化之前，请确保系统中安装了 Kubernetes 所需的基本工具和依赖库，并根据错误信息逐步修复系统环境的问题。通过安装缺失的工具和配置系统参数，您的 Kubernetes 集群应该能够顺利初始化。

///////////////////////////////可能会出现的网络环境导致失败的问题////////////////////////////////

### `pause` 容器（sandbox 容器）是什么？

在 Kubernetes 中，**每个 Pod** 都是由一个或多个容器组成的，Pod 中的所有容器共享同一个网络命名空间、存储卷、IP 地址等资源。为了实现这一点，Kubernetes 使用了一个特殊的容器，称为 **`pause` 容器** 或 **sandbox 容器**。

具体作用如下：

- **网络命名空间的持有者**：`pause` 容器的主要职责是为每个 Pod 创建并持有网络命名空间和其他共享资源。这样 Pod 内的其他容器可以共享这些资源，而不需要自己创建。
- **保持 Pod 生命周期**：即使 Pod 中的应用容器因为故障重启，`pause` 容器会继续运行，从而保持 Pod 的网络环境不变。

所以，**`pause` 容器是 Kubernetes 内部用来管理 Pod 网络的一种隐藏容器**。

### `sandbox_image` 是什么？

`sandbox_image` 是 `pause` 容器的镜像，这个镜像用于启动 `pause` 容器。默认情况下，Kubernetes 使用 `registry.k8s.io/pause:3.2` 作为 `pause` 镜像。

### `containerd` 配置中的 `sandbox_image` 是什么含义？

您提到的这段配置：

```
[plugins."io.containerd.grpc.v1.cri"]
  sandbox_image = "registry.k8s.io/pause:3.2"
```

这段配置允许您在 `containerd` 的配置文件中 **自定义** 或 **覆盖** Kubernetes 使用的 `pause` 容器镜像。如果您不做任何更改，`containerd` 会默认使用 `registry.k8s.io/pause:3.2` 这个镜像。

### 为什么需要自定义 `sandbox_image`？

自定义 `sandbox_image` 的原因可能包括：

1. **镜像源的修改**：在某些环境中，您可能无法直接从默认的镜像仓库（如 `registry.k8s.io`）下载镜像。这时，您可以指定一个自定义的镜像源，比如使用企业的私有镜像仓库。

   例如，您可以将 `sandbox_image` 配置为您的私有镜像仓库的地址：

   ```
   toml
   复制代码
   [plugins."io.containerd.grpc.v1.cri"]
     sandbox_image = "my.private.registry/pause:3.2"
   ```

2. **镜像版本的调整**：如果您需要使用特定版本的 `pause` 镜像（如旧版本的 Kubernetes 兼容性问题），可以在此处指定一个不同的版本。

3. **网络环境的限制**：在某些受限的网络环境中（例如无外网环境），直接从 Kubernetes 官方仓库下载镜像可能不可行。因此，您可以预先将 `pause` 镜像放入本地镜像仓库，修改 `containerd` 配置文件指向该仓库。

### 如何修改 `containerd` 的 `sandbox_image`？

1. 编辑 `containerd` 的配置文件，通常位于 `/etc/containerd/config.toml`：

   ```
   bash
   复制代码
   sudo vim /etc/containerd/config.toml
   ```

2. 在文件中找到或添加以下部分，确保 `sandbox_image` 设置为您需要的镜像：

   ```
   toml
   复制代码
   [plugins."io.containerd.grpc.v1.cri"]
     sandbox_image = "registry.k8s.io/pause:3.2"
   ```

3. 保存文件并重启 `containerd` 使配置生效：

   ```
   bash
   复制代码
   sudo systemctl restart containerd
   ```

### 总结

- **`pause` 容器（sandbox 容器）** 是 Kubernetes 用于管理 Pod 网络命名空间的重要组件。
- **`sandbox_image`** 是指用于启动 `pause` 容器的镜像，默认为 `registry.k8s.io/pause:3.2`。
- 您可以通过修改 `containerd` 的配置文件来自定义或覆盖 `pause` 镜像，例如指定不同版本或使用私有镜像仓库。
- 修改后的 `sandbox_image` 将影响所有新创建的 Pod 使用的 `pause` 容器镜像。

////////////////////////////////////////////////

### **在可访问互联网的本地机器上操作：**

#### **1. 安装 Docker 或 containerd（如果尚未安装）**

为了拉取所需的镜像，您可以在外网机器上安装 **Docker** 或 **containerd**。这里为了方便，我们使用 Docker。

**安装 Docker：**

- **CentOS/RHEL：**

  ```
  bash
  复制代码
  sudo yum install -y docker
  ```

- **Ubuntu：**

  ```
  bash
  复制代码
  sudo apt-get update
  sudo apt-get install -y docker.io
  ```

#### **2. 启动 Docker 服务**

```
bash
复制代码
sudo systemctl start docker
sudo systemctl enable docker
```

#### **3. 拉取所需的镜像**

```
bash
复制代码
docker pull registry.k8s.io/kube-apiserver:v1.30.4
docker pull registry.k8s.io/kube-controller-manager:v1.30.4
docker pull registry.k8s.io/kube-scheduler:v1.30.4
docker pull registry.k8s.io/kube-proxy:v1.30.4
docker pull registry.k8s.io/coredns/coredns:v1.11.1
docker pull registry.k8s.io/pause:3.9
docker pull registry.k8s.io/etcd:3.5.12-0
```

#### **4. 将镜像保存为 tar 文件**

```
bash
复制代码
docker save \
  -o k8s-images.tar \
  registry.k8s.io/kube-apiserver:v1.30.4 \
  registry.k8s.io/kube-controller-manager:v1.30.4 \
  registry.k8s.io/kube-scheduler:v1.30.4 \
  registry.k8s.io/kube-proxy:v1.30.4 \
  registry.k8s.io/coredns/coredns:v1.11.1 \
  registry.k8s.io/pause:3.9 \
  registry.k8s.io/etcd:3.5.12-0
```

#### **5. 将镜像文件传输到内网服务器**

使用 U 盘、SCP、FTP 或其他方式，将 `k8s-images.tar` 文件传输到内网服务器上的指定目录。

------

### **在内网服务器上操作：**

#### **1. 确保 containerd 已安装并运行**

既然您已经安装了 containerd，请确认它正在运行：

```
bash
复制代码
sudo systemctl status containerd
```

如果未运行，请启动：

```
bash
复制代码
sudo systemctl start containerd
sudo systemctl enable containerd
```

#### **2. 安装 `ctr` 或 `nerdctl` 工具**

**选项 A：使用 `ctr`（containerd 自带的 CLI 工具）**

`ctr` 是 containerd 的命令行工具，可以用于加载和管理镜像。

**选项 B：安装 `nerdctl`（推荐）**

`nerdctl` 是一个类 Docker 的 CLI 工具，与 containerd 一起工作，更加易于使用。

**安装 `nerdctl`：**

```
bash
复制代码
# 下载最新版本的 nerdctl
wget https://github.com/containerd/nerdctl/releases/download/v1.5.0/nerdctl-1.5.0-linux-amd64.tar.gz

# 解压并移动到 /usr/local/bin
sudo tar Cxzvf /usr/local/bin nerdctl-1.5.0-linux-amd64.tar.gz
```

#### **3. 使用 `nerdctl` 加载镜像**

```
bash
复制代码
sudo nerdctl load -i k8s-images.tar
```

#### **4. 验证镜像已成功加载**

```
bash
复制代码
sudo nerdctl images
```

您应该能看到刚刚加载的镜像列表。

#### **5. 运行 `kubeadm init`**

现在，您可以运行以下命令来初始化 Kubernetes 集群：

```
bash
复制代码
sudo kubeadm init --config kubeadm-config.yaml
```

如果没有配置文件，可以直接运行：

```
bash
复制代码
sudo kubeadm init
```

------

### **注意事项：**

#### **镜像命名空间可能导致的问题**

有时，镜像的命名空间可能会导致 `kubeadm` 无法找到所需的镜像。如果在运行 `kubeadm init` 时遇到类似以下错误：

```
vbnet
复制代码
Failed to pull image "k8s.gcr.io/kube-apiserver:v1.30.4": rpc error: code = NotFound desc = failed to pull and unpack image "k8s.gcr.io/kube-apiserver:v1.30.4": not found
```

这是因为镜像在加载后，命名空间可能与 `kubeadm` 期望的不同。

#### **解决方法：重新标记镜像**

您需要将镜像重新标记为 kubeadm 期望的命名空间和标签。具体步骤如下：

1. **查看已有镜像列表**：

   ```
   bash
   复制代码
   docker images
   ```

   找到已经下载的镜像，例如：

   ```
   bash
   复制代码
   myrepo/kube-apiserver   v1.30.4   <IMAGE_ID>   ...
   ```

2. **重新标记镜像**：

   使用 `docker tag` 命令将已有的镜像重新标记为 kubeadm 需要的名称。例如：

   ```
   bash
   复制代码
   docker tag myrepo/kube-apiserver:v1.30.4 k8s.gcr.io/kube-apiserver:v1.30.4
   ```

   **注意**：请将 `myrepo/kube-apiserver:v1.30.4` 替换为您实际的镜像名称和标签。

3. **验证重新标记是否成功**：

   再次查看镜像列表，确认新标签已添加：

   ```
   bash
   复制代码
   docker images
   ```

   您应该能看到：

   ```
   bash
   复制代码
   k8s.gcr.io/kube-apiserver   v1.30.4   <SAME_IMAGE_ID>   ...
   ```

4. **删除原有的镜像标签（可选）**：

   如果不再需要原有的镜像标签，可以删除：

   ```
   bash
   复制代码
   docker rmi myrepo/kube-apiserver:v1.30.4
   ```

5. **重新运行 kubeadm init**：

   现在，kubeadm 应该能够找到并使用正确的镜像。