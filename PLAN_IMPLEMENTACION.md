# 🚀 Plan de Implementación - SGV BRASA
## De Cero a Producción en Vercel

**Versión:** 1.0
**Fecha:** 24/11/2025
**Objetivo:** Desplegar MVP completo en Vercel en 6-8 semanas

---

## 📋 Resumen Ejecutivo

### Objetivo
Desarrollar e implementar el Sistema de Gestión de Ventas BRASA desde el setup inicial hasta el despliegue en producción en Vercel, siguiendo las especificaciones definidas en los PRDs.

### Duración Estimada
**6-8 semanas** (1 desarrollador full-time) o **4-5 semanas** (equipo de 2-3 desarrolladores)

### Tecnologías Core
- **Frontend:** Next.js 14 + React 18 + TypeScript + shadcn/ui + Tailwind CSS
- **Backend:** Next.js API Routes + Server Actions
- **Database:** Supabase (PostgreSQL)
- **Hosting:** Vercel
- **Version Control:** GitHub

---

## 🎯 Fases del Proyecto

```
FASE 0: Preparación                    (3-5 días)
FASE 1: Setup e Infraestructura        (3-5 días)
FASE 2: Design System & Componentes    (5-7 días)
FASE 3: Módulos Base                   (7-10 días)
FASE 4: Módulos Core                   (10-14 días)
FASE 5: Dashboard & Integraciones      (5-7 días)
FASE 6: Testing & QA                   (3-5 días)
FASE 7: Deploy & Go Live               (2-3 días)
```

---

## FASE 0: Preparación (3-5 días)

### Objetivos
- Configurar entorno de desarrollo
- Crear cuentas necesarias
- Planificar arquitectura de Git

### Checklist

#### 0.1 Cuentas y Servicios (Día 1)

```bash
# Crear cuentas si no existen:
□ GitHub Account (para repositorio)
□ Vercel Account (para hosting) - vincular con GitHub
□ Supabase Account (para base de datos)
```

**Enlaces:**
- GitHub: https://github.com/signup
- Vercel: https://vercel.com/signup
- Supabase: https://supabase.com/dashboard

#### 0.2 Entorno de Desarrollo (Día 1-2)

```bash
# Instalar herramientas necesarias

# 1. Node.js (versión 18.17 o superior)
node --version  # Debe ser >= 18.17

# Si no está instalado:
# macOS: brew install node
# Windows: https://nodejs.org/

# 2. Git
git --version

# 3. Visual Studio Code (recomendado)
# https://code.visualstudio.com/

# 4. Extensiones VSCode recomendadas:
- ES7+ React/Redux/React-Native snippets
- Tailwind CSS IntelliSense
- Prettier - Code formatter
- ESLint
- GitLens
- Database Client (para Supabase)
```

#### 0.3 Crear Repositorio GitHub (Día 2)

```bash
# En GitHub.com:
1. Crear nuevo repositorio "sgv-brasa"
2. Descripción: "Sistema de Gestión de Ventas BRASA"
3. Private/Public según preferencia
4. NO inicializar con README (lo haremos localmente)
5. Copiar URL del repositorio
```

#### 0.4 Estructura de Proyecto (Día 2-3)

```bash
# Crear estructura de carpetas local
mkdir sgv-brasa
cd sgv-brasa

# Crear .gitignore inicial
cat > .gitignore << EOF
node_modules/
.next/
.env*.local
.vercel
.DS_Store
*.log
EOF

# Inicializar Git
git init
git add .gitignore
git commit -m "Initial commit: Add .gitignore"
git branch -M main
git remote add origin [URL_TU_REPOSITORIO]
git push -u origin main
```

#### 0.5 Planificación de Branches (Día 3)

```bash
# Estrategia de branches
main          # Producción (protegida)
└── develop   # Desarrollo principal
    ├── feature/setup
    ├── feature/design-system
    ├── feature/auth
    ├── feature/clientes
    ├── feature/productos
    ├── feature/ventas
    ├── feature/pagos
    └── feature/dashboard

# Crear branch develop
git checkout -b develop
git push -u origin develop
```

#### 0.6 Documentación Inicial (Día 3)

```bash
# Copiar PRDs al repositorio
mkdir -p docs/prds
cp [RUTA_PRDS]/*.md docs/prds/

# Crear README.md básico
cat > README.md << EOF
# SGV BRASA - Sistema de Gestión de Ventas

Sistema web moderno para gestión de ventas, pagos y cobranzas.

## Stack Tecnológico
- Next.js 14 + React 18 + TypeScript
- Tailwind CSS + shadcn/ui
- Supabase (PostgreSQL)
- Vercel

## Documentación
Ver carpeta \`docs/prds/\` para PRDs completos.

## Setup
Ver \`docs/SETUP.md\`

## Estado
🚧 En desarrollo
EOF

git add .
git commit -m "docs: Add initial documentation"
git push
```

---

## FASE 1: Setup e Infraestructura (3-5 días)

### Objetivos
- Crear proyecto Next.js
- Configurar Supabase
- Setup inicial de Vercel
- Configurar TypeScript, ESLint, Prettier

### Checklist

#### 1.1 Crear Proyecto Next.js (Día 1)

```bash
# Crear branch feature/setup
git checkout -b feature/setup

# Crear proyecto Next.js 14 con TypeScript
npx create-next-app@latest . --typescript --tailwind --app --src-dir --import-alias "@/*"

# Responder las preguntas:
✔ Would you like to use TypeScript? Yes
✔ Would you like to use ESLint? Yes
✔ Would you like to use Tailwind CSS? Yes
✔ Would you like to use `src/` directory? Yes
✔ Would you like to use App Router? Yes
✔ Would you like to customize the default import alias? Yes (@/*)

# Instalar dependencias core
npm install zod react-hook-form @hookform/resolvers
npm install @supabase/supabase-js @supabase/ssr
npm install next-themes
npm install date-fns
npm install -D @types/node

# Instalar Drizzle ORM
npm install drizzle-orm drizzle-kit
npm install postgres

# Instalar herramientas de desarrollo
npm install -D prettier prettier-plugin-tailwindcss
npm install -D @typescript-eslint/parser @typescript-eslint/eslint-plugin
```

#### 1.2 Configurar Supabase (Día 1-2)

```bash
# 1. Crear proyecto en Supabase
# - Ir a https://supabase.com/dashboard
# - Clic en "New Project"
# - Nombre: sgv-brasa
# - Database Password: [GUARDAR EN LUGAR SEGURO]
# - Region: South America (sa-east-1) o el más cercano
# - Pricing Plan: Free (para empezar)

# 2. Obtener credenciales
# En Project Settings > API:
# - Project URL (NEXT_PUBLIC_SUPABASE_URL)
# - anon/public key (NEXT_PUBLIC_SUPABASE_ANON_KEY)
# - service_role key (SUPABASE_SERVICE_ROLE_KEY)

# 3. Crear archivo .env.local
cat > .env.local << EOF
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxxxxxxxxxxxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...
SUPABASE_SERVICE_ROLE_KEY=eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9...

# Database
DATABASE_URL=postgresql://postgres:[PASSWORD]@db.xxxxxxxxxxxxx.supabase.co:5432/postgres

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
EOF

# 4. Crear archivo .env.example (sin valores sensibles)
cat > .env.example << EOF
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Database
DATABASE_URL=

# App
NEXT_PUBLIC_APP_URL=
EOF
```

#### 1.3 Ejecutar Migraciones SQL en Supabase (Día 2)

```bash
# 1. Ir a Supabase Dashboard > SQL Editor
# 2. Crear nueva query
# 3. Copiar contenido de docs/prds/arquitectura.md - Sección 3.2
# 4. Ejecutar script completo (tablas + triggers + RLS)

# Verificar que se crearon las tablas:
# - clientes
# - productos
# - ventas
# - pagos

# Verificar triggers:
# - update_venta_monto_pagado
# - check_pago_no_excede_venta
# - update_updated_at (en todas las tablas)

# Verificar RLS policies:
# - Políticas activas en las 4 tablas
```

**Script SQL a ejecutar:**
```sql
-- Ver docs/prds/arquitectura.md - Sección 3.2
-- Copiar y ejecutar:
-- 1. Enable UUID extension
-- 2. CREATE TABLE clientes
-- 3. CREATE TABLE productos
-- 4. CREATE TABLE ventas
-- 5. CREATE TABLE pagos
-- 6. CREATE triggers
-- 7. Enable RLS (docs/prds/arquitectura.md - Sección 3.3)
```

#### 1.4 Configurar Estructura del Proyecto (Día 2-3)

```bash
# Crear estructura de carpetas
mkdir -p src/components/ui
mkdir -p src/components/layout
mkdir -p src/components/clientes
mkdir -p src/components/productos
mkdir -p src/components/ventas
mkdir -p src/components/pagos
mkdir -p src/components/dashboard
mkdir -p src/lib/db
mkdir -p src/lib/supabase
mkdir -p src/lib/utils
mkdir -p src/lib/validations
mkdir -p src/app/api
mkdir -p src/app/actions
mkdir -p src/types
mkdir -p src/hooks
mkdir -p public/images

# Crear archivos base de configuración
```

**src/lib/supabase/client.ts:**
```typescript
import { createBrowserClient } from '@supabase/ssr';

export function createClient() {
  return createBrowserClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!
  );
}
```

**src/lib/supabase/server.ts:**
```typescript
import { createServerClient, type CookieOptions } from '@supabase/ssr';
import { cookies } from 'next/headers';

export function createClient() {
  const cookieStore = cookies();

  return createServerClient(
    process.env.NEXT_PUBLIC_SUPABASE_URL!,
    process.env.NEXT_PUBLIC_SUPABASE_ANON_KEY!,
    {
      cookies: {
        get(name: string) {
          return cookieStore.get(name)?.value;
        },
      },
    }
  );
}
```

**src/lib/utils.ts:**
```typescript
import { type ClassValue, clsx } from 'clsx';
import { twMerge } from 'tailwind-merge';

export function cn(...inputs: ClassValue[]) {
  return twMerge(clsx(inputs));
}

export function formatCurrency(amount: number): string {
  return new Intl.NumberFormat('es-PE', {
    style: 'currency',
    currency: 'PEN',
  }).format(amount);
}

export function formatDate(date: Date | string): string {
  return new Intl.DateTimeFormat('es-PE', {
    day: '2-digit',
    month: '2-digit',
    year: 'numeric',
  }).format(new Date(date));
}
```

#### 1.5 Configurar Prettier y ESLint (Día 3)

**prettier.config.js:**
```javascript
module.exports = {
  plugins: ['prettier-plugin-tailwindcss'],
  semi: true,
  trailingComma: 'es5',
  singleQuote: true,
  printWidth: 80,
  tabWidth: 2,
  useTabs: false,
};
```

**.eslintrc.json:**
```json
{
  "extends": [
    "next/core-web-vitals",
    "plugin:@typescript-eslint/recommended"
  ],
  "rules": {
    "@typescript-eslint/no-unused-vars": "warn",
    "@typescript-eslint/no-explicit-any": "warn"
  }
}
```

**package.json (agregar scripts):**
```json
{
  "scripts": {
    "dev": "next dev",
    "build": "next build",
    "start": "next start",
    "lint": "next lint",
    "format": "prettier --write \"**/*.{ts,tsx,md}\"",
    "type-check": "tsc --noEmit"
  }
}
```

#### 1.6 Setup Vercel (Día 3)

```bash
# 1. Instalar Vercel CLI
npm install -g vercel

# 2. Login a Vercel
vercel login

# 3. Link project (NO desplegar aún)
vercel link

# Responder:
# - Set up and deploy? N (No)
# - Link to existing project? N
# - Project name: sgv-brasa
# - In which directory? ./
```

#### 1.7 Commit y Push (Día 3)

```bash
git add .
git commit -m "feat: Initial project setup with Next.js, Supabase, and Vercel"
git push -u origin feature/setup

# Crear Pull Request en GitHub
# - De: feature/setup
# - A: develop
# - Título: "Initial Setup: Next.js + Supabase + Vercel"
# - Merge cuando esté aprobado
```

---

## FASE 2: Design System & Componentes Base (5-7 días)

### Objetivos
- Implementar shadcn/ui
- Crear componentes base
- Setup de tema y dark mode
- Implementar layout principal

### Checklist

#### 2.1 Instalar shadcn/ui (Día 1)

```bash
git checkout develop
git pull
git checkout -b feature/design-system

# Inicializar shadcn/ui
npx shadcn-ui@latest init

# Responder:
# - Style: Default
# - Base color: Slate
# - CSS variables: Yes
# - Tailwind config: Yes
# - Import alias: @/components
# - RSC: Yes

# Instalar componentes base
npx shadcn-ui@latest add button
npx shadcn-ui@latest add card
npx shadcn-ui@latest add input
npx shadcn-ui@latest add label
npx shadcn-ui@latest add select
npx shadcn-ui@latest add table
npx shadcn-ui@latest add badge
npx shadcn-ui@latest add dropdown-menu
npx shadcn-ui@latest add dialog
npx shadcn-ui@latest add alert-dialog
npx shadcn-ui@latest add form
npx shadcn-ui@latest add toast
npx shadcn-ui@latest add skeleton
npx shadcn-ui@latest add progress
npx shadcn-ui@latest add avatar
npx shadcn-ui@latest add separator

# Instalar dependencias adicionales
npm install lucide-react
npm install sonner
npm install next-themes
npm install geist
```

#### 2.2 Configurar Tailwind y Colores (Día 1)

**tailwind.config.ts:**
```typescript
import type { Config } from 'tailwindcss';

const config: Config = {
  darkMode: ['class'],
  content: [
    './src/pages/**/*.{js,ts,jsx,tsx,mdx}',
    './src/components/**/*.{js,ts,jsx,tsx,mdx}',
    './src/app/**/*.{js,ts,jsx,tsx,mdx}',
  ],
  theme: {
    extend: {
      colors: {
        brand: {
          50: '#f0f9ff',
          500: '#0ea5e9',
          600: '#0284c7',
          700: '#0369a1',
        },
        success: {
          50: '#f0fdf4',
          500: '#22c55e',
          700: '#15803d',
        },
        warning: {
          50: '#fffbeb',
          500: '#eab308',
          700: '#a16207',
        },
        error: {
          50: '#fef2f2',
          500: '#ef4444',
          700: '#b91c1c',
        },
      },
    },
  },
  plugins: [require('tailwindcss-animate')],
};

export default config;
```

**src/app/globals.css:**
```css
@tailwind base;
@tailwind components;
@tailwind utilities;

@layer base {
  :root {
    --background: 0 0% 100%;
    --foreground: 0 0% 10%;
    --brand: 199 89% 48%;
    --success: 142 71% 45%;
    --warning: 43 96% 56%;
    --error: 0 84% 60%;
  }

  .dark {
    --background: 0 0% 5%;
    --foreground: 0 0% 95%;
  }
}
```

#### 2.3 Setup Theme Provider (Día 1)

**src/components/providers/theme-provider.tsx:**
```typescript
'use client';

import { ThemeProvider as NextThemesProvider } from 'next-themes';
import { type ThemeProviderProps } from 'next-themes/dist/types';

export function ThemeProvider({ children, ...props }: ThemeProviderProps) {
  return <NextThemesProvider {...props}>{children}</NextThemesProvider>;
}
```

**src/app/layout.tsx:**
```typescript
import type { Metadata } from 'next';
import { GeistSans } from 'geist/font/sans';
import { ThemeProvider } from '@/components/providers/theme-provider';
import { Toaster } from 'sonner';
import './globals.css';

export const metadata: Metadata = {
  title: 'SGV BRASA - Sistema de Gestión de Ventas',
  description: 'Sistema moderno de gestión de ventas y cobranzas',
};

export default function RootLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <html lang="es" suppressHydrationWarning>
      <body className={GeistSans.className}>
        <ThemeProvider
          attribute="class"
          defaultTheme="system"
          enableSystem
          disableTransitionOnChange
        >
          {children}
          <Toaster />
        </ThemeProvider>
      </body>
    </html>
  );
}
```

#### 2.4 Crear Layout Principal (Día 2)

**src/components/layout/sidebar.tsx:**
```typescript
'use client';

import Link from 'next/link';
import { usePathname } from 'next/navigation';
import { cn } from '@/lib/utils';
import {
  LayoutDashboard,
  ShoppingCart,
  CreditCard,
  Users,
  Package,
} from 'lucide-react';

const navigation = [
  { name: 'Dashboard', href: '/dashboard', icon: LayoutDashboard },
  { name: 'Ventas', href: '/ventas', icon: ShoppingCart },
  { name: 'Pagos', href: '/pagos', icon: CreditCard },
  { name: 'Clientes', href: '/clientes', icon: Users },
  { name: 'Productos', href: '/productos', icon: Package },
];

export function Sidebar() {
  const pathname = usePathname();

  return (
    <aside className="hidden w-64 border-r bg-background lg:block">
      <div className="flex h-full flex-col">
        {/* Logo */}
        <div className="flex h-16 items-center border-b px-6">
          <h1 className="text-xl font-bold">SGV BRASA</h1>
        </div>

        {/* Navigation */}
        <nav className="flex-1 space-y-1 p-4">
          {navigation.map((item) => {
            const isActive = pathname === item.href;
            return (
              <Link
                key={item.name}
                href={item.href}
                className={cn(
                  'flex items-center gap-3 rounded-lg px-3 py-2 text-sm font-medium transition-colors',
                  isActive
                    ? 'bg-brand-50 text-brand-600 dark:bg-brand-950 dark:text-brand-400'
                    : 'text-neutral-700 hover:bg-neutral-100 dark:text-neutral-300 dark:hover:bg-neutral-800'
                )}
              >
                <item.icon className="h-5 w-5" />
                {item.name}
              </Link>
            );
          })}
        </nav>
      </div>
    </aside>
  );
}
```

**src/components/layout/header.tsx:**
```typescript
'use client';

import { ThemeToggle } from '@/components/theme-toggle';

export function Header() {
  return (
    <header className="sticky top-0 z-40 border-b bg-background">
      <div className="flex h-16 items-center justify-between px-6">
        <div className="flex items-center gap-2">
          {/* Breadcrumb aquí */}
        </div>

        <div className="flex items-center gap-4">
          <ThemeToggle />
          {/* User menu aquí */}
        </div>
      </div>
    </header>
  );
}
```

**src/components/theme-toggle.tsx:**
```typescript
'use client';

import { Moon, Sun } from 'lucide-react';
import { useTheme } from 'next-themes';
import { Button } from '@/components/ui/button';

export function ThemeToggle() {
  const { theme, setTheme } = useTheme();

  return (
    <Button
      variant="ghost"
      size="icon"
      onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}
      aria-label="Toggle theme"
    >
      <Sun className="h-5 w-5 rotate-0 scale-100 transition-all dark:-rotate-90 dark:scale-0" />
      <Moon className="absolute h-5 w-5 rotate-90 scale-0 transition-all dark:rotate-0 dark:scale-100" />
    </Button>
  );
}
```

**src/app/(dashboard)/layout.tsx:**
```typescript
import { Sidebar } from '@/components/layout/sidebar';
import { Header } from '@/components/layout/header';

export default function DashboardLayout({
  children,
}: {
  children: React.ReactNode;
}) {
  return (
    <div className="flex h-screen overflow-hidden">
      <Sidebar />
      <div className="flex flex-1 flex-col overflow-hidden">
        <Header />
        <main className="flex-1 overflow-y-auto bg-neutral-50 dark:bg-neutral-900">
          {children}
        </main>
      </div>
    </div>
  );
}
```

#### 2.5 Crear Componentes Reutilizables (Día 3-4)

**src/components/ui/data-table.tsx:**
```typescript
// Componente de tabla reutilizable
// Ver docs/prds/frontend-design.md para implementación completa
```

**src/components/ui/page-header.tsx:**
```typescript
interface PageHeaderProps {
  title: string;
  description?: string;
  action?: React.ReactNode;
}

export function PageHeader({ title, description, action }: PageHeaderProps) {
  return (
    <div className="flex items-center justify-between">
      <div>
        <h1 className="text-3xl font-bold tracking-tight">{title}</h1>
        {description && (
          <p className="mt-1 text-neutral-500">{description}</p>
        )}
      </div>
      {action && <div>{action}</div>}
    </div>
  );
}
```

#### 2.6 Testing y Commit (Día 4-5)

```bash
# Probar el proyecto
npm run dev
# Abrir http://localhost:3000

# Verificar:
# - Layout principal renderiza correctamente
# - Sidebar navigation funciona
# - Theme toggle funciona (light/dark mode)
# - Componentes shadcn/ui se ven correctamente

# Ejecutar linter y type check
npm run lint
npm run type-check

# Commit y push
git add .
git commit -m "feat: Implement design system with shadcn/ui and layout"
git push -u origin feature/design-system

# Crear PR en GitHub
# Merge a develop cuando esté aprobado
```

---

## FASE 3: Módulos Base - Clientes y Productos (7-10 días)

### Objetivos
- Implementar autenticación
- CRUD de Clientes
- CRUD de Productos

### Checklist

#### 3.1 Implementar Autenticación (Día 1-2)

```bash
git checkout develop
git pull
git checkout -b feature/auth
```

**src/app/auth/login/page.tsx:**
```typescript
// Página de login con Supabase Auth
// Ver docs/prds/arquitectura.md para detalles
```

**Middleware de autenticación:**
```typescript
// src/middleware.ts
import { createServerClient } from '@supabase/ssr';
import { NextResponse, type NextRequest } from 'next/server';

export async function middleware(request: NextRequest) {
  // Verificar sesión
  // Redirigir a /auth/login si no está autenticado
}

export const config = {
  matcher: ['/((?!auth|_next/static|_next/image|favicon.ico).*)'],
};
```

#### 3.2 Módulo CLIENTES (Día 3-5)

```bash
git checkout develop
git pull
git checkout -b feature/clientes
```

**Implementar según [prds/clientes.md](prds/clientes.md):**

1. **Tipos TypeScript:**
```typescript
// src/types/cliente.ts
export interface Cliente {
  id: string;
  cliente_id: string;
  nombre: string;
  email?: string;
  telefono?: string;
  // ... resto de campos
}
```

2. **Validaciones Zod:**
```typescript
// src/lib/validations/cliente.ts
import { z } from 'zod';

export const clienteSchema = z.object({
  nombre: z.string().min(2),
  email: z.string().email().optional(),
  // ... resto de validaciones
});
```

3. **Server Actions:**
```typescript
// src/app/actions/clientes.ts
'use server';

export async function createCliente(data: CreateClienteInput) {
  // Implementación
}
```

4. **Componentes:**
```typescript
// src/app/(dashboard)/clientes/page.tsx - Lista
// src/app/(dashboard)/clientes/[id]/page.tsx - Detalle
// src/app/(dashboard)/clientes/nuevo/page.tsx - Crear
// src/components/clientes/cliente-form.tsx - Formulario
// src/components/clientes/cliente-table.tsx - Tabla
```

**Testing:**
```bash
# Probar:
# - Crear cliente nuevo
# - Listar clientes
# - Ver detalle de cliente
# - Editar cliente
# - Eliminar cliente
# - Búsqueda y filtros

git add .
git commit -m "feat: Implement Clientes module with CRUD operations"
git push -u origin feature/clientes
# Crear PR y merge
```

#### 3.3 Módulo PRODUCTOS (Día 6-8)

```bash
git checkout develop
git pull
git checkout -b feature/productos
```

**Implementar según [prds/productos.md](prds/productos.md):**

Similar estructura a Clientes:
1. Tipos TypeScript
2. Validaciones Zod
3. Server Actions
4. Páginas y componentes

**Testing:**
```bash
# Probar CRUD completo de productos
git add .
git commit -m "feat: Implement Productos module with CRUD operations"
git push -u origin feature/productos
# Crear PR y merge
```

---

## FASE 4: Módulos Core - Ventas y Pagos (10-14 días)

### Objetivos
- Implementar módulo VENTAS
- Implementar módulo PAGOS
- Actualización automática de estados

### Checklist

#### 4.1 Módulo VENTAS (Día 1-7)

```bash
git checkout develop
git pull
git checkout -b feature/ventas
```

**Implementar según [prds/ventas.md](prds/ventas.md):**

**Complejidad:** Este es el módulo más complejo

1. **Tipos y Validaciones:**
```typescript
// src/types/venta.ts
export interface Venta {
  id: string;
  venta_id: string;
  cliente_id: string;
  producto_id: string;
  monto_total: number;
  tipo_pago: 'contado' | 'cuotas';
  num_cuotas: number;
  monto_pagado: number;
  saldo_pendiente: number;
  estado: 'PAGADO' | 'PENDIENTE';
  // ... resto
}
```

2. **Server Actions con lógica compleja:**
```typescript
// src/app/actions/ventas.ts
'use server';

export async function createVenta(data: CreateVentaInput) {
  // 1. Validar datos
  // 2. Generar venta_id (V-2024-001)
  // 3. Obtener producto_nombre (snapshot)
  // 4. Calcular monto_cuota
  // 5. Insertar en BD
  // 6. Revalidar paths
}
```

3. **Formulario Complejo:**
```typescript
// src/components/ventas/venta-form.tsx
// - Autocomplete de clientes
// - Autocomplete de productos
// - Cálculo automático de cuotas
// - Validación condicional (contado vs cuotas)
```

4. **Vista Detalle:**
```typescript
// src/app/(dashboard)/ventas/[id]/page.tsx
// - Información de venta
// - Progreso de pago visual
// - Historial de pagos
// - Acciones (registrar pago, editar, eliminar)
```

**Testing exhaustivo:**
```bash
# Casos de prueba:
# 1. Crear venta al contado
# 2. Crear venta en cuotas (2, 3, 5 cuotas)
# 3. Ver detalle de venta
# 4. Editar venta sin pagos
# 5. Intentar editar venta con pagos (debe fallar)
# 6. Filtrar por estado, tipo de pago
# 7. Búsqueda de ventas

git add .
git commit -m "feat: Implement Ventas module with complete CRUD"
git push -u origin feature/ventas
# Crear PR y merge
```

#### 4.2 Módulo PAGOS (Día 8-12)

```bash
git checkout develop
git pull
git checkout -b feature/pagos
```

**Implementar según [prds/pagos.md](prds/pagos.md):**

**Complejidad:** Incluye actualización automática de ventas

1. **Server Actions con Triggers:**
```typescript
// src/app/actions/pagos.ts
'use server';

export async function createPago(data: CreatePagoInput) {
  // 1. Validar que venta existe
  // 2. Validar que monto no excede saldo
  // 3. Generar pago_id (P-2024-001)
  // 4. Insertar pago
  // 5. ✨ Trigger actualiza venta automáticamente
  // 6. Revalidar paths de venta y pagos
  // 7. Retornar venta actualizada
}
```

2. **Formulario de Pago:**
```typescript
// src/components/pagos/pago-form.tsx
// - Autocomplete de ventas pendientes
// - Mostrar cliente automáticamente
// - Mostrar saldo pendiente
// - Sugerir monto de cuota
// - Seleccionar método de pago
```

3. **Vista desde Venta:**
```typescript
// Botón "Registrar Pago" en detalle de venta
// - Pre-rellenar venta_id
// - Mostrar información de venta
// - Sugerir cuota siguiente
```

**Testing crítico:**
```bash
# Casos de prueba:
# 1. Registrar primer pago de venta en cuotas
# 2. Verificar actualización automática de venta
# 3. Registrar pagos parciales
# 4. Completar venta con último pago
# 5. Verificar cambio de estado a PAGADO
# 6. Intentar exceder saldo (debe fallar)
# 7. Eliminar pago y verificar actualización
# 8. Registrar pago de venta al contado

git add .
git commit -m "feat: Implement Pagos module with automatic venta updates"
git push -u origin feature/pagos
# Crear PR y merge
```

#### 4.3 Testing de Integración (Día 13-14)

```bash
# Flujo completo de prueba:
1. Crear cliente
2. Crear producto
3. Crear venta en 3 cuotas
4. Verificar venta con estado PENDIENTE
5. Registrar pago cuota 1
6. Verificar actualización automática (saldo, monto_pagado)
7. Registrar pago cuota 2
8. Verificar cálculos
9. Registrar pago cuota 3 (último)
10. Verificar estado cambió a PAGADO
11. Verificar venta ya no aparece en pendientes

# Documentar bugs encontrados
# Crear issues en GitHub si es necesario
```

---

## FASE 5: Dashboard & Integraciones (5-7 días)

### Objetivos
- Implementar Dashboard con KPIs
- Integrar gráficos
- Alertas de cobranza

### Checklist

#### 5.1 Instalar Librerías de Charts (Día 1)

```bash
git checkout develop
git pull
git checkout -b feature/dashboard

# Instalar Recharts y Tremor
npm install recharts
npm install @tremor/react
```

#### 5.2 Implementar KPIs (Día 1-2)

**Implementar según [prds/dashboard.md](prds/dashboard.md):**

```typescript
// src/app/actions/dashboard.ts
'use server';

export async function getDashboardKPIs(periodo: string) {
  // Queries para calcular:
  // - Ventas totales
  // - Monto total vendido
  // - Saldo pendiente
  // - Ventas pagadas
  // - Comparación con período anterior
}
```

```typescript
// src/components/dashboard/kpi-card.tsx
interface KPICardProps {
  title: string;
  value: string;
  change?: string;
  trend?: 'up' | 'down';
  icon: LucideIcon;
}
```

#### 5.3 Implementar Gráficos (Día 3-4)

```typescript
// src/components/dashboard/ventas-chart.tsx
// Gráfico de línea: Ventas por período

// src/components/dashboard/estado-chart.tsx
// Gráfico de dona: PAGADO vs PENDIENTE

// src/components/dashboard/productos-chart.tsx
// Gráfico de barras: Top 5 productos
```

#### 5.4 Alertas de Cobranza (Día 5)

```typescript
// src/components/dashboard/alertas-list.tsx
// Lista de ventas pendientes ordenadas por prioridad
```

#### 5.5 Página Dashboard Completa (Día 6)

```typescript
// src/app/(dashboard)/page.tsx
export default async function DashboardPage() {
  const kpis = await getDashboardKPIs('30d');
  const ventasPendientes = await getVentasPendientesPrioritarias();
  // ... resto de datos

  return (
    <div className="space-y-8 p-6">
      {/* KPIs Grid */}
      {/* Alertas */}
      {/* Charts Grid */}
      {/* Recent Activity */}
    </div>
  );
}
```

#### 5.6 Testing y Commit (Día 7)

```bash
# Probar:
# - Dashboard carga rápido (< 2s)
# - KPIs muestran datos correctos
# - Gráficos se renderizan
# - Alertas funcionan
# - Responsive en mobile

git add .
git commit -m "feat: Implement Dashboard with KPIs, charts, and alerts"
git push -u origin feature/dashboard
# Crear PR y merge
```

---

## FASE 6: Testing & QA (3-5 días)

### Objetivos
- Testing funcional completo
- Testing de accesibilidad
- Performance optimization
- Bug fixing

### Checklist

#### 6.1 Testing Funcional (Día 1-2)

**Crear checklist de testing:**

```markdown
### Autenticación
- [ ] Login funciona
- [ ] Logout funciona
- [ ] Sesión persiste

### Clientes
- [ ] Crear cliente
- [ ] Editar cliente
- [ ] Eliminar cliente sin ventas
- [ ] No se puede eliminar cliente con ventas
- [ ] Búsqueda funciona
- [ ] Filtros funcionan

### Productos
- [ ] Crear producto
- [ ] Editar producto
- [ ] Eliminar producto sin ventas
- [ ] Búsqueda funciona

### Ventas
- [ ] Crear venta al contado
- [ ] Crear venta en cuotas
- [ ] Ver detalle de venta
- [ ] Editar venta sin pagos
- [ ] No se puede editar venta con pagos
- [ ] Cálculo de cuotas correcto
- [ ] Filtros funcionan

### Pagos
- [ ] Registrar pago de venta
- [ ] Venta se actualiza automáticamente
- [ ] Estado cambia a PAGADO al completar
- [ ] No se puede exceder saldo
- [ ] Eliminar pago actualiza venta

### Dashboard
- [ ] KPIs muestran datos correctos
- [ ] Gráficos se renderizan
- [ ] Alertas funcionan
- [ ] Links a módulos funcionan

### Responsive
- [ ] Desktop (1920x1080)
- [ ] Tablet (768x1024)
- [ ] Mobile (375x667)
```

#### 6.2 Testing de Accesibilidad (Día 2)

```bash
# Instalar herramientas
npm install -D @axe-core/react

# Ejecutar auditoría de Lighthouse
# 1. Abrir Chrome DevTools
# 2. Tab "Lighthouse"
# 3. Categorías: Performance, Accessibility, Best Practices
# 4. Generate report

# Objetivo:
# - Accessibility Score: 100
# - Performance Score: > 90
# - Best Practices: > 90

# Verificar manualmente:
- [ ] Navegación completa por teclado
- [ ] Tab order lógico
- [ ] Focus visible
- [ ] ARIA labels presentes
- [ ] Contraste de colores AAA
- [ ] Screen reader friendly (probar con NVDA/JAWS)
```

#### 6.3 Performance Optimization (Día 3)

```bash
# Analizar bundle size
npm run build
npm run analyze

# Optimizaciones:
- [ ] Imágenes optimizadas (next/image)
- [ ] Lazy loading de componentes pesados
- [ ] Code splitting efectivo
- [ ] Fonts optimizados
- [ ] CSS minificado

# Medir Core Web Vitals
- [ ] LCP < 2.5s
- [ ] FID < 100ms
- [ ] CLS < 0.1
```

#### 6.4 Bug Fixing (Día 4-5)

```bash
# Crear issues en GitHub para cada bug
# Priorizar por severidad:
# - Critical: Bloquea funcionalidad core
# - High: Afecta UX significativamente
# - Medium: Problema menor pero visible
# - Low: Mejora cosmética

# Fixear bugs críticos y high
# Documentar bugs medium/low para post-launch
```

---

## FASE 7: Deploy & Go Live (2-3 días)

### Objetivos
- Deploy a Vercel
- Configurar dominio
- Monitoreo

### Checklist

#### 7.1 Pre-deployment Checklist (Día 1)

```bash
# Verificar configuración
- [ ] .env.example actualizado
- [ ] README.md actualizado con instrucciones
- [ ] Todos los PRs mergeados a main
- [ ] No hay console.logs en producción
- [ ] No hay TODOs críticos
- [ ] Build local exitoso: npm run build
- [ ] Type check exitoso: npm run type-check
- [ ] Lint exitoso: npm run lint

# Verificar Supabase
- [ ] RLS policies activas
- [ ] Triggers funcionando
- [ ] Backup de base de datos configurado
- [ ] Connection pooling configurado
```

#### 7.2 Deploy Inicial a Vercel (Día 1)

```bash
# 1. En GitHub, mergear develop a main
git checkout main
git pull
git merge develop
git push

# 2. Ir a Vercel Dashboard
# https://vercel.com/dashboard

# 3. Import Git Repository
# - Seleccionar repositorio sgv-brasa
# - Framework Preset: Next.js
# - Root Directory: ./
# - Build Command: (default)
# - Output Directory: (default)

# 4. Configurar Environment Variables
# Agregar todas las variables de .env.local:
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=
DATABASE_URL=
NEXT_PUBLIC_APP_URL=https://sgv-brasa.vercel.app

# 5. Deploy
# Esperar a que termine (5-10 minutos)

# 6. Verificar deployment
# Abrir URL de Vercel (sgv-brasa.vercel.app)
# Verificar que la app carga correctamente
```

#### 7.3 Testing en Producción (Día 1-2)

```bash
# Smoke testing en producción:
- [ ] Login funciona
- [ ] Crear cliente
- [ ] Crear producto
- [ ] Crear venta
- [ ] Registrar pago
- [ ] Dashboard muestra datos
- [ ] Dark mode funciona
- [ ] Mobile responsive funciona

# Si hay problemas:
# - Revisar logs en Vercel Dashboard
# - Verificar environment variables
# - Verificar conexión a Supabase
```

#### 7.4 Configurar Dominio Custom (Opcional, Día 2)

```bash
# Si tienes dominio propio:

# 1. En Vercel Dashboard > Project Settings > Domains
# 2. Agregar dominio: tudominio.com
# 3. Configurar DNS según instrucciones de Vercel
# 4. Esperar propagación (hasta 48 horas)
# 5. Actualizar NEXT_PUBLIC_APP_URL en environment variables

# Tipos de DNS records:
# A record: 76.76.21.21
# CNAME: cname.vercel-dns.com
```

#### 7.5 Monitoreo y Analytics (Día 3)

```bash
# 1. Habilitar Vercel Analytics
# En Project Settings > Analytics > Enable

# 2. Configurar Supabase Monitoring
# En Supabase Dashboard > Database > Monitoring
# Revisar:
# - Connection count
# - Query performance
# - Database size

# 3. Setup Error Tracking (opcional)
npm install @sentry/nextjs

# Configurar Sentry para production errors
# Ver: https://docs.sentry.io/platforms/javascript/guides/nextjs/
```

#### 7.6 Documentation Final (Día 3)

**Actualizar README.md:**
```markdown
# SGV BRASA - Sistema de Gestión de Ventas

## 🚀 Producción
URL: https://sgv-brasa.vercel.app

## 📖 Documentación
Ver carpeta `docs/prds/` para documentación completa.

## 🔑 Credenciales de Prueba
- Email: demo@sgvbrasa.com
- Password: [contactar admin]

## 🛠️ Tech Stack
- Next.js 14 + React 18 + TypeScript
- Tailwind CSS + shadcn/ui
- Supabase (PostgreSQL)
- Vercel

## 📊 Status
✅ MVP Deployed
```

**Crear CHANGELOG.md:**
```markdown
# Changelog

## [1.0.0] - 2025-01-XX

### Added
- ✨ Dashboard con KPIs y gráficos
- ✨ Módulo de Clientes (CRUD completo)
- ✨ Módulo de Productos (CRUD completo)
- ✨ Módulo de Ventas (contado y cuotas)
- ✨ Módulo de Pagos con actualización automática
- ✨ Dark mode
- ✨ Responsive design (mobile, tablet, desktop)
- ✨ Accesibilidad WCAG 2.1 Level AA

### Technical
- Next.js 14 con App Router
- Supabase con PostgreSQL
- shadcn/ui + Radix UI
- Vercel deployment
```

---

## 📊 Resumen de Tiempos

### Timeline Completo (1 Desarrollador Full-time)

```
Semana 1: Preparación + Setup + Design System
├── Fase 0: Preparación (3 días)
├── Fase 1: Setup (2 días)
└── Fase 2: Design System (2 días)

Semana 2: Design System + Módulos Base
├── Fase 2: Design System (cont. 3 días)
└── Fase 3: Clientes + Productos (2 días)

Semana 3: Módulos Base (continuación)
└── Fase 3: Clientes + Productos (5 días)

Semana 4-5: Módulos Core (Ventas)
└── Fase 4: Ventas (7-10 días)

Semana 6: Módulos Core (Pagos)
└── Fase 4: Pagos (5 días)

Semana 7: Dashboard + Testing
├── Fase 5: Dashboard (5 días)

Semana 8: Testing + Deploy
├── Fase 6: Testing (3 días)
└── Fase 7: Deploy (2 días)

TOTAL: 6-8 semanas
```

### Timeline con Equipo (2-3 Desarrolladores)

```
Semana 1: Setup paralelo
├── Dev 1: Fase 0-1 (Setup)
└── Dev 2: Fase 2 (Design System)

Semana 2-3: Módulos en paralelo
├── Dev 1: Clientes + Ventas
└── Dev 2: Productos + Pagos

Semana 4: Integración y Dashboard
├── Dev 1: Dashboard
└── Dev 2: Testing

Semana 5: Testing + Deploy
├── Todos: Testing
└── Todos: Deploy

TOTAL: 4-5 semanas
```

---

## 🎯 Criterios de Éxito

### Funcionales
- ✅ Usuario puede crear ventas al contado
- ✅ Usuario puede crear ventas en cuotas
- ✅ Usuario puede registrar pagos
- ✅ Ventas se actualizan automáticamente
- ✅ Dashboard muestra métricas correctas
- ✅ Sistema es responsive

### Técnicos
- ✅ Lighthouse Score > 90
- ✅ Accesibilidad WCAG 2.1 AA
- ✅ Cero errores TypeScript
- ✅ Build exitoso en Vercel
- ✅ Database con RLS activo

### UX
- ✅ Tiempo de carga < 2s
- ✅ Formularios intuitivos
- ✅ Feedback inmediato en acciones
- ✅ Dark mode funcional

---

## 🚨 Riesgos y Mitigaciones

| Riesgo | Probabilidad | Impacto | Mitigación |
|--------|--------------|---------|------------|
| Complejidad de triggers SQL | Media | Alto | Testing exhaustivo en Supabase local primero |
| Performance de queries | Media | Medio | Índices optimizados + Connection pooling |
| Límites Free Tier Supabase | Baja | Alto | Monitoreo de uso + Plan de upgrade |
| Bugs en actualización de ventas | Media | Alto | Testing de integración completo |
| Retrasos en desarrollo | Media | Medio | Buffer de 2 semanas en timeline |

---

## 📞 Soporte Post-Launch

### Semana 1 Post-Launch
- Monitoreo diario de errors en Sentry
- Revisar logs de Vercel
- Revisar métricas de Supabase
- Recolectar feedback de usuarios

### Semana 2-4 Post-Launch
- Implementar mejoras basadas en feedback
- Optimizaciones de performance
- Bug fixes

### Backlog Post-MVP
- Notificaciones por email
- Exportación a PDF/Excel
- Multi-usuario con roles
- Reportes avanzados

---

## 📚 Recursos de Referencia

### Durante Desarrollo
- [Next.js Docs](https://nextjs.org/docs)
- [Supabase Docs](https://supabase.com/docs)
- [shadcn/ui](https://ui.shadcn.com/)
- [Tailwind CSS](https://tailwindcss.com/docs)

### PRDs del Proyecto
- [INDICE_GENERAL.md](INDICE_GENERAL.md)
- [prds/arquitectura.md](prds/arquitectura.md)
- [prds/frontend-design.md](prds/frontend-design.md)
- [prds/clientes.md](prds/clientes.md)
- [prds/productos.md](prds/productos.md)
- [prds/ventas.md](prds/ventas.md)
- [prds/pagos.md](prds/pagos.md)
- [prds/dashboard.md](prds/dashboard.md)

---

## ✅ Checklist Final Pre-Launch

```markdown
### Código
- [ ] Todo el código en main
- [ ] Sin console.logs
- [ ] Sin TODOs críticos
- [ ] Build exitoso
- [ ] Tests pasando

### Supabase
- [ ] RLS activo en todas las tablas
- [ ] Triggers funcionando
- [ ] Backup configurado
- [ ] Environment variables correctas

### Vercel
- [ ] Deployment exitoso
- [ ] Environment variables configuradas
- [ ] Dominio configurado (si aplica)
- [ ] Analytics habilitado

### Testing
- [ ] Smoke test completo
- [ ] Testing en diferentes navegadores
- [ ] Testing en mobile
- [ ] Lighthouse Score > 90

### Documentación
- [ ] README.md actualizado
- [ ] CHANGELOG.md creado
- [ ] Credenciales de demo creadas
- [ ] User guide básico (opcional)
```

---

**Documento preparado por:** SGV BRASA Team
**Última actualización:** 24/11/2025
**Versión:** 1.0

🚀 **¡Listos para construir!**
