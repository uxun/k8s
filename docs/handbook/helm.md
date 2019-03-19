# Helm

## 1.Guide

#### From the Binary [Releases](https://helm.sh/docs/using_helm/#from-the-binary-releases)
#### Easy In-Cluster [Installation](https://helm.sh/docs/using_helm/#easy-in-cluster-installation)
#### Using SSL Between Helm and [Tiller](https://helm.sh/docs/using_helm/#using-ssl-between-helm-and-tiller)
#### Service account with cluster-admin [role](https://helm.sh/docs/using_helm/#example-service-account-with-cluster-admin-role)

## 2.Ansible install

> 在helm客户端和tiller服务器间建立安全的SSL/TLS认证机制；
> tiller服务器和helm客户端都是使用同一CA签发的`client cert`，在delpoy节点运行:
> 修改默认helm参数 vi  /etc/ansible/roles/helm/defaults/main.yml
> 执行安装 ansible-playbook /etc/ansible/roles/helm/helm.yml

```shell
roles/helm
├── defaults
│   └── main.yml
├── helm.yml
├── tasks
│   └── main.yml
└── templates
    ├── helm-csr.json.j2
    ├── helm-rbac.yaml.j2
    ├── strict-helm-rbac.yaml.j2
    └── tiller-csr.json.j2
```

### Steps

1. 下载最新release的helm客户端到/etc/ansible/bin目录下，再由它自动推送到deploy的{{ bin_dir }}目录下
2. 由集群CA签发helm客户端证书和私钥
3. 由集群CA签发tiller服务端证书和私钥
4. 创建tiller专用的RBAC配置，只允许helm在指定的namespace查看和安装应用
5. 安全安装tiller到集群，tiller服务启用tls验证
6. 配置helm客户端使用tls方式与tiller服务端通讯
