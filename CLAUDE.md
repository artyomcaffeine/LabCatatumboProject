# CLAUDE.md

Sitio Astro estático y multilingüe para turistas que visitan el Lago de Maracaibo para ver el Relámpago del Catatumbo. Idiomas: es (por defecto), en, de, fr. El diseño viene del sistema de diseño Catatumbo Field Guide (copiado del proyecto anterior `ClaudeCatatumboAstro`).

En esta fase el mismo sitio se empaqueta además como app para Android e iOS.

## Alcance del proyecto (fase actual: app)

- Un mismo código y un mismo contenido sirven para la web y para la app. La web sigue siendo un sitio Astro estático (`output: 'static'`, sin SSR ni adaptadores) y se sigue publicando en GitHub Pages. Ningún cambio hecho para la app puede romper la web.
- La app empaqueta el build estático (`dist/`). La herramienta nativa la elige quien desarrolla la app. Se recomienda Capacitor, porque usa `dist/` tal cual como `webDir`. Cuando se elija, anótala aquí.
- Idiomas: es, en, de, fr, con el enrutado i18n que trae Astro. Cada idioma tiene su prefijo (`/es/`, `/en/`, …).
- El contenido está en Markdown, en `src/content/pages/<lang>/`.
- **Entra en esta fase:** empaquetar la app para Android e iOS; que la app funcione sin internet, con todo el contenido, las fuentes y las imágenes dentro de la app (la web no necesita service worker); iconos, pantalla de inicio (splash) y preparación para las tiendas.
- **No entra (fases futuras, no las prepares):** base de datos, CMS, cuentas/login, reservas, clima, GPS, mapas, notificaciones.
- Dependencias: se pueden añadir las que hagan falta, incluidos frameworks de UI. Explica en el cambio por qué se añade cada una.
- Nada de embeds de terceros esenciales (mapas, vídeos, widgets): la app tiene que funcionar sin internet.
- Prefiere pocos archivos. Explica cada cambio en lenguaje sencillo antes de hacerlo. Trabaja en pasos pequeños.

## Comandos

```sh
npm run dev       # http://localhost:4321
npm run build     # sitio estático en dist/
npm run preview   # sirve dist/
npm run check     # revisa los tipos de .astro y .ts
```

No hay tests. Para verificar: `npm run check` y un build. Mira el resultado en un navegador.
- Mantén `typescript` en `^6`, porque `@astrojs/check` 0.9.x no es compatible con TS 7.
- Cuando los lanza un agente, `astro dev` / `astro preview` quedan en segundo plano. Se detienen con `npx astro dev stop` / `npx astro preview stop`.

## La app (notas técnicas)

- Para la app, compila con `npm run build` **sin** `PUBLIC_DEPLOY_TARGET`. Así la base es `/`. La variable `pages` es solo para la vista previa en GitHub Pages.
- Las URLs tienen forma de carpeta (`trailingSlash: 'always'`): `/es/relampago/` es `es/relampago/index.html`. Comprueba pronto que la navegación funciona en el WebView de Android y de iOS.
- La raíz (`src/pages/index.astro`) redirige con JavaScript al idioma guardado en `localStorage` (`ct.lang`) o, si no hay, al del teléfono.
- Los enlaces externos (créditos de las fotos) usan `target="_blank"`. Hay que decidir si se abren en el navegador del sistema.
- Las fotos son CC BY-SA 2.0: su crédito tiene que seguir visible en la app.
- Cuando se añadan las carpetas nativas y la configuración (p. ej. `android/`, `ios/`, `capacitor.config.*`), descríbelas aquí junto con los comandos para compilar la app.

## Cómo encaja todo

- `src/content.config.ts` define la colección `pages`. Cada archivo Markdown tiene `key` (igual en todos los idiomas; vincula las traducciones), `title`, `description`, `order` (posición en el menú) y, opcionalmente, `photo`/`photoAlt` y `draft_translation`. El nombre del archivo es la URL: `es/relampago.md` → `/es/relampago/`.
- `src/pages/[lang]/[...slug].astro` renderiza todas las páginas Markdown. `src/pages/[lang]/index.astro` es la página de inicio. `src/pages/index.astro` (la raíz del sitio) redirige al idioma del visitante y, sin JavaScript, muestra un selector de idioma.
- `src/i18n/ui.ts` contiene los textos de la interfaz (menú, botones, página de inicio) y la lista de idiomas (`LANGS`). Cada idioma se tipa contra `es`, así que una clave que falte es un error de tipos. Mantén todos los idiomas sincronizados.
- `src/lib/pages.ts` contiene las funciones para encontrar páginas y construir URLs.
- `src/layouts/Base.astro` contiene el `<head>`, la barra superior y el pie. `src/components/` contiene las piezas.

## Enlaces y ruta base

El mismo código se compila para la vista previa en GitHub Pages (servida bajo `/<repo>/`) y para la raíz (`/`, el hosting final o la app). Por eso:
- Enlaza las páginas con `pageUrl()`, `urlOf()` y `homeUrl()` de `src/lib/pages.ts`. Usan `getRelativeLocaleUrl`. Nunca escribas `/es/…` a mano.
- Enlaza los archivos de `public/` con `asset('img/…')`.
- `PUBLIC_DEPLOY_TARGET=pages` (lo pone `.github/workflows/deploy.yml`) cambia `site`/`base` en `astro.config.mjs` y añade `noindex`.

## Añadir cosas

- **Una página:** añade un archivo Markdown por idioma con la misma `key`, y añade la clave al `z.enum` de `content.config.ts`. Después dale un color de tarjeta y un icono en `src/pages/[lang]/index.astro` (`CARD`).
- **Un idioma:** añádelo a `LANGS`/`LANG_NAMES` en `ui.ts` y a `locales` en `astro.config.mjs`, añade su bloque en `ui.ts` y una carpeta con sus páginas Markdown.
- de y fr son traducciones en borrador (`draft_translation: true`) hasta que las revise un hablante nativo.

## Estilos

Los estilos se cargan en este orden: `tokens.css`, después `bundle.css`, después `site.css`.
- `tokens.css` contiene los tokens de diseño y las clases tipográficas (`display-*`, `copy-*`, `mono-*`). No lo edites a mano.
- `bundle.css` contiene los estilos de los componentes `ct-*` del sistema de diseño. No lo cambies.
- `site.css` es la capa de la página. Todo el CSS propio del sitio va aquí, incluidos los estilos `.prose` del Markdown.
- Las esquinas rectas en todas partes son parte del diseño. Las franjas oscuras usan `data-theme="night"`.
