# Capítulo 9 — Ventas, anulación, devolución y cierre de caja

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-11-02 |
| **Backend** | `sofia-apis`, rama `main`, commit `9c16f35` (más nuevo que la línea base `43f5529`; `git status --short` sin cambios locales al empezar) |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `7070e1a` (más nuevo que la línea base `eeffbb6`; `git status --short` sin cambios locales al empezar) |
| **Método** | Recorrido en la aplicación local más inventario de pantallas |

## Índice

1. Introducción
2. Antes de empezar
3. Buscar ventas
4. Ver el detalle de una venta y reimprimir el comprobante
5. Anular una venta cobrada
6. Anular una venta en espera
7. Devolver productos
8. Cerrar tu caja con arqueo
9. Lista y detalle de sesiones de caja
10. Lo que ve un cajero frente a un administrador
11. Diferencias con la documentación técnica
12. Pendientes y «No verificado»
13. Lista de cobertura
14. Fuentes

## 1. Introducción

En este capítulo aprendes a **revisar lo que vendiste y a corregirlo**, y a **cerrar tu caja** al terminar el turno. Vas a:

- **Buscar ventas** por empleado, cliente, fechas y estado, y **exportarlas a Excel**.
- Ver el **detalle de una venta**: líneas, pagos, receta verificada y comprobante, y **reimprimir** el comprobante.
- **Anular** una venta cobrada o una venta en espera, y entender por qué hace falta tu caja abierta.
- **Devolver** productos de una venta (total o parcialmente): nota de crédito, reembolso y destino de la mercadería.
- **Cerrar tu caja con arqueo**: declarar el efectivo contado, ver la diferencia y el estado «Cerrada» o «Cuadrada».
- Ver la **lista y el detalle de sesiones de caja** y exportarlas.
- Saber qué cambia entre un **cajero** y un **administrador**.

Es para el **cajero** y para el **administrador**. Abrir la caja y vender se explicó en el [capítulo 8](capitulo-08-caja-y-venta.md).

**Ideas que debes tener claras**

- **Anular una venta cobrada o devolver productos exige tu propia caja abierta.** El dinero que sale (el efectivo del cliente en una anulación, el reembolso en una devolución) sale de **tu** cajón, y por eso queda en **tu** arqueo.
- **Una venta con devoluciones ya no se puede anular**, y una venta anulada ya no se puede devolver.
- **El cierre de caja no tiene tolerancia**: la sesión queda «Cuadrada» solo si la diferencia es exactamente 0; con cualquier otra diferencia, aunque sea de un centavo, queda «Cerrada».
- **El sistema no envía comprobantes a SUNAT**, tampoco las notas de crédito ni las anulaciones: todo queda solo en SOFIA.

## 2. Antes de empezar

- Haz los capítulos 1 a 8. Este capítulo usa las ventas del capítulo 8: ocho ventas cobradas (`B001-00000001` a `B001-00000008`) y una venta en espera (código `115BF400`), todas del cajero `carlos.cajero` en la Sede Principal, y su caja abierta con un monto de apertura de S/ 100.00.
- Para anular, devolver y cerrar la caja trabajaste **como cajero** (`carlos.cajero`); el administrador `ana.manual` se usó solo para comparar lo que ve (sección 10) y para comprobar que **sin caja propia** no puede anular ni devolver.
- **Permisos.** Los nombres son los de la pantalla de permisos de un rol (Administración › Roles & Permisos): primero el módulo y luego la acción.

| Tarea | Módulo | Acción |
|---|---|---|
| Buscar ventas, ver el detalle, reimprimir y exportar | Ventas | `Leer` |
| Anular una venta | Ventas | `Anular` (además de `Leer`) |
| Devolver productos | Devoluciones | `Procesar devolución` (además de `Ventas › Leer`) |
| Ver la lista y el detalle de sesiones de caja y exportarlas | Sesiones de Caja | `Leer` |
| Cerrar una caja | Sesiones de Caja | `Cerrar caja` (además de `Leer`) |
| Filtrar por empleado o por cliente en la búsqueda de ventas | Empleados y Pacientes | `Leer` en cada uno |

- El rol «Cajero» del capítulo 3 tiene todo esto: `Ventas` (`Leer`, `Crear`, `Actualizar`, `Anular`), `Devoluciones` (`Leer`, `Procesar devolución`) y `Sesiones de Caja` (`Leer`, `Aperturar caja`, `Cerrar caja`).
- Los avisos aparecen abajo al centro, con un botón **Cerrar**, y desaparecen a los pocos segundos. Los textos están copiados de la pantalla.
- Los importes van en soles (S/) con dos decimales. Las horas son las de tu navegador.
- **Aviso sobre «hoy».** Las ventas del capítulo 8 se hicieron el 02/11/2026; este capítulo se recorrió el 03/11/2026. Por eso el Dashboard del cajero mostraba «Vendido hoy S/ 0.00» y «Ayer: S/ 104.10»: «hoy» es el día calendario, y las ventas de ayer ya no cuentan. La caja, en cambio, sigue abierta: el turno de una caja no termina a medianoche, termina cuando la cierras.

## 3. Buscar ventas

**Cómo llegar:** menú lateral › **PUNTO DE VENTA** › **Ventas**. La pantalla se titula «Ventas» y las migas dicen «Inicio / Punto de Venta / Ventas».

La pantalla **no muestra resultados al entrar**: primero debes pulsar **Buscar**. Arriba a la derecha de la tarjeta de filtros está el botón **Nueva Venta POS** (solo si tienes `Ventas › Crear`).

### Pasos

1. Entra a **Ventas**. Verás la tarjeta «Ventas» con el subtítulo «Consulta el historial de ventas registradas».
2. Elige los filtros que necesites (todos son opcionales; la tabla siguiente los describe).
3. Haz clic en **Buscar**. Aparece la tarjeta «Resultados (N)» con la tabla.
4. Haz clic en **Limpiar** para vaciar los filtros, ocultar la tabla y empezar de nuevo.

### Filtros

| Campo | Qué escribir o elegir | Obligatorio | Reglas |
|---|---|---|---|
| Empleado | Empieza a escribir un nombre (con lupa) y **elige una opción de la lista**; la opción muestra el nombre completo, por ejemplo «Carlos Cajero Demo» | No | **El filtro solo se aplica si eliges una opción de la lista.** Si escribes el nombre y pulsas **Buscar** sin elegirlo, el filtro se ignora sin avisar. Este es el filtro «Cajero»: no existe otro |
| Cliente | Empieza a escribir un nombre o un documento (con lupa) y elige una opción, por ejemplo «Luis Paciente Demo» | No | Igual que Empleado: solo filtra si eliges la opción. Las ventas sin cliente aparecen como «Público General» |
| Fecha desde | Una fecha, con el selector de calendario o escrita como `dd/mm/aaaa` | No | Incluye todo el día |
| Fecha hasta | Una fecha | No | Incluye todo el día |
| Estado | «— Todos —», «Completada», «Anulada», «Devuelta» o «Pendiente» | No | «Pendiente» es la venta en espera |

**No hay filtro por código de venta, por monto ni por sucursal.** La búsqueda siempre trae las ventas de **tu sucursal** (la del inicio de sesión), de **todos** los empleados: un cajero y un administrador ven lo mismo (sección 10).

### La tabla de resultados

Encabezado «Resultados (N)» y estas columnas, de la venta más reciente a la más antigua:

| Columna | Qué muestra |
|---|---|
| Código de Venta | El número de boleta (por ejemplo `B001-00000008`) o, si la venta está en espera y todavía no tiene boleta, un código provisional de 8 caracteres (por ejemplo `115BF400`). Es un enlace: abre el detalle |
| Fecha | `dd/mm/aaaa hh:mm`, en la hora de tu navegador |
| Cliente | El nombre, o «Público General» si no hay cliente |
| Empleado | El cajero que hizo la venta |
| Total (S/) | El total de la venta |
| Estado | «Completada», «Anulada», «Devuelta» o «Pendiente» (texto, sin colores) |
| Ítems | Cuántas líneas tiene la venta |

Bajo la tabla, a la izquierda el botón **Exportar** y a la derecha el paginador: «Items por página» con 10, 25 o 50 (10 por defecto), «Página 1 de 1» y los botones «Primera página», «Página anterior», «Siguiente página» y «Última página». Sin resultados la tabla dice «Sin resultados».

### Lo que se comprobó en la aplicación

| Búsqueda | Resultado |
|---|---|
| Solo **Buscar**, sin filtros | «Resultados (9)»: las 8 ventas cobradas y la venta en espera `115BF400` |
| Estado «Pendiente» | «Resultados (1)»: `115BF400`, S/ 7.00, 2 ítems |
| Estado «Anulada» | «Resultados (0)» y «Sin resultados» (todavía no se había anulado nada) |
| Cliente «Luis Paciente Demo» (elegido de la lista) | «Resultados (3)»: `B001-00000007`, `B001-00000006` y `B001-00000003` |
| Cliente escrito («Luis») **sin elegir la opción** | «Resultados (9)»: el filtro se ignoró |
| Empleado «Carlos Cajero Demo» + cliente Luis + «Fecha desde» 03/11/2026 | «Resultados (0)»: no hay ventas desde esa fecha |
| «Fecha desde» y «Fecha hasta» = 02/11/2026 | «Resultados (9)» |
| **Limpiar** | Vacía los cuatro campos y oculta la tabla |

### Exportar a Excel

Con los resultados a la vista, haz clic en **Exportar**. Se descarga el archivo `ventas.xlsx` con **los mismos filtros** que tienes puestos (no solo la página que ves). La hoja trae siete columnas, en este orden: «Código de Venta», «Fecha», «Cliente», «Empleado», «Total (S/)», «Estado» e «Ítems»; la exportada con «Fecha desde» y «Fecha hasta» del 02/11/2026 trajo las 9 ventas.

> **Cuidado con la fecha del Excel.** En la exportación probada, las ventas hechas el 02/11/2026 a las 19:24 y 19:25 (hora de Lima) aparecieron con la fecha **03/11/2026**. El Excel muestra la fecha en hora universal (UTC) sin convertirla a tu hora local, así que las ventas hechas a partir de las 19:00 salen con la fecha del día siguiente. En la pantalla la fecha es la correcta. Tenlo presente al revisar el Excel (regla `RN-VEN-078`; ver sección 11).

### Errores que puedes ver

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Error al buscar ventas» | Falló la búsqueda o la exportación | Vuelve a pulsar **Buscar**; si sigue, revisa tu conexión o avisa a soporte |
| Redirección al Dashboard con «No tienes permisos para acceder a esta pantalla.» | Tu rol no tiene `Ventas › Leer` | Pide ese permiso a tu administrador |
| Los desplegables de Empleado o Cliente no se llenan | Posible falta de `Empleados › Leer` o `Pacientes › Leer` en tu rol (**No verificado en la aplicación:** no se quitaron permisos a un rol; fuente `producto/pantallas/punto-de-venta` §6) | Pide el permiso |

**Paginación:** el paginador solo se exhibió con 9 resultados (una sola página); cambiar a 25 mostró «Página 1 de 1». **No verificado en la aplicación:** pasar a una segunda página, porque no hay más de 10 ventas en la empresa de prueba (fuente: `producto/pantallas/punto-de-venta` §6).

## 4. Ver el detalle de una venta y reimprimir el comprobante

**Cómo llegar:** en **Ventas**, haz clic en el código de la venta (primera columna). También puedes volver con el botón de flecha de la cabecera o con la miga «Ventas». Al volver a la lista, la pantalla recuerda tus filtros y los resultados.

### Qué muestra la pantalla «Detalle de Venta»

La pantalla es **solo de lectura**. De arriba hacia abajo:

| Zona | Qué ves |
|---|---|
| Encabezado de la tarjeta | El código de la venta (por ejemplo `B001-00000003`) y debajo «02/11/2026 19:21 — Completada» (fecha, hora y estado, escritos como texto). A la derecha, los botones **Reimprimir Comprobante**, **Devolver** y **Anular Venta**, según el estado y tus permisos (secciones 5 a 7) |
| Motivo de Anulación | Solo en una venta anulada: «Motivo de Anulación:» y el texto que escribió quien la anuló |
| Cabecera | «Cliente» (el nombre, o «Público General»), «Empleado» (el cajero), «Total (S/)» (por ejemplo «S/ 14.40») |
| Verificación de receta | Solo si la venta se cobró con la casilla «Receta verificada»: «Receta verificada por Carlos Cajero el 02/11/2026 19:23» (se vio en `B001-00000005`) |
| Comprobante | «Nº Comprobante:» con el tipo y el número («Boleta B001-00000003») y «Estado SUNAT:» con «Pendiente de envío a SUNAT». Solo si la venta tiene comprobante; una venta en espera no lo tiene |
| Productos Vendidos | Columnas «Producto», «Lote», «Cantidad», «Devuelto», «P. Unitario (S/)» y «Subtotal (S/)» |
| Pagos Registrados | Columnas «Método de Pago», «Monto (S/)», «Referencia de Operación» (con «—» si no hay) y «Fecha de Pago». Debajo, «Vuelto: S/ x.xx» si hubo vuelto |
| Devoluciones / Notas de crédito | Solo si la venta tiene devoluciones (sección 7) |

Datos que se comprobaron en la aplicación:

| Venta | Lo que mostró |
|---|---|
| `B001-00000003` | Cliente «Luis Paciente Demo». Una línea: «Paracetamol Demo 500 mg», lote `LOT-PARA-2027`, cantidad «3 × Blíster (30)» (tres blísteres, 30 unidades base), devuelto 0, precio unitario 0.48 y subtotal 14.40. **Dos pagos**: Efectivo 10.00 y Yape / Plin 4.40 con referencia `YAPE-0001`. Sin «Vuelto» |
| `B001-00000005` | Cliente «Público General». «Receta verificada por Carlos Cajero el 02/11/2026 19:23». Tres líneas (Paracetamol 4, Omeprazol 5 y Amoxicilina 2 unidades), un pago en Efectivo de 10.00 y «Vuelto: S/ 1.10» |

Dos detalles útiles:

- **La cantidad de una línea vendida por presentación** se lee «{n} × {presentación} ({unidades base})». **«Devuelto»** y las devoluciones siempre se cuentan en **unidades base**.
- **El «P. Unitario»** es el precio por unidad base (0.48 = 4.80 del blíster ÷ 10).

### Reimprimir el comprobante

1. En el detalle, haz clic en **Reimprimir Comprobante** (aparece si la venta tiene comprobante y no está «Anulada»; no necesita un permiso adicional a `Ventas › Leer`).
2. Se abre un cuadro con el título «Venta Procesada Exitosamente» y la línea «Comprobante registrado. Pendiente de envío a SUNAT.». **Ambos textos son fijos**: salen igual aunque no se haya vendido nada ahora y para cualquier estado del comprobante.
3. El cuadro muestra «Nº Comprobante:» («Boleta B001-00000003»), «Estado SUNAT:» («Pendiente de envío a SUNAT»), «Total Pagado:» («S/ 14.40») y «Medio de Pago:» («Efectivo, Yape / Plin»).
4. Abajo hay tres botones: **Nueva Venta POS** (te lleva al terminal de venta; solo con `Ventas › Crear`), **Imprimir Comprobante** y **Volver a Caja POS** (te lleva a la lista de ventas).
5. Haz clic en **Imprimir Comprobante** para abrir el diálogo de impresión de tu navegador (ticket de rollo de 80 mm). **No verificado en la aplicación:** no se pulsó **Imprimir Comprobante** porque abre el diálogo de impresión del sistema, que el navegador integrado no controla.

El ticket que se imprime, comprobado en la reimpresión de `B001-00000003`, dice en este orden: razón social («Botica Manual Demo S.A.C.»), «RUC: 20538291746», dirección de la sucursal, «Boleta B001-00000003», fecha y hora, la tabla «Producto / Lote», «Cant.», «P. Unit.» y «Subtotal» (la línea sale como «Paracetamol Demo 500 mg (LOT-PARA-2027)», «3 × Blíster (30)», 4.80 y 14.40), «Datos del Cliente:» y el nombre, «Documento:» («DNI 70000001»), «Subtotal (Op. Gravada): S/ 12.20», «IGV (18%): S/ 2.20», «TOTAL A PAGAR: S/ 14.40», cada pago, «Estado SUNAT: Pendiente de envío a SUNAT» y «Cajero: Carlos Cajero». No lleva código QR ni leyenda de SUNAT.

La reimpresión se puede hacer las veces que haga falta. En una venta **anulada** no aparece el botón **Reimprimir Comprobante**.

### Errores que puedes ver

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Error al cargar la venta» | No se pudo abrir la venta (por ejemplo, es de otra sucursal) | Vuelve a la lista y ábrela de nuevo; si la venta es de otra sucursal, solo se ve entrando con una cuenta de esa sucursal |

## 5. Anular una venta cobrada

**Cuándo anular.** Anula una venta cuando se cobró **por error o el cliente se arrepintió** y devuelves **todo** el dinero: la anulación vuelve **toda** la venta atrás (stock, comprobante y receta). Si el cliente solo devuelve una parte, no anules: haz una **devolución** (sección 7).

**Qué cambia al anular.** Estos efectos se comprobaron con la venta `B001-00000005` (8.90 soles: tres líneas, pago en efectivo de 10.00 y vuelto de 1.10):

| Qué | Antes | Después |
|---|---|---|
| Estado de la venta | «Completada» | «Anulada», con «Motivo de Anulación: Cliente se arrepintió de la compra» |
| «Estado SUNAT» del comprobante | «Pendiente de envío a SUNAT» | «Anulado» (solo dentro de SOFIA: no se avisa a SUNAT, porque el sistema no envía comprobantes) |
| Stock `LOT-PARA-2027` (Paracetamol), Sede Principal | 40 | 44 (volvieron las 4 unidades vendidas) |
| Stock `LOT-AMOX-2027` (Amoxicilina) | 88 | 90 (volvieron 2) |
| Stock `LOT-OME-2027` (Omeprazol) | 95 | 100 (volvieron 5) |
| «Efectivo esperado» de tu caja en el Dashboard | S/ 172.70 | **S/ 163.80** (bajó 8.90: lo cobrado en efectivo menos el vuelto) |
| Ventas de «ayer» en el Dashboard | S/ 104.10 | S/ 95.20 (el total vendido ya no cuenta la venta anulada) |
| Botones del detalle | **Reimprimir Comprobante**, **Devolver**, **Anular Venta** | Ninguno: solo queda la flecha de volver |
| La receta de la venta, si la tenía | Cuenta una dispensación | Se libera la dispensación (la venta anulada ya no cuenta). **No verificado en la aplicación:** `B001-00000005` no usó una receta registrada; sale de la regla `RN-VEN-068` |

Los **pagos** de la venta anulada siguen mostrándose en el detalle («Efectivo 10.00» y «Vuelto: S/ 1.10»): la anulación no los borra ni crea un reverso. La caja lo refleja de otra forma: como anulaste la venta desde la **misma caja en la que se cobró**, esa venta **sale del arqueo** (cobrada y devuelta en el mismo cajón: efecto neto cero).

### Por qué hace falta tu caja abierta

Para anular una venta **cobrada**, el sistema te pide **tu propia caja abierta en la sucursal de la venta**. Si el cliente recibe su dinero de vuelta, ese dinero sale de **tu** cajón, y la anulación queda anotada contra esa caja (reglas `RN-VEN-067` y `RN-CAJ-012`). **Una venta en espera no necesita caja** (sección 6). No hace falta ser el cajero que hizo la venta: cualquier usuario con `Ventas › Anular` de la misma sucursal puede anularla, siempre que tenga su caja abierta.

### Pasos

1. Entra a **Ventas** (menú lateral › **PUNTO DE VENTA** › **Ventas**), haz clic en **Buscar** y abre la venta con su código.
2. Comprueba que el detalle diga «Completada» y haz clic en **Anular Venta** (botón de borrado, arriba a la derecha de la tarjeta).
3. Se abre el cuadro «Anular Venta» con el texto «Esta acción revierte el stock descontado. Ingresa el motivo de la anulación.».
4. Escribe el **Motivo de Anulación**.
5. Haz clic en **Anular Venta** (en el cuadro). Si te arrepientes, haz clic en **Cancelar** (o pulsa Esc): no pasa nada.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Motivo de Anulación | Una frase corta, por ejemplo «Cliente se arrepintió de la compra» | Sí | Si lo dejas vacío y haces clic en **Anular Venta**: «El motivo es obligatorio.» y el cuadro sigue abierto. Máximo 255 caracteres (el campo **no** corta el texto: si te pasas, el botón no hace nada y no dice por qué). Si escribes solo espacios, el cuadro se cierra y aparece «Error al anular la venta.» (la venta no se anula) |

### Qué pasa al terminar

Aparece el aviso «Venta anulada correctamente.» y el detalle se recarga ya como «Anulada», con el motivo escrito. Se comprobó lo mismo en **Ventas**: al buscar con el estado «Anulada» la venta aparece (ver sección 3).

### Si no ves el botón

| Situación | Motivo |
|---|---|
| No aparece **Anular Venta** | Tu rol no tiene `Ventas › Anular`; o la venta no es «Completada» ni «Pendiente» (ya está «Anulada» o «Devuelta»); o la venta **ya tiene devoluciones** (aunque sea una parcial) |

### Errores que puedes ver

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «El motivo es obligatorio.» | Dejaste el motivo vacío | Escribe el motivo |
| «Necesitas tener tu propia caja abierta en esta sucursal para anular una venta cobrada.» | Anulas una venta cobrada sin tener **tu** caja abierta. Se comprobó con el administrador `ana.manual`, que no tiene caja: la venta quedó «Completada» | Abre tu caja (capítulo 8) y vuelve a anular. La caja abierta de otra persona no te sirve |
| «Esta venta tiene devoluciones registradas y ya no puede anularse. Gestiona los cambios desde Devoluciones.» | La venta ya tiene una devolución. **No viste este mensaje en la aplicación**: el botón **Anular Venta** desaparece en cuanto la venta tiene devoluciones. Además, el texto habla de «Devoluciones», que no es una opción de menú: las devoluciones se hacen desde el detalle de la venta | Si hace falta corregir más, haz otra devolución de lo que queda (sección 7) |
| «Error al anular la venta.» | Mensaje general: motivo con solo espacios, venta que ya no se puede anular o de otra sucursal | Revisa el estado de la venta y el motivo |
| «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» | Otra persona cambió la venta a la vez | Actualiza la pantalla y vuelve a intentar. **No verificado en la aplicación**: no se provocó un conflicto simultáneo (fuente: `producto/pantallas/punto-de-venta` §8) |

**Ya anulada, no hay vuelta atrás:** no existe un botón para reactivar una venta anulada ni se puede devolver. Si te equivocaste, vende de nuevo (capítulo 8).

## 6. Anular una venta en espera

Una venta en espera (estado «Pendiente») es un carrito guardado que todavía **no se cobró**: no tiene pagos ni boleta, pero **ya reservó el stock**. Si el cliente no vuelve, anúlala para devolver esas unidades al inventario. **No necesita caja abierta** y no toca el efectivo de ninguna caja.

### Pasos

1. Entra a **Ventas**, elige el estado «Pendiente» y haz clic en **Buscar** (o abre desde el Dashboard el enlace «Ver ventas en espera»).
2. Abre la venta con su código provisional (en la prueba, `115BF400`). El detalle dice «Pendiente» y arriba a la derecha tiene los botones **Cobrar Venta** y **Anular Venta**. No aparecen **Reimprimir Comprobante** ni **Devolver**, porque la venta aún no tiene comprobante.
3. Haz clic en **Anular Venta**, escribe el **Motivo de Anulación** (por ejemplo «Cliente no volvió a la caja») y confirma con **Anular Venta**. Las reglas del motivo son las de la sección 5.

### Qué pasa al terminar

Aparece «Venta anulada correctamente.» y el detalle queda en «Anulada» con el motivo y sin botones. Resultado comprobado con `115BF400` (S/ 7.00; 6 unidades de Paracetamol y 10 de Ibuprofeno):

| Qué | Antes | Después |
|---|---|---|
| Stock `LOT-PARA-2027`, Sede Principal | 44 | 50 (volvieron 6) |
| Stock `LOT-IBU-2027`, Sede Principal | 74 | 84 (volvieron 10) |
| Dashboard, tarjeta «Ventas en espera» | 1 («Por cobrar: S/ 7.00») | 0 («Por cobrar: S/ 0.00») |
| «Efectivo esperado» de tu caja | S/ 163.80 | S/ 163.80 (no cambia: la venta nunca cobró nada) |

Los errores posibles son los mismos de la sección 5, excepto el de la caja: en una venta en espera **no** se pide caja abierta.

## 7. Devolver productos

Haz una **devolución** cuando el cliente devuelve **una parte o todo** lo que compró y quieres dejar constancia con una **nota de crédito**. A diferencia de la anulación, la devolución se hace **por línea y por cantidad**, y deja la venta como está (solo pasa a «Devuelta» si se devolvió todo).

**Cómo llegar:** **Ventas** › abre la venta › botón **Devolver** (arriba a la derecha del detalle).

**Qué necesitas:** el permiso `Devoluciones › Procesar devolución`, **tu propia caja abierta** en la sucursal de la venta (el reembolso sale de tu caja; se pide siempre, incluso si reembolsas por tarjeta), que la venta esté «Completada», que tenga boleta y que la sucursal tenga una **serie de nota de crédito activa** (capítulo 2). Las ventas de otra sucursal no se pueden devolver, ni siquiera siendo administrador.

### Pasos

1. Abre la venta en **Ventas** y comprueba que diga «Completada».
2. Haz clic en **Devolver**. Se abre el cuadro «Procesar devolución — {código de la venta}» con el texto «Indica cuántas unidades devuelve el cliente en cada línea y su destino. Se emitirá una nota de crédito y el reembolso saldrá de tu caja abierta.».
3. En la tabla de líneas, escribe en **A devolver** cuántas unidades devuelve el cliente en cada línea (o haz clic en **Devolver todo lo pendiente** para poner en cada línea todo lo que queda por devolver).
4. En **Destino**, elige qué pasa con la mercadería de cada línea.
5. Revisa el **Motivo SUNAT**, elige el **Método de reembolso** y escribe el **Sustento**.
6. Mira el **Reembolso estimado** (a la derecha, abajo) y haz clic en **Procesar devolución**. Para salir sin devolver, haz clic en **Cancelar**.

### Tabla de líneas

Solo se listan las líneas que todavía tienen algo por devolver. **Todas las cantidades van en unidades base**, aunque la línea se haya vendido por presentación (la venta `B001-00000001` fue «1 × Caja (100)» y el cuadro mostró «Vendido 100»).

| Columna | Qué muestra |
|---|---|
| Producto, Lote | Qué se vendió y de qué lote |
| Vendido | Cantidad vendida |
| Devuelto | Cantidad ya devuelta antes |
| Devolvible | Lo que aún se puede devolver (vendido menos devuelto) |
| A devolver | Campo numérico, empieza en 0 |
| Destino | «Reingreso a venta» (por defecto) o «Cuarentena DIGEMID» |

### Campos del formulario

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| A devolver (por línea) | Un número desde 0 hasta lo «Devolvible» | Al menos una línea debe tener más de 0 | **El campo corrige solo lo que escribes:** si escribes más de lo devolvible, se ajusta al máximo (se probó escribir 200 en una línea de 100: quedó en 100). Si no pones ninguna cantidad mayor que 0 y haces clic en **Procesar devolución**: «Ingresa al menos una cantidad a devolver.» |
| Destino | «Reingreso a venta»: la mercadería vuelve al stock. «Cuarentena DIGEMID»: se aparta y **no** vuelve al stock | Sí (viene «Reingreso a venta») | Es por línea |
| Motivo SUNAT | «06 — Devolución total» o «07 — Devolución por ítem» | Sí | **Se pone solo**: «06» si devuelves todo lo devolvible de **todas** las líneas, «07» en cualquier otro caso. Puedes cambiarlo a mano, pero en cuanto cambies una cantidad vuelve a fijarse solo |
| Método de reembolso | «Efectivo», «Yape / Plin», «Tarjeta Débito/Crédito» o «Transferencia» | Sí (viene «Efectivo») | No tiene que ser el mismo método con que pagó el cliente |
| Sustento | Por qué se devuelve, en una frase | Sí | Máximo 255 caracteres. Vacío: «El sustento es obligatorio.». Solo espacios: el botón **no hace nada y no muestra mensaje** (se comprobó) |
| Reembolso estimado | (no se escribe) | — | Suma de «cantidad a devolver × precio unitario» de cada línea. Es el valor de la nota de crédito |

### Qué pasa al terminar

Aparece el aviso «Devolución procesada y nota de crédito emitida.», el cuadro se cierra y el detalle se recarga con una tabla nueva, **«Devoluciones / Notas de crédito»**, con las columnas «Nota de crédito», «Fecha», «Reembolsado (S/)» y «Método de reembolso». La columna «Devuelto» de la tabla de líneas sube.

### Lo que se comprobó en la aplicación

Se hicieron **dos devoluciones**:

| | Devolución 1: parcial, en efectivo | Devolución 2: total, con otro método y cuarentena |
|---|---|---|
| Venta | `B001-00000001` (S/ 45.00; 1 × Caja de Paracetamol = 100 unidades; efectivo 50.00 con vuelto 5.00) | `B001-00000002` (S/ 4.00; 10 unidades de Ibuprofeno; Tarjeta Débito/Crédito) |
| Lo que se devolvió | 10 de 100 unidades (Devolvible 100 → 90) | 10 de 10 (todo) |
| Destino | «Reingreso a venta» | «Cuarentena DIGEMID» |
| Método de reembolso | Efectivo | Tarjeta Débito/Crédito |
| Motivo SUNAT | «07 — Devolución por ítem» (puesto solo) | «06 — Devolución total» (puesto solo) |
| Sustento | «Cliente devuelve 10 tabletas sin abrir» | «Producto no era el pedido» |
| Reembolso estimado y reembolsado | S/ 4.50 (10 × 0.45) | S/ 4.00 (10 × 0.40) |
| Nota de crédito emitida | `BC01-00000001` | `BC01-00000002` |
| Estado de la venta | Sigue «Completada» (botones **Reimprimir Comprobante** y **Devolver**; **Anular Venta** desapareció) | «Devuelta» (solo **Reimprimir Comprobante**) |
| Stock de la Sede Principal | `LOT-PARA-2027`: 50 → **60** (volvieron las 10 unidades) | `LOT-IBU-2027`: 84 → **84** (**no** cambió: la cuarentena no repone stock) |
| «Efectivo esperado» del Dashboard | S/ 163.80 → **S/ 159.30** (bajó 4.50) | S/ 159.30 → **S/ 159.30** (no cambió: el reembolso por tarjeta no sale del cajón) |
| Dashboard, tarjeta «Devoluciones» | 1, «Monto reembolsado: S/ 4.50» | 2, «Monto reembolsado: S/ 8.50» |

Lo que aprendes de esto:

- **Una nota de crédito por devolución.** La numera la serie de nota de crédito de la sucursal (`BC01-00000001`, `BC01-00000002`; el capítulo 2 las creó). Es el comprobante del crédito; no se envía a SUNAT.
- **Solo el reembolso en efectivo baja el efectivo esperado de tu caja.** Yape / Plin, tarjeta y transferencia no lo tocan.
- **«Cuarentena DIGEMID» solo anota el destino.** La mercadería sale de la venta pero no vuelve al stock ni se crea ningún registro de cuarentena: el Dashboard siguió mostrando «Lotes en cuarentena 0». Si de verdad debes aislar el lote, se hace con la operación de cuarentena del módulo DIGEMID (hoy solo por API, sin pantalla).
- **Devolver no libera la dispensación de una receta** (regla `RN-DEV-026`): solo anular una venta la libera, y una venta con devoluciones ya no se puede anular. **No verificado en la aplicación:** las dos ventas devueltas no tenían receta registrada.
- **Una venta «Devuelta» no admite más acciones.** Una venta con devoluciones parciales sí admite más devoluciones hasta devolver todo; no hay plazo máximo para devolver.
- **Una venta anulada no se puede devolver:** su detalle no tiene el botón **Devolver**. El cuadro de devolución solo se alcanza desde una venta «Completada».

### Si no ves el botón **Devolver**

| Situación | Motivo |
|---|---|
| No aparece | Tu rol no tiene `Devoluciones › Procesar devolución`; o la venta no está «Completada» (está «Pendiente», «Anulada» o «Devuelta»); o no tiene comprobante; o ya no queda nada por devolver |
| Lo ves pero al procesar falla | Mira la tabla de errores de abajo |

### Errores que puedes ver

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Ingresa al menos una cantidad a devolver.» | Todas las cantidades están en 0 | Escribe una cantidad mayor que 0 en una línea |
| «El sustento es obligatorio.» | Dejaste el sustento vacío | Escribe el sustento |
| «Necesitas una caja abierta para procesar la devolución.» | No tienes caja abierta. Se comprobó con el administrador `ana.manual` (sin caja): no se emitió ninguna nota de crédito | Abre tu caja (capítulo 8) y repite |
| «La cantidad supera lo que queda por devolver en alguna línea. Actualiza la pantalla e inténtalo de nuevo.» | Otra persona devolvió antes y la pantalla quedó desactualizada. **No verificado en la aplicación:** el campo corrige las cantidades, por lo que este mensaje solo sale con una pantalla vieja (fuente: `producto/pantallas/punto-de-venta` §9) | Cierra el cuadro, recarga la venta y vuelve a intentar |
| «Solo se pueden devolver ventas completadas.» | La venta ya no está «Completada». **No verificado en la aplicación:** el botón ya no aparece en una venta anulada o devuelta | Revisa el estado de la venta |
| «Solo puedes devolver ventas de tu propia sucursal.» | Venta de otra sucursal. **No verificado** | Entra con una cuenta de esa sucursal |
| «La venta no tiene boleta ni factura emitida que acreditar.» | Venta sin comprobante. **No verificado** | — |
| «La sucursal no tiene una serie de nota de crédito activa. Configúrala en Series Fiscales.» | Falta la serie `BC01`/`BC02` activa. **No verificado** | Actívala como indica el capítulo 2 |
| «La venta ya no existe.» / «Una de las líneas ya no pertenece a la venta.» | La venta o una línea cambió. **No verificado** | Recarga la venta |
| «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» | Cambio simultáneo. **No verificado** | Actualiza y repite |
| «Error al procesar la devolución.» | Mensaje general. **No verificado** | Revisa los datos y repite |

**Importante:** la devolución es **permanente**: una vez emitida la nota de crédito, no se puede deshacer desde la pantalla.

## 8. Cerrar tu caja con arqueo

Al terminar el turno **cierras tu caja** contando el efectivo que hay en el cajón y declarándolo. El sistema lo compara con el efectivo que **debería haber** y guarda la diferencia. Esto es el **arqueo**.

**Cómo llegar:** menú lateral › **PUNTO DE VENTA** › **Sesiones de Caja** › **Buscar** › haz clic en el nombre de la sucursal de **tu** sesión abierta (columna «Sucursal»). También puedes entrar con el enlace «Ver mi caja» de la tarjeta «Caja» del Dashboard.

**Quién puede cerrar:** el cajero que abrió la caja o un administrador (rol Admin), y se necesita `Sesiones de Caja › Cerrar caja`. El botón **Cerrar Sesión** se muestra a quien tenga ese permiso en cualquier sesión abierta, pero si no eres quien la abrió ni administrador, al confirmar el sistema responde «Solo el cajero que abrió la caja o un administrador puede cerrarla.» (**No verificado en la aplicación:** no se intentó cerrar la caja de otra persona con un cajero; fuente: `producto/pantallas/punto-de-venta` §1.3).

### Cuánto efectivo debería haber (la fórmula)

**Efectivo esperado = monto de apertura + efectivo cobrado − vuelto entregado − reembolsos en efectivo.** Solo cuentan las ventas **cobradas** en tu caja y **no anuladas**; las ventas en espera no entran. Los pagos con Yape / Plin, tarjeta o transferencia **no** son efectivo y no cuentan (regla `RN-CAJ-011`).

**Cálculo a mano de la caja de la prueba**, antes de cerrar:

| Concepto | Ventas | Monto |
|---|---|---|
| Monto de apertura | — | S/ 100.00 |
| Efectivo cobrado y vuelto | `B001-00000001`: recibió 50.00 y dio vuelto 5.00 | + 45.00 |
| | `B001-00000003`: pago dividido, la parte en efectivo (Yape / Plin 4.40 no cuenta) | + 10.00 |
| | `B001-00000004`: efectivo | + 5.00 |
| | `B001-00000007`: recibió 5.00 y dio vuelto 1.20 | + 3.80 |
| No cuentan | `B001-00000002` (tarjeta), `B001-00000006` (transferencia), `B001-00000008` (Yape / Plin) | 0.00 |
| No cuenta (anulada desde esta misma caja) | `B001-00000005` (se cobró 8.90 en efectivo y se devolvió en la misma caja) | 0.00 |
| No cuenta (en espera, anulada) | `115BF400` | 0.00 |
| Reembolsos en efectivo | Devolución de `B001-00000001`: nota de crédito `BC01-00000001` | − 4.50 |
| No cuenta (reembolso por tarjeta) | Devolución de `B001-00000002`: nota de crédito `BC01-00000002` | 0.00 |
| **Efectivo esperado** | 100.00 + 45.00 + 10.00 + 5.00 + 3.80 − 4.50 | **S/ 159.30** |

**La pantalla coincide.** Antes de cerrar, el Dashboard mostraba «Efectivo esperado S/ 159.30» (pasó por S/ 172.70 tras el capítulo 8, S/ 163.80 al anular `B001-00000005` y S/ 159.30 al devolver 4.50 en efectivo), y la sesión cerrada guardó **S/ 159.30** como «Monto Cierre Calculado (S/)».

> **El panel de cierre no te muestra el efectivo esperado.** Para saber cuánto debería haber antes de contar, mira el Dashboard (tarjeta «Caja» › «Efectivo esperado»). Cuenta el efectivo, y solo entonces declara.

### Pasos

1. Abre el detalle de tu sesión abierta («Detalle de Sesión», «Datos del registro»). Los datos están deshabilitados: Sucursal, Cajero, Fecha / Hora Apertura, Fecha / Hora Cierre («—»), Estado («Abierta»), Monto Apertura (S/), y «Monto Cierre Declarado (S/)», «Monto Cierre Calculado (S/)» y «Diferencia Arqueo (S/)» con «—».
2. Haz clic en **Cerrar Sesión** (arriba a la derecha de la tarjeta). Aparece el panel «Cerrar Sesión» con el campo **Monto Cierre Declarado (S/)**, empezando en 0, y los botones **Cancelar** y **Confirmar Cierre**.
3. Escribe el efectivo que contaste.
4. Haz clic en **Confirmar Cierre**. **No hay una segunda confirmación** y el cierre no se puede deshacer: la caja no se reabre.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Monto Cierre Declarado (S/) | El efectivo contado, con decimales, por ejemplo 159.00 | Sí | No puede ser negativo ni quedar vacío: «Ingresa el monto declarado al cierre» (aparece al confirmar con el campo vacío; se comprobó). **Cancelar** cierra el panel sin cerrar la caja |

### Qué pasa al terminar

Aparece el aviso «Sesión cerrada correctamente» y la pantalla vuelve a la lista de sesiones. **El sistema no te muestra el resultado del arqueo en ese momento**: para ver el declarado, el calculado y la diferencia, abre otra vez el detalle de la sesión.

### Resultado del cierre de la prueba

Se declararon **S/ 159.00** (S/ 0.30 menos de lo esperado):

| Dato en el detalle | Valor |
|---|---|
| Estado | **Cerrada** |
| Fecha / Hora Cierre | 03/11/2026 07:44 |
| Monto Apertura (S/) | 100.00 |
| Monto Cierre Declarado (S/) | 159.00 |
| Monto Cierre Calculado (S/) | 159.30 (el efectivo esperado, calculado por el sistema al cerrar) |
| Diferencia Arqueo (S/) | −0.30 (declarado menos calculado: **negativo = faltante**) |

Cómo se lee:

- **Diferencia = declarado − calculado.** Negativa: faltó efectivo. Positiva: sobró (se ve sin el signo «+»). La pantalla no dice «faltante» ni «sobrante»: solo muestra el número.
- **Estado «Cuadrada» solo si la diferencia es exactamente 0.00.** No hay tolerancia: una diferencia de un centavo deja la caja «Cerrada». El estado «Cuadrada» se muestra en la sección 8.1.
- **El cálculo se hace una sola vez, al cerrar.** Lo que ocurra después con las ventas de esa caja ya no cambia estos tres montos.
- **Una caja cerrada no se reabre** y no admite más ventas. En el detalle ya no hay botones, solo la flecha de volver.
- **Las ventas en espera no impiden cerrar.** Si hubiera ventas en espera, el cierre no avisa ni las bloquea; quedan pendientes y las puede cobrar luego cualquier usuario de la sucursal con su propia caja abierta.
- **Si el efectivo esperado fuera negativo** (por ejemplo, reembolsos en efectivo mayores que lo que entró), el sistema no deja cerrar la caja y muestra «Error al cerrar la sesión». **No verificado en la aplicación** (fuente: `producto/pantallas/punto-de-venta` §1.3, regla `RN-CAJ-016`).

### Errores que puedes ver

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Ingresa el monto declarado al cierre» | El campo está vacío o es negativo | Escribe el efectivo contado |
| «Solo el cajero que abrió la caja o un administrador puede cerrarla.» | La caja es de otra persona y no eres administrador. **No verificado** | Pide al dueño de la caja, o a un administrador, que la cierre |
| «Error al cerrar la sesión» | Mensaje general: la sesión ya estaba cerrada, o el efectivo esperado salió negativo. **No verificado** | Actualiza la lista y revisa el estado de la sesión |
| «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» | Dos cierres a la vez. **No verificado** | Actualiza y revisa |
| «Error al cargar la sesión de caja» | No se pudo abrir el detalle | Vuelve a la lista y ábrelo de nuevo |

### 8.1 Mostrar el estado «Cuadrada»: una segunda caja corta

Como la primera caja quedó «Cerrada» por 30 céntimos, se hizo una **segunda caja corta** para ver el estado «Cuadrada». Una vez cerrada la primera, el cajero ya podía abrir otra (una sola caja abierta a la vez).

1. **Abrir la caja.** **Sesiones de Caja** › **Nuevo**. La pantalla dice «La caja se abrirá a tu nombre, en tu sucursal activa y con la hora actual.», muestra tu nombre en **Cajero** (deshabilitado) y pide el **Monto Apertura (S/)**, que empieza en 0. Se escribió 20 y se hizo clic en **Aperturar**: aviso «Sesión aperturada correctamente» y la lista de sesiones, con la caja nueva «Abierta» (03/11/2026 07:45, 20.00).
2. **Una venta mínima en efectivo.** En **Ventas** › **Nueva Venta POS** se vendió 1 unidad de Ibuprofeno Demo 400 mg (lote `LOT-IBU-2027`) por S/ 0.40, pagada en efectivo con S/ 0.40 recibidos (sin vuelto): boleta `B001-00000009`.
3. **Calcular a mano.** Efectivo esperado = 20.00 + 0.40 = **S/ 20.40**. El Dashboard mostró «Efectivo esperado S/ 20.40».
4. **Cerrar con el monto exacto.** En el detalle de la sesión: **Cerrar Sesión**, **Monto Cierre Declarado (S/)** = 20.40, **Confirmar Cierre**: aviso «Sesión cerrada correctamente».

Resultado en el detalle de la sesión:

| Dato | Valor |
|---|---|
| Estado | **Cuadrada** |
| Fecha / Hora Cierre | 03/11/2026 07:47 |
| Monto Apertura (S/) | 20.00 |
| Monto Cierre Declarado (S/) | 20.40 |
| Monto Cierre Calculado (S/) | 20.40 |
| Diferencia Arqueo (S/) | 0.00 |

Con esto se confirma la regla: **«Cuadrada» solo cuando la diferencia es exactamente 0.00**; con cualquier otra diferencia, la caja queda «Cerrada».

### Dos avisos de la apertura (ya probados en el capítulo 8)

El campo **Monto Apertura (S/)** de **Sesiones de Caja › Nuevo** exige un número mayor o igual a 0; vacío o negativo muestra «Ingresa un monto válido». Si ya tienes una caja abierta, el sistema no te deja abrir otra: «Ya tienes una caja abierta. Ciérrala antes de abrir otra.» (comprobado en el [capítulo 8](capitulo-08-caja-y-venta.md); en este capítulo solo se abrió la segunda caja con la primera ya cerrada).

## 9. Lista y detalle de sesiones de caja

**Cómo llegar:** menú lateral › **PUNTO DE VENTA** › **Sesiones de Caja**. La pantalla se titula «Sesiones de Caja» y la tarjeta «Búsqueda de Sesiones de Caja» («Ingresa los criterios de búsqueda»). Como en Ventas, la tabla **no aparece hasta que pulsas Buscar**. Arriba a la derecha está el botón **Nuevo** (abre una caja nueva; exige `Sesiones de Caja › Aperturar caja`).

### Filtros

| Campo | Qué elegir | Obligatorio | Reglas |
|---|---|---|---|
| Estado | «— Todos —», «Abierta», «Cerrada» o «Cuadrada» | No | |
| Apertura desde | Fecha | No | Filtra por la fecha en que se **abrió** la caja, no por la de cierre |
| Apertura hasta | Fecha | No | Incluye todo el día |

No hay filtro por sucursal ni por cajero. **Limpiar** vacía los filtros y oculta la tabla.

### La tabla

Encabezado «Resultados (N)», de la más reciente a la más antigua, con las columnas «Sucursal» (es el enlace al detalle), «Cajero», «Apertura» (`dd/mm/aaaa hh:mm`), «Cierre» («—» si la caja sigue abierta), «Monto Apertura» y «Estado» («Abierta», «Cerrada» o «Cuadrada»). **La tabla no trae el monto declarado, el calculado ni la diferencia**: esos tres datos están en el detalle y en el Excel. Bajo la tabla: **Exportar** y el paginador (10, 25 o 50 por página).

Lo que se comprobó, con las dos cajas del cajero:

| Búsqueda | Resultado |
|---|---|
| Sin filtros | «Resultados (2)»: la caja de las 07:45 («Cuadrada», apertura 20.00) y la del 02/11/2026 19:17 («Cerrada», apertura 100.00) |
| Estado «Cuadrada» | «Resultados (1)»: la caja de 20.00 |
| Estado «Cerrada» | «Resultados (1)»: la caja de 100.00 |
| Estado «Abierta» | «Resultados (0)» y «Sin resultados» (ya no hay ninguna caja abierta) |
| Apertura desde 03/11/2026 | «Resultados (1)»: solo la caja abierta ese día (la otra se abrió el 02/11/2026) |

### El detalle de una sesión

Haz clic en el nombre de la sucursal. La pantalla «Detalle de Sesión» (tarjeta «Datos del registro») es de **solo lectura** y muestra nueve datos:

| Dato | Caja 1 (cerrada con diferencia) | Caja 2 (cuadrada) |
|---|---|---|
| Sucursal | Sede Principal | Sede Principal |
| Cajero | Carlos Cajero | Carlos Cajero |
| Fecha / Hora Apertura | 02/11/2026 19:17 | 03/11/2026 07:45 |
| Fecha / Hora Cierre | 03/11/2026 07:44 | 03/11/2026 07:47 |
| Estado | Cerrada | Cuadrada |
| Monto Apertura (S/) | 100.00 | 20.00 |
| Monto Cierre Declarado (S/) | 159.00 | 20.40 |
| Monto Cierre Calculado (S/) | 159.30 | 20.40 |
| Diferencia Arqueo (S/) | −0.30 | 0.00 |

En una caja **abierta** los tres últimos datos (declarado, calculado y diferencia) y la fecha de cierre muestran «—»; solo en ella aparece el botón **Cerrar Sesión** (sección 8). En una caja cerrada o cuadrada solo queda la flecha de volver.

### Exportar a Excel

Con la tabla a la vista, haz clic en **Exportar**: se descarga `sesiones-caja.xlsx` con los filtros puestos. La hoja trae **nueve columnas**: «Sucursal», «Cajero», «Apertura», «Cierre», «Monto Apertura», «Monto Cierre Declarado (S/)», «Monto Cierre Calculado (S/)», «Diferencia Arqueo (S/)» y «Estado». Con la exportación sin filtros salieron dos filas: la caja de 20.00 con 20.4, 20.4 y 0, «Cuadrada»; y la de 100.00 con 159, 159.3 y −0.3, «Cerrada».

> **Las horas del Excel salen en hora universal (UTC), 5 horas por delante de Lima.** En la pantalla la caja se abrió el 03/11/2026 a las 07:45; en el Excel dice «03/11/2026 12:45». La caja que se abrió el 02/11/2026 a las 19:17 sale con «03/11/2026 00:17». Resta 5 horas al leer el Excel (regla `RN-CAJ-024`; ver sección 11).

### Errores que puedes ver

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Error al buscar sesiones de caja» | Falló la búsqueda o la exportación | Vuelve a pulsar **Buscar** |
| «Error al cargar la sesión de caja» | No se pudo abrir el detalle | Vuelve a la lista y ábrelo de nuevo |
| Redirección al Dashboard con «No tienes permisos para acceder a esta pantalla.» | Falta `Sesiones de Caja › Leer` | Pide el permiso |

## 10. Lo que ve un cajero frente a un administrador

Se recorrieron las mismas pantallas con el cajero `carlos.cajero` (rol «Cajero») y con el administrador `ana.manual` (rol «Admin»).

| Tema | Cajero | Administrador |
|---|---|---|
| Menú del grupo **PUNTO DE VENTA** | **Sesiones de Caja** y **Ventas** | Lo mismo, y además el menú completo de administración (Empresa, Sucursales, Roles & Permisos, Cuentas de Usuario, Series Fiscales, etc.) |
| **Ventas**: qué ventas aparecen | Las de la sucursal, de **todos** los empleados | **Las mismas**: ambos vieron «Resultados (10)» con las ventas de todos los cajeros. Para ver solo las propias se filtra por **Empleado** |
| **Ventas**: filtros, tabla y **Exportar** | Iguales | Iguales |
| Detalle de venta «Completada» | **Reimprimir Comprobante**, **Devolver** y **Anular Venta** (porque su rol tiene `Ventas › Anular` y `Devoluciones › Procesar devolución`) | Los mismos tres botones (el rol Admin pasa siempre) |
| Detalle de venta con devoluciones parciales (`B001-00000001`) | **Reimprimir Comprobante** y **Devolver** | Igual |
| Detalle de venta «Devuelta» (`B001-00000002`) | Solo **Reimprimir Comprobante** | Igual |
| Detalle de venta «Anulada» (`B001-00000005`) o en espera anulada (`115BF400`) | Sin botones | Igual |
| **Anular** una venta cobrada | Funciona, porque tiene **su propia caja abierta** | **Falla sin caja propia**: «Necesitas tener tu propia caja abierta en esta sucursal para anular una venta cobrada.» Ser administrador no evita esta regla |
| **Devolver** | Funciona con su caja abierta | **Falla sin caja propia**: «Necesitas una caja abierta para procesar la devolución.» |
| Anular una venta en espera | No pide caja | No pide caja |
| **Sesiones de Caja**: qué cajas aparecen | Las de las sucursales a las que tiene acceso, de todos los cajeros | Las de todas las sucursales de la empresa. Con una sola sucursal con cajas, vieron lo mismo |
| **Nuevo** (abrir caja) y **Cerrar Sesión** | Los ve (tiene `Aperturar caja` y `Cerrar caja`); solo puede cerrar la caja que él abrió | Los ve y puede cerrar la caja de cualquier cajero. **No verificado en la aplicación:** el cierre de la caja de otra persona (el administrador vio el botón **Cerrar Sesión** en la caja abierta del cajero, pero no lo confirmó) |
| Dashboard: tarjeta «Caja» | «Tienes una caja abierta» con sucursal, apertura, monto de apertura y «Efectivo esperado», y el enlace «Ver mi caja» | «No tienes una caja abierta» y el botón «Abrir caja» (el administrador no abrió caja) |
| Dashboard: ventas | «Vendido hoy», «Mis ventas de hoy», «Ventas en espera» y el cobro por método de la **sucursal** | Además la tabla «Ventas por sucursal» («Sede Principal», «Sucursal Demo Norte» y «Total empresa») |

Dos consecuencias prácticas:

- **El administrador no puede anular ni devolver «porque es administrador»**: necesita abrir su propia caja (capítulo 8), y entonces el efectivo de esa anulación o reembolso se cuenta en **su** arqueo.
- **El cajero y el administrador ven las mismas ventas y las mismas cajas** de la sucursal. Lo que cambia es qué puede hacer cada uno y el resumen del Dashboard.

## 11. Diferencias con la documentación técnica

No hubo contradicciones: la aplicación se comportó como describen el inventario `producto/pantallas/punto-de-venta.md` y las reglas `negocio/reglas/ventas-y-comprobantes` y `negocio/reglas/caja-devoluciones-y-dashboard`. Lo que se **confirmó en la aplicación** y conviene tener presente:

| Tema | Lo que dice la documentación | Lo que hizo la aplicación |
|---|---|---|
| Fecha del Excel de ventas (`producto/pantallas/punto-de-venta` §6, `RN-VEN-078`) | El Excel da formato a la fecha en UTC sin convertir a hora local | Confirmado: ventas del 02/11/2026 a las 19:24 y 19:25 salieron con «03/11/2026» |
| Hora del Excel de sesiones (`producto/pantallas/punto-de-venta` §1.1, `RN-CAJ-024`) | La apertura y el cierre salen en UTC | Confirmado: 07:45 en pantalla salió «03/11/2026 12:45» en el Excel |
| Resultado del cierre (`producto/pantallas/punto-de-venta` §1.3) | El SPA no muestra el arqueo al cerrar | Confirmado: tras «Sesión cerrada correctamente» vuelve a la lista; el resultado se ve abriendo el detalle |
| Efectivo esperado antes de cerrar (`producto/pantallas/punto-de-venta` §1.3) | El panel de cierre no lo muestra; el tablero sí | Confirmado |
| Cantidad devuelta mayor a lo vendido (`producto/pantallas/punto-de-venta` §9) | El campo se ajusta entre 0 y lo devolvible | Confirmado (200 pasó a 100); por eso el mensaje «La cantidad supera lo que queda por devolver…» no se ve desde un cuadro actualizado |
| «Cuarentena DIGEMID» (`producto/pantallas/punto-de-venta` §9, `RN-DEV-024`) | Solo anota el destino, no repone stock ni crea cuarentena | Confirmado: stock del lote sin cambio y «Lotes en cuarentena 0» |
| Texto de «Esta venta tiene devoluciones…» (`producto/pantallas/punto-de-venta` §8) | Remite a «Devoluciones», que no es opción de menú | Confirmado que no existe tal opción; además el botón **Anular Venta** desaparece antes de que se pueda ver ese mensaje |

## 12. Pendientes y «No verificado»

| # | Tema | Detalle |
|---|---|---|
| 1 | Imprimir el ticket | No se pulsó **Imprimir Comprobante** (abre el diálogo de impresión del sistema, que el navegador integrado no controla). Se leyó el contenido del ticket en el cuadro de reimpresión |
| 2 | Paginación de Ventas y de Sesiones | Solo hubo una página (10 ventas, 2 sesiones). Se comprobaron las opciones 10, 25 y 50 y que «Página 1 de 1» se mantiene. No se pasó a una segunda página |
| 3 | Filtro por empleado de Ventas | Se eligió «Carlos Cajero Demo» junto con otros filtros; no se probó solo ni con otro empleado distinto |
| 4 | Faltas de permiso | No se quitaron permisos a un rol: las condiciones de los botones salen del inventario `producto/pantallas/punto-de-venta` (§6 a §9) |
| 5 | Cerrar la caja de otra persona | No se intentó con un cajero ni se confirmó con el administrador (solo se vio el botón) |
| 6 | Efectivo esperado negativo al cerrar | No se provocó |
| 7 | Errores de devolución y anulación que la pantalla no deja provocar | «La cantidad supera lo que queda por devolver…», «Solo se pueden devolver ventas completadas.», «Solo puedes devolver ventas de tu propia sucursal.», «La venta no tiene boleta ni factura emitida que acreditar.», «La sucursal no tiene una serie de nota de crédito activa…», «La venta ya no existe.», «Una de las líneas ya no pertenece a la venta.», «Esta venta tiene devoluciones registradas y ya no puede anularse…» y los avisos de conflicto de concurrencia. Se documentan desde el inventario `producto/pantallas/punto-de-venta` |
| 8 | Límites de texto | No se escribió un motivo de anulación de más de 255 caracteres ni un sustento de más de 255 (el sustento tiene `maxlength` en el campo). Un monto declarado negativo no se probó (solo vacío) |
| 9 | Receta y dispensaciones | Las ventas anuladas y devueltas de esta prueba no usaron una receta registrada: la liberación de la dispensación al anular (`RN-VEN-068`) y su permanencia al devolver (`RN-DEV-026`) no se comprobaron |
| 10 | Cuarentena formal | La operación de cuarentena del módulo DIGEMID existe solo por API (sin pantalla) y no se usó |
| 11 | Cuenta del administrador con caja | No se abrió caja al administrador; por eso no se vio a un administrador anulando o devolviendo con caja propia |
| 12 | **En curso (sin commitear)** | Ninguno: ambos repositorios estaban sin cambios locales |

## 13. Lista de cobertura

Alcance: `producto/pantallas/punto-de-venta.md` §1 (sesiones: lista, detalle, cierre y arqueo), §6, §7, §8, §9 y la parte de reimpresión de §5.

| Elemento del inventario | Dónde quedó | Estado |
|---|---|---|
| §1.1 Pantalla `/sesiones-caja`: filtros Estado, Apertura desde, Apertura hasta | Sección 9 | Recorrido |
| §1.1 Botones **Nuevo**, **Limpiar**, **Buscar**, **Exportar** y enlace de la sucursal | Secciones 8.1 y 9 | Recorrido |
| §1.1 Tabla: Sucursal, Cajero, Apertura, Cierre, Monto Apertura, Estado | Sección 9 | Recorrido |
| §1.1 Exportación `sesiones-caja.xlsx` (9 columnas) y hora UTC | Sección 9 | Recorrido |
| §1.1 Mensaje «Error al buscar sesiones de caja» | Sección 9 | No verificado (no se provocó un fallo) |
| §1.2 Pantalla `/sesiones-caja/nueva` (apertura) | Sección 8.1 y capítulo 8 | Recorrido (abrir la 2ª caja) |
| §1.3 Datos de la sesión (9 campos) | Sección 9 | Recorrido |
| §1.3 Panel de cierre, campo «Monto Cierre Declarado (S/)», botones **Cerrar Sesión**, **Cancelar**, **Confirmar Cierre** | Sección 8 | Recorrido |
| §1.3 Mensajes: «Sesión cerrada correctamente», «Ingresa el monto declarado al cierre» | Sección 8 | Recorrido |
| §1.3 Mensajes: «Solo el cajero que abrió la caja o un administrador puede cerrarla.», «Error al cerrar la sesión», «Otra operación modificó estos datos…», «Error al cargar la sesión de caja» | Sección 8 | No verificado |
| §1.3 Estados «Cerrada» y «Cuadrada»; diferencia; fórmula del efectivo esperado | Secciones 8 y 8.1 | Recorrido |
| §6 Filtros Empleado, Cliente, Fecha desde, Fecha hasta, Estado | Sección 3 | Recorrido |
| §6 Tabla (7 columnas), paginación, **Limpiar**, **Buscar**, **Exportar**, `ventas.xlsx` | Sección 3 | Recorrido (segunda página: No verificado) |
| §6 Mensaje «Error al buscar ventas» | Sección 3 | No verificado |
| §7 Datos del detalle: cabecera, verificación de receta, comprobante, líneas, pagos, vuelto, devoluciones | Secciones 4 y 7 | Recorrido |
| §7 Botón **Reimprimir Comprobante** y su cuadro | Sección 4 | Recorrido (impresión: No verificado) |
| §7 Botones **Devolver** y **Anular Venta** y sus condiciones | Secciones 5, 6, 7 y 10 | Recorrido |
| §7 Botón **Cobrar Venta** | Capítulo 8 | No aplica en este capítulo |
| §7 Mensajes «Error al cargar la venta», «Venta anulada correctamente.» | Secciones 4 y 5 | Recorrido (el primero: No verificado) |
| §8 Diálogo de anulación: campo Motivo, **Cancelar**, **Anular Venta**, «El motivo es obligatorio.» | Sección 5 | Recorrido |
| §8 Errores: sin caja propia, con devoluciones, genérico, concurrencia | Sección 5 | Recorrido (sin caja y genérico); el resto No verificado |
| §9 Diálogo de devolución: tabla, **Devolver todo lo pendiente**, Destino, Motivo SUNAT, Método de reembolso, Sustento, Reembolso estimado | Sección 7 | Recorrido |
| §9 Botones **Cancelar** y **Procesar devolución**; éxito «Devolución procesada y nota de crédito emitida.» | Sección 7 | Recorrido |
| §9 Errores «Ingresa al menos una cantidad a devolver.», «El sustento es obligatorio.», «Necesitas una caja abierta para procesar la devolución.» | Sección 7 | Recorrido |
| §9 Resto de errores de devolución | Sección 7 | No verificado |
| §5 Reimpresión: título, subtítulo, Nº Comprobante, Estado SUNAT, Total Pagado, Medio de Pago, ticket | Sección 4 | Recorrido |
| Reglas `RN-VEN-062` a `RN-VEN-069`, `RN-CMP-012` (anulación) | Secciones 5 y 6 | Comprobadas en la aplicación (excepto lo de la tabla 12) |
| Reglas `RN-CAJ-011` a `RN-CAJ-018` (arqueo) y `RN-DEV-005` a `RN-DEV-026` (devolución) | Secciones 7, 8 y 8.1 | Comprobadas en la aplicación (excepto lo de la tabla 12) |

## 14. Fuentes

**Documentos:** [`producto/pantallas/punto-de-venta` — punto de venta](../producto/pantallas/punto-de-venta.md) (§1, §5, §6, §7, §8, §9); [`negocio/reglas/ventas-y-comprobantes` — ventas y comprobantes](../negocio/reglas/ventas-y-comprobantes.md) (`RN-VEN-062` a `RN-VEN-078`, `RN-CMP-012`); [`negocio/reglas/caja-devoluciones-y-dashboard` — caja, devoluciones y dashboard](../negocio/reglas/caja-devoluciones-y-dashboard.md) (`RN-CAJ-008` a `RN-CAJ-027`, `RN-DEV-002` a `RN-DEV-029`); [capítulo 8](capitulo-08-caja-y-venta.md).

**Recorridos hechos en la aplicación** (SPA `http://localhost:4200`, 03/11/2026):

| # | Pantalla y acción |
|---|---|
| 1 | Administrador: Ventas › Buscar; detalle `B001-00000005`; **Anular Venta** sin motivo y con motivo sin caja propia (rechazada); **Devolver**: sin cantidad, sin sustento y sin caja propia (rechazada); Sesiones de Caja › Buscar y detalle de la caja abierta |
| 2 | Cajero: Dashboard (efectivo esperado 172.70); Ventas: filtros Estado, Cliente, Empleado, fechas, **Limpiar**, paginación y **Exportar** (lectura del Excel) |
| 3 | Cajero: detalle `B001-00000003` y **Reimprimir Comprobante** (lectura del cuadro y del ticket) |
| 4 | Cajero: **Anular Venta** de `B001-00000005` (motivo vacío, **Cancelar**, solo espacios y motivo válido); revisión de Stock por Sucursal y Dashboard (163.80) |
| 5 | Cajero: anulación de la venta en espera `115BF400`; revisión de stock y Dashboard |
| 6 | Cajero: **Devolver** en `B001-00000001` (cantidad 200 ajustada a 100; devolución parcial de 10 unidades en efectivo) y en `B001-00000002` (devolución total con «Cuarentena DIGEMID» y tarjeta; sustento de solo espacios sin efecto); revisión de stock y Dashboard |
| 7 | Cajero: Sesiones de Caja › detalle › **Cerrar Sesión** (monto vacío, **Cancelar** y cierre con 159.00) |
| 8 | Cajero: Sesiones de Caja › **Nuevo** (apertura de 20.00), venta mínima en el terminal (`B001-00000009`) y cierre con 20.40 («Cuadrada») |
| 9 | Cajero: Sesiones de Caja: filtros Estado y Apertura desde, detalle de ambas cajas y **Exportar** (lectura del Excel) |
| 10 | Administrador: Dashboard, menú, Ventas (10 resultados y botones de cuatro estados distintos) |


