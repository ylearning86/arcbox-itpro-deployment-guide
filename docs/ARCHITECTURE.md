# プロジェクト構成

```
ArcBox/
├── main.bicep                    # メインの Bicep テンプレート
├── main.bicepparam               # パラメータファイル（機密情報を含む、.gitignoreに追加済み）
├── main.bicepparam.example       # パラメータファイルの例
├── README.md                     # プロジェクト概要と使用方法
├── DEPLOYMENT_GUIDE.md           # 詳細なデプロイガイド
├── QUICKSTART.md                 # クイックスタートガイド
├── .gitignore                    # Git除外設定
└── docs/
    ├── ARCHITECTURE.md           # このファイル
    ├── TROUBLESHOOTING.md        # トラブルシューティングガイド（作成予定）
    └── FAQ.md                    # よくある質問（作成予定）
```

## アーキテクチャ概要

### リソース構成

```
ArcBox リソースグループ
├── ネットワーク
│   ├── Virtual Network (10.16.0.0/16)
│   │   ├── ArcBox-Subnet (10.16.1.0/24)
│   │   ├── ArcBox-AKS-Subnet (10.16.3.0/24)
│   │   └── ArcBox-DC-Subnet (10.16.2.0/24)
│   ├── Network Security Group
│   ├── NAT Gateway
│   └── Public IP Address (オプション)
│
├── コンピューティング
│   ├── ArcBox-Client VM (Windows Server)
│   │   └── 6台の仮想 Azure Arc enabled servers
│   └── Azure Bastion (オプション)
│
├── 管理
│   ├── Log Analytics ワークスペース
│   ├── Storage Account (ステージング用)
│   └── Azure Policy 割り当て
│
└── Azure Arc
    ├── 6 x Azure Arc enabled servers
    └── 1 x Azure Arc enabled SQL Server
```

### デプロイフロー

```
1. リソースグループ作成
   ↓
2. ストレージアカウントデプロイ
   ↓
3. ネットワークリソースデプロイ
   ├── VNet/Subnet
   ├── NSG
   └── NAT Gateway
   ↓
4. Log Analytics ワークスペース
   ↓
5. Azure Policy 設定
   ↓
6. クライアント VM デプロイ
   ↓
7. VM 拡張機能 (CustomScriptExtension)
   ├── Bootstrap.ps1 実行
   ├── Arc エージェントインストール
   └── SQL Server セットアップ
   ↓
8. Bastion デプロイ (オプション)
```

## コンポーネント詳細

### 1. ArcBox-Client VM

**スペック:**
- VM サイズ: Standard_D4s_v4
  - vCPU: 4
  - メモリ: 16 GiB
- OS: Windows Server 2025 Datacenter
- ディスク: Premium SSD

**役割:**
- Azure Arc のデモ/学習環境
- 6台の仮想サーバーをホスト（Hyper-V）
- Arc エージェント管理

### 2. Azure Arc Enabled Servers

クライアント VM 上に6台の仮想サーバー:

| サーバー名 | OS | 役割 |
|-----------|-----|------|
| ArcBox-Win2K19 | Windows Server 2019 | Windows サーバー管理 |
| ArcBox-Win2K22 | Windows Server 2022 | Windows サーバー管理 |
| ArcBox-SQL | Windows Server + SQL | SQL Server on Arc |
| ArcBox-Ubuntu-01 | Ubuntu 22.04 | Linux サーバー管理 |
| ArcBox-Ubuntu-02 | Ubuntu 22.04 | Linux サーバー管理 |
| ArcBox-CentOS | CentOS | Linux サーバー管理 |

### 3. ネットワーク構成

**アドレス空間:**
- VNet: 10.16.0.0/16
- ArcBox-Subnet: 10.16.1.0/24
  - ArcBox-Client VM
- ArcBox-AKS-Subnet: 10.16.3.0/24
  - 将来の拡張用
- ArcBox-DC-Subnet: 10.16.2.0/24
  - DataOps フレーバー用

**セキュリティ:**
- NSG ルール: RDP (3389) のみ許可
- NAT Gateway: アウトバウンド接続
- Bastion (オプション): セキュアなアクセス

### 4. 監視とログ

**Log Analytics:**
- VM Insights
- Azure Arc insights
- カスタムログ収集
- パフォーマンスカウンター

**収集データ:**
- システムメトリクス
- イベントログ
- Arc エージェントログ
- カスタムアプリケーションログ

### 5. 自動化スクリプト

**Bootstrap.ps1:**
デプロイ後の初期設定を自動化:

```powershell
# 主な処理
1. 必要なツールのインストール
   - Azure CLI
   - PowerShell モジュール
   - Hyper-V 機能

2. Hyper-V VM の作成と構成

3. Arc エージェントのインストール

4. SQL Server のセットアップ

5. デスクトップショートカットの作成

6. ログとレポート生成
```

## セキュリティ考慮事項

### 1. 認証と認可

- **管理者アカウント**: 強力なパスワード必須
- **Azure RBAC**: 最小権限の原則
- **Managed Identity**: VM の Azure 認証

### 2. ネットワークセキュリティ

- **NSG**: 最小限のインバウンドルール
- **NAT Gateway**: 静的アウトバウンド IP
- **Bastion**: パブリック IP 不要のアクセス

### 3. データ保護

- **ディスク暗号化**: Azure Disk Encryption (オプション)
- **転送中の暗号化**: TLS/SSL
- **保存時の暗号化**: Azure Storage

### 4. コンプライアンス

- **Azure Policy**: 標準準拠を強制
- **更新管理**: セキュリティパッチ適用
- **監査ログ**: すべての操作を記録

## コスト最適化

### 推定月額コスト（日本東部）

| リソース | コスト/月 |
|---------|-----------|
| VM (D4s_v4) | ¥15,000-20,000 |
| ディスク (128GB Premium) | ¥1,500-2,000 |
| Log Analytics | ¥500-2,000 |
| ネットワーク | ¥500-1,000 |
| Bastion (オプション) | ¥15,000 |
| **合計** | **¥17,500-40,000** |

### コスト削減策

1. **自動シャットダウン**
   - 非営業時間に自動停止
   - 最大70%のコスト削減

2. **スポット VM** (オプション)
   - 最大90%割引
   - リスク: VM が削除される可能性

3. **適切なサイズ選択**
   - 学習用途: B シリーズでも可能
   - 本番評価: D シリーズ推奨

4. **リソースの削除**
   - 使用後は必ず削除
   - `az group delete` でまとめて削除

## パフォーマンス考慮事項

### VM サイズの選択

| 用途 | 推奨サイズ | 理由 |
|-----|-----------|------|
| 基本学習 | Standard_D2s_v4 | コスト効率的 |
| 標準デモ | Standard_D4s_v4 | バランス型 |
| 高負荷テスト | Standard_D8s_v4 | パフォーマンス重視 |

### ディスク構成

- **OS ディスク**: Premium SSD (P10)
- **データディスク**: なし（デフォルト）
- **キャッシュ**: ReadWrite

## スケーラビリティ

### 水平スケール

現在の制限:
- 6台の Arc enabled servers（固定）

拡張オプション:
- 追加 VM の手動作成
- Bicep テンプレートのカスタマイズ

### 垂直スケール

VM サイズの変更:
```bash
az vm deallocate --resource-group ArcBox-RG --name ArcBox-Client
az vm resize --resource-group ArcBox-RG --name ArcBox-Client --size Standard_D8s_v4
az vm start --resource-group ArcBox-RG --name ArcBox-Client
```

## 統合ポイント

### Azure サービス

- **Azure Monitor**: メトリクスとログ
- **Azure Policy**: ガバナンス
- **Azure Security Center**: セキュリティ管理
- **Azure Update Management**: パッチ管理
- **Azure Automation**: 自動化タスク

### サードパーティツール

- **Terraform**: IaC の代替
- **Ansible**: 構成管理
- **Grafana**: 可視化
- **Prometheus**: メトリクス収集

## 災害復旧

### バックアップ戦略

1. **Azure Backup** (オプション)
   - VM の定期バックアップ
   - RPO: 24時間
   - RTO: 1-2時間

2. **エクスポート**
   ```bash
   az vm export --resource-group ArcBox-RG --name ArcBox-Client
   ```

### リカバリ手順

1. Bicep テンプレートから再デプロイ
2. バックアップから復元（設定している場合）
3. カスタム設定の再適用

## 監視とアラート

### 主要メトリクス

- **CPU 使用率**: > 80% でアラート
- **メモリ使用率**: > 85% でアラート
- **ディスク空き容量**: < 10GB でアラート
- **Arc エージェント状態**: オフラインでアラート

### ログクエリ例

```kusto
// 過去24時間の Arc サーバー接続状態
Heartbeat
| where TimeGenerated > ago(24h)
| where ResourceType == "machines"
| summarize LastHeartbeat = max(TimeGenerated) by Computer
| where LastHeartbeat < ago(5m)
```

## カスタマイズガイド

### VM サイズの変更

`main.bicep` で変更:
```bicep
var vmSize = 'Standard_D8s_v4'  // デフォルト: D4s_v4
```

### 追加の NSG ルール

`mgmt/mgmtArtifacts.bicep` で追加:
```bicep
{
  name: 'Allow-HTTPS'
  properties: {
    priority: 110
    protocol: 'Tcp'
    access: 'Allow'
    direction: 'Inbound'
    sourceAddressPrefix: 'Internet'
    destinationAddressPrefix: '*'
    destinationPortRange: '443'
  }
}
```

### カスタムタグの追加

`main.bicepparam` で設定:
```bicep
param resourceTags = {
  Environment: 'Production'
  CostCenter: 'IT-1234'
  Owner: 'john.doe@company.com'
}
```

## トラブルシューティングフロー

```
問題発生
  ↓
1. ログ確認
   - Activity Log
   - VM ログ (C:\ArcBox\Logs)
   - Arc エージェントログ
  ↓
2. リソース状態確認
   - VM の電源状態
   - Arc 接続状態
   - NSG ルール
  ↓
3. 接続テスト
   - RDP 接続
   - Azure Portal からの接続
   - Bastion 経由の接続
  ↓
4. 再デプロイ（必要に応じて）
```

## ベストプラクティス

### デプロイ前

- [ ] クォータを確認
- [ ] 命名規則を決定
- [ ] タグ戦略を計画
- [ ] コスト予算を設定

### デプロイ中

- [ ] What-If を実行
- [ ] パラメータを検証
- [ ] デプロイを監視
- [ ] エラーに注意

### デプロイ後

- [ ] リソースを確認
- [ ] 接続をテスト
- [ ] 監視を設定
- [ ] ドキュメント化

### 運用中

- [ ] 定期的なバックアップ
- [ ] コストの監視
- [ ] セキュリティ更新
- [ ] 使用後は削除

## 参考資料

### 公式ドキュメント

- [Azure Arc Overview](https://docs.microsoft.com/azure/azure-arc/overview)
- [Azure Arc Jumpstart](https://azurearcjumpstart.io/)
- [Bicep Documentation](https://docs.microsoft.com/azure/azure-resource-manager/bicep/)

### コミュニティ

- [GitHub - Azure Arc](https://github.com/microsoft/azure_arc)
- [Microsoft Q&A](https://docs.microsoft.com/answers/topics/azure-arc.html)
- [Twitter - #AzureArc](https://twitter.com/search?q=%23AzureArc)

---

**更新日**: 2024-10-15  
**バージョン**: 1.0
