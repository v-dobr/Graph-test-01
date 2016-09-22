
# Perguntas frequentes sobre o Microsoft Graph

## Quais plataformas são compatíveis com a API do Microsoft Graph?
<!--
Apps can use the Microsoft Graph API to perform create, read, update, and delete (CRUD) operations on data sources and entities, giving them seamless access to work data. 

**Ease of use--one endpoint, all Office 365 data under one roof**

You can use the API in four steps:
1.  Select your programming language and development environment.
2.  Build your app.
3.  Optionally, host your app in Microsoft Azure or any cloud platform you choose.
4.  Authenticate your users by using single sign-on with Azure AD.

As a developer you can use the API to create custom apps that access and interact with all the richness of enterprise and productivity data--users, groups, organizational contacts, files, folders, mail, calendar, insights and relationships--and build apps across all mobile, web, and desktop platforms. No matter your development platform or tools. Using a single service endpoint to access those entities and data. And a single authentication flow.  -->

Você pode:

<!--Just like in Office 365 APIs, Office 365 unified endpoint API  allows you to build apps using any development environment of your choice:  -->

- Usar qualquer ambiente de desenvolvimento com o qual você esteja familiarizado, como .NET, PHP, Java, Python ou Ruby
- Usar qualquer linguagem de programação, plataforma de desenvolvimento e ambiente de hospedagem
- Compilar um aplicativo que acesse a API usando qualquer linguagem da Web, como JavaScript, HTML5, Python, Ruby, PHP e ASP.NET  
- Usar o IDE que preferir, seja ele o Visual Studio, o Eclipse, o Android Studio, o Xcode ou outro
- Hospedar seus aplicativos no Microsoft Azure ou em qualquer plataforma de nuvem
- Desenvolver aplicativos para o Windows Universal, o iOS, o Android ou em outra plataforma do dispositivo
- Chamar a API a partir dos Suplementos do Office (anteriormente conhecidos como aplicativos do Office) ou dos Suplementos do SharePoint (anteriormente conhecidos como aplicativos para SharePoint)
 


## Por que usar a API do Microsoft Graph?

Digamos que você queira recuperar programaticamente os arquivos de um usuário e a imagem de perfil e localizar o gerente da pessoa que editou pela última vez esse arquivo em sua organização. Como as informações estão armazenadas em diversos serviços (Azure Active Directory, SharePoint e Exchange), a tarefa envolve várias etapas usando as APIs do Office 365: 

1. Usar o Serviço de Descoberta para encontrar os vários pontos de extremidade de serviço 
2. Determinar a URL dos serviços aos quais seus aplicativos do Office 365 desejam se conectar
3. Em seguida, adquirir e gerenciar o token de acesso para cada serviço e fazer a solicitação para o serviço diretamente

Agora, você pode usar a API do Microsoft Graph para executar a mesma operação complexa por meio de um único ponto de extremidade da API REST. Você não precisa descobrir e navegar em um ponto de extremidade diferente para cada serviço, adquirir e gerenciar um token de acesso separado para cada serviço, lidar com serviços em silos e modelo de dados variável.

##Consultas de exemplo

O exemplo a seguir mostra o modelo atual de interação com a API do Office 365 usando pontos de extremidade de serviço distintos e como isso fica mais simples com a API do Microsoft Graph.

**Pontos de extremidade de serviço distintos**

|   **Operação**                  |  **API**                          |  **Ponto de extremidade de serviço** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| Descobrir pontos de extremidade de serviço para a API do Office 365               |     `Discovery Service`           | _https://_**api.office.com**_/discovery/v1.0/me/services_ |
| Obter usuários           |     `Azure AD Graph API` | =_https://_**graph.windows.net**_/contoso.com/users?api-version=2013-04-05_|
| Obter conjunto de mensagens da Caixa de Entrada       |     `Office 365 API`           | =_https://_**outlook.office365.com**_/api/v1.0/me/messages_  |
| Obter arquivos de Carlos   |     `Office 365 API`  | =_https://_**contoso-my.sharepoint.com**pessoal / _ / joe_contoso_com/_api/v1.0/files_ |


Usando a API do Microsoft Graph, você não precisa primeiro descobrir pontos de extremidade de serviço e depois percorrer pontos de extremidade diferentes para obter os arquivos de um usuário, email e assim por diante. Você só precisa interagir com um único namespace da URL REST, que é _**graph.microsoft.com**_.

**API do Microsoft Graph**

|   **Operação**                  |  **API**                          |  **Ponto de extremidade de serviço** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| Descobrir pontos de extremidade de serviço para a API do Office 365                |     `Microsoft Graph`           | Não é necessário |
| Obter usuários           |     `Microsoft Graph` | =_https://_**graph.microsoft.com**_/v1.0/contoso.onmicrosoft.com/users_ |
| Obter conjunto de mensagens da Caixa de Entrada       |     `Microsoft Graph`           | _https://_**graph.microsoft.com**_/v1.0/me/messages_  |
| Obter arquivos de Carlos   |     `Microsoft Graph `  | _https://_**graph.microsoft.com**_/v1.0/me/drive/root/children_ |


## Quais são os benefícios de usar a API do Microsoft Graph?

Estes são alguns dos benefícios do uso da API do Microsoft Graph:

**Experiência de desenvolvedor consistente e otimizada para o consumo de serviços de nuvem da Microsoft**

-   Namespace único para todos os pontos de extremidade do serviço. Não é necessária a descoberta de ponto de extremidade do serviço.
-   Um token para acessar todos os recursos
-   Navegação integrada e direta entre serviços atualmente em silos (por exemplo, obter a cadeia de departamentos e de gerência do usuário que criou um documento específico)
-   É necessário usar um único conjunto de APIs, ou seja, só será necessário usar a API do Microsoft Graph para a conexão a vários serviços
-   Entidades na plataforma do Office e API REST unificada e expandida 
-   Esquemas e nomenclatura de propriedades consistentes em entidades, incluindo propriedades de navegação entre entidades

**Experiências de trabalho e de vida mais produtivas usando o Office**

-   Conteúdo contextual. Por exemplo, a localização de documentos por grupo, por projeto, por equipe ou por tendências
-   Relacionamentos de usuário contextuais. Por exemplo, você pode encontrar usuários por associação a um grupo, por interesse, por habilidades e por experiência.  Você também pode obter o relacionamento de organograma

**Permitir o desenvolvimento de aplicativos em qualquer linguagem de programação em qualquer plataforma**

-   Ferramentas de desenvolvimento e recursos para todos os desenvolvedores. Você pode desenvolver usando qualquer plataforma e linguagem 
-   Desenvolvimento móvel para todas as plataformas usando tecnologias abertas  
-   Não é preciso ter conhecimento especializado sobre o Exchange, o SharePoint ou o Azure AD para acessar entidades da API do Microsoft Graph

<!---<a name="msg_v2auth"> </a>-->

## A API do Microsoft Graph é compatível com o ponto de extremidade de autenticação de aplicativo v2.0?

Sim. Para saber mais, confira [Autenticar pontos de extremidade do Microsoft Graph usando o ponto de extremidade de autenticação v2](http://graph.microsoft.io/docs/authorization/converged_auth).



  > Os seus comentários são importantes para nós. Junte-se a nós na página do [Stack Overflow](http://stackoverflow.com/questions/tagged/office365). Marque as suas perguntas com [MicrosoftGraph] e [office365].








