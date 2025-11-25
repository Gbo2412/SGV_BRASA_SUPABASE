# PRD - Módulo PAGOS
**Product Requirements Document**

Versión 1.0 | 24/11/2025

---

## 1. Resumen del Módulo

### 1.1 Propósito
El módulo PAGOS registra todos los pagos recibidos de los clientes. Cada pago está vinculado a una venta específica y actualiza automáticamente el estado de pago, el saldo pendiente y el estado de la venta en tiempo real.

### 1.2 Relaciones
- **Requiere:** Módulo VENTAS (vinculación de pago a venta)
- **Impacta:** Módulo VENTAS (actualiza monto_pagado, saldo_pendiente, estado)
- **Fuente para:** Módulo DASHBOARD (métricas de cobranza)

---

## 2. Modelo de Datos

### 2.1 Tabla: `pagos`

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| `id` | UUID | PRIMARY KEY, AUTO | Identificador único del pago |
| `pago_id` | VARCHAR(20) | UNIQUE, NOT NULL | ID legible (ej: P-2024-001) |
| `venta_id` | UUID | FOREIGN KEY, NOT NULL | Referencia a tabla ventas |
| `fecha_pago` | DATE | NOT NULL | Fecha en que se recibió el pago |
| `num_cuota` | INTEGER | NOT NULL | Número de cuota (0 si contado) |
| `monto` | DECIMAL(10,2) | NOT NULL | Monto recibido en este pago |
| `metodo_pago` | ENUM | NOT NULL | Método de pago utilizado |
| `comprobante` | VARCHAR(100) | NULL | Número de operación/recibo |
| `observacion` | TEXT | NULL | Notas sobre el pago |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Fecha de registro |
| `updated_at` | TIMESTAMP | DEFAULT NOW() | Fecha de actualización |
| `user_id` | UUID | FOREIGN KEY | ID del usuario propietario |

### 2.2 Enum: `metodo_pago`
```sql
CREATE TYPE metodo_pago_enum AS ENUM (
  'efectivo',
  'transferencia',
  'yape',
  'plin',
  'tarjeta_credito',
  'tarjeta_debito',
  'otro'
);
```

### 2.3 Índices
```sql
CREATE INDEX idx_pagos_pago_id ON pagos(pago_id);
CREATE INDEX idx_pagos_venta_id ON pagos(venta_id);
CREATE INDEX idx_pagos_fecha_pago ON pagos(fecha_pago);
CREATE INDEX idx_pagos_metodo_pago ON pagos(metodo_pago);
CREATE INDEX idx_pagos_user_id ON pagos(user_id);
```

### 2.4 Constraints
```sql
-- El monto del pago debe ser positivo
ALTER TABLE pagos ADD CONSTRAINT check_monto_positivo
  CHECK (monto > 0);

-- El número de cuota debe ser >= 0
ALTER TABLE pagos ADD CONSTRAINT check_num_cuota_valido
  CHECK (num_cuota >= 0);

-- La suma de pagos no puede exceder el monto total de la venta (Trigger)
CREATE OR REPLACE FUNCTION check_pago_no_excede_venta()
RETURNS TRIGGER AS $$
DECLARE
  v_monto_total DECIMAL(10,2);
  v_monto_pagado DECIMAL(10,2);
BEGIN
  -- Obtener monto total de la venta
  SELECT monto_total INTO v_monto_total
  FROM ventas
  WHERE id = NEW.venta_id;

  -- Calcular monto ya pagado + nuevo pago
  SELECT COALESCE(SUM(monto), 0) + NEW.monto INTO v_monto_pagado
  FROM pagos
  WHERE venta_id = NEW.venta_id
    AND id != COALESCE(NEW.id, '00000000-0000-0000-0000-000000000000');

  -- Validar que no exceda
  IF v_monto_pagado > v_monto_total THEN
    RAISE EXCEPTION 'El pago excede el saldo pendiente de la venta';
  END IF;

  RETURN NEW;
END;
$$ LANGUAGE plpgsql;

CREATE TRIGGER trigger_check_pago_no_excede
BEFORE INSERT OR UPDATE ON pagos
FOR EACH ROW
EXECUTE FUNCTION check_pago_no_excede_venta();
```

### 2.5 Ejemplo de Registro
```json
{
  "id": "456e7890-e89b-12d3-a456-426614174333",
  "pago_id": "P-2024-001",
  "venta_id": "345e6789-e89b-12d3-a456-426614174222",
  "fecha_pago": "2024-11-24",
  "num_cuota": 1,
  "monto": 200.00,
  "metodo_pago": "transferencia",
  "comprobante": "OP-123456789",
  "observacion": "Primera cuota pagada a tiempo",
  "created_at": "2024-11-24T10:00:00Z",
  "updated_at": "2024-11-24T10:00:00Z",
  "user_id": "user-uuid-here"
}
```

---

## 3. Funcionalidades

### 3.1 Crear Pago (CREATE)

**Entrada:**
```typescript
interface CreatePagoInput {
  venta_id: string;                        // Requerido (UUID)
  fecha_pago: string;                      // Requerido (formato: YYYY-MM-DD)
  num_cuota: number;                       // Requerido (0 si contado, >= 1 si cuotas)
  monto: number;                           // Requerido
  metodo_pago: MetodoPago;                 // Requerido
  comprobante?: string;                    // Opcional
  observacion?: string;                    // Opcional
}

type MetodoPago =
  | 'efectivo'
  | 'transferencia'
  | 'yape'
  | 'plin'
  | 'tarjeta_credito'
  | 'tarjeta_debito'
  | 'otro';
```

**Validaciones:**
- ✅ Todos los campos requeridos presentes
- ✅ `venta_id` debe existir en tabla ventas
- ✅ `monto` debe ser > 0
- ✅ `monto` no debe exceder `saldo_pendiente` de la venta
- ✅ `fecha_pago` no puede ser futura
- ✅ `num_cuota` debe ser válido para el tipo de pago:
  - Si venta es "contado": `num_cuota = 0`
  - Si venta es "cuotas": `num_cuota >= 1` y `<= num_cuotas`
- ✅ `pago_id` se genera automáticamente: `P-{año}-{número_secuencial}`

**Proceso:**
1. Validar datos de entrada
2. Verificar existencia de venta
3. Verificar que venta no esté completamente pagada
4. Validar que monto no exceda saldo pendiente
5. Generar `pago_id` único
6. Insertar pago en base de datos
7. **TRIGGER AUTOMÁTICO**: Actualizar `monto_pagado` en tabla ventas
8. **COLUMNA COMPUTADA**: Recalcular `saldo_pendiente`
9. **COLUMNA COMPUTADA**: Actualizar `estado` si corresponde
10. Retornar pago creado con datos de venta actualizada

**Respuesta:**
```typescript
interface PagoCreated {
  success: boolean;
  data: Pago & {
    venta: Venta & {
      cliente: Pick<Cliente, 'nombre'>;
    };
  };
  message: string;
  ventaActualizada: {
    monto_pagado: number;
    saldo_pendiente: number;
    estado: 'PAGADO' | 'PENDIENTE';
  };
}
```

### 3.2 Leer Pagos (READ)

**3.2.1 Listar Todos los Pagos**

**Entrada:**
```typescript
interface ListPagosInput {
  page?: number;                               // Default: 1
  limit?: number;                              // Default: 50
  venta_id?: string;                           // Filtro por venta
  cliente_id?: string;                         // Filtro por cliente
  metodo_pago?: MetodoPago;                    // Filtro por método
  fecha_desde?: string;                        // Filtro fecha inicio
  fecha_hasta?: string;                        // Filtro fecha fin
  sortBy?: 'fecha_pago' | 'monto';             // Default: 'fecha_pago'
  sortOrder?: 'asc' | 'desc';                  // Default: 'desc'
}
```

**Respuesta:**
```typescript
interface ListPagosResponse {
  data: Array<Pago & {
    venta: Pick<Venta, 'venta_id' | 'monto_total' | 'estado'> & {
      cliente: Pick<Cliente, 'nombre' | 'email'>;
      producto: Pick<Producto, 'nombre'>;
    };
  }>;
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
  summary: {
    totalPagos: number;
    montoTotal: number;
    porMetodo: Record<MetodoPago, number>;
  };
}
```

**3.2.2 Obtener Pago por ID**

**Entrada:**
```typescript
interface GetPagoInput {
  id: string; // UUID
}
```

**Respuesta:**
```typescript
interface GetPagoResponse {
  data: Pago & {
    venta: Venta & {
      cliente: Cliente;
      producto: Producto;
    };
  };
}
```

**3.2.3 Obtener Pagos de una Venta**

**Entrada:**
```typescript
interface GetPagosByVentaInput {
  venta_id: string; // UUID
}
```

**Respuesta:**
```typescript
interface GetPagosByVentaResponse {
  data: Pago[];
  summary: {
    totalPagos: number;
    montoPagado: number;
    saldoPendiente: number;
    cuotasPagadas: number;
  };
}
```

### 3.3 Actualizar Pago (UPDATE)

**Entrada:**
```typescript
interface UpdatePagoInput {
  id: string;                    // Requerido (UUID)
  fecha_pago?: string;
  monto?: number;                // ⚠️ Recalcula saldo de venta
  metodo_pago?: MetodoPago;
  comprobante?: string;
  observacion?: string;
}
```

**Validaciones:**
- ✅ Pago debe existir
- ❌ NO permitir cambiar `venta_id` (integridad)
- ❌ NO permitir cambiar `num_cuota` (integridad)
- ⚠️ Si se cambia `monto`, validar que no exceda saldo disponible
- ⚠️ Al cambiar `monto`, recalcular automáticamente estado de venta

**Proceso:**
1. Validar que pago existe
2. Validar nuevos datos
3. Si se cambia `monto`, verificar que no exceda límite
4. Actualizar pago
5. **TRIGGER AUTOMÁTICO**: Recalcular `monto_pagado` en venta
6. Retornar pago actualizado con venta actualizada

### 3.4 Eliminar Pago (DELETE)

**Entrada:**
```typescript
interface DeletePagoInput {
  id: string; // UUID
}
```

**Validaciones:**
- ✅ Pago debe existir
- ⚠️ Al eliminar, recalcular estado de venta

**Proceso:**
1. Verificar que pago existe
2. Eliminar pago de base de datos
3. **TRIGGER AUTOMÁTICO**: Recalcular `monto_pagado` en venta
4. **COLUMNA COMPUTADA**: Recalcular `saldo_pendiente`
5. **COLUMNA COMPUTADA**: Actualizar `estado` (puede volver a PENDIENTE)
6. Retornar confirmación

---

## 4. Interfaz de Usuario

### 4.1 Vista Principal: Lista de Pagos

**Componentes:**
- Tabla con columnas:
  - Pago ID
  - Fecha Pago
  - Venta ID (link a venta)
  - Cliente (nombre)
  - Cuota (ej: "1 de 3")
  - Monto
  - Método de Pago
  - Comprobante
- Barra de búsqueda
- Filtros: Método de Pago, Rango de fechas, Cliente
- Tarjeta resumen: Total Pagos, Monto Total, Distribución por método
- Botón "Nuevo Pago"
- Paginación

**Acciones por fila:**
- Ver detalle (icono ojo)
- Editar (icono lápiz)
- Eliminar (icono trash) - con confirmación

### 4.2 Formulario: Nuevo Pago

**Contexto 1: Desde Vista de Pagos**
```
┌─────────────────────────────────────┐
│  [Registrar Nuevo Pago]             │
├─────────────────────────────────────┤
│  Venta: [Buscar venta...] * 🔍      │
│  ┌─────────────────────────────┐   │
│  │ V-2024-001 - Juan Pérez     │   │
│  │ Parrilla Familiar           │   │
│  │ Saldo: S/ 400.00            │   │
│  └─────────────────────────────┘   │
│                                     │
│  Cliente: Juan Pérez García         │
│  Monto Total: S/ 600.00             │
│  Saldo Pendiente: S/ 400.00         │
│                                     │
│  ───────────────────────────────    │
│                                     │
│  Fecha Pago: [__/__/____] *         │
│  Número de Cuota: [___] *           │
│  Monto: S/ [_______] *              │
│  Sugerido: S/ 200.00                │
│                                     │
│  Método de Pago: *                  │
│  [ Transferencia ▼ ]                │
│                                     │
│  Comprobante: [________________]    │
│  Observaciones:                     │
│  [_________________________________]│
│                                     │
│  [Cancelar]  [Registrar Pago]      │
└─────────────────────────────────────┘
```

**Contexto 2: Desde Vista Detalle de Venta**
- Mismo formulario pero con `venta_id` pre-seleccionado
- Cliente y montos ya visibles
- Enfoque directo en datos del pago

**Funcionalidad Dinámica:**
- Al seleccionar venta:
  - Mostrar automáticamente nombre de cliente
  - Mostrar monto total y saldo pendiente
  - Sugerir monto de cuota
  - Pre-rellenar número de cuota siguiente
- Validación en tiempo real:
  - Monto no puede exceder saldo pendiente
  - Fecha no puede ser futura
- Mostrar advertencia si es el último pago que completa la venta

### 4.3 Vista Detalle: Información del Pago

```
┌─────────────────────────────────────────────────┐
│  PAGO P-2024-001                    [Editar]    │
│  Fecha: 24/11/2024                  [Eliminar]  │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│  INFORMACIÓN DEL PAGO                           │
├─────────────────────────────────────────────────┤
│  Venta:          V-2024-001 [Ver Venta]         │
│  Cliente:        Juan Pérez García              │
│  Producto:       Parrilla Familiar              │
│                                                 │
│  Fecha Pago:     24/11/2024                     │
│  Cuota:          1 de 3                         │
│  Monto Pagado:   S/ 200.00                      │
│  Método:         Transferencia                  │
│  Comprobante:    OP-123456789                   │
│  Observación:    Primera cuota pagada a tiempo  │
└─────────────────────────────────────────────────┘

┌─────────────────────────────────────────────────┐
│  ESTADO DE LA VENTA DESPUÉS DEL PAGO            │
├─────────────────────────────────────────────────┤
│  Monto Total:        S/ 600.00                  │
│  Total Pagado:       S/ 200.00                  │
│  Saldo Pendiente:    S/ 400.00                  │
│  Estado:             PENDIENTE 🟡               │
│                                                 │
│  [====--------]  33% completado                 │
└─────────────────────────────────────────────────┘
```

---

## 5. API Endpoints

### 5.1 REST API (Next.js API Routes)

```typescript
// GET /api/pagos
// Lista todos los pagos con filtros
GET /api/pagos?page=1&limit=50&venta_id=xxx&metodo_pago=transferencia

// GET /api/pagos/[id]
// Obtiene un pago específico con datos relacionados
GET /api/pagos/456e7890-e89b-12d3-a456-426614174333

// GET /api/pagos/venta/[venta_id]
// Obtiene todos los pagos de una venta
GET /api/pagos/venta/345e6789-e89b-12d3-a456-426614174222

// GET /api/pagos/stats
// Obtiene estadísticas de pagos por período
GET /api/pagos/stats?fecha_desde=2024-11-01&fecha_hasta=2024-11-30

// POST /api/pagos
// Crea un nuevo pago
POST /api/pagos
Body: { venta_id, fecha_pago, monto, metodo_pago, ... }

// PUT /api/pagos/[id]
// Actualiza un pago existente
PUT /api/pagos/456e7890-e89b-12d3-a456-426614174333
Body: { monto, metodo_pago, ... }

// DELETE /api/pagos/[id]
// Elimina un pago
DELETE /api/pagos/456e7890-e89b-12d3-a456-426614174333
```

### 5.2 Server Actions

```typescript
// app/actions/pagos.ts
export async function createPago(data: CreatePagoInput)
export async function updatePago(id: string, data: UpdatePagoInput)
export async function deletePago(id: string)
export async function getPagos(filters: ListPagosInput)
export async function getPagoById(id: string)
export async function getPagosByVenta(ventaId: string)
export async function getPaymentStats(fechaDesde: string, fechaHasta: string)
```

---

## 6. Reglas de Negocio

### 6.1 Generación de Pago ID
- Formato: `P-{YYYY}-{NNN}`
- Ejemplo: `P-2024-001`, `P-2024-002`
- Se reinicia numeración cada año

```typescript
async function generatePagoId(): Promise<string> {
  const year = new Date().getFullYear();
  const prefix = `P-${year}-`;

  const lastPago = await db
    .select()
    .from(pagos)
    .where(like(pago_id, `${prefix}%`))
    .orderBy(desc(pago_id))
    .limit(1);

  const lastNumber = lastPago?.[0]
    ? parseInt(lastPago[0].pago_id.split('-')[2])
    : 0;

  const newNumber = (lastNumber + 1).toString().padStart(3, '0');
  return `${prefix}${newNumber}`;
}
```

### 6.2 Actualización Automática de Venta

Cuando se registra, actualiza o elimina un pago, se deben actualizar automáticamente los siguientes campos en la tabla `ventas`:

**Trigger en Supabase:**
```sql
CREATE OR REPLACE FUNCTION update_venta_after_pago()
RETURNS TRIGGER AS $$
DECLARE
  v_venta_id UUID;
BEGIN
  -- Determinar venta_id según el tipo de operación
  IF TG_OP = 'DELETE' THEN
    v_venta_id := OLD.venta_id;
  ELSE
    v_venta_id := NEW.venta_id;
  END IF;

  -- Actualizar monto_pagado en la venta
  UPDATE ventas
  SET monto_pagado = (
    SELECT COALESCE(SUM(monto), 0)
    FROM pagos
    WHERE venta_id = v_venta_id
  ),
  updated_at = NOW()
  WHERE id = v_venta_id;

  -- Las columnas computadas saldo_pendiente y estado
  -- se actualizan automáticamente

  RETURN COALESCE(NEW, OLD);
END;
$$ LANGUAGE plpgsql;

-- Trigger para INSERT, UPDATE, DELETE
CREATE TRIGGER trigger_update_venta_after_pago
AFTER INSERT OR UPDATE OR DELETE ON pagos
FOR EACH ROW
EXECUTE FUNCTION update_venta_after_pago();
```

### 6.3 Validación de Monto

El monto de un pago no puede exceder el saldo pendiente de la venta:

```typescript
async function validateMontoNot ExceedsSaldo(
  ventaId: string,
  nuevoMonto: number,
  pagoIdExistente?: string
): Promise<boolean> {
  const venta = await getVentaById(ventaId);

  // Calcular saldo disponible
  const pagosExistentes = await getPagosByVenta(ventaId);
  const totalPagado = pagosExistentes
    .filter(p => p.id !== pagoIdExistente) // Excluir pago actual si es update
    .reduce((sum, p) => sum + p.monto, 0);

  const saldoDisponible = venta.monto_total - totalPagado;

  if (nuevoMonto > saldoDisponible) {
    throw new Error(
      `El monto del pago (S/ ${nuevoMonto}) excede el saldo pendiente (S/ ${saldoDisponible})`
    );
  }

  return true;
}
```

### 6.4 Número de Cuota

**Ventas al Contado:**
- `num_cuota = 0`
- Solo se permite un pago por el monto total

**Ventas en Cuotas:**
- `num_cuota >= 1` y `<= num_cuotas`
- Se puede registrar en cualquier orden (no necesariamente secuencial)
- Puede haber múltiples pagos para una misma cuota (pagos parciales)

**Sugerencia de Cuota (MVP):**
```typescript
async function sugerirNumeroCuota(ventaId: string): Promise<number> {
  const venta = await getVentaById(ventaId);
  const pagos = await getPagosByVenta(ventaId);

  if (venta.tipo_pago === 'contado') {
    return 0;
  }

  // Obtener última cuota registrada
  const ultimaCuota = Math.max(...pagos.map(p => p.num_cuota), 0);

  // Sugerir siguiente cuota (si no excede total)
  return Math.min(ultimaCuota + 1, venta.num_cuotas);
}
```

### 6.5 Métodos de Pago

Los métodos de pago disponibles son:
- **Efectivo**: Pago en efectivo
- **Transferencia**: Transferencia bancaria
- **Yape**: Aplicación de pago móvil (Perú)
- **Plin**: Aplicación de pago móvil (Perú)
- **Tarjeta Crédito**: Pago con tarjeta de crédito
- **Tarjeta Débito**: Pago con tarjeta de débito
- **Otro**: Otros métodos no listados

---

## 7. Casos de Uso

### 7.1 UC-PAG-01: Registrar Primer Pago de Venta en Cuotas

**Actor:** Usuario

**Precondiciones:**
- Usuario autenticado
- Existe venta en estado PENDIENTE

**Flujo Principal:**
1. Usuario navega a vista detalle de venta
2. Click en botón "Registrar Pago"
3. Sistema muestra formulario con venta pre-seleccionada
4. Sistema muestra:
   - Cliente: Juan Pérez García
   - Monto Total: S/ 600.00
   - Saldo Pendiente: S/ 600.00
   - Cuota sugerida: 1
   - Monto sugerido: S/ 200.00
5. Usuario confirma fecha de pago (hoy por default)
6. Usuario confirma número de cuota: 1
7. Usuario confirma monto: S/ 200.00
8. Usuario selecciona método de pago: Transferencia
9. Usuario ingresa comprobante: OP-123456789
10. Usuario agrega observación (opcional)
11. Click en "Registrar Pago"
12. Sistema valida datos
13. Sistema verifica que monto no exceda saldo
14. Sistema genera `pago_id`
15. Sistema registra pago
16. **Sistema actualiza automáticamente en venta:**
    - `monto_pagado = 200.00`
    - `saldo_pendiente = 400.00`
    - `estado = 'PENDIENTE'` (aún hay saldo)
17. Sistema muestra mensaje: "Pago registrado. Saldo pendiente: S/ 400.00"
18. Sistema redirige a vista detalle de venta actualizada

**Postcondición:**
- Pago registrado en sistema
- Venta actualizada en tiempo real
- Usuario puede ver progreso en barra visual

### 7.2 UC-PAG-02: Registrar Último Pago que Completa Venta

**Actor:** Usuario

**Flujo Principal:**
1-10. (Igual que UC-PAG-01, pero es la tercera cuota)
11. Usuario ingresa monto: S/ 200.00
12. Sistema detecta que es el último pago
13. Sistema muestra advertencia: "Este pago completará la venta"
14. Usuario confirma
15. Click en "Registrar Pago"
16. Sistema registra pago
17. **Sistema actualiza automáticamente en venta:**
    - `monto_pagado = 600.00`
    - `saldo_pendiente = 0.00`
    - `estado = 'PAGADO'` ✅
18. Sistema muestra mensaje: "¡Pago completado! La venta ha sido pagada en su totalidad"
19. Sistema muestra confetti/celebración (opcional)
20. Sistema redirige a vista detalle de venta

**Postcondición:**
- Venta cambia a estado PAGADO
- Ya no aparece en lista de ventas pendientes
- Se registra en métricas de ventas completadas

### 7.3 UC-PAG-03: Registrar Pago Parcial

**Actor:** Usuario

**Escenario:** Cliente paga menos de la cuota completa

**Flujo Principal:**
1-6. (Igual que UC-PAG-01)
7. Usuario ingresa monto: S/ 100.00 (solo la mitad)
8. Sistema acepta monto (es menor al saldo pendiente)
9. Usuario selecciona método y confirma
10. Sistema registra pago
11. **Sistema actualiza en venta:**
    - `monto_pagado = 100.00`
    - `saldo_pendiente = 500.00`
    - `estado = 'PENDIENTE'`
12. Usuario puede registrar otro pago posteriormente por los S/ 100.00 restantes

**Postcondición:**
- Sistema permite múltiples pagos para completar una cuota
- Flexibilidad para pagos parciales

### 7.4 UC-PAG-04: Eliminar Pago por Error

**Actor:** Usuario

**Flujo Principal:**
1. Usuario navega a lista de pagos o vista detalle de venta
2. Usuario identifica pago incorrecto
3. Click en "Eliminar" en pago específico
4. Sistema muestra confirmación:
   "¿Eliminar pago P-2024-001 de S/ 200.00?"
   "Esta acción actualizará el estado de la venta."
5. Usuario confirma
6. Sistema elimina pago
7. **Sistema actualiza automáticamente en venta:**
   - `monto_pagado` disminuye en S/ 200.00
   - `saldo_pendiente` aumenta en S/ 200.00
   - `estado` puede cambiar de PAGADO a PENDIENTE si corresponde
8. Sistema muestra mensaje: "Pago eliminado. Saldo actualizado."

**Postcondición:**
- Pago eliminado del sistema
- Venta actualizada correctamente
- Historial mantiene integridad

---

## 8. Validaciones y Errores

### 8.1 Validaciones Frontend (Zod Schema)

```typescript
import { z } from 'zod';

export const PagoSchema = z.object({
  venta_id: z.string()
    .uuid('Venta inválida'),

  fecha_pago: z.string()
    .regex(/^\d{4}-\d{2}-\d{2}$/, 'Formato de fecha inválido')
    .refine(date => new Date(date) <= new Date(), 'Fecha no puede ser futura'),

  num_cuota: z.number()
    .int('Número de cuota debe ser entero')
    .min(0, 'Número de cuota inválido'),

  monto: z.number()
    .positive('Monto debe ser mayor a 0')
    .multipleOf(0.01, 'Monto debe tener máximo 2 decimales'),

  metodo_pago: z.enum([
    'efectivo',
    'transferencia',
    'yape',
    'plin',
    'tarjeta_credito',
    'tarjeta_debito',
    'otro'
  ]),

  comprobante: z.string()
    .max(100, 'Comprobante demasiado largo')
    .optional(),

  observacion: z.string()
    .max(1000, 'Observación demasiado larga')
    .optional(),
});
```

### 8.2 Mensajes de Error

| Código | Mensaje | Acción |
|--------|---------|--------|
| `PAG_001` | Venta es obligatoria | Seleccionar venta |
| `PAG_002` | Fecha de pago es obligatoria | Ingresar fecha |
| `PAG_003` | Monto es obligatorio | Ingresar monto |
| `PAG_004` | Método de pago es obligatorio | Seleccionar método |
| `PAG_005` | Monto excede saldo pendiente | Reducir monto o verificar venta |
| `PAG_006` | Fecha no puede ser futura | Corregir fecha |
| `PAG_007` | Venta ya está completamente pagada | Verificar venta |
| `PAG_008` | Número de cuota inválido | Corregir cuota |
| `PAG_009` | Venta no encontrada | Verificar venta |
| `PAG_010` | Error al actualizar venta | Contactar soporte |

---

## 9. Testing

### 9.1 Tests Unitarios

```typescript
describe('Módulo Pagos', () => {
  describe('createPago', () => {
    it('debería crear pago y actualizar venta', async () => {
      const venta = await createVenta({
        monto_total: 600,
        tipo_pago: 'cuotas',
        num_cuotas: 3,
        /* ... */
      });

      const pago = await createPago({
        venta_id: venta.id,
        fecha_pago: '2024-11-24',
        num_cuota: 1,
        monto: 200,
        metodo_pago: 'transferencia',
      });

      expect(pago.success).toBe(true);

      const ventaActualizada = await getVentaById(venta.id);
      expect(ventaActualizada.monto_pagado).toBe(200);
      expect(ventaActualizada.saldo_pendiente).toBe(400);
      expect(ventaActualizada.estado).toBe('PENDIENTE');
    });

    it('debería cambiar estado a PAGADO al completar', async () => {
      const venta = await createVenta({ monto_total: 200, /* ... */ });

      await createPago({
        venta_id: venta.id,
        monto: 200,
        /* ... */
      });

      const ventaActualizada = await getVentaById(venta.id);
      expect(ventaActualizada.estado).toBe('PAGADO');
    });

    it('debería rechazar pago que excede saldo', async () => {
      const venta = await createVenta({ monto_total: 100, /* ... */ });

      await expect(
        createPago({
          venta_id: venta.id,
          monto: 150, // Excede monto total
          /* ... */
        })
      ).rejects.toThrow('excede el saldo pendiente');
    });
  });

  describe('deletePago', () => {
    it('debería actualizar venta al eliminar pago', async () => {
      const venta = await createVenta({ monto_total: 300, /* ... */ });
      const pago = await createPago({
        venta_id: venta.id,
        monto: 300,
        /* ... */
      });

      // Venta está PAGADA
      let ventaActualizada = await getVentaById(venta.id);
      expect(ventaActualizada.estado).toBe('PAGADO');

      // Eliminar pago
      await deletePago(pago.data.id);

      // Venta vuelve a PENDIENTE
      ventaActualizada = await getVentaById(venta.id);
      expect(ventaActualizada.estado).toBe('PENDIENTE');
      expect(ventaActualizada.monto_pagado).toBe(0);
    });
  });
});
```

### 9.2 Tests de Integración

- Crear pago y verificar trigger de actualización de venta
- Múltiples pagos para una misma venta
- Validación de constraint (monto no excede saldo)
- Eliminar pago y verificar recálculo
- Actualizar monto de pago y verificar impacto en venta

### 9.3 Tests E2E

- Flujo completo: crear venta → registrar pagos → completar venta
- Registrar pago desde vista de venta
- Registrar pago desde lista de pagos
- Eliminar pago y ver actualización en tiempo real
- Intentar exceder saldo (debe fallar)

---

## 10. Performance

### 10.1 Optimizaciones
- Índice en `venta_id` para joins rápidos
- Trigger optimizado para actualización de venta
- Cache de métodos de pago (static)
- Pagination en lista de pagos

### 10.2 Métricas
- Tiempo de registro de pago: < 1s
- Tiempo de actualización de venta: < 500ms (trigger)
- Tiempo de carga lista de pagos: < 700ms

---

## 11. Seguridad

### 11.1 Row Level Security (RLS)

```sql
-- Los usuarios solo ven sus propios pagos
CREATE POLICY "Users can view own pagos"
  ON pagos FOR SELECT
  USING (auth.uid() = user_id);

-- Los usuarios solo pueden crear pagos propios
CREATE POLICY "Users can insert own pagos"
  ON pagos FOR INSERT
  WITH CHECK (
    auth.uid() = user_id
    AND EXISTS (
      SELECT 1 FROM ventas
      WHERE id = venta_id AND user_id = auth.uid()
    )
  );

-- Los usuarios solo pueden actualizar sus propios pagos
CREATE POLICY "Users can update own pagos"
  ON pagos FOR UPDATE
  USING (auth.uid() = user_id);

-- Los usuarios solo pueden eliminar sus propios pagos
CREATE POLICY "Users can delete own pagos"
  ON pagos FOR DELETE
  USING (auth.uid() = user_id);
```

---

## 12. Features Futuras (Post-MVP)

### 12.1 Recordatorios de Pago
- Notificaciones automáticas antes de vencimiento
- Email/SMS a clientes
- Calendario de pagos próximos

### 12.2 Recibos de Pago
- Generación automática de PDF
- Envío por email al cliente
- Almacenamiento de comprobantes

### 12.3 Pagos Online
- Integración con pasarelas de pago
- Links de pago para clientes
- Pagos con tarjeta

### 12.4 Análisis de Cobranza
- Tiempo promedio de cobro
- Eficiencia por método de pago
- Predicción de pagos futuros

---

**Documento preparado por:** SGV BRASA Team
**Última actualización:** 24/11/2025
