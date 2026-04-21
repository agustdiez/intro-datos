---
title: Preprocesamiento Properati
layout: default
parent: EDA - Preprocesamiento
nav_order: 1
has_toc: true
---

# Preprocesamiento de Datos: Caso de Aplicación con Dataset de Properati
{: .no_toc }

## Tabla de Contenidos
{: .no_toc .text-delta }

1. TOC
{:toc}

---

## Introducción

El siguiente artículo actúa de guía sobre la notebook ubicada dentro de los ejercicios de la materia. Ver [Notebooks](../notebooks/index.md) para descarga y probar cosas distintas a las planteadas acá. La idea es ampliar conceptos vistos en clase, para que ganen mayor conocimiento de tipo de variables y como lidiar con ellas.

El preprocesamiento de datos constituye una etapa fundamental en cualquier proyecto de ciencia de datos. Se basa principalmente en llevar nuestros datos a una estructura predefinida y que servirá de entrada para la aplicación de nuestros modelos.

Utilizaremos como ejemplo práctico el dataset de propiedades de Properati, donde cada instancia representa una propiedad (publicación). Quien haya buscado alquiler sabrá que tendremos por delante todo lo siguiente ya visto en la clase: valores faltantes, variables georreferenciadas, información temporal, datos de texto no estructurado y valores atípicos.

La pregunta que motivará todo este análisis es asimilable a que alguien nos diga:
> *Nos interesa poder predecir el valor de propiedades ubicadas en el AMBA (casas o departamentos) para presentar una herramienta de consulta que deje al usuario estimar el valor de su propiedad en función de sus características*

Este dataset es la última versión pública del mismo, que data de 2021. [Properati](https://www.properati.com.ar/)

---

## Descarga y Carga del Dataset

Comencemos descargando los archivos. Para eso, importamos las librerías que estaremos usando.

```python
import pandas as pd
import numpy as np
import gdown

# Descarga del dataset comprimido
!gdown 1-B93jjaUuZpxhsJsLJzL9CXNLQ8eIY5w

# Descompresión y carga
properati = pd.read_csv('ar_properties.csv.gz', 
                        compression='gzip',
                        header=0, 
                        sep=',', 
                        quotechar='"')

# Crear copia de trabajo
df = properati.copy()
```

**Enlace al dataset:** [Properati Julio 2021](https://github.com/daterxs/datos-imobiliario/releases/tag/properati-julio-2021)

---

## Exploración Inicial del Dataset

Primero que nada, debemos comprender la estructura y naturaleza de nuestros datos. Esta fase de exploración nos permite identificar tipos de variables, detectar inconsistencias, evaluar la proporción de valores faltantes y definir el alcance del problema a resolver.

```python
# Información general del dataset
df.info()

# Muestra aleatoria de registros
df.sample(8)

# Exploración de valores únicos por columna
for column in df.columns:
    print(f'\n--- Columna: {column} ---')
    print(df[column].value_counts())
```

### Variables del Dataset

El dataset original contiene múltiples campos que podemos clasificar en:

**Variables para identificar propiedad:**
- `id`: identificador único de cada publicación
- `ad_type`: tipo de anuncio (constante en todos los registros)

**Variables temporales:**
- `start_date`: fecha de inicio de la publicación
- `end_date`: fecha de finalización de la publicación
- `created_on`: fecha de creación (redundante con start_date). Revisen que se trate de la misma columna

**Variables georreferenciadas:**
- `lat`: latitud del inmueble
- `lon`: longitud del inmueble
- `l1` a `l6`: niveles de localización jerárquica (país, provincia, localidad, barrio, etc.)

**Variables categóricas:**
- `property_type`: tipo de propiedad (Departamento, Casa, PH, etc.)
- `operation_type`: tipo de operación (Venta, Alquiler)
- `currency`: moneda de la transacción (USD, ARS, etc.)

**Variables numéricas:**
- `price`: precio de la propiedad
- `surface_total`: superficie total en m²
- `surface_covered`: superficie cubierta en m²
- `rooms`: cantidad de ambientes/habitaciones

**Variables de texto libre:**
- `title`: título de la publicación
- `description`: descripción detallada del inmueble

---

## Definición del Objetivo y Alcance

Para este ejercicio, estableceremos un objetivo claro que guiará todas nuestras decisiones de preprocesamiento:

**Objetivo:** Predecir el precio de venta de propiedades ubicadas en CABA y AMBA que sean departamentos, casas, PH o casas de campo, expresadas en dólares estadounidenses.

### Filtrado Inicial

Con base en nuestro objetivo, aplicamos los siguientes filtros al dataset:

```python
# Filtrado por tipo de propiedad
df = df[df['property_type'].isin(['Departamento', 'Casa', 'PH', 'Casa de campo'])]

# Filtrado por ubicación (CABA y AMBA)
df = df[df['l1'] == 'Argentina']
df = df[df['l2'].isin(['Capital Federal', 
                        'Bs.As. G.B.A. Zona Norte',
                        'Bs.As. G.B.A. Zona Sur', 
                        'Bs.As. G.B.A. Zona Oeste'])]

# Filtrado por tipo de operación y moneda
df = df[df['operation_type'] == 'Venta']
df = df[df['currency'] == 'USD']

# Reiniciar índices
df = df.reset_index(drop=True)
```

Esta selección nos permite ya trabajar con todas las propiedades que nos interesan.

{: .important}
> Esta definición es válida si todas las decisiones tomadas son correctas. 
> ¿Estamos seguros que todas las propiedades están bien imputadas en lugar?
> ¿Tenemos propiedades en otras monedas? ¿No podríamos aplicar una conversión?

---

## Análisis de Valores Faltantes

Una vez definido nuestro alcance, procedemos a analizar la presencia de valores faltantes en cada columna. Este paso es crucial porque determinará qué estrategias de imputación o eliminación aplicaremos. Retomando lo visto para bucles, iteremos por todas las columnas del dataset y saquemos un print:

```python
# Nulos por columna
for columna in df.columns:
    nulos = df[columna].isna().sum()
    proporcion = nulos / len(df)
    print(f'Columna: {columna}')
    print(f'  Nulos: {nulos} ({proporcion*100:.2f}%)')
```

### Decisiones sobre Valores Faltantes

**Eliminación de registros sin precio:**
Los registros sin precio no aportan valor para un modelo de predicción de precios. Aunque en algunos casos esta información podría extraerse de los campos de texto (título o descripción), por simplicidad optamos por eliminar estos registros.

```python
df = df.dropna(subset=['price'])
```

**Variables con alta proporción de faltantes:**
Columnas como `rooms`, `surface_total` y `surface_covered` presentan valores faltantes. En lugar de eliminar estos registros, aprovecharemos las variables de texto para intentar extraer esta información mediante expresiones regulares.

---

## Eliminación de Variables Redundantes

Algunas columnas no aportan información útil para el modelado o son redundantes con otras variables. Entendamos por ahora como redundante a columnas que san muy similares a otras, y por lo tanto no aporten información adicional. Identificamos y eliminamos:

- `ad_type`: todos los registros tienen el mismo valor
- `created_on`: idéntica a `start_date`
- `l6`: nivel de jerarquía que no tiene nada.

```python
df.drop(['ad_type', 'created_on', 'l6'], axis=1, inplace=True)
```

---

## Tratamiento de Variables Temporales

Las variables temporales requieren un tratamiento especial. En nuestro dataset, `start_date` y `end_date` contienen información sobre la vigencia de las publicaciones, pero presentan inconsistencias.

### Detección de Anomalías

Al explorar los valores de `end_date`, identificamos registros con fecha `9999-12-31`, lo cual indica un error de ingreso. Estos registros probablemente corresponden a publicaciones que estaban activas al momento de la extracción del dataset.

```python
# Explorar años en end_date
df['end_date'].str[:4].value_counts()
```

### Creación de Variables Derivadas

A partir de las fechas, creamos dos variables nuevas que capturan información relevante:

**1. Variable de estatus:**
Indica si la publicación estaba activa (1) o finalizada (0) al momento de la extracción.

```python
# Crear columna de estatus
columna_indice = df.columns.get_loc('end_date') + 1
df.insert(columna_indice, 'estatus', 
          np.where(df['end_date'].str[:4] == '9999', 1, 0))
```

{: .note}
> `np.where` es una función muy asimilable al IF() de Excel. [Ver cómo usarla](https://numpy.org/doc/2.2/reference/generated/numpy.where.html)

**2. Días activa:**
Para publicaciones finalizadas, calculamos la cantidad de días que estuvo activa la publicación.

```python
# Convertir a tipo datetime
df['start_date'] = pd.to_datetime(df['start_date'], 
                                   format='%Y-%m-%d', 
                                   errors='coerce')
df['end_date'] = pd.to_datetime(df['end_date'], 
                                 format='%Y-%m-%d', 
                                 errors='coerce')

# Calcular días activa
columna_indice = df.columns.get_loc('estatus') + 1
df.insert(columna_indice, 'dias_activa', 
          np.where(df['estatus'] == 0, 
                   (df['end_date'] - df['start_date']).dt.days, 
                   None))
```

Estas nuevas variables capturan información temporal sin depender de las fechas absolutas, haciéndolas más útiles para el modelado.

### ¿Cómo se suelen tratar las variables temporales?

Las variables temporales en pandas son del tipo `datetime64`. Tenemos varias cosas a tener en cuenta con estas variables. Primero veremos como transformarlas efectivamente a este tipo de objetos.

#### Conversión a datetime

El primer paso siempre es convertir columnas de texto a tipo fecha:

```python
df['fecha'] = pd.to_datetime(df['fecha'], format='%Y-%m-%d', errors='coerce')
```

El parámetro `errors='coerce'` convierte valores que no puedan parsearse en `NaT` (equivalente a `NaN` para fechas) en lugar de lanzar un error.

{. :important}
> El formato del string de fecha puede validarse acá: [strftime(); strptime()](https://docs.python.org/3/library/datetime.html#strftime-and-strptime-behavior)

#### Extracción de componentes

Una vez convertida la columna, con `.dt` pueden extraer componentes individuales propios de la fecha, ya que ahora se trata de un tipo de dato "inteligente" en ese sentido:

```python
df['año']       = df['fecha'].dt.year
df['mes']       = df['fecha'].dt.month
df['dia']       = df['fecha'].dt.day
df['dia_semana'] = df['fecha'].dt.dayofweek   # 0 = lunes, 6 = domingo
df['trimestre'] = df['fecha'].dt.quarter
```

Estos atributos pueden ser útiles en muchos casos para ingeniería de atributos, si quieren desglosar la fecha en algo más grande y mapearla en categorías.

#### Diferencias entre fechas

La resta entre dos columnas datetime devuelve un objeto `timedelta`, del cual se puede extraer la duración en distintas unidades:

```python
df['dias_entre'] = (df['fecha_fin'] - df['fecha_inicio']).dt.days
```

Esto es exactamente lo que hicimos para calcular `dias_activa` en el ejemplo arriba.

#### Variables de referencia

Cuando el momento en que ocurre un evento importa más que la fecha absoluta, conviene calcular la distancia respecto a un punto de referencia fijo:

```python
fecha_corte = pd.Timestamp('2021-07-01')
df['dias_desde_inicio'] = (fecha_corte - df['start_date']).dt.days
```

Este tipo de feature es útil si la antigüedad de un registro tiene valor para predecir una variable (por ejemplo, autos usados y años desde lanzamiento).

#### Interpolación en series temporales

Cuando el índice del dataframe es una fecha y los datos tienen una frecuencia regular (diaria, mensual, etc.), los valores faltantes pueden imputarse por interpolación:

```python
df_serie = df.set_index('fecha').sort_index()
df_serie['valor'] = df_serie['valor'].interpolate(method='time')
```

A diferencia de rellenar con media o mediana, la interpolación temporal respeta la tendencia local de la serie.

---

## Tratamiento de Variables Georreferenciadas

Las coordenadas geográficas (`lat`, `lon`) representan información espacial valiosa, que conociendo poco el mercado inmobiliario uno sabe que el principal factor es dónde está ubicado el inmueble. Utilizamos GeoPandas para enriquecer esta información y visualizarla en contexto.

### Instalación de Dependencias

```python
!pip install contextily geopandas

import geopandas as gpd
import matplotlib.pyplot as plt
import contextily as ctx
from shapely.geometry import Point
```

### Carga de Geometrías de Referencia

Descargamos archivos shapefile con las delimitaciones de barrios de CABA y partidos del AMBA para contextualizar nuestros datos:

```python
# Descargar archivos de referencia
!gdown 1YSlGwL22OqcZ2_Vu5VmKsu-cl3UdLihU  # Barrios CABA
!gdown 1-RTA3JH0oxJDUt1jdCi7dWc9EYgljwZx  # Partidos AMBA
```

{: .note}
>Un shapefile (.shp) es un formato vectorial de almacenamiento digital, desarrollado por Esri, utilizado en sistemas de información geográfica (SIG) para guardar la ubicación de elementos (puntos, líneas, polígonos) y sus atributos asociados


### Visualización Espacial

Convertimos nuestro DataFrame en un GeoDataFrame y lo visualizamos sobre mapas base:

```python
# Crear geometría de puntos
geometry = [Point(xy) for xy in zip(df['lon'], df['lat'])]
gdf = gpd.GeoDataFrame(df, geometry=geometry, crs='EPSG:4326')

# Cargar capas de referencia
barrios_caba = gpd.read_file('barrios_caba.shp')
partidos_amba = gpd.read_file('partidos_amba.shp')

# Visualización
fig, ax = plt.subplots(figsize=(12, 10))
partidos_amba.plot(ax=ax, color='lightgray', edgecolor='black', alpha=0.5)
barrios_caba.plot(ax=ax, color='white', edgecolor='gray', alpha=0.7)
gdf.plot(ax=ax, markersize=1, color='red', alpha=0.3)
ctx.add_basemap(ax, crs=gdf.crs.to_string(), source=ctx.providers.CartoDB.Positron)
plt.title('Distribución de Propiedades en CABA y AMBA')
plt.show()
```

### Feature Engineering Espacial

Podemos crear variables derivadas que capturen información geoespacial relevante:

- **Distancia al centro:** calculada desde cada propiedad hasta un punto de referencia. No está hecho acá, pero podríamos sumar datos de estaciones de subte y calcular su distancia entre puntos.
- **Cluster espacial:** agrupación de propiedades cercanas
- **Barrio/Partido:** asignación mediante operación de spatial join

Estas variables transforman coordenadas brutas en features interpretables y útiles para el modelo.

### ¿Qué es SIG?

Un **Sistema de Información Geográfica (SIG)** es un conjunto de herramientas que permite *capturar, almacenar, analizar y visualizar datos con una componente espacial*. En ciencia de datos los encontramos cada vez que trabajamos con coordenadas, polígonos que delimiten áreas o mapas.

#### Sistemas de referencia de coordenadas (CRS)

Todo dato geográfico necesita un sistema de referencia que indique cómo se proyectan las coordenadas sobre la superficie terrestre. Los dos más comunes en la práctica son:

- **WGS84 / EPSG:4326:** sistema global basado en latitud y longitud en grados. Es el que usan GPS, Google Maps y la mayoría de los datasets abiertos. Las coordenadas de nuestro dataset están en este sistema.
- **Proyecciones métricas (ej. POSGAR para Argentina):** convierten la superficie esférica a un plano en metros. Son necesarias cuando queremos calcular distancias o áreas con precisión, ya que operar con grados no da resultados correctos.

```python
# Ver el CRS de un GeoDataFrame
print(gdf.crs)          # EPSG:4326

# Reproyectar a sistema métrico para calcular distancias
gdf_metro = gdf.to_crs(epsg=22185) #Esto sería Posgar 94-Faja 5
```

{: .important}
> Los EPSG (European Petroleum Survey Group) se pueden ver en [epsg.io](https://epsg.io/?q=)


#### Formatos de datos geoespaciales

| Formato | Extensión | Descripción |
|---------|-----------|-------------|
| Shapefile | `.shp` + archivos auxiliares | Estándar histórico de ESRI. Requiere múltiples archivos para funcionar. Ubicar todos en la misma carpeta! |
| GeoJSON | `.geojson` | Basado en JSON, un solo archivo, ideal para web y APIs. |
| GeoPackage | `.gpkg` | Formato moderno basado en SQLite, reemplaza al shapefile. |
| WKT / WKB | — | Representación textual o binaria de geometrías, común en bases de datos. |

En Python, GeoPandas puede leer todos estos formatos con `gpd.read_file()`.

#### Tipos de geometría

Los elementos espaciales se representan con tres tipos de geometría básicos:

- **Point:** una ubicación (ej. coordenada de una propiedad, puntos de cateos, paradas de colectivo)
- **LineString:** una secuencia de puntos que forma una línea (ej. traza de una calle, un río, un recorrido de un colectivo, etc.)
- **Polygon:** una región cerrada (ej. un barrio, un partido)

```python
from shapely.geometry import Point, LineString, Polygon

punto    = Point(-58.4173, -34.6118)              # Obelisco
linea    = LineString([(-58.37, -34.60), (-58.40, -34.62)])
poligono = Polygon([(-58.37, -34.60), (-58.40, -34.60), 
                    (-58.40, -34.63), (-58.37, -34.63)])
```

#### Operaciones espaciales frecuentes

Las más útiles en un contexto de feature engineering son:

**Spatial join:** asigna a cada punto los atributos del polígono que lo contiene. Fue la operación que usamos para asignarle a cada propiedad su barrio o partido.

```python
# Asignar a cada propiedad el barrio de CABA en que se ubica
resultado = gpd.sjoin(gdf, barrios_caba[['geometry', 'BARRIO']], how='left')
```

**Cálculo de distancias:** distancia entre dos puntos o entre un punto y una geometría. Requiere CRS métrico.

```python
# Distancia al Obelisco (en metros)
obelisco = gpd.GeoDataFrame(geometry=[Point(-58.4173, -34.6118)], crs='EPSG:4326')
obelisco_metro = obelisco.to_crs(epsg=22185)

gdf_metro['dist_obelisco'] = gdf_metro.geometry.distance(obelisco_metro.geometry[0])
```

**Buffer:** genera un área de influencia alrededor de una geometría. Con esto pasamos de un punto por ejemplo a polígonos.

```python
# Área de 500m alrededor de cada estación de subte
estaciones_metro = estaciones.to_crs(epsg=22185)
area_influencia  = estaciones_metro.buffer(500)   # 500 metros
```

#### Recursos útiles

- [GeoPandas — documentación oficial](https://geopandas.org/en/stable/)
- [datos.gob.ar](https://datos.gob.ar) — portal de datos abiertos con shapefiles de Argentina (barrios, partidos, provincias, puntos de interés)
- [QGIS](https://qgis.org) — software de escritorio open source para explorar y editar datos geoespaciales sin código

---

## Tratamiento de Variables de Texto con Expresiones Regulares

Los campos de texto (`title` y `description`) contienen información valiosa que no está estructurada. Mediante expresiones regulares (regex), podemos extraer datos específicos como cantidad de ambientes y superficie.

### Fundamentos de Expresiones Regulares

Las expresiones regulares son patrones que permiten buscar y extraer texto que cumple ciertas características. En nuestro caso, buscamos patrones numéricos acompañados de palabras clave.

**Conceptos clave:**
- `\d`: representa cualquier dígito (0-9)
- `+`: indica una o más repeticiones del elemento anterior
- `\b`: límite de palabra (word boundary)
- `()`: grupos de captura para extraer partes específicas del patrón
- `|`: operador OR para múltiples alternativas

### Ejemplos de Patrones Utilizados

```python
import re

# Patrón para detectar ambientes
# Busca: número seguido de "amb", "ambiente", "ambientes", "dormitorio", etc.
patron_ambientes = r'(\d+)\s*(amb|ambiente|ambientes|dormitorio|dormitorios|habitacion|habitaciones)'

# Patrón para superficie
# Busca: número seguido de "m2", "m²", "metros", etc.
patron_superficie = r'(\d+)\s*(m2|m²|metros cuadrados|metros|mts)'

# Ejemplo de aplicación
texto = "Hermoso departamento de 3 ambientes, 75 m2"
ambientes = re.search(patron_ambientes, texto, re.IGNORECASE)
superficie = re.search(patron_superficie, texto, re.IGNORECASE)

if ambientes:
    print(f"Ambientes encontrados: {ambientes.group(1)}")
if superficie:
    print(f"Superficie encontrada: {superficie.group(1)}")
```

### Extracción de Ambientes

Definimos una función que aplica múltiples patrones para capturar diferentes formas de expresar la cantidad de ambientes:

```python
def extraer_ambientes(texto):
    """
    Extrae cantidad de ambientes de un texto usando múltiples patrones.
    Retorna None si no encuentra coincidencia.
    """
    if pd.isna(texto):
        return None
    
    texto = str(texto).lower()
    
    # Patrones ordenados por especificidad
    patrones = [
        r'(\d+)\s*ambientes?\b',
        r'(\d+)\s*amb\b',
        r'(\d+)\s*dormitorios?\b',
        r'(\d+)\s*habitaciones?\b',
        r'\b(\d+)\s*amb\b'
    ]
    
    for patron in patrones:
        match = re.search(patron, texto)
        if match:
            return int(match.group(1))
    
    return None

# Aplicar a las columnas de texto
df['ambientes_titulo'] = df['title'].apply(extraer_ambientes)
df['ambientes_descripcion'] = df['description'].apply(extraer_ambientes)

# Combinar información: priorizar rooms, luego título, luego descripción
df['ambientes_final'] = df['rooms'].fillna(
    df['ambientes_titulo'].fillna(df['ambientes_descripcion'])
)
```

### Extracción de Superficie

De manera similar, extraemos información sobre la superficie total de las propiedades:

```python
def extraer_superficie(texto):
    """
    Extrae superficie en m² de un texto.
    Maneja múltiples formatos: m2, m², metros cuadrados, etc.
    """
    if pd.isna(texto):
        return None
    
    texto = str(texto).lower()
    
    patrones = [
        r'(\d+)\s*m2\b',
        r'(\d+)\s*m²',
        r'(\d+)\s*metros\s*cuadrados',
        r'(\d+)\s*mts?\b',
        r'superficie[:\s]+(\d+)'
    ]
    
    for patron in patrones:
        match = re.search(patron, texto)
        if match:
            return int(match.group(1))
    
    return None

# Aplicar y combinar
df['superficie_titulo'] = df['title'].apply(extraer_superficie)
df['superficie_descripcion'] = df['description'].apply(extraer_superficie)

df['superficie_final'] = df['surface_total'].fillna(
    df['superficie_titulo'].fillna(df['superficie_descripcion'])
)
```

### Validación y Limpieza

Es importante validar los valores extraídos para evitar datos ilógicos:

```python
# Filtrar valores extremos o ilógicos
df = df[df['ambientes_final'] <= 10]  # Máximo razonable de ambientes
df = df[(df['superficie_final'] >= 20) & (df['superficie_final'] <= 500)]
```

### Otros ejemplos de RegEx...

#### Amenities

```python
# Objetivo: detectar presencia de amenities como variables binarias
# Input:    "Edificio con pileta, gimnasio y seguridad 24hs"

amenities = {
    'pileta'   : r'\b(?:pileta|piscina|pool)\b',
    'gimnasio' : r'\b(?:gimnasio|gym)\b',
    'cochera'  : r'\b(?:cochera|garage|garaje)\b',
    'seguridad': r'\bseguridad\b',
    'quincho'  : r'\bquincho\b',
    'balcon'   : r'\bbalc[oó]n\b',
}

texto = "Edificio con pileta, gimnasio y seguridad 24hs"
for amenity, patron in amenities.items():
    encontrado = bool(re.search(patron, texto, re.IGNORECASE))
    print(f"{amenity}: {int(encontrado)}")
# pileta: 1 / gimnasio: 1 / cochera: 0 / seguridad: 1 / quincho: 0 / balcon: 0
```

#### Precios

```python
# Objetivo: extraer valor numérico de un precio
# Input:    "Precio: $125.000" / "USD 85,000" / "u$s 90000"

patron_precio = r'(?:u\$s|usd|\$)\s*([\d.,]+)'

textos = ["Precio: $125.000", "USD 85,000", "u$s 90000"]
for t in textos:
    m = re.search(patron_precio, t, re.IGNORECASE)
    if m:
        print(m.group(1))   # '125.000', '85,000', '90000'
```


#### Herramientas útiles para desarrollar y testear patrones

- [regex101.com](https://regex101.com) — editor interactivo con explicación paso a paso de cada parte del patrón
- [pythex.org](https://pythex.org)

---

## Detección y Tratamiento de Valores Atípicos (Outliers)

Los valores atípicos pueden distorsionar el análisis y afectar el rendimiento de modelos predictivos. Su detección y tratamiento debe realizarse con criterio, considerando el contexto del problema.

### Métodos Univariados

**Z-Score:**
Identifica valores que se alejan de la media más de un número determinado de desviaciones estándar.

```python
from scipy import stats

# Calcular z-scores
z_scores = np.abs(stats.zscore(df['price']))

# Filtrar valores con |z| > 3
df_sin_outliers = df[z_scores < 3]
```

**IQR (Interquartile Range):**
Método que nos elimina valores por fuera del IQR (los bigotes del boxplot)

```python
Q1 = df['price'].quantile(0.25)
Q3 = df['price'].quantile(0.75)
IQR = Q3 - Q1

# Definir límites
limite_inferior = Q1 - 1.5 * IQR
limite_superior = Q3 + 1.5 * IQR

# Filtrar
df_sin_outliers = df[(df['price'] >= limite_inferior) & 
                      (df['price'] <= limite_superior)]
```

### Métodos Multivariados

**Isolation Forest:**
Algoritmo de machine learning que identifica anomalías en múltiples dimensiones.

```python
from sklearn.ensemble import IsolationForest

# Seleccionar variables numéricas
features = ['price', 'ambientes_final', 'superficie_final', 'lat', 'lon']
X = df[features].dropna()

# Entrenar Isolation Forest
iso_forest = IsolationForest(contamination=0.05, random_state=42)
outliers = iso_forest.fit_predict(X)

# -1 indica outlier, 1 indica normal
df_sin_outliers = df.loc[X.index[outliers == 1]]
```

### Decisión Contextual

En datos inmobiliarios, los "outliers" pueden ser propiedades de lujo por ejemplo. La decisión de eliminarlos debe basarse en:
- Objetivo o pregunta que perseguimos (¿es atípico predecir estas propiedades en CABA?)
- Cantidad de datos disponibles

---

## Imputación de Valores Faltantes con MICE

MICE (Multiple Imputation by Chained Equations) es una técnica avanzada que imputa valores faltantes considerando las relaciones entre múltiples variables.

### Funcionamiento de MICE

El algoritmo opera en iteraciones:
1. Inicializa valores faltantes con la media/moda
2. Para cada variable con faltantes:
   - Ajusta un modelo usando las demás variables como predictores
   - Imputa los valores faltantes basándose en ese modelo
3. Repite el ciclo hasta lograr convergencia

```python
from sklearn.experimental import enable_iterative_imputer
from sklearn.impute import IterativeImputer

# Seleccionar variables para imputación
variables_a_imputar = ['ambientes_final', 'superficie_final', 'dias_activa']

# Configurar MICE
imputer = IterativeImputer(max_iter=10, random_state=42)

# Aplicar imputación
df[variables_a_imputar] = imputer.fit_transform(df[variables_a_imputar])
```

### Ventajas de MICE

- Preservamos relaciones entre variables
- Es más robusto que una imputación simple ya que vemos las otras variables

---

## Normalización de Variables

La normalización transforma variables a una escala común, esencial para algoritmos sensibles a magnitudes, como veremos más adelante en aprendizaje no supervisado.

### Min-Max Scaling

Transforma valores al rango [0, 1]:

```python
from sklearn.preprocessing import MinMaxScaler

scaler = MinMaxScaler()

variables_numericas = ['price', 'superficie_final', 'ambientes_final']
df[variables_numericas] = scaler.fit_transform(df[variables_numericas])
```

### Standardización (Z-Score)

Transforma a media 0 y desviación estándar 1:

```python
from sklearn.preprocessing import StandardScaler

scaler = StandardScaler()
df[variables_numericas] = scaler.fit_transform(df[variables_numericas])
```

### Cuándo Usar Cada Método

- **Min-Max:** cuando conocemos límites naturales de las variables
- **Standardization:** cuando las variables siguen distribución normal

---

## Variables Dummy (One-Hot Encoding)

Las variables categóricas deben transformarse en representaciones numéricas para muchos algoritmos de aprendizaje automático que veremos más adelante. Un modelo no lee automáticamente que algo es `VILLA CRESPO` y otra instancia es `PATERNAL`

```python
# Ejemplo con property_type
df_con_dummies = pd.get_dummies(df, 
                                 columns=['property_type', 'l2'], 
                                 prefix=['tipo', 'zona'],
                                 drop_first=True,  # Evita multicolinealidad
                                 dtype=int)
```

**Comentario:**
- `drop_first=True` elimina una categoría para evitar redundancia.



---

## Evaluación del Impacto del Preprocesamiento

Para validar que nuestro trabajo de feature engineering fue efectivo, comparamos el rendimiento de un modelo simple con y sin preprocesamiento.

### Modelo con Dataset Procesado

```python
from sklearn.model_selection import train_test_split
from sklearn.ensemble import RandomForestRegressor
from sklearn.metrics import mean_absolute_error, r2_score

# Separar features y target
X = df_procesado.drop(['price'], axis=1)
y = df_procesado['price']

# Split
X_train, X_test, y_train, y_test = train_test_split(
    X, y, test_size=0.2, random_state=42
)

# Modelo
modelo = RandomForestRegressor(n_estimators=100, random_state=42)
modelo.fit(X_train, y_train)

# Predicción
y_pred = modelo.predict(X_test)

# Métricas
mae = mean_absolute_error(y_test, y_pred)
r2 = r2_score(y_test, y_pred)

print(f"MAE (dataset procesado): ${mae:,.2f}")
print(f"R² (dataset procesado): {r2:.3f}")
```

### Modelo con Dataset Crudo

```python
# Repetir con datos sin procesar
X_crudo = df_original[variables_basicas].dropna()
y_crudo = df_original.loc[X_crudo.index, 'price']

# Entrenar y evaluar
modelo_crudo = RandomForestRegressor(n_estimators=100, random_state=42)
modelo_crudo.fit(X_crudo_train, y_crudo_train)
y_pred_crudo = modelo_crudo.predict(X_crudo_test)

mae_crudo = mean_absolute_error(y_crudo_test, y_pred_crudo)
r2_crudo = r2_score(y_crudo_test, y_pred_crudo)

print(f"MAE (dataset crudo): ${mae_crudo:,.2f}")
print(f"R² (dataset crudo): {r2_crudo:.3f}")
```

### Comparación de Resultados

```python
mejora_mae = ((mae_crudo - mae) / mae_crudo) * 100
mejora_r2 = ((r2 - r2_crudo) / r2_crudo) * 100

print(f"\nMejoras obtenidas:")
print(f"  MAE: {mejora_mae:.1f}% de reducción")
print(f"  R²: {mejora_r2:.1f}% de incremento")
```

---

## Bibliografía y Recursos

- **Dataset:** [Properati - Datos Inmobiliarios Argentina](https://github.com/daterxs/datos-imobiliario)
- **Pandas:** [Documentación oficial](https://pandas.pydata.org/docs/)
- **Scikit-learn:** [Preprocessing guide](https://scikit-learn.org/stable/modules/preprocessing.html)
- **GeoPandas:** [User guide](https://geopandas.org/en/stable/docs/user_guide.html)
- **Expresiones regulares:** [Python re module](https://docs.python.org/3/library/re.html)
- **MICE:** Azur et al. (2011) "Multiple imputation by chained equations"

