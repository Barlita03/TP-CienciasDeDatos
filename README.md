# TP - Ciencia de Datos

## Contexto

Una organización dedicada al análisis de tendencias tecnológicas quiere entender cómo percibe el público hispanohablante la inteligencia artificial en redes sociales.

Para eso cuenta con un dataset de tweets en español sobre inteligencia artificial, que incluye el texto de cada tweet, metadatos del usuario y métricas de interacción.

El objetivo del TP es desarrollar un **pipeline completo de Machine Learning** que prediga el sentimiento de un tweet (positivo, negativo o neutral). Lo importante no es solo entrenar un modelo, sino **analizar y justificar las decisiones tomadas en cada etapa** del proceso.

La consigna completa está en [Consigna/Consigna.pdf](Consigna/Consigna.pdf).

## Estructura del repositorio

```
├── TP1_CdD.ipynb          # Notebook entregable
├── README.md              # Este archivo
├── requirements.txt       # Librerías que usa el notebook
├── Consigna/
│   ├── Consigna.pdf       # Enunciado del TP
│   └── ia_tweets.csv      # Dataset
├── Clases/                # Material de las clases (PDFs y links a las grabaciones)
└── Docs/
    ├── AVANCE.md          # Tablero de avance por etapa
    ├── CONTRIBUTING.md    # Cómo colaboramos (ramas, commits, PRs)
    ├── JUSTIFICACIONES.md # Registro de propuestas aprobadas y descartadas, con su evidencia
    └── CHANGELOG.md       # Historial de cambios del proyecto
```

## Avance

El estado de cada etapa y subtarea está en [Docs/AVANCE.md](Docs/AVANCE.md).

## Normas generales

1. **Toda propuesta se prueba y se documenta.** Cualquier idea de análisis, preprocesamiento, feature o modelo se prueba antes de adoptarse. La evidencia y la justificación de por qué se aprobó o se descartó quedan en [Docs/JUSTIFICACIONES.md](Docs/JUSTIFICACIONES.md), y todas las propuestas, incluidas las descartadas, tienen que mencionarse al menos en el notebook.
2. **Nada avanza sin la aprobación de todos.** Cada cambio entra por una Pull Request y se mergea solo cuando la aprobaron todos los integrantes. El detalle del flujo está en [Docs/CONTRIBUTING.md](Docs/CONTRIBUTING.md).
