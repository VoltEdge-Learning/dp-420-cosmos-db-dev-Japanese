---
lab:
    title: 'オフライン開発向けに Azure Cosmos DB SQL API SDK を構成する'
    module: 'モジュール 3 - SDK を使用して Azure Cosmos DB SQL API に接続する'
---

# オフライン開発向けに Azure Cosmos DB SQL API SDK を構成する

Azure Cosmos DB Emulator は、開発およびテストのために Azure Cosmos DB サービスをエミュレートするローカル ツールです。エミュレーターは SQL API をサポートしており、.NET 用 Azure SDK を使用してコードを開発する際にクラウド サービスの代わりに使用できます。

このラボでは、.NET 用 Azure SDK から Azure Cosmos DB Emulator に接続します。

## 開発環境を準備する

このラボを実施している環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、次の手順に従って実行してください。それ以外の場合は、以前にクローンしたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスにまだ慣れていない場合は、[Getting Started ドキュメント][code.visualstudio.com/docs/getstarted] を確認してください。

1. コマンド パレットを開いて **Git: Clone** を実行し、``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリを任意のローカル フォルダーにクローンします。

    > &#128161; **CTRL+SHIFT+P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリのクローンが完了したら、選択したローカル フォルダーを **Visual Studio Code** で開きます。

## Azure Cosmos DB Emulator を起動する

使用している環境には、エミュレーターがすでに事前インストールされているはずです。インストールされていない場合は、[インストール手順][docs.microsoft.com/azure/cosmos-db/local-emulator] を参照して Azure Cosmos DB Emulator をインストールしてください。エミュレーターの起動後、接続文字列を取得し、.NET 用 Azure SDK または任意のその他の SDK を使用してエミュレーターに接続できます。

1. **Azure Cosmos DB Emulator** を起動します。

    > &#128221; エミュレーターを起動するために管理者アクセスの許可を求められる場合があります。ラボ環境では、**Admin** アカウントは **Student** アカウントと同じパスワードです。

    > &#128161; Azure Cosmos DB Emulator は Windows のタスク バーとスタート メニューの両方にピン留めされています。***ピン留めされたアイコンから Emulator が起動しない場合は、*** **C:\Program Files\Azure Cosmos DB Emulator\CosmosDB.Emulator.exe** ***ファイルをダブルクリックして開いてみてください***。エミュレーターの起動には 20～30 秒かかることに注意してください。

1. エミュレーターが自動的に既定のブラウザーを開き、**localhost:8081/_explorer/index.html** ランディング ページに移動するまで待ちます。

1. **Azure Cosmos DB Emulator** のランディング ページで、**Quickstart** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。具体的には次のとおりです。

    1. **Primary Connection String** フィールドの値を記録します。この演習の後半でこの **connection string** 値を使用します。

1. **Explorer** ペインに移動します。

1. **Data Explorer** で、**SQL API** ナビゲーション ツリー内にノードがないことを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## SDK からエミュレーターに接続する

この演習で使用する .NET スクリプトには、**Microsoft.Azure.Cosmos** ライブラリがすでに事前インストールされています。さらに、時間を節約するために一部のボイラープレート コードはすでに記述されています。スクリプトを完成させるには、ボイラープレートの接続文字列値を更新し、数行のコードを記述する必要があります。

1. **Visual Studio Code** の **Explorer** ペインで、**05-sdk-offline** フォルダーを参照します。

1. **05-sdk-offline** フォルダー内の **script.cs** コード ファイルを開きます。

1. 既存の **connectionString** という名前の変数を更新し、その値を Azure Cosmos DB Emulator の **connection string** に設定します。
  
    ```
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    ```

    > &#128221; エミュレーターの URI は通常 ***localhost:[port]*** で、SSL を使用し、既定のポートは **8081** に設定されています。

    > &#128221; *C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==* は、エミュレーターのすべてのインストールで既定のキーです。このキーはコマンド ライン オプションを使用して変更できます。

1. **client** 変数の [CreateDatabaseIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync] メソッドを非同期に呼び出し、エミュレーター内に作成したい新しいデータベース名（**cosmicworks**）を渡し、結果を [Database][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database] 型の変数に格納します。

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、**New Database** という見出しで Database クラスの [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id] プロパティを出力します。

    ```
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. 完了すると、コード ファイルには次の内容が含まれているはずです。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;
    
    string connectionString = "AccountEndpoint=https://localhost:8081/;AccountKey=C2y6yDjf5/R+ob0N8A7Cgv30VRDJIWEHLM+4QDU5DE2nQ9nDuVTqobD4b8mGGyPMbIZnqyMsEcaGQy67XIw/Jw==";
    
    CosmosClient client = new (connectionString);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    Console.WriteLine($"New Database:\tId: {database.Id}");
    ```

1. **script.cs** コード ファイルを **Save** します。

1. **Visual Studio Code** で **05-sdk-offline** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドでは、開始ディレクトリがすでに **05-sdk-offline** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加します。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドし、実行します。

    ```
    dotnet run
    ```

1. 統合ターミナルを閉じます。

## エミュレーターで変更を確認する

Azure Cosmos DB エミュレーターに新しいデータベースを作成したので、オンラインの **Data Explorer** を使用して、エミュレーター内の新しい SQL API データベースを確認します。

1. Windows システム トレイのエミュレーター アイコンに移動し、コンテキスト メニューを開いて **Open Data Explorer...** を選択し、既定のブラウザーを使用して **localhost:8081/_explorer/** ランディング ページに移動します。

1. **Azure Cosmos DB Emulator** のランディング ページで、**Explorer** ペインに移動します。

1. **Data Explorer** で、**SQL API** ナビゲーション ツリー内の新しい **cosmicworks** データベース ノードを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## 新しいコンテナーを作成して確認する

新しいコンテナーの作成は、新しいデータベースを作成するために使用したパターンと似ています。ここで学ぶコードは、クラウドでリソースを作成する場合でもエミュレーターで作成する場合でも有効であり、接続文字列を変更するだけです。データベースに加えて新しいコンテナーを作成できるように、スクリプト ファイルをさらに拡張します。

1. **Visual Studio Code** の **Explorer** ペインで、**05-sdk-offline** フォルダーを参照します。

1. **05-sdk-offline** フォルダー内の **script.cs** コード ファイルを再度開きます。

1. **database** 変数の [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] メソッドを非同期に呼び出し、**cosmicworks** データベース内に作成したい新しいコンテナー名（**products**）、パーティション キー パス（**/categoryId**）、スループット（**400**）を渡し、結果を [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 型の変数に格納します。

    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、**New Container** という見出しで Container クラスの [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] プロパティを出力します。

    ```
    Console.WriteLine($"New Container:\tId: {container.Id}");
    ```

1. 完了すると、コード ファイルには次の内容が含まれているはずです。
  
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

1. **script.cs** コード ファイルを **Save** します。

1. **Visual Studio Code** で **05-sdk-offline** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開きます。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドし、実行します。

    ```
    dotnet run
    ```

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

1. Windows システム トレイのエミュレーター アイコンに移動し、コンテキスト メニューを開いて **Open Data Explorer...** を選択し、既定のブラウザーを使用して **localhost:8081/_explorer/** ランディング ページに移動します。

1. **Azure Cosmos DB Emulator** のランディング ページで、**Explorer** ペインに移動します。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## Azure Cosmos DB Emulator を停止する

環境内のシステム リソースを使用するため、使用が終わったらエミュレーターを停止することが重要です。システム トレイ アイコンを使用して、エミュレーターと実行中のすべてのインスタンスを停止します。

1. Windows システム トレイのエミュレーター アイコンに移動し、コンテキスト メニューを開いて **Exit** を選択し、エミュレーターをシャットダウンします。

    > &#128221; エミュレーターのすべてのインスタンスが終了するまでに 1 分ほどかかる場合があります。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.createdatabaseifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.id
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
