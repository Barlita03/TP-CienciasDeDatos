# Justificaciones

Registro de todas las propuestas que probamos (análisis, preprocesamiento, features, modelos), tanto las aprobadas como las descartadas, con la evidencia que respalda cada decisión.

Todas las propuestas, incluidas las descartadas, tienen que figurar con su código en el notebook, para que se vea qué probamos y por qué tomamos cada decisión.

## Cómo agregar una entrada

1. Copiar la plantilla en la sección de la etapa que corresponda.
2. Numerarla con el siguiente número libre (`J-01`, `J-02`, …). La numeración es única para todo el archivo, no por etapa.
3. En **Evidencia** van los resultados clave (números, una tabla o un gráfico), para que la entrada se entienda sin abrir el código. Las imágenes se guardan en [img/](img/).
4. Agregar la entrada en la misma PR que la propuesta, aunque se haya descartado.

## Plantilla

```markdown
### J-XX · Título corto de la propuesta
- **Etapa:** número y nombre de la etapa
- **Issue / PR:** #issue / #PR
- **Propuesta:** qué se quiere probar y por qué podría servir.
- **Cómo se probó:** qué se hizo para evaluarla.
- **Evidencia:** resultados concretos (números, tabla o gráfico).
- **Decisión:** ✅ Aprobada / ❌ Descartada
- **Justificación:** por qué se tomó esa decisión, a partir de la evidencia.
```

## 1. Exploración del dataset

## 2. Análisis de las variables disponibles

### J-01 · Histograma sin transformar y normalización min-max para graficar `favorite_count` y `retweet_count`
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #1 / #30
- **Propuesta:** graficar las métricas de interacción con un histograma común y, para mejorar la visualización, normalizarlas con min-max.
- **Cómo se probó:** histograma de cada variable sin transformar y de su versión normalizada con (x − min) / (max − min), y porcentaje de datos que cae en el primer bin.
- **Evidencia:** sin transformar, el 99,55% de `favorite_count` y el 99,11% de `retweet_count` cae en el primer bin. Con min-max los porcentajes son los mismos y el histograma tiene la misma forma, solo cambia el eje X a [0, 1].
- **Decisión:** ❌ Descartada
- **Justificación:** el histograma común muestra una sola barra y la cola no se ve. Min-max cambia la escala pero no la forma de la distribución, y depende del máximo, que es un valor extremo (2243 y 502), así que no ayuda a visualizarla.

### J-02 · Transformación `log1p` con todos los datos (opción A)
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #1 / #30
- **Propuesta:** graficar `log1p(x)` = log(1 + x) de todos los tweets, para comprimir la cola. Se usa `log1p` y no `log` porque la mayoría de los valores son 0 y log(0) no existe.
- **Cómo se probó:** histograma de `log1p` de cada variable, junto con el porcentaje de ceros y el resumen estadístico.
- **Evidencia:** el 53,15% de los tweets tiene 0 favoritos y el 84,74% tiene 0 retweets. Media vs. mediana: 3,82 vs. 0 en favoritos y 0,87 vs. 0 en retweets. El máximo es 2243 en favoritos y 502 en retweets.
- **Decisión:** ✅ Aprobada, como complemento de J-03
- **Justificación:** muestra todo el dataset y la proporción real de tweets sin interacción. Pero la barra del 0 domina el gráfico y el cuerpo de los tweets con interacción se distingue poco, por eso no alcanza sola.

### J-03 · Filtrar los tweets con interacción y usar eje logarítmico (opción B)
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #1 / #30
- **Propuesta:** graficar solo los tweets con valor mayor a 0, con el eje X en escala logarítmica, para ver la forma de la distribución sin que los ceros la tapen.
- **Cómo se probó:** histograma con `log_scale=True` de los valores mayores a 0 de cada variable, y resumen de esos valores.
- **Evidencia:** hay 1892 tweets con favoritos (46,85% del total) y 616 con retweets (15,26%). En ambos casos la mediana es 2. El percentil 90 es 10 en favoritos y 8,5 en retweets.
- **Decisión:** ✅ Aprobada, como complemento de J-02
- **Justificación:** deja ver cómo se reparten los tweets que sí interactúan: la mayoría tiene entre 1 y 2 y la frecuencia cae de forma gradual hasta los extremos. No muestra los ceros, así que hay que aclararlo y acompañarlo con el porcentaje de ceros y la opción A.

### J-04 · Comparar la proporción de tweets con interacción y los percentiles de los tweets con interacción por clase
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #2 / #30
- **Propuesta:** responder si los tweets positivos reciben más interacción que los negativos con dos medidas complementarias: el porcentaje de tweets con valor mayor a 0 (cuántos reciben interacción) y los percentiles 50, 75, 90 y 99 de los tweets con valor mayor a 0 (cuánta reciben). Se incluye la clase NEU como referencia.
- **Cómo se probó:** cálculo de ambas medidas por clase para `favorite_count` y `retweet_count`.
- **Evidencia:** tweets con favoritos: P 45,71%, NEU 47,86%, N 43,75%. Tweets con retweets: P 13,34%, NEU 16,03%, N 14,19%. Mediana de los tweets con interacción: favoritos P 1, NEU 2, N 2; retweets P 2, NEU 2, N 1.
- **Decisión:** ✅ Aprobada
- **Justificación:** son medidas relativas, así que no dependen del tamaño de cada clase, y no se ven arrastradas por valores extremos. Muestran diferencias pequeñas cuyo sentido se invierte según el indicador, por lo que no se observa que los positivos reciban más interacción que los negativos.

### J-05 · Comparar la media y el máximo de interacción por clase
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #2 / #30
- **Propuesta:** comparar las clases con la media y el máximo de `favorite_count` y `retweet_count`.
- **Cómo se probó:** media, mediana y máximo por clase, y media recalculada sin el tweet con el valor máximo de cada clase.
- **Evidencia:** la media de favoritos de P es 5,02 y la de N, 2,97. Sin el tweet máximo de cada clase, pasan a 2,06 y 2,38. En retweets, de 1,20 y 0,68 pasan a 0,54 y 0,54. El máximo de P es 2243 favoritos, un solo tweet.
- **Decisión:** ❌ Descartada
- **Justificación:** quitando un solo tweet, la media cambia de sentido. El máximo representa un único tweet y depende del tamaño del grupo. Ninguna de las dos describe el comportamiento de la clase.

### J-06 · Identificar los outliers de forma descriptiva
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #3 / #30
- **Propuesta:** responder si se observan outliers sin fijar un umbral, cuantificando qué tan lejos están los valores extremos del resto y verificando que correspondan a valores reales y no a errores de carga.
- **Cómo se probó:** mínimos de cada métrica, relación entre el máximo y el percentil 99, y revisión de los tweets con más interacción junto con los datos de su cuenta.
- **Evidencia:** el mínimo de ambas métricas es 0. El máximo es 2243 favoritos y 502 retweets, unas 55 y 34 veces el percentil 99. Los cinco tweets con más favoritos son también los cinco con más retweets, y dos de ellos son de cuentas verificadas con más de 140.000 seguidores (la mediana del dataset es 622).
- **Decisión:** ✅ Aprobada
- **Justificación:** las reglas automáticas no dan un criterio confiable para esta distribución (J-07), y un corte por percentil depende de un umbral arbitrario (p95, p98 o p99 dan respuestas distintas). La evidencia muestra que hay outliers y que son tweets reales con alta difusión, parte de la cola larga de la distribución. Se resigna una lista exacta de outliers, que no hace falta para responder la pregunta; su tratamiento se decide en el modelado.

### J-07 · Detectar outliers con la regla del IQR y el puntaje z
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #3 / #30
- **Propuesta:** marcar como outliers los valores mayores que Q3 + 1,5 × (Q3 − Q1), sobre los datos originales y sobre `log1p`, o los que están a más de 3 desvíos estándar por encima de la media.
- **Cómo se probó:** aplicación de cada regla a `favorite_count` y `retweet_count`, y desvío estándar recalculado sin el tweet con el valor máximo.
- **Evidencia:** con el IQR, en retweets Q1 = Q3 = 0 y el límite es 0, así que se marca el 15,26% de los tweets (todos los que tienen al menos un retweet); en favoritos se marca el 16,79%. Sobre `log1p`, el límite de retweets sigue en 0 y en favoritos se marca el 10,15%. El puntaje z marca 11 tweets en favoritos y 14 en retweets, pero sacando un solo tweet el desvío de favoritos baja de 45,04 a 28,05.
- **Decisión:** ❌ Descartada
- **Justificación:** el IQR colapsa porque la mayoría de los valores es 0. El puntaje z supone una distribución aproximadamente normal, y su límite depende de la media y el desvío, que se calculan con los mismos extremos que busca detectar.

### J-08 · Relacionar la interacción con el tamaño de la cuenta
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #2 / #30
- **Propuesta:** complementar la respuesta del #2 analizando si la interacción depende de la cuenta que publica el tweet: correlación con `user_followers_count` y comparación de la proporción de clases y la mediana de seguidores en distintos niveles de interacción.
- **Cómo se probó:** correlación de Pearson sobre `log1p` entre `favorite_count`, `retweet_count` y `user_followers_count`. Proporción de clases y mediana de seguidores en los tweets sin interacción, con interacción, en el 5% y en el 1% con más interacción, para `favorite_count` y `retweet_count`.
- **Evidencia:** sobre `log1p`, la correlación con los seguidores es 0,41 en favoritos y 0,33 en retweets. La mediana de seguidores pasa de 274 (sin favoritos) a 18.932 (1% con más favoritos) y de 473 (sin retweets) a 25.586 (1% con más retweets), mientras que la proporción de clases queda a no más de 4 puntos de la del dataset completo (P 18,7%, NEU 66,6%, N 14,7%) en todos los cortes.
- **Decisión:** ✅ Aprobada
- **Justificación:** se usa `log1p` porque sobre los valores originales la correlación queda dominada por los extremos (J-09). Usar varios cortes evita depender de un único umbral arbitrario para definir un tweet viral (J-06). El resultado muestra que la interacción está asociada al tamaño de la cuenta y no al sentimiento, lo que refuerza la conclusión de J-04 y explica el origen de los valores extremos (J-06).

### J-09 · Calcular la correlación con los seguidores sobre los valores originales
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #2 / #30
- **Propuesta:** medir la relación entre la interacción y los seguidores con la correlación de Pearson sobre los valores originales, sin transformar.
- **Cómo se probó:** matriz de correlación de `favorite_count`, `retweet_count` y `user_followers_count` sin transformar, comparada con la calculada sobre `log1p` (J-08), y recalculada sin el tweet con más favoritos.
- **Evidencia:** sin transformar, la correlación con los seguidores es 0,04 en favoritos y 0,05 en retweets, frente a 0,41 y 0,33 sobre `log1p`. Entre favoritos y retweets es 0,94, y quitar solo el tweet con más favoritos la baja a 0,84. La cuenta con más seguidores (más de 10 millones) tiene apenas 35 favoritos.
- **Decisión:** ❌ Descartada
- **Justificación:** la correlación de Pearson es sensible a los valores extremos, y en estas variables unos pocos tweets y cuentas están muy alejados del resto, así que el resultado depende de ellos y no del conjunto. Sobre `log1p` cada tweet pesa de forma más pareja.

## 3. Ingeniería de features

## 4. Preprocesamiento de texto

## 5. Modelado

## 6. Evaluación
