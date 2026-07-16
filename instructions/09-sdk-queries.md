# Azure Cosmos DB for NoSQL SDK を使用してクエリを実行する

## ラボ シナリオ

Azure Cosmos DB for NoSQL 向け .NET SDK の最新バージョンでは、C# の最新ベスト プラクティスと言語機能を活用して、コンテナーのクエリ実行や結果セットの非同期反復処理をこれまで以上に簡単に行えます。

このライブラリには、[https://learn.microsoft.com/en-us/dotnet/api/microsoft.azure.cosmos.feediterator?view=azure-dotnet] を使用して Azure Cosmos DB のクエリを簡単に行うための特別な機能があります。

このラボでは、非同期ストリームを使用して Azure Cosmos DB for NoSQL から返される大規模な結果セットを反復処理します。.NET SDK を使用してクエリを実行し、結果を反復処理します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Azure Cosmos DB NoSQL API アカウントを作成する。
- タスク 3: Azure Cosmos DB NoSQL API アカウントにデータを投入する。
- タスク 4: SDK を使用して SQL クエリ結果を反復処理する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab9.png)

### タスク 1: 開発環境を準備する

このタスクでは、Visual Studio Code をセットアップして、Azure Cosmos DB を操作するための開発環境を準備します。

1. デスクトップから **Visual Studio Code** を起動してください。

     ![Visual Studio Code Icon](./media/vscode1.jpg)

2. 左側のパネルで **Extensions (1)** を選択してください。**C# (2)** を検索し、**Install (3)** を選択して拡張機能をインストールしてください。インストールが完了するまで待機してください。

    ![06](media/New-image50.png)

3. 画面左上の **File (1)** を選択し、メニューから **Open Folder (2)** を選択してください。

    ![06](media/c3.png)

4. **C:\AllFiles (1)** に移動し、**dp-420-cosmos-db-dev-main (2)** を選択して **Select Folder (3)** をクリックしてください。

    ![06](media/c2.png)

5. **When Do you trust the author of the files in this folder** と表示された場合は、**Yes, I trust the authors** をクリックしてください。

### タスク 2: Azure Cosmos DB NoSQL API アカウントを作成する

このタスクでは、NoSQL API を使用して Azure Cosmos DB アカウントを作成します。アカウントのプロビジョニング後、エンドポイント (URI) とプライマリ キーを含む接続情報を取得します。これらの資格情報を使用して、Azure SDK または任意の SDK で Cosmos DB アカウントに接続できます。

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングする際は、サポート対象 API（例: **Mongo API** または **SQL API**）を選択します。Azure Cosmos DB SQL API アカウントのプロビジョニング完了後、エンドポイントとキーを取得し、Azure SDK for .NET または任意の SDK で Azure Cosmos DB SQL API アカウントに接続できます。

1. Azure Portal ページに戻り、ポータル上部の Search resources, services and docs (G+/) ボックスに **Azure Cosmos DB (1)** と入力し、services の下にある **Azure Cosmos DB (2)** を選択してください。

   ![06](media/New-image1.png)

1. **Azure Cosmos DB for NoSQL** の下で **+ Create (1)** を選択してください。

    ![06](media/New-image2.png)

    - **Create (2)** をクリックして **Azure Cosmos DB for NoSQL** アカウントを作成してください。

      ![06](media/New-image3.png)

1. 次の設定を指定し、その他の設定は既定値のままにして **Next: Global Distribution (9)** を選択してください。

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

     ![06](media/c12.png)

     ![06](media/c13.png)

1. Global Distribution ページで **Next: Networking** をクリックしてください。**Connectivity method** で **All networks (1)** を選択し、**Review + Create (2)** をクリックしてください。

    ![06](media/c14.png)

1. 検証完了後、**Create** をクリックしてください。

1. このタスクを続行する前に、デプロイが完了するまで待機してください。

1. **Go to resources** を選択してください。

    ![06](media/New-image6.png

1. 新しく作成した **Azure Cosmos DB** アカウントで、**Settings** の下にある **Keys (1)** ペインに移動してください。このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    - **URI (2)** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

        ![06](media/c18.png)

    - **Primary Connection String** フィールドを確認してください。**eye** アイコン **(1)** をクリックし、この演習の後半で使用する **connection string** 値 **(2)** をコピーしてください。

      - PRIMARY KEY の横にある **eye** アイコン **(3)** をクリックしてください。**PRIMARY KEY (4)** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

        ![06](media/c19.png)

1. **Visual Studio Code** に戻ってください。

    > ラボ完了おめでとうございます。次は検証です。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、ラボは正常に検証されています。
    > - 表示されない場合は、エラーメッセージをよく確認し、ラボ ガイドの手順に従って再試行してください。
    > - サポートが必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

    <validation step="1365501c-f12d-434e-9c4f-6262ecb20955" />

### タスク 3: Azure Cosmos DB NoSQL API アカウントにデータを投入する

このタスクでは、cosmicworks コマンドライン ツールをインストールし、アカウントの endpoint と key を含むコマンドを実行して、Azure Cosmos DB NoSQL API アカウントにサンプル製品データを投入します。

[cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールは、任意の Azure Cosmos DB SQL API アカウントにサンプル データをデプロイします。このツールはオープンソースで、NuGet から利用できます。このツールを Azure Cloud Shell にインストールし、データベースの初期データ投入に使用します。

1. **Visual Studio Code** で、**... (ellipses) (1)** > **Terminal (2)** > **New Terminal (3)** を選択して **Terminal** メニューを開き、既存インスタンス内で新しいターミナルを開いてください。

    ![06](media/New-image36.png)

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールを、マシン全体で利用できるようにインストールしてください。

    ```
    dotnet tool install cosmicworks --global --version 2.*
    ```

    ![06](media/c15.png)

     >**Note**: このコマンドの完了には数分かかる場合があります。過去にこのツールの最新バージョンをすでにインストールしている場合、このコマンドは警告メッセージ（*Tool 'cosmicworks' is already installed）を出力します。

1. インストール完了後、下記コマンドを実行するために **Visual Studio Code** を一度閉じて再度開いてください。

1. 次のコマンドを実行して、以下のコマンドライン オプションで Azure Cosmos DB アカウントにデータを投入してください。

    | **Option**       | **Value** |
    | ---------------- | ----------|
    | **CONNECTION STRING**   | *このラボで先ほどコピーした Primary Connection String の値* |

    ```
    cosmicworks --connection-string "<CONNECTION_STRING>" --disable-hierarchical-partition-keys
    ```

    > **Note:** 上記コマンドの実行時にエラーが出る場合は、Visual Studio Code を**閉じて**から**再度開き**、上記コマンドを実行してください。

1. **cosmicworks** コマンドが、データベース、コンテナー、項目をアカウントに投入し終えるまで待機してください。

   ![06](media/c16.png)

    >**Note**: エラーが発生する場合は、Visual Studio Code を閉じて再度開き、もう一度コマンドを実行してください。

1. 統合ターミナルを閉じてください。

### タスク 4: SDK を使用して NoSQL クエリの結果を反復処理する

このタスクでは、C# スクリプトを修正して Cosmos DB に接続し、全製品を取得する SQL クエリを実行し、非同期ループを使って各製品の ID、名前、価格を表示します。このタスクにより、Azure Cosmos DB SDK を使った NoSQL 環境でのクエリおよびデータ処理の理解を強化できます。

ここでは、非同期ストリームを使用して Azure Cosmos DB から返るページ分割結果に対する、わかりやすい for-each ループを作成します。内部的には SDK が feed iterator を管理し、後続リクエストを正しく実行します。

1. **Visual Studio Code** の **Explorer** ペインで、**09-execute-query-sdk (1)** フォルダーに移動してください。

    - **product.cs (2)** コード ファイルを開いてください。
    - **Product** クラスと対応するプロパティを確認してください。特に、このラボでは **id**、**name**、**price** プロパティを使用します。

      ![06](media/c17.png)

1. **Visual Studio Code** の **Explorer** ペインに戻り、**script.cs** コード ファイルを開いてください。

1. 既存の **endpoint** という名前の変数を更新し、先ほど作成した Azure Cosmos DB アカウントの **endpoint** を値として設定してください。

    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    >**Note**: たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. 既存の **key** という名前の変数を更新し、先ほど作成した Azure Cosmos DB アカウントの **key** を値として設定してください。

    ```
    string key = "<cosmos-key>";
    ```

    ![06](media/c-21.png)

     >**Note**: たとえば key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. `12` 行目の `CosmosContainer container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId");` を削除してください。

    ![06](media/c22.png)

1. 以下のステートメントを追加してください。

    ```
    CosmosContainer container = database.GetContainer("products");
    ```

    ![06](media/c23.png)

1. スクリプト末尾に、型 *string* の **sql** という新しい変数を作成し、値を **SELECT * FROM products p** にする次のコマンドを追加してください。

    ```
    string sql = "SELECT * FROM products p";
    ```

1. **sql** 変数をコンストラクターのパラメーターとして渡し、型 [QueryDefinition][docs.microsoft.com/dotnet/api/azure.cosmos.querydefinition] の新しい変数を作成してください。

    ```
    QueryDefinition query = new (sql);
    ```

1. [CosmosContainer][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer] クラスのジェネリック [GetItemQueryIterator][docs.microsoft.com/dotnet/api/azure.cosmos.cosmoscontainer.getitemqueryiterator] メソッドを **query** 変数をパラメーターとして呼び出し、結果を非同期反復処理する新しい **await foreach** ループを作成してください。結果の各要素は、型 **Product** のインスタンスを表す **product** 変数で受け取ってください。

    ```
    await foreach (Product product in container.GetItemQueryIterator<Product>(query))
    {
    }
    ```

1. **await foreach** ループ内で、組み込みの静的 **Console.WriteLine** メソッドを使用し、**product** 変数の **id**、**name**、**price** プロパティを整形して出力してください。

    ```
    Console.WriteLine($"[{product.id}]\t{product.name,35}\t{product.price,15:C}");
    ```

1. 完了後、コード ファイルに次の内容が含まれていることを確認してください。

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

    ![06](media/c24.png)

1. **Ctrl+S** を押して script.cs ファイルを **Save** してください。

1. **Visual Studio Code** で **09-execute-query-sdk** フォルダーのコンテキスト メニューを開き、フォルダーを右クリック **(1)** してから **Open in Integrated Terminal (2)** を選択し、新しいターミナルを開いてください。

    ![06](media/c25.png)

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```

1. スクリプトにより、コンテナー内のすべての製品が出力されます。

    ![06](media/c26.png)

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。


### まとめ

このラボでは、Visual Studio Code をセットアップし、Azure Cosmos DB NoSQL API アカウントを作成しました。次に cosmicworks ツールを使って製品データを投入した後、C# スクリプトを修正してデータベースを非同期クエリしました。Cosmos DB SDK を使用して SQL クエリを実行し、非同期ループで結果を反復処理して製品情報を効率よく表示しました。このラボを通して、.NET SDK と C# を使った Azure Cosmos DB のクエリおよびデータ処理を実践的に学習しました。


### ラボは正常に完了しました