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

1. Abre `datos/info_mupi.xlsx` y edítalo (Excel o Google Sheets; si usas Sheets,
   exporta de nuevo a `.xlsx`).
2. En GitHub, entra en la carpeta `datos/` y arrastra el archivo encima para
   sustituirlo. **El nombre tiene que seguir siendo `info_mupi.xlsx`.**
3. Confirma el cambio ("Commit changes").
4. Espera entre 30 segundos y 2 minutos y recarga la página.

No hay que tocar ningún otro archivo. El calendario lee el Excel cada vez que se
abre y, además, lo vuelve a comprobar solo cada 30 minutos, que es lo que importa
en el MOPI porque está encendido todo el día.

> **Los datos de septiembre de 2026 son de ejemplo.** El archivo trae 18 eventos
> inventados para ver el calendario lleno, con enlaces a `example.com`. Bórralos
> y escribe los reales encima. Las dos filas que venían de fábrica —*Living
> Trends* y *Property & Facility Management Summit*— siguen ahí; a la primera se
> le corrigió el año, que ponía 2016.

## Estructura del Excel

Una sola hoja con las cabeceras en la primera fila. **El nombre de la hoja da
igual**: si no hay ninguna llamada `Eventos`, se usa la primera del libro.

| Columna | Obligatoria | Qué hace |
|---|---|---|
| `Nombre evento` | **Sí** | Título del evento |
| `Fecha` | **Sí** | Formato fecha de Excel, o `DD/MM/AAAA` escrito a mano |
| `Horario` | No | `19:00`, o un rango en la misma casilla: `19:00 - 21:00` |
| `Lugar` | No | Sale bajo el título en el popup |
| `Tipo de evento` | No | Se muestra como etiqueta |
| `¿Quién participa?` | No | Varios nombres separados por punto y coma `;` |
| `Link QR` | No | De aquí sale el QR. Sin link, no se reserva el hueco |

Una fila sin fecha o sin nombre **se ignora en silencio** y el resto del
calendario se pinta igual. Los avisos quedan en la consola del navegador
(F12 → Consola) por si hay que revisarlos.

### Los nombres de las columnas admiten variantes

No hace falta escribirlas exactamente así. Se ignoran mayúsculas, tildes,
signos (`¿?`) y espacios sobrantes, y cada campo acepta varios nombres:

| Campo | Cabeceras que valen |
|---|---|
| Título | `Nombre evento`, `Evento`, `Nombre`, `Título` |
| Fecha | `Fecha`, `Día` |
| Hora | `Horario`, `Hora`, `Hora inicio`, `Inicio` |
| Hora de fin | `Hora fin`, `Fin`, `Hasta` |
| Lugar | `Lugar`, `Ubicación`, `Sala`, `Dónde` |
| Tipo | `Tipo de evento`, `Tipo`, `Categoría` |
| Participantes | `¿Quién participa?`, `Ponentes`, `Participantes` |
| Enlace | `Link QR`, `QR`, `Link`, `URL`, `Enlace`, `Inscripción` |

### Columnas opcionales que añaden cosas

Si en algún momento quieres más, basta con añadir la columna:

| Columna | Qué añade |
|---|---|
| `Descripción` | Texto largo bajo los datos del evento, en el popup |
| `Estado` | `cancelado` (título tachado, sin QR) o `completo` (registro desactivado) |
| `Destacado` | `SÍ` para entrar en la rotación automática del modo atracción |
| `Imagen` | Nombre de un archivo de `assets/img/` |
| `id` | Identificador propio; si no está, se genera solo |

Y una segunda hoja llamada `Categorias`, con las columnas `categoria` y `color`
(hexadecimal), pinta cada tipo de evento con su color. Sin esa hoja, las
etiquetas van en el amarillo de marca.

### Si el archivo se llama de otra forma

El calendario busca `datos/info_mupi.xlsx` y, si no lo encuentra,
`datos/eventos.xlsx`. Cualquiera de los dos nombres vale, pero **conviene tener
solo uno** en el repositorio para no acabar mirando datos viejos sin darte
cuenta.

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
- **QR.** Se generan en el navegador a partir de la columna `Link QR`, con corrección
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

### Popup del día

Implementa el frame `12:248 Muppie_1080x1920_POPUP`. Tocar una card tapa la banda
de la rejilla con el fondo `#25273a` —la cabecera con la foto y el pie de contacto
siguen a la vista— y abre una tarjeta amarilla de 985×774 con radio 19.

- Cabecera: día en rojo bold y mes en light, a 71 px, con la X roja arriba a la
  derecha (área táctil de 88 px alrededor del icono de 38).
- Cada evento: título 45 px bold y filas de 32 px con icono de reloj, ubicación,
  participantes y tipo de evento. El QR va a la derecha, blanco, de 146,6 px.
- Filete separador entre eventos, ninguno antes del primero.
- Cada fila desaparece si su casilla del Excel está vacía, y sin `Link QR` el
  hueco del QR no se reserva.
- La tarjeta crece con el contenido hasta el borde inferior de la máscara. Cuatro
  eventos caben enteros; a partir del quinto la lista se desliza y un degradado
  inferior lo indica.

Los iconos están dibujados a mano con la geometría de akar-icons porque el
entorno no tiene salida a `figma.com` para exportar los del archivo.

### Modo atracción

A los 2 minutos sin tocar, el cartel deja paso a una capa a pantalla completa que
rota los eventos destacados cada 12 segundos, con el QR a 420 px porque se lee
desde más lejos. Cualquier toque la corta. Ese frame no está en Figma todavía.

### Comprobar en local

```
python3 -m http.server 8000
```

Y abrir `http://localhost:8000/` (`?modo=mopi` fuerza el layout del MOPI
escalado a la ventana; `?modo=escritorio` y `?modo=movil` hacen lo propio).
