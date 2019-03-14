# Add kube-node 

## 1.Steps:

1. NEW_NODE chrony  [optional]
2. NEW_NODE Role [prepare](../../roles/prepare/tasks/main.yml)
3. NEW_NODE install [docker](../setup/03.Install_Docker.md) 
4. NEW_NODE install [kube-node](../setup/05.Install_Kube-node.md)
5. NEW_NODE install [Network-compoent](../setup/06.Install_Network-Component.md)

### In the deploy nodes

``` bash
$ ssh-copy-id -i ~/.ssh/id_rsa.pub NEW_NODE_IP
$ easzctl add-node NEW_NODE_IP
example:
$ easzctl add-node 192.168.0.33
```

## 2.Validation

``` bash
# 验证新节点状态
$ kubectl get node

# 验证新节点的网络插件calico 或flannel 的Pod 状态
$ kubectl get pod -n kube-system

# 验证新建负载能否调度到新节点，略
```

