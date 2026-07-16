# SDK を使用してユーザー定義関数を実装し利用する

## ラボ シナリオ

Azure Cosmos DB SQL API 用 .NET SDK は、サーバー側プログラミング構成要素をコンテナーから直接管理および呼び出すために使用できます。新しいコンテナーを準備する際は、Data Explorer で手動作業を行う代わりに、.NET SDK を使用して UDF をコンテナーへ直接公開する方法が適しています。

このラボでは、.NET SDK を使用して新しい UDF を作成し、その後 Data Explorer を使用して UDF が正しく動作することを検証します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する。
- タスク 3: Azure Cosmos DB SQL API アカウントにデータを投入する。
- タスク 4: .NET SDK を使用してユーザー定義関数（UDF）を作成する。
- タスク 5: Data Explorer を使用して UDF をテストする。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab32.png)

### タスク 1: 開発環境を準備する

このタスクでは、Visual Studio Code をセットアップして、Azure Cosmos DB の作業に向けた開発環境を準備します。

1. Visual Studio Code を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

   ![Visual Studio Code Icon](./media/vscode1.jpg)

2. 左側ペインの **Extension (1)** アイコンを選択してください。検索バーに **C# (2)** を入力し、表示された **extension (3)** を選択して、最後に **Install (4)** を選択してください。

    ![](media/visualstudioo.png)

3. 画面左上の **file** オプションを選択し、メニューから **Open Folder** を選択して **C:\AllFiles** に移動してください。

4. **dp-420-cosmos-db-dev-main** フォルダーを選択し、**Select Folder** をクリックしてください。

   ![](media/lab12-1.png)

    >**Note:** **Do you trust the authors of the files in this folder?** ポップアップで **Yes, I trust authors** を選択してください。

    ![06](media/lab12-2.png)


### タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する

このタスクでは、Azure Cosmos DB SQL アカウントをプロビジョニングし、重要な設定を構成して、今後の開発に必要な接続情報を取得します。

Azure Cosmos DB は複数の API をサポートするクラウドベースの NoSQL データベース サービスです。Azure Cosmos DB アカウントを初めてプロビジョニングする際は、アカウントでサポートする API（例: **Mongo API** または **NoSQL API**）を選択します。Azure Cosmos DB for NoSQL アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の SDK を使用して Azure Cosmos DB for NoSQL アカウントへ接続できます。

1. Azure Portal ページに戻ってください。ポータル上部の Search resources, services and docs (G+/) ボックスに **Azure Cosmos DB** と入力し、services の **Azure Cosmos DB** を選択してください。

   ![06](media/New-image1.png)
   
1. **Azure Cosmos DB for NoSQL** の下にある **+ Create** を選択し、**Create** をクリックして **Azure Cosmos DB for NoSQL** アカウントを作成してください。

    ![06](media/New-image2.png)

    ![06](media/New-image3.png)

1. 次の設定を指定し、それ以外の設定は既定値のままにして、**Review + create** を選択してください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Subscription** | *Your existing Azure subscription* |
    | **Resource group** | **Cosmosdb-<inject key="DeploymentID" enableCopy="false"/>** |
    | **Account Name** | *Enter a globally unique name* |
    | **Location** | *Choose any available region* |
    | **Capacity mode** | *Provisioned throughput* |
    | **Apply Free Tier Discount** | *Do Not Apply* |

1. 検証が Success になったら、**Create** をクリックしてください。

1. このタスクを続行する前に、デプロイ タスクが完了するまで待機してください。

1. **Go to resources** を選択してください。新しく作成した **Azure Cosmos DB** アカウントの **Settings** から **Keys** ペインに移動してください。

    ![06](media/New-image6.png)

    ![06](media/New-image7.png)

1. このペインには、SDK からアカウントへ接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

        ![06](media/New-image9.png)

1. ブラウザー ウィンドウを閉じずに、**Visual Studio Code** を開いてください。

    > ラボの完了おめでとうございます。ここで検証を行います。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、ラボの検証は完了です。
    > - 失敗した場合は、エラー メッセージを注意深く読み、ラボ ガイドの手順に沿って再試行してください。
    > - サポートが必要な場合は、cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。
    
    <validation step="74eda0bf-4b7b-47d2-9d83-0bb7e6bc8ffa" />

### タスク 3: Azure Cosmos DB SQL API アカウントにデータを投入する

このタスクでは、cosmicworks コマンドライン ツールを使用して、Azure Cosmos DB SQL API アカウントにサンプル データを展開します。このツールは NuGet 経由でインストールされ、製品データなどの事前定義データセットを使ってデータベースへ迅速にデータ投入できるため、テストと開発を容易にします。

[cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールは、任意の Azure Cosmos DB SQL API アカウントにサンプル データを展開します。このツールはオープンソースで、NuGet から利用できます。Azure Cloud Shell にインストールし、データベースへのデータ投入に使用します。

1. **Visual Studio Code** で **... (ellipses) (1)** > **Terminal (2)** > **New Terminal (3)** を選択して **Terminal** メニューを開き、既存インスタンスで新しいターミナルを開いてください。

    ![06](media/terminal.png)

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールを、マシンでグローバルに利用できるようにインストールしてください。

    ```
    dotnet tool install --global cosmicworks
    ```

    >**Note:** このコマンドの完了には数分かかる場合があります。すでにこのツールの最新バージョンをインストール済みの場合、このコマンドは警告メッセージ（*Tool 'cosmicworks' is already installed'*）を出力します。

1. インストール完了後、次のコマンドを実行するために **Visual Studio Code** を一度閉じて再度開いてください。

1. 次のコマンドライン オプションを使用して cosmicworks を実行し、Azure Cosmos DB アカウントにデータを投入してください。

    | **Option** | **Value** |
    | --- | --- |
    | **--endpoint** | *The endpoint value you copied earlier in this lab* |
    | **--key** | *The key value you coped earlier in this lab* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    >**For example:** endpoint が **https&shy;://dp420.documents.azure.com:443/** で key が **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。
    > ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``
    
    >**Note**: **What is your connection string** と表示された場合は、Azure Cosmos DB に戻り、左側ナビゲーション ペインで **Key** を選択して **primary connection string** をコピーし、Visual Studio 上で右クリックして貼り付けてください。

     ![06](media/New-image127.png)
    
1. **cosmicworks** コマンドが、アカウントへのデータベース、コンテナー、および項目の投入を完了するまで待機してください。
   
    >**Note**: エラーが発生した場合は Visual Studio Code を閉じて再度開き、コマンドをもう一度実行してください。

1. 統合ターミナルを閉じてください。

    > ラボの完了おめでとうございます。ここで検証を行います。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、ラボの検証は完了です。
    > - 失敗した場合は、エラー メッセージを注意深く読み、ラボ ガイドの手順に沿って再試行してください。
    > - サポートが必要な場合は、cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。
    
    <validation step="dd92f2ca-c14f-4181-8374-d60868d94589" />

### タスク 4: .NET SDK を使用してユーザー定義関数（UDF）を作成する

このタスクでは、Azure Cosmos DB .NET SDK を使用して、税込み製品価格を計算する UDF を作成します。このタスクでは C# スクリプトを記述して UDF を定義し、Azure Cosmos DB SQL API コンテナーにデプロイして、製品価格に対する税計算クエリを実行できるようにします。

.NET SDK の [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] クラスには [Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] プロパティがあり、SDK から直接、ストアド プロシージャ、UDF、トリガーに対する CRUD 操作を実行できます。このプロパティを使用して新しい UDF を作成し、その UDF を Azure Cosmos DB SQL API コンテナーにプッシュします。このラボで SDK を使って作成する UDF は、製品価格に税率を適用した価格を計算し、税込み価格を使った SQL クエリを製品に対して実行できるようにします。

1. **Visual Studio Code** の **Explorer** ペインで **33-create-use-udf-sdk** フォルダーに移動してください。

1. **script.cs** コード ファイルを開いてください。

1. [Microsoft.Azure.Cosmos.Scripts][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts] 名前空間の using ブロックを追加してください。

    ```
    using Microsoft.Azure.Cosmos.Scripts;
    ```

1. 既存の **endpoint** 変数を更新し、先ほど作成した Azure Cosmos DB アカウントの **endpoint** 値を設定してください。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > **For example:** endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. 既存の **key** 変数を更新し、先ほど作成した Azure Cosmos DB アカウントの **key** 値を設定してください。

    ```
    string key = "<cosmos-key>";
    ```

    > **For example:** key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. 既定の空コンストラクターを使用し、[UserDefinedFunctionProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties] 型の **props** という新しい変数を作成してください。

    ```
    UserDefinedFunctionProperties props = new ();
    ```

1. **props** 変数の [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.id] プロパティを **tax** に設定してください。

    ```
    props.Id = "tax";
    ```

1. **props** 変数の [Body][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionproperties.body] プロパティを **props.Body = "function tax(i) { return i * 1.25; }";** に設定してください。

    ```
    props.Body = "function tax(i) { return i * 1.25; }";
    ```

1. **container** 変数の [Scripts.CreateUserDefinedFunctionAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.scripts] メソッドを非同期で呼び出し、**props** 変数をパラメーターとして渡してください。結果は [UserDefinedFunctionResponse][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse] 型の **udf** 変数に保存してください。

    ```
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用し、UserDefinedFunctionResponse クラスの [Resource.Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.scripts.userdefinedfunctionresponse.resource] プロパティを **Created UDF** という見出し付きで出力してください。

    ```
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. 完了後、コード ファイルは次の内容になります。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    using Microsoft.Azure.Cosmos.Scripts;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");

    Container container = await database.CreateContainerIfNotExistsAsync("products", "/category/name");

    UserDefinedFunctionProperties props = new ();
    props.Id = "tax";
    props.Body = "function tax(i) { return i * 1.25; }";
    
    UserDefinedFunctionResponse udf = await container.Scripts.CreateUserDefinedFunctionAsync(props);
    
    Console.WriteLine($"Created UDF [{udf.Resource?.Id}]");
    ```

1. **script.cs** ファイルを **Save** してください。

1. **Visual Studio Code** で **33-create-use-udf-sdk** フォルダーを右クリックし、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドし実行してください。

    ```
    dotnet run
    ```

1. スクリプトは新しく作成された UDF の名前を出力します。

    ```
    Created UDF [tax]
    ```

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

    > ラボの完了おめでとうございます。ここで検証を行います。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、ラボの検証は完了です。
    > - 失敗した場合は、エラー メッセージを注意深く読み、ラボ ガイドの手順に沿って再試行してください。
    > - サポートが必要な場合は、cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。
    
    <validation step="01aa9434-9775-4fd0-baa1-1dcfc60bdba6" />

### タスク 5: Data Explorer を使用して UDF をテストする

このタスクでは、Data Explorer で SQL クエリを実行し、先ほど Azure Cosmos DB で作成したユーザー定義関数（UDF）を検証します。

1. Web ブラウザーに戻ってください。

1. **Azure Cosmos DB** アカウント リソース内で **Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**NOSQL API** ナビゲーション ツリー内に新しい **products** コンテナー ノードがあることを確認してください。

1. **NOSQL API** ナビゲーション ツリー内の **products** コンテナー ノード（**...**）を選択し、**New SQL Query** を選択してください。

1. クエリ タブで **Execute Query** を選択し、フィルターなしで全アイテムを取得する標準クエリを表示してください。

1. エディター領域の内容を削除してください。

1. 2 つの価格値を投影してすべてのドキュメントを返す新しい SQL クエリを作成してください。1 つ目はコンテナー内の生の価格値、2 つ目は UDF によって計算された価格値です。

    ```
    SELECT p.id, p.price, udf.tax(p.price) AS priceWithTax FROM products p
    ```

1. **Execute Query** を選択してください。

1. ドキュメントを確認し、**price** フィールドと **priceWithTax** フィールドを比較してください。

    >**Note:** **priceWithTax** フィールドは **price** フィールドより 25% 大きい値になるはずです。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

### まとめ

このラボでは、.NET SDK を使用して Azure Cosmos DB のユーザー定義関数（UDF）を実装およびテストしました。主な目的は、開発環境の準備、Azure Cosmos DB for NoSQL アカウントの作成と構成、データ投入、および税込み製品価格を計算する UDF の開発でした。

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- Azure Cosmos DB for NoSQL アカウントを作成した。
- Azure Cosmos DB SQL API アカウントにデータを投入した。
- .NET SDK を使用してユーザー定義関数（UDF）を作成した。
- Data Explorer を使用して UDF をテストした。

### ラボは正常に完了しました