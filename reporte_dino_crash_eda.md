# Reporte EDA — Operación Dino Crash

# 1. Problema y dataset — Misión 1

## P1 — ¿Morirá en el siguiente frame?

### Pregunta

¿Con la información que tenemos en un momento determinado podemos saber si el dinosaurio morirá en el siguiente frame?

### Variable objetivo (Y)

La variable `muere_siguiente_frame` sería la variable objetivo.

- Tipo: binaria.
- Valores posibles:
  - `0` = no muere en el siguiente frame.
  - `1` = muere en el siguiente frame.

### Variables de entrada (X)

| Variable | ¿Por qué sería útil? |
|---|---|
| `velocidad` | Permite saber qué tan rápido se mueve el escenario. |
| `dist_obstaculo` | Indica qué tan cerca está el siguiente obstáculo. |
| `obstaculo_type` | Permite saber si el obstáculo es un cactus pequeño, cactus grande o pájaro. |
| `salto` | Indica si el dinosaurio está saltando. |
| `score` | Ayuda a conocer cuánto ha avanzado la partida. |
| `tiempo_ms` | Indica cuánto tiempo lleva la partida. |
| `frame` | Permite saber en qué momento de la partida se encuentra el dinosaurio. |

### Granularidad

Usaría un registro por frame, porque la pregunta busca saber qué ocurre inmediatamente después.

No sería conveniente usar solamente un resumen de toda la partida, porque se perdería la información de cada momento.

### Tamaño mínimo razonable

Consideraría necesario tener varias cientos de partidas y miles de frames. Por ejemplo, un mínimo inicial de 300 partidas permitiría tener diferentes situaciones, velocidades y tipos de obstáculos.

Lo importante no es solamente tener muchas filas, sino que existan suficientes ejemplos de partidas que terminan y que no terminan en cada situación.

### Riesgo de un dataset mal definido

Un problema sería registrar solamente los frames donde el dinosaurio muere. El modelo aprendería únicamente situaciones de muerte y no tendría suficientes ejemplos para diferenciar una situación segura de una peligrosa.

---

## P2 — ¿Cuántos puntos alcanzará la partida al morir?

### Pregunta

¿Cuántos puntos tendrá el jugador cuando termine la partida?

### Variable objetivo (Y)

La variable `score_final` sería la variable objetivo.

- Tipo: numérica.
- Puede tomar diferentes valores, por ejemplo 50, 120, 250, 500, etc.

### Variables de entrada (X)

| Variable | ¿Por qué sería útil? |
|---|---|
| `velocidad_inicio` | Indica con qué velocidad comenzó la partida. |
| `velocidad_max` | Indica la velocidad máxima alcanzada. |
| `tiempo_ms` | Indica cuánto duró la partida. |
| `obstaculos_total` | Cantidad de obstáculos encontrados. |
| `saltos_total` | Cantidad de saltos realizados. |
| `distancia_recorrida` | Permite conocer cuánto avanzó el dinosaurio. |
| `tipo_obstaculo_frecuente` | Indica qué tipo de obstáculo apareció con mayor frecuencia. |

### Granularidad

En este caso usaría una fila por partida completa.

No necesito guardar cada frame como una fila para responder directamente esta pregunta, porque el resultado que quiero obtener corresponde al final de la partida.

### Tamaño mínimo razonable

Usaría al menos unas 300 partidas y, de ser posible, más. Se necesitan partidas diferentes para evitar que el modelo aprenda solamente unos pocos casos.

### Riesgo de un dataset mal definido

Si solo se guardaran partidas de jugadores muy buenos, el modelo podría aprender que casi todas las partidas terminan con puntuaciones altas y tendría dificultades con jugadores o situaciones diferentes.

---

## P3 — ¿Qué tipo de obstáculo viene próximo?

### Pregunta

¿Podemos saber qué tipo de obstáculo aparecerá o se encontrará próximamente?

### Variable objetivo (Y)

La variable `obstaculo_tipo_siguiente` sería la variable objetivo.

- Tipo: categórica.
- Posibles valores:
  - `cactus_pequeño`
  - `cactus_grande`
  - `pajaro`

### Variables de entrada (X)

| Variable | ¿Por qué sería útil? |
|---|---|
| `velocidad` | La velocidad puede cambiar el tiempo disponible para reaccionar. |
| `dist_obstaculo` | Indica qué tan cerca está el obstáculo. |
| `salto` | Indica si el dinosaurio está en el aire. |
| `score` | Ayuda a conocer el momento de la partida. |
| `tiempo_ms` | Permite identificar en qué momento aparece el obstáculo. |
| `obstaculo_tipo_actual` | Puede ayudar a estudiar si existen patrones entre obstáculos. |
| `frame` | Indica la posición temporal dentro de la partida. |

### Granularidad

Usaría un frame o un evento relacionado con la aparición de un obstáculo.

### Tamaño mínimo razonable

Intentaría obtener al menos 300 partidas y varios miles de eventos de obstáculos. También es importante que los tres tipos de obstáculo aparezcan suficientes veces.

### Riesgo de un dataset mal definido

Si casi todos los registros contienen cactus pequeños y hay muy pocos pájaros, el modelo podría aprender a responder siempre “cactus pequeño” y parecer bueno aunque realmente no identifique correctamente los otros tipos.

---

# 2. Diccionario y muestra — Misión 2

El borrador contiene información útil para estudiar el comportamiento del dinosaurio. Las columnas propuestas son `sesion_id`, `frame`, `tiempo_ms`, `score`, `velocidad`, `obstaculo_tipo`, `dist_obstaculo`, `salto` y `muere`.

## Patrón observado en `muere=1`

En el frame 82 se observa:

- `velocidad = 6.8`
- `obstaculo_tipo = cactus_pequeño`
- `dist_obstaculo = 12`
- `salto = 0`
- `muere = 1`

Esto muestra que el dinosaurio estaba muy cerca de un cactus pequeño, a solamente 12 píxeles, y no estaba saltando.

Comparando con el frame anterior, la distancia pasó de 38 a 12 píxeles. Por lo tanto, parece existir una relación entre una distancia muy pequeña al obstáculo, no estar saltando y la muerte.

Sin embargo, una sola fila no es suficiente para asegurar que siempre ocurra de esa manera.

## ¿`score` es buena variable para predecir la muerte en el siguiente frame?

Por sí sola, no sería una buena variable.

El `score` indica cuánto ha avanzado la partida, pero no indica directamente qué tan cerca está el obstáculo ni qué está haciendo el dinosaurio.

Por ejemplo, dos frames pueden tener el mismo score aproximado, pero uno puede tener un obstáculo muy cerca y otro puede estar completamente libre.

Por eso `score` podría utilizarse como una variable adicional, pero no debería ser la principal para tomar la decisión.

## ¿Falta alguna columna crítica?

Sí. Agregaría información como:

- `altura`: altura actual del dinosaurio.
- `agachado`: indica si está agachado.
- `velocidad_y`: velocidad vertical del salto.
- `reaccion_tiempo`: tiempo aproximado que tarda el jugador en reaccionar.
- `obstaculo_ancho` y `obstaculo_altura`: tamaño del obstáculo.

Estas variables podrían ayudar a distinguir situaciones que actualmente parecen iguales.

## ¿`muere` sirve para P1?

No exactamente.

La columna `muere` está definida como `1` solamente en el último frame de la sesión. Por lo tanto, indica que ese frame es el final de la partida.

Para P1 necesitamos saber si el dinosaurio morirá en el siguiente frame. Sería mejor crear una variable como `muere_siguiente_frame`, donde el valor indique específicamente si el siguiente frame provoca la muerte.

---

# 3. Checklist EDA — Misión 3

## Pregunta 1: ¿Cuántas observaciones hay?

Primero revisaría cuántas partidas y cuántos frames existen.

Si tenemos pocos datos, no sería recomendable comenzar con un modelo muy complejo. Por ejemplo, si solamente hubiera 20 partidas, sería difícil representar todas las situaciones posibles del juego.

## Pregunta 3: ¿La clase objetivo está balanceada?

En P1 esperamos que existan muchos más frames donde el dinosaurio no muere que frames donde muere.

Esto significa que probablemente tendremos una clase desbalanceada.

Por ejemplo, si tenemos 12,000 frames y solamente 50 representan una muerte, aproximadamente:

50 / 12,000 × 100 = 0.42%

Esto significa que alrededor del 99.58% de los frames no representan una muerte.

Por eso, tener una accuracy muy alta no necesariamente significaría que el modelo funciona bien.

## Pregunta 4: ¿Hay data leakage?

Sí puede existir.

Un ejemplo sería usar información que solamente conocemos después de que ocurrió la muerte. Si queremos predecir la muerte en el siguiente frame, no podemos utilizar una variable que indique que el dinosaurio ya murió.

También debemos tener cuidado con `score` y `tiempo_ms`, porque pueden dar información relacionada con el momento de la partida y podrían terminar revelando indirectamente información que no debería estar disponible para la predicción.

## ¿Por qué mezclar frames de una misma partida es un error?

Los frames de una misma partida están relacionados entre sí.

Por ejemplo, si usamos algunos frames de la partida 7 para entrenar y otros frames de esa misma partida para probar, el modelo puede encontrarse en la prueba con una situación muy parecida a una que ya vio durante el entrenamiento.

Eso puede hacer que el resultado parezca mejor de lo que realmente es.

Por eso sería mejor separar entrenamiento y prueba por `sesion_id`, de manera que una partida completa quede solamente en uno de los grupos.

---

# 4. Interpretación de los resúmenes — Misión 4

## ¿P1 está desbalanceado?

Sí.

Tenemos aproximadamente 12,000 frames y solamente 50 casos con `muere=1`.

El porcentaje de muertes sería:

50 / 12,000 × 100 = 0.42%

Por lo tanto:

- Muerte: aproximadamente 0.42%
- No muerte: aproximadamente 99.58%

Es un desbalance muy grande.

## ¿Qué implica para las métricas?

La accuracy no debería ser la única métrica.

Un modelo podría decir siempre “no muere” y tendría cerca de 99.58% de accuracy, pero no estaría detectando las muertes.

Por eso sería más útil revisar métricas como:

- Precision: qué tan confiables son las predicciones de muerte.
- Recall: cuántas de las muertes reales logra detectar.
- F1: combina precision y recall.

En este caso, el recall es especialmente importante porque queremos detectar situaciones donde el dinosaurio puede morir.

## ¿`dist_obstaculo` parece útil?

Sí.

En los datos proporcionados se menciona que las muertes suelen ocurrir cuando `dist_obstaculo < 20`.

Además, en el frame 82, donde ocurre la muerte, la distancia es 12.

Esto indica que la distancia al obstáculo podría ser una variable importante para predecir la muerte.

Aun así, no debería utilizarse sola. También habría que considerar la velocidad, el tipo de obstáculo y el estado del dinosaurio.

## ¿Qué indica la distribución de `score`?

La media de `score` es 28 y la mediana es 18, mientras que el máximo llega a 120.

Como la media es mayor que la mediana, algunos valores altos están elevando la media.

Esto coincide con la descripción de una cola larga hacia la derecha.

Por lo tanto, no asumiría inmediatamente una relación lineal simple. Primero revisaría la distribución y la relación entre `score` y las demás variables.

---

# 5. Elección de modelo — Misión 5

El modelo debe elegirse después de revisar los datos.

| Escenario | Tipo de problema | Modelo propuesto | Condiciones que deben cumplirse |
|---|---|---|---|
| P1 | Clasificación binaria | Árbol de decisión | Tener suficientes ejemplos de muerte y no muerte; evitar fuga de información. |
| P2 | Regresión | Regresión lineal | Tener suficientes partidas y una relación razonable entre las variables y el score final. |
| P3 | Clasificación multiclase | Árbol de decisión | Tener suficientes ejemplos de cada tipo de obstáculo y variables que permitan diferenciarlos. |

## P1

La respuesta es una clasificación binaria porque queremos decidir entre dos opciones: muere o no muere.

Un árbol de decisión sería una opción razonable porque puede tomar decisiones sencillas a partir de variables como distancia, velocidad y estado del dinosaurio.

## P2

La respuesta es una regresión porque queremos obtener un número: la puntuación final.

La regresión lineal podría ser un primer modelo sencillo si los datos muestran una relación aproximadamente lineal.

## P3

Es una clasificación multiclase porque existen varias categorías posibles de obstáculo.

Un árbol de decisión podría ser una opción inicial porque permite separar los casos usando las características disponibles.

---

# 6. Contraejemplos — Misión 6

## ¿Cuándo no usaría un árbol profundo?

Un árbol profundo podría parecer buena idea si existen muchas variables y diferentes situaciones.

Sin embargo, si tenemos pocas partidas, un árbol muy profundo podría aprenderse de memoria los ejemplos disponibles.

Por ejemplo, si solo tenemos 50 partidas, no tendría mucho sentido utilizar un modelo demasiado complejo. Primero sería necesario obtener más datos y revisar el EDA.

## ¿Cuándo tendría sentido una red neuronal?

Una red neuronal tendría más sentido si tuviéramos una gran cantidad de partidas y muchos datos por partida.

También necesitaríamos observar que las variables contienen suficiente información para reconocer los patrones.

Si solamente tenemos un dataset pequeño y con pocas variables, comenzaría con modelos más sencillos antes de utilizar una red neuronal.

## ¿Se puede resolver P1 con reglas fijas?

Sí.

Por ejemplo:

`Si dist_obstaculo < 20 y salto = 0 → posible muerte`

### Ventajas

- Es fácil de entender.
- Es rápida.
- No necesita entrenamiento.
- Puede servir como una primera solución sencilla.

### Limitaciones

- El valor 20 es fijo y puede no funcionar para todas las velocidades.
- No considera todos los tipos de obstáculos.
- No aprende de nuevos datos.
- Puede fallar en situaciones que no estén contempladas en la regla.

Un modelo aprendido podría encontrar relaciones entre varias variables al mismo tiempo y adaptarse mejor a diferentes situaciones.

---

# 7. Síntesis

Primero pediría un dataset que registre las partidas, los frames, la velocidad, la distancia al obstáculo, el tipo de obstáculo y el estado del dinosaurio.

Después revisaría la cantidad de datos, valores faltantes, datos extraños, balance de las clases y posibles fugas de información.

Para P1 propondría clasificación binaria porque queremos saber si ocurre una muerte o no.

Para P2 propondría regresión porque queremos estimar una puntuación numérica.

Para P3 propondría clasificación multiclase porque queremos identificar el tipo de obstáculo.

La idea principal es que el modelo no debe elegirse solamente porque sea conocido o avanzado. Primero debemos entender la forma de los datos y después elegir una opción adecuada.
