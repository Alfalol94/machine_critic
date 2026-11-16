![Machine Critic](Machine_critic.png)

#  Machine Critic

**Predicción del alcance de videojuegos de Steam con Machine Learning**

Machine Critic es un proyecto de Machine Learning que estima qué tan lejos llegará un videojuego en Steam (cuántos jugadores lo tendrán) a partir de sus características técnicas y de diseño. El problema se abordó de tres formas: **clasificación binaria**, **clasificación multiclase** y **regresión**.

**Autores:** Andrés Felipe Mantilla Almeyda · Manuel Alejandro Rojas Sierra

Link Video de Presentacion :
--->
---

##  Tabla de contenidos

1. [Motivación y objetivo](#-motivación-y-objetivo)
2. [Dataset](#-dataset)
3. [Diccionario de variables](#-diccionario-de-variables)
4. [Metodología](#-metodología)
5. [Resultados](#-resultados)
6. [Conclusiones](#-conclusiones)
7. [Estructura del repositorio](#-estructura-del-repositorio)
8. [Cómo ejecutarlo](#-cómo-ejecutarlo)
9. [Trabajo futuro](#-trabajo-futuro)

---

##  Motivación y objetivo

Muchos desarrolladores independientes sienten que sus juegos no reciben el apoyo esperado y no saben hacia dónde enfocar su desarrollo para destacar en la plataforma. Las empresas grandes, por su parte, tienden a repetir fórmulas que ya funcionan y pierden la identidad de sus juegos.

**Objetivo:** Construir un modelo que estime el alcance de un videojuego (qué tan de nicho es) para:

- Orientar el desarrollo con base en datos.
- Simular escenarios y apoyar la toma de decisiones.
- Aportar valor tanto a estudios pequeños como a empresas que no quieren arriesgarse demasiado.

**¿Por qué Machine Learning?** Puede analizar muchas variables a la vez sobre decenas de miles de juegos, detectar combinaciones que no se ven a simple vista y es reproducible y mejorable.

>  **Limitaciones:** factores como el marketing o el papel de las tiendas influyen en el éxito de un juego y no están en los datos.

---

##  Dataset

- **Fuente:** [Steam Games Dataset – FronkonGames](https://huggingface.co/datasets/FronkonGames/steam-games-dataset) (Hugging Face), con modificaciones propias.
- **Tamaño original:** ~124.000 juegos y 41 columnas.
- **Tamaño tras la limpieza:** 84.094 juegos, con 37 variables de entrada.
- **Variable objetivo:** `estimated_owners` (número estimado de dueños del juego).

### Limpieza y preparación

- Tratamiento de textos vacíos (no eran nulos, eran strings vacíos), duplicados y columnas irrelevantes.
- Eliminación de juegos sin dueños, sin reseñas o con pico de jugadores simultáneos igual a 0 (de 124.146 filas sobrevivieron 84.094 filas).
- Columnas tipo lista (`genres`, `categories`) separadas en columnas binarias.
- Columnas de plataformas (Windows, Mac, Linux) y otras características convertidas a binario.
- Variable objetivo: el rango de dueños se convirtió a un valor numérico como el punto intermedio del intervalo y se transformó a 7 clases (0 a 6) para clasificación. Ver el [diccionario de variables](#-diccionario-de-variables).

### Hallazgos del EDA

- El catálogo está dominado por pocos géneros (Indie, Action, Casual) y categorías (Single-player, Family Sharing, Steam Achievements).
- La actividad real (reseñas, jugadores simultáneos) está fuertemente correlacionada entre sí; el precio y la calificación de los usuarios **no** predicen el éxito.
- Aproximadamente el 65 % de los juegos vende menos de 20.000 copias estimadas: mercado extremadamente desigual y distribuciones con sesgo fuerte.
- Tener sitio web o portal de soporte se asocia con más dueños (posible señal del tamaño o profesionalismo del estudio).

---

##  Diccionario de variables

El dataset original tiene **41 columnas**. Tras la limpieza y la ingeniería de variables, los modelos usan **37 variables de entrada** y **1 variable objetivo**.

### Variables numéricas (de Steam)

| Variable | Tipo | Descripción | Tratamiento |
|---|---|---|---|
| `peak_ccu` | numérica (entero) | Pico de jugadores simultáneos | Sin cambios |
| `price` | numérica (decimal) | Precio del juego | Sin cambios |
| `dlc_count` | numérica (entero) | Cantidad de contenido descargable (DLC) | Sin cambios |
| `metacritic_score` | numérica (entero) | Puntaje en Metacritic (0 si no tiene) | Sin cambios |
| `user_score` | numérica (entero) | Puntaje de los usuarios (0–100) | Sin cambios |
| `positive` | numérica (entero) | Número de reseñas positivas | Sin cambios (en el EDA se graficó `log(1+x)`) |
| `negative` | numérica (entero) | Número de reseñas negativas | Sin cambios (en el EDA se graficó `log(1+x)`) |
| `achievements` | numérica (entero) | Cantidad de logros del juego | Sin cambios |
| `recommendations` | numérica (entero) | Número de recomendaciones | Sin cambios |
| `average_playtime_forever` | numérica (entero) | Tiempo de juego promedio histórico | Sin cambios |
| `median_playtime_forever` | numérica (entero) | Mediana del tiempo de juego histórico | Sin cambios |

### Variables binarias (presencia / plataforma)

| Variable | Descripción | Tratamiento |
|---|---|---|
| `website` | 1 si el juego tiene sitio web | Texto vacío → 0, con contenido → 1 |
| `support_url` | 1 si tiene URL de soporte | Texto vacío → 0, con contenido → 1 |
| `support_email` | 1 si tiene correo de soporte | Texto vacío → 0, con contenido → 1 |
| `windows` | 1 si está disponible en Windows | Booleano → 0/1 |
| `mac` | 1 si está disponible en macOS | Booleano → 0/1 |
| `linux` | 1 si está disponible en Linux | Booleano → 0/1 |

### Grupos de categorías (columnas binarias creadas a partir de `categories`)

Cada columna vale 1 si el juego tiene **al menos una** de las etiquetas del grupo.

| Variable | Etiquetas de Steam que agrupa |
|---|---|
| `Single_player` | Single-player |
| `Multiplayer` | Multi-player, PvP, Online PvP, LAN PvP, MMO, Shared/Split Screen PvP, Cross-Platform Multiplayer |
| `Coop` | Co-op, Online Co-op, LAN Co-op, Shared/Split Screen Co-op, Shared/Split Screen, Remote Play Together |
| `Controller` | Full / Partial / Tracked controller support, Touch Only, Mouse Only y Keyboard Only Option |
| `Accessibility` | Adjustable Difficulty, Adjustable Text Size, Camera Comfort, Captions available, Chat Speech-to-text, Chat Text-to-speech, Color Alternatives, Custom Volume Controls, Narrated Game Menus, Subtitle Options, Playable without Timed Input, Save Anytime, Stereo Sound, Surround Sound |
| `Platafforms` *(así se llama en el código)* | VR Only, VR Support, VR Supported, SteamVR Collectibles, Remote Play on Phone / TV / Tablet |
| `Steam` | Steam Cloud, Steam Trading Cards, Steam Turn Notifications, Steam Achievements, Stats, Steam Leaderboards, Steam Timeline |
| `Family_Sharing` | Family Sharing |
| `Otros` | In-App Purchases, Valve Anti-Cheat enabled, HDR available, Commentary available, Steam Workshop, Mods, Mods (require HL2), Includes level editor, Includes Source SDK |

### Grupos de géneros (columnas binarias creadas a partir de `genres`)

| Variable | Géneros de Steam que agrupa |
|---|---|
| `Indie` | Indie |
| `Action` | Action |
| `Casual` | Casual |
| `Adventure` | Adventure |
| `Simulation` | Simulation |
| `Strategy` | Strategy |
| `RPG` | RPG |
| `Sports_Racing` | Sports, Racing |
| `Mature_Content` | Violent, Gore, Nudity, Sexual Content |
| `Free to play` | Free To Play |
| `Otros generos` | Early Access, Massively Multiplayer, Utilities, Design & Illustration, Animation & Modeling, Education, Video Production, Game Development, Audio Production, Software Training, Photo Editing, Web Publishing, Accounting, Movie, Documentary, Episodic, Short, Tutorial, 360 Video |

### Variable objetivo: `estimated_owners`

En el dataset original viene como un **rango de texto** (por ejemplo `"0 - 20000"`). Se transformó así:

1. Se eliminaron los juegos con rango `"0 - 0"`.
2. Se calculó el **punto medio** de cada rango (por ejemplo `"0 - 20000"` → 10.000).
3. Para **regresión** se usa `log10(punto medio)`.
4. Para **clasificación multiclase** se asignó una clase de 0 a 6:

| Clase | Punto medio de dueños | Juegos |
|---|---|---|
| 0 | 10.000 | 58.544 |
| 1 | 35.000 | 11.386 |
| 2 | 75.000 | 5.352 |
| 3 | 150.000 | 3.450 |
| 4 | 350.000 | 2.854 |
| 5 | 750.000 | 1.154 |
| 6 | 1.500.000 o más (rangos superiores agrupados) | 1.354 |

5. Para **clasificación binaria**: clases 0–1 → **0** (poco alcance, 69.930 juegos, 71,2 %); clases 2–6 → **1** (populares, 14.164 juegos, 28,8 %).

### Columnas y filas eliminadas

**Filas (de 124.146 a 84.094):**

| Filtro | Cantidad |
|---|---|
| Juegos con `estimated_owners = "0 - 0"` (sin dueños, p. ej. playtests) | 23.033 |
| Juegos sin pico de jugadores, reseñas positivas ni negativas (`peak_ccu`, `positive` y `negative` = 0) | resto hasta llegar a 84.094 |

**Columnas:**

| Variable(s) | Motivo |
|---|---|
| `appID`, `required_age`, `full_audio_languages`, `supported_languages`, `header_image`, `notes`, `average_playtime_2weeks`, `median_playtime_2weeks`, `developers`, `publishers`, `screenshots`, `movies` | Innecesarias o irrelevantes por el momento (varias con muchos valores vacíos) |
| `tags` | Ningún juego tenía etiquetas (columna vacía) |
| `name`, `release_date`, `detailed_description`, `short_description`, `reviews` | Texto libre o fecha, no usados en los modelos |
| `categories`, `genres` | Reemplazadas por las columnas binarias agrupadas |
| `metacritic_url`, `score_rank`, `packages` | Descartadas antes de modelar |

---

##  Metodología

### Partición de datos

Los mismos conjuntos se usaron en todos los experimentos (`random_state=31`, estratificado):

| Conjunto | Proporción | Uso |
|---|---|---|
| Entrenamiento | 60 % | Ajuste de modelos |
| Validación | 20 % | Selección de hiperparámetros y modelos |
| Prueba (test) | 20 % | Evaluación final |

### Tareas

1. **Clasificación binaria:** juegos de *poco alcance* (clases 0–1, 71,2 %) vs. *populares* (clases 2–6, 28,8 %).
2. **Clasificación multiclase:** las 7 clases originales de `estimated_owners`.
3. **Regresión:** predicción de `log10(estimated_owners)`.

### Modelos

| Tarea | Modelos evaluados |
|---|---|
| Clasificación binaria | Gaussian Naive Bayes, Decision Tree, Random Forest, SVM |
| Clasificación multiclase | Random Forest, Decision Tree (los 2 mejores de la tarea binaria) |
| Regresión | Regresión lineal, Decision Tree, Random Forest, SVR |

Los hiperparámetros se exploraron con barridos y gráficas de entrenamiento vs. validación para detectar sobreajuste.

### Métricas

- **Clasificación:** F1 como métrica principal (la precisión desempata), junto con accuracy, precision, recall y especificidad. Se priorizó la precisión porque un falso positivo (predecir popularidad que no llega) es más costoso que un falso negativo.
- **Regresión:** R², MAE y RMSE (en escala `log10`).

---

## 📊 Resultados

### Clasificación binaria (validación)

| Modelo | Accuracy | Precision | Recall | F1 |
|---|---|---|---|---|
| **Random Forest** | 0.929 | 0.872 | 0.680 | **0.764** |
| Árbol de decisión | 0.923 | 0.889 | 0.618 | 0.729 |
| Gaussian Naive Bayes | 0.865 | 0.768 | 0.283 | 0.413 |

### Clasificación multiclase (7 clases)

| Modelo | Accuracy | Precision macro | Recall macro | F1 macro |
|---|---|---|---|---|
| **Random Forest** | 0.763 | 0.528 | 0.450 | **0.468** |
| Árbol de decisión | 0.745 | 0.373 | 0.394 | 0.381 |

Los modelos predicen bien las clases extremas (menos y más dueños) y confunden las intermedias con sus vecinas; el árbol no logró acertar las clases 2 y 5.

### Regresión

**Validación**

| Modelo | MAE | RMSE | R² | R² train |
|---|---|---|---|---|
| **Random Forest** | 0.182 | 0.271 | **0.742** | 0.779 |
| Árbol de decisión | 0.190 | 0.292 | 0.701 | 0.752 |
| SVR | 0.249 | 0.374 | 0.510 | 0.542 |
| Regresión lineal (Ridge) | 0.324 | 0.509 | 0.092 | 0.336 |

**Prueba final (test)**

| Modelo | MAE | RMSE | R² |
|---|---|---|---|
| **Random Forest** | 0.182 | 0.267 | **0.751** |
| Árbol de decisión | 0.189 | 0.282 | 0.723 |

Los resultados en test son consistentes con los de validación (sin sobreajuste serio). Los modelos tienden a subestimar a los juegos con gran éxito.

> _SVR se entrenó con una muestra de 10.000 filas por su costo computacional._

---

##  Conclusiones

- El mercado de Steam presenta un **fuerte desbalance de clases**.
- Predecir el número exacto de copias es extremadamente difícil por la altísima dispersión de los datos.
- Aun así, **los patrones existen y los modelos son utilizables**: Random Forest fue el mejor modelo tanto en clasificación como en regresión, seguido del árbol de decisión.
- La relación entre las características y el alcance **no es lineal** (la regresión lineal obtuvo R² ≈ 0.09).
- Es indispensable conocer el tema y el dataset antes de modelar.

---

##  Estructura del repositorio

```
Machine-Critic/
├── README.md
├── MachineLearningAvance3_regresion.ipynb
├── presentacion_avance_1.pptx
├── Segundo_avance_Machine_project.pdf
└── Machine_Critic_Avance_3.pdf

```

---


**Librerías principales:** `pandas`, `numpy`, `matplotlib`, `seaborn`, `scikit-learn`.


## Créditos

- Dataset original: [FronkonGames/steam-games-dataset](https://huggingface.co/datasets/FronkonGames/steam-games-dataset).
- Proyecto académico desarrollado para el curso de Machine Learning.
