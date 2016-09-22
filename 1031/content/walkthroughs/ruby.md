# Aufrufen von Microsoft Graph in einer Ruby-App 

In diesem Artikel werden Aufgaben beschrieben, die zum Abrufen eines Zugriffstokens aus Azure Active Directory (AD) und Aufrufen der Microsoft Graph-API mindestens erforderlich sind. Wir verwenden den Code aus dem [Office 365 Ruby Connect-Beispiel unter Verwendung von Microsoft Graph](https://github.com/microsoftgraph/ruby-connect-rest-sample), um die wichtigsten Konzepte zu erläutern, die Sie in Ihrer App implementieren müssen.

![Screenshot des Office 365 Ruby Connect-Beispiels](./images/web-screenshot.png)

## Übersicht

Zum Aufrufen der Microsoft Graph-API muss Ihre Ruby-App die folgenden Aufgaben ausführen:

1. Registrieren der Anwendung in Azure Active Directory
2. Weiterleiten des Browsers zur Anmeldeseite
3. Erhalten eines Autorisierungscodes auf Ihrer Antwort-URL-Seite
4. Anfordern eines Zugriffstokens vom Tokenendpunkt
5. Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API 

<!--<a name="register"/>-->
## Registrieren der Anwendung in Azure Active Directory

Bevor Sie  mit Office 365 arbeiten können, müssen Sie Ihre Anwendung registrieren und die Berechtigungen für die Verwendung der Microsoft Graph-Dienste festlegen. 
Mithilfe des [App-Registrierungstools](https://dev.office.com/app-registration) können Sie mit nur wenigen Klicks Ihre Anwendung für den Zugriff auf ein Geschäfts- oder Schulkonto des Benutzers registrieren. Zum Verwalten müssen Sie das [Microsoft Azure-Verwaltungsportal](https://manage.windowsazure.com) verwenden.

Alternativ können Sie die App manuell registrieren. Die Anweisungen dazu finden Sie im Abschnitt [Registrieren der Webserver-App mithilfe des Azure-Verwaltungsportals](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp). Beachten Sie dabei Folgendes:

* Geben Sie eine Route in Ihrer Ruby-App als die **Anmelde-URL** in Schritt 6 an. Im Connect-Beispiel ist dies [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L41).
* [Konfigurieren Sie die **delegierten Berechtigungen**](https://github.com/microsoftgraph/ruby-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure), die für Ihre App erforderlich sind. Für das Connect-Beispiel ist die Berechtigung zum **Senden von E-Mails als angemeldeter Benutzer** erforderlich.

Notieren Sie sich die folgenden Werte auf der Seite **Konfigurieren** Ihrer Azure-Anwendung.

* Client-ID
* Ein gültiger Schlüssel
* Eine Antwort-URL

Sie benötigen diese Werte als Parameter im OAuth-Fluss in Ihrer App.

<!--<a name="redirect"/>-->
## Weiterleiten des Browsers zur Anmeldeseite

Die App muss den Browser an die Anmeldeseite weiterleiten, um einen Autorisierungscode zu erhalten und den OAuth-Fluss fortzusetzen.

Im Connect-Beispiel wird die Weiterleitung von der OmniAuth-Bibliothek behandelt. Unsere App delegiert die Ausführung an die [`/auth/azureactivedirectory`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L30)-Weiterleitung, die von OmniAuth verwaltet wird.

<!--<a name="authcode"/>-->
## Erhalten eines Autorisierungscodes auf Ihrer Antwort-URL-Seite

Nachdem der Benutzer sich angemeldet hat, gibt der Fluss den Browser an die Antwort-URL in Ihrer App zurück. Azure fügt der Abfragezeichenfolge einen Autorisierungscode hinzu. Beispiel für Connect verwendet die [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L38) Route für diesen Zweck.

Der Autorisierungscode ist in der `code`-Variablen für die Abfragezeichenfolge verfügbar. Das Connect-Beispiel speichert den Code in einer Sitzungsvariable für die spätere Verwendung.

```ruby
code = params[:code]
```

<!--<a name="accesstoken"/>-->
## Anfordern eines Zugriffstokens vom Tokenendpunkt

Nachdem Sie den Autorisierungscode erhalten haben, können Sie diesen zusammen mit den Werten für die Client-ID, den Schlüssel und die Antwort-URL verwenden, die Sie von Azure AD zum Anfordern eines Zugriffstokens erhalten haben. 

> **Hinweis:** <br />
> Außerdem muss die Anforderung eine Ressource angeben, die wir nutzen möchten. Im Falle von Microsoft Graph lautet der Ressourcenwert `https://graph.microsoft.com`.

Das Connect-Beispiel delegiert diese Aufgabe wieder an die OmniAuth-Bibliothek. Die [`acquire_access_token`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L65)-Funktion ruft die Bibliothek auf und übergibt den im vorherigen Abschnitt gespeicherten Authentifizierungscode zusammen mit der Antwort-URL, der Client-ID, dem geheimen Clientschlüssel und der Ressourcen-ID.

```ruby
def acquire_access_token(auth_code, reply_url)
    AUTH_CTX.acquire_token_with_authorization_code(
                  auth_code,
                  reply_url,
                  CLIENT_CRED,
                  GRAPH_RESOURCE)
end
```

> **Hinweis:** <br />
> Die Client-ID und der geheime Clientschlüssel sind im `CLIENT_CRED`-Parameter im vorherigen Codeausschnitt angegeben.

<!--<a name="request"/>-->
## Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API

Mit einem Zugriffstoken kann Ihre App authentifizierte Anforderungen an die Microsoft Graph-API senden. Ihre App muss das Zugriffstoken an den **Autorisierungs**-Header jeder Anforderung anfügen.

Im Connect-Beispiel wird eine E-Mail mithilfe des **sendMail**-Endpunkts in der Microsoft Graph-API gesendet. Der Code ist in der Funktion [`send_mail`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L82) enthalten. Dies ist der Code, der zeigt, wie der Zugriffscode im Header „Autorisierung“ gesendet wird.

```ruby
def send_mail
  # Used in the template
  @name = session[:name]
  @email = params[:specified_email]
  @recipient = params[:specified_email]
  @mailSent = false
  
  sendMailEndpoint = URI("#{GRAPH_RESOURCE}#{SENDMAIL_ENDPOINT}")
  http = Net::HTTP.new(sendMailEndpoint.host, sendMailEndpoint.port)
  http.use_ssl = true
  
  emailBody = File.read("app/assets/MailTemplate.html")
  emailBody.sub! "{given_name}", @name
  
  emailMessage = "{
          Message: {
          Subject: 'Welcome to Office 365 development with Ruby',
          Body: {
              ContentType: 'HTML',
              Content: '#{emailBody}'
          },
          ToRecipients: [
              {
                  EmailAddress: {
                      Address: '#{@recipient}'
                  }
              }
          ]
          },
          SaveToSentItems: true
          }"

  response = http.post(
    SENDMAIL_ENDPOINT, 
    emailMessage, 
    initheader = 
    {
      "Authorization" => "Bearer #{session[:access_token]}", 
      "Content-Type" => CONTENT_TYPE
    }
  )

  # The send mail endpoint returns a 202 - Accepted code on success
  if response.code == "202"
    @mailSent = true
  else
    @mailSent = false
    flash[:httpError] = "#{response.code} - #{response.message}"
  end
  
  render "callback"
end
```

> **Hinweis:** <br />
> Die Anforderung muss auch einen Header des Typs **Content-Type** mit einem Wert senden, der von der Microsoft Graph-API, z. B. `application/json;odata.metadata=minimal;odata.streaming=true`, akzeptiert wird.

Die Microsoft Graph-API ist eine leistungsfähige einheitliche API, die für die Interaktion mit beliebigen Microsoft-Daten verwendet werden kann. Informationen zu weiteren Möglichkeiten mit der Microsoft Graph-API finden Sie in der API-Referenz.

<!--## Additional resources

-  [Office 365 Ruby Connect sample using Microsoft Graph](https://github.com/microsoftgraph/ruby-connect-rest-sample)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference](http://graph.microsoft.io/en-us/docs)-->
