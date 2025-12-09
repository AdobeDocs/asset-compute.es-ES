---
title: Solucionar problemas [!DNL Asset Compute Service]
description: Solucionar problemas y depurar aplicaciones personalizadas usando  [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: 63f83ff33ac6cd090fac4f6db18000155f464643
workflow-type: tm+mt
source-wordcount: '273'
ht-degree: 0%

---

# Solución de problemas {#troubleshoot}

Algunas sugerencias genéricas para la resolución de problemas que pueden ayudarle a solucionar problemas con el servicio Asset Compute son:

* Asegúrese de que la aplicación de JavaScript no se bloquee durante el inicio. Estos bloqueos suelen estar relacionados con la falta de una biblioteca o con una dependencia.
* Asegúrese de que se hace referencia a todas las dependencias que se van a instalar en el archivo `package.json` de la aplicación.
* Asegúrese de que los errores que puedan provenir de la limpieza en caso de error no generen sus propios errores que oculten el problema original.

* Al iniciar la herramienta para desarrolladores por primera vez con una nueva integración de [!DNL Asset Compute Service], puede fallar la primera solicitud de procesamiento si el diario de eventos de Asset Compute no está completamente configurado. Espere un poco para que el historial se configure antes de enviar otra solicitud.
* Asegúrese de que todas las API necesarias (Asset Compute, Adobe [!DNL I/O Events], administración de eventos y tiempo de ejecución) estén incluidas en Adobe [!DNL `I/O Project`] y Workspace para evitar errores de solicitud de `/register` o `/process`.

## Iniciar sesión mediante Adobe [!DNL aio-cli] {#login-via-aio-cli}

Si tiene problemas para iniciar sesión en [!DNL Adobe Developer Console] [a través de Adobe [!DNL aio-cli]](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#3-signing-in-from-cli), agregue manualmente las credenciales necesarias para desarrollar, probar e implementar su aplicación personalizada:

1. Vaya a su proyecto de Adobe Developer App Builder y al área de trabajo en [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis) y presione **[!UICONTROL Descargar]** desde la esquina superior derecha. Abra este archivo y guarde este JSON en un lugar seguro de su equipo.

1. Navegue hasta el archivo ENV en la aplicación de Adobe Developer App Builder.

1. Agregar las credenciales de Adobe [!DNL I/O Runtime]. Obtenga las credenciales de Adobe [!DNL I/O Runtime] del JSON descargado. Las credenciales están en `project.workspace.services.runtime`. Agregar las credenciales de tiempo de ejecución [!DNL Adobe I/O] en las variables `AIO_runtime_XXX`:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Añada la ruta absoluta al JSON descargado en el paso 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configure el resto de [las credenciales necesarias](develop-custom-application.md) para la herramienta para desarrolladores.

<!-- TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
