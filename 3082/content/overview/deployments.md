# Implementaciones de nube soberana


Este artículo proporciona información sobre las diferentes instancias de nube soberana de Microsoft Graph y las funcionalidades que están disponibles para los desarrolladores. 


## Microsoft Graph operado por 21Vianet en China

En este artículo se proporciona información sobre Microsoft Graph ofrecido por 21Vianet y las funcionalidades que están disponibles para los desarrolladores.

### Puntos de conexión de raíz del servicio
| Microsoft Graph operado por 21Vianet | Microsoft Graph|
|---------------------------|----------------|
| https://microsoftgraph.chinacloudapi.cn | https://graph.microsoft.com|

### Explorador de Microsoft Graph
| Microsoft Graph operado por 21Vianet | Microsoft Graph|
|---------------------------|----------------|
|https://graphexplorerchina.azurewebsites.net| https://graphexplorer2.azurewebsites.net|

### OpenID Connect y OAuth 2.0 de Azure
Los extremos que se utilizan para adquirir tokens de inicio de sesión o para llamar a Microsoft Graph ofrecido por 21Vianet difieren de las de otras ofertas. 

| Microsoft Graph operado por 21Vianet | Microsoft Graph|
|---------------------------|----------------|
| https://login.chinacloudapi.cn | https://login.microsoftonline.com|
 
Use https://login.chinacloudapi.cn/common/oauth2/authorize para autenticar el usuario y https://login.chinacloudapi.cn/common/oauth2/token para adquirir un token para su aplicación para llamar a Microsoft Graph operado por 21Vianet.

> **NOTA**: Los [puntos de conexión de tokens y de autorización de v2.0](https://azure.microsoft.com/en-us/documentation/articles/active-directory-appmodel-v2-overview/) más recientes NO están disponibles para su uso con Microsoft Graph operado por 21Vianet.  Las aplicaciones solo pueden acceder a los datos de la organización y no a los datos del cliente. 

### Funcionalidades del servicio ofrecidas por Microsoft Graph operado por 21Vianet
Las siguientes características de Microsoft Graph están generalmente disponibles (en el punto de conexión de `/v1.0`):

* Usuarios
* Groups
* Files
* Correo
* Calendario
* Contactos personales 
* Crear, leer, actualizar y eliminar (CRUD) operaciones 
* Compatibilidad para compartir recursos de origen cruzado (CORS).

Las siguientes características de Microsoft Graph también están disponibles en versión preliminar (en el punto de conexión de `/beta`):

* Contactos de la organización
* Aplicaciones
* Entidades de servicio
* Extensiones de esquema de directorio
