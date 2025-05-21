# Mentoría - Flujo ETL Marklines

## 1. Origen del archivo Excel
El usuario (cliente) coloca un archivo Excel en una carpeta compartida. Esta puede estar ubicada en:

- **EFS (Elastic File System)**: Disco duro compartido en la nube de AWS, accesible por varias aplicaciones.
- **S3 (Simple Storage Service)**: Sistema de almacenamiento en la nube de AWS. Archivos ("objetos") se guardan dentro de "buckets" (carpetas virtuales).

## 2. Disparo automático del proceso (Trigger)
Cuando se sube un archivo nuevo a S3, se activa automáticamente una función Lambda. Esto se llama un **trigger**, y permite iniciar un proceso sin intervención manual.

## 3. Limpieza y transformación del Excel
La función Lambda:
- Lee el archivo Excel con `openpyxl`.
- Toma solo columnas A a H.
- Renombra las columnas con nombres técnicos (por ejemplo, “Group” se convierte en “customer_marklines”).
- Reemplaza valores como `'-'` o `'N/A'` por **0** o los ignora.
- Convierte columnas a tipos de datos correctos (texto, entero, decimal).

Esto se trabaja dentro de un **DataFrame**, que es una tabla en memoria gestionada con la librería `pandas`.

## 4. Conversión a formato Parquet
El archivo limpio se guarda en formato **Parquet**, que es eficiente y comprimido, ideal para grandes volúmenes de datos.

## 5. Subida a la nube (S3)
El archivo `.parquet` se sube a un bucket S3, organizado por año y mes.

Para esto se usa `boto3`, una librería que permite interactuar con los servicios de AWS desde Python.

## 6. Registro de errores (logging en CloudWatch)
Si algo falla, se guarda un mensaje en **CloudWatch**, un servicio que registra todo lo que hace la función. Esto permite revisar fallas después.

## 7. Visualización en Power BI
Una vez cargados en la base de datos Redshift, los datos son consultados desde Power BI para hacer reportes. Esta parte fue mi responsabilidad.

---

## 🔍 Temas clave que podrían preguntarte

### ¿Qué pasa si el archivo Excel no tiene un identificador único?
El flujo actual **no valida unicidad de registros**. Cada vez que se sube un Excel, todos los datos se procesan como nuevos. Si hay duplicados, van a pasar tal cual al DataLake y después a Redshift. Se podría mejorar añadiendo validaciones con un campo ID o combinaciones de columnas únicas.

### ¿Cómo funciona la conexión a la carpeta compartida?
La función Lambda accede a los archivos cuando se suben a un **bucket S3**, que es como una carpeta en la nube. No se conecta directamente a EFS en este ejemplo, aunque el flujo conceptual lo muestra como opción. Todo se maneja por medio de AWS S3 y `boto3`.

### ¿Qué hace exactamente el método de limpieza de datos?
- Lee columnas específicas del Excel.
- Reemplaza valores vacíos o no válidos (`'-'`, `'N/A'`) por cero.
- Convierte tipos de datos (por ejemplo, texto a número).
- Formatea una fecha que viene como `202401` a formato `01/01/2024`.

### ¿Por qué se usa Parquet y no Excel o CSV?
Parquet es un formato **más rápido, liviano y eficiente para análisis de datos**, especialmente en la nube y Big Data. Redshift y otras herramientas de AWS lo leen más fácilmente. Ahorramos espacio y tiempo de procesamiento.

### ¿Y si el archivo tiene columnas nuevas o cambia el formato?
Actualmente no hay validación de estructura del archivo. El código asume que vienen las columnas A–H siempre. Si cambia, puede fallar o procesar mal. Una mejora sería validar la estructura antes de seguir el flujo.

---

## 🧑‍🏫 ¿Qué podrías decir en la sesión?

> “Mi rol en este flujo fue desarrollar el reporte en Power BI. Para eso, me conecté directamente a la base de datos Redshift, donde ya llegan los datos limpios desde el proceso de ETL.  
>
> El flujo completo parte de un archivo Excel que el cliente coloca en una carpeta. Ese archivo se sube a un bucket S3, lo que activa automáticamente una función Lambda. Esa función se encarga de limpiar los datos, convertirlos a formato Parquet, y subirlos nuevamente a S3 organizados por fecha.  
>
> Lo que hace la limpieza es muy específico: toma ciertas columnas, corrige valores como '-' o 'N/A', y asegura que todos los datos estén en el formato correcto. No hay validaciones de duplicados ni identificadores únicos por ahora.  
>
> Una vez que los datos están en el DataLake, un proceso ETL los mueve a Redshift, y desde ahí, se visualizan en Power BI.  
>
> Si se requiere extender o robustecer el flujo, se podrían agregar validaciones de estructura, identificadores únicos y manejo de cambios en el formato del Excel.”
