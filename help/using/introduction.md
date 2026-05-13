---
title: Introducción a  [!DNL Asset Compute Service]
description: '[!DNL Asset Compute Service] es un servicio de procesamiento de recursos nativo de la nube que reduce la complejidad y mejora la escalabilidad.'
exl-id: f8c89f65-5a94-44f3-aaac-4612ae291101
TQID: https://experienceleague.adobe.com/7B0ghzyLWIZFe5sthLaorFf8w4Az7FKW4ces27eo8I0
product_v2:
  - id: d09181b5-a36a-43de-ba01-36641440bc43
  - id: fd1f54a9-f50c-467d-8956-cebbaf4f3eb8
feature_v2:
  - id: a01bfd36-4ab8-4bf8-9dc0-5b45b890552e
  - id: da0dfbce-df02-4f8b-b32d-a4e3b1d05085
role_v2:
  - id: ff6a42d2-313e-452e-93a6-792e4fad9ff8
topic_v2:
  - id: a004cc84-67b9-4a33-a3a7-8ec7273ef4dc
source-git-commit: 2510f77fed8d0f0708e09f32d0b13a437d2ede4f
workflow-type: tm+mt
source-wordcount: 355
ht-degree: 12%

---

# Información general de [!DNL Asset Compute Service] {#overview}

[!DNL Asset Compute Service] es un servicio escalable y ampliable de [!DNL Adobe Experience Cloud] para procesar recursos digitales. Puede transformar los formatos de imagen, vídeo, documento y otros archivos en distintas representaciones, incluidas miniaturas, texto extraído y metadatos, y archivos.

Los desarrolladores pueden añadir aplicaciones de recursos personalizadas (también denominadas trabajadores personalizados) para tratar casos de uso personalizados. El servicio funciona en Adobe [!DNL I/O Runtime]. Se puede ampliar mediante [!DNL Adobe Developer App Builder] aplicaciones sin encabezado escritas en Node.js. Pueden realizar operaciones personalizadas, como llamar a API externas para realizar operaciones de imagen o aprovechar la compatibilidad con [!DNL Adobe Sensei].

[!DNL Adobe Developer App Builder] es un módulo para generar e implementar aplicaciones web personalizadas en Adobe [!DNL I/O Runtime] para ampliar las soluciones de Adobe Experience Cloud. Para crear aplicaciones personalizadas, los desarrolladores pueden aprovechar [!DNL React Spectrum] (el kit de herramientas de la interfaz de usuario de Adobe), crear microservicios, crear eventos personalizados y organizar API. Ver [documentación de Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/intro_and_overview/#).

>[!NOTE]
>
>Actualmente, [!DNL Asset Compute Service] solo se puede usar a través de [!DNL Experience Manager] como [!DNL Cloud Service]. Los administradores crean perfiles de procesamiento que pueden llamar a [!DNL Asset Compute Service] para pasar recursos para su procesamiento. Ver [usar microservicios de recursos y perfiles de procesamiento](https://experienceleague.adobe.com/es/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

## Casos de uso admitidos de [!DNL Asset Compute Service] {#possible-use-cases-benefits}

[!DNL Asset Compute Service] admite algunos casos de uso empresarial comunes, como el procesamiento básico de imágenes, las conversiones específicas de aplicaciones de Adobe y la creación de aplicaciones personalizadas que organizan requisitos empresariales complejos.

Puede usar el servicio web [!DNL Asset Compute] para generar miniaturas para diferentes tipos de archivo y representaciones de imágenes de alta calidad para los [formatos de archivo compatibles](https://experienceleague.adobe.com/es/docs/experience-manager-cloud-service/content/assets/file-format-support). Ver [casos de uso admitidos mediante la configuración personalizada](https://experienceleague.adobe.com/es/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>El servicio no proporciona almacenamiento de recursos. Los usuarios lo proporcionan y proporcionan referencias a ubicaciones de archivos de origen y representación en el almacenamiento en la nube.

<!-- 
TBD: Should this be mentioned in the docs?

|Asset Compute Service does not do this|Expectations from implementing client|
|---|---|
| Binary uploads or API-based asset ingestion. | Use other methods to ingest assets. |
| Store binaries or any persisted data across processing requests.| Each request is independent so treat it as a standalone request by sharing binary and processing instructions. |
| Store any configurations such as processing rules or settings for a user or an organization's account. | Add processing request to each request/instruction. |
| Direct event handling of asset creation events from storage systems and processing completed notifications, and errors. | Use [!DNL Adobe I/O] Events and other methods. |

-->

>[!MORELIKETHIS]
>
>* [Información general sobre el procesamiento de recursos con microservicios de recursos en [!DNL Adobe Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/es/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview).
>* [Documentación de Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/intro_and_overview/#).
>* [Formatos de archivo compatibles para el procesamiento](https://experienceleague.adobe.com/es/docs/experience-manager-cloud-service/content/assets/file-format-support).

<!-- 
**TBD:**
* Clarify the service can only be used within AEM as Cloud Service. The docs provided as context for custom application developers. Not to be used as a standalone service.
  ** and API as that plays a role in custom applications (accepting standard params, invoking Nui itself in the future, etc. (this is an outlook))

* link to aem as cloud service docs on asset ingestion and customization with processing profiles.
-->
