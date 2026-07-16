# Lab 02a - Azure Cosmos DB for NoSQL を計画して実装する

## ラボ シナリオ

Azure Cosmos DB for NoSQL で最も重要なポイントの 1 つは、スループット構成を理解することです。Azure Cosmos DB for NoSQL コンテナーを作成するには、順番として最初にアカウントを作成し、次にデータベースを作成する必要があります。
このラボでは、Data Explorer のさまざまな方法を使用してスループットをプロビジョニングします。データベース レベルとコンテナー レベルの両方で、手動またはオートスケールを使用してスループットをプロビジョニングします。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: サーバーレス アカウントを作成する。
- タスク 2: プロビジョニング済みアカウントを作成する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab2.png)

## 演習 1: Azure portal で Azure Cosmos DB for NoSQL のスループットを構成する

### タスク 1: サーバーレス アカウントを作成する

まずはシンプルにサーバーレス アカウントを作成します。ここではすべてがサーバーレスのため、構成する項目は多くありません。データベースとコンテナーを作成する際に、スループットをプロビジョニングする必要はありません。このアカウントの作成手順でそれを確認します。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Azure services** カテゴリ内で **Create a resource** を選択し、次に **Azure Cosmos DB** を選択してください。

    > &#128161; 別の方法として、**&#8801;** メニューを展開し、**All Services** を選択して、**Databases** カテゴリ内の **Azure Cosmos DB** を選択し、**Create** を選択してください。

1. **Select API option** ペインで、**Azure Cosmos DB for NoSQL** セクション内の **Create** オプションを選択してください。

1. **Create Azure Cosmos DB Account** ペインで **Basics** タブを確認してください。

1. **Basics** タブで、各設定に次の値を入力してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Subscription** | *Your existing Azure subscription* |
    | **Resource Group** | *Select an existing resource group.* |
    | **Account Name** | *Enter a globally unique name* |
    | **Location** | *Choose any available region* |
    | **Capacity mode** | *Select Serverless* |

1. **Review + Create** を選択して **Review + Create** タブに移動し、次に **Create** を選択してください。

    > &#128221; Azure Cosmos DB for NoSQL アカウントが使用可能になるまでに 10〜15 分かかる場合があります。

1. **Deployment** ペインを確認してください。デプロイが完了すると、ペインは **Deployment successful** メッセージで更新されます。

1. 引き続き **Deployment** ペイン内で **Go to resource** を選択してください。

1. **Azure Cosmos DB account** ペイン内で、リソース メニューから **Data Explorer** を選択してください。

1. **Data Explorer** ペインで **New Container** を展開し、**New Database** を選択してください。

1. **New Database** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Database id** | *`cosmicworks`* |

1. **Data Explorer** ペインに戻り、階層内の **cosmicworks** データベース ノードを確認してください。

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Database id** | *Use existing* &vert; *cosmicworks* |
    | **Container id** | *`products`* |
    | **Partition key** | *`/categoryId`* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認してください。

1. Azure portal の **Home** に戻ってください。

### タスク 2: プロビジョニング済みアカウントを作成する

次に、より一般的な構成オプションを持つプロビジョニング済みスループット アカウントを作成します。この種類のアカウントでは多くの構成オプションが利用できるため、やや複雑に感じる場合があります。ここでは、可能なデータベースとコンテナーの組み合わせ例をいくつか確認します。

1. **Azure services** カテゴリ内で **Create a resource** を選択し、次に **Azure Cosmos DB** を選択してください。

    > &#128161; 別の方法として、**&#8801;** メニューを展開し、**All Services** を選択して、**Databases** カテゴリ内の **Azure Cosmos DB** を選択し、**Create** を選択してください。

1. **Select API option** ペインで、**Azure Cosmos DB for NoSQL** セクション内の **Create** オプションを選択してください。

1. **Create Azure Cosmos DB Account** ペインで **Basics** タブを確認してください。

1. **Basics** タブで、各設定に次の値を入力してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Subscription** | *Your existing Azure subscription* |
    | **Resource Group** | *Select an existing resource group.* |
    | **Account Name** | *cosmosdb420-XXXXXX* |
    | **Location** | *Choose any available region* |
    | **Capacity mode** | *Select provisioned throughput* |
    | **Apply Free Tier Discount** | *Do Not Apply* |
    | **Limit the total amount of throughput that can be provisioned on this account** | *Unchecked* |
    
    >**Note**: XXXXXX は、環境の詳細ページで提供される DeploymentID の値に置き換えてください。

1. **Review + Create** を選択して **Review + Create** タブに移動し、次に **Create** を選択してください。

    > &#128221; Azure Cosmos DB for NoSQL アカウントが使用可能になるまでに 10〜15 分かかる場合があります。

1. **Deployment** ペインを確認してください。デプロイが完了すると、ペインは **Deployment successful** メッセージで更新されます。

1. 引き続き **Deployment** ペイン内で **Go to resource** を選択してください。

1. **Azure Cosmos DB account** ペイン内で、リソース メニューから **Data Explorer** を選択してください。

1. **Data Explorer** ペインで **New Container** を展開し、**New Database** を選択してください。

1. **New Database** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Database id** | *`nothroughputdb`* |
    | **Provision throughput** | *Do not select* |

1. **Data Explorer** ペインに戻り、階層内の **nothroughputdb** データベース ノードを確認してください。

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Database id** | *Use existing* &vert; *nothroughputdb* |
    | **Container id** | *`requiredthroughputcontainer`* |
    | **Partition key** | *`/primarykey`* |
    | **Container throughput** | *Manual* |
    | **RU/s** | *`400`* |

1. **Data Explorer** ペインに戻り、**nothroughputdb** データベース ノードを展開して、階層内の **requiredthroughputcontainer** コンテナー ノードを確認してください。

1. **Data Explorer** ペインで **New Container** を展開し、**New Database** を選択してください。

1. **New Database** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Database id** | *`manualthroughputdb`* |
    | **Provision throughput** | *Select this option* |
    | **Database throughput** | *Manual* |
    | **RU/s** | *`400`* |

1. **Data Explorer** ペインに戻り、階層内の **manualthroughputdb** データベース ノードを確認してください。

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Database id** | *Use existing* &vert; *manualthroughputdb* |
    | **Container id** | *`childcontainer`* |
    | **Partition key** | *`/primarykey`* |
    | **Provision dedicated throughput for this container** | *Select this option* |
    | **Container throughput** | *Manual* |
    | **RU/s** | *`1000`* |

1. **Data Explorer** ペインに戻り、**manualthroughputdb** データベース ノードを展開して、階層内の **childcontainer** コンテナー ノードを確認してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Keys** ペインに移動してください。

1. このペインには、SDK からアカウントへ接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

### レビュー

このラボでは、次を完了しました。

- サーバーレス アカウントを作成した。
- プロビジョニング済みアカウントを作成した。

### ラボは正常に完了しました