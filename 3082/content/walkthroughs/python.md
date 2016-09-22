# Llamar a Microsoft Graph en una aplicación de Python 

En este artículo, veremos las tareas mínimas necesarias para conectar la aplicación a Office 365 y llamar a la API de Microsoft Graph. Para explicar los conceptos principales que se deben implementar en la aplicación, usaremos el código del ejemplo [Connect de Python para Office 365 con Microsoft Graph](https://github.com/microsoftgraph/python3-connect-rest-sample).

![Captura de pantalla de ejemplo Connect de Python de Office 365](./images/web-screenshot.png)

##  Requisitos previos

En este artículo se da por hecho lo siguiente:

* Se encuentra cómodo leyendo código de Python.
* Está familiarizado con los conceptos de OAuth.

## Información general

Para llamar a la API de Microsoft Graph, su aplicación de Python debe completar las siguientes tareas.

1. Registrar la aplicación en Azure Active Directory
2. Redirigir el explorador a la página de inicio de sesión
3. Recibir un código de autorización en la página de la dirección URL de respuesta
4. Solicitar un token de acceso desde el token que emite un punto de conexión
5. Usar el token de acceso en una solicitud a la API de Microsoft Graph 

<!--<a name="register"></a>-->
## Registrar la aplicación en Azure Active Directory

Antes de empezar a trabajar con Office 365, registre la aplicación y establezca los permisos para usar los servicios de Microsoft Graph. Con la [herramienta de registro de aplicaciones](https://dev.office.com/app-registration) puede registrar en pocos pasos su aplicación para que tenga acceso a la cuenta profesional o educativa de un usuario. Para administrarla, vaya al [Portal de administración de Microsoft Azure](https://manage.windowsazure.com).

Si prefiere registrar manualmente la aplicación, consulte la sección [Registrar una aplicación de servidor web con el Portal de administración de Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) y tenga en cuenta lo siguiente:

* Asegúrese de especificar http://127.0.0.1:8000/connect/get_token/ como **dirección URL de inicio de sesión**.
* Después de registrar la aplicación, [configure los **permisos delegados**](https://github.com/microsoftgraph/python3-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) requeridos por la aplicación de Python. En el ejemplo Connect, se requiere el permiso **Enviar correo como usuario con sesión iniciada**.

Anote los siguientes valores de la página **Configurar** de la aplicación de Azure porque los necesitará para configurar el flujo de OAuth en su aplicación de Python.

* Id. de cliente (exclusivo de la aplicación)
* Dirección URL de respuesta (http://127.0.0.1:8000/connect/get_token/)
* Una clave de aplicación (exclusiva de la aplicación)

<!--<a name="redirect"></a>-->
## Redirigir al explorador a la página de inicio de sesión

Su aplicación necesita redirigir al explorador a la página de inicio de sesión para iniciar el flujo de OAuth y obtener un código de autorización. 

En el ejemplo Connect, el siguiente código (ubicado en [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py)) genera la dirección URL que la aplicación necesita para redirigir al usuario y se canaliza a la vista en la que puede usarse para el redireccionamiento. 

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
## Recibir un código de autorización en la página de la dirección URL de respuesta

Después de que el usuario inicie sesión, el explorador se redirige a su dirección URL de respuesta, la función ```get_token``` en [*connect/views.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/views.py), con un código de autorización anexado a la cadena de consulta como la variable ```code```. 

En el ejemplo Connect, se obtiene el código de la cadena de consulta para que, posteriormente, se pueda intercambiar por un token de acceso.

```python
auth_code = request.GET['code']
```

<!--<a name="accessToken"></a>-->
## Solicitar un token de acceso desde el token que emite un punto de conexión

Cuando tenga el código de autorización, podrá usarlo junto con los valores de los parámetros ID de cliente, Clave y Dirección URL de respuesta que obtuvo en Azure Active Directory para solicitar un token de acceso. 

> **Nota**: La solicitud debe especificar también un recurso que esté intentando usar. En el caso de Microsoft Graph, el valor de recurso es `https://graph.microsoft.com`.

En el ejemplo Connect, se solicita un token en la función ```get_token_from_code``` en el archivo [*connect/auth_helper.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/auth_helper.py).

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

> **Nota**: La respuesta proporciona más información aparte del token de acceso. Por ejemplo, su aplicación puede obtener un token de actualización para solicitar nuevos tokens de acceso sin que el usuario tenga que iniciar sesión de nuevo explícitamente.

<!--<a name="request"></a>-->
## Usar el token de acceso en una solicitud a la API de Microsoft Graph

El token de acceso permite que su aplicación cree solicitudes autenticadas en la API de Microsoft Graph. Su aplicación debe anexar el token de acceso al encabezado de **autorización** de cada solicitud.

En el ejemplo Connect, se envía un correo electrónico usando el punto de conexión ```me/microsoft.graph.sendMail``` en la API de Microsoft Graph. El código está en la función ```call_sendMail_endpoint``` del archivo [*connect/graph_service.py*](https://github.com/microsoftgraph/python3-connect-rest-sample/blob/master/connect/graph_service.py). Este es el código que muestra cómo anexar el código de acceso al encabezado de autorización.

```python
# Set request headers.
headers = { 
  'User-Agent' : 'python_tutorial/1.0',
  'Authorization' : 'Bearer {0}'.format(access_token),
  'Accept' : 'application/json',
  'Content-Type' : 'application/json'
}
```

> **Nota**: La solicitud también debe enviar un encabezado **Content-Type** con un valor que acepte la API de Graph. Por ejemplo, `application/json`.

La API de Microsoft Graph es una interfaz unificadora muy potente que permite trabajar con todos los tipos de datos de Microsoft. Consulte la referencia de la API para ver todas las posibilidades que ofrece la API de Microsoft Graph.

<!--
## Additional resources

-  [Office 365 Python Connect sample using Microsoft Graph](https://github.com/OfficeDev/O365-Python-Microsoft-Graph-Connect)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference]()-->
