# ストレージの実装と管理
# ラボ: Azure Storage アクセス制御の実装
  
### シナリオ
  
Adatum Corporation は、Azure Storage に存在するコンテンツを保護したいと考えています


### 目的
  
この課題を終了すると、下記ができるようになります。

-  Azure Storage アカウントを作成します。

-  データを Azure Storage にアップロードします

-  Azure Storage アクセス制御を実装します

### ラボの設定
  
予想時間: 30 分

ユーザー名: **学生**

パスワード: **Pa55w.rd**


## 演習 1: Azure Storage アカウントの作成と構成
  
このエクササイズの主なタスクは次のとおりです:

1. Azure でストレージ アカウントを作成する。

1. ストレージ アカウントのプロパティを表示する 


#### タスク 1: Azure でストレージ アカウントを作成する

1. ラボのバーチャルマシンから Microsoft Edge を起動し、[**http://portal.azure.com**](http://portal.azure.com) で Azure ポータルを拾い読み し、ターゲット Azure サブスクリプションの所有者ロールを持つ Microsoft アカウントを使用してサインインします。
  
1. Azure portal から、次のパラメーターで新しい SQL データベースを作成してください: 

    - サブスクリプション：お客様の Azure サブスクリプションの名称

    - リソースグループ： **az3000201-LabRG** という名前の新しいリソースグループ

    - ストレージ アカウント名: 小文字と数字で構成される 3 - 24 文字の有効で独特な名前

    - 場所: サブスクリプションに使用でき、ラボの場所に最も近い Azure リージョンの名前

    - パフォーマンス：**Standard**

    - アカウントの種類: **ストレージ (汎用 v1)**

    - レプリケーション: **ローカル冗長ストレージ (LRS)**
    
    - 安全な保護された転送が必要: **無効**

    - ネットワーク接続: **パブリック エンドポイント (すべてのネットワーク)**

    - BLOB ソフト削除: **無効**

    - Data Lake Storage Gen2: **無効**

1. ストレージ アカウントがプロビジョニングされるのを待ちます。これにはおよそ 1 分間かかります。


#### タスク 2: ストレージ アカウントのプロパティを表示する
  
1. Azure Portal　に、ストレージ アカウント ブレードを開き、場所、レプリケーション、パフォーマンスの設定など、**概要セクション**を確認します。

1. **アクセス キー** ブレードを表示します。アクセス キー ブレードには、key1 と key2 を含むストレージ アカウント名の値をコピーするオプションがあることに注意してください。また、両方のキーを再生成することもできます。

1. **構成**ブレードを表示します。

1. **構成**ブレードに、**汎用 v2** アカウントへのアップグレードを実行し、レプリケーション設定を変更するオプションがあることを注意してください。ただし、パフォーマンス設定を変更することはできません (これは、ストレージ アカウントの作成時にのみ割り当てることができます)。

> **結果**: この演習の完了後に、Azure Storage を作成し、そのプロパティを確認します。


## エクササイズ 2: BLOB の作成と管理
  
このエクササイズの主なタスクは次のとおりです。

1. コンテナーの作成

1. Azure portal を使用してコンテナーにデータをアップロードする

1. SAS トークンを使用して Azure Storage アカウントのコンテンツにアクセスします


#### タスク 1: コンテナーの作成
  
1. Azure portal で、前のタスクで作成したストレージ アカウントのプロパティを表示しているブレードに移動します。

1. ファイアウォール設定ブレードから、次の設定に従い、新しい　BLOB　コンテナーを作成してください：

    - 名前: **ラボ コンテナ**

    - アクセスの種類: **プライベート**


#### タスク 2: Azure portal を使用してコンテナーにデータをアップロードする
  
1. Azure portal　に **ラボ コンテナ**ブレードに移動します。

1. **ラボ コンテナ**ブレードから、ファイルをアップロードします。**C:\\Windows\\ImmersiveControlPanel\\images\\splashscreen.contrast-white_scale-400.png**。  


#### タスク 3: SAS トークンを使用して Azure Storage アカウントのコンテンツにアクセスする
  
1. **ラボコンテナ**ブレードから、新しくアップロードされた BLOB の URL を識別します。 

1. Microsoft Edge を起動し、その URL に移動します。

1. **ResourceNotFound**エラー メッセージに注意してください。BLOB がプライベート コンテナーに存在し、認証されたアクセスを必要としているため、これは予期されています。 

1. Azure Portal を表示する Microsoft Edge ウィンドウに切 り替え、**splashscreen.contrast-white_scale-400.png** ブレードで**SAS の生成**タブに切り替 えます。 

1. **SAS の生成** タブで **HTTP** オプションを有効に し、BLOB SAS トークンと対応する URL を生成します。

1. 新しい Microsoft Edge ウィンドウを開き、前の手順で生成された URL に移動します。

1. 画像を表示できることに注意してください。これは、URL に含まれる SAS トークンに基づいて BLOB にアクセスする権限が与えられたため、この時点で必要です。 

1. イメージを表示する Microsoft Edge ウィンドウを閉じます。


#### タスク 4: SAS トークンと保存されたアクセス ポリシーを使用して、Azure Storage アカウントのコンテンツにアクセスします。

1. Azure Portal で、**ラボコンテナ**ブレードに移動します。

1. **ラボコンテナー** ブレードから、 **ラボコンテナ -アクセスポリシー**ブレードに移動します。

1. 次の設定に新しいポリシーを追加する:

    - 識別子: **ラボコンテナー読**み取り

    - アクセス許可: **読み取り**

    - 開始時刻: 現在の日付と時刻

    - 有効期限: 現在の日付と時刻 + 24 時間

1. 「**OK**」 をクリックしてポリシーを作成し、「**保存**」 をクリックします。

1. Azure portal の Microsoft Edge ウィンドウで、「**Cloud Shell**」 内の 「**PowerShell**」 セッションを開始します。 

1. **マウントされたストレージがない**というメッセージが表示された場合は、次の設定を使用してストレージを構成します

    - サブスクリプション： ターゲット Azure サブスクリプションの名前

    - Cloud Shell リージョン： サブスクリプションに使用でき、課題の場所に最も近い Azure リージョンの名前

    - リソース グループ: **az3000201-LabRG**

    - ストレージアカウント：新しいストレージアカウントの名前

    - ファイル共有：新しいファイル共有の名前

1. Cloud Shell ペインから次のコマンドを実行して、このラボの最初の演習で作成したストレージ アカウント リソースを特定し、変数に保存します。

   ```pwsh
   $storageAccount = (Get-AzStorageAccount -ResourceGroupName az3000201-LabRG)[0]
   ```

1. Cloud Shell パネルから、次の操作を実行して、ストレージ アカウントに完全な制御を許可するセキュリティ コンテキストを実行します。

   ```pwsh
   $keyContext = $storageAccount.Context
   ```

1. Cloud Shell パネル、前のタスクで作成したアクセス ポリシーに基づいて BLOB 固有の SAS トークンを作成するには、次の操作を実行します。

   ```pwsh
   $sasToken = New-AzStorageBlobSASToken -Container 'labcontainer' -Blob 'splashscreen.contrast-white_scale-400.png' -Policy labcontainer-read -Context $keyContext
   ```

1. Cloud Shell パネルから、新しく作成された SAS トークンに基づいてセキュリティ コンテキストを確立するには、次の操作を実行します。 

   ```pwsh
   $sasContext = New-AzStorageContext $storageAccount.StorageAccountName -SasToken $sasToken
   ```

1. Cloud Shell パネルから、次の操作を実行して、BLOB のプロパティを取得します。 

   ```pwsh
   Get-AzStorageBlob -Container 'labcontainer' -Blob 'splashscreen.contrast-white_scale-400.png' -Context $sasContext
   ```

1. BLOB に正常にアクセスしたことを確認します。

1. Cloud Shell パネルを最小化します。


#### タスク 5: アクセス ポリシーを変更して SAS トークンを無効にします。

1. Azure Portal で、**ラボコンテナー - アクセス ポリシー**ブレードに移動します。

1. 開始 時刻と有効期限を昨日の日付に設定し、**既存のポリシーラボコンテナー読み取りを編集します**。 

1. Cloud Shell パネルを再度開きます。 

1. Cloud Shell パネルから、次の操作を再実行して、BLOB のプロパティの取得を試みます。 

   ```pwsh
   Get-AzStorageBlob -Container 'labcontainer' -Blob 'splashscreen.contrast-white_scale-400.png' -Context $sasContext
   ```

1. もうこれ以上BLOB にアクセスできないを確認します。


> **結果**: この演習の完了後に、BLOB コンテナーを作成し、そのコンテナーにファイルをアップロードし、SAS トークンと格納されたアクセス ポリシーを使用してアクセス制御をテストしました。

## 演習 3: ラボリソースを削除

#### タスク 1: Cloud Shell を開く

1. ポータルの上部にある 「**Cloud Shell**」 アイコンをクリックして、Cloud Shell ペインを開きます。

1. 必要に応じて、Cloud Shell ペインの左上隅にあるドロップ ダウン リストを使用して、Bash シェル セッションに切り替えます。

1. **「Cloud Shell」** コマンドプロンプトで、次のコマンドを入力し、**「Enter」** キーを押して、このラボで作成したすべてのリソース グループを一覧表示します。

   ```
   az group list --query "[?starts_with(name,'az30002')]".name --output tsv
   ```

1. このラボで作成したリソース グループのみが出力に含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク 2: リソース グループを削除する

1. 「**Cloud Shell**」 コマンド プロンプトで、次のコマンドを入力し、**Enter** キーを押してこのラボで作成したリソース グループを削除する

   ```sh
   az group list --query "[?starts_with(name,'az30002')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. ポータルの下部にある 「**Cloud Shell**」 プロンプトを閉じます。

> **結果**: このエクササイズでは、このラボで使用するリソースを削除しました。
