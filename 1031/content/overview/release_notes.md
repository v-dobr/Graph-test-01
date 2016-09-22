# Microsoft Graph – Versionshinweise und bekannte Probleme

Dieser Artikel enthält Informationen zu den neuen Features für Entwickler, die in der Version von November 2015 der Microsoft Graph-API verfügbar sind, und zu bekannten Problemen, die Sie ggf. berücksichtigen sollten. 


## Allgemein verfügbare Features in Microsoft Graph-API

Die folgenden Microsoft Graph-API-Features sind allgemein verfügbar:

* Benutzer
* Gruppen
* Dateien
* E-Mails
* Kalender
* Persönliche Kontakte 
* CRUD-Vorgänge (Erstellen, Lesen, Aktualisieren und Löschen)
* CORS-Unterstützung (Cross-Origin Resource Sharing)

    
## Vorschaufeatures in Microsoft Graph-API

Die folgenden Microsoft Graph-API-Vorschaufeatures sind verfügbar:

* Hinweise 
* Aufgaben
* Excel
* Kontakte
* Organisationskontakte
* Anwendungen
* Dienstprinzipale
* Verzeichnisschemaerweiterungen
* Webhooks
* Erkenntnisse und Beziehungen: Trends und Funktionsweise
* Direkter Gruppenzugriff auf Inhalte nach der Erstellung
* Zusammengeführtes Authentifizierungsmodell für persönliche Konten sowie Geschäfts- und Schulkonten. Obwohl dies ein Vorschaufeature ist, steht es in `/v1.0`- und `/beta`-Versionen zur Verfügung.


## Bekannte Probleme in Microsoft Graph

Die folgenden Probleme sind in Microsoft Graph bekannt.

### Benutzer
#### Kein direkter Zugriff nach der Erstellung
Benutzer können direkt über POST in der Benutzerentität erstellt werden. Eine Office 365-Lizenz muss zunächst einem Benutzer zugewiesen werden, damit dieser Zugriff auf Office 365-Dienste erhält. Aufgrund der verteilten Art des Diensts kann es bis zu 15 Minuten dauern, bis Dateien-, Nachrichten- und Ereignisentitäten für diesen Benutzer über die Microsoft Graph-API zur Verfügung stehen. In dieser Zeit erhalten Apps als Antwort den HTTP-Fehler 404. 

#### Fotoeinschränkungen
Benutzerprofilfotos können nur gelesen und aktualisiert werden, wenn der Benutzer über ein Postfach verfügt.  Darüber hinaus kann auf Fotos, die *ggf.* zuvor mithilfe der **thumbnailPhoto**-Eigenschaft (unter Verwendung der Office 365 Unified-API-Vorschau oder Azure AD Graph oder über die AD Connect-Synchronisierung) gespeichert wurden, nicht mehr über die Microsoft Graph-Benutzerfotoeigenschaft zugegriffen werden.  Beim Auftreten eines Fehlers beim Lesen oder Aktualisieren eines Fotos wird die folgende Meldung angezeigt:

```javascript
    {
      "error": {
        "code": "ErrorNonExistentMailbox",
        "message": "The SMTP address has no mailbox associated with it."
      }
    }
```

 > **Hinweis**:  Kurz nach der allgemeinen Verfügbarkeit wird das Speichern und Abrufen von Benutzerprofilfotos aktiviert, selbst wenn der Benutzer über kein Postfach verfügt. Dann sollte dieser Fehler nicht mehr auftreten.

#### Standardordner für Kontakte

In der `/v1.0`-Version enthält `GET /me/contactFolders` keinen Standardordner für Kontakte des Benutzers. 

Es wird ein Update zur Verfügung gestellt. In der Zwischenzeit können Sie die folgende [list contacts](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_contacts)-Abfrage und die **parentFolderId**-Eigenschaft als Problemumgehung verwenden, 
um die ID des Standardordners für Kontakte abzurufen:

```
GET https://graph.microsoft.com/v1.0/me/contacts?$top=1&$select=parentFolderId
```
In der obigen Abfrage gilt Folgendes:
1. `/me/contacts?$top=1` ruft die Eigenschaften eines [Kontakts](http://graph.microsoft.io/docs/api-reference/v1.0/resources/contact) im Standardordner für Kontakte ab.
2. Das Anfügen von `&$select=parentFolderId` gibt nur die **parentFolderId**-Eigenschaft des Kontakts zurück, also die ID des Standardordners für Kontakte.

#### Hinzufügen von und Zugreifen auf ICS-basierte Kalender im Postfach des Benutzers
Derzeit gibt es eine teilweise Unterstützung für einen Kalender, der auf einem Internet-Kalenderabonnement (ICS) basiert:
* Sie können einen ICS-basierten Kalender einem Benutzerpostfach über die Benutzeroberfläche, jedoch nicht über die Microsoft Graph-API hinzufügen. 
* [Das Auflisten der Kalender des Benutzers](http://graph.microsoft.io/docs/api-reference/v1.0/api/user_list_calendars) ermöglicht Ihnen, die Eigenschaften **Name**, **Farbe** und **ID** jedes 
[Kalenders](http://graph.microsoft.io/docs/api-reference/v1.0/resources/calendar) in der Standardkalendergruppe oder einer bestimmten Kalendergruppe des Benutzers abzurufen, einschließlich ICS-basierte Kalender. 
Sie können die ICS-URL in der Kalenderressource nicht speichern und nicht darauf zugreifen.
* Sie haben auch die Möglichkeit zum [Auflisten der Ereignisse](http://graph.microsoft.io/docs/api-reference/v1.0/api/calendar_list_events) eines ICS-basierten Kalenders.

### Gruppen
#### Richtlinie
Beim Erstellen und Benennen von einheitlichen Gruppen mit Microsoft Graph werden einheitliche Gruppenrichtlinien umgegangen, die über die Outlook Web App konfiguriert wurden. 

#### Berechtigungsbereiche für Gruppen
Microsoft Graph stellt zwei Berechtigungsbereiche (*Group.Read.All* und *Group.ReadWrite.All*) für den Zugriff auf Gruppen-APIs zur Verfügung.  Diesen Berechtigungsbereichen muss ein Administrator seine Zustimmung erteilen (Dies ist anders als in der Vorschauversion).  In Zukunft werden neue Bereiche für Gruppen hinzugefügt, für die Benutzer Ihre Zustimmung erteilen können.

#### Hinzufügen und Abrufen von Anlagen von Gruppenbeiträgen
Durch das [Hinzufügen](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_post_attachments) von Anlagen zu Gruppenbeiträgen, 
das [Auflisten](http://graph.microsoft.io/docs/api-reference/v1.0/api/post_list_attachments) und das Abrufen von Anlagen von Gruppenbeiträgen wird aktuell die Fehlermeldung „Die OData-Anforderung wird nicht unterstützt“ zurückgegeben. 
Für die `/v1.0`- und `/beta`-Versionen wurde eine Problemlösung entwickelt, die bis Ende Januar 2016 allgemein verfügbar sein sollte.

### Kontakte
* Derzeit werden nur persönliche Kontakten unterstützt. Organisationskontakte werden derzeit nicht in `/v1.0` unterstützt, sind jedoch in `/beta` vorhanden.
* Die Mobiltelefonnummer von persönlichen Kontakten wird für einen Kontakt nicht zurückgegeben. Diese wird in Kürze hinzugefügt. In der Zwischenzeit können sie über Outlook-APIs auf diese zugreifen.

### Laufwerke, Dateien und Streamen von Inhalten
* Beim ersten Zugriff auf ein persönliches Benutzerlaufwerk über Microsoft Graph tritt ein 401-Fehler auf, wenn der Benutzer seine persönliche Website noch nicht in einem Browser aufgerufen hat.
* Es können Dateien (Dateien in Office-Gruppen, Laufwerke oder E-Mail-Dateianlagen) mit einer Größe von bis zu 4 MB hoch- und heruntergeladen werden.

### In vorhandenen Office 365-REST-APIs verfügbare Funktionen
#### Synchronisierung
Outlook-, OneDrive- und Azure AD-Synchronisierungsfunktionen (in Azure AD auch differenzielle Abfrage genannt) stehen in `/v1.0` oder `/beta` nicht zur Verfügung.  Wenn für die Anwendung Synchronisierungsfunktionen erforderlich sind, verwenden Sie vorhandene Office 365- und Azure AD-REST-APIs oder testen Sie das neue Webhooks-Vorschaufeature über Microsoft Graph für Ereignisse, Nachrichten und Kontakte.

> **Hinweis**:  Die Lücke zwischen vorhandenen APIs und Microsoft Graph einschließlich Synchronisierung soll so schnell wie möglich geschlossen werden.

#### Batchverarbeitung
Batchverarbeitung wird von Microsoft Graph nicht unterstützt. Sie können jedoch die Outlook-Betaendpunkte und 
[Outlook-REST-Batchaufrufe](https://msdn.microsoft.com/en-us/office/office365/api/batch-outlook-rest-requests) verwenden. 

#### Verfügbarkeit in China
Der Microsoft Graph-Dienst wird von21Vianet betrieben (und ist jetzt in China verfügbar). Lesen Sie [Unabhängige Microsoft Graph-Cloudbereitstellungen](http://graph.microsoft.io/docs/overview/deployments), um weitere Informationen zu erhalten und mehr über die Einschränkungen zu erfahren.

#### Dienstaktionen und Funktionen
`isMemberOf` und `getObjectsById` sind in Microsoft Graph nicht verfügbar.

### Microsoft Graph-Berechtigungen
Neueste Informationen zu den von Microsoft Graph unterstützten Anwendungs- und delegierten Berechtigungen finden Sie im Thema [Berechtigungsbereiche](http://graph.microsoft.io/docs/authorization/permission_scopes).  Darüber hinaus gelten die folgenden Einschränkungen für `v1.0`:

|Berechtigung |   Berechtigungstyp | Einschränkung |  Alternative |
|-----------|-----------------|------------|--------------|
|_User.ReadWrite_| Delegiert    | Mobiltelefonnummer kann nicht aktualisiert werden.|    Wählen Sie `Directory.AccessAsUser.All`.| 
|_User.ReadWrite.All_|  Delegiert|  Es können keine anderen CRUD-Vorgänge für `User` ausgeführt werden als das Aktualisieren des HD-Fotos vom Benutzer und der erweiterten Profileigenschaften.| Wählen Sie `Directory.ReadWrite.All` oder `Directory.AccessAsUser.All`, wenn Löschen von Benutzern erforderlich ist.|
|_User.Read.All_|   Anwendung |Es können keine Lesevorgänge für andere Benutzer ausgeführt werden.| Wählen Sie `Directory.Read.All`.|
| _User.ReadWrite.All_ |    Anwendung |   Es können keine anderen CRUD-Vorgänge für `User` ausgeführt werden als das Aktualisieren des HD-Fotos vom Benutzer und der erweiterten Profileigenschaften. |    Wählen Sie `Directory.ReadWrite.All`. **Hinweis**: Löschen von Benutzern ist nicht möglich.|
|_Group.Read.All_   | Anwendung | Gruppen oder Gruppenmitgliedschaften können nicht aufgelistet werden.  Gruppeninhalte können weiterhin für Office-Gruppen gelesen werden.   | Wählen Sie `Directory.Read.All`. |
|_Group.ReadWrite.All_  | Anwendung   | Auflisten von Gruppen oder Gruppenmitgliedschaften, Erstellen von Gruppen, Aktualisieren von Gruppenmitgliedschaften oder Löschen von Gruppen ist nicht möglich.  Gruppeninhalte können weiterhin für Office-Gruppen gelesen und aktualisiert werden.	   | Wählen Sie `Directory.ReadWrite.All`. **Hinweis**:  Löschen von Gruppen ist nicht möglich.|

Darüber hinaus gelten die folgenden `/beta`-Einschränkungen:

|Berechtigung |   Berechtigungstyp | Einschränkung |  Alternative |
|-----------|-----------------|------------|--------------|
| _Group.ReadWrite.All_ | Delegiert | Planeraufgaben können in Office-Gruppen nicht gelesen oder aktualisiert werden.	  | Wählen Sie `Tasks.ReadWrite`.|
|_Tasks.ReadWrite_  | Delegiert | Aufgaben von angemeldeten Benutzern können nicht gelesen oder aktualisiert werden.| Wählen Sie `Group.ReadWrite.All`.|

### OData-bezogene Einschränkungen
* **$expand**-Einschränkungen: 
 * Keine Unterstützung für `nextLink`.
 * Keine Unterstützung für mehr als eine Erweiterungsebene.
 * Keine Unterstützung mit zusätzlichen Parametern (**$filter**, **$select**).
* Mehrere Namespaces werden nicht unterstützt.
* GET-Anforderungen für `$ref` und Umwandlung wird für Benutzer, Gruppen, Geräte, Dienstprinzipale und Anwendungen nicht unterstützt.
* `@odata.bind` wird nicht unterstützt.  Ein Entwickler kann daher `Accepted` oder `RejectedSenders` für eine Gruppe nicht ordnungsgemäß festlegen.
* `@odata.id` ist bei Nicht-Aufnahmenavigationen (z. B. Nachrichten) nicht vorhanden, wenn minimale Metadaten verwendet werden.
* Arbeitsauslastungsübergreifende Filter/Suche ist nicht verfügbar. 
* Volltextsuche (unter Verwendung von **$search**) ist nur für einige Entitäten wie Nachrichten verfügbar.

  >  Ihr Feedback ist uns wichtig. Nehmen Sie unter [Stack Overflow](http://stackoverflow.com/questions/tagged/office365) Kontakt mit uns auf. Taggen Sie Ihre Fragen mit [MicrosoftGraph] und [office365].

  
             .

