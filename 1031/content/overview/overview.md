


# Übersicht über Microsoft Graph

Microsoft Graph (zuvor einheitliche Office 365-API) stellt mehrere APIs aus Microsoft-Clouddiensten über einen einzelnen REST-API-Endpunkt (**https://graph.microsoft.com**) zur Verfügung. Mit Microsoft Graph werden früher schwierige oder komplexe Abfragen zu einfachen Navigationen. 
 
Mit Microsoft Graph erhalten Sie Folgendes:

- Einen einheitlichen API-Endpunkt für den Zugriff auf aggregierte Daten aus mehreren Microsoft-Clouddiensten in einer einzelnen Antwort. 
- Nahtlose Navigation zwischen Entitäten und den Beziehungen zwischen diesen 
- Zugriff auf Informationen und Einblicke aus der Microsoft-Cloud.

All das mit einem einzelnen Authentifizierungstoken.

Sie können die API für den Zugriff auf feste Entitäten wie Benutzer, Gruppen, E-Mails, Nachrichten, Kalender, Aufgaben und Notizen aus Diensten wie Outlook, OneDrive, Azure Active Directory, Planer, OneNote und anderen verwenden. Sie können auch von Office Graph ermittelte Beziehungen (nur für kommerzielle Benutzer) wie Liste der Benutzer, mit denen Sie arbeiten, oder der häufig verwendeten Dokumente abrufen.

Microsoft Graph stellt zwei Endpunkte bereit. Einen allgemeinen Endpunkt /v1.0 und einen Vorschauendpunkt /beta.  Sie können /v1.0 für Produktionsanwendungen verwenden, /beta jedoch nicht.  Mit dem Vorschauendpunkt /beta bieten wir Entwicklern die neuesten Features, damit diese damit experimentieren und uns ihr Feedback mitteilen können. APIs in der Betaversion können sich jederzeit ändern und sind nicht für Produktionszwecke geeignet.

<!--<a name="msg_queries"> </a>-->

##Häufig verwendete Abfragen

Es folgen einige Beispiele für häufig verwendete Abfragen mithilfe der Microsoft Graph-API:

| **Vorgang** | **Dienstendpunkt** |
|:--------------------------|:----------------------------------------|
|   Eigenes Profil mit GET abrufen |    `https://graph.microsoft.com/v1.0/me` |
|   Eigene Dateien mit GET abrufen|   `https://graph.microsoft.com/v1.0/me/drive/root/children` |
|   Eigenes Foto mit GET abrufen	     | `https://graph.microsoft.com/v1.0/me/photo/$value` |
|   Eigene E-Mail mit GET abrufen |   `https://graph.microsoft.com/v1.0/me/messages` |
|   Eigene E-Mails mit Wichtigkeit „Hoch“ mit GET abrufen | `https://graph.microsoft.com/v1.0/me/messages?$filter=importance%20eq%20'high'` |
|   Eigenen Kalender mit GET abrufen |   `https://graph.microsoft.com/v1.0/me/calendar` |
|   Eigenen Vorgesetzten mit GET abrufen	  | `https://graph.microsoft.com/v1.0/me/manager` |
|   Letzten Benutzer zum Ändern der Datei „foo.txt“ mit GET abrufen |  `https://graph.microsoft.com/v1.0/me/drive/root/children/foo.txt/lastModifiedByUser` |
|   Einheitliche Gruppen mit GET abrufen, bei denen ich Mitglied bin|   `https://graph.microsoft.com/v1.0/me/memberOf/$/microsoft.graph.group?$filter=groupTypes/any(a:a%20eq%20'unified')` |
|   Benutzer in meiner Organisation mit GET abrufen	     | `https://graph.microsoft.com/v1.0/users` |
|   Gruppenunterhaltungen mit GET abrufen |   `https://graph.microsoft.com/v1.0/groups/<id>/conversations` |
|   Verwandte Personen mit GET abrufen    | `https://graph.microsoft.com/beta/me/people` |
|   Häufig verwendete Dateien mit GET abrufen |  `https://graph.microsoft.com/beta/me/trendingAround` |
|   Personen mit GET abrufen, mit denen ich zusammenarbeite     | `https://graph.microsoft.com/beta/me/workingWith` |
|   Eigene Aufgaben mit GET abrufen    | `https://graph.microsoft.com/beta/me/tasks` |
|   Eigene Notizen mit GET abrufen |  `https://graph.microsoft.com/beta/me/notes/notebooks` |

<!-- <a name="msg_roof"> </a> -->

## Alle Office 365-Daten an einem Ort

Das folgende Diagramm veranschaulicht den Microsoft Graph-Entwicklerstack und seine Funktionsweise.

![Microsoft Graph-API-Entwicklerstack.](./images/MicrosoftGraph_DevStack.png)

 >  Ihr Feedback ist uns wichtig. Nehmen Sie unter [Stack Overflow](http://stackoverflow.com/questions/tagged/office365+or+microsoftgraph) Kontakt mit uns auf. Taggen Sie Ihre Fragen mit [MicrosoftGraph] und [office365].



