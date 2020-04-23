.. _files_multiprotocol:

---------------------
Files：マルチプロトコル
---------------------

マルチプロトコル
++++++++++++++

この演習では、NFSもサポートするように既存のSMB共有を構成します。 マルチプロトコルアクセスを有効にするには、ユーザーマッピングを構成し、共有のネイティブプロトコルと非ネイティブプロトコルを定義する必要があります

ユーザーマッピングの構成
.....................

Nutanix Files共有には、ネイティブプロトコルと非ネイティブプロトコルの概念があります
すべての権限は、ネイティブプロトコルを使用して適用されます。 非ネイティブプロトコルを使用するアクセス要求には、ネイティブ側から適用される権限へのユーザーまたはグループのマッピングが必要です
ルールベースのマッピング、明示的なマッピング、デフォルトのマッピングなど、ユーザーとグループのマッピングを適用する方法はいくつかあります。 最初にデフォルトのマッピングを設定します

#. **Prism** で > **File Server「ファイルサーバ」** > file server 「ファイルサーバー」を選択し、 **Protocol Management「プロトコル管理」** をクリックして、**User Mapping「ユーザーマッピング」をクリックします

   .. figure:: images/53.png

#. **User Mapping「ユーザーマッピング」** ダイアログで **Default Mapping「デフォルトのマッピング」** ページが表示されるまで、**Next「次へ」** を少なくとも2回クリックします


#. **Default Mapping「デフォルトのマッピング」** mapping「マッピング」が見つからない場合のデフォルトとして、**Deny access to NFS export「NFSエクスポートへのアクセスを拒否」** と **Deny access to SMB share「SMB共有へのアクセスを拒否」**　の両方を選択します

   .. figure:: images/54.png

#. **Next「次へ」**　を選択して初期マッピングを完了し、**Save「概要」** ページで[保存]を選択します。

#. **Prism** で > **File Server「ファイルサーバー」** > **Share/Export「共有/エクスポート」** > Marketing「マーケティング」共有をクリックし、**Update「更新」** を選択します

#. **Basics 「基本」**　ページで、下部にある **Enable multiprotocol access for NFS「NFSのマルチプロトコルアクセスを有効にする」** チェックボックスをオンにします

   .. figure:: images/55.png

#. **Next [次へ]** をクリックし、**Settings「設定」** ページから **Simultaneous access to the same files from both protocols「両方のプロトコルから同じファイルへの同時アクセス」** をチェックします

   .. figure:: images/56.png

#. **Next「次へ」** をクリックし、**Save「概要」** ページから[保存]をクリックします。

#. SSH経由で *Initials*\ -NFS-Client VM に接続します

#. 次のコマンドを実行します:

     .. code-block:: bash

       [root@CentOS ~]# mkdir /filesmulti
       [root@CentOS ~]# mount.nfs4 BootcampFS.ntnxlab.local:/Marketing /filesmulti
       [root@CentOS ~]# dir /filesmulti
       dir: cannot open directory /filesmulti: Permission denied
       [root@CentOS ~]#

   .. 注意:: マウント操作では大文字と小文字が区別されます

デフォルトのマッピングはアクセス拒否なので、Permission deniedエラーが予想されます。 次に、ネイティブでないNFSプロトコルユーザーへのアクセスを許可する明示的なマッピングを追加します
明示的なマッピングを作成するには、ユーザーID（UID）を取得する必要があります

#. Execute the following command and take note of the UID:

     .. code-block:: bash

       [root@CentOS ~]# id
       uid=0(root) gid=0(root) groups=0(root) context=unconfined_u:unconfined_r:unconfined_t:s0-s0:c0.c1023
       [root@CentOS ~]#

#. **Prism** で> **File Server「ファイルサーバー」** > ファイルサーバーを選択し、**Protocol Management 「プロトコル管理」** をクリックし、次に**User Mapping 「ユーザーマッピング」** をクリックします

#. 明示的なマッピングページが表示されるまで **Next「次へ」** をクリックします

#. クリックして、**+ Add one-to-one mapping 「1対1のマッピング」** を追加します

#. 次のフィールドに入力します:

   - **SMB Name「SMB名」** - ntnxlab\\administrator
   - **NFS ID** - 前のステップのUID（rootの場合は0）
   - **User/Group「ユーザー/グループ-ユーザー」** - User

   .. figure:: images/57.png

#. **Actions「アクション」** 列の下の **Save「保存」** をクリックします。

#. **Summary「概要」** ページが表示されるまで **Next [次へ]** をクリックし、**Save「保存」** をクリックします

#. **Close「閉じる」** をクリックします

#. NFS-Client VMに戻り、以下を実行します:

     .. code-block:: bash

       [root@CentOS ~]# dir /filesmulti
       MyMovie.flv Sample\ Data
       [root@CentOS ~]#
