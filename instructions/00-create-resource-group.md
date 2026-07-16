---
lab:
    title: 'ラボ用リソース グループを作成する'
    module: 'Setup'
---

# ラボ用 Azure リソース グループを作成する

このラボを完了する前に、新しくデプロイする Azure リソースを配置するための新しい [resource group][docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal] を作成してください。

1. 新しい Web ブラウザーのウィンドウまたはタブで Azure portal (``portal.azure.com``) に移動してください。

1. サブスクリプションに関連付けられた Microsoft 資格情報を使用してポータルにサインインしてください。

1. **Home** ページで **Resource groups** を選択してください。

    > &#128161; 別の方法として、**&#8801;** メニューを展開し、**All Services** を選択して、**All** カテゴリで **Resource groups** を選択してください。

1. **+ Create** を選択してください。

1. **Create a resource group** ポップアップで、次の設定を使用して新しいリソース グループを作成してください。残りの設定は既定値のままにしてください。

    | **Setting** | **Value** |
    | ---: | :--- |
    | **Subscription** | *Your existing Azure subscription* |
    | **Resource group** | *Give your resource group a unique name* |
    | **Region** | *Choose any available region* |

1. このタスクを続行する前に、デプロイ タスクが完了するまで待機してください。

[docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal]: https://docs.microsoft.com/azure/azure-resource-manager/management/manage-resource-groups-portal