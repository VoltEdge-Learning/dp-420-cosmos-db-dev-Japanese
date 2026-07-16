# SDK を使用して Azure Cosmos DB for NoSQL に接続する

### 推定所要時間: 60 分

## ラボ シナリオ

Azure SDK for .NET は、Azure サービスとやり取りするために一貫した開発体験を提供するライブラリ群です。.NET Standard 2.0 をベースとしており、.NET Framework 4.6.1 以降、.NET Core 2.1 以降、.NET 5 以降と互換性があります。

このラボでは、Azure SDK for .NET を使用して Azure Cosmos DB SQL API アカウントに接続します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Azure Cosmos DB SQL API アカウントを作成する。
- タスク 3: NuGet で Microsoft.Azure.Cosmos ライブラリを確認する。
- タスク 4: Microsoft.Azure.Cosmos ライブラリを .NET プロジェクトにインポートする。
- タスク 5: Microsoft.Azure.Cosmos ライブラリを使用する。
- タスク 6: スクリプトをテストする。

### タスク 1: 開発環境を準備する

このタスクでは、Visual Studio Code をセットアップして、Azure Cosmos DB を操作するための開発環境を準備します。

1. デスクトップから Visual Studio Code を開いてください。

     ![Visual Studio Code Icon](./media/vscode1.jpg)

1. 左側パネルの **Extensions (1)** ブレードを選択してください。**C# (2)** で検索し、**Install (3)** を選択して拡張機能をインストールしてください。

    ![06](media/New-image50.png)

1. 画面左上の **file (1)** を選択し、メニューから **Open Folder (2)** を選択してください。**C:\AllFiles\dp-420-cosmos-db-dev** に移動してください。

     ![06](media/New-image51.png)

1. **C:\AllFiles\dp-420-cosmos-db-dev** に移動し、**dp-420-cosmos-db-dev** を選択して **Select Folder** をクリックしてください。

    ![06](media/New-image54.png)

1. **Do you trust the author of the files in this folder** と表示された場合は、**Yes, I trust the authors** をクリックしてください。

   ![06](media/DB24.png)

### タスク 2: Azure Cosmos DB SQL API アカウントを作成する

このタスクでは、Azure Cosmos DB SQL API アカウントをプロビジョニングし、主要な設定を構成して、今後の開発に必要な接続情報を取得します。

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングするときは、アカウントでサポートする API（例: **Mongo API** または **NoSQL API**）を選択します。Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、それらを使用して Azure SDK for .NET または任意の SDK から Azure Cosmos DB NoSQL API アカウントに接続できます。

1. Azure Portal ページで、ポータル上部の Search resources, services and docs (G+/) ボックスに **Azure Cosmos DB (1)** と入力し、services の **Azure Cosmos DB (2)** を選択してください。

   ![06](media/New-image1.png)
   
1. **Azure Cosmos DB for NoSQL** の **+ Create (1)** を選択し、**Create (2)** をクリックして **Azure Cosmos DB for NoSQL** アカウントを作成してください。

    ![06](media/New-image2.png)

    ![06](media/New-image3.png)

1. 次の設定を指定し、その他の設定は既定値のままにして **Review + create (10)** を選択してください。

    | **Setting**         | **Value** |
    | --------------------|--------------------------------------------------- |
    | **Workload Type**   | *Production* (1) |
    | **Subscription**    | *既存の Azure サブスクリプション* (2) |
    | **Resource group**  | *既存の Cosmosdb-<inject key="DeploymentID" enableCopy="false"/> を選択する* (3) |
    | **Account Name**    | *sql-<inject key="DeploymentID" enableCopy="false"/>* (4) |
    | **Location**        | *既定のリージョンを選択する* (5) |
    | **Capacity mode**   | *Provisioned throughput* (6) |
    | **Apply Free Tier Discount** | *Do Not Apply* (7) |
    | **Limit the total amount of throughput that can be provisioned on this account** | *Unchecked* (8) |

     ![06](media/DB25.png)

1. **Create** をクリックしてください。

    ![06](media/New-image5.png)

1. このタスクを続行する前に、デプロイが完了するまで待機してください。

1. デプロイ完了後、**Go to resources** を選択してください。

    ![06](media/New-image6.png)

1. **Azure Cosmos DB account** で、左メニューの **Settings (1)** を展開し、**Keys (2)** を選択してください。

    ![06](media/DB15.png)

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    1. **URI (1)** フィールドの値を記録してください。この演習の後半で **endpoint** 値として使用します。

    1. **PRIMARY KEY (2)** フィールドの値を記録してください。この演習の後半で **key** 値として使用します。

        ![06](media/New-image9.png)
       
    > ラボ完了おめでとうございます。次は検証です。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、ラボは正常に検証されています。
    > - 表示されない場合は、エラーメッセージをよく確認し、ラボ ガイドの手順に従って再試行してください。
    > - サポートが必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。
 
    <validation step="ade422fd-22ef-466a-80b1-bd33186d9b51" />
    
### タスク 3: NuGet で Microsoft.Azure.Cosmos ライブラリを確認する
このタスクでは、NuGet Web サイトで Microsoft.Azure.Cosmos ライブラリを確認します。これは .NET アプリケーションで Azure Cosmos DB を操作するために重要なライブラリです。NuGet のパッケージ マネージャーとしての機能を理解し、対象ライブラリを検索し、.NET プロジェクトにインポートするためのコマンドを確認します。このタスクは、後続の開発作業に向けて、ライブラリ パッケージへのアクセスと管理に慣れるための準備となります。

NuGet Web サイトには、.NET アプリケーションにインポート可能なパッケージの検索可能なインデックスがあります。**Microsoft.Azure.Cosmos** のようなプレリリース パッケージをインポートするには、NuGet Web サイトを使用して適切なバージョンとインポート コマンドを確認できます。

1. ブラウザーを開いて **nuget.org (1)** に移動し、利用可能な .NET パッケージ **(2)** を確認してください。

    ![06](media/DB26.png)

2. **NuGet** ページで **Packages (1)** を選択し、**Microsoft.Azure.Cosmos (2)** を検索し、**.NET Standard (3)** を展開して **netstandard2.0 (4)** を選択してください。

   ![06](media/DB27.png)

3. **.NET CLI** タブを選択して、このライブラリの最新バージョンを .NET プロジェクトにインポートするためのコマンドを確認してください。
   
      >**Note**: このコマンドを記録する必要はありません。この演習の後半では特定バージョンのライブラリを使用します。
     
4. Web ブラウザーのウィンドウまたはタブを閉じてください。

### タスク 4: Microsoft.Azure.Cosmos ライブラリを .NET プロジェクトにインポートする
このタスクでは、Visual Studio Code を開いてプロジェクト ディレクトリに移動します。次に統合ターミナルにアクセスし、Microsoft.Azure.Cosmos ライブラリを .NET プロジェクトにインポートするコマンドを実行します。このライブラリにより Azure Cosmos DB とやり取りできるようになります。

.NET CLI には、事前構成済みのパッケージ フィードからパッケージをインポートする [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] コマンドが含まれています。.NET のインストールでは、既定のパッケージ フィードとして NuGet が使用されます。
     
1. **Visual Studio Code** を開いてください。

   ![Visual Studio Code Icon](./media/vscode1.jpg)

1. **Visual Studio Code** で **04-sdk-connect (1)** フォルダーを右クリックし、**Open in Integrated Terminal (2)** を選択して新しいターミナルを開いてください。

     ![06](media/2.png)

     >**Note**: このコマンドでは、開始ディレクトリがすでに **04-sdk-connect** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加してください。

   ```
   dotnet add package Microsoft.Azure.Cosmos --version 3.*
   ```
1. 統合ターミナルを閉じてください。

### タスク 5: Microsoft.Azure.Cosmos ライブラリを使用する

このタスクでは、Microsoft.Azure.Cosmos ライブラリを使用して Azure Cosmos DB アカウントに接続します。Visual Studio Code で script.cs ファイルを開き、アカウントの endpoint と key の変数を定義し、CosmosClient インスタンスを作成します。次に、アカウント名とプライマリ リージョンをコンソールに表示してから、ファイルを保存します。

Azure SDK for .NET の Azure Cosmos DB ライブラリをインポートすると、[Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] 名前空間内のクラスをすぐに使用して Azure Cosmos DB SQL API アカウントに接続できます。[CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] クラスは、Azure Cosmos DB SQL API アカウントへの初期接続に使用する中心的なクラスです。

1. **Visual Studio Code** で、**04-sdk-connect (1)** フォルダー内の空の **script.cs (2)** コード ファイルを開いてください。

    ![06](media/DB28.png)

1. 組み込みの **System** と **System.Linq** 名前空間の using ブロックを追加してください。
   
   ```
   using System;
   using System.Linq;
   ```
1. [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] 名前空間の using ブロックを追加してください。
   
   ```
   using Microsoft.Azure.Cosmos;
   ```
1. **endpoint** という名前の **string** 変数を追加し、値を先ほど作成した Azure Cosmos DB アカウントの **endpoint** に設定してください。
   
   ```
   string endpoint = "<cosmos-endpoint>";
   ```
   >**Note**: たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** となります。

1. **key** という名前の **string** 変数を追加し、値を先ほど作成した Azure Cosmos DB アカウントの **key** に設定してください。
   
   ```
   string key = "<cosmos-key>";
   ```
   >**Note**: たとえば key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** となります。

1. コンストラクターで **endpoint** 変数と **key** 変数を使用して、[CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] 型の **client** という新しい変数を追加してください。
   
   ```
   CosmosClient client = new (endpoint, key);
   ```
1. **client** 変数の [ReadAccountAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync] メソッドを呼び出した非同期結果を使用して、[AccountProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties] 型の **account** という新しい変数を追加してください。
   
   ```
   AccountProperties account = await client.ReadAccountAsync();
   ```
1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、AccountProperties クラスの [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id] プロパティを **Account Name** ヘッダー付きで出力してください。

   ```
   Console.WriteLine($"Account Name:\t{account.Id}");
   ```
   
1. 組み込みの **Console.WriteLine** 静的メソッドを使用し、AccountProperties クラスの [WritableRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions] プロパティを参照して、最初の結果の [Name][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name] プロパティを **Primary Region** ヘッダー付きで出力してください。
    
      ```
      Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
      ```
1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
   
    ```
    using System;
    using System.Linq;
    
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    AccountProperties account = await client.ReadAccountAsync();

    Console.WriteLine($"Account Name:\t{account.Id}");
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```
1. **script.cs** コード ファイルを **Save** してください。

    ![06](media/DB29.png)

### タスク 6: スクリプトをテストする
このタスクでは、Visual Studio Code の統合ターミナルを開き、dotnet run コマンドでプロジェクトを実行してスクリプトをテストします。出力にはアカウント名と最初の書き込み可能リージョンが表示されます。

Azure Cosmos DB SQL API アカウントに接続する .NET コードが完成したので、スクリプトをテストできます。このスクリプトはアカウント名と最初の書き込み可能リージョン名を表示します。アカウント作成時に指定した location と同じ値が結果として表示されるはずです。

1. **Visual Studio Code** で **04-sdk-connect (1)** フォルダーを右クリックし、**Open in Integrated Terminal (2)** を選択して新しいターミナルを開いてください。

     ![06](media/2.png)

2. 次のコマンドで Newtonsoft.JSON パッケージをインストールしてください。

   ```
   dotnet add package Newtonsoft.Json --version 13.0.3
   ```

2. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドし、実行してください。

   ```
   dotnet run
   ```
3. スクリプトによって、アカウント名と最初の書き込み可能リージョンが出力されます。たとえばアカウント名が **sql-<inject key="DeploymentID" enableCopy="false"/>** で、最初の書き込み可能リージョンが **West US 3** の場合です。

4. 統合ターミナルを閉じてください。

     ![06](media/DB30.png)

5. **Visual Studio Code** を閉じてください。

### まとめ

このラボでは、Azure SDK for .NET を使用して Azure Cosmos DB SQL API アカウントに接続する方法を学習しました。最初に Visual Studio Code をセットアップし、Cosmos DB アカウントをプロビジョニングして endpoint と key を取得しました。次に NuGet で Microsoft.Azure.Cosmos ライブラリを確認し、.NET プロジェクトにインポートする方法を学びました。さらに、Cosmos DB に接続してアカウント名とプライマリ書き込みリージョンを取得するスクリプトを作成し、動作を検証しました。

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- Azure Cosmos DB NoSQL API アカウントを作成した。
- NuGet で Microsoft.Azure.Cosmos ライブラリを確認した。
- Microsoft.Azure.Cosmos ライブラリを .NET プロジェクトにインポートした。
- Microsoft.Azure.Cosmos ライブラリを使用した。
- スクリプトをテストした。