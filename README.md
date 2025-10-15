# ArcBox ITPro - Bicep デプロイメント

[![License: MIT](https://img.shields.io/badge/License-MIT-yellow.svg)](https://opensource.org/licenses/MIT)
[![Azure](https://img.shields.io/badge/Azure-Arc-0078D4?logo=microsoft-azure)](https://azure.microsoft.com/products/azure-arc/)
[![Bicep](https://img.shields.io/badge/IaC-Bicep-blue)](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)
[![Documentation](https://img.shields.io/badge/docs-complete-success)](./PROJECT_OVERVIEW.md)
[![Language](https://img.shields.io/badge/language-Japanese-red)](./README.md)

Azure Arc Jumpstart ArcBox ITPro のデプロイ用ドキュメントとガイドです。

> **⚠️ 注意**: このリポジトリには完全な Bicep テンプレートは含まれていません。デプロイには[公式の Azure Arc リポジトリ](https://github.com/microsoft/azure_arc)を使用してください。

> **🎯 プロジェクトナビゲーション**: [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md) でドキュメント全体のガイドを確認できます。

## 📚 ドキュメント一覧

| ドキュメント | 説明 | 対象者 |
|-------------|------|--------|
| [QUICKSTART.md](QUICKSTART.md) | 5分でデプロイ開始 | すべて |
| [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md) | 詳細なデプロイ手順 | 初心者 |
| [DEPLOYMENT_CHECKLIST.md](DEPLOYMENT_CHECKLIST.md) | 実行チェックリスト | 実行者 |
| [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) | システムアーキテクチャ | 上級者 |
| [PROJECT_OVERVIEW.md](PROJECT_OVERVIEW.md) | プロジェクト概要 | すべて |

## 概要

ArcBox は、Azure Arc enabled servers の機能を実践的に学習できるサンドボックス環境です。このリポジトリには、ITPro フレーバー用の Bicep テンプレートが含まれています。

## デプロイされるリソース

- クライアント VM (Windows Server)
- 仮想ネットワークとサブネット
- Log Analytics ワークスペース
- ストレージアカウント
- Azure Arc enabled servers (6台のサーバー)
- オプション: Azure Bastion

## 前提条件

1. **Azure サブスクリプション**: アクティブな Azure サブスクリプションが必要です
2. **Azure CLI**: バージョン 2.53.0 以降
   ```bash
   az --version
   ```
3. **Bicep CLI**: Azure CLI に含まれています
4. **権限**: サブスクリプションレベルで Contributor 以上のロールが必要
5. **クォータ**: 以下のリソースに対する十分なクォータ
   - vCPUs: 約 16 個
   - Storage accounts
   - Public IP addresses

## デプロイ手順

### 1. リポジトリのクローン（またはファイルのダウンロード）

```bash
cd c:\git\ArcBox
```

### 2. パラメータファイルの編集

`main.bicepparam` ファイルを編集して、以下の値を設定してください:

```bicep
param tenantId = '<your-tenant-id>'  // Azure AD テナント ID
param windowsAdminUsername = 'arcdemo'  // 管理者ユーザー名
param windowsAdminPassword = '<your-secure-password>'  // 強力なパスワード
param logAnalyticsWorkspaceName = 'ArcBox-Workspace-001'  // 一意のワークスペース名
```

#### パスワード要件
- 12-123 文字
- 以下のうち3つを含む:
  - 小文字
  - 大文字
  - 数字
  - 特殊文字

### 3. Azure へのログイン

```bash
az login
az account set --subscription "<your-subscription-id>"
```

### 4. リソースグループの作成

```bash
az group create --name ArcBox-RG --location japaneast
```

### 5. デプロイの実行

**注意**: 現在、このリポジトリには基本的な `main.bicep` のみが含まれています。完全なデプロイには、公式の Azure Arc Jumpstart リポジトリからモジュールファイルをダウンロードする必要があります。

#### オプション A: 公式リポジトリを使用（推奨）

```bash
# 公式リポジトリをクローン
git clone https://github.com/microsoft/azure_arc.git
cd azure_arc/azure_jumpstart_arcbox/bicep

# パラメータファイルを編集
code main.bicepparam

# デプロイ
az deployment group create \
  --resource-group ArcBox-RG \
  --template-file main.bicep \
  --parameters main.bicepparam
```

#### オプション B: リモートテンプレートを使用

```bash
az deployment group create \
  --resource-group ArcBox-RG \
  --template-uri "https://raw.githubusercontent.com/microsoft/azure_arc/main/azure_jumpstart_arcbox/bicep/main.bicep" \
  --parameters @main.bicepparam
```

### 6. デプロイの監視

デプロイには約 60-90 分かかります。

```bash
# デプロイ状態の確認
az deployment group show \
  --resource-group ArcBox-RG \
  --name main \
  --query properties.provisioningState
```

## デプロイ後の確認

1. **Azure Portal でリソースを確認**
   - https://portal.azure.com
   - リソースグループ "ArcBox-RG" に移動

2. **クライアント VM に接続**
   ```bash
   # パブリック IP アドレスの取得
   az vm show -d -g ArcBox-RG -n ArcBox-Client --query publicIps -o tsv
   ```

3. **Azure Arc enabled servers の確認**
   - Azure Portal > Azure Arc > Servers
   - 6台のサーバーが登録されているか確認

## パラメータの説明

| パラメータ | 説明 | デフォルト値 |
|-----------|------|-------------|
| `tenantId` | Azure AD テナント ID | - |
| `windowsAdminUsername` | VM の管理者ユーザー名 | arcdemo |
| `windowsAdminPassword` | VM の管理者パスワード | - |
| `logAnalyticsWorkspaceName` | Log Analytics ワークスペース名 | - |
| `flavor` | デプロイするフレーバー | ITPro |
| `deployBastion` | Azure Bastion をデプロイするか | false |
| `location` | デプロイ先のリージョン | japaneast |
| `rdpPort` | RDP ポート番号 | 3389 |

## カスタマイズ

### Bastion のデプロイ

より安全なアクセスのために Azure Bastion を使用する場合:

```bicep
param deployBastion = true
param bastionSku = 'Basic'
```

### 自動シャットダウンの設定

コスト削減のために VM の自動シャットダウンを設定:

```bicep
param autoShutdownEnabled = true
param autoShutdownTime = '1800'  // 18:00 (24時間形式)
param autoShutdownTimezone = 'Tokyo Standard Time'
```

## トラブルシューティング

### デプロイエラー

1. **クォータ不足**
   ```bash
   az vm list-usage --location japaneast -o table
   ```

2. **名前の競合**
   - `logAnalyticsWorkspaceName` を一意の名前に変更

3. **ログの確認**
   ```bash
   az deployment group show \
     --resource-group ArcBox-RG \
     --name main \
     --query properties.error
   ```

### VM 接続の問題

1. **NSG ルールの確認**
   ```bash
   az network nsg rule list \
     --resource-group ArcBox-RG \
     --nsg-name ArcBox-NSG \
     -o table
   ```

2. **VM の状態確認**
   ```bash
   az vm get-instance-view \
     --resource-group ArcBox-RG \
     --name ArcBox-Client
   ```

## クリーンアップ

デプロイしたリソースを削除する場合:

```bash
az group delete --name ArcBox-RG --yes --no-wait
```

## 参考リンク

- [Azure Arc Jumpstart 公式サイト](https://azurearcjumpstart.io/)
- [ArcBox ITPro ドキュメント](https://azurearcjumpstart.io/azure_jumpstart_arcbox/ITPro/)
- [Azure Arc ドキュメント](https://learn.microsoft.com/azure/azure-arc/)
- [Azure Bicep ドキュメント](https://learn.microsoft.com/azure/azure-resource-manager/bicep/)

## ライセンス

このプロジェクトは MIT ライセンスの下でライセンスされています。

## サポート

問題や質問がある場合は、[Azure Arc Jumpstart GitHub Issues](https://github.com/microsoft/azure_arc/issues) で報告してください。
