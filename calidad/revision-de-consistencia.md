# Revisión de consistencia

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.1 |
| **Fecha** | 2026-11-21 |
| **Backend** | `sofia-apis`, rama `main`, commits `9c16f35` (primera ronda) y `0678e40` (segunda ronda) |
| **Frontend** | `sofia-web-spa`, rama `main`, commits `7070e1a` (primera ronda) y `37048ea` (segunda ronda) |
| **Método** | Lectura de código y de los documentos publicados; `git log`, `git show` y `git diff` de solo lectura; scripts de comprobación sobre todos los documentos. Sin navegador y sin API. |

## Historial de cambios

| Versión | Fecha | Cambio |
|---|---|---|
| 1.1 | 2026-11-21 | Se agrega la sección 8 con la segunda ronda: la revisión contra el código del 2026-11-09 al 2026-11-20. Las secciones 1 a 7 son las de la primera ronda (2026-11-04) y no se tocaron |
| 1.0 | 2026-11-04 | Primera ronda |

## 1. Alcance

Pasada de cierre sobre la documentación publicada: (A) corregir inconsistencias conocidas entre documentos, (B) reconciliar los commits de código posteriores a la línea base (`sofia-apis` `9c16f35`; `sofia-web-spa` `37d053d` y `7070e1a`) y (C) regenerar la matriz de trazabilidad.

## 2. Correcciones (A)

| # | Documento | Sección | Antes | Después | Evidencia |
|---|---|---|---|---|---|
| A1 | `tecnico/modelo-de-datos/transaccionales-e-historial` | §5 (tabla `Transferencias_Cab`) y §15 | «Nada impide en la base [...] la misma sucursal»; «Nada en el repositorio impide [...]» | La base no tiene `CHECK`, pero el dominio rechaza origen igual a destino con `Transferencia.SucursalDestinoId` | `sofia-apis/src/SOFIA.Domain/Entities/Transferencia.cs:47-50` |
| A2 | `desarrollo/estado-y-deuda-tecnica` | DT-096, §6 (R-14), conteo del resumen, pendiente 8, cobertura | R-14: `UnidadesMedidaController` «ya no» tiene la comprobación «ID mismatch» (13 resueltos + R-14 = 14) | R-14 eliminado (13 resueltos); DT-096 lista nueve controladores. **El hallazgo 17 de `negocio/reglas/catalogos` sigue vigente y no se tocó** (ver §3) | `sofia-apis/src/SOFIA.API/Controllers/UnidadesMedidaController.cs:48-51`; `git grep "ID mismatch" HEAD` lista nueve controladores; la comprobación viene del commit `ed9a464` y nunca se quitó |
| A3 | `negocio/reglas/administracion-y-seguridad` | §2 punto 4 | «Solo la usan el detalle de Empleados y el de Sucursales» | Empleados, Sucursales y Roles (desde `37d053d`); 13 componentes de detalle la importan en todo el SPA (eran 12 antes de `37d053d`) | `sofia-web-spa/src/app/pages/roles/roles-detail/roles-detail.component.ts:20,293,313,341`; búsqueda de `saveErrorKey` en `src/`: 13 importaciones y el método propio de `stock-detail.component.ts:222` |
| A4a | `negocio/procesos-de-negocio` | Proceso 1 (diagrama y paso 6) | «200 con token de acceso y cookie» | 201 | `sofia-apis/src/SOFIA.API/Controllers/EmpresasController.cs:31` (`StatusCode(201, ...)`) |
| A4b | `negocio/procesos-de-negocio` | §13.2 | «372 IDs citados explícitamente» | 373 (los 475 con rangos y los 789 definidos no cambian) | `cie-rn14.js` y `14-check.js`: 373 IDs distintos; ningún ID aparece solo en §13. No se pudo identificar el ID extra porque no hay historial del documento (no es un repositorio) |
| A5 | `producto/pantallas/punto-de-venta` y `negocio/procesos-de-negocio` | `producto/pantallas/punto-de-venta`: tabla de estados de §3, §3.1, tabla de condiciones del cobro, tabla de acciones (§3.11) y diálogo de apertura; `negocio/procesos-de-negocio`: proceso 6, paso 1 | Tarjeta «Estado de caja» | Tarjeta de aviso de caja requerida, sin título, con el texto «Debes aperturar la caja del turno antes de procesar una venta.»; la tarjeta con título «Caja» pertenece al Inicio | `sofia-web-spa/src/app/pages/pos/nueva-venta/nueva-venta.component.html:13-23`; `public/i18n/dashboard/es.json:46-47` (`caja.title`) |
| A6 | `negocio/glosario` | §6 (Precio de proveedor) y §10.4 punto 1 | `negocio/reglas/catalogos` y `tecnico/modelo-de-datos/maestros-y-seguridad` «dicen» que el registro no guarda nada; `negocio/reglas/administracion-y-seguridad` y `negocio/reglas/catalogos` desactualizados | Redactado en pasado: `negocio/reglas/administracion-y-seguridad`, `negocio/reglas/catalogos` y `tecnico/modelo-de-datos/maestros-y-seguridad` ya se reconciliaron en la versión 1.1; el registro guarda y cierra el precio anterior | `RegistrarPrecioProveedorCommandHandler.cs:26-46`; historial de `negocio/reglas/catalogos` (RN-PRV-020, commit `606cb69`) |
| A7a | `producto/pantallas/productos-proveedores-y-aseguradoras` | §4.3 (precios) y pendientes | Citas a `ProveedoresController` «viejas» | **Sin cambios**: las nueve citas (`:29`, `:37`, `:44`, `:52`, `:69`, `:78-108`, `:110`, `:110-115`) coinciden con el controlador actual | `sofia-apis/src/SOFIA.API/Controllers/ProveedoresController.cs` (atributos en 29, 37, 44, 52, 69, 78, 110) |
| A7b | `producto/pantallas/productos-proveedores-y-aseguradoras` | §4.3 y pendientes | «Los guards de RN-PRV-021 a RN-PRV-024 siguen vigentes»; «`negocio/reglas/catalogos` dice que no guarda nada» | Los guards existen pero son inalcanzables desde el registro (el validador responde antes); solo `CerrarVigencia` repite el control de fechas; `negocio/reglas/catalogos` ya está corregido | `HistorialPrecioProveedor.cs:30-63`, `:132-137`; `negocio/reglas/catalogos` RN-PRV-021 a 024 (v1.1) |
| A10 | `tecnico/seguridad/matriz-de-permisos` | §3.18 | «Endpoints: 10» | 11 | `ProveedoresController.cs:21,29,37,44,52,60,69,78,110,119,137` (11 acciones HTTP); la suma por módulo de `tecnico/seguridad/matriz-de-permisos` ya da los 232 del total |
| A11 | `negocio/reglas/administracion-y-seguridad` RN-IMP-013; `negocio/reglas/catalogos` (cabecera de §3.1, RN-UDM-017, RN-JER-022, RN-LAB-017, RN-ING-015, RN-MED-026, RN-PRV-017, RN-ASE-012, RN-DIG-013, RN-PAC-017, RN-PRO-012); `tecnico/carga-masiva-y-exportacion` §2 y tabla de errores; `producto/historias-de-usuario/acceso-y-administracion`, `producto/historias-de-usuario/catalogos-inventario-y-atencion`, `producto/casos-de-uso/acceso-y-administracion`, `producto/casos-de-uso/catalogos-inventario-y-atencion`, `calidad/casos-de-prueba/acceso-y-administracion`, `calidad/casos-de-prueba/catalogos-inventario-y-atencion` | Respuesta de la vista previa sin archivo | «Texto plano "Debe adjuntar un archivo Excel."» → sin el campo `file`: `ProblemDetails` de validación de modelo (`errors.file`); solo con un archivo de 0 bytes: ese texto plano | Los 12 controladores declaran `Previsualizar(IFormFile file, ...)` sin nulabilidad (p. ej. `UnidadesMedidaController.cs:79`); captura real en `tecnico/api/convenciones-y-administracion` (apartado de carga masiva) y `tecnico/api/catalogos-basicos` §6.2 punto 8 |
| A8 | `producto/pantallas/acceso-layout-y-compartidos` | §3.3 (formato de cifras) | Pipes `number` y `date` «usan el `LOCALE_ID` por defecto (`en-US`)» | Solo `number` depende del idioma (sin `LOCALE_ID`: `1,234.50`); los pipes `date` llevan patrón explícito; cifras de mil o más: No verificado en la aplicación (el manual solo vio cifras menores de mil) | `provide-sofia-dates.ts:19` (único `registerLocaleData`); plantillas: `number: '1.2-2'` 42 veces, `date: 'dd/MM/yyyy'`, `'dd/MM/yyyy HH:mm'`, `'HH:mm'`; manual cap. 10 tabla de no verificados |
| A9a | `negocio/reglas/modulos-solo-api` | §4 a §7 (tablas de endpoints y tabla resumen) | `Transferencias/{id}` y variantes | `Transferencias/{id:guid}` en las cinco rutas; Delivery, Magistrales y Servicios ya coincidían con `{id}` | `TransferenciasController.cs:25,43,51,59` (`{id:guid}`); `DeliveryController.cs:23,39,47,61`, `MagistralesController.cs:36`, `ServiciosController.cs:51-101` (`{id}`) |
| A9b | `negocio/reglas/modulos-solo-api` (RN-SRV-005, RN-SRV-019, pendientes 1 y 10), `negocio/procesos-de-negocio`, `producto/historias-de-usuario/punto-de-venta-y-caja`, `calidad/casos-de-prueba/punto-de-venta-y-caja` | RN-SRV-019 | «Con el script actual el guardado fallaría» / «No verificado en ejecución» | Se agrega la comprobación en vivo (agendar, listar y eliminar respondieron 200 en la base DEV, `tecnico/api/inventario-recetas-y-operaciones` §7). Discrepancia **sin explicar**: ningún script toca `Servicios_Agenda` salvo `0001` (y `0005` solo le suma `TenantId`) y la API no migra el esquema; la causa queda **No verificado** | `0001_Initial_Schema.sql:753-768`; `0005_Empresa_MultiTenant.sql:59-75`; `ServicioAgendaConfiguration.cs:38-45`; `SofiaWebAppFactory.cs:90` (único `EnsureCreated`, pruebas) |
| A12a | Manual cap. 11 | §2.9, filas «cliente», «fecha de vencimiento» y «borrador de 12 horas» | «No verificado en la aplicación» | «Verificado en el capítulo 8» (§10 y §19 punto 3; §8.2; §16). Del borrador solo se comprobó que se restaura y que se borra al cobrar; las 12 horas y el borrado al cerrar sesión siguen **No verificado** | Cap. 8 §8.2, §10, §16, §19 |
| A12b | Manual cap. 11 | §2.9, fila «Subtotal más IGV» | «La pantalla redondea cada cifra por separado (hasta un centavo de diferencia)» como hecho | Se presenta como deducción del código y se anota que el capítulo 8 no reprodujo la diferencia (suma exacta en 8 ventas) | Cap. 8 §19 punto 2 |
| A12c | Manual cap. 11 | §6, filas 5 y 11 de Pendientes | «Los capítulos 6 a 10 [...] no estaban publicados»; fila 11 sin distinguir lo comprobado | Los capítulos 6 a 10 están publicados; la fila 11 separa lo comprobado de lo no comprobado. El cruce de las filas de 3.11 a 3.14 con los capítulos 8 y 9 queda pendiente (ver Pendientes) | Cap. 8 y 9 |
| A12d | Manual cap. 11 | §2.5 | «No verificado en la aplicación en este capítulo» (cierre de caja) | «Verificado en el capítulo 9, §8» (cierre, que no se muestra el arqueo al terminar, estado «Cuadrada»); fila «Falta el monto declarado» marcada como verificada | Cap. 9 §8 y §8.1 |
| A13 | Manual cap. 10 | §9 | «El capítulo 1 §4.5 no menciona la condición» de los botones | La §4.5 sí dice que se ocultan cuando ves más de una sucursal; no hay diferencia | `manual-de-usuario/capitulo-01-primeros-pasos.md:193` |
| A14 | Manual cap. 5 | §2 tabla de permisos, §9.2, §9.3, §9.4, §10.6, §13 | «Seguros» como nombre del permiso y «el módulo de permisos se llama Seguros» | «Aseguradoras» (texto de pantalla); `Seguros` se conserva solo como clave técnica entre comillas inversas. Los capítulos 3 y 11 ya usaban «Aseguradoras» | `public/i18n/roles/es.json:81` (`modulo.Seguros` = «Aseguradoras»); `public/i18n/es.json:13,43`; `route-permissions.ts:16` |
| A15 | `calidad/casos-de-prueba/punto-de-venta-y-caja` | §2.2 (cajero) y nota final | `Inventario:Leer`, `Aseguradoras:Leer` como si fueran claves técnicas | Claves técnicas `Inventarios:Leer` y `Seguros:Leer`; párrafo nuevo que explica clave técnica frente a texto de pantalla («Inventario», «Aseguradoras») | `LotesInventarioController.cs` (9 × `HasPermission("Inventarios", ...)`), `SegurosController.cs` (`HasPermission("Seguros", ...)`); `roles/es.json:69,81` |

## 3. Comprobado y no corregido

- **A2, `negocio/reglas/catalogos` hallazgo 17 (no aplicado).** El encargo lo daba por obsoleto. El código lo contradice: `UnidadesMedidaController.cs:48-51` mantiene `command = command with { Id = id }` seguido de `if (id != command.Id) return BadRequest("ID mismatch.")`, igual en `HEAD`. El hallazgo sigue siendo cierto, así que no se marcó como resuelto; lo que estaba mal era la fila R-14 de `desarrollo/estado-y-deuda-tecnica`, ya corregida.
- **A7a, citas a `ProveedoresController` en `producto/pantallas/productos-proveedores-y-aseguradoras` (no aplicado).** Las nueve citas del documento coinciden con el controlador actual (atributos en `:29`, `:37`, `:44`, `:52`, `:69`, `:78`, `:110`). No hay citas viejas que corregir; el commit `606cb69` ya estaba reflejado cuando se escribió. La cita equivalente de `tecnico/seguridad/matriz-de-permisos` (`:54`, `:71`, `:30`) usa la línea del método y no la del atributo, que es su convención, y tampoco se tocó.
- **A9a, notación `{id}` en otros documentos (no aplicado).** `negocio/reglas/modulos-solo-api` quedó alineado con los controladores (`Transferencias` usa `{id:guid}`; `Delivery`, `Magistrales` y `Servicios` usan `{id}`). Las historias `producto/historias-de-usuario/punto-de-venta-y-caja`, los casos de uso `producto/casos-de-uso/punto-de-venta-y-caja`, el proceso `negocio/procesos-de-negocio`, los casos de prueba `calidad/casos-de-prueba/punto-de-venta-y-caja`, `tecnico/api/inventario-recetas-y-operaciones` y `tecnico/seguridad/matriz-de-permisos` escriben `Transferencias/{id}` como notación abreviada de ruta; no se cambió porque no describen la restricción del enrutador.
- **A9b, causa de que agendar funcione en la base de desarrollo (no explicada).** El código y los scripts no la explican (ver fila A9b); se dejó como **No verificado**.
- **A4b, el ID que explica 372 frente a 373 (no localizado).** `sofia-docs` no es un repositorio y el documento no guarda historial; ningún ID aparece solo en la sección 13.2. Se corrigió la cifra a 373, que es lo que cuenta el script.
- **A12 (alcance).** Solo se cruzaron con los capítulos 8 y 9 las filas que pedía el encargo; el resto de las filas «No verificado» de §3.11 a §3.14 del capítulo 11 no se volvió a cruzar uno por uno.
- **Citas de línea desplazadas por `37d053d` y `7070e1a` (no recalculadas).** En `producto/pantallas/administracion`, `producto/historias-de-usuario/acceso-y-administracion`, `producto/casos-de-uso/acceso-y-administracion`, `calidad/casos-de-prueba/acceso-y-administracion` y el manual quedan citas `.ts:NNN` de `roles-detail.component.ts` y `cuentas-detail.component.ts` anteriores a esos commits (se desplazaron unas 9 a 16 líneas) fuera de las tablas de mensajes que sí se actualizaron; `producto/pantallas/administracion` lo avisa en la ficha de Roles.
- **`negocio/reglas/catalogos` hallazgo 17 (no aplicado)**: ver A2 más arriba.

## 4. Reconciliación por commit (B)

Los documentos publicados traen en su cabecera `43f5529`/`eeffbb6`, pero en el repositorio hay tres commits de backend posteriores que cambian lo documentado: `34089ec` (seed del catálogo DIGEMID y script `0005` en lotes), `9c16f35` (catálogo DIGEMID compartido) y, ya fuera del encargo explícito, los commits de infraestructura `d79d6fa` y `5fc2ebf` (ver Pendientes). Se reconcilió `34089ec` además de lo pedido porque invalidaba afirmaciones de `tecnico/modelo-de-datos/maestros-y-seguridad`, `tecnico/modelo-de-datos/transaccionales-e-historial`, `tecnico/integraciones`, `desarrollo/despliegue-y-operacion` y `desarrollo/estado-y-deuda-tecnica` y desplazaba las citas a `0005`.

### `sofia-apis` `9c16f35` — «share the digemid catalog across companies as read-only»

Qué cambió (`git show 9c16f35`): la entidad `DigemidCatalogoProducto` implementa `IGlobalReadEntity` (`IGlobalReadEntity.cs:4`, `DigemidCatalogoProducto.cs:5`); las filas con `TenantId` nulo (`IsGlobal`, `:19`) las deja pasar el filtro de empresa a todas (`ApplicationDbContext.cs:109-113`); editar o eliminar una fila global responde 403 `DigemidCatalogo.Global.SoloLectura` (`UpdateDigemidProductoCommandHandler.cs:21-24`, `DeleteDigemidProductoCommandHandler.cs:20-23`, `DigemidCatalogoProducto.cs:16`, `:60-63`); `Create` acepta un `tenantId` opcional (`:32`, `:54`); más pruebas (`TenantIsolationIntegrationTests`, `DeleteDigemidProductoCommandHandlerTests`, `UpdateDigemidProductoCommandHandlerTests`, `DigemidCatalogoProductoTests`). No cambia el controlador ni ningún permiso (`tecnico/seguridad/matriz-de-permisos` sin cambios por este commit).

| Documento | Qué se hizo |
|---|---|
| `negocio/reglas/catalogos` (RN-DIG) | Introducción y nota de la sección 13, RN-DIG-002/003/006/008/009, pendiente 5, tabla 17.1 y citas desplazadas; versión 1.2 con historial. |
| `tecnico/modelo-de-datos/maestros-y-seguridad`, `tecnico/modelo-de-datos/transaccionales-e-historial` | Tabla DIGEMID (visibilidad, columnas, citas), seed y scripts; versión 1.2 (`tecnico/modelo-de-datos/maestros-y-seguridad`) y 1.1 (`tecnico/modelo-de-datos/transaccionales-e-historial`). |
| `tecnico/integraciones`, `desarrollo/despliegue-y-operacion`, `desarrollo/estado-y-deuda-tecnica` | Seed del `0004` (ya no depende de `SofiaDb`), riesgo de `desarrollo/despliegue-y-operacion` §4.5 y DT-081 marcado resuelto (queda en la tabla de deuda con la marca «Resuelto»; el conteo de 82 elementos no se tocó). |
| `tecnico/api/catalogos-basicos` (§5.1) | Notas de visibilidad del catálogo compartido y filas 403 `DigemidCatalogo.Global.SoloLectura` en `PUT` y `DELETE`; duplicados contra el compartido en `POST`; marcado «No verificado en vivo» (no se llamó a la API). Versión 1.1. |
| `producto/historias-de-usuario/catalogos-inventario-y-atencion` (HU-CAT-042 y HU-CAT-044), `producto/casos-de-uso/catalogos-inventario-y-atencion` (CU-CAT-42, CU-CAT-44, carga masiva), `calidad/casos-de-prueba/catalogos-inventario-y-atencion` (TC-CAT-190, TC-CAT-201, TC-CAT-202) | Se reescribió el escenario «Una empresa nueva tiene el catálogo vacío» (ahora «ve el catálogo compartido») y se agregaron los casos del 403 y del código repetido contra el catálogo compartido **dentro de escenarios y casos existentes**: no hay IDs ni escenarios nuevos. Versión 1.1. |
| `producto/pantallas/catalogos-basicos` (§5.1) | Nota de que la lista y el detalle incluyen el catálogo compartido y que la pantalla no distingue filas propias y compartidas (el DTO no trae empresa). Versión 1.1. |
| Manual cap. 4 | §3 (glosario de la tabla), §8.1 y fila 5 de las diferencias: catálogo compartido, con el resultado comprobado con `43f5529` conservado y el actual «No verificado en la aplicación». Versión 1.1. |
| `tecnico/seguridad/matriz-de-permisos` | Sin cambios por este commit: el controlador y los permisos no cambiaron (`git show 9c16f35 --stat` no toca `SOFIA.API`). |
| `calidad/pruebas-automatizadas/backend` | Archivos 193 → 195, pruebas 1 254 → 1 263 (9 métodos nuevos), DIGEMID 7/23 → 9/31, `DeleteDigemidProductoCommand` sale de la lista sin prueba (22 de 113). Versión 1.1. |
| `tecnico/catalogo-de-codigos-de-error` | Ver abajo (código nuevo `DigemidCatalogo.Global.SoloLectura`). |
### `sofia-web-spa` `37d053d` — «show specific messages for account, employee and role errors»

Qué cambió (`git show 37d053d`): Roles usa `saveErrorKey` con `SAVE_ERROR_KEYS` (`Rol.DuplicateName`, `Rol.PermisosPropios`, `Permiso.Duplicate`) y `DELETE_ERROR_KEYS` (`Rol.InUse`) (`roles-detail.component.ts:20-33`, `:293`, `:313`, `:341`); Cuentas agrega `Auth.DuplicateAccount` y `Auth.DuplicateUsername` a `ERROR_KEYS` y el alta pasa por el mismo mapa (`cuentas-detail.component.ts:29-30`, `:190`); Empleados agrega `Empleado.SucursalBase.AutoAsignacion` (`empleados-detail.component.ts:23`); seis claves i18n nuevas (`cuentas/es.json:64-65`, `empleados/es.json:15`, `roles/es.json:15-16,19`) y 7 specs.

### `sofia-web-spa` `7070e1a` — «lock the force password change switch when it is already required»

Qué cambió: en el detalle de la cuenta el interruptor «Forzar Cambio de Contraseña» refleja `requiereCambioClave` y queda deshabilitado con una nota (`cuentas-detail.component.ts:148-159`, `.html:234-245`, clave `cuentas.fields.forzarCambioClaveActivo`, `cuentas/es.json:34`); 4 specs.

| Documento | Qué se hizo |
|---|---|
| `negocio/reglas/administracion-y-seguridad` | RN-ROL-005/008/010/011, RN-CTA-006/007/011, RN-EPL-011, hallazgo 2 y sección 2 punto 4 (A3); versión 1.2 con historial. |
| `producto/pantallas/administracion` | Tablas de mensajes de Empleados, Roles y Cuentas, campo «Forzar Cambio de Contraseña» y nota sobre citas; versión 1.2 con historial. |
| `producto/historias-de-usuario/acceso-y-administracion`, `producto/casos-de-uso/acceso-y-administracion`, `calidad/casos-de-prueba/acceso-y-administracion`, `negocio/procesos-de-negocio` | Escenarios, excepciones y resultados esperados con los mensajes nuevos, dentro de historias, casos de uso y casos existentes (sin IDs nuevos). Versión 1.1 (`negocio/procesos-de-negocio`: ya en 1.1 por A). |
| Manual cap. 3 | Cada observación hecha con `eeffbb6` se conserva con su commit y se anota el aviso nuevo como «No verificado en la aplicación». Versión 1.1. |
| Manual cap. 11 | Seis filas nuevas de mensajes en §3.4, §3.5 y §3.6, seis entradas del índice y nota en §6. Versión 1.2. |
| `tecnico/catalogo-de-codigos-de-error` | Mapas de Roles, Cuentas y Empleados (29 mapas y 112 entradas en lugar de 28 y 106), textos de seis códigos que dejaron de ser «genéricos», quitados de §5.2, y la discrepancia 8; más el código `DigemidCatalogo.Global.SoloLectura` (348 códigos). Versión 1.1. |
| `calidad/pruebas-automatizadas/spa` | Specs: Empleados 5 → 6, Roles 16 → 19, Cuentas 18 → 25; 492 → 503 casos `it(`. Versión 1.1. |
### Otros cambios que no vienen de un commit de código

- `negocio/glosario`, `desarrollo/estado-y-deuda-tecnica`, `producto/casos-de-uso/catalogos-inventario-y-atencion`, `producto/casos-de-uso/punto-de-venta-y-caja`: se quitaron rutas absolutas de la máquina; `tecnico/api/catalogos-basicos`: ancla `#61-…` corregida; `tecnico/arquitectura`, `tecnico/modelo-de-datos/maestros-y-seguridad`, `tecnico/modelo-de-datos/transaccionales-e-historial`, `negocio/reglas/administracion-y-seguridad`, `negocio/reglas/catalogos`, `negocio/reglas/modulos-solo-api`, `tecnico/seguridad/matriz-de-permisos`, `tecnico/carga-masiva-y-exportacion`: citas a `0005` desplazadas por `34089ec`.

## 5. Trazabilidad (C)

`calidad/matriz-de-trazabilidad.md` se regeneró con los scripts `17-build.js` y los `17-*.js` que usa, con cambios mínimos en cuatro de ellos: `17-lib.js` suma los capítulos 7 a 10 a `MANUAL_DOCS`; `17-gen-lib.js` deja vacío `PEND_PREFIX` (ya no hay capítulos pendientes); `17-gen-a.js` y `17-gen-c.js` actualizan los textos que decían «capítulos 1 a 6 y 11», «Pendiente» y «372 IDs» y la cabecera (versión 1.1, 2026-11-04, `9c16f35` y `7070e1a`). El resultado se copió sobre el documento (2 634 líneas).

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
| `negocio/reglas/administracion-y-seguridad.md` | 1.2 | 2026-11-04 | 43f5529, 34089ec | eeffbb6, 37d053d, 7070e1a |
| `negocio/reglas/catalogos.md` | 1.2 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6 |
| `negocio/reglas/inventario-y-recetas.md` | 1.0 | 2026-10-08 | 9dd1705 | 063817c |
| `negocio/reglas/ventas-y-comprobantes.md` | 1.1 | 2026-10-14 | 43f5529 | eeffbb6 |
| `negocio/reglas/caja-devoluciones-y-dashboard.md` | 1.0 | 2026-10-08 | 606cb69, 9dd1705 | 9cbbe58, 063817c |
| `negocio/reglas/modulos-solo-api.md` | 1.1 | 2026-11-04 | 27597cd, 606cb69 | 01b8260 |
| `tecnico/arquitectura.md` | 1.2 | 2026-11-04 | 43f5529 | eeffbb6 |
| `tecnico/modelo-de-datos/maestros-y-seguridad.md` | 1.2 | 2026-11-04 | 43f5529, 34089ec, 9c16f35 | eeffbb6 |
| `tecnico/modelo-de-datos/transaccionales-e-historial.md` | 1.1 | 2026-11-04 | 43f5529, 34089ec | eeffbb6 |
| `tecnico/api/convenciones-y-administracion.md` | 1.0 | 2026-10-30 | 34089ec | 9ad15f0 |
| `tecnico/api/catalogos-basicos.md` | 1.1 | 2026-11-04 | 34089ec, 9c16f35 | 9ad15f0 |
| `tecnico/api/productos-proveedores-y-personas.md` | 1.0 | 2026-10-30 | 34089ec | 9ad15f0 |
| `tecnico/api/inventario-recetas-y-operaciones.md` | 1.0 | 2026-10-30 | 34089ec | 9ad15f0 |
| `tecnico/api/ventas-caja-y-devoluciones.md` | 1.0 | 2026-10-30 | 34089ec | 9ad15f0 |
| `tecnico/seguridad/autenticacion-y-controles.md` | 1.0 | 2026-10-15 | 43f5529 | eeffbb6 |
| `tecnico/seguridad/matriz-de-permisos.md` | 1.2 | 2026-11-04 | 43f5529 | eeffbb6 |
| `negocio/glosario.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `tecnico/integraciones.md` | 1.2 | 2026-11-04 | 43f5529, 34089ec, 9c16f35 | eeffbb6 |
| `desarrollo/guia-del-desarrollador.md` | 1.1 | 2026-10-14 | 43f5529 | eeffbb6 |
| `calidad/pruebas-automatizadas/backend.md` | 1.1 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6 |
| `calidad/pruebas-automatizadas/spa.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `desarrollo/despliegue-y-operacion.md` | 1.1 | 2026-11-04 | 43f5529, 34089ec | eeffbb6 |
| `desarrollo/estado-y-deuda-tecnica.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `producto/pantallas/acceso-layout-y-compartidos.md` | 1.2 | 2026-11-04 | 43f5529 | eeffbb6 |
| `producto/pantallas/administracion.md` | 1.2 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `producto/pantallas/catalogos-basicos.md` | 1.1 | 2026-11-04 | 27597cd, 9c16f35 | 01b8260 |
| `producto/pantallas/productos-proveedores-y-aseguradoras.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `producto/pantallas/inventario-y-atencion.md` | 1.0 | 2026-10-13 | 43f5529 | eeffbb6 |
| `producto/pantallas/punto-de-venta.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `tecnico/carga-masiva-y-exportacion.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `tecnico/catalogo-de-codigos-de-error.md` | 1.1 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6, 37d053d |
| `producto/historias-de-usuario/acceso-y-administracion.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `producto/historias-de-usuario/catalogos-inventario-y-atencion.md` | 1.1 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6 |
| `producto/historias-de-usuario/punto-de-venta-y-caja.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `producto/casos-de-uso/acceso-y-administracion.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `producto/casos-de-uso/catalogos-inventario-y-atencion.md` | 1.1 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6 |
| `producto/casos-de-uso/punto-de-venta-y-caja.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `negocio/procesos-de-negocio.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `manual-de-usuario/capitulo-01-primeros-pasos.md` | 1.1 | 2026-10-14 | 43f5529 | eeffbb6 |
| `manual-de-usuario/capitulo-02-empresa-sucursales-series.md` | 1.0 | 2026-10-14 | 43f5529 | eeffbb6 |
| `manual-de-usuario/capitulo-03-empleados-roles-cuentas.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `manual-de-usuario/capitulo-04-catalogos-basicos.md` | 1.1 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6 |
| `manual-de-usuario/capitulo-05-productos-y-proveedores.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `manual-de-usuario/capitulo-06-inventario.md` | 1.0 | 2026-10-30 | 34089ec, 43f5529 | 9ad15f0, eeffbb6 |
| `manual-de-usuario/capitulo-07-pacientes-y-recetas.md` | 1.0 | 2026-10-31 | 34089ec, 43f5529 | 9ad15f0, eeffbb6 |
| `manual-de-usuario/capitulo-08-caja-y-venta.md` | 1.0 | 2026-11-02 | 9c16f35, 43f5529 | 7070e1a, eeffbb6 |
| `manual-de-usuario/capitulo-09-ventas-devoluciones-y-cierre.md` | 1.0 | 2026-11-02 | 9c16f35, 43f5529 | 7070e1a, eeffbb6 |
| `manual-de-usuario/capitulo-10-inicio-y-roles.md` | 1.1 | 2026-11-04 | 9c16f35, 43f5529 | 7070e1a, eeffbb6 |
| `manual-de-usuario/capitulo-11-problemas-frecuentes.md` | 1.2 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `manual-de-usuario/README.md` | 1.0 | 2026-11-04 | 9c16f35 | 7070e1a |
| `calidad/casos-de-prueba/acceso-y-administracion.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6, 37d053d, 7070e1a |
| `calidad/casos-de-prueba/catalogos-inventario-y-atencion.md` | 1.1 | 2026-11-04 | 43f5529, 9c16f35 | eeffbb6 |
| `calidad/casos-de-prueba/punto-de-venta-y-caja.md` | 1.1 | 2026-11-04 | 43f5529 | eeffbb6 |
| `calidad/matriz-de-trazabilidad.md` | 1.1 | 2026-11-04 | 9c16f35 | 7070e1a |
| `calidad/revision-de-consistencia.md` | 1.0 | 2026-11-04 | 9c16f35 | 7070e1a |

Total: 55 documentos con cabecera. Los capítulos 8, 9 y 10 del manual y su documento integrador ya se escribieron sobre `9c16f35` y `7070e1a`; los capítulos 3 y 11 y `calidad/pruebas-automatizadas/spa` quedaron con la línea base original más la nota de reconciliación con `37d053d` y `7070e1a`.

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
| Voseo | 0 en los documentos publicados | Dos coincidencias que no cuentan: `escribí` en `negocio/glosario.md:20` es primera persona del pretérito («la escribí»), y `REGLAS-AGENTES.md:66` cita «podés»/«tenés» como ejemplo de lo prohibido |
| Rutas absolutas de la máquina | 0 en los documentos publicados de `ola-1` a `ola-4` | Se quitaron 3 (`desarrollo/estado-y-deuda-tecnica` §Fuentes, `producto/casos-de-uso/catalogos-inventario-y-atencion` y `producto/casos-de-uso/punto-de-venta-y-caja`). Quedan 12 en los archivos de trabajo de `_inventario` (`API-AGENTES`, `MANUAL-AGENTES`, `PLANTILLA-INVENTARIO-PANTALLAS`, `QA-AGENTES`, `RECONCILIACION`, `REGLAS-AGENTES`; uno de ellos nombra la ruta de un archivo de credenciales) y 1 en `arquitectura-ia/Arquitectura-Conceptual-IA-SOFIA.md:21`; ver Pendientes |

Falsos positivos descartados: el script marca 4 IDs de la matriz (`RN-XXX-999`, `HU-ADM-999`, `CU-ADM-99`, `TC-ADM-999`); son los IDs inventados que la propia matriz usa para probar su detector (`calidad/matriz-de-trazabilidad.md:2429`).

Comprobaciones adicionales sobre las ediciones de esta pasada: los conteos de reglas, historias, casos de uso, casos de prueba y endpoints de la matriz no cambiaron (sección 5); tras el cambio de `0005` se buscaron las citas viejas y no quedan (`0005:54-70`, `:54-69`, `:81-83`, `:75-84`, `:37-39`).

## 8. Segunda ronda: revisión contra el código (2026-11-09 al 2026-11-20)

Esta sección se agregó en la versión 1.1. Todos los documentos publicados se volvieron a revisar contra `sofia-apis` `0678e40` y `sofia-web-spa` `37048ea` (ambos repositorios sin cambios locales), en tres tramos: la reorganización de carpetas, la reconciliación de los commits nuevos y un cierre con comprobaciones globales. Sin navegador y sin llamar a la API; los scripts son de solo lectura.

### 8.1 Reorganización de carpetas (`35dceec`, 2026-11-09)

Los 54 documentos pasaron de las carpetas por ola (`ola-1-fundamentos`, `ola-2-funcional`, `ola-3-calidad`, `ola-4-cierre`) a carpetas por tema, sin prefijos numéricos:

| Antes | Ahora |
|---|---|
| `01-reglas-de-negocio`, glosario y procesos | `negocio/reglas`, `negocio/glosario`, `negocio/procesos-de-negocio` |
| `12-historias-de-usuario`, `13-casos-de-uso`, `19-inventario-de-pantallas` | `producto/historias-de-usuario`, `producto/casos-de-uso`, `producto/pantallas` |
| arquitectura, `03-modelo-de-datos`, `04-referencia-api`, `05-seguridad`, integraciones, carga masiva y códigos de error | `tecnico/` (`modelo-de-datos`, `api`, `seguridad`) |
| guía del desarrollador, despliegue y estado | `desarrollo/` |
| `manual-usuario` | `manual-de-usuario` |
| `09-pruebas`, `16-casos-de-prueba`, matriz y revisión | `calidad/` (`pruebas-automatizadas`, `casos-de-prueba`) |

Se reescribieron por script unos 2 950 enlaces y referencias por nombre (rutas, referencias numeradas del tipo «documento 04a» y títulos de apartados). Los IDs (`RN`, `HU`, `CU`, `TC`) no cambiaron. Las rutas viejas de los scripts `17-…` de la matriz dejaron de servir (ver 8.5).

### 8.2 Reconciliación de commits

| Commit | Fecha | Qué cambió | Dónde se reflejó |
|---|---|---|---|
| `9c16f35` (apis) | 2026-10-31 | Catálogo DIGEMID compartido entre empresas, de solo lectura | Primera ronda en reglas, producto, manual y casos de prueba; en la segunda, en desarrollo, pruebas automatizadas, modelo de datos y deuda técnica |
| `6f6b837` (apis) y `558ad35` (spa) | 2026-11-14 | Gestión de suscripciones por el operador de plataforma: vencimiento también para `Activo`, 4 endpoints de suscripción, 3 de plataforma, columna `Es_Admin_Plataforma`, tabla `Plataforma_Configuracion_Pago` (script `0029`), pantalla de configuración de pago, QR de pago en el login, aviso de vencimiento y sección «Suscripciones» del Inicio | Reglas nuevas (RN-GEN-042 y 043, RN-AUT-040, RN-CTA-021, RN-EMP-019 a 028, RN-DAS-025 y 026), proceso 10 de procesos, glosario, modelo de datos, seguridad, API, errores, pantallas, historias HU-ADM-051 a 057, casos de uso CU-ADM-38 a 43, casos de prueba TC-ADM-414 a 455, capítulo 12 del manual |
| `0678e40` (apis) y `37048ea` (spa) | 2026-11-14 | Listado de devoluciones con filtros, detalle con líneas, exportación a Excel (`POST Devoluciones/exportar`) y pantallas `/devoluciones` y `/devoluciones/:id` | RN-DEV-030 a 032 reescritas y RN-DEV-038 a 044 nuevas, API, matriz de permisos, carga masiva, errores, pantallas, HU-DEV-009 y 010, CU-DEV-09 y 10, TC-DEV-060 a 079, manual (capítulos 8, 9 y 11) |
| `7091c43`, `a71f480` | 2026-11-14 | Despliegue a UAT tras publicar las imágenes | Guía del desarrollador, despliegue y operación, deuda técnica (FP-19 parcial) |
| `9079e00` (spa) | 2026-11-14 | Solo una prueba (spec de pago) | Catálogo de pruebas del SPA |

### 8.3 Qué se revisó y con qué profundidad

«Mecánico» es lectura por script: citas `archivo:línea` (existe, rango y primera línea), códigos de error, HTTP, mensajes entre «» contra los `es.json`, endpoints, permisos, rutas y conteos. «Semántico» es leer el código y compararlo con lo que dice el documento, regla por regla o escenario por escenario.

| Carpeta | Documento | Profundidad |
|---|---|---|
| `negocio/reglas` | `administracion-y-seguridad` | Semántico regla por regla en las 203 reglas anteriores (ronda de 2026-11-09); las 14 nuevas se escribieron leyendo el código; citas desplazadas reubicadas por script |
| `negocio/reglas` | `catalogos`, `inventario-y-recetas`, `ventas-y-comprobantes`, `modulos-solo-api` | Mecánico: citas reubicadas por comparación de bloques de código entre commits y revisadas una a una; sin lectura semántica regla por regla |
| `negocio/reglas` | `caja-devoluciones-y-dashboard` | Mecánico, más lectura de código para RN-DEV-030 a 032, RN-DEV-038 a 044 y RN-DAS-025 y 026 |
| `negocio` | `procesos-de-negocio`, `glosario` | Solo lo que tocan los commits nuevos (proceso 10, devoluciones, vigencia, 3 términos nuevos); las demás 133 definiciones y los procesos 1 a 9 no se releyeron contra cada handler |
| `producto/pantallas` | acceso, administración y punto de venta | Lectura de código de lo nuevo (pago, suscripción, configuración de pago, devoluciones) y citas remapeadas con el diff de git (139) |
| `producto/pantallas` | catálogos, productos e inventario | Mecánico (citas); sin lectura semántica |
| `producto/historias-de-usuario`, `producto/casos-de-uso` | acceso y punto de venta | Lectura de lo nuevo y de los escenarios corregidos; el resto, mecánico |
| `producto/historias-de-usuario`, `producto/casos-de-uso` | catálogos, inventario y atención | Solo verificación mecánica (mensajes): los commits no los afectan |
| `tecnico/modelo-de-datos` | las dos partes | Mecánico completo del esquema: 51 tablas por columna, tipo, nulo y valor por defecto, 41 índices, 25 `CHECK`, 88 claves foráneas, 1 020 citas a scripts: 0 diferencias en lo anterior |
| `tecnico/api` | las cinco partes | Mecánico: método, ruta, permiso, límite y línea de cada ficha (249 endpoints); 101 cuerpos contra sus commands. La semántica de respuestas y errores ficha por ficha no se releyó en catálogos, productos, inventario ni ventas |
| `tecnico/seguridad` | autenticación y controles; matriz de permisos | Mecánico (más de 4 900 citas de `tecnico`, 105 pares de permisos); lectura de lo nuevo (operador, `IgnoreQueryFilters`, política `public-info`) |
| `tecnico` | códigos de error, arquitectura, integraciones, carga masiva | Recalculado por script (357 códigos, conteos, 39 claves de configuración); lectura de lo nuevo |
| `desarrollo` | guía del desarrollador, despliegue y operación | Mecánico (más de 500 citas, versiones, puertos, variables, scripts) y lectura de lo nuevo (imágenes, UAT, operador); la sección de convenciones se contrastó por la primera línea citada |
| `desarrollo` | estado y deuda técnica | Mecánico (unas 210 citas, 249 endpoints, conteos); DT-013 a DT-060 por la primera línea citada, no una por una |
| `manual-de-usuario` | capítulos 1 a 11 y README | Mecánico (unos 3 300 textos entre «» contra `es.json` y código); lectura de lo nuevo. Capítulos 5, 6 y 7 solo por muestreo; capítulo 2 (series), 4 (cargas masivas), 8 (secciones 15 a 17) y 9 (cierre) solo mensajes |
| `manual-de-usuario` | capítulo 12 (nuevo) | Escrito desde el código; todo «No verificado en la aplicación» |
| `calidad/pruebas-automatizadas` | backend y SPA | Mecánico: 204 archivos y 1 344 pruebas del backend, 83 specs y 584 casos del SPA, fila por fila; la columna «Qué verifica» no se releyó cuerpo a cuerpo |
| `calidad/casos-de-prueba` | las tres partes | Mecánico (referencias, mensajes, códigos, rutas, permisos) y casos nuevos (42 en la parte A, 20 en la C); no se releyó caso por caso la lógica de ventas, caja, stock, recetas y series |
| `calidad` | matriz de trazabilidad | Regenerada (ver 8.5) |
| `_inventario` | CSV y README | Regenerados desde los commits (249 endpoints, 105 permisos, 357 códigos, 204 archivos de prueba) |

### 8.4 Correcciones del cierre (2026-11-20 y 2026-11-21)

| # | Documento | Antes | Después | Evidencia |
|---|---|---|---|---|
| B1 | `negocio/reglas/caja-devoluciones-y-dashboard` | Las filas reescritas RN-DEV-030 a 032 y las nuevas RN-DEV-038 a 044 estaban dentro de la tabla de historial y el apartado 4.7 conservaba las filas viejas («Solo API») | Las filas nuevas están en 4.7 y las viejas se quitaron | Script de IDs: 812 definiciones sin repetidos |
| B2 | `producto/casos-de-uso/punto-de-venta-y-caja` y `producto/historias-de-usuario/punto-de-venta-y-caja` | «No existe un RN-DEV para filtros, alcance ni Excel»; HU-DEV-008 y 009 con reglas «de la versión anterior»; anclas al apartado 4.7 viejo | Citan RN-DEV-038 a 044; trazabilidad y anclas actualizadas; la afirmación «el listado y el detalle son Solo API» se quitó | `DevolucionesController.cs:15-55`, `GetDevolucionByIdQueryHandler.cs:16-32` (valor plano) |
| B3 | `tecnico/seguridad/autenticacion-y-controles` (9.2) | `GetDevolucionByIdQueryHandler.cs:15`, `GetDevolucionesQueryHandler.cs:17` | `:18-21` y `:19-20` | `ISucursalAccess.GetAllowedSucursalesAsync` en esas líneas |
| B4 | `tecnico/api/ventas-caja-y-devoluciones` | Anclas `#47-consultas-solo-api-sin-pantalla` | `#47-consultas-búsqueda-detalle-y-exportación` | Script de enlaces |
| B5 | Manual capítulos 1 y 10 | «24 opciones» del menú del administrador y 12 del cajero | 25 y 13 (la opción «Devoluciones» de `sidebar-data.ts:138`); la del cajero, No verificado | `sidebar-data.ts` |
| B6 | `producto/historias-de-usuario/acceso-y-administracion` y `calidad/casos-de-prueba/acceso-y-administracion` | Escenario «En estado Activo la fecha de vencimiento no se evalúa» (el cuerpo ya decía lo contrario) | «…también se evalúa» | `Empresa.cs:27-33` |
| B7 | `desarrollo/estado-y-deuda-tecnica`, `tecnico/carga-masiva-y-exportacion`, `negocio/procesos-de-negocio` (13.2), `calidad/casos-de-prueba` | Pendientes ya resueltos; 241 endpoints; 789 reglas; capítulo 5 «a medias»; `Devoluciones/{id}` con `Result` completo | Actualizados (249, 812, 395 y 497 IDs) | `_inventario/endpoints.csv` y scripts de 8.5 |
| B8 | Cinco documentos de `tecnico` y `producto/pantallas` | Cabeceras y apartados «En curso» que describían cambios locales de los repositorios | Sin referencias a cambios locales (el repositorio de documentación es público) | Búsqueda por script |

### 8.5 Matriz de trazabilidad y comprobaciones globales

Los scripts `17-…` ya no existían. Se reescribieron como `cie2-model.js` y `cie2-gen.js` con el mismo criterio de extracción y la matriz salió en la versión 1.2: 812 reglas, 201 historias, 187 casos de uso, 1 306 casos de prueba, 249 endpoints y el capítulo 12; la tabla 8.1 de la matriz cierra sin diferencias. La clasificación `BE` de RN-DEV-030 y 031 pasó a «Sí» (`DevolucionesConsultaIntegrationTests`). Quedó un hueco nuevo: RN-CTA-021 no la cita ningún caso de uso (HU-ADM-051 no tiene caso de uso).

Comprobaciones sobre todos los documentos publicados (todos menos `REGISTRO.md`, `_inventario/*-AGENTES.md`, plantillas, `RECONCILIACION.md` y `arquitectura-ia/`):

| Comprobación | Resultado |
|---|---|
| Enlaces y anclas (regla de GitHub) | 5 469 enlaces revisados, 0 rotos |
| IDs definidos | 812 RN, 201 HU, 187 CU, 1 306 TC, sin repetidos |
| IDs citados que no existen | 0 reales; los 4 de control (`RN-XXX-999`, `HU-ADM-999`, `CU-ADM-99`, `TC-ADM-999`) son los de 6.8 de la matriz y de 7 de este documento |
| Voseo | 0 (la forma «escribí» es primera persona del pretérito) |
| Rutas absolutas de la máquina | 0 |
| Bloques de código sin cerrar | 0 |
| Cambios locales sin commitear de los repositorios | Ninguna afirmación vigente; quedan los apartados «En curso (sin commitear)» con «ninguno» y filas de historial que recuerdan que un archivo ya está commiteado |

### 8.6 Hallazgos de código que la documentación dejó a la vista

| # | Hallazgo | Evidencia | Documento |
|---|---|---|---|
| H1 | El catálogo de permisos de roles ya no incluye `Empresas:Leer`: solo recorre `HasPermissionAttribute` y los tres endpoints de lectura de Empresas usan `HasPermissionOrPlataforma`; un rol que no sea `Admin` no puede recibir ese permiso desde la pantalla (deducción del código) | `RolesController.cs:110` | DT-097 |
| H2 | El Excel de devoluciones saca códigos sin traducir: el motivo SUNAT como `06` o `07` y el método de reembolso con el nombre interno | RN-DEV-041 | `negocio/reglas/caja-devoluciones-y-dashboard` |
| H3 | `Plataforma_Configuracion_Pago` no impide más de una fila (sin índice único ni `CHECK`) y el handler toma la más antigua | `0029_Plataforma_Suscripciones.sql:6-22`; `GuardarConfiguracionPagoCommandHandler.cs:22-25` | DT-098 |
| H4 | La marca de operador solo se cambia en la base de datos: ningún comando la asigna ni la audita | RN-CTA-021 | `negocio/reglas/administracion-y-seguridad` 12.3 |
| H5 | `GET plataforma/pago` es anónimo y devuelve el QR de Yape como `data:` URL (intencional) | RN-EMP-025 | ídem |
| H6 | El autocompletado «Autorizó» de la búsqueda de devoluciones llama a `GET Empleados` (exige `Empleados:Leer`) y el SPA no captura el error | RN-DEV-042 | `producto/historias-de-usuario/punto-de-venta-y-caja` (HU-DEV-009) |
| H7 | Los workflows de despliegue a UAT no corren pruebas; sigue sin haber una prueba por módulo de farmacia con un rol sin permiso que espere 403 | `docker-publish.yml`; `calidad/pruebas-automatizadas/backend` 6.6 | DT-070 |
| H8 | `_inventario/permisos.csv` y `endpoints.csv` no distinguían `PLATAFORMA_ADMIN`; se regeneraron con la columna `permiso` ampliada | `_inventario/README.md` | — |

### 8.7 «No verificado» y lo que no se revisó a fondo

- Todo lo nuevo (suscripciones, operador, devoluciones, capítulo 12) se escribió desde el código y no se recorrió en la aplicación.
- El contenido real del `devoluciones.xlsx`, más de 10 000 filas, el autocompletado sin `Empleados:Leer`, el `TenantId` de la fila de pago y la sustitución de `${API_BASE_URL}` en la imagen del SPA.
- La existencia de los recursos de UAT en Azure y el efecto del salto de línea CRLF en los hooks.
- Sin lectura semántica regla por regla: cuatro de los seis catálogos de reglas (8.3), procesos 1 a 9, glosario, fichas de API de catálogos, productos, inventario y ventas, pantallas de catálogos e inventario, capítulos 5, 6 y 7 del manual, y la lógica de ventas, caja, stock y recetas en los casos de prueba.
- La clasificación `BE` y `SPA` de la matriz es por contexto y por pantalla, no una lectura de las pruebas.
- Los números de línea citados son los de `0678e40` y `37048ea`; cualquier commit posterior los desplaza.

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
2. **Commits de infraestructura sin reconciliar:** `sofia-apis` `d79d6fa` (flujo `.github/workflows/docker-publish.yml`, publicación de las imágenes de la API y de Presidio en ghcr) y `5fc2ebf` (`.gitattributes` con fin de línea CRLF). `desarrollo/despliegue-y-operacion` (CI y despliegue) y `desarrollo/guia-del-desarrollador` (guía del desarrollador) no los describen. Lo mismo para el flujo que publica la imagen del SPA (`9ad15f0`).
3. **Documentos con línea base anterior a `43f5529`:** `negocio/reglas/inventario-y-recetas`, `negocio/reglas/caja-devoluciones-y-dashboard` y `producto/pantallas/inventario-y-atencion` conservan cabeceras de `9dd1705`/`606cb69`/`063817c`/`9cbbe58`; `negocio/reglas/modulos-solo-api` y `producto/pantallas/catalogos-basicos` de `27597cd`/`01b8260`. No se reconciliaron en esta pasada más allá de lo pedido.
4. **`tecnico/catalogo-de-codigos-de-error` §5.3** (códigos sin pantalla) no lista `DigemidCatalogo.Global.SoloLectura`; está en la tabla principal, en 3.18 y en el recuento, pero falta en esa lista.
5. **`desarrollo/estado-y-deuda-tecnica` DT-081** quedó marcado «Resuelto» dentro de la tabla de deuda: el resumen sigue diciendo 82 elementos y 13 hallazgos resueltos; conviene decidir si pasa a la sección 6.
6. **Rutas absolutas de la máquina** en archivos de trabajo (`_inventario/*-AGENTES.md`, `PLANTILLA-INVENTARIO-PANTALLAS.md`, `RECONCILIACION.md`, uno de ellos con la ruta de un archivo de credenciales) y en `arquitectura-ia/Arquitectura-Conceptual-IA-SOFIA.md:21`. No son documentos publicados de `ola-1` a `ola-4`, pero están en la carpeta; conviene excluirlos de la publicación o limpiarlos.
7. **Manual de usuario:** el capítulo 3 y el 11 describen mensajes nuevos sin haberlos visto en pantalla; conviene recorrerlos con `sofia-web-spa` `7070e1a`. Los capítulos 1, 2 y 6 y 7 no se tocaron.
8. **Pruebas:** `pruebas-backend.csv` y `pruebas-spa.csv` de `_inventario` no se regeneraron (193 y 75 filas); los totales de `calidad/pruebas-automatizadas/backend` y `calidad/pruebas-automatizadas/spa` se corrigieron a mano sobre el diff de los commits.
9. **`negocio/reglas/inventario-y-recetas`, `negocio/reglas/ventas-y-comprobantes`, `negocio/reglas/caja-devoluciones-y-dashboard` y `producto/historias-de-usuario/punto-de-venta-y-caja`/`producto/casos-de-uso/punto-de-venta-y-caja`/`calidad/casos-de-prueba/punto-de-venta-y-caja`**: no se revisaron contra cambios de `34089ec`, `9c16f35`, `37d053d` y `7070e1a` porque esos commits no tocan ventas, caja ni devoluciones (`git show --stat`).

## Tabla de cobertura

| Elemento del encargo | Dónde queda |
|---|---|
| A1 a A15 | Sección 2 (filas A1 a A15; A2 y A7a con matiz) y sección 3 |
| B `sofia-apis` `9c16f35` (`negocio/reglas/catalogos`, `tecnico/modelo-de-datos/maestros-y-seguridad`/`tecnico/modelo-de-datos/transaccionales-e-historial`, `tecnico/api/catalogos-basicos`, `producto/historias-de-usuario/catalogos-inventario-y-atencion`, `producto/casos-de-uso/catalogos-inventario-y-atencion`, `calidad/casos-de-prueba/catalogos-inventario-y-atencion`, `producto/pantallas/catalogos-basicos`, manual cap. 4, `tecnico/seguridad/matriz-de-permisos`, `calidad/pruebas-automatizadas/backend`) | Sección 4; `tecnico/seguridad/matriz-de-permisos` sin cambios por este commit |
| B `sofia-web-spa` `37d053d` y `7070e1a` (`negocio/reglas/administracion-y-seguridad`, `producto/pantallas/administracion`, `producto/historias-de-usuario/acceso-y-administracion`, `producto/casos-de-uso/acceso-y-administracion`, `calidad/casos-de-prueba/acceso-y-administracion`, manual cap. 3, cap. 11, `tecnico/catalogo-de-codigos-de-error`, `calidad/pruebas-automatizadas/spa`) | Sección 4 |
| C Matriz de trazabilidad | Sección 5 |
| D Informe de cierre: correcciones, no corregido, reconciliación, estado final, comprobaciones, pendientes | Secciones 2 a 7 y Pendientes |

## Fuentes

- Código de `sofia-apis` (`9c16f35`): `src/SOFIA.API/Controllers/` (`UnidadesMedidaController`, `ProveedoresController`, `TransferenciasController`, `DeliveryController`, `MagistralesController`, `ServiciosController`, `EmpresasController`, `DIGEMIDController`), `src/SOFIA.Domain/Entities/` (`Transferencia`, `DigemidCatalogoProducto`, `HistorialPrecioProveedor`), `src/SOFIA.Domain/Common/IGlobalReadEntity.cs`, `src/SOFIA.Application/DIGEMID/`, `src/SOFIA.Infrastructure/Persistence/ApplicationDbContext.cs`, `src/SOFIA.DbUp/Scripts/` (`0001`, `0003`, `0004`, `0005`, `0015`, `0020`), pruebas de `9c16f35`; `git show 9c16f35`, `git show 34089ec`, `git log`.
- Código de `sofia-web-spa` (`7070e1a`): `src/app/pages/` (roles, seguridad/cuentas-detail, empleados, pos/nueva-venta), `src/app/shared/utils/save-error-key.util.ts`, `public/i18n/` (`dashboard`, `cuentas`, `empleados`, `roles`, `es.json`), `git show 37d053d`, `git show 7070e1a`.
- Documentos de `sofia-docs` revisados o corregidos: los de la sección 6 con fecha 2026-11-04; más los capítulos 8 y 9 del manual como evidencia para A12.
- Scripts del scratchpad de la sesión: `cie-rn14.js`, `cie-checks.js`, `cie-state.js` y los `17-*.js` de la matriz.
