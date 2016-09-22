
# Paging der Microsoft Graph-Daten in Ihrer App 
 
Wenn die Microsoft Graph-Anforderungen zu viele Informationen zurückgeben, um sie auf einer Seite anzuzeigen, können Sie sie mit Paging in verwaltbare Abschnitte teilen. 

Sie können in Microsoft Graph-Antworten zur nächsten und vorherigen Seite wechseln. Eine Antwort mit seitennummerierten Ergebnissen enthält ein Überspringungstoken (**odata.nextLink**), mit dem Sie die nächste Ergebnisseite anzeigen können. Dieses Überspringungstoken kann zusammen mit einem **previous-page=true**-Abfrageargument verwendet werden, um zu einer vorherigen Seite zurückzukehren.

Im folgenden Anforderungsbeispiel wird das Wechseln zur nächsten Seite gezeigt:

```
https://graph.microsoft.com/v1.0/users?$top=5$skiptoken=X'4453707402.....0000'
```
Der **$skiptoken**-Parameter aus der vorherigen Antwort ist enthalten und ermöglicht das Wechseln zur nächsten Ergebnisseite.

Im folgenden Anforderungsbeispiel wird das Wechseln zur vorherigen Seite gezeigt:

```
https://graph.microsoft.com/v1.0/users?$top=5$skiptoken=X'4453707.....00000'&previous-page=true
```
Der **$skiptoken** -Parameter aus der vorherigen Antwort ist enthalten. Wenn dieser zusammen mit dem **&previous-page=true**-Parameter verwendet wird, wird die vorherige Ergebnisseite abgerufen.

In den folgenden Schritten werden die Anforderungs-/Antwortschritte zum Wechseln zur nächsten und vorherigen Seite erläutert:

1. Es wird eine Anforderung zum Abrufen einer Liste der ersten zehn von 15 Benutzern gesendet. Die Antwort enthält ein Überspringungstoken, um die letzte Seite der zehn Benutzer anzugeben.
2. Um die letzten fünf Benutzer abzurufen, wird eine andere Anforderung gesendet, die das in der vorherigen Antwort zurückgegebene Überspringungstoken enthält.
3. Um zu einer vorherigen Seite zu wechseln, werden das in Schritt 1 zurückgegebene Überspringungstoken und der **&previous-page=true**-Parameter zur Anforderung hinzugefügt.
4. Die Antwort enthält die vorherige (erste) Seite der zehn Benutzer. In anderen Szenarien mit mehreren Seiten, wird ein neues Überspringungstoken zurückgegeben. Dieses neue Überspringungstoken wird zusammen mit dem Parameter **&previous-page=true** zur Anforderung hinzugefügt, um erneut zur vorherigen Seite zu wechseln.

Folgende Einschränkungen gelten für seitennummerierte Anforderungen:

- Die Standardseitengröße beträgt 100. Die maximale Seitengröße beträgt 999.
- In Abfragen von Rollen wird Paging nicht unterstützt. Dies umfasst das Lesen von Rollenobjekten und Rollenmitgliedern.
- Paging wird nicht für die Linksuche unterstützt, beispielsweise für Abfragen von Gruppenmitgliedern.
