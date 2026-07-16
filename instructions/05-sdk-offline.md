# オフライン開発用に Azure Cosmos DB for NoSQL SDK を構成する

### 推定所要時間: 60 分

## ラボ シナリオ

Azure Cosmos DB Emulator は、開発およびテストのために Azure Cosmos DB サービスをローカルでシミュレートするツールです。API for NoSQL をサポートしており、クラウド サービスを使用せずにアプリケーションを構築およびテストできます。

このラボでは、Azure SDK for .NET を使用して Azure Cosmos DB Emulator に接続します。

## ラボの目的

このラボでは、次のタスクを完了します。

- タスク 1: Azure Cosmos DB Emulator を起動する。
- タスク 2: SDK からエミュレーターに接続する。
- タスク 3: エミュレーターで変更を確認する。
- タスク 4: 新しいコンテナーを作成して確認する。

### タスク 1: Azure Cosmos DB Emulator を起動する
このタスクでは、Azure Cosmos DB 環境をローカルでシミュレートするツールである Azure Cosmos DB Emulator を起動します。エミュレーターから接続文字列を取得します。エミュレーターが実行中であることを確認し、Data Explorer の初期状態を確認します。

環境にはすでにエミュレーターがインストールされているはずです。インストールされていない場合は、[installation instructions][docs.microsoft.com/azure/cosmos-db/local-emulator] を参照して Azure Cosmos DB Emulator をインストールしてください。エミュレーターを起動したら、接続文字列を取得し、Azure SDK for .NET または任意の SDK を使用してエミュレーターに接続できます。

1. Windows のスタート メニューから **Azure Cosmos DB Emulator** を検索して起動してください。
     ![06](media/DB31.png)

1. **3～4 分待機**してください。エミュレーターが起動すると、既定のブラウザーが自動的に開き、**localhost:8081/_explorer/index.html** のランディング ページに移動します。

1. **Azure Cosmos DB Emulator** のランディング ページで、**Quickstart** ペインに移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    1. **Primary Connection String** フィールドの値を記録してください。この演習の後半で **connection string** 値として使用します。

         ![06](media/New-image56.png)

1. **Explorer** ペインに移動してください。**Data Explorer** で、**API for NoSQL** ナビゲーション ツリー内にノードが存在しないことを確認してください。

   ![06](media/New-image57.png)

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

###  タスク 2: SDK からエミュレーターに接続する
このタスクでは、Microsoft.Azure.Cosmos SDK を使用して Azure Cosmos DB Emulator に接続します。提供されているスクリプト内の接続文字列を更新し、エミュレーター内に新しいデータベースを作成するコードを記述して、スクリプト実行により接続をテストします。

この演習で使用する .NET スクリプトには、**Microsoft.Azure.Cosmos** ライブラリがすでに事前インストールされています。さらに、時間短縮のためにいくつかのボイラープレート コードも記述済みです。スクリプトを完成させるには、ボイラープレートの接続文字列値を更新し、数行のコードを追加する必要があります。

1. デスクトップから Visual Studio Code に戻ってください。

     ![Visual Studio Code Icon](./media/vscode1.jpg)

1. 画面左上の **file (1)** を選択し、メニューから **Open Folder (2)** を選択してください。**C:\AllFiles\dp-420-cosmos-db-dev** に移動してください。

     ![06](media/New-image51.png)

1. **C:\AllFiles\dp-420-cosmos-db-dev** に移動し、**dp-420-cosmos-db-dev** を選択して **Select Folder** をクリックしてください。

    ![06](media/New-image54.png)

1. **05-sdk-offline** フォルダーを選択し、**Select Folder** をクリックしてください。

1. **Visual Studio Code** で、**05-sdk-offline (1)** フォルダー内の空の **script.cs (2)** コード ファイルを開いてください。

    ![06](media/DB32.png)

1. 既存の **connectionString** という名前の変数を更新し、値を Azure Cosmos DB Emulator の **connection string** に設定してください。
  
    ```
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    ```

    >**Note**: エミュレーターの URI は通常 ***localhost:[port]*** 形式で、SSL を使用し、既定ポートは **8081** です。

     >**Note**: *C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==* は、すべてのエミュレーター インストールで既定のキーです。このキーはコマンドライン オプションで変更できます。

1. **client** 変数の [CreateDatabaseIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync] メソッドを非同期で呼び出し、エミュレーター内に作成する新しいデータベース名（**cosmicworks**）を渡し、結果を [Database][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database] 型の変数に格納してください。

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、Database クラスの [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id] プロパティを **New Database** ヘッダー付きで出力してください。

    ```
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. **script.cs** コード ファイルを **Save** してください。

    ![06](media/DB33.png)

1. **Visual Studio Code** で **05-sdk-offline (1** フォルダーを右クリックし、**Open in Integrated Terminal (2)** を選択して新しいターミナルを開いてください。
    
    ![06](media/1.png)
    
    >**Note**: このコマンドでは、開始ディレクトリがすでに **05-sdk-offline** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加してください。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

    >**Note:** アプリケーション実行中に VS Code がクラッシュした場合は、すべてのアプリを一度閉じてからこの手順を再実行してください。

    ![06](media/DB35.png)

1. 統合ターミナルを閉じてください。

    > ラボ完了おめでとうございます。次は検証です。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、ラボは正常に検証されています。
    > - 表示されない場合は、エラーメッセージをよく確認し、ラボ ガイドの手順に従って再試行してください。
    > - サポートが必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

    <validation step="1e23a88b-ed78-4557-8384-97cc0833dcbb" />

###  タスク 3: エミュレーターで変更を確認する

このタスクでは、Azure Cosmos DB Emulator の Data Explorer を使用して、作成した新しい NoSQL データベースを確認します。ブラウザー経由でエミュレーターにアクセスし、API for NoSQL ナビゲーション ツリー内に新しい "cosmicworks" データベースが表示されることを確認します。

Azure Cosmos DB Emulator で新しいデータベースを作成したので、オンラインの **Data Explorer** を使用して、エミュレーター内の新しい API for NoSQL データベースを確認します。

1. Windows のシステム トレイにあるエミュレーター アイコンに移動し、コンテキスト メニューを開いて **Open Data Explorer...** を選択し、既定のブラウザーで **localhost:8081/_explorer/** のランディング ページに移動してください。

1. **Azure Cosmos DB Emulator** のランディング ページで、**Explorer** ペインに移動してください。

1. **Data Explorer** で、**API for NoSQL** ナビゲーション ツリー内の新しい **cosmicworks** データベース ノードを確認してください。

    ![06](media/DB34.png)

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

###  タスク 4: 新しいコンテナーを作成して確認する

このタスクでは、前のスクリプトを拡張して、"cosmicworks" データベースに "products" という新しいコンテナーを作成します。スクリプト実行後、Azure Cosmos DB Emulator の Data Explorer でコンテナーを確認して作成結果を検証します。この処理はデータベース作成と似ており、接続文字列を変更することでクラウド環境とエミュレーター環境の両方で同じコードを再利用できます。

新しいコンテナーの作成は、新しいデータベースを作成するパターンと同様です。ここで学ぶコードは、クラウドでもエミュレーターでもリソース作成時に有効で、必要なのは接続文字列の変更だけです。スクリプト ファイルをさらに拡張し、データベースとともに新しいコンテナーを作成します。

1. **Visual Studio Code** で、**05-sdk-offline (1)** フォルダー内の空の **script.cs (2)** コード ファイルを開いてください。

    ![06](media/DB32.png)

1. **database** 変数の [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] メソッドを非同期で呼び出し、新しいコンテナー名（**products**）、パーティション キー パス（**/categoryId**）、スループット（**400**）を渡して **cosmicworks** データベース内に作成し、結果を [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 型の変数に格納してください。

    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、Container クラスの [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] プロパティを **New Container** ヘッダー付きで出力してください。

    ```
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. **script.cs** コード ファイルを **Save** してください。

     ![06](media/DB36.png)

1. **Visual Studio Code** で **05-sdk-offline** フォルダーを右クリックし、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。
    
    ![06](media/1.png)

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

    ![06](media/DB37.png)

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

1. Windows のシステム トレイにあるエミュレーター アイコンに移動し、コンテキスト メニューを開いて **Open Data Explorer...** を選択し、既定のブラウザーで **localhost:8081/_explorer/index.html** のランディング ページに移動してください。

1. **Azure Cosmos DB Emulator** のランディング ページで、**Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**API for NoSQL** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認してください。

    ![06](media/New-image60.png)
   
1. Web ブラウザーのウィンドウまたはタブを閉じてください。

    > ラボ完了おめでとうございます。次は検証です。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、ラボは正常に検証されています。
    > - 表示されない場合は、エラーメッセージをよく確認し、ラボ ガイドの手順に従って再試行してください。
    > - サポートが必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

    <validation step="d95b17d3-9a2e-4a03-adde-9ad042168bea" />

### まとめ

このラボでは、オフライン開発のために API for NoSQL で Azure Cosmos DB Emulator を構成しました。主な作業は、エミュレーターの起動、Azure SDK for .NET を介した接続、"cosmicworks" という新しいデータベースの作成、"products" というコンテナーの追加です。最後に、エミュレーターの Data Explorer で変更内容を確認し、Cosmos DB 開発環境を実践的に体験しました。

### レビュー

このラボでは、次を完了しました。

- タスク 1: Azure Cosmos DB Emulator を起動した。
- タスク 2: SDK からエミュレーターに接続した。
- タスク 3: エミュレーターで変更を確認した。
- タスク 4: 新しいコンテナーを作成して確認した。

### ラボは正常に完了しました