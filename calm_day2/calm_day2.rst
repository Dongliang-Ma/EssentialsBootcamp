.. _calm_day2：

---------------------------------
Calm：第2天操作（可选）
---------------------------------

概述
++++++++

在：ref：`calm_linux`和：ref：`calm_win`实验室中，您探索了如何使用蓝图来模拟和部署复杂应用程序。然而，Calm，它能够在 **整个** 生命周期中管理应用程序。

**在本实验中，您将在Calm中实施自定义操作以解决“第2天”操作，包括横向扩展，纵向扩展和升级应用程序。**

实验室设置
++++++++++

本实验取决于：ref：`calm_linux`实验室中实现的多层 **Task Manger** Web应用程序的可用性。

如果您已经基本熟悉并且尚未完成：ref：`calm_linux`实验，请参阅：ref：`taskman`实验以获取有关导入基于  **Task Manager** 蓝图的说明。

 **导入后无需启动任务管理器蓝图。**

横向扩展
+++++++++++

假设您是我们正在构建的任务管理器应用程序的管理员，目前，您还不确定最终用户对该应用程序的需求量。或者，你可能认为需求会随着时间的推移而起伏。我们如何轻松扩展以满足这种不断变化的需求？

在创建任务管理器蓝图期间，**WebServer** 服务配置了最少2个副本，最多为4个。因此，Calm将在初始部署期间创建2个WebServer VM。如果2个副本不足以处理最终用户的负载，则需要进行 **Scale Out** 操作。

#. 在 **Application Overview> Application Profile** 部分中，展开 **Default** Application Profile。

   .. figure:: images/510scaleout0.png

#. 选择：fa：`plus-circle`旁边的 **Actions** 添加一个新的自定义动作。在右侧的 **Configuration Pane** 中，将新操作重命名为 **Scale Out**。

   .. figure:: images/510scaleout1.png

#. 在下面的 **below** 的 **WebServer** 服务标签中，单击 **+ Task** 按钮添加扩展任务，并填写以下字段：

   - **Task Name** - web_scale_out
   - **Scaling Type** - Scale Out
   - **Scaling Count** - 1

   .. figure:: images/510scaleout2.png

.. note::

   服务标签下方显示的 **+Task** 按钮仅用于向上和向下缩放副本数量，因此选择正确的选项很重要。 

   当用户稍后运行 **Scale Out** 任务时，将创建一个新的 **WebServer**  VM，并将执行该服务的 **Package Install** 任务。但是，我们需要修改 **HAProxy** 配置才能开始利用这个新的Web服务器。

#.  **Within** 的 **HAProxy** 服务标签中，单击 **+Task** 按钮，然后填写以下字段：

   - **Task Name** - add_webserver
   - **Type** - Execute
   - **Script Type** - Shell
   - **Credential** - CENTOS

#. 将以下脚本复制并粘贴到 **Script** 字段中：

 .. code-block:: bash

     #!/bin/bash
     set -ex

     host=$(echo "@@{WebServer.address}@@" | awk -F "," '{print $NF}')
     port=80
     echo " server host-${host} ${host}:${port} weight 1 maxconn 100 check" | sudo tee -a /etc/haproxy/haproxy.cfg

     sudo systemctl daemon-reload
     sudo systemctl restart haproxy

   该脚本将解析WebServer地址数组中的最后一个IP地址，并将其附加到haproxy.cfg文件中。但是，我们希望确保在新的WebServer完全启动后 **after** 不会发生这种情况，否则HAProxy服务器可能会向无法运行的WebServer发送请求。

#. 要解决此问题，请创建边界以强制依赖 **web_scale_out** 任务在 **add_webserver** 任务之前完成。

   您的 **Workspace** 现在应该如下所示：

   .. figure:: images/510scaleout3.png

缩小
++++++++++

在繁忙季节结束后，您希望通过缩减已部署的Web服务器的数量来优化资源利用率。

#. 选择：fa：`plus-circle`将名为 **Scale In** 的自定义动作添加到默认 **Application Profile**。

   .. figure:: images/510scalein1.png

#. 在 **WebServer** 服务标签 **下方**，单击 **+Task** 按钮添加扩展任务，并填写以下字段：

   - **Task Name** - web_scale_in
   - **Scaling Type** - Scale In
   - **Scaling Count** - 1

   .. figure:: images/510scalein2.png

   当用户稍后运行 **Scale In** 任务时，最后一个 **WebServer** 副本将运行其 **Package Uninstall** 任务，VM将被关闭，然后被删除，这将回收资源。但是，我们确实需要修改 **HAProxy** 配置，以确保我们不再向要删除的Web服务器发送流量。

#. 在 **HAProxy** 服务标签中 **Within**，单击 **+Task** 按钮，然后填写以下字段：

   - **Task Name** - del_webserver
   - **Type** - Execute
   - **Script Type** - Shell
   - **Credential** - CENTOS

#. 将以下脚本复制并粘贴到 **Script** 字段中：

   .. code-block:: bash

     #!/bin/bash
     set -ex

     host=$(echo "@@{WebServer.address}@@" | awk -F "," '{print $NF}')
     sudo sed -i "/$host/d" /etc/haproxy/haproxy.cfg

     sudo systemctl daemon-reload
     sudo systemctl restart haproxy

与scale out脚本类似，这个脚本将解析WebServer地址数组中的最后一个IP，并使用“sed <http://www.grymoire.com/Unix/Sed.html>”命令从haproxy.cfg中删除相应的条目。

同样，与scale out脚本类似，我们希望确保在删除请求之前 **before** 停止向VM 发送请求。

#. 要解决这个问题，创建一个edge来强制依赖于在 **web_scale_in** 任务之前完成的 **del_webserver** 任务。

   你的 **Workspace** 现在应该是这样的:

   .. figure:: images/510scalein3.png

#. 单击 **Save**，确保不会弹出错误或警告。如果他们这样做了，请解决这些问题，并再次 **Save**。

升级
+++++++++

您的公司有权保持所有应用程序代码都是最新的，以帮助最小化安全漏洞。你的公司也有严格的变更控制程序，这意味着你只能在周末更新你的应用程序。目前，在维护窗口期间，您每个月的一个星期六都要花费大量的时间来完成应用程序更新过程。让我们看看如何通过使用Calm对应用程序升级进行建模，从而使你重新找回周末的。

#. 选择 :fa:`plus-circle` 将一个名为 **Upgrade** 的自定义操作添加到默认的 **Application Profile** 中。

  我们首先需要做的是停止我们每项服务的相应流程。

#. 在我们提供的3项服务的每项 **Within each** 内，按 **+ Task** 按钮添加新任务，并填写以下资料:

   +------------------+-----------+---------------+-------------+
   | **Service Name** | MySQL     | WebServer     | HAProxy     |
   +------------------+-----------+---------------+-------------+
   | **Task Name**    | StopMySQL | StopWebServer | StopHAProxy |
   +------------------+-----------+---------------+-------------+
   | **Type**         | Execute   | Execute       | Execute     |
   +------------------+-----------+---------------+-------------+
   | **Script Type**  | Shell     | Shell         | Shell       |
   +------------------+-----------+---------------+-------------+
   | **Credential**   | CENTOS    | CENTOS        | CENTOS      |
   +------------------+-----------+---------------+-------------+
   | **Script**       | See Below | See Below     | See Below   |
   +------------------+-----------+---------------+-------------+

   **StopMySQL Script:**

   .. code-block:: bash

      #!/bin/bash
      set -ex

      sudo systemctl stop mysqld

   **StopWebServer Script:**

   .. code-block:: bash

      #!/bin/bash
      set -ex

      sudo systemctl stop php-fpm
      sudo systemctl stop nginx

   **StopHAProxy Script:**

   .. code-block:: bash

      #!/bin/bash
      set -ex

      sudo systemctl stop haproxy

   完成后，您的 **Workspace** 应该是这样的:

   .. figure:: images/upgrade1.png

   与扩展和初始部署操作类似，我们不希望web服务器先于HAProxy关机，也不希望MySQL数据库先于web服务器关机。

#. 重新定义服务之间的边界，比如HAProxy在webserver之前停止，所有的webserver在MySQL之前停止:

   .. figure:: images/upgrade2.png

   现在我们的关键服务已停止，我们就可以执行更新了。

#. 再重复一次，在每个服务中 **within each**，添加一个新任务。 除名称外，所有3个任务都是相同的：

   +------------------+--------------+------------------+----------------+
   | **Service Name** | MySQL        | WebServer        | HAProxy        |
   +------------------+--------------+------------------+----------------+
   | **Task Name**    | UpgradeMySQL | UpgradeWebServer | UpgradeHAProxy |
   +------------------+--------------+------------------+----------------+
   | **Type**         | Execute      | Execute          | Execute        |
   +------------------+--------------+------------------+----------------+
   | **Script Type**  | Shell        | Shell            | Shell          |
   +------------------+--------------+------------------+----------------+
   | **Credential**   | CENTOS       | CENTOS           | CENTOS         |
   +------------------+--------------+------------------+----------------+
   | **Script**       | See Below    | See Below        | See Below      |
   +------------------+--------------+------------------+----------------+

   **Script for all 3 Upgrade Tasks:**

   .. code-block:: bash

      #!/bin/bash
      set -ex

      sudo yum update -y

   这个脚本将使用Red Hat/CentOS包管理工具 `yum <https://access.redhat.com/solutions/9934>`_ 搜索并安装所有已安装包的更新。

   你的 **Workspace** 现在应该是这样的:

   .. figure:: images/upgrade3.png

    从任务排序的角度来看，我们是否需要绘制任何编排边缘？ 请注意，在上面的屏幕截图中，Calm自动从 **Stop** 任务绘制并到 **Upgrade** 任务的边界，这是必要的。 但是，我们需要任何一对一的依赖关系吗？

    如果你说“不”，那你就是对的。 关键组件是服务的启动和停止，没有理由一次升级一个服务。

    除非另有说明，否则Calm将始终并行运行任务以节省时间。

    现在我们的服务已升级，是时候启动它们了。

#. 再重复一次，在每个服务中 **within each**， 包含以下值：

   +------------------+--------------+------------------+----------------+
   | **Service Name** | MySQL        | WebServer        | HAProxy        |
   +------------------+--------------+------------------+----------------+
   | **Task Name**    | StartMySQL   | StartWebServer   | StartHAProxy   |
   +------------------+--------------+------------------+----------------+
   | **Type**         | Execute      | Execute          | Execute        |
   +------------------+--------------+------------------+----------------+
   | **Script Type**  | Shell        | Shell            | Shell          |
   +------------------+--------------+------------------+----------------+
   | **Credential**   | CENTOS       | CENTOS           | CENTOS         |
   +------------------+--------------+------------------+----------------+
   | **Script**       | See Below    | See Below        | See Below      |
   +------------------+--------------+------------------+----------------+

   **StartMySQL Script:**

   .. code-block:: bash

      #!/bin/bash
      set -ex

      sudo systemctl start mysqld

   **StartWebServer Script:**

   .. code-block:: bash

      #!/bin/bash
      set -ex

      sudo systemctl start php-fpm
      sudo systemctl start nginx

   **StartHAProxy Script:**

   .. code-block:: bash

      #!/bin/bash
      set -ex

      sudo systemctl start haproxy

   你的 **Workspace** 现在应该是这样的:

   .. figure:: images/upgrade4.png

   这一次，我们 **Do** 需要额外的编排边界。 如前所述，我们不希望在我们的WebServers之前启动HAProxy服务，或者在我们的MySQL数据库之前启动我们的WebServers。

#. 创建从MySQL开始的业务流程边界，然后是WebServers，最后是HAProxy：

   .. figure:: images/upgrade5.png

#. 单击 **Save** 并确保没有错误或警告弹出。 如果有错误，请解决这些问题，然后再次 **Save**。

启动和管理应用程序
++++++++++++++++++++++++++++++++++++++

#. 从Blueprint Editor的上方工具栏中，单击 **Launch**。

#. 为VM命名指定唯一的 **Application Name** (例如 *Initials*\ -CalmLinuxIntro1)和 **User_initials** 运行时变量值。

#. 单击 **Create**。

#. 一旦应用程序达到 **Running** 状态，导航到 **Manage** 选项卡，然后运行 **Scale Out** 操作。

   可以在 **Audit** 选项卡上监视对应用程序的更改。

   完成扩展操作后，您可以登录HAProxy VM并验证新的Web服务器是否已添加到 ``/etc/haproxy/haproxy.cfg``。

#. 运行 **Upgrade** 操作以更新每项服务。

#. 最后，运行 **Scale In** 操作以删除其他Web Server VM。

Variable Scaling变量缩放（可选）
++++++++++++++++++++++++++++++++

在这个实验室中，您配置了缩放操作，这些操作通过单个VM扩展或收缩webServer服务数组。

创建新的自定义操作时，可以在特定于该操作的配置窗格中定义其他变量。

.. figure :: images / optional1.png

利用运行时变量，可以修改scale out或scale in 操作来执行变量缩放操作吗?

这将需要一些bash脚本编写经验，以确保从haproxy.cfg文件中添加和/或删除适当数量的条目。

小贴士
+++++++++

关于** Nutanix Calm **你应该知道的关键事项是什么？

 - 通过对复杂的第2天操作进行建模，Calm不仅可以协调复杂的应用程序部署，还可以在整个生命周期内管理应用程序。

 - 无论是内置任务（如扩展）还是自定义任务（如升级），都可以指示Calm以特定顺序执行操作，或者如果顺序无关紧要，则可以并行执行以节省时间。

 - 您目前正在进行哪些操作？它很可能可以在Calm中建模，为您节省数不清的时间。享受你的周末吧！
