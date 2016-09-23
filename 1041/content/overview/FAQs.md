
# Microsoft Graph のよく寄せられる質問 (FAQ)

## Microsoft Graph API でサポートされるのはどんなプラットフォームですか?
<!--
Apps can use the Microsoft Graph API to perform create, read, update, and delete (CRUD) operations on data sources and entities, giving them seamless access to work data. 

**Ease of use--one endpoint, all Office 365 data under one roof**

You can use the API in four steps:
1.  Select your programming language and development environment.
2.  Build your app.
3.  Optionally, host your app in Microsoft Azure or any cloud platform you choose.
4.  Authenticate your users by using single sign-on with Azure AD.

As a developer you can use the API to create custom apps that access and interact with all the richness of enterprise and productivity data--users, groups, organizational contacts, files, folders, mail, calendar, insights and relationships--and build apps across all mobile, web, and desktop platforms. No matter your development platform or tools. Using a single service endpoint to access those entities and data. And a single authentication flow.  -->

以下の操作を実行できます。

<!--Just like in Office 365 APIs, Office 365 unified endpoint API  allows you to build apps using any development environment of your choice:  -->

- .NET、PHP、Java、Python、Ruby など、自分がよく知っている任意の開発環境を使用する
- 任意のプログラミング言語、開発プラットフォーム、ホスティング環境を使用する
- JavaScript、HTML5、Python、Ruby、PHP、ASP.NET など、任意の Web 言語を利用し、API にアクセスするアプリを作成する  
- Visual Studio、Eclipse、Android Studio、Xcode など、自分で選んだ IDE を使用する
- Microsoft Azure や任意のクラウド プラットフォームにアプリをホストさせる
- Windows Universal、iOS、Android 用に、または別のデバイス プラットフォームでアプリを開発する
- Office アドイン (以前の Office 用アプリ)、または SharePoint アドイン (以前の SharePoint 用アプリ) から API を呼び出す
 


## Microsoft Graph API を使用する理由は?

プログラミングにより、あるユーザーのファイルを取得して写真をプロファイリングしたり、組織内でそのファイルを最後に変更した人のマネージャーを見つけたりできます。情報は複数のサービス (Azure Active Directory、SharePoint、Exchange) に保存されているため、この作業は Office 365 API を使用して複数の手順で行われます。 

1. 探索サービスを利用し、さまざまなサービス エンドポイントを見つけます 
2. Office 365 アプリが接続を求めるサービスの URL を決定します
3. 次に、各サービスのアクセス トークンを取得して管理し、サービスに直接要求します

これで、Microsoft Graph API を利用し、1 つの REST API エンドポイント経由で同じ複雑な操作を実行できます。サービスごとに異なるエンドポイントを見つけてナビゲートし、サービス別のアクセス トークンを取得して管理し、サイロになっているサービスやさまざまなデータ モデルを処理する必要がありません。

##サンプル クエリ

次の例は、異種サービス エンドポイントを利用して Office 365 API を操作するための現行モデルと Microsoft Graph API でそのモデルがどのくらい簡単になるかを示したものです。

**異種サービス エンドポイント**

|   **Operation**                  |  **API**                          |  **サービス エンドポイント** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| Office 365 API のサービス エンドポイントを検出する               |     `Discovery Service`           | _https://_**api.office.com**_/discovery/v1.0/me/services_ |
| ユーザーを取得する           |     `Azure AD Graph API` | _https://_**graph.windows.net**_/contoso.com/users?api-version=2013-04-05_|
| 受信トレイからメッセージ コレクションを取得する       |     `Office 365 API`           | _https://_**outlook.office365.com**_/api/v1.0/me/messages_  |
| ジョーのファイルを取得する   |     `Office 365 API`  | _https://_**contoso-my.sharepoint.com**_/personal/joe_contoso_com/_api/v1.0/files_ |


Microsoft Graph API を利用すれば、最初にサービス ポイントを検出し、次にさまざまなエンドポイントをトラバースしてユーザーのファイルやメールなどを取得する必要がありません。唯一必要なことは 1 つの REST URL 名前空間 (_**graph.microsoft.com**_) につながることです。

**Microsoft Graph API**

|   **Operation**                  |  **API**                          |  **サービス エンドポイント** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| Office 365 API のサービス エンドポイントを検出する                |     `Microsoft Graph`           | 必要なし |
| ユーザーを取得する           |     `Microsoft Graph` | _https://_**graph.microsoft.com**_/v1.0/contoso.onmicrosoft.com/users_ |
| 受信トレイからメッセージ コレクションを取得する       |     `Microsoft Graph`           | _https://_**graph.microsoft.com**_/v1.0/me/messages_  |
| ジョーのファイルを取得する   |     `Microsoft Graph `  | _https://_**graph.microsoft.com**_/v1.0/me/drive/root/children_ |


## Microsoft Graph API を使用する利点は?

Microsoft Graph API には次のような利点があります。

**開発者は一貫性があり、無駄のない方法で Microsoft クラウド サービスを利用できる**

-   すべてのサービス エンドポイントで 1 つの名前空間を利用します。サービス エンドポイントを検出する必要がありません。
-   1 つのトークンですべてのリソースにアクセス
-   現在、サイロになっているサービスの間で統合された、直接的なナビゲーションが可能になります (たとえば、特定の文書を作成したユーザーの部門/管理チェーンを取得します)。
-   1 つの API セットを利用することだけを必要とします。つまり、複数のサービスに接続するために Microsoft Graph API だけを必要とします。
-   Office プラットフォーム全体で REST API とエンティティが統合され、拡張されます。 
-   エンティティ間のナビゲーション プロパティなど、エンティティ全体でプロパティの命名規則とスキームに一貫性があります。

**Office を利用し、仕事と生活がより生産的になる**

-   文脈に即したコンテンツ。たとえば、グループ、プロジェクト、チーム、傾向を基準に文書を検索できます。
-   文脈に即したユーザー関係。たとえば、グループ メンバーシップ、興味、スキル、専門知識を基準にユーザーを検索できます。組織図で関係を見ることもできます。

**任意のプラットフォームで任意のプログラミング言語を利用し、アプリを開発できる**

-   あらゆる開発者のための開発ツールとリソース。あらゆるプラットフォームと言語を使用して開発できます。 
-   オープン技術を使用し、すべてのプラットフォームのモバイルを開発できます。  
-   Exchange、SharePoint、Azure AD に関する特別な知識がなくても、Microsoft Graph API エンティティにアクセスできます。

<!---<a name="msg_v2auth"> </a>-->

## Microsoft Graph API は v2.0 アプリ認証エンドポイントをサポートしますか。

はい。詳細については、「[v2 認証エンドポイントを使用して Microsoft Graph のエンドポイントを認証する](http://graph.microsoft.io/docs/authorization/converged_auth)」を参照してください。



  > お客様からのフィードバックを重視しています。[Stack Overflow](http://stackoverflow.com/questions/tagged/office365)でご連絡いただけます。質問には [MicrosoftGraph] と [office365] でタグ付けしてください。








