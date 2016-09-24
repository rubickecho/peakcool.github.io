---
author: tangliangcheng
comments: true
date: 2016-01-07 01:50:26+00:00
layout: post
link: http://www.rmogo.com/wordpress/2016/01/07/openstack%e8%bf%90%e7%bb%b4%e5%a4%8d%e4%b9%a0%e6%80%bb%e7%bb%93/
slug: openstack%e8%bf%90%e7%bb%b4%e5%a4%8d%e4%b9%a0%e6%80%bb%e7%bb%93
title: OpenStack运维复习总结
wordpress_id: 56
categories:
- 技术分享
---

### 1.节点组件



    
    <code class="language-none">Control Node : Keystone glance nova cinder horizon
    
    Compute Node : Nova compute   Nova network   KVM</code>




### 2.配置网络接口



    
    <code class="language-none">vim /etc/network/interfaces
    auto eth0
    iface eth0 inet static
    address 10.18.0.40
    netmask 255.255.240.0
    network 10.18.0.0
    broadcast 10.18.0.255
    gateway 10.18.0.254
    dns-nameservers 61.139.2.69
    /etc/init.d/networking restart</code>


_2.1修改主机名并设置名称解析，控制节点ctrl，计算节点comp_

    
    <code class="language-none">vim /etc/hostname
    
    crtl
    
    vim /etc/hosts
    
    10.18.0.40  ctrl
    
    10.18.0.401 comp
    
    验证连通性：ping -c 4 ctrl/comp</code>


_2.2配置ipv4包转发_

    
    <code class="language-none">vim /etc/syscel.conf
    
    net.ipv4.ip-forword = 1</code>


_2.3更新系统及源_

    
    <code class="language-none">sudo apt-get update
    
    sudo apt-get install ubuntu-cloud-keyring(更新f版源)</code>


_2.4安转ntp服务（节点时间同步）_

    
    <code class="language-none">apt-get install -y ntp
    
    计算节点引用控制节点 server 10.18.0.40 iburst</code>




### 3.控制节点安装部分


_3.1 RabbitMQ安装，组件之间的通信_

    
    <code class="language-none">apt-get install -y rabbitmq-server</code>


_3.2 安转mysql数据库_

    
    <code class="language-none">apt-get install -y mysql-server python-mysqldb</code>


_3.3 安装keystone服务_

    
    <code class="language-none">创建数据库 
        mysql -u root -p       
        create database keystone
    安装keystone组件 apt-get install keystone -y
    修改配置文件 略
    初始化身份认证服务的数据 
        server keystone restart      
        keystone-manage db-sync
    生成管理员令牌 openssl rand -hex 10</code>


_3.4 安装glance服务_

    
    <code class="language-none">创建数据库 同keystone
    安装软件包  app-get install glance
    修改配置文件 
        vi /etc/glance/glance-api-paste.ini
        vi /etc/glance/glance-api.conf
        vi /etc/glance/glance-registry.conf
    重启服务
        service glance-api restart
        service glance-registry restart
        glance-manage db-sync</code>


_3.5 安装Nova_

    
    <code class="language-none">创建数据库 同keystone
    安装nova服务
        apt-get install -y nova-api nova-cert novnc nova-consoleauth nova-scheduler nova-novncproxy nova-network 
    修改配置文件 略
    同步数据库
        nova-manage db-sync
    查看服务器运行状态
        nova-manage service list
    查看可用镜像
        nova image-list</code>


_3.6 安装配置控制节点网络_

    
    <code class="language-none">安装软件包
        apt-get install -y bridge-utils</code>


_3.7 安装cinder_

    
    <code class="language-none">创建数据库 同ketstone
    安装cinder软件包
        apt-get install cinder-api cinder-scheduler cinder-volume iscsitarget iscsitarget-dkms
    同步数据库
        cinder-manage db-sync
    验证服务组件
        cinder list
    重启cinder服务
        service cinder-api restart
        service cinder-scheduler restart
        service cinder-rolume restart</code>


_3.8 安装horizon服务_

    
    <code class="language-none">apt-get install openstack-dashboard memcached -y</code>




### 安装计算节点网络


_－配置网络接口_

_－修改主机名并解析_

_－配置ipv4包转发_

_－更新源_

_－安装ntp_

以上5个步骤同安装控制节点

_安装软件包_

    
    <code class="language-none">apt-get install -y vlan bridge-utils</code>


_重启网络服务_

    
    <code class="language-none">brctl addbr br100
    /etc/init.d/networking restart</code>


_安装nova_

    
    <code class="language-none">apt-get -y install nova-api-metadata nova-compute-kvm nova-network</code>


_修改nova配置文件_

    
    <code class="language-none">2、修改配置文件
    vim  /etc/nova/api-paste.ini
    
    [filter:authtoken]
    auth_host = 10.18.2.195
    auth_port = 35357
    auth_protocol = http
    admin_tenant_name = service
    admin_user = nova
    admin_password = service_pass
    signing_dirname = /tmp/keystone-signing-nova
    vim  /etc/nova/nova.conf</code>


_kvm检测及安装_

    
    <code class="language-none">查看是否支持kvm
        kvm-ok
    安装kvm
        apt-get install -y kvm libvirt-bin pm-utils</code>




### 常见错误


错误1:

    
    <code class="language-none">error:Reading package lists... Done
        W: GPG error: http://ubuntu-cloud.archive.canonical.com precise-proposed/folsom Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 5EDB1B62EC4926EA
        W: GPG error: http://ubuntu-cloud.archive.canonical.com precise-updates/folsom Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY 5EDB1B62EC4926EA
    
    method: apt-get install ubuntu-cloud-keyring</code>


错误2:

    
    <code class="language-none">error: W: There is no public key available for the following key IDs:3B4FE6ACC0B21F32
    
    method:apt-key adv --recv-keys --keyserver keyserver.ubuntu.com 3B4FE6ACC0B21F32</code>


错误3;

    
    <code class="language-none">error:Reading package lists... Done
         W: GPG error: http://www.rabbitmq.com kitten Release: The following signatures couldn't be verified because the public key is not available: NO_PUBKEY F7B8CEA6056E8E56
    
    method: wget http://www.rabbitmq.com/rabbitmq-signing-key-public.asc
            apt-key add rabbitmq-signing-key-public.asc</code>


_通过管理界面创建实例不成功_

    
    <code class="language-none">1、检查Nova相关配置文件配置是否正确；
    2、检查Nova服务状态是否正常；
    3、检查虚拟机网络是否配置正确；
    4、检查Rabbitmq服务是否正常；
    5、检查所选镜像磁盘容量是否超过所分配实例磁盘容量；
    6、检查计算节点可利用资源是否不足；</code>


_Rabbitmq服务不正常_

    
    <code class="language-none">1、IP和主机名必须一致
    2、检查是否存在.erlang.cookie文件，以及属性是否正确
       chown rabbitmq:rabbitmq /var/lib/rabbitmq/.erlang.cookie
       chmod 400 /var/lib/rabbitmq/.erlang.cookie
       rabbitmqctl status
    3、检查是否存在rabbitmq.config
       gunzip -c  /usr/share/doc/rabbitmq-server/rabbitmq.config.example.gz  > /etc/rabbitmq/rabbitmq.config
    4、检查版本是否一致</code>
