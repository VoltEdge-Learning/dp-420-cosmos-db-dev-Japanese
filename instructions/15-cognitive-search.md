---
lab:
    title: 'Azure Cognitive Search と Azure Cosmos DB SQL API を使用してデータを検索する'
    module: 'モジュール 7 - Azure Cosmos DB SQL API を Azure サービスと統合する'
---

# Azure Cognitive Search と Azure Cosmos DB SQL API を使用してデータを検索する

Azure Cognitive Search は、サービスとしての検索エンジンと AI 機能との深い統合を組み合わせることで、検索インデックス内の情報を強化します。

このラボでは、Azure Cosmos DB SQL API コンテナー内のデータを自動的にインデックス化し、Azure Cognitive Services Translator 機能を使用してデータを強化する Azure Cognitive Search インデックスを構築します。

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
    | **Capacity mode** | *Serverless* |

    > &#128221; ラボ環境には、新しいリソース グループの作成を制限する制約がある場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. このタスクを続行する前に、デプロイ タスクが完了するまで待機してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Keys** ペインに移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

    1. **PRIMARY CONNECTION STRING** フィールドの値を記録してください。この演習の後半でこの **connection string** 値を使用します。

1. リソース メニューから **Data Explorer** を選択してください。

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --: | :-- |
    | **Database id** | *Create new* &vert; *cosmicworks* |
    | **Container id** | *products* |
    | **Partition key** | */categoryId* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

## Azure Cosmos DB SQL API アカウントにサンプル データをシードする

ここでは、**cosmicworks** データベースと **products** コンテナーを作成するコマンドライン ユーティリティを使用します。次にこのツールが項目セットを作成し、その変更をターミナル ウィンドウで実行中の変更フィード プロセッサーで確認します。

1. **Visual Studio Code** で **Terminal** メニューを開き、**Split Terminal** を選択して、既存のインスタンスと並べて新しいターミナルを開いてください。

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールを、マシン全体で使用できるようにインストールしてください。

    ```
    dotnet tool install --global cosmicworks
    ```

    > &#128161; このコマンドの完了には数分かかる場合があります。過去にこのツールの最新バージョンをすでにインストールしている場合、このコマンドは警告メッセージ（*Tool 'cosmicworks' is already installed'）を出力します。

1. 次のコマンドライン オプションを使用して cosmicworks を実行し、Azure Cosmos DB アカウントをシードしてください。

    | **Option** | **Value** |
    | ---: | :--- |
    | **--endpoint** | *このラボの前半でコピーした endpoint 値* |
    | **--key** | *このラボの前半でコピーした key 値* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; たとえば、endpoint が **https&shy;://dp420.documents.azure.com:443/** で、key が **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります:
    > ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

1. **cosmicworks** コマンドが、アカウントへのデータベース、コンテナー、および項目の追加を完了するまで待機してください。

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

## Azure Cognitive Search リソースを作成する

この演習を続行する前に、最初に新しい Azure Cognitive Search インスタンスを作成する必要があります。

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインしてください。

1. **+ Create a resource** を選択し、*Cognitive Search* を検索してから、次の設定で新しい **Azure Cognitive Search** アカウント リソースを作成してください。残りの設定はすべて既定値のままにしてください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Subscription** | *既存の Azure サブスクリプション* |
    | **Resource group** | *既存のリソース グループを選択するか新規作成する* |
    | **Name** | *グローバルで一意の名前を入力する* |
    | **Location** | *利用可能な任意のリージョンを選択する* |
    | **Pricing tier** | *Free* |

    > &#128221; ラボ環境には、新しいリソース グループの作成を制限する制約がある場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. このタスクを続行する前に、デプロイ タスクが完了するまで待機してください。

1. 新しく作成した **Azure Cognitive Search** アカウント リソースに移動してください。

## Azure Cosmos DB SQL API データ用のインデクサーとインデックスを構築する

ここでは、特定の Azure Cosmos DB SQL API コンテナー内のデータのサブセットを 1 時間ごとにインデックス化するインデクサーを作成します。

1. **Azure Cognitive Search** リソース ブレードで **Import data** を選択してください。

1. **Import data** ウィザードの **Connect to your data** ステップで、**Data Source** 一覧から **Azure Cosmos DB** を選択してください。

1. 次の設定でデータ ソースを構成してください。残りの設定はすべて既定値のままにしてください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Data source name** | *products-cosmossql-source* |
    | **Connection string** | *先ほど作成した Azure Cosmos DB SQL API アカウントの **connection string*** |
    | **Database** | *cosmicworks* |
    | **Collection** | *products* |

1. **query** フィールドに次の SQL クエリを入力して、コンテナー内データのサブセットに対するマテリアライズド ビューを作成してください。

    ```
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

1. **Query results ordered by _ts** チェックボックスを選択してください。

    > &#128221; このチェックボックスにより、クエリ結果が **_ts** フィールドで並べ替えられていることを Azure Cognitive Search に伝えられます。この種類の並べ替えにより、増分進捗の追跡が可能になります。結果がタイムスタンプ順に並んでいるため、インデクサーが失敗した場合でも同じ **_ts** 値から再開できます。

1. **Next: Add cognitive skills** を選択してください。

1. **Next: Customize target index** を選択してください。

1. ウィザードの **Customize target index** ステップで、次の設定でインデックスを構成してください。残りの設定はすべて既定値のままにしてください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Index name** | *products-index* |
    | **Key** | *id* |

1. フィールド テーブルで、次の表を使用して各フィールドの **Retrievable**、**Filterable**、**Sortable**、**Facetable**、**Searchable** オプションを構成してください。

    | **Field** | **Retrievable** | **Filterable** | **Sortable** | **Facetable** | **Searchable** |
    | ---: | :---: | :---: | :---: | :---: | :---: |
    | **id** | &#10004; | &#10004; | &#10004; | | |
    | **categoryId** | &#10004; | &#10004; | &#10004; | &#10004; | |
    | **name** | &#10004; | &#10004; | &#10004; | | &#10004; (English - Microsoft) |
    | **price** | &#10004; | &#10004; | &#10004; | &#10004; | |

1. **Next: Create an indexer** を選択してください。

1. ウィザードの **Create an indexer** ステップで、次の設定でインデクサーを構成してください。残りの設定はすべて既定値のままにしてください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Name** | *products-cosmosdb-indexer* |
    | **Schedule** | *Hourly* |

1. データ ソース、インデックス、インデクサーを作成するために **Submit** を選択してください。

    > &#128221; 最初のインデクサーを作成した後にアンケート ポップアップを閉じる必要がある場合があります。

1. **Azure Cognitive Search** リソース ブレードで **Indexers** タブに移動し、最初のインデックス作成操作の結果を確認してください。

1. このタスクを続行する前に、**products-cosmosdb-indexer** インデクサーのステータスが **Success** になるまで待機してください。

    > &#128221; ブレードが自動更新されない場合は、**Refresh** オプションを使用して更新する必要がある場合があります。

1. **Indexes** タブに移動し、**products-index** インデックスを選択してください。

## 例の検索クエリでインデックスを検証する

Azure Cosmos DB SQL API データのマテリアライズド ビューが検索インデックスに作成されたので、Azure Cognitive Search の機能を活用する基本的なクエリをいくつか実行できます。

> &#128221; このラボは Azure Cognitive Search の構文を学ぶことを目的としていません。これらのクエリは、検索インデックスと検索エンジンで利用可能な機能の一部を示すために用意されています。

1. **products-index** &vert; **index** ペインで **Search** を選択し、**\***（ワイルドカード）演算子を使用して、可能なすべての結果を返す既定の検索クエリを実行してください。

1. この検索クエリで可能なすべての結果が返されることを確認してください。

1. **Query string** エディターに次のクエリを入力し、**Search** を選択してください。

    ```
    touring 3000
    ```

1. この検索クエリでは、**touring** または **3000** のいずれかを含む結果が返され、両方の用語を含む結果にはより高いスコアが付与されることを確認してください。結果はその後 **@search.score** フィールドで降順に並べ替えられます。

1. **Query string** エディターに次のクエリを入力し、**Search** を選択してください。

    ```
    red&$count=true
    ```

1. この検索クエリでは **red** という用語を含む結果が返され、同じページにすべて含まれていない場合でも、結果の合計件数を示すメタデータ フィールドが含まれることを確認してください。

1. **Query string** エディターに次のクエリを入力し、**Search** を選択してください。

    ```
    blue&$count=true&$top=6
    ```

1. サーバー側に一致する結果がさらに存在していても、この検索クエリでは 1 回に 6 件のみが返されることを確認してください。

1. **Query string** エディターに次のクエリを入力し、**Search** を選択してください。

    ```
    mountain&$count=true&$top=25&$skip=50
    ```

1. この検索クエリでは最初の 50 件がスキップされ、25 件の結果が返されることを確認してください。これがクライアント側アプリケーションのページ分割ビューであれば、これは 3 ページ目の結果に相当すると判断できます。

1. **Query string** エディターに次のクエリを入力し、**Search** を選択してください。

    ```
    touring&$count=true&$filter=price lt 500
    ```

1. この検索クエリでは、数値の price フィールドの値が 500 未満の結果のみが返されることを確認してください。

1. **Query string** エディターに次のクエリを入力し、**Search** を選択してください。

    ```
    road&$count=true&$top=15&facet=price,interval:500
    ```

1. この検索クエリでは、現在の結果ページにすべて表示されていない場合でも、各カテゴリに属する項目数を示すファセット データのコレクションが返されることを確認してください。この例では、一致した項目が 500 間隔の数値 price カテゴリに分割されます。これは通常、クライアント側アプリケーションのフィルターやナビゲーション補助を構成するために使用されます。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
