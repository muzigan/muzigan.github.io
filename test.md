> [参考文档](http://docs.ceph.org.cn/start/quick-ceph-deploy/)

```bash
# hostname 更改并更改hosts解析
echo "
127.0.0.1   localhost
192.168.88.217   node217
192.168.88.212   node212
192.168.88.213   node213
" > /etc/hosts


# 更换base和epel源 （all-node）
wget -O /etc/yum.repos.d/CentOS-Base.repo http://mirrors.163.com/.help/CentOS7-Base-163.repo
rpm -Uvh http://mirrors.ustc.edu.cn/centos/7/extras/x86_64/Packages/epel-release-7-11.noarch.rpm
rpm --import /etc/pki/rpm-gpg/RPM-GPG-KEY-EPEL-7

# 安装ceph-deploy  安装高版本
yum install ceph-deploy

# 声明ceph-repo变量
export CEPH_DEPLOY_REPO_URL=http://mirrors.163.com/ceph/rpm-jewel/el7
export CEPH_DEPLOY_GPG_URL=http://mirrors.163.com/ceph/keys/release.asc

# 进入ceph的配置文件目录
mkdir /etc/ceph; cd /etc/ceph

# 新建集群，为节点安装ceph
ceph-deploy  new node217 
ceph-deploy  install node217 node212 node213 


#添加配置到ceph.conf,= 前后注意要有空格
echo "public_network = 192.168.88.0/24
cluster_network = 192.168.88.0/24" >>./ceph.conf

# 同步配置文件;配置初始 monitor(s)、并收集所有密钥
ceph-deploy --overwrite-conf  config push  node217 node212 node213
ceph-deploy mon create-initial

# 添加osd（使用目录并非磁盘）
ceph-deploy osd prepare  node217:/ceph    node212:/ceph   node213:/ceph
ceph-deploy osd activate  node217:/ceph    node212:/ceph   node213:/ceph

# 添加mon节点

ceph-deploy mon  add node213 node212

```
