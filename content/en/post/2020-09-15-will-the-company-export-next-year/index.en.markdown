---
title: Will the company export next year?
author: Nicolas Stumberger
date: '2020-09-15'
slug: will-the-company-export-next-year
categories:
  - R
tags:
  - R
  - Machine Learning
  - kNN
  - exports
  - International Trade
subtitle: 'How to predict if a company will export the following year with the algorithm k-NN classification. (Sorry, only in spanish &#128532;&#128591;).'
summary: 'The present analysis aims to answer the question in the subtitle: How to predict if a company will export the following year with the k-NN classification algorithm? This objective, in turn, serves a greater purpose, which is to bring techniques from the field of machine learning to the field of international trade, which is useful for both the public and private sectors. (Sorry, only in spanish &#128532;&#128591;).'
authors: []
lastmod: '2021-02-22T09:44:28-03:00'
featured: true
image:
  caption: ''
  focal_point: ''
  preview_only: no
  placement: 2
projects: []
---


# Introducción

El presente análisis tiene como objetivo responder a la pregunta del subtítulo: ¿Cómo predecir si una empresa exportará el siguiente año con el algoritmo de clasificación k-NN? Este objetivo sirve, a su vez, a un propósito mayor, que es el de aportar técnicas del campo del aprendizaje automático al ámbito del comercio internacional, que sea útil tanto para el sector público como para el privado.

Con este fin, se desarrolla un modelo predictivo de clasificación, utilizando el algoritmo k-NN y los datos utilizados para el mismo se ordenan en las siguientes variables generales: valores exportados, número y región de los destinos, número de productos diferentes y rubro al cual pertenece la empresa, tomando como horizonte temporal los últimos 6 años (2014-2019).

Este estudio no es un *paper* académico, por lo que no se reparará en detallar las citas a las fuentes consultadas. Sin embargo, resulta oportuno destacar que se utilizó como guía el capítulo 3 de *Machine Learning with R*, de Brett Lantz (2019).



# Sobre el algoritmo k-NN

## "Dios los cría y ellos se amontonan"

El algoritmo k-NN, por su siglas en inglés (k Nearest Neighbords: k Vecinos Cercanos) tiene por objetivo clasificar datos y colocarlos en la misma categoría de la de los "vecinos cercanos", por ser similares en otras caracterísitcas. Se basa en la hipótesis de que las cosas que son parecidas en algunos aspectos, comúnmente también lo son en otros. En este caso, la hipótesis bajada al caso práctico es que los exportadores parecidos entre sí (en su comportamiento exportador durante los últimos años) tendrán un resultado similar durante el siguiente año. La "clasificación" es la etiqueta que se desea predecir, llamada "target feature", y en este análisis será binaria: "exporta" o "no exporta" durante el siguiente año.

En este sentido, lo que hace este algoritmo es tomar observaciones sin clasificar (registros "nuevos") y les asigna la etiqueta de la clasificación de observaciones similares, de las que sí se cuenta con su clasificación. Para encontrar esa similitud, se toma variables (features) disponibles.

Ahora bien, ¿cómo calcula el algoritmo la cercanía entre las observaciones (entre las empresas en este caso)? Sintéticamente, utiliza un cálculo para medir distancias entre las variables de cada observación, llamado "distancia euclidiana", el cual se deduce a partir del Teorema de Pitágoras.

No es foco de este análisis entrar en detalles sobre el algoritmo (ya que hay mucha información al respecto en la web), pero sí interesa destacar algunos aspectos que ayudarán a entender mejor el resultado. Primero, este algoritmo se lo describe como "lazy" (vago, perezoso), porque en realidad no produce un modelo (no se produce ninguna abstracción), sino que utiliza los mismos datos para realizar la clasificación y, por esto mismo, la posibilidad de entender cómo cada variable está relacionada con el resultado (clasificación) es limitada. Por este motivo, el análisis no tendrá un resultado explicativo, sino que será únicamente predictivo.

Segundo, el algoritmo funciona únicamente con variables numéricas, por lo que cualquier variable categórica (si existiera, y en este caso existen) debe convertirse a una numérica creando variables "dummy" (proceso conocido como "dummy coding" o "one-hot encoding").

Tercero, el algoritmo debe entrenarse y testearse sobre datos históricos para luego poder ser empleado para predecir años futuros. Se debe contar con el resultado real de la clasificación para poder determinar la exactitud del algoritmo. Por este motivo, se realiza un análisis retrospectivo, en este caso 2014-2019, utilizando el periodo 2014-2018 para las variables predictoras y el año 2019 como \*target feature\*. Una vez entrenado y testeado el algoritmo sobre datos históricos, podría ser empleado para nuevos años (aún sin clasificar).

# Sobre los datos

Los datos que se utilizaron para el estudio, se obtuvieron de una fuente privada que recopila datos de aduana. A los efectos del análisis, las observaciones se agregaron a nivel anual por empresa. Estas últimas fueron anonimizadas. El rango de años tomados para el análisis es 2014-2019, utilizando el último año como *target feature* y no se lo utilizó para el entrenamiento del algoritmo.

Existen limitantes a la cantidad de variables que se pueden utilizar. Uno de ellos es la cantidad y qué variables cuenta la fuente primaria. La otra restricción, más subjetiva, es el tiempo y en este caso fue delimitado para que la etapa de preparación de datos, denominada *data wrangling* (o *data carpentry*) y *feature engineering* no se extienda demasiado y poder lograr un *MVP* (mínimo producto viable) en el corto plazo.

Esto no quiere decir que la selección de atributos haya sido aleatoria en pos de la celeridad. Por el contrario, las variables escogidas se sustentan en ciertas hipótesis sobre su correlación y potencial causalidad con la performance exportadora de las empresas: diversificación de cartera de productos; diversificación de destinos; regiones más estables que otras como mercados de destino; industrias más estables que otras; volúmenes exportados; entre otras. Sin embargo, sí se abre la discusión para permitir comentarios, sugerencias o modificaciones en los campos utilizados que tengan la intención de enriquecer el estudio.

Como resultado de lo antedicho, las variables que se utilizaron para realizar este análisis son:

1.  Número (cantidad ) de posiciones arancelarias distintas a nivel NCM (por cada año analizado): 5 variables, una por año.
2.  Número (cantidad) de países distintos a los que se exportó (por cada año analizado): 5 variables, una por año.
3.  Número (cantidad) de países distintos por región del mundo a los que se exportó (duranto todo el periodo analizado de 5 años): Latinoamérica, Europa, Asia-Pacífico, Maghreb y Medio Oriente, Resto de África, Norteamérica, Oceanía, Comunidad de Estados Independientes y Otros: 8 variables, una por región.
4.  Gran rubro (variables dummy) al que pertenecen las posiciones arancelarias exportadas, según la clasificación de INDEC: Manufacturas de Origen Agrícola (MOA), Manufacturas de Origien Industrial (MOI), Productos Primarios (PP), Combustible y Energía (CyE): 4 variables, una por gran rubro.
5.  Dólares FOB exportados por cada año analizado: 5 variables, una por año.

La descripción de este estudio no incluye todo el proceso de *data carpentry* realizado. Se suele decir que la etapa de preparación de datos ocupa más del 80% del proceso de análisis. Sin embargo, en este caso, fue cerca del 95%. Como el objetivo del informe no es describir cómo se prepararon los datos, sino predecir exportaciones, se consideró prescindible incluirlo aquí y se priorizó el desarrollo del tema del asunto.

# Prediciendo si una empresa va a exportar el siguiente año con k-NN

## Carga de librerías


```r
library(tidyverse) # para manipular todos los datos, graficar, etc.
library(class) # paquete que contiene el algoritmo knn de clasificacion
library(gmodels) # para evaluar el modelo con crosstable
library(plotly)
```

## Obtención de datos

Tal como se mencionó arriba, los datos fueron procesados previamente. El resultado de este proceso de *data carpentry* y *feature engineering* es el archivo exportadores\_prep.RDS.


```r
exportadores <- readRDS(file = "data/exportadores_prep.RDS")
```

El dataset cuenta con poco más de 16.000 filas, representando exportadores, y 29 columnas. La primera es *id* del exportador, la segunda es el *target feature* (la variable que se desea predecir) que indica si la empresa exportó o no en el último año. El resto de las variables (27 ) son los atributos que se utilizarán para predecir.


```r
str(exportadores)
```

```
## tibble [16,146 x 29] (S3: tbl_df/tbl/data.frame)
##  $ id                               : int [1:16146] 1 2 3 4 5 6 7 8 9 10 ...
##  $ target                           : Factor w/ 2 levels "export","no_export": 1 2 2 2 1 2 1 2 2 2 ...
##  $ ncm_1                            : num [1:16146] 3 2 16 1 1 2 10 1 4 1 ...
##  $ ncm_2                            : num [1:16146] 3 0 2 0 1 0 11 0 4 0 ...
##  $ ncm_3                            : num [1:16146] 3 0 0 0 3 0 12 0 5 0 ...
##  $ ncm_4                            : num [1:16146] 3 0 0 0 1 0 1 0 2 0 ...
##  $ ncm_5                            : num [1:16146] 3 0 0 0 1 0 0 0 0 0 ...
##  $ dest_1                           : num [1:16146] 2 1 2 1 2 2 2 1 2 1 ...
##  $ dest_2                           : num [1:16146] 3 0 1 0 2 0 1 0 2 0 ...
##  $ dest_3                           : num [1:16146] 2 0 0 0 3 0 1 0 2 0 ...
##  $ dest_4                           : num [1:16146] 3 0 0 0 1 0 1 0 1 0 ...
##  $ dest_5                           : num [1:16146] 3 0 0 0 1 0 0 0 0 0 ...
##  $ latam                            : num [1:16146] 3 0 2 0 3 1 0 1 2 0 ...
##  $ nort_amer                        : num [1:16146] 1 0 0 1 0 0 1 0 0 1 ...
##  $ europa                           : num [1:16146] 0 1 0 0 0 1 1 0 0 0 ...
##  $ mag_med_oriente                  : num [1:16146] 0 0 0 0 0 0 0 0 0 0 ...
##  $ asia_pacif                       : num [1:16146] 0 0 0 0 0 0 0 0 0 0 ...
##  $ oceania                          : num [1:16146] 0 0 0 0 0 0 0 0 0 0 ...
##  $ africa_resto                     : num [1:16146] 0 0 0 0 0 0 0 0 0 0 ...
##  $ cei                              : num [1:16146] 0 0 0 0 0 0 0 0 0 0 ...
##  $ manufacturas_de_origen_agricola  : num [1:16146] 1 0 0 0 0 0 0 0 0 0 ...
##  $ manufacturas_de_origen_industrial: num [1:16146] 1 1 1 1 1 1 1 1 1 0 ...
##  $ productos_primarios              : num [1:16146] 0 0 0 0 0 0 0 0 0 1 ...
##  $ combustible_y_energia            : num [1:16146] 0 0 0 0 0 0 0 0 0 0 ...
##  $ fob_1                            : num [1:16146] 310406 147 77341 4770 32220 ...
##  $ fob_2                            : num [1:16146] 87180 0 49374 0 35090 ...
##  $ fob_3                            : num [1:16146] 142667 0 0 0 55160 ...
##  $ fob_4                            : num [1:16146] 74055 0 0 0 3570 ...
##  $ fob_5                            : num [1:16146] 195425 0 0 0 12520 ...
```

La tabla metadatos contiene la descripción de cada campo:


```r
metadatos <-
        xlsx::read.xlsx2(file = "data/metadatos.xlsx", 
                         sheetName = "metadatos")

knitr::kable(metadatos, caption = "Metadatos")
```



Table: Table 1: Metadatos

|campo                             |descripcion                                                                                                                                                        |
|:---------------------------------|:------------------------------------------------------------------------------------------------------------------------------------------------------------------|
|id                                |id del exportador                                                                                                                                                  |
|target                            |variable a predecir: exporto o no exporto el ultimo año (2019)                                                                                                     |
|ncm_1                             |cantidad de posiciones arancelarias a nivel ncm exportadas en 2014                                                                                                 |
|ncm_2                             |cantidad de posiciones arancelarias a nivel ncm exportadas en 2015                                                                                                 |
|ncm_3                             |cantidad de posiciones arancelarias a nivel ncm exportadas en 2016                                                                                                 |
|ncm_4                             |cantidad de posiciones arancelarias a nivel ncm exportadas en 2017                                                                                                 |
|ncm_5                             |cantidad de posiciones arancelarias a nivel ncm exportadas en 2018                                                                                                 |
|dest_1                            |cantidad de países de destino a los que se exportó en 2014                                                                                                         |
|dest_2                            |cantidad de países de destino a los que se exportó en 2015                                                                                                         |
|dest_3                            |cantidad de países de destino a los que se exportó en 2016                                                                                                         |
|dest_4                            |cantidad de países de destino a los que se exportó en 2017                                                                                                         |
|dest_5                            |cantidad de países de destino a los que se exportó en 2018                                                                                                         |
|latam                             |cantidad de países de destino a los que se exportó dentro de la región de Latinoamérica durante todo el rango de años analizados (2014-2018)                       |
|nort_amer                         |cantidad de países de destino a los que se exportó dentro de la región de Norteamérica durante todo el rango de años analizados (2014-2018)                        |
|europa                            |cantidad de países de destino a los que se exportó dentro de la región de Europa durante todo el rango de años analizados (2014-2018)                              |
|mag_med_oriente                   |cantidad de países de destino a los que se exportó dentro de la región de Maghreb y Medio Oriente durante todo el rango de años analizados (2014-2018)             |
|asia_pacif                        |cantidad de países de destino a los que se exportó dentro de la región de Asia-Pacífico durante todo el rango de años analizados (2014-2018)                       |
|oceania                           |cantidad de países de destino a los que se exportó dentro de la región de Oceanía durante todo el rango de años analizados (2014-2018)                             |
|africa_resto                      |cantidad de países de destino a los que se exportó dentro de la región de Resto de África (sin Maghreb) durante todo el rango de años analizados (2014-2018)       |
|cei                               |cantidad de países de destino a los que se exportó dentro de la región de Comunidad de Estados Independientes durante todo el rango de años analizados (2014-2018) |
|manufacturas_de_origen_agricola   |1 si exportó posiciones arancelarias del gran rubro MOA. 0 si no exportó posiciones arancelarias de este gran rubro                                                |
|manufacturas_de_origen_industrial |1 si exportó posiciones arancelarias del gran rubro MOI. 0 si no exportó posiciones arancelarias de este gran rubro                                                |
|productos_primarios               |1 si exportó posiciones arancelarias del gran rubro PP. 0 si no exportó posiciones arancelarias de este gran rubro                                                 |
|combustible_y_energia             |1 si exportó posiciones arancelarias del gran rubro CyE. 0 si no exportó posiciones arancelarias de este gran rubro                                                |
|fob_1                             |valor exportado en dólares FOB en 2014                                                                                                                             |
|fob_2                             |valor exportado en dólares FOB en 2015                                                                                                                             |
|fob_3                             |valor exportado en dólares FOB en 2016                                                                                                                             |
|fob_4                             |valor exportado en dólares FOB en 2017                                                                                                                             |
|fob_5                             |valor exportado en dólares FOB en 2018                                                                                                                             |

### Un poco de EDA (Análisis Exploratorio de Datos)

A continuación, y a modo ilustrativo, se grafica la dispersión de las observaciones por las variables de número de destinos en los distintos años del análisis (atributos). Los colores representan la clasificación: exporta o no exporta el último año.


```r
gran_rubro <- exportadores %>% 
        select(21:24) %>% 
        names()

region <- exportadores %>% 
        select(13:20) %>% 
        names()

n_dest <- exportadores %>% 
        select(8:12) %>% 
        names()

n_ncm <- exportadores %>% 
        select(3:7) %>% 
        names()

fob <- exportadores %>% 
        select(25:29) %>% 
        names()

expo_long <- exportadores %>% 
        pivot_longer(cols = 3:29, names_to = "atributo")

expo_long %>%
        filter(atributo %in% n_dest) %>% 
        ggplot(aes(value, atributo)) +
        geom_jitter(aes(color = target), alpha = 0.1, height = 0.35) + 
        theme_minimal() +
        labs(title = "Número de destinos a los que se exportó en los años anteriores al analizado",
             subtitle = "Cada punto representa una empresa exportadora y el color determina si realizó una exportación al sexto año.",
             x = "Cantidad de destinos",
             y = "Atribito")
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-3-1.png" width="960" />

No se alcanza a detectar en detalle, pero sí se pueden apreciar dos particularidades: primero, se observa una mayor densidad de puntos en los valores más bajos, de 0 a 10 destinos aproximadamente; segundo, se advierte una mayor proporción de la clase "export" (en rojo), en aquellas observaciones que registran un mayor número de destinos (valores altos en el eje de las x). Esto es algo que lo puede dictar el sentido común (hay más empresas con pocos mercado de exportación y pocas empresas con muchos mercados; y también más destinos, significan más chances de continuar exportando), pero es importante demostrarlo con datos.

## Preparación de datos

La primer variable es el `id` del exportador. Como es un identificador único por cada exportador, no provee ninguna información útil para el modelo, por lo que se excluye.


```r
exportadores_1 <- exportadores %>% 
        select(-id)
```

La siguiente variable, `target`, es de particular importancia, debido a que es el resultado que se desea predecir. Este campo indica si el exportador realizó o no una venta al exterior en el último año del periodo analizado, en este caso durante el año 2019.


```r
table(exportadores_1$target)
```

```
## 
##    export no_export 
##      8293      7853
```

Se puede observar que de los 16 mil registros, aproximadamente la mitad realizó una exportación. El dataset está balanceado y esto es bueno para el tener un *baseline*: si usáramos solo el azar para predecir si una empresa exportará o no al siguiente año, conociendo esta proporción, tenemos un 50% de probabilidades de acertarle al resultado. A continuación, se observa la participación (%).


```r
round(prop.table(table(exportadores_1$target)) * 100, digits = 1)
```

```
## 
##    export no_export 
##      51.4      48.6
```

Las restantes 27 variables son numéricas, tal como el algoritmo lo requiere, y describen la cantidad de productos, cantidad de destinos, región del mundo a la que se exportó y montos despachados durante el rango de años estudiados.

A modo ilustrativo, se observa un resumen estadístico de 3 de estos campos: cantidad de productos distintos (`ncm_5`), cantidad de países distintos (`dest_5`) y valores exportados (`fob_5`) durante uno de los años.


```r
summary(exportadores_1[c("ncm_5", "dest_5", "fob_5")])
```

```
##      ncm_5            dest_5           fob_5          
##  Min.   :  0.00   Min.   : 0.000   Min.   :0.000e+00  
##  1st Qu.:  0.00   1st Qu.: 0.000   1st Qu.:0.000e+00  
##  Median :  1.00   Median : 1.000   Median :9.714e+03  
##  Mean   :  3.81   Mean   : 2.247   Mean   :3.770e+06  
##  3rd Qu.:  3.00   3rd Qu.: 2.000   3rd Qu.:1.610e+05  
##  Max.   :428.00   Max.   :97.000   Max.   :2.904e+09
```

El algoritmo k-NN es altamente dependiente de la escala de las variables de input. Al analizar esta información en detenimiento , se puede ver que los rangos de valores de las variables `ncm_5` y `dest_5` son muy diferentes de la de `fob_5`. Debido a que `dest_5` varía de 0 a 97 y `fob_5` varía de 0 a 2.904.000.000, el impacto de este último va a ser mayor en el cálculo de la distancia en el algoritmo, comparado con el primero. Por este motivo, el dataset debe ser normalizado.

### Normalización

Se presentan dos alternativas para normalizar los datos:

-   Normalización con *Min-Max*

-   Normalización con *Z-score* (desviaciones estándar)

Ambos métodos son útiles para el algoritmo. La normalización min-max convierte todos los valores a una escala de 0 a 1, comprimiendo los extremos hacia el centro, acortando el rango total. La normalización con *z-score*, por su parte, no tiene límites mínimos ni máximos definidos, por lo que los valores extremos no son comprimidos hacia el centro, sino que convierte la media del conjunto de valores en 0, y todos los valores se ubican por debajo o por encima del 0 en función a su distancia de la media, medida en desviaciones estándar.

La decisión para utilizar una u otra forma de normalizar los datos puede tomarse en base a los datos mismos: ¿interesa que ponderen más los valores extremos o no?

Primero se utilizará la normalización min-max. Para ello, se crea una función que permita convertir todo el dataset en escala normalizada.


```r
normalize <- function(x) {
        
        return((x - min(x, na.rm = TRUE)) / 
                       (max(x, na.rm = TRUE) - 
                                min(x, na.rm = TRUE)
                        )
               )
        
}
```

Con las siguientes lineas se testea que la función se ejecute bien.


```r
normalize(c(1,2,3,4, NA, 5))
```

```
## [1] 0.00 0.25 0.50 0.75   NA 1.00
```

```r
normalize(c(10, 20, 30, 40, 50))
```

```
## [1] 0.00 0.25 0.50 0.75 1.00
```

Por lo que se observa, los valores fueron comprimidos a una escala de 0 a 1, por lo que se da por sentado que funcionó ok.

A continuación, se aplica la funcion a las 27 variables numéricas del dataset. La función `lapply()` permite hacer eso. `lapply()` toma una lista y aplica una función específica a cada elemento de la lista. Como una tabla es como una lista de vectores, se puede utilizar `lapply()`. Por último, lo que hace el siguiente *chunk*, es convertir el resultado a un dataframe.


```r
# Excluyo la variable target para normalizar

exportadores_n <- lapply(exportadores_1[2:28], normalize) %>%
        as_tibble()
```

Para confirmar que las variables fueron normalizadas conformemente, se observa el resumen estadístico de una variable.


```r
summary(exportadores_n$fob_5)
```

```
##      Min.   1st Qu.    Median      Mean   3rd Qu.      Max. 
## 0.0000000 0.0000000 0.0000033 0.0012984 0.0000555 1.0000000
```

Según lo esperado, `fob_5` (exportaciones del último año de análisis) posee un rango 0-1.

### División: training y test

Tal como lo expresan las buenas prácticas del machine learning, el dataset se dividirá en 2. Una parte se utilizará para entrenar el modelo y la otra para testearlo. La intención es que el modelo sea testeado sobre datos no vistos previamente por éste. En este caso, se empleará el 80% de los registros para entrenar y el restante 20% para testear.

A continuación, se crea una muestra aleatoria para realizar la división y se utiliza `set.seed()` para su replicabilidad.


```r
# divido exportadores_n en expo_train y expo_test

# para poder replicar la mueastra aleatoria
set.seed(123)

# creo la muestra de un total de las filas del dataset, el 80%.
train_sample <- sample(nrow(exportadores_1), 
                       round(nrow(exportadores_1) * 0.8, 
                             digits = 0))


str(train_sample)
```

```
##  int [1:12917] 2463 2511 10419 8718 12483 2986 1842 9334 3371 13541 ...
```

```r
# creo el set de entrenamiento en base a exportadores_n, con las filas de la muestra y el set de test sin las filas de la muestra.
expo_train <- exportadores_n[train_sample,]
expo_test <- exportadores_n[-train_sample,]
```

Al normalizar el dataset, en el paso anterior, se excluyó la variable `target` . Ahora, para entrenar el algoritmo, se necesitará de esta variable para clasificar. Por este motivo, seguidamente se guarda esta variable como vector, tanto para el dataset de entrenamiento como para el de testeo.

Por otro lado, se chequea que la proporción de cada clase, en cada dataset, sea similar a la del dataset completo. De esta manera, se comprobará que la muestra fue realizada correctamente.


```r
expo_train_labels <- exportadores_1[train_sample, 1]
expo_test_labels <- exportadores_1[-train_sample, 1]

# Acá hequeo que la proporción de "export" o "no_export" sea similar a la del dataset completo (50%-50% aprox).

prop.table(table(expo_train_labels))
```

```
## expo_train_labels
##    export no_export 
## 0.5136642 0.4863358
```

```r
prop.table(table(expo_test_labels))
```

```
## expo_test_labels
##    export no_export 
## 0.5134717 0.4865283
```

Por lo que se observa, las proporciones de ambos datasets son semejantres y ahora está todo listo para entrenar el modelo.

## Entrenando el modelo

Para un *lazy* learner como *k-NN* no es necesaria ninguna contruccion de modelo. Solo se debe guardar la data de input en un formato estructurado para usarlo con la nueva data.

La función `knn()` del paquete *class* identifica el *k nearest neighbors*, usando distancia Euclideana, en donde *k* es un número especificado por el usuario y representa el número de vecinos a usar para decidir la clase de cada nuevo registro. El test clasifica mediante un voto entre los *k nearest neighbors*: se asigna la *class* de la mayoria de los vecinos y un empate se define aleatoreamente.

Algunos autores establecen que el número de *k* a utilizar debe ser igual a la raíz cuadrada del total de observaciones entrenadas. En este caso, como se entrenan 12917 observaciones, el *k* elegido en primer lugar, será 114 . El siguiente *chunck* calcula el número de *k* en base a lo que mencionan los autores.


```r
sqrt(nrow(expo_train)) %>% round(digits = 0)
```

```
## [1] 114
```

Este número será modificado en futuras iteraciones de este proceso para chequear que se utilice aquel que mejor performe sobre el dataset de testeo.

A continuación, el algoritmo:


```r
# IMPORTANTE: cl DEBE ser vector, no table. Por lo tanto DEBE seleccionarse la columna!!!

expo_test_pred <-
        knn(
                train = expo_train,
                test = expo_test,
                cl = expo_train_labels$target,
                k = 114,
                prob = TRUE
        )
```

Esto arroja como output un vector de clase factor, con los *predicted labels* para el test.

## Evaluando la performance del modelo

Para evaluar el resultado del algoritmo sobre los datos, se realiza un *cross table,* seguido de una matriz de confusón a partir de la cual se calculará el *accuracy*


```r
CrossTable(# nuevamente deben ser vectores
        x = expo_test_labels$target,
        y = expo_test_pred,
        prop.chisq = FALSE)
```

```
## 
##  
##    Cell Contents
## |-------------------------|
## |                       N |
## |           N / Row Total |
## |           N / Col Total |
## |         N / Table Total |
## |-------------------------|
## 
##  
## Total Observations in Table:  3229 
## 
##  
##                         | expo_test_pred 
## expo_test_labels$target |    export | no_export | Row Total | 
## ------------------------|-----------|-----------|-----------|
##                  export |      1232 |       426 |      1658 | 
##                         |     0.743 |     0.257 |     0.513 | 
##                         |     0.774 |     0.260 |           | 
##                         |     0.382 |     0.132 |           | 
## ------------------------|-----------|-----------|-----------|
##               no_export |       360 |      1211 |      1571 | 
##                         |     0.229 |     0.771 |     0.487 | 
##                         |     0.226 |     0.740 |           | 
##                         |     0.111 |     0.375 |           | 
## ------------------------|-----------|-----------|-----------|
##            Column Total |      1592 |      1637 |      3229 | 
##                         |     0.493 |     0.507 |           | 
## ------------------------|-----------|-----------|-----------|
## 
## 
```

En esta tabla de arriba las columnas representan las predicciones de clase y las filas las clases reales.


```r
# matriz de confusión casera para calcular el accuracy

confus_matrix <- table(expo_test_pred,expo_test_labels$target)

confus_matrix
```

```
##               
## expo_test_pred export no_export
##      export      1232       360
##      no_export    426      1211
```

Aquí las filas son las predicciones y las columnas representan la clasificación real. Como puede observarse, de 3229 predicciones, se registraron 360 falsos positivos y 426, falsos negativos.

A continación, se crea una función para calcular el *accuracy* general del modelo: predicciones correctas dividido por el total de predicciones.


```r
# Esta función divide las predicciones correctas por el total de predicciones. Nos da el accuracy del modelo.

accuracy <- function(x){sum(diag(x)/(sum(rowSums(x)))) * 100}

accuracy(confus_matrix)
```

```
## [1] 75.6581
```

Casi 76% de *accuracy*! Este resultado no está nada mal para ser el primer intento.






## Mejorando la performance del modelo

En esta sección se intentarán dos variaciones del modelo: primero, se modificará el método de normalización utilizado; segundo, intentarán diferentes valores de *k.*

### Normalización con *z-score*

Es importante remarcar que la normalización *z-score*, a diferencia de la min-max, no comprime los valores extremos hacia el centro y el rango va de valores negativos a positivos, convirtiendo la media en 0. En esta normalización, los outliers ponderan más en el cálculo de la distancia aplicado por el algoritmo que en la normalización utilizada en la primer iteración.


```r
# la función scale() se aplica directamente sobre un dataset

exportadores_z <- exportadores_1 %>% 
        select(-target) %>% 
        scale() %>% 
        as_tibble()
```

Para chequear que se normalizó correctamente, se observa el resumen estadístico de una de las variables.


```r
summary(exportadores_z$fob_5)
```

```
##     Min.  1st Qu.   Median     Mean  3rd Qu.     Max. 
## -0.05929 -0.05929 -0.05914  0.00000 -0.05676 45.60371
```

Se observa que la media es 0 y que el rango va de -0.059 a 45.603 (outlier). Un *z-score* menor a -3 o mayor a 3 indica un valor muy poco probable. Habiendo examinado esto, aparentemente la normalización funcionó bien.

Al igual que en el primer intento, ahora se debe dividir la data normalizada en *train* y *test sets* y clasificar el test utilizando el algoritmo `knn()`. Se finalizará esta iteración comparando las predicciónes con la clase real en una matriz de confusión y se aplicará la fórmula para conocer el accuracy del modelo, al igual a como se hizo anteriormente.


```r
# ya tengo creado un registro de filas de muestra, por lo que no necesito volver a hacer realizar el set.seed() y sample()

# creo el set de entrenamiento en base a exportadores_z, con las filas de la muestra y el set de test sin las filas de la muestra.
expo_train <- exportadores_z[train_sample,]
expo_test <- exportadores_z[-train_sample,]
expo_train_labels <- exportadores_1[train_sample, 1]
expo_test_labels <- exportadores_1[-train_sample, 1]

expo_test_pred <-
        knn(
                train = expo_train,
                test = expo_test,
                cl = expo_train_labels$target,
                k = 114,
                prob = TRUE
        )

confus_matrix <- table(expo_test_pred,expo_test_labels$target)

confus_matrix
```

```
##               
## expo_test_pred export no_export
##      export      1239       339
##      no_export    419      1232
```

```r
# Esta función divide las predicciones correctas por el total de predicciones. Nos da el accuracy del modelo.

accuracy <- function(x){sum(diag(x)/(sum(rowSums(x)))) * 100}

accuracy(confus_matrix)
```

```
## [1] 76.52524
```

Al parecer, favoreciendo el peso de los outliers (y manteniendo todo lo demás igual), mejora un poco el modelo: 76,52% de *accuracy*.

### Valores alternativos de *k*

En esta sección se probarán distintos valores de *k*. Para ello, se crearán funciones que realicen todo el proceso y permitan obtener, como *output*, los valores de: verdaderos positivos, verdaderos negativos, falsos positivos, falsos negativos y al *accuracy* general del modelo. Seguidamente, se creará una tabla con todos estos resultados y se graficará la performance del modelo en función a los diferentes valores de *k* posibles.


```
## # A tibble: 140 x 7
##    k_value verdad_pos verdad_neg falso_neg falso_pos accuracy_fc   tot
##      <int>      <int>      <int>     <int>     <int>       <dbl> <int>
##  1       1       1228       1129       442       430        73.0  3229
##  2       2       1251       1138       439       403        72.3  3231
##  3       3       1296       1191       380       362        77.0  3229
##  4       4       1297       1185       377       369        76.6  3228
##  5       5       1308       1195       376       349        77.5  3228
##  6       6       1296       1209       364       359        77.5  3228
##  7       7       1318       1210       362       338        78.3  3228
##  8       8       1318       1208       368       346        78.1  3240
##  9       9       1325       1228       345       335        79.0  3233
## 10      10       1318       1217       350       338        78.9  3223
## # ... with 130 more rows
```

Al iterar con diferentes valores de *k*, se observa que el mejor resultado se obtiene con un *k* igual a 11, 12 ó 13, con un *accuracy* de 79.22%. En otras palabras, este modelo (en promedio) logra predecir correctamente el resultado de 8 de cada 10 empresas que dispongan de estos datos y supera en 30 puntos porcentuales al *baseline* con el que se cuenta (50%).

Un aspecto que extraña del resultado es que la mejor perrformance se logró utilizando un número muy inferior al recomendado para un dataset de estas dimensiones: 11 en vez de 114.

```r
k_alternativos_z %>% 
        ggplot(aes(k_value, accuracy_fc)) +
        geom_line(group = 1) +
        theme_minimal() +
        labs(x = "k (número de vecinos cercanos)",
             y = "Accuracy")
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-23-1.png" width="960" />


Tal como se hizo con anterioridad, se crea una tabla de valores alternativos para *k*, pero con la normalización min-max, para constatar que su performance es inferior con cualquier número de vecinos utilizados.


```r
k_alternativos_n <- readRDS(file = "data/k_alternativos_n.RDS") %>% 
        select(-accuracy_m)

k_alternativos_n
```

```
## # A tibble: 140 x 7
##    k_value verdad_pos verdad_neg falso_neg falso_pos accuracy_fc   tot
##      <int>      <int>      <int>     <int>     <int>       <dbl> <int>
##  1       1       1220       1116       455       438        72.3  3229
##  2       2       1225       1114       446       425        72.4  3210
##  3       3       1291       1172       398       367        76.2  3228
##  4       4       1277       1157       414       374        74.9  3222
##  5       5       1294       1169       402       364        76.4  3229
##  6       6       1295       1178       402       361        76.5  3236
##  7       7       1325       1186       385       336        77.8  3232
##  8       8       1308       1181       383       338        77.3  3210
##  9       9       1317       1197       375       345        77.9  3234
## 10      10       1318       1198       381       344        77.9  3241
## # ... with 130 more rows
```

A través de un gráfico, se puede apreciar mejor el desempeño de ambos modelos, por lo que a continuación se recrea el mismo, adicionando los datos normalizados con min-max.


```r
k_alternativos_n <- k_alternativos_n %>% 
        mutate(normalizacion = "Min-Max")


k_alternativos_z <- k_alternativos_z %>% 
        mutate(normalizacion = "Z-score")

k_alternativos <- bind_rows(k_alternativos_n, k_alternativos_z)


k_alternativos %>%
        ggplot(aes(k_value, accuracy_fc)) +
        geom_line(aes(group = normalizacion, color = normalizacion), size = 1) +
        geom_vline(xintercept = 11, alpha = 0.3) +
        theme_minimal() +
        guides(color = guide_legend(title = "Normalización")) +
        scale_x_continuous(
                breaks = c(11, 50, 100)
        ) +
        labs(
                subtitle = "Performance general del modelo en función al valor de k con diferente método de normalización",
                title = "¿Cuántos vecinos pueden votar?",
                x = "k (número de vecinos cercanos)",
                y = "Accuracy"
        )
```

<img src="{{< blogdown/postref >}}index.en_files/figure-html/unnamed-chunk-25-1.png" width="960" />
Aquí se observa que los valores de *k* con los que se logra el mejor performance son coincidentes en ambos métodos de normalización y que se encuentran entre 11 y 13 (vecinos cercanos).

# Conclusiones

El objetivo de desarrollar un modelo que permita predecir si una empresa va a exportar el siguiente año fue cumplido, con un accuracy de 79%. Existen y se enumeran algunas alternativas para mejorar este resultado en nuevas iteraciones, realizando modificaciones en los datos y/o en el algoritmo empleado. El análisis no tiene poder explicativo, pero sí predictivo y puede ser de utilidad para algunos actores del comercio exterior argentino.

## Resultado
El modelo desarrollado predice correctamente 8 de cada 10 observaciones. Ante la incertidubre sobre la continuidad exportadora de una empresa durante el siguiente año, la primer conclusión a resaltar sobre el estudio es que, a partir de los datos y variables utilizadas, se logró un accuracy de 79.22%: 8 de cada 10 exportadores fueron acertadamente clasificados. Este valor es considerablemente superior al 50% de base con el que se contaba.

## Oportunidades de mejora
A partir del análisis realizado, se desprenden nuevas incógnitas y oportunidades de mejora que pueden merecer otras iteraciones. Se separan en mejoras sobre los datos y mejoras sobre el algoritmo.

En primer lugar, los datos, que son más importantes que los algoritmos. Esta es una de las principales máximas de la ciencia de datos (*data beats algorithm*) y resume la idea de que la calidad del modelo depende de la calidad, cantidad y variedad de los datos utilizados para construirlo. Para testear si los datos utilizados son los mejores posibles, es necesario probar el mismo modelo, empleando otra data. A continuación se sugieren tres alternativas a explorar en este sentido con algunas interrogantes. Para empezar, se pueden crear diferentes variables (*feature engineering): quizás ampliando o reduciendo el número de regiones de destino; ampliando la sectorización de las empresas a un nivel más desagregado; reduciendo el horizonte temporal a 3 años; entre otras. 

Otra opción es cambiar totalmente la fuente utilizada. Según se ha visto, el análisis evalúa empresas con historial exportador. Aquella empresa que nunca exportó, no puede predecirse según este modelo, por no contar con atributos predictores. Para solucionar este inconveniente, se deben utilizar otras nuevas variables. ¿Qué otra fuente de información con datos de empresas argentinas (exportadoras y no exportadoras) podría llegar a ser predictivamente útil? ¿Qué nuevas variables podrían predecir si una empresa exportará o no al siguiente año?

Una tercer alternativa es utilizar las mismas variables, pero aumentando la cantidad de años utilizados. Cabe recordar que los datos empleados corresponden al periodo 2014-2019, en donde este último solo se lo utilizó como clase a predecir. Para que sea replicable en otros años, se debería crear un dataset que no tenga únicamente este periodo de años. El lapso analizado puede esconder alguna tendencia propia del mismo que evite la replicabilidad para otros (futuros) años. 

Por otra parte, se proponen mejoras en el algoritmo utilizado. Un modelo simple es mejor que uno complejo: a menos que se puede justificar, no es necesario utilizar técnicas ni modelos complejos. Por esto mismo, ¿es k-NN el mejor algoritmo para predecir si exporta una empresa al siguiente año? Como ya se ha aclarado, este algoritmo es útil para clasificar cuando las relaciones entre los atributos y la clase son numerosas, complejas y difíciles de entender. Quizás no sea éste el caso ejemplar. Muchas veces, es clara la relación entre el volumen exportado, cantidad de destinos a los que se exporta, etc., con el resultado al siguiente año. ¿Sería interesante desarrollar un *decision tree* con esta misma data?

## Utilidad
El análisis no explica qué variables son las que más inciden en el resultado. Como ya se ha aclarado al comienzo, una de las debilidades del algoritmo k-NN es que no produce un modelo, propiamente dicho, por lo que no es posible entender cómo los atributos se relacionan con el resultado. Para tener un mayor poder explicativo, debería utilizar otro tipo de algoritmo (*desicion tree*, por ejemplo).

No obstante lo anterior, el estudio puede llegar a ser de utilidad para algunos actores involucrados en el comercio exterior argentino que, desde la perspectiva de su negocio o misión, les resulta conveniente identificar anticipadamente si una empresa logrará exportar o no con el objetivo de ofrecer sus servicios. Una empresa de servicios asociados a la exportación, por ejemplo, podría ser más eficaz si enfoca sus recursos comunicacionales en aquellas empresas que tengan más probabilidades de exportar al siguiente año, debido a que son potenciales clientes. Una agencia de promoción de exportaciónes (nacional o provincial), cuya misión es fomentar las exportaciones de su territorio, por su parte, podría redoblar sus esfuerzos en aquellas empresas con bajas chances de exportar al siguiente año para que, a pesar de ello, logren hacerlo.


























