# Lab 11d - Azure Cosmos DB for NoSQL ソリューションを監視およびトラブルシューティングする

## ラボ シナリオ

Azure Cosmos DB アカウントの接続コードをアプリケーションに追加するのは、アカウントの URI とキーを指定するだけなので簡単です。このセキュリティ情報は、アプリケーション コード内にハードコードされることがあります。しかし、アプリケーションを Azure App Service にデプロイする場合は、暗号化された接続情報を Azure Key Vault に保存できます。

このラボでは、Azure Cosmos DB アカウントの接続文字列を暗号化して Azure Key Vault に保存します。次に、その資格情報を Azure Key Vault から取得する Azure App Service Web アプリを作成します。アプリケーションはこれらの資格情報を使用して Azure Cosmos DB アカウントに接続します。その後、Azure Cosmos DB アカウントのコンテナーにいくつかのドキュメントを作成し、その状態を Web ページへ返します。

## ラボの目的

このラボでは、次のタスクを完了します。
- タスク 1: 開発環境を準備する。
- タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する。
- タスク 3: Azure Key Vault を作成し、Azure Cosmos DB アカウントの資格情報をシークレットとして保存する。
- タスク 4: Azure App Service Web アプリを作成する。
- タスク 5: .NET スクリプトに不足している複数のライブラリをインポートする。
- タスク 6: Web アプリに Secret Identifier を追加する。
- タスク 7: （任意）Azure App Services 拡張機能をインストールする。
- タスク 8: アプリケーションを Azure App Services にデプロイする。
- タスク 9: アプリがマネージド ID を使用できるようにする。
- タスク 10: Web アプリケーションに Key Vault シークレットへのアクセス ポリシーを付与する。

## 推定所要時間: 30 分

## アーキテクチャ図

![image](architecturedia/lab28.png)

## 演習 1: Azure Cosmos DB for NoSQL アカウントのキーを Azure Key Vault に保存する

### タスク 1: 開発環境を準備する

1. Visual Studio Code を起動してください（プログラム アイコンはデスクトップにピン留めされています）。

2. 左側ペインの **Extension (1)** アイコンを選択してください。検索バーに **C# (2)** を入力し、表示された **extension (3)** を選択して、最後に **Install (4)** を選択してください。

    ![](media/C-hash-extension.png)

3. 画面左上の **file** オプションを選択し、メニューから **Open Folder** を選択して **C:\AllFiles** に移動してください。

4. **dp-420-cosmos-db-dev** フォルダーを選択し、**Select Folder** をクリックしてください。

### タスク 2: Azure Cosmos DB for NoSQL アカウントを作成する

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

    >**Note** : DeploymentID は各環境に関連付けられた一意の ID です。値は environment details ページで確認できます。

1. **Review + Create** をクリックし、検証で Success が表示されたら **Create** をクリックしてください。

1. このタスクを続行する前に、デプロイが完了するまで待機してください。

1. 新しく作成した **Azure Cosmos DB** アカウント リソースに移動し、**Keys** ペインに移動してください。

1. このペインには、SDK からアカウントに接続するために必要な接続情報と資格情報が含まれています。具体的には次のとおりです。

    1. **PRIMARY CONNECTION STRING** フィールドの値を記録してください。この演習の後半でこの **connection string** 値を使用します。

### タスク 3: Azure Key Vault を作成し、Azure Cosmos DB アカウントの資格情報をシークレットとして保存する

Web アプリを作成する前に、Azure Cosmos DB アカウントの接続文字列を *Azure Key Vault* の暗号化 *secret* にコピーして保護します。ここで実施してください。

1. Azure portal で **Key vaults** ページに移動してください。

1. ***+ Create*** ボタンを選択して vault を追加し、次の設定で vault を構成してください。*その他の設定はすべて既定値のままにして*、vault を作成してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Subscription** | *Your existing Azure subscription* |
    | **Resource group** | *Select an existing or create a new resource group* |
    | **Key vault name** | *Enter a globally unique name* |
    | **Region** | *Choose any available region* |

1. **Review + Create** をクリックし、検証で Success が表示されたら **Create** をクリックしてください。

1. vault が作成されたら、その vault に移動してください。

1. *Settings* セクションで **Secrets** を選択してください。

1. **+ Generate/Import** を選択して資格情報の接続文字列を暗号化し、*secret* に次の設定を入力してください。*その他の設定はすべて既定値のままにして*、secret を作成してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Upload options** | *Manual* |
    | **Name** | *The name you will label your secret with* |
    | **Value** | *This field is the most important field to fill out. This value is the PRIMARY CONNECTION STRING you previously copied from the key section of your Azure Cosmos DB account. This value will be convert into a secret.* |
    | **Enabled** | *Yes* |

1. **Create** をクリックしてください。

1. Secrets の下に、新しい secret が一覧表示されているはずです。Web アプリのコードに追加する *secret identifier* を取得する必要があります。作成した **secret** を選択してください。

1. Azure Key Vault では secret の複数バージョンを作成できますが、このラボでは 1 バージョンのみ必要です。**Current version** を選択してください。

1. **Secret Identifier** フィールドの値を記録してください。この値をアプリケーション コードで使用して、Key Vault から secret を取得します。この値は URL であることに注意してください。この secret を正しく機能させるために、もう 1 つ手順が必要ですが、それは少し後で行います。

### タスク 4: Azure App Service Web アプリを作成する

Azure Cosmos DB アカウントに接続し、いくつかのコンテナーとドキュメントを作成する Web アプリを作成します。このアプリでは Azure Cosmos DB の *credentials* をハードコードせず、代わりに key vault の **Secret Identifier** をハードコードします。この identifier が、Azure レイヤーで Web アプリに適切な権限が割り当てられていなければ無意味であることを確認します。ここからコーディングを始めます。

1. **Visual Studio Code** を開いてください。File->Open folder を選択し、**28-key-vault** フォルダーの中まで移動して **28-key-vault** フォルダーを開いてください。

    > &#128221; **Explorer** ツリーには **28-key-vault** フォルダーとそのファイルおよびサブフォルダーのみが表示されている必要があります。先ほどクローンした GitHub リポジトリ全体が見えている場合、Web アプリは正しく動作しないため、**Explorer** ツリーには **28-key-vault** フォルダーとそのファイルおよびサブフォルダーのみが表示されていることを確認してください。

1. **28-key-vault** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **28-key-vault** フォルダーに設定された状態でターミナルが開きます。

1. MVC Web アプリのシェルを作成してください。生成されたファイルの一部は後で置き換えます。次のコマンドを実行して Web アプリを作成してください。

    ```
    dotnet new mvc
    ```

1. このコマンドで Web アプリのシェルが作成され、複数のファイルとディレクトリが追加されます。必要なコードを含むファイルはすでに用意されています。**.\Controllers\HomeController.cs** と **.\Views\Home\Index.cshtml** を、それぞれ **.\KeyvaultFiles** ディレクトリ内の対応ファイルで置き換えてください。

1. ファイルを置き換えたら、**.\KeyvaultFiles** ディレクトリを ***DELETE*** してください。

### タスク 5: .NET スクリプトに不足している複数のライブラリをインポートする

.NET CLI には、事前構成済みのパッケージ フィードからパッケージをインポートするための [add package][docs.microsoft.com/dotnet/core/tools/dotnet-add-package] コマンドが含まれています。.NET インストールでは、既定のパッケージ フィードとして NuGet を使用します。

1. まだ実施していない場合は、**Visual Studio Code** の **Explorer** ペインで **28-key-vault** フォルダーに移動してください。

1. まだ実施していない場合は、**28-key-vault** フォルダーのコンテキスト メニューを開き、**Open in Integrated Terminal** を選択して新しいターミナルを開いてください。

    > &#128221; このコマンドを実行すると、開始ディレクトリが **28-key-vault** フォルダーに設定された状態でターミナルが開きます。

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Cosmos][nuget.org/packages/microsoft.azure.cosmos/3.22.1] パッケージを追加してください。

    ```
    dotnet add package Microsoft.Azure.Cosmos --version 3.22.1
    ```

1. 次のコマンドを使用して、NuGet から [Newtonsoft.Json][nuget.org/packages/Newtonsoft.Json/13.0.1] パッケージを追加してください。

    ```
    dotnet add package Newtonsoft.Json --version 13.0.1
    ```

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.KeyVault][nuget.org/packages/Microsoft.Azure.KeyVault] パッケージを追加してください。

    ```
    dotnet add package Microsoft.Azure.KeyVault
    ```

1. 次のコマンドを使用して、NuGet から [Microsoft.Azure.Services.AppAuthentication][nuget.org/packages/Microsoft.Azure.Services.AppAuthentication] パッケージを追加してください。

    ```
    dotnet add package Microsoft.Azure.Services.AppAuthentication
    ```

### タスク 6: Web アプリに Secret Identifier を追加する

1. Visual Studio で `./Controllers/HomeControler.cs` ファイルを開いてください。

1. **GetKeyVaultSecret** ユーザー定義関数は、Azure Cosmos DB アカウントの secret を取得します。関数は *line 98* から始まり、次のスクリプトのようになっているはずです。

```
        private static async Task<Tuple<bool,string>>  GetKeyVaultSecret()
        {
            AzureServiceTokenProvider azureServiceTokenProvider = new AzureServiceTokenProvider("RunAs=App;");

            try
            {
                var KVClient = new KeyVaultClient(
                    new KeyVaultClient.AuthenticationCallback(azureServiceTokenProvider.KeyVaultTokenCallback));

                var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
                    .ConfigureAwait(false);

                return new Tuple<bool,string>(true, KeyVaultSecret.Value.ToString());

            }
            catch (Exception exp)
            {
                return new Tuple<bool,string>(false, exp.Message);
            }

        }
```

3. この関数が行う重要な呼び出しを確認してください。

    - *line 100* では、現在の Web アプリのトークンを定義しています。このトークンは Azure Key Vault に渡され、どのアプリが vault へアクセスしようとしているかを識別します。
    - *line 104-105* では、Azure Key Vault に接続する *Key Vault Client* を準備しています。Web アプリのトークンをパラメーターとして渡している点に注意してください。
    - *lines 107-108* では、**Secret Identifier** の URL を Key Vault Client に渡しており、その key vault に格納された secret を返します。

1. Web アプリをデプロイする前に、**Secret Identifier** URL を設定する必要があります。*line 107* で、文字列 ***<Key Vault Secret Identifier>*** を *secret* セクションで記録した **Secret Identifier** URL に置き換え、ファイルを保存してください。

```
        var KeyVaultSecret = await KVClient.GetSecretAsync("<Key Vault Secret Identifier>")
```

### タスク 7: （任意）Azure App Services 拡張機能をインストールする

Visual Studio でコマンド パレット（**CTRL+SHIFT+P**）を開き、Azure App Resource コマンドを検索しても何も表示されない場合は、この拡張機能をインストールする必要があります。

1. Visual Studio Code 左側メニューの **Extensions** を選択してください。

1. 検索バーで Azure App Service を検索し、選択してください。

1. Install ボタンを選択してインストールしてください。

1. **Extensions** タブを閉じ、コードに戻ってください。

### タスク 8: アプリケーションを Azure App Services にデプロイする

残りのコードは単純で、接続文字列を取得し、Azure Cosmos DB に接続し、いくつかのドキュメントを追加するだけです。アプリケーションは、問題があればそのフィードバックも提供するはずです。アプリケーションのデプロイ後に追加変更は不要なはずです。ここから開始してください。

> &#128221; 以下の手順の多くは、Visual Studio 画面上中央のコマンド パレットで実行します。

1. Visual Studio Code で、左側パネルから **Azure(shift+alt+a)** をクリックし、***App Service*** を右クリックして ***Create New Web App ... (Advanced)*** を選択してください。

    ![Screenshot of App Service.](media/DP-420-M11-lab4-appservice.png)

1. ***Sign-in to Azure...*** を選択してください。このオプションを選択すると Web ブラウザー ウィンドウが開くので、サインイン手順を完了し、完了後にブラウザーを閉じてください。

1. （任意）サブスクリプションの選択を求められた場合は、サブスクリプションを選択してください。

1. Web アプリに対してグローバルに一意な名前を入力してください。

1. 既存の Resource Group を選択するか、必要に応じて新規作成してください。

1. **.NET 6 (LTS)** を選択してください。

1. **Windows** を選択してください。

1. 使用可能な Location を選択してください。

1. **+ Create a new App Service Plan** を選択してください。

1. App Service Plan の既定名（通常は Web アプリ名と同じ）をそのまま使用するか、新しい名前を指定してください。

1. **Free (F1) Try out Azure at no cost** を選択してください。

1. Application Insights については **Skip for now** を選択してください。

1. デプロイが開始され、右下にステータス バーが表示されるはずです。

1. プロンプトが表示されたら **Deploy** を選択してください。

1. **Browse** を選択してください。**28-key-vault** フォルダー内にいるはずなので、そのフォルダーを選択してください。

1. **Required configuration to deploy is missing from "28-key-vault"** というメッセージのポップアップが表示されるはずです。Add Config ボタンを選択してください。この操作により不足している `.vscode` フォルダーが作成されます。

    > &#128221; 非常に重要です。アプリの初回デプロイ時にこのポップアップが表示されない場合、Azure App Services へのアップロード時にファイルが不足します。デプロイ自体は成功しますが、Web サイトには常に *You do not have permission to view this directory or page.* というメッセージが表示されます。最も可能性が高い原因は、Visual Studio Code を **28-key-vault** フォルダー単体ではなく、クローンした GitHub リポジトリ全体で開いていることです。

1. その workspace に常にデプロイするか確認されたら **Yes** を選択してください。

1. プロンプトが表示されたら Browse Website を選択してください。あるいは、ブラウザーを開いて **`https://<yourwebappname>.azurewebsites.net`** に移動してください。どちらの場合も、ここでは問題があります。本来であれば、Web ページ上にユーザー定義メッセージが表示されるはずです。表示されるべきメッセージは、拡張エラー メッセージ付きの **Key Vault was not accessible** です。これを修正します。

### タスク 9: アプリがマネージド ID を使用できるようにする

最初に修正すべき問題は、アプリがマネージド ID を使用できるようにすることです。マネージド ID を使用すると、アプリは Azure Key Vault などの Azure Services を利用できるようになります。

1. ブラウザーを開き、Azure portal にログインしてください。

1. **App Services** ページを開いてください。Web アプリ名が一覧表示されているはずなので、それを選択してください。

1. *Settings* セクションで **Identity** を選択してください。

1. Status を **On** に設定し、**Save** を選択してください。*Assigned Managed Identity* を有効にする確認が表示されたら **Yes** を選択してください。

1. Web アプリを再度試してください。ブラウザーで **`https://<yourwebappname>.azurewebsites.net`** に移動してください。

1. まだ 1 つ問題があります。最初のメッセージはプログラムが送信しているユーザー定義メッセージですが、2 つ目はシステム生成メッセージです。この 2 つ目のメッセージが意味するのは、Key vault へ接続するアクセスは許可されたものの、vault 内の secret を参照する権限はまだ付与されていないということです。この問題を修正する最後の設定を行います。

### タスク 10: Web アプリケーションに Key Vault シークレットへのアクセス ポリシーを付与する

このラボの元々の目的は、Azure Cosmos DB Accounts の資格情報をアプリケーションにハードコードしないようにすることでした。しかし、**Secret Identifier** URL はハードコードしており、これは誰でも見ることができます。では、どのように資格情報を保護するのでしょうか。良い知らせは、Secret Identifier 単体では無意味だということです。**Secret Identifier** は Azure Key vault の入口まで案内するだけで、vault が「誰を入れて誰を入れないか」を決めます。つまり、アプリケーションがその vault 内のシークレットを参照できるように、Key Vault のアクセス ポリシーを作成する必要があります。その方法を確認します。

1. （任意）ポリシーを作成する前に、Azure Cosmos DB データベースの現在の内容を確認してください。Azure portal で Azure Cosmos DB アカウントに移動し、**GlobalCustomers** Database が存在するか確認してください。存在しない場合は、Web アプリの正常実行により作成されます。存在する場合は、データベース内のアイテム数を確認してください。Web アプリの正常実行によりさらにアイテムが追加されます。

1. Azure portal で、先ほど作成した Key vault に移動してください。

1. *Settings* セクションで **Access policies** を選択してください。

1. **+ Create** を選択してください。

1. *Access policy* に次の設定を入力してください。*その他の設定はすべて既定値のままにして*、ポリシーを追加してください。

    | **Setting** | **Value** |
    | --- | --- |
    | **Key permissions** | *Get* |
    | **Secret permissions** | *Get* |

    > &#128221; Authorized application は選択しないでください。

1. **principal** ブレードで **Next** をクリックし、**Search and select your application name** を実行して、**Next** をクリックしてください。

1. **Application (optional)** ブレードは既定値のままにし、**Next** をクリックしてください。

1. 新しいポリシーを **Create** してください。

1. Web アプリを再度試してください。ブラウザーで **`https://<yourwebappname>.azurewebsites.net`** に移動してください。

1. 成功です。Web ページには、customer コンテナーに新しいアイテムを挿入したことが表示されるはずです。実際の Secret が表示されていることも確認できます。

    > &#128221; 本番環境では、secret を表示しては **いけません**。ここでは説明のためだけに表示しています。

1. Azure Cosmos DB アカウントに移動し、新しい **GlobalCustomers** データベースがデータ付きで作成されているか、または既に存在していた場合はアイテム数が増えているかを確認してください。

これで、Azure Key Vault を使用して Azure Cosmos DB アカウントのキーを保護できました。

## クリーンアップ

1. このラボで作成した Azure Cosmos DB アカウントを削除してください。

1. 元の Azure Cosmos DB アカウントを削除してください。

### レビュー

このラボでは、次を完了しました。

- 開発環境を準備した。
- Azure Cosmos DB for NoSQL アカウントを作成した。
- Azure Key Vault を作成し、Azure Cosmos DB アカウントの資格情報をシークレットとして保存した。
- Azure App Service Web アプリを作成した。
- .NET スクリプトに不足していた複数のライブラリをインポートした。
- Web アプリに Secret Identifier を追加した。
- （任意）Azure App Services 拡張機能をインストールした。
- アプリケーションを Azure App Services にデプロイした。
- アプリがマネージド ID を使用できるようにした。
- Web アプリケーションに Key Vault シークレットへのアクセス ポリシーを付与した。

### ラボは正常に完了しました