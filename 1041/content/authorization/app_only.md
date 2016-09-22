# サービスまたはデーモン アプリで Microsoft Graph を呼び出す

この記事では、シングル テナントのサービスまたはデーモン アプリを Office 365 に接続し、Microsoft Graph API を呼び出すために最小限必要なタスクについて説明します。

## 概要

サービスまたはデーモン アプリで Microsoft Graph API を呼び出すには、次のタスクを完了する必要があります。

1. Azure Active Directory にアプリケーションを登録する。
2. トークンを発行するエンドポイントからアクセス トークンを要求する。
3. Microsoft Graph API への要求にアクセス トークンを使用する。

## Azure Active Directory にアプリケーションを登録する

Office 365 を使用する前に、自分のアプリケーションを登録し、Microsoft Graph のサービスを使用するアクセス許可を設定する必要があります。[アプリケーション登録ツール](https://dev.office.com/app-registration)を使用すれば、数回クリックするだけでアプリケーションを登録することができます。
これを管理するには、[Microsoft Azure 管理ポータル](https://manage.windowsazure.com)に移動する必要があります。

あるいは、「[Azure 管理ポータルに Web サーバー アプリを登録する](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp)」のセクションを参照して、手動でアプリを登録する手順をご確認ください。以下の詳細な点にご注意ください。

* アプリケーションを登録した後、サービスまたはデーモン アプリで必要な**アプリケーションのアクセス許可**を設定します。

Azure アプリケーションの [構成] ページにある以下の値をメモしてください。これらの値は、サービスまたはデーモン アプリの OAuth フローを構成する際に必要になります。

* クライアント ID (あなたのアプリケーションに固有)
* アプリケーション キー (あなたのアプリケーションに固有)
* アプリの OAuth 2.0 のトークン エンドポイント
  * この値は、アプリのページの Azure 管理ポータルの下部にある *[エンドポイントの表示]* をクリックすると表示されます。 エンドポイントは `https://login.microsoftonline.com/<tenantId>/oauth2/token` のようになります。

## トークンを発行するエンドポイントからアクセス トークンを要求する

クライアント アプリとは異なり、サービスやデーモン アプリでは、ユーザーにサインインしてあなたのアプリケーションを承認してもらうことはできません。代わりに、アプリケーションに OAuth 2.0 クライアント資格情報付与フローが実装されている必要があります。これは、ユーザーを偽装するのではなく、Microsoft Graph を呼び出すときに独自の資格情報、そのクライアント ID、およびアプリケーション キーを使用して認証を行います。認証フローの詳細については、「[クライアント資格情報を使用したサービス間の呼び出し](https://msdn.microsoft.com/en-us/library/azure/dn645543.aspx)」を参照してください。

以下のパラメーターを使用してエンドポイントを発行するトークンに HTTP POST 要求を行います。その際、`<clientId>` と `<clientSecret>` をそれぞれ、自分のアプリのクライアント ID とアプリケーション キーに置き換えます。

```http
POST https://login.microsoftonline.com/<tenantId>/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=<clientId>
&client_secret=<clientSecret>
&resource=https://graph.microsoft.com
```

応答には、アクセス トークンと有効期限の情報が含まれます。

```json
{ 
  "token_type": "Bearer",
  "expires_in": "3599",
  "scope": "User.Read",
  "expires_on": "1449685363",
  "not_before": "1449681463",
  "resource": "https://graph.microsoft.com",
  "access_token": "<token>"
}
```

## Microsoft Graph API への要求にアクセス トークンを使用する

アクセス トークンを使用することにより、あなたのアプリは Microsoft Graph API へ認証された要求を行うことができます。アプリで各要求の **Authorization** ヘッダーにアクセス トークンを追加する必要があります。

たとえば、Azure 管理ポータルで*すべてのユーザーの完全なプロファイルの読み取り*アクセス許可が選択されている場合、サービスまたはデーモン アプリはテナント内のすべてのユーザーを取得することができます。 

```http
GET https://graph.microsoft.com/v1.0/users
Authorization: Bearer <token>
```

Microsoft Graph は、あらゆる種類の Microsoft データとの対話に使用できる、非常に強力な統合 API です。[API リファレンス](http://graph.microsoft.io/docs/api-reference/v1.0)をご覧になり、Microsoft Graph API でほかに何を行うことができるかを調べてください。
