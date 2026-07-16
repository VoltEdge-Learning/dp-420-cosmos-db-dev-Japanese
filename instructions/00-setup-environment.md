---
lab:
    title: 'ラボ環境をセットアップする'
    module: 'Setup'
---

# ローカル ラボ環境をセットアップする

理想的には、これらのラボはホストされたラボ環境で実施してください。自身のコンピューターで実施する場合は、以下のソフトウェアをインストールすることで実施できます。独自環境を使用すると、予期しないダイアログや動作が発生する場合があります。ローカル構成には非常に多くのパターンがあるため、コース チームは独自環境で発生する問題をサポートできません。

## Windows インストール

> &#128221; 以下の手順は Windows 10 コンピューター向けです。Linux または MacOS も利用できます。選択した OS に合わせてラボ手順を調整する必要がある場合があります。

### Windows 10 (OS)

1. Windows 10（*バージョン 2004 以降*）をインストールしてください。

1. 利用可能なすべての更新プログラムを適用してください。

### Edge

1. [microsoft.com/edge] から Microsoft Edge の最新バージョンをインストールしてください。

### .NET 6 SDK

1. [dotnet.microsoft.com/download/dotnet/6.0] から SDK（ランタイムではなく SDK）をダウンロードしてインストールしてください。

### PowerShell 7

1. [github.com/powershell/powershell/releases] からダウンロードしてインストールしてください。

### Git

1. [git-scm.com/downloads] からダウンロードしてインストールしてください。

    - インストーラーでは既定のオプションを使用してください。

### Windows Terminal

1. [github.com/microsoft/terminal/releases] からダウンロードしてインストールしてください。

1. **PowerShell** を既定のターミナルとして構成してください。

### Visual Studio Code (and extensions)

1. [code.visualstudio.com/download] からダウンロードしてインストールしてください。

    - インストーラーでは既定のオプションを使用してください。

1. インストール後、Visual Studio Code を起動してください。

1. **Extensions** メニューで、次の Microsoft 拡張機能を検索してインストールしてください。

    - [C#][marketplace.visualstudio.com/ms-dotnettools.csharp]

### Azure Cosmos DB Emulator

1. [docs.microsoft.com/azure/cosmos-db/local-emulator] からダウンロードしてインストールしてください。
    - インストーラーでは既定のオプションを使用してください。

[code.visualstudio.com/download]: https://code.visualstudio.com/download
[docs.microsoft.com/azure/cosmos-db/local-emulator]: https://docs.microsoft.com/azure/cosmos-db/local-emulator#download-the-emulator
[dotnet.microsoft.com/download/dotnet/6.0]: https://dotnet.microsoft.com/download/dotnet/6.0
[git-scm.com/downloads]: https://git-scm.com/downloads
[github.com/microsoft/terminal/releases]: https://github.com/microsoft/terminal/releases/latest
[github.com/powershell/powershell/releases]: https://github.com/powershell/powershell/releases/latest
[marketplace.visualstudio.com/ms-dotnettools.csharp]: https://marketplace.visualstudio.com/items?itemName=ms-dotnettools.csharp
[microsoft.com/edge]: https://microsoft.com/edge