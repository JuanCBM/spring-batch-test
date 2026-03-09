# Ejercicio Práctico: Procesamiento por Lotes con Spring Batch

## Introducción
Este proyecto tiene como objetivo familiarizarse con **Spring Batch**, un framework robusto para el procesamiento de grandes volúmenes de datos.

En este ejercicio, crearemos un "Job" que lea información de un fichero Excel (.xlsx) de entrada con cabecera, procese los datos y los distribuya en dos ficheros de salida distintos según una condición lógica.

## El Escenario
Imagina que recibimos un fichero con un listado de **Solicitudes de Pedido**. Cada solicitud tiene un estado: `NACIONAL` o `IMPORTACION`.
Tu misión es separar estas solicitudes en dos ficheros diferentes para que cada departamento pueda gestionar los suyos de forma independiente.

## Requisitos Técnicos
- **Java 17** o superior.
- **Maven**.
- **Spring Boot 3.x** con el starter de **Spring Batch**.
- **Apache POI** o **excel-streaming-reader** para la lectura de ficheros Excel.

## Pasos del Proyecto

### 1. Configuración de dependencias
Asegúrate de incluir en tu `pom.xml` las dependencias de Spring Batch y soporte para Excel:
```xml
<!-- Spring Batch -->
<dependency>
    <groupId>org.springframework.boot</groupId>
    <artifactId>spring-boot-starter-batch</artifactId>
</dependency>

<!-- Soporte para Excel (Apache POI) -->
<dependency>
    <groupId>org.apache.poi</groupId>
    <artifactId>poi-ooxml</artifactId>
    <version>5.3.0</version>
</dependency>
<!-- Opcional: Streaming reader para ficheros grandes -->
<dependency>
    <groupId>com.github.pjfanning</groupId>
    <artifactId>excel-streaming-reader</artifactId>
    <version>4.2.1</version>
</dependency>
```

### 2. Definición del Modelo
Crea una clase `Solicitud` con los siguientes campos:
- `id` (Long)
- `descripcion` (String)
- `tipo` (String) -> Valores: "NACIONAL" o "IMPORTACION"
- `cantidad` (Integer)

### 3. Configuración del Batch (Job & Steps)
Debes configurar un `Job` que contenga **dos pasos (Steps)**:

#### Paso 1: Procesamiento de datos (Chunk Step)
Este step utilizará el patrón **Chunk-oriented processing**:
- **Reader**: Un lector capaz de procesar Excel (como `PoiItemReader` de Spring Batch Extensions o una implementación personalizada usando `excel-streaming-reader`). Debe **saltar la primera línea (cabecera)** del fichero `solicitudes.xlsx`.
- **Processor**: Un `ItemProcessor` que valide los datos o transforme la descripción a mayúsculas.
- **Writer**: Un `ClassifierCompositeItemWriter` o simplemente dos `FlatFileItemWriter` para escribir en `nacional.csv` e `importacion.csv` (las salidas pueden seguir siendo CSV o texto).

#### Paso 2: Finalización (Tasklet Step)
Una vez terminado el procesamiento, añade un segundo step que use un `Tasklet`:
- **Lógica**: Debe imprimir por consola un mensaje indicando que el proceso ha terminado con éxito y, opcionalmente, mover el fichero de entrada a una carpeta llamada `procesados/`.

### 4. Ejecución
Prepara un fichero `src/main/resources/solicitudes.xlsx` con una fila de cabecera y datos de prueba, y lanza la aplicación. ¡Verifica que los ficheros de salida se generan correctamente y que el mensaje del Tasklet aparece en el log!

## Enlaces de Interés (Documentación)
- [Página Oficial de Spring Batch](https://spring.io/projects/spring-batch)
- [Guía Rápida: Creating a Batch Service](https://spring.io/guides/gs/batch-processing/)
- [Conceptos de Tasklet vs Chunk](https://docs.spring.io/spring-batch/docs/current/reference/html/step.html#taskletStep)
- [Spring Batch Excel (Extensiones)](https://github.com/spring-projects/spring-batch-extensions/tree/main/spring-batch-excel)
- [Documentación de Apache POI](https://poi.apache.org/)
- [Ejemplos de Multi-Output en Spring Batch](https://docs.spring.io/spring-batch/docs/current/reference/html/readersAndWriters.html#classifierCompositeItemWriter)

---
*Este enunciado ha sido simplificado para fines educativos.*
