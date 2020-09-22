# クラウド向けの開発
# ラボ: メッセージベースの統合アーキテクチャの構成
  
### シナリオ
  
Adatum には、オンプレミスのファイルサーバーに定期的にアップロードされるファイルを処理するいくつかの Web アプリケーションがあります。ファイルのサイズはさまざまですが、最大 100 MB まで可能です。Adatum は、アプリケーションを Azure App Service または Azure Functions ベースのアプリに移行し、Azure Storage を使用してアップロードされたファイルのホストを検討しています。次の 2 つのシナリオをテストする予定です。

  - Azure Functions を通して、Azure Storage コンテナーにアップロードされた新しい Blob を自動的に処理します。

  - Event Grid を使用して、Azure Storage コンテナーにアップロードされた新しい Blob を参照する Azure Storage キューメッセージを生成します。

これらのシナリオは、大きなメッセージを送信、受信、および操作するときに、メッセージングベースのアーキテクチャに共通の課題に対処することを目的としています。大きなメッセージをメッセージキューに直接送信することはお勧めしません。使用するリソースと消費する帯域幅が大きくなるからです。加えて、メッセージングプラットフォームは通常、大量の小さなメッセージを処理するように微調整されているため、処理速度に悪影響を与えてしまいます。さらに、メッセージングプラットフォームは通常、処理できる最大メッセージサイズを制限しています。

考えられるソリューションの 1 つは、Azure Blob Store、Azure SQL、Azure Cosmos DB などの外部サービスにメッセージペイロードを保存し、保存されたペイロードへの参照を取得してから、その参照だけをメッセージバスに送信します。このアーキテクチャパターンは、「クレームチェック」として知られています。その特定のメッセージの処理に関心があるクライアントは、必要に応じて、取得した参照を使ってペイロードを取得できます。
Azure では、このパターンはさまざまな方法により、さまざまなテクノロジーを使用して実装できますが、通常、クレームチェックの生成を自動化し、メッセージバスにプッシュしてクライアントが使用するか、ペイロード処理を直接トリガーするイベントに依存します。このような実装に含まれる一般的なコンポーネントの 1 つは Event Grid です。Event Grid は、構成可能な期間（最大24時間）以内にイベントを配信するイベントルーティングサービスです。その後、イベントは破棄されるか、デッドレターになります。イベントコンテンツのアーカイブまたはイベントストリームの再生が必要な場合は、Event Hub への Event Grid サブスクリプションまたはメッセージを長期間保持できる Azure Storage のキューをセットアップすることにより、この要件を容易にすることができます。 

このラボでは、Azure Storage Blob サービスを使用して、処理するファイルを保存します。クライアントは、指定された Azure Blob コンテナーに共有するファイルをドロップするだけです。最初のエクササイズでは、ファイルは Azure Function によって直接消費され、サーバーレスの性質を活用します。また、Application Insights を利用してインストルメンテーションを提供し、ファイル処理の監視と分析を促進します。2番目のエクササイズでは、 Event Grid を使用して、リクエストチェックメッセージを自動的に生成し、Azure Storage キューに送信します。これにより、クライアントアプリケーションはキューをポーリングます。メッセージを取得してから、格納された参照データを使用して、Azure Blob Storage から直接ペイロードをダウンロードできます。 

最初のエクササイズの Azure Function が Blob Storage トリガーに依存していることに注意するべきです。次の要件を処理する場合は、 Blob ストレージトリガーではなく Event Grid トリガーを選択する必要があります。

  - Blob 専用ストレージアカウント： Blob ストレージアカウントは、 Blob の入力および出力のバインディングでサポートされていますが、 Blob トリガーでサポートされていません。Blob ストレージトリガーには、汎用ストレージアカウントが必要です。

  - 高スケール。高スケールは、100,000 を超える Blob が含まれるコンテナー、または1秒あたり100を超える Blob 更新があるストレージアカウントとして大まかに定義できます。

  - 待機時間の短縮。機能アプリが消費計画にあった場合、機能アプリがアイドル状態になっていると、新しい Blob の処理に最大10分の遅延が生じる可能性があります。この遅延を回避するには、 Event Grid トリガーを使用するか、常にオンの設定を有効にして App Service プランに切り替えてください。 

  - Blob 削除イベントの処理： Blob 削除イベントは、Blob ストレージトリガーではサポートされていません。


### 目的
  
この課題を終了すると、下記ができるようになります。

-  Azure Function App Storage Blob トリガーを構成および検証

-  Azure Event Grid サブスクリプションベースのキューメッセージングを構成および検証 

### ラボ
  
予想時間: 60 分


## エクササイズ 1: Azure Function App Storage Blob トリガーを構成および検証
  
このエクササイズの主なタスクは次のとおりです。

1. Azure Function App Storage Blob トリガーの構成

1. Azure Function App Storage Blob トリガーの検証


#### タスク 1: Azure Function App Storage Blob トリガーの構成

1. ラボ仮想マシンから Microsoft Edge を起動し、 [**http://portal.azure.com**](http://portal.azure.com) ポータルにアクセスします。ターゲット Azure サブスクリプションで所有者の役割を持つ Microsoft アカウントを使用してサインインします。

1. Azure ポータルの Microsoft Edge ウィンドウ内の**Cloud Shell** 内で **Bash** セッションを起動します。 

1. **ストレージがマウントされていません**というメッセージが表示された場合は、次の設定を使用してストレージを構成します。

    - サブスクリプション：ターゲット Azure サブスクリプションの名前

    - Cloud Shell リージョン：サブスクリプションで利用可能で、ラボの場所に最も近いAzureリージョンの名前

    - リソースグループ： **az300T0601-LabRG**

    - ストレージアカウント：新しいストレージアカウントの名前

    - ファイル共有：新しいファイル共有の名前

1. Cloud Shell ペインから次を実行して、このエクササイズで指定するリソースの名前のプレフィックスとして使用される擬似ランダムな文字列を生成します。

   ```sh
   export PREFIX=$(echo `openssl rand -base64 5 | cut -c1-7 | tr '[:upper:]' '[:lower:]' | tr -cd '[[:alnum:]]._-'`)
   ```

1. Cloud Shellペインから、次を実行して、このラボでリソースをプロビジョニングする Azure リージョンを指定します（プレースホルダー`<Azure region>`をターゲット Azure リージョンの名前に置き換え、リージョン名のスペースを削除します）:

   ```sh
   export LOCATION='<Azure_region>'
   ```

1. Cloud Shell ペインから次を実行して、このラボでプロビジョニングするすべてのリソースをホストするリソースグループを作成します。

   ```sh
   export RESOURCE_GROUP_NAME='az300T0602-LabRG'

   az group create --name "${RESOURCE_GROUP_NAME}" --location "$LOCATION"
   ```

1. Cloud Shell ペインから、次を実行して、Azure Storage アカウントと、Azure 機能によって処理される Blob をホストするコンテナーを作成します。

   ```sh
   export STORAGE_ACCOUNT_NAME="az300t06st2${PREFIX}"

   export CONTAINER_NAME="workitems"

   export STORAGE_ACCOUNT=$(az storage account create --name "${STORAGE_ACCOUNT_NAME}" --kind "StorageV2" --location "${LOCATION}" --resource-group "${RESOURCE_GROUP_NAME}" --sku "Standard_LRS")

   az storage container create --name "${CONTAINER_NAME}" --account-name "${STORAGE_ACCOUNT_NAME}"
   ```

    > **注**: 同じストレージアカウントは、独自の処理要件を容易にするために Azure 機能でも使用されます。実際のシナリオでは、この目的のために別のストレージアカウントを作成することを検討する必要があります。

1. Cloud Shell ペインから、次を実行して、Azure Storage アカウントの接続文字列 プロパティの値を格納する変数を作成します。

   ```sh
   export STORAGE_CONNECTION_STRING=$(az storage account show-connection-string --name "${STORAGE_ACCOUNT_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" -o tsv)
   ```

1. Cloud Shell ペインから次のコマンドを実行して、Azure Function 処理ブロブの監視をします。そのキーを変数に保存する Application Insights リソースを作成します。

   ```sh
   export APPLICATION_INSIGHTS_NAME="az300t06ai${PREFIX}"

   az resource create --name "${APPLICATION_INSIGHTS_NAME}" --location "${LOCATION}" --properties '{"Application_Type": "other", "ApplicationId": "function", "Flow_Type": "Redfield"}' --resource-group "${RESOURCE_GROUP_NAME}" --resource-type "Microsoft.Insights/components"

   export APPINSIGHTS_KEY=$(az resource show --name "${APPLICATION_INSIGHTS_NAME}" --query "properties.InstrumentationKey" --resource-group "${RESOURCE_GROUP_NAME}" --resource-type "Microsoft.Insights/components" -o tsv)
   ```

1. Cloud Shell ペインから、次を実行して、Azure Storage Blob の作成に対応するイベントを処理する Azure Function を作成します。

   ```sh
   export FUNCTION_NAME="az300t06f${PREFIX}"

   az functionapp create --name "${FUNCTION_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" --storage-account "${STORAGE_ACCOUNT_NAME}" --consumption-plan-location "${LOCATION}" --runtime "dotnet"
   ```

1. Cloud Shell ペインから、次を実行して、新しく作成された関数のアプリケーション設定を構成し、それを Application Insights および Azure Storage アカウントにリンクします。

   ```sh
   az functionapp config appsettings set --name "${FUNCTION_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" --settings "APPINSIGHTS_INSTRUMENTATIONKEY=$APPINSIGHTS_KEY" FUNCTIONS_EXTENSION_VERSION=~2

   az functionapp config appsettings set --name "${FUNCTION_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" --settings "STORAGE_CONNECTION_STRING=$STORAGE_CONNECTION_STRING" FUNCTIONS_EXTENSION_VERSION=~2
   ```

1. Azure ポータルに切り替えて、このタスクの前半で作成した Azure Function アプリのブレードに移動します。

1. Azure Function アプリブレードで、**Functions** をクリックし、**+ New function** をクリックします。 

1. **下のテンプレートとquickstartを選択**のブレードで、 **Azure Blob Storage トリガー**テンプレートをクリックします。

1. 「**インストールされていない拡張機能**」 ブレードで、「**インストール**」 をクリックします。インストールが完了すると、**続行**をクリックします。

    > **注**: Azure Functions V2 ランタイムは、「バインディング拡張機能」と呼ばれる Nuget パッケージの形式でバインディングを実装します（特に、Azure Storage Blob バインディングは Microsoft.Azure .WebJobs.Extensions.Storage パッケージを使用します）。 

1. **Azure Blob Storage** トリガーブレードで、次を指定し、**作成**をクリックして Azure Function 内に新しい function を作成します。

    - 名前: **Blob Trigger**

    - パス： **workitems / {name}**

    - ストレージアカウントの接続： **STORAGE_CONNECTION_STRING**

1. Azure Function アプリの **Blob Trigger** function ブレードで、run.csx ファイルのコンテンツを確認します。 

   ```csharp
   public static void Run(Stream myBlob, string name, ILogger log)
   {
       log.LogInformation($"C# Blob trigger function Processed blob\n Name:{name} \n Size: {myBlob.Length} Bytes");
   }
   ```

    > **注**: デフォルトでは、この関数は、新しい Blob の作成に対応するイベントを単純に記録するようにと構成されています。Blob 処理タスクを実行するには、このファイルのコンテンツを変更する必要があります。


#### タスク 2: Azure Function App Storage Blob トリガーの検証

1. 必要に応じて、 Cloud Shell で Bash セッションを再起動します。

1. Cloud Shell ペインから次を実行して、前のタスクで使用した変数を再入力します。

   ```sh
   export RESOURCE_GROUP_NAME='az300T0602-LabRG'

   export STORAGE_ACCOUNT_NAME="$(az storage account list --resource-group "${RESOURCE_GROUP_NAME}" --query "[0].name" --output tsv)"

   export CONTAINER_NAME="workitems"
   ```

1. Cloud Shell ペインから次を実行して、このタスクで以前に作成した Azure ストレージアカウントにテスト Blob をアップロードします。

   ```sh
   export STORAGE_ACCESS_KEY="$(az storage account keys list --account-name "${STORAGE_ACCOUNT_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" --query "[0].value" --output tsv)"

   export WORKITEM='workitem1.txt'

   touch "${WORKITEM}"

   az storage blob upload --file "${WORKITEM}" --container-name "${CONTAINER_NAME}" --name "${WORKITEM}" --auth-mode key --account-key "${STORAGE_ACCESS_KEY}" --account-name "${STORAGE_ACCOUNT_NAME}"
   ```

1. Azure ポータルで、前のタスクで作成した Azure Function アプリが表示するブレードに戻ります。

1. Azure Function アプリブレードで、**Function sentry** セクションの **Monitor** をクリックします。 

1. Blob のアップロードを表す単一のイベントエントリに注意してください。エントリをクリックして、**詳細を呼び出し** ブレードを表示します。

    > **注**: このエクササイズの Azure Function アプリは消費計画で実行されるため、 Blob のアップロードとトリガーされる関数の間に最大数分の遅延が生じる場合があります。App Service（Consumption ではなく）プランを使用して Function アプリを実装することにより、待ち時間を最小限に抑えることができます。


1. **モニター** ブレードに戻り、「**Application Insightsで実行**」 リンクをクリックします。

1. Application Insights ポータルで、自動生成された Kusto クエリとその結果を確認します。


## 演習 2: Azure Event Grid サブスクリプションベースのキューメッセージングを構成および検証 
  
このエクササイズの主なタスクは次のとおりです。

1. Azure Event Grid サブスクリプションベースのキューメッセージングを構成 

1. Azure Event Grid サブスクリプションベースのキューメッセージングを検証 


#### タスク 1: Azure Event Grid サブスクリプションベースのキューメッセージングを構成 

1. 必要に応じて、 Cloud Shell で Bash セッションを再起動します。

1. Cloud Shell ペインから、次の操作を実行して、サブスクリプションに eventgrid リソース プロバイダーを登録します。

   ```sh
   az provider register --namespace microsoft.eventgrid
   ```
  
1. Cloud Shell ペインから次を実行して、このエクササイズで指定するリソースの名前のプレフィックスとして使用される擬似ランダムな文字列を生成します。

   ```sh
   export PREFIX=$(echo `openssl rand -base64 5 | cut -c1-7 | tr '[:upper:]' '[:lower:]' | tr -cd '[[:alnum:]]._-'`)
   ```

1. Cloud Shell ペインから次を実行して、ターゲットリソースグループとその既存のリソースをホストしている Azure リージョンを特定します。 

   ```sh
   export RESOURCE_GROUP_NAME_EXISTING='az300T0602-LabRG'

   export LOCATION=$(az group list --query "[?name == '${RESOURCE_GROUP_NAME_EXISTING}'].location" --output tsv)

   export RESOURCE_GROUP_NAME='az300T0603-LabRG'

   az group create --name "${RESOURCE_GROUP_NAME}" --location $LOCATION
   ```

1. Cloud Shell ペインから次を実行して、このタスクで構成する Event Grid サブスクリプションで使用される Azure ストレージアカウントとそのコンテナーを作成します。

   ```sh
   export STORAGE_ACCOUNT_NAME="az300t06st3${PREFIX}"

   export CONTAINER_NAME="workitems"

   export STORAGE_ACCOUNT=$(az storage account create --name "${STORAGE_ACCOUNT_NAME}" --kind "StorageV2" --location "${LOCATION}" --resource-group "${RESOURCE_GROUP_NAME}" --sku "Standard_LRS")

   az storage container create --name "${CONTAINER_NAME}" --account-name "${STORAGE_ACCOUNT_NAME}"
   ```

1. Cloud Shell ペインから、次を実行して、Azure Storage アカウントの Resource Id プロパティの値を格納する変数を作成します。

   ```sh
   export STORAGE_ACCOUNT_ID=$(az storage account show --name "${STORAGE_ACCOUNT_NAME}" --query "id" --resource-group "${RESOURCE_GROUP_NAME}" -o tsv)
   ```

1. Cloud Shell ペインから次を実行して、このタスクで構成する Event Grid サブスクリプションによって生成されたメッセージを保存するストレージアカウントキューを作成します。

   ```sh
   export QUEUE_NAME="az300t06q3${PREFIX}"

   az storage queue create --name "${QUEUE_NAME}" --account-name "${STORAGE_ACCOUNT_NAME}"
   ```

1. Cloud Shell ペインから、次を実行して、Azure Storage アカウントの指定されたコンテナーへの Blob アップロードに応じて Azure Storage キューでメッセージの生成を促進する Event Grid サブスクリプションを作成します。

   ```sh
   export QUEUE_SUBSCRIPTION_NAME="az300t06qsub3${PREFIX}"

   az eventgrid event-subscription create --name "${QUEUE_SUBSCRIPTION_NAME}" --included-event-types 'Microsoft.Storage.BlobCreated' --endpoint "${STORAGE_ACCOUNT_ID}/queueservices/default/queues/${QUEUE_NAME}" --endpoint-type "storagequeue" --source-resource-id "${STORAGE_ACCOUNT_ID}"
   ```


#### タスク 2: Azure Event Grid サブスクリプションベースのキューメッセージングを検証 

1. Cloud Shell ペインから次を実行して、このタスクで以前に作成した Azure ストレージアカウントにテスト Blob をアップロードします。

   ```sh
   export AZURE_STORAGE_ACCESS_KEY="$(az storage account keys list --account-name "${STORAGE_ACCOUNT_NAME}" --resource-group "${RESOURCE_GROUP_NAME}" --query "[0].value" --output tsv)"

   export WORKITEM='workitem2.txt'

   touch "${WORKITEM}"

   az storage blob upload --file "${WORKITEM}" --container-name "${CONTAINER_NAME}" --name "${WORKITEM}" --auth-mode key --account-key "${AZURE_STORAGE_ACCESS_KEY}" --account-name "${STORAGE_ACCOUNT_NAME}"
   ```
1. Azure ポータルで、このエクササイズの前のタスクで作成した Azure ストレージアカウントが表示されているブレードに移動します。 

1. Azure ストレージアカウントのブレードで、**キュー** をクリックして、キューのリストを表示します。 

1. このエクササイズの前のタスクで作成したキューを表すエントリをクリックします。

1. キューには単一のメッセージが含まれていることに注意してください。エントリをクリックして、**メッセージプロパティ**ブレードを表示します。 

1. **MESSAGE BODY** では、前のタスクでアップロードした Azure Storage Blob の URL を表す **URL** プロパティの値に注意してください。


## 演習 3: ラボリソースを削除

#### タスク 1: Cloud Shell を開く

1. ポータルの上部にある 「**Cloud Shell**」 アイコンをクリックして、Cloud Shell ペインを開きます。

1. 必要に応じて、Cloud Shell ペインの左上隅にあるドロップ ダウン リストを使用して、Bash シェル セッションに切り替えます。

1. **「Cloud Shell」** コマンドプロンプトで、次のコマンドを入力し、 **「Enter」** キーを押して、このラボで作成したすべてのリソース グループを一覧表示します。

   ```
   az group list --query "[?starts_with(name,'az300T06')]".name --output tsv
   ```

1. このラボで作成したリソース グループのみが出力に含まれていることを確認します。これらのグループは、次のタスクで削除されます。

#### タスク 2: リソース グループを削除する

1. 「**Cloud Shell**」 コマンド プロンプトで、次のコマンドを入力し、**Enter** キーを押して、このラボで作成したリソース グループを削除します

   ```sh
   az group list --query "[?starts_with(name,'az300T06')]".name --output tsv | xargs -L1 bash -c 'az group delete --name $0 --no-wait --yes'
   ```

1. ポータルの下部にある 「**Cloud Shell**」 プロンプトを閉じます。

> **結果**: このエクササイズでは、このラボで使用するリソースを削除しました。
