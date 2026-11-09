# Capítulo 10 — El Inicio y lo que ve un administrador frente a un cajero

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.1 |
| **Fecha** | 2026-11-04 |
| **Backend** | `sofia-apis`, rama `main`, commit `9c16f35` (más nuevo que la línea base `43f5529`; sin cambios locales al empezar) |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `7070e1a` (más nuevo que la línea base `eeffbb6`; sin cambios locales al empezar) |
| **Método** | Recorrido en la aplicación local más inventario de pantallas |

## Índice

1. Introducción
2. Antes de empezar
3. Leer el Inicio como administrador
4. Los botones de acceso directo
5. Ventas por sucursal
6. El Inicio como cajero
7. Cuándo se actualizan los indicadores
8. Administrador frente a cajero en toda la aplicación
9. Diferencias con la documentación técnica
10. Pendientes y «No verificado»
11. Tabla de cobertura
12. Fuentes

## 1. Introducción

En este capítulo aprendes a **leer el Inicio** de SOFIA: la primera pantalla que ves al entrar. Es un resumen de lo que pasa hoy en tu botica: ventas, caja, inventario y devoluciones. Vas a:

- Entender **cada tarjeta e indicador**: de dónde sale la cifra, qué ventas suma y qué periodo cubre.
- Usar los **botones de acceso directo** («Ver lotes», «Ver ventas de hoy», «Ver mis ventas», «Ver ventas en espera», «Ver cajas abiertas», «Abrir caja»).
- Saber cuándo aparece la tabla **«Ventas por sucursal»**.
- Ver **en qué cambia el Inicio** entre un administrador y un cajero.
- Tener a mano una **tabla de resumen** de lo que ve y puede hacer un administrador frente a un cajero en cada menú y pantalla de la aplicación.
- Saber **cuándo se actualizan** las cifras.

Es para el **administrador** y para el **cajero**. El [capítulo 1](capitulo-01-primeros-pasos.md) (sección 4.5) ya te mostró cómo reconocer la pantalla principal; aquí se explica cifra por cifra.

**Ideas que debes tener claras**

- **El Inicio no se actualiza solo.** Muestra las cifras del momento en que entraste o pulsaste **Actualizar**.
- **«Hoy» es el día de calendario de tu computador**, de las 00:00 a las 24:00 (hora de Perú si tu equipo está en hora de Perú). Las ventas de ayer salen en «Ayer», nunca en «Vendido hoy».
- **Los montos vendidos son brutos**: una devolución no los reduce; las devoluciones se muestran aparte.
- **Cada bloque depende de tus permisos.** Solo ves los bloques de los módulos que puedes leer.

## 2. Antes de empezar

- Haz los capítulos 1 a 9. Este capítulo usa los datos de la empresa de prueba: las ventas del 02/11 y del 03/11 (con 2 anulaciones y 2 devoluciones), los 7 lotes (uno vence pronto) y las cuentas `ana.manual` (administradora) y `carlos.cajero` (cajero).
- Los datos de esta empresa de prueba, antes de empezar el capítulo, eran:

| Qué | Valor |
|---|---|
| Fecha en que se recorrió | miércoles 04/11/2026 (por eso «hoy» es el 04/11 y «Ayer» es el 03/11) |
| Cajas abiertas | Ninguna |
| Ventas del 02/11 | `B001-00000001` S/ 45.00, `B001-00000002` S/ 4.00 (devuelta), `B001-00000003` S/ 14.40, `B001-00000004` S/ 5.00, `B001-00000006` S/ 11.00, `B001-00000007` S/ 3.80, `B001-00000008` S/ 12.00 (todas suman S/ 95.20); `B001-00000005` (S/ 8.90) quedó anulada |
| Ventas del 03/11 | `B001-00000009` S/ 0.40 |
| Ventas en espera | Ninguna (la `115BF400` se anuló) |
| Lotes | 7 lotes; `LOT-PARA-CERCA` (15 unidades en la Sede Principal) vence el 15/11/2026 |

- **Permisos.** El Inicio no pide un permiso para abrirse: cualquier persona con sesión iniciada lo ve. Lo que muestra depende de los permisos de lectura (el nombre es el de la pantalla de permisos de un rol: primero el módulo y luego la acción):

| Bloque del Inicio | Módulo | Acción |
|---|---|---|
| Ventas de hoy, Ventas de los últimos 7 días y Productos más vendidos | Ventas | `Leer` |
| Caja | Sesiones de Caja | `Leer` (y `Aperturar caja` para ver el botón **Abrir caja**) |
| Inventario (lotes, productos sin stock y con stock bajo) | Inventario | `Leer` |
| Inventario: «Lotes en cuarentena» | DIGEMID | `Leer` |
| Devoluciones de hoy | Devoluciones | `Leer` |

El administrador (rol «Admin») pasa siempre, sin casillas marcadas. El rol «Cajero» del [capítulo 3](capitulo-03-empleados-roles-cuentas.md) (sección 5.3) tiene los cinco `Leer` y por eso ve todos los bloques.

## 3. Leer el Inicio como administrador

**Cómo llegar:** al iniciar sesión llegas solo al Inicio. Después, menú lateral › grupo **INICIO** › **Dashboard**.

Esta sección se recorrió con la administradora `ana.manual` el 04/11/2026 a las 17:20, con la empresa tal como la dejó el capítulo 9 (sin ventas hoy, ninguna caja abierta). Al final se muestra la misma pantalla con una venta del día y una caja abierta.

### 3.1 Cómo es la pantalla

De arriba hacia abajo:

1. **Tarjeta de cabecera**: «Hola, Ana Demo Prueba» (tu nombre), «Este es el resumen de hoy.», «Actualizado a las 17:20» y el botón **Actualizar**.
2. **Ventas de hoy**: cuatro tarjetas pequeñas («Vendido hoy», «Comprobantes», «Mis ventas de hoy», «Ventas en espera») y debajo dos tarjetas grandes, «Cobrado por método de pago» y «Ventas por sucursal». Cierra la nota «Los montos vendidos son brutos: las devoluciones se muestran aparte.»
3. **Caja**.
4. **Inventario**: siete tarjetas.
5. **Ventas de los últimos 7 días** y **Productos más vendidos en 7 días**.
6. **Devoluciones de hoy**.

Mientras un bloque carga, ves en su lugar un círculo giratorio (con el texto accesible «Cargando»). Si no tienes permiso para algún bloque, ese bloque no aparece. Si no tienes ninguno, solo ves la cabecera con el aviso «Tu usuario no tiene indicadores disponibles aquí. Usa el menú lateral para ir a tus módulos.» (No verificado en la aplicación: habría que quitarle todos los permisos de lectura a un rol, y este capítulo no cambia permisos. Viene de `producto/pantallas/acceso-layout-y-compartidos` §3.1.)

### 3.2 Cabecera

| Elemento | Qué muestra |
|---|---|
| Saludo | «Hola, {tu nombre}». Si la cuenta no trae nombre, dice «Bienvenido a SOFIA» (No verificado en la aplicación: las cuentas de prueba tienen nombre) |
| Subtítulo | «Este es el resumen de hoy.» |
| Hora | «Actualizado a las {hora}» (formato `HH:mm`): la hora en que se pidieron las cifras por última vez |
| **Actualizar** | Vuelve a pedir todas las cifras y recalcula «hoy». Queda deshabilitado mientras carga |

### 3.3 Ventas de hoy

Una venta **cuenta** en estas cifras si es de las sucursales que ves y su estado **no es «Anulada» ni «Pendiente»**; es decir, cuentan las ventas «Completada» y «Devuelta» (`RN-DAS-005`; `sofia-apis/src/SOFIA.Application/Dashboard/Queries/GetDashboardVentas/GetDashboardVentasQueryHandler.cs:28`). El monto es siempre el **total bruto** de la venta. «Hoy» va de las 00:00 a las 24:00 del día de tu computador (`RN-DAS-004`).

| Indicador | Qué es | De dónde sale |
|---|---|---|
| **Vendido hoy** | Suma de los totales de las ventas que cuentan hoy. Debajo, la comparación con ayer: una flecha, el porcentaje «vs. ayer» y «Ayer: S/ …» | `RN-DAS-006`. Variación = (hoy − ayer) ÷ ayer × 100, con 1 decimal. Si ayer no vendiste nada, solo dice «Ayer: S/ 0.00» y no hay porcentaje |
| **Comprobantes** | Número de ventas que cuentan hoy (una venta, una boleta; las notas de crédito no se cuentan). Debajo, «Ticket promedio: S/ …» | `RN-DAS-007`. Ticket promedio = Vendido hoy ÷ número de ventas, con 2 decimales; 0 si no hubo ventas |
| **Mis ventas de hoy** | Lo mismo que «Vendido hoy», pero solo de las ventas hechas por **tu usuario** (tu empleado), y «{n} comprobantes» | `RN-DAS-008`. Solo aparece si tu cuenta está ligada a un empleado |
| **Ventas en espera** | Cuántas ventas están en espera y, debajo, «Por cobrar: S/ …» | `RN-DAS-009`. Cuenta **todas** las ventas en espera, de **cualquier día**: una venta en espera no caduca |
| **Cobrado por método de pago** | Cuatro filas, siempre en este orden: «Efectivo», «Yape / Plin», «Tarjeta», «Transferencia», con el monto y una barra proporcional. Nota: «El efectivo ya descuenta el vuelto entregado.» | `RN-DAS-010`. Suma los pagos de las ventas que cuentan hoy; el efectivo es lo cobrado menos el vuelto. No resta reembolsos de devoluciones |
| **Ventas por sucursal** | Tabla con «Sucursal», «Comprobantes» y «Vendido» | Solo con más de una sucursal visible. Ver la [sección 5](#5-ventas-por-sucursal) |

### 3.4 Caja

| Situación | Qué ves |
|---|---|
| Tú no tienes caja abierta | Un candado y «No tienes una caja abierta», con el botón **Abrir caja** (solo si tu rol tiene `Sesiones de Caja › Aperturar caja`) |
| Tú tienes caja abierta | «Tienes una caja abierta», con «Sucursal», «Abierta desde», «Monto de apertura», «Efectivo esperado» y el botón **Ver mi caja** (abre el detalle de tu sesión) |
| Hay cajas abiertas de otras personas | Además, «Cajas abiertas ({n})» con una tabla «Cajero», «Sucursal» y «Abierta desde», y el botón **Ver cajas abiertas** |

Tu caja (`RN-DAS-015`) es **tu** sesión abierta más reciente. «Efectivo esperado» se calcula igual que al cerrar la caja: monto de apertura, más el efectivo cobrado en esa caja (descontado el vuelto), menos los reembolsos en efectivo de devoluciones hechas desde esa caja, menos el efectivo de ventas de otras cajas que anulaste desde esta. Una venta anulada desde la misma caja que la cobró no cuenta. «Cajas abiertas» (`RN-DAS-016`) lista las sesiones abiertas de todas las sucursales que ves (máximo 50), incluida la tuya; solo aparece si hay alguna que no sea la tuya.

### 3.5 Inventario

Cuenta **filas de stock** (un lote en una sucursal), no lotes únicos: un mismo lote en dos sucursales cuenta dos veces (`RN-DAS-017`). Solo cuentan filas con unidades; «hoy» es el inicio del día de tu computador.

| Tarjeta | Qué cuenta | Regla |
|---|---|---|
| **Lotes vencidos con stock** | Lotes con unidades y fecha de caducidad **anterior a hoy**. Un lote que caduca hoy todavía no está vencido | `RN-DAS-018` |
| **Lotes que vencen en 30 días o menos** | Caducidad desde hoy hasta hoy + 30 días | `RN-DAS-019` |
| **Lotes que vencen en 31 a 60 días** | Caducidad de hoy + 31 a hoy + 60 días | `RN-DAS-019` |
| **Lotes que vencen en 61 a 90 días** | Caducidad de hoy + 61 a hoy + 90 días | `RN-DAS-019` |
| **Productos sin stock** («Sin unidades vendibles, por sucursal») | Pares sucursal + producto sin unidades vendibles: sin stock, o solo con lotes vencidos o retenidos | `RN-DAS-020` |
| **Productos con stock bajo** («10 unidades o menos, por sucursal») | Pares sucursal + producto con 1 a 10 unidades vendibles en total. El 10 es fijo, no se cambia | `RN-DAS-021` |
| **Lotes en cuarentena** («Retenidos: no se pueden vender») | Lotes en cuarentena con estado «Retenido». Pide el permiso `DIGEMID › Leer` | `RN-DAS-022` |

Los lotes por vencer y los vencidos incluyen también los lotes retenidos en cuarentena. «Productos sin stock» y «con stock bajo» no tienen botón.

### 3.6 Tendencia, más vendidos y devoluciones

- **Ventas de los últimos 7 días** (`RN-DAS-012`): siete barras, de la más antigua a hoy, con el monto sobre cada barra y la fecha `dd/MM` debajo; la última dice «Hoy». Las barras son proporcionales al día de mayor monto. Hoy aparece incompleto, porque el día aún no termina. Cada barra tiene un texto para lectores de pantalla: «{dia}: S/ {monto} en {comprobantes} comprobantes».
- **Productos más vendidos en 7 días** (`RN-DAS-013`): hasta 5 productos con más **unidades** vendidas en esos 7 días, con «{n} unidades · S/ {monto}» (las unidades son siempre unidades base, no cajas ni blísteres). Sin ventas dice «Aún no hay ventas en los últimos 7 días.»
- **Devoluciones de hoy** (`RN-DAS-023`): «Devoluciones» (cuántas se procesaron hoy, parciales y totales por igual) y «Monto reembolsado: S/ …» (de cualquier método de reembolso).

### 3.7 Cada indicador: valor visto frente a valor esperado

Esperado = lo que sale de las ventas, anulaciones y devoluciones que dejaron los capítulos 8 y 9. Visto = lo que mostró la aplicación el 04/11/2026 a las 17:20, antes de hacer ventas hoy.

| Indicador | Valor visto | Esperado y por qué | Coincide |
|---|---|---|---|
| Vendido hoy | S/ 0.00 | Hoy (04/11) todavía no hay ventas | Sí |
| Variación contra ayer | «-100.0% vs. ayer · Ayer: S/ 0.40», flecha hacia abajo | Ayer (03/11) solo se cobró `B001-00000009` por S/ 0.40; (0 − 0.40) ÷ 0.40 = −100.0 % | Sí |
| Comprobantes | 0, «Ticket promedio: S/ 0.00» | Sin ventas hoy | Sí |
| Mis ventas de hoy | S/ 0.00, «0 comprobantes» | La administradora no vendió | Sí |
| Ventas en espera | 0, «Por cobrar: S/ 0.00» | La venta `115BF400` se anuló en el capítulo 9 | Sí |
| Cobrado por método de pago | Cuatro filas en S/ 0.00 | Sin ventas hoy | Sí |
| Ventas por sucursal | «Sede Principal» 0 / S/ 0.00; «Sucursal Demo Norte» 0 / S/ 0.00; «Total empresa» 0 / S/ 0.00 | Sin ventas hoy | Sí |
| Caja | «No tienes una caja abierta» y botón **Abrir caja**; sin «Cajas abiertas» | Ninguna caja abierta (las dos sesiones del cajero estaban cerradas) | Sí |
| Lotes vencidos con stock | 0 | Ningún lote tiene caducidad anterior al 04/11/2026 | Sí |
| Lotes que vencen en 30 días o menos | 1 | `LOT-PARA-CERCA` caduca el 15/11/2026, dentro de los 30 días | Sí |
| Lotes que vencen en 31 a 60 días | 0 | Ningún otro lote cae en ese tramo | Sí |
| Lotes que vencen en 61 a 90 días | 0 | Ídem | Sí |
| Productos sin stock | 0 | Todos los productos tienen unidades vendibles | Sí |
| Productos con stock bajo | 0 | El menor stock vendible de un producto en una sucursal es 20 (Amoxicilina en «Sucursal Demo Norte», según el capítulo 6), por encima del umbral de 10 | Sí |
| Lotes en cuarentena | 0 | La devolución con destino «Cuarentena DIGEMID» del capítulo 9 no crea un registro de cuarentena (`RN-DAS-022`) | Sí |
| Ventas de los últimos 7 días | 29/10 0.00; 30/10 0.00; 31/10 0.00; 01/11 0.00; **02/11 S/ 95.20**; **03/11 S/ 0.40**; Hoy 0.00 | 02/11: 45.00 + 4.00 + 14.40 + 5.00 + 11.00 + 3.80 + 12.00 = 95.20 (la anulada `B001-00000005` de 8.90 no cuenta; la devuelta `B001-00000002` sí, porque el monto es bruto); 03/11: 0.40 | Sí |
| Productos más vendidos en 7 días | Paracetamol Demo 500 mg, 150 unidades, S/ 69.00; Ibuprofeno Demo 400 mg, 17 unidades, S/ 6.80; Amoxicilina Demo 500 mg, 10 unidades, S/ 11.00; Clonazepam Demo 2 mg, 2 unidades, S/ 5.00; Tramadol Demo 50 mg, 1 unidad, S/ 3.80 | Paracetamol: 100 (`01`) + 30 (`03`) + 20 (`08`) = 150 unidades y 45.00 + 14.40 + 9.60 = 69.00. Ibuprofeno: 10 + 6 + 1 = 17 y 4.00 + 2.40 + 0.40 = 6.80. Omeprazol no aparece porque su única venta (`05`) se anuló | Sí |
| Devoluciones | 0, «Monto reembolsado: S/ 0.00» | Las dos devoluciones se hicieron el 03/11, no hoy | Sí |

Todo coincidió. Dos detalles para no confundirte:

- La devolución parcial de `B001-00000001` (S/ 4.50) **no** baja la cifra del 02/11: «Vendido» es bruto. El reembolso se ve en «Devoluciones de hoy» el día en que se procesó.
- El texto «1 unidades» en el último producto es tal como lo escribe la pantalla.

### 3.8 El mismo Inicio con una venta y una caja abierta

Más adelante en el capítulo (secciones 5 y 6) el cajero abrió una caja con S/ 30.00 y cobró una venta de S/ 1.20 en efectivo (recibió S/ 2.00 y dio S/ 0.80 de vuelto). Con eso, el Inicio del **administrador** mostró:

| Indicador | Valor visto | Esperado | Coincide |
|---|---|---|---|
| Vendido hoy | S/ 1.20, flecha hacia arriba «+200.0% vs. ayer · Ayer: S/ 0.40» | (1.20 − 0.40) ÷ 0.40 = +200 % | Sí |
| Comprobantes | 1, «Ticket promedio: S/ 1.20» | Una venta de 1.20 | Sí |
| Mis ventas de hoy | S/ 0.00, «0 comprobantes» | La venta es del cajero, no de la administradora | Sí |
| Cobrado por método de pago | Efectivo S/ 1.20; los demás S/ 0.00 | 2.00 recibido − 0.80 de vuelto = 1.20 en efectivo | Sí |
| Ventas por sucursal | «Sede Principal» 1 / S/ 1.20; «Sucursal Demo Norte» 0 / S/ 0.00; «Total empresa» 1 / S/ 1.20 | La venta es de la Sede Principal | Sí |
| Caja | «No tienes una caja abierta» y **Abrir caja**; además «Cajas abiertas (1)» con «Carlos Cajero», «Sede Principal», «04/11/2026 17:21» y **Ver cajas abiertas** | La caja abierta es del cajero, no de la administradora | Sí |
| Ventas de los últimos 7 días | La última barra, «Hoy», S/ 1.20 | Igual que «Vendido hoy» | Sí |
| Productos más vendidos | Ibuprofeno Demo 400 mg pasó a 20 unidades · S/ 8.00 (17 + 3 unidades, 6.80 + 1.20) | Se vendieron 3 unidades más | Sí |

## 4. Los botones de acceso directo

Varias tarjetas tienen un botón que abre la pantalla correspondiente **ya filtrada**, para que no tengas que buscar. Se recorrieron así:

| Botón | Dónde está | Qué abre (comprobado) |
|---|---|---|
| **Ver lotes** (cuatro, uno por tarjeta de lotes) | Bloque Inventario | Stock por Sucursal con los filtros ya puestos y «Solo con stock disponible» marcado. En «Lotes que vencen en 30 días o menos» (administrador): «Caducidad Desde» 04/11/2026 y «Caducidad Hasta» 04/12/2026 (hoy y hoy + 30 días); resultado: 1 fila, «Sede Principal», «Paracetamol Demo 500 mg», «LOT-PARA-CERCA», «15/11/2026», cantidad 15. Los otros tres botones usan los tramos de la sección 3.5 (No verificado en la aplicación: no se abrieron; viene de `producto/pantallas/acceso-layout-y-compartidos` §3.1) |
| **Ver ventas de hoy** | Tarjeta «Vendido hoy» | Ventas (`/pos`) con «Fecha desde» y «Fecha hasta» del día de hoy y «Estado» en «Completada». Con la venta de S/ 1.20: «Resultados (1)», `B001-00000010` |
| **Ver mis ventas** | Tarjeta «Mis ventas de hoy» | Ventas con «Empleado» en «Carlos Cajero Demo» (tu nombre), «Estado» «Completada» y las fechas de hoy. «Resultados (1)» |
| **Ver ventas en espera** | Tarjeta «Ventas en espera» | Ventas con «Estado» en «Pendiente» y **sin fechas**, porque una venta en espera no caduca. Sin ventas en espera: «Resultados (0)» y «Sin resultados» |
| **Abrir caja** | Bloque Caja, cuando no tienes caja abierta | La pantalla de apertura de caja (`/sesiones-caja/nueva`) |
| **Ver cajas abiertas** | Bloque Caja, cuando hay cajas de otras personas | Sesiones de Caja con «Estado» en «Abierta». Con la caja del cajero abierta: «Resultados (1)», «Sede Principal», «Carlos Cajero», «04/11/2026 17:21», «30.00», «Abierta» |
| **Ver mi caja** | Bloque Caja, cuando tienes caja abierta | El detalle de tu sesión de caja (No verificado en la aplicación: no se pulsó; viene de `producto/pantallas/acceso-layout-y-compartidos` §3.1) |

Dos cosas que debes saber de estos botones:

- **«Ver ventas de hoy», «Ver mis ventas» y «Ver ventas en espera» no aparecen cuando ves más de una sucursal** (caso de un administrador con dos o más sucursales). Motivo: la lista de ventas solo muestra las ventas de **tu sucursal** (`RN-DAS-014`), y la cifra de la tarjeta sería de varias. El administrador de la empresa de prueba tiene dos sucursales y por eso **no** ve esos tres botones; el cajero, que ve una sola, **sí** los ve. Un administrador con una sola sucursal también los vería (No verificado en la aplicación: la empresa de prueba tiene dos).
- **La cifra de la tarjeta y la lista pueden no coincidir.** «Vendido hoy» incluye las ventas «Devuelta», pero **Ver ventas de hoy** filtra por «Completada»: si hoy devolviste todo lo de una venta, esa venta suma en la tarjeta y no sale en la lista (hallazgo 7 de `negocio/reglas/caja-devoluciones-y-dashboard` §6; No verificado hoy porque no hubo devoluciones el día del recorrido).
## 5. Ventas por sucursal

**Cuándo aparece.** La tarjeta «Ventas por sucursal» aparece **solo cuando ves más de una sucursal** (`RN-DAS-011`). Tiene una fila por cada sucursal (ordenadas por nombre, con 0 si no vendió) y un pie que suma lo mismo que «Vendido hoy» y «Comprobantes»:

| Columna | Qué muestra |
|---|---|
| Sucursal | El nombre de la sucursal |
| Comprobantes | Número de ventas que cuentan hoy en esa sucursal |
| Vendido | Total bruto de hoy en esa sucursal |
| Pie | «Total empresa» si ves toda la empresa (administrador); «Total de tus sucursales» si ves varias pero no todas |

- **Administrador (rol «Admin»):** ve **todas** las sucursales de la empresa, así que siempre ve la tabla cuando la empresa tiene dos o más, con el pie «Total empresa».
- **Otro rol:** ve su sucursal base más las que el administrador le asignó a su cuenta o a su rol (capítulo 3). Con una sola, no ve la tabla; con varias, la ve con el pie «Total de tus sucursales» (No verificado en la aplicación: ningún usuario de la empresa de prueba tiene sucursales adicionales y este capítulo no cambia cuentas).

**Lo que se vio.** En la empresa de prueba, la tabla del administrador salió así:

| Momento | Sede Principal | Sucursal Demo Norte | Total empresa |
|---|---|---|---|
| Sin ventas hoy (17:20) | 0 comprobantes, S/ 0.00 | 0 comprobantes, S/ 0.00 | 0, S/ 0.00 |
| Con la venta de S/ 1.20 (17:25) | 1 comprobante, S/ 1.20 | 0 comprobantes, S/ 0.00 | 1, S/ 1.20 |

**No se pudo abrir una caja en «Sucursal Demo Norte».** Se intentó el camino normal: iniciar sesión como administrador y abrir una caja en esa sucursal. Menú lateral › **Sesiones de Caja** › **Nuevo** (o el botón **Abrir caja** del Inicio) abre una pantalla con el texto «La caja se abrirá a tu nombre, en tu sucursal activa y con la hora actual.», el campo «Cajero» (con tu nombre, bloqueado) y el campo «Monto Apertura (S/)», más los botones **Cancelar** y **Aperturar**. **No hay un campo para elegir la sucursal**: la caja siempre se abre en tu sucursal base, que para la administradora de prueba es la «Sede Principal». Por eso no se abrió ninguna caja allí ni se hizo ninguna venta en esa sucursal, y no se buscó otro camino. La fila de «Sucursal Demo Norte» quedó en 0 durante todo el recorrido.

Lo que sigue es una deducción y **no se verificó**: para vender en una sucursal hace falta un empleado cuya sucursal base sea esa (la matriz de permisos `tecnico/seguridad/matriz-de-permisos` §7 indica que abrir caja y vender usan la sucursal base del usuario). En la empresa de prueba, «Nora Norte Demo» tiene su base en «Sucursal Demo Norte», pero todavía no tiene cuenta de usuario.

## 6. El Inicio como cajero

**Cómo llegar:** inicia sesión con tu cuenta de cajero (para esta prueba, `carlos.cajero`); llegas solo al Inicio.

### 6.1 Qué cambia respecto del administrador

El cajero tiene los mismos cinco bloques (su rol tiene los cinco `Leer`), con estas diferencias:

| Elemento | Administrador (`ana.manual`) | Cajero (`carlos.cajero`) |
|---|---|---|
| Alcance de las cifras | **Toda la empresa** (las dos sucursales) | **Solo las sucursales a las que tiene acceso**: la «Sede Principal» (su sucursal base) |
| Tarjeta «Ventas por sucursal» | La ve (tiene dos sucursales) | **No la ve** (solo ve una) |
| Botones **Ver ventas de hoy**, **Ver mis ventas**, **Ver ventas en espera** | **No los ve** (ve más de una sucursal) | **Los ve** |
| «Mis ventas de hoy» | Sus propias ventas (S/ 0.00 en la prueba) | Sus propias ventas (S/ 1.20 después de vender) |
| Bloque Caja | «No tienes una caja abierta» y **Abrir caja**; si el cajero tiene caja abierta, «Cajas abiertas (1)» con su fila | «No tienes una caja abierta» y **Abrir caja**; con caja propia, «Tienes una caja abierta» con el «Efectivo esperado» |
| Inventario, tendencia, más vendidos y devoluciones | Mismos bloques, de toda la empresa | Mismos bloques, de su sucursal |

Qué **no** ve el cajero: la tabla «Ventas por sucursal» ni el pie «Total empresa», ni las cifras de sucursales a las que no tiene acceso. En la prueba los totales del administrador y del cajero fueron **iguales** (S/ 0.00 y Ayer S/ 0.40, y después S/ 1.20) porque «Sucursal Demo Norte» no tiene ventas; la diferencia está en **qué** muestra la pantalla, no en las cifras.

El Inicio del cajero a las 17:21, antes de abrir su caja, mostró: «Vendido hoy» S/ 0.00 con «-100.0% vs. ayer · Ayer: S/ 0.40» y el botón **Ver ventas de hoy**; «Comprobantes» 0; «Mis ventas de hoy» S/ 0.00 con **Ver mis ventas**; «Ventas en espera» 0 con **Ver ventas en espera**; «Cobrado por método de pago» en cero; «No tienes una caja abierta» con **Abrir caja**; y los mismos indicadores de inventario, tendencia, más vendidos y devoluciones que el administrador.

### 6.2 Abrir una caja corta y ver el «Efectivo esperado» en vivo

Esta prueba usa las pantallas que ya conoces del [capítulo 8](capitulo-08-caja-y-venta.md) y del [capítulo 9](capitulo-09-ventas-devoluciones-y-cierre.md).

1. En el Inicio, en el bloque **Caja**, haz clic en **Abrir caja**.
2. En «Monto Apertura (S/)» escribe `30`.
3. Haz clic en **Aperturar**. La pantalla vuelve a la lista de Sesiones de Caja.
4. Vuelve al Inicio. El bloque Caja ahora dice «Tienes una caja abierta» con:

| Dato | Valor visto | Esperado |
|---|---|---|
| Sucursal | Sede Principal | La sucursal base del cajero |
| Abierta desde | 04/11/2026 17:21 | La hora de apertura |
| Monto de apertura | S/ 30.00 | Lo que escribiste |
| Efectivo esperado | S/ 30.00 | Apertura 30.00 + ventas en efectivo 0.00 |

5. Vende en el terminal: menú lateral › **Ventas** › **Nueva Venta POS**; elige «Ibuprofeno Demo 400 mg», el lote `LOT-IBU-2027`, cantidad 3; método «Efectivo», «Recibido (S/)» 2; el terminal muestra «Vuelto: S/ 0.80» y «TOTAL A PAGAR: S/ 1.20». Haz clic en **Procesar Venta** y luego en **Confirmar y Procesar Venta**. Resultado: «Venta Procesada Exitosamente», boleta `B001-00000010`, «Estado SUNAT: Pendiente de envío a SUNAT».
6. Vuelve al Inicio. El cajero vio:

| Indicador | Valor visto | Esperado | Coincide |
|---|---|---|---|
| Vendido hoy | S/ 1.20, «+200.0% vs. ayer · Ayer: S/ 0.40» | (1.20 − 0.40) ÷ 0.40 = +200 % | Sí |
| Comprobantes | 1, «Ticket promedio: S/ 1.20» | Una venta | Sí |
| Mis ventas de hoy | S/ 1.20, «1 comprobantes» | La venta es suya | Sí |
| Cobrado por método de pago, Efectivo | S/ 1.20 | 2.00 recibido − 0.80 de vuelto | Sí |
| Efectivo esperado | S/ 31.20 | 30.00 de apertura + 2.00 recibido − 0.80 de vuelto = 31.20 | Sí |
| Ventas de los últimos 7 días, «Hoy» | S/ 1.20 | Igual que «Vendido hoy» | Sí |

7. **Cierra la caja cuadrada.** Menú lateral › **Sesiones de Caja** › el enlace de la sucursal de tu sesión abierta › **Cerrar Sesión** › en «Monto Cierre Declarado (S/)» escribe `31.2` › **Confirmar Cierre**. La pantalla vuelve a la lista. Resultado: la sesión aparece con «Cierre» 04/11/2026 17:27, «Monto Apertura» 30.00 y «Estado» **Cuadrada** (declaraste exactamente el efectivo esperado: diferencia 0). La caja se cerró desde una segunda pestaña, para probar la actualización (sección 7).

Si algo no sale como esperabas, mira el capítulo 8 (apertura y venta) y el capítulo 9 (cierre con arqueo y sus mensajes).

## 7. Cuándo se actualizan los indicadores

El Inicio **no se actualiza solo**: no tiene temporizador ni recarga automática (`RN-DAS-024`; `sofia-web-spa/src/app/pages/dashboard/dashboard.component.ts:200-208`). Las cifras se piden de nuevo en estos momentos:

| Cuándo | Qué pasa | Comprobado |
|---|---|---|
| Entras al Inicio (inicias sesión o abres su dirección en el navegador) | Se piden de nuevo todas las cifras de los bloques que puedes ver y se recalcula «hoy» | Sí: después de la venta de S/ 1.20, al abrir de nuevo el Inicio las cifras ya la incluían y «Actualizado a las» cambió |
| Recargas la página del navegador | Igual que entrar: se piden de nuevo; no pierdes la sesión | Sí (cada vez que se abrió el Inicio por su dirección) |
| Vuelves al Inicio desde otra pantalla con el menú lateral | Igual: la pantalla se crea de nuevo y pide las cifras al abrirse | No verificado en la aplicación: se probó abriendo la dirección, no con el menú. Viene del código (`dashboard.component.ts:180-198`) |
| Pulsas **Actualizar** | Se piden de nuevo todas las cifras y se recalcula «hoy»; «Actualizado a las» muestra la nueva hora | Sí: ver la prueba de abajo |
| Otra persona vende, cobra o cierra su caja mientras tienes el Inicio abierto | **No cambia nada** hasta que entres de nuevo o pulses **Actualizar** | Sí: ver la prueba de abajo |
| Pasa la medianoche con el Inicio abierto | «Hoy» se calcula al entrar o al pulsar **Actualizar**; hasta entonces la pantalla sigue mostrando los datos del día anterior como si fueran «hoy». Pulsa **Actualizar** al empezar el día | No verificado en la aplicación: no se esperó a la medianoche. Viene del código (`dashboard.component.ts:200-201`) |

**La prueba.** Con el cajero en el Inicio («Actualizado a las 17:26», «Tienes una caja abierta», «Efectivo esperado S/ 31.20»), se cerró esa misma caja desde otra pestaña del navegador. La primera pestaña **siguió mostrando «Tienes una caja abierta» y S/ 31.20**. Al pulsar **Actualizar**, pasó a «Actualizado a las 17:27» y «No tienes una caja abierta». «Vendido hoy» siguió en S/ 1.20 (cerrar una caja no cambia lo vendido).

**El día calendario.** «Hoy» es el día calendario del **computador desde el que miras** (de 00:00 a 24:00 de su zona horaria), no un periodo de 24 horas hacia atrás ni el turno de caja (`RN-DAS-002` y `RN-DAS-004`). Se vio así: las ventas del 02/11 por la noche (19:18 a 19:26 hora de Lima) cuentan para el 02/11 en la tendencia, y el 04/11 empezó con «Vendido hoy» en 0 aunque ayer y anteayer sí hubo ventas. Dos computadores con la hora mal puesta (más de 36 horas de diferencia con el servidor) dejan sin datos las secciones de ventas, inventario y devoluciones, con el aviso «No se pudo cargar esta sección.» (No verificado en la aplicación: no se cambió la hora del equipo.)

**Una sección que falla no tumba las demás.** Si una sección no se puede cargar, solo esa tarjeta muestra «No se pudo cargar esta sección.» con el botón **Reintentar**; el resto del Inicio sigue. Si fallan tus permisos, aparece «No se pudieron cargar tus permisos.» con **Reintentar**. (No verificado en la aplicación: no se provocó ningún fallo.)

## 8. Administrador frente a cajero en toda la aplicación

Esta sección reúne en un solo lugar lo que cada capítulo anterior comprobó. **Cajero** significa aquí el rol «Cajero» creado en el [capítulo 3](capitulo-03-empleados-roles-cuentas.md) (sección 5.3, 19 permisos); **administrador** es el rol «Admin» (la cuenta que registró la empresa), que pasa siempre sin tener casillas marcadas. Un rol distinto de «Cajero» ve lo que sus permisos permitan: la regla es la misma, `Leer` del módulo abre la pantalla y cada botón pide su acción.

**Dos reglas que lo explican todo**

1. **El menú sigue a `Leer`.** Una opción del menú aparece si el rol tiene el permiso `Leer` de su módulo; un grupo sin opciones visibles desaparece. «Dashboard» siempre aparece.
2. **Cada botón sigue a su acción.** **Nuevo** y **Carga Masiva** piden `Crear`; **Editar**, `Actualizar`; **Eliminar**, `Eliminar`. **Exportar** lo ve quien puede abrir la búsqueda. Si escribes a mano la dirección de una pantalla sin permiso, el sistema te lleva al Inicio.

### 8.1 Menú lateral

| Grupo › opción | Administrador | Cajero | Dónde se documentó |
|---|---|---|---|
| INICIO › Dashboard | Sí | Sí | Capítulo 1 §4.5; este capítulo |
| ADMINISTRACIÓN › Empresa | Sí | No (falta `Empresa › Leer`) | Capítulo 2 §12 |
| ADMINISTRACIÓN › Sucursales | Sí | No | Capítulo 2 §12 |
| ADMINISTRACIÓN › Empleados | Sí | Sí, solo consulta (`Empleados › Leer`) | Capítulo 3 §8.2 y §11 |
| ADMINISTRACIÓN › Roles & Permisos | Sí | No | Capítulo 3 §11 |
| ADMINISTRACIÓN › Cuentas de Usuario | Sí | No | Capítulo 3 §11 |
| CATÁLOGOS › Unidades de Medida | Sí | No | Capítulo 4 §10 |
| CATÁLOGOS › Jerarquías de Unidades | Sí | No | Capítulo 5 §11 |
| CATÁLOGOS › Laboratorios | Sí | No | Capítulo 4 §10 |
| CATÁLOGOS › Ingredientes Activos | Sí | No | Capítulo 4 §10 |
| CATÁLOGOS › Medicamentos | Sí | Sí, solo consulta | Capítulo 5 §11 |
| CATÁLOGOS › Presentaciones de Venta | Sí | No | Capítulo 5 §11 |
| CATÁLOGOS › Formulaciones Clínicas | Sí | No | Capítulo 5 §11 |
| CATÁLOGOS › Proveedores | Sí | No | Capítulo 5 §11 |
| CATÁLOGOS › Aseguradoras | Sí | Sí, solo consulta | Capítulo 5 §11 |
| INVENTARIO › Lotes | Sí | Sí, solo consulta | Capítulo 6 §13 |
| INVENTARIO › Stock por Sucursal | Sí | Sí, solo su sucursal | Capítulo 6 §13 |
| PUNTO DE VENTA › Sesiones de Caja | Sí | Sí | Capítulos 8 §18 y 9 §10 |
| PUNTO DE VENTA › Ventas | Sí | Sí | Capítulos 8 §18 y 9 §10 |
| PACIENTES & ATENCIÓN › Pacientes | Sí | Sí, solo consulta | Capítulo 7 §13 |
| PACIENTES & ATENCIÓN › Profesionales de Salud | Sí | Sí, solo consulta | Capítulo 7 §13 |
| PACIENTES & ATENCIÓN › Recetas Médicas | Sí | Sí, con **Escanear receta** y **Nuevo** | Capítulo 7 §13 |
| DIGEMID › Catálogo DIGEMID | Sí | Sí, solo consulta y **Exportar** | Capítulo 4 §10 |
| SUNAT › Series Fiscales | Sí | No (el grupo SUNAT desaparece) | Capítulo 2 §12 |

Resumen: el administrador ve **24 opciones en 8 grupos**; el cajero del ejemplo ve **12 opciones en 7 grupos** (no ve SUNAT).

**Comprobado en este capítulo con el cajero (lo que faltaba):** escribir a mano `/series-fiscales` lo devolvió al Inicio con el aviso rojo «No tienes permisos para acceder a esta pantalla.»; escribir `/lotes/nueva` y `/proveedores` también lo devolvió al Inicio (el aviso no alcanzó a leerse en esas dos pruebas). Con esto quedan comprobadas las redirecciones que los capítulos 4, 5 y 6 habían dejado como «No verificado». El capítulo 3 ya había comprobado `/roles`, `/cuentas` y `/empleados/nueva`, y el capítulo 7 `/pacientes/nueva` y `/pacientes/carga-masiva`.

### 8.2 Qué puede hacer cada uno, pantalla por pantalla

| Pantalla | Administrador | Cajero | Capítulo |
|---|---|---|---|
| Empresa | Ve y edita los datos de la empresa | No la ve | 2 §12 |
| Sucursales y Series Fiscales | Búsqueda, **Nuevo**, **Carga Masiva** (sucursales), **Editar**, **Eliminar**, **Exportar** | No las ve | 2 §12 |
| Empleados | **Carga Masiva**, **Nuevo**, **Editar**, **Eliminar**, **Exportar** | Lista y ficha de solo lectura y **Exportar**; ve correos y licencias de todos los empleados | 3 §11 |
| Roles & Permisos y Cuentas de Usuario | Todo: crear roles, marcar permisos, crear cuentas, asignar roles, activar, desbloquear, eliminar. Es el único que puede modificar cuentas de otros administradores | Sin acceso | 3 §11 |
| Cambiar la propia contraseña | Sí | Sí, sin permiso especial; la primera vez es obligatorio si el administrador le dio una contraseña inicial | 1 §10 y 3 §11 |
| Unidades de Medida, Laboratorios, Ingredientes Activos | Según `Crear`, `Actualizar`, `Eliminar` | Sin acceso | 4 §10 |
| Catálogo DIGEMID | Filtros, tabla, **Exportar** y **Carga Masiva** | Filtros, tabla y **Exportar**; **no ve Carga Masiva** | 4 §10 |
| Medicamentos y Aseguradoras | Filtros, **Exportar**, **Carga Masiva**, **Nuevo**, **Editar**, **Presentaciones de Venta** (medicamentos) | Filtros, tabla, **Exportar**, ficha con «Precio de Venta Base» y «Stock por Sucursal»; sin **Nuevo**, **Carga Masiva**, **Editar** ni **Presentaciones de Venta** | 5 §11 |
| Jerarquías, Presentaciones, Formulaciones y Proveedores | Acceso completo; precios de compra con `Registrar precio de proveedor` | Sin acceso. No ve precios de compra | 5 §11 |
| Lotes | **Nuevo**, **Editar**, **Guardar**, **Eliminar**, **Exportar** | Mismos filtros y tabla (7 lotes) y **Exportar**; sin **Nuevo** ni **Editar** | 6 §13 |
| Stock por Sucursal | Stock de **todas** las sucursales (9 filas), **Nuevo**, ajustes | Solo stock de su sucursal (7 filas); el filtro «Sucursal» solo ofrece la suya; sin **Nuevo** ni **Editar** | 6 §13 |
| Pacientes | **Carga Masiva**, **Nuevo**, **Editar**, **Eliminar**, **Exportar** | Solo **Limpiar**, **Buscar**, **Exportar** y ficha de solo lectura: no puede registrar pacientes | 7 §13 |
| Profesionales de Salud | Todo | Solo **Limpiar** y **Buscar** (y **Exportar**); ficha de solo lectura | 7 §13 |
| Recetas Médicas | **Escanear receta**, **Nuevo**, **Editar**, **Eliminar**, **Exportar** | Los mismos cinco botones de la búsqueda (su rol tiene `Crear` y `Analizar con IA`); ficha de solo lectura. Puede registrar la receta de un cliente, pero no al paciente | 7 §13 |
| Sesiones de Caja: abrir | **Nuevo** abre la caja del administrador en su sucursal base | **Nuevo** abre la caja del cajero en su sucursal base. Ninguno elige sucursal (sección 5) | 8 §18 |
| Sesiones de Caja: ver | Cajas de **todas** las sucursales de la empresa | Cajas de las sucursales a las que tiene acceso; en la prueba, las mismas | 9 §10 |
| Sesiones de Caja: cerrar | Puede cerrar la caja de cualquier cajero (No verificado en la aplicación: vio el botón **Cerrar Sesión** en la caja ajena, pero no la cerró) | Solo la caja que él abrió | 9 §10 |
| Ventas: buscar y detalle | Ve las ventas de **su sucursal**, de todos los empleados; filtra por «Empleado» para ver solo las de uno | Igual: ve las de su sucursal, de todos los empleados | 9 §10 |
| Terminal de venta | Necesita **su propia caja abierta** para vender | Necesita **su propia caja abierta** | 8 §18 |
| **Anular Venta** y **Devolver** | Funcionan solo con **su propia caja abierta**; sin ella: «Necesitas tener tu propia caja abierta en esta sucursal para anular una venta cobrada.» y «Necesitas una caja abierta para procesar la devolución.» Ser administrador no evita la regla | Funcionan con su caja abierta, porque su rol tiene `Ventas › Anular` y `Devoluciones › Procesar devolución` | 9 §10 |
| Anular una venta en espera | No pide caja | No pide caja | 9 §10 |
| Inicio | Cifras de **toda la empresa**, tabla «Ventas por sucursal» y sin los botones de ventas (si hay varias sucursales) | Cifras de su sucursal, «Mis ventas de hoy», su caja con «Efectivo esperado» y los botones de ventas | Este capítulo (secciones 5 y 6) |
| Cabecera y menú de usuario | Fecha, campana, idioma, nombre, empresa, **Cambiar contraseña**, **Cerrar sesión**, versión | Igual | 1 §10 |

### 8.3 Lo que solo puede hacer un administrador

- Registrar la empresa, crear sucursales, series fiscales, empleados, roles y cuentas (capítulos 1 a 3).
- Asignar permisos a un rol y asignar o quitar el rol «Admin».
- Modificar o eliminar cuentas de **otros administradores**.
- Ver las cajas y el stock de **todas** las sucursales de la empresa.
- Cerrar la caja de **otro** cajero (visto el botón; el cierre ajeno no se comprobó).
- Todo lo que el cajero hace solo si su rol tiene el permiso: lo que cambia no es la pantalla, sino los permisos del rol. Si necesitas que un cajero registre pacientes, por ejemplo, agrega `Pacientes › Crear` a su rol (capítulo 3); este capítulo no cambia permisos.

## 9. Diferencias con la documentación técnica

No se encontró ninguna diferencia entre la aplicación y la documentación técnica (`producto/pantallas/acceso-layout-y-compartidos` §3, `RN-DAS-001` a `RN-DAS-024`): las cifras coincidieron con lo esperado y los textos de la pantalla coinciden con `public/i18n/dashboard/es.json`.

Una precisión sobre el capítulo 1: su sección 4.5 lista los botones **Ver ventas de hoy**, **Ver mis ventas** y **Ver ventas en espera** como parte del Inicio y ya dice que se ocultan cuando el usuario ve más de una sucursal, con remisión a este capítulo para la condición completa (`RN-DAS-014`; sección 4 de este capítulo). No hay diferencia entre ambos capítulos.

## 10. Pendientes y «No verificado»

| # | Qué | Motivo |
|---|---|---|
| 1 | Abrir una caja y vender en «Sucursal Demo Norte» | La pantalla de apertura no deja elegir la sucursal (sección 5). La fila de esa sucursal quedó en 0 durante todo el recorrido |
| 2 | Tabla «Ventas por sucursal» con pie «Total de tus sucursales» | Ningún usuario de la empresa de prueba tiene sucursales adicionales; el capítulo no cambia cuentas |
| 3 | Botones **Ver ventas de hoy**, **Ver mis ventas** y **Ver ventas en espera** de un administrador con una sola sucursal | La empresa de prueba tiene dos |
| 4 | Botón **Ver mi caja**; botones **Ver lotes** de las tarjetas «Lotes vencidos», «31 a 60 días» y «61 a 90 días» | No se pulsaron; el comportamiento viene de `producto/pantallas/acceso-layout-y-compartidos` §3.1 |
| 5 | Tarjetas de inventario con cifras distintas de cero para «Lotes vencidos con stock», «Productos sin stock», «Productos con stock bajo» y «Lotes en cuarentena» | La empresa de prueba no tiene lotes vencidos, ni productos sin stock o con stock bajo, ni lotes en cuarentena; tampoco se cambió el stock |
| 6 | Tarjeta «Devoluciones de hoy» con datos | Las dos devoluciones se hicieron el 03/11; hoy no hubo ninguna. El 03/11 se vio en el capítulo 9 (Devoluciones 2, S/ 8.50) |
| 7 | Flecha de «sin cambio» (`trending_flat`) y variación cuando ayer no vendió | Solo se vieron la flecha hacia abajo (−100.0 %) y hacia arriba (+200.0 %) |
| 8 | Aviso «Tu usuario no tiene indicadores disponibles aquí. Usa el menú lateral para ir a tus módulos.» | Habría que quitar todos los permisos de lectura de un rol |
| 9 | Saludo «Bienvenido a SOFIA» (cuenta sin nombre) | Todas las cuentas de prueba tienen nombre |
| 10 | Estados de error: «No se pudo cargar esta sección.», «No se pudieron cargar tus permisos.», **Reintentar** | No se provocó ningún fallo |
| 11 | Rechazo del servidor por un «hoy» a más de 36 horas de la hora real | No se cambió la hora del equipo |
| 12 | El Inicio abierto al pasar la medianoche | No se esperó a la medianoche |
| 13 | El administrador cerrando la caja de otro cajero | Solo se vio el botón; no se cerró (capítulo 9) |
| 14 | Volver al Inicio con el menú lateral vuelve a pedir las cifras | Se probó abriendo la dirección; viene del código |
| 15 | Una cuenta de cajero de otro rol (otros permisos) | Solo existe el rol «Cajero» del capítulo 3; este capítulo no cambia permisos |
| 16 | Formato de cifras de mil o más (coma de miles) | Ninguna cifra llegó a 1,000 |

**En curso (sin commitear):** ninguno; `git status --short` de ambos repositorios estaba limpio al empezar.

**Hallazgos de la aplicación para quien la mantiene**

- **No se puede elegir sucursal al abrir una caja** (pantalla `/sesiones-caja/nueva`), de modo que un administrador de varias sucursales no puede vender en una que no sea su sucursal base. La tabla «Ventas por sucursal» solo se puede llenar con ventas de empleados cuya base sea cada sucursal.
- Al cambiar el método de pago de «Efectivo» a «Tarjeta Débito/Crédito» en el terminal de venta, el campo «Monto (S/)» mostró **1.2000000000000002** en lugar de 1.20 (error de redondeo visible). Al volver a «Efectivo», el campo «Recibido (S/)» conservó ese valor largo hasta que se escribió otro.
- El texto «1 unidades» (plural fijo) en «Productos más vendidos en 7 días».

## 11. Tabla de cobertura

### 11.1 `/dashboard` (`producto/pantallas/acceso-layout-y-compartidos` §3)

| Elemento de `producto/pantallas/acceso-layout-y-compartidos` §3 | Dónde quedó | Estado |
|---|---|---|
| Ruta, permisos de ruta y de bloque (cinco permisos de lectura) | Secciones 2 y 3.1 | Recorrido (con el cajero, que tiene los cinco; con el administrador, que pasa siempre) |
| Sin selector de sucursal, de día ni de rango | Sección 3.1 y sección 7 | Recorrido (no existen) |
| Refresco solo manual con **Actualizar** | Sección 7 | Recorrido |
| Cabecera: saludo, subtítulo, «Actualizado a las {hora}», **Actualizar** | Sección 3.2 | Recorrido |
| Cabecera: «Bienvenido a SOFIA», «No se pudieron cargar tus permisos.» y **Reintentar**, «Tu usuario no tiene indicadores…» | Secciones 3.1, 3.2 y 7 | No verificado en la aplicación (sección 10, 8 a 10) |
| Vendido hoy, variación, «Ayer» | Secciones 3.3, 3.7 y 3.8 | Recorrido (−100.0 % y +200.0 %) |
| **Ver ventas de hoy** | Sección 4 | Recorrido (cajero) |
| Comprobantes y «Ticket promedio» | Secciones 3.3 y 3.7 | Recorrido |
| Mis ventas de hoy y **Ver mis ventas** | Secciones 3.3, 4 y 6 | Recorrido |
| Ventas en espera, «Por cobrar» y **Ver ventas en espera** | Secciones 3.3 y 4 | Recorrido (en 0; sin ventas en espera) |
| Cobrado por método de pago y nota del vuelto | Secciones 3.3, 3.7 y 3.8 | Recorrido (Efectivo con datos; los otros tres en 0) |
| Ventas por sucursal, columnas, «Total empresa» y «Total de tus sucursales» | Sección 5 | Recorrido «Total empresa»; «Total de tus sucursales»: No verificado |
| Nota «Los montos vendidos son brutos…» | Secciones 1 y 3.1 | Recorrido |
| Caja: «Tienes una caja abierta», Sucursal, Abierta desde, Monto de apertura, Efectivo esperado | Secciones 3.4 y 6.2 | Recorrido |
| Caja: **Ver mi caja** | Sección 4 | No verificado en la aplicación (no se pulsó) |
| Caja: «No tienes una caja abierta» y **Abrir caja** | Secciones 3.4, 4 y 6.2 | Recorrido |
| Caja: «Cajas abiertas ({n})», columnas y **Ver cajas abiertas** | Secciones 3.4, 3.8 y 4 | Recorrido |
| Inventario: las siete tarjetas y sus textos de ayuda | Secciones 3.5 y 3.7 | Recorrido (el valor 1 solo en «30 días»; las demás en 0) |
| Inventario: **Ver lotes** (cuatro) | Sección 4 | Recorrido el de «30 días»; los otros tres: No verificado |
| Tendencia de 7 días | Secciones 3.6, 3.7 y 3.8 | Recorrido |
| Productos más vendidos en 7 días y su mensaje vacío | Secciones 3.6 y 3.7 | Recorrido (el mensaje vacío: No verificado, había ventas) |
| Devoluciones de hoy: «Devoluciones» y «Monto reembolsado» | Secciones 3.6 y 3.7 | Recorrido (en 0); con datos: No verificado hoy |
| 3.2 Diferencias por permiso, rol y alcance (administrador: toda la empresa; botones de ventas ocultos con varias sucursales) | Secciones 4, 5 y 6.1 | Recorrido |
| 3.2 Qué cuenta como «vendido» (sin anuladas ni pendientes; bruto) | Sección 3.3 y 3.7 | Recorrido (comprobado con las ventas anuladas y devueltas) |
| 3.2 Efectivo esperado | Secciones 3.4 y 6.2 | Recorrido (30.00 → 31.20) |
| 3.3 Estados de carga, vacío y error | Secciones 3.1 y 7 | Parcial: error y carga lenta, No verificado; cifras en cero sin texto de «sin datos»: recorrido |
| 3.3 Llamadas a la API | No aplica | No aplica en un manual de usuario; ver las fuentes |
| Comportamientos: filtros guardados al abrir los botones | Sección 4 | Recorrido |
| Comportamientos: tolerancia de 36 horas | Sección 7 | No verificado |
| Comportamientos: formato de cifras `1,234.50` (coma de miles) | Sección 10, fila 16 | No verificado (ninguna cifra llegó a mil; todas se leen con punto decimal, por ejemplo «S/ 95.20») |

### 11.2 Reglas `RN-DAS` y partes de `producto/pantallas/acceso-layout-y-compartidos` §4 y §5

| Elemento | Dónde quedó | Estado |
|---|---|---|
| `RN-DAS-001` permisos y secciones | Secciones 2 y 3.1 | Recorrido / parcial (sin el caso «ningún permiso») |
| `RN-DAS-002` parámetro `hoy` | Sección 7 | No verificado el rechazo; el cálculo del día, recorrido |
| `RN-DAS-003` alcance por sucursal | Secciones 5 y 6.1 | Recorrido |
| `RN-DAS-004` día calendario | Sección 7 | Recorrido |
| `RN-DAS-005` a `RN-DAS-007` | Secciones 3.3, 3.7 y 3.8 | Recorrido |
| `RN-DAS-008`, `RN-DAS-009` | Secciones 3.3 y 6 | Recorrido |
| `RN-DAS-010` | Secciones 3.3 y 3.8 | Recorrido |
| `RN-DAS-011` | Sección 5 | Recorrido |
| `RN-DAS-012`, `RN-DAS-013` | Secciones 3.6 y 3.7 | Recorrido |
| `RN-DAS-014` | Sección 4 | Recorrido |
| `RN-DAS-015`, `RN-DAS-016` | Secciones 3.4, 3.8 y 6.2 | Recorrido |
| `RN-DAS-017` a `RN-DAS-021` | Secciones 3.5 y 3.7 | Recorrido solo el tramo «30 días»; las demás reglas con cifra 0: parcial |
| `RN-DAS-022` | Secciones 3.5 y 3.7 | Recorrido (0 con devolución a «Cuarentena DIGEMID») |
| `RN-DAS-023` | Secciones 3.6 y 3.7 | Recorrido en 0 |
| `RN-DAS-024` | Sección 7 | Recorrido (errores: No verificado) |
| `producto/pantallas/acceso-layout-y-compartidos` §4.3 menú lateral y regla de visibilidad | Sección 8.1 | Recorrido (administrador: 24 opciones; cajero: 12) |
| `producto/pantallas/acceso-layout-y-compartidos` §5.3 `permissionGuard` (aviso y redirección al Inicio) | Sección 8.1 | Recorrido |
| Matriz `tecnico/seguridad/matriz-de-permisos` §6.5 (permisos de un cajero) y §7 (acceso por sucursal) | Secciones 2, 5 y 8 | Consultados |

## 12. Fuentes

**Documentos:** [`producto/pantallas/acceso-layout-y-compartidos` — acceso, layout y compartidos](../producto/pantallas/acceso-layout-y-compartidos.md) (§3, §4.3, §5.3); [`negocio/reglas/caja-devoluciones-y-dashboard` — caja, devoluciones y dashboard](../negocio/reglas/caja-devoluciones-y-dashboard.md) (`RN-DAS-001` a `RN-DAS-024`); [`tecnico/seguridad/matriz-de-permisos` — matriz de permisos](../tecnico/seguridad/matriz-de-permisos.md) (§6.5, §7); capítulos [1](capitulo-01-primeros-pasos.md), [2](capitulo-02-empresa-sucursales-series.md), [3](capitulo-03-empleados-roles-cuentas.md), [4](capitulo-04-catalogos-basicos.md), [5](capitulo-05-productos-y-proveedores.md), [6](capitulo-06-inventario.md), [7](capitulo-07-pacientes-y-recetas.md), [8](capitulo-08-caja-y-venta.md) y [9](capitulo-09-ventas-devoluciones-y-cierre.md).

**Código consultado (solo lectura):** `sofia-web-spa/src/app/pages/dashboard/dashboard.component.ts` (líneas 112-117, 140, 180-208, 260); `sofia-web-spa/public/i18n/dashboard/es.json`; `sofia-apis/src/SOFIA.Application/Dashboard/Queries/GetDashboardVentas/GetDashboardVentasQueryHandler.cs` (línea 28); `sofia-apis/src/SOFIA.Application/Dashboard/Common/DashboardScopeResolver.cs`; `sofia-apis/src/SOFIA.Application/Dashboard/Common/DashboardDias.cs` (línea 6, tolerancia de 36 horas).

**Recorridos hechos en la aplicación** (SPA `http://localhost:4200`, 04/11/2026):

| # | Pantalla y acción |
|---|---|
| 1 | Administrador (`ana.manual`): Inicio completo, lectura de todos los indicadores; **Ver lotes** de «30 días»; **Abrir caja** (pantalla de apertura, sin elegir sucursal; no se abrió ninguna caja) |
| 2 | Cajero (`carlos.cajero`): Inicio completo antes de abrir caja |
| 3 | Cajero: **Abrir caja** del Inicio, apertura con S/ 30.00; Inicio con «Efectivo esperado» S/ 30.00 |
| 4 | Cajero: terminal de venta; 3 unidades de Ibuprofeno en efectivo, recibido S/ 2.00 (boleta `B001-00000010`) |
| 5 | Cajero: Inicio con la venta (S/ 1.20, +200.0 %, «Efectivo esperado» S/ 31.20); **Ver ventas de hoy**, **Ver mis ventas** y **Ver ventas en espera** |
| 6 | Administrador: Inicio con la venta y la caja abierta del cajero («Cajas abiertas (1)»); **Ver cajas abiertas** |
| 7 | Cajero: cierre de la caja desde una segunda pestaña con S/ 31.20 («Cuadrada»); primera pestaña sin cambios hasta **Actualizar** |
| 8 | Cajero: Sesiones de Caja › Buscar (3 sesiones, ninguna abierta); direcciones `/series-fiscales`, `/lotes/nueva` y `/proveedores` (redirigido al Inicio) |

