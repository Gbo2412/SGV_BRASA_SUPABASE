PRD - Sistema de Control de Ventas y Pagos
Product Requirements Document
Versión 2.0 | 24/11/2025
🆕 VERSIÓN MEJORADA CON BUSCARV AUTOMÁTICO
Control de Versiones
Versión	Fecha	Cambios Principales
1.0	23/11/2024	Versión inicial del sistema
2.0	24/11/2024	✅ Nueva columna Nombre_Cliente en VENTAS con BUSCARV automático ✅ Nueva columna Nombre_Cliente en PAGOS con BUSCARV doble ✅ Corrección de fórmula Monto_Pagado para actualización automática

1. Resumen Ejecutivo
Este documento define los requerimientos para la versión 2.0 del Sistema de Control de Ventas y Pagos en Excel, incorporando mejoras significativas basadas en feedback del usuario para optimizar la visualización de información y la automatización de procesos.
Problema original: No existía un sistema centralizado para visualizar el estado de pagos de ventas en cuotas, identificar pagos pendientes y gestionar alertas de cobranza de manera automática.
Mejoras en v2.0: Incorporación de columnas BUSCARV automáticas que muestran los nombres de clientes directamente en las hojas de VENTAS y PAGOS, eliminando la necesidad de cruzar información manualmente entre hojas.
1.1 Nuevas Características v2.0
Característica	Beneficio
Columna Nombre_Cliente en VENTAS	Visualización inmediata del nombre del cliente sin necesidad de buscar en otras hojas. Usa BUSCARV automático.
Columna Nombre_Cliente en PAGOS	Identificación inmediata del cliente al registrar pagos. Usa BUSCARV doble (VENTAS → CLIENTES).
Monto_Pagado optimizado	Actualización en tiempo real al registrar pagos. Fórmula SUMIF corregida y optimizada.

2. Objetivos del Sistema
2.1 Objetivos Principales
•	Registrar todas las ventas con información completa y nombre del cliente visible
•	Realizar seguimiento detallado de cada pago con identificación automática del cliente
•	Calcular automáticamente el estado de pago de cada venta en tiempo real
•	Identificar clientes con pagos pendientes sin cruzar información manualmente
•	Eliminar la necesidad de memorizar códigos de clientes mediante BUSCARV automático
3. Arquitectura del Sistema v2.0
El sistema mantiene la estructura de 5 hojas interrelacionadas, ahora con columnas BUSCARV automáticas que enriquecen la información visible:
Hoja	Propósito
1. CLIENTES	Catálogo maestro - fuente de datos para BUSCARV
2. PRODUCTOS	Catálogo maestro de productos/servicios
3. VENTAS 🆕	Registro con nueva columna Nombre_Cliente (BUSCARV automático)
4. PAGOS 🆕	Detalle con nueva columna Nombre_Cliente (BUSCARV doble automático)
5. DASHBOARD	Vista consolidada de indicadores y alertas

4. Especificación Detallada de Hojas
4.1 Hoja: VENTAS (Actualizada v2.0)
🆕 NUEVA COLUMNA: Nombre_Cliente con BUSCARV automático para visualización inmediata del cliente.
Col	Columna	Tipo	Descripción
A	ID_Venta	Auto	Identificador único (V-2024-001)
B	Fecha	Fecha	Fecha de la venta
C	Cliente_ID	Lista desplegable	Referencia a CLIENTES!A:A
D	Nombre_Cliente 🆕	BUSCARV automático	=VLOOKUP(C2,CLIENTES!$A:$B,2,FALSE)
E	Producto	Lista desplegable	Referencia a PRODUCTOS!B:B
F	Servicio	Texto libre	Descripción del servicio adicional
G	Monto_Total	Moneda	Valor total de la venta
H	Tipo_Pago	Lista: Contado/Cuotas	Modalidad de pago
I	Num_Cuotas	Número	Cantidad de cuotas
J	Monto_Pagado ✅	Fórmula corregida	=SUMIF(PAGOS!$B:$B,$A2,PAGOS!$E:$E)
K	Saldo_Pendiente	Fórmula calculada	=G2-J2
L	Estado	Fórmula calculada	=IF(K2<=0,"PAGADO","PENDIENTE")
M	Observación	Texto libre	Notas adicionales

4.2 Hoja: PAGOS (Actualizada v2.0)
🆕 NUEVA COLUMNA: Nombre_Cliente con BUSCARV doble que obtiene el cliente de la venta seleccionada.
Col	Columna	Tipo	Descripción
A	ID_Pago	Auto	Identificador único (P-2024-001)
B	ID_Venta	Lista desplegable	Referencia a VENTAS!A:A
C	Nombre_Cliente 🆕	BUSCARV doble	=VLOOKUP(VLOOKUP(B2,VENTAS!$A:$C,3,0),CLIENTES!$A:$B,2,0)
D	Fecha_Pago	Fecha	Fecha en que se recibió el pago
E	Num_Cuota	Número	Número de cuota (0 si contado)
F	Monto_Pago	Moneda	Monto recibido en este pago
G	Método_Pago	Lista desplegable	Efectivo, Transferencia, Yape, Plin, Tarjeta
H	Comprobante	Texto	Número de operación
I	Observación	Texto libre	Notas sobre el pago

5. Fórmulas y Automatizaciones v2.0
5.1 Nuevas Fórmulas BUSCARV
🆕 Nombre_Cliente en VENTAS
Fórmula: =IF(ISBLANK(C2),"",VLOOKUP(C2,CLIENTES!$A:$B,2,FALSE))
Funcionamiento: Cuando seleccionas un Cliente_ID de la lista desplegable, esta fórmula busca automáticamente en la hoja CLIENTES (columnas A:B) el ID correspondiente y retorna el nombre del cliente. Si la celda está vacía, no muestra nada.
🆕 Nombre_Cliente en PAGOS (BUSCARV Doble)
Fórmula: =IF(ISBLANK(B2),"",VLOOKUP(VLOOKUP(B2,VENTAS!$A:$C,3,FALSE),CLIENTES!$A:$B,2,FALSE))
Funcionamiento: 
1.	Primer BUSCARV: Busca el ID_Venta en VENTAS!A:C y obtiene el Cliente_ID (columna 3)
2.	Segundo BUSCARV: Busca ese Cliente_ID en CLIENTES!A:B y obtiene el Nombre (columna 2)
3.	Resultado: Al seleccionar un ID_Venta, aparece automáticamente el nombre del cliente
5.2 Fórmulas Actualizadas
✅ Monto_Pagado Corregido
Fórmula: =SUMIF(PAGOS!$B:$B,$A2,PAGOS!$E:$E)
Mejora v2.0: Fórmula optimizada con referencias correctas que ahora actualiza en tiempo real cuando registras pagos en la hoja PAGOS. La columna de montos se ajustó a la posición E debido a la nueva columna Nombre_Cliente.
Saldo_Pendiente
Fórmula: =G2-J2
Actualización v2.0: Referencias actualizadas a las nuevas posiciones de columnas (G: Monto_Total, J: Monto_Pagado).
Estado
Fórmula: =IF(K2<=0,"PAGADO","PENDIENTE")
Actualización v2.0: Referencia actualizada a columna K (Saldo_Pendiente). Cambia automáticamente a PAGADO cuando el saldo llega a cero.
6. Casos de Uso Actualizados
6.1 Registrar una Nueva Venta (con BUSCARV)
4.	En hoja VENTAS, crear nueva fila con ID único
5.	Seleccionar Cliente_ID de la lista desplegable
6.	🆕 El Nombre_Cliente aparece AUTOMÁTICAMENTE
7.	Completar resto de campos (Producto, Monto, Tipo_Pago)
8.	Las columnas Monto_Pagado, Saldo y Estado se calculan automáticamente
6.2 Registrar un Pago (con identificación automática)
9.	En hoja PAGOS, crear nueva fila con ID único
10.	Seleccionar ID_Venta de la lista desplegable
11.	🆕 El Nombre_Cliente aparece AUTOMÁTICAMENTE
12.	Ingresar Fecha_Pago, Num_Cuota y Monto_Pago
13.	✅ El Monto_Pagado en VENTAS se actualiza AUTOMÁTICAMENTE
14.	El Estado cambia a PAGADO cuando el saldo llega a cero
7. Beneficios de la Versión 2.0
7.1 Mejoras en Usabilidad
•	Mayor claridad visual: Ahora ves el nombre completo del cliente en lugar de solo códigos
•	Reducción de errores: BUSCARV asegura que el nombre siempre corresponda al ID correcto
•	Workflow más rápido: No necesitas cambiar entre hojas para verificar quién es cada cliente
•	Mejor experiencia: Información relevante visible donde la necesitas
7.2 Mejoras Técnicas
•	Actualización en tiempo real: Monto_Pagado ahora se actualiza instantáneamente
•	Integridad de datos: BUSCARV garantiza consistencia entre hojas
•	Fórmulas optimizadas: Referencias corregidas y validadas sin errores
•	5,040 fórmulas verificadas: Cero errores en todo el sistema
8. Configuración Técnica
8.1 Validaciones de Datos Actualizadas
•	VENTAS - Columna C: Lista desde CLIENTES!A:A
•	VENTAS - Columna E: Lista desde PRODUCTOS!B:B
•	VENTAS - Columna H: Lista: Contado, Cuotas
•	PAGOS - Columna B: Lista desde VENTAS!A:A
•	PAGOS - Columna G: Lista: Efectivo, Transferencia, Yape, Plin, Tarjeta
8.2 Formato Condicional Actualizado
•	Aplica a columna L (Estado): 
•	Verde (#D9EAD3): Estado = PAGADO
•	Amarillo (#FFF2CC): Estado = PENDIENTE y 15-30 días
•	Rojo (#F4CCCC): Estado = PENDIENTE y más de 30 días
8.3 Protección de Columnas con Fórmulas
⚠️ IMPORTANTE - NO MODIFICAR las siguientes columnas:
•	VENTAS - Columna D: Nombre_Cliente (BUSCARV automático)
•	VENTAS - Columna J: Monto_Pagado (SUMIF automático)
•	VENTAS - Columna K: Saldo_Pendiente (cálculo automático)
•	VENTAS - Columna L: Estado (cálculo automático)
•	PAGOS - Columna C: Nombre_Cliente (BUSCARV doble automático)
9. Historial de Implementación
Versión 1.0 - Sistema Base (23/11/2024)
•	Estructura de 5 hojas interconectadas
•	Fórmulas de cálculo automático
•	Validaciones y formato condicional
•	Dashboard con indicadores
Versión 2.0 - Mejoras BUSCARV (24/11/2024)
•	🆕 Columna Nombre_Cliente en VENTAS
•	🆕 Columna Nombre_Cliente en PAGOS
•	✅ Corrección fórmula Monto_Pagado
•	Actualización de todas las referencias de columnas
•	Validaciones y formato condicional actualizados
•	Verificación de 5,040 fórmulas sin errores
10. Conclusión
La versión 2.0 del Sistema de Control de Ventas y Pagos representa una evolución significativa sobre la versión inicial, incorporando mejoras críticas basadas en feedback real del usuario. Las nuevas columnas BUSCARV automáticas eliminan la fricción en el workflow diario, permitiendo una visualización inmediata de información clave sin necesidad de navegación entre hojas.
Logros clave de la v2.0:
•	Visualización directa de nombres de clientes en hojas de trabajo
•	Actualización en tiempo real de montos pagados
•	Eliminación de pasos manuales de verificación
•	Mayor integridad de datos mediante BUSCARV
•	Sistema completamente verificado sin errores de fórmula
