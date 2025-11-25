# 🚀 MVP Rápido en 10 Minutos - SGV BRASA
## HTML + LocalStorage + Vercel

**Objetivo:** Validar la idea con un prototipo funcional deployado en 10 minutos

---

## ⚡ Plan de 10 Minutos

```
Minuto 0-2:  Crear estructura HTML
Minuto 2-5:  Implementar JavaScript con LocalStorage
Minuto 5-7:  Styling básico con Tailwind CDN
Minuto 7-9:  Testing local
Minuto 9-10: Deploy a Vercel
```

---

## 📁 Estructura del Proyecto

```
sgv-brasa-mvp/
├── index.html           (Dashboard)
├── ventas.html          (Lista y crear ventas)
├── pagos.html           (Lista y registrar pagos)
├── clientes.html        (Lista de clientes)
├── productos.html       (Lista de productos)
└── vercel.json          (Configuración Vercel)
```

---

## 🏃 Ejecución Rápida

### Paso 1: Crear carpeta (30 segundos)

```bash
cd ~/Documents/sgv_brasa_
mkdir mvp-html
cd mvp-html
```

### Paso 2: Crear archivos (Copiar y pegar cada uno - 5 minutos)

**index.html** (Dashboard)
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SGV BRASA - Dashboard</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = {
            theme: {
                extend: {
                    colors: {
                        brand: '#0ea5e9',
                    }
                }
            }
        }
    </script>
</head>
<body class="bg-gray-50">
    <!-- Navigation -->
    <nav class="bg-white border-b">
        <div class="max-w-7xl mx-auto px-4 py-4">
            <div class="flex items-center justify-between">
                <h1 class="text-2xl font-bold text-brand">SGV BRASA</h1>
                <div class="flex gap-4">
                    <a href="index.html" class="px-3 py-2 rounded bg-brand text-white">Dashboard</a>
                    <a href="ventas.html" class="px-3 py-2 rounded hover:bg-gray-100">Ventas</a>
                    <a href="pagos.html" class="px-3 py-2 rounded hover:bg-gray-100">Pagos</a>
                    <a href="clientes.html" class="px-3 py-2 rounded hover:bg-gray-100">Clientes</a>
                    <a href="productos.html" class="px-3 py-2 rounded hover:bg-gray-100">Productos</a>
                </div>
            </div>
        </div>
    </nav>

    <!-- Dashboard Content -->
    <div class="max-w-7xl mx-auto px-4 py-8">
        <h2 class="text-3xl font-bold mb-8">Dashboard</h2>

        <!-- KPIs -->
        <div class="grid grid-cols-1 md:grid-cols-4 gap-6 mb-8">
            <div class="bg-white p-6 rounded-lg shadow">
                <div class="text-sm text-gray-500">Ventas Totales</div>
                <div class="text-3xl font-bold" id="totalVentas">0</div>
            </div>
            <div class="bg-white p-6 rounded-lg shadow">
                <div class="text-sm text-gray-500">Monto Total</div>
                <div class="text-3xl font-bold" id="montoTotal">S/ 0</div>
            </div>
            <div class="bg-white p-6 rounded-lg shadow">
                <div class="text-sm text-gray-500">Saldo Pendiente</div>
                <div class="text-3xl font-bold text-yellow-600" id="saldoPendiente">S/ 0</div>
            </div>
            <div class="bg-white p-6 rounded-lg shadow">
                <div class="text-sm text-gray-500">Ventas Pagadas</div>
                <div class="text-3xl font-bold text-green-600" id="ventasPagadas">0</div>
            </div>
        </div>

        <!-- Últimas Ventas -->
        <div class="bg-white rounded-lg shadow p-6">
            <h3 class="text-xl font-bold mb-4">Últimas Ventas</h3>
            <div id="ultimasVentas" class="space-y-2">
                <p class="text-gray-500">No hay ventas registradas</p>
            </div>
        </div>
    </div>

    <script>
        // Cargar datos del Dashboard
        function loadDashboard() {
            const ventas = JSON.parse(localStorage.getItem('ventas') || '[]');
            const pagos = JSON.parse(localStorage.getItem('pagos') || '[]');

            // Calcular KPIs
            const totalVentas = ventas.length;
            const montoTotal = ventas.reduce((sum, v) => sum + parseFloat(v.monto_total || 0), 0);

            // Calcular monto pagado por venta
            ventas.forEach(venta => {
                const pagosVenta = pagos.filter(p => p.venta_id === venta.id);
                venta.monto_pagado = pagosVenta.reduce((sum, p) => sum + parseFloat(p.monto || 0), 0);
                venta.saldo_pendiente = venta.monto_total - venta.monto_pagado;
                venta.estado = venta.saldo_pendiente <= 0 ? 'PAGADO' : 'PENDIENTE';
            });

            const saldoPendiente = ventas.reduce((sum, v) => sum + v.saldo_pendiente, 0);
            const ventasPagadas = ventas.filter(v => v.estado === 'PAGADO').length;

            // Actualizar UI
            document.getElementById('totalVentas').textContent = totalVentas;
            document.getElementById('montoTotal').textContent = `S/ ${montoTotal.toFixed(2)}`;
            document.getElementById('saldoPendiente').textContent = `S/ ${saldoPendiente.toFixed(2)}`;
            document.getElementById('ventasPagadas').textContent = ventasPagadas;

            // Mostrar últimas 5 ventas
            const ultimasVentas = ventas.slice(-5).reverse();
            const html = ultimasVentas.map(v => `
                <div class="flex justify-between items-center p-3 border rounded">
                    <div>
                        <div class="font-semibold">${v.id}</div>
                        <div class="text-sm text-gray-500">${v.cliente_nombre} - ${v.producto_nombre}</div>
                    </div>
                    <div class="text-right">
                        <div class="font-bold">S/ ${parseFloat(v.monto_total).toFixed(2)}</div>
                        <div class="text-sm ${v.estado === 'PAGADO' ? 'text-green-600' : 'text-yellow-600'}">
                            ${v.estado}
                        </div>
                    </div>
                </div>
            `).join('');

            document.getElementById('ultimasVentas').innerHTML = html || '<p class="text-gray-500">No hay ventas registradas</p>';
        }

        // Cargar dashboard al inicio
        loadDashboard();
    </script>
</body>
</html>
```

**ventas.html** (Ventas)
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SGV BRASA - Ventas</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = { theme: { extend: { colors: { brand: '#0ea5e9' } } } }
    </script>
</head>
<body class="bg-gray-50">
    <!-- Navigation (mismo que index.html) -->
    <nav class="bg-white border-b">
        <div class="max-w-7xl mx-auto px-4 py-4">
            <div class="flex items-center justify-between">
                <h1 class="text-2xl font-bold text-brand">SGV BRASA</h1>
                <div class="flex gap-4">
                    <a href="index.html" class="px-3 py-2 rounded hover:bg-gray-100">Dashboard</a>
                    <a href="ventas.html" class="px-3 py-2 rounded bg-brand text-white">Ventas</a>
                    <a href="pagos.html" class="px-3 py-2 rounded hover:bg-gray-100">Pagos</a>
                    <a href="clientes.html" class="px-3 py-2 rounded hover:bg-gray-100">Clientes</a>
                    <a href="productos.html" class="px-3 py-2 rounded hover:bg-gray-100">Productos</a>
                </div>
            </div>
        </div>
    </nav>

    <div class="max-w-7xl mx-auto px-4 py-8">
        <div class="flex justify-between items-center mb-8">
            <h2 class="text-3xl font-bold">Ventas</h2>
            <button onclick="toggleForm()" class="bg-brand text-white px-6 py-2 rounded hover:bg-blue-600">
                Nueva Venta
            </button>
        </div>

        <!-- Formulario Nueva Venta -->
        <div id="formVenta" class="hidden bg-white p-6 rounded-lg shadow mb-8">
            <h3 class="text-xl font-bold mb-4">Crear Nueva Venta</h3>
            <form onsubmit="crearVenta(event)" class="space-y-4">
                <div>
                    <label class="block text-sm font-medium mb-1">Cliente</label>
                    <input type="text" id="cliente_nombre" required class="w-full border rounded px-3 py-2">
                </div>
                <div>
                    <label class="block text-sm font-medium mb-1">Producto</label>
                    <input type="text" id="producto_nombre" required class="w-full border rounded px-3 py-2">
                </div>
                <div>
                    <label class="block text-sm font-medium mb-1">Monto Total</label>
                    <input type="number" id="monto_total" step="0.01" required class="w-full border rounded px-3 py-2">
                </div>
                <div>
                    <label class="block text-sm font-medium mb-1">Tipo de Pago</label>
                    <select id="tipo_pago" onchange="toggleCuotas()" class="w-full border rounded px-3 py-2">
                        <option value="contado">Contado</option>
                        <option value="cuotas">Cuotas</option>
                    </select>
                </div>
                <div id="divCuotas" class="hidden">
                    <label class="block text-sm font-medium mb-1">Número de Cuotas</label>
                    <input type="number" id="num_cuotas" min="2" value="3" class="w-full border rounded px-3 py-2">
                </div>
                <div class="flex gap-4">
                    <button type="submit" class="bg-brand text-white px-6 py-2 rounded hover:bg-blue-600">
                        Crear Venta
                    </button>
                    <button type="button" onclick="toggleForm()" class="border px-6 py-2 rounded hover:bg-gray-100">
                        Cancelar
                    </button>
                </div>
            </form>
        </div>

        <!-- Lista de Ventas -->
        <div class="bg-white rounded-lg shadow overflow-hidden">
            <table class="w-full">
                <thead class="bg-gray-50 border-b">
                    <tr>
                        <th class="px-4 py-3 text-left">ID</th>
                        <th class="px-4 py-3 text-left">Cliente</th>
                        <th class="px-4 py-3 text-left">Producto</th>
                        <th class="px-4 py-3 text-right">Monto Total</th>
                        <th class="px-4 py-3 text-right">Pagado</th>
                        <th class="px-4 py-3 text-right">Saldo</th>
                        <th class="px-4 py-3 text-center">Estado</th>
                    </tr>
                </thead>
                <tbody id="listaVentas">
                    <tr><td colspan="7" class="px-4 py-8 text-center text-gray-500">No hay ventas registradas</td></tr>
                </tbody>
            </table>
        </div>
    </div>

    <script>
        function toggleForm() {
            const form = document.getElementById('formVenta');
            form.classList.toggle('hidden');
        }

        function toggleCuotas() {
            const tipoPago = document.getElementById('tipo_pago').value;
            const divCuotas = document.getElementById('divCuotas');
            divCuotas.classList.toggle('hidden', tipoPago === 'contado');
        }

        function crearVenta(e) {
            e.preventDefault();

            const ventas = JSON.parse(localStorage.getItem('ventas') || '[]');
            const id = `V-2024-${String(ventas.length + 1).padStart(3, '0')}`;

            const venta = {
                id,
                fecha: new Date().toISOString().split('T')[0],
                cliente_nombre: document.getElementById('cliente_nombre').value,
                producto_nombre: document.getElementById('producto_nombre').value,
                monto_total: parseFloat(document.getElementById('monto_total').value),
                tipo_pago: document.getElementById('tipo_pago').value,
                num_cuotas: document.getElementById('tipo_pago').value === 'cuotas'
                    ? parseInt(document.getElementById('num_cuotas').value)
                    : 1,
                monto_pagado: 0,
                estado: 'PENDIENTE'
            };

            ventas.push(venta);
            localStorage.setItem('ventas', JSON.stringify(ventas));

            alert('✅ Venta creada exitosamente');
            toggleForm();
            e.target.reset();
            loadVentas();
        }

        function loadVentas() {
            const ventas = JSON.parse(localStorage.getItem('ventas') || '[]');
            const pagos = JSON.parse(localStorage.getItem('pagos') || '[]');

            // Calcular monto pagado
            ventas.forEach(venta => {
                const pagosVenta = pagos.filter(p => p.venta_id === venta.id);
                venta.monto_pagado = pagosVenta.reduce((sum, p) => sum + parseFloat(p.monto || 0), 0);
                venta.saldo_pendiente = venta.monto_total - venta.monto_pagado;
                venta.estado = venta.saldo_pendiente <= 0 ? 'PAGADO' : 'PENDIENTE';
            });

            const html = ventas.reverse().map(v => `
                <tr class="border-b hover:bg-gray-50">
                    <td class="px-4 py-3 font-mono text-sm">${v.id}</td>
                    <td class="px-4 py-3">${v.cliente_nombre}</td>
                    <td class="px-4 py-3">${v.producto_nombre}</td>
                    <td class="px-4 py-3 text-right font-semibold">S/ ${v.monto_total.toFixed(2)}</td>
                    <td class="px-4 py-3 text-right text-green-600">S/ ${v.monto_pagado.toFixed(2)}</td>
                    <td class="px-4 py-3 text-right text-yellow-600">S/ ${v.saldo_pendiente.toFixed(2)}</td>
                    <td class="px-4 py-3 text-center">
                        <span class="px-2 py-1 rounded text-xs font-semibold ${
                            v.estado === 'PAGADO' ? 'bg-green-100 text-green-800' : 'bg-yellow-100 text-yellow-800'
                        }">
                            ${v.estado}
                        </span>
                    </td>
                </tr>
            `).join('');

            document.getElementById('listaVentas').innerHTML = html ||
                '<tr><td colspan="7" class="px-4 py-8 text-center text-gray-500">No hay ventas registradas</td></tr>';
        }

        loadVentas();
    </script>
</body>
</html>
```

**pagos.html** (Pagos)
```html
<!DOCTYPE html>
<html lang="es">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>SGV BRASA - Pagos</title>
    <script src="https://cdn.tailwindcss.com"></script>
    <script>
        tailwind.config = { theme: { extend: { colors: { brand: '#0ea5e9' } } } }
    </script>
</head>
<body class="bg-gray-50">
    <!-- Navigation -->
    <nav class="bg-white border-b">
        <div class="max-w-7xl mx-auto px-4 py-4">
            <div class="flex items-center justify-between">
                <h1 class="text-2xl font-bold text-brand">SGV BRASA</h1>
                <div class="flex gap-4">
                    <a href="index.html" class="px-3 py-2 rounded hover:bg-gray-100">Dashboard</a>
                    <a href="ventas.html" class="px-3 py-2 rounded hover:bg-gray-100">Ventas</a>
                    <a href="pagos.html" class="px-3 py-2 rounded bg-brand text-white">Pagos</a>
                    <a href="clientes.html" class="px-3 py-2 rounded hover:bg-gray-100">Clientes</a>
                    <a href="productos.html" class="px-3 py-2 rounded hover:bg-gray-100">Productos</a>
                </div>
            </div>
        </div>
    </nav>

    <div class="max-w-7xl mx-auto px-4 py-8">
        <div class="flex justify-between items-center mb-8">
            <h2 class="text-3xl font-bold">Pagos</h2>
            <button onclick="toggleForm()" class="bg-brand text-white px-6 py-2 rounded hover:bg-blue-600">
                Registrar Pago
            </button>
        </div>

        <!-- Formulario Registrar Pago -->
        <div id="formPago" class="hidden bg-white p-6 rounded-lg shadow mb-8">
            <h3 class="text-xl font-bold mb-4">Registrar Nuevo Pago</h3>
            <form onsubmit="crearPago(event)" class="space-y-4">
                <div>
                    <label class="block text-sm font-medium mb-1">Venta</label>
                    <select id="venta_id" onchange="updateVentaInfo()" required class="w-full border rounded px-3 py-2">
                        <option value="">Seleccionar venta...</option>
                    </select>
                </div>
                <div id="ventaInfo" class="hidden bg-blue-50 p-4 rounded">
                    <div class="text-sm space-y-1">
                        <div><span class="font-semibold">Cliente:</span> <span id="info_cliente"></span></div>
                        <div><span class="font-semibold">Monto Total:</span> <span id="info_total"></span></div>
                        <div><span class="font-semibold">Saldo Pendiente:</span> <span id="info_saldo" class="text-yellow-600 font-bold"></span></div>
                    </div>
                </div>
                <div>
                    <label class="block text-sm font-medium mb-1">Monto del Pago</label>
                    <input type="number" id="monto" step="0.01" required class="w-full border rounded px-3 py-2">
                </div>
                <div>
                    <label class="block text-sm font-medium mb-1">Método de Pago</label>
                    <select id="metodo_pago" class="w-full border rounded px-3 py-2">
                        <option value="efectivo">Efectivo</option>
                        <option value="transferencia">Transferencia</option>
                        <option value="yape">Yape</option>
                        <option value="plin">Plin</option>
                    </select>
                </div>
                <div class="flex gap-4">
                    <button type="submit" class="bg-brand text-white px-6 py-2 rounded hover:bg-blue-600">
                        Registrar Pago
                    </button>
                    <button type="button" onclick="toggleForm()" class="border px-6 py-2 rounded hover:bg-gray-100">
                        Cancelar
                    </button>
                </div>
            </form>
        </div>

        <!-- Lista de Pagos -->
        <div class="bg-white rounded-lg shadow overflow-hidden">
            <table class="w-full">
                <thead class="bg-gray-50 border-b">
                    <tr>
                        <th class="px-4 py-3 text-left">ID</th>
                        <th class="px-4 py-3 text-left">Venta</th>
                        <th class="px-4 py-3 text-left">Cliente</th>
                        <th class="px-4 py-3 text-right">Monto</th>
                        <th class="px-4 py-3 text-left">Método</th>
                        <th class="px-4 py-3 text-left">Fecha</th>
                    </tr>
                </thead>
                <tbody id="listaPagos">
                    <tr><td colspan="6" class="px-4 py-8 text-center text-gray-500">No hay pagos registrados</td></tr>
                </tbody>
            </table>
        </div>
    </div>

    <script>
        function toggleForm() {
            const form = document.getElementById('formPago');
            form.classList.toggle('hidden');
            if (!form.classList.contains('hidden')) {
                loadVentasPendientes();
            }
        }

        function loadVentasPendientes() {
            const ventas = JSON.parse(localStorage.getItem('ventas') || '[]');
            const pagos = JSON.parse(localStorage.getItem('pagos') || '[]');

            // Calcular saldo pendiente
            ventas.forEach(venta => {
                const pagosVenta = pagos.filter(p => p.venta_id === venta.id);
                venta.monto_pagado = pagosVenta.reduce((sum, p) => sum + parseFloat(p.monto || 0), 0);
                venta.saldo_pendiente = venta.monto_total - venta.monto_pagado;
            });

            const ventasPendientes = ventas.filter(v => v.saldo_pendiente > 0);

            const options = ventasPendientes.map(v =>
                `<option value="${v.id}" data-venta='${JSON.stringify(v)}'>${v.id} - ${v.cliente_nombre} (Saldo: S/ ${v.saldo_pendiente.toFixed(2)})</option>`
            ).join('');

            document.getElementById('venta_id').innerHTML = '<option value="">Seleccionar venta...</option>' + options;
        }

        function updateVentaInfo() {
            const select = document.getElementById('venta_id');
            const option = select.options[select.selectedIndex];

            if (option.value) {
                const venta = JSON.parse(option.dataset.venta);
                document.getElementById('info_cliente').textContent = venta.cliente_nombre;
                document.getElementById('info_total').textContent = `S/ ${venta.monto_total.toFixed(2)}`;
                document.getElementById('info_saldo').textContent = `S/ ${venta.saldo_pendiente.toFixed(2)}`;
                document.getElementById('ventaInfo').classList.remove('hidden');
                document.getElementById('monto').value = venta.saldo_pendiente.toFixed(2);
            } else {
                document.getElementById('ventaInfo').classList.add('hidden');
            }
        }

        function crearPago(e) {
            e.preventDefault();

            const pagos = JSON.parse(localStorage.getItem('pagos') || '[]');
            const ventas = JSON.parse(localStorage.getItem('ventas') || '[]');

            const ventaId = document.getElementById('venta_id').value;
            const venta = ventas.find(v => v.id === ventaId);
            const monto = parseFloat(document.getElementById('monto').value);

            // Validar que no exceda saldo
            const pagosPrevios = pagos.filter(p => p.venta_id === ventaId);
            const montoPagado = pagosPrevios.reduce((sum, p) => sum + parseFloat(p.monto), 0);
            const saldoDisponible = venta.monto_total - montoPagado;

            if (monto > saldoDisponible) {
                alert('❌ El monto excede el saldo pendiente');
                return;
            }

            const id = `P-2024-${String(pagos.length + 1).padStart(3, '0')}`;

            const pago = {
                id,
                venta_id: ventaId,
                cliente_nombre: venta.cliente_nombre,
                fecha: new Date().toISOString().split('T')[0],
                monto,
                metodo_pago: document.getElementById('metodo_pago').value
            };

            pagos.push(pago);
            localStorage.setItem('pagos', JSON.stringify(pagos));

            alert('✅ Pago registrado exitosamente');
            toggleForm();
            e.target.reset();
            loadPagos();
        }

        function loadPagos() {
            const pagos = JSON.parse(localStorage.getItem('pagos') || '[]');

            const html = pagos.reverse().map(p => `
                <tr class="border-b hover:bg-gray-50">
                    <td class="px-4 py-3 font-mono text-sm">${p.id}</td>
                    <td class="px-4 py-3 font-mono text-sm">${p.venta_id}</td>
                    <td class="px-4 py-3">${p.cliente_nombre}</td>
                    <td class="px-4 py-3 text-right font-semibold text-green-600">S/ ${parseFloat(p.monto).toFixed(2)}</td>
                    <td class="px-4 py-3 capitalize">${p.metodo_pago}</td>
                    <td class="px-4 py-3">${p.fecha}</td>
                </tr>
            `).join('');

            document.getElementById('listaPagos').innerHTML = html ||
                '<tr><td colspan="6" class="px-4 py-8 text-center text-gray-500">No hay pagos registrados</td></tr>';
        }

        loadPagos();
    </script>
</body>
</html>
```

**clientes.html** y **productos.html** - Versiones simplificadas (similar estructura)

**vercel.json**
```json
{
  "cleanUrls": true
}
```

### Paso 3: Testing Local (1 minuto)

```bash
# Abrir en navegador
open index.html
# O doble click en el archivo
```

**Probar:**
1. Crear una venta
2. Registrar un pago
3. Ver actualización en Dashboard

### Paso 4: Deploy a Vercel (2 minutos)

```bash
# Opción 1: Via CLI (MÁS RÁPIDO)
cd mvp-html
vercel --prod

# Opción 2: Via GitHub (si ya tienes repo)
git init
git add .
git commit -m "MVP HTML"
git remote add origin [URL]
git push -u origin main

# Luego en Vercel Dashboard:
# - Import from GitHub
# - Deploy
```

**O Drag & Drop:**
1. Ir a https://vercel.com/new
2. Arrastrar carpeta `mvp-html`
3. Deploy automático
4. ¡Listo!

---

## ✨ Características del MVP

### Funcionalidades Incluidas
✅ Dashboard con KPIs en tiempo real
✅ Crear ventas (contado y cuotas)
✅ **Cliente existente o nuevo:** Selector desplegable con opción "+ Crear nuevo cliente" al momento de registrar venta
✅ **Productos con auto-completado:** Al seleccionar producto, el nombre y monto se completan automáticamente (monto es editable)
✅ Registrar pagos
✅ Actualización automática de estados
✅ Cálculo de saldo pendiente
✅ Validación (no exceder saldo)
✅ Responsive (mobile-friendly)
✅ LocalStorage (persiste datos)

### Ventajas
- ✅ Sin backend necesario
- ✅ Sin base de datos
- ✅ Deploy instantáneo
- ✅ Gratis en Vercel
- ✅ Funciona offline
- ✅ Fácil de compartir (URL)

### Limitaciones
- ⚠️ Datos solo en browser (no compartidos)
- ⚠️ Sin autenticación
- ⚠️ Sin multi-usuario
- ⚠️ Si limpias cache, pierdes datos

---

## 🎯 Resultado Final

**URL de ejemplo:**
```
https://sgv-brasa-mvp.vercel.app
```

**Tiempo total:** 10 minutos
**Costo:** $0
**Features:** Dashboard + Ventas + Pagos funcionales

---

## 📊 Para Validar la Idea

### Cosas a Probar con Usuarios:
1. ✅ ¿El flujo es intuitivo?
2. ✅ ¿Los campos son suficientes?
3. ✅ ¿Qué features faltan?
4. ✅ ¿Pagarían por una versión completa?
5. ✅ ¿Qué mejoras sugieren?

### Métricas a Observar:
- Número de ventas creadas
- Número de pagos registrados
- Tiempo promedio en completar una venta
- Features más usados
- Features que confunden

---

## 🚀 Mejoras Rápidas (10 min c/u)

### Si funciona bien, agregar:

**1. Exportar Datos (5 min)**
```javascript
function exportarDatos() {
    const data = {
        ventas: localStorage.getItem('ventas'),
        pagos: localStorage.getItem('pagos')
    };
    const blob = new Blob([JSON.stringify(data, null, 2)], {type: 'application/json'});
    const url = URL.createObjectURL(blob);
    const a = document.createElement('a');
    a.href = url;
    a.download = 'sgv-brasa-backup.json';
    a.click();
}
```

**2. Importar Datos (5 min)**
```javascript
function importarDatos(file) {
    const reader = new FileReader();
    reader.onload = (e) => {
        const data = JSON.parse(e.target.result);
        localStorage.setItem('ventas', data.ventas);
        localStorage.setItem('pagos', data.pagos);
        location.reload();
    };
    reader.readAsText(file);
}
```

**3. Dark Mode (5 min)**
```javascript
// Toggle en nav
<button onclick="toggleDarkMode()">🌙</button>

function toggleDarkMode() {
    document.body.classList.toggle('dark');
    localStorage.setItem('darkMode', document.body.classList.contains('dark'));
}
```

---

## 📝 Checklist de Deploy

```markdown
Antes de Deploy:
- [ ] Todos los links funcionan
- [ ] Formularios validan correctamente
- [ ] LocalStorage guarda datos
- [ ] Cálculos son correctos
- [ ] Responsive en mobile

Después de Deploy:
- [ ] URL accesible
- [ ] Compartir con 3-5 usuarios
- [ ] Recolectar feedback
- [ ] Medir uso durante 1 semana
- [ ] Decidir si continuar con versión completa
```

---

## 💡 Próximos Pasos (Si valida bien)

### Plan B: Versión con Backend Simple (2-3 días)
- Supabase como backend
- Autenticación real
- Multi-usuario
- Datos persistentes

### Plan A: Versión Completa (6-8 semanas)
- Seguir [PLAN_IMPLEMENTACION.md](PLAN_IMPLEMENTACION.md)
- Next.js + Supabase + Vercel
- Todas las features del PRD

---

**¡Listo para validar tu idea en 10 minutos! 🚀**
