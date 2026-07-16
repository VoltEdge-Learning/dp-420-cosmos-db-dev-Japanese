# Azure Functions を使用して Azure Cosmos DB for NoSQL のデータを処理する

## ラボ シナリオ

Azure Functions の Azure Cosmos DB トリガーは、変更フィード プロセッサを使用して実装されています。この知識を使うことで、Azure Cosmos DB for NoSQL コンテナーでの作成および更新操作に応答する関数を作成できます。変更フィード プロセッサを手動で実装したことがある場合、Azure Functions のセットアップはそれと似ています。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: Azure Cosmos DB for NoSQL アカウントを作成する
- タスク 2: Application Insight を作成する
- タスク 3: Azure Function アプリと Azure Cosmos DB トリガー関数を作成する
- タスク 4: .NET で関数コードを実装する
- タスク 5: Azure Cosmos DB for NoSQL アカウントにサンプル データを投入する

## 推定所要時間: 60 分

## アーキテクチャ図

![image](architecturedia/lab14.png)

## タスク 1: Azure Cosmos DB for NoSQL アカウントを作成する

このタスクでは、Azure Cosmos DB SQL アカウントをプロビジョニングし、主要設定を構成して、今後の開発に必要な接続情報を取得します。

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングする際は、そのアカウントでサポートする API（例: **Mongo API** または **NoSQL API**）を選択します。Azure Cosmos DB for NoSQL アカウントのプロビジョニング完了後、エンドポイントとキーを取得し、Azure SDK for .NET または任意の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続できます。

1. **Azure Portal** ページ上部の Search resources, services and docs (G+/) ボックスに **Azure Cosmos DB (1)** と入力し、services の **Azure Cosmos DB (2)** を選択してください。

   ![06](media/New-image1.png)

1. **Azure Cosmos DB for NoSQL** の下にある **+ Create (1)** を選択します。

    ![06](media/New-image2.png)

    - **Create (2)** をクリックして **Azure Cosmos DB for NoSQL** アカウントを作成します。

      ![06](media/New-image3.png)

1. 以下の設定を指定し、その他の設定は既定値のままにして **Review + create (9)** を選択してください。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Workload Type** | *Learning* **(1)** |
    | **Subscription** | *Your existing Azure subscription* **(2)** |
    | **Resource group** | *Select an existing Cosmosdb-<inject key="DeploymentID" enableCopy="false"/>* **(3)** |
    | **Account Name** | *sql-<inject key="DeploymentID" enableCopy="false"/>* **(4)** |
    | **Location** | *Choose any available region* **(5)** |
    | **Capacity mode** | *Provisioned throughput* **(6)** |
    | **Apply Free Tier Discount** | *Do Not Apply* **(7)** |
    | **Limit total account throughput** | *Disable* **(8)** |

    ![06](media/c28.png)
    ![06](media/c29.png)

1. 検証が成功したら **Create** をクリックします。

1. このタスクを続行する前に、デプロイの完了を待ちます。

1. **Go to resources** を選択します。新しく作成された **Azure Cosmos DB** アカウントの **Settings** で **Keys** ペインに移動します。

    ![06](media/New-image6.png

    ![06](media/New-image7.png)

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    - **URI** フィールドの値を記録します。この演習の後半でこの **endpoint** 値を使用します。

    - **PRIMARY KEY** フィールドの値を記録します。この演習の後半でこの **key** 値を使用します。

      ![06](media/New-image9.png)

1. リソース メニューから **Data Explorer** を選択します。

     >**Note**: 右上に表示されるポップアップの "X" ボタンをクリックしてください。

1. **Data Explorer** ペインで **+ New Container (1)** を展開し、ドロップダウンから **+ New Database (2)** を選択します。

    ![06](media/New-image80.png)

1. **New Database** ポップアップで、各設定に次の値を入力し、**OK (5)** を選択します。

    | **Setting** | **Value** |
    | --: | :-- |
    | **Database id** | *``cosmicworks``* **(1)** |
    | **Provision throughput** | enabled **(2)** |
    | **Database throughput** | **Manual (3)** |
    | **Database Required RU/s** | ``1000`` **(4)** |

    ![06](media/c30.png)

1. **Data Explorer** ペインに戻り、階層内にある **cosmicworks** データベース ノードを確認します。

      ![06](media/New-image82.png)

1. **Data Explorer** ペインで **+ New Container (1)** > **+ New Container (2)** を選択します。

     ![06](media/New-image83.png)

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK (5)** を選択します。

    | **Setting** | **Value** |
    | :-- | :-- |
    | **Database id** | *Use existing (1)* &vert; *cosmicworks (2)* |
    | **Container id** | *``products`` (3)* |
    | **Partition key** | *``/category/name`` (4)* |

    ![06](media/c31.png)
    ![06](media/c32.png)

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認します。

1. **Data Explorer** ペインでもう一度 **+ New Container (1)** > **+ New Container (2)** を選択します。

    ![06](media/New-image85.png)

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択します。

    | **Setting** | **Value** |
    | :-- | :-- |
    | **Database id** | *Use existing* &vert; *cosmicworks* |
    | **Container id** | *``productslease``* |
    | **Partition key** | *``/id``* |

      ![06](media/New-image86.png)

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **productslease** コンテナー ノードを確認します。

     ![06](media/New-image87.png)

1. Azure portal の **Home** に戻ります。

    > **Congratulations** タスク完了です。次に検証を行います。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押します。成功メッセージが表示されたら次のタスクに進みます。
    > - 成功しない場合は、エラー メッセージをよく読み、ラボ ガイドに従って手順を再実行します。
    > - 支援が必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

    <validation step="8ec90c15-4d62-42be-8eff-76a215f8689b" />

## タスク 2: Application Insight を作成する

このタスクでは、Azure Function アプリケーションを監視するために Azure Application Insights を設定します。まず、監視データを保存する Log Analytics ワークスペースを作成します。次に、Application Insights インスタンスを作成し、Log Analytics ワークスペースにリンクして、アプリケーションのパフォーマンスとアクティビティを追跡できるようにします。

1. **Azure Portal** ページ上部の Search resources, services and docs (G+/) ボックスに **Log Analytics workspaces (1)** と入力し、services の **Log Analytics workspaces (2)** を選択します。

    ![06](media/New-image88.png)

1. **+ Create** を選択して、新しい *Log Analytics* ワークスペースを作成します。

    ![06](media/New-image89.png)

1. **Log Analytics workspace** ダイアログで、各設定に次の値を入力し、**Review + Create (5)** を選択します。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Subscription** | *Your existing Azure subscription* **(1)** |
    | **Resource group** | *Select an existing or create a new resource group* **(2)** |
    | **Name** | *``lab14laworkspace``* **(3)** |
    | **Location** | *Choose any available region* **(4)** |

     ![06](media/New-image90.png)

1. 次に **Create** を選択します。

1. *Log Analytics workspace* の作成完了後、検索ボックスで **Application Insights** を検索します。

    ![06](media/New-image91.png)

1. **+ Create** を選択して、新しい *Application Insight* を作成します。

1. **Application Insights** ダイアログで、各設定に次の値を入力し、**Review + Create (5)** を選択します。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Subscription (both entries)** | *Your existing Azure subscription* **(1)** |
    | **Resource group** | *Select an existing Cosmosdb-<inject key="DeploymentID" enableCopy="false"/>* **(2)** |
    | **Name** | **``lab14appinsight`` (3)** |
    | **Location** | **Choose any available region (4)** |
    | **Log Analytics Workspace** | **lab14laworkspace (5)** |

     ![06](media/New-image92.png)

1. 次に **Create** を選択します。

1. これで、アプリケーション関数を監視できるようになります。

    > **Congratulations** タスク完了です。次に検証を行います。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押します。成功メッセージが表示されたら次のタスクに進みます。
    > - 成功しない場合は、エラー メッセージをよく読み、ラボ ガイドに従って手順を再実行します。
    > - 支援が必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

    <validation step="d6eb6de3-1d82-40ff-a083-c08a5887fd64" />

## タスク 3: Azure Function アプリと Azure Cosmos DB トリガー関数を作成する

このタスクでは、Cosmos DB トリガー関数を含む Azure Function アプリを作成します。まず Azure portal で必要な構成を指定して Function アプリを設定します。デプロイ後、Azure Cosmos DB トリガー テンプレートを使用して新しい関数を作成し、Cosmos DB アカウントに接続してデータベースとコンテナーの詳細を指定します。これにより、関数が Cosmos DB コンテナー内の変更に応答できるようになります。

1. Azure portal のホーム ページで **+ Create a resource** を選択します。

     ![06](media/New-image95.png)

1. **Functions** を検索して **Function app** を選択します。次に、market place ページで **Function app** を選択します。

    ![06](media/New-image96.png)

    ![06](media/New-image97.png)

1. **Function App** ページで **Create** をクリックします。

    ![06](media/New-image98.png)

1. **Select a hosting option** ページで **App service (1)** を選択し、続けて **Select (2)** をクリックします。

    ![06](media/c33.png)

1. 次の設定を指定し、その他の設定は既定値のままにして **Review + Create (8)** を選択します。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Subscription** | *Your existing Azure subscription* **(1)** |
    | **Resource group** | *Select an existing or create a new resource group* **(2)** |
    | **Name** | **functionapp-<inject key="DeploymentID" enableCopy="false"/> (3)** |
    | **Publish** | **Code (4)** |
    | **Runtime stack** | **.NET (5)** |
    | **Version** | **8 (LTS) in-process model (6)** |
    | **Region** | *Choose any available region* **(7)** |

    ![06](media/c34.png)

1. 次に **Create** を選択します。

1. このタスクを続行する前に、デプロイの完了を待ちます。

1. **Go to resource** を選択し、新しく作成した **Azure Functions** アカウント リソースに移動します。

1. **Functions (1)** ペインに移動します。**Functions** ペインで **Create in Azure Portal (2)** を選択します。

    ![06](media/c35.png)

1. **Create function** ポップアップの **Select a template** タブで **Azure Cosmos DB trigger (1)** を選択し、**Next (2)** をクリックします。

    ![06](media/New-image104.png)

1. **Create function** ポップアップの **Template details** タブで、以下の設定で新しい関数を作成します。その他の設定は既定値のままにし、**Create (11)** を選択します。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Select a template** | *Azure Cosmos DB trigger* **(1)** |
    | **Function Name** | *``ItemsListener``* **(2)** |
    | **Cosmos DB account connection** | Select **New (3)** &vert; Select **Azure Cosmos DB Account (4)** &vert; Select the Azure Cosmos DB account you created earlier **(5)** |
    | **Database name** | *``cosmicworks``* **(7)** |
    | **Container name** | *``products``* **(8)** |
    | **Container name for leases** | *``productslease``* **(9)** |
    | **Create lease container if it does not exist** | *No* **(10)** |

    ![06](media/New-image105.png)

    ![06](media/New-image106.png)

    > **Congratulations** タスク完了です。次に検証を行います。手順は次のとおりです。
    > - 対応するタスクの Validate ボタンを押します。成功メッセージが表示されたら次のタスクに進みます。
    > - 成功しない場合は、エラー メッセージをよく読み、ラボ ガイドに従って手順を再実行します。
    > - 支援が必要な場合は cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。

    <validation step="3d866562-569f-41c5-8f1c-286e16e92767" />

## タスク 4: .NET で関数コードを実装する

このタスクでは、Azure portal 内の run.csx スクリプトを編集して、Azure Cosmos DB トリガー関数を実装します。必要なライブラリを参照し、変更された項目数をログ出力する Run メソッドを作成し、各項目を反復して一意識別子をログに出力します。コード保存後、ストリーミング ログに接続して、Cosmos DB コンテナーで項目が生成された際の出力を確認します。

先ほど作成した関数は、ポータル内で編集する C# スクリプトです。ここからはポータルを使用して、コンテナー内で挿入または更新された項目の一意識別子を出力する短い関数を作成します。

1. **ItemsListener** &vert; **Function** ペインで、**Code + Test** ペインに移動します。

1. **run.csx** スクリプトのエディターで、編集領域の内容を削除します。

1. エディター領域で **Microsoft.Azure.DocumentDB.Core** ライブラリを参照します。

    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    ```

1. **System**、**System.Collections.Generic**、および [Microsoft.Azure.Documents][docs.microsoft.com/dotnet/api/microsoft.azure.documents] 名前空間の using ブロックを追加します。

    ```
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    ```

1. **Run** という名前の新しい static メソッドを作成し、2 つのパラメーターを定義します。

    1. [Document][docs.microsoft.com/dotnet/api/microsoft.azure.documents.document] をジェネリック型とする **IReadOnlyList\<\>** 型の **input** パラメーター。

    1. **ILogger** 型の **log** パラメーター。

    ```
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
    }
    ```

1. **Run** メソッド内で、**log** 変数の **LogInformation** メソッドを呼び出し、現在のバッチ内の項目数を計算する文字列を渡します。

    ```
    log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}");
    ```

1. 引き続き **Run** メソッド内で、**input** 変数を反復処理する foreach ループを作成し、**Document** 型のインスタンスを表す変数として **item** を使用します。

    ```
    foreach(Document item in input)
    {
    }
    ```

1. **Run** メソッドの foreach ループ内で、**log** 変数の **LogInformation** メソッドを呼び出し、**item** 変数の [Id][docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id] プロパティを出力する文字列を渡します。

    ```
    log.LogInformation($"Detected Operation:\t{item.Id}");
    ```

1. 完了後、コード ファイルに次の内容が含まれていることを確認します。

    ```
    #r "Microsoft.Azure.DocumentDB.Core"

    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;

    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
        log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}");

        foreach(Document item in input)
        {
            log.LogInformation($"Detected Operation:\t{item.Id}");
        }
    }
    ```

    ![06](media/c36.png)

1. **Logs** セクションを展開し、現在の関数のストリーミング ログに接続します。

    > &#128161; ストリーミング ログ サービスへの接続には数秒かかる場合があります。接続が完了すると、ログ出力にメッセージが表示されます。

1. 現在の関数コードを **Save** します。

1. C# コード コンパイル結果を確認します。ログ出力の最後に **Compilation succeeded** メッセージが表示されるはずです。

    > &#128221; ログ出力に警告メッセージが表示される場合がありますが、このラボには影響しません。

1. ログ セクションを **Maximize** して、出力ウィンドウを可能な限り最大サイズに拡張します。

    > &#128221; Azure Cosmos DB for NoSQL コンテナーで項目を生成するために別ツールを使用します。項目生成後、このブラウザー ウィンドウに戻って出力を確認します。ブラウザー ウィンドウを途中で閉じないでください。

## タスク 5: Azure Cosmos DB for NoSQL アカウントにサンプル データを投入する

このタスクでは、**cosmicworks** データベースと **products** コンテナーを作成するコマンドライン ユーティリティを使用します。次にツールが項目セットを作成し、ターミナル ウィンドウで実行中の変更フィード プロセッサを使ってその内容を確認します。

1. Visual Studio Code を起動します（プログラム アイコンはデスクトップにピン留めされています）。

   ![Visual Studio Code Icon](./media/vscode1.jpg)

1. **Visual Studio Code** で、**... (ellipses) (1)** > **Terminal (2)** > **New Terminal (3)** を選択して **Terminal** メニューを開き、既存インスタンスと並べて新しいターミナルを開きます。

    ![06](media/terminal.png)

1. [cosmicworks][nuget.org/packages/cosmicworks] コマンドライン ツールをマシン全体で利用できるようにインストールします。

    ```
    dotnet tool install cosmicworks --global --version 1.*
    ```

    > &#128161; このコマンドの完了には数分かかる場合があります。過去にこのツールの最新バージョンをすでにインストールしている場合、このコマンドは警告メッセージ（*Tool 'cosmicworks' is already installed'）を出力します。

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

    > &#128221; たとえば endpoint が **https&shy;://dp420.documents.azure.com:443/**、key が **fDR2ci9QgkdkvERTQ==** の場合、コマンドは次のようになります。
    > ``cosmicworks --endpoint https://dp420.documents.azure.com:443/ --key fDR2ci9QgkdkvERTQ== --datasets product``

    - dotnet の互換性関連エラーが発生する場合は、次のコマンド `choco install dotnet-6.0-runtime dotnet-7.0-runtime -y` を実行し、完了するまで待機してください。

1. **Connection String** の入力を求められた場合は、CosmosDB の **Keys (1)** に移動し、**eye (2)** アイコンを選択して値 **(3)** をコピーしてください。

    ![06](media/c37.png)

1. その値をターミナルに貼り付けます。

    ![06](media/c38.png)

    >**Note:** このコマンド実行中にパーティション キー エラーが発生する場合があります。通常、products コンテナーが既に存在し、異なるパーティション キー構成で作成されていることが原因です。

    - このエラーは無視しても問題ありません。完全に解消するには、既存の products コンテナーを削除してコマンドを再実行してください。コンテナー削除後は、エラーなしでコマンドが正常に実行されます。

1. **cosmicworks** コマンドがデータベース、コンテナー、項目の投入を完了するまで待機します。

1. 統合ターミナルを閉じます。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

### まとめ

このラボを完了することで、Azure Cosmos DB のセットアップ、Azure Functions の作成、データ変更を処理する変更フィード プロセッサの実装に関する実践的な経験を得られました。これにより、Azure のサーバーレス アーキテクチャと NoSQL の機能を活用した応答性の高いアプリケーションを効果的に構築できるようになります。

### ラボは正常に完了しました