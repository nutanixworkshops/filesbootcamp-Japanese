.. _files_nfs_export:

-------------------------
Files:NFSエクスポートの作成
-------------------------

概要
++++

この演習では、クラスター化されたアプリケーションのサポート、ロギングなどのアプリケーションデータの格納、またはLinuxクライアントが一般的にアクセスする他の非構造化ファイルデータの格納に使用されるNFSv4エクスポートを作成およびテストします。

NFSエクスポートの使用
+++++++++++++++++

NFSプロトコルを有効にする
.....................

.. 注意::

   NFSプロトコルの有効化は、ファイルサーバーごとに1回だけ実行する必要があり、環境内ですでに完了している場合があります。 NFSがすでに有効になっている場合は、「ユーザーマッピングの構成」に進みます

#. **Prism Element > File Server 「ファイルサーバーでファイルサーバー」 ** を選択し、**Protocol Management> Directory Services** をクリックします

   .. figure:: images/29.png

#. 管理されていないユーザー管理と認証で** (Use NFS Protocol) NFSプロトコル ** を使用を選択し、**Update「更新」** をクリックします

   .. figure:: images/30.png

エクスポートの作成
...............

#. **Prism > 「ファイルサーバー」File Server** で、**+ Share / Export** をクリックします

#. 次のフィールドに入力しますs:

   - **Name「名前」** - logs
   - **Description (Optional)「説明（オプション)」** - File share for system logs
   - **File Server「ファイルサーバ」** - BootcampFS
   - **Share Path (Optional)「共有パス（オプション)」** - 空白のままにします
   - **Max Size (Optional) 「最大サイズ（オプション)」** - 空白のままにします
   - **Select Protocol「プロトコルの選択」** - NFS

   .. figure:: images/24.png

#. **Next「次へ」** をクリックします

#. 次のフィールドに入力します:

   - **Enable Self Service Restore 「セルフサービスリストア」** を有効にするを選択します
      - これらのスナップショットは、NFSクライアントの.snapshotディレクトリとして表示されます
   - **Authentication「認証」** - System
   - **Default Access (For All Clients) 「デフォルトのアクセス（すべてのクライアント用)」** - No Access「認証」
   - **+ Add exceptions**
   - **「読み取り/書き込みアクセス権を持つ** - *クライアント-クラスターネットワークの最初の3オクテット*\ .*（例10.38.1\.*）
   .. figure:: images/25.png

   デフォルトでは、NFSエクスポートは、エクスポートをマウントするすべてのホストへの読み取り/書き込みアクセスを許可しますが、これは特定のIPまたはIP範囲に制限できます

#. **Next「次へ」** をクリックします

#. **Summary「概要」** を確認し、**Create「作成」**　をクリックします

エクスポートのテスト
.................

You will first provision a CentOS VM to use as a client for your Files export.

.. 注意:: If you have already deployed the :ref:`Linux Tools VM`を別のラボの一部としてすでに展開している場合は、代わりにこのVMをNFSクライアントとして使用できます

#. **Prism > VM > Table**, **+ Create VM「VMの作成」** をクリックします

#. 次のフィールドに入力します:

   - **Name「名前」** - *イニシャル*\ -NFS-Client
   - **Description「説明」** - ファイルNFSエクスポートをテストするためのCentOS VM
   - **vCPU(s)** - 2
   - **Number of Cores per vCPU「vCPUあたりのコア数」** - 1
   - **Memory「メモリ」** - 2 GiB
   - **Add New Disk「新しいディスクを追加　** を選択します
      - **Operation「操作」** - Clone from Image Service「イメージサービスからのクローン」
      - **Image「画像」** - CentOS
      - **Add「追加」**　を選択
   - Select **Add New NIC「新しいNICの追加」**　を選択します
      - **VLAN Name「名」** - Secondary
      - **Add「追加」**　を選択

#. **Save「保存」** をクリックします

#. **Initials-NFS-Client「イニシャル-NFS-Client」** を選択し、**Power on**　をクリックします。

#. PrismでVMのIPアドレスをメモし、次の資格情報を使用してSSH経由で接続します:

   - **Username「ユーザー名-ルート」** - root
   - **Password「パスワード-」** - nutanix/4u

#. 以下を実行します:

     .. code-block:: bash

       [root@CentOS ~]# yum install -y nfs-utils #This installs the NFSv4 client
       [root@CentOS ~]# mkdir /filesmnt
       [root@CentOS ~]# mount.nfs4 BootcampFS.ntnxlab.local:/ /filesmnt/
       [root@CentOS ~]# df -kh
       Filesystem                      Size  Used Avail Use% Mounted on
       /dev/mapper/centos_centos-root  8.5G  1.7G  6.8G  20% /
       devtmpfs                        1.9G     0  1.9G   0% /dev
       tmpfs                           1.9G     0  1.9G   0% /dev/shm
       tmpfs                           1.9G   17M  1.9G   1% /run
       tmpfs                           1.9G     0  1.9G   0% /sys/fs/cgroup
       /dev/sda1                       494M  141M  353M  29% /boot
       tmpfs                           377M     0  377M   0% /run/user/0
       BootcampFS.ntnxlab.local:/             1.0T  7.0M  1.0T   1% /afsmnt
       [root@CentOS ~]# ls -l /filesmnt/
       total 1
       drwxrwxrwx. 2 root root 2 Mar  9 18:53 logs

#. logsディレクトリが ``/filesmnt/logs`` にマウントされていることを確認します

#. VMを再起動し、エクスポートがマウントされていないことを確認します。 マウントを永続化するには、次のコマンドを実行して、マウントを ``/etc/fstab`` に追加します。

     .. code-block:: bash

       echo 'BootcampFS.ntnxlab.local:/ /filesmnt nfs4' >> /etc/fstab

#. 次のコマンドは、ランダムデータで満たされた100 MBの2MBファイルを ``/filesmnt/logs`` に追加します:

     .. code-block:: bash

       mkdir /filesmnt/logs/host1
       for i in {1..100}; do dd if=/dev/urandom bs=8k count=256 of=/filesmnt/logs/host1/file$i; done

#. Return to **Prism > File Server「ファイルサーバ」> Share > logs「ログ」** に戻り、パフォーマンスと使用状況を監視します

   使用率データは10分ごとに更新されるので注意してください
