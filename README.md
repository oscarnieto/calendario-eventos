# Calendario de eventos

Calendario mensual publicado en GitHub Pages. Un único archivo `index.html`
autocontenido que se adapta a tres contextos con el **mismo enlace**:

| Modo | Dónde | Cómo se activa |
|---|---|---|
| **MOPI** | Cartelería táctil vertical de oficina (1080×1920) | `?modo=mopi` en el player |
| **Escritorio** | Ordenador, desde el correo | Automático (pantalla horizontal) |
| **Móvil** | Teléfono, desde el correo | Automático (pantalla vertical estrecha) |

El enlace que se envía por correo es la URL sin parámetros: el propio
calendario detecta el dispositivo.

---

## Actualizar los eventos (equipo no técnico)

1. Descarga `datos/eventos.xlsx`.
2. Edítalo en Excel o Google Sheets (si usas Sheets, exporta de nuevo a `.xlsx`).
3. En GitHub, entra en la carpeta `datos/` y arrastra el archivo encima para
   sustituirlo. **El nombre tiene que seguir siendo `eventos.xlsx`.**
4. Confirma el cambio ("Commit changes").
5. Espera entre 30 segundos y 2 minutos y recarga la página del calendario.

No hay que tocar ningún otro archivo. El calendario lee el Excel cada vez que
se abre y, además, lo vuelve a comprobar solo cada 30 minutos (importante para
el MOPI, que está encendido todo el día).

### Imágenes

Si un evento lleva imagen, súbela a `assets/img/` y escribe **solo el nombre
del archivo** en la columna `imagen` (por ejemplo `afterwork.jpg`). Si el
archivo no existe, el evento se muestra igual, sin imagen.

---

## Estructura del Excel

### Hoja `Eventos`

La primera fila son las cabeceras y **no se deben renombrar**.

| Columna | Tipo | Obligatorio | Notas |
|---|---|---|---|
| `id` | texto | Sí | Identificador único (`EVT-001`, …) |
| `fecha` | fecha | Sí | `DD/MM/AAAA` |
| `hora_inicio` | texto | No | `HH:MM` |
| `hora_fin` | texto | No | `HH:MM` |
| `titulo` | texto | Sí | |
| `descripcion` | texto | No | Puede ser largo |
| `categoria` | texto | Sí | Determina el color |
| `ubicacion` | texto | No | |
| `ponentes` | texto | No | Separados por punto y coma `;` |
| `url_registro` | texto | No | Origen del QR y del botón de inscripción |
| `imagen` | texto | No | Nombre de archivo dentro de `assets/img/` |
| `destacado` | SÍ/NO | No | Entra en la rotación automática del MOPI |
| `estado` | texto | Sí | `activo`, `cancelado` o `completo` |

### Hoja `Categorias`

Dos columnas: `categoria` y `color` (hexadecimal, `#2F6FED`). De aquí sale el
código de color del calendario. Si una categoría usada en la hoja `Eventos` no
aparece aquí, se pinta en gris neutro y el calendario sigue funcionando.

### Estados

- **activo** — se muestra normal, con QR y botón de inscripción.
- **cancelado** — título tachado, sin QR ni botón.
- **completo** — visible, con etiqueta "Aforo completo" y la inscripción desactivada.

### Si una fila está mal

Una fila con la fecha mal escrita o sin título **se ignora en silencio**: el
resto del calendario se pinta igual. Nunca se queda en blanco por un error de
una celda. Los avisos quedan en la consola del navegador (F12 → Consola) por
si hay que revisarlos.

---

## Notas técnicas

- **Un solo archivo.** `index.html` lleva dentro el CSS, el JavaScript,
  [SheetJS](https://sheetjs.com) (lectura del Excel) y
  [qrcode-generator](https://github.com/kazuhikoarase/qrcode-generator) (QR).
  Cero peticiones a CDN ni a APIs externas: el MOPI puede tener la red capada.
- **Sin build ni dependencias.** HTML, CSS y JavaScript planos, en sintaxis ES5
  para que funcione en WebViews antiguos (Tizen, webOS).
- **Caché local.** Los últimos datos válidos se guardan en el navegador. Si el
  Excel no se puede descargar, el calendario pinta desde esa copia y muestra un
  aviso discreto con la fecha del último dato bueno. El Excel siempre manda
  cuando está disponible.
- **QR.** Se generan en el navegador a partir de `url_registro`, con corrección
  de errores M y zona de silencio, y llevan UTM según el modo
  (`mopi/cartel`, `email/escritorio`, `email/movil`). En móvil no hay QR: hay
  botón directo.
- **MOPI.** A los 45 s sin tocar vuelve a la vista de mes; a los 2 min entra en
  modo atracción y rota los eventos destacados cada 12 s. Cualquier toque lo
  interrumpe.

## Diseño del MOPI

El modo MOPI implementa el frame `1:2 Muppie_1080x1920` del Figma
[Calendario](https://www.figma.com/design/LqOyrOC477j02aAt3ok6az/Calendario?node-id=1-2).
Todo el color, la tipografía y la geometría viven en variables CSS bajo
`:root[data-modo="mopi"]`, así que un reajuste del Figma es un cambio localizado.

- **Rejilla de lunes a viernes**, 5 columnas × hasta 5 filas. Un mes nunca ocupa
  más de cinco filas laborables, por eso el lienzo de 1920 px siempre cuadra y
  no hay scroll en ningún estado.
- **Card amarilla** (`#ffdf00`) los días con eventos, número en rojo `#c90c0f`.
  **Card azul** (`#2d2e45`) los días sin eventos, número en amarillo.
- Hasta 3 eventos por card. A partir del cuarto, los tres primeros más un
  **Ver todos (N)** en rojo, que abre el popup del día con la lista completa.
- Los títulos que no caben se recortan con puntos suspensivos: cuantos menos
  eventos tenga el día, más líneas de título caben (1 evento son 5 líneas, 2 son
  4, y a partir de 3 son 2).
- Tipografía de la rejilla como en el Figma (evento 14 px, número 30 px); la
  cabecera de día sube a 20 px.
- Tipografía **Montserrat** (Light/Medium/Bold) embebida en base64 dentro del
  HTML, con WOFF2 y WOFF de reserva. Los modos escritorio y móvil siguen con la
  pila del sistema.
- Los eventos en **sábado o domingo** no caben en una rejilla de lunes a viernes:
  no salen en la rejilla pero sí en la rotación de destacados, y quedan
  registrados con `console.warn`.

### Imágenes de marca

| Archivo | Qué es |
|---|---|
| `assets/img/hero.jpg` | Foto de cabecera, 1080×542. Sustitúyela por la del Figma arrastrándola encima. |
| `assets/img/logo-savills.svg` | Logo de la esquina superior izquierda, 121×121. |

Las dos son **provisionales**: se generaron aquí porque el entorno no tiene
acceso de red a `figma.com` para exportarlas. Sustituirlas es arrastrar el
archivo con el mismo nombre; no hay que tocar código.

### Popup del día y modo atracción

El Figma cubre el estado en reposo (el cartel). La interacción táctil no está
dibujada y se resuelve con dos piezas en la misma paleta:

- **Popup del día.** Tocar una card abre un cuadro centrado sobre el cartel, con
  cabecera amarilla y la fecha en rojo (el mismo par de la card). Si el día tiene
  un evento, sale el detalle completo con su QR; si tiene varios, sale la lista y
  al tocar uno se abre su detalle. Se cierra con la X o tocando fuera. Nunca hay
  scroll: la lista evita que el contenido crezca.
- **Modo atracción.** A los 2 minutos sin tocar, el cartel deja paso a una capa a
  pantalla completa que rota los eventos destacados cada 12 segundos. Cualquier
  toque la corta.

Los tamaños del popup son de lectura a un brazo de distancia, que es como se usa
el MOPI. Cuando tengas ese frame en Figma, sustituye solo ese bloque.

### Comprobar en local

```
python3 -m http.server 8000
```

Y abrir `http://localhost:8000/` (`?modo=mopi` fuerza el layout del MOPI
escalado a la ventana; `?modo=escritorio` y `?modo=movil` hacen lo propio).
