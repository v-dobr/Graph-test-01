# Android アプリで Microsoft Graph を使ってみる

この記事では、v2.0 認証エンドポイントからアクセス トークンを取得し、Microsoft Graph を呼び出すために必要なタスクについて説明します。 ここでは、[Android 用接続サンプル](https://github.com/microsoftgraph/android-java-connect-sample)のビルドの手順と、Android 用アプリで Microsoft Graph を使用するために実装する主要な概念について説明します。 またこの記事では、[Microsoft Graph SDK for Android](https://github.com/microsoftgraph/msgraph-sdk-android) または未加工の REST 呼び出しを使用して Microsoft Graph にアクセスする方法についても説明します。

Android 用アプリで Microsoft Graph を使用するには、以下のスクリーンショットに示すような Microsoft のサインイン ページをユーザーに表示する必要があります。

![Android 上の Microsoft アカウントのサインイン ページ](images/AndroidConnect.png)

**アプリを構築するのは難しいと感じていますか?** この記事で取り扱っている [Android 用接続サンプル](https://github.com/microsoftgraph/android-java-connect-sample)をダウンロードすれば、簡単に開始できます。


## 前提条件

開始するには、次のものが必要です。 

- [Microsoft アカウント](https://www.outlook.com/)または [Office 365 for Business アカウント](http://dev.office.com/devprogram)
- Android Studio 2.0 以降のバージョン


## アプリケーションの登録
Microsoft アプリケーション登録ポータルでアプリケーションを登録します。 これにより、アプリの構成に使用するアプリ ID とパスワードが生成されます。

1. 個人用アカウント、あるいは職場または学校アカウントのいずれかを使用して、[Microsoft アプリ登録ポータル](https://apps.dev.microsoft.com/)にサインインします。

2. **[アプリの追加]** を選択します。

3. アプリの名前を入力して、**[アプリケーションの作成]** を選択します。 
    
   登録ページが表示され、アプリのプロパティが一覧表示されます。

4. アプリケーション ID をコピーします。 これは、アプリの一意識別子です。 

5. **[プラットフォームの追加]** および **[モバイル アプリケーション]** を選択します。

    > **注:**アプリケーション登録ポータルでは、値 *urn: ietf:wg:oauth:2.0:oob* のリダイレクト URI が表示されます。 ただし、リダイレクト URI の既定値 *https://login.microsoftonline.com/common/oauth2/nativeclient* を使用します。

6. **[保存]** を選択します。


## プロジェクトを構成する

Android Studio で新しいプロジェクトを開始します。 ほとんどのウィザードで既定値のままにしておきますが、次のオプションを選択するようにしてください。

* ターゲット Android デバイス - **携帯電話とタブレット**
    * 最小 SDK - **API 16:Android 4.1 (Jelly Bean)**
* モバイルにアクティビティを追加 - **基本的なアクティビティ**
 
これにより、Android プロジェクトにアクティビティとユーザーの認証に使用できるボタンが追加されます。

## ユーザーの認証とアクセス トークンの取得
OAuth ライブラリを使用して、認証プロセスを簡素化します。 [OpenID](http://openid.net) により、このプロジェクトで使用できるライブラリである [Android の AppAuth](https://github.com/openid/AppAuth-Android) が提供されます。

### app/build.gradle に依存関係を追加します。

アプリのモジュールで `build.gradle` ファイルを開き、次の依存関係を組み込みます。

```gradle
compile 'net.openid:appauth:0.3.0'
```

### 認証フローの開始

1. **MainActivity** ファイルを開き、**onCreate** メソッドで **AuthorizationService** オブジェクトを宣言します。
    ```java
    final AuthorizationService authorizationService =
        new AuthorizationService(this);
    ```
    
2. *FloatingActionButton* のクリック イベントのイベント ハンドラーを配置します。 **onClick** メソッドを次のコードに置き換えます。 アプリの **アプリケーション ID** を、**\<YOUR_APPLICATION_ID\>** とマークされているプレースホルダーに挿入します。
    ```java
    @Override
    public void onClick(View view) {
        Uri authorizationEndpoint =
            Uri.parse("https://login.microsoftonline.com/common/oauth2/v2.0/authorize");
        Uri tokenEndpoint =
            Uri.parse("https://login.microsoftonline.com/common/oauth2/v2.0/token");
        AuthorizationServiceConfiguration config =
            new AuthorizationServiceConfiguration(
                    authorizationEndpoint,
                    tokenEndpoint,null);

        List<String> scopes = new ArrayList<>(
            Arrays.asList("openid mail.send".split(" ")));

        AuthorizationRequest authorizationRequest = new AuthorizationRequest.Builder(
            config,
            "<YOUR_APPLICATION_ID>",
            ResponseTypeValues.CODE,
            Uri.parse("https://login.microsoftonline.com/common/oauth2/nativeclient"))
            .setScopes(scopes)
            .build();

        Intent intent = new Intent(view.getContext(), MainActivity.class);

        PendingIntent redirectIntent =
            PendingIntent.getActivity(
                    view.getContext(),
                    authorizationRequest.hashCode(),
                    intent, 0);

        authorizationService.performAuthorizationRequest(
            authorizationRequest,
            redirectIntent);
    }
    ```
    
この時点で、ボタンの付いた Android アプリができているはずです。 ボタンを押すと、デバイスのブラウザーを使用した認証ページがアプリに表示されます。 次の手順は、認証サーバーがリダイレクト URI にコードを送信してアクセス トークンと交換する作業です。

### 認証コードとアクセス トークンの交換

アクセス トークンと交換できるコードが含まれている認証サーバーの応答をアプリが処理できるようにする必要があります。

1. アプリが *https://login.microsoftonline.com/common/oauth2/nativeclient* への要求を処理できるということを、Android システムに認識させる必要があります。 これを行うには、**AndroidManifest** ファイルを開き、次のアクティビティ要素を追加します。
    ```xml
    <activity android:name="net.openid.appauth.RedirectUriReceiverActivity">
        <intent-filter>
            <action android:name="android.intent.action.VIEW"/>
            <category android:name="android.intent.category.DEFAULT"/>
            <category android:name="android.intent.category.BROWSABLE"/>
            <data android:scheme="https"/>
            <data android:host="login.microsoftonline.com"/>
            <data android:path="/common/oauth2/nativeclient"/>
        </intent-filter>
    </activity>
    ```

2. 認証サーバーが応答を送信すると、アクティビティが呼び出されます。 認証サーバーからの応答で、アクセス トークンを要求できます。 **MainActivity** に戻り、**onCreate** メソッドに次のコードを追加します。
    ```java
    Bundle extras = getIntent().getExtras();
    if (extras != null) {
        AuthorizationResponse authorizationResponse = AuthorizationResponse.fromIntent(getIntent());
        AuthorizationException authorizationException = AuthorizationException.fromIntent(getIntent());
        final AuthState authState = new AuthState(authorizationResponse, authorizationException);

        if (authorizationResponse != null) {
            HashMap<String, String> additionalParams = new HashMap<>();
            TokenRequest tokenRequest = authorizationResponse.createTokenExchangeRequest(additionalParams);

            authorizationService.performTokenRequest(
                tokenRequest,
                new AuthorizationService.TokenResponseCallback() {
                    @Override
                    public void onTokenRequestCompleted(
                            @Nullable TokenResponse tokenResponse,
                            @Nullable AuthorizationException ex) {
                        authState.update(tokenResponse, ex);
                        if (tokenResponse != null) {
                            String accessToken = tokenResponse.accessToken;
                        }
                    }
                });
        } else {
            Log.i("MainActivity", "Authorization failed: " + authorizationException);
        }
    }
    ```

この行 `String accessToken = tokenResponse.accessToken;` にアクセス トークンがあることにご注意ください。 これで、Microsoft Graph を呼び出すためのコードを追加する準備が整いました。 

## Microsoft Graph を呼び出す
[Microsoft Graph SDK](#call-microsoft-graph-using-the-microsoft-graph-sdk) または [Microsoft Graph REST API](#call-microsoft-graph-using-the-microsoft-graph-rest-api) を使用して、Microsoft Graph を呼び出すことができます。

### Microsoft Graph SDK を使用して Microsoft Graph を呼び出す
[Microsoft Graph SDK for Android](https://github.com/microsoftgraph/msgraph-sdk-android) では、要求をビルドして Microsoft Graph からの結果を処理するクラスを作成できます。 以下の手順に従って Microsoft Graph SDK を使用します。

1. アプリにインターネット アクセス許可を付与します。 **AndroidManifest** ファイルを開き、マニフェスト要素に次の子を追加します。
    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    ```

2. Microsoft Graph SDK および GSON に依存関係を追加します。
   ```gradle
    compile 'com.microsoft.graph:msgraph-sdk-android:1.0.0'
    compile 'com.google.code.gson:gson:2.4'
   ```
   
3. 行 `String accessToken = tokenResponse.accessToken;` を次のコードに置き換えます。 **\<YOUR_EMAIL_ADDRESS\>** とマークされているプレースホルダーに電子メール アドレスを挿入します。
    ```java
    final String accessToken = tokenResponse.accessToken;
    final IClientConfig clientConfig = 
            DefaultClientConfig.createWithAuthenticationProvider(new IAuthenticationProvider() {
        @Override
        public void authenticateRequest(IHttpRequest request) {
            request.addHeader("Authorization", "Bearer " + accessToken);
        }
    });

    final IGraphServiceClient graphServiceClient = new GraphServiceClient
        .Builder()
        .fromConfig(clientConfig)
        .buildClient();

    final Message message = new Message();
    EmailAddress emailAddress = new EmailAddress();
    emailAddress.address = "<YOUR_EMAIL_ADDRESS>";
    Recipient recipient = new Recipient();
    recipient.emailAddress = emailAddress;
    message.toRecipients = Collections.singletonList(recipient);
    ItemBody itemBody = new ItemBody();
    itemBody.content = "This is the email body";
    itemBody.contentType = BodyType.text;
    message.body = itemBody;
    message.subject = "Sent using the Microsoft Graph SDK";

    AsyncTask.execute(new Runnable() {
        @Override
        public void run() {
            graphServiceClient
                .getMe()
                .getSendMail(message, false)
                .buildRequest()
                .post();
        }
    });
    ```

### Microsoft Graph REST API を使用して Microsoft Graph を呼び出す
[Microsoft Graph REST API](http://graph.microsoft.io/docs) は 1 つの REST API エンドポイントを介して複数の API を Microsoft クラウド サービスから公開するものです。 以下の手順に従って、REST API を使用します。

1. アプリにインターネット アクセス許可を付与します。 **AndroidManifest** ファイルを開き、マニフェスト要素に次の子を追加します。
    ```xml
    <uses-permission android:name="android.permission.INTERNET" />
    ```

2. Volley HTTP ライブラリへの依存関係を追加します。
   ```gradle
    compile 'com.android.volley:volley:1.0.0'
   ```
   
3. 行 `String accessToken = tokenResponse.accessToken;` を次のコードに置き換えます。 **\<YOUR_EMAIL_ADDRESS\>** とマークされているプレースホルダーに電子メール アドレスを挿入します。
    ```java
    final String accessToken = tokenResponse.accessToken;

    final RequestQueue queue = Volley.newRequestQueue(getApplicationContext());
    String url ="https://graph.microsoft.com/v1.0/me/sendMail";
    final String body = "{" +
        "  Message: {" +
        "    subject: 'Sent using the Microsoft Graph REST API'," +
        "    body: {" +
        "      contentType: 'text'," +
        "      content: 'This is the email body'" +
        "    }," +
        "    toRecipients: [" +
        "      {" +
        "        emailAddress: {" +
        "          address: '<YOUR_EMAIL_ADDRESS>'" +
        "        }" +
        "      }" +
        "    ]}" +
        "}";

    final StringRequest stringRequest = new StringRequest(Request.Method.POST, url,
        new Response.Listener<String>() {
            @Override
            public void onResponse(String response) {
                Log.d("Response", response);
            }
        },
        new Response.ErrorListener() {
            @Override
            public void onErrorResponse(VolleyError error) {
                Log.d("ERROR","error => " + error.getMessage());
            }
        }
    ) {
        @Override
        public Map<String, String> getHeaders() throws AuthFailureError {
            Map<String,String> params = new HashMap<>();
            params.put("Authorization", "Bearer " + accessToken);
            params.put("Content-Length", String.valueOf(body.getBytes().length));
            return params;
        }
        @Override
        public String getBodyContentType() {
            return "application/json";
        }
        @Override
        public byte[] getBody() throws AuthFailureError {
            return body.getBytes();
        }
    };

    AsyncTask.execute(new Runnable() {
        @Override
        public void run() {
            queue.add(stringRequest);
        }
    });
    ```

## アプリの実行
Android アプリを試す準備ができました。

1. Android エミュレーターを起動するか、物理デバイスをコンピューターに接続します。
2. Android Studio で、Shift キーを押しながら F10 キーを押してアプリを実行します。
3. 配置ダイアログ ボックスから、Android のエミュレーターまたはデバイスを選択します。
4. メインのアクティビティ内のフローティング アクション ボタンをタップします。
5. 個人用あるいは職場または学校アカウントでサインインし、要求されたアクセス許可を付与します。
6. アプリの選択ダイアログで、アプリをタップして続行します。

["Microsoft Graph を呼び出す"](#call-the-microsoft-graph)で構成した電子メール アドレスの受信トレイを確認します。 アプリへのサインインに使用したアカウントからのメールを受信しているはずです。

## 次の手順
- [Graph エクスプローラー](https://graph.microsoft.io/graph-explorer)を使用して REST API を試してみます。
- [Android 用のスニペットのサンプル](https://github.com/microsoftgraph/android-java-snippets-sample)で一般的な操作の例を見つけるか、GitHub で別の [Android サンプル](https://github.com/microsoftgraph?utf8=%E2%9C%93&query=android)を探します。


## 関連項目
* [Microsoft Graph の概要](http://graph.microsoft.io/docs)
* [Microsoft Graph SDK for Android](https://github.com/microsoftgraph/msgraph-sdk-android) 
* [Azure AD v2.0 のプロトコル](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols/)
* [Azure AD v2.0 のトークン](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-tokens/)
