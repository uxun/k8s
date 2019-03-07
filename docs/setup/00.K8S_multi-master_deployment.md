## Kube  multi-master deployment

##### 快速部署: (Centos 7)

| Name       | Note          |                         |
| ---------- | ------------- | ----------------------- |
| OS         | Centos 7      | DisK > 30G ,Memory > 2G |
| ansible    | version 2.7.7 |                         |
| docker     | 17.03.2       |                         |
| kubernetes | 1.13.3        |                         |
| etcd       | V3.2.24       | 奇数{1,3,5…}个节点      |
|            |               |                         |

#### 1.Install depend 

```sh
# 安装epel源
$ yum install epel-release -y
# 安装依赖工具
$ yum install git python python-pip -y
```

#### 2.Install ansible

```shell
#安装ansible 
#pip install ansible
$ pip install pip --upgrade -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com

$ pip install --no-cache-dir ansible -i http://mirrors.aliyun.com/pypi/simple/ --trusted-host mirrors.aliyun.com
```

#### 3.Ansible K8S deploy

```shell
#1.download project 
$ git clone https://github.com/uxun/kube.git
$ mkdir -p /etc/ansible
$ mv kube/* /etc/ansible

#2.download Binaries (You need a network environment) 
$ cd down/
$ ./download.sh 
#----download {k8s,etcd,docker,cfss,docker-compose,cni...}

#3.配置ansible ssh密钥登陆
# $IP = {clusters IP} 
ssh-keygen -t rsa -b 2048
ssh-copy-id $IP 
```

#### 4.Configure the cluster

```shell
$ cd /etc/ansible && cp example/hosts.m-masters.example hosts
# alter IP and other
$ vim /etc/ansible/hosts
# verify
$ ansible all -m ping 
# A step
ansible-playbook 01.prepare.yml
ansible-playbook 02.etcd.yml
ansible-playbook 03.docker.yml
ansible-playbook 04.kube-master.yml
ansible-playbook 05.kube-node.yml
ansible-playbook 06.network.yml
ansible-playbook 07.cluster-addon.yml
```

