# AscentPeak — Decisiones y estado del proyecto

> Documento de traspaso. Si una conversación se corta o empiezas una nueva,
> este archivo es la fuente de verdad. Léelo antes de proponer cambios.
>
> Última actualización: 2026-07-30

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
- En estos ejercicios el piso de RIR es 2. Si reporta 0-1, el entrenador corrige.

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
{ id, n, pat, pri, sec[], eq, aq, w:{v, ps, nt, bw, pl}, rest }
```
`pat` = patrón de movimiento · `pri`/`sec` = músculo primario y secundarios ·
`eq` = equipo · `aq` = nivel de impacto en Aquiles (0-3) ·
`w.v` = kg numérico · `w.ps` = por lado · `w.bw` = corporal

**Los pesos son numéricos y tipados**, no texto libre. Sin esto, la IA no puede
escribir en los datos con seguridad.

### Plan (`g2_plan` / Firestore `data/plan`)
```
{ v, since, note, days: { 1..6: { l, it:[{e, s, r, sc[]}] }, 0: descanso } }
```
`sc` = esquema de pesos por serie (ej. `[30,30,30,25]`) · versionado con historial.

### Registro (`gt_log` / Firestore `log/{fecha}`)
```
{ day, prog:{exId:n}, sd:{exId:[{r, w, rir}]}, notes:[], tp, aq, light, t0, t1 }
```
`sd` = series reales con reps, peso y RIR · `tp` = total congelado ·
`aq` = bandera de molestia en Aquiles

### Otros
`gt_weight_hist` (historial de cargas) · `gt_measures` (medidas) ·
`g2_ai` (análisis) · `g2_diary` (propuestas aplicadas/rechazadas) ·
`g2_key` (API key — **solo local, nunca sube a Firestore ni al repo**)

En `g2_diary`, el campo `cap` guarda el valor original cuando la baranda recortó
una propuesta. Si es `null`, la IA se mantuvo dentro del tope.

### Carga por grupo (`loadByGroup`)
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
en el tanque: **0** (al fallo) · **1** (casi) · **2** (duro) · **3+** (cómodo).

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

### Pendiente
- **Ejercicios por tiempo (isométricos).** El plan v3 metió plancha y pallof, que
  se miden en segundos, pero el catálogo solo entiende repeticiones. Hoy la
  plancha muestra "objetivo 30-45s reps" y pregunta "¿cuántas hiciste?" con una
  grilla de números: funciona de casualidad porque el usuario elige segundos,
  pero la app no sabe qué está guardando. Falta un tipo `iso` en el catálogo que
  cambie la unidad a segundos y el texto de la pregunta. **Y afecta el conteo**:
  una plancha de 40s suma igual que una serie de crunch, y no son equivalentes.
  Mientras no se arregle, la carga de Core está mal medida.
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
- **Series extra.** Registrar la serie de más que a veces sale (`3+1`) en vez de
  perderla. Cambio chico y alimenta directo el análisis.
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
  Mientras no exista el Paso 4, la sustitución va en las notas del día.
- **Paso 5 — adaptación a la frecuencia real.** Reestructurar el plan cuando
  la constancia baja.
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
  Caché actual: `ascentpeak-v4`.

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
- **La conversación se comprime.** Por eso existe este archivo.
