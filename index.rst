.. title:: Nutanix Enterprise Private Cloud Bootcamp

.. toctree::
   :maxdepth: 2
   :caption: 企业私有云
   :name: _enterprise_privatecloud
   :hidden:

   dayinlife/dayinlife
   prismops/prismops_capacity_lab/prismops_capacity_lab
   prismops/prismops_rightsize_lab/prismops_rightsize_lab
   files/files
   flow_secure_fiesta/flow_secure_fiesta
..   beam_cost_governance/beam_cost_governance

.. toctree::
  :maxdepth: 2
  :caption: 可选实验
  :name: _optional_labs
  :hidden:

  image_create/image_create
.. lab_image_configuration/lab_image_configuration

.. toctree::
  :maxdepth: 2
  :caption: 附录
  :name: _appendix
  :hidden:

  tools_vms/windows_tools_vm
  tools_vms/linux_tools_vm
  appendix/glossary

.. _getting_started:

---------------
新手上路
---------------

欢迎来到Nutanix企业私有云新手训练营！ 该工作手册随附了一个由讲师指导的课程，其中介绍了Nutanix Core技术和许多常见的管理任务。

您将探索Prism Central，并熟悉其功能和导航。 您将使用Prism执行基本的群集管理任务，包括存储和网络。 您还将通过Prism和AHV逐步完成基本的VM部署和管理任务。 最后，您将探索VM数据保护，包括快照和复制。 讲师解释练习并回答您可能有的其他问题。

在训练营结束时，与会人员应了解构成Nutanix企业云堆栈的核心概念和技术，并应为托管或现场概念验证（POC）参与做好充分的准备。

新增亮点
++++++++++

- Workshop updated for the following software versions:
    - AOS & PC 5.11.2.x

- Optional Lab Updates:


日程
++++++

- 简介
- Nutanix 宣讲
- 日常的一天
- Prism Ops
- Files
- Flow

初始化设置
+++++++++++++

- 记下使用的密码 *Passwords* 
- 登录连接到实验环境

环境细节信息
+++++++++++++++++++

Nutanix Bootcamp旨在在Nutanix托管POC环境中运行。 将为您的群集提供完成练习所需的所有必要映像，网络和VM。

网络
..........

Hosted POC 集群遵循标准的命名原则:

- **Cluster Name** - POC\ *XYZ*
- **Subnet** - 10.**21**.\ *XYZ*\ .0
- **Cluster IP** - 10.**21**.\ *XYZ*\ .37

在整个Workshop中，有多个实例需要用子网的正确八位位组替换 *XYZ*，例如：

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - IP Address
     - Description
   * - 10.21.\ *XYZ*\ .37
     - Nutanix Cluster Virtual IP
   * - 10.21.\ *XYZ*\ .39
     - **PC** VM IP, Prism Central
   * - 10.21.\ *XYZ*\ .40
     - **DC** VM IP, NTNXLAB.local Domain Controller

每个集群有 2 个VLANs 供VM使用:

.. list-table::
  :widths: 25 25 10 40
  :header-rows: 1

  * - Network Name
    - Address
    - VLAN
    - DHCP Scope
  * - Primary
    - 10.21.\ *XYZ*\ .1/25
    - 0
    - 10.21.\ *XYZ*\ .50-10.21.\ *XYZ*\ .124
  * - Secondary
    - 10.21.\ *XYZ*\ .129/25
    - *XYZ1*
    - 10.21.\ *XYZ*\ .132-10.21.\ *XYZ*\ .253

用户凭据
...........

.. note::

  The *<Cluster Password>* is unique to each cluster and will be provided by the leader of the Workshop.

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - Credential
     - Username
     - Password
   * - Prism Element
     - admin
     - *<Cluster Password>*
   * - Prism Central
     - admin
     - *<Cluster Password>*
   * - Controller VM
     - nutanix
     - *<Cluster Password>*
   * - Prism Central VM
     - nutanix
     - *<Cluster Password>*

每个集群有一个专门的 domain controller VM, **DC**, 负责向 **NTNXLAB.local** 域提供 AD 服务。域中填充了以下用户和组：
.. list-table::
   :widths: 25 35 40
   :header-rows: 1
   
   * - Group
     - Username(s)
     - Password
   * - Administrators
     - Administrator
     - nutanix/4u
   * - SSP Admins
     - adminuser01-adminuser25
     - nutanix/4u
   * - SSP Developers
     - devuser01-devuser25
     - nutanix/4u
   * - SSP Consumers
     - consumer01-consumer25
     - nutanix/4u
   * - SSP Operators
     - operator01-operator25
     - nutanix/4u
   * - SSP Custom
     - custom01-custom25
     - nutanix/4u
   * - Bootcamp Users
     - user01-user25
     - nutanix/4u

接入环境指导
+++++++++++++++++++

The Nutanix Hosted POC 环境有很多方法可以接入：

Lab 接入用户凭据
...........................

PHX Based Clusters:
**Username:** PHX-POCxxx-User01 (up to PHX-POCxxx-User20), **Password:** *<Provided by Instructor>*

RTP Based Clusters:
**Username:** RTP-POCxxx-User01 (up to RTP-POCxxx-User20), **Password:** *<Provided by Instructor>*


**Non-Employees** - Use **Lab Access User** Credentials

Employee Pulse Secure VPN
..........................

下载客户端:

PHX Based Clusters Login to: https://xld-uswest1.nutanix.com

RTP Based Clusters Login to: https://xld-useast1.nutanix.com

**Nutanix 员工** - Use your **NUTANIXDC** credentials
**非Nutanix 员工** - Use **Lab Access User** Credentials

部署客户端

在Pulse Secure 客户端, **Add** 一个连接:

For PHX:

- **Type** - Policy Secure (UAC) or Connection Server
- **Name** - X-Labs - PHX
- **Server URL** - xlv-uswest1.nutanix.com

For RTP:

- **Type** - Policy Secure (UAC) or Connection Server
- **Name** - X-Labs - RTP
- **Server URL** - xlv-useast1.nutanix.com

Frame VDI
.........

Login to: https://frame.nutanix.com/x/labs

**Nutanix Employees** - Use your **NUTANIXDC** credentials
**Non-Employees** - Use **Lab Access User** Credentials

Parallels VDI
.................

PHX Based Clusters Login to: https://xld-uswest1.nutanix.com

RTP Based Clusters Login to: https://xld-useast1.nutanix.com

**Nutanix Employees** - Use your **NUTANIXDC** credentials


Nutanix Version Info
++++++++++++++++++++

- **AHV Version** - AHV 20170830.337
- **AOS Version** - 5.11.2.3
- **PC Version** - 5.11.2.1
