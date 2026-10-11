# JESI Estudio · Portfolio web

Sitio de una sola página (single-file) para mostrar y vender los servicios de **edición de video**, **páginas web** y **apps móviles**. Está hecho con HTML, CSS y JavaScript puro: **sin frameworks, sin build y sin dependencias** (solo Google Fonts por CDN).

> El archivo `portfolio.html` está comentado de punta a punta (CSS, HTML y JS). Este README es el mapa para moverte por él.

---

## Contenido

- [Características](#características)
- [Estructura del proyecto](#estructura-del-proyecto)
- [Cómo correrlo en local](#cómo-correrlo-en-local)
- [Qué editar (datos)](#qué-editar-datos)
- [Cómo funciona el código](#cómo-funciona-el-código)
- [Diseño y branding](#diseño-y-branding)
- [Accesibilidad](#accesibilidad)
- [Limitaciones conocidas](#limitaciones-conocidas)
- [Deploy](#deploy)
- [Pendientes (TODO)](#pendientes-todo)

---

## Características

- **Navbar fija** con navegación por anclas.
- **Carrusel de promociones** con autoplay, flechas y puntos (respeta `prefers-reduced-motion`).
- **Portfolio filtrable** por categoría (Video / Web / App) con visor modal.
- **Modo edición** para agregar y borrar trabajos desde el navegador (con portada opcional).
- **Precios con modal** de detalle: qué incluye y condiciones, por cada servicio.
- **Formulario de contacto sin backend**: arma el mensaje y abre WhatsApp.
- Diseño **dark** con la paleta y tipografías de la marca, 100% responsive.

## Estructura del proyecto

```
.
├── portfolio.html   # Todo el sitio: CSS + HTML + JS en un solo archivo
└── README.md
```

Dentro de `portfolio.html` el orden es:

| Bloque | Contenido |
|---|---|
| `<style>` | Variables de marca → base → navbar → botones → carrusel → secciones → modales → footer |
| `<header>` | Navbar |
| `<main>` | Secciones: `#inicio`, `#quien-soy`, `#trabajos`, `#contratar`, `#precios`, `#contacto` |
| `<footer>` | Pie de página |
| `<dialog>` ×3 | `#info` (detalle de servicio), `#viewer` (visor de trabajo), `#adder` (agregar trabajo) |
| `<script>` | Datos editables → helpers → carrusel → precios → contacto → portfolio |

## Cómo correrlo en local

No necesita instalación. Abrí `portfolio.html` en el navegador (doble click).

Si preferís un servidor local:

```bash
# Python
python3 -m http.server 8000

# o Node
npx serve .
```

Luego entrá a `http://localhost:8000/portfolio.html`.

> Nota: el reproductor de YouTube (`iframe`) y las Google Fonts requieren conexión a internet.

---

## Qué editar (datos)

Todo lo que cambia seguido está agrupado al inicio del `<script>`, bajo el comentario `EDITÁ ESTOS DATOS`.

### `WA` — WhatsApp

```js
var WA = "5491100000000"; // código de país + área + número, SIN "+" ni espacios
```

También cambiá el mail en la sección `#contacto` (hay un `mailto:` y un texto visible).

### `PROMOS` — slides del carrusel

```js
{
  tag: "El combo más elegido",   // etiqueta chica arriba del título
  t:   "12 reels con 25% de descuento.", // título
  p:   "Texto descriptivo…",     // párrafo
  big: "25% OFF",                // texto grande dentro del arco
  bs:  "Ahorrás $60.000",        // texto debajo
  s:   "Edición de video",       // servicio que se preselecciona en el formulario
  b:   "Quiero este combo"       // texto del botón
}
```

> `s` debe coincidir con alguna `<option>` del `<select id="cs">`.

### `SEED` — trabajos del portfolio (visibles para todos)

```js
{ t: "Título", c: "Video", u: "https://…", cover: "" }
```

- `c`: `"Video"` | `"Web"` | `"App"`.
- `u`: link al trabajo. Si es de YouTube se reproduce dentro de la web.
- `cover`: imagen en base64 (`data:image/jpeg;base64,…`) o `""` para usar un degradé según la plataforma.

### `ITEMS` — precios y modales

```js
{
  g:  "Edición de video",        // grupo (debe existir un contenedor id="p-Edición de video")
  n:  "Combo x12",               // nombre
  pr: "$180.000",                // precio mostrado
  sv: "$15.000 c/u",             // subtítulo
  hot:"EL MÁS ELEGIDO",          // (opcional) destaca la tarjeta
  r:  "Resumen corto de la tarjeta",
  d:  "Descripción larga del modal",
  inc:  ["Qué incluye 1", "Qué incluye 2"],   // [] oculta la sección
  cond: ["Condición 1"],                       // [] oculta la sección
  s:  "Edición de video"         // servicio que se preselecciona en el formulario
}
```

Grupos actuales: `Edición de video`, `Páginas web`, `Apps móviles`.

### Otros textos estáticos

Se editan directamente en el HTML: *Quién soy*, *Cómo contratar*, la nota de precios de referencia y el footer.

---

## Cómo funciona el código

### Renderizado desde datos
Los slides, las tarjetas de precios y la grilla de trabajos **no están escritos en el HTML**: se generan en JS a partir de `PROMOS`, `ITEMS` y `SEED`. Para cambiar contenido, tocá los arrays, no el HTML.

### Carrusel
`.track` es una fila flex y cada `.slide` ocupa el 100% del ancho. `go(i)` desplaza el track con `translateX(-i*100%)` y marca el punto activo con `aria-current`. `auto()` reinicia el autoplay (6 s) y no corre si el usuario prefiere menos movimiento.

### Precios + modal
Cada item de `ITEMS` crea una tarjeta en su grupo. El botón "Más info" llama a `openInfo(i)`, que llena `#info` y lo abre con `showModal()`. El botón "Consultar" cierra el modal y llama a `pick(servicio)`, que preselecciona el `<select>` del formulario y salta a `#contacto`.

### Portfolio
- `all()` une `SEED` (fijos) con `extra` (guardados en `localStorage`).
- `render()` filtra por categoría y dibuja la grilla.
- `show(x)` abre el visor: YouTube → `iframe`; con portada → imagen; si no → aviso + botón a la plataforma.
- `plat(url)` detecta la plataforma (TikTok / Instagram / YouTube / Link) y `ytId(url)` extrae el ID de YouTube.

### Modo edición
El botón "Editar portfolio" activa el modo (aparece la **×** en los trabajos agregados) y luego abre `#adder`. Las portadas pasan por `shrink()`: se redimensionan a un máximo de 700 px con `<canvas>` y se guardan como JPEG base64 para no llenar `localStorage`.

### Formulario de contacto
No hay backend. `onsubmit` arma un texto y abre `https://wa.me/<WA>?text=<mensaje>` (con `encodeURIComponent`).

### Seguridad
Todo texto dinámico que entra por `innerHTML` pasa por `esc()` para evitar inyección de HTML. Los links externos usan `rel="noopener"`.

---

## Videos: embeds y archivos propios

El visor (`show()` + `embedInfo()`) decide cómo mostrar cada link:

| `u` en `SEED` | Se muestra |
|---|---|
| `https://www.tiktok.com/@user/video/123…` | iframe oficial de TikTok, en marco de celular |
| `https://www.instagram.com/reel/XXXX/` (o `/p/`, `/tv/`) | iframe de Instagram, en marco de celular |
| `https://www.youtube.com/shorts/ID` | iframe de YouTube, en marco de celular |
| `https://www.youtube.com/watch?v=ID` | iframe de YouTube, 16:9 |
| `videos/reel1.mp4` (o `.webm`, `.mov`) | `<video>` propio; el marco (celular o 16:9) se elige según el tamaño real del video |

Campo opcional `o` para forzar orientación: `"v"` (celular) o `"h"` (horizontal).

### Portadas

La forma de la tarjeta sigue la orientación del trabajo (`orient()`):

| Trabajo | Portada |
|---|---|
| YouTube normal, webs | horizontal 16:9 |
| TikTok, Instagram, YouTube Shorts | vertical 9:16 |
| Video propio (`.mp4`) | se mide el video: vertical u horizontal |

Se puede forzar con `o: "v"` o `o: "h"`. La imagen de portada va en `cover` con ruta relativa y barras `/` (ej. `"videos/s.png"`). Usá imágenes 1280×720 (horizontal) o 720×1280 (vertical).

### Carruseles de diseño

Categoría `Diseño` (el filtro aparece solo si hay al menos un trabajo). Se cargan en `SEED` con una lista de imágenes; la portada es la primera:

```js
{ t: "Carrusel para Cliente X", c: "Diseño", imgs: ["img/c1.jpg", "img/c2.jpg", "img/c3.jpg"], u: "" }
```

El visor las muestra en formato 4:5 con flechas y contador. `u` es opcional (link al post original).

### Modo edición (solo para vos)

El botón **Editar portfolio** está oculto para los visitantes. Para verlo, abrí la web con `#admin` al final (ej. `index.html#admin`). Lo que se agregue ahí queda solo en tu navegador; lo definitivo va en `SEED`.

### Tamaño de las tarjetas

Los trabajos horizontales (YouTube, webs) ocupan 2 columnas de la grilla (`.item.h`) y abren en un visor más ancho (`dialog.wide`). Los verticales y los carruseles mantienen su tarjeta angosta.

### Subir videos propios (pesados)

El HTML no puede contener videos grandes (y `localStorage` tiene ~5 MB), así que los archivos van **al lado del HTML** y se referencian por ruta:

```
.
├── index.html
└── videos/
    ├── reel1.mp4
    └── reel2.mp4
```

```js
{ t: "Mi reel", c: "Video", u: "videos/reel1.mp4", cover: "" }
```

La portada se genera sola con el primer cuadro del video. Para achicar los archivos (vertical 720×1280, ~5–15 MB):

```bash
ffmpeg -i original.mp4 -vf "scale=-2:1280" -c:v libx264 -crf 28 -preset slow \
       -c:a aac -b:a 96k -movflags +faststart reel1.mp4
```

`+faststart` permite que el video empiece a reproducirse antes de descargarse completo. Si son muchos o muy pesados, conviene un servicio de video (YouTube "no listado", Cloudinary, Bunny Stream, Vimeo) y usar el link/embed correspondiente.

## Diseño y branding

Las variables CSS están en `:root`:

| Variable | Valor | Uso |
|---|---|---|
| `--bg` | `#0b0909` | Fondo (negro tinta) |
| `--s1` / `--s2` | `#151111` / `#1f1819` | Tarjetas / modales |
| `--ink` | `#efe8dc` | Texto (crema papel) |
| `--mut` | `#b3a58f` | Texto secundario (beige cálido) |
| `--bor` | `#7a1f2b` | Botones (borgoña) |
| `--ver` | `#1f3a2d` | Tarjeta destacada (verde bosque) |
| `--oro` | `#c9a24d` | Acentos (dorado suave) |

**Tipografías** (Google Fonts):
- **Bodoni Moda**: títulos.
- **Jost**: texto.
- **Mrs Saint Delafield**: acentos manuscritos (footer).

Para cambiar la marca basta con modificar las variables y el `<link>` de Google Fonts del `<head>`.

## Animaciones y efectos

Todo en CSS + JS puro, sin librerías (bloque `EFECTOS / ANIMACIONES` del script):

- **Hero** con palabra rotativa (`words`), luces de fondo, línea de tiempo con cabezal y cinta de servicios (`MQ`).
- **Reveal al scrollear** (`reveal()` + `IntersectionObserver`), escalonado.
- **Barra de progreso**, navbar que se compacta y link de sección activa (scrollspy).
- **Tarjetas**: foco de luz que sigue al mouse (`.spot`), elevación, zoom y botón ▶ en videos.
- Todo se desactiva con `prefers-reduced-motion`.

## Accesibilidad

- Modales con `<dialog>` nativo (foco y `Esc` resueltos por el navegador).
- Foco visible en todos los controles (`:focus-visible`).
- Botones y chips con `aria-label` / `aria-pressed` / `aria-current` donde corresponde.
- Carrusel con `role="region"` y `aria-roledescription`.
- Se respeta `prefers-reduced-motion` (CSS y autoplay).
- Un único `<h1>` (el del primer slide).

## Limitaciones conocidas

- **Los trabajos agregados con el modo edición se guardan solo en el navegador** (`localStorage`). Para que los vea todo el mundo, hay que sumarlos a `SEED`.
- **Los embeds de TikTok / Instagram / YouTube necesitan que la página esté en un hosting propio** (Netlify, Vercel, GitHub Pages…). En vistas previas con restricciones de seguridad (CSP) pueden no cargar. Solo funcionan con publicaciones **públicas**; para TikTok hay que usar el link completo (`…/video/123456…`), no el corto `vm.tiktok.com`.
- `localStorage` tiene un límite (~5 MB): por eso se comprimen las portadas.
- `<dialog>.showModal()` requiere navegadores modernos (todos los actuales lo soportan).
- Los precios son texto fijo: no hay CMS ni panel de administración.

## Deploy

Es un sitio estático: sirve cualquier hosting de archivos.

1. **Netlify Drop**: arrastrá la carpeta a [app.netlify.com/drop](https://app.netlify.com/drop).
2. **Vercel** o **GitHub Pages**: subí el repo; renombrá `portfolio.html` a `index.html` para que sea la página inicial.
3. **Dominio propio**: agregalo desde el panel del hosting y apuntá los DNS según sus instrucciones.

Antes de publicar:
- [ ] Reemplazar el número de `WA` y el mail.
- [ ] Cargar los trabajos reales en `SEED`.
- [ ] Completar *Quién soy* y las condiciones de edición en `ITEMS`.
- [ ] Revisar los precios (están en pesos: conviene actualizarlos periódicamente).
- [ ] Agregar `<meta name="description">` y un `og:image` para compartir en redes.

## Pendientes (TODO)

- [ ] Datos reales de contacto (WhatsApp y mail).
- [ ] Texto personal de *Quién soy*.
- [ ] Condiciones de edición (revisiones, plazos, formatos).
- [ ] Trabajos reales con portadas.
- [ ] Favicon propio y metadatos SEO / Open Graph.
- [ ] Opcional: mover los datos a un `data.json` si el sitio crece.
