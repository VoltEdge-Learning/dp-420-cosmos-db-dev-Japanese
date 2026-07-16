# 一般的な操作向けに Azure Cosmos DB for NoSQL コンテナーのインデックス作成ポリシーを最適化する

## ラボ シナリオ

書き込みが多いワークロードや大きな JSON オブジェクトを扱うワークロードでは、クエリで使用することが分かっているプロパティのみをインデックス化するように、インデックス作成ポリシーを最適化すると有利な場合があります。

このラボでは、テスト用 .NET アプリケーションを使用して、既定のインデックス作成ポリシーで大きな JSON アイテムを Azure Cosmos DB SQL API コンテナーに挿入し、その後、少し調整したインデックス作成ポリシーでも同様に挿入します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する。
- タスク 3: 既定のインデックス作成ポリシーでテスト .NET アプリケーションを実行する。
- タスク 4: インデックス作成ポリシーを更新して .NET アプリケーションを再実行する。

### 推定所要時間: 60 分

## アーキテクチャ図

![image](architecturedia/lab23.png)

### タスク 1: 開発環境を準備する

このタスクでは、Visual Studio Code をセットアップして Azure Cosmos DB の作業用開発環境を準備します。

1. Visual Studio Code を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

   ![Visual Studio Code Icon](./media/vscode1.jpg)

2. 左側パネルから **Extensions** ブレードを選択してください。**C#** で検索し、**Install** を選択して拡張機能をインストールしてください。

   ![06](media/New-image50.png)

3. 画面左上の **file** オプションを選択し、メニューから **Open Folder** を選択してください。**C:\AllFiles\dp-420-cosmos-db-dev** に移動してください。

   ![06](media/New-image51.png)

4. **C:\AllFiles\dp-420-cosmos-db-dev** に移動し、**dp-420-cosmos-db-dev** を選択して **Select Folder** をクリックしてください。

    ![06](media/New-image54.png)

5. **Do you trust the author of the files in this folder** が表示された場合は、**Yes, I trust the authors** をクリックしてください。

   ![06](media/lab12-2.png)

### タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する

このタスクでは、Azure Cosmos DB SQL アカウントをプロビジョニングし、重要な設定を行ったうえで、今後の開発に必要な接続情報を取得します。

Azure Cosmos DB は複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングするときは、アカウントでサポートする API（例: **Mongo API** または **NoSQL API**）を選択します。Azure Cosmos DB for NoSQL アカウントのプロビジョニング完了後、エンドポイントとキーを取得し、Azure SDK for .NET または任意の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続できます。

1. **Azure Portal** ページで、ポータル上部の Search resources, services and docs (G+/) ボックスに **Azure Cosmos DB** と入力し、services の **Azure Cosmos DB** を選択してください。

   ![06](media/New-image1.png)

1. **Azure Cosmos DB for NoSQL** の **+ Create** を選択し、**Create** をクリックして **Azure Cosmos DB for NoSQL** アカウントを作成してください。

   ![06](media/New-image2.png)

   ![06](media/New-image3.png)

1. 以下の設定を指定し、その他の設定は既定値のままにして **Review + create** を選択してください。

   | **Setting** | **Value** |
   | ---------------- | ---------------------------------- |
   | **Subscription** | *Your existing Azure subscription* |
   | **Resource group** | **DP-420-<inject key="DeploymentID" enableCopy="false"/>** |
   | **Account Name** | *Enter a globally unique name* |
   | **Location** | *Choose any available region* |
   | **Capacity mode** | *Serverless* |

1. 検証で Success が表示されたら **Create** をクリックしてください。

1. このタスクを続行する前に、デプロイが完了するまで待機してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Data Explorer** ペインに移動してください。

1. **Data Explorer** ペインで **+ New Container** > **+ New Container** を選択してください。

   ![06](media/New-image107.png)

1. **New Container** ポップアップで各設定に次の値を入力し、**OK** を選択してください。

   | **Setting** | **Value** |
   | :-- | :-- |
   | **Database id** | *Create new* &vert; *``cosmicworks``* |
   | **Container id** | *``products``* |
   | **Partition key** | *``/categoryId``* |

   ![06](media/New-image108.png)

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認してください。

   ![06](media/New-image109.png)

1. 左側ナビゲーション メニューの **Settings** セクションから **Keys** ペインに移動してください。

   ![06](media/New-image7.png)

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

   - **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

   - **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

     ![06](media/New-image9.png)

1. **Visual Studio Code** に戻ってください。

   > **Congratulations** on completing the lab! Now, it's time to validate it. Here are the steps:
   > - Hit the Validate button for the corresponding task. If you receive a success message, you have successfully validated the lab.
   > - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
   > - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help you out.

   <validation step="0e380f12-d3fe-4670-a788-3fa3a3687768" />

### タスク 3: 既定のインデックス作成ポリシーでテスト .NET アプリケーションを実行する

このタスクでは、事前構築済みの .NET アプリケーションを実行し、大きな JSON オブジェクトを Azure Cosmos DB for NoSQL コンテナーに挿入します。

1. **Explorer** ペインで **23-index-optimization** フォルダーに移動してください。

1. **23-index-optimization** フォルダーを右クリックし、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

   > **Note:** このコマンドを実行すると、開始ディレクトリが **23-index-optimization** フォルダーに設定された状態でターミナルが開きます。

1. [dotnet build][docs.microsoft.com/dotnet/core/tools/dotnet-build] コマンドを使用してプロジェクトをビルドしてください。

   ```
   dotnet build
   ```

   > **Note:** **endpoint** 変数と **key** 変数が現在未使用であるというコンパイラ警告が表示される場合があります。このタスクでこれらの変数を使用するため、警告は無視して問題ありません。

1. 統合ターミナルを閉じてください。

1. **script.cs** コード ファイルを開いてください。

1. **endpoint** という名前の **string** 変数を見つけてください。その値を先ほど作成した Azure Cosmos DB アカウントの **endpoint** に設定してください。

   ```
   string endpoint = "<cosmos-endpoint>";
   ```

   > **For example:** endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. **key** という名前の **string** 変数を見つけてください。その値を先ほど作成した Azure Cosmos DB アカウントの **key** に設定してください。

   ```
   string key = "<cosmos-key>";
   ```

   > **For example:** key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. **script.cs** コード ファイルを **Save** してください。

1. **Visual Studio Code** で **23-index-optimization** フォルダーを右クリックし、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用してプロジェクトをビルドおよび実行してください。

   ```
   dotnet run
   ```
1. ターミナル出力を確認してください。アイテムの一意識別子と、操作の要求課金（RU）がコンソールに表示されるはずです。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、さらに少なくとも 2 回プロジェクトをビルドおよび実行してください。コンソール出力の RU 課金を確認してください。

   ```
   dotnet run
   ```
1. 統合ターミナルは開いたままにしてください。

   > **Congratulations** on completing the lab! Now, it's time to validate it. Here are the steps:
   > - Hit the Validate button for the corresponding task. If you receive a success message, you have successfully validated the lab.
   > - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
   > - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help you out.

   <validation step="e487cd8b-7edb-4b80-a103-3036d37a92b4" />

### タスク 4: インデックス作成ポリシーを更新して .NET アプリケーションを再実行する

このタスクでは、今後のクエリが主に name と categoryName プロパティに集中すると仮定します。大きな JSON アイテム向けに最適化するため、まずすべてのパスを除外するインデックス作成ポリシーを作成して、その他すべてのフィールドをインデックスから除外します。その後、特定パスのみを選択的に含めます。

1. Web ブラウザーに戻ってください。

1. **Azure Cosmos DB** アカウント リソース内で **Data Explorer** ペインに移動してください。

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
             "path": "/name/?"
           },
           {
             "path": "/categoryName/?"
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

1. **Visual Studio Code** に戻り、開いたままのターミナルに戻ってください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、さらに少なくとも 2 回プロジェクトをビルドおよび実行してください。コンソール出力の新しい RU 課金を確認してください。これは元の課金より大幅に小さくなるはずです。すべてのアイテム プロパティをインデックス化しないため、インデックス更新時の書き込みコストは大幅に下がります。ただし、インデックス化されていないプロパティで読み取りクエリを実行する必要がある場合は大きなコスト増につながる可能性があります。

   ```
   dotnet run
   ```

   > **Note:** 更新後の RU 課金が表示されない場合は、数分待つ必要がある場合があります。

1. Web ブラウザーに戻ってください。

   >**Note:** **Indexing Policy** ページが開いていない場合は、**Data Explorer** に移動し、**cosmicworks** データベース ノードを展開し、**products** コンテナー ノードを展開して、**Settings** を選択し、**Indexing Policy** セクションに移動してください。

1. インデックス作成ポリシーを次の変更済み JSON オブジェクトに置き換え、変更を **Save** してください。

   ```
    {
      "indexingMode": "none"
    }
   ```

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

1. **Visual Studio Code** に戻り、開いたままのターミナルに戻ってください。

1. **[dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run]** コマンドを使用して、さらに少なくとも 2 回プロジェクトをビルドおよび実行してください。コンソール出力の新しい RU 課金を確認してください。これは元の課金よりさらに小さくなるはずです。これはなぜでしょうか。このスクリプトはアイテム書き込み時の RU を測定するため、インデックスを持たない設定を選択すると、そのインデックス維持のオーバーヘッドが発生しません。一方で、書き込み RU は小さくなる代わりに、読み取りは非常に高コストになります。

   ```
   dotnet run
   ```

    > **Note:** 更新後の RU 課金が表示されない場合は、数分待つ必要がある場合があります。

1. **Visual Studio Code** を閉じてください。

### まとめ

このラボでは、特に大きな JSON オブジェクトを扱う書き込み中心のワークロードに対して、Azure Cosmos DB for NoSQL コンテナーのインデックス作成ポリシーを最適化する方法を確認しました。目的は、インデックス化するフィールドを制限することで書き込み時の RU（Request Unit）課金を削減し、パフォーマンスを向上させることです。

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- Azure Cosmos DB for NoSQL アカウントを作成した。
- 既定のインデックス作成ポリシーでテスト .NET アプリケーションを実行した。
- インデックス作成ポリシーを更新して .NET アプリケーションを再実行した。

### ラボは正常に完了しました