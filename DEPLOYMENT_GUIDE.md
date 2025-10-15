# ArcBox ITPro デプロイガイド

このガイドでは、Azure Arc Jumpstart ArcBox ITPro を Bicep を使用してデプロイする手順を説明します。

## 目次

1. [環境準備](#環境準備)
2. [パラメータ設定](#パラメータ設定)
3. [デプロイ実行](#デプロイ実行)
4. [デプロイ後の設定](#デプロイ後の設定)
5. [よくある質問](#よくある質問)

## 環境準備

### 必要なツールのインストール

#### Azure CLI

**Windows (PowerShell)**
```powershell
# MSIインストーラーを使用
Invoke-WebRequest -Uri https://aka.ms/installazurecliwindows -OutFile .\AzureCLI.msi
Start-Process msiexec.exe -Wait -ArgumentList '/I AzureCLI.msi /quiet'
rm .\AzureCLI.msi

# インストール確認
az --version
```

**更新**
```bash
az upgrade
```

### Azure へのログイン

```bash
# インタラクティブログイン
az login

# テナント指定
az login --tenant <tenant-id>

# サブスクリプション一覧表示
az account list --output table

# サブスクリプション設定
az account set --subscription "<subscription-name-or-id>"

# 現在の設定確認
az account show
```

### 権限の確認

デプロイには以下の権限が必要です:

```bash
# 現在のロール割り当てを確認
az role assignment list --assignee $(az ad signed-in-user show --query id -o tsv) \
  --scope /subscriptions/<subscription-id> \
  --output table
```

必要な権限:
- `Contributor` または `Owner` (サブスクリプションまたはリソースグループレベル)

## パラメータ設定

### 1. テナント ID の取得

```bash
az account show --query tenantId -o tsv
```

### 2. main.bicepparam の編集

```bicep
using 'main.bicep'

// ========== 必須パラメータ ==========
// 以下の値を実際の値に置き換えてください

param tenantId = '00000000-0000-0000-0000-000000000000'  // ← ここに実際のテナントIDを入力
param windowsAdminUsername = 'arcdemo'
param windowsAdminPassword = 'P@ssw0rd123!'  // ← 強力なパスワードに変更
param logAnalyticsWorkspaceName = 'ArcBox-Workspace-001'  // ← 一意の名前に変更

// ========== 基本設定 ==========
param flavor = 'ITPro'
param location = 'japaneast'  // または 'eastus', 'westeurope' など

// ========== ネットワーク設定 ==========
param deployBastion = false  // true にすると Azure Bastion がデプロイされます
param rdpPort = '3389'  // RDPポート番号（変更推奨）

// ========== 運用設定 ==========
param vmAutologon = true  // 自動ログオンを有効化
param autoShutdownEnabled = true
param autoShutdownTime = '1800'  // 18:00に自動シャットダウン
param autoShutdownTimezone = 'Tokyo Standard Time'

// ========== GitHub設定（通常は変更不要） ==========
param githubAccount = 'microsoft'
param githubBranch = 'main'
param githubUser = 'Azure'

// ========== タグ（任意） ==========
param resourceTags = {
  Environment: 'Sandbox'
  Project: 'ArcBox-ITPro'
  Owner: 'your-email@company.com'
  CostCenter: 'IT-Training'
}
```

### パスワード生成（PowerShell）

```powershell
# 強力なランダムパスワードを生成
Add-Type -AssemblyName System.Web
[System.Web.Security.Membership]::GeneratePassword(16, 4)
```

## デプロイ実行

### 方法1: 公式リポジトリを使用（推奨）

```bash
# 1. 公式リポジトリをクローン
git clone https://github.com/microsoft/azure_arc.git
cd azure_arc/azure_jumpstart_arcbox/bicep

# 2. パラメータファイルをコピーして編集
cp main.bicepparam my-deployment.bicepparam
code my-deployment.bicepparam  # または notepad, vim など

# 3. リソースグループを作成
az group create \
  --name ArcBox-RG \
  --location japaneast

# 4. デプロイを実行
az deployment group create \
  --resource-group ArcBox-RG \
  --template-file main.bicep \
  --parameters my-deployment.bicepparam \
  --name ArcBox-Deployment-$(date +%Y%m%d-%H%M%S)
```

### 方法2: What-If デプロイ（推奨）

実際にリソースを作成する前に、何が作成されるかを確認:

```bash
az deployment group what-if \
  --resource-group ArcBox-RG \
  --template-file main.bicep \
  --parameters my-deployment.bicepparam
```

### 方法3: インラインパラメータ

```bash
az deployment group create \
  --resource-group ArcBox-RG \
  --template-file main.bicep \
  --parameters \
    tenantId="<your-tenant-id>" \
    windowsAdminUsername="arcdemo" \
    windowsAdminPassword="<your-password>" \
    logAnalyticsWorkspaceName="ArcBox-Workspace-001" \
    flavor="ITPro" \
    deployBastion=false
```

### デプロイの監視

**別のターミナルで監視:**

```bash
# デプロイ状態をリアルタイムで監視
watch -n 10 'az deployment group show \
  --resource-group ArcBox-RG \
  --name <deployment-name> \
  --query properties.provisioningState'
```

**詳細なログを確認:**

```bash
# デプロイ操作の一覧
az deployment operation group list \
  --resource-group ArcBox-RG \
  --name <deployment-name> \
  --query "[].{Resource:properties.targetResource.resourceName, State:properties.provisioningState, Status:properties.statusMessage}" \
  --output table
```

## デプロイ後の設定

### 1. リソースの確認

```bash
# デプロイされたリソースの一覧
az resource list \
  --resource-group ArcBox-RG \
  --output table
```

### 2. クライアント VM への接続

#### Bastion を使用しない場合

```bash
# パブリック IP アドレスの取得
VM_IP=$(az vm show -d \
  --resource-group ArcBox-RG \
  --name ArcBox-Client \
  --query publicIps -o tsv)

echo "RDP接続: $VM_IP"
```

**Windows から接続:**
```powershell
mstsc /v:$VM_IP
```

#### Bastion を使用する場合

Azure Portal から:
1. Virtual Machines > ArcBox-Client
2. 「Connect」> 「Bastion」を選択
3. 資格情報を入力して接続

### 3. Azure Arc Servers の確認

```bash
# Azure Arc enabled servers の一覧
az connectedmachine list \
  --resource-group ArcBox-RG \
  --output table
```

### 4. Log Analytics の確認

```bash
# ワークスペース情報の取得
az monitor log-analytics workspace show \
  --resource-group ArcBox-RG \
  --workspace-name <your-workspace-name> \
  --output json
```

### 5. 初回ログイン後の確認項目

クライアント VM にログイン後:

1. **デスクトップ上のショートカット**
   - Azure Arc documentation
   - Azure Portal
   - PowerShell スクリプト

2. **自動実行されるスクリプト**
   - `C:\ArcBox\ArcServersLogonScript.ps1`
   - このスクリプトが Azure Arc エージェントのインストールを実行します

3. **ログの確認**
   ```powershell
   Get-Content C:\ArcBox\Logs\*.log -Tail 50
   ```

## よくある質問

### Q1: デプロイに失敗した場合はどうすればいいですか?

**A:** まず、エラーメッセージを確認します:

```bash
az deployment group show \
  --resource-group ArcBox-RG \
  --name <deployment-name> \
  --query properties.error
```

よくある原因:
- クォータ不足
- 名前の重複
- 権限不足

### Q2: デプロイにどのくらい時間がかかりますか?

**A:** 通常 60-90 分程度です。主な段階:
1. ネットワークとストレージ: 5-10分
2. VM の作成とOS設定: 20-30分
3. Azure Arc エージェントのインストール: 30-50分

### Q3: コストはどのくらいかかりますか?

**A:** 主なコスト要因:
- VM (Standard_D4s_v4): ~$150-200/月
- ストレージ: ~$5-10/月
- Log Analytics: データ量による
- Bastion (オプション): ~$140/月

**コスト削減のヒント:**
- 自動シャットダウンを有効化
- 使用後は必ずリソースを削除
- Bastion は必要な場合のみデプロイ

### Q4: 本番環境で使用できますか?

**A:** ArcBox は学習とテスト用のサンドボックスです。本番環境での使用は推奨されません。

### Q5: 既存のネットワークに統合できますか?

**A:** 標準の ArcBox はスタンドアロン構成です。既存環境への統合には Bicep テンプレートのカスタマイズが必要です。

### Q6: 日本語環境で使用できますか?

**A:** クライアント VM は英語 OS ですが、Azure Portal は日本語で利用可能です。

## トラブルシューティング

### クォータエラー

```bash
# 現在の使用状況を確認
az vm list-usage \
  --location japaneast \
  --output table

# クォータの増加をリクエスト
az support tickets create \
  --ticket-name "Quota Increase Request" \
  --severity 3 \
  --contact-first-name "Your Name" \
  --contact-last-name "Last Name" \
  --contact-method "email" \
  --contact-email "your-email@company.com" \
  --title "VM Quota Increase" \
  --description "Need to increase vCPU quota for ArcBox deployment"
```

### デプロイのロールバック

```bash
# 失敗したデプロイを削除
az deployment group delete \
  --resource-group ArcBox-RG \
  --name <deployment-name>

# リソースグループ全体を削除して再デプロイ
az group delete \
  --name ArcBox-RG \
  --yes \
  --no-wait
```

### VM に接続できない

```bash
# NSG ルールを確認
az network nsg rule list \
  --resource-group ArcBox-RG \
  --nsg-name ArcBox-NSG \
  --output table

# RDP ルールを追加（必要に応じて）
az network nsg rule create \
  --resource-group ArcBox-RG \
  --nsg-name ArcBox-NSG \
  --name Allow-RDP \
  --priority 100 \
  --source-address-prefixes "YOUR_IP_ADDRESS" \
  --destination-port-ranges 3389 \
  --access Allow \
  --protocol Tcp
```

## 次のステップ

1. **Azure Arc 機能の探索**
   - [Azure Policy の適用](https://learn.microsoft.com/azure/azure-arc/servers/policy-reference)
   - [VM 拡張機能の管理](https://learn.microsoft.com/azure/azure-arc/servers/manage-vm-extensions)
   - [更新管理](https://learn.microsoft.com/azure/azure-arc/servers/plan-at-scale-deployment#update-management)

2. **追加シナリオ**
   - SQL Server on Azure Arc
   - Kubernetes on Azure Arc
   - データサービス on Azure Arc

3. **コミュニティ**
   - [Azure Arc Jumpstart GitHub](https://github.com/microsoft/azure_arc)
   - [Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-arc.html)

## クリーンアップ

使用後は必ずリソースを削除してください:

```bash
# リソースグループの削除
az group delete \
  --name ArcBox-RG \
  --yes \
  --no-wait

# 削除の進行状況を確認
az group show \
  --name ArcBox-RG \
  --query properties.provisioningState
```

**注意**: 削除には 10-20 分かかる場合があります。
