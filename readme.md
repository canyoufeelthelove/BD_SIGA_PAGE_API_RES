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