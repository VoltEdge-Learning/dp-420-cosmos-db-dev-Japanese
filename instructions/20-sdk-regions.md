# Lab 09a - Azure Cosmos DB for NoSQL のレプリケーション戦略を設計および実装する

## ラボ シナリオ

このラボでは、顧客エンティティを個別コンテナーとしてモデル化した場合と、NoSQL データベース向けに単一ドキュメントへ埋め込んでモデル化した場合の違いを測定します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する。
- タスク 3: SDK から Azure Cosmos DB for NoSQL アカウントに接続する。
- タスク 4: 優先リージョン リストで .NET SDK を構成する。

## 推定所要時間: 60 分

## アーキテクチャ図

![image](architecturedia/lab20.png)

## 演習 1: Azure Cosmos DB for NoSQL SDK を使用して異なるリージョンへ接続する

Azure Cosmos DB for NoSQL アカウントで geo-redundancy を有効にすると、構成した任意の順序で SDK から各リージョンのデータを読み取れるようになります。この手法は、利用可能なすべての読み取りリージョンへ読み取り要求を分散するときに有効です。

このラボでは、手動で構成したフォールバック順序で読み取りリージョンへ接続するように CosmosClient クラスを構成します。

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

1. **Review + Create** をクリックし、検証で Success が表示されたら **Create** をクリックしてください。

1. このタスクを続行する前に、デプロイが完了するまで待機してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Replicate data globally** ペインへ移動してください。

1. **Replicate data globally** ペインで、このアカウントに読み取りリージョンを 2 つ追加し、変更を **Save** してください。

1. このタスクを続行する前に、レプリケーション タスクが完了するまで待機してください。

    > **Note:** この操作には約 5〜10 分かかる場合があります。

1. **Write**（プライマリ）リージョン名と 2 つの **Read** リージョン名を記録してください。この演習の後半でこれらのリージョン名を使用します。

    > **Note:** たとえば、プライマリ リージョンが **North Europe** で、2 つの読み取りセカンダリ リージョンが **East US 2** および **South Africa North** の場合は、この 3 つの名前をそのまま記録してください。

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

### タスク 3: SDK から Azure Cosmos DB for NoSQL アカウントへ接続する

新しく作成したアカウントの資格情報を使用し、SDK クラスで接続して、別リージョンからデータベースおよびコンテナーのインスタンスへアクセスします。

1. **Visual Studio Code** の **Explorer** ペインで **20-sdk-regions** フォルダーに移動してください。

1. **20-sdk-regions** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

    > **Note:** このコマンドでは、開始ディレクトリが **20-sdk-regions** フォルダーに設定された状態でターミナルが開きます。

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドしてください。

    ```
    dotnet build
    ```

    > **Note:** **endpoint** と **key** 変数が現在未使用であるというコンパイラ警告が表示される場合があります。このタスクでこれらの変数を使用するため、この警告は無視して問題ありません。

1. 統合ターミナルを閉じてください。

1. **20-sdk-regions** フォルダー内の **script.cs** コード ファイルを開いてください。

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

### タスク 4: 優先リージョン リストで .NET SDK を構成する

**CosmosClientOptions** クラスには、SDK で接続したいリージョン リストを構成するプロパティが含まれています。このリストはフェールオーバー優先順位で並び、構成した順序で各リージョンへの接続を試行します。

1. **List\<string\>** ジェネリック型の新しい変数を作成し、アカウントで構成したリージョンを 3 番目のリージョンから 1 番目（プライマリ）リージョンの順で含めてください。たとえば Azure Cosmos DB for NoSQL アカウントを **West US** で作成し、次に **South Africa North**、最後に **East Asia** を追加した場合、リスト変数は次のようになります。

    ```
    List<string> regions = new()
    {
        "East Asia",
        "South Africa North",
        "West US"
    };
    ```

    > **Note:** 別の方法として、さまざまな Azure リージョンの組み込み文字列プロパティを含む [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions] 静的クラスを使用できます。

1. **regions** 変数を [ApplicationPreferredRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclientoptions.applicationpreferredregions] プロパティに設定した **CosmosClientOptions** クラスの新しいインスタンス **options** を作成してください。

    ```
    CosmosClientOptions options = new ()
    {
        ApplicationPreferredRegions = regions
    };
    ```

1. **endpoint**、**key**、**options** 変数をコンストラクター パラメーターとして渡し、**CosmosClient** クラスの新しいインスタンス **client** を作成してください。

    ```
    CosmosClient client = new (endpoint, key, options);
    ```

1. **client** 変数の **GetContainer** メソッドを使用し、データベース名（*cosmicworks*）とコンテナー名（*products*）で既存コンテナーを取得してください。

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. **container** 変数の [ReadItemAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] メソッドを使用し、サーバーから特定項目を取得して、nullable [ItemResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.itemresponse] 型の **response** という変数に結果を格納してください。

    ```
    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );
    ```

1. 静的 **Console.WriteLine** メソッドを呼び出し、現在の項目識別子と JSON 診断データを出力してください。

    ```
    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine($"Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. 完了後、コード ファイルに次の内容が含まれていることを確認してください。

    ```
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    List<string> regions = new()
    {
        "<read-region-2>",
        "<read-region-1>",
        "<write-region>"
    };

    CosmosClientOptions options = new ()
    {
        ApplicationPreferredRegions = regions
    };

    using CosmosClient client = new(endpoint, key, options);

    Container container = client.GetContainer("cosmicworks", "products");

    ItemResponse<dynamic> response = await container.ReadItemAsync<dynamic>(
        "7d9273d9-5d91-404c-bb2d-126abb6e4833",
        new PartitionKey("78d204a2-7d64-4f4a-ac29-9bfc437ae959")
    );

    Console.WriteLine($"Item Id:\t{response.Resource.Id}");
    Console.WriteLine("Response Diagnostics JSON");
    Console.WriteLine($"{response.Diagnostics}");
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **20-sdk-regions** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```

1. ターミナル出力を確認してください。コンテナー名と JSON 診断データがコンソールに出力されるはずです。

1. JSON 診断データを確認してください。**HttpResponseStats** というプロパティと、その子プロパティ **RequestUri** を探してください。このプロパティ値は、このラボで先ほど構成した名前とリージョンを含む URI になるはずです。

    > **Note:** たとえばアカウント名が **dp420** で、最初に構成したリージョンが **East Asia** の場合、JSON プロパティ値は **dp420-eastasia.documents.azure.com/dbs/cosmicworks/colls/products** になります。

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

## クリーンアップ

1. このラボで作成した Azure Cosmos DB アカウントを削除してください。

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- Azure Cosmos DB for NoSQL アカウントを作成した。
- SDK から Azure Cosmos DB for NoSQL アカウントへ接続した。
- 優先リージョン リストで .NET SDK を構成した。

### ラボは正常に完了しました