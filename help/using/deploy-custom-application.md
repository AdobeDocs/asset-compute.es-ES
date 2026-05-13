---
title: Implementar  [!DNL Asset Compute Service] aplicación personalizada
description: Implementar  [!DNL Asset Compute Service] aplicación personalizada.
exl-id: a68d4f59-8a8f-43b2-8bc6-19320ac8c9ef
TQID: https://experienceleague.adobe.com/JN29pTaNB93DKALUqIbXhwswlzHiYQSowFZAbgHA5TA
product_v2:
  - id: d09181b5-a36a-43de-ba01-36641440bc43
  - id: fd1f54a9-f50c-467d-8956-cebbaf4f3eb8
role_v2:
  - id: ff6a42d2-313e-452e-93a6-792e4fad9ff8
source-git-commit: 2510f77fed8d0f0708e09f32d0b13a437d2ede4f
workflow-type: tm+mt
source-wordcount: 210
ht-degree: 7%

---

# Implementación de una aplicación personalizada {#deploy-custom-application}

Para implementar su aplicación, use el comando [aio app deploy](https://github.com/adobe/aio-cli#aio-appdeploy). En el terminal, el comando muestra una URL para acceder a la aplicación personalizada. La dirección URL tiene el formato `https://[namespace].adobeio-static.net/api/v1/web/[appname]-[appversion]/[workername]`.

Para obtener la misma dirección URL sin volver a implementar la aplicación, use el comando [`aio app get-url`](https://github.com/adobe/aio-cli#aio-app-get-url-action).

Use la dirección URL de un [perfil de procesamiento en [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use) para integrar su aplicación con [!DNL Experience Manager] as a [!DNL Cloud Service].

Asegúrese de que el proyecto de App Builder y el área de trabajo se correspondan con el entorno [!DNL Experience Manager] as a [!DNL Cloud Service] en el que desea usar la acción. Tiene diferentes entornos para el desarrollo, el ensayo y la producción. Puede comprobar el entorno comprobando las credenciales de `AIO_runtime_*` definidas dentro del archivo ENV en la raíz de la aplicación de Adobe Developer App Builder. Por ejemplo, para implementarlo en un área de trabajo `Stage`, `AIO_runtime_namespace` tiene el formato `xxxxxx_xxxxxxxxx_stage`. Para integrarse con [!DNL Experience Manager] como un entorno de producción de [!DNL Cloud Service], use las direcciones URL de la aplicación del área de trabajo de Adobe Developer App Builder `Production`.

>[!CAUTION]
>
>No utilice un área de trabajo personal en entornos [!DNL Experience Manager] críticos.

>[!MORELIKETHIS]
>
>* [Comprenda y administre entornos en [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/implementing/using-cloud-manager/manage-environments).
