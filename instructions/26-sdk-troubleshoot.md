# Lab 11b - Azure Cosmos DB for NoSQL ソリューションを監視およびトラブルシューティングする

## ラボ シナリオ

Azure Cosmos DB には広範な応答コードのセットが用意されており、さまざまな操作種別で発生する可能性のある問題を容易にトラブルシューティングできます。重要なのは、Azure Cosmos DB 向けアプリを作成するときに、適切なエラー処理を実装することです。

このラボでは、2 つのドキュメントのいずれかを挿入または削除できるメニュー駆動型プログラムを作成します。このラボの主な目的は、よく使われる応答コードのいくつかをどのように使い、アプリのエラー処理コードでどのように活用するかを理解することです。複数の応答コードに対するエラー処理を実装しますが、実際に発生させる条件は 2 種類のみです。また、エラー処理自体は複雑なものではなく、応答コードに応じて画面にメッセージを表示するか、10 秒待ってもう一度操作を再試行するだけです。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Cosmos DB アカウントからキーと endpoint を取得する。
- タスク 3: .NET スクリプトに Microsoft.Azure.Cosmos ライブラリをインポートする。
- タスク 4: ドキュメントの挿入および削除を行うメニュー駆動型オプションを作成するスクリプトを実行する。
- タスク 5: ドキュメントを挿入および削除する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab26.png)

## 演習 1: Azure Cosmos DB for NoSQL SDK を使用してアプリケーションをトラブルシューティングする

### タスク 1: 開発環境を準備する

1. Visual Studio Code を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

2. 左側ペインの **Extension (1)** アイコンを選択してください。検索バーに **C# (2)** を入力し、表示された **extension (3)** を選択して、最後に拡張機能の **Install (4)** を選択してください。

    ![](media/C-hash-extension.png)

3. 画面左上の **file** オプションを選択し、メニューから **Open Folder** を選択して **C:\AllFiles** に移動してください。

4. **dp-420-cosmos-db-dev** フォルダーを選択し、**Select Folder** をクリックしてください。


### タスク 2: Cosmos DB アカウントからキーと endpoint を取得する

Azure Cosmos DB for NoSQL アカウントでは、endpoint と key を取得し、Azure SDK for .NET または任意の SDK を使用して Azure Cosmos DB for NoSQL アカウントに接続できます。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. まだサインインしていない場合は、サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. リソース グループ **DP-420-DeploymentID** を選択し、ラボ 1 で作成した **Cosmos DB** アカウントを選択してください。

1. **Keys** ペインに移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

1. **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。


### タスク 3: .NET スクリプトに Microsoft.Azure.Cosmos ライブラリをインポートする

.NET CLI には、事前構成済みのパッケージ フィードからパッケージをインポートするための [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] コマンドが含まれています。.NET インストールでは、既定のパッケージ フィードとして NuGet を使用します。

1. **Visual Studio Code** の **Explorer** ペインで **26-sdk-troubleshoot** フォルダーに移動してください。

1. **26-sdk-troubleshoot** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **26-sdk-troubleshoot** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加してください。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

### タスク 4: ドキュメントの挿入および削除を行うメニュー駆動型オプションを作成するスクリプトを実行する

アプリケーションを実行する前に、Azure Cosmos DB アカウントへ接続する必要があります。

1. **Visual Studio Code** の **Explorer** ペインで **26-sdk-troubleshoot** フォルダーに移動してください。

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

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドおよび実行してください。

    ```
    dotnet run
    ```
    > &#128221; これは非常に単純なプログラムです。次のように 5 つのオプションを持つメニューを表示します。定義済みドキュメントを挿入するオプションが 2 つ、定義済みドキュメントを削除するオプションが 2 つ、そしてプログラムを終了するオプションが 1 つあります。

    >```
    >1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'
    >4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'
    >5) Exit
    >Select an option:
    >```

### タスク 5: ドキュメントを挿入および削除する

1. 最初のドキュメントを挿入するため、**1** を入力して **ENTER** を押してください。プログラムは最初のドキュメントを挿入し、次のメッセージを返します。

    ```
    Insert Successful.
    Document for customer with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828' Inserted.
    Press [ENTER] to continue
    ```

1. もう一度、最初のドキュメントを挿入するために **1** を入力して **ENTER** を押してください。今回は、プログラムは例外でクラッシュします。エラー スタックを確認すると、プログラム失敗の理由を確認できます。エラー スタックから抽出されたメッセージを見ると、未処理例外 "Conflict (409)" が発生しています。

    ```
    Unhandled exception. Microsoft.Azure.Cosmos.CosmosException : Response status code does not indicate success: Conflict (409);
    ```

1. ドキュメントを挿入しているため、ドキュメント作成時に返される一般的な [create document status codes][/rest/api/cosmos-db/create-a-document#status-codes] の一覧を確認する必要があります。このコードの説明は、*新しいドキュメントに指定した ID が既存ドキュメントにより使用済みである* というものです。これは明らかで、少し前に同じドキュメントを作成するメニュー オプションを実行したためです。

1. さらにスタックを確認すると、この例外が 100 行目から呼び出され、さらにその呼び出し元が 64 行目であることが分かります。

    ```
    at Program.CreateDocument1(Container Customer) in C:\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 100
   at Program.CompleteTaskOnCosmosDB(String consoleinputcharacter, Container container) in C:\Git\dp-420-cosmos-db-dev\26-sdk-troubleshoot\Program.cs:line 64
    ```

1. 100 行目を確認すると、予想どおり *CreateItemAsync* 操作がエラーの原因です。

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
    ```

1. さらに 100 行目から 103 行目を確認すると、このコードにはエラー処理がないことが分かります。修正が必要です。

    ```C#
        ItemResponse<customerInfo> response = await Customer.CreateItemAsync<customerInfo>(customer, new PartitionKey(customerID));
        Console.WriteLine("Insert Successful.");
        Console.WriteLine("Document for customer with id = '" + customerID + "' Inserted.");
    ```

1. エラー処理コードで何を行うべきかを決める必要があります。[create document status codes][/rest/api/cosmos-db/create-a-document#status-codes] を確認すると、この操作に対して返され得るすべての状態コードに対するエラー処理を作成することもできます。このラボでは、この一覧のうち status code 403 から 409 のみを考慮します。それ以外の状態コードが返された場合は、システムのエラー メッセージを表示するだけにします。

    > &#128221; 403 例外に対するエラー処理も実装しますが、このラボでは 403 例外自体は発生させません。

1. **CompleteTaskOnCosmosDB** という名前の関数に対するエラー処理を追加してください。**Main** 関数の **while** ループを 45 行目付近で見つけ、**CompleteTaskOnCosmosDB** の呼び出しをエラー処理コードで囲んでください。47 行目付近の **CompleteTaskOnCosmosDB** ステートメントを以下のコードに置き換えます。この新しいコードでまず注目すべき点は、**catch** で **CosmosException** クラス型の例外を捕捉していることです。このクラスには **StatusCode** プロパティがあり、Azure Cosmos DB サービスから返される要求完了状態コードを返します。**StatusCode** プロパティは **System.Net.HttpStatusCode** 型であり、この値を使用して .NET の [HTTP Status Code][dotnet/api/system.net.httpstatuscode] のフィールド名と比較できます。

    ```C#
        try
        {
            await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
        }
        catch (CosmosException e)
        {
                    switch (e.StatusCode.ToString())
                    {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists.");
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                    }

        }

    ```

1. ファイルを保存してください。先ほどクラッシュしたため、メニュー プログラムをもう一度実行する必要があります。次のコマンドを実行してください。

    ```
    dotnet run
    ```

1. 再度、最初のドキュメントを挿入するために **1** を入力して **ENTER** を押してください。今回はクラッシュせず、何が起きたかを示す、より分かりやすいメッセージが表示されます。

    ```
    Insert Failed.
    Response Code (409).
    Can not insert a duplicate partition key, customer with the same ID already exists.
    ```

1. このコードにより *403* と *409* 例外へのエラー処理が追加されました。次に、一般的な通信系例外に対するコードも追加します。一般的な通信系例外には *429*、*503*、*408* の 3 つがあり、それぞれ too many request、service unavailable、request time out を意味します。66 行目付近には **default** ステートメントがあるはずなので、前の **break;** ステートメントの直後、**default** ステートメントの直前に次のコードを追加してください。このコードは、これらの通信系例外が発生したかどうかを確認し、発生した場合は 10 秒待機した後、ドキュメント挿入をもう一度試行します。次のコードを追加してください。

    ```C#
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
    ```

    > &#128221; 429、503、408 例外に遭遇した場合の処理も実装しますが、このラボではその種類の例外は発生させません。

1. **Main** 関数は次のような形になるはずです。

    ```C#
        public static async Task Main(string[] args)
        {

            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");

            string consoleinputcharacter;

            while((consoleinputcharacter = Console.ReadLine()) != "5")
            {
                 try
                 {
                     await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                 }
                 catch (CosmosException e)
                 {
                     switch (e.StatusCode.ToString())
                     {
                        case ("Conflict"):
                            Console.WriteLine("Insert Failed. Response Code (409).");
                            Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists.");
                            break;
                        case ("Forbidden"):
                            Console.WriteLine("Response Code (403).");
                            Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                            Console.WriteLine("Firewall blocking requests.");
                            Console.WriteLine("Partition key exceeding storage.");
                            Console.WriteLine("Non-data operations are not allowed.");
                            break;
                        case ("TooManyRequests"):
                        case ("ServiceUnavailable"):
                        case ("RequestTimeout"):
                            // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                            await Task.Delay(10000); // Wait 10 seconds
                            try
                            {
                                Console.WriteLine("Try one more time...");
                                await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                            }
                            catch (CosmosException e2)
                            {
                                Console.WriteLine("Insert Failed. " + e2.Message);
                                Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                break;
                            }
                            break;
                        default:
                            Console.WriteLine(e.Message);
                            break;
                     }
                }


                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

1. 上記の変更により **CreateDocument2** 関数も修正されることに注意してください。

1. 最後に、**DeleteDocument1** および **DeleteDocument2** 関数についても、**CreateDocument1** 関数と同様に適切なエラー処理コードへ置き換える必要があります。これらの関数は **CreateItemAsync** の代わりに **DeleteItemAsync** を使う点以外に、[deletes status codes][/rest/api/cosmos-db/delete-a-document] が挿入時の状態コードと異なります。削除では、ドキュメントが見つからないことを示す **404** 状態コードのみを考慮します。**CompleteTaskOnCosmosDB** 関数呼び出しのエラー処理に追加の case を加えてください。**Main** 関数では、次のコードを **default** case の前に追加する必要があります。

    ```C#
                    case ("NotFound"):
                        Console.WriteLine("Delete Failed. Response Code (404).");
                        Console.WriteLine("Can not delete customer, customer not found.");
                        break;
    ```

1. すべての関数の修正が完了したら、すべてのメニュー オプションを数回テストし、例外発生時にアプリがクラッシュせずメッセージを返すことを確認してください。もしアプリがクラッシュする場合はエラーを修正し、次のコマンドを再実行してください。

    ```
    dotnet run
    ```


1. 先に答えを見ないでください。完了後、`Main` コードはおおよそ次のようになります。

    ```C#
        public static async Task Main(string[] args)
        {
            CosmosClient client = new CosmosClient(connectionString,new CosmosClientOptions() { AllowBulkExecution = true, MaxRetryAttemptsOnRateLimitedRequests = 50, MaxRetryWaitTimeOnRateLimitedRequests = new TimeSpan(0,1,30)});

            Console.WriteLine("Creating Azure Cosmos DB Databases and containers");

            Database CustomersDB = await client.CreateDatabaseIfNotExistsAsync("CustomersDB");
            Container CustomersDB_Customer_container = await CustomersDB.CreateContainerIfNotExistsAsync(id: "Customer", partitionKeyPath: "/id", throughput: 400);

            Console.Clear();
            Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
            Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
            Console.WriteLine("5) Exit");
            Console.Write("\r\nSelect an option: ");

            string consoleinputcharacter;

            while((consoleinputcharacter = Console.ReadLine()) != "5")
            {
                    try
                    {
                        await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                    }
                    catch (CosmosException e)
                    {
                        switch (e.StatusCode.ToString())
                        {
                            case ("Conflict"):
                                Console.WriteLine("Insert Failed. Response Code (409).");
                                Console.WriteLine("Can not insert a duplicate partition key, customer with the same ID already exists.");
                                break;
                            case ("Forbidden"):
                                Console.WriteLine("Response Code (403).");
                                Console.WriteLine("The request was forbidden to complete. Some possible reasons for this exception are:");
                                Console.WriteLine("Firewall blocking requests.");
                                Console.WriteLine("Partition key exceeding storage.");
                                Console.WriteLine("Non-data operations are not allowed.");
                                break;
                            case ("TooManyRequests"):
                            case ("ServiceUnavailable"):
                            case ("RequestTimeout"):
                                // Check if the issues are related to connectivity and if so, wait 10 seconds to retry.
                                await Task.Delay(10000); // Wait 10 seconds
                                try
                                {
                                    Console.WriteLine("Try one more time...");
                                    await CompleteTaskOnCosmosDB(consoleinputcharacter, CustomersDB_Customer_container);
                                }
                                catch (CosmosException e2)
                                {
                                    Console.WriteLine("Insert Failed. " + e2.Message);
                                    Console.WriteLine("Can not insert a duplicate partition key, Connectivity issues encountered.");
                                    break;
                                }
                                break;
                            case ("NotFound"):
                                Console.WriteLine("Delete Failed. Response Code (404).");
                                Console.WriteLine("Can not delete customer, customer not found.");
                                break;
                            default:
                                Console.WriteLine(e.Message);
                                break;
                        }

                    }

                Console.WriteLine("Choose an action:");
                Console.WriteLine("1) Add Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("2) Add Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("3) Delete Document 1 with id = '0C297972-BE1B-4A34-8AE1-F39E6AA3D828'");
                Console.WriteLine("4) Delete Document 2 with id = 'AAFF2225-A5DD-4318-A6EC-B056F96B94B7'");
                Console.WriteLine("5) Exit");
                Console.Write("\r\nSelect an option: ");
            }
        }
    ```

## 結論

経験の浅い開発者であっても、すべてのコードに適切なエラー処理を追加しなければならないことは理解しています。このコードにおけるエラー処理はシンプルですが、Azure Cosmos DB の例外コンポーネントの基本を理解し、堅牢なエラー処理ソリューションをコード内で作成するための土台になるはずです。


[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
[/rest/api/cosmos-db/create-a-document#status-codes]:https://docs.microsoft.com/rest/api/cosmos-db/create-a-document#status-codes
[dotnet/api/system.net.httpstatuscode]:https://docs.microsoft.com/dotnet/api/system.net.httpstatuscode?view=net-6.0
[/rest/api/cosmos-db/delete-a-document]:https://docs.microsoft.com/rest/api/cosmos-db/delete-a-document#status-codes

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- Cosmos DB アカウントからキーと endpoint を取得した。
- .NET スクリプトに Microsoft.Azure.Cosmos ライブラリをインポートした。
- ドキュメントの挿入および削除を行うメニュー駆動型オプションを作成するスクリプトを実行した。
- ドキュメントを挿入および削除した。

### ラボは正常に完了しました