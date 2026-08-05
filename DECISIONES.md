# AscentPeak — Decisiones y estado del proyecto

> Documento de traspaso. Si una conversación se corta o empiezas una nueva,
> este archivo es la fuente de verdad. Léelo antes de proponer cambios.
>
> Última actualización: 2026-08-04

---

## 1. Qué es

Entrenador de fuerza personal para Jhair (Lima, Perú). PWA sin build, desplegada
en GitHub Pages, con Firebase para sincronización y la API de Claude para el
análisis semanal.

**La meta del proyecto no es registrar entrenamientos, es entrenar.** La app debe
convertirse en un entrenador autónomo que se actualiza con los datos, propone
mejoras, y se adapta tanto si la persona es constante como si no lo es.

- Repo: `Jh0708-cloude/AscentPeak`
- URL: `https://jh0708-cloude.github.io/AscentPeak/`
  **Las mayúsculas importan**: la ruta de GitHub Pages distingue mayúsculas de
  minúsculas. `/ascentpeak/` devuelve 404.
- Antecesores: `gymtrack` (v1, archivo histórico) y `gymtrack/V2` (pruebas del rediseño)

---

## 2. Contexto del usuario

- **Objetivo**: recomposición corporal. Métrica primaria: **cintura** (meta 85 cm
  al 31/10/2026, inicio 101 cm el 03/07). El peso corporal es de apoyo, no meta.
- **Aquiles en rehabilitación** — tendinopatía por lesión de vóley. Innegociable.
- Entrena 6 días/semana, sesiones de ~1h15. No es experto: espera que la app
  le diga qué le conviene, no solo que anote lo que hizo.
- Nutrición: 2553 kcal · 127P · 351C · 70G (revisar: se recomendó subir proteína
  a 1.8-2 g/kg).

---

## 3. Reglas del entrenador (innegociables)

Estas viven **en código**, validando la respuesta de la IA antes de aplicarla.
No se confían al prompt.

> **Dónde viven (desde 2026-07-27).** Las dos barandas duras están en `appProp()`,
> la función del botón ✓ Aplicar, y corren **antes** de escribir cualquier dato.
> Hasta esa fecha existían solo como texto dentro del prompt — o sea, como un
> pedido a la IA, no como un candado. Si se toca esa función, esto va primero.

### Protocolo Aquiles
- Ejercicios marcados sensibles (`aq >= 2`): **hack, búlgaras**. Nivel 1: prensa,
  gemelo en prensa, gemelo sentado. Nivel 3 (fuera del plan): elevaciones de
  talón en escalón — es el movimiento con más estiramiento profundo del tendón.
- Cualquier nota que mencione "Aquiles" o "tendón" **congela las subidas 14 días**
  en esos ejercicios, automáticamente.
- **Baranda en código**: con el congelamiento activo, una propuesta que suba peso
  en un ejercicio `aq >= 1` **no se aplica**. Avisa hasta qué fecha y deja la
  tarjeta pendiente. Las bajadas de peso sí pasan: el candado frena subidas, no
  correcciones a la baja.
- **El piso de RIR es 2, y desde el 02/08 es una baranda en código, no un consejo.**
  Registrar RIR 0 o 1 en cualquier ejercicio `aq >= 1` prende `log.aq` y **congela
  las subidas 14 días**, exactamente igual que una nota de molestia. La serie queda
  marcada `rf:1` y viaja al contexto como `[BAJO PISO AQ]`.
  Vive en `aqCheck()` y corre en las tres vías de registro: serie normal, serie
  corregida y serie extra.
  Antes de eso era un `toast()` de dos segundos, y solo en `aq >= 2` — o sea,
  prensa, gemelo en prensa y gemelo sentado no estaban cubiertos por nada.

### Progresión
- Doble progresión: al llegar al tope de reps dos sesiones seguidas, sube.
- **Máximos por semana: +5 kg en máquinas y compuestos, +2 kg en aislamientos.**
- **Baranda en código**: si la propuesta se pasa, se recorta al tope y se aplica
  recortada, avisando cuánto pedía la IA. El esquema de series (`sc`) se recorta
  con ella — no tendría sentido capar el peso y dejar entrar el número alto por
  la otra puerta. Queda anotado en el diario con el valor original.
  Aislamiento = `pat` que empieza en `iso_`, o `core`. Todo lo demás va con tope 5.
- Deload cada 6-8 semanas.

### Nutrición
- Si la cintura baja ≥0.5 cm por quincena → **no tocar el déficit.**
- Estancada 2 quincenas seguidas → recortar ~150 kcal de carbos. Nunca de
  proteína; la grasa solo hasta su piso (0.8 g/kg).

### Adaptación
- Si la frecuencia real baja de forma sostenida, **reestructurar el plan**
  (ej. PPL 6 días → torso/pierna 3 días), no insistir con lo que no se cumple.
  Una rutina que no se hace vale menos que una que sí.

---

## 4. Decisiones de producto

| Decisión | Razón |
|---|---|
| **Fuera la racha** | Premiaba la perfección y se rompía justo cuando la persona pasaba por algo. Sustituida por **carga por grupo muscular en 14 días**, que no se rompe y es lo que la IA usa para decidir. |
| **Fuera el bloque "Casa"** | Cumplimiento 0/3 semanas seguidas. Gemelos y abdomen se movieron **dentro del PPL**: si estás en el gym, ya estás en modo entreno. |
| **Nunca castigar el hábito fuerte** | Se descartó bloquear el gym por no hacer casa: habría creado un incentivo perverso. |
| **Un ejercicio a la vez** | El modo entreno usa pantalla completa, enfocado, con botón para alternar a lista. Es la pantalla más usada y se maneja con una mano. |
| **El descanso toma la pantalla** | Es el único momento en que se mira el celular sin hacer nada más. |
| **Historial congelado** | Cada día completado guarda su total planificado (`tp`). Cambiar la rutina **nunca** altera lo ya registrado. Bug aprendido a la mala. |
| **La IA propone, el humano decide** | Las propuestas bajan como tarjetas con ✓ Aplicar / ✗ Rechazar. Aplicar escribe el peso de verdad. Todo queda en el diario. |
| **Dos apps separadas** | AscentPeak y NutriTrack son productos independientes. NutriTrack espera a que AscentPeak esté maduro. |
| **La cinta manda, la foto acompaña** | La cintura es la métrica; el espejo varía por sal, hinchazón y congestión post-entreno. Las fotos sirven como respaldo **mensual**, en condiciones fijas: en ayunas, misma pared y luz, de frente y de perfil, relajado. Semana a semana no muestran nada y solo frustran. |
| **Series directas y de ayuda, separadas** | El contador sumaba la serie completa al músculo primario y media a cada secundario, pero se comparaba contra rangos pensados para trabajo **directo**. Bíceps marcaba 35 cuando de directo tenía 18: la mitad eran jalones. Ahora la barra separa los dos tramos y el semáforo mira solo el directo. |
| **Corregir se gana, no se desbloquea** | El mis-click se arregla con un gesto que no se dispara solo: 600 ms con barra que se llena, cancelable soltando. Se descartó el PIN: es tu teléfono y tus datos, no protege de nadie, y escribirlo con las manos sudadas en medio de una serie es peor que el problema que resuelve. |
| **Lo planificado y lo ejecutado son dos números** | `prog` cuenta series del plan (barra de avance, `tp`, día completo). `sd` guarda lo que pasó de verdad, extras incluidas. La serie extra **no toca `prog`**: si lo hiciera, cuatro extras en el primer ejercicio darían el día por terminado con la mitad de la sesión sin hacer. |
| **El estado nunca se dice con color** | Bajo/óptimo/alto se marca con flecha ↓ ↑ y con la banda del rango detrás de la barra. Los colores quedan reservados para el patrón. Se intentó primero pintar el número y reapareció el mismo problema: "óptimo" salía lavanda y "alto" durazno, robándole el significado a la marca y al empuje. |

---

## 5. Sistema visual

- **Base**: carbón con tinte ciruela. `--bg:#17141D` · `--s1:#211C2B` ·
  `--s2:#2A2436` · `--s3:#352E44` · `--line:#403852`
- **Morado lavanda** `#B39DE8` — acción, pestaña activa, marca. Exclusivo: nunca
  significa otra cosa.
- **Cuatro colores por patrón**, no nueve por músculo:
  empuje `#F7B489` (durazno) · tracción `#8FC0F5` · pierna `#7BE3AE` · core `#FFD98A`
  Cada grupo muscular hereda el color de su patrón (`GCOL`): pecho, hombro y
  tríceps van con empuje; espalda y bíceps con tracción.
- **Ningún color se usa para dos cosas.** Si algo necesita comunicar estado o
  intensidad, se resuelve con forma, posición u opacidad — flechas, bandas,
  transparencia — nunca tomando prestado un color que ya tiene dueño.
- Bebas Neue solo para números y títulos cortos. Inter para el resto.
- Inspiración: amanecer sobre una cima (lavanda → durazno).

---

## 6. Modelo de datos

### Catálogo (`g2_cat` / Firestore `data/cat2`)
```
{ id, n, pat, pri, sec[], eq, aq, unit, w:{v, ps, nt, bw, pl}, rest }
```
`pat` = patrón de movimiento · `pri`/`sec` = músculo primario y secundarios ·
`eq` = equipo · `aq` = nivel de impacto en Aquiles (0-3) ·
`w.v` = kg numérico · `w.ps` = por lado · `w.bw` = corporal ·
`unit` = unidad de medida: ausente o `rep` (repeticiones, por defecto) o `seg`
(isométricos). Ver §12.

**Los pesos son numéricos y tipados**, no texto libre. Sin esto, la IA no puede
escribir en los datos con seguridad.

**`CAT_DEF` es semilla, no fuente.** El catálogo real vive en `localStorage`
(`g2_cat`) y en Firestore (`data/cat2`); `CAT_DEF` solo se usa la primera vez.
Por eso existe **`migCat()`** (04/08): al arrancar y después de cada sync copia
los metadatos nuevos de `CAT_DEF` al catálogo guardado y lo vuelve a subir. Sin
esa función, cualquier campo agregado a `CAT_DEF` nunca llega al dispositivo —
fue exactamente lo que pasó con `unit` el 03/08.

### Plan (`g2_plan` / Firestore `data/plan`)
```
{ v, since, note, days: { 1..6: { l, it:[{e, s, r, sc[]}] }, 0: descanso } }
```
`sc` = esquema de pesos por serie (ej. `[30,30,30,25]`) · versionado con historial.

### Registro (`gt_log` / Firestore `log/{fecha}`)
```
{ day, prog:{exId:n}, sd:{exId:[{r, w, rir, rf}]}, notes:[], tp, aq, light, t0, t1 }
```
`sd` = series reales con reps, peso y RIR · `tp` = total congelado ·
`aq` = bandera de molestia en Aquiles

### Otros
`gt_weight_hist` (historial de cargas) · `gt_measures` (medidas) ·
`g2_ai` (análisis) · `g2_diary` (propuestas aplicadas/rechazadas) ·
`g2_key` (API key — **solo local, nunca sube a Firestore ni al repo**)

En `g2_diary`, el campo `cap` guarda el valor original cuando la baranda recortó
una propuesta. Si es `null`, la IA se mantuvo dentro del tope.

### Marcas en las series (`sd`)
`xt:1` = serie extra, fuera del plan · `ed:1` = serie corregida a mano ·
`rf:1` = rompió el piso de RIR 2 en un ejercicio del protocolo Aquiles.
Ambas viajan al contexto de la IA como `[extra]` y `[corregida]`: una serie
editada después no es lo mismo que una medida en el momento, y el entrenador
tiene que poder distinguirlas. Toda corrección deja además su rastro en `notes`
con la hora y el valor anterior.

### Carga por grupo (`loadByGroup`)
Cuenta **desde `sd`** (series realmente registradas), con `prog` como respaldo
para logs viejos sin series detalladas. Contaba desde `prog` hasta el 31/07, y
por eso las series extra no existían para el contador.
Devuelve `{ grupo: {d, i} }` — `d` = series directas (músculo primario),
`i` = series de ayuda (secundarios, a media serie cada uno). `RANGO` se compara
**solo contra `d`**, porque esos topes son de trabajo directo. El contexto que
recibe la IA también lleva los dos números separados.

---

## 7. Plan de entrenamiento v3 (desde 2026-07-27)

| Día | Sesión | Abdomen / Gemelos |
|---|---|---|
| Lunes | Push A | Plancha con peso (anti-extensión) |
| Martes | Pull A | Crunch en polea arrodillado |
| Miércoles | Pierna A | Gemelo en prensa + gemelo sentado |
| Jueves | Push B | Elevación de piernas en máquina |
| Viernes | Pull B | Pallof press (anti-rotación) |
| Sábado | Pierna B | Gemelo en prensa |
| Domingo | Descanso | — |

Abdomen 4×/semana con estímulos distintos, gemelos 2×. Sin flexiones laterales
cargadas: engrosan oblicuos y la meta es reducir cintura.

---

## 8. RIR (repeticiones en reserva)

Al terminar la última serie de cada ejercicio, la app pregunta cuánto quedaba
en el tanque. **El orden en pantalla es 3+ · 2 · 1 · 0**, sin valor
preseleccionado.

El orden importa y no es estético. Estaba al revés (0 primero) y el 0 caía justo
donde apoya el pulgar: 26 de los primeros 36 registros salieron en 0. Con esos
datos no se puede distinguir "fui al fallo" de "toqué el botón más cercano", y
la IA estaba decidiendo cargas sobre eso. Lo severo va lejos del pulgar.

Lectura para el entrenador:
- RIR 3+ con tope de reps → el peso quedó corto, subir
- RIR 0-1 repetido → al límite, mantener o bajar
- RIR subiendo con el mismo peso → se adaptó, toca progresar
- En ejercicios del protocolo Aquiles, piso RIR 2

---

## 9. Roadmap

### Hecho
- Fase 1 — app base (calendario, contadores, cronómetro, medidas, índices)
- Paso A — datos ricos (reps reales por serie, pesos por serie, notas, sesión ligera)
- Fase 2 — Firebase (Auth con Google, Firestore São Paulo, reglas, sync offline-first)
- Paso 1 y 2 — catálogo con metadatos + pesos tipados + plan versionado
- Paso 3 — loop cerrado: la IA devuelve propuestas aplicables, no texto muerto
- Rediseño v2 — 4 pestañas, modo entreno enfocado, mapa de calor, carga por grupo
- RIR
- Rebautizo a AscentPeak con paleta propia
- **Barandas en código** (27/07) — tope de subida y bloqueo por Aquiles dentro de
  `appProp`, con el recorte anotado en el diario
- **Corregir series + series extra** (31/07) — pulsación larga de 600 ms sobre una
  serie ya hecha abre la hoja de corrección (reps, peso, RIR, eliminar). El botón
  `+ Serie extra` registra la serie fuera del plan. `loadByGroup` pasó a contar
  desde `sd`, no desde `prog`
- **Ejercicios por tiempo** (02/08) — campo `unit:'seg'` en el catálogo. Cambia la
  pregunta, la grilla pasa a saltos de 5 s, el registro muestra `45s`, y los
  isométricos salen del cálculo de tonelaje. Hoy solo aplica a Plancha con peso (id 35)
- **Piso de RIR con candado** (02/08) — `aqCheck()`, ver §3
- **Orden del selector de RIR** (02/08) — ver §8
- **Series legacy marcadas** (02/08) — `legW()`: las series con peso guardado como
  texto viajan al contexto como `[legacy]` y el prompt le prohíbe a la IA usarlas
  para decidir progresión. Sin migrar nada
- **`migCat()`** (04/08) — ver §6. El fix de `unit` del 02/08 se subió pero nunca
  llegó al catálogo del teléfono; se detectó comparando el respaldo del 04/08
- **Lo real vs. lo planificado en el contexto** (04/08) — `realMap()` agrega
  `[REAL: X kg el DD-MM]` a la línea del catálogo y `[REAL: X series]` a la del
  plan cuando difieren. El catálogo y el plan solo se actualizan al aplicar una
  propuesta, así que se quedaban viejos cuando Jhair progresaba solo.
  Lleva además el **máximo reciente** (21 días) cuando supera al último dato:
  `[REAL: 40 kg el 31-07 · máx 60 el 26-07]`. Sin eso, un día flojo borra el techo
  y la IA felicita una subida que en realidad es un retroceso — pasó con el hip
  thrust (pico 60, actual 40) y con la prensa (pico 80, actual 75)
- **Doble progresión calculada en código** (04/08) — `dblProg()` entrega la
  conclusión ya hecha en un bloque propio. Antes la regla vivía solo en el prompt
  y la IA tenía que deducirla de listas de reps crudas — no lo hacía.
  **Corregida el mismo día**: la primera versión contaba solo las repeticiones e
  ignoraba a qué peso se hicieron. Marcaba 12 ejercicios; de esos, jalón al pecho
  (50 → 60 kg) ya había progresado y prensa (80 → 75 kg) había *retrocedido*, y a
  ambos les proponía subir. Contando solo sesiones consecutivas **al mismo peso
  numérico**, quedan 5 legítimos. Cambiar el peso corta la racha, porque cambiar
  el peso ya *es* la progresión
- **`rngFor()`** (04/08) — el rango de reps se busca según el día de la sesión.
  Press militar es 6-8 en Push A y 8-10 en Push B; la versión anterior se quedaba
  con el último rango que encontraba y evaluaba los dos días contra 8-10
- **Semana en curso marcada como parcial** (02/08) — el bloque de cumplimiento
  avisa cuándo la última semana está sin terminar. Antes la IA leía 5 sesiones
  como 4 y lo reportaba como incumplimiento

### Pendiente
> **Orden acordado el 02/08.** De más urgente a menos. El criterio no es el
> tamaño del cambio sino **qué tan reversible es**: lo que solo cambia lectura o
> presentación va junto; lo que cambia la forma del dato guardado va solo, con
> una sesión de gimnasio de por medio para probarlo.

**1. Paso 5 — desacoplar la rutina del día de la semana.** Subió al primer lugar
el 02/08. `dayKeyOf` cae al calendario (`getDay()`) y `doShift` solo escribe el
día corrido **dentro de la semana actual**: el lunes arranca semana nueva y el
ciclo se resetea a Push A, comiéndose lo que quedó pendiente. Como el corte
siempre cae al final, **se pierden siempre los días 5 y 6**. Medido sobre
06/07-02/08: días 1-4 hechos 4 veces cada uno, días 5 y 6 solo 3.
Pierna B es donde viven prensa a 4 series, búlgaras y gemelo en prensa — parte de
la explicación del isquios en 9 series y del gemelo estancado.
El ciclo no debe preguntar "¿qué día es hoy?" sino **"¿cuál fue la última sesión
que terminé?"** y ofrecer la siguiente (1→2→3→4→5→6→1). El calendario decide *si*
se entrena, no *qué*. Con vigilante: si un día del ciclo se atrasa más de 9-10
días, avisar — ahí sí toca reestructurar (la parte original del Paso 5).
Toca `dayKeyOf`, `shiftWeek`, `weekOf`, `rEntrenar` y `buildCtx`. Es el cambio
más grande de la lista.

**2. Paso 4 — sustitución de ejercicios** (detalle abajo). Necesita la marca
`sub:{id}` en la serie para que el peso del reemplazo no entre al histórico del
original.

**3. Drop sets.** La extensión de piernas se registra como 3×15 a 77 kg,
idéntico cinco sesiones seguidas, cuando en realidad es una serie descendente
(77/68/54/36/23). La IA lo lee como estancamiento perfecto y en algún momento va
a proponer subir de 77 — o sea, subir el escalón más alto de un drop set.
Campo `dr:[68,54,36,23]` dentro de la serie; el array de `sd` ya lo admite sin
romper nada.

**4. Escritura del progreso al catálogo y al plan.** Lo del 04/08 resolvió la
**lectura** (la IA ya ve lo real), no la **escritura**. `x.w.v` y el `sc` del plan
siguen cambiando solo con ✓ Aplicar. Consecuencias vivas: la pantalla de entreno
precarga el peso viejo (`setW` mira `it.sc` o `x.w.v`, nunca la última sesión) y
el tope de +5 kg de la baranda se calcula desde un número desactualizado.
Necesita reglas antes de escribir: no contar sustitutos, ni series `legacy`, ni
isométricos. Va después de `sub` porque depende de esa marca.

**5. Isquios y pecho fuera de rango** (plan, no código). Al 02/08: isquios 9
series directas en 14 días contra un piso de 10, con un solo ejercicio (curl
femoral); pecho 41 contra un techo de 40; hombro 38 contra 36. Falta peso muerto
rumano o buenos días en Pierna B. Va en el plan v4.

**6. Peso corporal sin registrar.** Dos entradas, ambas de la primera semana de
julio. Sin ese dato no se puede distinguir grasa perdida de músculo perdido.
No es código: es el hábito de medirlo.
- **Registro de cardio — diseñado, no construido (27/07).** Se decidió no armarlo
  todavía: el Aquiles está en rehabilitación y no hay carrera a la vista. Una
  sección vacía en Actividad no sería una función, sería un recordatorio de algo
  que no se puede hacer. El diseño queda cerrado para cuando llegue el momento:

  - **Dónde**: dentro de Progreso → Actividad. No una pestaña nueva. Cada
    elemento agregado cobra alquiler para siempre.
  - **Registro**: `{ id, d, t, km, min, src, note }` · `t` = correr / caminar /
    bici / otro.
  - **`src` (manual · gps · reloj)** — lo que permite que lo escrito a mano y lo
    capturado convivan sin migrar nada, y saber siempre qué se midió y qué se
    estimó.
  - **`id` único** — evita duplicados el día que se importe de un reloj, y es de
    donde colgará la ruta cuando exista el rastreador.
  - El ritmo (min/km) **se calcula, no se guarda.** Guardar lo que se puede
    deducir es cómo aparecen los datos que se contradicen entre sí.
  - Sincroniza como una colección más, igual que las medidas.
  - El entrenador recibe km y minutos de los últimos 14 días: explica por qué
    las piernas rindieron menos una semana.
  - Fuera de alcance del formulario: ruta, mapa, ritmo por kilómetro, calorías.
    Eso pertenece al rastreador GPS, no al registro manual.
  - Candidato a evaluar: esfuerzo percibido 1-10. Es el dato que más le sirve al
    entrenador y el único que ningún GPS puede medir.
  - **Orden**: manual primero, rastreador GPS propio dentro del APK después,
    relojes mucho más adelante. Health Connect queda como atajo si el plugin
    resulta simple — leer es mucho más barato que rastrear.
- **Paso 4 — sustitución de ejercicios.** "No puedo hacer este hoy" → la app
  propone el equivalente correcto por patrón y músculo, respetando el filtro
  del Aquiles. La equivalencia no es solo "mismo músculo": difieren en rango,
  estabilidad y carga articular.

  **Caso real (27/07)**: bancas ocupadas → press militar con mancuerna (22.5 por
  lado) sustituido por máquina de hombro de disco (15 por lado + brazo, y aun así
  costó más). Dos reglas que salieron de ahí:
  - **El peso del sustituto NO se registra en el histórico del original.** Son
    ejercicios distintos: palanca, curva de resistencia y estabilización cambian.
    Meterlo en la misma serie histórica le muestra al entrenador un salto que
    nunca pasó, y desde ahí propone seguir subiendo.
  - En máquinas de disco el peso incluye el brazo, que no se conoce. El número
    sirve **solo para compararse consigo mismo** en esa máquina; nunca contra una
    barra o mancuerna. Registrar como "X kg por lado + brazo".
  **Implementación acordada (02/08)**: marca `sub:{id}` en la serie. La serie se
  guarda en el log del día y se ve en el historial, pero **no entra al histórico
  de progresión** del ejercicio original.
  Mientras no exista el Paso 4, la sustitución va en las notas del día.
- **Paso 6 — autonomía graduada.** Dial en Ajustes: propone → aplica cargas →
  cambia ejercicios → rediseña el plan. Siempre con barandas en código y
  botón de revertir.
- Fase 3 — Capacitor (APK Android): vibración en segundo plano y notificaciones.
  Es la queja original que sigue viva.
- NutriTrack v2 — congelada hasta que la API le quite la fricción de registrar.
- Finanzas 2.0 — sin fecha.

---

## 10. Infraestructura

- **Firebase**: proyecto `gymtrack-897b5`, Firestore en São Paulo, Auth con Google.
  Reglas: `users/{uid}/{document=**}` solo si `request.auth.uid == uid`.
  Dominio autorizado: `jh0708-cloude.github.io`.
- **API de Claude**: modelo `claude-sonnet-4-6`, llamada directa desde el
  navegador. ~$0.04-0.06 por análisis, menos de $0.30 al mes.
- **Despliegue**: GitHub Pages, branch `main`, carpeta root. Ritual: respaldo →
  subir archivos → commit → esperar el ✅ en Actions → doble cierre/apertura de
  la app (service worker). **Subir el sw con la versión de caché bumpeada.**
  Caché actual: `ascentpeak-v11` (subida el 04/08).

---

## 11. Lecciones aprendidas

- **Validar siempre el JS antes de empaquetar** (`node --check`). Un error de
  sintaxis deja la app en pantalla negra.
- **Nada de cambios retroactivos.** Los ajustes de rutina son de hoy hacia
  adelante. Una migración mató la racha una vez.
- **El entorno de trabajo se reinicia.** Si algo no está en GitHub, puede
  perderse. Subir apenas se entrega.
- **Una regla escrita en el prompt no es una baranda.** Es un pedido. Si la regla
  importa de verdad, tiene que correr en código antes de escribir el dato. La
  diferencia es pedirle al cajero que no cobre de más versus que la caja no lo
  permita.
- **Un número solo significa algo si se compara con su misma unidad.** El
  contador de carga mezclaba trabajo directo con indirecto y lo medía contra
  rangos de trabajo directo: casi todo salía "alto" y el panel dejó de informar.
- **Las rutas de GitHub Pages distinguen mayúsculas.** El repo es `AscentPeak`;
  `/ascentpeak/` da 404 aunque el repo exista y el deploy esté en verde.
- **La app está afilada para una sola persona.** Que alguien de fuera no la
  entienda a la primera no es un defecto: es la consecuencia de estar hecha a
  medida. La prueba que vale es si Jhair se pierde, no si se pierde un tercero.
  Hacerla entendible para cualquiera la volvería más genérica y peor para su uso.
- **Nada se construye "por si acaso".** Una función que no se va a usar en meses
  ocupa lugar, confunde y envejece. El diseño se deja escrito —que es la parte
  cara— y se construye el día que haga falta. Mismo criterio con el que salió el
  bloque "Casa".
- **Un candado que estorba se termina esquivando.** La corrección se pensó primero
  con clave. No protege nada — el teléfono ya es de una sola persona — y el costo
  se paga cada vez, en medio del entreno. El gesto largo cuesta 600 ms y solo la
  primera vez.
- **Antes de contar, mirar de dónde sale el número.** La serie extra parecía un
  cambio de pantalla y era del contador: `loadByGroup` leía `prog`, que es el
  plan. Cualquier cosa que pase fuera del plan era invisible para el panel y para
  la IA, sin avisar.
- **Un aviso no es una baranda, aunque esté en la pantalla.** Corolario de la
  lección de arriba, aprendido el 02/08: el piso de RIR había salido del prompt
  y entrado al código… como un `toast()` de dos segundos que igual guardaba el
  dato. Sacar la regla del prompt no alcanza si lo que queda es el mismo pedido
  con otra cara. La prueba es simple: **si el usuario puede seguir de largo, no
  es una baranda.**
- **El orden de los botones es parte del dato.** Ver §8. Cuando un valor se
  captura entre serie y serie, sudado y sin mirar, la posición decide el
  resultado tanto como la intención.
- **Antes de programar un fix, mirar qué día cae en el calendario.** El fix del
  Aquiles se subió un domingo pensando probarlo el lunes; el lunes era Push A y
  no tiene un solo ejercicio sensible. Un cambio que no se puede probar hasta
  dentro de tres días es un cambio a medio subir.
- **La app guardaba, pero no leía.** Los tres huecos del 04/08 —peso, series y
  repeticiones— eran el mismo: el dato quedaba bien registrado en el log y nunca
  se comparaba con lo planificado. Registrar no es entrenar. Cada número que se
  guarda tiene que tener alguien que lo lea y saque una conclusión, o es archivo
  muerto que además da falsa sensación de control.
- **Una regla que no se calcula no existe, aunque esté en el prompt.** La doble
  progresión llevaba semanas escrita en `SYS` y ningún ejercicio subió por ella:
  la IA tenía que deducirla mirando listas de reps sesión por sesión. La versión
  corregida le entrega el resultado ya calculado. Segunda variante de la lección
  de las barandas: **el prompt sirve para criterio, no para aritmética.**
- **Lo que se despliega no es lo que corre.** El fix de `unit` se subió a GitHub,
  el commit salió verde y el código estaba correcto — y aun así nunca se ejecutó,
  porque el catálogo se carga de `localStorage` y `CAT_DEF` es solo semilla.
  **Verificar contra el respaldo, no contra el repo ni contra la pantalla.**
- **`node --check` valida la sintaxis, no la aritmética.** `dblProg()` se subió
  sintácticamente perfecta y contaba mal: le proponía subir carga a un ejercicio
  donde Jhair acababa de bajar el peso. El error no lo encontró ninguna revisión
  de código sino **ejecutar la función contra el respaldo real en Node** e imprimir
  la salida. Desde el 04/08, toda función que calcule algo se prueba así antes de
  subirse: extraerla, correrla con los datos de verdad, leer el resultado.
  El riesgo peligroso no es la pantalla negra —esa se ve en dos segundos— sino el
  código correcto que calcula mal en silencio.
- **La sospecha del usuario vale más que la lectura del código.** Los tres huecos
  del 04/08 salieron de "me parece que no lo reconoce", no de una auditoría. Quien
  usa la app todos los días detecta el desfase antes de poder explicarlo.
- **La conversación se comprime.** Por eso existe este archivo.

---

## 12. Auditoría de datos (2026-08-02)

Primera revisión completa del respaldo (22 sesiones, 06/07 al 02/08). Lo que
salió, para no volver a descubrirlo.

### Lo que va bien
- **Cintura**: 101 → 100 → 98.8 → **98**. Quincena 1: −2.2 cm. Quincena 2: −0.8.
  Ambas sobre el umbral de 0.5 → **no tocar el déficit**.
  ICC 0.96 → 0.94 · ICA 0.594 → 0.576.
- **Recomposición confirmada**: brazo 37 → 37.5, pierna 58.1 → 59, mientras la
  cintura baja 3 cm.
- **Adherencia**: 22 sesiones en 28 días (6/6, 6/6, 5/6, 5/6).

### La meta de cintura hay que corregirla
De 98 a 85 son 13 cm en 13 semanas: 1.0 cm/semana. El ritmo real de la última
quincena es **0.4 cm/semana**. Proyectado al 31/10 da **92-93 cm**, no 85.
0.4 cm/semana es el ritmo sostenible y el que preserva el músculo que está
ganando. **Meta ajustada: ~92 cm al 31/10.** Los 85 quedan para marzo.

### Datos que mienten (y por qué)
| Qué | Síntoma | Estado |
|---|---|---|
| Plancha | 47/41/45 guardados como reps; 0.4 t falsas de tonelaje el 27/07 | ✅ resuelto (`unit`) |
| RIR | 26 de 36 en 0 por posición del botón | ✅ resuelto (§8) |
| Press militar | 22.5 → **15** → 22.5; el 15 era la máquina sustituta | ⏳ pendiente (Paso 4) |
| Extensión de piernas | 3×15 a 77 kg idéntico ×5; es un drop set | ⏳ pendiente (`dr`) |
| Pesos en texto | 217 series con `w` tipo string ("30 kg x lado", "Corporal" → 0 t) | ✅ marcadas `[legacy]` |
| Cumplimiento | la semana en curso se leía como incumplimiento | ✅ resuelto |

Corte limpio: **todo lo anterior al 26/07 es texto, todo lo posterior es
numérico.** Por eso `legW()` mira el tipo del dato y no una fecha mágica — el
dato se describe solo.

### El día que lo detonó todo
**31/07, Pierna A**: hack RIR 1, prensa RIR 0, gemelo sentado RIR 0, gemelo en
prensa RIR 0. Cuatro violaciones del piso en una sesión, cero frenos.
El 02/08 la IA propuso subir la prensa de 65 a 70 kg **citando ese RIR 0 como
justificación**, y `appProp` la dejó pasar porque `aqFrozen()` devolvía null.
El sistema usó una violación de la regla como argumento para subir carga en el
ejercicio protegido.

Simulado contra el historial completo, el nuevo `aqCheck()` habría saltado
**exactamente una vez**: ese día.

### Segunda tanda de hallazgos (04/08)
Los tres salieron de una sospecha de Jhair, no de revisar código. Mismo origen:
**el catálogo y el plan solo se actualizan al aplicar una propuesta.**

| Qué | Medido | Estado |
|---|---|---|
| Peso real > catálogo | 9 ejercicios desfasados (fondos 52→60, jalón 55→60, búlgaras 15→20, prensa 70→75, hip thrust 35→40…) | ✅ `[REAL: X kg]` |
| Series reales ≠ plan | 8 sesiones. Press militar 4 series en vez de 3 **las 4 veces**; remo sentado 3 en vez de 4 | ✅ `[REAL: X series]` |
| Tope de reps sin subir carga | 12 ejercicios en la primera medición; **5 reales** al exigir mismo peso | ✅ `dblProg()` corregida |

La marca `[REAL]` funciona en las dos direcciones: gemelo en prensa figura
`40 kg [REAL: 30 kg]`, que es el estancamiento de §12 visto desde el otro lado.

Los 5 legítimos al 04/08: aperturas (17.5 kg), cruce en polea alta (20), pájaros
(6), remo sentado en polea (50) y extensión de piernas (77).

**Trampa conocida**: extensión de piernas cumple la condición de doble progresión
de forma perfecta (15/15/15 ×5 a 77 kg) porque es un drop set mal registrado.
Hasta que exista `dr`, esa propuesta hay que rechazarla a mano.

### Gemelo estancado
Gemelo en prensa: 40 kg×lado (23/07) → 30 (26/07) → 30 (31/07). Son 60 kg
totales contra un benchmark de ~126 kg (1.5× peso corporal) para volver a
pliometría y vóley. Cinco semanas sin moverse, y con RIR 0 a 30 kg. Subir reps
antes que carga.
