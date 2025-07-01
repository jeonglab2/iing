# lostitem_app システムドキュメント

## 概要

lostitem_appは、拾得物（落とし物）の管理を行うWebアプリケーションです。FlaskフレームワークをベースとしたPythonアプリケーションで、AI画像認識機能を搭載した包括的な拾得物管理システムです。

## システムの目的

- 拾得物の登録・管理
- 拾得者情報の記録
- 遺失者との連絡・返還処理
- 警察への届出書類作成
- AI技術を活用した物品の自動分類

## 技術スタック

### バックエンド
- **フレームワーク**: Flask 2.3.2
- **データベース**: SQLAlchemy 2.0.13 (SQLite)
- **認証**: Flask-Login 0.6.2
- **フォーム処理**: Flask-WTF 1.1.1, WTForms 3.0.1
- **データベース移行**: Flask-Migrate 4.0.4
- **PDF生成**: ReportLab 4.0.7

### AI・機械学習
- **深層学習**: PyTorch 2.2.0, TorchVision 0.17.0
- **画像認識**: Open-CLIP-Torch 2.24.0
- **自然言語処理**: Transformers 4.37.2, Tokenizers 0.15.1
- **画像処理**: Pillow 10.1.0

### その他
- **翻訳**: GoogleTrans 4.0.0rc1
- **HTTP通信**: Requests 2.31.0
- **環境変数管理**: Python-dotenv 1.0.0

## アーキテクチャ

### アプリケーション構造

```
lostitem_app/
├── apps/
│   ├── app.py              # メインアプリケーション
│   ├── config.sample       # 設定ファイルサンプル
│   ├── auth/               # 認証機能
│   ├── register/           # 拾得物登録機能
│   ├── items/              # 拾得物管理機能
│   ├── crud/               # ユーザー管理機能
│   ├── police/             # 警察届出機能
│   ├── return_item/        # 返還処理機能
│   ├── refund/             # 還付処理機能
│   ├── disposal/           # 廃棄処理機能
│   ├── bundleditems/       # 同梱物管理機能
│   ├── notfound/           # 遺失物管理機能
│   ├── images/             # 画像保存フォルダ
│   ├── renamed_images/     # リネーム済み画像フォルダ
│   ├── static/             # 静的ファイル
│   └── PDFfile/            # PDF出力フォルダ
├── requirements.txt        # 依存関係
└── README.md              # セットアップガイド
```

### データベース設計

#### 主要テーブル

1. **users** - ユーザー管理
   - id, username, password_hash, created_at

2. **lost_item** - 拾得物基本情報
   - 基本情報: id, main_id, current_year, choice_finder, track_num
   - 日時情報: get_item, recep_item, 各種時刻フィールド
   - 拾得者情報: finder_name, finder_age, finder_sex, finder_address等
   - 物品情報: item_class_L/M/S, item_feature, item_color等
   - 状況管理: item_situation, refund_situation
   - カード情報: card_campany, card_tel等
   - 遺失者情報: lost_person, lost_address等
   - 返還情報: return_date, return_person等
   - 警察・還付情報: police_date, refund_date等

3. **bundled_item** - 同梱物情報
   - lost_itemと同様の物品情報構造
   - lostitem_idで主物品と関連付け

4. **denomination** - 金種情報（現金の場合）
   - 各種紙幣・硬貨の枚数
   - 記念硬貨情報
   - lostitem_idで関連付け

5. **notfound** - 遺失物情報
   - 紛失物の届出情報を管理

## 主要機能

### 1. 認証システム (auth)
- **ユーザー登録**: 新規ユーザーアカウント作成
- **ログイン/ログアウト**: セッション管理
- **パスワード暗号化**: Werkzeugによる安全な暗号化

### 2. 拾得物登録システム (register)
- **写真撮影**: Webカメラによる物品撮影
- **AI画像認識**: 
  - AWS Lambda経由での画像分析
  - CoCa (Contrastive Captioner) モデルによるキャプション生成
  - VGG16モデルによる物品分類
- **拾得者分類**: 占有者拾得 vs 第三者拾得
- **詳細情報入力**: 物品特徴、拾得場所、拾得者情報等
- **階層分類**: 大分類→中分類→小分類の3段階物品分類

### 3. 拾得物管理システム (items)
- **一覧表示**: テーブル形式・写真形式での表示
- **検索・フィルタリング**: 
  - 日付範囲、物品特徴、拾得場所での絞り込み
  - 物品分類、色、貴重品フラグでの検索
- **詳細表示**: 拾得物の全情報表示
- **編集機能**: 登録済み情報の修正
- **削除機能**: 不要データの削除
- **ページネーション**: 大量データの効率的表示
- **自動削除**: 古いレコード・写真の自動削除機能

### 4. 警察届出システム (police)
- **届出対象選択**: 未届出物品の一覧表示・選択
- **PDF書類生成**: 
  - 占有者拾得物提出書
  - 第三者拾得物提出書
  - フレキシブルディスク提出票
- **状況更新**: 届出済みステータスの自動更新

### 5. 返還処理システム (return_item)
- **遺失者連絡**: 遺失者情報の登録・連絡記録
- **返還処理**: 返還手続きの記録
- **受領書生成**: PDF形式の受領確認書作成

### 6. 還付処理システム (refund)
- 還付手続きの管理
- 還付状況の追跡

### 7. 廃棄処理システム (disposal)
- 廃棄対象物品の管理
- 廃棄記録の保持

### 8. 同梱物管理システム (bundleditems)
- メイン物品に付随する同梱物の管理
- 個別の物品情報登録

### 9. 遺失物管理システム (notfound)
- 遺失物届出の受付・管理
- 拾得物とのマッチング支援

## AI機能詳細

### 画像認識システム
1. **AWS Lambda連携**
   - Base64エンコードされた画像をLambdaに送信
   - クラウドベースの高精度画像認識

2. **ローカルモデル（CoCa）**
   - Open-CLIPのCoCa_ViT-L-14モデル使用
   - 英語キャプション生成→日本語翻訳
   - GPU/CPU自動選択

3. **VGG16分類モデル**
   - 26カテゴリの物品分類
   - 事前学習済みモデルの活用

### 物品分類システム
- **大分類**: 26カテゴリ（現金、かばん類、電気製品類等）
- **中分類**: 各大分類の詳細分類
- **小分類**: さらに細かい分類
- **動的選択**: 上位分類に応じた下位選択肢の動的更新

## セットアップ手順

### 1. 環境構築
```bash
# 仮想環境作成
python -m venv venv
source venv/bin/activate  # Linux/Mac
# または
venv\Scripts\activate     # Windows

# 依存関係インストール
pip install -r requirements.txt
```

### 2. 設定ファイル作成
```bash
# config.pyの作成
cp apps/config.sample apps/config.py
# シークレットキー、CSRF秘密鍵、Lambda URLを設定
```

### 3. 環境変数設定
```bash
# .envファイル作成
echo 'FLASK_APP=apps.app:create_app("local")' > .env
echo 'FLASK_ENV=test' >> .env
```

### 4. データベース初期化
```bash
flask db init
flask db migrate
flask db upgrade
```

### 5. AIモデル配置
```bash
# CoCaモデルをregister/model_folderに配置
# model.pthとして保存
```

### 6. アプリケーション起動
```bash
flask run
```

## 運用上の注意点

### セキュリティ
- シークレットキーは20桁の英数字で設定
- パスワードはハッシュ化して保存
- CSRF保護が有効

### データ管理
- 画像ファイルは自動的にリネーム・移動
- 古いデータは自動削除（5年経過後）
- 写真は1年後に自動削除

### パフォーマンス
- ページネーション実装（50件/ページ）
- セッションベースの検索条件保持
- 効率的なデータベースクエリ

## 拡張可能性

### AI機能強化
- より高精度な画像認識モデルの導入
- 多言語対応の強化
- リアルタイム画像解析

### 機能追加
- モバイルアプリ対応
- API提供
- 統計・分析機能
- 通知システム

### インフラ強化
- クラウドデプロイメント
- 負荷分散
- バックアップシステム

## まとめ

lostitem_appは、従来の手作業による拾得物管理を大幅に効率化する現代的なWebアプリケーションです。AI技術の活用により、物品の分類・記録作業を自動化し、管理者の負担を軽減します。また、包括的な機能により、拾得から返還・廃棄まで一連の業務フローを統合的に管理できます。

システムの設計は拡張性を重視しており、将来的な機能追加や技術アップデートに対応可能な構造となっています。
