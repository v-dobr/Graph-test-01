# Android アプリで Microsoft Graph を呼び出す

ここでは、Azure Active Directory (AD) からアクセス トークンを取得して、Microsoft Graph を呼び出すために必要な最低限のタスクについて説明します。[Microsoft Graph を使用する Office 365 Android 接続サンプル](https://github.com/microsoftgraph/android-java-connect-rest-sample)のコードを使用して、アプリに実装する必要のある主要な概念を説明します。

次の図は、ユーザーが Office 365 に接続した後に行われる、サンプル アプリのメール送信のアクティビティを示しています。

![Office 365 の Android 統合 API 接続のサンプルのスクリーンショット](./images/AndroidConnect.png)

## 概要

Microsoft Graph API を呼び出すには、[Office 365 Android 接続サンプル](https://github.com/microsoftgraph/android-java-connect-rest-sample)で次のタスクを行います。

1. Azure Active Directory のライブラリのメソッドを呼び出すことにより、ユーザーを承認してアクセス トークンを取得します。
2. Microsoft Graph API エンドポイントの REST 操作として、メール メッセージの要求を作成します。

<!--<a name="register"/>-->
## Azure Active Directory にアプリケーションを登録する

Office 365 を使用する前に、アプリケーションを登録し、Microsoft Graph のサービスを使用するアクセス許可を設定する必要があります。数回クリックするだけで、[アプリケーションの登録ツール](https://dev.office.com/app-registration)を使用して、アプリケーションを登録しユーザーの職場または学校のアカウントにアクセスすることができます。
これを管理するには、[Microsoft Azure の管理ポータル](https://manage.windowsazure.com)に移動する必要があります

または、手動でアプリを登録する手順については、記事**Office 365 API にアクセスできるようにアプリを手動で Azure AD に登録する**の [Azure 管理ポータルを使用してネイティブ アプリを登録する](https://msdn.microsoft.com/en-us/office/office365/howto/add-common-consent-manually)セクションを参照してください。以下の詳細な点に注意してください。

* アプリに必要な**委任されたアクセス許可**を構成します。接続サンプルには、**サインインしているユーザーとしてメールを送信する**アクセス許可が必要です。

Azure アプリケーションの **[構成]** ページの次の値をメモしてください。

* クライアント ID
* リダイレクト URL

アプリの認証コードを構成するにはこれらの値が必要です。

## 接続サンプルの Gradle の依存関係
サンプルには、build.gradle の次のコード スニペットに示すように、ライブラリの依存関係があります

```gradle
dependencies {
    compile fileTree(dir: 'libs', include: ['*.jar'])
    compile 'com.android.support:appcompat-v7:22.1.1'

    // Azure Active Directory Library
    compile 'com.microsoft.aad:adal:1.1.7'

    // Retrofit + custom HTTP
    compile 'com.squareup.okhttp:okhttp-urlconnection:2.0.0'
    compile 'com.squareup.okhttp:okhttp:2.0.0'
    compile 'com.squareup.retrofit:retrofit:1.9.0'
}

```
<!--<a name="authenticate"/>-->
## 接続サンプルでの認証
接続サンプルでは、Azure アプリの登録値とユーザーの ID を認証に使用します。接続サンプルでサポートされている認証の動作は 2 つあります。

* メッセージ表示認証。ユーザーの ID が Android デバイスに保存されているユーザー設定にキャッシュされていない場合に使用します
* サイレント認証。ユーザー ID がキャッシュされ、メッセージを表示する必要がない場合に使用します。

[AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) クラスは、キャッシュされたユーザー ID を検索し、使用する認証の動作を決定する `isConnected()` ヘルパー メソッドを提供します。


```java
    private boolean isConnected(){
        SharedPreferences settings = this
                .mContextActivity
                .getSharedPreferences(PREFERENCES_FILENAME, Context.MODE_PRIVATE);

        return settings.contains(USER_ID_VAR_NAME);
    }

```

どちらの動作でも、ADAL の認証フローには Azure の登録プロセスで取得したクライアント ID とリダイレクト URL が必要です。サンプルでは、ソース コードでこれらの文字列を保持し取得してから、認証マネージャー オブジェクトがユーザーを認証します。

[Constants.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/Constants.java) インターフェイスは、クライアント ID とリダイレクト URL の 2 つの文字列を公開します。

```java
interface Constants {
    String AUTHORITY_URL = "https://login.microsoftonline.com/common";
    // Update these two constants with the values for your application:
    String CLIENT_ID = "<Your client id here>";
    String REDIRECT_URI = "<Your redirect uri here>";
    String UNIFIED_API_ENDPOINT = "https://graph.microsoft.com/v1.0/";
    String UNIFIED_ENDPOINT_RESOURCE_ID = "https://graph.microsoft.com/";
}
```
### AuthenticationManager クラスを作成する
[AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) コンストラクターは引数を取りませんが、Graph エンドポイントの URL を使用して Constants.java ファイルからのクラス文字列フィールドを設定します。このリソース文字列は、認証の両方の動作に使用されます。

```java
    private AuthenticationManager() {
        mResourceId = Constants.UNIFIED_ENDPOINT_RESOURCE_ID;
    }
```

### メッセージ表示認証

[AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) クラスは、統一されたエンドポイント上での REST の呼び出しに使用されるアクセス トークンを取得する `authenticatePrompt()` メソッドを提供します。

ADAL ライブラリ `acquireToken()` メソッドは非同期です。メソッドの引数には、現在のアクティビティのコンテキストおよびリソースへの参照、クライアント ID、リダイレクト URL が含まれます。
現在のアクティビティの参照では、ADAL ライブラリにアクティビティで資格証明のチャレンジ ページを表示させます。認証が成功すると、ADAL ライブラリは `onSuccess()` コールバックを呼び出します。このコールバックでは、2 つのことが実行されます

* アクセス トークンを `mAccessToken` に格納します。メール メッセージを送信するために REST 呼び出しを行うとき、サンプルでは、このアクセス トークンを認証ヘッダーに配置します。
* ユーザーの ID を保存されているユーザー設定に格納します。


```java
    /**
     * Calls acquireToken to prompt the user for credentials.
     *
     * @param authenticationCallback The callback to notify when the processing is finished.
     */
    private void authenticatePrompt(final AuthenticationCallback<AuthenticationResult> authenticationCallback) {
        getAuthenticationContext().acquireToken(
                this.mContextActivity,
                this.mResourceId,
                Constants.CLIENT_ID,
                Constants.REDIRECT_URI,
                PromptBehavior.Always,
                new AuthenticationCallback<AuthenticationResult>() {
                    @Override
                    public void onSuccess(final AuthenticationResult authenticationResult) {
                        if (authenticationResult != null) {
                            if (authenticationResult.getStatus() == AuthenticationStatus.Succeeded) {
                                setUserId(authenticationResult.getUserInfo().getUserId());
                                mAccessToken = authenticationResult.getAccessToken();
                                authenticationCallback.onSuccess(authenticationResult);
                            } else {
                                // We need to make sure that there is no data stored with the failed auth
                                AuthenticationManager.getInstance().disconnect();
                                // This condition can happen if user signs in with an MSA account
                                // instead of an Office 365 account
                                authenticationCallback.onError(
                                        new AuthenticationException(
                                                ADALError.AUTH_FAILED,
                                                authenticationResult.getErrorDescription()));
                            }
                        } else {
                            // I could not authenticate the user silently,
                            // falling back to prompt the user for credentials.
                            authenticatePrompt(authenticationCallback);
                        }
                    }

                    @Override
                    public void onError(Exception e) {
                        // We need to make sure that there is no data stored with the failed auth
                        AuthenticationManager.getInstance().disconnect();
                        authenticationCallback.onError(e);
                    }
                }
        );
    }

```

###サイレント認証
[AuthenticationManager.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/AuthenticationManager.java) クラスは統一されたエンドポイント上での REST の呼出しに使用されるアクセス トークンを取得する `authenticateSilent()` メソッドを提供します。

ADAL ライブラリ `acquireTokenSilent()` メソッドは非同期です。Azure 登録クライアント ID とリソース ID だけでなく、共有の設定に格納されているユーザー ID も取ります。ヘルパー メソッドでは、`getUserId()` がストレージからユーザー ID を取得します。

認証が成功した場合、`onSuccess()` メソッドが呼び出されます。`onSuccess` はアクセス トークンを `mAccessToken` に格納します。
メール メッセージを送信するために REST 呼び出しを行うとき、サンプルでは、このアクセス トークンを認証ヘッダーに配置します。
```java
    /**
     * Calls acquireTokenSilent with the user id stored in shared preferences.
     * In case of an error, it falls back to {@link AuthenticationManager#authenticatePrompt(AuthenticationCallback)}.
     *
     * @param authenticationCallback The callback to notify when the processing is finished.
     */
    private void authenticateSilent(final AuthenticationCallback<AuthenticationResult> authenticationCallback) {
        getAuthenticationContext().acquireTokenSilent(
                this.mResourceId,
                Constants.CLIENT_ID,
                getUserId(),
                new AuthenticationCallback<AuthenticationResult>() {
                    @Override
                    public void onSuccess(final AuthenticationResult authenticationResult) {
                        if (authenticationResult != null) {
                            if (authenticationResult.getStatus() == AuthenticationStatus.Succeeded) {
                                mAccessToken = authenticationResult.getAccessToken();
                                authenticationCallback.onSuccess(authenticationResult);
                            } else {
                                authenticationCallback.onError(
                                        new Exception(authenticationResult.getErrorDescription()));

                            }
                        } else {
                            // I could not authenticate the user silently,
                            // falling back to prompt the user for credentials.
                            authenticatePrompt(authenticationCallback);
                        }
                    }

                    @Override
                    public void onError(Exception e) {
                        // I could not authenticate the user silently,
                        // falling back to prompt the user for credentials.
                        authenticatePrompt(authenticationCallback);
                    }
                }
        );
    }

```
<!--<a name="sendmail"/>-->
## Office 365 を使用してメール メッセージを送信する

Azure にユーザーがサインインすると、接続サンプルは、メール メッセージを送信するアクティビティをユーザーに表示します。接続サンプルでは、ユーザーが [メールの送信] ボタンをクリックしたときに [MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java) クラスを使用してメッセージを送信します。

### REST アダプター ヘルパー クラス
[RESTHelper.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/RESTHelper.java) クラスでは、サンプルのすべての REST 呼び出しに承認ヘッダーを挿入するためのメソッドを提供します。ここでは認証マネージャーによって提供されるアクセス トークンを使用します。

```java
       //This method catches outgoing REST calls and injects the Authorization and host headers before
        //sending to REST endpoint
        RequestInterceptor requestInterceptor = new RequestInterceptor() {
            @Override
            public void intercept(RequestFacade request) {
                final String token = mAccessToken;
                if (null != token) {
                    request.addHeader("Authorization", "Bearer " + token);
                }
            }
        };
```
### UnifiedAPIController クラス
[MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java) クラスは `sendMail()` メソッドで REST 要求を生成します。


```java
    /**
     * Sends an email message using the Unified API on Office 365. The mail is sent
     * from the address of the signed in user.
     *
     * @param emailAddress The recipient email address.
     * @param subject      The subject to use in the mail message.
     * @param body         The body of the message.
     * @param callback     UI callback to be invoked by Retrofit call when
     *                     operation completed
     */
    public void sendMail(
            final String emailAddress,
            final String subject,
            final String body,
            Callback<Void> callback) {
        ensureService();
        // Use the Unified API service on Office 365 to create the message.
        mUnifiedAPIService.sendMail(
                "application/json",
                createMailPayload(
                        subject,
                        body,
                        emailAddress),
                callback);
    }

```
### UnifiedAPIService インターフェイス
[MSGraphAPIController.java](https://github.com/microsoftgraph/android-java-connect-rest-sample/blob/master/app/src/main/java/com/microsoft/office365/connectmicrosoftgraph/MSGraphAPIController.java) インターフェイスでは、Retrofit 注釈を使用して、サンプルによる REST 呼び出しのメソッド署名を提供します。

```java
    @POST("/me/sendMail")
    void sendMail(
            @Header("Content-type") String contentTypeHeader,
            @Body TypedString mail,
            Callback<Void> callback);


```

## 次の手順
Microsoft Graph API は、あらゆる種類の Microsoft データとの対話に使用できる、非常に強力な統合 API です。[Microsoft Graph ドキュメント](http://graph.microsoft.io/docs)を参照し、Microsoft Graph API でさらに何を行うことができるかを調べてください。

Office 365 用の Android サンプルを多数公開しました。これらのサンプルは、接続サンプルで紹介する概念に基づいて構築されています。Android アプリでさらに多くのことを実行する場合は、Office GitHub 組織の [Office 365 用の Android サンプルの詳細](http://aka.ms/androidgraphsamples)を参照してください。
 
