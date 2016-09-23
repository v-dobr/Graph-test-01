


# Microsoft Graph の概要

Microsoft Graph (以前は Office 365 統合 API と呼ばれていた) は 1 つの REST API エンドポイント (**https://graph.microsoft.com**) を介して複数の API を Microsoft クラウド サービスから公開するものです。Microsoft Graph を使用すれば、以前は難しかった、あるいは複雑だったクエリを単純なナビゲーションに変えることができます。 
 
Microsoft Graph で次のことが可能になります。

- 複数の Microsoft クラウド サービスから、集約されたデータに 1 つの応答でアクセスする統一された API エンドポイント 
- エンティティおよびエンティティ間のリレーションシップのシームレスなナビゲーション 
- Microsoft クラウドからもたらされるインテリジェンスとインサイトへのアクセス

これらすべてで単一の認証トークンを使用。

Outlook、OneDrive、Azure Active Directory、Planner、OneNote などのサービスからの、ユーザー、グループ、メール、メッセージ、予定表、タスク、ノートのような固定エンティティにアクセスするための API を使用することができます。いっしょに作業しているユーザーの一覧または周囲を流れているドキュメントのような計算された関係を、Office Graph で取得することもできます (商用ユーザーの場合のみ)。

Microsoft Graph では、2 つのエンドポイントを公開します。一般に利用可能なエンドポイントの /v1.0 と、プレビューのエンドポイントの /beta です。実稼働アプリケーションでは、/beta ではなく /v1.0 を使用できます。プレビューのエンドポイントの /beta では、実験し、フィードバックを提供する開発者のために最新の機能を提供します。ベータの API は任意の時点で変更される可能性があり、運用環境で使用の準備はできていません。

<!--<a name="msg_queries"> </a>-->

##共通クエリ

次に、Microsoft Graph API を使用した一般的なクエリの例をいくつか示します。

| **Operation** | **サービス エンドポイント** |
|:--------------------------|:----------------------------------------|
|   自分のプロファイルの取得 |    `https://graph.microsoft.com/v1.0/me` |
|   自分のファイルの取得|   `https://graph.microsoft.com/v1.0/me/drive/root/children` |
|   自分の写真の取得	     | `https://graph.microsoft.com/v1.0/me/photo/$value` |
|   自分のメールの取得 |   `https://graph.microsoft.com/v1.0/me/messages` |
|   自分にとって重要度の高いメールの取得 | `https://graph.microsoft.com/v1.0/me/messages?$filter=importance%20eq%20'high'` |
|   自分の予定表の取得 |   `https://graph.microsoft.com/v1.0/me/calendar` |
|   自分の上司の取得	  | `https://graph.microsoft.com/v1.0/me/manager` |
|   foo.txt ファイルを最後に変更したユーザーの取得 |  `https://graph.microsoft.com/v1.0/me/drive/root/children/foo.txt/lastModifiedByUser` |
|   自分がメンバーになっている統合グループの取得|   `https://graph.microsoft.com/v1.0/me/memberOf/$/microsoft.graph.group?$filter=groupTypes/any(a:a%20eq%20'unified')` |
|   自分の所属組織のユーザーの取得	     | `https://graph.microsoft.com/v1.0/users` |
|   グループ会話の取得 |   `https://graph.microsoft.com/v1.0/groups/<id>/conversations` |
|   自分に関連付けられたユーザーの取得    | `https://graph.microsoft.com/beta/me/people` |
|   自分の周りで人気上昇中のファイルの取得 |  `https://graph.microsoft.com/beta/me/trendingAround` |
|   一緒に作業しているユーザーの取得     | `https://graph.microsoft.com/beta/me/workingWith` |
|   自分のタスクの取得    | `https://graph.microsoft.com/beta/me/tasks` |
|   自分のノートの取得 |  `https://graph.microsoft.com/beta/me/notes/notebooks` |

<!-- <a name="msg_roof"> </a> -->

## すべての Office 365 データを 1 つ屋根の下に

Microsoft Graph 開発者スタックとそのしくみを次の図に示します。

![Microsoft Graph API 開発者のスタック。](./images/MicrosoftGraph_DevStack.png)

 >  お客様からのフィードバックを重視しています。[Stack Overflow](http://stackoverflow.com/questions/tagged/office365+or+microsoftgraph)でご連絡いただけます。質問には [MicrosoftGraph] と [office365] でタグ付けしてください。



