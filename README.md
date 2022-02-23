# :sheep: 無料ブラウザアプリ Yomukatsu! :book:

「あなたの積読を解消します!」をスローガンに掲げた無料ブラウザアプリです。

"読書メンタルマップ術"という手法を用いて、ユーザーの書籍完読に向けたモチベーション維持をサポートします。

URL: https://yomukatsu.com/

![top](https://user-images.githubusercontent.com/56747224/110200437-6c7e8f00-7ea1-11eb-8178-b06009c82a25.png)

ゲストユーザー機能もありますので、気軽に利用してみてほしいです:thumbsup:

※ 推奨ブラウザは Chrome、Safari になります

### 解説記事( Qiita )

URL: https://qiita.com/ddpmntcpbr/items/739dbb992b5ffac3fc2f

開発の背景から解説しています。

### 開発者 Twitter

URL: https://twitter.com/ddpmntcpbr

ご用の方はこちらまで！

## クラウドアーキテクチャー

![image](https://user-images.githubusercontent.com/56747224/110200543-dbf47e80-7ea1-11eb-8c91-227bd310faa9.png)

### 技術スタック概略(詳細後述)

- **Backend:** Rails ( API mode / Rspec / rubocop) + Nginx ( upstream puma-socket )
- **Frontend:** React ( create-react-app / Redux / Material-UI / eslint&prettier)
- **Infra:** AWS ( ECS Fargate/ ECR / RDS / ALB / Route53 ), Netlify, Docker & docker-compose, Circle CI, Slack Incommig Webhook

## 実装機能一覧

### ユーザー利用機能

- Twitter アカウントを利用したユーザー登録(OAuth による SNS 認証)
- ゲストログイン機能
- Google Books API を用いた書籍検索機能
- Google Books、Amazon、楽天ブックスへのリンクボタン配置
- Twitter シェア機能
  - ハッシュタグ「#yomukatsu」付き Tweet
  - Twitter card 表示
- Twitter card 用にリサイズした書籍画像を AWS S3 へ保存・管理
- ローディング画面
- お問い合わせ機能(Slack Incomming Webhook)
- Route53 による独自ドメイン + SSL 化
- レスポンシブ対応

### 非ユーザー利用機能

- puma-socket 通信による Rails の Nginx 配信
- Netlify の Pre-rendering 機能活用による OGP 情報の保持（Twitter card 表示用）
- Docker による開発環境の完全コンテナ化
- CircleCI による自動 CI/CD パイプライン構築
  - CI: Rspec, rubocop, eslint&prettier
  - CD: AWS ECR ( Rails, Nginx )
- その他セキュリティ対策(XXS, CSPF 等)

## 技術スタック詳細

### Back-end: Rails + Nginx

#### 主要 gem

- `devise_token_auth`： API モードでの devise。トークン認証を簡単に実装
- `twitter_omniauth`：Twitter 認証を簡単に実装
- `active_model_serializer`： Rails API からのレスポンス JSON を制御
- `imageMagic`: 画像のリサイズを実行。特に、Twitter card 用に書籍画像をリサイズする際に使用
- `aws-fog/carrierwave`: リサイズした書籍画像を AWS S3 に保存
- `rspec`： デファクトスタンダードになっている Ruby テスト用フレームワーク
- `rubocop`： Ruby の静的コード解析

### Front-end: React

`creat-react-app` をベースに開発。ホスティングには `Netlify` を使用。

#### 主要ライブラリ等

- `Redux`: State の一元管理するフレームワーク。Redux 関連ファイルは、reducks パターン則って管理
- `react-helmet`: meta タグの挿入による OGP 情報の保持(Twitter card 用)
- `Material-UI` ： Google が提供する UI コンポーネントライブラリ。
- `eslint & prettier`: javascript に対する静的コード解析。eslint は create-react-app に標準搭載されているものをベースに少しプラグインを追加 & prettier はイチから導入

### Infra

#### `Docker/docker-compose`

開発環境は、全て Docker コンテナ内で完結。また、AWS ECS(Fargate)へのコンテナデプロイより、開発環境と本番環境の差異を最小化。

#### `AWS(Amazon Web Service)`

主に Backend( Rails+Nginx )のデプロイで使用。

※利用サービス

- `ECS (Fargate)`: コンテナ向けサーバーレスコンピューティングエンジン。この中に Rails と Nginx の Docker イメージを入れて稼働
- `ECR`: Rails と Nginx の Docker イメージを保存
- `RDS (MySQL)`: AWS が用意しているスケーラブルなデータベースエンジン
- `ALB`: 負荷分散を担うロードバランシングサービス
- `Route53`: サイトの独自ドメイン化に使用
- `ACM`: サイトの https 化に使用
- `S3`: 静的ホスティングサービス。書籍画像も保存・管理に使用

#### `Netlify`

フロントエンド(create-react-app)のホスティングで利用。Pre-rendering 機能を利用することで、通常 SPA となる create-react-app でも、OGP 情報の保持(動的な Twitter Card 情報の保持)が可能

#### `CircleCI`

自動 CI/CD パイプラインの構築に利用。

- Github へ push した時
- master ブランチへ merge した時

に走るように設定。

※自動化項目

- Rspec
- rubocop
- eslint&prettier
- AWS ECR への Image push (master ブランチへ merge した時のみ)
- AWS ECS のタスク&サービスの更新(master ブランチへ merge した時のみ)
