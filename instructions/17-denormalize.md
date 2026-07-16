# Lab 08b - Azure Cosmos DB for NoSQL のデータ モデリングおよびパーティション戦略を実装する

## ラボ シナリオ

リレーショナル モデルでは、異なるエンティティをそれぞれ別のコンテナーに配置できます。一方、NoSQL データベースではコンテナー間の *join* がないため、*join* を使わずに済むようにデータの非正規化を進める必要があります。さらに NoSQL では、アプリケーションができるだけ少ないリクエストでデータを取得できるようにモデル化することで、リクエスト数を減らせます。データを非正規化する際の課題の 1 つは、エンティティ間の参照整合性です。この課題に対しては、変更フィードを使ってデータの同期を維持できます。group by の件数のような集計を非正規化することでも、リクエスト削減に役立ちます。

このラボでは、データと集計を非正規化することでコストを削減できる利点と、変更フィードを使用して非正規化データの参照整合性を維持する方法を確認します。

## ラボの目的

このラボでは、次の演習を完了します。
- 演習 1: データ非正規化時のパフォーマンス コストを測定する。
- 演習 2: 変更フィードを使用して参照整合性を管理する。
- 演習 3: 集計を非正規化する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab17.png)

## ラボ: データと集計の非正規化コスト、および変更フィードによる参照整合性の維持

## 開発環境を準備する

作業環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、次の手順に従ってください。すでにクローン済みの場合は、以前クローンしたフォルダーを **Visual Studio Code** で開いてください。

1. **Visual Studio Code** を起動してください。

    > &#128221; Visual Studio Code のインターフェイスに慣れていない場合は、[Get Started guide for Visual Studio Code][code.visualstudio.com/docs/getstarted] を確認してください。

1. コマンド パレットを開き、**Git: Clone** を実行して ``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリを任意のローカル フォルダーにクローンしてください。

    > &#128161; **CTRL+SHIFT+P** キーボード ショートカットでコマンド パレットを開けます。

1. リポジトリのクローンが完了したら、選択したローカル フォルダーを **Visual Studio Code** で開いてください。

1. **Visual Studio Code** の **Explorer** ペインで **17-denormalize** フォルダーに移動してください。

1. **17-denormalize** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. ターミナルが **Windows Powershell** で開いた場合は、新しい **Git Bash** ターミナルを開いてください。

    > &#128161; **Git Bash** ターミナルを開くには、ターミナル メニュー右側の **+** の横にあるプルダウンをクリックし、*Git Bash* を選択してください。

1. **Git Bash terminal** で次のコマンドを実行してください。これらのコマンドを実行すると、ブラウザー ウィンドウが開いて Azure portal に接続されます。提供されたラボ資格情報を使用してサインインし、新しい Azure Cosmos DB アカウントを作成するスクリプトを実行した後、データベースを投入して演習を完了するためのアプリをビルドして起動します。*スクリプトが Azure アカウント資格情報の入力を求めてから、ビルド完了まで 15〜20 分かかる場合があります。コーヒーやお茶を用意するのにちょうど良い時間です*。

    ```
    az login
    cd 17-denormalize
    bash init.sh
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    dotnet build
    dotnet run --load-data

    ```

1. 統合ターミナルを閉じてください。

## 演習 1: データ非正規化時のパフォーマンス コストを測定する

### タスク 1: 製品カテゴリ名をクエリする

データが個別コンテナーに格納されている **database-v2** で、製品カテゴリ名を取得するクエリを実行し、そのクエリの要求課金を確認します。

1. 新しい Web ブラウザー ウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. 左側ペインで **Azure Cosmos DB** を選択してください。
1. **cosmicworks** で始まる名前の Azure Cosmos DB アカウントを選択してください。
1. 左側ペインで **Data Explorer** を選択してください。
1. **database-v2** を展開してください。
1. **productCategory** コンテナーを選択してください。
1. ページ上部で **New SQL Query** を選択してください。
1. **Query 1** ペインで、次の SQL コードを貼り付けて **Execute Query** を選択してください。

    ```
    SELECT * FROM c where c.type = 'category' and c.id = "AB952F9F-5ABA-4251-BC2D-AFF8DF412A4A"
    ```

1. **Results** タブを選択して結果を確認してください。このクエリにより、製品カテゴリ名 "Components, Headsets." が返されます。

    ![Screenshot that shows the results of the query to the product category container.](media/16-product-category-results.png)

1. **Query Stats** タブを選択し、要求課金が 2.93 RU（request units）であることを確認してください。

    ![Screenshot of the query stats for the query you ran in Data Explorer.](media/16-product-category-stats.png)

### タスク 2: カテゴリ内の製品をクエリする

次に、"Components, Headsets" カテゴリ内のすべての製品を取得するために product コンテナーをクエリします。

1. **product** コンテナーを選択してください。
1. ページ上部で **New SQL Query** を選択してください。
1. **Query 2** ペインで、次の SQL コードを貼り付けて **Execute Query** を選択してください。

    ```
    SELECT * FROM c where c.categoryId = "AB952F9F-5ABA-4251-BC2D-AFF8DF412A4A"
    ```

1. **Results** タブを選択して結果を確認してください。HL Headset、LL Headset、ML Headset の 3 製品が返されます。各製品には SKU、名前、価格、および product tags の配列があります。

1. **Query Stats** タブを選択し、要求課金が 2.9 RU であることを確認してください。

    ![Screenshot of Azure Cosmos DB Data Explorer that shows the results of the query to the product container.](media/16-product-results.png)

### タスク 3: 各製品のタグをクエリする

次に、3 つの製品（HL Headset、LL Headset、ML Headset）それぞれについて、productTag コンテナーを 3 回クエリします。

#### HL headset tags

まず、HL Headset のタグを返すクエリを実行します。

1. **productTag** コンテナーを選択してください。
1. ページ上部で **New SQL Query** を選択してください。
1. **Query 3** ペインで、次の SQL コードを貼り付けて **Execute Query** を選択してください。

    ```
    SELECT * FROM c where c.type = 'tag' and c.id IN ('87BC6842-2CCA-4CD3-994C-33AB101455F4', 'F07885AF-BD6C-4B71-88B1-F04295992176')
    ```

    このクエリは HL Headset 製品の 2 つのタグを返します。

1. **Query Stats** タブを選択し、要求課金が 3.06 RU であることを確認してください。

    ![Screenshot of the results of the query to the product tag container for hl headsets query stats.](media/16-product-tag-hl-stats.png)

#### LL headset tags

次に、LL Headset のタグを返すクエリを実行します。

1. **productTag** コンテナーを選択してください。
1. ページ上部で **New SQL Query** を選択してください。
1. **Query 4** ペインで、次の SQL コードを貼り付けて **Execute Query** を選択してください。

    ```
    SELECT * FROM c where c.type = 'tag' and c.id IN ('18AC309F-F81C-4234-A752-5DDD2BEAEE83', '1B387A00-57D3-4444-8331-18A90725E98B', 'C6AB3E24-BA48-40F0-A260-CB04EB03D5B0', 'DAC25651-3DD3-4483-8FD1-581DC41EF34B', 'E6D5275B-8C42-47AE-BDEC-FC708DB3E0AC')
    ```

    このクエリは LL Headset 製品の 5 つのタグを返します。

1. **Query Stats** タブを選択し、要求課金が 3.47 RU であることを確認してください。

    ![Screenshot of the results of the query to the product tag container for 'LL Headset' query stats.](media/16-product-tag-ll-stats.png)

#### ML headset tags

最後に、ML Headset のタグを返すクエリを実行します。

1. **productTag** コンテナーを選択してください。
1. ページ上部で **New SQL Query** を選択してください。
1. **Query 5** ペインで、次の SQL コードを貼り付けて **Execute Query** を選択してください。

    ```
    SELECT * FROM c where c.type = 'tag' and c.id IN ('A34D34F7-3286-4FA4-B4B0-5E61CCEEE197', 'BA4D7ABD-2E82-4DC2-ACF2-5D3B0DEAE1C1', 'D69B1B6C-4963-4E85-8FA5-6A3E1CD1C83B')
    ```

    このクエリは ML Headset 製品の 3 つのタグを返します。

1. **Query Stats** タブを選択し、要求課金が 3.2 RU であることを確認してください。

    ![Screenshot of the results of our query to the product tag container for 'ML Headset' query stats.](media/16-product-tag-ml-stats.png)

### RU 課金を合計する

ここまでに実行した各クエリの RU コストを合計します。

|**Query**|**RU/s cost**|
|---------|---------|
|Category name|2.93|
|Product|2.9|
|HL product tags|3.06|
|LL product tags|3.47|
|ML product tags|3.2|
|**Total RU cost**|**15.56**|

### NoSQL 設計で同じクエリを実行する

次に、非正規化されたデータベースで同じ情報をクエリします。

1. Data Explorer で **database-v3** を選択してください。
1. **product** コンテナーを選択してください。
1. ページ上部で **New SQL Query** を選択してください。
1. **Query 6** ペインで、次の SQL コードを貼り付けて **Execute Query** を選択してください。

    ```
   SELECT * FROM c where c.categoryId = "AB952F9F-5ABA-4251-BC2D-AFF8DF412A4A"
   ```

    結果は次のようになります。

    ![Screenshot of the results of the query to the product container in the newly modeled product container.](media/16-product-query-v2.png)

1. このクエリで返されるデータを確認してください。3 製品それぞれのカテゴリ名とタグ名を含み、このカテゴリの製品を表示するために必要な情報がすべて含まれています。

1. **Query Stats** タブを選択し、要求課金が 2.9 RU であることを確認してください。

### 2 つのモデルのパフォーマンスを比較する

リレーショナル モデル（データを個別コンテナーに格納）では、カテゴリ名、そのカテゴリの全製品、各製品の全タグを取得するために 5 つのクエリを実行しました。5 クエリの要求課金合計は 15.56 RU でした。

同じ情報を NoSQL モデルで取得するには、1 つのクエリのみを実行し、要求課金は 2.9 RU でした。

このモデルの利点は、NoSQL 設計による低コストだけではありません。必要なリクエストが 1 回だけなので、処理も高速です。さらに、データは Web ページで表示される形に近い状態で提供されます。これは、e コマース アプリケーション下流で記述および保守するコードが少なくなることを意味します。

データを非正規化すると、e コマース アプリケーション向けのクエリをよりシンプルかつ効率的にできます。アプリケーションに必要なデータを 1 つのコンテナーに格納し、1 つのクエリで取得できます。高同時実行のクエリを扱う場合、この種のデータ モデリングはシンプルさ、速度、コストの面で大きな利点をもたらします。

---

## 演習 2: 変更フィードを使用して参照整合性を管理する

この単元では、Azure Cosmos DB の 2 つのコンテナー間で参照整合性を維持するのに変更フィードがどのように役立つかを確認します。このシナリオでは、productCategory コンテナーを変更フィードで監視します。製品カテゴリ名を更新すると、変更フィードが更新後の名前を検出し、そのカテゴリに属するすべての製品を新しい名前で更新します。

この演習では、次の手順を完了します。

- 理解しておくべき重要概念を示す C# コードを完成させる。
- 変更フィード プロセッサを開始して productCategory コンテナーの監視を開始する。
- 名前を変更するカテゴリと、そのカテゴリ内の製品数を product コンテナーでクエリする。
- カテゴリ名を更新し、変更フィードがその変更を product コンテナーへ反映することを確認する。
- 新しいカテゴリ名で product コンテナーをクエリし、製品数を数えてすべて更新されたことを確認する。
- 名前を元に戻し、変更フィードがその変更を再反映することを確認する。

### Azure Cloud Shell を開始して Visual Studio Code を開く

変更フィード向けに更新するコードへ移動するには、次を実行してください。

1. まだ開いていない場合は Visual Studio Code を開き、*17-denormalize* フォルダー内の *Program.cs* ファイルを開いてください。

### タスク 1: 変更フィード用コードを完成させる

デリゲートへ渡される変更を処理するコードを追加し、そのカテゴリ内の各製品をループして更新します。

1. 変更フィード プロセッサを開始する関数に移動してください。

1. Ctrl+G を押して **603** と入力し、ファイル内の該当行へ移動してください。

1. 次のコードが表示されることを確認してください。

    ![Screenshot of Cloud Shell, displaying the function where change feed has been implemented.](media/16-change-feed-function.png)

   588 行目と 589 行目には 2 つのコンテナー参照があります。正しいコンテナー名に更新する必要があります。変更フィードは、コンテナー参照に対して変更フィード プロセッサのインスタンスを作成して動作します。このケースでは productCategory コンテナーの変更を監視します。

1. 588 行目の **{container to watch}** を `productCategory` に置き換えてください。

1. 589 行目の **{container to update}** を `product` に置き換えてください。製品カテゴリ名が更新されたとき、そのカテゴリのすべての製品を新しいカテゴリ名で更新する必要があります。

    ![image](media/DP-420-m8-17.png)

1. *container to watch* と *container to update* の行の下にある *leaseContainer* 行を確認してください。leaseContainer はコンテナーに対するチェックポイントとして機能します。変更フィード プロセッサが前回確認してから何が更新されたかを把握します。

   変更フィードが新しい変更を検出すると、デリゲートを呼び出し、読み取り専用コレクションで変更を渡します。

1. 603 行目で、変更フィードが処理対象の新しい変更を検出したときに呼び出されるコードを追加してください。次のコード スニペットをコピーし、**//To-Do:** で始まる行の下に貼り付けてください。

    ```
    //Fetch each change to productCategory container
    foreach (ProductCategory item in input)
    {
        string categoryId = item.id;
        string categoryName = item.name;

        tasks.Add(UpdateProductCategoryName(productContainer, categoryId, categoryName));
    }
    ```

1. コードが次の画像のようになっていることを確認してください。

    ![Screenshot of the Cloud Shell window that displays the fully completed code for change feed.](media/16-change-feed-function-delegate-code.png)

    既定では、変更フィードは 1 秒ごとに実行されます。監視対象コンテナーで挿入や更新が多いシナリオでは、デリゲートに複数の変更が渡される場合があります。このため、デリゲートの **input** は **IReadOnlyCollection** 型にします。

    このコード スニペットはデリゲート **input** 内のすべての変更をループし、**categoryId** と **categoryName** の文字列として保持します。その後、別関数を呼び出して product コンテナー内のカテゴリ名を更新するタスクをタスク リストへ追加します。

1. Ctrl+G を押して **647** を入力し、**UpdateProductCategoryName()** 関数を表示してください。ここで、変更フィードが検出した新しいカテゴリ名で product コンテナー内の各製品を更新するコードを記述します。

1. 次のコード スニペットをコピーし、**//To-Do:** で始まる行の下に貼り付けてください。この関数は 2 つの処理を行います。まず渡された **categoryId** に対するすべての製品を product コンテナーからクエリし、次に各製品を新しいカテゴリ名で更新します。

    ```
    //Loop through all products
    foreach (Product product in response)
    {
        productCount++;
        //update category name for product
        product.categoryName = categoryName;

        //write the update back to product container
        await productContainer.ReplaceItemAsync(
            partitionKey: new PartitionKey(categoryId),
            id: product.id,
            item: product);
    }
    ```

    コードが次のようになっていることを確認してください。

    ![Screenshot of Cloud Shell, showing the fully implemented update product category name function called by the change feed function.](media/16-change-feed-function-update-product.png)

    このコードはクエリの response オブジェクトから行を読み取り、クエリで返されたすべての製品で product コンテナーを更新します。

    **foreach()** ループを使って、クエリで返された各製品を処理します。各行ごとにカウンターを更新して、更新された製品数を把握します。続いて、その製品のカテゴリ名を新しい **categoryName** に更新します。最後に **ReplaceItemAsync()** を呼び出して product コンテナーへ更新を反映します。

1. Ctrl+S を押して変更を保存してください。

1. まだ開いていない場合は Git Bash 統合ターミナルを開き、*17-denormalize* フォルダー配下にいることを確認してください。

1. プロジェクトをコンパイルして実行するには、次のコマンドを実行してください。

    ```
    dotnet build
    dotnet run
    ```

1. アプリケーションのメイン メニューが表示されることを確認してください。

    ![Screenshot that shows the main menu for the application with multiple options for working with the data.](media/16-main-menu.png)

### タスク 2: 変更フィード サンプルを実行する

変更フィード用コードの完成後、実際の動作を確認します。

1. メイン メニューで **a** を選択し、変更フィード プロセッサを開始してください。画面に進行状況が表示されます。

    ![Screenshot of the output of the application as it builds and then starts change feed.](media/16-change-feed-start.png)

1. 任意のキーを押してメイン メニューへ戻ってください。

1. メイン メニューで **b** を選択して製品カテゴリ名を更新してください。次のシーケンスが実行されます。

    a. products コンテナーで "Accessories, Tires, and Tubes" カテゴリをクエリし、そのカテゴリ内の製品数をカウントします。  
    b. カテゴリ名を更新し、"and" をアンパサンド（&）に置き換えます。  
    c. 変更フィードがこの変更を検出し、記述したコードを使ってそのカテゴリのすべての製品を更新します。  
    d. 変更フィードが名前変更を元に戻し、"&" を元の "and" に戻します。  
    e. 変更フィードがこの変更を検出し、すべての製品を元のカテゴリ名に更新し直します。

1. メイン メニューで **b** を選択し、変更フィードが 2 回目に実行されるまでプロンプトに従ってください。結果は次のようになります。

    ![Screenshot of the output of the application as the category name is changed.](media/16-change-feed-update-category-name.png)

1. 進め過ぎてメイン メニューに戻った場合は、再度 **b** を選択して変更を確認してください。

1. 完了したら **x** を入力して終了し、Cloud Shell に戻ってください。

---

## 演習 3: 集計の非正規化

この単元では、e コマース サイトの上位 10 顧客クエリを記述するために、集計をどのように非正規化するかを確認します。Azure Cosmos DB .NET SDK の transactional batch 機能を使用し、同一の論理パーティション内で新しい sales order の挿入と顧客の **salesOrderCount** プロパティ更新を同時に実行します。

この演習では、次の手順を完了します。

- 新しい sales order を作成するコードを確認する。
- 顧客の *salesOrderCount* を増分する C# コードを完成させる。
- *transactional batch* を使用して、新しい sales order 挿入と顧客レコード更新のトランザクションを実装する C# コードを完成させる。
- 特定顧客をクエリし、顧客レコードとその顧客の全注文を確認する。
- 同じ顧客の新しい sales order を作成し、**salesOrderCount** プロパティを更新する。
- 上位 10 顧客クエリを実行して現在の結果を確認する。
- 顧客が注文をキャンセルした場合に transactional batch をどう使えるかを確認する。

## Visual Studio Code を開く

この単元で使用するコードへ移動するには、次を実行してください。

1. まだ開いていない場合は Visual Studio Code を開き、*17-denormalize* フォルダー内の *Program.cs* ファイルを開いてください。

### タスク 1: 総売上注文数を更新するコードを完成させる

1. 新しい sales order を作成する関数へ移動してください。

1. Ctrl+G を押して **483** と入力し、ファイル内の該当行へ移動してください。

1. 次のコードが表示されることを確認してください。

    ![Screenshot of Cloud Shell that shows the create new order and update customer order total function.](media/16-create-order-function.png)

    この関数は transactional batch を使用して、新しい sales order を作成し、顧客レコードを更新します。

    最初に **ReadItemAsync()** を呼び出し、**customerId** をパーティション キーと ID の両方として渡して顧客レコードを取得します。

1. 483 行目の **//To-Do:** コメントの下に、次のコード スニペットを貼り付けて **salesOrderCount** の値を増分してください。

    ```
    //Increment the salesOrderTotal property
    customer.salesOrderCount++;
    ```

    画面が次のようになっていることを確認してください。

    ![Screenshot of the create new order and update customer total function with the line of code to increment sales order count by one.](media/16-create-order-sales-order-count.png)

### タスク 2: transactional batch 実装コードを完成させる

1. 数行下へスクロールし、顧客向けに作成する新しい sales order のデータを確認してください。

    新しい sales order オブジェクトは、e コマース アプリケーションで一般的な sales order のヘッダーと明細の構造を持っています。

    sales order ヘッダーには **orderId**、**customerId**、**orderDate**、**shipDate** があり、これらは空のままにします。

    customer コンテナーには customer と sales order の両エンティティが含まれるため、sales order オブジェクトには discriminator プロパティ **type**（値は **salesOrder**）も含まれます。これにより customer コンテナー内で sales order と customer オブジェクトを区別できます。

    さらに下へ進むと、sales order の明細セクションを構成する 2 つの製品も確認できます。

1. さらに少しスクロールして別の **//To-Do:** コメントへ移動してください。ここでは transactional batch を使って新しい sales order を挿入し、顧客レコードを更新するコードを追加します。

1. 次のコード スニペットをコピーし、**//To-Do:** コメントの直下に貼り付けてください。

    ```
    TransactionalBatchResponse txBatchResponse = await container.CreateTransactionalBatch(
        new PartitionKey(salesOrder.customerId))
        .CreateItem<SalesOrder>(salesOrder)
        .ReplaceItem<CustomerV4>(customer.id, customer)
        .ExecuteAsync();

    if (txBatchResponse.IsSuccessStatusCode)
        Console.WriteLine("Order created successfully");
    ```

    このコードは container オブジェクトに対して **CreateTransactionalBatch()** を呼び出します。すべてのトランザクションは単一の論理パーティションにスコープされるため、必須パラメーターとしてパーティション キー値を渡します。さらに、新しい sales order を **CreateItem()** で渡し、更新した customer オブジェクトを **ReplaceItem()** で渡します。その後、**ExecuteAsync()** を呼び出してトランザクションを実行します。

    最後に、response オブジェクトを確認してトランザクション成功を判定します。

    画面が次のようになっていることを確認してください。

    ![Screenshot of Cloud Shell, showing that the transactional batch code is now implemented in your function.](media/16-create-order-transactional-batch.png)

1. Ctrl+S を押して変更を保存してください。

1. まだ開いていない場合は Git Bash 統合ターミナルを開き、*17-denormalize* フォルダー配下にいることを確認してください。

1. プロジェクトをコンパイルして実行するには、次のコマンドを実行してください。

    ```
    dotnet build
    dotnet run
    ```

1. ここに示すように、アプリケーションのメイン メニューが表示されることを確認してください。

    ![Screenshot that shows the main menu for the application with multiple options for working with the data.](media/16-main-menu.png)

### タスク 3: 顧客とその sales orders をクエリする

このデータベースは **customerId** をパーティション キーとして使い、customer とその顧客の全 sales order を同じコンテナーに格納する設計にしているため、customer コンテナーをクエリするだけで顧客レコードとその顧客の全 sales order を 1 回の操作で返せます。

1. メイン メニューで **c** を選択し、**Query for customer and all orders** メニュー項目を実行してください。このクエリは顧客レコードに続いて、その顧客の全 sales order を返します。画面上に全 sales order の出力が表示されることを確認してください。

   最後の注文が **Road-650 Red, 58**（$782.99）であることに注目してください。

1. **Print out customer record and all their orders** までスクロールしてください。

   **salesOrderCount** プロパティが sales order 2 件を示していることに注目してください。

   画面は次のようになります。

    ![Screenshot of Cloud Shell, showing the output of the query customer and orders query with a customer record and two sales orders.](media/16-query-customer-and-orders-initial.png)

### タスク 4: 新しい sales order を作成し、トランザクションで総売上注文数を更新する

同じ顧客に対して新しい sales order を作成し、顧客レコードに保存されている総売上注文数を更新します。

1. ウィンドウで任意のキーを押し、メイン メニューへ戻ってください。
1. **d** を選択し、**Create new order and update order total** メニュー項目を実行してください。
1. 任意のキーを押してメイン メニューへ戻ってください。
1. **c** を選択して同じクエリを再実行してください。

   新しい sales order として **HL Mountain Frame - Black, 38** および **Racing Socks, M** が表示されることに注目してください。

1. **Print out customer record and all their orders** まで再度スクロールしてください。

   **salesOrderCount** プロパティが sales order 3 件を示していることに注目してください。

1. 画面が次のようになっていることを確認してください。

    ![Screenshot of Cloud Shell, with an updated customer record showing a value of 3 for the sales order count and three sales orders below it.](media/16-query-customer-and-orders-next.png)

### タスク 5: transactional batch を使って注文を削除する

一般的な e コマース アプリケーションと同様に、顧客は注文をキャンセルすることがあります。ここでも同じ処理を実行できます。

1. 任意のキーを押してメイン メニューへ戻ってください。

1. **f** を選択し、**Delete order and update order total** メニュー項目を実行してください。

1. 任意のキーを押してメイン メニューへ戻ってください。
1. **c** を選択して同じクエリを再実行し、顧客レコードが更新されたことを確認してください。

   新しい注文が返されなくなっていることに注目してください。上へスクロールすると **salesOrderCount** の値が **2** に戻っていることを確認できます。

1. 任意のキーを押してメイン メニューへ戻ってください。

### タスク 6: sales order を削除するコードを確認する

sales order の削除は作成とまったく同じ方法で行います。どちらの操作もトランザクションでラップされ、同じ論理パーティションで実行されます。実装コードを確認しましょう。

1. **x** を入力してアプリケーションを終了してください。
1. まだ開いていない場合は Visual Studio Code を開き、*17-denormalize* フォルダー内の *Program.cs* ファイルを開いてください。

1. Ctrl+G を押して **529** を入力してください。

    この関数は新しい sales order を削除し、顧客レコードを更新します。

    ここでは、コードが最初に顧客レコードを取得し、その後 **salesOrderCount** を 1 減らしていることを確認できます。

    続いて **CreateTransactionalBatch()** を呼び出します。ここでも論理パーティション キー値を渡しますが、今回は **DeleteItem()** に order ID を渡し、**ReplaceItem()** に更新済み customer レコードを渡します。

### タスク 7: 上位 10 顧客クエリのコードを確認する

上位 10 顧客クエリを確認しましょう。

1. Ctrl+G を押して **566** を入力してください。

    上部付近にクエリ定義があります。

    ```
    SELECT TOP 10 c.firstName, c.lastName, c.salesOrderCount
        FROM c WHERE c.type = 'customer'
        ORDER BY c.salesOrderCount DESC
    ```

    このクエリは比較的シンプルです。返却レコード数を制限する **TOP** 句と、**salesOrderCount** プロパティを降順で並べる **ORDER BY** を使用しています。

    また、**type** の discriminator プロパティが **customer** である点にも注目してください。customer コンテナーには customer と sales order の両方が含まれているため、customer のみを返すために使われています。

1. まだアプリケーションが起動していない場合は、次のコマンドを実行して再度起動してください。

    ```
    dotnet run
    ```

1. 最後に **e** を入力してクエリを実行してください。

    ![Screenshot of Cloud Shell, showing the output for your top 10 customers query.](media/16-top-10-customers.png)

    この上位 10 顧客クエリは、コンテナー内のすべてのパーティションにファンアウトするクロスパーティション クエリである点に気づかないかもしれません。

    このラボの関連ラボでは、クロスパーティション クエリはできるだけ避けるべきと説明していました。ただし実際には、コンテナーがまだ小さい場合やクエリ実行頻度が低い場合は許容できることがあります。クエリ実行頻度が高い、またはコンテナーが非常に大きい場合は、このデータを別コンテナーにマテリアライズしてそのクエリ提供に使用するコストを検討する価値があります。

### レビュー

このラボでは、次を完了しました。

- データ非正規化時のパフォーマンス コストを測定した。
- 変更フィードを使用して参照整合性を管理した。
- 集計を非正規化した。

### ラボは正常に完了しました