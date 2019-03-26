# Ingress-nginx

> 1.What is Ingress? 
>
> ​	进入集群的请求提供路由规则的集合
>
> 2.一个公网ip提供许多服务的访问，客户端向ingress 发送http请求，ingress 根据主机名和路径决定请求转发的到服务
>
> 3.function:实现网络栈的 cookie的会话亲和性(session affinity)

#### Workflow

> client 查找 kube.example.com,DNS 返回 Ingess 控制器的ip ，client 向控制器发送HTTP请求，ingress控制器通过头部指定的域名尝试访问该服务，通过关联该服务的Endpoint对象查看pod ip，并将客户端请求转发给其中一个pod
>

```
client -> Ingress controller -> ingress -> service -> endpoint -> pod
```

## Ingress-controller

### [Additional controllers](https://kubernetes.io/docs/concepts/services-networking/ingress-controllers/#additional-controllers)

>方案选择[Nginx-ingress-controller](https://kubernetes.github.io/ingress-nginx/)，[traefix(微服务而生)](https://github.com/containous/traefik)， [Envoy(微服务)](https://www.envoyproxy.io/)，
>
> 1.NGINX Ingress Controller for Kubernetes [official](https://kubernetes.github.io/ingress-nginx/deploy/#installation-guide)
>
> 2.traefix (微服务而生)

### 1.Nginx-ingress-controller

Service(分类) - Ingress(识别) - Ingress controler(注入)

> 两种部署方式：
>
> 1.Deployment
>
> ​	ingress-service 导入外部流量
>
> 2.DaemonSet
>
> ​	共享宿主机网络

### Deployment

Bare-metal install Ingress-controller [reference](https://kubernetes.github.io/ingress-nginx/deploy/#bare-metal)

> NodePort类型的服务默认执行[源地址转换](https://kubernetes.io/docs/tutorials/services/source-ip/#source-ip-for-services-with-type-nodeport)。这意味着HTTP请求的源IP始终**是**从NGINX的角度接收请求的Kubernetes节点的IP地址
>
> NodePort类型的服务保留源IP配置方式：
>
> ​	service.spec.externalTrafficPolicy: Local [reference](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/#over-a-nodeport-service)

```shell
# ingress-controller
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/mandatory.yaml

# ingress-service
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/master/deploy/provider/baremetal/service-nodeport.yaml

# modify
# service-nodeport.yaml 
# service.spec.ports.nodePort
```

#### Test

```shell
kubectl exec -it -n ingress-nginx nginx-ingress-controller-58475cf7d6-9vsxl -- cat nginx.conf | grep "YOU.COM"
```



### DaemonSet [reference](https://kubernetes.github.io/ingress-nginx/deploy/baremetal/#via-the-host-network)

配置ingress-nginx Pod以使用它们运行的主机网络而不是专用网络命名空间。这种方法的好处是NGINX Ingress控制器可以将端口80和443直接绑定到Kubernetes节点的网络接口，而无需NodePort服务强加的额外网络转换。

>  Modify：
>
>  1. kind: DaemonSet
>  2. Delete replicas  
>  3. spec.hostNetwork: ture
>
>  如果`ingress-nginx` 目标群集中存在服务，则将其删除

`$ kubectl apply -f https://raw.githubusercontent.com/uxun/kube/master/manifests/ingress/ingress-nginx/daemonset/mandatory-daemonset.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment # modify DaemonSet
metadata:
  name: nginx-ingress-controller
  namespace: ingress-nginx
  labels:
    app.kubernetes.io/name: ingress-nginx
    app.kubernetes.io/part-of: ingress-nginx
spec:
  #replicas: 1  # delete
  selector:
    matchLabels:
      app.kubernetes.io/name: ingress-nginx
      app.kubernetes.io/part-of: ingress-nginx
  template:
    metadata:
      labels:
        app.kubernetes.io/name: ingress-nginx
        app.kubernetes.io/part-of: ingress-nginx
      annotations:
        prometheus.io/port: "10254"
        prometheus.io/scrape: "true"
    spec:
      # add hostNetwork
      hostNetwork: true
      serviceAccountName: nginx-ingress-serviceaccount
      containers:
        - name: nginx-ingress-controller
          #image: quay.io/kubernetes-ingress-controller/nginx-ingress-controller:0.23.0
          image: uxun/nginx-ingress-controller:0.23.0
          args:
          ...
```

###### 共享宿主机: 考虑的问题 1000个 节点呢？

## Ingress Example

```yaml
# kube-ingress.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kube
spec:
  rules:
  - host: kubeA.example.com 
    http:
      paths:
      - path: /        #请求kubeA.example.com,转发到kube-nodeport的服务上
        backend:
          serviceName: kube-nodeport 
          servicePort: 80
	  - path: /kube    #请求kubeA.example.com/kube,转发到kube-xxxx的服务上
	    backend:
	      serviceName: kube-xxxx
	      servicePort: 80
  - host: kubeB.example.com  
    http: 
      paths:
      - path: /
	    backend:
	      serviceName: kubeB
	      servicePort: 80
# validation
$ kubectl get ingresses
```

### ingress TLS (HTTPS)

```yaml
# kube-ingress-tls.yaml
apiVersion: extensions/v1beta1
kind: Ingress
metadata:
  name: kube
spec:
  tls:              #这个属性下包含了所有的TLS配置
  - hosts:
    - kube.example.com
    sercretName: tls-secret        #从tls-secret中获得私钥和证书
  rules:
  - host: kube.example.com
    http:
      paths:
      - path: /
        backend:
          serviceName: kube-nodeport
          servicePort: 80
```


