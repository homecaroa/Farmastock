# 💊 FarmaStock — Control de Stock de Almacén Farmacéutico

Sistema web para la gestión visual del inventario de un almacén de farmacia. Permite ubicar artículos en zonas del plano del almacén, consultarlos y hacer seguimiento del stock por ubicación.

> **Sin servidor. Sin base de datos. Sin instalación.** Un único archivo HTML que funciona en cualquier navegador moderno.

---

## ✨ Funcionalidades

- 🗺️ **Plano interactivo** — Mapa fiel al almacén real con zonas 1–23 + Casonera
- ➕ **Añadir artículos** — Nombre obligatorio, Código Nacional (CN) opcional
- 🔒 **Control de duplicados** — Un artículo no puede estar en dos zonas a la vez (por nombre y por CN)
- 🔍 **Buscador** — Busca por nombre o CN y localiza la zona en el plano
- 💾 **Persistencia local** — Los datos se guardan en `localStorage` del navegador
- 🗑️ **Eliminar artículos** — Con un clic desde la ficha de zona

---

## 🚀 Uso

### Opción A — Directamente en el navegador
1. Descarga `farmacia-almacen.html`
2. Ábrelo con doble clic en cualquier navegador (Chrome, Firefox, Edge, Safari)
3. Empieza a añadir artículos

### Opción B — GitHub Pages (recomendado)
1. Haz fork o clona este repositorio
2. Renombra `farmacia-almacen.html` a `index.html`
3. Ve a **Settings → Pages → Branch: main / root**
4. Accede a `https://tu-usuario.github.io/nombre-repo/`

---

## 📋 Cómo añadir artículos

1. Haz clic en una zona del plano (ej: Zona 5, Zona 12, etc.)
2. En el panel derecho aparece el formulario de la zona
3. Escribe el **nombre del artículo** (obligatorio)
4. Escribe el **Código Nacional** si lo tienes (opcional)
5. Pulsa **Añadir** o la tecla Enter

---

## 🔍 Cómo buscar un artículo

1. En el campo **"Buscar artículo"** escribe el nombre o el CN
2. Pulsa **Buscar** o Enter
3. Aparecen los resultados con la zona donde está ubicado
4. Haz clic en el resultado para seleccionar esa zona en el plano

---

## 🗂️ Estructura del proyecto

```
farmastock/
├── farmacia-almacen.html   # Aplicación completa (archivo único)
├── README.md               # Este archivo
├── CHANGELOG.md            # Historial de cambios
├── LICENSE                 # Licencia MIT
└── .gitignore              # Archivos a ignorar por Git
```

---

## 💾 Almacenamiento de datos

Los datos se guardan en el `localStorage` del navegador bajo la clave `farmastock_v1`.

- ✅ Los datos persisten al cerrar y reabrir el navegador
- ✅ No requiere conexión a internet
- ⚠️ Los datos son locales al navegador/dispositivo
- ⚠️ Limpiar el caché del navegador borra los datos

---

## 🏗️ Tecnologías

| Tecnología | Uso |
|---|---|
| HTML5 | Estructura |
| CSS3 (variables, grid, flexbox) | Estilos y layout |
| JavaScript vanilla (ES6+) | Lógica de aplicación |
| SVG | Plano del almacén |
| localStorage | Persistencia de datos |
| Google Fonts (IBM Plex) | Tipografía |

---

## 🤝 Contribuir

1. Haz fork del repositorio
2. Crea una rama: `git checkout -b mejora/nombre-mejora`
3. Haz commit: `git commit -m 'Añade: descripción de la mejora'`
4. Push: `git push origin mejora/nombre-mejora`
5. Abre un Pull Request

---

## 📄 Licencia

MIT — Libre para uso, modificación y distribución. Ver `LICENSE`.
