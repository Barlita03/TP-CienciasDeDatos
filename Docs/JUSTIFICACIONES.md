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

### J-10 · Longitud del tweet en caracteres, sin URLs ni menciones
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #4 / #33
- **Propuesta:** crear la feature `largo_texto`, la cantidad de caracteres del tweet después de quitar las URLs y las menciones, para medir solo lo que escribió la persona.
- **Cómo se probó:** percentiles 25, 50 y 75 y boxplot de `largo_texto` por clase.
- **Evidencia:** medianas de 182 (P), 163 (NEU) y 171 (N) caracteres, y percentiles 75 de 245, 242 y 248. El percentil 25 de NEU es 71, frente a 105 en P y 96 en N. 52 tweets quedan con largo 0 porque solo tenían links o menciones.
- **Decisión:** ✅ Aprobada
- **Justificación:** P y N tienen una longitud casi igual, así que la feature no ayuda a separarlos, pero NEU tiene más tweets cortos, por lo que puede aportar para distinguir a los neutrales. Quitar URLs y menciones evita medir contenido que no es texto propio (J-11). Su aporte real se evalúa en el modelado.

### J-11 · Longitud en caracteres del texto original
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #4 / #33
- **Propuesta:** medir la longitud como la cantidad de caracteres del texto tal como viene, sin quitar nada.
- **Cómo se probó:** percentiles 25, 50 y 75 y boxplot por clase, comparados con J-10, y cantidad de tweets que superan el límite de 280 caracteres de Twitter.
- **Evidencia:** medianas de 192 (P), 184 (NEU) y 193 (N). El percentil 25 de NEU es un 19% más bajo que el de P (95 frente a 118), contra un 32% en J-10. 464 tweets superan los 280 caracteres y el 51% de ellos empieza con una mención; sin URLs ni menciones, solo 3 quedan por encima.
- **Decisión:** ❌ Descartada
- **Justificación:** la medida se infla con links y menciones, que no son texto escrito por la persona, y eso achica la diferencia de los neutrales que muestra J-10.

### J-12 · Longitud en palabras, sin URLs ni menciones
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #4 / #33
- **Propuesta:** medir la longitud como la cantidad de palabras del texto sin URLs ni menciones.
- **Cómo se probó:** percentiles 25, 50 y 75 y boxplot por clase, comparados con J-10.
- **Evidencia:** medianas de 28 (P), 27 (NEU) y 29 (N) palabras. El percentil 25 de NEU es 12, frente a 15 en P y 16 en N: un 20% más bajo que P, contra un 32% en caracteres (J-10).
- **Decisión:** ❌ Descartada
- **Justificación:** muestra el mismo patrón que contar caracteres pero con una diferencia menos marcada, así que no agrega información.

### J-13 · Cantidad de hashtags del tweet
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #5 / #33
- **Propuesta:** crear la feature `cant_hashtags`, la cantidad de palabras del tweet que empiezan con `#`.
- **Cómo se probó:** porcentaje de tweets con al menos un hashtag y media por clase, proporción de clases según la cantidad de hashtags (0, 1, 2 y 3 o más) y correlación con `largo_texto` sobre `log1p`.
- **Evidencia:** el 15,3% de los tweets tiene hashtags y el máximo es 21. Tiene algún hashtag el 23,5% de los positivos, el 14,8% de los neutrales y el 6,9% de los negativos. Los positivos pasan de ser el 16,9% de los tweets sin hashtags al 37,4% de los que tienen 2. Con 3 o más hashtags, el 71,4% es neutral y solo el 2,5% es negativo (contra 66,6% y 14,7% en el dataset). La correlación con `largo_texto` es de 0,17.
- **Decisión:** ✅ Aprobada
- **Justificación:** un tweet con hashtags rara vez es negativo, así que la feature puede ayudar a separar positivos de negativos, algo que la longitud no logra (J-10). Al tener una correlación baja con `largo_texto`, no repite su información. Los valores altos son reales (tweets de difusión con muchos hashtags) y se mantienen. Su aporte real se evalúa en el modelado.

### J-14 · Feature binaria: si el tweet tiene o no hashtags
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #5 / #33
- **Propuesta:** en lugar de contar los hashtags, indicar solo si el tweet tiene alguno.
- **Cómo se probó:** proporción de clases en los tweets con y sin hashtags, comparada con J-13.
- **Evidencia:** entre los tweets con hashtags hay un 28,9% de positivos y un 6,7% de negativos, contra 16,9% y 16,1% en los que no tienen. Pero el grupo "con hashtags" mezcla a los tweets con 1 hashtag con los que tienen 3 o más, donde la proporción de neutrales sube al 71,4%.
- **Decisión:** ❌ Descartada
- **Justificación:** capta la misma diferencia entre P y N que la cantidad, pero pierde el patrón de los tweets con muchos hashtags. La cantidad ya incluye esta información, porque un modelo puede separar 0 de "1 o más" a partir de ella.

### J-15 · Cantidad de hashtags sin los del tema del dataset
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #5 / #33
- **Propuesta:** no contar los hashtags sobre el tema del dataset (`#ia`, `#ai`, `#inteligenciaartificial`, `#artificialintelligence`, `#chatgpt`, `#openai`), que son los más usados y podrían aparecer por cómo se recolectaron los tweets y no por su sentimiento.
- **Cómo se probó:** las mismas tablas que en J-13, con la cantidad de hashtags sin los del tema.
- **Evidencia:** los tweets con hashtags bajan de 616 a 461. Tiene algún hashtag el 18,9% de los positivos, el 10,7% de los neutrales y el 5,1% de los negativos: los positivos los usan 3,7 veces más que los negativos, contra 3,4 veces contando todos.
- **Decisión:** ❌ Descartada
- **Justificación:** el patrón casi no cambia, quedan menos tweets con información y la feature pasa a depender de una lista armada a mano, que habría que justificar. La mejora es mínima frente a la decisión arbitraria que agrega.

### J-16 · Cantidad de menciones al inicio del tweet, con tope en 2
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #6 / #33
- **Propuesta:** crear la feature `menciones_inicio`, la cantidad de menciones con las que empieza el tweet (así se ven las respuestas en Twitter), con tope en 2.
- **Cómo se probó:** proporción de clases según las menciones al inicio (0, 1 y 2 o más) y correlación con `largo_texto` y `cant_hashtags` sobre `log1p`.
- **Evidencia:** el 44,3% de los tweets empieza con una mención. Los negativos son el 9,8% de los tweets sin menciones al inicio, el 19,2% de los que tienen 1 y el 23,4% de los que tienen 2 o más; los positivos, el 22,7%, el 15,5% y el 10,9%. La correlación es de -0,41 con `largo_texto` y de -0,27 con `cant_hashtags`.
- **Decisión:** ✅ Aprobada
- **Justificación:** la proporción de negativos sube con cada mención al inicio, así que la feature puede ayudar a reconocerlos. Contar solo las del inicio evita mezclarlas con las del resto del texto, que se comportan al revés (J-17 y J-20). Contarlas conserva la diferencia entre empezar con 1 o con 2 o más (J-18), y el tope evita que los pocos tweets con muchas menciones pesen demasiado (J-19). Las correlaciones con las otras features son moderadas, así que no repiten su información. El dataset no indica qué tweets son respuestas, por lo que es una aproximación. Su aporte real se evalúa en el modelado.

### J-17 · Cantidad total de menciones
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #6 / #33
- **Propuesta:** contar todas las menciones del tweet, sin distinguir dónde están.
- **Cómo se probó:** proporción de clases según la cantidad total de menciones (0, 1, 2, 3-4 y 5 o más).
- **Evidencia:** la proporción de negativos sube de 10,4% sin menciones a 21,2% con 3-4, pero con 5 o más baja a 14,1%; la de positivos baja de 23,4% a 9,6% y después sube a 20,3%.
- **Decisión:** ❌ Descartada
- **Justificación:** mezcla las menciones del inicio, que tienden a ser negativas, con las del resto del texto, que tienden a ser neutrales, y los dos efectos se tapan entre sí.

### J-18 · Feature binaria: si el tweet es una respuesta
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #6 / #33
- **Propuesta:** en lugar de contar las menciones del inicio, marcar solo si el tweet empieza con alguna (`es_respuesta`).
- **Cómo se probó:** proporción de clases en los tweets que empiezan y no empiezan con una mención, comparada con J-16.
- **Evidencia:** los negativos son el 20,8% de los tweets que empiezan con una mención y el 9,8% de los que no. Pero dentro de los primeros, son el 19,2% con 1 mención al inicio (1100 tweets) y el 23,4% con 2 o más (689 tweets).
- **Decisión:** ❌ Descartada
- **Justificación:** es la misma información que `menciones_inicio` con los grupos 1 y "2 o más" juntos. Capta el salto más grande, pero pierde una diferencia que se apoya en grupos grandes, y contar las menciones no es más difícil de calcular ni de explicar.

### J-19 · Cantidad de menciones al inicio sin tope
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #6 / #33
- **Propuesta:** usar la cantidad de menciones al inicio tal cual, sin tope.
- **Cómo se probó:** resumen de la variable y proporción de clases para cada cantidad de menciones al inicio.
- **Evidencia:** el percentil 99 es 5 y el máximo es 21. Los negativos suben hasta las 3 menciones (26,6%), pero desde 4 los grupos tienen entre 13 y 49 tweets y los porcentajes varían sin un patrón (20,4%, 23,1% y 14,3%).
- **Decisión:** ❌ Descartada
- **Justificación:** más allá de las 2 o 3 menciones no hay un patrón claro, y sin tope un modelo lineal (regresión logística o SVM) le daría a un tweet con 21 menciones un peso mucho mayor que a uno con 1 o 2. Los árboles sí podrían usarla sin problema, así que se puede comparar con J-16 en el modelado.

### J-20 · Cantidad de menciones en el resto del texto como segunda feature
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #6 / #33
- **Propuesta:** sumar a `menciones_inicio` una segunda feature, `cant_menciones_texto`, con la cantidad de menciones que no están al inicio del tweet.
- **Cómo se probó:** proporción de clases según las menciones en el resto del texto (0, 1 y 2 o más) y correlación con `menciones_inicio` sobre `log1p`.
- **Evidencia:** solo el 8,7% de los tweets (353) tiene menciones en el resto del texto, y el máximo es 9. A medida que aumentan, sube la proporción de neutrales (65,8%, 71,6% y 79,7%) y baja la de negativos (de 15,4% a 2,3%). La correlación con `menciones_inicio` es de -0,18.
- **Decisión:** ❌ Descartada
- **Justificación:** muestra una señal propia, distinta de la de las menciones al inicio, pero afecta a pocos tweets, así que por ahora se prioriza una sola feature más simple. Se puede volver a sumar si en el modelado hace falta.

### J-21 · Si el tweet tiene signos de exclamación
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #31 / #33
- **Propuesta:** crear la feature `tiene_exclamacion`, que vale 1 si el tweet tiene al menos un `!`, contado sin las URLs.
- **Cómo se probó:** proporción de clases en los tweets con y sin `!`, cantidad de tweets con `¡` pero sin `!`, y correlación con `largo_texto`, `cant_hashtags` y `menciones_inicio` sobre `log1p`.
- **Evidencia:** el 8,7% de los tweets tiene `!`. Entre ellos, el 53,8% es positivo, el 35,9% neutral y el 10,3% negativo, contra 15,4%, 69,5% y 15,1% en los que no tienen. Solo 2 tweets tienen `¡` sin `!`. La correlación con las otras features está entre -0,01 y 0,06.
- **Decisión:** ✅ Aprobada
- **Justificación:** es la diferencia entre clases más grande de las features analizadas: los tweets con `!` son positivos más del triple de veces que el resto. No se correlaciona con las otras features, así que aporta información nueva. El `¡` no se cuenta porque casi siempre va con un `!`. Su aporte real se evalúa en el modelado.

### J-22 · Cantidad de signos de exclamación
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #31 / #33
- **Propuesta:** contar los `!` de cada tweet, con tope en 3, en lugar de marcar solo si hay alguno.
- **Cómo se probó:** resumen de la variable y proporción de clases según la cantidad de `!` (0, 1, 2 y 3 o más).
- **Evidencia:** el percentil 99 es 2 y el máximo, 9. Los positivos son el 58,1% de los tweets con un `!`, el 44,7% de los que tienen 2 y el 35,9% de los que tienen 3 o más, donde los negativos también son el 35,9%. Los grupos con 2 y con 3 o más tienen 47 y 39 tweets.
- **Decisión:** ❌ Descartada
- **Justificación:** la relación sube y baja: un `!` suele acompañar a un tweet entusiasta, pero muchos también aparecen en críticas o quejas. Con grupos tan chicos, contar no agrega información confiable respecto de marcar si hay algún `!`.

### J-23 · Signos de exclamación repetidos como segunda feature
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #31 / #33
- **Propuesta:** sumar a `tiene_exclamacion` una segunda feature que marque si el tweet tiene `!` repetidos (`!!`).
- **Cómo se probó:** proporción de clases en los tweets con y sin `!!`.
- **Evidencia:** 52 tweets tienen `!!`. Entre ellos, el 32,7% es negativo, contra el 14,4% del resto.
- **Decisión:** ❌ Descartada
- **Justificación:** muestra una señal hacia los negativos, pero afecta a muy pocos tweets, el mismo criterio que en J-20. Se puede volver a sumar si en el modelado hace falta.

### J-24 · Cantidad de signos de pregunta, con tope en 2
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #32 / #33
- **Propuesta:** crear la feature `cant_preguntas`, la cantidad de `?` del tweet contados sin las URLs, con tope en 2.
- **Cómo se probó:** proporción de clases según la cantidad de `?` (0, 1 y 2 o más), cantidad de tweets con `¿` pero sin `?`, y correlación con `largo_texto`, `cant_hashtags`, `menciones_inicio` y `tiene_exclamacion` sobre `log1p`.
- **Evidencia:** el 12,4% de los tweets tiene `?`. Los positivos son el 20,1% de los tweets sin preguntas, el 11,2% de los que tienen una y el 1,8% de los que tienen 2 o más (109 tweets). Los neutrales son el 64,9%, el 78,6% y el 78,9%. Ningún tweet tiene `¿` sin `?`. La correlación con las otras features está entre 0,00 y 0,05.
- **Decisión:** ✅ Aprobada
- **Justificación:** la proporción de positivos baja con cada `?`, así que la relación va en una sola dirección y se usa la cantidad, el mismo criterio que con las menciones al inicio (J-16). El tope evita que los pocos tweets con muchas preguntas pesen demasiado (J-26). No se correlaciona con las otras features, así que aporta información nueva. Su aporte real se evalúa en el modelado.

### J-25 · Feature binaria: si el tweet tiene signos de pregunta
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #32 / #33
- **Propuesta:** marcar solo si el tweet tiene algún `?`, en lugar de contarlos.
- **Cómo se probó:** proporción de clases en los tweets con y sin `?`, comparada con J-24.
- **Evidencia:** los positivos son el 9,2% de los tweets con `?` y el 20,1% de los que no tienen. Dentro de los primeros, son el 11,2% con una pregunta y el 1,8% con 2 o más.
- **Decisión:** ❌ Descartada
- **Justificación:** junta a los tweets con una pregunta y con varias, y pierde que con 2 o más casi no hay positivos.

### J-26 · Cantidad de signos de pregunta sin tope
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #32 / #33
- **Propuesta:** usar la cantidad de `?` tal cual, sin tope.
- **Cómo se probó:** resumen de la variable y proporción de clases para cada cantidad de `?`.
- **Evidencia:** el percentil 99 es 2 y el máximo, 7. Los tweets con 3 `?` son 20 y los que tienen 4 o más, 17.
- **Decisión:** ❌ Descartada
- **Justificación:** desde 3 preguntas los grupos son demasiado chicos para sacar conclusiones, y sin tope los valores altos pesarían más que el resto en un modelo lineal.

### J-27 · Signos de pregunta repetidos como segunda feature
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #32 / #33
- **Propuesta:** sumar a `cant_preguntas` una segunda feature que marque si el tweet tiene `?` repetidos (`??`).
- **Cómo se probó:** proporción de clases en los tweets con y sin `??`.
- **Evidencia:** 22 tweets tienen `??`. Ninguno es positivo y el 27,3% es negativo.
- **Decisión:** ❌ Descartada
- **Justificación:** afecta a muy pocos tweets, el mismo criterio que en J-23.

## 4. Preprocesamiento de texto

## 5. Modelado

## 6. Evaluación
