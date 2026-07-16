---
lab:
    title: 'Azure Cosmos DB SQL API SDK でクロスプロダクト クエリ結果をページ分割する'
    module: 'モジュール 5 - Azure Cosmos DB SQL API でクエリを実行する'
---

# Azure Cosmos DB SQL API SDK でクロスプロダクト クエリ結果をページ分割する

Azure Cosmos DB のクエリ結果は、通常、複数ページに分かれます。Azure Cosmos DB が 1 回の実行ですべてのクエリ結果を返せない場合、ページ分割はサーバー側で自動的に行われます。多くのアプリケーションでは、SDK を使用してクエリ結果をバッチ単位で効率的に処理するコードを作成する必要があります。

このラボでは、ループ内で使用して結果セット全体を反復処理できるフィード イテレーターを作成します。

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

## SDK を使用して SQL クエリの小さな結果セットをページ分割する

クエリ結果を処理するときは、後続の要求を行う前に、コードが結果のすべてのページを順に処理し、さらにページが残っているかどうかを確認する必要があります。

1. **Visual Studio Code** の **Explorer** ペインで、**10-paginate-results-sdk** フォルダーを参照してください。

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

1. *string* 型の **sql** という名前の新しい変数を作成し、値を **SELECT p.id, p.name, p.price FROM products p** に設定してください。

    ```
    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    ```

1. [QueryDefinition][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition] 型の新しい変数を作成し、コンストラクターのパラメーターとして **sql** 変数を渡してください。

    ```
    QueryDefinition query = new (sql);
    ```

1. 既定の空コンストラクターを使用して、[QueryRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions] 型の **options** という名前の新しい変数を作成してください。

    ```
    QueryRequestOptions options = new ();
    ```

1. **options** 変数の [MaxItemCount][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount] プロパティを **50** に設定してください。

    ```
    options.MaxItemCount = 50;
    ```

1. [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] クラスのジェネリック [GetItemQueryIterator][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator] メソッドを呼び出し、**query** 変数と **options** 変数をパラメーターとして渡して、[FeedIterator<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1] 型の **iterator** という名前の新しい変数を作成してください。

    ```
    FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query, requestOptions: options);
    ```

1. **iterator** 変数の [HasMoreResults][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults] プロパティを確認する **while** ループを作成してください。

    ```
    while (iterator.HasMoreResults)
    {
        
    }
    ```

1. **while** ループ内で、**iterator** 変数の [ReadNextAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync] メソッドを非同期で呼び出し、結果を **Product** クラスを使用したジェネリック型 [FeedResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1] の **products** という変数に格納してください。

    ```
    FeedResponse<Product> products = await iterator.ReadNextAsync();
    ```

1. 引き続き **while** ループ内で、**Product** 型のインスタンスを表す変数 **product** を使用して **products** 変数を反復処理し、新しい **foreach** ループを作成してください。

    ```
    foreach (Product product in products)
    {

    }
    ```

1. **foreach** ループ内で、組み込みの **Console.WriteLine** 静的メソッドを使用して、**product** 変数の **id**、**name**、**price** プロパティを書式設定して出力してください。

    ```
    Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
    ```

1. **while** ループに戻り、組み込みの **Console.WriteLine** 静的メソッドを使用して *Press any key to get more results* というメッセージを出力してください。

    ```
    Console.WriteLine("Press any key to get more results");
    ```

1. 引き続き **while** ループ内で、組み込みの **Console.ReadKey** 静的メソッドを使用して次のキー入力を待機してください。

    ```
    Console.ReadKey();
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");

    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    QueryDefinition query = new (sql);

    QueryRequestOptions options = new ();
    options.MaxItemCount = 50;

    FeedIterator<Product> iterator = container.GetItemQueryIterator<Product>(query, requestOptions: options);

    while (iterator.HasMoreResults)
    {
        FeedResponse<Product> products = await iterator.ReadNextAsync();
        foreach (Product product in products)
        {
            Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
        }

        Console.WriteLine("Press any key for next page of results");
        Console.ReadKey();        
    }
    ```

1. **script.cs** ファイルを **Save** してください。

1. **Visual Studio Code** で **10-paginate-results-sdk** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

1. スクリプトにより、クエリに一致する最初の 50 件の項目が出力されます。クエリが一致するすべての項目を反復処理し終えるまで、任意のキーを押して次の 50 件の項目を取得してください。

    > &#128161; このクエリは、products コンテナー内の数百件の項目に一致します。

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.hasmoreresults
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1.readnextasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feedresponse-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
