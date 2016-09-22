# Node.js アプリで Microsoft Graph を呼び出す

この記事では、アプリケーションを Office 365 に接続し、Microsoft Graph AP を呼び出すために必要な最低限のタスクに着目します。[Microsoft Graph を使用する Office 365 Node.js 接続サンプル](https://github.com/microsoftgraph/nodejs-connect-rest-sample)のコードを使用して、アプリに実装する必要のある主要な概念を説明します。

![Office 365 Node.js Connect サンプルのスクリーンショット](./images/web-screenshot.png)

## 概要

Microsoft Graph API を呼び出すには、Web アプリで次のタスクを行う必要があります。

1. Azure Active Directory にアプリケーションを登録する 
2. ノード用の Azure Active Directory クライアント ライブラリをインストールする
3. ブラウザーをサインイン ページにリダイレクトする
4. 応答 URL ページで認証コードを受け取る
5. `adal-node` を使用してアクセス トークンを要求する
6. Microsoft Graph API に対して要求を行う

<!--<a name="register"/>-->
## Azure Active Directory にアプリケーションを登録する

Office 365 を使用する前に、アプリケーションを登録し、Microsoft Graph のサービスを使用するアクセス許可を設定する必要があります。数回クリックするだけで、[アプリケーションの登録ツール](https://dev.office.com/app-registration)を使用して、アプリケーションを登録しユーザーの職場または学校のアカウントにアクセスすることができます。
これを管理するには、[Microsoft Azure の管理ポータル](https://manage.windowsazure.com)に移動する必要があります

あるいは、「[Azure 管理ポータルに Web サーバー アプリを登録する](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp)」のセクションを参照して、手動でアプリを登録する手順をご確認ください。以下の詳細な点にご注意ください。

* 手順 6 の**サインオン URL** として Node.js アプリでページを指定します。接続サンプルでは、URL はhttp://localhost:8080/login です。これは [/login](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L33) ルートに対応しています。
* [アプリに必要な**委任されたアクセス許可**](https://github.com/microsoftgraph/nodejs-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure)を構成します。 接続サンプルには、**サインインしているユーザーとしてメールを送信する**アクセス許可が必要です。

Azure アプリケーションの **[構成]** ページの次の値をメモしてください。

* クライアント ID
* 有効なキー
* 応答 URL

アプリ内の OAuth フローのパラメーターとしてこれらの値が必要です。

<!--<a name="adal">-->
## ノード用の Azure Active Directory クライアント ライブラリをインストールする

Node.js ライブラリの ADAL は、AAD によって保護されている web リソースにアクセスするための Node.js アプリによる AAD への認証を簡単にします。
既存の `package.json` に adal-node を追加するには、優先ターミナルに次のように入力します。

`npm install adal-node --save`

adal-node のクライアント ライブラリの詳細については、[npm](https://www.npmjs.com/package/adal-node) のパッケージ情報を参照してください。問題、ソース コード、および今後の機能と修正の最新情報については、
[Github](https://github.com/AzureAD/azure-activedirectory-library-for-nodejs) の adal-node のプロジェクトを参照してください。

<!--<a name="redirect"/>-->
## ブラウザーをサインイン ページにリダイレクトする

アプリはブラウザーをサインイン ページにリダイレクトして認証コードを取得し、OAuth 2.0 のフローを続行する必要があります。

接続サンプルでは、[`authHelper.js#getAuthUrl`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js#L17) からの認証 URL はクライアント側の `onclick` イベント経由で [`login.hbs#login`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/views/login.hbs#L2) 関数によってリダイレクトされます。

**authHelper.js#getAuthUrl**
```javascript
/**
 * Generate a fully formed uri to use for authentication based on the supplied resource argument
 * @return {string} a fully formed uri with which authentcation can be completed
 */
function getAuthUrl() {
    return credentials.authority + "/oauth2/authorize" +
        "?client_id=" + credentials.client_id +
        "&response_type=code" +
        "&redirect_uri=" + credentials.redirect_uri;
};
```

**login.hbs#login**
```javascript
function login() {
    window.location = '{{auth_url}}'.replace(/&amp;/g, '&'); // transform HTML special char from .hbs template rendering
}
```

<!--<a name="authcode"/>-->
## 応答 URL ページで認証コードを受け取る

ユーザーがサインインすると、フローはアプリ内の応答 URL にブラウザーを返します。 認証コードは `code` クエリ文字列変数で提供されます。

```javascript
router.get('/<application reply url>', function (req, res, next) {
  var authCode = req.query.code;
  // your router's implementation
});
```

接続サンプルの[関連コード](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/routes/index.js#L34)をご覧ください

<!--<a name="accesstoken"/>-->
## `adal-node` を使用してアクセス トークンを要求する

これで、Azure Active Directory で認証されました。次の手順では、adal-node を介してアクセス トークンを取得します。これを完了すると、Microsoft Graph API への REST 要求を行う準備ができます。

アクセス トークンを要求するために、adal-node は 2 つのコールバック関数を提供します。

|                          関数                         |                                      パラメーター                                      | 説明                                                                                             |
|:---------------------------------------------------------:|:--------------------------------------------------------------------------------:|---------------------------------------------------------------------------------------------------------|
| `AuthenticationContext.acquireTokenWithAuthorizationCode` | `authCode`, `redirect_uri`, `resource`, `client_id`, `client_secret`, `callback` | ログイン時に返される認証コードに基づいて、指定したリソースのアクセス トークンを提供する |
| `AuthenticationContext.acquireTokenWithRefreshToken`      | `token`, `client_id`, `client_secret`, `resource`, `callback`                    | 更新トークンに基づいて、指定したリソースのアクセス トークンを提供する                             |

接続サンプルでは、`client_id` と `client_secret` を追加できるように、要求は [`authHelper.js`](https://github.com/microsoftgraph/nodejs-connect-rest-sample/blob/master/authHelper.js) を経由してルーティングされます。

```javascript
// The application registration (must match Azure AD config)
var credentials = {
    authority: "https://login.microsoftonline.com/common",
    client_id: "<your client id here>",
    client_secret: "<your client secret>",
    redirect_uri: "http://localhost:8080/login"
};

/**
 * Gets a token for a given resource.
 * @param {string} code An authorization code returned from a client.
 * @param {string} res A URI that identifies the resource for which the token is valid.
 * @param {AcquireTokenCallback} callback The callback function.
 */
function getTokenFromCode(res, code, callback) {
    var authContext = new AuthenticationContext(credentials.authority);
    authContext.acquireTokenWithAuthorizationCode(code, credentials.redirect_uri, res, credentials.client_id, credentials.client_secret, function (err, response) {
        if (err) {
            callback(null);
        }
        else {
            callback(response);
        }
    });
};
```

<!--<a name="request"/>-->
## Microsoft Graph API に対して要求を行う

Graph API への要求を識別するには、要求する任意の Web サービス リソースのアクセス トークンを含む `Authorization` ヘッダーで、要求に署名する必要があります。 適切な形式の承認ヘッダーには adal-node からのアクセス トークンが含まれ、次の形式になります。

`Authorization: Bearer <access token>`

前述のセクションの認証ロジックを `adal-node` と組み合わせて使用して、アクセス トークンで要求に署名できるようになりました。

```javascript
/* GET home page. */
router.get('/<application reply url>', function (req, res, next) {
    var authCode = req.query.code;
    authHelper.getTokenFromCode('https://graph.microsoft.com/', req.query.code, function (token) {
        if (token !== null) {
            // Use this token to sign requests
            var headers = {
                'Content-Type': 'application/json',
                'Authorization': 'Bearer ' + token
                };
            // request implementation...
        } else {
            // error handling
        }
    });
});
```

Microsoft Graph は、あらゆる種類の Microsoft データとの対話に使用できる、非常に強力な統合 API です。[API リファレンス](http://graph.microsoft.io/docs/api-reference/v1.0)をご覧になり、Microsoft Graph API でほかに何を行うことができるかを調べてください。

<!--## Additional resources

- [Office 365 Node.js Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Nodejs-Unified-API-Connect)-->

