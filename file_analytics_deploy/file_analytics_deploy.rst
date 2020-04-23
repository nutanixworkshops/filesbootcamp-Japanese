.. _file_analytics_deploy:

-----------------------
File Analytics: デプロイ
-----------------------

概要
++++

この演習では、ファイル分析 VMをデプロイし、既存の共有をスキャンしてダッシュボードを構築します。また、異常アラートを作成し、ファイルサーバーインスタンスの監査の詳細を表示します

ファイル分析をデプロイする
++++++++++++++++++++++

#. **Prism** > File Server 「ファイルサーバー」** で、**Deploy File Analytics 「ファイル分析を展開する」** をクリックします

   .. figure:: images/31.png

#. **Deploy「デプロイ」** を選択します

   時間を節約するために、ファイル分析 2.0.0パッケージはすでにクラスターにアップロードされています。ファイルバイナリは、Nutanixポータルからダウンロードして手動でアップロードできます

#. アップロードが完了したら、**Install インストール** を選択します

#. 詳細を記入してください

   - **Name「名前」** - Initials「-イニシャル」
   - **Storage Container「ストレージコンテナー」** – –ファイルサーバーインスタンスが使用するコンテナーを自動的に選択します
   - **Network List「ネットワークリスト–プライマリ」** – Primary - Managed

#. **Show Advanced Settings「詳細設定を表示」**　を選択します

#. **DNS Resolver IP「DNSリゾルバーIP」** がActive Directory、ntnxlab.local、ドメインコントローラー/ DNS IPアドレス

#. **Deploy**　を選択します

#. タスクページから展開を監視できます。 Analytics VMの展開には、約5分かかります

#. In **Prism** > **File Server「ファイルサーバー」** > **File Analytics「ファイル分析」** をクリックします

   .. figure:: images/33.png

#. **Enable File Analytics「ファイル分析の有効化」** ページで、ドメイン管理者を入力します。これは、ファイルサーバー管理者でもあります

   - **Username「ユーザー名」**: administrator
   - **Password「パスワード**: nutanix/4u

   .. figure:: images/34.png

#. **Enable「有効にする」**　を選択します
