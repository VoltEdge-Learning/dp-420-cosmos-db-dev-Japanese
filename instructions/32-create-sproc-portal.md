# Azure portal でストアド プロシージャを作成する

## ラボ シナリオ

ストアド プロシージャは、Azure Cosmos DB でサーバー側のビジネス ロジックを実行する方法の 1 つです。ストアド プロシージャを使用すると、単一のトランザクション スコープ内で、コンテナー内の複数ドキュメントに対して基本的な CRUD（Create、Read、Update、Delete）操作を実行できます。

このラボでは、コンテナー内にドキュメントを作成するストアド プロシージャを作成します。次に SQL クエリを使用して、ストアド プロシージャの実行結果を検証します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: ストアド プロシージャを作成する。
- タスク 2: ストアド プロシージャのベスト プラクティスを実装する。
- タスク 3: ドキュメントをクエリする。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab31.png)

## 演習 1:

### タスク 1: ストアド プロシージャを作成する

このタスクでは、Azure Cosmos DB SQL API アカウントをプロビジョニングします。

ストアド プロシージャは、言語統合された JavaScript で作成され、データベース エンジン内で基本的な CRUD 操作の実行をサポートします。データベース エンジン内で実行される JavaScript は、Azure Cosmos DB のサーバー側 JavaScript SDK と一連のヘルパー メソッドによって実現されています。

1. Azure Portal ページに戻ってください。ポータル上部の Search resources, services and docs (G+/) ボックスに **Azure Cosmos DB** と入力し、services の **Azure Cosmos DB** を選択してください。

   ![06](media/New-image1.png)
   
1. **Azure Cosmos DB for NoSQL** の下にある **+ Create** を選択し、**Create** をクリックして **Azure Cosmos DB for NoSQL** アカウントを作成してください。

    ![06](media/New-image2.png)

    ![06](media/New-image3.png)

1. 次の設定を指定し、それ以外の設定は既定値のままにして、**Review + create** を選択してください。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Subscription** | *Your existing Azure subscription* |
    | **Resource group** | **Cosmosdb-<inject key="DeploymentID" enableCopy="false"/>** |
    | **Account Name** | *Enter a globally unique name* |
    | **Location** | *Choose any available region* |
    | **Capacity mode** | *Provisioned throughput* |
    | **Apply Free Tier Discount** | *Do Not Apply* |

1. 検証が Success になったら、**Create** をクリックしてください。

1. このタスクを続行する前に、デプロイ タスクが完了するまで待機してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、左側のナビゲーション メニューから **Data Explorer** ペインに移動してください。

1. **Data Explorer** で **+ New Container** > **+ New Container** を選択してください。

     ![06](media/New-image85.png)

 1. 次の設定で新しいコンテナーを作成し、それ以外の設定は既定値のままにして、**OK** を選択してください。

    | **Setting** | **Value** |
    | :--- | :--- |
    | **Database id** | *Create new* &vert; *cosmicworks* |
    | **Share throughput across containers** | *Select this option* |
    | **Database throughput** | *Manual* &vert; *400* |
    | **Container id** | *products* |
    | **Indexing** | *Automatic* |
    | **Partition key** | */categoryId* |

1. **Data Explorer** 内で **cosmicworks** データベース ノードを展開し、ナビゲーション ツリー内の新しい **products** コンテナー ノードを選択してください。

1. **New Stored Procedure** を選択してください。

   ![06](media/New-image123.png)

1. **Stored Procedure Id** フィールドに **createDoc** を入力してください。

    ![06](media/New-image124.png)

1. エディター領域の内容を削除してください。

1. 入力パラメーターを持たない **createDoc** という名前の新しい JavaScript 関数を作成してください。

    ```
    function createDoc() {
        
    }
    ```

1. **createDoc** 関数内で組み込みの [getContext][azure.github.io/azure-cosmosdb-js-server/global.html] メソッドを呼び出し、結果を **context** という変数に格納してください。

    ```
    var context = getContext();
    ```

1. context オブジェクトの [getCollection][azure.github.io/azure-cosmosdb-js-server/context.html] メソッドを呼び出し、結果を **container** という変数に格納してください。

    ```
    var container = context.getCollection();
    ```

1. 2 つのプロパティを持つ **doc** という名前の新しいオブジェクトを作成してください。

    | **Property** | **Value** |
    | :--- | :--- |
    | **Name** | *first document* |
    | **Category ID** | *demo* |

    ```
    var doc = {
        name: 'first document',
        categoryId: 'demo'
    };
    ```

1. container オブジェクトの **getSelfLink** メソッド呼び出し結果と新しいドキュメントをパラメーターとして渡し、container オブジェクトの **createDocument** メソッドを呼び出してください。

    ```
    container.createDocument(
      container.getSelfLink(),
      doc
    );
    ```

1. 完了後、ストアド プロシージャのコードは次の内容になります。

    ```
    function createDoc() {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: 'first document',
        categoryId: 'demo'
      };
      container.createDocument(
        container.getSelfLink(),
        doc
      );
    }
    ```

1. **Save** を選択して、ストアド プロシージャへの変更を保存してください。

    ![06](media/New-image125.png)

1. **Execute** を選択し、次の入力パラメーターでストアド プロシージャを実行してください。

    | **Setting** | **Key** | **Value** |
    | :--- | :--- | :--- |
    | **Partition key value** | *String* | *demo* |

1. 空の結果を確認してください。ストアド プロシージャは正常に実行されましたが、JavaScript コードが人間可読な応答を返していません。


> ラボの完了おめでとうございます。ここで検証を行います。手順は次のとおりです。
> - 対応するタスクの Validate ボタンを押してください。成功メッセージが表示された場合、ラボの検証は完了です。
> - 失敗した場合は、エラー メッセージを注意深く読み、ラボ ガイドの手順に沿って再試行してください。
> - サポートが必要な場合は、cloudlabs-support@spektrasystems.com までご連絡ください。24 時間 365 日対応しています。
    
<validation step="f6406f6b-cf21-4093-a8e1-512fadade041" />

### タスク 2: ストアド プロシージャのベスト プラクティスを実装する

このタスクでは、エラー処理、応答処理、パラメーター管理を改善するために、ベスト プラクティスを実装してストアド プロシージャを強化します。

このラボの前半で作成したストアド プロシージャには基本機能がありますが、すべてのストアド プロシージャで実装すべき一般的なエラー処理手法が不足しています。1 つ目に、処理完了まで常に時間があると仮定しており、**createDocument** メソッドの戻り値を確認して実行時間の余裕を確認していません。2 つ目に、潜在的なエラー メッセージの確認やスローを行わず、すべてのドキュメントが正常に挿入されると仮定しています。最後に、ストアド プロシージャを呼び出した元の要求に対して、新規作成ドキュメントを HTTP 応答として返していません。これら 3 点を修正し、一般的なベスト プラクティスを実装します。

1. **createDoc** ストアド プロシージャのエディターに戻ってください。

1. **createDoc** 関数を定義しているコードの 1 行目を見つけてください。

    ```
    function createDoc() {
    ```

    次に、**title** というパラメーターを含むようにこの行を更新してください。

    ```
    function createDoc(title) {
    ```

1. **doc** オブジェクトの **name** プロパティを設定しているコードの 5 行目を見つけてください。

    ```
    name: 'first document',
    ```

    次に、**title** パラメーターの値を使用するようにこの行を更新してください。

    ```
    name: title,
    ```

1. **createDocument** メソッドを呼び出しているコードの 8 行目を見つけてください。

    ```
    container.createDocument(
    ```

    次に、メソッド呼び出しの結果を **accepted** という変数に格納するようにこの行を更新してください。

    ```
    var accepted = container.createDocument(
    ```

1. **createDocument** メソッド呼び出しの後に新しいコード行を追加し、**accepted** 変数の値を確認して、true でない場合はメソッドから return するようにしてください。

    ```
    if (!accepted) return;
    ```

1. 最後に、**createDocument** メソッド呼び出しに 3 つ目のパラメーターを追加してください。このパラメーターは **error** と **newDoc** の 2 つを受け取る関数で、error が null かどうかを確認し、新しいドキュメントをストアド プロシージャの応答本文に設定します。

    ```
    (error, newDoc) => {
      if (error) throw new Error(error.message);
      context.getResponse().setBody(newDoc);
    }
    ```

1. 完了後、ストアド プロシージャのコードは次の内容になります。

    ```
    function createDoc(title) {
      var context = getContext();
      var container = context.getCollection();
      var doc = {
        name: title,
        categoryId: 'demo'
      }
      var accepted = container.createDocument(
        container.getSelfLink(),
        doc,
        (error, newDoc) => {
          if (error) throw new Error(error.message);
          context.getResponse().setBody(newDoc);
        }
      );
      if (!accepted) return;
    }
    ```

1. **Update** を選択して、ストアド プロシージャへの変更を保存してください。

1. **Execute** を選択し、次の入力パラメーターでストアド プロシージャを実行してください。

    | **Setting** | **Key** | **Value** |
    | :--- | :--- | :--- |
    | **Partition key value** | *String* | *demo* |
    | **Input parameters** | *String* | *second document* |

1. JSON 結果を確認してください。ストアド プロシージャが正常に実行されると、新しく作成したドキュメントが元の HTTP 要求に対する応答として返されます。

### タスク 3: ドキュメントをクエリする

このタスクでは、仕上げとして Data Explorer を使用し、このラボで作成した 2 つのドキュメントを返す SQL クエリを実行します。

1. **Data Explorer** で **cosmicworks** データベース ノードを展開し、**NOSQL API** ナビゲーション ツリー内の **products** コンテナー ノードを選択してください。

1. **New SQL Query** を選択してください。

1. エディター領域の内容を削除してください。

1. **categoryId** が **demo** と等しいすべてのドキュメントを返す、新しい SQL クエリを作成してください。

    ```
    SELECT * FROM docs WHERE docs.categoryId = 'demo'
    ```

1. **Execute Query** を選択してください。

1. このクエリ実行結果として、このラボで作成した 2 つのドキュメントが表示されることを確認してください。


### まとめ

このラボでは、Azure Cosmos DB for NoSQL でストアド プロシージャを作成する方法、エラー処理のベスト プラクティスを実装する方法、および SQL クエリでプロシージャを検証する方法を学習しました。ストアド プロシージャを使用すると、単一のトランザクション スコープ内で複数ドキュメントに対するサーバー側ロジックを実行でき、Azure Cosmos DB での CRUD 操作に役立ちます。

### レビュー

このラボでは、次を完了しました。

- タスク 1: ストアド プロシージャを作成した。
- タスク 2: ストアド プロシージャのベスト プラクティスを実装した。
- タスク 3: ドキュメントをクエリした。


### ラボは正常に完了しました