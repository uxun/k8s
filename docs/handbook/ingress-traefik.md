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



## ingress-traefik TLS

### 1.prepare tls 

> Let's Encrypt 

使用自签证书 (tls.key/tls.crt)，

注意CN 配置为 ingress 的域名：

``` bash
$ openssl req -x509 -nodes -days 365 -newkey rsa:2048 -keyout tls.key -out tls.crt -subj "/CN=traefik-ui.test.com"

# kube-system namespaces create (secret: traefik-ui-tls-cert),traefik-controller Convenient for mounting certificates
$ kubectl -n kube-system create secret tls traefik-tls-cert --key=tls.key --cert=tls.crt
```

### 2.create traefik configmap

``` bash
cat <<EOF> traefik-tls-configmap.yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: kube-system
  name: traefik-tls-conf
data:
  traefik.toml: |
    # insecureSkipVerify = true，backend(不卸载tls)443，e.g dashboard ingress rules
    insecureSkipVerify = true
    # traefik.toml
    logLevel = "DEBUG"
    defaultEntryPoints = ["http","https"]
    [entryPoints]
      [entryPoints.http]
      address = ":80"
      ## http skip https 
      [entryPoints.http.redirect]
      #entryPoint = "https"
      [entryPoints.https]
      address = ":443"
      ## trustedIPs --> X-Frowarded-*,default(all)
      #[entryPoints.http.forwardedHeaders]
      #trustedIPs = ["10.1.0.0/16", "172.20.0.0/16", "192.168.1.x"]
      [entryPoints.https.tls]
      [[entryPoints.https.tls.certificates]]
      CertFile = "/ssl/tls.crt"
      KeyFile = "/ssl/tls.key"
EOF
```

### 3.apply trafik configmap

```
kubectl apply -f /etc/ansible/manifests/ingress/tls/traefik-tls-configmap.yaml
kubectl apply -f /etc/ansible/manifests/ingress/tls/traefik-tls-deployment.yaml
```

### 4.https traefik-ui

``` yaml
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
    targetPort: 8080
---
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: traefik-web-ui
  namespace: kube-system
  # add
  annotations:
    kubernetes.io/ingress.class: traefik
spec:
  rules:
  - host: traefik-ui.test.com
    http:
      paths:
      - path: /
        backend:
          serviceName: traefik-web-ui
          servicePort: web
  # add 
  tls:
   - secretName: traefik-tls-cert
   
```

### 5.validation

traefik-ingress svc

``` bash
kubectl get svc --all-namespaces -o wide
kube-system   traefik-ingress-service   NodePort    10.68.124.202   <none>        80:20080/TCP,443:20443/TCP,8080:20081/TCP   7h46m   k8s-app=traefik-ingress-lb

curl -k https://traefik-ui.test.com:20443
<a href="/dashboard/">Found</a>.
```

### 6.dashboard ingress

> 注意annotations 配置了 http 跳转 https 功能
>
> 注意后端服务是443端口
>
> 集群的dashboard 已安装

traefik-tls-configmap.yaml

`traefik.toml enable insecureSkipVerify = true`

```shell
# kubectl get svc -n kube-system | grep dashboard
kubernetes-dashboard      NodePort    10.68.30.99     <none>        443:24384/TCP                               14d
```
kubectl apply -f /etc/ansible/manifests/ingress/tls/ingress-dashboard.yaml

```yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name:  kubernetes-dashboard
  namespace: kube-system
  annotations:
    traefik.ingress.kubernetes.io/redirect-entry-point: https
spec:
  rules:
  # changeful
  - host: dashboard.YOU.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kubernetes-dashboard
          servicePort: 443
```
### REFERENCE

- [Add a TLS Certificate to the Ingress](https://docs.traefik.io/user-guide/kubernetes/#add-a-tls-certificate-to-the-ingress)
