---
title: 'Cómo construir tu blog desde 0, con Astro.'
description: 'Todo el camino para armar un blog propio con Markdown, Astro y hosting gratuito — desde elegir el stack hasta darle identidad visual.'
pubDate: '2026-07-21'
heroImage: '../../assets/posts/2.png'
---

## Introducción

Durante mucho tiempo pensé en escribir sobre lo que voy aprendiendo, pero siempre terminaba desechando la idea. La opción fácil hubiera sido abrir una cuenta en Medium o Substack y listo — pero cuanto más lo pensaba, menos sentido le encontraba. Quería algo que fuera realmente **mío**: un repositorio que pudiera mostrar en mi portafolio, con código que yo entendiera de punta a punta, sin depender de una plataforma de terceros que decide el diseño, el dominio, o si un día decide cobrar por lo que hoy es gratis.

Así que en vez de buscar la opción más rápida, decidí construir mi propio blog desde cero: Markdown para el contenido, un generador de sitios estáticos para convertirlo en HTML, y hosting gratuito para publicarlo. En este post cuento el camino completo — desde por qué elegí las herramientas que elegí, hasta cómo terminó desplegado y con identidad visual propia. La idea es que sirva tanto de bitácora personal como de guía para cualquiera que quiera hacer lo mismo.

## Eligiendo el stack: generador estático + Markdown

Lo primero que tuve que decidir fue **cómo** iba a construir el sitio. Existen básicamente dos caminos: un CMS tradicional (WordPress, Ghost) con base de datos y panel de administración, o un **generador de sitios estáticos**, donde escribís el contenido en archivos de texto (Markdown) y una herramienta los convierte en HTML plano en el momento del build.

Para un blog personal, la segunda opción tiene sentido casi siempre: no hay base de datos que mantener, no hay superficie de ataque de un panel de admin expuesto a internet, y el resultado final son archivos estáticos que cualquier hosting gratuito puede servir sin esfuerzo.

Dentro de los generadores estáticos, dudé entre dos: **Hugo** y **Astro**. Hugo está escrito en Go y es, sin exagerar, el más rápido a la hora de compilar sitios grandes — pero sus plantillas usan la sintaxis de templates de Go, que no se parece a nada que fuera a usar en otro lado. Astro, en cambio, se escribe en JavaScript/TypeScript, y aunque su sintaxis de componentes es propia, se siente mucho más cercana a lo que ya sabía (y a lo que probablemente use en el futuro). Terminé eligiendo Astro por eso: no solo iba a construir un blog, sino a practicar con una herramienta cuyo lenguaje de base me sirve para más cosas.

Lo que más me convenció de Astro, más allá del lenguaje, es su **arquitectura de islas** (*islands architecture*). La idea central es simple pero poderosa: por defecto, Astro renderiza todo como HTML estático — sin JavaScript enviado al navegador. Si una parte puntual de la página necesita interactividad (un componente de React, un buscador con fuse.js, un contador), esa parte se convierte en una "isla": un fragmento aislado que sí carga su propio JavaScript, mientras el resto de la página permanece como HTML puro.

Esto es lo opuesto a cómo funcionan frameworks como Next.js o una SPA pura de React, donde por defecto se envía el runtime completo del framework al navegador, así la página tenga una sola línea interactiva. En un blog, donde el 95% del contenido es texto que no necesita interactividad, esa diferencia es enorme: mi sitio carga rápido porque **no** está mandando kilobytes de JavaScript innecesario solo para mostrar un párrafo.

Por último, la decisión de usar **Markdown** para el contenido fue casi automática una vez elegido el generador estático: es el formato natural que Astro espera para posts de blog, es liviano, se versiona perfecto en Git (a diferencia de un binario de Word), y cualquier editor de texto — sin instalar nada especial — sirve para escribir un post nuevo.

## Setup inicial del proyecto

Con el stack decidido, el primer paso real es instalar [Node.js](https://nodejs.org) (versión 22 o superior, que es lo que pide Astro actualmente) y correr un único comando:

```bash
npm create astro@latest mi-blog
```

La CLI hace preguntas interactivas: elegí la plantilla **"Use blog template"**, que ya viene con un primer post de ejemplo, un layout para posts individuales, y la configuración de colecciones de contenido lista para usar. Aceptá instalar las dependencias e inicializar un repositorio Git cuando te lo pregunte.

Una vez terminado, `cd mi-blog` y `npm run dev` levantan el sitio en `http://localhost:4321`. Pero antes de tocar nada, vale la pena entender **qué generó exactamente** ese comando, porque casi todo el trabajo de personalización que viene después pasa por saber dónde vive cada cosa.

### Recorrido por la estructura de carpetas

```text
mi-blog/
├── public/
│   └── favicon.svg
├── src/
│   ├── assets/
│   ├── components/
│   │   ├── BaseHead.astro
│   │   ├── Header.astro
│   │   ├── Footer.astro
│   │   ├── HeaderLink.astro
│   │   └── FormattedDate.astro
│   ├── content/
│   │   └── blog/
│   │       └── primer-post.md
│   ├── content.config.ts
│   ├── layouts/
│   │   └── BlogPost.astro
│   ├── pages/
│   │   ├── index.astro
│   │   ├── about.astro
│   │   ├── rss.xml.js
│   │   └── blog/
│   │       ├── index.astro
│   │       └── [...slug].astro
│   ├── styles/
│   │   └── global.css
│   └── consts.ts
├── astro.config.mjs
├── package.json
└── tsconfig.json
```

**`public/`** — todo lo que va acá se copia tal cual al build final, sin procesar. Es el lugar para el favicon, imágenes que no necesitan optimización, o archivos como `robots.txt`.

**`src/assets/`** — a diferencia de `public/`, las imágenes que pongas acá **sí** pasan por el pipeline de optimización de Astro (compresión, generación de distintos tamaños) cuando las importás con el componente `<Image />`. Ahí guardo las portadas de los posts.

**`src/components/`** — piezas de UI reutilizables. `Header.astro` y `Footer.astro` son los más obvios, pero también viven acá cosas menos evidentes como `BaseHead.astro` (todo lo que va dentro del `<head>`: metadata, Open Graph, favicon, fuentes) y `HeaderLink.astro` (un link de navegación que sabe detectar si está "activo" según la ruta actual).

**`src/content/blog/`** — acá van los posts, un archivo `.md` por cada uno. El nombre del archivo se convierte en el slug de la URL (`hello-world.md` → `/blog/hello-world/`).

**`src/content.config.ts`** — define el **schema** de la colección de posts: qué campos son obligatorios en el frontmatter (`title`, `description`, `pubDate`) y cuáles opcionales (`heroImage`, `updatedDate`). Astro valida cada post contra este schema al hacer build, así que si te olvidás un campo requerido, te avisa antes de que el error llegue a producción.

**`src/layouts/BlogPost.astro`** — la plantilla que envuelve a **cada** post individual: título, fecha, imagen destacada, y el `<slot />` donde se inyecta el contenido del Markdown ya convertido a HTML.

**`src/pages/`** — acá es donde Astro define las **rutas** del sitio. Cada archivo se convierte en una URL: `index.astro` es la home, `about.astro` es `/about`. La carpeta `blog/` tiene dos piezas clave: `index.astro` (el listado de todos los posts) y `[...slug].astro` (una ruta dinámica que genera una página por cada post de la colección, usando `getStaticPaths()`).

**`src/styles/global.css`** — variables CSS y estilos base compartidos por todo el sitio.

**`src/consts.ts`** — un archivo chico pero importante: ahí vive el título y la descripción del sitio, para no tener ese texto repetido (y potencialmente desincronizado) en cada componente.

**`astro.config.mjs`** — configuración general del proyecto. Acá se define, entre otras cosas, el dominio (`site`) y, si el sitio no vive en la raíz del dominio, el `base` — un detalle que se vuelve crítico más adelante, cuando lleguemos al deploy.

## Escribiendo el primer post

Con la estructura clara, escribir un post es simplemente crear un archivo nuevo dentro de `src/content/blog/`. Por ejemplo, `hola-mundo.md`:

```markdown
---
title: 'Hola, este es mi blog'
description: 'Primera entrada de mi blog — quién soy y de qué voy a escribir.'
pubDate: '2026-07-15'
---

Este es el contenido del post, escrito en **Markdown** normal. Puedo usar
listas, código, imágenes, y todo lo que ya conozco del formato.
```

Las tres líneas entre `---` son el **frontmatter**: metadata en formato YAML que Astro lee antes de renderizar el contenido. Como vimos en `content.config.ts`, ese schema exige que `title`, `description` y `pubDate` estén presentes — si te olvidás alguno, Astro te lo va a marcar como error al correr `npm run dev` o `npm run build`, antes de que el problema llegue a producción.

Dos campos merecen atención aparte:

- **`description`** no es solo decorativo: es lo que se usa como meta description en el `<head>` (relevante para SEO) y como preview cuando alguien comparte el link en redes o apps de mensajería. Vale la pena escribir una específica para cada post, no una genérica repetida.
- **`pubDate`** se declara como string (`'2026-07-15'`), pero el schema lo transforma automáticamente a un objeto `Date` de JavaScript (`z.coerce.date()`), lo cual permite después ordenar los posts cronológicamente sin parsear nada a mano.

El resto del archivo, debajo del segundo `---`, es Markdown puro: encabezados con `#`, listas, bloques de código con triple backtick, imágenes con `![alt](ruta)`. Astro lo convierte a HTML automáticamente y lo inyecta en el `<slot />` del layout `BlogPost.astro` que vimos antes.

Con eso guardado, el post aparece solo en el listado del blog — no hace falta registrarlo en ningún otro lado ni tocar código de rutas, porque `[...slug].astro` genera la página automáticamente a partir de todo lo que encuentra en la colección.

## Deploy: de localhost a internet

Con el sitio funcionando en local, el siguiente paso es publicarlo. Elegí **GitHub Pages** por una razón simple: el código ya vive en GitHub (porque lo quiero mostrar como parte de mi portafolio), así que desplegar ahí mismo evita depender de un servicio externo adicional, y el deploy queda atado al mismo repositorio que un reclutador podría revisar.

### Configurar `site` y `base`

Lo primero es decirle a Astro dónde va a vivir el sitio. En `astro.config.mjs`:

```js
export default defineConfig({
	site: 'https://tu-usuario.github.io',
	base: '/nombre-del-repo',
	// ...el resto de tu configuración
});
```

`site` es el dominio raíz. `base` es necesario porque GitHub Pages no sirve tu sitio en la raíz del dominio, sino en un subdirectorio con el nombre del repositorio (`tu-usuario.github.io/nombre-del-repo`), a menos que el repo se llame exactamente `tu-usuario.github.io`.

Este detalle, que parece menor, es en realidad el punto donde más fácil se rompe un blog recién desplegado. Cualquier link interno que hayas escrito como ruta absoluta (`href="/blog"`, `href="/about"`) va a apuntar a `tu-usuario.github.io/blog` en vez de `tu-usuario.github.io/nombre-del-repo/blog` — es decir, un 404. Esto afecta más lugares de los que uno imagina al principio: los links del menú de navegación, los links a cada post desde el listado, el `href` del favicon, el `link` del feed RSS, incluso el link del logo/nombre del sitio en el header.

La solución es no hardcodear nunca una ruta absoluta y en cambio construirla siempre a partir de la variable que Astro expone para esto:

```astro
<a href={`${import.meta.env.BASE_URL}blog`}>Blog</a>
```

`import.meta.env.BASE_URL` resuelve automáticamente al valor que configuraste en `base`, así que si el día de mañana cambiás de hosting o de nombre de repo, no hay que salir a buscar y reemplazar rutas hardcodeadas por todo el proyecto — el sitio entero sigue funcionando porque todos los links se calculan relativos a esa única fuente de verdad. Vale la pena adoptar este patrón desde el primer componente que escribas, no como parche después de encontrar un link roto.

### Deploy automático con GitHub Actions

Para que cada `push` a `main` publique el sitio solo, sin pasos manuales, Astro se integra con GitHub Actions mediante una acción oficial. El archivo va en `.github/workflows/deploy.yml`:

```yaml
name: Deploy to GitHub Pages

on:
  push:
    branches: [ main ]
  workflow_dispatch:

permissions:
  contents: read
  pages: write
  id-token: write

concurrency:
  group: "pages"
  cancel-in-progress: false

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
      - name: Checkout
        uses: actions/checkout@v4
      - name: Install, build, and upload your site
        uses: withastro/action@v3
        with:
          node-version: 22

  deploy:
    needs: build
    runs-on: ubuntu-latest
    environment:
      name: github-pages
      url: ${{ steps.deployment.outputs.page_url }}
    steps:
      - name: Deploy to GitHub Pages
        id: deployment
        uses: actions/deploy-pages@v4
```

Especificar `node-version: 22` es importante: es la versión mínima que pide Astro actualmente, y el runner de GitHub Actions no siempre usa por defecto la más reciente.

El último paso es puramente de configuración en la interfaz de GitHub: en el repositorio, **Settings → Pages**, y cambiar la fuente ("Build and deployment → Source") de "Deploy from a branch" a **"GitHub Actions"**. A partir de ahí, cada `push` a `main` dispara el workflow, y en unos minutos el sitio queda publicado en `https://tu-usuario.github.io/nombre-del-repo`.

## Dándole identidad propia

Con el sitio ya funcionando, lo que quedaba era la parte más entretenida: que dejara de verse como la plantilla default de Astro. La plantilla de blog viene bien resuelta técnicamente, pero visualmente es genérica a propósito — está pensada como punto de partida, no como producto terminado.

### Paleta y tipografía como decisiones, no solo estética

Antes de tocar una sola línea de CSS, definí qué quería transmitir: un blog técnico pero con algo de personalidad, sin caer en lo genérico de "fondo blanco, Helvetica, azul de link default". Terminé en una paleta con **azul marino como color de acento** — links, hover, el borde del ítem activo en el menú — combinada con tipografía serif (Lora) para los títulos, que sí mantienen el color de texto principal (blanco sobre fondo oscuro, o casi negro sobre fondo claro) para priorizar la legibilidad. El acento aparece cuando hay algo interactivo o para destacar un elemento puntual, no como color de fondo de todo el texto grande — un poco al estilo de cómo se usa el color en diarios digitales serios, donde el acento marca "esto se puede clickear" sin gritar en cada título.

Estas decisiones viven como variables CSS en un solo lugar (`global.css`), y todo el resto de los componentes las consume por nombre en vez de tener colores hardcodeados repetidos:

```css
:root {
	--accent: #14213d;
	--bg: #ffffff;
	--text: 26, 26, 26;
}
```

Esto importa más de lo que parece al principio: cambiar el acento del sitio entero es editar un solo valor, no salir a buscar el mismo hex color pegado en diez archivos distintos.

### Tema claro/oscuro con `localStorage`

Quería que el sitio soportara modo oscuro, y que recordara la preferencia del visitante entre visitas — sin necesitar ninguna librería para algo tan simple. La solución completa son un atributo HTML, unas variables CSS condicionales, y un puñado de líneas de JavaScript.

Primero, el CSS define dos sets de variables, uno por tema, seleccionados por un atributo `data-theme` en el `<html>`:

```css
[data-theme='light'] {
	--accent: #14213d;
	--bg: #ffffff;
}
[data-theme='dark'] {
	--accent: #7aa3e0;
	--bg: #121212;
}
```

Después, un script chico se encarga de aplicar el tema guardado (o la preferencia del sistema operativo, si es la primera visita) **antes** de que la página termine de pintar, para evitar el parpadeo de "carga en un tema y salta al otro":

```js
const saved = localStorage.getItem('theme');
const theme = saved || (matchMedia('(prefers-color-scheme: dark)').matches ? 'dark' : 'light');
document.documentElement.setAttribute('data-theme', theme);
```

Y un botón en el header simplemente alterna el atributo y persiste la elección:

```js
toggleBtn.addEventListener('click', () => {
	const next = document.documentElement.getAttribute('data-theme') === 'dark' ? 'light' : 'dark';
	document.documentElement.setAttribute('data-theme', next);
	localStorage.setItem('theme', next);
});
```

Es, en esencia, exactamente el mismo principio que la arquitectura de islas de Astro: la menor cantidad de JavaScript posible para resolver el problema puntual, sin arrastrar una librería completa de manejo de estado para algo que se resuelve con un atributo HTML y un `if`.

### Buscador con Fuse.js

El listado de posts en la home tiene un buscador que filtra en tiempo real a medida que se escribe, tolerando errores de tipeo — para eso usé [Fuse.js](https://www.fusejs.io/), una librería liviana de *fuzzy search* que corre enteramente en el cliente, sin backend ni API de por medio.

La idea es simple: se le pasa un array con los datos de cada post (título y descripción) y una configuración de qué tan estricta debe ser la coincidencia:

```js
const fuse = new Fuse(searchIndex, {
	keys: ['title', 'description'],
	threshold: 0.35,
});

input.addEventListener('input', (e) => {
	const results = fuse.search(e.target.value);
	// mostrar u ocultar posts según coincidan o no
});
```

Este es, justamente, un buen ejemplo de "isla" dentro de un sitio que por lo demás es HTML estático: el resto de la home no necesita ni una línea de JavaScript, pero el buscador sí — y esa necesidad queda contenida a ese único componente, sin afectar el peso ni la velocidad de carga del resto de la página.

Cada resultado de la lista usa el mismo color de acento que el resto del sitio para indicar interactividad: al pasar el mouse sobre un post, tanto el título como la fecha cambian a azul marino, la misma señal visual que un link normal. Es un detalle chico, pero mantiene consistente la idea de "el acento marca lo que se puede clickear" en todos los rincones del sitio, no solo en el texto dentro de un párrafo.

## Qué aprendí

Empecé este proyecto pensando que iba a ser, en el peor de los casos, una tarde de trabajo: instalar algo, escribir un post, subirlo. Terminó tocando bastante más de lo que esperaba — configuración de rutas, CI/CD, diseño de sistema con variables CSS, manejo de estado sin frameworks, optimización de assets. Un blog "simple" resultó ser un recorrido por buena parte de lo que hace falta para llevar cualquier proyecto de desarrollo desde una idea hasta algo público y funcionando.

Si algo me queda claro después de este camino es que vale la pena entender las piezas en vez de copiar una plantilla y no tocarla más. Cada decisión — Astro sobre Hugo, GitHub Pages sobre otras opciones, resolver rutas relativas al `base` desde el principio, JavaScript mínimo en vez de una librería para cada cosa — terminó siendo también una excusa para entender un poco mejor cómo funciona la web por debajo.

Lo que sigue para este blog es, sobre todo, más contenido: la idea siempre fue que sirviera de bitácora de lo que voy construyendo y aprendiendo como estudiante de Ciencias de la Computación, no que el blog en sí fuera el único proyecto del que hablar acá. En el camino, seguramente termine agregando algún detalle más — un dominio propio, quizás algo de analítica básica — pero eso ya es para otro post.

Si estás pensando en armar algo parecido, mi consejo es simple: arrancá, aunque no tengas todo resuelto de antemano. Yo tampoco lo tenía.

Que la Fuerza (del `git commit`) te acompañe.