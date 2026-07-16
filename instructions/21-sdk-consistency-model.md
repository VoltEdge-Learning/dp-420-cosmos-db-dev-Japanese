# Lab 09b - Azure Cosmos DB for NoSQL のレプリケーション戦略を設計および実装する

## ラボ シナリオ

新しい Azure Cosmos DB for NoSQL アカウントの既定の整合性レベルはセッション整合性です。この既定設定は、将来のすべての要求に対して変更できます。さらに個々の要求レベルでは、その特定の要求に対して整合性レベルを緩和することも可能です。

このラボでは、Azure Cosmos DB for NoSQL アカウントの既定整合性レベルを構成し、次に SDK を使用して個別操作の整合性レベルを構成します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する。
- タスク 3: SDK から Azure Cosmos DB for NoSQL アカウントに接続する。
- タスク 4: point 操作の整合性レベルを構成する。

## 推定所要時間: 60 分

## アーキテクチャ図

![image](architecturedia/lab21.png)

## 演習 1: ポータルおよび Azure Cosmos DB for NoSQL SDK で整合性モデルを構成する

### タスク 1: 開発環境を準備する

1. Visual Studio Code を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

2. 左側ペインの **Extension (1)** アイコンを選択してください。検索バーに **C# (2)** を入力し、表示された **extension (3)** を選択して、最後に **Install (4)** を選択してください。

    ![](media/C-hash-extension.png)

3. 画面左上の **file** オプションを選択し、メニューから **Open Folder** を選択して **C:\AllFiles** に移動してください。

4. **dp-420-cosmos-db-dev** フォルダーを選択し、**Select Folder** をクリックしてください。

### タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングするときは、アカウントでサポートする API（例: **API for MongoDB** または **API for NoSQL**）を選択します。Azure Cosmos DB for NoSQL アカウントのプロビジョニング完了後、エンドポイントとキーを取得し、Azure SDK for .NET または任意の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続できます。

1. 新しい Web ブラウザー ウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報でポータルにサインインしてください。

1. **Azure services** カテゴリで **Create a resource** を選択し、次に **Azure Cosmos DB** を選択してください。

    > &#128161; 別の方法として、**&#8801;** メニューを展開し、**All Services** を選択して、**Databases** カテゴリの **Azure Cosmos DB** を選択し、**Create** を選択してください。

1. **Select API option** ペインで、**Azure Cosmos DB for NoSQL** セクション内の **Create** を選択してください。

1. **Create Azure Cosmos DB Account** ペインで、**Basics** タブを確認してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Subscription** | *Your existing Azure subscription* |
    | **Resource group** | *DP-420-DeploymentID* |
    | **Account Name** | *Enter a globally unique name* |
    | **Location** | *Choose any available region* |
    | **Capacity mode** | *Provisioned throughput* |
    | **Apply Free Tier Discount** | *Do Not Apply* |

    >**Note** : DeploymentID は各環境に関連付けられた一意の ID です。値は environment details ページで確認できます。

1. **Next: Global Distribution** をクリックし、Geo-Redundancy を **Enable** に設定して、**Review + Create** をクリックしてください。検証で Success が表示されたら **Create** をクリックしてください。

1. このタスクを続行する前に、デプロイが完了するまで待機してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Replicate data globally** ペインへ移動してください。

1. **Replicate data globally** ペインで、このアカウントに読み取りリージョンを 2 つ追加し、変更を **Save** してください。

1. このタスクを続行する前に、レプリケーション タスクが完了するまで待機してください。

    > **Note:** この操作には約 5〜10 分かかる場合があります。

1. リソース ブレードで **Default consistency** ペインへ移動してください。

1. **Default consistency** ペインで **Strong** オプションを選択し、変更を **Save** してください。

1. このタスクを続行する前に、既定整合性レベルの変更が反映されるまで待機してください。

1. リソース ブレードで **Data Explorer** ペインへ移動してください。

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Database id** | *Create new* &vert; *cosmicworks* |
    | **Share throughput across containers** | *Do not select* |
    | **Container id** | *products* |
    | **Partition key** | */categoryId* |
    | **Container throughput** | *Manual* &vert; *400* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認してください。

1. **Data Explorer** ペインで **cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを展開して、**Items** を選択してください。

1. 引き続き **Data Explorer** ペインで、コマンド バーの **New Item** を選択してください。エディターでプレースホルダー JSON 項目を次の内容に置き換えてください。

    ```
    {
      "id": "7d9273d9-5d91-404c-bb2d-126abb6e4833",
      "categoryId": "78d204a2-7d64-4f4a-ac29-9bfc437ae959",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. コマンド バーの **Save** を選択して JSON 項目を追加してください。

1. **Items** タブで、**Items** ペイン内の新しい項目を確認してください。

1. リソース ブレードで **Keys** ペインへ移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

### タスク 3: SDK から Azure Cosmos DB for NoSQL アカウントに接続する

新しく作成したアカウントの資格情報を使用し、SDK クラスで接続して、新しいデータベースとコンテナーのインスタンスを作成します。その後、Data Explorer を使用して Azure portal 内にインスタンスが存在することを検証します。

1. **Visual Studio Code** の **Explorer** ペインで **21-sdk-consistency-model** フォルダーに移動してください。

1. **21-sdk-consistency-model** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

    > **Note:** このコマンドでは、開始ディレクトリが **21-sdk-consistency-model** フォルダーに設定された状態でターミナルが開きます。

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドしてください。

    ```
    dotnet build
    ```

    > **Note:** **endpoint** と **key** 変数が現在未使用であるというコンパイラ警告が表示される場合があります。このタスクでこれらの変数を使用するため、この警告は無視して問題ありません。

1. 統合ターミナルを閉じてください。

1. **product.cs** コード ファイルを開いてください。

1. **Product** レコードと対応するプロパティを確認してください。特に、このラボでは **id**、**name**、**categoryId** プロパティを使用します。

1. **Visual Studio Code** の **Explorer** ペインに戻り、**script.cs** コード ファイルを開いてください。

    > **Note:** **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** ライブラリは NuGet からすでに事前インポートされています。

1. **endpoint** という名前の **string** 変数を見つけてください。その値を先ほど作成した Azure Cosmos DB アカウントの **endpoint** に設定してください。

    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > **Note:** たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の **string** 変数を見つけてください。その値を先ほど作成した Azure Cosmos DB アカウントの **key** に設定してください。

    ```
    string key = "<cosmos-key>";
    ```

    > **Note:** たとえば key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. **script.cs** コード ファイルを **Save** してください。

### タスク 4: point 操作の整合性レベルを構成する

**ItemRequestOptions** クラスには、要求単位の構成プロパティが含まれています。このクラスを使って、現在の既定値である strong から eventual 整合性へ緩和します。

1. **id** という名前の string 変数を作成し、値 **7d9273d9-5d91-404c-bb2d-126abb6e4833** を設定してください。

    ```
    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";
    ```

1. **categoryId** という名前の string 変数を作成し、値 **78d204a2-7d64-4f4a-ac29-9bfc437ae959** を設定してください。

    ```
    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    ```

1. **PartitionKey** 型の **partitionKey** という変数を作成し、コンストラクター パラメーターとして **categoryId** 変数を渡してください。

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. **container** 変数のジェネリック **ReadItemAsync\<\>** メソッドを非同期で呼び出し、**id** と **partitionkey** をメソッド パラメーターとして渡してください。ジェネリック型は **Product** を使用し、結果を **ItemResponse\<Product\>** 型の **response** という変数に格納してください。

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. 静的 **Console.WriteLine** メソッドを呼び出し、フォーマット済み出力文字列を使って要求課金を表示してください。

    ```
    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. 完了後、コード ファイルに次の内容が含まれていることを確認してください。

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Container container = client.GetContainer("cosmicworks", "products");

    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";

    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);

    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);

    Console.WriteLine($"STRONG Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **21-sdk-consistency-model** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```

1. ターミナル出力を確認してください。要求課金（RU）がコンソールに表示されるはずです。

    > **Note:** 現在の要求課金は **2 RU** になるはずです。これは strong 整合性が最新書き込みを保証するために少なくとも 2 つのレプリカからの読み取りを必要とするためです。

1. 統合ターミナルを閉じてください。

1. **script.cs** コード ファイルのエディター タブに戻ってください。

1. 次のコード行を削除してください。

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey);

    Console.WriteLine($"Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. [ItemRequestOptions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions] 型の **options** という新しい変数を作成し、[ConsistencyLevel][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemrequestoptions.consistencylevel] プロパティを [ConsistencyLevel.Eventual][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.consistencylevel] 列挙値に設定してください。

    ```
    ItemRequestOptions options = new()
    {
        ConsistencyLevel = ConsistencyLevel.Eventual
    };
    ```

1. **container** 変数のジェネリック **ReadItemAsync\<\>** メソッドを非同期で呼び出し、**id**、**partitionKey**、**options** をメソッド パラメーターとして渡してください。ジェネリック型は **Product** を使用し、結果を **ItemResponse\<Product\>** 型の **response** という変数に格納してください。

    ```
    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);
    ```

1. 静的 **Console.WriteLine** メソッドを呼び出し、フォーマット済み出力文字列を使って要求課金を表示してください。

    ```
    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. 完了後、コード ファイルに次の内容が含まれていることを確認してください。

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Container container = client.GetContainer("cosmicworks", "products");

    string id = "7d9273d9-5d91-404c-bb2d-126abb6e4833";

    string categoryId = "78d204a2-7d64-4f4a-ac29-9bfc437ae959";
    PartitionKey partitionKey = new (categoryId);

    ItemRequestOptions options = new()
    {
        ConsistencyLevel = ConsistencyLevel.Eventual
    };

    ItemResponse<Product> response = await container.ReadItemAsync<Product>(id, partitionKey, requestOptions: options);

    Console.WriteLine($"EVENTUAL Request Charge:\t{response.RequestCharge:0.00} RUs");
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **21-sdk-consistency-model** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```

1. ターミナル出力を確認してください。要求課金（RU）がコンソールに表示されるはずです。

    > **Note:** 現在の要求課金は **1 RU** になるはずです。これは eventual 整合性が単一レプリカからの読み取りのみを必要とするためです。

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。


## クリーンアップ

1. このラボで作成した Azure Cosmos DB アカウントを削除してください。

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- Azure Cosmos DB for NoSQL アカウントを作成した。
- SDK から Azure Cosmos DB for NoSQL アカウントへ接続した。
- point 操作の整合性レベルを構成した。

### ラボは正常に完了しました