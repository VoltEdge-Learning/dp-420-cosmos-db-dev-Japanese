# Azure Cosmos DB for NoSQL SDK を使用して複数のポイント操作をバッチ処理する

## ラボ シナリオ

[TransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch] クラスと [TransactionalBatchResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatchresponse] クラスを組み合わせることで、複数の操作を 1 つの論理ステップとして構成および分解できます。これらのクラスを使用すると、複数の操作を実行するコードを記述し、それらがサーバー側で正常に完了したかどうかを判定できます。

このラボでは、SDK を使用して、2 つの項目を 1 つの論理単位として作成しようとする二重項目操作を 2 パターン実行します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: Azure Cosmos DB for NoSQL アカウントを作成し、SDK プロジェクトを構成する。
- タスク 2: トランザクション バッチを作成する。
- タスク 3: エラーになるトランザクション バッチを作成する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab7.png)

## 開発環境を準備する

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

## タスク 1: Azure Cosmos DB for NoSQL アカウントを作成し、SDK プロジェクトを構成する

このタスクでは、Azure Cosmos DB SQL アカウントをプロビジョニングし、基本設定を構成したうえで、今後の開発に必要な接続情報を取得します。

1. Azure Portal ページに戻り、ポータル上部の Search resources, services and docs (G+/) ボックスに **Azure Cosmos DB (1)** と入力し、services の下にある **Azure Cosmos DB (2)** を選択してください。

   ![06](media/New-image1.png)
   
1. **Azure Cosmos DB for NoSQL** の下で **+ Create (1)** を選択してください。

    ![06](media/New-image2.png)

    - **Create (2)** をクリックして **Azure Cosmos DB for NoSQL** アカウントを作成してください。

      ![06](media/New-image3.png)
   
1. 次の設定を指定し、その他の設定は既定値のままにして **Review + create (8)** を選択してください。

    | **Setting** | **Value** |
    | ---------------- |---------------------------------- |
    | **Workload Type** | *Development/Testing* **(1)** |
    | **Subscription** | *Your existing Azure subscription* **(2)** |
    | **Resource group** | *Select an existing Cosmosdb-<inject key="DeploymentID" enableCopy="false"/>* **(3)** |
    | **Account Name** | *sql-<inject key="DeploymentID" enableCopy="false"/>* **(4)** |
    | **Location** | *Choose any available region* **(5)** |
    | **Capacity mode** | *Provisioned throughput* **(6)** |
    | **Apply Free Tier Discount** | *Do Not Apply* **(7)** |

    ![06](media/c4.png)
    ![06](media/c5.png)

1. 検証が完了したら **Create** をクリックしてください。

1. このタスクを続行する前に、デプロイが完了するまで待機してください。

1. **Go to resources** を選択してください。

    ![06](media/New-image6.png)

1. 新しく作成した **Azure Cosmos DB** アカウントで、**Settings** の下にある **Keys** ペインに移動してください。

    ![06](media/New-image7.png)

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    - **URI (1)** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    - **eye** アイコン **(2)** をクリックしてください。**PRIMARY KEY (3)** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

      ![06](media/New-image9.png)

1. **Visual Studio Code** に戻ってください。

1. **Visual Studio Code** の **Explorer** ペインで **07-sdk-batch (1)** フォルダーを展開し、**script.cs (2)** コード ファイルを選択してください。

    ![06](media/c6.png)

1. **07-sdk-batch** フォルダー内の **script.cs** コード ファイルを開いてください。

    >**Note**: **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** ライブラリは NuGet からすでに事前にインポートされています。

1. **endpoint** という名前の **string** 変数を見つけてください。前のラボで作成した Azure Cosmos DB アカウントの **endpoint** 値を設定してください。

    ```
    string endpoint = "<cosmos-endpoint>";
    ```
    ![06](media/New-image64.png)

   >**Note**: たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の **string** 変数を見つけてください。前のラボで作成した Azure Cosmos DB アカウントの **key** 値を設定してください。

    ```
    string key = "<cosmos-key>";
    ```

    ![06](media/New-image65.png)

    >**Note**: たとえば key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. **script.cs** コード ファイルを **Save** してください。

1. **07-sdk-batch** フォルダーのコンテキスト メニューを開き、右クリック **(1)** してから **Open in Integrated Terminal (2)** を選択し、新しいターミナルを開いてください。

    ![06](media/c7.png)

     >**Note**: このコマンドでは、開始ディレクトリがすでに **07-sdk-batch** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加してください。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドしてください。

    ```
    dotnet build
    ```

    ![06](media/c8.png)

1. 統合ターミナルを閉じてください。

    > ラボ完了おめでとうございます。次は検証です。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、ラボは正常に検証されています。
    > - 表示されない場合は、エラーメッセージをよく確認し、ラボ ガイドの手順に従って再試行してください。
    > - サポートが必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

<validation step="9388db32-62bd-416d-a11e-beba00d5bd19" />

### タスク 2: トランザクション バッチを作成する

このタスクでは、Azure Cosmos DB でトランザクション バッチを作成し、2 つの製品（"Worn Saddle" と "Rusty Handlebar"）を同じパーティションに挿入します。このバッチにより、2 つの項目はアトミックに挿入され、両方が成功するか、どちらも成功しないかのいずれかになります。バッチを作成して実行した後、ステータス コードを確認して結果を検証します。

まず、架空の 2 製品を作成するシンプルなトランザクション バッチを作成します。このバッチでは、同じ "used accessories" カテゴリ識別子を持つ worn saddle と rusty handlebar をコンテナーに挿入します。両方の項目は同じ論理パーティション キーを持つため、バッチ操作は成功します。

1. **script.cs** コード ファイルのエディター タブに戻ってください。既存コードの下に、次のコードを 1 つずつ追加してください。

1. 一意識別子 **0120**、名前 **Worn Saddle**、カテゴリ識別子 **9603ca6c-9e28-4a02-9194-51cdb7fea816** を持つ **saddle** という名前の **Product** 変数を作成してください。

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 一意識別子 **012A**、名前 **Rusty Handlebar**、カテゴリ識別子 **9603ca6c-9e28-4a02-9194-51cdb7fea816** を持つ **handlebar** という名前の **Product** 変数を作成してください。

    ```
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. **9603ca6c-9e28-4a02-9194-51cdb7fea816** をコンストラクター パラメーターとして渡し、**partitionKey** という名前の **PartitionKey** 型の変数を作成してください。

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. **container** 変数の [CreateTransactionalBatch][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createtransactionalbatch] メソッドを呼び出し、メソッド パラメーターとして **partitionkey** 変数を渡してください。さらに fluent 構文で [CreateItem<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.transactionalbatch.createitem] ジェネリック メソッドを呼び出し、個別操作で作成する項目として **saddle** と **handlebar** 変数を渡して、結果を **batch** という名前の **TransactionalBatch** 型変数に格納してください。

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);
    ```

1. using ステートメント内で、**batch** 変数の **ExecuteAsync** メソッドを非同期で呼び出し、結果を **response** という名前の **TransactionalBatchResponse** 型変数に格納してください。

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. 静的 **Console.WriteLine** メソッドを呼び出して、**response** 変数の **StatusCode** プロパティ値を出力してください。

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 完了後、コード ファイルに次の内容が含まれていることを確認してください。

    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";
    
    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");

    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");

    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(saddle)
        .CreateItem<Product>(handlebar);

    using TransactionalBatchResponse response = await batch.ExecuteAsync();

    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

    ![06](media/c9.png)

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **07-sdk-batch** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```

1. ターミナル出力を確認してください。ステータス コードは HTTP 200 の **OK** になるはずです。

    ![06](media/c10.png)

1. これで、アプリケーション機能を監視できるようになっているはずです。

### タスク 3: エラーになるトランザクション バッチを作成する

このタスクでは、異なるパーティション キーを持つ 2 項目（"Flickering Strobe Light" と "New Helmet"）を挿入しようとして、意図的にエラーを発生させます。Cosmos DB では同じバッチ内の項目が同一パーティション キーを共有する必要があるため、このトランザクションはエラー（HTTP 400 Bad Request）になります。

次に、意図的に失敗するトランザクション バッチを作成します。このバッチでは、異なる論理パーティション キーを持つ 2 項目を挿入しようとします。"used accessories" カテゴリに flickering strobe light を、"pristine accessories" カテゴリに new helmet を作成します。定義上、これは不正な要求となり、このトランザクション実行時にエラーが返されます。

1. **script.cs** コード ファイルのエディター タブに戻ってください。

1. 次のコード行を削除してください。

    ```
    Product saddle = new("0120", "Worn Saddle", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product handlebar = new("012A", "Rusty Handlebar", "9603ca6c-9e28-4a02-9194-51cdb7fea816");

    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");

    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
         .CreateItem<Product>(saddle)
         .CreateItem<Product>(handlebar);

    using TransactionalBatchResponse response = await batch.ExecuteAsync();

    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 一意識別子 **012B**、名前 **Flickering Strobe Light**、カテゴリ識別子 **9603ca6c-9e28-4a02-9194-51cdb7fea816** を持つ **light** という名前の **Product** 変数を作成してください。

    ```
    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. 一意識別子 **012C**、名前 **New Helmet**、カテゴリ識別子 **0feee2e4-687a-4d69-b64e-be36afc33e74** を持つ **helmet** という名前の **Product** 変数を作成してください。

    ```
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");
    ```

1. **9603ca6c-9e28-4a02-9194-51cdb7fea816** をコンストラクター パラメーターとして渡し、**partitionKey** という名前の **PartitionKey** 型の変数を作成してください。

    ```
    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");
    ```

1. **container** 変数の **CreateTransactionalBatch** メソッドを呼び出し、メソッド パラメーターとして **partitionkey** 変数を渡してください。さらに fluent 構文で **CreateItem<>** ジェネリック メソッドを呼び出し、個別操作で作成する項目として **light** と **helmet** 変数を渡して、結果を **batch** という名前の **TransactionalBatch** 型変数に格納してください。

    ```
    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
         .CreateItem<Product>(light)
         .CreateItem<Product>(helmet);
    ```

1. using ステートメント内で、**batch** 変数の **ExecuteAsync** メソッドを非同期で呼び出し、結果を **response** という名前の **TransactionalBatchResponse** 型変数に格納してください。

    ```
    using TransactionalBatchResponse response = await batch.ExecuteAsync();
    ```

1. 静的 **Console.WriteLine** メソッドを呼び出して、**response** 変数の **StatusCode** プロパティ値を出力してください。

    ```
    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. 完了後、コード ファイルに次の内容が含まれていることを確認してください。

    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product light = new("012B", "Flickering Strobe Light", "9603ca6c-9e28-4a02-9194-51cdb7fea816");
    Product helmet = new("012C", "New Helmet", "0feee2e4-687a-4d69-b64e-be36afc33e74");

    PartitionKey partitionKey = new ("9603ca6c-9e28-4a02-9194-51cdb7fea816");

    TransactionalBatch batch = container.CreateTransactionalBatch(partitionKey)
        .CreateItem<Product>(light)
        .CreateItem<Product>(helmet);

    using TransactionalBatchResponse response = await batch.ExecuteAsync();

    Console.WriteLine($"Status:\t{response.StatusCode}");
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **07-sdk-batch** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```

1. ターミナル出力を確認してください。ステータス コードは HTTP 400 **Bad Request** または 409 **Conflict** のいずれかになるはずです。これは、トランザクション内のすべての項目がトランザクション バッチと同じパーティション キー値を共有していないためです。

    ![06](media/c11.png)

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

### まとめ

このラボでは、Azure Cosmos DB for NoSQL SDK を使用して、複数の操作を 1 つのトランザクション単位としてバッチ処理する実践を行いました。TransactionalBatch と TransactionalBatchResponse クラスを活用することで、複数のポイント操作をグループ化し、すべての操作が同時に成功または失敗するアトミック性を確保する方法を学びました。また、Cosmos DB ではトランザクション内の項目が同じパーティション キーを共有する必要があるため、バッチ操作におけるパーティション キーの重要性を理解しました。

### ラボは正常に完了しました