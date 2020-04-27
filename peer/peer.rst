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

   - **Server**  - *Objects Client Used IP*
   - **Port**  - 443
   - **Access Key ID**  - *Generated When User Created*
   - **Password (Secret Key)** - *Generated When User Created*

      .. note::

     See the `Getting Client IP and Credentials for Nutanix Objects`_ section above for the appropriate access and secret keys, as well as the Client IP of the object store.

   .. figure:: images/buckets_08.png

#. Check the **Always Trust** checkbox, and then click **Continue** in the **The certificate is not valid** dialog box.

   .. figure:: images/invalid_certificate.png

#. Click **Yes** to continue installing the self-signed certificate.

#. Navigate to the appropriate bucket set above and verify that it contains content.

   .. figure:: images/cloud19.png

   **Congratulations!** You have successfully setup replication between Nutanix Files and Nutanix Objects! Using Peer, this same approach can be leveraged to support scenarios including coexistence of file data with object-based apps and services as well as point-in-time recovery of enterprise NAS data backed by Objects.

Analyzing Existing Environments
++++++++++++++++++++++++++++++++++++++++++

   .. note::

   This exercise requires the :ref:`windows_tools_vm`.

As the capacity of file server environments increase at a record pace, storage admins often do not know how users and applications are leveraging these file server environments. This fact becomes most evident when it is time to migrate to a new storage platform. The File System Analyzer is a tool from Peer Software that is designed to help partners discover and analyze existing file and folder structures for the purpose of planning and optimization.

The File System Analyzer performs a very fast scan of one or more specified paths, uploads results to Amazon S3, assembles key pieces of information into one or more Excel workbooks, and emails reports with links to access the workbooks.

As this tool is primarily for our partners, we would love to hear any feedback you have on it. Reach out to us on Slack via the **#_peer_software_ext** channel with comments and suggestions.

Installing and Running the File System Analyzer
............

#. Connect to your *Initials*\ **-Windows-ToolsVM** via RDP using the following credentials:

   - **Username** - NTNXLAB\\Administrator
   - **Password** - nutanix/4u

#. Within the VM, download the File System Analyzer installer: https://www.peersoftware.com/downloads/fsa/13/FileSystemAnalyzer_win64.exe

#. Run the installer and select **Immediate Installation**.

   .. figure:: images/fsa1.png

   Once the installation is complete, the File System Analyzer wizard is automatically launched.

#. The **Introduction** screen provides details on information collected and reported by the utility. Click **Next**.

   .. figure:: images/fsa2.png

#. The **Contact Information** screen collects information used to organize the output of the File System Analyzer and to send the final reports. Fill out the following fields:

   - **Company** – Enter your company name.
   - **Location** – Enter the physical location of the server that is running the File System Analyzer. In multi-site environments, this could be a city or state name. A data center name also works.
   - **Project** – Enter a project name or business reason for running this analysis. This (and the Company and Location fields) are used solely to organize the final reports.
   - **Mode** – Select the mode of operation to be used – **General Analysis** or **Migration Preparation**. **Migration Preparation** is useful when preparing for a migration project between storage systems. In addition to collecting standard telemetry on file systems, this mode also offers the option to test performance of both the existing and new storage systems to help gauge potential migration performance and timing. For this lab, we will use **General Analysis**.
   - **Name/Phone/Title** – *(Optional)* Enter your name and contact information.
   - **Email** – Enter the email address to which the final reports will be sent. For multiple addresses, enter a comma-separated list.
   - **Upload Region** – Select **US**, **EU**, or **APAC** to tell the File System Analyzer which S3 location to use for uploading the final reports.

   .. raw:: html

     <strong><font color="red">Be sure to enter your own details into the wizard page shown below. Otherwise, the final report will not be sent to you.</font></strong>

   .. figure:: images/fsa3.png

#. Click **Next**.

   The File System Analyzer can be configured to scan one or more paths. These paths can be local (e.g., ``D:\MyData``) or a remote UNC Path (e.g., ``\\files01\homes1``).

#. Add the following paths:

   - ``C:\`` - The local C: drive of *Initials*\ **-Windows-ToolsVM**
   - ``\\BootcampFS\<Your Share Name>\`` - A share previously created on Nutanix Files

   .. figure:: images/fsa4.png

     Click the **Search** button and enter the name of a file server if you wish to discover the available shares on that file server. You can also right-click within the dialog and select **Check All** to automatically add all discovered shares.

   .. figure:: images/fsa4a.png

     Selecting the **Log totals by owner** option will poke every file and folder within the selected scan path(s) for its owner. This owner information will be tallied by bytes, files, and folders and included in the final report.

#. Click **Next**.

   Click the **Start** button to begin scanning the entered paths. When all scans, analyses, and uploads are complete, you will see a status that is similar to the following:

   .. figure:: images/fsa5.png

#. File System Analyzer will also email the report to all configured addresses. To view the full report, click the hyperlink(s) listed under **Detailed Reports** in the email. If multiple paths were scanned, you will also see a link to a cumulative report across all paths.

   .. figure:: images/fsa6.png

   .. note::

     Report download links are active for **24 hours** only. Contact Peer Software to access any expired reports.

   Some systems may open these workbooks in a protected mode, displaying this message in Excel:

   .. figure:: images/fsa8.png

   If you see this message at the top of Excel, click **Enable Editing** to fully open the workbook. If you do not do this, the pivot tables and charts will not load properly.

Summary Reports
............
Summary reports contain overall statistical and historical information across all paths that have been selected to be scanned.  When you open a summary report, you are greeted with a worksheet like this:

   .. figure:: images/fsa7.png

   Each summary report may contain some or all of the following worksheets:

   - **InfoSheet** – Details about this specific run. This page will also show Total Bytes formatted in both decimal (1 KB is 1,000 bytes) and binary (1 KiB is 1,024 bytes) forms.
   - **CollectiveResults** – A list of all paths scanned along with high-level statistics for each.
   - **History-Bytes** – Contains historical changes in bytes for each time each path is scanned.
   - **History-Files** – Contains historical changes in total number of files for each time each path is scanned.
   - **History-Folders** – Contains historical changes in total numbers of folders for each time each path is scanned.

    .. note::

     History worksheets will only appear after running multiple scans.

Volume Reports
............
Volume reports give more detailed information about a specific path that has been scanned. When you open a volume report, you are greeted with a worksheet like this:

   .. figure:: images/fsa7a.png

   Each volume report may contain some or all of the following worksheets:

   - **Overview** – A series of pivot tables and charts showing high-level statistics about the path that was scanned.
   - **InfoSheet** – Details about this specific scan. This page will also show Total Bytes formatted in both decimal (1 KB is 1,000 bytes) and binary (1 KiB is 1,024 bytes) forms.
   - **OverallStats** – Overall statistics for the folder that was scanned. This includes total bytes, files, folders, etc.
   - **Analysis** – Includes a pivot table and a pair of charts highlighting additional statistics about the path that was scanned.
   - **History** – Shows statistics from each scan of this volume.
   - **HistoryCharts** – Contains charts showing historical changes in files, folders, and bytes for this volume.
   - **HighSubFolderCounts** – A list of all folders containing more than 100 child directories.
   - **HighByteCounts** – A list of all folders containing more than 10GB of child file data.
   - **HighFileCounts** – A list of all folders containing more than 10,000 child files.
   - **LargeFiles** – A list of all discovered files that are 10GB or larger.
   - **DeepPaths** – A list of all discovered folder paths that are 15 levels deep or deeper.
   - **LongPaths** – A list of all discovered folder paths that are 256 characters or longer.
   - **ReparsePointsSummary** – A summary of all reparse points discovered, regardless of file or folder.
   - **ReparsePoints** – A list of all folder reparse points discovered.
   - **TimeAnalysis** – A breakdown of total files, folders, and bytes by age.
   - **LastModifiedAnalysis** – A view of all files, folders, and bytes modified each hour for the past year. These numbers are then totaled and averaged to show files, folders, and bytes modified by: day of week; month; hour of the day; day of month; and day of year.
   - **CreatedAnalysis** – A view of all files, folders, and bytes created each hour for the past year. These numbers are then totaled and averaged to show files, folders, and bytes created by day of week, month, hour of the day, day of month, and day of year.
   - **LastAccessedAnalysis** – A view of all files, folders, and bytes accessed each hour for the past year. These numbers are then totaled and averaged to show files, folders, and bytes accessed by: day of week; month; hour of the day; day of month; and day of year.
   - **TLDAnalysis** - A list of each folder immediately under a specified path with statistics for each of these subfolders. In a user home directory environment, each of these subfolders should represent a different user.
   - **TopTLDsByTotals** – A series of pivot tables and charts showing the top ten top-level directories based on total bytes used, total files, and total folders.
   - **TopTLDsByLastModBytes** – A pivot table and chart showing top 10 top-level directories based on most bytes modified in the past year.
   - **TopTLDsByLastModFiles** – A pivot table and chart showing top 10 top-level directories based on most files modified in the past year.
   - **LegacyTLDs** – A list of all top-level directories that do not contain any files modified in the past 365 days.
   - **TreeDepth** – A tally of bytes, folders, and files found at each depth level of the folder structure. For customers doing a pre-migration analysis, depths that appear as green are good candidates for PeerSync Migration’s tree depth setting.
   - **FileExtInfo** – A list of all discovered extensions, including pivot tables sorted by total bytes and total files.
   - **FileAttributes** – A summary of all file and folder attributes found.
   - **SmallFileAnalysis** – A list of all files discovered below a certain size. This page is useful for estimating the storage impact of small files on storage platforms that have large minimum file sizes on disk.
   - **SIDCache** – A list of all the owners and SID strings that have been discovered.

    .. note::

     History worksheets will only appear after running multiple scans.

Here is a sample of the **LastModifiedAnalysis** page mentioned above:

   .. figure:: images/fsa7b.png

Integrating with Microsoft DFS Namespace
++++++++++++++++++++++++++++++++++++++++

Peer Global File Service includes the ability to create and manage Microsoft DFS Namespaces (DFS-N). When this DFS-N integration is combined with its real-time replication and file locking engine, PeerGFS powers a true global namespace that spans locations and storage devices.

As part of its DFS namespace management capabilities, PeerGFS also automatically redirects users away from a failed file server. When that failed server comes back online, PeerGFS brings this file server back in-sync, and then re-enables user access to it. *This is an essential Disaster Recovery feature for any deployment looking to leverage Nutanix Files for user profile and user data shares for VDI environments.*

The following screenshot shows the PMC interface with a DFS Namespace under management.

.. figure:: images/dfsn.png

Takeaways
+++++++++

- Peer Global File Service is the only solution which can provide Active-Active replication for Nutanix Files clusters.

- Peer also supports multiple legacy NAS platforms and supports replication within mixed environments. This helps ease adoption of and migration to Nutanix Files.

- Peer can directly manage Microsoft Distributed File Services (DFS) namespaces, allowing multiple file servers to be presented through a single namespace. This is a key component for supporting true Active-Active DR solutions for file sharing.

- Peer can replicate files from Nutanix Files and other NAS platforms into Nutanix Objects with optional snapshot capabilities for point-in-time recovery. All objects are in a transparent format that can be immediately used by other apps and services.

- Peer offers tools for analyzing existing file servers to help with resource planning, optimization, and minimally disruptive migrations.
