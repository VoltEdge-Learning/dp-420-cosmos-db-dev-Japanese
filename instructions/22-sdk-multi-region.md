# Lab 09c - Azure Cosmos DB for NoSQL のレプリケーション戦略を設計および実装する

## ラボ シナリオ

**CosmosClientBuilder** クラスは、コンテナーに接続して操作を実行する SDK クライアントを構築するための fluent クラスです。Azure Cosmos DB for NoSQL アカウントがすでにマルチリージョン書き込み向けに構成されている場合、この builder を使用して書き込み操作の優先アプリケーション リージョンを設定できます。

このラボでは、複数リージョンで Azure Cosmos DB for NoSQL アカウントを構成し、マルチリージョン書き込みを有効にします。その後、SDK を使用して特定リージョンに対する操作を実行します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する。
- タスク 3: SDK から Azure Cosmos DB for NoSQL アカウントに接続する。
- タスク 4: SDK の書き込みリージョンを構成する。

## 推定所要時間: 60 分

## アーキテクチャ図

![image](architecturedia/lab22.png)

## 演習 1: Azure Cosmos DB for NoSQL SDK を使用してマルチリージョン書き込みアカウントに接続する

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

1. **Replicate data globally** ペインで、このアカウントに追加リージョンを少なくとも 1 つ追加してください。

1. 引き続き **Replicate data globally** ペイン内で **Multi-region writes** を有効化し、変更を **Save** してください。

1. このタスクを続行する前に、レプリケーション タスクが完了するまで待機してください。

    > **Note:** この操作には約 5〜10 分かかる場合があります。

1. 追加したリージョンの値を少なくとも 1 つ記録してください。この演習の後半でこのリージョン値を使用します。

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

1. リソース ブレードで **Keys** ペインへ移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

### タスク 3: SDK から Azure Cosmos DB for NoSQL アカウントに接続する

新しく作成したアカウントの資格情報を使用し、SDK クラスで接続して、新しいデータベースとコンテナーのインスタンスを作成します。その後、Data Explorer を使用して Azure portal 内にインスタンスが存在することを検証します。

1. **Visual Studio Code** の **Explorer** ペインで **22-sdk-multi-region** フォルダーに移動してください。

1. **22-sdk-multi-region** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

    > **Note:** このコマンドでは、開始ディレクトリが **22-sdk-multi-region** フォルダーに設定された状態でターミナルが開きます。

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

### タスク 4: SDK の書き込みリージョンを構成する

fluent な **WithApplicationRegion** メソッドは、builder クラスを使用して後続操作の優先リージョンを構成するために使用します。

1. [CosmosClientBuilder][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder] クラスの新しいインスタンス **builder** を作成し、コンストラクター パラメーターとして **endpoint** と **key** 変数を渡してください。

    ```
    CosmosClientBuilder builder = new (endpoint, key);
    ```

1. **region** という名前の **string** 型の新しい変数を作成し、このラボで先ほど作成した追加リージョン名を設定してください。たとえば Azure Cosmos DB for NoSQL アカウントを **East US** リージョンで作成し、次に **Brazil South** を追加した場合、string 変数は次のようになります。

    ```
    string region = "Brazil South";
    ```

    > **Note:** 別の方法として、さまざまな Azure リージョンの組み込み文字列プロパティを含む [Microsoft.Azure.Cosmos.Regions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.regions] 静的クラスを使用できます。

1. **builder** 変数に対して [WithApplicationRegion][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.withapplicationregion] メソッド（パラメーターは **region**）と [Build][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.fluent.cosmosclientbuilder.build] メソッドを fluent に呼び出してください。結果は **CosmosClient** 型の **client** という変数に格納し、using ステートメント内で使用してください。

    ```
    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();
    ```

1. **client** 変数の **GetContainer** メソッドを使用し、データベース名（*cosmicworks*）とコンテナー名（*products*）で既存コンテナーを取得してください。

    ```
    Container container = client.GetContainer("cosmicworks", "products");
    ```

1. **Guid** の新しい値を生成して文字列として保持し、**id** と **categoryId** という 2 つの **string** 変数を作成してください。

    ```
    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    ```

1. **item** という名前の **Product** 型の新しい変数を作成し、コンストラクター パラメーターとして **id** 変数、文字列値 **Polished Bike Frame**、**categoryId** 変数を渡してください。

    ```
    Product item = new (id, "Polished Bike Frame", categoryId);
    ```

1. **container** 変数の **CreateItemAsync\<\>** メソッドを非同期で呼び出し、**item** 変数をパラメーターとして渡して、結果を **response** という変数に格納してください。

    ```
    var response = await container.CreateItemAsync<Product>(item);
    ```

1. 静的 **Console.WriteLine** メソッドを呼び出し、response の HTTP ステータス コードと要求課金（request units）を表示してください。

    ```
    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. 完了後、コード ファイルに次の内容が含まれていることを確認してください。

    ```
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Fluent;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClientBuilder builder = new (endpoint, key);

    string region = "West Europe";

    using CosmosClient client = builder
        .WithApplicationRegion(region)
        .Build();

    Container container = client.GetContainer("cosmicworks", "products");

    string id = $"{Guid.NewGuid()}";
    string categoryId = $"{Guid.NewGuid()}";
    Product item = new (id, "Polished Bike Frame", categoryId);

    var response = await container.CreateItemAsync<Product>(item);

    Console.WriteLine($"Status Code:\t{response.StatusCode}");
    Console.WriteLine($"Charge (RU):\t{response.RequestCharge:0.00}");
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **22-sdk-multi-region** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```

1. ターミナル出力を確認してください。HTTP ステータス コードと要求課金（RU）がコンソールに表示されるはずです。

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- Azure Cosmos DB for NoSQL アカウントを作成した。
- SDK から Azure Cosmos DB for NoSQL アカウントへ接続した。
- SDK の書き込みリージョンを構成した。

### ラボは正常に完了しました