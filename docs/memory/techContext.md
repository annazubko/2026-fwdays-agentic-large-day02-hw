# Tech Context: Excalidraw

## Core Stack

| Layer            | Technology         | Version |
| ---------------- | ------------------ | ------- |
| Language         | TypeScript         | 5.9.3   |
| UI Framework     | React              | 19.0.0  |
| Build Tool       | Vite               | 5.0.12  |
| Test Runner      | Vitest             | 3.0.6   |
| Package Manager  | Yarn Classic       | 1.22.22 |
| State Management | Jotai              | 2.11.0  |
| Canvas Drawing   | RoughJS            | 4.6.4   |
| Freehand Drawing | Perfect Freehand   | 1.2.0   |
| Collaboration    | Socket.io-client   | 4.7.2   |
| Backend          | Firebase           | 11.3.1  |
| CSS              | SASS + CSS Modules | 1.51.0  |

## Runtime Requirements

- **Node.js** >= 18.0.0
- **Browser**: Modern browsers with Canvas API support
- **Dev Server Port**: 3001 (configurable via `VITE_APP_PORT`)

## Monorepo Structure

```
excalidraw-monorepo/
├── excalidraw-app/          # Main web application (v1.0.0)
├── packages/
│   ├── common/              # @excalidraw/common — shared utilities, constants (v0.18.0)
│   ├── element/             # @excalidraw/element — element management, transforms (v0.18.0)
│   ├── excalidraw/          # @excalidraw/excalidraw — main React component (v0.18.0)
│   ├── math/                # @excalidraw/math — geometry, vectors, points (v0.18.0)
│   └── utils/               # @excalidraw/utils — export, import utilities (v0.1.2)
├── examples/                # Integration examples (Next.js, browser script)
├── firebase-project/        # Firebase configuration
└── scripts/                 # Build and utility scripts
```

## Key Dependencies

- **Radix UI** (1.4.3) — accessible UI primitives
- **Pako** (2.0.3) — data compression for sharing
- **png-chunks-extract/encode** — PNG metadata for scene embedding
- **Vite PWA plugin** — Progressive Web App support

## Environment Configuration

- `.env.development` — local dev (localhost services, port 3001)
- `.env.production` — production endpoints (excalidraw.com)
- `.env.local` — personal overrides (not committed)

Key env vars: `VITE_APP_WS_SERVER_URL`, `VITE_APP_FIREBASE_CONFIG`, `VITE_APP_AI_BACKEND`

## Build Pipeline

1. TypeScript compilation (strict mode)
2. Vite bundling with React plugin
3. SASS → CSS compilation
4. SVG inline processing (vite-plugin-svgr)
5. PWA manifest generation (dev/prod)

**Package build order** (dependency chain): `common` → `math` → `element` → `excalidraw`

## Quality Tools

- **ESLint** with custom Excalidraw config
- **Prettier** for formatting
- **Husky + lint-staged** for pre-commit hooks
- **CodeRabbit** for automated PR review
- **Vitest** with jsdom for unit/integration tests

## Related Documentation

- [Architecture](../technical/architecture.md)
- [Development Setup](../technical/dev-setup.md)
- [Product Requirements](../product/PRD.md)
- [Domain Glossary](../product/domain-glossary.md)
