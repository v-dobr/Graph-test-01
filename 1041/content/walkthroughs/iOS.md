# iOS アプリで Microsoft Graph を呼び出す

この記事では、アプリケーションを Office 365 に接続し、Microsoft Graph API を呼び出すために必要な最低限のタスクに着目します。[ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) のコードを使用して、アプリに実装する必要のある主要な概念を説明します。このサンプルでは、Microsoft Azure Active Directory (AAD) での認証、および Microsoft Graph API を使用して Office 365 のメール サービスに対して単純なサービス (メールを送信する) を呼び出す中核の基本部分を網羅します。この記事の内容に沿って、クローンを作成するか、このリポジトリからプロジェクトをダウンロードすることをお勧めします。 


この資料では、[ iOS および OSX 用 Microsoft Azure Active Directory 認証ライブラリ (ADAL)](https://github.com/AzureAD/azure-activedirectory-library-for-objc)、およびこのライブラリを使用する [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) サンプル認証を参照します。iOS プロジェクトでの使用方法と実装の詳細は、このリポジトリを参照してください。


## 概要

Microsoft Graph API を呼び出すには、iOS アプリで次のことを行う必要があります。

1. Microsoft Azure Active Directory (AD) からアプリケーションを登録する。
2. Azure AD からアクセス トークンを要求し取得する。
3. Microsoft Graph API への REST 要求にアクセス トークンを使用する。 



## Azure Active Directory にアプリケーションを登録する

Office 365 を使用する前に、アプリケーションを登録し、Microsoft Graph のサービスを使用するアクセス許可を設定する必要があります。数回クリックするだけで、[アプリケーションの登録ツール](https://dev.office.com/app-registration)を使用して、アプリケーションを登録しユーザーの職場または学校のアカウントにアクセスすることができます。
これを管理するには、[Microsoft Azure の管理ポータル](https://manage.windowsazure.com)に移動する必要があります

または、手動でアプリを登録する手順については、記事**Office 365 API にアクセスできるようにアプリを手動で Azure AD に登録する**の [Azure 管理ポータルを使用してネイティブ アプリを登録する](https://msdn.microsoft.com/en-us/office/office365/howto/add-common-consent-manually)セクションを参照してください。以下の詳細な点に注意してください。

* 登録するには、リダイレクト URI を指定する必要があります。これは、認証に成功した後にユーザーがリダイレクトされる場所を指定する必須の値です。正しいリダイレクト URI を指定しない場合、認証要求は失敗します。
* 登録では、アプリに **Microsoft Graph** の**サインインしているユーザーとしてメールを送信するアクセス許可**が付与される必要があります。  


Azure アプリケーションの **[構成]** ページの次の値をメモしてください。

* クライアント ID
* リダイレクト URI

アプリ内に OAuth フローを構成するにはこれらの値が必要です。 

## Azure AD のアクセス トークンを要求し取得する

Microsoft Graph API を呼び出すアクセス トークンを要求し取得するには、**iOS および OSX 用の Microsoft Azure Active Directory 認証ライブラリ (ADAL)** で提供される [acquireAuthTokenWithResource:clientId:redirectUri:completionBlock:](https://github.com/AzureAD/azure-activedirectory-library-for-objc) を使用できます。この SDK により、OAuth2 の業界標準プロトコル サポート、Web API とユーザー レベル コンセントの統合、デュアル ファクター認証サポートなどの Microsoft Azure AD の完全な機能がアプリケーションに与えられます。

このメソッドは、以下のパラメーターをとります。

1. **resourceID** - これはアクセスする必要のある必須のリソースです。たとえば、Microsoft Graph API にアクセスする場合は、この値が "https://graph.microsoft.com" になります。
2. **clientID** - AAD 登録を完了したときに、アプリを識別するために指定される値です。
3. **redirectURI** - 繰り返しになりますが、これは、認証に成功した後にユーザーがリダイレクトされる場所を指定する必須の値です。


まず、認証コンテキストを指定する必要があります。これは単にユーザーのアクセス トークンを取得する機関を定義します。この例の場合、AAD テナントからのものであり、これを宣言する必要があります。

    @property (strong,    nonatomic) ADAuthenticationContext *context;

その後、機関の場所 ("https://login.microsoftonline.com/common") で初期化します。

    self.context = [ADAuthenticationContext authenticationContextWithAuthority:self.authority]; 


[ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) サンプルでは、機関と必須パラメーターによって初期化されるデモを目的として、シングルトン認証クラス (**AuthenticationManager**) を作成します。繰り返しになりますが、このクラスは単に認証ワークフローを実行する方法を示す例です。目的のコード セグメント: 



    - (void)acquireAuthTokenWithResource:(NSString *)resourceID
                            clientID:(NSString*)clientID
                         redirectURI:(NSURL*)redirectURI
                          completion:(void (^)(ADAuthenticationError *error))completion {
    
    NSLog(@"acquireAuthTokenWithResource");
    [self.context acquireTokenWithResource:resourceID
                                  clientId:clientID
                               redirectUri:redirectURI
                           completionBlock:^(ADAuthenticationResult *result) {
                               NSLog(@"Completion");
                               
                               if (result.status !=AD_SUCCEEDED){
                                   NSLog(@"error");
                                   completion(result.error);
                               }
                               
                               else{
                                   NSLog(@"complete!");
                                   self.accessToken = result.accessToken;
                                   self.userID = result.tokenCacheStoreItem.userInformation.userId;
                                   completion(nil);
                               }
                           }];


初めてこのアプリを実行すると、認証マネージャーは機関への要求を送信し、ログイン ページにリダイレクトします。 
資格証明を入力すると、認証結果を含む応答が返されます。 成功した場合、更新トークンとアクセス トークンも含まれます。 
このアプリケーションの 2 回目の実行では、トークン キャッシュとクッキーをクリアしていない場合、認証マネージャーはキャッシュ内のアクセス 
トークンまたは更新トークンを使用してクライアント要求を認証します。 
このため、アクセス トークンを取得する必要がある場合、サービスへの呼び出しが行われます。 


## Microsoft Graph API への要求にアクセス トークンを使用する

アクセス トークンを使用することにより、アプリで、Microsoft Graph API への認証された要求を作成できます。アプリで HTTP 要求ヘッダーの **Authorization** にアクセス トークンを追加する必要があります。

[ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) サンプルは、Microsoft Graph API で sendMail エンドポイントを使用してメールを送信します。繰り返しになりますが、サンプルではアクセス トークンで初期化されるシングルトン認証クラス (AuthenticationManager) を作成しました。要求を作成するには、アクセス トークンが必要になります。



    - (void)sendMailREST {
    
    AuthenticationManager *authManager = [AuthenticationManager sharedInstance];

    //Helper method used to construct the email message
    NSData *postData = [self mailContent];
    
    //Microsoft Graph API endpoint for sending mail
    NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:[NSURL URLWithString:@"https://graph.microsoft.com/v1.0/me/microsoft.graph.sendmail"]];

    [request setHTTPMethod:@"POST"];
    [request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
    [request setValue:@"application/json, text/plain, */*" forHTTPHeaderField:@"Accept"];
    
    // Access token required for request header
    NSString *authorization = [NSString stringWithFormat:@"Bearer %@", authManager.accessToken];
    [request setValue:authorization forHTTPHeaderField:@"Authorization"];
    [request setHTTPBody:postData];

    NSURLConnection *conn = [[NSURLConnection alloc] initWithRequest:request delegate:self];
    
    if(conn) {
        NSLog(@"Connection Successful");
    } else {
        NSLog(@"Connection could not be made");
    }
    
    [conn start];

ご覧のように、応答は NSURLConnection の委任、つまり NSURLConnectionDelegate および NSURLConnectionDataDelegate で処理されます。

## 次の手順

アクセス トークンが期限切れ、または有効期限が近い場合、ADAuthenticationContext の **acquireTokenSilentWithResource:clientId:redirectUri:completionBlock:** を使用して新しいアクセス トークンを取得します。使用方法については、[ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample) サンプルで触れています。また、トークン キャッシュと格納されているクッキーをクリアするコードがあります。  

Microsoft Graph API は、あらゆる種類の Microsoft データとの対話に使用できる、非常に強力な統合 API です。[API リファレンス](http://graph.microsoft.io/docs/api-reference/v1.0)を参照し、Microsoft Graph API で何を行うことができるかを調べてください。

