# Helm

## 1.Guide

1. Install helm From the Binary [Releases](https://helm.sh/docs/using_helm/#from-the-binary-releases)
2. Install tiller (k8s集群内) [Installation](https://helm.sh/docs/using_helm/#easy-in-cluster-installation)
3. Using SSL Between Helm and [Tiller](https://helm.sh/docs/using_helm/#using-ssl-between-helm-and-tiller)
4. Service account with cluster-admin [role](https://helm.sh/docs/using_helm/#example-service-account-with-cluster-admin-role)

## 2.Ansible install

在delpoy节点运行:

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

#### 1.修改默认helm参数 vi  /etc/ansible/roles/helm/defaults/main.yml

- 下载二进制helm
- tiller拉取地址

#### 2.执行安装 ansible-playbook /etc/ansible/roles/helm/helm.yml

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

Helm [variables](https://helm.sh/docs/using_helm/#environment-variables)

Tiller [RBAC](https://helm.sh/docs/using_helm/#tiller-and-role-based-access-control)

Helm [RBAC](https://helm.sh/docs/using_helm/#helm-and-role-based-access-control)

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



## 3. Command

[ADVANCED USAGE(高级用法)](https://helm.sh/docs/using_helm/#advanced-usage)

## tiller

>  repository:
>
> - <https://hub.helm.sh/>
> - <https://github.com/kubernetes/charts>
> - <https://github.com/bitnami/charts
>
> stable repo modify
>
> - ~/.helm/repository/repositories.yaml

```shell
# auto update 
$ helm init --upgrade 
# 设置历史记录
$ helm init --history-max 200

# manual upgrades
$ kubectl --namespace=kube-system set image deployments/tiller-deploy tiller=gcr.io/kubernetes-helm/tiller:v2.12.3

# reinstall
# Delete，Tiller将其数据存储在Kubernetes ConfigMaps中，因此您可以安全地删除并重新安装
$ kubectl delete deployment tiller-deploy --namespace kube-system
$ helm reset

# upgrade chart repo
$ helm repo update 
$ helm repo list
$ helm repo add bitnami https://charts.bitnami.com/bitnami
# stable repo 
$ helm init -c --stable-repo-url https://kubernetes.oss-cn-hangzhou.aliyuncs.com/chart

# ADVANCED USAGE e.g
$ helm init --node-selectors "beta.kubernetes.io/os"="linux" 
```

## helm

> 1. `helm inspect` 查看Chart的信息
> 2. `helm install` 指定Chart 部署一个Release到Kubernetes
> 3. `helm package` 打包Chart(压缩包文件)
> 4. `helm list` 列出已经部署的Release
> 5. `helm delete [RELEASE]` 删除一个Release. 并没有物理删除， 出于审计需要，历史可查。
> 6. `helm status [RELEASE]` 查看指定的Release信息，即使使用`helm delete`命令删除的Release.
> 7. `helm upgrade` 升级某个Release
> 8. `helm rollback [RELEASE] [REVISION]` 回滚Release到指定发布序列
> 9. `helm get values [RELEASE]` 查看Release的配置文件值

#### 1.`helm inspect` 查看Chart的信息

```sh e
# description 
$ helm inspect stable/prometheus-operator

# customize the chart
$ helm inspect values stable/mariadb
$ cat << EOF > config.yaml
mariadbUser: user0
mariadbDatabase: user0db
EOF

# 修改配置
$ helm inspect values stable/prometheus-operator > prom.yaml
$ helm install --name monitor --namespace monitoring -f prom.yaml stable/prometheus-operator
# 查看变量是否生效
$ helm get values monitor
```

#### 2.`helm install` 指定Chart 部署一个Release到Kubernetes

> set的优先级最高，并将值合并进-f指定的配置文件中

```shell
$ helm install stable/mariadb --set persistence.enabled=false --name mydb

# 两种方式部署
1.-f(--values):使用YAML文件，文件内容要覆盖Chart的配置值。
2.--set: 指定具体要覆盖的默认配置值。参考https://helm.sh/docs/using_helm/#the-format-and-limitations-of-set
	--set name1=val1,name2=val2
	
# 打印清单文件内容，不执行部署
$ helm install . --dry-run --debug --set image.tag=latest
```

#### 3.other

```shell
# list
$ helm list --all

# delete
$ helm del grafana --purge 

# download helm charts
$ helm fetch --untar stable/prometheus
$ helm fetch --untar stable/grafana

# helm upgrade
$ helm upgrade -f mydb.yaml mydb stable/mariadb

# status 
$ helm status $HELM_NAME 

# hisory
$ helm history grafana
REVISION	UPDATED                 	STATUS  	CHART         	DESCRIPTION
1       	Fri Apr 26 17:07:08 2019	DEPLOYED	grafana-1.16.0	Install complete
# helm rollback [RELEASE] [REVISION]
$ helm rollback grafana 1
```

### Plugin

> 1. [helm-template](https://github.com/technosophos/helm-template) - Debug/render templates client-side
> 2. [Helm Value Store](https://github.com/skuid/helm-value-store) - Plugin for working with Helm deployment values
> 3. [Drone.io Helm Plugin](http://plugins.drone.io/ipedrazas/drone-helm/) - Run Helm inside of the Drone CI/CD system

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

工作流

> release 是 chart 的运行实例，代表了一个正在运行的应用。当 chart 被安装到 Kubernetes 集群，就生成一个 release。chart 能够多次安装到同一个集群，每次安装都是一个 release。
>
> Helm: 客户端，管理本地Chart仓库，管理Chart，与Tiller服务端交互，发送Chart，实例的安装，查询，卸载等操作
>
> Tiller：服务端，接受helm发来的Charts与config，合并成release，连接API server
>
> [Chart](https://github.com/helm/helm/blob/master/docs/charts.md)：类属性 -->赋值后--> release

Helm 支持两种方式管理依赖的方式：

- 直接把依赖的 package 放在 `charts/` 目录中
- 使用 `requirements.yaml` 并用 `helm dep up foochart` 来自动下载依赖的 packages

Helm 使用 [Chart](https://github.com/kubernetes/charts) 来管理 Kubernetes manifest 文件。每个 chart 都至少包括

- 应用的基本信息 `Chart.yaml`
- 一个或多个 Kubernetes manifest 文件模版（放置于 templates / 目录中），可以包括 Pod、Deployment、Service 等各种 Kubernetes 资源



## Helm UI

[Kubeapps](https://github.com/kubeapps/kubeapps) 提供了一个开源的 Helm UI 界面，方便以图形界面的形式管理 Helm 应用。

```sh
curl -s https://api.github.com/repos/kubeapps/kubeapps/releases/latest | grep -i $(uname -s) | grep browser_download_url | cut -d '"' -f 4 | wget -i -
sudo mv kubeapps-$(uname -s| tr '[:upper:]' '[:lower:]')-amd64 /usr/local/bin/kubeapps
sudo chmod +x /usr/local/bin/kubeapps

kubeapps up
kubeapps dashboard
```

更多使用方法请参考 [Kubeapps 官方网站](https://kubeapps.com/)。

