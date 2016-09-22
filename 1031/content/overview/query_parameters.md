# Optionale Abfrageparameter in Microsoft Graph 
## Optionale OData-Abfrageparameter
Microsoft Graph stellt mehrere optionale Abfrageparameter bereit, die Sie zum Festlegen und Steuern der in einer Antwort zurückgegebenen Datenmenge verwenden können. Microsoft Graph unterstützt die folgenden Abfrageoptionen. 

|Name|Wert|Beschreibung|
|:---------------|:--------|:-------|
|$select|string|Durch Trennzeichen getrennte Liste der Eigenschaften, die in der Antwort aufgenommen werden.|
|$expand|string|Durch Trennzeichen getrennte Liste der Beziehungen, die in der Antwort erweitert und aufgenommen werden.  |
|$orderby|string|Durch Trennzeichen getrennte Liste von Eigenschaften, die zum Sortieren der Elemente in der Antwortsammlung verwendet werden.|
|$filter|string|Filtert die Antwort basierend auf einer Reihe von Kriterien.|
|$top|int|Die Anzahl der Elemente, die in einem Resultset zurückgegeben werden.|
|$skip|int|Die Anzahl der Elemente, die in einem Resultset übersprungen werden.|
|$skipToken|string|Pagingtoken, das zum Abrufen des nächsten Resultsets verwendet wird.|
|$count|Keine|Eine Auflistung,und die Anzahl der Elemente in der Auflistung.|


**Abfrageparameter für Codierung**

- Wenn Sie Abfrageparameter in [Microsoft Graph Explorer](https://graphexplorer2.azurewebsites.net/) ausprobieren, können Sie einfach die unten aufgeführten Beispiele kopieren und einfügen, 
ohne der Abfragezeichenfolge eine URL-Codierung hinzufügen zu müssen. Das folgende Beispiel funktioniert einwandfrei _im Graph Explorer_, ohne die Leerzeichen und Anführungszeichen zu codieren:
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 
- Stellen Sie beim Festlegen von Abfrageparametern _in Ihrer App_ allgemein sicher, dass Sie solche Zeichen angemessen codieren, für die [im URI eine besondere Bedeutung reserviert ist](https://tools.ietf.org/html/rfc3986#section-2.2). 
Codieren Sie zum Beispiel die Leerzeichen und Anführungszeichen im letzten Beispiel wie unten dargestellt:
```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address%20eq%20%27jon@contoso.com%27
```

### $select
Verwenden Sie zum Festlegen einer zurückzugebenden Teilmenge von Eigenschaften die **$select**-Abfrageoption. Wenn Sie z. B. Ihre Nachrichten abrufen, möchten Sie ggf. festlegen, dass nur die Eigenschaften **Von** und **Betreff** der Nachrichten zurückgegeben werden.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$select=from,subject
```

<!--For example, when retrieving the children of an item on a drive, you want to select that only the **name** and **size** properties of items are returned.

```http
GET https://graph.microsoft.com/v1.0/me/drive/root/children?$select=name,size
```

By submitting the request with the `$select=name,size` query string, the objects
in the response will only have those property values included. 


```json
{
  "value": [
    {
      "id": "13140a9sd9aba",
      "name": "Documents",
      "size": 1024
    },
    {
      "id": "123901909124a",
      "name": "Pictures",
      "size": 1012010210
    }
  ]
}
```--> 

### $expand

In Microsoft Graph-API-Anforderungen wird die Navigation zu einem Objekt oder einer Auflistung des Elements, auf das verwiesen wird, nicht automatisch erweitert. 
Dies ist beabsichtigt, da der Netzwerkdatenverkehr und die Zeit zum Generieren einer Antwort vom Dienst auf diese Weise reduziert werden. Möglicherweise möchten Sie jedoch in einigen Fällen diese Ergebnisse in eine Antwort einbeziehen.

Sie können den **$expand**-Abfragezeichenfolgen-Parameter verwenden, um die API zum Erweitern eines untergeordneten Objekts oder einer Auflistung und zum Einbeziehen dieser Ergebnisse anzuweisen.

Verwenden Sie beispielsweise zum Abrufen von Stammlaufwerkinformationen und der untergeordneten Elemente auf oberster Ebene in einem Laufwerk den **$expand**-Parameter. In diesem Beispiel wird auch eine **$select**-Anweisung verwendet, 
um nur die Eigenschaften_ID_ und _Name_ der untergeordneten Elemente zurückzugeben.

```http
GET https://graph.microsoft.com/v1.0/me/drive/root?$expand=children($select=id,name)
```

>  **Hinweis**: Die maximale Anzahl der erweiterten Objekte für eine Anforderung beträgt 20. 

> Wenn Sie außerdem eine Abfrage für die [Benutzer](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/user)-Ressource durchführen, 
können Sie **$expand** zum Abrufen der Eigenschaften von jeweils nur einem untergeordneten Objekt oder einer Auflistung verwenden. 

Das folgende Beispiel ruft **Benutzer**-Objekte ab, jeweils mit bis zu 20 erweiterten **directReport**-Objekten in der **directReports**-Auflistung:
```http
GET https://graph.microsoft.com/v1.0/users?$expand=directReports
```
Einige andere Ressourcen haben möglicherweise ebenfalls einen Höchstwert, führen Sie daher immer eine Überprüfung auf mögliche Fehler durch.


<!---The following shows a sample result that is returned in the response body.-->


### $orderby

Verwenden Sie zum Festlegen der Sortierreihenfolge der aus der API zurückgegebenen Elemente die **$orderby**-Abfrageoption. 

Wenn die zurückgegebenen Benutzer in der Organisation nach dem Anzeigenamen sortiert werden sollen, lautet die Syntax hierfür wie folgt:

```http
GET https://graph.microsoft.com/v1.0/users?$orderBy=displayName
``` 

Sie können auch nach komplexen Typentitäten sortieren. Im folgenden Beispiel werden Nachrichten abgerufen und nach dem **Adresse**-Feld der **from**-Eigenschaft sortiert, die vom komplexen Typ **emailAddress** ist:

```http
GET https://graph.microsoft.com/v1.0/me/messages?$orderBy=from/emailAddress/address
``` 

Wenn Sie die Ergebnisse in aufsteigender oder absteigender Reihenfolge sortieren möchten, fügen Sie entweder `asc` oder `desc` an den Namen des Felds getrennt durch ein Leerzeichen an, z. B. 
`?orderby=name%20desc`.

 >  **Hinweis**: **$orderby** kann nicht zusammen mit $filter-Ausdrücken verwendet werden.

### $filter
Wenn Sie die zurückgegebenen Daten anhand bestimmter Kriterien filtern möchten, verwenden Sie die **$filter**-Abfrageoption. Wenn die zurückgegebenen Benutzer in der Organisation anhand des Anzeigenamens gefiltert werden sollen, 
der mit „Garth“ beginnt, lautet die Syntax hierfür wie folgt:

```http
GET https://graph.microsoft.com/v1.0/users?$filter=startswith(displayName,'Garth')
```

Sie können auch nach komplexen Typentitäten filtern. Das folgende Beispiel gibt Nachrichten zurück, bei denen das Feld **address** der **from**-Eigenschaft gleich „jon@contoso.com“ ist. 
Die **from**-Eigenschaft hat den komplexen Typ **emailAddress**.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$filter=from/emailAddress/address eq 'jon@contoso.com'
``` 

### $top
Verwenden Sie zum Festlegen der maximalen Anzahl der in einem Resultset zurückzugebenden Elemente die **$top**-Abfrageoption. Die **$top**-Abfrageoption ermittelt eine Teilmenge in der Sammlung. Diese Teilmenge setzt sich aus den festgelegten ersten N Elementen zusammen, wobei N eine positive ganze Zahl ist, die durch diese Abfrageoption angegeben ist. 
Wenn beispielsweise die ersten fünf Nachrichten im Postfach des Benutzers zurückgegeben werden sollen, lautet die Syntax wie folgt:

```http
GET https://graph.microsoft.com/v1.0/me/messages?$top=5
```

### $skip
Verwenden Sie zum Festlegen der Anzahl der Elemente, die vor dem Abrufen von Elementen in einer Sammlung übersprungen werden sollen, die **$skip**-Abfrageoption. 
Wenn die zurückgegebenen Ereignisse nach Erstellungsdatum sortiert werden, wobei mit dem 21. Ereignis begonnen wird, lautet die Syntax wie folgt.

```http
GET  https://graph.microsoft.com/v1.0/me/events?$orderby=createdDateTime&$skip=20
```

### $skipToken
Verwenden Sie zum Paginieren und Festlegen des nächsten zurückzugebenden Resultsets die **$skipToken**-Abfrageoption.  Der Wert einer **$skipToken**-Abfrageoption ist ein Token, das einen Startpunkt in einer Sammlung festlegt. Der Wert einer **$skipToken**-Abfrageoption könnte z. B. das erste Element in einer Sammlung oder das achte Elements in einer Sammlung mit 20 Elementen oder eine andere Position in der Sammlung angeben.

Da der Wert einer **$skipToken**-Abfrageoption einen Index in einer Sammlung ermittelt, wird bei deren Verwendung in Ihrer Abfrage eine Teilmenge der Elemente ermittelt. Die ermittelte Teilmenge beginnt beim ersten Element im Index (ermittelt durch den Wert der **$skipToken**-Abfrageoption) und endet beim letzten Element im Set.

In einigen Fällen wird ein `@odata.nextLink`-Wert angezeigt. Manchmal ist ein **$skipToken**-Wert enthalten.  Der **$skipToken**-Wert stellt eine Markierung für den Dienst dar, die angibt, an welcher Stelle das nächste Resultset beginnen soll.  Im Folgenden sehen Sie ein Beispiel eines `@odata.nextLink`-Werts aus einer Antwort.

```
"@odata.nextLink": "https://graph.microsoft.com/v1.0/users?$orderby=displayName&$top=3&$skiptoken=X%2783630372100000000000000000000%27"
```

Wenn das nächste Set von Benutzern in Ihrer Organisation zurückgegeben werden soll und dabei nie mehr als drei Benutzer auf einmal in den Ergebnissen angezeigt werden sollen, lautet die Syntax wie folgt.

```http
GET  https://graph.microsoft.com/v1.0/users?$orderby=displayName&$top=3&$skiptoken=X%2783630372100000000000000000000%27
```

### $count
Verwenden Sie **$count** als Abfrageparameter, wie im folgenden Beispiel dargestellt:
```http
GET  https://graph.microsoft.com/v1.0/me/contacts?$count=true
```
wodurch sowohl die **Kontakte**-Auflistung als auch die Anzahl der Elemente in der **Kontakte**-Auflistung in der `@odata.id`-Eigenschaft zurückgegeben werden.

Hinweis: Dies wird für [directoryObject](http://graph.microsoft.io/en-us/docs/api-reference/v1.0/resources/directoryobject)-Auflistungen nicht unterstützt.
