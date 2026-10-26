# Registro de decisiones (data log) · Grupo 5

Cada decisión del proyecto, con su motivo y la evidencia que la respalda. Las cifras salen del notebook `TP_Final_Grupo5_Segmentacion_y_Churn.ipynb`.

## Decisiones de diseño (antes de modelar)

| # | Decisión | Motivo |
|---|---|---|
| D1 | Una sola fecha de predicción para las dos fases: **01/09/2011** | Con una única fecha de predicción, el segmento de la Fase 1 puede usarse como variable en la Fase 2 sin fuga de datos. Se eligió el 1.º de septiembre para trabajar con meses enteros: la etiqueta usa septiembre, octubre y noviembre completos. Se pierden los últimos 9 días del dataset (1 al 9 de diciembre de 2011). |
| D2 | Ventana de **12 meses** antes de la fecha de predicción (01/09/2010 al 31/08/2011) para RFM y variables | Cubre un ciclo estacional completo, con una sola temporada navideña. El comportamiento reciente predice mejor que el lejano, y es la convención del análisis RFM. |
| D3 | Etiqueta de churn: **1 si el cliente no compra en septiembre, octubre ni noviembre de 2011** | Ventana de 3 meses posterior a la fecha de predicción. Se habla de "tasa de churn observada en la ventana de análisis", nunca de "tasa mensual" (devolución del tutor). |
| D4 | Referencias de la Fase B: **regla por Recencia** (contactar primero a quien hace más tiempo que no compra, sin entrenar) y **regresión logística** (baseline) | La regla es lo que haría el equipo comercial sin modelo y se explica en una frase a un directivo. La regresión logística es un piso simple y fácil de explicar: si XGBoost no la supera con claridad, no se justifica la complejidad. |
| D5 | **XGBoost** es el único modelo complejo, y se compara contra esas dos referencias con validación cruzada (80 % de entrenamiento) y con el 20 % de prueba | No se agregan modelos por agregar (devolución del tutor). La elección se justifica con métricas: PR-AUC en validación cruzada. |
| D6 | `RANDOM_STATE = 42` | Reproducibilidad. |
| D7 | Clientes con **un solo día de compra**: se completan los *días entre compras* con **365** (largo de la ventana). **No** se agrega la columna `compra_unica` | Son 1.648 de 4.324 clientes (38,1 %): 1.578 con una sola factura y 70 con varias facturas el mismo día. Abandonan más: tasa de churn observada en la ventana de 72,3 %, contra 36,7 % en quienes compraron en 2 o más días. Los *días entre compras* cuentan **días de compra distintos** (varias facturas el mismo día son un solo día): si se contaban facturas, esos 70 clientes recibían una distancia cercana a 0 y parecían compradores muy frecuentes, aunque compraron en un único día. La Frecuencia sigue contando facturas. Se descartaron vacío, 0, mediana y exclusión (el vacío no lo aceptan todos los modelos, el 0 y la mediana inventan un ritmo de compra y excluirlos deja afuera al grupo de mayor riesgo). 365 supera cualquier intervalo real (máximo 336) y no usa estadísticos de los datos: sin riesgo de fuga. La columna `compra_unica` se descartó porque era casi una copia de la Frecuencia (1.578 de los 1.648 tienen F = 1) y los árboles ya distinguen el 365; en SHAP, 365 y F = 1 pueden repartirse la importancia. Evidencia: sección 3.5 del notebook. |
| D8 | La población de estudio son los **4.324 clientes con compra en los 12 meses** previos a la fecha de predicción; los **900 clientes sin compras en la ventana** se tratan como grupo aparte de "inactivos", fuera de los modelos | Se aparta de la definición de las instrucciones del proyecto (todos los clientes con al menos una compra antes de la fecha de predicción, 5.224), y se declara en el informe. Motivos: un cliente sin compras en un año ya no es activo; con 21 meses enero-agosto y diciembre se cuentan dos veces; la misma base sirve a las dos fases. Costo: los inactivos son el 17,2 % de los clientes y el 4,2 % de la facturación previa (£562.914); solo 96 (10,7 %) compraron en sep-nov 2011. Robustez: Silhouette igual o mayor con 12 meses para k = 2 a 6 (k = 4: 0,373 contra 0,338); ninguna base llega a 0,45. Evidencia: secciones 3.6 y 4A.1 del notebook. |

## Limpieza (Etapa 3.1)

| Paso | Filas | Clientes | Motivo |
|---|---|---|---|
| 0. Dataset original | 1.067.371 | 5.942 | — |
| 1. Sin duplicados exactos | 1.033.036 | 5.942 | 34.335 copias; el 66,5 % cae entre el 1 y el 9/12/2010 (superposición de las dos hojas del Excel original). |
| 2. Con cliente identificado | 797.885 | 5.942 | Customer ID vacío en el 22,77 % de las filas. La pre-entrega decía 24,9 %, que corresponde a Online Retail I: **corregido**. |
| 3. Sin cancelaciones ni ajustes | 779.495 | 5.881 | Facturas que empiezan con "C" (cancelaciones) o "A" (ajustes contables). |
| 4. Cantidad y precio > 0 | 779.425 | 5.878 | No representan ventas. |
| 5. Solo productos | 776.596 | 5.852 | 24 códigos de cargos excluidos: POST, DOT, M, m, C2, D, S, BANK CHARGES, ADJUST, ADJUST2, AMAZONFEE, CRUK, TEST001, TEST002, B y vales gift_0001_*. |

- **Privacidad:** el Customer ID se reemplaza por un seudónimo (`CLI_0001`, …) asignado en orden aleatorio. La tabla de equivalencias no se guarda en ningún archivo.
- **Monto** = Quantity × Price (en £).
- **Dataset limpio:** 36.594 facturas y £17,07 millones de facturación.

## Tabla RFM y transformaciones (Etapas 3.3 y 3.4)

- **Población:** 4.324 clientes con al menos una compra en la ventana de 12 meses.
- **Recencia:** sin transformar. Su asimetría es 0,42 y el logaritmo la empeora (−0,83).
- **Frecuencia y Monetario:** `log(1 + x)`. La asimetría baja de 10,67 a 1,27 y de 19,21 a 0,36. La winsorización p1–p99 solo llegaba a 3,01 y 3,88, y además iguala a los clientes VIP.
- **Escalado:** `StandardScaler`. Fuera de los bigotes de Tukey quedan 0 % de los clientes en Recencia, 2,6 % en Frecuencia y 1,3 % en Monetario (antes del logaritmo eran 9,9 % y 10,5 % en Frecuencia y Monetario), así que `RobustScaler` no es necesario.
- **Anti-fuga:** el escalador se ajusta solo con el 80 % de entrenamiento, en la Etapa 4A.

## Hallazgos a llevar al informe

- **Estacionalidad:** la facturación sube de septiembre a noviembre en ambos años (pico en noviembre de 2010: £1,16 millones). La ventana de la etiqueta cae en temporada alta, lo que probablemente reduce la tasa de churn observada (limitación).
- **Concentración del gasto:** el 10 % de los clientes genera el 63,9 % de la facturación (el 20 %, el 77,2 %). La media es £2.917 y la mediana £856.
- **País:** el 91,1 % de los clientes es del Reino Unido (sesgo a declarar en ética).
- **Recencia bimodal:** hay un grupo de clientes con última compra entre octubre y diciembre de 2010, probables compradores de temporada. [Hipótesis a verificar en la Fase 2]

## Pendiente de actualizar en el borrador del informe

- El borrador dice 5.878 clientes y £17,7 millones. Con la exclusión de códigos que no son productos, las cifras pasan a **5.852 clientes y £17,07 millones**.
- La concentración del 20 % de clientes pasa de 77,3 % a **77,2 %**.

## Etapa 4A: segmentación (K-Means vs. quintiles RFM)

- **Separación:** 80 % de ajuste (3.459 clientes) y 20 % reservado (865), por cliente, `random_state = 42`. El escalador, la búsqueda de K, el modelo y los cortes de los quintiles se ajustan solo con el 80 %.
- **Escalador confirmado con el 80 %:** `StandardScaler` (Silhouette 0,373 y Davies-Bouldin 0,916) contra `RobustScaler` (0,347 y 0,984). ARI entre ambos: 0,679. Valores extremos con el 80 %: Frecuencia 9,5 % → 2,5 %, Monetario 10,6 % → 1,4 %, Recencia 0 %.
- **Búsqueda de K (2 a 8):** Silhouette máximo con k = 2 (0,430); k = 4: 0,373; ninguno llega a 0,45. Sin codo marcado (la inercia baja 32,7 % de 2 a 3, 23,2 % de 3 a 4 y menos de 16 % desde k = 5).
- **Decisión: K = 4**, por utilidad de negocio. Con k = 3 el grupo de mayor gasto mezcla VIP y Leales (25,3 % de los clientes, 76,5 % de la facturación). Con k = 4 se separan VIP (11,8 % de los clientes, 60,1 % de la facturación) y Leales (30,4 % y 26,5 %). k = 5 solo agrega un grupo de 14,4 % de los clientes y 8,7 % de la facturación.
- **Segmentos (80 %):** VIP, Leales, Nuevos / bajo valor (26,4 %; antigüedad mediana 185 días) y En riesgo (31,4 %; Recencia mediana 295 días, gasto mediano £331). Ojo: "En riesgo" son sobre todo compradores de una sola vez que no volvieron, no clientes valiosos que se van.
- **Baseline:** quintiles RFM (puntajes 1 a 5, cortes del 80 %) armado de dos maneras: tabla de reglas y puntaje total en cuartiles. Con el 80 %: Silhouette 0,373 (K-Means) contra 0,139 (reglas) y 0,247 (puntaje total); Davies-Bouldin 0,916 contra 1,400 y 1,407. K-Means está más equilibrado que las reglas (segmento más grande 31,4 % contra 48,2 %), pero no que el puntaje total (27,1 %), equilibrado por construcción. Limitación: las métricas de distancia favorecen a K-Means por construcción.
- **Estabilidad en el 20 % reservado (criterio ≤ 10 %):** Silhouette 0,373 → 0,364 (−2,6 %); inercia por cliente 0,768 → 0,811 (+5,6 %); Davies-Bouldin 0,916 → 0,920 (+0,4 %). Diferencia máxima de reparto por segmento: 1,9 puntos.
- **Ventana 12 vs. 21 meses con el 20 % reservado:** con k = 4, Silhouette 0,373 contra 0,334 (80 %) y 0,364 contra 0,357 (20 %). La ventana no cambia la conclusión.
- **Segmento de todos los clientes** (para la Fase 2): 521 VIP, 1.324 Leales, 1.125 Nuevos / bajo valor, 1.354 En riesgo.
- **Archivos:** figuras 5 a 9 del informe en `figures/` (`fig05_…` a `fig09_…`); tablas en `results/` (busqueda_k_80, perfil_segmentos_80, comparacion_kmeans_vs_quintiles, estabilidad_holdout, estabilidad_reparto, robustez_ventana_80, etc.).

## Etapa 4B y 5: predicción de abandono (Fase B)

- **Etiqueta y separación:** churn = 1 si no compra en septiembre, octubre ni noviembre de 2011. 4.324 clientes: 3.459 de entrenamiento y 865 de prueba (estratificado por cliente, `random_state = 42`). Tasa de churn observada en la ventana de análisis: 50,3 %.
- **Segmento sin fuga:** el segmento de la Fase A se recalcula solo con el 80 % de entrenamiento (coincide en 98,7 % con el de la Etapa 4A).
- **Validación cruzada (5 grupos estratificados, 80 %):** ROC-AUC y PR-AUC de la regla por Recencia 0,706 y 0,678; de la regresión logística 0,768 y 0,733; de XGBoost 0,774 y 0,745. Los hiperparámetros se eligen por PR-AUC. **Modelo final: XGBoost** (`max_depth = 4`, 100 árboles, `learning_rate = 0,03`).
- **Umbral de decisión: 0,5**, el corte estándar. Las clases están casi parejas, así que no hace falta moverlo. Se fija antes de abrir el 20 % de prueba. (Una versión anterior usaba 0,46 para forzar un Recall de 80 %; se descartó.)
- **20 % de prueba (se abre una sola vez):** XGBoost ROC-AUC 0,782 y PR-AUC 0,750; regresión logística 0,777 y 0,735; regla por Recencia 0,719 y 0,682. Con el corte de 0,5: Recall 73,8 % (321 de 435), Precisión 69,5 %, 141 clientes marcados de más y 114 que se escapan.
- **Lectura de captura:** contactando al 20 % de mayor riesgo (173 clientes de 865) se detecta al 31,5 % de los que abandonan (137 de 435); regresión logística 31,0 %, regla por Recencia 28,5 %, máximo posible 39,8 %. En el decil de mayor riesgo abandona el 82,8 % (72 de 87) y en el de menor riesgo el 4,7 % (4 de 86). Tabla completa en `results/f2_captura_por_decil.csv`.
- **¿Se distingue del azar?** Bootstrap pareado (1.000 remuestreos): la mejora de XGBoost sobre la regresión logística es de +0,005 en ROC-AUC (IC 95 %: −0,006 a +0,016) y +0,013 en PR-AUC (−0,013 a +0,040). Los dos intervalos incluyen el cero: no se puede afirmar que supere a la regresión logística, y sí que supera a la regla por Recencia.
- **Qué impulsa el abandono (SHAP):** Monetario 0,34, Recencia 0,28, Frecuencia 0,26 y días entre compras 0,22; el segmento (máximo 0,011) y el país (0,001) casi no pesan.
- **Dentro de cada segmento (simulación sobre el 20 % de prueba):** al elegir el 20 % de mayor riesgo de cada segmento, el modelo encuentra 2,7 veces más abandonos que el azar en los VIP, 1,6 en los Leales, 1,3 en los Nuevos y 1,1 en los En riesgo. No se contactó a nadie.
- **Decisión: sin «valor en riesgo».** Se exploró un cálculo de valor en riesgo (probabilidad × gasto previo) y se sacó del informe y del cuaderno por decisión del equipo.
- **Control de cifras:** la última celda del cuaderno compara las cifras del informe con las del código.
