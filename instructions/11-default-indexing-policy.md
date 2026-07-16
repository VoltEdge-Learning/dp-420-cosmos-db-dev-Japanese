# ポータルを使用して Azure Cosmos DB for NoSQL コンテナーの既定インデックス ポリシーを確認する

## ラボ シナリオ

Azure Cosmos DB のすべてのコンテナーには、コンテナー内の項目をどのようにインデックス化するかをサービスに指示するインデックス ポリシーがあります。既定では、このインデックス ポリシーはすべての項目のすべてのプロパティをインデックス化します。既定のインデックス ポリシーにより、プロジェクト開始時にインデックス、パフォーマンス、管理を細かく考えなくても、Azure Cosmos DB をすばやく使い始めることができます。

このラボでは、Data Explorer を使用していくつかのコンテナーの既定インデックス ポリシーを確認し、操作します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: Azure Cosmos DB NoSQL API アカウントを作成する。
- タスク 2: Azure Cosmos DB NoSQL API アカウントにデータを投入する。
- タスク 3: 既定のインデックス ポリシーを表示および操作する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab11.png)

## タスク 1: ポータルを使用して Azure Cosmos DB SQL API コンテナーの既定インデックス ポリシーを確認する

このタスクでは、Azure Cosmos DB SQL アカウントをプロビジョニングし、基本設定を構成するとともに、今後の開発に必要な接続情報を取得します。

Azure Cosmos DB は複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングする際には、そのアカウントでサポートする API（例: Mongo API または NoSQL API）を選択します。Azure Cosmos DB for NoSQL アカウントのプロビジョニングが完了したら、エンドポイントとキーを取得し、Azure SDK for .NET または任意の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続できます。

1. Azure Portal ページで、ポータル上部の Search resources, services and docs (G+/) ボックスに **Azure Cosmos DB (1)** と入力し、services の下にある **Azure Cosmos DB (2)** を選択してください。

   ![06](media/New-image1.png)

1. **Azure Cosmos DB for NoSQL** の下で **+ Create (1)** を選択し、**Create (2)** をクリックして **Azure Cosmos DB for NoSQL** アカウントを作成してください。

    ![06](media/New-image2.png)

    ![06](media/New-image3.png)

1. 次の設定を指定し、その他の設定は既定値のままにして **Review + create (10)** を選択してください。

    | **Setting**         | **Value** |
    | --------------------|--------------------------------------------------- |
    | **Workload Type**   | *Production* (1) |
    | **Subscription**    | *Your existing Azure subscription* (2) |
    | **Resource group**  | *Select an existing Cosmosdb-<inject key="DeploymentID" enableCopy="false"/>* (3) |
    | **Account Name**    | *sql-<inject key="DeploymentID" enableCopy="false"/>* (4) |
    | **Location**        | *Choose the default region* (5) |
    | **Capacity mode**   | *Provisioned throughput* (6) |
    | **Apply Free Tier Discount** | *Do Not Apply* (7) |
    | **Limit the total amount of throughput that can be provisioned on this account** | *Unchecked* (8) |

     ![06](media/DB25.png)

1. **Create** をクリックしてください。

    ![06](media/New-image5.png)

1. このタスクを続行する前に、デプロイが完了するまで待機してください。

1. デプロイ完了後、**Go to resources** を選択してください。

    ![06](media/New-image6.png)

1. **Azure Cosmos DB account** で、左側メニューの **Settings (1)** を展開し、**Keys (2)** を選択してください。

    ![06](media/DB15.png)


1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    1. **URI (1)** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY (2)** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

        ![06](media/New-image9.png)

1. Web ブラウザーのウィンドウまたはタブは開いたままにしてください。

    > タスク完了おめでとうございます。次は検証です。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、次のタスクに進めます。
    > - 表示されない場合は、エラーメッセージをよく確認し、ラボ ガイドの手順に従って再試行してください。
    > - サポートが必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

    <validation step="140fa89d-d46a-4ae0-a198-9c51019a9b40" />

### タスク 2: Azure Cosmos DB NoSQL API アカウントにデータを投入する

このタスクでは、CosmicWorks ツールを使用して Azure Cosmos DB NoSQL アカウントにサンプル製品データを投入します。Visual Studio Code のターミナルにツールをインストールした後、Cosmos DB の endpoint と key を指定して投入コマンドを実行します。ツールはデータベースとコンテナーを作成し、製品データをアカウントに挿入します。

[cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールは、任意の Azure Cosmos DB SQL API アカウントにサンプル データをデプロイします。このツールはオープンソースで NuGet から利用できます。このツールを Azure Cloud Shell にインストールし、データベースへのデータ投入に使用します。

1. Visual Studio Code を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

   ![Visual Studio Code Icon](./media/vscode1.jpg)

1. **Visual Studio Code** で、**... (ellipses) (1)** を選択し、**Terminal (2)** を選択してから **New Terminal (3)** を選択し、既存インスタンス内で新しいターミナルを開いてください。

    ![06](media/New-image36.png)

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをマシン全体で利用できるようにインストールしてください。

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    ![06](media/DB50.png)

    > &#128161; このコマンドの完了には数分かかる場合があります。過去にこのツールの最新バージョンをすでにインストールしている場合、このコマンドは警告メッセージ（*Tool 'cosmicworks' is already installed'）を出力します。

1. インストール完了後、以下のコマンドを実行するために **Visual Studio Code** を一度閉じて再度開いてください。

1. 次のコマンドライン オプションで cosmicworks を実行し、Azure Cosmos DB アカウントにデータを投入してください。

    | **Option** | **Value** |
    | --- | --- |
    | **--endpoint** | *このラボで先ほどコピーした endpoint 値* |
    | **--key** | *このラボで先ほどコピーした key 値* |
    | **--datasets** | *product* |

    ```
    cosmicworks --endpoint <cosmos-endpoint> --key <cosmos-key> --datasets product
    ```

    > &#128221; たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/** で key が **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。
    > ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

    >**Note**: エラーが発生する場合は、Visual Studio Code を閉じて再度開き、もう一度コマンドを実行してください。

1. **cosmicworks** コマンドがアカウントへのデータベース、コンテナー、項目の投入を完了するまで待機してください。

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

    > タスク完了おめでとうございます。次は検証です。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、次のタスクに進めます。
    > - 表示されない場合は、エラーメッセージをよく確認し、ラボ ガイドの手順に従って再試行してください。
    > - サポートが必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

    <validation step="eb8c6d06-bc6e-4170-a124-a95072d907a0" />

### タスク 3: 既定のインデックス ポリシーを表示および操作する

このタスクでは、Cosmos DB コンテナーの既定インデックス ポリシーを表示および変更します。Azure portal で Azure Cosmos DB に移動した後、_etag を除くすべてのパスをインデックス化する既定ポリシーを確認します。次に、**/price** パスのみをインデックス化するようポリシーを変更します。変更後、SQL クエリを実行してインデックス変更前後の要求料金を比較し、どのフィールドがインデックス化されているかによってクエリ効率がどのように変化するかを確認します。

コンテナーをコード、ポータル、ツールのいずれかで作成した場合、明示的に指定しない限り、インデックス ポリシーは既定の推奨設定になります。ここでは、その既定ポリシーを確認し、変更を加えます。

1. **Azure portal** に移動してください。

1. Azure Portal ページで、ポータル上部の Search resources, services and docs (G+/) ボックスに **Azure Cosmos DB** と入力し、services の下にある **Azure Cosmos DB** を選択してください。

   ![06](media/New-image1.png)

1. **sql-<inject key="DeploymentID" enableCopy="false"/>** を選択してください。

     ![06](media/New-image68.png)

1. **Azure Cosmos DB** アカウント リソース内で、**Data Explorer** ペインに移動してください。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、ナビゲーション ツリー内の新しい **products** コンテナー ノードを確認してから **New SQL Query** を選択してください。

     ![06](media/New-image74.png)

1. エディター領域の内容を削除してください。

1. **name** が **HL Headset** と等しいすべてのドキュメントを返す新しい SQL クエリを作成し、**Execute Query** を選択してください。

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

   ![06](media/New-image75.png)

1. クエリ結果を確認してください。

1. **Query Stats** を選択し、**Query Statistics** セクション内の **Request Charge** フィールドの値を確認してください。

     ![06](media/New-image76.png)

    > &#128221; 現在はすべてのパスがインデックス化されているため、このクエリは比較的効率的です。

1. **products** コンテナー ノード内で **Scale & Settings** を選択してください。

1. **Indexing Policy** セクション内の既定インデックス ポリシーを確認してください。

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

    > &#128221; この既定ポリシーは **_etag** を除くすべての可能なパスをインデックス化します。

1. エディター内でインデックス ポリシーの内容を **/price** パスのみをインデックス化する設定に置き換え、**Save** を選択して変更を保存してください。

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

   ![06](media/New-image77.png)

1. **New SQL Query** を選択してください。

1. エディター領域の内容を削除してください。

1. **name** が **HL Headset** と等しいすべてのドキュメントを返す新しい SQL クエリを作成し、**Execute Query** を選択してください。

    ```
    SELECT * FROM p WHERE p.name = 'HL Headset'
    ```

   ![06](media/New-image78.png)

1. クエリ結果を確認してください。

1. **Query Stats** を選択し、**Query Statistics** セクション内の **Request Charge** フィールドの値を確認してください。

    > &#128221; **name** プロパティがインデックス化されなくなったため、要求料金が増加しています。

    ![06](media/New-image79.png)

1. エディター領域の内容を削除してください。

1. **price** が **$3,000** より大きいすべてのドキュメントを返す新しい SQL クエリを作成してください。

    ```
    SELECT * FROM p WHERE p.price > 3000
    ```

1. **Execute Query** を選択してください。

1. クエリ結果を確認してください。

1. **Query Stats** を選択し、**Query Statistics** セクション内の **Request Charge** フィールドの値を確認してください。

## まとめ

このラボでは、コンテナー内の項目をどのようにインデックス化するかを制御する Azure Cosmos DB のインデックス ポリシーを学習しました。既定ではすべてのプロパティがインデックス化されるため、手動でインデックス管理を行わなくても効率的にクエリできます。ラボでは、Cosmos DB NoSQL アカウントのプロビジョニング、サンプル データの投入、既定インデックス ポリシーの確認を行いました。次に、**/price** パスのみをインデックス化するようポリシーを変更し、クエリ効率および要求料金への影響を確認しました。この演習を通して、インデックス ポリシーのカスタマイズが Cosmos DB のパフォーマンスとクエリ コストに与える影響を理解しました。

### レビュー

このラボでは、次を完了しました。

- Azure Cosmos DB for NoSQL アカウントを作成した。
- Azure Cosmos DB for NoSQL アカウントにデータを投入した。
- 既定インデックス ポリシーを表示および操作した。

### ラボは正常に完了しました