# Harbor

## 1.Installation preparation

1. on deploy node `cd /etc/ansible/down`

2. Download {docker-compose-1.23.0,harbor-offline-installer-v1.6.3}

    ```shell
    wget https://github.com/docker/compose/releases/download/1.23.0/docker-compose-Linux-x86_64
    wget https://storage.googleapis.com/harbor-releases/release-1.6.0/harbor-offline-installer-v1.6.3.tgz
    ```

3. deploy node modify `/etc/ansible/hosts`

``` bash
# 参数 NEW_INSTALL=(yes/no)：yes表示新建 harbor，并配置k8s节点的docker可以使用harbor仓库
# no 表示仅配置k8s节点的docker使用已有的harbor仓库
# 如果不需要设置域名访问 harbor，可以配置参数 HARBOR_DOMAIN=""
[harbor]
192.168.1.8 HARBOR_DOMAIN="harbor.yourdomain.com" NEW_INSTALL=yes
```

4. on deploy `ansible-playbook /etc/ansible/harbor.yml`

## 2.Detailed steps

```
roles/harbor
├── defaults
│   └── main.yml
├── tasks
│   └── main.yml
└── templates
    ├── harbor-csr.json.j2
    └── harbor-v1.6.cfg.j2
```

根据 `harbor.yml`文件，harbor节点需要以下步骤：

1. role `prepare` 基础系统环境准备
1. role `docker` 安装docker
1. role `harbor` 安装harbor

`kube-node`节点在harbor部署完之后，需要配置harbor的证书，并可以在hosts里面添加harbor的域名解析，如果你的环境中有dns服务器，可以跳过hosts文件设置

⌘+  [roles/harbor/tasks/main.yml](../../roles/harbor/tasks/main.yml)，对照以下讲解

1. 下载docker-compose可执行文件到$PATH目录
1. 自注册变量result判断是否已经安装harbor，避免重复安装问题
1. 解压harbor离线安装包到指定目录
1. 导入harbor所需 docker images
1. 创建harbor证书和私钥(复用集群的CA证书)
1. 修改harbor.cfg配置文件
1. 启动harbor安装脚本

## 3. Use harbor

> 1. 在harbor节点使用`docker ps -a` 查看harbor容器组件运行情况
> 2. 浏览器访问harbor节点的IP地址 `https://$NodeIP`，使用账号 admin 和 密码 Harbor12345 (harbor.cfg 配置文件中的默认)登陆系统
> 3. admin用户web登陆后可以方便的创建项目，并指定项目属性(公开或者私有)；然后创建用户，并在项目`成员`选项中选择用户和权限；

### Push image

在node上使用harbor私有镜像仓库首先需要在指定目录配置harbor的CA证书，详见 `harbor.yml`文件。

使用docker客户端登陆`harbor.test.com`，然后把镜像tag成 `harbor.test.com/$项目名/$镜像名:$TAG` 之后，即可使用docker push 上传

``` bash
# login YOU harbor
$ docker login harbor.test.com
Username: 
Password:
Login Succeeded

# tag you dockerimage
$ docker tag busybox:latest harbor.test.com/library/busybox:latest

# push docker images
$ docker push harbor.test.com/library/busybox:latest
The push refers to a repository [harbor.test.com/library/busybox]
0271b8eebde3: Pushed 
latest: digest: sha256:91ef6c1c52b166be02645b8efee30d1ee65362024f7da41c404681561734c465 size: 527
```
#### k8s中使用harbor

1. 如果镜像保存在harbor中的公开项目中，那么只需要在yaml文件中简单指定harbor私有镜像即可，例如

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-busybox
spec:
  containers:
  - name: test-busybox
    image: harbor.test.com/xxx/busybox:latest
    imagePullPolicy: Always
```

2. 如果镜像保存在harbor中的私有项目中，那么yaml文件中使用该私有项目的镜像需要指定`imagePullSecrets`，例如

``` yaml
apiVersion: v1
kind: Pod
metadata:
  name: test-busybox
spec:
  containers:
  - name: test-busybox
    image: harbor.test.com/xxx/busybox:latest
    imagePullPolicy: Always
  imagePullSecrets:
  - name: harborkey1
```
其中 `harborKey1`可以用以下两种方式生成：

+ 1.使用 `kubectl create secret docker-registry harborkey1 --docker-server=harbor.test.com --docker-username=admin --docker-password=Harbor12345 --docker-email=team@test.com`
+ 2.使用yaml配置文件生成 

``` yaml
//harborkey1.yaml
apiVersion: v1
kind: Secret
metadata:
  name: harborkey1
  namespace: default
data:
    .dockerconfigjson: {base64 -w 0 ~/.docker/config.json}
type: kubernetes.io/dockerconfigjson
```
前面docker login会在~/.docker下面创建一个config.json文件保存鉴权串，这里secret yaml的.dockerconfigjson后面的数据就是那个json文件的base64编码输出（-w 0让base64输出在单行上，避免折行）

### 管理harbor

+ 日志目录 `/var/log/harbor`
+ 数据目录 `/data` ，其中最主要是 `/data/database` 和 `/data/registry` 目录，如果你要彻底重新安装harbor，删除这两个目录即可

先进入harbor安装目录 `cd /data/harbor`，常规操作如下：

1. 暂停harbor `docker-compose stop` : docker容器stop，并不删除容器
2. 恢复harbor `docker-compose start` : 恢复docker容器运行
3. 停止harbor `docker-compose down -v` : 停止并删除docker容器
4. 启动harbor `docker-compose up -d` : 启动所有docker容器

修改harbor的运行配置，需要如下步骤：

``` bash
# 停止 harbor
 docker-compose down -v
# 修改配置
 vim harbor.cfg
# 执行./prepare已更新配置到docker-compose.yml文件
 ./prepare
# 启动 harbor
 docker-compose up -d
```
#### harbor 升级

以下步骤基于harbor 1.1.2 版本升级到 1.2.2版本 

``` bash
# 进入harbor解压缩后的目录，停止harbor
cd /data/harbor
docker-compose down

# 备份这个目录
cd ..
mkdir -p /backup && mv harbor /backup/harbor

# 下载更新的离线安装包，并解压
tar zxvf harbor-offline-installer-v1.2.2.tgz  -C /data

# 使用官方数据库迁移工具，备份数据库，修改数据库连接用户和密码，创建数据库备份目录
# 迁移工具使用docker镜像，镜像tag由待升级到目标harbor版本决定，这里由 1.1.2升级到1.2.2，所以使用 tag 1.2
docker pull vmware/harbor-db-migrator:1.2
mkdir -p /backup/db-1.1.2
docker run -it --rm -e DB_USR=root -e DB_PWD=xxxx -v /data/database:/var/lib/mysql -v /backup/db-1.1.2:/harbor-migration/backup vmware/harbor-db-migrator:1.2 backup

# 因为新老版本数据库结构不一样，需要数据库migration
docker run -it --rm -e DB_USR=root -e DB_PWD=xxxx -v /data/database:/var/lib/mysql vmware/harbor-db-migrator:1.2 up head

# 修改新版本 harbor.cfg配置，需要保持与老版本相关配置项保持一致，然后执行安装即可
cd /data/harbor
vi harbor.cfg
./install.sh
```
