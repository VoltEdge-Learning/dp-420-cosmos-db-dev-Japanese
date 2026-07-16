---
lab:
    title: 'リソース プロバイダーを有効化する'
    module: 'Setup'
---

# Azure リソース プロバイダーを有効化する

Azure サブスクリプションに登録しておく必要があるリソース プロバイダーがあります。次の手順に従って、登録されていることを確認してください。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Home** ページで **Subscriptions** を選択してください。

    > &#128161; 別の方法として、**&#8801;** メニューを展開し、**All Services** を選択して、**All** カテゴリで **Subscriptions** を選択してください。

1. Azure サブスクリプションを選択してください。

    > &#128221; サブスクリプションが複数ある場合は、Azure Pass を引き換えて作成したサブスクリプションを選択してください。

1. サブスクリプションのブレードで、**Settings** セクションの **Resource providers** を選択してください。

1. リソース プロバイダーの一覧で、次のプロバイダーが登録されていることを確認してください。
    - [Microsoft.DocumentDB][docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]
    - [Microsoft.Insights][docs.microsoft.com/azure/templates/microsoft.insights/components]
    - [Microsoft.KeyVault][docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]
    - [Microsoft.Search][docs.microsoft.com/azure/templates/microsoft.search/searchservices]
    - [Microsoft.Web][docs.microsoft.com/azure/templates/microsoft.web/sites]

    > &#128221; プロバイダーが未登録の場合は、そのプロバイダーを選択して **Register** を選択してください。

1. Web ブラウザーのウィンドウまたはタブを閉じてください。

[docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts]: https://docs.microsoft.com/azure/templates/microsoft.documentdb/databaseaccounts
[docs.microsoft.com/azure/templates/microsoft.insights/components]: https://docs.microsoft.com/azure/templates/microsoft.insights/components
[docs.microsoft.com/azure/templates/microsoft.keyvault/vaults]: https://docs.microsoft.com/azure/templates/microsoft.keyvault/vaults
[docs.microsoft.com/azure/templates/microsoft.search/searchservices]: https://docs.microsoft.com/azure/templates/microsoft.search/searchservices
[docs.microsoft.com/azure/templates/microsoft.web/sites]: https://docs.microsoft.com/azure/templates/microsoft.web/sites