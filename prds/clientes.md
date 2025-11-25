# PRD - Módulo CLIENTES
**Product Requirements Document**

Versión 1.0 | 24/11/2025

---

## 1. Resumen del Módulo

### 1.1 Propósito
El módulo CLIENTES gestiona el catálogo maestro de clientes del sistema. Sirve como fuente de datos para todos los demás módulos y mantiene la información de contacto actualizada.

### 1.2 Relaciones
- **Origen para:** Módulo VENTAS (selección de cliente)
- **Impacto:** Los cambios en este módulo se reflejan automáticamente en VENTAS y PAGOS mediante relaciones de base de datos

---

## 2. Modelo de Datos

### 2.1 Tabla: `clientes`

| Campo | Tipo | Restricciones | Descripción |
|-------|------|---------------|-------------|
| `id` | UUID | PRIMARY KEY, AUTO | Identificador único del cliente |
| `cliente_id` | VARCHAR(20) | UNIQUE, NOT NULL | ID legible (ej: CLI-2024-001) |
| `nombre` | VARCHAR(255) | NOT NULL | Nombre completo o razón social |
| `email` | VARCHAR(255) | UNIQUE, NULL | Correo electrónico |
| `telefono` | VARCHAR(20) | NULL | Número de contacto |
| `direccion` | TEXT | NULL | Dirección completa |
| `documento` | VARCHAR(20) | NULL | DNI/RUC |
| `tipo` | ENUM | DEFAULT 'persona' | 'persona' o 'empresa' |
| `notas` | TEXT | NULL | Observaciones adicionales |
| `estado` | ENUM | DEFAULT 'activo' | 'activo' o 'inactivo' |
| `created_at` | TIMESTAMP | DEFAULT NOW() | Fecha de creación |
| `updated_at` | TIMESTAMP | DEFAULT NOW() | Fecha de última actualización |
| `user_id` | UUID | FOREIGN KEY | ID del usuario propietario |

### 2.2 Índices
```sql
CREATE INDEX idx_clientes_cliente_id ON clientes(cliente_id);
CREATE INDEX idx_clientes_nombre ON clientes(nombre);
CREATE INDEX idx_clientes_user_id ON clientes(user_id);
CREATE INDEX idx_clientes_estado ON clientes(estado);
```

### 2.3 Ejemplo de Registro
```json
{
  "id": "123e4567-e89b-12d3-a456-426614174000",
  "cliente_id": "CLI-2024-001",
  "nombre": "Juan Pérez García",
  "email": "juan.perez@email.com",
  "telefono": "+51 987654321",
  "direccion": "Av. Principal 123, Lima",
  "documento": "12345678",
  "tipo": "persona",
  "notas": "Cliente frecuente, prefiere pago en cuotas",
  "estado": "activo",
  "created_at": "2024-11-24T10:00:00Z",
  "updated_at": "2024-11-24T10:00:00Z",
  "user_id": "user-uuid-here"
}
```

---

## 3. Funcionalidades

### 3.1 Crear Cliente (CREATE)

**Entrada:**
```typescript
interface CreateClienteInput {
  nombre: string;              // Requerido
  email?: string;              // Opcional
  telefono?: string;           // Opcional
  direccion?: string;          // Opcional
  documento?: string;          // Opcional
  tipo?: 'persona' | 'empresa'; // Opcional, default: 'persona'
  notas?: string;              // Opcional
}
```

**Validaciones:**
- ✅ `nombre` es requerido y no puede estar vacío
- ✅ `email` debe tener formato válido si se proporciona
- ✅ `cliente_id` se genera automáticamente: `CLI-{año}-{número_secuencial}`
- ✅ No puede existir otro cliente con el mismo `email` (si se proporciona)
- ✅ `telefono` debe tener formato válido si se proporciona

**Proceso:**
1. Validar datos de entrada
2. Generar `cliente_id` único
3. Insertar en base de datos
4. Retornar cliente creado con todos los campos

**Respuesta:**
```typescript
interface ClienteCreated {
  success: boolean;
  data: Cliente;
  message: string;
}
```

### 3.2 Leer Clientes (READ)

**3.2.1 Listar Todos los Clientes**

**Entrada:**
```typescript
interface ListClientesInput {
  page?: number;        // Default: 1
  limit?: number;       // Default: 50
  search?: string;      // Búsqueda por nombre/email/documento
  tipo?: 'persona' | 'empresa' | 'all'; // Default: 'all'
  estado?: 'activo' | 'inactivo' | 'all'; // Default: 'activo'
  sortBy?: 'nombre' | 'created_at'; // Default: 'nombre'
  sortOrder?: 'asc' | 'desc'; // Default: 'asc'
}
```

**Respuesta:**
```typescript
interface ListClientesResponse {
  data: Cliente[];
  pagination: {
    page: number;
    limit: number;
    total: number;
    totalPages: number;
  };
}
```

**3.2.2 Obtener Cliente por ID**

**Entrada:**
```typescript
interface GetClienteInput {
  id: string; // UUID
}
```

**Respuesta:**
```typescript
interface GetClienteResponse {
  data: Cliente;
  stats?: {
    totalVentas: number;
    montoTotal: number;
    saldoPendiente: number;
    ultimaVenta: Date | null;
  };
}
```

### 3.3 Actualizar Cliente (UPDATE)

**Entrada:**
```typescript
interface UpdateClienteInput {
  id: string;                  // Requerido (UUID)
  nombre?: string;
  email?: string;
  telefono?: string;
  direccion?: string;
  documento?: string;
  tipo?: 'persona' | 'empresa';
  notas?: string;
  estado?: 'activo' | 'inactivo';
}
```

**Validaciones:**
- ✅ Cliente debe existir
- ✅ Si se actualiza `email`, no puede duplicar otro cliente
- ✅ Actualizar automáticamente `updated_at`

**Proceso:**
1. Validar que cliente existe
2. Validar nuevos datos
3. Actualizar campos modificados
4. Actualizar `updated_at`
5. Retornar cliente actualizado

### 3.4 Eliminar Cliente (DELETE)

**Entrada:**
```typescript
interface DeleteClienteInput {
  id: string; // UUID
}
```

**Validaciones:**
- ✅ Cliente debe existir
- ❌ NO permitir eliminación si tiene ventas asociadas
- ✅ Opción alternativa: marcar como `inactivo` en lugar de eliminar

**Proceso Recomendado (Soft Delete):**
1. Verificar si tiene ventas asociadas
2. Si tiene ventas: cambiar `estado` a 'inactivo'
3. Si NO tiene ventas: permitir eliminación física
4. Retornar confirmación

---

## 4. Interfaz de Usuario

### 4.1 Vista Principal: Lista de Clientes

**Componentes:**
- Tabla con columnas: Cliente ID, Nombre, Email, Teléfono, Tipo, Estado
- Barra de búsqueda (nombre, email, documento)
- Filtros: Tipo, Estado
- Botón "Nuevo Cliente"
- Paginación

**Acciones por fila:**
- Ver detalle (icono ojo)
- Editar (icono lápiz)
- Desactivar/Eliminar (icono trash)

### 4.2 Formulario: Nuevo/Editar Cliente

**Layout:**
```
┌─────────────────────────────────────┐
│  [Crear/Editar Cliente]             │
├─────────────────────────────────────┤
│  Nombre: [________________] *       │
│  Email:  [________________]         │
│  Teléfono: [____________]           │
│  Documento: [__________]            │
│  Tipo: (•) Persona  ( ) Empresa     │
│  Dirección: [____________________]  │
│  [_________________________________]│
│  Notas: [_________________________] │
│  [_________________________________]│
│                                     │
│  [Cancelar]  [Guardar Cliente]     │
└─────────────────────────────────────┘
```

**Validación en Tiempo Real:**
- Nombre: obligatorio
- Email: formato válido
- Teléfono: formato válido
- Feedback visual: verde (válido), rojo (error)

### 4.3 Vista Detalle: Información del Cliente

**Secciones:**

**Información Básica:**
- Cliente ID
- Nombre
- Email, Teléfono
- Documento, Tipo
- Dirección
- Notas
- Estado
- Botón "Editar"

**Estadísticas:**
- Total de Ventas
- Monto Total Vendido
- Saldo Pendiente
- Última Venta

**Historial de Ventas:**
- Tabla con últimas 10 ventas
- Columnas: ID Venta, Fecha, Producto, Monto, Estado
- Link "Ver todas las ventas"

---

## 5. API Endpoints

### 5.1 REST API (Next.js API Routes)

```typescript
// GET /api/clientes
// Lista todos los clientes con filtros y paginación
GET /api/clientes?page=1&limit=50&search=juan&tipo=persona&estado=activo

// GET /api/clientes/[id]
// Obtiene un cliente específico con stats
GET /api/clientes/123e4567-e89b-12d3-a456-426614174000

// POST /api/clientes
// Crea un nuevo cliente
POST /api/clientes
Body: { nombre, email, telefono, ... }

// PUT /api/clientes/[id]
// Actualiza un cliente existente
PUT /api/clientes/123e4567-e89b-12d3-a456-426614174000
Body: { nombre, email, ... }

// DELETE /api/clientes/[id]
// Elimina o desactiva un cliente
DELETE /api/clientes/123e4567-e89b-12d3-a456-426614174000
```

### 5.2 Server Actions (Alternativa Moderna)

```typescript
// app/actions/clientes.ts
export async function createCliente(data: CreateClienteInput)
export async function updateCliente(id: string, data: UpdateClienteInput)
export async function deleteCliente(id: string)
export async function getClientes(filters: ListClientesInput)
export async function getClienteById(id: string)
```

---

## 6. Reglas de Negocio

### 6.1 Generación de Cliente ID
- Formato: `CLI-{YYYY}-{NNN}`
- Ejemplo: `CLI-2024-001`, `CLI-2024-002`
- Se reinicia numeración cada año
- Se rellena con ceros a la izquierda (001, 002, ..., 100)

**Algoritmo:**
```typescript
async function generateClienteId(): Promise<string> {
  const year = new Date().getFullYear();
  const prefix = `CLI-${year}-`;

  // Obtener último número del año actual
  const lastCliente = await db
    .select()
    .from(clientes)
    .where(like(cliente_id, `${prefix}%`))
    .orderBy(desc(cliente_id))
    .limit(1);

  const lastNumber = lastCliente?.[0]
    ? parseInt(lastCliente[0].cliente_id.split('-')[2])
    : 0;

  const newNumber = (lastNumber + 1).toString().padStart(3, '0');
  return `${prefix}${newNumber}`;
}
```

### 6.2 Búsqueda de Clientes
- Búsqueda insensible a mayúsculas/minúsculas
- Buscar en: nombre, email, documento, cliente_id
- Resultados ordenados por relevancia

### 6.3 Clientes Inactivos
- Clientes inactivos NO aparecen en listas desplegables
- Clientes inactivos NO se pueden eliminar si tienen ventas
- Se pueden reactivar en cualquier momento

---

## 7. Casos de Uso

### 7.1 UC-CLI-01: Registrar Nuevo Cliente

**Actor:** Usuario

**Precondiciones:**
- Usuario autenticado

**Flujo Principal:**
1. Usuario navega a módulo CLIENTES
2. Click en botón "Nuevo Cliente"
3. Sistema muestra formulario vacío
4. Usuario ingresa nombre (obligatorio)
5. Usuario ingresa email, teléfono (opcional)
6. Usuario selecciona tipo (persona/empresa)
7. Usuario ingresa documento y dirección (opcional)
8. Usuario agrega notas (opcional)
9. Click en "Guardar Cliente"
10. Sistema valida datos
11. Sistema genera `cliente_id` automáticamente
12. Sistema guarda en base de datos
13. Sistema muestra mensaje de éxito
14. Sistema redirige a lista de clientes

**Flujo Alternativo 10a: Datos inválidos**
- Sistema muestra errores en campos correspondientes
- Usuario corrige errores
- Continúa en paso 9

**Flujo Alternativo 10b: Email duplicado**
- Sistema muestra error "Email ya registrado"
- Usuario corrige email
- Continúa en paso 9

### 7.2 UC-CLI-02: Buscar Cliente para Venta

**Actor:** Usuario (desde módulo VENTAS)

**Flujo Principal:**
1. Usuario está creando una venta
2. Usuario hace click en campo "Cliente"
3. Sistema muestra autocomplete con lista de clientes activos
4. Usuario escribe nombre o código
5. Sistema filtra resultados en tiempo real
6. Usuario selecciona cliente de la lista
7. Sistema completa campo con cliente seleccionado
8. Sistema muestra nombre completo del cliente

### 7.3 UC-CLI-03: Ver Estado de Cuenta de Cliente

**Actor:** Usuario

**Flujo Principal:**
1. Usuario navega a lista de clientes
2. Usuario hace click en cliente específico
3. Sistema muestra vista detalle con:
   - Información básica
   - Estadísticas (total ventas, saldo pendiente)
   - Historial de ventas
4. Usuario puede editar información
5. Usuario puede ver ventas individuales

---

## 8. Validaciones y Errores

### 8.1 Validaciones Frontend (Zod Schema)

```typescript
import { z } from 'zod';

export const ClienteSchema = z.object({
  nombre: z.string()
    .min(2, 'Nombre debe tener al menos 2 caracteres')
    .max(255, 'Nombre demasiado largo'),

  email: z.string()
    .email('Email inválido')
    .optional()
    .or(z.literal('')),

  telefono: z.string()
    .regex(/^[+]?[(]?[0-9]{1,4}[)]?[-\s.]?[(]?[0-9]{1,4}[)]?[-\s.]?[0-9]{1,9}$/,
      'Formato de teléfono inválido')
    .optional()
    .or(z.literal('')),

  documento: z.string()
    .max(20, 'Documento demasiado largo')
    .optional(),

  tipo: z.enum(['persona', 'empresa']).default('persona'),

  direccion: z.string().max(500, 'Dirección demasiado larga').optional(),

  notas: z.string().max(1000, 'Notas demasiado largas').optional(),
});
```

### 8.2 Mensajes de Error

| Código | Mensaje | Acción |
|--------|---------|--------|
| `CLI_001` | Nombre es obligatorio | Solicitar nombre |
| `CLI_002` | Email inválido | Corregir formato |
| `CLI_003` | Email ya registrado | Usar otro email |
| `CLI_004` | Cliente no encontrado | Verificar ID |
| `CLI_005` | No se puede eliminar: tiene ventas | Desactivar en su lugar |
| `CLI_006` | Teléfono inválido | Corregir formato |

---

## 9. Testing

### 9.1 Tests Unitarios

```typescript
describe('Módulo Clientes', () => {
  describe('createCliente', () => {
    it('debería crear cliente con datos válidos', async () => {
      const input = { nombre: 'Juan Pérez', email: 'juan@test.com' };
      const result = await createCliente(input);
      expect(result.success).toBe(true);
      expect(result.data.cliente_id).toMatch(/CLI-\d{4}-\d{3}/);
    });

    it('debería rechazar cliente sin nombre', async () => {
      const input = { email: 'test@test.com' };
      await expect(createCliente(input)).rejects.toThrow();
    });

    it('debería rechazar email duplicado', async () => {
      await createCliente({ nombre: 'User 1', email: 'same@test.com' });
      await expect(
        createCliente({ nombre: 'User 2', email: 'same@test.com' })
      ).rejects.toThrow();
    });
  });

  describe('generateClienteId', () => {
    it('debería generar ID con formato correcto', async () => {
      const id = await generateClienteId();
      expect(id).toMatch(/CLI-\d{4}-\d{3}/);
    });

    it('debería incrementar número secuencial', async () => {
      await createCliente({ nombre: 'Cliente 1' });
      await createCliente({ nombre: 'Cliente 2' });
      const clientes = await getClientes({});
      expect(clientes.data[1].cliente_id).toBe('CLI-2024-002');
    });
  });
});
```

### 9.2 Tests de Integración

- CRUD completo en base de datos
- Relaciones con módulo VENTAS
- Búsqueda y filtros
- Paginación

### 9.3 Tests E2E (Playwright)

- Flujo completo de creación de cliente
- Búsqueda y selección desde módulo VENTAS
- Edición de cliente existente
- Desactivación de cliente con ventas

---

## 10. Performance

### 10.1 Optimizaciones
- Índices en columnas de búsqueda frecuente
- Paginación en lista de clientes (50 por página)
- Debounce en búsqueda autocomplete (300ms)
- Cache de clientes activos (5 minutos)

### 10.2 Métricas
- Tiempo de carga lista: < 500ms
- Tiempo de búsqueda: < 200ms
- Tiempo de creación: < 1s

---

## 11. Seguridad

### 11.1 Row Level Security (RLS)

```sql
-- Política: Los usuarios solo ven sus propios clientes
CREATE POLICY "Users can view own clientes"
  ON clientes FOR SELECT
  USING (auth.uid() = user_id);

-- Política: Los usuarios solo pueden crear sus propios clientes
CREATE POLICY "Users can insert own clientes"
  ON clientes FOR INSERT
  WITH CHECK (auth.uid() = user_id);

-- Política: Los usuarios solo pueden actualizar sus propios clientes
CREATE POLICY "Users can update own clientes"
  ON clientes FOR UPDATE
  USING (auth.uid() = user_id);

-- Política: Los usuarios solo pueden eliminar sus propios clientes
CREATE POLICY "Users can delete own clientes"
  ON clientes FOR DELETE
  USING (auth.uid() = user_id);
```

---

## 12. Migraciones

### 12.1 Script de Importación desde Excel

```typescript
// scripts/import-clientes-from-excel.ts
import * as XLSX from 'xlsx';

interface ExcelCliente {
  'Cliente ID': string;
  'Nombre': string;
  'Email': string;
  'Teléfono': string;
}

async function importClientesFromExcel(filePath: string) {
  const workbook = XLSX.readFile(filePath);
  const sheet = workbook.Sheets['CLIENTES'];
  const data: ExcelCliente[] = XLSX.utils.sheet_to_json(sheet);

  for (const row of data) {
    await createCliente({
      nombre: row.Nombre,
      email: row.Email || undefined,
      telefono: row.Teléfono || undefined,
    });
  }

  console.log(`Imported ${data.length} clientes`);
}
```

---

**Documento preparado por:** SGV BRASA Team
**Última actualización:** 24/11/2025
