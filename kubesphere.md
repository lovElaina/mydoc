能使用 KubeKey 安装的 Kubernetes 版本与 KubeSphere 3.4 支持的 Kubernetes 版本不同。如需[在现有 Kubernetes 集群上安装 KubeSphere 3.4](https://kubesphere.io/zh/docs/v3.4/installing-on-kubernetes/introduction/overview/)， Kubernetes 版本必须为 v1.21.x、v1.22.x、v1.23.x、* v1.24.x、* v1.25.x 和 * v1.26.x。

带星号的版本可能出现边缘节点部分功能不可用的情况。为了避免兼容性问题，建议安装 v1.23.x 版本的 Kubernetes。

- 购买三个云服务器，在同一区域，可以实现内网互通，分别为master、node1、node2。

配置信息如下：

![202406240855946](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/202406240855946.png)

实例列表如下：

![image-20240624141543244](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240624141543244.png)

- 在https://github.com/kubesphere/kubekey/releases页面下载kubekey-v3.1.1-linux-amd64.tar.gz，上传至master服务器。

- 解压文件**tar -xzvf kubekey-v3.1.1-linux-amd64.tar.gz**，得到二进制可执行文件kk。

- 执行./kk create config [--with-kubernetes version] [--with-kubesphere version] [(-f | --file) path]，如果不指定 Kubernetes 版本，KubeKey 将默认安装 Kubernetes v1.23.15。例如：**./kk create config --with-kubesphere 3.4.1**

- 执行上述命令后，将会在当前目录下，创建默认文件 config-sample.yaml。

- 调整host文件为云服务器配置，注意hosts下的内网ip地址、用户名、密码，并且etcd和control-plane均需要使用master节点。

  ![image-20240624141827829](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240624141827829.png)

另外由于需要启用devops，最好在安装之前就修改配置文件，如果等kubesphere安装完成后再修改配置文件，可能出现错误。

![image-20240624142021635](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240624142021635.png)

- 启用应用商店

![image-20240625091152744](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240625091152744.png)

- 编辑完成配置文件后，执行 **./kk create cluster -f config-sample.yaml**
- 执行上述命令后，首先会检测缺少的依赖项。一般来说会缺少conntrack（连接管理工具）和socat（双向数据传输工具），需要在三个节点上均安装他们。使用命令 **yum install -y conntrack  socat**

- 通过下面命令来查看哪些pod启动了：**kubectl get pod -A**

![image-20240624143921715](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240624143921715.png)

不应出现Running和Completed之外的其他状态。如图，现在安装顺利完成，可以可以正常启动kubesphere。

使用主节点的地址，加上默认端口30880，例如http://43.135.88.244:30880/ 访问kubesphere控制台。 注意，**初始用户名为admin，初始密码为P@88w0rd。**

出现以下页面，并可顺利登录进入控制台，说明kubesphere和k8s安装完成。

![image-20240624144334388](https://aemon-1309182592.cos.ap-hongkong.myqcloud.com/image-20240624144334388.png)