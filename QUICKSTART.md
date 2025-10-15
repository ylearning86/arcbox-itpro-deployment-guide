# 🚀 ArcBox ITPro - クイックスタート

Azure Arc Jumpstart ArcBox ITPro を最速でデプロイする手順です。

## ⚡ 5分でセットアップ

### 前提条件チェック

- [ ] Azure サブスクリプション
- [ ] Azure CLI インストール済み
- [ ] Contributor 権限あり

### 1️⃣ 公式リポジトリを使用（最も簡単）

```bash
# リポジトリをクローン
git clone https://github.com/microsoft/azure_arc.git
cd azure_arc/azure_jumpstart_arcbox/bicep

# Azure にログイン
az login
az account set --subscription "<your-subscription-id>"

# パラメータファイルをコピー
cp main.bicepparam my-arcbox.bicepparam

# パラメータを編集（下記の「最小限の設定」参照）
code my-arcbox.bicepparam
```

### 最小限の設定

`my-arcbox.bicepparam` ファイルで以下を設定:

```bicep
using 'main.bicep'

// この3つだけ必須
param tenantId = '<az account show --query tenantId -o tsv の結果>'
param windowsAdminPassword = '<12文字以上の強力なパスワード>'
param logAnalyticsWorkspaceName = '<一意の名前、例: ArcBox-WS-20241015>'

// これらはデフォルトでOK
param windowsAdminUsername = 'arcdemo'
param flavor = 'ITPro'
param deployBastion = false
```

### デプロイ実行

```bash
# リソースグループ作成
az group create --name ArcBox-RG --location japaneast

# デプロイ開始（60-90分かかります）
az deployment group create \
  --resource-group ArcBox-RG \
  --template-file main.bicep \
  --parameters my-arcbox.bicepparam \
  --name ArcBox-$(date +%Y%m%d-%H%M%S)
```

## 📊 デプロイ監視

```bash
# 状態確認
az deployment group show \
  --resource-group ArcBox-RG \
  --name ArcBox-<timestamp> \
  --query properties.provisioningState

# または Azure Portal で確認
echo "https://portal.azure.com/#view/HubsExtension/DeploymentDetailsBlade/~/overview/id/%2Fsubscriptions%2F<subscription-id>%2FresourceGroups%2FArcBox-RG%2Fproviders%2FMicrosoft.Resources%2Fdeployments%2FArcBox-<timestamp>"
```

## ✅ デプロイ完了後

### VM に接続

```bash
# パブリック IP を取得
az vm show -d \
  --resource-group ArcBox-RG \
  --name ArcBox-Client \
  --query publicIps -o tsv
```

**接続情報:**
- ユーザー名: `arcdemo`（またはカスタマイズした値）
- パスワード: パラメータで設定した値
- ポート: 3389（RDP）

### Azure Arc Servers を確認

```bash
# 登録されたサーバーを確認
az connectedmachine list --resource-group ArcBox-RG --output table
```

## 🧹 クリーンアップ

```bash
# すべてのリソースを削除
az group delete --name ArcBox-RG --yes --no-wait
```

## 💡 ワンライナーデプロイ

すべてをコマンドラインで:

```bash
# 変数設定
TENANT_ID=$(az account show --query tenantId -o tsv)
SUBSCRIPTION_ID=$(az account show --query id -o tsv)
ADMIN_PASSWORD="<your-secure-password>"
WORKSPACE_NAME="ArcBox-WS-$(date +%Y%m%d%H%M%S)"
RG_NAME="ArcBox-RG"
LOCATION="japaneast"

# デプロイ実行
az group create --name $RG_NAME --location $LOCATION && \
az deployment group create \
  --resource-group $RG_NAME \
  --template-uri "https://raw.githubusercontent.com/microsoft/azure_arc/main/azure_jumpstart_arcbox/bicep/main.bicep" \
  --parameters \
    tenantId=$TENANT_ID \
    windowsAdminUsername="arcdemo" \
    windowsAdminPassword=$ADMIN_PASSWORD \
    logAnalyticsWorkspaceName=$WORKSPACE_NAME \
    flavor="ITPro" \
    deployBastion=false \
    location=$LOCATION
```

## 🔧 トラブルシューティング

### デプロイが失敗した場合

```bash
# エラー詳細を確認
az deployment group show \
  --resource-group ArcBox-RG \
  --name <deployment-name> \
  --query properties.error
```

### よくあるエラー

| エラー | 原因 | 解決方法 |
|--------|------|---------|
| QuotaExceeded | vCPU クォータ不足 | `az vm list-usage` で確認、クォータ増加をリクエスト |
| DuplicateResourceName | 名前の重複 | `logAnalyticsWorkspaceName` を変更 |
| AuthorizationFailed | 権限不足 | サブスクリプションの Contributor ロールを確認 |

## 📚 次のステップ

1. **詳細なドキュメント**: [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md) を参照
2. **公式ドキュメント**: https://azurearcjumpstart.io/azure_jumpstart_arcbox/ITPro/
3. **GitHub リポジトリ**: https://github.com/microsoft/azure_arc

## ⚠️ 重要な注意事項

- **コスト**: このデプロイは課金対象です（約 $5-10/日）
- **時間**: デプロイには 60-90 分かかります
- **クリーンアップ**: 使用後は必ずリソースを削除してください
- **セキュリティ**: 強力なパスワードを使用し、パブリック IP のアクセスを制限してください

## 🎯 成功の確認

デプロイが成功したら:

- ✅ リソースグループに 10+ のリソースが作成されている
- ✅ ArcBox-Client VM が実行中
- ✅ 6台の Azure Arc enabled servers が登録されている
- ✅ Log Analytics ワークスペースが作成されている

---

**必要なヘルプ**: [GitHub Issues](https://github.com/microsoft/azure_arc/issues) でサポートを受けられます
