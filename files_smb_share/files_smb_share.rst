.. _files_smb_share:

-----------------
Files:SMB共有の作成
-----------------

概要
++++

この演習では、SMB共有を作成し、テストします。
これは、ホームディレクトリ、ユーザープロファイル、およびWindowsクライアントが一般的にアクセスする部門別共有など、他の非構造化ファイルデータをサポートするために使用されます。

ラボのセットアップ
+++++++++++++++

このラボでは、:ref:`windows_tools_vm` にプロビジョニングされたアプリケーションが必要です。

このVMがまだデプロイされていない場合は、ラボに進む前に、リンクされている手順を参照してください。

#. ファイルサーバーの展開を待っている間に、必要に応じてWindows Tools VMを展開します

#. RDPまたはコンソールを介してWindows Tools VMに接続します

#. ファイル分析のサンプルファイルをTools VMにダウンロードします:

   - `https://peerresources.blob.core.windows.net/sample-data/SampleData_Small.zip <https://peerresources.blob.core.windows.net/sample-data/SampleData_Small.zip>`_

SMB共有の使用
++++++++++++

共有を作成する
............

#. **Prism> ファイルサーバーで** Share / Export** をクリックします

#. 次のフィールドに入力します:

   - **Name「名」** - Marketing
   - **Description (Optional)「説明（オプション)」** - Departmental share for marketing team
   - **File Server** - BootcampFS
   - **Share Path (Optional)「共有パス（オプション)」** -　空白のままにします。このフィールドでは、ネストされた共有を作成する既存のパスを指定できます
   - **Max Size (Optional)「最大サイズ（オプション）」** - 空白のままにします。このフィールドでは、個々の共有にハードクォータを設定できます
   - **Select Protocol　「プロトコルの選択」** - SMB

   .. figure:: images/14.png

#. **Next** をクリックします.

#. **Enable Access Based Enumeration「アクセスベースの列挙」** と **Self Service Restore「セルフサービスの復元を有効にする」**　を選択します

   .. figure:: images/15.png

   部門別の共有を作成するときは、標準の共有として作成する必要があります。 これは、共有内のすべての最上位のディレクトリとファイル、および共有への接続が、単一のFiles VMから提供されることを意味します

   **Distributed「分散共有」** ホームディレクトリ、ユーザープロファイル、およびアプリケーションフォルダーに適しています。 このタイプの共有は、すべてのFiles VM間で最上位のディレクトリを分割し、Filesクラスター内

   **Access Based Enumeration (ABE)「アクセスベースの列挙」** は、特定のユーザーが読み取りアクセス権を持つファイルとフォルダーのみがそのユーザーに表示されるようにします。 これは通常、Windowsファイル共有で有効です

   **Self Service Restore　「セルフサービスリストア」** ユーザーはWindowsの[以前のバージョン]を活用して、Nutanixスナップショットに基づいて個々のファイルを以前のリビジョンに簡単に復元できます

#. **次へ** 次へをクリックします

#. **Summary「概要」** を確認し、**Create「作成」**　 をクリックします

   .. figure:: images/16.png

   共有のテスト
   ..........

#. RDPまたはコンソールを介して *Initials*\ **-ToolsVM** に接続します

   .. note::

     Tools VMはすでに **NTNXLAB.local** ドメインに参加しています。 ドメインに参加しているVMを使用して、次の手順を実行できます

#. **ファイルエクスプローラー** で ``\\BootcampFS.ntnxlab.local\`` を開きます.

   .. figure:: images/17.png

#. 前の手順でダウンロードした **SampleData_Small.zip** ファイルを共有に展開して、マーケティング共有へのアクセスをテストします

   .. figure:: images/18.png

   - The **NTNXLAB\\Administrator** ユーザーは、Filesクラスターの展開中にファイル管理者として指定され、デフォルトですべての共有への読み取り/書き込みアクセス権を付与しました
   - 他のユーザーのアクセス管理は、他のSMB共有と同じです

#. **Marketing > Properties** を右クリックします

#. **Security「セキュリティ」タブを選択し、**Advanced「詳細」** をクリックします

   .. figure:: images/19.png

#. **Users「ユーザー」（BootcampFS\\Users)** を選択し、**削除** をクリックします

#.  **Add 「追加」** をクリックします

#. **Select a principal「プリンシパルを選択」** をクリックし、**Object Name「オブジェクト名」** フィールドに **Everyone** を指定します。 **OK** をクリックします

   .. figure:: images/20.png

#. 以下をフィールドに入力して、**OK** をクリックします

   - **Type「タイプ」** - Allow「許可」
   - **Applies to「適用先」** - This folder only　「このフォルダのみ」
   - **Read & execute「読み取りと実行」**　を選択します
   - **List folder contents「フォルダコンテンツのリスト」**　を選択
   - **Read「既読」**　を選択
   - **Write「書き込み」**　を選択

   .. figure:: images/21.png

#. *OK**> **OK**> **OK** をクリックして、権限の変更を保存します

　　これで、すべてのユーザーがMarketing共有内にフォルダーとファイルを作成できるようになります

　　多くの人々が利用するシェアでは、リソースの公平な使用を確保するためにクォータを活用するのが一般的です。 ファイルは、Active Directory内の個々のユーザー、または特定のActive Directoryセキュリティグループに対して、共有ごとにソフトまたはハードクォータを設定する機能を提供します

#. **Prism > File Server 「ファイルサーバー」> Share > Marketing**, で、**+ Add Quota Policy** をクリックします

#. 次のフィールドに入力して、**Save「保存」** をクリックします:

   - **Group「グループ」**　を選択
   - **User or Group「ユーザーまたはグループ」** - SSP Developers
   - **Quota「割り当」** で- 10 GiB
   - **Enforcement Type「施行タイプ」** - Hard Limit「ハード制限」

   .. figure:: images/22.png

#. **Save「保存」**　をクリックします

#. Marketing 「マーケティング」共有を選択した状態で、 **Share Details「共有の詳細」**, **Usage「使用状況」** and **Performance「パフォーマンス」** の各タブを確認して、共有ごとに利用可能なファイルと接続の数、時間経過によるストレージ使用率、レイテンシ、スル

   .. figure:: images/23.png
