如果需要对客户端做细粒度的权限控制，可以通过以下方式生成 config 文件。

```
curl -k https://<YOUR_API_SERVER_PUBLIC_IP>:6443
```

创建 ~/.kube/config 文件，并修改文件内容

```
apiVersion: v1
clusters:
- cluster:
    # needed if you get error "Unable to connect to the server: x509: certificate signed by unknown authority"
    insecure-skip-tls-verify: true
    server: https://YOUR_API_SERVER_PUBLIC_IP:6443
  name: kubernetes
contexts:
- context:
    cluster: kubernetes
    user: kubernetes-admin
  name: kubernetes-admin@kubernetes
current-context: kubernetes-admin@kubernetes
kind: Config
preferences: {}
users:
- name: kubernetes-admin
  user:
    client-certificate-data: LS0tLS1CRUdJTi... (base64 /etc/kubernetes/ssl/node-node1.pem)
    client-key-data: LS0tLS1CRUdJTiBS.. (base64 /etc/kubernetes/ssl/node-node1-key.pem)
```

其中 client-certificate-data 来源于：

```
cat /etc/kubernetes/ssl/node-node1.pem | base64 -w 0
```

client-key-data 来源于以下命令的输出：

```
cat /etc/kubernetes/ssl/node-node1-key.pem | base64 -w 0
```

验证证书：

```
$ kubectl get nodes
NAME      STATUS    AGE       VERSION
my-kube   Ready     2h        v1.6.7+coreos.0
```