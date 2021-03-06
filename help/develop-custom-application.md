---
title: Desarrollo para [!DNL Asset Compute Service]
description: Cree aplicaciones personalizadas con [!DNL Asset Compute Service].
exl-id: a0c59752-564b-4bb6-9833-ab7c58a7f38e
source-git-commit: eed9da4b20fe37a4e44ba270c197505b50cfe77f
workflow-type: tm+mt
source-wordcount: '1605'
ht-degree: 0%

---

# Desarrollo de una aplicación personalizada {#develop}

Antes de comenzar a desarrollar una aplicación personalizada:

* Asegúrese de que se cumplen todos los [requisitos previos](/help/understand-extensibility.md#prerequisites-and-provisioning).
* Instale las [herramientas de software requeridas](/help/setup-environment.md#create-dev-environment).
* Consulte [configuración del entorno](setup-environment.md) para asegurarse de que está preparado para crear una aplicación personalizada.

## Crear una aplicación personalizada {#create-custom-application}

Asegúrese de tener la [[!DNL Adobe I/O] CLI](https://github.com/adobe/aio-cli) instalada localmente.

1. Para crear una aplicación personalizada, [cree una aplicación Firefly](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#4-bootstrapping-new-app-using-the-cli). Para ello, ejecute `aio app init <app-name>` en su terminal.

   Si aún no ha iniciado sesión, este comando le pedirá a un navegador que inicie sesión en la [Consola del desarrollador de Adobe](https://console.adobe.io/) con su Adobe ID. Consulte [aquí](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#3-signing-in-from-cli) para obtener más información sobre cómo iniciar sesión desde el cli.

   Adobe recomienda iniciar sesión. Si tiene problemas, siga las instrucciones [para crear una aplicación sin iniciar sesión](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user).

1. Después de iniciar sesión, siga los mensajes de la CLI y seleccione los `Organization`, `Project` y `Workspace` que se utilizarán para la aplicación. Elija el proyecto y el espacio de trabajo que creó al [configurar el entorno](setup-environment.md).

   ```sh
   $ aio app init <app-name>
   Retrieving information from [!DNL Adobe I/O] Console.
   ? Select Org My Adobe Org
   ? Select Project MyFireflyProject
   ? Select Workspace myworkspace
   create console.json
   ```

1. Cuando se le pida `Which Adobe I/O App features do you want to enable for this project?`, seleccione `Actions`. Asegúrese de anular la selección de la opción `Web Assets` porque los recursos web utilizan distintas comprobaciones de autenticación y autorización.

   ```bash
   ? Which Adobe I/O App features do you want to enable for this project?
   select components to include (Press <space> to select, <a> to toggle all, <i> to invert selection)
   ❯◉ Actions: Deploy Runtime actions
   ◯ Events: Publish to Adobe I/O Events
   ◯ Web Assets: Deploy hosted static assets
   ◯ CI/CD: Include GitHub Actions based workflows for Build, Test and Deploy
   ```

1. Cuando se le pida `Which type of sample actions do you want to create?`, asegúrese de seleccionar `Adobe Asset Compute Worker`:

   ```bash
   ? Which type of sample actions do you want to create?
   Select type of actions to generate
   ❯◉ Adobe Asset Compute Worker
   ◯ Generic
   ```

1. Siga el resto de las indicaciones y abra la nueva aplicación en Visual Studio Code (o su editor de código favorito). Contiene el scaffolding y el código de ejemplo para una aplicación personalizada.

   Lea aquí los [componentes principales de una aplicación Firefly](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#5-anatomy-of-a-project-firefly-application).

   La aplicación de plantilla aprovecha nuestro [Asset compute SDK](https://github.com/adobe/asset-compute-sdk#asset-compute-sdk) para la carga, descarga y organización de representaciones de aplicaciones, de modo que los desarrolladores solo tengan que implementar la lógica de aplicación personalizada. Dentro de la carpeta `actions/<worker-name>` , el archivo `index.js` es donde se agrega el código de aplicación personalizado.

Consulte [ejemplos de aplicaciones personalizadas](#try-sample) para ver ejemplos e ideas para aplicaciones personalizadas.

### Agregar credenciales {#add-credentials}

Al iniciar sesión al crear la aplicación, la mayoría de las credenciales de Firefly se recopilan en el archivo ENV. Sin embargo, el uso de la herramienta para desarrolladores requiere credenciales adicionales.

<!-- TBD: Check if manual setup of credentials is required.
Manual set up of credentials is removed from troubleshooting and best practices page. Link was broken.
If you did not log in, refer to our troubleshooting guide to [set up credentials manually](troubleshooting.md).
-->

#### Credenciales de almacenamiento de herramientas para desarrolladores {#developer-tool-credentials}

La herramienta para desarrolladores utilizada para probar aplicaciones personalizadas con el [!DNL Asset Compute service] real requiere un contenedor de almacenamiento en la nube para alojar archivos de prueba y para recibir y mostrar las representaciones generadas por las aplicaciones.

>[!NOTE]
>
>Esto es independiente del almacenamiento en la nube de [!DNL Adobe Experience Manager] como [!DNL Cloud Service]. Solo se aplica para desarrollar y probar con la herramienta para desarrolladores de Asset compute.

Asegúrese de tener acceso a un [contenedor de almacenamiento en la nube compatible](https://github.com/adobe/asset-compute-devtool#prerequisites). Este contenedor lo pueden compartir varios desarrolladores entre diferentes proyectos según sea necesario.

#### Agregar credenciales al archivo ENV {#add-credentials-env-file}

Agregue las siguientes credenciales para la herramienta de desarrollo al archivo ENV en la raíz de su proyecto de Firefly:

1. Añada la ruta absoluta al archivo de clave privada creado al agregar servicios al proyecto Firefly:

   ```conf
   ASSET_COMPUTE_PRIVATE_KEY_FILE_PATH=
   ```

1. Descargue el archivo desde Adobe Developer Console. Vaya a la raíz del proyecto y haga clic en Descargar todo en la esquina superior derecha. El archivo se descarga con `<namespace>-<workspace>.json` como nombre de archivo. Realice una de las acciones siguientes:

   * Cambie el nombre del archivo como `console.json` y muévalo a la raíz del proyecto.
   * De forma opcional, puede agregar la ruta absoluta al archivo JSON de integración de Adobe Developer Console. Este es el mismo archivo [`console.json`](https://www.adobe.io/project-firefly/docs/getting_started/first_app/#42-developer-is-not-logged-in-as-enterprise-organization-user) que se descarga en el espacio de trabajo del proyecto.

      ```conf
      ASSET_COMPUTE_INTEGRATION_FILE_PATH=
      ```

1. Agregue las credenciales de almacenamiento de S3 o Azure. Solo necesita acceder a una solución de almacenamiento en la nube.

   ```conf
   # S3 credentials
   S3_BUCKET=
   AWS_ACCESS_KEY_ID=
   AWS_SECRET_ACCESS_KEY=
   AWS_REGION=
   
   # Azure Storage credentials
   AZURE_STORAGE_ACCOUNT=
   AZURE_STORAGE_KEY=
   AZURE_STORAGE_CONTAINER_NAME=
   ```

>[!TIP]
>
>El archivo `config.json` contiene credenciales. Desde el proyecto, agregue el archivo JSON a su archivo `.gitignore` para evitar que se comparta. Lo mismo se aplica a los archivos .env y .aio.

## Ejecución de la aplicación {#run-custom-application}

Antes de ejecutar la aplicación con la herramienta para desarrolladores de Asset compute, configure correctamente las [credenciales](#developer-tool-credentials).

Para ejecutar la aplicación en la herramienta para desarrolladores, utilice el comando `aio app run`. Implementa la acción en [!DNL Adobe I/O] Runtime e inicia la herramienta de desarrollo en el equipo local. Esta herramienta se utiliza para probar las solicitudes de aplicación durante el desarrollo. A continuación se muestra un ejemplo de solicitud de representación:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg"
    }
]
```

>[!NOTE]
>
>No utilice el indicador `--local` con el comando `run`. No funciona con [!DNL Asset Compute] aplicaciones personalizadas ni con la herramienta para desarrolladores de Asset compute. Las aplicaciones personalizadas son invocadas por el [!DNL Asset Compute Service] que no puede acceder a las acciones que se ejecutan en los equipos locales del desarrollador.

Consulte [aquí](test-custom-application.md) cómo probar y depurar la aplicación. Cuando haya terminado de desarrollar la aplicación personalizada, [implemente su aplicación personalizada](deploy-custom-application.md).

## Pruebe la aplicación de ejemplo proporcionada por el Adobe {#try-sample}

A continuación se muestran ejemplos de aplicaciones personalizadas:

* [secundario](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic)
* [fotos obreras-animales](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-animal-pictures)

### Aplicación personalizada de plantilla {#template-custom-application}

El [worker-basic](https://github.com/adobe/asset-compute-example-workers/tree/master/projects/worker-basic) es una aplicación de plantilla. Genera una representación simplemente copiando el archivo de origen. El contenido de esta aplicación es la plantilla recibida al elegir `Adobe Asset Compute` en la creación de la aplicación de aio.

El archivo de aplicación, [`worker-basic.js`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-basic/worker-basic.js) utiliza el [`asset-compute-sdk`](https://github.com/adobe/asset-compute-sdk#overview) para descargar el archivo de origen, orquestar cada procesamiento de representación y cargar las representaciones resultantes de nuevo en el almacenamiento en la nube.

El [`renditionCallback`](https://github.com/adobe/asset-compute-sdk#rendition-callback-for-worker-required) definido dentro del código de la aplicación, es donde se realiza toda la lógica de procesamiento de la aplicación. La rellamada de representación en `worker-basic` simplemente copia el contenido del archivo de origen en el archivo de representación.

```javascript
const { worker } = require('@adobe/asset-compute-sdk');
const fs = require('fs').promises;

exports.main = worker(async (source, rendition) => {
    // copy source to rendition to transfer 1:1
    await fs.copyFile(source.path, rendition.path);
});
```

## Llamar a una API externa {#call-external-api}

En el código de la aplicación, puede realizar llamadas de API externas para ayudar en el procesamiento de la aplicación. A continuación se muestra un archivo de aplicación de ejemplo que invoca una API externa.

```javascript
exports.main = worker(async function (source, rendition) {

    const response = await fetch('https://adobe.com', {
        method: 'GET',
        Authorization: params.AUTH_KEY
    })
});
```

Por ejemplo, [`worker-animal-pictures`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L46) realiza una solicitud de recuperación a una URL estática de Wikimedia utilizando la biblioteca [`node-httptransfer`](https://github.com/adobe/node-httptransfer#node-httptransfer).

<!-- TBD: Revisit later to see if this note is required.
>[!NOTE]
>
>For extra authorization for these API calls, see [custom authorization checks](#custom-authorization-checks).
-->

### Pasar parámetros personalizados {#pass-custom-parameters}

Puede pasar parámetros definidos personalizados a través de los objetos de representación. Se puede hacer referencia a ellas dentro de la aplicación en [`rendition` instrucciones](https://github.com/adobe/asset-compute-sdk#rendition). Un ejemplo de objeto de representación es:

```json
"renditions": [
    {
        "worker": "https://1234_my_namespace.adobeioruntime.net/api/v1/web/example-custom-worker-master/worker",
        "name": "image.jpg",
        "my-custom-parameter": "my-custom-parameter-value"
    }
]
```

Un ejemplo de archivo de aplicación que accede a un parámetro personalizado es:

```javascript
exports.main = worker(async function (source, rendition) {

    const customParam = rendition.instructions['my-custom-parameter'];
    console.log('Custom paramter:', customParam);
    // should print out `Custom parameter: "my-custom-parameter-value"`
});
```

El `example-worker-animal-pictures` pasa un parámetro personalizado [`animal`](https://github.com/adobe/asset-compute-example-workers/blob/master/projects/worker-animal-pictures/worker-animal-pictures.js#L39) para determinar qué archivo se recuperará de Wikimedia.

## Compatibilidad con autenticación y autorización {#authentication-authorization-support}

De forma predeterminada, las aplicaciones personalizadas de Asset compute incluyen comprobaciones de autorización y autenticación para aplicaciones de Firefly. Esto se habilita estableciendo la anotación `require-adobe-auth` en `true` en `manifest.yml`.

### Acceso a otras API de Adobe {#access-adobe-apis}

<!-- TBD: Revisit this section. Where do we document console workspace creation?
-->

Agregue los servicios de API al espacio de trabajo de la consola [!DNL Asset Compute] creado en la configuración. Estos servicios forman parte del token de acceso JWT generado por [!DNL Asset Compute Service]. Se puede acceder al token y a otras credenciales dentro del objeto de acción `params` de la aplicación.

```javascript
const accessToken = params.auth.accessToken; // JWT token for Technical Account with entitlements from the console workspace to the API service
const clientId = params.auth.clientId; // Technical Account client Id
const orgId = params.auth.orgId; // Experience Cloud Organization
```

### Pasar credenciales para sistemas de terceros {#pass-credentials-for-tp}

Para gestionar las credenciales de otros servicios externos, pase estos como parámetros predeterminados en las acciones. Estos se cifran automáticamente en tránsito. Para obtener más información, consulte [creación de acciones en la guía para desarrolladores de Runtime](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/creating_actions.md). A continuación, configúrelos mediante variables de entorno durante la implementación. Se puede acceder a estos parámetros en el objeto `params` dentro de la acción.

Establezca los parámetros predeterminados dentro de `inputs` en el `manifest.yml`:

```yaml
packages:
  __APP_PACKAGE__:
    actions:
      worker:
        function: 'index.js'
        runtime: 'nodejs:10'
        web: true
        inputs:
           secretKey: $SECRET_KEY
        annotations:
          require-adobe-auth: true
```

La expresión `$VAR` lee el valor de una variable de entorno denominada `VAR`.

Durante el desarrollo, el valor se puede configurar en el archivo ENV local como `aio` lee automáticamente las variables de entorno de los archivos ENV, además de las variables configuradas desde el shell de invocación. En este ejemplo, el archivo ENV tiene el siguiente aspecto:

```CONF
#...
SECRET_KEY=secret-value
```

Para la implementación de producción, se pueden configurar las variables de entorno en el sistema CI, por ejemplo, utilizando secretos en las acciones de GitHub. Por último, acceda a los parámetros predeterminados dentro de la aplicación como estos:

```javascript
const key = params.secretKey;
```

## Cambio de tamaño de aplicaciones {#sizing-workers}

Una aplicación se ejecuta en un contenedor en [!DNL Adobe I/O] tiempo de ejecución con [límites](https://www.adobe.io/apis/experienceplatform/runtime/docs.html#!adobedocs/adobeio-runtime/master/guides/system_settings.md) que se puede configurar mediante el `manifest.yml`:

```yaml
    actions:
      myworker:
        function: /actions/myworker/index.js
        limits:
          timeout: 300000
          memorySize: 512
          concurrency: 1
```

Debido al procesamiento más extenso que suelen realizar las aplicaciones de Asset compute, es más probable que uno tenga que ajustar estos límites para obtener un rendimiento óptimo (lo suficientemente grande como para gestionar recursos binarios) y eficacia (no malgastar recursos debido a la memoria contenedora no utilizada).

El tiempo de espera predeterminado para las acciones en tiempo de ejecución es de un minuto, pero se puede aumentar estableciendo el límite `timeout` (en milisegundos). Si espera procesar archivos más grandes, incremente esta vez. Tenga en cuenta el tiempo total que se tarda en descargar el origen, procesar el archivo y cargar la representación. Si se agota el tiempo de espera de una acción, es decir, no devuelve la activación antes del límite de tiempo de espera especificado, Runtime descarta el contenedor y no lo vuelve a utilizar.

Por naturaleza, las aplicaciones de asset compute tienden a estar conectadas a la red y al disco de entrada o salida. El archivo de origen debe descargarse primero, el procesamiento suele requerir muchos recursos y, a continuación, las representaciones resultantes se cargan de nuevo.

La memoria disponible para un contenedor de acciones la especifica `memorySize` en MB. Actualmente, esto también define cuánto acceso de CPU obtiene el contenedor y, lo más importante, es un elemento clave del coste de uso del tiempo de ejecución (los contenedores más grandes cuestan más). Utilice aquí un valor mayor cuando el procesamiento requiera más memoria o CPU, pero tenga cuidado de no desperdiciar recursos, ya que cuanto más grandes sean los contenedores, menor será el rendimiento general.

Además, es posible controlar la concurrencia de acciones dentro de un contenedor utilizando la configuración `concurrency`. Este es el número de activaciones simultáneas que obtiene un solo contenedor (de la misma acción). En este modelo, el contenedor de acciones es como un servidor Node.js que recibe varias solicitudes simultáneas, hasta ese límite. Si no se establece, el valor predeterminado en Tiempo de ejecución es 200, que es bueno para las acciones más pequeñas de Firefly, pero normalmente es demasiado grande para las aplicaciones de Asset compute, dado que su procesamiento local y actividad de disco son más intensos. Algunas aplicaciones, dependiendo de su implementación, también podrían no funcionar bien con la actividad concurrente. El SDK de Asset compute garantiza que las activaciones se separen escribiendo archivos en distintas carpetas únicas.

Pruebe las aplicaciones para encontrar los números óptimos para `concurrency` y `memorySize`. Contenedores más grandes = un límite de memoria más alto podría permitir más concurrencia, pero también podría ser derrochador para tráfico más bajo.
