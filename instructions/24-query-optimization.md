# Lab 10b - Azure Cosmos DB for NoSQL でクエリ パフォーマンスを最適化する

## ラボ シナリオ

Azure Cosmos DB for NoSQL アカウントを設計する際、最もよく使われるクエリを把握しておくことで、クエリ パフォーマンスを最大化できるようにインデックス作成ポリシーを調整しやすくなります。

このラボでは、Data Explorer を使用して、既定のインデックス作成ポリシーと、複合インデックスを含むインデックス作成ポリシーで SQL クエリをテストします。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: Azure Cosmos DB for NoSQL アカウントを作成する。
- タスク 2: Azure Cosmos DB for NoSQL アカウントにサンプル データをシードする。
- タスク 3: SQL クエリを実行し、要求ユニット課金を測定する。
- タスク 4: インデックス作成ポリシーに複合インデックスを作成する。

## 推定所要時間: 60 分

## アーキテクチャ図

![image](architecturedia/lab24.png)

## 演習 1: クエリ向けに Azure Cosmos DB for NoSQL コンテナーのインデックス作成ポリシーを最適化する

### タスク 1: Azure Cosmos DB for NoSQL アカウントを作成する

Azure Cosmos DB は複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングするときは、アカウントでサポートする API（例: **API for MongoDB** または **API for NoSQL**）を選択します。Azure Cosmos DB for NoSQL アカウントのプロビジョニング完了後、エンドポイントとキーを取得し、Azure SDK for .NET または任意の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続できます。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Azure services** カテゴリで **Create a resource** を選択し、次に **Azure Cosmos DB** を選択してください。

    > &#128161; 別の方法として、**&#8801;** メニューを展開し、**All Services** を選択して、**Databases** カテゴリの **Azure Cosmos DB** を選択し、**Create** を選択してください。

1. **Select API option** ペインで、**Azure Cosmos DB for NoSQL** セクション内の **Create** オプションを選択してください。

1. **Create Azure Cosmos DB Account** ペインで **Basics** タブを確認してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Subscription** | *Your existing Azure subscription* |
    | **Resource group** | *DP-420-DeploymentID* |
    | **Account Name** | *Enter a globally unique name* |
    | **Location** | *Choose any available region* |
    | **Capacity mode** | *Serverless* |


    >**Note** : DeploymentID は各環境に関連付けられた一意の ID です。値は environment details ページで確認できます。

1. **Review + Create** をクリックし、検証で Success が表示されたら **Create** をクリックしてください。

1. このタスクを続行する前に、デプロイが完了するまで待機してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Data Explorer** ペインに移動してください。

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Database id** | *Create new* &vert; *cosmicworks* |
    | **Container id** | *products* |
    | **Partition key** | */categoryId* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認してください。

1. リソース ブレードで **Keys** ペインに移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

### タスク 2: Azure Cosmos DB for NoSQL アカウントにサンプル データをシードする

**cosmicworks** データベースと **products** コンテナーを作成するコマンドライン ユーティリティを使用します。次に、このツールで複数のアイテムを作成し、ターミナルで実行中の変更フィード プロセッサを使ってそれらを確認します。

1. **Visual Studio Code** で **Terminal** メニューを開き、**Split Terminal** を選択して、既存インスタンスと並べて新しいターミナルを開いてください。

1. マシン全体で使用できるように [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをインストールしてください。

    ```
    dotnet tool install --global cosmicworks
    ```

    > **Note:** このコマンドは完了までに数分かかる場合があります。過去にこのツールの最新バージョンをインストール済みの場合、警告メッセージ（*Tool 'cosmicworks' is already installed*）が表示されます。

1. 次のコマンドライン オプションを使用して cosmicworks を実行し、Azure Cosmos DB アカウントにデータをシードしてください。

    | **Option** | **Value** |
    | --- | --- |
    | **--endpoint** | *このラボで先ほどコピーした endpoint 値* |
    | **--key** | *このラボで先ほどコピーした key 値* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > **Note:** たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/**、key が **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。
    > ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. **cosmicworks** コマンドが、データベース、コンテナー、アイテムの作成を完了するまで待機してください。

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

### タスク 3: SQL クエリを実行し、要求ユニット課金を測定する

インデックス作成ポリシーを変更する前に、まずサンプル SQL クエリをいくつか実行して、RU で表される要求ユニット課金のベースラインを取得します。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Resource groups** を選択し、このラボで先ほど作成または確認したリソース グループを選択してから、このラボで作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で **Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを選択して、**New SQL Query** を選択してください。

1. **Execute Query** を選択して既定クエリを実行してください。

    ```
    SELECT * FROM c
    ```

1. クエリ結果を確認してください。**Query Stats** を選択し、RU で要求ユニット課金を確認してください。

1. エディター領域の内容を削除してください。

1. **name** が **HL Headset** に等しいすべてのドキュメントを返す新しい SQL クエリを作成してください。

    ```
    SELECT
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ```

1. **Execute Query** を選択してください。

1. クエリ結果と統計を確認してください。要求ユニット課金は最初のクエリとほぼ同じです。

1. エディター領域の内容を削除してください。

1. **name** が **HL Headset** に等しいすべてのドキュメントを返す新しい SQL クエリを作成してください。

    ```
    SELECT
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC
    ```

1. **Execute Query** を選択してください。

1. クエリ結果と統計を確認してください。**ORDER BY** 句により要求ユニット課金が増加しています。

### タスク 4: インデックス作成ポリシーに複合インデックスを作成する

複数プロパティを使用してアイテムを並べ替える場合は、複合インデックスを作成する必要があります。このタスクでは、categoryName で並べ替えた後に実際の name で並べ替えるための複合インデックスを作成します。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを選択して、**New SQL Query** を選択してください。

1. エディター領域の内容を削除してください。

1. まず **categoryName** の降順、次に **price** の昇順で結果を並べ替える新しい SQL クエリを作成してください。

    ```
    SELECT
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.price ASC
    ```

1. **Execute Query** を選択してください。

1. クエリは **The order by query does not have a corresponding composite index that it can be served from** というエラーで失敗するはずです。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを展開して、**Settings** を選択してください。

1. **Settings** タブで **Indexing Policy** セクションに移動してください。

1. 既定のインデックス作成ポリシーを確認してください。

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [
        {
          "path": "/\"_etag\"/?"
        }
      ]
    }
    ```

1. インデックス作成ポリシーを次の変更済み JSON オブジェクトに置き換え、変更を **Save** してください。

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを選択して、**New SQL Query** を選択してください。

1. エディター領域の内容を削除してください。

1. まず **categoryName** の降順、次に **price** の昇順で結果を並べ替える新しい SQL クエリを作成してください。

    ```
    SELECT
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.price ASC
    ```

1. **Execute Query** を選択してください。

1. クエリ結果と統計を確認してください。複合インデックスが適用されたため、要求ユニット課金は小さくなるはずです。

1. エディター領域の内容を削除してください。

1. まず **categoryName** の降順、次に **name** の昇順、最後に **price** の昇順で結果を並べ替える新しい SQL クエリを作成してください。

    ```
    SELECT
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.name ASC,
        p.price ASC
    ```

1. **Execute Query** を選択してください。

1. クエリは **The order by query does not have a corresponding composite index that it can be served from** というエラーで失敗するはずです。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを展開して、再度 **Settings** を選択してください。

1. **Settings** タブで **Indexing Policy** セクションに移動してください。

1. インデックス作成ポリシーを次の変更済み JSON オブジェクトに置き換え、変更を **Save** してください。

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/*"
        }
      ],
      "excludedPaths": [],
      "compositeIndexes": [
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ],
        [
          {
            "path": "/categoryName",
            "order": "descending"
          },
          {
            "path": "/name",
            "order": "ascending"
          },
          {
            "path": "/price",
            "order": "ascending"
          }
        ]
      ]
    }
    ```

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを選択して、**New SQL Query** を選択してください。

1. エディター領域の内容を削除してください。

1. まず **categoryName** の降順、次に **name** の昇順、最後に **price** の昇順で結果を並べ替える新しい SQL クエリを作成してください。

    ```
    SELECT
        p.name,
        p.categoryName,
        p.price
    FROM
        products p
    ORDER BY
        p.categoryName DESC,
        p.name ASC,
        p.price ASC
    ```

1. **Execute Query** を選択してください。

1. クエリ結果と統計を確認してください。複合インデックスが適用されたため、要求ユニット課金は低くなるはずです。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

### レビュー

このラボでは、次を完了しました。

- Azure Cosmos DB for NoSQL アカウントを作成した。
- Azure Cosmos DB for NoSQL アカウントにサンプル データをシードした。
- SQL クエリを実行し、要求ユニット課金を測定した。
- インデックス作成ポリシーに複合インデックスを作成した。

### ラボは正常に完了しました