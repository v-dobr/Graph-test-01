# Aufrufen von Microsoft Graph in einer iOS-App

In diesem Artikel werden Aufgaben beschrieben, die zum Herstellen einer Verbindung zwischen Ihrer Anwendung und Office 365 und zum Aufrufen der Microsoft Graph-API mindestens erforderlich sind. Wir verwenden den Code aus dem [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample), um die wichtigsten Konzepte zu erläutern, die Sie in Ihrer App implementieren müssen. In diesen Beispielen werden der grundlegende Kern für die Authentifizierung mit Microsoft Azure Active Directory (AAD) und einfache Dienstaufrufe für den Office 365-E-Mail-Dienst mithilfe der Microsoft Graph-API (Senden einer E-Mail) erläutert. Es wird empfohlen, eine Kopie des diesem Artikel beiliegenden Projekts zu erstellen oder das Projekt aus diesem Repository herunterzuladen. 


In diesem Artikel wird auf die [Microsoft Azure Active Directory-Authentifizierungsbibliothek (ADAL) für iOS und OSX](https://github.com/AzureAD/azure-activedirectory-library-for-objc) verwiesen, und das  [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample)-Beispiel veranschaulicht die Verwendung dieser Bibliothek. Weitere Informationen zur Verwendung und Implementierung in Ihrem iOS-Projekt finden Sie in diesem Repository.


## Übersicht

Zum Aufrufen der Microsoft Graph-API muss Ihre iOS-App die folgenden Aufgaben ausführen:

1. Registrieren der Anwendung aus Microsoft Azure Active Directory (AD)
2. Anfordern und Erwerben eines Zugriffstokens aus Azure AD
3. Verwenden des Zugriffstokens in einer REST-Anforderung an die Microsoft Graph-API 



## Registrieren der Anwendung in Azure Active Directory

Bevor Sie  mit Office 365 arbeiten können, müssen Sie Ihre Anwendung registrieren und die Berechtigungen für die Verwendung der Microsoft Graph-Dienste festlegen. 
Mithilfe des [App-Registrierungstools](https://dev.office.com/app-registration) können Sie mit nur wenigen Klicks Ihre Anwendung für den Zugriff auf ein Geschäfts- oder Schulkonto des Benutzers registrieren. Zum Verwalten müssen Sie das [Microsoft Azure-Verwaltungsportal](https://manage.windowsazure.com) verwenden.

Alternativ können Sie die App manuell registrieren. Die Anweisungen dazu finden Sie im Abschnitt **Registrieren der systemeigenen App mit dem Azure-Verwaltungsportal** des Artikels [Manuelles Registrieren der App mit Azure AD, damit Sie auf Office 365-APIs zugreifen kann](https://msdn.microsoft.com/en-us/office/office365/howto/add-common-consent-manually). Beachten Sie dabei Folgendes:

* Für die Registrierung müssen Sie einen Umleitungs-URI angeben. Dies ist ein erforderlicher Wert, der angibt, wohin ein Benutzer nach erfolgreicher Authentifizierung umgeleitet wird. Wenn Sie einen falschen Umleistungs-URI angeben, tritt bei der Authentifizierungsanforderung ein Fehler auf.
* Bei der Registrierung müssen der App **Berechtigungen zum Senden von E-Mails als angemeldeter Benutzer** für **Microsoft Graph** gewährt werden.  


Notieren Sie sich die folgenden Werte auf der Seite **Konfigurieren** Ihrer Azure-Anwendung.

* Client-ID
* Umleitungs-URI

Sie benötigen diese Werte zum Konfigurieren des OAuth-Flusses in Ihrer App. 

## Anfordern und Erwerben eines Zugriffstokens aus Azure AD

Zum Anfordern und Erwerben eines Zugriffstokens für die Microsoft Graph-API können Sie **AcquireAuthTokenWithResource:clientId:redirectUri:completionBlock:** verwenden, der von der [Microsoft Azure Active Directory-Authentifizierungsbibliothek (ADAL) für iOS und OSX](https://github.com/AzureAD/azure-activedirectory-library-for-objc) bereitgestellt wurde. Mit diesem SDK verfügt Ihre Anwendung über die volle Funktionalität von Microsoft Azure AD, einschließlich, dem Branchenstandard entsprechend, Unterstützung für OAuth2, Web-API-Integration mit Zustimmung auf Benutzerebene und zweistufige Authentifizierung.

Diese Methode enthält die folgenden Parameter:

1. **resourceID** Dies ist die erforderliche Ressource, auf die Sie zugreifen möchten. Wenn Sie beispielsweise auf die Microsoft Graph-API zugreifen möchten, lautet dieser Wert „https://graph.microsoft.com.“
2. **clientID** Der angegebene Wert zum Identifizieren Ihrer App nach Abschluss der AAD-Registrierung.
3. **redirectURI** Dies ist ein erforderlicher Wert, der angibt, wohin ein Benutzer nach erfolgreicher Authentifizierung umgeleitet wird.


Sie müssen zunächst einen Authentifizierungskontext angeben. Damit wird die Stelle definiert, von der Sie das Zugriffstoken abrufen möchten. In unserem Fall handelt es sich dabei um einen AAD-Mandanten, und Sie müssen diesen deklarieren:

    @property (strong,    nonatomic) ADAuthenticationContext *context;

Initialisieren Sie diesen dann mit dem Speicherort der entsprechenden Stelle („https://login.microsoftonline.com/common“):

    self.context = [ADAuthenticationContext authenticationContextWithAuthority:self.authority]; 


Im [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample)-Beispiel wird eine Singleton-Authentifizierungsklasse (**AuthenticationManager**) erstellt, um zu veranschaulichen, dass diese mit der entsprechenden Authentifizierungsstelle und den erforderlichen Parametern initialisiert ist. Diese Klasse dienst lediglich als Beispiel für den Authentifizierungsworkflow. Ein für Sie interessanter Codeausschnitt: 



    - (void)acquireAuthTokenWithResource:(NSString *)resourceID
                            clientID:(NSString*)clientID
                         redirectURI:(NSURL*)redirectURI
                          completion:(void (^)(ADAuthenticationError *error))completion {
    
    NSLog(@"acquireAuthTokenWithResource");
    [self.context acquireTokenWithResource:resourceID
                                  clientId:clientID
                               redirectUri:redirectURI
                           completionBlock:^(ADAuthenticationResult *result) {
                               NSLog(@"Completion");
                               
                               if (result.status !=AD_SUCCEEDED){
                                   NSLog(@"error");
                                   completion(result.error);
                               }
                               
                               else{
                                   NSLog(@"complete!");
                                   self.accessToken = result.accessToken;
                                   self.userID = result.tokenCacheStoreItem.userInformation.userId;
                                   completion(nil);
                               }
                           }];


Beim ersten Ausführen dieser App sendet der Authentifizierungs-Manager eine Anforderung an die Authentifizierungsstelle, 
die Sie zur Anmeldeseite weiterleitet. Sie müssen Ihre Anmeldeinformationen angeben und erhalten dann eine Antwort mit dem Authentifizierungsergebnis. 
Wenn die Authentifizierung erfolgreich war, ist in dieser auch das Aktualisierungs- und Zugriffstoken enthalten. Beim zweiten Ausführen dieser Anwendung verwendet der Authentifizierungs-Manager, 
vorausgesetzt Sie haben den Tokencache nicht geleert und die Cookies nicht gelöscht, das Zugriffs- oder Aktualisierungstoken im Cache zum Authentifizieren von Clientanforderungen. 
Dies führt zum Aufrufen des Diensts, falls Sie ein Zugriffstoken abrufen müssen. 


## Verwenden des Zugriffstokens in einer Anforderung an die Microsoft Graph-API

Mit einem Zugriffstoken kann Ihre App authentifizierte Anforderungen an die Microsoft Graph-API senden. Ihre App muss das Zugriffstoken an den HTTP-Anforderungsheader unter **Autorisierung** anfügen.

Im [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample)-Beispiel wird eine E-Mail mithilfe des sendMail-Endpunkts in der Microsoft Graph-API gesendet. In unserem Beispiel wurde eine Singleton-Authentifizierungsklasse (AuthenticationManager) erstellt, die mit dem Zugriffstoken initialisiert wird. Wir benötigen das Zugriffstoken zum Erstellen unserer Anforderung.



    - (void)sendMailREST {
    
    AuthenticationManager *authManager = [AuthenticationManager sharedInstance];

    //Helper method used to construct the email message
    NSData *postData = [self mailContent];
    
    //Microsoft Graph API endpoint for sending mail
    NSMutableURLRequest *request = [[NSMutableURLRequest alloc] initWithURL:[NSURL URLWithString:@"https://graph.microsoft.com/v1.0/me/microsoft.graph.sendmail"]];

    [request setHTTPMethod:@"POST"];
    [request setValue:@"application/json" forHTTPHeaderField:@"Content-Type"];
    [request setValue:@"application/json, text/plain, */*" forHTTPHeaderField:@"Accept"];
    
    // Access token required for request header
    NSString *authorization = [NSString stringWithFormat:@"Bearer %@", authManager.accessToken];
    [request setValue:authorization forHTTPHeaderField:@"Authorization"];
    [request setHTTPBody:postData];

    NSURLConnection *conn = [[NSURLConnection alloc] initWithRequest:request delegate:self];
    
    if(conn) {
        NSLog(@"Connection Successful");
    } else {
        NSLog(@"Connection could not be made");
    }
    
    [conn start];

Wie Sie sehen können, wird die Antwort mit den folgenden NSURLConnection-Delegaten verarbeitet: NSURLConnectionDelegate und NSURLConnectionDataDelegate.

## Nächste Schritte

Wenn das Zugriffstoken abgelaufen ist oder abläuft, können Sie mit **acquireTokenSilentWithResource:clientId:redirectUri:completionBlock:** von ADAuthenticationContext ein neues Zugriffstoken erwerben. Die Verwendung wird im [ios-objectivec-connect-rest-sample](https://github.com/microsoftgraph/ios-objectivec-connect-rest-sample)-Beispiel erläutert. Darüber hinaus finden Sie dort den Code zum Leeren des Cache und Löschen der gespeicherten Cookies.  

Die Microsoft Graph-API ist eine leistungsfähige einheitliche API, die für die Interaktion mit beliebigen Microsoft-Daten verwendet werden kann. Informationen zu weiteren Möglichkeiten mit der Microsoft Graph-API finden Sie in der [API-Referenz](http://graph.microsoft.io/docs/api-reference/v1.0).

