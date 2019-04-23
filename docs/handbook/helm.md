# Helm

## 1.Guide

1. Install helm From the Binary [Releases](https://helm.sh/docs/using_helm/#from-the-binary-releases)
2. Install tiller (k8s集群内) [Installation](https://helm.sh/docs/using_helm/#easy-in-cluster-installation)
3. Using SSL Between Helm and [Tiller](https://helm.sh/docs/using_helm/#using-ssl-between-helm-and-tiller)
4. Service account with cluster-admin [role](https://helm.sh/docs/using_helm/#example-service-account-with-cluster-admin-role)

## 2.Ansible install

Helm [variables](https://helm.sh/docs/using_helm/#environment-variables)

Tiller [RBAC](https://helm.sh/docs/using_helm/#tiller-and-role-based-access-control)

Helm [RBAC](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control)

```yaml
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

> 在helm客户端和tiller服务器间建立安全的SSL/TLS认证机制；
> tiller服务器和helm客户端都是使用同一CA签发的`client cert`
>
> Steps：
>
> 1. 下载最新release的helm客户端到/etc/ansible/bin目录下，再由它自动推送到deploy的{{ bin_dir }}目录下
> 2. 由集群CA签发helm客户端证书和私钥
> 3. 由集群CA签发tiller服务端证书和私钥
> 4. 创建tiller专用的RBAC配置，只允许helm在指定的namespace查看和安装应用
> 5. 安全安装tiller到集群，tiller服务启用tls验证
> 6. 配置helm客户端使用tls方式与tiller服务端通讯

在delpoy节点运行:
1.修改默认helm参数 vi  /etc/ansible/roles/helm/defaults/main.yml

- 下载二进制helm
- tiller拉取地址

2.执行安装 ansible-playbook /etc/ansible/roles/helm/helm.yml

```shell
helm init \
        --tiller-tls \
        --tiller-tls-verify \
        --tiller-tls-cert {{ ca_dir }}/{{ tiller_cert_cn }}.pem \
        --tiller-tls-key {{ ca_dir }}/{{ tiller_cert_cn }}-key.pem \
        --tls-ca-cert {{ ca_dir }}/ca.pem \
        --service-account {{ tiller_sa }} \
        --tiller-namespace {{ helm_namespace }} \
        --tiller-image {{ tiller_image }} \
        --stable-repo-url {{ repo_url }}"
```



## 3. Command

[ADVANCED USAGE(高级用法)](https://helm.sh/docs/using_helm/#advanced-usage)

### tiller

```shell
# auto update
$ helm init --upgrade 
# manual upgrades
$ kubectl --namespace=kube-system set image deployments/tiller-deploy tiller=gcr.io/kubernetes-helm/tiller:v2.12.3

# Delete，Tiller将其数据存储在Kubernetes ConfigMaps中，因此您可以安全地删除并重新安装
$ kubectl delete deployment tiller-deploy --namespace kube-system
$ helm reset

# re-installed
$ helm init

# upgrade chart repo
$ helm repo update 
$ helm repo list
$ helm repo add dev https://example.com/dev-charts

# ADVANCED USAGE e.g
$ helm init --node-selectors "beta.kubernetes.io/os"="linux"

```

### helm

```shell
# search
$ helm search

# description 
$ helm inspect stable/mariadb

# customize the chart
$ helm inspect values stable/mariadb
$ cat << EOF > config.yaml
mariadbUser: user0
mariadbDatabase: user0db
EOF
$ helm install -f config.yaml stable/mariadb

# install 
$ helm install --name mem1 stable/memcached

# delete
$ helm delete --dry-run
$ helm delete happy-panda

# list
$ helm list --all

# upgrad or rollback 
$ helm upgrade -f panda.yaml happy-panda stable/mariadb
$ helm rollback [RELEASE] [REVISION]

# hisory
$ helm hisory 

# status 
$ helm status $HELM_NAME 
```

### Plugin

```shell
# INSTALLING A PLUGIN
$ helm plugin install https://github.com/technosophos/helm-template
$ helm plugin install http://domain/path/to/plugin.tar.gz
```



## 4.Intro

### 核心术语

> Chart：一个helm程序包
>
> Repository：Charts仓库，https/http服务器
>
> Release：赋值后的Chart部署于目标集群上的一个实例
>
> Chart --> Config --> Release
>
> chart通过配置文件(config)提取属性赋值后生成对应的实例(Release)

### 程序架构

> Helm: 客户端，管理本地Chart仓库，管理Chart，与Tiller服务端交互，
>
> ​	发送Chart，实例的安装，查询，卸载等操作
>
> Tiller：服务端，接受helm发来的Charts与config，合并成release，连接API server
>
> Chart：类属性 -->赋值后--> release

##### 工作流

```
kubernetes-cluster --> API server --> Tiller --> helm
```

Helm 支持两种方式管理依赖的方式：

- 直接把依赖的 package 放在 `charts/` 目录中
- 使用 `requirements.yaml` 并用 `helm dep up foochart` 来自动下载依赖的 packages

Helm 使用 [Chart](https://github.com/kubernetes/charts) 来管理 Kubernetes manifest 文件。每个 chart 都至少包括

- 应用的基本信息 `Chart.yaml`
- 一个或多个 Kubernetes manifest 文件模版（放置于 templates / 目录中），可以包括 Pod、Deployment、Service 等各种 Kubernetes 资源

### Helm Repository

官方 repository:

- <https://hub.helm.sh/>
- <https://github.com/kubernetes/charts>

第三方 repository:

- <https://github.com/coreos/prometheus-operator/tree/master/helm>
- <https://github.com/deis/charts>
- <https://github.com/bitnami/charts>
- <https://github.com/att-comdev/openstack-helm>
- <https://github.com/sapcc/openstack-helm>
- <https://github.com/helm/charts>
- <https://github.com/jackzampolin/tick-charts>

### 常用 Helm 插件

1. [helm-tiller](https://github.com/adamreese/helm-tiller) - Additional commands to work with Tiller

2. [Technosophos's Helm Plugins](https://github.com/technosophos/helm-plugins) - Plugins for GitHub, Keybase, and GPG

3. [helm-template](https://github.com/technosophos/helm-template) - Debug/render templates client-side

4. [Helm Value Store](https://github.com/skuid/helm-value-store) - Plugin for working with Helm deployment values

5. ### [Drone.io Helm Plugin](http://plugins.drone.io/ipedrazas/drone-helm/) - Run Helm inside of the Drone CI/CD system