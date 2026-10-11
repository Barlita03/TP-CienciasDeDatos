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

### J-01 · Mantener los tweets con muchos likes y analizarlos en escala logarítmica
- **Etapa:** 2. Análisis de las variables disponibles
- **Issue / PR:** #1, #2, #3 / pendiente
- **Propuesta:** tratar los tweets con muchos likes como outliers según el criterio IQR, o mantenerlos y analizar `favorite_count` en escala `log(1 + x)`.
- **Cómo se probó:** se aplicó el criterio IQR a `favorite_count` y se comparó la media de likes por clase con y sin el tweet con más likes.
- **Evidencia:** el 53.15% de los tweets tiene 0 likes, la mediana es 0 y el máximo, 2243. Como Q1 = 0 y Q3 = 1, el IQR marca como outliers a 678 tweets (16.79%), todos los que tienen 3 likes o más. Con todos los datos, la media de los positivos (5.02) supera a la de los negativos (2.97); sin el tweet con más likes (2243, positivo) baja a 2.06 y queda por debajo de los negativos (2.97) y de los neutrales (3.66).
- **Decisión:** ✅ Aprobada (mantener los tweets y usar escala logarítmica)
- **Justificación:** con una distribución tan sesgada, el criterio IQR marca como outlier a casi cualquier tweet con interacción, y esos valores son reales, no errores. Eliminarlos borraría la parte más informativa de la variable. La escala logarítmica reduce el peso de los valores extremos sin descartarlos, y la comparación por clase muestra que la media sola puede llevar a una conclusión equivocada.

## 3. Ingeniería de features

## 4. Preprocesamiento de texto

## 5. Modelado

## 6. Evaluación
