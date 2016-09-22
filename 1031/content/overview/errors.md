# Microsoft Graph-Fehlerantworten und -Ressourcentypen

<!--In this article:
  
-   [Status code](#msg_status_code)
-   [Error resource type](#msg_error_resource_type)
-   [Code property](#msg_code_property)

<a name="msg_error_response"> </a> -->

Fehler in Microsoft Graph werden mithilfe der standardmäßigen HTTP-Statuscodes sowie eines JSON-Fehlerantwortobjekts 
zurückgegeben.

## HTTP-Statuscodes

In der folgenden Tabelle werden die HTTP-Statuscodes aufgeführt und beschrieben, die zurückgegeben werden können.

| Statuscode | Statusmeldung                  | Beschreibung                                                                                                                            |
|:------------|:--------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------|
| 400         | Bad Request                     | Die Anforderung kann nicht verarbeitet werden, da sie fehlerhaft oder falsch ist.                                                                       |
| 401         | Unauthorized                    | Erforderliche Authentifizierungsinformationen fehlen oder sind für die Ressource nicht gültig.                                                   |
| 403         | Forbidden                       | Der Zugriff auf die angeforderte Ressource wird verweigert. Der Benutzer verfügt möglicherweise nicht über ausreichende Berechtigungen.                                                 |
| 404         | Not Found                       | Die angeforderte Ressource ist nicht vorhanden.                                                                                                  |
| 405         | Method Not Allowed              | Die HTTP-Methode in der Anforderung ist für die Ressource nicht zulässig.                                                                         |
| 406         | Not Acceptable                  | Dieser Dienst unterstützt nicht das im Accept-Header angeforderte Format.                                                                |
| 409         | Conflict                        | Der aktuellen Status steht im Konflikt mit dem, was die Anforderung erwartet. Beispielsweise ist der angegebene übergeordnete Ordner nicht vorhanden.                   |
| 410         | Gone                            | Die angeforderte Ressource ist nicht mehr auf dem Server verfügbar.                                               |
| 411         | Length Required                 | Ein Content-Length-Header ist für die Anforderung erforderlich.                                                                                    |
| 412         | Precondition Failed             | Eine in der Anforderung bereitgestellte Vorbedingung (z. B. ein if-match-Header) entspricht nicht dem aktuellen Status der Ressource.                       |
| 413         | Request Entity Too Large        | Die Größe der Anforderung überschreitet den zulässigen Höchstwert.                                                                                            |
| 415         | Unsupported Media Type          | Der Inhaltstyp der Anfrage hat ein Format, das vom Dienst nicht unterstützt wird.                                                      |
| 416         | Requested Range Not Satisfiable | Der angegebene Bytebereich ist ungültig oder nicht verfügbar.                                                                                    |
| 422         | Unprocessable Entity            | Die Anforderung kann nicht verarbeitet werden, da sie semantisch falsch ist.                                                                       |
| 429         | Too Many Requests               | Die Clientanwendung wurde gedrosselt und sollte nicht versuchen, die Anforderung zu wiederholen, bis eine bestimmte Zeitspanne abgelaufen ist.                |
| 500         | Internal Server Error           | Beim Verarbeiten der Anforderung ist ein interner Serverfehler aufgetreten.                                                                       |
| 501         | Not Implemented                 | Das angeforderte Feature ist nicht implementiert.                                                                                               |
| 503         | Service Unavailable             | Der Dienst ist vorübergehend nicht verfügbar. Sie können die Anfrage nach einer kurzen Wartezeit wiederholen. Es ist möglicherweise ein Retry-After-Header vorhanden.                   |
| 507         | Insufficient Storage            | Das maximale Speicherkontingent wurde erreicht.                                                                                            |
| 509         | Bandwidth Limit Exceeded        | Die App wurde gedrosselt, da sie die maximale Bandbreite überschritten hat. Die App kann die Anforderung nach einiger Zeit wiederholen. |

Die Fehlerantwort ist ein einzelnes JSON-Objekt, das eine einzige Eigenschaft mit dem Namen **error** enthält. Dieses Objekt enthält alle Fehlerdetails. Sie können die hier zurückgegebenen Informationen anstelle von oder zusätzlich zu dem HTTP-Statuscode verwenden. 
Nachfolgend finden Sie ein Beispiel für einen vollständigen JSON-Fehlertext.

<!-- { "blockType": "example", "@odata.type": "sample.error", "expectError": true, "name": "example-error-response"} -->
```json
{
  "error": {
    "code": "invalidRange",
    "message": "Uploaded fragment overlaps with existing data.",
    "innerError": {
      "requestId": "request-id",
      "date": "date-time"
    }
  }
}
```

<!--<a name="msg_error_resource_type"> </a> -->

## Fehlerressourcentyp

Die Fehlerressource wird immer dann zurückgegeben, wenn ein Fehler bei der Verarbeitung der Anforderung auftritt.

Fehlerantworten entsprechen der Definition der 
[OData v4](http://docs.oasis-open.org/odata/odata-json-format/v4.0/os/odata-json-format-v4.0-os.html#_Toc372793091)-Spezifikation 
für Fehlerantworten.

### JSON-Darstellung

Die Fehlerressource besteht aus den folgenden Ressourcen:

<!-- { "blockType": "resource", "@odata.type": "sample.error" } -->
```json
{
  "error": { "@odata.type": "odata.error" }  
}
```

#### Ressourcentyp „odata.error“

In der Fehlerantwort befindet sich eine Fehlerressource, 
die die folgenden Eigenschaften umfasst:

<!-- { "blockType": "resource", "@odata.type": "odata.error", "optionalProperties": [ "target", "details", "innererror"] } -->
```json
{
  "code": "string",
  "message": "string",
  "innererror": { "@odata.type": "odata.error" }
}
```

| Eigenschaftenname  | Wert                  | Beschreibung                                                                                               |
|:---------------|:-----------------------|:-----------------------------------------------------------------------------------------------------------|
| **code**       | string                 | Eine Fehlercode-Zeichenfolge für den aufgetretenen Fehler                                                            |
| **message**    | string                 | Eine für Entwickler bereitgestellte Meldung über den aufgetretenen Fehler. Diese sollte dem Benutzer selbst nicht direkt angezeigt werden. |
| **innererror** | Fehlerobjekt           | Optional. Zusätzliche Fehlerobjekte, die eventuell spezifischer als der Fehler der obersten Ebene sind.                     |
<!-- {
  "type": "#page.annotation",
  "description": "Understand the error format for the API and error codes.",
  "keywords": "error response, error, error codes, innererror, message, code",
  "section": "documentation",
  "tocPath": "Misc/Error Responses"
} -->

<!--<a name="msg_code_property"> </a> -->

#### Codeeigenschaft

Die `code`-Eigenschaft enthält einen der folgenden möglichen Werte. 
Ihre Apps müssen in der Lage sein, diese Fehler behandeln zu können.

| Code                      | Beschreibung
|:--------------------------|:--------------
| **accessDenied**          | Der Aufrufer hat nicht die Berechtigung zum Ausführen der Aktion. 
| **activityLimitReached**  | Die App oder der Benutzer wurde gedrosselt.
| **generalException**      | Ein nicht angegebener Fehler ist aufgetreten.
| **invalidRange**          | Der angegebene Bytebereich ist ungültig oder nicht verfügbar.
| **invalidRequest**        | Die Anforderung ist fehlerhaft oder falsch.
| **itemNotFound**          | Die Ressource konnte nicht gefunden werden.
| **malwareDetected**       | In der angeforderten Ressource wurde Schadsoftware erkannt.
| **nameAlreadyExists**     | Der angegebene Elementname ist bereits vorhanden.
| **notAllowed**            | Diese Aktion ist im System nicht zulässig.
| **notSupported**          | Die Anforderung wird vom System nicht unterstützt.
| **resourceModified**      | Die zu aktualisierende Ressource wurde geändert, seit der Aufrufer sie zuletzt gelesen hat.Dies ist in der Regel ein eTag-Konflikt.
| **resyncRequired**        | Das Deltatoken ist nicht mehr gültig, und die App muss den Synchronisierungszustand zurücksetzen.
| **serviceNotAvailable**   | Der Dienst ist nicht verfügbar. Wiederholen Sie die Anforderung nach einer Weile. Es ist möglicherweise ein Retry-After-Header vorhanden. 
| **quotaLimitReached**     | Der Benutzer hat sein Kontingent erschöpft.
| **unauthenticated**       | Der Aufrufer konnte nicht authentifiziert werden.

Das `innererror`-Objekt kann rekursiv mehrere `innererror`-Objekte mit weiteren, 
spezifischeren Fehlercodes enthalten. Bei der Behandlung eines Fehlers sollten Apps 
alle verfügbaren Fehlercodes durchlaufen und den detailliertesten Fehlercode, 
den sie interpretieren können, verwenden. Einige der detaillierteren Codes sind 
unten auf dieser Seite aufgeführt.

Um zu überprüfen, ob es sich bei einem Fehlerobjekt um einen erwarteten Fehler handelt, 
müssen Sie die `innererror`-Objekte durchlaufen und dabei nach den erwarteten Fehlercodes suchen. Beispiel:

```csharp
public bool IsError(string expectedErrorCode)
{
    OneDriveInnerError errorCode = this.Error;
    while (null != errorCode)
    {
        if (errorCode.Code == expectedErrorCode)
            return true;
        errorCode = errorCode.InnerError;
    }
    return false;
}
```

Ein Beispiel, wie Fehler ordnungsgemäß behandelt werden, finden Sie unter 
[Fehlercodebehandlung](https://gist.github.com/rgregg/a1866be15e685983b441).

Die `message`-Eigenschaft im Stamm enthält eine Fehlermeldung, die für den Entwickler vorgesehen ist. 
Fehlermeldungen werden nicht lokalisiert und sollten dem Benutzer nicht direkt angezeigt werden. 
Bei der Behandlung von Fehlern sollte der Code keine `message`-Werte als Schlüsselwerte verwenden, 
da sie sich jederzeit ändern können und sie häufig dynamischen Informationen enthalten, 
die für die fehlerhafte Anforderung spezifisch sind. Sie sollten den Code nur in Bezug auf Fehlercodes schreiben, 
die in `code`-Eigenschaften zurückgegeben werden.

#### Detaillierte Fehlercodes
Nachfolgend sind einige weitere Fehler aufgeführt, denen Ihre App in den geschachtelten `innererror`-Objekten 
möglicherweise begegnet. Apps müssen diese Fehler nicht behandeln können, können dies jedoch tun, falls erwünscht. 
Der Dienst kann neue Fehlercodes hinzufügen oder die Rückgabe alter Fehlercodes jederzeit beenden. 
Daher ist es wichtig, dass alle Apps in der Lage sind, die [grundlegenden Fehlercodes](#code-property) zu behandeln.

| Code                               | Beschreibung
|:-----------------------------------|:----------------------------------------------------------
| **accessRestricted**               | Der Zugriff ist auf den Besitzer des Elements beschränkt.
| **cannotSnapshotTree**             | Fehler beim Abrufen eines konsistenten Deltasnapshots. Versuchen Sie es später erneut.
| **childItemCountExceeded**         | Höchstwert für die Anzahl der untergeordneten Elemente wurde erreicht.
| **entityTagDoesNotMatch**          | Das ETag stimmt nicht mit dem aktuellen Wert des Elements überein.
| **fragmentLengthMismatch**         | Die deklarierte Gesamtgröße für dieses Fragment unterscheidet sich von der Uploadsitzung.
| **fragmentOutOfOrder**             | Hochgeladenes Fragment funktioniert nicht ordnungsgemäß.
| **fragmentOverlap**                | Hochgeladenes Fragment überschneidet sich mit den vorhandenen Daten.
| **invalidAcceptType**              | Ungültiger Accept-Typ.
| **invalidParameterFormat**         | Ungültiges Parameterformat.
| **invalidPath**                    | Name enthält ungültige Zeichen.
| **invalidQueryOption**             | Ungültige Abfrageoption.
| **invalidStartIndex**              | Ungültiger Startindex.
| **lockMismatch**                   | Sperrtoken entspricht nicht der vorhandenen Sperre.
| **lockNotFoundOrAlreadyExpired**   | Das Element hat derzeit keine nicht abgelaufene Sperre.
| **lockOwnerMismatch**              | Die ID des Sperrenbesitzers entspricht nicht der angegebenen ID.
| **malformedEntityTag**             | ETag-Heder ist fehlerhaft. ETags müssen in Anführungszeichen eingeschlossene Zeichenfolgen sein.
| **maxDocumentCountExceeded**       | Höchstwert für die Anzahl der Dokumente wurde erreicht.
| **maxFileSizeExceeded**            | Die maximale Dateigröße wurde erreicht.
| **maxFolderCountExceeded**         | Die maximale Anzahl von Ordnern wurde erreicht.
| **maxFragmentLengthExceeded**      | Die maximale Dateigröße wurde erreicht.
| **maxItemCountExceeded**           | Die maximale Anzahl von Elementen wurde erreicht.
| **maxQueryLengthExceeded**         | Die maximale Abfragelänge wurde überschritten.
| **maxStreamSizeExceeded**          | Die maximale Streamgröße wurde überschritten.
| **parameterIsTooLong**             | Der Parameter überschreitet die maximale Länge.
| **parameterIsTooSmall**            | Der Parameter ist kleiner als der Mindestwert.
| **pathIsTooLong**                  | Der Pfad überschreitet die maximale Länge.
| **pathTooDeep**                    | Die maximale Tiefe der Ordnerhierarchie wurde erreicht.
| **propertyNotUpdateable**          | Die Eigenschaft kann nicht aktualisiert werden.
| **resyncApplyDifferences**         | Erneute Synchronisierung erforderlich. Ersetzen Sie alle lokalen Elemente durch die Serverversion (einschließlich gelöschter Elemente), wenn Sie sicher sind, dass der Dienst bei der letzten Synchronisierung mit den lokalen Änderungen aktualisiert wurde. Laden Sie alle lokalen Änderungen hoch, die dem Server noch nicht bekannt sind.
| **resyncRequired**                 | Erneute Synchronisierung erforderlich.
| **resyncUploadDifferences**        | Erneute Synchronisierung erforderlich. Laden Sie alle lokalen Elemente hoch, die der Dienst nicht zurückgegeben hat, und laden Sie alle Dateien hoch, die sich von der Version auf dem Server unterscheiden (wobei Sie beide Kopien beibehalten, wenn Sie nicht sicher sind, welche die aktuelle ist).
| **serviceNotAvailable**            | Der Server kann die aktuelle Anforderung nicht verarbeiten.
| **serviceReadOnly**                | Ressource ist vorübergehend schreibgeschützt.
| **throttledRequest**               | Zu viele Anforderungen.
| **tooManyResultsRequested**        | Zu viele Ergebnisse angefordert.
| **tooManyTermsInQuery**            | Zu viele Ausdrücke in der Abfrage.
| **totalAffectedItemCountExceeded** | Vorgang ist nicht zulässig, da die Anzahl der betroffenen Elemente den Schwellenwert überschreitet.
| **truncationNotAllowed**           | Datenkürzung ist nicht zulässig.
| **uploadSessionFailed**            | Fehler beim der Uploadsitzung.
| **uploadSessionIncomplete**        | Uploadsitzung ist nicht vollständig.
| **uploadSessionNotFound**          | Uploadsitzung nicht gefunden.
| **virusSuspicious**                | Dieses Dokument ist verdächtig und hat möglicherweise ein Virus.
| **zeroOrFewerResultsRequested**    | Es wurden null oder weniger Ergebnisse angefordert.

<!-- ##Additional Resources##

- [Microsoft Graph API release notes and known issues](microsoft-graph-api-release-notes-known-issues.md )
- [Hands on lab: Deep dive into the Microsoft Graph API](http://dev.office.com/hands-on-labs/4585) -->
