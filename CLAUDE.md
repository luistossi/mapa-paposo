# CLAUDE.md

Contexto para sesiones de Claude que trabajen en este repositorio.

## Qué es

«Huellas Changas: Descubriendo Paposo» (repo `luistossi/mapa-paposo`, antes «Recorrido por Paposo»): Recurso Pedagógico Digital (RPD) con un mapa interactivo de lugares significativos del pueblo chango en Paposo y alrededores (comuna de Taltal, Región de Antofagasta).

- Asignatura: Lengua y Cultura de los Pueblos Originarios Ancestrales, 1° a 6° básico.
- Proyecto: «Conectividad y competencias interculturales en escuelas rurales» (UCN – UAndes).
- Colaboran la Comunidad Indígena Changa Loreto y la escuela rural Paranal G-107 de Paposo.
- Programador: Luis Tossi.
- Textos e imágenes: guion del equipo de seminario.
- Interfaz y textos en español de Chile.

## Archivos

- `index.html`: el sitio completo, con HTML, CSS y JS en un solo archivo, sin dependencias ni build. Solo carga Google Fonts (Oswald y Atkinson Hyperlegible).
- `img/`: 143 imágenes. El código usa 128.
  - Las rutas son relativas (`img/...`).
  - Vercel distingue mayúsculas: los nombres van en minúsculas y con la extensión exacta (`.jpg` o `.webp`).
  - Los dibujos de los niños se buscan como `img/<código en minúsculas>.jpg` (por ejemplo `dppaej8b.jpg`).
  - Sin uso todavía: el-gaucho, escuela-paranal, flora-costera-1..3, hito-fronterizo, la-playita, loreto-alero, loreto-caleta, muelle-paposo, paposo-ilustracion.webp, playita-huiro-1..3, rinconada.
- `maqueta.html`: versión ANTERIOR del mapa (arquitectura vieja, con `utm()` y `COAST_RAW`). No está sincronizada con `index.html` y no se publica como sitio. No editarla salvo que Luis lo pida; está pendiente decidir si se elimina.
- `README.md`: descripción breve.

## Publicación

- Hacer `git push` a `main`. El proyecto Vercel `mapa-paposo` (equipo `luistossi-s-projects`) despliega solo en https://mapa-paposo.vercel.app.
- El repo es privado.
- CLI de Vercel, si hace falta: `npx vercel@65.16.0 ...`. En el PC de Luis, Node está en `C:\Program Files\nodejs`. Usar la CLI; no leer el token directamente.
- Antes de hacer push, hacer `git fetch` y revisar `origin/main`: a veces trabajan varias sesiones a la vez.

## Estructura de la página

1. **Portada** (`#portada`):
   - Título «rupestre» (filtro SVG `#rough`).
   - Botones: Iniciar recorrido, Sugerencias pedagógicas, Fuentes y Créditos.
   - Diseño final según la maqueta del equipo: HTML y CSS sobre `img/ilustracion-paposo-mar.webp` (sin texto incrustado), con bandada de pájaros en SVG (se oculta bajo 1100 px) y botones en `.portada-nav` (`.btn.primary` y `.btn.pill`).
2. **Recorrido** (`#recorrido`, oculto al inicio):
   - Header con: Inicio, ¿Sabías qué?, Guía docente y sonido del mar.
   - `.stage` con el mapa (`#mapView`) y un panel lateral plegable (`#panel`), que se puede minimizar o cerrar y reabrir con `#panelOpen`.
3. **Ventanas modales** (`<dialog class="dlg">`):
   - `#fichaDlg`: fichas de lugares y temas.
   - `#sabiasDlg`: ¿Sabías qué?
   - `#sugDlg`: guía docente, OA por nivel y tablas curriculares.
   - `#fuentesDlg`: fuentes.
   - `#creditosDlg`: créditos.
   - Tienen trampa de foco, se cierran con Escape o con clic fuera, y devuelven el foco al botón que las abrió (`opener`).
   - Las imágenes se amplían con un lightbox (`.lightbox`) dentro del dialog.
4. **Paposo ilustrado** (`#ilus`):
   - Capa encima del mapa con una «moneda» 3D de dos caras: `ilustracion-paposo-cerro.webp` y `-mar.webp`, que se giran con `setCoin`.
   - La cara del cerro tiene zonas clicables (`ZONAS`, en % de la imagen) que abren fichas.

## Mapa (todo en el `<script>`)

**Coordenadas.**
- Están en UTM 19S en **km**, como pares `E, N`, leídas a mano de los mapas «Sitios de significación cultural» (Recorrido Orilla y Recorrido Mar) de la Comunidad Loreto. Son aproximadas.
- Al sur de Las Cañas se ubicaron según las distancias del guion.
- Mundo: `W = { e0: 292, e1: 404, n0: 7206, n1: 7356 }` (112 × 150 km).
- `P(E, N)` → `[x, y]` del SVG, en km. Si `flip` es verdadero, invierte ambos ejes: es la «orientación local», con el norte abajo.

**Geometría.**
- `COAST`: lista `[E, N]` de norte a sur.
- `smoothPath` aplica Catmull-Rom.
- `band(dE, edgeE)` dibuja las franjas de mar y `bandBetween(d1, d2)` las de tierra (planicie y cordillera). Ambas son la costa desplazada en E.
- `QUEBRADAS`: líneas de las quebradas.
- `PARALELO_N`: el paralelo 24° S, línea clicable que abre el tema «hito».

**Vista.**
- `view = { cx, cy, k }`, donde `k` = px por km.
- `render()`:
  - escribe el `viewBox`;
  - escala cada pin con `scale(1/k)` para que mantenga su tamaño en pantalla;
  - actualiza la barra de escala;
  - pone las clases de nivel de zoom en `#map`:
    - `z-l1` (k ≥ 6.5): etiquetas;
    - `z-sites` (≥ 20): sitios comunitarios;
    - `z-sitelbl` (≥ 40): sus etiquetas;
    - `z-village` (≥ 110): puntos de la caleta (muelle, escuela, posta, hito).
- `computeKMin()`: `K_MIN = max(ancho/WW, alto/WH)`, para que el mapa siempre cubra el recuadro y nunca se vea el fondo. `K_MAX = 420`.
- `clampView()`: mantiene la vista dentro del mundo.
- Funciones de movimiento:
  - `fitRoute()`: encuadra el recorrido según la forma de la pantalla;
  - `flyTo()`: anima, y respeta `prefers-reduced-motion`;
  - `zoomAt()`: zoom hacia un punto;
  - `centerOn()`.
- Interacción: arrastre, pellizco y rueda con pointer events. `drag.moved` evita que un arrastre se tome como clic.
- Un `ResizeObserver` reencuadra el mapa si el usuario no lo ha movido (`userMoved`).
- `draw()` reconstruye todo el SVG. Se llama al iniciar y en `setFlip`.

**Simbología.**
- `buildLegend()`: cada botón pone o quita la clase `hide-<cat>` en `#map`, y con hover resalta su categoría (`data-hl`). El CSS se genera en un `<style>` dinámico.

## Datos

- `PLACES`: las 13 paradas, en orden **norte → sur**. Ese orden define `ROUTE`, el número de parada y la navegación ↑ ↓. Campos:
  - `id`, `name`, `cat`, `E`, `N`, `icon`, `media[]`, `relato[]` (párrafos);
  - opcionales: `sub`, `secciones[]` (`{ h, p[], media, cuento, autor, glosario, cita }`), `fold` (secciones como acordeón), `sabias` (clave de `SABIAS`), `dibujos`, `stories`, `explora` (chips de temas de Paposo), `temas`, `big`, `color`.
- `TEMAS`: fichas de la caleta (escuela, muelle, posta, hito, cancha, actividades, fauna, camanchaca) y temas de apoyo:
  - `herramientas`: comparadores antes/ahora (`TOOLS`);
  - `floramed`: tarjetas de flora medicinal, que se muestran en la posta.
  - `PAPOSO_TEMAS` define los chips que se muestran en la ficha de Paposo.
- `VILLAGE`: pines de la caleta, que solo se ven con mucho zoom.
- `SIN_FICHA`: lugares grises sin ficha (Panul, y El Cobre y Blanco Encalada como referencias).
- `SITIOS`: puntos de significación cultural de la Comunidad Loreto, como `[nombre, E, N]`.
- `SABIAS`: curiosidades, ligadas a un `lugar` o a un `tema`.
- `STORIES`: leyendas sueltas. «La isla encantada» tiene el texto **[PENDIENTE]**.
- `KEYS`: palabras que se destacan en burdeo la primera vez que aparecen. El formato de la clave es exacto y tiene que coincidir con los títulos:
  - `"<id>"`
  - `"<id>/<título de sección>"`
  - `"t:<tema>"`
  - `"t:<tema>/<título>"`
  - `"c:<título de tarjeta o herramienta>"`
  - Si se renombra una sección, hay que actualizar su clave.
- `OAS`: objetivos de aprendizaje transcritos textualmente del guion. **No editarlos.**
- `CATS` y `CAT_EYEBROW`: categorías, íconos (`<symbol id="i-*">` en el sprite) y colores.
- Helpers:
  - `foto(archivo, alt, cap)`;
  - `fuente(archivo, alt, origen)`;
  - `dib(CÓDIGO)` → dibujo infantil.

## Funciones principales

- `select(id, from, force)`: abre la ficha de una parada. Con `id === "paposo"` y sin `force`, abre Paposo ilustrado.
- `openTema(key, from)`: abre la ficha de un tema.
- `openIlus()` y `closeIlus()`.
- `setFlip(on)`.
- `step(±1)`.
- `showRecorrido()` y `showPortada()`.
- Cómo se distribuyen las imágenes en las fichas (`storyHtml`):
  - 1 imagen: héroe, o flotante si es `tall`, `contain` o un dibujo;
  - 2 imágenes: intercaladas entre párrafos;
  - 3 o más: carrusel.
- Comparador antes/ahora: `.compare`, con un `<input type=range>` que mueve `--pos`.
- Sonido del mar: ruido generado con Web Audio, sin archivo de audio.
- Cursor: una gaviota SVG, en las variables CSS `--cur` y `--cur-hover`.

## Convenciones

- Colores y fuentes desde los tokens de `:root`. Estética: mar turquesa en franjas, desierto durazno, burdeo para la simbología y los títulos, y pictografías rojas estilo El Médano como decoración.
- **No inventar contenido cultural ni histórico.** Lo que falta se marca como `[PENDIENTE: ...]` para que el equipo lo complete con la comunidad.
- Tiene que funcionar en celular, tablet y escritorio, sin desplazamiento horizontal. Los blancos táctiles miden ≥ 44 px (`--tap`), y hay foco visible y textos alternativos en todas las imágenes.
- Datos por confirmar: las ubicaciones aproximadas (Miguel Díaz, los sitios comunitarios y todo lo que está al sur de Las Cañas) y si Punta Negra corresponde a Guanillos.

## Cómo probar

No hay tests automatizados. Para revisar:

- Abrir `index.html` en el navegador (funciona con `file://`).
- Recorrer las 13 paradas con ↓ hasta llegar a «Fin del recorrido».
- Abrir Paposo ilustrado, tocar cada zona y girar la moneda.
- Abrir cada tema, el norte abajo, el zoom al máximo y al mínimo, y la simbología.
- Revisar que no haya errores en la consola ni imágenes rotas.

Se hizo con Playwright y Chrome headless, en tamaños 1400×900, 1900×1300, 2560×1000, 390×800 y 1024×1366.

## Pendientes conocidos

- Texto de la leyenda «La isla encantada».
- Ficha de Panul.
- Decidir qué hacer con `maqueta.html` y las 15 imágenes sin uso.
- Con el zoom mínimo actual, en pantallas anchas no se ve el mapa completo de norte a sur (se ven unos 113 de los 150 km). Hay que arrastrar el mapa para llegar al paralelo 24° S.
