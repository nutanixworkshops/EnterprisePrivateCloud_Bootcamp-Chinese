.. _dayinlife:

-----------------
日常的一天
-----------------


在本实验中，您将了解Carol O'Kay的生活，他是在3层架构上管理虚拟环境10年的资深人士，她最近部署了她的第一个Nutanix集群。 Nutanix群集用于混合生产IT工作负载，并支持其公司主要应用程序的工程工作，该程序是称为Fiesta的库存管理解决方案，用于支持公司的零售店面。

.. note::

   如果有多个人使用同一个Nutanix群集来完成此实验，则某些步骤可能已经完成。 如果发生这种情况，请跳过该特定步骤，并在确认步骤正确完成后继续进行实验。

配置存储
+++++++++++++++++++

在此简短的练习中，您将体验IT专家如何通过Prism只需单击几下即可通过Prism为虚拟化环境配置和监视主存储，这与规划和管理传统SAN存储形成了鲜明的对比。

#. 使用您的 **集群分配表格**，确定您的 **Prism Element** 集群IP。

#. 在浏览器中，打开 **Prism Element** 并使用以下本地用户凭据登录：

   - **Username** - admin
   - **Password** - *HPOC Password*

   .. figure:: images/1.png

#. 从下拉菜单中选择 **Storage** 。

   .. figure:: images/2.png

#. 将鼠标悬停在 **Storage Summary** 和 **Capacity Optimization** 小部件上，以解释每个显示的内容。

   重要的是要理解Nutanix认为 **Data Reduction** 比率仅来自压缩、重复数据删除和擦除编码的节省。 与许多其他供应商认为的“数据缩减”相比，**Overall Efficiency** 具有上述数据效率功能以及诸如精简配置，智能克隆和零抑制之类的数据避免功能。

   .. figure:: images/3.png

#. 选择 **Table** 并点击 **+ Storage Container**.

   存储容器代表用于存储的逻辑策略，使您可以创建保留，启用/禁用压缩，重复数据删除和擦除编码等数据效率功能，并配置冗余因子（RF）。 Nutanix群集上的每个存储容器仍会利用群集内的所有物理磁盘，称为存储池。 典型的Nutanix群集将具有少量的存储容器，通常对应于受益于不同数据效率技术的工作负载。

   .. figure:: images/4.png

#. 为 **Storage Container** 提供唯一的 **Name** ，然后单击 **Advanced Settings** 以浏览其他配置选项。

   .. figure:: images/5.png

   Nutanix提供了多种优化存储容量的方法，这些方法可以智能地适应工作负载的特性。 Nutanix使用原生数据节省（精简配置，智能克隆和零抑制）和数据缩减（压缩，重复数据删除和纠删码）技术来有效处理数据。 所有数据缩减优化均在容器级别执行，因此不同的容器可以使用不同的设置。

#. 单击 **Save** 以创建存储并将其安装到群集中的所有可用主机上。在vSphere或Hyper-V环境中，创建存储容器还将自动将存储安装到虚拟机管理程序。

#. 选择一个现有的存储容器，然后查看通过不同数据减少/避免功能以及 **Effective Capacity**（这是基于整体效率对可用存储的预测）得出的单个节省。 这些值可在 **Storage Container Details** 表中找到。


设置新网络
++++++++++++++++++++++++++++

在本练习中，Carol将使用Prism为群集配置新的VM网络。

AHV将Open vSwitch（OVS）用于所有VM网络。 OVS是在Linux内核中实现的开放源代码软件交换机，旨在在多服务器虚拟化环境中工作。 每个AHV服务器都维护一个OVS实例，并且所有OVS实例组合在一起形成一个逻辑交换机。 每个节点通常都上行链路到中继/标记到多个VLAN的物理交换机端口，这些端口将作为虚拟网络公开。

#.  从 **Prism Element** 下拉菜单中选择 **VM**

#. 选择 **Network Config**.

   .. figure:: images/9.png

#. 点击 **+ Create Network** 并使用 :ref:`clusterassignments`中的 **User** 特定网络详细信息填写以下字段：

   - **Name** - *Initials*-Network_IPAM
   - **VLAN ID** - A value (< 4096) other than your **Primary** or **Secondary** network VLANs
   - 选择 **Enable IP Address Management**
   - **Network IP Address / Prefix Length** - 10.0.0.0/24
   - **Gateway** - 10.0.0.1
   - 不要选择 **Configure Domain Settings**
   - 选择 **+ Create Pool**
   - **Start Address** - 10.0.0.100
   - **End Address** - 10.0.0.150
   - 点击 **Submit**

   .. figure:: images/network_config_03.png

    请注意，AHV能够提供集成的DHCP服务（IPAM），允许虚拟化管理员从配置的池中为虚拟机分配IP，或者在向虚拟机添加虚拟NIC时轻松地将IP指定为DHCP保留。

#. 点击 **Save**.

  现在，配置的虚拟网络将在群集中的所有节点上可用。 AHV中的虚拟网络的行为类似于ESXi中的分布式虚拟交换机，这意味着您无需在群集中的每个主机上配置相同的设置。

#. 关闭 **Network Configuration** 窗口。

响应VM创建请求
++++++++++++++++++++++++++++++++++

虚拟化管理员通常负责部署新的VM。 在本练习中，Carol将以Nutanix管理员的身份逐步在Prism中部署AHV VM。

#. 从 **Prism Element** 下拉菜单中返回 **VM** 页面

#. 点击 **+ Create VM**.

   .. figure:: images/10.png

#. 填写以下字段以完成用户VM请求：

   - **Name** - *Initials*\ -WinToolsVM
   - **Description** - 手动部署 Tools VM
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 4 GiB

   - 选择 **+ Add New Disk**
      - **Type** - DISK
      - **Operation** - Clone from Image Service
      - **Image** - WinToolsVM.qcow2
      - 选择**Add**

   - 选择 **Add New NIC**
      - **VLAN Name** - Secondary
      - 选择 **Add**

   与公共云提供商类似，Nutanix AHV提供映像服务功能，可让您建立导入文件的存储，可用于在创建VM时从ISO映像或操作系统磁盘挂载CD-ROM设备。 映像服务支持raw，vhd，vhdx，vmdk，vdi，iso和qcow2磁盘格式。

   请注意，VM创建向导还提供了为Windows Sysprep自动化指定Unattend.xml文件或为Linux OS配置指定Cloud-Init文件的功能。

#. 点击 **Save** 创建VM.

   .. note::

      可以使用AHV CLI ``acli``编写许多VM操作，包括创建VM。当前只能通过命令行为VM启用某些功能，例如安全启动和vNUMA。可以参考《ACLI参考指南》 `here <https://portal.nutanix.com/#/page/docs/details?targetId=Command-Ref-AOS-v5_16:acl-acli-vm-auto-r.html>`_.

      您可以通过SSH连接到任何Nutanix CVM，然后尝试使用 ``acli`` 创建其他VM。

#. 使用表顶部的搜索字段，过滤请求的VM。 选择虚拟机，然后从表下方的操作列表中单击 **Power On** 。

   .. figure:: images/12.png

#. VM完成启动后，记下 **IP Address**.

   .. figure:: images/11.png

  在以前的基础架构中，Carol遇到了新创建的VM网络无法正常工作的问题，并且不得不与网络管理员同行进行冗长的故障排除会话，以查明问题的根源。 借助AHV，Carol可以轻松地可视化已配置的虚拟机的完整网络路径。

#. 通过从 **Prism Element** 下拉菜单中选择 **Network** 页面并按VLAN或VM名称进行过滤来自己尝试。

   .. figure:: images/13.png

启动用户自助服务
++++++++++++++++++++++++++

虽然Prism和```acli``提供了用于创建VM的简单工作流，但Carol经常被这些请求所淹没，并且希望将自己的更多时间用于现代化组织老化的基础架构的其他部分以及参加儿子的足球比赛。

在以下练习中，Carol将使用自己的私有云，并利用Prism Central中的本机功能为用户提供IaaS自助服务。

＃ 返回 **Prism Element** 的 **Home** 页面。

＃ 通过单击 **Launch** 按钮并使用以下凭据登录来访问 **Prism Central** ：


   - **User Name** - admin
   - **Password** - *HPOC Password*

   .. figure:: images/6.png

探索 Categories
====================

一个类别 **Category** 是一个键值对。 根据某些条件（位置，生产级别，应用程序名称等），将Category分配给实体（例如VM，网络或映像）。 然后可以将策略映射到分配了特定Category值的那些实体。

例如，您可能具有一个Department类别，其中包含诸如Engineering，Finance和HR的值。 在这种情况下，您可以创建一个适用于Engineering和HR的备份策略，以及一个单独的（更严格）仅适用于Finance的备份策略。 Category允许您跨实体组实施各种策略，而Prism Central允许您快速查看任何已建立的关系。

在本练习中，您将为Carol创建一个自定义类别，以帮助调整对Fiesta应用程序团队适当资源的访问。

#. 在 **Prism Central**界面, 选择 :fa:`bars` **> Virtual Infrastructure > Categories**.

   .. figure:: images/14.png

#. 点击 **New Category** 并填写以下字段:

   - **Name** - *Initials*\ -Team
   - **Purpose** - Allowing resource access based on Application Team
   - **Values** - Fiesta

#. 点击 **Save**.

#. 点击现有的 **Environment** category 并记下相应的值. **Environment** 是一个 **SYSTEM** category, 尽管您可以添加其他值，但不能修改或删除category中或其任何现成的值。

   .. figure:: images/16.png

#. 选择 :fa:`bars` **> Virtual Infrastructure > VMs**.

#. 使用复选框，选择 **AutoAD** 和 **NTNX-BootcampFS-1** VM，然后单击 **Actions > Manage Categories**.

   .. figure:: images/17.png

   .. note::

      根据参与者的数量，您需要选择的某些VM可能在另一页上。 您可以搜索有问题的VM，单击以查看其他页面并选择VM，或者选择显示其他行。 这些技术中的任何一种都可以在界面的右上部分完成。

#. 在搜索栏首先键入 **Environment** 并选择 **Production** value, 并点击新增图标.

   .. figure:: images/18.png

   .. note::

      对于与安全，保护或恢复策略相关的category，相关策略将显示在此窗口中，以显示将类别应用于实体的影响。

#. 点击 **Save**.

#. 选择先前实验中的 *Initials*\ **-WinToolsVM** ，点击 **Actions > Manage Categories**. 分配 *Initials*\ **-Team: Fiesta** category, 点击新增按钮并点击 **Save**.

探索 Roles
===============

默认情况下，Prism Central附带有多个映射到普通用户角色的标准角色。 角色定义用户可以执行的操作，并映射到类别或其他实体。

Carol需要支持在Fiesta团队中工作的两种类型的用户：需要为测试环境提供VM的开发人员，以及监视组织内多个环境，但是修改每个环境的能力非常有限的操作员。

#. 在 **Prism Central**, 选择 :fa:`bars` **> Administration > Roles**.

   内置的Developer角色使用户可以创建和修改VM，创建，设置和管理 Calm Blueprints 等等。

#. 单击内置的 **Developer** 角色，并选择查看该角色的已批准操作。 请点击 **Manage Assignment**.

   .. figure:: images/19.png

#. 在 **Users and Groups**下，指定从NTNXLAB.local域自动发现的 **SSP Developers** 用户组。

#. 在 **Entities**下, 使用下拉菜单指定以下资源：

   - **AHV Cluster** - *Your Assigned Cluster*
   - **AHV Subnet** - Secondary
   - **Category** - Environment:Testing, Environment:Staging, Environment:Dev, *Initials*\ -Team:Fiesta

   .. figure:: images/20.png

#. 点击 **Save** 然后点击右上角的X关闭此屏幕。

    默认的Operator角色具有删除VM和从“蓝图”部署的应用程序的权限，在我们的环境中是不希望赋予这个权限的。 无需从头开始构建新角色，我们可以克隆到现有角色并进行修改以适应我们的需求。 所需的操作员角色应能够查看VM指标，执行电源操作并更新VM配置（例如vCPU或内存）以解决应用程序性能问题。


#. 单击内置的**Operator** 角色，然后单击 **Duplicate**.

#. 填写以下字段，然后单击 **Save** 以创建您的自定义角色:

   - **Role Name** - *Initials*\ -SmoothOperator
   - **Description** - Limited operator accounts
   - **App** - No Access
   - **VM** - Edit Access
   -  **不要** 选择 **Allow VM Creation**

   .. figure:: images/21.png

#. 刷新 **Prism** 并点击 **SmoothOperator** 角色. 点击 **Manage Assignment**.

#. 创建以下分配:

   - **Users and Groups** - operator01
   - **Entity Categories** - Environment:Production, Environment:Testing, Environment:Staging, Environment:Dev

   Operator01是有权访问所有带有任何Environment类别标记的VM的用户，但没有对特定集群的通用访问权限。

   单击 **New Users** 以向该角色添加其他分配：

   - **Users and Groups** - operator02
   - **Entity Categories** - Environment:Dev, *Initials*\ -Team:Fiesta

   Operator02是查看所有标记有Dev或Fiesta类别值的VM的用户。

   .. figure:: images/22.png

   点击**Save**.

#. 对于Carol等基础架构管理员，您可以将AD用户映射到 **Prism Admin** 或 **Super Admin** 角色，通过选择 :fa:`bars` **> Prism Central Settings > Role Mapping** 并添加新的 **Cluster Admin** 或 **User Admin** 映射到 AD 用户。

   .. figure:: images/28.png

探索 Projects
==================

前面的练习足以为Carol的用户提供基本的VM创建自助服务，但是他们的许多工作涉及由多个VM组成的应用程序。针对单个开发，测试或登台环境手动部署多个VM的速度很慢，并且会出现不一致和用户错误的情况。为了给用户提供更好的体验，Carol会将Nutanix Calm引入环境。

Nutanix Calm允许您跨私有（AHV，ESXi）和公共云（AWS，Azure，GCP）基础架构构建，配置和管理应用程序。

为了使非基础架构管理员能够访问Calm，从而允许他们创建或管理应用程序，必须首先将用户或组分配给 **Project**，该项目充当定义用户角色，基础结构资源和资源的逻辑容器。配额。项目定义了具有共同需求或共同结构与功能的一组用户，例如在Fiesta项目上进行协作的工程师团队。

#. 在 **Prism Central**, 选择 :fa:`bars` **> Services > Calm**.

#. 从左边菜单选择 **Projects** 并点击 **+ Create Project**.

   .. figure:: images/23.png

#. 填写以下字段：

   .. note::

      在添加基础架构资源之前添加用户/组映射可能会导致添加基础资源失败。 为了避免这种情况，请在用户/组映射之前添加基础资源。

   - **Project Name** - *Initials*\ -FiestaProject

   - 在 **Infrastructure**下, 选择 **Select Provider > Nutanix**

   - 点击 **Select Clusters & Subnets**

   - 选择 *Your Assigned Cluster*

   - 在 **Subnets**下, 选择 **Primary**, **Secondary**, 并点击 **Confirm**

   - 点击 :fa:`star` 标记 *Primary* 作为默认的网络。

   - 在 **Users, Groups, and Roles**下, 选择 **+ User**

      - **Name** - SSP Developers
      - **Role** - Developer
      - **Action** - Save

   - 选择 **+ User**

      - **Name** - Operator02
      - **Role** - *Initials*\ -SmoothOperator
      - **Action** - Save

   - 在 **Quotas**下, 详述

      - **vCPUs** - 100
      - **Storage** - <Leave Blank>
      - **Memory** - 100

   .. figure:: images/24.png

#. 点击 **Save & Configure Environment**.

``这会将您重定向到Envrionments页面，但是无需在此处进行配置。 进入下一步。``

请注意，只有 **Operator02** 有权访问 **Calm** 项目，而不是所有的Operator帐户。

Staging Blueprints
==================

蓝图是使用Nutanix Calm建模的每个应用程序的框架。蓝图是模板，描述了在已创建的服务和应用程序上置备，配置和执行任务所需的所有步骤。蓝图还定义了应用程序及其基础结构的生命周期，从创建应用程序到在应用程序上执行的操作（更新软件，向外扩展等）直到应用程序终止。

您可以使用蓝图对各种复杂的应用程序进行建模。从简单地配置单个虚拟机到配置和管理多节点，多层应用程序。

虽然开发人员用户可以创建和发布自己的蓝图，但Carol希望提供团队使用的通用Fiesta项目蓝图。


#. `下载 Fiesta-Multi Blueprint， 右击 <https://raw.githubusercontent.com/nutanixworkshops/ts2020/master/pc/dayinlife/Fiesta-Multi.json>`_.

#. 从 **Prism Central > Calm**页面, 选择 **Blueprints** 从左边菜单点击 **Upload Blueprint**.

   .. figure:: images/25.png

#. 选择 **Fiesta-Multi.json**.

#. 上传加上你名字缩写的蓝图 **Blueprint Name** ，即使跨多个项目 Calm Blueprint 名字也要唯一。

#. 选择 Calm project 并点击 **Upload**.

   .. figure:: images/26.png

#. 为了launch Blueprint 你必须先给VM分配网络. 选择 **NodeReact** 服务, 在右边的 **VM** 配置菜单, 选择 **Primary** 作为 **NIC 1** 网络。

#. 为 **NodeReact** 服务分配 *Initials*\ **-Team: Fiesta** and **Environment: Dev** categories.

   .. figure:: images/27.png

#. 为 **MySQL** 服务，重复 **NIC 1** and **Category** 分配.

#. 点击 **Credentials** 定义用于认证将由蓝图提供的CentOS VM的私钥。

   .. figure:: images/27b.png

#. 展开 **CENTOS** credential 并使用您的 SSH key, 或粘贴下面的 **SSH Private Key**:

   ::

      -----BEGIN RSA PRIVATE KEY-----
      MIIEowIBAAKCAQEAii7qFDhVadLx5lULAG/ooCUTA/ATSmXbArs+GdHxbUWd/bNG
      ZCXnaQ2L1mSVVGDxfTbSaTJ3En3tVlMtD2RjZPdhqWESCaoj2kXLYSiNDS9qz3SK
      6h822je/f9O9CzCTrw2XGhnDVwmNraUvO5wmQObCDthTXc72PcBOd6oa4ENsnuY9
      HtiETg29TZXgCYPFXipLBHSZYkBmGgccAeY9dq5ywiywBJLuoSovXkkRJk3cd7Gy
      hCRIwYzqfdgSmiAMYgJLrz/UuLxatPqXts2D8v1xqR9EPNZNzgd4QHK4of1lqsNR
      uz2SxkwqLcXSw0mGcAL8mIwVpzhPzwmENC5OrwIBJQKCAQB++q2WCkCmbtByyrAp
      6ktiukjTL6MGGGhjX/PgYA5IvINX1SvtU0NZnb7FAntiSz7GFrODQyFPQ0jL3bq0
      MrwzRDA6x+cPzMb/7RvBEIGdadfFjbAVaMqfAsul5SpBokKFLxU6lDb2CMdhS67c
      1K2Hv0qKLpHL0vAdEZQ2nFAMWETvVMzl0o1dQmyGzA0GTY8VYdCRsUbwNgvFMvBj
      8T/svzjpASDifa7IXlGaLrXfCH584zt7y+qjJ05O1G0NFslQ9n2wi7F93N8rHxgl
      JDE4OhfyaDyLL1UdBlBpjYPSUbX7D5NExLggWEVFEwx4JRaK6+aDdFDKbSBIidHf
      h45NAoGBANjANRKLBtcxmW4foK5ILTuFkOaowqj+2AIgT1ezCVpErHDFg0bkuvDk
      QVdsAJRX5//luSO30dI0OWWGjgmIUXD7iej0sjAPJjRAv8ai+MYyaLfkdqv1Oj5c
      oDC3KjmSdXTuWSYNvarsW+Uf2v7zlZlWesTnpV6gkZH3tX86iuiZAoGBAKM0mKX0
      EjFkJH65Ym7gIED2CUyuFqq4WsCUD2RakpYZyIBKZGr8MRni3I4z6Hqm+rxVW6Dj
      uFGQe5GhgPvO23UG1Y6nm0VkYgZq81TraZc/oMzignSC95w7OsLaLn6qp32Fje1M
      Ez2Yn0T3dDcu1twY8OoDuvWx5LFMJ3NoRJaHAoGBAJ4rZP+xj17DVElxBo0EPK7k
      7TKygDYhwDjnJSRSN0HfFg0agmQqXucjGuzEbyAkeN1Um9vLU+xrTHqEyIN/Jqxk
      hztKxzfTtBhK7M84p7M5iq+0jfMau8ykdOVHZAB/odHeXLrnbrr/gVQsAKw1NdDC
      kPCNXP/c9JrzB+c4juEVAoGBAJGPxmp/vTL4c5OebIxnCAKWP6VBUnyWliFhdYME
      rECvNkjoZ2ZWjKhijVw8Il+OAjlFNgwJXzP9Z0qJIAMuHa2QeUfhmFKlo4ku9LOF
      2rdUbNJpKD5m+IRsLX1az4W6zLwPVRHp56WjzFJEfGiRjzMBfOxkMSBSjbLjDm3Z
      iUf7AoGBALjvtjapDwlEa5/CFvzOVGFq4L/OJTBEBGx/SA4HUc3TFTtlY2hvTDPZ
      dQr/JBzLBUjCOBVuUuH3uW7hGhW+DnlzrfbfJATaRR8Ht6VU651T+Gbrr8EqNpCP
      gmznERCNf9Kaxl/hlyV5dZBe/2LIK+/jLGNu9EJLoraaCBFshJKF
      -----END RSA PRIVATE KEY-----

#. 当Blueprint 并保存好后，点击 **Save** 并点击 **Back** 。

   几分钟之内，Carol奠定了直接向最终用户提供虚拟基础架构和应用程序自助服务的基础。

开发人员工作流程
++++++++++++++++++

Dan是Fiesta工程团队的成员。 他落后于测试一项新功能，因为他要求IT部署执行测试所需的虚拟基础架构的请求已过期了几天。

Dan诉诸于在他最喜欢的公共云服务上将公司VM之外的VM部署在不受安全监督的情况下，并使公司IP处于危险之中。

卡洛（Carol）鼓励丹（Dan）进行以下练习，以使他能够通过Prism在Fiesta项目中轻松部署资源。

#. 登出本地 **admin** 账号并用Dan的账号登录 **Prism Central** :

   - **User Name** - devuser01@ntnxlab.local
   - **Password** - nutanix/4u

   .. note::

      如果登录缓慢，请尝试使用隐身/私密浏览会话登录。

#. 选择 :fa:`bars` 菜单并注意你现在在环境中的权利受限状态。

#. 在 **VMs** 页面, 你应该已经看见你的 *Initials*\ **-WinToolsVM** 可以被Dan管理。

#. 单击VM，注意Dan可以获取与其VM相关的基本指标，并控制VM的配置，电源操作，甚至删除VM。

   .. figure:: images/29.png

   自助创建VM可以遵循两个工作流程：传统VM创建向导和Calm。 Dan的要求之一是一台Linux虚拟机，该虚拟机必须运行其开发工作流程中所需的多种工具。

#. 点击 **Create VM** 并填写以下字段来配置传统虚拟机，类似于Carol在实验中先前遵循的手动VM部署过程：

   - **Create VM from** - Disk Images
   - **Select Disk Images** - Linux_ToolsVM.qcow2
   - **Name** - *Initials* -LinuxToolsVM
   - **Target Project** - *Initials* -FiestaProject
   - **Network** - Secondary
   - **Categories** - Envrionment:Dev
   - 选择 **Manually configure CPU and Memory for this VM**
   - **CPU** - 2
   - **Cores Per CPU** - 1
   - **Memory** - 4 GiB

#. 点击 **Save** 并注意立即开机VM并完成如下操作。 

   Carol除了VM工具外，Dan还希望部署可用于测试Fiesta应用程序新版本的基础架构。 让最终用户通过单VM调配和手动集成来部署多层应用程序是缓慢，不一致的，并且不会带来很高的用户满意度-幸运的是，我们可以利用Carol预先为我们的项目准备的Fiesta蓝图。

#. 选择 :fa:`bars` **> Services > Calm**.

#. 从左手目录里选择 **Blueprints** 并打开 **Fiesta-Multi** 蓝图。

   .. figure:: images/30.png

   .. note::

      如果你对 Calm Blueprints不熟悉, 花一点时间去观察 **Fiesta-Multi** 蓝图:

      - 选择 **NodeReact** 或 **MySQL** 任意一个服务并检查屏幕右边的配置面板中的 **VM** 配置。

         .. figure:: images/31.png

      - 选择 **Package** tab 并点击 **Configure Install** 查看选中服务的安装任务。这些脚本和动作都是和一个服务或VM的配置相关的。

         .. figure:: images/32.png

      - 在 **Application Profile**下面, 选择 **AHV** 并观察蓝图中定义的变量。Variables 变量允许运行时自定义，也可以在每个应用程序配置文件的基础上使用变量来构建单个应用程序蓝图，该蓝图可让您将应用程序配置到多个环境，包括AHV，ESXi，AWS，GCP和Azure。 

         .. figure:: images/33.png

      - 在 **Application Profile** 下面选择**Create** 查看服务之间的依赖关系。 依赖关系可以显式定义，但是根据变量的分配，Calm还将标识隐式依赖关系。在此蓝图中，您将看到直到MySQL数据库运行，Web层安装过程才会开始。

         .. figure:: images/34.png

      - 选择 **Credentials** in the toolbar at the top of the Blueprint Editor and expand the existing **CENTOS** credential. Blueprints can contain multiple credentials which can be used to authenticate to VMs to execute scripts, or securely pass credentials directly into scripts.

         .. figure:: images/35.png

      - 点击 **Back**.

#. 点击 **Launch** 部署一个 Blueprint实例。

   .. figure:: images/36.png

#. Fill out the following fields and click **Create**:

   - **Name of of the Application** - *Initials* -FiestaMySQL
   - **db_password** - nutanix/4u

   .. figure:: images/37.png

#. Select the **Audit** tab to monitor the deployment of the Fiesta development environment. Complete provisioning of the app should take approximately 5 minutes.

   .. figure:: images/38.png

#. While the application is provisioning, open :fa:`bars` **> Administration > Projects** and select your project.

#. Review the **Summary**, **Usage**, **VMs**, and **Users** tabs to see what type of data is made available to users. These breakouts make it easy to understand on a per project, vm, or user level, what resources are being consumed.

   .. figure:: images/39.png

#. Return to **Calm > Applications >** *Initials*\ **-FiestaMySQL** and wait for the application to move from **Provisioning** to **Running**. Select the **Services** tab and select the **NodeReact** Service to obtain the IP of the web tier.

   .. figure:: images/40.png

#. Open \http://<*NodeReact-VM-IP*> in a new browser tab and validate the app is running.

   .. figure:: images/41.png

   Instead of filing tickets and waiting days, Dan was able to get his test environment up and running before lunch. Instead of drowning his sorrows in Ben & Jerry's tonight, Dan is going to go to the gym, and eat vegetables with his dinner. Go, Dan!

Operator Workflows
++++++++++++++++++

Meet Ronald and Elise. Ronald works as a Level 3 engineering with the corporate IT helpdesk, and Elise works as a QA intern on the Fiesta team. In the brief exercise below you will explore and contrast their levels of access based on the roles defined and categories assigned by Carol.

#. Log out of the **devuser01** account and log back into **Prism Central** with Ronald's credentials:

   - **User Name** - operator01@ntnxlab.local
   - **Password** - nutanix/4u

#. As expected, all VMs with **Environment** category values assigned are available. Note that you have no ability to **Create** or **Delete** VMs, but the abilities to power manage and change VM configurations are present.

   What else can be accessed by this user? Is Calm available?

   .. figure:: images/42.png

#. Log out of the **operator01** account and log back into **Prism Central** with Elise's credentials:

   - **User Name** - operator02@ntnxlab.local
   - **Password** - nutanix/4u

#. Note that only resources tagged with the *Initials*\ **-Team: Fiesta** category are available to be managed.

   .. figure:: images/43.png

#. Elise receives an alert that memory utilization is high on the **nodereact** VM. Update the configuration to increase memory and power cycle the VM.

Using Entity Browser, Search, and Analysis
++++++++++++++++++++++++++++++++++++++++++

Now that Carol has freed up time to focus on replacing additional legacy infrastructure, it is important for her to understand how a large, diverse environment can all be managed and monitored via Prism Central. In the exercise below you will explore common workflows for working with entities across multiple clusters in a Nutanix environment.

#. Log out of the **operator02** account and log back into **Prism Central** with Carol's AD credentials:

   - **User Name** - adminuser01@ntnxlab.local
   - **Password** - nutanix/4u

#. Open :fa:`bars` **> Virtual Infrastructure > VMs**. Prism Central's **Entity Browser** provides a robust UI for sorting, searching, and viewing entities such as VMs, Images, Clusters, Hosts, Alerts, and more!

#. Select **Filters** and explore the available options. Specify the following example filters, and verify the corresponding box is checked:

   - **Name** - Contains *Initials*
   - **Categories** - *Initials*\ -Team: Fiesta
   - **Hypervisor** - AHV
   - **Power State** - On

   Take notice of other helpful filters available such as VM efficiency, memory usage, and storage latency.

#. Select all of the filtered VMs and click the **Label** icon to apply a custom label to your group of filtered VMs (e.g. *Initials* AHV Fiesta VMs).

   .. figure:: images/44.png

#. Clear all filters and select your new label to quickly return to your previously identified VMs. Labels provide an additional means of taxonomy for entities, without tying them to specific policies as is with categories.

   .. figure:: images/45.png

#. Select the **Focus** dropdown to access different out of box views. Which view should be used to understand if your VMs are included as part of a DR plan?

#. Click **Focus > + Add Custom** to create a VM view (e.g. *XYZ-VM-View*) that displays **CPU Usage**, **CPU Ready Time**, **IO Latency**, **Working Set Size Read**, and **Working Set Size Write**. Such a view could be used to helping to spot VM performance problems.

   .. figure:: images/46.png

#. To fully appreciate the power of Prism Central for searching, sorting, and analyzing entities, view the following brief video:

   .. raw:: html

     <center><iframe width="640" height="360" src="https://www.youtube.com/embed/HXWCExTlXm4?rel=0&amp;showinfo=0" frameborder="0" allow="accelerometer; autoplay; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe></center>

提升生命周期管理（LCM）
++++++++++++++++++++++++++++++

尽管不是日常活动，但Carol以前将多达40％的时间用于规划和执行遗留基础架构的软件和固件更新，因此很少有时间进行创新。 在她的Nutanix环境中，Carol利用Lifecycle Manager（LCM）中的规则引擎和丰富的自动化功能，摆脱了计划和应用基础架构软件更新的麻烦。

虽然在共享集群环境中，您无法直接测试LCM。 要更熟悉LCM的功能和易用性，请单击下面提供的每个交互式演示。

5.11 Prism Element LCM Interactive Demo
=======================================

.. figure:: https://demo-captures.s3-us-west-1.amazonaws.com/pe-5.11-lcm/story_content/thumbnail.jpg
   :target: https://demo-captures.s3-us-west-1.amazonaws.com/pe-5.11-lcm/story.html
   :alt: Prism Element 5.11 LCM Interactive Demo

5.11 Prism Central LCM Interactive Demo
=======================================

.. figure:: https://demo-captures.s3-us-west-1.amazonaws.com/pc-5.11-lcm/story_content/thumbnail.jpg
   :target: https://demo-captures.s3-us-west-1.amazonaws.com/pc-5.11-lcm/story.html
   :alt: Prism Central 5.11 LCM Interactive Demo

下一步
++++++++++

在不到2小时的时间内，我们向您展示了Prism如何为虚拟基础架构管理员提供无摩擦的体验，涉及部署存储，网络和工作负载，监视环境以及更新软件。 您已经了解了如何将本机Prism Central功能与Active Directory结合使用，以控制访问权限并为非管理员角色启用自助服务。 此外，您还通过Nutanix Calm为私有云启用了丰富的应用程序自动化功能。

但是，私有云并非仅建立在IaaS，自助服务和应用程序自动化之上。 在接下来的实验室中，您将看到Nutanix如何在其基础上通过其附加的 **Prism Pro** 功能提供先进的监视和操作功能，通过 **Files**合并存储技术，通过 **Flow** 进行本机微分段。 还有更多！
