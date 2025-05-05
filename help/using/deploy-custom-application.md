---
title: Implementar  [!DNL Asset Compute Service] aplicación personalizada
description: Implementar  [!DNL Asset Compute Service] aplicación personalizada.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '170'
ht-degree: 0%

---

# Implementación de una aplicación personalizada {#deploy-custom-application}

Para implementar su aplicación, use el comando [aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy). En el terminal, el comando muestra una URL para acceder a la aplicación personalizada. La dirección URL tiene el formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Para obtener la misma dirección URL sin volver a implementar la aplicación, use el comando [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action).

Use la dirección URL de un [perfil de procesamiento en [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/es/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use) para integrar su aplicación con [!DNL Experience Manager] as a [!DNL Cloud Service].

Asegúrese de que el proyecto de App Builder y el área de trabajo se correspondan con el entorno [!DNL Experience Manager] as a [!DNL Cloud Service] en el que desea usar la acción. Tiene diferentes entornos para el desarrollo, el ensayo y la producción. Puede comprobar el entorno comprobando las credenciales de `AIO_runtime_*` definidas dentro del archivo ENV en la raíz de la aplicación de Adobe Developer App Builder. Por ejemplo, para implementarlo en un área de trabajo `Stage`, `AIO_runtime_namespace` tiene el formato `xxxxxx_xxxxxxxxx_stage`. Para integrarse con [!DNL Experience Manager] como un entorno de producción de [!DNL Cloud Service], use las direcciones URL de la aplicación del área de trabajo de Adobe Developer App Builder `Production`.

>[!CAUTION]
>
>No utilice un área de trabajo personal en entornos [!DNL Experience Manager] críticos.

>[!MORELIKETHIS]
>
>* [Comprenda y administre entornos en [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/es/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-environments).
