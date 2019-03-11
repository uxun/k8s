# Kubernetes **Op**erations 

##### Ansible-playbook install kubernetes cluster

## Supported version

| Name    | Version    |
| ------- | ---------- |
| OS      | CentOS 7   |
| K8S     | >v1.8      |
| etcd    | >v3.1      |
| docker  | 18.06.x-ce |
| network | flannel    |

## Install the cluster

| Deployment                                                   | Hosts |
| ------------------------------------------------------------ | ----- |
| <a href="docs/setup/00.K8S_multi-master_deployment.md">Multi-master-Deployment</a> | 4     |

## Reference

| Task                                                         | Addons                                                       |
| ------------------------------------------------------------ | ------------------------------------------------------------ |
| <a href="docs/setup/01.Hosts_environment_preparation.md">NO.01-Hosts_preparation</a> | <a href="docs/setup/07.Install_Cluster-Addons.md">CoreDNS</a> |
| <a href="docs/setup/02.Install_ETCD_cluster.md">NO.02-Install_ETCD_cluster</a> | <a href="docs/setup/07.Install_Cluster-Addons.md">Dashboard</a> |
| <a href="docs/setup/03.Install_Docker.md">NO.03-Install_Docker</a> | <a href="docs/handbook/metrics-server.md">Metrics-server</a> |
| <a href="docs/setup/04.Install_Kube-master.md">NO.04-Install_Kube-master</a> |                                                              |
| <a href="docs/setup/05.Install_Kube-node.md">NO.05-Install_Kube-node</a> |                                                              |
| <a href="docs/setup/06.Install_Network-Component.md">NO.06-Install_Network-compoent</a> |                                                              |
| <a href="docs/setup/07.Install_Cluster-Addons.md">NO.07-Install_Cluster-addons</a> |                                                              |

