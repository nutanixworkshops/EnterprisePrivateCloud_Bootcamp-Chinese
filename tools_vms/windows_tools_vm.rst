.. _windows_tools_vm:

----------------
Windows Tools VM
----------------

介绍
+++++++++

这个 Windows Server 2012 R2 镜像预安装了以下其他工具, including:

- Microsoft Remote Server Administration Tools (RSAT)
- PuTTY, CyberDuck, WinSCP
- Sublime Text 3, Visual Studio Code
- OpenOffice
- Python
- pgAdmin
- Chocolatey Package Manager

在你的集群部署这个VM ，如果没有请参考 **Lab Setup** 部分。

.. raw:: html

  <strong><font color="red">仅部署一次虚拟机，不需要作为任何实验室完成工作的一部分进行清理。</font></strong>

部署 Tools VM
++++++++++++++++++

在 **Prism Central** > 选择 :fa:`bars` **> Virtual Infrastructure > VMs**，并点击 **Create VM**.

填写如下字段:

- **Name** - *Initials*-Windows-ToolsVM
- **Description** - (Optional) Description for your VM.
- **vCPU(s)** - 1
- **Number of Cores per vCPU** - 2
- **Memory** - 4 GiB

- 选择 **+ Add New Disk**
    - **Type** - DISK
    - **Operation** - Clone from Image Service
    - **Image** - ToolsVM.qcow2
    - Select **Add**

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
    - Select **Add**

点击 **Save** 创建VM。

打开VM电源。

使用以下凭据通过RDP或控制台会话登录到VM：

- **Username** - NTNXLAB\\Administrator
- **password** - nutanix/4u
