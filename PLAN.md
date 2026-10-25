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

- [x] **1. Capacitor 8** instalado: `@capacitor/core`, `@capacitor/android`, `@capacitor/ios` y `@capacitor/cli`. `capacitor.config.ts` con `webDir: 'dist'`. Proyectos nativos `android/` e `ios/` creados (iOS con Swift Package Manager, sin CocoaPods).
- [x] **2. Scripts npm**: `app:build` (`astro build --mode app && cap sync`), `app:android`, `app:ios`, `app:assets`. Se usa `--mode app` en vez de una variable de entorno para que funcione también en Windows.
- [x] **3. Navegación en el WebView.** Confirmado en el código de Capacitor (Android `WebViewLocalServer`, iOS `CapacitorRouter`): toda ruta sin extensión devuelve el `index.html` de la raíz. Con el build web la app se quedaba en un bucle de redirecciones. Solución: opción B. En modo `app`, `homeUrl()`/`pageUrl()` terminan en `index.html`. La web no cambia.
- [ ] **4. Prueba sin conexión en teléfonos reales.** Hecha en simulación: un servidor con la misma regla de rutas que Capacitor y Chromium sin acceso a internet. Las 20 páginas, los 4 idiomas, las fuentes y las fotos cargan, la raíz lleva al idioma del teléfono y se recuerda el idioma elegido. **Falta:** repetirla en Android y en iOS, en modo avión.
- [x] **5. Enlaces externos**: Capacitor ya los abre en el navegador del sistema en Android y en iOS, así que no hace falta `@capacitor/browser`. Sin conexión, el navegador del sistema muestra su propio aviso.
- [x] **6. Configuración**: `android` e `ios` excluidos en `tsconfig.json`.
- [x] **7. Icono y splash**: fuentes en `assets/*.svg` (el dibujo de `public/icons/catatumbo.svg` sin el texto, que no se lee en un icono), generados con `@capacitor/assets` (`npm run app:assets`). Fondo `#142d31`, también para el splash del sistema en Android 12+.
  - [x] Barra de estado: `viewport-fit=cover` y márgenes `env(safe-area-inset-*)` en `site.css`. El hueco bajo la barra de estado toma el color de la barra superior, en modo claro y oscuro. **Falta:** mirarlo en un iPhone con muesca y en un Android reciente. La orientación horizontal (muesca a un lado) aún no tiene margen lateral.
- [ ] **8. Preparación para las tiendas**:
  - Cuentas: Google Play Console (pago único) y Apple Developer Program (anual).
  - Android: keystore de firma fuera del repo, con copia de seguridad. Versión (`versionCode`/`versionName`) en `android/app/build.gradle`.
  - iOS: hace falta macOS con Xcode para compilar y subir.
  - Capturas de pantalla por idioma, textos de la ficha en es/en/de/fr (de y fr revisados por hablantes nativos antes de publicar).
  - Política de privacidad: página Markdown nueva en los 4 idiomas (las dos tiendas la piden). Formularios de privacidad de Apple y "Seguridad de los datos" de Google.
  - Ojo: Apple puede rechazar apps que solo envuelven una web (guía 4.2). El funcionamiento sin conexión ayuda; los widgets de la Fase 3 ayudarían más.
- [x] **9. Documentado** en `CLAUDE.md` (notas técnicas de la app) y en `README.md` (comandos).
- [x] **10. La web sigue igual**: `npm run check` sin errores; con `npm run build` los enlaces siguen siendo `/es/relampago/` y la barra superior mide lo mismo (70 px / 60 px). El APK de depuración de Android compila (`./gradlew assembleDebug`). iOS no se ha compilado (necesita macOS).

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

- [ ] `appId` y nombre de la app en las tiendas. Ahora hay uno provisional, `com.catatumbodigital.guia`, con el nombre "Catatumbo" (en `capacitor.config.ts`, `android/app/build.gradle` e `ios/App/App.xcodeproj`). El `appId` no se puede cambiar después de publicar.
- [ ] Quién tiene las cuentas de desarrollador y el Mac para compilar iOS.
- [ ] Lista inicial de operadores de tours y criterios para incluirlos.
- [ ] ¿Enlaces de afiliado o patrocinados? (afecta al aviso, a la privacidad y a las tiendas).
- [ ] Qué significa "cantidad de rayos" en la app: probabilidad, promedio histórico o conteo real.
- [ ] Coordenadas de referencia para el tiempo y la distancia.
- [ ] Presupuesto para APIs o servidor (si se elige algo que no sea gratis).

## Registro de decisiones

| Fecha | Decisión | Motivo |
|---|---|---|
| 2026-10-25 | Capacitor 8 para la app. | Usa `dist/` tal cual; un solo código para web y app. |
| 2026-10-25 | En la app, los enlaces internos terminan en `index.html` (build con `--mode app`). | Capacitor sirve el `index.html` de la raíz para rutas sin extensión. Es el arreglo más pequeño y no toca la web. |
| 2026-10-25 | Los enlaces externos se abren en el navegador del sistema, sin `@capacitor/browser`. | Capacitor ya lo hace solo en Android e iOS: una dependencia menos. |
| 2026-10-25 | Icono sin el texto "CATATUMBO". | No se lee a tamaño de icono y el teléfono ya muestra el nombre debajo. |
