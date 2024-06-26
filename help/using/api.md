---
title: "[!DNL Asset Compute Service] API HTTP"
description: '"[!DNL Asset Compute Service] API HTTP para crear aplicaciones personalizadas".'
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '2862'
ht-degree: 2%

---

# [!DNL Asset Compute Service] API HTTP {#asset-compute-http-api}

El uso de la API se limita a fines de desarrollo. La API se proporciona como contexto al desarrollar aplicaciones personalizadas. [!DNL Adobe Experience Manager] as a [!DNL Cloud Service] utiliza la API para pasar la información de procesamiento a una aplicación personalizada. Para obtener más información, consulte [Uso de microservicios de recursos y perfiles de procesamiento](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] solo está disponible para su uso con [!DNL Experience Manager] as a [!DNL Cloud Service].

Cualquier cliente del [!DNL Asset Compute Service] La API HTTP debe seguir este flujo de alto nivel:

1. Un cliente se aprovisiona como [!DNL Adobe Developer Console] proyecto en una organización IMS. Cada cliente independiente (sistema o entorno) requiere su propio proyecto independiente para separar el flujo de datos de evento.

1. Un cliente genera un token de acceso para la cuenta técnica mediante [Autenticación JWT (cuenta de servicio)](https://developer.adobe.com/developer-console/docs/guides/).

1. Un cliente llama a [`/register`](#register) solo una vez para recuperar la dirección URL del diario.

1. Un cliente llama a [`/process`](#process-request) para cada recurso para el que desea generar representaciones. La llamada es asincrónica.

1. Un cliente sondea regularmente el diario para [recibir eventos](#asynchronous-events). Recibe eventos para cada representación solicitada cuando la representación se procesa correctamente (`rendition_created` tipo de evento) o si hay un error (`rendition_failed` tipo de evento).

El [adobe-asset-compute-client](https://github.com/adobe/asset-compute-client) facilita el uso de la API en el código de Node.js.

## Autenticación y autorización {#authentication-and-authorization}

Todas las API requieren autenticación de token de acceso. Las solicitudes deben establecer los siguientes encabezados:

1. `Authorization` encabezado con token de portador, que es el token de la cuenta técnica, recibido a través de [Intercambio JWT](https://developer.adobe.com/developer-console/docs/guides/) del proyecto de Adobe Developer Console. El [ámbitos](#scopes) se documentan a continuación.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. `x-gw-ims-org-id` encabezado con el ID de organización de IMS.

1. `x-api-key` con el ID de cliente de [!DNL Adobe Developers Console] proyecto.

### Ámbitos {#scopes}

Asegúrese de que los siguientes ámbitos sean válidos para el token de acceso:

* `openid`
* `AdobeID`
* `asset_compute`
* `read_organizations`
* `event_receiver`
* `event_receiver_api`
* `adobeio_api`
* `additional_info.roles`
* `additional_info.projectedProductContext`

Estos ámbitos requieren lo siguiente [!DNL Adobe Developer Console] proyecto al que se va a suscribir `Asset Compute`, `I/O Events`, y `I/O Management API` servicios. El desglose de ámbitos individuales es el siguiente:

* Básica
   * ámbitos: `openid,AdobeID`

* Asset compute
   * metascope: `asset_compute_meta`
   * ámbitos: `asset_compute,read_organizations`

* Adobe[!DNL `I/O Events`]
   * metascope: `event_receiver_api`
   * ámbitos: `event_receiver,event_receiver_api`

* Adobe[!DNL `I/O Management API`]
   * metascope: `ent_adobeio_sdk`
   * ámbitos: `adobeio_api,additional_info.roles,additional_info.projectedProductContext`

## Registro {#register}

Cada cliente del [!DNL Asset Compute service] - un único [!DNL Adobe Developer Console] proyecto suscrito al servicio: debe [registrar](#register-request) antes de realizar solicitudes de procesamiento. El paso de registro devuelve el diario de eventos único necesario para recuperar los eventos asincrónicos del procesamiento de la representación.

Al final de su ciclo de vida, un cliente puede [anular el registro](#unregister-request).

### Solicitud de registro {#register-request}

Esta llamada de API configura un [!DNL Asset Compute] y proporciona la URL del diario de eventos. Este proceso es una operación idempotente y solo necesita llamarse una vez para cada cliente. Se puede volver a llamar para recuperar la dirección URL del historial.

| Parámetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Ruta | `/register` |
| Header `Authorization` | Todo [encabezados relacionados con la autorización](#authentication-and-authorization). |
| Header `x-request-id` | Opcional, establecida por los clientes para un identificador de extremo a extremo único de las solicitudes de procesamiento entre sistemas. |
| Cuerpo de solicitud | Debe estar vacío. |

### Registrar respuesta {#register-response}

| Parámetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Header `X-Request-Id` | Es igual que el `X-Request-Id` encabezado de solicitud o uno generado de forma única. Se utiliza para identificar solicitudes entre sistemas, solicitudes de soporte o ambas cosas. |
| Cuerpo de respuesta | Un objeto JSON con `journal`, `ok`, o `requestId` campos. |

Los códigos de estado HTTP son:

* **200 Correctos**: cuando la solicitud se realiza correctamente. El `journal` La URL recibe notificaciones sobre los resultados del procesamiento asincrónico iniciado mediante `/process`. Se alerta de `rendition_created` eventos tras su finalización correcta, o `rendition_failed` eventos si falla el proceso.

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 No autorizado**: se produce cuando la solicitud no tiene un valor válido [authentication](#authentication-and-authorization). Un ejemplo puede ser un token de acceso no válido o una clave de API no válida.

* **403 Prohibido**: se produce cuando la solicitud no tiene un valor válido [autorización](#authentication-and-authorization). Un ejemplo puede ser un token de acceso válido, pero el proyecto de Adobe Developer Console (cuenta técnica) no está suscrito a todos los servicios necesarios.

* **Demasiadas solicitudes**: se produce cuando este cliente o de otro modo sobrecarga el sistema. Los clientes deben volver a intentarlo con un [retroceso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). El cuerpo está vacío.
* **Error 4xx**: Cuando se produjo cualquier otro error de cliente y el registro falló. Normalmente se devuelve una respuesta JSON como esta, aunque no está garantizado para todos los errores:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Error 5xx**: se produce cuando se produjo cualquier otro error del lado del servidor y falló el registro. Normalmente se devuelve una respuesta JSON como esta, aunque no está garantizado para todos los errores:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### Solicitud de cancelación de registro {#unregister-request}

Esta llamada de API anula el registro de un [!DNL Asset Compute] cliente. Una vez cancelado el registro, ya no es posible llamar a `/process`. El uso de la llamada de API para un cliente no registrado o un cliente aún no registrado devuelve un `404` error.

| Parámetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Ruta | `/unregister` |
| Header `Authorization` | Todo [encabezados relacionados con la autorización](#authentication-and-authorization). |
| Header `x-request-id` | Opcional. Los clientes pueden configurarlo para obtener un identificador completo único de las solicitudes de procesamiento entre sistemas. |
| Cuerpo de solicitud | Vacío. |

### Anular registro de respuesta {#unregister-response}

| Parámetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Header `X-Request-Id` | Es igual que el `X-Request-Id` encabezado de solicitud o uno generado de forma única. Se utiliza para identificar solicitudes entre sistemas o solicitudes de asistencia. |
| Cuerpo de respuesta | Un objeto JSON con `ok` y `requestId` campos. |

Los códigos de estado son:

* **200 Correctos**: se produce cuando se encuentra y se elimina el registro y el historial.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401 No autorizado**: se produce cuando la solicitud no tiene un valor válido [authentication](#authentication-and-authorization). Un ejemplo puede ser un token de acceso no válido o una clave de API no válida.

* **403 Prohibido**: se produce cuando la solicitud no tiene un valor válido [autorización](#authentication-and-authorization). Un ejemplo puede ser un token de acceso válido, pero el proyecto de Adobe Developer Console (cuenta técnica) no está suscrito a todos los servicios necesarios.

* **404 No encontrado**: este estado aparece cuando las credenciales proporcionadas no están registradas o no son válidas.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **Demasiadas solicitudes**: se produce cuando el sistema está sobrecargado. Los clientes deben volver a intentarlo con un [retroceso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). El cuerpo está vacío.

* **Error 4xx**: se produce cuando se produjo cualquier otro error de cliente y falló la anulación del registro. Normalmente se devuelve una respuesta JSON como esta, aunque no está garantizado para todos los errores:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Error 5xx**: se produce cuando se produjo cualquier otro error del lado del servidor y falló el registro. Normalmente se devuelve una respuesta JSON como esta, aunque no está garantizado para todos los errores:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## Procesar solicitud {#process-request}

El `process` Esta operación envía un trabajo que transforma un recurso de origen en varias representaciones, según las instrucciones de la solicitud. Notificaciones sobre finalización correcta (tipo de evento) `rendition_created`) o cualquier error (tipo de evento `rendition_failed`) se envían a un diario de eventos que debe recuperarse mediante [`/register`](#register) una vez antes de realizar cualquier número de `/process` solicitudes. Las solicitudes formadas incorrectamente generan un error 400.

Se hace referencia a los binarios mediante direcciones URL, como URL prefirmadas de Amazon AWS S3 o URL de SAS de almacenamiento de Azure Blob. Se utiliza para leer el `source` recurso (`GET` URL) y escribir las representaciones (`PUT` URL). El cliente es responsable de generar estas direcciones URL prefirmadas.

| Parámetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Ruta | `/process` |
| Tipo MIME | `application/json` |
| Header `Authorization` | Todo [encabezados relacionados con la autorización](#authentication-and-authorization). |
| Header `x-request-id` | Opcional. Los clientes pueden establecer un identificador de extremo a extremo único para realizar el seguimiento de las solicitudes de procesamiento entre sistemas. |
| Cuerpo de solicitud | Debe tener el formato JSON de solicitud de proceso que se describe a continuación. Proporciona instrucciones sobre qué recurso procesar y qué representaciones generar. |

### Solicitud de proceso JSON {#process-request-json}

El cuerpo de la solicitud de `/process` es un objeto JSON con este esquema de alto nivel:

```json
{
    "source": "",
    "renditions" : []
}
```

Los campos disponibles son:

| Nombre | Tipo | Descripción | Ejemplos |
|--------------|----------|-------------|---------|
| `source` | `string` | URL del recurso de origen que se procesa. Opcional, según el formato de representación solicitado (por ejemplo, `fmt=zip`). | `"http://example.com/image.jpg"` |
| `source` | `object` | Descripción del recurso de origen que se procesa. Consulte la descripción de [Campos de objeto de Source](#source-object-fields) más abajo. Opcional en función del formato de representación solicitado (por ejemplo, `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Representaciones que se generarán a partir del archivo de origen. Cada objeto de representación admite un [instrucción de representación](#rendition-instructions). Requerido. | `[{ "target": "https://....", "fmt": "png" }]` |

El `source` puede ser un `<string>` que se ve como una dirección URL o puede ser un `<object>` con un campo adicional. Las siguientes variantes son similares:

```json
"source": "http://example.com/image.jpg"
```

```json
"source": {
    "url": "http://example.com/image.jpg"
}
```

### Campos de objeto de Source {#source-object-fields}

| Nombre | Tipo | Descripción | Ejemplos |
|-----------|----------|-------------|---------|
| `url` | `string` | URL del recurso de origen que se va a procesar. Requerido. | `"http://example.com/image.jpg"` |
| `name` | `string` | Nombre del archivo del recurso Source. Se puede usar una extensión de archivo en el nombre si no se detecta ningún tipo MIME. Tiene prioridad sobre el nombre de archivo especificado en la ruta de acceso URL. Además, tiene prioridad sobre el nombre del archivo en `content-disposition` encabezado del recurso binario. El valor predeterminado es &quot;archivo&quot;. | `"image.jpg"` |
| `size` | `number` | Tamaño del archivo del recurso de Source en bytes. Tiene prioridad sobre `content-length` encabezado del recurso binario. | `10234` |
| `mimetype` | `string` | Tipo MIME del archivo del recurso de Source. Tiene prioridad sobre `content-type` encabezado del recurso binario. | `"image/jpeg"` |

### Una completa `process` ejemplo de solicitud {#complete-process-request-example}

```json
{
    "source": "https://www.adobe.com/content/dam/acom/en/lobby/lobby-bg-bts2017-logged-out-1440x860.jpg",
    "renditions" : [{
            "name": "image.48x48.png",
            "target": "https://some-presigned-put-url-for-image.48x48.png",
            "fmt": "png",
            "width": 48,
            "height": 48
        },{
            "name": "image.200x200.jpg",
            "target": "https://some-presigned-put-url-for-image.200x200.jpg",
            "fmt": "jpg",
            "width": 200,
            "height": 200
        },{
            "name": "cqdam.xmp.xml",
            "target": "https://some-presigned-put-url-for-cqdam.xmp.xml",
            "fmt": "xmp"
        },{
            "name": "cqdam.text.txt",
            "target": "https://some-presigned-put-url-for-cqdam.text.txt",
            "fmt": "text"
    }]
}
```

## Respuesta del proceso {#process-response}

El `/process` La solicitud de se devuelve inmediatamente con un resultado correcto o incorrecto en función de la validación de solicitud básica. El procesamiento real de recursos se produce de forma asíncrona.

| Parámetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Header `X-Request-Id` | Es igual que el `X-Request-Id` encabezado de solicitud o uno generado de forma única. Se utiliza para identificar solicitudes entre sistemas o solicitudes de asistencia. |
| Cuerpo de respuesta | Un objeto JSON con `ok` y `requestId` campos. |

Códigos de estado:

* **200 Correctos**: si la solicitud se envió correctamente. El JSON de respuesta incluye `"ok": true`:

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **400 Solicitud no válida**: Si la solicitud está estructurada incorrectamente, por ejemplo, si carece de campos obligatorios en la carga útil JSON. El JSON de respuesta incluye `"ok": false`:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **401 No autorizado**: Cuando la solicitud no tiene validez [authentication](#authentication-and-authorization). Un ejemplo puede ser un token de acceso no válido o una clave de API no válida.
* **403 Prohibido**: Cuando la solicitud no tiene validez [autorización](#authentication-and-authorization). Un ejemplo puede ser un token de acceso válido, pero el proyecto de Adobe Developer Console (cuenta técnica) no está suscrito a todos los servicios necesarios.
* **Demasiadas solicitudes**: Se produce cuando el sistema está saturado, ya sea debido a este cliente en particular o a la demanda general. Los clientes pueden volver a intentarlo con un [retroceso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). El cuerpo está vacío.
* **Error 4xx**: Cuando se produjo cualquier otro error de cliente. Normalmente se devuelve una respuesta JSON como esta, aunque no está garantizado para todos los errores:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Error 5xx**: Cuando se produjo cualquier otro error del lado del servidor. Normalmente se devuelve una respuesta JSON como esta, aunque no está garantizado para todos los errores:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

La mayoría de los clientes tienden a reintentar la misma solicitud con [retroceso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff) sobre cualquier error *excepto* problemas de configuración como 401 o 403, o solicitudes no válidas como 400. Aparte de la limitación regular de la velocidad mediante respuestas 429, una interrupción temporal del servicio o una limitación podría provocar errores 5xx. Entonces, sería aconsejable volver a intentarlo después de un periodo de tiempo.

Todas las respuestas de JSON (si están presentes) incluyen las siguientes `requestId`, que es el mismo valor que `X-Request-Id` encabezado. El Adobe recomienda leer del encabezado porque siempre está presente. El `requestId` también se devuelve en todos los eventos relacionados con el procesamiento de solicitudes como `requestId`. Los clientes no deben suponer el formato de esta cadena. Es un identificador de cadena opaco.

## Inclusión en el posprocesamiento {#opt-in-to-post-processing}

El [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk) admite un conjunto de opciones básicas de posprocesamiento de imágenes. Los trabajadores personalizados pueden incluirse explícitamente en el posprocesamiento configurando el campo `postProcess` en el objeto de representación a `true`.

Los casos de uso admitidos son:

* Recortar es una representación en un rectángulo cuyos límites están definidos por crop.w, crop.h, crop.x y crop.y. Los detalles de recorte se especifican en el objeto de representación `instructions.crop` field.
* Cambie el tamaño de las imágenes mediante los valores de anchura, altura o ambos. El `instructions.width` y `instructions.height` lo define en el objeto de representación. Para cambiar el tamaño sólo con anchura o altura, defina sólo un valor. Compute Service conserva la relación de aspecto.
* Establezca la calidad de una imagen JPEG. El `instructions.quality` lo define en el objeto de representación. Un nivel de calidad de 100 representa la calidad más alta, mientras que números más bajos significan una disminución en la calidad.
* Cree imágenes entrelazadas. El `instructions.interlace` lo define en el objeto de representación.
* Establezca PPP para ajustar el tamaño procesado para la publicación en el escritorio ajustando la escala aplicada a los píxeles. El `instructions.dpi` lo define en el objeto de representación para cambiar la resolución de ppp. Sin embargo, para cambiar el tamaño de la imagen de modo que tenga el mismo tamaño con una resolución diferente, utilice el `convertToDpi` instrucciones.
* Cambie el tamaño de la imagen de modo que la anchura o altura procesadas sean las mismas que las originales con la resolución de destino especificada (PPP). El `instructions.convertToDpi` lo define en el objeto de representación.

## Recursos de marca de agua {#add-watermark}

El [ASSET COMPUTE SDK](https://github.com/adobe/asset-compute-sdk) admite agregar una marca de agua a los archivos de imagen PNG, JPEG, TIFF y GIF. La marca de agua se agrega siguiendo las instrucciones de representación de la `watermark` en la representación.

La marca de agua se realiza durante el procesamiento posterior de la representación. Para crear marcas de agua en los recursos, el trabajador personalizado [opta por el posprocesamiento](#opt-in-to-post-processing) estableciendo el campo `postProcess` en el objeto de representación a `true`. Si el trabajador no selecciona, la marca de agua no se aplica, incluso si el objeto de marca de agua está establecido en el objeto de representación de la solicitud.

## Instrucciones de representación {#rendition-instructions}

Las siguientes son las opciones disponibles para `renditions` matriz en [`/process`](#process-request).

### Campos comunes {#common-fields}

| Nombre | Tipo | Descripción | Ejemplos |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | El formato de destino de las representaciones también puede ser `text` para la extracción de texto y `xmp` XMP para extraer metadatos de la como xml. Consulte [formatos admitidos](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support) | `png` |
| `worker` | `string` | URL de un [aplicación personalizada](develop-custom-application.md). Debe ser un `https://` URL. Si este campo está presente, una aplicación personalizada crea la representación. A continuación, se utiliza cualquier otro campo de representación definido en la aplicación personalizada. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | Dirección URL a la que se debe cargar la representación generada mediante el PUT HTTP. | `http://w.com/img.jpg` |
| `target` | `object` | Información de carga de URL firmada previamente de varias partes para la representación generada. Esta información es para [AEM Carga binaria directa de Oak/](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) con esto [comportamiento de carga multiparte](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Campos:<ul><li>`urls`: matriz de cadenas, una para cada URL de parte firmada previamente</li><li>`minPartSize`: el tamaño mínimo que se debe utilizar para una parte = url</li><li>`maxPartSize`: el tamaño máximo que se utilizará para una parte = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Opcional. El cliente controla el espacio reservado y lo pasa tal cual a los eventos de representación. Permite que un cliente agregue información personalizada para identificar eventos de representación. No se debe modificar ni confiar en él en las aplicaciones personalizadas, ya que los clientes pueden cambiarlo en cualquier momento. | `{ ... }` |

### Campos específicos de representación {#rendition-specific-fields}

Para obtener una lista de los formatos de archivo admitidos actualmente, consulte [formatos de archivo compatibles](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/file-format-support).

| Nombre | Tipo | Descripción | Ejemplos |
|-------------------|----------|-------------|---------|
| `*` | `*` | Avanzados, se pueden añadir campos personalizados que [aplicación personalizada](develop-custom-application.md) lo entiende. | |
| `embedBinaryLimit` | `number` en bytes | Cuando el tamaño de archivo de la representación es menor que el valor especificado, se incluye en el evento enviado una vez finalizada su creación. El tamaño máximo permitido para la incrustación es de 32 KB (32 x 1024 bytes). Si una representación tiene un tamaño mayor que el `embedBinaryLimit` se coloca en una ubicación en el almacenamiento de la nube y no está incrustado en el evento. | `3276` |
| `width` | `number` | Anchura en píxeles. solo para representaciones de imágenes. | `200` |
| `height` | `number` | Altura en píxeles. solo para representaciones de imágenes. | `200` |
|                   |          | La relación de aspecto se mantiene siempre si: <ul> <li> Ambos `width` y `height` se especifican, la imagen se ajusta al tamaño manteniendo la relación de aspecto </li><li> Si solo `width` o `height` se especifica, la imagen resultante utiliza la dimensión correspondiente y mantiene la relación de aspecto</li><li> If `width` o `height` no se ha especificado, se utiliza el tamaño de píxel de imagen original. Depende del tipo de origen. En algunos formatos, como los archivos de PDF, se utiliza un tamaño predeterminado. Puede haber un límite de tamaño máximo.</li></ul> | |
| `quality` | `number` | Especifique la calidad JPEG en el rango de `1` hasta `100`. Aplicable solo para representaciones de imágenes. | `90` |
| `xmp` | `string` | XMP XMP Solo se utiliza para la reescritura de metadatos de la, se codifica en base64 para volver a escribir en la representación especificada. | |
| `interlace` | `bool` | Cree un PNG entrelazado, un GIF o un JPEG progresivo estableciéndolo en `true`. No afecta a otros formatos de archivo. | |
| `jpegSize` | `number` | Tamaño aproximado del archivo del JPEG en bytes. Anula cualquier `quality` configuración. No afecta a otros formatos. | |
| `dpi` | `number` o `object` | Establezca los ppp x e y. Para simplificar, también se puede configurar en un solo número, que se utiliza tanto para x como para y. No tiene ningún efecto en la propia imagen. | `96` o `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` o `object` | Los ppp x e y vuelven a muestrear los valores manteniendo el tamaño físico. Para simplificar, también se puede configurar en un solo número, que se utiliza tanto para x como para y. | `96` o `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lista de archivos que se incluirán en el archivo ZIP (`fmt=zip`). Cada entrada puede ser una cadena URL o un objeto con los campos:<ul><li>`url`: URL para descargar el archivo</li><li>`path`: almacena el archivo en esta ruta en el ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Administración de duplicados para archivos ZIP (`fmt=zip`). De forma predeterminada, varios archivos almacenados en la misma ruta en el ZIP generan un error. Configuración `duplicate` hasta `ignore` Esto hace que solo se almacene el primer recurso y que el resto se ignore. | `ignore` |
| `watermark` | `object` | Contiene instrucciones sobre el [filigrana](#watermark-specific-fields). |  |

### Campos específicos de marcas de agua {#watermark-specific-fields}

El formato PNG se utiliza como marca de agua.

| Nombre | Tipo | Descripción | Ejemplos |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Escala de la marca de agua, entre `0.0` y `1.0`. `1.0` significa que la marca de agua tiene su escala original (1:1) y los valores más bajos reducen el tamaño de la marca de agua. | Un valor de `0.5` significa la mitad del tamaño original. |
| `image` | `url` | URL del archivo PNG que se utilizará para la marca de agua. | |

## Eventos asíncronos {#asynchronous-events}

Cuando finaliza el procesamiento de una representación o se produce un error, se envía un evento a un Adobe [!DNL `I/O Events Journal`]. Los clientes deben escuchar la URL del diario proporcionada mediante [`/register`](#register). La respuesta del diario incluye un `event` matriz que consta de un objeto para cada evento, del cual el `event` incluye la carga útil del evento real.

El Adobe [!DNL `I/O Events`] tipo para todos los eventos del [!DNL Asset Compute Service] es `asset_compute`. El historial solo se suscribe automáticamente a este tipo de evento y no hay ningún requisito adicional de filtrar según el [!DNL Adobe Developer] Tipo de evento. Los tipos de eventos específicos del servicio están disponibles en la variable `type` propiedad del evento.

### Tipos de eventos {#event-types}

| Evento | Descripción |
|---------------------|-------------|
| `rendition_created` | Se envía por cada representación cargada y procesada correctamente. |
| `rendition_failed` | Se envía por cada representación que no se procesa ni se carga. |

### Atributos de evento {#event-attributes}

| Atributo | Tipo | Evento | Descripción |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Marca de tiempo de cuando el evento se envió en simplificado y extendido [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601) , tal como se define en JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | El ID de solicitud de la solicitud original a `/process`, igual que `X-Request-Id` encabezado. |
| `source` | `object` | `*` | El `source` de la `/process` solicitud. |
| `userData` | `object` | `*` | El `userData` de la representación desde el `/process` solicitud si está establecido. |
| `rendition` | `object` | `rendition_*` | El objeto de representación correspondiente pasado `/process`. |
| `metadata` | `object` | `rendition_created` | El [metadatos](#metadata) propiedades de la representación. |
| `errorReason` | `string` | `rendition_failed` | Error de representación [razonar](#error-reasons) en su caso. |
| `errorMessage` | `string` | `rendition_failed` | El texto que proporciona más detalles sobre el error de representación, si lo hay. |

### Metadatos {#metadata}

| Propiedad | Descripción |
|--------|-------------|
| `repo:size` | El tamaño de la representación en bytes. |
| `repo:sha1` | El resumen sha1 de la representación. |
| `dc:format` | Tipo MIME de la representación. |
| `repo:encoding` | La codificación charset de la representación en caso de que sea un formato basado en texto. |
| `tiff:ImageWidth` | Anchura de la representación en píxeles. Solo está presente para representaciones de imágenes. |
| `tiff:ImageLength` | Longitud de la representación en píxeles. Solo está presente para representaciones de imágenes. |

### Motivos de error {#error-reasons}

| Motivo | Descripción |
|---------|-------------|
| `RenditionFormatUnsupported` | El formato de representación solicitado no es compatible con el origen determinado. |
| `SourceUnsupported` | El origen específico no es compatible aunque el tipo sea compatible. |
| `SourceCorrupt` | Los datos de origen están dañados. Incluye archivos vacíos. |
| `RenditionTooLarge` | No se pudo cargar la representación mediante las direcciones URL firmadas previamente en `target`. El tamaño real de la representación está disponible como metadatos en `repo:size` y lo utiliza el cliente para volver a procesar esta representación con el número correcto de direcciones URL prefirmadas. |
| `GenericError` | Cualquier otro error inesperado. |
