# SKILL — Generador de guías interactivas de entrenamiento HTML

## Versión 2.3 — Abril 2026

Cambios respecto a v2.2 (consolidación de directrices derivadas de sesiones reales):
- Nuevo tipo de pantalla #8 "Información intermedia" para notificar omisiones u otras notas puntuales durante el flujo (sin cronómetro, con botón "Continuar"). Formaliza el tipo `info` que hasta ahora se implementaba ad-hoc en cada HTML con adaptaciones.
- Nuevo componente "Banner de adaptaciones en la intro" para sesiones con desviaciones respecto al protocolo estándar (episodios abiertos, omisiones). Incluye lista de adaptaciones activas y lista visual de NOs.
- Heurística de mapeo bloque → color de anillo del cronómetro (calentamiento/movilidad → rosa; trabajo específico → naranja; descanso → verde).
- Pantalla de transición de bloque: se añade shortBeep en cada segundo de la cuenta atrás 3-2-1 (antes solo longBeep al inicio), coherente con el patrón general de cuentas atrás del resto de pantallas.
- Pantalla final: filas opcionales "Bloques completados" y "Omisiones del día" cuando la sesión tiene adaptaciones.
- Exercise-timed: aclaración de que el botón "Terminar antes" es estándar en todos los timed (no solo openEnded).

Cambios de v2.1 → v2.2:
- El detalle del ejercicio pasa de panel expandible (botón "Ver técnica" + toast) a caja fija visible permanentemente entre el nombre del ejercicio y el cronómetro/contador.

Cambios de v2.0 → v2.1:
- Hora estimada de finalización visible junto a la barra de progreso global

Cambios de v1.0 → v2.0:
- Diferenciación entre tiempos de preparación intra-bloque e inter-bloque
- Detalle del ejercicio visible durante la ejecución
- Indicador visual de bloque actual y progreso intra-bloque
- Pantalla de transición de 3 segundos entre cambios de bloque

---

## Propósito

Generar archivos HTML autocontenidos que funcionan como guías de entrenamiento interactivas en móvil. El usuario abre el archivo en el navegador, pulsa "Comenzar", y el sistema le guía paso a paso por toda la sesión: preparación, ejecución, descansos y vuelta a la calma, con cronómetros, señales sonoras y pantalla siempre encendida.

## Contexto de uso

- Dispositivo principal: móvil (Pixel 9a, Vivaldi)
- Hosting: GitHub Pages (HTTPS obligatorio para Wake Lock API)
- Repositorio: `sndalx/entrenaLXdor`
- Estructura de archivos: HTML en raíz, imágenes en carpeta `img/`
- El usuario no tiene conocimientos técnicos de desarrollo web
- El archivo debe ser 100% autocontenido (un solo .html, sin dependencias externas salvo fuentes de Google Fonts)

---

## Arquitectura del HTML

### Estructura general

```
<!DOCTYPE html>
<html lang="es">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0, user-scalable=no">
  <title>[Nombre de la sesión]</title>
  <style>/* Estilos inline */</style>
</head>
<body>
  <div id="app"></div>
  <script>/* Toda la lógica */</script>
</body>
</html>
```

### Principio de pantalla única

La app muestra UNA sola pantalla a la vez. Cada pantalla ocupa el 100% del viewport (`100dvh`). La transición entre pantallas es instantánea (reemplazo de `innerHTML`). No hay scroll, no hay navegación, no hay menús. El usuario solo interactúa con el botón principal de cada pantalla.

### Tipos de pantalla

1. **Intro** — Nombre de la sesión, descripción, botón "Comenzar sesión". Cuando la sesión tiene adaptaciones respecto al protocolo estándar (episodios abiertos, omisiones), incluye además el componente "Banner de adaptaciones" (ver §Componentes).
2. **Preparación** — Nombre del ejercicio, instrucciones, equipamiento necesario, cuenta atrás de preparación. Botón "Saltar"
3. **Ejercicio con repeticiones** — Nombre, número de reps, tempo si aplica, indicación "por lado" si aplica. Caja de detalle visible entre nombre y contador. Botón "Hecho"
4. **Ejercicio cronometrado** — Nombre, cronómetro circular animado (cuenta atrás o cuenta arriba para open-ended). Caja de detalle visible entre nombre y cronómetro. Botón "Terminar antes" (estándar en todos los timed, permite salir del ejercicio antes del tiempo total). En ejercicios open-ended el botón se llama "Soltar".
5. **Descanso** — Cronómetro circular, indicación del siguiente ejercicio. Botón "Saltar descanso"
6. **Final** — Resumen con estadísticas, mensaje del siguiente día del plan
7. **Transición de bloque** — Pantalla cognitiva de 3 segundos que avisa del cambio entre bloques. Muestra "Comenzando: [nombre del nuevo bloque]" con cuenta atrás visual 3 → 2 → 1. Sin cronómetro interactivo ni botón. Sonidos: `longBeep` al iniciar, `shortBeep` en cada segundo de la cuenta atrás (3, 2, 1) para mantener el patrón sonoro general del resto de pantallas. No se cuenta como ejercicio en el progreso global. Se inserta automáticamente cuando el campo `block` cambia entre dos steps consecutivos (ver §Modelo de datos y §Flujo de pantallas).
8. **Información intermedia** — Pantalla para notificar al usuario una omisión puntual u otra nota de contexto durante el flujo (ejemplos: "Gemelo isométrico OMITIDO hoy por episodio de aquiles activo", "Hoy no hacemos dead hangs, en reincorporación día 4/7"). Muestra icono (por convención `⚠` para omisiones, `ⓘ` para info neutral), título corto, cuerpo explicativo, y botón "Continuar". Sin cronómetro. No cuenta hacia el progreso global. Se inserta explícitamente en el array WORKOUT como `{ type: 'info', ... }` cuando hay información específica que el usuario debe ver antes de avanzar.

---

## Sistema de diseño

### Fuentes

```css
@import url('https://fonts.googleapis.com/css2?family=DM+Sans:wght@400;500;600;700&family=JetBrains+Mono:wght@400;500;700&display=swap');
```

- Texto general: `'DM Sans', sans-serif`
- Números, cronómetros, contadores: `'JetBrains Mono', monospace`

### Paleta de colores

| Variable | Hex | Uso |
|---|---|---|
| Fondo principal | `#0f1117` | body background |
| Superficie | `#1a1d27` | cards, cajas de instrucción |
| Superficie secundaria | `#232733` | elementos elevados |
| Borde | `#2e3345` | bordes, separadores |
| Texto principal | `#e4e6ef` | títulos, contenido |
| Texto secundario | `#8b8fa3` | descripciones, labels |
| Acento principal | `#818cf8` | progress bars, fase labels, anillos de preparación |
| Ejercicio | `#f59e0b` | anillos de ejercicio, contadores de reps, equipamiento |
| Descanso / completado | `#10b981` | anillos de descanso, botón "hecho", indicadores completados |
| Calentamiento / movilidad | `#ec4899` | anillos de calentamiento y movilidad |

### Componentes

#### Botones

```css
.btn { padding: 16px 48px; border: none; border-radius: 12px; font-family: 'DM Sans'; font-size: 16px; font-weight: 600; cursor: pointer; -webkit-tap-highlight-color: transparent; touch-action: manipulation; }
.btn-primary { background: #818cf8; color: #fff; }
.btn-done { background: #10b981; color: #fff; width: 100%; max-width: 320px; padding: 20px; }
.btn-skip { background: transparent; color: #8b8fa3; border: 1px solid #2e3345; font-size: 14px; padding: 12px 32px; }
```

- Todos los botones deben tener `touch-action: manipulation` y `-webkit-tap-highlight-color: transparent` para evitar delay de toque en móvil
- Efecto `:active` con `transform: scale(0.97)` para feedback táctil

#### Anillo de cronómetro

Anillo SVG circular de 220×220px con dos círculos: fondo (`stroke: #2e3345`) y progreso (color según tipo de fase). El progreso se anima con `stroke-dashoffset` sobre un `stroke-dasharray` igual a la circunferencia (`2 * PI * 100`).

```html
<div class="timer-ring [tipo]">
  <svg viewBox="0 0 220 220">
    <circle class="bg" cx="110" cy="110" r="100"/>
    <circle class="progress" cx="110" cy="110" r="100" 
            stroke-dasharray="628.318" stroke-dashoffset="628.318"/>
  </svg>
  <div class="timer-text">
    <div class="timer-seconds">[número]</div>
    <div class="timer-label">[label]</div>
  </div>
</div>
```

Tipos de anillo y colores del stroke de progreso:
- `.warmup` → `#ec4899` (calentamiento, movilidad)
- `.exercise` → `#f59e0b` (ejercicios de fuerza, dead hangs)
- `.rest` → `#10b981` (descanso)
- Preparación (sin clase extra) → `#818cf8` (acento principal)

**Heurística de mapeo bloque → color de anillo:**

Cuando el step tiene un campo `block` (ver §Modelo de datos), el color del anillo de la pantalla de ejecución se deriva del bloque siguiendo esta regla por defecto:

| `block` contiene... | Clase del anillo | Color |
|---|---|---|
| "calentamiento", "movilidad" | `.warmup` | rosa `#ec4899` |
| "tendinoso", "bandas", "fuerza", "dead hangs", "mancuernas" | `.exercise` | naranja `#f59e0b` |
| cualquier otro bloque específico | `.exercise` por defecto | naranja `#f59e0b` |
| (pantalla de descanso) | `.rest` | verde `#10b981` |
| (pantalla de preparación) | sin clase | azul acento `#818cf8` |

```javascript
function ringClassForBlock(block) {
  if (!block) return 'exercise';
  const b = block.toLowerCase();
  if (b.includes('calentamiento') || b.includes('movilidad')) return 'warmup';
  return 'exercise';
}
```

Esta regla puede sobrescribirse puntualmente cuando un HTML específico tenga necesidades particulares (ej. una fase de trabajo cardiovascular que prefiera otro color), pero el comportamiento por defecto debe seguirla.

El SVG se rota -90° para que el progreso empiece desde las 12 en punto:
```css
.timer-ring svg { transform: rotate(-90deg); }
```

#### Barra de progreso global

Barra horizontal en la parte superior de cada pantalla que muestra el porcentaje de ejercicios completados sobre el total, con una fila meta debajo que incluye el porcentaje numérico a la izquierda y la hora estimada de finalización a la derecha.

```html
<div class="progress-bar"><div class="fill" style="width:[X]%"></div></div>
<div class="progress-meta">
  <span class="progress-pct">[X]%</span>
  <span class="progress-eta">≈ [HH:MM]</span>
</div>
```

```css
.progress-meta {
  display: flex;
  justify-content: space-between;
  align-items: center;
  font-family: 'JetBrains Mono', monospace;
  font-size: 10px;
  color: #8b8fa3;
  margin-top: 4px;
  padding: 0 2px;
}
```

#### Hora estimada de finalización

La fila meta de la barra de progreso muestra a la derecha la hora real (formato `HH:MM` en 24h) en la que la sesión terminará si se mantiene el ritmo previsto. Se actualiza al entrar en cada nuevo step, no en tiempo real segundo a segundo (para evitar ruido visual).

**Cálculo del tiempo restante:**

```javascript
function estimateRemainingSeconds() {
  let sec = 0;
  for (let i = currentStep; i < WORKOUT.length; i++) {
    const s = WORKOUT[i];
    if (s.prepTime) sec += s.prepTime;
    if (s.type === 'exercise-timed' || s.type === 'warmup-intro') {
      sec += s.duration || 0;
    } else if (s.type === 'exercise-reps') {
      sec += estimateRepsTime(s);
    } else if (s.type === 'rest') {
      sec += s.duration || 0;
    } else if (s.type === 'info') {
      sec += 10; // estimación para pantalla informativa
    }
    // Transición de bloque (3s) si el siguiente step pertenece a otro bloque
    const next = WORKOUT[i + 1];
    if (next && s.block && next.block && s.block !== next.block) sec += 3;
  }
  return sec;
}

function estimateRepsTime(step) {
  const perRep = step.tempo ? parseTempoSeconds(step.tempo) : 3;
  const reps = step.reps || 10;
  const multiplier = step.perSide ? 2 : 1;
  return perRep * reps * multiplier;
}

function parseTempoSeconds(tempo) {
  const parts = tempo.split('-').map(Number);
  const sum = parts.reduce((a, b) => a + b, 0);
  return sum || 3;
}

function formatETA(remainingSec) {
  const end = new Date(Date.now() + remainingSec * 1000);
  return end.toLocaleTimeString('es-ES', { hour: '2-digit', minute: '2-digit', hour12: false });
}
```

**Heurísticas de estimación:**

- **exercise-timed / warmup-intro:** se usa `duration` tal cual.
- **exercise-reps:** se usa `tempo` si está definido (suma de los números, ej. "3-1-2" = 6s por rep), o 3s por defecto. Si `perSide: true`, se multiplica por 2.
- **rest:** se usa `duration` tal cual.
- **info:** 10 segundos estimados (el usuario lee y pulsa "Continuar").
- **Transición de bloque:** 3s extra cuando el step actual y el siguiente pertenecen a bloques diferentes.
- **openEnded:** se usa `duration` como referencia (el usuario puede acabar antes o después).

**Importante:** la hora estimada se recalcula en cada llamada a `render()`, no con un interval. Si el usuario tarda más o menos de lo previsto, la hora se reajusta al avanzar al siguiente step. No intentar reflejar el desvío en tiempo real: el objetivo es orientativo, no métrico.

#### Caja de instrucciones

```html
<div class="instruction-box">
  <p>[instrucciones del ejercicio]</p>
  <p><span class="key">Tempo: 3-1-2</span></p>
  <p><span class="key">Por cada lado</span></p>
</div>
```

Fondo `#1a1d27`, borde `#2e3345`, border-radius 12px, texto `#8b8fa3`, highlights `.key` en `#f59e0b`.

#### Badge de ronda

```html
<div class="round-badge">Ronda <span>1</span> de 3</div>
```

#### Indicadores de sets completados (opcional)

```html
<div class="set-dots">
  <div class="set-dot done"></div>
  <div class="set-dot current"></div>
  <div class="set-dot"></div>
</div>
```

#### Banner de adaptaciones en la intro

Cuando la sesión tiene desviaciones respecto al protocolo estándar (episodios tendinosos abiertos, omisiones por material no disponible, traslados de día por condiciones externas, ventanas de protocolo de reincorporación), la pantalla de intro debe incluir un banner con las adaptaciones activas y una lista visual de elementos omitidos. Esto es un requisito operativo: el usuario debe ver de un vistazo, antes de pulsar "Comenzar sesión", qué se desvía del estándar y por qué.

**Estructura:**

```html
<div class="intro-banner">
  <strong>Adaptaciones activas:</strong><br>
  · Episodio aquiles bilateral ABIERTO (11/04)<br>
  · Reincorporación dead hangs día 4/7 (suspendidos hasta 15/04)
</div>

<div class="intro-list">
  <span class="no">NO</span> bici zona 2 &nbsp;·&nbsp; <span class="no">NO</span> dead hangs<br>
  <span class="no">NO</span> gemelo isométrico &nbsp;·&nbsp; <span class="no">NO</span> senderismo<br>
  <span class="no">NO</span> sentadilla profunda completa
</div>
```

**Estilo:**

```css
.intro-banner {
  background: #1a1d27;
  border: 1px solid #2e3345;
  border-left: 3px solid #ec4899;
  border-radius: 10px;
  padding: 14px 16px;
  margin: 0 auto 16px;
  max-width: 420px;
  font-size: 13px;
  color: #8b8fa3;
  line-height: 1.5;
}
.intro-banner strong { color: #ec4899; font-weight: 600; }

.intro-list {
  background: #1a1d27;
  border: 1px solid #2e3345;
  border-radius: 10px;
  padding: 14px 16px;
  margin: 0 auto 16px;
  max-width: 420px;
  font-size: 13px;
  color: #8b8fa3;
  line-height: 1.7;
}
.intro-list .no {
  color: #ec4899;
  font-weight: 600;
  font-family: 'JetBrains Mono', monospace;
}
```

**Reglas de uso:**

- El banner **solo** se incluye cuando hay adaptaciones reales. En una sesión sin desviaciones, la intro lleva solo nombre + descripción + botón de comenzar.
- La lista de adaptaciones activas describe incidencias abiertas o ventanas de protocolo en curso (episodios tendinosos abiertos, días de reincorporación, ajustes por lluvia/imprevistos).
- La lista de NOs enumera los elementos del protocolo estándar que se omiten o sustituyen hoy. Uso de `<span class="no">NO</span>` en rosa y fuente mono para contraste visual.
- El borde izquierdo rosa (`#ec4899`) es el indicador visual unificado de "atención, algo no es estándar hoy".

#### Detalle visible durante ejecución

El detalle completo del ejercicio (campo `detail` del modelo de datos) se muestra de forma permanente durante la pantalla de ejecución (cronometrado o de repeticiones), en una caja fija entre el nombre del ejercicio y el cronómetro/contador. No hay botón ni panel expandible: la información está siempre visible para consulta inmediata sin interacción.

```html
<div class="exercise-detail">
  <p>[texto completo del detail]</p>
  <p><span class="key">Tempo: 3-1-2</span></p>
  <p><span class="key">Por cada lado</span></p>
</div>
```

**Posición:** entre el nombre del ejercicio y la zona central de ejecución (cronómetro o contador de repeticiones). Compacta verticalmente para no desplazar el cronómetro fuera de la vista.

**Estilo:**

```css
.exercise-detail {
  background: #1a1d27;
  border: 1px solid #2e3345;
  border-radius: 10px;
  padding: 12px 14px;
  color: #8b8fa3;
  font-size: 13px;
  line-height: 1.45;
  max-width: 420px;
  width: 100%;
  margin: 0 auto 12px;
  max-height: 28vh;
  overflow-y: auto;
}
.exercise-detail p { margin-bottom: 6px; }
.exercise-detail p:last-child { margin-bottom: 0; }
```

**Comportamiento:**
- La caja aparece en ambas pantallas de ejecución (`exercise-reps` y `exercise-timed`).
- El cronómetro/contador **no se pausa** en ningún caso. La caja es solo visual.
- Si el detalle es muy largo, la caja permite scroll vertical interno (`max-height: 28vh`). El contenido no debería exceder este límite; si lo hace de forma sistemática, acortar el campo `detail`.
- Para ejercicios con tempo o marcador "por lado", usar el componente `.key` (ya definido) dentro de la caja.

**Justificación del diseño respecto a v2.1:** el panel expandible exigía un toque para leer la técnica durante la ejecución. La consulta rápida es más eficiente con la información siempre visible que tras una interacción extra. La caja fija no compite visualmente con el cronómetro porque vive en la zona superior-media de la pantalla, y el cronómetro sigue dominando el centro visual.

#### Indicador de progreso de bloque

En la parte superior de la pantalla de ejercicio, debajo de la barra de progreso global, se muestra el bloque actual y el progreso dentro del bloque. Permite al usuario saber qué está entrenando y cuánto falta del bloque sin perder la noción de la sesión completa.

```html
<div class="block-indicator">
  <div class="block-name">Tendinoso · Tren inferior</div>
  <div class="block-progress">Ejercicio 2 de 3 · Set 1 de 3</div>
</div>
```

```css
.block-indicator {
  background: transparent;
  padding: 8px 16px;
  border-bottom: 1px solid #232733;
}
.block-name {
  font-size: 11px;
  color: #818cf8;
  letter-spacing: 1px;
  text-transform: uppercase;
  font-weight: 600;
}
.block-progress {
  font-size: 10px;
  color: #8b8fa3;
  font-family: 'JetBrains Mono', monospace;
  margin-top: 2px;
}
```

**Compatibilidad hacia atrás:** si el step actual no tiene el campo `block` definido, el indicador no se renderiza para ese step. HTMLs antiguos generados con v1.0 siguen funcionando sin cambios.

### Imágenes/GIFs de ejercicios

Si hay material multimedia disponible en la carpeta `img/`:

```html
<img src="img/[nombre_ejercicio].gif" 
     style="width:240px;border-radius:12px;margin-bottom:16px;border:1px solid #2e3345" 
     alt="[nombre ejercicio]">
```

- Mostrar en la pantalla de preparación, antes de la caja de instrucciones
- Formato preferido: GIF de 3-4 segundos en loop mostrando el movimiento
- Alternativa: imagen estática JPG/PNG mostrando posición correcta
- Ancho fijo 240px, border-radius 12px, borde sutil `#2e3345`
- Si no hay imagen disponible para un ejercicio, no mostrar placeholder: simplemente omitir

---

## Lógica de la aplicación

### Modelo de datos del entrenamiento

El entrenamiento se define como un array de objetos `WORKOUT` con los siguientes tipos:

```javascript
// Pantalla de inicio
{ type: 'intro', name: 'Nombre de la sesión', detail: 'Descripción', prepTime: 0 }

// Marcador de ronda (se salta automáticamente, solo para organización)
{ type: 'round-header', round: 1 }

// Ejercicio de calentamiento cronometrado
{ type: 'warmup-intro', phase: 'Calentamiento', name: 'Nombre', detail: 'Instrucciones', prepTime: 10, duration: 60, block: 'Calentamiento dinámico' }

// Ejercicio con repeticiones
{ type: 'exercise-reps', phase: 'Ronda 1 de 3', name: 'Nombre', detail: 'Instrucciones', reps: 10, prepTime: 15, tempo: '3-1-2', equipment: 'Kettlebell', perSide: false, block: 'Tendinoso · Tren inferior' }

// Ejercicio cronometrado (duración fija)
{ type: 'exercise-timed', phase: 'Movilidad', name: 'Nombre', detail: 'Instrucciones', duration: 90, prepTime: 10, block: 'Movilidad' }

// Ejercicio cronometrado open-ended (cuenta arriba, el usuario decide cuándo parar)
{ type: 'exercise-timed', phase: 'Dead hangs', name: 'Nombre', detail: 'Instrucciones', duration: 60, prepTime: 12, openEnded: true, block: 'Tendinoso · Tren superior' }

// Descanso
{ type: 'rest', duration: 75, next: 'Nombre del siguiente ejercicio' }

// Información intermedia (omisiones, notas de contexto)
{ type: 'info', title: 'Gemelo isométrico — OMITIDO', icon: '⚠', body: 'Suspendido hoy por episodio de aquiles bilateral activo. Reincorporación condicional a resolución del episodio.' }

// Pantalla final
{ type: 'finish' }
```

### Campos del modelo

| Campo | Tipo | Descripción |
|---|---|---|
| `type` | string | Tipo de pantalla (ver arriba) |
| `phase` | string | Etiqueta de fase mostrada encima del nombre (ej. "Ronda 1 de 3", "Movilidad") |
| `name` | string | Nombre del ejercicio |
| `detail` | string | Instrucciones completas del ejercicio |
| `reps` | number | Número de repeticiones (solo exercise-reps) |
| `duration` | number | Duración en segundos (exercise-timed, warmup-intro, rest) |
| `prepTime` | number | Segundos de preparación antes de ejecutar |
| `tempo` | string | Indicación de tempo, ej. "3-1-2" (opcional) |
| `equipment` | string | Material necesario, ej. "Kettlebell" (opcional) |
| `perSide` | boolean | Si el ejercicio es por cada lado (opcional) |
| `openEnded` | boolean | Si el cronómetro cuenta arriba en lugar de abajo (opcional) |
| `next` | string | Nombre del siguiente ejercicio (solo rest) |
| `round` | number | Número de ronda (solo round-header) |
| `block` | string | Nombre del bloque al que pertenece el ejercicio (opcional). Valores típicos: `"Calentamiento dinámico"`, `"Tendinoso · Tren inferior"`, `"Tendinoso · Tren superior"`, `"Movilidad"`, `"Bandas"`, `"Bici zona 2"`. Si no está presente, el indicador de bloque no se muestra. Cuando el valor cambia entre dos steps consecutivos, se inserta pantalla de transición de 3 segundos (ver §Tipos de pantalla y §Flujo de pantallas). |
| `title` | string | Título corto de la pantalla informativa (solo `type: 'info'`). |
| `icon` | string | Icono Unicode de la pantalla informativa, por convención `⚠` para omisiones y `ⓘ` para info neutral (solo `type: 'info'`). |
| `body` | string | Cuerpo explicativo de la pantalla informativa (solo `type: 'info'`). |

### Flujo de pantallas

```
intro → [preparación → ejercicio] → [descanso → preparación → ejercicio] → ... → finish
```

Para cada ejercicio:
1. Se muestra la pantalla de preparación con cuenta atrás
2. Al terminar la cuenta atrás (o al pulsar "Saltar"), se muestra la pantalla de ejecución
3. Al completar la ejecución (cronómetro a 0, o pulsar "Hecho"/"Soltar"), se pasa al siguiente step
4. **Detección de cambio de bloque:** antes de renderizar la preparación del siguiente ejercicio, el sistema compara `steps[current].block` con `steps[next].block`. Si difieren y ambos están definidos, se renderiza la pantalla de transición de bloque (3 segundos, ver §Tipos de pantalla §7) antes de continuar con la preparación. Si coinciden o el campo `block` está ausente en alguno de los dos steps, no se inserta pantalla de transición.

Los `round-header` se saltan automáticamente (no generan pantalla).

### Señales sonoras

Sistema de audio basado en Web Audio API (AudioContext). Tres tipos de señal:

```javascript
function shortBeep() { /* 880 Hz, 0.1s — aviso 3-2-1 antes de fin */ }
function longBeep()  { /* 660 Hz, 0.3s — fin de preparación, cambio de fase */ }
function doneBeep()  { /* 1100 Hz + 1320 Hz — ejercicio completado */ }
```

Cuándo suenan:
- `shortBeep`: cuando quedan 3, 2, 1 segundos en cualquier cronómetro
- `longBeep`: al terminar la cuenta atrás de preparación, al terminar un descanso
- `doneBeep`: al completar un ejercicio (automático o por botón)

En ejercicios open-ended, `shortBeep` cada 15 segundos como referencia de tiempo.

`audioCtx.resume()` se llama en `startSession()` porque los navegadores móviles requieren interacción del usuario para activar audio.

### Conteo de progreso

```javascript
function getTotalSteps() {
  // Cuenta solo ejercicios reales (exercise-reps, exercise-timed, warmup-intro)
  // No cuenta descansos, round-headers, intro ni finish
}

function getCompletedSteps() {
  // Cuenta ejercicios completados hasta currentStep
}

function progressPercent() {
  return Math.round((getCompletedSteps() / getTotalSteps()) * 100);
}
```

### Estadísticas finales

La pantalla final muestra como mínimo:

- Duración total de la sesión (desde startSession hasta renderFinish)
- Número total de ejercicios completados
- Número de rondas de fuerza (cuando aplica)
- Mensaje con la actividad del día siguiente según el plan semanal

**Filas opcionales cuando aplica:**

- **Bloques completados:** número total de bloques de la sesión (derivado del conjunto de valores únicos del campo `block` entre los steps de ejercicio). Útil como métrica resumida en sesiones estructuradas por bloques.
- **Omisiones del día:** lista corta de los elementos del protocolo estándar que se omitieron (ej. "Gemelo iso · Sóleo"). Solo presente si la sesión incluyó pantallas tipo `info` por omisión o si el banner de intro enumeraba NOs. Ayuda al registro retrospectivo y a la memoria operativa.

**Estructura sugerida:**

```html
<div class="finish-stats">
  <div class="row"><span>Duración</span><strong>45 min</strong></div>
  <div class="row"><span>Ejercicios</span><strong>37</strong></div>
  <div class="row"><span>Bloques</span><strong>5</strong></div>
  <div class="row"><span>Omisiones</span><strong>Gemelo iso · Sóleo</strong></div>
</div>
```

El mensaje del día siguiente se renderiza en un bloque separado con borde izquierdo verde (`#10b981`) para distinguirlo de las stats.

---

## Funcionalidades de plataforma

### Wake Lock API (pantalla encendida)

```javascript
let wakeLock = null;

async function requestWakeLock() {
  if ('wakeLock' in navigator) {
    try {
      wakeLock = await navigator.wakeLock.request('screen');
      wakeLock.addEventListener('release', () => { wakeLock = null; });
    } catch (err) { console.log('Wake lock failed:', err.message); }
  }
}

function releaseWakeLock() {
  if (wakeLock) { wakeLock.release(); wakeLock = null; }
}

// Re-adquirir al volver a la pestaña
document.addEventListener('visibilitychange', () => {
  if (document.visibilityState === 'visible' && sessionStart && !wakeLock) {
    requestWakeLock();
  }
});
```

- Se activa en `startSession()` (requiere interacción del usuario)
- Se libera en `renderFinish()`
- Requiere HTTPS (GitHub Pages lo proporciona)

### Pantalla completa

```javascript
// Activar en startSession()
if (document.documentElement.requestFullscreen) {
  document.documentElement.requestFullscreen().catch(() => {});
}

// Desactivar en renderFinish()
if (document.fullscreenElement) {
  document.exitFullscreen().catch(() => {});
}
```

Oculta la barra del navegador para experiencia inmersiva.

---

## Directrices de contenido

### Instrucciones de ejercicios

- Lenguaje directo, segunda persona ("Agarra la barra", "Baja controlado")
- Incluir cues posturales específicos ("ombligo hacia dentro", "espalda neutra")
- Mencionar señales de parada si aplica ("si la espalda baja protesta, reduce rango")
- Indicar lateralidad explícitamente cuando aplique ("8 repeticiones POR LADO")
- Para ejercicios con tempo, especificar formato "X-Y-Z" (bajada-pausa-subida)

**Completitud del campo `detail`:**

El detalle del ejercicio debe ser lo más completo posible porque se muestra de forma permanente durante la ejecución (ver §Componentes → Detalle visible durante ejecución). El usuario debe poder leerlo mientras entrena para consultar cues sin detener la sesión. Incluir, cuando aplique:

1. **Setup de la posición inicial**: de pie / tumbado / en cuadrupedia, orientación corporal, posición de manos y pies, alineación articular.
2. **Cues posturales específicos**: activación de músculos clave ("glúteo activo", "ombligo hacia dentro"), alineación ("rodillas sobre los pies", "espalda neutra"), respiración ("exhala al abrir").
3. **Movimiento exacto**: qué se mueve, qué se mantiene fijo, rango de movimiento, tempo si aplica.
4. **Señales de parada específicas del contexto**: condiciones en las que se debe interrumpir el ejercicio (ej. "si aparece sensación en aquiles, salta este ejercicio"). Siempre incluir estas señales cuando hay una incidencia activa relacionada con la estructura trabajada.
5. **Material con especificación precisa**: cuando un ejercicio usa material con variantes de peso/fuerza (bandas elásticas de distintas resistencias, mancuernas de distintos kilos, kettlebells), **especificar siempre qué versión usar**. No escribir "con banda" a secas — escribir "con banda de 5kg" o "con banda de 15kg". Para mancuernas: "mancuerna de 12 kg". Es información esencial que evita tener que interrumpir el cronómetro para decidir qué coger.
6. **Contexto del episodio abierto** si el ejercicio está relacionado con una incidencia: mencionar explícitamente el estado (ej. "hombro izquierdo asintomático: rango completo"; "retracción escapular, NO es dead hang — los dead hangs están suspendidos hasta el 15/04").

El umbral es: si el usuario necesitara consultar otra fuente durante el ejercicio, el `detail` no está completo. Si el campo excede los ~28vh visibles (rango útil con scroll interno de la caja), considerar dividir el ejercicio en dos steps más cortos o reescribir de forma más concisa sin perder información crítica.

### Tiempos de preparación

| Tipo de transición | Tiempo prep |
|---|---|
| Primer ejercicio de la sesión | 12-15 segundos |
| Cambio de ejercicio dentro del mismo bloque (mismo material, misma posición general) | 5-8 segundos |
| Cambio de ejercicio dentro del mismo bloque con material diferente | 8-10 segundos |
| Cambio entre bloques completos (ej. tendinoso → movilidad → bici) | 10-12 segundos |
| Primer ejercicio de nueva ronda (cuando aplica) | 8 segundos |
| Ejercicio de movilidad dentro del bloque de movilidad | 5-8 segundos |

Los tiempos de preparación se han reducido respecto a la versión inicial del SKILL. Justificación: el tiempo de preparación es logístico (cambio de posición, ajuste de material, lectura de instrucción), no fisiológico. Reducir prep time NO compromete la recuperación porque los descansos fisiológicos entre series del mismo ejercicio se mantienen en 60-90 segundos. La reducción aplica a transiciones entre ejercicios distintos, no a descansos entre series del mismo ejercicio.

### Tiempos de descanso

| Contexto | Descanso |
|---|---|
| Entre ejercicios dentro de una ronda | 60-90 segundos |
| Entre rondas | 90-120 segundos |
| Entre ejercicios de movilidad | 0 (encadenados) |
| Entre series de dead hangs | 60-90 segundos |

### Estructura de una sesión tipo

```
1. Intro
2. Calentamiento (2-3 ejercicios de movilidad, 60s cada uno)
3. Rondas de fuerza (3-4 rondas × 4-6 ejercicios)
4. Movilidad / vuelta a la calma (5-7 ejercicios de estiramiento)
5. Trabajo específico (dead hangs, bandas, etc.)
6. Pantalla final
```

---

## Nomenclatura y ubicación de archivos

- Carpeta destino: `protocolos_sesiones/` (en la raíz del repositorio). Los HTMLs de protocolos y sesiones viven aquí. El `index.html` de la raíz los lista automáticamente.
- Excepciones que NO van en la subcarpeta: `index.html` (raíz) y `02_fase1_cronologia_plan_diario.html` (raíz, plan diario interactivo accedido directamente).
- Formato: `[número]_[nombre_descriptivo].html`
- Ejemplos:
  - `protocolos_sesiones/01_sesion_complementaria_guia.html`
  - `protocolos_sesiones/02_protocolo_tendinoso_inferior.html`
  - `protocolos_sesiones/03_sesion_bici_zona2.html`
  - `protocolos_sesiones/04_movilidad_completa.html`
- Imágenes: `img/[nombre_ejercicio].[gif|jpg|png]` en la raíz (compartidas). Si se añaden imágenes específicas de un protocolo, mantener las rutas relativas desde el HTML teniendo en cuenta la subcarpeta (`../img/...`).
  - Ejemplos: `img/sentadilla_goblet.gif`, `img/dead_hang.jpg`

---

## Checklist de generación

Antes de entregar un HTML de entrenamiento, verificar:

- [ ] `<meta name="viewport">` con `user-scalable=no`
- [ ] Fuentes DM Sans + JetBrains Mono cargadas desde Google Fonts
- [ ] Wake Lock API implementada con re-adquisición en visibilitychange
- [ ] Pantalla completa activada en startSession
- [ ] AudioContext con resume() tras interacción del usuario
- [ ] Todos los botones con `touch-action: manipulation`
- [ ] Pantallas usan `100dvh` para altura completa en móvil
- [ ] Cada ejercicio tiene prepTime, detail, y type correctos
- [ ] Los descansos indican el nombre del siguiente ejercicio
- [ ] La pantalla final libera wake lock y sale de fullscreen
- [ ] La pantalla final indica la actividad del día siguiente
- [ ] La barra de progreso refleja ejercicios completados / total
- [ ] La fila meta de la barra de progreso muestra porcentaje y hora estimada de finalización (formato `≈ HH:MM`)
- [ ] Señales sonoras en los momentos correctos
- [ ] Probado mentalmente el flujo completo de inicio a fin
- [ ] Cada step del WORKOUT tiene asignado el campo `block` correspondiente
- [ ] Los tiempos de prep siguen la nueva tabla diferenciada (5-15s según tipo de transición)
- [ ] El detalle completo del ejercicio está visible de forma permanente durante la ejecución en una caja fija entre el nombre y el cronómetro/contador
- [ ] El detalle especifica material con peso/fuerza cuando aplica (ej. "banda de 5 kg", "mancuerna de 12 kg")
- [ ] El indicador de bloque y progreso intra-bloque se muestra en cada ejercicio
- [ ] El color del anillo del cronómetro sigue la heurística de mapeo bloque → clase (warmup/exercise/rest)
- [ ] Los cambios de bloque incluyen pantalla de transición de 3 segundos con longBeep inicial + shortBeep en cada segundo de la cuenta atrás
- [ ] Las omisiones puntuales (ejercicios del protocolo que no se hacen hoy) se notifican mediante pantalla `type: 'info'` en el array WORKOUT
- [ ] Si la sesión tiene adaptaciones respecto al protocolo estándar, la pantalla de intro incluye el banner de adaptaciones con lista de NOs

---

## Ejemplo de generación

Al recibir un prompt como "genera una guía HTML para el protocolo tendinoso de tren inferior", el proceso es:

1. Consultar el plan de entrenamiento del proyecto para obtener los ejercicios, series, tiempos y detalles
2. Construir el array WORKOUT con la secuencia correcta
3. Aplicar el sistema de diseño (colores, fuentes, componentes)
4. Incluir wake lock, fullscreen, audio
5. Incluir referencias a imágenes si existen en `img/`
6. Nombrar el archivo según la nomenclatura
7. Verificar checklist
8. Entregar archivo listo para subir al repositorio
