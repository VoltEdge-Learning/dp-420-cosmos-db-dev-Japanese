# Azure Cosmos DB for NoSQL SDK を使用して複数のドキュメントを一括移動する

## ラボ シナリオ

一括操作の実行方法を学ぶ最も簡単な方法は、クラウド上の Azure Cosmos DB for NoSQL アカウントに大量のドキュメントを送信してみることです。SDK の一括機能を使用すれば、[System.Threading.Tasks][docs.microsoft.com/dotnet/api/system.threading.tasks] 名前空間を少し活用するだけで実現できます。

このラボでは、NuGet の [Bogus][nuget.org/packages/bogus/33.1.1] ライブラリを使用して架空データを生成し、それを Azure Cosmos DB アカウントに格納します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: Azure Cosmos DB for NoSQL アカウントを作成し、SDK プロジェクトを構成する。
- タスク 2: 25,000 件のドキュメントを一括挿入する。
- タスク 3: 結果を確認する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab8.png)

## 開発環境を準備する

1. Visual Studio Code を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

   ![Visual Studio Code Icon](./media/vscode1.jpg)

3. 左側ペインの **Extension (1)** アイコンを選択してください。検索バーに **C# (2)** と入力し、表示された **Extension (3)** を選択して、最後に拡張機能の **Install (4)** を選択してください。

    ![](media/visualstudioo.png)

4. 画面左上の **file** を選択し、メニューから **Open Folder** を選択して **C:\AllFiles** に移動してください。

5. **dp-420-cosmos-db-dev-main** フォルダーを選択し、**Select Folder** をクリックしてください。

   ![06](media/New-image54.png)

   >**Note:** **Do you trust the authors of the files in this folder?** ポップアップでは、**Yes, I trust authors** を選択してください。

    ![06](media/DB24.png)

### タスク 1: Azure Cosmos DB for NoSQL アカウントを作成し、SDK プロジェクトを構成する

このタスクでは、Azure Cosmos DB for NoSQL アカウントを作成し、基本設定を構成したうえで、Visual Studio Code の SDK プロジェクトを準備して新しいデータベースと連携できるようにします。

1. Azure Portal ページで、ポータル上部の Search resources, services and docs (G+/) ボックスに **Azure Cosmos DB (1)** と入力し、services の下にある **Azure Cosmos DB (2)** を選択してください。

   ![06](media/New-image1.png)

1. **Azure Cosmos DB for NoSQL** の下で **+ Create (1)** を選択し、**Create (2)** をクリックして **Azure Cosmos DB for NoSQL** アカウントを作成してください。

    ![06](media/New-image2.png)

    ![06](media/New-image3.png)

1. 次の設定を指定し、その他の設定は既定値のままにして **Review + create (10)** を選択してください。

    | **Setting**         | **Value** |
    | --------------------|--------------------------------------------------- |
    | **Workload Type**   | *Production* (1) |
    | **Subscription**    | *Your existing Azure subscription* (2) |
    | **Resource group**  | *Select an existing Cosmosdb-<inject key="DeploymentID" enableCopy="false"/>* (3) |
    | **Account Name**    | *sql-<inject key="DeploymentID" enableCopy="false"/>* (4) |
    | **Location**        | *Choose the default region* (5) |
    | **Capacity mode**   | *Provisioned throughput* (6) |
    | **Apply Free Tier Discount** | *Do Not Apply* (7) |
    | **Limit the total amount of throughput that can be provisioned on this account** | *Unchecked* (8) |

     ![06](media/DB25.png)

1. **Create** をクリックしてください。

    ![06](media/New-image5.png)

1. このタスクを続行する前に、デプロイが完了するまで待機してください。

1. デプロイ完了後、**Go to resources** を選択してください。

    ![06](media/New-image6.png)

1. **Azure Cosmos DB account** で、左側メニューの **Settings (1)** を展開し、**Keys (2)** を選択してください。

    ![06](media/DB15.png)

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

   - **URI** フィールドをコピーしてください。この演習の後半でこの **endpoint** 値を使用します。

   - **PRIMARY KEY** フィールドをコピーしてください。この演習の後半でこの **key** 値を使用します。

       ![06](media/New-image9.png)

1. **Azure Cosmos DB** アカウント リソースの **overview page (1)** で、**Data Explorer (2)** ペインに移動してください。

    ![06](media/DB04.png)

1. **Data Explorer (1)** ページで **New (2)** をクリックし、次に **New Container (3)** を選択してください。

     ![06](media/DB42.png)

1. **New Container** ペインで次の内容を入力し、**OK (6)** をクリックしてください。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Database id** | Create new \| `cosmicworks` **(1)** |
    | **Share throughput across containers** | Unchecked **(2)** |
    | **Container id** | `products` **(3)** |
    | **Partition key** | `/categoryId` **(4)** |
    | **Container throughput** | Autoscale \| `4000` **(5)** |

    ![06](media/DB44.png)

1. **Visual Studio Code** に戻ってください。

1. **Visual Studio Code** の **Explorer** ペインで、**08-sdk-bulk** フォルダーを開いてください。

1. **Visual Studio Code** の **06-sdk-crud (1)** フォルダーで、空の **script.cs (2)** コード ファイルを開いてください。

    ![06](media/DB45.png)

    >**Note**: **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** ライブラリは NuGet からすでに事前にインポートされています。

1. **endpoint** という名前の **string** 変数を見つけてください。前のラボで作成した Azure Cosmos DB アカウントの **endpoint** 値を設定してください。

    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    >**Note**: たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の **string** 変数を見つけてください。前のラボで作成した Azure Cosmos DB アカウントの **key** 値を設定してください。

    ```
    string key = "<cosmos-key>";
    ```

    >**Note**: たとえば key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. **Ctrl+S** を押して script.cs コード ファイルを **Save** してください。

1. **08-sdk-bulk (1)** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal (2)** を選択して新しいターミナルを開いてください。

    ![](media/DB46.png)

   >**Note**: このコマンドでは、開始ディレクトリがすでに **08-sdk-bulk** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加してください。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドしてください。

    ```
    dotnet build
    ```

1. 統合ターミナルを閉じてください。

    > タスク完了おめでとうございます。次は検証です。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、次のタスクに進めます。
    > - 表示されない場合は、エラーメッセージをよく確認し、ラボ ガイドの手順に従って再試行してください。
    > - サポートが必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

    <validation step="8b577a01-0d31-4606-8273-71efbf77241f" />

### タスク 2: 25,000 件のドキュメントを一括挿入する

このタスクでは、多数のドキュメントを挿入して動作を確認します。内部テストでは、ラボ VM と Azure Cosmos DB NoSQL API アカウントの地理的距離が比較的近い場合、約 1～2 分かかります。

1. **script.cs** コード ファイルのエディター タブに戻ってください。

1. **AllowBulkExecution** プロパティを **true** に設定した、**options** という名前の [CosmosClientOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions] クラスの新しいインスタンスを作成してください。

    ```
    CosmosClientOptions options = new ()
    {
        AllowBulkExecution = true
    };
    ```

1. **endpoint**、**key**、**options** 変数をコンストラクター パラメーターとして渡し、**client** という名前の **CosmosClient** クラスの新しいインスタンスを作成してください。

    ```
    CosmosClient client = new (endpoint, key, options);
    ```

1. **client** 変数の [GetContainer][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.getcontainer] メソッドを使用して、データベース名（*cosmicworks*）とコンテナー名（*products*）を指定し、既存のコンテナーを取得してください。

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. NuGet からインポートした Bogus ライブラリの **Faker** クラスを使用して、次のサンプル コードで架空の製品 **25,000** 件を生成してください。

    ```
    List<Product> productsToInsert = new Faker<Product>()
        .StrictMode(true)
        .RuleFor(o => o.id, f => Guid.NewGuid().ToString())
        .RuleFor(o => o.name, f => f.Commerce.ProductName())
        .RuleFor(o => o.price, f => Convert.ToDouble(f.Commerce.Price(max: 1000, min: 10, decimals: 2)))
        .RuleFor(o => o.categoryId, f => f.Commerce.Department(1))
        .Generate(25000);
    ```

    >**Note**: [Bogus][nuget.org/packages/bogus/33.1.1] ライブラリは、UI アプリケーションのテスト用に架空データを設計するオープンソース ライブラリであり、一括インポート/エクスポート アプリケーション開発の学習に最適です。

1. **concurrentTasks** という名前の、**Task** 型を持つ新しいジェネリック **List<>** を作成してください。

    ```
    List<Task> concurrentTasks = new List<Task>();
    ```

1. このアプリケーションで先ほど生成した製品一覧を反復処理する for-each ループを作成してください。

    ```
    foreach(Product product in productsToInsert)
    {
    }
    ```

1. for-each ループ内で、製品を Azure Cosmos DB NoSQL API に非同期挿入する **Task** を作成してください。パーティション キーを明示的に指定し、**concurrentTasks** というタスク リストに追加してください。

    ```
    concurrentTasks.Add(
        container.CreateItemAsync(product, new PartitionKey(product.categoryId))
    );
    ```

1. foreach ループの後で、**concurrentTasks** 変数に対する **Task.WhenAll** の結果を非同期で待機してください。

    ```
    await Task.WhenAll(concurrentTasks);
    ```

1. 組み込みの静的 **Console.WriteLine** メソッドを使用し、**Bulk tasks complete** という固定メッセージをコンソールに出力してください。

    ```
    Console.WriteLine("Bulk tasks complete");
    ```

1. 完了後、コード ファイルに次の内容が含まれていることを確認してください。

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

1. **Ctrl+S** を押して script.cs コード ファイルを **Save** してください。

1. **08-sdk-bulk (1)** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal (2)** を選択して新しいターミナルを開いてください。

    ![](media/DB46.png)

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```

1. アプリケーションは無言で実行され、完了までに約 1～2 分かかるはずです。

    ![](media/DB47.png)

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

    > タスク完了おめでとうございます。次は検証です。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、次のタスクに進めます。
    > - 表示されない場合は、エラーメッセージをよく確認し、ラボ ガイドの手順に従って再試行してください。
    > - サポートが必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

    <validation step="5bf085fc-efec-4bde-9f59-38074db269a2" />

### タスク 3: 結果を確認する

25,000 件の項目を Azure Cosmos DB に送信したので、Data Explorer で確認します。

1. ブラウザーで Azure portal (``portal.azure.com``) に戻り、このラボで作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソースの **overview page (1)** で、**Data Explorer (2)** ペインに移動してください。

    ![06](media/DB04.png)

1. **Data Explorer** で、**cosmicworks (2)** データベース ノードを展開し、**products (3)** コンテナー ノードを展開してください。

    ![06](media/DB41.png)

1. **NoSQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選択し、**... (1)** をクリックしてから **New SQL Query (2)** を選択してください。

   ![](media/DB48.png)

1. エディター領域の内容を削除してください。

1. 次のクエリを **(1)** のとおり入力し、**Execute Query (2)** をクリックしてください。

    ```
    SELECT COUNT(1) FROM items
    ```

     ![](media/DB49.png)

1. **results (3)** を表示し、コンテナー内の項目数を確認してください。

### レビュー

このラボでは、次を完了しました。

- Azure Cosmos DB for NoSQL アカウントを作成し、SDK プロジェクトを構成した。
- 25,000 件のドキュメントを一括挿入した。
- 結果を確認した。

### まとめ
このラボでは、Azure Cosmos DB SDK と Bogus ライブラリを使用して、Azure Cosmos DB for NoSQL アカウントに架空のドキュメント 25,000 件を一括挿入する方法を学び、必要な環境構成と結果確認も実施しました。

### ラボは正常に完了しました