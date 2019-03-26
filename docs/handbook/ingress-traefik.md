# Ingress-traefik

## Kubernetes Ingress Controller[¶](https://docs.traefik.io/user-guide/kubernetes/#kubernetes-ingress-controller)

> Ingress是授权入站连接到达集群服务的规则集合
>
> Ingress controller 通过apiserver监听ingress和service的变化，并根据规则配置负载均衡并提供访问入口

```
   internet
       |
   [ Ingress ]
   --|-----|--
   [ Services ]
```

### official-example[¶](https://github.com/containous/traefik/tree/v1.7/examples/k8s)

### Annotations[¶](https://docs.traefik.io/configuration/backends/kubernetes/#annotations)

## 1.1 Install (damonset)

traefik-ingress-controller 

```shell
# kubectl create -f /etc/ansible/manifests/ingress/ingress-traefix/daemonset/traefik-ds.yaml
```

节点亲和只调度到特定需要服务的节点 [reference](https://kubernetes.io/docs/concepts/configuration/assign-pod-node/#affinity-and-anti-affinity)

查看node label

```shell
kubectl get nodes --show-labels
NAME           STATUS                     ROLES    AGE   VERSION   LABELS
192.168.0.25   Ready,SchedulingDisabled   master   14d   v1.13.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=192.168.0.25,kubernetes.io/role=master
192.168.0.26   Ready,SchedulingDisabled   master   14d   v1.13.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=192.168.0.26,kubernetes.io/role=master
192.168.0.27   Ready                      node     14d   v1.13.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=192.168.0.27,kubernetes.io/role=node
192.168.0.29   Ready                      node     14d   v1.13.3   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=192.168.0.29,kubernetes.io/role=node
```

kubectl explain DaemonSet.spec.template.spec.affinity.nodeAffinity

```yaml
kind: DaemonSet
apiVersion: extensions/v1beta1
metadata:
  name: traefik-ingress-controller
  namespace: kube-system
  labels:
    k8s-app: traefik-ingress-lb
spec:
  template:
    metadata:
      labels:
        k8s-app: traefik-ingress-lb
        name: traefik-ingress-lb
    spec:
      serviceAccountName: traefik-ingress-controller
      terminationGracePeriodSeconds: 60
      # add
      affinity:
        nodeAffinity:
          requiredDuringSchedulingIgnoredDuringExecution:
            nodeSelectorTerms:
            - matchExpressions:
              - key: kubernetes.io/hostname
                operator: In
                values:
                - 192.168.0.27
                - 192.168.0.29
                ...
```

#### validation

```shell
$ kubectl get svc -n kube-system traefik-ingress-service
NAME                      TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)           AGE
traefik-ingress-service   ClusterIP   10.68.186.217   <none>        80/TCP,8080/TCP   24m

$ kubectl run --generator=run-pod/v1 hello-world --image=nginx --expose --port=80
$ kubectl create -f ingress-hello-world.yaml
```

ingress-hello-world.yaml

```yaml
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-world
spec:
  rules:
  - host: hello.com
    http:
      paths:
      - path: /
        backend:
          serviceName: hello-world
          servicePort: 80
---
```

vim /etc/hosts

```
192.168.0.27 hello.com
```

> $ curl hello.com
>
> <head>
> <title>Welcome to nginx!</title>



## 1.2 Install (Deployment)

install

```shell
# kubectl create -f /etc/ansible/manifests/ingress/ingress-traefix/deployment/traefik-deployment.yaml
```

Notice

```yaml
kind: Service
apiVersion: v1
metadata:
  name: traefik-ingress-service
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
    - protocol: TCP
      # traefik ingress-controller service port
      port: 80
      name: web
      # custom default ports (20000~40000)
      nodePort: 20080
    - protocol: TCP
      # web manager port
      port: 8080
      nodePort: 20081
      name: admin
  type: NodePort
```

kubectl get po --all-namespaces -o wide

```
kube-system   traefik-ingress-controller-8c8b85bbc-9dpch   1/1     Running   0          5m9s   172.20.3.40    192.168.0.27   <none>           <none>
```

curl hell.com:20080

```
[root@k8s01 traefix]# curl hello.com:20080
<!DOCTYPE html>
<html>
<head>
<title>Welcome to nginx!</title>
```



## Install Traefix UI

> 1.dwonload [ui.yaml](https://github.com/containous/traefik/blob/v1.7/examples/k8s/ui.yaml)
>
> ​	modify: targetPort (Collision avoidance port)

Traffic-web-ui.yaml

```yaml
---
apiVersion: v1
kind: Service
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  selector:
    k8s-app: traefik-ingress-lb
  ports:
  - name: web
    port: 80
    # Notice 
    targetPort: 8080
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
spec:
  rules:
  # Notice:you must be changed e.g: traefix-ui.test.com
  - host: traefik.YOU.com
    http:
      paths:
      - path: /
        backend:
          serviceName: traefik-web-ui
          servicePort: web
```

Vim /etc/hosts

```
192.168.0.29 traefix-ui.test.com
```

curl http://traefik-ui.test.com:8088  or curl traefik-ui.test.com

```
curl http://traefik-ui.test.com:8088
<a href="/dashboard/">Found</a>.
```

# 使用 traefik 配置 https ingress

本文档基于 traefik 配置 https ingress 规则，请先阅读[配置基本 ingress](ingress.md)。与基本 ingress-controller 相比，需要额外配置 https tls 证书，主要步骤如下：

## 1.准备 tls 证书

可以使用Let's Encrypt签发的免费证书，这里为了测试方便使用自签证书 (tls.key/tls.crt)，注意CN 配置为 ingress 的域名：

``` bash
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=hello.test.com"
```

## 2.在 kube-system 命名空间创建 secret: traefik-cert，以便后面 traefik-controller 挂载该证书

``` bash
$ kubectl -n kube-system create secret tls traefik-cert --key=tls.key --cert=tls.crt
```

## 3.创建 traefik-controller，增加 traefik.toml 配置文件及https 端口暴露等，详见该 yaml 文件

``` bash
$ kubectl apply -f /etc/ansible/manifests/ingress/traefik/tls/traefik-controller.yaml
```

## 4.创建 https ingress 例子

``` bash
# 创建示例应用
$ kubectl run test-hello --image=nginx --port=80 --expose
# hello-tls-ingress 示例
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: hello-tls-ingress
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: hello.test.com
    http:
      paths:
      - backend:
          serviceName: test-hello
          servicePort: 80
  tls:
  - secretName: traefik-cert
# 创建https ingress
$ kubectl apply -f /etc/ansible/manifests/ingress/traefik/tls/hello-tls.ing.yaml
# 注意根据hello示例，需要在default命名空间创建对应的secret: traefik-cert
$ kubectl create secret tls traefik-cert --key=tls.key --cert=tls.crt
```

## 5.验证 https 访问

验证 traefik-ingress svc

``` bash
$ kubectl get svc -n kube-system traefik-ingress-service 
NAME                      TYPE       CLUSTER-IP      EXTERNAL-IP   PORT(S)                                     AGE
traefik-ingress-service   NodePort   10.68.250.253   <none>        80:23456/TCP,443:23457/TCP,8080:35941/TCP   66m
```

可以看到项目默认使用nodePort 23456暴露traefik 80端口，nodePort 23457暴露 traefik 443端口，因此在客户端 hosts 增加记录 `$Node_IP hello.test.com`之后，可以在浏览器验证访问如下：

``` bash
https://hello.test.com:23457
```

如果你已经配置了[转发 ingress nodePort](../op/loadballance_ingress_nodeport.md)，那么增加对应 hosts记录后，可以验证访问 `https://hello.test.com`

## 配置 dashboard ingress

前提1：k8s 集群的dashboard 已安装

```
$ kubectl get svc -n kube-system | grep dashboard
kubernetes-dashboard      NodePort    10.68.211.168   <none>        443:39308/TCP	3d11h
```
前提2：`/etc/ansible/manifests/ingress/traefik/tls/traefik-controller.yaml`的配置文件`traefik.toml`开启了`insecureSkipVerify = true`

配置 dashboard ingress：`kubectl apply -f /etc/ansible/manifests/ingress/traefik/tls/k8s-dashboard.ing.yaml` 内容如下：

```
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name:  kubernetes-dashboard
  namespace: kube-system
  annotations:
    traefik.ingress.kubernetes.io/redirect-entry-point: https
spec:
  rules:
  - host: dashboard.test.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kubernetes-dashboard
          servicePort: 443
```
- 注意annotations 配置了 http 跳转 https 功能
- 注意后端服务是443端口

## 参考

- [Add a TLS Certificate to the Ingress](https://docs.traefik.io/user-guide/kubernetes/#add-a-tls-certificate-to-the-ingress)
