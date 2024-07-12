---
title: Obtener información sobre la ampliación de  [!DNL Asset Compute Service]
description: Cuándo y cómo ampliar la funcionalidad  [!DNL Asset Compute Service]  para realizar el procesamiento de recursos personalizado.
exl-id: 3b903364-34cc-44d5-9a03-24a0102cf85d
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '229'
ht-degree: 1%

---

# Introducción a la extensibilidad {#introduction-to-extensibilty}

[Perfiles de procesamiento en [!DNL Experience Manager] as a [!DNL Cloud Service]](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/asset-microservices-overview) satisfacen muchos requisitos de representación, como la conversión a formatos y el cambio de tamaño de imágenes. Los requisitos empresariales más complejos pueden necesitar una solución personalizada que se adapte a las necesidades de una organización. [!DNL Asset Compute Service] se puede ampliar creando aplicaciones personalizadas a partir de los perfiles de procesamiento de [!DNL Experience Manager]. Estas aplicaciones personalizadas atienden a [casos de uso admitidos](https://experienceleague.adobe.com/en/docs/experience-manager-cloud-service/content/assets/manage/asset-microservices-configure-and-use).

>[!NOTE]
>
>[!DNL Asset Compute Service] solo está disponible para su uso con [!DNL Experience Manager] como [!DNL Cloud Service].

Las aplicaciones personalizadas son aplicaciones [Adobe Developer App Builder](https://github.com/AdobeDocs/app-builder) sin encabezado. La ampliación de [!DNL Asset Compute Service] con aplicaciones personalizadas se simplifica gracias al [SDK de Asset compute](https://github.com/adobe/asset-compute-sdk) y a las herramientas para desarrolladores de Adobe Developer App Builder. Estas herramientas permiten a los desarrolladores centrarse en la lógica empresarial. Crear aplicaciones personalizadas es tan sencillo como crear una acción de Adobe [!DNL I/O Runtime] sin servidor. Es una función JavaScript única de Node.js. El [ejemplo básico de aplicación personalizada](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) lo ilustra.

## Requisitos previos y requisitos de aprovisionamiento {#prerequisites-and-provisioning}

Asegúrese de cumplir los siguientes requisitos previos:

* Las herramientas de Adobe Developer App Builder están instaladas en el equipo.
* Una organización [!DNL Experience Cloud]. Para obtener más información, ve a [Iniciar tu Recorrido de App Builder](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials).
* La organización de experiencia debe tener [!DNL Experience Manager] como [!DNL Cloud Service] habilitado.
* La organización [!DNL Adobe Experience Cloud] es parte del programa de información para desarrolladores [!DNL Adobe Developer App Builder]. Vaya a [cómo solicitar acceso](https://developer.adobe.com/app-builder/docs/overview/getting_access).
* Asegúrese de tener una función de desarrollador o permisos de administrador en la organización para el desarrollador.
* Asegúrese de que el Adobe [[!DNL aio-cli]](https://github.com/adobe/aio-cli) esté instalado localmente.

<!-- TBD for later:

* What all accesses and licenses are required?
* What all permissions are required to create, debug, and deploy custom applications?
* How do developers get access and provision the required apps?
* What is repository management?
* Anything on security and data transfer?
* What about handling personal or sensitive information?
* Custom application SLA is dependent on SLAs of various services it depends on.
* Document how the devs can get to know the KPIs of their custom applications. The KPIs are dependent on the performance at Adobe's side, amongst other things.
-->
