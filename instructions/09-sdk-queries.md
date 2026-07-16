---
lab:
    title: 'Azure Cosmos DB SQL API SDK を使用してクエリを実行する'
    module: 'モジュール 5 - Azure Cosmos DB SQL API でクエリを実行する'
---

# Azure Cosmos DB SQL API SDK を使用してクエリを実行する

Azure Cosmos DB SQL API 向け .NET SDK の最新バージョンでは、C# の最新のベスト プラクティスと言語機能を使用して、コンテナーへのクエリ実行と結果セットの非同期反復処理をこれまでになく簡単に行えます。

> &#128161; このラボでは、NuGet の [Azure.Cosmos][nuget.org/packages/azure.cosmos/4.0.0-preview3] ライブラリの *4.0.0-preview3* リリースを使用します。このライブラリには、[非同期ストリーム][docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams] を使用して Azure Cosmos DB に対するクエリを簡単に実行できる特別な機能があります。

このラボでは、非同期ストリームを使用して Azure Cosmos DB SQL API から返される大きな結果セットを反復処理します。.NET SDK を使用してクエリを実行し、結果を反復処理します。

## 開発環境を準備する

このラボを実施している環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、次の手順に従って実行してください。それ以外の場合は、以前にクローンしたフォルダーを **Visual Studio Code** で開いてください。

1. **Visual Studio Code** を起動してください。

    > &#128221; Visual Studio Code のインターフェイスにまだ慣れていない場合は、[Visual Studio Code の Get Started ガイド][code.visualstudio.com/docs/getstarted] を確認してください。

1. コマンド パレットを開いて **Git: Clone** を実行し、``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリを任意のローカル フォルダーにクローンしてください。

    > &#128161; **CTRL+SHIFT+P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリのクローンが完了したら、選択したローカル フォルダーを **Visual Studio Code** で開いてください。

## Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングするときは、アカウントでサポートする API（たとえば **Mongo API** や **SQL API**）を選択します。Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、それらを使用して Azure SDK for .NET または任意の SDK から Azure Cosmos DB SQL API アカウントに接続できます。

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインしてください。

1. **+ Create a resource** を選択し、*Cosmos DB* を検索してから、次の設定で新しい **Azure Cosmos DB SQL API** アカウント リソースを作成してください。残りの設定はすべて既定値のままにしてください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Subscription** | *既存の Azure サブスクリプション* |
    | **Resource group** | *既存のリソース グループを選択するか新規作成する* |
    | **Account Name** | *グローバルで一意の名前を入力する* |
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

## Azure Cosmos DB SQL API アカウントにデータをシードする

[cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールは、任意の Azure Cosmos DB SQL API アカウントにサンプル データをデプロイします。このツールはオープンソースで、NuGet から利用できます。このツールを Azure Cloud Shell にインストールし、データベースのシードに使用します。

1. **Visual Studio Code** で **Terminal** メニューを開き、**New Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールを、マシン全体で使用できるようにインストールしてください。

    ```
    dotnet tool install --global cosmicworks
    ```

    > &#128161; このコマンドの完了には数分かかる場合があります。過去にこのツールの最新バージョンをすでにインストールしている場合、このコマンドは警告メッセージ（*Tool 'cosmicworks' is already installed'）を出力します。

1. 次のコマンドライン オプションを使用して cosmicworks を実行し、Azure Cosmos DB アカウントをシードしてください。

    | **Option** | **Value** |
    | ---: | :--- |
    | **--endpoint** | *このラボの前半でコピーした endpoint 値* |
    | **--key** | *このラボの前半でコピーした key 値* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; たとえば、endpoint が **https&shy;://dp420.documents.azure.com:443/** で、key が **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります:
    > ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. **cosmicworks** コマンドが、アカウントへのデータベース、コンテナー、および項目の追加を完了するまで待機してください。

1. 統合ターミナルを閉じてください。

## SDK を使用して SQL クエリ結果を反復処理する

次に、非同期ストリームを使用して、Azure Cosmos DB から返されるページ分割された結果に対して、理解しやすい foreach ループを作成します。内部では SDK がフィード イテレーターを管理し、後続の要求が正しく実行されるようにします。

1. **Visual Studio Code** の **Explorer** ペインで、**09-execute-query-sdk** フォルダーを参照してください。

1. **product.cs** コード ファイルを開いてください。

1. **Product** クラスと対応するプロパティを確認してください。具体的には、このラボでは **id**、**name**、**price** プロパティを使用します。

1. **Visual Studio Code** の **Explorer** ペインに戻り、**script.cs** コード ファイルを開いてください。

1. 既存の **endpoint** という名前の変数を更新し、先ほど作成した Azure Cosmos DB アカウントの **endpoint** を値として設定してください。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは次のようになります: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**。

1. 既存の **key** という名前の変数を更新し、先ほど作成した Azure Cosmos DB アカウントの **key** を値として設定してください。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは次のようになります: **string key = "fDR2ci9QgkdkvERTQ==";**。

1. *string* 型の **sql** という名前の新しい変数を作成し、値を **SELECT * FROM products p** に設定してください。

    ```
    string sql = "SELECT * FROM products p";
    ```

1. [QueryDefinition][docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition] 型の新しい変数を作成し、コンストラクターのパラメーターとして **sql** 変数を渡してください。

    ```
    QueryDefinition query = new (sql);
    ```

1. [CosmosContainer][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer] クラスのジェネリック [GetItemQueryIterator][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator] メソッドを呼び出して、パラメーターとして **query** 変数を渡した新しい **await foreach** ループを作成してください。その後、**Product** 型のインスタンスを表す変数 **product** を使用して結果を非同期に反復処理してください。

    ```
    await foreach (Product product in container.GetItemQueryIterator<Product>(query))
    {
    }
    ```

1. **await foreach** ループ内で、組み込みの **Console.WriteLine** 静的メソッドを使用して、**product** 変数の **id**、**name**、**price** プロパティを書式設定して出力してください。

    ```
    Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using System;
    using Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    CosmosDatabase database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    CosmosContainer container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    string sql = "SELECT * FROM products p";
    QueryDefinition query = new (sql);

    await foreach (Product product in container.GetItemQueryIterator<Product>(query))
    {
        Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    }
    ```

1. **script.cs** ファイルを **Save** してください。

1. **Visual Studio Code** で **09-execute-query-sdk** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

1. スクリプトによって、コンテナー内のすべての製品が出力されます。

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer
[docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator
[docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams]: https://docs.microsoft.com/dotnet/csharp/whats-new/csharp-8#asynchronous-streams
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/azure.cosmos/4.0.0-preview3]: https://www.nuget.org/packages/azure.cosmos/4.0.0-preview3
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
