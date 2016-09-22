
# Paginación de los datos de Microsoft Graph en su aplicación 
 
Cuando las solicitudes de Microsoft Graph devuelven demasiada información para mostrarla en una página, puede usar la paginación para dividir la información en fragmentos manejables. 

Puede adelantar o retroceder una página en las respuestas de Microsoft Graph. Una respuesta que contiene resultados paginados incluirá un token de omisión (**odata.nextLink**) que le permite obtener la siguiente página de resultados. Este token de omisión puede combinarse con un argumento de consulta **previous-page=true** para retroceder una página.

En el siguiente ejemplo de solicitud se muestra la paginación hacia adelante:

```
https://graph.microsoft.com/v1.0/users?$top=5$skiptoken=X'4453707402.....0000'
```
El parámetro **$skiptoken** de la respuesta anterior se incluye, y le permite obtener la siguiente página de resultados.

En el siguiente ejemplo de solicitud se muestra la paginación hacia atrás:

```
https://graph.microsoft.com/v1.0/users?$top=5$skiptoken=X'4453707.....00000'&previous-page=true
```
El parámetro **$skiptoken** de la respuesta anterior se incluye. Cuando esto se combina con el parámetro **&previous-page=true**, la página anterior de los resultados se recuperará.

Los siguientes son los pasos de solicitud y respuesta para adelantar y retroceder de página:

1. Se realiza una solicitud para obtener una lista de los primeros 10 usuarios de 15. La respuesta contiene un token de omisión para indicar la página final de los 10 usuarios.
2. Para obtener los 5 usuarios finales, se realiza otra solicitud que contiene el token de omisión devuelto de la respuesta previa.
3. Para retroceder la página, se realiza una solicitud usando el token de omisión devuelto en el paso 1 y se agrega el parámetro **&previous-page=true** a la solicitud.
4. La respuesta contiene la página anterior (primera) de 10 usuarios. En un escenario diferente donde quedan más páginas, se devolverá un nuevo token de omisión. Este nuevo token de omisión puede agregarse a la solicitud junto con **&previous-page=true** para retroceder una página de nuevo.

Las siguientes restricciones se aplican a las solicitudes de paginación:

- El tamaño predeterminado de la página es 100. El tamaño de página máximo es 999.
- Las consultas contra los roles no son compatibles con la paginación. Esto incluye los objetos de rol de lectura así como los miembros de rol.
- La paginación no es compatible con las búsquedas de vínculos, como consultar los miembros de grupo.
