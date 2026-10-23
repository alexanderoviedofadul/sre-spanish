# Dependencias vulnerables del monorepo — 22-oct-2026

Corrección de los avisos CRITICAL/HIGH de OSV.dev sobre `pnpm-lock.yaml`, cada paquete a la versión más baja que
los corrige. El lock se regeneró con el pnpm que declara `packageManager` (10.12.1, por corepack).

| Paquete | De → a | Cómo |
|---|---|---|
| `vitest` / `@vitest/coverage-v8` | 3.2.3 / 3.2.2 → 3.2.6 | raíz `devDependencies` |
| `@modelcontextprotocol/sdk` | 1.12.1 · 1.13.0 · 1.14.0 → 1.32.0 | `packages/core`, `packages/sdk`, `packages/cli` `^1.32.0` |
| `@faker-js/faker` | 9.8.0 → 10.5.0 | `packages/core` (solo lo usa un test: `faker.string.uuid()`, que sigue en la 10) |
| `@grpc/grpc-js` | 1.7.3 → 1.13.6 | `overrides` (llega por `@zilliz/milvus2-sdk-node`) |
| `protobufjs` | 7.5.3 → 7.5.5 | `overrides` |
| `fast-xml-parser` | 4.4.1 → 4.5.4 | `overrides` (AWS SDK) |
| `form-data` | 4.0.3 → 4.0.4 | `overrides` |
| `proxy-addr` | 2.0.7 → 2.0.8 | `overrides` (express 5 del SDK MCP) |
| `tinypool` | 1.1.0 → 2.1.2 | `overrides` (vitest) |
| `@isaacs/brace-expansion` | 5.0.0 → 5.0.1 | `overrides` |
| `@xmldom/xmldom` | 0.8.10 → 0.8.15 | `overrides` (mammoth) |

Los `overrides` van en `pnpm-workspace.yaml`, junto al que ya había (`list`).

## Pruebas

- Los siete módulos del SDK MCP que importan `packages/*/src` (`client/index.js`, `client/sse.js`,
  `client/streamableHttp.js`, `server/index.js`, `server/sse.js`, `server/stdio.js`, `types.js`) se resuelven y cargan
  con la 1.32.0.
- `vitest run packages/core/tests/unit/core` falla igual **antes y después** del cambio (comprobado volviendo al lock
  anterior): `JSONFileVault` recibe una ruta vacía porque el entorno de pruebas no tiene configurada la bóveda. No es
  de las dependencias.
- No se compiló el monorepo (`pnpm -r build`): en este servidor nunca se ha compilado y la API en español
  (`api-example/`) no lo usa; tiene su propio `package-lock.json`.

## Lo que queda

`pnpm audit` (base de datos de GitHub, más amplia que la lista de OSV que se corrigió) sigue dando 93 avisos high en el
monorepo, casi todos transitivos de las dependencias de SmythOS (axios, minimatch, js-yaml, lodash, vite…). Cerrarlos
pide actualizar el monorepo contra el upstream de SmythOS, no parchear el lock.

## API en español (`api-example/package-lock.json`) — segunda pasada

La API que corre de verdad (`api-example/server-enhanced.js`) tiene su propio lock, que no estaba en la primera lista.
`npm audit --omit=dev` daba 1 critical y 11 high; cada paquete a la versión más baja que los corrige todos:

| Paquete | De → a | Cómo |
|---|---|---|
| `axios` | 1.11.0 → 1.20.0 | directa `^1.20.0` (con 1.16.0 entraba en avisos high de 1.15–1.19) |
| `compression` | 1.8.1 → 1.8.2 | directa |
| `proxy-addr` | 2.0.7 → 2.0.8 | `overrides` (critical) |
| `path-to-regexp` (0.x) | 0.1.12 → 0.1.13 | `overrides` |
| `form-data` | 4.0.4 → 4.0.6 | `overrides` |
| `js-yaml` (4.x) | 4.1.0 → 4.3.2 | `overrides` |
| `jws` (3.x) | 3.2.2 → 3.2.3 | `overrides` |
| `lodash` | 4.17.21 → 4.18.0 | `overrides` |
| `minimatch` (3.x) / `brace-expansion` (1.x) | 3.1.2 → 3.1.4 / 1.1.12 → 1.1.20 | `overrides` |
| `validator` | 13.12.0 → 13.15.22 | `overrides` |

Queda: 8 avisos **moderate** (body-parser, qs, morgan, brace-expansion < 1.1.21). Instalado con `npm ci --omit=dev`
y reiniciado; las respuestas de `/api/asistentes`, `/api/proveedores`, `/api/modelos`, `/api-docs.json`, `/docs/`,
`/redoc` y `/docs-ui/` son idénticas byte a byte a las de antes; `/api/estado` y `/api/agentes` solo cambian en
`uptime`/memoria y la fecha de los agentes por defecto, y la validación de `POST /api/agentes` sigue devolviendo 400.
