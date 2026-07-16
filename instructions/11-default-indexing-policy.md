---
lab:
    title: 'ポータルを使用して Azure Cosmos DB SQL API コンテナーの既定のインデックス ポリシーを確認する'
    module: 'モジュール 6 - Azure Cosmos DB SQL API のインデックス戦略を定義して実装する'
---

# ポータルを使用して Azure Cosmos DB SQL API コンテナーの既定のインデックス ポリシーを確認する

Azure Cosmos DB のすべてのコンテナーには、コンテナー内の項目をどのようにインデックス化するかをサービスに指示するインデックス ポリシーがあります。既定では、このインデックス ポリシーはすべての項目のすべてのプロパティをインデックス化します。既定のインデックス ポリシーにより、プロジェクトの開始時にインデックス、パフォーマンス、管理について考える必要がないため、Azure Cosmos DB をすばやく使い始めることができます。

このラボでは、Data Explorer を使用して、いくつかのコンテナーの既定のインデックス ポリシーを確認し、変更します。

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

## Azure Cosmos DB SQL API アカウントにデータをシードする

[cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールは、任意の Azure Cosmos DB SQL API アカウントにサンプル データをデプロイします。このツールはオープンソースで、NuGet から利用できます。このツールを Azure Cloud Shell にインストールし、データベースのシードに使用します。

1. **Visual Studio Code** を起動してください。

1. **Visual Studio Code** で **Terminal** メニューを開き、**New Terminal** を選択して新しいターミナル インスタンスを開いてください。

    > &#128221; Visual Studio Code のインターフェイスにまだ慣れていない場合は、[Visual Studio Code の Get Started ガイド][code.visualstudio.com/docs/getstarted] を確認してください。

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

## 既定のインデックス ポリシーを表示して変更する

コード、ポータル、またはツールでコンテナーを作成した場合、明示的に指定しなければインデックス ポリシーは適切な既定値に設定されます。この既定のインデックス ポリシーを確認し、ポリシーに変更を加えます。

1. Web ブラウザーで Azure portal (``portal.azure.com``) に移動してください。

1. **Resource groups** を選択し、次にこのラボで先ほど作成または確認したリソース グループを選択し、さらにこのラボで作成した **Azure Cosmos DB account** リソースを選択してください。

1. **Azure Cosmos DB** アカウント リソース内で、**Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**SQL API** ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認してください。

1. **SQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選択し、**New SQL Query** を選択してください。

1. エディター領域の内容を削除してください。

1. **name** が **HL Headset** と等しいすべてのドキュメントを返す新しい SQL クエリを作成してください。

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. **Execute Query** を選択してください。

1. クエリ結果を確認してください。

1. **Query** タブで **Query Stats** を選択してください。

1. **Query** タブで、**Query Statistics** セクション内の **Request Charge** フィールドの値を確認してください。

    > &#128221; 現在はすべてのパスがインデックス化されているため、このクエリは比較的効率的であるはずです。

1. **SQL API** ナビゲーション ツリーの **products** コンテナー ノード内で、**Scale & Settings** を選択してください。

1. **Indexing Policy** セクション内の既定のインデックス ポリシーを確認してください。

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

    > &#128221; この既定ポリシーは、**_etag** を除くすべての可能なパスをインデックス化します。

1. エディター内でインデックス ポリシーの内容を置き換え、**/price** パスのみをインデックス化するようにしてください。

    ```
    {
      "indexingMode": "consistent",
      "automatic": true,
      "includedPaths": [
        {
          "path": "/price/?"
        }
      ],
      "excludedPaths": [
        {
          "path": "/*"
        }
      ]
    }
    ```

1. 変更を保存するために **Save** を選択してください。

1. **New SQL Query** を選択してください。

1. エディター領域の内容を削除してください。

1. **name** が **HL Headset** と等しいすべてのドキュメントを返す新しい SQL クエリを作成してください。

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

1. **Execute Query** を選択してください。

1. クエリ結果を確認してください。

1. **Query** タブで **Query Stats** を選択してください。

1. **Query** タブで、**Query Statistics** セクション内の **Request Charge** フィールドの値を確認してください。

    > &#128221; **name** プロパティがインデックス化されなくなったため、要求料金が増加しています。

1. エディター領域の内容を削除してください。

1. **price** が **$3,000** より大きいすべてのドキュメントを返す新しい SQL クエリを作成してください。

    ```
    SELECT * FROM p WHERE p.price > 3000
    ```

1. **Execute Query** を選択してください。

1. クエリ結果を確認してください。

1. **Query** タブで **Query Stats** を選択してください。

1. **Query** タブで、**Query Statistics** セクション内の **Request Charge** フィールドの値を確認してください。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[nuget.org/packages/cosmicworks]: https://www.nuget.org/packages/cosmicworks/
