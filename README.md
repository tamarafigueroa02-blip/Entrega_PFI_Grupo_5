# Segmentación de clientes y predicción de churn · Grupo 5

Proyecto Final Integrador de la Diplomatura en Ciencia de Datos y Análisis Avanzado (UTN BA).

Un e-commerce británico de artículos de regalo, con clientes mayoristas y minoristas, quiere personalizar sus campañas y anticipar qué clientes van a dejar de comprar. El proyecto tiene dos fases:

- **Fase A · Segmentación (K-Means sobre RFM):** ¿quiénes son los clientes? Se compara contra la segmentación tradicional por quintiles RFM.
- **Fase B · Predicción de abandono (churn):** ¿a quién contactar primero? Se comparan dos referencias (regla por Recencia y regresión logística) contra XGBoost.

Metodología: CRISP-DM.

## Datos

[Online Retail II](https://archive.ics.uci.edu/dataset/502/online+retail+ii), del UCI Machine Learning Repository: 1.067.371 transacciones entre el 01/12/2009 y el 09/12/2011.

## Cómo ejecutarlo

**En Google Colab**
1. Abrí `TP_Final_Grupo5_Segmentacion_y_Churn.ipynb` en Colab.
2. Elegí *Entorno de ejecución → Ejecutar todas*.
3. Si el archivo `online_retail_II.csv` no está cargado, el cuaderno lo descarga desde UCI. También podés subirlo a mano al panel de archivos.

**En una PC local**
```bash
pip install -r requirements.txt
jupyter notebook TP_Final_Grupo5_Segmentacion_y_Churn.ipynb
```
Ubicá `online_retail_II.csv` en la carpeta `Dataset/` (dentro de esta carpeta o en la de arriba). Si no está, el cuaderno lo descarga.

El cuaderno se ejecuta de arriba hacia abajo, usa rutas relativas y fija la semilla aleatoria (`RANDOM_STATE = 42`), así que los resultados se reproducen igual en cada ejecución (con las versiones de `requirements.txt`, tarda unos minutos).

## Cómo se relaciona con el informe

El cuaderno es la fuente de todas las cifras, tablas y figuras del informe final:

- Las figuras llevan el **mismo número que en el informe** (`fig05_busqueda_k.png` es la Figura 5, `figA1_embudo_limpieza.png` es la Figura A1). Las que empiezan con `figC` son complementarias y no están en el informe.
- Al final del cuaderno hay un **mapa informe ↔ cuaderno** que indica, para cada figura y tabla del informe, en qué sección del cuaderno se calcula y en qué archivo queda guardada.
- La última celda es un **control**: compara las cifras que cita el informe con las que salen del código y avisa si alguna no coincide.

## Estructura

| Archivo o carpeta | Contenido |
|---|---|
| `TP_Final_Grupo5_Segmentacion_y_Churn.ipynb` | Cuaderno principal, organizado por etapas CRISP-DM, con las salidas ya ejecutadas |
| `data_log.md` | Registro de cada decisión, con su motivo y su cifra |
| `diccionario_de_datos.md` | Qué significa cada columna, original o creada |
| `figures/` | Gráficos del cuaderno (numerados como en el informe) |
| `results/` | Tablas de resultados en CSV y `f2_resumen.json` |
| `requirements.txt` | Librerías necesarias |
| `Dataset/` | Datos de entrada (no se suben al repositorio) |

## Privacidad

El identificador de cliente se reemplaza por un seudónimo (`CLI_0001`, …) en orden aleatorio antes de cualquier análisis, y la tabla de equivalencias no se guarda. El repositorio no guarda datos a nivel de cliente.
