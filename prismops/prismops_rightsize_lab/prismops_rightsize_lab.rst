-------------------------------
使用Prism Ops调整VM大小
-------------------------------

.. figure:: images/operationstriangle.png

Prism Ops为客户的日常IT操作带来了智能自动化。 典型的操作工作流程是连续的监视，分析和必要时采取行动的周期。 Prism Ops反映了传统IT Admin的工作流程，以提高运营效率。 借助Prism Ops，IT管理员可以使用机器学习引擎X-FIT和X-Play自动化引擎的强大功能，将机器数据中的洞察力连接起来，以自动化该典型流程。

在本实验中，您将学习Prism Ops如何帮助IT管理员在VM的内存资源受限时监控，分析并自动采取措施。

实验设置
+++++++++

＃. 打开您的 **Prism Central** 并导航至 **VMs** 页面。 记下 **PrismOpsLabUtilityServer** 的IP地址。 在整个实验过程中，您将需要访问此IP地址。

   .. figure:: images/init1.png

#. 在浏览器中打开一个新标签，然后浏览至 http://`<PrismOpsLabUtilityServer_IP_ADDRESS>`/alerts  [示例 http://10.38.17.12/alerts]。 如果您是第一个使用该虚拟机的人，则可能需要登录该虚拟机。 在这种情况下，请填写 **Prism Central IP** （从分配电子表格中，而不是我们刚刚记录的IP），**Username** 和 **Password**，然后单击 **Login**。

   .. figure:: images/init2.png

#. 进入警报页面后，请保持此选项卡处于打开状态。 它将在本实验的后续部分中使用。

   .. figure:: images/init2b.png

#. 在新标签中，导航至 http://`<PrismOpsLabUtilityServer_IP_ADDRESS>`/ 从 [示例 http://10.38.17.12/]完成实验。 除非另有明确说明，否则使用此选项卡可以完成实验。

   .. figure:: images/init3.png

Prism Ops X-FIT的低效率检测
+++++++++++++++++++++++++++++++++++++++++++

Prism Ops使用X-FIT机器学习来检测和监视在托管集群中运行的VM的行为。

使用机器学习Prism Ops会分析数据，并对效率低下的VM进行分类。 以下是不同分类的简要说明:

  * **Overprovisioned:** 标识为使用最少数量的已分配资源的虚拟机。
  * **Inactive:** 已关机一段时间或正在运行不消耗任何CPU、内存或I/O资源的虚拟机。
  * **Constrained:** 增加资源可以提高性能的vm。
  * **Bully:** 该虚拟机被标识为使用了大量资源并影响其他虚拟机。

#. 在 **Prism Central** 界面,选择 :fa:`bars` **> Dashboard** (如果还没有)。

#. 在仪表板上，查看 **VM Efficiency** 小部件。 该小部件提供了Prism Ops的X-FIT机器学习在您的环境中检测到的低效VM的摘要。 单击小部件底部的 **View All Inefficeint VMs** 链接，以进行更仔细的查看。

   .. figure:: images/ppro_58.png

#. 现在，您将在VM列表视图中查看Efficiency聚集视图，并详细了解为何Prism Pro标记了这些VM。 您可以将鼠标悬停在Efficiency detail列中的文本以查看完整的描述。

   .. figure:: images/ppro_59.png

#. 一旦管理员检查了效率列表上的VM列表，他们就可以决定对任何VM采取操作。资源过多或过少的虚拟机将需要重新调整单个虚拟机的大小。这可以使用下面列出的各种方法中的任何一种来实现:

   * **Manually:** 管理员通过Prism（或ESXi的vCenter）编辑VM配置并更改分配的资源。
   * **X-Play:** 使用X-Play的自动Playbook，通过触发器或管理员的指示自动调整VM的大小。 在本实验的后面，将有一个实验的例子。
   * **Automation:** 使用其他一些自动化方法，例如powershell或REST-API来调整VM的大小。


   通过使用这种机器学习数据，Prism Ops还能够为VM、主机和集群度量数据生成基准或预期范围。X-FIT算法学习这些实体的正常行为，并将其表示为不同图表上的基线范围。当一个度量值偏离这个预期范围时，Prism Ops就会产生异常。

#. 现在让我们通过搜索并选择 ‘bootcamp_good_1’ 来查看VM。

   .. figure:: images/ppro_61.png

#. 单击 Metrics> CPU Usage。 请注意，一条深蓝色的线及其周围的浅蓝色区域。 深蓝色线是CPU使用率。 浅蓝色区域是此VM的预期CPU使用率范围。 该特定的VM运行的应用程序每天在同一时间进行升级，这说明了使用模式。 请注意，X-FIT检测到此使用模式，并已自动调整了预期范围。 在这种情况下，此虚拟机已出现异常，因为CPU使用率远远超出预期范围。 您也可以将时间范围缩短为 *Last 24 hours* ，以更详细地检查图表。

   .. figure:: images/ppro_60.png

#. 点击右上角的 “Alert Setting” 以设置警报策略。

#. 在右侧，您可以选择更改某些配置。 在下面的示例中，在 *Behavioral Anomaly* 下，*Every time there is an anomaly* 设置为警告，并将 *Ignore anomalies between* 设置为10％和70％。 在Static threshold下，将 *Alert Critical if* 配置为> 95％。

   .. figure:: images/ppro_25.png

#. 点击 “Cancel” 退出策略创建工作流程。

使用X-Play自动增加受限的VM内存
++++++++++++++++++++++++++++++++++++++++++++++++++++++++

现在让我们看一下如何采取自动化措施来解决其中的一些低效率问题。 在本实验中，我们将假定此VM受内存限制，并说明如何自动修复此VM的正确资源配置。 我们还将使用自定义工单系统来说明如何将这种典型工作流程与工单系统（例如ServiceNow）集成。

#. 导航至您的 **`姓名缩写`-LinuxToolsVM**。 这些示例将使用名为 **ABC-VM** 的虚拟机。

   .. figure:: images/rs1.png

#. 注意虚拟机的当前 **Memory Capacity**，我们稍后将通过X-Play增加它。 与以下示例相比，窗口小部件的位置可能与您有所不同。 另外，您可能需要在 **Properties** 窗口内向下滚动才能找到此值。

   .. figure:: images/rs2.png

#. 使用汉堡包菜单，导航至 **Operations** > **Playbooks**.

   .. figure:: images/rs3.png

#. 我们将需要创建几个Playbook，以实现此工作流程。 让我们先点击 **Create Playbook** 。 我们将首先创建Playbook，该Playbook将增加VM的内存。

   .. figure:: images/rs3b.png

#. 选择“ Webhook”作为触发器。 使用此触发器将公开一个公共API，该API允许脚本和第三方工具（例如ServiceNow）使用此Webhook回调Prism Central并触发此Playbook。 在我们的情况下，工单系统将调用此Playbook来启动修复步骤。

   .. figure:: images/rs16.png

#. 点击左侧的 **Add Action** 项。

   .. figure:: images/rs17.png

#. 接下来，我们要选择 **VM Add Memory** 操作。

   .. figure:: images/rs18.png

#. 使用 **Parameters** 链接来填充从Webhook触发器公开的 **entity1** 参数。 调用方将传入VM以充当entity1。 根据以下屏幕设置其余字段。 然后点击 **Add Action** 以添加下一个操作。
   .. figure:: images/rs19.png

#. Select the **Resolve Alert** action.

   .. figure:: images/rs19b.png

#. 使用 **Parameters** 链接来填充从Webhook触发器公开的 **entity2** 参数。 调用方将传递警报以将其解析为entity2。 然后单击 **Add Action** ，然后选择Email操作。

   .. figure:: images/rs19c.png

#. 填写电子邮件操作中的字段。 这里是例子。

   - **Recipient:** - Fill in your email address.
   - **Subject:** - ``Playbook {{playbook.playbook_name}} was executed.``
   - **Message:** - ``{{playbook.playbook_name}} has run and has added 1GiB of Memory to the VM {{trigger[0].entity1.name}}.``

   .. note::

     欢迎您撰写您自己的主题信息。 以上仅是示例。 您可以使用 “parameters” 来丰富消息。

   .. figure:: images/rs20.png
#. 最后，我们想返回工单服务以解决工单服务中的工单。 单击 **Add Action** 以添加REST API操作。 填写以下值，替换URL字段中的 <PrismOpsLabUtilityServer_IP_ADDRESS> 。

   - **Method:** PUT
   - **URL:** http://<PrismOpsLabUtilityServer_IP_ADDRESS>/resolve_ticket
   - **Request Body:** ``{"incident_id":"{{trigger[0].entity1.uuid}}"}``
   - **Request Header:** Content-Type:application/json;charset=utf-8

   .. figure:: images/rs21.png
#. 单击 **Save & Close** 按钮，并将其保存为名称 “*姓名缩写* - Resolve Service Ticket”。 **请确保启用 ‘Enabled’ 选项。**

   .. figure:: images/rs22.png
#. 接下来，我们将创建一个自定义动作以在我们的第二本Playbook中使用。 点击左侧菜单中的 **Action Gallery** 。

   .. figure:: images/rs3c.png

#. 选择 **REST API** 操作，然后从操作菜单中选择 **Clone** 操作。
   .. figure:: images/rs4.png

#. 填写以下值，替换 *姓名缩写* 部分，并在URL字段中输入<PrismOpsLabUtilityServer_IP_ADDRESS>。 点击 **Copy** 。

   - **Name:** *Initials* - Generate Service Ticket
   - **Method:** POST
   - **URL:** http://<PrismOpsLabUtilityServer_IP_ADDRESS>/generate_ticket/
   - **Request Body:** ``{"vm_name":"{{trigger[0].source_entity_info.name}}","vm_id":"{{trigger[0].source_entity_info.uuid}}","alert_name":"{{trigger[0].alert_entity_info.name}}","alert_id":"{{trigger[0].alert_entity_info.uuid}}", "webhook_id":"<ENTER_ID_HERE>","string1":"Request 1GiB memory increase."}``
   - **Request Header:** Content-Type:application/json;charset=utf-8

   .. figure:: images/rs5.png
#. 现在，通过点击左侧菜单中的 **List** 项，切换到Playbooks列表。

   .. figure:: images/rs6.png

#. 我们将需要从创建的第一本Playbook复制Webhook ID，以便可以在generate ticket 步骤中传递它。 打开您的Resolve Service Ticket Playbook，然后将Webhook ID复制到剪贴板。

   .. figure:: images/rs6a.png


#. 现在，我们将创建一个Playbook，以自动生成服务工单。 点击表格视图顶部的 **Create Playbook** 。

   .. figure:: images/rs7.png

#. 选择 **Alert** 作为触发器

   .. figure:: images/rs8.png

#. 搜索并选择 **VM {vm_name} Memory Constrained** 作为警报策略。

   .. figure:: images/rs9.png

#. 选择 *Specify VMs* 单选按钮，然后选择 **_ 姓名缩写_-LinuxToolsVM** 。 这样一来，只有在您的VM上发出的警报才会触发此Playbook。

   .. figure:: images/rs10.png

#. 首先，我们要为此警报生成工单。 点击左侧的 **Add Action** ，然后选择您创建的 **Generate Service Ticket** 操作。 请注意，您创建的 **Generate Service Ticket** 操作中的详细信息会自动为您填写。 继续，并用复制到剪贴板的Webhook ID替换 ** <ENTER_ID_HERE> ** 文本。

   .. figure:: images/rs11.png

#. 接下来，我们想通知管理员该工单是由X-Play创建的。 点击 **Add Action** ，然后选择Email操作。 填写Email操作中的字段。 这里是例子。 确保将消息中的<PrismOpsLabUtilityServer_IP_ADDRESS>替换为其IP地址。

   - **Recipient:** - Fill in your email address.
   - **Subject :** - ``Service Ticket Pending Approval: {{trigger[0].alert_entity_info.name}}``
   - **Message:** - ``The alert {{trigger[0].alert_entity_info.name}} triggered Playbook {{playbook.playbook_name}} and has generated a Service ticket for the VM: {{trigger[0].source_entity_info.name}} which is now pending your approval. A ticket has been generated for you to take action on at http://<PrismOpsLabUtilityServer_IP_ADDRESS>/ticketsystem``

   .. figure:: images/rs13.png

#. 单击 **Save & Close** 按钮，并使用名称  **_姓名缩写_ - Generate Service Ticket for Constrained VM**。 **请确保单击选中 ‘Enabled’ 键。**

   .. figure:: images/rs14.png

#. 现在让我们触发工作流程。 使用 **/alerts** URL [例如 10.38.17.12/alerts] 导航到在设置中打开的标签。 选择 **VM Memory Constrained** ，然后输入您的VM。 单击 **Simulate Alert** 按钮。 这将在您的VM上模拟内存受限警报。

   .. figure:: images/rs23.png

#. 你应该收到一封电子邮件到你在第一个Playbook中写下的电子邮件地址。可能需要5分钟。

   .. figure:: images/rs24.png

#. 在电子邮件中，单击链接以访问工单系统。 或者，您可以通过从浏览器的新选项卡导航到 http://`<PrismOpsLabUtilityServer_IP_ADDRESS>`/ticketsystem 来直接访问工单系统。

   .. figure:: images/rs25.png

#. 确定为您的VM创建的工单，然后单击垂直点图标以显示操作菜单。 点击 **Trigger Remediation** 选项。 这将调用REST API中传递的Webhook来生成服务工单，这将触发Resolve Service Ticket Playbook。 它将传递触发工作流程的VM和Alert的信息。

   .. figure:: images/rs26.png

#.  打开Prism Central控制台，切换回上一个选项卡。 打开 **`姓名缩写` - Resolve Service Ticket** Playbook的详细信息，然后单击视图顶部的 **Plays** 选项卡以查看为此操作执行的Playbook。 单击表格中 **Plays** 的标题以进行仔细查看。

   .. figure:: images/rs29.png

#. 可以展开此视图中的节，以显示每个项的更多详细信息。如果有任何错误，它们也会出现在这个视图中。

   .. figure:: images/rs30.png

#. 您可以导航回您的VM，并确认内存确实增加了1 GiB。

   .. figure:: images/rs31.png

#. 您还应该收到一封电子邮件，告诉您playbook已经执行。

   .. figure:: images/rs32.png

重点回顾
.........

- Prism Ops是我们使IT OPS更加智能和自动化的解决方案。 它涵盖了IT OPS流程，从智能检测到自动修复。

- X-FIT是我们的机器学习引擎，可支持智能IT OPS，包括异常检测和效率低下检测。

- X-Play - 企业的IFTTT-是我们实现日常操作任务自动化的引擎。

- X-Play使管理员可以在数分钟内自信地自动化其日常任务。

- X-Play是可扩展的，可以使用客户现有的api和脚本作为其Playbook的一部分，与他们现有的工单工作流程集成。
