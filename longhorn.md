关于 Kubernetes 集群中的存储解决方案选择，Longhorn 和 Ceph RBD 是两个广泛使用的存储选项。为了帮助您做出决策，下面将从多个方面对它们进行对比分析，包括部署难易度、后期维护难度、性能表现、扩展性、数据一致性与高可用性、社区支持以及场景适配等因素。

### 1. **部署难易度**

**Longhorn**:

- **部署简便性**：Longhorn 是一个 Kubernetes 原生的存储解决方案，部署相对简单。它提供了非常容易使用的 Helm Chart 或者是官方的 Kubernetes Operator 来简化安装流程。只需要在集群中执行几个简单的命令，便可以启动 Longhorn 并开始使用。它的原生集成特性使得与 Kubernetes 环境的融合非常顺畅。
- **依赖项**：Longhorn 不需要复杂的外部依赖，只需要 Kubernetes 集群本身，因此非常轻量级。

**Ceph RBD**:

- **部署复杂性**：Ceph 本质上是一个更复杂的分布式存储系统，它需要多个组件（如 Ceph Monitor、OSD、MDS 等）来进行协调。部署 Ceph RBD 相对更复杂，尤其是在 Ceph 与 Kubernetes 的集成方面。虽然可以通过工具（例如 Rook Operator）来简化部署，但配置和管理上依然存在一定的复杂性。
- **依赖项**：Ceph 系统有一定的硬件需求（例如 OSD 需要专用的硬盘），而且 Ceph RBD 的配置需要更多的时间和资源进行规划。

**结论**：在部署难度方面，Longhorn 更加简单、轻量，适合中小型企业或初次实施 Kubernetes 存储解决方案的团队。Ceph RBD 适合已经具备一定运维经验、需要更复杂配置的大型企业。

### 2. **后期维护难易度**

**Longhorn**:

- **维护简便性**：Longhorn 的设计目标是尽量降低运维复杂度，它有着友好的 Web 界面，可以直接在界面上完成快照、卷备份、还原等操作。它还集成了健康检查和自动修复功能，简化了存储卷的管理和维护工作。
- **弹性扩展**：Longhorn 采用了存储卷复制机制，通过 Kubernetes 内置的机制可以实现卷的动态调整和扩展，基本不需要太多的手工干预。

**Ceph RBD**:

- **维护复杂性**：Ceph 是一个强大但复杂的存储系统，后期维护的工作量较大，涉及到监控、健康状态管理、OSD 故障恢复等运维任务。运维人员需要对 Ceph 的架构、存储池、对象存储和文件系统有较深入的理解。此外，Ceph 的性能调优相对复杂，尤其是在高并发场景下。
- **维护工具**：Ceph 提供了非常多的运维工具来帮助管理员进行系统管理和故障排查，Rook Operator 也简化了很多 Kubernetes 上的管理工作，但相比 Longhorn 依然需要更深入的运维知识。

**结论**：Longhorn 的维护难度较低，适合运维资源有限的团队，而 Ceph 需要专业的运维人员，维护成本相对较高。

### 3. **性能表现**

**Longhorn**:

- **性能表现**：Longhorn 的性能较为适中，它更适合中小规模的存储需求。由于 Longhorn 是基于文件存储系统和网络复制的设计，它的 I/O 性能会受到网络延迟和带宽的影响。因此，在高性能和低延迟需求的场景中，Longhorn 可能会显得有些力不从心。
- **场景适配**：对于一般的业务系统，Longhorn 可以提供足够的性能支持。但对于高并发、数据库型负载，性能表现可能不足。

**Ceph RBD**:

- **性能表现**：Ceph RBD 是高性能存储解决方案，特别是在大规模集群中表现出色。由于其基于块存储，Ceph RBD 的性能相较于文件系统类存储（如 Longhorn）要更优，尤其在数据库、虚拟化和高并发访问场景中，Ceph 的块设备读写性能更具优势。此外，Ceph 支持 SSD 等硬件加速优化，性能表现也更加灵活。
- **场景适配**：对于高 I/O、低延迟需求的应用，Ceph RBD 具有非常明显的性能优势。

**结论**：Ceph RBD 在性能表现上优于 Longhorn，尤其在高并发、高性能场景中。如果性能是您的主要考虑因素，Ceph RBD 是一个更好的选择。

### 4. **扩展性**

**Longhorn**:

- **扩展性**：Longhorn 可以非常轻松地进行水平扩展，通过 Kubernetes 资源的自动扩展来动态增加存储卷。Longhorn 的存储池扩展也相对简单，可以通过增加节点和磁盘的方式来提高存储能力。不过由于其网络复制机制，在规模特别大的集群中可能会影响性能和网络带宽。

**Ceph RBD**:

- **扩展性**：Ceph 是一个设计用于大规模集群的存储系统，具备极强的扩展能力。它可以支持数百甚至上千个存储节点。Ceph 支持存储池的动态扩展，可以通过简单地增加硬盘或节点来扩展集群的存储能力，同时能够高效地处理大规模数据并发访问。

**结论**：在扩展性方面，Ceph RBD 更为强大，适合那些未来需要不断扩展存储能力的大规模企业。

### 5. **数据一致性与高可用性**

**Longhorn**:

- **数据一致性**：Longhorn 支持存储卷的自动复制，能够在节点出现故障时自动恢复数据。它通过多副本机制保证了数据的冗余和一致性。不过，Longhorn 使用的是同步复制，这可能在网络延迟较高的环境中影响性能。
- **高可用性**：Longhorn 提供了较好的高可用性特性，能够在节点失效的情况下自动恢复存储卷，并确保数据的一致性和持久性。

**Ceph RBD**:

- **数据一致性**：Ceph 支持强一致性，并且支持副本和纠删码（Erasure Coding）两种方式来保证数据安全。通过多个副本或者纠删码机制，Ceph 能够在节点故障时快速恢复数据，并且它的同步机制在分布式系统中表现得更加可靠。
- **高可用性**：Ceph 是一个为高可用设计的系统，其架构能够处理多节点故障，并确保数据的安全和一致性。

**结论**：在数据一致性和高可用性方面，Ceph RBD 更为健壮，适合需要更高数据安全性和一致性的场景。

### 6. **社区支持和开发活跃度**

**Longhorn**:

- **社区支持**：Longhorn 是由 Rancher Labs 开发并开源的项目，随着 Rancher 在 Kubernetes 领域的影响力增长，Longhorn 也在快速发展，社区比较活跃。
- **开发活跃度**：尽管 Longhorn 是一个相对较新的项目，但其发布频率较高，开发进展稳健。随着 Kubernetes 的发展，Longhorn 也在不断优化和增强。

**Ceph RBD**:

- **社区支持**：Ceph 是一个成熟的开源项目，有着广泛的社区支持。作为一个经过多年的发展和生产验证的系统，Ceph 的社区规模庞大，拥有丰富的文档和经验分享。
- **开发活跃度**：Ceph 仍在不断更新，特别是针对性能和稳定性的持续改进。它的开发节奏较为稳定，新特性和功能的发布相对频繁。

**结论**：Ceph RBD 的社区和开发活跃度更高，更加适合那些需要长期技术支持和社区资源的大规模部署。

### 7. **场景适配**

**Longhorn**：

- 更适合中小型 Kubernetes 集群；
- 适用于一般业务系统、开发测试环境，或者性能要求相对一般的工作负载；
- 对于那些想要快速部署、简化运维的团队是一个理想的选择。

**Ceph RBD**：

- 更适合大规模集群和复杂的企业级环境；
- 适用于高性能、低延迟、高并发的生产环境，例如数据库、虚拟机存储等；
- 对存储性能、数据一致性和高可用性要求较高的企业。

------

- ### **总结和建议**

  - 如果您所在的公司 **需要快速部署、维护简便、并且性能要求适中**，且集群规模较小或中等规模，那么 **Longhorn** 是一个较为合适的选择。
  - 如果您的业务对 **性能、扩展性、数据一致性和高可用性** 有更高的要求，尤其是在 **高并发、大规模集群** 中部署，那么 **Ceph RBD** 将更为适合，尽管它的部署和维护相对复杂，但在大规模场景下表现更为出色。

  根据以上分析，您可以根据公司现有的技术储备、业务需求以及未来的扩展计划做出选择。如果团队的运维能力较强，并且未来有扩展需求，Ceph RBD 是更好的选择；如果您的团队规模较小，且希望快速获得稳定的存储系统，Longhorn 则更为合适。















很高兴听到您已经成功安装了 Longhorn UI。

您提到 Longhorn 只在两个节点（`node1` 和 `node2`）处于 **Schedulable** 状态，而 **master** 节点没有。这是因为 Kubernetes 默认在 **master 节点**上添加了 **污点（Taint）**，以防止工作负载被调度到 master 节点上。

------

### **回答您的问题**

**一般情况下，在生产环境中，不建议移除 master 节点上的污点。** 这是为了确保集群的稳定性和安全性。但是，如果您的集群资源有限，且希望让 master 节点参与存储资源的调度，可以考虑移除污点。

------

### **详细解释**

#### **1. 为什么 master 节点被污点（Tainted）？**

- **默认行为**：Kubernetes 为了保护控制平面的性能和稳定性，默认会在 master 节点上添加一个污点，阻止 Pods 被调度到该节点。
- **污点详情**：这个污点通常是 `node-role.kubernetes.io/master:NoSchedule`，这意味着除非 Pod 明确地容忍（Tolerate）该污点，否则不会被调度到 master 节点。

#### **2. Longhorn 对节点的要求**

- **Longhorn 组件**：Longhorn 需要在节点上运行一些 DaemonSet，包括 `longhorn-manager` 和 `longhorn-driver` 等。
- **Schedulable 节点**：Longhorn 只会在标记为 **Schedulable** 的节点上调度其组件和存储卷。

#### **3. 是否需要移除 master 节点的污点？**

- **生产环境（推荐保留污点）**：
  - **理由**：保持 master 节点的污点，以确保控制平面的安全和性能。
  - **影响**：Longhorn 不会在 master 节点上调度存储卷，您的存储容量仅限于 `node1` 和 `node2`。
- **资源有限的环境（可以考虑移除污点）**：
  - **理由**：在测试环境或小型集群中，资源有限，希望充分利用所有节点的资源，包括 master 节点。
  - **影响**：移除污点后，Longhorn 和其他工作负载可以被调度到 master 节点。

------

### **如何移除 master 节点的污点**

如果您决定让 master 节点也参与到 Longhorn 的存储调度中，可以按照以下步骤移除污点：

#### **步骤 1：查看当前的污点**

```
bash
复制代码
kubectl describe node master
```

在输出中，您会看到类似以下的污点信息：

```
bash
复制代码
Taints:
  node-role.kubernetes.io/master:NoSchedule
```

#### **步骤 2：移除污点**

执行以下命令移除污点：

```
bash
复制代码
kubectl taint nodes master node-role.kubernetes.io/master:NoSchedule-
```

注意命令末尾的 **减号（-）**，表示移除指定的污点。

#### **步骤 3：验证污点已移除**

再次查看节点的污点信息，确保污点已被移除：

```
bash
复制代码
kubectl describe node master
```

如果 `Taints` 列表为空，或者不再包含 `node-role.kubernetes.io/master:NoSchedule`，则表示污点已成功移除。

#### **步骤 4：更新 Longhorn 节点状态**

等待一段时间，或者手动刷新 Longhorn UI，您应该会看到 master 节点现在显示为 **Schedulable** 状态。

------

### **注意事项**

- **控制平面性能**：将工作负载调度到 master 节点可能会影响控制平面的性能和稳定性。在生产环境中，通常不建议这样做。

- **资源监控**：如果您移除了污点，建议密切监控 master 节点的资源使用情况，避免 CPU、内存过载。

- **调度策略**：如果您只希望 Longhorn 使用 master 节点，而不希望其他应用的 Pods 被调度到 master 节点，可以采取以下措施：

  - **为 master 节点添加自定义污点**：

    ```
    bash
    复制代码
    kubectl taint nodes master node-role.kubernetes.io/master-storage:NoSchedule
    ```

  - **为 Longhorn Pods 添加相应的容忍（tolerations）**：

    编辑 Longhorn 的部署 YAML 文件，为其添加容忍：

    ```
    yaml
    复制代码
    tolerations:
    - key: "node-role.kubernetes.io/master-storage"
      operator: "Exists"
      effect: "NoSchedule"
    ```

  - **这样，只有带有相应容忍的 Pods（例如 Longhorn 的组件）才能被调度到 master 节点，其他 Pods 则不会被调度到该节点。**

------

### **总结**

- 是否移除 master 节点的污点

  ，取决于您的集群资源和需求。

  - **生产环境**：建议保留污点，确保控制平面的稳定性。
  - **测试环境或资源有限的情况下**：可以移除污点，充分利用所有节点的资源。

- **移除污点后**，Longhorn 将能够在 master 节点上调度其组件和存储卷，提高存储容量和可用性。

- **如果只希望 Longhorn 使用 master 节点**，可以通过添加自定义污点和容忍，控制调度策略。

------

### **建议**

- **评估需求和风险**：在移除污点之前，仔细评估这样做的必要性和可能带来的风险。
- **监控资源使用**：使用 Kubernetes 的监控工具，密切关注 master 节点的资源消耗。
- **规划集群扩展**：如果可能，考虑增加工作节点，避免在 master 节点上运行工作负载。



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