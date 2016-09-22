# Chamando a API do Microsoft Graph

Para acessar e manipular um recurso do Microsoft Graph, você chama e especifica as URLs de recurso usando uma das operações a seguir:   

- GET
- POST
- PATCH
- PUT
- DELETE 

Todas as solicitações de API do Microsoft Graph usam o seguinte padrão de URL básico:

```
    https://graph.microsoft.com/{version}/{resource}?[odata_query_parameters]
```

Para esta URL:

- `https://graph.microsoft.com` é o ponto de extremidade da API do Microsoft Graph.
- `{version}` é a versão de serviço de destino, por exemplo, `v1.0` ou `beta`.
- `{resource}` é o caminho ou o segmento de recursos, como:
  - `users`, `groups`, `devices`, `organization`
  - O alias `me`, que é resolvido como o usuário conectado
   - Os recursos que pertencem a um usuário, como `me/events`, `me/drive` ou `me/messages`
  - O alias `myOrganization`, que é resolvido como o locatário do usuário conectado da organização
- `[odata_query_parameters]` representa os parâmetros de consulta OData adicionais, como `$filter` e `$select`.

Opcionalmente, você também pode especificar o locatário como parte da sua solicitação. Ao usar `me`, não especifique o locatário. Para obter uma lista com as solicitações comuns, consulte [Visão geral do Microsoft Graph](overview.md).

##Metadados da API do Microsoft Graph
O documento de serviço ($metadata) é publicado na raiz do serviço. Por exemplo, você pode exibir o documento de serviço para as versões 1.0 e beta com as URLs a seguir.

Metadados `v1.0` da API do Microsoft Graph.
```
    https://graph.microsoft.com/v1.0/$metadata
```
Metadados `beta` da API do Microsoft Graph.
```
    https://graph.microsoft.com/beta/$metadata
```

Os metadados permitem que você veja entidades, tipos e conjuntos de entidade, tipos complexos e enumerações da API do Microsoft Graph. Usando os metadados e as ferramentas de terceiros disponíveis, você pode criar objetos serializados e gerar bibliotecas de cliente para uso simplificado da API REST.  

Uma URL de recurso é determinada pelo modelo de dados da entidade Microsoft Graph. A prescrição é esboçada no esquema de metadados de entidade ($metadata). 

Os nomes de recursos e parâmetros de consulta da URL do caminho e os parâmetros e valores de ação não diferenciam maiúsculas de minúsculas. 
No entanto, os valores que atribuir, as IDs de entidade e outros valores codificados na base 64 diferenciam maiúsculas de minúsculas.

As seguintes seções mostram algumas chamadas básicas de padrão de programação para a API do Microsoft Graph.

##Navegar de uma coleção para um membro

Para exibir as informações sobre um usuário, você obtém a entidade `User` da coleção `users` para o usuário específico identificado por seu identificador usando uma solicitação HTTPS GET. Para uma entidade `User`, tanto a propriedade `id` quanto a `userPrincipalName` podem ser usadas como o identificador. A solicitação de exemplo a seguir usa o valor `userPrincipalName` como ID do usuário. 

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com HTTP/1.1
Authorization : Bearer <access_token>
```

Se tiver êxito, você obterá uma resposta 200 OK contendo a representação de recursos do usuário na carga, conforme mostrado abaixo:

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 982

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
    "id": "c95e3b3a-c33b-48da-a6e9-eb101e8a4205",
    "city": "Redmond",
    "country": "USA",
    "department": "Help Center",
    "displayName": "John Doe",
    "givenName": "John",
    "userPrincipalName": "Johndoe@contoso.onmicrosoft.com",

    ... 
}
```


##Projetar de uma entidade para as propriedades
Para recuperar apenas os dados biográficos do usuário, conforme fornecido por ele na descrição _Sobre mim_, e suas habilidades, você pode adicionar a opção de consulta $select à solicitação anterior. Por exemplo:

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com?$select=displayName,aboutMe,skills HTTP/1.1
Authorization : Bearer <access_token>
```

A resposta bem-sucedida retorna o status 200 OK e uma carga com o seguinte formato:

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 169

{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users/$entity",
    "aboutMe": "A cool and nice guy.",
    "displayName": "John Doe",
    "skills": [
        "n-Lingual",
        "public speaking",
        "doodling"
    ]
}
```

Aqui, em vez de todos os conjuntos de propriedade na entidade `user`, somente as propriedades `aboutMe`, `displayName` e `skills` são retornadas.

##Passar para outro recurso por meio de relação
Um gerente tem uma relação `directReports` com outros usuários diretamente subordinados a ele. Para consultar a lista de subordinados de um usuário, você pode usar a solicitação HTTPS GET a seguir para navegar para o destino pretendido via passagem de relação. 

```no-highlight 
GET https://graph.microsoft.com/v1.0/users/john.doe@contoso.onmicrosoft.com/directReports HTTP/1.1
Authorization : Bearer <access_token>
```

A resposta bem-sucedida retorna o status 200 OK e uma carga com o seguinte formato:

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
content-length: 152
    
{
    "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#directoryObjects/$entity",
    "@odata.type": "#microsoft.graph.user",
    "id": "c37b074d-fe9d-4e68-83ad-b4401d3be174",
    "department": "Sales & Marketing",
    "displayName": "Bonnie Kearney",

    ...
}
```

Da mesma forma, você pode seguir um relacionamento para navegar até os recursos relacionados. Por exemplo, a relação `user => messages` habilita a passagem de um nó do Azure AD para um nó do Exchange Online. O exemplo a seguir mostra como fazer isso em uma chamada à API REST:


```no-highlight 
GET https://graph.microsoft.com/v1.0/me/messages HTTP/1.1
Authorization : Bearer <access_token>
```

    
A resposta bem-sucedida retorna o status 200 OK e uma carga com o seguinte formato:


```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal
odata-version: 4.0
content-length: 147
    
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('john.doe%40contoso.onmicrosoft.com')/Messages",
  "@odata.nextLink": "https://graph.microsoft.com/v1.0/me/messages?$top=1&$skip=1",
  "value": [
    {
      "@odata.etag": "W/\"FwAAABYAAABMR67yw0CmT4x0kVgQUH/3AAJL+Kej\"",
      "id": "<id-value>",
      "createdDateTime": "2015-11-14T00:24:42Z",
      "lastModifiedDateTime": "2015-11-14T00:24:42Z",
      "changeKey": "FwAAABYAAABMR67yw0CmT4x0kVgQUH/3AAJL+Kej",
      "categories": [],
      "receivedDateTime": "2015-11-14T00:24:42Z",
      "sentDateTime": "2015-11-14T00:24:28Z",
      "hasAttachments": false,
      "subject": "Did you see last night's game?",
      "body": {
        "ContentType": "HTML",
        "Content": "<content>"
      },
      "BodyPreview": "it was great!",
      "Importance": "Normal",
            
       ...
    }
  ]
}
```

##Projetar das entidades para as propriedades
Além da projeção de uma entidade única em suas propriedades, você também pode aplicar a opção de consulta `$select` semelhante a uma coleção de entidades para projetá-las em uma coleção de algumas de suas propriedades. Por exemplo, para consultar o nome dos itens na unidade do usuário conectado, você pode enviar a seguinte solicitação HTTPS GET:

```no-highlight 
GET https://graph.microsoft.com/v1.0/me/drive/root/children?$select=name HTTP/1.1
Authorization : Bearer <access_token>
```

A resposta bem-sucedida retorna um código de status OK 200 e uma carga que contém os nomes e os tipos de arquivos compartilhados, conforme mostrado no exemplo abaixo:

```no-highlight 
{
  "@odata.context": "https://graph.microsoft.com/v1.0/$metadata#users('john.doe%40contoso.onmicrosoft.com')/drive/root/children(name,type)",
  "value": [
    {
      "@odata.etag": "\"{896A8E4D-27BF-424B-A0DA-F073AE6570E2},2\"",
      "name": "Shared with Everyone"
    },
    {
      "@odata.etag": "\"{B39D5D2E-E968-491A-B0EB-D5D0431CB423},1\"",
      "name": "Documents"
    },
    {
      "@odata.etag": "\"{9B51EA38-3EE6-4DC1-96A6-230E325EF054},2\"",
      "name": "newFile.txt"
    }
  ]
}
```

##Consulta um subconjunto de usuários com a opção de filtragem de consulta
Para encontrar os funcionários em um determinado cargo dentro de uma organização, você pode navegar de uma coleção de usuários e especificar uma opção de consulta $filter. Um exemplo é mostrado da seguinte maneira:

    
```no-highlight 
GET https://graph.microsoft.com/v1.0/users/?$filter=jobTitle+eq+%27Helper%27 HTTP/1.1
Authorization : Bearer <access_token>
```

A resposta bem-sucedida retorna o código de status OK 200 e uma lista de usuários com o cargo especificado (`'Helper'`), conforme mostrado no exemplo abaixo:

```no-highlight 
HTTP/1.1 200 OK
content-type: application/json;odata.metadata=minimal;odata.streaming=true;IEEE754Compatible=false;charset=utf-8
odata-version: 4.0
content-length: 986

{
    "@odata.context": "https://graph.microsoft.com/v1.0/contoso.onmicrosoft.com/$metadata#users",
    "value": [
        {
            "id": "c95e3b3a-c33b-48da-a6e9-eb101e8a4205",
            "city": "Redmond",
            "country": "USA",
            "department": "Help Center",
            "displayName": "Jane Doe",
            "givenName": "Jane",
            "jobTitle": "Helper",
            ......
            "mailNickname": "Jane",
            "mobile": null,
            "otherMails": [
                "jane.doe@contoso.onmicrosoft.com"
            ],
            ......
            "surname": "Doe",
            "usageLocation": "US",
            "userPrincipalName": "help@contoso.onmicrosoft.com",
            "userType": "Member"
        },
        
        ...
    ]
}
```

##Chamar funções ou ações de OData
O Microsoft Graph também dá suporte a ações e funções de OData para manipular os recursos. 
Por exemplo, a seguinte solicitação HTTPS POST permite que o usuário conectado (`me`) envie uma mensagem de email:
```no-highlight 
POST https://graph.microsoft.com/v1.0/me/microsoft.graph.sendMail HTTP/1.1
authorization: bearer <access_token>
content-type: application/json
content-length: 96

{
  "message": {
    "subject": "Meet for lunch?",
    "body": {
      "contentType": "Text",
      "content": "The new cafeteria is open."
    },
    "toRecipients": [
      {
        "emailAddress": {
          "address": "garthf@a830edad9050849NDA1.onmicrosoft.com"
        }
      }
    ],
    "attachments": [
      {
        "@odata.type": "#Microsoft.OutlookServices.FileAttachment",
        "name": "menu.txt",
        "contentBytes": "bWFjIGFuZCBjaGVlc2UgdG9kYXk="
      }
    ]
  },
  "saveToSentItems": "false"
}
```

A carga de solicitação contém a entrada para a ação `microsoft.graph.sendMail`, que também é definida no $metadata.

Com um único ponto de extremidade unificado, o Microsoft Graph simplifica a interface de programação de aplicativo para os serviços em nuvem da Microsoft. Como resultado, os limites dos serviços em silo desaparecem. Como um desenvolvedor de aplicativos, você não precisa mais rastrear as fontes de dados e implementar interfaces personalizadas entre várias fontes de dados. 


