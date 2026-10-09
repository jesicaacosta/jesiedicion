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

## Accesibilidad

- Modales con `<dialog>` nativo (foco y `Esc` resueltos por el navegador).
- Foco visible en todos los controles (`:focus-visible`).
- Botones y chips con `aria-label` / `aria-pressed` / `aria-current` donde corresponde.
- Carrusel con `role="region"` y `aria-roledescription`.
- Se respeta `prefers-reduced-motion` (CSS y autoplay).
- Un único `<h1>` (el del primer slide).

## Limitaciones conocidas

- **Los trabajos agregados con el modo edición se guardan solo en el navegador** (`localStorage`). Para que los vea todo el mundo, hay que sumarlos a `SEED`.
- **TikTok e Instagram no se embeben**: sus reproductores requieren scripts propios. Se muestra un botón que abre la plataforma.
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
