.. _windows_tools_vm:

----------------
Windows Tools VM
----------------

概要
+++++

このWindows Server 2012 R2 イメージには以下ツールがインストールされています:

- Microsoft Remote Server Administration Tools (RSAT)
- PuTTY, CyberDuck, WinSCP
- Sublime Text 3, Visual Studio Code
- OpenOffice
- Python
- pgAdmin
- Chocolatey Package Manager

**Lab Setup** の一環として指示された場合は、割り当てられたクラスターにこのVMをデプロイします

.. raw:: html

  <strong><font color="red">VMは1度だけデプロイします。ラボの完了時にクリーンアップする必要はありません。</font></strong>

Tools VMのデプロイ
++++++++++++++++++

**Prism Central** > :fa:`bars` **> Virtual Infrastructure > VMs** と進み、 **Create VM** をクリックする。

以下を入力する。:

- **Name** - *Initials*-Windows-ToolsVM
- **Description** - (Optional) Description for your VM.
- **vCPU(s)** - 1
- **Number of Cores per vCPU** - 2
- **Memory** - 4 GiB
- Select **+ Add New Disk**
    - **Type** - DISK
    - **Operation** - Clone from Image Service
    - **Image** - ToolsVM.qcow2
    - Select **Add**

- **Add New NIC** を選択
    - **VLAN Name** - Secondary
    - **Add** をクリック

**Save** をクリックする。

VMの電源を入れる。

ログインの資格情報は以下:

- **Username** - NTNXLAB\\Administrator
- **password** - nutanix/4u
