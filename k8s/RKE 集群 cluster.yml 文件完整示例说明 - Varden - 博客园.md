## 最小文件示例

```
nodes:
  - address: 1.2.3.4
    user: ubuntu
    role:
      - controlplane
      - etcd
      - worker
```

## 完整文件示例

```
nodes:  # 节点选项配置
  - address: 1.1.1.1  # 用于设置节点的主机名或 IP 地址。RKE 必须能够连接到这个地址。
    user: ubuntu  # 对于每个节点，您指定连接到这个节点时要使用的user。这个用户必须是 Docker 组的成员，或者允许向节点的 Docker 套接字写入。
    role:  # 支持的角色有三种：controlplane、etcd和worker。controlplane、etcd和worker。节点角色不是相互排斥的。可以将任意角色组合分配给任何节点。也可以通过升级过程改变节点的角色。
      - controlplane
      - etcd
    port: 2222  # 在每个节点中，您可以指定连接到这个节点时要使用的端口。默认的端口是22。
    docker_socket: /var/run/docker.sock  # 如果 Docker 套接字和默认的不一样，您可以设置docker_socket。默认是/var/run/docker.sock。
  - address: 2.2.2.2
    user: ubuntu
    role:
      - worker
    ssh_key_path: /home/user/.ssh/id_rsa  # 对于每个节点，您可以指定路径，即ssh_key_path，用于连接到这个节点时要使用的 SSH 私钥。每个节点的默认密钥路径是~/.ssh/id_rsa。每个节点中设置的 SSH 密钥路径总是优先集群级SSH密钥的。
    ssh_key: |-  # 您可以不设置 SSH 密钥的路径，而是指定实际的密钥，即ssh_key，用来连接到节点。
      -----BEGIN RSA PRIVATE KEY-----
      -----END RSA PRIVATE KEY-----
    ssh_cert_path: /home/user/.ssh/test-key-cert.pub  # 对于每个节点，您可以指定路径，即ssh_cert_path，用于连接到这个节点时要使用的签名 SSH 证书。
    ssh_cert: |-  # 您可以指定实际的证书，即ssh_cert，用来连接到节点，而不是设置签名的 SSH 证书的路径。
      ssh-rsa-cert-v01@openssh.com AAAAHHNzaC1yc2EtY2VydC12MDFAb3Bl....
  - address: example.com
    user: ubuntu
    role:
      - worker
    hostname_override: node3  # 用于能够提供一个友好的名称，供 RKE 在 Kubernetes 中注册节点时使用。这个 hostname 不需要是可路由地址，但必须是一个有效的Kubernetes 资源名。如果没有设置hostname_override，那么在 Kubernetes 中注册节点时就会使用address指令。
    internal_address: 192.168.1.6  # 提供了让具有多个地址的节点设置一个特定的地址用于私有网络上的主机间通信的能力。如果没有设置internal_address，则使用address进行主机间通信。
    labels:  # 您可以为每个节点添加一个任意的标签映射。当使用入口控制器的node_selector选项时，可以使用它。
      app: ingress
    taints:  # 您能够为每个节点添加污点。
      - key: test-key
        value: test-value
        effect: NoSchedule
# If set to true, RKE will not fail when unsupported Docker version
# are found
ignore_docker_version: false  # 检查 Docker 版本

# Enable running cri-dockerd
# Up to Kubernetes 1.23, kubelet contained code called dockershim 
# to support Docker runtime. The replacement is called cri-dockerd 
# and should be enabled if you want to keep using Docker as your
# container runtime
# Only available to enable in Kubernetes 1.21 and higher
enable_cri_dockerd: true  # cri-dockerd

# Cluster level SSH private key
# Used if no ssh information is set for the node
ssh_key_path: ~/.ssh/test  # 集群级 SSH 密钥路径
# Enable use of SSH agent to use SSH private keys with passphrase
# This requires the environment `SSH_AUTH_SOCK` configured pointing
#to your SSH agent which has the private key added
ssh_agent_auth: true  # SSH Agent
# List of registry credentials
# If you are using a Docker Hub registry, you can omit the `url`
# or set it to `docker.io`
# is_default set to `true` will override the system default
# registry set in the global settings
private_registries:  # RKE 支持在cluster.yml中配置多个私有镜像仓库。您可以在cluster.yml中配置私有镜像仓库和凭证，然后从私有镜像仓拉取镜像。
     - url: registry.com  # 如果您使用的是 Docker Hub 镜像仓库，您可以省略url，或者将url的值设置为 docker.io。
       user: Username
       password: password
       is_default: true  # 默认镜像仓库，所有的系统镜像都将使用该镜像仓库进行拉取。
# 从 v0.1.10 开始，您必须配置您的私有镜像仓库凭证，但您可以指定这个镜像仓库为默认镜像仓库，这样所有的系统镜像都会从指定的私有镜像仓库中提取。您可以使用rke config --system-images命令来获取默认系统映像的列表来填充您的私有镜像仓库。
# Bastion/Jump host configuration
bastion_host:  # 堡垒主机配置
    address: x.x.x.x  # address指令用于设置堡垒主机的主机名或 IP 地址。RKE 必须能够连接到这个地址。
    user: ubuntu  # 指定连接到这个节点时要使用的用户。
    port: 22  # 您可以指定连接到堡垒主机时要使用的 端口，如果不指定，会使用默认端口22。
    ssh_key_path: /home/user/.ssh/bastion_rsa  # 指定路径，即ssh_key_path，用于连接堡垒主机时要使用的 SSH 私钥。
# or
#   ssh_key: |-  # 你可以指定连接到堡垒主机使用的密钥，即ssh_key，这样的话就不需要设置 SSH 密钥的路径ssh_key_path。
#     -----BEGIN RSA PRIVATE KEY-----
#
#     -----END RSA PRIVATE KEY-----
# Set the name of the Kubernetes cluster
cluster_name: mycluster  # 集群名称
# The Kubernetes version used. The default versions of Kubernetes
# are tied to specific versions of the system images.
#
# For RKE v0.2.x and below, the map of Kubernetes versions and their system images is
# located here:
# https://github.com/rancher/types/blob/release/v2.2/apis/management.cattle.io/v3/k8s_defaults.go
#
# For RKE v0.3.0 and above, the map of Kubernetes versions and their system images is
# located here:
# https://github.com/rancher/kontainer-driver-metadata/blob/master/rke/k8s_rke_system_images.go
#
# In case the kubernetes_version and kubernetes image in
# system_images are defined, the system_images configuration
# will take precedence over kubernetes_version.
kubernetes_version: v1.10.3-rancher2  # K8S版本
# System Images are defaulted to a tag that is mapped to a specific
# Kubernetes Version and not required in a cluster.yml.
# Each individual system image can be specified if you want to use a different tag.
#
# For RKE v0.2.x and below, the map of Kubernetes versions and their system images is
# located here:
# https://github.com/rancher/types/blob/release/v2.2/apis/management.cattle.io/v3/k8s_defaults.go
#
# For RKE v0.3.0 and above, the map of Kubernetes versions and their system images is
# located here:
# https://github.com/rancher/kontainer-driver-metadata/blob/master/rke/k8s_rke_system_images.go
#
system_images:  # 系统镜像，优先级高于kubernetes_version
    kubernetes: rancher/hyperkube:v1.10.3-rancher2
    etcd: rancher/coreos-etcd:v3.1.12
    alpine: rancher/rke-tools:v0.1.9
    nginx_proxy: rancher/rke-tools:v0.1.9
    cert_downloader: rancher/rke-tools:v0.1.9
    kubernetes_services_sidecar: rancher/rke-tools:v0.1.9
    kubedns: rancher/k8s-dns-kube-dns-amd64:1.14.8
    dnsmasq: rancher/k8s-dns-dnsmasq-nanny-amd64:1.14.8
    kubedns_sidecar: rancher/k8s-dns-sidecar-amd64:1.14.8
    kubedns_autoscaler: rancher/cluster-proportional-autoscaler-amd64:1.0.0
    pod_infra_container: rancher/pause-amd64:3.1
services:  # 默认的K8S服务
    etcd:
      # Custom uid/guid for etcd directory and files
      uid: 52034
      gid: 52034
      # if external etcd is used  # 外部etcd
      # path: /etcdcluster
      # external_urls:
      #   - https://etcd-example.com:2379
      # ca_cert: |-
      #   -----BEGIN CERTIFICATE-----
      #   xxxxxxxxxx
      #   -----END CERTIFICATE-----
      # cert: |-
      #   -----BEGIN CERTIFICATE-----
      #   xxxxxxxxxx
      #   -----END CERTIFICATE-----
      # key: |-
      #   -----BEGIN PRIVATE KEY-----
      #   xxxxxxxxxx
      #   -----END PRIVATE KEY-----
    # Note for Rancher v2.0.5 and v2.0.6 users: If you are configuring
    # Cluster Options using a Config File when creating Rancher Launched
    # Kubernetes, the names of services should contain underscores
    # only: `kube_api`.
    kube-api:
      # IP range for any services created on Kubernetes  在Kubernetes上创建的服务的IP范围。
      # This must match the service_cluster_ip_range in kube-controller  必须与kube-controller中的service_cluster_ip_range匹配
      service_cluster_ip_range: 10.43.0.0/16  # 这是将分配给 Kubernetes 上创建的服务的虚拟 IP 地址。默认情况下，服务集群 IP 范围是10.43.0.0/16。如果您改变了这个值，那么也必须在 Kubernetes 控制器管理器（kube-controller）上设置相同的值。
      # Expose a different port range for NodePort services 为NodePort服务提供不同的端口范围
      service_node_port_range: 30000-32767  # 使用type NodePort创建的 Kubernetes 服务的端口范围。默认情况下，端口范围是30000-32767。
      pod_security_policy: false  # 启用Kubernetes Pod 安全策略的选项。默认情况下，我们不会启用 pod 安全策略，因为它被设置为false。注意：如果您将pod_security_policy值设置为true，RKE 将配置一个开放的策略，允许任何 pod 在集群上工作。您需要配置您自己的策略来充分利用 PSP。
      # Encrypt secret data at Rest
      # Available as of v0.3.1
      secrets_encryption_config:  # 管理 Kubernetes 静态数据加密。https://docs.rancher.cn/docs/rke/config-options/secrets-encryption/_index
        enabled: true  # 启用加密功能
        custom_config:
          apiVersion: apiserver.config.k8s.io/v1
          kind: EncryptionConfiguration
          resources:
          - resources:
            - secrets
            providers:
            - aescbc:
                keys:
                - name: k-fw5hn
                  secret: RTczRjFDODMwQzAyMDVBREU4NDJBMUZFNDhCNzM5N0I=
            - identity: {}
      # Enable audit logging
      # Available as of v1.0.0
      audit_log:  # Kubernetes 审计提供了关于集群的安全相关的时间顺序记录集。Kube-apiserver 执行审计，每个请求都会产生一个事件，然后根据一定的策略进行预处理，并写入后端。策略决定了记录的内容，而后端则会将记录持久化。为了遵守 CIS（Center for Internet Security）Kubernetes Benchmark，您需要配置审计日志。
        enabled: true  # 启用审计日志
        configuration:
          max_age: 6  # 最大年龄，保留旧审计日志文件的最长天数。
          max_backup: 6  # 最大备份数量，保留旧审计日志文件的最大数量。
          max_size: 110  # 最大大小（兆字节），审计日志文件在被替换之前的最大大小（MB）。
          path: /var/log/kube-audit/audit-log.json  # 审计日志路径，日志后端用于写入审计事件的日志文件路径。
          format: json  # 日志文件格式
          policy:
            apiVersion: audit.k8s.io/v1 # This is required.
            kind: Policy
            omitStages:
              - "RequestReceived"
            rules:
              # Log pod changes at RequestResponse level
              - level: RequestResponse  # 在RequestResponse级别记录pod变化
                resources:
                - group: ""
                  # Resource "pods" doesn't match requests to any subresource of pods,  资源 "pods "不匹配对pods的任何子资源的请求
                  # which is consistent with the RBAC policy.  与RBAC策略是一致的
                  resources: ["pods"]

              # 在元数据层记录 "pods/log"、"pods/status"
              - level: Metadata
                resources:
                  - group: ""
                    resources: ["pods/log", "pods/status"]

              # 不要将请求记录到名为 "controller-leader "的配置图上
              - level: None
                resources:
                  - group: ""
                    resources: ["configmaps"]
                    resourceNames: ["controller-leader"]

              # 不要在端点或服务上记录 "system:keube-proxy "的监视请求
              - level: None
                users: ["system:kube-proxy"]
                verbs: ["watch"]
                resources:
                  - group: "" # core API group
                    resources: ["endpoints", "services"]

              # 不要记录对某些非资源URL路径的认证请求
              - level: None
                userGroups: ["system:authenticated"]
                nonResourceURLs:
                  - "/api*" # Wildcard matching.
                  - "/version"

              # 在kube-system中记录configmap变更的请求体
              - level: Request
                resources:
                  - group: "" # core API group
                    resources: ["configmaps"]
                # 此规则只适用于 "kube-system "命名空间中的资源
                # 空字符串""可用于选择非命名间隔的资源
                namespaces: ["kube-system"]

              # 在元数据级别记录所有其他命名空间的configmap和密钥变化
              - level: Metadata
                resources:
                  - group: "" # core API group
                    resources: ["secrets", "configmaps"]

              # 在请求层记录核心和扩展的所有其他资源
              - level: Request
                resources:
                  - group: "" # core API group
                  - group: "extensions" # 不应包括组的版本

              # 一个全面的规则，用于记录元数据级别的所有其他请求
              - level: Metadata
                # 在此规则下，像监控这样的长期运行的请求不会在RequestReceived中产生审计事件
                omitStages:
                  - "RequestReceived"
      # Using the EventRateLimit admission control enforces a limit on the number of events
      # that the API Server will accept in a given time period
      # Available as of v1.0.0
      event_rate_limit:  # 使用 EventRateLimit接纳控制对 API 服务器在特定时间段内接受的事件数量进行限制。在一个大型多租户集群中，可能会有一小部分租户用事件请求淹没服务器，这可能会对集群的整体性能产生重大影响。因此，建议限制 API 服务器接受事件的速率。
        enabled: true
        configuration:
          apiVersion: eventratelimit.admission.k8s.io/v1alpha1
          kind: Configuration
          limits:
          - type: Server  # 默认值：Server
            qps: 6000  # 默认值：5000
            burst: 30000  # 默认值：20000
      # Enable AlwaysPullImages Admission controller plugin  启用AlwaysPullImagesAdmission controller插件
      # Available as of v0.2.0  v0.2.0或更新版本可用
      always_pull_images: false  # 启用AlwaysPullImages Admission 控制器插件。启用AlwaysPullImages是一个安全的最佳实践。它强制 Kubernetes 与远程镜像镜像仓库验证镜像和拉动凭证。本地镜像层缓存仍将被使用，但它确实会在启动容器拉取和比较镜像哈希时增加一点开销。注：从 v0.2.0 开始提供。
      # Add additional arguments to the kubernetes API server
      # This WILL OVERRIDE any existing defaults
      extra_args:
        # Enable audit log to stdout
        audit-log-path: "-"
        # Increase number of delete workers
        delete-collection-workers: 3
        # Set the level of log output to debug-level
        v: 4
    # Note for Rancher 2 users: If you are configuring Cluster Options
    # using a Config File when creating Rancher Launched Kubernetes,
    # the names of services should contain underscores only:
    # `kube_controller`. This only applies to Rancher v2.0.5 and v2.0.6.
    kube-controller:
      # CIDR pool used to assign IP addresses to pods in the cluster
      cluster_cidr: 10.42.0.0/16  # 用于为集群中的 pod 分配 IP 地址的 CIDR 池。默认情况下，集群中的每个节点都会从该池中分配一个/24网络，用于分配 pod IP。该选项的默认值是10.42.0.0/16。
      # IP range for any services created on Kubernetes
      # This must match the service_cluster_ip_range in kube-api
      service_cluster_ip_range: 10.43.0.0/16  # 这是将分配给 Kubernetes 上创建的服务的虚拟 IP 地址。默认情况下，服务集群 IP 范围是10.43.0.0/16。如果您改变了这个值，那么也必须在 Kubernetes API 服务器（kube-api）上设置相同的值。
      # Add additional arguments to the kubernetes API server
      # This WILL OVERRIDE any existing defaults
      extra_args:
        # Set the level of log output to debug-level
        v: 4
        # Enable RotateKubeletServerCertificate feature gate
        feature-gates: RotateKubeletServerCertificate=true
        # Enable TLS Certificates management
        # https://kubernetes.io/docs/tasks/tls/managing-tls-in-a-cluster/
        cluster-signing-cert-file: "/etc/kubernetes/ssl/kube-ca.pem"
        cluster-signing-key-file: "/etc/kubernetes/ssl/kube-ca-key.pem"
    kubelet:
      # Base domain for the cluster
      cluster_domain: cluster.local  # 集群的基本域。集群上创建的所有服务和 DNS 记录。默认情况下，域被设置为cluster.local。
      # IP address for the DNS service endpoint
      cluster_dns_server: 10.43.0.10  # 分配给集群内 DNS 服务端点的 IP 地址。DNS 查询将被发送到这个 IP 地址，该地址被 KubeDNS 使用。这个选项的默认值是10.43.0.10。
      # Fail if swap is on
      fail_swap_on: false  # 在 Kubernetes 中，如果节点上启用了 swap，kubelet 的默认行为是失败。RKE 不会遵循这个默认值，而是允许在启用 swap 的节点上进行部署。默认情况下，这个值是false。如果您想恢复到默认的 kubelet 行为，请将此选项设置为 "true"。
      # Configure pod-infra-container-image argument
      pod-infra-container-image: "k8s.gcr.io/pause:3.2"
      # Generate a certificate signed by the kube-ca Certificate Authority
      # for the kubelet to use as a server certificate
      # Available as of v1.0.0
      generate_serving_certificate: true  # 为 kubelet 生成一个由kube-ca证书颁发机构签署的证书作为服务器证书。这个选项的默认值是false。在启用这个选项之前，请参考下文的 Kubelet Serving Certificate 需求。
# 如果在cluster.yml中为一个或多个节点配置了hostname_override，请确保在address中配置了正确的 IP 地址（以及internal_address中的内部地址），以确保生成的证书包含正确的 IP 地址。
# 一个错误的例子是，在 EC2 实例中，如果在address中配置了公共 IP 地址，并且使用了hostname_override，那么kube-apiserver和kubelet之间的连接将失败，因为kubelet将在私有 IP 地址上被联系，生成的证书将无效（将看到错误x509：证书对 value_in_address 有效，而不是 private_ip）。解决方法是在internal_address中提供内部 IP 地址。
      extra_args:  # 自定义参数
        # Set max pods to 250 instead of default 110
        max-pods: 250
        # Enable RotateKubeletServerCertificate feature gate
        feature-gates: RotateKubeletServerCertificate=true
      # Optionally define additional volume binds to a service
      extra_binds:  # Docker 挂载绑定
        - "/usr/libexec/kubernetes/kubelet-plugins:/usr/libexec/kubernetes/kubelet-plugins"
      extra_env:  # 环境变量
        - "HTTP_PROXY=http://your_proxy"
    scheduler:  # 目前，RKE 并不支持scheduler服务的任何特定选项。
      extra_args:
        # Set the level of log output to debug-level
        v: 4
    kubeproxy:  # 目前，RKE 不支持 "kubeproxy "服务的任何特定选项。
      extra_args:
        # Set the level of log output to debug-level
        v: 4
# Currently, only authentication strategy supported is x509.
# You can optionally create additional SANs (hostnames or IPs) to
# add to the API server PKI certificate.
# This is useful if you want to use a load balancer for the
# control plane servers.
authentication:
  strategy: x509
  sans:
    - "10.18.160.10"
    - "my-loadbalancer-1234567890.us-west-2.elb.amazonaws.com"

# Kubernetes Authorization mode
# Use `mode: rbac` to enable RBAC
# Use `mode: none` to disable authorization
authorization:  # 默认情况下，RKE 会启用 RBAC 授权策略。如果你想关闭 RBAC 授权策略，请在cluster.yml中将设置授权模式为none。虽然 RKE 支持手动关闭 RBAC 授权策略，但是关闭 RBAC 授权策略会有一定的风险，我们不建议用户关闭 RBAC 授权策略。
  mode: rbac

# If you want to set a Kubernetes cloud provider, you specify
# the name and configuration
cloud_provider:  # RKE 支持为 Kubernetes 集群设置特定的云服务提供商。这些云服务提供商有特定的配置参数。RKE 支持的云服务提供商如下：AWS、Azure、OpenStack、vSphere。
  name: aws

# Add-ons are deployed using kubernetes jobs. RKE will give
# up on trying to get the job status after this timeout in seconds..
addon_job_timeout: 30  # RKE 使用 Kubernetes job 的方式部署插件。在某些情况下，部署插件的时间会比预期时间长。从 v0.1.7 开始，RKE 提供集群层级的addon_job_timeout选项，以检查 job 的连接是否超时，默认值为 30，单位是秒。

# Specify network plugin-in (canal, calico, flannel, weave, or none)
network:  # 网络插件配置
  plugin: canal  # RKE 提供了以下网络插件，作为附加组件部署：flannel、calico、canal、weave。默认情况下，RKE 使用的网络插件是canal。
  # Specify MTU
  mtu: 1400
  options:
    # Configure interface to use for Canal
    canal_iface: eth1  # 通过设置canal_iface，可以配置主机间通信使用的接口。
    canal_flannel_backend_type: vxlan  # canal_flannel_backend_type选项允许你指定要使用的flannel backend的类型。默认情况下使用vxlan后端。
    # Available as of v1.2.6
    canal_autoscaler_priority_class_name: system-cluster-critical
    canal_priority_class_name: system-cluster-critical
  # Available as of v1.2.4
  tolerations:  # Canal 网络插件容忍度
  - key: "node.kubernetes.io/unreachable"
    operator: "Exists"
    effect: "NoExecute"
    tolerationseconds: 300
  - key: "node.kubernetes.io/not-ready"
    operator: "Exists"
    effect: "NoExecute"
    tolerationseconds: 300
  # Available as of v1.1.0
  update_strategy:
    strategy: RollingUpdate
    rollingUpdate:
      maxUnavailable: 6

# Specify DNS provider (coredns or kube-dns)
dns:  # DNS服务配置
  provider: coredns  # 在 RKE v0.2.5 及更新版本中，使用 Kubernetes 1.14 及以上版本时，CoreDNS 是默认 DNS 提供商。
  # Available as of v1.1.0
  update_strategy:
    strategy: RollingUpdate
    rollingUpdate:
      maxUnavailable: 20%
      maxSurge: 15%
  linear_autoscaler_params:
    cores_per_replica: 0.34
    nodes_per_replica: 4
    prevent_single_point_failure: true
    min: 2
    max: 3
  node_selector:  # 如果只想在特定的节点上部署 CoreDNS pod ，可以在dns部分设置一个node_selector。node_selector中的标签需要与要部署 CoreDNS pod 的节点上的标签相匹配。
    app: dns
  upstreamnameservers:  # 默认情况下，CoreDNS 使用主机配置的命名服务器（通常保存在/etc/resolv.conf路径下）来解析外部查询。如果想配置特定的上游名称服务器，可以使用upstreamnameservers指令。
    - 1.1.1.1
    - 8.8.4.4
  tolerations:  # 配置的容忍度适用于coredns和coredns-autoscaler部署。
    - key: "node.kubernetes.io/unreachable"
      operator: "Exists"
      effect: "NoExecute"
      tolerationseconds: 300
    - key: "node.kubernetes.io/not-ready"
      operator: "Exists"
      effect: "NoExecute"
      tolerationseconds: 300
  options:  # CoreDNS 优先级类名称
    coredns_autoscaler_priority_class_name: system-cluster-critical
    coredns_priority_class_name: system-cluster-critical
# Specify monitoring provider (metrics-server)
monitoring:  # 默认情况下，RKE 会部署 Metrics Server来提供集群中资源的指标。RKE 会将 Metrics Server 部署为一个 Deployment。
  provider: metrics-server  # 您可以将provider的值修改为none，禁用默认控制器。
  metrics_server_priority_class_name: system-cluster-critical  # 度量衡服务器优先级类别名称
  # Available as of v1.1.0
  update_strategy:
    strategy: RollingUpdate
    rollingUpdate:
      maxUnavailable: 8
  tolerations:  # 容忍度
    - key: "node.kubernetes.io/unreachable"
      operator: "Exists"
      effect: "NoExecute"
      tolerationseconds: 300
    - key: "node.kubernetes.io/not-ready"
      operator: "Exists"
      effect: "NoExecute"
      tolerationseconds: 300
# Currently only nginx ingress provider is supported.
# To disable ingress controller, set `provider: none`
# `node_selector` controls ingress placement and is optional
ingress:  # 默认情况下，RKE 会在所有可调度节点上部署 NGINX ingress controller。从 v0.1.8 开始，只有 worker 节点是可调度节点。RKE 将以 DaemonSet 的形式部署 Ingress Controller，并使用 hostnetwork: true，因此在部署控制器的每个节点上都会打开 80和443端口。
  provider: nginx
  node_selector:  # 如果只想在特定的节点上部署 Ingress Controller ，可以在dns部分设置一个node_selector。node_selector中的标签需要与要部署 Ingress Controller 的节点上的标签相匹配。
    app: ingress
  # Available as of v1.1.0
  update_strategy:
    strategy: RollingUpdate
    rollingUpdate:
      maxUnavailable: 5
  extra_envs:  # 环境变量
    - name: TZ
      value: Asia/Shanghai
  ingress_priority_class_name: system-cluster-critical  # 入站优先级类别名称
  tolerations:  # 容忍度
    - key: "node.kubernetes.io/unreachable"
      operator: "Exists"
      effect: "NoExecute"
      tolerationseconds: 300
    - key: "node.kubernetes.io/not-ready"
      operator: "Exists"
      effect: "NoExecute"
      tolerationseconds: 300
  options:  # Kubernetes 中有 NGINX 选项：NGINX 配置图的选项列表、命令行 extra_args和注释。
    map-hash-bucket-size: "128"
    ssl-protocols: SSLv2
  extra_args:
    enable-ssl-passthrough: ""
    default-ssl-certificate: "ingress-nginx/ingress-default-cert"  # 默认证书由下面的自定义插件生成
  default_backend: false  # 禁用 NGINX Ingress 默认后端
  network_mode: hostNetwork  # 默认情况下，nginx ingress controller 使用 hostNetwork: true 配置，默认端口为80和443。对于 Kubernetes v1.21 及更高版本，NGINX ingress controller 不再运行在 hostNetwork: true 中，而是使用 hostPort 的端口 80 和端口 443。
# All add-on manifests MUST specify a namespace
addons: |-  # 自定义插件，直接嵌入。RKE 将 YAML 清单作为 configmap 上传至 Kubernetes 集群。然后，它运行一个 Kubernetes job，挂载 configmap 并使用kubectl apply -f部署插件。RKE 只有在多次使用rke up时才会添加额外的插件。当使用不同的附加组件列表进行rke up时，RKE 不支持删除集群附加组件。从 v0.1.8 开始，RKE 会更新同名的插件。如果要在 YAML 文件中直接定义一个插件，一定要使用 YAML 的 block indicator |-，因为addons指令是一个多行字符串选项。可以用---指令将多个 YAML 资源定义分开来指定。
  ---
  apiVersion: v1
  kind: Pod
  metadata:
    name: my-nginx
    namespace: default  # 说明：当使自定义的附加组件时，必须为你的所有资源定义一个命名空间，否则它们会进入kube-system命名空间。
  spec:
    containers:
    - name: my-nginx
      image: nginx
      ports:
      - containerPort: 80
  ---  # 下面清单由此命令生成：kubectl -n ingress-nginx create secret tls ingress-default-cert --cert=mycert.cert --key=mycert.key -o yaml --dry-run=true > ingress-default-cert.yaml，然后将 ingress-default-cert.yml的内容嵌入。
  apiVersion: v1
  data:
    tls.crt: [ENCODED CERT]
    tls.key: [ENCODED KEY]
  kind: Secret
  metadata:
    creationTimestamp: null
    name: ingress-default-cert
    namespace: ingress-nginx  # 说明：当使自定义的附加组件时，必须为你的所有资源定义一个命名空间，否则它们会进入kube-system命名空间。
  type: kubernetes.io/tls

addons_include:  # 引用插件的 YAML 文件。使用addons_include指令，提供自定义插件引用的本地文件路径或 URL 地址。
  - https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/rook-operator.yaml
  - https://raw.githubusercontent.com/rook/rook/master/cluster/examples/kubernetes/rook-cluster.yaml
  - /path/to/manifest
```