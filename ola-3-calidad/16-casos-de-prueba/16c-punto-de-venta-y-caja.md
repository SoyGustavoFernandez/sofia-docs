# Casos de prueba funcionales — Parte C: punto de venta, caja, devoluciones, dashboard y módulos solo API

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-30 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` (el de la línea base y de los documentos fuente; en esta tarea no se ejecutan comandos git, por lo que no se comprobó si hay commits más nuevos) |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` (ídem) |
| **Método** | Lectura de documentos validados y comprobación puntual en el código; casos no ejecutados |

## 1. Convenciones

Este documento reúne los casos de prueba funcionales de la parte C: terminal de venta y ventas (`TC-VEN`), caja (`TC-CAJ`), devoluciones (`TC-DEV`), dashboard (`TC-DAS`) y los módulos que solo existen en la API (`TC-OPS`). Se escribió desde las historias de usuario [12c](../../ola-2-funcional/12-historias-de-usuario/12c-punto-de-venta-y-caja.md), los casos de uso [13c](../../ola-2-funcional/13-casos-de-uso/13c-punto-de-venta-y-caja.md), las reglas [01c1](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md), [01c2](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [01c3](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) y [01d](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md), los inventarios de pantallas [19a](../../ola-1-fundamentos/19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md) y [19f](../../ola-1-fundamentos/19-inventario-de-pantallas/19f-punto-de-venta.md) y el [catálogo de errores](../../ola-1-fundamentos/21-catalogo-codigos-error.md). Ningún caso se ha ejecutado: el estado de todos es **No ejecutado**.

### 1.1 Cómo leer un caso

- **ID:** `TC-<MOD>-<nnn>`, con numeración propia por módulo (`VEN`, `CAJ`, `DEV`, `DAS`, `OPS`).
- **Tipo:** Positivo (el flujo funciona), Negativo (validación o regla que rechaza), Permisos (qué permiso falta y qué ocurre), Límite (valores en el borde), Concurrencia (dos operaciones a la vez) o Integración (servicios externos; en esta parte no hay casos de ese tipo).
- **Prioridad:** Alta, Media o Baja según el criterio de la sección 1.2.
- **Cubre:** el escenario de la historia (`HU-XXX-nnn` con el título del escenario entre « »), el caso de uso (`CU-XXX-nn` y el flujo: «básico», alternativo `An`, o excepción) y las reglas `RN-…`. Los casos de uso no numeran sus excepciones; se citan como «excepción, paso N» con la situación de su tabla de excepciones.
- **Esquema del escenario:** un caso con la tabla de ejemplos completa (datos y resultados); no hay un caso por fila.
- **Hallazgo:** cuando la fuente documenta un comportamiento raro (algo que el sistema no valida, un mensaje genérico), el resultado esperado es el comportamiento **actual** y el caso lo marca «Hallazgo: …» con la referencia del documento fuente. No se inventa el comportamiento deseado.
- **No verificado:** lo que ningún documento ni el código respalda se marca «No verificado: motivo».
- **Casos «Solo API»:** llevan la petición (método, ruta, cuerpo) y la respuesta esperada (HTTP y `code`). El token se escribe `<token>`. Los campos del cuerpo están en camelCase (por ejemplo `loteId`, `pagos[].montoPagado`, los de `CreateVentaCommand` en `sofia-apis/src/SOFIA.Application/Ventas/Commands/CreateVenta/CreateVentaCommand.cs:12-31`); el formato exacto con que se serializan los valores de los enums (texto o número) es **No verificado**, por lo que los cuerpos de ejemplo usan el nombre del enum.
- **Identificadores entre corchetes:** `[HU-…]`, `[CU-…]` y `[RN-…]` se escriben entre corchetes y se resuelven con la tabla siguiente (no cada uno lleva su propio enlace, para no repetir cientos de definiciones).

| Identificador | Documento |
|---|---|
| `HU-VEN`, `HU-CAJ`, `HU-DEV`, `HU-DAS`, `HU-OPS` | [12c — historias de usuario](../../ola-2-funcional/12-historias-de-usuario/12c-punto-de-venta-y-caja.md) |
| `CU-VEN`, `CU-CAJ`, `CU-DEV`, `CU-DAS`, `CU-OPS` | [13c — casos de uso](../../ola-2-funcional/13-casos-de-uso/13c-punto-de-venta-y-caja.md) |
| `RN-VEN`, `RN-CMP` (y `RN-GEN-…` citadas desde ventas) | [01c2 — ventas y comprobantes](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| `RN-CAJ`, `RN-DEV`, `RN-DAS` | [01c3 — caja, devoluciones y dashboard](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| `RN-TRA`, `RN-CUA`, `RN-MAG`, `RN-SRV`, `RN-DEL`, `RN-AUD` | [01d — módulos solo API](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) |
| `RN-REC`, `RN-LOT`, `RN-STK` | [01c1 — inventario y recetas](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) |
| `RN-GEN`, `RN-EMP`, `RN-SER`, `RN-IMP` | [01a — administración y seguridad](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |

- **Mensajes:** los textos entre « » son los de los `es.json` copiados a través de los documentos fuente. Donde el sistema no tiene mensaje propio se escribe el genérico que ve el usuario (en el cobro, «Ocurrió un error al procesar la venta.»).
- **Importes** en soles (S/). Los redondeos que dependen de la base de datos se marcan **No verificado** donde la fuente así lo dice.

### 1.2 Criterio de prioridad

| Prioridad | Qué entra |
|---|---|
| Alta | Dinero (cobro, vuelto, pago mixto, seguro, arqueo, reembolsos, notas de crédito), stock y lotes, comprobantes, medicamentos controlados y recetas, anulaciones y devoluciones, y todo lo que garantice el acceso y el aislamiento entre empresas, sucursales y cajeros (caja propia, permisos que protegen dinero). |
| Media | Resto del flujo principal: búsquedas, filtros, exportaciones, consultas, indicadores, avisos y flujos alternativos que no tocan dinero. En los módulos «Solo API», la operación normal de cada módulo. |
| Baja | Presentación, ordenación, textos informativos y mensajes de error de carga. |

## 2. Entorno y datos de prueba

### 2.1 Entorno

Para levantar el entorno local (SQL Server en Docker, scripts de DbUp, API y SPA) sigue la [guía del desarrollador](../../ola-1-fundamentos/08-guia-del-desarrollador.md) (sección 3) y la [guía de despliegue y operación](../../ola-1-fundamentos/10-despliegue-y-operacion.md); no se copian aquí. Para los casos de pantalla usa el SPA local; para los marcados «Solo API» usa un cliente HTTP (por ejemplo Postman o `curl`) contra la API local, con el token de un usuario que tenga los permisos que indica cada caso.

### 2.2 Empresa, sucursales y usuarios (ficticios)

Los datos siguen los del [manual de usuario, capítulo 3](../../ola-2-funcional/manual-usuario/capitulo-03-empleados-roles-cuentas.md). Todas las contraseñas son «una contraseña válida de 8 caracteres o más» que defines tú; nunca uses contraseñas reales.

| Dato | Valor |
|---|---|
| Empresa A | «Botica Manual Demo S.A.C.» con RUC registrado (el cobro lo exige, [RN-VEN-007]) y una serie de boleta activa `B001` y una de nota de crédito `BC01` en cada sucursal |
| Empresa B | «Botica Otra Demo S.A.C.» (solo para casos de aislamiento entre empresas) |
| Sucursal base | «Sede Principal» |
| Segunda sucursal | «Sucursal Demo Norte» |
| Admin | «Ana Demo Prueba», rol `Admin`, sucursal base «Sede Principal» |
| Cajero | «Carlos Cajero Demo» (usuario `carlos.cajero`), rol «Cajero» con los **19 permisos recomendados** del [manual, capítulo 3, sección 5.3](../../ola-2-funcional/manual-usuario/capitulo-03-empleados-roles-cuentas.md): `Ventas:Leer`, `Ventas:Crear`, `Ventas:Actualizar`, `Ventas:Anular`, `POS:Leer`, `POS:AperturarCaja`, `POS:CerrarCaja`, `Devoluciones:ProcesarDevolucion`, `Devoluciones:Leer`, `Inventario:Leer`, `DIGEMID:Leer`, `Medicamentos:Leer`, `Pacientes:Leer`, `Aseguradoras:Leer`, `Recetas:Leer`, `Recetas:Analizar`, `Recetas:Crear`, `ProfesionalesSalud:Leer` y `Empleados:Leer` |
| Segundo cajero | «Nora Norte Demo», mismo rol, sucursal base «Sucursal Demo Norte» |
| Cajero reducido | Un rol «Cajero sin dinero» con los mismos permisos menos `Ventas:Anular`, `Devoluciones:ProcesarDevolucion` y `POS:CerrarCaja` (para los casos de permisos); se crea con el administrador de roles |

Cada caso dice qué usuario lo ejecuta. Cuando un caso pide quitar un permiso, se hace sobre el rol «Cajero sin dinero» o una copia, nunca sobre el rol «Cajero» de las demás pruebas.

### 2.3 Productos, lotes, clientes y recetas (ficticios)

| Dato | Valor |
|---|---|
| `DEMO-MED-0001` | «Paracetamol Demo 500 mg», Venta Libre, S/ 0.50 por unidad; presentación «Caja x10» a S/ 4.50 (10 unidades base) |
| `DEMO-MED-0002` | «Amoxicilina Demo 500 mg», Receta Simple, S/ 1.20 |
| `DEMO-MED-0003` | «Clonazepam Demo 2 mg», Receta Retenida, S/ 2.50 |
| `DEMO-MED-0004` | «Tramadol Demo 50 mg», Estupefaciente, S/ 3.80 |
| Lotes de «Sede Principal» | `LOT-QA-P01` (Paracetamol, stock 500, vence en más de un año); `LOT-QA-P02` (Paracetamol, stock 30, vence antes que `LOT-QA-P01`); `LOT-QA-P03` (Paracetamol, vencido); `LOT-QA-P04` (Paracetamol, stock 40, aislado en cuarentena con `POST /api/v1/DIGEMID/cuarentena`); `LOT-QA-A01` (Amoxicilina, stock 200); `LOT-QA-C01` (Clonazepam, stock 100); `LOT-QA-T01` (Tramadol, stock 50) |
| Lote de otra sucursal | `LOT-QA-N01` (Paracetamol, stock 100) con fila de stock solo en «Sucursal Demo Norte» |
| Cliente | «Rosa Quispe Demo», DNI `45781236`, registrada como paciente |
| Recetas de Rosa | `RX-QA-001` (Receta Simple, con Amoxicilina, 1 dispensación máxima), `RX-QA-002` (Receta Retenida, con Clonazepam, número `RET-0001`, 2 dispensaciones máximas), `RX-QA-003` (Receta Retenida, con Clonazepam, **sin número**) |
| Aseguradora | «Aseguradora Demo» |
| Empresa flexible | La empresa A tiene el indicador `ExigirRegistroReceta` en verdadero (modo estricto); los casos que piden «empresa flexible» lo cambian a falso por base de datos en una empresa de prueba aparte (no se modifica la empresa A) |

Los números de lote, de receta y de operación de los casos (`OP-QA-0001`, etc.) son inventados. Si tu base no tiene estos datos, créalos con los maestros de la parte B (productos, lotes, pacientes, recetas) antes de empezar; los pasos exactos están en esos documentos.

## 3. Suite de humo

Son 28 casos de prioridad Alta; córrelos en este orden, porque cada bloque deja los datos del siguiente (caja abierta, venta cobrada, lote con stock conocido). Si uno falla, los de su bloque que dependen de él no son concluyentes.

| Orden | Caso | Qué comprueba |
|---|---|---|
| 1 | TC-CAJ-001 | Abrir la caja de Carlos con S/ 100.00 |
| 2 | TC-CAJ-005 | No se puede abrir una segunda caja |
| 3 | TC-VEN-002 | Otro usuario sin caja solo ve la tarjeta de aviso |
| 4 | TC-VEN-005 | La caja de otro cajero no habilita el terminal |
| 5 | TC-VEN-001 | Entrar al terminal con la caja propia abierta |
| 6 | TC-VEN-022 | Agregar 100 unidades al carrito (totales S/ 42.37, S/ 7.63, S/ 50.00) |
| 7 | TC-VEN-035 | Vender dos cajas (stock en unidades base) |
| 8 | TC-VEN-088 | Cobro en efectivo con vuelto |
| 9 | TC-VEN-104 | Pago dividido Yape / Plin y efectivo |
| 10 | TC-VEN-112 | Procesar la venta: boleta, stock y diálogo de éxito |
| 11 | TC-VEN-128 | Primera boleta `B001-00000001` |
| 12 | TC-VEN-054 | Venta de receta retenida con receta registrada y número |
| 13 | TC-VEN-073 | Segunda venta con la misma receta retenida: rechazo |
| 14 | TC-OPS-049 | Lote retenido en cuarentena no se vende |
| 15 | TC-DEV-001 | Devolución parcial |
| 16 | TC-DEV-017 | Reingreso a venta: el stock vuelve al lote |
| 17 | TC-DEV-023 | Reembolso en efectivo baja el esperado de la caja |
| 18 | TC-DEV-033 | Nota de crédito `BC01-00000001` |
| 19 | TC-DEV-047 | Mensajes de rechazo de una devolución |
| 20 | TC-VEN-183 | Una venta con devoluciones no se anula |
| 21 | TC-VEN-172 | Anular una venta cobrada |
| 22 | TC-VEN-175 | Anular exige caja propia abierta |
| 23 | TC-DAS-017 | El Inicio muestra el efectivo esperado de la caja |
| 24 | TC-CAJ-029 | Ejemplo completo del arqueo: apertura + efectivo − vuelto − reembolsos |
| 25 | TC-CAJ-016 | Cierre con monto exacto: «Cuadrada» |
| 26 | TC-CAJ-017 | Resultado del cierre: faltante, sobrante y centavo |
| 27 | TC-CAJ-037 | Efectivo esperado negativo impide cerrar |
| 28 | TC-OPS-013 | Despachar una transferencia descuenta el stock del origen |

## 4. Casos del módulo Ventas (`TC-VEN`)

### HU-VEN-001 — Entrar al terminal de venta con la caja propia abierta

#### TC-VEN-001 — Entrar al terminal con la caja propia abierta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-001] «Entrar al terminal con mi caja abierta»; [CU-VEN-01] flujo básico; [RN-VEN-001], [RN-VEN-002], [RN-VEN-077] |

**Precondiciones:** «Carlos Cajero Demo» con su caja en estado «Abierta» en «Sede Principal».
**Pasos:**
1. Inicia sesión con `carlos.cajero`.
2. Ve a `/pos` y haz clic en «Nueva Venta POS» (llegas a `/pos/nueva`).
**Resultado esperado:** primero un indicador de carga; luego el terminal con la búsqueda de productos, el carrito, «Datos del Cliente» y «Método de Pago». El terminal no muestra la hora de apertura, el monto inicial ni el efectivo esperado de la caja.

#### TC-VEN-002 — Sin caja abierta solo se ve la tarjeta de aviso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-001] «No tengo caja abierta»; [CU-VEN-01] alternativo A1; [RN-VEN-003] |

**Precondiciones:** «Carlos Cajero Demo» sin ninguna caja abierta.
**Pasos:** 1. Inicia sesión con `carlos.cajero`. 2. Abre `/pos/nueva`.
**Resultado esperado:** solo una tarjeta con «Debes aperturar la caja del turno antes de procesar una venta.»; no se ve el catálogo, el carrito ni los pagos; no hay forma de vender ni de dejar una venta en espera.

#### TC-VEN-003 — Abrir la caja desde la tarjeta de aviso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-001] «Abrir la caja desde la tarjeta de aviso»; [CU-VEN-01] alternativo A1; [RN-VEN-003] |

**Precondiciones:** «Carlos Cajero Demo» sin caja abierta y con `POS:AperturarCaja`.
**Pasos:** 1. Abre `/pos/nueva`. 2. Haz clic en «Aperturar Caja». 3. En el diálogo «Aperturar Caja» completa los datos con un monto inicial de S/ 100.00 y confirma (el detalle del diálogo está en `TC-CAJ-011`).
**Resultado esperado:** el diálogo se cierra, el terminal vuelve a buscar la caja y se habilita (aparecen la búsqueda de productos, el carrito y los pagos).

#### TC-VEN-004 — Sin permiso para abrir caja la tarjeta no ofrece el botón

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-VEN-001] «Sin permiso para abrir caja»; [CU-VEN-01] alternativo A1; [RN-VEN-001] |

**Precondiciones:** usuario del rol «Cajero sin dinero» sin `POS:AperturarCaja` y sin caja abierta.
**Pasos:** abre `/pos/nueva`.
**Resultado esperado:** la tarjeta solo dice «Debes aperturar la caja del turno antes de procesar una venta.» y no muestra el botón «Aperturar Caja».

#### TC-VEN-005 — La caja abierta de otro cajero no habilita el terminal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-001] «Una caja abierta por otro cajero no sirve»; [CU-VEN-01] alternativo A2; [RN-VEN-004] |

**Precondiciones:** «Ana Demo Prueba» (u otro empleado) tiene una caja «Abierta» en «Sede Principal»; «Carlos Cajero Demo» no tiene caja abierta.
**Pasos:** inicia sesión con `carlos.cajero` y abre `/pos/nueva`.
**Resultado esperado:** el aviso «Debes aperturar la caja del turno antes de procesar una venta.»; el terminal busca la sesión del empleado y la sucursal de la sesión de usuario, así que la caja ajena no sirve.

#### TC-VEN-006 — La API rechaza cobrar con una caja ajena, cerrada o inexistente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-001] «La API rechaza una caja ajena o cerrada»; [CU-VEN-01] excepción, paso «Venta» (dos filas); [RN-VEN-003], [RN-VEN-004], [RN-VEN-005] |

**Precondiciones:** `carlos.cajero` con caja abierta `S1`; «Nora Norte Demo» con caja abierta `S2` en «Sucursal Demo Norte»; una caja `S3` de «Sede Principal» ya cerrada; `LOT-QA-P01` con stock.
**Datos:** token de `carlos.cajero`.
**Pasos:**
1. `POST /api/v1/ventas` con `{"sesionId":"<S2>","estado":"Completada","detalles":[{"loteId":"<LOT-QA-P01>","cantidad":1}],"pagos":[{"metodoPago":"Efectivo","montoPagado":0.50}]}` (caja de otro cajero y otra sucursal).
2. Repite con `"sesionId":"<S3>"` (caja cerrada).
3. Repite con un `sesionId` que no existe.
**Resultado esperado:** paso 1: HTTP 403, `code` `Venta.Caja.NoPropia`; en el SPA el mismo rechazo se ve como «La caja seleccionada pertenece a otro cajero o sucursal. Recarga la página para usar tu propia caja abierta.». Pasos 2 y 3: HTTP 400, `code` `Venta.Caja`; en el SPA el genérico «Ocurrió un error al procesar la venta.». En ningún caso se crea la venta ni baja el stock de `LOT-QA-P01`.
**Hallazgo:** `Venta.Caja` no tiene traducción (HU-VEN-001, nota «Brecha conocida»).

#### TC-VEN-007 — La ruta del terminal exige Ventas:Leer y Ventas:Crear

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-VEN-001] «La ruta exige permisos de ventas»; [CU-VEN-01] excepción, paso 1; [RN-VEN-001] |

**Precondiciones:** usuario con `Ventas:Leer` pero sin `Ventas:Crear`.
**Pasos:** escribe `/pos/nueva` en la barra de direcciones.
**Resultado esperado:** el sistema te lleva a `/dashboard` con el aviso «No tienes permisos para acceder a esta pantalla.».

#### TC-VEN-008 — Sucursal y empleado salen de la sesión de usuario

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-001] «La sucursal y el empleado salen de mi sesión de usuario»; [CU-VEN-01] excepción, paso 3 (`Auth.Sucursal`); [RN-VEN-002], [RN-GEN-024] |

**Precondiciones:** una cuenta cuyo token no traiga sucursal. **No verificado:** cómo preparar esa cuenta desde la interfaz (los documentos no lo describen; consulta [05a](../../ola-1-fundamentos/05-seguridad/05a-autenticacion-y-controles.md)).
**Pasos:** 1. Con esa cuenta abre el terminal y busca lotes. 2. Con `carlos.cajero` registra una venta y revisa en `/pos/{id}` el empleado y la sucursal de la venta.
**Resultado esperado:** paso 1: «No se pudo determinar el contexto de sucursal asignada del usuario.». Paso 2: la venta queda a nombre de «Carlos Cajero Demo» y en «Sede Principal»; el terminal no ofrece elegirlos.

#### TC-VEN-009 — Si la configuración de la empresa no carga el terminal es estricto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-001] «Si la configuración de la empresa no carga, el terminal es estricto»; [CU-VEN-01] alternativo A4; [RN-VEN-077], [RN-VEN-028] |

**Precondiciones:** empresa flexible (`ExigirRegistroReceta` en falso); caja abierta de `carlos.cajero`.
**Pasos:** 1. En las herramientas del navegador bloquea la petición `GET /api/v1/ventas/configuracion` (o simula un error 500). 2. Abre `/pos/nueva`. 3. Agrega `LOT-QA-A01` (Amoxicilina, Receta Simple) al carrito sin elegir receta.
**Resultado esperado:** no aparece ningún mensaje de error por la configuración; el terminal se comporta en modo estricto: exige receta registrada para la Receta Simple (aviso «Hay productos que requieren…», ver `TC-VEN-057`).

### HU-VEN-002 — Buscar productos y elegir el lote

#### TC-VEN-010 — Ver los productos más vendidos al entrar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-002] «Ver los productos más vendidos al entrar»; [CU-VEN-02] flujo básico; [RN-VEN-075] |

**Precondiciones:** caja abierta; ventas de los últimos 30 días de al menos dos productos.
**Pasos:** abre `/pos/nueva` sin escribir ni elegir chips.
**Resultado esperado:** hasta 12 tarjetas, con los más vendidos de los últimos 30 días primero; cada tarjeta muestra el nombre comercial, «{laboratorio} · {código nacional}», el precio «S/ 0.50 c/u» y una insignia de stock.

#### TC-VEN-011 — Buscar por nombre comercial

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-002] «Buscar por nombre comercial»; [CU-VEN-02] flujo básico; [RN-VEN-075] |

**Pasos:** escribe «paracetamol» en «Buscar producto...» y deja de teclear.
**Resultado esperado:** unos 300 milisegundos después la lista se filtra por nombre comercial y solo muestra productos con stock vendible en «Sede Principal» (el producto con stock solo en «Sucursal Demo Norte» no aparece).

#### TC-VEN-012 — Filtrar por condición de venta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-002] «Filtrar por condición de venta»; [CU-VEN-02] flujo básico |

**Pasos:** 1. Mira los chips. 2. Haz clic en «Receta Retenida».
**Resultado esperado:** los chips son «Todos», «Venta Libre (OTC)», «Receta Simple», «Receta Retenida» y «Estupefaciente»; al elegir «Receta Retenida» la búsqueda se repite con ese filtro y el texto actual y muestra «Clonazepam Demo 2 mg».

#### TC-VEN-013 — Insignia de stock de la tarjeta (esquema)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-VEN-002] «Insignia de stock de la tarjeta»; [CU-VEN-02] flujo básico, paso 3 |

**Precondiciones:** cuatro productos de prueba cuyo stock vendible total en «Sede Principal» sea 0, 6, 10 y 11.
**Pasos:** busca cada producto en el terminal y lee su tarjeta.
**Resultado esperado:**

| Stock vendible | Insignia |
|---|---|
| 0 | Agotado |
| 6 | Stock bajo |
| 10 | Stock bajo |
| 11 | Disponible |

#### TC-VEN-014 — Los productos de receta lo indican

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-002] «Los productos de receta lo indican»; [CU-VEN-02] flujo básico, paso 3 |

**Pasos:** busca «Clonazepam».
**Resultado esperado:** la tarjeta de «Clonazepam Demo 2 mg» muestra el aviso «Requiere receta».

#### TC-VEN-015 — Producto sin precio en el catálogo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-VEN-002] «Producto sin precio en el catálogo»; [CU-VEN-02] flujo básico, paso 3; [RN-VEN-019] |

**Precondiciones:** un producto con stock y sin precio de venta base ni presentaciones.
**Pasos:** busca ese producto.
**Resultado esperado:** su tarjeta muestra «Sin precio» en lugar de «S/ … c/u».

#### TC-VEN-016 — Búsqueda sin resultados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-VEN-002] «Sin resultados»; [CU-VEN-02] alternativo A1 |

**Pasos:** escribe «zzzxxx».
**Resultado esperado:** «No se encontraron medicamentos o lotes disponibles.».

#### TC-VEN-017 — Elegir un producto y ver sus lotes ordenados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-002] «Elegir un producto y ver sus lotes»; [CU-VEN-02] flujo básico, pasos 4 a 6; [RN-VEN-075] |

**Precondiciones:** «Paracetamol Demo 500 mg» con `LOT-QA-P01` (500) y `LOT-QA-P02` (30, vence antes).
**Pasos:** haz clic en la tarjeta «Paracetamol Demo 500 mg».
**Resultado esperado:** se ve «Lote:» y una fila por lote con el número y «Stock Disp: 500» / «Stock Disp: 30». Orden: vencimiento más próximo primero (`LOT-QA-P02`, luego `LOT-QA-P01`) y, a igual vencimiento, por número de lote. Como máximo 50 lotes. La fila no muestra la fecha de vencimiento.
**Hallazgo:** el terminal no muestra la fecha de vencimiento (HU-VEN-002, «Brecha conocida»).

#### TC-VEN-018 — Solo se ofrecen lotes vendibles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-002] «Solo se ofrecen lotes vendibles»; [CU-VEN-02] flujo básico, paso 5; [RN-VEN-012], [RN-VEN-013] |

**Precondiciones:** `LOT-QA-P03` vencido, `LOT-QA-P04` en cuarentena, un lote de Paracetamol con cantidad 0 y un lote que vence hoy.
**Pasos:** elige la tarjeta «Paracetamol Demo 500 mg».
**Resultado esperado:** no aparecen el vencido, el de cuarentena ni el de cantidad 0; el lote que caduca hoy sí aparece (vencido es fecha de caducidad menor o igual que ayer a esta hora, así que se vende hasta el final del día).

#### TC-VEN-019 — El sistema no obliga a vender primero el lote que vence antes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-002] «El sistema no obliga a vender primero el lote que vence antes»; [CU-VEN-02] flujo básico, paso 7 |

**Pasos:** elige `LOT-QA-P01` (vence después) y agrega 1 unidad.
**Resultado esperado:** el sistema lo permite; la línea queda en el carrito sin aviso.

#### TC-VEN-020 — Fallan la carga de lotes y la búsqueda de productos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-VEN-002] «Falla la carga de lotes», «Falla la búsqueda de productos»; [CU-VEN-02] excepción, pasos 1-2 y 4 |

**Pasos:** 1. Bloquea `GET /api/v1/ventas/lotes-disponibles` en las herramientas del navegador y elige una tarjeta. 2. Desbloquea; bloquea `GET /api/v1/medicamentos` y recarga o escribe en la búsqueda; desbloquea y escribe otro texto.
**Resultado esperado:** paso 1: la lista de lotes queda vacía y se ve «No se pudieron cargar los lotes disponibles. Inténtalo nuevamente.». Paso 2: «No se pudo cargar el catálogo de medicamentos. Inténtalo nuevamente.»; con el siguiente texto la búsqueda vuelve a funcionar.

#### TC-VEN-021 — Un lote con stock solo en otra sucursal no se puede vender

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-002] «Un lote que solo tiene stock en otra sucursal no se vende»; [CU-VEN-02] excepción, «Cobro posterior» (`Venta.Lote`); [RN-VEN-011] |

**Precondiciones:** `LOT-QA-N01` con stock solo en «Sucursal Demo Norte»; `carlos.cajero` con caja `S1` en «Sede Principal».
**Pasos:** `POST /api/v1/ventas` con token de `carlos.cajero`: `{"sesionId":"<S1>","estado":"Completada","detalles":[{"loteId":"<LOT-QA-N01>","cantidad":1}],"pagos":[{"metodoPago":"Efectivo","montoPagado":0.50}]}`.
**Resultado esperado:** HTTP 400, `code` `Venta.Lote`; en el SPA se vería «Ocurrió un error al procesar la venta.». No se crea la venta y el stock de Norte no cambia.
**Hallazgo:** mensaje genérico (01c2 §5.3 hallazgo 3).

### HU-VEN-003 — Agregar productos al carrito

#### TC-VEN-022 — Agregar 100 unidades de un producto de venta libre

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-003] «Agregar 100 unidades de un producto de venta libre»; [CU-VEN-03] flujo básico; [RN-VEN-019], [RN-VEN-021] |

**Precondiciones:** caja abierta; `LOT-QA-P01` (stock 500) a S/ 0.50.
**Pasos:** elige «Paracetamol Demo 500 mg» y el lote `LOT-QA-P01`; escribe Cantidad 100 y haz clic en «Agregar».
**Resultado esperado:** una línea con «S/ 0.50 c/u» y total de línea «S/ 50.00»; el encabezado dice «Resumen de Venta · 1»; los totales muestran «Subtotal (Op. Gravada):» S/ 42.37, «IGV (18%):» S/ 7.63 y «TOTAL A PAGAR:» S/ 50.00; se limpia la selección de lote y producto.

#### TC-VEN-023 — El precio no se puede cambiar ni enviar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-003] «El precio no se puede cambiar»; [CU-VEN-03] flujo básico, pasos 1 y 7; [RN-VEN-019] |

**Pasos:** 1. Elige un lote y trata de escribir en «Precio Unitario (S/)». 2. Revisa en las herramientas del navegador el cuerpo del `POST /api/v1/ventas` al cobrar (`TC-VEN-112`).
**Resultado esperado:** el campo es de solo lectura; el cuerpo de la petición no incluye precio ni costo (`detalles[]` solo tiene `loteId`, `cantidad`, `recetaId` y `presentacionVentaId`); el servidor resuelve el precio del catálogo.

#### TC-VEN-024 — La cantidad supera el stock del lote

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-VEN-003] «La cantidad supera el stock del lote»; [CU-VEN-03] excepción, paso 3; [RN-VEN-014], [RN-VEN-024] |

**Precondiciones:** `LOT-QA-P02` con 30 unidades.
**Pasos:** 1. Escribe Cantidad 30 y agrega. 2. Vacía el carrito. 3. Escribe Cantidad 31 y agrega.
**Resultado esperado:** paso 1: la línea se agrega con 30 unidades. Paso 3: «Stock máximo disponible alcanzado (30 un.).» y no se agrega la línea.

#### TC-VEN-025 — Producto sin precio configurado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-003] «Producto sin precio configurado»; [CU-VEN-03] excepción, pasos 3 y 7 (`Venta.SinPrecio`); [RN-VEN-019] |

**Precondiciones:** un lote de un producto sin precio de venta en el catálogo ni presentaciones.
**Pasos:** 1. En el terminal elige ese lote y haz clic en «Agregar». 2. Por API: `POST /api/v1/ventas` con ese `loteId` y `cantidad` 1.
**Resultado esperado:** paso 1: «Este producto no tiene precio de venta configurado en el catálogo. Pide a un administrador que lo registre.» y no se agrega la línea. Paso 2: HTTP 400, `code` `Venta.SinPrecio`, con el mismo texto en el SPA.

#### TC-VEN-026 — Cantidad inválida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-003] «Cantidad inválida»; [CU-VEN-03] excepción, paso 3; [RN-VEN-024] |

**Pasos:** deja Cantidad vacía (o escribe 0) y haz clic en «Agregar».
**Resultado esperado:** «Ingresa una cantidad válida para agregar el producto.»; no se agrega la línea.

#### TC-VEN-027 — Orden de las validaciones al agregar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-VEN-003] «Orden de las validaciones al agregar»; [CU-VEN-03] flujo básico, paso 3; [RN-VEN-024] |

**Precondiciones:** lote de un producto con precio 0 y stock 30.
**Pasos:** escribe Cantidad 31 y haz clic en «Agregar»; corrige a 30 y agrega; corrige a 0 y agrega.
**Resultado esperado:** primero «Stock máximo disponible alcanzado (30 un.).»; luego el aviso de precio («Este producto no tiene precio de venta configurado en el catálogo. Pide a un administrador que lo registre.»); por último «Ingresa una cantidad válida para agregar el producto.».

#### TC-VEN-028 — Repetir el mismo lote suma a la línea existente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-VEN-003] «Repetir el mismo lote suma a la línea existente»; [CU-VEN-03] flujo básico, paso 4; [RN-VEN-014], [RN-VEN-024] |

**Precondiciones:** `LOT-QA-P01` con stock 500.
**Pasos:** 1. Agrega 100 unidades sin presentación. 2. Agrega 50 más del mismo lote. 3. Agrega 400 más.
**Resultado esperado:** paso 2: una sola línea con 150 unidades. Paso 3: «Stock máximo disponible alcanzado (500 un.).» y la cantidad queda en 500 (la existencia).

#### TC-VEN-029 — El mismo producto con otro lote es otra línea

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-003] «El mismo producto con otro lote es otra línea»; [CU-VEN-03] alternativo A1 |

**Pasos:** agrega 1 unidad de `LOT-QA-P01` y 1 de `LOT-QA-P02`.
**Resultado esperado:** el carrito muestra dos líneas («Resumen de Venta · 2»).

#### TC-VEN-030 — Cantidades con decimales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-VEN-003] «Cantidades con decimales»; [CU-VEN-03] alternativo A2; [RN-VEN-010], [RN-VEN-020] |

**Pasos:** escribe Cantidad 0.5 y haz clic en «Agregar».
**Resultado esperado:** el terminal la acepta (solo exige que sea mayor que 0). **No verificado:** cómo redondea la base de datos las cantidades con más de 2 decimales al cobrar; no cobres la venta para comprobarlo sin acordar el resultado esperado.

#### TC-VEN-031 — El servidor vuelve a validar el stock al cobrar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-VEN-003] «El servidor vuelve a validar al cobrar»; [CU-VEN-03] excepción, paso 7 (`Venta.Stock`); [RN-VEN-014] |

**Precondiciones:** `LOT-QA-P02` con 30 unidades; dos cajeros (`carlos.cajero` y otro de «Sede Principal») con caja abierta.
**Pasos:** 1. Carlos agrega 30 unidades de `LOT-QA-P02` al carrito sin cobrar. 2. El otro cajero vende las 30 unidades del mismo lote y cobra. 3. Carlos intenta cobrar.
**Resultado esperado:** HTTP 400, `code` `Venta.Stock`; Carlos ve «Ocurrió un error al procesar la venta.» y su carrito se conserva. El stock de `LOT-QA-P02` es 0, no negativo.

#### TC-VEN-032 — Un lote que venció antes de cobrar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-003] «Un lote que venció antes de cobrar»; [CU-VEN-03] excepción, paso 7 (`Venta.Lote.Vencido`); [RN-VEN-013] |

**Precondiciones:** un lote de prueba que vence hoy; carrito con ese lote.
**Pasos:** cambia por base de datos la caducidad del lote a una fecha pasada (ambiente de prueba) y haz clic en «Procesar Venta», confirma.
**Resultado esperado:** «Uno de los lotes está vencido y no se puede vender. Retíralo del carrito.»; no se registra la venta ni baja el stock.

#### TC-VEN-033 — Qué guarda el servidor de cada línea

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-003] «Qué guarda el servidor de cada línea»; [CU-VEN-03] flujo básico, paso 7; [RN-VEN-008], [RN-VEN-009], [RN-VEN-010], [RN-VEN-011], [RN-VEN-020], [RN-VEN-022], [RN-VEN-080] |

**Precondiciones:** `LOT-QA-P01` con precio de proveedor vigente; caja abierta.
**Pasos:** 1. Vende 100 unidades de `LOT-QA-P01` en efectivo (S/ 50.00 exactos). 2. Abre el detalle de la venta en `/pos/{id}`. 3. Comprueba el stock del lote en «Stock por Sucursal» o «Lotes».
**Resultado esperado:** la línea guarda cantidad 100 (unidades base), precio S/ 0.50 y el total de la venta es S/ 50.00 (suma exacta de precio por cantidad, sin descuentos); el stock del lote baja de 500 a 400. El costo histórico no se muestra en pantalla. Si el producto no tiene precio de proveedor, el costo guardado es 0 (**No verificado** en pantalla: el costo no se ve).

#### TC-VEN-034 — No existen descuentos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-003] «No existen descuentos»; [CU-VEN-03] flujo básico, paso 5; [RN-VEN-023] |

**Pasos:** arma una venta y revisa todo el terminal.
**Resultado esperado:** no hay ningún campo de descuento; el único ajuste al monto a cobrar es la cobertura de seguro (`TC-VEN-141`).

### HU-VEN-004 — Vender por presentación (caja, blíster)

#### TC-VEN-035 — Vender dos cajas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-004] «Vender dos cajas»; [CU-VEN-04] flujo básico; [RN-VEN-017], [RN-VEN-019], [RN-VEN-076] |

**Precondiciones:** presentación «Caja x10» a S/ 4.50; `LOT-QA-P01` con 500 unidades; caja abierta.
**Pasos:** 1. Elige el lote; en «Presentación» elige «Caja x10 (S/ 4.50)». 2. Escribe Cantidad 2 y haz clic en «Agregar». 3. Cobra en efectivo S/ 9.00. 4. Revisa el stock del lote.
**Resultado esperado:** la línea muestra «2 × Caja x10» y «S/ 0.45 c/u»; total de línea «S/ 9.00». Al cobrar el servidor descuenta 20 unidades base (el stock del lote pasa de 500 a 480) y guarda ambas cantidades: 20 vendidas y 2 en presentación.

#### TC-VEN-036 — Cambiar entre presentación y unidad

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-004] «Cambiar entre presentación y unidad»; [CU-VEN-04] alternativo A1 |

**Pasos:** elige «Caja x10», escribe Cantidad 3, luego elige «Unidad».
**Resultado esperado:** la cantidad vuelve a 1 y el precio vuelve a S/ 0.50.

#### TC-VEN-037 — El máximo se calcula en presentaciones enteras

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-VEN-004] «El máximo se calcula en presentaciones enteras»; [CU-VEN-04] flujo básico, paso 3 |

**Precondiciones:** un lote de Paracetamol con 35 unidades.
**Pasos:** elige la presentación «Caja x10» y escribe Cantidad 4.
**Resultado esperado:** el campo Cantidad no admite más de 3 cajas.

#### TC-VEN-038 — Los botones del carrito avanzan de a una presentación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-004] «Los botones del carrito avanzan de a una presentación»; [CU-VEN-04] alternativo A3 |

**Pasos:** con la línea «2 × Caja x10» en el carrito, haz clic en «+».
**Resultado esperado:** la cantidad en unidades base sube de 20 a 30 y la de presentaciones de 2 a 3.

#### TC-VEN-039 — Presentación con precio aunque el medicamento no tenga precio base

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-004] «Presentación con precio aunque el medicamento no tenga precio base»; [CU-VEN-04] alternativo A4; [RN-VEN-019] |

**Precondiciones:** un producto sin precio base y con una presentación con precio (por ejemplo, «Caja x10» a S/ 4.50).
**Pasos:** vende 1 presentación de ese producto.
**Resultado esperado:** el precio unitario es el de la presentación dividido entre sus unidades base (S/ 0.45) y el servidor no rechaza la línea por falta de precio.

#### TC-VEN-040 — La presentación debe ser del mismo producto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-004] «La presentación debe ser del mismo producto»; [CU-VEN-04] excepción, paso 5 (`Venta.Presentacion`); [RN-VEN-018] |

**Pasos:** `POST /api/v1/ventas` (token de `carlos.cajero`, caja `S1`) con `detalles:[{"loteId":"<LOT-QA-A01>","cantidad":1,"presentacionVentaId":"<id de «Caja x10» de Paracetamol>"}]` y un pago en efectivo suficiente.
**Resultado esperado:** HTTP 400, `code` `Venta.Presentacion`; en el SPA «Ocurrió un error al procesar la venta.». No se crea la venta. Repite con un `presentacionVentaId` inexistente: mismo resultado.

#### TC-VEN-041 — Producto sin presentaciones y fallo al cargarlas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-VEN-004] «Producto sin presentaciones», «Falla la carga de presentaciones»; [CU-VEN-04] alternativo A2 y excepción, paso 1 |

**Pasos:** 1. Elige un lote de un producto sin presentaciones. 2. Bloquea `GET /api/v1/ventas/presentaciones` y elige un lote de Paracetamol.
**Resultado esperado:** paso 1: no aparece el selector «Presentación»; solo se vende por unidad. Paso 2: «No se pudieron cargar las presentaciones del producto. Inténtalo nuevamente.» y solo se puede vender por unidad.

### HU-VEN-005 — Ajustar, quitar y vaciar el carrito

#### TC-VEN-042 — Carrito vacío

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-005] «Carrito vacío»; [CU-VEN-05] flujo básico, paso 1; [RN-VEN-008] |

**Pasos:** abre el terminal sin agregar nada.
**Resultado esperado:** «El carrito está vacío.»; no se ven los totales; «Procesar Venta» y «Guardar y Atender Siguiente» están deshabilitados.

#### TC-VEN-043 — Subir, bajar y los límites de la cantidad en el carrito

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-VEN-005] «Subir y bajar la cantidad», «No se baja del mínimo», «No se sube por encima del stock»; [CU-VEN-05] flujo básico, paso 2 y excepciones; [RN-VEN-014], [RN-VEN-024] |

**Precondiciones:** línea de 100 unidades de `LOT-QA-P01` (stock 500).
**Pasos:** 1. «+» una vez; «−» una vez. 2. Deja la línea en 1 unidad y haz clic en «−». 3. Sube la línea hasta 500 unidades y haz clic en «+».
**Resultado esperado:** paso 1: 101 y luego 100, con el total de la línea y los totales recalculados. Paso 2: no pasa nada y no hay aviso. Paso 3: «Stock máximo disponible alcanzado (500 un.).» y la cantidad queda en 500.

#### TC-VEN-044 — Quitar una línea y vaciar el carrito

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-005] «Quitar una línea», «Vaciar el carrito», «La cantidad no se escribe en el carrito»; [CU-VEN-05] flujo básico, pasos 3 y 4, alternativo A1; [RN-VEN-045] |

**Precondiciones:** carrito con dos líneas, cliente «Rosa Quispe Demo» elegido, un monto de pago escrito en «Tarjeta».
**Pasos:** 1. Haz clic en la «×» de una línea. 2. Haz clic en «Vaciar». 3. Busca en el carrito un campo para escribir la cantidad.
**Resultado esperado:** la línea desaparece sin pedir confirmación; «Vaciar» borra las líneas sin pedir confirmación; los pagos vuelven a un único pago en «Efectivo» por 0; el cliente, la receta y la cobertura elegidos se conservan; el carrito no tiene campo para escribir la cantidad (solo «+» y «−»).

### HU-VEN-006 — Recuperar la venta en curso tras recargar (borrador local)

#### TC-VEN-045 — Recargar la página con una venta a medias

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-006] «Recargar la página con una venta a medias», «Qué se guarda y dónde»; [CU-VEN-06] flujo básico; [RN-VEN-045], [RN-VEN-053] |

**Precondiciones:** 2 líneas en el carrito, cliente «Rosa Quispe Demo», un pago por Yape / Plin con número de operación `OP-QA-0001`.
**Pasos:** 1. Recarga el navegador. 2. Revisa en las herramientas del navegador el almacenamiento local (`localStorage`).
**Resultado esperado:** se restauran carrito, pagos, cliente, receta elegida y monto de aseguradora; la casilla «Receta verificada» no se restaura. El borrador está en el navegador con una clave por usuario y guarda solo identificadores del cliente y de la receta, sin datos personales.

#### TC-VEN-046 — El borrador caduca y se borra

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-VEN-006] «El borrador caduca», «Se borra al cobrar con éxito», «Se borra al cerrar sesión o al entrar otro usuario»; [CU-VEN-06] alternativo A1 y flujo básico, paso 4 |

**Pasos:** 1. Con un borrador guardado, cambia en `localStorage` la marca de hora a más de 12 horas atrás y recarga. 2. Arma otra venta, cóbrala y recarga. 3. Arma una venta, cierra sesión e inicia sesión con otro usuario del mismo navegador.
**Resultado esperado:** paso 1: el terminal empieza vacío. Paso 2: tras cobrar con éxito no hay borrador. Paso 3: el borrador del usuario anterior se elimina. **No verificado:** el nombre de la clave y de la marca de hora en `localStorage` (consulta [19f](../../ola-1-fundamentos/19-inventario-de-pantallas/19f-punto-de-venta.md) §3.9).

#### TC-VEN-047 — Al retomar una venta en espera solo sobreviven los pagos no efectivo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-006] «Al retomar una venta en espera solo sobreviven los pagos no efectivo»; [CU-VEN-06] alternativo A2; [RN-VEN-053] |

**Precondiciones:** una venta en espera retomada en `/pos/nueva?continuar={id}`; pago con tarjeta con monto y `OP-QA-0002`, y un monto en efectivo escrito.
**Pasos:** recarga el navegador.
**Resultado esperado:** se recupera el pago con tarjeta con su monto y número de operación; el monto en efectivo escrito no.

### HU-VEN-007 — Elegir al cliente de la venta

#### TC-VEN-048 — Vender sin identificar al cliente (cliente eventual)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-007] «Vender sin identificar al cliente», «El cliente no es obligatorio en el sistema»; [CU-VEN-07] alternativo A1; [RN-VEN-025], [RN-VEN-026], [RN-CMP-005] |

**Precondiciones:** carrito solo con productos de venta libre; caja abierta.
**Pasos:** sin elegir cliente, cobra S/ 0.50 en efectivo; abre el detalle de la venta y el comprobante.
**Resultado esperado:** bajo el campo se lee «Cliente Eventual (Público General)»; el terminal no exige cliente ni existe un monto desde el cual se obligue a identificarlo; la boleta sale a nombre de «CLIENTE EVENTUAL» con documento «00000000»; el listado y el detalle de ventas muestran «Público General».

#### TC-VEN-049 — Buscar y elegir al cliente por documento o por nombre

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-007] «Buscar al cliente por documento», «Buscar al cliente por nombre», «Elegir al cliente»; [CU-VEN-07] flujo básico; [RN-VEN-025] |

**Pasos:** 1. En «Buscar paciente por nombre o documento...» escribe `45781236`. 2. Borra y escribe «Quispe». 3. Elige la opción «Rosa Quispe Demo · DNI 45781236».
**Resultado esperado:** paso 1: tras unos 300 milisegundos busca por documento y muestra hasta 20 opciones con la forma «{nombre} · DNI 45781236». Paso 2: busca por nombre y apellidos. Paso 3: bajo el campo se ve su documento, aparece el selector «Receta médica» y se cargan sus recetas.

#### TC-VEN-050 — La búsqueda sin coincidencias y el cambio de cliente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-007] «La búsqueda no encuentra a nadie», «Cambiar de cliente»; [CU-VEN-07] alternativo A2 |

**Pasos:** 1. Escribe «zzzxxx» en el campo de cliente. 2. Elige a «Rosa Quispe Demo» y una receta; luego escribe otro texto en el campo.
**Resultado esperado:** paso 1: la lista queda vacía, sin la opción «Sin resultados». Paso 2: se descartan el cliente y su receta elegida.
**Hallazgo:** no hay botón para quitar al cliente; vaciar el campo por completo no lo descarta (**No verificado** en ejecución; HU-VEN-007, «Brecha conocida»).

#### TC-VEN-051 — Los productos controlados exigen cliente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-007] «El cliente es indispensable para productos controlados»; [CU-VEN-07] alternativo A3; [RN-VEN-027] |

**Pasos:** agrega `LOT-QA-C01` (Clonazepam, Receta Retenida) sin elegir cliente y mira «Datos del Cliente».
**Resultado esperado:** sin cliente no se puede elegir una receta registrada, así que la venta de «Clonazepam Demo 2 mg» no se puede cobrar (ver `TC-VEN-055`; con receta y número sí se vende, `TC-VEN-054`).

#### TC-VEN-052 — El cliente ya no existe al cobrar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-007] «El cliente ya no existe»; [CU-VEN-07] excepción (`Paciente.NotFound`) |

**Pasos:** `POST /api/v1/ventas` (token de `carlos.cajero`, caja `S1`) con un `clienteId` inexistente (un GUID que no está en la base) y una línea de venta libre con pago exacto.
**Resultado esperado:** HTTP 404, `code` `Paciente.NotFound`; en el SPA «Ocurrió un error al procesar la venta.». No se crea la venta.

#### TC-VEN-053 — Al actualizar una venta en espera el cliente enviado reemplaza al guardado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-007] «Al actualizar una venta en espera el cliente enviado reemplaza al guardado»; [HU-VEN-021] «El cliente enviado reemplaza al guardado»; [CU-VEN-07] alternativo A4; [CU-VEN-21] alternativo A4; [RN-VEN-027] |

**Precondiciones:** una venta en espera con cliente «Rosa Quispe Demo».
**Pasos:** 1. Retómala en el terminal, quita la selección del cliente (escribe otro texto) y haz clic en «Guardar y Atender Siguiente». 2. Abre el detalle de esa venta.
**Resultado esperado:** el cliente de la venta queda en blanco («Público General»); el terminal siempre envía el cliente seleccionado en ese momento.
**Hallazgo:** si se omite `clienteId` al actualizar los detalles, el servidor borra el cliente (01c2 §5.3 hallazgo 9).

### HU-VEN-008 — Vender productos que exigen receta registrada

#### TC-VEN-054 — Vender receta retenida con receta registrada que tiene número

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-008] «Vender un producto de receta retenida con receta registrada que tiene número»; [CU-VEN-08] flujo básico; [RN-VEN-026], [RN-VEN-029], [RN-VEN-031], [RN-VEN-032], [RN-VEN-036], [RN-VEN-037] |

**Precondiciones:** caja abierta; «Rosa Quispe Demo» con `RX-QA-002` (número `RET-0001`); `LOT-QA-C01` (Clonazepam, S/ 2.50) con stock 100.
**Pasos:**
1. Agrega 2 unidades de `LOT-QA-C01` (S/ 5.00).
2. Elige a «Rosa Quispe Demo» y, en «Receta médica», la opción «{fecha} · {médico} · N.° RET-0001».
3. En «Método de Pago» deja «Efectivo» con S/ 5.00 recibidos.
4. Haz clic en «Procesar Venta» y confirma.
**Resultado esperado:** no aparece ningún aviso de receta; «Procesar Venta» queda habilitado una vez cubierto el pago; la venta se registra (estado `Completada`), el stock de `LOT-QA-C01` baja de 100 a 98 y el detalle de la venta muestra a «Rosa Quispe Demo».

#### TC-VEN-055 — Producto controlado sin receta elegida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-008] «Producto controlado sin receta elegida»; [CU-VEN-08] flujo básico y alternativo A1; [RN-VEN-029], [RN-VEN-037] |

**Precondiciones:** caja abierta.
**Pasos:** 1. Agrega `LOT-QA-C01` (o `LOT-QA-T01`, Estupefaciente) al carrito. 2. Deja «Receta médica» en «Sin receta» (o no elijas cliente). 3. Cubre el monto en efectivo.
**Resultado esperado:** se lee «Hay productos de receta retenida o estupefacientes: selecciona al cliente y una receta registrada que tenga número de receta.» y «Procesar Venta» está bloqueado.

#### TC-VEN-056 — Receta elegida sin número

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-008] «Receta elegida sin número»; [CU-VEN-08] alternativo A2; [RN-VEN-032], [RN-VEN-037] |

**Precondiciones:** `RX-QA-003` (Receta Retenida sin número) de «Rosa Quispe Demo».
**Pasos:** agrega `LOT-QA-C01`, elige a Rosa y la receta `RX-QA-003`.
**Resultado esperado:** «La receta seleccionada no tiene número de receta: agrégalo en la receta para vender productos de receta retenida o estupefacientes.» y «Procesar Venta» bloqueado.

#### TC-VEN-057 — Receta simple en una empresa que exige registrar la receta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-008] «Receta simple en una empresa que exige registrar la receta»; [CU-VEN-08] alternativo A1; [RN-VEN-028], [RN-VEN-029], [RN-VEN-037], [RN-VEN-077] |

**Precondiciones:** empresa A (modo estricto); caja abierta.
**Pasos:** agrega `LOT-QA-A01` (Amoxicilina, Receta Simple) sin elegir receta.
**Resultado esperado:** «Hay productos que requieren una receta registrada: selecciona al cliente y su receta antes de cobrar.» y «Procesar Venta» bloqueado.

#### TC-VEN-058 — Venta libre no exige receta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-008] «Venta libre no exige receta»; [CU-VEN-08] alternativo A4; [RN-VEN-029] |

**Pasos:** agrega solo `LOT-QA-P01` (Paracetamol, Venta Libre).
**Resultado esperado:** el terminal no muestra aviso de receta y no la exige.

#### TC-VEN-059 — Qué ofrece el selector de recetas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-008] «Qué ofrece el selector de recetas»; [CU-VEN-08] flujo básico |

**Precondiciones:** «Rosa Quispe Demo» con `RX-QA-001`, `RX-QA-002` y `RX-QA-003`.
**Pasos:** elige a Rosa y abre «Receta médica».
**Resultado esperado:** «Sin receta» es la primera opción; después una opción por receta registrada (hasta 50) con la forma «{fecha de expedición dd/MM/yyyy} · {médico} · N.° {número}»; bajo el campo se lee «Receta retenida y estupefacientes siempre exigen una receta registrada con su número.».

#### TC-VEN-060 — La receta elegida se adjunta a todas las líneas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-008] «La receta elegida se adjunta a todas las líneas»; [CU-VEN-08] flujo básico; [RN-VEN-030], [RN-VEN-036] |

**Pasos:** 1. Agrega 1 unidad de `LOT-QA-P01` (venta libre) y 1 de `LOT-QA-C01`. 2. Elige a Rosa y la receta `RX-QA-002`. 3. Revisa en las herramientas del navegador el cuerpo de `POST /api/v1/ventas` al cobrar.
**Resultado esperado:** el cuerpo trae `recetaId` en las dos líneas, también en la de venta libre; el servidor comprueba para cada línea que la receta exista y sea del cliente.
**Hallazgo:** el terminal adjunta una sola receta a todo el carrito aunque la API admite una por línea (01c2 hallazgo 14).

#### TC-VEN-061 — Registrar una receta nueva sin salir del terminal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-008] «Registrar una receta nueva sin salir del terminal»; [CU-VEN-08] alternativo A3 |

**Precondiciones:** `Recetas:Crear` y `ProfesionalesSalud:Leer`; cliente elegido.
**Pasos:** 1. Haz clic en «Registrar receta». 2. Registra una receta nueva en la pestaña que se abre. 3. Vuelve a la pestaña del terminal.
**Resultado esperado:** se abre `/recetas-medicas/nueva` en otra pestaña; al volver al terminal las recetas del cliente se recargan y la nueva aparece en la lista.

#### TC-VEN-062 — La receta no existe o es de otro paciente (servidor)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-008] «La receta no existe (el servidor)», «La receta es de otro paciente (el servidor)»; [CU-VEN-08] excepciones, paso «Venta» (receta inexistente y de otro paciente); [RN-VEN-030], [RN-VEN-031] |

**Precondiciones:** caja `S1` abierta; otra paciente («Paciente Otra Demo») con una receta propia.
**Pasos:** `POST /api/v1/ventas` con `clienteId` de Rosa y una línea de `LOT-QA-C01` con: (a) un `recetaId` que no existe; (b) el `recetaId` de la receta de «Paciente Otra Demo»; (c) un `recetaId` válido pero sin `clienteId`.
**Resultado esperado:** cada intento es rechazado y la venta no se crea. Mensajes en el SPA: (a) «La receta médica seleccionada no existe o fue eliminada.»; (b) y (c) «La receta médica no pertenece al cliente de la venta.». Los códigos HTTP y `code` exactos: **No verificado** en este caso (consulta el [catálogo de errores](../../ola-1-fundamentos/21-catalogo-codigos-error.md) para los códigos `Venta.Receta…`).

#### TC-VEN-063 — Falta la receta o su número (servidor)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-008] «Falta la receta o su número (el servidor)»; [CU-VEN-08] excepciones, paso «Venta»; [RN-VEN-029], [RN-VEN-032] |

**Pasos:** por API, con `clienteId` de Rosa: (a) `POST /api/v1/ventas` con una línea de `LOT-QA-C01` sin `recetaId`; (b) la misma con `recetaId` de `RX-QA-003`.
**Resultado esperado:** (a) en el SPA «Uno de los productos requiere receta médica. Selecciona el cliente y una de sus recetas registradas.»; (b) «La receta seleccionada no tiene número de receta. Agrégalo en la receta para vender productos de receta retenida o estupefacientes.». No se crea la venta ni baja el stock.

#### TC-VEN-064 — Dejar la venta en espera no exige receta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-008] «Dejar la venta en espera no exige receta»; [CU-VEN-08] alternativo A5; [RN-VEN-037] |

**Pasos:** agrega `LOT-QA-C01`, no elijas receta y haz clic en «Guardar y Atender Siguiente».
**Resultado esperado:** la venta se guarda en espera sin exigir receta ni número (estado `Pendiente`).

### HU-VEN-009 — Confirmar «Receta verificada» en empresas flexibles

#### TC-VEN-065 — Vender receta simple sin receta registrada marcando la casilla

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-009] «Vender receta simple sin receta registrada marcando la casilla»; [CU-VEN-09] flujo básico; [RN-VEN-028], [RN-VEN-034], [RN-VEN-035], [RN-EMP-016] |

**Precondiciones:** empresa flexible (`ExigirRegistroReceta` en falso), caja abierta.
**Pasos:** 1. Agrega `LOT-QA-A01` (Amoxicilina) sin receta ni productos controlados. 2. Observa la casilla y «Procesar Venta». 3. Marca la casilla y cobra en efectivo.
**Resultado esperado:** casilla «Receta verificada» con el texto «Confirmas que revisaste la receta física del cliente. Quedará registrado a tu nombre.»; «Procesar Venta» deshabilitado hasta marcarla; al cobrar, la venta se registra y guarda quién la verificó y cuándo.
**Hallazgo:** solo se guarda quién y cuándo; el número de la receta de papel no se guarda (RN-VEN-035).

#### TC-VEN-066 — El detalle de la venta muestra la verificación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-009] «El detalle de la venta muestra la verificación»; [HU-VEN-027] «Verificación de receta»; [CU-VEN-09] flujo básico; [CU-VEN-27] flujo básico; [RN-VEN-035] |

**Pasos:** abre `/pos/{id}` de la venta de `TC-VEN-065`.
**Resultado esperado:** bloque «Verificación de receta» con «Receta verificada por Carlos Cajero Demo el {fecha y hora}».

#### TC-VEN-067 — Con receta registrada o producto controlado la casilla no aparece

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-009] «Con receta registrada no se pide confirmación», «Con un producto controlado la casilla no sirve»; [CU-VEN-09] alternativos A1 y A2; [RN-VEN-029], [RN-VEN-034] |

**Precondiciones:** empresa flexible; caja abierta.
**Pasos:** 1. Agrega `LOT-QA-A01`, elige a Rosa y `RX-QA-001`. 2. Quita la receta, agrega además `LOT-QA-C01`.
**Resultado esperado:** paso 1: la casilla «Receta verificada» no aparece y no se guarda verificación al cobrar. Paso 2: la casilla no aparece; sigue haciendo falta una receta registrada con número (aviso de `TC-VEN-055`).

#### TC-VEN-068 — La casilla se desmarca sola y no se guarda en el borrador

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-009] «La casilla se desmarca sola»; [CU-VEN-09] alternativo A3 |

**Precondiciones:** empresa flexible.
**Pasos:** agrega Amoxicilina, marca la casilla y quita la línea; agrega otra línea de Amoxicilina.
**Resultado esperado:** al quitar la línea la casilla se desmarca; al volver a agregar debe marcarse de nuevo.

#### TC-VEN-069 — Dejar en espera sin confirmar y cobrar verificando con otro cajero

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-009] «Dejar la venta en espera no pide la confirmación», «Quien cobra es quien verifica»; [CU-VEN-09] alternativo A4; [RN-VEN-034], [RN-VEN-035] |

**Precondiciones:** empresa flexible; dos cajeros con caja abierta (Carlos y otro).
**Pasos:** 1. Carlos agrega Amoxicilina y hace clic en «Guardar y Atender Siguiente» sin marcar la casilla. 2. El otro cajero (con `Ventas:Actualizar`) retoma la venta, marca «Receta verificada» y cobra.
**Resultado esperado:** paso 1: la venta queda en espera. Paso 2: se cobra y el detalle muestra «Receta verificada por {nombre del otro cajero} el {fecha y hora}».

#### TC-VEN-070 — El servidor exige la confirmación que el terminal no mostró

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-009] «El servidor exige la confirmación que el terminal no mostró»; [CU-VEN-09] flujo básico, excepción; [RN-VEN-028], [RN-VEN-034] |

**Precondiciones:** empresa en modo flexible (`ExigirRegistroReceta` en falso) pero con el terminal abierto en modo estricto (por ejemplo, porque falló la carga de la configuración, `TC-VEN-009`); caja abierta.
**Pasos:** 1. Abre el terminal con la configuración bloqueada (modo estricto). 2. Ajusta el estado para que el servidor exija la confirmación (envía por API `POST /api/v1/ventas` con una línea de Amoxicilina, sin `recetaId` y con `"recetaVerificada":false`).
**Resultado esperado:** HTTP 400; el SPA muestra «Marca "Receta verificada" para confirmar que revisaste la receta del cliente antes de cobrar.», pasa a modo flexible y muestra la casilla. **No verificado:** el `code` exacto y si el SPA reproduce el mensaje sin pasar por la API (19f §3.5).

#### TC-VEN-071 — Si ninguna línea necesita verificación el valor enviado se ignora

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-009] «Si ninguna línea la necesita, el valor enviado se ignora»; [CU-VEN-09] alternativo A5; [RN-VEN-034], [RN-VEN-035] |

**Pasos:** `POST /api/v1/ventas` con una línea de `LOT-QA-P01` y `"recetaVerificada":true`; abre el detalle.
**Resultado esperado:** la venta se registra y el detalle no muestra «Verificación de receta»: el servidor no guarda quién ni cuándo.

### HU-VEN-010 — Respetar el máximo de dispensaciones de una receta

#### TC-VEN-072 — Cuántas dispensaciones admite una receta (esquema)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-VEN-010] «Cuántas dispensaciones admite una receta»; [CU-VEN-10] flujo básico; [RN-VEN-033], [RN-REC-031] |

**Precondiciones:** una receta por condición (Venta Libre, Receta Simple con 0 repeticiones, Receta Retenida, Estupefaciente), todas del mismo paciente.
**Pasos:** para cada una, cobra ventas consecutivas con esa receta hasta que el sistema rechace.
**Resultado esperado:**

| Condición | Máximo de ventas que la pueden usar |
|---|---|
| Venta Libre (OTC) | sin límite (no cuenta ni consume) |
| Receta Simple | 1 más las repeticiones máximas |
| Receta Retenida | 1 |
| Estupefaciente | 1 |

El rechazo, al pasar el máximo, es «La receta médica ya fue dispensada el máximo de veces permitido.».

#### TC-VEN-073 — Segunda venta con una receta retenida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-010] «Segunda venta con una receta retenida»; [CU-VEN-10] flujo básico; [RN-VEN-033] |

**Precondiciones:** `RX-QA-002` ya usada en una venta cobrada (`TC-VEN-054`).
**Pasos:** arma otra venta de `LOT-QA-C01` con la misma receta y cobra.
**Resultado esperado:** «La receta médica ya fue dispensada el máximo de veces permitido.»; no se guarda la venta y el stock no baja.

#### TC-VEN-074 — Receta simple con 2 repeticiones máximas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-VEN-010] «Receta simple con 2 repeticiones máximas»; [CU-VEN-10] flujo básico; [RN-VEN-033] |

**Precondiciones:** una receta simple de Rosa (con Amoxicilina) con 2 repeticiones máximas.
**Pasos:** cobra tres ventas distintas con esa receta y luego una cuarta.
**Resultado esperado:** las tres primeras se registran; la cuarta se rechaza con «La receta médica ya fue dispensada el máximo de veces permitido.».

#### TC-VEN-075 — Varias líneas de la misma receta son una sola dispensación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-010] «Varias líneas de la misma receta son una sola dispensación»; [CU-VEN-10] flujo básico; [RN-VEN-033] |

**Precondiciones:** receta simple con 1 repetición máxima (admite 2 ventas) que cubra dos productos de receta.
**Pasos:** vende una sola vez dos líneas de productos de receta con esa receta; luego una segunda venta con la misma receta; luego una tercera.
**Resultado esperado:** la primera venta consume una sola dispensación (la segunda se registra); la tercera se rechaza con «La receta médica ya fue dispensada el máximo de veces permitido.».

#### TC-VEN-076 — Una venta en espera ya cuenta como dispensación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-VEN-010] «Una venta en espera ya cuenta como dispensación»; [CU-VEN-10] alternativo A1; [RN-VEN-033] |

**Precondiciones:** receta retenida `RX-QA-002` sin usar; dos cajeros con caja abierta.
**Pasos:** 1. Carlos deja en espera una venta de `LOT-QA-C01` con `RX-QA-002`. 2. El otro cajero intenta cobrar otra venta con la misma receta.
**Resultado esperado:** la segunda venta se rechaza con «La receta médica ya fue dispensada el máximo de veces permitido.».

#### TC-VEN-077 — Anular la venta libera la dispensación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-010] «Anular la venta libera la dispensación»; [CU-VEN-10] alternativo A2; [RN-VEN-033] |

**Precondiciones:** venta cobrada con `RX-QA-002`, anulable (sin devoluciones, ver [HU-VEN-023]).
**Pasos:** 1. Anula esa venta (`TC-VEN-172`). 2. Cobra otra venta de `LOT-QA-C01` con la misma receta.
**Resultado esperado:** el sistema permite la segunda venta, porque el conteo ignora las ventas anuladas.

#### TC-VEN-078 — Devolver no libera la dispensación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-010] «Devolver no libera la dispensación»; [CU-VEN-10] alternativo A3; [RN-VEN-033], [RN-DEV-026] |

**Precondiciones:** venta cobrada con `RX-QA-002` y con una devolución (parcial o total).
**Pasos:** 1. Intenta anular la venta. 2. Cobra otra venta con la misma receta.
**Resultado esperado:** paso 1: la venta no se puede anular (ver `TC-VEN-183`). Paso 2: se rechaza con «La receta médica ya fue dispensada el máximo de veces permitido.»; la venta con devolución sigue contando.

#### TC-VEN-079 — Las líneas de venta libre con receta no cuentan

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-010] «Las líneas de venta libre con receta no cuentan»; [CU-VEN-10] flujo básico; [RN-VEN-033] |

**Pasos:** vende solo `LOT-QA-P01` (venta libre) adjuntando por API `recetaId` de `RX-QA-002`, dos veces.
**Resultado esperado:** ambas ventas se registran: esas líneas no consumen ni verifican dispensaciones; `RX-QA-002` sigue disponible.

#### TC-VEN-080 — Actualizar una venta en espera no se cuenta contra sí misma

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-010] «Actualizar una venta en espera no se cuenta contra sí misma»; [CU-VEN-10] alternativo A4; [RN-VEN-033] |

**Precondiciones:** venta en espera con `RX-QA-002`.
**Pasos:** retoma la venta, cambia la cantidad de la línea de Clonazepam y haz clic en «Guardar y Atender Siguiente».
**Resultado esperado:** la actualización se guarda; las líneas anteriores de esa venta no cuentan para el máximo de esa misma venta.
**Hallazgo:** al cobrar una venta en espera sin reenviar las líneas (solo API) no se vuelve a comprobar el máximo ([RN-VEN-016]).

### HU-VEN-011 — Agregar al carrito los medicamentos de una receta escaneada

#### TC-VEN-081 — La pestaña de escaneo solo aparece con el permiso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-VEN-011] «La pestaña solo aparece con el permiso»; [CU-VEN-11] flujo básico, paso 1 |

**Pasos:** abre el terminal con un rol sin `Recetas:Analizar` y luego con `carlos.cajero`.
**Resultado esperado:** sin el permiso no se muestran las pestañas «Producto» y «Receta», solo la búsqueda de productos; con el permiso aparecen las dos pestañas.

#### TC-VEN-082 — Interpretar una receta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Media | [HU-VEN-011] «Interpretar una receta»; [CU-VEN-11] flujo básico |

**Precondiciones:** servicio de análisis de recetas con IA configurado (parte B); una imagen de receta de prueba con medicamentos conocidos.
**Pasos:** 1. Abre la pestaña «Receta». 2. Elige la imagen con «Galería» (o arrástrala). 3. Haz clic en «Interpretar receta».
**Resultado esperado:** mientras se analiza se lee «Interpretando...»; luego el banner «{n} medicamento(s) detectado(s) en la receta» y una fila por cada medicamento de tipo receta.
**Hallazgo:** `n` cuenta todos los ítems devueltos, también los de tipo «Recomendacion», que el terminal no lista (HU-VEN-011, «Brecha conocida»).

#### TC-VEN-083 — Cada fila indica confianza, stock y validación DIGEMID

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Baja | [HU-VEN-011] «Cada fila indica confianza, stock y validación DIGEMID»; [CU-VEN-11] flujo básico |

**Pasos:** interpreta la receta de `TC-VEN-082` y lee cada fila.
**Resultado esperado:** cada fila muestra la confianza «Alta» (0,85 o más), «Media» (0,65 o más) o «Baja»; «En stock» o «Sin stock»; y la insignia «DIGEMID» si está validado.

#### TC-VEN-084 — Agregar un medicamento detectado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-011] «Agregar un medicamento detectado»; [CU-VEN-11] flujo básico; [RN-VEN-036], [RN-VEN-075] |

**Precondiciones:** un medicamento detectado con stock en «Sede Principal».
**Pasos:** haz clic en el botón «+» de su fila; vuelve a hacer clic.
**Resultado esperado:** se agrega al carrito el primer lote disponible (el que vence antes), sin presentación y a precio de catálogo; la cantidad es la sugerida o 1 sin pasar de la existencia del lote; el botón cambia a «✓» y no se vuelve a agregar.

#### TC-VEN-085 — El medicamento detectado sin stock o sin precio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-011] «El medicamento no tiene stock en mi sucursal», «El medicamento no tiene precio»; [CU-VEN-11] flujo básico; [RN-VEN-075] |

**Pasos:** 1. Agrega un medicamento detectado que no tiene lotes en la sucursal. 2. Agrega uno cuyo primer lote no tiene precio de catálogo.
**Resultado esperado:** paso 1: «El medicamento detectado no tiene stock en esta sucursal.». Paso 2: «Este producto no tiene precio de venta configurado en el catálogo. Pide a un administrador que lo registre.».

#### TC-VEN-086 — No se detecta nada o falla el análisis

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-VEN-011] «No se detectó nada», «Falla el análisis»; [CU-VEN-11] alternativo A1 |

**Pasos:** 1. Interpreta una imagen sin medicamentos. 2. Interpreta con el servicio de análisis caído (o bloqueando `POST /api/v1/recetas/analizar`).
**Resultado esperado:** paso 1: «No se detectaron medicamentos en la imagen. Intenta con una foto más nítida.». Paso 2: «Error al interpretar la receta. Intenta nuevamente.».

#### TC-VEN-087 — Escanear no asocia ninguna receta registrada a la venta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-011] «Escanear no asocia ninguna receta registrada a la venta»; [CU-VEN-11] flujo básico; [RN-VEN-036] |

**Pasos:** agrega desde el escaneo un medicamento de receta retenida y trata de cobrar sin elegir cliente ni receta.
**Resultado esperado:** aparece el aviso de `TC-VEN-055`; para vender productos de receta hay que elegir al cliente y su receta registrada.

### HU-VEN-012 — Cobrar en efectivo y entregar el vuelto

#### TC-VEN-088 — El cliente paga con más efectivo y recibe vuelto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-012] «El cliente paga con más efectivo y recibe vuelto»; [CU-VEN-12] flujo básico; [RN-VEN-043], [RN-VEN-044] |

**Precondiciones:** caja abierta; carrito con 100 unidades de `LOT-QA-P01` (TOTAL A PAGAR S/ 50.00).
**Pasos:** 1. Elige la ficha «Efectivo». 2. Escribe «Recibido (S/)» 60.00. 3. Haz clic en «Procesar Venta».
**Resultado esperado:** junto al campo se lee «Vuelto: S/ 10.00»; en los totales «Vuelto a Entregar:» S/ 10.00; «Procesar Venta» queda habilitado; en el diálogo de confirmación se lee «Vuelto Entregado:» S/ 10.00.

#### TC-VEN-089 — El cliente paga el monto exacto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-VEN-012] «El cliente paga el monto exacto»; [CU-VEN-12] alternativo A1; [RN-VEN-043] |

**Precondiciones:** TOTAL A PAGAR S/ 50.00.
**Pasos:** escribe «Recibido (S/)» 50.00.
**Resultado esperado:** el monto pendiente es 0, no hay vuelto y «Procesar Venta» queda habilitado.

#### TC-VEN-090 — El efectivo no alcanza

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-VEN-012] «El efectivo no alcanza»; [CU-VEN-12] alternativo A2; [RN-VEN-043] |

**Pasos:** con TOTAL A PAGAR S/ 50.00, escribe «Recibido (S/)» 40.00 y luego 49.99.
**Resultado esperado:** con 40.00, en los totales se lee «Monto Pendiente:» S/ 10.00 y «Procesar Venta» está deshabilitado; con 49.99, «Monto Pendiente:» S/ 0.01 y sigue deshabilitado. No aparece ningún mensaje de «monto insuficiente».

#### TC-VEN-091 — El pago inicial es efectivo por 0 y no se puede cobrar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-012] «El pago inicial»; [CU-VEN-12] flujo básico |

**Pasos:** agrega productos y no toques los pagos.
**Resultado esperado:** hay un único pago en «Efectivo» por 0; un monto de 0 no cubre el total y «Procesar Venta» está deshabilitado.

#### TC-VEN-092 — Qué se registra del pago en efectivo y el vuelto en el detalle

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-012] «Qué se registra del pago en efectivo»; [CU-VEN-12] flujo básico; [RN-VEN-041], [RN-VEN-044] |

**Precondiciones:** venta de S/ 50.00 cobrada con «Recibido (S/)» 60.00 (`TC-VEN-088`).
**Pasos:** abre `/pos/{id}` de esa venta.
**Resultado esperado:** el pago queda registrado por S/ 60.00 (lo que entregó el cliente), no por 50.00; el vuelto no se guarda sino que se recalcula como suma de pagos menos el monto a pagar; el detalle muestra «Vuelto:» S/ 10.00.

#### TC-VEN-093 — La suma de pagos debe cubrir el monto sin tolerancia (API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-VEN-012] «La suma de pagos debe cubrir el monto a pagar sin tolerancia»; [CU-VEN-12] excepción (`Venta.Pagos`); [RN-VEN-043] |

**Precondiciones:** caja `S1` abierta; `LOT-QA-P01`.
**Pasos:** `POST /api/v1/ventas` con 100 unidades de `LOT-QA-P01` (total S/ 50.00) y `"pagos":[{"metodoPago":"Efectivo","montoPagado":49.99}]`; luego con `montoPagado` 50.00.
**Resultado esperado:** con 49.99: HTTP 400, `code` `Venta.Pagos`; en el SPA «Ocurrió un error al procesar la venta.»; no se crea la venta. Con 50.00: se crea la venta.
**Hallazgo:** `Venta.Pagos` no está traducido (HU-VEN-012, «Brecha conocida»).

#### TC-VEN-094 — El vuelto sale del cajón: no puede superar el efectivo entregado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-012] «El vuelto sale del cajón»; [CU-VEN-12] excepción (`VueltoExcedeEfectivo`); [RN-VEN-044] |

**Pasos:** 1. En el SPA, con total S/ 50.00, usa pago dividido: «Tarjeta Débito/Crédito» por 60.00 con número de operación y sin efectivo (o con efectivo menor que el vuelto, ver `TC-VEN-105`). 2. Por API envía los mismos pagos.
**Resultado esperado:** paso 1: «Procesar Venta» queda deshabilitado. Paso 2: «El vuelto no puede ser mayor que el efectivo recibido. Ajusta los montos de pago.».

#### TC-VEN-095 — Cada pago debe ser mayor que 0 (API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-012] «Cada pago debe ser mayor que 0»; [CU-VEN-12] excepción; [RN-VEN-041], [RN-GEN-011] |

**Pasos:** `POST /api/v1/ventas` con `"pagos":[{"metodoPago":"Efectivo","montoPagado":0}]`, luego con `montoPagado` -5.
**Resultado esperado:** HTTP 400, `code` `CreateVentaCommand.Pagos[0].MontoPagado`; en el SPA «Ocurrió un error al procesar la venta.»; no se crea la venta.

#### TC-VEN-096 — Una venta cobrada pasa a «Completada»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-012] «Una venta cobrada pasa a "Completada"»; [CU-VEN-12] flujo básico; [RN-VEN-046], [RN-VEN-041], [RN-VEN-081] |

**Pasos:** cobra una venta en efectivo y abre su detalle.
**Resultado esperado:** estado «Completada»; el instante del pago es el del cobro (hora del servidor). **No verificado:** cómo convierte la base de datos un monto con más de 2 decimales (RN-VEN-041, RN-VEN-043).

### HU-VEN-013 — Cobrar con Yape / Plin, tarjeta o transferencia

#### TC-VEN-097 — Cobrar con Yape / Plin

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-013] «Cobrar con Yape / Plin»; [CU-VEN-13] flujo básico; [RN-VEN-040], [RN-VEN-042] |

**Precondiciones:** carrito de S/ 50.00.
**Pasos:** 1. Haz clic en la ficha «Yape / Plin». 2. Escribe «Nro Operación» `YP-778812`. 3. Cobra.
**Resultado esperado:** queda un único pago por Yape / Plin con monto S/ 50.00; aparecen «Monto (S/)» y «Nro Operación»; con el número escrito «Procesar Venta» queda habilitado; la venta se registra con el método `YapePlin` y el número de operación `YP-778812`, sin vuelto.

#### TC-VEN-098 — Falta el número de operación o solo espacios

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-013] «Falta el número de operación», «Un número de solo espacios cuenta como faltante»; [CU-VEN-13] flujo básico; [RN-VEN-042] |

**Pasos:** 1. Elige «Tarjeta Débito/Crédito» y no escribas «Nro Operación». 2. Escribe solo espacios.
**Resultado esperado:** en rojo bajo los pagos «Ingresa el N° de operación para poder cobrar.» y «Procesar Venta» deshabilitado; con solo espacios el terminal lo trata como vacío y no deja cobrar.

#### TC-VEN-099 — Límite de largo y recorte del número de operación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-VEN-013] «Límite de largo y recorte», «Un número de operación de más de 100 caracteres (el servidor)»; [CU-VEN-13] flujo básico; [RN-VEN-042], [RN-VEN-081] |

**Pasos:** 1. Escribe 101 caracteres en «Nro Operación». 2. Escribe `  OP-QA-0003  ` (con espacios) y cobra; revisa el cuerpo de la petición. 3. Por API envía `referenciaOperacion` de 101 caracteres.
**Resultado esperado:** paso 1: el campo corta en 100 caracteres. Paso 2: el terminal envía `OP-QA-0003` sin espacios. Paso 3: el servidor rechaza la petición y el SPA mostraría «Ocurrió un error al procesar la venta.» (el `code` exacto es **No verificado**).

#### TC-VEN-100 — El efectivo no pide número de operación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-013] «El efectivo no pide número de operación»; [CU-VEN-13] alternativo A1 |

**Pasos:** elige la ficha «Efectivo».
**Resultado esperado:** no aparece el campo «Nro Operación».

#### TC-VEN-101 — Métodos de pago disponibles (esquema)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-013] «Métodos de pago disponibles»; [CU-VEN-13] flujo básico; [RN-VEN-040], [RN-VEN-081] |

**Pasos:** elige cada ficha y observa si pide número de operación; cobra una venta con cada método.
**Resultado esperado:**

| Ficha | Método | Pide |
|---|---|---|
| Efectivo | Efectivo | no pide número de operación |
| Yape / Plin | YapePlin | pide número de operación |
| Tarjeta Débito/Crédito | Tarjeta | pide número de operación |
| Transferencia | Transferencia | pide número de operación |

#### TC-VEN-102 — El servidor repite la exigencia del número de operación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-013] «El servidor repite la exigencia»; [CU-VEN-13] excepción; [RN-VEN-042] |

**Pasos:** `POST /api/v1/ventas` con `"pagos":[{"metodoPago":"Tarjeta","montoPagado":50.00}]` (sin `referenciaOperacion`).
**Resultado esperado:** rechazo; el SPA mostraría «Ingresa el número de operación de cada pago que no sea en efectivo.». No se crea la venta. El `code` y el HTTP exactos: **No verificado** en este caso (ver [catálogo de errores](../../ola-1-fundamentos/21-catalogo-codigos-error.md)).

#### TC-VEN-103 — El mismo número de operación en dos ventas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-013] «El mismo número en dos ventas»; [CU-VEN-13] alternativo A2; [RN-VEN-040] |

**Precondiciones:** una venta ya cobrada con `YP-778812` (`TC-VEN-097`).
**Pasos:** cobra una venta nueva con el mismo `YP-778812`.
**Resultado esperado:** el sistema la acepta.
**Hallazgo:** el número de operación no es único ni se valida su formato (RN-VEN-040, RN-VEN-042; 01c2 hallazgo 18).

### HU-VEN-014 — Dividir el pago en varios métodos

#### TC-VEN-104 — Pagar S/ 50.00 con Yape / Plin y efectivo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-014] «Pagar S/ 50.00 con Yape / Plin y efectivo»; [CU-VEN-14] flujo básico; [RN-VEN-039], [RN-VEN-041], [RN-VEN-042], [RN-VEN-043], [RN-VEN-044] |

**Precondiciones:** total S/ 50.00; caja abierta con apertura de S/ 100.00.
**Pasos:** 1. Haz clic en «+ Dividir en varios métodos de pago». 2. Registra «Yape / Plin» por 40.00 con `YP-778812` y «Efectivo» por 15.00. 3. Cobra. 4. Cierra la caja luego y mira el efectivo esperado (ver `TC-CAJ-028`).
**Resultado esperado:** la suma de pagos es S/ 55.00 y se lee «Vuelto a Entregar:» S/ 5.00; como el vuelto no supera el efectivo, «Procesar Venta» queda habilitado; el efectivo neto que entra al cajón es S/ 10.00 (15.00 − 5.00).

#### TC-VEN-105 — El vuelto supera el efectivo entregado en pago dividido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-VEN-014] «El vuelto supera el efectivo entregado»; [CU-VEN-14] excepción (`VueltoExcedeEfectivo`); [RN-VEN-044] |

**Pasos:** 1. Con total S/ 50.00 registra «Tarjeta Débito/Crédito» por 60.00 (con número de operación) y «Efectivo» por 5.00. 2. Por API envía esos mismos pagos.
**Resultado esperado:** la suma es 65.00 y el vuelto 15.00 supera el efectivo 5.00; «Procesar Venta» queda deshabilitado sin ningún mensaje. Por API: «El vuelto no puede ser mayor que el efectivo recibido. Ajusta los montos de pago.». En consecuencia, los pagos no efectivo no pueden sumar más que el monto a pagar.

#### TC-VEN-106 — Agregar otro método de pago

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-014] «Agregar otro método»; [CU-VEN-14] flujo básico |

**Pasos:** en el modo dividido haz clic en «Agregar Método de Pago» varias veces.
**Resultado esperado:** se agrega una fila con el primer método que aún no usaste (o «Efectivo» si ya usaste los cuatro) y con el monto pendiente.

#### TC-VEN-107 — No se repite un método de pago

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-014] «No se repite un método»; [CU-VEN-14] alternativo A1 |

**Precondiciones:** una fila «Efectivo» de 20.00 y otra fila «Yape / Plin» con `YP-1` y monto 10.00.
**Pasos:** cambia la fila de Yape / Plin a «Efectivo».
**Resultado esperado:** las dos filas se fusionan en una, se suman los montos (30.00) y se conserva el primer número de operación no vacío.

#### TC-VEN-108 — Quitar un pago y volver a pago simple

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-014] «Quitar un pago», «Volver a pago simple»; [CU-VEN-14] alternativos A2 y A3 |

**Pasos:** 1. Con dos filas, haz clic en la «×» de una; luego intenta quitar la única. 2. Con varias filas haz clic en «Volver a pago simple».
**Resultado esperado:** paso 1: la fila desaparece, pero no se puede quitar el único pago. Paso 2: queda un único pago en «Efectivo» por el total a pagar y se pierden los demás.

#### TC-VEN-109 — Un pago no efectivo sin número se marca

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-014] «Un pago no efectivo sin número se marca»; [CU-VEN-14] flujo básico; [RN-VEN-042] |

**Pasos:** en el modo dividido deja una fila «Transferencia» sin «Nro Operación».
**Resultado esperado:** ese campo se marca en rojo y se lee «Ingresa el N° de operación para poder cobrar.».

#### TC-VEN-110 — La API acepta cualquier cantidad de pagos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-014] «La API acepta cualquier cantidad de pagos»; [CU-VEN-14] alternativo A4; [RN-VEN-045] |

**Pasos:** `POST /api/v1/ventas` con total S/ 50.00 y `"pagos":[{"metodoPago":"Efectivo","montoPagado":20},{"metodoPago":"Efectivo","montoPagado":30}]`.
**Resultado esperado:** el servidor los acepta y registra la venta como «Completada»; no hay máximo de cantidad de pagos mientras cada uno sea mayor que 0 y la suma cubra el monto.
**Hallazgo:** el SPA fusiona dos filas del mismo método (`TC-VEN-107`); el servidor no lo exige y acepta pagos repetidos del mismo método (RN-VEN-045).

#### TC-VEN-111 — Cobrar sin pagos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-014] «Cobrar sin pagos»; [CU-VEN-14] excepción; [RN-VEN-039] |

**Pasos:** `POST /api/v1/ventas` con `"estado":"Completada"` y `"pagos":[]`.
**Resultado esperado:** HTTP 400, `code` `CreateVentaCommand.Pagos`; el SPA mostraría «Ocurrió un error al procesar la venta.»; no se crea la venta.

### HU-VEN-015 — Confirmar y procesar la venta

#### TC-VEN-112 — Procesar una venta correcta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-015] «Procesar una venta correcta»; [CU-VEN-15] flujo básico; [RN-VEN-015], [RN-VEN-021], [RN-VEN-049], [RN-VEN-050], [RN-VEN-051] |

**Precondiciones:** caja abierta de Carlos; `LOT-QA-P01` con 500 unidades; carrito con 100 unidades (S/ 50.00), pago en efectivo con 60.00.
**Pasos:** 1. Haz clic en «Procesar Venta». 2. Lee el diálogo «Confirmar Procesamiento de Venta». 3. Haz clic en «Confirmar y Procesar Venta».
**Resultado esperado:** el diálogo dice «Verifica los detalles antes de registrar la transacción.» y muestra «Cliente:» («Cliente Eventual (Público General)»), «Cantidad de Ítems:» (1), «Método de Pago:», «Total a Pagar:» S/ 50.00 y «Vuelto Entregado:» S/ 10.00. Al confirmar: la venta queda «Completada», el stock de `LOT-QA-P01` baja de 500 a 400, se emite la boleta y se abre el diálogo «Venta Procesada Exitosamente». No aparece ningún aviso adicional.

#### TC-VEN-113 — Cancelar la confirmación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-015] «Cancelar la confirmación»; [CU-VEN-15] alternativo A1 |

**Pasos:** abre el diálogo de confirmación y haz clic en «Cancelar»; ábrelo otra vez y pulsa Esc.
**Resultado esperado:** vuelves al terminal sin cambios y no se envía nada (sin petición `POST /api/v1/ventas`).

#### TC-VEN-114 — «Vuelto Entregado» solo aparece si se pagó de más

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-015] «"Vuelto Entregado" solo aparece si se pagó de más»; [CU-VEN-15] flujo básico |

**Pasos:** paga el monto exacto y abre el diálogo de confirmación.
**Resultado esperado:** el diálogo no muestra «Vuelto Entregado:».

#### TC-VEN-115 — Todo se guarda en una sola operación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-015] «Todo se guarda en una sola operación»; [CU-VEN-15] flujo básico; [RN-VEN-015], [RN-VEN-050] |

**Precondiciones:** `LOT-QA-P01` (stock 500) y `LOT-QA-P02` (stock 30).
**Pasos:** por API, `POST /api/v1/ventas` con dos líneas: 10 unidades de `LOT-QA-P01` y 31 unidades de `LOT-QA-P02`, con pago en efectivo suficiente.
**Resultado esperado:** HTTP 400, `code` `Venta.Stock`; no se guarda nada de la venta y el stock de `LOT-QA-P01` sigue en 500 (el descuento en memoria se descarta). El único dato que puede cambiar es el correlativo de la serie, dentro de la misma transacción (**No verificado** en ejecución).

#### TC-VEN-116 — Quién registra la venta y cuándo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-015] «Quién registra y cuándo»; [CU-VEN-15] flujo básico; [RN-VEN-002], [RN-VEN-058] |

**Pasos:** cobra una venta; abre el detalle; intenta enviar por API un campo de fecha en el cuerpo.
**Resultado esperado:** la venta queda en la sucursal base de Carlos, a nombre de su empleado, con fecha y hora del servidor (UTC); el cliente no puede enviar la fecha (el comando no tiene ese campo).

#### TC-VEN-117 — Se registra un evento para integraciones futuras

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-015] «Se registra un evento para integraciones futuras»; [CU-VEN-15] flujo básico; [RN-VEN-060], [RN-GEN-007] |

**Pasos:** tras `TC-VEN-112`, consulta en la base de datos la bandeja de salida de eventos.
**Resultado esperado:** existe el evento `VentaCompletadaEvent` y ningún proceso lo consume. **No verificado:** el nombre de la tabla de la bandeja de salida (consulta el [modelo de datos](../../ola-1-fundamentos/03-modelo-de-datos/03b-transaccionales-e-historial.md)).

#### TC-VEN-118 — Cobrar no deja evento de auditoría de seguridad

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-015] «Cobrar no deja evento de auditoría de seguridad»; [CU-VEN-15] flujo básico; [RN-VEN-061] |

**Pasos:** tras `TC-VEN-112`, revisa la tabla de auditoría.
**Resultado esperado:** no hay evento de auditoría para el cobro; la trazabilidad es el usuario y la fecha de creación y modificación de las filas. (Anular una venta sí deja un evento: ver `TC-VEN-173`.)

#### TC-VEN-119 — Falla el cobro y el carrito se conserva

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-015] «Falla el cobro y el carrito se conserva»; [CU-VEN-15] excepciones; [RN-VEN-015] |

**Pasos:** 1. Con un carrito armado, bloquea la red (o bloquea `POST /api/v1/ventas` en las herramientas del navegador) y confirma el cobro. 2. Repite con un lote agotado por otro cajero (`TC-VEN-031`).
**Resultado esperado:** para cualquier código no traducido o sin red, se ve «Ocurrió un error al procesar la venta.»; el carrito y los pagos se conservan para corregir y reintentar; ninguna venta se registra.

#### TC-VEN-120 — Otra operación modificó el stock o la venta (409)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-VEN-015] «Otra operación modificó el stock o la venta»; [CU-VEN-15] excepción (`Concurrency.Conflict`); [RN-VEN-057], [RN-GEN-035] |

**Precondiciones:** dos cajeros con caja abierta; `LOT-QA-P01` con stock suficiente para ambos.
**Pasos:** los dos cajeros confirman al mismo tiempo (envíalo con dos clientes HTTP en paralelo) una venta del mismo lote.
**Resultado esperado:** si el servidor detecta el conflicto de versión de fila, uno de los cobros recibe HTTP 409 y se ve «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.»; el stock final es coherente (suma de lo vendido). **No verificado en ejecución:** se deduce de la versión de fila del stock y de la venta; puede que ambos cobros se registren si hay stock para los dos.

#### TC-VEN-121 — Restricciones de la base de datos que respaldan el cobro

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-015] «Restricciones de la base de datos que respaldan el cobro»; [CU-VEN-15] excepciones (`Db.ValorInvalido`, `Db.ReferenciaInvalida`); [RN-VEN-079], [RN-VEN-080], [RN-VEN-081] |

**Pasos:** por API intenta guardar: un pago con monto 0 o negativo, una línea con cantidad 0 o negativa, un `clienteId`/`loteId` inexistente, y un estado fuera de `Completada`, `Anulada`, `Devuelta` o `Pendiente`.
**Resultado esperado:** la base de datos o el validador lo rechazan con HTTP 400 `Db.ValorInvalido` o 409 `Db.ReferenciaInvalida` (o el código de validación del comando, según el caso); el cajero ve «Ocurrió un error al procesar la venta.» porque esos códigos no están en el mapa de errores del cobro. **No verificado:** qué petición dispara cuál código; consulta [01c2](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) §5.

#### TC-VEN-122 — Un doble clic no cobra dos veces

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-VEN-015] «Un doble clic no cobra dos veces»; [CU-VEN-15] alternativo A2 |

**Pasos:** en el diálogo de confirmación haz doble clic rápido en «Confirmar y Procesar Venta».
**Resultado esperado:** el diálogo se cierra al instante y «Procesar Venta» queda «procesando»; se registra una sola venta y el stock baja una sola vez.

#### TC-VEN-123 — La API acepta dos estados al crear

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-015] «La API acepta dos estados al crear»; [CU-VEN-15] alternativo A3; [RN-VEN-051] |

**Pasos:** `POST /api/v1/ventas` con: (a) `"estado":"Completada"` y un pago suficiente; (b) `"estado":"Pendiente"` y `"pagos":[]`; (c) `"estado":"Pendiente"` con un pago suficiente; (d) `"estado":"Anulada"` con pagos; (e) `"estado":"Devuelta"` con pagos; (f) `"estado":"Completada"` sin pagos.
**Resultado esperado:** (a) se cobra («Completada»); (b) queda en espera («Pendiente»); (c) se cobra de inmediato como «Completada»; (d) y (e) se rechazan; (f) lo rechaza el validador (`CreateVentaCommand.Pagos`). Los `code` exactos de (d) y (e): **No verificado**.

### HU-VEN-016 — Cobrar solo con RUC de la empresa y serie de boleta activa

#### TC-VEN-124 — La empresa no tiene RUC

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-016] «La empresa no tiene RUC»; [CU-VEN-16] flujo básico; [RN-VEN-007], [RN-CMP-002], [RN-EMP-003], [RN-EMP-004] |

**Precondiciones:** una empresa de prueba aparte («Botica Sin RUC Demo S.A.C.») a la que se le quitó el RUC al editarla; su cajero con caja abierta.
**Pasos:** arma una venta de S/ 0.50 en efectivo y cobra.
**Resultado esperado:** «Tu empresa no tiene RUC registrado, por lo que no puede emitir comprobantes. Complétalo en Administración > Empresa para poder cobrar.»; no se guarda la venta ni se consume ningún correlativo de la serie.

#### TC-VEN-125 — La sucursal sin serie de boletas activa (o con la serie inactiva)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-016] «La sucursal no tiene serie de boletas activa», «Una serie inactiva no sirve»; [CU-VEN-16] flujo básico; [RN-VEN-007], [RN-CMP-003], [RN-SER-008] |

**Precondiciones:** una sucursal de prueba sin serie de tipo Boleta; otra cuya única serie de boletas está «Inactiva».
**Pasos:** en cada una, arma una venta y cobra.
**Resultado esperado:** en ambas «Esta sucursal no tiene una serie de boletas activa. Configúrala en SUNAT > Series Fiscales para poder cobrar.»; no se guarda la venta ni se consume correlativo.

#### TC-VEN-126 — Orden de las comprobaciones: primero el RUC

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-016] «Orden de las comprobaciones»; [CU-VEN-16] flujo básico; [RN-VEN-007] |

**Precondiciones:** empresa de prueba sin RUC y sin serie de boletas activa en la sucursal.
**Pasos:** cobra una venta.
**Resultado esperado:** se ve primero el aviso del RUC («Tu empresa no tiene RUC registrado, por lo que no puede emitir comprobantes. Complétalo en Administración > Empresa para poder cobrar.»); al completar el RUC, aparece el de la serie.

#### TC-VEN-127 — Dejar la venta en espera no necesita RUC ni serie

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-016] «Dejar la venta en espera no necesita RUC ni serie»; [CU-VEN-16] alternativo A1; [RN-VEN-007] |

**Precondiciones:** empresa de prueba sin RUC (o sucursal sin serie activa).
**Pasos:** arma una venta y haz clic en «Guardar y Atender Siguiente».
**Resultado esperado:** la venta se guarda en espera; el rechazo ocurre recién al cobrarla (`TC-VEN-124`).

### HU-VEN-017 — Emitir la boleta al cobrar

#### TC-VEN-128 — La primera boleta de la serie

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-017] «La primera boleta de la serie»; [CU-VEN-17] flujo básico; [RN-VEN-059], [RN-CMP-001], [RN-CMP-004] |

**Precondiciones:** serie `B001` con correlativo inicial 0 en la sucursal; caja abierta.
**Pasos:** cobra la primera venta con esa serie.
**Resultado esperado:** boleta `B001-00000001`; el diálogo muestra «Nº Comprobante:» «Boleta B001-00000001», «Estado SUNAT:» «Pendiente de envío a SUNAT» y el subtítulo «Comprobante registrado. Pendiente de envío a SUNAT.». La siguiente venta emite `B001-00000002`.

#### TC-VEN-129 — Importes de la boleta y el desglose que suma el total (esquema)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-VEN-017] «Importes de la boleta», «El desglose siempre suma el total»; [CU-VEN-17] flujo básico; [RN-CMP-006] |

**Datos:** ventas con total S/ 10.00, S/ 50.00 y S/ 100.00 (precios con IGV incluido, tasa 18 %).
**Pasos:** cobra cada venta y revisa los importes de la boleta en el detalle o en la base de datos.
**Resultado esperado:**

| Total | Base gravada | IGV |
|---|---|---|
| 10.00 | 8.47 | 1.53 |
| 50.00 | 42.37 | 7.63 |
| 100.00 | 84.75 | 15.25 |

El monto exonerado es 0.00 y base más IGV suman exactamente el total. La base es el total dividido entre 1,18 redondeado a 2 decimales con la mitad hacia arriba.
**Hallazgo:** los importes del ticket impreso los calcula el navegador y pueden diferir en un centavo del guardado (01c2 hallazgo 11; **No verificado** en ejecución).

#### TC-VEN-130 — Boleta sin cliente y con cliente identificado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-017] «Venta sin cliente», «Venta con cliente identificado»; [CU-VEN-17] flujo básico; [RN-CMP-005] |

**Pasos:** 1. Cobra una venta sin cliente. 2. Cobra una venta eligiendo a «Rosa Quispe Demo» (DNI `45781236`).
**Resultado esperado:** paso 1: la boleta va a nombre de «CLIENTE EVENTUAL», tipo de documento «-» y número «00000000». Paso 2: la boleta lleva su tipo de documento (código SUNAT «1» para DNI), su número de documento y su nombre y apellidos.

#### TC-VEN-131 — Cliente con documento largo o con RUC

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-VEN-017] «Cliente con documento de más de 20 caracteres», «Aunque el cliente tenga RUC, la venta emite boleta»; [CU-VEN-17] flujo básico; [RN-CMP-001], [RN-CMP-005] |

**Precondiciones:** un paciente con número de documento de más de 20 caracteres; un paciente con tipo de documento RUC.
**Pasos:** vende a cada uno.
**Resultado esperado:** con el documento de más de 20 caracteres la boleta va al cliente eventual; con el cliente RUC el comprobante sigue siendo una boleta (no se emite factura).

#### TC-VEN-132 — Una venta en espera no tiene comprobante y el código de venta cambia al cobrarla

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-017] «Una venta en espera no tiene comprobante», «El código de venta es el número de la boleta»; [CU-VEN-17] alternativo A1; [RN-CMP-001], [RN-VEN-059] |

**Pasos:** 1. Deja una venta en espera y anota su código en `/pos`. 2. Cóbrala. 3. Anula otra venta cobrada y revisa su código.
**Resultado esperado:** paso 1: sin boleta y sin consumir correlativo; el código son los primeros 8 caracteres del identificador, en mayúsculas. Paso 2: el código pasa a ser el número de la boleta (por ejemplo `B001-00000003`). Paso 3: la venta anulada conserva el código de su boleta.

#### TC-VEN-133 — Los datos del emisor no se guardan en la boleta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-017] «Los datos del emisor no se guardan en la boleta»; [CU-VEN-17] flujo básico; [RN-CMP-011] |

**Precondiciones:** una venta cobrada; una empresa de prueba cuyo nombre se pueda editar.
**Pasos:** cambia la razón social de la empresa (o la dirección de la sucursal) y reimprime el comprobante de la venta anterior (`TC-VEN-138`).
**Resultado esperado:** la boleta anterior muestra los datos nuevos al reimprimirse.
**Hallazgo:** el emisor no se guarda en la boleta (HU-VEN-017).

#### TC-VEN-134 — Un número fiscal no se repite

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-VEN-017] «Un número fiscal no se repite»; [CU-VEN-17] excepción (`Db.Duplicado`); [RN-CMP-004], [RN-CMP-009] |

**Pasos:** lanza dos cobros simultáneos en la misma serie desde dos cajeros y revisa los números emitidos.
**Resultado esperado:** los números de boleta son distintos; si dos cobros obtuvieran el mismo número, el segundo guardado se rechaza con `Db.Duplicado` (HTTP 409) y se ve «Ocurrió un error al procesar la venta.». **No verificado en ejecución:** el correlativo se incrementa con un contador atómico y el caso esperado es que no se repita.

#### TC-VEN-135 — Estructura de la boleta y estado del comprobante

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-017] «Estructura de la boleta», «Estado del comprobante»; [CU-VEN-17] flujo básico; [RN-CMP-007], [RN-CMP-008], [RN-CMP-010], [RN-CMP-012] |

**Pasos:** 1. Revisa en el detalle de una venta cobrada el estado del comprobante. 2. En la base de datos revisa la fila de la tabla de comprobantes de esa venta.
**Resultado esperado:** el comprobante nace como `NoEnviado` y se muestra «Pendiente de envío a SUNAT»; los estados «Aceptado por SUNAT» y «Rechazado por SUNAT» existen pero nada los asigna; «Anulado» lo asigna la anulación de la venta (`TC-VEN-173`). La fila tiene venta, serie, correlativo, tipo y número de documento del cliente, razón social, los cuatro importes y el estado.
**Hallazgo:** el estado por defecto del script de base de datos es `Pendiente` mientras el código escribe `NoEnviado`; el SPA los traduce igual (01c2 hallazgo 13). No existe envío a SUNAT (RN-CMP-007).

### HU-VEN-018 — Imprimir y reimprimir el comprobante

#### TC-VEN-136 — Qué muestra el diálogo tras cobrar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-018] «Qué muestra el diálogo tras cobrar», «Nueva venta o volver»; [CU-VEN-18] flujo básico; [RN-CMP-014] |

**Pasos:** 1. Cobra una venta. 2. Intenta cerrar el diálogo con Esc y haciendo clic fuera. 3. Haz clic en «Nueva Venta POS»; repite otra venta y haz clic en «Volver a Caja POS».
**Resultado esperado:** se lee «Venta Procesada Exitosamente» y «Comprobante registrado. Pendiente de envío a SUNAT.», con «Nº Comprobante:», «Estado SUNAT:», «Total Pagado:» y «Medio de Pago:», y los botones «Nueva Venta POS» (solo con `Ventas:Crear`), «Imprimir Comprobante» y «Volver a Caja POS». No se cierra con Esc ni haciendo clic fuera. «Nueva Venta POS» deja el terminal limpio en `/pos/nueva`; «Volver a Caja POS» lo limpia y va a `/pos`.

#### TC-VEN-137 — Imprimir el ticket y su contenido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-018] «Imprimir el ticket», «Contenido del ticket», «Línea vendida por presentación», «El cliente eventual no imprime documento»; [CU-VEN-18] flujo básico; [RN-CMP-014] |

**Precondiciones:** una venta cobrada con una línea por presentación («1 × Caja x10») y otra venta al cliente eventual.
**Pasos:** haz clic en «Imprimir Comprobante» y revisa la vista previa; repite con la venta al cliente eventual y haz clic otra vez.
**Resultado esperado:** se abre el diálogo de impresión del navegador con un ticket de rollo de 80 mm; no se imprime solo, se repite cada vez que haces clic. El ticket muestra la razón social, «RUC:», la dirección de la sucursal, el tipo y número de la boleta y la fecha «dd/MM/yyyy HH:mm»; la tabla «Producto / Lote», «Cant.», «P. Unit.» y «Subtotal»; «Datos del Cliente:», «Documento:», «Subtotal (Op. Gravada):», «IGV (18%):» y «TOTAL A PAGAR:»; una fila por pago y «Vuelto:» solo si es mayor que 0; «Estado SUNAT:» y «Cajero:». La línea por presentación dice «1 × Caja (10)» con el precio por presentación. Con cliente eventual el ticket no imprime «Documento:».
**Hallazgo:** el ticket no incluye QR, hash ni leyenda legal (19f §5.2); la columna del importe se rotula «Subtotal».

#### TC-VEN-138 — Reimprimir desde el detalle de la venta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-018] «Reimprimir desde el detalle de la venta», «Datos del emisor al reimprimir»; [CU-VEN-18] alternativo A1; [RN-CMP-011], [RN-CMP-012], [RN-CMP-014] |

**Pasos:** abre `/pos/{id}` de una venta cobrada con comprobante y haz clic en «Reimprimir Comprobante».
**Resultado esperado:** se abre el mismo diálogo con los datos de la venta guardada; el ticket usa la fecha de la venta y el nombre del cajero que la hizo; el vuelto se recalcula como pagos menos total de la venta; razón social, RUC y dirección se leen de la empresa y la sucursal en ese momento.
**Hallazgo:** la reimpresión muestra el mismo título «Venta Procesada Exitosamente» (HU-VEN-018).

#### TC-VEN-139 — Una venta anulada no se reimprime

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-018] «Una venta anulada no se reimprime»; [CU-VEN-18] alternativo A2; [RN-CMP-012] |

**Pasos:** abre el detalle de una venta «Anulada».
**Resultado esperado:** el detalle no ofrece «Reimprimir Comprobante».

### HU-VEN-019 — Cubrir parte de la venta con una aseguradora

#### TC-VEN-140 — La cobertura no se ofrece en pantalla

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-019] «La cobertura no se ofrece en pantalla»; [CU-VEN-19] flujo básico |

**Pasos:** abre el terminal con Admin y con Carlos; revisa todo el panel de pago.
**Resultado esperado:** no existe el bloque «Cobertura de Seguro (opcional)»; ningún rol, empresa ni configuración puede mostrarlo (Parcial: el bloque está oculto por una constante).

#### TC-VEN-141 — Cobrar con cobertura por la API

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-019] «Cobrar con cobertura por la API»; [CU-VEN-19] flujo básico; [RN-VEN-047] |

**Precondiciones:** «Aseguradora Demo» registrada; caja `S1` abierta con apertura S/ 100.00; productos que sumen S/ 100.00 (por ejemplo 200 unidades de `LOT-QA-P01`).
**Pasos:** `POST /api/v1/ventas` con las 200 unidades, `"aseguradoraId":"<Aseguradora Demo>"`, `"montoCubiertoSeguro":40.00` y `"pagos":[{"metodoPago":"Efectivo","montoPagado":100.00}]`.
**Resultado esperado:** el monto a pagar es S/ 60.00; los pagos solo tienen que cubrir S/ 60.00; como el cliente pagó S/ 100.00 en efectivo, el vuelto es S/ 40.00 y el efectivo neto de la venta es S/ 60.00 (el arqueo suma 60.00, ver `TC-CAJ-031`). Con `montoPagado` 59.99 la petición se rechaza (`Venta.Pagos`).

#### TC-VEN-142 — Se crea un reclamo de seguro

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-019] «Se crea un reclamo de seguro»; [CU-VEN-19] flujo básico; [RN-VEN-048] |

**Pasos:** tras `TC-VEN-141`, consulta en la base de datos el reclamo de seguro de la venta; repite con `montoCubiertoSeguro` 0 y aseguradora.
**Resultado esperado:** existe un reclamo ligado a la primera línea de la venta, con estado «Aprobado», el monto cubierto (40.00), el copago del paciente (total menos cobertura, 60.00) y un código de autorización «AUTH_» más 8 caracteres hexadecimales generados por el sistema; con monto 0 también se crea el reclamo. La boleta se emite por el total completo (S/ 100.00).
**Hallazgo:** el código de autorización lo inventa el sistema, el estado siempre es «Aprobado» y nada impide dos reclamos para la misma línea (01c2 hallazgo 5).

#### TC-VEN-143 — Monto cubierto fuera de rango

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-VEN-019] «El monto cubierto está fuera de rango»; [CU-VEN-19] excepción; [RN-VEN-047] |

**Pasos:** `POST /api/v1/ventas` por S/ 100.00 con aseguradora válida y `montoCubiertoSeguro` -1; luego 100.01; luego 100.00; luego 0.
**Resultado esperado:** -1 y 100.01 se rechazan y el SPA mostraría «El monto cubierto por el seguro debe estar entre 0 y el total de la venta.»; 100.00 y 0 se aceptan (con 100.00 el monto a pagar es S/ 0.00; la aceptación del pago en 0 es **No verificado**).

#### TC-VEN-144 — Monto cubierto sin aseguradora y aseguradora inexistente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-019] «Monto sin aseguradora», «La aseguradora no existe»; [CU-VEN-19] excepciones; [RN-VEN-047] |

**Pasos:** 1. `POST /api/v1/ventas` con `montoCubiertoSeguro` 10.00 y sin `aseguradoraId`. 2. Repite con un `aseguradoraId` eliminado o de otra empresa («Botica Otra Demo S.A.C.»).
**Resultado esperado:** paso 1: «Selecciona una aseguradora para aplicar la cobertura del seguro.». Paso 2: HTTP 404 y «La aseguradora seleccionada no existe o fue eliminada.». No se crea la venta.

#### TC-VEN-145 — Anular la venta no toca el reclamo; estructura del reclamo; borrador antiguo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-019] «Anular la venta no toca el reclamo», «Estructura del reclamo de seguro», «Un borrador antiguo con aseguradora»; [CU-VEN-19] alternativo A1 |

**Pasos:** 1. Anula la venta de `TC-VEN-141` (con caja abierta). 2. Consulta el reclamo y su tabla. 3. Si dispones de un borrador antiguo con aseguradora en `localStorage`, restáuralo.
**Resultado esperado:** paso 1-2: el reclamo queda como estaba (estado «Aprobado»); la tabla de reclamos no tiene restricciones CHECK ni índice único. Paso 3: la cobertura reduce el «TOTAL A PAGAR» sin forma visible de quitarla. **No verificado en ejecución** (el borrador caduca a las 12 horas).

### HU-VEN-020 — Guardar la venta en espera

#### TC-VEN-146 — Dejar una venta en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-020] «Dejar una venta en espera»; [CU-VEN-20] flujo básico; [RN-VEN-003], [RN-VEN-004], [RN-VEN-049], [RN-VEN-051] |

**Precondiciones:** caja abierta de Carlos; `LOT-QA-P01` con 500 unidades.
**Pasos:** agrega 100 unidades de «Paracetamol Demo 500 mg» y haz clic en «Guardar y Atender Siguiente».
**Resultado esperado:** «Venta guardada en espera. Retómala desde Ventas cuando el cliente vuelva.»; el terminal queda limpio para el siguiente cliente; en `/pos` la venta aparece en estado «Pendiente».

#### TC-VEN-147 — Qué reserva la venta en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-020] «Qué reserva la venta en espera»; [CU-VEN-20] alternativo A1; [RN-VEN-015], [RN-VEN-052] |

**Precondiciones:** `LOT-QA-P01` con 500 unidades; venta en espera de 100 unidades de `TC-VEN-146`.
**Pasos:** revisa el stock de `LOT-QA-P01` en «Stock por Sucursal» y, si la venta lleva una receta de producto controlado, intenta una segunda venta con esa receta.
**Resultado esperado:** el stock del lote baja de 500 a 400 desde que se guarda; la venta cuenta como dispensación de la receta que use (ver `TC-VEN-076`) y queda ligada a la caja con que se creó.

#### TC-VEN-148 — Qué no tiene una venta en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-020] «Qué no tiene una venta en espera»; [CU-VEN-20] flujo básico; [RN-VEN-052], [RN-VEN-059], [RN-VEN-060] |

**Pasos:** tras `TC-VEN-146`, abre el detalle de la venta en `/pos/{id}`; revisa en la base de datos pagos, comprobante, correlativo de la serie y eventos de la bandeja de salida.
**Resultado esperado:** no tiene pagos, ni boleta, ni correlativo (la serie no avanza); no exige «Receta verificada», RUC ni serie de boletas; no genera `VentaCompletadaEvent`; su código de venta son los 8 primeros caracteres del identificador, en mayúsculas.

#### TC-VEN-149 — No se pide receta, pagos ni número de operación al dejar en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-020] «No se pide receta, pagos ni número de operación»; [CU-VEN-20] flujo básico |

**Pasos:** 1. Con `LOT-QA-C01` en el carrito, sin receta ni pagos, haz clic en «Guardar y Atender Siguiente». 2. Con el carrito vacío mira el botón.
**Resultado esperado:** paso 1: la venta se guarda. Paso 2: el botón se deshabilita solo con el carrito vacío, sin caja o mientras se procesa.

#### TC-VEN-150 — Sin caja abierta no se puede dejar en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-020] «Sin caja abierta no se puede dejar en espera»; [CU-VEN-20] excepción (`Venta.Caja`); [RN-VEN-003], [RN-VEN-004] |

**Pasos:** 1. Sin caja abierta abre el terminal. 2. Por API: `POST /api/v1/ventas` con `"estado":"Pendiente"`, `"pagos":[]` y un `sesionId` de una caja cerrada.
**Resultado esperado:** paso 1: no hay botón; solo el aviso «Debes aperturar la caja del turno antes de procesar una venta.». Paso 2: HTTP 400, `code` `Venta.Caja`; el SPA mostraría «Ocurrió un error al procesar la venta.».

#### TC-VEN-151 — Falla al guardar la venta en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-020] «Falla al guardar»; [CU-VEN-20] excepciones |

**Pasos:** deja en espera una venta con más unidades de las que tiene el lote (por API, por ejemplo 31 unidades de `LOT-QA-P02` con 30 en stock).
**Resultado esperado:** `Venta.Stock` (HTTP 400); el SPA muestra «Ocurrió un error al procesar la venta.»; no se guarda nada.

#### TC-VEN-152 — La venta en espera no caduca

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-020] «La venta en espera no caduca»; [CU-VEN-20] alternativo A2; [RN-VEN-052] |

**Pasos:** deja una venta en espera y compruébala varios días después (o cambia la fecha de creación en la base de datos de prueba).
**Resultado esperado:** sigue «Pendiente», con su stock y su dispensación reservados; solo se cobra o se anula.
**Hallazgo:** no hay vencimiento ni proceso que la libere (01c2 hallazgo 7).

### HU-VEN-021 — Retomar y modificar una venta en espera

#### TC-VEN-153 — Encontrar la venta en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-021] «Encontrar la venta en espera»; [CU-VEN-21] flujo básico; [CU-VEN-26] alternativo A3; [RN-VEN-053] |

**Precondiciones:** una venta en espera de Carlos y otra de otro cajero de la misma sucursal.
**Pasos:** en `/pos` elige «Estado» = «Pendiente» y haz clic en «Buscar».
**Resultado esperado:** aparecen las ventas en espera de la sucursal, de cualquier cajero, y puedes abrir su código.

#### TC-VEN-154 — Retomarla desde el detalle

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-021] «Retomarla desde el detalle»; [CU-VEN-21] flujo básico; [RN-VEN-053] |

**Precondiciones:** venta en espera con cliente «Rosa Quispe Demo» y una línea que usa una receta; Carlos con `Ventas:Crear` y `Ventas:Actualizar`.
**Pasos:** abre el detalle de la venta «Pendiente» y haz clic en «Cobrar Venta».
**Resultado esperado:** el terminal carga las líneas, el cliente y la receta de la primera línea que tenga una; aviso «Estás retomando una venta en espera: puedes agregar, quitar o modificar productos antes de cobrar.»; las líneas cargadas conservan el precio guardado y las nuevas usan el del catálogo.

#### TC-VEN-155 — La existencia incluye lo que la venta ya reservó

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-VEN-021] «La existencia incluye lo que la venta ya reservó»; [CU-VEN-21] flujo básico; [RN-VEN-055] |

**Precondiciones:** `LOT-QA-P02` con 30 unidades de stock; una venta en espera que reservó 100 unidades de `LOT-QA-P01` y deja stock vendible actual de 400.
**Pasos:** retoma la venta y sube la línea de `LOT-QA-P01` con «+» hasta el máximo.
**Resultado esperado:** puedes subir esa línea hasta el stock vendible actual (400) más las 100 unidades reservadas (500); con una más aparece «Stock máximo disponible alcanzado (500 un.).».

#### TC-VEN-156 — Sin los dos permisos no aparece «Cobrar Venta»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-VEN-021] «Sin los dos permisos no aparece el botón»; [CU-VEN-21] alternativo A2; [RN-VEN-001] |

**Pasos:** con un rol sin `Ventas:Actualizar` (y otro sin `Ventas:Crear`), abre el detalle de una venta «Pendiente».
**Resultado esperado:** el detalle no muestra «Cobrar Venta».
**Hallazgo:** la ruta `/pos/nueva?continuar=` solo exige `Ventas:Leer` y `Ventas:Crear`: quien escribe la URL sin `Ventas:Actualizar` abre la pantalla y el servidor responde 403 al guardar o cobrar (19f §0.2).

#### TC-VEN-157 — Cualquier usuario de la misma sucursal puede retomarla

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-021] «Cualquier usuario de la misma sucursal puede retomarla»; [CU-VEN-21] flujo básico; [RN-VEN-054] |

**Pasos:** un segundo cajero de «Sede Principal» con los permisos necesarios abre y retoma la venta en espera de Carlos y la modifica.
**Resultado esperado:** puede retomarla y modificarla (no hace falta ser quien la guardó). Un cajero de «Sucursal Demo Norte» no la ve en su listado.

#### TC-VEN-158 — Guardar los cambios de una venta en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-021] «Guardar los cambios»; [CU-VEN-21] flujo básico; [RN-VEN-054], [RN-VEN-055] |

**Precondiciones:** venta en espera con 100 unidades de `LOT-QA-P01` (stock actual 400).
**Pasos:** retómala, quita esa línea, agrega 50 unidades de `LOT-QA-A01` y haz clic en «Actualizar Venta en Espera».
**Resultado esperado:** «Venta guardada en espera. Retómala desde Ventas cuando el cliente vuelva.»; el terminal se limpia y navega a `/pos/nueva` sin `continuar`; el stock de `LOT-QA-P01` vuelve a 500 y el de `LOT-QA-A01` baja en 50; el total se recalcula; no se pidió caja abierta ni cambió la caja de la venta.

#### TC-VEN-159 — Las líneas nuevas se validan como al crear

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-021] «Las líneas se validan como al crear»; [CU-VEN-21] flujo básico; [RN-VEN-054] |

**Pasos:** por API, `PUT /api/v1/ventas/{id}/detalles` con una línea de `LOT-QA-P03` (vencido) y otra con `recetaId` de otro paciente.
**Resultado esperado:** el servidor rechaza con los mismos mensajes que al cobrar («Uno de los lotes está vencido y no se puede vender. Retíralo del carrito.» y «La receta médica no pertenece al cliente de la venta.»), sin pedir «Receta verificada»; la venta conserva sus líneas anteriores.

#### TC-VEN-160 — La venta ya no está en espera o no se puede cargar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-021] «La venta ya no está en espera», «No se puede cargar la venta»; [CU-VEN-21] alternativo A3 |

**Pasos:** 1. Cobra una venta en espera y abre `/pos/nueva?continuar={id}` con su identificador. 2. Abre `/pos/nueva?continuar=` con el identificador de una venta de «Sucursal Demo Norte» (o inexistente).
**Resultado esperado:** paso 1: «Esta venta ya no está en espera; te llevamos a su detalle.» y pasas al detalle. Paso 2: «Error al cargar la venta».

#### TC-VEN-161 — Actualizar una venta de otra sucursal o no pendiente (API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-021] «Actualizar una venta de otra sucursal o no pendiente (la API)»; [CU-VEN-21] excepciones; [RN-VEN-054] |

**Pasos:** `PUT /api/v1/ventas/{id}/detalles` (token de Carlos) con una venta «Completada» y otra de «Sucursal Demo Norte».
**Resultado esperado:** HTTP 400, `code` `Venta.Actualizar` (en ambos casos); el SPA mostraría «Ocurrió un error al procesar la venta.». No cambia ninguna venta ni el stock.

### HU-VEN-022 — Cobrar una venta en espera

#### TC-VEN-162 — Cobrar la venta en espera con mi caja

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-022] «Cobrar la venta en espera con mi caja»; [CU-VEN-22] flujo básico; [RN-VEN-006], [RN-VEN-046], [RN-VEN-056] |

**Precondiciones:** venta en espera de S/ 50.00 guardada por otro cajero con su caja `S2`; Carlos con su caja `S1` abierta (apertura S/ 100.00).
**Pasos:** 1. Carlos retoma la venta. 2. Cubre el pago en efectivo S/ 50.00 y confirma con «Confirmar y Procesar Venta». 3. Revisa el arqueo de su caja al cerrarla (ver `TC-CAJ-028`).
**Resultado esperado:** la venta pasa a «Completada» y se emite la boleta; se reasigna a la caja de Carlos (`S1`) aunque se haya guardado con otra; el efectivo cobrado cuenta en el arqueo de `S1`; el empleado que registró la venta no cambia; se abre «Venta Procesada Exitosamente» con el nuevo número de boleta.

#### TC-VEN-163 — No tengo caja propia abierta al cobrar una venta en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-022] «No tengo caja propia abierta»; [CU-VEN-22] excepciones; [RN-VEN-006] |

**Precondiciones:** venta en espera; el cajero que cobra tiene su caja cerrada. **Nota:** el SPA no deja abrir el terminal sin caja, por lo que el rechazo se dispara por API.
**Pasos:** `PUT /api/v1/ventas/{id}/completar` con token del cajero sin caja abierta y pagos suficientes.
**Resultado esperado:** rechazo con el mensaje «Necesitas tener tu propia caja abierta en esta sucursal para cobrar la venta.»; la comprobación va antes que las de líneas, pagos y comprobante; la venta sigue «Pendiente».

#### TC-VEN-164 — Al cobrar con líneas reenviadas se valida todo otra vez

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-022] «Las líneas se reenvían y se valida todo otra vez»; [CU-VEN-22] flujo básico; [RN-VEN-056] |

**Precondiciones:** venta en espera con un lote que luego se vence (cambia su caducidad en base de datos) o se aísla en cuarentena.
**Pasos:** retoma la venta y cóbrala.
**Resultado esperado:** el servidor aplica de nuevo cuarentena, vencimiento, stock, precio, receta y dispensaciones como en una venta nueva (por ejemplo, «Uno de los lotes está vencido y no se puede vender. Retíralo del carrito.»); la venta sigue «Pendiente».

#### TC-VEN-165 — Cobrar sin reenviar las líneas (solo API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-022] «Cobrar sin reenviar las líneas (solo API)»; [CU-VEN-22] alternativo A1; [RN-VEN-016], [RN-VEN-038] |

**Precondiciones:** venta en espera cuyo lote ya venció o está en cuarentena.
**Pasos:** `PUT /api/v1/ventas/{id}/completar` con `{"pagos":[…suficientes…]}` sin `detalles`.
**Resultado esperado:** se conservan las líneas guardadas y la venta se cobra sin volver a revisar cuarentena, vencimiento, stock, precio ni dispensaciones; solo se vuelve a juzgar la política de receta y que el lote y el producto existan.
**Hallazgo:** el cobro sin reenviar líneas no revalida vencimiento, cuarentena, stock, paciente de la receta ni dispensaciones (01c2 hallazgo 10); el SPA siempre reenvía las líneas.

#### TC-VEN-166 — La empresa pasó a exigir registro de receta mientras la venta esperaba

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-022] «La empresa pasó a exigir registro de receta mientras la venta esperaba»; [CU-VEN-22] flujo básico; [RN-VEN-038] |

**Precondiciones:** venta en espera con Amoxicilina sin receta registrada en una empresa de prueba flexible; luego la empresa se pasa a modo estricto por base de datos.
**Pasos:** cobra la venta sin reenviar las líneas (API).
**Resultado esperado:** el servidor la rechaza; en el SPA «Uno de los productos requiere receta médica. Selecciona el cliente y una de sus recetas registradas.».

#### TC-VEN-167 — Reenviar una lista de líneas vacía (solo API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-022] «Reenviar una lista de líneas vacía (solo API)»; [CU-VEN-22] flujo básico; [RN-VEN-008] |

**Pasos:** `PUT /api/v1/ventas/{id}/completar` con `"detalles":[]`.
**Resultado esperado:** `Venta.Detalles`; el SPA mostraría «Ocurrió un error al procesar la venta.».

#### TC-VEN-168 — Falta «Receta verificada» al cobrar una venta en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-022] «Falta "Receta verificada"»; [CU-VEN-22] flujo básico; [RN-VEN-034] |

**Precondiciones:** empresa flexible; venta en espera con Amoxicilina sin receta registrada.
**Pasos:** retómala y trata de cobrar sin marcar la casilla (por API con `"recetaVerificada":false`).
**Resultado esperado:** «Marca "Receta verificada" para confirmar que revisaste la receta del cliente antes de cobrar.».

#### TC-VEN-169 — La venta ya no está pendiente o es de otra sucursal al cobrar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-022] «La venta ya no está pendiente, o es de otra sucursal»; [CU-VEN-22] excepciones; [RN-VEN-056] |

**Pasos:** `PUT /api/v1/ventas/{id}/completar` con una venta ya «Completada» y con una de «Sucursal Demo Norte».
**Resultado esperado:** HTTP 400, `code` `Venta.Completar`; el SPA mostraría «Ocurrió un error al procesar la venta.».

#### TC-VEN-170 — Dos cajeros cobran la misma venta a la vez

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-VEN-022] «Dos cajeros cobran la misma venta a la vez»; [CU-VEN-22] excepciones; [RN-VEN-057] |

**Pasos:** dos cajeros con caja abierta parten de la misma venta en espera y confirman al mismo tiempo (dos clientes HTTP en paralelo).
**Resultado esperado:** solo se guarda un cobro y el otro ve «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (HTTP 409); solo se emite una boleta y se descuenta una sola vez en el arqueo. **No verificado en ejecución.**

#### TC-VEN-171 — Sin RUC o sin serie al cobrar una venta en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-022] «Sin RUC o sin serie de boletas»; [CU-VEN-22] flujo básico; [RN-VEN-007] |

**Precondiciones:** venta en espera en una sucursal cuya serie de boletas se desactivó.
**Pasos:** retoma y cobra.
**Resultado esperado:** «Esta sucursal no tiene una serie de boletas activa. Configúrala en SUNAT > Series Fiscales para poder cobrar.» (igual que `TC-VEN-125`); la venta sigue «Pendiente».

### HU-VEN-023 — Anular una venta cobrada

#### TC-VEN-172 — Anular una venta cobrada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-023] «Anular una venta cobrada»; [CU-VEN-23] flujo básico; [RN-VEN-062], [RN-VEN-063], [RN-VEN-067] |

**Precondiciones:** venta `B001-00000001` «Completada», sin devoluciones y de «Sede Principal»; Carlos con `Ventas:Anular` y su caja abierta.
**Pasos:** 1. Abre el detalle y haz clic en «Anular Venta». 2. Lee el diálogo. 3. Escribe «Error en el cobro» en «Motivo de Anulación» y haz clic en «Anular Venta».
**Resultado esperado:** el diálogo «Anular Venta» dice «Esta acción revierte el stock descontado. Ingresa el motivo de la anulación.»; al confirmar aparece «Venta anulada correctamente.» y el detalle se recarga con estado «Anulada».

#### TC-VEN-173 — Qué se revierte y qué queda registrado al anular

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-023] «Qué se revierte y qué queda registrado»; [CU-VEN-23] flujo básico; [RN-VEN-061], [RN-VEN-068], [RN-VEN-069], [RN-CMP-012] |

**Precondiciones:** venta de 100 unidades de `LOT-QA-P01` cobrada (stock del lote 400 tras la venta).
**Pasos:** anula la venta (`TC-VEN-172`); revisa el stock, la boleta, la venta y la tabla de auditoría.
**Resultado esperado:** el stock de `LOT-QA-P01` vuelve de 400 a 500 (si la fila de stock ya no existe, esa línea se omite sin aviso); la boleta pasa a «Anulado»; la venta queda «Anulada» con su motivo, la caja desde la que se anuló y el usuario y fecha; hay un evento de auditoría `VENTA_ANULAR` con detalle «estado: Anulada»; los pagos y el reclamo de seguro quedan como estaban, sin filas de reverso; no se informa a SUNAT.
**Hallazgo:** la anulación no crea fila de reverso de pagos (01c2 hallazgo 19).

#### TC-VEN-174 — El detalle de la venta anulada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-023] «El detalle de la venta anulada»; [CU-VEN-23] flujo básico; [RN-VEN-069], [RN-CMP-012] |

**Pasos:** abre el detalle de la venta anulada de `TC-VEN-172`; intenta una nueva venta con la receta que usó (si la usó).
**Resultado esperado:** el detalle muestra «Motivo de Anulación:» y el estado «Anulada»; no ofrece «Reimprimir Comprobante»; la dispensación de la receta usada queda liberada.

#### TC-VEN-175 — La anulación necesita mi propia caja abierta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-023] «La anulación necesita mi propia caja abierta»; [CU-VEN-23] excepciones; [RN-VEN-067], [RN-CAJ-012] |

**Precondiciones:** venta «Completada» de «Sede Principal»; el usuario que anula (con `Ventas:Anular`) no tiene caja abierta.
**Pasos:** abre el detalle, haz clic en «Anular Venta», escribe un motivo y confirma.
**Resultado esperado:** «Necesitas tener tu propia caja abierta en esta sucursal para anular una venta cobrada.»; la venta sigue «Completada» y el stock no cambia.

#### TC-VEN-176 — Efecto en el arqueo cuando se anula desde otra caja

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-023] «Efecto en el arqueo cuando se anula desde otra caja»; [CU-VEN-23] alternativo A1; [RN-VEN-067], [RN-CAJ-012] |

**Precondiciones:** una venta de S/ 20.00 cobrada en efectivo en la caja A (de otro cajero); Carlos con su caja B abierta (apertura S/ 100.00).
**Pasos:** 1. Carlos anula esa venta. 2. Revisa el efectivo esperado de su caja B (al cerrarla, ver `TC-CAJ-032`, caso C) y de la caja A.
**Resultado esperado:** el arqueo de la caja B resta S/ 20.00 (el efectivo sale del cajón de quien devuelve); la caja A sigue contando la venta mientras no se cierre (si se cierra después de la anulación, el sistema calcula con los datos del momento).

#### TC-VEN-177 — Anular cualquier venta de mi sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-VEN-023] «Anular cualquier venta de mi sucursal»; [CU-VEN-23] flujo básico; [RN-VEN-062] |

**Pasos:** un usuario con `Ventas:Anular` y su caja abierta anula una venta hecha por otro cajero de la misma sucursal.
**Resultado esperado:** la anulación se realiza; no hace falta ser el cajero que la vendió.

#### TC-VEN-178 — El motivo de anulación es obligatorio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-023] «El motivo es obligatorio»; [CU-VEN-23] excepciones; [RN-VEN-063] |

**Pasos:** en el diálogo «Anular Venta» deja «Motivo de Anulación» vacío y haz clic en «Anular Venta».
**Resultado esperado:** «El motivo es obligatorio.» y el diálogo no se cierra.

#### TC-VEN-179 — Otra operación modificó la venta o el stock al anular

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-VEN-023] «Otra operación modificó la venta o el stock»; [CU-VEN-23] excepciones |

**Pasos:** anula una venta mientras otro cajero cobra o modifica el mismo stock al mismo tiempo (dos clientes HTTP en paralelo).
**Resultado esperado:** si se detecta el conflicto: «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.». **No verificado en ejecución.**

#### TC-VEN-180 — Sin el permiso no hay botón «Anular Venta»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-VEN-023] «Sin el permiso no hay botón»; [CU-VEN-23] alternativo A2; [RN-VEN-062] |

| Permiso que falta | Qué no se ve o qué responde |
|---|---|
| `Ventas:Anular` | El detalle de una venta «Completada» o «Pendiente» no muestra «Anular Venta»; por API `PUT /api/v1/ventas/{id}/anular` responde 403 |

**Pasos:** con el rol «Cajero sin dinero» abre el detalle de una venta «Completada» y repite por API con su token.
**Resultado esperado:** según la tabla; la venta no cambia.

### HU-VEN-024 — Anular una venta en espera

#### TC-VEN-181 — Anular una venta en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-VEN-024] «Anular una venta en espera»; [CU-VEN-24] flujo básico; [RN-VEN-052], [RN-VEN-065], [RN-VEN-068], [RN-VEN-069] |

**Precondiciones:** venta «Pendiente» de 100 unidades de `LOT-QA-P01` (stock actual 400) que usa una receta.
**Pasos:** anúlala con el motivo «Cliente no volvió».
**Resultado esperado:** «Venta anulada correctamente.»; el stock reservado vuelve a la sucursal (400 a 500); la dispensación de la receta se libera; la venta queda «Anulada».

#### TC-VEN-182 — Anular una venta en espera no necesita caja ni comprobante

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-024] «No necesita caja abierta», «Una venta en espera no tiene comprobante»; [CU-VEN-24] alternativo A1; [RN-VEN-067] |

**Precondiciones:** el usuario que anula no tiene caja abierta; venta «Pendiente».
**Pasos:** anula la venta en espera.
**Resultado esperado:** la anulación se hace igual y no se guarda caja de anulación porque nunca se cobró; no hay boleta que pasar a «Anulado».

### HU-VEN-025 — Saber por qué una venta no se puede anular

#### TC-VEN-183 — La venta tiene devoluciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-025] «La venta tiene devoluciones»; [CU-VEN-25] flujo básico y alternativo A1; [RN-VEN-066] |

**Precondiciones:** venta con una devolución parcial (30 de 100 unidades).
**Pasos:** 1. Abre el detalle. 2. Por API `PUT /api/v1/ventas/{id}/anular` con un motivo.
**Resultado esperado:** paso 1: no se muestra «Anular Venta». Paso 2: «Esta venta tiene devoluciones registradas y ya no puede anularse. Gestiona los cambios desde Devoluciones.» (`Venta.Anular.ConDevoluciones`); la venta y el stock no cambian.
**Hallazgo:** el texto remite a «Devoluciones», que no es una pantalla: las devoluciones se procesan desde el mismo detalle.

#### TC-VEN-184 — La venta está totalmente devuelta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-025] «La venta está totalmente devuelta»; [CU-VEN-25] flujo básico; [RN-VEN-065], [RN-VEN-066] |

**Precondiciones:** venta con estado «Devuelta».
**Pasos:** abre el detalle e intenta anular por API.
**Resultado esperado:** el detalle solo permite «Reimprimir Comprobante»; por API se ve «Esta venta tiene devoluciones registradas y ya no puede anularse. Gestiona los cambios desde Devoluciones.».

#### TC-VEN-185 — La venta ya está anulada y solo se anulan Completada o Pendiente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-025] «La venta ya está anulada», «Solo se anulan ventas "Completada" o "Pendiente"»; [CU-VEN-25] flujo básico; [RN-VEN-065] |

**Pasos:** abre el detalle de una venta «Anulada» y de una «Devuelta»; intenta anular la anulada por API.
**Resultado esperado:** el detalle de la anulada no ofrece botones; el botón «Anular Venta» solo aparece en «Completada» y «Pendiente»; por API se ve «Error al anular la venta.».

#### TC-VEN-186 — La venta es de otra sucursal (también para el administrador)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-VEN-025] «La venta es de otra sucursal»; [CU-VEN-25] excepciones; [RN-VEN-062] |

**Precondiciones:** venta de «Sucursal Demo Norte»; usuarios de «Sede Principal»: Carlos y «Ana Demo Prueba» (Admin).
**Pasos:** intenta anular la venta con cada uno (por API con el token respectivo).
**Resultado esperado:** «Error al anular la venta.» para ambos; el `Admin` tampoco anula ventas de otra sucursal que la de su token; la venta no cambia.

#### TC-VEN-187 — Motivo de solo espacios

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-VEN-025] «Motivo de solo espacios»; [CU-VEN-25] excepciones; [RN-VEN-063] |

**Pasos:** en el diálogo escribe solo espacios en «Motivo de Anulación» y haz clic en «Anular Venta».
**Resultado esperado:** el diálogo lo acepta pero el servidor lo rechaza y se ve «Error al anular la venta.»; la venta no cambia.

#### TC-VEN-188 — Motivo demasiado largo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-VEN-025] «Motivo demasiado largo»; [CU-VEN-25] excepciones; [RN-VEN-063] |

**Pasos:** escribe un motivo de 255 caracteres y luego de 256 caracteres y haz clic en «Anular Venta».
**Resultado esperado:** con 255 caracteres la venta se anula; con 256 el botón «Anular Venta» no hace nada y no aparece ningún mensaje.

#### TC-VEN-189 — La venta no existe al anular

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-VEN-025] «La venta no existe»; [CU-VEN-25] excepciones; [RN-VEN-064] |

**Pasos:** `PUT /api/v1/ventas/{id}/anular` con el identificador de una venta eliminada o de otra empresa («Botica Otra Demo S.A.C.»).
**Resultado esperado:** HTTP 404, `code` `Venta.Anular`; el SPA mostraría «Error al anular la venta.».

### HU-VEN-026 — Buscar las ventas de mi sucursal

#### TC-VEN-190 — Entrar a Ventas sin resultados hasta buscar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-026] «Entrar sin resultados hasta buscar»; [CU-VEN-26] flujo básico; [RN-VEN-001] |

**Pasos:** entra a `/pos` con Carlos.
**Resultado esperado:** se ve «Ventas», los filtros «Empleado», «Cliente», «Fecha desde», «Fecha hasta» y «Estado», y ningún resultado hasta pulsar «Buscar».

#### TC-VEN-191 — Buscar las ventas de hoy

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-026] «Buscar las ventas de hoy»; [CU-VEN-26] flujo básico; [RN-VEN-072], [RN-VEN-073], [RN-GEN-008], [RN-GEN-009] |

**Precondiciones:** al menos 3 ventas de hoy en «Sede Principal» con distinta hora.
**Pasos:** elige «Fecha desde» y «Fecha hasta» con el día de hoy y haz clic en «Buscar».
**Resultado esperado:** «Resultados (N)» con las columnas «Código de Venta», «Fecha», «Cliente», «Empleado», «Total (S/)», «Estado» e «Ítems»; las fechas se interpretan como el inicio y el fin del día en tu zona horaria (una venta de las 23:30 de hoy aparece); orden por fecha y hora descendente.

#### TC-VEN-192 — Solo se ven las ventas de la sucursal de mi sesión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-VEN-026] «Solo se ven las ventas de la sucursal de mi sesión»; [CU-VEN-26] flujo básico; [RN-VEN-070] |

**Precondiciones:** ventas en «Sede Principal» y en «Sucursal Demo Norte».
**Pasos:** busca sin filtros con Carlos, con Ana (Admin, sucursal «Sede Principal») y con Nora (Norte).
**Resultado esperado:** cada usuario ve solo las ventas de la sucursal de su sesión; no hay filtro de sucursal; también vale para el administrador.

#### TC-VEN-193 — Cajero y administrador ven las mismas ventas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-026] «Cajero y administrador ven lo mismo»; [CU-VEN-26] flujo básico; [RN-VEN-071] |

**Pasos:** busca con Carlos sin filtros; luego filtra por «Empleado» = «Carlos Cajero Demo».
**Resultado esperado:** sin filtro, Carlos ve las ventas de todos los empleados de su sucursal; para ver solo las suyas filtra por «Empleado».
**Hallazgo:** no hay vista «mis ventas» (01c2 hallazgo 16).

#### TC-VEN-194 — Filtrar por empleado o cliente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-026] «Filtrar por empleado o cliente»; [CU-VEN-26] flujo básico; [RN-VEN-072] |

**Pasos:** 1. Escribe «Carlos» en «Empleado» y elige la opción; busca. 2. En «Cliente» escribe «45781236» y elige la opción; busca. 3. Escribe «Quispe» y elige. 4. Escribe un texto sin elegir ninguna opción y busca. 5. Escribe «zzzxxx» en «Cliente».
**Resultado esperado:** al elegir una opción el filtro se aplica; con solo dígitos busca por documento, con otro texto por nombre; si solo escribes el texto sin elegir, el filtro se ignora en silencio; sin coincidencias se ve la opción deshabilitada «Sin resultados».

#### TC-VEN-195 — Filtrar por estado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-026] «Filtrar por estado»; [CU-VEN-26] flujo básico; [RN-VEN-072] |

**Pasos:** abre «Estado» y elige «Pendiente»; busca.
**Resultado esperado:** las opciones son «— Todos —», «Completada», «Anulada», «Devuelta» y «Pendiente»; con «Pendiente» solo se ven las ventas en espera.

#### TC-VEN-196 — Cada fila del listado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-026] «Cada fila»; [CU-VEN-26] flujo básico; [RN-VEN-059], [RN-VEN-073] |

**Precondiciones:** una venta cobrada con boleta, una en espera y una sin cliente.
**Pasos:** busca y mira cada fila.
**Resultado esperado:** el código de venta es un enlace al detalle con la forma `B001-00000001` si hay boleta, o los 8 primeros caracteres del identificador si está en espera; el cliente es «Público General» si no hubo cliente.

#### TC-VEN-197 — Paginación y memoria de filtros

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-026] «Paginación y memoria de filtros»; [CU-VEN-26] alternativo A4; [RN-GEN-009] |

**Precondiciones:** más de 10 ventas.
**Pasos:** 1. Busca y mira el tamaño de página. 2. Cambia a 25 y 50. 3. Aplica un filtro, abre una venta y vuelve a `/pos`. 4. Recarga el navegador. 5. Haz clic en «Limpiar».
**Resultado esperado:** 10 filas por defecto con opciones 10, 25 y 50; al volver a la pantalla se recuerdan filtros, página y tamaño mientras no recargues el navegador; tras recargar se pierden; «Limpiar» vacía los filtros, oculta la tabla y borra lo recordado.

#### TC-VEN-198 — Botón «Nueva Venta POS»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Baja | [HU-VEN-026] «Botón de nueva venta»; [CU-VEN-26] alternativo A2; [RN-VEN-001] |

| Permiso que falta | Qué no se ve |
|---|---|
| `Ventas:Crear` | No aparece «Nueva Venta POS» en `/pos` |

**Pasos:** abre `/pos` con un rol con `Ventas:Crear` y con uno sin él.
**Resultado esperado:** con el permiso se ve «Nueva Venta POS» y lleva a `/pos/nueva`; sin él no se ve.

#### TC-VEN-199 — Falla la búsqueda y no hay filtro por código ni por monto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-VEN-026] «Falla la búsqueda», «No se puede buscar por código ni por monto»; [CU-VEN-26] excepciones |

**Pasos:** bloquea `GET /api/v1/ventas` y haz clic en «Buscar»; revisa los filtros.
**Resultado esperado:** «Error al buscar ventas»; no existe filtro por código de venta ni por total.

### HU-VEN-027 — Ver el detalle de una venta

#### TC-VEN-200 — Ver una venta cobrada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-027] «Ver una venta cobrada»; [CU-VEN-27] flujo básico; [RN-VEN-074], [RN-CMP-013] |

**Precondiciones:** venta «Completada» con pago en efectivo S/ 60.00 de un total de S/ 50.00.
**Pasos:** abre `/pos/{id}`.
**Resultado esperado:** código de venta, «{dd/MM/yyyy HH:mm} — Completada», Cliente, Empleado y «Total (S/)»; «Comprobante» con «Nº Comprobante:» y «Estado SUNAT:»; «Productos Vendidos» con Producto, Lote, Cantidad, «Devuelto», «P. Unitario (S/)» y «Subtotal (S/)»; «Pagos Registrados» con «Método de Pago», «Monto (S/)», «Referencia de Operación» y «Fecha de Pago»; «Vuelto:» S/ 10.00.

#### TC-VEN-201 — Línea vendida por presentación en el detalle

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-027] «Línea vendida por presentación»; [CU-VEN-27] flujo básico |

**Pasos:** abre el detalle de la venta de `TC-VEN-035`.
**Resultado esperado:** la columna Cantidad muestra «2 × Caja x10 (20)».

#### TC-VEN-202 — Detalle de una venta anulada y de una con devoluciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-027] «Venta anulada», «Devoluciones»; [CU-VEN-27] flujo básico; [RN-VEN-035], [RN-VEN-074] |

**Pasos:** abre el detalle de una venta «Anulada» y el de una venta con devolución (ver `TC-DEV-001`).
**Resultado esperado:** la anulada muestra «Motivo de Anulación:» y el estado «Anulada»; la que tiene devoluciones muestra la sección «Devoluciones / Notas de crédito» (detalle en `TC-DEV-052`).

#### TC-VEN-203 — Qué botones se ofrecen según el estado de la venta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-VEN-027] «Qué botones se ofrecen según el estado»; [CU-VEN-27] flujo básico; [RN-VEN-053], [RN-VEN-062], [RN-CMP-012] |

**Pasos:** abre el detalle de ventas en cada estado con un usuario con todos los permisos y con el rol «Cajero sin dinero».
**Resultado esperado:**

| Botón | Aparece si |
|---|---|
| «Reimprimir Comprobante» | hay comprobante y la venta no está «Anulada» |
| «Cobrar Venta» | está «Pendiente» y tienes `Ventas:Crear` y `Ventas:Actualizar` |
| «Devolver» | está «Completada», tiene comprobante, queda algo por devolver y tienes `Devoluciones:ProcesarDevolucion` |
| «Anular Venta» | está «Completada» o «Pendiente», sin devoluciones, y tienes `Ventas:Anular` |

**Hallazgo:** «Cobrar Venta» no exige caja abierta; si no la hay, el terminal avisa (HU-VEN-027, «Brecha conocida»).

#### TC-VEN-204 — Venta de otra sucursal o inexistente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-VEN-027] «Venta de otra sucursal o inexistente»; [CU-VEN-27] excepciones; [RN-VEN-074] |

**Pasos:** con Carlos abre `/pos/{id}` de una venta de «Sucursal Demo Norte», de una venta inexistente y de una de «Botica Otra Demo S.A.C.».
**Resultado esperado:** «Error al cargar la venta» y la pantalla queda sin datos en los tres casos (por API: 404 `Venta.NotFound` si no existe; 400 `Venta.Forbidden` si es de otra sucursal).

#### TC-VEN-205 — Sin permiso de lectura de ventas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-VEN-027] «Sin permiso de lectura»; [CU-VEN-27] excepciones; [RN-VEN-001] |

| Permiso que falta | Qué se ve |
|---|---|
| `Ventas:Leer` | Al abrir `/pos` o `/pos/{id}` el sistema te lleva a `/dashboard` con «No tienes permisos para acceder a esta pantalla.» |

**Pasos:** abre `/pos/{id}` con un rol sin `Ventas:Leer`.
**Resultado esperado:** según la tabla.

### HU-VEN-028 — Exportar ventas a Excel

#### TC-VEN-206 — Exportar los resultados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-VEN-028] «Exportar los resultados», «El estado sale como nombre del enumerado»; [CU-VEN-28] flujo básico; [RN-VEN-078], [RN-IMP-008] |

**Precondiciones:** más de 10 ventas con distintos estados.
**Pasos:** busca con un filtro y haz clic en «Exportar».
**Resultado esperado:** se descarga `ventas.xlsx` con las mismas ventas del filtro (sin paginar); columnas en orden «Código de Venta», «Fecha», «Cliente», «Empleado», «Total (S/)», «Estado» e «Ítems»; «Estado» dice Completada, Anulada, Devuelta o Pendiente.

#### TC-VEN-207 — Formato de fecha del Excel

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-VEN-028] «Formato de fecha»; [CU-VEN-28] flujo básico; [RN-IMP-009] |

**Pasos:** exporta con el idioma en español y luego con otro idioma; si dispones de una venta después de las 19:00 hora de Lima, mira su fecha.
**Resultado esperado:** español «dd/MM/yyyy», otro idioma «MM/dd/yyyy»; la fecha se formatea sobre fecha y hora en UTC sin convertir a hora local. **No verificado:** el archivo no se descargó al documentar; una venta de después de las 19:00 de Lima saldría con la fecha del día siguiente (01c2 hallazgo 8).

#### TC-VEN-208 — Exportar más de 10 000 ventas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-VEN-028] «Demasiadas filas»; [CU-VEN-28] excepciones; [RN-IMP-010] |

**Precondiciones:** un filtro que devuelva más de 10 000 ventas (requiere cargar datos de prueba).
**Pasos:** haz clic en «Exportar».
**Resultado esperado:** el archivo no se genera y se ve «Error al buscar ventas».

#### TC-VEN-209 — Falla la exportación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-VEN-028] «Falla la exportación»; [CU-VEN-28] excepciones; [RN-IMP-011] |

**Pasos:** bloquea `POST /api/v1/ventas/exportar` y haz clic en «Exportar».
**Resultado esperado:** «Error al buscar ventas».

## 5. Casos del módulo Caja (`TC-CAJ`)

### HU-CAJ-001 — Abrir la caja del turno

#### TC-CAJ-001 — Abrir la caja con S/ 100.00

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-001] «Abrir la caja con S/ 100.00»; [CU-CAJ-01] flujo básico; [RN-CAJ-001], [RN-CAJ-002], [RN-CAJ-003], [RN-CAJ-005] |

**Precondiciones:** `carlos.cajero` con `POS:Leer` y `POS:AperturarCaja` y sin ninguna caja abierta.
**Pasos:**
1. Entra a `/sesiones-caja` y haz clic en «Nuevo».
2. Lee el aviso del formulario.
3. Escribe «Monto Apertura (S/)» 100 y haz clic en «Aperturar».
**Resultado esperado:** el aviso dice «La caja se abrirá a tu nombre, en tu sucursal activa y con la hora actual.»; al confirmar aparece «Sesión aperturada correctamente» y vuelves a la lista de sesiones. Existe una sesión «Abierta» a nombre de Carlos, en «Sede Principal» y con la hora del servidor; los campos de cierre y de arqueo quedan vacíos; se registra el evento de auditoría `CAJA_APERTURAR`.

#### TC-CAJ-002 — La sucursal, el cajero y la hora no se eligen

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-CAJ-001] «La sucursal, el cajero y la hora no se eligen»; [CU-CAJ-01] flujo básico; [RN-CAJ-002] |

**Pasos:** abre `/sesiones-caja/nueva` con Carlos.
**Resultado esperado:** el campo «Cajero» está bloqueado con su nombre; no hay campo de sucursal ni de hora; la sesión creada usa la sucursal base de la sesión de usuario y la hora del servidor.

#### TC-CAJ-003 — Abrir con monto cero

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-CAJ-001] «Abrir con monto cero»; [CU-CAJ-01] flujo básico; [RN-CAJ-003] |

**Pasos:** escribe «Monto Apertura (S/)» 0 (el valor inicial) y haz clic en «Aperturar».
**Resultado esperado:** la caja se abre: cero es válido.

#### TC-CAJ-004 — Monto de apertura vacío o negativo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-CAJ-001] «Monto vacío o negativo»; [CU-CAJ-01] excepciones; [RN-CAJ-003] |

**Pasos:** deja el monto vacío y haz clic en «Aperturar»; luego escribe -5.
**Resultado esperado:** «Ingresa un monto válido» y no se envía nada (sin petición `POST /api/v1/pos/apertura`).

#### TC-CAJ-005 — Ya tengo una caja abierta (también en otra sucursal)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-CAJ-001] «Ya tengo una caja abierta»; [CU-CAJ-01] excepciones; [RN-CAJ-004] |

**Precondiciones:** Carlos con una caja «Abierta» en «Sede Principal».
**Pasos:** 1. Intenta abrir otra desde `/sesiones-caja/nueva`. 2. Por API, `POST /api/v1/pos/apertura` con el token de un usuario con una caja abierta en otra sucursal.
**Resultado esperado:** «Ya tienes una caja abierta. Ciérrala antes de abrir otra.» en ambos casos; no se crea ninguna sesión nueva.

#### TC-CAJ-006 — Dos aperturas al mismo tiempo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-CAJ-001] «Dos aperturas al mismo tiempo»; [CU-CAJ-01] alternativo A2; [RN-CAJ-026] |

**Precondiciones:** Carlos sin caja abierta.
**Pasos:** envía dos `POST /api/v1/pos/apertura` simultáneos con el token de Carlos (dos clientes HTTP en paralelo).
**Resultado esperado:** solo se crea una sesión «Abierta»; la otra petición recibe el mismo aviso de caja ya abierta («Ya tienes una caja abierta. Ciérrala antes de abrir otra.»). **No verificado en ejecución:** lo garantiza un índice único filtrado de la base.

#### TC-CAJ-007 — La sesión de usuario no tiene empleado o sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-CAJ-001] «Mi sesión de usuario no tiene empleado o sucursal»; [CU-CAJ-01] excepciones; [RN-CAJ-002] |

**Precondiciones:** una cuenta cuyo token no traiga empleado o sucursal (**No verificado:** cómo prepararla; ver [05a](../../ola-1-fundamentos/05-seguridad/05a-autenticacion-y-controles.md)).
**Pasos:** intenta abrir la caja.
**Resultado esperado:** «Error al aperturar la sesión».

#### TC-CAJ-008 — Sin permiso para abrir caja

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-CAJ-001] «Sin permiso para abrir caja»; [CU-CAJ-01] excepciones; [RN-CAJ-001] |

| Permiso que falta | Qué no se ve o qué responde |
|---|---|
| `POS:AperturarCaja` | En `/sesiones-caja` no se ve «Nuevo»; al escribir `/sesiones-caja/nueva` el sistema lleva a `/dashboard` con «No tienes permisos para acceder a esta pantalla.»; por API `POST /api/v1/pos/apertura` responde 403 |
| `POS:Leer` | La ruta `/sesiones-caja` no es accesible; el terminal de venta no encuentra la caja (`TC-VEN-002`) |

**Pasos:** con el rol «Cajero sin dinero» menos `POS:AperturarCaja`, recorre lo de la tabla.
**Resultado esperado:** según la tabla.

#### TC-CAJ-009 — Estructura de la sesión de caja

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-CAJ-001] «Estructura de la sesión de caja»; [CU-CAJ-01] excepciones; [RN-CAJ-025], [RN-CAJ-026], [RN-CAJ-027] |

**Pasos:** en la base de datos de prueba revisa la tabla de sesiones de caja y sus restricciones; prueba insertar (solo en un ambiente descartable) una sesión con monto de apertura negativo y una segunda sesión abierta del mismo empleado.
**Resultado esperado:** la base exige sucursal, empleado, fecha de apertura, monto de apertura y estado (Abierta, Cerrada o Cuadrada); los montos de apertura, calculado y declarado no pueden ser negativos (la diferencia sí); el índice único permite una sola sesión abierta por empleado y cualquier número de cerradas o cuadradas; las ventas y devoluciones anotan la sesión que las cobró, que las anuló o de la que sale el reembolso. Una violación distinta de la caja ya abierta se ve como «Error al aperturar la sesión».

#### TC-CAJ-010 — Abrir la caja desde el Inicio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-CAJ-001] «Abrir la caja desde el Inicio»; [CU-CAJ-01] alternativo A1; [RN-CAJ-001] |

**Precondiciones:** Carlos con `POS:AperturarCaja` y sin caja abierta.
**Pasos:** en `/dashboard`, con el mensaje «No tienes una caja abierta», haz clic en «Abrir caja».
**Resultado esperado:** vas a `/sesiones-caja/nueva`.

### HU-CAJ-002 — Abrir la caja desde el terminal de venta

#### TC-CAJ-011 — Abrir la caja desde el diálogo del terminal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-002] «Abrir la caja desde el diálogo»; [CU-CAJ-02] flujo básico; [RN-CAJ-002], [RN-CAJ-003], [RN-CAJ-005], [RN-VEN-003] |

**Precondiciones:** Carlos sin caja abierta, en `/pos/nueva`.
**Pasos:** 1. Haz clic en «Aperturar Caja». 2. Lee el diálogo. 3. Escribe 100 y haz clic en «Aperturar Caja».
**Resultado esperado:** el diálogo «Aperturar Caja» dice «Ingresa los datos para aperturar la caja de esta sucursal y continuar con la venta.», con «Empleado» bloqueado con su nombre y «Monto de Apertura (S/)» con valor inicial 0. Al confirmar: «Caja aperturada correctamente», el diálogo se cierra y el terminal se habilita.

#### TC-CAJ-012 — Monto inválido en el diálogo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-CAJ-002] «Monto inválido»; [CU-CAJ-02] excepciones; [RN-CAJ-003] |

**Pasos:** en el diálogo deja el monto vacío; luego escribe un valor negativo; haz clic en «Aperturar Caja».
**Resultado esperado:** «Ingresa un monto válido» y no se envía nada.

#### TC-CAJ-013 — Ya tengo una caja abierta, desde el diálogo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-CAJ-002] «Ya tengo una caja abierta»; [CU-CAJ-02] excepciones; [RN-CAJ-004] |

**Precondiciones:** Carlos tiene caja abierta pero el terminal no la ve (por ejemplo, rol sin `POS:Leer`, ver `TC-VEN-002`).
**Pasos:** abre el diálogo y confirma con 100.
**Resultado esperado:** «Ya tienes una caja abierta. Ciérrala antes de abrir otra.».

#### TC-CAJ-014 — Otro fallo al abrir desde el diálogo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-CAJ-002] «Otro fallo»; [CU-CAJ-02] excepciones |

**Pasos:** bloquea `POST /api/v1/pos/apertura` y confirma el diálogo.
**Resultado esperado:** «No se pudo aperturar la caja».

#### TC-CAJ-015 — Cancelar el diálogo de apertura

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-CAJ-002] «Cancelar»; [CU-CAJ-02] alternativo A1 |

**Pasos:** abre el diálogo y haz clic en «Cancelar»; ábrelo otra vez y pulsa Esc; ábrelo y haz clic fuera.
**Resultado esperado:** el diálogo se cierra y el terminal no vuelve a comprobar la caja (sigue el aviso de `TC-VEN-002`).

### HU-CAJ-003 — Cerrar la caja y declarar el efectivo contado

#### TC-CAJ-016 — Cerrar la caja con el monto exacto (queda «Cuadrada»)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-003] «Cerrar la caja con el monto exacto (queda "Cuadrada")»; [CU-CAJ-03] flujo básico; [RN-CAJ-001], [RN-CAJ-007], [RN-CAJ-015] |

**Precondiciones:** caja de Carlos «Abierta» con apertura S/ 100.00 y una venta en efectivo de S/ 50.00 sin vuelto (efectivo esperado S/ 150.00).
**Pasos:** abre la sesión en `/sesiones-caja/{id}`, haz clic en «Cerrar Sesión», escribe «Monto Cierre Declarado (S/)» 150 y haz clic en «Confirmar Cierre».
**Resultado esperado:** «Sesión cerrada correctamente» y vuelves a la lista de sesiones; la sesión queda «Cuadrada» con diferencia 0.00.

#### TC-CAJ-017 — Resultado del cierre según lo declarado (esquema)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-CAJ-003] «Resultado del cierre según lo declarado»; [CU-CAJ-03] flujo básico; [RN-CAJ-015] |

**Precondiciones:** cuatro cajas de prueba (por ejemplo cuatro cajeros o cuatro días), cada una con efectivo esperado S/ 150.00 (apertura S/ 100.00 más venta en efectivo de S/ 50.00).
**Pasos:** cierra cada caja declarando un monto distinto y abre el detalle de cada sesión.
**Resultado esperado:**

| Declarado | Diferencia | Estado | Lectura |
|---|---|---|---|
| 150.00 | 0.00 | Cuadrada | cuadre |
| 145.00 | -5.00 | Cerrada | faltante |
| 152.50 | 2.50 | Cerrada | sobrante |
| 150.01 | 0.01 | Cerrada | sobrante de un centavo |

La diferencia se muestra sin rótulo de «faltante» o «sobrante»: solo el número con signo.

#### TC-CAJ-018 — No existe tolerancia en el cuadre

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-CAJ-003] «No existe tolerancia»; [CU-CAJ-03] flujo básico; [RN-CAJ-015] |

**Pasos:** con esperado S/ 150.00 declara S/ 150.01 y cierra.
**Resultado esperado:** la sesión queda «Cerrada» y no «Cuadrada» (diferencia 0.01).

#### TC-CAJ-019 — Qué se guarda al cerrar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-003] «Qué se guarda al cerrar»; [CU-CAJ-03] flujo básico; [RN-CAJ-010], [RN-CAJ-019] |

**Pasos:** tras cerrar la caja de `TC-CAJ-017` con 145.00, abre el detalle, intenta reabrirla o editarla y revisa la tabla de auditoría.
**Resultado esperado:** se guardan la hora de cierre (UTC del servidor), el monto declarado (145.00), el efectivo esperado como «Monto Cierre Calculado» (150.00) y la diferencia con signo (-5.00); el estado final es definitivo (no se puede reabrir, editar ni eliminar la sesión); hay un evento `CAJA_CERRAR` y su detalle no incluye el esperado ni la diferencia.

#### TC-CAJ-020 — El panel de cierre no muestra el efectivo esperado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-CAJ-003] «El panel de cierre no muestra el efectivo esperado»; [CU-CAJ-03] flujo básico |

**Pasos:** abre el detalle de una sesión «Abierta» y haz clic en «Cerrar Sesión».
**Resultado esperado:** el detalle muestra «—» en calculado y diferencia; el panel solo pide «Monto Cierre Declarado (S/)»; el esperado se ve antes en el Inicio, en «Efectivo esperado» (`TC-DAS-017`).

#### TC-CAJ-021 — Monto declarado vacío o negativo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-CAJ-003] «Monto declarado vacío o negativo»; [CU-CAJ-03] excepciones; [RN-CAJ-009] |

**Pasos:** en el panel de cierre deja el campo vacío, tócalo y sal; escribe -1; haz clic en «Confirmar Cierre».
**Resultado esperado:** «Ingresa el monto declarado al cierre» y no se envía nada. Con 0 el cierre sí se acepta (cero es válido; el monto debe ser mayor o igual a 0).

#### TC-CAJ-022 — Solo el titular o un administrador cierra

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-CAJ-003] «Solo el titular o un administrador cierra», «El botón se ofrece aunque la caja no sea mía»; [CU-CAJ-03] excepciones; [RN-CAJ-007] |

**Precondiciones:** caja «Abierta» de otro cajero de «Sede Principal»; Carlos con `POS:CerrarCaja` y sin rol `Admin`.
**Pasos:** abre la sesión del otro cajero, comprueba el botón «Cerrar Sesión», escribe un monto y haz clic en «Confirmar Cierre».
**Resultado esperado:** el botón «Cerrar Sesión» se ofrece y el rechazo llega al confirmar: «Solo el cajero que abrió la caja o un administrador puede cerrarla.»; vale aunque tengas `POS:CerrarCaja` y seas de la misma sucursal. La sesión sigue «Abierta».

#### TC-CAJ-023 — Un administrador cierra la caja de otro cajero

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-CAJ-003] «Un administrador cierra la caja de otro cajero»; [CU-CAJ-03] alternativo A1; [RN-CAJ-007] |

**Precondiciones:** caja «Abierta» de Carlos (o de Nora en «Sucursal Demo Norte»).
**Pasos:** con «Ana Demo Prueba» (Admin) cierra la sesión declarando un monto.
**Resultado esperado:** el cierre se realiza; el administrador puede cerrar la sesión de cualquier cajero y sucursal de su empresa. La sesión de una caja de «Botica Otra Demo S.A.C.» no se puede cerrar (`TC-CAJ-039`).

#### TC-CAJ-024 — La sesión ya no está abierta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-CAJ-003] «La sesión ya no está abierta»; [CU-CAJ-03] excepciones; [RN-CAJ-010] |

**Pasos:** abre el detalle de una sesión «Cerrada» o «Cuadrada»; por API `POST /api/v1/pos/cierre` con su identificador.
**Resultado esperado:** el detalle no ofrece «Cerrar Sesión»; por API se rechaza y el SPA mostraría «Error al cerrar la sesión».

#### TC-CAJ-025 — Cancelar el panel de cierre

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-CAJ-003] «Cancelar el panel»; [CU-CAJ-03] alternativo A2 |

**Pasos:** haz clic en «Cerrar Sesión» y luego en «Cancelar».
**Resultado esperado:** el panel se oculta y no se cierra nada; la sesión sigue «Abierta».

#### TC-CAJ-026 — Hay ventas en espera al cerrar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-CAJ-003] «Hay ventas en espera al cerrar»; [CU-CAJ-03] alternativo A3; [RN-CAJ-017] |

**Precondiciones:** Carlos con una venta «Pendiente» de S/ 20.00 y caja con apertura S/ 100.00 sin otras ventas.
**Pasos:** cierra la caja declarando 100.00. Luego otro cajero con caja propia cobra esa venta en espera en efectivo.
**Resultado esperado:** el cierre no se bloquea ni avisa; la sesión de Carlos queda «Cuadrada» (esperado 100.00: la venta en espera no entra); la venta cobrada después cuenta en el arqueo de la sesión del otro cajero.

#### TC-CAJ-027 — El arqueo es una foto del momento del cierre

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-003] «El arqueo es una foto del momento del cierre»; [CU-CAJ-03] alternativo A4; [RN-CAJ-018] |

**Precondiciones:** caja de Carlos cerrada con esperado S/ 150.00 (incluye una venta en efectivo cobrada en ella); otro cajero con su caja abierta.
**Pasos:** 1. El otro cajero anula (`Ventas:Anular`) esa venta cobrada en la caja ya cerrada. 2. Abre el detalle de la sesión cerrada de Carlos. 3. Por API intenta `POST /api/v1/ventas` con `sesionId` de esa sesión cerrada.
**Resultado esperado:** la sesión de Carlos conserva el valor ya guardado (150.00 esperado y su diferencia); el esperado no se recalcula; la venta con la sesión cerrada se rechaza (`Venta.Caja`, HTTP 400); una sesión cerrada no admite más ventas ni devoluciones.

### HU-CAJ-004 — Calcular el efectivo esperado (arqueo)

#### TC-CAJ-028 — Fórmula del efectivo esperado: ejemplo sencillo con vuelto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-004] «Fórmula del efectivo esperado», «Ejemplo sencillo con vuelto»; [CU-CAJ-04] flujo básico; [RN-CAJ-011], [RN-CAJ-012], [RN-CAJ-013], [RN-VEN-044] |

**Precondiciones:** caja nueva de Carlos con apertura S/ 100.00.
**Pasos:** 1. Vende V1 de S/ 50.00 pagada con S/ 60.00 en efectivo (vuelto S/ 10.00). 2. Vende V2 de S/ 30.00 pagada con tarjeta (con número de operación). 3. Mira «Efectivo esperado» en el Inicio (`TC-DAS-017`) y declara 150 al cerrar.
**Resultado esperado:** efectivo neto de V1 = 60.00 − 10.00 = 50.00; el de V2 = 0.00; esperado = 100.00 + 50.00 + 0.00 = 150.00 (apertura más efectivo neto de las ventas que entran menos anulaciones y reembolsos). Al declarar 150 la sesión queda «Cuadrada».

#### TC-CAJ-029 — Ejemplo completo con anulaciones y reembolso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-004] «Ejemplo completo con anulaciones y reembolso»; [CU-CAJ-04] flujo básico; [RN-CAJ-011], [RN-CAJ-012], [RN-CAJ-014], [RN-VEN-067] |

**Precondiciones:** dos cajas abiertas en «Sede Principal»: la de Carlos (A, apertura S/ 100.00) y la de otro cajero (B).
**Pasos:**
1. En A: V1 de 50.00 en efectivo con vuelto 10.00 (cobrada con 60.00).
2. En A: V2 de 30.00 con tarjeta.
3. En B: V3 de 20.00 en efectivo; Carlos la anula desde A.
4. En A: V4 de 40.00 en efectivo; el otro cajero la anula desde B.
5. En A: una devolución con reembolso en efectivo de 15.00 sobre una venta cobrada en B (ver `TC-DEV-023`).
6. Cierra A declarando 155.
**Resultado esperado:** aportes: V1 +50.00, V2 +0.00, V3 −20.00 (cobrada en otra sesión y anulada desde esta), V4 +40.00 (cobrada en esta y anulada desde otra); reembolso −15.00. Esperado = 100.00 + (50.00 + 0.00 − 20.00 + 40.00) − 15.00 = 155.00. Con 155.00 la sesión queda «Cuadrada»; con 150.00 la diferencia es −5.00 y queda «Cerrada»; con 160.00 es +5.00 («5.00» sin signo en pantalla, ver `TC-CAJ-044`) y queda «Cerrada».

#### TC-CAJ-030 — El vuelto se descuenta siempre del efectivo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-004] «El vuelto se descuenta siempre del efectivo»; [CU-CAJ-04] flujo básico; [RN-CAJ-013], [RN-VEN-044] |

**Pasos:** vende S/ 50.00 pagados con S/ 40.00 por Yape / Plin y S/ 15.00 en efectivo (`TC-VEN-104`); mira el esperado.
**Resultado esperado:** suma de pagos 55.00, vuelto 5.00, efectivo neto 15.00 − 5.00 = 10.00; el pago por Yape / Plin no aporta efectivo; el esperado sube en 10.00 sobre la apertura.

#### TC-CAJ-031 — Venta con cobertura de seguro

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-004] «Venta con cobertura de seguro»; [CU-CAJ-04] flujo básico; [RN-CAJ-013], [RN-VEN-047] |

**Pasos:** vende por API la venta de `TC-VEN-141` (S/ 100.00 con S/ 40.00 de cobertura, pagada con S/ 100.00 en efectivo); mira el esperado.
**Resultado esperado:** monto a pagar 60.00, vuelto 40.00 y efectivo neto 60.00; el esperado sube en 60.00 sobre la apertura.

#### TC-CAJ-032 — Qué ventas entran en el arqueo y con qué signo (esquema)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-004] «Qué ventas entran en el arqueo y con qué signo»; [CU-CAJ-04] flujo básico; [RN-CAJ-012], [RN-VEN-067] |

**Precondiciones:** dos cajas abiertas (A y B) y un conjunto de ventas en efectivo de S/ 20.00 cada una según el caso. El caso E requiere una venta anulada anterior al script `0028` sin sesión de anulación (datos antiguos; **No verificado** si hay datos así en tu base).
**Pasos:** para cada caso arma la venta, ciérrala o consulta el esperado de la caja A y compara el aporte.
**Resultado esperado:**

| Caso | Situación | Entra |
|---|---|---|
| A | cobrada en esta sesión y «Completada» o «Devuelta» | con signo + |
| B | cobrada en esta sesión y anulada desde esta misma sesión | no entra (neto 0) |
| C | cobrada en otra sesión y anulada desde esta | con signo − (sale de aquí) |
| D | cobrada en esta sesión y anulada desde otra sesión | con signo + |
| E | cobrada en esta sesión, anulada y sin sesión de anulación (anterior al script 0028) | no entra |

#### TC-CAJ-033 — Las ventas en espera nunca entran en el arqueo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-004] «Las ventas en espera nunca entran»; [CU-CAJ-04] flujo básico; [RN-CAJ-012], [RN-CAJ-017] |

**Pasos:** deja una venta de S/ 20.00 en espera; mira el esperado de tu caja; luego cóbrala con tu propia caja.
**Resultado esperado:** mientras es «Pendiente» no aporta nada; al cobrarla pertenece a la sesión de quien la cobra y entra con su efectivo neto.

#### TC-CAJ-034 — Un reembolso por medio digital no toca el arqueo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-004] «Un reembolso por medio digital no toca el arqueo»; [CU-CAJ-04] flujo básico; [RN-CAJ-014], [RN-DEV-018], [RN-DEV-019] |

**Pasos:** procesa tres devoluciones con método de reembolso «Yape / Plin», «Tarjeta Débito/Crédito» y «Transferencia» (`TC-DEV-025`) y una con «Efectivo»; mira el esperado.
**Resultado esperado:** solo el reembolso en «Efectivo» se resta del esperado (aunque la venta devuelta se haya cobrado en otra sesión); los tres digitales no lo modifican.

#### TC-CAJ-035 — Una venta totalmente devuelta sigue sumando

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-CAJ-004] «Una venta totalmente devuelta sigue sumando»; [CU-CAJ-04] alternativo A1; [RN-CAJ-012], [RN-CAJ-014] |

**Pasos:** vende S/ 20.00 en efectivo, devuélvela por completo con reembolso en efectivo, y mira el esperado.
**Resultado esperado:** la venta «Devuelta» sigue entrando con signo + (+20.00) y el reembolso se resta aparte (−20.00); el neto de ambas es 0.

#### TC-CAJ-036 — Qué columnas lee el arqueo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-CAJ-004] «Qué columnas lee el arqueo»; [CU-CAJ-04] flujo básico; [RN-CAJ-027] |

**Pasos:** tras cobrar una venta, anularla desde otra caja y procesar una devolución, revisa en la base de datos las columnas de sesión de cobro, de anulación y de la devolución.
**Resultado esperado:** la venta anota la sesión que la cobró; si se anuló una venta cobrada, la sesión de anulación; la devolución, la sesión de quien la procesó. Las ventas y devoluciones anteriores a los scripts `0016` y `0028` pueden no tenerlas.

### HU-CAJ-005 — Entender por qué no se puede cerrar una caja

#### TC-CAJ-037 — El efectivo esperado sería negativo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-CAJ-005] «El efectivo esperado sería negativo»; [CU-CAJ-05] flujo básico; [RN-CAJ-016] |

**Precondiciones:** caja de Carlos con apertura S/ 10.00 y sin ventas en efectivo; una venta cobrada en otra caja con S/ 15.00 en efectivo, devuelta desde la caja de Carlos con reembolso en efectivo de S/ 15.00.
**Pasos:** abre la sesión, haz clic en «Cerrar Sesión», declara cualquier monto (por ejemplo 0 y luego 10) y haz clic en «Confirmar Cierre».
**Resultado esperado:** el sistema rechaza el cierre y se ve «Error al cerrar la sesión»; la sesión sigue «Abierta» y el código no ofrece otra vía para cerrarla (esperado 10.00 − 15.00 = −5.00).
**Hallazgo:** la pantalla solo dice «Error al cerrar la sesión» (el mensaje del servidor está en inglés y no llega al usuario); el usuario no puede resolverlo en pantalla (01c3 §6.3 hallazgo 1).

#### TC-CAJ-038 — Dos cierres simultáneos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-CAJ-005] «Dos cierres simultáneos»; [CU-CAJ-05] excepciones; [RN-CAJ-010] |

**Pasos:** el titular y un administrador envían `POST /api/v1/pos/cierre` a la vez para la misma sesión (dos clientes HTTP en paralelo).
**Resultado esperado:** solo se guarda uno y el otro ve «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (409). **No verificado en ejecución.**

#### TC-CAJ-039 — La sesión no existe o no es de una sucursal accesible

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-CAJ-005] «La sesión no existe o no es de una sucursal accesible»; [CU-CAJ-05] excepciones; [RN-CAJ-008] |

**Pasos:** `POST /api/v1/pos/cierre` con un identificador inexistente y con el de una sesión de «Botica Otra Demo S.A.C.».
**Resultado esperado:** HTTP 404, `code` `Caja`; el SPA muestra «Error al cerrar la sesión». No se cierra nada.

#### TC-CAJ-040 — Cerrar una sesión que no está «Abierta» (API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-CAJ-005] «La sesión no está "Abierta"»; [CU-CAJ-05] excepciones; [RN-CAJ-010] |

**Pasos:** `POST /api/v1/pos/cierre` con una sesión «Cerrada» y otra «Cuadrada».
**Resultado esperado:** rechazo; el SPA mostraría «Error al cerrar la sesión»; los montos de la sesión no cambian.

#### TC-CAJ-041 — Falta el empleado en la sesión de usuario al cerrar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-CAJ-005] «Falta el empleado en mi sesión de usuario»; [CU-CAJ-05] excepciones; [RN-CAJ-002] |

**Precondiciones:** cuenta sin empleado en el token (**No verificado:** cómo prepararla).
**Pasos:** intenta cerrar una sesión.
**Resultado esperado:** «Error al cerrar la sesión».

### HU-CAJ-006 — Ver el resultado del arqueo de una sesión

#### TC-CAJ-042 — Ver una sesión cerrada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-CAJ-006] «Ver una sesión cerrada»; [CU-CAJ-06] flujo básico; [RN-CAJ-015], [RN-CAJ-023] |

**Precondiciones:** sesión «Cerrada» con apertura 100.00, esperado 150.00 y declarado 145.00 (`TC-CAJ-017`).
**Pasos:** abre `/sesiones-caja/{id}`.
**Resultado esperado:** «Detalle de Sesión» con Sucursal, Cajero, «Fecha / Hora Apertura», «Fecha / Hora Cierre» y Estado «Cerrada»; «Monto Apertura (S/)» 100.00, «Monto Cierre Declarado (S/)» 145.00, «Monto Cierre Calculado (S/)» 150.00 y «Diferencia Arqueo (S/)» -5.00; todos los campos son de solo lectura.

#### TC-CAJ-043 — Una sesión abierta en el detalle

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-CAJ-006] «Una sesión abierta»; [CU-CAJ-06] flujo básico; [RN-CAJ-023] |

**Pasos:** abre el detalle de una sesión «Abierta» con un usuario con y sin `POS:CerrarCaja`.
**Resultado esperado:** los campos de cierre, calculado y diferencia muestran «—»; con `POS:CerrarCaja` se ve «Cerrar Sesión», sin él no.

#### TC-CAJ-044 — La diferencia positiva se muestra sin signo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-CAJ-006] «La diferencia positiva se muestra sin signo»; [CU-CAJ-06] flujo básico; [RN-CAJ-015] |

**Pasos:** cierra una caja con diferencia +5.00 (`TC-CAJ-029` con 160.00) y abre el detalle.
**Resultado esperado:** «Diferencia Arqueo (S/)» muestra «5.00» sin «+»; las negativas llevan «-».

#### TC-CAJ-045 — Sesión de una sucursal que no puedo ver

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-CAJ-006] «Sesión de una sucursal que no puedo ver»; [CU-CAJ-06] excepciones; [RN-CAJ-020], [RN-CAJ-023] |

**Precondiciones:** una sesión de «Sucursal Demo Norte»; Carlos sin acceso a esa sucursal (sucursal base «Sede Principal», sin asignación a Norte).
**Pasos:** abre `/sesiones-caja/{id de la sesión de Norte}` con Carlos.
**Resultado esperado:** «Error al cargar la sesión de caja».

#### TC-CAJ-046 — La respuesta del detalle (API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-CAJ-006] «La respuesta del detalle»; [CU-CAJ-06] flujo básico; [RN-CAJ-023] |

**Pasos:** `GET /api/v1/pos/{id}` con el token de un usuario con `POS:Leer`.
**Resultado esperado:** HTTP 200 con el objeto de resultado completo y la sesión en el campo `value`.
**Hallazgo:** `GET Pos/{id}` devuelve el objeto `Result` completo en vez del valor directo (01c3 hallazgo 8).

### HU-CAJ-007 — Buscar sesiones de caja

#### TC-CAJ-047 — Buscar sesiones abiertas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-CAJ-007] «Buscar sesiones abiertas»; [CU-CAJ-07] flujo básico; [RN-CAJ-021], [RN-CAJ-022] |

**Precondiciones:** al menos dos sesiones «Abiertas» y varias cerradas en «Sede Principal».
**Pasos:** en `/sesiones-caja` elige «Estado» = «Abierta» y haz clic en «Buscar».
**Resultado esperado:** tarjeta «Búsqueda de Sesiones de Caja»; «Resultados (N)» con las columnas «Sucursal» (enlace al detalle), «Cajero», «Apertura», «Cierre» («—» si está abierta), «Monto Apertura» y «Estado»; orden por apertura descendente.

#### TC-CAJ-048 — Filtros disponibles

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-CAJ-007] «Filtros disponibles»; [CU-CAJ-07] flujo básico; [RN-CAJ-021], [RN-GEN-008] |

**Pasos:** abre «Estado» y los campos de fecha; busca con «Apertura desde» y «Apertura hasta» del día de hoy.
**Resultado esperado:** «Estado» ofrece «— Todos —», «Abierta», «Cerrada» y «Cuadrada»; también «Apertura desde» y «Apertura hasta»; las fechas se interpretan como inicio y fin del día en tu zona horaria; no hay filtros por sucursal ni por cajero.
**Hallazgo:** la API admite filtrar por sucursal y por cajero; la pantalla no (estado Parcial).

#### TC-CAJ-049 — Qué sesiones veo según mi alcance

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-CAJ-007] «Qué sesiones veo»; [CU-CAJ-07] flujo básico; [RN-CAJ-020], [RN-GEN-023] |

**Precondiciones:** sesiones en «Sede Principal» (de varios cajeros) y en «Sucursal Demo Norte»; Carlos sin acceso a Norte.
**Pasos:** busca sin filtros con Carlos (no Admin), con una cuenta con Norte asignada en su cuenta o rol, y con Ana (Admin).
**Resultado esperado:** Carlos ve las sesiones de «Sede Principal» de todos sus cajeros (no solo las suyas); quien tenga Norte asignada ve también las de Norte; el administrador ve las de todas las sucursales de la empresa; nadie ve sesiones de «Botica Otra Demo S.A.C.».
**Hallazgo:** no existe una vista «solo mis sesiones» (01c3 hallazgo 9).

#### TC-CAJ-050 — Los resultados no aparecen hasta buscar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-CAJ-007] «Los resultados no aparecen hasta buscar»; [CU-CAJ-07] flujo básico; [RN-GEN-009] |

**Pasos:** entra a `/sesiones-caja`; busca; cambia el tamaño de página; vuelve a la pantalla; haz clic en «Limpiar».
**Resultado esperado:** la tabla se oculta hasta pulsar «Buscar» o hasta que haya estado guardado; tamaños de página 10, 25 y 50; «Limpiar» vacía el formulario y borra lo recordado.

#### TC-CAJ-051 — Sin resultados, falla la búsqueda y columnas ausentes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-CAJ-007] «Sin resultados», «Falla la búsqueda», «Calculado, declarado y diferencia no están en la tabla»; [CU-CAJ-07] excepciones |

**Pasos:** busca con un rango de fechas sin sesiones; bloquea `GET /api/v1/pos/sesiones` y busca; revisa las columnas de la tabla.
**Resultado esperado:** «Sin resultados»; «Error al buscar sesiones de caja»; la tabla no muestra calculado, declarado ni diferencia (se ven en el detalle y en el Excel).

#### TC-CAJ-052 — Filtros de la API con un identificador vacío

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-CAJ-007] «Filtros de la API con un identificador vacío»; [CU-CAJ-07] excepciones; [RN-CAJ-021] |

**Pasos:** `GET /api/v1/pos/sesiones?sucursalId=00000000-0000-0000-0000-000000000000` y lo mismo con `empleadoId`; luego con el `sucursalId` de una sucursal no accesible.
**Resultado esperado:** HTTP 400, `code` `GetSesionesQuery.SucursalId` o `GetSesionesQuery.EmpleadoId`; con la sucursal no accesible, una página vacía (no un error).

### HU-CAJ-008 — Exportar sesiones de caja a Excel

#### TC-CAJ-053 — Exportar las sesiones del filtro

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-CAJ-008] «Exportar las sesiones del filtro»; [CU-CAJ-08] flujo básico; [RN-CAJ-024], [RN-IMP-008] |

**Pasos:** busca sesiones con estado y fechas y haz clic en «Exportar».
**Resultado esperado:** se descarga `sesiones-caja.xlsx` con 9 columnas, en este orden: «Sucursal», «Cajero», «Apertura», «Cierre», «Monto Apertura», «Monto Cierre Declarado (S/)», «Monto Cierre Calculado (S/)», «Diferencia Arqueo (S/)» y «Estado»; respeta el estado y las fechas del filtro, con el mismo alcance de sucursales.

#### TC-CAJ-054 — El filtro por empleado no se aplica en la exportación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-CAJ-008] «El filtro por empleado no se aplica»; [CU-CAJ-08] flujo básico; [RN-CAJ-024] |

**Pasos:** `POST /api/v1/pos/sesiones/exportar` con un `empleadoId` y compara con el listado `GET /api/v1/pos/sesiones?empleadoId=…`.
**Resultado esperado:** la exportación no admite el filtro de empleado: trae las sesiones de todos los cajeros del alcance, aunque el listado de la API sí lo respeta. **No verificado:** el nombre exacto del campo del cuerpo de la exportación.

#### TC-CAJ-055 — Formato de fecha del Excel de sesiones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-CAJ-008] «Formato de fecha»; [CU-CAJ-08] flujo básico; [RN-IMP-009] |

**Pasos:** exporta con el idioma en español y con otro idioma; si dispones de una apertura a las 20:00 hora de Lima, mira su día.
**Resultado esperado:** español «dd/MM/yyyy HH:mm», otro idioma «MM/dd/yyyy HH:mm»; fechas en UTC sin convertir a hora local (una apertura a las 20:00 de Lima saldría con el día siguiente). **No verificado:** el archivo no se descargó al documentar (01c3 hallazgo 3).

#### TC-CAJ-056 — Más de 10 000 sesiones o falla la exportación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-CAJ-008] «Más de 10 000 sesiones», «Falla la exportación»; [CU-CAJ-08] excepciones; [RN-IMP-010], [RN-IMP-011] |

**Pasos:** con un filtro que devuelva más de 10 000 sesiones (datos de prueba) haz clic en «Exportar»; luego bloquea `POST /api/v1/pos/sesiones/exportar`.
**Resultado esperado:** en ambos casos el archivo no se genera y se ve «Error al buscar sesiones de caja».

## 6. Casos del módulo Devoluciones (`TC-DEV`)

Los casos de pantalla se ejecutan sobre el detalle de una venta (`/pos/{id}`): no existe pantalla de listado ni de detalle de devoluciones. El cuerpo de `POST /api/v1/devoluciones` lleva en `comprobanteOrigenId` el identificador de la **venta** (HU-DEV-001). Los nombres exactos de los demás campos del cuerpo (líneas, destino, motivo, sustento, método de reembolso) son **No verificado**: consulta `sofia-apis/src/SOFIA.Application/Devoluciones/` antes de armar la petición.

### HU-DEV-001 — Devolver parte o todo lo vendido

#### TC-DEV-001 — Devolución parcial

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DEV-001] «Devolución parcial»; [CU-DEV-01] alternativo A1; [RN-DEV-001], [RN-DEV-002], [RN-DEV-009], [RN-DEV-010], [RN-DEV-020], [RN-DEV-025] |

**Precondiciones:** venta `B001-00000001` «Completada» con 100 unidades de «Paracetamol Demo 500 mg» a S/ 0.50 (S/ 50.00), en «Sede Principal»; Carlos con `Devoluciones:ProcesarDevolucion`, su caja abierta (apertura S/ 100.00) y serie de nota de crédito `BC01` activa.
**Pasos:**
1. Abre el detalle de la venta y haz clic en «Devolver».
2. Lee el diálogo «Procesar devolución — B001-00000001».
3. Escribe «A devolver» 30, elige «Método de reembolso» «Efectivo», escribe «Sustento» «Producto en mal estado» y haz clic en «Procesar devolución».
**Resultado esperado:** el diálogo dice «Indica cuántas unidades devuelve el cliente en cada línea y su destino. Se emitirá una nota de crédito y el reembolso saldrá de tu caja abierta.»; al confirmar aparece «Devolución procesada y nota de crédito emitida.»; el detalle se recarga con «Devuelto» 30 en la línea y la sección «Devoluciones / Notas de crédito»; la venta sigue «Completada» (quedan 70 unidades devolvibles); el stock del lote sube en 30.

#### TC-DEV-002 — Devolución total en dos pasos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DEV-001] «Devolución total en dos pasos»; [CU-DEV-01] alternativo A2; [RN-DEV-025] |

**Precondiciones:** la venta de `TC-DEV-001` con 30 unidades ya devueltas.
**Pasos:** haz clic en «Devolver», luego en «Devolver todo lo pendiente» (70 unidades) y confirma con un sustento.
**Resultado esperado:** el «Motivo SUNAT» propuesto es «06 — Devolución total»; la venta pasa a «Devuelta»; una venta «Devuelta» ya no admite más devoluciones ni anulación.

#### TC-DEV-003 — Quién autoriza la devolución

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-001] «Quién autoriza»; [CU-DEV-01] flujo básico; [RN-DEV-002], [RN-DEV-027] |

**Pasos:** procesa una devolución con Carlos y consulta `GET /api/v1/devoluciones/{id}` (`TC-DEV-057`) o la base de datos.
**Resultado esperado:** la devolución guarda como empleado que autoriza al empleado de la sesión de Carlos; el SPA no pide supervisor ni PIN.

#### TC-DEV-004 — Qué no cambia al devolver

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DEV-001] «Qué no cambia al devolver»; [CU-DEV-01] alternativo A3; [RN-DEV-026] |

**Precondiciones:** venta de `LOT-QA-C01` con la receta `RX-QA-002` (`TC-VEN-054`).
**Pasos:** devuelve parte o todo; mira los pagos, el total, la boleta y su estado; intenta una nueva venta con la misma receta.
**Resultado esperado:** los pagos, el total, la boleta y su estado no cambian; la receta médica y sus dispensaciones no se tocan: la nueva venta con `RX-QA-002` se rechaza con «La receta médica ya fue dispensada el máximo de veces permitido.».

#### TC-DEV-005 — Todo o nada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-DEV-001] «Todo o nada»; [CU-DEV-01] excepciones; [RN-DEV-028] |

**Precondiciones:** venta con dos líneas; la sucursal sin serie de nota de crédito activa (se rechaza al final de la validación) o una línea que no pertenece a la venta.
**Pasos:** por API, `POST /api/v1/devoluciones` con una línea válida y otra inexistente.
**Resultado esperado:** rechazo («Una de las líneas ya no pertenece a la venta.»); no queda nada: ni la devolución, ni el reintegro de stock de la línea válida, ni la nota de crédito, ni el cambio de estado, ni la auditoría; el correlativo de la nota de crédito no avanza.

#### TC-DEV-006 — Repetir la misma devolución

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-DEV-001] «Repetir la misma devolución»; [CU-DEV-01] alternativo A4; [RN-DEV-028] |

**Precondiciones:** una línea con 10 unidades vendidas y ninguna devuelta.
**Pasos:** 1. En el diálogo, haz doble clic en «Procesar devolución» con "A devolver" 10. 2. Repite la misma petición por API después de la primera.
**Resultado esperado:** el botón se deshabilita mientras guarda; el segundo intento, si ya no queda saldo, recibe «La cantidad supera lo que queda por devolver en alguna línea. Actualiza la pantalla e inténtalo de nuevo.» (o el aviso de conflicto de concurrencia); se registra una sola devolución y el stock sube una sola vez.
**Hallazgo:** la operación no tiene clave de idempotencia (RN-DEV-028).

#### TC-DEV-007 — Dos devoluciones simultáneas de la misma venta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | [HU-DEV-001] «Dos devoluciones simultáneas de la misma venta»; [CU-DEV-01] excepciones; [RN-DEV-028] |

**Pasos:** dos cajeros con caja abierta envían a la vez la devolución de la misma línea completa (dos clientes HTTP en paralelo).
**Resultado esperado:** solo se guarda una y la otra ve «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.». **No verificado en ejecución.**

#### TC-DEV-008 — Auditoría de la devolución

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-001] «Auditoría»; [CU-DEV-01] flujo básico; [RN-DEV-029] |

**Pasos:** procesa una devolución de 2 líneas con método «Efectivo» y revisa la tabla de auditoría.
**Resultado esperado:** hay un evento `DEVOLUCION_PROCESAR` con el id de la venta, el número de líneas del pedido (2) y el método de reembolso.

#### TC-DEV-009 — El botón «Devolver» solo aparece cuando corresponde

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-DEV-001] «El botón "Devolver" solo aparece cuando corresponde»; [CU-DEV-01] flujo básico; [RN-DEV-001], [RN-DEV-008] |

**Pasos:** abre el detalle de ventas «Completada» con saldo, «Completada» totalmente devuelta pero no marcada, «Pendiente», «Anulada» y «Devuelta», con un usuario con el permiso y con el rol «Cajero sin dinero».
**Resultado esperado:**

| Situación | ¿Se ve «Devolver»? |
|---|---|
| «Completada», con comprobante, con saldo devolvible y con `Devoluciones:ProcesarDevolucion` | Sí |
| Sin el permiso `Devoluciones:ProcesarDevolucion` | No |
| «Pendiente», «Anulada» o «Devuelta» | No |

#### TC-DEV-010 — Respuesta de la API al procesar la devolución

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-001] «Respuesta de la API»; [CU-DEV-01] flujo básico; [RN-DEV-029] |

**Pasos:** `POST /api/v1/devoluciones` válido con token de Carlos.
**Resultado esperado:** HTTP 200 con el identificador de la devolución.

#### TC-DEV-011 — Una venta con devoluciones ya no se puede anular

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-DEV-001] «Una venta con devoluciones ya no se puede anular»; [CU-DEV-01] alternativo A3; [RN-VEN-066] |

**Pasos:** abre el detalle de una venta con una devolución parcial.
**Resultado esperado:** el detalle no ofrece «Anular Venta» (el rechazo del servidor está en `TC-VEN-183`).

#### TC-DEV-012 — Error no traducido al devolver

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-DEV-001] «Error no traducido»; [CU-DEV-01] excepciones |

**Pasos:** bloquea `POST /api/v1/devoluciones` (o usa un token sin empleado o sucursal) y confirma una devolución.
**Resultado esperado:** «Error al procesar la devolución.».

### HU-DEV-002 — Elegir la cantidad y el destino de cada línea devuelta

#### TC-DEV-013 — La tabla lista solo lo devolvible

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DEV-002] «La tabla lista solo lo devolvible»; [CU-DEV-02] flujo básico; [RN-DEV-009], [RN-DEV-012] |

**Precondiciones:** venta con una línea de 10 unidades con 4 ya devueltas y otra línea totalmente devuelta; otra línea vendida por presentación (por ejemplo «2 × Caja x10»).
**Pasos:** abre «Devolver».
**Resultado esperado:** la tabla (Producto, Lote, Vendido, Devuelto, Devolvible, A devolver, Destino) muestra Vendido 10, Devuelto 4 y Devolvible 6; las líneas totalmente devueltas no aparecen; las cantidades son siempre en unidades base (la línea por presentación aparece con 20).

#### TC-DEV-014 — El máximo es lo devolvible

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-DEV-002] «El máximo es lo devolvible»; [CU-DEV-02] flujo básico y excepción; [RN-DEV-012] |

**Precondiciones:** Devolvible = 6.
**Pasos:** 1. Escribe «A devolver» 6 y sal del campo. 2. Escribe 7 y sal del campo. 3. Por API envía 7.
**Resultado esperado:** 6 es válido y la línea queda completa; 7 se ajusta a 6 al salir del campo; por API: «La cantidad supera lo que queda por devolver en alguna línea. Actualiza la pantalla e inténtalo de nuevo.».

#### TC-DEV-015 — Ninguna cantidad a devolver

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-DEV-002] «Ninguna cantidad»; [CU-DEV-02] excepciones; [RN-DEV-009], [RN-DEV-010] |

**Pasos:** deja todas las cantidades en 0 y haz clic en «Procesar devolución»; luego pon cantidad solo en una de dos líneas y procesa.
**Resultado esperado:** «Ingresa al menos una cantidad a devolver.» y no se envía nada; en el segundo caso solo se envían las líneas con cantidad mayor que 0.

#### TC-DEV-016 — Devolver todo lo pendiente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-002] «Devolver todo lo pendiente»; [CU-DEV-02] flujo básico |

**Pasos:** haz clic en «Devolver todo lo pendiente».
**Resultado esperado:** cada línea toma su cantidad devolvible.

#### TC-DEV-017 — Destino «Reingreso a venta»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DEV-002] «Destino «Reingreso a venta»»; [CU-DEV-02] flujo básico; [RN-DEV-023], [RN-STK-011] |

**Precondiciones:** venta de 100 unidades de `LOT-QA-P01` (stock actual 400); el destino por defecto es «Reingreso a venta».
**Pasos:** devuelve 30 unidades; revisa el stock del lote; repite con un lote que venció o quedó en cuarentena después de la venta.
**Resultado esperado:** el stock del mismo lote en la sucursal de la venta sube de 400 a 430; si el lote venció o quedó retenido, recupera su stock físico pero deja de ser vendible (no aparece en la lista de lotes del terminal).

#### TC-DEV-018 — Si la fila de stock ya no existe

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-DEV-002] «Si la fila de stock ya no existe»; [CU-DEV-02] alternativo A1; [RN-DEV-023] |

**Precondiciones:** venta cuyo lote ya no tiene fila de stock en la sucursal (borrada en base de datos de prueba).
**Pasos:** devuelve con «Reingreso a venta».
**Resultado esperado:** la devolución, la nota de crédito y el reembolso se registran igual; el stock no se repone, sin ningún aviso.
**Hallazgo:** el reintegro se omite sin aviso (01c3 hallazgo 6).

#### TC-DEV-019 — Destino «Cuarentena DIGEMID»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DEV-002] «Destino «Cuarentena DIGEMID»»; [CU-DEV-02] alternativo A2; [RN-DEV-024], [RN-CUA-011] |

**Precondiciones:** venta de `LOT-QA-P01` (stock actual conocido).
**Pasos:** devuelve 10 unidades con el destino «Cuarentena DIGEMID»; revisa el stock, el registro de cuarentena y «Lotes en cuarentena» del Inicio (`TC-DAS-030`).
**Resultado esperado:** la línea guarda el destino pero no se repone stock ni se crea registro de cuarentena; la nota de crédito y el reembolso se calculan igual; el lote no aparece en «Lotes en cuarentena» hasta que alguien lo aísle con la API (`TC-OPS-047`).
**Hallazgo:** «Cuarentena DIGEMID» no crea el aislamiento (01c3 hallazgo 4).

#### TC-DEV-020 — La misma línea repetida en la petición (API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-DEV-002] «La misma línea repetida en la petición (API)»; [CU-DEV-02] flujo básico; [RN-DEV-013] |

**Precondiciones:** una línea con Devolvible = 10.
**Pasos:** `POST /api/v1/devoluciones` con la misma línea dos veces con 6 y 6 unidades; luego con 5 y 5.
**Resultado esperado:** con 6 + 6 se rechaza porque las cantidades se suman (12 > 10): «La cantidad supera lo que queda por devolver en alguna línea. Actualiza la pantalla e inténtalo de nuevo.»; con 5 + 5 se acepta y cada repetición se guarda como una línea de devolución propia.

#### TC-DEV-021 — Una línea que no es de la venta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-DEV-002] «Una línea que no es de la venta»; [CU-DEV-02] excepciones; [RN-DEV-011] |

**Pasos:** `POST /api/v1/devoluciones` con una línea inexistente, con una línea de otra venta y con una línea de «Botica Otra Demo S.A.C.».
**Resultado esperado:** en los tres casos «Una de las líneas ya no pertenece a la venta.»; no se crea nada.

#### TC-DEV-022 — Cantidad con más de 2 decimales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-DEV-002] «Cantidad con más de 2 decimales»; [CU-DEV-02] excepciones; [RN-DEV-036] |

**Pasos:** `POST /api/v1/devoluciones` con cantidad 0.125.
**Resultado esperado:** **No verificado:** qué hace la base de datos con más de 2 decimales (columna `DECIMAL(8,2)`); no hay resultado esperado documentado.

### HU-DEV-003 — Reembolsar al cliente

#### TC-DEV-023 — Reembolso en efectivo de una devolución parcial

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DEV-003] «Reembolso en efectivo de una devolución parcial»; [CU-DEV-03] alternativo A1; [RN-DEV-016], [RN-DEV-017], [RN-DEV-019], [RN-CAJ-014] |

**Precondiciones:** venta de 100 unidades a S/ 0.50 sin seguro; caja de Carlos con apertura S/ 100.00 y sin otras operaciones.
**Pasos:** devuelve 30 unidades con método «Efectivo»; cierra la caja declarando 85 (esperado 100.00 − 15.00).
**Resultado esperado:** el «Reembolso estimado» del diálogo es S/ 15.00; el monto devuelto es 30 × 0.50 = 15.00 y el reembolsado también 15.00; el esperado de la caja baja en 15.00 (85.00) y declarando 85 la sesión queda «Cuadrada».

#### TC-DEV-024 — El método de reembolso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-003] «El método de reembolso»; [CU-DEV-03] flujo básico; [RN-DEV-018] |

**Pasos:** abre el selector «Método de reembolso»; por API envía la devolución sin método.
**Resultado esperado:** ofrece «Efectivo» (valor inicial), «Yape / Plin», «Tarjeta Débito/Crédito» y «Transferencia»; si la petición omite el método, el servidor asume efectivo.

#### TC-DEV-025 — Reembolso por un medio digital

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DEV-003] «Reembolso por un medio digital»; [CU-DEV-03] alternativo A2; [RN-DEV-018], [RN-DEV-019], [RN-CAJ-014] |

**Pasos:** devuelve 30 unidades (S/ 15.00) con «Yape / Plin»; revisa el detalle de la venta y el esperado de la caja.
**Resultado esperado:** el reembolso queda registrado con ese método y no resta del arqueo (el esperado no cambia).

#### TC-DEV-026 — El método de reembolso no tiene que coincidir con cómo pagó el cliente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DEV-003] «El método no tiene que coincidir con cómo pagó el cliente»; [CU-DEV-03] alternativo A3; [RN-DEV-018] |

**Precondiciones:** venta de S/ 30.00 pagada con «Tarjeta Débito/Crédito».
**Pasos:** devuelve S/ 15.00 con método «Efectivo»; mira el esperado de tu caja.
**Resultado esperado:** el sistema lo permite y el efectivo sale de tu caja (el esperado baja en 15.00) aunque la venta no haya entrado en efectivo.
**Hallazgo:** el reembolso no se compara con los pagos originales (RN-DEV-018).

#### TC-DEV-027 — El sistema solo registra el dinero, no lo mueve

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-DEV-003] «El sistema solo registra el dinero, no lo mueve»; [CU-DEV-03] flujo básico; [RN-DEV-019] |

**Precondiciones:** caja con apertura S/ 0.00 y sin efectivo.
**Pasos:** devuelve una venta cobrada en otra caja con reembolso en «Efectivo» de S/ 5.00.
**Resultado esperado:** la devolución se procesa: no se comprueba que el cajón tenga efectivo suficiente (el esperado queda en −5.00 y la caja luego no se podrá cerrar, `TC-CAJ-037`).

#### TC-DEV-028 — Redondeo del monto devuelto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-DEV-003] «Redondeo del monto devuelto»; [CU-DEV-03] flujo básico; [RN-DEV-016] |

**Precondiciones:** venta con precio S/ 0.50 por unidad.
**Pasos:** por API devuelve 0.25 unidades y, en otra venta, 0.27 unidades; revisa el monto devuelto.
**Resultado esperado:** 0.25 × 0.50 = 0.125 se redondea a 0.12 (a 2 decimales, mitad hacia el par); 0.27 × 0.50 = 0.135 se redondea a 0.14. Una cantidad con decimales puede ser rechazada o truncada por la columna `DECIMAL(8,2)`: ver `TC-DEV-022` (**No verificado**).

#### TC-DEV-029 — Venta con cobertura de seguro (solo API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DEV-003] «Venta con cobertura de seguro (solo API)»; [CU-DEV-03] flujo básico; [RN-DEV-016], [RN-DEV-017], [RN-DEV-021] |

**Precondiciones:** venta de 10 unidades a S/ 3.00 (total S/ 30.00) con cobertura de seguro de S/ 12.00 (se arma con `TC-VEN-141` adaptado).
**Pasos:** `POST /api/v1/devoluciones` de 4 unidades con método «Efectivo».
**Resultado esperado:** monto devuelto 12.00; proporción que pagó el cliente (30.00 − 12.00) ÷ 30.00 = 0.6; reembolso al cliente 12.00 × 0.6 = 7.20; la nota de crédito va por 12.00; el esperado de la caja baja en 7.20.

#### TC-DEV-030 — El reembolso estimado puede no coincidir con el real

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-003] «El estimado puede no coincidir con el real»; [CU-DEV-03] flujo básico; [RN-DEV-016] |

**Pasos:** en una venta con cobertura de seguro (por API) abre el diálogo «Devolver» y compara «Reembolso estimado» con el «Reembolsado (S/)» del detalle tras procesar.
**Resultado esperado:** el estimado (cantidad × precio, sin redondear ni descontar el seguro) puede diferir del real (por ejemplo 12.00 contra 7.20).
**Hallazgo:** 01c3 hallazgo 10.

#### TC-DEV-031 — Dónde se ve el reembolso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-003] «Dónde se ve el reembolso»; [CU-DEV-03] flujo básico; [RN-DEV-035], [RN-DEV-036] |

**Pasos:** tras `TC-DEV-023` abre el detalle de la venta.
**Resultado esperado:** «Devoluciones / Notas de crédito» muestra «Reembolsado (S/)» (15.00) y «Método de reembolso» (Efectivo).

#### TC-DEV-032 — Hace falta caja propia abierta para cualquier método

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-DEV-003] «Hace falta caja propia abierta para cualquier método»; [CU-DEV-03] excepciones; [RN-DEV-005] |

**Precondiciones:** el usuario no tiene caja abierta (el botón «Devolver» sí se ofrece).
**Pasos:** procesa una devolución con «Transferencia».
**Resultado esperado:** «Necesitas una caja abierta para procesar la devolución.»; no se crea la devolución, ni la nota de crédito, ni se repone el stock.

### HU-DEV-004 — Emitir la nota de crédito de la devolución

#### TC-DEV-033 — Una nota de crédito por devolución

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DEV-004] «Una nota de crédito por devolución»; [CU-DEV-04] flujo básico; [RN-DEV-020], [RN-DEV-033], [RN-SER-013] |

**Precondiciones:** serie de nota de crédito activa `BC01` con correlativo 0.
**Pasos:** procesa una devolución con monto devuelto S/ 15.00; revisa el detalle y la base de datos.
**Resultado esperado:** se emite la nota `BC01-00000001`, con estado `NoEnviado` y sin hash, XML, constancia ni URL; no hay devolución sin nota de crédito. Una segunda devolución emite `BC01-00000002`.

#### TC-DEV-034 — Importes de la nota de crédito

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-DEV-004] «Importes de la nota»; [CU-DEV-04] flujo básico; [RN-DEV-021] |

**Datos:** montos devueltos S/ 15.00 y S/ 12.00.
**Pasos:** procesa cada devolución y revisa la fila de la nota de crédito.
**Resultado esperado:**

| Monto devuelto | Base gravada | IGV | Exonerado |
|---|---|---|---|
| 15.00 | 12.71 | 2.29 | 0.00 |
| 12.00 | 10.17 | 1.83 | 0.00 |

La nota va por el 100 % del crédito, incluida la parte de la aseguradora (`TC-DEV-029`).

#### TC-DEV-035 — Datos del cliente de la nota

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-004] «Datos del cliente de la nota»; [CU-DEV-04] flujo básico; [RN-DEV-022] |

**Pasos:** devuelve una venta a «Rosa Quispe Demo» (DNI `45781236`) y otra al cliente eventual; compara la nota con su boleta.
**Resultado esperado:** la nota repite el tipo de documento, el número y el nombre del cliente de la boleta que revierte (DNI `45781236` y «Rosa Quispe Demo»; «CLIENTE EVENTUAL» con «00000000»).

#### TC-DEV-036 — Qué ve el cajero de la nota de crédito

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-004] «Qué ve el cajero de la nota»; [CU-DEV-04] flujo básico; [RN-DEV-022], [RN-CMP-013] |

**Pasos:** abre el detalle de la venta tras una devolución.
**Resultado esperado:** «Devoluciones / Notas de crédito» muestra «Nota de crédito» (el número o «—»), «Fecha» («dd/MM/yyyy HH:mm»), «Reembolsado (S/)» y «Método de reembolso»; la nota no cambia el código de venta (sigue `B001-…`) ni se toma como «el comprobante» de la venta.
**Hallazgo:** el SPA no imprime ni muestra la nota de crédito como documento (RN-DEV-022).

#### TC-DEV-037 — La venta no tiene boleta ni factura

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-DEV-004] «La venta no tiene boleta ni factura»; [CU-DEV-04] excepciones; [RN-DEV-006] |

**Pasos:** por API, `POST /api/v1/devoluciones` sobre una venta «Pendiente» (sin boleta).
**Resultado esperado:** «La venta no tiene boleta ni factura emitida que acreditar.»; no se crea nada.

#### TC-DEV-038 — La sucursal no tiene serie de nota de crédito activa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-DEV-004] «La sucursal no tiene serie de nota de crédito activa»; [CU-DEV-04] excepciones; [RN-DEV-007] |

**Precondiciones:** una sucursal de prueba sin serie de nota de crédito activa; venta «Completada» en ella.
**Pasos:** procesa una devolución.
**Resultado esperado:** «La sucursal no tiene una serie de nota de crédito activa. Configúrala en Series Fiscales.»; no se consume correlativo; no se crea la devolución.

#### TC-DEV-039 — La nota se emite al final

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-DEV-004] «La nota se emite al final»; [CU-DEV-04] excepciones; [RN-DEV-020], [RN-DEV-028] |

**Pasos:** provoca un rechazo de validación (cantidad mayor que lo devolvible) con la serie `BC01` en correlativo N.
**Resultado esperado:** el correlativo de `BC01` sigue en N: se consume solo cuando ya se validó todo lo demás; si el guardado lanza una excepción, el aumento también se revierte. **No verificado en ejecución** para el caso de excepción de guardado.

#### TC-DEV-040 — Boletas y notas comparten numeración única

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-004] «Boletas y notas comparten numeración única»; [CU-DEV-04] flujo básico; [RN-DEV-037], [RN-CMP-015] |

**Pasos:** en la base de datos revisa el índice único por serie y correlativo de la tabla de comprobantes y las filas de una venta con dos devoluciones.
**Resultado esperado:** el índice único vale también para las notas; la venta tiene una boleta (o factura) y dos notas de crédito.

#### TC-DEV-041 — Estructura de las devoluciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-DEV-004] «Estructura de las devoluciones»; [CU-DEV-04] excepciones; [RN-DEV-035], [RN-DEV-036], [RN-DEV-033], [RN-DEV-034] |

**Pasos:** revisa en la base la estructura de las tablas de devoluciones (solo lectura); por API prueba un motivo de 3 caracteres y un destino fuera de los permitidos.
**Resultado esperado:** la base exige comprobante de origen, empleado que autoriza, motivo (2 caracteres) y sustento (hasta 255); las líneas exigen cantidad mayor que 0 y un destino solo `Reingreso_Venta` o `Cuarentena_DIGEMID`; el monto reembolsado no es negativo y el método se guarda como EFECTIVO, YAPE_PLIN, TARJETA o TRANSFERENCIA; una venta admite varias devoluciones. Una violación se ve como «Error al procesar la devolución.».

### HU-DEV-005 — Indicar el motivo SUNAT y el sustento

#### TC-DEV-042 — El motivo se propone según las cantidades y solo hay dos motivos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-005] «El motivo se propone según las cantidades», «Solo hay dos motivos»; [CU-DEV-05] flujo básico; [RN-DEV-014] |

**Pasos:** 1. Abre «Devolver» y pon en cada línea su cantidad devolvible completa. 2. Reduce una cantidad. 3. Cambia el motivo a mano y luego cambia otra cantidad. 4. Por API envía el motivo `99`.
**Resultado esperado:** con todo devolvible el «Motivo SUNAT» propone «06 — Devolución total»; en cualquier otro caso «07 — Devolución por ítem»; el cambio manual se pisa con el siguiente cambio de cantidades; el selector solo ofrece esos dos motivos. El servidor acepta cualquier par de caracteres (`99` se registra): no valida contra el catálogo SUNAT ni que «06» coincida con una devolución total.
**Hallazgo:** el motivo no se valida contra ningún catálogo (01c3 hallazgo 6).

#### TC-DEV-043 — Sustento obligatorio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-DEV-005] «Sustento obligatorio»; [CU-DEV-05] excepciones; [RN-DEV-015] |

**Pasos:** deja «Sustento» vacío y haz clic en «Procesar devolución».
**Resultado esperado:** «El sustento es obligatorio.» y no se envía nada.

#### TC-DEV-044 — Sustento de solo espacios y largo del sustento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-DEV-005] «Sustento de solo espacios», «Largo del sustento»; [CU-DEV-05] excepciones; [RN-DEV-015] |

**Pasos:** 1. Escribe solo espacios en «Sustento» y procesa. 2. Pega 300 caracteres. 3. Escribe un sustento de 255 caracteres con espacios al inicio y al final y procesa.
**Resultado esperado:** 1: el botón no hace nada y no aparece mensaje. 2: el campo corta la escritura en 255 caracteres. 3: se recorta de espacios al enviar y la devolución se registra.

#### TC-DEV-045 — Motivo de un solo carácter (API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-DEV-005] «Motivo de un solo carácter (API)»; [CU-DEV-05] excepciones; [RN-DEV-014], [RN-DEV-033] |

**Pasos:** `POST /api/v1/devoluciones` con motivo `6` (1 carácter).
**Resultado esperado:** el validador lo acepta, la entidad lo rechaza y el SPA mostraría «Error al procesar la devolución.»; no se crea nada.
**Hallazgo:** el mensaje de la entidad está en inglés y no llega al usuario.

#### TC-DEV-046 — Texto con HTML en el sustento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-DEV-005] «Texto con HTML»; [CU-DEV-05] flujo básico; [RN-GEN-010] |

**Pasos:** escribe en «Sustento» `<b>Mal estado</b><script>alert(1)</script>` y procesa; revisa el sustento guardado.
**Resultado esperado:** el servidor limpia las etiquetas HTML antes de validar y guardar (el sustento guardado no contiene etiquetas ni se ejecuta ningún script). **No verificado:** el texto exacto resultante tras la limpieza.

### HU-DEV-006 — Saber por qué no se puede devolver una venta

#### TC-DEV-047 — Mensajes de rechazo de una devolución (esquema)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-DEV-006] «Mensajes de rechazo de una devolución»; [CU-DEV-06] excepciones; [RN-DEV-002], [RN-DEV-003], [RN-DEV-004], [RN-DEV-005], [RN-DEV-006], [RN-DEV-007], [RN-DEV-008] |

**Pasos:** provoca cada situación de la tabla (por API cuando el SPA no deja llegar).
**Resultado esperado:**

| Situación | Mensaje |
|---|---|
| No tengo caja propia abierta en la sucursal | Necesitas una caja abierta para procesar la devolución. |
| La venta es de otra sucursal (403) | Solo puedes devolver ventas de tu propia sucursal. |
| La venta ya no existe | La venta ya no existe. |
| La venta no tiene boleta ni factura | La venta no tiene boleta ni factura emitida que acreditar. |
| La sucursal no tiene serie de nota de crédito | La sucursal no tiene una serie de nota de crédito activa. Configúrala en Series Fiscales. |
| La venta está «Anulada» o «Devuelta» | Solo se pueden devolver ventas completadas. |
| La cantidad supera lo devolvible | La cantidad supera lo que queda por devolver en alguna línea. Actualiza la pantalla e inténtalo de nuevo. |
| Una línea no pertenece a la venta | Una de las líneas ya no pertenece a la venta. |
| Otra operación modificó los datos | Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo. |
| Cualquier otro error | Error al procesar la devolución. |

#### TC-DEV-048 — Orden en que se evalúan las condiciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-DEV-006] «Orden en que se evalúan las condiciones»; [CU-DEV-06] excepciones; [RN-DEV-005] |

**Precondiciones:** el usuario no tiene caja abierta y la venta no tiene comprobante.
**Pasos:** intenta devolver (por API).
**Resultado esperado:** solo se ve «Necesitas una caja abierta para procesar la devolución.», porque esa comprobación va antes que las de comprobante, estado y cantidades.

#### TC-DEV-049 — Una venta en espera no se devuelve

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-DEV-006] «Una venta en espera»; [CU-DEV-06] alternativo A1; [RN-DEV-008], [RN-DEV-006] |

**Pasos:** abre el detalle de una venta «Pendiente»; por API intenta devolverla con caja abierta.
**Resultado esperado:** no se ofrece «Devolver»; por API: «La venta no tiene boleta ni factura emitida que acreditar.».

#### TC-DEV-050 — El administrador tampoco devuelve ventas de otra sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-DEV-006] «El Admin tampoco devuelve ventas de otra sucursal»; [CU-DEV-06] excepciones; [RN-DEV-004] |

**Precondiciones:** venta de «Sucursal Demo Norte»; Ana (Admin) con sesión en «Sede Principal» y caja abierta.
**Pasos:** por API, Ana procesa la devolución de esa venta.
**Resultado esperado:** HTTP 403; «Solo puedes devolver ventas de tu propia sucursal.»; no se crea nada.

#### TC-DEV-051 — No hay plazo ni límite de devoluciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-006] «No hay plazo ni límite de devoluciones»; [CU-DEV-06] alternativo A2; [RN-DEV-008] |

**Precondiciones:** venta antigua (cambia su fecha en base de datos de prueba a más de un año) con saldo devolvible.
**Pasos:** procesa una devolución parcial y luego otra.
**Resultado esperado:** el sistema permite seguir devolviendo mientras haya cantidad devolvible.
**Hallazgo:** no hay plazo máximo ni límite de devoluciones por venta (RN-DEV-008).

### HU-DEV-007 — Ver las devoluciones de una venta

#### TC-DEV-052 — Ver lo devuelto en la línea y las devoluciones de la venta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-007] «Ver lo devuelto en la línea», «Ver las devoluciones de la venta»; [CU-DEV-07] flujo básico; [RN-DEV-032], [RN-VEN-074] |

**Precondiciones:** venta de 100 unidades con dos devoluciones (30 y 20 unidades).
**Pasos:** abre `/pos/{id}`.
**Resultado esperado:** la columna «Devuelto» de «Productos Vendidos» muestra 50 (30 + 20); «Devoluciones / Notas de crédito» muestra una fila por devolución, de la más antigua a la más reciente, con «Nota de crédito» (número o «—»), «Fecha», «Reembolsado (S/)» y «Método de reembolso».

#### TC-DEV-053 — Venta sin devoluciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-DEV-007] «Venta sin devoluciones»; [CU-DEV-07] flujo básico; [RN-DEV-032] |

**Pasos:** abre el detalle de una venta sin devoluciones.
**Resultado esperado:** la sección «Devoluciones / Notas de crédito» no aparece.

#### TC-DEV-054 — El dashboard cuenta las devoluciones del día

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-007] «El dashboard cuenta las devoluciones del día»; [CU-DEV-07] flujo básico; [RN-DAS-023] |

**Pasos:** procesa dos devoluciones hoy y mira el Inicio (`TC-DAS-032`).
**Resultado esperado:** el Inicio muestra el número y el monto de las devoluciones de hoy.

### HU-DEV-008 — Consultar devoluciones por API

#### TC-DEV-055 — Listar devoluciones por API

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-008] «Listar devoluciones»; [CU-DEV-08] flujo básico; [RN-DEV-030] |

**Precondiciones:** token con `Devoluciones:Leer`; varias devoluciones en «Sede Principal» y en «Sucursal Demo Norte».
**Pasos:** `GET /api/v1/devoluciones?fechaDesde=<hoy>&fechaHasta=<hoy>` con token de Carlos. **No verificado:** los nombres exactos de los parámetros de fecha (consulta el controlador).
**Resultado esperado:** HTTP 200 con las devoluciones de las sucursales a las que Carlos tiene acceso, ordenadas por fecha descendente; cada fila trae solo identificador, comprobante de origen, empleado que autoriza, fecha, motivo SUNAT y sustento (no monto, método de reembolso, nota de crédito, líneas ni destino); paginación de 10 por defecto; sin las devoluciones de «Sucursal Demo Norte» ni de otra empresa.

#### TC-DEV-056 — Filtro por empleado vacío en el listado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-DEV-008] «Filtro por empleado vacío»; [CU-DEV-08] excepciones; [RN-DEV-030] |

**Pasos:** `GET /api/v1/devoluciones` con el empleado que autoriza igual a `00000000-0000-0000-0000-000000000000`.
**Resultado esperado:** HTTP 400, `code` `GetDevolucionesQuery.EmpleadoAutorizaId`.

#### TC-DEV-057 — Consultar una devolución por identificador

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DEV-008] «Consultar una devolución»; [CU-DEV-08] flujo básico; [RN-DEV-031] |

**Pasos:** `GET /api/v1/devoluciones/{id}` de una devolución de una sucursal accesible y de una de «Sucursal Demo Norte» (sin acceso).
**Resultado esperado:** la primera: HTTP 200 con el objeto de resultado completo y los mismos seis campos en `value`; la segunda: no se devuelve (el comportamiento exacto, 404 o 403, es **No verificado** para sucursal no accesible).
**Hallazgo:** `GET Devoluciones/{id}` devuelve el objeto `Result` completo y el listado el valor directo (01c3 hallazgo 8).

#### TC-DEV-058 — La devolución no existe

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-DEV-008] «La devolución no existe»; [CU-DEV-08] excepciones; [RN-DEV-031] |

**Pasos:** `GET /api/v1/devoluciones/{id}` con un GUID inexistente.
**Resultado esperado:** HTTP 404, `code` `Devolucion.NotFound`.

#### TC-DEV-059 — No existe exportación de devoluciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-DEV-008] «No existe exportación»; [CU-DEV-08] excepciones; [RN-DEV-032] |

**Pasos:** `POST /api/v1/devoluciones/exportar`.
**Resultado esperado:** la ruta no existe (404 o 405: el código exacto es **No verificado**); no hay endpoint para exportar devoluciones.

## 7. Casos del módulo Dashboard (`TC-DAS`)

Los casos del Inicio dependen de la fecha y de los datos del día: antes de ejecutarlos arma las ventas de «hoy» y «ayer» indicadas en cada caso (ayer se puede simular cambiando la fecha de la venta en una base de pruebas). **No verificado:** el separador decimal exacto que muestra el Inicio depende del idioma de la aplicación.

### HU-DAS-001 — Ver el resumen de ventas de hoy

#### TC-DAS-001 — «Vendido hoy» contra ayer

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-001] «Vendido hoy contra ayer»; [CU-DAS-01] flujo básico; [RN-DAS-002], [RN-DAS-004], [RN-DAS-006] |

**Precondiciones:** hoy 2 ventas por S/ 80.00 en total y ayer S/ 64.00; `Ventas:Leer`.
**Pasos:** abre `/dashboard`.
**Resultado esperado:** «Vendido hoy» S/ 80.00 y la variación «+25.0% vs. ayer · Ayer: S/ 64.00» con una flecha hacia arriba (el separador decimal depende del idioma, **No verificado**).

#### TC-DAS-002 — Ayer no vendió nada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-DAS-001] «Ayer no vendió nada»; [CU-DAS-01] flujo básico; [RN-DAS-006] |

**Precondiciones:** ventas hoy y ninguna ayer.
**Pasos:** abre `/dashboard`.
**Resultado esperado:** solo se ve «Ayer: S/ 0.00», sin porcentaje.

#### TC-DAS-003 — Comprobantes y ticket promedio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-001] «Comprobantes y ticket promedio»; [CU-DAS-01] flujo básico; [RN-DAS-007] |

**Pasos:** con 2 ventas de hoy por S/ 80.00 mira «Comprobantes»; luego en una sucursal sin ventas.
**Resultado esperado:** «Comprobantes» 2 y «Ticket promedio: S/ 40.00»; sin ventas el ticket promedio es 0.

#### TC-DAS-004 — Qué ventas cuentan en las cifras

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DAS-001] «Qué ventas cuentan»; [CU-DAS-01] flujo básico; [RN-DAS-005] |

**Precondiciones:** hoy una venta «Completada» de S/ 50.00, una «Devuelta» de S/ 30.00 (ya reembolsada), una «Anulada» de S/ 20.00 y una «Pendiente» de S/ 10.00; con seguro, una venta de S/ 100.00 con cobertura de S/ 40.00.
**Pasos:** abre `/dashboard` y suma.
**Resultado esperado:** cuentan «Completada» y «Devuelta» (50.00 + 30.00 + 100.00 de la venta con seguro = 180.00: el monto es el total bruto antes de restar la cobertura) y no cuentan «Anulada» ni «Pendiente»; las devoluciones no se restan; se lee «Los montos vendidos son brutos: las devoluciones se muestran aparte.».
**Hallazgo:** «Vendido hoy» incluye las ventas «Devuelta» y los montos siguen siendo brutos tras una devolución (01c3 hallazgo 7).

#### TC-DAS-005 — Mis ventas de hoy

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-001] «Mis ventas de hoy»; [CU-DAS-01] flujo básico; [RN-DAS-008] |

**Pasos:** abre `/dashboard` con Carlos tras hacer 2 ventas de S/ 20.00 y con una cuenta sin empleado.
**Resultado esperado:** Carlos ve «Mis ventas de hoy» con S/ 40.00 y «2 comprobantes»; sin empleado en la sesión la tarjeta no aparece.

#### TC-DAS-006 — Ventas en espera

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-001] «Ventas en espera»; [CU-DAS-01] flujo básico; [RN-DAS-009] |

**Precondiciones:** dos ventas «Pendiente» (una de hace varios días) por S/ 30.00 y S/ 20.00.
**Pasos:** abre `/dashboard`.
**Resultado esperado:** «Ventas en espera» muestra 2 y «Por cobrar: S/ 50.00» (cualquier fecha).

#### TC-DAS-007 — Qué es «hoy» y el límite de 36 horas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-DAS-001] «Qué es "hoy"»; [CU-DAS-01] excepciones; [RN-DAS-002], [RN-DAS-004] |

**Pasos:** 1. Abre el Inicio en una zona horaria (cambia la zona horaria del equipo) y anota «Vendido hoy». 2. Cambia el reloj del equipo 40 horas adelante y pulsa «Actualizar».
**Resultado esperado:** «hoy» es el día local del navegador, de medianoche a medianoche (24 horas exactas), enviado como parámetro; dos usuarios en zonas distintas ven «hoy» distinto; con un «hoy» a más de 36 horas del servidor, la sección muestra «No se pudo cargar esta sección.» con «Reintentar».
**Hallazgo:** un reloj desfasado más de 36 horas deja sin datos las secciones de ventas, inventario y devoluciones (HU-DAS-001).

### HU-DAS-002 — Ver lo cobrado por método de pago y por sucursal

#### TC-DAS-008 — Cobrado por método de pago

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DAS-002] «Cobrado por método de pago»; [CU-DAS-02] flujo básico; [RN-DAS-010] |

**Precondiciones:** hoy la venta A de S/ 50.00 pagada con S/ 60.00 en efectivo (vuelto S/ 10.00) y la venta B de S/ 30.00 con S/ 20.00 en tarjeta y S/ 10.00 en Yape / Plin.
**Pasos:** abre `/dashboard`.
**Resultado esperado:** cuatro filas en orden: «Efectivo» S/ 50.00, «Yape / Plin» S/ 10.00, «Tarjeta» S/ 20.00 y «Transferencia» S/ 0.00; cada barra es el porcentaje sobre la suma de las cuatro filas (62.5 %, 12.5 %, 25.0 % y 0.0 %); se lee «El efectivo ya descuenta el vuelto entregado.».

#### TC-DAS-009 — Las ventas anuladas no suman y los reembolsos no restan

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-002] «Las ventas anuladas no suman y los reembolsos no restan»; [CU-DAS-02] flujo básico; [RN-DAS-010] |

**Pasos:** anula una venta de hoy en efectivo y procesa una devolución con reembolso en efectivo; abre `/dashboard`.
**Resultado esperado:** los pagos de la venta anulada no se suman; los reembolsos de devoluciones no se restan de esta tarjeta.

#### TC-DAS-010 — Ventas por sucursal con varias sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-002] «Ventas por sucursal para varias sucursales»; [CU-DAS-02] flujo básico; [RN-DAS-003], [RN-DAS-011] |

**Precondiciones:** ventas de hoy en «Sede Principal» y ninguna en «Sucursal Demo Norte»; Ana (Admin).
**Pasos:** abre `/dashboard` con Ana y con un cajero con dos sucursales asignadas.
**Resultado esperado:** tabla «Ventas por sucursal» con «Sucursal», «Comprobantes» y «Vendido», una fila por sucursal ordenada por nombre (con 0 si no vendió); el pie dice «Total empresa» para el administrador y «Total de tus sucursales» para el otro; el pie suma lo mismo que «Vendido hoy» y «Comprobantes».

#### TC-DAS-011 — Una sola sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-DAS-002] «Una sola sucursal»; [CU-DAS-02] alternativo A1; [RN-DAS-011] |

**Pasos:** abre `/dashboard` con Carlos (alcance «Sede Principal») y con Ana en una empresa de una sola sucursal.
**Resultado esperado:** no se ve «Ventas por sucursal».

#### TC-DAS-012 — Alcance sin sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-DAS-002] «Alcance sin sucursales»; [CU-DAS-02] alternativo A2; [RN-DAS-003] |

**Pasos:** abre `/dashboard` con un usuario no administrador sin sucursal base accesible.
**Resultado esperado:** todas las cifras valen 0. **No verificado:** cómo preparar un usuario con alcance vacío desde la interfaz.

### HU-DAS-003 — Ver la tendencia de 7 días y los productos más vendidos

#### TC-DAS-013 — Tendencia de siete días

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-DAS-003] «Tendencia de siete días»; [CU-DAS-03] flujo básico; [RN-DAS-012] |

**Precondiciones:** ventas en varios de los últimos 7 días.
**Pasos:** abre `/dashboard` y mira «Ventas de los últimos 7 días».
**Resultado esperado:** 7 barras, de la más antigua a hoy, cada una con su monto bruto y su número de ventas; fechas «dd/MM» y la última «Hoy» (incluye el día en curso); cada barra proporcional al día de mayor monto.

#### TC-DAS-014 — Productos más vendidos en 7 días

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-003] «Productos más vendidos»; [CU-DAS-03] flujo básico; [RN-DAS-013] |

**Precondiciones:** ventas de 7 productos distintos en la semana, incluida 1 venta de 2 «Caja x10» de Paracetamol (20 unidades base) y dos productos empatados en unidades.
**Pasos:** abre `/dashboard` y mira «Productos más vendidos en 7 días».
**Resultado esperado:** hasta 5 productos con «{unidades} unidades · S/ {monto}»; las unidades son base (2 cajas de 10 cuentan 20); los empates se resuelven por el identificador del producto.

#### TC-DAS-015 — Sin ventas en la semana

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-DAS-003] «Sin ventas en la semana»; [CU-DAS-03] alternativo A1 |

**Pasos:** abre `/dashboard` con una sucursal sin ventas en 7 días.
**Resultado esperado:** «Aún no hay ventas en los últimos 7 días.».

#### TC-DAS-016 — Líneas cuyo lote o producto ya no existe

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-DAS-003] «Líneas cuyo lote o producto ya no existe»; [CU-DAS-03] alternativo A2; [RN-DAS-013] |

**Pasos:** en la base de pruebas elimina (borrado lógico) el lote o producto de una línea vendida esta semana y abre `/dashboard`.
**Resultado esperado:** esa línea se omite del ranking, sin error.

### HU-DAS-004 — Ver mi caja y las cajas abiertas

#### TC-DAS-017 — Tengo una caja abierta: efectivo esperado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DAS-004] «Tengo una caja abierta»; [CU-DAS-04] flujo básico; [RN-DAS-015], [RN-CAJ-011], [RN-CAJ-013] |

**Precondiciones:** caja de Carlos con apertura S/ 100.00 y ventas en efectivo netas de S/ 50.00 (ver `TC-CAJ-028`).
**Pasos:** abre `/dashboard` y haz clic en «Ver mi caja».
**Resultado esperado:** «Tienes una caja abierta» con «Sucursal», «Abierta desde», «Monto de apertura» S/ 100.00 y «Efectivo esperado» S/ 150.00; «Ver mi caja» abre tu sesión.

#### TC-DAS-018 — No tengo caja abierta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-004] «No tengo caja abierta»; [CU-DAS-04] alternativo A1; [RN-DAS-015] |

**Pasos:** abre `/dashboard` con un usuario sin caja abierta, con y sin `POS:AperturarCaja`.
**Resultado esperado:** «No tienes una caja abierta» y, con `POS:AperturarCaja`, el botón «Abrir caja» que va a `/sesiones-caja/nueva`.

#### TC-DAS-019 — El esperado es solo de mi caja

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-DAS-004] «El esperado es solo de mi caja»; [CU-DAS-04] alternativo A2 (disparador); [RN-DAS-015], [RN-CAJ-012], [RN-CAJ-014] |

**Precondiciones:** dos cajas abiertas en la sucursal (Carlos y otro cajero) con ventas distintas; una devolución con reembolso en efectivo desde la caja de Carlos.
**Pasos:** abre `/dashboard` con cada cajero y compara con la fórmula de `TC-CAJ-029`.
**Resultado esperado:** cada uno ve el esperado de su propia sesión con la misma fórmula del cierre (apertura + efectivo neto − anulaciones − reembolsos en efectivo), sin depender de las sucursales de su alcance; con una sesión sin empleado la cifra es nula y no se muestra.

#### TC-DAS-020 — Cajas abiertas de la sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-004] «Cajas abiertas de la sucursal»; [CU-DAS-04] flujo básico; [RN-DAS-016] |

**Precondiciones:** cajas abiertas de otros cajeros en el alcance.
**Pasos:** abre `/dashboard` y haz clic en «Ver cajas abiertas».
**Resultado esperado:** «Cajas abiertas ({n})» con «Cajero», «Sucursal» y «Abierta desde»; hasta 50 sesiones, de la más reciente a la más antigua; el botón abre `/sesiones-caja` filtrada por estado «Abierta».
**Hallazgo:** la API entrega el monto de apertura de las cajas ajenas a cualquiera con `POS:Leer`, aunque la pantalla no lo muestre (01c3 hallazgo 9).

#### TC-DAS-021 — Solo mi caja

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-DAS-004] «Solo mi caja»; [CU-DAS-04] alternativo A2; [RN-DAS-016] |

**Pasos:** abre `/dashboard` cuando la única sesión abierta del alcance es la tuya.
**Resultado esperado:** la tabla «Cajas abiertas» no aparece.

### HU-DAS-005 — Ver las alertas de inventario

#### TC-DAS-022 — Lotes vencidos con stock

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-005] «Lotes vencidos con stock»; [CU-DAS-05] flujo básico; [RN-DAS-017], [RN-DAS-018], [RN-LOT-006] |

**Precondiciones:** un lote vencido con stock, uno que caduca hoy y uno vencido en cuarentena; `Inventarios:Leer`.
**Pasos:** abre `/dashboard` y mira «Lotes vencidos con stock».
**Resultado esperado:** cuenta las filas de stock con cantidad mayor que 0 y caducidad anterior al inicio de hoy; el lote que caduca hoy no está vencido; cuentan también los lotes retenidos en cuarentena.

#### TC-DAS-023 — Tramos de vencimiento (esquema)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-DAS-005] «Tramos de vencimiento (hoy es 08/10)»; [CU-DAS-05] flujo básico; [RN-DAS-019] |

**Precondiciones:** (hoy es 08/10) filas de stock con cantidad mayor que 0 que caducan el 08/10, el 07/11, el 08/11 y el 08/01 siguiente.
**Pasos:** abre `/dashboard` y mira las tarjetas de vencimiento.
**Resultado esperado:**

| Caduca el | Cuenta en |
|---|---|
| 08/10 | «Lotes que vencen en 30 días o menos» |
| 07/11 (hoy + 30 d) | «Lotes que vencen en 30 días o menos» |
| 08/11 | «Lotes que vencen en 31 a 60 días» |
| 08/01 siguiente | ninguno (pasa de hoy + 90 días) |

#### TC-DAS-024 — Las cifras cuentan filas de stock, no lotes únicos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-DAS-005] «Las cifras cuentan filas de stock, no lotes únicos»; [CU-DAS-05] flujo básico; [RN-DAS-017] |

**Precondiciones:** un mismo lote vencido con stock en «Sede Principal» y en «Sucursal Demo Norte»; Ana (Admin).
**Pasos:** abre `/dashboard` con Ana.
**Resultado esperado:** el lote cuenta dos veces en «Lotes vencidos con stock».

#### TC-DAS-025 — Productos sin stock

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-005] «Productos sin stock»; [CU-DAS-05] flujo básico; [RN-DAS-020] |

**Precondiciones:** un producto con filas de stock solo vencidas o en cuarentena en la sucursal.
**Pasos:** abre `/dashboard`.
**Resultado esperado:** «Productos sin stock» cuenta los pares (sucursal, producto) con alguna fila de stock pero ninguna vendible (vendible: cantidad mayor que 0, lote no vencido y sin retención en cuarentena); se lee «Sin unidades vendibles, por sucursal».

#### TC-DAS-026 — Productos con stock bajo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-DAS-005] «Productos con stock bajo»; [CU-DAS-05] flujo básico; [RN-DAS-021], [RN-STK-017] |

**Precondiciones:** un producto con un lote vendible de 6, otro vencido de 20 y otro en cuarentena de 15 en la misma sucursal.
**Pasos:** abre `/dashboard`.
**Resultado esperado:** su stock vendible es 6; cuenta en «Productos con stock bajo» («10 unidades o menos, por sucursal») y no en «Productos sin stock»; el umbral 10 es fijo en el código.

#### TC-DAS-027 — Botón «Ver lotes»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-DAS-005] «Botón "Ver lotes"»; [CU-DAS-05] flujo básico; [RN-DAS-014] |

**Pasos:** en una tarjeta de vencimiento haz clic en «Ver lotes».
**Resultado esperado:** se abre el stock por sucursal con la caducidad ya filtrada y «solo con stock».

### HU-DAS-006 — Ver los lotes en cuarentena

#### TC-DAS-028 — Contar lotes retenidos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-006] «Contar lotes retenidos»; [CU-DAS-06] flujo básico; [RN-DAS-022] |

**Precondiciones:** dos registros de cuarentena «Retenido» de lotes distintos en «Sede Principal» (ver `TC-OPS-040`); `DIGEMID:Leer`.
**Pasos:** abre `/dashboard`.
**Resultado esperado:** «Lotes en cuarentena» con 2 y «Retenidos: no se pueden vender».

#### TC-DAS-029 — Un lote aislado varias veces cuenta una vez por sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-DAS-006] «Un lote aislado varias veces cuenta una vez por sucursal»; [CU-DAS-06] flujo básico; [RN-DAS-022], [RN-CUA-007] |

**Pasos:** aísla dos veces el mismo lote en la misma sucursal; abre `/dashboard`.
**Resultado esperado:** cuenta una vez.

#### TC-DAS-030 — Una devolución a cuarentena no aparece sola

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-006] «Una devolución a cuarentena no aparece sola»; [CU-DAS-06] alternativo A1; [RN-DAS-022] |

**Pasos:** procesa una devolución con destino «Cuarentena DIGEMID» (`TC-DEV-019`) y mira «Lotes en cuarentena».
**Resultado esperado:** el lote no cuenta hasta que alguien registre el aislamiento por la API (`TC-OPS-047`).

#### TC-DAS-031 — La tarjeta de cuarentena exige DIGEMID:Leer

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-DAS-006] «Solo con permiso»; [CU-DAS-06] alternativo A2; [RN-DAS-001] |

| Permiso que falta | Qué no se ve |
|---|---|
| `DIGEMID:Leer` | La tarjeta «Lotes en cuarentena» no se pide ni se dibuja (sin petición `GET /api/v1/dashboard/cuarentena`) |

**Pasos:** abre `/dashboard` con un rol sin `DIGEMID:Leer`.
**Resultado esperado:** según la tabla.

### HU-DAS-007 — Ver las devoluciones de hoy

#### TC-DAS-032 — Devoluciones del día

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-007] «Devoluciones del día»; [CU-DAS-07] flujo básico; [RN-DAS-023] |

**Precondiciones:** hoy dos devoluciones, una parcial con reembolso de S/ 15.00 en efectivo y una total con S/ 35.00 por Yape / Plin; `Devoluciones:Leer`.
**Pasos:** abre `/dashboard`.
**Resultado esperado:** «Devoluciones» 2 y «Monto reembolsado: S/ 50.00»; cuenta parciales y totales por igual, de cualquier método de reembolso.

#### TC-DAS-033 — Solo las devoluciones de mis sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-DAS-007] «Solo las de mis sucursales»; [CU-DAS-07] flujo básico; [RN-DAS-023], [RN-DAS-003] |

**Precondiciones:** devoluciones de hoy en «Sede Principal» y en «Sucursal Demo Norte».
**Pasos:** abre `/dashboard` con Carlos y con Ana (Admin).
**Resultado esperado:** Carlos cuenta solo las devoluciones cuya venta de origen es de su alcance; Ana cuenta las de toda la empresa; ninguno cuenta las de «Botica Otra Demo S.A.C.».

#### TC-DAS-034 — El bloque de devoluciones depende del permiso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Baja | [HU-DAS-007] «El bloque depende del permiso, no de una pantalla»; [CU-DAS-07] flujo básico; [RN-DAS-001] |

| Permiso que falta | Qué no se ve |
|---|---|
| `Devoluciones:Leer` | El bloque «Devoluciones de hoy» no se pide ni se dibuja |

**Pasos:** abre `/dashboard` con y sin `Devoluciones:Leer`.
**Resultado esperado:** con el permiso se muestra el bloque aunque el módulo no tenga pantalla de listado; sin él, no.

### HU-DAS-008 — Ver solo lo que mi rol y mis sucursales permiten

#### TC-DAS-035 — Saludo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-DAS-008] «Saludo»; [CU-DAS-08] flujo básico |

**Pasos:** inicia sesión con `carlos.cajero`.
**Resultado esperado:** «Hola, Carlos Cajero Demo» y «Este es el resumen de hoy.».

#### TC-DAS-036 — Un bloque por permiso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-DAS-008] «Un bloque por permiso»; [CU-DAS-08] flujo básico; [RN-DAS-001] |

**Pasos:** abre `/dashboard` con un rol con los cinco permisos y, repitiendo, quitando uno cada vez.
**Resultado esperado:**

| Permiso | Bloque |
|---|---|
| `Ventas:Leer` | Ventas |
| `POS:Leer` | Caja |
| `Inventarios:Leer` | Inventario |
| `DIGEMID:Leer` | Tarjeta de cuarentena del bloque de inventario |
| `Devoluciones:Leer` | Devoluciones |

Si falta un permiso, ese bloque no se pide (sin petición al endpoint) ni se dibuja.

#### TC-DAS-037 — Sin ningún permiso de lectura

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-DAS-008] «Sin ningún permiso de lectura»; [CU-DAS-08] alternativo A1; [RN-DAS-001] |

**Pasos:** abre `/dashboard` con un rol sin ninguno de los cinco permisos de lectura.
**Resultado esperado:** «Tu usuario no tiene indicadores disponibles aquí. Usa el menú lateral para ir a tus módulos.».

#### TC-DAS-038 — El administrador ve todo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-DAS-008] «El administrador ve todo»; [CU-DAS-08] flujo básico; [RN-DAS-003], [RN-GEN-021] |

**Pasos:** abre `/dashboard` con Ana (Admin) en una empresa con dos sucursales.
**Resultado esperado:** todos los bloques con el total de la empresa y la tarjeta «Ventas por sucursal».

#### TC-DAS-039 — Un cajero con una sola sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-008] «Un cajero con una sola sucursal»; [CU-DAS-08] flujo básico; [RN-DAS-003] |

**Pasos:** abre `/dashboard` con Carlos (solo «Sede Principal»).
**Resultado esperado:** ve los totales de su sucursal, «Mis ventas de hoy» y su caja.

#### TC-DAS-040 — Alcance de un usuario no administrador

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-DAS-008] «Alcance de un usuario no administrador»; [CU-DAS-08] flujo básico; [RN-DAS-003], [RN-GEN-023] |

**Precondiciones:** una cuenta con sucursal base «Sede Principal» y «Sucursal Demo Norte» asignada a su cuenta (o a su rol); otra cuenta solo con la base.
**Pasos:** abre `/dashboard` con cada una y compara las cifras con las ventas de cada sucursal.
**Resultado esperado:** las cifras cubren la sucursal base más las asignadas a la cuenta y a sus roles; quien solo tiene la base no incluye las de Norte; ninguno incluye otra empresa.

#### TC-DAS-041 — La ruta del Inicio no exige permiso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Baja | [HU-DAS-008] «La ruta no exige permiso»; [CU-DAS-08] alternativo A2 |

**Pasos:** abre `/dashboard` con una cuenta sin permisos de lectura; luego intenta una pantalla sin permiso (por ejemplo `/pos`).
**Resultado esperado:** cualquier usuario autenticado entra a `/dashboard`; el rechazo de la otra pantalla te devuelve a `/dashboard` con «No tienes permisos para acceder a esta pantalla.».

#### TC-DAS-042 — Fallan mis permisos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-DAS-008] «Fallan mis permisos»; [CU-DAS-08] excepciones |

**Pasos:** bloquea `GET /api/v1/auth/me` y abre `/dashboard`; desbloquea y haz clic en «Reintentar».
**Resultado esperado:** «No se pudieron cargar tus permisos.» con «Reintentar»; al reintentar con éxito se muestran los bloques.

### HU-DAS-009 — Actualizar el resumen y reintentar una sección

#### TC-DAS-043 — Actualizar y no hay recarga automática

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-009] «Actualizar», «No hay recarga automática»; [CU-DAS-09] flujo básico; [RN-DAS-002] |

**Pasos:** abre `/dashboard`; haz una venta en otra pestaña; espera 2 minutos sin pulsar nada; luego haz clic en «Actualizar».
**Resultado esperado:** las cifras no cambian solas; al pulsar «Actualizar» se recalcula «hoy», se piden de nuevo todas las secciones permitidas, se lee «Actualizado a las {hora}», el botón queda deshabilitado mientras carga y se conservan los datos previos hasta que lleguen los nuevos.

#### TC-DAS-044 — Una sección falla y se reintenta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-DAS-009] «Una sección falla»; [CU-DAS-09] alternativo A1; [RN-DAS-024] |

**Pasos:** bloquea `GET /api/v1/dashboard/caja`, abre el Inicio; desbloquea y haz clic en «Reintentar».
**Resultado esperado:** la tarjeta de caja muestra «No se pudo cargar esta sección.» con «Reintentar» y las demás secciones siguen visibles; al reintentar solo se vuelve a pedir esa sección.

#### TC-DAS-045 — Cargando y cifras en cero

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-DAS-009] «Cargando», «Cifras en cero»; [CU-DAS-09] alternativo A2 |

**Pasos:** abre el Inicio con la red lenta y luego en una sucursal sin datos.
**Resultado esperado:** cada sección sin datos muestra un indicador de carga («Cargando»); las cifras en cero se muestran como 0 o «S/ 0.00», sin texto de «sin datos».

### HU-DAS-010 — Abrir los listados desde las tarjetas

#### TC-DAS-046 — Enlaces de ventas con una sola sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-010] «Enlaces de ventas con una sola sucursal»; [CU-DAS-10] flujo básico; [RN-DAS-014] |

**Pasos:** con Carlos (una sola sucursal) haz clic en cada botón del bloque de ventas.
**Resultado esperado:** se ven «Ver ventas de hoy», «Ver mis ventas» y «Ver ventas en espera»; los dos primeros abren `/pos` con la fecha de hoy y el estado «Completada» (y tu empleado en el segundo); «Ver ventas en espera» abre `/pos` con el estado «Pendiente» y sin fechas.

#### TC-DAS-047 — Con varias sucursales no hay enlaces de ventas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-DAS-010] «Con varias sucursales no hay enlaces de ventas»; [CU-DAS-10] alternativo A1; [RN-DAS-014] |

**Pasos:** abre `/dashboard` con Ana (Admin, dos sucursales).
**Resultado esperado:** no se ven los botones «Ver ventas de hoy», «Ver mis ventas» ni «Ver ventas en espera», porque el listado de ventas solo cubre la sucursal propia.

#### TC-DAS-048 — El listado puede mostrar menos ventas que la tarjeta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-DAS-010] «El listado puede mostrar menos ventas que la tarjeta»; [CU-DAS-10] flujo básico; [RN-DAS-005] |

**Precondiciones:** hoy una venta «Devuelta» y una «Completada».
**Pasos:** compara «Vendido hoy» con el listado de «Ver ventas de hoy».
**Resultado esperado:** «Vendido hoy» incluye la «Devuelta»; el listado, al filtrar por «Completada», no.
**Hallazgo:** HU-DAS-001, «Brecha conocida».

#### TC-DAS-049 — Otros destinos desde las tarjetas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-DAS-010] «Otros destinos»; [CU-DAS-10] flujo básico; [RN-DAS-016] |

**Pasos:** haz clic en «Ver mi caja», «Ver cajas abiertas» y «Ver lotes».
**Resultado esperado:** «Ver mi caja» abre tu sesión; «Ver cajas abiertas» abre `/sesiones-caja` con estado «Abierta»; «Ver lotes» abre el stock por sucursal con la caducidad y «solo con stock».

## 8. Casos de los módulos «Solo API» (`TC-OPS`)

Ninguno de estos módulos tiene pantalla (HU-OPS, sección 12 de las historias): todos los casos se ejecutan con un cliente HTTP contra la API local. El token (`<token>`) es el de un usuario con los permisos que indica cada caso; ningún script los siembra, así que hay que concederlos al rol de prueba. Ninguna de estas operaciones deja evento de auditoría de seguridad (RN-AUD-006). Los enumerados que viajan en un cuerpo JSON se esperan como número (**No verificado** en ejecución, 01d §2 punto 6). Los errores de tipo «prohibido» de transferencias responden 400 y no 403 (01d §2 punto 1). Se usa el lote `LOT-QA-P01` (500 unidades en «Sede Principal») como «L-001» de los ejemplos.

### 8.1 Transferencias entre sucursales

### HU-OPS-001 — Crear una transferencia entre sucursales

#### TC-OPS-001 — Crear una transferencia

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-001] «Crear una transferencia»; [CU-OPS-01] flujo básico; [RN-TRA-001], [RN-TRA-002], [RN-TRA-003] |

**Precondiciones:** token de un usuario de «Sede Principal» con `Transferencias:Crear`; `LOT-QA-P01` con 500 unidades en esa sucursal.
**Petición:** `POST /api/v1/Transferencias` con `{"sucursalDestinoId":"<Sucursal Demo Norte>","detalles":[{"loteId":"<LOT-QA-P01>","cantidadEnviada":100}]}`.
**Resultado esperado:** HTTP 201 con el identificador y la cabecera `Location` hacia `GET transferencias/{id}`; la transferencia queda en estado `Iniciada` con el empleado emisor de la sesión; el stock no se toca (`LOT-QA-P01` sigue en 500: crear no descuenta ni reserva).

#### TC-OPS-002 — El origen y el emisor salen de la sesión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-001] «El origen y el emisor salen de mi sesión»; [CU-OPS-01] excepciones; [RN-TRA-001], [RN-TRA-002], [RN-GEN-024] |

**Petición:** `POST /api/v1/Transferencias` (cuerpo de `TC-OPS-001`) con el token de Ana (Admin, «Sede Principal») y con un token sin sucursal o sin empleado (**No verificado:** cómo prepararlo).
**Resultado esperado:** con Ana la sucursal de origen es «Sede Principal» (la de su token) y el emisor es su empleado, aunque sea administrador; sin sucursal o empleado: HTTP 400, `code` `Auth.Sucursal` o `Auth.Empleado`.

#### TC-OPS-003 — Destino obligatorio y distinto del origen

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-001] «Destino obligatorio y distinto del origen»; [CU-OPS-01] excepciones; [RN-TRA-004] |

**Petición:** `POST /api/v1/Transferencias` (1) con `sucursalDestinoId` vacío (`00000000-0000-0000-0000-000000000000`); (2) con el destino igual a la sucursal de origen.
**Resultado esperado:** (1) HTTP 400, `code` `CreateTransferenciaCommand.SucursalDestinoId`; (2) HTTP 400, `code` `Transferencia.SucursalDestino`. No se crea nada.

#### TC-OPS-004 — El destino no existe o es de otra empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-OPS-001] «El destino no existe»; [CU-OPS-01] excepciones; [RN-TRA-005] |

**Petición:** `POST /api/v1/Transferencias` con un `sucursalDestinoId` inexistente y con el de una sucursal de «Botica Otra Demo S.A.C.».
**Resultado esperado:** HTTP 404, `code` `Transferencia.SucursalDestinoNotFound` en ambos casos; no se crea nada. El sistema no revisa si el usuario tiene acceso a la sucursal destino (cualquier sucursal de su empresa es válida).

#### TC-OPS-005 — Al menos una línea con lote y cantidad

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-001] «Al menos una línea con lote y cantidad»; [CU-OPS-01] excepciones; [RN-TRA-006], [RN-TRA-007] |

**Petición:** `POST /api/v1/Transferencias` con (1) `"detalles":[]`; (2) una línea con `loteId` vacío; (3) `cantidadEnviada` 0; (4) `cantidadEnviada` -1.
**Resultado esperado:** HTTP 400 con `code` `CreateTransferenciaCommand.Detalles`, `CreateTransferenciaCommand.Detalles[0].LoteId` y `CreateTransferenciaCommand.Detalles[0].CantidadEnviada` (en (3) y (4)). No hay máximo de líneas ni de cantidad.

#### TC-OPS-006 — El lote no tiene stock registrado en mi sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-OPS-001] «El lote no tiene stock registrado en mi sucursal»; [CU-OPS-01] excepciones; [RN-TRA-008] |

**Petición:** `POST /api/v1/Transferencias` desde «Sede Principal» con `LOT-QA-N01` (stock solo en «Sucursal Demo Norte»).
**Resultado esperado:** HTTP 404, `code` `Transferencia.LoteNotFound`; no se crea nada.

#### TC-OPS-007 — Stock insuficiente al crear

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-OPS-001] «Stock insuficiente al crear»; [CU-OPS-01] excepciones; [RN-TRA-009] |

**Petición:** `POST /api/v1/Transferencias` con `cantidadEnviada` 501 y luego 500 (el lote tiene 500).
**Resultado esperado:** 501: HTTP 400, `code` `Transferencia.StockInsuficiente` con la cantidad disponible (500) en el mensaje; 500: se crea (pedir exactamente 500 es válido).

#### TC-OPS-008 — Lotes vencidos o retenidos se pueden transferir

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-001] «Lotes vencidos o retenidos se pueden transferir»; [CU-OPS-01] alternativo A1; [RN-TRA-010] |

**Petición:** `POST /api/v1/Transferencias` con `LOT-QA-P03` (vencido) y con `LOT-QA-P04` (en cuarentena).
**Resultado esperado:** la transferencia se crea igual (HTTP 201), porque solo se comprueba la existencia de la fila de stock y la cantidad física.
**Hallazgo:** no se revisa vencimiento ni cuarentena al transferir (RN-TRA-010; ver también `TC-OPS-049`).

#### TC-OPS-009 — El mismo lote en varias líneas al crear

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-001] «El mismo lote en varias líneas»; [CU-OPS-01] flujo básico; [RN-TRA-009] |

**Petición:** `POST /api/v1/Transferencias` con dos líneas del mismo lote de 300 y 300 (stock 500).
**Resultado esperado:** se crea (cada línea se compara por separado con el stock al crear); el despacho es el que acumula (`TC-OPS-018`).

#### TC-OPS-010 — Dos transferencias pueden pedir el mismo stock

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-OPS-001] «Dos transferencias pueden pedir el mismo stock»; [CU-OPS-01] alternativo A2; [RN-TRA-010] |

**Petición:** crea dos transferencias de 400 unidades cada una del mismo lote (stock 500); despacha la primera y luego la segunda.
**Resultado esperado:** ambas se crean (crear no reserva nada); la primera se despacha y la segunda falla al despachar con `Transferencia.StockInsuficiente` (`TC-OPS-017`).

#### TC-OPS-011 — El estado «Aprobada» no se alcanza

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-OPS-001] «El estado "Aprobada" no existe en la práctica»; [CU-OPS-01] alternativo A3; [RN-TRA-011] |

**Petición:** recorre los endpoints de transferencias (despachar, recibir, cancelar) y consulta el estado tras cada paso.
**Resultado esperado:** ningún endpoint pasa una transferencia a `Aprobada`; sale de `Iniciada` directamente a `En_Transito` o `Cancelada`.

#### TC-OPS-012 — Crear una transferencia sin el permiso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-OPS-001] «Sin permiso»; [CU-OPS-01] excepciones; [RN-GEN-020] |

| Permiso que falta | Qué responde |
|---|---|
| `Transferencias:Crear` | `POST /api/v1/Transferencias` niega el acceso (403) |
| `Transferencias:Actualizar` | `PUT /api/v1/Transferencias/{id}/despachar`, `/recibir` y `/cancelar` niegan el acceso (403) |
| `Transferencias:Leer` | `GET /api/v1/Transferencias` y `GET /api/v1/Transferencias/{id}` niegan el acceso (403) |

**Petición:** cada endpoint con un token sin el permiso correspondiente.
**Resultado esperado:** según la tabla; no cambia ningún dato. **No verificado:** la forma exacta del cuerpo del 403 (13c §3.2).

### HU-OPS-002 — Despachar una transferencia

#### TC-OPS-013 — Despachar una transferencia

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-OPS-002] «Despachar»; [CU-OPS-02] flujo básico; [RN-TRA-012], [RN-TRA-013] |

**Precondiciones:** transferencia `Iniciada` de 100 unidades de `LOT-QA-P01` (stock origen 500); token de «Sede Principal» con `Transferencias:Actualizar`.
**Petición:** `PUT /api/v1/Transferencias/{id}/despachar`.
**Resultado esperado:** HTTP 204 sin cuerpo; la transferencia pasa a `En_Transito` con la fecha de despacho en la hora actual UTC; el stock del origen baja de 500 a 400.

#### TC-OPS-014 — Solo el origen despacha

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-OPS-002] «Solo el origen despacha»; [CU-OPS-02] excepciones; [RN-TRA-012] |

**Petición:** `PUT /api/v1/Transferencias/{id}/despachar` con el token de Nora («Sucursal Demo Norte», destino) y con el de un administrador de otra sucursal.
**Resultado esperado:** HTTP 400, `code` `Transferencia.Forbidden` (declarado como prohibido pero sin número, por eso 400 y no 403); la transferencia sigue `Iniciada` y el stock no cambia.

#### TC-OPS-015 — Despachar una transferencia que no existe

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-002] «La transferencia no existe»; [CU-OPS-02] excepciones; [RN-TRA-013] |

**Petición:** `PUT /api/v1/Transferencias/{id}/despachar` con un GUID inexistente.
**Resultado esperado:** HTTP 404, `code` `Transferencia.NotFound`.

#### TC-OPS-016 — Estado que no admite el despacho

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-002] «Estado que no admite el despacho»; [CU-OPS-02] excepciones; [RN-TRA-013] |

**Petición:** despacha transferencias que ya están `En_Transito`, `Completada`, `Recibida_Parcial` y `Cancelada`.
**Resultado esperado:** HTTP 400, `code` `Transferencia.EstadoLogistico` en los cuatro casos; esta comprobación va antes que la de stock.

#### TC-OPS-017 — Stock insuficiente al despachar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-OPS-002] «Stock insuficiente al despachar»; [CU-OPS-02] excepciones; [RN-TRA-014] |

**Precondiciones:** transferencia `Iniciada` de 100 unidades; luego otra operación baja el stock del lote en el origen a 50 (por ejemplo, ventas).
**Petición:** `PUT /api/v1/Transferencias/{id}/despachar`.
**Resultado esperado:** HTTP 400, `code` `Transferencia.StockInsuficiente` con la cantidad disponible (50; 0 si ya no hay fila); no se guarda nada, ni el cambio de estado.

#### TC-OPS-018 — Líneas con el mismo lote se acumulan al despachar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-OPS-002] «Líneas con el mismo lote se acumulan»; [CU-OPS-02] excepciones; [RN-TRA-014] |

**Precondiciones:** transferencia `Iniciada` con dos líneas del mismo lote de 300 y 300 (`TC-OPS-009`), stock 500.
**Petición:** `PUT /api/v1/Transferencias/{id}/despachar`.
**Resultado esperado:** HTTP 400, `code` `Transferencia.StockInsuficiente` (la suma, 600, excede el stock); la transferencia sigue `Iniciada`.

#### TC-OPS-019 — Conflicto de concurrencia al despachar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-OPS-002] «Conflicto de concurrencia»; [CU-OPS-02] excepciones; [RN-TRA-014] |

**Petición:** despacha la transferencia mientras otra operación modifica la misma fila de stock (por ejemplo, una venta del mismo lote enviada en paralelo).
**Resultado esperado:** HTTP 409, `code` `Concurrency.Conflict` si se detecta el conflicto de versión de fila. **No verificado en ejecución.**

### HU-OPS-003 — Recibir una transferencia

#### TC-OPS-020 — Recepción completa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-OPS-003] «Recepción completa»; [CU-OPS-03] flujo básico; [RN-TRA-015], [RN-TRA-016] |

**Precondiciones:** transferencia `En_Transito` con 100 unidades de `LOT-QA-P01`; token de Nora (destino «Sucursal Demo Norte») con `Transferencias:Actualizar`; el lote sin fila de stock en Norte.
**Petición:** `PUT /api/v1/Transferencias/{id}/recibir` con `[{"loteId":"<LOT-QA-P01>","cantidadRecibida":100}]`.
**Resultado esperado:** HTTP 204; la transferencia queda `Completada` con el empleado de Nora como receptor y la hora de recepción; el stock de Norte suma 100 (se crea la fila si no existía).

#### TC-OPS-021 — Recepción parcial

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-OPS-003] «Recepción parcial»; [CU-OPS-03] alternativo A1; [RN-TRA-017] |

**Petición:** `PUT /api/v1/Transferencias/{id}/recibir` con `cantidadRecibida` 90 de las 100 unidades.
**Resultado esperado:** HTTP 204; la transferencia queda `Recibida_Parcial`; el destino suma solo 90.

#### TC-OPS-022 — La diferencia enviada y no recibida se pierde

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-OPS-003] «La diferencia se pierde»; [CU-OPS-03] alternativo A1; [RN-TRA-017], [RN-TRA-020] |

**Petición:** tras `TC-OPS-021`, suma el stock de `LOT-QA-P01` en las dos sucursales (origen + destino).
**Resultado esperado:** origen 400 + destino 90 = 490, no 500: las 10 unidades ya salieron del origen en el despacho y no vuelven ni se registran como merma; el cuerpo de la recepción no admite sustento ni motivo de la diferencia.
**Hallazgo:** la diferencia desaparece del stock total entre sucursales (01d hallazgo 7); los textos de `transferencias/es.json` hablan de un «sustento obligatorio» que el backend no implementa.

#### TC-OPS-023 — Cantidad recibida 0

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-003] «Cantidad recibida 0»; [CU-OPS-03] flujo básico; [RN-TRA-018] |

**Petición:** `PUT /api/v1/Transferencias/{id}/recibir` con `cantidadRecibida` 0.
**Resultado esperado:** es válido («no llegó nada»): la línea queda en 0 sin tocar el stock del destino.

#### TC-OPS-024 — Las líneas no listadas se ignoran

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-003] «Las líneas no listadas se ignoran»; [CU-OPS-03] alternativo A1; [RN-TRA-019] |

**Precondiciones:** transferencia `En_Transito` con dos líneas (lotes A y B).
**Petición:** recibe listando solo el lote A completo.
**Resultado esperado:** la transferencia queda `Completada` aunque la línea B no se recibió; esa línea conserva la cantidad recibida en NULL sin sumar stock.
**Hallazgo:** una línea omitida no se contabiliza como diferencia (01d).

#### TC-OPS-025 — Reglas por línea al recibir

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-OPS-003] «Reglas por línea»; [CU-OPS-03] excepciones; [RN-TRA-018] |

**Petición:** recibe con (1) un lote que no pertenece a la transferencia; (2) `cantidadRecibida` -1; (3) 101 (mayor que lo enviado, 100); luego (4) exactamente 100.
**Resultado esperado:** (1), (2) y (3): HTTP 400, `code` `Transferencia.Recepcion`, sin guardar nada de la recepción (la transferencia sigue `En_Transito`); (4): válido.

#### TC-OPS-026 — Forma del cuerpo de la recepción

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-003] «Forma del cuerpo»; [CU-OPS-03] excepciones; [RN-TRA-018] |

**Petición:** `PUT /api/v1/Transferencias/{id}/recibir` con un arreglo vacío, con un `loteId` vacío y con `cantidadRecibida` negativa.
**Resultado esperado:** HTTP 400, `code` `RecibirTransferenciaCommand.Recepciones` o `RecibirTransferenciaCommand.Recepciones[0].CantidadRecibida` (el lote vacío da el `code` de su propiedad).

#### TC-OPS-027 — Solo el destino recibe

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-OPS-003] «Solo el destino recibe»; [CU-OPS-03] excepciones; [RN-TRA-015] |

**Petición:** recibe con el token de «Sede Principal» (origen) o de un administrador de otra sucursal; con un token sin empleado.
**Resultado esperado:** HTTP 400, `code` `Transferencia.Forbidden` (aunque sea administrador); sin empleado: HTTP 400 `Auth.Empleado`. La transferencia no cambia.

#### TC-OPS-028 — Solo se recibe una transferencia en tránsito

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-003] «Solo se recibe una transferencia en tránsito»; [CU-OPS-03] excepciones; [RN-TRA-016] |

**Petición:** intenta recibir otra vez una transferencia `Completada` o `Recibida_Parcial`.
**Resultado esperado:** HTTP 400, `code` `Transferencia.EstadoLogistico`: no se puede recibir dos veces; el stock del destino no cambia.

#### TC-OPS-029 — Recibir y cancelar a la vez

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-OPS-003] «Recibir y cancelar a la vez»; [CU-OPS-03] alternativo A2; [RN-TRA-024] |

**Petición:** envía en paralelo `PUT .../recibir` (destino) y `PUT .../cancelar` (origen) sobre la misma transferencia `En_Transito`.
**Resultado esperado:** solo una operación se guarda y la otra recibe HTTP 409, `code` `Concurrency.Conflict`. **No verificado en ejecución.**

### HU-OPS-004 — Cancelar una transferencia

#### TC-OPS-030 — Cancelar una transferencia en tránsito

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-OPS-004] «Cancelar una transferencia en tránsito»; [CU-OPS-04] flujo básico; [RN-TRA-021], [RN-TRA-022] |

**Precondiciones:** transferencia `En_Transito` de 100 unidades de `LOT-QA-P01` (origen en 400).
**Petición:** `PUT /api/v1/Transferencias/{id}/cancelar` con el token de «Sede Principal».
**Resultado esperado:** HTTP 204; la transferencia queda `Cancelada`; el origen recupera las 100 unidades (400 a 500; se suman a la fila existente o se crea si ya no existe).

#### TC-OPS-031 — Cancelar antes del despacho

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-004] «Cancelar antes del despacho»; [CU-OPS-04] flujo básico; [RN-TRA-022] |

**Petición:** `PUT .../cancelar` sobre una transferencia `Iniciada`.
**Resultado esperado:** queda `Cancelada` sin mover stock (nunca se descontó).

#### TC-OPS-032 — Solo el origen cancela

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-OPS-004] «Solo el origen cancela»; [CU-OPS-04] excepciones; [RN-TRA-021] |

**Petición:** `PUT .../cancelar` con el token del destino (Nora).
**Resultado esperado:** HTTP 400, `code` `Transferencia.Forbidden`: el destino no puede rechazar una transferencia; el estado no cambia.

#### TC-OPS-033 — Estados que no se cancelan

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-004] «Estados que no se cancelan»; [CU-OPS-04] excepciones; [RN-TRA-023] |

**Petición:** `PUT .../cancelar` sobre transferencias `Completada`, `Recibida_Parcial` y `Cancelada`.
**Resultado esperado:** HTTP 400, `code` `Transferencia.EstadoLogistico`, y el mensaje indica el estado actual.

#### TC-OPS-034 — Cancelar una transferencia que no existe

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-004] «La transferencia no existe»; [CU-OPS-04] excepciones; [RN-TRA-024] |

**Petición:** `PUT /api/v1/Transferencias/{id}/cancelar` con un GUID inexistente.
**Resultado esperado:** HTTP 404, `code` `Transferencia.NotFound`.

### HU-OPS-005 — Consultar transferencias

#### TC-OPS-035 — Listar mis transferencias

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-OPS-005] «Listar mis transferencias»; [CU-OPS-05] flujo básico; [RN-TRA-025], [RN-TRA-026], [RN-TRA-027] |

**Precondiciones:** transferencias entre «Sede Principal» y «Sucursal Demo Norte» y una entre otras dos sucursales; `Transferencias:Leer`.
**Petición:** `GET /api/v1/Transferencias` con el token de Carlos, de Ana (Admin) y de Nora; luego con `SucursalFiltroId` y con `Estado=En_Transito`.
**Resultado esperado:** solo las transferencias donde la sucursal de la sesión es origen o destino (también para el administrador); los filtros por `SucursalFiltroId` y por `Estado` (nombre o número) funcionan; orden por fecha de despacho descendente, de a 10 por página; cada fila trae sucursales y empleados con sus nombres, el estado como texto (`En_Transito`), las fechas y las líneas con número de lote, cantidad enviada y recibida.
**Hallazgo:** las transferencias ignoran las sucursales accesibles: solo cuenta la sucursal del token (01d hallazgo 8). No hay exportación.

#### TC-OPS-036 — Estado inexistente en el listado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-005] «Estado inexistente»; [CU-OPS-05] excepciones; [RN-GEN-041] |

**Petición:** `GET /api/v1/Transferencias?Estado=Inventado`.
**Resultado esperado:** falla la lectura del parámetro (no hay validador de la consulta). **No verificado:** el HTTP y el cuerpo exactos.

#### TC-OPS-037 — Consultar una transferencia

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-005] «Consultar una transferencia»; [CU-OPS-05] flujo básico; [RN-TRA-028] |

**Petición:** `GET /api/v1/Transferencias/{id}` de una transferencia donde la sucursal del token es origen o destino; repite con un lote eliminado.
**Resultado esperado:** HTTP 200 con el mismo contenido que el listado; el número de lote es «Desconocido» si el lote ya no se encuentra.

#### TC-OPS-038 — Identificador inexistente o ajeno

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-OPS-005] «Identificador inexistente o ajeno»; [CU-OPS-05] excepciones; [RN-TRA-029] |

**Petición:** `GET /api/v1/Transferencias/{id}` con un GUID inexistente y con el de una transferencia entre otras dos sucursales.
**Resultado esperado:** 404 `Transferencia.NotFound` si no existe; 400 `Transferencia.Forbidden` si existe pero no es de la sucursal del token.
**Hallazgo:** una sucursal puede así saber que un identificador existe aunque no sea suyo.

#### TC-OPS-039 — Estructura y dependencias de transferencias

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-005] «Estructura y dependencias»; [CU-OPS-05] alternativo A1; [RN-TRA-030] |

**Pasos:** revisa en la base las restricciones de `Transferencias_Cab` y `Transferencias_Det`; intenta (SPA o API de sucursales, ver parte B) eliminar una sucursal con transferencias.
**Resultado esperado:** estados, cantidades y referencias las impone la base con restricciones; desde el commit `27597cd` una sucursal con transferencias no se puede eliminar (HTTP 409, `Sucursal.InUse`).

### 8.2 Cuarentena y actas de destrucción DIGEMID

Los nombres de los campos de los cuerpos de este módulo se deducen de los códigos de error de las historias (`AislarLoteCuarentenaCommand.LoteId`, `.CantidadAislada`, `.MotivoAislamiento`, `.EstadoResolucion`); el nombre exacto del campo de la línea de devolución, y el formato de los estados (texto o número), son **No verificado**: consulta `sofia-apis/src/SOFIA.API/Controllers/` antes de armar la petición.

### HU-OPS-006 — Aislar un lote en cuarentena

#### TC-OPS-040 — Aislar un lote como «Retenido»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-OPS-006] «Aislar un lote como "Retenido"»; [CU-OPS-06] flujo básico; [RN-CUA-001], [RN-CUA-002], [RN-CUA-003] |

**Precondiciones:** token con `DIGEMID:AislarLoteCuarentena` de «Sede Principal»; `LOT-QA-P04` existe en la empresa.
**Petición:** `POST /api/v1/DIGEMID/cuarentena` con `{"loteId":"<LOT-QA-P04>","cantidadAislada":20,"motivoAislamiento":"Alerta de calidad","estadoResolucion":"Retenido"}`.
**Resultado esperado:** HTTP 200 con el identificador del registro; el registro guarda la sucursal y el empleado de la sesión y la hora actual UTC como fecha de ingreso; el lote queda bloqueado para la venta de inmediato en toda la empresa (`TC-OPS-049`).

#### TC-OPS-041 — Datos obligatorios del aislamiento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-006] «Datos obligatorios»; [CU-OPS-06] excepciones; [RN-CUA-004], [RN-CUA-005] |

**Petición:** `POST /api/v1/DIGEMID/cuarentena` con (1) lote vacío; (2) `cantidadAislada` 0; (3) motivo vacío; (4) motivo de 51 caracteres (y uno de 50, válido); (5) `estadoResolucion` fuera de `Retenido`, `Liberado`, `Destruido`, `Devuelto`.
**Resultado esperado:** HTTP 400 con `code` `AislarLoteCuarentenaCommand.LoteId`, `.CantidadAislada`, `.MotivoAislamiento` (3 y 4) y `.EstadoResolucion` (5); el motivo de 50 caracteres se acepta.

#### TC-OPS-042 — El estado lo elige el integrador al crear

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-006] «El estado lo elijo yo al crear»; [CU-OPS-06] flujo básico; [RN-CUA-006] |

**Petición:** `POST /api/v1/DIGEMID/cuarentena` con `estadoResolucion` `Liberado`, `Destruido` y `Devuelto` (un lote distinto cada vez o el mismo).
**Resultado esperado:** el registro se guarda con ese estado y no bloquea la venta.

#### TC-OPS-043 — Lote o línea de devolución inexistentes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-OPS-006] «Lote o línea de devolución inexistentes»; [CU-OPS-06] excepciones; [RN-CUA-010] |

**Petición:** `POST /api/v1/DIGEMID/cuarentena` con un `loteId` inexistente, con el de un lote de «Botica Otra Demo S.A.C.» y con un identificador de línea de devolución inexistente.
**Resultado esperado:** HTTP 404, `code` `LoteInventario.NotFound` (lote) o `DetalleDevolucion.NotFound` (línea); un identificador de otra empresa se informa como inexistente.

#### TC-OPS-044 — No se comprueba el stock al aislar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-006] «No se comprueba el stock»; [CU-OPS-06] flujo básico; [RN-CUA-011] |

**Petición:** `POST /api/v1/DIGEMID/cuarentena` con `LOT-QA-N01` (sin fila de stock en «Sede Principal») y con `cantidadAislada` 100000 sobre `LOT-QA-P01`; revisa el stock.
**Resultado esperado:** ambos se aceptan: el sistema no verifica que el lote tenga fila de stock en la sucursal, que la cantidad aislada no supere el stock ni que la línea de devolución sea de ese lote; aislar no mueve stock (el stock físico sigue contando).
**Hallazgo:** 01d §9.3.

#### TC-OPS-045 — Un lote puede tener varios registros

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-OPS-006] «Un lote puede tener varios registros»; [CU-OPS-06] alternativo A1; [RN-CUA-006] |

**Petición:** aísla el mismo lote dos veces en la misma sucursal y una vez desde otra sucursal, con distintos estados.
**Resultado esperado:** los tres registros se crean.

#### TC-OPS-046 — La sucursal y el empleado salen de la sesión al aislar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-006] «La sucursal y el empleado salen de mi sesión»; [CU-OPS-06] excepciones; [RN-CUA-002] |

**Petición:** `POST /api/v1/DIGEMID/cuarentena` con un token sin sucursal o sin empleado (**No verificado:** cómo prepararlo).
**Resultado esperado:** HTTP 400, `code` `Auth.Sucursal` o `Auth.Empleado`.

#### TC-OPS-047 — Aislar desde una devolución con destino «Cuarentena DIGEMID»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | [HU-OPS-006] «Aislar desde una devolución con destino «Cuarentena DIGEMID»»; [CU-OPS-06] alternativo A2; [RN-CUA-011], [RN-DEV-024] |

**Precondiciones:** devolución con destino «Cuarentena DIGEMID» (`TC-DEV-019`).
**Petición:** `POST /api/v1/DIGEMID/cuarentena` informando el lote y el identificador de la línea de devolución.
**Resultado esperado:** queda el vínculo con la línea de devolución, sin que el sistema compare cantidades ni lote; la devolución por sí sola nunca crea el registro de cuarentena (antes de esta llamada no hay registro y el Inicio no cuenta el lote, `TC-DAS-030`).

#### TC-OPS-048 — Estructura y auditoría del aislamiento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-OPS-006] «Estructura y auditoría»; [CU-OPS-06] flujo básico; [RN-CUA-020] |

**Pasos:** tras `TC-OPS-040` revisa la tabla de auditoría y las restricciones de la tabla de cuarentena.
**Resultado esperado:** las referencias a sucursal, lote, empleado y línea de devolución las impone la base; la operación no deja evento de auditoría de seguridad (queda solo el usuario y la fecha de creación); la base no tiene restricciones sobre el estado ni la cantidad.
**Hallazgo:** un hecho regulado por DIGEMID queda sin rastro en la auditoría (RN-CUA-005).

### HU-OPS-007 — Bloquear la venta de un lote retenido (y sus límites)

#### TC-OPS-049 — El lote retenido no se vende en ninguna sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-OPS-007] «El lote retenido no se vende en ninguna sucursal»; [CU-OPS-07] flujo básico; [RN-CUA-007], [RN-VEN-012] |

**Precondiciones:** `LOT-QA-P04` con registro «Retenido» (`TC-OPS-040`) y stock en «Sede Principal» y «Sucursal Demo Norte».
**Pasos:** 1. En el terminal de «Sede Principal» busca el producto y mira los lotes. 2. En el Inicio mira «Lotes en cuarentena». 3. `POST /api/v1/ventas` que incluya `LOT-QA-P04` (con caja `S1` abierta). 4. Repite el paso 1 en «Sucursal Demo Norte».
**Resultado esperado:** el terminal no ofrece el lote ni cuenta como stock vendible; cuenta en «Lotes en cuarentena» del Inicio; por API: HTTP 400, `code` `Venta.Cuarentena` y en el SPA «Ocurrió un error al procesar la venta.»; no depende de la sucursal ni de la cantidad aislada (se bloquea el lote entero), también en Norte.

#### TC-OPS-050 — Los estados «Liberado», «Destruido» y «Devuelto» no bloquean

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-OPS-007] «Los estados "Liberado", "Destruido" y "Devuelto" no bloquean»; [CU-OPS-07] flujo básico; [RN-CUA-008] |

**Precondiciones:** un lote solo con registros en esos tres estados (`TC-OPS-042`).
**Pasos:** busca el lote en el terminal y vende 1 unidad.
**Resultado esperado:** se puede vender y cuenta como stock vendible.

#### TC-OPS-051 — Crear un registro «Liberado» no levanta un bloqueo anterior

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-OPS-007] «Crear un registro "Liberado" no levanta un bloqueo anterior»; [CU-OPS-07] flujo básico; [RN-CUA-008] |

**Precondiciones:** `LOT-QA-P04` con un registro «Retenido».
**Pasos:** crea otro registro «Liberado» para el mismo lote y busca el lote en el terminal.
**Resultado esperado:** el lote sigue bloqueado mientras exista cualquier registro «Retenido».

#### TC-OPS-052 — No existe forma de liberar un lote retenido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-007] «No existe forma de liberar un lote retenido»; [CU-OPS-07] alternativo A2; [RN-CUA-009] |

**Pasos:** revisa los endpoints de `DIGEMID/*` en el Swagger local de la API (solo lectura) e intenta una operación `PUT` o `DELETE` sobre un registro de cuarentena.
**Resultado esperado:** ningún endpoint cambia el estado de un registro ni lo elimina; la única vía es modificar la base de datos a mano.
**Hallazgo:** 01d §9.3 hallazgos 1 y 5.

#### TC-OPS-053 — Los demás flujos no consultan la cuarentena

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-007] «Los demás flujos no consultan la cuarentena»; [CU-OPS-07] alternativo A1; [RN-CUA-009] |

**Pasos:** con `LOT-QA-P04` «Retenido»: crea una transferencia (`TC-OPS-008`), úsalo como insumo de una orden magistral (`TC-OPS-074`) y regístralo en una inmunización (`TC-OPS-101`).
**Resultado esperado:** las tres operaciones se aceptan: solo la venta respeta la cuarentena.

### HU-OPS-008 — Consultar la cuarentena

#### TC-OPS-054 — Listar registros de cuarentena

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-OPS-008] «Listar registros»; [CU-OPS-08] flujo básico; [RN-CUA-012], [RN-GEN-009], [RN-GEN-023] |

**Precondiciones:** registros en «Sede Principal» y «Sucursal Demo Norte»; `DIGEMID:Leer`.
**Petición:** `GET /api/v1/DIGEMID/cuarentena` con el token de Carlos y de Ana (Admin).
**Resultado esperado:** HTTP 200 con los registros de las sucursales accesibles (el administrador, todas), por fecha de ingreso descendente, de a 10; cada fila trae identificador, sucursal, lote, cantidad, motivo, estado y fecha, pero no la línea de devolución ni el empleado que registró.

#### TC-OPS-055 — Filtros de la cuarentena

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-008] «Filtros»; [CU-OPS-08] flujo básico; [RN-CUA-012], [RN-GEN-008] |

**Petición:** `GET /api/v1/DIGEMID/cuarentena` con filtro de sucursal, de estado (`Retenido`; también un texto inexistente), de fechas de ingreso desde y hasta (el día límite) y de una sucursal no accesible.
**Resultado esperado:** filtra por sucursal; el estado es igualdad exacta sin validar que sea uno de los cuatro (un texto inexistente da una lista vacía); las fechas son inclusivas, en UTC; una sucursal no accesible devuelve una página vacía.

#### TC-OPS-056 — Sucursal con identificador vacío en la consulta de cuarentena

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-008] «Sucursal con identificador vacío»; [CU-OPS-08] excepciones; [RN-CUA-012] |

**Petición:** `GET /api/v1/DIGEMID/cuarentena?sucursalId=00000000-0000-0000-0000-000000000000`.
**Resultado esperado:** HTTP 400, `code` `GetCuarentenaQuery.SucursalId`.

### HU-OPS-009 — Generar un acta de destrucción

#### TC-OPS-057 — Generar un acta de destrucción

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-009] «Generar un acta»; [CU-OPS-09] flujo básico; [RN-CUA-013], [RN-CUA-014] |

**Precondiciones:** token con `DIGEMID:GenerarActaDestruccion`; un empleado existente como regente.
**Petición:** `POST /api/v1/DIGEMID/acta-destruccion` con número de resolución interna `RES-2026-001`, empresa de residuos «Residuos Demo S.A.C.», fecha de ejecución y el regente responsable (los nombres exactos de los campos son **No verificado**).
**Resultado esperado:** HTTP 200 con el identificador del acta; el acta guarda solo la cabecera.

#### TC-OPS-058 — Datos del acta: longitudes y fecha

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-009] «Datos del acta»; [CU-OPS-09] excepciones; [RN-CUA-015], [RN-CUA-021] |

**Petición:** `POST /api/v1/DIGEMID/acta-destruccion` con: número de resolución de 51 caracteres; empresa de residuos de 151; manifiesto de 51; ruta del acta firmada de 501; fecha de ejecución futura.
**Resultado esperado:** los cuatro textos largos responden HTTP 400 con el código de la entidad (por ejemplo `DigemidActaDestruccion.NumeroResolucionInterna`; el validador está vacío); los límites son 50, 150, 50 y 500 (manifiesto y ruta opcionales). La fecha de ejecución no tiene ninguna comprobación (puede ser futura): se acepta. La ruta del PDF es un texto: el sistema no recibe ni guarda el archivo.

#### TC-OPS-059 — El regente debe existir

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-009] «El regente debe existir»; [CU-OPS-09] excepciones; [RN-CUA-016] |

**Petición:** `POST /api/v1/DIGEMID/acta-destruccion` con un regente inexistente y con un empleado que no es regente ni tiene licencia.
**Resultado esperado:** el inexistente: HTTP 404, `code` `Empleado.NotFound`; el segundo se acepta (cualquier empleado vale: no se comprueba que sea regente ni que tenga licencia).

#### TC-OPS-060 — Número de resolución repetido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-009] «Número de resolución repetido»; [CU-OPS-09] excepciones; [RN-CUA-017], [RN-CUA-023] |

**Precondiciones:** acta `RES-2026-001` ya registrada (`TC-OPS-057`).
**Petición:** `POST /api/v1/DIGEMID/acta-destruccion` con el mismo número de resolución.
**Resultado esperado:** HTTP 409, `code` `Db.Duplicado` (el handler no lo comprueba antes: lo rechaza la base). Un acta eliminada libera su número (**No verificado** desde la API: no hay endpoint de eliminación).

#### TC-OPS-061 — El acta no incluye lotes ni cambia nada más

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-009] «El acta no incluye lotes ni cambia nada más»; [CU-OPS-09] flujo básico; [RN-CUA-018] |

**Pasos:** tras `TC-OPS-057` revisa los detalles del acta, los registros de cuarentena y el stock.
**Resultado esperado:** no se llenan los detalles del acta (registro de cuarentena y cantidad destruida); ningún registro de cuarentena pasa a «Destruido» ni se toca el stock.
**Hallazgo:** «El acta de destrucción es solo una cabecera» (01d hallazgo 2).

#### TC-OPS-062 — El acta no se edita ni se elimina

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-009] «El acta no se edita ni se elimina»; [CU-OPS-09] flujo básico; [RN-CUA-022] |

**Petición:** `PUT /api/v1/DIGEMID/acta-destruccion/{id}` y `DELETE /api/v1/DIGEMID/acta-destruccion/{id}`.
**Resultado esperado:** la ruta no existe (404 o 405; el código exacto es **No verificado**); no hay endpoints para modificar o eliminar un acta.

#### TC-OPS-063 — Estructura y auditoría del acta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-OPS-009] «Estructura»; [CU-OPS-09] flujo básico; [RN-CUA-021] |

**Pasos:** revisa en la base la tabla de actas y la auditoría tras `TC-OPS-057`.
**Resultado esperado:** la clave foránea al regente y el índice único por empresa y número los impone la base; la operación no deja evento de auditoría de seguridad.

### HU-OPS-010 — Consultar las actas de destrucción

#### TC-OPS-064 — Listar actas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-010] «Listar actas»; [CU-OPS-10] flujo básico; [RN-CUA-019] |

**Precondiciones:** varias actas; `DIGEMID:Leer`.
**Petición:** `GET /api/v1/DIGEMID/actas` con fechas de ejecución desde y hasta.
**Resultado esperado:** HTTP 200 con las actas de la empresa (no se filtra por sucursal; ninguna de otra empresa), por fecha de ejecución descendente, de a 10; cada fila trae identificador, número de resolución, fecha y empresa de residuos; no trae regente, manifiesto ni ruta del PDF.

#### TC-OPS-065 — No hay consulta individual de actas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-010] «No hay consulta individual»; [CU-OPS-10] excepciones; [RN-CUA-019] |

**Petición:** `GET /api/v1/DIGEMID/actas/{id}`.
**Resultado esperado:** la ruta no existe (404 o 405; **No verificado** el código exacto); tampoco hay un validador de la consulta del listado.

### 8.3 Órdenes magistrales

Los nombres de los campos de `POST /api/v1/Magistrales/iniciar` se deducen de los códigos de error: `productoResultanteId`, `cantidadProducida`, `consumos[]` con `inventarioSucursalId` y `cantidadConsumida`; el identificador de la receta es opcional y su nombre es **No verificado**.

### HU-OPS-011 — Iniciar una orden magistral

#### TC-OPS-066 — Iniciar una orden magistral

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-OPS-011] «Iniciar una orden»; [CU-OPS-11] flujo básico; [RN-MAG-001], [RN-MAG-002], [RN-MAG-003], [RN-MAG-004] |

**Precondiciones:** token con `Magistrales:IniciarOrdenMagistral` de «Sede Principal»; el producto resultante existe; el insumo `LOT-QA-P01` tiene fila de stock en «Sede Principal» (stock 500).
**Petición:** `POST /api/v1/Magistrales/iniciar` con producto resultante, `cantidadProducida` 50 y un consumo (`inventarioSucursalId` de la fila de `LOT-QA-P01`, `cantidadConsumida` 10).
**Resultado esperado:** HTTP 200 con el identificador de la orden; la orden queda `Iniciada` con la sucursal y el empleado de la sesión como químico preparador y la hora actual UTC; el stock del insumo baja 10 (de 500 a 490) y se guarda un consumo con unidad de medida fija `UND`.

#### TC-OPS-067 — La sucursal y el químico salen de la sesión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-011] «La sucursal y el químico salen de mi sesión»; [CU-OPS-11] excepciones; [RN-MAG-005] |

**Petición:** `POST /api/v1/Magistrales/iniciar` con un empleado que no es químico, y con un token sin sucursal o empleado.
**Resultado esperado:** la sucursal y el químico no se eligen en el request ni se comprueba que el empleado sea químico; sin sucursal o empleado: HTTP 400, `code` `Auth.Sucursal` o `Auth.Empleado`.

#### TC-OPS-068 — Datos obligatorios de la orden

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-011] «Datos obligatorios»; [CU-OPS-11] excepciones; [RN-MAG-006] |

**Petición:** `POST /api/v1/Magistrales/iniciar` con producto vacío; cantidad nula y 0; cero insumos; fila de stock vacía; cantidad consumida 0.
**Resultado esperado:** HTTP 400 con `code` `IniciarOrdenMagistralCommand.ProductoResultanteId`, `.CantidadProducida`, `.Consumos`, `.Consumos[0].InventarioSucursalId` y `.Consumos[0].CantidadConsumida`, respectivamente.

#### TC-OPS-069 — El producto o la receta no existen

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-011] «El producto o la receta no existen»; [CU-OPS-11] excepciones; [RN-MAG-007] |

**Petición:** `POST /api/v1/Magistrales/iniciar` con un producto inexistente y con una receta inexistente.
**Resultado esperado:** HTTP 404, `code` `Medicamento.NotFound` o `RecetaMedica.NotFound`; solo se comprueba la existencia: no que la receta sea del paciente, esté vigente, ni que el producto sea magistral.

#### TC-OPS-070 — El insumo es de otra sucursal o de otra empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-OPS-011] «El insumo es de otra sucursal»; [CU-OPS-11] excepciones; [RN-MAG-008] |

**Petición:** `POST /api/v1/Magistrales/iniciar` con la fila de stock de `LOT-QA-N01` (Norte) y con una fila de «Botica Otra Demo S.A.C.».
**Resultado esperado:** HTTP 404, `code` `Inventario.NotFound`; no cambia ningún stock.

#### TC-OPS-071 — Insumo con lote vencido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-OPS-011] «Insumo con lote vencido»; [CU-OPS-11] excepciones; [RN-MAG-009] |

**Petición:** `POST /api/v1/Magistrales/iniciar` con la fila de stock de `LOT-QA-P03` (vencido); repite con un lote que caduca hoy.
**Resultado esperado:** el vencido: HTTP 400, `code` `Magistral.Insumo.LoteVencido`; el que caduca hoy se acepta (el lote se usa hasta el final de su día de caducidad).

#### TC-OPS-072 — Stock insuficiente del insumo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-OPS-011] «Stock insuficiente»; [CU-OPS-11] excepciones; [RN-MAG-010] |

**Precondiciones:** un insumo con 8 unidades.
**Petición:** `POST /api/v1/Magistrales/iniciar` con `cantidadConsumida` 10; luego 8.
**Resultado esperado:** 10: HTTP 400, `code` `Inventario.StockInsuficiente` y no se guarda nada de la orden; 8: se acepta y el stock queda en 0.

#### TC-OPS-073 — El mismo insumo en dos líneas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-011] «El mismo insumo en dos líneas»; [CU-OPS-11] flujo básico; [RN-MAG-010] |

**Precondiciones:** un insumo con 8 unidades.
**Petición:** `POST /api/v1/Magistrales/iniciar` con dos consumos del mismo insumo de 5 y 5.
**Resultado esperado:** se consume dos veces y la comprobación de stock ve lo ya descontado: la segunda línea falla con `Inventario.StockInsuficiente` (5 + 5 > 8) y no se guarda nada.

#### TC-OPS-074 — La cuarentena no impide usar un lote como insumo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-011] «La cuarentena no impide usar un lote como insumo»; [CU-OPS-11] flujo básico; [RN-MAG-011] |

**Petición:** `POST /api/v1/Magistrales/iniciar` con la fila de stock de `LOT-QA-P04` («Retenido»).
**Resultado esperado:** un lote con registro «Retenido» puede consumirse (se acepta la orden).
**Hallazgo:** la cuarentena solo la respeta la venta (01d §9.3).

#### TC-OPS-075 — Una orden iniciada no se puede deshacer

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-011] «Una orden iniciada no se puede deshacer»; [CU-OPS-11] alternativo A1; [RN-MAG-020] |

**Pasos:** tras `TC-OPS-066`, busca en la API una forma de cancelar o anular la orden y de devolver los insumos.
**Resultado esperado:** no existe cancelación, anulación ni devolución de insumos; una orden iniciada y nunca completada deja el stock descontado para siempre.

#### TC-OPS-076 — Conflicto de concurrencia sobre el stock del insumo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-OPS-011] «Conflicto de concurrencia sobre el stock»; [CU-OPS-11] excepciones; [RN-MAG-021] |

**Petición:** inicia dos órdenes en paralelo con el mismo insumo.
**Resultado esperado:** si se detecta el conflicto de versión de fila, una recibe HTTP 409, `code` `Concurrency.Conflict`. **No verificado en ejecución.**

#### TC-OPS-077 — Estructura y dependencias de las órdenes magistrales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-011] «Estructura»; [CU-OPS-11] excepciones; [RN-MAG-022], [RN-MAG-023] |

**Pasos:** revisa las restricciones de las tablas de la orden y sus consumos; intenta eliminar (parte B) el medicamento y la sucursal con órdenes.
**Resultado esperado:** la base impone las claves foráneas, sin restricciones sobre el estado ni la cantidad; desde el commit `27597cd` un medicamento o una sucursal con órdenes magistrales no se puede eliminar (HTTP 409, `Medicamento.InUse`, `Sucursal.InUse`).
**Hallazgo:** el estado por defecto de la base (`En_Proceso`) no coincide con el que escribe el código (`Iniciada`) (01d hallazgo 12).

### HU-OPS-012 — Completar una orden magistral

#### TC-OPS-078 — Completar la orden

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | [HU-OPS-012] «Completar la orden»; [CU-OPS-12] flujo básico; [RN-MAG-012], [RN-MAG-013], [RN-LOT-004], [RN-LOT-010] |

**Precondiciones:** orden `Iniciada` de 50 unidades de `TC-OPS-066`; token con `Magistrales:CompletarOrdenMagistral` de la misma sucursal.
**Petición:** `POST /api/v1/Magistrales/completar` con el identificador, número de lote `MAG-0001` y una fecha de caducidad futura (`ordenId`, `numeroLoteMfr`, `fechaCaducidad`).
**Resultado esperado:** HTTP 200 con el identificador de la orden; se crea un lote del producto resultante con fecha de fabricación igual a la hora actual; se crea su fila de stock en la sucursal de la orden con 50 unidades; la orden queda `Completada` con el lote generado (estado final).

#### TC-OPS-079 — Datos de completar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-012] «Datos de completar»; [CU-OPS-12] excepciones; [RN-MAG-014], [RN-LOT-006] |

**Petición:** `POST /api/v1/Magistrales/completar` con identificador vacío; número de lote vacío y de 51 caracteres (y de 50, válido); caducidad ya vencida.
**Resultado esperado:** HTTP 400 con `code` `CompletarOrdenMagistralCommand.OrdenId`, `.NumeroLoteMfr` y `.FechaCaducidad`; el número de lote admite 50 caracteres aunque un lote normal admite 100.

#### TC-OPS-080 — La orden no existe al completar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-012] «La orden no existe»; [CU-OPS-12] excepciones; [RN-MAG-015] |

**Petición:** `POST /api/v1/Magistrales/completar` con un GUID inexistente.
**Resultado esperado:** HTTP 404, `code` `Orden.NotFound`.

#### TC-OPS-081 — La orden es de otra sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-OPS-012] «La orden es de otra sucursal»; [CU-OPS-12] excepciones; [RN-MAG-013] |

**Petición:** `POST /api/v1/Magistrales/completar` con el token de Nora (u otro administrador de otra sucursal) sobre una orden de «Sede Principal».
**Resultado esperado:** HTTP 403, `code` `OrdenMagistral.OtraSucursal`, también si es administrador; la orden sigue `Iniciada`.

#### TC-OPS-082 — Completar una orden ya completada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-OPS-012] «Completar una orden ya completada»; [CU-OPS-12] excepciones; [RN-MAG-016] |

**Petición:** repite `POST /api/v1/Magistrales/completar` sobre la orden de `TC-OPS-078`; luego dos completados simultáneos de una orden `Iniciada`.
**Resultado esperado:** el repetido: HTTP 400, `code` `Orden.Estado`; los simultáneos: **No verificado:** la orden no tiene versión de fila, dos completados simultáneos podrían pasar los dos (duplicando el lote o fallando por número repetido).

#### TC-OPS-083 — Cualquier empleado con permiso de la sucursal puede completarla

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Baja | [HU-OPS-012] «Cualquier empleado con permiso de la sucursal puede completarla»; [CU-OPS-12] flujo básico; [RN-MAG-012] |

**Petición:** completa una orden iniciada por otro empleado de la misma sucursal.
**Resultado esperado:** se completa; no hace falta ser quien la inició.

#### TC-OPS-084 — Número de lote repetido para el producto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-012] «Número de lote repetido para el producto»; [CU-OPS-12] excepciones; [RN-MAG-017] |

**Precondiciones:** un lote del producto resultante con número `MAG-0001`; otra orden `Iniciada` del mismo producto.
**Petición:** `POST /api/v1/Magistrales/completar` con `MAG-0001`.
**Resultado esperado:** HTTP 409, `code` `Db.Duplicado` (lo rechaza la base al guardar, no una comprobación previa); la orden sigue `Iniciada`.

#### TC-OPS-085 — No se informa la cantidad realmente obtenida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-012] «No se informa la cantidad realmente obtenida»; [CU-OPS-12] flujo básico; [RN-MAG-013] |

**Pasos:** completa una orden iniciada con `cantidadProducida` 50 y revisa el stock del lote nuevo.
**Resultado esperado:** el stock del lote nuevo es la cantidad declarada al iniciar (50); la petición de completar no admite otra cantidad.
**Hallazgo:** la cantidad producida se declara al iniciar y no se corrige al completar (01d hallazgo 4).

### HU-OPS-013 — Consultar órdenes magistrales

#### TC-OPS-086 — Listar órdenes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-OPS-013] «Listar órdenes»; [CU-OPS-13] flujo básico; [RN-MAG-018], [RN-GEN-023] |

**Precondiciones:** órdenes en «Sede Principal» y «Sucursal Demo Norte»; `Magistrales:Leer`.
**Petición:** `GET /api/v1/Magistrales/ordenes` con el token de Carlos y de Ana (Admin).
**Resultado esperado:** HTTP 200 con las órdenes de las sucursales accesibles (el administrador, todas), por fecha de preparación descendente, de a 10; cada fila trae identificador, sucursal, producto, cantidad, estado y fecha.

#### TC-OPS-087 — Filtros de las órdenes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-013] «Filtros»; [CU-OPS-13] flujo básico; [RN-MAG-018], [RN-GEN-008] |

**Petición:** `GET /api/v1/Magistrales/ordenes` con sucursal, estado (texto), fechas de preparación desde y hasta, una sucursal no accesible y una sucursal igual al GUID vacío.
**Resultado esperado:** filtra por sucursal; estado por igualdad sin validar; fechas inclusivas en UTC; la sucursal no accesible devuelve una página vacía; el GUID vacío responde HTTP 400, `code` `GetOrdenesQuery.SucursalId`.

#### TC-OPS-088 — Consultar una orden

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-013] «Consultar una orden»; [CU-OPS-13] flujo básico; [RN-MAG-019] |

**Petición:** `GET /api/v1/Magistrales/{id}` de una orden de una sucursal accesible.
**Resultado esperado:** HTTP 200 con el objeto de resultado completo y los mismos seis campos en `value`; no incluye insumos, lote generado, receta ni químico.

#### TC-OPS-089 — La orden no existe o no es accesible

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Baja | [HU-OPS-013] «La orden no existe o no es accesible»; [CU-OPS-13] excepciones; [RN-MAG-019] |

**Petición:** `GET /api/v1/Magistrales/{id}` con un GUID inexistente y con una orden de una sucursal no accesible.
**Resultado esperado:** HTTP 404, `code` `OrdenMagistral.NotFound` en ambos casos.

### 8.4 Servicios e inmunizaciones

La historia HU-OPS-014 advierte que la tabla `Servicios_Agenda` del script `0001` no tiene las columnas `LastModifiedAt`, `LastModifiedBy`, `DeletedAt` ni `DeletedBy` que EF mapea: si la base se creó solo con scripts, agendar, listar, consultar y eliminar citas podrían fallar con un error 500 (RN-SRV-019, **No verificado** en ejecución). Antes de ejecutar estos casos revisa en tu base si esas columnas existen; si no, el resultado esperado de los casos de citas pasa a ser ese error.

### HU-OPS-014 — Agendar un servicio

#### TC-OPS-090 — Agendar una cita

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-014] «Agendar una cita»; [CU-OPS-14] flujo básico; [RN-SRV-001], [RN-SRV-002], [RN-SRV-019] |

**Precondiciones:** token con `Servicios:AgendarServicio`; «Rosa Quispe Demo» y un producto existen.
**Petición:** `POST /api/v1/Servicios/agenda` con el paciente, el producto, la fecha y hora programada y el estado `Programada` (los nombres exactos de los campos son **No verificado**).
**Resultado esperado:** HTTP 200 con el identificador de la cita. **No verificado en ejecución:** puede fallar con error 500 por columnas faltantes (nota de la sección 8.4).

#### TC-OPS-091 — Paciente, producto o venta inexistentes al agendar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-014] «Paciente, producto o venta inexistentes»; [CU-OPS-14] excepciones; [RN-SRV-003], [RN-SRV-004] |

**Petición:** `POST /api/v1/Servicios/agenda` con un paciente inexistente; con un producto inexistente; con una venta inexistente; y con una venta vacía (GUID vacío).
**Resultado esperado:** HTTP 404, `code` `Paciente.NotFound`, `Medicamento.NotFound` o `Venta.NotFound`; la venta es opcional: un identificador vacío se guarda como «sin venta».

#### TC-OPS-092 — Sin reglas sobre estado ni fecha

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-014] «Sin reglas sobre estado ni fecha»; [CU-OPS-14] flujo básico; [RN-SRV-005], [RN-SRV-021] |

**Petición:** `POST /api/v1/Servicios/agenda` con estado de 21 caracteres, estado vacío o de espacios, una fecha pasada, dos citas a la misma hora y un producto que no es un servicio.
**Resultado esperado:** el estado es un texto libre de hasta 20 caracteres, no vacío ni de espacios (por defecto `Programada`): el de 21 y el vacío se rechazan; la fecha puede ser pasada o futura, no hay horario de atención ni choque de citas; el producto puede ser cualquier medicamento.
**Hallazgo:** no se exige que el producto sea un servicio (01d).

#### TC-OPS-093 — Agendar sin validador

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-014] «Sin validador»; [CU-OPS-14] excepciones; [RN-SRV-019] |

**Petición:** `POST /api/v1/Servicios/agenda` con el estado vacío.
**Resultado esperado:** no se validan campos antes del handler; el error sale con un código de la entidad, por ejemplo `ServicioAgenda.EstadoCita`.

### HU-OPS-015 — Registrar una inmunización

Los nombres de los campos del cuerpo de `POST /api/v1/Servicios/inmunizacion` son **No verificado**; la historia nombra: vía (`Intramuscular`), sitio anatómico (`Deltoides`), volumen (0.5), modalidad (`Corriente` o `Historica`), fecha de administración (`FechaAdmnFisica`) y la fecha de entrega de la hoja informativa (opcional).

#### TC-OPS-094 — Registrar una dosis

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-015] «Registrar una dosis»; [CU-OPS-15] flujo básico; [RN-SRV-007], [RN-SRV-008], [RN-SRV-010] |

**Precondiciones:** token con `Servicios:RegistrarInmunizacion`; paciente, profesional (cualquier empleado), producto y un lote del producto existen.
**Petición:** `POST /api/v1/Servicios/inmunizacion` con vía `Intramuscular`, sitio `Deltoides`, volumen 0.5, modalidad `Corriente` y fecha de administración.
**Resultado esperado:** HTTP 200 con el identificador del registro.

#### TC-OPS-095 — La fecha de administración es obligatoria

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-015] «La fecha de administración es obligatoria»; [CU-OPS-15] excepciones; [RN-SRV-009] |

**Petición:** `POST /api/v1/Servicios/inmunizacion` con la fecha de administración por defecto (`0001-01-01`).
**Resultado esperado:** HTTP 400, `code` `RegistrarInmunizacionCommand.FechaAdmnFisica`.

#### TC-OPS-096 — Inexistentes al registrar una inmunización

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-015] «Inexistentes»; [CU-OPS-15] excepciones; [RN-SRV-010] |

**Petición:** `POST /api/v1/Servicios/inmunizacion` con paciente, empleado (profesional), producto o venta inexistentes; con un empleado que no es profesional de salud.
**Resultado esperado:** HTTP 404, `code` `Paciente.NotFound`, `Empleado.NotFound`, `Medicamento.NotFound` o `Venta.NotFound` (la venta es opcional); un empleado que no es profesional de salud se acepta.

#### TC-OPS-097 — Lote de otro producto o inexistente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-015] «Lote de otro producto o inexistente»; [CU-OPS-15] excepciones; [RN-SRV-011] |

**Petición:** `POST /api/v1/Servicios/inmunizacion` con el lote de otro producto y con un lote inexistente.
**Resultado esperado:** HTTP 404, `code` `LoteInventario.NotFound`.

#### TC-OPS-098 — Lote vencido en la fecha de administración

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | [HU-OPS-015] «Lote vencido en la fecha de administración»; [CU-OPS-15] excepciones; [RN-SRV-012] |

**Precondiciones:** un lote que vencía el 30/06/2026.
**Petición:** `POST /api/v1/Servicios/inmunizacion` con fecha de administración 15/07/2026; luego con 30/06/2026; luego con 15/06/2026 (histórica).
**Resultado esperado:** 15/07/2026: HTTP 400, `code` `Inmunizacion.Lote.Vencido`; 15/06/2026: se acepta (una inmunización histórica puede registrarse con un lote que venció después de la dosis). **No verificado:** el comportamiento exacto del día de caducidad (30/06/2026); la regla general es que un lote se usa hasta el final de su día de caducidad.

#### TC-OPS-099 — Datos clínicos de la inmunización

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-015] «Datos clínicos»; [CU-OPS-15] excepciones; [RN-SRV-013] |

**Petición:** `POST /api/v1/Servicios/inmunizacion` con vía vacía y de 51 caracteres; sitio vacío y de 101; volumen 0 y negativo; una fecha futura; un volumen enorme; sin fecha de entrega de la hoja informativa.
**Resultado esperado:** la vía es obligatoria y de hasta 50 caracteres; el sitio obligatorio y de hasta 100; el volumen mayor que 0 (0 y negativo se rechazan); la fecha de entrega de la hoja informativa es opcional; no hay catálogo de vías ni de sitios, ni límite de volumen, ni se rechaza una fecha futura (se aceptan).

#### TC-OPS-100 — Modalidad de registro

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-OPS-015] «Modalidad de registro»; [CU-OPS-15] flujo básico; [RN-SRV-013] |

**Petición:** `POST /api/v1/Servicios/inmunizacion` con modalidad `Corriente` y con `Historica`.
**Resultado esperado:** ambas se guardan; ninguna regla cambia según sea corriente o histórica.

#### TC-OPS-101 — Registrar una dosis no consume stock ni consulta la cuarentena

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | [HU-OPS-015] «Registrar una dosis no consume stock»; [CU-OPS-15] flujo básico; [RN-SRV-015] |

**Precondiciones:** `LOT-QA-P04` en cuarentena «Retenido» y un lote sin existencia.
**Petición:** `POST /api/v1/Servicios/inmunizacion` con cada uno; revisa el stock del lote antes y después.
**Resultado esperado:** se registran: el sistema no lee ni modifica el stock, no comprueba que el lote tenga existencia y no consulta la cuarentena; un lote «Retenido» puede registrarse.
**Hallazgo:** «Consumo de lote» significa hoy solo «dejar registrado qué lote se usó» (01d hallazgo 3).

#### TC-OPS-102 — Sin sucursal ni auditoría

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-OPS-015] «Sin sucursal ni auditoría»; [CU-OPS-15] flujo básico; [RN-SRV-020] |

**Pasos:** tras `TC-OPS-094` revisa la fila en la base y la tabla de auditoría.
**Resultado esperado:** el registro no tiene sucursal ni quién lo registró (solo el usuario de creación) y no deja evento de auditoría.

#### TC-OPS-103 — Volumen con 3 o 4 decimales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | [HU-OPS-015] «Volumen con 3 o 4 decimales»; [CU-OPS-15] excepciones; [RN-SRV-013] |

**Petición:** `POST /api/v1/Servicios/inmunizacion` con volumen 0.1234.
**Resultado esperado:** **No verificado:** el script define `DECIMAL(8,2)` y EF declara `DECIMAL(12,4)`; no hay resultado documentado.

#### TC-OPS-104 — Dependencias de pacientes y medicamentos con citas o inmunizaciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-015] «Dependencias»; [CU-OPS-15] excepciones; [RN-SRV-020] |

**Pasos:** intenta eliminar (parte B) un paciente y un medicamento con citas o inmunizaciones.
**Resultado esperado:** desde el commit `27597cd` no se pueden eliminar: HTTP 409, `Paciente.InUse` y `Medicamento.InUse`.

### HU-OPS-016 — Consultar la agenda y las inmunizaciones

#### TC-OPS-105 — Listar la agenda

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-016] «Listar la agenda»; [CU-OPS-16] flujo básico; [RN-SRV-006] |

**Precondiciones:** citas en estados distintos y fechas distintas; `Servicios:Leer`.
**Petición:** `GET /api/v1/Servicios/agenda` con estado y fechas programadas desde y hasta.
**Resultado esperado:** HTTP 200 con las citas de toda la empresa, por fecha programada descendente, de a 10; cada fila trae identificador, paciente, producto, fecha y hora y estado; no se filtra por sucursal (con Nora se ven las citas de «Sede Principal»).

#### TC-OPS-106 — Listar inmunizaciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-016] «Listar inmunizaciones»; [CU-OPS-16] flujo básico; [RN-SRV-014] |

**Petición:** `GET /api/v1/Servicios/inmunizaciones` con paciente y fechas de administración; luego con un paciente igual al GUID vacío.
**Resultado esperado:** identificador, paciente, fecha y vía de administración, por fecha descendente; el GUID vacío responde HTTP 400, `code` `GetInmunizacionesQuery.ClienteId`.

#### TC-OPS-107 — Detalle por identificador

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | [HU-OPS-016] «Detalle por identificador»; [CU-OPS-16] flujo básico; [RN-SRV-016] |

**Petición:** `GET /api/v1/Servicios/servicio/{id}` y `GET /api/v1/Servicios/inmunizacion/{id}` con identificadores existentes y con uno inexistente.
**Resultado esperado:** HTTP 200 con el objeto de resultado completo y en `value` únicamente el identificador; si no existe, el código literal `NotFound` con HTTP 404.

### HU-OPS-017 — Actualizar o eliminar una cita o una inmunización

#### TC-OPS-108 — «Actualizar» no cambia nada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-017] «"Actualizar" no cambia nada»; [CU-OPS-17] flujo básico; [RN-SRV-017] |

**Petición:** `PUT /api/v1/Servicios/servicio/{id}` con el mismo identificador en el cuerpo y un estado distinto; consulta la cita.
**Resultado esperado:** HTTP 200 con el objeto de resultado, pero no se modifica ningún dato; no se puede corregir una cita, una dosis ni el estado de una cita después de creada.
**Hallazgo:** actualizar no está implementado (Parcial).

#### TC-OPS-109 — Identificador de ruta distinto del del cuerpo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-017] «Identificador de ruta distinto del del cuerpo»; [CU-OPS-17] excepciones; [RN-SRV-017] |

**Petición:** `PUT /api/v1/Servicios/servicio/{id}` con un identificador distinto en el cuerpo.
**Resultado esperado:** HTTP 400 con un texto plano «El ID de la ruta no coincide con el comando.» y sin código.

#### TC-OPS-110 — Actualizar o eliminar algo inexistente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-017] «Actualizar algo inexistente», «Eliminar algo inexistente»; [CU-OPS-17] excepciones; [RN-SRV-018] |

**Petición:** `PUT` y `DELETE` sobre `/api/v1/Servicios/servicio/{id}` y `/api/v1/Servicios/inmunizacion/{id}` con identificadores inexistentes.
**Resultado esperado:** HTTP 404 con el código literal `NotFound` en los cuatro casos.

#### TC-OPS-111 — Eliminar una cita o una inmunización

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-017] «Eliminar una cita o una inmunización»; [CU-OPS-17] flujo básico; [RN-SRV-018], [RN-SRV-021] |

**Petición:** `DELETE /api/v1/Servicios/inmunizacion/{id}` con el token de Nora sobre un registro de «Sede Principal».
**Resultado esperado:** el registro queda eliminado (borrado lógico) y HTTP 200 con el objeto de resultado; no se pide motivo ni queda rastro de auditoría, aunque sea un registro clínico; no se restringe por sucursal. Consultarlo después da `NotFound`.

### 8.5 Delivery

### HU-OPS-018 — Programar un delivery

#### TC-OPS-112 — Programar el delivery de una venta cobrada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-018] «Programar el delivery de una venta cobrada»; [CU-OPS-18] flujo básico; [RN-DEL-001], [RN-DEL-002], [RN-DEL-003] |

**Precondiciones:** token con `Delivery:ProgramarDelivery`; venta «Completada» de una sucursal accesible.
**Petición:** `POST /api/v1/Delivery` con la venta, plataforma `Rappi`, dirección «Av. Ficticia 123, Miraflores» y repartidor opcional (los nombres exactos de los campos son **No verificado**).
**Resultado esperado:** HTTP 200 (no 201) con el identificador del despacho; el despacho nace en `Preparando` y no afecta la venta, el stock ni el comprobante.

#### TC-OPS-113 — Venta inexistente al programar un delivery

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-018] «Venta inexistente»; [CU-OPS-18] excepciones; [RN-DEL-004] |

**Petición:** `POST /api/v1/Delivery` con una venta inexistente, con una venta de otra empresa y con el identificador vacío.
**Resultado esperado:** HTTP 404, `code` `Venta.NotFound` en los tres casos.

#### TC-OPS-114 — Sucursal de la venta no accesible

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-OPS-018] «Sucursal de la venta no accesible»; [CU-OPS-18] excepciones; [RN-DEL-005] |

**Petición:** `POST /api/v1/Delivery` con el token de Carlos sobre una venta de «Sucursal Demo Norte»; repite con Ana (Admin).
**Resultado esperado:** Carlos: HTTP 403, `code` `Delivery.Venta.SucursalNoPermitida`; Ana (administrador) accede a todas las sucursales y el despacho se crea.

#### TC-OPS-115 — Venta anulada y estados aceptados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-018] «Venta anulada»; [CU-OPS-18] excepciones; [RN-DEL-006] |

**Petición:** `POST /api/v1/Delivery` con ventas `Anulada`, `Pendiente`, `Completada` y `Devuelta`.
**Resultado esperado:** la `Anulada`: HTTP 409, `code` `Delivery.Venta.Anulada`; las otras tres se aceptan.

#### TC-OPS-116 — La venta ya tiene un despacho

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | [HU-OPS-018] «La venta ya tiene un despacho»; [CU-OPS-18] excepciones; [RN-DEL-007], [RN-DEL-008] |

**Petición:** `POST /api/v1/Delivery` dos veces sobre la misma venta; luego dos simultáneas sobre una venta sin despacho; luego elimina el despacho (`TC-OPS-128`) y programa otro.
**Resultado esperado:** la segunda: HTTP 409, `code` `Delivery.Venta.YaProgramado`; en la simultánea, la segunda recibe HTTP 409 `Db.Duplicado`; eliminar el despacho libera la venta para programar otro.

#### TC-OPS-117 — Datos del despacho: longitudes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-018] «Datos del despacho»; [CU-OPS-18] excepciones; [RN-DEL-016], [RN-DEL-017] |

**Petición:** `POST /api/v1/Delivery` con plataforma vacía y de 51 caracteres; dirección vacía y de 256; código de rastreo de 101; repartidor de 151; URL de evidencia de 501 y una URL de formato inválido.
**Resultado esperado:** la plataforma es obligatoria y de hasta 50; la dirección obligatoria y de hasta 255; el código de rastreo opcional y de hasta 100; el repartidor opcional y de hasta 150; la URL de evidencia opcional y de hasta 500. Los excesos responden HTTP 400 con el código de la entidad (por ejemplo `DespachoDelivery.PlataformaServicio`; el validador está vacío). No se valida el formato de la URL, ni que la dirección sea la del paciente, ni un catálogo de plataformas (se aceptan).

#### TC-OPS-118 — Un despacho no tiene relación con el resto del negocio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-018] «Un despacho no tiene relación con el resto del negocio»; [CU-OPS-18] flujo básico; [RN-DEL-018] |

**Pasos:** programa un delivery; revisa el comprobante y el arqueo; anula o devuelve la venta; consulta el despacho.
**Resultado esperado:** el despacho no tiene costo, no se refleja en el comprobante ni en el arqueo, y anular o devolver la venta después no cambia el despacho.

### HU-OPS-019 — Cambiar el estado de un delivery

#### TC-OPS-119 — Despacho en camino

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-019] «Despacho en camino»; [CU-OPS-19] flujo básico; [RN-DEL-009] |

**Precondiciones:** despacho `Preparando`; token con `Delivery:Actualizar`.
**Petición:** `PUT /api/v1/Delivery/{id}/estado` con `{"id":"<id>","nuevoEstado":"En_Camino"}`.
**Resultado esperado:** HTTP 200 con el identificador; el despacho queda `En_Camino` (se verifica en el listado, `TC-OPS-125`).

#### TC-OPS-120 — Transiciones de estado del delivery (esquema)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-019] «Transiciones»; [CU-OPS-19] flujo básico y excepciones; [RN-DEL-010] |

**Petición:** `PUT /api/v1/Delivery/{id}/estado` para cada fila de la tabla (un despacho nuevo por fila).
**Resultado esperado:**

| Actual | Nuevo | Resultado |
|---|---|---|
| Preparando | En_Camino | permitido |
| Preparando | Devuelto | permitido |
| En_Camino | Entregado | permitido |
| En_Camino | Devuelto | permitido |
| Preparando | Entregado | HTTP 400 `Delivery.Estado.TransicionInvalida` |
| Entregado | En_Camino | HTTP 400 `Delivery.Estado.TransicionInvalida` |
| Devuelto | En_Camino | HTTP 400 `Delivery.Estado.TransicionInvalida` |
| Entregado | Entregado | válido, no cambia nada (el mismo estado) |

#### TC-OPS-121 — El cuerpo manda, no la ruta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-019] «El cuerpo manda, no la ruta»; [CU-OPS-19] excepciones; [RN-DEL-011] |

**Petición:** `PUT /api/v1/Delivery/{idA}/estado` con el cuerpo `{"id":"<idB>","nuevoEstado":"En_Camino"}`; luego con `id` vacío; luego con un estado fuera del enumerado.
**Resultado esperado:** el identificador de la ruta no se usa: se cambia el despacho `idB`; el `id` vacío o el estado fuera del enumerado responden HTTP 400, `code` `UpdateDeliveryEstadoCommand.Id` o `UpdateDeliveryEstadoCommand.NuevoEstado`.

#### TC-OPS-122 — El despacho no existe al cambiar el estado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-019] «El despacho no existe»; [CU-OPS-19] excepciones; [RN-DEL-009] |

**Petición:** `PUT /api/v1/Delivery/{id}/estado` con un identificador inexistente.
**Resultado esperado:** HTTP 404 con el código literal `Delivery`.

#### TC-OPS-123 — No se comprueba la sucursal al cambiar el estado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | [HU-OPS-019] «No se comprueba la sucursal»; [CU-OPS-19] flujo básico; [RN-DEL-009] |

**Precondiciones:** despacho de una venta de «Sede Principal»; Nora (Norte) con `Delivery:Actualizar`.
**Petición:** `PUT /api/v1/Delivery/{id}/estado` con el token de Nora.
**Resultado esperado:** el cambio se acepta: quien tenga `Delivery:Actualizar` puede cambiar el estado de un despacho de cualquier sucursal de su empresa; uno de «Botica Otra Demo S.A.C.» no (404).
**Hallazgo:** el cambio de estado no revisa la sucursal, a diferencia de programar (01d).

#### TC-OPS-124 — Pasar a «Entregado» o «Devuelto» no afecta la venta ni el stock

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-019] «Pasar a "Entregado" o "Devuelto" no tiene efecto en la venta ni el stock»; [CU-OPS-19] flujo básico; [RN-DEL-011] |

**Pasos:** lleva un despacho a `Entregado` y otro a `Devuelto`; revisa la venta, el stock y el arqueo.
**Resultado esperado:** son solo seguimiento: la venta, el stock y el arqueo no cambian.

### HU-OPS-020 — Consultar, «actualizar» y eliminar despachos de delivery

#### TC-OPS-125 — Listar despachos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | [HU-OPS-020] «Listar despachos»; [CU-OPS-20] flujo básico; [RN-DEL-012] |

**Precondiciones:** despachos de ventas de «Sede Principal» y de Norte; `Delivery:Leer`.
**Petición:** `GET /api/v1/Delivery/despachos` con estado y fechas desde y hasta con el token de Carlos.
**Resultado esperado:** HTTP 200 con los despachos cuya venta es de una sucursal accesible, de a 10; cada fila trae solo identificador, venta, estado y dirección de entrega; las fechas se comparan tal cual con la fecha de creación, sin normalizar a UTC; el orden es por identificador descendente, que no es cronológico.

#### TC-OPS-126 — El detalle del despacho solo devuelve el identificador

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-020] «El detalle solo devuelve el identificador»; [CU-OPS-20] flujo básico; [RN-DEL-013] |

**Petición:** `GET /api/v1/Delivery/{id}` con un despacho accesible, con uno inexistente y con uno de una sucursal no accesible.
**Resultado esperado:** el existente: HTTP 200 con el objeto de resultado cuyo `value` tiene únicamente el identificador (para ver el estado hay que usar el listado); los otros: HTTP 404 con el código literal `NotFound`.

#### TC-OPS-127 — «Actualizar» un despacho no hace nada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-020] «"Actualizar" un despacho no hace nada»; [CU-OPS-20] flujo básico; [RN-DEL-014] |

**Petición:** `PUT /api/v1/Delivery/{id}` con otra dirección y repartidor; consulta el listado.
**Resultado esperado:** HTTP 200 con el objeto de resultado sin cambios; plataforma, rastreo, dirección, repartidor y evidencia no se pueden corregir después de programar.

#### TC-OPS-128 — Eliminar un despacho

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-020] «Eliminar un despacho»; [CU-OPS-20] flujo básico; [RN-DEL-015] |

**Petición:** `DELETE /api/v1/Delivery/{id}` sobre un despacho `Entregado`; luego sobre uno inexistente; luego programa otro para la misma venta.
**Resultado esperado:** el despacho queda eliminado (borrado lógico) y la venta queda libre para programar otro; no se comprueba sucursal ni estado (se puede eliminar un despacho `Entregado`); el inexistente responde HTTP 404 con el código literal `NotFound`.

### 8.6 Auditoría

### HU-OPS-021 — Solicitar la anonimización de datos

#### TC-OPS-129 — La anonimización responde «no implementado»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-021] «La anonimización responde "no implementado"»; [CU-OPS-21] flujo básico; [RN-AUD-001], [RN-AUD-002] |

**Precondiciones:** token con `Auditoria:AnonimizarDatos`.
**Petición:** `POST /api/v1/Auditoria/anonimizar` con la tabla `Pacientes` y el identificador de «Rosa Quispe Demo» (`tabla`, `registroId`).
**Resultado esperado:** HTTP 501, `code` `Auditoria.Anonimizacion.NoImplementada`, con el detalle «Data anonymization is not implemented yet; no data was changed.»; ningún dato se modifica (los datos de Rosa siguen iguales) y no se registra ningún evento de auditoría. **No verificado:** la forma exacta del `ProblemDetails`.

#### TC-OPS-130 — Datos obligatorios de la anonimización

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | [HU-OPS-021] «Datos obligatorios»; [CU-OPS-21] excepciones; [RN-AUD-003] |

**Petición:** `POST /api/v1/Auditoria/anonimizar` con la tabla vacía; con el identificador vacío; con el GUID vacío; con una tabla que no existe.
**Resultado esperado:** HTTP 400, `code` `AnonimizarDatosCommand.Tabla` o `AnonimizarDatosCommand.RegistroId` en los tres primeros; con una tabla inexistente no se valida que exista ni que sea anonimizable (responde 501).

#### TC-OPS-131 — Orden de evaluación y por qué 501

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Baja | [HU-OPS-021] «Orden de evaluación», «Por qué 501 y no un falso éxito»; [CU-OPS-21] excepciones; [RN-AUD-004] |

| Situación | Qué responde |
|---|---|
| Sin `Auditoria:AnonimizarDatos` | 403 (primero se comprueba el permiso) |
| Con permiso y datos inválidos | 400 del validador |
| Con permiso y datos válidos | 501 (último, el handler) |

**Petición:** una petición por fila de la tabla.
**Resultado esperado:** según la tabla; el 501 evita confirmar falsamente la solicitud de un titular de datos.

### HU-OPS-022 — Saber qué operaciones dejan evento de auditoría

#### TC-OPS-132 — Un comando auditable exitoso deja un evento (esquema de eventos)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | [HU-OPS-022] «Un comando auditable exitoso deja un evento», «Eventos de esta parte»; [CU-OPS-22] flujo básico; [RN-AUD-005], [RN-AUD-006], [RN-GEN-013] |

**Pasos:** anula una venta, procesa una devolución, abre y cierra una caja y ajusta un stock (parte B); después consulta directamente la tabla `Auditoria_Eventos_Seguridad` en la base.
**Resultado esperado:** por cada operación hay una fila con el empleado que actúa, la tabla afectada, el identificador del registro, la acción, un detalle breve, la IP y la fecha-hora UTC:

| Comando | Evento | Tabla |
|---|---|---|
| AnularVentaCommand | `VENTA_ANULAR` (detalle «estado: Anulada») | `Ventas_Cabecera` |
| ProcesarDevolucion | `DEVOLUCION_PROCESAR` | `Devoluciones_Cabecera` |
| AperturarCajaCommand | `CAJA_APERTURAR` | `POS_Sesiones_Caja` |
| CerrarCajaCommand | `CAJA_CERRAR` | `POS_Sesiones_Caja` |
| AdjustStockCommand | `STOCK_AJUSTAR` | `Inventario_Sucursal` |

#### TC-OPS-133 — Lo que no deja evento de auditoría

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | [HU-OPS-022] «Lo que no deja evento»; [CU-OPS-22] flujo básico; [RN-AUD-006], [RN-GEN-014] |

**Pasos:** crea, cobra y actualiza una venta; crea una transferencia, un delivery, una orden magistral, una cita, un aislamiento y un acta; revisa la tabla de auditoría.
**Resultado esperado:** ninguna de esas operaciones deja evento de auditoría de seguridad; solo queda el usuario y la fecha de creación y modificación de las filas.

#### TC-OPS-134 — El valor anterior no se guarda; un flujo sin empleado no deja evento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-022] «El valor anterior no se guarda»; [CU-OPS-22] flujo básico; [RN-AUD-007] |

**Pasos:** revisa la columna `Payload_Anterior` de los eventos de `TC-OPS-132`.
**Resultado esperado:** `Payload_Anterior` queda siempre vacío y solo se guarda una descripción del hecho; un flujo sin empleado en la sesión no deja evento.

#### TC-OPS-135 — No se pueden consultar los eventos de auditoría

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-022] «No se pueden consultar los eventos»; [CU-OPS-22] excepciones; [RN-AUD-008] |

**Pasos:** busca en el Swagger local de la API (solo lectura) un endpoint que lea eventos de auditoría; prueba `GET /api/v1/Auditoria`.
**Resultado esperado:** ningún endpoint, consulta ni exportación lee los eventos (la ruta no existe); la tabla solo se lee directamente en la base de datos; el rol «Auditor» sembrado no tiene con qué leer.
**Hallazgo:** «La auditoría solo se escribe: no hay consulta de eventos» (01d hallazgo 6).

#### TC-OPS-136 — Los eventos son de solo agregar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-022] «Los eventos son de solo agregar»; [CU-OPS-22] excepciones; [RN-AUD-009] |

**Pasos:** en una base de pruebas descartable intenta modificar y eliminar una fila de `Auditoria_Eventos_Seguridad` con SQL directo.
**Resultado esperado:** la base lo permite (ningún disparador lo impide); el código nunca modifica ni elimina eventos: la inmutabilidad depende solo del código. No ejecutes esto en un ambiente compartido.

#### TC-OPS-137 — Estructura de la tabla de auditoría

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | [HU-OPS-022] «Estructura»; [CU-OPS-22] excepciones; [RN-AUD-010] |

**Pasos:** revisa la tabla y prueba insertar (base descartable) una fila sin empleado y otra con un empleado inexistente.
**Resultado esperado:** la tabla exige un empleado y rechaza referencias inexistentes; los guards de la entidad nunca llegan al cliente.

## 9. Pendientes y «No verificado»

### 9.1 Resumen de los casos

| Módulo | Casos | Alta | Media | Baja |
|---|---|---|---|---|
| Ventas (`TC-VEN`) | 209 | 91 | 91 | 27 |
| Caja (`TC-CAJ`) | 56 | 24 | 20 | 12 |
| Devoluciones (`TC-DEV`) | 59 | 26 | 27 | 6 |
| Dashboard (`TC-DAS`) | 49 | 7 | 25 | 17 |
| Solo API (`TC-OPS`) | 137 | 32 | 72 | 33 |
| **Total** | **510** | **180** | **235** | **95** |

Estado de ejecución de los 510 casos: **No ejecutado**. En esta tarea no se ejecutó la aplicación, no se llamó a la API y no se ejecutó ningún comando git.

### 9.2 Hallazgos documentados que los casos reproducen tal cual

Los casos describen el comportamiento **actual** de estos puntos (no el deseado); cada uno remite al hallazgo del documento fuente.

| Hallazgo | Casos |
|---|---|
| Casi todos los rechazos del cobro (`Venta.Caja`, `Venta.Stock`, `Venta.Lote`, `Venta.Pagos`, etc.) se ven con el genérico «Ocurrió un error al procesar la venta.» (01c2 §5.3 hallazgo 3) | TC-VEN-006, TC-VEN-021, TC-VEN-031, TC-VEN-093, TC-VEN-119 |
| Sin `POS:Leer` el terminal trata la falla como «sin caja» sin avisar | TC-CAJ-013 |
| La cobertura de seguro existe en la API pero el bloque está oculto en la pantalla | TC-VEN-140 a TC-VEN-145 |
| El cobro sin reenviar líneas (solo API) no revalida vencimiento, cuarentena, stock ni dispensaciones | TC-VEN-165, TC-VEN-080 |
| El número de operación no es único ni se valida su formato | TC-VEN-103 |
| La boleta nace `NoEnviado`; no existe envío a SUNAT; el emisor no se guarda en la boleta | TC-VEN-133, TC-VEN-135, TC-DEV-036 |
| Un efectivo esperado negativo impide cerrar la caja y solo se ve «Error al cerrar la sesión» | TC-CAJ-037, TC-DEV-027 |
| No existe vista «solo mis ventas» ni «solo mis sesiones» | TC-VEN-193, TC-CAJ-049 |
| Los Excel salen con fechas en UTC sin convertir | TC-VEN-207, TC-CAJ-055 |
| `GET Pos/{id}` y `GET Devoluciones/{id}` devuelven el objeto `Result` completo | TC-CAJ-046, TC-DEV-057 |
| El reembolso no se compara con los pagos originales; el estimado puede diferir del real | TC-DEV-026, TC-DEV-030 |
| El motivo SUNAT no se valida contra el catálogo; no hay plazo ni límite de devoluciones | TC-DEV-042, TC-DEV-051 |
| El destino «Cuarentena DIGEMID» de una devolución no repone stock ni crea el aislamiento | TC-DEV-019, TC-OPS-047 |
| La cuarentena solo la respeta la venta; no se puede liberar un lote retenido | TC-OPS-008, TC-OPS-049, TC-OPS-052, TC-OPS-053, TC-OPS-074, TC-OPS-101 |
| El acta de destrucción es solo una cabecera | TC-OPS-061 |
| La diferencia enviada y no recibida de una transferencia se pierde | TC-OPS-022, TC-OPS-024 |
| El cambio de estado de delivery no comprueba la sucursal | TC-OPS-123 |
| La anonimización de datos responde 501 y no hace nada | TC-OPS-129 |
| Los eventos de auditoría no se pueden consultar | TC-OPS-135 |
| «Vendido hoy» incluye las ventas «Devuelta» y los montos son brutos | TC-DAS-004, TC-DAS-048 |
| La ruta `/pos/nueva?continuar=` no exige `Ventas:Actualizar` | TC-VEN-156 |

### 9.3 No verificado

| Tema | Casos |
|---|---|
| Nombres exactos de campos y de parámetros de las peticiones de devoluciones, cuarentena, actas, servicios, delivery, exportación y listados; formato de serialización de los enumerados | TC-DEV-001 a TC-DEV-059 (cuerpo de `POST /api/v1/devoluciones`), TC-CAJ-054, TC-DEV-055, TC-OPS-040 a TC-OPS-137 |
| Redondeo de la base de datos con más de 2 decimales (cantidades, montos) | TC-VEN-030, TC-VEN-096, TC-DEV-022, TC-DEV-028, TC-OPS-103 |
| Comportamiento bajo concurrencia (versión de fila, índices únicos) | TC-VEN-120, TC-VEN-134, TC-VEN-170, TC-VEN-179, TC-CAJ-006, TC-CAJ-038, TC-DEV-007, TC-OPS-019, TC-OPS-029, TC-OPS-076, TC-OPS-082 |
| Códigos HTTP y `code` exactos de algunos rechazos del cobro (receta, número de operación, estados `Anulada`/`Devuelta` al crear, base de datos) | TC-VEN-062, TC-VEN-070, TC-VEN-099, TC-VEN-102, TC-VEN-121, TC-VEN-123 |
| Cómo preparar una cuenta cuyo token no traiga sucursal o empleado, o un usuario con alcance vacío | TC-VEN-008, TC-CAJ-007, TC-CAJ-041, TC-DAS-012, TC-OPS-002, TC-OPS-046 |
| Nombre de la clave del borrador en `localStorage` y de la tabla de la bandeja de salida de eventos | TC-VEN-046, TC-VEN-117 |
| Separador decimal del Inicio según el idioma; contenido de los Excel (no se descargaron) | TC-DAS-001, TC-VEN-207, TC-CAJ-055 |
| Forma exacta del cuerpo de un 403 y del `ProblemDetails` del 501 | TC-OPS-012, TC-OPS-129 |
| Si agendar, listar y eliminar citas funciona con una base creada solo con scripts (columnas faltantes en `Servicios_Agenda`) | TC-OPS-090 a TC-OPS-093, TC-OPS-105, TC-OPS-107, TC-OPS-111 |
| Resultado exacto de una limpieza de HTML en el sustento; ruta y método no existentes (404 o 405) | TC-DEV-046, TC-DEV-059, TC-OPS-062, TC-OPS-065 |

### 9.4 En curso (sin commitear)

Ninguno. Las historias (12c, 2026-10-26) y los casos de uso (13c, 2026-10-27) declaran `git status --short` limpio en ambos repositorios sobre `43f5529` y `eeffbb6`. En esta tarea no se ejecutó git, por lo que no se comprobó si el árbol cambió después; ninguna afirmación de este documento se apoya en un archivo con cambios sin commitear que yo haya leído.

### 9.5 Observaciones sobre las fuentes y sobre los datos de prueba

- El manual (capítulo 3, sección 5.3) rotula el permiso del dashboard de inventario «Inventario › Leer», mientras las historias y las reglas lo escriben `Inventarios:Leer`; en la sección 2.2 se copió el rótulo del manual y en los casos del Inicio se usa `Inventarios:Leer`. Comprueba cuál es el nombre vigente en la matriz de roles ([05b](../../ola-1-fundamentos/05-seguridad/05b-matriz-de-permisos.md)).
- Los casos de uso no numeran sus excepciones; los casos citan «excepción, paso N» más la situación.
- Varios casos necesitan datos que el entorno por defecto no trae: una empresa de prueba sin RUC y sin serie de boletas, una empresa en modo flexible (`ExigirRegistroReceta` en falso, solo por base de datos), una sucursal sin serie de nota de crédito, cuentas con token sin sucursal o sin empleado, más de 10 000 ventas o sesiones para los casos de límite de exportación, y ventas anteriores a los scripts `0016` y `0028`. Si no puedes prepararlos, el caso queda «Bloqueado por datos» en tu ejecución, no «Fallido».
- Los casos de concurrencia se ejecutan con dos clientes HTTP en paralelo; su resultado es probabilístico y debe repetirse varias veces antes de darlo por pasado.

## 10. Tabla de cobertura

La cobertura se comprobó con un script de solo lectura (Node) sobre este documento y sus fuentes: parte el documento por `#### TC-`, lee el campo «Cubre» de cada caso, compara los títulos de escenario entre « » con los de las historias (`Escenario:` y `Esquema del escenario:` dentro de cada `### HU-…`), y comprueba que todos los `HU-…`, `CU-…` y `RN-…` citados existen en sus documentos.

### 10.1 Escenarios de las historias → casos

Los números siguen el orden en que aparecen las líneas `Escenario:` y `Esquema del escenario:` dentro de cada historia de [12c](../../ola-2-funcional/12-historias-de-usuario/12c-punto-de-venta-y-caja.md); el caso se escribe sin el prefijo `TC-` (por ejemplo `VEN-001` es `TC-VEN-001`). Cuando un caso cubre varios escenarios, su ID se repite. **566 de 566 escenarios cubiertos.**

| Historia | Escenarios (orden en la historia → caso) | Casos |
|---|---|---|
| HU-VEN-001 | 1→VEN-001; 2→VEN-002; 3→VEN-003; 4→VEN-004; 5→VEN-005; 6→VEN-006; 7→VEN-007; 8→VEN-008; 9→VEN-009 | 9 |
| HU-VEN-002 | 1→VEN-010; 2→VEN-011; 3→VEN-012; 4→VEN-013; 5→VEN-014; 6→VEN-015; 7→VEN-016; 8→VEN-017; 9→VEN-018; 10→VEN-019; 11→VEN-020; 12→VEN-020; 13→VEN-021 | 12 |
| HU-VEN-003 | 1→VEN-022; 2→VEN-023; 3→VEN-024; 4→VEN-025; 5→VEN-026; 6→VEN-027; 7→VEN-028; 8→VEN-029; 9→VEN-030; 10→VEN-031; 11→VEN-032; 12→VEN-033; 13→VEN-034 | 13 |
| HU-VEN-004 | 1→VEN-035; 2→VEN-036; 3→VEN-037; 4→VEN-038; 5→VEN-039; 6→VEN-040; 7→VEN-041; 8→VEN-041 | 7 |
| HU-VEN-005 | 1→VEN-042; 2→VEN-043; 3→VEN-043; 4→VEN-043; 5→VEN-044; 6→VEN-044; 7→VEN-044 | 3 |
| HU-VEN-006 | 1→VEN-045; 2→VEN-046; 3→VEN-045; 4→VEN-046; 5→VEN-046; 6→VEN-047 | 3 |
| HU-VEN-007 | 1→VEN-048; 2→VEN-049; 3→VEN-049; 4→VEN-049; 5→VEN-050; 6→VEN-050; 7→VEN-048; 8→VEN-051; 9→VEN-052; 10→VEN-053 | 6 |
| HU-VEN-008 | 1→VEN-054; 2→VEN-055; 3→VEN-056; 4→VEN-057; 5→VEN-058; 6→VEN-059; 7→VEN-060; 8→VEN-061; 9→VEN-062; 10→VEN-062; 11→VEN-063; 12→VEN-064 | 11 |
| HU-VEN-009 | 1→VEN-065; 2→VEN-066; 3→VEN-067; 4→VEN-067; 5→VEN-068; 6→VEN-069; 7→VEN-069; 8→VEN-070; 9→VEN-071 | 7 |
| HU-VEN-010 | 1→VEN-072; 2→VEN-073; 3→VEN-074; 4→VEN-075; 5→VEN-076; 6→VEN-077; 7→VEN-078; 8→VEN-079; 9→VEN-080 | 9 |
| HU-VEN-011 | 1→VEN-081; 2→VEN-082; 3→VEN-083; 4→VEN-084; 5→VEN-085; 6→VEN-085; 7→VEN-086; 8→VEN-086; 9→VEN-087 | 7 |
| HU-VEN-012 | 1→VEN-088; 2→VEN-089; 3→VEN-090; 4→VEN-091; 5→VEN-092; 6→VEN-093; 7→VEN-094; 8→VEN-095; 9→VEN-096 | 9 |
| HU-VEN-013 | 1→VEN-097; 2→VEN-098; 3→VEN-098; 4→VEN-099; 5→VEN-100; 6→VEN-101; 7→VEN-102; 8→VEN-099; 9→VEN-103 | 7 |
| HU-VEN-014 | 1→VEN-104; 2→VEN-105; 3→VEN-106; 4→VEN-107; 5→VEN-108; 6→VEN-108; 7→VEN-109; 8→VEN-110; 9→VEN-111 | 8 |
| HU-VEN-015 | 1→VEN-112; 2→VEN-113; 3→VEN-114; 4→VEN-115; 5→VEN-116; 6→VEN-117; 7→VEN-118; 8→VEN-119; 9→VEN-120; 10→VEN-121; 11→VEN-122; 12→VEN-123 | 12 |
| HU-VEN-016 | 1→VEN-124; 2→VEN-125; 3→VEN-125; 4→VEN-126; 5→VEN-127 | 4 |
| HU-VEN-017 | 1→VEN-128; 2→VEN-129; 3→VEN-129; 4→VEN-130; 5→VEN-130; 6→VEN-131; 7→VEN-131; 8→VEN-132; 9→VEN-132; 10→VEN-133; 11→VEN-134; 12→VEN-135; 13→VEN-135 | 8 |
| HU-VEN-018 | 1→VEN-136; 2→VEN-137; 3→VEN-137; 4→VEN-137; 5→VEN-137; 6→VEN-136; 7→VEN-138; 8→VEN-139; 9→VEN-138 | 4 |
| HU-VEN-019 | 1→VEN-140; 2→VEN-141; 3→VEN-142; 4→VEN-143; 5→VEN-144; 6→VEN-144; 7→VEN-145; 8→VEN-145; 9→VEN-145 | 6 |
| HU-VEN-020 | 1→VEN-146; 2→VEN-147; 3→VEN-148; 4→VEN-149; 5→VEN-150; 6→VEN-151; 7→VEN-152 | 7 |
| HU-VEN-021 | 1→VEN-153; 2→VEN-154; 3→VEN-155; 4→VEN-156; 5→VEN-157; 6→VEN-158; 7→VEN-159; 8→VEN-053; 9→VEN-160; 10→VEN-160; 11→VEN-161 | 10 |
| HU-VEN-022 | 1→VEN-162; 2→VEN-163; 3→VEN-164; 4→VEN-165; 5→VEN-166; 6→VEN-167; 7→VEN-168; 8→VEN-169; 9→VEN-170; 10→VEN-171 | 10 |
| HU-VEN-023 | 1→VEN-172; 2→VEN-173; 3→VEN-174; 4→VEN-175; 5→VEN-176; 6→VEN-177; 7→VEN-178; 8→VEN-179; 9→VEN-180 | 9 |
| HU-VEN-024 | 1→VEN-181; 2→VEN-182; 3→VEN-182 | 2 |
| HU-VEN-025 | 1→VEN-183; 2→VEN-184; 3→VEN-185; 4→VEN-185; 5→VEN-186; 6→VEN-187; 7→VEN-188; 8→VEN-189 | 7 |
| HU-VEN-026 | 1→VEN-190; 2→VEN-191; 3→VEN-192; 4→VEN-193; 5→VEN-194; 6→VEN-195; 7→VEN-196; 8→VEN-197; 9→VEN-198; 10→VEN-199; 11→VEN-199 | 10 |
| HU-VEN-027 | 1→VEN-200; 2→VEN-201; 3→VEN-202; 4→VEN-066; 5→VEN-202; 6→VEN-203; 7→VEN-204; 8→VEN-205 | 7 |
| HU-VEN-028 | 1→VEN-206; 2→VEN-207; 3→VEN-208; 4→VEN-209; 5→VEN-206 | 4 |
| HU-CAJ-001 | 1→CAJ-001; 2→CAJ-002; 3→CAJ-003; 4→CAJ-004; 5→CAJ-005; 6→CAJ-006; 7→CAJ-007; 8→CAJ-008; 9→CAJ-009; 10→CAJ-010 | 10 |
| HU-CAJ-002 | 1→CAJ-011; 2→CAJ-012; 3→CAJ-013; 4→CAJ-014; 5→CAJ-015 | 5 |
| HU-CAJ-003 | 1→CAJ-016; 2→CAJ-017; 3→CAJ-018; 4→CAJ-019; 5→CAJ-020; 6→CAJ-021; 7→CAJ-022; 8→CAJ-023; 9→CAJ-022; 10→CAJ-024; 11→CAJ-025; 12→CAJ-026; 13→CAJ-027 | 12 |
| HU-CAJ-004 | 1→CAJ-028; 2→CAJ-028; 3→CAJ-029; 4→CAJ-030; 5→CAJ-031; 6→CAJ-032; 7→CAJ-033; 8→CAJ-034; 9→CAJ-035; 10→CAJ-036 | 9 |
| HU-CAJ-005 | 1→CAJ-037; 2→CAJ-038; 3→CAJ-039; 4→CAJ-040; 5→CAJ-041 | 5 |
| HU-CAJ-006 | 1→CAJ-042; 2→CAJ-043; 3→CAJ-044; 4→CAJ-045; 5→CAJ-046 | 5 |
| HU-CAJ-007 | 1→CAJ-047; 2→CAJ-048; 3→CAJ-049; 4→CAJ-050; 5→CAJ-051; 6→CAJ-051; 7→CAJ-051; 8→CAJ-052 | 6 |
| HU-CAJ-008 | 1→CAJ-053; 2→CAJ-054; 3→CAJ-055; 4→CAJ-056; 5→CAJ-056 | 4 |
| HU-DEV-001 | 1→DEV-001; 2→DEV-002; 3→DEV-003; 4→DEV-004; 5→DEV-005; 6→DEV-006; 7→DEV-007; 8→DEV-008; 9→DEV-009; 10→DEV-010; 11→DEV-011; 12→DEV-012 | 12 |
| HU-DEV-002 | 1→DEV-013; 2→DEV-014; 3→DEV-015; 4→DEV-016; 5→DEV-017; 6→DEV-018; 7→DEV-019; 8→DEV-020; 9→DEV-021; 10→DEV-022 | 10 |
| HU-DEV-003 | 1→DEV-023; 2→DEV-024; 3→DEV-025; 4→DEV-026; 5→DEV-027; 6→DEV-028; 7→DEV-029; 8→DEV-030; 9→DEV-031; 10→DEV-032 | 10 |
| HU-DEV-004 | 1→DEV-033; 2→DEV-034; 3→DEV-035; 4→DEV-036; 5→DEV-037; 6→DEV-038; 7→DEV-039; 8→DEV-040; 9→DEV-041 | 9 |
| HU-DEV-005 | 1→DEV-042; 2→DEV-042; 3→DEV-043; 4→DEV-044; 5→DEV-044; 6→DEV-045; 7→DEV-046 | 5 |
| HU-DEV-006 | 1→DEV-047; 2→DEV-048; 3→DEV-049; 4→DEV-050; 5→DEV-051 | 5 |
| HU-DEV-007 | 1→DEV-052; 2→DEV-052; 3→DEV-053; 4→DEV-054 | 3 |
| HU-DEV-008 | 1→DEV-055; 2→DEV-056; 3→DEV-057; 4→DEV-058; 5→DEV-059 | 5 |
| HU-DAS-001 | 1→DAS-001; 2→DAS-002; 3→DAS-003; 4→DAS-004; 5→DAS-005; 6→DAS-006; 7→DAS-007 | 7 |
| HU-DAS-002 | 1→DAS-008; 2→DAS-009; 3→DAS-010; 4→DAS-011; 5→DAS-012 | 5 |
| HU-DAS-003 | 1→DAS-013; 2→DAS-014; 3→DAS-015; 4→DAS-016 | 4 |
| HU-DAS-004 | 1→DAS-017; 2→DAS-018; 3→DAS-019; 4→DAS-020; 5→DAS-021 | 5 |
| HU-DAS-005 | 1→DAS-022; 2→DAS-023; 3→DAS-024; 4→DAS-025; 5→DAS-026; 6→DAS-027 | 6 |
| HU-DAS-006 | 1→DAS-028; 2→DAS-029; 3→DAS-030; 4→DAS-031 | 4 |
| HU-DAS-007 | 1→DAS-032; 2→DAS-033; 3→DAS-034 | 3 |
| HU-DAS-008 | 1→DAS-035; 2→DAS-036; 3→DAS-037; 4→DAS-038; 5→DAS-039; 6→DAS-040; 7→DAS-041; 8→DAS-042 | 8 |
| HU-DAS-009 | 1→DAS-043; 2→DAS-043; 3→DAS-044; 4→DAS-045; 5→DAS-045 | 3 |
| HU-DAS-010 | 1→DAS-046; 2→DAS-047; 3→DAS-048; 4→DAS-049 | 4 |
| HU-OPS-001 | 1→OPS-001; 2→OPS-002; 3→OPS-003; 4→OPS-004; 5→OPS-005; 6→OPS-006; 7→OPS-007; 8→OPS-008; 9→OPS-009; 10→OPS-010; 11→OPS-011; 12→OPS-012 | 12 |
| HU-OPS-002 | 1→OPS-013; 2→OPS-014; 3→OPS-015; 4→OPS-016; 5→OPS-017; 6→OPS-018; 7→OPS-019 | 7 |
| HU-OPS-003 | 1→OPS-020; 2→OPS-021; 3→OPS-022; 4→OPS-023; 5→OPS-024; 6→OPS-025; 7→OPS-026; 8→OPS-027; 9→OPS-028; 10→OPS-029 | 10 |
| HU-OPS-004 | 1→OPS-030; 2→OPS-031; 3→OPS-032; 4→OPS-033; 5→OPS-034 | 5 |
| HU-OPS-005 | 1→OPS-035; 2→OPS-036; 3→OPS-037; 4→OPS-038; 5→OPS-039 | 5 |
| HU-OPS-006 | 1→OPS-040; 2→OPS-041; 3→OPS-042; 4→OPS-043; 5→OPS-044; 6→OPS-045; 7→OPS-046; 8→OPS-047; 9→OPS-048 | 9 |
| HU-OPS-007 | 1→OPS-049; 2→OPS-050; 3→OPS-051; 4→OPS-052; 5→OPS-053 | 5 |
| HU-OPS-008 | 1→OPS-054; 2→OPS-055; 3→OPS-056 | 3 |
| HU-OPS-009 | 1→OPS-057; 2→OPS-058; 3→OPS-059; 4→OPS-060; 5→OPS-061; 6→OPS-062; 7→OPS-063 | 7 |
| HU-OPS-010 | 1→OPS-064; 2→OPS-065 | 2 |
| HU-OPS-011 | 1→OPS-066; 2→OPS-067; 3→OPS-068; 4→OPS-069; 5→OPS-070; 6→OPS-071; 7→OPS-072; 8→OPS-073; 9→OPS-074; 10→OPS-075; 11→OPS-076; 12→OPS-077 | 12 |
| HU-OPS-012 | 1→OPS-078; 2→OPS-079; 3→OPS-080; 4→OPS-081; 5→OPS-082; 6→OPS-083; 7→OPS-084; 8→OPS-085 | 8 |
| HU-OPS-013 | 1→OPS-086; 2→OPS-087; 3→OPS-088; 4→OPS-089 | 4 |
| HU-OPS-014 | 1→OPS-090; 2→OPS-091; 3→OPS-092; 4→OPS-093 | 4 |
| HU-OPS-015 | 1→OPS-094; 2→OPS-095; 3→OPS-096; 4→OPS-097; 5→OPS-098; 6→OPS-099; 7→OPS-100; 8→OPS-101; 9→OPS-102; 10→OPS-103; 11→OPS-104 | 11 |
| HU-OPS-016 | 1→OPS-105; 2→OPS-106; 3→OPS-107 | 3 |
| HU-OPS-017 | 1→OPS-108; 2→OPS-109; 3→OPS-110; 4→OPS-111; 5→OPS-110 | 4 |
| HU-OPS-018 | 1→OPS-112; 2→OPS-113; 3→OPS-114; 4→OPS-115; 5→OPS-116; 6→OPS-117; 7→OPS-118 | 7 |
| HU-OPS-019 | 1→OPS-119; 2→OPS-120; 3→OPS-121; 4→OPS-122; 5→OPS-123; 6→OPS-124 | 6 |
| HU-OPS-020 | 1→OPS-125; 2→OPS-126; 3→OPS-127; 4→OPS-128 | 4 |
| HU-OPS-021 | 1→OPS-129; 2→OPS-130; 3→OPS-131; 4→OPS-131 | 3 |
| HU-OPS-022 | 1→OPS-132; 2→OPS-132; 3→OPS-133; 4→OPS-134; 5→OPS-135; 6→OPS-136; 7→OPS-137 | 6 |

Las 76 historias tienen al menos un caso (la columna «Casos» cuenta los casos distintos que citan la historia).

### 10.2 Casos de uso → casos

Los 76 casos de uso tienen al menos un caso (los rangos usan el prefijo del módulo: `TC-CAJ-001 a 010` es de `TC-CAJ-001` a `TC-CAJ-010`).

| Caso de uso | Casos |
|---|---|
| CU-VEN-01 | TC-VEN-001 a 009 |
| CU-VEN-02 | TC-VEN-010 a 021 |
| CU-VEN-03 | TC-VEN-022 a 034 |
| CU-VEN-04 | TC-VEN-035 a 041 |
| CU-VEN-05 | TC-VEN-042 a 044 |
| CU-VEN-06 | TC-VEN-045 a 047 |
| CU-VEN-07 | TC-VEN-048 a 053 |
| CU-VEN-08 | TC-VEN-054 a 064 |
| CU-VEN-09 | TC-VEN-065 a 071 |
| CU-VEN-10 | TC-VEN-072 a 080 |
| CU-VEN-11 | TC-VEN-081 a 087 |
| CU-VEN-12 | TC-VEN-088 a 096 |
| CU-VEN-13 | TC-VEN-097 a 103 |
| CU-VEN-14 | TC-VEN-104 a 111 |
| CU-VEN-15 | TC-VEN-112 a 123 |
| CU-VEN-16 | TC-VEN-124 a 127 |
| CU-VEN-17 | TC-VEN-128 a 135 |
| CU-VEN-18 | TC-VEN-136 a 139 |
| CU-VEN-19 | TC-VEN-140 a 145 |
| CU-VEN-20 | TC-VEN-146 a 152 |
| CU-VEN-21 | TC-VEN-053, TC-VEN-153 a 161 |
| CU-VEN-22 | TC-VEN-162 a 171 |
| CU-VEN-23 | TC-VEN-172 a 180 |
| CU-VEN-24 | TC-VEN-181, TC-VEN-182 |
| CU-VEN-25 | TC-VEN-183 a 189 |
| CU-VEN-26 | TC-VEN-153, TC-VEN-190 a 199 |
| CU-VEN-27 | TC-VEN-066, TC-VEN-200 a 205 |
| CU-VEN-28 | TC-VEN-206 a 209 |
| CU-CAJ-01 | TC-CAJ-001 a 010 |
| CU-CAJ-02 | TC-CAJ-011 a 015 |
| CU-CAJ-03 | TC-CAJ-016 a 027 |
| CU-CAJ-04 | TC-CAJ-028 a 036 |
| CU-CAJ-05 | TC-CAJ-037 a 041 |
| CU-CAJ-06 | TC-CAJ-042 a 046 |
| CU-CAJ-07 | TC-CAJ-047 a 052 |
| CU-CAJ-08 | TC-CAJ-053 a 056 |
| CU-DEV-01 | TC-DEV-001 a 012 |
| CU-DEV-02 | TC-DEV-013 a 022 |
| CU-DEV-03 | TC-DEV-023 a 032 |
| CU-DEV-04 | TC-DEV-033 a 041 |
| CU-DEV-05 | TC-DEV-042 a 046 |
| CU-DEV-06 | TC-DEV-047 a 051 |
| CU-DEV-07 | TC-DEV-052 a 054 |
| CU-DEV-08 | TC-DEV-055 a 059 |
| CU-DAS-01 | TC-DAS-001 a 007 |
| CU-DAS-02 | TC-DAS-008 a 012 |
| CU-DAS-03 | TC-DAS-013 a 016 |
| CU-DAS-04 | TC-DAS-017 a 021 |
| CU-DAS-05 | TC-DAS-022 a 027 |
| CU-DAS-06 | TC-DAS-028 a 031 |
| CU-DAS-07 | TC-DAS-032 a 034 |
| CU-DAS-08 | TC-DAS-035 a 042 |
| CU-DAS-09 | TC-DAS-043 a 045 |
| CU-DAS-10 | TC-DAS-046 a 049 |
| CU-OPS-01 | TC-OPS-001 a 012 |
| CU-OPS-02 | TC-OPS-013 a 019 |
| CU-OPS-03 | TC-OPS-020 a 029 |
| CU-OPS-04 | TC-OPS-030 a 034 |
| CU-OPS-05 | TC-OPS-035 a 039 |
| CU-OPS-06 | TC-OPS-040 a 048 |
| CU-OPS-07 | TC-OPS-049 a 053 |
| CU-OPS-08 | TC-OPS-054 a 056 |
| CU-OPS-09 | TC-OPS-057 a 063 |
| CU-OPS-10 | TC-OPS-064, TC-OPS-065 |
| CU-OPS-11 | TC-OPS-066 a 077 |
| CU-OPS-12 | TC-OPS-078 a 085 |
| CU-OPS-13 | TC-OPS-086 a 089 |
| CU-OPS-14 | TC-OPS-090 a 093 |
| CU-OPS-15 | TC-OPS-094 a 104 |
| CU-OPS-16 | TC-OPS-105 a 107 |
| CU-OPS-17 | TC-OPS-108 a 111 |
| CU-OPS-18 | TC-OPS-112 a 118 |
| CU-OPS-19 | TC-OPS-119 a 124 |
| CU-OPS-20 | TC-OPS-125 a 128 |
| CU-OPS-21 | TC-OPS-129 a 131 |
| CU-OPS-22 | TC-OPS-132 a 137 |

### 10.3 Reglas de negocio → casos

Reglas de los catálogos [01c2](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) (`RN-VEN`, `RN-CMP`), [01c3](../../ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) (`RN-CAJ`, `RN-DEV`, `RN-DAS`) y [01d](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) (`RN-TRA`, `RN-CUA`, `RN-MAG`, `RN-SRV`, `RN-DEL`, `RN-AUD`), más las transversales y de otras partes que mis casos citan. El caso se escribe sin el prefijo `TC-`. Las 320 reglas con comportamiento comprobable tienen caso; las cuatro restantes figuran en 10.4.

| Regla | Casos |
|---|---|
| RN-AUD-001 | OPS-129 |
| RN-AUD-002 | OPS-129 |
| RN-AUD-003 | OPS-130 |
| RN-AUD-004 | OPS-131 |
| RN-AUD-005 | OPS-132 |
| RN-AUD-006 | OPS-132, OPS-133 |
| RN-AUD-007 | OPS-134 |
| RN-AUD-008 | OPS-135 |
| RN-AUD-009 | OPS-136 |
| RN-AUD-010 | OPS-137 |
| RN-CAJ-001 | CAJ-001, CAJ-008, CAJ-010, CAJ-016 |
| RN-CAJ-002 | CAJ-001, CAJ-002, CAJ-007, CAJ-011, CAJ-041 |
| RN-CAJ-003 | CAJ-001, CAJ-003, CAJ-004, CAJ-011, CAJ-012 |
| RN-CAJ-004 | CAJ-005, CAJ-013 |
| RN-CAJ-005 | CAJ-001, CAJ-011 |
| RN-CAJ-007 | CAJ-016, CAJ-022, CAJ-023 |
| RN-CAJ-008 | CAJ-039 |
| RN-CAJ-009 | CAJ-021 |
| RN-CAJ-010 | CAJ-019, CAJ-024, CAJ-038, CAJ-040 |
| RN-CAJ-011 | CAJ-028, CAJ-029, DAS-017 |
| RN-CAJ-012 | VEN-175, VEN-176, CAJ-028, CAJ-029, CAJ-032, CAJ-033, CAJ-035, DAS-019 |
| RN-CAJ-013 | CAJ-028, CAJ-030, CAJ-031, DAS-017 |
| RN-CAJ-014 | CAJ-029, CAJ-034, CAJ-035, DEV-023, DEV-025, DAS-019 |
| RN-CAJ-015 | CAJ-016 a 018, CAJ-042, CAJ-044 |
| RN-CAJ-016 | CAJ-037 |
| RN-CAJ-017 | CAJ-026, CAJ-033 |
| RN-CAJ-018 | CAJ-027 |
| RN-CAJ-019 | CAJ-019 |
| RN-CAJ-020 | CAJ-045, CAJ-049 |
| RN-CAJ-021 | CAJ-047, CAJ-048, CAJ-052 |
| RN-CAJ-022 | CAJ-047 |
| RN-CAJ-023 | CAJ-042, CAJ-043, CAJ-045, CAJ-046 |
| RN-CAJ-024 | CAJ-053, CAJ-054 |
| RN-CAJ-025 | CAJ-009 |
| RN-CAJ-026 | CAJ-006, CAJ-009 |
| RN-CAJ-027 | CAJ-009, CAJ-036 |
| RN-CMP-001 | VEN-128, VEN-131, VEN-132 |
| RN-CMP-002 | VEN-124 |
| RN-CMP-003 | VEN-125 |
| RN-CMP-004 | VEN-128, VEN-134 |
| RN-CMP-005 | VEN-048, VEN-130, VEN-131 |
| RN-CMP-006 | VEN-129 |
| RN-CMP-007 | VEN-135 |
| RN-CMP-008 | VEN-135 |
| RN-CMP-009 | VEN-134 |
| RN-CMP-010 | VEN-135 |
| RN-CMP-011 | VEN-133, VEN-138 |
| RN-CMP-012 | VEN-135, VEN-138, VEN-139, VEN-173, VEN-174, VEN-203 |
| RN-CMP-013 | VEN-200, DEV-036 |
| RN-CMP-014 | VEN-136 a 138 |
| RN-CMP-015 | DEV-040 |
| RN-CUA-001 | OPS-040 |
| RN-CUA-002 | OPS-040, OPS-046 |
| RN-CUA-003 | OPS-040 |
| RN-CUA-004 | OPS-041 |
| RN-CUA-005 | OPS-041 |
| RN-CUA-006 | OPS-042, OPS-045 |
| RN-CUA-007 | DAS-029, OPS-049 |
| RN-CUA-008 | OPS-050, OPS-051 |
| RN-CUA-009 | OPS-052, OPS-053 |
| RN-CUA-010 | OPS-043 |
| RN-CUA-011 | DEV-019, OPS-044, OPS-047 |
| RN-CUA-012 | OPS-054 a 056 |
| RN-CUA-013 | OPS-057 |
| RN-CUA-014 | OPS-057 |
| RN-CUA-015 | OPS-058 |
| RN-CUA-016 | OPS-059 |
| RN-CUA-017 | OPS-060 |
| RN-CUA-018 | OPS-061 |
| RN-CUA-019 | OPS-064, OPS-065 |
| RN-CUA-020 | OPS-048 |
| RN-CUA-021 | OPS-058, OPS-063 |
| RN-CUA-022 | OPS-062 |
| RN-CUA-023 | OPS-060 |
| RN-DAS-001 | DAS-031, DAS-034, DAS-036, DAS-037 |
| RN-DAS-002 | DAS-001, DAS-007, DAS-043 |
| RN-DAS-003 | DAS-010, DAS-012, DAS-033, DAS-038 a 040 |
| RN-DAS-004 | DAS-001, DAS-007 |
| RN-DAS-005 | DAS-004, DAS-048 |
| RN-DAS-006 | DAS-001, DAS-002 |
| RN-DAS-007 | DAS-003 |
| RN-DAS-008 | DAS-005 |
| RN-DAS-009 | DAS-006 |
| RN-DAS-010 | DAS-008, DAS-009 |
| RN-DAS-011 | DAS-010, DAS-011 |
| RN-DAS-012 | DAS-013 |
| RN-DAS-013 | DAS-014, DAS-016 |
| RN-DAS-014 | DAS-027, DAS-046, DAS-047 |
| RN-DAS-015 | DAS-017 a 019 |
| RN-DAS-016 | DAS-020, DAS-021, DAS-049 |
| RN-DAS-017 | DAS-022, DAS-024 |
| RN-DAS-018 | DAS-022 |
| RN-DAS-019 | DAS-023 |
| RN-DAS-020 | DAS-025 |
| RN-DAS-021 | DAS-026 |
| RN-DAS-022 | DAS-028 a 030 |
| RN-DAS-023 | DEV-054, DAS-032, DAS-033 |
| RN-DAS-024 | DAS-044 |
| RN-DEL-001 | OPS-112 |
| RN-DEL-002 | OPS-112 |
| RN-DEL-003 | OPS-112 |
| RN-DEL-004 | OPS-113 |
| RN-DEL-005 | OPS-114 |
| RN-DEL-006 | OPS-115 |
| RN-DEL-007 | OPS-116 |
| RN-DEL-008 | OPS-116 |
| RN-DEL-009 | OPS-119, OPS-122, OPS-123 |
| RN-DEL-010 | OPS-120 |
| RN-DEL-011 | OPS-121, OPS-124 |
| RN-DEL-012 | OPS-125 |
| RN-DEL-013 | OPS-126 |
| RN-DEL-014 | OPS-127 |
| RN-DEL-015 | OPS-128 |
| RN-DEL-016 | OPS-117 |
| RN-DEL-017 | OPS-117 |
| RN-DEL-018 | OPS-118 |
| RN-DEV-001 | DEV-001, DEV-009 |
| RN-DEV-002 | DEV-001, DEV-003, DEV-047 |
| RN-DEV-003 | DEV-047 |
| RN-DEV-004 | DEV-047, DEV-050 |
| RN-DEV-005 | DEV-032, DEV-047, DEV-048 |
| RN-DEV-006 | DEV-037, DEV-047, DEV-049 |
| RN-DEV-007 | DEV-038, DEV-047 |
| RN-DEV-008 | DEV-009, DEV-047, DEV-049, DEV-051 |
| RN-DEV-009 | DEV-001, DEV-013, DEV-015 |
| RN-DEV-010 | DEV-001, DEV-015 |
| RN-DEV-011 | DEV-021 |
| RN-DEV-012 | DEV-013, DEV-014 |
| RN-DEV-013 | DEV-020 |
| RN-DEV-014 | DEV-042, DEV-045 |
| RN-DEV-015 | DEV-043, DEV-044 |
| RN-DEV-016 | DEV-023, DEV-028 a 030 |
| RN-DEV-017 | DEV-023, DEV-029 |
| RN-DEV-018 | CAJ-034, DEV-024 a 026 |
| RN-DEV-019 | CAJ-034, DEV-023, DEV-025, DEV-027 |
| RN-DEV-020 | DEV-001, DEV-033, DEV-039 |
| RN-DEV-021 | DEV-029, DEV-034 |
| RN-DEV-022 | DEV-035, DEV-036 |
| RN-DEV-023 | DEV-017, DEV-018 |
| RN-DEV-024 | DEV-019, OPS-047 |
| RN-DEV-025 | DEV-001, DEV-002 |
| RN-DEV-026 | VEN-078, DEV-004 |
| RN-DEV-027 | DEV-003 |
| RN-DEV-028 | DEV-005 a 007, DEV-039 |
| RN-DEV-029 | DEV-008, DEV-010 |
| RN-DEV-030 | DEV-055, DEV-056 |
| RN-DEV-031 | DEV-057, DEV-058 |
| RN-DEV-032 | DEV-052, DEV-053, DEV-059 |
| RN-DEV-033 | DEV-033, DEV-041, DEV-045 |
| RN-DEV-034 | DEV-041 |
| RN-DEV-035 | DEV-031, DEV-041 |
| RN-DEV-036 | DEV-022, DEV-031, DEV-041 |
| RN-DEV-037 | DEV-040 |
| RN-GEN-007 | VEN-117 |
| RN-GEN-008 | VEN-191, CAJ-048, OPS-055, OPS-087 |
| RN-GEN-009 | VEN-191, VEN-197, CAJ-050, OPS-054 |
| RN-GEN-010 | DEV-046 |
| RN-GEN-011 | VEN-095 |
| RN-GEN-020 | OPS-012 |
| RN-GEN-023 | CAJ-049, DAS-040, OPS-054, OPS-086 |
| RN-GEN-024 | VEN-008, OPS-002 |
| RN-GEN-035 | VEN-120 |
| RN-GEN-041 | OPS-036 |
| RN-LOT-006 | DAS-022, OPS-079 |
| RN-MAG-001 | OPS-066 |
| RN-MAG-002 | OPS-066 |
| RN-MAG-003 | OPS-066 |
| RN-MAG-004 | OPS-066 |
| RN-MAG-005 | OPS-067 |
| RN-MAG-006 | OPS-068 |
| RN-MAG-007 | OPS-069 |
| RN-MAG-008 | OPS-070 |
| RN-MAG-009 | OPS-071 |
| RN-MAG-010 | OPS-072, OPS-073 |
| RN-MAG-011 | OPS-074 |
| RN-MAG-012 | OPS-078, OPS-083 |
| RN-MAG-013 | OPS-078, OPS-081, OPS-085 |
| RN-MAG-014 | OPS-079 |
| RN-MAG-015 | OPS-080 |
| RN-MAG-016 | OPS-082 |
| RN-MAG-017 | OPS-084 |
| RN-MAG-018 | OPS-086, OPS-087 |
| RN-MAG-019 | OPS-088, OPS-089 |
| RN-MAG-020 | OPS-075 |
| RN-MAG-021 | OPS-076 |
| RN-MAG-022 | OPS-077 |
| RN-MAG-023 | OPS-077 |
| RN-REC-031 | VEN-072 |
| RN-SRV-001 | OPS-090 |
| RN-SRV-002 | OPS-090 |
| RN-SRV-003 | OPS-091 |
| RN-SRV-004 | OPS-091 |
| RN-SRV-005 | OPS-092 |
| RN-SRV-006 | OPS-105 |
| RN-SRV-007 | OPS-094 |
| RN-SRV-008 | OPS-094 |
| RN-SRV-009 | OPS-095 |
| RN-SRV-010 | OPS-094, OPS-096 |
| RN-SRV-011 | OPS-097 |
| RN-SRV-012 | OPS-098 |
| RN-SRV-013 | OPS-099, OPS-100, OPS-103 |
| RN-SRV-014 | OPS-106 |
| RN-SRV-015 | OPS-101 |
| RN-SRV-016 | OPS-107 |
| RN-SRV-017 | OPS-108, OPS-109 |
| RN-SRV-018 | OPS-110, OPS-111 |
| RN-SRV-019 | OPS-090, OPS-093 |
| RN-SRV-020 | OPS-102, OPS-104 |
| RN-SRV-021 | OPS-092, OPS-111 |
| RN-TRA-001 | OPS-001, OPS-002 |
| RN-TRA-002 | OPS-001, OPS-002 |
| RN-TRA-003 | OPS-001 |
| RN-TRA-004 | OPS-003 |
| RN-TRA-005 | OPS-004 |
| RN-TRA-006 | OPS-005 |
| RN-TRA-007 | OPS-005 |
| RN-TRA-008 | OPS-006 |
| RN-TRA-009 | OPS-007, OPS-009 |
| RN-TRA-010 | OPS-008, OPS-010 |
| RN-TRA-011 | OPS-011 |
| RN-TRA-012 | OPS-013, OPS-014 |
| RN-TRA-013 | OPS-013, OPS-015, OPS-016 |
| RN-TRA-014 | OPS-017 a 019 |
| RN-TRA-015 | OPS-020, OPS-027 |
| RN-TRA-016 | OPS-020, OPS-028 |
| RN-TRA-017 | OPS-021, OPS-022 |
| RN-TRA-018 | OPS-023, OPS-025, OPS-026 |
| RN-TRA-019 | OPS-024 |
| RN-TRA-020 | OPS-022 |
| RN-TRA-021 | OPS-030, OPS-032 |
| RN-TRA-022 | OPS-030, OPS-031 |
| RN-TRA-023 | OPS-033 |
| RN-TRA-024 | OPS-029, OPS-034 |
| RN-TRA-025 | OPS-035 |
| RN-TRA-026 | OPS-035 |
| RN-TRA-027 | OPS-035 |
| RN-TRA-028 | OPS-037 |
| RN-TRA-029 | OPS-038 |
| RN-TRA-030 | OPS-039 |
| RN-VEN-001 | VEN-001, VEN-004, VEN-007, VEN-156, VEN-190, VEN-198, VEN-205 |
| RN-VEN-002 | VEN-001, VEN-008, VEN-116 |
| RN-VEN-003 | VEN-002, VEN-003, VEN-006, VEN-146, VEN-150, CAJ-011 |
| RN-VEN-004 | VEN-005, VEN-006, VEN-146, VEN-150 |
| RN-VEN-005 | VEN-006 |
| RN-VEN-006 | VEN-162, VEN-163 |
| RN-VEN-007 | VEN-124 a 127, VEN-171 |
| RN-VEN-008 | VEN-033, VEN-042, VEN-167 |
| RN-VEN-009 | VEN-033 |
| RN-VEN-010 | VEN-030, VEN-033 |
| RN-VEN-011 | VEN-021, VEN-033 |
| RN-VEN-012 | VEN-018, OPS-049 |
| RN-VEN-013 | VEN-018, VEN-032 |
| RN-VEN-014 | VEN-024, VEN-028, VEN-031, VEN-043 |
| RN-VEN-015 | VEN-112, VEN-115, VEN-119, VEN-147 |
| RN-VEN-016 | VEN-165 |
| RN-VEN-017 | VEN-035 |
| RN-VEN-018 | VEN-040 |
| RN-VEN-019 | VEN-015, VEN-022, VEN-023, VEN-025, VEN-035, VEN-039 |
| RN-VEN-020 | VEN-030, VEN-033 |
| RN-VEN-021 | VEN-022, VEN-112 |
| RN-VEN-022 | VEN-033 |
| RN-VEN-023 | VEN-034 |
| RN-VEN-024 | VEN-024, VEN-026 a 028, VEN-043 |
| RN-VEN-025 | VEN-048, VEN-049 |
| RN-VEN-026 | VEN-048, VEN-054 |
| RN-VEN-027 | VEN-051, VEN-053 |
| RN-VEN-028 | VEN-009, VEN-057, VEN-065, VEN-070 |
| RN-VEN-029 | VEN-054, VEN-055, VEN-057, VEN-058, VEN-063, VEN-067 |
| RN-VEN-030 | VEN-060, VEN-062 |
| RN-VEN-031 | VEN-054, VEN-062 |
| RN-VEN-032 | VEN-054, VEN-056, VEN-063 |
| RN-VEN-033 | VEN-072 a 080 |
| RN-VEN-034 | VEN-065, VEN-067, VEN-069 a 071, VEN-168 |
| RN-VEN-035 | VEN-065, VEN-066, VEN-069, VEN-071, VEN-202 |
| RN-VEN-036 | VEN-054, VEN-060, VEN-084, VEN-087 |
| RN-VEN-037 | VEN-054 a 057, VEN-064 |
| RN-VEN-038 | VEN-165, VEN-166 |
| RN-VEN-039 | VEN-104, VEN-111 |
| RN-VEN-040 | VEN-097, VEN-101, VEN-103 |
| RN-VEN-041 | VEN-092, VEN-095, VEN-096, VEN-104 |
| RN-VEN-042 | VEN-097 a 099, VEN-102, VEN-104, VEN-109 |
| RN-VEN-043 | VEN-088 a 090, VEN-093, VEN-104 |
| RN-VEN-044 | VEN-088, VEN-092, VEN-094, VEN-104, VEN-105, CAJ-028, CAJ-030 |
| RN-VEN-045 | VEN-044, VEN-045, VEN-110 |
| RN-VEN-046 | VEN-096, VEN-162 |
| RN-VEN-047 | VEN-141, VEN-143, VEN-144, CAJ-031 |
| RN-VEN-048 | VEN-142 |
| RN-VEN-049 | VEN-112, VEN-146 |
| RN-VEN-050 | VEN-112, VEN-115 |
| RN-VEN-051 | VEN-112, VEN-123, VEN-146 |
| RN-VEN-052 | VEN-147, VEN-148, VEN-152, VEN-181 |
| RN-VEN-053 | VEN-045, VEN-047, VEN-153, VEN-154, VEN-203 |
| RN-VEN-054 | VEN-157 a 159, VEN-161 |
| RN-VEN-055 | VEN-155, VEN-158 |
| RN-VEN-056 | VEN-162, VEN-164, VEN-169 |
| RN-VEN-057 | VEN-120, VEN-170 |
| RN-VEN-058 | VEN-116 |
| RN-VEN-059 | VEN-128, VEN-132, VEN-148, VEN-196 |
| RN-VEN-060 | VEN-117, VEN-148 |
| RN-VEN-061 | VEN-118, VEN-173 |
| RN-VEN-062 | VEN-172, VEN-177, VEN-180, VEN-186, VEN-203 |
| RN-VEN-063 | VEN-172, VEN-178, VEN-187, VEN-188 |
| RN-VEN-064 | VEN-189 |
| RN-VEN-065 | VEN-181, VEN-184, VEN-185 |
| RN-VEN-066 | VEN-183, VEN-184, DEV-011 |
| RN-VEN-067 | VEN-172, VEN-175, VEN-176, VEN-182, CAJ-029, CAJ-032 |
| RN-VEN-068 | VEN-173, VEN-181 |
| RN-VEN-069 | VEN-173, VEN-174, VEN-181 |
| RN-VEN-070 | VEN-192 |
| RN-VEN-071 | VEN-193 |
| RN-VEN-072 | VEN-191, VEN-194, VEN-195 |
| RN-VEN-073 | VEN-191, VEN-196 |
| RN-VEN-074 | VEN-200, VEN-202, VEN-204, DEV-052 |
| RN-VEN-075 | VEN-010, VEN-011, VEN-017, VEN-084, VEN-085 |
| RN-VEN-076 | VEN-035 |
| RN-VEN-077 | VEN-001, VEN-009, VEN-057 |
| RN-VEN-078 | VEN-206 |
| RN-VEN-079 | VEN-121 |
| RN-VEN-080 | VEN-033, VEN-121 |
| RN-VEN-081 | VEN-096, VEN-099, VEN-101, VEN-121 |

Otras reglas de las partes A y B que los casos citan para trazabilidad (se comprueban en esas partes): RN-EMP-003, RN-EMP-004, RN-EMP-016 (VEN-065, VEN-124), RN-SER-008, RN-SER-013 (VEN-125, DEV-033), RN-IMP-008 a RN-IMP-011 (VEN-206 a VEN-209, CAJ-053, CAJ-055, CAJ-056), RN-STK-011 y RN-STK-017 (DEV-017, DAS-026), RN-LOT-004 y RN-LOT-010 (OPS-078), RN-GEN-013 y RN-GEN-014 (OPS-132, OPS-133) y RN-GEN-021 (DAS-038).

### 10.4 Reglas del alcance sin caso propio

| Regla | Motivo |
|---|---|
| RN-VEN-082 | Índices de rendimiento de ventas, detalle y pagos: sin efecto visible para el usuario ni resultado comprobable por un tester. |
| RN-VEN-083 | Guards de la entidad `Venta` que ninguna ruta alcanza: no producen mensajes (la API o el validador responden antes). |
| RN-CAJ-006 | Guards de la entidad `PosSesionCaja` que ninguna ruta alcanza (`TC-CAJ-009` revisa las restricciones de la base, no los guards). |
| RN-GEN-015 | Denegación por defecto de controladores sin `[Authorize]`: regla transversal de acceso, cubierta por los casos de autenticación de la parte A. |

### 10.5 Verificación

| Comprobación | Resultado |
|---|---|
| Casos escritos | 510, sin IDs repetidos |
| Escenarios (`Escenario:` y `Esquema del escenario:`) de las 76 historias | 566 de 566 con caso |
| Historias con al menos un caso | 76 de 76 |
| Casos de uso con al menos un caso | 76 de 76 |
| Reglas del alcance con comportamiento comprobable con caso | 320 de 324 (las cuatro restantes, en 10.4) |
| Los títulos de escenario citados en «Cubre» coinciden con los de las historias | Sí (0 sin coincidencia) |
| Los `HU-…`, `CU-…` y `RN-…` citados existen en sus documentos | Sí (0 inexistentes) |
| Referencias cruzadas `TC-…` dentro del documento apuntan a casos existentes | Sí (0 inexistentes) |
| Bloques de código cerrados | El documento no usa bloques de código |
| Enlaces relativos a archivos | Todos los enlaces relativos del documento (secciones 1, 2, 9 y 10) apuntan a archivos existentes bajo `sofia-docs` |

## 11. Fuentes

Documentos del repositorio de documentación (`sofia-docs`):

- `_inventario/REGLAS-AGENTES.md` y `_inventario/QA-AGENTES.md` (reglas comunes e instrucciones de los casos de prueba).
- `ola-2-funcional/12-historias-de-usuario/12c-punto-de-venta-y-caja.md` (fuente principal: 76 historias, 566 escenarios).
- `ola-2-funcional/13-casos-de-uso/13c-punto-de-venta-y-caja.md` (76 casos de uso; flujos básicos, alternativos y tablas de excepciones).
- `ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md`, `01c2-ventas-y-comprobantes.md`, `01c3-caja-devoluciones-dashboard.md`, `01d-modulos-solo-api.md` (y, solo para comprobar la existencia de reglas citadas, `01a-administracion-y-seguridad.md` y `01b-catalogos.md`).
- `ola-1-fundamentos/19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md` y `19f-punto-de-venta.md` (a través de las historias).
- `ola-1-fundamentos/21-catalogo-codigos-error.md` (códigos de error; enlazado).
- `ola-1-fundamentos/05-seguridad/05a-autenticacion-y-controles.md` y `05b-matriz-de-permisos.md` (enlazados).
- `ola-1-fundamentos/08-guia-del-desarrollador.md` y `10-despliegue-y-operacion.md` (entorno; no copiados).
- `ola-2-funcional/manual-usuario/capitulo-03-empleados-roles-cuentas.md` (rol «Cajero» con 19 permisos, sección 5.3; datos ficticios del manual).

Código leído para comprobar los nombres de los campos de las peticiones de venta:

- `sofia-apis/src/SOFIA.Application/Ventas/Commands/CreateVenta/CreateVentaCommand.cs:12-31`
- `sofia-apis/src/SOFIA.Application/Ventas/Commands/CompletarVenta/CompletarVentaCommand.cs`
- `sofia-apis/src/SOFIA.Application/Ventas/Commands/AnularVenta/AnularVentaCommand.cs`
- `sofia-apis/src/SOFIA.Application/Ventas/Commands/ActualizarVentaPendiente/ActualizarVentaPendienteCommand.cs`

El resto de las afirmaciones del documento proviene de las historias y los casos de uso citados, que a su vez declaran sus fuentes en el código y en los `es.json` del SPA; en esta tarea no se leyeron esos archivos adicionales ni se ejecutó la aplicación.

