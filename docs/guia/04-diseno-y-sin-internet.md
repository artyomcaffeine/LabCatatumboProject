# 4 · Diseño y sin internet

## 4.1 · Tres capas de CSS

Los estilos se cargan siempre en este orden (`Base.astro`):

| Archivo | Qué contiene | ¿Se edita? |
|---|---|---|
| `src/styles/tokens.css` | Variables de diseño (colores, tamaños) y clases tipográficas (`display-*`, `copy-*`, `mono-*`), y las `@font-face`. | No a mano: viene del sistema de diseño. |
| `src/styles/bundle.css` | Los componentes del sistema de diseño (`ct-topbar`, `ct-place`, `ct-footer`…). | No. |
| `src/styles/site.css` | Todo lo propio del sitio: ajustes, estilos del Markdown (`.prose`), piezas nuevas. | Sí: aquí va todo lo nuevo. |

Ventaja: si el sistema de diseño se actualiza, se sustituyen `tokens.css` y `bundle.css` sin perder los cambios del sitio. Como `site.css` va el último, gana cuando la especificidad es igual. **Ojo:** una regla de `site.css` fuera de un `@media` también pisa la regla de móvil de `bundle.css`; si tocas `height` o `padding` de un componente, repite el ajuste dentro del mismo `@media`.

El modo oscuro lo pone el sistema (`prefers-color-scheme` en `tokens.css`), y las franjas oscuras usan `data-theme="night"`.

## 4.2 · Nada que venga de internet

Para que la app funcione sin conexión, cada página tiene que poder dibujarse solo con los archivos de `dist/`:

- **Fuentes:** los `.woff2` están en `src/assets/fonts/` y se declaran en `tokens.css`. Nunca enlaces Google Fonts ni otra CDN.
- **Imágenes:** en `src/assets/img/`, mostradas con `<Image>` o `<Picture>` de `astro:assets`, que genera tamaños y formatos (webp, avif). Lo que no necesita optimizarse (iconos, banderas SVG) va en `public/` y se enlaza con `asset()`.
- **Sin embeds:** ni mapas, ni vídeos de YouTube, ni widgets. Un enlace a otra web (un crédito, una fuente) sí vale.
- **Créditos visibles:** las fotos del ejemplo son CC BY-SA 2.0. Su crédito aparece bajo cada foto y en el pie, también en la app.

Cómo comprobarlo: `npm run build && npm run preview`, abre las herramientas del navegador, pestaña **Red** (Network), y recorre las páginas. Todas las peticiones tienen que ir a `localhost`; cualquier otro dominio es algo que fallará sin conexión.

## 4.3 · Pantallas con muesca

En el móvil, la app ocupa toda la pantalla, también bajo la hora y la batería. Por eso:

```html
<meta name="viewport" content="width=device-width, initial-scale=1, viewport-fit=cover">
```

y en `site.css` la barra superior y el pie suman el hueco del sistema:

```css
.ct-topbar { height: calc(var(--topbar-height) + env(safe-area-inset-top, 0px)); padding-top: env(safe-area-inset-top, 0px); }
```

En un navegador de escritorio `env(safe-area-inset-top)` vale 0 y nada cambia. Así la franja de la hora toma el color de la barra superior, en modo claro y oscuro.

## Errores comunes

- Copiar un `<link>` de Google Fonts "porque es más rápido": la app sin conexión pierde las fuentes.
- Poner imágenes grandes en `public/`: no se optimizan. Si es una foto, va en `src/assets/`.
- Editar `bundle.css` para un ajuste: se perderá en la próxima actualización del sistema de diseño.

## Ejercicio

Añade una fuente local y una foto optimizada a tu proyecto. Después busca en `dist/` recursos que se carguen de otra web (los enlaces `<a href>` no cuentan).

**Comprobación:** tras `npm run build`, estos dos comandos no muestran nada, y en la pestaña Red solo aparece `localhost`:

```sh
grep -rhoE '<(link|script|img|source|iframe)[^>]*(src|href)="https?://[^"]*' dist
grep -rhoE 'url\(https?://[^)]*' dist --include=*.css
```

(Hazlo con `npm run build` normal: el build de GitHub Pages añade a propósito enlaces `hreflang` absolutos, que no son recursos.)
