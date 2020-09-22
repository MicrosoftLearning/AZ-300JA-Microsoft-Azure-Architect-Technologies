# ID のセキュリティ保護
# ラボ: カスタム ロール ベースのアクセス制御 (RBAC) ロールの実装
  
### シナリオ
  
Adatum Corporation は、Azure VM の開始と停止 (割り当て解除) にアクセス許可を委託するカスタム RBAC ロールを実装したいと考えています。


### 目的
  
この課題を終了すると、下記ができるようになります。

-  カスタム RBAC ロールの定義 

-  カスタム RBAC ロールの割り当て

### ラボの設定
  
予想時間: 30 分

ユーザー名：**学生**

パスワード: **Pa55w.rd**


## 演習 1: カスタム RBAC ロールの定義
  
この演習の主なタスクは次のとおりです。

1. Azure Resource Manager テンプレートを使用することで Azure VM をデプロイする

1. RBAC を介して委任するアクションを識別する

1. Azure AD テナントでカスタム RBAC ロールを作成する


#### タスク 1: Azure Resource Manager テンプレートを使用して Azure VM をデプロイします

1. ラボの仮想マシンから Microsoft Edge を起動し、[**http://portal.azure.com**](http://portal.azure.com) で Azure Portalを参照 し、ターゲット Azure サブスクリプションの所有者ロールを持つ Microsoft アカウントを使用することでサインインします。

1. Azure portal で、Microsoft Edge ウィンドウで、 **Cloud Shell** 内で **PowerShell** セッションを開始します。 

1. **ストレージにマウントされたメッセージがない**というメッセージが表示された場合は、次の設定を使用し、ストレージを構成します。

    - サブシプション: ターゲット Azure サブスクリプションの名前

    - Cloud Shell リージョン: サブスクリプションに使用でき、ラボの場所に最も近い Azure リージョンの名前

    - リソース グループ: 新しいリソース グループの名前 **az3000900-LabRG**

    - ストレージ アカウント: 新しいストレージ アカウントの名前

    - ファイル共有:新しいファイル共有の名前

1. Cloud Shell ウィンドウから、実行してリソース グループを作成します (`<Azure region>`プレースホルダーを、サブスクリプションに使用でき、ラボの場所に最も近い Azure リージョンの名前に置き換えます)。

   ```pwsh
   New-AzResourceGroup -Name az3000901-LabRG -Location <Azure region>
   ```

1. Cloud Shell ペインから、Azure Resource Manager テンプレート **\\allfiles\\AZ-300T03\\Module_04\\azuredeploy09.json** をホーム ディレクトリにアップロードします。

1. Cloud Shell ウィンドウから、パラメータ ファイルをアップロードします **\\allfiles\\AZ-300T03\\Module_04\\azuredeploy09.parameters.json** をホームディレクトリにアップロードします。

1. Cloud Shell パネルから、次の実行によって Ubuntu をホストする Azure VM をデプロイします。

   ```pwsh
   New-AzResourceGroupDeployment -ResourceGroupName az3000901-LabRG -TemplateFile $home/azuredeploy09.json -TemplateParameterFile $home/azuredeploy09.parameters.json
   ```

    > **注**: デプロイメントが完了するのを待つのではなく、次のタスクに進んでください。 

1. Azure portal に、Cloud Shell ペインを閉じます。


#### タスク 2: RBAC を通して委託するアクションを識別します

1. Azure portal　に、**az3000901-LabRG** ブレードに移動します。

1. **az3000901-LabRG** ブレードに、**アクセス制御 (IAM)** をクリックします。

1. **az3000901-LabRG - アクセス制御(IAM)** ブレードに、**ロール**をクリックします。

1. 「**ロール**」 ブレードで、「**所有者**」 をクリック します。

1. **所有者** ブレードに、**アクセス許可** をクリック します。

1. **アクセス許可 (プレビュー)** ブレードに、**Microsoft コンピューティング**をクリック します。

1. **Microsoft コンピューティング** ブレードに、**仮想マシン**をクリック します。

1. **仮想マシン**ブレードに、RBAC を使用して委託できる管理アクションの一覧を確認します。これには、**仮想マシンの割り出し解除**と **仮想マシンの開始**アクションが含まれます。


#### タスク 3: Azure AD テナントでカスタム RBAC ロールを作成する

1. ラボ コンピュータで ファイル **\\allfiles\\AZ-300T03\\Module_04\\customRoleDefinition09.json** を開き、その内容を確認します。

   ```json
   {
      "Name": "Virtual Machine Operator (Custom)",
      "Id": null,
      "IsCustom": true,
      "Description": "Allows to start and stop (deallocate) Azure VMs",
      "Actions": [
          "Microsoft.Compute/*/read",
          "Microsoft.Compute/virtualMachines/deallocate/action",
          "Microsoft.Compute/virtualMachines/start/action"
      ],
      "NotActions": [
      ],
      "AssignableScopes": [
          "/subscriptions/SUBSCRIPTION_ID"
      ]
   }
   ```

1. Azure portal で、Microsoft Edge ウィンドウで、 **Cloud Shell** 内で **PowerShell** セッションを開始します。 

1. Cloud Shell ウィンドウから、Azure Resource Manager テンプレート **\\allfiles\\AZ-300T03\\Module_04\\customRoleDefinition09.json** をホーム ディレクトリにアップロードします。

1. Cloud Shell ウィンドウ から、**$SUBSCRIPTION\_ID** プレースホルダを Azure サブスクリプションの ID 値に置き換えるには、次の手順を実行します。

   ```pwsh
   $subscription_id = (Get-AzContext).Subscription.id
   (Get-Content -Path $HOME/customRoleDefinition09.json) -Replace 'SUBSCRIPTION_ID', "$subscription_id" | Set-Content -Path $HOME/customRoleDefinition09.json
   ```
 
1. Cloud Shell ウィンドウから、次の操作を実行して、カスタム ロール定義を作成します。

   ```pwsh
   New-AzRoleDefinition -InputFile $HOME/customRoleDefinition09.json
   ```

1. Cloud Shell ウィンドウから、次の操作を実行して、ロールが正常に作成されたことを確認します。

   ```pwsh
   Get-AzRoleDefinition -Name 'Virtual Machine Operator (Custom)'
   ```

1. 「Cloud Shell」 ペインを閉じます。


> **結果**: このエクササイズを完了したら、カスタム RBAC ロールを定義しました。


## エクササイズ 2: カスタム RBAC ロールの割り当てとテスト
  
この演習の主なタスクは次のとおりです。

1. Azure AD ユーザーの作成

1. RBAC ロール割り当ての作成

1. RBAC ロール割り当てのテスト


#### タスク 1: Azure AD ユーザーの作成

1. Azure portal　に、Microsoft Edge ウィンドウに**Cloud Shell**内の**PowerShell**セッションを開始します。 

1. Cloud Shell ペインから次のコマンドを実行して、Azure AD テナントに対して明示的に認証します。

   ```pwsh
   Connect-AzureAD
   ```
   
1. Cloud Shell ペインから、次の操作を実行して、Azure AD DNS ドメイン名を識別します。

   ```pwsh
   $domainName = ((Get-AzureAdTenantDetail).VerifiedDomains)[0].Name
   ```

1. Cloud Shell ウィンドウから、次の操作を実行して、新しい Azure AD ユーザーを作成します。

   ```pwsh
   $passwordProfile = New-Object -TypeName Microsoft.Open.AzureAD.Model.PasswordProfile
   $passwordProfile.Password = 'Pa55w.rd1234'
   $passwordProfile.ForceChangePasswordNextLogin = $false
   New-AzureADUser -AccountEnabled $true -DisplayName 'lab user0901' -PasswordProfile $passwordProfile -MailNickName 'labuser0901' -UserPrincipalName "labuser0901@$domainName"
   ```

1. Cloud Shell ウィンドウから、次の操作を実行して、新しく作成された Azure AD ユーザーのユーザー プリンシパル名を識別します。

   ```pwsh
   (Get-AzureADUser -Filter "MailNickName eq 'labuser0901'").UserPrincipalName
   ```

1. 「Cloud Shell」 ペインを閉じます。


#### タスク 2: RBAC ロール割り当ての作成
 
1. Azure portal で、**az3000901-LabRG** ブレードに移動します。

1. **az3000901-LabRG** ブレードで、**アクセス制御 (IAM) をクリック** します。

1. **az3000901-LabRG - アクセス制御(IAM)** ブレードで、**+ 追加** をクリックして**ロール割り当てを追加**オプションを選択します。

1. **ロール割り当ての追加** ブレードで、次の設定を指定し、**保存** をクリック します。

    - ロール： **仮想マシンオペレータ (カスタム)**

    - アクセス権を割り当てる: **Azure AD ユーザー、グループ、またはサービス プリンシパル**

    - 選択: **lab user0901**


#### タスク 3: RBAC ロール割り当てのテスト

1. 新しい in-private Microsoft Edge ウィンドウを開始し、[**http://portal.azure.com**](http://portal.azure.com) で Azure portal を参照 し、新しく作成したユーザー アカウントを使用することでサインインします。

    - ユーザー名: このエクササイズの最初のタスクで識別したユーザー プリンシパル名

    - パスワード: **Pa55w.rd1234**

1. Azure portal で、**リソース グループ**ブレードに移動します。リソース グループは表示されません。 

1. Azure portal で、**すべてのリソース**ブレードに移動します。**az3000901-vm** とその管理ディスクのみが表示されます。

1. Azure portal で、**az3000901-vm** ブレードに移動します。仮想マシンを再起動してください。通知領域のエラーメッセージを確認し、現在のユーザーが実行を許可されていないため、この操作が失敗したことに注意してください。

1. 仮想マシンを停止し、アクションが正常に完了したことを確認します。

> **結果**: このエクササイズを完了したら、カスタム RBAC ロールを割り当て、テストしました。

## 演習 3: ラボリソースを削除

#### タスク 1: Cloud Shell を開く

1. ポータルの上部にある 「**Cloud Shell**」 アイコンをクリックして、Cloud Shell ペインを開きます。

1. 必要に応じて、Cloud Shell ペインの左上隅にあるドロップ ダウン リストを使用して、Bash シェル セッションに切り替えます。

1. **「Cloud Shell」** コマンドプロンプトで、次のコマンドを入力し、**「Enter」** キーを押して、このラボで作成したすべてのリソース グループを一覧表示します。

   ```
   az group list --query "[?starts_with(name,'az30009')]".name --output tsv
   ```

1. このラボで作成したリソース グループのみが出力に含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク 2: リソース グループを削除する

1. 「**Cloud Shell**」 コマンド プロンプトで、次のコマンドを入力し、**Enter** キーを押してこのラボで作成したリソース グループを削除する

   ```sh
   az group list --query "[?starts_with(name,'az30009')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. ポータルの下部にある 「**Cloud Shell**」 プロンプトを閉じます。

> **結果**: このエクササイズでは、このラボで使用するリソースを削除しました。
