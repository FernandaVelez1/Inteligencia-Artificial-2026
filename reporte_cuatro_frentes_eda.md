# Reporte EDA — Operación Cuatro Frentes

## Misión 1 — Semáforo Académico

### Pregunta de negocio

¿Qué nivel de riesgo de reprobar presenta cada alumno al cierre del parcial?

### Tipo de problema propuesto

**Clasificar.**

La variable objetivo es `riesgo`, que tiene tres categorías: **verde, amarillo y rojo**. Por lo tanto, no se busca obtener un número, sino determinar a qué categoría pertenece cada alumno.

Es una clasificación **multiclase**, porque existen tres posibles resultados.

### Variable Y

**Y = `riesgo`**

- `verde` → riesgo bajo de reprobar.
- `amarillo` → riesgo medio.
- `rojo` → riesgo alto de reprobar.

### Variables X candidatas

1. **`porcentaje_asistencia`:** puede ayudar porque una menor asistencia puede estar relacionada con un mayor riesgo.
2. **`tareas_entregadas`:** permite observar el cumplimiento de las actividades.
3. **`promedio_parciales`:** representa el desempeño académico que lleva el alumno.
4. **`horas_plataforma`:** permite conocer qué tanto utiliza el alumno la plataforma.
5. **`materias_reprobadas_previas`:** aporta información sobre su historial académico.
6. **`turno`:** permite revisar si existen diferencias entre los alumnos de turno matutino y vespertino.

### Hallazgos del EDA

Se observa una diferencia clara entre los grupos.

Los alumnos con riesgo **verde** tienen una asistencia promedio de 91% y un promedio de parciales de 8.4. En cambio, los alumnos con riesgo **rojo** tienen una asistencia promedio de 52% y un promedio de parciales de 5.0.

También aumenta el número de materias reprobadas anteriormente:

| Riesgo | Asistencia promedio | Promedio de parciales | Materias reprobadas previas |
|---|---:|---:|---:|
| Verde | 91% | 8.4 | 0.2 |
| Amarillo | 76% | 6.8 | 0.9 |
| Rojo | 52% | 5.0 | 2.4 |

Esto muestra un patrón: conforme aumenta el riesgo, disminuyen la asistencia y el promedio, mientras que aumentan las materias reprobadas previamente.

### Distribución de Y

| Riesgo | Cantidad | Porcentaje |
|---|---:|---:|
| Verde | 120 | 40% |
| Amarillo | 105 | 35% |
| Rojo | 75 | 25% |

La distribución no está perfectamente balanceada, pero tampoco existe una clase extremadamente pequeña.

Por esta razón, no usaría solamente el porcentaje de aciertos para evaluar el modelo. También revisaría el desempeño de cada clase por separado.

### Calidad de los datos

Un posible problema sería encontrar valores fuera de los rangos esperados. Por ejemplo, una asistencia de 150% no sería válida o una cantidad negativa de tareas entregadas tampoco tendría sentido.

Para detectarlo revisaría los valores mínimos y máximos de cada variable y también buscaría datos faltantes y registros duplicados.

### Fuga de información

No utilizaría la **calificación final del curso** como variable de entrada si el objetivo es conocer el riesgo del alumno antes de terminar el curso.

La razón es que la calificación final se obtiene después y contiene información que todavía no estaría disponible en el momento en que se quiere identificar el riesgo.

### Métricas

Usaría:

- **Exactitud:** para conocer el porcentaje general de predicciones correctas.
- **Precisión:** para saber qué tan correctas fueron las predicciones de cada clase.
- **Sensibilidad:** para revisar qué tan bien se detectan los casos de una clase.
- **F1:** para combinar precisión y sensibilidad.
- **F1 promedio:** para tener una medida general considerando las tres clases.

### Modelo propuesto

Después del EDA propondría un **árbol de decisión**.

Una condición importante sería contar con suficientes ejemplos de las tres categorías y asegurar que las variables utilizadas estén disponibles antes de calcular el riesgo.

---

## Misión 2 — Alerta de Abandono Estudiantil

### Pregunta de negocio

¿Este alumno abandonará la materia a mitad del semestre?

### Tipo de problema propuesto

**Clasificar.**

La variable objetivo solamente puede tomar dos valores:

- `0` → permanece.
- `1` → abandona.

Por lo tanto, es una **clasificación binaria**.

### Variable Y

**Y = `abandona`**

### Distribución de Y

El conjunto tiene 500 alumnos:

| Resultado | Cantidad | Porcentaje |
|---|---:|---:|
| Permanece | 430 | 86% |
| Abandona | 70 | 14% |

Por lo tanto:

**500 × 0.14 = 70 alumnos**

Hay 70 casos de abandono.

Esto significa que un modelo que simplemente dijera "todos permanecen" tendría aproximadamente 86% de aciertos, pero no estaría detectando ningún alumno que abandona.

Por eso, en este caso no sería suficiente reportar solamente el porcentaje de aciertos.

### Variables X candidatas

1. **`dias_sin_acceso`:** un número alto puede indicar que el alumno dejó de utilizar la plataforma.
2. **`participaciones_foros`:** permite observar qué tanto participa el alumno.
3. **`avance_contenido`:** indica el porcentaje de contenido que ha revisado.
4. **`calificacion_actividad_1`:** puede mostrar el desempeño inicial.
5. **`tiene_beca`:** permite analizar si existe alguna relación entre tener beca y abandonar.
6. **`trabaja`:** permite revisar si existe alguna relación entre trabajar y abandonar.

### Valores faltantes

La variable `calificacion_actividad_1` tiene varios valores faltantes.

Una posible explicación es que algunos alumnos no hayan realizado la actividad. Esto podría ser importante porque precisamente los alumnos con mayor riesgo de abandono podrían ser quienes no tienen una calificación.

Una opción sería crear una variable adicional llamada `tiene_calificacion_actividad_1`, indicando:

- `1` → sí tiene calificación.
- `0` → no tiene calificación.

Después se podría analizar si la ausencia de la calificación está relacionada con el abandono.

También se tendría que decidir cómo tratar los valores faltantes antes de entrenar cualquier modelo.

### Hallazgos del EDA

La relación más fuerte observada es entre los **días sin acceso** y el abandono. Los alumnos que llevan más días sin entrar presentan mayor relación con el abandono.

También se observa una relación fuerte entre un **menor avance del contenido** y abandonar la materia.

La relación entre tener beca y abandonar es moderada, mientras que trabajar presenta una relación débil a moderada.

| Variable | Relación observada con abandono |
|---|---|
| Días sin acceso | Fuerte |
| Avance del contenido | Fuerte |
| Tener beca | Moderada |
| Trabajar | Débil a moderada |
| Falta de calificación de actividad 1 | Fuerte, pero debe revisarse |

### Fuga de información

No utilizaría variables como una **fecha definitiva de baja** o una **calificación final** para predecir el abandono a mitad del semestre.

Estas variables podrían contener información que solamente se conoce después de que el evento ya ocurrió.

### Comparación con la Misión 1

Las dos misiones son problemas de clasificación porque la variable Y representa categorías.

La diferencia es que la Misión 1 tiene **tres categorías** de riesgo, mientras que esta misión solamente tiene **dos resultados**: abandona o permanece.

Por eso la primera es multiclase y esta es binaria.

### Métricas

En este caso pondría especial atención en:

- **Sensibilidad:** porque es importante detectar a los alumnos que realmente abandonarán.
- **Precisión:** para evitar señalar como posibles desertores a demasiados alumnos que realmente permanecerán.
- **F1:** para considerar ambas medidas.
- **Matriz de confusión:** para observar los aciertos y errores de cada grupo.

### Modelo propuesto

Después del EDA propondría un **árbol de decisión**.

El modelo tendría que evaluarse especialmente en su capacidad para detectar los casos de abandono, ya que son los casos menos frecuentes y los que podrían requerir atención de la universidad.

---

## Misión 3 — Pronóstico de Puntaje Final

### Pregunta de negocio

¿Qué calificación final obtendrá un alumno?

### Tipo de problema propuesto

**Predecir.**

En esta misión la variable objetivo es una calificación numérica de 0 a 100. Por lo tanto, se busca estimar un número y no asignar al alumno a una categoría.

### Variable Y

**Y = `calificacion_final`**

### Variables X candidatas

1. **`promedio_tareas`:** representa el desempeño en las tareas.
2. **`examen_1`:** muestra el resultado del primer examen.
3. **`examen_2`:** muestra el resultado del segundo examen.
4. **`porcentaje_asistencia`:** puede estar relacionado con el desempeño final.
5. **`horas_estudio_semana`:** representa el tiempo que el alumno dedica al estudio.

### Hallazgos del EDA

La calificación final tiene:

- Media: 73
- Mediana: 74
- Mínimo: 28
- Máximo: 99
- Desviación estándar: 13

Una de las relaciones más importantes es la de `examen_1` y `examen_2` con la calificación final. La actividad indica que ambas tienen una correlación fuerte con la calificación final, aproximadamente de **0.85**.

Esto significa que podrían ser variables importantes para estimar la calificación.

También se observa que `horas_estudio_semana` tiene una distribución con una cola hacia la derecha, porque algunos alumnos reportan entre 20 y 25 horas de estudio por semana.

### Media y mediana de horas de estudio

La media es de **5.5 horas**, mientras que la mediana es de **5 horas**.

La media es ligeramente mayor que la mediana, lo que puede indicar que algunos alumnos que estudian muchas horas están aumentando el promedio.

Por eso sería importante revisar esos valores antes de eliminarlos, porque podrían ser casos reales y no necesariamente errores.

### Conversión a aprobado/reprobado

Otra posibilidad sería convertir la calificación final en dos categorías:

- Aprobado.
- Reprobado.

Esto facilitaría convertir el problema en clasificación, pero se perdería información.

Por ejemplo, una calificación de 60 y una de 90 podrían quedar dentro del mismo grupo de "aprobado", aunque existe una diferencia importante entre ambas.

Por eso considero mejor conservar la calificación numérica si la pregunta de negocio realmente busca conocer el puntaje final.

### Valores mayores a 100

La actividad indica que existen tres registros con una calificación final mayor a 100 en el archivo original.

Como la escala establecida es de 0 a 100, esos valores parecen ser errores de captura.

Antes de eliminarlos revisaría los registros originales para confirmar si realmente son errores. Si se confirma que lo son, los corregiría o excluiría siguiendo el criterio definido para la limpieza de los datos.

### Métricas

Para este problema usaría:

- **MAE:** indica, en promedio, cuántos puntos se equivoca el modelo respecto a la calificación real.
- **RMSE:** permite dar mayor importancia a errores grandes.

### Si la relación fuera casi una línea

Si al comparar `examen_1` con `calificacion_final` se observa una relación casi lineal, un modelo sencillo que podría funcionar sería una **regresión lineal**.

Si la relación tuviera una forma diferente, por ejemplo con varios escalones, tendría sentido considerar otro modelo que pueda representar mejor esa forma.

### Modelo propuesto

Después del EDA propondría una **regresión lineal** como primera opción, porque la variable objetivo es numérica y algunas de las variables muestran una relación fuerte con ella.

---

## Misión 4 — Estimación de Tiempo de Estudio

### Pregunta de negocio

¿Cuántas horas adicionales necesita un alumno para dominar un tema?

### Tipo de problema propuesto

**Predecir.**

La variable objetivo representa una cantidad de horas, por ejemplo 2, 18 o 35 horas. Es un valor numérico continuo, por lo que considero que corresponde a un problema de predicción numérica.

### Variable Y

**Y = `horas_adicionales`**

### Variables X candidatas

1. **`puntaje_inicial`:** representa el conocimiento que tiene el alumno antes de comenzar.
2. **`dificultad_tema`:** permite considerar si el tema es fácil, medio o difícil.
3. **`videos_vistos`:** muestra cuánto material ha revisado.
4. **`porcentaje_ejercicios_correctos`:** indica qué tan bien está resolviendo los ejercicios.
5. **`dispositivo`:** permite revisar si existen diferencias según el dispositivo utilizado.

### Hipótesis sobre la dificultad

Una hipótesis que se puede plantear es:

> A mayor dificultad del tema, mayor cantidad de horas adicionales necesarias para dominarlo.

Los datos apoyan esta idea:

| Dificultad del tema | Cantidad de alumnos | Media de horas | Mediana |
|---|---:|---:|---:|
| Baja | 80 | 3.2 | 2.5 |
| Media | 90 | 7.8 | 7.0 |
| Alta | 70 | 16.5 | 14.0 |

Se observa un aumento importante en las horas conforme aumenta la dificultad.

### Distribución de horas adicionales

| Rango de horas | Porcentaje de alumnos |
|---|---:|
| 0–5 | 35% |
| 5–10 | 30% |
| 10–20 | 22% |
| 20–40 | 10% |
| Más de 40 | 3% |

La mayoría de los alumnos necesita menos de 20 horas adicionales.

Existe una pequeña cantidad de alumnos que necesita más de 40 horas, solamente el 3%.

### ¿Eliminar los valores mayores a 40?

No los eliminaría automáticamente.

Primero revisaría si realmente son errores o si corresponden a alumnos que tuvieron dificultades importantes con el tema.

Si los datos son reales, eliminarlos podría hacer que el modelo no aprenda sobre alumnos que necesitan mucho tiempo.

Por lo tanto, primero revisaría esos registros y después tomaría la decisión.

### Variables que podrían estar relacionadas

El `puntaje_inicial` y el `porcentaje_ejercicios_correctos` podrían estar relacionados entre sí.

Si ambas variables contienen información muy parecida, podrían ser redundantes.

Para comprobarlo compararía su relación mediante una matriz de correlaciones y también revisaría visualmente su comportamiento.

### Alternativa: convertir las horas en una categoría

También se podría crear una clasificación utilizando un límite de 15 horas:

- Menos de 15 horas → no necesita tutoría intensiva.
- 15 horas o más → necesita tutoría intensiva.

Esto tendría sentido si la universidad necesitara identificar rápidamente a los alumnos que requieren apoyo especial.

Sin embargo, se perdería la cantidad exacta de horas. Por ejemplo, ya no sería posible distinguir entre un alumno que necesita 16 horas y otro que necesita 30 horas.

Por eso considero que, para la pregunta original, es mejor mantener las horas como un valor numérico.

### Métricas

Usaría:

- **MAE:** para conocer el error promedio en horas.
- **RMSE:** para detectar si existen errores grandes en las estimaciones.

### Modelo propuesto

Propondría inicialmente una **regresión lineal**, siempre revisando antes si las relaciones entre las variables y las horas adicionales son suficientemente adecuadas para este modelo.

Antes de entrenarlo haría al menos estos dos chequeos:

1. Revisar los valores extremos de `horas_adicionales`.
2. Revisar la relación entre las variables numéricas para detectar posibles variables redundantes.

---

# Síntesis

| Misión | Variable Y | Tipo propuesto | Modelo inicial |
|---|---|---|---|
| 1. Semáforo Académico | `riesgo` | Clasificar | Árbol de decisión |
| 2. Abandono Estudiantil | `abandona` | Clasificar | Árbol de decisión |
| 3. Puntaje Final | `calificacion_final` | Predecir | Regresión lineal |
| 4. Tiempo de Estudio | `horas_adicionales` | Predecir | Regresión lineal |

La principal pista que utilicé para decidir el tipo de problema fue observar qué representa la variable Y. Si Y contiene categorías o clases, considero que es un problema de clasificación. Si Y representa un número que quiero estimar, considero que es un problema de predicción numérica.

El tipo de problema se deduce de la pregunta y de Y porque primero necesito saber **qué quiero obtener como resultado**: una categoría o un valor numérico. Después de entender esto mediante el EDA puedo elegir métricas y un modelo adecuado.