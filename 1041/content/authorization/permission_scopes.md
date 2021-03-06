# Microsoft Graph のアクセス許可スコープ

Microsoft Graph は、アプリケーションがデータに対して持っているアクセス権を制御するために使用されている OAuth 2.0 のアクセス許可スコープを公開します。開発者は、必要とされるアクセス権に適切なアクセス許可スコープを指定してアプリを構成します。通常、Azure ポータルでこれを行います。サインイン時に、ユーザーまたは管理者には、構成したアクセス許可スコープを使用してアプリがデータにアクセスできるようにすることに同意する機会が与えられます。このため、アプリが必要とする最低レベルの特権を提供するアクセス許可スコープを選ぶ必要があります。アプリのアクセス許可を構成する方法と同意プロセスの詳細については、「[アプリケーションを Azure Active Directory と統合する](https://azure.microsoft.com/en-us/documentation/articles/active-directory-integrating-applications/)」をご覧ください。


##アクセス許可スコープの概念

###アプリ専用スコープ vs. 委任されたスコープ
アクセス許可スコープは、アプリ専用スコープまたは委任されたスコープのいずれかに指定できます。アプリ専用スコープ (アプリケーション ロールとも呼ばれます) では、スコープによって提供されるすべての特権がアプリに付与されます。通常、アプリ専用スコープは、サインイン中のユーザーが不在でもサービスとして実行されるアプリによって使用されます。委任されたアクセス許可スコープは、ユーザーがサインインするアプリを対象にしています。これらのスコープでは、サインインしているユーザーの特権がアプリに委任され、サインインしているユーザーのようにアプリが動作できるようになります。アプリに実際に付与される特権は、スコープによって付与された特権とサインインしているユーザーによって所有されている特権の最低限の特権の結合 (交差部分) になります。たとえば、アクセス許可のスコープによって、すべてのディレクトリ オブジェクトの書き込みのための委任された特権が付与されているものの、サインしているユーザーが自分自身のユーザー プロファイルを更新するための特権しか持っていない場合、アプリはサインインしているユーザーのプロファイルの書き込みのみができ、他のオブジェクトの書き込みはできません。

###ユーザーおよびグループのフル プロファイルと基本プロファイル
ユーザーまたはグループのフル プロファイル (またはプロファイル) には、エンティティの宣言されたプロパティがすべて含まれています。プロファイルには機密性の高いディレクトリ情報、または個人情報 (PII) が含まれている可能性があるため、いくつかのスコープは、アプリのアクセスを基本プロファイルと呼ばれる限られたプロパティだけに制限しています。ユーザーの場合、基本プロファイルには次のプロパティのみが含まれます: 表示名、氏名、写真、メール アドレス。グループの場合、基本プロファイルには表示名のみが含まれます。 

<!---   <a name="msg_perm_details"> </a>  -->

##アクセス許可スコープの詳細
Microsoft Graph API リソースにアクセスするための必要なアクセス許可を持つようにアプリを構成する必要があります。読み取り、書き込み、またはその両方の権限に対するアクセス許可が個別のリソースにスコープ指定されます。 

次の表では、Microsoft Graph API のアクセス許可スコープを一覧表示し、各スコープによって付与されるアクセス権を説明します。 
- **スコープ**列にはスコープ名が一覧表示されます。スコープ名の形式は、resource.operation.constraint です。たとえば、Group.ReadWrite.All となります。制約が "All" の場合、スコープはディレクトリですべての指定されたリソース (グループ) を操作 (読み取り書き込み) を実行できる権限をアプリに付与します。それ以外の場合は、スコープはサインインしているユーザーのプロファイルの操作のみを許可します。スコープは、指定された操作に対して制限された権限を与える可能性があります。詳しくは、**説明**の列をご覧ください。
- **アクセス許可**の列には、Azure ポータルでスコープを表示する方法を示します。 
- **説明**の列では、スコープが与える権限の完全なセットについて説明します。委任されたスコープの場合、アプリに実際に付与されるアクセス権は、スコープによって付与されたアクセス権とサインインしているユーザー特権の最低限の特権の結合 (交差部分) になります。 
- スコープはアクセス許可が管理者の同意を必要とするかどうかに基づいてグループ化されます。

  > **注**:`v1.0` および `beta` のアクセス許可スコープの制限事項については、[リリース ノート](http://graph.microsoft.io/docs/overview/release_notes)をご覧ください。
  
###管理者の同意が必要なアクセス許可

|   **範囲**                  |  **Azure 管理ポータルでのアクセス許可**                          |  **説明** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| _User.Read.All_                |     `Read all user's full profiles`           | User.ReadBasic.All とほぼ同じですが、異なる点は、アプリがマネージャーや直属の部下などナビゲーション プロパティを読み取る場合に、組織のすべてのユーザーの完全なプロファイルを読み取ることが可能になることです。完全なプロファイルには、**ユーザー** エンティティの宣言されたプロパティがすべて含まれます。ユーザーがメンバーになっているグループを読み取るためには、アプリにさらに Group.Read.All または Group.ReadWrite.All のいずれかが必要となります。 |
| _User.ReadWrite.All_           |     `Read and write all user's full profiles` | アプリで、サインインしているユーザーの代わりに、組織内の他のユーザーのプロファイル プロパティ、部下、上司の完全なセットを読み書きできるようにします。 |
| _Directory.Read.All_           |     `Read directory data`                     | アプリで、ユーザー、グループ、アプリなどの組織のディレクトリ内のデータを読み取ることができるようにします。 |
| _Directory.ReadWrite.All_      |     `Read and write directory data`           | アプリで、ユーザーやグループなどの組織のディレクトリ内のデータを読み書きできるようにします。ユーザーまたはグループの削除はできません。アプリでユーザーまたはグループの削除や、ユーザー パスワードのリセットはできません。 |
| _Directory.AccessAsUser.All_   |     `Access directory as the signed-in user`  | アプリは、サインインしているユーザーと同じようにディレクトリ内の情報にアクセスできます。|
| _Group.Read.All_ |    `Read all groups` | アプリで、サインインしているユーザーの代わりに、グループを一覧表示し、グループのプロパティとすべてのグループ メンバーシップを読み取ることができるようにします。アプリで、サインインしているユーザーがアクセスできるすべてのグループの予定表、会話、ファイル、およびその他のグループのコンテンツを読み取ることができるようにします。 |
| _Group.ReadWrite.All_ |    `Read and write all groups`| アプリで、サインインしているユーザーの代わりに、グループを作成したり、すべてのグループのプロパティとメンバーシップを読み取ったりできるようにします。さらに、グループの所有者が自身のグループを管理できるよう、またグループ メンバーがグループのコンテンツを更新できるようにします。 |


###管理者の同意が必要でないアクセス許可

|   **範囲**    |  **Azure 管理ポータルでのアクセス許可**   |  **説明** |
|:---------------|:------------------|:-----------------|
| _User.Read_       |    `Sign-in and read user profile` | ユーザーがアプリにサインインできるようにします。またサインインしているユーザーのプロファイルをアプリで読み取ることができるようにします。完全なプロファイルには、ユーザー エンティティの宣言されたプロパティがすべて含まれます。アプリは、上司や直属の部下などのナビゲーション プロパティを読み取ることができません。さらに、サインインしているユーザーの次の基本的な会社情報も (**TenantDetail** オブジェクトを通じて) 読み取ることができるようにします: テナント ID、テナントの表示名、検証済みのドメイン。|
| _User.ReadWrite_ |    `Read and write access to user profile` | アプリはユーザーのプロファイルを読み取ることができます。また、アプリはユーザーの代わりにプロファイル情報を更新することもできます。 |
| _User.ReadBasic.All_ |    `Read all user's basic profiles` | アプリで、サインインしているユーザーの代わりに、組織内のすべてのユーザーの基本的なプロファイルを読み取ることができるようにします。次のプロパティは、ユーザーの基本プロファイルが含まれます: 表示名、氏名、写真、メール アドレス。ユーザーがメンバーになっているグループを読み取るためには、アプリにさらに Group.Read.All または Group.ReadWrite.All が必要となります。| 
| _Mail.Read_ |    `Read user mail` | アプリで、ユーザー メールボックス内のメールを読み取ることができるようにします。  |
| _Mail.ReadWrite_ |    `Read and write access to user mail` | アプリで、ユーザー メールボックス内のメールの作成、読み取り、更新、削除を行えるようにします。メールを送信するためのアクセス許可は含まれません。|
| _Mail.Send_ |    `Send mail as a user` | アプリで、組織内のユーザーとしてメールを送信できるようにします。 |
| _Calendars.Read_ |    `Read user calendars`  | アプリで、ユーザー予定表内のイベントを読み取ることができるようにします。|
| _Calendars.ReadWrite_ |    `Have full access to user calendars`  | アプリで、ユーザー予定表内のイベントの作成、読み取り、更新、削除を行えるようにします。 |
| _Contacts.Read_ |    `Read user contacts`  | アプリは、ユーザーの連絡先を読み取ることができます。 |
| _Contacts.ReadWrite_ |    `Have full access to user contacts`  | アプリでユーザーの連絡先の作成、読み取り、更新、削除を行えるようにします。 |
| _Files.Read_ |    `Read user files and files shared with user` | アプリで、サインインしているユーザーのファイルと、そのユーザーと共有されているファイルを読み取ることができるようにします。| 
| _Files.ReadWrite_ |   `Have full access to user files and files shared with user` | アプリで、サインインしているユーザーのファイルと、そのユーザーと共有されているファイルの読み取り、作成、更新、削除を行えるようにします。 |
| _Files.ReadWrite.Selected_ |    `Read and write files that the user selects` | アプリでユーザーが選ぶファイルを読み書きできるようにします。ユーザーがファイルを選んだ後、アプリで数時間アクセスできます。 |
| _Files.Read.Selected_ |    `Read files that the user selects`  | アプリでユーザーが選ぶファイルを読み取ることができるようにします。ユーザーがファイルを選んだ後、アプリで数時間アクセスできます。 |
| _Sites.Read.All_ |    `Read items in all site collections` | アプリケーションで、サインインしているユーザーの代わりに、すべてのサイト コレクションにあるドキュメントおよびリスト アイテムを読み取ることができるようにします。 |
| _openid_ |    `Sign users in` (プレビュー) | ユーザーが職場または学校アカウントでアプリにサインインできるようにします。またアプリで、ユーザーの基本的なプロファイル情報を読み取れるようにします。|
| _offline_access_ |    `Access user's data anytime` (プレビュー) | 現在アプリを使用していない場合も、アプリでユーザー データの読み取りと更新ができるようにします。|

###管理者の同意が必要なアプリ専用のアクセス許可

|   **範囲**    |  **Azure 管理ポータルでのアクセス許可**   |  **説明** |
|:---------------|:------------------|:-----------------|
| _Mail.Read_       |    `Read mail in all mailboxes` | サインインしているユーザーなしで、アプリですべてのメールボックス内のメールを読み取れるようにします。|
| _Mail.ReadWrite_ |    `Read and write mail in all mailboxes` | サインインしているユーザーなしで、アプリですべてのメールボックス内のメールの作成、読み取り、更新、削除を行えるようにします。メールを送信するためのアクセス許可は含まれません。 |
| _Mail.Send_ |    `Send mail as any user` | サインインしているユーザーなしで、アプリで任意のユーザーとしてメールを送信できるようにします。 | 
| _Calendars.Read_ |    `Read calendars in all mailboxes` | アプリは、サインインしているユーザーなしで、すべての予定表のイベントを読み取ることができます。 |
| _Calendars.ReadWrite_ |    `Read and write calendars in all mailboxes` | アプリは、サインインしているユーザーなしで、すべての予定表のイベントを作成、読み取り、更新、および削除できます。|
| _Contacts.Read_ |    `Read contacts in all mailboxes` | アプリは、サインインしているユーザーなしで、すべてのメールボックス内のすべての連絡先を読み取ることができます。 |
| _Contacts.ReadWrite_ |    `Read and write contacts in all mailboxes`  |アプリは、サインインしているユーザーなしで、すべてのメールボックス内のすべての連絡先を作成、読み取り、更新、および削除できます。|
| _User.ReadBasic.All_ |    `Read all users' basic profiles`  | サインインしているユーザーなしで、アプリで組織内の他のユーザーのプロファイル プロパティの基本的なセットを読み取ることができるようにします。表示名、氏名、写真、勤務時間外メッセージが含まれます。|
| _User.Read.All_ |    `Read all users' full profiles` | サインインしているユーザーなしで、アプリで組織内の他のユーザーのプロファイル プロパティ、グループ メンバーシップ、部下、上司の完全なセットを読み取ることができるようにします。| 
| _User.ReadWrite.All_ |   `Read and write all users' full profiles` | サインインしているユーザーなしで、アプリで組織内の他のユーザーのプロファイル プロパティ、グループ メンバーシップ、部下、上司の完全なセットを読み書きできるようにします。|


##プレビュー
###管理者の同意が必要でないアクセス許可 (プレビュー)

|   **範囲**    |  **Azure 管理ポータルでのアクセス許可**   |  **説明** |
|:---------------|:------------------|:-----------------|
| _Tasks.ReadWrite_ |    `Create, read, update and delete user tasks and plans` (プレビュー) | アプリで、サインインしているユーザーに割り当てられている、またはサインしているユーザーと共有しているタスクとプラン (およびプラン内のタスク) の作成、読み取り、更新、削除を行えるようにします。|
| _People.Read_ |    `Read users' relevant people lists` (プレビュー) | アプリで、サインインしているユーザーの関連する人に関するランク付けされたリストを読み取れるようにします。リストには、個人の連絡先、ソーシャル ネットワーキングの連絡先、組織のディレクトリ、最近 (メール、Skype などで) 通信した人が含まれます。|
| _People.ReadWrite_ |    `Read and write users' relevant people lists` (プレビュー) | アプリで、サインインしているユーザーの関連する人に関するランク付けされたリストを読み書きできるようにします。リストには、個人の連絡先、ソーシャル ネットワーキングの連絡先、組織のディレクトリ、最近 (メール、Skype などで) 通信した人が含まれます。|
| _Notes.Create_ |    `Create pages in users' notebooks` (プレビュー) | アプリで、サインインしているユーザーの代わりに、ノートブックとセクションのタイトルの読み取りと、新しいページ、ノートブック、セクションの作成を行えるようにします。|
| _Notes.ReadWrite.CreatedByApp_ |    `Limited notebook access` (プレビュー) | アプリで、サインインしているユーザーの代わりに、ノートブックとセクションのタイトルの読み取りと、新しいページの作成を行えるようにします。また、アプリによって作成されたページの読み取りと更新をアプリで行えるようにもします。 |
| _Notes.Read_ |    `Read user notebooks` (プレビュー) | サインインしているユーザーの代わりに、アプリで、OneNote のノートブックとセクションのタイトルの表示と、すべてのページの読み取りを行えるようにします。パスワードで保護されたセクションを表示することはできません。 |
| _Notes.ReadWrite_ |    `Read and write user notebooks` (プレビュー) | サインインしているユーザーの代わりに、アプリで、ノートブックとセクションのタイトルの読み取り、すべてのページの読み取りと書き込み、および新しいページの作成を行えるようにします。 パスワードで保護されたセクションにアクセスすることはできません。 |
| _Notes.Read.All_ |    `Read all notebooks that the user can access` (プレビュー) | アプリで、サインしているユーザーがアクセスできるすべてのノートブックとセクションのコンテンツを読み取れるようにします。 パスワードで保護されたセクションを読み取ることはできません。 |
| _Notes.ReadWrite.All_ |    `Read and write notebooks that the user can access` (プレビュー) | アプリで、サインしているユーザーがアクセスできるすべてのノートブックとセクションのコンテンツを読み書きできるようにします。パスワードで保護されたセクションにアクセスすることはできません。|


<!-- -->

##アクセス許可スコープのシナリオ
`User` と `Group` のリソース、およびそれらに対応する必須スコープを使用したアプリのシナリオを次にいくつか示します。次の表に、アプリが特定の操作を実行するために必要なアクセス許可スコープを示します。次の点にご注意ください。場合によっては、いくつかの操作を実行するためのアプリの権限はアクセス許可スコープがアプリ専用であるか、または委任されたものであるかによって異なります。また委任されたアクセス許可スコープの場合は、サインインしているユーザーの特権に応じて異なります。 

###ユーザー リソースと必須スコープを使用したアクセス シナリオ

| **ユーザーが関与するアプリ タスク**   |  **必要なスコープ** | **Azure 管理ポータルでのアクセス許可** |
|:-------------------------------|:---------------------|:---------------|
| アプリの目的は、人材選択画面への表示などのために、他のユーザーの基本情報 (表示名と写真のみ) を読み取ること   | _User.ReadBasic.All_  |  `Read all user's basic profiles` |
| アプリの目的は、サインインしているユーザー (直属の部下や上司などを参照) の完全なユーザー プロファイルを読み取ること  | _User.Read_ | `Enable sign-in and read user profile`|
| アプリの目的は、すべてのユーザーの完全なユーザー プロファイルを読み取ること  | _User.Read.All_ |  `Read all user's full profiles`   |
| アプリの目的は、サインインしているユーザーのファイル、メール、および予定表情報を読み取ること  | _User.Read_, _Files.Read_, _Mail.Read_, _Calendar.Read_ | `Enable sign-in and read user profile`, `Read users' files`,  `Read user mail`,  `Read user calendars` |
| アプリの目的は、サインインしているユーザーの (マイ) ファイルと他のユーザーがサインインしているユーザー (自分) と共有しているファイルを読み取ること | _User.Read_, _Files.Read_, _Sites.Read.All_ | `Enable sign-in and read user profile`, `Read users' files`,  `Read items in all site collections` |
| アプリの目的は、サインインしているユーザーの完全なユーザー プロファイルを読み書きすること   | _User.ReadWrite_ | `Read and write access to user profile` |
| アプリの目的は、すべてのユーザーの完全なユーザー プロファイルを読み書きすること    | _User.ReadWrite.All_ | `Read and write all user's full profiles` |
| アプリの目的は、サインインしているユーザーのファイル、メール、および予定表情報を読み書きすること    | _User.ReadWrite_, _Files.ReadWrite_, _Mail.ReadWrite_, _Calendar.ReadWrite_  |  `Read and write access to user profile`,  `Read and write access to user profile`,  `Read and write access to user mail`, `Have full access to user calendars` |
   

###グループ リソースと必須スコープを使用したアクセス シナリオ
    
| **グループが関与するアプリ タスク**  |  **必要なスコープ** |  **Azure 管理ポータルでのアクセス許可** |
|:-------------------------------|:---------------------|:---------------|
| アプリの目的は、グループ選択画面への表示などのために、基本グループ情報 (表示名と写真のみ) を読み取ること  | _Group.Read.All_  | `Read all groups`|
| アプリの目的は、ファイルや会話を含むすべての統合グループ内のすべてのコンテンツを読み取ること。グループ メンバーシップを表示したり、グループ メンバーシップを更新できたり (所有者の場合) する必要もあります。  |  _Group.Read.All_ | `Read items in all site collections`, `Read all groups`|
| アプリの目的は、ファイルや会話を含むすべての統合グループ内のすべてのコンテンツを読み書きすること。グループ メンバーシップを表示したり、グループ メンバーシップを更新できたり (所有者の場合) する必要もあります。  |  _Group.ReadWrite.All_, _Sites.ReadWrite.All_ |  `Read and write all groups`, `Edit or delete items in all site collections` |
| アプリの目的は、統合グループを検出 (検索) すること。ユーザーに特定のグループの検索と一覧表から選択できるようにし、グループに参加できるようにします。     | _Group.ReadWrite.All_ | `Read and write all groups`|
| アプリの目的は、AAD グラフを経由してグループを作成すること |   _Group.ReadWrite.All_ | `Read and write all groups`|
 

<!---   ##Additional Resources##


- [Authenticate Microsoft Graph endpoints using the v2.0 app model preview](authenticate-MSGraph-using-v2.md )
- [Hands on lab: Deep dive into the Office 365 unified API](http://dev.office.com/hands-on-labs/4585) -->

