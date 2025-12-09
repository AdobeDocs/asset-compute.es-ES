---
title: Probar y depurar  [!DNL Asset Compute Service] aplicación personalizada
description: Probar y depurar  [!DNL Asset Compute Service] aplicación personalizada.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: f199cecfe4409e2370b30783f984062196dd807d
workflow-type: tm+mt
source-wordcount: '775'
ht-degree: 0%

---

# Prueba y depuración de una aplicación personalizada {#test-debug-custom-worker}

## Ejecutar pruebas unitarias para una aplicación personalizada {#test-custom-worker}

Instale [Docker Desktop](https://www.docker.com/get-started) en su equipo. Para probar un trabajador personalizado, ejecute el siguiente comando en la raíz de la aplicación:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Este comando ejecuta un marco de prueba unitario personalizado para las acciones de la aplicación Asset Compute en el proyecto como se describe a continuación. Se conectó a través de una configuración en el archivo `package.json`. También es posible tener pruebas unitarias de JavaScript como Jest. El `aio app test` ejecuta ambos.

El complemento [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) está incrustado como una dependencia de desarrollo en la aplicación de aplicación personalizada, de modo que no es necesario instalarlo en sistemas de compilación o prueba.

### Marco de prueba unitario de aplicación {#unit-test-framework}

El marco de pruebas unitarias de la aplicación de Asset Compute permite probar aplicaciones sin escribir código. Se basa en el principio del archivo de origen a representación de las aplicaciones. Se debe configurar una estructura de archivos y carpetas determinada para definir casos de prueba con archivos de origen de prueba, parámetros opcionales, representaciones esperadas y scripts de validación personalizados. De forma predeterminada, las representaciones se comparan para la igualdad de bytes. Además, los servicios HTTP externos se pueden burlar fácilmente mediante archivos JSON simples.

### Agregar pruebas {#add-tests}

Se esperan pruebas dentro de la carpeta `test` en el nivel raíz del proyecto. Los casos de prueba para cada aplicación deben estar en la ruta de acceso `test/asset-compute/<worker-name>`, con una carpeta para cada caso de prueba:

```yaml
action/
manifest.yml
package.json
...
test/
  asset-compute/
    <worker-name>/
        <testcase1>/
            file.jpg
            params.json
            rendition.png
        <testcase2>/
            file.jpg
            params.json
            rendition.gif
            validate
        <testcase3>/
            file.jpg
            params.json
            rendition.png
            mock-adobe.com.json
            mock-console.adobe.io.json
```

Eche un vistazo a [aplicaciones personalizadas de ejemplo](https://github.com/adobe/asset-compute-example-workers/) para ver algunos ejemplos. A continuación se muestra una referencia detallada.

### Resultado de prueba {#test-output}

El directorio `build` en la raíz de la aplicación de Adobe Developer App Builder contiene los resultados detallados de las pruebas y los registros de la aplicación personalizada. Estos detalles también se muestran en la salida del comando `aio app test`.

### Burlar servicios externos {#mock-external-services}

Puede simular llamadas de servicio externas dentro de sus acciones creando `mock-<HOST_NAME>.json` archivos para sus escenarios de prueba, con HOST_NAME como el host específico que desea imitar. Un ejemplo de caso de uso es una aplicación que realiza una llamada independiente a S3. La nueva estructura de prueba tendría este aspecto:

```json
test/
  asset-compute/
    <worker-name>/
      <testcase3>/
        file.jpg
        params.json
        rendition.png
        mock-<HOST_NAME1>.json
        mock-<HOST_NAME2>.json
```

El archivo de prueba es una respuesta http con formato JSON. Para obtener más información, consulte [esta documentación](https://www.mock-server.com/mock_server/creating_expectations.html). Si hay varios nombres de host para burlar, defina varios archivos `mock-<mocked-host>.json`. A continuación se muestra un ejemplo de archivo de simulación para `google.com` con el nombre `mock-google.com.json`:

```json
[{
    "httpRequest": {
        "path": "/images/hello.txt"
        "method": "GET"
    },
    "httpResponse": {
        "statusCode": 200,
        "body": {
          "message": "hello world!"
        }
    }
}]
```

El ejemplo `worker-animal-pictures` contiene un [archivo de prueba](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) para el servicio Wikimedia con el que interactúa.

#### Uso compartido de archivos entre casos de prueba {#share-files-across-test-cases}

Adobe recomienda utilizar enlaces simbólicos relativos si comparte scripts de `file.*`, `params.json` o `validate` en varias pruebas. Son compatibles con Git. Asegúrese de asignar un nombre único a los archivos compartidos, ya que es posible que tenga otros distintos. En el ejemplo siguiente, las pruebas mezclan y hacen coincidir algunos archivos compartidos, y los suyos propios:

```json
tests/
    file-one.jpg
    params-resize.json
    params-crop.json
    validate-image-compare

    jpg-png-resize/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-resize.json
        rendition.png
        validate    - symlink: ../validate-image-compare

    jpg2-png-crop/
        file.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate    - symlink: ../validate-image-compare

    jpg-gif-crop/
        file.jpg    - symlink: ../file-one.jpg
        params.json - symlink: ../params-crop.json
        rendition.gif
        validate
```

### Prueba de errores esperados {#test-unexpected-errors}

Los casos de pruebas de error no deben contener un archivo `rendition.*` esperado y deben definir el `errorReason` esperado dentro del archivo `params.json`.

>[!NOTE]
>
>Si un caso de prueba no contiene un archivo `rendition.*` esperado y no define el archivo `errorReason` esperado dentro del archivo `params.json`, se supone que es un caso de error con cualquier `errorReason`.

Estructura de caso de prueba de error:

```json
<error_test_case>/
    file.jpg
    params.json
```

Archivo de parámetros con motivo del error:

```javascript
{
    "errorReason": "SourceUnsupported",
    // ... other params
}
```

Ver una lista completa y una descripción de [razones de error de Asset Compute](https://github.com/adobe/asset-compute-commons#error-reasons).

## Depuración de una aplicación personalizada {#debug-custom-worker}

Los pasos siguientes muestran cómo se puede depurar una aplicación personalizada mediante Visual Studio Code. Permite ver registros en directo, puntos de interrupción de visitas y paso a paso por el código, así como la recarga en directo de los cambios del código local en cada activación.

`aio` automatiza muchos de estos pasos de forma predeterminada. Vaya a la sección Depuración de la aplicación en [Adobe Developer App Builder documentation](https://developer.adobe.com/app-builder/docs/get_started/app_builder_get_started/first-app#). Por ahora, los pasos siguientes incluyen una solución.

1. Instale el [wskdebug](https://github.com/apache/openwhisk-wskdebug) más reciente de GitHub y el [ngrok](https://www.npmjs.com/package/ngrok) opcional.

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Añada a la configuración de usuario el archivo JSON. Sigue utilizando el depurador de código antiguo de Visual Studio. El nuevo tiene [algunos problemas](https://github.com/apache/openwhisk-wskdebug/issues/74) con wskdebug: `"debug.javascript.usePreview": false`.
1. Cierre todas las instancias de aplicaciones abiertas mediante `aio app run`.
1. Implemente el código más reciente con `aio app deploy`.
1. Ejecute solamente la herramienta de desarrollo de Asset Compute con `aio asset-compute devtool`. Manténgalo abierto.
1. En el Editor de código de Visual Studio, agregue la siguiente configuración de depuración a su `launch.json`:

   ```json
   {
     "type": "node",
     "request": "launch",
     "name": "wskdebug worker",
     "runtimeExecutable": "wskdebug",
     "args": [
       "PROJECT-0.0.1/__secured_worker",           // Replace this with your ACTION NAME
       "${workspaceFolder}/actions/worker/index.js",
       "-l",
       "--ngrok"
     ],
     "localRoot": "${workspaceFolder}",
     "remoteRoot": "/code",
     "outputCapture": "std",
     "timeout": 30000
   }
   ```

   Recuperar `ACTION NAME` de la salida de `aio app deploy`.

1. Seleccione `wskdebug worker` de la configuración de ejecución/depuración y pulse el icono de reproducción. Espere a que se inicie hasta que muestre **[!UICONTROL Listo para activaciones]** en la ventana **[!UICONTROL Consola de depuración]**.

1. Haga clic en **[!UICONTROL ejecutar]** en Devtool. Puede ver las acciones que se ejecutan en el editor de código de Visual Studio y los registros comienzan a mostrarse.

1. Establezca un punto de interrupción en su código. Entonces corre de nuevo y debería golpear.

Los cambios de código se cargan en tiempo real y son efectivos en cuanto se produce la siguiente activación.

>[!NOTE]
>
>Hay dos activaciones presentes para cada solicitud en las aplicaciones personalizadas. La primera solicitud es una acción web que se llama a sí misma asincrónicamente en el código SDK. La segunda activación es la que se ejecuta en el código.
