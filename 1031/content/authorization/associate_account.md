<!---<a name="bk_CreateAzureSubscription"> </a>-->

# Zuordnen des Office 365-Kontos zu Azure AD zum Erstellen und Verwalten von Apps

Um die Anwendung zu authentifizieren, müssen Sie diese in Microsoft Azure Active Directory (Azure AD) registrieren. Hier werden die Office 365-Benutzerkonto- und Anwendungsinformationen gespeichert. Zum Verwalten von Azure AD über das Azure-Verwaltungsportal benötigen Sie ein Microsoft Azure-Abonnement. Mithilfe des Verwaltungsportals in Microsoft Azure können Sie Benutzer, Rollen und Apps verwalten. 

Dieser Artikel führt Sie durch das Zuordnen Ihres Office 365-Kontos zu Azure AD, damit Sie Apps erstellen und verwalten können.


## Anforderungen

**Office 365 for Business-Konten**

Wenn Sie über kein Office 365 for Business-Konto verfügen, müssen Sie folgende Aktionen durchführen:

- Registrieren Sie sich für einen oben aufgeführten [Office 365 for Business-Plan](http://products.office.com/en-us/business/compare-office-365-for-business-plans), oder
- [Nehmen Sie am Office 365 Entwicklerprogramm teil, und erhalten Sie ein kostenloses Abonnement für Office 365 für 1 Jahr](https://aka.ms/devprogramsignup).

**Microsoft Azure-Abonnement** 

- Wenn Sie über ein Microsoft Azure-Abonnement verfügen, können Sie dieses zu Ihrem Office 365 for Business-Abonnement zuordnen. 

- Andernfalls müssen Sie ein neues Azure-Abonnement erstellen und dieses Ihrem Office 365-Konto zuordnen, um Apps zu registrieren und zu verwalten.


<!---<a name="bk_AssociateExistingAzureSubscription"> </a>-->

## So ordnen Sie ein vorhandenes Azure-Abonnement zu Ihrem Office 365-Konto zu


1. Melden Sie sich bei dem [Microsoft Azure-Verwaltungsportal](https://manage.windowsazure.com) mit Ihren vorhandenen Azure-Anmeldeinformationen an (z. B. Microsoft-ID wie user@live.com).
        
2. Wählen Sie **Active Directory**, und wählen Sie dann die Registerkarte **Verzeichnis**, und wählen Sie am unteren Rand des Bildschirms **Neu**. 
     
4. Wählen Sie im Menü **Neu**die Option **Active Directory**  >  **Verzeichnis**  >  **Benutzerdefiniert erstellen**.
    
5. Wählen Sie unter **Verzeichnis hinzufügen** in der Dropdownliste **Verzeichnis** die Option **Vorhandenes Verzeichnis verwenden**. Aktivieren Sie das Kontrollkästchen **Ich bin jetzt für die Abmeldung bereit** in der unteren rechten Ecke. 
    
    Damit kehren Sie zum Azure-Verwaltungsportal zurück.
        
3. Melden Sie sich mit Ihren Office 365-Kontoinformationen an. 
    
    Sie werden gefragt, ob Sie das Verzeichnis mit Azure verwenden möchten. 
    
    **Hinweis** Zum Zuordnen Ihres Office 365-Kontos zu Azure AD benötigen Sie ein Office 365 Business-Konto mit globalen Administratorrechten. 
    
        
4. Wählen Sie  **Weiter**, und klicken Sie dann auf **Jetzt abmelden**.
        
5. Schließen Sie den Browser, und öffnen Sie das [Portal](https://manage.windowsazure.com) erneut. Andernfalls erhalten Sie die Fehlermeldung, dass der Zugriff verweigert wurde.
    
        
6. Melden Sie sich erneut mit Ihren vorhandenen Azure-Anmeldeinformationen an (z. B. Microsoft-ID wie user@live.com). Navigieren Sie zum Knoten **Active Directory**. Unter **Verzeichnis** sollte jetzt Ihr Office 365-Konto aufgeführt sein.
    

<!--<a name="bk_AssociateNewAzureSubscription"> </a>-->

## So erstellen Sie ein neues Azure-Abonnements und ordnen es Ihrem Office 365-Konto zu


1. Melden Sie sich bei Office 365 an. Wählen Sie auf der Seite **Start** das Symbol **Admin**, um das Office 365 Admin Center zu öffnen.
2. Scrollen Sie auf der Menüseite auf der linken Seite bis **Admin** nach unten, und wählen Sie **Azure AD**.

    **Hinweis** Zum Öffnen des Office 365 Admin Centers und zum Zugreifen auf Azure AD benötigen Sie ein Office 365 Business-Konto mit globalen Administratorrechten. 
    
3. Erstellen Sie ein neues Abonnement.
        
    Wenn Sie eine Testversion von Office 365 verwenden, wird eine Meldung mit dem Hinweis angezeigt, dass Azure AD nur für Kunden mit kostenpflichtigen Diensten verfügbar ist. Sie können ein 30-tägiges kostenloses Azure-Abonnement erstellen, Sie müssen dann jedoch einige zusätzliche Schritte ausführen:
    
    1. Wählen Sie Ihr Land oder Ihre Region aus, und klicken Sie dann auf **Azure-Abonnement**.
    2. Geben Sie ihre persönlichen Daten an. Geben Sie zu Verifizierungszwecken eine Telefonnummer ein, unter der Sie erreichbar sind, und geben Sie an, ob diese per Textnachricht oder Anruf erfolgen soll.
    3. Geben Sie, nachdem Sie den Verifizierungscode erhalten haben, diesen ein, und klicken Sie auf **Code überprüfen**.
    4. Geben Sie Zahlungsinformationen ein, überprüfen Sie die Vereinbarung, und wählen Sie **Registrieren**.
        
        Ihre Kreditkarte wird nicht belastet.
        
        Schließen bzw. aktualisieren Sie nicht Ihren Browser, während Ihr Azure-Abonnement erstellt wird.
            
4. Wählen Sie nach Erstellung des Azure-Abonnements **Portal** aus.
        
5. Azure Tour wird angezeigt. Sie können Azure Tour anzeigen oder**X** wählen, um es zu schließen.
        
    Jetzt sollten alle Elemente im Azure-Abonnement angezeigt werden. Es wird ein Verzeichnis mit dem Namen Ihres Office 365-Mandanten aufgeführt.
    
