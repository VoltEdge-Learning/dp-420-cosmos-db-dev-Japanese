# Lab 11c - Azure Cosmos DB for NoSQL ソリューションを監視およびトラブルシューティングする

## ラボ シナリオ

Azure はデータの暗号化バックアップを自動的に取得します。これらのバックアップは **Periodic** と **Continuous** の 2 つのバックアップ モードで取得されます。

このラボでは、continuous backup mode を使用して `backup` と `restores` を行います。最初に Azure Cosmos DB アカウントを作成します。次に 2 つのコンテナーを作成し、それらにいくつかのドキュメントを追加します。その後、それらのコンテナー内のドキュメントをいくつか更新します。最後に、各削除操作の前の時点にアカウントを復元します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: Azure Cosmos DB for NoSQL アカウントを作成する。
- タスク 2: アカウントにデータベースと 2 つのコンテナーを追加する。
- タスク 3: コンテナーにアイテムを追加する。
- タスク 4: 既定のバックアップ モードを continuous に変更する。
- タスク 5: salesOrder ドキュメントの 1 つを削除する。
- タスク 6: salesOrder ドキュメントを削除する前の時点にデータベースを復元する。
- タスク 7: customer コンテナーを削除する。
- タスク 8: salesOrder ドキュメントを削除する前の時点にデータベースを復元する。
- タスク 9: 復元されたデータを確認する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab27.png)

## 演習 1: 復旧ポイントからデータベースまたはコンテナーを復元する

### タスク 1: Azure Cosmos DB for NoSQL アカウントを作成する

Azure Cosmos DB は複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングするときは、アカウントでサポートする API（例: **API for MongoDB** または **API for NoSQL**）を選択します。Azure Cosmos DB for NoSQL アカウントのプロビジョニング完了後、endpoint と key を取得できます。endpoint と key を使用して Azure Cosmos DB for NoSQL アカウントへプログラムから接続します。Azure SDK for .NET またはその他の SDK の接続文字列でも endpoint と key を使用します。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Azure services** カテゴリで **Create a resource** を選択し、次に **Azure Cosmos DB** を選択してください。

  > &#128161; 別の方法として、**&#8801;** メニューを展開し、**All Services** を選択して、**Databases** カテゴリの **Azure Cosmos DB** を選択し、**Create** を選択してください。

1. **Select API option** ペインで、**Azure Cosmos DB for NoSQL** セクション内の **Create** オプションを選択してください。

1. **Create Azure Cosmos DB Account** ペインで **Basics** タブを確認してください。

  | **Setting** | **Value** |
  | --- | --- |
  | **Subscription** | *Your existing Azure subscription* |
  | **Resource group** | *DP-420-DeploymentID* |
  | **Account Name** | *Enter a globally unique name* |
  | **Location** | *Choose any available region* |
  | **Capacity mode** | *Provisioned throughput* |
  | **Apply Free Tier Discount** | *Do Not Apply* |
  | **Global Distribution** TAB | Disable Multi-region Writes |

  >**Note** : DeploymentID は各環境に関連付けられた一意の ID です。値は environment details ページで確認できます。

  >&#128221;**Note**: Azure Cosmos DB アカウント作成時に、**Backup Policy** タブで **Continuous** モードを選択することで有効化できます。このラボでは、アカウント作成時にこの機能を有効にするか、以下のオプション手順でアカウント作成後に有効化するかを選択できます。**アカウント作成後に機能を有効化すると、5 分以上かかる場合があります。**

  > &#128221;**Note** : *[Multi-regions write accounts are not currently supported for continuous backups][/azure/cosmos-db/continuous-backup-restore-introduction]* です。

  > &#128221; **Note** : ラボ環境によっては新しいリソース グループの作成が制限されている場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. **Next: Networking** をクリックしてください。Networking ブレードは既定値のままにし、**Next: Backup Policy** をクリックしてください。

1. Backup Policy ブレードで Backup policy を **Continuous (7 days)** に設定してください。

1. **Review + Create** をクリックし、検証で Success が表示されたら **Create** をクリックしてください。

### タスク 2: アカウントにデータベースと 2 つのコンテナーを追加する

データベースと 2 つのコンテナーを作成します。

1. Azure portal で Azure Cosmos DB アカウントのページに移動してください。

1. **Data Explorer** ペインで **New Container** を展開し、**New Database** を選択してください。

1. **New Database** ポップアップで各設定に次の値を入力し、**OK** を選択してください。

  | **Setting** | **Value** |
  | --- | --- |
  | **Database id** | *`Sales`* |
  | **Provision throughput** | *Do not select* |

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで各設定に次の値を入力し、**OK** を選択してください。

  | **Setting** | **Value** |
  | --- | --- |
  | **Database id** | *Use existing* name: *Sales* |
  | **Container id** | *`customer`* |
  | **Partition key** | *`/id`* |
  | **Container throughput (400 - unlimited RU/s)** | *Manual* throughput: *400*|

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで各設定に次の値を入力し、**OK** を選択してください。

  | **Setting** | **Value** |
  | --- | --- |
  | **Database id** | *Use existing* name: *Sales* |
  | **Container id** | *`salesOrder`* |
  | **Partition key** | *`/id`* |
  | **Container throughput (400 - unlimited RU/s)** | *Manual* throughput: *400*|

### タスク 3: コンテナーにアイテムを追加する

これらのコンテナーにドキュメントを追加します。

1. Azure portal で Azure Cosmos DB アカウントのページに移動してください。

2. **Data Explorer** で、次の 2 つのドキュメントを **customer** コンテナーに追加してください。

3. **customer** コンテナーで **Items** を選択し、**New Items** をクリックして、**Save** をクリックしてください。

```
  {
    "id": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "title": "",
    "firstName": "Franklin",
    "lastName": "Ye",
    "emailAddress": "franklin9@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0139",
    "creationDate": "2014-02-05T00:00:00",
    "addresses": [
      {
        "addressLine1": "1796 Westbury Dr.",
        "addressLine2": "",
        "city": "Melton",
        "state": "VIC",
        "country": "AU",
        "zipCode": "3337"
      }
    ],
    "password": {
      "hash": "GQF7qjEgMl3LUppoPfDDnPtHp1tXmhQBw0GboOjB8bk=",
      "salt": "12C0F5A5"
    }
  }
```

4. **Items** を選択し、**New Items** をクリックして、**Save** をクリックしてください。

```
  {
    "id": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "title": "",
    "firstName": "Victor",
    "lastName": "Moreno",
    "emailAddress": "victor8@adventure-works.com",
    "phoneNumber": "1 (11) 500 555-0134",
    "creationDate": "2011-10-09T00:00:00",
    "addresses": [
      {
        "addressLine1": "Parkstr 42",
        "addressLine2": "",
        "city": "Hamburg",
        "state": "HH ",
        "country": "DE",
        "zipCode": "20354"
      }
    ],
    "password": {
      "hash": "n8l+wY/klP/hwTC3wSr8BLMA9tm3tGTyDsCgG/Q9EYI=",
      "salt": "AC22BC8C"
    }
  }
```
5. **Data Explorer** で、次の 3 つのドキュメントを **salesOrder** コンテナーに追加してください。

6. **Items** を選択し、**New Items** をクリックして、**Save** をクリックしてください。

```
  {
    "id": "000C23D8-B8BC-432E-9213-6473DFDA2BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2014-02-16T00:00:00",
    "shipDate": "2014-02-23T00:00:00",
    "details": [
      {
        "sku": "BK-R64Y-42",
        "name": "Road-550-W Yellow, 42",
        "price": 1120.49,
        "quantity": 1
      },
      {
        "sku": "HL-U509-B",
        "name": "Sport-100 Helmet, Blue",
        "price": 34.99,
        "quantity": 1
      }
    ]
  }
  ```

7. **Items** を選択し、**New Items** をクリックして、**Save** をクリックしてください。

  ```
  {
    "id": "001676F7-0B70-400B-9B7D-24BA37B97F70",
    "customerId": "001C8C0B-9B91-47A5-A198-8770E60CFF38",
    "orderDate": "2013-06-02T00:00:00",
    "shipDate": "2013-06-09T00:00:00",
    "details": [
      {
        "sku": "HL-U509-R",
        "name": "Sport-100 Helmet, Red",
        "price": 34.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      }
    ]
  }
  ```

8. **Items** を選択し、**New Items** をクリックして、**Save** をクリックしてください。

  ```
  {
    "id": "0019092E-BD25-48F5-8050-7051B2655BC5",
    "customerId": "0012D555-C7DE-4C4B-B4A4-2E8A6B8E1161",
    "orderDate": "2013-09-14T00:00:00",
    "shipDate": "2013-09-21T00:00:00",
    "details": [
      {
        "sku": "TI-T723",
        "name": "Touring Tire",
        "price": 28.99,
        "quantity": 1
      },
      {
        "sku": "BK-T79Y-50",
        "name": "Touring-1000 Yellow, 50",
        "price": 2384.07,
        "quantity": 1
      },
      {
        "sku": "TT-T092",
        "name": "Touring Tire Tube",
        "price": 4.99,
        "quantity": 1
      }
    ]
  }
```

### タスク 4: 既定のバックアップ モードを continuous に変更する（アカウント作成時に機能を有効化していない場合は任意）

*Azure Cosmos DB アカウント作成時にこの機能を有効化していない場合は、ここで有効化する必要があります。* バックアップ モードの変更は簡単で、1 つの設定を **On** にするだけです。ここで変更してください。

1. Azure portal で Azure Cosmos DB アカウントのページに移動してください。

1. **Settings** セクションで **Features** を選択してください。

1. **Continuous Backup** オプションを選択して機能を有効にしてください。このオプションを選択するとウィンドウが表示されるので、**Enable** ボタンを選択してください。この機能の有効化には 5 分以上かかる場合があります。

  > &#128221; *[Multi-regions write accounts are not currently supported for continuous backups][/azure/cosmos-db/continuous-backup-restore-introduction]* であることに注意してください。Azure Cosmos DB アカウント作成時に Multi-region writes を無効化していない場合は、ここで無効化する必要があります。無効化しないと continuous backup 機能の有効化は失敗します。multi-region writes は **Replicate data globally** の *Settings* セクションで無効化できます。

### タスク 5: salesOrder ドキュメントの 1 つを削除する

1. **Data Explorer** で、現在の日付と時刻を取得するために次のクエリを実行してください。このタイムスタンプをメモ帳にコピーしてください。このタイムスタンプは UTC である必要があります。

  ```
  SELECT GetCurrentDateTime ()
  ```

1. **Data Explorer** で **id** が `0019092E-BD25-48F5-8050-7051B2655BC5` の **salesOrder** ドキュメントを見つけてください。ドキュメントを削除し、存在しなくなったことを確認してください。

### タスク 6: salesOrder ドキュメントを削除する前の時点にデータベースを復元する

1. Azure portal で Azure Cosmos DB アカウントのページに移動してください。

1. *Settings* セクションで **Point in Time Restore** を選択してください。次の設定を使用して、**Submit** をクリックしてください。

  | **Setting** | **Value** |
  | --- | --- |
  | **Restore Point (UTC)** | 日付と時刻を適切に変換してください。時刻は AM/PM 形式である必要があります |
  | **Location** | *Selected an available location* |
  | **Select resources you would like to restore** | *Selected database/containers* |
  | **Restore Resource** | *salesOrder* |
  | **Resource Group** | _DP-420-DeploymentID_|
  | **Restore Target Account** | *choose a* ***new*** *Azure Cosmos DB account name* |

  > &#128221; Azure Cosmos DB の復元では、*existing* アカウントの上に復元することは ***決して*** なく、常に新しい Azure Cosmos DB アカウントを作成する必要があります。

  > &#128221; データベース全体やアカウント全体を復元することもできますが、実際の本番環境ではデータベースが非常に大きい場合があります。多くのシナリオでは、必要なコンテナーやデータベースだけを復元した方が速い可能性があります。

1. この復元には 15 分以上かかる場合があります。次のセクションに進み、この復元はバックグラウンドで実行したままにしてください。

### タスク 7: customer コンテナーを削除する

1. **Data Explorer** で、現在の日付と時刻を取得するために次のクエリを実行してください。このタイムスタンプをメモ帳にコピーしてください。

  ```
  SELECT GetCurrentDateTime ()
  ```

1. **customer** コンテナーを削除してください。

### タスク 8: salesOrder ドキュメントを削除する前の時点にデータベースを復元する

1. Azure portal で Azure Cosmos DB アカウントのページに移動してください。

1. *Settings* セクションで **Point in Time Restore** を選択してください。次の設定を使用してください。

  | **Setting** | **Value** |
  | --- | --- |
  | **Location** | *Selected an available location* |
  | **Restore Point (UTC)** | 日付と時刻を適切に変換してください。時刻は AM/PM 形式である必要があります |
  | **Select resources you would like to restore** | *Selected database/containers* |
  | **Restore Resource** | *`customer`* |
  | **Restore Target Account** | *choose a* ***new*** *Azure Cosmos DB account name* |

  > &#128221; Azure Cosmos DB の復元では、*existing* アカウントの上に復元することは ***決して*** なく、常に新しい Azure Cosmos DB アカウントを作成する必要があります。

  > &#128221; データベース全体やアカウント全体を復元することもできますが、実際の本番環境ではデータベースが非常に大きい場合があります。多くのシナリオでは、必要なコンテナーやデータベースだけを復元した方が速い可能性があります。

1. この復元には 15 分以上かかる場合があります。次のセクションに進み、この復元はバックグラウンドで実行したままにしてください。

### タスク 9: 復元されたデータを確認する

データベースのサイズやその他の要因によっては、復元に長い時間がかかる場合があります。Azure Cosmos DB アカウントの復元が完了したら、次を確認してください。

1. 最初の復元では、3 つ目のドキュメントが復旧していることを確認してください。

1. 2 回目の復元では、customer テーブルが復元されているはずです。

## クリーンアップ

1. アカウント復元により作成された 2 つの新しい Azure Cosmos DB アカウントを削除してください。

1. Sales データベースを削除し、必要であれば元の Azure Cosmos DB アカウントも削除してください。

### レビュー

このラボでは、次を完了しました。

- Azure Cosmos DB for NoSQL アカウントを作成した。
- アカウントにデータベースと 2 つのコンテナーを追加した。
- コンテナーにアイテムを追加した。
- 既定のバックアップ モードを continuous に変更した。
- salesOrder ドキュメントの 1 つを削除した。
- salesOrder ドキュメントを削除する前の時点にデータベースを復元した。
- customer コンテナーを削除した。
- salesOrder ドキュメントを削除する前の時点にデータベースを復元した。
- 復元されたデータを確認した。

### ラボは正常に完了しました