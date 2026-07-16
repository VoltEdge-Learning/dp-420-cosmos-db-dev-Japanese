# Lab 07a - Azure Cosmos DB for NoSQL を Azure サービスと統合する

## ラボ シナリオ

Azure Cosmos DB for NoSQL の変更フィードは、プラットフォームのイベントを契機として動作する補助アプリケーションを作成するための重要な仕組みです。Azure Cosmos DB for NoSQL 向け .NET SDK には、変更フィードと統合し、コンテナー内の操作通知を受信するアプリケーションを構築するためのクラス群が含まれています。

このラボでは、.NET SDK の変更フィード プロセッサ機能を使用して、指定コンテナー内の項目に対して作成または更新操作が実行されたときに通知を受け取るアプリケーションを作成します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する。
- タスク 3: .NET SDK で変更フィード プロセッサを実装する。
- タスク 4: Azure Cosmos DB for NoSQL アカウントにサンプル データを投入する。

## 推定所要時間: 60 分

## アーキテクチャ図

![image](architecturedia/lab13.png)

## 演習 1: Azure Cosmos DB for NoSQL SDK を使用して変更フィード イベントを処理する

### タスク 1: 開発環境を準備する

作業環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、次の手順でクローンしてください。すでにクローン済みの場合は、以前クローンしたフォルダーを **Visual Studio Code** で開いてください。

1. Visual Studio Code を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

1. 左側ペインの **Extension (1)** アイコンを選択してください。検索バーに **C# (2)** を入力し、表示された **extension (3)** を選択して、最後に拡張機能の **Install (4)** を選択してください。

    ![](media/C-hash-extension.png)

1. ファイルを開きます。左上メニューから **file->Open Folder** をクリックし、**C:\AllFiles** に移動してください。

1. **dp-420-cosmos-db-dev-stage** フォルダーを選択し、**Select Folder** をクリックしてください。

### タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングするときは、アカウントでサポートする API（例: **API for MongoDB** または **API for NoSQL**）を選択します。Azure Cosmos DB for NoSQL アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の SDK を使用して接続できます。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報でポータルにサインインしてください。

1. **Azure services** カテゴリ内で **Create a resource** を選択し、次に **Azure Cosmos DB** を選択してください。

    > &#128161; 別の方法として、**&#8801;** メニューを展開し、**All Services** を選択して、**Databases** カテゴリの **Azure Cosmos DB** を選択し、**Create** を選択してください。

1. **Select API option** ペインで、**Azure Cosmos DB for NoSQL** セクション内の **Create** を選択してください。

1. **Create Azure Cosmos DB Account** ペインで、**Basics** タブを確認してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Subscription** | *Your existing Azure subscription* |
    | **Resource group** | *Select an existing resource group* |
    | **Account Name** | *Enter a globally unique name* |
    | **Location** | *Choose any available region* |
    | **Capacity mode** | *Serverless* |

    > &#128221; ラボ環境によっては新しいリソース グループの作成が制限されている場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. **Review + Create** をクリックし、検証で Success が表示されたら **Create** をクリックしてください。

1. このタスクを続行する前に、デプロイが完了するまで待機してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Keys** ペインに移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

1. リソース メニューから **Data Explorer** を選択してください。

1. **Data Explorer** ペインで **New Container** を展開し、**New Database** を選択してください。

1. **New Database** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Database id** | *cosmicworks* |

1. **Data Explorer** ペインに戻り、階層内の **cosmicworks** データベース ノードを確認してください。

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Database id** | *Use existing* &vert; *cosmicworks* |
    | **Container id** | *products* |
    | **Partition key** | */categoryId* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認してください。

1. **Data Explorer** ペインで再度 **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Database id** | *Use existing* &vert; *cosmicworks* |
    | **Container id** | *productslease* |
    | **Partition key** | */partitionKey* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **productslease** コンテナー ノードを確認してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

### タスク 3: .NET SDK で変更フィード プロセッサを実装する

**Microsoft.Azure.Cosmos.Container** クラスには、変更フィード プロセッサを fluent に構築するための一連のメソッドがあります。開始するには、監視対象コンテナー、リース コンテナー、および C\# のデリゲート（変更バッチを処理するため）が必要です。

1. **Visual Studio Code** の **Explorer** ペインで、**13-change-feed** フォルダーに移動してください。

1. **product.cs** コード ファイルを開いてください。

1. **Product** クラスと対応するプロパティを確認してください。特にこのラボでは **id** と **name** プロパティを使用します。

1. **Visual Studio Code** の **Explorer** ペインに戻り、**script.cs** コード ファイルを開いてください。

1. 既存の **endpoint** という名前の変数を更新し、先ほど作成した Azure Cosmos DB アカウントの **endpoint** を設定してください。

    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. 既存の **key** という名前の変数を更新し、先ほど作成した Azure Cosmos DB アカウントの **key** を設定してください。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. **client** 変数の **GetContainer** メソッドを使用し、データベース名（*cosmicworks*）とコンテナー名（*products*）で既存コンテナーを取得して、**sourceContainer** という名前の **Container** 型変数に格納してください。

    ```
    Container sourceContainer = client.GetContainer("cosmicworks", "products");
    ```

1. **client** 変数の **GetContainer** メソッドを使用し、データベース名（*cosmicworks*）とコンテナー名（*productslease*）で既存コンテナーを取得して、**leaseContainer** という名前の **Container** 型変数に格納してください。

    ```
    Container leaseContainer = client.GetContainer("cosmicworks", "productslease");
    ```

1. [ChangesHandler<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.changefeedhandler-1] 型の **handleChanges** という新しいデリゲート変数を作成してください。次の 2 つの入力パラメーターを持つ空の非同期匿名関数を使用します。

    1. **IReadOnlyCollection\<Product\>** 型の **changes** という名前のパラメーター。

    1. **CancellationToken** 型の **cancellationToken** という名前のパラメーター。

    ```
    ChangesHandler<Product> handleChanges = async (
        IReadOnlyCollection<Product> changes,
        CancellationToken cancellationToken
    ) => {
    };
    ```

1. 匿名関数内で、組み込みの静的 **Console.WriteLine** メソッドを使用し、生文字列 **START\tHandling batch of changes...** を出力してください。

    ```
    Console.WriteLine($"START\tHandling batch of changes...");
    ```

1. 引き続き匿名関数内で、**changes** 変数を反復処理する foreach ループを作成し、型 **Product** のインスタンスを表す変数として **product** を使用してください。

    ```
    foreach(Product product in changes)
    {
    }
    ```

1. 匿名関数内の foreach ループで、組み込みの非同期静的 **Console.WriteLineAsync** メソッドを使用し、**product** 変数の **id** と **name** プロパティを出力してください。

    ```
    await Console.Out.WriteLineAsync($"Detected Operation:\t[{product.id}]\t{product.name}");
    ```

1. foreach ループと匿名関数の外側で、**sourceContainer** 変数に対して [GetChangeFeedProcessorBuilder<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.getchangefeedprocessorbuilder] を次のパラメーターで呼び出し、結果を格納する **builder** という新しい変数を作成してください。

    | **Parameter** | **Value** |
    | --- | --- |
    | **processorName** | *productsProcessor* |
    | **onChangesDelegate** | *handleChanges* |

    ```
    var builder = sourceContainer.GetChangeFeedProcessorBuilder<Product>(
        processorName: "productsProcessor",
        onChangesDelegate: handleChanges
    );
    ```

1. **builder** 変数に対して fluent に [WithInstanceName][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withinstancename]（パラメーター **consoleApp**）、[WithLeaseContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.withleasecontainer]（パラメーター **leaseContainer**）、[Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessorbuilder.build] を呼び出し、結果を **processor** という名前の [ChangeFeedProcessor][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor] 型変数に格納してください。

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

1. 組み込みの静的 **Console.WriteLine** および **Console.ReadKey** メソッドを使用し、コンソールに出力してキー入力待ち状態にしてください。

    ```
    Console.WriteLine($"RUN\tListening for changes...");
    Console.WriteLine("Press any key to stop");
    Console.ReadKey();
    ```

1. **processor** 変数の [StopAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.changefeedprocessor.stopasync] を非同期で呼び出してください。

    ```
    await processor.StopAsync();
    ```

1. 完了後、コード ファイルに次の内容が含まれていることを確認してください。

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

1. **Visual Studio Code** で **13-change-feed** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```

1. **Visual Studio Code** とターミナルの両方を開いたままにしてください。

    > &#128221; Azure Cosmos DB for NoSQL コンテナーに項目を生成するため、別のツールを使用します。項目生成後、このターミナルに戻って出力を確認します。ターミナルを途中で閉じないでください。

### タスク 4: Azure Cosmos DB for NoSQL アカウントにサンプル データを投入する

**cosmicworks** データベースと **products** コンテナーを作成するコマンドライン ユーティリティを使用します。ツールはその後、項目セットを作成し、ターミナルで実行中の変更フィード プロセッサでその内容を確認できます。

1. **Visual Studio Code** で **Terminal** メニューを開き、**Split Terminal** を選択して、既存インスタンスの横に新しいターミナルを開いてください。

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをマシン全体で利用できるようにインストールしてください。

    ```
    dotnet tool install --global cosmicworks
    ```

    > &#128161; このコマンドの完了には数分かかる場合があります。過去にこのツールの最新バージョンをすでにインストールしている場合、このコマンドは警告メッセージ（*Tool 'cosmicworks' is already installed'）を出力します。

1. 次のコマンドライン オプションで cosmicworks を実行し、Azure Cosmos DB アカウントにデータを投入してください。

    | **Option** | **Value** |
    | --- | --- |
    | **--endpoint** | *このラボで先ほどコピーした endpoint 値* |
    | **--key** | *このラボで先ほどコピーした key 値* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/** で key が **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。
    > ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

    >**Note**: エラーが発生する場合は、Visual Studio Code を閉じて再度開き、もう一度コマンドを実行してください。

1. **cosmicworks** コマンドがアカウントへのデータベース、コンテナー、項目の投入を完了するまで待機してください。

1. .NET アプリケーション側のターミナル出力を確認してください。変更フィード経由で送信された各変更について、ターミナルに **Detected Operation** メッセージが出力されます。

1. 統合ターミナルを両方とも閉じてください。

1. **Visual Studio Code** を閉じてください。

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- Azure Cosmos DB for NoSQL アカウントを作成した。
- .NET SDK で変更フィード プロセッサを実装した。
- Azure Cosmos DB for NoSQL アカウントにサンプル データを投入した。

### ラボは正常に完了しました