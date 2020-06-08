.. _linux_tools_vm:

---------------
Linux Tools VM
---------------

简介
+++++++++

此CentOS VM映像将与用于支持多个实验室练习的软件包一起暂存。

如果指示这样做，请在 **Lab Setup** 中将此虚拟机部署到您分配的群集上。

.. raw:: html

  <strong><font color="red">Only deploy the VM once, it does not need to be cleaned up as part of any lab completion.</font></strong>

部署 CentOS
++++++++++++++++

在 **Prism Central** > 选择 :fa:`bars` **> Virtual Infrastructure > VMs**, 并点击 **Create VM**.

填写以下字段:

- **Name** - *Initials*-Linux-ToolsVM
- **Description** - (Optional) Description for your VM.
- **vCPU(s)** - 1
- **Number of Cores per vCPU** - 2
- **Memory** - 2 GiB

- 选择 **+ Add New Disk**
    - **Type** - DISK
    - **Operation** - Clone from Image Service
    - **Image** - CentOS7.qcow2
    - 选择 **Add**

.. -------------------------------------------------------------------------------------
.. The Below as soon as 5.11 is GA and we want to run that version for our workshops!!!!

.. - **Boot Configuration**
 ..  - Leave the default selected **Legacy Boot**

   .. .. note::
   ..  At the following URL you can find the supported Operating Systems
   ..  http://my.nutanix.com/uefi_boot_support

.. -------------------------------------------------------------------------------------


- 选择 **Add New NIC**
    - **VLAN Name** - Secondary
    - 选择 **Add**

点击 **Save** 创建 VM.

给这个VM开机。

安装工具
++++++++++++++++

使用以下凭据通过ssh或控制台会话登录到VM：

- **Username** - root
- **password** - nutanix/4u

通过运行以下命令来安装所需的软件：

.. code-block:: bash

  yum update -y
  yum install -y ntp ntpdate unzip stress nodejs python-pip s3cmd awscli
  npm install -g request
  npm install -g express


配置 NTP
...............

通过运行以下命令来启用和配置NTP:

.. code-block:: bash

  systemctl start ntpd
  systemctl enable ntpd
  ntpdate -u -s 0.pool.ntp.org 1.pool.ntp.org 2.pool.ntp.org 3.pool.ntp.org
  systemctl restart ntpd

关闭防火墙和SELinux
..............................

通过运行如下命令关闭防火墙和SELinux:

.. code-block:: bash

  systemctl disable firewalld
  systemctl stop firewalld
  setenforce 0
  sed -i 's/enforcing/disabled/g' /etc/selinux/config /etc/selinux/config

安装 Python
.................

通过如下命令行运行Python:

.. code-block:: bash

  yum -y install python36
  python3.6 -m ensurepip
  yum -y install python36-setuptools
  pip install -U pip
  pip install boto3
