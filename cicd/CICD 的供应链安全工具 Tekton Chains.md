软件供应链是指进入软件中的所有内容及其来源，简单地可以理解成软件的依赖项。依赖项是软件运行时所需的重要内容，可以是代码、二进制文件或其他组件，也可以是这些组件的来源，比如存储库或者包管理器之类的。包括代码的已知漏洞、受支持的版本、许可证信息、作者、贡献时间，以及在整个过程中的行为和任何时候接触到它的任何内容，比如用于编译、分发软件的基础架构组件。

[[CICD]] 流水线作为基础架构组件，承担着软件的构建、测试、分发和部署。作为供应链的一环，其也成为恶意攻击的目标。[[CICD]] 的行为信息可以作为安全审查的重要依据，比如构建打包的环境、操作流程、处理结果等等。

今天介绍的 Tekton Chains，便是这样的行为收集工具。

## Tekton Chains

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/05/14/16524602574062.png)

[Chains](https://github.com/tektoncd/chains) 是一个 Kubernetes CRD 控制器，用于管理 Tekton 中的供应链安全。Chains 使 Tekton 能够在持续交付中，捕获 `PipelineRun` 和 `TaskRun` 运行的元数据用于安全审计，实现二进制溯源和可验证的构建，成为构成保证供应链安全的基础设施的一部分。

当前 Chains（v0.9.0）提供如下功能：

-   使用用户提供的加密密钥，将 TaskRun 的结果（包含 TaskRun 本身和 OCI 镜像）镜像进行签名
-   支持类似 [in-toto](https://in-toto.io/) 的证明格式
-   使用多种加密密钥类型和服务（x509、KMS）进行签名
-   签名的存储支持多种实现

接下来的 Demo，我们还是继续使用之前在 [Tekton Pipeline 实战](https://atbug.com/tekton-pipeline-practice/) 用的 Java 项目 [tekton-test](https://github.com/addozhang/tekton-test)，在代码中同样还包含了 `Pipeline` 和 `Task` 的定义。

为了支持 Chains，原有 task 增加了新的参数。

## Demo

### 环境准备

使用 k3d 快速搭建 k3s 的集群

```
k3d cluster create tekton-test
```

安装 Teketon Pipelines 及 CLI

```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/pipeline/latest/release.yaml
brew install tektoncd-cli

kubectl get po -n tekton-pipelines
NAME                                           READY   STATUS    RESTARTS   AGE
tekton-pipelines-webhook-7f5d9fc745-cr7th      1/1     Running   0          12m
tekton-pipelines-controller-7c95d87d96-vvkgr   1/1     Running   0          12m
```

安装 tekton chains

```
kubectl apply --filename https://storage.googleapis.com/tekton-releases/chains/latest/release.yaml

kubectl get po -n tekton-chains
NAME                                        READY   STATUS    RESTARTS   AGE
tekton-chains-controller-7d9fb75899-l9d4j   1/1     Running   0          9m8s
```

安装流水线，这里还是使用 tekton-test 中的流水线定义，详细的说明查看[Tekton Pipelines 实战](https://atbug.com/tekton-pipeline-practice/#%E5%AE%9E%E8%B7%B5) 中的说明。

```
cd tekton-test/tekton
#创建sa
kubectl apply -f serviceaccount.yaml
# 创建 docker hub 凭证 secret，这里需要命令行工具 jq
kubectl create secret docker-registry dockerhub --docker-server=https://index.docker.io/v1/ --docker-username=[USERNAME] --docker-password=[PASSWORD] --dry-run=client -o json | jq -r '.data.".dockerconfigjson"' | base64 -d > /tmp/config.json && kubectl create secret generic docker-config --from-file=/tmp/config.json -n tekton-pipelines && rm -f /tmp/config.json
#安装 git-clone task
tkn hub install task git-clone
#安装镜像构建 task
kubectl apply -f tasks/source-to-image.yaml
#安装部署 task
kubectl apply -f tasks/deploy-to-k8s.yaml
#安装 pipeline
kubectl apply -f pipeline/build-pipeline.yaml
#查看 task
tkn t list                  
NAME              DESCRIPTION              AGE
git-clone         These Tasks are Git...   2 minutes ago
source-to-image                            1 minute ago
deploy-to-k8s                              31 seconds ago
#查看 pipeline
tkn p list 
NAME             AGE              LAST RUN   STARTED   DURATION   STATUS
build-pipeline   35 seconds ago   ---        ---       ---        ---
```

### 配置

#### 1\. 添加认证凭证

Chains controller 需要使用进行：

-   在镜像签名完成后，向 OCI 镜像仓库发送签名
-   在使用密钥签名时，使用 Fulcio 获取签名证书

这里我们需要配置用于访问 Docker 镜像仓库的凭证，Chains controller 使用执行 `PipelineRun` 的 service account 将镜像签名推送到镜像仓库。

这里直接使用前面安装 pipeline 时创建的 secret，并授权 service account 访问 secret。

```
kubectl patch serviceaccount tekton-build \
  -p "{\"imagePullSecrets\": [{\"name\": \"docker-config\"}]}" -n tekton-pipelines 
```

#### 2\. 签名密钥

为 Chains 生成并配置用于签名的签名密钥，支持下面的任何一种方式：

-   x509
-   Cosign
-   KMS
-   EXPERIMENTAL: Keyless signing

我们使用 [cosign](https://github.com/sigstore/cosign) （安装非常简单，在 macOS 上执行 `brew install cosign`）创建 secret，输入密码时直接回车，不设置密码。

```
cosign generate-key-pair k8s://tekton-chains/signing-secrets
Enter password for private key: 
Enter password for private key again: 
Successfully created secret signing-secrets in namespace tekton-chains
Public key written to cosign.pub
```

下面就是创建的包含了密钥的 secret。

![](https://atbug.oss-cn-hangzhou.aliyuncs.com/2022/05/14/20220514-at-095829.png)

#### 3\. 配置 Chains

-   `artifacts.taskrun.format=in-toto`
-   `artifacts.taskrun.storage=tekton`
-   `artifacts.taskrun.signer=x509`

```
kubectl patch configmap chains-config -n tekton-chains -p='{"data":{"artifacts.taskrun.format": "in-toto"}}'
kubectl patch configmap chains-config -n tekton-chains -p='{"data":{"artifacts.taskrun.storage": "tekton"}}'
kubectl patch configmap chains-config -n tekton-chains -p='{"data":{"artifacts.taskrun.signer": "x509"}}'
```

### 验证

创建 `PipelineRun` 开始执行 `Pipeline`。

```
#tekton-test 根目录
kubectl apply -f ./tekton/run/run.yaml
pipelinerun.tekton.dev/generic-pipeline-run created
```

执行完成后，检查 `TaskRun` 的运行结果。

```
tkn tr list
NAME                                   STARTED        DURATION     STATUS
generic-pipeline-run-source-to-image   1 minute ago   1 minute     Succeeded
generic-pipeline-run-fetch-from-git    1 minute ago   11 seconds   Succeeded
```

```
kubectl get tr generic-pipeline-run-source-to-image -o json | jq -r .metadata.annotations

{
  "chains.tekton.dev/cert-taskrun-5bf67bed-af59-4744-a6a7-487de359cbae": "",
  "chains.tekton.dev/chain-taskrun-5bf67bed-af59-4744-a6a7-487de359cbae": "",
  "chains.tekton.dev/payload-taskrun-5bf67bed-af59-4744-a6a7-487de359cbae": "eyJfdHlwZSI6Imh0dHBzOi8vaW4tdG90by5pby9TdGF0ZW1lbnQvdjAuMSIsInByZWRpY2F0ZVR5cGUiOiJodHRwczovL3Nsc2EuZGV2L3Byb3ZlbmFuY2UvdjAuMiIsInN1YmplY3QiOlt7Im5hbWUiOiJpbmRleC5kb2NrZXIuaW8vYWRkb3poYW5nL3Rla3Rvbi10ZXN0IiwiZGlnZXN0Ijp7InNoYTI1NiI6ImFjYmQ5OWY4YmUwMDNlOGI3ZTc3NjY1ZWUwZTMzMmU5NmRhMmNiNTRiMzI3MTk4YWJiNDY3MWM4ZGIxZTk5OGEifX1dLCJwcmVkaWNhdGUiOnsiYnVpbGRlciI6eyJpZCI6Imh0dHBzOi8vdGVrdG9uLmRldi9jaGFpbnMvdjIifSwiYnVpbGRUeXBlIjoiaHR0cHM6Ly90ZWt0b24uZGV2L2F0dGVzdGF0aW9ucy9jaGFpbnNAdjIiLCJpbnZvY2F0aW9uIjp7ImNvbmZpZ1NvdXJjZSI6e30sInBhcmFtZXRlcnMiOnsiQ0hBSU5TLUdJVF9DT01NSVQiOiJ7c3RyaW5nIDQ2OTMxZmJjZjAzMDMyMGMzOTlmZjljMzA1MGQ2YjczMTI1OWFiZmMgW119IiwiQ0hBSU5TLUdJVF9VUkwiOiJ7c3RyaW5nIGh0dHBzOi8vZ2l0aHViLmNvbS9hZGRvemhhbmcvdGVrdG9uLXRlc3QuZ2l0IFtdfSIsIklNQUdFIjoie3N0cmluZyBhZGRvemhhbmcvdGVrdG9uLXRlc3QgW119IiwiaW1hZ2VUYWciOiJsYXRlc3QiLCJpbWFnZVVybCI6IntzdHJpbmcgYWRkb3poYW5nL3Rla3Rvbi10ZXN0IFtdfSIsInBhdGhUb0RvY2tlckZpbGUiOiJEb2NrZXJmaWxlIn19LCJidWlsZENvbmZpZyI6eyJzdGVwcyI6W3siZW50cnlQb2ludCI6Im12biIsImFyZ3VtZW50cyI6WyJjbGVhbiIsImluc3RhbGwiLCItRHNraXBUZXN0cyJdLCJlbnZpcm9ubWVudCI6eyJjb250YWluZXIiOiJtYXZlbiIsImltYWdlIjoiZG9ja2VyLmlvL2xpYnJhcnkvbWF2ZW5Ac2hhMjU2OjcyOTIyYWJjOTVkMzhlMDJmNzUwYjM0ODAwMjM5ZGMwZTJjMjk4ZTc0YmZkZDk3MDAxODM2N2YwZDkyODFkNWMifSwiYW5ub3RhdGlvbnMiOm51bGx9LHsiZW50cnlQb2ludCI6Ii9rYW5pa28vZXhlY3V0b3IiLCJhcmd1bWVudHMiOlsiLS1kb2NrZXJmaWxlPSQocGFyYW1zLnBhdGhUb0RvY2tlckZpbGUpIiwiLS1kZXN0aW5hdGlvbj0kKHBhcmFtcy5pbWFnZVVybCk6JChwYXJhbXMuaW1hZ2VUYWcpIiwiLS1jb250ZXh0PSQod29ya3NwYWNlcy5zb3VyY2UucGF0aCkiLCItLWRpZ2VzdC1maWxlPSQocmVzdWx0cy5JTUFHRV9ESUdFU1QucGF0aCkiXSwiZW52aXJvbm1lbnQiOnsiY29udGFpbmVyIjoiYnVpbGQtYW5kLXB1c2giLCJpbWFnZSI6Imdjci5pby9rYW5pa28tcHJvamVjdC9leGVjdXRvckBzaGEyNTY6ZmNjY2QyYWI5ZjM4OTJlMzNmYzdmMmU5NTBjOGU0ZmM2NjVlN2E0YzY2ZjZhOWQ3MGIzMDBkN2EyMTAzNTkyZiJ9LCJhbm5vdGF0aW9ucyI6bnVsbH0seyJlbnRyeVBvaW50Ijoic2V0IC1lXG5lY2hvICQocGFyYW1zLklNQUdFKSB8IHRlZSAkKHJlc3VsdHMuSU1BR0VfVVJMLnBhdGgpICAgICAgICBcbiIsImFyZ3VtZW50cyI6bnVsbCwiZW52aXJvbm1lbnQiOnsiY29udGFpbmVyIjoid3JpdGUtdXJsIiwiaW1hZ2UiOiJkb2NrZXIuaW8vbGlicmFyeS9iYXNoQHNoYTI1NjpiM2FiZTQyNTU3MDY2MThjNTUwZThkYjVlYzA4NzUzMjgzMzNhMTRkYmY2NjNlNmYxZTJiNjg3NWY0NTUyMWU1In0sImFubm90YXRpb25zIjpudWxsfV19LCJtZXRhZGF0YSI6eyJidWlsZFN0YXJ0ZWRPbiI6IjIwMjItMDUtMTRUMDU6NDI6MTFaIiwiYnVpbGRGaW5pc2hlZE9uIjoiMjAyMi0wNS0xNFQwNTo0Mjo0MloiLCJjb21wbGV0ZW5lc3MiOnsicGFyYW1ldGVycyI6ZmFsc2UsImVudmlyb25tZW50IjpmYWxzZSwibWF0ZXJpYWxzIjpmYWxzZX0sInJlcHJvZHVjaWJsZSI6ZmFsc2V9LCJtYXRlcmlhbHMiOlt7InVyaSI6ImdpdCtodHRwczovL2dpdGh1Yi5jb20vYWRkb3poYW5nL3Rla3Rvbi10ZXN0LmdpdC5naXQiLCJkaWdlc3QiOnsic2hhMSI6IjQ2OTMxZmJjZjAzMDMyMGMzOTlmZjljMzA1MGQ2YjczMTI1OWFiZmMifX1dfX0=",
  "chains.tekton.dev/retries": "0",
  "chains.tekton.dev/signature-taskrun-5bf67bed-af59-4744-a6a7-487de359cbae": "eyJwYXlsb2FkVHlwZSI6ImFwcGxpY2F0aW9uL3ZuZC5pbi10b3RvK2pzb24iLCJwYXlsb2FkIjoiZXlKZmRIbHdaU0k2SW1oMGRIQnpPaTh2YVc0dGRHOTBieTVwYnk5VGRHRjBaVzFsYm5RdmRqQXVNU0lzSW5CeVpXUnBZMkYwWlZSNWNHVWlPaUpvZEhSd2N6b3ZMM05zYzJFdVpHVjJMM0J5YjNabGJtRnVZMlV2ZGpBdU1pSXNJbk4xWW1wbFkzUWlPbHQ3SW01aGJXVWlPaUpwYm1SbGVDNWtiMk5yWlhJdWFXOHZZV1JrYjNwb1lXNW5MM1JsYTNSdmJpMTBaWE4wSWl3aVpHbG5aWE4wSWpwN0luTm9ZVEkxTmlJNkltRmpZbVE1T1dZNFltVXdNRE5sT0dJM1pUYzNOalkxWldVd1pUTXpNbVU1Tm1SaE1tTmlOVFJpTXpJM01UazRZV0ppTkRZM01XTTRaR0l4WlRrNU9HRWlmWDFkTENKd2NtVmthV05oZEdVaU9uc2lZblZwYkdSbGNpSTZleUpwWkNJNkltaDBkSEJ6T2k4dmRHVnJkRzl1TG1SbGRpOWphR0ZwYm5NdmRqSWlmU3dpWW5WcGJHUlVlWEJsSWpvaWFIUjBjSE02THk5MFpXdDBiMjR1WkdWMkwyRjBkR1Z6ZEdGMGFXOXVjeTlqYUdGcGJuTkFkaklpTENKcGJuWnZZMkYwYVc5dUlqcDdJbU52Ym1acFoxTnZkWEpqWlNJNmUzMHNJbkJoY21GdFpYUmxjbk1pT25zaVEwaEJTVTVUTFVkSlZGOURUMDFOU1ZRaU9pSjdjM1J5YVc1bklEUTJPVE14Wm1KalpqQXpNRE15TUdNek9UbG1aamxqTXpBMU1HUTJZamN6TVRJMU9XRmlabU1nVzExOUlpd2lRMGhCU1U1VExVZEpWRjlWVWt3aU9pSjdjM1J5YVc1bklHaDBkSEJ6T2k4dloybDBhSFZpTG1OdmJTOWhaR1J2ZW1oaGJtY3ZkR1ZyZEc5dUxYUmxjM1F1WjJsMElGdGRmU0lzSWtsTlFVZEZJam9pZTNOMGNtbHVaeUJoWkdSdmVtaGhibWN2ZEdWcmRHOXVMWFJsYzNRZ1cxMTlJaXdpYVcxaFoyVlVZV2NpT2lKc1lYUmxjM1FpTENKcGJXRm5aVlZ5YkNJNkludHpkSEpwYm1jZ1lXUmtiM3BvWVc1bkwzUmxhM1J2YmkxMFpYTjBJRnRkZlNJc0luQmhkR2hVYjBSdlkydGxja1pwYkdVaU9pSkViMk5yWlhKbWFXeGxJbjE5TENKaWRXbHNaRU52Ym1acFp5STZleUp6ZEdWd2N5STZXM3NpWlc1MGNubFFiMmx1ZENJNkltMTJiaUlzSW1GeVozVnRaVzUwY3lJNld5SmpiR1ZoYmlJc0ltbHVjM1JoYkd3aUxDSXRSSE5yYVhCVVpYTjBjeUpkTENKbGJuWnBjbTl1YldWdWRDSTZleUpqYjI1MFlXbHVaWElpT2lKdFlYWmxiaUlzSW1sdFlXZGxJam9pWkc5amEyVnlMbWx2TDJ4cFluSmhjbmt2YldGMlpXNUFjMmhoTWpVMk9qY3lPVEl5WVdKak9UVmtNemhsTURKbU56VXdZak0wT0RBd01qTTVaR013WlRKak1qazRaVGMwWW1aa1pEazNNREF4T0RNMk4yWXdaRGt5T0RGa05XTWlmU3dpWVc1dWIzUmhkR2x2Ym5NaU9tNTFiR3g5TEhzaVpXNTBjbmxRYjJsdWRDSTZJaTlyWVc1cGEyOHZaWGhsWTNWMGIzSWlMQ0poY21kMWJXVnVkSE1pT2xzaUxTMWtiMk5yWlhKbWFXeGxQU1FvY0dGeVlXMXpMbkJoZEdoVWIwUnZZMnRsY2tacGJHVXBJaXdpTFMxa1pYTjBhVzVoZEdsdmJqMGtLSEJoY21GdGN5NXBiV0ZuWlZWeWJDazZKQ2h3WVhKaGJYTXVhVzFoWjJWVVlXY3BJaXdpTFMxamIyNTBaWGgwUFNRb2QyOXlhM053WVdObGN5NXpiM1Z5WTJVdWNHRjBhQ2tpTENJdExXUnBaMlZ6ZEMxbWFXeGxQU1FvY21WemRXeDBjeTVKVFVGSFJWOUVTVWRGVTFRdWNHRjBhQ2tpWFN3aVpXNTJhWEp2Ym0xbGJuUWlPbnNpWTI5dWRHRnBibVZ5SWpvaVluVnBiR1F0WVc1a0xYQjFjMmdpTENKcGJXRm5aU0k2SW1kamNpNXBieTlyWVc1cGEyOHRjSEp2YW1WamRDOWxlR1ZqZFhSdmNrQnphR0V5TlRZNlptTmpZMlF5WVdJNVpqTTRPVEpsTXpObVl6ZG1NbVU1TlRCak9HVTBabU0yTmpWbE4yRTBZelkyWmpaaE9XUTNNR0l6TURCa04yRXlNVEF6TlRreVppSjlMQ0poYm01dmRHRjBhVzl1Y3lJNmJuVnNiSDBzZXlKbGJuUnllVkJ2YVc1MElqb2ljMlYwSUMxbFhHNWxZMmh2SUNRb2NHRnlZVzF6TGtsTlFVZEZLU0I4SUhSbFpTQWtLSEpsYzNWc2RITXVTVTFCUjBWZlZWSk1MbkJoZEdncElDQWdJQ0FnSUNCY2JpSXNJbUZ5WjNWdFpXNTBjeUk2Ym5Wc2JDd2laVzUyYVhKdmJtMWxiblFpT25zaVkyOXVkR0ZwYm1WeUlqb2lkM0pwZEdVdGRYSnNJaXdpYVcxaFoyVWlPaUprYjJOclpYSXVhVzh2YkdsaWNtRnllUzlpWVhOb1FITm9ZVEkxTmpwaU0yRmlaVFF5TlRVM01EWTJNVGhqTlRVd1pUaGtZalZsWXpBNE56VXpNamd6TXpOaE1UUmtZbVkyTmpObE5tWXhaVEppTmpnM05XWTBOVFV5TVdVMUluMHNJbUZ1Ym05MFlYUnBiMjV6SWpwdWRXeHNmVjE5TENKdFpYUmhaR0YwWVNJNmV5SmlkV2xzWkZOMFlYSjBaV1JQYmlJNklqSXdNakl0TURVdE1UUlVNRFU2TkRJNk1URmFJaXdpWW5WcGJHUkdhVzVwYzJobFpFOXVJam9pTWpBeU1pMHdOUzB4TkZRd05UbzBNam8wTWxvaUxDSmpiMjF3YkdWMFpXNWxjM01pT25zaWNHRnlZVzFsZEdWeWN5STZabUZzYzJVc0ltVnVkbWx5YjI1dFpXNTBJanBtWVd4elpTd2liV0YwWlhKcFlXeHpJanBtWVd4elpYMHNJbkpsY0hKdlpIVmphV0pzWlNJNlptRnNjMlY5TENKdFlYUmxjbWxoYkhNaU9sdDdJblZ5YVNJNkltZHBkQ3RvZEhSd2N6b3ZMMmRwZEdoMVlpNWpiMjB2WVdSa2IzcG9ZVzVuTDNSbGEzUnZiaTEwWlhOMExtZHBkQzVuYVhRaUxDSmthV2RsYzNRaU9uc2ljMmhoTVNJNklqUTJPVE14Wm1KalpqQXpNRE15TUdNek9UbG1aamxqTXpBMU1HUTJZamN6TVRJMU9XRmlabU1pZlgxZGZYMD0iLCJzaWduYXR1cmVzIjpbeyJrZXlpZCI6IlNIQTI1NjppRlk2RVhnWWt5enlabzgxU29MS1FPRy9CdjZyZnorZFYrMkZxNXFjY2NBIiwic2lnIjoiTUVVQ0lRREZ4bzE0bzVldDFRVkJKUklQK3liSlhmQ2VFTjJaK3ZFNWROQlpMQmhvSWdJZ0hwL3B4VDF0SDZleHF0d2x2UStML0prazhXVFMzbTJhL1BWN3AzeEdrTkE9In1dfQ==",
  "kubectl.kubernetes.io/last-applied-configuration": "{\"apiVersion\":\"tekton.dev/v1beta1\",\"kind\":\"PipelineRun\",\"metadata\":{\"annotations\":{},\"generateName\":\"generic-pr-\",\"name\":\"generic-pipeline-run\",\"namespace\":\"tekton-pipelines\"},\"spec\":{\"params\":[{\"name\":\"git-revision\",\"value\":\"main\"},{\"name\":\"git-url\",\"value\":\"https://github.com/addozhang/tekton-test.git\"},{\"name\":\"imageUrl\",\"value\":\"addozhang/tekton-test\"},{\"name\":\"imageTag\",\"value\":\"latest\"}],\"pipelineRef\":{\"name\":\"build-pipeline\"},\"serviceAccountName\":\"tekton-build\",\"workspaces\":[{\"name\":\"git-source\",\"volumeClaimTemplate\":{\"spec\":{\"accessModes\":[\"ReadWriteOnce\"],\"resources\":{\"requests\":{\"storage\":\"1Gi\"}}}}},{\"name\":\"docker-config\",\"secret\":{\"secretName\":\"docker-config\"}}]}}\n",
  "pipeline.tekton.dev/affinity-assistant": "affinity-assistant-876633b0fe",
  "pipeline.tekton.dev/release": "422a468"
}
```

将 payload 的内容解码可以看到 TaskRun 的输入、输出以及 Task 本身使用的镜像等信息。

```
{
  "_type": "https://in-toto.io/Statement/v0.1",
  "predicateType": "https://slsa.dev/provenance/v0.2",
  "subject": [
    {
      "name": "index.docker.io/addozhang/tekton-test",
      "digest": {
        "sha256": "acbd99f8be003e8b7e77665ee0e332e96da2cb54b327198abb4671c8db1e998a"
      }
    }
  ],
  "predicate": {
    "builder": {
      "id": "https://tekton.dev/chains/v2"
    },
    "buildType": "https://tekton.dev/attestations/chains@v2",
    "invocation": {
      "configSource": {},
      "parameters": {
        "CHAINS-GIT_COMMIT": "{string 46931fbcf030320c399ff9c3050d6b731259abfc []}",
        "CHAINS-GIT_URL": "{string https://github.com/addozhang/tekton-test.git []}",
        "IMAGE": "{string addozhang/tekton-test []}",
        "imageTag": "latest",
        "imageUrl": "{string addozhang/tekton-test []}",
        "pathToDockerFile": "Dockerfile"
      }
    },
    "buildConfig": {
      "steps": [
        {
          "entryPoint": "mvn",
          "arguments": [
            "clean",
            "install",
            "-DskipTests"
          ],
          "environment": {
            "container": "maven",
            "image": "docker.io/library/maven@sha256:72922abc95d38e02f750b34800239dc0e2c298e74bfdd970018367f0d9281d5c"
          },
          "annotations": null
        },
        {
          "entryPoint": "/kaniko/executor",
          "arguments": [
            "--dockerfile=$(params.pathToDockerFile)",
            "--destination=$(params.imageUrl):$(params.imageTag)",
            "--context=$(workspaces.source.path)",
            "--digest-file=$(results.IMAGE_DIGEST.path)"
          ],
          "environment": {
            "container": "build-and-push",
            "image": "gcr.io/kaniko-project/executor@sha256:fcccd2ab9f3892e33fc7f2e950c8e4fc665e7a4c66f6a9d70b300d7a2103592f"
          },
          "annotations": null
        },
        {
          "entryPoint": "set -e\necho $(params.IMAGE) | tee $(results.IMAGE_URL.path)        \n",
          "arguments": null,
          "environment": {
            "container": "write-url",
            "image": "docker.io/library/bash@sha256:b3abe4255706618c550e8db5ec0875328333a14dbf663e6f1e2b6875f45521e5"
          },
          "annotations": null
        }
      ]
    },
    "metadata": {
      "buildStartedOn": "2022-05-14T05:42:11Z",
      "buildFinishedOn": "2022-05-14T05:42:42Z",
      "completeness": {
        "parameters": false,
        "environment": false,
        "materials": false
      },
      "reproducible": false
    },
    "materials": [
      {
        "uri": "git+https://github.com/addozhang/tekton-test.git.git",
        "digest": {
          "sha1": "46931fbcf030320c399ff9c3050d6b731259abfc"
        }
      }
    ]
  }
}
```