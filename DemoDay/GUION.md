# NextLap · Demo Day · Guion (Grupo 8)

> **La presentación es en inglés:** el guion oficial es `SCRIPT.md` y las tarjetas son `CARDS.pdf`. Este archivo queda solo como traducción de referencia al español.

**Viernes 20 nov · 13:05–13:12 presentación · 13:12–13:17 preguntas · Lab 033 RESB**
Andy Villarroel + Agustín Reyes · bloque de 7 min de presentación (usamos **~5 min**), 5 de preguntas. Ambos responden.

Reparto: **Andy** abre y cierra (slides 1, 2, 6). **Agustín** cuenta los resultados (slides 3, 4, 5).
Ritmo: ~140 palabras por minuto, ~5 min en total. Si sobra tiempo, pueden alargar el caso de la slide 4 (tienen hasta 7:00).

---

## Cómo usar el deck

**Truco: el slide es el guion.** El texto de cada slide está en el mismo orden en que se dice. Lean de arriba hacia abajo y de izquierda a derecha, apuntando a lo que dicen. Para pillar el hilo si se pierden: `TARJETAS.pdf` (una hoja por persona, en la mano o en el celular).

- Abrir `NextLap_DemoDay.html` en **Chrome** → `F` pantalla completa.
- `→` / espacio / click = avanzar · `←` = volver · `1`–`8` = saltar a slide.
- `N` = muestra el guion abajo (solo para practicar, **no** con el proyector conectado en espejo).
- `T` = cronómetro (`mm:ss / 05:00`, se pone rojo si van atrasados en esa slide). Parte solo al pasar a la slide 2.
- Slides 7 y 8 = **respaldo** para preguntas (no se presentan).
- Si el HTML falla: `NextLap_DemoDay_backup.pdf` (mismo contenido, estático).
- Funciona **sin internet** (fuentes y Three.js van dentro del archivo).

---

## 01 · Título — ANDY · 0:00–0:35

> Hola, somos Andy Villarroel y Agustín Reyes, grupo 8, y este es NextLap. La pregunta: cuando un piloto termina una vuelta, **¿cuánto va a tardar en la siguiente?** Predecimos ese tiempo en segundos al final de la vuelta *t*, usando solo lo que ya se sabe. Podría ayudar a un ingeniero a anticipar si el ritmo se cae o se recupera. Todo es validación sobre carreras pasadas, no una prueba en vivo.

---

## 02 · Qué cambiamos — ANDY · 0:35–1:20

> El modelo base usa el promedio de las últimas vueltas, pero el promedio no distingue a un piloto **regular** de uno que va **a saltos**. Agregamos la **dispersión**: cuánto varían sus últimas hasta tres vueltas limpias, calculada hasta la vuelta *t*, nunca después. Si hay menos de dos, usamos una bandera y la mediana del entrenamiento. Entrenamos con las carreras 1 a 7 y validamos en Austria y Gran Bretaña: **las mismas 1.808 vueltas para todos**. Antes de mirar resultados pasamos cuatro chequeos anti-fuga. El control es el mismo modelo **sin** la dispersión. Agustín.

---

## 03 · Resultado — AGUSTÍN · 1:20–2:25

> Medimos el **MAE**: en promedio, por cuántos segundos fallamos. Menos es mejor. La referencia del curso: **0,3867**. Nuestro candidato, con la dispersión: **0,3697**. El control, igual pero sin la dispersión: **0,3877**; la bandera sola no ayuda. Control menos candidato: **+0,018 segundos** a favor del candidato, cerca de 4,6 % menos error. Un bootstrap por piloto-carrera da un intervalo de **0,009 a 0,028**: no incluye cero. Y se cumple la regla que fijamos en el Hito 1 antes de correr nada. Siendo honestos: es una mejora real, pero **pequeña** frente a un error típico de 0,37.

---

## 04 · Dónde le cuesta — AGUSTÍN · 2:25–3:20

> Mejora en las dos carreras: Austria **0,013**, Gran Bretaña **0,025**. Pero casi toda la ganancia viene de **ventanas inestables**: ahí baja **0,085**. Con poco historial, empeora un poco. Caso real: el piloto 23 en Silverstone venía en 96 segundos y hace una vuelta de **105**. El control se la cree y predice 102,1: falla por **4,2**. El candidato ve la ventana inestable y predice **99,1**; la real fue **97,9**, error de **1,24**. El límite: la dispersión **no sabe la dirección**; si el ritmo lento se mantiene, corrige de más.

---

## 05 · SHAP — AGUSTÍN · 3:20–4:20

> ¿Qué movió esa predicción? Con **SHAP** sobre el candidato entrenado: partimos de **86,73**, el promedio de entrenamiento. El promedio reciente suma **8,2** y la vuelta actual **6,7**, porque Silverstone es largo. La dispersión **resta 2,47**: con una ventana inestable, el modelo espera una vuelta más rápida. Base más aportes da **99,121**, exactamente la predicción. Esto explica la predicción, no el error, que es aparte: 1,24. En global, en las 1.808 vueltas, la dispersión es el **tercer input**. Pero que mejore el error lo prueba la ablación, no SHAP. Andy.

---

## 06 · Decisión — ANDY · 4:20–5:00

> Decisión: **mantenemos la dispersión.** Mismo modelo, mismas vueltas, mismo control: mejora en las dos carreras y el intervalo no incluye cero. Aplica a regresión lineal, vueltas limpias, Austria y Gran Bretaña; no con historial corto, pits ni otros circuitos. Una variante sin bandera dio algo mejor, pero la elegimos después de ver validación, así que no la adoptamos. Siguiente chequeo: una **desviación con signo**, que sepa si la vuelta fue más lenta o más rápida. **Gracias, ¿preguntas?**

---

## Preguntas probables (5 min · ambos responden)

Regla: respuesta corta → número → límite. Si no saben algo: "no lo probamos; lo que sí medimos es…".

| Pregunta | Quién | Respuesta corta |
|---|---|---|
| ¿Cómo saben que no usan información del futuro? | **Andy** | Cuatro chequeos fijados en Hito 1, corridos **antes** de ver el MAE. Borramos todas las vueltas después de *t* en 700 filas (500 train + 200 validación, seed 414): 0 cambios. Ninguna de las 7.580 filas usa una vuelta que termine después de *t*. Slide 7 (respaldo A). |
| ¿Por qué extendieron el chequeo a validación? | **Andy** | Las filas de validación son las que puntuamos; una fuga ahí sesgaría directo la comparación. Límite: es una muestra de 700, correrlo en las 7.580 es barato y es el siguiente paso. |
| ¿Qué pasa si no hay historial suficiente? | **Andy** | Con < 2 vueltas limpias: `has_history2 = 0` y la mediana de train (0,258 s), ajustada solo en train. Son 148 filas de validación (8 %). Ahí no mejora: −0,008 s, y en R9 −0,023 (n = 66). |
| ¿Qué es el filtro *clean pair*? ¿No es trampa? | **Andy** | Selecciona pares donde *t* y *t+1* son vueltas limpias. Usa info de *t+1*, por eso **solo define la población evaluada** y nunca entra como input. Por lo mismo, el resultado no aplica a pits ni safety car. |
| ¿Por qué no se quedaron con T3 si era mejor (0,3652)? | **Agustín** | Lo probamos **después** de ver validación; elegirlo sería ajustar sobre validación. La brecha T3−T1 es 0,0045 s, chica frente al intervalo [0,009; 0,028]. Queda como siguiente chequeo. Slide 8. |
| 0,018 s, ¿sirve de algo? | **Agustín** | Es pequeño en promedio (≈ 4,6 %), pero en ventanas inestables es 0,085 s, y el caso L7 baja de 4,2 a 1,2 s. No prometemos rendimiento operativo: es validación retrospectiva en 2 carreras. |
| ¿Por qué un bootstrap por piloto-carrera? | **Agustín** | Las vueltas de un mismo piloto en una carrera están correlacionadas; remuestrear filas sueltas daría un intervalo demasiado estrecho. 43 clusters, 2.000 réplicas, seed 414. |
| ¿Cómo eligieron el caso que muestran? | **Agustín** | Regla fijada antes de mirar: la fila de validación donde la dispersión movió más la predicción, máx \|pred(c) − pred(b)\|. No lo elegimos por ser el que mejor sale. |
| ¿Cuál fue el peor error? | **Agustín** | `2026-R9-23-L6`, 8,25 s: la vuelta lenta es el **objetivo** y nada visible en L6 la anuncia (σ = 0,98). Ninguna variable de historial puede arreglarlo. |
| ¿Qué es el "base value" de SHAP? | **Agustín** | La predicción promedio sobre el fondo de 5.772 filas de entrenamiento: 86,734 s. LinearExplainer, masker independiente. Base + aportes = predicción (error máx 1,4e-14 s en 1.808 filas). |
| ¿SHAP dice que la dispersión mejora el error? | **Agustín** | No. SHAP explica la salida del modelo; que mejore el error lo muestra la ablación (control vs candidato, mismas filas). |
| ¿Por qué Gran Bretaña tiene más error? | ambos | Todos los modelos fallan más ahí (control 0,41 vs 0,37). Es también donde más ayuda la dispersión (+0,025). Con 2 carreras no podemos separar si es el circuito o las condiciones. |
| ¿Funcionaría con otro modelo (árboles, boosting)? | ambos | No lo probamos. La afirmación es solo para OLS; un modelo no lineal podría capturar la inestabilidad de otra forma. |
| ¿Por qué ventana de 3 vueltas? | **Andy** | Es la misma ventana que usa la referencia para `recent_mean_s`, así son comparables. Con mínimo 3 vueltas (T2) la ganancia fue menor (+0,0156). |
| ¿Cómo usaron IA? | ambos | Claude como asistente de código: Andy para `history_features()` y los asserts; Agustín para bootstrap y SHAP. La hipótesis, la regla de decisión y el modelo reportado los decidimos nosotros. Lo verificamos con los chequeos: reconstrucción de `recent_mean_s`, reproducción fila a fila de la referencia, reconstrucción SHAP. |
| ¿Qué hizo cada uno? | cada uno | **Andy**: variable sobre la línea de tiempo original, los 4 chequeos anti-fuga, reproducibilidad (README, hashes, Run All). **Agustín**: ablación a/b/c, T2/T3, errores por carrera y slices, caso inspeccionado, bootstrap, SHAP. |

**La decisión que defiende cada uno (la piden):**
- **Andy** → extender el chequeo de truncado a 200 filas de validación. Resultado: 700 filas, 0 cambios. Límite: es una muestra.
- **Agustín** → mantener (c) y no cambiar a T3. Resultado: +0,018 s, IC [+0,009; +0,028]. Límite: la ganancia está concentrada en dispersión alta y empeora con historial corto en R9.

---

## Checklist para mañana

- [ ] Ensayar 1 vez con cronómetro (`T`). Meta: terminar la slide 6 cerca de 5:00 (máximo 7:00).
- [ ] Copiar `NextLap_DemoDay.html` **y** `NextLap_DemoDay_backup.pdf` a un pendrive + mandárselos a Agustín.
- [ ] Abrir el HTML en el computador del lab antes (13:00). Si va lento: el PDF.
- [ ] Llegar con el repo abierto en el notebook §5 / §8 por si piden ver el código.
- [ ] Anotar el feedback del jurado: lo necesitan para la sección 4 del informe final (domingo 22, 23:59).
