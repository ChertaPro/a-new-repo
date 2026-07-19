# A New Repo

Blog personal construido con [Astro](https://astro.build), donde documento proyectos, herramientas que uso y mi camino como estudiante de Ciencias de la Computación.

🔗 **Sitio en vivo:** [chertapro.github.io/a-new-repo](https://chertapro.github.io/a-new-repo/)

## Sobre el proyecto

Este blog nació como un espacio propio para escribir sobre lo que voy aprendiendo: proyectos personales, herramientas de desarrollo, y reflexiones sobre la carrera. El contenido se escribe en Markdown y el sitio se genera de forma estática, sin backend ni base de datos.

## Stack técnico

- **[Astro](https://astro.build)** — generador de sitios estáticos, con contenido en Markdown/MDX
- **TypeScript** — tipado en componentes y configuración de colecciones de contenido
- **[Fuse.js](https://www.fusejs.io/)** — búsqueda difusa de posts por título y descripción, corriendo en el cliente
- **CSS puro con variables** — sin frameworks de estilos; tema claro/oscuro implementado con `data-theme` y `localStorage`
- **GitHub Actions** — build y deploy automático a GitHub Pages en cada push a `main`

## Funcionalidades

- 📝 Posts en Markdown con frontmatter (título, descripción, fecha)
- 🔍 Buscador en la home que filtra posts en tiempo real, tolerante a errores de tipeo
- 🌗 Selector de tema claro/oscuro, con persistencia entre visitas
- 📄 Página About con mi presentación
- 📡 Feed RSS generado automáticamente
- 🗺️ Sitemap generado automáticamente
- ♿ Atención a accesibilidad básica (`aria-label`, `prefers-reduced-motion`, texto alternativo)

## Estructura del proyecto

```text
/
├── public/                  # Assets estáticos (favicon, etc)
├── src/
│   ├── components/          # Header, Footer, BaseHead, HeaderLink, etc
│   ├── content/
│   │   └── blog/            # Posts en formato .md
│   ├── layouts/              # Layout de posts individuales
│   ├── pages/
│   │   ├── index.astro       # Home con buscador
│   │   ├── about.astro        # Página About
│   │   └── blog/               # Listado y rutas dinámicas de posts
│   ├── styles/
│   │   └── global.css         # Variables de tema y estilos base
│   └── consts.ts               # Título y descripción del sitio
├── astro.config.mjs
└── package.json
```

## Correr el proyecto localmente

Requiere [Node.js](https://nodejs.org) 22 o superior.

```bash
npm install
npm run dev
```

El sitio queda disponible en `http://localhost:4321`.

| Comando             | Acción                                              |
| :------------------ | :--------------------------------------------------- |
| `npm install`        | Instala las dependencias                              |
| `npm run dev`         | Levanta el servidor de desarrollo local               |
| `npm run build`       | Genera el sitio estático en `./dist/`                 |
| `npm run preview`      | Sirve la build de producción localmente para probarla |

## Despliegue

El sitio se despliega automáticamente a **GitHub Pages** mediante un workflow de GitHub Actions (`.github/workflows/deploy.yml`) que corre en cada push a la rama `main`.

## Autor

**Ramón Cherta González**
[GitHub](https://github.com/ChertaPro)