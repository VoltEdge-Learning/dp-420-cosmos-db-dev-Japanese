---
lab:
    title: 'Azure Cosmos DB SQL API SDK で複数のポイント操作をまとめてバッチ処理する'
    module: 'モジュール 4 - Azure Cosmos DB SQL API SDK でデータにアクセスして管理する'
---

# Azure Cosmos DB SQL API SDK で複数のポイント操作をまとめてバッチ処理する

[TransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch] クラスと [TransactionalBatchResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse] クラスは、操作を 1 つの論理ステップにまとめたり分解したりするための重要な要素です。これらのクラスを使用すると、複数の操作を実行するコードを記述し、サーバー側で正常に完了したかどうかを判定できます。

このラボでは、SDK を使用して、2 つのアイテムを 1 つの論理単位として作成しようとする 2 つのデュアル アイテム操作を実行します。

## 開発環境を準備する

このラボを実施している環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、次の手順に従って実行してください。それ以外の場合は、以前にクローンしたフォルダーを **Visual Studio Code** で開いてください。

1. **Visual Studio Code** を起動してください。

    > &#128221; Visual Studio Code インターフェイスにまだ慣れていない場合は、[Getting Started ドキュメント][code.visualstudio.com/docs/getstarted] を確認してください。

1. コマンド パレットを開いて **Git: Clone** を実行し、``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリを任意のローカル フォルダーにクローンしてください。

    > &#128161; **CTRL+SHIFT+P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリのクローンが完了したら、選択したローカル フォルダーを **Visual Studio Code** で開いてください。

## Azure Cosmos DB SQL API アカウントを作成し、SDK プロジェクトを構成する

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインしてください。

1. **+ Create a resource** を選択し、*Cosmos DB* を検索してから、次の設定で新しい **Azure Cosmos DB SQL API** アカウント リソースを作成してください。残りのすべての設定は既定値のままにしてください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Subscription** | *既存の Azure サブスクリプション* |
    | **Resource group** | *既存のリソース グループを選択するか、新しく作成する* |
    | **Account Name** | *グローバルに一意の名前を入力する* |
    | **Location** | *利用可能な任意のリージョンを選択する* |
    | **Capacity mode** | *プロビジョニングされたスループット* |
    | **Apply Free Tier Discount** | *適用しない* |

    > &#128221; ラボ環境には、新しいリソース グループの作成を制限する制約がある場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. このタスクを続行する前に、デプロイ タスクが完了するまで待機してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Keys** ペインに移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

1. **Visual Studio Code** の **Explorer** ペインで、**07-sdk-batch** フォルダーを参照してください。

1. **07-sdk-batch** フォルダー内の **script.cs** コード ファイルを開いてください。

    > &#128221; **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** ライブラリは、NuGet からすでに事前インポートされています。

1. **endpoint** という名前の **string** 変数を見つけてください。その値を先ほど作成した Azure Cosmos DB アカウントの **endpoint** に設定してください。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは次のようになります: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**。

1. **key** という名前の **string** 変数を見つけてください。その値を先ほど作成した Azure Cosmos DB アカウントの **key** に設定してください。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは次のようになります: **string key = "fDR2ci9QgkdkvERTQ==";**。

1. **script.cs** コード ファイルを **Save** してください。

1. **07-sdk-batch** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

    > &#128221; このコマンドでは、開始ディレクトリがすでに **07-sdk-batch** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加してください。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドしてください。

    ```
    dotnet build
    ```

1. 統合ターミナルを閉じてください。

## トランザクション バッチを作成する

まず、2 つの架空の製品を作成するシンプルなトランザクション バッチを作成します。このバッチでは、同じ「used accessories」カテゴリ識別子を持つコンテナーに、使い古したサドルとさびたハンドルバーを挿入します。両方のアイテムは同じ論理パーティション キーを持つため、バッチ操作は成功します。

1. **script.cs** コード ファイルのエディター タブに戻ってください。

1. 一意識別子 **0120**、名前 **Worn Saddle**、カテゴリ識別子 **9603ca6c-9e28-4a02-9194-51cdb7fea816** を持つ **saddle** という名前の **Product** 変数を作成してください。

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 一意識別子 **012A**、名前 **Rusty Handlebar**、カテゴリ識別子 **9603ca6c-9e28-4a02-9194-51cdb7fea816** を持つ **handlebar** という名前の **Product** 変数を作成してください。

    ```
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. **9603ca6c-9e28-4a02-9194-51cdb7fea816** をコンストラクター パラメーターとして渡し、**partitionKey** という名前の **PartitionKey** 型の変数を作成してください。

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. **container** 変数の [CreateTransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch] メソッドを呼び出し、メソッド パラメーターとして **partitionkey** 変数を渡してください。さらに、fluent 構文で [CreateItem<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem] ジェネリック メソッドを呼び出し、個別操作で作成するアイテムとして **saddle** 変数と **handlebar** 変数を渡し、結果を **batch** という名前の **TransactionalBatch** 型の変数に格納してください。

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    ```

1. using ステートメント内で **batch** 変数の **ExecuteAsync** メソッドを非同期に呼び出し、結果を **response** という名前の **TransactionalBatchResponse** 型の変数に格納してください。

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. 静的 **Console.WriteLine** メソッドを呼び出し、**response** 変数の **StatusCode** プロパティの値を出力してください。

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);
        
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **07-sdk-batch** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

1. ターミナルの出力を確認してください。ステータス コードは HTTP 200 **OK** になるはずです。

1. 統合ターミナルを閉じてください。

## エラーとなるトランザクション バッチを作成する

次に、意図的にエラーとなるトランザクション バッチを作成します。このバッチでは、異なる論理パーティション キーを持つ 2 つのアイテムを挿入しようとします。「used accessories」カテゴリに点滅するストロボ ライトを作成し、「pristine accessories」カテゴリに新しいヘルメットを作成します。定義上、これは不正な要求となり、このトランザクションの実行時にエラーが返されます。

1. **script.cs** コード ファイルのエディター タブに戻ってください。

1. 次のコード行を削除してください。

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 一意識別子 **012B**、名前 **Flickering Strobe Light**、カテゴリ識別子 **9603ca6c-9e28-4a02-9194-51cdb7fea816** を持つ **light** という名前の **Product** 変数を作成してください。

    ```
    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 一意識別子 **012C**、名前 **New Helmet**、カテゴリ識別子 **0feee2e4-687a-4d69-b64e-be36afc33e74** を持つ **helmet** という名前の **Product** 変数を作成してください。

    ```
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    ```

1. **9603ca6c-9e28-4a02-9194-51cdb7fea816** をコンストラクター パラメーターとして渡し、**partitionKey** という名前の **PartitionKey** 型の変数を作成してください。

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. **container** 変数の **CreateTransactionalBatch** メソッドを呼び出し、メソッド パラメーターとして **partitionkey** 変数を渡してください。さらに、fluent 構文で **CreateItem<>** ジェネリック メソッドを呼び出し、個別操作で作成するアイテムとして **light** 変数と **helmet** 変数を渡し、結果を **batch** という名前の **TransactionalBatch** 型の変数に格納してください。

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    ```

1. using ステートメント内で **batch** 変数の **ExecuteAsync** メソッドを非同期に呼び出し、結果を **response** という名前の **TransactionalBatchResponse** 型の変数に格納してください。

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. 静的 **Console.WriteLine** メソッドを呼び出し、**response** 変数の **StatusCode** プロパティの値を出力してください。

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);
        
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);
    
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **07-sdk-batch** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

1. ターミナルの出力を確認してください。ステータス コードは HTTP 400 **Bad Request** または 409 **Conflict** のいずれかになるはずです。これは、トランザクション内のすべてのアイテムがトランザクション バッチと同じパーティション キー値を共有していなかったために発生しました。

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
