# Revisión de consistencia

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-11-04 |
| **Backend** | `sofia-apis`, rama `main`, commit `9c16f35` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `7070e1a` |
| **Método** | Lectura de código y de los documentos publicados; `git log`, `git show` y `git diff` de solo lectura; scripts de comprobación sobre todos los documentos. Sin navegador y sin API. |

## 1. Alcance

Pasada de cierre sobre la documentación publicada: (A) corregir inconsistencias conocidas entre documentos, (B) reconciliar los commits de código posteriores a la línea base (`sofia-apis` `9c16f35`; `sofia-web-spa` `37d053d` y `7070e1a`) y (C) regenerar la matriz de trazabilidad.

## 2. Correcciones (A)

| # | Documento | Sección | Antes | Después | Evidencia |
|---|---|---|---|---|---|
| A1 | `03b` | §5 (tabla `Transferencias_Cab`) y §15 | «Nada impide en la base [...] la misma sucursal»; «Nada en el repositorio impide [...]» | La base no tiene `CHECK`, pero el dominio rechaza origen igual a destino con `Transferencia.SucursalDestinoId` | `sofia-apis/src/SOFIA.Domain/Entities/Transferencia.cs:47-50` |
| A2 | `11` | DT-096, §6 (R-14), conteo del resumen, pendiente 8, cobertura | R-14: `UnidadesMedidaController` «ya no» tiene la comprobación «ID mismatch» (13 resueltos + R-14 = 14) | R-14 eliminado (13 resueltos); DT-096 lista nueve controladores. **El hallazgo 17 de `01b` sigue vigente y no se tocó** (ver §3) | `sofia-apis/src/SOFIA.API/Controllers/UnidadesMedidaController.cs:48-51`; `git grep "ID mismatch" HEAD` lista nueve controladores; la comprobación viene del commit `ed9a464` y nunca se quitó |
| A3 | `01a` | §2 punto 4 | «Solo la usan el detalle de Empleados y el de Sucursales» | Empleados, Sucursales y Roles (desde `37d053d`); 13 componentes de detalle la importan en todo el SPA (eran 12 antes de `37d053d`) | `sofia-web-spa/src/app/pages/roles/roles-detail/roles-detail.component.ts:20,293,313,341`; búsqueda de `saveErrorKey` en `src/`: 13 importaciones y el método propio de `stock-detail.component.ts:222` |
| A4a | `14` | Proceso 1 (diagrama y paso 6) | «200 con token de acceso y cookie» | 201 | `sofia-apis/src/SOFIA.API/Controllers/EmpresasController.cs:31` (`StatusCode(201, ...)`) |
| A4b | `14` | §13.2 | «372 IDs citados explícitamente» | 373 (los 475 con rangos y los 789 definidos no cambian) | `cie-rn14.js` y `14-check.js`: 373 IDs distintos; ningún ID aparece solo en §13. No se pudo identificar el ID extra porque no hay historial del documento (no es un repositorio) |
| A5 | `19f` y `14` | `19f`: tabla de estados de §3, §3.1, tabla de condiciones del cobro, tabla de acciones (§3.11) y diálogo de apertura; `14`: proceso 6, paso 1 | Tarjeta «Estado de caja» | Tarjeta de aviso de caja requerida, sin título, con el texto «Debes aperturar la caja del turno antes de procesar una venta.»; la tarjeta con título «Caja» pertenece al Inicio | `sofia-web-spa/src/app/pages/pos/nueva-venta/nueva-venta.component.html:13-23`; `public/i18n/dashboard/es.json:46-47` (`caja.title`) |
| A6 | `06` | §6 (Precio de proveedor) y §10.4 punto 1 | `01b` y `03a` «dicen» que el registro no guarda nada; `01a` y `01b` desactualizados | Redactado en pasado: 01a, 01b y 03a ya se reconciliaron en la versión 1.1; el registro guarda y cierra el precio anterior | `RegistrarPrecioProveedorCommandHandler.cs:26-46`; historial de `01b` (RN-PRV-020, commit `606cb69`) |
| A7a | `19d` | §4.3 (precios) y pendientes | Citas a `ProveedoresController` «viejas» | **Sin cambios**: las nueve citas (`:29`, `:37`, `:44`, `:52`, `:69`, `:78-108`, `:110`, `:110-115`) coinciden con el controlador actual | `sofia-apis/src/SOFIA.API/Controllers/ProveedoresController.cs` (atributos en 29, 37, 44, 52, 69, 78, 110) |
| A7b | `19d` | §4.3 y pendientes | «Los guards de RN-PRV-021 a RN-PRV-024 siguen vigentes»; «01b dice que no guarda nada» | Los guards existen pero son inalcanzables desde el registro (el validador responde antes); solo `CerrarVigencia` repite el control de fechas; 01b ya está corregido | `HistorialPrecioProveedor.cs:30-63`, `:132-137`; 01b RN-PRV-021 a 024 (v1.1) |
| A10 | `05b` | §3.18 | «Endpoints: 10» | 11 | `ProveedoresController.cs:21,29,37,44,52,60,69,78,110,119,137` (11 acciones HTTP); la suma por módulo de 05b ya da los 232 del total |
| A11 | `01a` RN-IMP-013; `01b` (cabecera de §3.1, RN-UDM-017, RN-JER-022, RN-LAB-017, RN-ING-015, RN-MED-026, RN-PRV-017, RN-ASE-012, RN-DIG-013, RN-PAC-017, RN-PRO-012); `20` §2 y tabla de errores; `12a`, `12b`, `13a`, `13b`, `16a`, `16b` | Respuesta de la vista previa sin archivo | «Texto plano "Debe adjuntar un archivo Excel."» → sin el campo `file`: `ProblemDetails` de validación de modelo (`errors.file`); solo con un archivo de 0 bytes: ese texto plano | Los 12 controladores declaran `Previsualizar(IFormFile file, ...)` sin nulabilidad (p. ej. `UnidadesMedidaController.cs:79`); captura real en `04a` (apartado de carga masiva) y `04b` §6.2 punto 8 |
| A8 | `19a` | §3.3 (formato de cifras) | Pipes `number` y `date` «usan el `LOCALE_ID` por defecto (`en-US`)» | Solo `number` depende del idioma (sin `LOCALE_ID`: `1,234.50`); los pipes `date` llevan patrón explícito; cifras de mil o más: No verificado en la aplicación (el manual solo vio cifras menores de mil) | `provide-sofia-dates.ts:19` (único `registerLocaleData`); plantillas: `number: '1.2-2'` 42 veces, `date: 'dd/MM/yyyy'`, `'dd/MM/yyyy HH:mm'`, `'HH:mm'`; manual cap. 10 tabla de no verificados |
| A9a | `01d` | §4 a §7 (tablas de endpoints y tabla resumen) | `Transferencias/{id}` y variantes | `Transferencias/{id:guid}` en las cinco rutas; Delivery, Magistrales y Servicios ya coincidían con `{id}` | `TransferenciasController.cs:25,43,51,59` (`{id:guid}`); `DeliveryController.cs:23,39,47,61`, `MagistralesController.cs:36`, `ServiciosController.cs:51-101` (`{id}`) |
| A9b | `01d` (RN-SRV-005, RN-SRV-019, pendientes 1 y 10), `14`, `12c`, `16c` | RN-SRV-019 | «Con el script actual el guardado fallaría» / «No verificado en ejecución» | Se agrega la comprobación en vivo (agendar, listar y eliminar respondieron 200 en la base DEV, `04d` §7). Discrepancia **sin explicar**: ningún script toca `Servicios_Agenda` salvo `0001` (y `0005` solo le suma `TenantId`) y la API no migra el esquema; la causa queda **No verificado** | `0001_Initial_Schema.sql:753-768`; `0005_Empresa_MultiTenant.sql:59-75`; `ServicioAgendaConfiguration.cs:38-45`; `SofiaWebAppFactory.cs:90` (único `EnsureCreated`, pruebas) |
| A12a | Manual cap. 11 | §2.9, filas «cliente», «fecha de vencimiento» y «borrador de 12 horas» | «No verificado en la aplicación» | «Verificado en el capítulo 8» (§10 y §19 punto 3; §8.2; §16). Del borrador solo se comprobó que se restaura y que se borra al cobrar; las 12 horas y el borrado al cerrar sesión siguen **No verificado** | Cap. 8 §8.2, §10, §16, §19 |
| A12b | Manual cap. 11 | §2.9, fila «Subtotal más IGV» | «La pantalla redondea cada cifra por separado (hasta un centavo de diferencia)» como hecho | Se presenta como deducción del código y se anota que el capítulo 8 no reprodujo la diferencia (suma exacta en 8 ventas) | Cap. 8 §19 punto 2 |
| A12c | Manual cap. 11 | §6, filas 5 y 11 de Pendientes | «Los capítulos 6 a 10 [...] no estaban publicados»; fila 11 sin distinguir lo comprobado | Los capítulos 6 a 10 están publicados; la fila 11 separa lo comprobado de lo no comprobado. El cruce de las filas de 3.11 a 3.14 con los capítulos 8 y 9 queda pendiente (ver Pendientes) | Cap. 8 y 9 |
| A12d | Manual cap. 11 | §2.5 | «No verificado en la aplicación en este capítulo» (cierre de caja) | «Verificado en el capítulo 9, §8» (cierre, que no se muestra el arqueo al terminar, estado «Cuadrada»); fila «Falta el monto declarado» marcada como verificada | Cap. 9 §8 y §8.1 |
| A13 | Manual cap. 10 | §9 | «El capítulo 1 §4.5 no menciona la condición» de los botones | La §4.5 sí dice que se ocultan cuando ves más de una sucursal; no hay diferencia | `capitulo-01-primeros-pasos.md:193` |
| A14 | Manual cap. 5 | §2 tabla de permisos, §9.2, §9.3, §9.4, §10.6, §13 | «Seguros» como nombre del permiso y «el módulo de permisos se llama Seguros» | «Aseguradoras» (texto de pantalla); `Seguros` se conserva solo como clave técnica entre comillas inversas. Los capítulos 3 y 11 ya usaban «Aseguradoras» | `public/i18n/roles/es.json:81` (`modulo.Seguros` = «Aseguradoras»); `public/i18n/es.json:13,43`; `route-permissions.ts:16` |
| A15 | `16c` | §2.2 (cajero) y nota final | `Inventario:Leer`, `Aseguradoras:Leer` como si fueran claves técnicas | Claves técnicas `Inventarios:Leer` y `Seguros:Leer`; párrafo nuevo que explica clave técnica frente a texto de pantalla («Inventario», «Aseguradoras») | `LotesInventarioController.cs` (9 × `HasPermission("Inventarios", ...)`), `SegurosController.cs` (`HasPermission("Seguros", ...)`); `roles/es.json:69,81` |

## 3. Comprobado y no corregido

- **A2, `01b` hallazgo 17 (no aplicado).** El encargo lo daba por obsoleto. El código lo contradice: `UnidadesMedidaController.cs:48-51` mantiene `command = command with { Id = id }` seguido de `if (id != command.Id) return BadRequest("ID mismatch.")`, igual en `HEAD`. El hallazgo sigue siendo cierto, así que no se marcó como resuelto; lo que estaba mal era la fila R-14 de `11`, ya corregida.
- **A7a, citas a `ProveedoresController` en `19d` (no aplicado).** Las nueve citas del documento coinciden con el controlador actual (atributos en `:29`, `:37`, `:44`, `:52`, `:69`, `:78`, `:110`). No hay citas viejas que corregir; el commit `606cb69` ya estaba reflejado cuando se escribió. La cita equivalente de `05b` (`:54`, `:71`, `:30`) usa la línea del método y no la del atributo, que es su convención, y tampoco se tocó.
- **A9a, notación `{id}` en otros documentos (no aplicado).** `01d` quedó alineado con los controladores (`Transferencias` usa `{id:guid}`; `Delivery`, `Magistrales` y `Servicios` usan `{id}`). Las historias `12c`, los casos de uso `13c`, el proceso `14`, los casos de prueba `16c`, `04d` y `05b` escriben `Transferencias/{id}` como notación abreviada de ruta; no se cambió porque no describen la restricción del enrutador.
- **A9b, causa de que agendar funcione en la base de desarrollo (no explicada).** El código y los scripts no la explican (ver fila A9b); se dejó como **No verificado**.
- **A4b, el ID que explica 372 frente a 373 (no localizado).** `sofia-docs` no es un repositorio y el documento no guarda historial; ningún ID aparece solo en la sección 13.2. Se corrigió la cifra a 373, que es lo que cuenta el script.
- **A12 (alcance).** Solo se cruzaron con los capítulos 8 y 9 las filas que pedía el encargo; el resto de las filas «No verificado» de §3.11 a §3.14 del capítulo 11 no se volvió a cruzar uno por uno.
- **Citas de línea desplazadas por `37d053d` y `7070e1a` (no recalculadas).** En `19b`, `12a`, `13a`, `16a` y el manual quedan citas `.ts:NNN` de `roles-detail.component.ts` y `cuentas-detail.component.ts` anteriores a esos commits (se desplazaron unas 9 a 16 líneas) fuera de las tablas de mensajes que sí se actualizaron; `19b` lo avisa en la ficha de Roles.
- **`01b` hallazgo 17 (no aplicado)**: ver A2 más arriba.

## 4. Reconciliación por commit (B)

Los documentos publicados traen en su cabecera `43f5529`/`eeffbb6`, pero en el repositorio hay tres commits de backend posteriores que cambian lo documentado: `34089ec` (seed del catálogo DIGEMID y script `0005` en lotes), `9c16f35` (catálogo DIGEMID compartido) y, ya fuera del encargo explícito, los commits de infraestructura `d79d6fa` y `5fc2ebf` (ver Pendientes). Se reconcilió `34089ec` además de lo pedido porque invalidaba afirmaciones de `03a`, `03b`, `07`, `10` y `11` y desplazaba las citas a `0005`.

### `sofia-apis` `9c16f35` — «share the digemid catalog across companies as read-only»

Qué cambió (`git show 9c16f35`): la entidad `DigemidCatalogoProducto` implementa `IGlobalReadEntity` (`IGlobalReadEntity.cs:4`, `DigemidCatalogoProducto.cs:5`); las filas con `TenantId` nulo (`IsGlobal`, `:19`) las deja pasar el filtro de empresa a todas (`ApplicationDbContext.cs:109-113`); editar o eliminar una fila global responde 403 `DigemidCatalogo.Global.SoloLectura` (`UpdateDigemidProductoCommandHandler.cs:21-24`, `DeleteDigemidProductoCommandHandler.cs:20-23`, `DigemidCatalogoProducto.cs:16`, `:60-63`); `Create` acepta un `tenantId` opcional (`:32`, `:54`); más pruebas (`TenantIsolationIntegrationTests`, `DeleteDigemidProductoCommandHandlerTests`, `UpdateDigemidProductoCommandHandlerTests`, `DigemidCatalogoProductoTests`). No cambia el controlador ni ningún permiso (`05b` sin cambios por este commit).

| Documento | Qué se hizo |
|---|---|
| `01b` (RN-DIG) | Introducción y nota de la sección 13, RN-DIG-002/003/006/008/009, pendiente 5, tabla 17.1 y citas desplazadas; versión 1.2 con historial. |
| `03a`, `03b` | Tabla DIGEMID (visibilidad, columnas, citas), seed y scripts; versión 1.2 (`03a`) y 1.1 (`03b`). |
| `07`, `10`, `11` | Seed del `0004` (ya no depende de `SofiaDb`), riesgo de `10` §4.5 y DT-081 marcado resuelto (queda en la tabla de deuda con la marca «Resuelto»; el conteo de 82 elementos no se tocó). |
| `04b` (§5.1) | Notas de visibilidad del catálogo compartido y filas 403 `DigemidCatalogo.Global.SoloLectura` en `PUT` y `DELETE`; duplicados contra el compartido en `POST`; marcado «No verificado en vivo» (no se llamó a la API). Versión 1.1. |
| `12b` (HU-CAT-042 y HU-CAT-044), `13b` (CU-CAT-42, CU-CAT-44, carga masiva), `16b` (TC-CAT-190, TC-CAT-201, TC-CAT-202) | Se reescribió el escenario «Una empresa nueva tiene el catálogo vacío» (ahora «ve el catálogo compartido») y se agregaron los casos del 403 y del código repetido contra el catálogo compartido **dentro de escenarios y casos existentes**: no hay IDs ni escenarios nuevos. Versión 1.1. |
| `19c` (§5.1) | Nota de que la lista y el detalle incluyen el catálogo compartido y que la pantalla no distingue filas propias y compartidas (el DTO no trae empresa). Versión 1.1. |
| Manual cap. 4 | §3 (glosario de la tabla), §8.1 y fila 5 de las diferencias: catálogo compartido, con el resultado comprobado con `43f5529` conservado y el actual «No verificado en la aplicación». Versión 1.1. |
| `05b` | Sin cambios por este commit: el controlador y los permisos no cambiaron (`git show 9c16f35 --stat` no toca `SOFIA.API`). |
| `09a` | Archivos 193 → 195, pruebas 1 254 → 1 263 (9 métodos nuevos), DIGEMID 7/23 → 9/31, `DeleteDigemidProductoCommand` sale de la lista sin prueba (22 de 113). Versión 1.1. |
| `21` | Ver abajo (código nuevo `DigemidCatalogo.Global.SoloLectura`). |
### `sofia-web-spa` `37d053d` — «show specific messages for account, employee and role errors»

Qué cambió (`git show 37d053d`): Roles usa `saveErrorKey` con `SAVE_ERROR_KEYS` (`Rol.DuplicateName`, `Rol.PermisosPropios`, `Permiso.Duplicate`) y `DELETE_ERROR_KEYS` (`Rol.InUse`) (`roles-detail.component.ts:20-33`, `:293`, `:313`, `:341`); Cuentas agrega `Auth.DuplicateAccount` y `Auth.DuplicateUsername` a `ERROR_KEYS` y el alta pasa por el mismo mapa (`cuentas-detail.component.ts:29-30`, `:190`); Empleados agrega `Empleado.SucursalBase.AutoAsignacion` (`empleados-detail.component.ts:23`); seis claves i18n nuevas (`cuentas/es.json:64-65`, `empleados/es.json:15`, `roles/es.json:15-16,19`) y 7 specs.

### `sofia-web-spa` `7070e1a` — «lock the force password change switch when it is already required»

Qué cambió: en el detalle de la cuenta el interruptor «Forzar Cambio de Contraseña» refleja `requiereCambioClave` y queda deshabilitado con una nota (`cuentas-detail.component.ts:148-159`, `.html:234-245`, clave `cuentas.fields.forzarCambioClaveActivo`, `cuentas/es.json:34`); 4 specs.

| Documento | Qué se hizo |
|---|---|
| `01a` | RN-ROL-005/008/010/011, RN-CTA-006/007/011, RN-EPL-011, hallazgo 2 y sección 2 punto 4 (A3); versión 1.2 con historial. |
| `19b` | Tablas de mensajes de Empleados, Roles y Cuentas, campo «Forzar Cambio de Contraseña» y nota sobre citas; versión 1.2 con historial. |
| `12a`, `13a`, `16a`, `14` | Escenarios, excepciones y resultados esperados con los mensajes nuevos, dentro de historias, casos de uso y casos existentes (sin IDs nuevos). Versión 1.1 (`14`: ya en 1.1 por A). |
| Manual cap. 3 | Cada observación hecha con `eeffbb6` se conserva con su commit y se anota el aviso nuevo como «No verificado en la aplicación». Versión 1.1. |
| Manual cap. 11 | Seis filas nuevas de mensajes en §3.4, §3.5 y §3.6, seis entradas del índice y nota en §6. Versión 1.2. |
| `21` | Mapas de Roles, Cuentas y Empleados (29 mapas y 112 entradas en lugar de 28 y 106), textos de seis códigos que dejaron de ser «genéricos», quitados de §5.2, y la discrepancia 8; más el código `DigemidCatalogo.Global.SoloLectura` (348 códigos). Versión 1.1. |
| `09b` | Specs: Empleados 5 → 6, Roles 16 → 19, Cuentas 18 → 25; 492 → 503 casos `it(`. Versión 1.1. |
### Otros cambios que no vienen de un commit de código

- `06`, `11`, `13b`, `13c`: se quitaron rutas absolutas de la máquina; `04b`: ancla `#61-…` corregida; `02`, `03a`, `03b`, `01a`, `01b`, `01d`, `05b`, `20`: citas a `0005` desplazadas por `34089ec`.

## 5. Trazabilidad (C)

`ola-3-calidad/17-matriz-de-trazabilidad.md` se regeneró con los scripts `17-build.js` y los `17-*.js` que usa, con cambios mínimos en cuatro de ellos: `17-lib.js` suma los capítulos 7 a 10 a `MANUAL_DOCS`; `17-gen-lib.js` deja vacío `PEND_PREFIX` (ya no hay capítulos pendientes); `17-gen-a.js` y `17-gen-c.js` actualizan los textos que decían «capítulos 1 a 6 y 11», «Pendiente» y «372 IDs» y la cabecera (versión 1.1, 2026-11-04, `9c16f35` y `7070e1a`). El resultado se copió sobre el documento (2 634 líneas).

| Conteo | Versión 1.0 | Versión 1.1 | Explicación |
|---|---|---|---|
| Reglas de negocio | 789 | 789 | Sin cambio (ninguna regla nueva) |
| Historias de usuario | 192 | 192 | Sin cambio |
| Casos de uso | 179 | 179 | Sin cambio |
| Casos de prueba | 1 244 | 1 244 | Sin cambio (413 + 321 + 510) |
| Endpoints | 241 | 241 | Sin cambio |
| Reglas con capítulo del manual | 183 (23,2 %) | 319 (40,4 %) | Capítulos 7 a 10 |
| Reglas «Pendiente» en la columna Manual | 176 | 0 | Capítulos 7 a 10 publicados |
| Reglas con historia / caso de uso / caso de prueba / prueba automática / spec / proceso | 789 / 775 / 786 / 69 / 484 / 474 | igual | Sin cambio |

La comprobación interna del script (inventario frente a lo hallado) da 0 faltantes, 0 sobrantes y 0 duplicados en reglas, historias, casos de uso y endpoints.

## 6. Estado final de la documentación

Versión y línea base que declara la cabecera de cada documento al cierre (script `cie-state.js`; las columnas de commit son todos los hashes citados en las filas Backend y Frontend de la cabecera: el primero es el de la línea base original y los siguientes, los commits posteriores con los que se reconcilió una parte). Solo se listan documentos con cabecera de versión; el diseño conceptual `arquitectura-ia` y los archivos de `_inventario` no la tienen.

| Documento | Versión | Fecha | Backend | Frontend |
|---|---|---|---|---|
| `ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md` | 1.2 | 2026-11-04 | 43f5529, 34089ec | eeffbb6, 37d053d, 7070e1a |
| `ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md` | 1.2 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6 |
| `ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md` | 1.0 | 2026-10-08 | 9dd1705 | 063817c |
| `ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md` | 1.1 | 2026-10-14 | 43f5529 | eeffbb6 |
| `ola-1-fundamentos/01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md` | 1.0 | 2026-10-08 | 606cb69, 9dd1705 | 9cbbe58, 063817c |
| `ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md` | 1.1 | 2026-11-04 | 27597cd, 606cb69 | 01b8260 |
| `ola-1-fundamentos/02-arquitectura.md` | 1.2 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-1-fundamentos/03-modelo-de-datos/03a-maestros-y-seguridad.md` | 1.2 | 2026-11-04 | 43f5529, 34089ec, 9c16f35 | eeffbb6 |
| `ola-1-fundamentos/03-modelo-de-datos/03b-transaccionales-e-historial.md` | 1.1 | 2026-11-04 | 43f5529, 34089ec | eeffbb6 |
| `ola-1-fundamentos/04-referencia-api/04a-convenciones-y-administracion.md` | 1.0 | 2026-10-30 | 34089ec | 9ad15f0 |
| `ola-1-fundamentos/04-referencia-api/04b-catalogos-basicos.md` | 1.1 | 2026-11-04 | 34089ec, 9c16f35 | 9ad15f0 |
| `ola-1-fundamentos/04-referencia-api/04c-productos-proveedores-personas.md` | 1.0 | 2026-10-30 | 34089ec | 9ad15f0 |
| `ola-1-fundamentos/04-referencia-api/04d-inventario-recetas-operaciones.md` | 1.0 | 2026-10-30 | 34089ec | 9ad15f0 |
| `ola-1-fundamentos/04-referencia-api/04e-ventas-caja-devoluciones.md` | 1.0 | 2026-10-30 | 34089ec | 9ad15f0 |
| `ola-1-fundamentos/05-seguridad/05a-autenticacion-y-controles.md` | 1.0 | 2026-10-15 | 43f5529 | eeffbb6 |
| `ola-1-fundamentos/05-seguridad/05b-matriz-de-permisos.md` | 1.2 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-1-fundamentos/06-glosario.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-1-fundamentos/07-integraciones.md` | 1.2 | 2026-11-04 | 43f5529, 34089ec, 9c16f35 | eeffbb6 |
| `ola-1-fundamentos/08-guia-del-desarrollador.md` | 1.1 | 2026-10-14 | 43f5529 | eeffbb6 |
| `ola-1-fundamentos/09-pruebas/09a-backend.md` | 1.1 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6 |
| `ola-1-fundamentos/09-pruebas/09b-spa.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `ola-1-fundamentos/10-despliegue-y-operacion.md` | 1.1 | 2026-11-04 | 43f5529, 34089ec | eeffbb6 |
| `ola-1-fundamentos/11-estado-y-deuda-tecnica.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-1-fundamentos/19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md` | 1.2 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-1-fundamentos/19-inventario-de-pantallas/19b-administracion.md` | 1.2 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `ola-1-fundamentos/19-inventario-de-pantallas/19c-catalogos-1.md` | 1.1 | 2026-11-04 | 27597cd, 9c16f35 | 01b8260 |
| `ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-1-fundamentos/19-inventario-de-pantallas/19e-inventario-y-atencion.md` | 1.0 | 2026-10-13 | 43f5529 | eeffbb6 |
| `ola-1-fundamentos/19-inventario-de-pantallas/19f-punto-de-venta.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-1-fundamentos/20-carga-masiva-y-exportacion.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-1-fundamentos/21-catalogo-codigos-error.md` | 1.1 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6, 37d053d |
| `ola-2-funcional/12-historias-de-usuario/12a-acceso-y-administracion.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `ola-2-funcional/12-historias-de-usuario/12b-catalogos-inventario-atencion.md` | 1.1 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6 |
| `ola-2-funcional/12-historias-de-usuario/12c-punto-de-venta-y-caja.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-2-funcional/13-casos-de-uso/13a-acceso-y-administracion.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `ola-2-funcional/13-casos-de-uso/13b-catalogos-inventario-atencion.md` | 1.1 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6 |
| `ola-2-funcional/13-casos-de-uso/13c-punto-de-venta-y-caja.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-2-funcional/14-procesos-de-negocio.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-2-funcional/manual-usuario/capitulo-01-primeros-pasos.md` | 1.1 | 2026-10-14 | 43f5529 | eeffbb6 |
| `ola-2-funcional/manual-usuario/capitulo-02-empresa-sucursales-series.md` | 1.0 | 2026-10-14 | 43f5529 | eeffbb6 |
| `ola-2-funcional/manual-usuario/capitulo-03-empleados-roles-cuentas.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `ola-2-funcional/manual-usuario/capitulo-04-catalogos-basicos.md` | 1.1 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6 |
| `ola-2-funcional/manual-usuario/capitulo-05-productos-y-proveedores.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-2-funcional/manual-usuario/capitulo-06-inventario.md` | 1.0 | 2026-10-30 | 34089ec, 43f5529 | 9ad15f0, eeffbb6 |
| `ola-2-funcional/manual-usuario/capitulo-07-pacientes-y-recetas.md` | 1.0 | 2026-10-31 | 34089ec, 43f5529 | 9ad15f0, eeffbb6 |
| `ola-2-funcional/manual-usuario/capitulo-08-caja-y-venta.md` | 1.0 | 2026-11-02 | 9c16f35, 43f5529 | 7070e1a, eeffbb6 |
| `ola-2-funcional/manual-usuario/capitulo-09-ventas-devoluciones-y-cierre.md` | 1.0 | 2026-11-02 | 9c16f35, 43f5529 | 7070e1a, eeffbb6 |
| `ola-2-funcional/manual-usuario/capitulo-10-inicio-y-roles.md` | 1.1 | 2026-11-04 | 9c16f35, 43f5529 | 7070e1a, eeffbb6 |
| `ola-2-funcional/manual-usuario/capitulo-11-problemas-frecuentes.md` | 1.2 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `ola-2-funcional/manual-usuario/manual-de-usuario.md` | 1.0 | 2026-11-04 | 9c16f35 | 7070e1a |
| `ola-3-calidad/16-casos-de-prueba/16a-acceso-y-administracion.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `ola-3-calidad/16-casos-de-prueba/16b-catalogos-inventario-atencion.md` | 1.1 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6 |
| `ola-3-calidad/16-casos-de-prueba/16c-punto-de-venta-y-caja.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `ola-3-calidad/17-matriz-de-trazabilidad.md` | 1.1 | 2026-11-04 | 9c16f35 | 7070e1a |
| `ola-4-cierre/18-revision-de-consistencia.md` | 1.0 | 2026-11-04 | 9c16f35 | 7070e1a |

Total: 55 documentos con cabecera. Los capítulos 8, 9 y 10 del manual y su documento integrador ya se escribieron sobre `9c16f35` y `7070e1a`; los capítulos 3 y 11 y `09b` quedaron con la línea base original más la nota de reconciliación con `37d053d` y `7070e1a`.

## 7. Comprobaciones por script

Script `cie-checks.js`, de solo lectura, sobre los 65 archivos `.md` de `sofia-docs` (incluidos `_inventario` y `arquitectura-ia`), ejecutado al terminar las ediciones.

| Comprobación | Resultado | Detalle |
|---|---|---|
| Enlaces relativos (5 256 enlaces, de los que 3 113 llevan ancla) | 0 archivos inexistentes, 0 anclas inexistentes | Las anclas se calculan con la regla de GitHub: minúsculas, se quitan signos salvo guion y guion bajo, espacios a guiones y sufijo `-1`, `-2` en títulos repetidos. Se ignoran los bloques de código y el código en línea |
| Bloques de código sin cerrar | 0 | Número par de vallas ``` en los 65 archivos |
| IDs `RN` inexistentes | 0 | 789 reglas definidas |
| IDs `HU` inexistentes | 0 | 192 historias definidas |
| IDs `CU` inexistentes | 0 | 179 casos de uso definidos |
| IDs `TC` inexistentes | 0 | 1 244 casos de prueba definidos |
| Voseo | 0 en los documentos publicados | Dos coincidencias que no cuentan: `escribí` en `06-glosario.md:20` es primera persona del pretérito («la escribí»), y `REGLAS-AGENTES.md:66` cita «podés»/«tenés» como ejemplo de lo prohibido |
| Rutas absolutas de la máquina | 0 en los documentos publicados de `ola-1` a `ola-4` | Se quitaron 3 (`11` §Fuentes, `13b` y `13c`). Quedan 12 en los archivos de trabajo de `_inventario` (`API-AGENTES`, `MANUAL-AGENTES`, `PLANTILLA-INVENTARIO-PANTALLAS`, `QA-AGENTES`, `RECONCILIACION`, `REGLAS-AGENTES`; uno de ellos nombra la ruta de un archivo de credenciales) y 1 en `arquitectura-ia/Arquitectura-Conceptual-IA-SOFIA.md:21`; ver Pendientes |

Falsos positivos descartados: el script marca 4 IDs de la matriz (`RN-XXX-999`, `HU-ADM-999`, `CU-ADM-99`, `TC-ADM-999`); son los IDs inventados que la propia matriz usa para probar su detector (`17-matriz-de-trazabilidad.md:2429`).

Comprobaciones adicionales sobre las ediciones de esta pasada: los conteos de reglas, historias, casos de uso, casos de prueba y endpoints de la matriz no cambiaron (sección 5); tras el cambio de `0005` se buscaron las citas viejas y no quedan (`0005:54-70`, `:54-69`, `:81-83`, `:75-84`, `:37-39`).

## Pendientes y «No verificado»

**No verificado en la aplicación** (no se usó el navegador ni la API en esta pasada):

1. Todo lo que cambia con `sofia-apis` `9c16f35`: que una empresa nueva vea el catálogo DIGEMID compartido, el 403 `DigemidCatalogo.Global.SoloLectura` al editar o eliminar una fila compartida y el rechazo de códigos que ya existen en el catálogo compartido. Tampoco se consultó ninguna base para saber si las filas sembradas tienen `TenantId` nulo.
2. Los mensajes nuevos de `sofia-web-spa` `37d053d` (seis textos de roles, cuentas y empleados) y el interruptor bloqueado de `7070e1a`.
3. Que el seed `0004` (29 698 filas, `34089ec`) se ejecute sin errores en una base nueva.
4. Cifras de mil o más con separador de miles (A8).
5. La causa de que agendar un servicio funcione en la base de desarrollo (A9b) y qué hace una base creada solo con los scripts.

**En curso (sin commitear):** ninguno. `git status --short` no mostró cambios locales en `sofia-apis` (`9c16f35`) ni en `sofia-web-spa` (`7070e1a`) al terminar.

**Pendientes para el orquestador y el usuario:**

1. **README.md** (no se tocó): sus cifras ya no coinciden con los documentos: «785 reglas» (son 789), 193 clases y 1 254 pruebas del backend (ahora 195 y 1 263), «347 códigos de error» (ahora 348), versiones 1.0 de varios documentos que pasaron a 1.1 o 1.2, matriz «con pantallas, pruebas automatizadas, procesos y manual» con los capítulos 7 a 10 ya cruzados, y la sección «En preparación» (revisión final de consistencia, esta pasada).
2. **Commits de infraestructura sin reconciliar:** `sofia-apis` `d79d6fa` (flujo `.github/workflows/docker-publish.yml`, publicación de las imágenes de la API y de Presidio en ghcr) y `5fc2ebf` (`.gitattributes` con fin de línea CRLF). `10` (CI y despliegue) y `08` (guía del desarrollador) no los describen. Lo mismo para el flujo que publica la imagen del SPA (`9ad15f0`).
3. **Documentos con línea base anterior a `43f5529`:** `01c1`, `01c3` y `19e` conservan cabeceras de `9dd1705`/`606cb69`/`063817c`/`9cbbe58`; `01d` y `19c` de `27597cd`/`01b8260`. No se reconciliaron en esta pasada más allá de lo pedido.
4. **`21` §5.3** (códigos sin pantalla) no lista `DigemidCatalogo.Global.SoloLectura`; está en la tabla principal, en 3.18 y en el recuento, pero falta en esa lista.
5. **`11` DT-081** quedó marcado «Resuelto» dentro de la tabla de deuda: el resumen sigue diciendo 82 elementos y 13 hallazgos resueltos; conviene decidir si pasa a la sección 6.
6. **Rutas absolutas de la máquina** en archivos de trabajo (`_inventario/*-AGENTES.md`, `PLANTILLA-INVENTARIO-PANTALLAS.md`, `RECONCILIACION.md`, uno de ellos con la ruta de un archivo de credenciales) y en `arquitectura-ia/Arquitectura-Conceptual-IA-SOFIA.md:21`. No son documentos publicados de `ola-1` a `ola-4`, pero están en la carpeta; conviene excluirlos de la publicación o limpiarlos.
7. **Manual de usuario:** el capítulo 3 y el 11 describen mensajes nuevos sin haberlos visto en pantalla; conviene recorrerlos con `sofia-web-spa` `7070e1a`. Los capítulos 1, 2 y 6 y 7 no se tocaron.
8. **Pruebas:** `pruebas-backend.csv` y `pruebas-spa.csv` de `_inventario` no se regeneraron (193 y 75 filas); los totales de `09a` y `09b` se corrigieron a mano sobre el diff de los commits.
9. **`01c1`, `01c2`, `01c3` y `12c`/`13c`/`16c`**: no se revisaron contra cambios de `34089ec`, `9c16f35`, `37d053d` y `7070e1a` porque esos commits no tocan ventas, caja ni devoluciones (`git show --stat`).

## Tabla de cobertura

| Elemento del encargo | Dónde queda |
|---|---|
| A1 a A15 | Sección 2 (filas A1 a A15; A2 y A7a con matiz) y sección 3 |
| B `sofia-apis` `9c16f35` (`01b`, `03a`/`03b`, `04b`, `12b`, `13b`, `16b`, `19c`, manual cap. 4, `05b`, `09a`) | Sección 4; `05b` sin cambios por este commit |
| B `sofia-web-spa` `37d053d` y `7070e1a` (`01a`, `19b`, `12a`, `13a`, `16a`, manual cap. 3, cap. 11, `21`, `09b`) | Sección 4 |
| C Matriz de trazabilidad | Sección 5 |
| D Informe de cierre: correcciones, no corregido, reconciliación, estado final, comprobaciones, pendientes | Secciones 2 a 7 y Pendientes |

## Fuentes

- Código de `sofia-apis` (`9c16f35`): `src/SOFIA.API/Controllers/` (`UnidadesMedidaController`, `ProveedoresController`, `TransferenciasController`, `DeliveryController`, `MagistralesController`, `ServiciosController`, `EmpresasController`, `DIGEMIDController`), `src/SOFIA.Domain/Entities/` (`Transferencia`, `DigemidCatalogoProducto`, `HistorialPrecioProveedor`), `src/SOFIA.Domain/Common/IGlobalReadEntity.cs`, `src/SOFIA.Application/DIGEMID/`, `src/SOFIA.Infrastructure/Persistence/ApplicationDbContext.cs`, `src/SOFIA.DbUp/Scripts/` (`0001`, `0003`, `0004`, `0005`, `0015`, `0020`), pruebas de `9c16f35`; `git show 9c16f35`, `git show 34089ec`, `git log`.
- Código de `sofia-web-spa` (`7070e1a`): `src/app/pages/` (roles, seguridad/cuentas-detail, empleados, pos/nueva-venta), `src/app/shared/utils/save-error-key.util.ts`, `public/i18n/` (`dashboard`, `cuentas`, `empleados`, `roles`, `es.json`), `git show 37d053d`, `git show 7070e1a`.
- Documentos de `sofia-docs` revisados o corregidos: los de la sección 6 con fecha 2026-11-04; más los capítulos 8 y 9 del manual como evidencia para A12.
- Scripts del scratchpad de la sesión: `cie-rn14.js`, `cie-checks.js`, `cie-state.js` y los `17-*.js` de la matriz.
