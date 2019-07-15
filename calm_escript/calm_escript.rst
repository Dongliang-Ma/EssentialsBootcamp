*Script Provided Below*.. _calm_escript:

-----------------------------------------
Calm: EScript和任务库(可选)
-----------------------------------------

概述
++++++++

在 :ref:`calm_linux` 和 :ref:`calm_win` 实验中，您探索了Calm如何利用Bash和PowerShell脚本自动化应用程序部署。虽然shell脚本既强大又通用，但它们需要部署端点VM，以便在该VM上本地复制和执行脚本。

有一些用例可以更好地直接在Calm中执行代码，例如对其他RESTful服务（如Nutanix Era，GitHub，IFTTT等）进行API调用。

为了满足这个需求，Calm提供了一个名为 `EScript <https://portal.nutanix.com/#/page/docs/details?targetId=Nutanix-Calm-Admin-Operations-Guide-v250:nuc-supported-escript-modules-functions-c.html>`_ 的脚本类型。它是Epsilon脚本的缩写(Epsilon是驱动Calm的编制引擎)，EScript是一个沙箱Python编译器。它包含许多用于脚本编制和自动化的常用模块。特别是，它将 **requests** 模块包含为 **urlreq**，用于创建外部API调用。

.. note::

  编译器是沙箱的，因为脚本 **直接** 运行在Calm引擎中，这意味着它直接运行在Prism Central中。允许用户将未知和未经审核的模块导入Prism Central是一个安全问题。

**在本实验中，您将利用EScript对现有的Web服务Prism Central VM执行API调用。 您还将利用Set Variable将API调用返回的数据设置为Calm宏，并使用Task Library了解如何在多个蓝图或项目中使用公共代码。**

Lab 设置
+++++++++

本实验假设你基本熟悉Nutanix Calm。

**它不需要将任务管理器应用程序部署为其他Calm实验的一部分。** 相反，您将创建一个新的空白蓝图，并添加一个用于向Prism Central进行身份验证的凭据。

#. 在 **Prism Central** 中, 选择 :fa:`bars` **> Services > Calm > Blueprints**.

#. 点击 **+ Create Blueprint > Multi VM/Pod Blueprint**.

   .. figure:: images/create_blueprint.png

#. 填写以下字段，然后单击 **Proceed**:

   - **Name** - *Initials*\ -EScript
   - **Description** - My First EScript Blueprint
   - **Project** - default

#. 从蓝图顶部的工具栏中，单击 **Credentials**.

#. 单击 **Credentials** :fa:`plus-circle` 并填写以下字段:

   - **Credential Name** - PC_Creds
   - **Username** - admin
   - **Secret Type** - Password
   - **Password** - techX2019!

   .. figure:: images/credentials.png

#. 单击 **Save** 然后单击 **Back**, 确保没有出现错误或警告。

使用现有的机器服务
+++++++++++++++++++++++++++++++

#. 在 **Application Overview > Services** 中, 单击 :fa:`plus-circle` 以添加新的服务。

#. 在 **VM** 标签下, 填写以下字段:

   +------------------------------+------------------+
   | **Service Name**             | PC               |
   +------------------------------+------------------+
   | **Name**                     | PrismCentral     |
   +------------------------------+------------------+
   | **Cloud**                    | Existing Machine |
   +------------------------------+------------------+
   | **Operating System**         | Linux            |
   +------------------------------+------------------+
   | **IP Address**               | localhost        |
   +------------------------------+------------------+
   | **Check log-in upon create** | **Unselect**     |
   +------------------------------+------------------+
   | **Credential**               | Leave default    |
   +------------------------------+------------------+
   | **Address**                  | Leave default    |
   +------------------------------+------------------+
   | **Connection Type**          | Leave default    |
   +------------------------------+------------------+
   | **Connection Port**          | Leave default    |
   +------------------------------+------------------+
   | **Delay**                    | Leave default    |
   +------------------------------+------------------+

   .. figure:: images/existing_machine.png

   在上面的配置中有几个有趣的新项目:

   - **Cloud** - 我们没有在Nutanix或公共云提供商上创建一个新的VM，而是选择在现有机器上实现自动化。我们只需要输入机器的IP地址，在我们的示例中是Prism Central。根据您的用例，您可以将诸如Ansible Tower或Era Server之类的东西指定为现有的机器。

   - **IP Address** - 因为我们要对Prism Central进行API调用，而Calm直接运行在Prism Central中，所以我们只是将localhost作为IP输入。如果您要对Ansible Tower或Era之类的东西进行自动化，您需要将您的Ansible Tower或Era服务器IP地址放在这个字段中，而不是localhost。IP地址也可以由变量定义。

   - **Check log-in upon create** - 这是我们第一次看到这个未选中的框。因为EScript任务直接在Calm中运行，所以不需要SSH到相关服务。相反，我们将在EScript代码中直接使用凭证对REST API调用进行身份验证。

#. 单击 **Save**, 并确保没有出现错误或警告。

RESTList自定义操作
++++++++++++++++++++++

在本练习中，我们将为应用程序创建一个自定义操作，以便对Prism Central进行REST API调用。具体来说， 它将是一个 POST /list 调用, 其中要列出的实体 (kind)(例如 应用, 主机, 群集, 角色等) 将在运行时由一个变量定义。然后输出这个调用的结果。

#. 在 **Application Overview > Application Profile** 部分, 展开 **Default** Application Profile.

   .. figure:: images/addaction.png

#. 选择 :fa:`plus-circle` 然后下一步 **Actions** 添加一个新的自定义操作。

#. 在 **Configuration Pane** 的右侧, 将操作命名为 **RESTList**, 并添加一个变量:

   - **Name** - kind
   - **Value** - apps
   - Select **Runtime**

   .. figure:: images/restlist.png

 稍后运行自定义操作时，Calm将提示用户输入。**Apps** 将是预填充的默认值，但可以在执行脚本操作之前更改。

#. 单击 **+ Task** 按键将任务添加到 **RESTList** 自定义操作。 填写以下字段：

   - **Task Name** - RuntimePost
   - **Type** - Execute
   - **Script Type** - EScript
   - **Script** - *Script Provided Below*

   .. code-block:: python

     # Set the credentials
     pc_user = '@@{PC_Creds.username}@@'
     pc_pass = '@@{PC_Creds.secret}@@'

     # Set the headers, url, and payload
     headers = {'Content-Type': 'application/json', 'Accept': 'application/json'}
     url     = "https://@@{address}@@:9440/api/nutanix/v3/@@{kind}@@/list"
     payload = {}

     # Make the request
     resp = urlreq(url, verb='POST', auth='BASIC', user=pc_user, passwd=pc_pass, params=json.dumps(payload), headers=headers)

     # If the request went through correctly, print it out.  Otherwise error out, and print the response.
     if resp.ok:
        print json.dumps(json.loads(resp.content), indent=4)
        exit(0)
     else:
        print "Post request failed", resp.content
        exit(1)

   .. figure:: images/runtime_post.png

  这项任务有一些有趣的新特点:

    注意，Calm UI中没有凭据下拉框，而是将Python变量设置为前面指定的PC_Creds用户名和密码。其他API可能不需要身份验证，或者需要提供API密钥作为URL的一部分。

   我们还看到 `urlreq <https://portal.nutanix.com/#/page/docs/details?targetId=Nutanix-Calm-Admin-Operations-Guide-v250:nuc-supported-escript-modules-functions-c.html>`_ 模块被使用，这正是我们的API调用的确切行。如果响应按预期返回，JSON响应将被格式化并显示，否则将显示相应的错误消息。

#. 单击 **Save**, 并确保没有出现错误或警告。

GetDefaultSubnet自定义操作
++++++++++++++++++++++++++++++

在这个练习中，我们将创建一个额外的自定义操作来执行一个不同的REST API调用。调用将返回这个Prism中心实例上的 **Projects** 列表。然后，我们将解析该API调用的输出，以获得为正在运行的应用程序所属的项目配置的默认子网的UUID。这个UUID将被设置为一个平静的变量，允许在蓝图的其他地方重用。然后，我们将执行另一个Rest API调用，即GET on the default子网(使用这个新设置的变量)。

#. 选择 **PC** 服务. 在 **Configuration Pane** 中, 选择 **Service** 选项卡. 添加名为 **SUBNET** 的变量, 将所有其他字段留空。
   .. figure:: images/subnet_variable.png

#. 在 **Application Overview > Application Profile > Default** 部分, 选择 :fa:`plus-circle` 旁边的 **Actions** 以添加新的自定义操作。

#. 将操作命名为 **GetDefaultSubnet**.

   .. figure:: images/get_default_subnet.png

#. 单击 **+ Task** 按键将任务添加到 **GetDefaultSubnet** 自定义操作。 填写以下字段：

   - **Task Name** - GetSubnetUUID
   - **Type** - Set Variable
   - **Script Type** - EScript
   - **Script** - *Script Provided Below*
   - **Output** - SUBNET

   .. code-block:: python

     # Get the JWT
     jwt = '@@{calm_jwt}@@'

     # Set the headers, url, and payload
     headers = {'Content-Type': 'application/json', 'Accept': 'application/json', 'Authorization': 'Bearer {}'.format(jwt)}
     url     = "https://@@{address}@@:9440/api/nutanix/v3/projects/list"
     payload = {}

     # Make the request
     resp = urlreq(url, verb='POST', params=json.dumps(payload), headers=headers, verify=False)

     # If the request went through correctly
     if resp.ok:

      # Cycle through the project "entities", and check if its name matches the current project
      for project in json.loads(resp.content)['entities']:
        if project['spec']['name'] == '@@{calm_project_name}@@':

          # If there's a default subnet reference, print UUID to set variable and exit success, otherwise error out
          if 'uuid' in project['status']['resources']['default_subnet_reference']:
            print "SUBNET={0}".format(project['status']['resources']['default_subnet_reference']['uuid'])
            exit (0)
          else:
            print "The '@@{calm_project_name}@@' project does not have a default subnet set."
            exit(1)

      # If we've reached this point in the code, none of our projects matched the calm_project_name macro
      print "The '@@{calm_project_name}@@' project does not match any of our /projects/list api call."
      print json.dumps(json.loads(resp.content), indent=4)
      exit(0)

     # In case the request returns an error
     else:
      print "Post clusters/list request failed", resp.content
      exit(1)

   .. figure:: images/get_subnet_uuid.png

  在 **RESTList** 和 **GetDefaultSubnet** 任务之间有两个关键区别。 第一个区别是使用 **Set Variable** 任务类型， 请注意 **print "SUBNET={0}"** 行: Calm将解析格式为 **variable=value** 的输出, 并将变量设置为该值。 在本例中，我们打印的变量名为 **SUBNET** 等于初始API调用响应中的 "default_subnet_reference" 字段的UUID。 在脚本主体下面的 **Output** 字段中， 我们必须粘贴变量名以便Calm适当地设置变量。该变量必须已经在Calm blueprint中定义，无论是全局的，还是在本例中，作为**PC**服务的局部变量。

   第二个区别是 **PC_Cred** 凭证没有用于授权针对Prism Central的API调用。 相反， 我们使用的是内置的 **calm_jwt** 宏提供的一个 `JSON Web Token <https://en.wikipedia.org/wiki/JSON_Web_Token>`_。

#. 单击 **+ Task** 按钮再次添加第二个任务到 **GetDefaultSubnet** 自定义操作。  填写以下字段：

   - **Task Name** - GetSubnetInfo
   - **Type** - Execute
   - **Script Type** - EScript
   - **Script** - *Script Provided Below*

   .. code-block:: python

     # Get the JWT
     jwt = '@@{calm_jwt}@@'

     # Set the headers, url, and payload
     headers = {'Content-Type': 'application/json', 'Accept': 'application/json', 'Authorization': 'Bearer {}'.format(jwt)}
     url     = "https://@@{address}@@:9440/api/nutanix/v3/subnets/@@{SUBNET}@@"
     payload = {}

     # Make the request
     resp = urlreq(url, verb='GET', params=json.dumps(payload), headers=headers, verify=False)

     # If the request went through correctly, print it out.  Otherwise error out, and print the response.
     if resp.ok:
        print json.dumps(json.loads(resp.content), indent=4)
        exit(0)
     else:
        print "Get request failed", resp.content
        exit(1)

   在这个任务中，我们使用GET API调用和前一个任务返回的 **SUBNET** UUID变量动态地返回关于默认子网的详细信息。

   .. figure:: images/get_subnet_info.png

#. 单击 **Save**, 并确保不会出现任何错误或警告。

运行自定义操作
++++++++++++++++++++++++++

#. **Launch** 启动蓝图。 将应用程序命名为 *Initials*\ **-RestCalls**， 然后单击 **Create**。

   The **Create** 任务会很快完成，因为没有部署任何虚拟机，也没有运行安装包的脚本。 
   
#. 一旦应用程序达到 **Running** 状态， 请选择 **Manage** 选项卡。

   .. figure:: images/app_create.png

#. 接下来，通过单击:fa:`play` 图标来运行 **RESTList** 操作。 将出现一个新窗口，显示 **kind** 变量和默认 **apps** 值。单击 **Run**。

   .. figure:: images/apps_run.png

#. 在右侧窗口的输出中，最大化 **RuntimePost** 任务，并查看API输出。可以通过单击 :fa:`eye` 图标来切换输出窗口，最大化 output/script窗口，使查看更容量。 正如所料，该脚本返回一个JSON主体，其中包含一个描述Calm中每个已启动应用程序的数组。

   .. figure:: images/apps_run2.png

#. 再次运行 **RESTList** 操作，将值更改为另一个 `Prism Central API 实例<https://developer.nutanix.com/reference/prism_central/v3/>`_, 例如 **images**, **clusters**, **hosts**, 或 **vms**。

#. 最后，运行 **GetDefaultSubnet** 操作。展开 **GetSubnetUUID** 和 **GetSubnetInfo** 任务， 查看每个任务的输出。 默认子网的名称和VLAN ID是多少？

   .. figure:: images/GetDefaultSubnet.png

   .. figure:: images/GetDefaultSubnet2.png

发布到任务库
++++++++++++++++++++++++++++++

诸如通用API调用、通用服务的安装包、域连接等任务可以广泛应用于多个蓝图。这些任务可以在不使用第三方工具或手动复制和粘贴脚本的情况下使用，而是将其发布到Task Library(用于代码重用的平静的中央存储库)。

#. 在蓝图编辑器中打开你的 *Initials*\ **-EScript** 蓝图。

#. 在 **Application Overview > Application Profile** 窗口中，选择 **RESTList** 操作。

#. 选择 **RuntimeList** 任务以在**控制面板** 中打开任务。

#. 单击 **Publish to Library**.

#. 在 **Publish Task** 窗口中，进行以下更改：

   - **Name** - *Initials* Prism Central Runtime List
   - Replace **address** with **Prism_Central_IP**

   .. figure:: images/publish_task.png

#. 单击 **Apply** ，注意脚本窗口中的原始 **address** 宏被替换为 **Prism_Central_IP**。 替换宏名可以使您更通用或更具描述性，从而提高任务的可移植性。
#. 单击 **Publish**.

#. 在侧栏中打开 **Task Library**。选择已发布的任务。默认情况下，该任务将对最初发布该任务的项目可用，但您可以指定与之共享该任务的其他项目。

Takeaways
+++++++++

关于 **Nutanix Calm** 你应该知道的关键事项是什么？

- 任务库允许将常用操作写入一次并反复重复使用。 随着时间的推移，更多的对象将被集成到任务库中，从Nutanix提供的常见任务到整个服务对象。

- Calm 2.7引入了HTTP任务，允许更容易实现Escript的最常见用法（发送API调用）。

- 除了能够使用Bash和Powershell脚本之外，Nutanix Calm还可以使用EScript(一种沙箱Python编译器)来提供应用程序生命周期管理。

- EScript任务直接在Calm引擎中运行，而不是在远程机器上执行。

- Shell、Powershell和EScript任务都可以根据脚本输出设置变量。然后，该变量可以用于蓝图的其他部分。

- 任务库允许将常用的任务发布到中央存储库中，从而允许跨项目和蓝图共享这些脚本。


.. |proj-icon| image:: ../images/projects_icon.png
.. |mktmgr-icon| image:: ../images/marketplacemanager_icon.png
.. |mkt-icon| image:: ../images/marketplace_icon.png
.. |bp-icon| image:: ../images/blueprints_icon.png
.. |blueprints| image:: images/blueprints.png
.. |applications| image:: images/blueprints.png
