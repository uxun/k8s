# Kubernetes **Op**erations 

##### Ansible-playbook install kubernetes cluster

## Supported version

| Name       | Version                                         |
| ---------- | ----------------------------------------------- |
| OS         | CentOS 7                                        |
| Kubernetes | >v1.8                                           |
| Etcd       | >v3.1                                           |
| Docker     | >18.06.x-ce \| 18.09.x (version num is changed) |
| Network    | flannel \| calico                               |

## Install the cluster

| Deployment                                                   | Hosts |
| ------------------------------------------------------------ | ----- |
| <a href="docs/setup/00.K8S_multi-master_deployment.md">Multi-master-Deployment</a> | 4     |

## Reference

| Task                                                         | Addons                                                       | Cluster management                             | Other                                             |
| ------------------------------------------------------------ | ------------------------------------------------------------ | ---------------------------------------------- | ------------------------------------------------- |
| <a href="docs/setup/01.Hosts_environment_preparation.md">NO.01-Hosts_preparation</a> | <a href="docs/handbook/dns.md">DNS</a>                       | <a href="docs/op/AddNode.md">Add-Node</a>      | <a href="docs/setup/Install_Harbor.md">Harbor</a> |
| <a href="docs/setup/02.Install_ETCD_cluster.md">NO.02-Install_ETCD_cluster</a> | <a href="docs/handbook/metrics-server.md">Metrics-server</a> | <a href="docs/op/AddMaster.md">Add-Master</a>  |                                                   |
| <a href="docs/setup/03.Install_Docker.md">NO.03-Install_Docker</a> | <a href="docs/handbook/dashboard.md">Dashboard</a>           | <a href="docs/op/ManageETCD.md">ManageETCD</a> |                                                   |
| <a href="docs/setup/04.Install_Kube-master.md">NO.04-Install_Kube-master</a> | <a href="docs/handbook/heapster.md">Heapster(abandoned)</a>  |                                                |                                                   |
| <a href="docs/setup/05.Install_Kube-node.md">NO.05-Install_Kube-node</a> |                                                              |                                                |                                                   |
| <a href="docs/setup/06.Install_Network-Component.md">NO.06-Install_Network-compoent</a> |                                                              |                                                |                                                   |
| <a href="docs/setup/07.Install_Cluster-Addons.md">NO.07-Install_Cluster-addons</a> |                                                              |                                                |                                                   |

