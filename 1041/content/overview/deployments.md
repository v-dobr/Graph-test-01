# 独立したクラウドの展開


この記事では、Microsoft Graph と、開発者が利用できる機能の、独立したクラウド インスタンスに関する情報を提供します。 


## 中国の 21Vianet によって運営されている Microsoft Graph

このセクションでは、21Vianet によって運営されている Microsoft Graph と、開発者が利用できる機能に関する情報を提供します。

### サービスのルート エンドポイント
| 21Vianet によって運営されている Microsoft Graph | Microsoft Graph|
|---------------------------|----------------|
| https://microsoftgraph.chinacloudapi.cn | https://graph.microsoft.com|

### Microsoft Graph Explorer
| 21Vianet によって運営されている Microsoft Graph | Microsoft Graph|
|---------------------------|----------------|
|https://graphexplorerchina.azurewebsites.net| https://graphexplorer2.azurewebsites.net|

### Azure OpenID Connect と OAuth 2.0
21Vianet によって運営されている Microsoft Graph へのサインインや呼び出しのためのトークンを取得するのに使用されるエンドポイントは、他の製品のエンドポイントとは異なります。 

| 21Vianet によって運営されている Microsoft Graph | Microsoft Graph|
|---------------------------|----------------|
| https://login.chinacloudapi.cn | https://login.microsoftonline.com|
 
ユーザーを認証するには https://login.chinacloudapi.cn/common/oauth2/authorize を、21Vianet によって運営されている Microsoft Graph をアプリが呼び出すためのトークンを取得するには https://login.chinacloudapi.cn/common/oauth2/token を使用します。

> **注**:最新の[バージョン 2.0 およびトークン エンドポイントの認証](https://azure.microsoft.com/en-us/documentation/articles/active-directory-appmodel-v2-overview/)は 21Vianet によって運営されている Microsoft Graph で使用することはできません。アプリは、コンシューマー データではなく組織のデータにのみアクセスできます。 

### 21Vianet によって運営されている Microsoft Graph が提供するサービス機能
次の Microsoft Graph の機能が一般提供 (`/v1.0` エンドポイントにて) されています。

* ユーザー
* グループ
* ファイル
* メール
* 予定表
* 個人用連絡先 
* 作成、読み取り、更新、削除 (CRUD) 操作
* CORS (クロスオリジン リソース共有) サポート。

次の Microsoft Graph の機能はプレビューでも公開 (`/beta` エンドポイントにて) されています。

* 組織の連絡先
* アプリケーション
* サービス プリンシパル
* ディレクトリ スキーマの拡張
