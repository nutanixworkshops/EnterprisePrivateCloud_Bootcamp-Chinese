.. _pcflow_secure_fiesta:

-------------------------------
使用Flow保护应用程序
-------------------------------

     Flow是一款以应用程序为中心的网络安全产品，已紧密集成到Nutanix AHV和Prism Central中。Flow为在AHV上运行的VM提供了丰富的网络流量可视化、自动化和安全的功能。

     微分段是Flow的一种特性，它使用基于策略的简单管理的方式来保护虚拟化环境中的网络安全。使用Prism Central的“类别（逻辑组）”功能，可以创建功能强大的分布式防火墙。它还可以与Calm结合使用，可在创建虚拟机或在虚拟机环境中部署应用的时候，就可以自动为需要保护的应用程序提供可靠的安全保护策略。

**在本练习中，您将限制对Fiesta应用程序之间的访问，并保护应用程序之间的通信流量。**

保护Fiesta应用程序
+++++++++++++++++++++++

Flow提供了多种开箱即用的“类别”，例如AppType，AppTier和Environment，可用于快速对虚拟机进行分组。我们使用这些类别，应用于安全策略。您可以立即使用这些预设的类别，也可以添加自己的类别，以进行自定义分组。

定义类别值
........................

Prism Central使用类别作为元数据来标记VM，以确定如何应用策略。

#. 在 **Prism Central** 中, 选择 :fa:`bars` **> Virtual Infrastructure > Categories**.

#. 选中 **AppType** 并点击 **Actions > Update**.

   .. figure:: images/12.png

#. 点击最后一个值旁边的 :fa:`plus-circle` 图标以添加其他类别的值。

#. 将 *Initials*-**Fiesta**  指定为值名称。

   .. figure:: images/13.png

#. 点击 **Save**.

#. 选择 **AppTier** 并点击 **Actions > Update**.

#. 点击最后一个值旁边的 :fa:`plus-circle` 图标以添加其他类别值。

#. 指定 *Initials*-**Web**  作为值名称。此类别将应用于应用程序的Web层。

#. 点击 :fa:`plus-circle` 并指定Initials - DB。此类别将应用于应用程序的MySQL数据库层。

   .. figure:: images/14.png

#. 点击 **Save**.

创建安全策略
..........................

     Nutanix Flow拥有一个策略驱动的安全框架，该框架使用以工作负载为中心的方法，而不是以网络为中心的方法。因此，无论网络配置如何更改，以及VM在数据中心中的位置在哪里，Flow都可以检查往返VM的流量。以工作负载为中心，与网络无关的策略，使得虚拟化团队能够自己部署这些安全策略，而不必依赖网络安全团队。

     安全策略基于类别，而不基于VM本身。因此，在给定类别中添加多少个VM无关紧要。

     创建将保护Fiesta应用程序的安全策略。

#. 在 **Prism Central** 中, 选择 :fa:`bars` **> Policies > Security Policies**.

#. 点击 **Create Security Policy > Secure Applications (App Policy) > Create**.

#. 填写以下字段：

   - **Name** - *Initials*-Fiesta
   - **Purpose** - Restrict unnecessary access to Fiesta
   - **Secure this app** - AppType: *Initials*-Fiesta
   - 千万 **不要** 选择 **Filter the app type by category**.

   .. figure:: images/18.png

#. 点击 **Next**

#. 如果出现提示，就在 **Create App Security Policy** 向导的教程图中，选择 **OK, Got it!**

#. 为了更详细地配置安全策略，请单击 **Set rules on App Tiers, instead** 而不是将相同的规则应用于应用程序的所有组件。

   .. figure:: images/19.png

#. 点击 **+ Add Tier**.

#. 在下拉列表中选择 **AppTier:**\ *Initials*-**Web** .

#. 对 **AppTier:**\ *Initials*-**DB** 重复步骤 7-8 .

   .. figure:: images/20.png

     接下来，您将定义入站（inbound）规则，该规则控制允许与应用程序进行通信的源。您可以允许所有入站流量，或定义列入白名单的访问来源。默认情况下，安全策略设置为拒绝所有的如站流量。

     在这种情况下，我们要允许从所有客户端到TCP端口80上的Web层的入站TCP通信。

#. 在 **Inbound**, 点击 **+ Add Source**.

#. 填写以下字段以允许所有入站IP地址:

   - **Add source by:** - 选择 **Subnet/IP**
   - 指定 **0.0.0.0/0**

   .. 注意::

     来源也可以按类别指定，以增强灵活性，因为安全策略可以跟随VM移动而移动，无论VM在网络中的什么位置，安全策略都能执行。

#. 要创建入站规则，请选择在 **+** 图标，它显示在 **AppTier:**\ *Initials*-**Web** 左侧.

   .. figure:: images/21.png

#. 填写以下字段:

   - **Protocol** - TCP
   - **Ports** - 80

   .. figure:: images/22.png

   .. 注意::

     可以将多个协议和端口添加到单个规则中。

#. 点击 **Save**.

   Calm可能还需要访问VM，以自动化地部署工作流，包括横向扩展，横向扩展或升级。Calm使用TCP端口22通过SSH与这些VM进行通信。

#. 在 **Inbound**, 点击 **+ Add Source**.

#. 填写以下字段:

   - **Add source by:** - 选择 **Subnet/IP**
   - 指定 *你的 Prism Central IP*\ /32

   .. 注意::

     **/32** 表示一个IP，而不是一个网段。

   .. figure:: images/23.png

#. 点击 **Add**.

#. 选择 **+** 图标，它出现在 **AppTier:**\ *Initials*-**Web** 左侧, 指定TCP **TCP** 端口 **22** 并点击 **Save**.

#. 对 **AppTier:**\ *Initials*-**DB** 重复步骤18，以使Calm与数据库VM通信。

   .. figure:: images/24.png

   默认情况下，安全策略允许应用程序将所有出站流量发送到任何目的地。应用程序所需的唯一出站通信是与DNS服务器的通信。

#. 在 **Outbound**, 从下拉菜单中选择 **Whitelist Only** ，然后点击 **+ Add Destination**.

#. 填写以下字段:

   - **Add source by:** - 选择 **Subnet/IP**
   - 指定 *你的域控制器 IP*\ /32

   .. figure:: images/25.png

#. 点击 **Add**.

#. 选择 **+** 图标，它出现在 **AppTier:**\ *Initials*-**Web** 的右侧, 指定 **UDP** 端口 **53** 然后点击 **Save** 以允许DNS流量。 对 **AppTier:**\ *Initials*-**DB** 重复此步骤.

   .. figure:: images/26.png

   应用程序的每一层都可以与其他层进行通信，并且策略必须对该流量放行。

#. 要定义应用间通信，请点击 **Set Rules within App**.

   .. figure:: images/27.png

#. 点击 **AppTier:**\ *Initials*-**Web** 并选择 **No** 以防止该层中的VM之间进行通信。该层中只有一个Web VM。

#. 在仍旧选择 **AppTier:**\ *Initials*-**Web** 的情况下，点击 :fa:`plus-circle` 图标，它出现在 **AppTier:**\ *Initials*-**DB** 右侧，以创建层级之间的规则。

#. 填写以下字段，以允许Web和数据库层之间在TCP端口3306上进行通信：

   - **Protocol** - TCP
   - **Ports** - 3306

   .. figure:: images/28.png

#. 点击 **Save**.

#. 点击 **Next** 查看安全策略。

#. 点击 **Save and Monitor** 以保存策略。

分配类别值
.........................

现在，您将先前创建的类别应用于从Fiesta蓝图配置的VM。Flow的类别可以作为Calm蓝图的一部分进行分配，但是本练习的目的是了解对现有虚拟机的类别分配。

#. 在 **Prism Central**, 选择 :fa:`bars` **> Virtual Infrastructure > VMs**.

#. 点击 **Filters** 然后选择 *Initials AHV Fiesta VMs* 的标签，以列出您的虚拟机。

   .. figure:: images/15.png

#. 使用复选框，选择与该应用程序关联的2个VM（Web和DB），然后选择 **Actions > Manage Categories**.

   .. figure:: images/16.png

#. 在搜索栏中指定 **AppType:**\ *Initials*-**Fiesta** 然后点击 **Save** 图标将类别批量分配给所有VM。

   .. figure:: images/16a.png

#. 选择 **Actions > Manage Categories**, 指定 **AppTier:**\ *Initials*-**Web** 类别，然后单击 **Save**.

   .. figure:: images/17.png

#. 重复步骤 5 将 **AppTier:**\ *Initials*-**DB** 分配给MySQL VM。

#. 最后， 重复步骤 5 将 **Environment:Dev** 分配给Windows Tools VM。

监视并应用安全策略
+++++++++++++++++++++++++++++++++++++++++

在应用Flow策略之前，您将确保Fiesta应用程序按预期运行。

测试应用
.......................

#. 从 **Prism Central > Virtual Infrastructure > VMs**, 记录 **-nodereact...** 和 **-MYSQL-...** VM 的IP地址.

#. 启动你的 *Initials*\ **-WinToolsVM** VM控制台。

#. 在 *Initials*\ **-WinToolsVM** 控制台中，打开浏览器并访问 \http://*node-VM-IP*/.

#. 验证应用程序已加载并且可以添加和删除任务。

   .. figure:: images/30.png

#. 打开 **Command Prompt** 并运行 ``ping -t MYSQL-VM-IP`` 以验证客户端和数据库之间的连接。保持ping运行。

#. 打开第二个 **Command Prompt** 并运行 ``ping -t node-VM-IP`` 以验证客户端和Web服务器之间的连接。保持ping运行。

   .. figure:: images/31.png

使用Flow可视化
........................

#. 返回到 **Prism Central** 并选择 :fa:`bars` **> Virtual Infrastructure > Policies > Security Policies >**\ *Initials*-**Fiesta**.

#. 验证 **Environment: Dev** 显示为入站源。源以黄色显示，表示已检测到来自客户端VM的流量。

   .. figure:: images/32.png

   是否还有其他检测到的出站流量？将鼠标悬停在这些连接上，并确定正在使用的端口。

#. 点击 **Update** 以编辑策略。

   .. figure:: images/34.png

#. 点击 **Next** 等待检测到的流量。

#. 将鼠标悬停在 **Environment: Dev** 的源上，它显示连接到了 **AppTier:**\ *Initials*-**Web** 然后单击出现的 :fa:`check` 图标。

   .. figure:: images/35.png

#. 点击 **OK** 完成添加规则。

   现在 **Environment: Dev** 源应变为蓝色，表明它是策略的一部分。将鼠标悬停在流线上，并验证是否同时显示了ICMP（ping通信）和TCP端口80。

#. 点击 **Next > Save and Monitor** 以更新策略。

应用Flow策略
......................

为了执行已定义的策略，必须应用该策略。

#. 选择 *Initials*-**Fiesta**  然后点击 **Actions > Apply**.

   .. figure:: images/36.png

#. 在确认对话框中点击 **APPLY** 然后单击 **OK** 开始阻止流量。

#. 返回 *Initials*\ **-WinToolsVm** 控制台.

   从Windows客户端到数据库服务器的持续ping通信会发生什么？该流量被阻止了吗？

#. 验证Windows客户端VM仍可以使用Web浏览器和Web服务器IP地址访问Fiesta应用程序。

   你仍然可以在添加新的 **Products** ，更新的产品数量的 **Inventory** 吗?

总结
+++++++++

- 微分段技术可提供细颗粒度的保护，在数据中心内部，抵御从一台计算机横向传播到另一台计算机的恶意威胁。
- 在Calm的蓝图中，也中可以找到在Prism Central中创建的类别。
- 安全策略，利用的是Prism Central中基于文本的类别。
- Flow可以限制AHV上运行的VM的某些端口和协议上的流量。
- 该策略以 **Monitor** 模式创建，这意味着在应用该策略之前不会阻止流量。这有助于了解通信是否正常，并确保不会影响并误伤其他流量。
