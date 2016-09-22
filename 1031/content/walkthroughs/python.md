# Aufrufen von Microsoft Graph in einer Python-App 

In diesem Artikel werden Aufgaben beschrieben, die zum Herstellen einer Verbindung zwischen Ihrer Anwendung und Office 365 und zum Aufrufen der Microsoft Graph-API mindestens erforderlich sind. Wir verwenden den Code aus [Office 365 Python Connect-Beispiel unter Verwendung von Microsoft Graph](https://github.com/microsoftgraph/python3-connect-rest-sample), um die wichtigsten Konzepte zu erläutern, die Sie in Ihrer App implementieren müssen.

![Screenshot des Office 365 Phyton Connect-Beispiels](./images/web-screenshot.png)

##  Anforderungen

Folgendes wird in diesem Thema vorausgesetzt:

* Sie sind mit dem Python-Code zu vertraut.
* Sie sind mit den OAuth-Konzepten vertraut.

## Übersicht

Zum Aufrufen der Microsoft Graph-API muss Ihre Python-App die folgenden Aufgaben ausführen:

1. Registrieren der Anwendung in Azure Active Directory
2. Weiterleiten des Browsers zur Anmeldeseite
3. Erhalten eines Autorisierungscodes auf Ihrer Antwort-URL-Seite
4. Anfordern eines Zugriffstokens vom Endpunkt, der ein Token ausgibt
5. Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API 

<!--<a name="register"></a>-->
## Registrieren der Anwendung in Azure Active Directory

Bevor Sie  mit Office 365 arbeiten können, müssen Sie Ihre Anwendung registrieren und die Berechtigungen für die Verwendung der Microsoft Graph-Dienste festlegen. 
Mithilfe des [App-Registrierungstools](https://dev.office.com/app-registration) können Sie mit nur wenigen Klicks Ihre Anwendung für den Zugriff auf ein Geschäfts- oder Schulkonto des Benutzers registrieren. Zum Verwalten müssen Sie das [Microsoft Azure-Verwaltungsportal](https://manage.windowsazure.com) verwenden.

Alternativ können Sie die App manuell registrieren. Die Anweisungen dazu finden Sie im Abschnitt [Registrieren der Webserver-App mithilfe des Azure-Verwaltungsportals](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp). Beachten Sie dabei Folgendes:

* Stellen Sie sicher, dass Sie „http://127.0.0.1:8000/connect/get_token/“ als **Anmelde-URL** angeben.
* [Konfigurieren Sie nach der Registrierung der Anwendung die **delegierten Berechtigungen**](https://github.com/microsoftgraph/python3-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure), die für Ihre Python-App erforderlich sind. Für das Connect-Beispiel ist die Berechtigung zum **Senden von E-Mails als angemeldeter Benutzer** erforderlich.

Notieren Sie sich die folgenden Werte auf der Seite **Konfigurieren** in Ihrer Azure-Anwendung, da Sie diese Werte zum Konfigurieren des OAuth-Flusses in Ihrer Python-App benötigen.

* Client-ID (eindeutig für Ihre Anwendung)
* Eine Antwort-URL (http://127.0.0.1:8000/connect/get_token/)
* Ein Anwendungsschlüssel (eindeutig für Ihre Anwendung)

<!--<a name="redirect"></a>-->
## Weiterleiten des Browsers zur Anmeldeseite

Die App muss den Browser an die Anmeldeseite weiterleiten, um den OAuth-Fluss zu beginnen und einen Autorisierungscode zu erhalten. 

Im Connect-Beispiel erstellt der folgende Code (befindet sich in [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py)) die URL, die die App an den Benutzer weiterleiten muss und die an die Ansicht weitergeleitet wird, in der sie für die Umleitung verwendet werden kann. 

```python
# This function creates the signin URL that the app will
# direct the user to in order to sign in to Office 365 and
# give the app consent.
def get_signin_url(redirect_uri):
  # Build the query parameters for the signin URL.
  params = { 'client_id': client_id,
             'redirect_uri': redirect_uri,
             'response_type': 'code'
           }

  authorize_url = '{0}{1}'.format(authority, '/common/oauth2/authorize?{0}')
  signin_url = authorize_url.format(urlencode(params))
  return signin_url
```

<!--<a name="authCode"></a>-->
## Erhalten eines Autorisierungscodes auf Ihrer Antwort-URL-Seite

Nachdem der Benutzer sich angemeldet hat, wird der Browser an Ihre Antwort URL, die ```get_token```-Funktion in [*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py), mit einem Autorisierungscode weitergeleitet, der der Abfragezeichenfolge als ```code```-Variable angefügt ist. 

Das Connect-Beispiel ruft den Code aus der Abfragezeichenfolge ab, sodass es diesen anschließend gegen ein Zugriffstoken austauschen kann.

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## Anfordern eines Zugriffstokens vom Endpunkt, der ein Token ausgibt

Nachdem Sie den Autorisierungscode erhalten haben, können Sie diesen zusammen mit den Werten für die Client-ID, den Schlüssel und die Antwort-URL verwenden, die Sie von Azure Active Directory zum Anfordern eines Zugriffstokens erhalten haben. 

> **Hinweis** Die Anforderung muss auch eine Ressource angeben, die Sie nutzen möchten. Im Falle von Microsoft Graph lautet der Ressourcenwert `https://graph.microsoft.com`.

Das Connect-Beispiel fordert ein Token in der ```get_token_from_code```-Funktion in der Datei [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py) an.

```python
# This function passes the authorization code to the token
# issuing endpoint, gets the token, and then returns it.
def get_token_from_code(auth_code, redirect_uri):
  # Build the post form for the token request
  post_data = { 'grant_type': 'authorization_code',
                'code': auth_code,
                'redirect_uri': redirect_uri,
                'client_id': client_id,
                'client_secret': client_secret,
                'resource': 'https://graph.microsoft.com'
              }
              
  r = requests.post(token_url, data = post_data)
  
  try:
    return r.json()
  except:
    return 'Error retrieving token: {0} - {1}'.format(r.status_code, r.text)
```

> **Hinweis** Die Antwort bietet mehr Informationen als nur das Zugriffstoken. Beispielsweise kann Ihre App ein Aktualisierungstoken abrufen, um neue Zugriffstoken anzufordern, ohne dass sich der Benutzer erneut explizit anmelden muss.

<!--<a name="request"></a>-->
## Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API

Mit einem Zugriffstoken kann Ihre App authentifizierte Anforderungen an die Microsoft Graph-API senden. Ihre App muss das Zugriffstoken an den **Autorisierungs**-Header jeder Anforderung anfügen.

Im Connect-Beispiel wird eine E-Mail mithilfe des ```me/microsoft.graph.sendMail```-Endpunkts in der Microsoft Graph-API gesendet. Der Code ist in der ```call_sendMail_endpoint```-Funktion in der Datei [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py) vorhanden. Dies ist der Code, der zeigt, wie dem Header „Autorisierung“ der Zugriffscode angehängt wird.

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **Hinweis** Die Anforderung muss auch einen Header des Typs **Content-Type** mit einem Wert senden, der von der Graph-API, z. B. `application/json`, akzeptiert wird.

Die Microsoft Graph-API ist eine leistungsfähige einheitliche API, die für die Interaktion mit beliebigen Microsoft-Daten verwendet werden kann. Informationen zu weiteren Möglichkeiten mit der Microsoft Graph-API finden Sie in der API-Referenz.

<!--
## Additional resources

-  [Office 365 Python Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Python-Microsoft-Graph-Connect)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference]()-->
