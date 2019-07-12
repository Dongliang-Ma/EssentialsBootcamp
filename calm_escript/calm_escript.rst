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

#. Select the **PC** service. In the **Configuration Pane**, select the **Service** tab. Add a variable named **SUBNET**, leaving all other fields blank.

   .. figure:: images/subnet_variable.png

#. In the **Application Overview > Application Profile > Default**, section, select :fa:`plus-circle` next to **Actions** to add a new, custom action.

#. Name the action **GetDefaultSubnet**.

   .. figure:: images/get_default_subnet.png

#. Click the **+ Task** button to add a task to the **GetDefaultSubnet** custom action.  Fill in the following fields:

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

   There are two key differences between the **RESTList** and **GetDefaultSubnet** tasks. The first difference is the use of the **Set Variable** task type. Take note of the **print "SUBNET={0}"** line: Calm will parse output in the format of **variable=value**, and set the variable equal to the value.  In this example, we're printing the variable called **SUBNET** is equal to the UUID of the "default_subnet_reference" field in the initial API call response. In the **Output** field below the Script body, we must paste in the variable name for Calm to set the variable appropriately. The variable must already be defined in the Calm blueprint, whether globally, or in this case, as a variable local to the **PC** service.

   The second difference is that the **PC_Cred** credential was not used to authorize the API call against Prism Central. Instead, we're using a `JSON Web Token <https://en.wikipedia.org/wiki/JSON_Web_Token>`_ provided by the built-in **calm_jwt** macro.

#. Click the **+ Task** button again to add a second task to the **GetDefaultSubnet** custom action.  Fill in the following fields:

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

   In this task we're dynamically returning details about the default subnet using a GET API call and the **SUBNET** UUID variable returned by the previous task.

   .. figure:: images/get_subnet_info.png

#. Click **Save**, and ensure no errors or warnings appear.

Running the Custom Actions
++++++++++++++++++++++++++

#. **Launch** the blueprint. Name the application *Initials*\ **-RestCalls**, and then click **Create**.

   The **Create** task should complete quickly, as no VMs are being provisioned or Package Install scripts being run.

#. Once the application reaches **Running** status, select the **Manage** tab.

   .. figure:: images/app_create.png

#. Next, run the **RESTList** action by clicking its :fa:`play` icon. A new window appears displaying the **kind** variable and default **apps** value. Click **Run**.

   .. figure:: images/apps_run.png

#. In the output on the right pane, maximize the **RuntimePost** task, and view the API output. The output pane can be toggled by clicking the :fa:`eye` icon. Maximize the output/script window to make viewing easier. As expected, the script returns a JSON body with an array describing each launched application in Calm.

   .. figure:: images/apps_run2.png

#. Run the **RESTList** action again, altering the value to another `Prism Central API entity <https://developer.nutanix.com/reference/prism_central/v3/>`_, such as **images**, **clusters**, **hosts**, or **vms**.

#. Finally, run the **GetDefaultSubnet** action. Expand both the **GetSubnetUUID** and **GetSubnetInfo** tasks, reviewing the output for each task. What is the name and VLAN id of your default subnet?

   .. figure:: images/GetDefaultSubnet.png

   .. figure:: images/GetDefaultSubnet2.png

Publishing to the Task Library
++++++++++++++++++++++++++++++

Tasks such as common API calls, package installations for common services, domain joins, etc. can be broadly applicable to multiple blueprints. These tasks can be used without leveraging third party tools or manually copying and pasting scripts by instead publishing into the Task Library, Calm's central repository for code re-use.

#. Open your *Initials*\ **-EScript** blueprint in the Blueprint Editor.

#. In the **Application Overview > Application Profile** pane, select the **RESTList** action.

#. Select the **RuntimeList** task to open the task in the **Configuration Pane**.

#. Click **Publish to Library**.

#. In the **Publish Task** window, make the following changes:

   - **Name** - *Initials* Prism Central Runtime List
   - Replace **address** with **Prism_Central_IP**

   .. figure:: images/publish_task.png

#. Click **Apply** and note that the original **address** macro was replaced with **Prism_Central_IP** in the script window. Replacing macro names allows you to be more generic or descriptive to increase task portability.

#. Click **Publish**.

#. Open the **Task Library** in the sidebar.  Select your published task. By default, the task will be available to the project from which it was originally published, but you can specify additional projects with which to share the task.

Takeaways
+++++++++

What are the key things you should know about **Nutanix Calm**?

- The task library allows commonly used operations to be written once and reused over and over again.  As time goes on more objects will be integrated into the task library, from Nutanix-provided common tasks to entire service objects

- Calm 2.7 introduced the HTTP task, allowing the most common use of Escript to be more easily implemented (sending API calls)

- In addition to being able to use Bash and Powershell scripts, Nutanix Calm can use EScript, which is a sandboxed Python interpreter, to provide application lifecycle management.

- EScript tasks are run directly within the Calm engine, rather than being executed on the remote machine.

- Shell, Powershell, and EScript tasks can all be utilized to set a variable based on script output.  That variable can then be used in other portions of the blueprint.

- The Task Library allows for publishing of commonly used tasks into a central repository, giving the ability to share these scripts across Projects and Blueprints.


.. |proj-icon| image:: ../images/projects_icon.png
.. |mktmgr-icon| image:: ../images/marketplacemanager_icon.png
.. |mkt-icon| image:: ../images/marketplace_icon.png
.. |bp-icon| image:: ../images/blueprints_icon.png
.. |blueprints| image:: images/blueprints.png
.. |applications| image:: images/blueprints.png
