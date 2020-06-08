.. title:: Nutanix Files Bootcamp

.. toctree::
  :maxdepth: 2
  :caption:  Files Labs
  :name: _files_labs
  :hidden:

  files_smb_share/files_smb_share
  files_nfs_export/files_nfs_export
  files_file_blocking/files_file_blocking
  files_multiprotocol/files_multiprotocol

.. toctree::
  :maxdepth: 2
  :caption:  File Analytics Labs
  :name: _file_analytics_labs
  :hidden:

  file_analytics_scan/file_analytics_scan
  file_analytics_anomaly/file_analytics_anomaly

.. toctree::
   :maxdepth: 2
   :caption: Bonus Labs
   :name: _bonus
   :hidden:

   peer/peer

.. toctree::
  :maxdepth: 2
  :caption: Optional Labs
  :name: _optional_labs
  :hidden:

  files_deploy/files_deploy
  file_analytics_deploy/file_analytics_deploy
  files_expand_cluster/files_expand_cluster

.. toctree::
  :maxdepth: 2
  :caption: Appendix
  :name: _appendix
  :hidden:

  appendix/glossary
  tools_vms/windows_tools_vm
  tools_vms/linux_tools_vm

.. _getting_started:

---------------
さぁ、はじめましょう Files Bootcamp
---------------

Nutanix Files Bootcamp へようこそ！
このワークブックには、Nutanix Files と多くの一般的な管理タスクを紹介する、インストラクター主導のセッションが含まれています。
各セクションには、実践的な演習を行うためのレッスンと演習があります。
インストラクターが演習を説明し、あなたが持つかもしれない追加の質問に答えます。
従来、ファイルストレージはITインフラにおける1つの[サイロ]であり、不必要な複雑さをもたらし、SAN ストレージによく見られるスケーラビリティと継続的な革新の欠如と同じ問題に悩まされています。
Nutanix は、Enterprise Cloud には[サイロ]の余地はないと考えています。
Nutanix Files は、ファイルストレージをアプリとしてアプローチし、実績のあるHCIコア上でソフトウェアを実行することにより、1-Click による高いパフォーマンス、スケーラビリティ、迅速なイノベーションを実現します。
このラボでは、SMB 共有とNFSエクスポートを管理する手順を実行し、この環境をスケールアウトし、ファイル機能を使って探索をします。
またこのラボでは、展開、構成、およびユースケースに関する重要な考慮事項について説明します。

What's New
++++++++++

- このワークショップは次のソフトウェアバージョンに更新されています：
    - AOS & PC 5.11.2.x
    - Files 3.6.1.2
    - File Analytics 2.1.0

- オプショナルラボの更新:

アジェンダ
++++++++

- Nutanix Files ラボ
    - Files: SMB共有の作成
    - Files: NFSエクスポートの作成
    - Files: 選択的なファイルブロッキング
    - File Analytics: 初期スキャンの確認
    - File Analytics: 異常ルール

- ボーナスラボ
    - Peer Software

- オプショナルラボ
    - Files: デプロイ
    - Files: クラスター拡張
    - File Analytics: デプロイ

はじめに
+++++++

- 名前
- Nutanixの知識

初期設定
+++++++

- 使用されている*パスワード*をメモします
- 仮想デスクトップにログインします（接続情報は以下にあります）

環境の詳細
+++++++++

- Nutanixワークショップは、Nutanix Hosted POC環境で実行することを目的としています
- 演習を完了するために必要なすべての必要なイメージ、ネットワーク、VMがクラスターにプロビジョニングされてます

ネットワーキング
+++++++++++++

ホストされたPOCクラスターは、標準の命名規則に従います:

- **クラスター名** - POC\ *XYZ*
- **サブネット** - 10.**21**.\ *XYZ*\ .0
- **クラスターIP** - 10.**21**.\ *XYZ*\ .37

マーケティングプールからプロビジョニングされた場合:

- **クラスター名** - MKT\ *XYZ*
- **サブネット** - 10.**20**.\ *XYZ*\ .0
- **クラスターIP** - 10.**20**.\ *XYZ*\ .37

例:

- **クラスター名** - POC055
- **サブネット** - 10.21.55.0
- **クラスターIP** - 10.21.55.37

ワークショップ全体を通して、XYZをサブネットの正しいオクテットに置き換える必要がある複数のインスタンスがあります。次に例を示します

.. list-table::
   :widths: 25 75
   :header-rows: 1

   * - IP アドレス
     - 解説
   * - 10.21.\ *XYZ*\ .37
     - Nutanix Cluster Virtual IP
   * - 10.21.\ *XYZ*\ .39
     - **PC** VM IP, Prism Central
   * - 10.21.\ *XYZ*\ .40
     - **DC** VM IP, NTNXLAB.local Domain Controller

各クラスターは、VMに使用できる2つのVLANで構成されています:

.. list-table::
  :widths: 25 25 10 40
  :header-rows: 1

  * - ネットワーキング 名
    - ネットワーキング アトレス
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

クレデンシャル
............

.. 注意::

  *<クラスタパスワード>*は各クラスタ固有であり、ワークショップのリーダーによって提供されます。

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - クレデンシャル
     - ユーザー名
     - パスワード
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

各クラスターには専用ドメインコントローラーVM・DCがあり、NTNXLAB.localドメインにADサービスを提供します。
ドメインには、次のユーザーとグループが用意されています。

.. list-table::
   :widths: 25 35 40
   :header-rows: 1

   * - グループ
     - ユーザー名
     - パスワード
   * - Administrators
     - Administrator
     - nutanix/4u
   * - SSP Admins
     - adminuser01-adminuser25
     - nutanix/4u
   * - SSP Developers
     - devuser01-devuser25
     - nutanix/4u
   * - SSP Consumers
     - consumer01-consumer25
     - nutanix/4u
   * - SSP Operators
     - operator01-operator25
     - nutanix/4u
   * - SSP Custom
     - custom01-custom25
     - nutanix/4u
   * - Bootcamp Users
     - user01-user25
     - nutanix/4u

アクセス手順
++++++++++

Nutanixホスト型POC環境には、さまざまな方法でアクセスできます:

ラボアクセスユーザー認証情報
.......................

PHXベースのクラスター:
**ユーザー名:** PHX-POCxxx-User01 (up to PHX-POCxxx-User20), **パスワード:** *<インストラクターが提供>*

RTPベースのクラスター:
**ユーザー名:** RTP-POCxxx-User01 (up to RTP-POCxxx-User20), **パスワード:** *<インストラクターが提供>*

Frame VDI
.........

ログイン: https://frame.nutanix.com/x/labs

- **Nutanix社員** - **NUTANIXDC**　資格情報を使用します
- **非従業員** - **ラボアクセスユーザー**　資格情報を使用します

Parallels VDI
.................

PHXベースのクラスターログイン: https://xld-uswest1.nutanix.com

RTPベースのクラスターログイン: https://xld-useast1.nutanix.com

- **Nutanix社員** - **NUTANIXDC**　資格情報を使用します
- **非従業員** - **ラボアクセスユーザー**　資格情報を使用します

Employee Pulse Secure VPN
..........................

クライアントをダウンロードします:

PHXベースのクラスターログイン: https://xld-uswest1.nutanix.com

RTPベースのクラスターログイン: https://xld-useast1.nutanix.com

- **Nutanix社員** - **NUTANIXDC**　資格情報を使用します
- **非従業員** - **ラボアクセスユーザー**　資格情報を使用します

クライアントを設定します.

Pulse Secure Clientで、接続を**追加**します:

PHXの場合:

- **タイプ** - ポリシー保護（UAC）または接続サーバー
- **名前** - X-Labs - PHX
- **サーバーのURL** - xlv-uswest1.nutanix.com

RTPの場合:

- **タイプ** - Policy Secure (UAC) or Connection Server
- **名前** - X-Labs - RTP
- **サーバーのURL** - xlv-useast1.nutanix.com


Nutanixバージョン情報
+++++++++++++++++++

- **AHV バージョン** - AHV 20170830.337
- **AOS バージョン** - 5.11.2.3
- **PC バージョン** - 5.11.2.1
