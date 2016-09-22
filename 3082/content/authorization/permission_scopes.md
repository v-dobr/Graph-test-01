# Ámbitos de permiso de Microsoft Graph

Microsoft Graph expone los ámbitos de permiso de OAuth 2.0 que se usan para controlar el acceso que tiene una aplicación a los datos. Como desarrollador, configura su aplicación con los ámbitos de permiso adecuados para el acceso que requiere. Normalmente hace esto a través del portal de Azure. Durante el inicio de sesión, a los usuarios o los administradores se les da una oportunidad para permitir que su aplicación acceda a sus datos son los ámbitos de permiso que configuró. Por este motivo, debe elegir los ámbitos de permiso que proporcionen el menos nivel de privilegios necesarios para su aplicación. Para más detalles sobre cómo configurar los permisos para la aplicación y en el proceso de consentimiento, vea [Integración de aplicaciones con Azure Active Directory](https://azure.microsoft.com/en-us/documentation/articles/active-directory-integrating-applications/).


##Conceptos del ámbito de permiso

###Solo para la aplicación vs. ámbitos delegados
Los ámbitos de permiso pueden ser solo para la aplicación o delegados. Los ámbitos solo para la aplicación (también conocidos como roles de la aplicación) conceden a la aplicación el conjunto completo de privilegios ofrecidos por el ámbito. Los ámbitos solo para la aplicación se usan típicamente por aplicaciones que se ejecutan como un servicio sin que esté presente un usuario que haya iniciado sesión. Los ámbitos de permiso delegados son para las aplicaciones en las que un usuario inicia sesión. Estos ámbitos delegan los privilegios del usuario que ha iniciado sesión a la aplicación, permitiendo que la aplicación actúe como si fuera el usuario. Los privilegios reales concedidos a la aplicación será la combinación menos privilegiada (la intersección) de los privilegios concedidos por el ámbito y los que posea el usuario que ha iniciado sesión. Por ejemplo, si el ámbito de permiso concede privilegios delegados para escribir todos los objetos del directorio, pero el usuario que ha iniciado sesión tiene privilegios solo para actualizar su propio perfil de usuario, la aplicación solo podrá escribir el perfil del usuario que ha iniciado sesión, pero no otros objetos.

###Perfiles completos y básicos para usuarios y grupos
El perfil completo (o perfil) de un usuario o de un grupo incluye todas las propiedades declaradas por la entidad. Ya que el perfil puede contener información del directorio confidencial o información personal identificable (PII), varios ámbitos limitan el acceso de la aplicación a un conjunto de propiedades limitadas conocidas como perfil básico. Para los usuarios, el perfil básico incluye solo las siguientes propiedades: nombre para mostrar, nombre y apellidos, foto y dirección de correo electrónico. Para los grupos, el perfil básico contiene solo el nombre para mostrar. 

<!---   <a name="msg_perm_details"> </a>  -->

##Detalles del ámbito de permiso
Debe configurar la aplicación para tener los permisos necesarios para tener acceso a los recursos de la API de Microsoft Graph. Los permisos se aplican a recursos individuales para los derechos de leer, escribir o ambos. 

Las tablas siguientes enumeran los ámbitos de permiso de la API de Microsoft Graph y explican el acceso concedido por cada uno. 
- La columna **Ámbito** enumera el nombre del ámbito. Los nombres del ámbito adoptan la forma resource.operation.constraint; por ejemplo, Group.ReadWrite.All. Si la restricción es "All", el ámbito concede a la aplicación la capacidad para realizar la operación (ReadWrite) en todos los recursos especificados (Group) en el directorio; de lo contrario, el ámbito solo permite la operación en el perfil del usuario que ha iniciado sesión. Los ámbitos pueden conceder privilegios limitados para la operación especificada. Vea la columna **Descripción** para más detalles.
- En la columna **Permiso** se ofrece información sobre cómo se muestra el ámbito en el portal de Azure. 
- En la columna **Descripción** se describe el conjunto completo de privilegios que concede el ámbito. Para ámbitos delegados, el acceso real concedido a la aplicación será la combinación menos privilegiada (la intersección) del acceso concedido por el ámbito y los privilegios del usuario que ha iniciado sesión. 
- Los ámbitos se agrupan en función de si los permisos requieren el consentimiento del administrador.

  > **Nota**: Vea las [Notas de la versión](http://graph.microsoft.io/docs/overview/release_notes) para las limitaciones de ámbito de permiso de `v1.0` y `beta`.
  
###Permisos que requieren el consentimiento del administrador

|   **Ámbito**                  |  **Permiso en el Portal de administración de Azure**                          |  **Descripción** |
|:-----------------------------|:-----------------------------------------|:-----------------|
| _User.Read.All_                |     `Read all user's full profiles`           | Igual que User.ReadBasic.All, excepto que permite que la aplicación lea el perfil completo de todos los usuarios de la organización y cuando lee las propiedades de navegación como el administrador y los subordinados. El perfil completo incluye todas las propiedades declaradas de la entidad **Usuario**. Para leer los grupos de los que es miembro un usuario, la aplicación requerirá también Group.Read.All o Group.ReadWrite.All. |
| _User.ReadWrite.All_           |     `Read and write all user's full profiles` | Permite que la aplicación lea y escriba el conjunto completo de las propiedades del perfil, los informes y los administradores de otros usuarios de su organización, en nombre del usuario que ha iniciado sesión. |
| _Directory.Read.All_           |     `Read directory data`                     | Permite que la aplicación lea los datos del directorio de su organización, como los usuarios, los grupos y las aplicaciones. |
| _Directory.ReadWrite.All_      |     `Read and write directory data`           | Permite que la aplicación lea y escriba los datos del directorio de su organización, como los usuarios y los grupos.  No se permite la eliminación de un usuario o un grupo. No permite que la aplicación elimine usuarios o grupos ni que restablezca contraseñas de usuario. |
| _Directory.AccessAsUser.All_   |     `Access directory as the signed-in user`  | Permite que la aplicación tenga el mismo acceso a la información del directorio que el usuario que ha iniciado sesión.|
| _Group.Read.All_ |    `Read all groups` | Permite que la aplicación enumere grupos y lea sus propiedades y todas las pertenencias a los grupos en nombre del usuario que ha iniciado sesión.  También permite que la aplicación lea el calendario, las conversaciones, los archivos y otro contenido del grupo de todos los grupos en los que el usuario que ha iniciado sesión tiene acceso. |
| _Group.ReadWrite.All_ |    `Read and write all groups`| Permite que la aplicación cree grupos y lea todas las propiedades de los grupos y las pertenencias en nombre del usuario que ha iniciado sesión.  Además, permite a los propietarios del grupo que administren sus grupos y permite a los miembros del grupo que actualicen su contenido del grupo. |


###Permisos que no requieren el consentimiento del administrador

|   **Ámbito**    |  **Permiso en el Portal de administración de Azure**   |  **Descripción** |
|:---------------|:------------------|:-----------------|
| _User.Read_       |    `Sign-in and read user profile` | Permite a los usuarios iniciar sesión en la aplicación y permite a la aplicación leer el perfil de los usuarios que han iniciado sesión. El perfil completo incluye todas las propiedades declaradas de la entidad usuario. La aplicación no puede leer las propiedades de navegación, como el administrador o los subordinados. También permite que la aplicación lea la siguiente información básica de la empresa del usuario que ha iniciado sesión (a través del objeto **TenantDetail**): id. de inquilino, nombre para mostrar del inquilino y dominios comprobados.|
| _User.ReadWrite_ |    `Read and write access to user profile` | Permite que la aplicación lea su perfil. También permite que la aplicación actualice su información de perfil en su nombre. |
| _User.ReadBasic.All_ |    `Read all user's basic profiles` | Permite que la aplicación lea el perfil básico de todos los usuarios de la organización en nombre del usuario que ha iniciado sesión. Las siguientes propiedades comprenden el perfil básico del usuario: nombre para mostrar, nombre y apellidos, foto y dirección de correo electrónico. Para leer los grupos de los que es miembro un usuario, la aplicación requerirá también Group.Read.All o Group.ReadWrite.All.| 
| _Mail.Read_ |    `Read user mail` | Permite que la aplicación lea el correo electrónico en los buzones del usuario.  |
| _Mail.ReadWrite_ |    `Read and write access to user mail` | Permite que la aplicación cree, lea, actualice y elimine correos electrónicos en los buzones del usuario. No incluye el permiso para enviar correos.|
| _Mail.Send_ |    `Send mail as a user` | Permite que la aplicación envíe correos como si fueran usuarios de la organización.  |
| _Calendars.Read_ |    `Read user calendars`  | Permite que la aplicación lea los eventos en los calendarios del usuario.|
| _Calendars.ReadWrite_ |    `Have full access to user calendars`  | Permite que la aplicación cree, lea, actualice y elimine eventos de los calendarios del usuario. |
| _Contacts.Read_ |    `Read user contacts`  | Permite que la aplicación lea los contactos del usuario. |
| _Contacts.ReadWrite_ |    `Have full access to user contacts`  | Permite que la aplicación cree, lea, actualice y elimine los contactos del usuario. |
| _Files.Read_ |    `Read user files and files shared with user` | Permite que la aplicación lea los archivos del usuario que ha iniciado sesión y los archivos compartidos con el usuario.| 
| _Files.ReadWrite_ |   `Have full access to user files and files shared with user` | Permite que la aplicación lea, cree, actualice y elimine los archivos del usuario que ha iniciado sesión y los archivos compartidos con el usuario. |
| _Files.ReadWrite.Selected_ |    `Read and write files that the user selects` | Permite que la aplicación lea y escriba archivos que el usuario selecciona. La aplicación tiene acceso durante varias horas después de que el usuario seleccione un archivo. |
| _Files.Read.Selected_ |    `Read files that the user selects`  | Permite que la aplicación lea los archivos que el usuario selecciona. La aplicación tiene acceso durante varias horas después de que el usuario seleccione un archivo. |
| _Sites.Read.All_ |    `Read items in all site collections` | Permite que la aplicación lea los documentos y enumere los elementos en todas las colecciones del sitio en nombre del usuario que ha iniciado sesión. |
| _OpenID_ |    `Sign users in` (vista previa) | Permite que los usuarios inicien sesión en la aplicación con sus cuentas profesionales o educativas y permite que la aplicación vea la información básica del perfil del usuario.|
| _offline_access_ |    `Access user's data anytime` (vista previa) | Permite que la aplicación lea y actualice los datos del usuario, incluso cuando no están usando en ese momento la aplicación.|

###Los permisos solo para la aplicación requieren el consentimiento del administrador

|   **Ámbito**    |  **Permiso en el Portal de administración de Azure**   |  **Descripción** |
|:---------------|:------------------|:-----------------|
| _Mail.Read_       |    `Read mail in all mailboxes` | Permite que la aplicación lea el correo en todos los buzones sin un usuario que ha iniciado sesión.|
| _Mail.ReadWrite_ |    `Read and write mail in all mailboxes` | Permite que la aplicación cree, lea, actualice y elimine correos en todos los buzones sin un usuario que ha iniciado sesión. No incluye el permiso para enviar correos. |
| _Mail.Send_ |    `Send mail as any user` | Permite que la aplicación envíe correos como cualquier usuario sin un usuario con la sesión iniciada. | 
| _Calendars.Read_ |    `Read calendars in all mailboxes` | Permite que la aplicación lea los eventos de todos los calendarios sin un usuario que ha iniciado sesión. |
| _Calendars.ReadWrite_ |    `Read and write calendars in all mailboxes` | Permite que la aplicación cree, lea, actualice y elimine eventos de todos los calendarios sin un usuario que ha iniciado sesión.|
| _Contacts.Read_ |    `Read contacts in all mailboxes` | Permite que la aplicación lea todos los contactos en todos los buzones sin un usuario que ha iniciado sesión. |
| _Contacts.ReadWrite_ |    `Read and write contacts in all mailboxes`  |Permite que la aplicación cree, lea, actualice y elimine todos los contactos en todos los buzones sin un usuario que ha iniciado sesión.|
| _User.ReadBasic.All_ |    `Read all users' basic profiles`  | Permite que la aplicación lea un conjunto de propiedades básicas del perfil de otros usuarios de su organización sin un usuario que ha iniciado sesión. Incluye el nombre para mostrar, nombre y apellidos, foto y mensaje de fuera de la oficina.|
| _User.Read.All_ |    `Read all users' full profiles` | Permite que la aplicación lea el conjunto completo de las propiedades del perfil, las pertenencias del grupo, los informes y los administradores de otros usuarios de su organización, sin un usuario que ha iniciado sesión.| 
| _User.ReadWrite.All_ |   `Read and write all users' full profiles` | Permite que la aplicación lea y escriba el conjunto completo de las propiedades del perfil, las pertenencias del grupo, los informes y los administradores de otros usuarios de su organización, sin un usuario que ha iniciado sesión.|


##Vista previa
###Permisos que no requieren el consentimiento del administrador (vista previa)

|   **Ámbito**    |  **Permiso en el Portal de administración de Azure**   |  **Descripción** |
|:---------------|:------------------|:-----------------|
| _Tasks.ReadWrite_ |    `Create, read, update and delete user tasks and plans` (vista previa) | Permite que la aplicación cree, lea, actualice y elimine tareas y planes (y las tareas en ellos), que se han asignado o que se han compartido con el usuario que ha iniciado sesión.|
| _People.Read_ |    `Read users' relevant people lists` (vista previa) | Permite que la aplicación lea un listado ordenado de los contactos relevantes del usuario que ha iniciado sesión. La lista incluye los contactos locales, los contactos de las redes sociales, el directorio de su organización y los contactos de comunicaciones recientes (como el correo electrónico y Skype).|
| _People.ReadWrite_ |    `Read and write users' relevant people lists` (vista previa) | Permite que la aplicación cree, lea y escriba en el listado ordenado de los contactos relevantes del usuario que ha iniciado sesión. La lista incluye los contactos locales, los contactos de las redes sociales, el directorio de su organización y los contactos de comunicaciones recientes (como el correo electrónico y Skype).|
| _Notes.Create_ |    `Create pages in users' notebooks` (vista previa) | Permite que la aplicación lea los títulos de los blocs de notas y las secciones y cree páginas nuevas, blocs de notas y secciones en nombre del usuario que ha iniciado sesión.|
| _Notes.ReadWrite.CreatedByApp_ |    `Limited notebook access` (vista previa) | Permite que la aplicación lea los títulos de los blocs de notas y las secciones y cree páginas nuevas en nombre del usuario que ha iniciado sesión. También permite que la aplicación lea y actualice las páginas creadas por la aplicación. |
| _Notes.Read_ |    `Read user notebooks` (vista previa) | Permite que la aplicación ver los títulos de los blocs de notas de OneNote y las secciones y lea todas las páginas en nombre del usuario que ha iniciado sesión. No puede ver las secciones protegidas con contraseña. |
| _Notes.ReadWrite_ |    `Read and write user notebooks` (vista previa) | Permite que la aplicación lea los títulos de los blocs de notas y las secciones, lea todas las páginas, escriba todas las páginas y cree nuevas páginas en nombre del usuario que ha iniciado sesión.  No puede acceder a las secciones protegidas con contraseña. |
| _Notes.Read.All_ |    `Read all notebooks that the user can access` (vista previa) | Permite que la aplicación lea los contenidos de todos los blocs de notas y las secciones a las que el usuario que ha iniciado sesión tiene acceso.   No puede leer las secciones protegidas con contraseña. |
| _Notes.ReadWrite.All_ |    `Read and write notebooks that the user can access` (vista previa) | Permite que la aplicación lea y escriba los contenidos de todos los blocs de notas y las secciones a las que el usuario que ha iniciado sesión tiene acceso.  No puede acceder a las secciones protegidas con contraseña.|


<!-- -->

##Escenarios de ámbito de permiso
Los siguientes son algunos escenarios de la aplicación que usan los recursos `User` y `Group` y sus correspondientes ámbitos requeridos. La siguiente tabla muestra los ámbitos de permiso necesarios para que una aplicación pueda realizar operaciones específicas. Tenga en cuenta que en algunos casos, capacidad de la aplicación para realizar algunas operaciones dependerá de si el ámbito del permiso es solo para la aplicación o delegado, y, en el caso de ámbitos de permiso delegados, en los privilegios del usuario que ha iniciado sesión. 

###Obtener acceso a los escenarios usando el recurso usuario y los ámbitos requeridos

| **Tareas de la aplicación que impliquen al usuario**   |  **Ámbitos necesarios** | **Permisos en el Portal de administración de Azure** |
|:-------------------------------|:---------------------|:---------------|
| La aplicación desea leer otra información básica de los usuarios (solo el nombre para mostrar y la imagen), por ejemplo para mostrar en una experiencia de selección de personas   | _User.ReadBasic.All_  |  `Read all user's basic profiles` |
| La aplicación desea leer el perfil de usuario completo para usuarios que han iniciado sesión (consulte subordinados y administrador, etc)  | _User.Read_ | `Enable sign-in and read user profile`|
| La aplicación desea leer el perfil de usuario completo de todos los usuarios  | _User.Read.All_ |  `Read all user's full profiles`   |
| La aplicación desea leer archivos, información del calendario y el correo del usuario que ha iniciado sesión  | _User.Read_, _Files.Read_, _Mail.Read_, _Calendar.Read_ | `Enable sign-in and read user profile`, `Read users' files`,  `Read user mail`,  `Read user calendars` |
| La aplicación desea leer los (mis) archivos del usuario que ha iniciado sesión y los archivos que otros usuarios han compartido con el usuario que ha iniciado sesión (yo). | _User.Read_, _Files.Read_, _Sites.Read.All_ | `Enable sign-in and read user profile`, `Read users' files`,  `Read items in all site collections` |
| La aplicación quiere leer y escribir el perfil completo de usuario para el usuario que ha iniciado sesión   | _User.ReadWrite_ | `Read and write access to user profile` |
| La aplicación quiere leer y escribir el perfil de usuario completo de todos los usuarios    | _User.ReadWrite.All_ | `Read and write all user's full profiles` |
| La aplicación quiere leer y escribir los archivos, el correo y la información del calendario del usuario que ha iniciado sesión    | _User.ReadWrite_, _Files.ReadWrite_, _Mail.ReadWrite_, _Calendar.ReadWrite_  |  `Read and write access to user profile`,  `Read and write access to user profile`,  `Read and write access to user mail`, `Have full access to user calendars` |
   

###Escenarios de acceso mediante el recurso Group y los ámbitos requeridos
    
| **Tareas de la aplicación relacionadas con Group**  |  **Ámbitos necesarios** |  **Permisos en el Portal de administración de Azure** |
|:-------------------------------|:---------------------|:---------------|
| La aplicación desea leer la información básica del grupo (solo el nombre para mostrar y la imagen), por ejemplo para mostrar en una experiencia de selección de grupos  | _Group.Read.All_  | `Read all groups`|
| La aplicación quiere leer todo el contenido de todos los grupos unificados, incluidos los archivos y las conversaciones.  También necesita mostrar las pertenencias a grupos, ser capaz de actualizar las pertenencias a grupos (si es el propietario).  |  _Group.Read.All_ | `Read items in all site collections`, `Read all groups`|
| La aplicación desea leer y escribir todo el contenido de todos los grupos unificados, incluidos los archivos y las conversaciones.  También necesita mostrar las pertenencias a grupos, ser capaz de actualizar las pertenencias a grupos (si es el propietario).  |  _Group.ReadWrite.All_, _Sites.ReadWrite.All_ |  `Read and write all groups`, `Edit or delete items in all site collections` |
| La aplicación desea descubrir (buscar) un grupo unificado. Permite al usuario buscar un determinado grupo y elegir uno desde la lista enumerada para permitir que el usuario se una al grupo.     | _Group.ReadWrite.All_ | `Read and write all groups`|
| La aplicación desea crear un grupo a través del gráfico de AAD |   _Group.ReadWrite.All_ | `Read and write all groups`|
 

<!---   ##Additional Resources##


- [Authenticate Microsoft Graph endpoints using the v2.0 app model preview](authenticate-MSGraph-using-v2.md )
- [Hands on lab: Deep dive into the Office 365 unified API](http://dev.office.com/hands-on-labs/4585) -->

