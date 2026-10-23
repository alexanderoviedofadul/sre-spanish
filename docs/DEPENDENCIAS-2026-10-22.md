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
