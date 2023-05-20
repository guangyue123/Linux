# 云计算竞赛训练平台
**竞赛训练平台网址**：[云计算竞赛平台](http://10.16.87.67:7070/ "竞赛训练")    
**云电脑网址**： [VMware® vSphere](https://10.16.86.111/ "竞赛用机")
![1682243815588](image/jingsai/1682243815588.png)

- （1）更改网卡名称  
    **此界面按Tab键**
 ![1684392330105](image/jingsai/1684392330105.png)
    > net.ifnames=0  biosdevname=0
- （2）网络配置
    > vi /etc/sysconfig/network-scripts/ifcfg-eth1
    ![1684392984932](image/jingsai/1684392984932.png)
    - [x] **systemctl restart network**重启网络服务
    - [x] **ip a**查看是否有ip
    ![1684393143920](image/jingsai/1684393143920.png)
- （3）设置控制节点主机名为controller，设置计算节点主机名为compute
    > hostnamectl set-hostname <mark>主机名</mark>

    - [x] **hostnamectl**查看是否为更改的主机名
- （4）修改hosts文件将IP地址映射为主机名
    > vi /etc/hosts

    ![修改hosts文件](image/jingsai/1682245822093.png)  
    - [x] **ping controller**和**ping compute**可以ping得通
    - [x] moba ssh服务连接得上 
---
![1684393496041](image/jingsai/1684393496041.png)

- （1）移除原有yum源文件
    > mv /etc/yum.repos.d/* /media/
- （2）新建一个yum源文件http.repo
    > vi /etc/yum.repos.d/http.repo
- （3）编辑文件填写以下内容
    ![1684393970236](image/jingsai/1684393970236.png)
- （4）添加本地缓存
    > yum makecache  
    
    ![1684394115160](image/jingsai/1684394115160.png)
    - [x] **yum repolist**查看yum源是否配置成功
    ![1684394101602](image/jingsai/1684394101602.png)
---
![1684394313617](image/jingsai/1684394313617.png)
- （1）安装iaas-xiandian
    > yum install iaas-xiandian -y
- （2）配置xiandan文件
    > vi /etc/xiandian/openrc.sh
- （3）跑脚本iaas-pre-host.sh
    > iaas-pre-host.sh
    - [x] 跑脚本没有出错
---
![1684395299833](image/jingsai/1684395299833.png)

- （1）fdisk命令创建分区
  - [x] lsblk查看是否分区成功
---
![1684395469799](image/jingsai/1684395469799.png)
- [x] 直接跑分
---
![1684395497565](image/jingsai/1684395497565.png)
- （1）**controller**上跑脚本iaas-install-mysql.sh
    > iaas-install-mysql.sh
- （2）编辑数据库文件
    > vi /etc/my.cof

    ![1684395749740](image/jingsai/1684395749740.png)
    - [x] systemctl restart mariadb重启数据库服务
---
![1684395840264](image/jingsai/1684395840264.png)
- （1）运行脚本
    > iaas-install-keystone.sh
- （2）
    > source /etc/keystone/admin-openrc.sh
- （3）创建openstack用户
    > openstack user create --domin demo --password 000000 chinaskill
    ![1684396183383](image/jingsai/1684396183383.png)
    - [x] openstack user list
    ![1684396166733](image/jingsai/1684396166733.png)
---
![1684396254073](image/jingsai/1684396254073.png)

- （1）运行脚本
    > iaas-install-glance.sh
- （2）下载镜像
    > curl -O http://10.16.81.47:30808/1-iaas/cirros-0.3.4-x86_64-disk.img
- （3）命名为cirros，并设置最小启动需要的硬盘为10G
    > glance image-create --name "cirros" --disk format qcow2 --min-disk 10 --container-format bare --progress < ./cirros-0.3.4-x86_64-disk.img
    - [x] openstack image list
    ![1684397017571](image/jingsai/1684397017571.png)
---
![1684397282206](image/jingsai/1684397282206.png)

- （1）controller，compute分别运行脚本
    > iaas-install-nova-controller.sh
    > iaas-install-nova-compute.sh
- （2）配置nova文件
    > vi /etc/nova/nova.conf
  