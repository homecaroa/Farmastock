# 💊 FarmaStock — Control de Stock de Almacén Farmacéutico

Sistema web para la gestión visual del inventario de un almacén de farmacia. Permite ubicar artículos en zonas del plano, consultarlos, editarlos, moverlos entre zonas y exportar/importar el inventario.

> **Sin servidor. Sin base de datos. Sin instalación.** Un único archivo HTML que funciona en cualquier navegador moderno.

---

## ✨ Funcionalidades

- 🗺️ **Plano interactivo** — Mapa SVG fiel al almacén real con zonas 1–23 + Casonera
- ➕ **Añadir artículos** — Nombre obligatorio, Código Nacional (CN) opcional
- ✏️ **Editar artículos** — Edición inline sin perder la ubicación
- ↔️ **Mover entre zonas** — Modal con selector de zona destino
- 🔒 **Sin duplicados** — Un artículo no puede estar en dos zonas a la vez (por nombre ni por CN). Cada artículo tiene un ID interno único para garantizarlo incluso tras ediciones y movimientos
- 🔍 **Buscador** — Busca por nombre o CN en todo el almacén y localiza la zona
- ⬇️ **Exportar** — Descarga el inventario completo como archivo `.json`
- ⬆️ **Importar** — Carga un backup `.json` con validación previa
- 💾 **Persistencia local** — Los datos se guardan en `localStorage` del navegador

---

## 🚀 Despliegue en GitHub Pages

1. Sube todos los archivos al repositorio
2. Ve a **Settings → Pages**
3. En *Branch* selecciona `main` y carpeta `/ (root)`
4. Guarda — en unos segundos tendrás la URL pública

La app funciona con `index.html` como punto de entrada.

---

## 📋 Uso básico

| Acción | Cómo |
|---|---|
| Ver zona | Clic en la zona del plano |
| Añadir artículo | Selecciona zona → rellena el formulario → Añadir |
| Editar artículo | Botón ✏️ en la tarjeta del artículo |
| Mover artículo | Botón ↔ → elige zona destino en el modal |
| Eliminar artículo | Botón ✕ en la tarjeta |
| Buscar | Campo de búsqueda superior → Enter o botón Buscar |
| Exportar backup | Botón **⬇ Exportar** en la cabecera |
| Importar backup | Botón **⬆ Importar** en la cabecera → selecciona `.json` |

---

## 🗂️ Archivos del repositorio

```
farmastock/
├── index.html       # Aplicación completa (punto de entrada GitHub Pages)
├── README.md        # Este archivo
├── CHANGELOG.md     # Historial de cambios
├── LICENSE          # Licencia MIT
└── .gitignore       # Archivos ignorados por Git
```

---

## 💾 Sobre el almacenamiento

Los datos se guardan en `localStorage` bajo la clave `farmastock_v1`.

- ✅ Persisten al cerrar y reabrir el navegador
- ✅ No requieren conexión a internet
- ⚠️ Son locales al navegador/dispositivo — usa **Exportar** para hacer copias de seguridad
- ⚠️ Limpiar el caché del navegador borra los datos

---

## 🏗️ Tecnologías

HTML5 · CSS3 · JavaScript ES6+ · SVG · localStorage · Google Fonts (IBM Plex)

---

## 📄 Licencia

MIT — libre para uso, modificación y distribución.
