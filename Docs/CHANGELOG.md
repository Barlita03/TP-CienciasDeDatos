# Changelog

Línea de tiempo de los cambios del proyecto.

## Cómo agregar una entrada

- Cada PR suma una línea con un resumen corto y su número, por ejemplo: `- Agrega feature de longitud del tweet (#35)`.
- Las líneas se agrupan bajo la fecha del merge (`## AAAA-MM-DD`), con las fechas más nuevas arriba.
- El número y la fecha se conocen recién al abrir y al mergear la PR, así que se completan en esos momentos.

## Ejemplo

```markdown
## 2026-10-12
- Agrega feature de longitud del tweet (#35)
- Descarta eliminar emojis del texto (#36)
```

---
## 2026-07-11
- Se tokeniza, se remueve la puntuacion y ciertas stopwords (#37)
- Se aplica Stemming y Lemming (#37)

## AAAA-MM-DD
- Pasa a una división train/test 80/20 con validación cruzada y rehace el análisis sobre el nuevo train (#XX)

## 2026-11-06
- Define el tratamiento de URLs en el preprocesamiento de texto (#36)
- Define el tratamiento de hashtags en el preprocesamiento de texto (#36)
- Define el tratamiento de menciones en el preprocesamiento de texto (#36)
- Define el tratamiento de emojis en el preprocesamiento de texto (#36)
- Rehace el análisis de URLs y hashtags sobre train y aplica el tratamiento a validación y test (#36)

## 2026-11-06
- Divide el dataset en train, validación y test, quita los tweets repetidos antes de la división y rehace el análisis sobre train (#33)

## 2026-11-04
- Agrega feature de cantidad de emojis (#33)

## 2026-11-02
- Agrega las features de la etapa 3: longitud, hashtags, menciones, exclamaciones y preguntas, y quita los tweets con texto repetido (#33)

## 2026-10-18
- Agrega el análisis de las variables disponibles: distribución, interacción por clase y outliers (#30)

## 2026-10-10
- Indica a Claude que ignore la carpeta Otros (#29)
- Arma la documentación y organización inicial del TP
