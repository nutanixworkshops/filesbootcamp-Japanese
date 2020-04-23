.. _files_file_blocking:

------------------------
Files：ファイルブロッキング
------------------------

選択なファイルブロッキング
+++++++++++++++++++++++

この演習では、ファイルサーバーとマーケティング共有の特定のファイル拡張子をブロックするようにファイルを構成します

#. **Prism** で、**File Server ファイルサーバ** を選択し、[更新]をクリックして、[Blocked File Types]をクリックします。

   .. figure:: images/47.png

#. **Blocked File Types** の下に、.flv、.mov のような拡張子のコンマ区切りリストを入力し、**Save** をクリックします。

   .. figure:: images/48.png

#. タスクバーのPowerShellアイコンをクリックしてPowerShellウィンドウを開きます。 次のコマンドを入力すると、アクセス拒否のエラーメッセージが表示されます

   .. code-block:: bash

	    new-item \\xyz-files.ntnxlab.local\marketing\MyMovie.flv

   .. figure:: images/49.png

#. **Prism** で　**File Server「ファイルサーバ」** > **Share/Export「共有/エクスポート」** > Marketing share「マーケティング共有」 をクリックし、update「更新」を選択します

   .. figure:: images/50.png

#.  **Next「次へ」** を選択して **Settings「設定」** ページに移動します。

#. **Blocked File Types** をチェックし、ファイル拡張子として .none を入力します。

   .. figure:: images/51.png

#. **Next「次へ」**  ページで「次へ」 **Summary「保存」** の順に選択して、更新を完了します。

#. 共有レベルでブロックされたファイルタイプの設定は、サーバーレベルの設定を上書きします。 PowerShellを使用して、前のステップと同じコマンドを発行します。 これでコマンドは正常に完了します

   .. figure:: images/52.png
