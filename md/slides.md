class: center, middle, slide-title

# HBase y bases de datos columnares

---

# Índice

1. ¿Qué es HBase?

2. Estructura de HBase

3. Arquitectura de HBase

4. Comandos sobre HBase

5. Diseño de tablas en HBase

6. Ejemplo: Twitter <img src="./img/twitter.png" alt="Twitter" height="21" width="21"/>

7. Enlaces

---
class: center, middle, slide-title

# 1. ¿Qué es HBase?

---

## 1.1. HBase 

.right[![hbase](./img/hbase.png "HBase")]



* Apache HBase es una base de datos NoSQL de tipo columnar, Open Source, que se construye sobre HDFS (*Hadoop Distributed File System*) y que permite almacenar grandes colecciones de datasets de una forma distribuida, escalable y tolerante a fallos.

* HBase está desarrollado en Java y se inspira en *Bigtable* de Google


---

## 1.1. HBase

![nosql](./img/nosql.png "Tipos NoSQL")

---

## 1.2. ¿Qué es una BD columnar o *wide column store*?

* Es un tipo de base de datos NoSQL que almacena los datos en columnas en lugar de filas, con el objetivo de poder leer y escribir datos de manera eficiente, disminuyendo el tiempo que se tarda en devolver el resultado de una consulta.

---

## 1.2. ¿Qué es una BD columnar o *wide column store*?

* Ejemplo de base de datos orientada a filas:

![rdbs](./img/rdbs.png "Row-oriented")

---

## 1.2. ¿Qué es una BD columnar o *wide column store*?

* Ejemplo de base de datos orientada a columnas, como HBase:

![columnar](./img/columnar.png "Column-oriented")

---

## 1.3. ¿Cuándo usar HBase?

* Cuando tenemos datasets MUY grandes (millones/billones de filas y columnas), y necesitamos lecturas y escrituras rápidas.

* Cuando los datos se reciben de diferentes orígenes de datos y estos están semi o no estructurados (no como en una RDBMS).

* Cuando tenemos muchas versiones de un dataset y queremos almacenarlas todas.

* HBase está diseñado para tener lecturas y escrituras consistentes. Es decir, una vez que se realiza una escritura, todas las lecturas sobre ese dato devolverán el mismo valor. Esto es algo que Cassandra no cumple, es eventualmente consistente.

* No es la mejor opción para aplicaciones transaccionales o de analíticas relacionales, con queries complejas.

---
class: center, middle, slide-title

# 2. Estructura de HBase

---

# 2. Estructura de HBase

* **Table**
* **Row**
* **Column Family**
* **Column qualifier**
* **Cell**
* **Timestamp**

![table-hbase](./img/tabla-hbase.png "Tabla HBase")

---

# 2. Estructura de HBase

* A veces es más fácil entender el modelo de datos como un *map* multidimensional.

![row-hbase](./img/fila-hbase.png "Fila HBase")

---
class: center, middle, slide-title

# 3. Arquitectura de HBase

---

# 3. Arquitectura de HBase

* HMaster Server
* HBase Region Server
* Zookeeper

![architecture](./img/hbase-architecture.png "HBase architecture")

---
class: center, middle, slide-title

# 4. Comandos sobre HBase

---

# 4. Comandos sobre HBase

* Para conectarnos a la shell de HBase, desde la máquina donde esté instalado:

```shell
hbase shell
```

---

## 4.1. Gestionar tablas

* Crear tabla: `create 'tablename', 'columnfamilyname'`.

```shell
 create 'flights', 'operationalInfo', 'departureInfo' 
```

* List de tablas: `list`.
* Mostrar información de una tabla: `describe 'tablename'`.
* Deshabilitar una tabla: `disable 'tablename'`.
* Borrar tabla: `drop 'tablename'`. Previamente hay que deshabilitarla.

---

## 4.2. Gestionar datos

* **Get:**   `get 'tablename', 'rowname', {parametros adicionales...}`. Devuelve una sola fila. Los parámetros adicionales son, por ejemplo, TIMERANGE, TIMESTAMP, VERSIONS y FILTERS.

```shell
get 'flights', 'id-myflightid', {COLUMN => 'operationalInfo'}
get 'flights', 'id-myflightid', {COLUMN => ['operationalInfo', 'departureInfo']}
get 'flights', 'id-myflightid', {TIMESTAMP => [ts1, ts2]}
```

* **Put:**   `put 'tablename', 'rowname', 'columnfamily:columnvalue', 'value'`. 

* **Scan:**   `scan 'tablename', {parametros opcionales...}`.

```shell
scan 'flights', {COLUMNS => ['operationalInfo', 'departureInfo'], LIMIT => 10, STARTROW => 'xyz'}
```

* **Count:**   `count 'tablename', CACHE =>1000`.
* **Delete:**   `delete 'tablename','rowname','columnname'`.
* **Delete all:**   `deleteall 'tablename', 'rowname'`.
* **Truncate:**   `truncate 'tablename'`

---
class: center, middle, slide-title

# 5. Diseño de tablas en HBase

---

# 5. Diseño de tablas en HBase

* ¿Cuál debe ser la estructura de la *row key* y qué debe contener?

* ¿Cuántas *column families* debe tener la tabla?

* ¿Qué datos contendrá cada *column family*?

* ¿Cuántas columnas tendrá cada *column family*?

* ¿Cuáles deben ser los nombres de las columnas? Aunque no se definan en tiempo de creación de la tabla.

* ¿Qué información debe contener las celdas?

* ¿Cuántas versiones deben almacenarse para cada celda?

---

## 5.1. Diseño de *rowkey*

![diseno_rowkey](./img/diseno_rowkey.png "Diseño")

![diseno_rowkey_performance](./img/diseno_rowkey_performance.png "Diseño")

---

# 5.2. *Tall-Narrow* vs *Flat-Wide*

* *Flat-Wide*: una tabla con pocas filas pero muchas columnas.
![flat](./img/flat-wide.png "flat-wide")

* *Tall-Narrow*: una tabla con pocas columnas pero muchas filas.
![tall](./img/tall-narrow.png "tall-narrow")

---
class: center, middle, slide-title

# 6. Ejemplo: Twitter

<img src="./img/twitter.png" alt="Twitter" height="42" width="42"/>

---

# 6. Ejemplo: Twitter <img src="./img/twitter.png" alt="Twitter" height="42" width="42"/>

El primer paso es definir los patrones de acceso desde la aplicación, es decir, cómo vamos a leer y escribir en la tabla.

* Patrones de lectura:
    1. ¿A quién sigue un usuario?
    2. ¿Sigue un usuario concreto "A" a un usuario concreto "B"?
    3. ¿Quién sigue a un usuario concreto "A"?

* Patrones de escritura:
    1. Hacer *follow*: un usuario comienza a seguir a otro.
    2. Hacer *unfollow*: un usuario deja de seguir a otro.

---

## 6.1 Twitter: primera opción

La *row key* es el ID del usuario seguidor, y cada columna contiene el ID del usuario siendo seguido.

![twitter-1](./img/twitter-1-tabla.png "Tabla twitter opción 1")

![twitter-1-1](./img/twitter-1-tabla2.png "Tabla twitter opción 1")

---

## 6.1 Twitter: primera opción

* ¿A quién sigue un usuario? &#128516;

* ¿Sigue un usuario concreto "A" a un usuario concreto "B"? &#128528;

* Hacer *follow* &#128534;

---

## 6.1 Twitter: primera opción

Una solución es mantener un contador en una columna.

![twitter-1](./img/twitter-1-v2.png "Tabla twitter opción 1 columna")

---

## 6.1 Twitter: primera opción

Y los pasos necesarios para añadir un nuevo usuario a la lista de usuarios seguidos:

![twitter-1](./img/twitter-1-v2-1.png "Tabla twitter opción 1 columna")

---

## 6.2 Twitter: segunda opción

![twitter-2](./img/twitter-2-tabla.png "Tabla twitter opción 2")


---

## 6.2 Twitter: segunda opción

* ¿A quién sigue un usuario? &#128516;

* ¿Sigue un usuario concreto "A" a un usuario concreto "B"? &#128516;

* Hacer *follow* &#128516;

* Hacer *unfollow* &#128516;

* ¿Quién sigue a un usuario concreto "A"?  &#128534;  (*Full table scan*)

---

## 6.2 Twitter: segunda opción

Para solucionarlo tenemos dos opciones:

* Mantener otra tabla que contenga la lista invertida (un usuario y una lista de quiénes le siguen).
* Usar la misma tabla, cambiando la *row key*

![twitter-2-1](./img/twitter-2-1.png "Tabla twitter opción 2.1")


---
## 6.2 Twitter: segunda opción

.right[![twitter-2-2](./img/twitter-2-2.png "Tabla twitter opción 2.2")]

---

# Enlaces

* [Introduction to HBase Schema Design](http://0b4af6cdc2f0c5998459-c0245c5c937c5dedcca3f1764ecc9b2f.r43.cf2.rackcdn.com/9353-login1210_khurana.pdf)
* [HBase documentation](https://hbase.apache.org/book.html)
* [Understanding HBase and Bigtable](https://dzone.com/articles/understanding-hbase-and-bigtab)
* [HBase in Docker](https://github.com/dajobe/hbase-docker)
* [HBase shell commands](https://www.guru99.com/hbase-shell-general-commands.html)
* [HBase tutorial: HBase Introduction and Facebook Case Study](https://www.edureka.co/blog/hbase-tutorial)
* [Fundamentos de una base de datos columnar](https://www.informaticaparatunegocio.com/blog/fundamentos-una-base-datos-columnar/)
* [HBase architecture](https://www.guru99.com/hbase-architecture-data-flow-usecases.html)
