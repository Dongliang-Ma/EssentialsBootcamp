.. title:: Nutanix Essentials Bootcamp

.. toctree::
  :maxdepth: 2
  :caption: Prism Central
  :name: _prism_central
  :hidden:

  what_is_prism_central/what_is_prism_central
  prism_central_overview/prism_central_overview
  prism_central_dashboards_reports/prism_central_dashboards_reports
  prism_central_resource_planning/prism_central_resource_planning

.. toctree::
  :maxdepth: 2
  :caption: Calm Lab
  :name: _calm_lab
  :hidden:

  what_is_calm/what_is_calm
  calm_enable/calm_enable
  calm_projects/calm_projects
  calm_linux/calm_linux
  calm_day2/calm_day2
  calm_escript/calm_escript
  calm_win/calm_win
  calm_windows_blueprint/calm_windows_blueprint
  calm_marketplace/calm_marketplace

.. toctree::
  :maxdepth: 2
  :caption: Flow Lab
  :name: _flow_lab
  :hidden:

  what_is_flow/what_is_flow
  flow_lab_staging/flow_lab_staging
  flow_enable/flow_enable
  flow_secure_app/flow_secure_app
  flow_isolate_environments/flow_isolate_environments
  flow_quarantine_vm/flow_quarantine_vm

.. toctree::
  :maxdepth: 2
  :caption: Optional Labs
  :name: _optional_labs
  :hidden:

  calm_sshkey_creation/calm_sshkey_creation
  flow_assign_categories_in_calm/flow_assign_categories_in_calm


.. toctree::
  :maxdepth: 2
  :caption: Appendix
  :name: _appendix
  :hidden:

  appendix/glossary
  appendix/basics
  taskman/taskman

.. _getting_started:

---------------
Getting Started
---------------

Welcome to the Nutanix Essentials Bootcamp! This workbook accompanies an instructor-led session that introduces Nutanix solutions and many common management tasks. Each section has a lesson and an exercise to give you hands-on practice. The instructor explains the exercises and answers any additional questions that you may have.

At the end of the bootcamp, attendees should understand the basic concepts and technologies that make up the Nutanix Enterprise Cloud stack and should be well prepared for a hosted or onsite proof-of-concept (POC) engagement.

What's New
++++++++++

- Workshop updated for the following software versions:
    - AOS & PC 5.10.x

- Optional Lab Updates:


Agenda
++++++

- Introductions
- Prism Pro
- Nutanix Calm
- Nutanix Flow

Introductions
+++++++++++++

- Name
- Familiarity with Nutanix

Initial Setup
+++++++++++++

- Take note of the *Passwords* being used.
- Log into your virtual desktops (connection info below)

Environment Details
+++++++++++++++++++

Nutanix Workshops are intended to be run in the Nutanix Hosted POC environment. Your cluster will be provisioned with all necessary images, networks, and VMs required to complete the exercises.

Networking
..........

Hosted POC clusters follow a standard naming convention:

- **Cluster Name** - POC\ *XYZ*
- **Subnet** - 10.**21**.\ *XYZ*\ .0
- **Cluster IP** - 10.**21**.\ *XYZ*\ .37

If provisioned from the marketing pool:
- **Cluster Name** - MKT\ *XYZ*
- **Subnet** - 10.**20**.\ *XYZ*\ .0
- **Cluster IP** - 10.**20**.\ *XYZ*\ .37

For example:

- **Cluster Name** - POC055
- **Subnet** - 10.21.55.0
- **Cluster IP** - 10.21.55.37

Throughout the Workshop there are multiple instances where you will need to substitute *XYZ* with the correct octet for your subnet, for example:

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - IP Address
     - Description
   * - 10.21.\ *XYZ*\ .37
     - Nutanix Cluster Virtual IP
   * - 10.21.\ *XYZ*\ .39
     - **PC** VM IP, Prism Central
   * - 10.21.\ *XYZ*\ .40
     - **DC** VM IP, NTNXLAB.local Domain Controller

Each cluster is configured with 2 VLANs which can be used for VMs:

.. list-table::
  :widths: 25 25 10 40
  :header-rows: 1

  * - Network Name
    - Address
    - VLAN
    - DHCP Scope
  * - Primary
    - 10.21.\ *XYZ*\ .1/25
    - 0
    - 10.21.\ *XYZ*\ .50-10.21.\ *XYZ*\ .124
  * - Secondary
    - 10.21.\ *XYZ*\ .129/25
    - *XYZ1*
    - 10.21.\ *XYZ*\ .132-10.21.\ *XYZ*\ .253

Credentials
...........

.. note::

  The *<Cluster Password>* is unique to each cluster and will be provided by the leader of the Workshop.

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - Credential
     - Username
     - Password
   * - Prism Element
     - admin
     - *<Cluster Password>*
   * - Prism Central
     - admin
     - *<Cluster Password>*
   * - Controller VM
     - nutanix
     - *<Cluster Password>*
   * - Prism Central VM
     - nutanix
     - *<Cluster Password>*

Each cluster has a dedicated domain controller VM, **DC**, responsible for providing AD services for the **NTNXLAB.local** domain. The domain is populated with the following Users and Groups:

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - Group
     - Username(s)
     - Password
   * - Administrators
     - Administrator
     - nutanix/4u
   * - SSP Admins
     - adminuser01-adminuser25
     - nutanix/4u
   * - SSP Developers
     - devuser01-devuser25
     - nutanix/4u
   * - SSP Power Users
     - poweruser01-poweruser25
     - nutanix/4u
   * - SSP Basic Users
     - basicuser01-basicuser25
     - nutanix/4u

Access Instructions
+++++++++++++++++++

The Nutanix Hosted POC environment can be accessed a number of different ways:

Citrix XenDesktop
.................

https://citrixready.nutanix.com - *Accessible via the Citrix Receiver client or HTML5*

**Nutanix Employees** - Use your NUTANIXDC credentials

**Non-Employees** - **Username:** POCxxx-User01 (up to POCxxx-User20), **Password:** *<Provided by Instructor>*

Employee Pulse Secure VPN
..........................

https://sslvpn.nutanix.com - Use your CORP credentials

Non-Employee Pulse Secure VPN
..............................

https://lab-vpn.nutanix.com - **Username:** POCxxx-User01 (up to POCxxx-User20), **Password:** *<Provided by Instructor>*

Under **Client Application Sessions**, click **Start** to the right of **Pulse Secure** to download the client.

Install and open **Pulse Secure**.

Add a connection:

- **Type** - Policy Secure (UAC) or Connection Server
- **Name** - HPOC VPN
- **Server URL** - lab-vpn.nutanix.com
