# ユニバーサル Windows 10 アプリで Microsoft Graph を呼び出す

ここでは、Azure Active Directory (AD) からアクセス トークンを取得して、Microsoft Graph を呼び出すために必要な最低限のタスクに着目します。[Microsoft Graph を使用する UWP 用の Office 365 接続サンプル](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample)のコードを使用して、アプリに実装する必要のある主要な概念を説明します。

## サンプル ユーザー インターフェイス

サンプルには、上部のコマンド バー、**[接続]** ボタン､**[メールの送信]** ボタン、およびサインイン ユーザーの電子メールアドレスが自動入力された編集可能なテキスト ボックスで構成される非常にシンプルなユーザー インターフェイスが含まれています。コマンド バーには、開発者がアプリのリダイレクト URI を検索できるボタンも含まれています。

**[メールの送信]** ボタンは、ユーザーが接続されていないときは無効になります。

![[接続] ボタンが有効になっていて、[メールの送信] ボタンが無効になっていることを示す画面](images/SignedOut.png)

ユーザーが接続されている場合、上部のコマンド バーに [切断] ボタンが表示されます。

![接続されたユーザーの電子メール アドレスと [メールの送信] ボタンが有効になっていることを示す画面](images/SignedIn.png)

すべてのサンプルの UI 文字列は、Assets フォルダー内の Resources.resw ファイルに格納されています。

## アプリを登録する
 
Windows 10 では、各アプリケーションに一意の URI があり、その URI に送信されたメッセージだけがそのアプリケーションに送信されます。アプリを作成し、アプリを登録する前に、このシステムによって生成される URI を検索する必要があります。サンプルでは、AuthenticationHelper.cs ファイルにこのメソッドが用意されています。

```c#
        public static string GetAppRedirectURI()
        {
            // Windows 10 universal apps require redirect URI in the format below. Add a breakpoint to this line and run the app before you register it, so that
            // you can supply the correct redirect URI value.
            return string.Format("ms-appx-web://microsoft.aad.brokerplugin/{0}", WebAuthenticationBroker.GetCurrentApplicationCallbackUri().Host).ToUpper();
        }
```

このメソッドは、サンプルで **[リダイレクト URI にコピー]** ボタンによってトリガーされますが、[AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM) サンプルのパターンを使用することもできます。ここでは、MainPage クラスの宣言で文字列が定義されており、Visual Studio デバッガーを使用して取り出すことができます。 

サンプルの Readme にある[アプリを登録して設定する](https://github.com/microsoftgraph/uwp-csharp-connect-rest-sample#register)手順に従って、リダイレクト URI 値を取得した後にアプリを登録します。

アプリの認証を設定するときには、Azure アプリケーションの **[構成]** ページのクライアント ID が必要です。

## Microsoft Graph へ接続する

サンプルでは、ネイティブの Windows 10 WebAccountManager API を使用して、ユーザーを認証します。[AzureAD NativeClient UWP WAM](http://blogs.technet.com/b/ad/archive/2015/08/03/develop-windows-universal-apps-with-azure-ad-and-the-windows-10-identity-api.aspx) サンプルでは、[Azure AD および Windows 10 Identity API による Windows ユニバーサル アプリの開発](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM)のブログ投稿で説明されているものと同様のパターンに沿ったデモを示します。

App.xaml ファイルには、ユーザーを認証するため、および電子メールの送信を承認するためにアプリで必要なキーと値のペアが含まれています。

```xml
    <Application.Resources>
        <!-- Add your client id here. -->
        <x:String x:Key="ida:ClientID"><your client id></x:String>
        <x:String x:Key="ida:AADInstance">https://login.microsoftonline.com/</x:String>
        <!-- Add your developer tenant domain here. -->
        <x:String x:Key="ida:Domain">yourtenant.onmicrosoft.com</x:String>
    </Application.Resources>
```

アプリを登録したときに取得したクライアント ID の値を、**ida:ClientID** キーとして追加します。**ida:Domain** キーの値をOffice 365 テナントと一致するように変更します。

AuthenticationHelper.cs ファイルには、ユーザー情報を格納し、アプリからユーザーが切断されたときにのみ認証を強制する追加のロジックとともに、すべての認証コードが含まれています。

このファイルで定義されている ``GetTokenHelperAsync`` メソッドは、ユーザーを認証し、その後、Microsoft Graph を呼び出すたびに実行されます。その最初のタスクは Azure AD account プロバイダーを検索することです。

```c#
           aadAccountProvider = await WebAuthenticationCoreManager.FindAccountProviderAsync("https://login.microsoft.com", authority);
```

``authority`` の値は App.xaml ファイルに格納されている 2 つの値、**ida:AADInstance** キーの値と **ida:Domain** キーの値から作成された連結文字列です。これによって、テナント固有の権限が作成されます。また、任意の Azure AD テナントでアプリを実行する場合は、"organizations" 文字列も使用できます。

ユーザーの認証後、アプリによってそのユーザー ID の値が ``ApplicationData.Current.RoamingSettings`` に格納されます。``GetTokenHelperAsync`` メソッドでは、最初にこの値があるかどうかを確認し、ある場合はサイレント モードで認証しようとします。

```c#
            // Check if there's a record of the last account used with the app
            var userID = _settings.Values["userID"];

            if (userID != null)
            {

                WebTokenRequest webTokenRequest = new WebTokenRequest(aadAccountProvider, string.Empty, clientId);
                webTokenRequest.Properties.Add("resource", ResourceUrl);

                // Get an account object for the user
                userAccount = await WebAuthenticationCoreManager.FindAccountAsync(aadAccountProvider, (string)userID);


                // Ensure that the saved account works for getting the token we need
                WebTokenRequestResult webTokenRequestResult = await WebAuthenticationCoreManager.RequestTokenAsync(webTokenRequest, userAccount);
                if (webTokenRequestResult.ResponseStatus == WebTokenRequestStatus.Success || webTokenRequestResult.ResponseStatus == WebTokenRequestStatus.AccountSwitch)
                {
                    WebTokenResponse webTokenResponse = webTokenRequestResult.ResponseData[0];
                    userAccount = webTokenResponse.WebAccount;
                    token = webTokenResponse.Token;

                }
                else
                {
                    // The saved account could not be used for getting a token
                    // Make sure that the UX is ready for a new sign in
                    SignOut();
                }

            }
```

アプリでは Microsoft Graph エンドポイント ( **https://graph.microsoft.com/**) がリソース値として使用されます。``WebTokenRequest`` が構成されると、App.xaml ファイルに追加したクライアント ID の値が使用されます。ユーザー ID がアプリで認識されており、ユーザーが切断されていないため、WebAccountManager API ではユーザー アカウントが検出され、トークン要求に渡されます。``WebAuthenticationCoreManager.RequestTokenAsync`` メソッドによって、アクセス トークンとそれに割り当てられている適切なアクセス許可が返されます。

ローミングの設定に ``userID`` の値が見つからない場合は、UI を介してユーザーに認証させる ``WebTokenRequest`` が構成されます。

```c#
            else
            {
                // There is no recorded user. Start a sign in flow without imposing a specific account.

                WebTokenRequest webTokenRequest = new WebTokenRequest(aadAccountProvider, string.Empty, clientId, WebTokenRequestPromptType.ForceAuthentication);
                webTokenRequest.Properties.Add("resource", ResourceUrl);

                WebTokenRequestResult webTokenRequestResult = await WebAuthenticationCoreManager.RequestTokenAsync(webTokenRequest);
                if (webTokenRequestResult.ResponseStatus == WebTokenRequestStatus.Success)
                {
                    WebTokenResponse webTokenResponse = webTokenRequestResult.ResponseData[0];
                    userAccount = webTokenResponse.WebAccount;
                    token = webTokenResponse.Token;

                }
            }
```

いずれかの方法でトークンを取得できた場合は、``GetTokenHelperAsync`` メソッドにより、ローミング設定へのユーザーの重要設定の格納が完了し、トークンの値が返されます。取得できなかった場合は、ローミング設定が null であることが確認され、null 値が返されます。

```c#
            // We succeeded in getting a valid user.
            if (userAccount != null)
            {
                // save user ID in local storage
                _settings.Values["userID"] = userAccount.Id;
                _settings.Values["userEmail"] = userAccount.UserName;
                _settings.Values["userName"] = userAccount.Properties["DisplayName"];

                return token;
            }

            // We didn't succeed in getting a valid user. Clear the app settings so that another user can sign in.
            else
            {
                
                SignOut();
                return null;
            }
```

## Microsoft Graph を使用して電子メールを送信する

MailHelper.cs ファイルには、電子メールを作成して送信するコードが含まれています。それは、POST 要求を作成して **https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail** エンドポイントに送信する単一のメソッド (``ComposeAndSendMailAsync``) で構成されています。 

``ComposeAndSendMailAsync`` メソッドは、MainPage.xaml.cs ファイルによって渡される 3 つの文字列値 (``subject``、``bodyContent``、``recipients``) を取得します。``subject`` および ``bodyContent`` の文字列は、その他のすべての UI 文字列とともに Resources.resw ファイルに格納されます。``recipients`` の文字列はアプリのインターフェイスにあるアドレス ボックスのものです。 

アドレスは複数渡すことができるため、最初に、要求の POST 本文に入れて渡せるように、``recipients`` 文字列を一連の EmailAddress オブジェクトに分割します。

```c#
            // Prepare the recipient list
            string[] splitter = { ";" };
            var splitRecipientsString = recipients.Split(splitter, StringSplitOptions.RemoveEmptyEntries);
            string recipientsJSON = null;

            int n = 0;
            foreach (string recipient in splitRecipientsString)
            {
                if ( n==0)
                recipientsJSON += "{'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                else
                {
                    recipientsJSON += ", {'EmailAddress':{'Address':'" + recipient.Trim() + "'}}";
                }
                n++;
            }
```

次に、有効な JSON メッセージ オブジェクトを作成し、HTTP POST 要求を使用して **me/microsoft.graph.SendMail** エンドポイントに送信します。``bodyContent`` の文字列は HTML 文書であるため、要求によって **ContentType** 値が HTML に設定されます。また、``AuthenticationHelper.GetTokenHelperAsync`` への呼び出しにより、要求に渡される新規のアクセス トークンがあることを確認します。

```c#
                HttpClient client = new HttpClient();
                var token = await AuthenticationHelper.GetTokenHelperAsync();
                client.DefaultRequestHeaders.Add("Authorization", "Bearer " + token);

                // Build contents of post body and convert to StringContent object.
                // Using line breaks for readability.
                string postBody = "{'Message':{" 
                    +  "'Body':{ " 
                    + "'Content': '" + bodyContent + "'," 
                    + "'ContentType':'HTML'}," 
                    + "'Subject':'" + subject + "'," 
                    + "'ToRecipients':[" + recipientsJSON +  "]}," 
                    + "'SaveToSentItems':true}";

                var emailBody = new StringContent(postBody, System.Text.Encoding.UTF8, "application/json");

                HttpResponseMessage response = await client.PostAsync(new Uri("https://graph.microsoft.com/v1.0/me/microsoft.graph.SendMail"), emailBody);

                if ( !response.IsSuccessStatusCode)
                {

                    throw new Exception("We could not send the message: " + response.StatusCode.ToString());
                }
```

REST 要求の作成が完了すると、Microsoft Graph と対話するために必要な、アプリの登録、ユーザーの認証、および REST 要求の作成の 3 つの手順が完了したことになります。


<!--## Additional resources

* [Develop Windows Universal Apps with Azure AD and the Windows 10 Identity API](http://blogs.technet.com/b/ad/archive/2015/08/03/develop-windows-universal-apps-with-azure-ad-and-the-windows-10-identity-api.aspx)
* [AzureAD-NativeClient-UWP-WAM](https://github.com/Azure-Samples/AzureAD-NativeClient-UWP-WAM)
* [Office Dev Center](http://dev.office.com)-->

