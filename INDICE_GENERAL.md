# 📋 ÍNDICE GENERAL - SGV BRASA
## Sistema de Gestión de Ventas BRASA

**Versión:** 1.0
**Fecha:** 24/11/2025
**Proyecto:** Aplicación Web para Gestión de Ventas y Cobranzas

---

## 🎯 Visión del Proyecto

Transformar el sistema Excel de control de ventas en una **aplicación web moderna, escalable y colaborativa** que permita a pequeñas y medianas empresas gestionar ventas, pagos y cobranzas de forma eficiente y en tiempo real.

---

## 📚 Estructura de Documentación

```
sgv_brasa_/
│
├── 📄 PLAN_IMPLEMENTACION.md           (38 KB) ⭐⭐⭐ CRÍTICO
│   └── Plan Completo de Implementación
│       • 7 fases detalladas paso a paso
│       • De cero a producción en Vercel
│       • Timeline: 6-8 semanas
│       • Comandos exactos para cada paso
│       • Checklists completos
│       • Mitigación de riesgos
│
├── 📄 FRONTEND_SUMMARY.md              (18 KB)
│   └── Resumen Visual del Frontend
│       • Quick reference del design system
│       • Componentes y ejemplos
│       • Checklists de implementación
│
├── 📄 prd_mvp_excelversion.md          (9.4 KB)
│   └── Referencia: Sistema Excel v2.0
│       • 5 hojas interconectadas
│       • Fórmulas BUSCARV automáticas
│       • Control de ventas en cuotas
│
├── 📄 prd_sgv_brasa.md                 (9.7 KB)
│   └── PRD MASTER - Visión General
│       • Resumen ejecutivo
│       • Objetivos de negocio
│       • Roadmap de desarrollo
│       • Stack tecnológico
│       • Priorización MVP vs Futuro
│
└── 📁 prds/
    │
    ├── 📄 README.md                    (6.7 KB)
    │   └── Guía de navegación
    │       • Guía por rol (PM, Dev, QA, UX)
    │       • Checklist de implementación
    │       • Convenciones del proyecto
    │
    ├── 📄 arquitectura.md              (22 KB) ⭐
    │   └── Arquitectura Técnica Completa
    │       • Stack: Next.js 14 + Supabase + Vercel
    │       • Schema SQL completo con triggers
    │       • RLS policies
    │       • Estructura de directorios
    │       • Patrones de arquitectura
    │       • Deployment y CI/CD
    │
    ├── 📄 frontend-design.md           (52 KB) ⭐⭐⭐
    │   └── Frontend & Design System
    │       • Design system completo (shadcn/ui + Radix)
    │       • Paleta de colores y tipografía
    │       • Componentes base documentados
    │       • Layouts y vistas detalladas
    │       • Accesibilidad WCAG 2.1 Level AA
    │       • Dark mode implementation
    │       • Responsive design patterns
    │       • Micro-animaciones y transiciones
    │       • Inspiración: Stripe, Linear, Vercel
    │
    ├── 📄 clientes.md                  (16 KB)
    │   └── Módulo CLIENTES
    │       • CRUD completo
    │       • Modelo de datos
    │       • API endpoints
    │       • Validaciones Zod
    │       • Casos de uso
    │
    ├── 📄 productos.md                 (18 KB)
    │   └── Módulo PRODUCTOS
    │       • Gestión de catálogo
    │       • Precios y categorías
    │       • Relación con ventas
    │       • API y validaciones
    │
    ├── 📄 ventas.md                    (27 KB)
    │   └── Módulo VENTAS (CORE)
    │       • Ventas contado y cuotas
    │       • Cálculos automáticos
    │       • Estados y triggers
    │       • Casos de uso detallados
    │
    ├── 📄 pagos.md                     (28 KB)
    │   └── Módulo PAGOS (CORE)
    │       • Registro de pagos
    │       • Actualización automática de ventas
    │       • Múltiples métodos de pago
    │       • Validaciones y triggers
    │
    └── 📄 dashboard.md                 (19 KB)
        └── Módulo DASHBOARD
            • KPIs y métricas
            • Visualizaciones (Recharts/Tremor)
            • Alertas de cobranza
            • Queries optimizadas

TOTAL: ~264 KB de documentación técnica completa
      • 52 KB de Frontend/Design System
      • 38 KB de Plan de Implementación paso a paso
```

---

## 🗂️ Guía de Lectura Rápida

### 🚀 Para Empezar la Implementación

**1. [PLAN_IMPLEMENTACION.md](PLAN_IMPLEMENTACION.md)** ⭐⭐⭐ **EMPEZAR AQUÍ**
   - Plan completo de 7 fases
   - Comandos exactos paso a paso
   - Timeline: 6-8 semanas
   - De cero a producción en Vercel
   - Checklists completos por fase

### 📖 Para Entender el Proyecto

**2. [INDICE_GENERAL.md](INDICE_GENERAL.md)** (Este documento)
   - Vista general del proyecto
   - Estructura de documentación
   - Guías por rol

**3. [prd_sgv_brasa.md](prd_sgv_brasa.md)**
   - Visión del producto
   - Objetivos de negocio
   - Roadmap general

### 🎨 Para Desarrolladores Frontend

**4. [FRONTEND_SUMMARY.md](FRONTEND_SUMMARY.md)** - Quick Reference
   - Resumen visual del design system
   - Componentes principales
   - Checklists rápidos

**5. [prds/frontend-design.md](prds/frontend-design.md)** - Documentación Completa
   - Design system detallado
   - Todos los componentes con código
   - Layouts y vistas completas
   - Accesibilidad WCAG 2.1

### 🔧 Para Desarrolladores Backend

**6. [prds/arquitectura.md](prds/arquitectura.md)**
   - Stack técnico completo
   - Scripts SQL completos
   - Triggers y RLS policies
   - Estructura de proyecto

### 📚 Para Project Managers

**7. [prds/README.md](prds/README.md)**
   - Guía de navegación de PRDs
   - Checklist de implementación
   - Guía por rol (PM, Dev, QA, UX)

### Orden de Implementación de Módulos

```
Semana 1-2: Setup + CLIENTES + PRODUCTOS
    ├── prds/arquitectura.md
    ├── prds/clientes.md
    └── prds/productos.md

Semana 3-5: VENTAS + PAGOS (Core del sistema)
    ├── prds/ventas.md
    └── prds/pagos.md

Semana 6: DASHBOARD
    └── prds/dashboard.md

Semana 7: Testing + Deploy
```

---

## 🏗️ Arquitectura de Alto Nivel

```
┌─────────────────────────────────────────────────┐
│                   USUARIO                       │
└───────────────────┬─────────────────────────────┘
                    │
                    ▼
┌───────────────────────────────────────────────────┐
│              FRONTEND (Next.js 14)                │
│  ┌──────────┐  ┌──────────┐  ┌──────────┐       │
│  │Dashboard │  │ Ventas   │  │  Pagos   │       │
│  └──────────┘  └──────────┘  └──────────┘       │
│  ┌──────────┐  ┌──────────┐                     │
│  │Clientes  │  │Productos │                     │
│  └──────────┘  └──────────┘                     │
└───────────────────┬───────────────────────────────┘
                    │
         ┌──────────┴──────────┐
         │                     │
         ▼                     ▼
┌─────────────────┐   ┌─────────────────┐
│  Server Actions │   │   API Routes    │
└────────┬────────┘   └────────┬────────┘
         │                     │
         └──────────┬──────────┘
                    │
                    ▼
         ┌──────────────────┐
         │  Drizzle ORM     │
         └────────┬─────────┘
                  │
                  ▼
         ┌──────────────────┐
         │   SUPABASE       │
         │  PostgreSQL      │
         │  + Auth + RLS    │
         └──────────────────┘
                  │
                  ▼
         ┌──────────────────┐
         │     VERCEL       │
         │   (Hosting)      │
         └──────────────────┘
```

---

## 🔑 Componentes Clave

### 1. Base de Datos (PostgreSQL/Supabase)

**Tablas principales:**
- `clientes` - Catálogo de clientes
- `productos` - Catálogo de productos/servicios
- `ventas` - Registro de ventas (contado/cuotas)
- `pagos` - Registro de pagos

**Features:**
- ✅ Columnas computadas (saldo_pendiente, estado)
- ✅ Triggers automáticos (actualización de monto_pagado)
- ✅ Row Level Security (RLS)
- ✅ Validaciones a nivel de BD

### 2. Frontend (Next.js 14)

**Stack:**
- React 18 + TypeScript
- Tailwind CSS + shadcn/ui
- Recharts / Tremor para gráficos
- Zod para validaciones

**Arquitectura:**
- App Router
- Server Components (por defecto)
- Client Components (cuando necesario)
- Server Actions para mutations

### 3. Módulos Funcionales

| Módulo | Descripción | Complejidad |
|--------|-------------|-------------|
| CLIENTES | Gestión de clientes | 🟢 Baja |
| PRODUCTOS | Gestión de productos | 🟢 Baja |
| VENTAS | Core - Ventas en cuotas | 🔴 Alta |
| PAGOS | Core - Actualización automática | 🔴 Alta |
| DASHBOARD | Métricas y visualización | 🟡 Media |

---

## 📊 Modelo de Datos (Relaciones)

```
USERS (Supabase Auth)
  │
  ├─1:N─► CLIENTES
  │         │
  │         └─1:N─► VENTAS ◄─N:1─┐
  │                   │           │
  ├─1:N─► PRODUCTOS──┘           │
  │                               │
  └─1:N─► PAGOS ─────────────N:1─┘

Actualizaciones Automáticas:
PAGOS → (trigger) → VENTAS.monto_pagado
VENTAS.monto_pagado → (computed) → saldo_pendiente
VENTAS.saldo_pendiente → (computed) → estado
```

---

## 🎨 Features Principales

### MVP (Fase 1) - 6 semanas
- ✅ Autenticación (Supabase Auth)
- ✅ CRUD Clientes
- ✅ CRUD Productos
- ✅ CRUD Ventas (contado + cuotas)
- ✅ CRUD Pagos
- ✅ Actualización automática de estados
- ✅ Dashboard básico con KPIs
- ✅ Responsive design

### Post-MVP (Fase 2)
- 📧 Notificaciones por email
- 📄 Exportación PDF/Excel
- 🔍 Búsqueda avanzada
- 👥 Multi-usuario con roles
- 📊 Reportes avanzados

### Futuro (Fase 3)
- 💳 Integración con pasarelas de pago
- 📱 App móvil nativa
- 🤖 Predicciones con IA
- 🔗 API pública

---

## 🚀 Quick Start

### Setup Inicial

```bash
# 1. Clonar y instalar
git clone <repo>
cd sgv-brasa
npm install

# 2. Configurar variables de entorno
cp .env.example .env.local
# Editar .env.local con credenciales de Supabase

# 3. Ejecutar migraciones
# Ver: prds/arquitectura.md - Sección 3.2
# Ejecutar SQL en Supabase Dashboard

# 4. Iniciar desarrollo
npm run dev
```

### Checklist de Implementación

Ver checklist detallado en: [prds/README.md](prds/README.md)

---

## 📖 Referencias Externas

### Documentación Oficial
- [Next.js 14](https://nextjs.org/docs)
- [Supabase](https://supabase.com/docs)
- [Drizzle ORM](https://orm.drizzle.team/)
- [shadcn/ui](https://ui.shadcn.com/)
- [Tailwind CSS](https://tailwindcss.com/)

### Tutoriales Recomendados
- [Next.js + Supabase Auth](https://supabase.com/docs/guides/auth/server-side/nextjs)
- [Server Actions](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)
- [Drizzle + Supabase](https://orm.drizzle.team/docs/get-started-postgresql#supabase)

---

## 📞 Soporte

### Documentos por Pregunta

| Pregunta | Documento |
|----------|-----------|
| "¿Cómo funciona el sistema en general?" | [prd_sgv_brasa.md](prd_sgv_brasa.md) |
| "¿Qué tecnologías usar?" | [prds/arquitectura.md](prds/arquitectura.md) |
| "¿Cómo crear la base de datos?" | [prds/arquitectura.md](prds/arquitectura.md) - Sección 3 |
| "¿Cómo funcionan las ventas en cuotas?" | [prds/ventas.md](prds/ventas.md) |
| "¿Cómo se actualizan los pagos?" | [prds/pagos.md](prds/pagos.md) - Sección 6 |
| "¿Qué métricas mostrar?" | [prds/dashboard.md](prds/dashboard.md) - Sección 2 |
| "¿Por dónde empiezo?" | [prds/README.md](prds/README.md) |

---

## 🎯 Métricas de Éxito

### Técnicas
- ✅ Lighthouse Score > 90
- ✅ Tiempo de carga < 2s
- ✅ Cobertura de tests > 80%
- ✅ Zero errores de TypeScript

### Producto
- 📈 Reducir tiempo de registro de venta en 50%
- 📉 Reducir pagos olvidados en 80%
- ⚡ Actualización de estados en < 500ms
- 😊 NPS > 8/10

---

## 📝 Convenciones

### Nomenclatura de IDs
```
CLI-2024-001   # Clientes
PROD-2024-001  # Productos
V-2024-001     # Ventas
P-2024-001     # Pagos
```

### Estados
```
Ventas:              PAGADO | PENDIENTE
Clientes/Productos:  activo | inactivo
```

### Formato de Moneda
```
Display:  S/ 1,500.00
Storage:  DECIMAL(10,2)
```

---

## 🏆 Resumen de Tamaños de PRDs

```
📊 Estadísticas de Documentación:

PRD Master:           9.7 KB
Excel Reference:      9.4 KB

Módulos:
├── frontend-design: 52 KB  ⭐⭐⭐ NUEVO - Design System Completo
├── pagos:           28 KB  ⭐ Más detallado
├── ventas:          27 KB  ⭐ Más complejo
├── arquitectura:    22 KB  ⭐ SQL completo
├── dashboard:       19 KB
├── productos:       18 KB
├── clientes:        16 KB
└── README:           6.7 KB

TOTAL: ~207 KB de documentación técnica
```

---

## ✅ Estado del Proyecto

```
[✅] PRD Master creado
[✅] PRD de Arquitectura completo (con SQL)
[✅] PRD de Frontend/Design System completo ⭐ NUEVO
[✅] PRD de CLIENTES completo
[✅] PRD de PRODUCTOS completo
[✅] PRD de VENTAS completo
[✅] PRD de PAGOS completo
[✅] PRD de DASHBOARD completo
[✅] README y guías creadas
[✅] Documentación de accesibilidad WCAG 2.1
[✅] Design system basado en shadcn/ui + Radix
[⏳] Implementación pendiente
```

---

## 🎉 ¡Listo para Desarrollar!

**Próximos pasos:**

1. Leer [prds/arquitectura.md](prds/arquitectura.md)
2. Setup del proyecto Next.js
3. Configurar Supabase y ejecutar migraciones SQL
4. Comenzar con módulo CLIENTES
5. Seguir el checklist en [prds/README.md](prds/README.md)

---

**Preparado por:** Sistema de Documentación SGV BRASA
**Última actualización:** 24/11/2025
**Versión:** 1.0

🚀 **¡Manos a la obra!**
