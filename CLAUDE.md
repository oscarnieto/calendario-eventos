# Contexto del proyecto

Calendario mensual de eventos de Savills. Un único archivo `index.html`
autocontenido, publicado en GitHub Pages, que se adapta a tres contextos con el
mismo enlace: la cartelería táctil de oficina (**MOPI**), **escritorio** y
**móvil**.

- Repo: `oscarnieto/calendario-eventos`, rama `main`.
- Publicado en <https://oscarnieto.github.io/calendario-eventos/>
- Documentación de uso para el equipo (cómo se actualizan los eventos, formato
  del Excel, detalle de cada diseño): **`README.md`**. Este archivo es el
  contexto de trabajo; el README es el manual.

---

## Reglas duras (no negociables)

Vienen del encargo original y condicionan cualquier cambio:

1. **Un solo archivo.** CSS, JavaScript y librerías embebidos dentro de
   `index.html`. Nada de `<link>` ni `<script src>` a terceros.
2. **Cero peticiones a CDN o APIs externas.** El player del MOPI puede tener la
   red capada. Todo lo que hace falta va embebido: SheetJS (`xlsx.mini.min.js`
   0.18.5), `qrcode-generator` 1.4.4, la tipografía Montserrat en base64
   (Light/Medium/Bold, WOFF2 + WOFF de reserva) y el favicon.
3. **Sin build, sin npm, sin framework.** HTML, CSS y JavaScript planos. Nada de
   React, Vue ni empaquetadores.
4. **Sintaxis conservadora (ES5).** El player puede ser un WebView antiguo
   (Tizen, webOS): `var`, `function`, `XMLHttpRequest`. Nada de `fetch`,
   promesas, funciones flecha, plantillas, `let`/`const`, encadenamiento
   opcional ni anidamiento CSS nativo. Flexbox y Grid sí. **Todo `clamp()` o
   `min()` lleva antes una declaración con valor plano de reserva.**
5. **Tolerancia a errores.** Una fila del Excel con fecha mala o sin título se
   ignora en silencio; el calendario no puede quedarse nunca en blanco. Los
   avisos van a `console.warn`.
6. **Nada de backend, base de datos ni panel de administración.** El Excel del
   repo es la única fuente de verdad; `localStorage` es solo caché.

---

## Cómo está montado `index.html`

Unos 5.000 renglones, casi todos base64 de las fuentes y las librerías. El
orden es:

| Zona | Qué hay |
|---|---|
| `<head>` | favicon embebido y un `<style>` con seis secciones numeradas |
| CSS 1–3 | tokens compartidos, reset, Montserrat en base64, la vista única |
| CSS 4 | **lenguaje visual común**: cabecera, rejilla, cards, popup, agenda |
| CSS 5 | `:root[data-modo="mopi"]` — lienzo fijo de 1080×1920 escalado |
| CSS 6 | `:root[data-modo="escritorio"]` y `[data-modo="movil"]` — fluido |
| `<body>` | **un solo marcado** para los tres modos |
| `<script>` | la app (IIFE `CAL`), luego SheetJS, luego qrcode-generator, luego el arranque |

**La idea central:** los tres modos comparten marcado y un único renderizador.
Lo que cambia es el CSS que cuelga de `:root[data-modo="…"]`, para que el
override por URL (`?modo=mopi|escritorio|movil`) siga mandando por encima de la
detección automática. Si añades algo a un modo, hazlo con una regla que cuelgue
de su `data-modo`, no con un marcado nuevo.

### Detección de modo — `detectarModo()`

```
?modo=…            → manda siempre
vertical, ≥1000px de ancho y proporción ≤ 0,66  → mopi   (un tótem 9:16)
ancho < 760px                                    → movil
resto                                            → escritorio
```
Una tablet en vertical (3:4) **no** es un MOPI: se lleva el diseño de
escritorio. Al redimensionar, `revisarModo()` decide si repinta.

### Funciones que conviene conocer

| Función | Para qué |
|---|---|
| `pintarMopi()` | el renderizador único: cabecera, rejilla, rail, agenda |
| `ajustarRejilla()` | encuadre + recorte de títulos + encaje de cards, con segunda pasada diferida |
| `encuadrarRejilla()` | acota la proporción de la card entre 0,85 y 1,45 y centra la rejilla |
| `recortarTitulos()` | busca por bisección dónde cortar cada título y le pone `…` |
| `ajustarCards()` | baja el título a 2 líneas, luego a 1, y solo entonces esconde eventos |
| `pintarRail()` / `espaciarMeses()` / `medirDock()` | el menú de meses de escritorio y su efecto dock |
| `pintarAgenda()` / `alternarAgenda()` | la lista desplegable de móvil |
| `pintarPanelMopi()` / `pintarModalMopi()` | el popup del día |
| `cargarDatos()` / `leerLibro()` / `aFechasIso()` | lectura y saneado del Excel |

### Datos

`datos/calendario.xlsx` con cache-busting `?v=' + Date.now()`, caché en
`localStorage` (**el Excel manda siempre que se pueda leer**), recarga cada 30
minutos y salto automático al mes en curso cuando cambia la fecha. El parser es
deliberadamente tolerante: busca la fila de cabeceras en las doce primeras,
acepta alias de nombre de columna, fechas multi-día en una celda
(`29, 30 ,1 y 2 /09 y 10/2026`) y rangos de hora (`09:30 a 13:00`). El detalle
está en el README.

---

## Diseño

Figma [Calendario](https://www.figma.com/design/LqOyrOC477j02aAt3ok6az/Calendario):

| Frame | Nodo | Qué es |
|---|---|---|
| `Muppie_1080x1920` | `1:2` | el cartel del MOPI |
| `Muppie_1080x1920_POPUP` | `12:248` | el popup del día |
| `MacBook Pro 14"` | `19:24` | la retícula de escritorio |

Colores de marca: amarillo `#ffdf00`, rojo `#c90c0f`, azul de fondo `#25273a`,
card sin eventos `#2d2e45`.

**Cuando llegue un diseño nuevo de Figma:** aplícalo **solo al modo que
corresponda**, sin tocar los otros dos, actualizando las variables CSS de su
bloque `:root[data-modo="…"]`. Si el Figma introduce un color de categoría que
no existía, **avísale en vez de decidir un tono por tu cuenta**. Confirma en una
línea qué frame has aplicado a qué modo.

Decisiones ya tomadas, por si se replantean:

- El bloque blanco del frame de escritorio es **el hueco** donde va el
  calendario, no un panel blanco: las cards van sobre el fondo oscuro, igual
  que en el resto de adaptaciones.
- El **QR es solo del MOPI** (la única pantalla que se escanea desde otro
  aparato). En escritorio y móvil va un botón **«Apúntate aquí»**.
- El MOPI usa rejilla de **lunes a viernes**; escritorio y móvil, de **lunes a
  domingo**, para que no se pierdan los eventos de fin de semana.
- El menú de meses de escritorio no lleva selector de año ni marcas de «mes con
  eventos»: solo los doce meses, como en el Figma.

---

## Cómo se trabaja

- Tras cada cambio que quede en estado probable (no a medio hacer): **commit y
  push directamente a `main`**. Sin rama ni PR.
- Mensajes de commit **en español**, cortos y descriptivos.
- Al terminar, **una línea** diciendo qué se ha commiteado y que ya está
  desplegado. Sin resúmenes largos.
- Si un cambio deja algo roto o incompleto, **no** hagas push: dilo y espera.
- Si tocas el diseño o el comportamiento de un modo, actualiza el `README.md`
  en el mismo commit.

### Probar en local

```bash
python3 -m http.server 8000
```

Y abrir `http://localhost:8000/`. Los tres modos se fuerzan con
`?modo=mopi`, `?modo=escritorio` y `?modo=movil`.

Para los cambios de maqueta merece la pena verificar con un navegador
automatizado (Playwright sirve) en varias resoluciones a la vez, comprobando:

- que en escritorio **no hay scroll** y el pie entra en la ventana;
- que ninguna `.cal-card-lista` desborda (`scrollHeight > clientHeight`);
- que la proporción de la card se queda entre 0,85 y 1,45;
- que no hay scroll horizontal en ningún modo;
- que el MOPI sigue midiendo exactamente 1920 de alto.

---

## Trampas conocidas

Cosas que ya han mordido una vez:

- **Columnas de rejilla deformadas por el contenido.** Siempre
  `repeat(N, minmax(0, 1fr))` y `min-width:0` en los hijos.
- **`-webkit-line-clamp` que no aplica.** Un `max-height` con
  `box-sizing:border-box` recorta antes que el clamp; y las reglas `[data-n]`
  ganan en especificidad al `@supports`, así que hay que listar **todos** los
  selectores dentro del bloque `@supports`.
- **Elementos posicionados que tapan el contenido.** En escritorio la foto y el
  velo van en `position:absolute`, así que el contenido necesita su propio
  `z-index`; si no, se pinta debajo.
- **Cajas al 100% que crecen con un `transform`.** Los meses del menú van a
  ancho de contenido: con la caja al 100%, al aumentar con el dock se estiraba
  por encima del calendario y le robaba los clics.
- **Delegación de clics por subcadena.** Comprobar la clase por token
  (`(' '+className+' ').indexOf(' '+c+' ') !== -1`), no con `indexOf` pelado:
  `cal-celda` casaba con `cal-celda-num`.
- **Métricas de tipografía que llegan tarde.** El encaje de las cards se mide
  dos veces: en el pintado y otra vez en diferido, porque las medidas finales
  de la fuente pueden llegar después y descuadrar por tres píxeles.
- **`clamp()` sin valor de reserva.** Rompe el layout entero en el WebView del
  MOPI. Declara siempre el valor plano antes.

---

## Preferencias de Oscar

Diseñador gráfico, no desarrollador. Responde **siempre en español**, tono
directo, sin florituras ni frases motivacionales. Dale el resultado sin
explicarle el proceso salvo que lo pida, sin introducciones ni resúmenes al
final. Si necesitas una aclaración antes de empezar, pregúntale: no asumas ni
rellenes con suposiciones.
