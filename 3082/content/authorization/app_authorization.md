
# Autorización de aplicaciones de Microsoft Graph


En este artículo se describe cómo autenticar a un usuario, obtener un token de acceso y renovar un token de acceso usando un token de actualización. El flujo de autenticación se puede dividir en dos pasos básicos:

1. Solicitar un código de autorización
2. Use el código de autorización para solicitar un token de acceso y actualizar el token. 

>  **Nota**: Puede usar el token de actualización para adquirir un token de acceso nuevo cuando expire el actual.

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
 
###Autenticar un usuario y autorizar la aplicación
Para autorizar la aplicación, primero debe autenticarse el usuario. Para ello, redirija el usuario y la información de la aplicación hacia el punto de conexión de autorización de Azure Active Directory (Azure AD) para que inicie sesión en su cuenta de Office 365. Una vez que el usuario inicia sesión y da su consentimiento a los permisos que solicita la aplicación (si todavía no lo ha hecho), la aplicación recibirá el código de autorización necesario para adquirir un token de acceso de OAuth.

> 
  **Nota**:  Para realizar esta acción, llame a métodos en la [biblioteca de autenticación de Azure AD (ADAL)](https://msdn.microsoft.com/en-us/library/azure/jj573266.aspx). Para obtener más información acerca del flujo de autorización, consulte [Flujo de concesión de códigos de autorización](https://msdn.microsoft.com/en-us/library/azure/dn645542.aspx).

La autorización de una aplicación se inicia con el envío de una solicitud HTTPS GET usando la siguiente dirección URL:
 
```GET https://login.microsoftonline.com/common/oauth2/authorize?response_type=code&redirect_uri=<uri>&client_id=<id>```

**Parámetros necesarios de la cadena de consulta**

| Nombre del parámetro  | Valor  | Descripción                                                                                            |
|:----------------|:-------|:-------------------------------------------------------------------------------------------------------|
| *client_id*     | cadena | ID de cliente creado para su aplicación. Este es el valor de **ID. DE CLIENTE** de su aplicación establecido en el registro de la aplicación del inquilino de Azure.                                                                  |
| *response_type* | cadena | Especifica el tipo de respuesta solicitada. En una solicitud de concesión de código de autorización, el valor debe ser el código. |
| *redirect_uri*  | cadena | Dirección URL de redireccionamiento a la que se envía el explorador una vez finalizada la autenticación.  Este valor debe coincidir con el valor **URL DE RESPUESTA** previamente configurado.                        |
 


A continuación se muestra un ejemplo de este tipo de solicitud tal y como se implementa en una aplicación en ejecución:


```GET https://login.microsoftonline.com/common/oauth2/authorize?response_type=code&redirect_uri=http%3a%2f%2flocalhost:1339/auth/azureoauth/callback&client_id=8b8539cd-7b75-427f-bef1-4a6264fd4940``` 

Esta solicitud devuelve una respuesta de tipo `200 OK` y presenta la página de inicio de sesión de la cuenta de Azure AD. 

Después de que el usuario inicie sesión con unas credenciales válidas y dé su consentimiento a los permisos que se concederán a la aplicación, la página de inicio de sesión envía una solicitud de tipo `POST` a Azure. Si la solicitud anterior se acepta, Azure responde con un mensaje de tipo `302 Found` para reenviar la llamada al URI de redireccionamiento de la aplicación a fin de que la aplicación reciba el token de acceso. El URI reenviado, que se especifica en el encabezado `Location` de la respuesta, corresponde a la dirección URL de respuesta de la aplicación y tiene dos parámetros de consulta anexados (`code=...` y `session_state=...`). En el ejemplo siguiente se muestra un extracto de una respuesta de este tipo: 

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

En este ejemplo, la dirección URL de respuesta de la aplicación es `http://localhost:1339/auth/azureoauth/callback`. En el procesamiento de esta respuesta, debe extraer el valor del parámetro `code` y usarlo para adquirir los tokens iniciales de acceso y actualización de OAuth 2.0 (consulte la sección siguiente).

> La respuesta `302 Found` anterior es diferente a la respuesta `302 Found` que se obtendría si el proceso de inicio de sesión se iniciase con la dirección URL `https://login.windows.net/<tenantId>/oauth2/authorize?...`. En este último caso, la respuesta `302 Found` reenvía la solicitud a `login.microsoftonline.com`.
 
<!---<a name="msg_get_app_authenticated"> </a> -->

###Adquirir un token de acceso
Para tener acceso a los recursos de la API de Microsoft Graph, la aplicación debe incluir un token de acceso de OAuth 2.0 válido en todas las solicitudes HTTP. El token de acceso se puede obtener con la siguiente solicitud POST:

```no-highlight 
POST https://login.microsoftonline.com/common/oauth2/token HTTP/1.1
content-type : application/x-www-form-urlencoded
content-length : 144
```
 
Esta solicitud requiere una carga de codificación URL con el formato siguiente:
 
```no-highlight 
grant_type=authorization_code
&redirect_uri=<uri>
&client_id=<id>
&client_secret=<secret_key>
&code=<code>
&resource=https%3A%2F%2Fgraph.microsoft.com%2F
```

**Parámetros necesarios de la cadena de consulta**

| Nombre del parámetro  | Valor  | Descripción                                                                                            |
|:----------------|:-------|:-------------------------------------------------------------------------------------------------------|
| *client_id*     | cadena | ID de cliente creado para su aplicación.  |
| *client_secret*  | cadena | Clave creada para su aplicación. Este valor es el mismo que el de la sección **Claves** de la página de configuración de la aplicación del Portal de administración de Azure.|
| *redirect_uri*  | cadena | Dirección URL de redireccionamiento a la que se envía el explorador una vez finalizada la autenticación.  |
| *código*  | cadena | El código de autorización. Valor del parámetro de consulta `code` que devuelva la respuesta a la solicitud de autorización. |
| *recurso*   | cadena | El recurso al que desea tener acceso. Para llamar a la API de Microsoft Graph, establezca el valor de este parámetro en "https://graph.microsoft.com/".|

El fragmento de código siguiente muestra un ejemplo de la carga de solicitud usada para adquirir el token de acceso inicial de OAuth 2.0:

```no-highlight  
grant_type=authorization_code&
redirect_uri=http%3A%2F%2Flocalhost%3A1339%2Fauth%2Fazureoauth%2Fcallback&
client_id=8b8539cd-7b75-427f-bef1-4a6264fd4940&client_secret=PJW3dznGfyNSm3rM9aHeXWGKsTMepKXT1Eqy45XXdU4%3D&
code=AAABAAAAvPM1KaPlrEqdFSBzjqfTGBLRVQc6BtQmJ_9DQZUg8Tb7TJgTmbTE7AHM93qB5EKc4Bf-bOgzt3mebAywK-09X1uBHwOluuqSWfd9LU2HHgZtxcZFIYI5UL7L1UEvhrJRvX2iHhfz9ZSRMZMVL55n_K79gCOxtSATeCUw52zPk5ZaQ87Y42SCLsRZN4Y_zddhD3mMpkObiHVT8HzfhBUiT0oX0e-Q439vkbZoKiq1HaqMR3IPHiCXDbPPH5u7a4NTe5xAhh-o2MUIe6s4Xqql86sv1-IwyroOJJMueGUarkfbgwqmYL9Tm-jWab8o-BAK_plVsN73GU8cXO8ts30wa2YmNR5ZxSkw8oiB4mSZwGzGQlch55DRnucDs0SZBgj5etGi3SeXv5jhKlDU2S0bAPnGxF3QAH0N_zBpfakETVlcsHKi714u-tn9da6aTPQsE0IYKTAYgxjTMei6zfRFvCZi-tKdFR6X9TvvmF2iPdGQGWKeLw8CMWUzU8VmOhiHc0aBKG6RaXAOTM067J_9WKYPxMopcytD2z8HVkL1QhggAA&
resource=https%3A%2F%2Fgraph.microsoft.com%2F
```

Si la solicitud se acepta, se devuelve una respuesta `200 OK`. A continuación se muestra un ejemplo:

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

 
El cuerpo de la respuesta es una cadena con formato JSON que contiene el token de acceso (`access_token`). Este token debe suministrarse en las solicitudes HTTP subsiguientes para proporcionar acceso a los recursos de la API de Microsoft Graph. 

El valor de la propiedad `scope` debe coincidir con los permisos concedidos para la aplicación durante su registro.

El token de acceso mantiene su validez durante el intervalo de tiempo especificado (`3599` segundos en el ejemplo anterior o 1 hora) que se cuenta desde el momento de su emisión, tal y como se especifica en la propiedad `expires_in`. El resultado también contiene un token de actualización (`refresh_token`) que debe usarse para renovar un token de acceso expirado o que va a expirar. 

En cualquier código de producción, la aplicación debe vigilar la expiración de estos tokens y renovar el token de acceso antes de que expire el token de actualización. 


<!---<a name="msg_renew_access_token using refresh token"> </a> -->

###Usar un token de actualización para renovar un token de acceso que va a expirar
Para actualizar un token de acceso expirado, use una solicitud POST similar a la del ejemplo siguiente (siempre que no haya expirado el token de actualización):

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

**Parámetros necesarios de la cadena de consulta**

| Nombre del parámetro  | Valor  | Descripción                                                                                                                                         |
|:----------------|:-------|:----------------------------------------------------------------------------------------------------------------------------------------------------|
| *client_id*     | cadena | ID de cliente que se crea para su aplicación.  |
| *redirect_uri*  | cadena | Dirección URL de redireccionamiento a la que se envía el explorador una vez finalizada la autenticación. Debe coincidir con el valor del parámetro *redirect_uri* que se usó en la primera solicitud. |
| *client_secret* | cadena | Uno de los valores de la sección Claves creados para la aplicación.                                                                                                     |
| *refresh_token* | cadena | El token de actualización que recibió antes.    |
| *recurso*      | cadena | El recurso al que desea tener acceso.|

Tenga en cuenta que esta solicitud es casi idéntica a la solicitud de adquisición inicial del token. Hay dos diferencias en la carga de la solicitud. Más concretamente, el parámetro `grant_type` ahora tiene el valor del parámetro `refresh_token` en lugar del valor de `code`.
 
Si la respuesta es correcta, se devuelve la carga de una cadena JSON similar al resultado siguiente:

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
 
Aparte de la propiedad `id_token` que falta, el cuerpo de la respuesta tiene una sintaxis y una semántica idénticas a las de la respuesta inicial de la adquisición del token. La duración de los nuevos valores `access_token` y `refresh_token` se ha ampliado. El nuevo tiempo de expiración es el número de segundos especificado en los valores del parámetro `expires_in` y se cuenta desde el momento en que se envió la solicitud del token de actualización. 
 
Si el token de actualización expira, no podrá renovar los tokens de acceso expirados mediante la solicitud POST descrita anteriormente. En su lugar, deberá reiniciar el proceso de [autenticación y autorización de la aplicación](#msg_get_app_authorized).


<!--##Additional Resources##

- [Hands on lab: Deep dive into the Office 365 unified API](http://dev.office.com/hands-on-labs/4585)  -->

