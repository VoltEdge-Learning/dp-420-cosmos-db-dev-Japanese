# Lab 08a - Azure Cosmos DB for NoSQL のデータ モデリングおよびパーティション戦略を実装する

## ラボ シナリオ

このラボでは、顧客エンティティを個別コンテナーとしてモデル化した場合と、NoSQL データベース向けに単一ドキュメントへ埋め込んでモデル化した場合の違いを測定します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: 顧客エンティティをクエリする。
- タスク 3: 顧客住所をクエリする。
- タスク 4: 顧客パスワードをクエリする。
- タスク 5: 要求課金を合計する。
- タスク 6: 埋め込みエンティティのパフォーマンスを測定する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab16.png)

## 演習 1: 分離コンテナーと埋め込みコンテナーのエンティティ パフォーマンスを測定する

### タスク 1: 開発環境を準備する

作業環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、次の手順に従ってください。すでにクローン済みの場合は、以前クローンしたフォルダーを **Visual Studio Code** で開いてください。

1. **Visual Studio Code** を起動してください。

    > &#128221; Visual Studio Code のインターフェイスに慣れていない場合は、[Get Started guide for Visual Studio Code][code.visualstudio.com/docs/getstarted] を確認してください。

1. Visual Studio Code を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

1. 左側ペインの **Extension (1)** アイコンを選択してください。検索バーに **C# (2)** と入力し、表示された **extension (3)** を選択して、最後に **Install (4)** を選択してください。

    ![](media/C-hash-extension.png)

1. ファイルを開き、左上のオプションから **file->Open Folder** をクリックして **C:\AllFiles** に移動してください。

1. **dp-420-cosmos-db-dev-stage** フォルダーを選択し、**Select Folder** をクリックしてください。

1. **Visual Studio Code** の **Explorer** ペインで **16-measure-performance** フォルダーに移動してください。

1. **16-measure-performance** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

1. ターミナルが **Windows Powershell** で開いた場合は、新しい **Git Bash** ターミナルを開いてください。

    > &#128161; **Git Bash** ターミナルを開くには、ターミナル メニュー右側の **+** の横にあるプルダウンをクリックし、*Git Bash* を選択してください。

1. **Git Bash terminal** で次のコマンドを実行してください。これらのコマンドを実行すると、ブラウザー ウィンドウが開いて Azure portal に接続されます。提供されたラボ資格情報を使用してサインインし、新しい Azure Cosmos DB アカウントを作成するスクリプトを実行した後、データベースを投入して演習を完了するためのアプリをビルドして起動します。*スクリプトが Azure アカウント資格情報の入力を求めてから、ビルド完了まで 15〜20 分かかる場合があります。コーヒーやお茶を用意するのにちょうど良い時間です*。

    ```
    az login
    cd 16-measure-performance
    bash init.sh
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    dotnet build
    dotnet run --load-data

    ```

1. 統合ターミナルを閉じてください。

## 分離コンテナーのエンティティ パフォーマンスを測定する

Database-v1 では、データは個別コンテナーに格納されています。このデータベースで、顧客、顧客住所、顧客パスワードを取得するクエリを実行し、それぞれの要求課金を確認します。

### タスク 2: 顧客エンティティをクエリする

Database-v1 で顧客エンティティを取得するクエリを実行し、要求課金を確認します。

1. 新しい Web ブラウザー ウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. Azure portal メニュー、または **Home** ページから **Azure Cosmos DB** を選択してください。
1. **cosmicworks** で始まる名前の Azure Cosmos DB アカウントを選択してください。
1. 左側の **Data Explorer** を選択してください。
1. **Database-v1** を展開してください。
1. **Customer** コンテナーを選択してください。
1. 画面上部で **New SQL Query** を選択してください。
1. 次の SQL テキストをコピーして貼り付け、**Execute Query** を選択してください。

    ```
    SELECT * FROM c WHERE c.id = "FFD0DD37-1F0E-4E2E-8FAC-EAF45B0E9447"
    ```

1. **Query Stats** タブを選択し、要求課金が 2.83 であることを確認してください。

    ![Screenshot that shows the query stats for customer query in the database.](media/17-customer-query-v1-1.png)

### タスク 3: 顧客住所をクエリする

顧客住所エンティティを取得するクエリを実行し、要求課金を確認します。

1. **CustomerAddress** コンテナーを選択してください。
1. 画面上部で **New SQL Query** を選択してください。
1. 次の SQL テキストをコピーして貼り付け、**Execute Query** を選択してください。

    ```
    SELECT * FROM c WHERE c.customerId = "FFD0DD37-1F0E-4E2E-8FAC-EAF45B0E9447"
    ```

1. **Query Stats** タブを選択し、要求課金が 2.83 であることを確認してください。

    ![Screenshot that shows the query stats for customer address query in the database.](media/17-customer-address-query-v1-1.png)

### タスク 4: 顧客パスワードをクエリする

顧客パスワード エンティティを取得するクエリを実行し、要求課金を確認します。

1. **CustomerPassword** コンテナーを選択してください。
1. 画面上部で **New SQL Query** を選択してください。
1. 次の SQL テキストをコピーして貼り付け、**Execute Query** を選択してください。

    ```
    SELECT * FROM c WHERE c.id = "FFD0DD37-1F0E-4E2E-8FAC-EAF45B0E9447"
    ```

1. **Query Stats** タブを選択し、要求課金が 2.83 であることを確認してください。

    ![Screenshot that shows the query stats for customer password query in the database.](media/17-customer-password-query-v1-1.png)

### タスク 5: 要求課金を合計する

ここまでですべてのクエリを実行したので、それぞれの Request Unit コストを合計します。

|**Query**|**RU/s cost**|
| --- | --- |
|Customer|2.83|
|Customer Address|2.83|
|Customer Password|2.83|
|**Total RU/s**|**8.49**|

### タスク 6: 埋め込みエンティティのパフォーマンスを測定する

次に、同じ情報を、エンティティが単一ドキュメントに埋め込まれた形でクエリします。

1. **Database-v2** データベースを選択してください。
1. **Customer** コンテナーを選択してください。
1. 次のクエリを実行してください。

    ```
    SELECT * FROM c WHERE c.id = "FFD0DD37-1F0E-4E2E-8FAC-EAF45B0E9447"
    ```

1. 返却されるデータが、顧客、住所、パスワードの階層データになっていることを確認してください。

    ![Screenshot that shows the query results for customer in the database.](media/17-customer-query-v2-1.png)

1. **Query Stats** を選択してください。要求課金が 2.83 であり、先ほど 3 つのクエリで実行した合計 8.49 RU/s より低いことを確認してください。

## 2 つのモデルのパフォーマンスを比較する

実行した各クエリの RU/s を比較すると、顧客エンティティを単一ドキュメントに格納した最後のクエリは、3 つのクエリを個別に実行した合計コストより大幅に低コストです。データが 1 回の操作で返されるため、レイテンシも低くなります。

単一アイテムを検索し、パーティション キーとデータ ID が分かっている場合は、Azure Cosmos DB SDK の `ReadItemAsync()` を呼び出す *point-read* でデータを取得できます。point-read はクエリよりさらに高速です。同じ顧客データでもコストは 1 RU/s で、ほぼ 3 倍の改善になります。

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- 顧客エンティティをクエリした。
- 顧客住所をクエリした。
- 顧客パスワードをクエリした。
- 要求課金を合計した。
- 埋め込みエンティティのパフォーマンスを測定した。

### ラボは正常に完了しました