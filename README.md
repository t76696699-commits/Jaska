node_modules/
dist/
.env
.DS_Store

interface User {
  id: number;
  name: string;
  email: string;
  passwordHash: string;
  createdAt: string;
}

interface Issue {
  id: number;
  title: string;
  description: string;
  status: 'open' | 'in_progress' | 'closed';
  assigneeId: number | null;
  reporterId: number;
  createdAt: string;
}

export type { User, Issue };

{
  "compilerOptions": {
    "target": "ES2020",
    "module": "commonjs",
    "strict": true,
    "esModuleInterop": true,
    "skipLibCheck": true,
    "forceConsistentCasingInFileNames": true,
    "outDir": "./dist"
  },
  "include": ["src/**/*", "../shared/**/*"]
}
{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "moduleResolution": "bundler",
    "strict": true,
    "jsx": "react-jsx",
    "skipLibCheck": true
  },
  "include": ["src/**/*", "../shared/**/*"]
}

{
  "compilerOptions": {
    "target": "ES2020",
    "useDefineForClassFields": true,
    "module": "ESNext",
    "lib": ["ES2020", "DOM", "DOM.Iterable"],
    "moduleResolution": "bundler",
    "strict": true,
    "jsx": "react-jsx",
    "skipLibCheck": true
  },
  "include": ["src/**/*", "../shared/**/*"]
}

# IssueForge

## Loyiha haqida
IssueForge — jamoaviy xato va vazifalarni kuzatuvchi (issue tracker) to'liq stack dastur.

## Texnologiyalar
- **Backend:** Node.js, Express, TypeScript, PostgreSQL
- **Frontend:** React, Redux Toolkit, TypeScript
- **Shared:** TypeScript interfaces

## Umumiy Types Strategiyasi
Backend va frontend loyihalari `shared/types.ts` faylidagi bitta umumiy interfeyslardan foydalanadi. Bu yondashuv ma'lumotlar tuzilmasining har ikkala tomonda bir xil bo'lishini va compile vaqtida xatolarni aniqlashni ta'minlaydi.

## Holat
- [x] Loyihalash va repo skeleton
- [ ] Backend API
- [ ] PostgreSQL CRUD
- [ ] Autentifikatsiya
- [ ] React frontend
- [ ] Testing
- [ ] Deploy
