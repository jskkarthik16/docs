---
title: Construir y probar Java con Gradle
intro: Puedes crear un flujo de trabajo de integración continua (CI) en acciones de GitHub para construir y probar tu proyecto Java con Gradle.
redirect_from:
  - /actions/language-and-framework-guides/building-and-testing-java-with-gradle
  - /actions/guides/building-and-testing-java-with-gradle
versions:
  fpt: '*'
  ghes: '*'
  ghae: '*'
  ghec: '*'
type: tutorial
topics:
  - CI
  - Java
  - Gradle
shortTitle: Crear & probar con Java & Gradle
---

{% data reusables.actions.enterprise-beta %}
{% data reusables.actions.enterprise-github-hosted-runners %}

## Introducción

Esta guía te muestra cómo crear un flujo de trabajo que realiza la integración continua (CI) para tu proyecto Java usando el sistema de construcción Gradle. El flujo de trabajo que creas te permitirá ver cuándo las confirmaciones de una solicitud de extracción causan la construcción o las fallas de prueba en tu rama por defecto; este enfoque puede ayudar a garantizar que tu código siempre sea correcto. Puedes extender tu flujo de trabajo de CI para almacenar en caché los archivos y cargar artefactos desde una ejecución de flujo de trabajo.

{% ifversion ghae %}
{% data reusables.actions.self-hosted-runners-software %}
{% else %}
Los ejecutores alojados {% data variables.product.prodname_dotcom %} tienen una caché de herramientas con software preinstalado, que incluye kits de desarrollo de Java (JDK) y Gradle. Para encontrar una lista de software y de las versiones pre-instaladas de JDK y de Gradle, consulta la sección "[Especificaciones para los ejecutores hospedados en {% data variables.product.prodname_dotcom %}](/actions/reference/specifications-for-github-hosted-runners/#supported-software)".
{% endif %}

## Prerrequisitos

Deberías estar familiarizado con YAML y la sintaxis para las {% data variables.product.prodname_actions %}. Para obtener más información, consulta:
- "[Sintaxis de flujo de trabajo para las {% data variables.product.prodname_actions %}](/actions/automating-your-workflow-with-github-actions/workflow-syntax-for-github-actions)"
- "[Aprende sobre las {% data variables.product.prodname_actions %}](/actions/learn-github-actions)"

Te recomendamos que tengas una comprensión básica de Java y del marco de Gradle. Para obtener más información, consulta [Empezar](https://docs.gradle.org/current/userguide/getting_started.html) en la documentación de Gradle.

{% data reusables.actions.enterprise-setup-prereq %}

## Using the Gradle starter workflow

{% data variables.product.prodname_dotcom %} provides a Gradle starter workflow that will work for most Gradle-based Java projects. For more information, see the [Gradle starter workflow](https://github.com/actions/starter-workflows/blob/main/ci/gradle.yml).

To get started quickly, you can choose the preconfigured Gradle starter workflow when you create a new workflow. Para obtener más información, consulta la "[guía rápida de {% data variables.product.prodname_actions %}](/actions/quickstart)".

También puedes agregar este flujo de trabajo de forma manual al crear un archivo nuevo en el directorio de tu repositorio `.github/workflows`.

```yaml{:copy}
{% data reusables.actions.actions-not-certified-by-github-comment %}

name: Java CI

on: [push]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v2
      - name: Set up JDK 11
        uses: actions/setup-java@v2
        with:
          java-version: '11'
          distribution: 'adopt'
      - name: Validate Gradle wrapper
        uses: gradle/wrapper-validation-action@e6e38bacfdf1a337459f332974bb2327a31aaf4b
      - name: Build with Gradle
        run: ./gradlew build
```

Este flujo de trabajo realiza los siguientes pasos:

1. El paso `checkout (comprobación)` descarga una copia de tu repositorio en el ejecutor.
2. El paso `setup-java` configura el JDK de Java 11 por Adoptium.
3. El paso de "Validar el wrapper de Gradle" valida la sumas de verificaciones de los archivos JAR del wrapper de Gradle que estén presentes en el árbol fuente.
4. El paso "Build with Gradle" (construir con Gradle) ejecuta el script contenedor `gradlew` para asegurar que tu código se cree, las pruebas pasen y se pueda crear un paquete.

The default starter workflows are excellent starting points when creating your build and test workflow, and you can customize the starter workflow to suit your project’s needs.

{% data reusables.github-actions.example-github-runner %}

{% data reusables.github-actions.java-jvm-architecture %}

## Construir y probar tu código

Puedes usar los mismos comandos que usas de forma local para construir y probar tu código.

El flujo de trabajo de inicio ejecutará la tarea `build` por defecto. En la configuración de Gradle predeterminada, este comando descargará las dependencias, construirá clases, ejecutará pruebas y empaquetará las clases en su formato distribuible, por ejemplo, un archivo JAR.

Si usas diferentes comandos para construir tu proyecto, o si quieres usar una tarea diferente, puedes especificarlo. Por ejemplo, es posible que desees ejecutar la tarea `package` que está configurada en tu archivo _ci.gradle_.

{% raw %}
```yaml{:copy}
steps:
  - uses: actions/checkout@v2
  - uses: actions/setup-java@v2
    with:
      java-version: '11'
      distribution: 'adopt'
  - name: Validate Gradle wrapper
    uses: gradle/wrapper-validation-action@e6e38bacfdf1a337459f332974bb2327a31aaf4b
  - name: Run the Gradle package task
    run: ./gradlew -b ci.gradle package
```
{% endraw %}

## Almacenar dependencias en caché

Cuando utilizas ejecutores hospedados en {% data variables.product.prodname_dotcom %}, puedes guardar tus dependencias en el caché para acelerar tus ejecuciones de flujo de trabajo. Después de una ejecución exitosa, tu caché de paquete de Gradle local se almacenará en la infraestructura de acciones de GitHub. En las ejecuciones de flujo de trabajo futuras, la caché se restaurará para que las dependencias no necesiten ser descargadas desde los repositorios de paquetes remotos. Puedes guardar las dependencias en caché utilizando simplemente la [acción `setup-java`](https://github.com/marketplace/actions/setup-java-jdk) o puedes utilizar la [Acción `cache`](https://github.com/actions/cache) para tener una configuración personalizada y más avanzada.

{% raw %}
```yaml{:copy}
steps:
  - uses: actions/checkout@v2
  - name: Set up JDK 11
    uses: actions/setup-java@v2
    with:
      java-version: '11'
      distribution: 'adopt'
      cache: gradle
  - name: Validate Gradle wrapper
    uses: gradle/wrapper-validation-action@e6e38bacfdf1a337459f332974bb2327a31aaf4b
  - name: Build with Gradle
    run: ./gradlew build
  - name: Cleanup Gradle Cache
    # Remove some files from the Gradle cache, so they aren't cached by GitHub Actions.
    # Restoring these files from a GitHub Actions cache might cause problems for future builds.
    run: |
      rm -f ~/.gradle/caches/modules-2/modules-2.lock
      rm -f ~/.gradle/caches/modules-2/gc.properties
```
{% endraw %}

Este flujo de trabajo guardará el contenido del caché de tu paquete local de Gradle, el cual se ubica en los directorios `.gradle/caches` y `.gradle/wrapper` dek directorio de inicio del ejecutor. La clave del caché será el contenido cifrado de los archivos de compilación de gradle (incluyendo las propiedades del archivo de envoltorio (wrapper)), así que cualquier cambio que se realice sobre ellos invalidará el caché.

## Empaquetar datos de flujo de trabajo como artefactos

Una vez que tu compilación haya tenido éxito y tus pruebas hayan pasado, es posible que desees cargar los paquetes Java resultantes como un artefacto de construcción. Esto almacenará los paquetes construidos como parte de la ejecución del flujo de trabajo y te permitirá descargarlos. Los artefactos pueden ayudarte a probar y depurar solicitudes de extracción en tu entorno local antes de que se fusionen. Para obtener más información, consulta "[Conservar datos de flujo de trabajo mediante artefactos](/actions/automating-your-workflow-with-github-actions/persisting-workflow-data-using-artifacts)."

Por lo general, Gradle creará archivos de salida como JAR, EAR o WAR en el directorio `build/libs`. Puedes cargar los contenidos de ese directorio utilizando la acción `upload-Artifact`.

{% raw %}
```yaml{:copy}
steps:
  - uses: actions/checkout@v2
  - uses: actions/setup-java@v2
    with:
      java-version: '11'
      distribution: 'adopt'
  - name: Validate Gradle wrapper
    uses: gradle/wrapper-validation-action@e6e38bacfdf1a337459f332974bb2327a31aaf4b
  - run: ./gradlew build
  - uses: actions/upload-artifact@v2
    with:
      name: Package
      path: build/libs
```
{% endraw %}
