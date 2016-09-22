# Escopos de permissão do Microsoft Graph

O Microsoft Graph expõe os escopos de permissão do OAuth 2.0 que são usados para controlar o acesso que um aplicativo tem aos dados. Como desenvolvedor, você configura seu aplicativo com os escopos de permissão apropriados para o acesso necessário. Geralmente, você faz isso por meio do portal do Azure. Ao entrar, os usuários ou administradores recebem a oportunidade de consentir em permitir o acesso do aplicativo aos seus dados com os escopos de permissão que você configurou. Por esse motivo, você deve escolher os escopos de permissão que fornecem o menor nível de privilégios exigido por seu aplicativo. Para mais detalhes sobre como configurar permissões para seu aplicativo e sobre o processo de consentimento, confira [Integrar Aplicativos com o Azure Active Directory](https://azure.microsoft.com/en-us/documentation/articles/active-directory-integrating-applications/).


##Conceitos de escopo de permissão

###Escopos somente do aplicativo versus delegados
Os escopos de permissão podem ser somente do aplicativo ou delegado. Os escopos somente do aplicativo (também conhecidos como funções do aplicativo) concedem ao aplicativo o conjunto completo de privilégios oferecidos pelo escopo. Os escopos somente do aplicativo são, geralmente, usados por aplicativos executados como um serviço, sem a presença de um usuário conectado. Os escopos de permissão delegados são para aplicativos aos quais um usuário se conecta. Esses escopos delegam privilégios do usuário conectado ao aplicativo, permitindo que o aplicativo haja como o usuário conectado. Os reais privilégios concedidos ao aplicativo serão a combinação menos privilegiada (a interseção) dos privilégios concedidos pelo escopo e aqueles que pertencem ao usuário conectado. Por exemplo, se o escopo de permissão conceder privilégios delegados para gravar todos os objetos do diretório, mas o usuário conectado tiver privilégios apenas para atualizar seu próprio perfil de usuário, o aplicativo só poderá gravar o perfil do usuário conectado, mas não outros objetos.

###Perfis completos e básicos para usuários e grupos
O perfil completo (ou perfil) de um Usuário ou de um Grupo inclui todas as propriedades declaradas da entidade. Como o perfil pode conter informações confidenciais do diretório ou informações de identificação pessoal (PII), vários escopos restringem o acesso do aplicativo a um conjunto limitado de propriedades, conhecidas como um perfil básico. Para os usuários, o perfil básico inclui apenas as seguintes propriedades: nome de exibição, nome e sobrenome, foto e endereço de email. Para grupos, o perfil básico contém apenas o nome de exibição. 

<!---   <a name="msg_perm_details"> </a>  -->

##Detalhes do escopo de permissão
Você deve configurar seu aplicativo para ter as permissões necessárias para acessar os recursos da API do Microsoft Graph. As permissões têm como escopo recursos individuais para os direitos de ler, de gravar ou ambos. 

As tabelas a seguir listam os escopos de permissão da API do Microsoft Graph e explicam o acesso concedido por cada um. 
- A coluna **Escopo** lista o nome do escopo. Nomes de escopos têm o formato recurso.operação.restrição. Por exemplo, Group.ReadWrite.All. Se a restrição for "All", o escopo concederá ao aplicativo a possibilidade de realizar a operação (ReadWrite) em todos os recursos especificados (Group) no diretório; caso contrário, o escopo só permitirá a operação no perfil do usuário conectado. Os escopos podem conceder privilégios limitados para a operação especificada, confira a coluna **Descrição** para obter detalhes.
- A coluna **Permissão** mostra como o escopo é exibido no portal do Azure. 
- A coluna **Descrição** descreve o conjunto completo de privilégios concedidos pelo escopo. Para escopos delegados, o acesso real concedido ao aplicativo será a combinação menos privilegiada (interseção) do acesso concedido pelo escopo e aqueles que pertencem ao usuário conectado. 
- Os escopos são agrupados dependendo se as permissões exigem consentimento do administrador.

  > **Observação**: confira [Notas de versão](http://graph.microsoft.io/docs/overview/release_notes) para saber as limitações do escopo de permissão de `v1.0` e `beta`.
  
###Permissões que exigem o consentimento do administrador

|   **Scope**                  |  **Permissão no Portal de Gerenciamento do Azure**                          |  **Descrição** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| _User.Read.All_                |     `Read all user's full profiles`           | Igual a User.ReadBasic.All, exceto por permitir que o aplicativo leia o perfil completo de todos os usuários na organização e ao ler as propriedades de navegação como gerente e subordinado direto. O perfil completo inclui todas as propriedades declaradas da entidade **Usuário**. Para ler os grupos dos quais um usuário é membro, o aplicativo também exigirá Group.Read.All ou Group.ReadWrite.All. |
| _User.ReadWrite.All_           |     `Read and write all user's full profiles` | Permite ao aplicativo ler e gravar o conjunto completo de propriedades do perfil, relatórios e gerenciadores de outros usuários na sua organização, em nome do usuário conectado. |
| _Directory.Read.All_           |     `Read directory data`                     | Permite ao aplicativo ler dados no diretório da sua organização, como usuários, grupos e aplicativos. |
| _Directory.ReadWrite.All_      |     `Read and write directory data`           | Permite ao aplicativo ler e gravar dados no diretório da sua organização, como usuários e grupos.  Não permite a exclusão de usuários ou grupos. Não permite ao aplicativo excluir usuários ou grupos, ou redefinir senhas de usuário. |
| _Directory.AccessAsUser.All_   |     `Access directory as the signed-in user`  | Permite ao aplicativo ter o mesmo acesso que o usuário conectado a informações no diretório.|
| _Group.Read.All_ |    `Read all groups` | Permite ao aplicativo listar grupos, e ler suas propriedades e todas as associações do grupo em nome do usuário conectado.  Também permite ao aplicativo ler calendário, conversas, arquivos e outros tipos de conteúdo de todos os grupos que o usuário conectado pode acessar. |
| _Group.ReadWrite.All_ |    `Read and write all groups`| Permite ao aplicativo criar grupos e ler todas as propriedades e associações do grupo em nome do usuário conectado.  Além disso, permite aos proprietários do grupo gerenciar seus grupos, e permite aos membros do grupo atualizar o conteúdo do grupo. |


###Permissões que não exigem o consentimento do administrador

|   **Scope**    |  **Permissão no Portal de Gerenciamento do Azure**   |  **Descrição** |
|:---------------|:------------------|:-----------------|
| _User.Read_       |    `Sign-in and read user profile` | Permite aos usuários entrar no aplicativo e permite ao aplicativo ler o perfil de usuários conectados. O perfil completo inclui todas as propriedades declaradas da entidade Usuário. O aplicativo não pode ler as propriedades de navegação, como gerente ou subordinados diretos. Além disso, permite ao aplicativo ler as seguintes informações básicas da empresa do usuário conectado (por meio do objeto **TenantDetail**): ID de locatário, nome de exibição do locatário e domínios verificados.|
| _User.ReadWrite_ |    `Read and write access to user profile` | Permite ao aplicativo ler o seu perfil. Ele também permite ao aplicativo atualizar suas informações de perfil em seu nome. |
| _User.ReadBasic.All_ |    `Read all user's basic profiles` | Permite ao aplicativo ler o perfil básico de todos os usuários da organização em nome do usuário conectado. As seguintes propriedades compõem um perfil básico de usuário: nome de exibição, nome e sobrenome, foto e endereço de email. Para ler os grupos dos quais um usuário é membro, o aplicativo também exigirá Group.Read.All ou Group.ReadWrite.All.| 
| _Mail.Read_ |    `Read user mail` | Permite ao aplicativo ler emails em caixas de correio do usuário.  |
| _Mail.ReadWrite_ |    `Read and write access to user mail` | Permite ao aplicativo criar, ler, atualizar e excluir emails em caixas de correio do usuário. Não inclui a permissão para enviar emails.|
| _Mail.Send_ |    `Send mail as a user` | Permite ao aplicativo enviar emails como usuários na organização.  |
| _Calendars.Read_ |    `Read user calendars`  | Permite ao aplicativo ler eventos nos calendários do usuário.|
| _Calendars.ReadWrite_ |    `Have full access to user calendars`  | Permite ao aplicativo criar, ler, atualizar e excluir eventos em calendários do usuário. |
| _Contacts.Read_ |    `Read user contacts`  | Permite ao aplicativo ler os contatos do usuário. |
| _Contacts.ReadWrite_ |    `Have full access to user contacts`  | Permite ao aplicativo criar, ler, atualizar e excluir contatos do usuário. |
| _Files.Read_ |    `Read user files and files shared with user` | Permite ao aplicativo ler os arquivos do usuário conectado e os arquivos compartilhados com o usuário.| 
| _Files.ReadWrite_ |   `Have full access to user files and files shared with user` | Permite ao aplicativo ler, criar, atualizar e excluir os arquivos do usuário conectado e os arquivos compartilhados com o usuário. |
| _Files.ReadWrite.Selected_ |    `Read and write files that the user selects` | Permite ao aplicativo ler e gravar arquivos selecionados pelo usuário. O aplicativo tem acesso por várias horas depois que o usuário seleciona um arquivo. |
| _Files.Read.Selected_ |    `Read files that the user selects`  | Permite ao aplicativo ler arquivos selecionados pelo usuário. O aplicativo tem acesso por várias horas depois que o usuário seleciona um arquivo. |
| _Sites.Read.All_ |    `Read items in all site collections` | Permite ao aplicativo ler documentos e listar itens em todos os conjuntos de sites em nome do usuário conectado. |
| _openid_ |    `Sign users in`(visualização) | Permite aos usuários entrar no aplicativo com contas corporativas ou de estudante e permite ao aplicativo ver informações básicas do perfil do usuário.|
| _offline_access_ |    `Access user's data anytime` (visualização) | Permite ao aplicativo ler e atualizar dados do usuário, mesmo quando eles não estiver usando o aplicativo.|

###Permissões somente do aplicativo exigindo o consentimento do administrador

|   **Scope**    |  **Permissão no Portal de Gerenciamento do Azure**   |  **Descrição** |
|:---------------|:------------------|:-----------------|
| _Mail.Read_       |    `Read mail in all mailboxes` | Permite ao aplicativo ler emails em todas as caixas de correio sem um usuário conectado.|
| _Mail.ReadWrite_ |    `Read and write mail in all mailboxes` | Permite ao aplicativo criar, ler, atualizar e excluir emails em todas as caixas de correio sem um usuário conectado. Não inclui a permissão para enviar emails. |
| _Mail.Send_ |    `Send mail as any user` | Permite ao aplicativo enviar emails como qualquer usuário sem um usuário conectado. | 
| _Calendars.Read_ |    `Read calendars in all mailboxes` | Permite ao aplicativo ler eventos de todos os calendários sem um usuário conectado. |
| _Calendars.ReadWrite_ |    `Read and write calendars in all mailboxes` | Permite ao aplicativo criar, ler, atualizar e excluir eventos de todos os calendários sem um usuário conectado.|
| _Contacts.Read_ |    `Read contacts in all mailboxes` | Permite ao aplicativo ler todos os contatos em todas as caixas de correio sem um usuário conectado. |
| _Contacts.ReadWrite_ |    `Read and write contacts in all mailboxes`  |Permite ao aplicativo criar, ler, atualizar e excluir todos os contatos em todas as caixas de correio sem um usuário conectado.|
| _User.ReadBasic.All_ |    `Read all users' basic profiles`  | Permite ao aplicativo ler um conjunto básico de propriedades de perfil de outros usuários em sua organização sem um usuário conectado. Inclui o nome para exibição, nome e sobrenome, foto e mensagem de ausência temporária.|
| _User.Read.All_ |    `Read all users' full profiles` | Permite ao aplicativo ler o conjunto completo de propriedades do perfil, relatórios e gerenciadores de outros usuários na sua organização, sem um usuário conectado.| 
| _User.ReadWrite.All_ |   `Read and write all users' full profiles` | Permite ao aplicativo ler e gravar o conjunto completo de propriedades do perfil, relatórios e gerenciadores de outros usuários na sua organização, sem um usuário conectado.|


##Visualização
###Permissões que não exigem o consentimento do administrador (visualização)

|   **Scope**    |  **Permissão no Portal de Gerenciamento do Azure**   |  **Descrição** |
|:---------------|:------------------|:-----------------|
| _Tasks.ReadWrite_ |    `Create, read, update and delete user tasks and plans` (visualização) | Permite ao aplicativo criar, ler, atualizar e excluir tarefas e planos (e tarefas neles), que são atribuídos ou compartilhado com o usuário conectado.|
| _People.Read_ |    `Read users' relevant people lists`(visualização) | Permite ao aplicativo ler uma lista classificada de pessoas relevantes do usuário conectado. A lista inclui contatos locais, contatos das redes sociais, diretório da sua organização e as pessoas de comunicações recentes (como emails e Skype).|
| _People.ReadWrite_ |    `Read and write users' relevant people lists`(visualização) | Permite ao aplicativo criar, ler e gravar na lista classificada de pessoas relevantes do usuário conectado. A lista inclui contatos locais, contatos das redes sociais, diretório da sua organização e as pessoas de comunicações recentes (como emails e Skype).|
| _Notes.Create_ |    `Create pages in users' notebooks` (visualização) | Permite ao aplicativo ler os títulos dos blocos de anotações e seções, e criar novas páginas, blocos de anotações e seções em nome do usuário conectado.|
| _Notes.ReadWrite.CreatedByApp_ |    `Limited notebook access` (visualização) | Permite ao aplicativo ler os títulos de blocos de anotações e seções, criar novas páginas em nome do usuário conectado. Também permite ao aplicativo ler e atualizar as páginas criadas pelo aplicativo. |
| _Notes.Read_ |    `Read user notebooks` (visualização) | Permite ao aplicativo exibir os títulos dos blocos de anotações e seções do OneNote, e ler todas as páginas em nome do usuário conectado. Não pode exibir seções protegidas por senha. |
| _Notes.ReadWrite_ |    `Read and write user notebooks` (visualização) | Permite ao aplicativo ler os títulos dos blocos de anotações e seções, ler todas as páginas, gravar todas as páginas e criar novas páginas em nome do usuário conectado.  Não pode acessar seções protegidas por senha. |
| _Notes.Read.All_ |    `Read all notebooks that the user can access` (visualização) | Permite ao aplicativo ler os conteúdos de todos os blocos de anotações e seções que o usuário conectado pode acessar.   Não pode ler seções protegidas por senha. |
| _Notes.ReadWrite.All_ |    `Read and write notebooks that the user can access` (visualização) | Permite ao aplicativo ler e gravar os conteúdos de todos os blocos de anotações e seções que o usuário conectado pode acessar.  Não pode acessar seções protegidas por senha.|


<!-- -->

##Cenários de escopo de permissão
A seguir, alguns cenários de aplicativo usando os recursos `User` e `Group` e seus escopos necessários correspondentes. A tabela a seguir mostra os escopos de permissão necessários para que um aplicativo seja capaz de executar operações específicas. Observe que, em alguns casos, a capacidade do aplicativo de executar algumas operações dependerá de se o escopo de permissão é somente do aplicativo ou delegado, e, no caso de escopos de permissão delegada, dos privilégios do usuário conectado. 

###Acessar cenários usando o recurso Usuário e os escopos necessários

| **Tarefas do aplicativo envolvendo o Usuário**   |  **Escopos necessários** | **Permissões no Portal de Gerenciamento do Azure** |
|:-------------------------------|:---------------------|:---------------|
| O aplicativo deseja ler as informações básicas de outros usuários (somente o nome para exibição e a imagem), por exemplo, para mostrar uma experiência de seleção de pessoas   | _User.ReadBasic.All_  |  `Read all user's basic profiles` |
| O aplicativo deseja ler o perfil completo do usuário de um usuário conectado (ver subordinados diretos, gerente etc.)  | _User.Read_ | `Enable sign-in and read user profile`|
| O aplicativo deseja ler o perfil completo de todos os usuários  | _User.Read.All_ |  `Read all user's full profiles`   |
| O aplicativo deseja ler informações de arquivos, email e calendário do usuário conectado  | _User.Read_, _Files.Read_, _Mail.Read_, _Calendar.Read_ | `Enable sign-in and read user profile`, `Read users' files`,  `Read user mail`,  `Read user calendars` |
| O aplicativo deseja ler os arquivos dos usuários (meus) conectados e os arquivos que outros usuários compartilharam com o usuário conectado (eu). | _User.Read_, _Files.Read_, _Sites.Read.All_ | `Enable sign-in and read user profile`, `Read users' files`,  `Read items in all site collections` |
| O aplicativo deseja ler e gravar o perfil completo do usuário conectado   | _User.ReadWrite_ | `Read and write access to user profile` |
| O aplicativo deseja ler e gravar o perfil completo de todos os usuários    | _User.ReadWrite.All_ | `Read and write all user's full profiles` |
| O aplicativo deseja ler e gravar informações de arquivos, de email e de calendário do usuário conectado    | _User.ReadWrite_, _Files.ReadWrite_, _Mail.ReadWrite_, _Calendar.ReadWrite_  |  `Read and write access to user profile`,  `Read and write access to user profile`,  `Read and write access to user mail`, `Have full access to user calendars` |
   

###Acessar cenários usando o recurso Grupo e os escopos necessários
    
| **Tarefas do aplicativo envolvendo o Grupo**  |  **Escopos necessários** |  **Permissões no Portal de Gerenciamento do Azure** |
|:-------------------------------|:---------------------|:---------------|
| O aplicativo deseja ler as informações básicas do grupo (somente o nome para exibição e a imagem), por exemplo, para mostrar uma experiência de seleção de um grupo  | _Group.Read.All_  | `Read all groups`|
| O aplicativo deseja ler todo o conteúdo em todos os grupos unificados, incluindo arquivos, conversas.  Também precisa mostrar associações de grupo, ser capaz de atualizar associações de grupo (caso seja o proprietário).  |  _Group.Read.All_ | `Read items in all site collections`, `Read all groups`|
| O aplicativo deseja ler e gravar todo o conteúdo em todos os grupos unificados, incluindo arquivos, conversas.  Também precisa mostrar associações de grupo, ser capaz de atualizar associações de grupo (caso seja o proprietário).  |  _Group.ReadWrite.All_, _Sites.ReadWrite.All_ |  `Read and write all groups`, `Edit or delete items in all site collections` |
| O aplicativo deseja descobrir (localizar) um grupo unificado. Permite ao usuário procurar um grupo específico e escolher um deles na lista enumerada para ingressar no grupo.     | _Group.ReadWrite.All_ | `Read and write all groups`|
| O aplicativo deseja criar um grupo por meio do AAD Graph |   _Group.ReadWrite.All_ | `Read and write all groups`|
 

<!---   ##Additional Resources##


- [Authenticate Microsoft Graph endpoints using the v2.0 app model preview](authenticate-MSGraph-using-v2.md )
- [Hands on lab: Deep dive into the Office 365 unified API](http://dev.office.com/hands-on-labs/4585) -->

