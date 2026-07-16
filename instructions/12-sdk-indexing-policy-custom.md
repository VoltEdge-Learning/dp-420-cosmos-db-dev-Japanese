# Lab 06b - Azure Cosmos DB for NoSQL のインデックス戦略を定義して実装する

## ラボ シナリオ

インデックス ポリシーは、Azure Cosmos DB のいずれの SDK からでも管理できます。特に .NET SDK には、Azure Cosmos DB for NoSQL のコンテナーに新しいインデックス ポリシーを設計して適用するためのクラス群が用意されています。

このラボでは、.NET SDK を使用してコンテナーのカスタム インデックス ポリシーを作成します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: .NET SDK を使用して新しいインデックス ポリシーを作成する。
- タスク 3: .NET SDK で作成したインデックス ポリシーを Data Explorer で確認する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab12.png)


## 演習 1: ポータルを使用して Azure Cosmos DB for NoSQL コンテナーのインデックス ポリシーを構成する

### タスク 1: 開発環境を準備する

作業環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、次の手順でクローンしてください。すでにクローン済みの場合は、以前クローンしたフォルダーを **Visual Studio Code** で開いてください。

1. **Visual Studio Code** を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

    > &#128221; Visual Studio Code のインターフェイスにまだ慣れていない場合は、[Get Started guide for Visual Studio Code][code.visualstudio.com/docs/getstarted] を確認してください。

1. 左側ペインの **Extension (1)** アイコンを選択してください。検索バーに **C# (2)** を入力し、表示された **extension (3)** を選択して、最後に拡張機能の **Install (4)** を選択してください。

    ![](media/C-hash-extension.png)

1. ファイルを開きます。左上メニューから **file->Open Folder** をクリックし、**C:\AllFiles** に移動してください。

1. **dp-420-cosmos-db-dev-stage** フォルダーを選択し、**Select Folder** をクリックしてください。

### タスク 2: .NET SDK を使用して新しいインデックス ポリシーを作成する

.NET SDK には、親クラス [Microsoft.Azure.Cosmos.IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] に関連するクラス群が含まれており、コードで新しいインデックス ポリシーを構築できます。

1. **Visual Studio Code** の **Explorer** ペインで、**12-custom-index-policy** フォルダーに移動してください。

1. **script.cs** コード ファイルを開いてください。

1. 既存の **endpoint** という名前の変数を更新し、前のラボで作成した Azure Cosmos DB アカウントの **endpoint** を設定してください。

    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. 既存の **key** という名前の変数を更新し、前のラボで作成した Azure Cosmos DB アカウントの **key** を設定してください。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. 既定の空コンストラクターを使用して、**policy** という名前の [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy] 型の新しい変数を作成してください。

    ```
    IndexingPolicy policy = new ();
    ```

1. **policy** 変数の [IndexingMode][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.indexingmode] プロパティを [IndexingMode.Consistent][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingmode#fields] に設定してください。

    ```
    policy.IndexingMode = IndexingMode.Consistent;
    ```

1. [ExcludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath] 型の新しいオブジェクトを作成し、[Path][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.excludedpath.path] プロパティを **/*** に設定して、**policy** 変数の [ExcludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.excludedpaths] コレクション プロパティに追加してください。

    ```
    policy.ExcludedPaths.Add(
        new ExcludedPath{ Path = "/*" }
    );
    ```

1. [IncludedPath][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath] 型の新しいオブジェクトを作成し、[Path][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.includedpath.path] プロパティを **/name/?** に設定して、**policy** 変数の [IncludedPaths][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.indexingpolicy.includedpaths] コレクション プロパティに追加してください。

    ```
    policy.IncludedPaths.Add(
        new IncludedPath{ Path = "/name/?" }
    );
    ```

1. ``products`` と ``/categoryId`` の値をコンストラクター パラメーターとして渡し、**options** という名前の [ContainerProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties] 型の新しい変数を作成してください。

    ```
    ContainerProperties options = new ("products", "/categoryId");
    ```

1. **options** 変数の [IndexingPolicy][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.containerproperties.indexingpolicy] プロパティに **policy** 変数を割り当ててください。

    ```
    options.IndexingPolicy = policy;
    ```

1. **database** 変数の [CreateContainerIfNotExistsAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.database.createcontainerifnotexistsasync] メソッドを非同期で呼び出し、コンストラクター パラメーターとして **options** 変数を渡し、結果を **container** という名前の [Container][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container] 型変数に格納してください。

    ```
    Container container = await database.CreateContainerIfNotExistsAsync(options);
    ```

1. 組み込みの静的 **Console.WriteLine** メソッドを使用して、Container クラスの [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.container.id] プロパティを **Container Created** ヘッダー付きで出力してください。

    ```
    Console.WriteLine($"Container Created [{container.Id}]");
    ```

1. 完了後、コード ファイルに次の内容が含まれていることを確認してください。

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

1. **Visual Studio Code** で **12-custom-index-policy** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```

1. スクリプトにより、新しく作成されたコンテナー名が出力されます。

    ```
    Container Created [products]
    ```

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

### タスク 3: .NET SDK で作成したインデックス ポリシーを Data Explorer で確認する

他のインデックス ポリシーと同様に、.NET SDK で適用したポリシーは Data Explorer で確認できます。ここではポータルを使用して、このラボでコードから作成したポリシーを確認します。

1. Web ブラウザーで Azure portal (``portal.azure.com``) に移動してください。

1. **Resource groups** を選択し、このラボで作成または確認したリソース グループを選択してから、このラボで作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で **Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**API for NoSQL** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認してください。

1. **API for NoSQL** ナビゲーション ツリーの **products** コンテナー ノード内で **Scale & Settings** を選択してください。

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

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- .NET SDK を使用して新しいインデックス ポリシーを作成した。
- .NET SDK で作成したインデックス ポリシーを Data Explorer で確認した。

### ラボは正常に完了しました