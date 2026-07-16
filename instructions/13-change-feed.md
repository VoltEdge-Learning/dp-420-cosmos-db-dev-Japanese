---
lab:
    title: 'Azure Cosmos DB SQL API SDK を使用して変更フィード イベントを処理する'
    module: 'モジュール 7 - Azure Cosmos DB SQL API を Azure サービスと統合する'
---

# Azure Cosmos DB SQL API SDK を使用して変更フィード イベントを処理する

Azure Cosmos DB SQL API の変更フィードは、プラットフォームからのイベントによって駆動される補助アプリケーションを作成するための重要な機能です。Azure Cosmos DB SQL API 向け .NET SDK には、変更フィードと統合し、コンテナー内の操作通知を受け取るアプリケーションを構築するためのクラス群が用意されています。

このラボでは、.NET SDK の変更フィード プロセッサー機能を使用して、指定したコンテナー内の項目に対して作成または更新操作が実行されたときに通知を受け取るアプリケーションを作成します。

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
    | **Capacity mode** | *Serverless* |

    > &#128221; ラボ環境には、新しいリソース グループの作成を制限する制約がある場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. このタスクを続行する前に、デプロイ タスクが完了するまで待機してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Keys** ペインに移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

1. リソース メニューから **Data Explorer** を選択してください。

1. **Data Explorer** ペインで **New Container** を展開し、**New Database** を選択してください。

1. **New Database** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --: | :-- |
    | **Database id** | *cosmicworks* |

1. **Data Explorer** ペインに戻り、階層内の **cosmicworks** データベース ノードを確認してください。

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --: | :-- |
    | **Database id** | *Use existing* &vert; *cosmicworks* |
    | **Container id** | *products* |
    | **Partition key** | */categoryId* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認してください。

1. **Data Explorer** ペインで、再度 **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --: | :-- |
    | **Database id** | *Use existing* &vert; *cosmicworks* |
    | **Container id** | *productslease* |
    | **Partition key** | */partitionKey* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **productslease** コンテナー ノードを確認してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

## .NET SDK で変更フィード プロセッサーを実装する

**Microsoft.Azure.Cosmos.Container** クラスには、変更フィード プロセッサーを fluent に構築するための一連のメソッドが用意されています。開始するには、監視対象コンテナー、リース コンテナー、および C\# のデリゲート（変更バッチごとの処理用）が必要です。

1. **Visual Studio Code** の **Explorer** ペインで、**13-change-feed** フォルダーを参照してください。

1. **product.cs** コード ファイルを開いてください。

1. **Product** クラスと対応するプロパティを確認してください。具体的には、このラボでは **id** と **name** プロパティを使用します。

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

1. **client** 変数の **GetContainer** メソッドを使用して、データベース名（*cosmicworks*）とコンテナー名（*products*）で既存のコンテナーを取得し、結果を **Container** 型の **sourceContainer** という名前の変数に格納してください。

    ```
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    ```

1. **client** 変数の **GetContainer** メソッドを使用して、データベース名（*cosmicworks*）とコンテナー名（*productslease*）で既存のコンテナーを取得し、結果を **Container** 型の **leaseContainer** という名前の変数に格納してください。

    ```
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    ```

1. [ChangesHandler<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1] 型の **handleChanges** という名前の新しいデリゲート変数を作成してください。2 つの入力パラメーターを持つ空の非同期匿名関数を使用します。

    1. **IReadOnlyCollection\<Product\>** 型の **changes** という名前のパラメーター。

    1. **CancellationToken** 型の **cancellationToken** という名前のパラメーター。

    ```
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes, 
        CancellationToken cancellationToken
    ) => {
    };
    ```

1. 匿名関数内で、組み込みの **Console.WriteLine** 静的メソッドを使用して、生文字列 **START\tHandling batch of changes...** を出力してください。

    ```
    Console.WriteLine($"START\tHandling batch of changes...");
    ```

1. 引き続き匿名関数内で、**changes** 変数を反復処理する foreach ループを作成してください。ここでは **Product** 型のインスタンスを表す変数として **product** を使用します。

    ```
    foreach(Product product in changes)
    {
    }
    ```

1. 匿名関数の foreach ループ内で、組み込みの非同期 **Console.WriteLineAsync** 静的メソッドを使用して **product** 変数の **id** と **name** プロパティを出力してください。

    ```
    await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
    ```

1. foreach ループおよび匿名関数の外で、**sourceContainer** 変数に対して [GetChangeFeedProcessorBuilder<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder] を次のパラメーターで呼び出した結果を格納する、**builder** という名前の新しい変数を作成してください。

    | **Parameter** | **Value** |
    | ---: | :--- |
    | **processorName** | *productsProcessor* |
    | **onChangesDelegate** | *handleChanges* |

    ```
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
        processorName: "productsProcessor",
        onChangesDelegate: handleChanges
    );
    ```

1. **builder** 変数に対して、[WithInstanceName][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename] メソッド（パラメーター: **consoleApp**）、[WithLeaseContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer] メソッド（パラメーター: **leaseContainer**）、[Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build] メソッドを fluent に呼び出し、結果を [ChangeFeedProcessor][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor] 型の **processor** という名前の変数に格納してください。

    ```
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    ```

1. **processor** 変数の [StartAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync] を非同期で呼び出してください。

    ```
    await processor.StartAsync();
    ```

1. 組み込みの **Console.WriteLine** および **Console.ReadKey** 静的メソッドを使用して、コンソールへ出力し、アプリケーションがキー入力を待機するようにしてください。

    ```
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();  
    ```

1. **processor** 変数の [StopAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync] を非同期で呼び出してください。

    ```
    await processor.StopAsync();
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using Microsoft.Azure.Cosmos;
    using static Microsoft.Azure.Cosmos.Container;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes, 
        CancellationToken cancellationToken
    ) => {
        Console.WriteLine($"START\tHandling batch of changes...");
        foreach(Product product in changes)
        {
            await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
        }
    };
    
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
            processorName: "productsProcessor",
            onChangesDelegate: handleChanges
        );
    
    ChangeFeedProcessor processor = builder
        .WithInstanceName("consoleApp")
        .WithLeaseContainer(leaseContainer)
        .Build();
    
    await processor.StartAsync();
    
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();
    
    await processor.StopAsync();
    ```

1. **script.cs** ファイルを **Save** してください。

1. **Visual Studio Code** で **13-change-feed** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

1. **Visual Studio Code** とターミナルの両方を開いたままにしてください。

    > &#128221; 別のツールを使用して Azure Cosmos DB SQL API コンテナーに項目を生成します。項目を生成したら、このターミナルに戻って出力を確認してください。ターミナルを早く閉じないでください。

## Azure Cosmos DB SQL API アカウントにサンプル データをシードする

ここでは、**cosmicworks** データベースと **products** コンテナーを作成するコマンドライン ユーティリティを使用します。次にこのツールが項目セットを作成し、その変更をターミナル ウィンドウで実行中の変更フィード プロセッサーで確認します。

1. **Visual Studio Code** で **Terminal** メニューを開き、**Split Terminal** を選択して、既存のインスタンスと並べて新しいターミナルを開いてください。

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

1. .NET アプリケーションのターミナル出力を確認してください。変更フィードを通じて送信された各変更に対して、ターミナルに **Detected Operation** メッセージが出力されます。

1. 2 つの統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.startasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
