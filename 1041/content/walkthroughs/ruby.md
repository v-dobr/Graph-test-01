# Ruby アプリで Microsoft Graph を呼び出す 

ここでは、Azure Active Directory (AD) からアクセス トークンを取得して、Microsoft Graph API を呼び出すために必要な最低限のタスクに着目します。[Microsoft Graph を使用する Office 365 Ruby 接続サンプル](https://github.com/microsoftgraph/ruby-connect-rest-sample)のコードを使用して、アプリに実装する必要のある主要な概念を説明します。

![Office 365 Ruby Connect サンプルのスクリーンショット](./images/web-screenshot.png)

## 概要

Microsoft Graph API を呼び出すには、Ruby アプリで次のタスクを行う必要があります。

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

* 手順 6 の**サインオン URL** として Ruby アプリでルートを指定します。接続サンプルの場合、これは [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L41) です。
* アプリに必要な[**委任されたアクセス許可**を構成](https://github.com/microsoftgraph/ruby-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure)します。接続サンプルには、**サインインしているユーザーとしてメールを送信する**アクセス許可が必要です。

Azure アプリケーションの **[構成]** ページの次の値をメモしてください。

* クライアント ID
* 有効なキー
* 応答 URL

アプリ内の OAuth フローのパラメーターとしてこれらの値が必要です。

<!--<a name="redirect"/>-->
## ブラウザーをサインイン ページにリダイレクトする

アプリはブラウザーをサインイン ページにリダイレクトして認証コードを取得し、OAuth のフローを続行する必要があります。

接続サンプルでは、リダイレクトは、OmniAuth ライブラリによって処理されます。アプリは単に、その実行を OmniAuth によって管理されている [`/auth/azureactivedirectory`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L30) ルートに委任します。

<!--<a name="authcode"/>-->
## 応答 URL ページで認証コードを受け取る

ユーザーがサインインすると、フローはアプリの応答 URL にブラウザーを返します。Azure では、クエリ文字列に認証コードを追加します。接続サンプルは、この目的のために [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L38) ルートを使用します。

認証コードは `code` クエリ文字列変数で提供されます。接続サンプルでは、後で使用するためコードをローカル変数に保存します。

```ruby
code = params[:code]
```

<!--<a name="accesstoken"/>-->
## トークン エンドポイントからアクセス トークンを要求する

認証コードを作成したら、Azure AD から取得したクライアント ID、キー、応答 URL の値に沿ってこれを使用して、アクセス トークンを要求できます。 

> **注:** <br />
> 要求には、使用する必要のあるリソースも指定してください。Microsoft Graph API の場合、リソースの値は `https://graph.microsoft.com` です。

再び、接続サンプルはこのタスクを OmniAuth ライブラリに委任します。[`acquire_access_token`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L65) 関数はライブラリを呼び出し、前のセクションで保存した認証コードを応答 URL、クライアント ID、クライアント シークレット、およびリソース ID とともに渡します。

```ruby
def acquire_access_token(auth_code, reply_url)
    AUTH_CTX.acquire_token_with_authorization_code(
                  auth_code,
                  reply_url,
                  CLIENT_CRED,
                  GRAPH_RESOURCE)
end
```

> **注:** <br />
> クライアント ID とクライアント シークレットは、前のコード スニペット内の `CLIENT_CRED` パラメーターで提供されます。

<!--<a name="request"/>-->
## Microsoft Graph API への要求にアクセス トークンを使用する

アクセス トークンを使用することにより、ご自分のアプリで Microsoft Graph API へ認証された要求を行うことができます。アプリの各要求の **Authorization** ヘッダーで、アクセス トークンを指定する必要があります。

接続サンプルでは、Microsoft Graph API で **sendMail** エンドポイントを使用してメールを送信します。このコードは [`send_mail`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L82) 関数内にあります。これは、Authorization ヘッダーに入れてアクセス コードを送信する方法を示すコードです。

```ruby
def send_mail
  # Used in the template
  @name = session[:name]
  @email = params[:specified_email]
  @recipient = params[:specified_email]
  @mailSent = false
  
  sendMailEndpoint = URI("#{GRAPH_RESOURCE}#{SENDMAIL_ENDPOINT}")
  http = Net::HTTP.new(sendMailEndpoint.host, sendMailEndpoint.port)
  http.use_ssl = true
  
  emailBody = File.read("app/assets/MailTemplate.html")
  emailBody.sub! "{given_name}", @name
  
  emailMessage = "{
          Message: {
          Subject: 'Welcome to Office 365 development with Ruby',
          Body: {
              ContentType: 'HTML',
              Content: '#{emailBody}'
          },
          ToRecipients: [
              {
                  EmailAddress: {
                      Address: '#{@recipient}'
                  }
              }
          ]
          },
          SaveToSentItems: true
          }"

  response = http.post(
    SENDMAIL_ENDPOINT, 
    emailMessage, 
    initheader = 
    {
      "Authorization" => "Bearer #{session[:access_token]}", 
      "Content-Type" => CONTENT_TYPE
    }
  )

  # The send mail endpoint returns a 202 - Accepted code on success
  if response.code == "202"
    @mailSent = true
  else
    @mailSent = false
    flash[:httpError] = "#{response.code} - #{response.message}"
  end
  
  render "callback"
end
```

> **注:** <br />
> 要求は **Content-Type** ヘッダーに `application/json;odata.metadata=minimal;odata.streaming=true` など、Microsoft Graph API に受け入れられる値を指定して送信します。

Microsoft Graph API は、あらゆる種類の Microsoft データとの対話に使用できる、非常に強力な統合 API です。API リファレンスを参照し、Microsoft Graph API で何を行うことができるかを調べてください。

<!--## Additional resources

-  [Office 365 Ruby Connect sample using Microsoft Graph](https://github.com/microsoftgraph/ruby-connect-rest-sample)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference](http://graph.microsoft.io/en-us/docs)-->
