---
title: Implementar [!DNL Asset Compute Service] aplicación personalizada
description: Implementar [!DNL Asset Compute Service] aplicación personalizada.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
source-git-commit: 5257e091730f3672c46dfbe45c3e697a6555e6b1
workflow-type: tm+mt
source-wordcount: '190'
ht-degree: 3%

---

# Implementación de una aplicación personalizada {#deploy-custom-application}

Para implementar la aplicación, utilice [implementación de aplicación aio](https://github.com/adobe/aio-cli#aio-appdeploy) comando. En el terminal, el comando muestra una URL para acceder a la aplicación personalizada. La dirección URL tiene el formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Para obtener la misma URL sin volver a implementar la aplicación, utilice [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action) comando.

Utilice la dirección URL en una [Perfil de procesamiento en [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/assets/manage/asset-microservices-configure-and-use.html?lang=es) para integrar su aplicación con [!DNL Experience Manager] as a [!DNL Cloud Service].

Asegúrese de que el proyecto del Generador de aplicaciones y el espacio de trabajo se corresponden con [!DNL Experience Manager] as a [!DNL Cloud Service] entorno en el que desea utilizar la acción. Tiene diferentes entornos para el desarrollo, el ensayo y la producción. Puede verificar el entorno marcando `AIO_runtime_*` credenciales definidas dentro del archivo ENV en la raíz de la aplicación Adobe Developer App Builder. Por ejemplo, para implementar en un `Stage` workspace, la `AIO_runtime_namespace` es del formato `xxxxxx_xxxxxxxxx_stage`. Para integrar con [!DNL Experience Manager] as a [!DNL Cloud Service] Entorno de producción, usar URL de aplicación del Generador de aplicaciones de Adobe Developer `Production` workspace.

>[!CAUTION]
>
>No utilice un espacio de trabajo personal en [!DNL Experience Manager] entornos.

>[!MORELIKETHIS]
>
>* [Explicación y administración de entornos en [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/docs/experience-manager-cloud-service/implementing/using-cloud-manager/manage-environments.html).
