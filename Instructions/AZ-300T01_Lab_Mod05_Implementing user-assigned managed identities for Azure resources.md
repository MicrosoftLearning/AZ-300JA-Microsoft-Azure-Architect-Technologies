# 管理　ID

# ラボ：Azure リソースのユーザー割り当てマネージド ID　の実装

### シナリオ

アダタムコーポレーションは、マネージド ID を使用して Azure VM で実行されているアプリケーションを認証したいと考えています。

### 目的

この課題を終了すると、下記ができるようになります。

- ユーザーが割り当てたマネージド ID の作成と構成

- ユーザーが割り当てたマネージド ID の機能の検証

### ラボのセットアップ

予想時間: 30 分

ユーザー名: **学生**

パスワード: **Pa55w.rd**

## 演習 1: ユーザーが割り当てたマネージド ID の作成と構成。

このエクササイズの主なタスクは次のとおりです：

1. Windows Server 2016 データセンターを開き、Azure VM をデプロイします

1. ユーザーが割り当てたマネージド ID を作成します。

1. ユーザーが割り当てたマネージド ID を Azure VM に割り当てます。

1. ユーザーが割り当てたマネージド ID　へ　Grant RBAC　に基づくアクセス許可を付与します。

#### タスク 1: Windows Server 2016 データセンターを開き、Azure VM をデプロイします

1. ラボの仮想マシンから Microsoft Edge を起動し、[**http://portal.azure.com**](http://portal.azure.com) で Azure portalを参照 し、ターゲット Azure サブスクリプションの所有者ロールを持つ Microsoft アカウントを使用してサインインします。

1. Azure portal、Microsoft Edge ウィンドウに、**Cloud Shell** 内で 「**Bash**」 セッションを開始します。

1. **マウントされたストレージがない**というメッセージが表示された場合は、次の設定を使用してストレージを構成します：

   - サブスクリプション：ターゲット　Azure　サブスクリプションの名前

   - Cloud Shell リージョン: サブスクリプションに使用でき、ラボの場所に最も近い Azure リージョンの名前

   - リソース グループ：**az3000500-LabRG**

   - ストレージ アカウント： 新しいストレージ アカウントの名前

   - ファイル共有: 新しいファイル共有の名前

1. Cloud Shell　ペインから、リソース グループを作成するためには、(`<Azure region>`プレースホルダを、サブスクリプションに使用でき、ラボの場所に最も近い　Azure リージョンの名前に置き換えます)を実行します。

   ```
   az group create --resource-group az3000501-LabRG --location <Azure region>
   ```

1. Cloud Shell　ペインから、Azure リソース マネージャー テンプレート **\\allfile\\AZ-300T01\\Module_05\\azuredeploy05.json** を主目次にアップロードします。

1. Cloud Shell　ペインから、パラメータ ファイル **\\allfile\\AZ-300T01\\Module_05\\azuredeploy05.parameters.json** を主目次にアップロードします。

1. Cloud Shell　ペインから、下記を実行して、Windows Server 2016 データセンターをホストする Azure VM　を最初の仮想ネットワークにデプロイします：

   ```
   az deployment group create --resource-group az3000501-LabRG --template-file azuredeploy05.json --parameters @azuredeploy05.parameters.json
   ```

   > **注**: デプロイメントが完了するのを待ちます。これにはおよそ　5　分かかる場合があります。

#### タスク 2: ユーザー割り当てマネージド管理　ID　を作成し、Azure VM　に割り当てます。

1. Cloud Shell　ペインから、次の操作を実行して、ユーザー割り当て管理 ID を作成します：

   ```
   az identity create --resource-group az3000501-LabRG --name az3000501-mi
   ```

1. Cloud Shell　ペインから、次の操作を実行して、ユーザー割り当てたマネージド　ID を Azure VM に割り当てます：

   ```
   az vm identity assign --resource-group az3000501-LabRG --name az3000501-vm --identities az3000501-mi
   ```

#### タスク 3: ユーザーが割り当てたマネージド ID　を参照する　RBAC　を構成します。

1. Cloud Shell ウィンドウから、下記を実行して、新しいリソース グループを作成します (この演習で Azure VM をデプロイした Azure リージョンの名前に「`<Azure region>`」プレースホルダーを置き換えます)。

   ```
   az group create --resource-group az3000502-LabRG --location <Azure region>
   ```

1. Azure portal　で、新しく作成した **az3000502-LabRG** の **アクセス制御 (IAM)** ブレードに移動します。

1. **az3000502-LabRG - アクセス制御 (IAM)** ブレードから、新しく作成されたユーザー割り当てマネージド ID に所有者ロールを割り当てます。

> **結果**: ユーザー割り当てマネージド ID を作成および構成しました。

## 演習 2: ユーザーが割り当てた管理 ID の機能の検証

このエクササイズの主なタスクは次のとおりです：

1. ユーザーが割り当てたマネージド ID を利用して、認証用の Azure VM を構成します。

1. Azure VM からユーザーが割り当てた管理 ID の機能を検証します。

#### タスク 1: ユーザー割り当てマネージド ID を利用して、認証用の　Azure VM　を構成します。

1. Azure portal　で、**az3000501-vm** ブレードに移動します。

1. リモート デスクトップを使用して Azure VM に接続し、次の認証情報を提供して認証します：

   - ユーザー名：**student**

   - パスワード: **Pa55w.rd1234**

1. コマンドプロンプトで、**PowerShell** と入力して PowerShell セッションを開始します。

1. PowerShell　プロンプトから、次の操作を実行して、PowerShellGet モジュールの最新バージョンをインストールします (確認のプロンプトが表示されたら Enter キーを押します)。

   ```pwsh
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name PowerShellGet -Force -SkipPublisherCheck
   ```

1. PowerShell　プロンプトから、次の操作を実行して、Az モジュールの最新バージョンをインストールします (確認のプロンプトが表示されたら**Y**を入力し、Enter キーを押します)。

   ```pwsh
   Install-Module -Name Az -AllowClobber -SkipPublisherCheck
   ```

1. `exit`　を入力し、Enter キーを押し、現在の PowerShell　セッションを終了したあと、もう一度 `PowerShell`　セッションを開始します。

1. PowerShell プロンプトから、次の操作を実行して、AzureRM.ManagedServiceIdentity モジュールをインストールします (**Y** を入力し、確認を求められたら、Enter キーを押します)。

   ```pwsh
   [Net.ServicePointManager]::SecurityProtocol = [Net.SecurityProtocolType]::Tls12
   Install-Module -Name Az.ManagedServiceIdentity
   ```

#### タスク 2: Azure VM からユーザーが割り当てた管理 ID の機能を検証します。

1. PowerShell　プロンプトから、次の操作を実行して、ユーザーが割り当てた管理　ID としてサインインします：

   ```pwsh
   Add-AzAccount -Identity
   ```

1. PowerShell　プロンプトから、次の操作を実行して、現在使用されている管理　ID　の取得を試みます。

   ```pwsh
   (Get-AzVM -ResourceGroupName az3000501-LabRG -Name az3000501-vm).Identity
   ```

1. エラー メッセージを注意します。メッセージが述べているように、現在のセキュリティ コンテキストでは、リソースグループに十分な権限が与えられていません。この問題を解決するには、Azure portal　に切り替えて、**az3000501-LabRG** の **アクセス制御 (IAM)** ブレードに移動します。

1. **az3000501-LabRG - Access Control (IAM)** ブレードから、ユーザー割り当てマネージド ID **az3000501-mi** に **共同作成者ロール** を割り当てます。

1. リモート デスクトップ セッションに戻り、PowerShell　プロンプトから次の操作を実行して、現在使用されている管理　ID　の取得を試みます：

   ```pwsh
   (Get-AzVM -ResourceGroupName az3000501-LabRG -Name az3000501-vm).Identity
   ```

     >**注意**: PowerShell プロンプトから、権限が不十分であることを示すエラー メッセージを受け取った場合は、次を実行します
   
   ```pwsh
   Remove-AzAccount
   ```
   
   followed by:
   
   ```pwsh
   Add-AzAccount -Identity
   ```
      
1. PowerShell　プロンプトから、次の操作を実行して、変数に場所を保存します：

   ```pwsh
   $location = (Get-AzResourceGroup -Name az3000502-LabRG).Location
   ```

1. PowerShell　プロンプトから、次の操作を実行して、パブリック IP アドレス リソースを作成します：

   ```pwsh
   New-AzPublicIpAddress -Name az3000502-pip -ResourceGroupName az3000502-LabRG -AllocationMethod Dynamic -Location $location
   ```

1. コマンドが成功に完了したことを確認します。

> **結果**: このエクササイズでは、ユーザー割り当てマネージド　ID　の機能を検証しました。

## 演習 3: ラボリソースを削除

#### タスク 1: Cloud Shell を開く

1. ポータルの上部にある 「**Cloud Shell**」 アイコンをクリックして、Cloud Shell ペインを開きます。

1. **「Cloud Shell」** コマンド プロンプトで、次のコマンドを入力し、 **「Enter」** キーを押して、この実習ラボで作成したすべてのリソース グループを一覧表示します。

   ```
   az group list --query "[?starts_with(name,'az30005')].name" --output tsv
   ```

1. このラボで作成したリソース グループのみが出力に含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク 2: リソース グループを削除する

1. 「**Cloud Shell**」 コマンド プロンプトで、次のコマンドを入力し、**Enter** キーを押してこのラボで作成したリソース グループを削除する

   ```sh
   az group list --query "[?starts_with(name,'az30005')].name" --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. ポータルの下部にある 「**Cloud Shell**」 プロンプトを閉じます。

> **結果**: このエクササイズでは、このラボで使用するリソースを削除しました。
