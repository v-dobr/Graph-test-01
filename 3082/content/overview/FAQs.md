
# Preguntas más frecuentes de Microsoft Graph

## ¿Qué plataformas son compatibles con la API de Microsoft Graph?
<!--
Apps can use the Microsoft Graph API to perform create, read, update, and delete (CRUD) operations on data sources and entities, giving them seamless access to work data. 

**Ease of use--one endpoint, all Office 365 data under one roof**

You can use the API in four steps:
1.  Select your programming language and development environment.
2.  Build your app.
3.  Optionally, host your app in Microsoft Azure or any cloud platform you choose.
4.  Authenticate your users by using single sign-on with Azure AD.

As a developer you can use the API to create custom apps that access and interact with all the richness of enterprise and productivity data--users, groups, organizational contacts, files, folders, mail, calendar, insights and relationships--and build apps across all mobile, web, and desktop platforms. No matter your development platform or tools. Using a single service endpoint to access those entities and data. And a single authentication flow.  -->

Puede:

<!--Just like in Office 365 APIs, Office 365 unified endpoint API  allows you to build apps using any development environment of your choice:  -->

- Usar cualquier entorno de desarrollo con el que esté familiarizado, como .NET, PHP, Java, Python o Ruby
- Usar cualquier lenguaje de programación, plataforma de desarrollo y el entorno de hospedaje
- Crear una aplicación que tenga acceso a la API mediante cualquier lenguaje web, como JavaScript, HTML5, Python, Ruby, PHP o ASP.NET  
- Usar el IDE que desee, ya sea Visual Studio, Eclipse, Android Studio, Xcode u otro de su elección
- Alojar sus aplicaciones en Microsoft Azure o en cualquier plataforma en la nube
- Desarrollar aplicaciones para Windows Universal, iOS, Android, o en otra plataforma de dispositivos
- Llamar a la API de Complementos de Office (anteriormente aplicaciones de Office) o Complementos de SharePoint (anteriormente aplicaciones de SharePoint)
 


## ¿Por qué debería usar la API de Microsoft Graph?

Supongamos que desea recuperar mediante programación los archivos de un usuario y la imagen del perfil, así como encontrar al administrador de ese usuario que modificó por última vez ese archivo en su organización. Dado que la información se almacena en varios servicios (Azure Active Directory, SharePoint y Exchange), la tarea implica varios pasos mediante las API de Office 365: 

1. Usar el Servicio de detección para buscar los diversos puntos de conexión de servicio 
2. Determinar la dirección URL de los servicios a la que sus aplicaciones de Office 365 desean conectarse
3. A continuación, adquirir y administrar el token de acceso para cada servicio y realizar la solicitud al servicio directamente

Ahora, puede usar el uso de la API de Microsoft Graph para realizar la misma operación compleja a través de un único punto de conexión de la API de REST. No tiene que descubrir y navegar a un punto de conexión diferente para cada servicio, adquirir y administrar un token de acceso separado para cada servicio, tratar con servicios en silos y modelos de datos diferentes.

##Consultas de ejemplo

El ejemplo siguiente muestra el modelo actual para interactuar con la API de Office 365 usando puntos de conexión de servicio dispares y lo sencillo que se convierte con la API de Microsoft Graph.

**Puntos de conexión de servicios dispares**

|   **Operación**                  |  **API**                          |  **Punto de conexión de servicio** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| Descubrir los puntos de conexión de servicio para la API de Office 365               |     `Discovery Service`           | _https://_**api.office.com**_/discovery/v1.0/me/services_ |
| Obtener usuarios           |     `Azure AD Graph API` | _https://_**graph.windows.net**_/contoso.com/users?api-version=2013-04-05_|
| Obtener colección de mensajes de la Bandeja de entrada       |     `Office 365 API`           | _https://_**outlook.office365.com**_/api/v1.0/me/messages_  |
| Obtener los archivos de Jorge   |     `Office 365 API`  | _https://_**contoso-my.sharepoint.com**_/personal/joe_contoso_com/_api/v1.0/files_ |


Con la API de Microsoft Graph, no tiene que descubrir primero los puntos de conexión de servicio y luego atravesar diferentes puntos de conexión para obtener los archivos del usuario, el correo, etc. Solo necesita interactuar con un único espacio de nombres de direcciones URL de REST, que es _**graph.microsoft.com**_.

**API de Microsoft Graph**

|   **Operación**                  |  **API**                          |  **Punto de conexión de servicio** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| Descubrir los puntos de conexión de servicio para la API de Office 365                |     `Microsoft Graph`           | No es necesario |
| Obtener usuarios           |     `Microsoft Graph` | _https://_**graph.microsoft.com**_/v1.0/contoso.onmicrosoft.com/users_ |
| Obtener colección de mensajes de la Bandeja de entrada       |     `Microsoft Graph`           | _https://_**graph.microsoft.com**_/v1.0/me/messages_  |
| Obtener los archivos de Jorge   |     `Microsoft Graph `  | _https://_**graph.microsoft.com**_/v1.0/me/drive/root/children_ |


## ¿Cuáles son las ventajas de usar la API de Microsoft Graph?

Algunas de las ventajas de usar la API de Microsoft Graph son las siguientes:

**Experiencia de desarrollador consistente y optimizada para consumir servicios en la nube de Microsoft**

-   Espacio de nombres único para todos los puntos de conexión de servicio. No es necesario para el descubrimiento del punto de conexión del servicio.
-   Un token para acceder a todos los recursos
-   Navegación integrada y directa entre los servicios actualmente en silos (por ejemplo, obtener el departamento y la cadena de administración del usuario que ha creado un documento concreto)
-   Solo necesita usar un conjunto de API único, es decir, solo necesita usar la API de Microsoft Graph para conectarse a múltiples servicios
-   La API REST unificada y expandida y las entidades en la plataforma de Office 
-   Nomenclatura de propiedades coherente y esquemas de las entidades (las propiedades de navegación se incluyen entre las entidades)

**Habilitar experiencias de trabajo y de vida más productivas usando Office**

-   Contenido contextual. Por ejemplo, buscar documentos por grupo, proyecto, equipo o tendencias
-   Relaciones de usuario contextual. Por ejemplo, puede buscar los usuarios por la pertenencia a grupos, intereses, habilidades y experiencia.  También puede obtener la relación del organigrama de la organización.

**Permiten desarrollar aplicaciones en cualquier lenguaje de programación en cualquier plataforma**

-   Herramientas de desarrollo y recursos para todos los desarrolladores. Puede desarrollar usando cualquier plataforma y lenguaje 
-   Desarrollo de aplicaciones móviles para todas las plataformas usando tecnologías abiertas  
-   No es necesario contar con ningún conocimiento especializado de Exchange, SharePoint o Azure AD para tener acceso a las entidades de la API de Microsoft Graph.

<!---<a name="msg_v2auth"> </a>-->

## ¿Es compatible la API de Microsoft Graph con el modelo de autenticación de la aplicación v2.0?

Sí. Para obtener más información, consulte [Autenticar puntos de conexión de Microsoft Graph usando el punto de conexión de autenticación v2](http://graph.microsoft.io/docs/authorization/converged_auth).



  > Su opinión es importante para nosotros. Conecte con nosotros en [Stack Overflow](http://stackoverflow.com/questions/tagged/office365). Etiquete sus preguntas con [MicrosoftGraph] y [office365].








