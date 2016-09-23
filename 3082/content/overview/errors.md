# Respuestas de error de Microsoft Graph y tipos de recursos

<!--In this article:
  
-   [Status code](#msg_status_code)
-   [Error resource type](#msg_error_resource_type)
-   [Code property](#msg_code_property)

<a name="msg_error_response"> </a> -->

Los errores en Microsoft Graph se devuelven usando los códigos de estado HTTP estándar, así como un objeto de respuesta de error JSON.

## Códigos de estado HTTP

La siguiente tabla enumera y describe los códigos de estado HTTP que se pueden devolver.

| Código de estado | Mensaje de estado                  | Descripción                                                                                                                            |
|:------------|:--------------------------------|:---------------------------------------------------------------------------------------------------------------------------------------|
| 400         | Solicitud incorrecta                     | No se puede procesar la solicitud porque es incorrecta o tiene un formato no válido.                                                                       |
| 401         | No autorizado                    | La información de autenticación requerida no se encuentra o no es válida para el recurso.                                                   |
| 403         | Prohibido                       | Se denegó el acceso al recurso solicitado. Puede que el usuario no tenga permisos suficientes.                                                 |
| 404         | No encontrado                       | El recurso solicitado no existe.                                                                                                  |
| 405         | Método no permitido              | No se permite el método HTTP de la solicitud en el recurso.                                                                         |
| 406         | No es aceptable                  | Este servicio no es compatible con el formato solicitado en el encabezado Accept.                                                                |
| 409         | Conflicto                        | El estado actual entra en conflicto con lo que la solicitud espera. Por ejemplo, la carpeta principal especificada podría no existir.                   |
| 410         | Pasado                            | El recurso solicitado ya no está disponible en el servidor.                                               |
| 411         | Longitud requerida                 | Se requiere un encabezado Content-Length en la solicitud.                                                                                    |
| 412         | Error de condición previa             | Una condición previa proporcionada en la solicitud (por ejemplo, un encabezado if-match) no coincide con el estado actual del recurso.                       |
| 413         | Entidad de solicitud demasiado grande        | El tamaño de la solicitud supera el límite máximo.                                                                                            |
| 415         | Tipo de medio no compatible          | El tipo de contenido de la solicitud es un formato que no es compatible con el servicio.                                                      |
| 416         | El rango solicitado no se cumple | El rango de bytes especificado no es válido o no está disponible.                                                                                    |
| 422         | Entidad no procesable            | No se puede procesar la solicitud porque es semánticamente incorrecta.                                                                       |
| 429         | Demasiadas solicitudes               | La aplicación del cliente se ha limitado y no debería intentar repetir la solicitud hasta haya transcurrido un periodo de tiempo.                |
| 500         | Error interno del servidor           | Se produjo un error interno del servidor al procesar la solicitud.                                                                       |
| 501         | No implementado                 | La característica solicitada no se implementó.                                                                                               |
| 503         | Servicio no disponible             | El servicio no está disponible temporalmente. Puede repetir la solicitud después de un retraso. Puede haber un encabezado Retry-After.                   |
| 507         | Almacenamiento insuficiente            | Se ha alcanzado la cuota de almacenamiento máxima.                                                                                            |
| 509         | Ha superado el límite de ancho de banda        | Su aplicación se ha limitado por superar el extremo máximo de ancho de banda. Su aplicación puede reintentar la solicitud de nuevo cuando haya transcurrido más tiempo. |

La respuesta de error es un solo objeto JSON que contiene una propiedad única denominada **error**. Este objeto incluye todos los detalles del error. Puede usar la información devuelta aquí en lugar o además del código de estado HTTP. A continuación se muestra un ejemplo de un cuerpo completo de error JSON.

<!-- { "blockType": "example", "@odata.type": "sample.error", "expectError": true, "name": "example-error-response"} -->
```json
{
  "error": {
    "code": "invalidRange",
    "message": "Uploaded fragment overlaps with existing data.",
    "innerError": {
      "requestId": "request-id",
      "date": "date-time"
    }
  }
}
```

<!--<a name="msg_error_resource_type"> </a> -->

## Tipo de recurso de error

El recurso de error se devuelve cuando se produce un error en el procesamiento de una solicitud.

Las respuestas de error siguen la definición en la especificación [OData v4](http://docs.oasis-open.org/odata/odata-json-format/v4.0/os/odata-json-format-v4.0-os.html#_Toc372793091) de las respuestas de error.

### Representación JSON

El recurso de error se compone de estos recursos:

<!-- { "blockType": "resource", "@odata.type": "sample.error" } -->
```json
{
  "error": { "@odata.type": "odata.error" }  
}
```

#### tipo de recurso de error OData

En la respuesta del error se encuentra un recurso de error que incluye las siguientes propiedades:

<!-- { "blockType": "resource", "@odata.type": "odata.error", "optionalProperties": [ "target", "details", "innererror"] } -->
```json
{
  "code": "string",
  "message": "string",
  "innererror": { "@odata.type": "odata.error" }
}
```

| Nombre de la propiedad  | Valor                  | Descripción\                                                                                               |
|:---------------|:-----------------------|:-----------------------------------------------------------------------------------------------------------|
| **código**       | cadena                 | Una cadena de código de error para el error que se ha producido                                                            |
| **mensaje**    | cadena                 | Un mensaje preparado de desarrollador sobre el error que se ha producido. Esto no debe mostrarse al usuario directamente. |
| **innererror** | objeto de error           | Opcional. Objetos de error adicionales que pueden ser más específicos que el error de nivel superior.                     |
<!-- {
  "type": "#page.annotation",
  "description": "Understand the error format for the API and error codes.",
  "keywords": "error response, error, error codes, innererror, message, code",
  "section": "documentation",
  "tocPath": "Misc/Error Responses"
} -->

<!--<a name="msg_code_property"> </a> -->

#### Propiedad del código

La propiedad `code` contiene uno de los siguientes valores posibles. Sus aplicaciones deben estar preparadas para controlar cualquiera de estos errores.

| Código                      | Descripción
|:--------------------------|:--------------
| **accessDenied**          | El autor de la llamada no tiene permiso para realizar la acción. 
| **activityLimitReached**  | Se ha limitado la aplicación o el usuario.
| **generalException**      | Se ha producido un error no especificado.
| **invalidRange**          | El rango de bytes especificado no es válido o no está disponible.
| **invalidRequest**        | La solicitud es incorrecta o tiene un formato no válido.
| **itemNotFound**          | No se pudo encontrar el recurso.
| **malwareDetected**       | Se ha detectado malware en el recurso solicitado.
| **nameAlreadyExists**     | El nombre del elemento especificado ya existe.
| **notAllowed**            | El sistema no permite esta acción.
| **notSupported**          | La solicitud no es compatible con el sistema.
| **resourceModified**      | El recurso que está siendo actualizado ha cambiado desde que el autor de llamada lo leyó la última vez, normalmente un error de coincidencia eTag.
| **resyncRequired**        | El token delta ya no es válido y la aplicación debe restablecer el estado de sincronización.
| **serviceNotAvailable**   | El servicio no está disponible. Intente la solicitud de nuevo tras un retraso. Puede haber un encabezado Retry-After. 
| **quotaLimitReached**     | El usuario ha alcanzado su límite de cuota.
| **unauthenticated**       | El autor de llamada no está autenticado.

El objeto `innererror` puede contener de forma recursiva más objetos `innererror` con códigos de error adicionales y más concretos. Al controlar un error, las aplicaciones deben recorrer todos los códigos de error disponibles y usar el más detallado que comprendan. Algunos de los códigos más detallados se enumeran en la parte inferior de esta página.

Para comprobar que un objeto de error es un error que está esperando, debe recorrer los objetos `innererror`, buscando los códigos de error que espera. Por ejemplo:

```csharp
public bool IsError(string expectedErrorCode)
{
    OneDriveInnerError errorCode = this.Error;
    while (null != errorCode)
    {
        if (errorCode.Code == expectedErrorCode)
            return true;
        errorCode = errorCode.InnerError;
    }
    return false;
}
```

Para obtener un ejemplo que muestra cómo controlar correctamente los errores, vea [Error Code Handling](https://gist.github.com/rgregg/a1866be15e685983b441) (Controlar códigos de error).

La propiedad `message` en la raíz contiene un mensaje de error destinado a que el desarrollador lo lea. Los mensajes de error no están localizados y no deberían mostrarse directamente al usuario. Al controlar los errores, el código no debería desconectarse de los valores `message` porque pueden cambiar en cualquier momento y, a menudo, contienen información dinámica específica de la solicitud de error. Solo debería codificar contra los códigos de error devueltos en las propiedades `code`.

#### Códigos de error detallados
A continuación, se presentan algunos errores adicionales que su aplicación puede encontrarse dentro de los objetos `innererror` anidados. Las aplicaciones no están obligadas a controlarlos, pero pueden hacerlo si lo desean. El servicio puede agregar nuevos códigos de error o dejar de devolver los antiguos en cualquier momento, por lo que es importante que todas las aplicaciones puedan controlar los [códigos de error básicos](#code-property).

| Código                               | Descripción
|:-----------------------------------|:----------------------------------------------------------
| **accessRestricted**               | Acceso restringido para el propietario del elemento.
| **cannotSnapshotTree**             | No se pudo obtener ninguna instantánea delta coherente. Inténtelo de nuevo más tarde.
| **childItemCountExceeded**         | Se ha alcanzado el límite máximo del número de elementos secundarios.
| **entityTagDoesNotMatch**          | La ETag no coincide con el valor del elemento actual.
| **fragmentLengthMismatch**         | El tamaño total declarado para este fragmento es distinto del de la sesión de carga.
| **fragmentOutOfOrder**             | El fragmento cargado está fuera de servicio.
| **fragmentOverlap**                | El fragmento cargado se superpone a los datos existentes.
| **invalidAcceptType**              | Tipo de aceptación no válido.
| **invalidParameterFormat**         | Formato de parámetro no válido.
| **invalidPath**                    | El nombre contiene caracteres no válidos.
| **invalidQueryOption**             | Opción de consulta no válida.
| **invalidStartIndex**              | Índice de inicio no válido.
| **lockMismatch**                   | El token de bloqueo no coincide con el bloqueo existente.
| **lockNotFoundOrAlreadyExpired**   | Actualmente no hay ningún bloqueo no caducado en el elemento.
| **lockOwnerMismatch**              | El id. del propietario de bloqueo no coincide con el id. proporcionado.
| **malformedEntityTag**             | El encabezado ETag tiene un formato no válido. Las ETags deben ser cadenas entre comillas.
| **maxDocumentCountExceeded**       | Se ha alcanzado el límite máximo del número de documentos.
| **maxFileSizeExceeded**            | Tamaño máximo de archivo excedido.
| **maxFolderCountExceeded**         | Se ha alcanzado el límite máximo del número de carpetas.
| **maxFragmentLengthExceeded**      | Tamaño máximo de archivo excedido.
| **maxItemCountExceeded**           | Se ha alcanzado el límite máximo de número de elementos.
| **maxQueryLengthExceeded**         | Se ha superado la longitud de consulta máxima.
| **maxStreamSizeExceeded**          | Se ha superado el tamaño máximo de secuencia.
| **parameterIsTooLong**             | El parámetro supera la longitud máxima.
| **parameterIsTooSmall**            | El parámetro es inferior al valor mínimo.
| **pathIsTooLong**                  | La ruta de acceso supera la longitud máxima.
| **pathTooDeep**                    | Se ha alcanzado el límite de profundidad de la jerarquía de carpetas.
| **propertyNotUpdateable**          | No se puede actualizar la propiedad.
| **resyncApplyDifferences**         | Es necesario volver a sincronizar. Reemplace cualquier elemento local con la versión del servidor (incluyendo eliminaciones) si está seguro de que el servicio estaba actualizado con sus cambios locales cuando lo sincronizó por última vez. Cargar cualquier cambio local que no conoce el servidor.
| **resyncRequired**                 | Es necesario volver a sincronizar.
| **resyncUploadDifferences**        | Es necesario volver a sincronizar. Cargue cualquier elemento local que el servicio no devolvió y cargue cualquier archivo que difiera de la versión del servidor (manteniendo ambas copias si no está seguro de cuál está más actualizada).
| **serviceNotAvailable**            | El servidor no puede procesar la solicitud actual.
| **serviceReadOnly**                | El recurso es temporalmente de solo lectura.
| **throttledRequest**               | Demasiadas solicitudes.
| **tooManyResultsRequested**        | Demasiados resultados solicitados.
| **tooManyTermsInQuery**            | Demasiados términos en la consulta.
| **totalAffectedItemCountExceeded** | No se permite la operación porque el número de elementos afectados supera el umbral.
| **truncationNotAllowed**           | No se permite el truncamiento de datos.
| **uploadSessionFailed**            | Error de la sesión de carga.
| **uploadSessionIncomplete**        | Sesión de carga incompleta.
| **uploadSessionNotFound**          | Sesión de carga no encontrada.
| **virusSuspicious**                | Este documento es sospechoso y puede tener un virus.
| **zeroOrFewerResultsRequested**    | Cero o menos resultados solicitados.

<!-- ##Additional Resources##

- [Microsoft Graph API release notes and known issues](microsoft-graph-api-release-notes-known-issues.md )
- [Hands on lab: Deep dive into the Microsoft Graph API](http://dev.office.com/hands-on-labs/4585) -->
