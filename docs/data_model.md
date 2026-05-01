# Data Model - Proyecto E-commerce

## 1. Descripción general

Este documento describe el modelo de datos diseñado para un sistema de e-commerce.

El objetivo del modelo es representar de forma estructurada la información relacionada con clientes, productos, categorías, pedidos, líneas de pedido, pagos y valoraciones.

El modelo está diseñado siguiendo criterios de normalización hasta **Tercera Forma Normal (3NF)**, con el fin de reducir redundancias, mejorar la consistencia de los datos y facilitar su análisis posterior en **BigQuery**.

El diagrama ER visual utiliza nombres conceptuales en español para facilitar la comprensión del modelo durante la presentación. Sin embargo, la implementación técnica en BigQuery utiliza nombres físicos en inglés para las tablas: `customers`, `categories`, `products`, `orders`, `order_items`, `payments` y `reviews`.

Este modelo ya fue implementado en BigQuery dentro del proyecto `techzone-494713`, en el dataset `TechZone`.

---

## 2. Entidades principales

El modelo está formado por las siguientes tablas:

| Tabla | Descripción |
|---|---|
| `customers` | Información básica de los clientes |
| `categories` | Clasificación de productos |
| `products` | Catálogo de productos disponibles |
| `orders` | Pedidos realizados por los clientes |
| `order_items` | Detalle de productos incluidos en cada pedido |
| `payments` | Información de pagos asociados a pedidos |
| `reviews` | Valoraciones de productos comprados |

---

## 3. Nomenclatura conceptual y física

Durante el diseño del modelo se utilizan nombres conceptuales en español para facilitar la interpretación del diagrama ER visual.

No obstante, la implementación técnica en BigQuery utiliza nombres físicos en inglés, siguiendo una nomenclatura más estándar para bases de datos y entornos analíticos.

| Nombre conceptual en el diagrama | Nombre físico en BigQuery |
|---|---|
| Clientes | `customers` |
| Categorías | `categories` |
| Productos | `products` |
| Pedidos | `orders` |
| Líneas de pedido | `order_items` |
| Pagos | `payments` |
| Valoraciones | `reviews` |

El modelo fue implementado en BigQuery dentro del proyecto `techzone-494713`, en el dataset `TechZone`.

---

## 4. Diagrama lógico del modelo

Relaciones principales del modelo:

```text
customers 1 ─── N orders

categories 1 ─── N products

orders 1 ─── N order_items

products 1 ─── N order_items

orders 1 ─── N payments

order_items 1 ─── 0..1 reviews
```

La relación entre `orders` y `products` es de tipo **N:M**.

Esta relación se resuelve mediante la tabla intermedia `order_items`, ya que:

- Un pedido puede contener varios productos.
- Un producto puede aparecer en muchos pedidos.
- Cada línea de pedido necesita guardar datos propios de la compra, como cantidad, precio unitario en el momento de la compra y descuento aplicado.

---

## 5. Tablas y campos

### 5.1 Tabla `customers`

Contiene la información básica de contacto, localización y registro de los clientes.

| Campo | Tipo BigQuery | Obligatorio | Descripción |
|---|---|---|---|
| `customer_id` | `INT64` | Sí | Identificador único del cliente |
| `first_name` | `STRING` | Sí | Nombre del cliente |
| `last_name` | `STRING` | Sí | Apellidos del cliente |
| `email` | `STRING` | Sí | Correo electrónico del cliente |
| `phone` | `STRING` | No | Teléfono de contacto del cliente |
| `country` | `STRING` | Sí | País del cliente |
| `city` | `STRING` | Sí | Ciudad del cliente |
| `acquisition_channel` | `STRING` | No | Canal por el que el cliente conoció la empresa |
| `registered_at` | `TIMESTAMP` | Sí | Fecha y hora de registro del cliente |

#### Claves

- **Primary Key:** `customer_id`

#### Comentarios de diseño

El campo `phone` se define como `STRING` porque los teléfonos pueden contener prefijos internacionales, espacios, guiones o ceros iniciales. No se recomienda almacenarlos como valores numéricos.

El campo `email` se considera obligatorio porque es un dato básico de identificación y contacto del cliente.

---

### 5.2 Tabla `categories`

Contiene la clasificación de los productos del e-commerce.

| Campo | Tipo BigQuery | Obligatorio | Descripción |
|---|---|---|---|
| `category_id` | `INT64` | Sí | Identificador único de la categoría |
| `name` | `STRING` | Sí | Nombre de la categoría |
| `description` | `STRING` | No | Descripción de la categoría |

#### Claves

- **Primary Key:** `category_id`

#### Comentarios de diseño

La tabla `categories` permite evitar repetir el nombre y la descripción de la categoría dentro de cada producto.

---

### 5.3 Tabla `products`

Contiene el catálogo de productos disponibles en el e-commerce.

| Campo | Tipo BigQuery | Obligatorio | Descripción |
|---|---|---|---|
| `product_id` | `INT64` | Sí | Identificador único del producto |
| `category_id` | `INT64` | Sí | Identificador de la categoría del producto |
| `name` | `STRING` | Sí | Nombre del producto |
| `description` | `STRING` | No | Descripción del producto |
| `sale_price` | `NUMERIC` | Sí | Precio actual de venta del producto |
| `cost_price` | `NUMERIC` | Sí | Coste del producto |
| `stock_qty` | `INT64` | Sí | Stock disponible |
| `is_active` | `BOOL` | Sí | Indica si el producto está activo o inactivo |

#### Claves

- **Primary Key:** `product_id`
- **Foreign Key:** `category_id` referencia a `categories.category_id`

#### Comentarios de diseño

Los campos `sale_price` y `cost_price` se definen como `NUMERIC` porque representan importes monetarios. Este tipo es más adecuado que `FLOAT64` para evitar problemas de precisión en cálculos económicos.

El campo `is_active` permite mantener productos históricos en la base de datos aunque ya no estén disponibles para la venta.

---

### 5.4 Tabla `orders`

Contiene los pedidos realizados por los clientes.

| Campo | Tipo BigQuery | Obligatorio | Descripción |
|---|---|---|---|
| `order_id` | `INT64` | Sí | Identificador único del pedido |
| `customer_id` | `INT64` | Sí | Cliente que realizó el pedido |
| `status` | `STRING` | Sí | Estado actual del pedido |
| `shipping_address` | `STRING` | Sí | Dirección de envío |
| `shipping_city` | `STRING` | Sí | Ciudad de envío |
| `shipping_country` | `STRING` | Sí | País de envío |
| `order_date` | `TIMESTAMP` | Sí | Fecha y hora en la que se realizó el pedido |
| `shipped_at` | `TIMESTAMP` | No | Fecha y hora de envío del pedido |
| `delivered_at` | `TIMESTAMP` | No | Fecha y hora de entrega del pedido |

#### Claves

- **Primary Key:** `order_id`
- **Foreign Key:** `customer_id` referencia a `customers.customer_id`

#### Valores posibles para `status`

```text
pending
confirmed
shipped
delivered
cancelled
returned
```

#### Comentarios de diseño

Los campos `shipped_at` y `delivered_at` son opcionales porque no todos los pedidos estarán enviados o entregados en el momento de registrarse.

Los datos de envío se almacenan en `orders` porque representan la dirección concreta utilizada para ese pedido. Aunque el cliente cambie su dirección posteriormente, el pedido debe conservar la información histórica del envío.

---

### 5.5 Tabla `order_items`

Contiene el detalle de los productos incluidos en cada pedido.

Esta tabla actúa como tabla intermedia entre `orders` y `products`, resolviendo la relación **N:M** entre ambas entidades.

| Campo | Tipo BigQuery | Obligatorio | Descripción |
|---|---|---|---|
| `order_item_id` | `INT64` | Sí | Identificador único de la línea de pedido |
| `order_id` | `INT64` | Sí | Pedido al que pertenece la línea |
| `product_id` | `INT64` | Sí | Producto comprado |
| `quantity` | `INT64` | Sí | Cantidad comprada |
| `purchase_unit_price` | `NUMERIC` | Sí | Precio unitario en el momento de la compra |
| `discount_amount` | `NUMERIC` | Sí | Descuento aplicado a la línea de pedido |

#### Claves

- **Primary Key:** `order_item_id`
- **Foreign Key:** `order_id` referencia a `orders.order_id`
- **Foreign Key:** `product_id` referencia a `products.product_id`

#### Comentarios de diseño

El campo `purchase_unit_price` se almacena en `order_items` porque el precio actual de un producto puede cambiar con el tiempo. De esta forma, se conserva el precio real pagado por el cliente en el momento de la compra.

El campo `discount_amount` se considera obligatorio y puede tomar valor `0` cuando no exista descuento aplicado.

La tabla `order_items` permite calcular métricas como:

- Importe bruto por línea.
- Descuento aplicado.
- Importe neto.
- Número de unidades vendidas.
- Productos más vendidos.

---

### 5.6 Tabla `payments`

Contiene la información de pagos asociados a los pedidos.

| Campo | Tipo BigQuery | Obligatorio | Descripción |
|---|---|---|---|
| `payment_id` | `INT64` | Sí | Identificador único del pago |
| `order_id` | `INT64` | Sí | Pedido asociado al pago |
| `payment_method` | `STRING` | Sí | Método de pago utilizado |
| `payment_status` | `STRING` | Sí | Estado del pago |
| `amount` | `NUMERIC` | Sí | Importe del pago |
| `payment_date` | `TIMESTAMP` | No | Fecha y hora del pago |

#### Claves

- **Primary Key:** `payment_id`
- **Foreign Key:** `order_id` referencia a `orders.order_id`

#### Valores posibles para `payment_status`

```text
completed
refunded
pending
failed
```

#### Valores posibles para `payment_method`

```text
card
paypal
bank_transfer
bizum
```

#### Comentarios de diseño

Los pagos se modelan en una tabla separada de `orders` porque un pedido puede tener varios eventos de pago, como:

- Reintentos de pago.
- Pagos fallidos.
- Pagos parciales.
- Reembolsos.

El campo `payment_date` se deja como opcional porque puede no existir si el pago está pendiente o ha fallado antes de procesarse.

---

### 5.7 Tabla `reviews`

Contiene las valoraciones realizadas por los clientes sobre productos comprados.

La valoración se asocia a `order_items` y no directamente a `orders`, porque un pedido puede contener varios productos y cada producto puede recibir una valoración distinta.

| Campo | Tipo BigQuery | Obligatorio | Descripción |
|---|---|---|---|
| `review_id` | `INT64` | Sí | Identificador único de la valoración |
| `order_item_id` | `INT64` | Sí | Línea de pedido valorada |
| `rating` | `INT64` | Sí | Puntuación numérica de la valoración |
| `comment` | `STRING` | No | Comentario opcional del cliente |
| `review_date` | `TIMESTAMP` | Sí | Fecha y hora de la valoración |

#### Claves

- **Primary Key:** `review_id`
- **Foreign Key:** `order_item_id` referencia a `order_items.order_item_id`

#### Restricción lógica recomendada

```text
rating debe estar entre 1 y 5
```

#### Comentarios de diseño

La relación entre `order_items` y `reviews` es de tipo **1:0..1**, ya que una línea de pedido puede no tener valoración o puede tener como máximo una valoración.

---

## 6. Relaciones entre tablas

| Relación | Cardinalidad | Descripción |
|---|---|---|
| `customers` → `orders` | 1:N | Un cliente puede realizar varios pedidos |
| `categories` → `products` | 1:N | Una categoría puede contener varios productos |
| `orders` → `order_items` | 1:N | Un pedido puede tener varias líneas de pedido |
| `products` → `order_items` | 1:N | Un producto puede aparecer en varias líneas de pedido |
| `orders` → `payments` | 1:N | Un pedido puede tener varios pagos o intentos de pago |
| `order_items` → `reviews` | 1:0..1 | Una línea de pedido puede tener ninguna o una valoración |

La relación `order_items` → `reviews` se representa como `1:0..1`, lo que equivale a una relación **1:1 opcional**: una línea de pedido puede no tener valoración o tener como máximo una.

---

## 7. Campos obligatorios y opcionales

### 7.1 Campos opcionales

Los siguientes campos pueden aceptar valores `NULL`:

| Tabla | Campo | Motivo |
|---|---|---|
| `customers` | `phone` | El cliente puede no proporcionar teléfono |
| `customers` | `acquisition_channel` | Puede no conocerse el canal de adquisición |
| `categories` | `description` | La descripción puede no ser necesaria |
| `products` | `description` | Puede existir un producto sin descripción detallada |
| `orders` | `shipped_at` | El pedido puede no haberse enviado todavía |
| `orders` | `delivered_at` | El pedido puede no haberse entregado todavía |
| `payments` | `payment_date` | Puede no existir si el pago está pendiente o fallido |
| `reviews` | `comment` | El cliente puede valorar sin escribir comentario |

### 7.2 Campos obligatorios

El resto de campos se consideran obligatorios porque son necesarios para:

- Identificar registros.
- Mantener relaciones entre tablas.
- Registrar hechos principales del negocio.
- Calcular métricas analíticas.
- Asegurar la trazabilidad de pedidos, pagos y valoraciones.

---

## 8. Justificación de tipos de datos en BigQuery

| Tipo BigQuery | Uso en el modelo |
|---|---|
| `INT64` | Identificadores, cantidades y puntuaciones |
| `STRING` | Textos, nombres, estados, métodos, direcciones y teléfonos |
| `NUMERIC` | Importes monetarios, precios, costes y descuentos |
| `BOOL` | Indicadores verdadero/falso |
| `TIMESTAMP` | Fechas con hora |

### Decisiones sobre tipos de datos

Se utiliza `INT64` para identificadores como `customer_id`, `product_id` u `order_id`, ya que son claves numéricas.

Se utiliza `STRING` para campos textuales como nombres, estados, métodos de pago y direcciones.

Se utiliza `NUMERIC` para precios e importes porque es más adecuado que `FLOAT64` para valores monetarios. `NUMERIC` evita problemas de precisión en operaciones financieras.

Se utiliza `BOOL` para `is_active`, ya que solo necesita representar dos estados: activo o inactivo.

Se utiliza `TIMESTAMP` para fechas con hora, como `registered_at`, `order_date`, `payment_date` o `review_date`.

---

## 9. Decisiones de diseño ER

### 9.1 Separación de entidades principales

Se han separado las entidades principales del negocio para evitar duplicidad de información y mejorar la trazabilidad:

- Clientes en `customers`.
- Categorías en `categories`.
- Productos en `products`.
- Pedidos en `orders`.
- Líneas de pedido en `order_items`.
- Pagos en `payments`.
- Valoraciones en `reviews`.

Esta separación permite que cada tabla represente una única entidad o concepto de negocio.

---

### 9.2 Relación entre clientes y pedidos

Un cliente puede realizar varios pedidos, pero cada pedido pertenece a un único cliente.

Por eso se define la relación:

```text
customers 1 ─── N orders
```

En la tabla `orders` se almacena `customer_id` como clave foránea.

Esta decisión evita repetir los datos del cliente en cada pedido. Por ejemplo, no se almacenan `first_name`, `last_name` o `email` dentro de `orders`.

---

### 9.3 Relación entre categorías y productos

Una categoría puede contener varios productos, pero cada producto pertenece a una única categoría.

Por eso se define la relación:

```text
categories 1 ─── N products
```

En la tabla `products` se almacena `category_id` como clave foránea.

Esta decisión evita repetir el nombre y la descripción de la categoría en cada producto.

---

### 9.4 Relación entre pedidos y productos

La relación entre pedidos y productos es de tipo **N:M**:

- Un pedido puede incluir varios productos.
- Un producto puede aparecer en muchos pedidos.

Para resolver esta relación se crea la tabla intermedia `order_items`.

```text
orders 1 ─── N order_items N ─── 1 products
```

Además, `order_items` permite almacenar datos propios de cada línea de pedido:

- `quantity`
- `purchase_unit_price`
- `discount_amount`

Estos campos no pertenecen únicamente al pedido ni únicamente al producto, sino a la combinación entre ambos.

---

### 9.5 Separación de pagos respecto a pedidos

Los pagos se almacenan en una tabla independiente llamada `payments`.

La relación definida es:

```text
orders 1 ─── N payments
```

Aunque un pedido normalmente tenga un único pago completado, el modelo permite varios registros de pago para soportar situaciones reales como:

- Reintentos de pago.
- Pagos fallidos.
- Pagos parciales.
- Reembolsos.

Esto hace que el modelo sea más flexible y más cercano al funcionamiento real de un e-commerce.

---

### 9.6 Valoraciones asociadas a líneas de pedido

Las valoraciones se relacionan con `order_items` en lugar de relacionarse directamente con `orders` o `products`.

La relación definida es:

```text
order_items 1 ─── 0..1 reviews
```

Esta decisión permite que una valoración corresponda a un producto concreto que ha sido comprado dentro de un pedido.

Si la valoración se asociara solo al pedido, no sería posible saber qué producto concreto se está valorando cuando un pedido contiene varios productos.

---

## 10. Justificación de normalización hasta 3NF

El modelo está normalizado hasta **Tercera Forma Normal (3NF)**.

Esto significa que el diseño evita:

- Grupos repetidos.
- Dependencias parciales.
- Dependencias transitivas.
- Duplicidad innecesaria de datos.

---

### 10.1 Primera Forma Normal, 1NF

El modelo cumple la Primera Forma Normal porque todos los campos contienen valores atómicos.

Esto significa que cada campo almacena un único valor y no listas o conjuntos de valores.

#### Ejemplo incorrecto

No sería correcto guardar varios productos dentro de un único campo en la tabla `orders`:

```text
orders
- order_id
- customer_id
- products = "Laptop, Ratón, Teclado"
```

#### Ejemplo correcto

En el modelo propuesto, los productos de cada pedido se almacenan en filas separadas dentro de `order_items`:

```text
orders
- order_id
- customer_id

order_items
- order_item_id
- order_id
- product_id
- quantity
```

De esta forma, cada fila de `order_items` representa un único producto dentro de un pedido.

---

### 10.2 Segunda Forma Normal, 2NF

El modelo cumple la Segunda Forma Normal porque los atributos dependen completamente de la clave primaria de cada tabla.

Por ejemplo, en `order_items`, los campos:

```text
quantity
purchase_unit_price
discount_amount
```

dependen de la línea de pedido concreta, representada por `order_item_id`.

Los datos descriptivos del producto, como:

```text
name
description
sale_price
cost_price
```

no se repiten en `order_items`, sino que se almacenan únicamente en la tabla `products`.

Esto evita dependencias parciales y duplicidad de información.

---

### 10.3 Tercera Forma Normal, 3NF

El modelo cumple la Tercera Forma Normal porque no existen dependencias transitivas entre campos no clave.

Un atributo no clave debe depender únicamente de la clave primaria de su tabla, y no de otro atributo no clave.

#### Ejemplo con productos y categorías

En la tabla `products` no se almacena el nombre de la categoría directamente.

Diseño correcto:

```text
products
- product_id
- category_id
- name
- sale_price

categories
- category_id
- name
- description
```

El nombre y la descripción de la categoría dependen de `category_id`, no de `product_id`.

Por tanto, los datos de categoría se almacenan en `categories` y no se repiten en `products`.

#### Ejemplo con clientes y pedidos

En la tabla `orders` no se repiten datos del cliente como:

```text
customer_email
customer_city
customer_country
```

En su lugar, `orders` almacena únicamente:

```text
customer_id
```

Los datos del cliente se almacenan en la tabla `customers`.

Esto evita que, si cambia el email o algún dato del cliente, haya que actualizar múltiples pedidos históricos.

---

## 11. Consideraciones para BigQuery

Aunque BigQuery permite definir claves primarias y foráneas, estas restricciones no se aplican de la misma forma que en bases de datos transaccionales tradicionales. Aun así, se documentan en el modelo para mantener claridad en el diseño lógico y facilitar el análisis.

El modelo está pensado para poder utilizarse en BigQuery con fines analíticos, permitiendo consultas como:

- Ventas totales por cliente.
- Ventas por producto.
- Ventas por categoría.
- Ingresos por método de pago.
- Productos más vendidos.
- Margen de beneficio por producto.
- Pedidos por estado.
- Valoración media por producto.
- Evolución temporal de ventas.

---

## 12. Ejemplos de métricas derivadas

A partir del modelo se pueden calcular métricas importantes para el negocio.

### Importe bruto de una línea de pedido

```text
quantity * purchase_unit_price
```

### Importe neto de una línea de pedido

```text
(quantity * purchase_unit_price) - discount_amount
```

### Margen unitario estimado

```text
purchase_unit_price - cost_price
```

### Margen total estimado por línea

```text
(quantity * (purchase_unit_price - cost_price)) - discount_amount
```

---

## 13. Resumen de claves

| Tabla | Primary Key | Foreign Keys |
|---|---|---|
| `customers` | `customer_id` | No aplica |
| `categories` | `category_id` | No aplica |
| `products` | `product_id` | `category_id` |
| `orders` | `order_id` | `customer_id` |
| `order_items` | `order_item_id` | `order_id`, `product_id` |
| `payments` | `payment_id` | `order_id` |
| `reviews` | `review_id` | `order_item_id` |

---

## 14. Resumen de tablas para BigQuery

### 14.1 `customers`

```sql
customer_id INT64 NOT NULL,
first_name STRING NOT NULL,
last_name STRING NOT NULL,
email STRING NOT NULL,
phone STRING,
country STRING NOT NULL,
city STRING NOT NULL,
acquisition_channel STRING,
registered_at TIMESTAMP NOT NULL
```

### 14.2 `categories`

```sql
category_id INT64 NOT NULL,
name STRING NOT NULL,
description STRING
```

### 14.3 `products`

```sql
product_id INT64 NOT NULL,
category_id INT64 NOT NULL,
name STRING NOT NULL,
description STRING,
sale_price NUMERIC NOT NULL,
cost_price NUMERIC NOT NULL,
stock_qty INT64 NOT NULL,
is_active BOOL NOT NULL
```

### 14.4 `orders`

```sql
order_id INT64 NOT NULL,
customer_id INT64 NOT NULL,
status STRING NOT NULL,
shipping_address STRING NOT NULL,
shipping_city STRING NOT NULL,
shipping_country STRING NOT NULL,
order_date TIMESTAMP NOT NULL,
shipped_at TIMESTAMP,
delivered_at TIMESTAMP
```

### 14.5 `order_items`

```sql
order_item_id INT64 NOT NULL,
order_id INT64 NOT NULL,
product_id INT64 NOT NULL,
quantity INT64 NOT NULL,
purchase_unit_price NUMERIC NOT NULL,
discount_amount NUMERIC NOT NULL
```

### 14.6 `payments`

```sql
payment_id INT64 NOT NULL,
order_id INT64 NOT NULL,
payment_method STRING NOT NULL,
payment_status STRING NOT NULL,
amount NUMERIC NOT NULL,
payment_date TIMESTAMP
```

### 14.7 `reviews`

```sql
review_id INT64 NOT NULL,
order_item_id INT64 NOT NULL,
rating INT64 NOT NULL,
comment STRING,
review_date TIMESTAMP NOT NULL
```

---

## 15. Ejemplo de definición SQL para BigQuery

A continuación se muestra una posible definición de las tablas en BigQuery.

```sql
CREATE TABLE `project.dataset.customers` (
  customer_id INT64 NOT NULL,
  first_name STRING NOT NULL,
  last_name STRING NOT NULL,
  email STRING NOT NULL,
  phone STRING,
  country STRING NOT NULL,
  city STRING NOT NULL,
  acquisition_channel STRING,
  registered_at TIMESTAMP NOT NULL,
  PRIMARY KEY (customer_id) NOT ENFORCED
);

CREATE TABLE `project.dataset.categories` (
  category_id INT64 NOT NULL,
  name STRING NOT NULL,
  description STRING,
  PRIMARY KEY (category_id) NOT ENFORCED
);

CREATE TABLE `project.dataset.products` (
  product_id INT64 NOT NULL,
  category_id INT64 NOT NULL,
  name STRING NOT NULL,
  description STRING,
  sale_price NUMERIC NOT NULL,
  cost_price NUMERIC NOT NULL,
  stock_qty INT64 NOT NULL,
  is_active BOOL NOT NULL,
  PRIMARY KEY (product_id) NOT ENFORCED,
  FOREIGN KEY (category_id) REFERENCES `project.dataset.categories`(category_id) NOT ENFORCED
);

CREATE TABLE `project.dataset.orders` (
  order_id INT64 NOT NULL,
  customer_id INT64 NOT NULL,
  status STRING NOT NULL,
  shipping_address STRING NOT NULL,
  shipping_city STRING NOT NULL,
  shipping_country STRING NOT NULL,
  order_date TIMESTAMP NOT NULL,
  shipped_at TIMESTAMP,
  delivered_at TIMESTAMP,
  PRIMARY KEY (order_id) NOT ENFORCED,
  FOREIGN KEY (customer_id) REFERENCES `project.dataset.customers`(customer_id) NOT ENFORCED
);

CREATE TABLE `project.dataset.order_items` (
  order_item_id INT64 NOT NULL,
  order_id INT64 NOT NULL,
  product_id INT64 NOT NULL,
  quantity INT64 NOT NULL,
  purchase_unit_price NUMERIC NOT NULL,
  discount_amount NUMERIC NOT NULL,
  PRIMARY KEY (order_item_id) NOT ENFORCED,
  FOREIGN KEY (order_id) REFERENCES `project.dataset.orders`(order_id) NOT ENFORCED,
  FOREIGN KEY (product_id) REFERENCES `project.dataset.products`(product_id) NOT ENFORCED
);

CREATE TABLE `project.dataset.payments` (
  payment_id INT64 NOT NULL,
  order_id INT64 NOT NULL,
  payment_method STRING NOT NULL,
  payment_status STRING NOT NULL,
  amount NUMERIC NOT NULL,
  payment_date TIMESTAMP,
  PRIMARY KEY (payment_id) NOT ENFORCED,
  FOREIGN KEY (order_id) REFERENCES `project.dataset.orders`(order_id) NOT ENFORCED
);

CREATE TABLE `project.dataset.reviews` (
  review_id INT64 NOT NULL,
  order_item_id INT64 NOT NULL,
  rating INT64 NOT NULL,
  comment STRING,
  review_date TIMESTAMP NOT NULL,
  PRIMARY KEY (review_id) NOT ENFORCED,
  FOREIGN KEY (order_item_id) REFERENCES `project.dataset.order_items`(order_item_id) NOT ENFORCED
);
```

---

## 16. Conclusión

El modelo propuesto permite representar correctamente el funcionamiento básico de un e-commerce, manteniendo una estructura clara, escalable y normalizada hasta **3NF**.

La separación de entidades reduce la redundancia, mejora la consistencia de los datos y facilita el análisis posterior en BigQuery.

Además, el uso de la tabla `order_items` permite resolver correctamente la relación **N:M** entre pedidos y productos, conservando información clave de cada compra, como la cantidad, el precio unitario en el momento de la compra y los descuentos aplicados.

En resumen, el diseño prioriza:

- Consistencia de datos.
- Escalabilidad.
- Trazabilidad de pedidos y pagos.
- Calidad analítica.
- Normalización hasta 3NF.