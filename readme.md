BD_SIGA_PAGE_API_RES/
│
├── .nojekyll                       ← OBLIGATORIO (ver nota abajo)
├── index.html                      ← Landing / documentación visual
├── README.md                       ← Documentación para GitHub
├── LICENSE                         ← Opcional
│
├── data/                           ← Datos "públicos" que consumirá el front/back
│   ├── catalogo_optimizado.json    ← Tu archivo principal (~53 MB)
│   ├── meta.json                   ← Metadatos: versión, fecha, total items
│   ├── tipos.json                  ← ["02","03","04",...]
│   └── unidades.json               ← ["UNIDAD","PAR","METRO",...]
│
├── api/                            ← Endpoints "limpios" (URLs más bonitas)
│   └── v1/
│       ├── catalog.json            → alias de data/catalogo_optimizado.json
│       ├── meta.json               → alias de data/meta.json
│       ├── types.json              → alias de data/tipos.json
│       └── units.json              → alias de data/unidades.json
│
└── docs/                           ← Documentación extendida (opcional)
    ├── endpoints.md
    └── ejemplo_consumo.gs          ← Ejemplo de uso desde Apps Script