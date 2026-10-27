# CLAUDE.md — FarmaStock

> Documento de referencia para agentes Claude que trabajen en este proyecto.
> Describe la arquitectura, convenciones, tareas paralelas y skills asociadas.

---

## Descripción del proyecto

**FarmaStock** es una aplicación web de página única (SPA) para la gestión visual del inventario de un almacén farmacéutico. Permite ubicar artículos en zonas de un plano interactivo SVG, consultarlos, editarlos, moverlos entre zonas y hacer copias de seguridad en JSON.

- **Stack**: HTML5 + CSS3 + JavaScript ES6 vanilla. Sin frameworks, sin servidor, sin dependencias externas de runtime.
- **Despliegue**: GitHub Pages — archivo único `index.html` como punto de entrada.
- **Persistencia**: `localStorage` del navegador. `farmastock_v1` = artículos por zona; `farmastock_zones_v1` = metadatos de zona (nombre, color, geometría, altas y bajas); `farmastock_v1_respaldo` = copia cruda si al cargar hubo que descartar datos malformados.
- **Única dependencia externa de runtime**: `xlsx` (CDN jsdelivr) para «Desde Excel». Si no carga, el resto de la app funciona.
- **Audiencia**: Personal de farmacia sin formación técnica.

---

## Estructura del repositorio

```
farmastock/
├── index.html       # Aplicación completa (único archivo de la app)
├── farmacia-almacen.html  # v1.0 original (1-abr-2026). No es la app desplegada; no tocar
├── CLAUDE.md        # Este archivo — guía para agentes
├── README.md        # Documentación para usuarios/GitHub
├── CHANGELOG.md     # Historial de versiones
├── LICENSE          # MIT
└── .gitignore
```

---

## Arquitectura de index.html

El archivo tiene tres secciones bien delimitadas:

### 1. `<style>` — CSS con variables
Todas las variables de diseño están en `:root`. El sistema de clases es deliberadamente corto (abreviado) para minimizar el tamaño del archivo.

```
Variables clave:
  --accent    → #1a3a5c  (azul marino corporativo)
  --green     → #0d7a4e  (zonas con artículos)
  --red       → #b91c1c  (errores / eliminar)
  --ze        → #f7f9fc  (zona vacía)
  --zf        → #dcfce7  (zona con artículos)
  --za        → #bfdbfe  (zona seleccionada)
```

### 2. `<body>` — HTML
- `<header>` → logo + botones Exportar/Importar
- `.layout` → grid 2 columnas: `.mapp` (plano SVG) + `.sidebar` (panel)
- `.mapp` → botón «Editar plano», leyenda de riesgo y SVG (`viewBox="0 0 620 800"`) con 26 zonas base `<rect class="zone-rect" data-zone="N">` + textos (las zonas creadas se añaden por JS)
- `.sidebar` → búsqueda + detalle de zona + formulario de alta + lista de artículos
- `#mm` → modal de movimiento entre zonas
- `#toast` → notificaciones flotantes

### 3. `<script>` — JavaScript
Funciones principales:

| Función | Propósito |
|---|---|
| `loadDB()` / `saveDB()` | Leer/escribir localStorage |
| `ensureIds()` | Migrar artículos sin `_id` |
| `dupName(name, xid)` | Detectar nombre duplicado (excluye al propio artículo por `_id`) |
| `dupCode(code, xid)` | Detectar CN duplicado |
| `byId(id)` | Localizar artículo por `_id` → `{zone, idx, item}` |
| `renderMap()` | Actualizar clases y contadores del SVG |
| `renderZone()` | Renderizar lista de artículos del panel lateral |
| `addItem()` | Alta de artículo con validación anti-duplicados |
| `openEdit(id)` / `saveEdit(id)` | Edición inline por `_id` |
| `openMM(id)` / `execMove(tz)` | Modal de mover — relocaliza por `_id` en ejecución |
| `delItem(id)` | Borrar por `_id` con confirmación |
| `doSearch()` | Buscar en todo el `db` por nombre o CN |
| `exportData()` / `importData(ev)` | Exportar/importar JSON con validación |

### Modelo de datos

```json
// localStorage: farmastock_v1
{
  "5": [
    { "_id": "a1b2c3d4", "name": "Paracetamol 650mg", "code": "123456" },
    { "_id": "e5f6g7h8", "name": "Ibuprofeno 400mg",  "code": ""       }
  ],
  "12": [
    { "_id": "x9y0z1w2", "name": "Omeprazol 20mg",    "code": "789012" }
  ]
}
```

**Invariante crítica** (desde la v1.4): `_id` es único globalmente. Un mismo nombre o CN (case-insensitive) puede estar en **como máximo 2 zonas** y nunca dos veces en la misma. `dupName`/`dupCode` solo bloquean a partir de ahí; `enEstaZona`/`codeEnZona` evitan repetir nombre/CN dentro de una misma zona al añadir y editar (al editar solo si cambian), y `choqueCon(it,lista)` lo hace al mover (artículo suelto y zona completa). Un traslado no cambia en cuántas zonas está cada nombre/CN, por eso `planTraslado` solo comprueba choques con lo que ya hay en el destino.

### Modelo de zonas (v1.5)
`farmastock_zones_v1` = `{ id: {name,color,x,y,w,h,custom?,g0?} | {deleted:true} }`.
- **Existe** una zona si es base (`BASE_IDS`) y no tiene tumba, o es `custom:true`. **Geometría por defecto** ≠ existencia: base = `DEFAULT_GEOM` (leída del SVG del propio código al arrancar), custom = `g0` (la de creación). «Restaurar geometría» solo toca zonas que existen.
- Eliminar deja una **tumba** `{deleted:true}`: el ID no se reutiliza (comparación sin distinguir mayúsculas) ni la zona reaparece al recargar. Solo una copia de seguridad la recupera.
- La geometría es el dato; el DOM es su reflejo (`applyGeometry`/`layoutZone`). Todo en unidades del `viewBox` (`ptSvg` usa `getScreenCTM().inverse()`), nunca en píxeles.
- `reconcileZones()` (al cargar e importar) garantiza que **ningún artículo queda en una zona que no existe**: restaura la zona base eliminada o crea una custom.
- Eliminar zona / importar escriben con `commitStrict` (artículos y zonas juntos o ninguno).
- `loadDB`/`sanitizeZones`/`normItem` validan lo que viene de `localStorage` o de un `.json`; lo descartado se respalda en `farmastock_v1_respaldo`.

### Clasificación de riesgo (informativa)
- Campo opcional `item.riesgo = {estado:'sin_peligro'|'clasificado', cat:[ids ADR], fuente, obs, fecha}`. Sin campo = «No evaluado». **La fuente es opcional** (decisión del dueño, 27-oct-2026; antes era obligatoria para que «Sin peligro identificado» dejara siempre constancia de su origen: ahora esa constancia depende de quien lo registra). «Con peligro documentado» exige ≥1 clase. **Nunca se deduce del nombre ni del CN.**
- Catálogo `RIESGOS`: las 9 clases del **ADR 2025** tal como figuran en su apartado 2.1.1.1 (13 epígrafes: 1, 2, 3, 4.1, 4.2, 4.3, 5.1, 5.2, 6.1, 6.2, 7, 8, 9). Fuente verificada: texto oficial en [BOE-A-2025-77](https://www.boe.es/diario_boe/txt.php?id=BOE-A-2025-77) (publicado 3-ene-2025, aplicable desde 1-ene-2025; el periodo transitorio del ADR 2023 acabó el 30-jun-2025). Los colores son solo para distinguir en pantalla, no son los de las placas.
- **Ámbito y límites**: el ADR regula el **transporte** por carretera (y no el almacenamiento). Se usa solo como vocabulario normalizado de clases. Esta etiqueta es un registro interno informativo: **no garantiza cumplimiento normativo ni sustituye** la clasificación, el etiquetado ni las obligaciones de almacenamiento y manipulación. Para medicamentos acabados, el CLP (Reglamento 1272/2008) los excluye como producto terminado destinado al usuario final (art. 1); el almacenamiento de productos químicos tiene su propia normativa.
- **Pendiente de validar con quien corresponda (no verificado)**: qué normativa de almacenamiento es exigible en este almacén (p. ej. APQ/RD 656/2017, CLP/FDS para productos no medicamentosos) y si procede ampliar o sustituir el catálogo por una clasificación interna. Mientras tanto la estructura es configurable en `RIESGOS`.
- El mismo producto en 2 zonas son dos entradas independientes: su riesgo **no se sincroniza** automáticamente (sincronizar por nombre/CN sería deducir).
- **Peligrosidad por zona (v1.6)**: `zones[id].riesgo` = mismo formato que el de artículo pero solo `estado:'clasificado'` (con ≥1 clase; fuente opcional). Es una **designación** de la zona, independiente de la de sus artículos; nunca se copia de uno a otro. En el plano se suman (`zoneCats`: `art:true` = chip relleno, `art:false` = chip con borde). `zonaMatch` la usa el filtro; «No evaluado»/«Sin peligro» solo miran artículos. La ficha avisa de clases de artículos no designadas en la zona (solo si la zona tiene alguna designación). Registro interno: no equivale a una autorización de almacenamiento.

---

## Zonas del almacén

Zonas base (la lista real es `BASE_IDS`; las del plano pueden renombrarse, moverse, eliminarse y crearse):

| Columna | Zonas |
|---|---|
| Columna derecha | 1, 2, 3, 4, 5, 5b, 6, 6.2, 7, 7b, 8, 9 |
| Parte superior | 10, 11 |
| Columna izquierda | 12, 13, 14, 15, 16, 17 |
| Parte inferior izquierda | 18, 19 |
| Centro inferior | 20, 21, 22, 23 |

Todas las zonas son `<rect class="zone-rect" data-zone="ID">` en el SVG con `viewBox="0 0 620 800"`. IDs válidos: `/^[A-Za-z0-9][A-Za-z0-9._-]{0,11}$/` (se interpolan en selectores y atributos).

---

## Convenciones de código

- **Sin frameworks**: JavaScript vanilla ES6, sin imports, sin npm.
- **IDs por `_id`**: Nunca usar índices de array para identificar artículos entre operaciones. Siempre `byId(id)` antes de mutar.
- **Anti-duplicados en todas las entradas**: Llamar `dupName` y `dupCode` en `addItem`, `saveEdit` e `importData`. Nunca omitir.
- **Clases CSS abreviadas**: El CSS usa clases cortas (`.ic`, `.icv`, `.ii`, `.bc`) para minimizar tamaño. Mantener este estilo al añadir CSS.
- **Sin innerHTML con datos sin escapar**: Usar siempre `eh(s)` (escape HTML) y `ea(s)` (escape atributo) antes de interpolar en HTML.
- **GitHub Pages compatible**: No usar `import/export`, no requerir servidor, no usar APIs bloqueadas por CORS.

---

## Skills disponibles y cuándo usarlas

### `frontend-design`
**Cuándo**: Rediseñar la UI, añadir nuevos componentes visuales, mejorar el plano SVG, crear nuevas vistas o secciones.
**Capacidad clave**: Diseño con CSS variables, SVG, animaciones CSS, tipografía con Google Fonts.
**Ruta**: `/mnt/skills/public/frontend-design/SKILL.md`

### `xlsx`
**Cuándo**: Generar un informe de inventario en Excel a partir de los datos del `localStorage`, crear plantillas de importación masiva, exportar el stock por zonas a hoja de cálculo.
**Ruta**: `/mnt/skills/public/xlsx/SKILL.md`

### `pdf`
**Cuándo**: Generar un informe PDF del inventario (listado por zonas, resumen de stock), crear documentación imprimible del almacén.
**Ruta**: `/mnt/skills/public/pdf/SKILL.md`

### `docx`
**Cuándo**: Crear manual de usuario, guía de uso del sistema, documentación técnica del proyecto en formato Word.
**Ruta**: `/mnt/skills/public/docx/SKILL.md`

### `pptx`
**Cuándo**: Crear presentación del proyecto para farmacia, demostración de funcionalidades, formación para el personal.
**Ruta**: `/mnt/skills/public/pptx/SKILL.md`

### `skill-creator`
**Cuándo**: Crear skills nuevas específicas para FarmaStock (p.ej. un skill de auditoría de inventario, o de generación automática de informes de caducidad).
**Ruta**: `/mnt/skills/examples/skill-creator/SKILL.md`

---

## Plan de agentes en paralelo

Los agentes pueden trabajar en paralelo en los siguientes tracks independientes. Cada track es autónomo y no bloquea a los demás.

```
┌─────────────────────────────────────────────────────────────────┐
│                    AGENTES EN PARALELO                          │
│                                                                 │
│  Track A: UI/UX           Track B: Datos          Track C: Docs │
│  ─────────────            ──────────              ────────────  │
│  frontend-design          xlsx / pdf              docx / pptx   │
│                                                                 │
│  · Rediseño SVG           · Exportar a Excel      · Manual PDF  │
│  · Nuevas zonas           · Informe por zonas     · Guia Word   │
│  · Modo oscuro            · Plantilla importar    · Formacion   │
│  · Vista movil            · Caducidades           · Presentación│
└─────────────────────────────────────────────────────────────────┘
```

### Track A — Interfaz (skill: `frontend-design`)
Tareas independientes entre sí:
- **A1**: Mejorar el plano SVG con más detalle visual (pasillos, etiquetas de estantería)
- **A2**: Añadir modo oscuro (toggle en cabecera, vars CSS)
- **A3**: Vista móvil mejorada (colapso del panel, plano táctil)
- **A4**: Animación de transición al seleccionar zona
- **A5**: Panel de estadísticas (total artículos, zonas vacías, zona más llena)

### Track B — Datos (skills: `xlsx`, `pdf`)
Tareas independientes entre sí:
- **B1**: Script Python que convierte el JSON exportado a `.xlsx` con una hoja por zona
- **B2**: Generador de informe PDF del inventario completo ordenado por zona
- **B3**: Plantilla `.xlsx` para importación masiva de artículos
- **B4**: Función de exportación a CSV dentro del propio `index.html`

### Track C — Documentación (skills: `docx`, `pptx`)
Tareas independientes entre sí:
- **C1**: Manual de usuario en `.docx` con capturas y pasos ilustrados
- **C2**: Presentación `.pptx` de 8 slides para presentar el sistema al equipo
- **C3**: Guía de administrador (backup, restauración, migración de datos)

---

## Tareas de un único agente (secuenciales o con dependencias)

Estas tareas **no** deben paralelizarse porque tienen dependencias entre sí:

1. **Añadir campo "cantidad"**: Requiere modificar modelo de datos + UI + validaciones + exportación/importación (todo en `index.html`).
2. **Sistema de caducidades**: Requiere nuevo campo fecha en el modelo + lógica de alertas + render de avisos + exportación.
3. **Multi-almacén**: Cambio profundo en la estructura del `db` — afecta a todas las funciones.

---

## Cómo contribuir / instrucciones para agentes

1. **Leer este archivo primero** antes de modificar cualquier cosa.
2. **No romper la invariante de `_id`**: Cualquier función que añada o duplique artículos debe asignar `_id: uid()`.
3. **Validar siempre antes de escribir**: Llamar `dupName` y `dupCode` antes de cualquier push al `db`.
4. **Probar en navegador real**: GitHub Pages sirve HTML estático — probar abriendo `index.html` directamente con `file://` o con `python3 -m http.server`.
5. **Un solo archivo**: Mantener todo en `index.html`. No añadir archivos `.js` o `.css` separados (GitHub Pages sin build step).
6. **Actualizar `CHANGELOG.md`** con cada cambio relevante siguiendo el formato existente.

---

## Versión actual

`v1.6.0` (sobre la 1.5.x) — Nombres de zona, riesgo documentado, edición del plano y baja segura de zonas. Ver `CHANGELOG.md` (la v1.4 se documentó a posteriori).
