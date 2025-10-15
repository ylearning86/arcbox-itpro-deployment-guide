# 📁 ArcBox ITPro - プロジェクト概要

## ファイル構成

```
ArcBox/
├── 📄 README.md                      ★ 最初に読むファイル
├── 📄 QUICKSTART.md                  ★ 5分でデプロイ開始
├── 📄 DEPLOYMENT_GUIDE.md            詳細なデプロイ手順
├── 📄 DEPLOYMENT_CHECKLIST.md        実行チェックリスト
├── 📄 main.bicep                     メイン Bicep テンプレート
├── 📄 main.bicepparam                パラメータファイル（機密情報）
├── 📄 main.bicepparam.example        パラメータ例
├── 📄 .gitignore                     Git除外設定
└── 📁 docs/
    └── 📄 ARCHITECTURE.md            アーキテクチャドキュメント
```

## 🚀 クイックリンク

### 初めての方
1. [README.md](README.md) - プロジェクト概要
2. [QUICKSTART.md](QUICKSTART.md) - 最速デプロイ
3. [DEPLOYMENT_CHECKLIST.md](DEPLOYMENT_CHECKLIST.md) - 実行チェック

### 詳細を知りたい方
1. [DEPLOYMENT_GUIDE.md](DEPLOYMENT_GUIDE.md) - 完全なデプロイガイド
2. [docs/ARCHITECTURE.md](docs/ARCHITECTURE.md) - システムアーキテクチャ
3. [main.bicepparam.example](main.bicepparam.example) - パラメータ例

## 📋 各ファイルの説明

### README.md
**目的**: プロジェクトの入口となるドキュメント

**含まれる内容**:
- プロジェクト概要
- デプロイされるリソース
- 前提条件
- 基本的なデプロイ手順
- トラブルシューティング
- 参考リンク

**こんな時に読む**:
- 初めてこのプロジェクトを見る時
- プロジェクトの全体像を把握したい時
- 基本的なデプロイ方法を知りたい時

---

### QUICKSTART.md
**目的**: 最速でデプロイを開始する

**含まれる内容**:
- 5分でセットアップ
- 最小限の設定
- ワンライナーデプロイ
- よくあるエラーと解決方法

**こんな時に読む**:
- すぐにデプロイを始めたい時
- 詳細な説明は後で読みたい時
- 最小限の手順で進めたい時

---

### DEPLOYMENT_GUIDE.md
**目的**: 詳細で丁寧なデプロイガイド

**含まれる内容**:
- 環境準備の詳細手順
- パラメータの詳細説明
- デプロイ方法（複数の方法）
- デプロイ後の設定
- よくある質問（FAQ）
- 詳細なトラブルシューティング

**こんな時に読む**:
- 初めてAzureをデプロイする時
- 各ステップを理解しながら進めたい時
- エラーが発生して詳しく調べたい時
- カスタマイズ方法を知りたい時

---

### DEPLOYMENT_CHECKLIST.md
**目的**: デプロイを確実に実行するためのチェックリスト

**含まれる内容**:
- ステップバイステップのチェック項目
- 各段階での確認事項
- 記入欄（情報を記録できる）
- タイムライン記録
- 完了確認

**こんな時に読む**:
- 実際にデプロイを実行する時
- 抜け漏れなく進めたい時
- デプロイの記録を残したい時
- チームで作業を共有する時

---

### main.bicep
**目的**: Azure リソースをデプロイする Bicep テンプレート

**含まれる内容**:
- リソース定義
- パラメータ宣言
- 変数定義
- モジュール参照
- 出力定義

**注意**:
- ⚠️ このファイルは直接編集しない
- 公式リポジトリから最新版を取得することを推奨
- カスタマイズが必要な場合は、コピーしてから編集

---

### main.bicepparam / main.bicepparam.example
**目的**: デプロイパラメータの設定

**main.bicepparam**:
- 実際のデプロイで使用
- 機密情報を含む（.gitignore に含まれる）
- このファイルは Git にコミットしない

**main.bicepparam.example**:
- パラメータのテンプレート
- コメント付きで各項目を説明
- このファイルをコピーして使用

**含まれる設定**:
- テナント ID
- 管理者資格情報
- リソース名
- リージョン
- オプション設定（Bastion、自動シャットダウンなど）
- タグ

---

### docs/ARCHITECTURE.md
**目的**: システムアーキテクチャの詳細説明

**含まれる内容**:
- リソース構成図
- コンポーネント詳細
- ネットワーク構成
- セキュリティ設計
- コスト見積もり
- パフォーマンス考慮事項
- カスタマイズガイド

**こんな時に読む**:
- システム全体を理解したい時
- カスタマイズを計画する時
- セキュリティ要件を確認する時
- コストを見積もる時

## 🎯 使用シナリオ別ガイド

### シナリオ 1: 初めてのデプロイ
```
1. README.md を読む（5分）
   ↓
2. QUICKSTART.md で環境確認（5分）
   ↓
3. DEPLOYMENT_GUIDE.md を読みながら準備（15分）
   ↓
4. DEPLOYMENT_CHECKLIST.md に従ってデプロイ（90分）
   ↓
5. 動作確認とクリーンアップ
```

### シナリオ 2: 急いでデプロイしたい
```
1. QUICKSTART.md を開く
   ↓
2. 前提条件を確認
   ↓
3. ワンライナーデプロイを実行
   ↓
4. 完了
```

### シナリオ 3: カスタマイズしたい
```
1. ARCHITECTURE.md でシステムを理解
   ↓
2. DEPLOYMENT_GUIDE.md のカスタマイズセクションを確認
   ↓
3. main.bicepparam.example をコピー
   ↓
4. 必要に応じて main.bicep を修正
   ↓
5. デプロイ
```

### シナリオ 4: トラブルシューティング
```
1. エラーメッセージを確認
   ↓
2. README.md のトラブルシューティングセクション
   ↓
3. DEPLOYMENT_GUIDE.md の詳細トラブルシューティング
   ↓
4. GitHub Issues で検索
```

## 📊 ドキュメント詳細度マトリックス

| ドキュメント | 初心者 | 中級者 | 上級者 | 所要時間 |
|-------------|--------|--------|--------|----------|
| README.md | ★★★ | ★★☆ | ★☆☆ | 10分 |
| QUICKSTART.md | ★★★ | ★★★ | ★★☆ | 5分 |
| DEPLOYMENT_GUIDE.md | ★★★ | ★★★ | ★★☆ | 30分 |
| DEPLOYMENT_CHECKLIST.md | ★★★ | ★★☆ | ★☆☆ | 実行時 |
| ARCHITECTURE.md | ★☆☆ | ★★★ | ★★★ | 20分 |

## 🔄 デプロイフロー

```
準備フェーズ
├── README.md で概要理解
├── 前提条件確認
└── 必要な情報収集
    ↓
設定フェーズ
├── main.bicepparam.example をコピー
├── パラメータ編集
└── What-If で確認
    ↓
実行フェーズ
├── DEPLOYMENT_CHECKLIST.md を開く
├── ステップバイステップで実行
└── 各チェックポイントで確認
    ↓
確認フェーズ
├── リソース確認
├── 動作確認
└── コスト確認
    ↓
学習フェーズ
├── ARCHITECTURE.md で理解を深める
├── Azure Arc 機能を試す
└── カスタマイズを検討
    ↓
クリーンアップフェーズ
└── リソース削除
```

## 💡 ベストプラクティス

### ドキュメントの読み方

1. **最初は README.md から**
   - プロジェクト全体を理解する
   - 自分のスキルレベルを確認

2. **手を動かすなら QUICKSTART.md**
   - すぐに始められる
   - 基本を体験できる

3. **深く理解するなら DEPLOYMENT_GUIDE.md**
   - 詳細な説明
   - トラブル対応も網羅

4. **確実に進めるなら DEPLOYMENT_CHECKLIST.md**
   - 抜け漏れ防止
   - 進捗管理に最適

5. **カスタマイズなら ARCHITECTURE.md**
   - システム設計の理解
   - 変更ポイントの特定

### ファイル管理

```bash
# パラメータファイルのバージョン管理
cp main.bicepparam.example main-dev.bicepparam
cp main.bicepparam.example main-prod.bicepparam

# デプロイごとに別ファイルを作成
cp main.bicepparam.example deployment-$(date +%Y%m%d).bicepparam
```

### 情報の記録

```bash
# デプロイ記録フォルダを作成
mkdir deployments
mkdir deployments/$(date +%Y%m%d)

# チェックリストをコピーして使用
cp DEPLOYMENT_CHECKLIST.md deployments/$(date +%Y%m%d)/checklist.md

# デプロイ結果を保存
az deployment group show ... > deployments/$(date +%Y%m%d)/result.json
```

## 🆘 サポート情報

### 公式リソース
- **Azure Arc ドキュメント**: https://learn.microsoft.com/azure/azure-arc/
- **ArcBox 公式サイト**: https://azurearcjumpstart.io/azure_jumpstart_arcbox/ITPro/
- **GitHub リポジトリ**: https://github.com/microsoft/azure_arc

### コミュニティ
- **GitHub Issues**: バグ報告と機能要望
- **Microsoft Q&A**: 技術的な質問
- **Twitter**: #AzureArc でディスカッション

### 緊急時の連絡先
- **Azure サポート**: Azure Portal からチケット作成
- **ドキュメント**: このリポジトリの Issues セクション

## 📚 学習パス

### レベル 1: 基礎
1. README.md を読む
2. QUICKSTART.md でデプロイ
3. Azure Portal で確認

### レベル 2: 実践
1. DEPLOYMENT_GUIDE.md を精読
2. カスタムパラメータでデプロイ
3. Azure Arc 機能を試す

### レベル 3: 応用
1. ARCHITECTURE.md を理解
2. Bicep テンプレートをカスタマイズ
3. 独自のシナリオを実装

## 🎓 次のステップ

このプロジェクトを完了したら:

1. **他のフレーバーを試す**
   - ArcBox DevOps
   - ArcBox DataOps

2. **本番環境への適用を検討**
   - セキュリティ強化
   - 高可用性構成
   - コスト最適化

3. **コミュニティに貢献**
   - 改善提案
   - ドキュメント更新
   - 事例共有

---

**最終更新**: 2024-10-15  
**プロジェクトバージョン**: 1.0  
**対応 ArcBox バージョン**: ITPro (最新)

**質問やフィードバック**: [GitHub Issues](https://github.com/microsoft/azure_arc/issues)
