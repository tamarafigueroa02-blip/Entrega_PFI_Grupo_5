# Segmentación de clientes con RFM y clustering

Proyecto Final Integrador del **Grupo 5**, de la Diplomatura en Ciencia de Datos y Análisis Avanzado.

**Integrantes:** Avaro Lucena, Figueroa Tamara, Méndez Julieta, Milicich Marcos y Robledo Maximiliano.

## Objetivo

Dividimos a los clientes de un comercio online británico en **4 segmentos accionables** (VIP, En riesgo, Activos recientes e Inactivos de bajo valor). Para eso usamos las variables RFM (Recencia, Frecuencia y Monetario) y algoritmos de clustering. La idea es que marketing pueda aplicar una acción distinta a cada segmento.

## Resultados principales

| Modelo | Silhouette (holdout) | Davies-Bouldin (holdout) |
|---|---|---|
| Quintiles RFM (benchmark) | 0,226 | 1,536 |
| **K-Means (elegido)** | **0,356** | **0,929** |
| Jerárquico Ward | 0,350 | 0,934 |
| DBSCAN | 0,287 | 0,962 |

El segmento **VIP** reúne al 20% de los clientes y genera el **74% de los ingresos**.

## Datos

Usamos el dataset **Online Retail II**, del UCI Machine Learning Repository (licencia CC BY 4.0):
https://archive.ics.uci.edu/dataset/502/online+retail+ii

El archivo no está incluido en el repositorio por su tamaño (unos 45 MB). Para usarlo, descargá `online_retail_II.xlsx` y guardalo en la misma carpeta que el notebook.

## Cómo ejecutarlo localmente

1. Cloná el repositorio o descargalo como ZIP:
   ```bash
   git clone https://github.com/tamarafigueroa02-blip/Entrega_PFI_Grupo_5.git
   cd Entrega_PFI_Grupo_5
   ```
2. Instalá las dependencias. Hace falta Python 3.10 o una versión más nueva:
   ```bash
   pip install -r requirements.txt
   ```
3. Copiá `online_retail_II.xlsx` a esta carpeta.
4. Abrí el notebook y ejecutá todas las celdas:
   ```bash
   jupyter notebook RFM_Clustering_TFI.ipynb
   ```

El notebook también funciona en **Google Colab**: si detecta que está en Colab, monta Google Drive y lee el archivo desde ahí.

La ejecución completa tarda unos minutos, y la mayor parte de ese tiempo se va en leer el Excel. Al terminar se genera el archivo `segmentos_clientes.csv`, con el segmento asignado a cada cliente.

## Estructura del notebook (CRISP-DM)

1. Comprensión del negocio
2. Comprensión y exploración de los datos
3. Limpieza y construcción de la tabla RFM
4. Modelado y validación (80% desarrollo / 20% holdout)
5. Evaluación y elección del modelo
6. Interpretación para el negocio
7. Ética, privacidad y limitaciones

Todos los resultados son reproducibles porque usamos una semilla fija (`RANDOM_STATE = 42`).
