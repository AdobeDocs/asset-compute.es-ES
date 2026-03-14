---
title: Solucionar problemas de  [!DNL Asset Compute Service]
description: Solucionar problemas y depurar aplicaciones personalizadas con  [!DNL Asset Compute Service].
exl-id: 017fff91-e5e9-4a30-babf-5faa1ebefc2f
source-git-commit: aed361a577fc53caec4118e417b1c0c814617b51
workflow-type: tm+mt
source-wordcount: '293'
ht-degree: 0%

---

# Solución de problemas {#troubleshoot}

Algunas sugerencias genéricas de solución de problemas que pueden ayudarle a solucionar problemas con el servicio de Asset compute son:

* Asegúrese de que la aplicación JavaScript no se bloquee al iniciarse. Estos bloqueos suelen estar relacionados con una biblioteca que falta o una dependencia.
* Asegúrese de que todas las dependencias que se vayan a instalar estén referenciadas en el archivo `package.json` de la aplicación.
* Asegúrese de que los errores que puedan surgir de la limpieza en caso de error no generen sus propios errores que oculten el problema original.

* Al iniciar la herramienta de programador por primera vez con una nueva integración de [!DNL Asset Compute Service], se puede producir un error en la primera solicitud de procesamiento si el Diario de eventos de Asset compute no está configurado completamente. Espere a que el diario se configure antes de enviar otra solicitud.
* Asegúrese de que todas las Assets computes de API necesarias, el Adobe [!DNL I/O Events], la administración de eventos y el motor en tiempo de ejecución estén incluidos en el Adobe [!DNL `I/O Project`] y el área de trabajo para evitar errores de solicitud `/register` o `/process`.

## Problemas de inicio de sesión mediante el Adobe [!DNL aio-cli] {#login-via-aio-cli}

Si tiene problemas para iniciar sesión en [!DNL Adobe Developer Console] [ a través del Adobe  [!DNL aio-cli]](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#3-signing-in-from-cli), agregue manualmente las credenciales necesarias para desarrollar, probar e implementar la aplicación personalizada:

1. Desplácese al proyecto y espacio de trabajo de Adobe Developer App Builder en [Adobe Developer Console](https://developer.adobe.com/console/user/servicesandapis) y presione **[!UICONTROL Descargar]** en la esquina superior derecha. Abra este archivo y guarde este JSON en un lugar seguro del equipo.

1. Acceda al archivo ENV en la aplicación Adobe Developer App Builder.

1. Agregue las credenciales de Adobe [!DNL I/O Runtime]. Obtenga las credenciales de Adobe [!DNL I/O Runtime] del JSON descargado. Las credenciales están por debajo de `project.workspace.services.runtime`. Agregue las credenciales de tiempo de ejecución [!DNL Adobe I/O] en las variables `AIO_runtime_XXX`:

   ```json
   AIO_runtime_auth=
   AIO_runtime_namespace=
   ```

1. Añada la ruta absoluta al JSON descargado en el paso 1:

   ```json
       ASSET_COMPUTE_INTEGRATION_FILE_PATH=
   ```

1. Configure el resto de las [credenciales requeridas](develop-custom-application.md) necesarias para la herramienta de desarrollo.

<!-- 
TBD for later:
Add any best practices for developers in this section:
* Any items to take care of when creating projects.
* Any naming conventions, reserved keywords, etc.?
* Any terms that can become a source of confusion later based on our OOTB naming.

* If required, add limitations for custom applications and spin those off as best practices.
* Do NOT borrow any content from https://git.corp.adobe.com/nui/nui/blob/master/doc/worker_api.md. It is outdated and irrelevant for 3rd party custom applications.
-->
