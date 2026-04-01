# Changelog

Todos los cambios notables de este proyecto se documentan aquí.
Formato basado en [Keep a Changelog](https://keepachangelog.com/es/1.0.0/).

---

## [1.1.0] — 2026-04-01

### Añadido
- Validación de nombre duplicado entre zonas: un mismo artículo no puede registrarse en dos ubicaciones simultáneamente
- Validación de Código Nacional duplicado entre zonas

### Cambiado
- El Código Nacional (CN) pasa a ser campo opcional en el formulario de alta

### Corregido
- El buscador fallaba con artículos sin código nacional asignado

---

## [1.0.0] — 2026-04-01

### Añadido
- Plano interactivo SVG del almacén con 23 zonas numeradas + Casonera
- Alta de artículos por zona (nombre + CN)
- Eliminación de artículos por zona
- Buscador por nombre o Código Nacional
- Indicador visual de zonas con artículos (color verde)
- Contador de artículos por zona en el plano
- Persistencia de datos con localStorage
- Interfaz responsive para móvil y escritorio
