# PRD - Arquitectura Técnica
**Product Requirements Document**

Versión 1.0 | 24/11/2025

---

## 1. Resumen

Este documento define la arquitectura técnica completa del Sistema de Gestión de Ventas BRASA, incluyendo stack tecnológico, estructura de base de datos, patrones de arquitectura, infraestructura y mejores prácticas de desarrollo.

---

## 2. Stack Tecnológico

### 2.1 Frontend

**Framework Principal: Next.js 14+**
- App Router (nueva arquitectura)
- React 18+ con Server Components
- TypeScript para type safety
- Turbopack para builds rápidos

**UI y Styling**
```json
{
  "tailwindcss": "^3.4.0",
  "shadcn/ui": "latest",
  "lucide-react": "^0.300.0",
  "@tremor/react": "^3.14.0"
}
```

**Estado y Data Fetching**
- React Server Components (para fetch inicial)
- Zustand (estado global cliente)
- React Query / SWR (cache y revalidación)
- Optimistic updates

**Validación y Forms**
```json
{
  "zod": "^3.22.0",
  "react-hook-form": "^7.49.0",
  "@hookform/resolvers": "^3.3.0"
}
```

**Visualización de Datos**
```json
{
  "recharts": "^2.10.0",
  "@tremor/react": "^3.14.0"
}
```

### 2.2 Backend

**Platform: Next.js API**
- API Routes para REST endpoints
- Server Actions para mutations
- Edge Functions para operaciones rápidas

**Database ORM**
```json
{
  "drizzle-orm": "^0.29.0",
  "drizzle-kit": "^0.20.0",
  "@supabase/supabase-js": "^2.39.0"
}
```

**Autenticación**
- Supabase Auth
- JWT tokens
- Row Level Security (RLS)

### 2.3 Base de Datos

**Supabase (PostgreSQL)**
- PostgreSQL 15+
- Row Level Security activado
- Realtime subscriptions
- Storage para archivos (Fase 2)

### 2.4 Infraestructura

**Hosting: Vercel**
- Edge Network global
- Automatic deployments
- Preview deployments por PR
- Analytics integrado

**CI/CD**
- GitHub Actions
- Vercel automatic deployments
- Tests automatizados

**Monitoreo**
- Vercel Analytics
- Sentry para error tracking (Fase 2)
- Logs con Vercel Logs

---

## 3. Arquitectura de Base de Datos

### 3.1 Diagrama ERD

```
┌─────────────┐
│   USERS     │
│ (Supabase)  │
└──────┬──────┘
       │
       │ 1:N
       │
┌──────┴────────┐
│   CLIENTES    │◄────────┐
├───────────────┤         │
│ id (UUID)     │         │
│ cliente_id    │         │
│ nombre        │         │
│ email         │         │
│ telefono      │         │
│ user_id       │         │
└───────┬───────┘         │
        │                 │
        │ 1:N             │
        │                 │
┌───────┴───────┐         │
│   PRODUCTOS   │         │
├───────────────┤         │
│ id (UUID)     │         │
│ producto_id   │         │
│ nombre        │         │
│ precio_base   │         │
│ user_id       │         │
└───────┬───────┘         │
        │                 │
        │ 1:N             │
        │                 │
┌───────┴──────────┐      │
│     VENTAS       │      │
├──────────────────┤      │
│ id (UUID)        │      │
│ venta_id         │      │
│ cliente_id ──────┼──────┘
│ producto_id      │
│ monto_total      │
│ monto_pagado     │◄────────┐
│ saldo_pendiente  │ COMPUTED│
│ estado           │ COMPUTED│
│ user_id          │         │
└─────────┬────────┘         │
          │                  │
          │ 1:N              │
          │                  │
┌─────────┴────────┐         │
│     PAGOS        │         │
├──────────────────┤         │
│ id (UUID)        │         │
│ pago_id          │         │
│ venta_id ────────┼─────────┘
│ monto            │ (Actualiza monto_pagado vía trigger)
│ metodo_pago      │
│ fecha_pago       │
│ user_id          │
└──────────────────┘
```

### 3.2 Scripts de Creación de Tablas

**Archivo: `supabase/migrations/001_initial_schema.sql`**

```sql
-- Enable UUID extension
CREATE EXTENSION IF NOT EXISTS "uuid-ossp";

-- ======================================
-- TABLA: clientes
-- ======================================
CREATE TABLE clientes (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  cliente_id VARCHAR(20) UNIQUE NOT NULL,
  nombre VARCHAR(255) NOT NULL,
  email VARCHAR(255) UNIQUE,
  telefono VARCHAR(20),
  direccion TEXT,
  documento VARCHAR(20),
  tipo VARCHAR(20) DEFAULT 'persona' CHECK (tipo IN ('persona', 'empresa')),
  notas TEXT,
  estado VARCHAR(20) DEFAULT 'activo' CHECK (estado IN ('activo', 'inactivo')),
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE
);

CREATE INDEX idx_clientes_cliente_id ON clientes(cliente_id);
CREATE INDEX idx_clientes_nombre ON clientes(nombre);
CREATE INDEX idx_clientes_user_id ON clientes(user_id);
CREATE INDEX idx_clientes_estado ON clientes(estado);

-- ======================================
-- TABLA: productos
-- ======================================
CREATE TABLE productos (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  producto_id VARCHAR(20) UNIQUE NOT NULL,
  nombre VARCHAR(255) NOT NULL,
  descripcion TEXT,
  categoria VARCHAR(100),
  precio_base DECIMAL(10,2) NOT NULL CHECK (precio_base > 0),
  unidad VARCHAR(20) DEFAULT 'unidad',
  sku VARCHAR(50) UNIQUE,
  estado VARCHAR(20) DEFAULT 'activo' CHECK (estado IN ('activo', 'inactivo')),
  notas TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE
);

CREATE INDEX idx_productos_producto_id ON productos(producto_id);
CREATE INDEX idx_productos_nombre ON productos(nombre);
CREATE INDEX idx_productos_categoria ON productos(categoria);
CREATE INDEX idx_productos_user_id ON productos(user_id);
CREATE INDEX idx_productos_estado ON productos(estado);

-- ======================================
-- TABLA: ventas
-- ======================================
CREATE TABLE ventas (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  venta_id VARCHAR(20) UNIQUE NOT NULL,
  fecha DATE NOT NULL,
  cliente_id UUID NOT NULL REFERENCES clientes(id) ON DELETE RESTRICT,
  producto_id UUID NOT NULL REFERENCES productos(id) ON DELETE RESTRICT,
  producto_nombre VARCHAR(255) NOT NULL,
  servicio_adicional TEXT,
  monto_total DECIMAL(10,2) NOT NULL CHECK (monto_total > 0),
  tipo_pago VARCHAR(20) NOT NULL CHECK (tipo_pago IN ('contado', 'cuotas')),
  num_cuotas INTEGER DEFAULT 1 CHECK (num_cuotas >= 1),
  monto_cuota DECIMAL(10,2),
  monto_pagado DECIMAL(10,2) DEFAULT 0 CHECK (monto_pagado >= 0),
  saldo_pendiente DECIMAL(10,2) GENERATED ALWAYS AS (monto_total - monto_pagado) STORED,
  estado VARCHAR(20) GENERATED ALWAYS AS (
    CASE WHEN (monto_total - monto_pagado) <= 0 THEN 'PAGADO' ELSE 'PENDIENTE' END
  ) STORED,
  fecha_primer_pago DATE,
  observacion TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE
);

CREATE INDEX idx_ventas_venta_id ON ventas(venta_id);
CREATE INDEX idx_ventas_cliente_id ON ventas(cliente_id);
CREATE INDEX idx_ventas_producto_id ON ventas(producto_id);
CREATE INDEX idx_ventas_fecha ON ventas(fecha);
CREATE INDEX idx_ventas_estado ON ventas(estado);
CREATE INDEX idx_ventas_user_id ON ventas(user_id);

-- ======================================
-- TABLA: pagos
-- ======================================
CREATE TYPE metodo_pago_enum AS ENUM (
  'efectivo',
  'transferencia',
  'yape',
  'plin',
  'tarjeta_credito',
  'tarjeta_debito',
  'otro'
);

CREATE TABLE pagos (
  id UUID PRIMARY KEY DEFAULT uuid_generate_v4(),
  pago_id VARCHAR(20) UNIQUE NOT NULL,
  venta_id UUID NOT NULL REFERENCES ventas(id) ON DELETE CASCADE,
  fecha_pago DATE NOT NULL,
  num_cuota INTEGER NOT NULL CHECK (num_cuota >= 0),
  monto DECIMAL(10,2) NOT NULL CHECK (monto > 0),
  metodo_pago metodo_pago_enum NOT NULL,
  comprobante VARCHAR(100),
  observacion TEXT,
  created_at TIMESTAMPTZ DEFAULT NOW(),
  updated_at TIMESTAMPTZ DEFAULT NOW(),
  user_id UUID NOT NULL REFERENCES auth.users(id) ON DELETE CASCADE
);

CREATE INDEX idx_pagos_pago_id ON pagos(pago_id);
CREATE INDEX idx_pagos_venta_id ON pagos(venta_id);
CREATE INDEX idx_pagos_fecha_pago ON pagos(fecha_pago);
CREATE INDEX idx_pagos_user_id ON pagos(user_id);

-- ======================================
-- TRIGGERS
-- ======================================

-- Trigger: Actualizar monto_pagado en ventas al insertar/actualizar/eliminar pago
CREATE OR REPLACE FUNCTION update_venta_monto_pagado()
RETURNS TRIGGER AS $$
DECLARE
  v_venta_id UUID;
BEGIN
  IF TG_OP = 'DELETE' THEN
    v_venta_id := OLD.venta_id;
  ELSE
    v_venta_id := NEW.venta_id;
  END IF;

  UPDATE ventas
  SET
    monto_pagado = (
      SELECT COALESCE(SUM(monto), 0)
      FROM pagos
      WHERE venta_id = v_venta_id
    ),
    updated_at = NOW()
  WHERE id = v_venta_id;

  RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_monto_pagado
AFTER INSERT OR UPDATE OR DELETE ON pagos
FOR EACH ROW
EXECUTE FUNCTION update_venta_monto_pagado();

-- Trigger: Validar que el pago no exceda el saldo pendiente
CREATE OR REPLACE FUNCTION check_pago_no_excede_venta()
RETURNS TRIGGER AS $$
DECLARE
  v_monto_total DECIMAL(10,2);
  v_monto_pagado DECIMAL(10,2);
BEGIN
  SELECT monto_total INTO v_monto_total
  FROM ventas
  WHERE id = NEW.venta_id;

  SELECT COALESCE(SUM(monto), 0) INTO v_monto_pagado
  FROM pagos
  WHERE venta_id = NEW.venta_id
    AND id != COALESCE(NEW.id, '00000000-0000-0000-0000-000000000000');

  IF (v_monto_pagado + NEW.monto) > v_monto_total THEN
    RAISE EXCEPTION 'El pago excede el saldo pendiente de la venta';
  END IF;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_check_pago_no_excede
BEFORE INSERT OR UPDATE ON pagos
FOR EACH ROW
EXECUTE FUNCTION check_pago_no_excede_venta();

-- Trigger: Actualizar updated_at automáticamente
CREATE OR REPLACE FUNCTION update_updated_at()
RETURNS TRIGGER AS $$
BEGIN
  NEW.updated_at = NOW();
  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER update_clientes_updated_at
BEFORE UPDATE ON clientes
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_productos_updated_at
BEFORE UPDATE ON productos
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_ventas_updated_at
BEFORE UPDATE ON ventas
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();

CREATE TRIGGER update_pagos_updated_at
BEFORE UPDATE ON pagos
FOR EACH ROW
EXECUTE FUNCTION update_updated_at();
```

### 3.3 Row Level Security (RLS)

**Archivo: `supabase/migrations/002_rls_policies.sql`**

```sql
-- Enable RLS on all tables
ALTER TABLE clientes ENABLE ROW LEVEL SECURITY;
ALTER TABLE productos ENABLE ROW LEVEL SECURITY;
ALTER TABLE ventas ENABLE ROW LEVEL SECURITY;
ALTER TABLE pagos ENABLE ROW LEVEL SECURITY;

-- ======================================
-- POLÍTICAS: clientes
-- ======================================
CREATE POLICY "Users can view own clientes"
  ON clientes FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own clientes"
  ON clientes FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own clientes"
  ON clientes FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own clientes"
  ON clientes FOR DELETE
  USING (auth.uid() = user_id);

-- ======================================
-- POLÍTICAS: productos
-- ======================================
CREATE POLICY "Users can view own productos"
  ON productos FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own productos"
  ON productos FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own productos"
  ON productos FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own productos"
  ON productos FOR DELETE
  USING (auth.uid() = user_id);

-- ======================================
-- POLÍTICAS: ventas
-- ======================================
CREATE POLICY "Users can view own ventas"
  ON ventas FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own ventas"
  ON ventas FOR INSERT
  WITH CHECK (auth.uid() = user_id);

CREATE POLICY "Users can update own ventas"
  ON ventas FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own ventas without payments"
  ON ventas FOR DELETE
  USING (
    auth.uid() = user_id
    AND NOT EXISTS (
      SELECT 1 FROM pagos WHERE venta_id = ventas.id
    )
  );

-- ======================================
-- POLÍTICAS: pagos
-- ======================================
CREATE POLICY "Users can view own pagos"
  ON pagos FOR SELECT
  USING (auth.uid() = user_id);

CREATE POLICY "Users can insert own pagos"
  ON pagos FOR INSERT
  WITH CHECK (
    auth.uid() = user_id
    AND EXISTS (
      SELECT 1 FROM ventas
      WHERE id = venta_id AND user_id = auth.uid()
    )
  );

CREATE POLICY "Users can update own pagos"
  ON pagos FOR UPDATE
  USING (auth.uid() = user_id);

CREATE POLICY "Users can delete own pagos"
  ON pagos FOR DELETE
  USING (auth.uid() = user_id);
```

---

## 4. Estructura del Proyecto

### 4.1 Organización de Directorios

```
sgv-brasa/
├── .github/
│   └── workflows/
│       └── ci.yml
├── app/
│   ├── (auth)/
│   │   ├── login/
│   │   │   └── page.tsx
│   │   └── register/
│   │       └── page.tsx
│   ├── (dashboard)/
│   │   ├── layout.tsx
│   │   ├── page.tsx              # Dashboard
│   │   ├── clientes/
│   │   │   ├── page.tsx          # Lista
│   │   │   ├── [id]/
│   │   │   │   └── page.tsx      # Detalle
│   │   │   └── nuevo/
│   │   │       └── page.tsx      # Crear
│   │   ├── productos/
│   │   │   └── ...
│   │   ├── ventas/
│   │   │   └── ...
│   │   └── pagos/
│   │       └── ...
│   ├── api/
│   │   ├── clientes/
│   │   │   ├── route.ts
│   │   │   └── [id]/
│   │   │       └── route.ts
│   │   ├── productos/
│   │   ├── ventas/
│   │   ├── pagos/
│   │   └── dashboard/
│   ├── actions/
│   │   ├── clientes.ts
│   │   ├── productos.ts
│   │   ├── ventas.ts
│   │   ├── pagos.ts
│   │   └── dashboard.ts
│   ├── layout.tsx
│   └── globals.css
├── components/
│   ├── ui/                       # shadcn components
│   ├── clientes/
│   │   ├── cliente-form.tsx
│   │   ├── cliente-list.tsx
│   │   └── cliente-card.tsx
│   ├── productos/
│   ├── ventas/
│   ├── pagos/
│   └── dashboard/
│       ├── kpi-card.tsx
│       ├── ventas-chart.tsx
│       └── alertas-list.tsx
├── lib/
│   ├── db/
│   │   ├── index.ts              # Drizzle client
│   │   └── schema.ts             # Schema definitions
│   ├── supabase/
│   │   ├── client.ts             # Cliente Supabase
│   │   └── server.ts             # Server client
│   ├── utils.ts
│   └── validations/
│       ├── cliente.ts            # Zod schemas
│       ├── producto.ts
│       ├── venta.ts
│       └── pago.ts
├── types/
│   ├── cliente.ts
│   ├── producto.ts
│   ├── venta.ts
│   └── pago.ts
├── hooks/
│   ├── use-clientes.ts
│   ├── use-productos.ts
│   ├── use-ventas.ts
│   └── use-pagos.ts
├── supabase/
│   └── migrations/
│       ├── 001_initial_schema.sql
│       └── 002_rls_policies.sql
├── public/
│   └── images/
├── .env.local
├── .env.example
├── next.config.js
├── tailwind.config.ts
├── tsconfig.json
├── package.json
└── README.md
```

---

## 5. Patrones de Arquitectura

### 5.1 Server Components y Client Components

**Server Components (Default):**
- Fetch de datos en el servidor
- Acceso directo a base de datos
- No aumentan bundle size
- SEO-friendly

**Client Components (cuando necesario):**
- Interactividad (useState, useEffect)
- Event handlers
- Browser APIs
- Context providers

**Ejemplo:**
```tsx
// app/(dashboard)/clientes/page.tsx - SERVER COMPONENT
import { getClientes } from '@/app/actions/clientes';
import ClienteList from '@/components/clientes/cliente-list';

export default async function ClientesPage() {
  const clientes = await getClientes(); // Fetch en servidor

  return <ClienteList clientes={clientes} />; // Pasa data a cliente
}

// components/clientes/cliente-list.tsx - CLIENT COMPONENT
'use client';

import { useState } from 'react';

export default function ClienteList({ clientes }) {
  const [search, setSearch] = useState('');
  // ... lógica de búsqueda y filtrado
  return <div>{/* UI */}</div>;
}
```

### 5.2 Server Actions

**Definición:**
```typescript
// app/actions/ventas.ts
'use server';

import { db } from '@/lib/db';
import { ventas } from '@/lib/db/schema';
import { revalidatePath } from 'next/cache';

export async function createVenta(data: CreateVentaInput) {
  try {
    const venta = await db.insert(ventas).values(data).returning();
    revalidatePath('/ventas');
    return { success: true, data: venta[0] };
  } catch (error) {
    return { success: false, error: error.message };
  }
}
```

**Uso en Cliente:**
```tsx
'use client';

import { createVenta } from '@/app/actions/ventas';

export function VentaForm() {
  async function handleSubmit(formData: FormData) {
    const result = await createVenta({
      // ... data from form
    });

    if (result.success) {
      toast.success('Venta creada');
    }
  }

  return <form action={handleSubmit}>{/* ... */}</form>;
}
```

### 5.3 Optimistic Updates

```tsx
'use client';

import { useOptimistic } from 'react';
import { deletePago } from '@/app/actions/pagos';

export function PagosList({ pagos }) {
  const [optimisticPagos, removeOptimisticPago] = useOptimistic(
    pagos,
    (state, pagoId) => state.filter(p => p.id !== pagoId)
  );

  async function handleDelete(pagoId: string) {
    removeOptimisticPago(pagoId); // UI actualiza inmediatamente
    await deletePago(pagoId);      // Actualiza en servidor
  }

  return (
    <div>
      {optimisticPagos.map(pago => (
        <PagoCard key={pago.id} pago={pago} onDelete={handleDelete} />
      ))}
    </div>
  );
}
```

---

## 6. Configuración de Ambiente

### 6.1 Variables de Entorno

**Archivo: `.env.local`**
```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=eyJxxx...
SUPABASE_SERVICE_ROLE_KEY=eyJxxx...

# Database
DATABASE_URL=postgresql://xxx

# App
NEXT_PUBLIC_APP_URL=http://localhost:3000
```

**Archivo: `.env.example`**
```bash
# Supabase
NEXT_PUBLIC_SUPABASE_URL=
NEXT_PUBLIC_SUPABASE_ANON_KEY=
SUPABASE_SERVICE_ROLE_KEY=

# Database
DATABASE_URL=

# App
NEXT_PUBLIC_APP_URL=
```

---

## 7. Deployment

### 7.1 Vercel Configuration

**Archivo: `vercel.json`**
```json
{
  "buildCommand": "npm run build",
  "devCommand": "npm run dev",
  "installCommand": "npm install",
  "framework": "nextjs",
  "regions": ["iad1"],
  "env": {
    "NEXT_PUBLIC_SUPABASE_URL": "@supabase-url",
    "NEXT_PUBLIC_SUPABASE_ANON_KEY": "@supabase-anon-key"
  }
}
```

### 7.2 GitHub Actions

**Archivo: `.github/workflows/ci.yml`**
```yaml
name: CI

on:
  push:
    branches: [main, develop]
  pull_request:
    branches: [main]

jobs:
  test:
    runs-on: ubuntu-latest
    steps:
      - uses: actions/checkout@v3
      - uses: actions/setup-node@v3
        with:
          node-version: '20'
      - run: npm ci
      - run: npm run lint
      - run: npm run type-check
      - run: npm run test
```

---

## 8. Seguridad

### 8.1 Mejores Prácticas

- ✅ Row Level Security habilitado en todas las tablas
- ✅ Variables de entorno para secretos
- ✅ HTTPS obligatorio en producción
- ✅ Validación de inputs con Zod
- ✅ Sanitización de datos
- ✅ Rate limiting (Vercel built-in)
- ✅ CORS configurado apropiadamente

### 8.2 Autenticación

```typescript
// lib/supabase/server.ts
import { createServerClient } from '@supabase/ssr';
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

---

## 9. Testing

### 9.1 Estrategia de Testing

**Unit Tests:**
- Funciones de utilidad
- Validaciones Zod
- Cálculos de negocio

**Integration Tests:**
- Server Actions
- API Routes
- Database operations

**E2E Tests:**
- Flujos críticos de usuario
- Playwright o Cypress

### 9.2 Configuración

**Archivo: `jest.config.js`**
```javascript
const nextJest = require('next/jest');

const createJestConfig = nextJest({
  dir: './',
});

const customJestConfig = {
  testEnvironment: 'jest-environment-jsdom',
  setupFilesAfterEnv: ['<rootDir>/jest.setup.js'],
  moduleNameMapper: {
    '^@/(.*)$': '<rootDir>/$1',
  },
};

module.exports = createJestConfig(customJestConfig);
```

---

## 10. Performance

### 10.1 Optimizaciones

- ✅ Server Components por defecto
- ✅ Image optimization con next/image
- ✅ Font optimization con next/font
- ✅ Code splitting automático
- ✅ Lazy loading de componentes pesados
- ✅ Cache de queries con React Query
- ✅ Índices en base de datos

### 10.2 Métricas

- Lighthouse Score: > 90
- First Contentful Paint: < 1.5s
- Time to Interactive: < 3s
- Total Blocking Time: < 200ms

---

**Documento preparado por:** SGV BRASA Team
**Última actualización:** 24/11/2025
