您好，我现在有一个三节点组成的集群，希望使用kubeadm安装Kubernetes，您可以给出大致的步骤吗，我会根据您提供的步骤来逐步执行，感谢您的帮助。



##### ChatGPT said:

ChatGPT

您好！使用 `kubeadm` 来安装 Kubernetes 集群是一个比较标准的方式。以下是三节点集群（一个 master 节点和两个 worker 节点）的安装步骤概要。请确保您的节点已经使用 Linux 系统（比如 Ubuntu 或 CentOS），并且已经能够通过网络互相访问。

### 1. 准备工作

#### 1.1 环境准备

- 三台主机

  ：

  - 1 个主节点（Control Plane）
  - 2 个工作节点（Worker Nodes）

- 硬件要求

  ：

  - CPU：每个节点至少 2 核
  - 内存：至少 2 GB
  - 磁盘空间：至少 20 GB
  - 每个节点都有唯一的主机名、MAC 地址和产品 UUID

- 软件要求

  ：

  - 64 位 Linux 系统（Ubuntu 20.04+ 或 CentOS 7+）
  - Docker 或其他 CRI（容器运行时）
  - kubeadm、kubelet、kubectl
  - 网络插件，如 Calico、Flannel

#### 1.2 基本设置

在所有节点上执行以下操作：

1. **关闭 swap 分区**：

   ```
   bash
   复制代码
   sudo swapoff -a
   sudo sed -i '/ swap / s/^\(.*\)$/#\1/g' /etc/fstab
   ```

2. **确保时间同步**： 安装 `ntp` 或 `chrony`，确保各个节点时间一致。

3. **配置防火墙（可选）**： 如果启用了防火墙，请确保以下端口打开：

   - **Master 节点**：6443, 2379-2380, 10250, 10251, 10252
   - **Worker 节点**：10250, 30000-32767

4. **禁用 SELinux**：

   ```
   bash
   复制代码
   sudo setenforce 0
   sudo sed -i 's/^SELINUX=enforcing$/SELINUX=permissive/' /etc/selinux/config
   ```

#### 1.3 安装 Docker

```
bash
复制代码
# 安装 Docker
sudo apt-get update
sudo apt-get install -y docker.io
sudo systemctl enable docker
sudo systemctl start docker
```

#### 1.4 安装 kubeadm、kubelet、kubectl

```
bash
复制代码
sudo apt-get update
sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### 2. 初始化 Kubernetes Master 节点

在 **master 节点** 上执行：

```
bash
复制代码
# 初始化 master 节点
sudo kubeadm init --pod-network-cidr=192.168.0.0/16
```

- `--pod-network-cidr=192.168.0.0/16` 是为 Calico 网络插件预留的。

初始化完成后，命令输出中会有一个 `kubeadm join` 命令，包含 token 和哈希值。请保存该命令，后续将用于在 worker 节点加入集群。

#### 2.1 配置 kubectl 命令行工具

在 **master 节点** 上执行：

```
bash
复制代码
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 2.2 安装网络插件（以 Calico 为例）

```
bash
复制代码
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### 3. 加入 Worker 节点

在 **每个 worker 节点** 上执行 `kubeadm join` 命令，该命令在主节点初始化时生成。例如：

```
bash
复制代码
sudo kubeadm join <master-ip>:6443 --token <your-token> --discovery-token-ca-cert-hash sha256:<your-hash>
```

替换 `<master-ip>`、`<your-token>` 和 `<your-hash>` 为在 master 节点初始化时输出的具体值。

### 4. 验证集群状态

在 master 节点上，可以通过以下命令验证所有节点是否已成功加入集群：

```
bash
复制代码
kubectl get nodes
```

### 5. 安装其他组件（可选）

- **Dashboard**：如果需要安装 Kubernetes Dashboard，可以参考官方文档来部署。
- **存储类插件**：如 `NFS`、`Ceph` 等，用于动态存储卷的创建。

------

### 小结

这是使用 `kubeadm` 部署 Kubernetes 的基本步骤。通过这些步骤，您可以搭建一个基础的 Kubernetes 集群，接下来可以根据需要安装其他的插件或服务。





##### You said:

谢谢您，请问：1、您提供的这些操作步骤在所有节点上都需要执行，还是一部分需要都执行，剩下的部分只在主节点上执行呢？2、关于网络插件，它有什么作用呢？据我所知有很多不同的网络插件，有没有最受欢迎/最好的选择呢？感谢您的帮助。



##### ChatGPT said:

ChatGPT

很高兴能帮助您！关于您提出的两个问题，我来详细解答：

### 1. 操作步骤在各个节点上的执行方式

- **所有节点都需要执行的步骤**： 以下操作需要在所有节点（包括主节点和工作节点）上都执行：
  - **1.2 基本设置**（关闭 `swap`，时间同步，防火墙配置，禁用 `SELinux`）
  - **1.3 安装 Docker**
  - **1.4 安装 kubeadm、kubelet、kubectl**
- **仅在主节点执行的步骤**：
  - **2. 初始化 Kubernetes Master 节点**：只在主节点上执行 `kubeadm init`，初始化控制平面。
  - **2.1 配置 kubectl**：配置 Kubernetes 控制台 `kubectl`，用于与集群通信。
  - **2.2 安装网络插件**：网络插件只需要在主节点上通过 `kubectl` 部署，工作节点无需手动安装。
- **仅在工作节点上执行的步骤**：
  - **3. 加入 Worker 节点**：通过 `kubeadm join` 命令让工作节点加入集群，这一步在工作节点上执行。

### 2. 关于网络插件

#### 2.1 网络插件的作用

在 Kubernetes 中，网络插件（CNI，Container Network Interface）负责管理 Pod 间的通信和网络路由。每个 Pod 需要一个独立的 IP 地址，且这些地址需要在整个集群中保持唯一。此外，Pod 和 Service 之间的通信需要通过网络插件来进行调度和管理。

网络插件的主要功能包括：

- **Pod 间通信**：确保集群中所有 Pod 能够相互通信，不论它们位于哪个节点上。
- **Service 通信**：管理 Pod 与 Kubernetes Service 之间的通信，提供负载均衡和 IP 地址分配。
- **网络策略**：一些网络插件支持高级的网络策略（Network Policies），允许您定义允许或拒绝哪些流量进出 Pod。

#### 2.2 常见的 Kubernetes 网络插件

有多种流行的网络插件，每个都有自己的特点和适用场景：

1. **Calico**：

   - **特点**：Calico 支持基于 IP 的路由，是目前最常见的网络插件之一。它不仅支持基本的 Pod 间通信，还支持网络策略（Network Policies）和 BGP 等高级功能，适合大规模生产环境。

   - **优点**：高性能、支持网络策略。

   - **适用场景**：大规模、高性能集群，安全性要求高的场景。

   - 安装命令

     ：

     ```
     bash
     复制代码
     kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
     ```

2. **Flannel**：

   - **特点**：Flannel 是一个简单易用的网络插件，提供最基本的 Pod 间通信功能，不支持网络策略。

   - **优点**：简单易用，适合小规模集群。

   - **缺点**：不支持网络策略，功能相对较少。

   - **适用场景**：简单的开发或测试环境。

   - 安装命令

     ：

     ```
     bash
     复制代码
     kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
     ```

3. **Weave**：

   - **特点**：Weave 提供了一种高效的覆盖网络，支持加密和网络策略。

   - **优点**：支持网络加密和策略，配置灵活。

   - **适用场景**：对安全性有需求的场景。

   - 安装命令

     ：

     ```
     bash
     复制代码
     kubectl apply -f https://cloud.weave.works/k8s/net?k8s-version=$(kubectl version | base64 | tr -d '\n')
     ```

4. **Cilium**：

   - **特点**：Cilium 是一个现代化的网络插件，基于 eBPF 技术，支持更复杂的网络策略、负载均衡和安全控制。适用于云原生和微服务架构。

   - **优点**：支持高级的网络策略和安全功能，适合高安全性和高性能要求的场景。

   - **适用场景**：需要微服务通信监控和高级策略的企业级应用。

   - 安装命令

     ：

     ```
     bash
     复制代码
     kubectl apply -f https://raw.githubusercontent.com/cilium/cilium/v1.9/install/kubernetes/quick-install.yaml
     ```

5. **Kube-Router**：

   - **特点**：Kube-Router 将 Kubernetes 网络、网络策略和负载均衡整合在一起，简化了网络管理。
   - **优点**：轻量级，集成多个功能。
   - **适用场景**：需要整合多种功能的小型集群。

#### 2.3 最受欢迎/最好的选择

- **Calico**：Calico 是生产环境中最常用的网络插件之一，提供了强大的网络策略功能，高度可扩展，且性能较好。Calico 是大多数企业生产环境的默认选择。
- **Flannel**：Flannel 简单易用，适合初学者或小规模集群部署，特别是在测试或开发环境中。
- **Cilium**：如果您需要使用高级的网络策略、安全功能，或者对性能要求特别高（如 eBPF 技术），可以考虑使用 Cilium。

总结：

- **简单场景**：选择 **Flannel**，易于配置，适合测试环境。
- **生产环境**：选择 **Calico**，功能强大且性能较好，支持网络策略。
- **高安全、高性能场景**：选择 **Cilium**，支持 eBPF，提供现代化的网络管理功能。

希望这些信息对您有所帮助！如果有更多问题，随时问我。





##### You said:

谢谢您，我明白了，现在，如果我准备使用containerd作为容器运行时（不安装docker）请问安装过程是怎样的呢？感谢您的帮助。



##### ChatGPT said:

ChatGPT

很高兴您问到这个问题！`containerd` 是一个流行的容器运行时，它作为 Kubernetes 的默认容器运行时是非常合适的选择。下面是关于如何使用 `containerd` 而不是 Docker 作为容器运行时来部署 Kubernetes 集群的详细步骤。

### 安装 `containerd` 的步骤

以下步骤适用于 Ubuntu 20.04 及更新版本的系统（对于 CentOS 系统，安装方式会略有不同，但大体思路相同）。请在每个节点（包括主节点和工作节点）上执行以下操作。

### 1. 安装依赖

首先，更新系统并安装必要的依赖项。

```
bash
复制代码
sudo apt-get update
sudo apt-get install -y apt-transport-https curl
```

### 2. 安装 `containerd`

接下来，安装 `containerd` 并进行配置。

#### 2.1 安装 containerd

```
bash
复制代码
sudo apt-get install -y containerd
```

#### 2.2 配置 `containerd`

创建 `containerd` 的默认配置文件：

```
bash
复制代码
sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
```

在生成的 `config.toml` 文件中，确保 `SystemdCgroup` 被设置为 `true`，这对 Kubernetes 的稳定运行至关重要。可以使用编辑器来编辑此文件：

```
bash
复制代码
sudo vim /etc/containerd/config.toml
```

找到以下部分并将 `SystemdCgroup` 设置为 `true`：

```
toml
复制代码
[plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
  SystemdCgroup = true
```

保存并退出编辑器。

#### 2.3 重启并启用 `containerd`

配置完成后，重启并启用 `containerd`：

```
bash
复制代码
sudo systemctl restart containerd
sudo systemctl enable containerd
```

### 3. 安装 Kubernetes 组件

接下来，安装 Kubernetes 组件（`kubeadm`、`kubelet` 和 `kubectl`）。这些步骤在每个节点上都需要执行。

```
bash
复制代码
sudo apt-get update
sudo apt-get install -y apt-transport-https curl
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

### 4. 使用 kubeadm 初始化集群

#### 4.1 在 Master 节点上初始化 Kubernetes

在 **主节点** 上，使用 `kubeadm` 初始化 Kubernetes 集群，同时指定 `containerd` 作为容器运行时。初始化命令如下：

```
bash
复制代码
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 --cri-socket /run/containerd/containerd.sock
```

- `--pod-network-cidr=192.168.0.0/16` 是为网络插件（如 Calico）预留的 Pod CIDR 范围。
- `--cri-socket /run/containerd/containerd.sock` 是指定 `containerd` 的 socket，表示使用 `containerd` 作为容器运行时。

如果集群初始化成功，您会看到 `kubeadm join` 命令输出，稍后您将在工作节点上使用该命令加入集群。

#### 4.2 配置 `kubectl`

在主节点上，您需要为非 root 用户配置 `kubectl` 访问：

```
bash
复制代码
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

#### 4.3 安装网络插件

安装网络插件（例如 Calico）。在主节点上执行以下命令：

```
bash
复制代码
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### 5. 将工作节点加入集群

在 **工作节点** 上，通过执行 `kubeadm join` 命令将它们加入集群。

首先，使用从主节点初始化时得到的 `kubeadm join` 命令在每个工作节点上执行。例如：

```
bash
复制代码
sudo kubeadm join <master-ip>:6443
```