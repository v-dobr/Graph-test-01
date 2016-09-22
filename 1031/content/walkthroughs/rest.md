# Erste Schritte mit Microsoft Graph und REST

In diesem Artikel wird beschrieben, wie Microsoft Graph zum Abrufen von E-Mail-Nachrichten in Office 365 und Outlook.com aufgerufen wird. Dieser Artikel konzentriert sich auf die OAuth- und REST-Anforderungen und -Antworten. Es wird die Abfolge der Anforderungen und Antworten behandelt, die eine App zum Authentifizieren und Abrufen von Nachrichten verwendet.

## Verwenden der OAuth 2.0-Authentifizierung

Zum Aufrufen von Microsoft Graph benötigt Ihre App einen Zugriffstoken aus Azure Active Directory (Azure AD). Im folgenden Beispiel implementiert die App den Authorization Code Grant-Datenfluss zum Abrufen der Zugriffstoken aus Azure AD gemäß der standardmäßigen [OAuth 2.0-Protokolle](http://tools.ietf.org/html/rfc6749)

### Registrieren einer App

Es gibt derzeit zwei Optionen zur Registrierung einer App:

  1. Registrieren einer App mit dem Modell, das nur kommerzielle Office 365-Benutzer, nur Firmen- oder nur Schulkonten unterstützt.
 
  Dieses Modell funktioniert nur mit kommerziellen Office 365-Angeboten. Sobald Sie Ihre App registriert haben, können Sie sie über das [Azure-Verwaltungsportal](https://manage.windowsazure.com) verwalten.

  2. Registrieren Sie sich mit der neuesten-Funktionalität, die für Verbraucher- und für kommerzielle Office 365-Dienste funktioniert (als Autorisierungsendpunkt v2.0 bezeichnet).
 
  Ein einziger Authentifizierungsdienst für Geschäfts- oder Schulkonten und persönliche Konten ist jetzt verfügbar. Dieses Modell bietet einen Authentifizierungsdienst für Geschäfts- und Schulkonten (Azure AD) und für persönliche Konten (Microsoft). Jetzt müssen Sie nur einen Authentifizierungsfluss in Ihrer App implementieren, um Benutzern das Verwenden von Geschäfts- oder Schulkonten, z. B. Office 365 oder OneDrive for Business, oder persönlichen Konten wie Outlook.com oder OneDrive zu ermöglichen.
   
Verwendung Sie das [Anwendungsregistrierungsportal](https://apps.dev.microsoft.com/), um Ihre App zu registrieren und Firmen-, Schul- und persönliche Konten zu unterstützen.

Beachten Sie, dass der v2.0-Endpunkt nach und nach alle Szenarios aus dem vorherigen Autorisierungsendpunkt abdecken wird. Weitere Informationen zur richtigen Auswahl finden Sie in [diesem Artikel](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-limitations/).

Nach dem Registrieren erhalten Sie eine Client-ID und einen geheimen Schlüssel. Diese Werte werden in dem Authorization Code Grant-Datenfluss verwendet.

Im weiteren Verlauf dieses Dokuments wird eine Registrierung unter Verwendung des v2.0-Modells vorausgesetzt. Eine vollständige Anleitung zu den im v2.0-Endpunkt unterstützten Flüssen finden Sie in [diesem Artikel](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-flows/). Eine vollständige Anleitung zum Autorisierungscodeerteilungsfluss finden Sie [in diesem Artikel](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oauth-code/)

### Abrufen eines Autorisierungscodes

Im ersten Schritt des Authorization Code Grant-Datenflusses wird der Autorisierungscode abgerufen. Dieser Code wird vom Autorisierungsserver an die App zurückgegeben, wenn der Benutzer sich anmeldet und der Zugriffsstufe zustimmt, die die App angefordert hat.

Die App generiert zunächst eine Anmelde-URL für den Benutzer. Diese URL muss in einem Browser geöffnet werden, damit der Benutzer sich anmelden und seine Zustimmung erteilen kann.

Die Basis-URL für die Anmeldung sieht wie folgt aus: `https://login.microsoftonline.com/common/oauth2/v2.0/authorize`.

Die App fügt Abfrageparameter an diese Basis-URL an, um dem Autorisierungsserver mitzuteilen, welche App die Anmeldung anfordert und welche Berechtigungen angefordert werden.

- `client_id` Die beim Registrieren der App generierte Client-ID. Anhand dieser erkennt Azure AD, welche App die Anmeldung anfordert.
- `redirect_uri` Der Speicherort, zu dem Azure umleitet, nachdem der Benutzer seine Zustimmung für diese App erteilt hat. Dieser Wert muss mit dem Wert des beim Registrieren der App verwendeten **Umleitungs-URI** übereinstimmen.
- `response_type` Der Antworttyp, den die App erwartet. Dieser Wert ist `code` für den Authorization Code Grant-Datenfluss.
- `scope` – eine durch Leerzeichen getrennte Liste von Bereichen, die die App anfordert. Weitere Informationen finden Sie in [diesem Artikel](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/).
- `state` – ein Wert, der in der Anforderung enthalten ist, die in der Tokenantwort zurückgegeben wird.

Die Anforderungs-URL für unsere Anwendung, die den Lesezugriff auf E-Mails anfordert, sieht beispielsweise wie folgt aus.

```http
GET https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<CLIENT ID>&redirect_uri=http%3A%2F%2Flocalhost/myapp%2F&response_type=code&state=1234&scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
```

Leiten Sie dann den Benutzer zu der Anmelde-URL weiter. Es wird ein Anmeldefenster mit dem Namen der App angezeigt. Nach dem Anmelden wird eine Liste der Berechtigungen angezeigt, die die App anfordert, und der Benutzer wird aufgefordert, diesen zuzustimmen oder sie zu verweigern. Wenn Benutzer dem erforderlichen Zugriff zustimmen, erfolgt im Browser eine Umleitung an den Umleitungs-URI, der in der ursprünglichen Anforderung mit dem Autorisierungscode in der Abfragezeichenfolge angegeben wurde.

```http
http://localhost/myapp/?code=AwABAAAA...cZZ6IgAA&state=1234
```

Wenn Sie auch OpenId Connect für einmaliges Anmelden verwenden, sind zusätzliche Parameter erforderlich. Weitere Informationen dazu finden Sie in [diesem Artikel](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-protocols-oidc/). 

Im nächsten Schritt wird der für ein Zugriffstoken zurückgegebene Autorisierungscode ausgetauscht.

### Abrufen eines Zugriffstokens

Zum Abrufen eines Zugriffstokens stellt die App fomularcodierte Parameter für die Tokenanforderungs-URL (`https://login.microsoftonline.com/common/oauth2/v2.0/token`) mit den folgenden Parametern bereit.

- `client_id` Die beim Registrieren der App generierte Client-ID.
- `client_secret` Der beim Registrieren der App generierte geheime Clientschlüssel.
- `code` Der im vorherigen Schritt abgerufene Autorisierungscode.
- `redirect_uri` Dieser Wert muss mit dem in der Autorisierungscodeanforderung verwendeten Wert übereinstimmen.
- `grant_type` Der von der App verwendete Typ des Zugriffs. Dieser Wert ist `code` für den Authorization Code Grant-Datenfluss.
- `scope` – eine durch Leerzeichen getrennte Liste von Bereichen, die die App anfordert. Weitere Informationen finden Sie in [diesem Artikel](https://azure.microsoft.com/en-us/documentation/articles/active-directory-v2-scopes/).

Die Anforderungs-URL für unsere Anwendung sieht unter Verwendung des Codes aus dem vorherigen Schritt wie folgt aus.

```http
POST https://login.microsoftonline.com/common/oauth2/v2.0/token
Content-Type: application/x-www-form-urlencoded

{
  grant_type=authorization_code
  &code=AwABAAAA...cZZ6IgAA
  &redirect_uri=http%3A%2F%2Flocalhost%2Fmyapp%2F
  &resource=https%3A%2F%2Fgraph.microsoft.com%2F
  &scope=https%3A%2F%2Fgraph.microsoft.com%2Fmail.read
  &client\_id=<CLIENT ID>
  &client\_secret=<CLIENT SECRET>
}
```

Der Server antwortet mit einer JSON-Nutzlast, die das Zugriffstoken enthält.

```http
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8

{
  "token_type":"Bearer",
  "expires_in":"3600",
  "access_token":"eyJ0eXAi...b66LoPVA",
  "scope":"https://graph.microsoft.com/mail.read",
  ...
}
```

Das Zugriffstoken befindet sich im Feld `access_token` der JSON-Nutzlast. Die App verwendet diesen Wert zum Festlegen des Autorisierungsheaders bei REST-Aufrufen für die API.

## Aufrufen von Microsoft Graph

Wenn die App über ein Zugriffstoken verfügt, kann Microsoft Graph aufgerufen werden. Da diese Beispiel-App Nachrichten abruft, verwendet sie eine HTTP-GET-Anforderung für den `https://graph.microsoft.com/v1.0/me/messages`-Endpunkt.

### Optimieren der Anforderung

Apps können das Verhalten der GET-Anforderungen mithilfe von OData-Abfrageparametern steuern. Diese Parameter sollten von den Apps verwendet werden, um die Anzahl der zurückgegebenen Ergebnisse und der für jedes Element zurückgegebenen Felder einzuschränken. 

In dieser Beispiel-App werden die Nachrichten in einer Tabelle angezeigt, in der Betreff, Absender und der Zeitpunkt enthalten sind, zu dem die Nachricht empfangen wurde. In der Tabelle werden maximal 25 Zeilen angezeigt, wobei diese so sortiert sind, dass zuletzt empfangene Nachrichten oben aufgeführt sind. Die App verwendet die folgenden Abfrageparameter, um diese Ergebnisse zu erzielen.

- `$select` - Gibt nur die Felder `subject`, `sender` und `dateTimeReceived` an.
- `$top` - Gibt maximal 25 Elemente an.
- `$orderby` - Sortiert die Ergebnisse nach dem `dateTimeReceived`-Feld.

Als Ergebnis erhält man die folgende Anforderung.

```http
GET https://graph.microsoft.com/v1.0/me/messages?$select=subject,from,receivedDateTime&$top=25&$orderby=receivedDateTime%20DESC
Accept: application/json
Authorization: Bearer eyJ0eXAi...b66LoPVA
```

Da Sie nun gelernt haben, wie Aufrufe für Microsoft Graph getätigt werden, können Sie anhand der API-Referenz beliebige Arten von Aufrufen erstellen, die für Ihre App erforderlich sind. Denken Sie daran, dass entsprechende Berechtigungen für die App bei der App-Registrierung für die Aufrufe konfiguriert sein müssen.


