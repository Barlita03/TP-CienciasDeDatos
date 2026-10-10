# Avance del TP

Las etapas 1 a 6 salen de los títulos de los Lineamientos de la consigna, y la 7 a la 9, de los Entregables. Cuándo se tilda una subtarea y cómo sumar una nueva está en [CONTRIBUTING.md](CONTRIBUTING.md).

## 1. Exploración del dataset

- [x] ¿Cuántos tweets hay por clase?
- [x] ¿El dataset está balanceado?
- [x] ¿Cómo puede afectar esto al entrenamiento de modelos?
- [x] ¿Cuántos tweets contiene?
- [x] ¿Qué columnas están disponibles?
- [x] ¿Qué tipos de datos tiene cada variable?
- [x] ¿Existen valores faltantes?

## 2. Análisis de las variables disponibles

- [ ] ¿Qué distribución tienen las métricas de interacción? ([#1](https://github.com/Barlita03/TP-CienciasDeDatos/issues/1))
- [ ] ¿Los tweets positivos reciben más interacción que los negativos? ([#2](https://github.com/Barlita03/TP-CienciasDeDatos/issues/2))
- [ ] ¿Se observan outliers? ([#3](https://github.com/Barlita03/TP-CienciasDeDatos/issues/3))

## 3. Ingeniería de features

- [ ] Longitud del tweet ([#4](https://github.com/Barlita03/TP-CienciasDeDatos/issues/4))
- [ ] Cantidad de hashtags ([#5](https://github.com/Barlita03/TP-CienciasDeDatos/issues/5))
- [ ] Cantidad de usuarios mencionados ([#6](https://github.com/Barlita03/TP-CienciasDeDatos/issues/6))
- [ ] Justificar por qué cada feature podría ser útil ([#7](https://github.com/Barlita03/TP-CienciasDeDatos/issues/7))

## 4. Preprocesamiento de texto

Para cada elemento: decidir si se elimina, se transforma, se construye una feature o se mantiene tal cual.

- [ ] URLs ([#8](https://github.com/Barlita03/TP-CienciasDeDatos/issues/8))
- [ ] Hashtags ([#9](https://github.com/Barlita03/TP-CienciasDeDatos/issues/9))
- [ ] Menciones a otros usuarios (@usuario) ([#10](https://github.com/Barlita03/TP-CienciasDeDatos/issues/10))
- [ ] Emojis ([#11](https://github.com/Barlita03/TP-CienciasDeDatos/issues/11))

## 5. Modelado

- [ ] Gaussian Naive Bayes ([#12](https://github.com/Barlita03/TP-CienciasDeDatos/issues/12))
- [ ] Logistic Regression ([#13](https://github.com/Barlita03/TP-CienciasDeDatos/issues/13))
- [ ] Decision Tree ([#14](https://github.com/Barlita03/TP-CienciasDeDatos/issues/14))
- [ ] Random Forest ([#15](https://github.com/Barlita03/TP-CienciasDeDatos/issues/15))
- [ ] Support Vector Machines ([#16](https://github.com/Barlita03/TP-CienciasDeDatos/issues/16))
- [ ] Investigar y ajustar los hiperparámetros de cada modelo ([#17](https://github.com/Barlita03/TP-CienciasDeDatos/issues/17))
- [ ] Analizar cómo escala SVM con la cantidad de features ([#18](https://github.com/Barlita03/TP-CienciasDeDatos/issues/18))

## 6. Evaluación

- [ ] Definir la métrica a optimizar (clasificación multiclase) ([#19](https://github.com/Barlita03/TP-CienciasDeDatos/issues/19))
- [ ] Analizar los resultados de los modelos ([#20](https://github.com/Barlita03/TP-CienciasDeDatos/issues/20))
- [ ] Comparar el desempeño entre modelos ([#21](https://github.com/Barlita03/TP-CienciasDeDatos/issues/21))

## 7. Conclusiones

- [ ] Redactar las conclusiones en el notebook ([#22](https://github.com/Barlita03/TP-CienciasDeDatos/issues/22))

## 8. Uso de IA

Se arma a partir del campo "Uso de IA" de cada PR.

- [ ] Redactar la sección final del notebook: en qué partes del proceso se usó IA y con qué fin ([#23](https://github.com/Barlita03/TP-CienciasDeDatos/issues/23))

## 9. Presentación

- [ ] Cómo abordamos el problema ([#24](https://github.com/Barlita03/TP-CienciasDeDatos/issues/24))
- [ ] Hallazgos al analizar el dataset ([#25](https://github.com/Barlita03/TP-CienciasDeDatos/issues/25))
- [ ] Decisiones tomadas (features, limpieza, etc.) ([#26](https://github.com/Barlita03/TP-CienciasDeDatos/issues/26))
- [ ] Qué funcionó mejor y qué probamos ([#27](https://github.com/Barlita03/TP-CienciasDeDatos/issues/27))
- [ ] Mejoras a futuro ([#28](https://github.com/Barlita03/TP-CienciasDeDatos/issues/28))
