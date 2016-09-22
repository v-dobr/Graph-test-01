# Python アプリで Microsoft Graph を呼び出す 

この記事では、アプリケーションを Office 365 に接続し、Microsoft Graph AP を呼び出すために必要な最低限のタスクに着目します。[Microsoft Graph を使用する Office 365 Python 接続サンプル](https://github.com/microsoftgraph/python3-connect-rest-sample)のコードを使用して、アプリに実装する必要のある主要な概念を説明します。

![Office 365 Python 接続サンプルのスクリーンショット](./images/web-screenshot.png)

##  前提条件

このトピックでは、以下のことを前提としています。

* Python コードの知識があること。
* OAuth の概念に精通していること。

## 概要

Microsoft Graph API を呼び出すには、Python アプリで次のタスクを行う必要があります。

1. Azure Active Directory にアプリケーションを登録する
2. ブラウザーをサインイン ページにリダイレクトする
3. 応答 URL ページで認証コードを受け取る
4. トークンを発行するエンドポイントからアクセス トークンを要求する
5. Microsoft Graph API への要求にアクセス トークンを使用する 

<!--<a name="register"></a>-->
## Azure Active Directory にアプリケーションを登録する

Office 365 を使用する前に、アプリケーションを登録し、Microsoft Graph のサービスを使用するアクセス許可を設定する必要があります。数回クリックするだけで、[アプリケーションの登録ツール](https://dev.office.com/app-registration)を使用して、アプリケーションを登録しユーザーの職場または学校のアカウントにアクセスすることができます。
これを管理するには、[Microsoft Azure の管理ポータル](https://manage.windowsazure.com)に移動する必要があります

あるいは、「[Azure 管理ポータルに Web サーバー アプリを登録する](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp)」のセクションを参照して、手動でアプリを登録する手順をご確認ください。以下の詳細な点にご注意ください。

* **サインオン URL** として、必ず http://127.0.0.1:8000/connect/get_token/ を指定してください。
* アプリケーションを登録した後、Python アプリで必要な[**委任されたアクセス許可**を構成します](https://github.com/microsoftgraph/python3-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure)。接続サンプルには、**サインインしているユーザーとしてメールを送信する**アクセス許可が必要です。

Azure アプリケーションの **[構成]** ページの次の値をメモしてください。この値は Python アプリの OAuth フローを構成する上で必要になります。

* クライアント ID (アプリケーションで一意)
* 返信の URL (http://127.0.0.1:8000/connect/get_token/)
* アプリケーション キー (あなたのアプリケーションに固有)

<!--<a name="redirect"></a>-->
## ブラウザーをサインイン ページにリダイレクトする

アプリはブラウザーをサインイン ページにリダイレクトして OAuth フローを開始し、認証コードを取得する必要があります。 

接続サンプルの次のコード ([*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py) を参照) では、アプリユーザーをリダイレクトするため、またリダイレクトに使用するビューにパイプされるためにアプリが必要とする URL を構築します。 

```python
# This function creates the signin URL that the app will
# direct the user to in order to sign in to Office 365 and
# give the app consent.
def get_signin_url(redirect_uri):
  # Build the query parameters for the signin URL.
  params = { 'client_id': client_id,
             'redirect_uri': redirect_uri,
             'response_type': 'code'
           }

  authorize_url = '{0}{1}'.format(authority, '/common/oauth2/authorize?{0}')
  signin_url = authorize_url.format(urlencode(params))
  return signin_url
```

<!--<a name="authCode"></a>-->
## 応答 URL ページで認証コードを受け取る

ユーザーがサインインした後、ブラウザーは応答 URL にリダイレクトされ、[*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py) 内の ```get_token``` 関数と認証コードがクエリ文字列に ```code``` 変数として付加されます。 

接続サンプルは、クエリ文字列からコードを取得し、アクセス トークン用に交換することができます。

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## トークンを発行するエンドポイントからアクセス トークンを要求する

認証コードを作成したら、Azure Active Directory から取得したクライアント ID、キー、応答 URL の値に沿ってこれを使用して、アクセス トークンを要求できます。 

> **注**: 要求には、使用する必要のあるリソースも指定してください。Microsoft Graph の場合、リソースの値は `https://graph.microsoft.com` です。

接続サンプルでは、[*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py) ファイルの ```get_token_from_code``` 関数を要求しています。

```python
# This function passes the authorization code to the token
# issuing endpoint, gets the token, and then returns it.
def get_token_from_code(auth_code, redirect_uri):
  # Build the post form for the token request
  post_data = { 'grant_type': 'authorization_code',
                'code': auth_code,
                'redirect_uri': redirect_uri,
                'client_id': client_id,
                'client_secret': client_secret,
                'resource': 'https://graph.microsoft.com'
              }
              
  r = requests.post(token_url, data = post_data)
  
  try:
    return r.json()
  except:
    return 'Error retrieving token: {0} - {1}'.format(r.status_code, r.text)
```

> **注**: 応答では、アクセス トークン以外の情報も提供されます。たとえば、アプリは更新トークンを取得することにより、ユーザーが再度明示的にサインインしなくても、新しいアクセス トークンを要求できます。

<!--<a name="request"></a>-->
## Microsoft Graph API への要求にアクセス トークンを使用する

アクセス トークンを使用することにより、あなたのアプリは Microsoft Graph API へ認証された要求を行うことができます。アプリで各要求の **Authorization** ヘッダーにアクセス トークンを追加する必要があります。

接続サンプルは、Microsoft Graph API で ```me/microsoft.graph.sendMail``` エンドポイントを使用してメールを送信します。コードは [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py) ファイル内の ```call_sendMail_endpoint``` 関数に記述されています。これは、Authorization ヘッダーにアクセス コードを付加する方法を示すコードです。

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **注**: 要求は **Content-Type** ヘッダーに `application/json` など、Graph API に受け入れられる値も指定して送信します。

Microsoft Graph API は、あらゆる種類の Microsoft データとの対話に使用できる、非常に強力な統合 API です。API リファレンスを参照し、Microsoft Graph API で何を行うことができるかを調べてください。

<!--
## Additional resources

-  [Office 365 Python Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Python-Microsoft-Graph-Connect)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference]()-->
