#  Angular アプリで Microsoft Graph を呼び出す 

この記事では、アプリケーションを Office 365 に接続し、Microsoft Graph AP を呼び出すために必要な最低限のタスクについて説明します。[Microsoft Graph を使用する Office 365 Angular 接続サンプル](https://github.com/microsoftgraph/angular-connect-rest-sample)のコードを使用して、アプリに実装する必要のある主要な概念を説明します。

![Office 365 Angular 接続サンプルのスクリーンショット](./images/web-screenshot.png)

## 前提条件  

このトピックでは、以下のことを前提としています。

* JavaScript と [AngularJS](https://angularjs.org/) コードについて知識があること。

## 概要

Microsoft Graph API を呼び出すには、次のタスクを完了する必要があります。

1. Azure Active Directory にアプリケーションを登録する
2. JavaScript (ADAL JS) の Azure Active Directory ライブラリを構成する
3. ADAL JS を使用して、アクセス トークンを取得する
4. Microsoft Graph API への要求にアクセス トークンを使用する

<!--<a name="register"></a>-->
## Azure Active Directory にアプリケーションを登録する

Office 365 を使用する前に、アプリケーションを登録し、Microsoft Graph のサービスを使用するアクセス許可を設定する必要があります。数回クリックするだけで、[アプリケーションの登録ツール](https://dev.office.com/app-registration)を使用して、
アプリケーションを登録しユーザーの職場または学校のアカウントにアクセスすることができます。これを管理するには、[Microsoft Azure の管理ポータル](https://manage.windowsazure.com)に移動する必要があります

または、手動でアプリを登録する手順について、[Azure 管理ポータルでブラウザー ベースの Web アプリを登録する](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterWebApp) 記事を参照してください。以下の詳細な点に注意してください。

* **サインオン URL** として、必ず http://127.0.0.1:8080/ を指定してください。
* アプリケーションを登録した後、Angular アプリで必要な[**委任されたアクセス許可**を設定](https://github.com/microsoftgraph/angular-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure)します。接続サンプルには、**サインインしているユーザーとしてメールを送信する**アクセス許可が必要です。

Azure アプリケーションの **[構成]** ページの次の値をメモしてください。Angular アプリで [ADAL JS](https://github.com/AzureAD/azure-activedirectory-library-for-js) を構成するためにこれらの値が必要です。

* クライアント ID (アプリケーションで一意)
* 応答 URL (http://127.0.0.1:8080/)

<!--<a name="adal"></a>-->
## JavaScript (ADAL JS) の Azure Active Directory ライブラリを構成する

[ADAL JS](https://github.com/AzureAD/azure-activedirectory-library-for-js) は、接続サンプルのようなシングルページ アプリケーション (SPA) での Azure AD ユーザーのサインオン、トークン管理やその他の機能を完全にサポートする JavaScript ライブラリです。このライブラリを利用するには、Angular アプリで取り込みと構成を行う必要があります。

Microsoft CDN を使用すると、ライブラリと Angular 固有のモジュールを簡単に取り込めます。

```html
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.7/js/adal.min.js"></script>
<script src="https://secure.aadcdn.microsoftonline-p.com/lib/1.0.7/js/adal-angular.min.js"></script>
```

次に、Angular アプリの依存関係を構成する場所すべてに ADAL JS サービスを構成する必要があります。接続サンプルの構成は、[*public/app.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/scripts/app.js) で行われます。 

ADAL JS を構成するには、モジュールに必須の配列に ```AdalAngular``` を追加し、```adalAuthenticationServiceProvider``` を ```config``` 関数に渡して、まず、ADAL モジュールへの参照を含めます。```init``` 関数を使用してライブラリを構成し、アプリケーションのクライアント ID、および Angular アプリで CORS 要求を作成する必要のある API を宣言する ```endpoints``` オブジェクトを渡します。

```javascript
// Initialize the ADAL provider with your clientID (found in the Azure Management Portal) and 
// the API URL (to enable CORS requests).
adalAuthenticationServiceProvider.init(
  {
    clientId: clientId,
    // The endpoints here are resources for cross origin requests.
    endpoints: {
      'https://graph.microsoft.com': 'https://graph.microsoft.com'
    }
  },
  $httpProvider
);
```

<!--<a name="accessToken"></a>-->
## ADAL JS を使用して、アクセス トークンを取得する

アプリによってサインイン ページにブラウザーをダイレクトし、ユーザーがサインインし、そのアプリケーションにデータへのアクセスを付与できるようにする必要があります。接続サンプルは、ADAL JS を使用してこのタスクを処理します。 

アプリケーションのいずれかのコントローラーで、最初に ```adalAuthenticationService``` をコントローラーに挿入して ADAL サービスへの参照を追加し、UI が呼び出せるサービスの ```login``` 関数を使用する関数を定義します。接続サンプルでは、[*controllers/mainController.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/controllers/mainController.js) ファイルでこれを行います。 

```javascript
/**
  * Expose the login method from ADAL to the view.
  */
function connect() {
  adalAuthenticationService.login();
};
```

この関数が呼び出されると、アプリケーションはユーザーをサインイン ページにリダイレクトします。サインインしてアプリが承認されると、ADAJ JS が取得し格納するクエリ文字列にアクセス トークンが設定されてアプリに戻ります。 

<!--<a name="request"></a>-->
## Microsoft Graph API への要求にアクセス トークンを使用する

アクセス トークンを使用することにより、アプリで、Microsoft Graph API への認証された要求を作成できます。ADAL JS は自動的にすべての HTTP 要求をインターセプトし、それらにアクセス トークンを追加するため、ライブラリを使用するときに手動でヘッダーを設定する必要はありません。 

接続サンプルは、[*controllers/mainController.js*](https://github.com/microsoftgraph/angular-connect-rest-sample/blob/master/public/controllers/mainController.js) ファイルにある Microsoft Graph API の ```me/sendMail``` エンドポイントを使用してメールを送信します。 

Microsoft Graph は、あらゆる種類の Microsoft データとの対話に使用できる、非常に強力な統合 API です。[API リファレンス](http://graph.microsoft.io/docs/api-reference/v1.0)をご覧になり、Microsoft Graph API でほかに何を行うことができるかを調べてください。

