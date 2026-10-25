# PLAN.md

Plan de trabajo vivo del proyecto. `CLAUDE.md` dice qué es el proyecto y qué entra en la fase actual. Este archivo dice qué falta por hacer y en qué orden.

## Cómo usar este archivo

- Marca cada paso como hecho (`[x]`) cuando esté en `main`.
- Cuando se tome una decisión, quítala de "Decisiones pendientes" y anótala con fecha en "Registro de decisiones".
- Si un paso cambia el alcance (por ejemplo, la Fase 3), actualiza también `CLAUDE.md`.
- Pasos pequeños: cada casilla debería caber en un cambio.

## Punto de partida

- El sitio ya funciona sin internet una vez descargado: las fuentes están en `src/assets/fonts/`, las fotos en `src/assets/img/` (optimizadas con `astro:assets`) y los iconos en `public/icons/`. No se carga nada de otros servidores.
- Lo único externo son enlaces: los créditos CC BY-SA de las fotos (`Hero.astro`, `Footer.astro`, `[...slug].astro`) y las fuentes al final de los Markdown.
- El único JavaScript de cliente es la redirección de idioma de `src/pages/index.astro` (usa `localStorage`, clave `ct.lang`).
- Las URLs tienen forma de carpeta (`trailingSlash: 'always'`): `/es/relampago/` es `es/relampago/index.html`.

---

## Fase 1 · App sin conexión (fase actual)

Objetivo: el mismo build estático (`dist/`) empaquetado como app de Android e iOS, con todo el contenido dentro. La web no cambia y no necesita service worker.

- [ ] **1. Elegir Capacitor** y anotarlo en `CLAUDE.md`. Dependencias (y por qué):
  - `@capacitor/core`, `@capacitor/cli`: el puente entre la web y la app nativa, y la herramienta de línea de comandos.
  - `@capacitor/android`, `@capacitor/ios`: generan los proyectos nativos `android/` e `ios/`.
  - `capacitor.config.ts` con `webDir: 'dist'`, más `appId` y `appName` (ver decisiones pendientes).
- [ ] **2. Scripts npm**: `app:build` (`astro build && cap sync`), `app:android` (`cap open android`), `app:ios` (`cap open ios`). Siempre **sin** `PUBLIC_DEPLOY_TARGET`, para que la base sea `/`.
- [ ] **3. Probar la navegación en el WebView, lo antes posible.** Riesgo conocido: el servidor local de Capacitor trata las rutas sin extensión (`/es/relampago/`) como una SPA y devuelve el `index.html` de la raíz. Eso rompería los enlaces entre páginas y podría crear un bucle con la redirección de idioma. Si falla:
  - Opción A: un build solo para la app con `build.format: 'file'` (`es/relampago.html`), activado con una variable (`PUBLIC_DEPLOY_TARGET=app`) en `astro.config.mjs`. El build web no cambia.
  - Opción B: que en la app los enlaces apunten a `…/index.html`.
  - Elegir tras la prueba y anotarlo en el registro.
- [ ] **4. Prueba sin conexión** (modo avión, en Android y en iOS): todas las páginas, los 4 idiomas, fuentes, fotos, selector de idioma, redirección de la raíz y que el idioma elegido se recuerda.
- [ ] **5. Enlaces externos** (`target="_blank"`): abrirlos en el navegador del sistema con `@capacitor/browser` (dependencia: abre el navegador del teléfono desde la app). Un script pequeño en `Base.astro` que solo actúa si `Capacitor.isNativePlatform()`. Sin conexión, mostrar un aviso corto en vez de fallar en silencio. Los créditos CC BY-SA siguen visibles en la app.
- [ ] **6. Configuración**: añadir `android` e `ios` al `exclude` de `tsconfig.json` (hoy incluye `**/*`). Las copias de `dist/` dentro de las carpetas nativas ya las ignoran los `.gitignore` que crea Capacitor.
- [ ] **7. Icono y splash**: imágenes fuente en `assets/` (icono 1024×1024, splash 2732×2732, versión clara y oscura) y generarlas con `@capacitor/assets` (dependencia de desarrollo: crea todos los tamaños para ambas tiendas). Barra de estado y fondo del splash con `#142d31` (el `theme-color` actual).
- [ ] **8. Preparación para las tiendas**:
  - Cuentas: Google Play Console (pago único) y Apple Developer Program (anual).
  - Android: keystore de firma fuera del repo, con copia de seguridad.
  - iOS: hace falta macOS con Xcode para compilar y subir.
  - Capturas de pantalla por idioma, textos de la ficha en es/en/de/fr (de y fr revisados por hablantes nativos antes de publicar).
  - Política de privacidad: página Markdown nueva en los 4 idiomas (las dos tiendas la piden). Formularios de privacidad de Apple y "Seguridad de los datos" de Google.
  - Ojo: Apple puede rechazar apps que solo envuelven una web (guía 4.2). El funcionamiento sin conexión ayuda; los widgets de la Fase 3 ayudarían más.
- [ ] **9. Documentar** en `CLAUDE.md` las carpetas `android/`, `ios/`, `capacitor.config.ts` y los comandos para compilar la app.
- [ ] **10. Comprobar que la web sigue igual**: `npm run check`, `npm run build` y un build con `PUBLIC_DEPLOY_TARGET=pages`.

---

## Fase 2 · Enlaces a tours de la zona

Encaja en la fase actual: son enlaces, no embeds de terceros.

- [ ] Nueva página por idioma con `key: tours` (por ejemplo `es/tours.md`, `en/tours.md`…). Añadir `'tours'` al `z.enum` de `src/content.config.ts` y su tarjeta (color e icono, p. ej. `boat`) en `CARD` de `src/pages/[lang]/index.astro`.
- [ ] Formato: empezar con una lista en Markdown (pocos archivos). Si la lista crece o se repite en varios sitios, pasarla a una colección de datos (`src/content/tours.yaml`: nombre, URL, idiomas, punto de salida, contacto) y renderizarla con un componente.
- [ ] Texto de aviso en cada idioma: no hay relación comercial (o sí, si se decide lo contrario), no se avala a los operadores, fecha de la última revisión de la lista.
- [ ] Criterios para incluir a un operador (local, con permisos del parque nacional, buenas prácticas con las comunidades). Anotarlos aquí.
- [ ] En la app los enlaces se abren en el navegador del sistema (Fase 1, paso 5). Sin conexión la lista se ve, pero los sitios de los operadores no.
- [ ] Revisar la lista periódicamente (enlaces rotos, operadores que cierran).

---

## Fase 3 · Widgets de tiempo, ubicación y rayos (fase futura)

Hoy `CLAUDE.md` deja clima y GPS fuera de alcance. Antes de empezar, actualizar el alcance en `CLAUDE.md`.

### Principios para todos los widgets

- El sitio sigue siendo estático (`output: 'static'`). Cada widget es un componente Astro con un `<script>` de cliente que pide los datos al abrir la página. No hace falta un framework de UI.
- Sin conexión: guardar el último dato en `localStorage` con su fecha y mostrar "sin conexión · dato del <fecha>". Nunca bloquear el contenido de la guía.
- Textos de los widgets en `src/i18n/ui.ts`, en los 4 idiomas.
- Atribución visible de cada fuente de datos (como se hace con las fotos).
- Revisar la política de privacidad y la ficha de las tiendas con cada widget.

### Tiempo

- Fuente propuesta: [Open-Meteo](https://open-meteo.com/): gratis, sin clave, permite llamadas desde el navegador (CORS), licencia CC BY 4.0 (exige atribución). El plan gratuito es para uso no comercial: revisar si la app será de pago o tendrá anuncios.
- Pronóstico de hasta 16 días para coordenadas fijas (Congo Mirador, Ologá, Puerto Concha: coordenadas por confirmar).
- Datos útiles: probabilidad de lluvia y de tormenta (códigos 95–99), nubosidad, viento, hora de la puesta de sol.

### Ubicación

- Web: `navigator.geolocation` (requiere HTTPS; GitHub Pages ya lo tiene).
- App: `@capacitor/geolocation` y permisos: `NSLocationWhenInUseUsageDescription` en iOS, `ACCESS_COARSE_LOCATION` en Android.
- Uso propuesto, sin mapas (siguen fuera de alcance): distancia y dirección hasta el embarcadero o la zona de observación, calculadas en el teléfono. Funciona sin conexión.
- No guardar ni enviar la ubicación a ningún sitio, y decirlo en la política de privacidad.

### Rayos: semana, mes, año

Nadie puede saber cuántos rayos *caerán*. Hay tres tipos de dato, y hay que decidir cuál se muestra (ver decisiones pendientes):

| Periodo | Qué se puede mostrar | Fuente | Sin conexión |
|---|---|---|---|
| Próxima semana | Probabilidad de tormenta por noche (no un conteo) | Pronóstico de Open-Meteo (códigos de tormenta, CAPE) | Último dato guardado |
| Mes / año | Promedio histórico de rayos y de noches con tormenta | Climatología de LIS/TRMM (Albrecht et al., BAMS 2016) o GOES‑16 GLM | Sí: es un JSON fijo en el repo |
| Últimos 7 / 30 / 365 días | Conteo real observado | Datos GLM de NOAA, procesados por un trabajo programado | Último dato guardado |

- Para el conteo real: un GitHub Action diario descarga y resume los datos GLM de la zona y publica un JSON pequeño junto a la web. La app lo descarga de la URL de la web (variable `PUBLIC_DATA_URL`), porque el `dist/` empaquetado en la app no se actualiza solo.
- Blitzortung no ofrece una API pública para esto y sus condiciones limitan el uso. Los proveedores comerciales (Vaisala, Earth Networks) son de pago.
- Recomendación inicial: empezar por climatología (mes/año, sin servidor, funciona sin conexión) + probabilidad de tormenta (semana). El conteo real solo si el presupuesto y el mantenimiento lo justifican.

---

## Decisiones pendientes

- [ ] `appId` (p. ej. dominio invertido) y nombre de la app en las tiendas.
- [ ] Quién tiene las cuentas de desarrollador y el Mac para compilar iOS.
- [ ] Solución de rutas en el WebView si falla la prueba de la Fase 1 (opción A o B).
- [ ] Lista inicial de operadores de tours y criterios para incluirlos.
- [ ] ¿Enlaces de afiliado o patrocinados? (afecta al aviso, a la privacidad y a las tiendas).
- [ ] Qué significa "cantidad de rayos" en la app: probabilidad, promedio histórico o conteo real.
- [ ] Coordenadas de referencia para el tiempo y la distancia.
- [ ] Presupuesto para APIs o servidor (si se elige algo que no sea gratis).

## Registro de decisiones

| Fecha | Decisión | Motivo |
|---|---|---|
| | | |
