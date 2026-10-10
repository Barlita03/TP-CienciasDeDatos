# Cómo colaboramos

## Flujo de trabajo

1. **Tomar una tarea.** Elegir un issue del repo y asignárselo, para que el resto sepa que está tomado.
2. **Crear una rama** desde `main` actualizado, con el número de issue y una descripción corta:
   ```
   git checkout main
   git pull
   git checkout -b 4-longitud-tweet
   ```
3. **Trabajar y commitear.** Los mensajes van en español, son cortos y dicen qué se hizo. Por ejemplo: `Agrega feature de longitud del tweet`.
4. **Abrir la PR** hacia `main`. La descripción tiene que incluir `Closes #<número de issue>`, para que el issue se cierre solo al mergear. La plantilla de PR ya trae ese campo, el de uso de IA y la lista de chequeo.
5. **Revisión y merge.** Revisan los demás integrantes, y el último en aprobar hace el merge.

## Antes de pedir revisión

- [ ] El notebook corre completo con *Run All*.
- [ ] La propuesta está registrada en [JUSTIFICACIONES.md](JUSTIFICACIONES.md), con su evidencia, tanto si se aprobó como si se descartó.
- [ ] Si la PR resuelve una subtarea, está tildada en [AVANCE.md](AVANCE.md).
- [ ] Hay una entrada en [CHANGELOG.md](CHANGELOG.md).
- [ ] Si se agregó una librería, está en [requirements.txt](../requirements.txt).

## Trabajo sobre el notebook

- **Un solo notebook.** Todo se trabaja sobre `TP1_CdD.ipynb`. Las pruebas sueltas se pueden hacer en un notebook propio llamado `prueba_*.ipynb` o dentro de una carpeta `pruebas/` (git los ignora), y al principal pasa solo lo que vale la pena registrar.
- **Las propuestas descartadas también van.** Cada una queda con su código en una subsección "Alternativas probadas" dentro de su etapa.
- **Resultados reproducibles.** Todo lo que tenga aleatoriedad lleva `random_state=42`: la división train/test, los modelos y la validación cruzada.
- **Se sube con las salidas**, así los gráficos y resultados se pueden revisar en la PR sin correr el notebook.

### Cómo evitar conflictos

Para git, el notebook es un JSON enorme, así que dos cambios al mismo tiempo casi siempre terminan en conflicto.

- Hacer PRs chicas y que duren poco.
- No tener dos PRs abiertas que toquen el notebook en la misma etapa.
- Antes de abrir la PR, traer lo último de `main` a la rama.
- Si hay conflicto, no editar el JSON a mano. Quedarse con la versión de `main` y volver a agregar las celdas propias.

## Tablero de avance

Una subtarea de [AVANCE.md](AVANCE.md) se tilda recién cuando la PR que la resuelve está mergeada.

### Subtareas nuevas

Para sumar una subtarea que no estaba en el tablero:

1. Crear su issue con la etiqueta de la etapa que corresponda.
2. Agregarla en [AVANCE.md](AVANCE.md) debajo de esa etapa, con el link al issue.

El cambio en AVANCE puede ir en la misma PR que resuelve la subtarea.
