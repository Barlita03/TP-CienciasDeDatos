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

### J-28 · Quitar los tweets con el texto exactamente repetido
- **Etapa:** 1. Exploración del dataset
- **Issue / PR:** — / #33
- **Propuesta:** dejar una sola copia de cada texto antes de dividir el dataset, para que un tweet repetido no pese varias veces en los resultados ni quede en train y en test a la vez.
- **Cómo se probó:** cantidad de textos repetidos, clases de sus copias y proporción de clases antes y después de quitarlos.
- **Evidencia:** hay 11 textos repetidos y 57 filas de más, y ningún texto repetido tiene más de una clase. Uno solo aparece 48 veces, siempre como positivo. El dataset pasa de 4038 a 3981 tweets, y la proporción de clases casi no cambia: P pasa de 18,7% a 17,8%, NEU de 66,6% a 67,3% y N de 14,7% a 14,9%.
- **Decisión:** ✅ Aprobada
- **Justificación:** es una sola línea, evita que esas copias inflen los porcentajes y que el modelo se evalúe con tweets que ya vio al entrenar (J-29). Se aplica antes de la división (J-55), así que todo el análisis usa el dataset sin repetidos.

### J-29 · Mantener los tweets repetidos
- **Etapa:** 1. Exploración del dataset
- **Issue / PR:** — / #33
- **Propuesta:** no quitar nada y usar los 4038 tweets.
- **Cómo se probó:** división de los 4038 tweets igual que en J-55 y cantidad de filas de test cuyo texto también está en train.
- **Evidencia:** 11 de las 808 filas de test tienen un texto que también está en train.
- **Decisión:** ❌ Descartada
- **Justificación:** el modelo se evaluaría en parte con tweets que ya vio al entrenar, lo que haría parecer mejor su desempeño. Además, un solo texto pesaría 48 veces en los porcentajes del análisis.

### J-30 · Quitar también los tweets casi iguales
- **Etapa:** 1. Exploración del dataset
- **Issue / PR:** — / #33
- **Propuesta:** tratar como repetidos a los tweets que solo difieren en links, menciones, mayúsculas o espacios.
- **Cómo se probó:** cantidad de filas de más con este criterio y clases de los textos más repetidos.
- **Evidencia:** sobran 89 filas, contra 57 en J-28. Entre los repetidos aparecen frases cortas como "totalmente de acuerdo" (3 P y 1 NEU) y tweets con solo emojis.
- **Decisión:** ❌ Descartada
- **Justificación:** agrega un criterio más complejo y puede borrar tweets que no son copias, a cambio de quitar solo 32 filas más.

### J-33 · Dividir en train, validación y test de forma estratificada
- **Etapa:** 1. Exploración del dataset
- **Issue / PR:** — / #33
- **Propuesta:** dividir el dataset sin repetidos en train (70%), validación (10%) y test (20%), estratificando por `polarity`, y hacer todo el análisis solo sobre train.
- **Cómo se probó:** dos `train_test_split` con `stratify` y `random_state=42`: primero el 20% para test y, del resto, el 12,5% para validación (el 10% del total). Tamaño y proporción de clases de cada set.
- **Evidencia:** quedan 2786 tweets en train, 398 en validación y 797 en test. La proporción de clases es la misma en los tres: P 17,8%, NEU 67,3%-67,4% y N 14,8%-14,9%. En validación hay 59 negativos.
- **Decisión:** ❌ Descartada
- **Justificación:** validación queda muy chica para elegir hiperparámetros: con 59 negativos, un solo error en un negativo mueve el recall de N en 1,7 puntos, así que el resultado depende mucho de qué tweets tocaron. Además, esos 398 tweets no se usan para entrenar. Se reemplaza por la validación cruzada sobre un train del 80% (J-55).

### J-55 · Dividir en train (80%) y test (20%) y ajustar con validación cruzada
- **Etapa:** 1. Exploración del dataset
- **Issue / PR:** — / #XX
- **Propuesta:** dividir el dataset sin repetidos en train (80%) y test (20%), estratificando por `polarity`, y hacer todo el análisis solo sobre train. En el modelado, comparar modelos y ajustar hiperparámetros con validación cruzada estratificada de 5 partes sobre train (`StratifiedKFold(n_splits=5, shuffle=True, random_state=42)`).
- **Cómo se probó:** un `train_test_split` con `stratify` y `random_state=42`. Tamaño y proporción de clases de cada set, comparados con la división con un set de validación aparte (J-33).
- **Evidencia:** quedan 3184 tweets en train y 797 en test, con la misma proporción de clases: P 17,8%, NEU 67,3%-67,4% y N 14,8%-14,9%. Train tiene 474 negativos, contra 59 en el set de validación de J-33.
- **Decisión:** ✅ Aprobada
- **Justificación:** si el análisis mira test, las decisiones quedan influidas por esos datos y la evaluación deja de mostrar cómo le iría al modelo con tweets nuevos, así que test se usa solo para la evaluación final. Con validación cruzada, cada combinación de hiperparámetros se evalúa 5 veces, con partes de unos 637 tweets (unos 95 negativos), y se promedia, así que el resultado depende menos de qué tweets tocaron; además, todo train se usa para entrenar. El vectorizador del texto va dentro de un `Pipeline`, para que en cada parte el vocabulario se aprenda solo con la porción de entrenamiento. Estratificar asegura la misma proporción de clases en los dos sets, algo importante por el desbalance (J-34). La evidencia de J-01 a J-32 y de J-35 a J-54 se calcula sobre este train.

### J-34 · Dividir sin estratificar
- **Etapa:** 1. Exploración del dataset
- **Issue / PR:** — / #33
- **Propuesta:** hacer la misma división 80/20 sin `stratify`.
- **Cómo se probó:** proporción de clases de cada set con `random_state=42`, comparada con la del total.
- **Evidencia:** la proporción queda cerca de la del total, pero no igual: la mayor diferencia es de 0,9 puntos, en los positivos de test (16,9% contra 17,8%).
- **Decisión:** ❌ Descartada
- **Justificación:** deja la proporción de clases librada al azar, sin ninguna ventaja frente a estratificar, que lo resuelve en una línea.

## 2. Análisis de las variables disponibles

La evidencia de esta etapa y de la siguiente se calcula sobre el train set (J-55).

### J-01 · Histograma sin transformar y normalización min-max para graficar `favorite_count` y `retweet_count`
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #1 / #30
- **Propuesta:** graficar las métricas de interacción con un histograma común y, para mejorar la visualización, normalizarlas con min-max.
- **Cómo se probó:** histograma de cada variable sin transformar y de su versión normalizada con (x − min) / (max − min), y porcentaje de datos que cae en el primer bin.
- **Evidencia:** sin transformar, el 99,53% de `favorite_count` y el 99,15% de `retweet_count` cae en el primer bin. Con min-max los porcentajes son los mismos y el histograma tiene la misma forma, solo cambia el eje X a [0, 1].
- **Decisión:** ❌ Descartada
- **Justificación:** el histograma común muestra una sola barra y la cola no se ve. Min-max cambia la escala pero no la forma de la distribución, y depende del máximo, que es un valor extremo (2243 y 502), así que no ayuda a visualizarla.

### J-02 · Transformación `log1p` con todos los datos (opción A)
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #1 / #30
- **Propuesta:** graficar `log1p(x)` = log(1 + x) de todos los tweets, para comprimir la cola. Se usa `log1p` y no `log` porque la mayoría de los valores son 0 y log(0) no existe.
- **Cómo se probó:** histograma de `log1p` de cada variable, junto con el porcentaje de ceros y el resumen estadístico.
- **Evidencia:** el 52,01% de los tweets tiene 0 favoritos y el 84,48% tiene 0 retweets. Media vs. mediana: 4,20 vs. 0 en favoritos y 0,95 vs. 0 en retweets. El máximo es 2243 en favoritos y 502 en retweets.
- **Decisión:** ✅ Aprobada, como complemento de J-03
- **Justificación:** muestra todos los tweets y la proporción real de tweets sin interacción. Pero la barra del 0 domina el gráfico y el cuerpo de los tweets con interacción se distingue poco, por eso no alcanza sola.

### J-03 · Filtrar los tweets con interacción y usar eje logarítmico (opción B)
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #1 / #30
- **Propuesta:** graficar solo los tweets con valor mayor a 0, con el eje X en escala logarítmica, para ver la forma de la distribución sin que los ceros la tapen.
- **Cómo se probó:** histograma con `log_scale=True` de los valores mayores a 0 de cada variable, y resumen de esos valores.
- **Evidencia:** hay 1528 tweets con favoritos (47,99% del total) y 494 con retweets (15,52%). En ambos casos la mediana es 2. El percentil 90 es 10 en favoritos y 8 en retweets.
- **Decisión:** ✅ Aprobada, como complemento de J-02
- **Justificación:** deja ver cómo se reparten los tweets que sí interactúan: la mayoría tiene entre 1 y 2 y la frecuencia cae de forma gradual hasta los extremos. No muestra los ceros, así que hay que aclararlo y acompañarlo con el porcentaje de ceros y la opción A.

### J-04 · Comparar la proporción de tweets con interacción y los percentiles de los tweets con interacción por clase
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #2 / #30
- **Propuesta:** responder si los tweets positivos reciben más interacción que los negativos con dos medidas complementarias: el porcentaje de tweets con valor mayor a 0 (cuántos reciben interacción) y los percentiles 50, 75, 90 y 99 de los tweets con valor mayor a 0 (cuánta reciben). Se incluye la clase NEU como referencia.
- **Cómo se probó:** cálculo de ambas medidas por clase para `favorite_count` y `retweet_count`.
- **Evidencia:** tweets con favoritos: P 48,06%, NEU 48,88%, N 43,88%. Tweets con retweets: P 13,43%, NEU 16,37%, N 14,14%. Mediana de los tweets con interacción: favoritos P 1, NEU 2, N 2; retweets P 2, NEU 2, N 1. Percentil 75 de favoritos: P 3, NEU 4, N 4.
- **Decisión:** ✅ Aprobada
- **Justificación:** son medidas relativas, así que no dependen del tamaño de cada clase, y no se ven arrastradas por valores extremos. Ninguna de las dos clases queda por encima en todos los indicadores: P tiene más tweets con favoritos (4,2 puntos más), pero N tiene la mediana y el percentil 75 de favoritos algo más altos y un poco más de tweets con retweets (0,7 puntos más). Las diferencias son chicas y no van en una sola dirección, así que no hay evidencia clara de que el sentimiento influya.

### J-05 · Comparar la media y el máximo de interacción por clase
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #2 / #30
- **Propuesta:** comparar las clases con la media y el máximo de `favorite_count` y `retweet_count`.
- **Cómo se probó:** media, mediana y máximo por clase, y media recalculada sin el tweet con el valor máximo de cada clase.
- **Evidencia:** la media de favoritos de P es 6,07 y la de N, 3,14. Sin el tweet máximo de cada clase, pasan a 2,11 y 2,41. En retweets, de 1,48 y 0,74 pasan a 0,60 y 0,56. El máximo de P es 2243 favoritos, un solo tweet.
- **Decisión:** ❌ Descartada
- **Justificación:** quitando un solo tweet, en favoritos N pasa a quedar por encima de P y en retweets la diferencia se achica de 0,74 a 0,04. El máximo representa un único tweet y depende del tamaño del grupo. Ninguna de las dos describe el comportamiento de la clase.

### J-06 · Identificar los outliers de forma descriptiva
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #3 / #30
- **Propuesta:** responder si se observan outliers sin fijar un umbral, cuantificando qué tan lejos están los valores extremos del resto y verificando que correspondan a valores reales y no a errores de carga.
- **Cómo se probó:** mínimos de cada métrica, relación entre el máximo y el percentil 99, y revisión de los tweets con más interacción junto con los datos de su cuenta.
- **Evidencia:** el mínimo de ambas métricas es 0. El máximo es 2243 favoritos y 502 retweets, unas 59 y 38 veces el percentil 99. Los cinco tweets con más favoritos son también los cinco con más retweets, y dos de ellos son de cuentas verificadas con más de 140.000 seguidores (la mediana de train es 642).
- **Decisión:** ✅ Aprobada
- **Justificación:** las reglas automáticas no dan un criterio confiable para esta distribución (J-07), y un corte por percentil depende de un umbral arbitrario (p95, p98 o p99 dan respuestas distintas). La evidencia muestra que hay outliers y que son tweets reales con alta difusión, parte de la cola larga de la distribución. Se resigna una lista exacta de outliers, que no hace falta para responder la pregunta; su tratamiento se decide en el modelado.

### J-07 · Detectar outliers con la regla del IQR y el puntaje z
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #3 / #30
- **Propuesta:** marcar como outliers los valores mayores que Q3 + 1,5 × (Q3 − Q1), sobre los datos originales y sobre `log1p`, o los que están a más de 3 desvíos estándar por encima de la media.
- **Cómo se probó:** aplicación de cada regla a `favorite_count` y `retweet_count`, y desvío estándar recalculado sin el tweet con el valor máximo.
- **Evidencia:** con el IQR, en retweets Q1 = Q3 = 0 y el límite es 0, así que se marca el 15,52% de los tweets (todos los que tienen al menos un retweet); en favoritos se marca el 8,67%. Sobre `log1p`, el límite de retweets sigue en 0 y en favoritos se marca el 3,17%. El puntaje z marca 11 tweets en favoritos y 12 en retweets, pero sacando un solo tweet el desvío de favoritos baja de 50,49 a 31,21.
- **Decisión:** ❌ Descartada
- **Justificación:** el IQR colapsa porque la mayoría de los valores es 0. El puntaje z supone una distribución aproximadamente normal, y su límite depende de la media y el desvío, que se calculan con los mismos extremos que busca detectar.

### J-08 · Relacionar la interacción con el tamaño de la cuenta
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #2 / #30
- **Propuesta:** complementar la respuesta del #2 analizando si la interacción depende de la cuenta que publica el tweet: correlación con `user_followers_count` y comparación de la proporción de clases y la mediana de seguidores en distintos niveles de interacción.
- **Cómo se probó:** correlación de Pearson sobre `log1p` entre `favorite_count`, `retweet_count` y `user_followers_count`. Proporción de clases y mediana de seguidores en los tweets sin interacción, con interacción, en el 5% y en el 1% con más interacción, para `favorite_count` y `retweet_count`.
- **Evidencia:** sobre `log1p`, la correlación con los seguidores es 0,40 en favoritos y 0,33 en retweets. La mediana de seguidores pasa de 315 (sin favoritos) a 17.062 (1% con más favoritos) y de 486,5 (sin retweets) a 17.692 (1% con más retweets), mientras que la proporción de clases queda a no más de 6 puntos de la de todo train (P 17,8%, NEU 67,3%, N 14,9%) en todos los cortes.
- **Decisión:** ✅ Aprobada
- **Justificación:** se usa `log1p` porque sobre los valores originales la correlación queda dominada por los extremos (J-09). Usar varios cortes evita depender de un único umbral arbitrario para definir un tweet viral (J-06). El resultado muestra que la interacción está asociada al tamaño de la cuenta y no al sentimiento, lo que refuerza la conclusión de J-04 y explica el origen de los valores extremos (J-06).

### J-09 · Calcular la correlación con los seguidores sobre los valores originales
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #2 / #30
- **Propuesta:** medir la relación entre la interacción y los seguidores con la correlación de Pearson sobre los valores originales, sin transformar.
- **Cómo se probó:** matriz de correlación de `favorite_count`, `retweet_count` y `user_followers_count` sin transformar, comparada con la calculada sobre `log1p` (J-08), y recalculada sin el tweet con más favoritos.
- **Evidencia:** sin transformar, la correlación con los seguidores es 0,04 en favoritos y 0,05 en retweets, frente a 0,40 y 0,33 sobre `log1p`. Entre favoritos y retweets es 0,94, y quitar solo el tweet con más favoritos la baja a 0,84. La cuenta con más seguidores (más de 10 millones) tiene apenas 35 favoritos.
- **Decisión:** ❌ Descartada
- **Justificación:** la correlación de Pearson es sensible a los valores extremos, y en estas variables unos pocos tweets y cuentas están muy alejados del resto, así que el resultado depende de ellos y no del conjunto. Sobre `log1p` cada tweet pesa de forma más pareja.

## 3. Ingeniería de features

### J-10 · Longitud del tweet en caracteres, sin URLs ni menciones
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #4 / #33
- **Propuesta:** crear la feature `largo_texto`, la cantidad de caracteres del tweet después de quitar las URLs y las menciones, para medir solo lo que escribió la persona.
- **Cómo se probó:** percentiles 25, 50 y 75 y boxplot de `largo_texto` por clase.
- **Evidencia:** medianas de 187 (P), 163 (NEU) y 171 (N) caracteres, y percentiles 75 de 247,75, 241 y 250,75. El percentil 25 de NEU es 71, frente a 92,25 en P y 97 en N. 41 tweets quedan con largo 0 porque solo tenían links o menciones.
- **Decisión:** ✅ Aprobada
- **Justificación:** P y N tienen una longitud casi igual, así que la feature no ayuda a separarlos, pero NEU tiene más tweets cortos, por lo que puede aportar para distinguir a los neutrales. Quitar URLs y menciones evita medir contenido que no es texto propio (J-11). Su aporte real se evalúa en el modelado.

### J-11 · Longitud en caracteres del texto original
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #4 / #33
- **Propuesta:** medir la longitud como la cantidad de caracteres del texto tal como viene, sin quitar nada.
- **Cómo se probó:** percentiles 25, 50 y 75 y boxplot por clase, comparados con J-10, y cantidad de tweets que superan el límite de 280 caracteres de Twitter.
- **Evidencia:** medianas de 197 (P), 185 (NEU) y 195,5 (N). El percentil 25 de NEU es un 24% más bajo que el de P (92,75 frente a 122), casi igual que el 23% de J-10. 386 tweets superan los 280 caracteres y el 51% de ellos empieza con una mención; sin URLs ni menciones, solo 3 quedan por encima.
- **Decisión:** ❌ Descartada
- **Justificación:** la medida se infla con links y menciones, que no son texto escrito por la persona, y no separa mejor a los neutrales que J-10.

### J-12 · Longitud en palabras, sin URLs ni menciones
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #4 / #33
- **Propuesta:** medir la longitud como la cantidad de palabras del texto sin URLs ni menciones.
- **Cómo se probó:** percentiles 25, 50 y 75 y boxplot por clase, comparados con J-10.
- **Evidencia:** medianas de 29 (P), 27 (NEU) y 29 (N) palabras. El percentil 25 de NEU es 12, frente a 15 en P y 16,25 en N: un 20% más bajo que P, contra un 23% en caracteres (J-10).
- **Decisión:** ❌ Descartada
- **Justificación:** muestra el mismo patrón que contar caracteres con una diferencia algo menor, así que no agrega información.

### J-13 · Cantidad de hashtags del tweet
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #5 / #33
- **Propuesta:** crear la feature `cant_hashtags`, la cantidad de palabras del tweet que empiezan con `#`.
- **Cómo se probó:** porcentaje de tweets con al menos un hashtag y media por clase, proporción de clases según la cantidad de hashtags (0, 1, 2 y 3 o más) y correlación con `largo_texto` sobre `log1p`.
- **Evidencia:** el 15,5% de los tweets tiene hashtags y el máximo es 19. Tiene algún hashtag el 25,6% de los positivos, el 14,7% de los neutrales y el 7,2% de los negativos. Los positivos pasan de ser el 15,7% de los tweets sin hashtags al 41,5% de los que tienen 2. Con 3 o más hashtags, el 70,5% es neutral y solo el 2,3% es negativo (contra 67,3% y 14,9% en train). La correlación con `largo_texto` es de 0,18.
- **Decisión:** ✅ Aprobada
- **Justificación:** un tweet con hashtags rara vez es negativo, así que la feature puede ayudar a separar positivos de negativos, algo que la longitud no logra (J-10). Al tener una correlación baja con `largo_texto`, no repite su información. Los valores altos son reales (tweets de difusión con muchos hashtags) y se mantienen. Su aporte real se evalúa en el modelado.

### J-14 · Feature binaria: si el tweet tiene o no hashtags
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #5 / #33
- **Propuesta:** en lugar de contar los hashtags, indicar solo si el tweet tiene alguno.
- **Cómo se probó:** proporción de clases en los tweets con y sin hashtags, comparada con J-13.
- **Evidencia:** entre los tweets con hashtags hay un 29,4% de positivos y un 6,9% de negativos, contra 15,7% y 16,4% en los que no tienen. Pero el grupo "con hashtags" mezcla a los tweets con 1 hashtag con los que tienen 3 o más, donde la proporción de neutrales sube al 70,5%.
- **Decisión:** ❌ Descartada
- **Justificación:** capta la misma diferencia entre P y N que la cantidad, pero pierde el patrón de los tweets con muchos hashtags. La cantidad ya incluye esta información, porque un modelo puede separar 0 de "1 o más" a partir de ella.

### J-15 · Cantidad de hashtags sin los del tema del dataset
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #5 / #33
- **Propuesta:** no contar los hashtags sobre el tema del dataset (`#ia`, `#ai`, `#inteligenciaartificial`, `#artificialintelligence`, `#chatgpt`, `#openai`), que son los más usados y podrían aparecer por cómo se recolectaron los tweets y no por su sentimiento.
- **Cómo se probó:** las mismas tablas que en J-13, con la cantidad de hashtags sin los del tema.
- **Evidencia:** los tweets con hashtags bajan de 494 a 370. Tiene algún hashtag el 20,5% de los positivos, el 10,7% de los neutrales y el 5,3% de los negativos: los positivos los usan 3,9 veces más que los negativos, contra 3,6 veces contando todos.
- **Decisión:** ❌ Descartada
- **Justificación:** la separación entre positivos y negativos mejora apenas, pero a cambio quedan 124 tweets menos con información y la feature pasa a depender de una lista armada a mano, que habría que justificar.

### J-16 · Cantidad de menciones al inicio del tweet, con tope en 2
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #6 / #33
- **Propuesta:** crear la feature `menciones_inicio`, la cantidad de menciones con las que empieza el tweet (así se ven las respuestas en Twitter), con tope en 2.
- **Cómo se probó:** proporción de clases según las menciones al inicio (0, 1 y 2 o más) y correlación con `largo_texto` y `cant_hashtags` sobre `log1p`.
- **Evidencia:** el 44,9% de los tweets empieza con una mención. Los negativos son el 10,3% de los tweets sin menciones al inicio, el 18,9% de los que tienen 1 y el 23,3% de los que tienen 2 o más; los positivos, el 20,9%, el 15,6% y el 11,1%. La correlación es de -0,41 con `largo_texto` y de -0,26 con `cant_hashtags`.
- **Decisión:** ✅ Aprobada
- **Justificación:** la proporción de negativos sube con cada mención al inicio, así que la feature puede ayudar a reconocerlos. Contar solo las del inicio evita mezclarlas con las del resto del texto, que se comportan al revés (J-17 y J-20). Contarlas conserva la diferencia entre empezar con 1 o con 2 o más (J-18), y el tope evita que los pocos tweets con muchas menciones pesen demasiado (J-19). Las correlaciones con las otras features son moderadas, así que no repiten su información. El dataset no indica qué tweets son respuestas, por lo que es una aproximación. Su aporte real se evalúa en el modelado.

### J-17 · Cantidad total de menciones
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #6 / #33
- **Propuesta:** contar todas las menciones del tweet, sin distinguir dónde están.
- **Cómo se probó:** proporción de clases según la cantidad total de menciones (0, 1, 2, 3-4 y 5 o más).
- **Evidencia:** la proporción de negativos sube de 10,7% sin menciones a 20,9% con 3-4, pero con 5 o más baja a 11,8%; la de positivos baja de 21,2% a 10,5% y después sube a 21,6%.
- **Decisión:** ❌ Descartada
- **Justificación:** mezcla las menciones del inicio, que tienden a ser negativas, con las del resto del texto, que tienden a ser neutrales, y los dos efectos se tapan entre sí.

### J-18 · Feature binaria: si el tweet es una respuesta
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #6 / #33
- **Propuesta:** en lugar de contar las menciones del inicio, marcar solo si el tweet empieza con alguna (`es_respuesta`).
- **Cómo se probó:** proporción de clases en los tweets que empiezan y no empiezan con una mención, comparada con J-16.
- **Evidencia:** los negativos son el 20,6% de los tweets que empiezan con una mención y el 10,3% de los que no. Pero dentro de los primeros, son el 18,9% con 1 mención al inicio (889 tweets) y el 23,3% con 2 o más (540 tweets).
- **Decisión:** ❌ Descartada
- **Justificación:** es la misma información que `menciones_inicio` con los grupos 1 y "2 o más" juntos. Capta el salto más grande, pero pierde una diferencia que se apoya en grupos grandes, y contar las menciones no es más difícil de calcular ni de explicar.

### J-19 · Cantidad de menciones al inicio sin tope
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #6 / #33
- **Propuesta:** usar la cantidad de menciones al inicio tal cual, sin tope.
- **Cómo se probó:** resumen de la variable y proporción de clases para cada cantidad de menciones al inicio.
- **Evidencia:** el percentil 99 es 5 y el máximo es 15. Los negativos suben hasta las 3 menciones (26,6%), pero desde 4 los grupos tienen entre 11 y 40 tweets y los porcentajes varían sin un patrón (20,0%, 9,1% y 14,8%).
- **Decisión:** ❌ Descartada
- **Justificación:** más allá de las 2 o 3 menciones no hay un patrón claro, y sin tope un modelo lineal (regresión logística o SVM) le daría a un tweet con 15 menciones un peso mucho mayor que a uno con 1 o 2. Los árboles sí podrían usarla sin problema, así que se puede comparar con J-16 en el modelado.

### J-20 · Cantidad de menciones en el resto del texto como segunda feature
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #6 / #33
- **Propuesta:** sumar a `menciones_inicio` una segunda feature, `cant_menciones_texto`, con la cantidad de menciones que no están al inicio del tweet.
- **Cómo se probó:** proporción de clases según las menciones en el resto del texto (0, 1 y 2 o más) y correlación con `menciones_inicio` sobre `log1p`.
- **Evidencia:** solo el 8,5% de los tweets (271) tiene menciones en el resto del texto, y el máximo es 9. A medida que aumentan, sube la proporción de neutrales (66,8%, 68,5% y 79,2%) y baja la de negativos (de 15,5% a 2,8%). La correlación con `menciones_inicio` es de -0,17.
- **Decisión:** ❌ Descartada
- **Justificación:** muestra una señal propia, distinta de la de las menciones al inicio, pero afecta a pocos tweets, así que por ahora se prioriza una sola feature más simple. Se puede volver a sumar si en el modelado hace falta.

### J-21 · Si el tweet tiene signos de exclamación
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #31 / #33
- **Propuesta:** crear la feature `tiene_exclamacion`, que vale 1 si el tweet tiene al menos un `!`, contado sin las URLs.
- **Cómo se probó:** proporción de clases en los tweets con y sin `!`, cantidad de tweets con `¡` pero sin `!`, y correlación con `largo_texto`, `cant_hashtags` y `menciones_inicio` sobre `log1p`.
- **Evidencia:** el 7,5% de los tweets tiene `!`. Entre ellos, el 46,9% es positivo, el 41,0% neutral y el 12,1% negativo, contra 15,4%, 69,5% y 15,1% en los que no tienen. Solo 2 tweets tienen `¡` sin `!`. La correlación con las otras features está entre -0,01 y 0,09.
- **Decisión:** ✅ Aprobada
- **Justificación:** es la diferencia entre clases más grande de las features analizadas: los tweets con `!` son positivos el triple de veces que el resto. No se correlaciona con las otras features, así que aporta información nueva. El `¡` no se cuenta porque casi siempre va con un `!`. Su aporte real se evalúa en el modelado.

### J-22 · Cantidad de signos de exclamación
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #31 / #33
- **Propuesta:** contar los `!` de cada tweet, con tope en 3, en lugar de marcar solo si hay alguno.
- **Cómo se probó:** resumen de la variable y proporción de clases según la cantidad de `!` (0, 1, 2 y 3 o más).
- **Evidencia:** el percentil 99 es 2 y el máximo, 9. Los positivos son el 50,0% de los tweets con un `!`, el 41,2% de los que tienen 2 y el 35,5% de los que tienen 3 o más, donde los negativos suben al 35,5%. Los grupos con 2 y con 3 o más tienen 34 y 31 tweets.
- **Decisión:** ❌ Descartada
- **Justificación:** la relación no va en una sola dirección: un `!` suele acompañar a un tweet entusiasta, pero muchos también aparecen en críticas o quejas. Con grupos tan chicos, contar no agrega información confiable respecto de marcar si hay algún `!`.

### J-23 · Signos de exclamación repetidos como segunda feature
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #31 / #33
- **Propuesta:** sumar a `tiene_exclamacion` una segunda feature que marque si el tweet tiene `!` repetidos (`!!`).
- **Cómo se probó:** proporción de clases en los tweets con y sin `!!`.
- **Evidencia:** 41 tweets tienen `!!`. Entre ellos, el 34,1% es negativo, contra el 14,6% del resto.
- **Decisión:** ❌ Descartada
- **Justificación:** muestra una señal hacia los negativos, pero afecta a muy pocos tweets, el mismo criterio que en J-20. Se puede volver a sumar si en el modelado hace falta.

### J-24 · Cantidad de signos de pregunta, con tope en 2
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #32 / #33
- **Propuesta:** crear la feature `cant_preguntas`, la cantidad de `?` del tweet contados sin las URLs, con tope en 2.
- **Cómo se probó:** proporción de clases según la cantidad de `?` (0, 1 y 2 o más), cantidad de tweets con `¿` pero sin `?`, y correlación con `largo_texto`, `cant_hashtags`, `menciones_inicio` y `tiene_exclamacion` sobre `log1p`.
- **Evidencia:** el 12,6% de los tweets tiene `?`. Los positivos son el 19,0% de los tweets sin preguntas, el 11,5% de los que tienen una y el 2,3% de los que tienen 2 o más (88 tweets). Los neutrales son el 65,8%, el 78,7% y el 76,1%. Ningún tweet tiene `¿` sin `?`. La correlación con las otras features está entre 0,00 y 0,06.
- **Decisión:** ✅ Aprobada
- **Justificación:** la proporción de positivos baja con cada `?`, así que la relación va en una sola dirección y se usa la cantidad, el mismo criterio que con las menciones al inicio (J-16). El tope evita que los pocos tweets con muchas preguntas pesen demasiado (J-26). No se correlaciona con las otras features, así que aporta información nueva. Su aporte real se evalúa en el modelado.

### J-25 · Feature binaria: si el tweet tiene signos de pregunta
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #32 / #33
- **Propuesta:** marcar solo si el tweet tiene algún `?`, en lugar de contarlos.
- **Cómo se probó:** proporción de clases en los tweets con y sin `?`, comparada con J-24.
- **Evidencia:** los positivos son el 9,5% de los tweets con `?` y el 19,0% de los que no tienen. Dentro de los primeros, son el 11,5% con una pregunta y el 2,3% con 2 o más.
- **Decisión:** ❌ Descartada
- **Justificación:** junta a los tweets con una pregunta y con varias, y pierde que con 2 o más casi no hay positivos.

### J-26 · Cantidad de signos de pregunta sin tope
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #32 / #33
- **Propuesta:** usar la cantidad de `?` tal cual, sin tope.
- **Cómo se probó:** resumen de la variable y proporción de clases para cada cantidad de `?`.
- **Evidencia:** el percentil 99 es 2 y el máximo, 7. Los tweets con 3 `?` son 17 y los que tienen 4 o más, 12.
- **Decisión:** ❌ Descartada
- **Justificación:** desde 3 preguntas los grupos son demasiado chicos para sacar conclusiones, y sin tope los valores altos pesarían más que el resto en un modelo lineal.

### J-27 · Signos de pregunta repetidos como segunda feature
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #32 / #33
- **Propuesta:** sumar a `cant_preguntas` una segunda feature que marque si el tweet tiene `?` repetidos (`??`).
- **Cómo se probó:** proporción de clases en los tweets con y sin `??`.
- **Evidencia:** 18 tweets tienen `??`. Ninguno es positivo y el 27,8% es negativo.
- **Decisión:** ❌ Descartada
- **Justificación:** afecta a muy pocos tweets, el mismo criterio que en J-23.

### J-31 · Cantidad de emojis, con tope en 2
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #35 / #33
- **Propuesta:** crear la feature `cant_emojis`, la cantidad de emojis del tweet contados sin las URLs, con tope en 2.
- **Cómo se probó:** porcentaje de tweets con al menos un emoji y media por clase, proporción de clases según la cantidad de emojis (0, 1 y 2 o más) y correlación con el resto de las features sobre `log1p`. Se usó la librería `emoji` para no separar emojis compuestos.
- **Evidencia:** el 24,3% de los tweets tiene emojis (773). El máximo es 17. Tienen algún emoji el 39,4% de los positivos, el 22,0% de los neutrales y el 16,7% de los negativos. La proporción de positivos sube de 14,2% (0 emojis) a 26,4% (1 emoji) y a 32,2% (2 o más). La de negativos baja de 16,4% a 10,9% y a 9,3%. La correlación más alta es de 0,22 (con `cant_hashtags`).
- **Decisión:** ✅ Aprobada
- **Justificación:** la proporción de positivos sube con cada emoji, y la de negativos baja. Como la relación va en una sola dirección, se usa la cantidad y no solo si hay alguno (J-32). El tope evita que los pocos tweets con muchos emojis pesen demasiado (J-32). Tiene correlaciones bajas, así que aporta información nueva.

### J-32 · Alternativas descartadas para los emojis (sin tope, presencia y filtro temático)
- **Etapa:** 3. Ingeniería de features
- **Issue / PR:** #35 / #33
- **Propuesta:** evaluar tres formas alternativas de medir los emojis: usar la cantidad sin tope, indicar solo si el tweet tiene o no emojis, y contar la cantidad omitiendo el emoji del tema (🤖).
- **Cómo se probó:** proporción de clases para los distintos agrupamientos de cada alternativa, comparadas con la feature aprobada (J-31).
- **Evidencia:** con "tiene o no", la proporción de positivos es de 28,8% cuando hay emojis, juntando a los grupos de 1 y de 2 o más. "Sin tope" llega hasta 17 emojis, pero desde 2 los positivos dejan de subir (32,2%, 32,3% y 32,1%) en grupos chicos (96 y 78 tweets desde 3). Al descartar el emoji 🤖, los tweets con emojis bajan de 773 a 763, y la proporción de positivos sube de 14,2% a 27,4% (1 emoji) y a 31,6% (2 o más).
- **Decisión:** ❌ Descartada
- **Justificación:** "tiene o no" pierde que la proporción de positivos sigue subiendo. "Sin tope" deja que valores extremos pesen demasiado en modelos lineales sin agregar un patrón claro. "Sin el emoji del tema" tiene una mejora casi nula frente a J-31, sumando una regla basada en una lista manual de emojis descartables.

## 4. Preprocesamiento de texto

### J-35 · Reemplazar cada URL por un token (camino C)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #8 / #36
- **Propuesta:** reemplazar cada URL del texto por el token `URLTOKEN`, en una columna nueva `texto_procesado`, para descartar el contenido del link y dejar explícita su presencia.
- **Cómo se probó:** análisis del porcentaje de tweets con URL por clase, del dominio, la posición y la cantidad de URLs por tweet, y comparación de los cinco caminos (J-35 a J-39) según si conservan la presencia del link y si descartan el código aleatorio.
- **Evidencia:** tiene URL el 36,24% de los neutrales, el 31,10% de los positivos y el 20,68% de los negativos, con al menos 474 tweets por clase. Las 1198 URLs son de `t.co`, con un código aleatorio.
- **Decisión:** ✅ Aprobada
- **Justificación:** lo que puede aportar es la presencia del link, no su contenido. C conserva la presencia y descarta los códigos aleatorios de `t.co`, igual que D, pero se resuelve dentro del texto sin agregar columnas.

### J-36 · Mantener las URLs tal cual (camino A)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #8 / #36
- **Propuesta:** no tratar las URLs y dejarlas en el texto.
- **Cómo se probó:** análisis del porcentaje de tweets con URL por clase, del dominio, la posición y la cantidad de URLs por tweet, y revisión de las palabras que TF-IDF obtiene de una URL.
- **Evidencia:** TF-IDF parte `https://t.co/Hv4UBFRjXb` en `https`, `co` y `hv4ubfrjxb`.
- **Decisión:** ❌ Descartada
- **Justificación:** `https` y `co` ya marcan que el tweet tiene link, pero además deja en el texto los códigos aleatorios de `t.co`, que no aportan información.

### J-37 · Eliminar las URLs (camino B)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #8 / #36
- **Propuesta:** borrar las URLs del texto sin dejar rastro.
- **Cómo se probó:** análisis del porcentaje de tweets con URL por clase, del dominio, la posición y la cantidad de URLs por tweet.
- **Evidencia:** la proporción de tweets con URL cambia entre clases (36,24%, 31,10% y 20,68%) y la diferencia es de casi 16 puntos.
- **Decisión:** ❌ Descartada
- **Justificación:** pierde la información de que el tweet tenía link, que se relaciona con la clase.

### J-38 · Eliminar las URLs y agregar la feature `tiene_url` (camino D)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #8 / #36
- **Propuesta:** borrar las URLs del texto y guardar su presencia en una columna binaria `tiene_url`.
- **Cómo se probó:** análisis del porcentaje de tweets con URL por clase, del dominio, la posición y la cantidad de URLs por tweet.
- **Evidencia:** la misma que J-35.
- **Decisión:** ❌ Descartada, como alternativa equivalente
- **Justificación:** guarda la misma información que C pero en una columna aparte. Queda como alternativa si en el modelado se combinan el texto con features numéricas.

### J-39 · Eliminar las URLs y agregar la feature `cant_urls` (camino E)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #8 / #36
- **Propuesta:** borrar las URLs del texto y guardar la cantidad de URLs en una columna `cant_urls`.
- **Cómo se probó:** análisis del porcentaje de tweets con URL por clase, del dominio, la posición y la cantidad de URLs por tweet.
- **Evidencia:** 921 tweets tienen una URL y solo 130 tienen dos o más.
- **Decisión:** ❌ Descartada
- **Justificación:** como casi todos los tweets con link tienen uno solo, el conteo agrega poco frente a la presencia.

### J-40 · Sacar el `#`, separar y usar la feature `cant_hashtags` (camino E)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #9 / #36
- **Propuesta:** en `texto_procesado`, reemplazar cada hashtag por sus palabras. Se saca el `#`, se separan las palabras unidas con mayúsculas antes de pasar a minúsculas y se borran los `#` sueltos. La cantidad de hashtags ya está en `cant_hashtags` (J-13).
- **Cómo se probó:** análisis del porcentaje de tweets con hashtag y de la cantidad promedio por clase, de los hashtags más usados, de su posición en el texto y de las variantes de "inteligencia artificial" por clase. Después, comparación de los cinco caminos (J-40 a J-44) según si conservan la palabra y la presencia y cantidad. El impacto en el desempeño no se midió todavía: se va a validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** tiene hashtag el 25,62% de los positivos, el 14,69% de los neutrales y el 7,17% de los negativos. Entre los tweets con hashtag, el promedio es 2,00, 2,27 y 1,56, y 238 de los 494 tienen dos o más. El 63,56% de los tweets con hashtag tiene alguno dentro de la frase. Se considera que un hashtag está "al final" cuando después solo hay otros hashtags, URLs, espacios o signos de puntuación. El 23,46% de los hashtags junta varias palabras con mayúsculas. "Inteligencia artificial" aparece en 206 hashtags escrita de seis formas, y en todas predomina la clase neutral.
- **Decisión:** ✅ Aprobada
- **Justificación:** conserva la palabra, que muchas veces forma parte de la oración, y guarda la presencia y la cantidad, que cambian entre clases. Separar hace que el hashtag termine en las mismas palabras que el texto. Las variantes de "inteligencia artificial" (`ai`, `inteligencia artificial`, `artificial intelligence`, con o sin espacio) no se unifican en `ia`, por dos motivos: la unificación se haría solo en los hashtags y no en el cuerpo del tweet, lo que contradice el argumento de relacionar el hashtag con las palabras del texto, y el concepto es el tema de todo el dataset, por lo que discrimina poco entre clases. `cant_hashtags` también conserva la presencia (`cant_hashtags > 0`) y por eso se prefirió a una columna binaria (J-14).

### J-41 · Mantener los hashtags tal cual (camino A)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #9 / #36
- **Propuesta:** no tratar los hashtags y dejarlos en el texto.
- **Cómo se probó:** mismo análisis que J-40 y revisión de las palabras que TF-IDF obtiene de un tweet con hashtags. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** TF-IDF convierte `#IA` en `ia` y `#Tecnología` en `tecnología`.
- **Decisión:** ❌ Descartada
- **Justificación:** como TF-IDF no toma el `#` como parte de la palabra, para el modelo este camino es igual a C (J-43). Se descarta por lo mismo: no separa los hashtags compuestos y pierde la presencia y la cantidad de hashtags.

### J-42 · Eliminar los hashtags (camino B)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #9 / #36
- **Propuesta:** borrar los hashtags del texto sin dejar rastro.
- **Cómo se probó:** mismo análisis que J-40. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** el 63,56% de los tweets con hashtag tiene alguno dentro de la frase, y la proporción de tweets con hashtag varía más de 18 puntos entre clases.
- **Decisión:** ❌ Descartada
- **Justificación:** deja oraciones incompletas y pierde la presencia del hashtag, que se relaciona con la clase.

### J-43 · Sacar solo el `#` (camino C)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #9 / #36
- **Propuesta:** borrar el `#` y dejar la palabra tal cual, sin columna extra.
- **Cómo se probó:** mismo análisis que J-40. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** TF-IDF obtiene las mismas palabras que en A: `#IA` y `IA` terminan como `ia`.
- **Decisión:** ❌ Descartada, equivalente a A con TF-IDF
- **Justificación:** sacar el `#` a mano solo cambia el texto guardado, no las palabras que ve el modelo, así que tiene los mismos problemas que A (J-41): no separa los hashtags compuestos y pierde la presencia y la cantidad de hashtags, que cambian entre clases.

### J-44 · Reemplazar cada hashtag por un token (camino D)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #9 / #36
- **Propuesta:** reemplazar cada hashtag por el token `HASHTAGTOKEN`, como se hizo con las URLs.
- **Cómo se probó:** mismo análisis que J-40. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** el 63,56% de los tweets con hashtag tiene alguno dentro de la frase.
- **Decisión:** ❌ Descartada
- **Justificación:** a diferencia del link de una URL, la palabra del hashtag aporta contenido, y el token la pierde. Se podría dejar la palabra junto al token, pero preferimos guardar la señal en una columna, porque queda separada del texto y es más fácil de interpretar.

### J-45 · Reemplazar cada mención por un token (camino C)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #10 / #36
- **Propuesta:** reemplazar cada mención del texto por el token `USERTOKEN` en `texto_procesado`, para descartar el nombre del usuario y dejar explícita la presencia de la mención.
- **Cómo se probó:** análisis del porcentaje de tweets con mención por clase y posición, de los usuarios más mencionados, del tamaño del vocabulario y de los tweets vacíos, y comparación de los cinco caminos (J-45 a J-49). Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** el 52,26% de los tweets tiene alguna mención. Tiene mención al inicio el 62,03% de los negativos, el 43,66% de los neutrales y el 35,16% de los positivos. Hay 1342 usuarios distintos y el 56,48% aparece una sola vez. El vocabulario pasa de 14307 palabras con menciones a 12985 con el token, una más que al eliminarlas, y ningún tweet queda vacío.
- **Decisión:** ✅ Aprobada
- **Justificación:** lo que puede aportar es la presencia de la mención, no el nombre del usuario. Marca también las menciones del resto del texto, que `menciones_inicio` no cuenta, casi sin costo en el vocabulario. Sigue la misma lógica que `URLTOKEN`. Repite en parte lo que dice `menciones_inicio` y pierde el nombre de la cuenta, como `OpenAI`.

### J-46 · Mantener las menciones tal cual (camino A)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #10 / #36
- **Propuesta:** no tratar las menciones y dejarlas en el texto.
- **Cómo se probó:** mismo análisis que J-45 y revisión de las palabras que TF-IDF obtiene de un tweet con menciones. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** TF-IDF convierte `@OpenAI` en `openai`. Los nombres de usuario suman 1323 palabras al vocabulario, un 9,25% del total. `@lareimers` aparece 13 de 20 veces en negativos.
- **Decisión:** ❌ Descartada
- **Justificación:** la mayoría de los usuarios aparece muy pocas veces, y el modelo podría memorizar cuentas en lugar de aprender cómo se expresa el sentimiento.

### J-47 · Eliminar las menciones (camino B)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #10 / #36
- **Propuesta:** borrar las menciones del texto sin dejar rastro.
- **Cómo se probó:** mismo análisis que J-45. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** 9 tweets quedan vacíos y 271 tweets tienen menciones fuera del inicio, que ninguna columna cuenta. El vocabulario queda en 12984 palabras.
- **Decisión:** ❌ Descartada
- **Justificación:** saca los nombres de usuario y `menciones_inicio` conserva la señal del inicio, pero no deja rastro de las demás menciones y ahorra una sola palabra frente a C. Queda como alternativa para comparar en el modelado.

### J-48 · Eliminar las menciones y agregar la feature `tiene_mencion` (camino D)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #10 / #36
- **Propuesta:** borrar las menciones del texto y guardar su presencia en una columna binaria `tiene_mencion`.
- **Cómo se probó:** mismo análisis que J-45. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** de los 1664 tweets con mención, 1429 tienen alguna al inicio, una señal que ya guarda `menciones_inicio`.
- **Decisión:** ❌ Descartada
- **Justificación:** guarda casi la misma información que `menciones_inicio`, y una feature binaria parecida ya se descartó (J-18).

### J-49 · Eliminar solo las menciones del inicio (camino E)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #10 / #36
- **Propuesta:** borrar las menciones del inicio del tweet y dejar tal cual las del resto del texto.
- **Cómo se probó:** mismo análisis que J-45. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** el vocabulario queda en 13327 palabras, 343 más que al eliminarlas todas, y 9 tweets quedan vacíos.
- **Decisión:** ❌ Descartada
- **Justificación:** conserva nombres que forman parte de la frase con poco costo, pero necesita dos reglas según la posición y mantiene nombres de usuario poco repetidos. Queda como alternativa para comparar en el modelado.

### J-50 · Pasar cada emoji a palabras (camino D)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #11 / #36
- **Propuesta:** reemplazar cada emoji por su nombre en español en `texto_procesado`, con las palabras separadas.
- **Cómo se probó:** análisis del porcentaje de tweets con emoji por clase, de los emojis más usados y su clase, del tamaño del vocabulario y de los tweets sin palabras propias, y comparación de los cinco caminos (J-50 a J-54). Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** el 24,28% de los tweets tiene emojis. 👏 aparece 27 de 32 veces en positivos, 🚀 20 de 27, 🤔 27 de 29 en neutrales y 🤡 13 de 17 en negativos. Hay 373 emojis distintos y el 51,21% aparece una sola vez. El vocabulario pasa de 14307 a 14526 palabras.
- **Decisión:** ✅ Aprobada
- **Justificación:** conserva qué emoji se usó, que `cant_emojis` no distingue y se relaciona con la clase. Los emojis parecidos comparten palabras como `cara` o `corazón`, lo que ayuda con los que aparecen poco. A cambio, algunas palabras pueden confundir (😂 pasa a `cara llorando de risa`) y algunos símbolos suman ruido (`5⃣` pasa a `tecla 5`).

### J-51 · Mantener los emojis tal cual (camino A)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #11 / #36
- **Propuesta:** no tratar los emojis y dejarlos en el texto.
- **Cómo se probó:** mismo análisis que J-50 y revisión de las palabras que TF-IDF obtiene de un tweet con emojis. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** de `Me encanta la IA 😍`, TF-IDF solo obtiene `encanta`, `ia`, `la` y `me`. El vocabulario es de 14307 palabras con emojis y sin ellos.
- **Decisión:** ❌ Descartada
- **Justificación:** TF-IDF no ve los emojis, así que para el modelo es lo mismo que eliminarlos (J-52).

### J-52 · Eliminar los emojis (camino B)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #11 / #36
- **Propuesta:** borrar los emojis del texto sin dejar rastro.
- **Cómo se probó:** mismo análisis que J-50. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** 32 tweets se quedan sin palabras propias: pasan de 41 a 73, sin contar URLs ni menciones.
- **Decisión:** ❌ Descartada
- **Justificación:** la cantidad queda en `cant_emojis`, pero se pierde qué emoji se usó, que se relaciona con la clase.

### J-53 · Reemplazar cada emoji por un token (camino C)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #11 / #36
- **Propuesta:** reemplazar cada emoji por el token `EMOJITOKEN`, como se hizo con las URLs y las menciones.
- **Cómo se probó:** mismo análisis que J-50. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** el vocabulario suma una sola palabra (14308), pero todos los emojis quedan iguales.
- **Decisión:** ❌ Descartada
- **Justificación:** solo guarda la presencia, que ya está en `cant_emojis`, y pierde qué emoji se usó.

### J-54 · Un token por emoji (camino E)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #11 / #36
- **Propuesta:** reemplazar cada emoji por su nombre en español como una sola palabra, por ejemplo `cara_sonriendo_con_ojos_de_corazón`.
- **Cómo se probó:** mismo análisis que J-50. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** el vocabulario pasa a 14641 palabras, 115 más que D. El 51,21% de los emojis aparece una sola vez.
- **Decisión:** ❌ Descartada
- **Justificación:** evita mezclar el emoji con palabras comunes, pero muchos tokens se repetirían muy poco y los emojis parecidos no comparten palabras. Queda como alternativa para comparar en el modelado.

### J-56 · Tokenizar por palabras, en minúsculas y sin puntuación, conservando las tildes
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #37 / #XX
- **Propuesta:** separar `texto_procesado` en palabras con la expresión regular `\w+` después de pasarlo a minúsculas, y guardar el resultado en la columna `tokens`. Quedan las letras (con tilde y ñ) y los números, y la puntuación queda afuera.
- **Cómo se probó:** cantidad de signos de puntuación en train y cuántos son `!`, `?`, `¡` o `¿`, que ya están en las features; cantidad de tweets con emoticonos de texto (`:)`, `xD`, `<3`) y sus clases; y cantidad de veces que aparece cada versión de palabras que solo se distinguen por la tilde.
- **Evidencia:** hay 12141 signos en train y los más usados son el punto (4393) y la coma (3373). Solo el 11,03% son `!`, `?`, `¡` o `¿`. 19 tweets (0,60%) tienen emoticonos de texto, sin una clase dominante (14 neutrales, 3 positivos y 2 negativos). `sí` aparece 62 veces y `si` 350; `más` 432 veces y `mas` 36.
- **Decisión:** ✅ Aprobada
- **Justificación:** la puntuación casi no aporta: la señal de `!` y `?` ya está en `tiene_exclamacion` y `cant_preguntas` (J-21 y J-24), así que se puede sacar del texto sin perderla. La teoría recomienda no sacar la puntuación en general, pero acá se saca porque lo que informa ya está guardado en columnas. Perder los emoticonos de texto es un costo chico, porque casi no aparecen. Las tildes se conservan porque distinguen palabras (J-58).

### J-57 · Conservar la puntuación en el texto
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #37 / #XX
- **Propuesta:** no sacar los signos de puntuación y dejarlos en el texto, como recomienda la teoría en general.
- **Cómo se probó:** mismo análisis de la puntuación que en J-56.
- **Evidencia:** el 88,97% de los 12141 signos no son `!`, `?`, `¡` ni `¿`, y los más usados son el punto (4393) y la coma (3373).
- **Decisión:** ❌ Descartada
- **Justificación:** lo que informa de la puntuación (`!` y `?`) ya está en las features, y el resto son sobre todo puntos y comas. A cambio se resigna lo que esos signos pudieran aportar, que no se midió por clase.

### J-58 · Sacar las tildes
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #37 / #XX
- **Propuesta:** pasar las palabras a su forma sin tilde, para juntar las variantes que algunos tweets escriben sin ella.
- **Cómo se probó:** cantidad de veces que aparece cada versión de cuatro pares que solo se distinguen por la tilde: `sí` y `si`, `más` y `mas`, `él` y `el`, `tú` y `tu`.
- **Evidencia:** `sí` aparece 62 veces y `si` 350; `más` 432 veces y `mas` 36.
- **Decisión:** ❌ Descartada
- **Justificación:** sin tildes se juntarían palabras distintas (`sí` y `si`, `él` y `el`, `tú` y `tu`). Los tweets que escriben sin tilde son pocos (`mas` aparece 36 veces contra 432 de `más`), así que lo que se ganaría al juntar variantes es poco.

### J-59 · Tokenizar por caracteres
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #37 / #XX
- **Propuesta:** usar caracteres (o secuencias de caracteres) como unidad en lugar de palabras.
- **Cómo se probó:** no se probó en el notebook; se descarta por razonamiento.
- **Evidencia:** sin evidencia propia por ahora.
- **Decisión:** ❌ Descartada, como opción principal
- **Justificación:** el vocabulario sería chico, pero las secuencias son larguísimas y se pierde el significado de las palabras. Queda como alternativa para comparar en el modelado.

### J-60 · Quitar las stopwords de NLTK conservando ocho palabras
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #37 / #XX
- **Propuesta:** quitar las stopwords de la lista de NLTK para español (313 palabras), salvo `no`, `ni`, `nada`, `sin`, `pero`, `muy`, `más` y `sí`: negaciones, contraste, intensidad y afirmación.
- **Cómo se probó:** porcentaje de tokens de train que son stopwords y, para cada stopword que aparece en al menos 30 tweets, proporción de cada clase entre los tweets que la usan, comparada con la de todo train. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** el 42,31% de los tokens de train son stopwords. Palabras como `de`, `la` o `el` tienen una proporción de clases casi igual a la de train. Entre las de más desvío están `ni` (37,1% de negativos, contra 14,9% en train), `nada` (35,3%) y `sin` (29,8%); pronombres como `tu`, `tus` o `nuestro` se inclinan a los positivos.
- **Decisión:** ✅ Aprobada
- **Justificación:** las stopwords son casi la mitad del texto y la mayoría no distingue clases, así que quitarlas lo achica sin perder señal. Las que cambian el sentido de la frase o se asocian a una clase se conservan: en sentimiento "me gusta" y "no me gusta" son opuestos. La lista de palabras conservadas se armó a mano a partir de esa tabla y de ese criterio.

### J-61 · Quitar la lista completa de stopwords de NLTK
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #37 / #XX
- **Propuesta:** quitar las 313 stopwords de la lista de NLTK sin conservar ninguna.
- **Cómo se probó:** mismo análisis que J-60. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** la lista incluye `no`, `ni`, `nada`, `sin`, `pero`, `muy`, `más` y `sí`. Entre los tweets que usan `ni`, el 37,1% es negativo, contra 14,9% en train; con `nada` es el 35,3% y con `sin`, el 29,8%.
- **Decisión:** ❌ Descartada
- **Justificación:** borra la negación y otras palabras que cambian el sentido: "me gusta" y "no me gusta" quedarían iguales.

### J-62 · Texto con stemming (Snowball)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #37 / #XX
- **Propuesta:** reducir cada palabra a su raíz con `SnowballStemmer('spanish')`, a partir del texto sin stopwords, en la columna `texto_stem`. Los marcadores `urltoken` y `usertoken` no se modifican. Sirve para juntar las variantes de una misma palabra.
- **Cómo se probó:** cantidad de palabras distintas del texto sin reducir y del texto con stemming, sin filtro y con `min_df=2` (con `CountVectorizer`), y cómo queda un mismo tweet en cada versión. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** hay 11778 palabras distintas sin reducir y 6850 con stemming (un 41,8% menos). Con `min_df=2` quedan 4858 sin reducir y 3392 con stemming. En un tweet de ejemplo, `puede` queda como `pued`.
- **Decisión:** ✅ Aprobada, como candidata (se compara con J-63 y J-64 en el modelado)
- **Justificación:** es la que más reduce el vocabulario, y con 3184 tweets juntar las variantes hace que cada palabra aparezca más veces. A cambio, la raíz no siempre es una palabra real (`pued`). La decisión final se completa con los resultados del modelado.

### J-63 · Texto con lematización (spaCy)
- **Etapa:** 4. Preprocesamiento de texto
- **Issue / PR:** #37 / #XX
- **Propuesta:** pasar cada palabra a su forma de diccionario con el modelo `es_core_news_sm` de spaCy, a partir del texto sin stopwords, en la columna `texto_lemma`. Se desactivan el parser y el reconocimiento de entidades, se evita que spaCy vuelva a tokenizar, y `urltoken` y `usertoken` no se modifican.
- **Cómo se probó:** mismo análisis que J-62. Queda por validar en el modelado, con validación cruzada (J-55) y la métrica que se defina.
- **Evidencia:** la lematización baja las palabras distintas de 11778 a 8867 (un 24,7% menos). Con `min_df=2` quedan 3823. En un tweet de ejemplo, `puede` queda como `poder`.
- **Decisión:** ✅ Aprobada, como candidata (se compara con J-62 y J-64 en el modelado)
- **Justificación:** reduce menos el vocabulario que el stemming, pero deja palabras reales (`poder`), más fáciles de leer e interpretar. Una limitación: se aplica sobre el texto sin stopwords (J-60), así que spaCy analiza oraciones incompletas y puede equivocarse más que con la oración entera. La decisión final se completa con los resultados del modelado.

## 5. Modelado

## 6. Evaluación
