# Déploiements cloud souverains


Cet article fournit des informations sur les différentes instances cloud souveraines de Microsoft Graph, ainsi que sur leurs fonctionnalités disponibles pour les développeurs. 


## Microsoft Graph géré par 21Vianet en Chine

Cet article fournit des informations sur Microsoft Graph géré par 21Vianet, ainsi que sur les fonctionnalités disponibles pour les développeurs.

### Points de terminaison racine de service
| Microsoft Graph géré par 21Vianet | Microsoft Graph|
|---------------------------|----------------|
| https://microsoftgraph.chinacloudapi.cn | https://graph.microsoft.com|

### Explorateur Microsoft Graph
| Microsoft Graph géré par 21Vianet | Microsoft Graph|
|---------------------------|----------------|
|https://graphexplorerchina.azurewebsites.net| https://graphexplorer2.azurewebsites.net|

### Azure OpenID Connect et OAuth2.0
Les points de terminaison utilisés pour acquérir des jetons de connexion ou pour appeler Microsoft Graph géré par 21Vianet diffèrent de ceux des autres offres. 

| Microsoft Graph géré par 21Vianet | Microsoft Graph|
|---------------------------|----------------|
| https://login.chinacloudapi.cn | https://login.microsoftonline.com|
 
Utilisez https://login.chinacloudapi.cn/common/oauth2/authorize pour authentifier l'utilisateur et https://login.chinacloudapi.cn/common/oauth2/token pour obtenir un jeton pour que votre application puisse appeler Microsoft Graph géré par 21Vianet.

> **REMARQUE** : les derniers [points de terminaison de jeton et d’autorisation v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-appmodel-v2-overview/) ne sont PAS disponibles pour une utilisation avec Microsoft Graph géré par 21Vianet.  Les applications peuvent accéder uniquement aux données d'organisation et non de consommation. 

### Fonctionnalités de service proposées par Microsoft Graph géré par 21Vianet
Les fonctionnalités de Microsoft Graph suivantes sont généralement disponibles (sur le point de terminaison `/v1.0`) :

* Users
* Groups
* Files
* Application de messagerie
* Calendrier
* Contacts personnels 
* Créer, lire, mettre à jour et supprimer des opérations
* Prise en charge du partage des ressources d’origine croisée (CORS).

Les fonctionnalités de Microsoft Graph suivantes sont également disponibles dans la version d’évaluation (sur le point de terminaison `/beta`) :

* Contacts organisationnels
* Applications
* Principaux de service
* Extensions du schéma Directory
