# SQL Server — Write Advanced T-SQL Queries

## 📌 Descripción

Este repositorio contiene la implementación del laboratorio **"Write advanced T-SQL queries"** de Microsoft Learn, realizado utilizando **SQL Server** y **SQL Server Management Studio (SSMS)**.

El objetivo del laboratorio es trabajar con consultas avanzadas de **T-SQL**, utilizando funciones JSON, expresiones de tabla comunes (CTE) y funciones de ventana para generar informes y procesar información de productos.

Durante el laboratorio se utilizó la base de datos de ejemplo **AdventureWorksLT**, trabajando principalmente con las tablas `SalesLT.Product` y `SalesLT.ProductCategory`.

## 🛠️ Tecnologías utilizadas

* **SQL Server 2022+**
* **SQL Server Management Studio (SSMS)**
* **T-SQL**
* **AdventureWorksLT**
* **JSON**
* **Git / GitHub**

## 🗄️ Base de datos

El laboratorio utiliza la base de datos de ejemplo **AdventureWorksLT**.

Antes de comenzar se verificó que las tablas principales estuvieran disponibles mediante consultas sobre:

* `SalesLT.Product`
* `SalesLT.ProductCategory`

Estas consultas permitieron comprobar que la base de datos estaba correctamente restaurada y accesible.

## 📋 Desarrollo del laboratorio

### 1. Generación de datos JSON

Se utilizó `FOR JSON PATH` para convertir los resultados de una consulta SQL en un **array JSON**.

Se seleccionaron productos con información de color y se ordenaron por precio de forma descendente.

```sql
SELECT
    ProductID,
    Name,
    Color,
    ListPrice
FROM SalesLT.Product
WHERE Color IS NOT NULL
ORDER BY ListPrice DESC
FOR JSON PATH;
```

De esta forma, cada fila de la consulta se convierte automáticamente en un objeto JSON.

### 2. Creación de JSON anidado

Se utilizó `JSON_OBJECT()` para crear información de categoría como un **objeto JSON anidado** dentro de cada producto.

La consulta relaciona:

* `SalesLT.Product`
* `SalesLT.ProductCategory`

mediante un `INNER JOIN`.

El resultado contiene información del producto junto con su categoría agrupada en un objeto JSON.

### 3. CTE y función de ventana `ROW_NUMBER()`

Se creó una **Common Table Expression (CTE)** llamada:

```sql
RankedProducts
```

Esta CTE utiliza `ROW_NUMBER()` para asignar una posición a cada producto según su precio dentro de su categoría.

```sql
ROW_NUMBER() OVER (
    PARTITION BY pc.ProductCategoryID
    ORDER BY p.ListPrice DESC
) AS PriceRank
```

El `PARTITION BY` reinicia la numeración para cada categoría, mientras que `ORDER BY ListPrice DESC` coloca primero los productos con mayor precio.

Posteriormente se filtraron únicamente los **tres primeros productos de cada categoría**.

### 4. Generación del ranking en JSON

El resultado anterior se transformó nuevamente a JSON utilizando:

```sql
FOR JSON PATH, ROOT('TopProducts');
```

`ROOT('TopProducts')` añade una propiedad raíz llamada `TopProducts`, facilitando el uso del resultado en aplicaciones que esperan una estructura JSON con un elemento raíz.

La estructura resultante permite representar un informe de productos clasificados por categoría y precio en un formato adecuado para APIs o aplicaciones web.

### 5. Lectura de JSON con `OPENJSON`

Se creó una variable `NVARCHAR(MAX)` que contiene información de actualización de precios en formato JSON.

Ejemplo:

```json
[
    {
        "ProductID": 680,
        "NewPrice": 1250.00
    },
    {
        "ProductID": 706,
        "NewPrice": 1450.00
    },
    {
        "ProductID": 707,
        "NewPrice": 38.99
    }
]
```

Posteriormente se utilizó `OPENJSON()` para convertir el array JSON en **filas y columnas SQL**.

### 6. Definición del esquema con `WITH`

Dentro de `OPENJSON()` se utilizó `WITH` para definir las columnas que se quieren obtener:

```sql
WITH (
    ProductID INT '$.ProductID',
    NewPrice DECIMAL(10,2) '$.NewPrice'
)
```

Esto permite indicar tanto el **tipo de dato** como la ruta JSON (`$.PropertyName`) correspondiente a cada propiedad.

### 7. Combinar JSON con datos existentes

Finalmente, los datos obtenidos mediante `OPENJSON()` se combinaron con `SalesLT.Product` mediante un `INNER JOIN`.

La consulta permite comparar:

* Precio actual.
* Nuevo precio.
* Diferencia entre ambos precios.

```sql
updates.NewPrice - p.ListPrice AS PriceDifference
```

De esta manera se puede analizar información recibida en formato JSON junto con los datos almacenados actualmente en la base de datos.

## 📁 Conceptos trabajados

| Concepto        | Función                                        |
| --------------- | ---------------------------------------------- |
| `FOR JSON PATH` | Convierte resultados SQL a JSON                |
| `JSON_OBJECT()` | Crea objetos JSON                              |
| `CTE`           | Organiza y reutiliza la lógica de una consulta |
| `ROW_NUMBER()`  | Asigna posiciones dentro de grupos             |
| `PARTITION BY`  | Reinicia el ranking por categoría              |
| `OPENJSON()`    | Convierte JSON en filas SQL                    |
| `WITH`          | Define columnas, tipos y rutas JSON            |
| `ROOT()`        | Añade un elemento raíz al JSON                 |
| `INNER JOIN`    | Combina JSON procesado con datos SQL           |

## 🎯 Objetivos alcanzados

Durante este laboratorio se practicó:

* Generación de JSON desde consultas SQL.
* Creación de estructuras JSON anidadas.
* Uso de `JSON_OBJECT()`.
* Uso de `FOR JSON PATH`.
* Creación y utilización de CTE.
* Uso de funciones de ventana.
* Ranking de productos mediante `ROW_NUMBER()`.
* Obtención de los productos con mayor precio por categoría.
* Conversión de JSON a filas mediante `OPENJSON()`.
* Definición de esquemas para datos JSON.
* Combinación de datos JSON con tablas relacionales.
* Cálculo de diferencias entre precios actuales y nuevos.

## ✅ Resultado

Laboratorio completado correctamente, trabajando con funcionalidades avanzadas de **T-SQL** para generar y procesar datos JSON, crear rankings mediante funciones de ventana y combinar información procedente de JSON con datos almacenados en tablas relacionales.

Este ejercicio permitió practicar técnicas especialmente útiles para escenarios de **APIs, catálogos web, informes y procesamiento de datos JSON**.

## 📚 Referencia

Laboratorio realizado siguiendo el ejercicio oficial de Microsoft Learn:

**Write advanced T-SQL queries**

[Microsoft Learn — Write advanced T-SQL queries](https://microsoftlearning.github.io/mslearn-sql-developer/Instructions/Labs/03-write-advanced-tsql-code.html?utm_source=chatgpt.com)
