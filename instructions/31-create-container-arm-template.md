# Lab 12b - DevOps プラクティスを使用して Azure Cosmos DB for NoSQL ソリューションを管理する

## ラボ シナリオ

Azure Resource Manager テンプレートは、Azure にデプロイしたいインフラストラクチャを宣言的に定義する JSON ファイルです。Azure Resource Manager テンプレートは、Azure にサービスをデプロイするための一般的な infrastructure-as-code ソリューションです。Bicep はこの考え方をさらに発展させ、JSON テンプレートを作成するための、より読みやすいドメイン固有言語を提供します。

このラボでは、Azure Resource Manager テンプレートを使用して新しい Azure Cosmos DB アカウント、データベース、およびコンテナーを作成します。最初に生の JSON からテンプレートを作成し、次に Bicep ドメイン固有言語を使用してテンプレートを作成します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Azure Resource Manager テンプレートを使用して Azure Cosmos DB for NoSQL リソースを作成する。
- タスク 3: デプロイされた Azure Cosmos DB リソースを確認する。
- タスク 4: Bicep テンプレートを使用して Azure Cosmos DB for NoSQL リソースを作成する。
- タスク 5: Bicep テンプレートのデプロイ結果を確認する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab30.png)

## 演習 1: Azure Resource Manager テンプレートを使用して Azure Cosmos DB for NoSQL コンテナーを作成する

### タスク 1: 開発環境を準備する

1. Visual Studio Code を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

2. 左側ペインの **Extension (1)** アイコンを選択してください。検索バーに **C# (2)** を入力し、表示された **extension (3)** を選択して、最後に **Install (4)** を選択してください。

    ![](media/C-hash-extension.png)

3. 画面左上の **file** オプションを選択し、メニューから **Open Folder** を選択して **C:\AllFiles** に移動してください。

4. **dp-420-cosmos-db-dev** フォルダーを選択し、**Select Folder** をクリックしてください。

### タスク 2: Azure Resource Manager テンプレートを使用して Azure Cosmos DB for NoSQL リソースを作成する

Azure Resource Manager の **Microsoft.DocumentDB** リソース プロバイダーにより、JSON ファイルを使用してアカウント、データベース、およびコンテナーをデプロイできます。ファイルは複雑になることがありますが、予測可能な形式に従っており、Visual Studio Code 拡張機能の支援を受けて作成できます。

> **Note** : テンプレートの構文エラーが解決できない場合は、[solution Azure Resource Manager template][github.com/arm-template-guide] を参照してください。

1. **Visual Studio Code** の **Explorer** ペインで **31-create-container-arm-template** フォルダーに移動してください。

1. **deploy.json** ファイルを開いてください。

1. 空の Azure Resource Manager テンプレートを確認してください。

    ```
    {
        "$schema": "https://schema.management.azure.com/schemas/2019-04-01/deploymentTemplate.json#",
        "contentVersion": "1.0.0.0",
        "resources": [
        ]
    }
    ```

1. **resources** 配列内に、新しい Azure Cosmos DB アカウントを作成するための新しい JSON オブジェクトを追加してください。

    ```
    {
        "type": "Microsoft.DocumentDB/databaseAccounts",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id))]",
        "location": "[resourceGroup().location]",
        "properties": {
            "databaseAccountOfferType": "Standard",
            "locations": [
                {
                    "locationName": "westus"
                }
            ]
        }
    }
    ```

    このオブジェクトは次の設定で構成されています。

    | **Setting** | **Value** |
    | --- | --- |
    | **Resource type** | *Microsoft.DocumentDB/databaseAccounts* |
    | **API version** | *2021-05-15* |
    | **Account name** | *csmsarm* &amp; *unique string generated from account name*  |
    | **Location** | *Resource group's current location* |
    | **Account offer type** | *Standard* |
    | **Locations** | *Only West US* |

1. **deploy.json** ファイルを保存してください。

1. **31-create-container-arm-template** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

    > **Note** : このコマンドを実行すると、開始ディレクトリが **31-create-container-arm-template** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを使用して Azure CLI の対話型ログイン手順を開始してください。

    ```
    az login
    ```

1. Azure CLI は自動的に Web ブラウザー ウィンドウまたはタブを開きます。ブラウザー上で、サブスクリプションに関連付けられた Microsoft 資格情報を使用して Azure CLI にサインインしてください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

1. ラボ プロバイダーがリソース グループを作成しているか確認してください。作成されている場合は次のセクションで必要になるため、その名前を記録してください。

    ```
    az group list --query "[].{ResourceGroupName:name}" -o table
    ```
    
    このコマンドは複数の Resource Group 名を返す場合があります。

1. （任意）***リソース グループが作成されていない場合*** は、リソース グループ名を決めて作成してください。*一部のラボ環境は制限されており、管理者によるリソース グループ作成が必要な場合があります。*

    i. 次の一覧から、最寄りの location 名を取得してください。

    ```
    az account list-locations --query "sort_by([].{YOURLOCATION:name, DisplayName:regionalDisplayName}, &YOURLOCATION)" --output table
    ```

    ii. リソース グループを作成してください。*一部のラボ環境は制限されており、管理者によるリソース グループ作成が必要な場合があります。*
    ```
    az group create --name YOURRESOURCEGROUPNAME --location YOURLOCATION
    ```

1. 次のコマンドを使用し、このラボで先ほど作成または確認したリソース グループ名で **resourceGroup** という新しい変数を作成してください。

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > **Note** : たとえばリソース グループ名が **DP-420-xxxxxx** の場合、コマンドは **$resourceGroup="DP-420-xxxxxx"** になります。

1. 次のコマンドを使用して **echo** コマンドレットを実行し、**$resourceGroup** 変数の値をターミナル出力に表示してください。

    ```
    echo $resourceGroup
    ```

1. [az deployment group create][docs.microsoft.com/cli/azure/deployment/group] コマンドを使用して Azure Resource Manager テンプレートをデプロイしてください。

    ```
    az deployment group create --name "arm-deploy-account" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. 統合ターミナルは開いたままにし、**deploy.json** ファイルのエディターに戻ってください。

1. **resources** 配列内に、新しい Azure Cosmos DB for NoSQL データベースを作成するための JSON オブジェクトを追加してください。

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]"
        ],
        "properties": {
            "resource": {
                "id": "cosmicworks"
            }
        }
    }
    ```

    このオブジェクトは次の設定で構成されています。

    | **Setting** | **Value** |
    | --- | --- |
    | **Resource type** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API version** | *2021-05-15* |
    | **Account name** | *csmsarm* &amp; *unique string generated from account name* &amp; */cosmicworks*  |
    | **Resource id** | *cosmicworks* |
    | **Dependencies** | *databaseAccount created earlier in the template* |

1. **deploy.json** ファイルを保存してください。

1. 統合ターミナルに戻ってください。

1. **az deployment group create** コマンドを使用して Azure Resource Manager テンプレートをデプロイしてください。

    ```
    az deployment group create --name "arm-deploy-database" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. 統合ターミナルは開いたままにし、**deploy.json** ファイルのエディターに戻ってください。

1. **resources** 配列内に、新しい Azure Cosmos DB for NoSQL コンテナーを作成するための JSON オブジェクトを追加してください。

    ```
    ,
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers",
        "apiVersion": "2021-05-15",
        "name": "[concat('csmsarm', uniqueString(resourceGroup().id), '/cosmicworks/products')]",
        "dependsOn": [
            "[resourceId('Microsoft.DocumentDB/databaseAccounts', concat('csmsarm', uniqueString(resourceGroup().id)))]",
            "[resourceId('Microsoft.DocumentDB/databaseAccounts/sqlDatabases', concat('csmsarm', uniqueString(resourceGroup().id)), 'cosmicworks')]"
        ],
        "properties": {
            "options": {
                "throughput": 400
            },
            "resource": {
                "id": "products",
                "partitionKey": {
                    "paths": [
                        "/categoryId"
                    ]
                }
            }
        }
    }
    ```

    このオブジェクトは次の設定で構成されています。

    | **Setting** | **Value** |
    | --- | --- |
    | **Resource type** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers* |
    | **API version** | *2021-05-15* |
    | **Account name** | *csmsarm* &amp; *unique string generated from account name* &amp; */cosmicworks/products*  |
    | **Resource id** | *products* |
    | **Throughput** | *400* |
    | **Partition key** | */categoryId* |
    | **Dependencies** | *Account and database created earlier in the template* |

1. **deploy.json** ファイルを保存してください。

1. 統合ターミナルに戻ってください。

1. **az deployment group create** コマンドを使用して最終的な Azure Resource Manager テンプレートをデプロイしてください。

    ```
    az deployment group create --name "arm-deploy-container" --resource-group $resourceGroup --template-file .\deploy.json
    ```

1. 統合ターミナルを閉じてください。

### タスク 3: デプロイされた Azure Cosmos DB リソースを確認する

Azure Cosmos DB for NoSQL リソースがデプロイされたら、Azure portal でリソースに移動できます。Data Explorer を使用して、アカウント、データベース、およびコンテナーが正しくデプロイおよび構成されていることを検証します。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Resource groups** を選択し、このラボで先ほど作成または確認したリソース グループを選択してから、**csmsarm** プレフィックスでこのラボ中に作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で **Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**API for NoSQL** ナビゲーション ツリー内に新しい **products** コンテナー ノードがあることを確認してください。

1. **API for NoSQL** ナビゲーション ツリーで **products** コンテナー ノードを選択し、**Scale & Settings** を選択してください。

1. **Scale** セクションの値を確認してください。特に、**Throughput** セクションで **Manual** オプションが選択され、プロビジョニング済みスループットが **400** RU/s に設定されていることを確認してください。

1. **Settings** セクションの値を確認してください。特に、**Partition key** の値が **/categoryId** に設定されていることを確認してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

### タスク 4: Bicep テンプレートを使用して Azure Cosmos DB for NoSQL リソースを作成する

Bicep は効率的なドメイン固有言語であり、Azure Resource Manager テンプレートよりも Azure リソースのデプロイをシンプルかつ容易にします。違いを示すために、同じリソースを Bicep と別の名前でデプロイします。

> **Note** : テンプレートの構文エラーが解決できない場合は、[solution Bicep template][github.com/bicep-template-guide] を参照してください。

1. **Visual Studio Code** の **Explorer** ペインで **31-create-container-arm-template** フォルダーに移動してください。

1. 空の **deploy.bicep** ファイルを開いてください。

1. ファイル内に、新しい Azure Cosmos DB アカウントを作成するための新しいオブジェクトを追加してください。

    ```
    resource Account 'Microsoft.DocumentDB/databaseAccounts@2021-05-15' = {
      name: 'csmsbicep${uniqueString(resourceGroup().id)}'
      location: resourceGroup().location
      properties: {
        databaseAccountOfferType: 'Standard'
        locations: [
          { 
            locationName: 'westus' 
          }
        ]
      }
    }
    ```

    このオブジェクトは次の設定で構成されています。

    | **Setting** | **Value** |
    | --- | --- |
    | **Alias** | *Account* |
    | **Name** | *csmsarm* &amp; *unique string generated from account name* |
    | **Resource type** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API version** | *2021-05-15* |
    | **Location** | *Resource group's current location* |
    | **Account offer type** | *Standard* |
    | **Locations** | *Only West US* |

1. **deploy.bicep** ファイルを保存してください。

1. **31-create-container-arm-template** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. 次のコマンドを使用し、このラボで先ほど作成または確認したリソース グループ名で **resourceGroup** という新しい変数を作成してください。

    ```
    $resourceGroup="<resource-group-name>"
    ```

    > **Note** : たとえばリソース グループ名が **DP-420-xxxxxx** の場合、コマンドは **$resourceGroup="DP-420-xxxxxx"** になります。

1. **az deployment group create** コマンドを使用して Bicep テンプレートをデプロイしてください。

    ```
    az deployment group create --name "bicep-deploy-account" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. 統合ターミナルは開いたままにし、**deploy.bicep** ファイルのエディターに戻ってください。

1. ファイル内に、新しい Azure Cosmos DB データベースを作成するためのオブジェクトを追加してください。

    ```
    resource Database 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases@2021-05-15' = {
      parent: Account
      name: 'cosmicworks'
      properties: {
        resource: {
            id: 'cosmicworks'
        }
      }
    }
    ```

    このオブジェクトは次の設定で構成されています。

    | **Setting** | **Value** |
    | --- | --- |
    | **Parent** | *Account created earlier in the template* |
    | **Alias** | *Database* |
    | **Name** | *cosmicworks*  |
    | **Resource type** | *Microsoft.DocumentDB/databaseAccounts/sqlDatabases* |
    | **API version** | *2021-05-15* |
    | **Resource id** | *cosmicworks* |

1. **deploy.bicep** ファイルを保存してください。

1. 統合ターミナルに戻ってください。

1. **az deployment group create** コマンドを使用して Bicep テンプレートをデプロイしてください。

    ```
    az deployment group create --name "bicep-deploy-database" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. 統合ターミナルは開いたままにし、**deploy.bicep** ファイルのエディターに戻ってください。

1. ファイル内に、新しい Azure Cosmos DB コンテナーを作成するためのオブジェクトを追加してください。

    ```
    resource Container 'Microsoft.DocumentDB/databaseAccounts/sqlDatabases/containers@2021-05-15' = {
      parent: Database
      name: 'products'
      properties: {
        options: {
          throughput: 400
        }
        resource: {
          id: 'products'
          partitionKey: {
            paths: [
              '/categoryId'
            ]
          }
        }
      }
    }
    ```

    このオブジェクトは次の設定で構成されています。

    | **Setting** | **Value** |
    | --- | --- |
    | **Parent** | *Database created earlier in the template* |
    | **Alias** | *Container* |
    | **Name** | *products*  |
    | **Resource id** | *products* |
    | **Throughput** | *400* |
    | **Partition key path** | */categoryId* |

1. **deploy.bicep** ファイルを保存してください。

1. 統合ターミナルに戻ってください。

1. **az deployment group create** コマンドを使用して最終的な Bicep テンプレートをデプロイしてください。

    ```
    az deployment group create --name "bicep-deploy-container" --resource-group $resourceGroup --template-file .\deploy.bicep
    ```

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

### タスク 5: Bicep テンプレートのデプロイ結果を確認する

Bicep デプロイは、Azure Resource Manager デプロイと同様の手法で検証できます。アカウント、データベース、およびコンテナーが正常にデプロイされたことを検証するだけでなく、6 回すべてのデプロイ履歴も確認します。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Resource groups** を選択し、このラボで先ほど作成または確認したリソース グループを選択してください。

1. リソース グループ内で **Deployments** ペインに移動してください。

1. Azure Resource Manager テンプレートと Bicep ファイルによる 6 つのデプロイを確認してください。

1. 引き続き同じリソース グループ内で **Overview** ペインに移動してください。

1. 引き続き同じリソース グループ内で、このラボ中に **csmsbicep** プレフィックスで作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で **Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**API for NoSQL** ナビゲーション ツリー内に新しい **products** コンテナー ノードがあることを確認してください。

1. **API for NoSQL** ナビゲーション ツリーで **products** コンテナー ノードを選択し、**Scale & Settings** を選択してください。

1. **Scale** セクションの値を確認してください。特に、**Throughput** セクションで **Manual** オプションが選択され、プロビジョニング済みスループットが **400** RU/s に設定されていることを確認してください。

1. **Settings** セクションの値を確認してください。特に、**Partition key** の値が **/categoryId** に設定されていることを確認してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- Azure Resource Manager テンプレートを使用して Azure Cosmos DB for NoSQL リソースを作成した。
- デプロイされた Azure Cosmos DB リソースを確認した。
- Bicep テンプレートを使用して Azure Cosmos DB for NoSQL リソースを作成した。
- Bicep テンプレートのデプロイ結果を確認した。

### ラボは正常に完了しました