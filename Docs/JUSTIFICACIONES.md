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
- **Issue / PR:** #1 / PR pendiente
- **Propuesta:** graficar las métricas de interacción con un histograma común y, para mejorar la visualización, normalizarlas con min-max.
- **Cómo se probó:** histograma de cada variable sin transformar y de su versión normalizada con (x − min) / (max − min), y porcentaje de datos que cae en el primer bin.
- **Evidencia:** sin transformar, el 99,55% de `favorite_count` y el 99,11% de `retweet_count` cae en el primer bin. Con min-max los porcentajes son los mismos y el histograma tiene la misma forma, solo cambia el eje X a [0, 1].
- **Decisión:** ❌ Descartada
- **Justificación:** el histograma común muestra una sola barra y la cola no se ve. Min-max cambia la escala pero no la forma de la distribución, y depende del máximo, que es un valor extremo (2243 y 502), así que no ayuda a visualizarla.

### J-02 · Transformación `log1p` con todos los datos (opción A)
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #1 / PR pendiente
- **Propuesta:** graficar `log1p(x)` = log(1 + x) de todos los tweets, para comprimir la cola. Se usa `log1p` y no `log` porque la mayoría de los valores son 0 y log(0) no existe.
- **Cómo se probó:** histograma de `log1p` de cada variable, junto con el porcentaje de ceros y el resumen estadístico.
- **Evidencia:** el 53,15% de los tweets tiene 0 favoritos y el 84,74% tiene 0 retweets. Media vs. mediana: 3,82 vs. 0 en favoritos y 0,87 vs. 0 en retweets. El máximo es 2243 en favoritos y 502 en retweets.
- **Decisión:** ✅ Aprobada, como complemento de J-03
- **Justificación:** muestra todo el dataset y la proporción real de tweets sin interacción. Pero la barra del 0 domina el gráfico y el cuerpo de los tweets con interacción se distingue poco, por eso no alcanza sola.

### J-03 · Filtrar los tweets con interacción y usar eje logarítmico (opción B)
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #1 / PR pendiente
- **Propuesta:** graficar solo los tweets con valor mayor a 0, con el eje X en escala logarítmica, para ver la forma de la distribución sin que los ceros la tapen.
- **Cómo se probó:** histograma con `log_scale=True` de los valores mayores a 0 de cada variable, y resumen de esos valores.
- **Evidencia:** hay 1892 tweets con favoritos (46,85% del total) y 616 con retweets (15,26%). En ambos casos la mediana es 2. El percentil 90 es 10 en favoritos y 8,5 en retweets.
- **Decisión:** ✅ Aprobada, como complemento de J-02
- **Justificación:** deja ver cómo se reparten los tweets que sí interactúan: la mayoría tiene entre 1 y 2 y la frecuencia cae de forma gradual hasta los extremos. No muestra los ceros, así que hay que aclararlo y acompañarlo con el porcentaje de ceros y la opción A.

## 3. Ingeniería de features

## 4. Preprocesamiento de texto

## 5. Modelado

## 6. Evaluación
