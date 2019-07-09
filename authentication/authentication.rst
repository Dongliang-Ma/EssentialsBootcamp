.. _authentication：

-------------------------------
身份验证和角色映射
-------------------------------

在大多数环境中，您需要将Nutanix群集连接到公司的Active Directory或LDAP服务器。

这允许管理员使用这些凭据而不是Nutanix群集本地凭据登录。

..注意::
 Prism Central和Prism Element的步骤相同

在** Prism **中， 点击 :fa:`cog` **> Authentication**

点击 **+ New Directory**

填写以下字段，然后单击 **Save**：

- **Directory Type** - Active Directory
- **Name** - NTNXLAB
- **Domain** - ntnxlab.local
- **Directory URL** - ldaps://10.21.XX.40
- **Service Account Name** - administrator@ntnxlab.local
- **Service Account Password** - nutanix/4u

.. figure :: images / authentication_01.png

点击 **NTNXLAB** 旁边的黄色感叹号！

.. figure :: images / authentication_02.png

单击 **Click Here** 转到角色映射界面

点击 **+ New Mapping**

- **Directory** - NTNXLAB
- **LDAP Type** - user
- **Role** - Cluster Admin
- **Values** - administrator

.. figure :: images / authentication_03.png

关闭角色映射和身份验证窗口
