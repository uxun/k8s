## Kube  multi-master deployment

##### 快速部署: (Centos 7)

| Name           | Note          |                                  |
| -------------- | ------------- | -------------------------------- |
| OS             | Centos 7      | DisK > 30G ,Memory > 2G          |
| ansible        | version 2.7.7 |                                  |
| docker         | 18.06.3-ce    |                                  |
| kubernetes     | 1.13.3        |                                  |
| etcd           | V3.2.24       | 奇数{1,3,5…}个节点               |
| docker-compose | 1.23.0        |                                  |
| cfssl          | R1.2          | {cfssl,cfssljson,cfssl-certinfo} |
| CNI            | V0.6.0        |                                  |

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
# 需要下载
$ cd down/
$ ./download.sh 
#----download {k8s,etcd,docker,cfss,docker-compose,cni...}

#3.配置ansible ssh密钥登陆
# $IP = {clusters IP} 
ssh-keygen -t rsa -b 2048
ssh-copy-id -i ~/.ssh/id_rsz.pub $IP 
```

#### 4.Configure the cluster

```shell
$ cd /etc/ansible && cp example/hosts.m-masters.example hosts
# alter IP and other 
# 注意需要修改的ip和组件
$ vim /etc/ansible/hosts

# verify
$ ansible all -m ping 
# A step
ansible-playbook 01.prepare.yml
PLAY RECAP ***********************************************************************************************
192.168.0.25               : ok=51   changed=41   unreachable=0    failed=0
192.168.0.26               : ok=31   changed=25   unreachable=0    failed=0
192.168.0.27               : ok=17   changed=14   unreachable=0    failed=0
192.168.0.29               : ok=17   changed=14   unreachable=0    failed=0

ansible-playbook 02.etcd.yml
PLAY RECAP ***********************************************************************************************
192.168.0.25               : ok=11   changed=8    unreachable=0    failed=0
192.168.0.26               : ok=11   changed=9    unreachable=0    failed=0
192.168.0.27               : ok=11   changed=9    unreachable=0    failed=0

ansible-playbook 03.docker.yml
PLAY RECAP *****************************************************************************************************************
192.168.0.25               : ok=12   changed=11   unreachable=0    failed=0
192.168.0.26               : ok=11   changed=10   unreachable=0    failed=0
192.168.0.27               : ok=11   changed=10   unreachable=0    failed=0
192.168.0.29               : ok=11   changed=10   unreachable=0    failed=0

ansible-playbook 04.kube-master.yml
PLAY RECAP *****************************************************************************************************************
192.168.0.25               : ok=39   changed=34   unreachable=0    failed=0
192.168.0.26               : ok=38   changed=36   unreachable=0    failed=0

ansible-playbook 05.kube-node.yml
PLAY RECAP *****************************************************************************************************************
192.168.0.27               : ok=24   changed=22   unreachable=0    failed=0
192.168.0.29               : ok=24   changed=23   unreachable=0    failed=0

ansible-playbook 06.network.yml
PLAY RECAP *****************************************************************************************************************
192.168.0.25               : ok=10   changed=9    unreachable=0    failed=0
192.168.0.26               : ok=6    changed=5    unreachable=0    failed=0
192.168.0.27               : ok=6    changed=5    unreachable=0    failed=0
192.168.0.29               : ok=6    changed=5    unreachable=0    failed=0

ansible-playbook 07.cluster-addon.yml
```


