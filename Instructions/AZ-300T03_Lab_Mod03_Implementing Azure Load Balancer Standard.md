# 高度な仮想ネットワーキングの実装
# ラボ: Azure Load Balancer 標準の実装
  
### シナリオ
  
Adatum Corporation は、Azure VM の受信トラフィックと送信トラフィックを送信する Azure Load Balancer Standard を実装したいと考えています。 


### 目的
  
この課題を終了すると、下記ができるようになります。

-  Azure Load Balancer Standard を使用することで、受信の負荷分散を実装する 

-  Azure Load Balancer 標準を使用し、送信 SNAT トラフィックを構成する 

### ラボのセットアップ
  
予想時間: 45 分

ユーザー名：**学生**

パスワード: **Pa55w.rd**


## 演習 1: Azure Load Balancer Standard を使用することで、受信の負荷分散と NAT を実装する 
  
このエクササイズの主なタスクは次のとおりです:

1. Azure リソース マネージャー テンプレートを使用し、可用性セットに Azure VM をデプロイする

1. Azure Load Balancer 標準のインスタンスを作成する

1. Azure Load Balancer 標準の負荷分散規則を作成する

1. Azure Load Balancer 標準の NAT 規則を作成する

1. Azure Load Balancer 標準のテスト機能


#### タスク 1: Azure Resource Manager テンプレートを使用し、可用性セットに Azure VM をデプロイする

1. ラボの仮想マシンから Microsoft Edge を起動し、[**http://portal.azure.com**](http://portal.azure.com) で Azure portalを参照 し、ターゲット Azure サブスクリプションの所有者ロールを持つ Microsoft アカウントを使用することでサインインします。

1. Azure portal で、Microsoft Edge ウィンドウで、 **Cloud Shell** 内で Bash セッションを開始 します。 

1. **ストレージにマウントされたメッセージがない**というメッセージが表示された場合は、次の設定を使用し、ストレージを構成します。

    - サブシプション: ターゲット Azure サブスクリプションの名前

    - Cloud Shell リージョン: サブスクリプションに使用でき、ラボの場所に最も近い Azure リージョンの名前

    - リソースグループ： **az3000800-LabRG** という名前の新しいリソースグループ

    - ストレージ アカウント: 新しいストレージ アカウントの名前

    - ファイル共有:新しいファイル共有の名前

1. Cloud Shell ウィンドウから、実行してリソース グループを作成します (`<Azure region>`プレースホルダーを、サブスクリプションに使用でき、ラボの場所に最も近い Azure リージョンの名前に置き換えます)。

   ```
   az group create --name az3000801-LabRG --location <Azure region>
   ```

1. From the Cloud Shell pane, upload the Azure Resource Manager template **\\allfiles\\AZ-300T03\\Module_03\\azuredeploy0801.json** into the home directory.

1. Cloud Shell パネルから、パラメータ ファイル をアップロードします **\\allfiles\\AZ-300T03\\Module_03\\azuredeploy0801.parameters.json** をホームディレクトリにアップロードします。

1. Cloud Shell パネルから、次の方法で Windows Server 2016 データセンターをホストする Azure VM のペアをデプロイします。

   ```
   az deployment group create --resource-group az3000801-LabRG --template-file azuredeploy0801.json --parameters @azuredeploy0801.parameters.json
   ```

    > **注**: 次のタスクに進む前に、デプロイを待ちます。これにはおよそ10 分かかる場合があります。

1. Azure portal で、Cloud Shell パネルを閉じます。


#### タスク 2: Azure Load Balancer Standard のインスタンスを作成する

1. Azure portal から、次の設定を持つ新しい Azure Load Balancer を作成してください:

    - サブシプション: ターゲット Azure サブスクリプションの名前

    - リソース グループ: **az3000801-LabRG**
    
    - 名前: **az3000801-lb**

    - リージョン: このエクササイズの前のタスクで Azure VM がデプロイされた Azure リージョンの名前
    
    - タイプ：**パブリック**

    - SKU: **Standard**

    - パブリック IP アドレス: **az3000801-lb-pip01** という名前の 「**新しい名前を作成**」 します

    - 可用性ゾーン：**ゾーン冗長**


#### タスク 3: Azure Load Balancer Standard の負荷分散規則を作成する

1. Azure portal で、新しくデプロイされた Azure Load Balancer **az3000801-lb** のプロパティを表示するブレードに移動します。

1. **az3000801-lb** ブレードで、**バックエンド プール** をクリックします。

1. **az3000801-lb - バックエンドプール** ブレードで、**+ 追加** をクリックします。

1. **バックエンド プールの追加** ブレードで、次の設定を指定し、**追加** をクリック します。

    - 名前: **az3000801-bepool**

    - 仮想ネットワーク: **az3000801-vnet (2VM)**

    - 仮想マシン: **az3000801-vm0** IP アドレス: **ipconfig1 (10.0.0.4)** または **ipconfig1 (10.0.0.5)**

    - 仮想マシン: **az3000801-vm1** IP アドレス: **ipconfig1 (10.0.0.5)** または **ipconfig1 (10.0.0.4)**

    > **注**: 仮想マシンの IP アドレスが逆の順序で割り当てられる可能性があります。 

    > **注**: 操作が完了するまでお待ちください。これは 1 分以上かからないようにしてください。

1. **az3000801-lb - バックエンドプールブレードに戻** り、**正常性プローブ** をクリックします。

1. **az3000801-lb - ヘルスプローブ** ブレードで、**+ 追加** をクリック します。

1. **正常性プローブの追加** ブレードで、次の設定を指定し、**OK**をクリックします。

    - 名前: **az3000801-healthprobe**

    - プロトコル: **TCP**

    - ポート: **80**

    - 間隔: **5**

    - 異常しきい値**2**

    > **注**: 操作が完了するまでお待ちください。これは 1 分以上かからないようにしてください。

1. **az3000801-lb に戻る - ヘルスプローブ** ブレードに戻り、**ロード バランシング規則** をクリック します。

1. **az3000801-lb - ロードバランシング規則** ブレードで、**+ 追加** をクリック します。

1. **負荷分散規則の追加** ブレードで、次の設定を指定し、**OK**をクリックします。

    - 名前: **az3000801-lbrule01**

    - IP バージョン: **IPv4**

    - フロントエンド IP アドレス: ドロップダウン リストから **LoadBalancedFrontEnd** に割り当てられたパブリック IP アドレスを選択します。

    - プロトコル: **TCP**

    - ポート: **80**

    - バックエンド ポート: **80**

    - バックエンド プール: **az3000801--bepool (2台の仮想マシン)**

    - ヘルスプローブ: **az3000801-healthprobe (TCP:80)**

    - セッション永続化: **なし**

    - アイドル タイムアウト (分): **4**

    - フローティング IP (ダイレクト サーバー リターン): **無効**

    > **注**: 操作が完了するまでお待ちください。これは 1 分以上かからないようにしてください。


#### タスク 4: Azure Load Balancer Standard の NAT 規則を作成する

1. Azure portal で、**az3000801-lb** ポンド ブレードで、**受信 NAT 規則** をクリック します。

1. **az3000801-lb -インバウンド NAT ルール** ブレードに **+ 追加** をクリックします。

1. **インバウンド NAT ルールの追加**ブレードに、次の設定を指定し、**OK**をクリックします。

    - 名前: **az3000801-vm0-RDP**

    - フロントエンド IP アドレス: ドロップダウン リストから **LoadBalancedFrontEnd** に割り当てられたパブリック IP アドレスを選択します

    - IP バージョン: **IPv4**

    - サービス**RDP**

    - プロトコル: **TCP**

    - ポート: **33890**

    - ターゲット仮想マシン: **az3000801-vm0**

    - ネットワーク IP 構成: **ipconfig1 (10.0.0.4)** または **ipconfig1 (10.0.0.5)**

    - ポート マッピング: **カスタム**

    - Floating IP (Direct Server Return): **無効**

    - ターゲットポート: **3389**

    > **注**: 操作が完了するまでお待ちください。これは 1 分以上かからないようにしてください。

1. **az3000801-lb に戻る -  受信 NAT 規則** ブレードをクリック し、**+ 追加** をクリックします。

1. **受信 NAT 規則 の追加** 規則 ブレードで、次の設定を指定し、**OK**をクリックします。

    - 名前: **az3000801-vm1-RDP**

    - フロントエンド IP アドレス: ドロップダウン リストから **LoadBalancedFrontEnd** に割り当てられたパブリック IP アドレスを選択します

    - IP バージョン: **IPv4**

    - サービス**RDP**

    - プロトコル: **TCP**

    - ポート: **33891**

    - ターゲット仮想マシン: **az3000801-vm1**

    - ネットワーク IP 構成: **ipconfig1 (10.0.0.5)** または **ipconfig1 (10.0.0.4)**

    - ポート マッピング: **カスタム**

    - Floating IP (Direct Server Return): **無効**

    - ターゲットポート: **3389**

    > **注**: 操作が完了するまでお待ちください。これは 1 分以上かからないようにしてください。


#### タスク 5: Azure Load Balancer Standard のテスト機能

1. Azure portal で、**az3000801-lb** ブレードに移動 し、**パブリック IP アドレスエントリ**の値をメモ します。

1. ラボのコンピューターから、Internet Explorer を起動し、以前の手順で識別されたIPアドレスまでブラウズしてください。

1. 既定値の**インターネット インフォメーション サービス歓迎** ページが表示されることを検証してください。 

1. ラボ コンピュータに**スタート**を右クリックし、**実行**を クリック し、**開く**テキスト ボックスから次を実行します(`<IP address>`プレースホルダをこのタスクに前述のパブリック IP アドレスに置き換えます)。

   ```
   mstsc /v:<IP address>:33890
   ```

1. プロンプトが表示された場合は、次の値を指定して認証します：

    - ユーザー名：**学生**

    - パスワード: **Pa55w.rd1234**

1. リモート デスクトップ セッション内 で、**ローカル サーバー** に切り替 え、サーバー マネージャーで表示し、**az3000801-vm0**　Azure VM に接続されていることを確認 します。

1. ラボ コンピュータに切り替え、**スタート**を右クリックし、**実行**をクリックし、**開く**テキスト ボックスから次を実行します(`<IP address>`プレースホルダをこのタスクに前に識別した IP アドレスに置き換えます）。

   ```
   mstsc /v:<IP address>:33891
   ```

1. プロンプトが表示された場合は、次の値を指定して認証します：

    - ユーザー名：**学生**

    - パスワード: **Pa55w.rd1234**

1. リモート デスクトップ セッション内 で、**ローカル サーバー** に切り替 え、サーバー マネージャーで表示し、**az3000801-vm1**　Azure VM に接続されていることを確認 します。

1. リモート デスクトップ セッション内の Windows PowerShell セッションを開始し、次の操作を実行して、現在のパブリック IP アドレスを確認します。

   ```pwsh
   Invoke-RestMethod http://ipinfo.io/json 
   ```

1. コマンドレットの出力を確認し、IP アドレスエントリがこのタスクで前に指定したパブリック IP アドレスと一致していることを確認します。

1. リモート デスクトップ セッションを開いたままにします。次の演習で使用します。



> **結果**: この演習を完了すると、Azure Load Balancer Standard の受信の負荷分散および NAT ルールを実装およびテストしたことになります。


## エクササイズ 2: Azure Load Balancer Standard を使用し、送信 SNAT トラフィックを構成する
  
このエクササイズの主なタスクは次のとおりです:

1. Azure リソース マネージャー テンプレートを使用することで、Azure VM を既存の仮想ネットワークにデプロイする

1. Azure Standard Load Balancer を作成し、送信 SNAT 規則を構成する

1. Azure Standard Load Balancer のアウトバウンド 規則のテスト


#### タスク 1: Azure Resource Manager テンプレートを使用することで、Azure VM を既存の仮想ネットワークにデプロイする

1. ラボの仮想マシンから Microsoft Edge を起動し、[**http://portal.azure.com**](http://portal.azure.com) で Azure portal を参照 し、ターゲット Azure サブスクリプションの所有者ロールを持つ Microsoft アカウントを使用することでサインインします。

1. Azure portal で、Microsoft Edge ウィンドウで、 **Cloud Shell** 内で **Bash** セッションを開始 します。 

1. Cloud Shell ウィンドウから、Azure Resource Manager テンプレート **\\allfiles\\AZ-300T03\\Module_03\\azuredeploy0802.json** をホーム ディレクトリにアップロードします。

1. Cloud Shell ウィンドウから、パラメータ ファイルをアップロードします **\\allfiles\\AZ-300T03\\Module_03\\azuredeploy0802.parameters.json** をホームディレクトリにアップロードします。

1. Cloud Shell パネルから、次の方法で Windows Server 2016 データセンターをホストする Azure VM のペアをデプロイします。

   ```
   az deployment group create --resource-group az3000801-LabRG --template-file azuredeploy0802.json --parameters @azuredeploy0802.parameters.json
   ```

    > **注**: 次のタスクに進む前に、デプロイを待ちます。これにはおよそ5分間かかる場合があります。

1. Azure portal　に、Cloud Shell　ペインを閉じます。


#### タスク 2: Azure Standard Load Balancer を作成し、送信 SNAT 規則を構成する

1. Azure portal で、Microsoft Edge ウィンドウで、 **Cloud Shell** 内で **Bash** セッションを開始 します。 

1. Azure Portal では、Cloud Shell パネルから次の操作を実行し、ロード バランサーの送信パブリック IP アドレスを作成します。

   ```
   az network public-ip create --resource-group az3000801-LabRG --name az3000802-lb-pip01 --sku standard
   ```

1. Azure portal で、Cloud Shell パネルから、次の操作を実行して、Azure Load Balancer 標準を作成します。

   ```sh
   LOCATION=$(az group show --name az3000801-LabRG --query location --out tsv)
   az network lb create --resource-group az3000801-LabRG --name az3000802-lb --sku standard --backend-pool-name az3000802-bepool --frontend-ip-name loadBalancedFrontEndOutbound --location $LOCATION --public-ip-address az3000802-lb-pip01
   ```

1. Cloud Shell パネルから、次の操作を実行して、送信規則を作成します。

   ```
   az network lb outbound-rule create --resource-group az3000801-LabRG --lb-name az3000802-lb --name outboundRuleaz30000802 --frontend-ip-configs loadBalancedFrontEndOutbound --protocol All --idle-timeout 15 --outbound-ports 10000 --address-pool az3000802-bepool
   ```

    > **注**: 操作が完了するまでお待ちください。これは 1 分以上かからないようにしてください。

1. Cloud Shell パネルを閉じます。

1. Azure portal で、Azure Load Balancer **az3000802-lb** のプロパティを表示するブレードに移動します。

1. **az3000802 -lb** ブレードで、**バックエンド プール** をクリックします。

1. **az3000802-lb - バックエンドプール** ブレードで、**az3000802-bepool** をクリックします。

1. **az3000802-bepool** ブレードで次の値を変更してから **保存する** を選びます。

    - 仮想ネットワーク: **az3000801-vnet (4VM)**

    - 仮想マシン: **az3000802-vm0** IP アドレス: **ipconfig1 (10.0.1.4)** または **ipconfig1 (10.0.1.5)**

    - 仮想マシン: **az3000802-vm1** IP アドレス: **ipconfig1 (10.0.1.5)** または **ipconfig1 (10.0.1.4)**

    > **注**: 操作が完了するまでお待ちください。これは 1 分以上かからないようにしてください。


#### タスク 3: 送信規則が有効であることを確認する
 
1. Azure portal で、**az3000802-lb** ブレードに移動 し、**パブリック IP アドレスエントリ**の値をメモ します。

1. ラボ コンピュータで、リモート デスクトップ セッションから **az3000801-vm0** まで、次の操作を実行し、**az3000802-vm0** にリモート デスクトップ セッションを開始します。

   ```
   mstsc /v:az3000802-vm0
   ```

1. プロンプトが表示された場合は、次の値を指定して認証します：

    - ユーザー名：**学生**

    - パスワード: **Pa55w.rd1234**

1. **az3000802-vm0** へのリモート デスクトップ セッション内で、Windows PowerShell セッションを開始し、次の操作を実行して、現在のパブリック IP アドレスを確認します。

   ```pwsh
   Invoke-RestMethod http://ipinfo.io/json 
   ```

1. コマンドレットの出力を確認し、IP アドレスエントリがこのタスクで前に指定したパブリック IP アドレスと一致していることを確認します。


> **結果**: この演習を完了すると、Azure Load Balancer Standard の送信ルールを構成およびテストしたことになります。 


## 演習 3: ラボリソースを削除

#### タスク 1: Cloud Shell を開く

1. ポータルの上部にある 「**Cloud Shell**」 アイコンをクリックして、Cloud Shell ペインを開きます。

1. 必要に応じて、Cloud Shell ペインの左上隅にあるドロップ ダウン リストを使用して、Bash シェル セッションに切り替えます。

1. **「Cloud Shell」** コマンドプロンプトで、次のコマンドを入力し、**「Enter」** キーを押して、このラボで作成したすべてのリソース グループを一覧表示します。

   ```
   az group list --query "[?starts_with(name,'az30008')]".name --output tsv
   ```

1. このラボで作成したリソース グループのみが出力に含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク 2: リソース グループを削除する

1. 「**Cloud Shell**」 コマンド プロンプトで、次のコマンドを入力し、**Enter** キーを押してこのラボで作成したリソース グループを削除する

   ```sh
   az group list --query "[?starts_with(name,'az30008')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. ポータルの下部にある 「**Cloud Shell**」 プロンプトを閉じます。

> **結果**: このエクササイズでは、このラボで使用するリソースを削除しました。
