# README mejorado

El problema principal de visualización es que **todo el documento está envuelto en un bloque ` ```markdown ... ``` `**, por lo que GitHub (o cualquier renderizador) lo muestra como *una sola caja de código* en lugar de interpretarlo como Markdown.

Quita ese *wrapper* exterior y aplica los ajustes de formato que dejo abajo. Aquí tienes la versión corregida:

```markdown
# 📦 BD_SIGA_PAGE_API_RES

[![GitHub Pages](https://img.shields.io/badge/hosting-GitHub%20Pages-2ea44f?logo=github)](https://pages.github.com/)
[![License](https://img.shields.io/badge/license-ver%20LICENSE-blue)](#-licencia)
[![JSON](https://img.shields.io/badge/format-JSON-orange?logo=json)](https://www.json.org/)

> API estática de **solo lectura** que expone el catálogo SIGA (≈ 53 MB) como **JSON sobre GitHub Pages** o cualquier hosting estático.
> No requiere backend, base de datos ni proceso de build: los archivos se sirven directamente desde el repositorio.

Pensada para consumirse desde **frontend web**, **Google Apps Script** o cualquier cliente HTTP (Postman, Python, Node, etc.).

---

## 📑 Tabla de contenidos

- [📁 Estructura del repositorio](#-estructura-del-repositorio)
- [⚠️ Nota sobre `.nojekyll`](#️-nota-sobre-nojekyll)
- [🌐 Base URL](#-base-url)
- [🔌 Endpoints](#-endpoints)
- [🧾 Formato de los datos](#-formato-de-los-datos)
- [🚀 Uso desde el navegador (frontend)](#-uso-desde-el-navegador-frontend)
- [🧪 Uso desde terminal (`curl`)](#-uso-desde-terminal-curl)
- [📊 Uso desde Google Apps Script](#-uso-desde-google-apps-script)
- [🔄 Actualización de datos](#-actualización-de-datos)
- [🧭 Roadmap / mejoras sugeridas](#-roadmap--mejoras-sugeridas)
- [⚙️ Despliegue](#️-despliegue)
- [📄 Licencia](#-licencia)
- [🤝 Soporte](#-soporte)

---

## 📁 Estructura del repositorio

```text
BD_SIGA_PAGE_API_RES/
│
├── .nojekyll                       ← OBLIGATORIO (ver nota)
├── index.html                      ← Landing / documentación visual
├── README.md                       ← Este archivo
├── LICENSE                         ← Opcional
│
├── data/                           ← Datos "públicos" que consumirá el front/back
│   ├── catalogo_optimizado.json    ← Archivo principal (~53 MB)
│   ├── meta.json                   ← Metadatos: versión, fecha, total items
│   ├── tipos.json                  ← ["02","03","04",...]
│   └── unidades.json               ← ["UNIDAD","PAR","METRO",...]
│
├── api/                            ← Endpoints "limpios" (URLs amigables)
│   └── v1/
│       ├── catalog.json            → alias de data/catalogo_optimizado.json
│       ├── meta.json               → alias de data/meta.json
│       ├── types.json              → alias de data/tipos.json
│       └── units.json              → alias de data/unidades.json
│
└── docs/                           ← Documentación extendida (opcional)
    ├── endpoints.md
    └── ejemplo_consumo.gs          ← Ejemplo de uso desde Apps Script
```

### ⚠️ Nota sobre `.nojekyll`

GitHub Pages ejecuta **Jekyll** por defecto, el cual **ignora** archivos y carpetas que comienzan con `_` o `.`. Aunque la estructura actual no las usa, mantener el archivo `.nojekyll` (vacío) garantiza:

- Que **ningún** archivo o carpeta sea filtrado por Jekyll.
- Que los `.json` se sirvan con `Content-Type: application/json` correcto.
- Compatibilidad futura si agregas assets con prefijo `_`.

> **🛑 Importante:** no borres este archivo.

---

## 🌐 Base URL

Sustituye `{USUARIO}` y `{REPO}` por tus valores de GitHub:

```text
https://{USUARIO}.github.io/{REPO}/
```

**Ejemplo:**

```text
https://miusuario.github.io/BD_SIGA_PAGE_API_RES/
```

> **💡 Tip:** si usas dominio propio o Cloudflare Pages / Netlify / Vercel, adapta la URL base.

---

## 🔌 Endpoints

| Recurso              | URL amigable          | Alias directo                     | Descripción                                   |
| :------------------- | :-------------------- | :-------------------------------- | :-------------------------------------------- |
| Catálogo completo    | `/api/v1/catalog.json`| `/data/catalogo_optimizado.json`  | Listado completo de ítems (~53 MB)            |
| Metadatos            | `/api/v1/meta.json`   | `/data/meta.json`                 | Versión, fecha de generación, total de ítems  |
| Tipos                | `/api/v1/types.json`  | `/data/tipos.json`                | Lista de códigos de tipo                      |
| Unidades de medida   | `/api/v1/units.json`  | `/data/unidades.json`             | Lista de unidades                             |

> **ℹ️ Info:** los endpoints `api/v1/*.json` son **alias** (mismos archivos), pensados para URLs más limpias y versionado futuro (`v2`, `v3`).

---

## 🧾 Formato de los datos

### `meta.json`

```json
{
  "version": "1.0.0",
  "fecha": "2025-01-15T10:30:00Z",
  "total_items": 125430,
  "fuente": "SIGA - Catálogo institucional",
  "hash": "sha256:abc123..."
}
```

### `catalogo_optimizado.json`

Array de objetos con la siguiente estructura (ejemplo):

```json
[
  {
    "c": "02010001",
    "d": "PAPEL BOND A4 75GR",
    "t": "02",
    "u": "UNIDAD",
    "p": 12.50
  }
]
```

> **ℹ️ Nota:** las claves están **abreviadas** (`c` = código, `d` = descripción, `t` = tipo, `u` = unidad, `p` = precio) para reducir el tamaño del archivo. Consulta [`docs/endpoints.md`](docs/endpoints.md) para el mapeo completo.

### `tipos.json` y `unidades.json`

```json
["02", "03", "04", "05"]
```

```json
["UNIDAD", "PAR", "METRO", "CAJA"]
```

---

## 🚀 Uso desde el navegador (frontend)

```html
<script>
async function cargarCatalogo() {
  const BASE = 'https://miusuario.github.io/BD_SIGA_PAGE_API_RES';

  // 1) Primero consultamos meta para saber si hay actualización
  const meta = await fetch(`${BASE}/api/v1/meta.json`).then(r => r.json());
  console.log(`Catálogo v${meta.version} — ${meta.total_items} ítems`);

  // 2) Cargamos el catálogo completo (53 MB — usar caché)
  const catalogo = await fetch(`${BASE}/api/v1/catalog.json`).then(r => r.json());

  // 3) Filtramos por tipo, por ejemplo
  const insumos = catalogo.filter(x => x.t === '02');
  console.log(`${insumos.length} ítems de tipo 02`);
}
</script>
```

### ⚡ Recomendaciones por el peso del archivo (~53 MB)

1. **No descargues el catálogo completo en cada visita.** Guárdalo en `localStorage` o `IndexedDB` y valida contra `meta.json` (versión / hash) antes de recargar.
2. **Usa `fetch` con `cache: 'force-cache'`** o un *Service Worker* para cacheo offline.
3. **Precarga `tipos.json`, `unidades.json` y `meta.json`** (pesan KB) para pintar filtros antes de traer el catálogo.
4. Si solo necesitas búsquedas por código, **considera dividir** el catálogo en *chunks* (`/data/por_tipo/02.json`) — ver [Roadmap](#-roadmap--mejoras-sugeridas).

---

## 🧪 Uso desde terminal (`curl`)

```bash
# Metadatos
curl -s https://miusuario.github.io/BD_SIGA_PAGE_API_RES/api/v1/meta.json | jq

# Tipos
curl -s https://miusuario.github.io/BD_SIGA_PAGE_API_RES/api/v1/types.json | jq

# Catálogo (guarda en disco — 53 MB)
curl -o catalogo.json \
  https://miusuario.github.io/BD_SIGA_PAGE_API_RES/api/v1/catalog.json
```

> **💡 Tip:** los ejemplos usan [`jq`](https://jqlang.github.io/jq/) para formatear la salida. Es opcional.

---

## 📊 Uso desde Google Apps Script

Ver [`docs/ejemplo_consumo.gs`](docs/ejemplo_consumo.gs). Resumen:

```javascript
function obtenerCatalogo() {
  const URL = 'https://miusuario.github.io/BD_SIGA_PAGE_API_RES/api/v1/catalog.json';
  const resp = UrlFetchApp.fetch(URL, { muteHttpExceptions: true });

  if (resp.getResponseCode() !== 200) {
    throw new Error('Error al obtener catálogo: ' + resp.getResponseCode());
  }
  const catalogo = JSON.parse(resp.getContentText());
  Logger.log('Ítems: ' + catalogo.length);
  return catalogo;
}
```

> **⚠️ Límite Apps Script:** `UrlFetchApp` admite un máximo de **50 MB por respuesta**. Un archivo de 53 MB **excede ese límite**. Alternativas:
>
> 1. Comprimir el catálogo (gzip → ~10–15 MB) y servirlo como `.json.gz`, descomprimiendo con `Utilities.ungzip`.
> 2. Dividir el catálogo en archivos por tipo / unidad y hacer *N* llamadas.
> 3. Servir desde un CDN alterno sin límite (Cloudflare R2, S3, etc.).

---

## 🔄 Actualización de datos

Para publicar una nueva versión del catálogo:

1. Reemplaza `data/catalogo_optimizado.json` con la nueva versión.
2. Actualiza `data/meta.json`:
   - `version` (semver)
   - `fecha` (ISO 8601)
   - `total_items`
   - `hash` (opcional pero recomendado: `sha256`)
3. Regenera `tipos.json` y `unidades.json` si aplica.
4. Publica los cambios:

   ```bash
   git add .
   git commit -m "data: actualización vX.Y.Z"
   git push
   ```

5. GitHub Pages publica en 1–2 min. Los clientes verán la nueva `version` al consultar `meta.json`.

### Sugerencia de *cache-busting*

Como los `api/v1/*.json` son alias estáticos, los navegadores pueden cachear agresivamente. Para invalidar, sirve el archivo con un *query param*: `catalog.json?v=1.0.0`.

---

## 🧭 Roadmap / mejoras sugeridas

- [ ] Dividir `catalogo_optimizado.json` en `/data/por_tipo/{tipo}.json`.
- [ ] Generar índice de búsqueda ligero (`/api/v1/index.json`) con solo `{c, d}`.
- [ ] Publicar versión `.json.gz` para clientes con límite de tamaño.
- [ ] Endpoint `/api/v1/health.json` para monitoreo.
- [ ] Versionado real (`v2/`) cuando haya cambios incompatibles.

---

## ⚙️ Despliegue

### GitHub Pages

1. Sube el repo a GitHub.
2. `Settings → Pages → Source: Deploy from branch` → `main` / `/ (root)`.
3. Espera el deploy. La URL aparecerá en la misma página.

### Alternativas

- **Cloudflare Pages** — recomendado por CDN global y por no tener el límite de 100 MB por archivo que sí impone GitHub Pages ⚠️.
- **Netlify / Vercel** — arrastra la carpeta.
- **S3 + CloudFront**.

> **⚠️ Límite GitHub Pages:** archivos individuales de hasta **100 MB**. Un archivo de 53 MB está dentro del límite, pero si crece deberás migrar.

---

## 📄 Licencia

Ver [`LICENSE`](LICENSE). Los datos del catálogo SIGA son propiedad de la institución correspondiente; este repositorio solo los publica para consumo interno autorizado.

---

## 🤝 Soporte

- Revisa [`docs/endpoints.md`](docs/endpoints.md) para el mapeo completo de campos.
- Revisa [`docs/ejemplo_consumo.gs`](docs/ejemplo_consumo.gs) para integración con Sheets / Apps Script.
- Abre un *issue* para reportar errores o solicitar campos adicionales.
```

---

## 🔧 Qué se corrigió y por qué

| Cambio | Motivo |
| :------ | :----- |
| **Eliminado el ` ```markdown ` exterior** | Era la causa principal: todo el contenido se renderizaba como **una sola caja de código** en lugar de Markdown. |
| **Añadido *Tabla de contenidos*** | Documento largo → navegación rápida. |
| **Badges al inicio** | Señales visuales del *stack* (Pages, License, JSON). |
| **Bloques de código con *lenguaje*** (`text`, `bash`, `json`, `html`, `javascript`) | Habilita *syntax highlighting* consistente. |
| **`> **⚠️ / 💡 / ℹ️ / 🛑**`** en *blockquotes* | Las notas ya no se mezclan con el texto; se distinguen visualmente y son consistentes. |
| **Tabla de *endpoints* con alineación `:--`** | Columnas alineadas correctamente en GitHub. |
| **Enlaces relativos** (`docs/endpoints.md`, `docs/ejemplo_consumo.gs`) | Clicables desde el README. |
| **Enlaces ancla** en TOC → `#...` | Saltos internos funcionales. |
| **Sub-bloques de comandos en listas numeradas** (sección *Actualización*) | El `git add` ya no rompe la numeración. |
| **Espaciado uniforme entre secciones** (`---`) | Separación visual clara entre bloques temáticos. |
| **`text` en el árbol de directorios** | Evita que algunos renderizadores interpreten `├──` como viñetas. |

> **Extra:** si más adelante el README crece, considera mover las notas largas (`.nojekyll`, límites de Apps Script, cache-busting) a `docs/` y dejar solo el resumen + enlace.