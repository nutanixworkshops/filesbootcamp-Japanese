.. _file_analytics_anomaly:

-----------------------------------------------------
File Analytics: Anomaly Rules 「ファイル分析：異常ルール」
-----------------------------------------------------

概要
++++++++



Define Anomaly Rules「異常ルールを定義する」
++++++++++++++++++++++++++++++++++++++++

#. 歯車アイコンの下から　**Define Anomaly Rules「異常ルールの定義」**　に移動して、2つの異常ルールを作成します

   .. figure:: images/39.png

#. **Define Anomaly Rules** を選択し、次の設定でルールを作成します

   - **Events「イベント:」** Delete　削除
   - **Minimum Operation %　「最小操作％」:** 1
   - **Minimum Operation Count　「最小操作数」:** 10
   - **User「ユーザー」:** All Users 「すべてのユーザー」
   - **Type「タイプ」:** Hourly 「毎時」
   - **Interval「間隔」:** 1

#. その異常テーブルエントリの「保存」を**Save「選択」**します。

#. Choose **+ Configure new anomaly** and create a second rule with the following settings

   - **Events**: Create
   - **Minimum Operation %**: 1
   - **Minimum Operation Count**: 10
   - **User**: All Users
   - **Type**: Hourly
   - **Interval**: 1

#. Choose **Save** for that anomaly table entry

   .. figure:: images/40.png

#. Select **Save** to exit the Define Anomaly Rules window

Load Sample Data
+++++++++++++++++++++

#. Go to the Sample Data folder in the Marketing share and copy, then paste that folder to the same share.

   .. figure:: images/42.png

#. Now delete the original Sample Data folder.

Cause Error Condition
+++++++++++++++++++++

#. While waiting for the Anomaly Alerts to populate we’ll create a permission denial.

   .. note:: The Anomaly engine runs every 30 minutes.  While this setting is configurable from the File Analytics VM, modifying this variable is outside the scope of this lab.

#. Create a new directory called **RO** in the Marketing share

#. Create a text file in the **RO** directory with some text like “hello world” called **myfile.txt**

#. Go to the **Properties** of the **RO** folder and select the Security tab

#. Select **Advanced**

#. Choose **Disable inheritance** and select the **Convert…** option

#. Then add the **Everyone** permissions with the following:

   - Read & Execute
   - List folder contents
   - Read

   .. figure:: images/43.png

#. Choose **OK** then **OK** again

#. Open a PowerShell window as a specific user

   - Hold down **Shift** and **right click** on the **PowerShell icon** on the taskbar
   - Select **Run as different user**

   .. figure:: images/44.png

#. Enter the following

   - **User name**: Poweruser01
   - **Password**: nutanix/4u

#. Change Directories into the Marketing share and the **RO** directory

     .. code-block:: bash

        cd \\xyz-files.ntnxlab.local\marketing\RO

#. Execute the following commands, the first should succeed, the second should fail:

     .. code-block:: bash

        more .\myfile.txt
        rm .\myfile.txt

   .. figure:: images/45.png

#. After a minute or so you should see **Permission Denials** in both the dashboard and the **Audit Trails** view.  You may need to refresh your browser.

   .. figure:: images/46.png

   .. note:: The Capacity Trend dashboard panel updates every 24 hrs.
