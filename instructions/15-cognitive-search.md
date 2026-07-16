# Azure AI Search と Azure Cosmos DB for NoSQL を使用してデータを検索する

## ラボ シナリオ

Azure Cognitive Search は、サービスとしての検索エンジンと、AI 機能との高度な統合を組み合わせることで、検索インデックス内の情報を強化します。

このラボでは、Azure Cosmos DB SQL API コンテナー内のデータを自動的にインデックス化し、Azure Cognitive Services Translator 機能を使用してデータを強化する Azure Cognitive Search インデックスを構築します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: Azure Cosmos DB for NoSQL アカウントを作成する。
- タスク 2: Azure Cosmos DB for NoSQL アカウントにサンプル データを送信する。
- タスク 3: Azure AI Search リソースを作成する。
- タスク 4: Azure Cosmos DB for NoSQL データ用のインデクサーとインデックスを構築する。
- タスク 5: 例の検索クエリでインデックスを検証する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab15.png)

### タスク 1: Azure Cosmos DB for NoSQL アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングする際は、アカウントでサポートする API（例: **Mongo API** または **NoSQL API**）を選択します。Azure Cosmos DB for NoSQL アカウントのプロビジョニング完了後、エンドポイントとキーを取得し、Azure SDK for .NET または任意の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続できます。

1. Azure Portal ページに戻り、ポータル上部の Search resources, services and docs (G+/) ボックスに **Azure Cosmos DB** と入力して、services の **Azure Cosmos DB** を選択します。

   ![06](media/New-image1.png)

1. **Azure Cosmos DB for NoSQL** の下にある **+ Create** を選択し、**Create** をクリックして **Azure Cosmos DB for NoSQL** アカウントを作成します。

    ![06](media/New-image2.png)

    ![06](media/New-image3.png)

1. 次の設定でリソースを作成し、その他の設定は既定値のままにして **Review + create** **(7)** を選択します。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Workload Type** | Production **(1)** |
    | **Subscription** | Select your Azure subscription **(2)** |
    | **Resource Group** | Select existing resource group **Cosmosdb-<inject key="DeploymentID" enableCopy="false"/> (3)** |
    | **Account Name** | Enter **sql-<inject key="DeploymentID" enableCopy="false"/> (4)** |
    | **Location** | Select any available location **(5)** |
    | **Capacity mode** | Serverless **(6)** |

    ![06](media/DB51.png)

1. 設定を確認し、**Create** をクリックします。

     ![06](media/DB52.png)

1. このタスクを続行する前に、デプロイの完了を待ちます。

1. **Go to resources** を選択します。新しく作成した **Azure Cosmos DB** アカウントの **Settings** で **Keys** ペインに移動します。

    ![06](media/New-image6.png)

    ![06](media/New-image7.png)

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    - **URI (1)** フィールドの値を記録します。この演習の後半でこの **endpoint** 値を使用します。

    - **PRIMARY KEY (2)** フィールドの値を記録します。この演習の後半でこの **key** 値を使用します。

        ![06](media/New-image9.png)

1. **Azure Cosmos DB** アカウント リソースの **overview page (1)** で、**Data Explorer (2)** ペインに移動します。

    ![06](media/DB04.png)

1. **Data Explorer (1)** ページで **New (2)** をクリックし、**New Container (3)** を選択します。

     ![06](media/DB42.png)

1. **New Container** ペインで、次の詳細を入力します。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Database id** | `cosmicworks` **(1)** |
    | **Container id** | `products` **(2)** |
    | **Partition key** | `/categoryId` **(3)** |

    ![06](media/DB044.png)

1. **OK (4)** をクリックします。

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認します。

    ![06](media/cosmosdbproducts.png)

    > **Congratulations** タスク完了です。次に検証を行います。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押します。成功メッセージが表示されたら次のタスクに進みます。
    > - 成功しない場合は、エラー メッセージをよく読み、ラボ ガイドに従って手順を再実行します。
    > - 支援が必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

    <validation step="527d4167-c9f9-49aa-9489-c384ed66b37f" />

### タスク 2: Azure Cosmos DB for NoSQL アカウントにサンプル データを送信する

**cosmicworks** データベースと **products** コンテナーを作成するコマンドライン ユーティリティを使用します。次にツールが項目セットを作成し、ターミナル ウィンドウで実行中の変更フィード プロセッサを使用してその内容を確認します。

1. Visual Studio Code を起動します（プログラム アイコンはデスクトップにピン留めされています）。

   ![Visual Studio Code Icon](./media/vscode1.jpg)

1. **Visual Studio Code** で、**... (ellipses) (1)** を選択して **Terminal (2)** を選び、**New Terminal (3)** を選択して既存のインスタンスと並べて新しいターミナルを開きます。

    ![06](media/New-image36.png)

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをマシン全体で利用できるようにインストールします。

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    ![06](media/DB50.png)

    >**Note:** このコマンドの完了には数分かかる場合があります。過去にこのツールの最新バージョンをすでにインストールしている場合、このコマンドは警告メッセージ（*Tool 'cosmicworks' is already installed'）を出力します。

1. インストール完了後、次のコマンドを実行する前に **Visual Studio Code** を一度閉じて再度開いてください。

1. 次のコマンドライン オプションで cosmicworks を実行し、Azure Cosmos DB アカウントにデータを投入します。

    | **Option** | **Value** |
    | :--- | :--- |
    | **--endpoint** | *このラボで先ほどコピーした endpoint 値* |
    | **--key** | *このラボで先ほどコピーした key 値* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > **For example:** endpoint が **https&shy;://dp420.documents.azure.com:443/**、key が **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。
    > ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

    >**Note**: エラーが発生する場合は、Visual Studio Code を閉じて再度開き、もう一度コマンドを実行してください。

    > **Note:** それでもエラーが発生する場合は、次の手順を実行してからコマンドを再実行してください。

    1. Azure portal で **Cosmos DB account** を開き、**Settings (1)** を展開して **Networking (2)** を選択します。

    1. **Firewall** で **Add your current IP (3)** をクリックするか、IP アドレスを手動入力します。

        ![06](media/DB60.png)

    1. **Save (4)** をクリックして変更を適用します。

        ![06](media/DB61.png)

    1. ファイアウォール ルールが有効になるまで数分待機します。

    1. **Cosmos DB account** で **Settings (1)** を展開し、**Keys (2)** を選択します。

    1. **Primary Connection String (4)** をコピーし、必要に応じて **Show/Hide (3)** を使用して値を表示します。

     ![06](media/DB63.png)

    ```
    cosmicworks --connection-string "<your-connection-string>" --datasets product
    ```

1. **cosmicworks** コマンドがデータベース、コンテナー、項目の投入を完了するまで待機します。

1. 統合ターミナルを閉じて、**Visual Studio Code** も閉じます。

### タスク 3: Azure AI Search リソースを作成する

この演習を続行する前に、まず新しい Azure Cognitive Search インスタンスを作成する必要があります。

1. Azure portal に戻り、**Create a resource** をクリックします。

     ![06](media/DB53.png)

1. **Azure AI Search (1)** を検索し、**Create (2)** を選択して **Azure AI Search (3)** を選びます。

    ![06](media/DB55.png)

1. 次の詳細を入力します。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Subscription** | Select your Azure subscription **(1)** |
    | **Resource group** | Select **cosmosdb-<inject key="DeploymentID" enableCopy="false"/>** **(2)** |
    | **Service name** | Enter **aisearch-<inject key="DeploymentID" enableCopy="false"/>** **(3)** |
    | **Location** | Use the default location **(4)** |

1. **Review + create (5)** をクリックします。

    ![06](media/DB56.png)

    >**Note:** サブスクリプション エラーが表示される場合は、**Next: Scale** を選択してから **Previous** を選択してください。

1. 検証が成功したら設定を確認し、**Create** をクリックします。

    ![06](media/DB57.png)

4. デプロイ完了後、**Go to resource** をクリックして新しく作成した **Azure AI Search** アカウント リソースに移動します。

    ![06](media/DB58.png)

> **Congratulations** タスク完了です。次に検証を行います。手順は次のとおりです。
> - 対応するタスクの Validate ボタンを押します。成功メッセージが表示されたら次のタスクに進みます。
> - 成功しない場合は、エラー メッセージをよく読み、ラボ ガイドに従って手順を再実行します。
> - 支援が必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

<validation step="6dc66a44-b1ce-4dc5-91f8-35180452aaaa" />

### タスク 4: Azure Cosmos DB for NoSQL データ用のインデクサーとインデックスを構築する

特定の Azure Cosmos DB for NoSQL コンテナー内のデータのサブセットを 1 時間ごとにインデックス化するインデクサーを作成します。

1. **AI Search** リソース ブレードで **Import data** を選択します。

    ![06](media/importdata.png)

1. **Import data** ウィザードの **Data Source** リストで **Azure Cosmos DB** を選択します。

    ![06](media/DB59.png)

1. 次の設定でデータ ソースを構成し、その他の設定は既定値のままにします。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Data source name** | **products-cosmossql-source (2)** |
    | **Connection string** | **Choose an existing connection of the Azure Cosmos DB for NoSQL account created earlier (3)** |
    | **Database** | **cosmicworks (4)** |
    | **Collection** | **products (5)** |

    ![06](media/importyourdata.png)

1. **query** フィールドに次の SQL クエリを入力し、コンテナー内データのサブセットのマテリアライズド ビューを作成します。

    ```SQL
    SELECT
        p.id,
        p.categoryId,
        p.name,
        p.price,
        p._ts
    FROM
        products p
    WHERE
        p._ts > @HighWaterMark
    ORDER BY
        p._ts
    ```

1. **Query results ordered by _ts** チェック ボックスを選択します。

    >**Note:** このチェック ボックスは、クエリ結果が **_ts** フィールドでソートされることを Azure AI Search に通知します。この種のソートにより増分進行の追跡が可能になります。インデクサーが失敗した場合でも、結果がタイムスタンプ順に並んでいるため、同じ **_ts** 値から再開できます。

1. **Next: Add cognitive skills** を選択します。

1. **Skip to: Customize target index** を選択します。

1. ウィザードの **Customize target index** ステップで、次の設定でインデックスを構成し、その他の設定は既定値のままにします。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Index name** | *``products-index``* |
    | **Key** | *id* |

1. フィールド テーブルで、次の表に従って各フィールドの **Retrievable**、**Filterable**、**Sortable**、**Facetable**、**Searchable** オプションを設定します。

    ![06](media/categoryid.png)

1. **Next: Create an indexer** を選択します。

1. ウィザードの **Create an indexer** ステップで、次の設定でインデクサーを構成し、その他の設定は既定値のままにします。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Name** | *``products-cosmosdb-indexer``* |
    | **Schedule** | *Hourly* |

1. **Submit** を選択して、データ ソース、インデックス、インデクサーを作成します。

    >**Note:** 最初のインデクサー作成後、アンケートのポップアップを閉じる必要がある場合があります。

1. **AI Search** リソース ブレードの左ナビゲーション メニューで、**Search Management** タブ配下の **Indexers (1)** を選択し、最初のインデックス処理結果を確認します。

1. **products-cosmosdb-indexer** のステータスが **Success (2)** になるまで待ってから、このタスクを続行します。

    >**Note:** 自動更新されない場合は、ブレードを更新するために **Refresh** を使用する必要があります。

    ![06](media/indexers.png)

1. 左ナビゲーション ペインの **Search Management** 配下で **Indexes** タブに移動し、**products-index** インデックスを選択します。

> **Congratulations** タスク完了です。次に検証を行います。手順は次のとおりです。
> - 対応するタスクの Validate ボタンを押します。成功メッセージが表示されたら次のタスクに進みます。
> - 成功しない場合は、エラー メッセージをよく読み、ラボ ガイドに従って手順を再実行します。
> - 支援が必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

<validation step="0e28166a-b18c-4b9b-8f4b-0b4d113890bc" />

### タスク 5: 例の検索クエリでインデックスを検証する

Azure Cosmos DB for NoSQL データのマテリアライズド ビューが検索インデックスに作成されたので、Azure AI Search の機能を活用した基本的なクエリをいくつか実行できます。

> **Note:** このラボは Azure AI Search 構文そのものを学習することを目的としていません。ここでのクエリは、検索インデックスとエンジンで利用可能な機能の一部を示すために厳選されています。

1. **Search explorer** タブで **View** のプルダウンを選択し、**JSON view** を選択します。

1. **JSON query editor** で、**\***（ワイルドカード）演算子を使用してすべての結果を返す既定の JSON 検索クエリ構文を確認します。

   ```json
   {
       "search": "*"
   }
   ```

1. **Search** ボタンを選択して検索を実行します。

1. この検索クエリがすべての結果を返すことを確認します。

1. **JSON query editor** で次のクエリを入力し、**Search** を選択します。

    ```json
    {
        "search": "touring 3000"
    }
    ```

1. このクエリは **touring** または **3000** のいずれかを含む結果を返し、両方を含む結果にはより高いスコアが付与されることを確認します。結果はその後、**@search.score** フィールドの降順で並び替えられます。

1. **JSON query editor** で次のクエリを入力し、**Search** を選択します。

    ```json
    {
        "search": "red"
        , "count": true
    }
    ```

1. このクエリは **red** を含む結果を返し、同時に、同じページにすべて含まれていなくても結果総数を示すメタデータ フィールドが含まれることを確認します。

1. **JSON query editor** で次のクエリを入力し、**Search** を選択します。

    ```json
    {
        "search": "blue"
        , "count": true
        , "top": 6
    }
    ```

1. サーバー側にはさらに一致結果がある場合でも、このクエリが一度に 6 件だけ返すことを確認します。

1. **JSON query editor** で次のクエリを入力し、**Search** を選択します。

    ```json
    {
        "search": "mountain"
        , "count": true
        , "top": 25
        , "skip": 50
    }
    ```

1. このクエリが最初の 50 件をスキップし、25 件を返すことを確認します。これがクライアント側アプリケーションのページング表示であれば、結果の 3 ページ目に相当すると推測できます。

1. **JSON query editor** で次のクエリを入力し、**Search** を選択します。

    ```json
    {
        "search": "touring"
        , "count": true
        , "filter": "price lt 500"
    }
    ```

1. このクエリが、数値 price フィールドの値が 500 未満の結果のみを返すことを確認します。

1. **JSON query editor** で次のクエリを入力し、**Search** を選択します。

    ```json
    {
        "search": "road"
        , "count": true
        , "top": 15
        , "facets": ["price,interval:500"]
    }
    ```

1. このクエリが、現在の結果ページにすべてが表示されていない場合でも、各カテゴリに属する項目数を示すファセット データのコレクションを返すことを確認します。この例では、一致した項目は 500 間隔の数値 price カテゴリに分解されます。これは一般に、クライアント側アプリケーションでフィルターやナビゲーション補助を構成するために使用されます。

    ![06](media/products-index.png)

1. Web ブラウザーのウィンドウまたはタブを閉じます。

### レビュー

このラボでは、次を完了しました。

- Azure Cosmos DB for NoSQL アカウントを作成した。
- Azure Cosmos DB for NoSQL アカウントにサンプル データを送信した。
- Azure AI Search リソースを作成した。
- Azure Cosmos DB for NoSQL データ用のインデクサーとインデックスを構築した。
- 例の検索クエリでインデックスを検証した。

### まとめ

このラボでは、Azure AI Search と Azure Cosmos DB for NoSQL の統合手順を学習します。Cosmos DB アカウントの作成、サンプル データの投入、Azure Cognitive Search リソースの設定、インデクサーの構築を実施します。

### ラボは正常に完了しました