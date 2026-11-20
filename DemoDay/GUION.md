# NextLap · Demo Day · Guion (Grupo 8)

**Viernes 20 nov · 13:05–13:12 presentación · 13:12–13:17 preguntas · Lab 033 RESB**
Andy Villarroel + Agustín Reyes · 7 min de presentación, 5 de preguntas. Ambos responden.

Reparto: **Andy** abre y cierra (slides 1, 2, 6). **Agustín** cuenta los resultados (slides 3, 4, 5).
Ritmo: ~140 palabras por minuto. Si van atrasados, recorten lo que está entre [corchetes].

---

## Cómo usar el deck

- Abrir `NextLap_DemoDay.html` en **Chrome** → `F` pantalla completa.
- `→` / espacio / click = avanzar · `←` = volver · `1`–`8` = saltar a slide.
- `N` = muestra el guion abajo (solo para practicar, **no** con el proyector conectado en espejo).
- `T` = cronómetro (`mm:ss / 07:00`, se pone rojo si van atrasados en esa slide). Parte solo al pasar a la slide 2.
- Slides 7 y 8 = **respaldo** para preguntas (no se presentan).
- Si el HTML falla: `NextLap_DemoDay_backup.pdf` (mismo contenido, estático).
- Funciona **sin internet** (fuentes y Three.js van dentro del archivo).

---

## 01 · Título — ANDY · 0:00–0:45

> Hola, somos Andy Villarroel y Agustín Reyes, grupo 8. Nuestro proyecto es NextLap.
> La pregunta es simple: cuando un piloto termina una vuelta, **¿cuánto va a tardar en la siguiente?**
> Predecimos ese tiempo en segundos, justo al final de la vuelta *t*, usando solo lo que ya se sabe en ese momento.
> Eso podría ayudar a un ingeniero de carrera a anticipar si el ritmo se cae o se recupera, y decidir cuándo reaccionar.
> Ojo: todo lo que vamos a mostrar es validación sobre carreras pasadas, no una prueba en vivo.

---

## 02 · Qué cambiamos — ANDY · 0:45–1:45

> El modelo base usa el promedio de las últimas vueltas. Pero el promedio no distingue a un piloto **regular** de uno que va **a saltos**.
> Por eso agregamos la **dispersión**: cuánto varían sus últimas hasta tres vueltas limpias, mismo piloto, carrera y stint, calculada hasta la vuelta *t* y nunca después. *(apuntar a la línea "FIN DE t")*
> Si hay menos de dos vueltas limpias, la marcamos con una bandera y usamos la mediana del entrenamiento.
> Para que sea justo: entrenamos con las carreras 1 a 7 y validamos en Austria y Gran Bretaña, 1.808 vueltas, **las mismas para todos los modelos**.
> [Antes de mirar resultados corrimos cuatro chequeos anti-fuga; por ejemplo, borramos las vueltas futuras en 700 filas y la variable no cambió ni una vez.]
> El control es el mismo modelo **sin** la dispersión: lo único distinto es el cambio. Le paso a Agustín.

---

## 03 · Resultado — AGUSTÍN · 1:45–3:25

> Gracias. Medimos el **MAE**: en promedio, por cuántos segundos fallamos la vuelta siguiente. Menos es mejor.
> Tres corridas, mismas 1.808 vueltas. La referencia del curso, regresión lineal con seis variables: **0,3867**. El control, que agrega solo la bandera: **0,3877** — o sea, la bandera sola no ayuda. Nuestro candidato, con la dispersión: **0,3697**.
> La resta que importa es **control menos candidato: +0,018 segundos**. Positivo significa que el candidato se equivoca menos, cerca de un 4,6 % menos.
> ¿Es suerte? Hicimos un bootstrap remuestreando piloto-carreras completas dos mil veces: el intervalo va de **0,009 a 0,028** y todas las réplicas dieron positivo.
> [Además se cumple la regla que escribimos en el Hito 1 antes de correr nada.]
> Siendo honestos: 18 milésimas es poco frente a un error típico de 0,37 segundos. Es una mejora real, pero **pequeña**.

---

## 04 · Dónde le cuesta — AGUSTÍN · 3:25–4:40

> Por carrera mejora en las dos: Austria **0,013** y Gran Bretaña **0,025**, casi el doble; Gran Bretaña es donde todos se equivocan más. Ponderando por vueltas recuperamos exactamente el 0,018 total.
> Pero la ganancia no es pareja: casi toda viene de **ventanas inestables**, donde el error baja **0,085** segundos. Con poco historial, empeora un poco.
> Un caso real *(apuntar al gráfico)*: piloto 23, Williams, Silverstone. Venía en 96 segundos y en la vuelta 7 hace **105**. El control se cree esa vuelta lenta y predice 102,1: falla por **4,2 segundos**. El candidato ve que la ventana está inestable y predice **99,1**. La vuelta real fue **97,9**: error de **1,24**.
> El límite: la dispersión **no sabe la dirección**. En la vuelta 12 del mismo piloto el ritmo lento se mantuvo y el candidato corrigió de más.

---

## 05 · SHAP — AGUSTÍN · 4:40–6:00

> ¿Qué movió esa predicción? Usamos **SHAP** sobre el candidato ya entrenado, en esa misma vuelta. SHAP reparte la predicción en aportes en segundos.
> Partimos del promedio de predicciones del entrenamiento: **86,73**. El promedio reciente suma **8,2** y la vuelta actual **6,7**, sobre todo porque Silverstone es un circuito largo. La dispersión **resta 2,47**: el modelo aprendió que una ventana muy inestable suele venir seguida de una vuelta más rápida que esa vuelta lenta. El resto casi no aporta.
> Chequeo: base más aportes da **99,121**, exactamente la predicción. Y ojo: esto explica la **predicción**, no el error; el error observado es aparte, 1,24 segundos.
> En global, sobre las 1.808 vueltas de validación y con fondo de entrenamiento, la dispersión es el **tercer input**, 0,15 segundos promedio en ambas carreras. Pero SHAP mide influencia; que mejore el error lo prueba la ablación. Le devuelvo a Andy.

---

## 06 · Decisión — ANDY · 6:00–7:00

> Entonces, ¿qué decisión defendemos? **Mantenemos la dispersión.** Con el mismo modelo, las mismas vueltas y el mismo control, se equivoca menos en las dos carreras, el intervalo no incluye cero y cumple la regla fijada antes de ver datos.
> ¿Hasta dónde aplica? Regresión lineal, vueltas limpias, Austria y Gran Bretaña; ayuda sobre todo cuando el ritmo viene irregular, y con historial corto no ayuda. No lo afirmamos para pits, safety car ni otros circuitos.
> [Probamos una versión sin bandera que dio un poco mejor, pero no la adoptamos: la elegimos después de ver validación, y eso sería ajustar sobre validación.]
> Siguiente chequeo: una **desviación con signo**, que sepa si la última vuelta fue más lenta o más rápida, con el criterio ya fijado. **Gracias, ¿preguntas?**

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

- [ ] Ensayar 1 vez con cronómetro (`T`). Meta: terminar la slide 6 antes de 7:00.
- [ ] Copiar `NextLap_DemoDay.html` **y** `NextLap_DemoDay_backup.pdf` a un pendrive + mandárselos a Agustín.
- [ ] Abrir el HTML en el computador del lab antes (13:00). Si va lento: el PDF.
- [ ] Llegar con el repo abierto en el notebook §5 / §8 por si piden ver el código.
- [ ] Anotar el feedback del jurado: lo necesitan para la sección 4 del informe final (domingo 22, 23:59).
