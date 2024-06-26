---
title: Prueba y depuración [!DNL Asset Compute Service] aplicación personalizada
description: Prueba y depuración [!DNL Asset Compute Service] aplicación personalizada.
exl-id: c2534904-0a07-465e-acea-3cb578d3bc08
source-git-commit: c6f747ebd6d1b17834f1af0837609a148804f8a9
workflow-type: tm+mt
source-wordcount: '775'
ht-degree: 0%

---

# Prueba y depuración de una aplicación personalizada {#test-debug-custom-worker}

## Ejecutar pruebas unitarias para una aplicación personalizada {#test-custom-worker}

Instalar [Docker Desktop](https://www.docker.com/get-started) en su máquina. Para probar un trabajador personalizado, ejecute el siguiente comando en la raíz de la aplicación:

```bash
$ aio app test
```

<!-- TBD
To run tests for a custom application, run `aio asset-compute test-worker` command at the root of the custom application application.

Document interactively running `adobe-asset-compute` commands `test-worker` and `run-worker`.
-->

Este comando ejecuta un marco de prueba unitario personalizado para las acciones de la aplicación de Asset compute en el proyecto como se describe a continuación. Se conecta a través de una configuración en la `package.json` archivo. También es posible tener pruebas unitarias de JavaScript como Jest. El `aio app test` ejecuta ambas.

El [aio-cli-plugin-asset-compute](https://github.com/adobe/aio-cli-plugin-asset-compute#install-as-local-devdependency) El complemento está incrustado como una dependencia de desarrollo en la aplicación de aplicación personalizada, por lo que no es necesario instalarlo en los sistemas de compilación/prueba.

### Marco de prueba unitario de aplicación {#unit-test-framework}

El marco de pruebas unitarias de la aplicación de Asset compute permite probar aplicaciones sin escribir código. Se basa en el principio del archivo de origen a representación de las aplicaciones. Se debe configurar una estructura de archivos y carpetas determinada para definir casos de prueba con archivos de origen de prueba, parámetros opcionales, representaciones esperadas y scripts de validación personalizados. De forma predeterminada, las representaciones se comparan para la igualdad de bytes. Además, los servicios HTTP externos se pueden burlar fácilmente mediante archivos JSON simples.

### Agregar pruebas {#add-tests}

Se esperan pruebas dentro del `test` en el nivel raíz del proyecto. Los casos de prueba para cada aplicación deben estar en la ruta `test/asset-compute/<worker-name>`, con una carpeta para cada caso de prueba:

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

El `build` en la raíz de la aplicación de Adobe Developer App Builder se encuentran los resultados detallados de las pruebas y los registros de la aplicación personalizada. Estos detalles también se muestran en la salida del `aio app test` comando.

### Burlar servicios externos {#mock-external-services}

Puede simular llamadas de servicio externas dentro de sus acciones creando lo siguiente `mock-<HOST_NAME>.json` archivos para los escenarios de prueba, siendo HOST_NAME el host específico que desea imitar. Un ejemplo de caso de uso es una aplicación que realiza una llamada independiente a S3. La nueva estructura de prueba tendría este aspecto:

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

El archivo de prueba es una respuesta http con formato JSON. Para obtener más información, consulte [esta documentación](https://www.mock-server.com/mock_server/creating_expectations.html). Si hay varios nombres de host para imitar, defina varios `mock-<mocked-host>.json` archivos. A continuación se muestra un ejemplo de archivo de prueba para `google.com` nombrado `mock-google.com.json`:

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

El ejemplo `worker-animal-pictures` contiene un [archivo ficticio](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/test/asset-compute/worker-animal-pictures/simple-test/mock-upload.wikimedia.org.json) para el servicio Wikimedia con el que interactúa.

#### Uso compartido de archivos entre casos de prueba {#share-files-across-test-cases}

Adobe recomienda utilizar enlaces simbólicos relativos si comparte `file.*`, `params.json` o `validate` scripts en varias pruebas. Son compatibles con Git. Asegúrese de asignar un nombre único a los archivos compartidos, ya que es posible que tenga otros distintos. En el ejemplo siguiente, las pruebas mezclan y hacen coincidir algunos archivos compartidos, y los suyos propios:

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

Los casos de pruebas de error no deben contener un valor esperado `rendition.*` y debe definir el valor esperado `errorReason` dentro de `params.json` archivo.

>[!NOTE]
>
>Si un caso de prueba no contiene un `rendition.*` y no define el valor esperado `errorReason` dentro de `params.json` , se supone que es un caso de error con cualquier `errorReason`.

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

Ver una lista y una descripción completas de [razones de error de asset compute](https://github.com/adobe/asset-compute-commons#error-reasons).

## Depuración de una aplicación personalizada {#debug-custom-worker}

Los pasos siguientes muestran cómo se puede depurar una aplicación personalizada mediante Visual Studio Code. Permite ver registros en directo, puntos de interrupción de visitas y paso a paso por el código, así como la recarga en directo de los cambios del código local en cada activación.

El `aio` de forma predeterminada automatiza muchos de estos pasos. Vaya a la sección Depuración de la aplicación en [Documentación de Adobe Developer App Builder](https://developer.adobe.com/app-builder/docs/getting_started/first_app). Por ahora, los pasos siguientes incluyen una solución.

1. Instale la última versión [wskdebug](https://github.com/apache/openwhisk-wskdebug) desde GitHub y la [ngrok](https://www.npmjs.com/package/ngrok).

   ```shell
   npm install -g @openwhisk/wskdebug
   npm install -g ngrok --unsafe-perm=true
   ```

1. Añada a la configuración de usuario el archivo JSON. Sigue utilizando el depurador de código antiguo de Visual Studio. El nuevo tiene [algunos problemas](https://github.com/apache/openwhisk-wskdebug/issues/74) con wskdebug: `"debug.javascript.usePreview": false`.
1. Cierre todas las instancias de aplicaciones abiertas mediante `aio app run`.
1. Implemente el código más reciente utilizando `aio app deploy`.
1. Ejecute solo la Asset compute Devtool mediante `aio asset-compute devtool`. Manténgalo abierto.
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

   Recupere el `ACTION NAME` de la salida de `aio app deploy`.

1. Seleccionar `wskdebug worker` en la configuración ejecutar/depurar y pulse el icono reproducir. Espere a que se inicie hasta que se muestre **[!UICONTROL Listo para activaciones]** en el **[!UICONTROL Consola de depuración]** ventana.

1. Clic **[!UICONTROL correr]** en el Devtool. Puede ver las acciones que se ejecutan en el editor de código de Visual Studio y los registros comienzan a mostrarse.

1. Establezca un punto de interrupción en su código. Entonces corre de nuevo y debería golpear.

Los cambios de código se cargan en tiempo real y son efectivos en cuanto se produce la siguiente activación.

>[!NOTE]
>
>Hay dos activaciones presentes para cada solicitud en las aplicaciones personalizadas. La primera solicitud es una acción web que se llama a sí misma asincrónicamente en el código SDK. La segunda activación es la que se ejecuta en el código.
