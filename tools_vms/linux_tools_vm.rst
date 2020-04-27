.. _linux_tools_vm:

---------------
Linux Tools VM
---------------

概要
+++++++++

このCentOS VMイメージは、複数のラボ演習をサポートするために使用されるパッケージを持ちます。

**Lab Setup** の一環として指示された場合は、割り当てられたクラスターにこのVMをデプロイします。

.. raw:: html

  <strong><font color="red">VMは1度だけデプロイします。ラボの完了時にクリーンアップする必要はありません。</font></strong>

Linux-ToolsVMのデプロイ
++++++++++++++++

**Prism Central** > :fa:`bars` **> Virtual Infrastructure > VMs** にて **Create VM** をクリックする。

以下情報を入力する。:

- **Name** - *Initials*-Linux-ToolsVM
- **Description** - (Optional) Description for your VM.
- **vCPU(s)** - 1
- **Number of Cores per vCPU** - 2
- **Memory** - 2 GiB

- Select **+ Add New Disk**
    - **Type** - DISK
    - **Operation** - Clone from Image Service
    - **Image** - Linux_ToolsVM.qcow2
    - Select **Add**

- **Add New NIC** を選択
  　- **VLAN Name** - Secondary
  　- **Add** をクリック

**Save** をクリックします

VM をパワーオン
