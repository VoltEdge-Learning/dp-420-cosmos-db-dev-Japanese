# Module 2 - Azure Cosmos DB SQL API を計画して実装する

## Azure Data Factory を使用して既存データを移行する

Azure Data Factory では、Azure Cosmos DB はデータ取り込みのソースとしても、データ出力のターゲット（シンク）としてもサポートされています。

このラボでは、便利なコマンドライン ユーティリティを使用して Azure Cosmos DB にデータを投入し、その後 Azure Data Factory を使用して、データのサブセットを 1 つのコンテナーから別のコンテナーへ移動します。

### ラボ 1: Azure Cosmos DB SQL API アカウントを作成してデータを投入する

コマンドライン ユーティリティを使用して、**4,000** request units per second（RU/s）で **cosmicworks** データベースと **products** コンテナーを作成します。作成後、スループットを 400 RU/s に下げます。

products コンテナーに対応して、このラボの最後に実施する ETL 変換およびロード操作のターゲットとして **flatproducts** コンテナーを手動で作成します。

1. **Visual Studio Code** を起動してください。

    > &#128221; Visual Studio Code のインターフェイスに不慣れな場合は、[Get Started guide for Visual Studio Code][code.visualstudio.com/docs/getstarted] を確認してください。

1. **Visual Studio Code** で **Terminal** メニューを開き、**New Terminal** を選択して新しいターミナルを開いてください。

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールを、マシンでグローバルに利用できるようにインストールしてください。

    ```
    dotnet tool install --global cosmicworks
    ```

    > &#128161; このコマンドの完了には数分かかる場合があります。過去にこのツールの最新バージョンをインストール済みであれば、警告メッセージ（*Tool 'cosmicworks' is already installed'*）が出力されます。

1. 次のコマンドライン オプションで cosmicworks を実行し、Azure Cosmos DB アカウントへデータを投入してください。

    | **Option** | **Value** |
    | ---: | :--- |
    | **--endpoint** | *The endpoint value you copied earlier in this lab* |
    | **--key** | *The key value you coped earlier in this lab* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; 例として、endpoint が **https&shy;://dp420.documents.azure.com:443/** で key が **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。
    > ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. **cosmicworks** コマンドによる、データベース、コンテナー、項目の投入が完了するまで待機してください。

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Resource groups** を選択し、このラボで先ほど作成または確認したリソース グループを選択してから、このラボで作成した **cosmosdb420-XXXXX** Azure Cosmos DB アカウント リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で **Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを展開して、**Items** を選択してください。

1. **products** コンテナー内の複数の JSON 項目を確認して選択してください。これらは前の手順で使用したコマンドライン ツールによって作成された項目です。

1. **Scale & Settings** ノードを選択してください。**Scale & Settings** タブで **Manual** を選択し、**required throughput** を **4000 RU/s** から **400 RU/s** に更新して、変更を **Save** してください。

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --: | :-- |
    | **Database id** | *Use existing* &vert; *cosmicworks* |
    | **Container id** | *`flatproducts`* |
    | **Partition key** | *`/category`* |
    | **Container throughput (autoscale)** | *Manual* |
    | **RU/s** | *`400`* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **flatproducts** コンテナー ノードを確認してください。

1. Azure portal の **Home** に戻ってください。

## Azure Data Factory リソースを作成する

Azure Cosmos DB SQL API リソースの準備ができたので、次に Azure Data Factory リソースを作成し、必要なコンポーネントと接続を構成して、1 つの SQL API コンテナーから別の SQL API コンテナーへ 1 回限りのデータ移動を実行します。これにより、データの抽出、変換、ロードを行います。

1. **+ Create a resource** を選択し、*Data Factory* を検索して、新しい **Azure Data Factory** リソースを作成してください。残りの設定は既定値のままにし、次の設定を入力してください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Subscription** | *Your existing Azure subscription* |
    | **Resource group** | *Select an existing resource group* |
    | **Name** | *Enter a globally unique name* |
    | **Region** | *Choose any available region* |
    | **Version** | *V2* |
   
1. **Next: Git configuration** をクリックしてください。**Git configuration** ブレードで *Configure Git later* を選択してください。

    > &#128221; ラボ環境では、新しいリソース グループの作成が制限されている場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. **Review + Create** を選択して **Review + Create** タブに移動し、次に **Create** を選択してください。

1. 新しく作成した **Azure Data Factory** リソースに移動し、**Open Azure Data Factory Studio** を選択してください。

    > &#128161; 別の方法として、(``adf.azure.com/home``) に移動し、新しく作成した Data Factory リソースを選択してからホーム アイコンを選択できます。

1. ホーム画面で **Ingest** オプションを選択し、1 回限りの大規模コピー操作を実行するクイック ウィザードを開始して、ウィザードの **Properties** ステップに進んでください。

1. ウィザードの **Properties** ステップで、**Task type** セクションの **Built-in copy task** を選択してください。

1. **Task cadence or task schedule** セクションで **Run once now** を選択し、**Next** を選択してウィザードの **Source** ステップに進んでください。

1. ウィザードの **Source** ステップで、**Source type** 一覧から **Azure Cosmos DB (SQL API)** を選択してください。

1. **Connection** セクションで **+ New connection** を選択してください。

1. **New connection (Azure Cosmos DB (SQL API))** ポップアップで、次の値で新しい接続を構成し、**Create** を選択してください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Name** | *`CosmosSqlConn`* |
    | **Connect via integration runtime** | *AutoResolveIntegrationRuntime* |
    | **Authentication method** | *Account key* &vert; *Connection string* |
    | **Account selection method** | *From Azure subscription* |
    | **Azure subscription** | *Your existing Azure subscription* |
    | **Azure Cosmos DB account name** | *Your existing Azure Cosmos DB account name you chose earlier in this lab* |
    | **Database name** | *cosmicworks* |

1. **Source data store** セクションに戻り、**Source tables** セクションで **Use query** を選択してください。

1. **Table name** 一覧で **products** を選択してください。

1. **Query** エディターで既存の内容を削除し、次のクエリを入力してください。

    ```
    SELECT 
        p.name, 
        p.categoryName as category, 
        p.price 
    FROM 
        products p
    ```

1. **Preview data** を選択してクエリの有効性をテストしてください。**Next** を選択してウィザードの **Destination** ステップに進んでください。

1. ウィザードの **Destination** ステップで、**Destination type** 一覧から **Azure Cosmos DB (SQL API)** を選択してください。

1. **Connection** 一覧で **CosmosSqlConn** を選択してください。

1. **Target** 一覧で **flatproducts** を選択し、**Next** を選択してウィザードの **Settings** ステップに進んでください。

1. ウィザードの **Settings** ステップで、**Task name** フィールドに **`FlattenAndMoveData`** を入力してください。

1. 残りのフィールドは既定の空白値のままにして、**Next** を選択し、ウィザードの最終ステップに進んでください。

1. ウィザードで選択した手順の **Summary** を確認し、**Next** を選択してください。

1. デプロイの各ステップを確認してください。デプロイが完了したら、**Finish** を選択してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Resource groups** を選択し、このラボで先ほど作成または確認したリソース グループを選択してから、このラボで作成した **cosmosdb420-XXXXX** Azure Cosmos DB アカウント リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で **Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**flatproducts** コンテナー ノードを選択して、**New SQL Query** を選択してください。

1. エディター領域の内容を削除してください。

1. **name** が **HL Headset** に等しいすべてのドキュメントを返す新しい SQL クエリを作成してください。

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

1. **Execute Query** を選択してください。

1. クエリ結果を確認してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。