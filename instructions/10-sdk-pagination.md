# Lab 05b - Azure Cosmos DB for NoSQL でクエリを実行する

## ラボ シナリオ

Azure Cosmos DB のクエリでは、通常、結果が複数ページに分かれます。Azure Cosmos DB が 1 回の実行で全結果を返せない場合、ページ分割はサーバー側で自動的に行われます。多くのアプリケーションでは、SDK を使用してクエリ結果を高性能にバッチ処理するコードを実装したくなります。

このラボでは、ループ内で使用して結果セット全体を反復処理できる feed iterator を作成します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Azure Cosmos DB for NoSQL アカウントにデータを投入する。
- タスク 3: SDK を使用して SQL クエリの小さな結果セットをページ分割して処理する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab10.png)

## 演習 1: Azure Cosmos DB for NoSQL SDK を使用してクロスプロダクト クエリ結果をページ分割する

### タスク 1: 開発環境を準備する

1. Visual Studio Code を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

2. 左側ペインの **Extension (1)** アイコンを選択してください。検索バーに **C# (2)** と入力し、表示された **extension (3)** を選択して、最後に拡張機能の **Install (4)** を選択してください。

    ![](media/C-hash-extension.png)

3. 画面左上の **file** を選択し、メニューから **Open Folder** を選択して **C:\AllFiles** に移動してください。

4. **dp-420-cosmos-db-dev** フォルダーを選択し、**Select Folder** をクリックしてください。

### タスク 2: Azure Cosmos DB for NoSQL アカウントにデータを投入する

[cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールは、任意の Azure Cosmos DB for NoSQL アカウントにサンプル データをデプロイします。このツールはオープンソースで NuGet から利用できます。このツールを Azure Cloud Shell にインストールし、データベースへのデータ投入に使用します。

1. **Visual Studio Code** で **Terminal** メニューを開き、**New Terminal** を選択して新しいターミナルを開いてください。

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをマシン全体で使えるようにインストールしてください。

    ```
    dotnet tool install --global cosmicworks
    ```

    >**Note**: このコマンドは完了までに数分かかる場合があります。過去にこのツールの最新バージョンをすでにインストールしている場合、このコマンドは警告メッセージ（*Tool 'cosmicworks' is already installed'）を出力します。

1. 次のコマンドライン オプションで cosmicworks を実行し、Azure Cosmos DB アカウントにデータを投入してください。

    | **Option** | **Value** |
    | --- | --- |
    | **--endpoint** | *このラボで先ほどコピーした endpoint 値* |
    | **--key** | *このラボで先ほどコピーした key 値* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    >**Note**: たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/**、key が **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。
    > ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. **cosmicworks** コマンドがアカウントにデータベース、コンテナー、項目を投入し終えるまで待機してください。

1. 統合ターミナルを閉じてください。

### タスク 3: SDK を使用して SQL クエリの小さな結果セットをページ分割して処理する

クエリ結果を処理する際は、後続リクエストを行う前に、結果の全ページを順に処理し、さらにページが残っているかどうかを確認する必要があります。

1. **Visual Studio Code** の **Explorer** ペインで、**10-paginate-results-sdk** フォルダーに移動してください。

1. **product.cs** コード ファイルを開いてください。

1. **Product** クラスと対応するプロパティを確認してください。特にこのラボでは **id**、**name**、**price** プロパティを使用します。

1. **Visual Studio Code** の **Explorer** ペインに戻り、**script.cs** コード ファイルを開いてください。

1. 既存の **endpoint** という名前の変数を更新し、前のラボで作成した Azure Cosmos DB アカウントの **endpoint** を設定してください。

    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    >**Note**: たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. 既存の **key** という名前の変数を更新し、前のラボで作成した Azure Cosmos DB アカウントの **key** を設定してください。

    ```
    string key = "<cosmos-key>";
    ```

    >**Note**: たとえば key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. 型 *string* の新しい変数 **sql** を作成し、値を **SELECT p.id, p.name, p.price FROM products p** に設定してください。

    ```
    string sql = "SELECT p.id, p.name, p.price FROM products p ";
    ```

1. [QueryDefinition][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.querydefinition] 型の新しい変数を作成し、コンストラクターのパラメーターとして **sql** 変数を渡してください。

    ```
    QueryDefinition query = new (sql);
    ```

1. 既定の空コンストラクターを使用して、**options** という名前の [QueryRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions] 型の新しい変数を作成してください。

    ```
    QueryRequestOptions options = new ();
    ```

1. **options** 変数の [MaxItemCount][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.queryrequestoptions.maxitemcount] プロパティを **50** に設定してください。

    ```
    options.MaxItemCount = 50;
    ```

1. [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] クラスのジェネリック [GetItemQueryIterator][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getitemqueryiterator] メソッドを呼び出し、パラメーターとして **query** と **options** 変数を渡して、[FeedIterator<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.feediterator-1] 型の新しい変数 **iterator** を作成してください。

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

1. 同じく **while** ループ内で、**products** 変数を反復処理する新しい **foreach** ループを作成し、型 **Product** のインスタンスを表す変数として **product** を使用してください。

    ```
    foreach (Product product in products)
    {

    }
    ```

1. **foreach** ループ内で、組み込みの静的 **Console.WriteLine** メソッドを使用し、**product** 変数の **id**、**name**、**price** プロパティを整形して出力してください。

    ```
    Console.WriteLine($"[{product.id}]\t[{product.name,40}]\t[{product.price,10}]");
    ```

1. **while** ループに戻り、組み込みの静的 **Console.WriteLine** メソッドを使用して *Press any key to get more results* というメッセージを出力してください。

    ```
    Console.WriteLine("Press any key to get more results");
    ```

1. 同じく **while** ループ内で、組み込みの静的 **Console.ReadKey** メソッドを使用して次のキー入力を待機してください。

    ```
    Console.ReadKey();
    ```

1. 完了後、コード ファイルに次の内容が含まれていることを確認してください。

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

1. **Visual Studio Code** で **10-paginate-results-sdk** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```

1. スクリプトにより、クエリに一致する最初の 50 件が出力されます。任意のキーを押して次の 50 件を取得し、すべての一致項目を反復処理し終えるまで続けてください。

    >**Note**: クエリは products コンテナー内の数百件に一致します。

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

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- Azure Cosmos DB for NoSQL アカウントにデータを投入した。
- SDK を使用して SQL クエリの小さな結果セットをページ分割して処理した。

### ラボは正常に完了しました