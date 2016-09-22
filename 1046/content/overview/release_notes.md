# Problemas conhecidos e notas de versão do Microsoft Graph

Este artigo fornece informações sobre os novos recursos para desenvolvedores que estão disponíveis na versão de novembro de 2015 da API do Microsoft Graph e sobre problemas conhecidos a que você deva prestar atenção. 


## Recursos de GA na API do Microsoft Graph

Os seguintes recursos da API do Microsoft Graph estão disponíveis de maneira geral:

* Usuários
* Grupos
* Arquivos
* Email
* Calendário
* Contatos pessoais 
* CRUD (criar, ler, atualizar e excluir) operações
* Suporte a CORS (compartilhamento de recursos entre origens).

    
## Recursos de visualização na API do Microsoft Graph

Os seguintes recursos de visualização da API do Microsoft Graph estão disponíveis:

* Anotações 
* Tarefas
* Excel
* Pessoas
* Contatos organizacionais
* Aplicativos
* Entidades de serviço
* Extensões de esquema de diretório
* Webhooks
* Informações e relacionamentos: Populares e Trabalhando com
* Acesso instantâneo de Grupos ao conteúdo após a criação
* Modelo de autenticação convergida para contas pessoais, corporativas ou de estudante. Embora esse seja um recurso de visualização, ele está disponível em ambas as versões `/v1.0` e `/beta`.


## Problemas conhecidos do Microsoft Graph

Conhecemos os problemas a seguir com o Microsoft Graph.

### Usuários
#### Sem acesso instantâneo após a criação
Os usuários podem ser criados imediatamente por um POST na entidade do usuário. Uma licença do Office 365 deve ser atribuída a um usuário primeiro para que ele possa ter acesso aos serviços do Office 365. Mesmo assim, devido à natureza distribuída do serviço, pode demorar 15 minutos antes que os arquivos, as mensagens e as entidades de eventos fiquem disponíveis para uso por esse usuário na API do Microsoft Graph. Durante esse período, os aplicativos receberão uma resposta de erro 404 HTTP. 

#### Restrições de foto
A leitura e a atualização da foto do perfil do usuário só serão possíveis se o usuário tiver uma caixa de correio.  Além disso, as fotos que *possam* ter sido previamente armazenadas usando a propriedade **thumbnailPhoto** (usando a visualização da API unificada do Office 365, o Azure AD Graph ou por meio da sincronização do AD Connect) deixarão de estar acessíveis com a propriedade das fotos do usuário do Microsoft Graph.  A falha na leitura ou na atualização de uma foto, nesse caso, resultaria no seguinte erro:

```javascript
    {
      "error": {
        "code": "ErrorNonExistentMailbox",
        "message": "The SMTP address has no mailbox associated with it."
      }
    }
```

 > **OBSERVAÇÃO**:  logo após a GA, o armazenamento e a recuperação de fotos de perfil de usuário serão habilitados, mesmo que o usuário não tenha uma caixa de correio, e esse erro deve desaparecer.

#### Pasta Contatos padrão

Na versão `/v1.0`, `GET /me/contactFolders` não inclui a pasta de contatos do usuário padrão. 

Uma correção será disponibilizada. Enquanto isso, você pode usar a seguinte consulta [list contacts](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_contacts) e a propriedade **parentFolderId** 
como uma solução alternativa para obter a ID da pasta de contatos padrão:

```
GET https://graph.microsoft.com/v1.0/me/contacts?$top=1&$select=parentFolderId
```
Na consulta acima:
1. `/me/contacts?$top=1` obtém as propriedades de um [contato](http://graph.microsoft.io/docs/api-reference/v1.0/resources/contact) na pasta de contatos padrão.
2. A anexação de `&$select=parentFolderId` retorna apenas a propriedade **parentFolderId** do contato, que é a ID da pasta de contatos padrão.

#### Adicionando e acessando calendários de baseados em ICS na caixa de correio do usuário
Atualmente, há suporte parcial para um calendário com base em uma Inscrição em Calendário da Internet (ICS):
* Você pode adicionar um calendário baseado em ICS para uma caixa de correio do usuário por meio da interface do usuário, mas não através da API do Microsoft Graph. 
* [Listar os calendários do usuário](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_calendars) permite que você obtenha as propriedades **name**, **color** e **id** de cada [calendar](http://graph.microsoft.io/docs/api-reference/v1.0/resources/calendar) 
no grupo de calendários padrão do usuário ou em um grupo de calendários especificado, inclusive todos os calendários com base em ICS. 
Não é possível armazenar ou acessar a URL da ICS no recurso de calendário.
* Você também pode [listar os eventos](http://graph.microsoft.io/docs/api-reference/v1.0/api/calendar_list_events) de um calendário baseado em ICS.

### Grupos
#### Política
O uso do Microsoft Graph para criar e nomear um grupo unificado ultrapassa qualquer política de grupo unificado que seja configurada pelo Outlook Web App. 

#### Escopos de permissão de grupo
O Microsoft Graph expõe dois escopos de permissão (*Group.Read.All* e *Group.ReadWrite.All*) para obter acesso a APIs de grupos.  Esses escopos de permissão devem ser permitidos por um administrador (que é uma alteração da visualização).  No futuro, pretendemos adicionar novos escopos para grupos que possam ser permitidos pelos usuários.

#### Adicionando e obtendo anexos de postagens de grupo
A [adição](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_post_attachments) de anexos a postagens de grupo a [listagem](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_list_attachments) 
e a obtenção de anexos de postagens do grupo atualmente retornam a mensagem de erro "A solicitação de OData não tem suporte." Uma correção foi implementada nas versões `/v1.0` e `/beta`, 
e deve estar amplamente disponível até o final de janeiro de 2016.

### Contatos
* Somente os contatos pessoais têm suporte no momento. Os contatos organizacionais atualmente não têm suporte na `/v1.0`, mas podem ser encontrados na versão `/beta`.
* O celular de um contato pessoal não está sendo retornado para um contato. Ele será adicionado em breve. Enquanto isso, ele pode ser acessado por meio de APIs do Outlook.

### Unidades, arquivos e streaming de conteúdo
* O acesso pela primeira vez a uma unidade pessoal de um usuário pelo Microsoft Graph antes que ele acesse seu site pessoal por um navegador resulta em uma resposta 401.
* O carregamento ou download de arquivos (em grupos do Office, unidades ou anexos de arquivo de email) é limitado a 4 MB.

### Funcionalidade disponível apenas nas APIs REST do Office 365 existentes
#### Sincronização
Recursos de sincronização do Outlook, do OneDrive e do AD do Azure (no AD do Azure, isso também é conhecido como consulta diferencial) não estão disponíveis na `/v1.0` ou na versão `/beta`.  Se seu aplicativo exigir recursos de sincronização, continue a usar o Office 365 e as APIs do AD REST do Azure existentes ou explore o novo recurso de visualização, os webhooks, oferecido pelo Microsoft Graph para eventos, mensagens e contatos.

> **OBSERVAÇÃO**:  existe o objetivo de acabar com a lacuna entre as APIs existentes e o Microsoft Graph o mais rápido possível, incluindo a questão da sincronização.

#### Envio em lote
O envio em lote não tem suporte do Microsoft Graph. No entanto, você pode usar o ponto de extremidade do Outlook beta e 
[enviar as chamadas REST do Outlook em lote](https://msdn.microsoft.com/en-us/office/office365/api/batch-outlook-rest-requests). 

#### Disponibilidade na China
O serviço Microsoft Graph é operado pela 21Vianet (e agora está disponível na China). Revise [Implantações na nuvem soberana do Microsoft Graph](http://graph.microsoft.io/docs/overview/deployments) para obter mais detalhes, incluindo restrições.

#### Funções e ações de serviço
`isMemberOf` e `getObjectsById` não estão disponíveis no Microsoft Graph

### Permissões do Microsoft Graph
Examine o [tópico de escopos de permissão](http://graph.microsoft.io/docs/authorization/permission_scopes) para obter os detalhes mais recentes sobre as permissões delegadas e os aplicativos com suporte do Microsoft Graph.  Além disso, existem as seguintes limitações da `v1.0`:

|Permissão |   Tipo de permissão | Limitação |  Alternativa |
|-----------|-----------------|------------|--------------|
|_User.ReadWrite_| Delegado    | Não é possível atualizar o número de celular|    Selecione também `Directory.AccessAsUser.All`| 
|_User.ReadWrite.All_|  Delegado|  Não é possível realizar operações de CRUD no `User` além de atualizar a foto do usuário em alta definição e as propriedades de perfil avançadas| Também selecione `Directory.ReadWrite.All` ou `Directory.AccessAsUser.All` se a exclusão do usuário é necessária.|
|_User.Read.All_|   Aplicativo |Não é possível executar operações de leitura em outros usuários| Selecione também `Directory.Read.All`|
| _User.ReadWrite.All_ |    Aplicativo |   Não é possível realizar operações de CRUD no `User` além de atualizar a foto do usuário em alta definição e as propriedades de perfil avançadas |    Selecione também`Directory.ReadWrite.All`. **OBSERVAÇÃO**: Não será possível excluir o usuário.|
|_Group.Read.All_   | Aplicativo | Não é possível enumerar grupos ou associações de grupo.  Ainda pode ler o conteúdo de grupo para Grupos do Office   | Selecione também `Directory.Read.All` |
|_Group.ReadWrite.All_  | Aplicativo   | Não é possível enumerar associações de grupo ou grupos, criar grupos, atualizar associações de grupo ou excluir grupos.  Ainda é possível ler e atualizar o conteúdo de grupo para os Grupos do Office.   | Selecione também `Directory.ReadWrite.All`. **OBSERVAÇÃO**:  Não será possível excluir o grupo.|

Além disso, existem as seguintes limitações da `/beta`:

|Permissão |   Tipo de permissão | Limitação |  Alternativa |
|-----------|-----------------|------------|--------------|
| _Group.ReadWrite.All_ | Delegado | Não é possível ler ou atualizar tarefas do Planejador nos Grupos do Office  | Selecione também `Tasks.ReadWrite`|
|_Tasks.ReadWrite_  | Delegado | Não é possível ler ou atualizar tarefas do usuário conectado| Selecione também `Group.ReadWrite.All`|

### Limitações relacionadas ao OData
* Limitações **$expand**: 
 * Sem suporte para `nextLink`
 * Não há suporte para mais de um nível de expansão
 * Não há suporte com parâmetros adicionais (**$filter**, **$select**)
* Não há suporte para vários namespaces
* GETs em `$ref` e conversão não têm suporte em usuários, grupos, dispositivos, entidades de serviço e aplicativos.
* Não há suporte para `@odata.bind`.  Isso significa que os desenvolvedores não serão capazes de definir corretamente `Accepted` ou `RejectedSenders` em um grupo.
* `@odata.id` não está presente na navegação sem confinamento (como mensagens) quando há o uso de metadados mínimos
* Filtragem/pesquisa entre cargas não disponível. 
* A pesquisa de texto completo (usando **$search**) só está disponível para alguns entidades, como mensagens.

  >  Os seus comentários são importantes para nós. Junte-se a nós na página [Stack Overflow](http://stackoverflow.com/questions/tagged/office365). Marque as suas perguntas com [MicrosoftGraph] e [office365].

  
             .

