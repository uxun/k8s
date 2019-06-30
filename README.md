# Kubernetes **Op**erations 

##### Ansible-playbook install kubernetes cluster

## Supported version

| Name       | Version  |
| ---------- | -------- |
| Kubernetes | >v1.13   |
| Etcd       | >v3.1    |
| Docker     | >18.09.x |

## Install the cluster

| Deployment                                                   | Hosts |
| ------------------------------------------------------------ | ----- |
| <a href="docs/setup/00.K8S_Deployment.md">K8S-Deployment</a> | 5     |

## Reference

> Note：Addons {DNS,Metrics-server,Dashboard} integration info the [NO.07](docs/setup/07.Install_Cluster-Addons.md)

| Task                                                         | Addons                                                       | Cluster management                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------- |
| <a href="docs/setup/01.Install_Prepare.md">01.Install_Prepare</a> | <a href="docs/handbook/dns.md">DNS</a>                       | <a href="docs/op/AddNode.md">Add-Node</a>      |
| <a href="docs/setup/02.Install_ETCD_cluster.md">02.Install_ETCD_cluster</a> | <a href="docs/handbook/metrics-server.md">Metrics-server</a> | <a href="docs/op/AddMaster.md">Add-Master</a>  |
| <a href="docs/setup/03.Install_Docker.md">03.Install_Docker</a> | <a href="docs/handbook/dashboard.md">Dashboard</a>           | <a href="docs/op/ManageETCD.md">ManageETCD</a> |
| <a href="docs/setup/04.Install_Kube-master.md">04.Install_Kube-master</a> | <a href="docs/handbook/ingress-nginx.md">Ingress-nginx</a>   |                                                |
| <a href="docs/setup/05.Install_Kube-node.md">05.Install_Kube-node</a> | <a href="docs/handbook/ingress-traefik.md">Ingress-traefik(Top)</a> |                                                |
| <a href="docs/setup/06.Install_Network-Component.md">06.Install_Network-compoent</a> | <a href="docs/setup/Install_Harbor.md">Harbor</a>            |                                                |
| <a href="docs/setup/07.Install_Cluster-Addons.md">07.Install_Cluster-addons</a> | <a href="docs/handbook/helm.md">Helm</a>                     |                                                |

