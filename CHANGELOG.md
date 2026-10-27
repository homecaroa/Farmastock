# Changelog

Formato basado en [Keep a Changelog](https://keepachangelog.com/es/1.0.0/).

---

## [1.6.0] — 2026-10-27

### Añadido
- **Peligrosidad por zona**, además de la de cada artículo: en «Configurar zona» se pueden designar las clases ADR para las que está pensada una zona (por ejemplo, un armario de inflamables), con fuente obligatoria y observaciones. Es independiente de la de los artículos y se suman:
  - En el plano, **chip relleno** = hay artículos con esa clase; **chip con borde** = la zona está designada para esa clase. Una zona vacía designada también se marca.
  - El filtro por riesgo resalta las zonas designadas y las lista en los resultados; «No evaluado» y «Sin peligro» siguen mirando solo artículos.
  - La ficha de zona muestra su designación y avisa si hay artículos con una clase que la zona no tiene designada.
  - Al eliminar una zona se avisa de que pierde su designación (los artículos conservan la suya).
  - Va en `farmastock_zones_v1` (`zones[id].riesgo`), con la misma validación que el riesgo de artículo (sin fuente o con clase inexistente se descarta al cargar o importar). Exportación `_v:"1.6"`; las copias anteriores se importan igual.
- Es un registro interno informativo: **no equivale a una autorización de almacenamiento** (el diálogo y la leyenda lo dicen).

---

## [1.5.1] — 2026-10-27

### Corregido
- **Mover un artículo a una zona que ya tenía ese nombre o CN lo duplicaba dentro de la zona.** Ahora se rechaza con el motivo y el diálogo sigue abierto para elegir otra zona.
- **Editar un artículo permitía repetir un nombre o CN que ya existía en la misma zona** (solo se miraba si estaba en otras 2 zonas). Solo se comprueba lo que cambia, así que un artículo que ya tuviera un «gemelo» por este fallo se puede seguir editando.
- **«Mover zona completa» tampoco miraba el destino** (la comprobación era sobre otras zonas). Ahora simula artículo a artículo contra lo que habrá en el destino: los que chocan se quedan en el origen y el resto se mueve. Si chocan todos, no mueve nada.
- El alta de un artículo no rechazaba un CN repetido dentro de la misma zona.
- **Color de zona**: el diálogo «Configurar zona» pedía escribir un código hexadecimal (`#dcfce7`). Ahora hay un selector de color y la casilla «Sin color propio». Las zonas nuevas nacen sin color propio (antes quedaban fijas en verde y no reflejaban su estado). El texto de la zona pasa a claro u oscuro según el color para seguir leyéndose.

### Cambiado
- (Esta nota se sustituyó en la 1.6.0: el diálogo ahora permite designar peligrosidad por zona.)

---

## [1.5.0] — 2026-10-26

### Añadido
- **Nombres de zona**: cada zona conserva su ID técnico y puede tener un nombre visible («Zona 5 — Analgésicos»). Se edita con el botón «✎ Zona» del panel lateral (o clic derecho en el plano) y se muestra en el plano, la ficha de zona, los resultados de búsqueda, los selectores de destino y el `<title>` accesible de cada zona. Sin nombre se muestra «Zona [ID]».
- **Riesgo documentado de cada artículo** (informativo): estado «No evaluado» (por defecto), «Sin peligro identificado (verificado)» o «Con peligro documentado», con clase(s) ADR, fuente (obligatoria si se verifica) y observaciones. Filtro por riesgo en el buscador, resaltado de zonas en el plano, señal ⚠ + códigos de clase en cada zona afectada (con «+n» si no caben) y leyenda con texto, iconos y colores. Ver «Clasificación de riesgo» en `claude.md` para ámbito y límites.
- **Modo «Editar plano»**: mover y redimensionar zonas con ratón, táctil o teclado (flechas / Mayús+flechas), en coordenadas del `viewBox` (correcto con el plano escalado), con tamaño mínimo 30×24 y límites del plano, «Deshacer último cambio» y «Restaurar geometría inicial» (con confirmación). El resto de acciones quedan desactivadas mientras se edita.
- **Eliminar zona con seguridad**: el diálogo muestra nombre, ID y nº de artículos. Una zona con artículos no se puede eliminar sin trasladarlos antes a una zona elegida explícitamente; si hay conflicto de nombre/CN en el destino, no se hace nada y se explica por qué. Operación atómica (artículos y zonas se guardan juntos o no se guarda nada).
- Zonas navegables con teclado (Tab, Enter/Espacio) con indicador de foco visible.

### Cambiado
- **Persistencia de zonas** (`farmastock_zones_v1`, ya existente desde la 1.4): ahora distingue entre *zonas que existen* y *geometría por defecto*. Zonas base sin cambios; zonas nuevas llevan `custom:true` y su geometría de creación `g0`; una zona eliminada deja una «tumba» `{deleted:true}` (su ID no se reutiliza ni reaparece al recargar). `farmastock_v1` (artículos) mantiene su forma; los artículos admiten un campo opcional `riesgo`.
- Arrastrar ya no mueve zonas en el uso normal: solo en «Editar plano».
- Una zona nueva se coloca en un hueco libre del plano (antes se solapaba con las zonas 1 y 2) y su ID se valida (letras, números, `.`, `-`, `_`; máx. 12).
- Exportar usa `_v:"1.5"` e incluye zonas, nombres, geometría y riesgos. Las copias antiguas siguen importándose: sin metadatos de zona no se tocan las zonas actuales, y un artículo importado en una zona eliminada la restaura para no quedar huérfano.
- Importar es todo-o-nada y valida IDs de zona, artículos y riesgos; `_id` con caracteres no seguros se regeneran.
- Cabecera en móvil: los botones pasan a varias líneas (antes la página desbordaba a 773 px en un móvil de 390 px).

### Corregido
- **No se podía eliminar ningún artículo**: la 1.4 perdió la función `delItem` y el botón ✕ lanzaba un error. Recuperada de la 1.3.
- **Zonas creadas desaparecían del plano al recargar** dejando sus artículos huérfanos e invisibles. Ahora se reconstruyen y, si hay artículos en una zona inexistente, se restaura la zona.
- **Eliminar una zona borraba sus artículos** sin alternativa y la zona base reaparecía vacía al recargar.
- **«Mover zona completa» destruía los artículos omitidos** por duplicado al borrar la zona de origen; ahora se quedan donde estaban.
- Al editar un artículo se descartaba cualquier campo que el formulario no conociera; ahora se conservan.
- El arrastre mezclaba píxeles de pantalla con unidades del `viewBox` (movía distinto según el tamaño de pantalla) y seleccionaba la zona al soltar.
- El tooltip de zona no se veía (`title` como atributo no funciona en SVG) y cada render añadía otro manejador de clic derecho.
- `ea()` no escapaba `&` ni `<`: un nombre con `&amp;` se alteraba al editarlo.
- Datos malformados en `localStorage` (JSON ilegible, artículos sin nombre, zonas con geometría inválida) ya no rompen el arranque: se conserva lo válido y una copia cruda del original en `farmastock_v1_respaldo`.
- Eliminadas funciones muertas e incompatibles con el nuevo modelo (`resizeZone`, `resetZoneLayout`, `importZoneConfig`).

---

## [1.4.0] — 2026-05-13 *(documentada a posteriori a partir del código; no se anotó en su día)*

### Añadido
- Importación desde Excel/CSV (`xlsx` desde CDN, única dependencia externa de runtime) con asignación masiva de zonas.
- Zonas 6.2 y 7b; panel de estadísticas; «Mover zona completa»; configuración de zona (nombre y color), «Nueva zona», arrastre de zonas y exportación de la configuración de zonas (`farmastock_zones_v1`).

### Cambiado
- Un mismo nombre o CN puede estar en hasta **2 zonas** (antes, 1). `dupName`/`dupCode` solo bloquean a partir de ahí.

---

## [1.3.0] — 2026-04-03

### Corregido
- **Bug crítico**: un artículo podía aparecer en 2 ubicaciones simultáneas.
  - Causa raíz 1: los índices de array pasados como strings desde `onclick` fallaban la comparación estricta `===` en JavaScript, por lo que el artículo nunca se excluía de su propia validación al editar.
  - Causa raíz 2: el índice capturado al abrir el modal de mover quedaba obsoleto si se modificaba la lista antes de confirmar, causando que `splice` eliminara el artículo equivocado.
  - Solución: cada artículo tiene ahora un `_id` único e inmutable. Todas las operaciones (editar, mover, borrar, validar) localizan el artículo por `_id` en tiempo de ejecución, nunca por índice.
- Migración automática de datos existentes en `localStorage` para asignar `_id` a artículos sin él.

---

## [1.2.0] — 2026-04-03

### Añadido
- Edición inline de artículos con formulario dentro de la tarjeta
- Mover artículos entre zonas mediante modal con selector visual
- Exportar inventario a archivo `.json` con fecha en el nombre
- Importar inventario desde `.json` con validación de estructura y duplicados

### Cambiado
- Los botones de acción de cada artículo usan delegación de eventos (más robusto)

---

## [1.1.0] — 2026-04-01

### Añadido
- Validación de nombre duplicado entre zonas
- Validación de Código Nacional duplicado entre zonas

### Cambiado
- El Código Nacional (CN) pasa a ser campo opcional

### Corregido
- El buscador fallaba con artículos sin CN asignado

---

## [1.0.0] — 2026-04-01

### Añadido
- Plano interactivo SVG con 23 zonas numeradas + Casonera
- Alta de artículos por zona (nombre + CN)
- Eliminación de artículos
- Buscador por nombre o CN
- Indicador visual de zonas con artículos
- Contador de artículos por zona en el plano
- Persistencia con localStorage
- Interfaz responsive
