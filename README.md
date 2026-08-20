// ════════════════════════════════════════════════════════════════════
// 7-BOSQICH (CAPSTONE YAKUNI): Deploy va path alias xatosi
// ════════════════════════════════════════════════════════════════════

// ─────────────────────────────────────────────────────────────────────
// 1) backend/tsconfig.json - path alias sozlash
// ─────────────────────────────────────────────────────────────────────

// {
//   "compilerOptions": {
//     "baseUrl": ".",
//     "paths": { "@shared/*": ["../shared/*"] }
//   }
// }

import { Issue } from '@shared/types';

// ─────────────────────────────────────────────────────────────────────
// 2) package.json - dev va TO'G'RI build (izohda - JSON, kod emas)
// ─────────────────────────────────────────────────────────────────────

// {
//   "scripts": {
//     "dev": "ts-node -r tsconfig-paths/register src/server.ts",
//     "build": "tsc && tsc-alias"
//   }
// }
//
// npm install -D tsconfig-paths tsc-alias

// ─────────────────────────────────────────────────────────────────────
// 3) Ataylab xato - tsc-alias'siz build (izohda)
// ─────────────────────────────────────────────────────────────────────

// {
//   "scripts": { "build": "tsc" }        // tsc-alias YO'Q!
// }
//
// $ npm run build   -> tsc: 0 xato
// $ cat dist/server.js
//   const types_1 = require("@shared/types");   // o'zgartirilmagan!
// $ node dist/server.js
//   -> Error: Cannot find module '@shared/types'
