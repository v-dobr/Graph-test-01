# PHP アプリで Microsoft Graph を使ってみる

この記事では、v2.0 認証エンドポイントからアクセス トークンを取得し、Microsoft Graph を呼び出すために必要なタスクについて説明します。 ここでは、[PHP 用接続サンプル](https://github.com/microsoftgraph/php-connect-rest-sample)のビルドの手順と、Microsoft Graph を使用するために実装する主要な概念について説明します。 また、REST 呼び出しを使用して Microsoft Graph にアクセスする方法についても説明します。

PHP アプリで Microsoft Graph を使用するには、ユーザーに Microsoft のサインイン ページを表示する必要があります。 次のスクリーン ショットは、Microsoft アカウントのサインイン ページを示しています。

![Microsoft アカウントのサインイン ページ](images/MicrosoftSignIn.png)

**アプリを構築するのは難しいと感じていますか?** この記事で取り扱っている [PHP 用接続サンプル](https://github.com/microsoftgraph/php-connect-rest-sample)をダウンロードすれば、簡単に開始できます。


## 前提条件

開始するには、次のものが必要です。 

- [Microsoft アカウント](https://www.outlook.com/)または [Office 365 for Business アカウント](http://dev.office.com/devprogram)
- PHP バージョン 5.5.9 以降
- [Composer](https://getcomposer.org/)


## アプリケーションの登録
Microsoft アプリケーション登録ポータルでアプリケーションを登録します。 これにより、アプリの構成に使用するアプリ ID とパスワードが生成されます。

1. 個人用アカウント、あるいは職場または学校アカウントのいずれかを使用して、[Microsoft アプリ登録ポータル](https://apps.dev.microsoft.com/)にサインインします。

2. **[アプリの追加]** を選択します。

3. アプリの名前を入力して、**[アプリケーションの作成]** を選択します。 
    
   登録ページが表示され、アプリのプロパティが一覧表示されます。

4. **[新しいパスワードを生成]** を選択します。

5. アプリケーション ID とパスワードをコピーします。

6. **[プラットフォームの追加]** および **[Web]** を選択します。

7. **[リダイレクト URI]** フィールドに `http://localhost:8000/oauth` と入力します。

8. **[保存]** を選択します。


## プロジェクトを構成する

Composer を使用して、新しいプロジェクトを開始します。 Laravel フレームワークを使用して新しい PHP プロジェクトを作成するには、次のコマンドを入力します。

```bash
composer create-project --prefer-dist laravel/laravel getstarted
```
 
これにより、このプロジェクトに使用できる **getstarted** フォルダーが作成されます。

## ユーザーの認証とアクセス トークンの取得
OAuth ライブラリを使用して、認証プロセスを簡素化します。 [PHP League](http://thephpleague.com/) により、このプロジェクトで使用できる [OAuth クライアント ライブラリ](https://github.com/thephpleague/oauth2-client) が作成されます。

### Composer に依存関係を追加する

`composer.json` ファイルを開き、次の依存関係を**必須**セクションに組み込みます。

```json
"league/oauth2-client": "^1.4"
```

次のコマンドを実行して、依存関係を更新します。

```bash
composer update
```

### 認証フローの開始

1. **resources** > **views** > **welcome.blade.php** ファイルを開きます。 **title** div 要素を次のコードに置き換えます。
    ```html
    <div class="title" onClick="window.location='/oauth'">Sign in to Microsoft</div>
    ```
    
2. **app** > **Http** > **routes.php** ファイルで `Illuminate\Http\Request` クラスの型ヒントを記述します。 すべてのルート宣言の前に、次の行を追加します。
    ```php
    use Illuminate\Http\Request;
    ```
    
3. */oauth* ルートを **app** > **Http** > **routes.php** ファイルに追加します。 ルートを追加するには、以下のコードを既定のルート宣言の後に追加します。 アプリの **アプリケーション ID** および**パスワード**を、**\<YOUR_APPLICATION_ID\>** および **\<YOUR_PASSWORD\>** とマークされているプレースホルダーにそれぞれ挿入します。
    ```php
    Route::get('/oauth', function () {
        $provider = new \League\OAuth2\Client\Provider\GenericProvider([
            'clientId'                => '<YOUR_APPLICATION_ID>',
            'clientSecret'            => '<YOUR_PASSWORD>',
            'redirectUri'             => 'http://localhost:8000/oauth',
            'urlAuthorize'            => 'https://login.microsoftonline.com/common/oauth2/v2.0/authorize',
            'urlAccessToken'          => 'https://login.microsoftonline.com/common/oauth2/v2.0/token',
            'urlResourceOwnerDetails' => '',
            'scopes'                  => 'openid mail.send'
        ]);

        if (!$request->has('code')) {
            return redirect($provider->getAuthorizationUrl());
        }
    });
    ```
    
この時点で、*"Microsoft にサインイン"* と表示している PHP アプリができたはずです。 テキストをクリックすると、アプリに Microsoft のサインイン ページが表示されます。 次の手順は、認証サーバーがリダイレクト URI にコードを送信してアクセス トークンと交換する作業です。

### 認証コードとアクセス トークンの交換

アクセス トークンと交換できるコードが含まれている認証サーバーの応答を処理できるようにする必要があります。

*/oauth* ルートを更新して、認証コードでアクセス トークンを取得できるようにします。 これを行うには、**app** > **Http** > **routes.php** ファイルを開き、以下の *else* 条件付き句を既存の *if* ステートメントに追加します。

```php
if (!$request->has('code')) {
    ...
    // add the following lines
} else {
    $accessToken = $provider->getAccessToken('authorization_code', [
        'code'     => $request->input('code')
    ]);
    exit($accessToken->getToken());
}
```
    
この行 `exit($accessToken->getToken());` にアクセス トークンがあることにご注意ください。 これで、Microsoft Graph を呼び出すためのコードを追加する準備が整いました。 

## REST を使用して Microsoft Graph を呼び出す
REST を使用して Microsoft Graph を呼び出すことができます。 [Microsoft Graph REST API](http://graph.microsoft.io/docs) は 1 つの REST API エンドポイントを介して複数の API を Microsoft クラウド サービスから公開するものです。 REST API を使用するには、行 `exit($accessToken->getToken());` を次のコードに置き換えます。 **\<YOUR_EMAIL_ADDRESS\>** とマークされているプレースホルダーに電子メール アドレスを挿入します。

```php
$client = new \GuzzleHttp\Client();

$email = "{
    Message: {
    Subject: 'Sent using the Microsoft Graph REST API',
    Body: {
        ContentType: 'text',
        Content: 'This is the email body'
    },
    ToRecipients: [
        {
            EmailAddress: {
            Address: '<YOUR_EMAIL_ADDRESS>'
            }
        }
    ]
    }}";

$response = $client->request('POST', 'https://graph.microsoft.com/v1.0/me/sendmail', [
    'headers' => [
        'Authorization' => 'Bearer ' . $accessToken->getToken(),
        'Content-Type' => 'application/json;odata.metadata=minimal;odata.streaming=true'
    ],
    'body' => $email
]);
if($response.getStatusCode() === 201) {
    exit('Email sent, check your inbox');
} else {
    exit('There was an error sending the email. Status code: ' . $response.getStatusCode());
}
```

## アプリの実行
PHP アプリを試す準備ができました。

1. シェルで、次のコマンドを入力します。
    ```bash
    php artisan serve
    ```
    
2. Web ブラウザーで `http://localhost:8000` に移動します。
3. **[Microsoft にサインイン]** を選択します。
4. 個人用あるいは職場または学校アカウントでサインインし、要求されたアクセス許可を付与します。

["REST を使用して Microsoft Graph を呼び出す"](#call-the-microsoft-graph-using-rest) で構成した電子メール アドレスの受信トレイを確認します。 アプリへのサインインに使用したアカウントからのメールを受信しているはずです。

## 次の手順
- [Graph エクスプローラー](https://graph.microsoft.io/graph-explorer)を使用して REST API を試してみます。


## 関連項目
* [Microsoft Graph の概要](http://graph.microsoft.io/docs)
* [Azure AD v2.0 のプロトコル](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
* [Azure AD v2.0 のトークン](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
