---
title: Arquitectura de  [!DNL Asset Compute Service]
description: Cómo  [!DNL Asset Compute Service] API, aplicaciones y SDK trabajan juntos para proporcionar un servicio de procesamiento de recursos nativo de la nube.
exl-id: 658ee4b7-5eb1-4109-b263-1b7d705e49d6
source-git-commit: f199cecfe4409e2370b30783f984062196dd807d
workflow-type: tm+mt
source-wordcount: '478'
ht-degree: 0%

---

# Arquitectura de [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] se ha creado sobre la plataforma sin servidor Adobe [!DNL `I/O Runtime`]. Proporciona compatibilidad con servicios de contenido de Adobe Sensei para recursos. El cliente que realiza la invocación (solo se admite [!DNL Experience Manager] como [!DNL Cloud Service]) se proporciona con la información generada por Adobe Sensei que buscó para el recurso. La información devuelta está en formato JSON.

[!DNL Asset Compute Service] se puede ampliar creando aplicaciones personalizadas basadas en [!DNL Adobe Developer App Builder]. Estas aplicaciones personalizadas son [!DNL Project Adobe Developer App Builder] aplicaciones sin encabezado y realizan tareas como agregar herramientas de conversión personalizadas o llamar a API externas para realizar operaciones de imagen.

[!DNL Project Adobe Developer App Builder] es un marco para generar e implementar aplicaciones web personalizadas en Adobe [!DNL `I/O Runtime`]. Para crear aplicaciones personalizadas, los desarrolladores pueden aprovechar [!DNL React Spectrum] (el kit de herramientas de la interfaz de usuario de Adobe), crear microservicios, crear eventos personalizados y organizar API. Ver [documentación de Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/intro_and_overview/#).

La base sobre la que se basa la arquitectura incluye lo siguiente:

* La modularidad de las aplicaciones (que solo contienen lo necesario para una tarea determinada) permite desacoplar las aplicaciones entre sí y mantenerlas ligeras.

* El concepto sin servidor de [!DNL Adobe I/O] Runtime ofrece numerosas ventajas: procesamiento asincrónico, altamente escalable, aislado y basado en trabajos, que es perfecto para el procesamiento de recursos.

* El almacenamiento en la nube binaria proporciona las funciones necesarias para almacenar y acceder a los archivos y representaciones de recursos de forma individual, sin requerir permisos de acceso completos al almacenamiento, mediante referencias de URL firmadas previamente. La aceleración de la transferencia, el almacenamiento en caché de CDN y la co-ubicación de aplicaciones informáticas con almacenamiento en la nube permiten un acceso óptimo a contenido de baja latencia. Se admiten las nubes de AWS y Azure.

![Arquitectura del servicio Asset Compute](assets/architecture-diagram.png)

*Figura: Arquitectura de [!DNL Asset Compute Service] y cómo se integra con [!DNL Experience Manager], almacenamiento y aplicación de procesamiento.*

La arquitectura consta de las siguientes partes:

* **Una API y una capa de orquestación** reciben solicitudes (en formato JSON) que indican al servicio que transforme un recurso de origen en varias representaciones. Las solicitudes son asíncronas y se devuelven con un ID de activación que es el ID del trabajo. Las instrucciones son puramente declarativas y, para todo el trabajo de procesamiento estándar (por ejemplo, la generación de miniaturas, la extracción de texto), los consumidores solo especifican el resultado deseado, pero no las aplicaciones que administran determinadas representaciones. Las características genéricas de API, como autenticación, análisis y limitación de velocidad, se administran mediante la puerta de enlace de API de Adobe delante del servicio y administran todas las solicitudes que se dirigen al tiempo de ejecución de [!DNL Adobe I/O]. El enrutamiento de la aplicación se realiza dinámicamente mediante la capa de orquestación. Los clientes definen aplicaciones personalizadas para representaciones particulares, que vienen con su propio conjunto de parámetros únicos. La ejecución de la aplicación se puede paralelizar completamente porque son funciones independientes sin servidor en Adobe [!DNL `I/O Runtime`].

* **Aplicaciones para procesar recursos** que se especializan en ciertos tipos de formatos de archivo o representaciones de destino. Conceptualmente, una aplicación es como el concepto de canalización UNIX®: un archivo de entrada se transforma en uno o más archivos de salida.

* **Una [biblioteca de aplicaciones comunes](https://github.com/adobe/asset-compute-sdk)** administra tareas comunes. Por ejemplo, descargar el archivo de origen, cargar las representaciones, crear informes de errores, enviar eventos y supervisar. Este diseño garantiza que el desarrollo de aplicaciones se mantenga sencillo, respetando el concepto sin servidor, con interacciones limitadas al sistema de archivos local.

<!-- TBD:

* About the YAML file?
* minimize description to custom applications
* remove all internal stuff (e.g. Photoshop application, API Gateway) from text and diagram
* update diagram to focus on 3rd party custom applications ONLY
* Explain important transactions/handshakes?
* Flow of assets/control? See the illustration on the Nui diagrams wiki.
* Illustrations. See the SVG shared by Alex.
* Exceptions? Limitations? Call-outs? Gotchas?
* Do we want to add what basic processing is not available currently, that is expected by existing AEM customers?
-->
