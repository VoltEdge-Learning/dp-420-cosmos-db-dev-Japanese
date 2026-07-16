---
lab:
    title: 'Azure Cosmos DB SQL API SDK を使用して複数のドキュメントを一括移動する'
    module: 'モジュール 4 - Azure Cosmos DB SQL API SDK でデータにアクセスして管理する'
---

# Azure Cosmos DB SQL API SDK を使用して複数のドキュメントを一括移動する

一括操作の実行方法を学ぶ最も簡単な方法は、クラウド上の Azure Cosmos DB SQL API アカウントに多数のドキュメントをプッシュしてみることです。SDK の一括機能を使用すると、[System.Threading.Tasks][docs.microsoft.com/dotnet/api/system.threading.tasks] 名前空間の少しの補助とともにこれを実行できます。

このラボでは、NuGet の [Bogus][nuget.org/packages/bogus/33.1.1] ライブラリを使用して架空データを生成し、それを Azure Cosmos DB アカウントに格納します。

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

1. 新しく作成した **Azure Cosmos DB** アカウント リソース内で、**Data Explorer** ペインに移動してください。

1. **Data Explorer** で **New Container** を選択し、次の設定で新しいコンテナーを作成してください。残りのすべての設定は既定値のままにしてください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Database id** | *新規作成* &vert; *`cosmicworks`* |
    | **Share throughput across containers** | *選択しない* |
    | **Container id** | *`products`* |
    | **Partition key** | *`/categoryId`* |
    | **Container throughput** | *Autoscale* &vert; *`4000`* |

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

1. **Visual Studio Code** の **Explorer** ペインで、**08-sdk-bulk** フォルダーを参照してください。

1. **08-sdk-bulk** フォルダー内の **script.cs** コード ファイルを開いてください。

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

1. **08-sdk-bulk** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

    > &#128221; このコマンドでは、開始ディレクトリがすでに **08-sdk-bulk** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加してください。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドしてください。

    ```
    dotnet build
    ```

1. 統合ターミナルを閉じてください。

## 25,000 件のドキュメントを一括挿入する

それでは、仕組みを確認するために多くのドキュメントを挿入してみます。内部テストでは、ラボの仮想マシンと Azure Cosmos DB SQL API アカウントの地理的な距離が比較的近い場合、これにはおよそ 1～2 分かかることがあります。

1. **script.cs** コード ファイルのエディター タブに戻ってください。

1. **AllowBulkExecution** プロパティを **true** に設定した [CosmosClientOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions] クラスの新しいインスタンスを **options** という名前で作成してください。

    ```
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    ```

1. **endpoint**、**key**、**options** 変数をコンストラクター パラメーターとして渡し、**CosmosClient** クラスの新しいインスタンスを **client** という名前で作成してください。

    ```
    CosmosClient client = new (endpoint, key, options); 
    ```

1. **client** 変数の [GetContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer] メソッドを使用し、データベース名（*cosmicworks*）とコンテナー名（*products*）を使用して既存のコンテナーを取得してください。

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. この特別なサンプル コードを使用して、NuGet からインポートした Bogus ライブラリの **Faker** クラスで **25,000** 件の架空の製品を生成してください。

    ```
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
    ```

    > &#128161; [Bogus][nuget.org/packages/bogus/33.1.1] ライブラリは、ユーザー インターフェイス アプリケーションをテストするための架空データを作成するオープンソース ライブラリであり、一括インポート/エクスポート アプリケーションの開発方法を学ぶのに適しています。

1. **Task** 型の新しいジェネリック **List<>** を **concurrentTasks** という名前で作成してください。

    ```
    List<Task> concurrentTasks = new List<Task>();
    ```

1. このアプリケーションで先ほど生成した製品リストを反復処理する foreach ループを作成してください。

    ```
    foreach(Product product in productsToInsert)
    {
    }
    ```

1. foreach ループ内で、製品を Azure Cosmos DB SQL API に非同期で挿入する **Task** を作成してください。パーティション キーを明示的に指定し、そのタスクを **concurrentTasks** という名前のタスク リストに追加してください。

    ```
    concurrentTasks.Add(
        container.CreateItemAsync(product, new PartitionKey(product.categoryId))
    );   
    ```

1. foreach ループの後で、**concurrentTasks** 変数に対する **Task.WhenAll** の結果を非同期で await してください。

    ```
    await Task.WhenAll(concurrentTasks);
    ```

1. 組み込みの静的メソッド **Console.WriteLine** を使用して、**Bulk tasks complete** という固定メッセージをコンソールに出力してください。

    ```
    Console.WriteLine("Bulk tasks complete");
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using System;
    using System.Collections.Generic;
    using System.Threading.Tasks;
    using Bogus;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClientOptions options = new () 
    { 
        AllowBulkExecution = true 
    };
    
    CosmosClient client = new (endpoint, key, options);  
    
    Container container = client.GetContainer("cosmicworks", "products");
    
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
        
    List<Task> concurrentTasks = new List<Task>();
    
    foreach(Product product in productsToInsert)
    {    
        concurrentTasks.Add(
            container.CreateItemAsync(product, new PartitionKey(product.categoryId))
        );
    }
    
    await Task.WhenAll(concurrentTasks);   

    Console.WriteLine("Bulk tasks complete");
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **08-sdk-bulk** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

1. アプリケーションは静かに実行され、完了までにおよそ 1～2 分かかるはずです。

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

## 結果を確認する

25,000 件のアイテムを Azure Cosmos DB に送信したので、Data Explorer を確認します。

1. Web ブラウザーで Azure portal (``portal.azure.com``) に移動してください。

1. **Resource groups** を選択し、次にこのラボで先ほど作成または確認したリソース グループを選択し、さらにこのラボで作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で、**Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**SQL API** ナビゲーション ツリー内の **products** コンテナー ノードを確認してください。

1. **products** ノードを展開し、**Items** ノードを選択してください。コンテナー内のアイテム一覧を確認してください。

1. **SQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選択し、**New SQL Query** を選択してください。

1. エディター領域の内容を削除してください。

1. 一括操作で作成されたすべてのドキュメント数を返す新しい SQL クエリを作成してください。

    ```
    SELECT COUNT(1) FROM items
    ```

1. **Execute Query** を選択してください。

1. コンテナー内のアイテム数を確認してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions
[docs.microsoft.com/dotnet/api/system.threading.tasks]: https://docs.microsoft.com/dotnet/api/system.threading.tasks
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/bogus/33.1.1]: https://www.nuget.org/packages/bogus/33.1.1
