---
title: "[!DNL Asset Compute Service] API HTTP"
description: "[!DNL Asset Compute Service] API HTTP para crear aplicaciones personalizadas."
exl-id: 4b63fdf9-9c0d-4af7-839d-a95e07509750
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '2862'
ht-degree: 2%

---

# API HTTP [!DNL Asset Compute Service] {#asset-compute-http-api}

El uso de la API se limita a fines de desarrollo. La API se proporciona como contexto al desarrollar aplicaciones personalizadas. [!DNL Adobe Experience Manager] as a [!DNL Cloud Service] usa la API para pasar la información de procesamiento a una aplicación personalizada. Para obtener más información, consulte [Usar microservicios de recursos y Perfiles de procesamiento](https://experienceleague.adobe.com/es/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] solo está disponible para su uso con [!DNL Experience Manager] como [!DNL Cloud Service].

Cualquier cliente de la API HTTP [!DNL Asset Compute Service] debe seguir este flujo de alto nivel:

1. Se ha aprovisionado un cliente como un proyecto [!DNL Adobe Developer Console] en una organización IMS. Cada cliente independiente (sistema o entorno) requiere su propio proyecto independiente para separar el flujo de datos de evento.

1. Un cliente genera un token de acceso para la cuenta técnica mediante la autenticación [JWT (cuenta de servicio)](https://developer.adobe.com/developer-console/docs/guides/).

1. Un cliente llama a [`/register`](#register) solo una vez para recuperar la dirección URL del diario.

1. Un cliente llama a [`/process`](#process-request) para cada recurso para el que desea generar representaciones. La llamada es asincrónica.

1. Un cliente sondea regularmente el diario para [recibir eventos](#asynchronous-events). Recibe eventos para cada representación solicitada cuando la representación se procesa correctamente (tipo de evento `rendition_created`) o si hay un error (tipo de evento `rendition_failed`).

El módulo [adobe-asset-compute-client](https://github.com/adobe/asset-compute-client) facilita el uso de la API en el código de Node.js.

## Autenticación y autorización {#authentication-and-authorization}

Todas las API requieren autenticación de token de acceso. Las solicitudes deben establecer los siguientes encabezados:

1. `Authorization` encabezado con token de portador, que es el token de cuenta técnica, recibido a través de [intercambio JWT](https://developer.adobe.com/developer-console/docs/guides/) desde el proyecto Adobe Developer Console. Los [ámbitos](#scopes) se documentan a continuación.

<!-- TBD: Change the existing URL to a new path when a new path for docs is available. The current path contains master word that is not an inclusive term. Logged ticket in Adobe I/O's GitHub repo to get a new URL.
-->

1. Encabezado `x-gw-ims-org-id` con ID de organización de IMS.

1. `x-api-key` con el ID de cliente del proyecto [!DNL Adobe Developers Console].

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

Estos ámbitos requieren que el proyecto [!DNL Adobe Developer Console] se suscriba a los servicios `Asset Compute`, `I/O Events` y `I/O Management API`. El desglose de ámbitos individuales es el siguiente:

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

Cada cliente de [!DNL Asset Compute service] (un proyecto único de [!DNL Adobe Developer Console] suscrito al servicio) debe [registrarse](#register-request) antes de realizar solicitudes de procesamiento. El paso de registro devuelve el diario de eventos único necesario para recuperar los eventos asincrónicos del procesamiento de la representación.

Al final de su ciclo de vida, un cliente puede [anular el registro](#unregister-request).

### Solicitud de registro {#register-request}

Esta llamada de API configura un cliente [!DNL Asset Compute] y proporciona la dirección URL del diario de eventos. Este proceso es una operación idempotente y solo necesita llamarse una vez para cada cliente. Se puede volver a llamar para recuperar la dirección URL del historial.

| Parámetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Ruta | `/register` |
| Encabezado `Authorization` | Todos los [encabezados relacionados con la autorización](#authentication-and-authorization). |
| Encabezado `x-request-id` | Opcional, establecida por los clientes para un identificador de extremo a extremo único de las solicitudes de procesamiento entre sistemas. |
| Cuerpo de solicitud | Debe estar vacío. |

### Registrar respuesta {#register-response}

| Parámetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Encabezado `X-Request-Id` | Igual que el encabezado de solicitud `X-Request-Id` o uno generado de forma única. Se utiliza para identificar solicitudes entre sistemas, solicitudes de soporte o ambas cosas. |
| Cuerpo de respuesta | Un objeto JSON con `journal`, `ok` o `requestId` campos. |

Los códigos de estado HTTP son:

* **200 con éxito**: cuando la solicitud se realiza correctamente. La dirección URL `journal` recibe notificaciones sobre los resultados del procesamiento asincrónico iniciado mediante `/process`. Advierte de `rendition_created` eventos al completarse correctamente, o de `rendition_failed` eventos si el proceso falla.

  ```json
  {
      "ok": true,
      "journal": "https://api.adobe.io/events/organizations/xxxxx/integrations/xxxx/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
      "requestId": "1234567890"
  }
  ```

* **401 No autorizado**: se produce cuando la solicitud no tiene una [autenticación](#authentication-and-authorization) válida. Un ejemplo puede ser un token de acceso no válido o una clave de API no válida.

* **403 Prohibido**: se produce cuando la solicitud no tiene una [autorización](#authentication-and-authorization) válida. Un ejemplo puede ser un token de acceso válido, pero el proyecto de Adobe Developer Console (cuenta técnica) no está suscrito a todos los servicios necesarios.

* **429 Demasiadas solicitudes**: se produce cuando este cliente o de otro modo sobrecarga el sistema. Los clientes deben volver a intentarlo con un [retroceso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). El cuerpo está vacío.
* **Error de 4xx**: cuando hubo cualquier otro error de cliente y el registro falló. Normalmente se devuelve una respuesta JSON como esta, aunque no está garantizado para todos los errores:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Error 5xx**: se produce cuando se produjo cualquier otro error del lado del servidor y se produjo un error de registro. Normalmente se devuelve una respuesta JSON como esta, aunque no está garantizado para todos los errores:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

### Solicitud de cancelación de registro {#unregister-request}

Esta llamada de API anula el registro de un cliente [!DNL Asset Compute]. Una vez cancelado el registro, ya no es posible llamar a `/process`. El uso de la llamada de API para un cliente no registrado o un cliente aún no registrado devuelve un error `404`.

| Parámetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Ruta | `/unregister` |
| Encabezado `Authorization` | Todos los [encabezados relacionados con la autorización](#authentication-and-authorization). |
| Encabezado `x-request-id` | Opcional. Los clientes pueden configurarlo para obtener un identificador completo único de las solicitudes de procesamiento entre sistemas. |
| Cuerpo de solicitud | Vacío. |

### Anular registro de respuesta {#unregister-response}

| Parámetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Encabezado `X-Request-Id` | Igual que el encabezado de solicitud `X-Request-Id` o uno generado de forma única. Se utiliza para identificar solicitudes entre sistemas o solicitudes de asistencia. |
| Cuerpo de respuesta | Un objeto JSON con `ok` y `requestId` campos. |

Los códigos de estado son:

* **200 con éxito**: se produce cuando se encuentra y se quita el registro y el diario.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **401 No autorizado**: se produce cuando la solicitud no tiene una [autenticación](#authentication-and-authorization) válida. Un ejemplo puede ser un token de acceso no válido o una clave de API no válida.

* **403 Prohibido**: se produce cuando la solicitud no tiene una [autorización](#authentication-and-authorization) válida. Un ejemplo puede ser un token de acceso válido, pero el proyecto de Adobe Developer Console (cuenta técnica) no está suscrito a todos los servicios necesarios.

* **404 No encontrado**: Este estado aparece cuando las credenciales proporcionadas no están registradas o no son válidas.

  ```json
  {
      "ok": true,
      "requestId": "1234567890"
  }
  ```

* **429 Demasiadas solicitudes**: se produce cuando el sistema está sobrecargado. Los clientes deben volver a intentarlo con un [retroceso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). El cuerpo está vacío.

* **Error de 4xx**: se produce cuando se produjo cualquier otro error de cliente y se produjo un error al anular el registro. Normalmente se devuelve una respuesta JSON como esta, aunque no está garantizado para todos los errores:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Error 5xx**: se produce cuando se produjo cualquier otro error del lado del servidor y se produjo un error de registro. Normalmente se devuelve una respuesta JSON como esta, aunque no está garantizado para todos los errores:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

## Procesar solicitud {#process-request}

La operación `process` envía un trabajo que transforma un recurso de origen en varias representaciones, según las instrucciones de la solicitud. Las notificaciones sobre la finalización correcta (tipo de evento `rendition_created`) o cualquier error (tipo de evento `rendition_failed`) se envían a un diario de eventos que debe recuperarse mediante [`/register`](#register) una vez antes de realizar cualquier número de `/process` solicitudes. Las solicitudes formadas incorrectamente generan un error 400.

Se hace referencia a los binarios mediante direcciones URL, como URL prefirmadas de Amazon AWS S3 o URL de SAS de almacenamiento de Azure Blob. Se usa tanto para leer el recurso `source` (`GET` direcciones URL) como para escribir las representaciones (`PUT` direcciones URL). El cliente es responsable de generar estas direcciones URL prefirmadas.

| Parámetro | Valor |
|--------------------------|------------------------------------------------------|
| Método | `POST` |
| Ruta | `/process` |
| Tipo MIME | `application/json` |
| Encabezado `Authorization` | Todos los [encabezados relacionados con la autorización](#authentication-and-authorization). |
| Encabezado `x-request-id` | Opcional. Los clientes pueden establecer un identificador de extremo a extremo único para realizar el seguimiento de las solicitudes de procesamiento entre sistemas. |
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
| `source` | `object` | Descripción del recurso de origen que se procesa. Vea la descripción de [campos de objeto de Source](#source-object-fields) a continuación. Opcional en función del formato de representación solicitado (por ejemplo, `fmt=zip`). | `{"url": "http://example.com/image.jpg", "mimeType": "image/jpeg" }` |
| `renditions` | `array` | Representaciones que se generarán a partir del archivo de origen. Cada objeto de representación admite [instrucción de representación](#rendition-instructions). Requerido. | `[{ "target": "https://....", "fmt": "png" }]` |

`source` puede ser un `<string>` que se vea como una dirección URL o puede ser un `<object>` con un campo adicional. Las siguientes variantes son similares:

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
| `name` | `string` | Nombre del archivo del recurso Source. Se puede usar una extensión de archivo en el nombre si no se detecta ningún tipo MIME. Tiene prioridad sobre el nombre de archivo especificado en la ruta de acceso URL. Además, tiene prioridad sobre el nombre de archivo en el encabezado `content-disposition` del recurso binario. El valor predeterminado es &quot;archivo&quot;. | `"image.jpg"` |
| `size` | `number` | Tamaño del archivo del recurso de Source en bytes. Tiene prioridad sobre el encabezado `content-length` del recurso binario. | `10234` |
| `mimetype` | `string` | Tipo MIME del archivo del recurso de Source. Tiene prioridad sobre el encabezado `content-type` del recurso binario. | `"image/jpeg"` |

### Un ejemplo de solicitud `process` completa {#complete-process-request-example}

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

La solicitud `/process` se devuelve inmediatamente con un resultado correcto o erróneo según la validación básica de la solicitud. El procesamiento real de recursos se produce de forma asíncrona.

| Parámetro | Valor |
|-----------------------|------------------------------------------------------|
| Tipo MIME | `application/json` |
| Encabezado `X-Request-Id` | Igual que el encabezado de solicitud `X-Request-Id` o uno generado de forma única. Se utiliza para identificar solicitudes entre sistemas o solicitudes de asistencia. |
| Cuerpo de respuesta | Un objeto JSON con `ok` y `requestId` campos. |

Códigos de estado:

* **200 con éxito**: si la solicitud se envió correctamente. El JSON de respuesta incluye `"ok": true`:

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

* **401 No autorizado**: Cuando la solicitud no tiene una [autenticación](#authentication-and-authorization) válida. Un ejemplo puede ser un token de acceso no válido o una clave de API no válida.
* **403 Prohibido**: Cuando la solicitud no tiene una [autorización](#authentication-and-authorization) válida. Un ejemplo puede ser un token de acceso válido, pero el proyecto de Adobe Developer Console (cuenta técnica) no está suscrito a todos los servicios necesarios.
* **429 Demasiadas solicitudes**: Se produce cuando el sistema está sobrecargado, ya sea debido a este cliente en particular o a una demanda general. Los clientes pueden volver a intentarlo con un [retroceso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff). El cuerpo está vacío.
* **Error de 4xx**: cuando había algún otro error de cliente. Normalmente se devuelve una respuesta JSON como esta, aunque no está garantizado para todos los errores:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

* **Error 5xx**: cuando había algún otro error del lado del servidor. Normalmente se devuelve una respuesta JSON como esta, aunque no está garantizado para todos los errores:

  ```json
  {
      "ok": false,
      "requestId": "1234567890",
      "message": "error message"
  }
  ```

Es probable que la mayoría de los clientes prefieran reintentar la misma solicitud con [retroceso exponencial](https://en.wikipedia.org/wiki/Exponential_backoff) en cualquier error *excepto* problemas de configuración como 401 o 403, o solicitudes no válidas como 400. Aparte de la limitación regular de la velocidad mediante respuestas 429, una interrupción temporal del servicio o una limitación podría provocar errores 5xx. Entonces, sería aconsejable volver a intentarlo después de un periodo de tiempo.

Todas las respuestas JSON (si están presentes) incluyen `requestId`, que es el mismo valor que el encabezado `X-Request-Id`. El Adobe recomienda leer del encabezado porque siempre está presente. El `requestId` también se devuelve en todos los eventos relacionados con el procesamiento de solicitudes como `requestId`. Los clientes no deben suponer el formato de esta cadena. Es un identificador de cadena opaco.

## Inclusión en el posprocesamiento {#opt-in-to-post-processing}

El [SDK de Asset compute](https://github.com/adobe/asset-compute-sdk) admite un conjunto de opciones básicas de procesamiento posterior de imágenes. Los trabajadores personalizados pueden incluirse explícitamente en el posprocesamiento estableciendo el campo `postProcess` en el objeto de representación en `true`.

Los casos de uso admitidos son:

* Recortar es una representación en un rectángulo cuyos límites están definidos por crop.w, crop.h, crop.x y crop.y. Los detalles de recorte se especifican en el campo `instructions.crop` del objeto de representación.
* Cambie el tamaño de las imágenes mediante los valores de anchura, altura o ambos. `instructions.width` y `instructions.height` lo definen en el objeto de representación. Para cambiar el tamaño sólo con anchura o altura, defina sólo un valor. Compute Service conserva la relación de aspecto.
* Establezca la calidad de una imagen JPEG. `instructions.quality` lo define en el objeto de representación. Un nivel de calidad de 100 representa la calidad más alta, mientras que números más bajos significan una disminución en la calidad.
* Cree imágenes entrelazadas. `instructions.interlace` lo define en el objeto de representación.
* Establezca PPP para ajustar el tamaño procesado para la publicación en el escritorio ajustando la escala aplicada a los píxeles. El `instructions.dpi` lo define en el objeto de representación para cambiar la resolución de los ppp. Sin embargo, para cambiar el tamaño de la imagen de modo que tenga el mismo tamaño con una resolución diferente, siga las instrucciones de `convertToDpi`.
* Cambie el tamaño de la imagen de modo que la anchura o altura procesadas sean las mismas que las originales con la resolución de destino especificada (PPP). `instructions.convertToDpi` lo define en el objeto de representación.

## Recursos de marca de agua {#add-watermark}

El [SDK de Asset compute](https://github.com/adobe/asset-compute-sdk) admite la adición de una marca de agua a los archivos de imagen PNG, JPEG, TIFF y GIF. La marca de agua se agrega siguiendo las instrucciones de representación del objeto `watermark` de la representación.

La marca de agua se realiza durante el procesamiento posterior de la representación. Para marcar los recursos, el trabajador personalizado [opta por el posprocesamiento](#opt-in-to-post-processing) al establecer el campo `postProcess` en el objeto de representación en `true`. Si el trabajador no selecciona, la marca de agua no se aplica, incluso si el objeto de marca de agua está establecido en el objeto de representación de la solicitud.

## Instrucciones de representación {#rendition-instructions}

Las siguientes son las opciones disponibles para la matriz `renditions` en [`/process`](#process-request).

### Campos comunes {#common-fields}

| Nombre | Tipo | Descripción | Ejemplos |
|-------------------|----------|-------------|---------|
| `fmt` | `string` | XMP El formato de destino de las representaciones también puede ser `text` para la extracción de texto y `xmp` para la extracción de metadatos de como xml. Ver [formatos compatibles](https://experienceleague.adobe.com/es/docs/experience-manager-cloud-service/content/assets/file-format-support) | `png` |
| `worker` | `string` | URL de [aplicación personalizada](develop-custom-application.md). Debe ser una dirección URL `https://`. Si este campo está presente, una aplicación personalizada crea la representación. A continuación, se utiliza cualquier otro campo de representación definido en la aplicación personalizada. | `"https://1234.adobeioruntime.net`<br>`/api/v1/web`<br>`/example-custom-worker-master/worker"` |
| `target` | `string` | Dirección URL a la que se debe cargar la representación generada mediante el PUT HTTP. | `http://w.com/img.jpg` |
| `target` | `object` | Información de carga de URL firmada previamente de varias partes para la representación generada. AEM Esta información es para [/ Carga binaria directa de Oak](https://jackrabbit.apache.org/oak/docs/features/direct-binary-access.html) con este [comportamiento de carga multiparte](https://jackrabbit.apache.org/oak/docs/apidocs/org/apache/jackrabbit/api/binary/BinaryUpload.html).<br>Campos:<ul><li>`urls`: matriz de cadenas, una para cada URL de parte firmada previamente</li><li>`minPartSize`: el tamaño mínimo que se va a usar para una parte = url</li><li>`maxPartSize`: el tamaño máximo que se va a usar para una parte = url</li></ul> | `{ "urls": [ "https://part1...", "https://part2..." ], "minPartSize": 10000, "maxPartSize": 100000 }` |
| `userData` | `object` | Opcional. El cliente controla el espacio reservado y lo pasa tal cual a los eventos de representación. Permite que un cliente agregue información personalizada para identificar eventos de representación. No se debe modificar ni confiar en él en las aplicaciones personalizadas, ya que los clientes pueden cambiarlo en cualquier momento. | `{ ... }` |

### Campos específicos de representación {#rendition-specific-fields}

Para obtener una lista de los formatos de archivo admitidos actualmente, consulte [formatos de archivo admitidos](https://experienceleague.adobe.com/es/docs/experience-manager-cloud-service/content/assets/file-format-support).

| Nombre | Tipo | Descripción | Ejemplos |
|-------------------|----------|-------------|---------|
| `*` | `*` | Se pueden agregar campos personalizados avanzados que una [aplicación personalizada](develop-custom-application.md) comprenda. | |
| `embedBinaryLimit` | `number` en bytes | Cuando el tamaño de archivo de la representación es menor que el valor especificado, se incluye en el evento enviado una vez finalizada su creación. El tamaño máximo permitido para la incrustación es de 32 KB (32 x 1024 bytes). Si el tamaño de una representación es mayor que el límite de `embedBinaryLimit`, se coloca en una ubicación del almacenamiento en la nube y no está incrustada en el evento. | `3276` |
| `width` | `number` | Anchura en píxeles. solo para representaciones de imágenes. | `200` |
| `height` | `number` | Altura en píxeles. solo para representaciones de imágenes. | `200` |
|                   |          | La relación de aspecto se mantiene siempre si: <ul> <li> Se especificaron `width` y `height`, y la imagen se ajustará al tamaño sin perder la relación de aspecto </li><li> Si solo se especifica `width` o `height`, la imagen resultante utilizará la dimensión correspondiente y mantendrá la relación de aspecto</li><li> Si no se especifica `width` o `height`, se utilizará el tamaño de píxel de la imagen original. Depende del tipo de origen. En algunos formatos, como los archivos de PDF, se utiliza un tamaño predeterminado. Puede haber un límite de tamaño máximo.</li></ul> | |
| `quality` | `number` | Especifique la calidad JPEG en el intervalo de `1` a `100`. Aplicable solo para representaciones de imágenes. | `90` |
| `xmp` | `string` | XMP XMP Solo se utiliza para la reescritura de metadatos de la, se codifica en base64 para volver a escribir en la representación especificada. | |
| `interlace` | `bool` | Cree PNG entrelazado, GIF o JPEG progresivo al establecerlo en `true`. No afecta a otros formatos de archivo. | |
| `jpegSize` | `number` | Tamaño aproximado del archivo del JPEG en bytes. Anula cualquier configuración de `quality`. No afecta a otros formatos. | |
| `dpi` | `number` o `object` | Establezca los ppp x e y. Para simplificar, también se puede configurar en un solo número, que se utiliza tanto para x como para y. No tiene ningún efecto en la propia imagen. | `96` o `{ xdpi: 96, ydpi: 96 }` |
| `convertToDpi` | `number` o `object` | Los ppp x e y vuelven a muestrear los valores manteniendo el tamaño físico. Para simplificar, también se puede configurar en un solo número, que se utiliza tanto para x como para y. | `96` o `{ xdpi: 96, ydpi: 96 }` |
| `files` | `array` | Lista de archivos para incluir en el archivo ZIP (`fmt=zip`). Cada entrada puede ser una cadena URL o un objeto con los campos:<ul><li>`url`: URL para descargar el archivo</li><li>`path`: almacena el archivo en esta ruta de acceso en el ZIP</li></ul> | `[{ "url": "https://host/asset.jpg", "path": "folder/location/asset.jpg" }]` |
| `duplicate` | `string` | Administración de duplicados para los archivos ZIP (`fmt=zip`). De forma predeterminada, varios archivos almacenados en la misma ruta en el ZIP generan un error. Si se establece `duplicate` en `ignore`, solo se almacenará el primer recurso y se omitirá el resto. | `ignore` |
| `watermark` | `object` | Contiene instrucciones acerca de [watermark](#watermark-specific-fields). |  |

### Campos específicos de marcas de agua {#watermark-specific-fields}

El formato PNG se utiliza como marca de agua.

| Nombre | Tipo | Descripción | Ejemplos |
|-------------------|----------|-------------|---------|
| `scale` | `number` | Escala de la marca de agua, entre `0.0` y `1.0`. `1.0` significa que la marca de agua tiene su escala original (1:1) y los valores inferiores reducen el tamaño de la marca de agua. | Un valor de `0.5` significa la mitad del tamaño original. |
| `image` | `url` | URL del archivo PNG que se utilizará para la marca de agua. | |

## Eventos asíncronos {#asynchronous-events}

Cuando finaliza el procesamiento de una representación o se produce un error, se envía un evento a un Adobe [!DNL `I/O Events Journal`]. Los clientes deben escuchar la dirección URL del diario proporcionada a través de [`/register`](#register). La respuesta del diario incluye una matriz `event` que consta de un objeto para cada evento, de los cuales el campo `event` incluye la carga útil del evento real.

El tipo de Adobe [!DNL `I/O Events`] para todos los eventos de [!DNL Asset Compute Service] es `asset_compute`. El historial solo se suscribe automáticamente a este tipo de evento y no se requiere ningún otro filtro basado en el tipo de evento [!DNL Adobe Developer]. Los tipos de eventos específicos del servicio están disponibles en la propiedad `type` del evento.

### Tipos de eventos {#event-types}

| Evento | Descripción |
|---------------------|-------------|
| `rendition_created` | Se envía por cada representación cargada y procesada correctamente. |
| `rendition_failed` | Se envía por cada representación que no se procesa ni se carga. |

### Atributos de evento {#event-attributes}

| Atributo | Tipo | Evento | Descripción |
|-------------|----------|---------------|-------------|
| `date` | `string` | `*` | Marca de tiempo de cuando el evento se envió en formato extendido simplificado [ISO-8601](https://en.wikipedia.org/wiki/ISO_8601), tal como se define en JavaScript [Date.toISOString()](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Date/toISOString). |
| `requestId` | `string` | `*` | El id. de solicitud de la solicitud original de `/process`, igual que el encabezado de `X-Request-Id`. |
| `source` | `object` | `*` | El `source` de la solicitud `/process`. |
| `userData` | `object` | `*` | El `userData` de la representación de la solicitud `/process` si está establecido. |
| `rendition` | `object` | `rendition_*` | Se pasó el objeto de representación correspondiente en `/process`. |
| `metadata` | `object` | `rendition_created` | Las propiedades de [metadata](#metadata) de la representación. |
| `errorReason` | `string` | `rendition_failed` | Error de representación [reason](#error-reasons), si lo hay. |
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
| `RenditionTooLarge` | No se pudo cargar la representación mediante las direcciones URL firmadas previamente proporcionadas en `target`. El tamaño real de la representación está disponible como metadatos en `repo:size` y el cliente lo usa para volver a procesar esta representación con el número correcto de direcciones URL firmadas previamente. |
| `GenericError` | Cualquier otro error inesperado. |
