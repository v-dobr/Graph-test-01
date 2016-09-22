# Llamar a Microsoft Graph en una aplicación de Ruby 

En este artículo veremos las tareas mínimas necesarias para obtener un token de acceso de Azure Active Directory (AD) y llamar a la API de Microsoft Graph. Para explicar los conceptos principales que se deben implementar en la aplicación, usaremos el código del [ejemplo Connect de Ruby para Office 365 con Microsoft Graph](https://github.com/microsoftgraph/ruby-connect-rest-sample).

![Captura de pantalla del ejemplo Connect de Ruby para Office 365](./images/web-screenshot.png)

## Información general

Para llamar a la API de Microsoft Graph, su aplicación Ruby debe completar las siguientes tareas.

1. Registrar la aplicación en Azure Active Directory
2. Redirigir el explorador a la página de inicio de sesión
3. Recibir un código de autorización en la página de la dirección URL de respuesta
4. Solicitar un token de acceso desde el punto de conexión del token
5. Usar el token de acceso en una solicitud a la API de Microsoft Graph 

<!--<a name="register"/>-->
## Registrar la aplicación en Azure Active Directory

Antes de empezar a trabajar con Office 365, registre la aplicación y establezca los permisos para usar los servicios de Microsoft Graph. Con la [herramienta de registro de aplicaciones](https://dev.office.com/app-registration) puede registrar en pocos pasos su aplicación para que tenga acceso a la cuenta profesional o educativa de un usuario. Para administrarla, vaya al [Portal de administración de Microsoft Azure](https://manage.windowsazure.com).

Si prefiere registrar manualmente la aplicación, consulte la sección [Registrar una aplicación de servidor web con el Portal de administración de Azure](https://msdn.microsoft.com/en-us/office/office365/HowTo/add-common-consent-manually#bk_RegisterServerApp) y tenga en cuenta lo siguiente:

* Especifique una ruta en su aplicación de Ruby como **dirección URL de inicio de sesión** en el paso 6. En el caso del ejemplo Connect, la ruta es [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L41).
* [Configure los **permisos delegados**](https://github.com/microsoftgraph/ruby-connect-rest-sample/wiki/Grant-permissions-to-the-Connect-application-in-Azure) que requiera su aplicación. En el ejemplo Connect, se requiere el permiso **Enviar correo como usuario con sesión iniciada**.

Anote los siguientes valores de la página **Configurar** de la aplicación de Azure.

* Identificador de cliente
* Una clave válida
* Una dirección URL de respuesta

Necesita estos valores como parámetros en el flujo de OAuth en su aplicación.

<!--<a name="redirect"/>-->
## Redirigir al explorador a la página de inicio de sesión

Su aplicación necesita redirigir el explorador a la página de inicio de sesión para obtener un código de autorización y continuar el flujo de OAuth.

En el ejemplo Connect, la biblioteca OmniAuth controla el redireccionamiento. Nuestra aplicación simplemente delega la ejecución a la ruta [`/auth/azureactivedirectory`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L30) que administra OmniAuth.

<!--<a name="authcode"/>-->
## Recibir un código de autorización en la página de la dirección URL de respuesta

Cuando el usuario inicia sesión, el flujo devuelve el explorador a la dirección URL de respuesta en su aplicación. Azure anexa un código de autorización a la cadena de consulta. En el ejemplo Connect, se usa la ruta [`/auth/azureactivedirectory/callback`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L38) para este propósito.

El código de autorización se proporciona en la variable de la cadena de consulta `code`. En el ejemplo Connect, se guarda el código en una variable local para usarlo más adelante.

```ruby
code = params[:code]
```

<!--<a name="accesstoken"/>-->
## Solicitar un token de acceso desde el punto de conexión del token

Cuando tenga el código de autorización, podrá usarlo junto con los valores de los parámetros ID de cliente, Clave y Dirección URL de respuesta que obtuvo en Azure AD para solicitar un token de acceso. 

> **Nota:** <br />La solicitud debe especificar también un recurso que estemos intentando usar. En el caso de Microsoft Graph, el valor de recurso es `https://graph.microsoft.com`.

De nuevo, en el ejemplo Connect, se delega esta tarea a la biblioteca OmniAuth. La función [`acquire_access_token`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L65) llama a la biblioteca y pasa el código de autenticación guardado en la sección anterior junto con la dirección URL de respuesta, el ID de cliente, el secreto de cliente y el ID de recurso.

```ruby
def acquire_access_token(auth_code, reply_url)
    AUTH_CTX.acquire_token_with_authorization_code(
                  auth_code,
                  reply_url,
                  CLIENT_CRED,
                  GRAPH_RESOURCE)
end
```

> **Nota:** <br />
> El ID de cliente y el secreto de cliente se proporcionan en el parámetro `CLIENT_CRED` en el fragmento de código anterior.

<!--<a name="request"/>-->
## Usar el token de acceso en una solicitud a la API de Microsoft Graph

El token de acceso permite que su aplicación cree solicitudes autenticadas en la API de Microsoft Graph. Su aplicación debe proporcionar el token de acceso en el encabezado de **autorización** de cada solicitud.

En el ejemplo Connect, se envía un correo electrónico usando el punto de conexión **sendEmail** en la API de Microsoft Graph. El código está en la función [`send_mail`](https://github.com/microsoftgraph/ruby-connect-rest-sample/blob/master/app/controllers/pages_controller.rb#L82). Este es el código que muestra cómo enviar el código de acceso en el encabezado de autorización.

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

> **Nota:** <br />
> La solicitud también debe enviar un encabezado **Content-Type** con un valor que acepte la API de Microsoft Graph. Por ejemplo, `application/json;odata.metadata=minimal;odata.streaming=true`.

La API de Microsoft Graph es una interfaz unificadora muy potente que permite trabajar con todos los tipos de datos de Microsoft. Consulte la referencia de la API para ver todas las posibilidades que ofrece la API de Microsoft Graph.

<!--## Additional resources

-  [Office 365 Ruby Connect sample using Microsoft Graph](https://github.com/microsoftgraph/ruby-connect-rest-sample)
-  [Office Dev Center](http://dev.office.com) 
-  [Microsoft Graph API reference](http://graph.microsoft.io/en-us/docs)-->
