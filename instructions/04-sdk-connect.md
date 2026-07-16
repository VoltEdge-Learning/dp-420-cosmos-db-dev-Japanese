---
lab:
    title: 'SDK を使用して Azure Cosmos DB SQL API に接続する'
    module: 'モジュール 3 - SDK を使用して Azure Cosmos DB SQL API に接続する'
---

# SDK を使用して Azure Cosmos DB SQL API に接続する

.NET 用 Azure SDK は、多くの Azure サービスと対話するための一貫した開発者インターフェイスを提供するライブラリ スイートです。.NET 用 Azure SDK は .NET Standard 2.0 仕様に基づいて構築されており、.NET Framework（4.6.1 以上）、.NET Core（2.1 以上）、および .NET（5 以上）のアプリケーションで使用できることが保証されています。

このラボでは、.NET 用 Azure SDK を使用して Azure Cosmos DB SQL API アカウントに接続します。

## 開発環境を準備する

このラボを実施している環境に **DP-420** のラボ コード リポジトリをまだクローンしていない場合は、次の手順に従って実行してください。それ以外の場合は、以前にクローンしたフォルダーを **Visual Studio Code** で開きます。

1. **Visual Studio Code** を起動します。

    > &#128221; Visual Studio Code インターフェイスにまだ慣れていない場合は、[Visual Studio Code の Get Started ガイド][code.visualstudio.com/docs/getstarted] を確認してください。

1. コマンド パレットを開いて **Git: Clone** を実行し、``https://github.com/microsoftlearning/dp-420-cosmos-db-dev`` GitHub リポジトリを任意のローカル フォルダーにクローンします。

    > &#128161; **CTRL+SHIFT+P** キーボード ショートカットを使用してコマンド パレットを開くことができます。

1. リポジトリのクローンが完了したら、選択したローカル フォルダーを **Visual Studio Code** で開きます。

## Azure Cosmos DB SQL API アカウントを作成する

Azure Cosmos DB は、複数の API をサポートするクラウドベースの NoSQL データベース サービスです。初めて Azure Cosmos DB アカウントをプロビジョニングする際は、アカウントでサポートする API（たとえば **Mongo API** または **SQL API**）を選択します。Azure Cosmos DB SQL API アカウントのプロビジョニングが完了したら、endpoint と key を取得し、.NET 用 Azure SDK または任意のその他の SDK を使用して Azure Cosmos DB SQL API アカウントに接続できます。

1. 新しい Web ブラウザーのウィンドウまたはタブで、Azure portal (``portal.azure.com``) に移動します。

1. サブスクリプションに関連付けられている Microsoft 資格情報を使用してポータルにサインインします。

1. **+ Create a resource** を選択し、*Cosmos DB* を検索してから、次の設定で新しい **Azure Cosmos DB SQL API** アカウント リソースを作成します。残りのすべての設定は既定値のままにします。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Subscription** | *既存の Azure サブスクリプション* |
    | **Resource group** | *既存のリソース グループを選択するか、新しく作成する* |
    | **Account Name** | *グローバルに一意の名前を入力する* |
    | **Location** | *利用可能な任意のリージョンを選択する* |
    | **Capacity mode** | *プロビジョニングされたスループット* |
    | **Apply Free Tier Discount** | *適用しない* |
    | **Limit the total amount of throughput that can be provisioned on this account** | *オフ* |

    > &#128221; ラボ環境には、新しいリソース グループの作成を制限する制約がある場合があります。その場合は、既存の事前作成済みリソース グループを使用してください。

1. このタスクを続行する前に、デプロイ タスクが完了するまで待ちます。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Keys** ペインに移動します。

1. このペインには、SDK からアカウントに接続するために必要な接続の詳細と資格情報が含まれています。具体的には次のとおりです。

    1. **URI** フィールドの値を記録します。この演習の後半でこの **endpoint** 値を使用します。

    1. **PRIMARY KEY** フィールドの値を記録します。この演習の後半でこの **key** 値を使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## NuGet で Microsoft.Azure.Cosmos ライブラリを確認する

NuGet Web サイトには、.NET アプリケーションにインポート可能なパッケージの検索可能なインデックスがあります。**Microsoft.Azure.Cosmos** などのプレリリース パッケージをインポートするには、NuGet Web サイトを使用して、アプリケーションにパッケージをインポートするための適切なバージョンとコマンドを取得できます。

1. Web ブラウザーで、NuGet Web サイト (``nuget.org``) に移動します。

1. .NET 用パッケージ マネージャーである NuGet の説明とその機能を確認します。

1. NuGet.org で **Microsoft.Azure.Cosmos** ライブラリを検索します。

1. **.NET CLI** タブを選択して、このライブラリの最新バージョンを .NET プロジェクトにインポートするために必要なコマンドを確認します。

    > &#128161; このコマンドを記録する必要はありません。この演習の後半でライブラリの特定バージョンを使用します。

1. Web ブラウザーのウィンドウまたはタブを閉じます。

## Microsoft.Azure.Cosmos ライブラリを .NET プロジェクトにインポートする

.NET CLI には、事前構成されたパッケージ フィードからパッケージをインポートするための [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] コマンドが含まれています。.NET のインストールでは、既定のパッケージ フィードとして NuGet を使用します。

1. **Visual Studio Code** の **Explorer** ペインで、**04-sdk-connect** フォルダーを参照します。

1. **04-sdk-connect** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開きます。

    > &#128221; このコマンドでは、開始ディレクトリがすでに **04-sdk-connect** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加します。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. 統合ターミナルを閉じます。

## Microsoft.Azure.Cosmos ライブラリを使用する

.NET 用 Azure SDK の Azure Cosmos DB ライブラリをインポートしたら、すぐに [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] 名前空間内のクラスを使用して Azure Cosmos DB SQL API アカウントに接続できます。[CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] クラスは、Azure Cosmos DB SQL API アカウントへの初期接続を行うために使用される中核クラスです。

1. **Visual Studio Code** の **Explorer** ペインで、**04-sdk-connect** フォルダーを参照します。

1. 空の **script.cs** コード ファイルを開きます。

1. 組み込みの **System** および **System.Linq** 名前空間の using ブロックを追加します。

    ```
    using System;
    using System.Linq;
    ```

1. [Microsoft.Azure.Cosmos][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos] 名前空間の using ブロックを追加します。

    ```
    using Microsoft.Azure.Cosmos;
    ```

1. **endpoint** という名前の **string** 変数を追加し、その値を先ほど作成した Azure Cosmos DB アカウントの **endpoint** に設定します。
  
    ```
    string endpoint = "<cosmos-endpoint>";
    ```

    > &#128221; たとえば、endpoint が **https&shy;://dp420.documents.azure.com:443/** の場合、C# ステートメントは次のようになります: **string endpoint = "https&shy;://dp420.documents.azure.com:443/";**。

1. **key** という名前の **string** 変数を追加し、その値を先ほど作成した Azure Cosmos DB アカウントの **key** に設定します。

    ```
    string key = "<cosmos-key>";
    ```

    > &#128221; たとえば、key が **fDR2ci9QgkdkvERTQ==** の場合、C# ステートメントは次のようになります: **string key = "fDR2ci9QgkdkvERTQ==";**。

1. コンストラクターで **endpoint** 変数と **key** 変数を使用し、[CosmosClient][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient] 型の **client** という名前の新しい変数を追加します。
  
    ```
    CosmosClient client = new (endpoint, key);
    ```

1. **client** 変数の [ReadAccountAsync][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync] メソッドを呼び出した非同期結果を使用して、[AccountProperties][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties] 型の **account** という名前の新しい変数を追加します。

    ```
    AccountProperties account = await client.ReadAccountAsync();
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して、**Account Name** という見出しで AccountProperties クラスの [Id][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id] プロパティを出力します。

    ```
    Console.WriteLine($"Account Name:\t{account.Id}");
    ```

1. 組み込みの **Console.WriteLine** 静的メソッドを使用して AccountProperties クラスの [WritableRegions][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions] プロパティをクエリし、最初の結果の [Name][docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name] プロパティを **Primary Region** という見出しで出力します。

    ```
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. 完了すると、コード ファイルには次の内容が含まれているはずです。
  
    ```
    using System;
    using System.Linq;
    
    using Microsoft.Azure.Cosmos;

    string endpoint = "<cosmos-endpoint>";
    string key = "<cosmos-key>";

    CosmosClient client = new (endpoint, key);

    AccountProperties account = await client.ReadAccountAsync();

    Console.WriteLine($"Account Name:\t{account.Id}");
    Console.WriteLine($"Primary Region:\t{account.WritableRegions.FirstOrDefault()?.Name}");
    ```

1. **script.cs** コード ファイルを **Save** します。

## スクリプトをテストする

Azure Cosmos DB SQL API アカウントに接続する .NET コードが完成したので、スクリプトをテストできます。このスクリプトは、アカウント名と最初の書き込み可能リージョンの名前を出力します。アカウント作成時に場所を指定したため、このスクリプトの結果として同じ場所の値が出力されるはずです。

1. **Visual Studio Code** で **04-sdk-connect** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナル インスタンスを開きます。

1. [dotnet run][docs.microsoft.com/dotnet/core/tools/dotnet-run] コマンドを使用してプロジェクトをビルドし、実行します。

    ```
    dotnet run
    ```

1. これでスクリプトはアカウント名と最初の書き込み可能リージョンを出力します。たとえば、アカウント名を **dp420** にし、最初の書き込み可能リージョンが **West US 2** だった場合、スクリプトは次のように出力します。

    ```
    Account Name:   dp420
    Primary Region: West US 2
    ```

1. 統合ターミナルを閉じます。

1. **Visual Studio Code** を閉じます。

[code.visualstudio.com/docs/getstarted]: https://code.visualstudio.com/docs/getstarted/tips-and-tricks
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.id
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountproperties.writableregions
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.accountregion.name
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient
[docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync]: https://docs.microsoft.com/dotnet/api/microsoft.azure.cosmos.cosmosclient.readaccountasync
[docs.microsoft.com/dotnet/core/tools/dotnet-add-package]: https://docs.microsoft.com/dotnet/core/tools/dotnet-add-package
[docs.microsoft.com/dotnet/core/tools/dotnet-run]: https://docs.microsoft.com/dotnet/core/tools/dotnet-run
[nuget.org/packages/microsoft.azure.cosmos/3.22.1]: https://www.nuget.org/packages/Microsoft.Azure.Cosmos/3.22.1
