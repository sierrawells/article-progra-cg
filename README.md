# Pruebas de dependencia espacial en homicidios en México

## Descripción

Artículo de la clase de Programación (Geointeligencia Computacional) de la
Maestría en Ciencias de Información Geoespacial. Replica la metodología de
Anselin, Bera, Florax y Yoon (1996), *Simple diagnostic tests for spatial
dependence* (Regional Science and Urban Economics 26: 77–104), con datos de
homicidios en México: se estima un modelo por mínimos cuadrados ordinarios
(MCO) de la tasa de homicidios de 2018 por **municipio** (con la tasa suavizada
por el método bayesiano empírico), con predictores socioeconómicos y
demográficos del Censo de Población y Vivienda 2010 y el número de grupos
criminales con presencia violenta en 2017. A partir de sus residuos se prueba
si hay dependencia espacial en el error o en la variable dependiente (rezago
espacial) con las pruebas de multiplicadores de Lagrange (LM) estándar y
robustas.

Los residuos tienen autocorrelación espacial positiva (I de Moran = 0.37). Las
pruebas LM estándar detectan tanto rezago como error espacial, pero con las
pruebas robustas solo el rezago sigue siendo significativo, así que un modelo
de rezago espacial es la especificación más adecuada.

El artículo está organizado en 2 tareas: `clean` y `replicate`.

## Directorios principales

* `clean`: limpia y resume las fuentes de datos por municipio (clave
  de 5 dígitos `cve_mun`). Cada notebook guarda una tabla por municipio. Los
  municipios son los 2,456 del censo 2010 (los mismos del Marco Geoestadístico
  2010); los registros de municipios creados después de 2010 se asignan a su
  municipio "padre" según `clean/hand/mun_crosswalk.yaml`:
  * `1_clean_homicides.ipynb`: número de homicidios ocurridos en 2018 en cada
    municipio.
  * `2_clean_sociodem.ipynb`: población total e indicadores socioeconómicos y
    demográficos de cada municipio, del censo 2010.
  * `3_clean_ocved.ipynb`: número de grupos criminales identificados distintos
    y presencia de los 10 grupos con más registros en cada municipio, en 2017.
* `replicate`: une las tres bases por municipio, estima el modelo MCO,
  construye la matriz de pesos espaciales con los polígonos municipales del
  Marco Geoestadístico 2010 y calcula las pruebas de dependencia espacial de
  Anselin et al. (1996). Los resultados (tablas, mapas e interpretación) están
  en el propio notebook, así que `replicate` no tiene carpeta `output/`.

## Organización de los directorios

Cada tarea es una carpeta independiente con estos subdirectorios:

* `input/`: archivos que lee la tarea. Los datos de una tarea anterior son
  enlaces simbólicos ("symlinks") relativos al `output/` de esa tarea; por
  ejemplo, `replicate/input/` contiene enlaces a archivos en `clean/output/`.
  La excepción son los polígonos: los municipios del Marco Geoestadístico 2010
  no necesitan limpieza, así que `replicate/input/municipios.*` son enlaces
  directos a los archivos en `clean/input/`. Nada en `input/` se
  edita a mano ni con código.
* `src/`: notebooks de Jupyter que convierten `input/` en `output/`. Están
  numerados en el orden en que se corren (`src/1_...`, luego `src/2_...`), y
  cada archivo de salida empieza con el número del notebook que lo generó.
* `output/`: todo lo que generan los notebooks. Si se borra y se vuelven a
  correr los notebooks, se regenera por completo. Solo la tienen las tareas
  que guardan archivos; `replicate` no la tiene, porque sus resultados están
  en el propio notebook.
* `hand/` (opcional): archivos pequeños en formato yaml que se escriben a mano,
  como tablas de equivalencias. Los notebooks los leen, pero nunca los
  modifican. Por ejemplo, `clean/hand/mun_crosswalk.yaml` lista los
  municipios creados después de 2010 y su municipio padre, con una nota sobre
  cómo se obtuvo.

Los datos limpios (`clean/output/`) se guardan en el repositorio. Los
datos crudos de `clean/input/` no se suben a GitHub y se descargan de
las fuentes listadas abajo. Por eso, en un clon nuevo, los enlaces a los
polígonos en `replicate/input/` no apuntan a ningún archivo hasta que
se descarga el Marco Geoestadístico 2010. Tampoco se sube el artículo de
Anselin et al. (1996), por derechos de autor; ningún notebook lo lee.

## Reproducir los resultados

Requisitos: Python 3 con Jupyter, `numpy`, `pandas`, `pyyaml`, `pyogrio`,
`openpyxl`, `scipy`, `statsmodels`, `matplotlib`, `geopandas`, `libpysal`,
`esda` y `spreg`.

1. Datos crudos en `clean/input/`:
   * Homicidios: microdatos de defunciones registradas de 2018 a 2024 del
     [INEGI](https://www.inegi.org.mx/programas/edr/#microdatos) (algunos
     homicidios ocurridos en 2018 se registraron en años posteriores).
     Descargar la base de cada año en formato DBF y guardar el archivo `DEFUN`
     (por ejemplo, `DEFUN18.DBF` o `DEFUN20.dbf`) en `clean/input/`.
   * Datos socioeconómicos y demográficos: principales resultados por
     localidad (ITER) del
     [Censo de Población y Vivienda 2010](https://www.inegi.org.mx/programas/ccpv/2010/)
     del INEGI. Descargar el ITER nacional en formato CSV y guardar
     `iter_00_cpv2010.csv` (en la carpeta `conjunto_de_datos/` de la descarga)
     en `clean/input/`.
   * Grupos criminales: [OCVED 2.0](https://github.com/javierosorio/OCVED_2.0)
     (Osorio y Beltrán, 2020). Descargar `OCVED_2.0.xlsx` en
     `clean/input/`.
   * Límites municipales:
     [Marco Geoestadístico 2010, versión 5.0.A](https://www.inegi.org.mx/temas/mg/)
     del INEGI (la versión que se usó en el censo 2010). Descargar el producto,
     descomprimir `mgm2010v5_0.zip` (municipios) y guardar `municipios.dbf`,
     `municipios.prj`, `municipios.shp` y `municipios.shx` en
     `clean/input/`.
2. Correr, en orden, los notebooks de `clean/src/`:
   1. `1_clean_homicides.ipynb`
   2. `2_clean_sociodem.ipynb`
   3. `3_clean_ocved.ipynb`
3. Los enlaces simbólicos de `replicate/input/` a los datos limpios y
   a los polígonos ya están en el repositorio. Para crear uno nuevo, desde esa
   carpeta: `ln -s ../../clean/output/<archivo> .` (o
   `ln -s ../../clean/input/<archivo> .` para los polígonos).
4. Correr el notebook `replicate/src/1_replicate.ipynb`.

Los notebooks deben correrse desde su propia carpeta `src/`, porque las rutas a
`input/` y `output/` se construyen a partir de ella.

## Contacto

Para preguntas sobre este repositorio, contactar a Sierra Wells en
sierrawells@gmail.com.
