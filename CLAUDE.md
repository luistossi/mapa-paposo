# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Qué es

"Recorrido por Paposo": mapa interactivo de lugares significativos del pueblo chango en Paposo y alrededores (comuna de Taltal). Es un Recurso Pedagógico Digital para Lengua y Cultura de los Pueblos Originarios Ancestrales, de 1° a 6° básico. La interfaz y los textos están en español de Chile.

## Comandos

No hay build, dependencias, lint ni tests. Es un sitio estático de un solo archivo.

- Ver en local: abrir `index.html` en el navegador. Las fuentes vienen de Google Fonts y no hay nada más externo.
- Publicar: hacer `git push` a `main`. El proyecto Vercel `mapa-paposo` (equipo `luistossi-s-projects`) está conectado al repo y publica solo en https://mapa-paposo.vercel.app.
- CLI de Vercel, si hace falta: `npx vercel@65.16.0 ...`. Node está en `C:\Program Files\nodejs`; si no aparece en el PATH de la shell, agregarlo antes. Usar la CLI y no leer el token de Vercel directamente.

## Dos archivos con la misma página

- `index.html` es el sitio publicado: documento completo con `<!doctype>`, `<head>` y metadatos.
- `maqueta.html` tiene el mismo CSS y JS, pero sin `<html>`, `<head>` ni `<body>`. Es el artifact de claude.ai (https://claude.ai/artifact/HmaWeTiUrUUBxP7LbMZARD), y la plataforma le agrega ese esqueleto al publicar.
- Todo cambio de mapa, datos o estilos va en **los dos** archivos. Si cambia `maqueta.html`, hay que volver a publicar el artifact.

## Arquitectura (todo en el `<script>` de cada archivo)

**Geometría real, sin librería de mapas.**
- `utm(lon, lat)` proyecta WGS84 a UTM zona 19S y devuelve `E`, `N` en metros y `x`, `y` en km para el SVG, con `y = 7400 − N/1000` para que el norte quede arriba.
- La unidad del `viewBox` es el kilómetro.
- `COAST_RAW` es la costa de Natural Earth 10m afinada con puntas de OpenStreetMap, ordenada de sur a norte. `COAST` la reordena por latitud al norte de la bahía de Taltal y descarta los nodos de `DROP`.
- La tierra y las franjas de mar son la misma costa desplazada hacia el oeste (`shore(dx)`).

**Datos.**
- `PLACES`: recorrido principal, en orden norte → sur. Ese orden define `ROUTE`, la navegación anterior/siguiente y el número de parada. Cada lugar tiene `cat` (`caleta` | `arq` | `nat` | `paposo`), `ll` [lon, lat], `side` (lado de la etiqueta) y los textos del guion. Los flags opcionales son `approx`, `hierbas`, `sub` y `media`.
- `PAPOSO_TEMAS`: subtemas que se abren desde la ficha de Paposo. Un tema sin `body`, `list` ni `tools` se muestra como "pendiente".
- `SITES`: capa de la Comunidad Indígena Loreto. Usa `ll` cuando hay coordenada de OSM. Si no, usa `n`, el norte UTM en km leído de las fotos de sus cartas (±1 km), y se ubica sobre la costa con `coastAtY`. `h` marca los sitios que salen en el mapa de hierbas medicinales.
- `OAS`: objetivos de aprendizaje por nivel, para la guía docente.

**Vista y zoom.**
- `view = {cx, cy, h}` está en km. `apply()` reescribe el `viewBox`.
- Los marcadores y etiquetas son `screenItems`: con `scale(s)`, donde `s` = km por px, mantienen tamaño fijo en pantalla.
- Las montañas crecen con el zoom, con un tope.
- `declutter()` oculta las etiquetas que chocan según `prio`: primero el lugar seleccionado, luego Paposo, ciudades, lugares y al final la capa comunitaria. Con el mapa alejado (`view.h > 45`) se ocultan las etiquetas de la capa comunitaria.
- `flyTo` anima. Arrastre, pellizco y rueda usan pointer events.

**Ficha.**
- `select`, `selectSite` y `openTema` renderizan `#panel`. `intro` y `home` vuelven al inicio.
- Bajo 1024 px la ficha va debajo del mapa y aparece la tarjeta flotante `.peek`.

## Convenciones

- Colores y fuentes salen de los tokens en `:root`. La estética imita la referencia de Genially: mar turquesa en franjas, desierto durazno, montañas y simbología burdeo, y Oswald para los títulos.
- No inventar contenido cultural ni histórico. Lo que falta se marca como pendiente para que lo complete el equipo con la comunidad.
- Datos por confirmar: Hito 25 (OSM, en Paposo) frente al paralelo 24° del guion; si Punta Negra corresponde a Guanillos; ubicaciones aproximadas de Miguel Díaz, el humedal, Los Perales y los sitios de `SITES` que usan `n`.
- Tiene que funcionar en celular, tablet y escritorio, sin desplazamiento horizontal.
