# kolla_template

#### 介绍
使用kolla-ansible部署openstack所用到的所有配置文件.

#### 软件架构
项目按openstack版本进行分支，每个分支下一级目录是操作系统，二级目录是网络平面。
#### kolla支持系统
CentOS 7  
CentOS Stream 8,9
Rocky Linux 8,9  
Ubuntu 16.04,18.04,20.04,22.04,24.04   
Debian 11,12   
OpenEuler 22.03 LTS   

#### 网络平面规划
Openstack是SOA架构的软件，所有组件都通过一个URL对外提供服务。这个URL叫Endpoint（端点）。
在Openstack中端点分为三类：管理URL、内部URL、外部URL。   
为了给这三类端点分配ip， kolla设计了两种模式：融合模式，就是三种端点使用同一个IP地址段。分离模式，将外部URL单独一个IP地址段，让内部URL和管理URL使用相同IP地址段。

进而在 Kolla 中，开发人员定义了如下几个网络平面：
1. network_interface - 虽然它本身不单独使用，但为下面的其他接口提供了所需的默认值。
2. api_interface（管理） - 这个接口用于管理网络。管理网络是 OpenStack 服务用于相互通信和与数据库通信的网络。这里存在已知的安全风险，因此建议将此网络设置为内部网络，不对外部开放。默认为 network_interface。
3. kolla_external_vip_interface（外部） - 这个接口是面向公众的。当你希望 HAProxy 的公共端点在不同于内部端点的网络中暴露时使用。当 kolla_enable_tls_external 设置为 yes 时，必须设置此选项。默认为 network_interface。
4. swift_storage_interface（内部） - 这个接口被 Swift 用于存储访问流量。由于可能会被大量使用，建议使用高速网络结构。默认为 network_interface。
5. swift_replication_interface（内部） - 这个接口被 Swift 用于存储复制流量。由于可能会被大量使用，建议使用高速网络结构。默认为 swift_storage_interface。
6. tunnel_interface（内部） - 这个接口被 Neutron 用于虚拟机间通过隧道网络（如 VxLan）的流量。默认为 network_interface。
7. neutron_external_interface（外部） - 这个接口是 Neutron 所需的。Neutron 会在其上放置 br-ex。它将用于平面网络以及标记的 VLAN 网络。必须单独设置。
8. dns_interface（外部） - 这个接口是 Designate 和 Bind9 所需的。用于面向公众的 DNS 请求和对 bind9 及 designate mDNS 服务的查询。默认为 network_interface。
9. bifrost_network_interface（管理） - 这个接口是 Bifrost 所需的。用于配置裸金属云主机，需要与裸金属云主机进行 L2 连接，以便提供带有 PXE 启动选项的 DHCP 租约。默认为 network_interface。

最理想的情况是，服务器上网口足够多，每一个网络平面处于一个vlan网段中，使用一个或多个网口。虽然网络相互隔离具有较高的安全性，但是这会造成大量的资源浪费。
经过总结，本项目规划如下：

1. 管理面网络（M）= api_interface + bifrost_network_interface
2. 存储面网络（S）= swift_storage_interface + swift_replication_interface
3. 租户面网络（T）= tunnel_interface
4. 外部网络（E）= neutron_external_interface 
5. 公共网络（P）= kolla_external_vip_interface + dns_interface

管理面网络、租户网络、外部网络是必须的，存储面网络和公共网络在某些场景下是不存在的。   
因此，在使用kolla部署openstack前，需要合理规划哪个网络运行在哪个网口上。
#### 服务器分类
为了更好的管理服务器，Kolla将服务器分为以下几类：
1. 管理节点（Controller） - 管理节点主要用于部署数据库、消息队列、用户认证、计算调度等Openstack组件。管理节点瘫痪不影响已有客户虚拟机的运行，
2. 计算节点（Compute） - 计算节点主要用于运行虚拟机，需要有足够的CPU、内存、网络带宽等资源。
3. 存储节点（Storage） - 存储节点主要用于部署Openstack组件，如Swift、Cinder等，或者部署Ceph分布式存储。存储节点不需要非常强大的CPU和内存，但是需要很高的硬盘空间和网络带宽。
4. 网络节点（Network） - 这类主机的特点是连接了可以访问互联网的路由器，处于公网NAT下，或者可以被分配到公网IP，br-ex网桥将被部署到这类服务器上。
5. 监控节点（Monitor） - 用于部署计费组件，如ceilometer、gnocchi等。私有云，或者不计成本的项目，可以不需要这类服务器。

#### 安装教程

1.  选择自己想要部署的openstack版本，克隆本项目，例如：git clone https://gitee.com/yangjiacheng1996/kolla_template.git -b zed
2.  根据自己的操作系统和网络平面，进入对应的目录，例如：cd kolla_template/debian/m_e
3.  to be continued...

#### 软件版本对应表
Kolla-Ansible Version|Ansible Version|OpenStack Version|Supported Operating Systems|
|-|-|-|-|
|18.1.1.dev21|8 to 9 (ansible-core 2.15 to 2.16)|Antelope, Caracal	|CentOS Stream 9, Debian 11 (Bullseye), Debian 12 (Bookworm), openEuler 22.03 LTS, Rocky Linux 9, Ubuntu 22.04 (Jammy)|

#### 作者与合作
项目Owner拼音：Yang Jiacheng   
Kolla一对一教学：2290906844，闲鱼下单。
私有云建设项目合作：根据节点数计算维护费用，每台1000元/年。签订合同。   









