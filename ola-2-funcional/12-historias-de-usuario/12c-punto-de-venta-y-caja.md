# Historias de usuario — Parte C: punto de venta, caja, devoluciones, dashboard y módulos solo API

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-26 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de los documentos validados (reglas de negocio 01c1, 01c2 v1.1, 01c3 y 01d; inventario de pantallas 19a y 19f; matriz de permisos 05b; procesos de negocio 4, 5, 6, 8 y 9; glosario; manual de usuario capítulos 3 y 5) y comprobación puntual en el código y en los `es.json` del SPA. `git status --short` está limpio en ambos repositorios. No se usó el navegador ni se llamó a la API: los mensajes salen de los documentos fuente y de los archivos `es.json`. |

## 1. Cómo leer este documento

Este documento reúne las historias de usuario de la **parte C**: el punto de venta (ventas, recetas en la venta, pagos, venta en espera, anulación, boleta e impresión, búsqueda y detalle de ventas), la caja (apertura, cierre, arqueo y sesiones), las devoluciones, el dashboard y los módulos que solo existen en la API (transferencias, cuarentena y actas, órdenes magistrales, servicios e inmunizaciones, delivery y auditoría). La parte A cubre acceso y administración (`HU-ADM-…`) y la parte B catálogos, inventario y atención (`HU-CAT-…`, `HU-INV-…`); viven en la misma carpeta.

**Convenciones**

- Los IDs son `HU-VEN-nnn` (ventas y comprobantes), `HU-CAJ-nnn` (caja), `HU-DEV-nnn` (devoluciones), `HU-DAS-nnn` (dashboard) y `HU-OPS-nnn` (módulos solo API). Cada historia es una capacidad de negocio, no un botón.
- Las reglas `RN-…` son las de los catálogos [01c2 (ventas y comprobantes)](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [01c3 (caja, devoluciones y dashboard)](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [01d (módulos solo API)](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) y, para `RN-REC-031`, [01c1](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md). En cada historia las reglas de un mismo documento se enlazan juntas.
- Los textos entre « » son los que ve el usuario, copiados de los `es.json` a través de los documentos fuente (la sección 15.4 describe cómo se comprobaron). Si el sistema no tiene un mensaje propio para un caso, el escenario dice cuál es el mensaje genérico que se ve.
- **Estado.** «Implementada»: hay pantalla (o endpoint, si la historia es de API) y funciona como describen los escenarios. «Parcial»: falta algo y la historia lo dice. «Solo API»: existe el endpoint y ninguna pantalla lo usa.
- **Brecha conocida.** Cuando el sistema hoy se comporta distinto de lo que el negocio esperaría (mensaje genérico, comportamiento sin aviso), el escenario describe el comportamiento **actual** y una nota «Brecha conocida» cita la fuente. Ningún escenario da por cumplido algo que el sistema no hace.
- **No verificado en la aplicación.** Los escenarios salen de la lectura del código y de los documentos validados; no se ejecutó la aplicación. Los casos que dependen de ejecución (concurrencia, redondeos de base de datos, formato de números) se marcan con un comentario `# No verificado`.
- Los permisos se escriben `Módulo:Acción` como los exige el endpoint. En la matriz de permisos de la pantalla de roles, el módulo `POS` se rotula «Sesiones de Caja».
- Los montos están en soles (S/). Los datos de los ejemplos son los ficticios del manual: la empresa «Botica Manual Demo S.A.C.», el cajero «Carlos Cajero Demo» (usuario `carlos.cajero`), la administradora «Ana Demo Prueba» y los productos `DEMO-MED-0001` «Paracetamol Demo 500 mg» (Venta Libre, S/ 0.50 por unidad), `DEMO-MED-0002` «Amoxicilina Demo 500 mg» (Receta Simple, S/ 1.20), `DEMO-MED-0003` «Clonazepam Demo 2 mg» (Receta Retenida, S/ 2.50) y `DEMO-MED-0004` «Tramadol Demo 50 mg» (Estupefaciente, S/ 3.80). La presentación «Caja x10» y los números de lote, de receta y de operación de los escenarios, los prefijos de serie (`B001` para boletas, `BC01` para notas de crédito) y el cliente «Rosa Quispe Demo» son inventados para ilustrar.

**Actores**

| Actor | Quién es |
|---|---|
| Cajero | Usuario de mostrador con un rol limitado (el rol «Cajero» del manual, capítulo 3, sección 5.3): vende, cobra, abre y cierra su caja, anula y devuelve según los permisos que se le marcaron. |
| Supervisor / administrador | Usuario con el rol `Admin` (pasa cualquier comprobación de permiso, RN-GEN-021) o con permisos de gestión. En ventas y devoluciones el `Admin` tampoco ve ni opera otras sucursales distintas de la de su token. |
| Cualquier usuario autenticado | Ve el Inicio (`/dashboard`): cada bloque exige el `Leer` de su módulo. |
| Integrador / equipo técnico | Quien consume la API directamente; es el único actor de las historias `HU-OPS-…` (módulos sin pantalla). |

Los flujos de punta a punta están en el [documento de procesos de negocio](../14-procesos-de-negocio.md): proceso 4 (venta), 5 (devolución y anulación), 6 (caja), 8 (dashboard) y 9 (procesos solo API). Estas historias son su desglose por capacidad.

## 2. Índice

| Sección | Historias |
|---|---|
| 3. Terminal de venta: caja, productos, carrito y cliente | HU-VEN-001 a HU-VEN-007 |
| 4. Recetas en la venta | HU-VEN-008 a HU-VEN-011 |
| 5. Pagos, cobro y comprobante | HU-VEN-012 a HU-VEN-019 |
| 6. Venta en espera | HU-VEN-020 a HU-VEN-022 |
| 7. Anulación de ventas | HU-VEN-023 a HU-VEN-025 |
| 8. Consulta de ventas | HU-VEN-026 a HU-VEN-028 |
| 9. Caja: apertura, cierre y arqueo | HU-CAJ-001 a HU-CAJ-008 |
| 10. Devoluciones | HU-DEV-001 a HU-DEV-008 |
| 11. Dashboard (Inicio) | HU-DAS-001 a HU-DAS-010 |
| 12. Módulos que solo existen en la API | HU-OPS-001 a HU-OPS-022 (transferencias 001 a 005, cuarentena y actas 006 a 010, magistrales 011 a 013, servicios e inmunizaciones 014 a 017, delivery 018 a 020, auditoría 021 y 022) |
| 13. Trazabilidad | Historia → reglas, pantallas y endpoints |
| 14. Pendientes y «No verificado» | — |
| 15. Tabla de cobertura | Reglas, rutas, diálogos y endpoints → historias; verificación |
| 16. Fuentes | — |

Total: 76 historias (28 de ventas, 8 de caja, 8 de devoluciones, 10 de dashboard y 22 de módulos solo API); 46 «Implementada», 17 «Parcial» y 13 «Solo API».

---

## 3. Terminal de venta: caja, productos, carrito y cliente

### HU-VEN-001 — Entrar al terminal de venta con la caja propia abierta

**Como** cajero **quiero** que el terminal de venta compruebe que tengo mi caja abierta **para** poder vender sabiendo que el efectivo de cada venta cae en mi propio cajón.

| | |
|---|---|
| **Módulo** | Ventas / Caja |
| **Pantalla** | `/pos/nueva` — «Terminal de Punto de Venta (POS)» (no está en el menú: se llega desde «Nueva Venta POS» de `/pos`) |
| **Endpoint(s)** | `GET /api/v1/pos/sesiones` (filtro sucursal, empleado y estado Abierta), `GET /api/v1/ventas/configuracion` |
| **Permiso** | Ruta: `Ventas:Leer` y `Ventas:Crear`; buscar la caja: `POS:Leer`; configuración: `Ventas:Crear` |
| **Reglas** | [RN-VEN-001, RN-VEN-002, RN-VEN-003, RN-VEN-004, RN-VEN-005, RN-VEN-077](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Entrar al terminal con mi caja abierta
  Dado que soy el cajero "Carlos Cajero Demo" con los permisos "Ventas:Leer", "Ventas:Crear" y "POS:Leer"
  Y tengo una caja en estado "Abierta" en mi sucursal base
  Cuando abro "/pos/nueva"
  Entonces veo un indicador de carga mientras el terminal comprueba mi caja
  Y luego veo el terminal con la búsqueda de productos, el carrito, los datos del cliente y el método de pago
  Y el terminal no muestra la hora de apertura, el monto inicial ni el efectivo esperado de mi caja

Escenario: No tengo caja abierta
  Dado que no tengo ninguna caja abierta en mi sucursal
  Cuando abro "/pos/nueva"
  Entonces solo veo una tarjeta con el texto "Debes aperturar la caja del turno antes de procesar una venta."
  Y no veo el catálogo, el carrito ni los pagos
  Y no puedo vender ni dejar una venta en espera

Escenario: Abrir la caja desde la tarjeta de aviso
  Dado que no tengo caja abierta y tengo el permiso "POS:AperturarCaja"
  Cuando hago clic en "Aperturar Caja"
  Entonces se abre el diálogo "Aperturar Caja" (ver HU-CAJ-002)
  Y si la apertura termina bien, el terminal vuelve a buscar mi caja y se habilita

Escenario: Sin permiso para abrir caja
  Dado que no tengo caja abierta y no tengo el permiso "POS:AperturarCaja"
  Cuando abro "/pos/nueva"
  Entonces la tarjeta solo informa "Debes aperturar la caja del turno antes de procesar una venta." y no ofrece el botón "Aperturar Caja"

Escenario: Una caja abierta por otro cajero no sirve
  Dado que otro cajero tiene una caja abierta en mi sucursal pero yo no
  Cuando abro "/pos/nueva"
  Entonces veo el aviso de aperturar la caja del turno
  Porque el terminal busca la sesión del empleado y la sucursal de mi sesión de usuario

Escenario: La API rechaza una caja ajena o cerrada
  Dado que el comando de venta llega con una caja de otro cajero o de otra sucursal
  Entonces el servidor responde 403 y veo "La caja seleccionada pertenece a otro cajero o sucursal. Recarga la página para usar tu propia caja abierta."
  Y si la caja indicada no existe o ya no está abierta, el servidor la rechaza y veo "Ocurrió un error al procesar la venta."

Escenario: La ruta exige permisos de ventas
  Dado que no tengo el permiso "Ventas:Crear"
  Cuando escribo "/pos/nueva" en el navegador
  Entonces el sistema me lleva al Inicio con el aviso "No tienes permisos para acceder a esta pantalla."

Escenario: La sucursal y el empleado salen de mi sesión de usuario
  Dado que mi sesión de usuario no trae sucursal
  Cuando el terminal consulta lotes o configuración de venta
  Entonces veo "No se pudo determinar el contexto de sucursal asignada del usuario."
  Y la venta, cuando se registra, queda a nombre del empleado de mi sesión y en mi sucursal base; yo no los elijo

Escenario: Si la configuración de la empresa no carga, el terminal es estricto
  Dado que la consulta "GET ventas/configuracion" falla
  Cuando se abre el terminal
  Entonces no veo ningún mensaje de error
  Y el terminal asume que la empresa exige registro de receta (modo estricto)
```

> **Brecha conocida.** Si el rol no tiene `POS:Leer`, la búsqueda de la caja falla y el terminal lo trata como «sin caja» **sin ningún aviso de error**; el cajero que sí tiene caja abierta ve «Debes aperturar la caja…» y, al intentar abrir, «Ya tienes una caja abierta. Ciérrala antes de abrir otra.» (inventario 19f §3.1). Además, el terminal consulta endpoints de otros módulos (`Medicamentos:Leer`, `Pacientes:Leer`, `Recetas:Leer`; `Recetas:Analizar` y `Recetas:Crear` solo para esas funciones) y ningún control de esos permisos existe en el SPA: si faltan, cada lista falla con su mensaje (19f §0.4; [05b](../../ola-1-fundamentos/05-seguridad/05b-matriz-de-permisos.md) §6.5). Una venta con la caja cerrada, de otra sede o inexistente se rechaza con `Venta.Caja`, que no tiene traducción (RN-VEN-003, RN-VEN-004).

### HU-VEN-002 — Buscar productos y elegir el lote

**Como** cajero **quiero** buscar el producto y ver sus lotes con stock **para** elegir de cuál lote vender.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/nueva` — columna «Producto» (campo «Buscar producto...», chips de condición, tarjetas, filas «Lote:») |
| **Endpoint(s)** | `GET /api/v1/medicamentos`, `GET /api/v1/medicamentos/condiciones-venta`, `GET /api/v1/ventas/lotes-disponibles` |
| **Permiso** | `Medicamentos:Leer` (catálogo) y `Ventas:Crear` (lotes) |
| **Reglas** | [RN-VEN-001, RN-VEN-011, RN-VEN-012, RN-VEN-013, RN-VEN-014, RN-VEN-075](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Ver los productos más vendidos al entrar
  Dado que estoy en el terminal con mi caja abierta y no he escrito nada ni elegido un chip
  Entonces veo hasta 12 tarjetas de producto, con los más vendidos de los últimos 30 días primero
  Y cada tarjeta muestra el nombre comercial, "{laboratorio} · {código nacional}", el precio "S/ 0.50 c/u" y una insignia de stock

Escenario: Buscar por nombre comercial
  Cuando escribo "paracetamol" en "Buscar producto..."
  Entonces, tras unos 300 milisegundos sin teclear, la lista se filtra por nombre comercial
  Y solo se consideran productos con stock vendible en mi sucursal

Escenario: Filtrar por condición de venta
  Cuando hago clic en el chip "Receta Retenida"
  Entonces la búsqueda se repite con ese filtro y el texto actual
  Y los chips disponibles son "Todos", "Venta Libre (OTC)", "Receta Simple", "Receta Retenida" y "Estupefaciente"

Esquema del escenario: Insignia de stock de la tarjeta
  Dado que el stock vendible total del producto en mi sucursal es <stock>
  Entonces la tarjeta muestra la insignia <insignia>

  Ejemplos:
    | stock | insignia   |
    | 0     | Agotado    |
    | 6     | Stock bajo |
    | 10    | Stock bajo |
    | 11    | Disponible |

Escenario: Los productos de receta lo indican
  Dado que el producto es "Clonazepam Demo 2 mg" de condición "Receta Retenida"
  Entonces su tarjeta muestra el aviso "Requiere receta"

Escenario: Producto sin precio en el catálogo
  Dado que un producto no tiene precio de venta base
  Entonces su tarjeta muestra "Sin precio" en lugar de "S/ … c/u"

Escenario: Sin resultados
  Cuando escribo un texto que no coincide con ningún producto vendible
  Entonces veo "No se encontraron medicamentos o lotes disponibles."

Escenario: Elegir un producto y ver sus lotes
  Cuando hago clic en la tarjeta "Paracetamol Demo 500 mg"
  Entonces veo "Lote:" y una fila por lote con el número de lote y "Stock Disp: 500"
  Y se listan como máximo 50 lotes vendibles de mi sucursal, ordenados por vencimiento más próximo y luego por número de lote

Escenario: Solo se ofrecen lotes vendibles
  Dado que un lote está vencido, retenido en cuarentena o con cantidad 0
  Entonces ese lote no aparece en la lista de lotes
  Y "vencido" significa que su fecha de caducidad es menor o igual que ayer a esta hora: un lote que caduca hoy todavía se vende hasta el final del día

Escenario: El sistema no obliga a vender primero el lote que vence antes
  Dado que el producto tiene dos lotes con distinto vencimiento
  Cuando elijo el lote que vence después
  Entonces el sistema lo permite

Escenario: Falla la carga de lotes
  Dado que la consulta de lotes falla
  Cuando elijo un producto
  Entonces la lista de lotes queda vacía y veo "No se pudieron cargar los lotes disponibles. Inténtalo nuevamente."

Escenario: Falla la búsqueda de productos
  Dado que la consulta de medicamentos falla
  Entonces veo "No se pudo cargar el catálogo de medicamentos. Inténtalo nuevamente."
  Y la búsqueda sigue funcionando con el siguiente texto que escriba

Escenario: Un lote que solo tiene stock en otra sucursal no se vende
  Dado que el lote existe en la empresa pero solo tiene fila de stock en otra sucursal
  Cuando una petición de venta lo incluye
  Entonces el servidor la rechaza con "Venta.Lote" y yo veo "Ocurrió un error al procesar la venta."
```

> **Brecha conocida.** El terminal **no muestra la fecha de vencimiento** del lote (el dato llega pero ninguna plantilla lo pinta) ni avisa si hay stock en otra sucursal: los textos de pedido de transferencia desde el POS existen en `pos/es.json` sin uso. Los rechazos de lote inexistente, sin stock en la sucursal o en cuarentena se ven con el mensaje genérico (RN-VEN-011, RN-VEN-012; 19f §3.2, §3.12).

### HU-VEN-003 — Agregar productos al carrito

**Como** cajero **quiero** agregar al carrito la cantidad que pide el cliente, con el precio del catálogo **para** armar la venta sin equivocarme de precio ni de stock.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/nueva` — formulario de agregado y panel «Resumen de Venta» |
| **Endpoint(s)** | Validación local; el servidor revalida al cobrar (`POST /api/v1/ventas`) |
| **Permiso** | `Ventas:Crear` |
| **Reglas** | [RN-VEN-008, RN-VEN-009, RN-VEN-010, RN-VEN-011, RN-VEN-014, RN-VEN-019, RN-VEN-020, RN-VEN-021, RN-VEN-022, RN-VEN-023, RN-VEN-024, RN-VEN-080](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Agregar 100 unidades de un producto de venta libre
  Dado que elegí el lote del producto "Paracetamol Demo 500 mg", con stock 500, a S/ 0.50 por unidad
  Cuando escribo Cantidad 100 y hago clic en "Agregar"
  Entonces el carrito muestra una línea con "S/ 0.50 c/u" y total de línea "S/ 50.00"
  Y el encabezado dice "Resumen de Venta · 1"
  Y los totales muestran "Subtotal (Op. Gravada):" S/ 42.37, "IGV (18%):" S/ 7.63 y "TOTAL A PAGAR:" S/ 50.00
  Y se limpia la selección de lote y producto

Escenario: El precio no se puede cambiar
  Dado que elegí un lote
  Entonces el campo "Precio Unitario (S/)" es de solo lectura
  Y el precio sale del catálogo; el servidor tampoco lo recibe: lo resuelve él mismo

Escenario: La cantidad supera el stock del lote
  Dado que el lote tiene 30 unidades
  Cuando escribo Cantidad 31 y hago clic en "Agregar"
  Entonces veo "Stock máximo disponible alcanzado (30 un.)."
  Y no se agrega la línea

Escenario: Producto sin precio configurado
  Dado que el lote pertenece a un producto sin precio de venta en el catálogo
  Cuando hago clic en "Agregar"
  Entonces veo "Este producto no tiene precio de venta configurado en el catálogo. Pide a un administrador que lo registre."

Escenario: Cantidad inválida
  Cuando escribo Cantidad 0 (o la dejo vacía) y hago clic en "Agregar"
  Entonces veo "Ingresa una cantidad válida para agregar el producto."

Escenario: Orden de las validaciones al agregar
  Dado que la cantidad supera el stock y además el precio es 0
  Cuando hago clic en "Agregar"
  Entonces veo primero el aviso de stock máximo, después el de precio y por último el de cantidad inválida

Escenario: Repetir el mismo lote suma a la línea existente
  Dado que ya tengo en el carrito 100 unidades del lote "L-001" sin presentación
  Cuando agrego 50 unidades más del mismo lote
  Entonces el carrito muestra una sola línea con 150 unidades
  Y si la suma superara el stock del lote, veo "Stock máximo disponible alcanzado ({{stock}} un.)." y la cantidad queda en la existencia

Escenario: El mismo producto con otro lote es otra línea
  Cuando agrego el mismo producto desde un lote distinto
  Entonces el carrito muestra una línea nueva

Escenario: Cantidades con decimales
  Cuando escribo Cantidad 0.5 y agrego
  Entonces el terminal lo acepta (solo exige que sea mayor que 0)
  # No verificado: cómo redondea la base de datos las cantidades con más de 2 decimales (RN-VEN-010, RN-VEN-020)

Escenario: El servidor vuelve a validar al cobrar
  Dado que otro cajero agotó el lote mientras tenía mi carrito armado
  Cuando intento cobrar
  Entonces el servidor responde "Venta.Stock" y veo "Ocurrió un error al procesar la venta."
  Y el carrito se conserva para corregirlo

Escenario: Un lote que venció antes de cobrar
  Dado que un lote del carrito venció mientras armaba la venta
  Cuando intento cobrar
  Entonces veo "Uno de los lotes está vencido y no se puede vender. Retíralo del carrito."

Escenario: Qué guarda el servidor de cada línea
  Dado que cobro la venta
  Entonces cada línea guarda la cantidad en unidades base, el precio fijado de esa fecha y el costo histórico vigente del precio de proveedor (0 si el producto no tiene precio de proveedor)
  Y el costo no se muestra en pantalla
  Y el total de la venta es la suma exacta de precio por cantidad de cada línea, sin descuentos

Escenario: No existen descuentos
  Dado que estoy armando una venta
  Entonces el terminal no ofrece ningún campo de descuento
  Y el único ajuste al monto a cobrar es la cobertura de seguro (ver HU-VEN-019)
```

> **Brecha conocida.** Casi todos los rechazos de negocio del cobro (`Venta.Stock`, `Venta.Cuarentena`, `Venta.Lote`, `Venta.Presentacion`, `Venta.Caja`, `Venta.Pagos`) se ven con el mensaje genérico «Ocurrió un error al procesar la venta.»; el cajero no sabe si falló el stock, la cuarentena, la caja o los pagos (01c2 §5.3 hallazgo 3). Los precios 0 no se rechazan en el servidor (solo el nulo): el terminal los bloquea con el aviso de «sin precio». Subtotal e IGV de la pantalla son `total ÷ 1,18` sin redondear antes de mostrarse y pueden diferir en un centavo del desglose guardado en la boleta (hallazgo 11; **No verificado** en ejecución).

### HU-VEN-004 — Vender por presentación (caja, blíster)

**Como** cajero **quiero** vender por presentación de venta (por ejemplo una caja) **para** cobrar el precio de la presentación sin convertir yo las unidades.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/nueva` — selector «Presentación» del formulario de agregado |
| **Endpoint(s)** | `GET /api/v1/ventas/presentaciones?productoId=`, `POST /api/v1/ventas` |
| **Permiso** | `Ventas:Crear` |
| **Reglas** | [RN-VEN-017, RN-VEN-018, RN-VEN-019, RN-VEN-020, RN-VEN-076](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-PRE-001 a RN-PRE-015](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) (catálogo de presentaciones) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Vender dos cajas
  Dado que "Paracetamol Demo 500 mg" tiene la presentación "Caja x10" a S/ 4.50 (10 unidades base)
  Y elegí un lote con 500 unidades
  Cuando elijo "Caja x10 (S/ 4.50)" en "Presentación", escribo Cantidad 2 y hago clic en "Agregar"
  Entonces la línea del carrito muestra "2 × Caja x10" y "S/ 0.45 c/u" (precio por unidad base)
  Y el total de la línea es "S/ 9.00"
  Y el servidor descuenta 20 unidades base del stock del lote y guarda ambas cantidades: 20 vendidas y 2 en presentación

Escenario: Cambiar entre presentación y unidad
  Dado que elegí "Caja x10" y escribí Cantidad 3
  Cuando elijo "Unidad" en "Presentación"
  Entonces la cantidad vuelve a 1 y el precio vuelve a S/ 0.50 (el del catálogo)

Escenario: El máximo se calcula en presentaciones enteras
  Dado que el lote tiene 35 unidades y la presentación es "Caja x10"
  Entonces el campo Cantidad no admite más de 3 cajas

Escenario: Los botones del carrito avanzan de a una presentación
  Dado que la línea es "2 × Caja x10"
  Cuando hago clic en "+"
  Entonces la cantidad en unidades base sube de 20 a 30 y la cantidad en presentaciones de 2 a 3

Escenario: Presentación con precio aunque el medicamento no tenga precio base
  Dado que el medicamento no tiene precio de venta base pero la presentación sí tiene precio
  Cuando vendo esa presentación
  Entonces el precio unitario es el precio de la presentación dividido entre sus unidades base
  Y el servidor no rechaza la línea por falta de precio

Escenario: La presentación debe ser del mismo producto
  Dado que una petición trae una presentación de otro producto o inexistente
  Cuando intento cobrar
  Entonces el servidor la rechaza con "Venta.Presentacion" y veo "Ocurrió un error al procesar la venta."

Escenario: Producto sin presentaciones
  Dado que el producto no tiene presentaciones de venta
  Entonces no aparece el selector "Presentación" y solo se vende por unidad

Escenario: Falla la carga de presentaciones
  Dado que la consulta de presentaciones falla
  Cuando elijo un lote
  Entonces veo "No se pudieron cargar las presentaciones del producto. Inténtalo nuevamente."
```

> **Brecha conocida.** Si el precio de la presentación no divide exacto entre sus unidades base (por ejemplo S/ 10.00 entre 3 unidades), el servidor no redondea el precio unitario y no se verificó cómo lo convierte la base de datos a 2 decimales (RN-VEN-020, 01c2 §5.1 puntos 1 y 2). Al repetir una presentación del mismo lote, la cantidad en presentaciones se suma sin tope y es el servidor quien detiene el exceso con `Venta.Stock`, mensaje genérico (19f §3.15; **No verificado** en ejecución).

### HU-VEN-005 — Ajustar, quitar y vaciar el carrito

**Como** cajero **quiero** corregir las cantidades, quitar líneas o vaciar el carrito **para** rectificar la venta antes de cobrar.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/nueva` — panel «Resumen de Venta» |
| **Endpoint(s)** | Ninguno (acciones locales) |
| **Permiso** | `Ventas:Crear` |
| **Reglas** | [RN-VEN-008, RN-VEN-014, RN-VEN-024, RN-VEN-045](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Carrito vacío
  Dado que no agregué nada
  Entonces veo "El carrito está vacío."
  Y no veo los totales
  Y "Procesar Venta" y "Guardar y Atender Siguiente" están deshabilitados

Escenario: Subir y bajar la cantidad
  Dado que la línea tiene 100 unidades de un lote con stock 500
  Cuando hago clic en "+"
  Entonces la cantidad pasa a 101 y el total de la línea y los totales se recalculan
  Y al hacer clic en "−" vuelve a 100

Escenario: No se baja del mínimo
  Dado que la línea tiene 1 unidad
  Cuando hago clic en "−"
  Entonces no pasa nada y no veo ningún aviso

Escenario: No se sube por encima del stock
  Dado que la línea ya tiene todo el stock del lote (500 unidades)
  Cuando hago clic en "+"
  Entonces veo "Stock máximo disponible alcanzado (500 un.)." y la cantidad queda igual

Escenario: Quitar una línea
  Cuando hago clic en la "×" de una línea
  Entonces la línea desaparece sin pedir confirmación

Escenario: Vaciar el carrito
  Dado que el carrito tiene líneas y escribí montos de pago
  Cuando hago clic en "Vaciar"
  Entonces se borran todas las líneas sin pedir confirmación
  Y los pagos vuelven a un único pago en "Efectivo" por 0
  Y el cliente, la receta y la cobertura de seguro elegidos no se descartan

Escenario: La cantidad no se escribe en el carrito
  Dado que quiero cambiar la cantidad de una línea a un valor exacto
  Entonces debo usar "+" y "−" o quitar la línea y agregarla de nuevo, porque el carrito no tiene campo para escribir la cantidad
```

### HU-VEN-006 — Recuperar la venta en curso tras recargar (borrador local)

**Como** cajero **quiero** que el terminal recuerde mi carrito si recargo la página **para** no perder la venta que estaba armando.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/nueva` (comportamiento automático) |
| **Endpoint(s)** | `GET /api/v1/pacientes/{id}` (solo para restaurar el cliente) |
| **Permiso** | `Ventas:Crear`; `Pacientes:Leer` para restaurar el cliente |
| **Reglas** | [RN-VEN-045, RN-VEN-053](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Recargar la página con una venta a medias
  Dado que tengo 2 líneas en el carrito, un cliente elegido y un pago por Yape / Plin con número de operación
  Cuando recargo el navegador o cierro y vuelvo a abrir la pestaña dentro de las 12 horas
  Entonces el terminal restaura el carrito, los pagos, el cliente, la receta elegida y el monto de aseguradora
  Y la casilla "Receta verificada" no se restaura: debo marcarla de nuevo

Escenario: El borrador caduca
  Dado que pasaron más de 12 horas desde el último cambio
  Cuando abro el terminal
  Entonces el borrador se descarta y empiezo con el terminal vacío

Escenario: Qué se guarda y dónde
  Dado que tengo un carrito en curso
  Entonces el borrador queda en el navegador de este equipo, con una clave por usuario
  Y guarda solo identificadores del cliente y de la receta, no datos personales

Escenario: Se borra al cobrar con éxito
  Dado que cobré la venta
  Entonces el borrador se elimina

Escenario: Se borra al cerrar sesión o al entrar otro usuario
  Cuando cierro sesión, o cuando otro usuario inicia sesión en el mismo navegador
  Entonces el borrador de este usuario se elimina o los de los demás usuarios se eliminan

Escenario: Al retomar una venta en espera solo sobreviven los pagos no efectivo
  Dado que retomo una venta en espera y había escrito un pago con tarjeta con monto y número de operación, y un monto en efectivo
  Cuando recargo el navegador
  Entonces se recupera el pago con tarjeta y no el monto en efectivo escrito
```

> **Brecha conocida.** El borrador no vuelve a validar los lotes: las líneas conservan la existencia con que se guardaron. `saveDraft` y `restoreDraft` leen y escriben `localStorage` sin `try/catch`, así que un navegador que bloquee el almacenamiento podría fallar (**No verificado**, 19f §3.9). Un borrador antiguo con aseguradora puede reducir el «TOTAL A PAGAR» sin forma visible de quitarla, porque el bloque de seguro está oculto (HU-VEN-019).

### HU-VEN-007 — Elegir al cliente de la venta

**Como** cajero **quiero** vender a un cliente identificado o al cliente eventual **para** que la boleta salga a nombre correcto y poder usar las recetas del cliente.

| | |
|---|---|
| **Módulo** | Ventas / Pacientes |
| **Pantalla** | `/pos/nueva` — panel «Datos del Cliente» |
| **Endpoint(s)** | `GET /api/v1/pacientes` (búsqueda) |
| **Permiso** | `Pacientes:Leer` (búsqueda); `Ventas:Crear` |
| **Reglas** | [RN-VEN-025, RN-VEN-026, RN-VEN-027, RN-CMP-005](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Vender sin identificar al cliente
  Dado que el carrito solo tiene productos de venta libre
  Cuando no elijo ningún cliente
  Entonces bajo el campo veo "Cliente Eventual (Público General)"
  Y la boleta sale a nombre de "CLIENTE EVENTUAL" con documento "00000000"
  Y en el listado y el detalle de ventas el cliente aparece como "Público General"

Escenario: Buscar al cliente por documento
  Cuando escribo "45781236" (solo dígitos) en "Buscar paciente por nombre o documento..."
  Entonces tras unos 300 milisegundos el sistema busca por documento y muestra hasta 20 opciones con la forma "{nombre} · DNI 45781236"

Escenario: Buscar al cliente por nombre
  Cuando escribo "Quispe" en el mismo campo
  Entonces el sistema busca por nombre y apellidos

Escenario: Elegir al cliente
  Cuando elijo una opción
  Entonces bajo el campo veo su documento y aparece el selector "Receta médica"
  Y se cargan sus recetas registradas

Escenario: La búsqueda no encuentra a nadie
  Cuando escribo un texto sin coincidencias
  Entonces la lista queda vacía, sin la opción "Sin resultados"

Escenario: Cambiar de cliente
  Dado que elegí un cliente y una receta
  Cuando escribo otro texto en el campo
  Entonces se descartan el cliente y su receta elegida

Escenario: El cliente no es obligatorio en el sistema
  Dado que vendo solo productos de venta libre
  Entonces el terminal no exige cliente
  Y no existe un monto a partir del cual se obligue a identificar al comprador de la boleta

Escenario: El cliente es indispensable para productos controlados
  Dado que el carrito tiene "Clonazepam Demo 2 mg" (Receta Retenida)
  Entonces hace falta elegir un cliente para poder elegir una receta registrada (ver HU-VEN-008)

Escenario: El cliente ya no existe
  Dado que la petición trae un cliente que se eliminó mientras armaba la venta
  Cuando intento cobrar
  Entonces el servidor responde 404 "Paciente.NotFound" y veo "Ocurrió un error al procesar la venta."

Escenario: Al actualizar una venta en espera el cliente enviado reemplaza al guardado
  Dado que retomo una venta en espera con un cliente
  Cuando quito la selección y guardo de nuevo
  Entonces el cliente de la venta queda en blanco
  Y el terminal siempre envía el cliente que tengo seleccionado
```

> **Brecha conocida.** No hay botón para quitar al cliente: vaciar por completo el campo no lo descarta (el cliente solo se descarta al escribir otro texto o al limpiar la venta; **No verificado** en ejecución). El terminal no permite registrar un paciente nuevo. Si se omite `ClienteId` al actualizar los detalles o cobrar con líneas reenviadas, el servidor borra el cliente, y al cobrar sin reenviar líneas lo ignora (01c2 §5.3 hallazgo 9). Desde el commit `27597cd` un paciente con ventas no eliminadas ya no se puede eliminar (409 `Paciente.InUse`, RN-PAC-010 en 01b).

## 4. Recetas en la venta

### HU-VEN-008 — Vender productos que exigen receta registrada

**Como** cajero **quiero** que el terminal me indique cuándo hace falta una receta y cuál **para** vender receta simple, receta retenida y estupefacientes cumpliendo la norma.

| | |
|---|---|
| **Módulo** | Ventas / Recetas |
| **Pantalla** | `/pos/nueva` — selector «Receta médica» del panel «Datos del Cliente» |
| **Endpoint(s)** | `GET /api/v1/recetas?clienteId=…` (lista de recetas del cliente), `POST /api/v1/ventas` |
| **Permiso** | `Recetas:Leer` (lista); `Recetas:Crear` (enlace «Registrar receta»); `Ventas:Crear` |
| **Reglas** | [RN-VEN-026, RN-VEN-028, RN-VEN-029, RN-VEN-030, RN-VEN-031, RN-VEN-032, RN-VEN-036, RN-VEN-037, RN-VEN-077](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Vender un producto de receta retenida con receta registrada que tiene número
  Dado que el cliente "Rosa Quispe Demo" tiene una receta registrada con N.° "000123"
  Y el carrito tiene 2 unidades de "Clonazepam Demo 2 mg" (Receta Retenida) por S/ 5.00
  Cuando elijo al cliente y en "Receta médica" elijo "{fecha} · {médico} · N.° 000123"
  Entonces no veo ningún aviso de receta
  Y "Procesar Venta" queda habilitado una vez cubierto el pago

Escenario: Producto controlado sin receta elegida
  Dado que el carrito tiene un producto de receta retenida o estupefaciente
  Y la opción de receta está en "Sin receta"
  Entonces veo "Hay productos de receta retenida o estupefacientes: selecciona al cliente y una receta registrada que tenga número de receta."
  Y "Procesar Venta" está bloqueado

Escenario: Receta elegida sin número
  Dado que el carrito tiene un producto controlado
  Y elegí una receta que no tiene número de receta
  Entonces veo "La receta seleccionada no tiene número de receta: agrégalo en la receta para vender productos de receta retenida o estupefacientes."
  Y "Procesar Venta" está bloqueado

Escenario: Receta simple en una empresa que exige registrar la receta
  Dado que la empresa tiene activado "exigir registro de receta"
  Y el carrito tiene "Amoxicilina Demo 500 mg" (Receta Simple) sin receta elegida
  Entonces veo "Hay productos que requieren una receta registrada: selecciona al cliente y su receta antes de cobrar."
  Y "Procesar Venta" está bloqueado

Escenario: Venta libre no exige receta
  Dado que el carrito solo tiene "Paracetamol Demo 500 mg" (Venta Libre)
  Entonces el terminal no muestra aviso de receta y no la exige

Escenario: Qué ofrece el selector de recetas
  Dado que elegí un cliente
  Entonces el selector "Receta médica" tiene "Sin receta" como primera opción
  Y después una opción por receta registrada del cliente, hasta 50, con la forma "{fecha de expedición dd/MM/yyyy} · {médico} · N.° {número}"
  Y bajo el campo leo "Receta retenida y estupefacientes siempre exigen una receta registrada con su número."

Escenario: La receta elegida se adjunta a todas las líneas
  Dado que el carrito tiene una línea de venta libre y una de receta retenida
  Cuando elijo una receta del cliente
  Entonces el terminal envía esa misma receta en todas las líneas, también en la de venta libre
  Y el servidor comprueba que la receta exista y sea del cliente para cada línea

Escenario: Registrar una receta nueva sin salir del terminal
  Dado que tengo el permiso "Recetas:Crear" y elegí un cliente
  Cuando hago clic en "Registrar receta"
  Entonces se abre "/recetas-medicas/nueva" en otra pestaña
  Y al volver a la pestaña del terminal las recetas del cliente se recargan y la nueva aparece en la lista

Escenario: La receta no existe (el servidor)
  Dado que la receta se eliminó mientras armaba la venta
  Cuando intento cobrar
  Entonces veo "La receta médica seleccionada no existe o fue eliminada."

Escenario: La receta es de otro paciente (el servidor)
  Dado que la petición trae una receta de otro paciente o una venta sin cliente
  Cuando intento cobrar
  Entonces veo "La receta médica no pertenece al cliente de la venta."

Escenario: Falta la receta o su número (el servidor)
  Dado que la petición trae un producto controlado sin receta, o con una receta sin número
  Cuando intento cobrar
  Entonces veo "Uno de los productos requiere receta médica. Selecciona el cliente y una de sus recetas registradas." si falta la receta
  Y veo "La receta seleccionada no tiene número de receta. Agrégalo en la receta para vender productos de receta retenida o estupefacientes." si la receta no tiene número

Escenario: Dejar la venta en espera no exige receta
  Dado que el carrito tiene un producto controlado y no elegí receta
  Cuando hago clic en "Guardar y Atender Siguiente"
  Entonces la venta se guarda en espera sin exigir receta ni número
```

> **Brecha conocida.** El terminal **no tiene campo para escribir el número de la receta**: el número es un dato de la receta registrada y debe agregarse en su propia pantalla. La política «exigir registro de receta» de la empresa solo se cambia directamente en la base de datos; no hay endpoint ni pantalla (RN-EMP-016, ver [HU-VEN-009](#hu-ven-009--confirmar-receta-verificada-en-empresas-flexibles)). Las líneas de un borrador antiguo sin condición de venta guardada no las bloquea el terminal; decide el servidor (19f §3.5). El terminal adjunta una sola receta a todo el carrito aunque la API admite una por línea (01c2 hallazgo 14).

### HU-VEN-009 — Confirmar «Receta verificada» en empresas flexibles

**Como** cajero de una empresa que no exige registrar la receta **quiero** confirmar que revisé la receta de papel **para** vender receta simple dejando constancia de quién la verificó.

| | |
|---|---|
| **Módulo** | Ventas / Recetas |
| **Pantalla** | `/pos/nueva` — casilla «Receta verificada»; `/pos/:id` — «Verificación de receta» |
| **Endpoint(s)** | `POST /api/v1/ventas`, `PUT /api/v1/ventas/{id}/completar`, `GET /api/v1/ventas/{id}` |
| **Permiso** | `Ventas:Crear` o `Ventas:Actualizar` (al cobrar una venta en espera); `Ventas:Leer` |
| **Reglas** | [RN-VEN-028, RN-VEN-029, RN-VEN-034, RN-VEN-035, RN-VEN-037](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-EMP-016](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Vender receta simple sin receta registrada marcando la casilla
  Dado que la empresa no exige registro de receta (valor con el que nace toda empresa nueva)
  Y el carrito tiene "Amoxicilina Demo 500 mg" (Receta Simple) sin receta elegida y sin productos controlados
  Entonces veo la casilla "Receta verificada" con el texto "Confirmas que revisaste la receta física del cliente. Quedará registrado a tu nombre."
  Y "Procesar Venta" está deshabilitado hasta que la marque
  Cuando la marco y cobro
  Entonces la venta se registra y guarda quién la verificó y cuándo

Escenario: El detalle de la venta muestra la verificación
  Dado que la venta se cobró con "Receta verificada"
  Cuando abro su detalle en "/pos/:id"
  Entonces veo "Verificación de receta": "Receta verificada por Carlos Cajero Demo el 26/10/2026 10:15"

Escenario: Con receta registrada no se pide confirmación
  Dado que elegí una receta registrada del cliente
  Entonces la casilla "Receta verificada" no aparece y no se guarda verificación

Escenario: Con un producto controlado la casilla no sirve
  Dado que el carrito tiene un producto de receta retenida o estupefaciente
  Entonces la casilla no aparece; sigo necesitando una receta registrada con número

Escenario: La casilla se desmarca sola
  Dado que marqué la casilla
  Cuando el carrito deja de necesitarla (por ejemplo, quito la línea de receta simple)
  Entonces la casilla se desmarca y no se guarda en el borrador

Escenario: Dejar la venta en espera no pide la confirmación
  Cuando hago clic en "Guardar y Atender Siguiente" sin marcar la casilla
  Entonces la venta se guarda en espera; la confirmación se pide al cobrar

Escenario: Quien cobra es quien verifica
  Dado que otro cajero dejó la venta en espera
  Cuando yo la cobro y marco "Receta verificada"
  Entonces la verificación queda a mi nombre

Escenario: El servidor exige la confirmación que el terminal no mostró
  Dado que el servidor considera que hace falta la confirmación y el terminal no mostraba la casilla
  Cuando intento cobrar
  Entonces veo "Marca "Receta verificada" para confirmar que revisaste la receta del cliente antes de cobrar."
  Y el terminal pasa a modo flexible y muestra la casilla

Escenario: Si ninguna línea la necesita, el valor enviado se ignora
  Dado que el carrito no necesita verificación
  Entonces el servidor no guarda quién ni cuándo, aunque la petición traiga la marca
```

> **Brecha conocida.** Solo se guarda **quién y cuándo** verificó la receta; el número de la receta de papel no se guarda en ninguna parte (RN-VEN-035). La opción de la empresa que activa o apaga esta confirmación no se puede cambiar desde ninguna pantalla (RN-EMP-016); por eso el estado es «Implementada» para el flujo del cajero pero la política no tiene gestión.

### HU-VEN-010 — Respetar el máximo de dispensaciones de una receta

**Como** cajero **quiero** que el sistema impida vender de nuevo con una receta ya agotada **para** no dispensar más veces de las permitidas.

| | |
|---|---|
| **Módulo** | Ventas / Recetas |
| **Pantalla** | `/pos/nueva` (el rechazo se ve al cobrar) |
| **Endpoint(s)** | `POST /api/v1/ventas`, `PUT /api/v1/ventas/{id}/completar` |
| **Permiso** | `Ventas:Crear` / `Ventas:Actualizar` |
| **Reglas** | [RN-VEN-033](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-REC-031](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-DEV-026](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Esquema del escenario: Cuántas dispensaciones admite una receta
  Dado una receta cuyo producto es de condición <condición>
  Entonces el máximo de ventas que la pueden usar es <máximo>

  Ejemplos:
    | condición                       | máximo                                  |
    | Venta Libre (OTC)               | sin límite (no cuenta ni consume)       |
    | Receta Simple                   | 1 más las repeticiones máximas          |
    | Receta Retenida                 | 1                                       |
    | Estupefaciente                  | 1                                       |

Escenario: Segunda venta con una receta retenida
  Dado que ya vendí "Clonazepam Demo 2 mg" con la receta N.° "000123"
  Cuando intento cobrar otra venta con producto de receta retenida y la misma receta
  Entonces veo "La receta médica ya fue dispensada el máximo de veces permitido."
  Y no se guarda la venta

Escenario: Receta simple con 2 repeticiones máximas
  Dado que la receta simple permite 2 repeticiones máximas
  Entonces admite 3 ventas distintas y la cuarta se rechaza con "La receta médica ya fue dispensada el máximo de veces permitido."

Escenario: Varias líneas de la misma receta son una sola dispensación
  Dado que una venta tiene dos líneas de productos de receta con la misma receta
  Entonces esa venta consume una sola dispensación

Escenario: Una venta en espera ya cuenta como dispensación
  Dado que guardé en espera una venta con una receta retenida
  Cuando otro cajero intenta cobrar otra venta con esa misma receta
  Entonces la segunda se rechaza porque la venta en espera ya consumió la dispensación

Escenario: Anular la venta libera la dispensación
  Dado que anulé la venta que usó la receta
  Cuando vendo otra vez con esa receta
  Entonces el sistema lo permite, porque el conteo ignora las ventas anuladas

Escenario: Devolver no libera la dispensación
  Dado que la venta tuvo una devolución parcial o total
  Entonces la venta sigue contando como dispensación
  Y como una venta con devoluciones no se puede anular, la dispensación no se libera

Escenario: Las líneas de venta libre con receta no cuentan
  Dado que una línea de venta libre lleva una receta adjunta
  Entonces esa línea no consume ni verifica dispensaciones

Escenario: Actualizar una venta en espera no se cuenta contra sí misma
  Dado que reemplazo las líneas de una venta en espera que ya usaba la receta
  Entonces sus líneas anteriores no cuentan para el máximo de esa misma venta
```

> **Brecha conocida.** El terminal no muestra cuántas dispensaciones le quedan a la receta: el rechazo llega recién al cobrar. Las recetas no se marcan «usadas» en ninguna columna; el conteo se recalcula con las ventas (RN-VEN-033). Al cobrar una venta en espera **sin reenviar las líneas** (solo por API) no se vuelve a comprobar el máximo (RN-VEN-016).

### HU-VEN-011 — Agregar al carrito los medicamentos de una receta escaneada

**Como** cajero **quiero** fotografiar la receta del cliente y agregar al carrito los medicamentos que indica **para** armar la venta más rápido.

| | |
|---|---|
| **Módulo** | Ventas / Recetas con IA |
| **Pantalla** | `/pos/nueva` — pestaña «Receta» |
| **Endpoint(s)** | `POST /api/v1/recetas/analizar` (multipart), `GET /api/v1/ventas/lotes-disponibles?productoNombre=` |
| **Permiso** | `Recetas:Analizar` (la pestaña solo existe con este permiso); `Ventas:Crear` |
| **Reglas** | [RN-VEN-036, RN-VEN-075](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: La pestaña solo aparece con el permiso
  Dado que no tengo el permiso "Recetas:Analizar"
  Entonces el terminal no muestra las pestañas "Producto" y "Receta", solo la búsqueda de productos

Escenario: Interpretar una receta
  Dado que tengo el permiso "Recetas:Analizar"
  Cuando abro la pestaña "Receta", elijo una imagen con "Tomar foto" o "Galería" (o la arrastro) y hago clic en "Interpretar receta"
  Entonces veo "Interpretando..." mientras se analiza
  Y luego el banner "{n} medicamento(s) detectado(s) en la receta" y una fila por cada medicamento de tipo receta

Escenario: Cada fila indica confianza, stock y validación DIGEMID
  Dado que se detectó un medicamento
  Entonces veo su confianza "Alta" (0,85 o más), "Media" (0,65 o más) o "Baja"
  Y "En stock" o "Sin stock"
  Y la insignia "DIGEMID" si está validado

Escenario: Agregar un medicamento detectado
  Dado que el medicamento tiene stock
  Cuando hago clic en el botón "+" de su fila
  Entonces se agrega al carrito el primer lote disponible (el que vence antes), sin presentación y a precio de catálogo
  Y la cantidad agregada es la sugerida o 1, sin pasar de la existencia del lote
  Y el botón cambia a "✓" y no se vuelve a agregar

Escenario: El medicamento no tiene stock en mi sucursal
  Cuando intento agregarlo y no hay lotes
  Entonces veo "El medicamento detectado no tiene stock en esta sucursal."

Escenario: El medicamento no tiene precio
  Cuando el primer lote no tiene precio de catálogo
  Entonces veo "Este producto no tiene precio de venta configurado en el catálogo. Pide a un administrador que lo registre."

Escenario: No se detectó nada
  Entonces veo "No se detectaron medicamentos en la imagen. Intenta con una foto más nítida."

Escenario: Falla el análisis
  Entonces veo "Error al interpretar la receta. Intenta nuevamente."

Escenario: Escanear no asocia ninguna receta registrada a la venta
  Dado que agregué medicamentos desde el escaneo
  Entonces para vender productos de receta sigo necesitando elegir al cliente y su receta registrada (ver HU-VEN-008)
```

> **Brecha conocida.** El número `n` del banner cuenta todos los ítems devueltos, también los de tipo «Recomendacion», que el terminal no lista; tampoco muestra los equivalentes terapéuticos. No hay botón «Limpiar» en esta pestaña (19f §3.6). El análisis de recetas con IA como tal pertenece a la parte B.

## 5. Pagos, cobro y comprobante

### HU-VEN-012 — Cobrar en efectivo y entregar el vuelto

**Como** cajero **quiero** registrar el efectivo que me entrega el cliente y ver cuánto vuelto devolverle **para** cobrar correctamente y que mi caja cuadre.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/nueva` — panel «Método de Pago» (ficha «Efectivo», campo «Recibido (S/)») |
| **Endpoint(s)** | `POST /api/v1/ventas` |
| **Permiso** | `Ventas:Crear` |
| **Reglas** | [RN-VEN-039, RN-VEN-041, RN-VEN-043, RN-VEN-044, RN-VEN-046, RN-VEN-081](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: El cliente paga con más efectivo y recibe vuelto
  Dado que el carrito suma "TOTAL A PAGAR:" S/ 50.00
  Cuando elijo "Efectivo" y escribo "Recibido (S/)" 60.00
  Entonces veo "Vuelto: S/ 10.00" junto al campo
  Y en los totales veo "Vuelto a Entregar:" S/ 10.00
  Y "Procesar Venta" queda habilitado
  Y en el diálogo de confirmación veo "Vuelto Entregado:" S/ 10.00

Escenario: El cliente paga el monto exacto
  Cuando escribo "Recibido (S/)" 50.00
  Entonces el monto pendiente es 0, no hay vuelto y "Procesar Venta" queda habilitado

Escenario: El efectivo no alcanza
  Cuando escribo "Recibido (S/)" 40.00
  Entonces en los totales veo "Monto Pendiente:" S/ 10.00
  Y "Procesar Venta" está deshabilitado
  Y no aparece ningún mensaje de "monto insuficiente"

Escenario: El pago inicial
  Dado que agregué productos y no toqué los pagos
  Entonces hay un único pago en "Efectivo" por 0
  Y un monto de 0 no cubre el total, de modo que no se puede cobrar

Escenario: Qué se registra del pago en efectivo
  Dado que cobré con "Recibido (S/)" 60.00 un total de S/ 50.00
  Entonces el pago queda registrado por 60.00 (lo que entregó el cliente) y no por 50.00
  Y el vuelto de 10.00 no se guarda: se recalcula como suma de pagos menos el monto a pagar
  Y en el detalle de la venta veo "Vuelto:" S/ 10.00

Escenario: La suma de pagos debe cubrir el monto a pagar sin tolerancia
  Dado que una petición llega con pagos que suman S/ 49.99 para un total de S/ 50.00
  Entonces el servidor la rechaza ("Venta.Pagos") y veo "Ocurrió un error al procesar la venta."

Escenario: El vuelto sale del cajón
  Dado que el vuelto es mayor que 0
  Entonces debe haber al menos un pago en efectivo, y el vuelto no puede superar el efectivo entregado
  Y si lo supera el botón "Procesar Venta" queda deshabilitado
  Y si la petición llega al servidor veo "El vuelto no puede ser mayor que el efectivo recibido. Ajusta los montos de pago."

Escenario: Cada pago debe ser mayor que 0
  Dado que una petición trae un pago de monto 0 o negativo
  Entonces el servidor la rechaza con "CreateVentaCommand.Pagos[i].MontoPagado" y veo "Ocurrió un error al procesar la venta."

Escenario: Una venta cobrada pasa a "Completada"
  Cuando el cobro termina bien
  Entonces la venta queda en estado "Completada" y el instante del pago es el del cobro (hora del servidor)
  # No verificado: cómo convierte la base de datos un monto con más de 2 decimales (RN-VEN-041, RN-VEN-043)
```

> **Brecha conocida.** Si el servidor rechaza la suma de pagos o un vuelto sin efectivo, el cajero ve el mensaje genérico (`Venta.Pagos` no está traducido salvo `VueltoExcedeEfectivo`); el terminal lo evita deshabilitando el botón. Los textos `pos.messages.montoInsuficiente`, `sumaPagosInsuficiente` y `vueltoSinEfectivoError` existen en `pos/es.json` pero ningún componente los usa (01c2 hallazgo 4). El monto de un pago no se reajusta solo cuando cambia el carrito (19f §3.7; **No verificado** en ejecución).

### HU-VEN-013 — Cobrar con Yape / Plin, tarjeta o transferencia

**Como** cajero **quiero** cobrar con medios digitales y anotar el número de operación **para** que el pago quede conciliable.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/nueva` — panel «Método de Pago» (fichas, «Monto (S/)», «Nro Operación») |
| **Endpoint(s)** | `POST /api/v1/ventas` |
| **Permiso** | `Ventas:Crear` |
| **Reglas** | [RN-VEN-040, RN-VEN-042, RN-VEN-081](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Cobrar con Yape / Plin
  Dado que el carrito suma S/ 50.00
  Cuando hago clic en la ficha "Yape / Plin"
  Entonces queda un único pago por Yape / Plin con monto S/ 50.00
  Y aparecen "Monto (S/)" y "Nro Operación"
  Cuando escribo el número de operación "YP-778812"
  Entonces "Procesar Venta" queda habilitado

Escenario: Falta el número de operación
  Dado que elegí "Tarjeta Débito/Crédito" y no escribí "Nro Operación"
  Entonces veo, en rojo bajo los pagos, "Ingresa el N° de operación para poder cobrar."
  Y "Procesar Venta" está deshabilitado

Escenario: Un número de solo espacios cuenta como faltante
  Cuando escribo solo espacios en "Nro Operación"
  Entonces el terminal lo trata como vacío y no deja cobrar

Escenario: Límite de largo y recorte
  Entonces el campo "Nro Operación" corta en 100 caracteres
  Y el terminal recorta los espacios al inicio y al final antes de enviar

Escenario: El efectivo no pide número de operación
  Cuando elijo "Efectivo"
  Entonces no aparece el campo "Nro Operación"

Esquema del escenario: Métodos de pago disponibles
  Entonces la ficha <ficha> corresponde al método <método> y <pide>

  Ejemplos:
    | ficha                    | método         | pide                          |
    | Efectivo                 | Efectivo       | no pide número de operación   |
    | Yape / Plin              | YapePlin       | pide número de operación      |
    | Tarjeta Débito/Crédito   | Tarjeta        | pide número de operación      |
    | Transferencia            | Transferencia  | pide número de operación      |

Escenario: El servidor repite la exigencia
  Dado que una petición trae un pago no efectivo sin número de operación
  Cuando intento cobrar
  Entonces veo "Ingresa el número de operación de cada pago que no sea en efectivo."

Escenario: Un número de operación de más de 100 caracteres (el servidor)
  Entonces el servidor lo rechaza y veo "Ocurrió un error al procesar la venta."

Escenario: El mismo número en dos ventas
  Dado que ya registré el número "YP-778812" en otra venta
  Cuando lo uso en una venta nueva
  Entonces el sistema lo acepta
```

> **Brecha conocida.** El número de operación no es único ni se valida su formato (ni en el código ni en la base de datos), y el sistema no distingue tarjeta de débito de tarjeta de crédito (RN-VEN-040, RN-VEN-042; 01c2 hallazgo 18). La base de datos no impone el método ni la exigencia del número: viven solo en el código (RN-VEN-081).

### HU-VEN-014 — Dividir el pago en varios métodos

**Como** cajero **quiero** cobrar una misma venta con varios métodos **para** atender al cliente que paga parte en efectivo y parte con Yape o tarjeta.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/nueva` — «+ Dividir en varios métodos de pago», «Agregar Método de Pago», «Volver a pago simple» |
| **Endpoint(s)** | `POST /api/v1/ventas` |
| **Permiso** | `Ventas:Crear` |
| **Reglas** | [RN-VEN-039, RN-VEN-041, RN-VEN-042, RN-VEN-043, RN-VEN-044, RN-VEN-045](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Pagar S/ 50.00 con Yape / Plin y efectivo
  Dado que el total es S/ 50.00
  Cuando hago clic en "+ Dividir en varios métodos de pago"
  Y registro "Yape / Plin" por 40.00 con número de operación "YP-778812"
  Y "Efectivo" por 15.00
  Entonces la suma de pagos es S/ 55.00 y veo "Vuelto a Entregar:" S/ 5.00
  Y como el vuelto (5.00) no supera el efectivo (15.00), "Procesar Venta" queda habilitado
  Y el efectivo neto que entra al cajón es 10.00

Escenario: El vuelto supera el efectivo entregado
  Dado que el total es S/ 50.00
  Cuando registro "Tarjeta Débito/Crédito" por 60.00 con número de operación y "Efectivo" por 5.00
  Entonces la suma es 65.00 y el vuelto 15.00 supera el efectivo 5.00
  Y "Procesar Venta" queda deshabilitado sin ningún mensaje
  Y si la petición llega al servidor veo "El vuelto no puede ser mayor que el efectivo recibido. Ajusta los montos de pago."
  # En consecuencia, los pagos no efectivo no pueden sumar más que el monto a pagar

Escenario: Agregar otro método
  Dado que estoy en el modo dividido
  Cuando hago clic en "Agregar Método de Pago"
  Entonces se agrega una fila con el primer método que aún no usé (o "Efectivo" si ya usé los cuatro) y con el monto pendiente

Escenario: No se repite un método
  Dado que ya tengo una fila "Efectivo" de 20.00
  Cuando cambio otra fila a "Efectivo"
  Entonces las dos filas se fusionan en una, se suman los montos y se conserva el primer número de operación no vacío

Escenario: Quitar un pago
  Cuando hago clic en la "×" de una fila
  Entonces desaparece, salvo que sea el único pago

Escenario: Volver a pago simple
  Cuando hago clic en "Volver a pago simple"
  Entonces queda un único pago en "Efectivo" por el total a pagar y se pierden los demás pagos

Escenario: Un pago no efectivo sin número se marca
  Cuando una fila no efectivo no tiene número de operación
  Entonces ese campo se marca en rojo y veo "Ingresa el N° de operación para poder cobrar."

Escenario: La API acepta cualquier cantidad de pagos
  Dado que llamo a la API con dos pagos del mismo método
  Entonces el servidor los acepta mientras cada uno sea mayor que 0 y la suma cubra el monto a pagar
  Y no hay máximo de cantidad de pagos

Escenario: Cobrar sin pagos
  Dado que la petición no trae pagos y el estado pedido no es "Pendiente"
  Entonces el servidor la rechaza con "CreateVentaCommand.Pagos"
  Y veo "Ocurrió un error al procesar la venta."
```

### HU-VEN-015 — Confirmar y procesar la venta

**Como** cajero **quiero** revisar un resumen y confirmar el cobro **para** registrar la venta, descontar el stock y emitir la boleta en un solo paso.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/nueva` — botón «Procesar Venta»; diálogo «Confirmar Procesamiento de Venta» |
| **Endpoint(s)** | `POST /api/v1/ventas` |
| **Permiso** | `Ventas:Crear` |
| **Reglas** | [RN-VEN-002, RN-VEN-015, RN-VEN-021, RN-VEN-049, RN-VEN-050, RN-VEN-051, RN-VEN-057, RN-VEN-058, RN-VEN-060, RN-VEN-061, RN-VEN-079, RN-VEN-080, RN-VEN-081, RN-VEN-082, RN-VEN-083](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Procesar una venta correcta
  Dado que el carrito tiene 100 unidades de "Paracetamol Demo 500 mg" (S/ 50.00), pagadas en efectivo con 60.00, y tengo mi caja abierta
  Cuando hago clic en "Procesar Venta"
  Entonces se abre "Confirmar Procesamiento de Venta" con el texto "Verifica los detalles antes de registrar la transacción."
  Y veo "Cliente:" (el nombre o "Cliente Eventual (Público General)"), "Cantidad de Ítems:" (el número de líneas), "Método de Pago:", "Total a Pagar:" S/ 50.00 y "Vuelto Entregado:" S/ 10.00
  Cuando hago clic en "Confirmar y Procesar Venta"
  Entonces el servidor registra la venta como "Completada", descuenta 100 unidades base del stock del lote y emite la boleta
  Y se abre el diálogo "Venta Procesada Exitosamente"
  Y no aparece ningún aviso adicional (el diálogo es la confirmación)

Escenario: Cancelar la confirmación
  Cuando hago clic en "Cancelar" o cierro el diálogo con Esc
  Entonces vuelvo al terminal sin cambios y no se envía nada

Escenario: "Vuelto Entregado" solo aparece si se pagó de más
  Dado que pagué el monto exacto
  Entonces el diálogo de confirmación no muestra "Vuelto Entregado:"

Escenario: Todo se guarda en una sola operación
  Dado que cualquier paso del cobro falla (stock, receta, RUC, serie, pagos)
  Entonces no se guarda nada de la venta y el stock descontado en memoria se descarta
  # La única escritura inmediata es el aumento del correlativo de la serie, dentro de la misma transacción

Escenario: Quién registra y cuándo
  Entonces la venta queda en mi sucursal base, a nombre del empleado de mi sesión, con la fecha y hora del servidor en UTC
  Y el cliente no puede enviar la fecha

Escenario: Se registra un evento para integraciones futuras
  Cuando el cobro termina bien
  Entonces se guarda el evento "VentaCompletadaEvent" en la bandeja de salida y ningún proceso lo consume hoy

Escenario: Cobrar no deja evento de auditoría de seguridad
  Entonces la trazabilidad es el usuario y la fecha de creación y modificación de las filas, no la tabla de auditoría

Escenario: Falla el cobro y el carrito se conserva
  Dado que el servidor rechaza el cobro
  Entonces veo el mensaje que corresponde al código de error (ver las historias de receta, pagos y comprobante)
  Y para cualquier código no traducido, o si no hay red, veo "Ocurrió un error al procesar la venta."
  Y el carrito y los pagos se conservan para corregir y reintentar

Escenario: Otra operación modificó el stock o la venta
  Dado que otro cajero vendió el mismo lote al mismo tiempo
  Cuando mi cobro intenta guardarse
  Entonces el servidor responde 409 y veo "Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo."
  # No verificado en ejecución: se deduce de la versión de fila del stock y de la venta

Escenario: Restricciones de la base de datos que respaldan el cobro
  Dado que una petición intenta guardar un estado de venta fuera de Completada, Anulada, Devuelta o Pendiente, un total negativo, una cantidad de línea 0 o negativa, un pago de monto 0 o negativo, o una referencia a un registro inexistente
  Entonces la base de datos lo rechaza con 400 "Db.ValorInvalido" o 409 "Db.ReferenciaInvalida"
  Y el cajero ve "Ocurrió un error al procesar la venta." porque esos códigos no están en el mapa de errores del cobro

Escenario: Un doble clic no cobra dos veces
  Cuando hago clic dos veces en "Confirmar y Procesar Venta"
  Entonces el diálogo se cierra al instante y el botón "Procesar Venta" queda en "procesando", lo que impide un segundo envío

Escenario: La API acepta dos estados al crear
  Dado que llamo a la API con estado "Completada" y al menos un pago
  Entonces la venta se cobra
  Y con estado "Pendiente" y sin pagos queda en espera (ver HU-VEN-020)
  Y con estado "Pendiente" y con pagos se cobra de inmediato como "Completada"
  Y "Anulada" o "Devuelta" con pagos se rechazan y "Completada" sin pagos lo rechaza el validador
```

> **Brecha conocida.** La API responde **200** (no 201) con `{ ventaId, comprobante }` al crear y cobrar (01c2 §2). Las restricciones de la base (estado válido, monto total no negativo, claves foráneas, versión de fila) no tienen traducción en el mapa de cobro: se ven con el mensaje genérico (RN-VEN-079 a RN-VEN-081). Los índices de rendimiento de ventas, detalle y pagos no tienen efecto visible (RN-VEN-082). Los guards de la entidad `Venta` que ninguna ruta alcanza (RN-VEN-083) no producen mensajes.

### HU-VEN-016 — Cobrar solo con RUC de la empresa y serie de boleta activa

**Como** administrador de la botica **quiero** que el sistema no permita cobrar si falta el RUC o la serie de boletas **para** no emitir comprobantes inválidos.

| | |
|---|---|
| **Módulo** | Ventas / Comprobantes |
| **Pantalla** | `/pos/nueva` (el rechazo se ve al confirmar el cobro) |
| **Endpoint(s)** | `POST /api/v1/ventas`, `PUT /api/v1/ventas/{id}/completar` |
| **Permiso** | `Ventas:Crear` / `Ventas:Actualizar` |
| **Reglas** | [RN-VEN-007, RN-CMP-002, RN-CMP-003](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-EMP-003, RN-EMP-004](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-SER-008](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: La empresa no tiene RUC
  Dado que a la empresa se le quitó el RUC al editarla
  Cuando intento cobrar una venta
  Entonces veo "Tu empresa no tiene RUC registrado, por lo que no puede emitir comprobantes. Complétalo en Administración > Empresa para poder cobrar."
  Y no se guarda la venta ni se consume ningún correlativo

Escenario: La sucursal no tiene serie de boletas activa
  Dado que la sucursal no tiene una serie de tipo Boleta en estado "Activa"
  Cuando intento cobrar
  Entonces veo "Esta sucursal no tiene una serie de boletas activa. Configúrala en SUNAT > Series Fiscales para poder cobrar."
  Y no se guarda la venta ni se consume ningún correlativo

Escenario: Una serie inactiva no sirve
  Dado que la única serie de boletas de la sucursal está "Inactiva"
  Cuando intento cobrar
  Entonces veo el mismo aviso de que no hay serie de boletas activa

Escenario: Orden de las comprobaciones
  Dado que faltan el RUC y la serie
  Cuando intento cobrar
  Entonces veo primero el aviso del RUC

Escenario: Dejar la venta en espera no necesita RUC ni serie
  Dado que falta el RUC o la serie
  Cuando hago clic en "Guardar y Atender Siguiente"
  Entonces la venta se guarda en espera
  Y el rechazo ocurrirá recién al cobrarla
```

### HU-VEN-017 — Emitir la boleta al cobrar

**Como** cajero **quiero** que cada venta cobrada genere su boleta con número correlativo **para** entregar un comprobante al cliente.

| | |
|---|---|
| **Módulo** | Comprobantes |
| **Pantalla** | Diálogo «Venta Procesada Exitosamente» (en `/pos/nueva`) y «Comprobante» en `/pos/:id` |
| **Endpoint(s)** | `POST /api/v1/ventas`, `PUT /api/v1/ventas/{id}/completar`, `GET /api/v1/ventas/{id}` |
| **Permiso** | `Ventas:Crear` / `Ventas:Actualizar`; `Ventas:Leer` para ver |
| **Reglas** | [RN-VEN-059, RN-CMP-001, RN-CMP-004, RN-CMP-005, RN-CMP-006, RN-CMP-007, RN-CMP-008, RN-CMP-009, RN-CMP-010, RN-CMP-011, RN-CMP-013, RN-CMP-015](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Parcial (la boleta se registra solo en SOFIA: no se envía a SUNAT, no tiene hash, XML, constancia ni URL de verificación, y las ventas nunca emiten factura) |

**Criterios de aceptación**

```gherkin
Escenario: La primera boleta de la serie
  Dado que la serie de boletas "B001" tiene correlativo inicial 0
  Cuando cobro mi primera venta
  Entonces la boleta tiene el número "B001-00000001"
  Y el diálogo muestra "Nº Comprobante:" "Boleta B001-00000001"
  Y "Estado SUNAT:" "Pendiente de envío a SUNAT"
  Y el subtítulo "Comprobante registrado. Pendiente de envío a SUNAT."

Escenario: Importes de la boleta
  Dado que el total de la venta es S/ 50.00 (precios con IGV incluido, tasa fija de 18 %)
  Entonces la base gravada es 42.37 y el IGV es 7.63 (la base es el total dividido entre 1,18, redondeado a 2 decimales con la mitad hacia arriba)
  Y el monto exonerado es 0.00
  Y base más IGV suman exactamente el total

Esquema del escenario: El desglose siempre suma el total
  Dado un total de venta de <total>
  Entonces la base gravada es <base> y el IGV es <igv>

  Ejemplos:
    | total  | base  | igv   |
    | 10.00  | 8.47  | 1.53  |
    | 50.00  | 42.37 | 7.63  |
    | 100.00 | 84.75 | 15.25 |

Escenario: Venta sin cliente
  Dado que no elegí cliente
  Entonces la boleta va a nombre de "CLIENTE EVENTUAL", tipo de documento "-" y número "00000000"

Escenario: Venta con cliente identificado
  Dado que elegí a un cliente con DNI "45781236"
  Entonces la boleta lleva su tipo de documento (código SUNAT "1" para DNI), su número de documento y su nombre y apellidos

Escenario: Cliente con documento de más de 20 caracteres
  Dado que el número de documento del cliente supera los 20 caracteres
  Entonces la boleta va al cliente eventual

Escenario: Aunque el cliente tenga RUC, la venta emite boleta
  Dado que el cliente tiene tipo de documento RUC
  Entonces el comprobante sigue siendo una boleta y no se emite factura

Escenario: Una venta en espera no tiene comprobante
  Dado que la venta está en estado "Pendiente"
  Entonces no tiene boleta ni consume correlativo
  Y su código de venta son los primeros 8 caracteres de su identificador, en mayúsculas

Escenario: El código de venta es el número de la boleta
  Dado que cobré la venta en espera
  Entonces su código en el listado y el detalle pasa a ser el número de la boleta, por ejemplo "B001-00000001"
  Y una venta anulada conserva el código de su boleta

Escenario: Los datos del emisor no se guardan en la boleta
  Dado que la empresa cambia de nombre, de RUC o de dirección de la sucursal
  Entonces las boletas anteriores muestran los datos nuevos al reimprimirse

Escenario: Un número fiscal no se repite
  Dado que dos cobros obtuvieran el mismo número de la misma serie
  Entonces el segundo guardado se rechaza con "Db.Duplicado" y veo "Ocurrió un error al procesar la venta."
  # No verificado en ejecución

Escenario: Estructura de la boleta
  Entonces la tabla de comprobantes exige venta, serie, correlativo, tipo y número de documento del cliente, razón social, los cuatro importes y el estado, con claves foráneas a la venta y a la serie
  Y la base no tiene restricción sobre el estado ni sobre los importes: eso lo cuida el código
  Y los guards de la entidad (correlativo mayor que 0, documento de hasta 20 caracteres, razón social de hasta 200, importes no negativos) no pueden fallar con los datos que arma el generador, así que el cajero nunca ve sus mensajes

Escenario: Estado del comprobante
  Entonces el comprobante nace como "NoEnviado" y se muestra "Pendiente de envío a SUNAT"
  Y los estados "Aceptado" y "Rechazado" ("Aceptado por SUNAT", "Rechazado por SUNAT") existen pero nada los asigna; "Anulado" lo asigna la anulación de la venta
```

> **Brecha conocida.** No existe envío a SUNAT: no hay OSE/PSE, hash, XML, constancia ni URL pública (RN-CMP-007; 01c2 hallazgo 6). Los importes del ticket impreso los calcula el navegador y no se leen de la boleta guardada: pueden diferir en un centavo (hallazgo 11). Con una cobertura de seguro la boleta se emite por el total completo y la pantalla muestra el monto a pagar. El valor por defecto del estado en el script de base de datos es `Pendiente`, mientras el código escribe `NoEnviado` (el SPA traduce ambos igual; hallazgo 13). Si un `Result.Failure` ocurriera después de consumir el correlativo quedaría un hueco en la numeración (RN-CMP-004; con el código actual no hay entradas que lo provoquen). Las restricciones de la tabla de comprobantes y los guards de la entidad (RN-CMP-008, RN-CMP-010) no producen mensajes al usuario. Las notas de crédito comparten entidad, contador e índice único (RN-CMP-015; ver HU-DEV-004).

### HU-VEN-018 — Imprimir y reimprimir el comprobante

**Como** cajero **quiero** imprimir el ticket de la venta y reimprimirlo después **para** entregar o reponer el comprobante al cliente.

| | |
|---|---|
| **Módulo** | Comprobantes |
| **Pantalla** | Diálogo «Venta Procesada Exitosamente» (tras cobrar y desde «Reimprimir Comprobante» en `/pos/:id`) |
| **Endpoint(s)** | Ninguno (los datos viajan en la respuesta del cobro o en `GET /api/v1/ventas/{id}`) |
| **Permiso** | Ninguno propio; «Nueva Venta POS» del diálogo exige `Ventas:Crear` |
| **Reglas** | [RN-CMP-011, RN-CMP-012, RN-CMP-014](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Qué muestra el diálogo tras cobrar
  Cuando termino de cobrar una venta
  Entonces veo "Venta Procesada Exitosamente" y "Comprobante registrado. Pendiente de envío a SUNAT."
  Y "Nº Comprobante:", "Estado SUNAT:", "Total Pagado:" y "Medio de Pago:"
  Y los botones "Nueva Venta POS" (solo con el permiso "Ventas:Crear"), "Imprimir Comprobante" y "Volver a Caja POS"
  Y no puedo cerrar el diálogo con Esc ni haciendo clic fuera: solo con sus botones

Escenario: Imprimir el ticket
  Cuando hago clic en "Imprimir Comprobante"
  Entonces se abre el diálogo de impresión del navegador con un ticket continuo de rollo de 80 mm
  Y la impresión no es automática: se repite cada vez que hago clic

Escenario: Contenido del ticket
  Entonces el ticket muestra la razón social, "RUC:", la dirección de la sucursal, el tipo y número de la boleta y la fecha "dd/MM/yyyy HH:mm"
  Y la tabla "Producto / Lote", "Cant.", "P. Unit." y "Subtotal"
  Y "Datos del Cliente:", "Documento:", "Subtotal (Op. Gravada):", "IGV (18%):" y "TOTAL A PAGAR:"
  Y una fila por pago y "Vuelto:" solo si es mayor que 0
  Y "Estado SUNAT:" y "Cajero:"

Escenario: Línea vendida por presentación
  Dado que vendí 1 caja de 10 unidades
  Entonces la fila del ticket muestra la cantidad "1 × Caja (10)" y el precio por presentación

Escenario: El cliente eventual no imprime documento
  Dado que la venta es al cliente eventual
  Entonces el ticket no imprime la línea "Documento:"

Escenario: Nueva venta o volver
  Cuando hago clic en "Nueva Venta POS"
  Entonces el terminal se limpia y queda listo en "/pos/nueva"
  Y con "Volver a Caja POS" el terminal se limpia y voy a "/pos"

Escenario: Reimprimir desde el detalle de la venta
  Dado que abro el detalle de una venta con comprobante que no está "Anulada"
  Cuando hago clic en "Reimprimir Comprobante"
  Entonces se abre el mismo diálogo con los datos de la venta guardada
  Y el ticket usa la fecha de la venta y el nombre del cajero que la hizo
  Y el vuelto se recalcula como pagos menos total de la venta

Escenario: Una venta anulada no se reimprime
  Dado que la venta está "Anulada"
  Entonces el detalle no ofrece "Reimprimir Comprobante"

Escenario: Datos del emisor al reimprimir
  Entonces razón social, RUC y dirección se leen de la empresa y la sucursal en ese momento
```

> **Brecha conocida.** La reimpresión muestra el mismo título «Venta Procesada Exitosamente» aunque no se haya procesado nada. El ticket no incluye código QR, hash de firma, enlace de verificación ni leyenda legal de representación impresa (19f §5.2). La fecha del ticket recién cobrado es la hora del navegador, no la del servidor. La columna del importe se rotula «Subtotal» (19f §5.3).

### HU-VEN-019 — Cubrir parte de la venta con una aseguradora

**Como** cajero **quiero** registrar que una aseguradora cubre parte de la venta **para** cobrarle al cliente solo su copago.

| | |
|---|---|
| **Módulo** | Ventas / Seguros |
| **Pantalla** | `/pos/nueva` — bloque «Cobertura de Seguro (opcional)» (construido pero **oculto**) |
| **Endpoint(s)** | `POST /api/v1/ventas`, `PUT /api/v1/ventas/{id}/completar` (campos `aseguradoraId` y `montoCubiertoSeguro`) |
| **Permiso** | `Ventas:Crear` / `Ventas:Actualizar`; `Seguros:Leer` (lista) |
| **Reglas** | [RN-VEN-023, RN-VEN-047, RN-VEN-048](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Parcial (la API está completa; la pantalla tiene el bloque oculto con una constante en el código, así que ningún cobro desde el SPA envía cobertura) |

**Criterios de aceptación**

```gherkin
Escenario: La cobertura no se ofrece en pantalla
  Dado que estoy en el terminal de venta
  Entonces no veo el bloque "Cobertura de Seguro (opcional)"
  Y ningún rol, empresa ni configuración puede mostrarlo

Escenario: Cobrar con cobertura por la API
  Dado que llamo a la API con una venta de S/ 100.00, una aseguradora válida y un monto cubierto de S/ 40.00
  Entonces el monto a pagar es S/ 60.00
  Y los pagos solo tienen que cubrir S/ 60.00
  Y si el cliente paga S/ 100.00 en efectivo, el vuelto es S/ 40.00 y el efectivo neto de la venta es S/ 60.00

Escenario: Se crea un reclamo de seguro
  Dado que la venta trae aseguradora y monto (aunque sea 0)
  Entonces se crea un reclamo ligado a la primera línea de la venta con estado "Aprobado", el monto cubierto, el copago del paciente (total menos cobertura) y un código de autorización "AUTH_" más 8 caracteres hexadecimales generados por el sistema
  Y el comprobante se emite por el total completo

Escenario: El monto cubierto está fuera de rango
  Dado que el monto cubierto es negativo o mayor que el total de la venta
  Entonces veo "El monto cubierto por el seguro debe estar entre 0 y el total de la venta."

Escenario: Monto sin aseguradora
  Dado que el monto cubierto es mayor que 0 y no se envió aseguradora
  Entonces veo "Selecciona una aseguradora para aplicar la cobertura del seguro."

Escenario: La aseguradora no existe
  Dado que la aseguradora fue eliminada o es de otra empresa
  Entonces el servidor responde 404 y veo "La aseguradora seleccionada no existe o fue eliminada."

Escenario: Anular la venta no toca el reclamo
  Dado que anulo una venta con reclamo de seguro
  Entonces el reclamo queda como estaba

Escenario: Estructura del reclamo de seguro
  Entonces el reclamo exige línea de venta y aseguradora, montos no negativos, estado de hasta 50 caracteres y código de autorización de hasta 100, con claves foráneas a la línea y a la aseguradora
  Y la base no tiene restricciones CHECK ni índice único sobre los reclamos
  Y con los valores que genera la venta, esas comprobaciones no pueden fallar: no hay otra ruta que cree o edite reclamos

Escenario: Un borrador antiguo con aseguradora
  Dado que quedó guardado un borrador con aseguradora y monto cubierto
  Cuando lo restauro
  Entonces la cobertura reduce el "TOTAL A PAGAR" sin forma visible de quitarla
  # No verificado en ejecución (el borrador caduca a las 12 horas)
```

> **Brecha conocida.** El código de autorización lo inventa el sistema (no hay integración con la aseguradora), el estado siempre es «Aprobado», nada impide dos reclamos para la misma línea (sin restricciones únicas en la base) y anular no los toca (01c2 hallazgo 5; RN-VEN-048). Los reclamos los consumen el arqueo, el dashboard y las devoluciones. No existen descuentos: la cobertura es el único ajuste al monto a cobrar (RN-VEN-023).

## 6. Venta en espera

### HU-VEN-020 — Guardar la venta en espera

**Como** cajero **quiero** dejar una venta en espera y atender al siguiente cliente **para** no frenar la fila mientras el primero busca el dinero o la receta.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/nueva` — botón «Guardar y Atender Siguiente» |
| **Endpoint(s)** | `POST /api/v1/ventas` con estado `Pendiente` y sin pagos |
| **Permiso** | `Ventas:Crear` |
| **Reglas** | [RN-VEN-003, RN-VEN-004, RN-VEN-008, RN-VEN-015, RN-VEN-049, RN-VEN-051, RN-VEN-052, RN-VEN-059](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Dejar una venta en espera
  Dado que tengo mi caja abierta y el carrito tiene 100 unidades de "Paracetamol Demo 500 mg"
  Cuando hago clic en "Guardar y Atender Siguiente"
  Entonces veo "Venta guardada en espera. Retómala desde Ventas cuando el cliente vuelva."
  Y el terminal queda limpio para el siguiente cliente
  Y la venta queda en estado "Pendiente"

Escenario: Qué reserva la venta en espera
  Dado que dejé una venta en espera
  Entonces el stock de las 100 unidades se descuenta de la sucursal desde que se guarda
  Y cuenta como dispensación de la receta que use (ver HU-VEN-010)
  Y queda ligada a la caja con que se creó

Escenario: Qué no tiene una venta en espera
  Entonces no tiene pagos, ni boleta, ni correlativo
  Y no exige "Receta verificada", RUC de la empresa ni serie de boletas
  Y no genera el evento de dominio de venta cobrada
  Y su código de venta son los 8 primeros caracteres del identificador, en mayúsculas

Escenario: No se pide receta, pagos ni número de operación
  Dado que el carrito tiene un producto controlado, sin receta elegida y sin pagos
  Cuando hago clic en "Guardar y Atender Siguiente"
  Entonces la venta se guarda; el botón solo se deshabilita con el carrito vacío, sin caja o mientras se procesa

Escenario: Sin caja abierta no se puede dejar en espera
  Dado que no tengo caja abierta
  Entonces el terminal no muestra el botón (solo el aviso "Debes aperturar la caja del turno antes de procesar una venta.")
  Y si la petición llega al servidor con una sesión inexistente o cerrada, se rechaza con "Venta.Caja" y veo "Ocurrió un error al procesar la venta."

Escenario: Falla al guardar
  Dado que el servidor rechaza la venta (por ejemplo, stock insuficiente)
  Entonces veo el mismo mensaje del mapa de cobro o "Ocurrió un error al procesar la venta."

Escenario: La venta en espera no caduca
  Dado que pasaron varios días
  Entonces la venta sigue "Pendiente", con su stock y su dispensación reservados
  Y solo se cobra o se anula
```

> **Brecha conocida.** No hay vencimiento ni proceso que libere una venta en espera: puede dejar stock y dispensaciones reservados indefinidamente (01c2 hallazgo 7). Al cerrar la caja no hay aviso ni bloqueo por ventas en espera (RN-CAJ-017, ver HU-CAJ-003).

### HU-VEN-021 — Retomar y modificar una venta en espera

**Como** cajero **quiero** retomar una venta en espera y cambiar sus productos o su cliente **para** ajustarla antes de cobrar.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos` — «Ventas» (filtro «Estado»); `/pos/:id` — «Cobrar Venta»; `/pos/nueva?continuar={id}` |
| **Endpoint(s)** | `GET /api/v1/ventas`, `GET /api/v1/ventas/{id}`, `PUT /api/v1/ventas/{id}/detalles` |
| **Permiso** | `Ventas:Leer`; «Cobrar Venta» exige `Ventas:Crear` y `Ventas:Actualizar`; la API exige `Ventas:Actualizar` |
| **Reglas** | [RN-VEN-008, RN-VEN-009, RN-VEN-010, RN-VEN-025, RN-VEN-027, RN-VEN-053, RN-VEN-054, RN-VEN-055](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Encontrar la venta en espera
  Dado que estoy en "/pos"
  Cuando elijo "Estado" = "Pendiente" y hago clic en "Buscar"
  Entonces veo las ventas en espera de mi sucursal, de cualquier cajero, y puedo abrir su código

Escenario: Retomarla desde el detalle
  Dado que abrí el detalle de una venta "Pendiente" y tengo los permisos "Ventas:Crear" y "Ventas:Actualizar"
  Cuando hago clic en "Cobrar Venta"
  Entonces el terminal carga las líneas, el cliente y la receta de la primera línea que tenga una
  Y veo el aviso "Estás retomando una venta en espera: puedes agregar, quitar o modificar productos antes de cobrar."
  Y las líneas cargadas conservan el precio guardado y las nuevas usan el del catálogo

Escenario: La existencia incluye lo que la venta ya reservó
  Dado que la venta reservó 100 unidades de un lote
  Entonces al retomarla puedo subir esa línea hasta el stock vendible actual más esas 100 unidades

Escenario: Sin los dos permisos no aparece el botón
  Dado que no tengo "Ventas:Actualizar"
  Entonces el detalle no muestra "Cobrar Venta"

Escenario: Cualquier usuario de la misma sucursal puede retomarla
  Dado que otro cajero guardó la venta
  Cuando yo la abro con los permisos necesarios
  Entonces puedo retomarla y modificarla

Escenario: Guardar los cambios
  Dado que quité una línea y agregué otra
  Cuando hago clic en "Actualizar Venta en Espera"
  Entonces veo "Venta guardada en espera. Retómala desde Ventas cuando el cliente vuelva."
  Y el terminal se limpia y navega a "/pos/nueva" sin "continuar"
  Y el servidor devolvió al stock todas las líneas anteriores y descontó las nuevas, recalculando el total
  Y no pidió caja abierta ni cambió la caja de la venta

Escenario: Las líneas se validan como al crear
  Dado que las nuevas líneas traen una receta de otro paciente o un lote vencido
  Cuando guardo los cambios
  Entonces el servidor las rechaza con los mismos mensajes que al cobrar, sin pedir "Receta verificada"

Escenario: El cliente enviado reemplaza al guardado
  Cuando guardo los cambios sin cliente seleccionado
  Entonces la venta queda sin cliente

Escenario: La venta ya no está en espera
  Dado que otro cajero la cobró mientras yo la abría
  Cuando abro "/pos/nueva?continuar={id}"
  Entonces veo "Esta venta ya no está en espera; te llevamos a su detalle." y paso a su detalle

Escenario: No se puede cargar la venta
  Dado que la venta no existe o es de otra sucursal
  Entonces veo "Error al cargar la venta"

Escenario: Actualizar una venta de otra sucursal o no pendiente (la API)
  Dado que llamo a "PUT ventas/{id}/detalles" para una venta de otra sucursal o que ya no es "Pendiente"
  Entonces el servidor la rechaza con "Venta.Actualizar" y veo "Ocurrió un error al procesar la venta."
```

> **Brecha conocida.** La ruta `/pos/nueva?continuar=` solo exige `Ventas:Leer` y `Ventas:Crear`: quien escriba la URL sin `Ventas:Actualizar` abre la pantalla y el servidor responde 403 al guardar o cobrar (19f §0.2, hallazgo a1 de [05b](../../ola-1-fundamentos/05-seguridad/05b-matriz-de-permisos.md)). `Venta.Actualizar` y `Venta.Completar` responden 400 aunque sean de tipo no encontrado o prohibido (01c2 §2).

### HU-VEN-022 — Cobrar una venta en espera

**Como** cajero **quiero** cobrar una venta que quedó en espera **para** completarla con mi propia caja y emitir la boleta.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/nueva?continuar={id}` — botón «Procesar Venta» |
| **Endpoint(s)** | `PUT /api/v1/ventas/{id}/completar` |
| **Permiso** | `Ventas:Actualizar` |
| **Reglas** | [RN-VEN-002, RN-VEN-006, RN-VEN-007, RN-VEN-016, RN-VEN-027, RN-VEN-034, RN-VEN-038, RN-VEN-046, RN-VEN-056, RN-VEN-057, RN-VEN-060](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Cobrar la venta en espera con mi caja
  Dado que retomé una venta en espera de S/ 50.00 y tengo mi propia caja abierta en esa sucursal
  Cuando completo el pago y confirmo con "Confirmar y Procesar Venta"
  Entonces la venta pasa a "Completada" y se emite la boleta
  Y la venta se reasigna a mi caja, sea cual fuere la caja con que se guardó
  Y el efectivo cobrado cuenta en el arqueo de mi sesión
  Y el empleado que registró la venta no cambia
  Y se abre el diálogo "Venta Procesada Exitosamente" con el nuevo número de boleta

Escenario: No tengo caja propia abierta
  Dado que mi caja está cerrada o no la abrí en esa sucursal
  Cuando intento cobrar
  Entonces veo "Necesitas tener tu propia caja abierta en esta sucursal para cobrar la venta."
  Y la comprobación va antes que las de líneas, pagos y comprobante

Escenario: Las líneas se reenvían y se valida todo otra vez
  Dado que el terminal reenvía todas las líneas del carrito al cobrar
  Entonces el servidor aplica de nuevo cuarentena, vencimiento, stock, precio, receta y dispensaciones, como en una venta nueva

Escenario: Cobrar sin reenviar las líneas (solo API)
  Dado que llamo a la API sin enviar "Detalles"
  Entonces se conservan las líneas guardadas y no se vuelven a revisar cuarentena, vencimiento, stock, precio ni dispensaciones
  Y solo se vuelve a juzgar la política de receta y que el lote y el producto existan

Escenario: La empresa pasó a exigir registro de receta mientras la venta esperaba
  Dado que la venta en espera tiene una línea de receta simple sin receta registrada
  Y la empresa activó "exigir registro de receta" después
  Cuando la cobro sin reenviar las líneas
  Entonces el servidor la rechaza y veo "Uno de los productos requiere receta médica. Selecciona el cliente y una de sus recetas registradas."

Escenario: Reenviar una lista de líneas vacía (solo API)
  Dado que llamo a la API con "Detalles" vacío
  Entonces el servidor la rechaza con "Venta.Detalles" y veo "Ocurrió un error al procesar la venta."

Escenario: Falta "Receta verificada"
  Dado que la venta tiene receta simple sin receta registrada en una empresa flexible
  Cuando intento cobrar sin marcar la casilla
  Entonces veo "Marca "Receta verificada" para confirmar que revisaste la receta del cliente antes de cobrar."

Escenario: La venta ya no está pendiente, o es de otra sucursal
  Entonces el servidor responde "Venta.Completar" y veo "Ocurrió un error al procesar la venta."

Escenario: Dos cajeros cobran la misma venta a la vez
  Dado que dos cajeros parten de la misma lectura de la venta
  Cuando ambos confirman
  Entonces solo se guarda un cobro y el otro ve "Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo."
  # No verificado en ejecución

Escenario: Sin RUC o sin serie de boletas
  Entonces se aplican los avisos de HU-VEN-016, porque recién ahora se emite la boleta
```

> **Brecha conocida.** El cobro sin reenviar líneas no revalida vencimiento, cuarentena, stock, paciente de la receta ni dispensaciones; el terminal siempre reenvía las líneas, de modo que el caso solo es alcanzable llamando a la API (01c2 hallazgo 10).

## 7. Anulación de ventas

### HU-VEN-023 — Anular una venta cobrada

**Como** supervisor o cajero con permiso **quiero** anular una venta cobrada indicando el motivo **para** revertir el stock y el comprobante cuando hubo un error.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/:id` — botón «Anular Venta»; diálogo «Anular Venta» |
| **Endpoint(s)** | `PUT /api/v1/ventas/{id}/anular` (cuerpo: el motivo como texto JSON) |
| **Permiso** | `Ventas:Anular` |
| **Reglas** | [RN-VEN-002, RN-VEN-061, RN-VEN-062, RN-VEN-063, RN-VEN-064, RN-VEN-067, RN-VEN-068, RN-VEN-069, RN-CMP-012](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CAJ-012](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Anular una venta cobrada
  Dado que la venta "B001-00000001" está "Completada", no tiene devoluciones y es de mi sucursal
  Y tengo "Ventas:Anular" y mi propia caja abierta en esa sucursal
  Cuando hago clic en "Anular Venta"
  Entonces se abre el diálogo "Anular Venta" con "Esta acción revierte el stock descontado. Ingresa el motivo de la anulación."
  Cuando escribo en "Motivo de Anulación" "Error en el cobro" y hago clic en "Anular Venta"
  Entonces veo "Venta anulada correctamente." y el detalle se recarga

Escenario: Qué se revierte y qué queda registrado
  Dado que anulé la venta de 100 unidades de "Paracetamol Demo 500 mg"
  Entonces cada línea devuelve sus 100 unidades base al stock del lote en la sucursal (si la fila de stock ya no existe, esa línea se omite sin aviso)
  Y la boleta pasa a estado "Anulado"
  Y la venta queda "Anulada" con su motivo, la caja desde la que se anuló y mi usuario y fecha
  Y se registra el evento de auditoría "VENTA_ANULAR"
  Y los pagos y el reclamo de seguro quedan como estaban, sin filas de reverso
  Y no se informa a SUNAT: no existe comunicación de baja

Escenario: El detalle de la venta anulada
  Entonces el detalle muestra "Motivo de Anulación:" y el estado "Anulada"
  Y ya no ofrece "Reimprimir Comprobante"
  Y la dispensación de la receta usada queda liberada

Escenario: La anulación necesita mi propia caja abierta
  Dado que no tengo caja propia abierta en la sucursal de la venta
  Cuando confirmo la anulación
  Entonces veo "Necesitas tener tu propia caja abierta en esta sucursal para anular una venta cobrada."

Escenario: Efecto en el arqueo cuando se anula desde otra caja
  Dado que una venta de S/ 20.00 se cobró en efectivo en la caja A
  Cuando la anulo con mi caja B abierta
  Entonces el arqueo de mi caja B resta S/ 20.00 (el efectivo sale del cajón de quien devuelve)
  Y la caja A sigue contando esa venta mientras no se cierre (ver HU-CAJ-004)

Escenario: Anular cualquier venta de mi sucursal
  Dado que la venta la hizo otro cajero de mi misma sucursal
  Entonces puedo anularla con mi permiso "Ventas:Anular"
  Y no hace falta ser el cajero que la vendió

Escenario: El motivo es obligatorio
  Cuando hago clic en "Anular Venta" con el motivo vacío
  Entonces veo "El motivo es obligatorio." y el diálogo no se cierra

Escenario: Otra operación modificó la venta o el stock
  Dado que alguien cobró o modificó el mismo stock al mismo tiempo
  Cuando confirmo la anulación
  Entonces veo "Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo."
  # No verificado en ejecución

Escenario: Sin el permiso no hay botón
  Dado que no tengo "Ventas:Anular"
  Entonces el detalle no muestra "Anular Venta"
```

> **Brecha conocida.** La anulación no crea ninguna fila de reverso de pagos: el efectivo devuelto se refleja solo por la caja de anulación (01c2 hallazgo 19). El permiso `Ventas:Anular` es aparte de `Ventas:Actualizar` (script `0017`). El `Admin` solo anula ventas de la sucursal de su token (RN-VEN-062).

### HU-VEN-024 — Anular una venta en espera

**Como** cajero **quiero** anular una venta que dejé en espera **para** liberar el stock y la receta que reservaba.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/:id` — botón «Anular Venta» |
| **Endpoint(s)** | `PUT /api/v1/ventas/{id}/anular` |
| **Permiso** | `Ventas:Anular` |
| **Reglas** | [RN-VEN-052, RN-VEN-065, RN-VEN-067, RN-VEN-068, RN-VEN-069](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Anular una venta en espera
  Dado que la venta está "Pendiente"
  Cuando la anulo con el motivo "Cliente no volvió"
  Entonces veo "Venta anulada correctamente."
  Y el stock reservado vuelve a la sucursal
  Y la dispensación de la receta se libera
  Y la venta queda "Anulada"

Escenario: No necesita caja abierta
  Dado que no tengo caja abierta
  Cuando anulo una venta "Pendiente"
  Entonces la anulación se hace igual y no se guarda caja de anulación, porque nunca se cobró

Escenario: Una venta en espera no tiene comprobante
  Entonces no hay boleta que pasar a "Anulado"
```

### HU-VEN-025 — Saber por qué una venta no se puede anular

**Como** cajero **quiero** entender los motivos por los que el sistema no me deja anular una venta **para** gestionar el caso correctamente (por ejemplo, con una devolución).

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/:id` — diálogo «Anular Venta» |
| **Endpoint(s)** | `PUT /api/v1/ventas/{id}/anular` |
| **Permiso** | `Ventas:Anular` |
| **Reglas** | [RN-VEN-062, RN-VEN-063, RN-VEN-064, RN-VEN-065, RN-VEN-066](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: La venta tiene devoluciones
  Dado que a la venta ya se le devolvió una línea, aunque sea parcialmente
  Entonces el detalle no muestra el botón "Anular Venta"
  Y si la petición llega al servidor veo "Esta venta tiene devoluciones registradas y ya no puede anularse. Gestiona los cambios desde Devoluciones."
  # La razón: anular restauraría y reembolsaría dos veces las unidades ya devueltas

Escenario: La venta está totalmente devuelta
  Dado que la venta está "Devuelta"
  Entonces solo permite "Reimprimir Comprobante" y, si se intenta anular, veo el mismo mensaje de devoluciones

Escenario: La venta ya está anulada
  Dado que la venta está "Anulada"
  Entonces el detalle no ofrece botones
  Y si la petición llega al servidor veo "Error al anular la venta."

Escenario: Solo se anulan ventas "Completada" o "Pendiente"
  Entonces el botón "Anular Venta" solo aparece en esos dos estados

Escenario: La venta es de otra sucursal
  Dado que mi sesión pertenece a la sucursal "Sede Principal" y la venta es de "Sucursal Demo Norte"
  Cuando intento anularla
  Entonces veo "Error al anular la venta."
  Y esto vale también si soy administrador

Escenario: Motivo de solo espacios
  Cuando escribo solo espacios en "Motivo de Anulación" y hago clic en "Anular Venta"
  Entonces el diálogo lo acepta pero el servidor lo rechaza y veo "Error al anular la venta."

Escenario: Motivo demasiado largo
  Cuando escribo más de 255 caracteres
  Entonces el botón "Anular Venta" no hace nada y no aparece ningún mensaje

Escenario: La venta no existe
  Dado que la venta fue eliminada o es de otra empresa
  Entonces el servidor responde 404 y veo "Error al anular la venta."
```

> **Brecha conocida.** Solo `Venta.Anular.ConDevoluciones`, `Venta.Anular.SinSesionAbierta` y `Concurrency.Conflict` tienen mensaje propio; todo lo demás (ya anulada, otra sucursal, motivo en blanco, sin `Auth.Empleado`) se ve como «Error al anular la venta.» (01c2 §2 punto 5). El texto de devoluciones remite a «Devoluciones», que no es una pantalla: las devoluciones se procesan desde el mismo detalle (HU-DEV-001).

## 8. Consulta de ventas

### HU-VEN-026 — Buscar las ventas de mi sucursal

**Como** cajero o supervisor **quiero** buscar ventas por fecha, estado, empleado y cliente **para** encontrar una venta, retomar una en espera o revisar lo vendido.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos` — «Ventas» |
| **Endpoint(s)** | `GET /api/v1/ventas`, `GET /api/v1/empleados`, `GET /api/v1/pacientes` (autocompletar de los filtros) |
| **Permiso** | `Ventas:Leer`; `Empleados:Leer` y `Pacientes:Leer` para los filtros; «Nueva Venta POS» exige `Ventas:Crear` |
| **Reglas** | [RN-VEN-001, RN-VEN-059, RN-VEN-070, RN-VEN-071, RN-VEN-072, RN-VEN-073](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-GEN-008, RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Entrar sin resultados hasta buscar
  Dado que entro a "/pos"
  Entonces veo "Ventas", los filtros "Empleado", "Cliente", "Fecha desde", "Fecha hasta" y "Estado" y ningún resultado hasta pulsar "Buscar"

Escenario: Buscar las ventas de hoy
  Cuando elijo "Fecha desde" y "Fecha hasta" con el día de hoy y hago clic en "Buscar"
  Entonces veo "Resultados (N)" con las columnas "Código de Venta", "Fecha", "Cliente", "Empleado", "Total (S/)", "Estado" e "Ítems"
  Y las fechas se interpretan como el inicio y el fin del día en mi zona horaria
  Y el orden es por fecha y hora descendente

Escenario: Solo se ven las ventas de la sucursal de mi sesión
  Entonces el listado nunca muestra ventas de otras sucursales ni hay filtro de sucursal
  Y esto vale también para el administrador

Escenario: Cajero y administrador ven lo mismo
  Entonces veo las ventas de todos los empleados de mi sucursal
  Y para ver solo las mías filtro por "Empleado"

Escenario: Filtrar por empleado o cliente
  Cuando escribo en "Empleado" o "Cliente" y elijo una opción de la lista
  Entonces el filtro se aplica
  Y si solo escribo el texto sin elegir una opción, el filtro se ignora en silencio
  Y en "Cliente", un texto de solo dígitos busca por documento y otro texto por nombre
  Y si no hay coincidencias veo la opción deshabilitada "Sin resultados"

Escenario: Filtrar por estado
  Cuando elijo "Estado" = "Pendiente"
  Entonces veo solo las ventas en espera
  Y las opciones son "— Todos —", "Completada", "Anulada", "Devuelta" y "Pendiente"

Escenario: Cada fila
  Entonces el código de venta es un enlace al detalle y tiene la forma "B001-00000001" si hay boleta, o los 8 primeros caracteres del identificador si la venta está en espera
  Y el cliente aparece como "Público General" si no hubo cliente

Escenario: Paginación y memoria de filtros
  Entonces la página es de 10 filas por defecto (opciones 10, 25 y 50)
  Y al volver a la pantalla se recuerdan filtros, página y tamaño mientras no recargue el navegador
  Y "Limpiar" vacía los filtros, oculta la tabla y borra lo recordado

Escenario: Botón de nueva venta
  Dado que tengo el permiso "Ventas:Crear"
  Entonces veo "Nueva Venta POS" y al hacer clic voy a "/pos/nueva"

Escenario: Falla la búsqueda
  Entonces veo "Error al buscar ventas"

Escenario: No se puede buscar por código ni por monto
  Entonces no existe filtro por código de venta ni por total
```

> **Brecha conocida.** No hay vista «mis ventas» ni acceso a otras sucursales desde esta pantalla (01c2 hallazgo 16); cómo se cambia la sucursal activa del token no se verificó (RN-VEN-071, 01c2 §5.1 punto 6). `search()` no reinicia la página al cambiar un filtro (19f §6). **No verificado:** qué nombre muestra el listado cuando el empleado de la venta fue eliminado.

### HU-VEN-027 — Ver el detalle de una venta

**Como** cajero o supervisor **quiero** ver una venta completa **para** revisar sus productos, pagos, comprobante, devoluciones y operar sobre ella.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos/:id` — «Detalle de Venta» |
| **Endpoint(s)** | `GET /api/v1/ventas/{id}` |
| **Permiso** | `Ventas:Leer` |
| **Reglas** | [RN-VEN-001, RN-VEN-035, RN-VEN-059, RN-VEN-074, RN-CMP-013](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Ver una venta cobrada
  Dado que abro una venta "Completada" de mi sucursal
  Entonces veo el código de venta, "26/10/2026 10:15 — Completada", Cliente, Empleado y "Total (S/)"
  Y "Comprobante" con "Nº Comprobante:" y "Estado SUNAT:"
  Y "Productos Vendidos" con Producto, Lote, Cantidad, "Devuelto", "P. Unitario (S/)" y "Subtotal (S/)"
  Y "Pagos Registrados" con "Método de Pago", "Monto (S/)", "Referencia de Operación" y "Fecha de Pago"
  Y "Vuelto:" si es mayor que 0

Escenario: Línea vendida por presentación
  Entonces la columna Cantidad muestra "2 × Caja x10 (20)" (presentación y unidades base)

Escenario: Venta anulada
  Entonces veo "Motivo de Anulación:" y el estado "Anulada"

Escenario: Verificación de receta
  Dado que la venta se cobró con "Receta verificada"
  Entonces veo "Verificación de receta": "Receta verificada por {nombre} el {dd/MM/yyyy HH:mm}"

Escenario: Devoluciones
  Dado que la venta tiene devoluciones
  Entonces veo "Devoluciones / Notas de crédito" (ver HU-DEV-007)

Escenario: Qué botones se ofrecen según el estado
  Entonces "Reimprimir Comprobante" aparece si hay comprobante y la venta no está "Anulada"
  Y "Cobrar Venta" aparece si está "Pendiente" y tengo "Ventas:Crear" y "Ventas:Actualizar"
  Y "Devolver" aparece si está "Completada", tiene comprobante, queda algo por devolver y tengo "Devoluciones:ProcesarDevolucion"
  Y "Anular Venta" aparece si está "Completada" o "Pendiente", sin devoluciones, y tengo "Ventas:Anular"

Escenario: Venta de otra sucursal o inexistente
  Dado que abro el identificador de una venta de otra sucursal
  Entonces veo "Error al cargar la venta" y la pantalla queda sin datos

Escenario: Sin permiso de lectura
  Dado que no tengo "Ventas:Leer"
  Entonces el sistema me lleva al Inicio con el aviso "No tienes permisos para acceder a esta pantalla."
```

> **Brecha conocida.** Cualquier fallo al abrir la venta (no existe, otra sucursal, error de red) muestra el mismo «Error al cargar la venta»; la API responde 404 `Venta.NotFound` y 400 `Venta.Forbidden` (RN-VEN-074). El estado se ve como texto junto a la fecha, no como chip. «Cobrar Venta» no exige caja abierta: si no la hay, el terminal avisa (HU-VEN-001).

### HU-VEN-028 — Exportar ventas a Excel

**Como** supervisor **quiero** exportar a Excel las ventas que busqué **para** revisarlas o archivarlas.

| | |
|---|---|
| **Módulo** | Ventas |
| **Pantalla** | `/pos` — botón «Exportar» |
| **Endpoint(s)** | `POST /api/v1/ventas/exportar` |
| **Permiso** | `Ventas:Leer` |
| **Reglas** | [RN-VEN-078](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-IMP-008, RN-IMP-009, RN-IMP-010, RN-IMP-011](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Exportar los resultados
  Dado que busqué ventas con filtros
  Cuando hago clic en "Exportar"
  Entonces se descarga el archivo "ventas.xlsx" con las mismas ventas del filtro (sin paginar)
  Y las columnas, en orden, son "Código de Venta", "Fecha", "Cliente", "Empleado", "Total (S/)", "Estado" e "Ítems"

Escenario: Formato de fecha
  Dado que el idioma es español
  Entonces la fecha se escribe "dd/MM/yyyy" y con otro idioma "MM/dd/yyyy"
  Y se formatea sobre la fecha y hora en UTC, sin convertir a hora local
  # No verificado: no se descargó el archivo

Escenario: Demasiadas filas
  Dado que el filtro devuelve más de 10 000 ventas
  Cuando hago clic en "Exportar"
  Entonces el archivo no se genera y veo "Error al buscar ventas"

Escenario: Falla la exportación
  Entonces veo "Error al buscar ventas"

Escenario: El estado sale como nombre del enumerado
  Entonces la columna "Estado" dice Completada, Anulada, Devuelta o Pendiente
```

> **Brecha conocida.** Una venta hecha después de las 19:00 hora de Lima (UTC−5) saldría con la fecha del día siguiente en el Excel (01c2 hallazgo 8; **No verificado** en ejecución). Cualquier fallo de la exportación se ve como «Error al buscar ventas».

## 9. Caja: apertura, cierre y arqueo

### HU-CAJ-001 — Abrir la caja del turno

**Como** cajero **quiero** abrir mi caja con el efectivo con que empiezo el turno **para** poder vender y que el sistema controle el efectivo que entra y sale de mi cajón.

| | |
|---|---|
| **Módulo** | Caja (Sesiones de Caja) |
| **Pantalla** | `/sesiones-caja` › «Nuevo» › `/sesiones-caja/nueva` — «Sesiones de Caja»; también el botón «Abrir caja» del Inicio |
| **Endpoint(s)** | `POST /api/v1/pos/apertura` |
| **Permiso** | `POS:AperturarCaja` (la ruta exige además `POS:Leer`) |
| **Reglas** | [RN-CAJ-001, RN-CAJ-002, RN-CAJ-003, RN-CAJ-004, RN-CAJ-005, RN-CAJ-006, RN-CAJ-025, RN-CAJ-026, RN-CAJ-027](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Abrir la caja con S/ 100.00
  Dado que tengo los permisos "POS:Leer" y "POS:AperturarCaja" y no tengo ninguna caja abierta
  Cuando entro a "/sesiones-caja", hago clic en "Nuevo" y veo "La caja se abrirá a tu nombre, en tu sucursal activa y con la hora actual."
  Y escribo "Monto Apertura (S/)" 100 y hago clic en "Aperturar"
  Entonces veo "Sesión aperturada correctamente" y vuelvo a la lista de sesiones
  Y existe una sesión "Abierta" a mi nombre, en mi sucursal base y con la hora del servidor
  Y los campos de cierre y de arqueo quedan vacíos
  Y se registra el evento de auditoría "CAJA_APERTURAR"

Escenario: La sucursal, el cajero y la hora no se eligen
  Entonces el campo "Cajero" está bloqueado con mi nombre
  Y el sistema usa la sucursal base de mi sesión de usuario y la hora del servidor

Escenario: Abrir con monto cero
  Cuando escribo "Monto Apertura (S/)" 0 (el valor inicial) y hago clic en "Aperturar"
  Entonces la caja se abre: cero es válido

Escenario: Monto vacío o negativo
  Cuando dejo el monto vacío o escribo -5
  Entonces veo "Ingresa un monto válido" y no se envía nada

Escenario: Ya tengo una caja abierta
  Dado que tengo una caja "Abierta" en cualquier sucursal
  Cuando intento abrir otra
  Entonces veo "Ya tienes una caja abierta. Ciérrala antes de abrir otra."
  Y esto vale aunque la otra caja sea de otra sucursal

Escenario: Dos aperturas al mismo tiempo
  Dado que envío dos aperturas simultáneas
  Entonces solo se crea una y la otra recibe el mismo aviso de caja ya abierta
  # No verificado en ejecución: lo garantiza un índice único filtrado de la base

Escenario: Mi sesión de usuario no tiene empleado o sucursal
  Entonces veo "Error al aperturar la sesión"

Escenario: Sin permiso para abrir caja
  Dado que no tengo "POS:AperturarCaja"
  Entonces no veo el botón "Nuevo"
  Y si escribo "/sesiones-caja/nueva" el sistema me lleva al Inicio con "No tienes permisos para acceder a esta pantalla."

Escenario: Estructura de la sesión de caja
  Entonces la base exige sucursal, empleado, fecha de apertura, monto de apertura y estado (Abierta, Cerrada o Cuadrada)
  Y los montos de apertura, calculado y declarado no pueden ser negativos (la diferencia sí puede)
  Y un índice único permite una sola sesión abierta por empleado y cualquier número de cerradas o cuadradas
  Y las ventas y devoluciones anotan la sesión que las cobró, que las anuló o de la que sale el reembolso; las tres columnas admiten vacío en datos anteriores
  Y una violación de estas restricciones que no sea la caja ya abierta se ve como "Error al aperturar la sesión"

Escenario: Abrir la caja desde el Inicio
  Dado que el Inicio muestra "No tienes una caja abierta"
  Cuando hago clic en "Abrir caja"
  Entonces voy a "/sesiones-caja/nueva"
```

> **Brecha conocida.** Cualquier error no traducido (por ejemplo, `Auth.Empleado`) se ve como «Error al aperturar la sesión». Tras abrir, la pantalla vuelve a la lista y no al terminal de venta. El script de base de datos define el monto como `DECIMAL(10,2)` (máximo 99 999 999,99) y la configuración de EF como `DECIMAL(12,2)`: qué hace la base con montos mayores o con más de 2 decimales no se verificó (RN-CAJ-003, RN-CAJ-025). Las claves de la sesión (sucursal, empleado, fecha, monto no negativo) tienen guards de entidad que ninguna ruta alcanza (RN-CAJ-006).

### HU-CAJ-002 — Abrir la caja desde el terminal de venta

**Como** cajero **quiero** abrir mi caja sin salir del terminal de venta **para** empezar a vender de inmediato.

| | |
|---|---|
| **Módulo** | Caja / Ventas |
| **Pantalla** | `/pos/nueva` — diálogo «Aperturar Caja» (botón «Aperturar Caja» de la tarjeta de estado) |
| **Endpoint(s)** | `POST /api/v1/pos/apertura` |
| **Permiso** | `POS:AperturarCaja` |
| **Reglas** | [RN-CAJ-002, RN-CAJ-003, RN-CAJ-004, RN-CAJ-005](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-VEN-003](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Abrir la caja desde el diálogo
  Dado que estoy en "/pos/nueva" sin caja abierta
  Cuando hago clic en "Aperturar Caja"
  Entonces veo el diálogo "Aperturar Caja": "Ingresa los datos para aperturar la caja de esta sucursal y continuar con la venta."
  Y el campo "Empleado" bloqueado con mi nombre y "Monto de Apertura (S/)" con valor inicial 0
  Cuando escribo 100 y hago clic en "Aperturar Caja"
  Entonces veo "Caja aperturada correctamente", el diálogo se cierra y el terminal se habilita

Escenario: Monto inválido
  Cuando dejo el monto vacío o negativo
  Entonces veo "Ingresa un monto válido" y no se envía nada

Escenario: Ya tengo una caja abierta
  Entonces veo "Ya tienes una caja abierta. Ciérrala antes de abrir otra."

Escenario: Otro fallo
  Entonces veo "No se pudo aperturar la caja"

Escenario: Cancelar
  Cuando hago clic en "Cancelar", pulso Esc o hago clic fuera
  Entonces el diálogo se cierra y el terminal no vuelve a comprobar la caja
```

### HU-CAJ-003 — Cerrar la caja y declarar el efectivo contado

**Como** cajero **quiero** cerrar mi caja declarando el efectivo que conté **para** terminar el turno y que el sistema compare lo contado con lo que debería haber.

| | |
|---|---|
| **Módulo** | Caja (Sesiones de Caja) |
| **Pantalla** | `/sesiones-caja/:id` — «Detalle de Sesión», botón «Cerrar Sesión», panel «Cerrar Sesión» |
| **Endpoint(s)** | `POST /api/v1/pos/cierre`, `GET /api/v1/pos/{id}` |
| **Permiso** | `POS:CerrarCaja` (la ruta exige `POS:Leer`) |
| **Reglas** | [RN-CAJ-001, RN-CAJ-007, RN-CAJ-008, RN-CAJ-009, RN-CAJ-010, RN-CAJ-015, RN-CAJ-017, RN-CAJ-018, RN-CAJ-019](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Cerrar la caja con el monto exacto (queda "Cuadrada")
  Dado que mi caja está "Abierta" y el efectivo esperado es S/ 150.00
  Cuando abro mi sesión, hago clic en "Cerrar Sesión", escribo "Monto Cierre Declarado (S/)" 150 y hago clic en "Confirmar Cierre"
  Entonces veo "Sesión cerrada correctamente" y vuelvo a la lista de sesiones
  Y la sesión queda "Cuadrada" con diferencia 0.00

Esquema del escenario: Resultado del cierre según lo declarado
  Dado que el efectivo esperado calculado por el sistema es S/ 150.00
  Cuando declaro <declarado>
  Entonces la diferencia de arqueo es <diferencia> y la sesión queda <estado>

  Ejemplos:
    | declarado | diferencia | estado   | lectura  |
    | 150.00    | 0.00       | Cuadrada | cuadre   |
    | 145.00    | -5.00      | Cerrada  | faltante |
    | 152.50    | 2.50       | Cerrada  | sobrante |
    | 150.01    | 0.01       | Cerrada  | sobrante de un centavo |

Escenario: No existe tolerancia
  Dado que declaro S/ 0.01 más de lo esperado
  Entonces la sesión queda "Cerrada" y no "Cuadrada"

Escenario: Qué se guarda al cerrar
  Entonces se guardan la hora de cierre (UTC del servidor), el monto declarado, el efectivo esperado ("Monto Cierre Calculado") y la diferencia con signo
  Y el estado final es definitivo: no se puede reabrir, editar ni eliminar la sesión
  Y se registra el evento de auditoría "CAJA_CERRAR" (el esperado y la diferencia no van en el detalle del evento)

Escenario: El panel de cierre no muestra el efectivo esperado
  Dado que la sesión está "Abierta"
  Entonces el detalle muestra "—" en calculado y diferencia
  Y el panel de cierre solo pide "Monto Cierre Declarado (S/)"
  Y puedo ver el esperado antes en el Inicio, en "Efectivo esperado" (ver HU-DAS-004)

Escenario: Monto declarado vacío o negativo
  Cuando dejo el campo vacío, lo toco o escribo un negativo
  Entonces veo "Ingresa el monto declarado al cierre" y no se envía nada

Escenario: Solo el titular o un administrador cierra
  Dado que la sesión es de otro cajero y no soy administrador
  Cuando hago clic en "Confirmar Cierre"
  Entonces veo "Solo el cajero que abrió la caja o un administrador puede cerrarla."
  Y esto vale aunque yo tenga "POS:CerrarCaja" y sea de la misma sucursal

Escenario: Un administrador cierra la caja de otro cajero
  Dado que soy administrador (rol "Admin")
  Entonces puedo cerrar la sesión de cualquier cajero y sucursal de mi empresa

Escenario: El botón se ofrece aunque la caja no sea mía
  Dado que la sesión es de otro cajero y tengo "POS:CerrarCaja"
  Entonces veo "Cerrar Sesión" y el rechazo llega recién al confirmar

Escenario: La sesión ya no está abierta
  Dado que la sesión está "Cerrada" o "Cuadrada"
  Entonces el detalle no ofrece "Cerrar Sesión"
  Y si la petición llega al servidor veo "Error al cerrar la sesión"

Escenario: Cancelar el panel
  Cuando hago clic en "Cancelar" en el panel de cierre
  Entonces el panel se oculta y no se cierra nada

Escenario: Hay ventas en espera al cerrar
  Dado que tengo ventas "Pendiente"
  Cuando cierro la caja
  Entonces el cierre no se bloquea ni avisa
  Y esas ventas no entran en el arqueo; las cobrará más tarde quien tenga su propia caja abierta y contarán en el arqueo de esa otra sesión

Escenario: El arqueo es una foto del momento del cierre
  Dado que cerré la caja con esperado S/ 150.00
  Cuando después se anula desde otra caja una venta cobrada en la mía
  Entonces mi sesión conserva el valor ya guardado
  Y una sesión cerrada no admite más ventas ni devoluciones
```

> **Brecha conocida.** El SPA no muestra el resultado del arqueo al cerrar: vuelve a la lista y el resultado se ve al reabrir el detalle (HU-CAJ-006). La diferencia no se rotula «faltante» ni «sobrante»: solo se ve el número con signo (los textos «FALTANTE EN CAJA» y «SOBRANTE EN CAJA» de `pos/es.json` no los usa ningún componente). Si una venta se crea mientras otro usuario cierra la misma caja, podría quedar fuera del arqueo (01c3 §6.1 punto 2, **No verificado**).

### HU-CAJ-004 — Calcular el efectivo esperado (arqueo)

**Como** cajero **quiero** que el sistema calcule cuánto efectivo debería haber en mi cajón **para** compararlo con lo que conté y detectar faltantes o sobrantes.

| | |
|---|---|
| **Módulo** | Caja |
| **Pantalla** | Sin pantalla propia: el esperado se ve en «Monto Cierre Calculado (S/)» tras el cierre (`/sesiones-caja/:id`) y en «Efectivo esperado» del Inicio |
| **Endpoint(s)** | `POST /api/v1/pos/cierre`, `GET /api/v1/dashboard/caja` |
| **Permiso** | `POS:CerrarCaja` / `POS:Leer` |
| **Reglas** | [RN-CAJ-011, RN-CAJ-012, RN-CAJ-013, RN-CAJ-014, RN-CAJ-017, RN-CAJ-018, RN-CAJ-027](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-VEN-006, RN-VEN-044, RN-VEN-047, RN-VEN-067](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Fórmula del efectivo esperado
  Entonces el efectivo esperado es el monto de apertura, más el efectivo neto de las ventas que entran, menos el efectivo neto de las ventas de otras cajas anuladas desde esta sesión, menos los reembolsos en efectivo de las devoluciones procesadas desde esta sesión

Escenario: Ejemplo sencillo con vuelto
  Dado que abrí con S/ 100.00
  Y vendí V1 de S/ 50.00 pagada con S/ 60.00 en efectivo (vuelto S/ 10.00)
  Y vendí V2 de S/ 30.00 pagada con tarjeta
  Entonces el efectivo neto de V1 es 60.00 menos 10.00 de vuelto, es decir 50.00
  Y el efectivo neto de V2 es 0.00
  Y el efectivo esperado es 100.00 + 50.00 + 0.00 = 150.00

Escenario: Ejemplo completo con anulaciones y reembolso
  Dado que abrí con S/ 100.00
  Y V1 de 50.00 en efectivo con vuelto 10.00 aporta + 50.00
  Y V2 de 30.00 con tarjeta aporta + 0.00
  Y V3 de 20.00 cobrada en efectivo en otra caja y anulada desde mi caja aporta - 20.00
  Y V4 de 40.00 en efectivo cobrada en mi caja y anulada después desde otra caja aporta + 40.00
  Y una devolución con reembolso en efectivo de 15.00 procesada desde mi caja resta 15.00
  Entonces el efectivo esperado es 100.00 + (50.00 + 0.00 - 20.00 + 40.00) - 15.00 = 155.00
  Y declarando 155.00 la sesión queda "Cuadrada", con 150.00 la diferencia es -5.00 y queda "Cerrada", y con 160.00 es +5.00 y queda "Cerrada"

Escenario: El vuelto se descuenta siempre del efectivo
  Dado una venta de S/ 50.00 pagada con S/ 40.00 por Yape / Plin y S/ 15.00 en efectivo
  Entonces la suma de pagos es 55.00, el vuelto es 5.00 y el efectivo neto es 15.00 - 5.00 = 10.00
  Y el pago por Yape / Plin no aporta efectivo

Escenario: Venta con cobertura de seguro
  Dado una venta de S/ 100.00 con cobertura de seguro de S/ 40.00 pagada con S/ 100.00 en efectivo
  Entonces el monto a pagar es 60.00, el vuelto 40.00 y el efectivo neto 60.00

Esquema del escenario: Qué ventas entran en el arqueo y con qué signo
  Dado una venta según dónde se cobró y dónde se anuló
  Entonces el caso <caso> entra <entra>

  Ejemplos:
    | caso | situación                                                              | entra                      |
    | A    | cobrada en esta sesión y "Completada" o "Devuelta"                      | con signo +                |
    | B    | cobrada en esta sesión y anulada desde esta misma sesión               | no entra (neto 0)          |
    | C    | cobrada en otra sesión y anulada desde esta                            | con signo - (sale de aquí) |
    | D    | cobrada en esta sesión y anulada desde otra sesión                     | con signo +                |
    | E    | cobrada en esta sesión, anulada y sin sesión de anulación (anterior al script 0028) | no entra      |

Escenario: Las ventas en espera nunca entran
  Dado una venta "Pendiente"
  Entonces no aporta nada al arqueo hasta que se cobre
  Y al cobrarla pertenece a la sesión de quien la cobra

Escenario: Un reembolso por medio digital no toca el arqueo
  Dado una devolución con método de reembolso "Yape / Plin", "Tarjeta Débito/Crédito" o "Transferencia"
  Entonces no se resta del efectivo esperado
  Y solo el reembolso en "Efectivo" lo resta, aunque la venta devuelta se haya cobrado en otra sesión

Escenario: Una venta totalmente devuelta sigue sumando
  Dado una venta que quedó "Devuelta"
  Entonces sigue entrando con signo + y lo que salió se resta aparte como reembolso

Escenario: Qué columnas lee el arqueo
  Entonces lee la sesión de cobro y la sesión de anulación de cada venta y la sesión de cada devolución, los tres datos que el sistema anota al operar
```

> **Brecha conocida.** El esperado se calcula una sola vez, al cerrar, y no se recalcula (RN-CAJ-018). El cajero no lo ve en el panel de cierre. Las ventas y devoluciones anteriores a los scripts `0016` y `0028` pueden no tener sesión anotada (RN-CAJ-027).

### HU-CAJ-005 — Entender por qué no se puede cerrar una caja

**Como** cajero **quiero** saber qué impide el cierre de mi caja **para** pedir ayuda con el dato correcto.

| | |
|---|---|
| **Módulo** | Caja |
| **Pantalla** | `/sesiones-caja/:id` — panel «Cerrar Sesión» |
| **Endpoint(s)** | `POST /api/v1/pos/cierre` |
| **Permiso** | `POS:CerrarCaja` |
| **Reglas** | [RN-CAJ-008, RN-CAJ-010, RN-CAJ-016, RN-CAJ-025](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Parcial (cuando el efectivo esperado es negativo no existe otra vía de cierre) |

**Criterios de aceptación**

```gherkin
Escenario: El efectivo esperado sería negativo
  Dado que abrí con S/ 10.00, no vendí en efectivo y procesé una devolución con reembolso en efectivo de S/ 15.00 de una venta cobrada en otra caja
  Entonces el efectivo esperado es 10.00 - 15.00 = -5.00
  Cuando intento cerrar declarando cualquier monto
  Entonces el sistema rechaza el cierre y veo "Error al cerrar la sesión"
  Y la sesión sigue "Abierta"
  Y el código no ofrece otra vía para cerrarla

Escenario: Dos cierres simultáneos
  Dado que dos usuarios cierran la misma caja a la vez
  Entonces solo se guarda uno y el otro ve "Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo."
  # No verificado en ejecución

Escenario: La sesión no existe o no es de una sucursal accesible
  Entonces el servidor responde 404 con código "Caja" y veo "Error al cerrar la sesión"

Escenario: La sesión no está "Abierta"
  Entonces veo "Error al cerrar la sesión"

Escenario: Falta el empleado en mi sesión de usuario
  Entonces veo "Error al cerrar la sesión"
```

> **Brecha conocida.** Un efectivo esperado negativo impide cerrar la caja y la pantalla solo dice «Error al cerrar la sesión» (el mensaje del servidor está en inglés y no llega al usuario); el usuario no puede resolverlo en pantalla (01c3 §6.3 hallazgo 1, proceso 6 §8.5). Tanto el cierre como la restricción de la base (`CHK_POS_Sesiones_Caja_Montos`) rechazan un calculado negativo.

### HU-CAJ-006 — Ver el resultado del arqueo de una sesión

**Como** cajero o supervisor **quiero** abrir una sesión y ver sus montos y su estado **para** revisar cómo cerró.

| | |
|---|---|
| **Módulo** | Caja |
| **Pantalla** | `/sesiones-caja/:id` — «Detalle de Sesión» |
| **Endpoint(s)** | `GET /api/v1/pos/{id}` |
| **Permiso** | `POS:Leer` |
| **Reglas** | [RN-CAJ-015, RN-CAJ-020, RN-CAJ-023](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Ver una sesión cerrada
  Dado que abro una sesión "Cerrada" con esperado 150.00 y declarado 145.00
  Entonces veo "Detalle de Sesión" con Sucursal, Cajero, "Fecha / Hora Apertura", "Fecha / Hora Cierre" y Estado "Cerrada"
  Y "Monto Apertura (S/)" 100.00, "Monto Cierre Declarado (S/)" 145.00, "Monto Cierre Calculado (S/)" 150.00 y "Diferencia Arqueo (S/)" -5.00
  Y todos los campos son de solo lectura

Escenario: Una sesión abierta
  Entonces los campos de cierre, calculado y diferencia muestran "—"
  Y si tengo "POS:CerrarCaja" veo "Cerrar Sesión"

Escenario: La diferencia positiva se muestra sin signo
  Dado que la diferencia es +5.00
  Entonces veo "5.00" sin el signo "+", y las negativas llevan "-"

Escenario: Sesión de una sucursal que no puedo ver
  Dado que la sesión es de una sucursal a la que no tengo acceso
  Entonces veo "Error al cargar la sesión de caja"

Escenario: La respuesta del detalle
  Entonces la API devuelve el objeto de resultado completo y la sesión viene en el campo "value"
```

> **Brecha conocida.** `GET Pos/{id}` devuelve el objeto `Result` completo en vez del valor directo (01c3 hallazgo 8). No hay botón para imprimir un reporte de arqueo ni textos de «Faltante», «Sobrante» o «Cuadre Exacto» en uso (los de `pos.reportDialog.*` y `pos.dialogs.*` son restos de un diálogo que ya no existe).

### HU-CAJ-007 — Buscar sesiones de caja

**Como** supervisor o cajero **quiero** buscar sesiones por estado y fecha de apertura **para** revisar cajas abiertas y cierres anteriores.

| | |
|---|---|
| **Módulo** | Caja |
| **Pantalla** | `/sesiones-caja` — «Sesiones de Caja» |
| **Endpoint(s)** | `GET /api/v1/pos/sesiones` |
| **Permiso** | `POS:Leer` |
| **Reglas** | [RN-CAJ-020, RN-CAJ-021, RN-CAJ-022](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-GEN-008, RN-GEN-009, RN-GEN-023](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Parcial (la API admite filtrar por sucursal y por cajero; la pantalla no ofrece esos filtros) |

**Criterios de aceptación**

```gherkin
Escenario: Buscar sesiones abiertas
  Dado que estoy en "/sesiones-caja" con el título de tarjeta "Búsqueda de Sesiones de Caja"
  Cuando elijo "Estado" = "Abierta" y hago clic en "Buscar"
  Entonces veo "Resultados (N)" con las columnas "Sucursal" (enlace al detalle), "Cajero", "Apertura", "Cierre" ("—" si está abierta), "Monto Apertura" y "Estado"
  Y el orden es por apertura descendente

Escenario: Filtros disponibles
  Entonces la pantalla ofrece "Estado" ("— Todos —", "Abierta", "Cerrada", "Cuadrada"), "Apertura desde" y "Apertura hasta"
  Y las fechas se interpretan como inicio y fin del día en mi zona horaria
  Y no hay filtros por sucursal ni por cajero

Escenario: Qué sesiones veo
  Dado que no soy administrador
  Entonces veo las sesiones de las sucursales a las que tengo acceso (la base, las asignadas a mi cuenta y a mis roles) y de todos los cajeros de esas sucursales
  Y si soy administrador veo las de todas las sucursales de la empresa

Escenario: Los resultados no aparecen hasta buscar
  Entonces la tabla se oculta hasta pulsar "Buscar" o hasta que haya estado guardado
  Y los tamaños de página son 10, 25 y 50
  Y "Limpiar" vacía el formulario y borra lo recordado

Escenario: Sin resultados
  Entonces veo "Sin resultados"

Escenario: Falla la búsqueda
  Entonces veo "Error al buscar sesiones de caja"

Escenario: Calculado, declarado y diferencia no están en la tabla
  Entonces se ven en el detalle (HU-CAJ-006) y en el Excel (HU-CAJ-008)

Escenario: Filtros de la API con un identificador vacío
  Dado que llamo a la API con sucursal o empleado igual al GUID vacío
  Entonces el validador responde 400 "GetSesionesQuery.SucursalId" o "GetSesionesQuery.EmpleadoId"
  Y pedir una sucursal no accesible devuelve una página vacía, no un error
```

> **Brecha conocida.** No existe una vista «solo mis sesiones» y cualquier usuario con `POS:Leer` ve las sesiones de todos los cajeros de sus sucursales; el Inicio entrega además el monto de apertura de las cajas ajenas aunque la pantalla solo muestre cajero, sucursal y hora (01c3 hallazgo 9). Qué nombre muestra el listado si la sucursal o el empleado fueron eliminados no se verificó. El terminal de venta usa este mismo listado para encontrar la caja del cajero (HU-VEN-001).

### HU-CAJ-008 — Exportar sesiones de caja a Excel

**Como** supervisor **quiero** exportar las sesiones que busqué **para** auditar los cierres fuera del sistema.

| | |
|---|---|
| **Módulo** | Caja |
| **Pantalla** | `/sesiones-caja` — botón «Exportar» |
| **Endpoint(s)** | `POST /api/v1/pos/sesiones/exportar` |
| **Permiso** | `POS:Leer` |
| **Reglas** | [RN-CAJ-024](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-IMP-008, RN-IMP-009, RN-IMP-010, RN-IMP-011](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Exportar las sesiones del filtro
  Dado que busqué sesiones
  Cuando hago clic en "Exportar"
  Entonces se descarga "sesiones-caja.xlsx" con 9 columnas, en este orden: "Sucursal", "Cajero", "Apertura", "Cierre", "Monto Apertura", "Monto Cierre Declarado (S/)", "Monto Cierre Calculado (S/)", "Diferencia Arqueo (S/)" y "Estado"
  Y respeta el estado y las fechas del filtro, con el mismo alcance de sucursales

Escenario: El filtro por empleado no se aplica
  Entonces la exportación no admite el filtro de empleado, aunque el listado de la API sí

Escenario: Formato de fecha
  Dado que el idioma es español
  Entonces las fechas salen "dd/MM/yyyy HH:mm" y en otro idioma "MM/dd/yyyy HH:mm"
  Y se escriben en UTC sin convertir a hora local
  # No verificado: no se descargó el archivo

Escenario: Más de 10 000 sesiones
  Entonces el archivo no se genera y veo "Error al buscar sesiones de caja"

Escenario: Falla la exportación
  Entonces veo "Error al buscar sesiones de caja"
```

> **Brecha conocida.** Una apertura a las 20:00 hora de Lima se exportaría con la hora y el día del día siguiente (01c3 hallazgo 3; **No verificado**).

## 10. Devoluciones

### HU-DEV-001 — Devolver parte o todo lo vendido

**Como** cajero **quiero** procesar la devolución de una venta cobrada **para** reembolsar al cliente, reponer la mercadería y dejar la nota de crédito emitida.

| | |
|---|---|
| **Módulo** | Devoluciones |
| **Pantalla** | `/pos/:id` — botón «Devolver» y diálogo «Procesar devolución» |
| **Endpoint(s)** | `POST /api/v1/devoluciones` (el campo `comprobanteOrigenId` lleva el id de la **venta**) |
| **Permiso** | `Devoluciones:ProcesarDevolucion` (y `Ventas:Leer` para llegar al detalle) |
| **Reglas** | [RN-DEV-001, RN-DEV-002, RN-DEV-009, RN-DEV-010, RN-DEV-020, RN-DEV-025, RN-DEV-026, RN-DEV-027, RN-DEV-028, RN-DEV-029](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-VEN-066](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Devolución parcial
  Dado que la venta "B001-00000001" está "Completada": 100 unidades de "Paracetamol Demo 500 mg" a S/ 0.50 (S/ 50.00)
  Y tengo "Devoluciones:ProcesarDevolucion" y mi propia caja abierta en la sucursal de la venta
  Cuando abro el detalle y hago clic en "Devolver"
  Entonces se abre "Procesar devolución — B001-00000001" con "Indica cuántas unidades devuelve el cliente en cada línea y su destino. Se emitirá una nota de crédito y el reembolso saldrá de tu caja abierta."
  Cuando escribo "A devolver" 30, elijo "Método de reembolso" "Efectivo", escribo el "Sustento" "Producto en mal estado" y hago clic en "Procesar devolución"
  Entonces veo "Devolución procesada y nota de crédito emitida."
  Y el detalle se recarga con "Devuelto" 30 en la línea y la sección "Devoluciones / Notas de crédito"
  Y la venta sigue "Completada" porque quedan 70 unidades devolvibles

Escenario: Devolución total en dos pasos
  Dado que ya devolví 30 de las 100 unidades
  Cuando hago clic en "Devolver", en "Devolver todo lo pendiente" (70 unidades) y confirmo con el sustento
  Entonces el motivo propuesto es "06 — Devolución total"
  Y la venta pasa a "Devuelta"
  Y una venta "Devuelta" ya no admite más devoluciones ni anulación

Escenario: Quién autoriza
  Entonces la devolución guarda como empleado que autoriza a quien tiene la sesión iniciada
  Y no se pide supervisor ni PIN

Escenario: Qué no cambia al devolver
  Entonces los pagos, el total, la boleta y su estado no cambian
  Y la receta médica y sus dispensaciones no se tocan (ver HU-VEN-010)

Escenario: Todo o nada
  Dado que algo falla antes de guardar
  Entonces no queda nada: ni la devolución, ni el reintegro de stock, ni la nota de crédito, ni el cambio de estado, ni la auditoría

Escenario: Repetir la misma devolución
  Dado que hago clic dos veces en "Procesar devolución" o repito la petición
  Entonces el botón se deshabilita mientras guarda
  Y el segundo intento recibe "La cantidad supera lo que queda por devolver en alguna línea. Actualiza la pantalla e inténtalo de nuevo." si ya no queda saldo, o el aviso de conflicto de concurrencia

Escenario: Dos devoluciones simultáneas de la misma venta
  Entonces solo se guarda una y la otra ve "Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo."
  # No verificado en ejecución

Escenario: Auditoría
  Entonces se registra el evento "DEVOLUCION_PROCESAR" con el id de la venta, el número de líneas del pedido y el método de reembolso

Escenario: El botón "Devolver" solo aparece cuando corresponde
  Entonces aparece si la venta está "Completada", tiene comprobante, queda alguna unidad devolvible y tengo "Devoluciones:ProcesarDevolucion"
  Y no aparece en ventas "Pendiente", "Anulada" ni "Devuelta"

Escenario: Respuesta de la API
  Entonces responde 200 con el identificador de la devolución

Escenario: Una venta con devoluciones ya no se puede anular
  Dado que la venta tiene al menos una devolución
  Entonces el detalle no ofrece "Anular Venta" (ver HU-VEN-025)

Escenario: Error no traducido
  Dado que ocurre cualquier otro error (por ejemplo, el token no trae empleado o sucursal)
  Entonces veo "Error al procesar la devolución."
```

> **Brecha conocida.** El diálogo no tiene listado ni pantalla de devoluciones: solo se llega desde el detalle de una venta (ver HU-DEV-007 y HU-DEV-008). El permiso `Devoluciones:ProcesarDevolucion` no lo siembra ningún rol: hay que concederlo (RN-DEV-001). No hay plazo máximo para devolver ni límite de devoluciones por venta (RN-DEV-008). La operación no tiene clave de idempotencia (RN-DEV-028).

### HU-DEV-002 — Elegir la cantidad y el destino de cada línea devuelta

**Como** cajero **quiero** indicar cuánto devuelve el cliente de cada línea y qué se hace con la mercadería **para** reponer el stock o apartarla.

| | |
|---|---|
| **Módulo** | Devoluciones |
| **Pantalla** | `/pos/:id` — tabla del diálogo «Procesar devolución» (Producto, Lote, Vendido, Devuelto, Devolvible, A devolver, Destino) |
| **Endpoint(s)** | `POST /api/v1/devoluciones` |
| **Permiso** | `Devoluciones:ProcesarDevolucion` |
| **Reglas** | [RN-DEV-009, RN-DEV-010, RN-DEV-011, RN-DEV-012, RN-DEV-013, RN-DEV-023, RN-DEV-024, RN-DEV-034](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CUA-011](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-STK-011](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) |
| **Estado** | Parcial (el destino «Cuarentena DIGEMID» solo queda anotado: no repone stock ni crea el registro de cuarentena) |

**Criterios de aceptación**

```gherkin
Escenario: La tabla lista solo lo devolvible
  Dado que una línea vendió 10 unidades y ya se devolvieron 4
  Entonces la tabla muestra Vendido 10, Devuelto 4 y Devolvible 6
  Y las líneas totalmente devueltas no aparecen
  Y las cantidades son siempre en unidades base, aunque la línea se vendiera por presentación

Escenario: El máximo es lo devolvible
  Dado que Devolvible es 6
  Cuando escribo "A devolver" 6
  Entonces es válido y la línea queda completa
  Y si escribo 7, al salir del campo la cantidad se ajusta a 6
  Y si la petición llega con 7, el servidor responde "La cantidad supera lo que queda por devolver en alguna línea. Actualiza la pantalla e inténtalo de nuevo."

Escenario: Ninguna cantidad
  Dado que todas las cantidades están en 0
  Cuando hago clic en "Procesar devolución"
  Entonces veo "Ingresa al menos una cantidad a devolver." y no se envía nada
  Y solo se envían las líneas con cantidad mayor que 0

Escenario: Devolver todo lo pendiente
  Cuando hago clic en "Devolver todo lo pendiente"
  Entonces cada línea toma su cantidad devolvible

Escenario: Destino «Reingreso a venta»
  Dado que el destino es "Reingreso a venta" (valor por defecto)
  Cuando devuelvo 30 unidades
  Entonces las 30 unidades se suman al stock del mismo lote en la sucursal de la venta
  Y si el lote venció o quedó retenido en cuarentena después de la venta, recupera su stock físico pero deja de ser vendible

Escenario: Si la fila de stock ya no existe
  Dado que ya no existe la fila de stock del lote en la sucursal
  Cuando devuelvo con "Reingreso a venta"
  Entonces la devolución, la nota de crédito y el reembolso se registran igual
  Y el stock no se repone, sin ningún aviso

Escenario: Destino «Cuarentena DIGEMID»
  Dado que elijo el destino "Cuarentena DIGEMID" para una línea
  Cuando procesa la devolución
  Entonces la línea guarda el destino pero no se repone stock ni se crea un registro de cuarentena
  Y la nota de crédito y el reembolso se calculan igual
  Y el lote no aparece en "Lotes en cuarentena" del Inicio hasta que alguien lo aísle con la API (ver HU-OPS-006)

Escenario: La misma línea repetida en la petición (API)
  Dado que la petición trae la misma línea de venta dos veces
  Entonces para el tope se suman sus cantidades
  Y cada repetición se guarda como una línea de devolución propia

Escenario: Una línea que no es de la venta
  Dado que la petición trae una línea inexistente, de otra venta o de otra empresa
  Entonces veo "Una de las líneas ya no pertenece a la venta."

Escenario: Cantidad con más de 2 decimales
  # No verificado: qué hace la base de datos con más de 2 decimales (columna DECIMAL(8,2))
```

> **Brecha conocida.** «Cuarentena DIGEMID» no crea el aislamiento (01c3 hallazgo 4; RN-DEV-024, RN-CUA-011). Si la fila de stock ya no existe, el reintegro se omite sin aviso (hallazgo 6).

### HU-DEV-003 — Reembolsar al cliente

**Como** cajero **quiero** registrar cuánto y por qué medio se reembolsa al cliente **para** que mi caja refleje la salida de efectivo.

| | |
|---|---|
| **Módulo** | Devoluciones / Caja |
| **Pantalla** | `/pos/:id` — «Método de reembolso» y «Reembolso estimado» del diálogo |
| **Endpoint(s)** | `POST /api/v1/devoluciones` |
| **Permiso** | `Devoluciones:ProcesarDevolucion` |
| **Reglas** | [RN-DEV-005, RN-DEV-016, RN-DEV-017, RN-DEV-018, RN-DEV-019, RN-DEV-035, RN-DEV-036](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CAJ-014](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Reembolso en efectivo de una devolución parcial
  Dado que devuelvo 30 unidades de un producto a S/ 0.50 en una venta sin seguro
  Entonces el "Reembolso estimado" del diálogo es S/ 15.00
  Y el monto devuelto es 30 × 0.50 = 15.00 y el monto reembolsado también 15.00
  Y con método "Efectivo" el esperado de mi caja baja en 15.00 (ver HU-CAJ-004)

Escenario: El método de reembolso
  Entonces el selector ofrece "Efectivo" (valor inicial), "Yape / Plin", "Tarjeta Débito/Crédito" y "Transferencia"
  Y si la petición omite el método, el servidor asume efectivo

Escenario: Reembolso por un medio digital
  Dado que elijo "Yape / Plin"
  Entonces el reembolso queda registrado con ese método y no resta del arqueo

Escenario: El método no tiene que coincidir con cómo pagó el cliente
  Dado que la venta se pagó con tarjeta
  Cuando elijo reembolsar en "Efectivo"
  Entonces el sistema lo permite
  Y el efectivo sale de mi caja aunque la venta no haya entrado en efectivo

Escenario: El sistema solo registra el dinero, no lo mueve
  Entonces no se comprueba que el cajón tenga efectivo suficiente

Escenario: Redondeo del monto devuelto
  Dado que devuelvo 0.25 unidades a S/ 0.50
  Entonces el monto devuelto es 0.125, redondeado a 0.12 (a 2 decimales, la mitad hacia el par)
  Y con 0.27 unidades el resultado 0.135 se redondea a 0.14

Escenario: Venta con cobertura de seguro (solo API)
  Dado una venta de 10 unidades a S/ 3.00 (total S/ 30.00) con cobertura de seguro de S/ 12.00
  Cuando se devuelven 4 unidades
  Entonces el monto devuelto es 12.00 y la proporción que pagó el cliente es (30.00 - 12.00) ÷ 30.00 = 0.6
  Y el reembolso al cliente es 12.00 × 0.6 = 7.20
  Y la nota de crédito va por 12.00

Escenario: El estimado puede no coincidir con el real
  Dado que la venta tuvo cobertura de seguro o el cálculo requiere redondeo
  Entonces el "Reembolso estimado" del diálogo (cantidad × precio, sin redondear ni descontar el seguro) puede diferir del reembolso real

Escenario: Dónde se ve el reembolso
  Entonces el detalle de la venta muestra "Reembolsado (S/)" y "Método de reembolso" en "Devoluciones / Notas de crédito"

Escenario: Hace falta caja propia abierta para cualquier método
  Dado que no tengo caja abierta
  Cuando intento devolver con "Transferencia"
  Entonces veo "Necesitas una caja abierta para procesar la devolución."
```

> **Brecha conocida.** El reembolso de la devolución no se compara con los pagos originales (RN-DEV-018); el «Reembolso estimado» no aplica el seguro ni el redondeo del servidor (01c3 hallazgo 10). La cobertura de seguro hoy no se captura desde la pantalla (HU-VEN-019). El monto reembolsado es nulo en devoluciones anteriores al script `0016` (RN-DEV-035).

### HU-DEV-004 — Emitir la nota de crédito de la devolución

**Como** cajero **quiero** que cada devolución emita su nota de crédito **para** acreditar la boleta de la venta.

| | |
|---|---|
| **Módulo** | Devoluciones / Comprobantes |
| **Pantalla** | `/pos/:id` — sección «Devoluciones / Notas de crédito» |
| **Endpoint(s)** | `POST /api/v1/devoluciones` |
| **Permiso** | `Devoluciones:ProcesarDevolucion` |
| **Reglas** | [RN-DEV-006, RN-DEV-007, RN-DEV-020, RN-DEV-021, RN-DEV-022, RN-DEV-033, RN-DEV-035, RN-DEV-036, RN-DEV-037](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CMP-015](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-SER-004, RN-SER-013](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Parcial (la nota se registra en SOFIA; no se envía a SUNAT y la pantalla solo muestra su número: no la imprime ni la muestra como documento) |

**Criterios de aceptación**

```gherkin
Escenario: Una nota de crédito por devolución
  Dado que la sucursal tiene una serie de nota de crédito activa con prefijo "BC01"
  Cuando proceso una devolución con monto devuelto S/ 15.00
  Entonces se emite una nota de crédito con número "BC01-00000001"
  Y nace con estado "NoEnviado", sin hash, XML, constancia ni URL
  Y no hay devolución sin nota de crédito

Escenario: Importes de la nota
  Dado que el monto devuelto es S/ 15.00
  Entonces la base gravada es 12.71 y el IGV es 2.29 (la base es el total dividido entre 1,18 redondeado a 2 decimales con la mitad hacia arriba)
  Y el monto exonerado es 0.00
  Y con un monto devuelto de 12.00 la base es 10.17 y el IGV 1.83
  Y la nota va por el 100 % del crédito, incluida la parte de la aseguradora

Escenario: Datos del cliente de la nota
  Entonces la nota repite el tipo de documento, el número y el nombre del cliente de la boleta o factura que revierte

Escenario: Qué ve el cajero de la nota
  Entonces la sección "Devoluciones / Notas de crédito" muestra "Nota de crédito" (el número o "—"), "Fecha" ("dd/MM/yyyy HH:mm"), "Reembolsado (S/)" y "Método de reembolso"
  Y la nota de crédito no cambia el código de venta ni se toma como "el comprobante" de la venta

Escenario: La venta no tiene boleta ni factura
  Dado que la venta no tiene boleta ni factura emitida
  Entonces veo "La venta no tiene boleta ni factura emitida que acreditar."

Escenario: La sucursal no tiene serie de nota de crédito activa
  Entonces veo "La sucursal no tiene una serie de nota de crédito activa. Configúrala en Series Fiscales."
  Y no se consume correlativo

Escenario: La nota se emite al final
  Entonces el correlativo se consume solo cuando ya se validó todo lo demás
  Y si el guardado lanza una excepción, el aumento también se revierte

Escenario: Boletas y notas comparten numeración única
  Entonces el índice único por serie y correlativo vale también para las notas, y una venta tiene una boleta o factura y cero o más notas

Escenario: Estructura de las devoluciones
  Entonces la base exige comprobante de origen, empleado que autoriza, motivo (2 caracteres) y sustento (hasta 255), y las líneas exigen cantidad mayor que 0 y un destino que sea solo "Reingreso_Venta" o "Cuarentena_DIGEMID"
  Y el monto reembolsado no puede ser negativo y se guarda con 2 decimales; el método se guarda como EFECTIVO, YAPE_PLIN, TARJETA o TRANSFERENCIA
  Y no hay restricción sobre el motivo ni el método ni unicidad: una venta admite varias devoluciones
  Y una violación de estas restricciones se ve como "Error al procesar la devolución."
  Y las comprobaciones de la entidad (comprobante, empleado, sesión, monto) no pueden fallar con los datos que arma el servidor, salvo el motivo de 1 carácter
```

> **Brecha conocida.** No se envía nada a SUNAT. La serie de nota de crédito activa de la sucursal se usa tanto para devoluciones de boletas como de facturas: no se compara con el comprobante de origen y el prefijo `B`/`F` que exige RN-SER-004 no se contrasta (01c3 hallazgo 6). El SPA no imprime ni muestra la nota de crédito como documento (RN-DEV-022).

### HU-DEV-005 — Indicar el motivo SUNAT y el sustento

**Como** cajero **quiero** registrar por qué se devuelve **para** dejar constancia de la razón y cumplir con el motivo de la nota de crédito.

| | |
|---|---|
| **Módulo** | Devoluciones |
| **Pantalla** | `/pos/:id` — campos «Motivo SUNAT» y «Sustento» del diálogo |
| **Endpoint(s)** | `POST /api/v1/devoluciones` |
| **Permiso** | `Devoluciones:ProcesarDevolucion` |
| **Reglas** | [RN-DEV-014, RN-DEV-015, RN-DEV-033](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-GEN-010](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: El motivo se propone según las cantidades
  Cuando cada línea tiene su cantidad devolvible completa
  Entonces "Motivo SUNAT" propone "06 — Devolución total"
  Y en cualquier otro caso propone "07 — Devolución por ítem"
  Y puedo cambiarlo a mano, pero el siguiente cambio de cantidades lo vuelve a fijar

Escenario: Solo hay dos motivos
  Entonces el selector ofrece "06 — Devolución total" y "07 — Devolución por ítem"
  Y el servidor acepta cualquier par de caracteres: no valida contra el catálogo SUNAT ni que "06" coincida con una devolución total

Escenario: Sustento obligatorio
  Cuando dejo el "Sustento" vacío y hago clic en "Procesar devolución"
  Entonces veo "El sustento es obligatorio." y no se envía nada

Escenario: Sustento de solo espacios
  Cuando escribo solo espacios en "Sustento"
  Entonces el botón no hace nada y no aparece mensaje

Escenario: Largo del sustento
  Entonces el campo corta la escritura en 255 caracteres y se recorta de espacios al enviar

Escenario: Motivo de un solo carácter (API)
  Dado que la petición trae un motivo de 1 carácter
  Entonces el validador lo acepta (solo exige no vacío y como máximo 2), la entidad lo rechaza y veo "Error al procesar la devolución."

Escenario: Texto con HTML
  Dado que el sustento incluye etiquetas HTML
  Entonces el servidor las limpia antes de validar y guardar
```

> **Brecha conocida.** El mensaje de la entidad para un motivo de un carácter está en inglés y no llega al usuario; el motivo no se valida contra ningún catálogo (01c3 hallazgo 6).

### HU-DEV-006 — Saber por qué no se puede devolver una venta

**Como** cajero **quiero** entender qué condición impide la devolución **para** resolverla (abrir mi caja, cambiar de sucursal, etc.).

| | |
|---|---|
| **Módulo** | Devoluciones |
| **Pantalla** | `/pos/:id` — diálogo «Procesar devolución» |
| **Endpoint(s)** | `POST /api/v1/devoluciones` |
| **Permiso** | `Devoluciones:ProcesarDevolucion` |
| **Reglas** | [RN-DEV-002, RN-DEV-003, RN-DEV-004, RN-DEV-005, RN-DEV-006, RN-DEV-007, RN-DEV-008](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Esquema del escenario: Mensajes de rechazo de una devolución
  Dado que intento procesar una devolución y ocurre <situación>
  Entonces veo <mensaje>

  Ejemplos:
    | situación                                       | mensaje                                                                                                          |
    | no tengo caja propia abierta en la sucursal     | Necesitas una caja abierta para procesar la devolución.                                                          |
    | la venta es de otra sucursal (403)              | Solo puedes devolver ventas de tu propia sucursal.                                                               |
    | la venta ya no existe                           | La venta ya no existe.                                                                                           |
    | la venta no tiene boleta ni factura             | La venta no tiene boleta ni factura emitida que acreditar.                                                       |
    | la sucursal no tiene serie de nota de crédito   | La sucursal no tiene una serie de nota de crédito activa. Configúrala en Series Fiscales.                        |
    | la venta está "Anulada" o "Devuelta"            | Solo se pueden devolver ventas completadas.                                                                      |
    | la cantidad supera lo devolvible                | La cantidad supera lo que queda por devolver en alguna línea. Actualiza la pantalla e inténtalo de nuevo.        |
    | una línea no pertenece a la venta               | Una de las líneas ya no pertenece a la venta.                                                                    |
    | otra operación modificó los datos               | Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.                 |
    | cualquier otro error                            | Error al procesar la devolución.                                                                                 |

Escenario: Orden en que se evalúan las condiciones
  Dado que no tengo caja abierta y además la venta no tiene comprobante
  Cuando intento devolver
  Entonces solo veo el aviso de la caja, porque esa comprobación va antes que las de comprobante, estado y cantidades

Escenario: Una venta en espera
  Dado que la venta está "Pendiente"
  Entonces no se ofrece "Devolver"
  Y si la petición llega al servidor, se detiene por no tener boleta ni factura emitida

Escenario: El Admin tampoco devuelve ventas de otra sucursal
  Dado que soy administrador y la venta es de otra sucursal distinta a la de mi sesión de usuario
  Entonces veo "Solo puedes devolver ventas de tu propia sucursal."

Escenario: No hay plazo ni límite de devoluciones
  Dado que la venta es antigua y todavía queda cantidad devolvible
  Entonces el sistema permite seguir devolviendo
```

### HU-DEV-007 — Ver las devoluciones de una venta

**Como** cajero o supervisor **quiero** ver qué se devolvió de una venta **para** saber cuánto queda por devolver y a quién se reembolsó.

| | |
|---|---|
| **Módulo** | Devoluciones |
| **Pantalla** | `/pos/:id` — columna «Devuelto» y sección «Devoluciones / Notas de crédito» |
| **Endpoint(s)** | `GET /api/v1/ventas/{id}` |
| **Permiso** | `Ventas:Leer` |
| **Reglas** | [RN-DEV-022, RN-DEV-032](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-VEN-074](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Parcial (solo se ven dentro del detalle de la venta; no existe pantalla de listado ni de detalle de devoluciones) |

**Criterios de aceptación**

```gherkin
Escenario: Ver lo devuelto en la línea
  Dado que devolví 30 unidades de una línea de 100
  Entonces la columna "Devuelto" de "Productos Vendidos" muestra 30

Escenario: Ver las devoluciones de la venta
  Dado que la venta tiene dos devoluciones
  Entonces veo "Devoluciones / Notas de crédito" con una fila por devolución, de la más antigua a la más reciente
  Y cada fila muestra "Nota de crédito" (número o "—"), "Fecha", "Reembolsado (S/)" y "Método de reembolso"

Escenario: Venta sin devoluciones
  Entonces la sección "Devoluciones / Notas de crédito" no aparece

Escenario: El dashboard cuenta las devoluciones del día
  Entonces el Inicio muestra el número y el monto de las devoluciones de hoy (ver HU-DAS-007)
```

### HU-DEV-008 — Consultar devoluciones por API

**Como** integrador **quiero** listar devoluciones y consultar una por identificador **para** integrarlas con otros sistemas.

| | |
|---|---|
| **Módulo** | Devoluciones |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `GET /api/v1/devoluciones`, `GET /api/v1/devoluciones/{id}` |
| **Permiso** | `Devoluciones:Leer` |
| **Reglas** | [RN-DEV-030, RN-DEV-031, RN-DEV-032](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Listar devoluciones
  Dado que tengo "Devoluciones:Leer"
  Cuando llamo a "GET /api/v1/devoluciones" con fechas desde y hasta
  Entonces recibo las devoluciones de las sucursales a las que tengo acceso, ordenadas por fecha descendente
  Y cada fila trae solo identificador, comprobante de origen, empleado que autoriza, fecha, motivo SUNAT y sustento
  Y no trae monto, método de reembolso, nota de crédito, líneas ni destino
  Y la paginación es de 10 por defecto

Escenario: Filtro por empleado vacío
  Dado que envío el empleado que autoriza igual al GUID vacío
  Entonces recibo 400 "GetDevolucionesQuery.EmpleadoAutorizaId"

Escenario: Consultar una devolución
  Cuando llamo a "GET /api/v1/devoluciones/{id}" de una devolución de una sucursal accesible
  Entonces recibo 200 con el objeto de resultado completo y los mismos seis campos en "value"

Escenario: La devolución no existe
  Entonces recibo 404 con el código "Devolucion.NotFound"

Escenario: No existe exportación
  Entonces no hay endpoint para exportar devoluciones
```

> **Brecha conocida.** Ni el listado ni el detalle tienen pantalla; el SPA solo usa `POST`. `GET Devoluciones/{id}` devuelve el objeto `Result` completo y el listado el valor directo (01c3 hallazgo 8).

## 11. Dashboard (Inicio)

### HU-DAS-001 — Ver el resumen de ventas de hoy

**Como** cajero o supervisor **quiero** ver lo vendido hoy contra ayer **para** saber cómo va el día.

| | |
|---|---|
| **Módulo** | Dashboard |
| **Pantalla** | `/dashboard` — bloque «Ventas de hoy» |
| **Endpoint(s)** | `GET /api/v1/dashboard/ventas?hoy=` |
| **Permiso** | `Ventas:Leer` |
| **Reglas** | [RN-DAS-002, RN-DAS-004, RN-DAS-005, RN-DAS-006, RN-DAS-007, RN-DAS-008, RN-DAS-009](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Vendido hoy contra ayer
  Dado que hoy se vendió S/ 80.00 en 2 ventas y ayer S/ 64.00
  Entonces veo "Vendido hoy" S/ 80.00
  Y la variación "+25.0% vs. ayer · Ayer: S/ 64.00" con una flecha hacia arriba
  # No verificado: el separador decimal exacto depende del idioma de la aplicación

Escenario: Ayer no vendió nada
  Dado que ayer se vendió S/ 0.00
  Entonces solo veo "Ayer: S/ 0.00", sin porcentaje

Escenario: Comprobantes y ticket promedio
  Dado que hoy hubo 2 ventas por S/ 80.00
  Entonces "Comprobantes" muestra 2 y "Ticket promedio: S/ 40.00"
  Y si no hubo ventas, el ticket promedio es 0

Escenario: Qué ventas cuentan
  Entonces cuentan las ventas "Completada" y "Devuelta" de las sucursales del alcance
  Y no cuentan las "Anulada" ni las "Pendiente"
  Y el monto es siempre el total bruto de la venta, antes de restar la cobertura de seguro y sin restar devoluciones
  Y veo "Los montos vendidos son brutos: las devoluciones se muestran aparte."

Escenario: Mis ventas de hoy
  Dado que mi sesión de usuario tiene empleado
  Entonces veo "Mis ventas de hoy" con el monto de mis ventas y "{count} comprobantes"
  Y si no tiene empleado, la tarjeta no aparece

Escenario: Ventas en espera
  Entonces "Ventas en espera" muestra el número de ventas "Pendiente" de cualquier fecha y "Por cobrar: S/ …"

Escenario: Qué es "hoy"
  Entonces "hoy" es el día local de mi navegador (de medianoche a medianoche, 24 horas exactas), enviado al servidor como parámetro
  Y dos usuarios en zonas horarias distintas ven "hoy" distinto
  Y el servidor rechaza un "hoy" a más de 36 horas de su hora, en cuyo caso la sección muestra "No se pudo cargar esta sección." con "Reintentar"
```

> **Brecha conocida.** La tarjeta «Vendido hoy» incluye las ventas «Devuelta» y el botón «Ver ventas de hoy» filtra solo «Completada» (HU-DAS-010). Los montos siguen siendo brutos tras una devolución (01c3 hallazgo 7). Un reloj del equipo desfasado más de 36 horas deja sin datos las secciones de ventas, inventario y devoluciones.

### HU-DAS-002 — Ver lo cobrado por método de pago y por sucursal

**Como** supervisor **quiero** ver cuánto se cobró por cada método y por cada sucursal **para** conciliar los ingresos del día.

| | |
|---|---|
| **Módulo** | Dashboard |
| **Pantalla** | `/dashboard` — tarjetas «Cobrado por método de pago» y «Ventas por sucursal» |
| **Endpoint(s)** | `GET /api/v1/dashboard/ventas?hoy=` |
| **Permiso** | `Ventas:Leer` |
| **Reglas** | [RN-DAS-003, RN-DAS-010, RN-DAS-011](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Cobrado por método de pago
  Dado que hoy hubo la venta A de S/ 50.00 pagada con S/ 60.00 en efectivo (vuelto S/ 10.00)
  Y la venta B de S/ 30.00 pagada con S/ 20.00 en tarjeta y S/ 10.00 en Yape / Plin
  Entonces veo siempre cuatro filas, en este orden: "Efectivo" S/ 50.00, "Yape / Plin" S/ 10.00, "Tarjeta" S/ 20.00 y "Transferencia" S/ 0.00
  Y cada barra es el porcentaje sobre la suma de las cuatro filas (62.5 %, 12.5 %, 25.0 % y 0.0 %)
  Y leo "El efectivo ya descuenta el vuelto entregado."

Escenario: Las ventas anuladas no suman y los reembolsos no restan
  Dado que una venta de hoy se anuló
  Entonces sus pagos no se suman
  Y los reembolsos de devoluciones tampoco se restan de esta tarjeta

Escenario: Ventas por sucursal para varias sucursales
  Dado que mi alcance tiene más de una sucursal
  Entonces veo la tabla "Ventas por sucursal" con "Sucursal", "Comprobantes" y "Vendido", una fila por sucursal ordenada por nombre (con 0 si no vendió)
  Y el pie dice "Total empresa" si soy administrador o "Total de tus sucursales" en otro caso
  Y el pie suma lo mismo que "Vendido hoy" y "Comprobantes"

Escenario: Una sola sucursal
  Dado que mi alcance es una sola sucursal (incluido un administrador cuya empresa tiene una)
  Entonces no veo "Ventas por sucursal"

Escenario: Alcance sin sucursales
  Dado que mi alcance no tiene sucursales
  Entonces todas las cifras valen 0
```

### HU-DAS-003 — Ver la tendencia de 7 días y los productos más vendidos

**Como** supervisor **quiero** ver la tendencia semanal y los productos que más rotan **para** decidir compras y promociones.

| | |
|---|---|
| **Módulo** | Dashboard |
| **Pantalla** | `/dashboard` — «Ventas de los últimos 7 días» y «Productos más vendidos en 7 días» |
| **Endpoint(s)** | `GET /api/v1/dashboard/ventas?hoy=` |
| **Permiso** | `Ventas:Leer` |
| **Reglas** | [RN-DAS-012, RN-DAS-013](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Tendencia de siete días
  Entonces veo 7 barras, de la más antigua a hoy, cada una con su monto bruto y su número de ventas
  Y las fechas se rotulan "dd/MM" y la última dice "Hoy" (incluye el día en curso, aún incompleto)
  Y cada barra es proporcional al día de mayor monto

Escenario: Productos más vendidos
  Entonces veo hasta 5 productos con más unidades base vendidas en los últimos 7 días, con "{unidades} unidades · S/ {monto}"
  Y las unidades son base, no presentaciones (2 cajas de 10 cuentan 20)
  Y los empates se resuelven por el identificador del producto

Escenario: Sin ventas en la semana
  Entonces veo "Aún no hay ventas en los últimos 7 días."

Escenario: Líneas cuyo lote o producto ya no existe
  Entonces esas líneas se omiten del ranking
```

### HU-DAS-004 — Ver mi caja y las cajas abiertas

**Como** cajero **quiero** ver desde el Inicio si tengo caja abierta y cuánto efectivo debería haber **para** controlar mi cajón durante el turno.

| | |
|---|---|
| **Módulo** | Dashboard / Caja |
| **Pantalla** | `/dashboard` — bloque «Caja» |
| **Endpoint(s)** | `GET /api/v1/dashboard/caja` |
| **Permiso** | `POS:Leer` (y `POS:AperturarCaja` para «Abrir caja») |
| **Reglas** | [RN-DAS-015, RN-DAS-016](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CAJ-011, RN-CAJ-012, RN-CAJ-013, RN-CAJ-014](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Tengo una caja abierta
  Dado que tengo una caja abierta con apertura S/ 100.00 y ventas en efectivo netas de S/ 50.00
  Entonces veo "Tienes una caja abierta" con "Sucursal", "Abierta desde", "Monto de apertura" S/ 100.00 y "Efectivo esperado" S/ 150.00
  Y el botón "Ver mi caja" que abre mi sesión

Escenario: No tengo caja abierta
  Entonces veo "No tienes una caja abierta"
  Y, si tengo "POS:AperturarCaja", el botón "Abrir caja" que va a "/sesiones-caja/nueva"

Escenario: El esperado es solo de mi caja
  Entonces el efectivo esperado se calcula con la misma fórmula del cierre y solo para mi propia sesión, sin depender de las sucursales de mi alcance
  Y si mi sesión de usuario no trae empleado, la cifra es nula y no se muestra

Escenario: Cajas abiertas de la sucursal
  Dado que hay sesiones abiertas de otros cajeros en mi alcance
  Entonces veo "Cajas abiertas ({n})" con "Cajero", "Sucursal" y "Abierta desde"
  Y la lista incluye hasta 50 sesiones, de la más reciente a la más antigua
  Y el botón "Ver cajas abiertas" abre "/sesiones-caja" filtrada por estado "Abierta"

Escenario: Solo mi caja
  Dado que no hay otras sesiones abiertas distintas de la mía
  Entonces la tabla "Cajas abiertas" no aparece
```

> **Brecha conocida.** La API entrega el monto de apertura de las cajas ajenas a cualquiera con `POS:Leer`, aunque la pantalla solo muestre cajero, sucursal y hora (01c3 hallazgo 9).

### HU-DAS-005 — Ver las alertas de inventario

**Como** responsable de inventario **quiero** ver lotes vencidos, por vencer y productos sin stock o con stock bajo **para** actuar antes de que se agoten o caduquen.

| | |
|---|---|
| **Módulo** | Dashboard / Inventario |
| **Pantalla** | `/dashboard` — bloque «Inventario» |
| **Endpoint(s)** | `GET /api/v1/dashboard/inventario?hoy=` |
| **Permiso** | `Inventarios:Leer` |
| **Reglas** | [RN-DAS-017, RN-DAS-018, RN-DAS-019, RN-DAS-020, RN-DAS-021](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-LOT-006, RN-STK-017](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Lotes vencidos con stock
  Entonces "Lotes vencidos con stock" cuenta las filas de stock con cantidad mayor que 0 y fecha de caducidad anterior al inicio de hoy
  Y un lote que caduca hoy no está vencido
  Y cuentan también los lotes retenidos en cuarentena

Esquema del escenario: Tramos de vencimiento (hoy es 08/10)
  Dado una fila de stock con cantidad mayor que 0 que caduca el <fecha>
  Entonces cuenta en <tramo>

  Ejemplos:
    | fecha                | tramo                                         |
    | 08/10                | "Lotes que vencen en 30 días o menos"          |
    | 07/11 (hoy + 30 d)   | "Lotes que vencen en 30 días o menos"          |
    | 08/11                | "Lotes que vencen en 31 a 60 días"             |
    | 08/01 siguiente      | ninguno (pasa de hoy + 90 días)                |

Escenario: Las cifras cuentan filas de stock, no lotes únicos
  Dado que un mismo lote está en dos sucursales del alcance
  Entonces cuenta dos veces

Escenario: Productos sin stock
  Entonces "Productos sin stock" cuenta los pares (sucursal, producto) que tienen alguna fila de stock pero ninguna vendible
  Y "vendible" es cantidad mayor que 0, lote no vencido y sin retención en cuarentena
  Y veo "Sin unidades vendibles, por sucursal"

Escenario: Productos con stock bajo
  Dado que un producto tiene en la sucursal un lote vendible de 6, otro vencido de 20 y otro en cuarentena de 15
  Entonces su stock vendible es 6
  Y cuenta en "Productos con stock bajo" ("10 unidades o menos, por sucursal") y no en "Productos sin stock"
  Y el umbral 10 es fijo en el código

Escenario: Botón "Ver lotes"
  Entonces las tarjetas de vencimiento ofrecen "Ver lotes", que abre el stock por sucursal con la caducidad ya filtrada y "solo con stock"
```

### HU-DAS-006 — Ver los lotes en cuarentena

**Como** regente **quiero** ver cuántos lotes están retenidos **para** saber qué mercadería no se puede vender.

| | |
|---|---|
| **Módulo** | Dashboard / DIGEMID |
| **Pantalla** | `/dashboard` — tarjeta «Lotes en cuarentena» del bloque «Inventario» |
| **Endpoint(s)** | `GET /api/v1/dashboard/cuarentena` |
| **Permiso** | `DIGEMID:Leer` |
| **Reglas** | [RN-DAS-022](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CUA-007](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Contar lotes retenidos
  Dado que hay registros de cuarentena en estado "Retenido" en mi alcance
  Entonces veo "Lotes en cuarentena" con el número de pares distintos (sucursal, lote) y "Retenidos: no se pueden vender"

Escenario: Un lote aislado varias veces cuenta una vez por sucursal
  Dado que un lote tiene dos registros "Retenido" en la misma sucursal
  Entonces cuenta una vez

Escenario: Una devolución a cuarentena no aparece sola
  Dado que devolví mercadería con destino "Cuarentena DIGEMID"
  Entonces el lote no cuenta aquí hasta que alguien registre el aislamiento por la API (HU-OPS-006)

Escenario: Solo con permiso
  Dado que no tengo "DIGEMID:Leer"
  Entonces la tarjeta no se pide ni se dibuja
```

### HU-DAS-007 — Ver las devoluciones de hoy

**Como** supervisor **quiero** ver cuántas devoluciones hubo hoy y cuánto se reembolsó **para** vigilar el volumen de devoluciones.

| | |
|---|---|
| **Módulo** | Dashboard / Devoluciones |
| **Pantalla** | `/dashboard` — bloque «Devoluciones de hoy» |
| **Endpoint(s)** | `GET /api/v1/dashboard/devoluciones?hoy=` |
| **Permiso** | `Devoluciones:Leer` |
| **Reglas** | [RN-DAS-023](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Devoluciones del día
  Dado que hoy hubo dos devoluciones con reembolsos de S/ 15.00 (efectivo) y S/ 35.00 (Yape / Plin)
  Entonces veo "Devoluciones" 2 y "Monto reembolsado: S/ 50.00"
  Y cuenta devoluciones parciales y totales por igual, de cualquier método de reembolso

Escenario: Solo las de mis sucursales
  Entonces cuenta las devoluciones cuya venta de origen es de una sucursal de mi alcance

Escenario: El bloque depende del permiso, no de una pantalla
  Entonces el bloque se muestra con "Devoluciones:Leer" aunque el módulo no tenga pantalla de listado
```

### HU-DAS-008 — Ver solo lo que mi rol y mis sucursales permiten

**Como** usuario **quiero** que el Inicio me muestre solo los indicadores de mi competencia **para** no ver datos que no me corresponden.

| | |
|---|---|
| **Módulo** | Dashboard |
| **Pantalla** | `/dashboard` — «Inicio» |
| **Endpoint(s)** | Los cinco `GET /api/v1/dashboard/*`; `GET /api/v1/auth/me` |
| **Permiso** | Ruta sin permiso; cada bloque exige el `Leer` de su módulo |
| **Reglas** | [RN-DAS-001, RN-DAS-003](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-GEN-021, RN-GEN-023](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Saludo
  Dado que mi nombre es "Carlos Cajero Demo"
  Entonces veo "Hola, Carlos Cajero Demo" y "Este es el resumen de hoy."

Escenario: Un bloque por permiso
  Dado que tengo "Ventas:Leer", "POS:Leer", "Inventarios:Leer", "DIGEMID:Leer" y "Devoluciones:Leer"
  Entonces veo los bloques de ventas, caja, inventario (con la tarjeta de cuarentena) y devoluciones
  Y si me falta uno, ese bloque no se pide ni se dibuja

Escenario: Sin ningún permiso de lectura
  Dado que no tengo ninguno de esos permisos
  Entonces veo "Tu usuario no tiene indicadores disponibles aquí. Usa el menú lateral para ir a tus módulos."

Escenario: El administrador ve todo
  Dado que soy administrador
  Entonces veo todos los bloques con el total de la empresa y, si hay más de una sucursal, la tarjeta "Ventas por sucursal"

Escenario: Un cajero con una sola sucursal
  Entonces ve los totales de su sucursal, "Mis ventas de hoy" y su caja

Escenario: Alcance de un usuario no administrador
  Entonces las cifras cubren su sucursal base más las asignadas a su cuenta y a sus roles

Escenario: La ruta no exige permiso
  Entonces cualquier usuario autenticado entra a "/dashboard"
  Y es el destino de los rechazos de permiso de otras pantallas

Escenario: Fallan mis permisos
  Cuando no se pueden cargar mis permisos
  Entonces veo "No se pudieron cargar tus permisos." con "Reintentar"
```

### HU-DAS-009 — Actualizar el resumen y reintentar una sección

**Como** usuario **quiero** refrescar el Inicio y reintentar lo que falló **para** tener cifras al día.

| | |
|---|---|
| **Módulo** | Dashboard |
| **Pantalla** | `/dashboard` — botón «Actualizar» y «Reintentar» |
| **Endpoint(s)** | Los cinco `GET /api/v1/dashboard/*` |
| **Permiso** | El `Leer` de cada módulo |
| **Reglas** | [RN-DAS-002, RN-DAS-024](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Actualizar
  Cuando hago clic en "Actualizar"
  Entonces se recalcula "hoy" y se piden de nuevo todas las secciones permitidas
  Y veo "Actualizado a las 10:15"
  Y el botón queda deshabilitado mientras carga, y se conservan los datos previos hasta que lleguen los nuevos

Escenario: No hay recarga automática
  Entonces el Inicio no se actualiza solo: sin pulsar "Actualizar" las cifras quedan como estaban

Escenario: Una sección falla
  Dado que la consulta de caja falla
  Entonces esa tarjeta muestra "No se pudo cargar esta sección." con "Reintentar"
  Y las demás secciones siguen visibles
  Cuando hago clic en "Reintentar"
  Entonces solo se vuelve a pedir esa sección

Escenario: Cargando
  Entonces cada sección sin datos muestra un indicador de carga ("Cargando")

Escenario: Cifras en cero
  Entonces se muestran como 0 o "S/ 0.00", sin texto de "sin datos"
```

### HU-DAS-010 — Abrir los listados desde las tarjetas

**Como** cajero **quiero** saltar de una tarjeta al listado ya filtrado **para** ver el detalle sin armar los filtros a mano.

| | |
|---|---|
| **Módulo** | Dashboard |
| **Pantalla** | `/dashboard` → `/pos`, `/sesiones-caja`, `/stock-por-sucursal` |
| **Endpoint(s)** | Ninguno propio (navegación con filtros guardados) |
| **Permiso** | El de la pantalla de destino |
| **Reglas** | [RN-DAS-014, RN-DAS-016](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Enlaces de ventas con una sola sucursal
  Dado que mi alcance es una sola sucursal
  Entonces veo "Ver ventas de hoy", "Ver mis ventas" y "Ver ventas en espera"
  Y "Ver ventas de hoy" y "Ver mis ventas" abren "/pos" con la fecha de hoy y el estado "Completada" (y mi empleado en el segundo)
  Y "Ver ventas en espera" abre "/pos" con el estado "Pendiente" y sin fechas

Escenario: Con varias sucursales no hay enlaces de ventas
  Dado que mi alcance tiene más de una sucursal
  Entonces no veo esos tres botones, porque el listado de ventas solo cubre la sucursal propia

Escenario: El listado puede mostrar menos ventas que la tarjeta
  Dado que hoy hubo una venta "Devuelta"
  Entonces "Vendido hoy" la incluye y "Ver ventas de hoy" no, porque filtra "Completada"

Escenario: Otros destinos
  Entonces "Ver mi caja" abre mi sesión, "Ver cajas abiertas" abre "/sesiones-caja" con estado "Abierta" y "Ver lotes" abre el stock por sucursal con la caducidad y "solo con stock"
```

## 12. Módulos que solo existen en la API (sin pantalla)

El SPA no tiene ruta, menú ni botón para ninguno de estos módulos (verificado en el catálogo [01d](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md), sección 1). Los permisos de estos módulos sí aparecen en la matriz de la pantalla «Roles & Permisos» y se pueden conceder, pero no abren ninguna pantalla; ningún script los siembra. En estas historias el actor es un **integrador** que llama a la API con un cliente HTTP autenticado y los escenarios usan los códigos de error y el estado HTTP que devuelve la API, porque no existe «mensaje que ve el usuario». Ninguna de las operaciones de estos módulos deja evento de auditoría de seguridad (RN-AUD-006). Los enumerados que viajan en un cuerpo JSON se esperan como número (**No verificado** en ejecución, 01d §2 punto 6).

### 12.1 Transferencias entre sucursales

### HU-OPS-001 — Crear una transferencia entre sucursales

**Como** integrador (personal de la sucursal de origen) **quiero** registrar el envío de lotes a otra sucursal **para** iniciar el traslado de stock.

| | |
|---|---|
| **Módulo** | Transferencias |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `POST /api/v1/Transferencias` |
| **Permiso** | `Transferencias:Crear` |
| **Reglas** | [RN-TRA-001, RN-TRA-002, RN-TRA-003, RN-TRA-004, RN-TRA-005, RN-TRA-006, RN-TRA-007, RN-TRA-008, RN-TRA-009, RN-TRA-010, RN-TRA-011](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Crear una transferencia
  Dado que mi sesión es de la sucursal "Sede Principal" y el lote "L-001" tiene 500 unidades en esa sucursal
  Cuando llamo a "POST /api/v1/Transferencias" con la sucursal destino "Sucursal Demo Norte" y una línea del lote "L-001" con cantidad enviada 100
  Entonces recibo 201 con el identificador y la cabecera "Location" hacia "GET transferencias/{id}"
  Y la transferencia queda en estado "Iniciada" con el empleado emisor de mi sesión
  Y el stock no se toca: crear no descuenta ni reserva nada

Escenario: El origen y el emisor salen de mi sesión
  Entonces la sucursal de origen es la de mi token y el emisor es mi empleado, aunque sea administrador
  Y si mi sesión no trae sucursal o empleado recibo 400 "Auth.Sucursal" o "Auth.Empleado"

Escenario: Destino obligatorio y distinto del origen
  Dado que envío la sucursal destino vacía
  Entonces recibo 400 "CreateTransferenciaCommand.SucursalDestinoId"
  Y si el destino es mi misma sucursal recibo 400 "Transferencia.SucursalDestino"

Escenario: El destino no existe
  Dado que el destino es de otra empresa o no existe
  Entonces recibo 404 "Transferencia.SucursalDestinoNotFound"
  Y el sistema no revisa si tengo acceso a la sucursal destino

Escenario: Al menos una línea con lote y cantidad
  Dado que envío cero líneas, un lote vacío o una cantidad enviada 0 o negativa
  Entonces recibo 400 "CreateTransferenciaCommand.Detalles", "…Detalles[i].LoteId" o "…Detalles[i].CantidadEnviada"
  Y no hay máximo de líneas ni de cantidad

Escenario: El lote no tiene stock registrado en mi sucursal
  Dado que el lote existe pero solo tiene stock en otra sucursal
  Entonces recibo 404 "Transferencia.LoteNotFound"

Escenario: Stock insuficiente al crear
  Dado que el lote tiene 500 unidades
  Cuando pido 501
  Entonces recibo 400 "Transferencia.StockInsuficiente" con la cantidad disponible en el mensaje
  Y pedir exactamente 500 es válido

Escenario: Lotes vencidos o retenidos se pueden transferir
  Dado que el lote está vencido o retenido en cuarentena
  Entonces la transferencia se crea igual, porque solo se comprueba la existencia de la fila de stock y la cantidad física

Escenario: El mismo lote en varias líneas
  Dado que repito el lote en dos líneas
  Entonces cada línea se compara por separado con el stock al crear
  Y el despacho es el que acumula (ver HU-OPS-002)

Escenario: Dos transferencias pueden pedir el mismo stock
  Entonces crear no reserva nada y la que se despache segunda puede fallar

Escenario: El estado "Aprobada" no existe en la práctica
  Entonces ningún endpoint pasa una transferencia a "Aprobada"; sale de "Iniciada" directamente a "En_Transito" o "Cancelada"

Escenario: Sin permiso
  Dado que no tengo "Transferencias:Crear"
  Entonces el sistema me niega el acceso (RN-GEN-020)
```

> **Brecha conocida.** No existe pantalla: los textos de `transferencias/es.json` («Gestión de Transferencias de Inventario» y otros) no los carga ningún componente. Los errores `Forbidden` de transferencias responden 400 y no 403 (01d §2 punto 1).

### HU-OPS-002 — Despachar una transferencia

**Como** integrador (personal del origen) **quiero** despachar la transferencia **para** que el stock salga de mi sucursal y quede en tránsito.

| | |
|---|---|
| **Módulo** | Transferencias |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `PUT /api/v1/Transferencias/{id}/despachar` |
| **Permiso** | `Transferencias:Actualizar` |
| **Reglas** | [RN-TRA-012, RN-TRA-013, RN-TRA-014](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Despachar
  Dado que la transferencia está "Iniciada" y mi sucursal es la de origen
  Cuando llamo a "PUT /api/v1/Transferencias/{id}/despachar"
  Entonces recibo 204 sin cuerpo
  Y la transferencia pasa a "En_Transito" con la fecha de despacho en la hora actual UTC
  Y se resta la cantidad enviada de cada fila de stock del origen (100 unidades de "L-001")

Escenario: Solo el origen despacha
  Dado que mi sesión es de otra sucursal, incluso si soy administrador
  Entonces recibo 400 "Transferencia.Forbidden" (declarado como prohibido pero sin número, por eso 400 y no 403)

Escenario: La transferencia no existe
  Entonces recibo 404 "Transferencia.NotFound"

Escenario: Estado que no admite el despacho
  Dado que la transferencia ya está "En_Transito", "Completada", "Recibida_Parcial" o "Cancelada"
  Entonces recibo 400 "Transferencia.EstadoLogistico"
  Y esta comprobación va antes que la de stock

Escenario: Stock insuficiente al despachar
  Dado que otra operación bajó el stock del lote a 50 y la transferencia pide 100
  Entonces recibo 400 "Transferencia.StockInsuficiente" con la cantidad disponible (0 si ya no hay fila)
  Y no se guarda nada, ni el cambio de estado

Escenario: Líneas con el mismo lote se acumulan
  Dado que la transferencia tiene dos líneas del mismo lote de 300 y 300 y el stock es 500
  Entonces el despacho se rechaza con "Transferencia.StockInsuficiente" porque la suma excede el stock

Escenario: Conflicto de concurrencia
  Dado que otra operación modificó la misma fila de stock
  Entonces recibo 409 "Concurrency.Conflict"
```

### HU-OPS-003 — Recibir una transferencia

**Como** integrador (personal del destino) **quiero** registrar lo que llegó **para** que entre al stock de mi sucursal.

| | |
|---|---|
| **Módulo** | Transferencias |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `PUT /api/v1/Transferencias/{id}/recibir` (cuerpo: arreglo `[{loteId, cantidadRecibida}]`) |
| **Permiso** | `Transferencias:Actualizar` |
| **Reglas** | [RN-TRA-015, RN-TRA-016, RN-TRA-017, RN-TRA-018, RN-TRA-019, RN-TRA-020, RN-TRA-024](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Recepción completa
  Dado que la transferencia está "En_Transito" con 100 unidades de "L-001" y mi sucursal es la de destino
  Cuando llamo a "PUT /api/v1/Transferencias/{id}/recibir" con [{loteId: L-001, cantidadRecibida: 100}]
  Entonces recibo 204
  Y la transferencia queda "Completada" con mi empleado como receptor y la hora de recepción
  Y el destino suma 100 unidades (si no tenía fila de stock de ese lote, se crea)

Escenario: Recepción parcial
  Cuando recibo 90 de las 100 unidades
  Entonces la transferencia queda "Recibida_Parcial"
  Y el destino suma solo 90

Escenario: La diferencia se pierde
  Dado que se enviaron 100 y se recibieron 90
  Entonces las 10 unidades ya salieron del origen en el despacho y no vuelven ni se registran como merma
  Y el request no admite sustento ni motivo de la diferencia

Escenario: Cantidad recibida 0
  Cuando recibo 0 de un lote
  Entonces es válido ("no llegó nada") y la línea queda en 0 sin tocar el stock

Escenario: Las líneas no listadas se ignoran
  Dado que la transferencia tiene dos líneas y solo listo una completa
  Entonces la transferencia queda "Completada" aunque la otra línea no se recibió, y la línea omitida conserva la cantidad recibida en NULL sin sumar stock

Escenario: Reglas por línea
  Dado que listo un lote que no pertenece a la transferencia, una cantidad negativa o mayor que lo enviado
  Entonces recibo 400 "Transferencia.Recepcion"
  Y no se guarda nada de la recepción
  Y recibir exactamente lo enviado es válido

Escenario: Forma del cuerpo
  Dado que envío un arreglo vacío, un lote vacío o una cantidad negativa
  Entonces recibo 400 "RecibirTransferenciaCommand.Recepciones" o "…Recepciones[i].CantidadRecibida"

Escenario: Solo el destino recibe
  Dado que mi sucursal no es la de destino (aunque sea administrador)
  Entonces recibo 400 "Transferencia.Forbidden"
  Y si falta el empleado en mi sesión recibo 400 "Auth.Empleado"

Escenario: Solo se recibe una transferencia en tránsito
  Dado que la transferencia ya está "Completada" o "Recibida_Parcial"
  Entonces recibo 400 "Transferencia.EstadoLogistico": no se puede recibir dos veces

Escenario: Recibir y cancelar a la vez
  Dado que el destino recibe mientras el origen cancela
  Entonces solo una operación se guarda y la otra recibe 409 "Concurrency.Conflict"
  # No verificado en ejecución
```

> **Brecha conocida.** Los textos de `transferencias/es.json` hablan de un «sustento obligatorio» para la recepción parcial que el backend no implementa (RN-TRA-020). La diferencia enviada y no recibida desaparece del stock total entre sucursales (01d hallazgo 7).

### HU-OPS-004 — Cancelar una transferencia

**Como** integrador (personal del origen) **quiero** cancelar una transferencia **para** devolver el stock a mi sucursal si todavía no se recibió.

| | |
|---|---|
| **Módulo** | Transferencias |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `PUT /api/v1/Transferencias/{id}/cancelar` |
| **Permiso** | `Transferencias:Actualizar` |
| **Reglas** | [RN-TRA-021, RN-TRA-022, RN-TRA-023, RN-TRA-024](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Cancelar una transferencia en tránsito
  Dado que la transferencia está "En_Transito" con 100 unidades de "L-001" y mi sucursal es la de origen
  Cuando llamo a "PUT /api/v1/Transferencias/{id}/cancelar"
  Entonces recibo 204 y la transferencia queda "Cancelada"
  Y el origen recupera las 100 unidades (se suman a la fila existente o se crea si ya no existe)

Escenario: Cancelar antes del despacho
  Dado que la transferencia está "Iniciada"
  Entonces queda "Cancelada" sin mover stock, porque nunca se descontó

Escenario: Solo el origen cancela
  Dado que soy del destino
  Entonces recibo 400 "Transferencia.Forbidden": el destino no puede rechazar una transferencia

Escenario: Estados que no se cancelan
  Dado que la transferencia está "Completada", "Recibida_Parcial" o "Cancelada"
  Entonces recibo 400 "Transferencia.EstadoLogistico" y el mensaje indica el estado actual

Escenario: La transferencia no existe
  Entonces recibo 404 "Transferencia.NotFound"
```

### HU-OPS-005 — Consultar transferencias

**Como** integrador **quiero** listar y consultar transferencias de mi sucursal **para** seguir su estado.

| | |
|---|---|
| **Módulo** | Transferencias |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `GET /api/v1/Transferencias`, `GET /api/v1/Transferencias/{id}` |
| **Permiso** | `Transferencias:Leer` |
| **Reglas** | [RN-TRA-025, RN-TRA-026, RN-TRA-027, RN-TRA-028, RN-TRA-029, RN-TRA-030](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Listar mis transferencias
  Cuando llamo a "GET /api/v1/Transferencias"
  Entonces recibo solo las transferencias donde la sucursal de mi sesión es origen o destino, también si soy administrador
  Y puedo filtrar por "SucursalFiltroId" y por "Estado" (nombre o número)
  Y el orden es por fecha de despacho descendente, de a 10 por página
  Y cada fila trae sucursales y empleados con sus nombres, el estado como texto ("En_Transito"), las fechas y las líneas con número de lote, cantidad enviada y recibida

Escenario: Estado inexistente
  Dado que envío un "Estado" que no existe
  Entonces falla la lectura del parámetro (RN-GEN-041), no hay validador de la consulta

Escenario: Consultar una transferencia
  Cuando llamo a "GET /api/v1/Transferencias/{id}" de una transferencia donde mi sucursal es origen o destino
  Entonces recibo 200 con el mismo contenido y el número de lote "Desconocido" si el lote ya no se encuentra

Escenario: Identificador inexistente o ajeno
  Entonces recibo 404 "Transferencia.NotFound" si no existe y 400 "Transferencia.Forbidden" si existe pero no es de mi sucursal
  # una sucursal puede así saber que un identificador existe aunque no sea suyo

Escenario: Estructura y dependencias
  Entonces los estados, las cantidades y las referencias de las tablas "Transferencias_Cab" y "Transferencias_Det" las impone la base con restricciones
  Y desde el commit 27597cd una sucursal con transferencias no se puede eliminar ("Sucursal.InUse", 409)
```

> **Brecha conocida.** Las transferencias ignoran las sucursales accesibles: solo cuenta la sucursal del token, también para el administrador (01d hallazgo 8). No hay exportación.

### 12.2 Cuarentena y actas de destrucción DIGEMID

### HU-OPS-006 — Aislar un lote en cuarentena

**Como** integrador (regente) **quiero** registrar el aislamiento de un lote **para** impedir su venta.

| | |
|---|---|
| **Módulo** | Cuarentena DIGEMID |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `POST /api/v1/DIGEMID/cuarentena` |
| **Permiso** | `DIGEMID:AislarLoteCuarentena` |
| **Reglas** | [RN-CUA-001, RN-CUA-002, RN-CUA-003, RN-CUA-004, RN-CUA-005, RN-CUA-006, RN-CUA-010, RN-CUA-011, RN-CUA-020](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-DEV-024](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Aislar un lote como "Retenido"
  Dado que el lote "L-001" existe en la empresa
  Cuando llamo a "POST /api/v1/DIGEMID/cuarentena" con lote "L-001", cantidad aislada 20, motivo "Alerta de calidad" y estado de resolución "Retenido"
  Entonces recibo 200 con el identificador del registro
  Y el registro guarda la sucursal y el empleado de mi sesión y la hora actual UTC como fecha de ingreso
  Y el lote queda bloqueado para la venta de inmediato en toda la empresa

Escenario: Datos obligatorios
  Dado que envío el lote vacío, la cantidad 0, el motivo vacío o de más de 50 caracteres, o un estado fuera de "Retenido", "Liberado", "Destruido" o "Devuelto"
  Entonces recibo 400 "AislarLoteCuarentenaCommand.LoteId", "…CantidadAislada", "…MotivoAislamiento" o "…EstadoResolucion"

Escenario: El estado lo elijo yo al crear
  Dado que envío estado "Liberado", "Destruido" o "Devuelto"
  Entonces el registro se guarda con ese estado y no bloquea la venta

Escenario: Lote o línea de devolución inexistentes
  Entonces recibo 404 "LoteInventario.NotFound" o 404 "DetalleDevolucion.NotFound"
  Y un identificador de otra empresa se informa como inexistente

Escenario: No se comprueba el stock
  Entonces el sistema no verifica que el lote tenga fila de stock en mi sucursal, que la cantidad aislada no supere el stock ni que la línea de devolución sea de ese lote
  Y aislar no mueve stock: el stock físico sigue contando

Escenario: Un lote puede tener varios registros
  Entonces puedo aislar el mismo lote varias veces, en distintas sucursales o con distintos estados

Escenario: La sucursal y el empleado salen de mi sesión
  Dado que mi sesión no trae sucursal o empleado
  Entonces recibo 400 "Auth.Sucursal" o "Auth.Empleado"

Escenario: Aislar desde una devolución con destino «Cuarentena DIGEMID»
  Dado que una devolución se procesó con destino "Cuarentena DIGEMID"
  Cuando aíslo el lote informando el identificador de la línea de devolución
  Entonces queda el vínculo, sin que el sistema compare cantidades ni lote
  Y la devolución por sí sola nunca crea el registro de cuarentena

Escenario: Estructura y auditoría
  Entonces las referencias a sucursal, lote, empleado y línea de devolución las impone la base
  Y esta operación no deja evento de auditoría de seguridad: queda solo el usuario y la fecha de creación
```

> **Brecha conocida.** Un hecho regulado por DIGEMID queda sin rastro en la auditoría (RN-CUA-005). La base no tiene restricciones sobre el estado ni la cantidad (RN-CUA-020).

### HU-OPS-007 — Bloquear la venta de un lote retenido (y sus límites)

**Como** regente **quiero** que un lote retenido no se pueda vender **para** proteger al paciente, sabiendo qué flujos sí lo ignoran.

| | |
|---|---|
| **Módulo** | Cuarentena DIGEMID / Ventas |
| **Pantalla** | Solo API para el registro; el efecto se ve en el terminal de venta y en el Inicio |
| **Endpoint(s)** | `POST /api/v1/DIGEMID/cuarentena`; `POST /api/v1/ventas` |
| **Permiso** | `DIGEMID:AislarLoteCuarentena` / `Ventas:Crear` |
| **Reglas** | [RN-CUA-007, RN-CUA-008, RN-CUA-009](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-VEN-012](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| **Estado** | Parcial (solo la venta respeta la cuarentena y no existe forma de liberar un lote retenido) |

**Criterios de aceptación**

```gherkin
Escenario: El lote retenido no se vende en ninguna sucursal
  Dado que el lote "L-001" tiene un registro de cuarentena "Retenido" no eliminado
  Entonces el terminal de venta no lo ofrece, no cuenta como stock vendible y cuenta en "Lotes en cuarentena" del Inicio
  Y si una petición lo incluye, el servidor responde 400 "Venta.Cuarentena" y el cajero ve "Ocurrió un error al procesar la venta."
  Y esto no depende de la sucursal ni de la cantidad aislada: se bloquea el lote entero

Escenario: Los estados "Liberado", "Destruido" y "Devuelto" no bloquean
  Dado que el lote solo tiene registros en esos tres estados
  Entonces se puede vender y cuenta como stock vendible

Escenario: Crear un registro "Liberado" no levanta un bloqueo anterior
  Dado que el lote tiene un registro "Retenido"
  Cuando creo otro registro "Liberado" para el mismo lote
  Entonces el lote sigue bloqueado mientras exista cualquier registro "Retenido"

Escenario: No existe forma de liberar un lote retenido
  Entonces ningún endpoint cambia el estado de un registro ni lo elimina
  Y la única vía es modificar la base de datos a mano

Escenario: Los demás flujos no consultan la cuarentena
  Entonces se pueden transferir lotes "Retenido" (HU-OPS-001), consumirlos como insumo de una orden magistral (HU-OPS-011) o registrarlos en una inmunización (HU-OPS-015)
```

> **Brecha conocida.** Es el hallazgo 1 y 5 de 01d §9.3: la cuarentena solo la respeta la venta y un lote retenido no se puede liberar por la API.

### HU-OPS-008 — Consultar la cuarentena

**Como** regente **quiero** listar los registros de cuarentena **para** saber qué se aisló, dónde y por qué.

| | |
|---|---|
| **Módulo** | Cuarentena DIGEMID |
| **Pantalla** | Solo API (sin pantalla); el Inicio muestra solo el conteo (HU-DAS-006) |
| **Endpoint(s)** | `GET /api/v1/DIGEMID/cuarentena` |
| **Permiso** | `DIGEMID:Leer` |
| **Reglas** | [RN-CUA-012](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-GEN-008, RN-GEN-009, RN-GEN-023](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Listar registros
  Cuando llamo a "GET /api/v1/DIGEMID/cuarentena"
  Entonces recibo los registros de las sucursales accesibles (el administrador, todas), por fecha de ingreso descendente, de a 10
  Y cada fila trae identificador, sucursal, lote, cantidad, motivo, estado y fecha, pero no la línea de devolución ni el empleado que registró

Escenario: Filtros
  Entonces puedo filtrar por sucursal, estado (igualdad exacta, sin validar que sea uno de los cuatro) y fechas de ingreso desde y hasta (inclusivas, en UTC)
  Y una sucursal no accesible devuelve una página vacía

Escenario: Sucursal con identificador vacío
  Dado que envío la sucursal igual al GUID vacío
  Entonces recibo 400 "GetCuarentenaQuery.SucursalId"
```

### HU-OPS-009 — Generar un acta de destrucción

**Como** integrador (regente) **quiero** registrar un acta de destrucción **para** dejar constancia de la destrucción de mercadería.

| | |
|---|---|
| **Módulo** | Actas DIGEMID |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `POST /api/v1/DIGEMID/acta-destruccion` |
| **Permiso** | `DIGEMID:GenerarActaDestruccion` |
| **Reglas** | [RN-CUA-013, RN-CUA-014, RN-CUA-015, RN-CUA-016, RN-CUA-017, RN-CUA-018, RN-CUA-021, RN-CUA-022, RN-CUA-023](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Parcial (el acta es solo una cabecera: no lista lotes ni cantidades, no cambia la cuarentena ni el stock) |

**Criterios de aceptación**

```gherkin
Escenario: Generar un acta
  Cuando llamo a "POST /api/v1/DIGEMID/acta-destruccion" con número de resolución interna "RES-2026-001", empresa de residuos, fecha de ejecución y regente responsable
  Entonces recibo 200 con el identificador del acta
  Y el acta guarda solo la cabecera

Escenario: Datos del acta
  Entonces el número de resolución es obligatorio y de hasta 50 caracteres, la empresa de residuos obligatoria y de hasta 150, el manifiesto de transporte opcional y de hasta 50, y la ruta del acta firmada opcional y de hasta 500
  Y la fecha de ejecución no tiene ninguna comprobación (puede ser futura)
  Y la ruta del PDF es un texto: el sistema no recibe ni guarda el archivo
  Y un dato inválido responde 400 con el código de la entidad, por ejemplo "DigemidActaDestruccion.NumeroResolucionInterna" (el validador está vacío)

Escenario: El regente debe existir
  Dado que el regente no existe en la empresa
  Entonces recibo 404 "Empleado.NotFound"
  Y cualquier empleado vale: no se comprueba que sea regente ni que tenga licencia

Escenario: Número de resolución repetido
  Dado que ya existe un acta no eliminada de mi empresa con "RES-2026-001"
  Entonces recibo 409 "Db.Duplicado"
  Y un acta eliminada libera su número

Escenario: El acta no incluye lotes ni cambia nada más
  Entonces no se llenan los detalles del acta (registro de cuarentena y cantidad destruida)
  Y ningún registro de cuarentena pasa a "Destruido" ni se toca el stock

Escenario: El acta no se edita ni se elimina
  Entonces no hay endpoints para modificar o eliminar un acta

Escenario: Estructura
  Entonces la clave foránea al regente y el índice único por empresa y número los impone la base
  Y esta operación no deja evento de auditoría de seguridad
```

> **Brecha conocida.** «El acta de destrucción es solo una cabecera» (01d hallazgo 2): una destrucción real no deja rastro enlazado. Hay protección tardía del número: el handler no lo comprueba antes, lo rechaza la base como `Db.Duplicado`.

### HU-OPS-010 — Consultar las actas de destrucción

**Como** integrador **quiero** listar las actas **para** presentarlas a DIGEMID.

| | |
|---|---|
| **Módulo** | Actas DIGEMID |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `GET /api/v1/DIGEMID/actas` |
| **Permiso** | `DIGEMID:Leer` |
| **Reglas** | [RN-CUA-019](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Listar actas
  Cuando llamo a "GET /api/v1/DIGEMID/actas" con fechas de ejecución desde y hasta
  Entonces recibo las actas de la empresa (no se filtra por sucursal), por fecha de ejecución descendente, de a 10
  Y cada fila trae identificador, número de resolución, fecha y empresa de residuos
  Y no trae regente, manifiesto ni ruta del PDF

Escenario: No hay consulta individual
  Entonces no existe un endpoint para consultar una sola acta ni un validador de la consulta
```

### 12.3 Órdenes magistrales

### HU-OPS-011 — Iniciar una orden magistral

**Como** integrador (químico) **quiero** iniciar una orden de preparación magistral **para** descontar los insumos del stock de mi sucursal.

| | |
|---|---|
| **Módulo** | Magistrales |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `POST /api/v1/Magistrales/iniciar` |
| **Permiso** | `Magistrales:IniciarOrdenMagistral` |
| **Reglas** | [RN-MAG-001, RN-MAG-002, RN-MAG-003, RN-MAG-004, RN-MAG-005, RN-MAG-006, RN-MAG-007, RN-MAG-008, RN-MAG-009, RN-MAG-010, RN-MAG-011, RN-MAG-020, RN-MAG-021, RN-MAG-022, RN-MAG-023](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Parcial (no hay cancelación ni devolución de insumos de una orden iniciada y nunca completada) |

**Criterios de aceptación**

```gherkin
Escenario: Iniciar una orden
  Dado que el producto resultante existe y los insumos tienen filas de stock en mi sucursal
  Cuando llamo a "POST /api/v1/Magistrales/iniciar" con producto resultante, cantidad a producir 50 y un insumo (fila de stock, cantidad consumida 10)
  Entonces recibo 200 con el identificador de la orden
  Y la orden queda "Iniciada" con mi sucursal, mi empleado como químico preparador y la hora actual UTC
  Y el stock del insumo baja 10 y se guarda un consumo con unidad de medida fija "UND"

Escenario: La sucursal y el químico salen de mi sesión
  Entonces no se eligen en el request, y no se comprueba que mi empleado sea químico
  Y si mi sesión no trae sucursal o empleado recibo 400 "Auth.Sucursal" o "Auth.Empleado"

Escenario: Datos obligatorios
  Dado que envío producto vacío, cantidad nula o 0, cero insumos, una fila de stock vacía o una cantidad consumida 0
  Entonces recibo 400 "IniciarOrdenMagistralCommand.ProductoResultanteId", "…CantidadProducida", "…Consumos", "…Consumos[i].InventarioSucursalId" o "…Consumos[i].CantidadConsumida"

Escenario: El producto o la receta no existen
  Entonces recibo 404 "Medicamento.NotFound" o 404 "RecetaMedica.NotFound"
  Y solo se comprueba la existencia: no que la receta sea del paciente, esté vigente, ni que el producto sea magistral

Escenario: El insumo es de otra sucursal
  Dado que la fila de stock es de otra sucursal o de otra empresa
  Entonces recibo 404 "Inventario.NotFound"

Escenario: Insumo con lote vencido
  Entonces recibo 400 "Magistral.Insumo.LoteVencido" (el lote se usa hasta el final de su día de caducidad)

Escenario: Stock insuficiente
  Dado que el insumo tiene 8 unidades y pido consumir 10
  Entonces recibo 400 "Inventario.StockInsuficiente"
  Y no se guarda nada de la orden

Escenario: El mismo insumo en dos líneas
  Entonces se consume dos veces y la comprobación de stock ve lo ya descontado

Escenario: La cuarentena no impide usar un lote como insumo
  Entonces un lote con registro "Retenido" puede consumirse

Escenario: Una orden iniciada no se puede deshacer
  Entonces no existe cancelación, anulación ni devolución de insumos
  Y una orden iniciada y nunca completada deja el stock descontado para siempre

Escenario: Conflicto de concurrencia sobre el stock
  Entonces recibo 409 "Concurrency.Conflict"

Escenario: Estructura
  Entonces la base impone las claves foráneas de la orden y de sus consumos, sin restricciones sobre el estado ni la cantidad
  Y desde el commit 27597cd un medicamento o una sucursal con órdenes magistrales no se puede eliminar ("Medicamento.InUse", "Sucursal.InUse", 409)
```

> **Brecha conocida.** La cantidad producida se declara al iniciar y no se corrige al completar; la unidad es siempre `UND`; no se compara con ninguna formulación (01d hallazgo 4). El estado por defecto de la base (`En_Proceso`) no coincide con el que escribe el código (`Iniciada`) (hallazgo 12).

### HU-OPS-012 — Completar una orden magistral

**Como** integrador (químico) **quiero** completar la orden **para** que nazca el lote terminado con su stock.

| | |
|---|---|
| **Módulo** | Magistrales |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `POST /api/v1/Magistrales/completar` |
| **Permiso** | `Magistrales:CompletarOrdenMagistral` |
| **Reglas** | [RN-MAG-012, RN-MAG-013, RN-MAG-014, RN-MAG-015, RN-MAG-016, RN-MAG-017](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-LOT-004, RN-LOT-006, RN-LOT-010](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Completar la orden
  Dado que la orden está "Iniciada" y es de mi sucursal
  Cuando llamo a "POST /api/v1/Magistrales/completar" con el identificador, número de lote del fabricante "MAG-0001" y una fecha de caducidad futura
  Entonces recibo 200 con el identificador de la orden
  Y se crea un lote del producto resultante con fecha de fabricación igual a la hora actual
  Y se crea su fila de stock en la sucursal de la orden con la cantidad declarada al iniciar (50)
  Y la orden queda "Completada" con el lote generado, estado final

Escenario: Datos de completar
  Dado que envío el identificador vacío, el número de lote vacío o de más de 50 caracteres, o una caducidad ya vencida
  Entonces recibo 400 "CompletarOrdenMagistralCommand.OrdenId", "…NumeroLoteMfr" o "…FechaCaducidad"
  Y el número de lote admite 50 caracteres aunque un lote normal admite 100

Escenario: La orden no existe
  Entonces recibo 404 "Orden.NotFound"

Escenario: La orden es de otra sucursal
  Entonces recibo 403 "OrdenMagistral.OtraSucursal", también si soy administrador

Escenario: Completar una orden ya completada
  Entonces recibo 400 "Orden.Estado"
  # No verificado: la orden no tiene versión de fila, dos completados simultáneos podrían pasar los dos

Escenario: Cualquier empleado con permiso de la sucursal puede completarla
  Entonces no hace falta ser quien la inició

Escenario: Número de lote repetido para el producto
  Dado que ya existe un lote de ese producto con el mismo número
  Entonces recibo 409 "Db.Duplicado" (lo rechaza la base al guardar, no una comprobación previa)

Escenario: No se informa la cantidad realmente obtenida
  Entonces el stock del lote nuevo es la cantidad declarada al iniciar
```

### HU-OPS-013 — Consultar órdenes magistrales

**Como** integrador **quiero** listar y consultar órdenes **para** seguir su estado.

| | |
|---|---|
| **Módulo** | Magistrales |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `GET /api/v1/Magistrales/ordenes`, `GET /api/v1/Magistrales/{id}` |
| **Permiso** | `Magistrales:Leer` |
| **Reglas** | [RN-MAG-018, RN-MAG-019](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Listar órdenes
  Cuando llamo a "GET /api/v1/Magistrales/ordenes"
  Entonces recibo las órdenes de las sucursales accesibles (el administrador, todas), por fecha de preparación descendente, de a 10
  Y cada fila trae identificador, sucursal, producto, cantidad, estado y fecha

Escenario: Filtros
  Entonces puedo filtrar por sucursal, estado (texto, igualdad, sin validar), y fechas de preparación desde y hasta (inclusivas, UTC)
  Y una sucursal no accesible devuelve una página vacía
  Y una sucursal igual al GUID vacío responde 400 "GetOrdenesQuery.SucursalId"

Escenario: Consultar una orden
  Cuando llamo a "GET /api/v1/Magistrales/{id}" de una orden de una sucursal accesible
  Entonces recibo 200 con el objeto de resultado completo y los mismos seis campos en "value"
  Y no incluye insumos, lote generado, receta ni químico

Escenario: La orden no existe o no es accesible
  Entonces recibo 404 "OrdenMagistral.NotFound"
```

### 12.4 Servicios e inmunizaciones

### HU-OPS-014 — Agendar un servicio

**Como** integrador **quiero** registrar una cita de servicio para un paciente **para** programar su atención.

| | |
|---|---|
| **Módulo** | Servicios |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `POST /api/v1/Servicios/agenda` |
| **Permiso** | `Servicios:AgendarServicio` |
| **Reglas** | [RN-SRV-001, RN-SRV-002, RN-SRV-003, RN-SRV-004, RN-SRV-005, RN-SRV-019, RN-SRV-021](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Parcial (el validador está vacío; con el script de base de datos actual el guardado podría fallar por columnas faltantes: no verificado en ejecución) |

**Criterios de aceptación**

```gherkin
Escenario: Agendar una cita
  Dado que el paciente y el producto existen
  Cuando llamo a "POST /api/v1/Servicios/agenda" con el paciente, el producto, la fecha y hora programada y el estado "Programada"
  Entonces recibo 200 con el identificador de la cita
  # No verificado en ejecución: ver la brecha sobre las columnas de la tabla

Escenario: Paciente, producto o venta inexistentes
  Entonces recibo 404 "Paciente.NotFound", 404 "Medicamento.NotFound" o 404 "Venta.NotFound"
  Y la venta es opcional: un identificador vacío se guarda como "sin venta"

Escenario: Sin reglas sobre estado ni fecha
  Entonces el estado es un texto libre de hasta 20 caracteres, no vacío ni de espacios (por defecto "Programada")
  Y la fecha puede ser pasada o futura, no hay horario de atención ni choque de citas
  Y el producto puede ser cualquier medicamento: no se exige que sea un servicio

Escenario: Sin validador
  Entonces no se validan campos antes del handler y los errores de campos salen con códigos de la entidad, por ejemplo "ServicioAgenda.EstadoCita"
```

> **Brecha conocida.** La tabla `Servicios_Agenda` del script `0001` no tiene las columnas `LastModifiedAt`, `LastModifiedBy`, `DeletedAt` ni `DeletedBy` que EF mapea: si la base se creó solo con scripts, agendar, listar, consultar y eliminar citas fallarían con un error 500 (RN-SRV-019, **No verificado** en ejecución). Los textos de `servicios/es.json` describen un catálogo de servicios con precio que no existe.

### HU-OPS-015 — Registrar una inmunización

**Como** integrador (vacunador) **quiero** dejar el registro clínico de una dosis **para** que quede constancia del lote usado.

| | |
|---|---|
| **Módulo** | Servicios e inmunizaciones |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `POST /api/v1/Servicios/inmunizacion` |
| **Permiso** | `Servicios:RegistrarInmunizacion` |
| **Reglas** | [RN-SRV-007, RN-SRV-008, RN-SRV-009, RN-SRV-010, RN-SRV-011, RN-SRV-012, RN-SRV-013, RN-SRV-015, RN-SRV-020](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Parcial (registrar la dosis no descuenta stock ni consulta la cuarentena) |

**Criterios de aceptación**

```gherkin
Escenario: Registrar una dosis
  Dado que el paciente, el profesional (cualquier empleado), el producto y el lote existen y el lote es del producto
  Cuando llamo a "POST /api/v1/Servicios/inmunizacion" con vía "Intramuscular", sitio anatómico "Deltoides", volumen 0.5, modalidad "Corriente" y fecha de administración
  Entonces recibo 200 con el identificador del registro

Escenario: La fecha de administración es obligatoria
  Dado que la fecha de administración es la fecha por defecto
  Entonces recibo 400 "RegistrarInmunizacionCommand.FechaAdmnFisica"

Escenario: Inexistentes
  Entonces recibo 404 "Paciente.NotFound", "Empleado.NotFound", "Medicamento.NotFound" o "Venta.NotFound" (la venta es opcional)
  Y el profesional puede ser cualquier empleado: no se exige que sea profesional de salud

Escenario: Lote de otro producto o inexistente
  Entonces recibo 404 "LoteInventario.NotFound"

Escenario: Lote vencido en la fecha de administración
  Dado que el lote vencía el 30/06/2026
  Cuando la fecha de administración es el 15/07/2026
  Entonces recibo 400 "Inmunizacion.Lote.Vencido"
  Y una inmunización histórica puede registrarse con un lote que venció después de la dosis

Escenario: Datos clínicos
  Entonces la vía es obligatoria y de hasta 50 caracteres, el sitio obligatorio y de hasta 100, y el volumen mayor que 0
  Y la fecha de entrega de la hoja informativa es opcional
  Y no hay catálogo de vías ni de sitios, ni límite de volumen, ni se rechaza una fecha futura

Escenario: Modalidad de registro
  Entonces la modalidad es "Corriente" o "Historica" y solo se guarda: ninguna regla cambia según sea corriente o histórica

Escenario: Registrar una dosis no consume stock
  Entonces el sistema no lee ni modifica el stock, no comprueba que el lote tenga existencia y no consulta la cuarentena
  Y un lote "Retenido" puede registrarse en una inmunización

Escenario: Sin sucursal ni auditoría
  Entonces el registro no tiene sucursal ni quién lo registró (solo el usuario de creación) y no deja evento de auditoría

Escenario: Volumen con 3 o 4 decimales
  # No verificado: el script define DECIMAL(8,2) y EF declara DECIMAL(12,4)

Escenario: Dependencias
  Entonces desde el commit 27597cd un paciente o un medicamento con citas o inmunizaciones no se puede eliminar ("Paciente.InUse", "Medicamento.InUse", 409)
```

> **Brecha conocida.** «Consumo de lote» significa hoy solo «dejar registrado qué lote se usó» (01d hallazgo 3).

### HU-OPS-016 — Consultar la agenda y las inmunizaciones

**Como** integrador **quiero** listar citas e inmunizaciones **para** revisar la atención.

| | |
|---|---|
| **Módulo** | Servicios e inmunizaciones |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `GET /api/v1/Servicios/agenda`, `GET /api/v1/Servicios/inmunizaciones`, `GET /api/v1/Servicios/servicio/{id}`, `GET /api/v1/Servicios/inmunizacion/{id}` |
| **Permiso** | `Servicios:Leer` |
| **Reglas** | [RN-SRV-006, RN-SRV-014, RN-SRV-016](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Parcial (los detalles por identificador solo devuelven el identificador) |

**Criterios de aceptación**

```gherkin
Escenario: Listar la agenda
  Cuando llamo a "GET /api/v1/Servicios/agenda" con estado y fechas programadas desde y hasta
  Entonces recibo las citas de toda la empresa, por fecha programada descendente, de a 10
  Y cada fila trae identificador, paciente, producto, fecha y hora y estado
  Y no se filtra por sucursal

Escenario: Listar inmunizaciones
  Cuando llamo a "GET /api/v1/Servicios/inmunizaciones" con paciente y fechas de administración
  Entonces recibo identificador, paciente, fecha y vía de administración, por fecha descendente
  Y un paciente igual al GUID vacío responde 400 "GetInmunizacionesQuery.ClienteId"

Escenario: Detalle por identificador
  Cuando llamo a "GET /api/v1/Servicios/servicio/{id}" o "…/inmunizacion/{id}"
  Entonces recibo 200 con el objeto de resultado completo y en "value" únicamente el identificador
  Y si no existe, el código literal "NotFound" con 404
```

### HU-OPS-017 — Actualizar o eliminar una cita o una inmunización

**Como** integrador **quiero** corregir o eliminar una cita o una dosis **para** mantener limpia la agenda.

| | |
|---|---|
| **Módulo** | Servicios e inmunizaciones |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `PUT /api/v1/Servicios/servicio/{id}`, `PUT /api/v1/Servicios/inmunizacion/{id}`, `DELETE /api/v1/Servicios/servicio/{id}`, `DELETE /api/v1/Servicios/inmunizacion/{id}` |
| **Permiso** | `Servicios:Actualizar` / `Servicios:Eliminar` |
| **Reglas** | [RN-SRV-017, RN-SRV-018, RN-SRV-021](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Parcial (actualizar no está implementado: responde 200 sin cambiar nada) |

**Criterios de aceptación**

```gherkin
Escenario: "Actualizar" no cambia nada
  Cuando llamo a "PUT /api/v1/Servicios/servicio/{id}" con el mismo identificador en el cuerpo
  Entonces recibo 200 con el objeto de resultado, pero no se modifica ningún dato
  Y no puedo corregir una cita, una dosis ni el estado de una cita después de creada

Escenario: Identificador de ruta distinto del del cuerpo
  Entonces recibo 400 con un texto plano "El ID de la ruta no coincide con el comando." y sin código

Escenario: Actualizar algo inexistente
  Entonces recibo 404 con el código literal "NotFound"

Escenario: Eliminar una cita o una inmunización
  Cuando llamo a "DELETE /api/v1/Servicios/inmunizacion/{id}"
  Entonces el registro queda eliminado (borrado lógico) y recibo 200 con el objeto de resultado
  Y no se pide motivo ni queda rastro de auditoría, aunque sea un registro clínico
  Y no se restringe por sucursal

Escenario: Eliminar algo inexistente
  Entonces recibo 404 con el código literal "NotFound"
```

### 12.5 Delivery

### HU-OPS-018 — Programar un delivery

**Como** integrador **quiero** registrar el despacho a domicilio de una venta **para** hacer su seguimiento.

| | |
|---|---|
| **Módulo** | Delivery |
| **Pantalla** | Solo API (sin pantalla; el menú horizontal enlaza `/delivery`, ruta inexistente) |
| **Endpoint(s)** | `POST /api/v1/Delivery` |
| **Permiso** | `Delivery:ProgramarDelivery` |
| **Reglas** | [RN-DEL-001, RN-DEL-002, RN-DEL-003, RN-DEL-004, RN-DEL-005, RN-DEL-006, RN-DEL-007, RN-DEL-008, RN-DEL-016, RN-DEL-017, RN-DEL-018](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Programar el delivery de una venta cobrada
  Dado que la venta existe, es de una sucursal accesible y no está "Anulada"
  Cuando llamo a "POST /api/v1/Delivery" con la venta, plataforma "Rappi", dirección "Av. Ficticia 123, Miraflores" y repartidor opcional
  Entonces recibo 200 (no 201) con el identificador del despacho
  Y el despacho nace en "Preparando" y no afecta la venta, el stock ni el comprobante

Escenario: Venta inexistente
  Entonces recibo 404 "Venta.NotFound" (también una venta de otra empresa o un identificador vacío)

Escenario: Sucursal de la venta no accesible
  Entonces recibo 403 "Delivery.Venta.SucursalNoPermitida"
  Y un administrador accede a todas las sucursales

Escenario: Venta anulada
  Entonces recibo 409 "Delivery.Venta.Anulada"
  Y las ventas "Pendiente", "Completada" y "Devuelta" se aceptan

Escenario: La venta ya tiene un despacho
  Entonces recibo 409 "Delivery.Venta.YaProgramado"
  Y si dos programaciones son simultáneas, la segunda recibe 409 "Db.Duplicado"
  Y eliminar el despacho libera la venta para programar otro

Escenario: Datos del despacho
  Entonces la plataforma es obligatoria y de hasta 50 caracteres, la dirección obligatoria y de hasta 255, el código de rastreo opcional y de hasta 100, el repartidor opcional y de hasta 150 y la URL de evidencia opcional y de hasta 500
  Y no se valida el formato de la URL, ni que la dirección sea la del paciente, ni un catálogo de plataformas
  Y un dato inválido responde 400 con el código de la entidad, por ejemplo "DespachoDelivery.PlataformaServicio" (el validador está vacío)

Escenario: Un despacho no tiene relación con el resto del negocio
  Entonces no tiene costo, no se refleja en el comprobante ni en el arqueo, y anular o devolver la venta después no cambia el despacho
```

### HU-OPS-019 — Cambiar el estado de un delivery

**Como** integrador **quiero** avanzar el despacho a «En camino», «Entregado» o «Devuelto» **para** reflejar el seguimiento.

| | |
|---|---|
| **Módulo** | Delivery |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `PUT /api/v1/Delivery/{id}/estado` |
| **Permiso** | `Delivery:Actualizar` |
| **Reglas** | [RN-DEL-009, RN-DEL-010, RN-DEL-011](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Despacho en camino
  Dado que el despacho está en "Preparando"
  Cuando llamo a "PUT /api/v1/Delivery/{id}/estado" con el cuerpo { id, nuevoEstado: "En_Camino" }
  Entonces recibo 200 con el identificador y el despacho queda "En_Camino"

Esquema del escenario: Transiciones
  Dado un despacho en <actual>
  Cuando pido el estado <nuevo>
  Entonces el resultado es <resultado>

  Ejemplos:
    | actual      | nuevo      | resultado                                          |
    | Preparando  | En_Camino  | permitido                                          |
    | Preparando  | Devuelto   | permitido                                          |
    | En_Camino   | Entregado  | permitido                                          |
    | En_Camino   | Devuelto   | permitido                                          |
    | Preparando  | Entregado  | 400 Delivery.Estado.TransicionInvalida             |
    | Entregado   | En_Camino  | 400 Delivery.Estado.TransicionInvalida             |
    | Devuelto    | En_Camino  | 400 Delivery.Estado.TransicionInvalida             |
    | Entregado   | Entregado  | válido, no cambia nada (el mismo estado)           |

Escenario: El cuerpo manda, no la ruta
  Entonces el identificador de la ruta no se usa: el cuerpo debe repetir el identificador
  Y un identificador vacío o un estado fuera del enumerado responde 400 "UpdateDeliveryEstadoCommand.Id" o "…NuevoEstado"

Escenario: El despacho no existe
  Entonces recibo 404 con el código literal "Delivery"

Escenario: No se comprueba la sucursal
  Entonces quien tenga "Delivery:Actualizar" puede cambiar el estado de un despacho de cualquier sucursal de su empresa

Escenario: Pasar a "Entregado" o "Devuelto" no tiene efecto en la venta ni el stock
  Entonces son solo seguimiento
```

### HU-OPS-020 — Consultar, «actualizar» y eliminar despachos de delivery

**Como** integrador **quiero** listar, ver y eliminar despachos **para** gestionar el seguimiento.

| | |
|---|---|
| **Módulo** | Delivery |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `GET /api/v1/Delivery/despachos`, `GET /api/v1/Delivery/{id}`, `PUT /api/v1/Delivery/{id}`, `DELETE /api/v1/Delivery/{id}` |
| **Permiso** | `Delivery:Leer` / `Delivery:Actualizar` / `Delivery:Eliminar` |
| **Reglas** | [RN-DEL-012, RN-DEL-013, RN-DEL-014, RN-DEL-015](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Parcial (editar los datos del despacho no está implementado y el detalle solo devuelve el identificador) |

**Criterios de aceptación**

```gherkin
Escenario: Listar despachos
  Cuando llamo a "GET /api/v1/Delivery/despachos" con estado y fechas desde y hasta
  Entonces recibo los despachos cuya venta es de una sucursal accesible, de a 10
  Y cada fila trae solo identificador, venta, estado y dirección de entrega
  Y las fechas se comparan tal cual con la fecha de creación, sin normalizar a UTC
  Y el orden es por identificador descendente, que no es cronológico

Escenario: El detalle solo devuelve el identificador
  Cuando llamo a "GET /api/v1/Delivery/{id}"
  Entonces recibo 200 con el objeto de resultado cuyo "value" tiene únicamente el identificador
  Y si no existe o es de una sucursal no accesible, el código literal "NotFound" con 404
  Y para ver el estado hay que usar el listado

Escenario: "Actualizar" un despacho no hace nada
  Cuando llamo a "PUT /api/v1/Delivery/{id}"
  Entonces recibo 200 con el objeto de resultado sin cambios
  Y plataforma, rastreo, dirección, repartidor y evidencia no se pueden corregir después de programar

Escenario: Eliminar un despacho
  Cuando llamo a "DELETE /api/v1/Delivery/{id}"
  Entonces el despacho queda eliminado (borrado lógico) y la venta queda libre para programar otro
  Y no se comprueba sucursal ni estado: se puede eliminar un despacho "Entregado"
  Y un despacho inexistente responde 404 con el código literal "NotFound"
```

### 12.6 Auditoría

### HU-OPS-021 — Solicitar la anonimización de datos

**Como** responsable de datos personales **quiero** pedir la anonimización de un registro **para** atender a un titular de datos (Ley 29733).

| | |
|---|---|
| **Módulo** | Auditoría |
| **Pantalla** | Solo API (sin pantalla) |
| **Endpoint(s)** | `POST /api/v1/Auditoria/anonimizar` |
| **Permiso** | `Auditoria:AnonimizarDatos` |
| **Reglas** | [RN-AUD-001, RN-AUD-002, RN-AUD-003, RN-AUD-004](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| **Estado** | Parcial (la operación existe pero responde 501 y no hace nada) |

**Criterios de aceptación**

```gherkin
Escenario: La anonimización responde "no implementado"
  Dado que tengo el permiso "Auditoria:AnonimizarDatos"
  Cuando llamo a "POST /api/v1/Auditoria/anonimizar" con la tabla "Pacientes" y un identificador de registro válido
  Entonces recibo 501 con el código "Auditoria.Anonimizacion.NoImplementada"
  Y el detalle "Data anonymization is not implemented yet; no data was changed."
  Y ningún dato se modifica y no se registra ningún evento de auditoría
  # No verificado: la forma exacta del ProblemDetails; se lee del código

Escenario: Datos obligatorios
  Dado que envío la tabla vacía o el identificador del registro vacío o igual al GUID vacío
  Entonces recibo 400 "AnonimizarDatosCommand.Tabla" o "…RegistroId"
  Y no se valida que la tabla exista ni que sea anonimizable

Escenario: Orden de evaluación
  Entonces primero se comprueba el permiso, luego el validador y por último el handler que responde 501

Escenario: Por qué 501 y no un falso éxito
  Entonces la decisión evita confirmar falsamente la solicitud de un titular de datos
```

### HU-OPS-022 — Saber qué operaciones dejan evento de auditoría

**Como** auditor **quiero** conocer qué hechos quedan registrados y cómo consultarlos **para** saber hasta dónde llega el rastro del sistema.

| | |
|---|---|
| **Módulo** | Auditoría |
| **Pantalla** | Solo API (sin pantalla; las carpetas de la pantalla de auditoría están vacías) |
| **Endpoint(s)** | Ninguno: no existe endpoint que lea los eventos |
| **Permiso** | Ninguno (el único permiso del módulo es `Auditoria:AnonimizarDatos`) |
| **Reglas** | [RN-AUD-005, RN-AUD-006, RN-AUD-007, RN-AUD-008, RN-AUD-009, RN-AUD-010](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-GEN-013, RN-GEN-014](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Parcial (los eventos se escriben pero no existe forma de consultarlos por API ni por pantalla) |

**Criterios de aceptación**

```gherkin
Escenario: Un comando auditable exitoso deja un evento
  Dado que anulo una venta
  Entonces se guarda una fila en "Auditoria_Eventos_Seguridad" con el empleado que actúa, la tabla afectada "Ventas_Cabecera", el identificador del registro, la acción "VENTA_ANULAR", un detalle breve ("estado: Anulada"), la IP y la fecha-hora UTC

Esquema del escenario: Eventos de esta parte
  Dado que el comando <comando> termina con éxito
  Entonces se registra el evento <evento> sobre la tabla <tabla>

  Ejemplos:
    | comando                | evento              | tabla                |
    | AnularVentaCommand     | VENTA_ANULAR        | Ventas_Cabecera      |
    | ProcesarDevolucion     | DEVOLUCION_PROCESAR | Devoluciones_Cabecera|
    | AperturarCajaCommand   | CAJA_APERTURAR      | POS_Sesiones_Caja    |
    | CerrarCajaCommand      | CAJA_CERRAR         | POS_Sesiones_Caja    |
    | AdjustStockCommand     | STOCK_AJUSTAR       | Inventario_Sucursal  |

Escenario: Lo que no deja evento
  Entonces crear, cobrar o actualizar una venta, y todas las operaciones de transferencias, delivery, magistrales, servicios, cuarentena y actas, no dejan evento de auditoría de seguridad
  Y solo queda el usuario y la fecha de creación y modificación de las filas

Escenario: El valor anterior no se guarda
  Entonces "Payload_Anterior" queda siempre vacío y solo se guarda una descripción del hecho
  Y un flujo sin empleado en la sesión no deja evento

Escenario: No se pueden consultar los eventos
  Entonces ningún endpoint, consulta ni exportación lee los eventos; la tabla solo se lee directamente en la base de datos
  Y el rol «Auditor» sembrado no tiene con qué leer

Escenario: Los eventos son de solo agregar
  Entonces el código nunca modifica ni elimina eventos, pero ningún disparador de la base lo impide: la inmutabilidad depende solo del código

Escenario: Estructura
  Entonces la tabla exige un empleado y rechaza referencias inexistentes; los guards de la entidad nunca llegan al cliente
```

> **Brecha conocida.** «La auditoría solo se escribe: no hay consulta de eventos» (01d hallazgo 6). Los textos de `auditoria/es.json` («Auditoría del Sistema», «No se encontraron registros de auditoría.») describen una pantalla que no existe.

---

## 13. Trazabilidad

La tabla resume las filas «Reglas», «Pantalla» y «Endpoint(s)» de cada historia (las pantallas y endpoints se abrevian). La columna «Reglas» se generó con un script de lectura a partir de la fila «Reglas» de cada historia (sección 15.4).

| Historia | Reglas | Pantallas | Endpoints |
|---|---|---|---|
| [HU-VEN-001](#hu-ven-001--entrar-al-terminal-de-venta-con-la-caja-propia-abierta) | RN-VEN-001 a RN-VEN-005, RN-VEN-077 | `/pos/nueva` — «Terminal de Punto de Venta (POS)» | `GET /api/v1/pos/sesiones`, `GET /api/v1/ventas/configuracion` |
| [HU-VEN-002](#hu-ven-002--buscar-productos-y-elegir-el-lote) | RN-VEN-001, RN-VEN-011 a RN-VEN-014, RN-VEN-075 | `/pos/nueva` — columna «Producto» | `GET /api/v1/medicamentos`, `GET /api/v1/medicamentos/condiciones-venta`, `GET /api/v1/ventas/lotes-disponibles` |
| [HU-VEN-003](#hu-ven-003--agregar-productos-al-carrito) | RN-VEN-008 a RN-VEN-011, RN-VEN-014, RN-VEN-019 a RN-VEN-024, RN-VEN-080 | `/pos/nueva` — formulario de agregado y «Resumen de Venta» | Validación local; `POST /api/v1/ventas` al cobrar |
| [HU-VEN-004](#hu-ven-004--vender-por-presentación-caja-blíster) | RN-PRE-001 a RN-PRE-015, RN-VEN-017 a RN-VEN-020, RN-VEN-076 | `/pos/nueva` — selector «Presentación» | `GET /api/v1/ventas/presentaciones`, `POST /api/v1/ventas` |
| [HU-VEN-005](#hu-ven-005--ajustar-quitar-y-vaciar-el-carrito) | RN-VEN-008, RN-VEN-014, RN-VEN-024, RN-VEN-045 | `/pos/nueva` — «Resumen de Venta» | Ninguno |
| [HU-VEN-006](#hu-ven-006--recuperar-la-venta-en-curso-tras-recargar-borrador-local) | RN-VEN-045, RN-VEN-053 | `/pos/nueva` (automático) | `GET /api/v1/pacientes/{id}` |
| [HU-VEN-007](#hu-ven-007--elegir-al-cliente-de-la-venta) | RN-VEN-025 a RN-VEN-027, RN-CMP-005 | `/pos/nueva` — «Datos del Cliente» | `GET /api/v1/pacientes` |
| [HU-VEN-008](#hu-ven-008--vender-productos-que-exigen-receta-registrada) | RN-VEN-026, RN-VEN-028 a RN-VEN-032, RN-VEN-036, RN-VEN-037, RN-VEN-077 | `/pos/nueva` — «Receta médica» | `GET /api/v1/recetas`, `POST /api/v1/ventas` |
| [HU-VEN-009](#hu-ven-009--confirmar-receta-verificada-en-empresas-flexibles) | RN-VEN-028, RN-VEN-029, RN-VEN-034, RN-VEN-035, RN-VEN-037, RN-EMP-016 | `/pos/nueva` — «Receta verificada»; `/pos/:id` | `POST /api/v1/ventas`, `PUT /api/v1/ventas/{id}/completar`, `GET /api/v1/ventas/{id}` |
| [HU-VEN-010](#hu-ven-010--respetar-el-máximo-de-dispensaciones-de-una-receta) | RN-VEN-033, RN-REC-031, RN-DEV-026 | `/pos/nueva` (al cobrar) | `POST /api/v1/ventas`, `PUT /api/v1/ventas/{id}/completar` |
| [HU-VEN-011](#hu-ven-011--agregar-al-carrito-los-medicamentos-de-una-receta-escaneada) | RN-VEN-036, RN-VEN-075 | `/pos/nueva` — pestaña «Receta» | `POST /api/v1/recetas/analizar`, `GET /api/v1/ventas/lotes-disponibles` |
| [HU-VEN-012](#hu-ven-012--cobrar-en-efectivo-y-entregar-el-vuelto) | RN-VEN-039, RN-VEN-041, RN-VEN-043, RN-VEN-044, RN-VEN-046, RN-VEN-081 | `/pos/nueva` — «Método de Pago» (Efectivo) | `POST /api/v1/ventas` |
| [HU-VEN-013](#hu-ven-013--cobrar-con-yape--plin-tarjeta-o-transferencia) | RN-VEN-040, RN-VEN-042, RN-VEN-081 | `/pos/nueva` — «Método de Pago» | `POST /api/v1/ventas` |
| [HU-VEN-014](#hu-ven-014--dividir-el-pago-en-varios-métodos) | RN-VEN-039, RN-VEN-041 a RN-VEN-045 | `/pos/nueva` — pago dividido | `POST /api/v1/ventas` |
| [HU-VEN-015](#hu-ven-015--confirmar-y-procesar-la-venta) | RN-VEN-002, RN-VEN-015, RN-VEN-021, RN-VEN-049 a RN-VEN-051, RN-VEN-057, RN-VEN-058, RN-VEN-060, RN-VEN-061, RN-VEN-079 a RN-VEN-083 | `/pos/nueva`; diálogo «Confirmar Procesamiento de Venta» | `POST /api/v1/ventas` |
| [HU-VEN-016](#hu-ven-016--cobrar-solo-con-ruc-de-la-empresa-y-serie-de-boleta-activa) | RN-VEN-007, RN-CMP-002, RN-CMP-003, RN-EMP-003, RN-EMP-004, RN-SER-008 | `/pos/nueva` (al confirmar el cobro) | `POST /api/v1/ventas`, `PUT /api/v1/ventas/{id}/completar` |
| [HU-VEN-017](#hu-ven-017--emitir-la-boleta-al-cobrar) | RN-VEN-059, RN-CMP-001, RN-CMP-004 a RN-CMP-011, RN-CMP-013, RN-CMP-015 | Diálogo «Venta Procesada Exitosamente»; «Comprobante» en `/pos/:id` | `POST /api/v1/ventas`, `PUT /api/v1/ventas/{id}/completar`, `GET /api/v1/ventas/{id}` |
| [HU-VEN-018](#hu-ven-018--imprimir-y-reimprimir-el-comprobante) | RN-CMP-011, RN-CMP-012, RN-CMP-014 | Diálogo «Venta Procesada Exitosamente»; «Reimprimir Comprobante» en `/pos/:id` | Ninguno |
| [HU-VEN-019](#hu-ven-019--cubrir-parte-de-la-venta-con-una-aseguradora) | RN-VEN-023, RN-VEN-047, RN-VEN-048 | `/pos/nueva` — «Cobertura de Seguro (opcional)» (oculto) | `POST /api/v1/ventas`, `PUT /api/v1/ventas/{id}/completar` |
| [HU-VEN-020](#hu-ven-020--guardar-la-venta-en-espera) | RN-VEN-003, RN-VEN-004, RN-VEN-008, RN-VEN-015, RN-VEN-049, RN-VEN-051, RN-VEN-052, RN-VEN-059 | `/pos/nueva` — «Guardar y Atender Siguiente» | `POST /api/v1/ventas` (estado `Pendiente`) |
| [HU-VEN-021](#hu-ven-021--retomar-y-modificar-una-venta-en-espera) | RN-VEN-008 a RN-VEN-010, RN-VEN-025, RN-VEN-027, RN-VEN-053 a RN-VEN-055 | `/pos`; `/pos/:id`; `/pos/nueva?continuar={id}` | `GET /api/v1/ventas`, `GET /api/v1/ventas/{id}`, `PUT /api/v1/ventas/{id}/detalles` |
| [HU-VEN-022](#hu-ven-022--cobrar-una-venta-en-espera) | RN-VEN-002, RN-VEN-006, RN-VEN-007, RN-VEN-016, RN-VEN-027, RN-VEN-034, RN-VEN-038, RN-VEN-046, RN-VEN-056, RN-VEN-057, RN-VEN-060 | `/pos/nueva?continuar={id}` | `PUT /api/v1/ventas/{id}/completar` |
| [HU-VEN-023](#hu-ven-023--anular-una-venta-cobrada) | RN-VEN-002, RN-VEN-061 a RN-VEN-064, RN-VEN-067 a RN-VEN-069, RN-CMP-012, RN-CAJ-012 | `/pos/:id`; diálogo «Anular Venta» | `PUT /api/v1/ventas/{id}/anular` |
| [HU-VEN-024](#hu-ven-024--anular-una-venta-en-espera) | RN-VEN-052, RN-VEN-065, RN-VEN-067 a RN-VEN-069 | `/pos/:id` | `PUT /api/v1/ventas/{id}/anular` |
| [HU-VEN-025](#hu-ven-025--saber-por-qué-una-venta-no-se-puede-anular) | RN-VEN-062 a RN-VEN-066 | `/pos/:id`; diálogo «Anular Venta» | `PUT /api/v1/ventas/{id}/anular` |
| [HU-VEN-026](#hu-ven-026--buscar-las-ventas-de-mi-sucursal) | RN-VEN-001, RN-VEN-059, RN-VEN-070 a RN-VEN-073, RN-GEN-008, RN-GEN-009 | `/pos` — «Ventas» | `GET /api/v1/ventas`, `GET /api/v1/empleados`, `GET /api/v1/pacientes` |
| [HU-VEN-027](#hu-ven-027--ver-el-detalle-de-una-venta) | RN-VEN-001, RN-VEN-035, RN-VEN-059, RN-VEN-074, RN-CMP-013 | `/pos/:id` — «Detalle de Venta» | `GET /api/v1/ventas/{id}` |
| [HU-VEN-028](#hu-ven-028--exportar-ventas-a-excel) | RN-VEN-078, RN-IMP-008 a RN-IMP-011 | `/pos` — «Exportar» | `POST /api/v1/ventas/exportar` |
| [HU-CAJ-001](#hu-caj-001--abrir-la-caja-del-turno) | RN-CAJ-001 a RN-CAJ-006, RN-CAJ-025 a RN-CAJ-027 | `/sesiones-caja`; `/sesiones-caja/nueva` | `POST /api/v1/pos/apertura` |
| [HU-CAJ-002](#hu-caj-002--abrir-la-caja-desde-el-terminal-de-venta) | RN-CAJ-002 a RN-CAJ-005, RN-VEN-003 | `/pos/nueva` — diálogo «Aperturar Caja» | `POST /api/v1/pos/apertura` |
| [HU-CAJ-003](#hu-caj-003--cerrar-la-caja-y-declarar-el-efectivo-contado) | RN-CAJ-001, RN-CAJ-007 a RN-CAJ-010, RN-CAJ-015, RN-CAJ-017 a RN-CAJ-019 | `/sesiones-caja/:id` — «Cerrar Sesión» | `POST /api/v1/pos/cierre`, `GET /api/v1/pos/{id}` |
| [HU-CAJ-004](#hu-caj-004--calcular-el-efectivo-esperado-arqueo) | RN-CAJ-011 a RN-CAJ-014, RN-CAJ-017, RN-CAJ-018, RN-CAJ-027, RN-VEN-006, RN-VEN-044, RN-VEN-047, RN-VEN-067 | Sin pantalla propia (se ve en «Monto Cierre Calculado (S/)» y en el Inicio) | `POST /api/v1/pos/cierre`, `GET /api/v1/dashboard/caja` |
| [HU-CAJ-005](#hu-caj-005--entender-por-qué-no-se-puede-cerrar-una-caja) | RN-CAJ-008, RN-CAJ-010, RN-CAJ-016, RN-CAJ-025 | `/sesiones-caja/:id` — panel de cierre | `POST /api/v1/pos/cierre` |
| [HU-CAJ-006](#hu-caj-006--ver-el-resultado-del-arqueo-de-una-sesión) | RN-CAJ-015, RN-CAJ-020, RN-CAJ-023 | `/sesiones-caja/:id` — «Detalle de Sesión» | `GET /api/v1/pos/{id}` |
| [HU-CAJ-007](#hu-caj-007--buscar-sesiones-de-caja) | RN-CAJ-020 a RN-CAJ-022, RN-GEN-008, RN-GEN-009, RN-GEN-023 | `/sesiones-caja` | `GET /api/v1/pos/sesiones` |
| [HU-CAJ-008](#hu-caj-008--exportar-sesiones-de-caja-a-excel) | RN-CAJ-024, RN-IMP-008 a RN-IMP-011 | `/sesiones-caja` — «Exportar» | `POST /api/v1/pos/sesiones/exportar` |
| [HU-DEV-001](#hu-dev-001--devolver-parte-o-todo-lo-vendido) | RN-DEV-001, RN-DEV-002, RN-DEV-009, RN-DEV-010, RN-DEV-020, RN-DEV-025 a RN-DEV-029, RN-VEN-066 | `/pos/:id` — «Devolver»; «Procesar devolución» | `POST /api/v1/devoluciones` |
| [HU-DEV-002](#hu-dev-002--elegir-la-cantidad-y-el-destino-de-cada-línea-devuelta) | RN-DEV-009 a RN-DEV-013, RN-DEV-023, RN-DEV-024, RN-DEV-034, RN-CUA-011, RN-STK-011 | `/pos/:id` — tabla del diálogo | `POST /api/v1/devoluciones` |
| [HU-DEV-003](#hu-dev-003--reembolsar-al-cliente) | RN-DEV-005, RN-DEV-016 a RN-DEV-019, RN-DEV-035, RN-DEV-036, RN-CAJ-014 | `/pos/:id` — «Método de reembolso», «Reembolso estimado» | `POST /api/v1/devoluciones` |
| [HU-DEV-004](#hu-dev-004--emitir-la-nota-de-crédito-de-la-devolución) | RN-DEV-006, RN-DEV-007, RN-DEV-020 a RN-DEV-022, RN-DEV-033, RN-DEV-035 a RN-DEV-037, RN-CMP-015, RN-SER-004, RN-SER-013 | `/pos/:id` — «Devoluciones / Notas de crédito» | `POST /api/v1/devoluciones` |
| [HU-DEV-005](#hu-dev-005--indicar-el-motivo-sunat-y-el-sustento) | RN-DEV-014, RN-DEV-015, RN-DEV-033, RN-GEN-010 | `/pos/:id` — «Motivo SUNAT», «Sustento» | `POST /api/v1/devoluciones` |
| [HU-DEV-006](#hu-dev-006--saber-por-qué-no-se-puede-devolver-una-venta) | RN-DEV-002 a RN-DEV-008 | `/pos/:id` — diálogo «Procesar devolución» | `POST /api/v1/devoluciones` |
| [HU-DEV-007](#hu-dev-007--ver-las-devoluciones-de-una-venta) | RN-DEV-022, RN-DEV-032, RN-VEN-074 | `/pos/:id` — «Devuelto»; «Devoluciones / Notas de crédito» | `GET /api/v1/ventas/{id}` |
| [HU-DEV-008](#hu-dev-008--consultar-devoluciones-por-api) | RN-DEV-030 a RN-DEV-032 | Solo API | `GET /api/v1/devoluciones`, `GET /api/v1/devoluciones/{id}` |
| [HU-DAS-001](#hu-das-001--ver-el-resumen-de-ventas-de-hoy) | RN-DAS-002, RN-DAS-004 a RN-DAS-009 | `/dashboard` — «Ventas de hoy» | `GET /api/v1/dashboard/ventas` |
| [HU-DAS-002](#hu-das-002--ver-lo-cobrado-por-método-de-pago-y-por-sucursal) | RN-DAS-003, RN-DAS-010, RN-DAS-011 | `/dashboard` — «Cobrado por método de pago», «Ventas por sucursal» | `GET /api/v1/dashboard/ventas` |
| [HU-DAS-003](#hu-das-003--ver-la-tendencia-de-7-días-y-los-productos-más-vendidos) | RN-DAS-012, RN-DAS-013 | `/dashboard` — tendencia y más vendidos | `GET /api/v1/dashboard/ventas` |
| [HU-DAS-004](#hu-das-004--ver-mi-caja-y-las-cajas-abiertas) | RN-DAS-015, RN-DAS-016, RN-CAJ-011 a RN-CAJ-014 | `/dashboard` — «Caja» | `GET /api/v1/dashboard/caja` |
| [HU-DAS-005](#hu-das-005--ver-las-alertas-de-inventario) | RN-DAS-017 a RN-DAS-021, RN-LOT-006, RN-STK-017 | `/dashboard` — «Inventario» | `GET /api/v1/dashboard/inventario` |
| [HU-DAS-006](#hu-das-006--ver-los-lotes-en-cuarentena) | RN-DAS-022, RN-CUA-007 | `/dashboard` — «Lotes en cuarentena» | `GET /api/v1/dashboard/cuarentena` |
| [HU-DAS-007](#hu-das-007--ver-las-devoluciones-de-hoy) | RN-DAS-023 | `/dashboard` — «Devoluciones de hoy» | `GET /api/v1/dashboard/devoluciones` |
| [HU-DAS-008](#hu-das-008--ver-solo-lo-que-mi-rol-y-mis-sucursales-permiten) | RN-DAS-001, RN-DAS-003, RN-GEN-021, RN-GEN-023 | `/dashboard` | Los cinco `GET /api/v1/dashboard/*`; `GET /api/v1/auth/me` |
| [HU-DAS-009](#hu-das-009--actualizar-el-resumen-y-reintentar-una-sección) | RN-DAS-002, RN-DAS-024 | `/dashboard` — «Actualizar», «Reintentar» | Los cinco `GET /api/v1/dashboard/*` |
| [HU-DAS-010](#hu-das-010--abrir-los-listados-desde-las-tarjetas) | RN-DAS-014, RN-DAS-016 | `/dashboard` → `/pos`, `/sesiones-caja`, `/stock-por-sucursal` | Ninguno propio |
| [HU-OPS-001](#hu-ops-001--crear-una-transferencia-entre-sucursales) | RN-TRA-001 a RN-TRA-011 | Solo API | `POST /api/v1/Transferencias` |
| [HU-OPS-002](#hu-ops-002--despachar-una-transferencia) | RN-TRA-012 a RN-TRA-014 | Solo API | `PUT /api/v1/Transferencias/{id}/despachar` |
| [HU-OPS-003](#hu-ops-003--recibir-una-transferencia) | RN-TRA-015 a RN-TRA-020, RN-TRA-024 | Solo API | `PUT /api/v1/Transferencias/{id}/recibir` |
| [HU-OPS-004](#hu-ops-004--cancelar-una-transferencia) | RN-TRA-021 a RN-TRA-024 | Solo API | `PUT /api/v1/Transferencias/{id}/cancelar` |
| [HU-OPS-005](#hu-ops-005--consultar-transferencias) | RN-TRA-025 a RN-TRA-030 | Solo API | `GET /api/v1/Transferencias`, `GET /api/v1/Transferencias/{id}` |
| [HU-OPS-006](#hu-ops-006--aislar-un-lote-en-cuarentena) | RN-CUA-001 a RN-CUA-006, RN-CUA-010, RN-CUA-011, RN-CUA-020, RN-DEV-024 | Solo API | `POST /api/v1/DIGEMID/cuarentena` |
| [HU-OPS-007](#hu-ops-007--bloquear-la-venta-de-un-lote-retenido-y-sus-límites) | RN-CUA-007 a RN-CUA-009, RN-VEN-012 | Solo API para el registro; efecto en el terminal y el Inicio | `POST /api/v1/DIGEMID/cuarentena`; `POST /api/v1/ventas` |
| [HU-OPS-008](#hu-ops-008--consultar-la-cuarentena) | RN-CUA-012, RN-GEN-008, RN-GEN-009, RN-GEN-023 | Solo API | `GET /api/v1/DIGEMID/cuarentena` |
| [HU-OPS-009](#hu-ops-009--generar-un-acta-de-destrucción) | RN-CUA-013 a RN-CUA-018, RN-CUA-021 a RN-CUA-023 | Solo API | `POST /api/v1/DIGEMID/acta-destruccion` |
| [HU-OPS-010](#hu-ops-010--consultar-las-actas-de-destrucción) | RN-CUA-019 | Solo API | `GET /api/v1/DIGEMID/actas` |
| [HU-OPS-011](#hu-ops-011--iniciar-una-orden-magistral) | RN-MAG-001 a RN-MAG-011, RN-MAG-020 a RN-MAG-023 | Solo API | `POST /api/v1/Magistrales/iniciar` |
| [HU-OPS-012](#hu-ops-012--completar-una-orden-magistral) | RN-MAG-012 a RN-MAG-017, RN-LOT-004, RN-LOT-006, RN-LOT-010 | Solo API | `POST /api/v1/Magistrales/completar` |
| [HU-OPS-013](#hu-ops-013--consultar-órdenes-magistrales) | RN-MAG-018, RN-MAG-019 | Solo API | `GET /api/v1/Magistrales/ordenes`, `GET /api/v1/Magistrales/{id}` |
| [HU-OPS-014](#hu-ops-014--agendar-un-servicio) | RN-SRV-001 a RN-SRV-005, RN-SRV-019, RN-SRV-021 | Solo API | `POST /api/v1/Servicios/agenda` |
| [HU-OPS-015](#hu-ops-015--registrar-una-inmunización) | RN-SRV-007 a RN-SRV-013, RN-SRV-015, RN-SRV-020 | Solo API | `POST /api/v1/Servicios/inmunizacion` |
| [HU-OPS-016](#hu-ops-016--consultar-la-agenda-y-las-inmunizaciones) | RN-SRV-006, RN-SRV-014, RN-SRV-016 | Solo API | Cuatro `GET /api/v1/Servicios/…` |
| [HU-OPS-017](#hu-ops-017--actualizar-o-eliminar-una-cita-o-una-inmunización) | RN-SRV-017, RN-SRV-018, RN-SRV-021 | Solo API | `PUT` y `DELETE` de `servicio/{id}` e `inmunizacion/{id}` |
| [HU-OPS-018](#hu-ops-018--programar-un-delivery) | RN-DEL-001 a RN-DEL-008, RN-DEL-016 a RN-DEL-018 | Solo API | `POST /api/v1/Delivery` |
| [HU-OPS-019](#hu-ops-019--cambiar-el-estado-de-un-delivery) | RN-DEL-009 a RN-DEL-011 | Solo API | `PUT /api/v1/Delivery/{id}/estado` |
| [HU-OPS-020](#hu-ops-020--consultar-actualizar-y-eliminar-despachos-de-delivery) | RN-DEL-012 a RN-DEL-015 | Solo API | `GET /api/v1/Delivery/despachos`, `GET`, `PUT` y `DELETE /api/v1/Delivery/{id}` |
| [HU-OPS-021](#hu-ops-021--solicitar-la-anonimización-de-datos) | RN-AUD-001 a RN-AUD-004 | Solo API | `POST /api/v1/Auditoria/anonimizar` |
| [HU-OPS-022](#hu-ops-022--saber-qué-operaciones-dejan-evento-de-auditoría) | RN-AUD-005 a RN-AUD-010, RN-GEN-013, RN-GEN-014 | Solo API | Ninguno |

---

## 14. Pendientes y «No verificado»

### 14.1 Historias en estado «Parcial» (qué falta)

| Historia | Qué falta o no hace el sistema |
|---|---|
| HU-VEN-017 | La boleta se registra solo en SOFIA: no se envía a SUNAT (sin OSE/PSE, hash, XML, constancia ni URL), y las ventas nunca emiten factura |
| HU-VEN-019 | La cobertura de seguro está completa en la API, pero el bloque de la pantalla está oculto con una constante del código |
| HU-CAJ-005 | Con un efectivo esperado negativo no hay otra vía para cerrar la caja |
| HU-CAJ-007 | La pantalla no ofrece filtro por sucursal ni por cajero ni vista «solo mis sesiones» |
| HU-DEV-002 | El destino «Cuarentena DIGEMID» solo queda anotado: no repone stock ni crea el registro de cuarentena |
| HU-DEV-004 | La nota de crédito no se envía a SUNAT y la pantalla solo muestra su número (no la imprime) |
| HU-DEV-007 | Las devoluciones solo se ven dentro del detalle de la venta; no hay listado ni detalle de devoluciones |
| HU-OPS-007 | Solo la venta respeta la cuarentena; no existe forma de liberar un lote retenido |
| HU-OPS-009 | El acta de destrucción es solo una cabecera (sin lotes ni cantidades, sin cambio de estado) |
| HU-OPS-011 | No hay cancelación ni devolución de insumos de una orden iniciada |
| HU-OPS-014 | Validador vacío; el guardado podría fallar por columnas que faltan en el script de la tabla |
| HU-OPS-015 | Registrar la dosis no descuenta stock ni consulta la cuarentena |
| HU-OPS-016 | Los detalles por identificador solo devuelven el identificador |
| HU-OPS-017 | Actualizar no está implementado (responde 200 sin cambiar nada) |
| HU-OPS-020 | Editar el despacho no está implementado y el detalle solo devuelve el identificador |
| HU-OPS-021 | La anonimización responde 501 y no hace nada |
| HU-OPS-022 | Los eventos se escriben, pero no hay forma de consultarlos |

Otras brechas relevantes (descritas en las notas «Brecha conocida» de cada historia): los rechazos de negocio del cobro se ven casi todos como «Ocurrió un error al procesar la venta.»; el terminal no muestra la fecha de vencimiento del lote ni cuántas dispensaciones le quedan a la receta; la política «exigir registro de receta» de la empresa no tiene pantalla (RN-EMP-016); el panel de cierre de caja no muestra el esperado ni el resultado del arqueo; las exportaciones de ventas y de sesiones formatean la fecha en UTC; «Vendido hoy» incluye ventas «Devuelta» y «Ver ventas de hoy» no.

### 14.2 No verificado

| # | Qué no se verificó | Por qué | Historias |
|---|---|---|---|
| 1 | Los mensajes y comportamientos de pantalla en la aplicación corriendo | Los escenarios salen de los documentos validados (que recorrieron el código) y de los `es.json`; no se usó el navegador. Los textos entre comillas se contrastaron con los `es.json` (sección 15.4) | Todas |
| 2 | Cómo convierte SQL Server los valores con más decimales de los que admiten las columnas (precio unitario, cantidad, pagos, total, monto de apertura y cierre, cantidad devuelta, volumen de dosis) y el tope de 99 999 999,99 de los montos de caja | Depende del controlador de base de datos; no se ejecutó nada | HU-VEN-003, HU-VEN-004, HU-VEN-012, HU-CAJ-001, HU-DEV-002, HU-OPS-015 |
| 3 | Comportamiento ante peticiones simultáneas (dos cobros de la misma venta en espera, cobro y anulación a la vez, dos cierres, dos devoluciones, dos despachos o recepciones, dos «completar» de una orden magistral) | Se lee la versión de fila y los mapas de errores; no se probó con concurrencia | HU-VEN-015, HU-VEN-022, HU-VEN-023, HU-CAJ-005, HU-DEV-001, HU-OPS-002 a HU-OPS-004, HU-OPS-012 |
| 4 | Respuesta exacta de la API (código y cuerpo) ante un cuerpo que no se puede convertir, un enumerado enviado como texto, o la falta de permiso | Ocurre antes de los validadores o en el middleware (RN-GEN-020, RN-GEN-041); no se llamó a la API | HU-OPS-019, HU-OPS-015 y las demás historias de API |
| 5 | Que la exportación de ventas y de sesiones muestre la fecha en UTC | Se lee en el código; no se descargaron los archivos | HU-VEN-028, HU-CAJ-008 |
| 6 | Formato numérico y separador decimal de los importes, del porcentaje del Inicio y diferencia de un centavo entre subtotal, IGV y total en la pantalla | El formato depende del idioma activo; no se vio la pantalla | HU-VEN-003, HU-VEN-017, HU-DAS-001 |
| 7 | Cómo se cambia la sucursal activa del token y si un administrador puede operar otra sede desde el POS o la caja | Está fuera de este alcance (autenticación y sucursales) | HU-VEN-026, HU-CAJ-001 |
| 8 | Comportamientos del terminal deducidos de la lectura: Enter en «Cantidad», cliente que no se puede quitar, montos de pago que no se reajustan al cambiar el carrito, `localStorage` bloqueado, aseguradora en un borrador antiguo | No se ejecutó la pantalla | HU-VEN-006, HU-VEN-007, HU-VEN-012, HU-VEN-019 |
| 9 | Qué nombre muestran los listados cuando el empleado o la sucursal de una venta o sesión fue eliminado | Depende de cómo EF aplica el filtro global dentro de una proyección | HU-VEN-026, HU-CAJ-007 |
| 10 | Si una venta creada mientras otro usuario cierra la misma caja puede quedar fuera del arqueo | No se probó con peticiones simultáneas | HU-CAJ-003 |
| 11 | Si la tabla `Servicios_Agenda` falla en ejecución por las columnas que EF mapea y el script no crea | No se consultó la base ni se llamó a la API | HU-OPS-014 |
| 12 | Si algún despliegue tiene permisos de transferencias, delivery, magistrales, servicios, cuarentena y actas o auditoría concedidos a un rol | Los permisos viven en datos | HU-OPS-001 a HU-OPS-022 |
| 13 | La forma exacta del `ProblemDetails` de la respuesta 501 | Se lee `ToProblemResult`; no se llamó a la API | HU-OPS-021 |

Los nombres de clientes, números de lote, de receta y de operación, y los prefijos de serie usados en los escenarios son ejemplos inventados; no son datos de la aplicación.

### 14.3 En curso (sin commitear)

Ninguno. `git status --short` está limpio en `sofia-apis` (`43f5529`) y en `sofia-web-spa` (`eeffbb6`) al 2026-10-26; ninguna afirmación de este documento se apoya en un archivo con cambios sin commitear.

---

## 15. Tabla de cobertura

### 15.1 Reglas de negocio del alcance → historias

Alcance: las reglas `RN-VEN` y `RN-CMP` de 01c2, las `RN-CAJ`, `RN-DEV` y `RN-DAS` de 01c3, las `RN-TRA`, `RN-DEL`, `RN-MAG`, `RN-SRV`, `RN-CUA` y `RN-AUD` de 01d y `RN-REC-031` de 01c1: **312 reglas**, todas con al menos una historia (tabla generada por el script de la sección 15.4 a partir de la fila «Reglas» de cada historia). Las reglas que describen estructura de base de datos o comprobaciones de entidad que ninguna ruta alcanza no producen un mensaje que vea el usuario; se cubren con un escenario de «Estructura», con una nota de brecha o solo con su mención en la fila «Reglas» de la historia indicada: `RN-VEN-082`, `RN-VEN-083`, `RN-CAJ-006`, `RN-TRA-027`, `RN-DEL-016`, `RN-MAG-022`, `RN-SRV-021`, `RN-CUA-006` y `RN-CUA-023`.

| Regla(s) | Historias |
|---|---|
| RN-AUD-001 a RN-AUD-004 | HU-OPS-021 |
| RN-AUD-005 a RN-AUD-010 | HU-OPS-022 |
| RN-CAJ-001 | HU-CAJ-001, HU-CAJ-003 |
| RN-CAJ-002 a RN-CAJ-005 | HU-CAJ-001, HU-CAJ-002 |
| RN-CAJ-006 | HU-CAJ-001 |
| RN-CAJ-007 | HU-CAJ-003 |
| RN-CAJ-008 | HU-CAJ-003, HU-CAJ-005 |
| RN-CAJ-009 | HU-CAJ-003 |
| RN-CAJ-010 | HU-CAJ-003, HU-CAJ-005 |
| RN-CAJ-011 | HU-CAJ-004, HU-DAS-004 |
| RN-CAJ-012 | HU-VEN-023, HU-CAJ-004, HU-DAS-004 |
| RN-CAJ-013 | HU-CAJ-004, HU-DAS-004 |
| RN-CAJ-014 | HU-CAJ-004, HU-DEV-003, HU-DAS-004 |
| RN-CAJ-015 | HU-CAJ-003, HU-CAJ-006 |
| RN-CAJ-016 | HU-CAJ-005 |
| RN-CAJ-017, RN-CAJ-018 | HU-CAJ-003, HU-CAJ-004 |
| RN-CAJ-019 | HU-CAJ-003 |
| RN-CAJ-020 | HU-CAJ-006, HU-CAJ-007 |
| RN-CAJ-021, RN-CAJ-022 | HU-CAJ-007 |
| RN-CAJ-023 | HU-CAJ-006 |
| RN-CAJ-024 | HU-CAJ-008 |
| RN-CAJ-025 | HU-CAJ-001, HU-CAJ-005 |
| RN-CAJ-026 | HU-CAJ-001 |
| RN-CAJ-027 | HU-CAJ-001, HU-CAJ-004 |
| RN-CMP-001 | HU-VEN-017 |
| RN-CMP-002, RN-CMP-003 | HU-VEN-016 |
| RN-CMP-004 | HU-VEN-017 |
| RN-CMP-005 | HU-VEN-007, HU-VEN-017 |
| RN-CMP-006 a RN-CMP-010 | HU-VEN-017 |
| RN-CMP-011 | HU-VEN-017, HU-VEN-018 |
| RN-CMP-012 | HU-VEN-018, HU-VEN-023 |
| RN-CMP-013 | HU-VEN-017, HU-VEN-027 |
| RN-CMP-014 | HU-VEN-018 |
| RN-CMP-015 | HU-VEN-017, HU-DEV-004 |
| RN-CUA-001 a RN-CUA-006 | HU-OPS-006 |
| RN-CUA-007 | HU-DAS-006, HU-OPS-007 |
| RN-CUA-008, RN-CUA-009 | HU-OPS-007 |
| RN-CUA-010 | HU-OPS-006 |
| RN-CUA-011 | HU-DEV-002, HU-OPS-006 |
| RN-CUA-012 | HU-OPS-008 |
| RN-CUA-013 a RN-CUA-018 | HU-OPS-009 |
| RN-CUA-019 | HU-OPS-010 |
| RN-CUA-020 | HU-OPS-006 |
| RN-CUA-021 a RN-CUA-023 | HU-OPS-009 |
| RN-DAS-001 | HU-DAS-008 |
| RN-DAS-002 | HU-DAS-001, HU-DAS-009 |
| RN-DAS-003 | HU-DAS-002, HU-DAS-008 |
| RN-DAS-004 a RN-DAS-009 | HU-DAS-001 |
| RN-DAS-010, RN-DAS-011 | HU-DAS-002 |
| RN-DAS-012, RN-DAS-013 | HU-DAS-003 |
| RN-DAS-014 | HU-DAS-010 |
| RN-DAS-015 | HU-DAS-004 |
| RN-DAS-016 | HU-DAS-004, HU-DAS-010 |
| RN-DAS-017 a RN-DAS-021 | HU-DAS-005 |
| RN-DAS-022 | HU-DAS-006 |
| RN-DAS-023 | HU-DAS-007 |
| RN-DAS-024 | HU-DAS-009 |
| RN-DEL-001 a RN-DEL-008 | HU-OPS-018 |
| RN-DEL-009 a RN-DEL-011 | HU-OPS-019 |
| RN-DEL-012 a RN-DEL-015 | HU-OPS-020 |
| RN-DEL-016 a RN-DEL-018 | HU-OPS-018 |
| RN-DEV-001 | HU-DEV-001 |
| RN-DEV-002 | HU-DEV-001, HU-DEV-006 |
| RN-DEV-003, RN-DEV-004 | HU-DEV-006 |
| RN-DEV-005 | HU-DEV-003, HU-DEV-006 |
| RN-DEV-006, RN-DEV-007 | HU-DEV-004, HU-DEV-006 |
| RN-DEV-008 | HU-DEV-006 |
| RN-DEV-009, RN-DEV-010 | HU-DEV-001, HU-DEV-002 |
| RN-DEV-011 a RN-DEV-013 | HU-DEV-002 |
| RN-DEV-014, RN-DEV-015 | HU-DEV-005 |
| RN-DEV-016 a RN-DEV-019 | HU-DEV-003 |
| RN-DEV-020 | HU-DEV-001, HU-DEV-004 |
| RN-DEV-021 | HU-DEV-004 |
| RN-DEV-022 | HU-DEV-004, HU-DEV-007 |
| RN-DEV-023 | HU-DEV-002 |
| RN-DEV-024 | HU-DEV-002, HU-OPS-006 |
| RN-DEV-025 | HU-DEV-001 |
| RN-DEV-026 | HU-VEN-010, HU-DEV-001 |
| RN-DEV-027 a RN-DEV-029 | HU-DEV-001 |
| RN-DEV-030, RN-DEV-031 | HU-DEV-008 |
| RN-DEV-032 | HU-DEV-007, HU-DEV-008 |
| RN-DEV-033 | HU-DEV-004, HU-DEV-005 |
| RN-DEV-034 | HU-DEV-002 |
| RN-DEV-035, RN-DEV-036 | HU-DEV-003, HU-DEV-004 |
| RN-DEV-037 | HU-DEV-004 |
| RN-MAG-001 a RN-MAG-011 | HU-OPS-011 |
| RN-MAG-012 a RN-MAG-017 | HU-OPS-012 |
| RN-MAG-018, RN-MAG-019 | HU-OPS-013 |
| RN-MAG-020 a RN-MAG-023 | HU-OPS-011 |
| RN-REC-031 | HU-VEN-010 |
| RN-SRV-001 a RN-SRV-005 | HU-OPS-014 |
| RN-SRV-006 | HU-OPS-016 |
| RN-SRV-007 a RN-SRV-013 | HU-OPS-015 |
| RN-SRV-014 | HU-OPS-016 |
| RN-SRV-015 | HU-OPS-015 |
| RN-SRV-016 | HU-OPS-016 |
| RN-SRV-017, RN-SRV-018 | HU-OPS-017 |
| RN-SRV-019 | HU-OPS-014 |
| RN-SRV-020 | HU-OPS-015 |
| RN-SRV-021 | HU-OPS-014, HU-OPS-017 |
| RN-TRA-001 a RN-TRA-011 | HU-OPS-001 |
| RN-TRA-012 a RN-TRA-014 | HU-OPS-002 |
| RN-TRA-015 a RN-TRA-020 | HU-OPS-003 |
| RN-TRA-021 a RN-TRA-023 | HU-OPS-004 |
| RN-TRA-024 | HU-OPS-003, HU-OPS-004 |
| RN-TRA-025 a RN-TRA-030 | HU-OPS-005 |
| RN-VEN-001 | HU-VEN-001, HU-VEN-002, HU-VEN-026, HU-VEN-027 |
| RN-VEN-002 | HU-VEN-001, HU-VEN-015, HU-VEN-022, HU-VEN-023 |
| RN-VEN-003 | HU-VEN-001, HU-VEN-020, HU-CAJ-002 |
| RN-VEN-004 | HU-VEN-001, HU-VEN-020 |
| RN-VEN-005 | HU-VEN-001 |
| RN-VEN-006 | HU-VEN-022, HU-CAJ-004 |
| RN-VEN-007 | HU-VEN-016, HU-VEN-022 |
| RN-VEN-008 | HU-VEN-003, HU-VEN-005, HU-VEN-020, HU-VEN-021 |
| RN-VEN-009, RN-VEN-010 | HU-VEN-003, HU-VEN-021 |
| RN-VEN-011 | HU-VEN-002, HU-VEN-003 |
| RN-VEN-012 | HU-VEN-002, HU-OPS-007 |
| RN-VEN-013 | HU-VEN-002 |
| RN-VEN-014 | HU-VEN-002, HU-VEN-003, HU-VEN-005 |
| RN-VEN-015 | HU-VEN-015, HU-VEN-020 |
| RN-VEN-016 | HU-VEN-022 |
| RN-VEN-017, RN-VEN-018 | HU-VEN-004 |
| RN-VEN-019, RN-VEN-020 | HU-VEN-003, HU-VEN-004 |
| RN-VEN-021 | HU-VEN-003, HU-VEN-015 |
| RN-VEN-022 | HU-VEN-003 |
| RN-VEN-023 | HU-VEN-003, HU-VEN-019 |
| RN-VEN-024 | HU-VEN-003, HU-VEN-005 |
| RN-VEN-025 | HU-VEN-007, HU-VEN-021 |
| RN-VEN-026 | HU-VEN-007, HU-VEN-008 |
| RN-VEN-027 | HU-VEN-007, HU-VEN-021, HU-VEN-022 |
| RN-VEN-028, RN-VEN-029 | HU-VEN-008, HU-VEN-009 |
| RN-VEN-030 a RN-VEN-032 | HU-VEN-008 |
| RN-VEN-033 | HU-VEN-010 |
| RN-VEN-034 | HU-VEN-009, HU-VEN-022 |
| RN-VEN-035 | HU-VEN-009, HU-VEN-027 |
| RN-VEN-036 | HU-VEN-008, HU-VEN-011 |
| RN-VEN-037 | HU-VEN-008, HU-VEN-009 |
| RN-VEN-038 | HU-VEN-022 |
| RN-VEN-039 | HU-VEN-012, HU-VEN-014 |
| RN-VEN-040 | HU-VEN-013 |
| RN-VEN-041 | HU-VEN-012, HU-VEN-014 |
| RN-VEN-042 | HU-VEN-013, HU-VEN-014 |
| RN-VEN-043 | HU-VEN-012, HU-VEN-014 |
| RN-VEN-044 | HU-VEN-012, HU-VEN-014, HU-CAJ-004 |
| RN-VEN-045 | HU-VEN-005, HU-VEN-006, HU-VEN-014 |
| RN-VEN-046 | HU-VEN-012, HU-VEN-022 |
| RN-VEN-047 | HU-VEN-019, HU-CAJ-004 |
| RN-VEN-048 | HU-VEN-019 |
| RN-VEN-049 | HU-VEN-015, HU-VEN-020 |
| RN-VEN-050 | HU-VEN-015 |
| RN-VEN-051 | HU-VEN-015, HU-VEN-020 |
| RN-VEN-052 | HU-VEN-020, HU-VEN-024 |
| RN-VEN-053 | HU-VEN-006, HU-VEN-021 |
| RN-VEN-054, RN-VEN-055 | HU-VEN-021 |
| RN-VEN-056 | HU-VEN-022 |
| RN-VEN-057 | HU-VEN-015, HU-VEN-022 |
| RN-VEN-058 | HU-VEN-015 |
| RN-VEN-059 | HU-VEN-017, HU-VEN-020, HU-VEN-026, HU-VEN-027 |
| RN-VEN-060 | HU-VEN-015, HU-VEN-022 |
| RN-VEN-061 | HU-VEN-015, HU-VEN-023 |
| RN-VEN-062 a RN-VEN-064 | HU-VEN-023, HU-VEN-025 |
| RN-VEN-065 | HU-VEN-024, HU-VEN-025 |
| RN-VEN-066 | HU-VEN-025, HU-DEV-001 |
| RN-VEN-067 | HU-VEN-023, HU-VEN-024, HU-CAJ-004 |
| RN-VEN-068, RN-VEN-069 | HU-VEN-023, HU-VEN-024 |
| RN-VEN-070 a RN-VEN-073 | HU-VEN-026 |
| RN-VEN-074 | HU-VEN-027, HU-DEV-007 |
| RN-VEN-075 | HU-VEN-002, HU-VEN-011 |
| RN-VEN-076 | HU-VEN-004 |
| RN-VEN-077 | HU-VEN-001, HU-VEN-008 |
| RN-VEN-078 | HU-VEN-028 |
| RN-VEN-079 | HU-VEN-015 |
| RN-VEN-080 | HU-VEN-003, HU-VEN-015 |
| RN-VEN-081 | HU-VEN-012, HU-VEN-013, HU-VEN-015 |
| RN-VEN-082, RN-VEN-083 | HU-VEN-015 |

### 15.2 Rutas y diálogos del SPA → historias

| Ruta o diálogo | Historias |
|---|---|
| `/sesiones-caja` | HU-CAJ-001, HU-CAJ-007, HU-CAJ-008 |
| `/sesiones-caja/nueva` | HU-CAJ-001 |
| `/sesiones-caja/:id` | HU-CAJ-003, HU-CAJ-004, HU-CAJ-005, HU-CAJ-006 |
| `/pos` | HU-VEN-021, HU-VEN-026, HU-VEN-028 |
| `/pos/nueva` (incluido `?continuar=<id>`) | HU-VEN-001 a HU-VEN-015, HU-VEN-016, HU-VEN-019, HU-VEN-020, HU-VEN-021, HU-VEN-022 |
| `/pos/:id` | HU-VEN-009, HU-VEN-017, HU-VEN-018, HU-VEN-021, HU-VEN-023 a HU-VEN-025, HU-VEN-027, HU-DEV-001 a HU-DEV-007 |
| `/dashboard` | HU-DAS-001 a HU-DAS-010 |
| Diálogo «Aperturar Caja» (`AperturarCajaDialogComponent`) | HU-CAJ-002, HU-VEN-001 |
| Diálogo «Confirmar Procesamiento de Venta» (`ConfirmarVentaDialogComponent`) | HU-VEN-015 |
| Diálogo «Venta Procesada Exitosamente» (`ComprobanteDialogComponent`) | HU-VEN-017, HU-VEN-018 |
| Diálogo «Anular Venta» (`AnularVentaDialogComponent`) | HU-VEN-023, HU-VEN-024, HU-VEN-025 |
| Diálogo «Procesar devolución» (`DevolverVentaDialogComponent`) | HU-DEV-001 a HU-DEV-006 |
| Menú «Punto de Venta» → «Sesiones de Caja» y «Ventas» | HU-CAJ-007, HU-VEN-026 |
| Sin opción de menú (terminal, devoluciones) | HU-VEN-001, HU-DEV-007 |

### 15.3 Endpoints del alcance → historias

Los 54 endpoints de `Ventas`, `POS`, `Devoluciones`, `Dashboard`, `Transferencias`, `Delivery`, `Magistrales`, `Servicios`, `Auditoria` y las cuatro operaciones de cuarentena y actas de `DIGEMID` figuran en [`endpoints.csv`](../../_inventario/endpoints.csv) y aparecen en la fila «Endpoint(s)» de alguna historia.

| Método y ruta | Permiso | Historias |
|---|---|---|
| `POST /api/v1/Auditoria/anonimizar` | `Auditoria:AnonimizarDatos` | HU-OPS-021 |
| `POST /api/v1/DIGEMID/cuarentena` | `DIGEMID:AislarLoteCuarentena` | HU-OPS-006, HU-OPS-007 |
| `POST /api/v1/DIGEMID/acta-destruccion` | `DIGEMID:GenerarActaDestruccion` | HU-OPS-009 |
| `GET /api/v1/DIGEMID/cuarentena` | `DIGEMID:Leer` | HU-OPS-008 |
| `GET /api/v1/DIGEMID/actas` | `DIGEMID:Leer` | HU-OPS-010 |
| `GET /api/v1/Dashboard/ventas` | `Ventas:Leer` | HU-DAS-001, HU-DAS-002, HU-DAS-003, HU-DAS-008, HU-DAS-009 |
| `GET /api/v1/Dashboard/caja` | `POS:Leer` | HU-CAJ-004, HU-DAS-004, HU-DAS-008, HU-DAS-009 |
| `GET /api/v1/Dashboard/inventario` | `Inventarios:Leer` | HU-DAS-005, HU-DAS-008, HU-DAS-009 |
| `GET /api/v1/Dashboard/cuarentena` | `DIGEMID:Leer` | HU-DAS-006, HU-DAS-008, HU-DAS-009 |
| `GET /api/v1/Dashboard/devoluciones` | `Devoluciones:Leer` | HU-DAS-007, HU-DAS-008, HU-DAS-009 |
| `POST /api/v1/Delivery` | `Delivery:ProgramarDelivery` | HU-OPS-018 |
| `PUT /api/v1/Delivery/{id}/estado` | `Delivery:Actualizar` | HU-OPS-019 |
| `GET /api/v1/Delivery/despachos` | `Delivery:Leer` | HU-OPS-020 |
| `GET /api/v1/Delivery/{id}` | `Delivery:Leer` | HU-OPS-020 |
| `PUT /api/v1/Delivery/{id}` | `Delivery:Actualizar` | HU-OPS-020 |
| `DELETE /api/v1/Delivery/{id}` | `Delivery:Eliminar` | HU-OPS-020 |
| `POST /api/v1/Devoluciones` | `Devoluciones:ProcesarDevolucion` | HU-DEV-001 a HU-DEV-006 |
| `GET /api/v1/Devoluciones` | `Devoluciones:Leer` | HU-DEV-008 |
| `GET /api/v1/Devoluciones/{id}` | `Devoluciones:Leer` | HU-DEV-008 |
| `POST /api/v1/Magistrales/iniciar` | `Magistrales:IniciarOrdenMagistral` | HU-OPS-011 |
| `POST /api/v1/Magistrales/completar` | `Magistrales:CompletarOrdenMagistral` | HU-OPS-012 |
| `GET /api/v1/Magistrales/ordenes` | `Magistrales:Leer` | HU-OPS-013 |
| `GET /api/v1/Magistrales/{id}` | `Magistrales:Leer` | HU-OPS-013 |
| `POST /api/v1/POS/apertura` | `POS:AperturarCaja` | HU-CAJ-001, HU-CAJ-002 |
| `POST /api/v1/POS/cierre` | `POS:CerrarCaja` | HU-CAJ-003, HU-CAJ-004, HU-CAJ-005 |
| `GET /api/v1/POS/sesiones` | `POS:Leer` | HU-VEN-001, HU-CAJ-007 |
| `GET /api/v1/POS/{id}` | `POS:Leer` | HU-CAJ-003, HU-CAJ-006 |
| `POST /api/v1/POS/sesiones/exportar` | `POS:Leer` | HU-CAJ-008 |
| `POST /api/v1/Servicios/agenda` | `Servicios:AgendarServicio` | HU-OPS-014 |
| `POST /api/v1/Servicios/inmunizacion` | `Servicios:RegistrarInmunizacion` | HU-OPS-015 |
| `GET /api/v1/Servicios/agenda` | `Servicios:Leer` | HU-OPS-016 |
| `GET /api/v1/Servicios/inmunizaciones` | `Servicios:Leer` | HU-OPS-016 |
| `GET /api/v1/Servicios/servicio/{id}` | `Servicios:Leer` | HU-OPS-016 |
| `PUT /api/v1/Servicios/servicio/{id}` | `Servicios:Actualizar` | HU-OPS-017 |
| `DELETE /api/v1/Servicios/servicio/{id}` | `Servicios:Eliminar` | HU-OPS-017 |
| `GET /api/v1/Servicios/inmunizacion/{id}` | `Servicios:Leer` | HU-OPS-016 |
| `PUT /api/v1/Servicios/inmunizacion/{id}` | `Servicios:Actualizar` | HU-OPS-017 |
| `DELETE /api/v1/Servicios/inmunizacion/{id}` | `Servicios:Eliminar` | HU-OPS-017 |
| `GET /api/v1/Transferencias` | `Transferencias:Leer` | HU-OPS-005 |
| `GET /api/v1/Transferencias/{id}` | `Transferencias:Leer` | HU-OPS-005 |
| `POST /api/v1/Transferencias` | `Transferencias:Crear` | HU-OPS-001 |
| `PUT /api/v1/Transferencias/{id}/despachar` | `Transferencias:Actualizar` | HU-OPS-002 |
| `PUT /api/v1/Transferencias/{id}/recibir` | `Transferencias:Actualizar` | HU-OPS-003 |
| `PUT /api/v1/Transferencias/{id}/cancelar` | `Transferencias:Actualizar` | HU-OPS-004 |
| `GET /api/v1/Ventas` | `Ventas:Leer` | HU-VEN-021, HU-VEN-026 |
| `GET /api/v1/Ventas/{id}` | `Ventas:Leer` | HU-VEN-009, HU-VEN-017, HU-VEN-018, HU-VEN-021, HU-VEN-027, HU-DEV-007 |
| `POST /api/v1/Ventas` | `Ventas:Crear` | HU-VEN-003, HU-VEN-004, HU-VEN-008, HU-VEN-009, HU-VEN-010, HU-VEN-012 a HU-VEN-017, HU-VEN-019, HU-VEN-020, HU-OPS-007 |
| `GET /api/v1/Ventas/lotes-disponibles` | `Ventas:Crear` | HU-VEN-002, HU-VEN-011 |
| `GET /api/v1/Ventas/presentaciones` | `Ventas:Crear` | HU-VEN-004 |
| `GET /api/v1/Ventas/configuracion` | `Ventas:Crear` | HU-VEN-001 |
| `PUT /api/v1/Ventas/{id}/anular` | `Ventas:Anular` | HU-VEN-023, HU-VEN-024, HU-VEN-025 |
| `PUT /api/v1/Ventas/{id}/completar` | `Ventas:Actualizar` | HU-VEN-009, HU-VEN-010, HU-VEN-016, HU-VEN-017, HU-VEN-019, HU-VEN-022 |
| `PUT /api/v1/Ventas/{id}/detalles` | `Ventas:Actualizar` | HU-VEN-021 |
| `POST /api/v1/Ventas/exportar` | `Ventas:Leer` | HU-VEN-028 |

Sin pantalla (Solo API): los cuatro endpoints de cuarentena y actas, los de Transferencias, Delivery, Magistrales, Servicios y Auditoría, y `GET /api/v1/Devoluciones` y `GET /api/v1/Devoluciones/{id}`.

### 15.4 Verificación

Se ejecutaron scripts de lectura (Node, en el directorio temporal de la sesión, sin modificar ningún repositorio ni la carpeta de documentación) que:

1. Extrajeron los IDs `RN-…` definidos en los seis catálogos de reglas (01a, 01b, 01c1, 01c2, 01c3, 01d: filas de tabla que empiezan por `| RN-`) y comprobaron que **los 352 IDs citados en este documento existen** (ninguno inexistente; los rangos «RN-VEN-001 a RN-VEN-005» se expandieron).
2. Comprobaron que las **312 reglas del alcance** (VEN, CMP, CAJ, DEV, DAS, TRA, DEL, MAG, SRV, CUA y AUD de 01c2, 01c3 y 01d, más `RN-REC-031`) aparecen en la fila «Reglas» de alguna historia. La primera pasada dejó sin historia solo a `RN-DEV-034`, que se agregó a HU-DEV-002; resultado final: ninguna regla sin historia.
3. Comprobaron que los **54 endpoints** del alcance de `endpoints.csv` se mencionan en el documento (la tabla 15.3 asocia cada uno con sus historias, sin ninguno «SIN HISTORIA»), que las **7 rutas** del SPA (`/sesiones-caja`, `/sesiones-caja/nueva`, `/sesiones-caja/:id`, `/pos`, `/pos/nueva`, `/pos/:id`, `/dashboard`) y los **5 diálogos** de `pages/pos` (Aperturar Caja, Confirmar Procesamiento de Venta, Venta Procesada Exitosamente, Anular Venta y Procesar devolución) aparecen.
4. Extrajeron los 584 textos entre comillas del documento y los contrastaron con todos los `es.json` del SPA (`public/i18n/**/es.json`): los mensajes de interfaz coinciden con el archivo (las cadenas sin coincidencia son códigos de error, rutas, permisos, nombres de campo, datos de ejemplo y frases propias del documento). Se corrigió un texto: «Estado de caja» no es un título visible de la tarjeta (es solo una descripción del inventario de pantallas) y se quitó.

Resultado: 76 historias (46 «Implementada», 17 «Parcial», 13 «Solo API»), 312 reglas cubiertas, 7 rutas, 5 diálogos y 54 endpoints, sin elementos sin cubrir.

---

## 16. Fuentes

Documentos (todos bajo `sofia-docs/`):

- `_inventario/REGLAS-AGENTES.md`, `_inventario/README.md`, `_inventario/endpoints.csv`, `_inventario/rutas-spa.csv`, `_inventario/dialogos.txt`.
- `ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md` (versión 1.1): reglas `RN-VEN` y `RN-CMP`.
- `ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md`: reglas `RN-CAJ`, `RN-DEV` y `RN-DAS`.
- `ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md`: reglas `RN-TRA`, `RN-DEL`, `RN-MAG`, `RN-SRV`, `RN-CUA` y `RN-AUD`.
- `ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md` (`RN-REC-031`), `01a-administracion-y-seguridad.md` (reglas `RN-GEN`, `RN-IMP`, `RN-EMP`, `RN-SER` citadas) y `01b-catalogos.md` (`RN-PRE`, `RN-PAC`).
- `ola-1-fundamentos/19-inventario-de-pantallas/19f-punto-de-venta.md` y `19a-acceso-layout-y-compartidos.md` (apartado 3, dashboard).
- `ola-1-fundamentos/05-seguridad/05b-matriz-de-permisos.md` (apartados 1.3, 4 y 6.5).
- `ola-1-fundamentos/06-glosario.md` (términos: venta en espera, sesión de caja, efectivo esperado, diferencia de arqueo, cliente eventual, receta verificada).
- `ola-2-funcional/14-procesos-de-negocio.md` (procesos 4, 5, 6, 8 y 9).
- `ola-2-funcional/manual-usuario/capitulo-03-empleados-roles-cuentas.md` (permisos del rol «Cajero», sección 5.3) y `capitulo-05-productos-y-proveedores.md` (productos de ejemplo y condiciones de venta).
- `ola-2-funcional/12-historias-de-usuario/12a-acceso-y-administracion.md` (solo como modelo de formato).

Código consultado directamente (solo lectura):

- `sofia-web-spa/public/i18n/**/es.json` (contraste de textos: `pos`, `sesiones-caja`, `dashboard`, raíz y `recetas-medicas`).
- `sofia-web-spa/src/app/pages/pos/nueva-venta/nueva-venta.component.html` (líneas 9 a 24: tarjeta de aviso de caja) y `sofia-web-spa/src/app/pages/dashboard/dashboard.component.html` (formato de la variación contra ayer).
- `git log -1` y `git status --short` de `sofia-apis` y `sofia-web-spa`.

No se usaron el navegador, la API local, `README.md`/`CLAUDE.md` de los repositorios ni `swagger.json`.

