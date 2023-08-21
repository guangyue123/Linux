# 1、Mariadb设置

## 1、数据库服务

### 启动数据库服务

`systemctl enable mariadb.service
systemctl start mariadb.service`



## 1.2、登陆数据库

### 使用mysql登陆数据库，数据库密码：zhitu2017

`mysql -u root -p`


## 1.3、创建数据库实例

### 在mariadb中创建库实例 ，名为keystone的数据库

`CREATE DATABASE keystone;`

## 1.4、创建用户

### 建用户keystone,密码zhitu2017，然后退出


`GRANT ALL PRIVILEGES ON keystone.* TO 'keystone'@'localhost' IDENTIFIED BY 'zhitu2017';`

​`exit;`


# 2、安装openstack-keystone

## 2.1、生成随机数

### 生成随机数作为管理用户Token，记下随机值

`openssl rand -hex 10`


## 2.2、安装openstack-keystone

### 已使用yum install openstack-keystone安装keystone服务，查看服务状态

`yum list openstack-keystone`


## 2.3、更改keystone.conf配置

### 编辑keystone.conf文件

 `vi /etc/keystone/keystone.conf`

修改内容：

```
[DEFAULT]

admin_token =  4f0b0f958218b53ec9b1

[database]

connection = mysql+pymysql://keystone:zhitu2017@controller/keystone

[token]

provider = fernet
```



## 2.4、初始化操作

### 初始化身份认证服务的数据库：

`su -s /bin/sh -c "keystone-manage db_sync" keystone`

### 初始化Fernet keys：

`keystone-manage fernet_setup \
  --keystone-user keystone \
  --keystone-group keystone
`




# 3、配置apache

## 3.1、更改httpd.conf配置

### 编辑httpd.conf

`vi /etc/httpd/conf/httpd.conf设置ServerName：
ServerName controller`



## 3.2、更改wsgi-keystone.conf配置

### 编辑wsgi-keystone.conf

`vi /etc/httpd/conf.d/wsgi-keystone.conf`
编辑内容如下：

```
Listen 5000

Listen 35357

<VirtualHost *:5000>

    WSGIDaemonProcess keystone-public processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}

    WSGIProcessGroup keystone-public

    WSGIScriptAlias / /usr/bin/keystone-wsgi-public

    WSGIApplicationGroup %{GLOBAL}

    WSGIPassAuthorization On

    ErrorLogFormat "%{cu}t %M"

    ErrorLog /var/log/httpd/keystone-error.log

    CustomLog /var/log/httpd/keystone-access.log combined



    <Directory /usr/bin>

        Require all granted

    </Directory>

</VirtualHost>

<VirtualHost *:35357>

    WSGIDaemonProcess keystone-admin processes=5 threads=1 user=keystone group=keystone display-name=%{GROUP}

    WSGIProcessGroup keystone-admin

    WSGIScriptAlias / /usr/bin/keystone-wsgi-admin

    WSGIApplicationGroup %{GLOBAL}

    WSGIPassAuthorization On

    ErrorLogFormat "%{cu}t %M"

    ErrorLog /var/log/httpd/keystone-error.log

    CustomLog /var/log/httpd/keystone-access.log combined



    <Directory /usr/bin>

        Require all granted

    </Directory>

</VirtualHost>
```


## 3.3、启动httpd服务

### 启动httpd

`systemctl enable httpd.service`

`systemctl start httpd.service`


# 4、创建服务

## 4.1、配置临时环境变量

### 配置环境变量OS_TOKEN、OS_URL、OS_IDENTITY_API_VERSION，OS_TOKEN值是上面生成随机数的值

`export OS_TOKEN=4f0b0f958218b53ec9b1`

`export OS_URL=http://controller:35357/v3`

`export OS_IDENTITY_API_VERSION=3`


## 4.2、创建服务

### 创建keystone服务

`openstack service create  --name keystone --description "OpenStack Identity" identity`


## 4.3、创建服务端点

### 创建public、internal、admin服务端点

`openstack endpoint create --region RegionOne identity public http://controller:5000/v3`

`openstack endpoint create --region RegionOne identity internal http://controller:5000/v3`

`openstack endpoint create --region RegionOne identity admin http://controller:35357/v3`


## 4.4、创建域名default

### 创建域名default

`openstack domain create --description "Default Domain" default`


## 4.5、创建项目admin及service

### 基于域default，创建项目admin及service

`openstack project create --domain default --description "Admin Project" admin`

`openstack project create --domain default --description "Service Project" service`


## 4.6、创建用户admin

### 基于域default，创建用户admin ，密码zhitu2017

`openstack user create --domain default --password-prompt admin`


## 4.7、创建角色admin

### 创建角色admin，关联角色到项目及用户

`openstack role create admin`

`openstack role add --project admin --user admin admin`

# 5、验证

## 5.1、更改keystone-paste.ini配置

### 编辑keystone-paste.ini，去除临时认证，基于安全考虑(admin_token_auth删除，其它不动)

`vi /etc/keystone/keystone-paste.ini`

编辑内容如下：
```
[pipeline:public_api]

pipeline = healthcheck cors sizelimit http_proxy_to_wsgi osprofiler url_normalize request_id build_auth_context token_auth json_body ec2_extension public_service 

[pipeline:admin_api]

pipeline = healthcheck cors sizelimit http_proxy_to_wsgi osprofiler url_normalize request_id build_auth_context token_auth json_body ec2_extension public_service 

[pipeline:api_v3]

pipeline = healthcheck cors sizelimit http_proxy_to_wsgi osprofiler url_normalize request_id build_auth_context token_auth json_body ec2_extension public_service
```

## 5.2、验证

### 去除环境变量

### unset OS_TOKEN OS_URL请求验证,输入密码后（zhitu2017），得到期望结果
`openstack --os-auth-url http://controller:35357/v3 \
--os-project-domain-name default --os-user-domain-name default \
--os-project-name admin --os-username admin token issue`

### 创建脚本
`vi ~/admin-openrc`

增加以下内容
```
export OS_PROJECT_DOMAIN_NAME=default

export OS_USER_DOMAIN_NAME=default

export OS_PROJECT_NAME=admin

export OS_USERNAME=admin

export OS_PASSWORD=zhitu2017

export OS_AUTH_URL=http://controller:35357/v3

export OS_IDENTITY_API_VERSION=3

export OS_IMAGE_API_VERSION=2
```
### 验证结果：未出错，并显示提示，如失效日期等。

### 下次如果想用此环境变量，可以先执行： . ~/admin-openrc

`. ~/admin-openrc`

`openstack token issue`