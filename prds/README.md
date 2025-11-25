# PRDs - Sistema de Gestión de Ventas BRASA

Documentación técnica y de producto del sistema SGV BRASA.

---

## 📚 Estructura de Documentación

### Documento Principal
- **[prd_sgv_brasa.md](../prd_sgv_brasa.md)** - PRD Master que contiene la visión general del producto, objetivos, roadmap y referencias a todos los módulos

### PRDs por Módulo

1. **[arquitectura.md](./arquitectura.md)** - Arquitectura Técnica
   - Stack tecnológico completo
   - Estructura de base de datos con SQL
   - Patrones de arquitectura
   - Configuración de infraestructura
   - **EMPEZAR AQUÍ** si eres desarrollador backend

2. **[frontend-design.md](./frontend-design.md)** - Frontend & Design System ⭐⭐⭐
   - Design system completo (shadcn/ui + Radix UI)
   - Paleta de colores, tipografía, espaciado
   - Componentes base documentados con código
   - Layouts y vistas detalladas de todas las páginas
   - Accesibilidad WCAG 2.1 Level AA
   - Dark mode implementation
   - Responsive design patterns
   - Micro-animaciones y transiciones
   - Inspiración: Stripe, Linear, Vercel
   - **EMPEZAR AQUÍ** si eres desarrollador frontend o diseñador

3. **[clientes.md](./clientes.md)** - Módulo CLIENTES
   - Gestión de catálogo de clientes
   - CRUD completo
   - Búsqueda y autocomplete
   - Validaciones y API

3. **[productos.md](./productos.md)** - Módulo PRODUCTOS
   - Gestión de catálogo de productos/servicios
   - Precios y categorías
   - Relación con ventas
   - Validaciones y API

5. **[ventas.md](./ventas.md)** - Módulo VENTAS
   - Registro de ventas (contado y cuotas)
   - Cálculos automáticos de estado
   - Actualización en tiempo real
   - Casos de uso detallados

6. **[pagos.md](./pagos.md)** - Módulo PAGOS
   - Registro de pagos
   - Actualización automática de ventas
   - Múltiples métodos de pago
   - Triggers y validaciones

7. **[dashboard.md](./dashboard.md)** - Módulo DASHBOARD
   - KPIs y métricas
   - Visualizaciones y gráficos
   - Alertas de cobranza
   - Queries optimizadas

---

## 🎯 Guía de Lectura por Rol

### Para Product Managers
1. Lee el [PRD Master](../prd_sgv_brasa.md) completo
2. Revisa los casos de uso en cada módulo
3. Enfócate en la sección de "Features Futuras" de cada PRD

### Para Desarrolladores Backend
1. Comienza con [arquitectura.md](./arquitectura.md) - especialmente la sección de Base de Datos
2. Revisa el modelo de datos de cada módulo
3. Estudia los triggers y validaciones en [ventas.md](./ventas.md) y [pagos.md](./pagos.md)
4. Lee las secciones de API Endpoints y Server Actions

### Para Desarrolladores Frontend
1. **EMPEZAR AQUÍ:** Lee [frontend-design.md](./frontend-design.md) completo
2. Implementa el design system con shadcn/ui + Radix UI
3. Estudia los layouts y vistas detalladas con código
4. Implementa los componentes siguiendo los ejemplos
5. Revisa las secciones de "Interfaz de Usuario" en cada módulo
6. Estudia las validaciones con Zod en cada PRD
7. Lee [arquitectura.md](./arquitectura.md) - sección de Stack Frontend

### Para Diseñadores UI/UX
1. **EMPEZAR AQUÍ:** Lee [frontend-design.md](./frontend-design.md) completo
2. Estudia la paleta de colores, tipografía y espaciado
3. Revisa los componentes base y sus variantes
4. Analiza los layouts y vistas detalladas
5. Estudia los principios de accesibilidad WCAG 2.1
6. Revisa las animaciones y transiciones
7. Consulta las referencias de Stripe, Linear y Vercel
8. Revisa las secciones de "Interfaz de Usuario" en cada módulo
9. Presta atención a los layouts ASCII en [ventas.md](./ventas.md) y [pagos.md](./pagos.md)
10. Revisa los "Casos de Uso" para entender los flujos de usuario

### Para QA/Testers
1. Lee los "Casos de Uso" de cada módulo
2. Revisa las secciones de "Validaciones y Errores"
3. Estudia la sección de "Testing" en cada PRD
4. Enfócate en los flujos alternativos y casos edge

---

## 📋 Checklist de Implementación

### Fase 1: Setup (Semana 1)
- [ ] Configurar proyecto Next.js 14
- [ ] Setup Supabase y base de datos
- [ ] Ejecutar migraciones SQL ([arquitectura.md](./arquitectura.md))
- [ ] Configurar autenticación
- [ ] Setup Tailwind y shadcn/ui
- [ ] Configurar Vercel

### Fase 2: Módulos Base (Semana 2-3)
- [ ] Implementar módulo CLIENTES ([clientes.md](./clientes.md))
- [ ] Implementar módulo PRODUCTOS ([productos.md](./productos.md))
- [ ] Testing de CRUD básico
- [ ] Validaciones con Zod

### Fase 3: Módulos Core (Semana 4-5)
- [ ] Implementar módulo VENTAS ([ventas.md](./ventas.md))
- [ ] Implementar módulo PAGOS ([pagos.md](./pagos.md))
- [ ] Configurar triggers de actualización automática
- [ ] Testing de integración
- [ ] Validar cálculos automáticos

### Fase 4: Dashboard (Semana 6)
- [ ] Implementar módulo DASHBOARD ([dashboard.md](./dashboard.md))
- [ ] Integrar Recharts o Tremor
- [ ] Implementar KPIs
- [ ] Implementar gráficos
- [ ] Alertas de cobranza

### Fase 5: Testing y Deploy (Semana 7)
- [ ] Tests E2E con Playwright
- [ ] Testing de performance
- [ ] Optimizaciones
- [ ] Deploy a producción
- [ ] Documentación de usuario

---

## 🔧 Tecnologías Principales

### Frontend
- Next.js 14 (App Router)
- React 18
- TypeScript
- Tailwind CSS
- shadcn/ui
- Recharts / Tremor

### Backend
- Next.js API Routes
- Server Actions
- Drizzle ORM
- Supabase (PostgreSQL)
- Supabase Auth

### Infraestructura
- Vercel (Hosting)
- Supabase Cloud (Database)
- GitHub (Version Control)

---

## 📊 Diagrama de Flujo de Datos

```
┌─────────────┐
│   Usuario   │
└──────┬──────┘
       │
       ▼
┌─────────────────┐
│   Next.js App   │
│  (App Router)   │
└────┬────────┬───┘
     │        │
     ▼        ▼
┌─────────┐ ┌─────────────┐
│ Server  │ │   Server    │
│Components│ │   Actions   │
└────┬────┘ └──────┬──────┘
     │             │
     └──────┬──────┘
            ▼
    ┌───────────────┐
    │  Drizzle ORM  │
    └───────┬───────┘
            │
            ▼
    ┌───────────────┐
    │   Supabase    │
    │  PostgreSQL   │
    └───────────────┘
```

---

## 🔗 Enlaces Útiles

### Documentación Oficial
- [Next.js 14 Docs](https://nextjs.org/docs)
- [Supabase Docs](https://supabase.com/docs)
- [Drizzle ORM](https://orm.drizzle.team/)
- [shadcn/ui](https://ui.shadcn.com/)
- [Tailwind CSS](https://tailwindcss.com/)

### Tutoriales Relevantes
- [Next.js + Supabase Auth](https://supabase.com/docs/guides/auth/server-side/nextjs)
- [Server Actions in Next.js](https://nextjs.org/docs/app/building-your-application/data-fetching/server-actions-and-mutations)
- [Drizzle with Supabase](https://orm.drizzle.team/docs/get-started-postgresql#supabase)

---

## 📝 Convenciones

### Nomenclatura de IDs
- Clientes: `CLI-2024-001`
- Productos: `PROD-2024-001`
- Ventas: `V-2024-001`
- Pagos: `P-2024-001`

### Estados
- Ventas: `PAGADO` | `PENDIENTE`
- Clientes/Productos: `activo` | `inactivo`

### Moneda
- Formato: `S/ XX,XXX.XX` (Soles peruanos)
- Almacenamiento: `DECIMAL(10,2)`

---

## 🤝 Contribución

Si encuentras inconsistencias o tienes sugerencias:
1. Documenta el cambio necesario
2. Actualiza el PRD correspondiente
3. Actualiza este README si es necesario
4. Mantén la trazabilidad de cambios en "Control de Versiones"

---

## 📞 Contacto

Para preguntas sobre estos PRDs, contacta al equipo de producto.

---

**Última actualización:** 24/11/2025
**Versión de PRDs:** 1.0
