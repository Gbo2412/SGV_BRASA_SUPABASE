# PRD - Módulo VENTAS
**Product Requirements Document**

Versión 1.0 | 24/11/2025

---

## 1. Resumen del Módulo

### 1.1 Propósito
El módulo VENTAS es el núcleo del sistema, donde se registran todas las transacciones comerciales. Gestiona ventas al contado y en cuotas, calculando automáticamente estados de pago, saldos pendientes y alertas de cobranza.

### 1.2 Relaciones
- **Requiere:** Módulo CLIENTES (selección de cliente)
- **Requiere:** Módulo PRODUCTOS (selección de producto)
- **Fuente para:** Módulo PAGOS (ventas pendientes)
- **Fuente para:** Módulo DASHBOARD (métricas y alertas)

---

## 2. Modelo de Datos

### 2.1 Tabla: `ventas`

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| `id` | UUID | PRIMARY KEY, AUTO | Identificador único de la venta |
| `venta_id` | VARCHAR(20) | UNIQUE, NOT NULL | ID legible (ej: V-2024-001) |
| `fecha` | DATE | NOT NULL | Fecha de la venta |
| `cliente_id` | UUID | FOREIGN KEY, NOT NULL | Referencia a tabla clientes |
| `producto_id` | UUID | FOREIGN KEY, NOT NULL | Referencia a tabla productos |
| `producto_nombre` | VARCHAR(255) | NOT NULL | Snapshot del nombre del producto |
| `servicio_adicional` | TEXT | NULL | Descripción de servicios extra |
| `monto_total` | DECIMAL(10,2) | NOT NULL | Valor total de la venta |
| `tipo_pago` | ENUM | NOT NULL | 'contado' o 'cuotas' |
| `num_cuotas` | INTEGER | DEFAULT 1 | Número de cuotas (1 si contado) |
| `monto_cuota` | DECIMAL(10,2) | NULL | Monto por cuota (calculado) |
| `monto_pagado` | DECIMAL(10,2) | DEFAULT 0 | Suma de pagos recibidos |
| `saldo_pendiente` | DECIMAL(10,2) | COMPUTED | monto_total - monto_pagado |
| `estado` | ENUM | COMPUTED | 'PAGADO' o 'PENDIENTE' |
| `fecha_primer_pago` | DATE | NULL | Fecha esperada del primer pago |
| `observacion` | TEXT | NULL | Notas adicionales |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Fecha de creación |
| `updated_at` | TIMESTAMP | DEFAULT NOW() | Fecha de actualización |
| `user_id` | UUID | FOREIGN KEY | ID del usuario propietario |

### 2.2 Columnas Computadas (Generated Columns)

```sql
-- Saldo Pendiente (se actualiza automáticamente)
saldo_pendiente DECIMAL(10,2) GENERATED ALWAYS AS (monto_total - monto_pagado) STORED;

-- Estado (se actualiza automáticamente)
estado VARCHAR(20) GENERATED ALWAYS AS (
  CASE
    WHEN monto_total - monto_pagado <= 0 THEN 'PAGADO'
    ELSE 'PENDIENTE'
  END
) STORED;
```

### 2.3 Índices
```sql
CREATE INDEX idx_ventas_venta_id ON ventas(venta_id);
CREATE INDEX idx_ventas_cliente_id ON ventas(cliente_id);
CREATE INDEX idx_ventas_producto_id ON ventas(producto_id);
CREATE INDEX idx_ventas_fecha ON ventas(fecha);
CREATE INDEX idx_ventas_estado ON ventas(estado);
CREATE INDEX idx_ventas_tipo_pago ON ventas(tipo_pago);
CREATE INDEX idx_ventas_user_id ON ventas(user_id);
```

### 2.4 Ejemplo de Registro
```json
{
  "id": "345e6789-e89b-12d3-a456-426614174222",
  "venta_id": "V-2024-001",
  "fecha": "2024-11-24",
  "cliente_id": "123e4567-e89b-12d3-a456-426614174000",
  "producto_id": "234e5678-e89b-12d3-a456-426614174111",
  "producto_nombre": "Parrilla Familiar",
  "servicio_adicional": "Incluye decoración especial",
  "monto_total": 600.00,
  "tipo_pago": "cuotas",
  "num_cuotas": 3,
  "monto_cuota": 200.00,
  "monto_pagado": 200.00,
  "saldo_pendiente": 400.00,
  "estado": "PENDIENTE",
  "fecha_primer_pago": "2024-11-24",
  "observacion": "Cliente prefiere pago mensual",
  "created_at": "2024-11-24T10:00:00Z",
  "updated_at": "2024-11-24T15:30:00Z",
  "user_id": "user-uuid-here"
}
```

---

## 3. Funcionalidades

### 3.1 Crear Venta (CREATE)

**Entrada:**
```typescript
interface CreateVentaInput {
  fecha: string;                      // Requerido (formato: YYYY-MM-DD)

  // Cliente: Puede ser existente o nuevo
  cliente_id?: string;                // UUID si es existente
  cliente_nuevo?: {                   // Si no existe, crear nuevo
    nombre: string;
    email?: string;
    telefono?: string;
  };

  // Producto: Selección de producto existente
  producto_id: string;                // Requerido (UUID)

  servicio_adicional?: string;        // Opcional
  monto_total: number;                // Requerido (editable, pre-cargado del producto)
  tipo_pago: 'contado' | 'cuotas';    // Requerido
  num_cuotas?: number;                // Requerido si tipo_pago = 'cuotas'
  fecha_primer_pago?: string;         // Opcional
  observacion?: string;               // Opcional
}
```

**Validaciones:**
- ✅ Todos los campos requeridos deben estar presentes
- ✅ **Cliente:** Debe proporcionar `cliente_id` (existente) O `cliente_nuevo` (crear nuevo)
- ✅ Si `cliente_id` proporcionado, debe existir en tabla clientes
- ✅ Si `cliente_nuevo` proporcionado, se crea automáticamente antes de la venta
- ✅ `producto_id` debe existir en tabla productos
- ✅ `monto_total` se pre-carga con el `precio_base` del producto pero es editable
- ✅ `monto_total` debe ser > 0
- ✅ Si `tipo_pago` = 'cuotas', `num_cuotas` debe ser >= 2
- ✅ Si `tipo_pago` = 'contado', `num_cuotas` = 1
- ✅ `fecha` no puede ser futura
- ✅ `venta_id` se genera automáticamente: `V-{año}-{número_secuencial}`

**Cálculos Automáticos:**
```typescript
// Monto por cuota
monto_cuota = tipo_pago === 'cuotas'
  ? Math.round((monto_total / num_cuotas) * 100) / 100
  : monto_total;

// Estado inicial
estado = 'PENDIENTE';
monto_pagado = 0;
saldo_pendiente = monto_total;
```

**Proceso:**
1. Validar datos de entrada
2. **Gestión de Cliente:**
   - Si `cliente_id` proporcionado: Verificar existencia
   - Si `cliente_nuevo` proporcionado: Crear cliente nuevo automáticamente
3. Verificar existencia de producto
4. **Auto-completar monto:** Pre-cargar `monto_total` con `precio_base` del producto
5. Generar `venta_id` único
6. Capturar snapshot del nombre del producto
7. Calcular `monto_cuota`
8. Insertar en base de datos con valores iniciales
9. Retornar venta creada con datos del cliente (nuevo o existente)

**Respuesta:**
```typescript
interface VentaCreated {
  success: boolean;
  data: Venta & {
    cliente: Cliente;
    producto: Producto;
  };
  message: string;
}
```

### 3.2 Leer Ventas (READ)

**3.2.1 Listar Todas las Ventas**

**Entrada:**
```typescript
interface ListVentasInput {
  page?: number;                          // Default: 1
  limit?: number;                         // Default: 50
  search?: string;                        // Búsqueda por venta_id, cliente, producto
  cliente_id?: string;                    // Filtro por cliente
  estado?: 'PAGADO' | 'PENDIENTE' | 'all'; // Default: 'all'
  tipo_pago?: 'contado' | 'cuotas' | 'all'; // Default: 'all'
  fecha_desde?: string;                   // Filtro fecha inicio
  fecha_hasta?: string;                   // Filtro fecha fin
  sortBy?: 'fecha' | 'monto_total' | 'saldo_pendiente'; // Default: 'fecha'
  sortOrder?: 'asc' | 'desc';             // Default: 'desc'
}
```

**Respuesta:**
```typescript
interface ListVentasResponse {
  data: Array<Venta & {
    cliente: Pick<Cliente, 'nombre' | 'email'>;
    producto: Pick<Producto, 'nombre' | 'categoria'>;
  }>;
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
  summary: {
    totalVentas: number;
    montoTotal: number;
    montoPagado: number;
    saldoPendiente: number;
  };
}
```

**3.2.2 Obtener Venta por ID**

**Entrada:**
```typescript
interface GetVentaInput {
  id: string; // UUID
}
```

**Respuesta:**
```typescript
interface GetVentaResponse {
  data: Venta & {
    cliente: Cliente;
    producto: Producto;
    pagos: Pago[];
  };
  stats: {
    progresoPago: number;        // Porcentaje (0-100)
    cuotasPagadas: number;
    cuotasPendientes: number;
    proximoPago?: Date;
    diasAtraso?: number;
  };
}
```

### 3.3 Actualizar Venta (UPDATE)

**Entrada:**
```typescript
interface UpdateVentaInput {
  id: string;                           // Requerido (UUID)
  fecha?: string;
  servicio_adicional?: string;
  monto_total?: number;                 // ⚠️ Recalcula saldo_pendiente
  tipo_pago?: 'contado' | 'cuotas';
  num_cuotas?: number;                  // ⚠️ Recalcula monto_cuota
  fecha_primer_pago?: string;
  observacion?: string;
}
```

**Validaciones:**
- ✅ Venta debe existir
- ❌ NO permitir cambiar `cliente_id` ni `producto_id` (integridad)
- ⚠️ Si se cambia `monto_total` y ya hay pagos registrados, advertir al usuario
- ⚠️ Si se cambia `num_cuotas`, recalcular `monto_cuota`

**Proceso:**
1. Validar que venta existe
2. Validar nuevos datos
3. Recalcular campos derivados si es necesario
4. Actualizar `updated_at`
5. Retornar venta actualizada

**Nota Importante:**
- Los cambios en `monto_total` afectan el `saldo_pendiente`
- Si ya hay pagos registrados, puede generar inconsistencias
- Recomendar NO editar monto si ya hay pagos (mostrar warning)

### 3.4 Eliminar Venta (DELETE)

**Entrada:**
```typescript
interface DeleteVentaInput {
  id: string; // UUID
}
```

**Validaciones:**
- ✅ Venta debe existir
- ❌ NO permitir eliminación si tiene pagos registrados
- ✅ Mostrar advertencia antes de eliminar

**Proceso:**
1. Verificar si tiene pagos asociados
2. Si tiene pagos: rechazar eliminación y sugerir anular venta (Fase 2)
3. Si NO tiene pagos: permitir eliminación física
4. Retornar confirmación

---

## 4. Interfaz de Usuario

### 4.1 Vista Principal: Lista de Ventas

**Componentes:**
- Tabla con columnas:
  - Venta ID
  - Fecha
  - Cliente (nombre)
  - Producto
  - Monto Total
  - Pagado
  - Saldo Pendiente
  - Tipo Pago
  - Estado (badge: verde/amarillo)
- Barra de búsqueda
- Filtros: Cliente, Estado, Tipo Pago, Rango de fechas
- Tarjeta resumen: Total Ventas, Monto Total, Saldo Pendiente
- Botón "Nueva Venta"
- Paginación

**Acciones por fila:**
- Ver detalle (icono ojo)
- Editar (icono lápiz) - solo si no tiene pagos
- Registrar pago (icono $)
- Ver pagos (icono lista)

**Indicadores Visuales:**
- Estado PAGADO: badge verde
- Estado PENDIENTE: badge amarillo
- Ventas con pagos atrasados: fila con borde rojo (Fase 2)

### 4.2 Formulario: Nueva Venta

**Layout:**
```
┌─────────────────────────────────────┐
│  [Crear Nueva Venta]                │
├─────────────────────────────────────┤
│  Fecha: [__/__/____] *              │
│                                     │
│  Cliente: [Buscar cliente...] * 🔍  │
│  ┌─────────────────────────────┐   │
│  │ Juan Pérez García           │   │
│  │ María López Ruiz            │   │
│  └─────────────────────────────┘   │
│                                     │
│  Producto: [Buscar producto...] * 🔍│
│  ┌─────────────────────────────┐   │
│  │ Parrilla Familiar - S/150   │   │
│  │ Parrilla Premium - S/250    │   │
│  └─────────────────────────────┘   │
│                                     │
│  Servicio Adicional:                │
│  [_________________________________]│
│                                     │
│  Monto Total: S/ [_______] *        │
│                                     │
│  Tipo de Pago: *                    │
│  (•) Contado  ( ) Cuotas            │
│                                     │
│  [Si Cuotas está seleccionado:]    │
│  Número de Cuotas: [___]            │
│  Monto por Cuota: S/ 200.00         │
│  Fecha Primer Pago: [__/__/____]    │
│                                     │
│  Observaciones:                     │
│  [_________________________________]│
│  [_________________________________]│
│                                     │
│  [Cancelar]  [Crear Venta]         │
└─────────────────────────────────────┘
```

**Funcionalidad Dinámica:**
- Campo Cliente: Autocomplete con búsqueda en tiempo real
- Campo Producto: Autocomplete que muestra precio base
- Monto Total: Pre-rellenado con precio del producto (editable)
- Al seleccionar "Cuotas": habilitar campos de cuotas
- Monto por Cuota: calculado automáticamente al ingresar num_cuotas
- Validación en tiempo real

### 4.3 Vista Detalle: Información de Venta

**Secciones:**

**Panel Superior: Información de la Venta**
```
┌─────────────────────────────────────────────────┐
│  VENTA V-2024-001                               │
│  Fecha: 24/11/2024                              │
│  Estado: [PENDIENTE 🟡]         [Editar]        │
└─────────────────────────────────────────────────┘
```

**Sección Cliente y Producto:**
```
┌──────────────────────┬──────────────────────────┐
│  CLIENTE             │  PRODUCTO                │
│  Juan Pérez García   │  Parrilla Familiar       │
│  juan@email.com      │  Categoría: Parrillas    │
│  +51 987654321       │  Precio: S/ 600.00       │
│  [Ver Cliente]       │  [Ver Producto]          │
└──────────────────────┴──────────────────────────┘
```

**Sección Montos:**
```
┌─────────────────────────────────────────────────┐
│  RESUMEN DE PAGOS                               │
├─────────────────────────────────────────────────┤
│  Monto Total:        S/ 600.00                  │
│  Monto Pagado:       S/ 200.00                  │
│  Saldo Pendiente:    S/ 400.00                  │
│                                                 │
│  Tipo de Pago:       Cuotas (3)                 │
│  Monto por Cuota:    S/ 200.00                  │
│  Cuotas Pagadas:     1 de 3                     │
│                                                 │
│  [====--------]  33% completado                 │
└─────────────────────────────────────────────────┘
```

**Sección Historial de Pagos:**
```
┌─────────────────────────────────────────────────┐
│  HISTORIAL DE PAGOS              [+ Nuevo Pago] │
├─────────────────────────────────────────────────┤
│  P-2024-001  │ 24/11/24 │ Cuota 1 │ S/ 200.00 │
│  ...         │ ...      │ ...     │ ...       │
└─────────────────────────────────────────────────┘
```

**Botones de Acción:**
- Registrar Nuevo Pago
- Editar Venta (solo si no tiene pagos)
- Ver PDF/Imprimir (Fase 2)
- Eliminar Venta (solo si no tiene pagos)

---

## 5. API Endpoints

### 5.1 REST API (Next.js API Routes)

```typescript
// GET /api/ventas
// Lista todas las ventas con filtros
GET /api/ventas?page=1&limit=50&estado=PENDIENTE&cliente_id=xxx

// GET /api/ventas/[id]
// Obtiene una venta específica con todos sus datos relacionados
GET /api/ventas/345e6789-e89b-12d3-a456-426614174222

// GET /api/ventas/cliente/[cliente_id]
// Obtiene todas las ventas de un cliente
GET /api/ventas/cliente/123e4567-e89b-12d3-a456-426614174000

// POST /api/ventas
// Crea una nueva venta
POST /api/ventas
Body: { fecha, cliente_id, producto_id, monto_total, tipo_pago, ... }

// PUT /api/ventas/[id]
// Actualiza una venta existente
PUT /api/ventas/345e6789-e89b-12d3-a456-426614174222
Body: { monto_total, observacion, ... }

// DELETE /api/ventas/[id]
// Elimina una venta (solo si no tiene pagos)
DELETE /api/ventas/345e6789-e89b-12d3-a456-426614174222
```

### 5.2 Server Actions

```typescript
// app/actions/ventas.ts
export async function createVenta(data: CreateVentaInput)
export async function updateVenta(id: string, data: UpdateVentaInput)
export async function deleteVenta(id: string)
export async function getVentas(filters: ListVentasInput)
export async function getVentaById(id: string)
export async function getVentasByCliente(clienteId: string)
export async function getVentasPendientes()
```

---

## 6. Reglas de Negocio

### 6.1 Generación de Venta ID
- Formato: `V-{YYYY}-{NNN}`
- Ejemplo: `V-2024-001`, `V-2024-002`
- Se reinicia numeración cada año
- Se rellena con ceros a la izquierda

```typescript
async function generateVentaId(): Promise<string> {
  const year = new Date().getFullYear();
  const prefix = `V-${year}-`;

  const lastVenta = await db
    .select()
    .from(ventas)
    .where(like(venta_id, `${prefix}%`))
    .orderBy(desc(venta_id))
    .limit(1);

  const lastNumber = lastVenta?.[0]
    ? parseInt(lastVenta[0].venta_id.split('-')[2])
    : 0;

  const newNumber = (lastNumber + 1).toString().padStart(3, '0');
  return `${prefix}${newNumber}`;
}
```

### 6.2 Snapshot de Producto
- Al crear una venta, se guarda el `producto_nombre` actual
- Esto preserva el historial incluso si el producto se edita después
- Los cambios en el catálogo de productos NO afectan ventas históricas

### 6.3 Cálculo de Monto por Cuota
```typescript
function calcularMontoCuota(montoTotal: number, numCuotas: number): number {
  // Redondear a 2 decimales para evitar fracciones de céntimos
  return Math.round((montoTotal / numCuotas) * 100) / 100;
}
```

**Manejo de Diferencias por Redondeo:**
- Si hay diferencia por redondeo (ej: 100/3 = 33.33 x 3 = 99.99)
- La última cuota absorbe la diferencia (33.34)
- Fase 2: Permitir especificar montos personalizados por cuota

### 6.4 Actualización Automática de Estado
- El estado se calcula automáticamente basado en `saldo_pendiente`
- NO se actualiza manualmente
- Si `saldo_pendiente <= 0` → `estado = 'PAGADO'`
- Si `saldo_pendiente > 0` → `estado = 'PENDIENTE'`

### 6.5 Actualización de Monto Pagado
- `monto_pagado` se actualiza automáticamente vía trigger o computed column
- Se recalcula como: `SUM(pagos.monto) WHERE pagos.venta_id = ventas.id`
- En tiempo real al registrar/eliminar pagos

**Implementación (Trigger en Supabase):**
```sql
CREATE OR REPLACE FUNCTION update_venta_monto_pagado()
RETURNS TRIGGER AS $$
BEGIN
  UPDATE ventas
  SET monto_pagado = (
    SELECT COALESCE(SUM(monto), 0)
    FROM pagos
    WHERE venta_id = NEW.venta_id
  )
  WHERE id = NEW.venta_id;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_update_monto_pagado
AFTER INSERT OR UPDATE OR DELETE ON pagos
FOR EACH ROW
EXECUTE FUNCTION update_venta_monto_pagado();
```

---

## 7. Casos de Uso

### 7.1 UC-VEN-01: Registrar Venta al Contado

**Actor:** Usuario

**Precondiciones:**
- Usuario autenticado
- Existen clientes y productos en el sistema

**Flujo Principal:**
1. Usuario navega a módulo VENTAS
2. Click en "Nueva Venta"
3. Sistema muestra formulario
4. Usuario selecciona fecha (hoy por default)
5. Usuario busca y selecciona cliente
6. Usuario busca y selecciona producto
7. Sistema pre-rellena monto con precio del producto
8. Usuario ajusta monto si es necesario
9. Usuario selecciona "Contado"
10. Usuario agrega observaciones (opcional)
11. Click en "Crear Venta"
12. Sistema valida datos
13. Sistema genera `venta_id`
14. Sistema crea venta con:
    - `tipo_pago = 'contado'`
    - `num_cuotas = 1`
    - `monto_pagado = 0`
    - `estado = 'PENDIENTE'`
15. Sistema muestra mensaje de éxito
16. Sistema redirige a vista detalle de venta

**Postcondición:**
- Venta creada aparece en lista con estado PENDIENTE
- Usuario puede registrar pago para completar venta

### 7.2 UC-VEN-02: Registrar Venta en Cuotas

**Actor:** Usuario

**Flujo Principal:**
1-7. (Igual que UC-VEN-01)
8. Usuario selecciona "Cuotas"
9. Sistema habilita campo "Número de Cuotas"
10. Usuario ingresa número de cuotas (ej: 3)
11. Sistema calcula y muestra monto por cuota (ej: S/ 200.00)
12. Usuario opcionalmente ingresa fecha del primer pago
13. Usuario agrega observaciones
14. Click en "Crear Venta"
15. Sistema valida datos (num_cuotas >= 2)
16. Sistema genera `venta_id`
17. Sistema crea venta con:
    - `tipo_pago = 'cuotas'`
    - `num_cuotas = 3`
    - `monto_cuota = 200.00`
    - `monto_pagado = 0`
    - `estado = 'PENDIENTE'`
18. Sistema muestra mensaje de éxito
19. Sistema redirige a vista detalle

**Postcondición:**
- Venta creada con plan de cuotas
- Usuario puede registrar pagos parciales

### 7.3 UC-VEN-03: Consultar Estado de Venta

**Actor:** Usuario

**Flujo Principal:**
1. Usuario navega a lista de ventas
2. Usuario hace click en venta específica
3. Sistema muestra vista detalle con:
   - Información de venta
   - Datos de cliente y producto
   - Resumen de montos y progreso
   - Historial completo de pagos
4. Usuario visualiza:
   - Cuánto ha pagado el cliente
   - Cuánto falta por pagar
   - Progreso visual (barra de progreso)
5. Usuario puede tomar acciones:
   - Registrar nuevo pago
   - Editar venta (si no tiene pagos)
   - Ver cliente o producto
   - Eliminar venta (si no tiene pagos)

---

## 8. Validaciones y Errores

### 8.1 Validaciones Frontend (Zod Schema)

```typescript
import { z } from 'zod';

export const VentaSchema = z.object({
  fecha: z.string()
    .regex(/^\d{4}-\d{2}-\d{2}$/, 'Formato de fecha inválido')
    .refine(date => new Date(date) <= new Date(), 'La fecha no puede ser futura'),

  cliente_id: z.string()
    .uuid('Cliente inválido'),

  producto_id: z.string()
    .uuid('Producto inválido'),

  servicio_adicional: z.string()
    .max(500, 'Descripción demasiado larga')
    .optional(),

  monto_total: z.number()
    .positive('Monto debe ser mayor a 0')
    .multipleOf(0.01, 'Monto debe tener máximo 2 decimales'),

  tipo_pago: z.enum(['contado', 'cuotas']),

  num_cuotas: z.number()
    .int('Número de cuotas debe ser entero')
    .min(1, 'Mínimo 1 cuota')
    .optional(),

  fecha_primer_pago: z.string()
    .regex(/^\d{4}-\d{2}-\d{2}$/)
    .optional(),

  observacion: z.string()
    .max(1000, 'Observación demasiado larga')
    .optional(),
}).refine(data => {
  // Si tipo_pago es 'cuotas', num_cuotas debe ser >= 2
  if (data.tipo_pago === 'cuotas') {
    return data.num_cuotas && data.num_cuotas >= 2;
  }
  return true;
}, {
  message: 'Ventas en cuotas deben tener al menos 2 cuotas',
  path: ['num_cuotas'],
});
```

### 8.2 Mensajes de Error

| Código | Mensaje | Acción |
|--------|---------|--------|
| `VEN_001` | Fecha es obligatoria | Solicitar fecha |
| `VEN_002` | Cliente es obligatorio | Seleccionar cliente |
| `VEN_003` | Producto es obligatorio | Seleccionar producto |
| `VEN_004` | Monto total debe ser mayor a 0 | Corregir monto |
| `VEN_005` | Fecha no puede ser futura | Corregir fecha |
| `VEN_006` | Cuotas deben ser al menos 2 | Aumentar cuotas o elegir contado |
| `VEN_007` | Cliente no encontrado | Verificar cliente |
| `VEN_008` | Producto no encontrado | Verificar producto |
| `VEN_009` | No se puede editar: tiene pagos registrados | Crear nueva venta o ajustar pagos |
| `VEN_010` | No se puede eliminar: tiene pagos registrados | Contactar soporte (anular venta) |

---

## 9. Testing

### 9.1 Tests Unitarios

```typescript
describe('Módulo Ventas', () => {
  describe('createVenta', () => {
    it('debería crear venta al contado correctamente', async () => {
      const input: CreateVentaInput = {
        fecha: '2024-11-24',
        cliente_id: 'cliente-uuid',
        producto_id: 'producto-uuid',
        monto_total: 150,
        tipo_pago: 'contado',
      };
      const result = await createVenta(input);
      expect(result.success).toBe(true);
      expect(result.data.num_cuotas).toBe(1);
      expect(result.data.estado).toBe('PENDIENTE');
      expect(result.data.monto_pagado).toBe(0);
    });

    it('debería crear venta en cuotas con cálculo correcto', async () => {
      const input: CreateVentaInput = {
        fecha: '2024-11-24',
        cliente_id: 'cliente-uuid',
        producto_id: 'producto-uuid',
        monto_total: 600,
        tipo_pago: 'cuotas',
        num_cuotas: 3,
      };
      const result = await createVenta(input);
      expect(result.data.monto_cuota).toBe(200);
    });

    it('debería rechazar venta en cuotas con solo 1 cuota', async () => {
      const input: CreateVentaInput = {
        fecha: '2024-11-24',
        cliente_id: 'cliente-uuid',
        producto_id: 'producto-uuid',
        monto_total: 100,
        tipo_pago: 'cuotas',
        num_cuotas: 1,
      };
      await expect(createVenta(input)).rejects.toThrow();
    });
  });

  describe('calcularMontoCuota', () => {
    it('debería calcular cuotas exactas', () => {
      expect(calcularMontoCuota(600, 3)).toBe(200);
    });

    it('debería redondear a 2 decimales', () => {
      expect(calcularMontoCuota(100, 3)).toBe(33.33);
    });
  });

  describe('updateVenta', () => {
    it('debería actualizar venta sin pagos', async () => {
      const venta = await createVenta({ /* ... */ });
      const updated = await updateVenta(venta.id, { monto_total: 200 });
      expect(updated.data.monto_total).toBe(200);
      expect(updated.data.saldo_pendiente).toBe(200);
    });

    it('debería rechazar actualización de venta con pagos', async () => {
      const venta = await createVenta({ /* ... */ });
      await createPago({ venta_id: venta.id, monto: 100 });
      await expect(
        updateVenta(venta.id, { monto_total: 200 })
      ).rejects.toThrow();
    });
  });
});
```

### 9.2 Tests de Integración

- Crear venta y verificar en base de datos
- Registrar pago y verificar actualización de monto_pagado
- Verificar cambio automático de estado a PAGADO
- Relaciones con clientes y productos
- Triggers de actualización automática

### 9.3 Tests E2E

- Flujo completo: crear venta → registrar pagos → verificar estado PAGADO
- Búsqueda y filtros en lista
- Vista detalle con todos los datos relacionados
- Editar venta sin pagos
- Intentar editar venta con pagos (debe fallar)

---

## 10. Performance

### 10.1 Optimizaciones
- Índices en columnas de filtro frecuente
- Join con clientes y productos en queries de lista
- Paginación (50 ventas por página)
- Cache de ventas pendientes (2 minutos)
- Lazy loading de historial de pagos en vista detalle

### 10.2 Métricas
- Tiempo de carga lista: < 700ms
- Tiempo de crear venta: < 1s
- Tiempo de actualización de monto_pagado: < 500ms

---

## 11. Seguridad

### 11.1 Row Level Security (RLS)

```sql
-- Los usuarios solo ven sus propias ventas
CREATE POLICY "Users can view own ventas"
  ON ventas FOR SELECT
  USING (auth.uid() = user_id);

-- Los usuarios solo pueden crear ventas propias
CREATE POLICY "Users can insert own ventas"
  ON ventas FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- Los usuarios solo pueden actualizar sus propias ventas
CREATE POLICY "Users can update own ventas"
  ON ventas FOR UPDATE
  USING (auth.uid() = user_id);

-- Los usuarios solo pueden eliminar sus propias ventas sin pagos
CREATE POLICY "Users can delete own ventas without payments"
  ON ventas FOR DELETE
  USING (
    auth.uid() = user_id
    AND NOT EXISTS (
      SELECT 1 FROM pagos WHERE venta_id = ventas.id
    )
  );
```

---

## 12. Features Futuras (Post-MVP)

### 12.1 Cuotas Personalizadas
- Permitir definir monto diferente por cada cuota
- Tabla adicional: `cuotas_plan`
- Fechas de vencimiento por cuota

### 12.2 Alertas de Cobranza
- Email/SMS automático antes de vencimiento
- Notificaciones de pagos atrasados
- Recordatorios personalizables

### 12.3 Anulación de Ventas
- Soft delete con motivo
- Historial de ventas anuladas
- Reversa de pagos

### 12.4 Documentos y Comprobantes
- Generación de PDF con datos de venta
- Proformas y cotizaciones
- Comprobantes de pago

### 12.5 Ventas Multi-Producto
- Permitir múltiples productos por venta
- Tabla intermedia: `venta_items`
- Subtotales y descuentos

---

**Documento preparado por:** SGV BRASA Team
**Última actualización:** 24/11/2025
