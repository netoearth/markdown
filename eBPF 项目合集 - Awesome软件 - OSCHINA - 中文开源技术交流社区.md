许可证：Apache-2.0 开发语言：C/C++、Python BCC 是一个工具包，用于创建基于 eBPF 的高效内核跟踪和操作程序，包括几个有用的命令行工具和示例。 BCC 简化了用 C 编写用于内核检测的 eBPF 程序，包括围绕 LLVM 的包装器，以及 Python 和 Lua 的前端。它还提供了一个用于直接集成到应用程序中的高级库。

![BCC Linux 动态跟踪工具](https://static.oschina.net/uploads/logo/bcc_wHVKI.png)

许可证：Apache-2.0 开发语言：Go、C 官网：https://cilium.io/ Cilium 是一个开源项目，提供由 eBPF 驱动的网络、安全性和可观察性。它经过专门设计，旨在将 eBPF 的优势带入 Kubernetes 世界，并满足容器工作负载的新可扩展性、安全性和可见性要求。

![Cilium 基于 BPF 用于容器的关键网络技术](https://static.oschina.net/uploads/logo/cilium_2pXcM.png)

许可证：Apache-2.0 开发语言：C/C++ 官网：https://bpftrace.org/ bpftrace 是 Linux eBPF 的高级跟踪语言。它的语言受到 awk 和 C 以及 DTrace 和 SystemTap 等前身跟踪器的启发。 bpftrace 使用 LLVM 作为后端将脚本编译为 eBPF 字节码，并利用 BCC 与 Linux BPF 系统进行交互。

![bpftrace Linux eBPF 的高级跟踪语言](https://static.oschina.net/uploads/logo/bpftrace_GtGJl.png)

许可证：Apache-2.0 开发语言：C/C++、Python 官网：https://falco.org/ Falco 是一种行为活动监视器，旨在检测应用程序中的异常活动。Falco 借助 eBPF 在 Linux 内核层审计系统。它通过容器运行时指标和 Kubernetes 指标等其他输入流丰富收集的数据，并允许持续监控和检测容器、应用程序、主机和网络活动。

![Falco Kubernetes 威胁检测引擎](https://static.oschina.net/uploads/logo/falco_nQ9kD.png)

许可证：GPL-2.0 开发语言：C/C++、Go Katran 是一个 C++ 库和 eBPF 程序，用于构建高性能的第 4 层负载平衡转发平面。 Katran 利用 Linux 内核中的 XDP 基础设施来提供用于快速数据包处理的内核设施。它的性能与 NIC 的接收队列数量成线性关系，并且它使用 RSS 友好封装转发到 L7 负载平衡器。

![Katran 高性能第 4 层负载均衡器](https://static.oschina.net/uploads/logo/katran_kNpsh.png)

许可证：Apache-2.0 开发语言：C/C++、Python、Go 官网：https://px.dev/ Pixie 是一个用于 Kubernetes 应用程序的开源可观察性工具。Pixie 使用 eBPF 自动捕获遥测数据，无需手动检测。开发人员可以使用 Pixie 查看其集群的高级状态（服务地图、集群资源、应用程序流量），还可以深入查看更详细的视图（pod 状态、火焰图、单独的 full body 应用程序请求）。

![Pixie 用于 Kubernetes 应用的可观察性工具](https://static.oschina.net/uploads/logo/pixie_N5xIc.png)

许可证：Apache-2.0 开发语言：Go、TypeScript 官网：https://pyroscope.io/ Pyroscope 是一个以持续分析为中心的开源项目，尤其是在 Kubernetes 环境中。它利用 eBPF 作为其核心技术以及自定义存储引擎，以最小的开销以及高效的存储和查询功能提供系统范围的连续分析。

![Pyroscope 连续分析平台](https://static.oschina.net/uploads/logo/pyroscope_0PRzw.png)

许可证：AGPL 开发语言：C/C++、Go 官网：https://ecapture.cc/ eCapture是一款Go语言编写的工具，无需CA证书即可抓取HTTPS/TLS明文。支持openssl、boringssl、gnutls、nspr等TLS加密库。它可以运行在 Linux 内核 4.18 或更高版本的 x86\_64 CPU 架构和 Linux/Android 内核 5.5 或更高版本的 aarch64 CPU 架构上，支持 CO-RE 和非 CO-RE 模式，无需 BTF。

![eCapture 用户态数据捕获工具](https://static.oschina.net/uploads/logo/ecapture_bEfFi.png)

许可证：Apache-2.0 开发语言：Go、JavaScript、TypeScript 官网：https://www.parca.dev/ Parca 是一个针对应用程序和基础设施的持续分析项目。持续分析以分析 CPU、内存使用情况，直至行号。节省基础设施成本、提高性能并提高可靠性。Parca 使用 eBPF 收集分析数据并使用 libbpf-go 与内核交互。

![Parca 针对开源基础设施的持续分析监控工具](https://static.oschina.net/uploads/logo/parca_XqfXJ.png)

许可证：Apache-2.0 开发语言：C/C++、Go 官网：https://aquasecurity.github.io/tracee/v0.9/ Tracee 使用 eBPF 技术来检测和过滤操作系统事件，帮助你揭示安全洞察、检测可疑行为并捕获取证指标。 Tracee 是用于基于 Linux 的云部署的运行时安全和取证工具。它使用 eBPF 在运行时跟踪主机操作系统和应用程序，并分析收集到的事件以检测可疑的行为模式。它可以在你的 kubernetes 环境中作为守护程序集运行，但可以灵活地在任何基于 Linux 的主机上出于多种目的运行。它可以通过 Helm 作为 docker 容器或作为一小组静态二进制文件交付。

![Tracee 使用 eBPF 的运行时安全和取证工具](https://static.oschina.net/uploads/logo/tracee_XX8x3.png)

许可证：Apache-2.0 开发语言：C/C++、Go Tetragon 是 Cilium 开源的基于 eBPF 的安全可观察性和运行时增强组件。在 Kubernetes 环境中使用时，Tetragon 是 Kubernetes 感知的 —— 也就是说，它了解 Kubernetes 身份，例如命名空间、Pod 等 —— 因此可以针对单个工作负载配置安全事件检测。

![Tetragon 基于 eBPF 的安全可观测性 & 运行时增强组件](https://static.oschina.net/uploads/logo/tetragon_kcI5c.png)

许可证：MIT 开发语言：Go kubectl-trace 是一个 kubectl 插件，它可以使用基于 bpftrace 的编程能力，来分析系统的性能问题。你只需编写好程序文件，就可以通过这个插件将其运行到 K8s 集群中，不需要任何额外工作。

![kubectl-trace kubectl 插件](https://static.oschina.net/uploads/logo/kubectl-trace_w0d5k.png)

许可证：Apache-2.0 开发语言：C/C++、Go 官网：https://www.inspektor-gadget.io/ Inspektor Gadget 是一个工具（或小工具）的集合，用于调试和检查 Kubernetes 资源和应用程序。它管理 Kubernetes 集群中 eBPF 程序的打包、部署和执行，包括许多基于 BCC 的工具，以及一些专门为 Inspektor Gadget 开发的工具。它自动将低级别的内核基元映射到高级别的 Kubernetes 资源，使其更容易和更快地找到相关信息。

![Inspektor Gadget 调试和检查 Kubernetes 应用和资源](https://static.oschina.net/uploads/logo/inspektor-gadget_kymWp.png)

许可证：MIT 开发语言：C/C++ Sysmon for Linux 是一种监视和记录系统活动的工具，包括进程生命周期、网络连接、文件系统写入等。Sysmon 在重新启动后工作，并支持高级过滤以帮助识别恶意活动以及入侵者和恶意软件如何在你的网络上运行。

![SysmonForLinux 监视和记录系统活动](https://static.oschina.net/uploads/logo/sysmonforlinux_7WRcI.png)

许可证：GPL-2.0 开发语言：C/C++、Go pwru 是一个基于 eBPF 的工具，用于追踪 Linux 内核中的网络数据包，具有高级过滤能力。pwru 支持对内核状态进行细粒度的内省 (introspection )，方便调试网络连接问题。

![pwru Linux 内核网络调试工具](https://static.oschina.net/uploads/logo/pwru_56VVZ.png)

许可证：Apache-2.0 开发语言：C/C++、Go 官网：https://bumblebee.io/ZH BumbleBee 简化了 eBPF 工具的构建，并允许你使用 OCI 图像打包、分发和运行 eBPF 程序。它允许你只关注代码的 eBPF 部分，而 BumbleBee 会自动处理 boilerplate，包括用户空间代码。

![BumbleBee OCI 兼容的 eBPF 工具](https://static.oschina.net/uploads/logo/bumblebee_fnXzo.png)

许可证：Apache-2.0 开发语言：C/C++、Go 官网：https://wkz.github.io/ply/ ply 是基于 eBPF 构建的 Linux 动态跟踪器。它的设计考虑了嵌入式系统，用 C 语言编写，运行 ply 所需的只是 libc 和支持 eBPF 的现代 Linux 内核，这意味着它的程序生成不依赖于 LLVM。它具有用于编写脚本的类 C 语法，并且深受 awk(1) 和 dtrace(1) 的启发。

![ply Linux 动态跟踪器](https://static.oschina.net/uploads/logo/ply_aQXPK.png)

许可证：Apache-2.0 开发语言：C/C++、Go、TypeScript 官网：http://kindling.harmonycloud.cn/ Kindling 是一个基于 eBPF 的云原生可观测性开源工具，旨在帮助用户理解应用从内核层到代码层的行为，以便用户更好更快的排查故障。 目前，它提供了一种简单的方式来获取 kubernetes 环境中的网络流视图，还提供了许多内置的网络监控概览信息：比如重传、DNS、吞吐量、TPS 等。

![Kindling 云原生可观测性开源工具](https://static.oschina.net/uploads/logo/kindling_4gbc5.png)

许可证：Apache-2.0 开发语言：C/C++、Go 官网：https://kubearmor.io/ KubeArmor 是一种云原生运行时安全执行系统，可在系统级别限制 Pod、容器和节点 (VM) 的行为（例如流程执行、文件访问和网络操作）。

![KubeArmor 云原生运行时安全执行系统](https://static.oschina.net/uploads/logo/kubearmor_g56rm.png)

许可证：Apache-2.0 开发语言：C/C++、Go 官网：https://merbridge.io/ Merbridge 旨在让 Service Mesh 的流量拦截和转发更加高效。借助 Merbridge，开发人员可以使用 eBPF 而不是 iptables 来加速他们的服务网格，而无需任何额外的操作或代码更改。目前，Merbridge 已经支持 Istio、Linkerd 和 Kuma。

![Merbridge 使用 eBPF 加速服务网格](https://static.oschina.net/uploads/logo/merbridge_2IQTu.png)