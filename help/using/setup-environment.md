---
title: Establecer el entorno de desarrollo necesario para  [!DNL Asset Compute Service]
description: Configuración del entorno de desarrollo para  [!DNL Asset Compute Service] para empezar a crear y probar código personalizado.
exl-id: 91c12889-01d8-4757-9bdd-f73c491cd9d5
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '324'
ht-degree: 2%

---

# Configuración de un entorno de desarrollo {#create-dev-environment}

Para crear una configuración que le permita desarrollar para [!DNL Asset Compute Service], siga estos requisitos e instrucciones.

1. [Obtener acceso y credenciales](https://developer.adobe.com/app-builder/docs/getting_started/#acquire-access-and-credentials) para [!DNL Adobe Developer App Builder].

1. [Configure el entorno local](https://developer.adobe.com/app-builder/docs/getting_started/#local-environment-set-up) y las herramientas necesarias.

1. Algunas herramientas adicionales que le ayudan a empezar a desarrollar sin problemas son las siguientes:

   * [Git](https://git-scm.com/)
   * [Escritorio Docker](https://www.docker.com/get-started)
   * [NodeJS](https://nodejs.org) (v14 LTS, no se recomiendan versiones impares) y [NPM](https://www.npmjs.com). El usuario de OS X HomeBrew puede hacer `brew install node` para instalar ambos. De lo contrario, descárguelo de la [página de descarga de NodeJS](https://nodejs.org/en/)
   * Un IDE que es bueno para NodeJS, Adobe recomienda [Visual Studio Code (VS Code)](https://code.visualstudio.com) ya que es el IDE compatible para el depurador. Puede utilizar cualquier otro IDE como editor de código, pero aún no se admite el uso avanzado (por ejemplo, depurador)
   * Instalar el Adobe más reciente [[!DNL aio-cli]](https://github.com/adobe/aio-cli) (`aio`)
   <!-- - install using `npm install -g @adobe/aio-cli@7.1.0` -->

1. Asegúrese de cumplir los [requisitos previos](/help/using/understand-extensibility.md#prerequisites-and-provisioning)

<!--
>[!NOTE]
>
>For now, use [!DNL Adobe I/O] CLI v7.1.0 of and do not use [!DNL Adobe I/O] CLI v8.
-->

## Configuración de un proyecto de App Builder {#create-App-Builder-project}

1. Asegúrese de que haya un administrador del sistema o un rol de desarrollador en la organización [!DNL Experience Cloud]. Un administrador del sistema, en el [Admin Console](https://adminconsole.adobe.com/overview), configura este rol.

1. Inicie sesión en [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis). Asegúrese de que forma parte de la misma organización [!DNL Experience Cloud] que la integración [!DNL Experience Manager] como [!DNL Cloud Service]. Para obtener más información acerca de Adobe Developer Console, vaya a [Documentación de la consola](https://developer.adobe.com/developer-console/docs/guides/).

1. [Crear un proyecto de App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app/). Haga clic en **[!UICONTROL Crear nuevo proyecto]** > **[!UICONTROL Proyecto a partir de la plantilla]**. Seleccione App Builder. Crea un nuevo proyecto de App Builder con dos espacios de trabajo: `Production` y `Stage`. Agregue espacios de trabajo adicionales, por ejemplo `Development`, según sea necesario.

1. En el proyecto de App Builder, seleccione un espacio de trabajo y suscríbase a los servicios necesarios para la Asset compute. Haga clic en **Agregar al proyecto** > **API** y agregue los servicios `Asset Compute`, `IO Events` y `IO Events Management`. Al añadir la primera API, se solicita crear una clave privada. Guarde esta información en el equipo, ya que necesita esta clave para probar la aplicación personalizada con la herramienta para desarrolladores.

## Siguiente paso {#next-step}

Con el entorno configurado, está listo para [crear una aplicación personalizada](develop-custom-application.md).

<!-- More ideas:
 
* Any steps in the beginning that lead to gotchas later should be called out for caution? For example,
  * don't change some defaults initially
  * know risks when deviating from standard path
  * naming conventions to follow
  * Retrieve and format credentials (YAML file details)

TBD: When aio-cli v8 bugs are resolved, update the AIO CLI install command to remove v7.x reference and instruct users to use the latest version. See CQDOC-18346.

-->
