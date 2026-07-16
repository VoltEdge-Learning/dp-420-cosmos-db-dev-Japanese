---
lab:
    title: 'Azure Data Factory を使用して既存データを移行する'
    module: 'モジュール 2 - Azure Cosmos DB SQL API を計画および実装する'
---

# Azure Data Factory を使用して既存データを移行する

Azure Data Factory では、Azure Cosmos DB はデータ取り込みのソースとしても、データ出力のターゲット（シンク）としてもサポートされています。

このラボでは、便利なコマンドライン ユーティリティを使用して Azure Cosmos DB にデータを投入し、その後 Azure Data Factory を使用してデータのサブセットを 1 つのコンテナーから別のコンテナーへ移動します。

## Azure Cosmos DB SQL API アカウントを作成してシードする

**4,000** 要求ユニット/秒（RU/s）で **cosmicworks** データベースと **products** コンテナーを作成するコマンドライン ユーティリティを使用します。作成後、スループットを 400 RU/s に下げて調整します。

products コンテナーに対応させるため、このラボの最後に ETL 変換および読み込み操作のターゲットとなる **flatproducts** コンテナーを手動で作成します。

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインします。

1. **+ Create a resource** を選択し、*Cosmos DB* を検索してから、次の設定で新しい **Azure Cosmos DB SQL API** アカウント リソースを作成します。残りのすべての設定は既定値のままにします。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Subscription** | *既存の Azure サブスクリプション* |
    | **Resource group** | *既存のリソース グループを選択するか、新しく作成する* |
    | **Account Name** | *グローバルに一意の名前を入力する* |
    | **Location** | *利用可能な任意のリージョンを選択する* |
    | **Capacity mode** | *プロビジョニングされたスループット* |
    | **Apply Free Tier Discount** | *適用しない* |
    | **Limit the total amount of throughput that can be provisioned on this account** | *オフ* |

    > &#128221; ラボ環境には、新しいリソース グループの作成を制限する制約がある場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. このタスクを続行する前に、デプロイ タスクが完了するまで待ちます。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Keys** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録します。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録します。この演習の後半でこの **key** 値を使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスにまだ慣れていない場合は、[Visual Studio Code の Get Started ガイド][code.visualstudio.com/docs/getstarted] を確認してください。

1. **Visual Studio Code** で **Terminal** メニューを開き、**New Terminal** を選択して新しいターミナル インスタンスを開きます。

1. マシン全体で使用できるように、[cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをインストールします。

    ```
    dotnet tool install --global cosmicworks
    ```

    > &#128161; このコマンドは完了までに数分かかる場合があります。過去にこのツールの最新バージョンをすでにインストールしている場合、このコマンドは警告メッセージ（*Tool 'cosmicworks' is already installed'）を出力します。

1. 次のコマンドライン オプションで cosmicworks を実行し、Azure Cosmos DB アカウントにデータをシードします。

    | **Option** | **Value** |
    | ---: | :--- |
    | **--endpoint** | *このラボで先ほどコピーした endpoint 値* |
    | **--key** | *このラボで先ほどコピーした key 値* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; たとえば、endpoint が **https&shy;://dp420.documents.azure.com:443/** で、key が **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。
    > ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. **cosmicworks** コマンドが、データベース、コンテナー、アイテムをアカウントに投入し終えるまで待ちます。

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインします。

1. **Resource groups** を選択し、次にこのラボで先ほど作成または確認したリソース グループを選択し、さらにこのラボで作成した **Azure Cosmos DB account** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、**Data Explorer** ペインに移動します。

1. **Data Explorer** で、**cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを展開してから、**Items** を選択します。

1. **products** コンテナー内のさまざまな JSON アイテムを確認して選択します。これらは前の手順で使用したコマンドライン ツールによって作成されたアイテムです。

1. **Scale & Settings** ノードを選択します。**Scale & Settings** タブで **Manual** を選択し、**required throughput** 設定を **4000 RU/s** から **400 RU/s** に更新してから、変更を **Save** します**。

1. **Data Explorer** ペインで **New Container** を選択します。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択します。

    | **Setting** | **Value** |
    | --: | :-- |
    | **Database id** | *既存を使用* &vert; *cosmicworks* |
    | **Container id** | *`flatproducts`* |
    | **Partition key** | *`/category`* |
    | **Container throughput (autoscale)** | *Manual* |
    | **RU/s** | *`400`* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **flatproducts** コンテナー ノードを確認します。

1. Azure portal の **Home** に戻ります。

## Azure Data Factory リソースを作成する

Azure Cosmos DB SQL API リソースの準備が整ったので、Azure Data Factory リソースを作成し、必要なすべてのコンポーネントと接続を構成して、1 回限りのデータ移動を 1 つの SQL API コンテナーから別の SQL API コンテナーへ実行し、データを抽出、変換し、別の SQL API コンテナーへ読み込みます。

1. **+ Create a resource** を選択し、*Data Factory* を検索してから、次の設定で新しい **Azure Data Factory** リソースを作成します。残りのすべての設定は既定値のままにします。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Subscription** | *既存の Azure サブスクリプション* |
    | **Resource group** | *既存のリソース グループを選択するか、新しく作成する* |
    | **Name** | *グローバルに一意の名前を入力する* |
    | **Region** | *利用可能な任意のリージョンを選択する* |
    | **Version** | *V2* |
    | **Git configuration** | *後で Git を構成する* |

    > &#128221; ラボ環境には、新しいリソース グループの作成を制限する制約がある場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. このタスクを続行する前に、デプロイ タスクが完了するまで待ちます。

1. 新しく作成した **Azure Data Factory** リソースに移動し、**Open Azure Data Factory Studio** を選択します。

    > &#128161; あるいは、(``adf.azure.com/home``) に移動し、新しく作成した Data Factory リソースを選択してから、ホーム アイコンを選択することもできます。

1. ホーム画面から、**Ingest** オプションを選択して、1 回限りの大規模データ コピー操作を実行するクイック ウィザードを開始し、ウィザードの **Properties** ステップに進みます。

1. ウィザードの **Properties** ステップで、**Task type** セクションにある **Built-in copy task** を選択します。

1. **Task cadence or task schedule** セクションで **Run once now** を選択し、次に **Next** を選択してウィザードの **Source** ステップに進みます。

1. ウィザードの **Source** ステップで、**Source type** リストから **Azure Cosmos DB (SQL API)** を選択します。

1. **Connection** セクションで **+ New connection** を選択します。

1. **New connection (Azure Cosmos DB (SQL API))** ポップアップで、次の値を使用して新しい接続を構成し、**Create** を選択します。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Name** | *`CosmosSqlConn`* |
    | **Connect via integration runtime** | *AutoResolveIntegrationRuntime* |
    | **Authentication method** | *Account key* &vert; *Connection string* |
    | **Account selection method** | *From Azure subscription* |
    | **Azure subscription** | *既存の Azure サブスクリプション* |
    | **Azure Cosmos DB account name** | *このラボで先ほど選択した既存の Azure Cosmos DB アカウント名* |
    | **Database name** | *cosmicworks* |

1. **Source data store** セクションに戻り、**Source tables** セクション内で **Use query** を選択します。

1. **Table name** リストで **products** を選択します。

1. **Query** エディターで既存の内容を削除し、次のクエリを入力します。

    ```
    SELECT 
        p.name, 
        p.categoryName as category, 
        p.price 
    FROM 
        products p
    ```

1. **Preview data** を選択してクエリの有効性をテストします。**Next** を選択してウィザードの **Target** ステップに進みます。

1. ウィザードの **Target** ステップで、**Target type** リストから **Azure Cosmos DB (SQL API)** を選択します。

1. **Connection** リストで **CosmosSqlConn** を選択します。

1. **Target** リストで **flatproducts** を選択し、次に **Next** を選択してウィザードの **Settings** ステップに進みます。

1. ウィザードの **Settings** ステップで、**Task name** フィールドに **`FlattenAndMoveData`** と入力します。

1. 残りのすべてのフィールドは既定の空白値のままにし、**Next** を選択してウィザードの最終ステップに進みます。

1. ウィザードで選択した手順の **Summary** を確認してから、**Next** を選択します。

1. デプロイのさまざまなステップを確認します。デプロイが完了したら、**Finish** を選択します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインします。

1. **Resource groups** を選択し、次にこのラボで先ほど作成または確認したリソース グループを選択し、さらにこのラボで作成した **Azure Cosmos DB account** リソースを選択します。

1. **Azure Cosmos DB** アカウント リソース内で、**Data Explorer** ペインに移動します。

1. **Data Explorer** で、**cosmicworks** データベース ノードを展開し、**flatproducts** コンテナー ノードを選択してから、**New SQL Query** を選択します。

1. エディター領域の内容を削除します。

1. **name** が **HL Headset** と等しいすべてのドキュメントを返す新しい SQL クエリを作成します。

    ```
    SELECT 
        p.name, 
        p.category, 
        p.price 
    FROM
        products p
    WHERE
        p.name = 'HL Headset'
    ```

1. **Execute Query** を選択します。

1. クエリの結果を確認します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks
