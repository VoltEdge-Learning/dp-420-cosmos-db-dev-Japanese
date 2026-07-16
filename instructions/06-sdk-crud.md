---
lab:
    title: 'Azure Cosmos DB SQL API SDK を使用してドキュメントを作成および更新する'
    module: 'モジュール 4 - Azure Cosmos DB SQL API のポイント操作を実装する'
---

# Azure Cosmos DB SQL API SDK を使用してドキュメントを作成および更新する

[Microsoft.Azure.Cosmos.Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] クラスには、Azure Cosmos DB SQL API コンテナー内のアイテムを作成、取得、更新、削除するための一連のメンバー メソッドが含まれています。これらのメソッドを組み合わせることで、SQL API コンテナー内のさまざまなアイテムに対して最も一般的な「CRUD」操作の一部を実行できます。

このラボでは、SDK を使用して Azure Cosmos DB SQL API コンテナー内のアイテムに対する日常的な CRUD 操作を実行します。

## 開発環境を準備する

このラボを実施している環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、次の手順に従って実行してください。それ以外の場合は、以前にクローンしたフォルダーを **Visual Studio Code** で開いてください。

1. **Visual Studio Code** を起動してください。

    > &#128221; Visual Studio Code インターフェイスにまだ慣れていない場合は、[Getting Started ドキュメント][code.visualstudio.com/docs/getstarted] を確認してください。

1. コマンド パレットを開いて **Git: Clone** を実行し、``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリを任意のローカル フォルダーにクローンしてください。

    > &#128161; **CTRL+SHIFT+P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリのクローンが完了したら、選択したローカル フォルダーを **Visual Studio Code** で開いてください。

## Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングする際は、アカウントでサポートする API（たとえば **Mongo API** または **SQL API**）を選択します。Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、endpoint と key を取得し、.NET 用 Azure SDK または任意のその他の SDK を使用して Azure Cosmos DB SQL API アカウントに接続できます。

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインしてください。

1. **+ Create a resource** を選択し、*Cosmos DB* を検索してから、次の設定で新しい **Azure Cosmos DB SQL API** アカウント リソースを作成してください。残りのすべての設定は既定値のままにしてください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Subscription** | *既存の Azure サブスクリプション* |
    | **Resource group** | *既存のリソース グループを選択するか、新しく作成する* |
    | **Account Name** | *グローバルに一意の名前を入力する* |
    | **Location** | *利用可能な任意のリージョンを選択する* |
    | **Capacity mode** | *プロビジョニングされたスループット* |
    | **Apply Free Tier Discount** | *適用しない* |

    > &#128221; ラボ環境には、新しいリソース グループの作成を制限する制約がある場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. このタスクを続行する前に、デプロイ タスクが完了するまで待機してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Keys** ペインに移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

## SDK から Azure Cosmos DB SQL API アカウントに接続する

新しく作成したアカウントの資格情報を使用して SDK クラスで接続し、新しいデータベースとコンテナーのインスタンスを作成します。次に、Data Explorer を使用して、それらのインスタンスが Azure portal に存在することを確認します。

1. **Visual Studio Code** の **Explorer** ペインで、**06-sdk-crud** フォルダーを参照してください。

1. **06-sdk-crud** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

    > &#128221; このコマンドでは、開始ディレクトリがすでに **06-sdk-crud** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加してください。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドしてください。

    ```
    dotnet build
    ```

1. 統合ターミナルを閉じてください。

1. **06-sdk-crud** フォルダー内の **script.cs** コード ファイルを開いてください。

    > &#128221; **[Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1]** ライブラリは、NuGet からすでに事前インポートされています。

1. **endpoint** という名前の **string** 変数を見つけてください。その値を先ほど作成した Azure Cosmos DB アカウントの **endpoint** に設定してください。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは次のようになります: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**。

1. **key** という名前の **string** 変数を見つけてください。その値を先ほど作成した Azure Cosmos DB アカウントの **key** に設定してください。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは次のようになります: **string key = "fDR2ci9QgkdkvERTQ==";**。

1. **client** 変数の CreateDatabaseIfNotExistsAsync メソッドを非同期に呼び出し、作成したい新しいデータベース名（**cosmicworks**）を渡して、結果を **Database** 型の変数に格納してください。

    ```
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    ```

1. **database** 変数の **CreateContainerIfNotExistsAsync** メソッドを非同期に呼び出し、**cosmicworks** データベース内に作成したい新しいコンテナー名（**products**）、パーティション キー パス（**/categoryId**）、スループット（**400**）を渡して、結果を **Container** 型の変数に格納してください。
  
    ```
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);    
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **06-sdk-crud** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

1. 統合ターミナルを閉じてください。

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Resource groups** を選択し、次にこのラボで先ほど作成または確認したリソース グループを選択し、さらにこのラボで作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で、**Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

## SDK でアイテムに対する作成および読み取りのポイント操作を実行する

これから、Microsoft.Azure.Cosmos.Container クラスの一連の非同期メソッドを使用して、SQL API コンテナー内のアイテムに対する一般的な操作を実行します。これらの操作はすべて、C# の task 非同期プログラミング モデルを使用して実行されます。

1. **Visual Studio Code** に戻ってください。**06-sdk-crud** フォルダー内の **product.cs** コード ファイルを開いてください。

    > &#128221; **script.cs** ファイルのエディターは閉じないでください。

1. このコード ファイル内の **Product** クラスを確認してください。このクラスは、このコンテナー内で保存および操作される製品アイテムを表します。

1. **script.cs** コード ファイルのエディター タブに戻ってください。

1. 次のプロパティを持つ **saddle** という名前の **Product** 型の新しいオブジェクトを作成してください。

    | Property | Value |
    | ---: | :--- |
    | **id** | *706cd7c6-db8b-41f9-aea2-0e0c7e8eb009* |
    | **categoryId** | *9603ca6c-9e28-4a02-9194-51cdb7fea816* |
    | **name** | *Road Saddle* |
    | **price** | *45.99d* |
    | **tags** | *{ tan, new, crisp }* |

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };
    ```

1. **container** 変数のジェネリック [CreateItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync] メソッドを非同期に呼び出し、メソッド パラメーターとして **saddle** 変数を渡し、ジェネリック型として **Product** を使用してください。

    ```
    await container.CreateItemAsync<Product>(saddle);
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **06-sdk-crud** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

1. 統合ターミナルを閉じてください。

1. **script.cs** コード ファイルのエディター タブに戻ってください。

1. 次のコード行を削除してください。

    ```
    Product saddle = new()
    {
        id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009",
        categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816",
        name = "Road Saddle",
        price = 45.99d,
        tags = new string[]
        {
            "tan",
            "new",
            "crisp"
        }
    };

    await container.CreateItemAsync<Product>(saddle);
    ```

1. **706cd7c6-db8b-41f9-aea2-0e0c7e8eb009** を値に持つ **id** という名前の string 変数を作成してください。

    ```
    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";
    ```

1. **9603ca6c-9e28-4a02-9194-51cdb7fea816** を値に持つ **categoryId** という名前の string 変数を作成してください。

    ```
    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    ```

1. **categoryId** 変数をコンストラクター パラメーターとして渡し、[PartitionKey][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey] 型の **partitionKey** という名前の変数を作成してください。

    ```
    PartitionKey partitionKey = new (categoryId);
    ```

1. **container** 変数のジェネリック [ReadItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync] メソッドを非同期に呼び出し、**id** 変数と **partitionkey** 変数をメソッド パラメーターとして渡し、ジェネリック型として **Product** を使用して、結果を **Product** 型の **saddle** という名前の変数に格納してください。

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);
    ```

1. 静的 **Console.WriteLine** メソッドを呼び出し、書式付きの出力文字列を使用して saddle オブジェクトを出力してください。

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **06-sdk-crud** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

1. ターミナルの出力を確認してください。特に、アイテムの id、name、price を含む書式付き出力テキストを確認してください。

1. 統合ターミナルを閉じてください。

## SDK で更新および削除のポイント操作を実行する

SDK を学習する際、オンラインの Azure Cosmos DB SDK アカウントまたはエミュレーターを使用してアイテムを更新し、操作を実行して変更が適用されたかどうかを確認しながら Data Explorer と任意の IDE の間を行き来することは珍しくありません。ここでは、SDK を使用してアイテムを更新および削除することで、まさにそれを行います。

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Resource groups** を選択し、次にこのラボで先ほど作成または確認したリソース グループを選択し、さらにこのラボで作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で、**Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、次に **SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを展開してください。

1. **Items** ノードを選択してください。コンテナー内の唯一のアイテムを選択し、アイテムの **name** プロパティと **price** プロパティの値を確認してください。

    | **Property** | **Value** |
    | ---: | :--- |
    | **Name** | *Road Saddle* |
    | **Price** | *$45.99* |

    > &#128221; この時点では、アイテムを作成して以来これらの値は変更されていないはずです。この演習でこれらの値を変更します。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

1. **Visual Studio Code** に戻ってください。**script.cs** コード ファイルのエディター タブに戻ってください。

1. 次のコード行を削除してください。

    ```
    Console.WriteLine($"[{saddle.id}]\t{saddle.name} ({saddle.price:C})");
    ```

1. price プロパティの値を **32.55** に設定して **saddle** 変数を変更してください。

    ```
    saddle.price = 32.55d;
    ```

1. **name** プロパティの値を **Road LL Saddle** に設定して **saddle** 変数を再度変更してください。

    ```
    saddle.name = "Road LL Saddle";
    ```

1. **container** 変数のジェネリック [UpsertItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync] メソッドを非同期に呼び出し、メソッド パラメーターとして **saddle** 変数を渡し、ジェネリック型として **Product** を使用してください。

    ```
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);
    
    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    Container container = await database.CreateContainerIfNotExistsAsync("products", "/categoryId", 400);

    string id = "706cd7c6-db8b-41f9-aea2-0e0c7e8eb009";

    string categoryId = "9603ca6c-9e28-4a02-9194-51cdb7fea816";
    PartitionKey partitionKey = new (categoryId);

    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **06-sdk-crud** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

1. 統合ターミナルを閉じてください。

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Resource groups** を選択し、次にこのラボで先ほど作成または確認したリソース グループを選択し、さらにこのラボで作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で、**Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、次に **SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを展開してください。

1. **Items** ノードを選択してください。コンテナー内の唯一のアイテムを選択し、アイテムの **name** プロパティと **price** プロパティの値を確認してください。

    | **Property** | **Value** |
    | ---: | :--- |
    | **Name** | *Road LL Saddle* |
    | **Price** | *$32.55* |

    > &#128221; この時点では、アイテムを確認した後なので、これらの値は変更されているはずです。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

1. **Visual Studio Code** に戻ってください。**script.cs** コード ファイルのエディター タブに戻ってください。

1. 次のコード行を削除してください。

    ```
    Product saddle = await container.ReadItemAsync<Product>(id, partitionKey);

    saddle.price = 32.55d;
    saddle.name = "Road LL Saddle";
    
    await container.UpsertItemAsync<Product>(saddle);
    ```

1. **container** 変数のジェネリック [DeleteItemAsync<>][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync] メソッドを非同期に呼び出し、メソッド パラメーターとして **id** 変数と **partitionkey** 変数を渡し、ジェネリック型として **Product** を使用してください。

    ```
    await container.DeleteItemAsync<Product>(id, partitionKey);
    ```

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **06-sdk-crud** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

1. 統合ターミナルを閉じてください。

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Resource groups** を選択し、次にこのラボで先ほど作成または確認したリソース グループを選択し、さらにこのラボで作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で、**Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、次に **SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを展開してください。

1. **Items** ノードを選択してください。アイテム一覧が空になっていることを確認してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

1. **Visual Studio Code** を閉じてください。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.createitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.deleteitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.readitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.upsertitemasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.partitionkey
[docs.microsoft.com/dotnet/core/tools/dotnet-build]: https://docs.microsoft.com/dotnet/core/tools/dotnet-build
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
