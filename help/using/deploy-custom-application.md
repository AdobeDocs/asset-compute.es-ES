---
title: Implementar [!DNL Asset Compute Service] aplicación personalizada
description: Implementar [!DNL Asset Compute Service] aplicación personalizada.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: f15b9819d3319d22deccdf7e39c0f72728baaa39
workflow-type: tm+mt
source-wordcount: '170'
ht-degree: 0%

---

# Implementación de una aplicación personalizada {#deploy-custom-application}

Para implementar la aplicación, utilice [implementación de aplicación aio](https://github.com/adobe/aio-cli#aio-appdeploy) comando. En el terminal, el comando muestra una URL para acceder a la aplicación personalizada. La dirección URL tiene el formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Para obtener la misma URL sin volver a implementar la aplicación, utilice [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) comando.

Utilice la dirección URL en una [Perfil de procesamiento en [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use) para integrar su aplicación con [!DNL Experience Manager] as a [!DNL Cloud Service].

Asegúrese de que el proyecto de App Builder y el espacio de trabajo se corresponden con el [!DNL Experience Manager] as a [!DNL Cloud Service] entorno en el que desea utilizar la acción. Tiene diferentes entornos para el desarrollo, el ensayo y la producción. Puede verificar el entorno marcando `AIO_runtime_*` credenciales definidas dentro del archivo ENV en la raíz de la aplicación de Adobe Developer App Builder. Por ejemplo, para implementar en un `Stage` workspace, la `AIO_runtime_namespace` es del formato `xxxxxx_xxxxxxxxx_stage`. Para integrar con [!DNL Experience Manager] as a [!DNL Cloud Service] Entorno de producción, utilice las URL de aplicación de su Adobe Developer App Builder `Production` workspace.

>[!CAUTION]
>
>No utilice un espacio de trabajo personal en tareas críticas [!DNL Experience Manager] entornos.

>[!MORELIKETHIS]
>
>* [Explicación y administración de entornos en [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-environments).
