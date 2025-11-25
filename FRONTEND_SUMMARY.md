# 🎨 Frontend & Design System - Resumen Visual
## Sistema de Gestión de Ventas BRASA

**Documento:** Resumen ejecutivo del PRD de Frontend
**Fecha:** 24/11/2025
**PRD Completo:** [prds/frontend-design.md](prds/frontend-design.md)

---

## ✨ Características Destacadas

### Design System de Clase Mundial

Inspirado en las mejores prácticas de:
- ✅ **Stripe** - Flujos de pago y dashboards limpios
- ✅ **Linear** - Velocidad y atajos de teclado
- ✅ **Vercel** - Design system Geist, tipografía moderna

### Stack Tecnológico Frontend

```typescript
{
  // Component Library
  "shadcn/ui": "latest",           // Copy-paste components
  "@radix-ui/react-*": "^1.0.0",   // Accessible primitives

  // Styling
  "tailwindcss": "^3.4.0",
  "class-variance-authority": "^0.7.0",

  // Icons & Charts
  "lucide-react": "^0.300.0",
  "recharts": "^2.10.0",
  "@tremor/react": "^3.14.0",

  // Fonts
  "geist/font": "latest",          // Vercel's font

  // Animations
  "framer-motion": "^10.0.0",

  // Notifications
  "sonner": "^1.0.0"
}
```

---

## 🎨 Design System

### Paleta de Colores

```css
/* Brand Colors */
Primary (Brand):    #0ea5e9  /* Sky blue */
Success:            #22c55e  /* Green */
Warning:            #eab308  /* Yellow */
Error:              #ef4444  /* Red */

/* Neutral (Gray Scale) */
Background (Light): #fafafa
Background (Dark):  #0a0a0a
Foreground (Light): #171717
Foreground (Dark):  #f5f5f5

/* Semantic Usage */
PAGADO:      success-500  /* Verde */
PENDIENTE:   warning-500  /* Amarillo */
ACTIVO:      success-500  /* Verde */
INACTIVO:    neutral-400  /* Gris */
```

**Contraste:** WCAG 2.1 Level AA (mínimo 4.5:1)
**Objetivo:** AAA donde sea posible (7:1)

### Tipografía

```typescript
// Font Family
Font Sans: Geist Sans (Vercel) o Inter
Font Mono: Geist Mono

// Escala Tipográfica
H1: text-3xl (30px) font-bold      // Títulos de página
H2: text-2xl (24px) font-semibold  // Secciones principales
H3: text-xl  (20px) font-semibold  // Subsecciones
H4: text-lg  (18px) font-medium    // Títulos de tarjetas
Body: text-base (16px) font-normal // Texto principal
Small: text-sm (14px) font-normal  // Texto secundario
Caption: text-xs (12px) font-normal // Metadatos
```

### Espaciado (8px base)

```css
Padding de tarjetas:      24px (p-6)
Entre secciones:          48px (space-y-12)
Entre tarjetas:           24px (space-y-6)
Entre elementos de form:  16px (space-y-4)
Padding de botones:       16px horizontal, 8px vertical
```

---

## 🧩 Componentes Base

### Botones

```tsx
// Variants
<Button variant="default">Primary Action</Button>
<Button variant="secondary">Secondary</Button>
<Button variant="outline">Outline</Button>
<Button variant="ghost">Ghost</Button>
<Button variant="destructive">Delete</Button>

// Sizes
<Button size="sm">Small</Button>      // 36px height
<Button size="default">Default</Button> // 40px height
<Button size="lg">Large</Button>      // 44px height

// With Icons
<Button>
  <Plus className="mr-2 h-4 w-4" />
  Nueva Venta
</Button>
```

**Especificaciones:**
- Altura mínima: 40px (WCAG target size)
- Border radius: 6px
- Estados: hover, active, disabled, loading
- Focus ring visible

### Cards

```tsx
<Card className="p-6">
  <CardHeader>
    <CardTitle>Ventas del Mes</CardTitle>
  </CardHeader>
  <CardContent>
    {content}
  </CardContent>
</Card>
```

**Especificaciones:**
- Border radius: 12px
- Shadow: subtle
- Padding: 24px default
- Hover: shadow-lg transition

### Badges (Estados)

```tsx
<Badge variant="success">PAGADO</Badge>      // Verde
<Badge variant="warning">PENDIENTE</Badge>   // Amarillo
<Badge variant="default">ACTIVO</Badge>      // Azul
<Badge variant="destructive">ERROR</Badge>   // Rojo
```

### Forms

```tsx
<div className="space-y-2">
  <Label htmlFor="email">Email *</Label>
  <Input
    id="email"
    type="email"
    placeholder="tu@email.com"
    aria-required="true"
    aria-describedby="email-error"
  />
  <FormMessage id="email-error">{error}</FormMessage>
</div>
```

**Validaciones:**
- Validación con Zod
- react-hook-form
- Error states visuales
- Mensajes de error descriptivos

---

## 📱 Layouts Principales

### 1. Layout General (Desktop)

```
┌─────────────────────────────────────────────┐
│ ┌────┐ ┌──────────────────────────────┐   │
│ │    │ │ Header / Breadcrumb          │   │
│ │  S │ └──────────────────────────────┘   │
│ │  I │ ┌──────────────────────────────┐   │
│ │  D │ │                              │   │
│ │  E │ │  MAIN CONTENT                │   │
│ │  B │ │                              │   │
│ │  A │ │                              │   │
│ │  R │ │                              │   │
│ │    │ └──────────────────────────────┘   │
│ └────┘                                     │
└─────────────────────────────────────────────┘

Sidebar: 256px (w-64)
Content: flex-1
```

### 2. Layout Mobile

```
┌─────────────────┐
│  Header + Menu  │  64px height
├─────────────────┤
│                 │
│  Main Content   │  Full width, scrollable
│  (Full Width)   │
│                 │
├─────────────────┤
│  Bottom Nav     │  Fixed bottom
└─────────────────┘

Bottom Nav: Dashboard, Ventas, Pagos, Clientes
```

---

## 🖥️ Vistas Principales

### Dashboard

**Componentes:**
```
1. KPI Cards Grid (4 columnas)
   ├── Ventas Totales
   ├── Monto Vendido
   ├── Saldo Pendiente
   └── Ventas Pagadas

2. Alertas Section
   └── Ventas Pendientes (lista con prioridad)

3. Charts Grid (2 columnas)
   ├── Ventas por Período (line/bar chart)
   └── Estado de Ventas (donut chart)

4. Recent Activity (2 columnas)
   ├── Últimas Ventas (table)
   └── Últimos Pagos (table)
```

**KPI Card Visual:**
```
┌──────────────────────────────┐
│ Ventas Totales       [Icon] │
│                              │
│ 45                    ↑12%  │
│ vs período anterior          │
└──────────────────────────────┘
```

### Lista de Ventas

**Features:**
- Search bar con debounce
- Filtros: Estado, Tipo Pago, Rango de fechas
- Stats summary (3 cards)
- Tabla/Cards responsive
- Paginación
- Actions dropdown por row

**Desktop Table:**
```
ID    | Fecha | Cliente | Producto | Total | Pagado | Saldo | Estado | •••
V-001 | 24/11 | Juan P. | Parrilla | S/600 | S/200  | S/400 | 🟡     | ⋮
```

**Mobile Cards:**
```
┌────────────────────────────┐
│ V-001          🟡 PENDIENTE│
│ Juan Pérez García          │
│ Parrilla Familiar          │
│                            │
│ Total: S/600  [Ver detalle]│
└────────────────────────────┘
```

### Formulario Nueva Venta

**Layout:**
```
┌─────────────────────────────────┐
│ Nueva Venta                     │
├─────────────────────────────────┤
│ Información General             │
│ • Fecha                         │
│ • Cliente (autocomplete)        │
│ • Producto (autocomplete)       │
│ • Servicios adicionales         │
│                                 │
│ ─────────────────────────       │
│                                 │
│ Información de Pago             │
│ • Monto Total                   │
│ • Tipo de Pago (radio)          │
│ • [Si cuotas]                   │
│   - Número de cuotas            │
│   - Monto por cuota (calc)      │
│   - Fecha primer pago           │
│ • Observaciones                 │
│                                 │
│ [Cancelar]  [Crear Venta] ✓    │
└─────────────────────────────────┘
```

**Validación en tiempo real:**
- Campos obligatorios marcados con *
- Error messages inline
- Success states visuales
- Loading states en submit

---

## ♿ Accesibilidad (WCAG 2.1 Level AA)

### Principios Implementados

**1. Perceptible**
- ✅ Contraste mínimo 4.5:1 (texto normal)
- ✅ Contraste mínimo 3:1 (texto grande, UI)
- ✅ Información no solo por color
- ✅ Texto redimensionable hasta 200%

**2. Operable**
- ✅ Navegación completa por teclado
- ✅ No trampas de teclado
- ✅ Skip links
- ✅ Focus visible en todos los elementos

**3. Comprensible**
- ✅ Idioma de página: español
- ✅ Navegación consistente
- ✅ Identificación clara de errores
- ✅ Labels asociados con inputs

**4. Robusto**
- ✅ HTML semántico
- ✅ ARIA labels donde necesario
- ✅ Compatible con screen readers

### Implementación

**Skip Links:**
```tsx
<a href="#main-content" className="sr-only focus:not-sr-only">
  Saltar al contenido principal
</a>
```

**Keyboard Navigation:**
```
g + d  →  Dashboard
g + v  →  Ventas
g + p  →  Pagos
g + c  →  Clientes
/      →  Search
⌘ + k  →  Command Palette
```

**ARIA Labels:**
```tsx
<button
  onClick={handleDelete}
  aria-label="Eliminar venta V-2024-001"
>
  <Trash2 />
</button>

<table>
  <caption className="sr-only">
    Lista de ventas registradas
  </caption>
  {/* ... */}
</table>
```

**Live Regions:**
```tsx
<div role="status" aria-live="polite" className="sr-only">
  {notification.message}
</div>

<div role="alert" aria-live="assertive">
  {errorMessage}
</div>
```

---

## 🌙 Dark Mode

```tsx
// Theme Provider
<ThemeProvider
  attribute="class"
  defaultTheme="system"
  enableSystem
>
  {children}
</ThemeProvider>

// Theme Toggle
<Button onClick={() => setTheme(theme === 'dark' ? 'light' : 'dark')}>
  <Sun className="dark:hidden" />
  <Moon className="hidden dark:block" />
</Button>
```

**Colores Dark Mode:**
- Background: `hsl(0, 0%, 5%)` - Casi negro
- Foreground: `hsl(0, 0%, 95%)` - Casi blanco
- Contraste: 17:1 (AAA ✅)

---

## 📱 Responsive Design

### Breakpoints

```typescript
sm:  640px   // Mobile landscape
md:  768px   // Tablet
lg:  1024px  // Desktop
xl:  1280px  // Large desktop
2xl: 1536px  // Extra large
```

### Patrones

**Navigation:**
- Desktop: Sidebar persistente
- Mobile: Hamburger menu + Bottom nav bar

**Tables:**
- Desktop: Tabla completa
- Mobile: Cards con información condensada

**Forms:**
- Desktop: Grid 2-3 columnas
- Mobile: Single column stack

**Grids:**
```tsx
<div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
  {/* KPIs */}
</div>
```

---

## 🎬 Animaciones y Transiciones

### Principios

- **Propósito**: Feedback, orientación, deleite
- **Duración**: 150-300ms
- **Easing**: ease-in-out
- **Respeto**: prefers-reduced-motion

### Ejemplos

**Loading States:**
```tsx
<Skeleton className="h-4 w-full" />
<Loader2 className="animate-spin" />
<Progress value={progress} />
```

**Transitions:**
```tsx
<motion.div
  initial={{ opacity: 0, y: 20 }}
  animate={{ opacity: 1, y: 0 }}
  transition={{ duration: 0.2 }}
>
  {children}
</motion.div>
```

**Hover Effects:**
```tsx
<Button className="transition-all hover:scale-105 active:scale-95">
  Acción
</Button>
```

**Toasts:**
```tsx
toast.success('Venta creada exitosamente', {
  description: `Venta ${venta.venta_id} ha sido registrada`,
  action: {
    label: 'Ver',
    onClick: () => router.push(`/ventas/${venta.id}`),
  },
});
```

---

## ⚡ Performance

### Core Web Vitals Objetivos

```
LCP (Largest Contentful Paint): < 2.5s
FID (First Input Delay):         < 100ms
CLS (Cumulative Layout Shift):   < 0.1
INP (Interaction to Next Paint): < 200ms
```

### Optimizaciones

```tsx
// Images
import Image from 'next/image';
<Image src="/logo.png" alt="Logo" width={200} height={50} priority />

// Fonts
import { GeistSans } from 'geist/font/sans';

// Code Splitting
const Chart = dynamic(() => import('@/components/chart'), {
  loading: () => <ChartSkeleton />,
  ssr: false,
});

// Parallel Data Fetching
const [ventas, pagos] = await Promise.all([
  getVentas(),
  getPagos(),
]);
```

---

## 📚 Referencias y Recursos

### Soluciones de Clase Mundial
- [Stripe Dashboard](https://dashboard.stripe.com/)
- [Linear App](https://linear.app/)
- [Vercel Dashboard](https://vercel.com/dashboard)
- [Geckoboard Sales Dashboard](https://www.geckoboard.com/)

### Design Systems
- [shadcn/ui Documentation](https://ui.shadcn.com/)
- [Radix UI Primitives](https://www.radix-ui.com/)
- [Tremor Dashboard Components](https://www.tremor.so/)
- [Vercel Geist Design System](https://vercel.com/geist)

### Accesibilidad
- [WCAG 2.1 Guidelines](https://www.w3.org/TR/WCAG21/)
- [WCAG 2.2 Updates](https://accessibe.com/blog/knowledgebase/wcag-two-point-two)

### Artículos y Estudios
- [Dashboard Design Inspirations 2024](https://muz.li/blog/dashboard-design-inspirations-in-2024/)
- [Modern Dashboard UI/UX Principles](https://medium.com/@allclonescript/20-best-dashboard-ui-ux-design-principles-you-need-in-2025-30b661f2f795)
- [Styling React Apps 2024](https://nishangiri.dev/blog/styling-react-apps)

---

## 🚀 Quick Start

### 1. Install Dependencies

```bash
npx shadcn-ui@latest init

# Instalar componentes necesarios
npx shadcn-ui@latest add button card input label select
npx shadcn-ui@latest add table badge dropdown-menu
npx shadcn-ui@latest add form dialog alert-dialog
npx shadcn-ui@latest add toast skeleton progress

# Instalar adicionales
npm install lucide-react recharts @tremor/react
npm install framer-motion sonner
npm install geist
```

### 2. Configurar Tailwind

```javascript
// tailwind.config.ts
module.exports = {
  darkMode: ["class"],
  content: [
    './pages/**/*.{ts,tsx}',
    './components/**/*.{ts,tsx}',
    './app/**/*.{ts,tsx}',
  ],
  theme: {
    extend: {
      colors: {
        brand: { /* ... */ },
        success: { /* ... */ },
        warning: { /* ... */ },
        error: { /* ... */ },
      },
    },
  },
};
```

### 3. Setup Theme Provider

```tsx
// app/providers.tsx
import { ThemeProvider } from 'next-themes';

export function Providers({ children }) {
  return (
    <ThemeProvider attribute="class" defaultTheme="system" enableSystem>
      {children}
    </ThemeProvider>
  );
}
```

### 4. Crear Primera Vista

```tsx
// app/(dashboard)/page.tsx
import { Card, CardContent, CardHeader, CardTitle } from '@/components/ui/card';
import { Button } from '@/components/ui/button';

export default function Dashboard() {
  return (
    <div className="space-y-8 p-6">
      <div className="flex items-center justify-between">
        <h1 className="text-3xl font-bold">Dashboard</h1>
        <Button>Nueva Venta</Button>
      </div>

      <div className="grid gap-4 md:grid-cols-2 lg:grid-cols-4">
        <Card>
          <CardHeader>
            <CardTitle>Ventas Totales</CardTitle>
          </CardHeader>
          <CardContent>
            <p className="text-3xl font-bold">45</p>
          </CardContent>
        </Card>
        {/* Más KPIs... */}
      </div>
    </div>
  );
}
```

---

## ✅ Checklist de Implementación

### Design System Setup
- [ ] Instalar shadcn/ui y configurar
- [ ] Instalar Radix UI primitives
- [ ] Configurar Tailwind con colores personalizados
- [ ] Setup Theme Provider (dark mode)
- [ ] Instalar Geist o Inter font
- [ ] Configurar Lucide icons

### Componentes Base
- [ ] Implementar Button variants
- [ ] Implementar Card component
- [ ] Implementar Form components
- [ ] Implementar Badge variants
- [ ] Implementar Table component
- [ ] Implementar Dialog/Modal
- [ ] Implementar Toast notifications

### Layouts
- [ ] Crear layout principal con sidebar
- [ ] Implementar responsive navigation
- [ ] Crear mobile bottom nav
- [ ] Implementar breadcrumb
- [ ] Setup skip links

### Vistas Principales
- [ ] Dashboard con KPIs
- [ ] Lista de Ventas (table + filters)
- [ ] Formulario Nueva Venta
- [ ] Vista Detalle de Venta
- [ ] Lista de Pagos
- [ ] Formulario Nuevo Pago
- [ ] Lista de Clientes
- [ ] Lista de Productos

### Accesibilidad
- [ ] Implementar skip links
- [ ] Asegurar focus visible en todos los elementos
- [ ] Asociar labels con inputs
- [ ] Implementar ARIA labels
- [ ] Testear con screen reader
- [ ] Validar contraste de colores
- [ ] Implementar keyboard shortcuts

### Performance
- [ ] Optimizar imágenes con next/image
- [ ] Implementar code splitting
- [ ] Setup fonts optimization
- [ ] Implementar loading states
- [ ] Medir Core Web Vitals
- [ ] Optimizar bundle size

---

## 📊 Métricas de Éxito

### Performance
- [ ] Lighthouse Score > 90
- [ ] LCP < 2.5s
- [ ] FID < 100ms
- [ ] CLS < 0.1

### Accesibilidad
- [ ] WCAG 2.1 Level AA compliance
- [ ] Axe DevTools 0 violations
- [ ] Navegación completa por teclado
- [ ] Screen reader compatible

### UX
- [ ] Tiempo de carga percibido < 1s
- [ ] Tiempo de completar venta < 2min
- [ ] 0 confusión en navegación
- [ ] Feedback inmediato en acciones

---

**Documento preparado por:** SGV BRASA Team
**PRD Completo:** [prds/frontend-design.md](prds/frontend-design.md)
**Última actualización:** 24/11/2025
