# Lab 12a - DevOps プラクティスを使用して Azure Cosmos DB for NoSQL ソリューションを管理する

## ラボ シナリオ

Azure CLI は、Azure 全体のさまざまなリソースを管理するために使用できるコマンド セットです。Azure Cosmos DB には、選択した API に関係なく Azure Cosmos DB アカウントのさまざまな側面を管理できる豊富なコマンド グループが用意されています。

このラボでは、Azure CLI を使用して Azure Cosmos DB アカウント、データベース、およびコンテナーを作成します。その後、Azure CLI を使用してプロビジョニング済みスループットを調整します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: Azure CLI にログインする。
- タスク 2: Azure CLI を使用して Azure Cosmos DB アカウントを作成する。
- タスク 3: Azure CLI を使用して Azure Cosmos DB for NoSQL リソースを作成する。
- タスク 4: Azure CLI を使用して既存コンテナーのスループットを調整する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab29.png)

## 演習 1: Azure CLI スクリプトを使用してプロビジョニング済みスループットを調整する

### タスク 1: Azure CLI にログインする

Azure CLI を使用する前に、まず CLI のバージョンを確認し、Azure 資格情報でログインする必要があります。

1. **Visual Studio Code** を起動してください。

1. **Terminal** メニューを開き、**New Terminal** を選択して新しいターミナルを開いてください。

1. 次のコマンドを使用して Azure CLI のバージョンを確認してください。

    ```
    az --version
    ```

1. 次のコマンドを使用して、よく使用される Azure CLI コマンド グループを確認してください。

    ```
    az --help
    ```

1. 次のコマンドを使用して Azure CLI の対話型ログイン手順を開始してください。

    ```
    az login
    ```

1. Azure CLI は自動的に Web ブラウザー ウィンドウまたはタブを開きます。ブラウザー上で、サブスクリプションに関連付けられた Microsoft 資格情報を使用して Azure CLI にサインインしてください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

1. ラボ プロバイダーがリソース グループを作成しているか確認してください。作成されている場合は、次のセクションで必要になるため、その名前を記録してください。

    ```
    az group list --query "[].{ResourceGroupName:name}" -o table
    ```

    このコマンドは複数の Resource Group 名を返す場合があります。

### タスク 2: Azure CLI を使用して Azure Cosmos DB アカウントを作成する

**cosmosdb** コマンド グループには、CLI を使用して Azure Cosmos DB アカウントを作成および管理するための基本コマンドが含まれています。Azure Cosmos DB アカウントにはアドレス可能な URI があるため、スクリプトで作成する場合でも、新しいアカウント名はグローバルに一意である必要があります。

1. **Visual Studio Code** で既に開いているターミナルに戻ってください。

1. 次のコマンドを使用して、**Azure Cosmos DB** に関連する主な Azure CLI コマンドを確認してください。

    ```
    az cosmosdb --help
    ```

1. 次のコマンドを使用して、[Get-Random][docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random] PowerShell コマンドレットで **suffix** という新しい変数を作成してください。

    ```
    $suffix=Get-Random -Maximum 1000000
    ```

    >**Note**: Get-Random コマンドレットは 0 から 1,000,000 の間のランダム整数を生成します。サービスにはグローバル一意名が必要なため、これは有用です。

1. 次のコマンドを使用して、固定文字列 **csms** と変数置換により **$suffix** 変数の値を注入し、**accountName** という新しい変数を作成してください。

    ```
    $accountName="csms$suffix"
    ```

1. 次のコマンドを使用して、このラボで先ほど作成または確認したリソース グループ名で **resourceGroup** という新しい変数を作成してください。

    ```
    $resourceGroup="<resource-group-name>"
    ```

    >**Note**: たとえばリソース グループ名が **DP-420-xxxxx** の場合、コマンドは **$resourceGroup="DP-420-xxxxx"** になります。

1. 次のコマンドを使用して **echo** コマンドレットを実行し、**$accountName** と **$resourceGroup** の値をターミナル出力に表示してください。

    ```
    echo $accountName
    echo $resourceGroup
    ```

1. 次のコマンドを使用して **az cosmosdb create** のオプションを確認してください。

    ```
    az cosmosdb create --help
    ```

1. 次のコマンドと事前定義した変数を使用して、新しい Azure Cosmos DB アカウントを作成してください。

    ```
    az cosmosdb create --name $accountName --resource-group $resourceGroup
    ```

1. このラボを続行する前に、**create** コマンドの実行が完了して戻るまで待機してください。

    >**Note**: **create** コマンドの完了には平均で 2〜12 分かかる場合があります。

### タスク 3: Azure CLI を使用して Azure Cosmos DB for NoSQL リソースを作成する

**cosmosdb sql** コマンド グループには、Azure Cosmos DB の API for NoSQL 固有リソースを管理するためのコマンドが含まれています。これらのコマンド グループのオプションは **--help** フラグでいつでも確認できます。

1. **Visual Studio Code** で既に開いているターミナルに戻ってください。

1. 次のコマンドを使用して、**Azure Cosmos DB for NoSQL** に関連する主な Azure CLI コマンド グループを確認してください。

    ```
    az cosmosdb sql --help
    ```

1. 次のコマンドを使用して、**Azure Cosmos DB for NoSQL** のデータベース管理用 Azure CLI コマンドを確認してください。

    ```
    az cosmosdb sql database --help
    ```

1. 次のコマンド、事前定義した変数、およびデータベース名 **cosmicworks** を使用して、新しい Azure Cosmos DB データベースを作成してください。

    ```
    az cosmosdb sql database create --name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. このラボを続行する前に、**create** コマンドの実行が完了して戻るまで待機してください。

1. 次のコマンドを使用して、**Azure Cosmos DB for NoSQL** のコンテナー管理用 Azure CLI コマンドを確認してください。

    ```
    az cosmosdb sql container --help
    ```

1. 次のコマンドと事前定義した変数、データベース名 **cosmicworks**、コンテナー名 **products** を使用して、新しい Azure Cosmos DB コンテナーを作成してください。

    ```
    az cosmosdb sql container create --name "products" --throughput 400 --partition-key-path "/categoryId" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. このラボを続行する前に、**create** コマンドの実行が完了して戻るまで待機してください。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Resource groups** を選択し、このラボで先ほど作成または確認したリソース グループを選択してから、**csms** プレフィックスでこのラボ中に作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で **Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**API for NoSQL** ナビゲーション ツリー内に新しい **products** コンテナー ノードがあることを確認してください。

1. **API for NoSQL** ナビゲーション ツリーで **products** コンテナー ノードを選択し、**Scale & Settings** を選択してください。

1. **Scale** タブの値を確認してください。特に、**Throughput** セクションで **Manual** オプションが選択され、プロビジョニング済みスループットが **400** RU/s に設定されていることを確認してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

### タスク 4: Azure CLI を使用して既存コンテナーのスループットを調整する

Azure CLI は、コンテナーのスループットを manual と autoscale の間で移行するために使用できます。コンテナーが autoscale スループットを使用している場合、CLI を使って許可される最大スループット値を動的に調整できます。

1. **Visual Studio Code** で既に開いているターミナルに戻ってください。

1. 次のコマンドを使用して、**Azure Cosmos DB for NoSQL** のコンテナー スループット管理用 Azure CLI コマンドを確認してください。

    ```
    az cosmosdb sql container throughput --help
    ```

1. 次のコマンドを使用して、**products** コンテナーのスループットを manual プロビジョニングから autoscale へ移行してください。

    ```
    az cosmosdb sql container throughput migrate --name "products" --throughput-type autoscale --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. このラボを続行する前に、**migrate** コマンドの実行が完了して戻るまで待機してください。

1. 次のコマンドを使用して **products** コンテナーを問い合わせ、最小可能スループット値を確認してください。

    ```
    az cosmosdb sql container throughput show --name "products" --query "resource.minimumThroughput" --output "tsv" --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. 次のコマンドを使用して、**products** コンテナーの autoscale 最大スループットを既定値 **4,000** から新しい値 **5,000** に更新してください。

    ```
    az cosmosdb sql container throughput update --name "products" --max-throughput 5000 --database-name "cosmicworks" --account-name $accountName --resource-group $resourceGroup
    ```

1. このラボを続行する前に、**update** コマンドの実行が完了して戻るまで待機してください。

1. **Visual Studio Code** を閉じてください。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Resource groups** を選択し、このラボで先ほど作成または確認したリソース グループを選択してから、**csms** プレフィックスでこのラボ中に作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で **Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**API for NoSQL** ナビゲーション ツリー内に新しい **products** コンテナー ノードがあることを確認してください。

1. **API for NoSQL** ナビゲーション ツリーで **products** コンテナー ノードを選択し、**Scale & Settings** を選択してください。

1. **Scale** タブの値を確認してください。特に、**Throughput** セクションで **Autoscale** オプションが選択され、プロビジョニング済みスループットが **5,000** RU/s に設定されていることを確認してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

[docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random]: https://docs.microsoft.com/powershell/module/microsoft.powershell.utility/get-random

### レビュー

このラボでは、次を完了しました。

- Azure CLI にログインした。
- Azure CLI を使用して Azure Cosmos DB アカウントを作成した。
- Azure CLI を使用して Azure Cosmos DB for NoSQL リソースを作成した。
- Azure CLI を使用して既存コンテナーのスループットを調整した。

### ラボは正常に完了しました