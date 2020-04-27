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

#. その異常テーブルエントリの「保存]を**Save「選択」**します。

#. **+ Configure new anomaly「新しい異常」** を選択して構成し、次の設定で2番目のルールを作成します

   - **Events「イベント」**: Create
   - **Minimum Operation「最小操作 %」**: 1
   - **Minimum Operation Count「最小操作数」**: 10
   - **User「ユーザー」**: All Users 「すべてのユーザー」
   - **Type「タイプ」**: Hourly 毎時
   - **Interval「間隔」**: 1

#. その異常テーブルエントリの保存を **Save「選択」**します

   .. figure:: images/40.png

#. **Save「保存」** を選択して「異常ルールの定義」ウィンドウを終了します

サンプルデータの読み込み
+++++++++++++++++++++

#. Marketing共有のSample Dataフォルダーに移動してコピーし、そのフォルダーを同じ共有に貼り付けます

   .. figure:: images/42.png

#. 元のサンプルデータフォルダーを削除します

エラー状態の原因
++++++++++++++

#. 異常アラートが入力されるのを待つ間、許可拒否を作成します

   .. note::
    注意: Anomalyエンジンは30分ごとに実行されます。 この設定はファイル分析 VMから構成できますが、この変数の変更はこのラボの範囲外です


#. Marketing「マーケティング」共有にROという新しいディレクトリを作成します

#. **myfile.txt** という「hello world」などのテキストを含むテキストファイルをROディレクトリに作成します

#. **RO** フォルダーの **Properties「プロパティ」**に移動し、[セキュリティ]タブを選択します

#. **Advanced「詳細」**　を選択します

#. **Disable inheritance 「継承を無効にする」** を選択し、**Convert… 「変換...」** オプションを選択します

#. 次に、次のように **Everyone** 権限を追加します:

   - Read & Execute 「読み取りと実行」
   - List folder contents 「フォルダーの内容を一覧表示する」
   - Read 「読み取り」

   .. figure:: images/43.png

#. **OK** を選択して、もう一度 **OK** を選択します

#. PowerShellウィンドウを特定のユーザーとして開く

   - **Shift** キーを押しながらタスクバーの **PowerShell** アイコンを右クリックします
   - **Run as different user 「別のユーザー」** として実行を選択します

   .. figure:: images/44.png

#. 以下を入力

   - **User name　「ユーザー名」**　: Poweruser01
   - **Password　「パスワード」**　: nutanix/4u

#. ディレクトリをマーケティング共有と **RO** ディレクトリに変更します

     .. code-block:: bash

        cd \\xyz-files.ntnxlab.local\marketing\RO

#. 次のコマンドを実行します。最初のコマンドは成功し、2番目のコマンドは失敗します:

     .. code-block:: bash

        more .\myfile.txt
        rm .\myfile.txt

   .. figure:: images/45.png

#. 1分ほどすると、ダッシュボードと **Audit Trails [監査証跡]** ビューの両方にアクセス　**Permission Denials]「許可の拒否」** が表示されます。 ブラウザを更新する必要があるかもしれません

   .. figure:: images/46.png

   .. note:: 注意:キャパシティトレンドのダッシュボードパネルは24時間ごとに更新されます
