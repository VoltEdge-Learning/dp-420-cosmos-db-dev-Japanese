# Lab 01 - Azure Cosmos DB SQL API を使い始める

## ラボ シナリオ

Azure Cosmos DB を深く学ぶ前に、最もよく使用するリソースの作成に関する基本を理解しておくことが重要です。ほとんどのシナリオでは、アカウント、データベース、コンテナー、項目を作成できることが必要になります。実運用のシナリオでは、すべてのリソースが正しく作成されたことを確認するために、いくつかの基本クエリを手元に用意しておくことも重要です。

このラボでは、SQL API を使用して新しい Azure Cosmos DB アカウントを作成します。次に Data Explorer を使用してデータベース、コンテナー、および 2 つの項目を作成します。最後に、作成した項目をデータベースに対してクエリします。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 新しい Azure Cosmos DB アカウントを作成する。
- タスク 2: Data Explorer を使用して新しいデータベースとコンテナーを作成する。
- タスク 3: Data Explorer を使用して新しい項目を作成する。
- タスク 4: Data Explorer を使用して基本クエリを実行する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab1.png)

## 演習 1: Azure Cosmos DB SQL API アカウントを作成する

### タスク 1: 新しい Azure Cosmos DB アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。Azure Cosmos DB アカウントを初めてプロビジョニングする際には、アカウントでサポートする API（例: **Mongo API** または **SQL API**）を選択します。

1. 新しい Web ブラウザー ウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Azure services** カテゴリ内で **Create a resource** を選択し、次に **Azure Cosmos DB** を選択してください。

    > &#128161; 別の方法として、**&#8801;** メニューを展開し、**All Services** を選択して、**Databases** カテゴリ内の **Azure Cosmos DB** を選択し、**Create** を選択してください。

1. **Select API option** ペインで、**Azure Cosmos DB for NoSQL** セクション内の **Create** オプションを選択してください。

1. **Create Azure Cosmos DB Account** ペインで **Basics** タブを確認してください。

1. **Basics** タブで、各設定に次の値を入力してください。

    | **Setting** | **Value** |
    | --: | :-- |
    | **Subscription** | *Your existing Azure subscription* |
    | **Resource Group** | *DP-420-DeploymentID* |
    | **Account Name** | *Enter a globally unique name* |
    | **Location** | *Choose any available region* |
    | **Capacity mode** | *Select provisioned throughput* |
    | **Apply Free Tier Discount** | *Do Not Apply* |

    >**Note** : DeploymentID は各環境に紐づく一意の ID です。この値は環境の詳細ページで確認できます。

1. **Review + Create** を選択して **Review + Create** タブに移動し、次に **Create** を選択してください。

    > &#128221; Azure Cosmos DB SQL API アカウントが使用可能になるまでに 10〜15 分かかる場合があります。

1. **Deployment** ペインを確認してください。デプロイが完了すると、ペインは **Deployment successful** メッセージで更新されます。

1. 引き続き **Deployment** ペイン内で **Go to resource** を選択してください。

### タスク 2: Data Explorer を使用して新しいデータベースとコンテナーを作成する

Data Explorer は、Azure portal で Azure Cosmos DB SQL API のデータベースとコンテナーを管理するための主要ツールです。このラボでは、基本的なデータベースとコンテナーを作成します。

1. **Azure Cosmos DB account** ペイン内で、リソース メニューから **Data Explorer** を選択してください。

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --: | :-- |
    | **Database id** | *cosmicworks* |
    | **Share throughput across containers** | *Do not select* |
    | **Container id** | *products* |
    | **Partition key** | */categoryId* |
    | **Container throughput (autoscale)** | *Manual* |
    | **RU/s** | *400* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開し、階層内の **products** コンテナー ノードを確認してください。

### タスク 3: Data Explorer を使用して新しい項目を作成する

Data Explorer には、Azure Cosmos DB SQL API コンテナー内の項目をクエリ、作成、管理するための機能群も含まれています。ここでは Data Explorer で生の JSON を使用して 2 つの基本項目を作成します。

1. **Data Explorer** ペインで、**cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを展開して、**Items** を選択してください。

1. 引き続き **Data Explorer** ペインで、コマンド バーから **New Item** を選択してください。エディターでプレースホルダーの JSON 項目を次の内容に置き換えてください。

    ```
    {
      "categoryId": "4F34E180-384D-42FC-AC10-FEC30227577F",
      "categoryName": "Components, Pedals",
      "sku": "PD-R563",
      "name": "ML Road Pedal",
      "price": 62.09
    }
    ```

1. コマンド バーから **Save** を選択し、最初の JSON 項目を追加してください。

1. **Items** タブに戻り、コマンド バーから **New Item** を選択してください。エディターでプレースホルダーの JSON 項目を次の内容に置き換えてください。

    ```
    {
      "categoryId": "75BF1ACB-168D-469C-9AA3-1FD26BB4EA4C",
      "categoryName": "Bikes, Touring Bikes",
      "sku": "BK-T18Y-44",
      "name": "Touring-3000 Yellow, 44",
      "price": 742.35
    }
    ```

1. コマンド バーから **Save** を選択し、2 つ目の JSON 項目を追加してください。

1. **Items** タブで、**Items** ペイン内に 2 つの新しい項目があることを確認してください。

### タスク 4: Data Explorer を使用して基本クエリを実行する

最後に、Data Explorer には組み込みのクエリ エディターがあり、クエリの実行、結果の確認、および 1 秒あたりの要求ユニット（RU/s）の観点で影響を測定できます。

1. **Data Explorer** ペインで **New SQL Query** を選択してください。

1. クエリ タブで **Execute Query** を選択し、フィルターなしで全項目を選択する標準クエリを表示してください。

1. エディター領域の内容を削除してください。

1. **Query** タブで、プレースホルダー クエリを次の内容に置き換えてください。

    ```
    SELECT * FROM products p WHERE p.price > 500
    ```

    > &#128221; このクエリは、**price** が $500 を超えるすべての項目を選択します。

1. **Execute Query** を選択してください。

1. クエリ結果を確認してください。結果には 1 つの JSON 項目とそのすべてのプロパティが含まれるはずです。

1. **Query** タブで **Query Stats** を選択してください。

1. 引き続き **Query** タブで、**Query Statistics** セクション内の **Request Charge** フィールドの値を確認してください。

    > &#128221; 一般的に、コンテナー サイズが小さい場合、この単純なクエリの要求課金は 2〜3 RU/s の範囲です。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

### レビュー

このラボでは、次を完了しました。

- 新しい Azure Cosmos DB アカウントを作成した。
- Data Explorer を使用して新しいデータベースとコンテナーを作成した。
- Data Explorer を使用して新しい項目を作成した。
- Data Explorer を使用して基本クエリを実行した。

### ラボは正常に完了しました