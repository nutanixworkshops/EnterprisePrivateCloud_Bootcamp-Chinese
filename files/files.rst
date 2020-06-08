.. title:: Nutanix Files

--------------------------------
通过Nutanix Files 整合您的存储系统
--------------------------------


 *完成本实验预计需要45分钟*
 

传统的专有文件存储环境一直是IT部门的另一个信息孤岛，不仅导致了不必要的复杂性，也会遭受到与SAN存储中相同的扩容问题和难以持续创新的困扰。Nutanix相信在企业云环境中没有信息孤岛存在的空间。通过将文件存储作为一种应用程序，并运行于经过验证的HCI核心之上，Nutanix Files通过一致的一键式的体验，为用户提供了新的高性能，弹性扩展并具备快速创新的能力的文件服务选项。


 **在本实验中，您将通过Nutanix Files，来实现管理SMB共享和NFS导出，并使用File Analytics服务来探索文件部署的新功能** 


为了节省实验时间并最大程度实现基础架构的资源共享，讲师已经在您的群集上部署了一个Files的服务实例，名为 **BootcampFS** 。 **BootcampFS** 是一个单节点的实例，但典型的 **Nutanix Files** 部署将从3个文件服务器VM开始，并具有根据性能要求进行横向或纵向扩展的能力。

 
 **BootcampFS** 已配置为使用 **Primary** 网络用于从 **CVM** 到 **Volume Groups** 的iSCSI连接，从而实现与后端存储进行通信，并且将 **Secondary** 网络用于与客户端，Active Directory，防病毒服务等的通信。


.. figure:: images / 1.png


.. Note::

  在生产环境中部署Files时，通常需要使用专用的虚拟网络来承载客户端和存储流量，当使用两个网络时，根据设计，文件将禁止客户端网络中的虚拟机直接访问存储网络，这意味着此环境中分配给主网络的VM将无法直接访问文件共享目录。
..

由于Nutanix Files利用Nutanix Volume Group 进行数据存储，因此可以直接复用相同的存储优化算法，例如压缩，纠删码，快照和数据复制功能等。


在 **Prism Element** > **File Server** > **File Server** 中，选择 **BootcampFS** ，然后单击 **Protect** 。


.. figure:: images / 10.png


观察默认的自助服务还原计划，此功能控制着Windows文件夹属性中- **以前的版本** 功能的快照计划。支持Windows **以前的版本** 的功能，可以允许最终用户自助回滚对文件的更改，而无需依赖存储或备份管理员。请注意，这些本地快照无法保护文件服务器群集免受本地故障的影响，如果要实现此功能，可以将整个文件服务器群集复制到远程Nutanix群集中。


管理Windows SMB共享
+++++++++++++++++++

在本练习中，您将尝试创建和测试SMB共享，以支持跨部门团队对Fiesta应用程序的非结构化文件数据的存储需求。

创建共享
.....................

1. 在 **Prism Element>File Server** 中，单击 **++Share/Export**。


2. 填写以下字段：

   - **Name** - *Initials*\ **-FiestaShare**  
   - **Description (Optional)** - Fiesta app team share, used by PM, ENG, and MKT
   - **File Server** - **BootcampFS** 
   - **Share Path (Optional)** - 空白.此项将用于在已有的路径中创建嵌套共享目录
   - **Max Size (Optional)** - 200GiB
   - **Select Protocol** - SMB

.. figure:: images / 2.png


   因为此环境为单节点AOS群集，因此只有一个文件服务器VM，所以所有共享都是 **Standard** 共享。Standard共享意味着共享中的所有一级目录和文件以及与共享的连接均由单个文件服务器VM提供。

   如果这是三个节点的文件集群或更大的集群，则可以选择创建 **Distributed** 共享。Distributed共享适用于主目录，用户配置文件和应用程序文件夹。这种类型的共享在所有文件VM上进行一级目录的均匀分布，并在文件群集内的所有文件VM之间实现连接的负载均衡。


3. 点击 **下一步** 。


4. 选择 **Enable Access Based Enumeration** 和 **Self Service Restore** 。选择 **Blocked File Types** ，然后输入以逗号分隔的扩展名列表，例如.flv，.mov。

.. figure:: images / 3.png

.. Note::
     
    **Enable Access Based Enumeration** （基于访问的枚举（ABE）功能）将确保文件和文件夹只会对具有读取访问权限的文件和文件夹的用户可见，Windows文件共享通常启用此功能。
    
    **Self Service Restore** （自助服务还原）允许用户利用Windows“以前的版本”功能轻松地将单个文件还原到基于Nutanix快照的历史版本。
      
    **Blocked File Types** 允许文件管理员限制某些类型的文件（例如大的个人媒体文件）被写入公司共享。可以在每个服务器或每个共享的基础上进行配置，如果存在不一致，基于每个共享的设置将覆盖服务器范围的设置.
    
..

5. 点击 **Next** 。


6. 查看 **Summary** ，然后单击 **Create** 。


.. figure:: images / 4.png


   为了确保共享资源可能被多人公平地使用，采用配额是推荐的手段。通过Files，可以基于每个共享，为AD内的单个用户或特定的AD安全组按份额设置软配额或硬配额。


7. 在 **Prism Element > File Server > Share/Export** 中，选择您的共享，然后单击 **+ Add Quota Policy** 。

8. 填写以下字段，然后单击 **Save** ：

   - Select **Group** 
   - **User or Group** - SSP Developers
   - **Quota** - 10 GiB
   - **Enforcement Type** - Hard Limit

.. figure:: images / 9.png

9. 点击 **Save** 。


测试共享
.....................

1. 通过VM控制台以 **non-Administrator NTNXLAB** 域帐户连接到您的 *Initials* \ **-WinTools** VM：

.. Note::

      您将无法通过RDP使用这些帐户进行连接。

   - user01 - user25

   - devuser01 - devuser25

   - operator01 - operator25

   - **密码**    nutanix/4u
..

.. figure:: images / 16.png
   
.. Note:: Windows Tools VM已加入 **NTNXLAB.local** 域。您可以使用任何加入域的VM来完成以下步骤。

2. 在 **File Explorer** 中打开 ``\\ BootcampFS.ntnxlab.local \`` 

3. 在您的虚拟机 *Initials* \ **-WinTools** 桌面中打开浏览器，然后下载示例数据以填充到您的共享中：


   - **如果使用PHX群集** -http://10.42.194.11/workshop_staging/peer/SampleData_Small.zip

   - **如果使用RTP群集** -http://10.55.251.38/workshop_staging/peer/SampleData_Small.zip


4. 将zip文件的内容解压到您的文件共享中。

.. figure:: images / 5.png

   -在Files群集的部署过程中， **NTNXLAB \\ Administrator** 用户被默认指定为文件管理员，默认情况下，授予该用户对所有共享的读/写访问权限。

   -管理其他用户的访问权限与任何其他SMB共享的方式相同。


5. 在 ``\\ BootcampFS.ntnxlab.local \`` 中，右键单击 *Initials* \ **-FiestaShare> Properties** 。

6. 选择 **Security **选项卡，然后单击 **Advanced** 。

.. figure:: images / 6.png


7. 选择 **Users（BootcampFS \\ Users**，然后单击 **Remove** 。


8. 点击 **Add** 。


9. 单击 **Select a principal**，然后在 **Object Name ** 字段中指定 **Everyone** 。点击 **OK** 。


.. figure:: images / 7.png


10. 填写以下字段，然后单击 **OK** ：

      - **Type** -允许

      - **Applies to** -仅此文件夹

      -选择 **Read & execute** 

      -选择 **List folder contents** 

      -选择 **Read** 

      -选择 **Write** 

.. figure:: images / 8.png


11. 单击 **OK > OK > OK** 以保存权限更改。


   现在，所有用户都可以在 *Initials* \ **-FiestaShare** 共享中创建文件夹和文件。


12. 打开 **PowerShell** 并尝试通过执行以下命令来创建文件类型被设定为被阻止的文件：

.. code-block:: PowerShell

       New-Item \\BootcampFS\INITIALS-FiestaShare\MyFile.flv


观察到创建新文件被拒绝。


13. 返回 **Prism Element > File Server > Share/Expor** ，选择您的共享。查看 **Share Details** ， **Usage** 和 **Performance** 选项卡，以了解每个共享的高级信息，包括文件和连接的数量，一段时间内的存储利用率，延迟，吞吐量和IOPS。


.. figure:: images / 11.png


   在下一个练习中，您将看到Files如何提供有关每个文件服务器和共享使用情况的更深入的洞察分析。


File Analytics
++++++++++++++

在本练习中，您将探索Nutanix Files中集成的新的文件分析功能，包括扫描现有共享，创建异常警报以及查看审核详细信息。通过Prism Element中的自动 **一键式** 操作，几分钟即可将File Analytics作为独立的VM进行部署。为了节省实验时间，此VM已在您的环境中部署并启用。

1. 在 **Prism Element > File Server > File Server** 中，选择 **BootcampFS** ，然后单击 **File Analytics** 。

.. figure:: images / 12.png


.. Note::


      文件分析应该已经启用，但是如果出现提示，您将需要提供Files的管理员帐户，因为分析将需要能够扫描所有共享。

      - **用户名**：NTNXLAB \\ administrator

      - **密码**：nutanix/4u
..

.. figure:: images / old13.png


2. 由于这是一个共享环境，因此仪表板可能已经填充了其他用户创建的共享中的数据。要扫描新创建的共享，请单击 :fa:`gear` **>Scan File System** 。选择您的共享，然后单击 **Scan** 。


.. figure:: images / 14.png


.. Note::


      如果您的共享未显示在主页面，请给它一些时间来完成数据填充...

..

3. 关闭 **Scan File System** 窗口并刷新浏览器。

4. 您应该会看到 **Data Age** ，**File Distribution by Size** 和 **File Distribution by Type** 仪表板更新。


.. figure:: images / 15.png


5. 在您的 *Initials* \ **-WinTools** VM中，通过打开 **Sample Data** 目录下的几个文件来触发一些审计跟踪活动。


.. Note:: 如果使用OpenOffice打开文件，则可能需要完成该应用程序的一些简短向导操作。


6. 在浏览器中刷新 **Dashboard** 页面，以查看 **Top 5 Active Users**，**Top 5 Accessed Files** 和 **File Operations** 面板的更新。


.. figure:: images / 17.png


7. 要访问您的用户帐户的审计跟踪活动，请在 **Top 5 Active Users** 下单击您的用户。


.. figure:: images / 17b.png


8. 另外，您也可以从工具栏中选择 **Audit Trails**，然后搜索您的用户或指定的文件名。


.. figure:: images / 18.png


.. Note::


      您可以使用通配符进行搜索，例如 **.doc** 

 ..

9. 接下来，我们将创建规则以检测文件服务器上的异常行为。在工具栏中，点击 :fa:`gear` **> Define Anomaly Rules**。


.. figure:: images / 19.png


.. Note::


         Anomaly Rules（异常规则）是基于每个文件服务器定义的，因此以下规则可能已经由其他用户创建。

..


10. 点击 **Define Anomaly Rules** ，并使用以下设置创建规则：


      - **Events:** Delete
      - **Minimum Operation %:** 1
      - **Minimum Operation Count:** 10
      - **User:** All Users
      - **Type:** Hourly
      - **Interval:** 1

11. 在 **Actions** 下，单击 **Save** 。

12. 选择 **+ Configure new anomaly** ，并使用以下设置创建其他规则：

      - **Events**: Create
      - **Minimum Operation %**: 1
      - **Minimum Operation Count**: 10
      - **User**: All Users
      - **Type**: Hourly
      - **Interval**: 1

13. 在 **Actions** 下，单击 **Save** 。


.. figure:: images / 20.png


14. 单击 **Save** 以退出 **Define Anomaly Rules** 窗口。


15. 要测试异常警报，请返回您的 *Initials* \ **-WinTools** VM，并在您的 *Initials* \ **-FiestaShare** 共享中制作第二个样本数据副本（通过复制/粘贴）。


16. 删除原始样本数据文件夹。


.. figure:: images / 21.png


  在等待异常警报展现之前，接下来我们将创建一个权限拒绝分析。


.. Note:: 异常警报引擎每30分钟运行一次。虽然可以从File Analytics VM配置此设置，但修改此变量不在本练习的范围之内。


    1). 在 *Initials* \ **-FiestaShare** 共享中创建一个名为 *Initials* \ **-MyFolder** 的新目录。


    2). 在 *Initials* \ **-MyFolder** 目录中创建一个文本文件，并借用此机会短暂发泄一下心中的压抑和不满的情绪，并将他们写入到此文件中。将文件另存为 *Initials* \ **-file.txt** 。


.. figure:: images / 22.png


    3). 右键单击 *Initials* \ **-MyFolder>Properties** 。 选择 **Security** 选项卡，然后单击 **Advanced** 。请注意，**用户（BootcampFS \\ Users)** 缺乏 **Full Control** 权限，这意味着他们将无法删除其他用户拥有的文件。

.. figure:: images / 23.png


    4). 按住 **Shift** 键并右键单击任务栏中的 **PowerShell** 图标，然后选择以其他用户身份运行，以另一个非管理员用户帐户的身份打开PowerShell窗口。

.. figure:: images / 24.png


    5). 将目录更改为 *Initials* \ **-FiestaShare** 共享中的 * Initials * \ **-MyFolder** 。

.. code-block:: bash


           cd \\ BootcampFS.ntnxlab.local \ XYZ-FiestaShare \ XYZ-MyFolder


    6). 执行以下命令：


        .. code-block:: bash

           cat .\XYZ-file.txt
           rm .\XYZ-file.txt


.. figure:: images / 25.png


    7). 返回 **Analytics > Dashboard** ，并注意 **Permission Denials** 和 **Anomaly Alerts** 小部件已更新。


.. figure:: images / 26.png


    8). 在 **Permission Denials** 下，选择您的用户帐户以查看完整的 **Audit Trail** ，并确认可以观察到您尝试删除的特定文件的事件，以及相应的IP地址和时间戳信息都已被记录。


.. figure:: images / 27.png


    9). 从工具栏中选择 **Anomalies** ，以查看检测到的异常的概述。


.. figure:: images / 28.png


File Analytics将简单而强大的信息交给存储管理员，使他们能够对Nutanix Files环境中的使用情况和访问权限有更加清晰的了解和审核能力。

 
使用NFS导出
+++++++++++++++++

在本练习中，您将创建和测试NFSv4导出，该导出通常可用于支持集群应用程序，存储应用程序数据（例如日志记录）或存储Linux客户端通常访问的其他非结构化文件数据。

 启用NFS协议
 .....................

.. Note::

   每个文件服务器只需要执行一次NFS协议启用操作，并且在您的环境中可能已经完成。如果已启用NFS，请继续进行 `配置用户映射` 。
..

1. 在 **Prism Element > File Server** 中，选择您的文件服务器，然后单击 **Protocol Management > Directory Services** 。


.. figure:: images / 29b.png


2. 选择 **Use NFS Protocol** 和 **Unmanaged** 用户管理和身份验证，然后单击 **Update** 。


.. figure:: images / 30b.png


创建导出
.....................

1. 在 **Prism > File Server** 中，单击 **+ Share/Export** 。

2. 填写以下字段：

   - **Name** - *initials*\ -logs
   - **Description (Optional)** - File share for system logs
   - **File Server** - **BootcampFS**
   - **Share Path (Optional)** - 不填
   - **Max Size (Optional)** - 不填
   - **Select Protocol** - NFS


.. figure:: images / 24b.png

3. 点击 **Next** 。

4. 填写以下字段：

   -选择 **Enable Self Service Restore**  
       -这些快照显示为NFS客户端的.snapshot目录。
   - **Authentication** - System
   - **Default Access (For All Clients)** - No Access
   - Select **+ Add exceptions** 
   - **Clients with Read-Write Access** - *群集网络的前三个八位地址段* （例如10.38.1. \*）

.. figure:: images / 25b.png


   默认情况下，NFS导出将允许对挂载了该导出的任何主机进行读/写访问，但也可以将其限制为特定的IP或IP段。

5. 点击 **Next** 。

6. 查看 **Summary** ，然后单击 **Create** 。

测试导出
.....................


首先，您将需要准备一个CentOS VM用作NFS导出的客户端。

.. Note:: 如果您已将linux_tools_vm 在另一个实验中部署过，则可以将此VM用作NFS客户端。

1. 在 **Prism> VM> Table** 中，单击 **Create VM** 。

2. 填写以下字段：

   - **Name** - *Initials*\ -NFS-客户端
   - **Description** - CentOS VM for testing Files NFS export
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU** - 1
   - **Memory** - 2 GiB
   - Select **+ Add New Disk**
      - **Operation** - Clone from Image Service
      - **Image** - CentOS
      - Select **Add**
   - Select **Add New NIC**
      - **VLAN Name** - Secondary
      - Select **Add**

3. 点击 **Save** 。


4. 选择 *Initials* \ **-NFS-Client** VM，然后单击 **Power on** 。


5. 在Prism中记下VM的IP地址，并使用以下凭据通过SSH连接：

   - **Username** - root
   - **Password** - nutanix/4u

6. 执行以下命令：

.. code-block:: bash

       [root @ CentOS〜] #. yum install -y nfs-utils #. 这将安装NFSv4客户端

       [root @ CentOS〜] #. mkdir / filesmnt

       [root @ CentOS〜] #. mount.nfs4 BootcampFS.ntnxlab.local：/ / filesmnt /

             [root@CentOS ~]# df -kh
       Filesystem                      Size  Used Avail Use% Mounted on
       /dev/mapper/centos_centos-root  8.5G  1.7G  6.8G  20% /
       devtmpfs                        1.9G     0  1.9G   0% /dev
       tmpfs                           1.9G     0  1.9G   0% /dev/shm
       tmpfs                           1.9G   17M  1.9G   1% /run
       tmpfs                           1.9G     0  1.9G   0% /sys/fs/cgroup
       /dev/sda1                       494M  141M  353M  29% /boot
       tmpfs                           377M     0  377M   0% /run/user/0
       *initials*-Files.ntnxlab.local:/  1.0T  7.0M  1.0T   1% /afsmnt
       [root@CentOS ~]# ls -l /filesmnt/
       total 1
       drwxrwxrwx. 2 root root 2 Mar  9 18:53 xyz-logs


7. 观察 **logs** 目录已挂载在 ``/filesmnt/xyz-logs`` 中。

8. 重新启动VM，并观察到NFS导出目录不再被挂载。要保持永久挂载，通过执行以下命令将其添加到 ``/etc/fstab`` 配置文件中：

.. code-block:: bash

       echo 'BootcampFS.ntnxlab.local:/ /filesmnt nfs4' >> /etc/fstab


9. 以下命令会用随机数据填充100个2MB的文件到 ``/filesmnt/logs`` 中：

.. code-block:: bash

       mkdir /filesmnt/xyz-logs/host1
       for i in {1..100}; do dd if=/dev/urandom bs=8k count=256 of=/filesmnt/xyz-logs/host1/file$i; done


10. 返回到 **Prism > File Server > Share > logs** ，以监视性能和使用情况。

   请注意，利用率数据每10分钟更新一次。

多协议共享
+++++++++++++++++++++

Files不只可以分别提供SMB共享和NFS导出的功能-还可以支持提供对同一共享的多协议访问的功能。在下面的练习中，您将配置现有的 
 *Initials* \ **-FiestaShare** 以允许NFS访问，从而使开发人员可以将应用程序日志重定向到该位置进行存储。

配置用户映射
.......................

Nutanix Files 共享具有原生和非原生协议的概念。所有权限都使用原生协议应用。使用非原生协议的任何访问请求都需要用户或组映射到从原生协议端应用的权限。用户和组映射的映射可以通过多种方法实现，包括基于规则的映射，精确映射和默认映射。我们将首先配置默认映射：

1. 在 **Prism Element > File Server** 中，选择文件服务器，然后单击 **Protocol Management > User Mapping** 。 

2. 单击两次 **Next** ，前进至 **Default Mapping** 。

3. 在 **Default Mapping** 页面上，选择 **Deny access to NFS export** 和 **Deny access to SMB share** 作为未找到映射时的默认值。

.. figure:: images / 31.png

4. 单击 **Next > Save** 以完成默认映射。

5. 在 **Prism Element > File Server** 中，选择您的 *Initials*\ **-FiestaShare** ，然后单击 **Update** 。

6. 在 **Basics** 下，选择 **Enable multiprotocol access for NFS** ，然后单击 **Next** 。

.. figure:: images / 32.png


7. 在 **Settings > Multiprotocol Access** 下，选择 **Simultaneous access to the same files from both protocols** 。

.. figure:: images / 33.png


8. 单击 **Next > Save** 以完成更新共享设置。

测试导出
.......................
 
1. 要测试NFS导出，请通过SSH连接到 *Initials* \ **-LinuxToolsVM** VM：

   - **User Name** - root
   - **Password** - nutanix/4u

2. 执行以下命令：

.. code-block:: bash

       [root@CentOS ~]# yum install -y nfs-utils #This installs the NFSv4 client
       [root@CentOS ~]# mkdir /filesmulti
       [root@CentOS ~]# mount.nfs4 bootcampfs.ntnxlab.local:/<Initials>-FiestaShare /filesmulti
       [root@CentOS ~]# dir /filesmulti
       dir: cannot open directory /filesmulti: Permission denied
       [root@CentOS ~]#

.. note:: 安装操作区分大小写。

因为默认映射是拒绝访问，所以会出现“权限被拒绝”错误。现在，您将添加一个精确（explicit）映射，以允许非原生NFS协议用户访问。我们将需要获取用户ID（UID）来创建精确映射。

3. 执行以下命令并记下UID：

.. code-block:: bash

       [root@CentOS ~]# id
       uid=0(root) gid=0(root) groups=0(root)
       [root@CentOS ~]#

4. 在 **Prism Element > File Server** 中，选择文件服务器，然后单击 **Protocol Management > User Mapping** 。

5. 单击 **Next** 前进到 **Explicit Mapping** 。

6. 在 **One-to-onemapping list** 下，单击 **Add manually** 。

7. 填写以下字段：

   - **SMB Name** - NTNXLAB\\devuser01
   - **NFS ID** - UID from previous step (0 if root)
   - **User/Group** - User

.. figure:: images / 34.png

8. 在 **Actions** 下，单击 **Save** 。

9. 单击 **Next > Next > Save** 以完成更新映射。

10. 返回您的 *Initials*\ **-LinuxToolsVM** 的SSH会话，然后尝试再次访问共享：

.. code-block:: bash

       [root@CentOS ~]# dir /filesmulti
       Documents\ -\ Copy  Graphics\ -\ Copy  Pictures\ -\ Copy  Presentations\ -\ Copy  Recordings\ -\ Copy  Technical\ PDFs\ -\ Copy  XYZ-MyFolder
       [root@CentOS ~]#

11. 在SSH会话中，创建一个文本文件，然后确认您可以从Windows客户端访问该文件。

小贴士

+++++++++

关于 **Nutanix Files** ，您应该了解哪些关键知识？

-Files可以快速部署在现有Nutanix群集之上，从而为个人用户，主目录，部门共享，应用程序和任何其他通用文件存储需求提供SMB和NFS存储。
-Files不是单点解决方案。VM，文件，块和对象存储都可以在同一平台上使用相同的管理工具来交付，从而降低了复杂性和管理孤岛。
-通过一键式性能优化，Files可以方便地实现纵向扩展和横向扩展。
-Files Analytics可帮助您更好地了解您所在的组织如何利用数据来满足您的数据审核，最小化数据访问和法规遵从性要求。


