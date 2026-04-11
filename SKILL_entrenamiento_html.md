# SKILL — Generador de guías interactivas de entrenamiento HTML

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

1. **Intro** — Nombre de la sesión, descripción, botón "Comenzar sesión"
2. **Preparación** — Nombre del ejercicio, instrucciones, equipamiento necesario, cuenta atrás de preparación. Botón "Saltar"
3. **Ejercicio con repeticiones** — Nombre, número de reps, tempo si aplica, indicación "por lado" si aplica. Botón "Hecho"
4. **Ejercicio cronometrado** — Nombre, cronómetro circular animado (cuenta atrás o cuenta arriba para open-ended). Botón "Soltar" para open-ended
5. **Descanso** — Cronómetro circular, indicación del siguiente ejercicio. Botón "Saltar descanso"
6. **Final** — Resumen con estadísticas, mensaje del siguiente día del plan

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

El SVG se rota -90° para que el progreso empiece desde las 12 en punto:
```css
.timer-ring svg { transform: rotate(-90deg); }
```

#### Barra de progreso global

Barra horizontal en la parte superior de cada pantalla que muestra el porcentaje de ejercicios completados sobre el total.

```html
<div class="progress-bar"><div class="fill" style="width:[X]%"></div></div>
```

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
{ type: 'warmup-intro', phase: 'Calentamiento', name: 'Nombre', detail: 'Instrucciones', prepTime: 10, duration: 60 }

// Ejercicio con repeticiones
{ type: 'exercise-reps', phase: 'Ronda 1 de 3', name: 'Nombre', detail: 'Instrucciones', reps: 10, prepTime: 15, tempo: '3-1-2', equipment: 'Kettlebell', perSide: false }

// Ejercicio cronometrado (duración fija)
{ type: 'exercise-timed', phase: 'Movilidad', name: 'Nombre', detail: 'Instrucciones', duration: 90, prepTime: 10 }

// Ejercicio cronometrado open-ended (cuenta arriba, el usuario decide cuándo parar)
{ type: 'exercise-timed', phase: 'Dead hangs', name: 'Nombre', detail: 'Instrucciones', duration: 60, prepTime: 12, openEnded: true }

// Descanso
{ type: 'rest', duration: 75, next: 'Nombre del siguiente ejercicio' }

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

### Flujo de pantallas

```
intro → [preparación → ejercicio] → [descanso → preparación → ejercicio] → ... → finish
```

Para cada ejercicio:
1. Se muestra la pantalla de preparación con cuenta atrás
2. Al terminar la cuenta atrás (o al pulsar "Saltar"), se muestra la pantalla de ejecución
3. Al completar la ejecución (cronómetro a 0, o pulsar "Hecho"/"Soltar"), se pasa al siguiente step

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

La pantalla final muestra:
- Duración total de la sesión (desde startSession hasta renderFinish)
- Número total de ejercicios completados
- Número de rondas de fuerza
- Mensaje con la actividad del día siguiente según el plan semanal

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

### Tiempos de preparación

| Tipo de transición | Tiempo prep |
|---|---|
| Primer ejercicio de la sesión | 15 segundos |
| Cambio de ejercicio con mismo material | 8-10 segundos |
| Cambio de ejercicio con material diferente | 12-15 segundos |
| Primer ejercicio de nueva ronda | 10 segundos |
| Ejercicio de movilidad | 8-10 segundos |

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
- [ ] Señales sonoras en los momentos correctos
- [ ] Probado mentalmente el flujo completo de inicio a fin

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
