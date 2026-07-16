---
lab:
    title: 'Azure Functions を使用して Azure Cosmos DB SQL API データを処理する'
    module: 'モジュール 7 - Azure Cosmos DB SQL API を Azure サービスと統合する'
---

# Azure Functions を使用して Azure Cosmos DB SQL API データを処理する

Azure Functions の Azure Cosmos DB トリガーは、変更フィード プロセッサーを使用して実装されています。この仕組みにより、Azure Cosmos DB SQL API コンテナーでの作成および更新操作に応答する関数を作成できます。変更フィード プロセッサーを手動で実装したことがある場合、Azure Functions のセットアップはそれと似ています。

このラボでは、次を行います。

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
    | **Capacity mode** | *Serverless* |

    > &#128221; ラボ環境には、新しいリソース グループの作成を制限する制約がある場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. このタスクを続行する前に、デプロイ タスクが完了するまで待機してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Keys** ペインに移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録してください。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録してください。この演習の後半でこの **key** 値を使用します。

1. リソース メニューから **Data Explorer** を選択してください。

1. **Data Explorer** ペインで **New Container** を展開し、**New Database** を選択してください。

1. **New Database** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --: | :-- |
    | **Database id** | *cosmicworks* |

1. **Data Explorer** ペインに戻り、階層内の **cosmicworks** データベース ノードを確認してください。

1. **Data Explorer** ペインで **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --: | :-- |
    | **Database id** | *Use existing* &vert; *cosmicworks* |
    | **Container id** | *products* |
    | **Partition key** | */categoryId* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **products** コンテナー ノードを確認してください。

1. **Data Explorer** ペインで、再度 **New Container** を選択してください。

1. **New Container** ポップアップで、各設定に次の値を入力し、**OK** を選択してください。

    | **Setting** | **Value** |
    | --: | :-- |
    | **Database id** | *Use existing* &vert; *cosmicworks* |
    | **Container id** | *productslease* |
    | **Partition key** | */id* |

1. **Data Explorer** ペインに戻り、**cosmicworks** データベース ノードを展開して、階層内の **productslease** コンテナー ノードを確認してください。

1. Azure portal の **Home** に戻ってください。

## Azure Function アプリと Azure Cosmos DB トリガー関数を作成する

コードの作成を開始する前に、作成ウィザードを使用して Azure Functions リソースと依存リソース（Application Insights、Storage）を作成する必要があります。

1. **+ Create a resource** を選択し、*Functions* を検索してから、次の設定で新しい **Function App** アカウント リソースを作成してください。残りの設定はすべて既定値のままにしてください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Subscription** | *既存の Azure サブスクリプション* |
    | **Resource group** | *既存のリソース グループを選択するか新規作成する* |
    | **Name** | *グローバルで一意の名前を入力する* |
    | **Publish** | *Code* |
    | **Runtime stack** | *.NET* |
    | **Version** | *6* |
    | **Region** | *利用可能な任意のリージョンを選択する* |
    | **Storage account** | *新しいストレージ アカウントを作成する* |

    > &#128221; ラボ環境には、新しいリソース グループの作成を制限する制約がある場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. このタスクを続行する前に、デプロイ タスクが完了するまで待機してください。

1. 新しく作成した **Azure Functions** アカウント リソースに移動し、**Functions** ペインに移動してください。

1. **Functions** ペインで **+ Create** を選択してください。

1. **Create function** ポップアップで、次の設定で新しい関数を作成してください。残りの設定はすべて既定値のままにしてください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Development environment** | *Develop in portal* |
    | **Select a template** | *Azure Cosmos DB trigger* |
    | **New Function** | *ItemsListener* |
    | **Cosmos DB account connection** | *Select New* &vert; *Select Azure Cosmos DB Account* &vert; *Select the Azure Cosmos DB account you created earlier* |
    | **Database name** | *cosmicworks* |
    | **Collection name** | *products* |
    | **Collection name for leases** | *productslease* |
    | **Create lease collection if it does not exist** | *No* |

## .NET で関数コードを実装する

先ほど作成した関数は、ポータル内で編集する C# スクリプトです。ここではポータルを使用して、コンテナー内で挿入または更新された任意の項目の一意識別子を出力する短い関数を作成します。

1. **ItemsListener** &vert; **Function** ペインで、**Code + Test** ペインに移動してください。

1. **run.csx** スクリプトのエディターで、エディター領域の内容を削除してください。

1. エディター領域で **Microsoft.Azure.DocumentDB.Core** ライブラリを参照してください。

    ```
    #r "Microsoft.Azure.DocumentDB.Core"
    ```

1. **System**、**System.Collections.Generic**、および [Microsoft.Azure.Documents][docs.microsoft.com/dotnet/api/microsoft.azure.documents] 名前空間の using ブロックを追加してください。

    ```
    using System;
    using System.Collections.Generic;
    using Microsoft.Azure.Documents;
    ```

1. 2 つのパラメーターを持つ **Run** という名前の新しい static メソッドを作成してください。

    1. [Document][docs.microsoft.com/dotnet/api/microsoft.azure.documents.document] をジェネリック型に持つ **IReadOnlyList\<\>** 型の **input** という名前のパラメーター。

    1. **ILogger** 型の **log** という名前のパラメーター。

    ```
    public static void Run(IReadOnlyList<Document> input, ILogger log)
    {
    }
    ```

1. **Run** メソッド内で、**log** 変数の **LogInformation** メソッドを呼び出し、現在のバッチ内の項目数を計算する文字列を渡してください。

    ```
    log.LogInformation($"# Modified Items:\t{input?.Count ?? 0}"); 
    ```

1. 引き続き **Run** メソッド内で、**input** 変数を反復処理する foreach ループを作成してください。ここでは **Document** 型のインスタンスを表す変数として **item** を使用します。

    ```
    foreach(Document item in input)
    {
    }
    ```

1. **Run** メソッドの foreach ループ内で、**log** 変数の **LogInformation** メソッドを呼び出し、**item** 変数の [Id][docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id] プロパティを出力する文字列を渡してください。

    ```
    log.LogInformation($"Detected Operation:\t{item.Id}");
    ```

1. 完了したら、コード ファイルに次の内容が含まれていることを確認してください。
  
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

1. **Logs** セクションを展開して、現在の関数のストリーミング ログに接続してください。

    > &#128161; ストリーミング ログ サービスへの接続には数秒かかる場合があります。接続されると、ログ出力にメッセージが表示されます。

1. 現在の関数コードを **Save** してください。

1. C# コードのコンパイル結果を確認してください。ログ出力の最後に **Compilation succeeded** メッセージが表示されるはずです。

    > &#128221; ログ出力に警告メッセージが表示される場合があります。これらの警告はこのラボには影響しません。

1. ログ セクションを **Maximize** して、出力ウィンドウを利用可能な最大サイズまで拡大してください。

    > &#128221; 別のツールを使用して Azure Cosmos DB SQL API コンテナーに項目を生成します。項目を生成したら、このブラウザー ウィンドウに戻って出力を確認してください。ブラウザー ウィンドウを早く閉じないでください。

## Azure Cosmos DB SQL API アカウントにサンプル データをシードする

ここでは、**cosmicworks** データベースと **products** コンテナーを作成するコマンドライン ユーティリティを使用します。次にこのツールが項目セットを作成し、その変更をターミナル ウィンドウで実行中の変更フィード プロセッサーで確認します。

1. **Visual Studio Code** を起動してください。

    > &#128221; Visual Studio Code のインターフェイスにまだ慣れていない場合は、[Visual Studio Code の Get Started ガイド][code.visualstudio.com/docs/getstarted] を確認してください。

1. **Visual Studio Code** で **Terminal** メニューを開き、**New Terminal** を選択して新しいターミナル インスタンスを開いてください。

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

1. Azure Functions のログ セクションを展開したまま、現在開いているブラウザー ウィンドウまたはタブに戻ってください。

1. 関数のログ出力を確認してください。変更フィードを通じて送信された各変更に対して、ターミナルに **Detected Operation** メッセージが出力されます。操作は約 100 件ごとのグループにバッチ化されます。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.documents]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.document]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.document
[docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.documents.resource.id
