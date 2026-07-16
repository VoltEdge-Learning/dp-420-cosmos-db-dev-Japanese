---
lab:
    title: 'ポータルを使用して Azure Cosmos DB SQL API コンテナーのインデックス ポリシーを構成する'
    module: 'モジュール 6 - Azure Cosmos DB SQL API のインデックス戦略を定義して実装する'
---

# SDK を使用して Azure Cosmos DB SQL API コンテナーのインデックス ポリシーを構成する

インデックス ポリシーは、どの Azure Cosmos DB SDK からでも管理できます。.NET SDK には特に、Azure Cosmos DB SQL API のコンテナーに新しいインデックス ポリシーを設計して適用するために使用できるクラス群が含まれています。

このラボでは、.NET SDK を使用してコンテナー用のカスタム インデックス ポリシーを作成します。

## 開発環境を準備する

このラボを実施している環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、次の手順に従って実行してください。それ以外の場合は、以前にクローンしたフォルダーを **Visual Studio Code** で開いてください。

1. **Visual Studio Code** を起動してください。

    > &#128221; Visual Studio Code のインターフェイスにまだ慣れていない場合は、[Visual Studio Code の Get Started ガイド][code.visualstudio.com/docs/getstarted] を確認してください。

1. コマンド パレットを開いて **Git: Clone** を実行し、``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリを任意のローカル フォルダーにクローンしてください。

    > &#128161; **CTRL+SHIFT+P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリのクローンが完了したら、選択したローカル フォルダーを **Visual Studio Code** で開いてください。

## Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングするときは、アカウントでサポートする API（たとえば **Mongo API** や **SQL API**）を選択します。Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、それらを使用して Azure SDK for .NET または任意の SDK から Azure Cosmos DB SQL API アカウントに接続できます。

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインしてください。

1. **+ Create a resource** を選択し、*Cosmos DB* を検索してから、次の設定で新しい **Azure Cosmos DB SQL API** アカウント リソースを作成してください。残りの設定はすべて既定値のままにしてください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Subscription** | *既存の Azure サブスクリプション* |
    | **Resource group** | *既存のリソース グループを選択するか新規作成する* |
    | **Account Name** | *グローバルで一意の名前を入力する* |
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

## .NET SDK を使用して新しいインデックス ポリシーを作成する

.NET SDK には、コードで新しいインデックス ポリシーを構築するための、親クラス [Microsoft.Azure.Cosmos.IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] に関連するクラス群が含まれています。

1. **Visual Studio Code** の **Explorer** ペインで、**12-custom-index-policy** フォルダーを参照してください。

1. **script.cs** コード ファイルを開いてください。

1. 既存の **endpoint** という名前の変数を更新し、先ほど作成した Azure Cosmos DB アカウントの **endpoint** を値として設定してください。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは次のようになります: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**。

1. 既存の **key** という名前の変数を更新し、先ほど作成した Azure Cosmos DB アカウントの **key** を値として設定してください。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは次のようになります: **string key = "fDR2ci9QgkdkvERTQ==";**。

1. 既定の空コンストラクターを使用して、[IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] 型の **policy** という名前の新しい変数を作成してください。

    ```
    IndexingPolicy policy = new ();
    ```

1. **policy** 変数の [IndexingMode][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode] プロパティを [IndexingMode.Consistent][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields] に設定してください。

    ```
    policy.IndexingMode = IndexingMode.Consistent;
    ```

1. [ExcludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath] 型の新しいオブジェクトを **policy** 変数の [ExcludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths] コレクション プロパティに追加してください。このとき、[Path][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path] プロパティは **/*** に設定してください。

    ```
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    ```

1. [IncludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath] 型の新しいオブジェクトを **policy** 変数の [IncludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths] コレクション プロパティに追加してください。このとき、[Path][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path] プロパティは **/name/?** に設定してください。

    ```
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );
    ```

1. [ContainerProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties] 型の **options** という名前の新しい変数を作成し、コンストラクター パラメーターとして ``products`` と ``/categoryId`` を渡してください。

    ```
    ContainerProperties options = new ("products", "/categoryId");
    ```

1. **options** 変数の [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy] プロパティに **policy** 変数を割り当ててください。

    ```
    options.IndexingPolicy = policy;
    ```

1. **database** 変数の [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] メソッドを非同期で呼び出し、コンストラクター パラメーターとして **options** 変数を渡し、結果を [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 型の **container** という変数に格納してください。

    ```
    Container container = await database.CreateContainerIfNotExistsAsync(options);
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、Container クラスの [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] プロパティを、**Container Created** というヘッダー付きで出力してください。

    ```
    Console.WriteLine($"Container Created [{container.Id}]");
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
    ```
    using System;
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";

    string key = "<cosmos-key>";

    CosmosClient client = new CosmosClient(endpoint, key);

    Database database = await client.CreateDatabaseIfNotExistsAsync("cosmicworks");
    
    IndexingPolicy policy = new ();
    policy.IndexingMode = IndexingMode.Consistent;
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );

    ContainerProperties options = new ("products", "/categoryId");
    options.IndexingPolicy = policy;

    Container container = await database.CreateContainerIfNotExistsAsync(options);
    Console.WriteLine($"Container Created [{container.Id}]");
    ```

1. **script.cs** ファイルを **Save** してください。

1. **Visual Studio Code** で **12-custom-index-policy** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開いてください。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドし、実行してください。

    ```
    dotnet run
    ```

1. スクリプトにより、新しく作成されたコンテナーの名前が出力されます。

    ```
    Container Created [products]
    ```

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

## Data Explorer を使用して .NET SDK で作成したインデックス ポリシーを確認する

他のインデックス ポリシーと同様に、.NET SDK を使用して適用したポリシーも Data Explorer で表示できます。ここでは、ポータルを使用してこのラボでコードから作成したポリシーを確認します。

1. Web ブラウザーで Azure portal (``portal.azure.com``) に移動してください。

1. **Resource groups** を選択し、次にこのラボで先ほど作成または確認したリソース グループを選択し、さらにこのラボで作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で、**Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認してください。

1. **SQL API** ナビゲーション ツリーの **products** コンテナー ノード内で、**Scale & Settings** を選択してください。

1. **Indexing Policy** セクション内のインデックス ポリシーを確認してください。

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/name/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        },
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

    > &#128221; これは、このラボで .NET SDK を使用して作成したインデックス ポリシーの JSON 表現です。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
