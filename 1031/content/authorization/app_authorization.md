
# Microsoft Graph-App-Autorisierung


In diesem Artikel wird erläutert, wie ein Benutzer authentifiziert, ein Zugriffstoken abgerufen und ein Zugriffstoken mithilfe eines Aktualisierungstokens erneuert wird. 
Der Authentifizierungsablauf wird in zwei grundlegende Schritte unterteilt:

1. Anfordern eines Autorisierungscodes
2. Verwenden des Autorisierungscodes zum Anfordern eines Zugriffstokens und eines Aktualisierungstokens 

>  **Hinweis**: Mit dem Aktualisierungstoken können Sie ein neues Zugriffstoken anfordern, wenn das aktuelle Zugriffstoken abläuft.

<!--To call the Microsoft Graph API, you have to complete the following tasks.

1. Register the application in Azure Active Directory
2. Authenticate a user and get an access token by calling methods on the Azure AD Authentication Library (ADAL)
3. Use ADAL to get an access token
4. Use the access token in a request to the Microsoft Graph API
5. Disconnect the session

In this article:

- [Authenticate a user and get app authorized](#msg_get_app_authorized)
- [Acquire access token](#msg_get_app_authenticated)
- [Renew access token using refresh token](#msg_renew_access_token)

 <a name="msg_get_app_authorized"> </a> -->
 
###Authentifizieren eines Benutzers und der App
Um Ihre App zu autorisieren, müssen Sie zunächst den Benutzer authentifizieren. Zu diesem Zweck leiten Sie den Benutzer zum Azure Active Directory-Endpunkt (Azure AD) zusammen mit Ihren App-Informationen um, um ihn beim Office 365-Konto anzumelden. 
Nachdem der Benutzer angemeldet ist und den von der App angeforderten Berechtigungen zustimmt (sofern der Benutzer dies nicht bereits getan hat), erhält Ihre App einen Autorisierungscode, der zum Anfordern eines OAuth-Zugriffstokens erforderlich ist.

> **Hinweis**:  Sie erreichen dies durch Aufrufen von Methoden für die [Azure AD-Authentifizierungsbibliothek (ADAL)](https://msdn.microsoft.com/en-us/library/azure/jj573266.aspx). Weitere Informationen zum Autorisierungsablauf finden Sie unter [Authorization Code Grant-Datenfluss](https://msdn.microsoft.com/en-us/library/azure/dn645542.aspx).

Das Autorisierung einer App beginnt durch das Senden einer HTTPS GET-Anforderung mithilfe der folgenden URL:
 
```GET https://login.microsoftonline.com/common/oauth2/authorize?response_type=code&redirect_uri=<uri>&client_id=<id>```

**Erforderliche Parameter der Abfragezeichenfolge**

| Parametername  | Wert  | Beschreibung                                                                                            |
|:----------------|:-------|:-------------------------------------------------------------------------------------------------------|
| *client_id*     | string | Die für Ihre App erstellte Client-ID. Dies ist der Wert **CLIENT-ID** Ihrer App, der in der Anwendungsregistrierung des Azure-Mandanten festgelegt ist.                                                                  |
| *response_type* | string | Gibt den angeforderten Antworttyp an. In einer Anforderung zur Gewährung eines Autorisierungscodes muss der Wert ein Code sein. |
| *redirect_uri*  | string | Die Umleitungs-URL, an die der Browser gesendet wird, wenn die Authentifizierung abgeschlossen ist.  Dieser Wert muss dem vorkonfigurierten Wert **Antwort-URL** der App entsprechen.                        |
 


Im Folgenden finden ein Beispiel für eine solche Anforderung, wie sie in einer laufenden Anwendung implementiert ist:


```GET https://login.microsoftonline.com/common/oauth2/authorize?response_type=code&redirect_uri=http%3a%2f%2flocalhost:1339/auth/azureoauth/callback&client_id=8b8539cd-7b75-427f-bef1-4a6264fd4940``` 

Diese Anforderung gibt eine `200 OK`-Antwort zurück und zeigt die Azure AD-Kontoanmeldeseite an. 

Nachdem sich der Benutzer mit gültigen Anmeldeinformationen angemeldet und den Berechtigungen für die App zugestimmt hat, sendet die Anmeldeseite eine `POST`-Anfrage an Azure Wenn die oben genannte Anfrage erfolgreich ist, 
antwortet Azure mit einer `302 Found`-Nachricht für das Weiterleiten des Anrufs an den Umleitungs-URI der App, damit die App das erforderliche Zugriffstoken erhält. Der weitergeleitete URI, der im `Location`-Header der Antwort angegeben ist, 
entspricht der REPLY URL der App mit zwei angehängten Abfrageparametern, `code=...` und `session_state=...`. Das folgende Beispiel zeigt einen Auszug aus einer solchen Antwort: 

```no-highlight 
HTTP/1.1 302 Found
Cache-Control: no-cache, no-store
Pragma: no-cache
Content-Type: text/html; charset=utf-8
Expires: -1
Location: http://localhost:1339/auth/azureoauth/callback?code=AAABAAAAvPM...&session_state=a9556cd3-cae6-4bc9-bf51-672f7b79b7c6
Server: Microsoft-IIS/8.5
P3P: CP="DSP CUR OTPi IND OTRi ONL FIN"

..... 
```

In diesem Beispiel ist die Antwort-URL der App `http://localhost:1339/auth/azureoauth/callback`. 
Bei der Verarbeitung dieser Antwort extrahieren Sie den `code`-Parameterwert und verwenden ihn, um die anfänglichen OAuth 2.0-Zugriffs- und -Aktualisierungstoken abzurufen (siehe nächsten Abschnitt).

> Die `302 Found`-Antwort oben unterscheidet sich von der `302 Found`-Antwort, die Sie erhalten, wenn Sie den Anmeldevorgang für die `https://login.windows.net/<tenantId>/oauth2/authorize?...`-URL starten. 
Im letztgenannten Fall leitet die `302 Found`-Antwort Ihre Anforderung an `login.microsoftonline.com` um.
 
<!---<a name="msg_get_app_authenticated"> </a> -->

###Abrufen eines Zugriffstokens
Um auf Microsoft Graph-API-Ressourcen zugreifen zu können, muss Ihre App ein gültiges OAuth 2.0-Zugriffstoken in jeder HTTP-Anforderung enthalten. Sie können das Zugriffstoken mit der folgenden POST-Anforderung erhalten:

```no-highlight 
POST https://login.microsoftonline.com/common/oauth2/token HTTP/1.1
content-type : application/x-www-form-urlencoded
content-length : 144
```
 
Diese Anforderung erfordert eine URL-codierte Nutzlast im folgenden Format:
 
```no-highlight 
grant_type=authorization_code
&redirect_uri=<uri>
&client_id=<id>
&client_secret=<secret_key>
&code=<code>
&resource=https%3A%2F%2Fgraph.microsoft.com%2F
```

**Erforderliche Parameter der Abfragezeichenfolge**

| Parametername  | Wert  | Beschreibung                                                                                            |
|:----------------|:-------|:-------------------------------------------------------------------------------------------------------|
| *client_id*     | string | Die für Ihre App erstellte Client-ID.  |
| *client_secret*  | string | Der für Ihre App erstellte Schlüssel. Dieser Wert ist mit dem Wert im Abschnitt **Schlüssel** der Seite für die App-Konfiguration im Azure-Verwaltungsportal identisch.|
| *redirect_uri*  | string | Die Umleitungs-URL, an die der Browser gesendet wird, wenn die Authentifizierung abgeschlossen ist.  |
| *code*  | string | Der Autorisierungscode. Der `code`-Abfrageparameterwert, der von der Antwort an die Autorisierungsanforderung zurückgegeben wird. |
| *resource*   | string | Die Ressource, auf die Sie zugreifen möchten. Legen Sie diesen Parameterwert auf "https://graph.microsoft.com/" fest, um die Microsoft Graph-API aufzurufen.|

Der folgende Ausschnitt zeigt ein Beispiel für die Anforderungsnutzlast, die verwendet wird, um das anfängliche OAuth 2.0-Zugriffstoken zu erhalten:

```no-highlight  
grant_type=authorization_code&
redirect_uri=http%3A%2F%2Flocalhost%3A1339%2Fauth%2Fazureoauth%2Fcallback&
client_id=8b8539cd-7b75-427f-bef1-4a6264fd4940&client_secret=PJW3dznGfyNSm3rM9aHeXWGKsTMepKXT1Eqy45XXdU4%3D&
code=AAABAAAAvPM1KaPlrEqdFSBzjqfTGBLRVQc6BtQmJ_9DQZUg8Tb7TJgTmbTE7AHM93qB5EKc4Bf-bOgzt3mebAywK-09X1uBHwOluuqSWfd9LU2HHgZtxcZFIYI5UL7L1UEvhrJRvX2iHhfz9ZSRMZMVL55n_K79gCOxtSATeCUw52zPk5ZaQ87Y42SCLsRZN4Y_zddhD3mMpkObiHVT8HzfhBUiT0oX0e-Q439vkbZoKiq1HaqMR3IPHiCXDbPPH5u7a4NTe5xAhh-o2MUIe6s4Xqql86sv1-IwyroOJJMueGUarkfbgwqmYL9Tm-jWab8o-BAK_plVsN73GU8cXO8ts30wa2YmNR5ZxSkw8oiB4mSZwGzGQlch55DRnucDs0SZBgj5etGi3SeXv5jhKlDU2S0bAPnGxF3QAH0N_zBpfakETVlcsHKi714u-tn9da6aTPQsE0IYKTAYgxjTMei6zfRFvCZi-tKdFR6X9TvvmF2iPdGQGWKeLw8CMWUzU8VmOhiHc0aBKG6RaXAOTM067J_9WKYPxMopcytD2z8HVkL1QhggAA&
resource=https%3A%2F%2Fgraph.microsoft.com%2F
```

Wenn diese Anforderung erfolgreich ist, wird eine `200 OK`-Antwort zurückgegeben.

```no-highlight  
HTTP/1.1 200 OK
Content-Type: application/json; charset=utf-8
Expires: -1
Content-Length: 2978
Access-Control-Allow-Origin: *

{
    "token_type":"Bearer",
    "expires_in":"3599",
    "expires_on":"1426551729",
    "not_before":"1426547829",
    "resource":"https://graph.microsoft.com/",
    "access_token":"eyJ0eXAiOiJKV1QiLCJhb...",
    "refresh_token":"AAABAAAAvPM1KaPlrEqd...",
    "scope": "Calendar.ReadWrite Directory.Read.All Files.ReadWrite Group.ReadWrite.All Mail.ReadWrite Mail.Send User.ReadBasic.All",
    "id_token":"eyJ0eXAiOiJKV1QiLCJhbGci..."
}
```

 
Der Antworttext ist eine JSON-formatierte Zeichenfolge, die das Zugriffstoken (`access_token`) enthält. 
Sie müssen dieses Token bereitstellen, damit alle nachfolgenden HTTP-Anforderungen auf Microsoft Graph-API-Ressourcen zugreifen können.

Der `scope`-Eigenschaftswert sollte den Berechtigungen entsprechen, die für die App bei der Registrierung der App erteilt werden.

Das Zugriffstoken bleibt innerhalb des angegebenen Zeitintervalls (`3599` im vorangehenden Beispiel) Sekunden (oder 1 Stunde) ab dem Zeitpunkt der Ausstellung gültig, 
wie in der `expires_in`-Eigenschaft angegeben. Das Ergebnis enthält darüber hinaus ein Aktualisierungstoken (`refresh_token`), das verwendet werden muss, um ein ablaufendes oder abgelaufenes Zugriffstoken zu erneuern. 

Im Produktionscode muss Ihre App auf den Ablauf dieser Token achten und das ablaufende Zugriffstoken vor Ablauf des Aktualisierungstoken erneuern. 


<!---<a name="msg_renew_access_token using refresh token"> </a> -->

###Erneuern des ablaufenden Zugriffstokens mit dem Aktualisierungstoken
Verwenden Sie zum Aktualisieren eines abgelaufenen Zugriffstokens eine POST-Anforderung ähnlich wie im folgenden Beispiel (vorausgesetzt, dass das Aktualisierungstoken nicht abgelaufen ist):

```no-highlight  
POST https://login.microsoftonline.com/common/oauth2/token HTTP/1.1
Host: login.microsoftonline.com
Content-Type: application/x-www-form-urlencoded
Content-Length: 897


grant_type=refresh_token
&redirect_uri=http%3A%2F%2Flocalhost%3A1339%2Fauth%2Fazureoauth%2Fcallback
&client_id=8b8539cd-7b75-427f-bef1-4a6264fd4940
&client_secret=PJW3dznGfyNSm3rM9aHeXWGKsTMepKXT1Eqy45XXdU4%3D
&refresh_token=AAABAAAAvPM1KaPlrEqdFSBzjqfTGM74--...
&resource=https%3A%2F%2Fgraph.microsoft.com%2F
```

**Erforderliche Parameter der Abfragezeichenfolge**

| Parametername  | Wert  | Beschreibung                                                                                                                                         |
|:----------------|:-------|:----------------------------------------------------------------------------------------------------------------------------------------------------|
| *client_id*     | string | Die für Ihre Anwendung erstellte Client-ID.  |
| *redirect_uri*  | string | Die Umleitungs-URL, an die der Browser gesendet wird, wenn die Authentifizierung abgeschlossen ist. Diese sollte mit dem *redirect_uri*-Wert übereinstimmen, der in der ersten Anforderung verwendet wird. |
| *client_secret* | string | Einer der für die Anwendung erstellten Schlüsselwerte.                                                                                                     |
| *refresh_token* | string | Das zuvor erhaltene Aktualisierungstoken.    |
| *Ressourcen*      | string | Die Ressource, auf die Sie zugreifen möchten.|

Beachten Sie, dass diese Anforderung mit der anfänglichen Tokenabrufanforderung beinahe identisch ist. Es bestehen zwei Unterschiede hinsichtlich der Anforderungsnutzlast. 
Der `grant_type`-Parameter hat jetzt nämlich den Wert `refresh_token` (anstelle von `code`).
 
Die erfolgreiche Antwort gibt die Nutzlast einer JSON-Zeichenfolge ähnlich der folgenden Ausgabe zurück:

```no-highlight 
{
    "token_type":"Bearer",
    "expires_in":"3600",
    "expires_on":"1426561346",
    "not_before":"1426557446",
    "resource":"https://graph.microsoft.com/",
    "access_token":"eyJ0eXAiOiJKV1QiLCJhbGciOi...", 
    "refresh_token":"AAABAAAAvPM1KaPlrEqdFSBzj...",
   "scope":"Graph.Read",
    "pwd_exp":"6553342",
    "pwd_url":"https://portal.microsoftonline.com/ChangePassword.aspx"
}
```
 
Mit Ausnahme der fehlenden `id_token`-Eigenschaft hat dieser Antworttext die gleiche Syntax und Semantik wie der Text der anfänglichen Tokenanforderungsantwort. 
Die Nutzungszeiten der neu zurückgegebenen `access_token`- und `refresh_token`-Werte werden verlängert. 
Die neue Ablaufzeit des Zugriffstokens entspricht der Anzahl von Sekunden, die im `expires_in`-Wert angegeben ist, 
ab dem Zeitpunkt, als die Tokenaktualisierungsanforderung erfolgreich übermittelt wurde. 
 
Wenn das Aktualisierungstoken abgelaufen ist, können Sie kein abgelaufenes Zugriffstoken mehr mithilfe der eben beschriebenen POST-Anforderung erneuern. 
Sie müssen stattdessen den Prozess der [App-Autorisierung und -Authentifizierung](#msg_get_app_authorized) neu starten.


<!--##Additional Resources##

- [Hands on lab: Deep dive into the Office 365 unified API](http://dev.office.com/hands-on-labs/4585)  -->

