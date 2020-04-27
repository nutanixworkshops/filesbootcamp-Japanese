.. role:: html(raw)
   :format: html

.. _peer:

------------------------
Peer Global File Service
------------------------

*この実習ラボの推定所要時間は60分です*

**このラボでは、Google Chrome、Apple Safari、またはMicrosoft Edgeをお勧めします**

概要
++++

非構造化データの爆発的な成長により、組織は新しい価値とインテリジェンスを引き出しながら、増え続けるデータのユニバースを効率的に保存、共有、管理、保護するソリューションを模索しています。 1993年以来、Peerソフトウェアは、オンプレミスおよびクラウドストレージの分散環境向けに最高のデータ管理とリアルタイムレプリケーションソリューションを構築することにより、これらの要件に注力してきました。

Peerの主力製品であるPeerグローバルファイルサービス（PeerGFS）は、統合されたファイルロックとグローバルにアクセス可能な名前空間を備えたエンタープライズクラスのレプリケーションテクノロジーを特徴としており、マルチサイト、マルチベンダー、マルチクラウドの導入を促進します。

PeerGFSは、さまざまな場所にいるユーザーとアプリケーションの高速ローカルデータアクセスを可能にし、バージョンの競合から保護し、データの可用性を高め、Nutanix FilesをレガシーNASプラットフォームと共存させて、既存の環境へのファイルの導入を容易にします

PeerソフトウェアとNutanixを組み合わせる主な使用例は次のとおりです:

- **Global File Sharing and Collaboration** - バージョンの整合性と高可用性を確保しながら、分散したチームに共有プロジェクトファイルへの高速ローカルアクセスを提供します
- **HA and Load Balancing for User and Application Data** - エンドユーザーデータとアプリケーションデータの高可用性と負荷分散を可能にします
- **Storage Interoperability** - クロスプラットフォームのサポートにより、Nutanix Filesと既存のNASおよびハイブリッドクラウドストレージシステムとの共存が可能になり、ファイルとオブジェクトのフォーマット間の共存も可能になります
- **Analysis and Migration** - リソースの計画、最適化、移行のために既存のストレージを分析します。 分析とリアルタイムのレプリケーションを組み合わせることで、中断の少ないデータ移行が可能になります

*それはどのように機能しますか？*

.. figure:: images/integration.png

ユーザーは左から右に向かって、パブリックLAN経由でNutanix Filesクラスター上のSMB共有とやり取りします。 これらの共有を介してFilesクラスターでSMBアクティビティが発生すると、Peerパートナーサーバー（Peerエージェントと呼ばれる）は、ファイルからファイルアクティビティモニタリングAPIを介して通知を受けます。 Peerエージェントは、SMB経由で更新されたコンテンツにアクセスし、1つまたは複数のリモートファイルサーバーまたはローカルファイルサーバー、あるいはその両方へのデータフローを容易にします

**このラボでは、Peer Global File Serviceを構成して、Nutanix Filesでアクティブ-アクティブファイルサービスソリューションを作成し、Nutanix FilesからNutanix Objectsにコンテンツを複製し、ファイルシステムアナライザーツールを使用していくつかのサンプルデータを分析します**

Lab Setup
+++++++++

   .. note::

    注意: このラボには :ref:`windows_tools_vm` が必要です


Files
.....

このラボでは、割り当てられたクラスターに既存のNutanix Filesをデプロイする必要があります。Peerグローバルファイルサービスで使用するNutanix Filesを構成する方法の詳細については、以下のNutanix Filesの構成のセクションを参照してください

Peer VMs
........

この演習では、3つの共有VMを使用します。これらのVMはすべて、割り当てられたクラスターですでに使用可能になっています

.. list-table::
   :widths: 20 40
   :header-rows: 1

   * - **VM Name「名」**
     - **Description「説明」**
   * - **PeerMgmt**
     - このサーバーはPeer管理センターを実行しています
   * - **PeerAgent-Files**
     - このサーバーはNutanix Filesクラスターを管理します
   * - **PeerAgent-Win**
     - このWindowsファイルサーバーはレプリケーションのターゲットとして使用されます

Nutanix Filesの構成
...................

Peerグローバルファイルサービスでは、Nutanix Filesとの間でレプリケーションをオーケストレーションするために、ファイルサーバー管理者アカウントとREST APIアクセスの両方が必要です

#. Nutanixクラスターの **Prism Element**（10.XX.YY.37など）にログインします

#. ドロップダウンナビゲーションからファイルサーバーに移動し、**BootcampFS** デプロイメントを選択します

#. **+ Share/Export 「+共有/エクスポート」 ** をクリックし、次のフィールドに入力します

   - **Name 「名前」** - *Initials*\ **-Peer**
   - **Description (Optional)  「説明（オプション）」** - Leave blank「空白のままにします」
   - **File Server 「ファイルサーバ」** - **BootcampFS**
   - **Share Path (Optional) 「共有パス（オプション）」** - Leave blank 「空白のままにします」
   - **Max Size (Optional) 「最大サイズ（オプション）」** - Leave blank 「空白のままにします」
   - **Select Protocol「プロトコルの選択」** - SMB

   .. figure:: images/5.png

#. **Next 「次へ」**, **Next「次へ」**, **Create「作成」** の順にクリックします

#. **Manage roles「役割の管理」**　をクリックします

   .. figure:: images/6.png

#. **Add admins 「管理者の追加」** で、**NTNXLAB\\Administrator** が　**File Server Admin 「ファイルサーバー管理者」** として既に追加されている必要があります。そうでない場合は、**+ New user 「+新しいユーザー」** をクリックして、**NTNXLAB\\Administrator** を追加します

   .. figure:: images/7.png

   .. note::

     注意：運用環境では、PeerのActive Directoryサービスアカウントを使用する可能性があります

#. **REST API access users 「REST APIアクセスユーザーの下」**で、**peer** カウントがすでに作成されているかどうかを確認します。そうでない場合は、**+ Add new user 「+新しいユーザーを追加」** をクリックし、次のフィールドに入力して、**Save「保存」** をクリックします

   - **Username ユーザー名** - peer

     *ユーザー名はすべて小文字でなければなりません*

   - **Password　パスワード** - nutanix/4u

   .. figure:: images/8.png

   .. note::

     注意：単一のNutanix AOSクラスタのすべての参加者は、同じ　**BootcampFS**　ファイルサーバーと　**peer**　APIアカウントを共有します

#. **Close「閉じる」** をクリックします

PeerAgent-Winでのテストデータのステージング
......................................

ラボのステージングの最後のステップは、Windowsファイルサーバーとして機能するPeerAgent-Winでサンプルデータを作成することです。Peerは、複数のFilesクラスター間だけでなく、ファイルと他のNASプラットフォームの混在間でも複製できます。このラボでは、Nutanix FilesクラスターとWindowsファイルサーバー間でレプリケーションを行います

#. 次の資格情報を使用して、RDP経由で *Initials*\ **-Windows-ToolsVM** に接続します

   - **Username 「ユーザー名」** - NTNXLAB\\Administrator
   - **Password 「パスワード」** - nutanix/4u

#. **File Explorer 「エクスプローラー」** を開き、``\\PeerAgent-Win\Data`` に移動します

#. **Sample Data 「フォルダーのコピー」** を作成します。以下に示すように、コピーの名前を *Initials*\ **-Data** に変更します

   .. figure:: images/2.png


Peer Management Center Webインターフェイスへの接続
...............................................

Peer管理センター（PMC）は、Peerグローバルファイルサービスの集中管理コンポーネントとして機能します。ファイルデータは保存されませんが、場所間の通信を容易にするため、接続性が最も良い場所に展開する必要があります。 PMCの単一の展開で、100以上のエージェント/ファイルサーバーを管理できます

このラボでは、Webインターフェイスを介して共有PMC展開にアクセスします

#. Firefox以外のブラウザー（Chrome、Edge、およびSafariはすべて機能します）を、*Initials*\ **-Windows-ToolsVM** VMまたはラップトップで開きます

#. *Initials*\ **-Windows-ToolsVM** VMでブラウザーを使用している場合は、``https://PeerMgmt:8443/hub`` にアクセスします

#. ラップトップでブラウザーを使用している場合は、Nutanixクラスターの **Prism Element**（例：10.XX.YY.37）にログインして、PeerMgmt VMのIPを見つけ、``https://IP-of-PeerMgmt-Server:8443/hub``

#. ログインを求められたら、次の資格情報を使用します:

   - **Username 「ユーザー名」** - admin
   - **Password 「パスワード」** - nutanix/4u

#. 接続が完了したら、**PeerAgent-Files** と **PeerAgent-Win** の両方が、PMC Webインターフェイスの左下の **Agents 「エージェント」** ビューに緑色で表示されていることを確認します

   .. figure:: images/pmc.png

Peerグローバルファイルサービスの概要
++++++++++++++++++++++++++++++++

Peerグローバルファイルサービスは、ジョブベースの構成エンジンを利用します。さまざまなファイル管理の課題への取り組みを支援するために、いくつかの異なるジョブタイプを利用できます。ジョブは次の組み合わせを表します:

- Peerエージェント.
- それらのエージェントによって監視されているファイルサーバー
- 各ファイルサーバー上のデータの特定の共有/ボリューム/フォルダ
- レプリケーション、同期、ロックに関連する各種設定

新しいジョブを作成するとき、さまざまなジョブタイプと各タイプを使用する理由を概説するダイアログが表示されます

使用可能なジョブタイプは次のとおりです:

- **Cloud Backup and Replication** - ボリューム全体のポイントインタイムリカバリをサポートするエンタープライズNASデバイスからパブリックおよびプライベートオブジェクトストレージへのリアルタイムレプリケーション。各ファイルは、オプションのバージョントラッキングを備えた単一の透明なオブジェクトとして保存されます
- **DFS-N Management** - 新規および既存のMicrosoft DFS名前空間を管理します。ファイルコラボレーションジョブやファイル同期ジョブと組み合わせて、DFSフェイルオーバーとフェイルバックを自動化できます
- **File Collaboration** - リアルタイム同期と分散ファイルロックを組み合わせることで、エンタープライズNASプラットフォーム、ロケーション、クラウドインフラストラクチャ、組織全体でグローバルコラボレーションとプロジェクト共有を強化します
- **File Replication** - エンタープライズNASプラットフォームから任意のSMB宛先への一方向のリアルタイムレプリケーション
- **File Synchronization** - エンタープライズNASプラットフォーム、ロケーション、クラウドインフラストラクチャ、および組織全体でユーザーおよびアプリケーションデータの高可用性を実現する多方向のリアルタイム同期

新しいファイルコラボレーションジョブの作成
++++++++++++++++++++++++++++++++++++

このセクションでは、**File Collaboration** に焦点を当てます

#. **PMC Web Interface 「インターフェイス」** で、[File]> [New Job]をクリックします

#. **File Collaboration　「ファイルコラボレーション」** を選択し、**Click「作成」** をクリックします

   .. figure:: images/17.png

#. ジョブの名前として *Initials*\  - **Collab** を入力し、 **OK** をクリックします

   .. figure:: images/18.png

FilesとPeerAgent-ファイル
........................

#. **Add 「追加」** をクリックして、PeerエージェントとNutanix Filesクラスターのペアリングを開始します

   .. figure:: images/19.png

#. **Nutanix Files** を選択し、**Next「次へ」** をクリックします

   .. figure:: images/20.png

#. **PeerAgent-Files** という名前のエージェントを選択し、**Next「次へ」**をクリックします。このエージェントはFilesクラスターを管理します

   .. figure:: images/21.png

#. **Storage Information 「ストレージ情報」** ページで、ストレージデバイスにアクセスするための資格情報を入力するように求められます。Filesクラスターを共有している別の参加者がすでにPeerラボを実施している場合は、次のように既存の　**Existing Credentials「認証情報」** を選択できます

   .. figure:: images/22a.png

   このクラスターでPeerラボを実行する最初の参加者である場合、新しい認証情報が自動的に選択されます。次のフィールドに入力します:

   - **Nutanix Files Cluster Name「クラスター名」** - BootcampFS

     *前の手順で選択したエージェントとペアになるFilesクラスターのNETBIOS名*

   - **Username「ユーザー名]** - peer

     *これは、ラボで以前に構成したFiles APIアカウントのユーザー名であり、すべて小文字でなければなりません*

   - **Password「パスワード-」** - nutanix/4u

     *Files APIアカウントに関連付けられているパスワード*

   - **Peer Agent IP Peerエージェント** - **PeerAgent-Files** IP Address

     * ファイルに組み込まれたファイルアクティビティ監視APIからリアルタイム通知を受信するエージェントサーバーのIPアドレス。このエージェントサーバーで使用可能なIPのドロップダウンリストから選択できます*

#. **Validate 「検証」** をクリックして、提供された資格情報を使用してAPI経由でファイルにアクセスできることを確認します

   .. figure:: images/22.png

   .. note::

     注意 : これらの資格情報を入力すると、この特定のエージェントを使用する新しいジョブを作成するときに再利用できます。次のジョブを作成するときは、このページで「既存の資格情報」を選択して、以前に構成された資格情報のリストを表示します

#. **Next「次へ」** をクリックします

#. **Browse「参照」** をクリックして、複製する共有を選択します。共有の下のサブフォルダーに移動することもできます

#. *Initials*\ **-Peer** 共有を選択し、**OK** をクリックします

   .. figure:: images/23.png

   .. note::

     注意 : Peerグローバルファイルサービスは、Nutanix Filesv3.5.1 以降でネストされた共有内でのデータのレプリケーションをサポートしています

   .. note::

     注意 : 選択できる共有またはフォルダは1つだけです。レプリケートする共有を追加するたびに、追加のジョブを作成する必要があります

#. **Finish 「完了」**をクリックします。これで、**PeerAgent-Files** とNutanix Filesのペアリングが完了しました

   .. figure:: images/24.png

PeerAgent-Win
..........

このラボの演習を簡略化するために、同じクラスターで実行されている2番目のPeerエージェントサーバーは、標準のWindowsファイルサーバーとして機能します。 Peerを使用してNutanix Filesクラスター間で共有をレプリケートできますが、その主な利点の1つは、NASプラットフォームの組み合わせを操作できることです。 これにより、Nutanix Filesで単一のサイトのみが更新された場合でもNutanix Filesの採用を促進できますが、コラボレーションまたは災害復旧をサポートするにはレプリケーションが必要です

#. ファイルと　`Files and PeerAgent-Files`　の手順1〜8を繰り返して、**PeerAgent-Win** をジョブに追加し、次の変更を行います

   - **Storage Platform「ストレージプラットフォーム」** - Windows File Server
   - **Management Agent「管理エージェント」** - PeerAgent-Win
   - **Path「パス」** - C:\\Data\\*Initials*\ **-Data**

   .. figure:: images/25.png

#. **Next「次へ」** をクリックします

コラボレーションジョブ構成の完了
...........................

Peerは、共有間のNTFSアクセス許可の同期を処理するための堅牢な機能を提供します:

- **Enable synchronizing NTFS security descriptors in real-time**

  *ファイルおよびフォルダーのアクセス許可への変更をリモートファイルサーバーにレプリケートする場合は、このチェックボックスをオンにします*

- **Enable synchronizing NTFS security descriptors with master host during initial scan**

  *最初のスキャンでファイルサーバー間で同期されていない権限を検索して複製する場合は、これを選択します。これには、エンジンが権限の不一致で勝者を選択できない状況を解決するために、マスターホストを選択する必要があります*

- **Synchronize Security Description Options**

  *(オプション）複製するNTFSアクセス許可の種類を選択します*

  - **Owner**

    *オブジェクトを所有するNTFS作成者-所有者（デフォルトでは、作成者）*

  - **DACL**

    *随意アクセス制御リストは、ファイルまたはフォルダーへのアクセス許可が割り当てられている、または拒否されているユーザーとグループを識別します*

  - **SACL**

    *管理者は、システムアクセスコントロールリストを使用して、保護されたファイルまたはフォルダへのアクセス試行をログに記録できます。監査に使用されます*

- **File Metadata Conflict Resolution**

  *2つ以上のサイト間に権限の不一致がある場合、マスターホストに関連付けられたファイルサーバーに設定された権限は、他のファイルサーバーの権限よりも優先されます*

#. この実習ラボでは、デフォルトの構成を受け入れて、*Next「次へ」** をクリックします**Next**.

   .. figure:: images/26.png

#. **Application Support 「アプリケーションサポート」** で、**Microsoft Office** を選択します

   Peerの同期およびロックエンジンは自動的に最適化され、選択したアプリケーションのいずれかを最適にサポートします

   .. figure:: images/27.png

#. **Finish「完了」** をクリックして、ジョブのセットアップを完了します

コラボレーションジョブの開始
++++++++++++++++++++++++

ジョブが作成されたら、同期とファイルロックを開始するためにジョブを開始する必要があります

#. In the **PMC Web Interface**, under **Jobs**, right-click on your newly created job, and then select **Start**

#. **PMC Web Interface 「PMC Webインターフェイス」** の **Jobs「ジョブ」** で、新しく作成したジョブを右クリックし、**Start「開始」** を選択します

   .. figure:: images/28.png

   ジョブが開始したとき:

   - すべてのエージェントとFilesのクラスター（または他のNASデバイス）への接続がチェックされます
   - リアルタイム監視エンジンが初期化されます
   - すべてのファイルサーバーが別のサーバーと同期していることを確認するために、バックグラウンドスキャンが開始されます

#. ジョブペインでジョブをダブルクリックして、ランタイム情報と統計を表示します

   .. note::

     注意 : Auto-update 「自動更新　」をクリックして、ファイルの複製が開始されるときにコンソールを定期的に更新します

   .. figure:: images/29.png

コラボレーションのテスト
+++++++++++++++++++++

同期が適切に機能していることを確認する最も簡単な方法は、Nutanix FilesとWindowsファイルサーバーのパスに対して別々のファイルエクスプローラーウィンドウを開くことです

.. note::

  注意　：エージェントサーバーVMを使用してテストしないでください。これらのサーバーからのすべてのアクティビティは自動的にフィルタリングされ、Nutanix Filesクラスターのオーバーヘッドが削減されます

#. 次の資格情報を使用して、RDP経由で *Initials*\ **-Windows-ToolsVM** に接続します

   - **Username 「ユーザー名」** - NTNXLAB\\Administrator
   - **Password 「パスワード」** - nutanix/4u

#. ファイルエクスプローラーを開き、Nutanix Files共有（例： ``\\BootcampFS\Initials-Peer`` ）を参照します。このウィンドウをデスクトップの左側にドラッグします

   ラボのセットアップ中にWindowsファイルサーバーにシードされたサンプルデータは、既にNutanix Filesに複製されていることに注意してください

   .. note::

     注意 : **Prism > File Server** で複製されたファイルを確認することもできます

#. 2番目のファイルエクスプローラーウィンドウを開き、Windowsファイルサーバー共有（例： ``\\PeerAgent-Win\Data\Initials-Data`` ）を参照します。このウィンドウをデスクトップの右側にドラッグします

   .. figure:: images/30.png

#. 左側のファイルエクスプローラーで、共有のルート内にコピーアンドペーストして、サンプルデータディレクトリの1つのコピーを作成します（以下を参照)

   .. figure:: images/31.png

   .. figure:: images/32.png

#. Nutanix Files共有で実行される変更は、ペアになっているエージェントに送信されます。エージェントは、これらのファイルとフォルダを他のサーバーに（およびその逆に）複製しやすくします

   .. figure:: images/33.png

#. ファイルロックをテストするには、Nutanix Files共有のルート内に新しいOpenDocumentテキストファイルを作成します（例：``\\BootcampFS\Initials-Peer`` ）

   .. figure:: images/34.png

#. ファイルに名前を付けます。数秒以内に、Windowsファイルサーバー共有の下に表示されます（例： ``\\PeerAgent-Win\Data\Initials-Data`` )

   .. figure:: images/35.png

#. OpenOffice WriterでNutanix Files共有の下にあるファイルを開きます。次に、``\\PeerAgent-Win\Data\Initials-Data`` で同じ名前のファイルを開きます。ファイルがロックされているという次の警告が表示されます

   .. figure:: images/36.png

   **おめでとうございます!** 2つのファイルサーバー間でレプリケートされたアクティブ-アクティブファイル共有を正常に展開しました。Peerを使用すると、この同じアプローチを活用して、サイト間のファイルコラボレーション、レガシーソリューションからNutanix Filesへの移行、またはビジネス継続性のために複数のサイトからユーザーデータとプロファイルにアクセスする必要があるVDIなどのユースケースの災害復旧をサポートできます

Nutanix Objectsの操作
+++++++++++++++++++++

Peerグローバルファイルサービスには、NASデバイスからオブジェクトストレージにデータをプッシュする機能が含まれています。上記のコラボレーションシナリオを強化するために使用されたものと同じリアルタイムレプリケーションテクノロジーを使用して、ポイントインタイムリカバリ用のオプションのスナップショット機能を備えたNutanix Objectsにデータをレプリケートすることもできます。すべてのオブジェクトは、他のアプリやサービスですぐに使用できる透過的な形式で複製されます

このラボセクションでは、**Nutanix Files** から **Nutanix Objects** にデータを複製するために必要な手順について説明します

Nutanix ObjectsのクライアントIPと認証情報を取得する
..............................................

データをObjectsに複製するには、オブジェクトストアのクライアントIPが必要であり、アクセスキーと秘密キーを生成する必要があります。前のラボからこの情報を既に入手している場合は、このセクションをスキップして、既存の情報を再利用できます

#. Nutanixクラスターの **Prism Central** （10.XX.YY.39 など）にログインし、**Servicesサービス** > **Objectsオブジェクト** に移動します

#. **Object Stores「オブジェクトストア」** セクションで、テーブルから適切なオブジェクトストアを見つけ、クライアントが使用しているIPをメモします

   .. figure:: images/clientip.png

#. **Access Keys「アクセスキー」** セクションをクリックし、ユーザーの追加をクリックして、資格情報の作成プロセスを開始します

   .. figure:: images/buckets_add_people.png

#. **Add people not in Active Directory　「Active Directoryに含まれていないユーザー」** を追加する]を選択し、電子メールアドレスを入力します

   .. figure:: images/buckets_add_people2.png

#. **Next「次へ」** をクリックします

#. **Download Keys 「キーのダウンロード」** をクリックして、**Access Key 「アクセスキー」** と **Secret Key 「シークレットキー」**を含む **.csv** ファイルをダウンロードします

   .. figure:: images/buckets_add_people3.png

#. **Close「閉じる」**　をクリックします.

#. テキストエディタでファイルを開きます.

   .. figure:: images/buckets_csv_file.png

   .. note::

     注意 ：　テキストファイルを開いたままにして、以下のセクションですぐに使用できるアクセスキーと秘密キーを用意します

新しいクラウドレプリケーションジョブの作成
....................................

このセクションでは、**Nutanix Files** から **Nutanix Objects** にデータを複製する **Cloud Backup and Replication 「クラウドバックアップおよびレプリケーションジョブ」** の作成に焦点を当てます

#. **PMC Web Interface 「PMC Webインターフェイス」** で、**File > New Job** をクリックします

   .. figure:: images/cloud1.png

#. **Cloud Backup and Replication「クラウドのバックアップとレプリケーション」** を選択し、**Create「作成」**をクリックします

#. ジョブの名前として *Initials*\  - **Replication to Objects**と入力し、**OK** をクリックします。

   .. figure:: images/cloud2.png

#. **Nutanix Files** を選択し、**Next** をクリックします

   .. figure:: images/cloud3.png

#. **PeerAgent-Files** という名前のエージェントを選択し、**Next** をクリックします。このエージェントはFilesクラスターを管理します

   .. figure:: images/cloud4.png

#. **Storage Information 「ストレージ情報」**　ページ に、2つのページのいずれかが表示されます。Filesクラスターを共有する別の参加者がすでにPeerラボを実施している場合は、ここに示すように、既存の認証情報を選択できます

   .. figure:: images/cloud5.png

   このクラスターでPeerラボを行う最初の参加者である場合は、次のフィールドに入力します:

   - **Nutanix Files Cluster Name「NutanixFilesのクラスター名」** - **BootcampFS**

     *前の手順で選択したエージェントとペアになるFilesクラスターのNETBIOS名*

   - **Username「ユーザー名」** - peer

     *これは、ラボで以前に構成したFiles APIアカウントのユーザー名であり、すべて小文字にする必要があります*

   - **Password「パスワード」** - nutanix/4u

     *Files APIアカウントに関連付けられているパスワード*

   - **Peer Agent IP「PeerエージェントIP」** - **PeerAgent-Files** IP Address

     *Filesに組み込まれたファイルアクティビティ監視APIからリアルタイム通知を受信するエージェントサーバーのIPアドレス。このエージェントサーバーで使用可能なIPのドロップダウンリストから選択できます*

#. **Validate「検証」** をクリックして、提供された資格情報を使用してAPI経由でFilesにアクセスできることを確認します

   .. figure:: images/cloud6.png

   .. note::

     注意：これらの資格情報を入力すると、この特定のエージェントを使用する新しいジョブを作成するときに再利用できます。次のジョブを作成するときは、このページで「既存の資格情報」を選択して、以前に構成された資格情報のリストを表示します

#. **Next「次へ」**　をクリックします

#. *Initials*\ **-Peer** 共有を選択し、**OK**　をクリックします

   .. figure:: images/cloud7.png

   .. note::

     注意：Peerグローバルファイルサービスは、Nutanix Filesv3.5.1以降でネストされた共有内でのデータのレプリケーションをサポートしています

   .. note::

     注意：**Cloud Backup and Replication「クラウドバックアップとレプリケーション」** を使用すると、1つのジョブに対して複数の共有やフォルダを選択できます

#. **File Filters 「ファイルフィルター」** ページで、選択した[デフォルトフィルター]および **Include Files Without Extensions「拡張子のないインクルードファイル」**　を確認し、**Next[次へ]**　をクリックします

   .. figure:: images/cloud8.png

#. **Destination　「宛先」** ページで、**Nutanix Objects**　を選択し、**Next「次へ」** をクリックします

   .. figure:: images/cloud9.png

#. **Nutanix Objects Credentials** ページで、次のフィールドに入力します:

   - **説明** -目的地に名前を付けます

     *これは、Objects資格情報構成の短い名前です*

   - **Access Key 「アクセスキー」**

     *Objectsアカウントに関連付けられたアクセスキー*

   - **Secret Key「秘密鍵」**

     *Objectsアカウントに関連付けられた秘密鍵*

   - **Service Point「サービスポイント」**

     * オブジェクトストアのクライアントアクセスIPアドレスまたはFDQN名*

   .. figure:: images/cloud10.png

      .. note::

     Refer to the `Getting Client IP and Credentials for Nutanix Objects`_ section above for the appropriate access and secret keys, as well as the Client IP of the object store.

#. **Validate「検証」**　をクリックして、提供された構成を使用してObjectsにアクセスできることを確認します

   .. figure:: images/cloud11.png

#. **Success「成功」**　ウィンドウで　**OK**　をクリックし、**Next「次へ」**　をクリックします

#. On the **Bucket Details「バケットの詳細」** ページで、**Automatically name　自動的に名前を付ける]** チェックボックスをオフにし、*initials*\ -**peer-objects**　の一意のバケット名を指定します

   .. figure:: images/cloud12.png

      .. note::

       The bucket name MUST be in all lower case.

#. **Replication and Retention Policy 「レプリケーションと保存ポリシー」** ページで、**Existing Policy 「既存のポリシー」** 、**Continuous Data Protection 「継続的なデータ保護」** を選択し、**Next「次へ」**をクリックします

   .. figure:: images/cloud13.png

#. **Miscellaneous Options「その他のオプション」、**Email Alerts「電子メールアラート」** 、および　**SNMP Alerts「SNMPアラート」**　ページで　**Next「次へ」**　をクリックします

#. **Confirmation 「確認」** 画面で構成を確認し、**Finish 「完了」**をクリックします

   .. figure:: images/cloud14.png

クラウドレプリケーションジョブの開始
................................

ジョブが作成されたら、複製を開始するためにジョブを開始する必要があります

#. **PMC Web Interface「PMC Webインターフェイス」** で、新しく作成したジョブを右クリックし、**Start「開始」**を選択します

   .. figure:: images/cloud15.png

#. ジョブペインで **Job「ジョブ」** をダブルクリックして、ランタイム情報と統計を表示します

   .. figure:: images/cloud16.png

   .. note::

     注意：**Auto-Update「自動更新」** をクリックして、ファイルの複製が開始されるときにコンソールを定期的に更新します

レプリケーションの確認
....................

   .. note::

    この演習では、:ref:`windows_tools_vm` が必要です

ファイルがNutanix Objectsに複製されたことを確認する最も簡単な方法は、*Initials*\ **-Windows-ToolsVM** でCyberduckツールを使用することです

#. 次の資格情報を使用して、RDP経由で **Initials*\ **-Windows-ToolsVM** に接続します

   - **Username 「ユーザー名」** - NTNXLAB\\Administrator
   - **Password 「パスワード」** - nutanix/4u

#. **Cyberduck**　を起動します（ウィンドウアイコン>下矢印> Cyberduckをクリックします）

   Cyberduckを更新するように求められたら、**Skip This Version「このバージョンをスキップ」** をクリックします .

#. **Open Connection「接続を開く」** をクリックします

   .. figure:: images/buckets_06.png

#. ドロップダウンリストから　**Amazon S3**　を選択します

   .. figure:: images/buckets_07.png

#. 前に作成したユーザーの次のフィールドに入力し、**Connect「接続」** をクリックします

   - **Server 「サーバー」**  - *Objects Client Used IP*
   - **Port 「ポートー」**  - 443
   - **Access Key ID「アクセスキーID」**  - *Generated When User Created「作成時に生成」*
   - **Password (Secret Key)「 パスワード（秘密鍵）」** - *Generated When User Created「ユーザー作成時に生成」*

      .. note::

       適切なアクセスキーと秘密キー、およびオブジェクトストアのクライアントIPについては、上記の　`Getting Client IP and Credentials for Nutanix Objects`_　と認証情報の取得セクションを参照してください

   .. figure:: images/buckets_08.png

#. **Always Trust「常に信頼する」**　チェックボックスをオンにして、**The certificate is not valid「証明書が無効です」** ダイアログボックスで　**Continue「続行」**をクリックします

   .. figure:: images/invalid_certificate.png

#. **Yes「はい」** をクリックして、自己署名証明書のインストールを続行します

#. 上記の適切なバケットセットに移動し、コンテンツが含まれていることを確認します

   .. figure:: images/cloud19.png

   **おめでとうございます!** Nutanix FilesとNutanix Objects間の複製の設定が完了しました！Peerを使用すると、この同じアプローチを活用して、オブジェクトベースのアプリやサービスとのファイルデータの共存や、オブジェクトに基づくエンタープライズNASデータのポイントインタイムリカバリなどのシナリオをサポートできます

既存の環境の分析
+++++++++++++++

.. note::

 注意:の演習では、:ref:`windows_tools_vm`　が必要です

ファイルサーバー環境の容量が記録的なペースで増加しているため、ストレージ管理者は、ユーザーやアプリケーションがこれらのファイルサーバー環境をどのように活用しているかを知らないことがよくあります。この事実は、新しいストレージプラットフォームに移行するときに明らかになります。ファイルシステムアナライザーは、パートナーが既存のファイルとフォルダーの構造を発見して分析し、計画と最適化を行うためのPeerソフトウェアのツールです

ファイルシステムアナライザーは、1つ以上の指定されたパスの非常に高速なスキャンを実行し、結果をAmazon S3にアップロードし、主要な情報を1つ以上のExcelワークブックにまとめ、ワークブックにアクセスするためのリンクを含むレポートを電子メールで送信します

このツールは主にパートナー向けであるため、ご意見やご感想をお待ちしております

**#_peer_software_ext** チャネルを介してSlackでコメントや提案を添えてご連絡ください

ファイルシステムアナライザーのインストールと実行
.........................................

#. 次の資格情報を使用して、RDP経由でI　*Initials*\ **-Windows-ToolsVM**　に接続します

   - **Username「ユーザー名」** - NTNXLAB\\Administrator
   - **Password「パスワード-」** - nutanix/4u

#. VM内で、File System Analyzer installer「ファイルシステムアナライザーインストーラー」をダウンロードします：https://www.peersoftware.com/downloads/fsa/13/FileSystemAnalyzer_win64.exe

#. インストーラーを実行し、**Immediate Installation 「即時インストール」** を選択します

   .. figure:: images/fsa1.png

   インストールが完了すると、File System Analyzer wizard 「ファイルシステムアナライザウィザード」が自動的に起動します

#. **Introduction 「概要画面」** には、ユーティリティによって収集および報告された情報の詳細が表示されます。**Next「次へ」** をクリックします

   .. figure:: images/fsa2.png

#. The **Contact Information** screen collects information used to organize the output of the File System Analyzer and to send the final reports. Fill out the following fields:

#. **Contact Information「連絡先情報」** 画面には、ファイルシステムアナライザーの出力を整理し、最終的なレポートを送信するために使用される情報が収集されます。次のフィールドに入力します:

   - **Company「会社」** – 会社名を入力します
   - **Location 「場所」** – ファイルシステムアナライザーを実行しているサーバーの物理的な場所を入力します。マルチサイト環境では、これは都市名または州名になります。データセンター名も機能します
   - **Project 「プロジェクト–」** – この分析を実行するプロジェクト名またはビジネス上の理由を入力します。これ（およびCompanyフィールドとLocationフィールド）は、最終的なレポートを整理するためにのみ使用されます
   - **Mode「モード–」** – 使用する操作モードを選択します–一般分析または移行準備。移行の準備は、ストレージシステム間の移行プロジェクトを準備するときに役立ちます。このモードでは、ファイルシステムの標準テレメトリを収集するだけでなく、既存のストレージシステムと新しいストレージシステムの両方のパフォーマンスをテストして、移行の潜在的なパフォーマンスとタイミングを測定することもできます。このラボでは、一般分析を使用します　
   - **Name/Phone/Title「名前/電話/タイトル」** – （オプション）名前と連絡先情報を入力します。
   - **Email 「電子メール–」** – 最終レポートの送信先の電子メールアドレスを入力します。複数のアドレスの場合は、コンマ区切りのリストを入力します
   - **Upload Region「アップロードのリージョン」** – 最終レポートのアップロードに使用するS3の場所をFile System Analyzer「ファイルシステムアナライザー」に指示するには、**US**、**EU**、または　**APAC**　を選択します

   .. note::
    .. raw:: html

      <strong><font color="red">注意 : 以下に示すウィザードページに、独自の詳細を入力してください。それ以外の場合、最終レポートは送信されません</font></strong>

   .. figure:: images/fsa3.png

#. **Next「次へ」** をクリックします

   File System Analyzer「ファイルシステムアナライザー」は、1つ以上のパスをスキャンするように構成できます。これらのパスは、ローカル（ ``D:\MyData`` など）またはリモートのUNCパス（ ``\\files01\homes1`` など）にすることができます

#. 次のパスを追加します:

   - ``C:\`` - ローカル C: drive of *Initials*\ **-Windows-ToolsVM**
   - ``\\BootcampFS\<Your Share Name>\`` - **Nutanix Files** で以前に作成された共有

   .. figure:: images/fsa4.png

     ファイルサーバーで使用可能な共有を検出する場合は、**Search「検索」** ボタンをクリックしてファイルサーバーの名前を入力します。ダイアログ内を右クリックして **Check All「すべてチェック」**を選択し、検出されたすべての共有を自動的に追加することもできます

   .. figure:: images/fsa4a.png

     **Log totals by owner「所有者ごとに合計をログに記録する」** オプションを選択すると、選択したスキャンパス内のすべてのファイルとフォルダーがその所有者に送信されます。この所有者情報は、バイト、ファイル、およびフォルダーごとに集計され、最終的なレポートに含まれます

#. **Next「次へ」** をクリックします

   **Start「スタート」** ボタンをクリックして、入力したパスのスキャンを開始します。すべてのスキャン、分析、アップロードが完了すると、次のようなステータスが表示されます:

   .. figure:: images/fsa5.png

#. **File System Analyzer** は、構成されたすべてのアドレスにレポートを電子メールで送信します。完全なレポートを表示するには、電子メールの詳細レポートの下に表示されているハイパーリンクをクリックします。複数のパスがスキャンされた場合は、すべてのパスにわたる累積レポートへのリンクも表示されます。

   .. figure:: images/fsa6.png

   .. note::

     注意:レポートのダウンロードリンクは、**24時間** のみアクティブです。期限切れのレポートにアクセスするには、Peerソフトウェアに連絡してください

   Some systems may open these workbooks in a protected mode, displaying this message in Excel:

   .. figure:: images/fsa8.png

   一部のシステムでは、これらのワークブックをプロテクトモードで開き、次のメッセージをExcelで表示します

サマリーレポート
..............

要約レポートには、スキャン対象として選択されたすべてのパスの全体的な統計情報と履歴情報が含まれています。概要レポートを開くと、次のようなワークシートが表示されます

.. figure:: images/fsa7.png

   各要約レポートには、次のワークシートの一部またはすべてが含まれている場合があります:

   - **InfoSheet** – この特定の実行に関する詳細。このページには、10進数（1 KBは1,000バイト）と2進数（1 KiBは1,024バイト）の両方の形式でフォーマットされた合計バイトも表示されます
   - **CollectiveResults** – スキャンされたすべてのパスのリストと、それぞれの高レベルの統計
   - **History-Bytes** – 各パスがスキャンされるたびに、バイト単位の履歴変更が含まれます
   - **History-Files** – 各パスがスキャンされるたびに、ファイルの総数の履歴変更が含まれます
   - **History-Folders** – 各パスがスキャンされるたびにフォルダーの総数の履歴変更が含まれます

.. note::

  注意:履歴ワークシートは、複数のスキャンを実行した後にのみ表示されます

ボリュームレポート
...............

ボリュームレポートは、スキャンされた特定のパスに関するより詳細な情報を提供します。ボリュームレポートを開くと、次のようなワークシートが表示されます

.. figure:: images/fsa7a.png

   各ボリュームレポートには、次のワークシートの一部またはすべてが含まれている場合があります:

   - **Overview「概要」** – スキャンされたパスに関する高レベルの統計を示す一連のピボットテーブルとグラフ
   - **InfoSheet** – この特定のスキャンに関する詳細。このページには、10進数（1 KBは1,000バイト）と2進数（1 KiBは1,024バイト）の両方の形式でフォーマットされた合計バイトも表示されます
   - **OverallStats** – スキャンされたフォルダーの全体的な統計。これには、合計バイト数、ファイル、フォルダなどが含まれます
   - **Analysis「分析」** – ピボットテーブルと、スキャンされたパスに関する追加の統計を強調表示する2つのチャートが含まれます
   - **History** – このボリュームの各スキャンの統計を表示します
   - **HistoryCharts** – このボリュームのファイル、フォルダー、およびバイトの履歴変更を示すグラフが含まれます
   - **HighSubFolderCounts** – 100を超える子ディレクトリを含むすべてのフォルダのリスト
   - **HighByteCounts** – 10GBを超える子ファイルデータを含むすべてのフォルダーのリスト
   - **HighFileCounts** – 10,000を超える子ファイルを含むすべてのフォルダーのリスト
   - **LargeFiles** – 10GB以上の検出されたすべてのファイルのリスト
   - **DeepPaths** – 15レベル以上の、検出されたすべてのフォルダーパスのリスト
   - **LongPaths** – 256文字以上の検出されたすべてのフォルダーパスのリスト
   - **ReparsePointsSummary** – ファイルやフォルダーに関係なく、検出されたすべての再解析ポイントの概要
   - **ReparsePoints** – 検出されたすべてのフォルダー再解析ポイントのリスト
   - **TimeAnalysis** – 経過時間ごとのファイル、フォルダー、およびバイトの合計の内訳
   - **LastModifiedAnalysis** – month; hour of the day; day of month; and day of year.
   - **CreatedAnalysis** – 過去1年間に1時間ごとに変更されたすべてのファイル、フォルダー、およびバイトのビュー。次に、これらの数値を合計して平均し、ファイル、フォルダ、およびバイトを次のように変更して表示します。月;一日の時間;月の日。と年の日
   - **LastAccessedAnalysis** – 過去1年間に1時間ごとに作成されたすべてのファイル、フォルダー、およびバイトのビュー。次に、これらの数値を合計して平均し、曜日、月、時間、日、年ごとに作成されたファイル、フォルダー、およびバイトを表示します
   - **TLDAnalysis** - 指定されたパスの直下にある各フォルダーのリストと、これらの各サブフォルダーの統計。ユーザーのホームディレクトリ環境では、これらの各サブフォルダーは異なるユーザーを表す必要があります
   - **TopTLDsByTotals** – 指定されたパスの直下にある各フォルダーのリストと、これらの各サブフォルダーの統計。ユーザーのホームディレクトリ環境では、これらの各サブフォルダーは異なるユーザーを表す必要があります
   - **TopTLDsByLastModBytes** – 過去1年間に変更されたほとんどのバイトに基づくトップ10のトップレベルディレクトリを示すピボットテーブルとチャート
   - **TopTLDsByLastModFiles** – 過去1年間に変更されたほとんどのファイルに基づくトップ10のトップレベルディレクトリを示すピボットテーブルとチャート
   - **LegacyTLDs** – 過去365日間に変更されたファイルを含まないすべてのトップレベルディレクトリのリスト
   - **TreeDepth** – フォルダ構造の各深度レベルで見つかったバイト、フォルダ、およびファイルの集計。移行前の分析を行うお客様にとって、緑色で表示される深度は、PeerSync移行のツリー深度設定の良い候補です
   - **FileExtInfo** – 総バイト数と総ファイル数でソートされたピボットテーブルを含む、検出されたすべての拡張機能のリスト
   - **FileAttributes** – 見つかったすべてのファイルとフォルダーの属性の概要
   - **SmallFileAnalysis** – 特定のサイズ以下で発見されたすべてのファイルのリスト。このページは、ディスク上の最小ファイルサイズが大きいストレージプラットフォームでの小さなファイルのストレージへの影響を見積もるのに役立ちます
   - **SIDCache** – 検出されたすべての所有者とSID文字列のリスト

.. note::

 注意：履歴ワークシートは、複数のスキャンを実行した後にのみ表示されます

上記の **LastModifiedAnalysis** ページのサンプルは次のとおりです:

.. figure:: images/fsa7b.png

Microsoft DFS名前空間との統合
+++++++++++++++++++++++++++

Peerグローバルファイルサービスには、Microsoft DFS名前空間（DFS-N）を作成および管理する機能が含まれています。このDFS-N統合をリアルタイムのレプリケーションおよびファイルロックエンジンと組み合わせると、PeerGFSは場所やストレージデバイスにまたがる真のグローバルネームスペースを強化します

DFS名前空間管理機能の一部として、PeerGFSは、障害が発生したファイルサーバーからユーザーを自動的にリダイレクトします。障害が発生したサーバーがオンラインに戻ると、PeerGFSはこのファイルサーバーを同期状態に戻し、ユーザーアクセスを再び有効にします。これは、VDI環境のユーザープロファイルとユーザーデータ共有にNutanix Filesを活用しようとするあらゆる展開に不可欠な障害復旧機能です

次のスクリーンショットは、DFS名前空間が管理されているPMCインターフェイスを示しています

.. figure:: images/dfsn.png

持ち帰り
+++++++

- Peer Global File Service 「Peerグローバルファイルサービス」　は、Nutanix Filesクラスタにアクティブ-アクティブレプリケーションを提供できる唯一のソリューションです

- Peerは、複数のレガシーNASプラットフォームもサポートし、混合環境内でのレプリケーションをサポートします。これにより、Nutanix Filesの採用と移行が容易になります

- PeerはMicrosoft分散ファイルサービス（DFS）名前空間を直接管理できるため、単一の名前空間を通じて複数のファイルサーバーを提供できます。これは、ファイル共有のための真のアクティブ-アクティブDRソリューションをサポートするための重要なコンポーネントです

- Peerは、ポイントインタイムリカバリ用のオプションのスナップショット機能を使用して、Nutanix Filesおよびその他のNASプラットフォームからNutanix Objectsにファイルを複製できます。すべてのオブジェクトは、他のアプリやサービスですぐに使用できる透過的な形式です

- Peerは、既存のファイルサーバーを分析するためのツールを提供し、リソースの計画、最適化、最小限の中断による移行を支援します
