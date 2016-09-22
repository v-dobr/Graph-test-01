# Aufrufen von Microsoft Graph in einem Dienst oder einer Dämon-App

In diesem Artikel werden Aufgaben beschrieben, die zum Herstellen einer Verbindung zwischen Ihrem Einzelinstanzdienst oder Dämon-App und Office 365 und zum Aufrufen der Microsoft Graph-API mindestens erforderlich sind.

## Übersicht

Zum Aufrufen der Microsoft Graph-API in einem Dienst oder einer Dämon-App müssen Sie die folgenden Aufgaben ausführen.

1. Registrieren der Anwendung in Azure Active Directory
2. Anfordern eines Zugriffstokens vom Endpunkt, der ein Token ausgibt.
3. Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API

## Registrieren der Anwendung in Azure Active Directory

Bevor Sie  mit Office 365 arbeiten können, müssen Sie Ihre Anwendung registrieren und die Berechtigungen für die Verwendung der Microsoft Graph-Dienste festlegen. 
Mithilfe des [App-Registrierungstools](https://dev.office.com/app-registration) können Sie mit nur wenigen Klicks Ihre Anwendung registrieren. Zum Verwalten müssen Sie das [Microsoft Azure-Verwaltungsportal](https://manage.windowsazure.com) verwenden.

Alternativ können Sie die App manuell registrieren. Die Anweisungen dazu finden Sie im Abschnitt [Registrieren der Webserver-App mithilfe des Azure-Verwaltungsportals](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp). Beachten Sie dabei Folgendes:

* Konfigurieren Sie nach der Registrierung der Anwendung die **Anwendungsberechtigungen**, die für Ihren Dienst oder Ihre Dämon-App erforderlich sind.

Notieren Sie sich die folgenden Werte auf der Seite „Konfigurieren“ in Ihrer Azure-Anwendung, da Sie diese Werte zum Konfigurieren des OAuth-Flusses in Ihrem Dienst oder Ihrer Dämon-App benötigen.

* Client-ID (eindeutig für Ihre Anwendung)
* Ein Anwendungsschlüssel (eindeutig für Ihre Anwendung)
* OAuth 2.0-Tokenendpunkt Ihrer App
  * Klicken Sie für diesen Wert auf *Endpunkte anzeigen* am unteren Rand des Azure-Verwaltungsportals auf der App-Seite. Der Endpunkt sieht wie `https://login.microsoftonline.com/<tenantId>/oauth2/token` aus.

## Anfordern eines Zugriffstokens vom Endpunkt, der ein Token ausgibt

Im Gegensatz zu Client-Apps kann sich ein Benutzer nicht bei Ihrem Dienst oder Ihrer Dämon-App anmelden und die Anwendung autorisieren. In diesem Fall muss die Anwendung den Zugriffsfluss für die OAuth 2.0-Clientanmeldeinformationen implementieren, damit die Anwendung eigene Anmeldeinformationen, die Client-ID und einen Anwendungsschlüssel für die Authentifzierung zum Aufrufen von Microsoft Graph verwenden kann, statt die Identität eines Benutzers anzunehmen. Weitere Informationen zum Authentifizierungsfluss finden Sie unter [Aufrufe zwischen Diensten unter Verwendung von Clientanmeldeinformationen](https://msdn.microsoft.com/en-us/library/azure/dn645543.aspx)

Führen Sie eine HTTP POST-Anforderung an den Endpunkt, der ein Token ausgibt, mit den folgenden Parametern durch. Ersetzen Sie dabei `<clientId>` und `<clientSecret>` durch die Client-ID Ihrer App bzw. den Anwendungsschlüssel.

```http
POST https://login.microsoftonline.com/<tenantId>/oauth2/token HTTP/1.1
Content-Type: application/x-www-form-urlencoded

grant_type=client_credentials
&client_id=<clientId>
&client_secret=<clientSecret>
&resource=https://graph.microsoft.com
```

Als Antwort werden ein Zugriffstoken und Ablauf Access-Token und Gültigkeitsinformationen zurückgegeben.

```json
{ 
  "token_type": "Bearer",
  "expires_in": "3599",
  "scope": "User.Read",
  "expires_on": "1449685363",
  "not_before": "1449681463",
  "resource": "https://graph.microsoft.com",
  "access_token": "<token>"
}
```

## Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API

Mit einem Zugriffstoken kann Ihre App authentifizierte Anforderungen an die Microsoft Graph-API senden. Ihre App muss das Zugriffstoken an den **Autorisierungs**-Header jeder Anforderung anfügen.

Ein Dienst oder eine Dämon-App kann z. B. alle Benutzer in einem Mandanten abrufen, wenn für diesen bzw. diese die Berechtigung *Vollständige Profile aller Benutzer lesen* im Azure-Verwaltungsportal ausgewählt ist. 

```http
GET https://graph.microsoft.com/v1.0/users
Authorization: Bearer <token>
```

Microsoft Graph ist eine leistungsfähige einheitliche API, die für die Interaktion mit beliebigen Microsoft-Daten verwendet werden kann. Informationen zu weiteren Möglichkeiten mit der Microsoft Graph-API finden Sie in der [API-Referenz](http://graph.microsoft.io/docs/api-reference/v1.0).
