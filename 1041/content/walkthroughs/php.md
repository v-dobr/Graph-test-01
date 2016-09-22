# PHP アプリで Microsoft Graph を呼び出す 

ここでは、Azure Active Directory (AD) からアクセス トークンを取得して、Microsoft Graph API を呼び出すために必要な最低限のタスクに着目します。[Microsoft Graph を使用する Office 365 PHP 接続サンプル](https://github.com/microsoftgraph/php-connect-rest-sample)のコードを使用して、アプリに実装する必要のある主要な概念を説明します。

![Office 365 PHP Connect サンプルのスクリーンショット](./images/web-screenshot.png)

## 概要

Microsoft Graph API を呼び出すには、PHP アプリで次のタスクを行う必要があります。

1. Azure Active Directory にアプリケーションを登録する
2. ブラウザーをサインイン ページにリダイレクトする
3. 応答 URL ページで認証コードを受け取る
4. トークン エンドポイントからアクセス トークンを要求する
5. Microsoft Graph API への要求にアクセス トークンを使用する

<!--<a name="register"/>-->
## Azure Active Directory にアプリケーションを登録する

Office 365 を使用する前に、アプリケーションを登録し、Microsoft Graph のサービスを使用するアクセス許可を設定する必要があります。数回クリックするだけで、[アプリケーションの登録ツール](https://dev.office.com/app-registration)を使用して、アプリケーションを登録しユーザーの職場または学校のアカウントにアクセスすることができます。
これを管理するには、[Microsoft Azure の管理ポータル](https://manage.windowsazure.com)に移動する必要があります

あるいは、「[Azure 管理ポータルに Web サーバー アプリを登録する](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp)」のセクションを参照して、手動でアプリを登録する手順をご確認ください。以下の詳細な点にご注意ください。

* 手順 6 の**サインオン URL** として PHP アプリでページを指定します。接続サンプルの場合、このページは [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php) です。
* アプリに必要な[**委任されたアクセス許可**を構成](https://github.com/microsoftgraph/php-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure)します。接続サンプルには、**サインインしているユーザーとしてメールを送信する**アクセス許可が必要です。

Azure アプリケーションの **[構成]** ページの次の値をメモしてください。

* クライアント ID
* 有効なキー
* 応答 URL

アプリ内の OAuth フローのパラメーターとしてこれらの値が必要です。

<!--<a name="redirect"/>-->
## ブラウザーをサインイン ページにリダイレクトする

アプリはブラウザーをサインイン ページにリダイレクトして認証コードを取得し、OAuth のフローを続行する必要があります。

接続サンプルでは、ブラウザーをリダイレクトするコードは [`AuthenticationManager.connect`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L41) 関数内にあります。

```php
// Redirect the browser to the authorization endpoint. Auth endpoint is
// https://login.microsoftonline.com/common/oauth2/authorize
$redirect = Constants::AUTHORITY_URL . Constants::AUTHORIZE_ENDPOINT . 
            '?response_type=code' . 
            '&client_id=' . urlencode(Constants::CLIENT_ID) . 
            '&redirect_uri=' . urlencode(Constants::REDIRECT_URI);
header("Location: {$redirect}");
exit();
```

> **注:** <br />
> 出力をページに書き込む前に、**Location** ヘッダーを送信する必要があります。

<!--<a name="authcode"/>-->
## 応答 URL ページで認証コードを受け取る

ユーザーがサインインすると、フローはアプリの応答 URL にブラウザーを返します。Azure では、クエリ文字列に認証コードを追加します。接続サンプルでは、このために [`Callback.php`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/app/callback.php) ページを使用します。

認証コードは `code` クエリ文字列変数で提供されます。接続サンプルでは、後で使用するため接続セッション変数へのコードが保存されます。

```php
if (isset($_GET['code'])) {
    $_SESSION['code'] =  $_GET['code'];
}
```

<!--<a name="accesstoken"/>-->
## トークン エンドポイントからアクセス トークンを要求する

認証コードを作成したら、Azure AD から取得したクライアント ID、キー、応答 URL の値に沿ってこれを使用して、アクセス トークンを要求できます。 

> **注:** <br />
> 要求には、使用する必要のあるリソースも指定してください。Microsoft Graph API の場合、リソースの値は `https://graph.microsoft.com` です。

接続サンプルでは、[`AuthenticationManager.acquireToken`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/AuthenticationManager.php#L62) 関数のコードを使用してトークンを要求します。ここでは最も関連するコードを示します。

```php
$tokenEndpoint = Constants::AUTHORITY_URL . Constants::TOKEN_ENDPOINT;

// Send a POST request to the token endpoint to retrieve tokens.
// Token endpoint is:
// https://login.microsoftonline.com/common/oauth2/token
$response = RequestManager::sendPostRequest(
    $tokenEndpoint, 
    array(),
    array(
        'client_id' => Constants::CLIENT_ID,
        'client_secret' => Constants::CLIENT_SECRET,
        'code' => $_SESSION['code'],
        'grant_type' => 'authorization_code',
        'redirect_uri' => Constants::REDIRECT_URI,
        'resource' => Constants::RESOURCE_ID
    )

// Store the raw response in JSON format.
$jsonResponse = json_decode($response, true);

// The access token response has the following parameters:
// access_token - The requested access token.
// expires_in - How long the access token is valid.
// expires_on - The time when the access token expires.
// id_token - An unsigned JSON Web Token (JWT).
// refresh_token - An OAuth 2.0 refresh token.
// resource - The App ID URI of the web API (secured resource).
// scope - Impersonation permissions granted to the client application.
// token_type - Indicates the token type value.
foreach ($jsonResponse as $key=>$value) {
    $_SESSION[$key] = $value;
}
```

> **注:** <br />
> 応答では、アクセス トークン以外の情報も提供されます。たとえば、アプリはユーザーが明示的にサインインしなくても新しいアクセス トークンを要求できる更新トークンを取得できます。

PHP アプリでは、セッション変数 `access_token` を使用して、認証された要求を Microsoft Graph API に発行できるようになりました。

<!--<a name="request"/>-->
## Microsoft Graph API への要求にアクセス トークンを使用する

アクセス トークンを使用することにより、ご自分のアプリで Microsoft Graph API へ認証された要求を行うことができます。アプリの各要求の **Authorization** ヘッダーで、アクセス トークンを指定する必要があります。

接続サンプルでは、Microsoft Graph API で **sendMail** エンドポイントを使用してメールを送信します。このコードは [`MailManager.sendWelcomeMail`](https://github.com/microsoftgraph/php-connect-rest-sample/blob/master/src/MailManager.php#L40) 関数内にあります。これは、Authorization ヘッダーに入れてアクセス コードを送信する方法を示すコードです。

```php
// Send the email request to the sendmail endpoint, 
// which is in the following URI:
// https://graph.microsoft.com/v1.0/me/microsoft.graph.sendMail
// Note that the access token is attached in the Authorization header
RequestManager::sendPostRequest(
    Constants::RESOURCE_ID . Constants::SENDMAIL_ENDPOINT,
    array(
        'Authorization: Bearer ' . $_SESSION['access_token'],
        'Content-Type: application/json;' . 
                      'odata.metadata=minimal;' .
                      'odata.streaming=true'
    ),
    $email
);
```

> **注:** <br />
> 要求は **Content-Type** ヘッダーに `application/json;odata.metadata=minimal;odata.streaming=true` など、Microsoft Graph API に受け入れられる値を指定して送信します。

Microsoft Graph API は、あらゆる種類の Microsoft データとの対話に使用できる、非常に強力な統合 API です。API リファレンスを参照し、Microsoft Graph API で何を行うことができるかを調べてください。

<!--## Additional resources

-  [Office 365 PHP Connect sample using Microsoft Graph API](https://github.com/OfficeDev/O365-PHP-Unified-API-Connect)-->
