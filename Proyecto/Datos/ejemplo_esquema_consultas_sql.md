# Cómo usar un esquema para escribir consultas SQL

El **esquema de una base de datos** describe qué tablas existen, qué columnas tiene cada tabla y cómo se relacionan. Puedes imaginar una tabla como una hoja de Excel: las columnas representan características y las filas contienen registros. Conocer el esquema te permite localizar la información necesaria para responder una pregunta mediante SQL.

## 1. Una base de datos pequeña

Supongamos que una tienda tiene dos tablas: `clientes` y `pedidos`.

**Tabla `clientes`:**

| id_cliente | nombre |
|---|---|
| 1 | Ana |
| 2 | Luis |

**Tabla `pedidos`:**

| id_pedido | producto | id_cliente |
|---|---|---|
| 101 | Teclado | 1 |
| 102 | Monitor | 2 |
| 103 | Ratón | 1 |

En este ejemplo simplificado, cada pedido registra un producto. Los valores de las tablas son los **datos**; su organización se describe mediante este **esquema**:

```text
clientes
  id_cliente  (entero, clave primaria)
  nombre      (texto)

pedidos
  id_pedido   (entero, clave primaria)
  producto    (texto)
  id_cliente  (entero, clave foránea hacia clientes.id_cliente)
```

La clave primaria (**PK**) identifica de forma única cada fila de su tabla. La clave foránea (**FK**) permite asociar una fila con un registro de otra tabla. Aquí, `pedidos.id_cliente` guarda el identificador del cliente que hizo el pedido:

```text
pedidos.id_cliente → clientes.id_cliente
```

Por ejemplo, los pedidos 101 y 103 pertenecen a Ana porque ambos tienen `id_cliente = 1`. Ese valor puede repetirse en `pedidos`, ya que un cliente puede hacer varios pedidos.

## 2. Consultar una sola tabla

**Pregunta: ¿Cuáles son los nombres de los clientes?**

El esquema indica que la información está en la columna `nombre` de la tabla `clientes`:

```sql
SELECT nombre
FROM clientes;
```

- `SELECT nombre` indica qué columna queremos obtener.
- `FROM clientes` señala en qué tabla buscarla.

**Resultado:**

| nombre |
|---|
| Ana |
| Luis |

## 3. Filtrar los registros

**Pregunta: ¿Qué productos pidió el cliente con identificador 1?**

La tabla `pedidos` contiene tanto el producto como el identificador del cliente. Podemos resolver la pregunta usando únicamente esa tabla:

```sql
SELECT producto
FROM pedidos
WHERE id_cliente = 1;
```

`WHERE` establece una condición: solamente se muestran los productos de las filas cuyo `id_cliente` sea igual a `1`.

**Resultado:**

| producto |
|---|
| Teclado |
| Ratón |

## 4. Consultar dos tablas relacionadas

**Pregunta: ¿Qué productos pidió Ana?**

El nombre del cliente está en `clientes`, mientras que los productos están en `pedidos`. El esquema muestra que ambas tablas se conectan mediante `id_cliente`:

```sql
SELECT pedidos.producto
FROM pedidos
JOIN clientes
    ON pedidos.id_cliente = clientes.id_cliente
WHERE clientes.nombre = 'Ana';
```

Podemos leer la consulta así:

1. `FROM pedidos` toma la tabla de pedidos como punto de partida.
2. `JOIN clientes ON ...` combina cada pedido con el cliente cuyo identificador coincide.
3. `WHERE clientes.nombre = 'Ana'` conserva las filas correspondientes a clientes llamados Ana.
4. `SELECT pedidos.producto` muestra el producto de esas filas.

La expresión `pedidos.id_cliente` significa «la columna `id_cliente` de la tabla `pedidos`». Escribir el nombre de la tabla ayuda a distinguir columnas que tienen el mismo nombre. Las comillas en `'Ana'` indican que se trata de un valor de texto.

**Resultado para los datos del ejemplo:**

| producto |
|---|
| Teclado |
| Ratón |

La relación declarada como clave foránea nos orienta para escribir la condición del `JOIN`. La consulta debe indicar explícitamente esa condición. Si hubiera varios clientes llamados Ana, esta consulta incluiría los pedidos de todos ellos.

Los resultados se muestran aquí en un orden ilustrativo: SQL no garantiza un orden de presentación a menos que la consulta incluya `ORDER BY`.

## 5. Relación con Text-to-SQL y Spider

En Text-to-SQL, el sistema recibe una pregunta y el esquema de la base objetivo. El esquema le permite identificar las tablas y columnas disponibles y las relaciones que puede utilizar para construir la consulta.

En Spider, `db_id` identifica la base a la que pertenece cada ejemplo. El notebook utiliza ese identificador para asociar la pregunta con su esquema. La anotación `query` contiene el SQL de referencia con el que se puede comparar la consulta generada.

```text
Pregunta: «¿Qué productos pidió Ana?»
                         +
Esquema: clientes, pedidos y su relación mediante id_cliente
                         ↓
Consulta SQL con JOIN y filtro por nombre
                         ↓
Ejecución sobre los datos de la base
                         ↓
Respuesta: Teclado y Ratón
```

El esquema describe dónde y cómo buscar. Para obtener los resultados de una consulta también se necesita acceder a los registros almacenados en la base de datos.
