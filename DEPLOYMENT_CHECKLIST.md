# 🎯 ArcBox ITPro デプロイ - 実行チェックリスト

このチェックリストに従って、ArcBox ITPro を確実にデプロイしましょう。

## ✅ デプロイ前の準備

### 環境確認
- [ ] Azure サブスクリプションがある
- [ ] Azure CLI がインストール済み（`az --version` で確認）
- [ ] Contributor 権限がある（`az role assignment list` で確認）
- [ ] 十分なクォータがある（vCPU x16 以上）

### 情報収集
```bash
# テナント ID を取得
az account show --query tenantId -o tsv

# サブスクリプション ID を確認
az account show --query id -o tsv

# 利用可能なリージョンを確認
az account list-locations --query "[].name" -o tsv
```

収集した情報:
- テナント ID: `____________________________`
- サブスクリプション ID: `____________________________`
- デプロイ先リージョン: `____________________________`

### パスワード準備
強力なパスワードを生成（12文字以上、大文字・小文字・数字・特殊文字を含む）:
```powershell
# PowerShell で生成
Add-Type -AssemblyName System.Web
[System.Web.Security.Membership]::GeneratePassword(16, 4)
```

生成したパスワード: `____________________________`

## 🚀 デプロイ実行

### ステップ 1: 公式リポジトリのクローン

```bash
# 作業ディレクトリに移動
cd c:/git

# リポジトリをクローン
git clone https://github.com/microsoft/azure_arc.git

# ArcBox ディレクトリに移動
cd azure_arc/azure_jumpstart_arcbox/bicep
```

- [ ] リポジトリのクローン完了

### ステップ 2: Azure へのログイン

```bash
# Azure にログイン
az login

# 正しいサブスクリプションを選択
az account set --subscription "<your-subscription-id>"

# 確認
az account show --output table
```

- [ ] Azure ログイン完了
- [ ] 正しいサブスクリプション選択完了

### ステップ 3: パラメータファイルの準備

```bash
# パラメータファイルをコピー
cp main.bicepparam my-arcbox-deployment.bicepparam

# エディタで開く
code my-arcbox-deployment.bicepparam
# または
notepad my-arcbox-deployment.bicepparam
```

**編集する項目:**
```bicep
using 'main.bicep'

// 必須: 実際の値に置き換え
param tenantId = '<ステップ1で取得したテナントID>'
param windowsAdminUsername = 'arcdemo'
param windowsAdminPassword = '<準備したパスワード>'
param logAnalyticsWorkspaceName = 'ArcBox-Workspace-20241015-001'  // 一意の名前に

// 基本設定
param flavor = 'ITPro'
param location = 'japaneast'  // または 'eastus'
param deployBastion = false
param vmAutologon = true

// コスト最適化
param autoShutdownEnabled = true
param autoShutdownTime = '1800'  // 18:00
param autoShutdownTimezone = 'Tokyo Standard Time'

// タグ（任意）
param resourceTags = {
  Environment: 'Sandbox'
  Project: 'ArcBox-ITPro'
  Owner: '<your-email>'
}
```

- [ ] パラメータファイル編集完了
- [ ] 必須パラメータすべて設定済み

### ステップ 4: What-If 実行（推奨）

```bash
az deployment group what-if \
  --resource-group ArcBox-RG \
  --template-file main.bicep \
  --parameters my-arcbox-deployment.bicepparam \
  --location japaneast
```

**確認事項:**
- [ ] 作成されるリソース数を確認（約20個）
- [ ] 予期しないリソースがないか確認
- [ ] エラーがないか確認

### ステップ 5: リソースグループ作成

```bash
az group create \
  --name ArcBox-RG \
  --location japaneast
```

- [ ] リソースグループ作成完了

### ステップ 6: デプロイ実行

```bash
# デプロイ開始（タイムスタンプ付き）
az deployment group create \
  --resource-group ArcBox-RG \
  --template-file main.bicep \
  --parameters my-arcbox-deployment.bicepparam \
  --name ArcBox-Deployment-$(date +%Y%m%d-%H%M%S)

# デプロイ名を記録
DEPLOYMENT_NAME="ArcBox-Deployment-YYYYMMDD-HHMMSS"
```

デプロイ名: `____________________________`  
開始時刻: `____________________________`

- [ ] デプロイ実行開始

## ⏱️ デプロイ中の監視（60-90分）

### 別のターミナルで状態を監視

```bash
# リアルタイム監視（10秒ごと更新）
watch -n 10 'az deployment group show \
  --resource-group ArcBox-RG \
  --name <your-deployment-name> \
  --query properties.provisioningState'
```

または Azure Portal で確認:
```
https://portal.azure.com/#view/HubsExtension/DeploymentDetailsBlade/~/overview/id/%2Fsubscriptions%2F<subscription-id>%2FresourceGroups%2FArcBox-RG%2Fproviders%2FMicrosoft.Resources%2Fdeployments%2F<deployment-name>
```

### チェックポイント

**10分後:**
- [ ] ストレージアカウント作成完了
- [ ] VNet/Subnet 作成完了

**20分後:**
- [ ] Log Analytics ワークスペース作成完了
- [ ] NSG 作成完了

**40分後:**
- [ ] VM 作成完了
- [ ] VM が実行中

**60分後:**
- [ ] CustomScriptExtension 実行中
- [ ] Arc エージェントインストール開始

**90分後:**
- [ ] デプロイ完了（Succeeded）

### デプロイ完了の確認

```bash
# デプロイ状態を確認
az deployment group show \
  --resource-group ArcBox-RG \
  --name <deployment-name> \
  --query properties.provisioningState

# 出力: "Succeeded" になっていること
```

完了時刻: `____________________________`  
所要時間: `____________________________`

- [ ] デプロイが "Succeeded" で完了

## ✓ デプロイ後の確認

### リソースの確認

```bash
# すべてのリソースを一覧表示
az resource list \
  --resource-group ArcBox-RG \
  --output table

# リソース数をカウント
az resource list \
  --resource-group ArcBox-RG \
  --query "length(@)"
```

期待されるリソース数: 約20個

- [ ] 期待通りのリソース数が作成されている

### VM の確認

```bash
# VM の状態を確認
az vm show \
  --resource-group ArcBox-RG \
  --name ArcBox-Client \
  --show-details \
  --query "{Name:name, PowerState:powerState, PublicIP:publicIps}" \
  --output table
```

- [ ] VM が "VM running" 状態
- [ ] パブリック IP が割り当てられている（Bastion なしの場合）

### Arc Servers の確認

```bash
# Arc enabled servers の一覧
az connectedmachine list \
  --resource-group ArcBox-RG \
  --output table

# Arc servers の数を確認
az connectedmachine list \
  --resource-group ArcBox-RG \
  --query "length(@)"
```

期待される Arc servers 数: 6個

- [ ] 6台の Arc servers が登録されている
- [ ] すべて "Connected" 状態

### VM への接続テスト

```bash
# パブリック IP を取得
VM_IP=$(az vm show -d \
  --resource-group ArcBox-RG \
  --name ArcBox-Client \
  --query publicIps -o tsv)

echo "RDP接続先: $VM_IP"
echo "ユーザー名: arcdemo"
echo "ポート: 3389"
```

接続情報:
- IP アドレス: `____________________________`
- ユーザー名: `arcdemo`
- パスワード: `<設定したパスワード>`

**Windows から RDP 接続:**
```powershell
mstsc /v:$VM_IP
```

- [ ] RDP で VM に接続成功
- [ ] デスクトップが表示される

### VM 内部の確認（VM にログイン後）

```powershell
# ログフォルダを確認
Get-ChildItem C:\ArcBox\Logs

# 最新のログを表示
Get-Content C:\ArcBox\Logs\Bootstrap.log -Tail 50

# Arc エージェントの状態を確認
azcmagent show
```

- [ ] ログファイルが存在する
- [ ] Arc エージェントが正常に動作している

## 📊 動作確認

### Azure Portal での確認

1. **リソースグループ**
   - [ ] https://portal.azure.com
   - [ ] ArcBox-RG に移動
   - [ ] すべてのリソースが表示される

2. **Azure Arc**
   - [ ] Azure Arc > Servers に移動
   - [ ] 6台のサーバーが表示される
   - [ ] すべて "Connected" 状態

3. **Log Analytics**
   - [ ] Log Analytics ワークスペースを開く
   - [ ] ログクエリを実行:
     ```kusto
     Heartbeat
     | where TimeGenerated > ago(1h)
     | summarize count() by Computer
     ```

### コスト確認

```bash
# 現在のコストを確認
az consumption usage list \
  --start-date $(date -d '1 day ago' +%Y-%m-%d) \
  --end-date $(date +%Y-%m-%d) \
  --query "[?contains(instanceName, 'ArcBox')].{Name:instanceName, Cost:pretaxCost}" \
  --output table
```

- [ ] 予想コスト範囲内である

### 自動シャットダウンの確認

```bash
# 自動シャットダウン設定を確認
az vm show \
  --resource-group ArcBox-RG \
  --name ArcBox-Client \
  --query "scheduledEventsProfile" \
  --output json
```

- [ ] 自動シャットダウンが設定されている（設定した場合）

## 🎓 次のステップ

### 学習リソース

- [ ] [Azure Arc ドキュメント](https://learn.microsoft.com/azure/azure-arc/)を読む
- [ ] [ArcBox ITPro ガイド](https://azurearcjumpstart.io/azure_jumpstart_arcbox/ITPro/)を実施
- [ ] Azure Policy を試す
- [ ] VM 拡張機能を試す

### 実践演習

1. **Azure Policy の適用**
   ```bash
   az policy assignment create \
     --name 'audit-vm-managed-disks' \
     --scope "/subscriptions/<subscription-id>/resourceGroups/ArcBox-RG" \
     --policy "/providers/Microsoft.Authorization/policyDefinitions/06a78e20-9358-41c9-923c-fb736d382a4d"
   ```

2. **VM 拡張機能のインストール**
   ```bash
   az connectedmachine extension create \
     --machine-name ArcBox-Win2K19 \
     --resource-group ArcBox-RG \
     --name CustomScriptExtension \
     --type CustomScriptExtension \
     --publisher Microsoft.Compute
   ```

3. **Log Analytics クエリ**
   - カスタムクエリを作成
   - ダッシュボードを作成

## 🗑️ クリーンアップ（使用後）

### デプロイ情報の記録

デプロイ情報を記録してから削除:
```bash
# リソースリストをエクスポート
az resource list \
  --resource-group ArcBox-RG \
  --output json > arcbox-resources-$(date +%Y%m%d).json

# デプロイ情報をエクスポート
az deployment group export \
  --resource-group ArcBox-RG \
  --name <deployment-name> > arcbox-deployment-$(date +%Y%m%d).json
```

- [ ] リソース情報をバックアップ

### リソースの削除

```bash
# リソースグループを削除（すべてのリソースが削除されます）
az group delete \
  --name ArcBox-RG \
  --yes \
  --no-wait

# 削除状態を確認
az group exists --name ArcBox-RG
# 出力: false になればOK
```

削除開始時刻: `____________________________`

- [ ] リソースグループ削除実行
- [ ] 削除完了確認（10-20分後）

### コスト確認

```bash
# 最終的なコストを確認
az consumption usage list \
  --start-date <deployment-date> \
  --end-date $(date +%Y-%m-%d) \
  --query "[?contains(instanceName, 'ArcBox')].{Name:instanceName, Cost:pretaxCost}" \
  --output table
```

総コスト: `____________________________`

- [ ] 最終コスト確認
- [ ] 予算内に収まっている

## 📝 振り返り

### 成功要因
記録してください:
- 
- 
- 

### 課題・問題点
記録してください:
- 
- 
- 

### 学んだこと
記録してください:
- 
- 
- 

## 🆘 トラブルシューティング

### よくある問題

**問題1: デプロイが失敗する**
```bash
# エラー詳細を確認
az deployment group show \
  --resource-group ArcBox-RG \
  --name <deployment-name> \
  --query properties.error
```

**問題2: VM に接続できない**
```bash
# NSG ルールを確認
az network nsg rule list \
  --resource-group ArcBox-RG \
  --nsg-name ArcBox-NSG \
  --output table
```

**問題3: Arc servers が登録されない**
- VM 内のログを確認: `C:\ArcBox\Logs\Bootstrap.log`
- Arc エージェントを手動で再インストール

## ✅ 完了チェック

すべてのステップが完了したら、このチェックリストをチェック:

- [ ] デプロイ前の準備完了
- [ ] デプロイ実行完了
- [ ] デプロイ後の確認完了
- [ ] 動作確認完了
- [ ] 学習リソース確認
- [ ] クリーンアップ完了（使用後）
- [ ] ドキュメント化完了

**おめでとうございます! 🎉**

ArcBox ITPro のデプロイが完了しました。Azure Arc の世界をお楽しみください!

---

**チェックリスト記入日**: ____________________________  
**実施者**: ____________________________  
**メモ**:
