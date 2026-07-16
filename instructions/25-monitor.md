# Azure Monitor を使用して Azure Cosmos DB for NoSQL アカウントを分析する

## ラボ シナリオ

Azure Monitor は Azure リソースを監視するための機能を包括的に提供する Azure のフルスタック監視サービスです。Azure Cosmos DB は Azure Monitor を使用して監視データを生成します。Azure Monitor は Cosmos DB のメトリックとテレメトリ データを取得します。

このラボでは、Azure Cosmos DB コンテナーに対してシミュレートされたワークロードを実行し、そのワークロードが Azure Cosmos DB アカウントに与える影響を分析します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する。
- タスク 3: .NET スクリプトに Microsoft.Azure.Cosmos と Newtonsoft.Json ライブラリをインポートする。
- タスク 4: コンテナーとワークロードを作成するスクリプトを実行する。
- タスク 5: Azure Monitor を使用して Azure Cosmos DB アカウント使用状況を分析する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab25.png)

### タスク 1: 開発環境を準備する

このタスクでは、Visual Studio Code で開発環境をセットアップします。

1. Visual Studio Code を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

   ![Visual Studio Code Icon](./media/vscode1.jpg)

2. 左側ペインの **Extension (1)** アイコンを選択してください。検索バーに **C# (2)** を入力し、表示された **extension (3)** を選択して、最後に拡張機能の **Install (4)** を選択してください。

    ![](media/visualstudioo.png)

3. 画面左上の **file** オプションを選択し、メニューから **Open Folder** を選択して **C:\AllFiles** に移動してください。

4. **dp-420-cosmos-db-dev-main** フォルダーを選択し、**Select Folder** をクリックしてください。

    ![](media/lab12-1.png)

    >**Note:** **Do you trust the authors of the files in this folder?** ポップアップが表示されたら、**Yes, I trust the authors** を選択してください。

      ![06](media/lab12-2.png)

### タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する

このタスクでは、NoSQL API を使用して Azure Cosmos DB アカウントを作成します。アカウントのプロビジョニング後、endpoint (URI) と primary key を含む必要な接続情報を取得します。

Azure Cosmos DB は複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングするときは、アカウントでサポートする API（例: **Mongo API** または **NoSQL API**）を選択します。Azure Cosmos DB for NoSQL アカウントのプロビジョニング完了後、エンドポイントとキーを取得し、Azure SDK for .NET または任意の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続できます。

1. **Azure Portal** に戻ってください。

1. *Azure Cosmos DB (1)* を検索し、**Azure Cosmos DB (2)** を選択してください。

     ![05](media/lab12-3.png)

1. **Azure Cosmos DB** ページで **+ Create** を選択してください。

     ![05](media/lab12-4.png)

1. Create an Azure Cosmos DB account ページで、**Azure Cosmos DB for NoSQL** タブの **Create** を選択してください。

     ![05](media/T2S9.png)

1. **Create Azure Cosmos DB Account** ペインで **Basics** タブを確認してください。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Subscription** | *Your existing Azure subscription* |
    | **Resource group** | **cosmosdb-<inject key="DeploymentID" enableCopy="false"/>** |
    | **Account Name** | **cosmosdb-<inject key="DeploymentID" enableCopy="false"/>** |
    | **Location** | *Choose any available region* |
    | **Capacity mode** | *Provisioned throughput* |
    | **Apply Free Tier Discount** | *`Do Not Apply`* |
    | **Limit the total amount of throughput that can be provisioned on this account** | *Uncheck* |

     ![05](media/lab12-6.png)

1. **Review + Create** をクリックし、検証で Success が表示されたら **Create** をクリックしてください。

1. このタスクを続行する前に、デプロイが完了するまで待機してください。

1. **Go to resource** をクリックして新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、左メニューの settings から **Keys** ペインに移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. show primary key アイコンを選択して **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

1. ブラウザー ウィンドウは閉じずに最小化してください。次の手順でバックグラウンド ワークロードを開始した数分後に Azure portal に戻ります。

    > **Congratulations** on completing the lab! Now, it's time to validate it. Here are the steps:
    > - Hit the Validate button for the corresponding task. If you receive a success message, you have successfully validated the lab.
    > - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
    > - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help you out.

    <validation step="f7d09acb-1ee3-4b09-97b4-04f9af7a3aa9" />

### タスク 3: .NET スクリプトに Microsoft.Azure.Cosmos と Newtonsoft.Json ライブラリをインポートする

このタスクでは、.NET CLI の [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] コマンドを使用して、事前構成済みのパッケージ フィードからパッケージをインポートします。.NET のインストールでは、既定のパッケージ フィードとして NuGet を使用します。

1. **Visual Studio Code** の **Explorer** ペインで **25-monitor** フォルダーに移動してください。

1. **25-monitor** フォルダーを右クリックし、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

    >**Note:** このコマンドを実行すると、開始ディレクトリが **25-monitor** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを実行して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加してください。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. 次のコマンドを実行して、NuGet から [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.1] パッケージを追加してください。

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

### タスク 4: コンテナーとワークロードを作成するスクリプトを実行する

このタスクでは、Azure Cosmos DB アカウントを使用してワークロードを作成および監視するスクリプトを実行します。スクリプトは 3 つのコンテナーをセットアップし、データを読み込み、複数ユーザー アプリケーションが SQL クエリでデータベースにアクセスする状況をシミュレートします。これにより、Azure Cosmos DB の使用状況とパフォーマンス メトリックを監視できます。

これで、Azure Cosmos DB Account の使用状況を監視するワークロードを実行する準備が整いました。バックグラウンドで実行されるこのスクリプトは、3 つのコンテナーを作成し、それらにデータを読み込みます。その後、複数ユーザー アプリケーションが Azure Cosmos DB アカウントにアクセスする状況を再現するために、SQL クエリをランダムに実行します。

1. **Visual Studio Code** の **Explorer** ペインで **25-monitor** フォルダーに移動してください。

1. **Program.cs** コード ファイルを開いてください。

1. 既存の **endpoint** という変数を更新し、先ほど作成した Azure Cosmos DB アカウントの **endpoint** を設定してください。

    ```
    private static readonly string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは **private static readonly string endpoint = "https&shy;://dp420.documents.azure.com:443/";** になります。

1. 既存の **key** という変数を更新し、先ほど作成した Azure Cosmos DB アカウントの **key** を設定してください。

    ```
    private static readonly string key = "<cosmos-key>";
    ```

    > &#128221; たとえば key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは **private static readonly string key = "fDR2ci9QgkdkvERTQ==";** になります。

1. **Program.cs** ファイルを保存してください。

1. *Integrated Terminal* に戻ってください。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```
    > &#128221; このスクリプトの前半では 3 つのコンテナーを作成してデータを読み込みます。所要時間は約 2 分です。レート制限イベントを再現するため、スクリプトはその後、プロビジョニング済みスループットを 400 RU/s に設定します。続いて ***Creating simulated background workload, wait 5-10 minutes and go to the next step of the exercise.*** というメッセージが表示されるはずです。Azure リソースは監視データを Azure Monitor に非同期で送信するため、Azure Monitor Metrics と Insights で診断データの取得が始まるまで少し待つ必要があります。5〜10 分後に次のステップへ進んでください。追加の診断データを収集したい場合は、5〜10 分後にスクリプトを停止せず、ラボの最後まで実行し続けてもかまいません。

    > &#128221; コンパイラが「スクリプトが多くの操作を同期的に実行し、操作の応答待ちをしていない」ことを検出するため、黄色の警告がいくつか表示されます。これは複数の SQL スクリプトを同時に実行するための想定動作なので、警告は無視して問題ありません。

    >**Note**: 上記コマンド実行後に Visual Studio Code がクラッシュする場合があります。その場合はコマンドを再実行して次のタスクに進んでください。Visual Studio が 2 回以上クラッシュする場合は、次の手順に従って Visual Studio をアンインストールして再インストールしてください。

      - スタート メニューで **Control Panel** を検索して選択してください。
      - **Programs** の **Uninstall a program** リンクを選択し、**Microsoft visual studio code (user)** を見つけて右クリックし、**Uninstall** を選択してください。
      - **Microsoft edge** に戻り、アドレス バーに https://code.visualstudio.com/download と入力して、**Windows** のダウンロード アイコンをクリックしてください。
      - ダウンロード完了後、ダウンロードしたファイルを開いて Visual Studio Code をインストールしてください。
      - インストール完了後、Visual Studio Code を開いてステップ 7 を再実行してください。

    > **Congratulations** on completing the lab! Now, it's time to validate it. Here are the steps:
    > - Hit the Validate button for the corresponding task. If you receive a success message, you have successfully validated the lab.
    > - If not, carefully read the error message and retry the step, following the instructions in the lab guide.
    > - If you need any assistance, please contact us at cloudlabs-support@spektrasystems.com. We are available 24/7 to help you out.

    <validation step="ff1fcfa9-5e37-4665-85cc-9628e4ffb657" />


### タスク 5: Azure Monitor を使用して Azure Cosmos DB アカウント使用状況を分析する

このタスクでは、ブラウザーに戻り、Azure Monitor の Insights および Metrics レポートを確認します。

#### サブタスク 1: Azure Monitor Metrics レポート

1. 先ほど最小化したブラウザー ウィンドウに戻ってください。閉じた場合は新しく開き、[Azure Portal](portal.azure.com) の Azure Cosmos DB アカウント ページに移動してください。

1. Azure Cosmos DB の左メニューで *Monitoring* の **Metrics** を選択してください。**Scope** と **Metric Namespace** フィールドには正しい情報が事前入力されています。以降の手順では、いくつかの **Metric** オプションと、*Add filter*、*Apply splitting* 機能を確認します。

1. 既定では *Metrics* セクションに過去 24 時間の診断情報が表示されます。前のステップで作成したワークロード中のメトリックを確認するため、より細かい時間範囲に変更します。右上の ***Local time: Last 24 hours (Automatic)*** ボタンを選択すると、複数のラジオ ボタン時間範囲オプションが表示されます。**Last 30 minutes** を選択し、**Apply** を選択してください。必要に応じて *Custom* ラジオ ボタンを選び、開始日時と終了日時を指定してさらに細かく設定できます。

1. 診断チャートに適した時間範囲が設定できたので、いくつかの Metrics を確認します。まず一般的なメトリックとして、*Metric* プルダウンから **Total Request Units** を選択してください。既定では RU の合計値として表示されます。Aggregation プルダウンを average や max に変更することもできます。これら 2 つを確認したら、以降の手順のため *Sum* に戻してください。

1. このメトリックにより、Azure Cosmos DB Account で使用された要求ユニット数の概要が把握できます。ただし、アカウント内に複数のデータベースやコンテナーがある場合、現在のチャートでは問題箇所を特定しにくいことがあります。そこで、データベースごとの RU 消費を確認します。チャート タイトル下のメニューで **Apply splitting** を選択し、**Values** プルダウンで **DatabaseName** を選び、チャート上の任意の場所を選択して変更を確定してください。チャートの直上に **Split by = DatabaseName** ボタンが表示されます。

1. これで、どのデータベースが主に処理を行っているかが分かります。ただし、どのコンテナーが主に処理しているかはまだ分かりません。**Split by = DatabaseName** ボタンを選択して Split 条件を変更し、*Values* プルダウンで **CollectionName** を選択してください。これで **customer** と **salesOrder** コレクションのデータが表示されるはずです。ここで 1 つ問題があり、**salesOrder** コレクションは **database-v2** と **database-v3** の 2 つのデータベースに存在するため、この値は両方のデータベースで同名コレクションを集計したものになっています。

1. これは簡単に修正できます。**Add filter** ボタンを選択し、*Property* プルダウンで **DatabaseName**、*Values* で **database-V3** を選択してください。

1. さらに 2 つのメトリックを確認します。既存チャートを編集しますが、必要であれば新しいチャートを作成してもかまいません。チャート上部で *Azure Cosmos DB account name* と **Total Request Unit** ラベルが表示されたボタンを選択し、*Metric* プルダウンから **Total Requests** を選択してください。使用可能な集計が *Count* のみであることを確認できます。

1. ここでの 2 つの主要フィルターは、異なる種類の問題のトラブルシューティングに役立ちます。Property に **StatusCode**（詳細粒度が異なる類似フィルターとして **Status** もあります）を追加し、*Values* に **200** と **429** を選択してください。Split を StatusCode に変更してください。status 200（成功要求）に比べて、429（レート制限要求）が非常に多いことが分かります。429 例外が発生したのは、スクリプトが毎秒数千リクエストを送信する一方で、プロビジョニング済みスループットを 400 RU/s に設定していたためです。*成功要求に対して 429 例外がこのように多い状態は、本番環境では通常ではありません。健全な Azure Cosmos DB アカウントでは、本番環境での 429 例外は低頻度であるべきです*。**Total Request Units** に対しても同様に **StatusCode** または **Status** の *Properties* を使ってトラブルシューティングできます。

1. 引き続き **Total Request** を確認し、split を **OperationType** に変更してください。このプロパティを使うと、どの読み取りまたは書き込み操作が処理の大部分を占めているかを判断できます。このプロパティは **Total Request Units** に対しても同様に使用できます。

1. **Total Request Units** のときと同様に、さまざまなフィルターと splitting オプションを試してください。

1. この演習で最後に確認するメトリックは **normalised RU Consumption** です。split を **PartitionKeyRangeId** に変更してください。このメトリックは、どのパーティション キー範囲の使用率が高いかを特定するのに役立ちます。パーティション キー範囲へのスループット偏りを示します。*Metric* プルダウンからこのメトリックを選択してください。チャートには、100% の **Normalied RU Consumption** が継続する、非常に不健全なシステム状態が表示されるはずです。

> &#128221; 同時に複数のチャートを表示したい場合は、チャート名の上にある **+ New Chart** をクリックしてください。

> &#128221; メトリック自体を直接保存することはできませんが、既存または新規のダッシュボードを作成し、チャート右上の **Pin to dashboard** ボタンで追加できます。ボタンをクリックして **Create new** タブを選択し、名前を *DP-420 labs* にして **Create and pin** をクリックしてください。プライベート ダッシュボードを表示するには、左上の Portal Menu から Azure Resource options の Dashboard を選択してください。初回表示には数分かかる場合があります。

> &#128221; チャートを共有する別の方法として、Share プルダウンから Excel ファイルとしてダウンロードするか、Copy link オプションを使用できます。

#### サブタスク 2: Azure Monitor Insights レポート

Azure Monitor Metrics の診断レポートは、微調整に時間をかける必要がある場合があります。Cosmos DB Insights は、Azure Cosmos DB リソースの全体的なパフォーマンス、障害、運用状態を可視化します。これらの Insight チャートは、Metric と同様に事前構築されたチャートです。いくつか確認していきます。

1. Azure Cosmos DB 左メニューの *Monitoring* で **Insights** を選択してください。Overview から Management Options まで複数のタブがあることを確認できます。これら **Insight** チャートの一部を確認します。最初の Overview タブには、よく使う代表的なチャートのサマリーが表示されます。たとえば、total request、Data and Index usage、429 exceptions、Normalized RU consumption などです。これらの多くは前のセクションで確認しました。

1. チャート上部の **Time Range** で、この演習のワークロードを評価するため *15* または *30* 分を選択してください。

1. *各* チャートの右上に ***Open Metric Explorer*** オプションがあります。**Total Requests** チャートで **Open Metric Explorer** を選択してください。このオプションを選択すると、先ほど確認した Metric レポート画面へ移動します。Metric Explorer を開く利点は、チャートの多くがすでに構成済みである点です。

1. Metric チャート右上の **X** を選択して Insights ページに戻ってください。

1. Throughput タブを選択してください。これらのチャートはスループット問題の特定に有効です。特に、ホット パーティション検出に使える **Normalized RU Consumption (%) By PartitionKeyRangeID** チャートに注目してください。

1. Requests タブを選択してください。これらのチャートは、アカウントで発生した制限イベント数（429 vs. 200）の分析と、操作種別ごとの要求数確認の両方に適しています。

1. Storage タブを選択してください。これらのチャートは、コレクションの増加状況とデータおよびインデックス使用状況を表示します。

1. System タブを選択してください。アプリケーションがアカウント メタデータを頻繁に作成、削除、またはクエリしている場合、429 例外が発生する可能性があります。これらのチャートは、メタデータへの頻繁なアクセスが 429 例外の原因かどうかを判断するのに役立ちます。加えて、メタデータ要求の状態も確認できます。

#### サブタスク 3: Azure Monitor Insights レポート

1. Program がまだ実行中の場合は、Visual Studio Code のコマンド ターミナルに戻ってください。

1. 統合ターミナルを閉じてください。

1. **Visual Studio Code** を閉じてください。

### まとめ

このラボでは、Azure Monitor が Azure Cosmos DB for NoSQL とどのように統合され、詳細な監視およびパフォーマンス分析情報を提供するかを確認しました。Cosmos DB コンテナーに対してシミュレートされたワークロードを実行することで、さまざまなメトリックとテレメトリ データが Azure Monitor でどのように収集・分析されるかを確認しました。

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- Azure Cosmos DB NoSQL API アカウントを作成した。
- .NET スクリプトに Microsoft.Azure.Cosmos と Newtonsoft.Json ライブラリをインポートした。
- コンテナーとワークロードを作成するスクリプトを実行した。
- Azure Monitor を使用して Azure Cosmos DB アカウント使用状況を分析した。

### ラボは正常に完了しました