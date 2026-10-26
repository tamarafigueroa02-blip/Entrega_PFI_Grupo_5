# Diccionario de datos · Online Retail II

Fuente: [UCI Machine Learning Repository](https://archive.ics.uci.edu/dataset/502/online+retail+ii). Cada fila del archivo es **una línea de una factura**: un producto dentro de una compra. Una factura con 10 productos distintos ocupa 10 filas.

## 1. Columnas originales del dataset (8)

| Columna | Tipo | Qué significa | Ejemplo | Observaciones de calidad | Uso en el proyecto |
|---|---|---|---|---|---|
| **Invoice** | Texto (código) | Número de factura. Agrupa todas las líneas de una misma compra. | `489434` | Si empieza con **"C"**, es una cancelación o devolución (19.494 filas). Si empieza con **"A"**, es un ajuste contable (6 filas). | Se excluyen las C y las A. Se usa para contar la **Frecuencia** (facturas distintas por cliente). |
| **StockCode** | Texto (código) | Código del producto. | `85048` | Algunos códigos no son productos sino cargos: POST (envío), M (manual), BANK CHARGES, AMAZONFEE, vales de regalo, pruebas. Son 24 códigos y 5.911 filas. | Se excluyen los códigos que no son productos. |
| **Description** | Texto | Nombre del producto. | `15CM CHRISTMAS GLASS BALL 20 LIGHTS` | 4.382 vacíos (0,41 %). | Solo para interpretar códigos. No se usa para modelar. |
| **Quantity** | Número entero | Unidades compradas en esa línea. | `12` | 22.950 filas con valor ≤ 0 (devoluciones o ajustes). | Se excluyen las ≤ 0. Se usa para calcular el Monto. |
| **InvoiceDate** | Fecha y hora | Momento de la compra. | `2009-12-01 07:45:00` | Rango: 01/12/2009 al 09/12/2011. | Define las ventanas de tiempo y la **Recencia**. |
| **Price** | Número decimal | Precio unitario en libras esterlinas (£). | `6.95` | 6.207 filas con valor ≤ 0. | Se excluyen las ≤ 0. Se usa para calcular el Monto. |
| **Customer ID** | Número (identificador) | Código del cliente. | `13085` | 243.007 vacíos (22,77 %): compras sin cliente identificado. | Se eliminan los vacíos. Se reemplaza por un **seudónimo** (columna *Cliente*) por privacidad. |
| **Country** | Texto | País del cliente. | `United Kingdom` | 43 países. El 91,9 % de las filas es del Reino Unido. | Variable descriptiva. Se analiza el sesgo por concentración en Reino Unido. |

## 2. Columnas creadas durante el proyecto

| Columna | Nivel | Cómo se calcula | Qué significa | Etapa |
|---|---|---|---|---|
| **Cliente** | Fila | Seudónimo `CLI_0001`, `CLI_0002`, … asignado en orden aleatorio | Reemplaza al Customer ID real. La tabla de equivalencias no se guarda. | 3.1 |
| **Monto** | Fila | `Quantity × Price` | Importe de la línea de factura en £. | 3.1 |
| **Recencia (R)** | Cliente | Días entre la última compra y la fecha de predicción (01/09/2011), dentro de la ventana de 12 meses | Cuánto hace que el cliente no compra. Más alto = más tiempo sin comprar. | 3.3 |
| **Frecuencia (F)** | Cliente | Cantidad de facturas distintas entre el 01/09/2010 y el 31/08/2011 | Cuántas veces compró en el último año. | 3.3 |
| **Monetario (M)** | Cliente | Suma del Monto entre el 01/09/2010 y el 31/08/2011 | Cuánto gastó en el último año, en £. | 3.3 |
| **churn** *(próxima etapa)* | Cliente | 1 si no compró en septiembre, octubre ni noviembre de 2011; 0 si compró | Variable objetivo de la Fase 2: si el cliente abandonó. | 4B |

> **Nivel "fila"** significa que hay un valor por cada línea de factura. **Nivel "cliente"** significa que hay un valor por cada cliente, después de agrupar todas sus compras.

> Este diccionario se irá ampliando con las variables de la Fase 2 (ticket promedio, antigüedad, días promedio entre compras, variedad de productos y segmento de la Fase 1).
