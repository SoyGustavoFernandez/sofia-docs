# Referencia de la API — Parte C: productos, proveedores y personas

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-30 |
| **Backend** | `sofia-apis`, rama `main`, commit `34089ec` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `9ad15f0` |
| **Método** | Lectura de código y llamadas reales a la API local |

## Contenido

- [0. Alcance, convenciones y datos de prueba](#0-alcance-convenciones-y-datos-de-prueba)
- [1. `Medicamentos`](#1-medicamentos)
- [2. `PresentacionesVenta`](#2-presentacionesventa)
- [3. `FormulacionesClinicas`](#3-formulacionesclinicas)
- [4. `Proveedores`](#4-proveedores)
- [5. `Seguros`](#5-seguros)
- [6. `Pacientes`](#6-pacientes)
- [7. `ProfesionalesSalud`](#7-profesionalessalud)
- [8. Pendientes y «No verificado»](#8-pendientes-y-no-verificado)
- [9. Tabla de cobertura](#9-tabla-de-cobertura)
- [10. Fuentes](#10-fuentes)

## 0. Alcance, convenciones y datos de prueba

Este documento describe los 61 endpoints de siete controladores: `Medicamentos` (10), `PresentacionesVenta` (6), `FormulacionesClinicas` (7), `Proveedores` (11, incluidos el registro y el historial de precios de compra), `Seguros` (9), `Pacientes` (9) y `ProfesionalesSalud` (9). Las reglas de negocio están en [01b-catalogos.md](../01-reglas-de-negocio/01b-catalogos.md) (prefijos `MED`, `PRE`, `FOR`, `PRV`, `ASE`, `PAC`, `PRO`), los permisos en la [matriz de permisos][05B] (apartados 3.7, 3.13, 3.15, 3.16, 3.17, 3.18 y 3.21) y la mecánica del Excel en [20-carga-masiva-y-exportacion.md](../20-carga-masiva-y-exportacion.md). El orden de las fichas dentro de cada controlador es el de `_inventario/endpoints.csv`.

**Convenciones comunes.** Autenticación con `Authorization: Bearer <token>`, forma de los errores (`ProblemDetails` con la extensión `code`), paginación, exportación a Excel y carga masiva en tres pasos se describen una sola vez en [04a, apartado 0 «Convenciones comunes»][04A0]. Aquí no se repiten. En todas las fichas, un `401` (sin token o token vencido) o un `403` (sin el permiso indicado) son posibles y se resuelven según ese apartado.

**Lo que sí es común a estos siete controladores** (comprobado en el código y con llamadas reales):

- Ninguno declara una política de límite de peticiones propia: aplica solo el límite global (`rate_limit` vacío en `_inventario/endpoints.csv` para los 61 endpoints).
- El JSON usa `camelCase` en peticiones y respuestas. Los enumerados viajan como número (`condicionVenta`: `0` venta libre, `1` receta simple, `2` receta retenida, `3` estupefaciente) y las fechas sin hora (`DateOnly`) como `yyyy-MM-dd`.
- Cuando una validación del pipeline falla, el `code` es `{NombreDelComando}.{Propiedad}` de la **primera** falla y el `detail` une los mensajes de todas con `; ` (`ValidationBehavior.cs:30-31`). Si la validación la hace la entidad de dominio, el `code` es `{Entidad}.{Propiedad}` (por ejemplo `ProveedorDistribuidor.TasaCumplimiento`).
- Los `PUT` ignoran el identificador del cuerpo: el controlador siempre usa el de la ruta (`command with { Id = id }`). Por eso la comprobación «ID mismatch» de `Medicamentos` y `FormulacionesClinicas` nunca se cumple (RN-MED-011, RN-FOR-009).
- Las eliminaciones son lógicas: un registro eliminado deja de aparecer, responde 404 en las consultas por identificador y su código o nombre queda libre.
- Los controladores de carga masiva reciben el archivo en `previsualizar` como `multipart/form-data` (campo `file`, `.xlsx`, máximo 5 MB) y las filas confirmadas en `carga-masiva` como arreglo JSON de 1 a 5 000 filas (`ImportLimits.cs`). Las claves de las filas de la vista previa llevan la primera letra en minúscula.

**Qué devuelve cada operación.** Los siete controladores no se comportan igual, por eso se resume aquí y cada ficha lo repite:

| Controlador | Crear | Actualizar | Eliminar | Eliminar un inexistente |
|---|---|---|---|---|
| `Medicamentos` | 201 con el GUID y `Location` | 204 | 204 | 404 `Medicamento.NotFound` |
| `PresentacionesVenta` | 201 con el GUID y `Location` | 204 | 204 | 404 `PresentacionVenta.NotFound` |
| `FormulacionesClinicas` | 201 con el GUID y `Location` | 204 | 204 | 404 `Formulacion.NotFound` |
| `Proveedores` | **200** con el GUID | 204 | **200** con el GUID | 404 con el código genérico `NotFound` |
| `Seguros` | **200** con el GUID | 204 | **200** con el GUID | **400** con el código genérico `NotFound` |
| `Pacientes` | 201 con el GUID y `Location` | 204 | **200** con el GUID | 404 con el código genérico `NotFound` |
| `ProfesionalesSalud` | 201 con el GUID y `Location` | 204 | 204 | 404 `ProfesionalSalud.NotFound` |

### 0.1 Datos de prueba creados para esta parte

Todo se creó en la empresa de prueba «Farmacia API Referencia S.A.C.» con el prefijo `4C-`. Los datos son ficticios (incluidos los documentos de identidad, los números de colegiatura y los RUC). Los registros temporales se eliminaron para documentar el borrado.

| Qué | Registros | Identificador usado en los ejemplos |
|---|---|---|
| Datos previos (se crearon con otros controladores y no se documentan aquí) | Unidades de medida `4C-UND` «4C-Unidad», `4C-CAJ` «4C-Caja x 10» y `4C-MG` «4C-Miligramo»; laboratorio «4C-Laboratorio Referencia»; ingrediente activo «4C-Paracetamol de prueba» (`4C-N02BE01`); jerarquía de 1 «4C-Caja x 10» = 10 «4C-Unidad» para el medicamento `4C-MED-001` | Unidad base `f44c1220-a874-44a9-9751-d6b7b14abad0`, caja `d9596eca-dae5-40db-8a6b-959223731d9f`, miligramo `30019644-fcdb-4159-948a-9a5e80da32c2`, laboratorio `f1aae6d0-c52b-4b40-b7e7-e9581071f621`, ingrediente `d1dca746-6702-4934-a2fe-9d33ef104649` |
| Medicamentos | `4C-MED-001` «4C-Paracetamol 500 mg» (con presentación, formulación y precios, por eso no se puede eliminar), `4C-MED-002` «4C-Ibuprofeno 400 mg», `4C-MED-TMP` (eliminado), `4C-MED-101` y `4C-MED-102` (carga masiva) | `10420632-a172-4ee9-be87-937270292dd9` (`4C-MED-001`), `2dfbfe8c-fb18-4eca-a37a-206cc8587559` (`4C-MED-002`) |
| Presentaciones de venta | «4C-Caja x 10» (10 unidades base) de `4C-MED-001`; una presentación «4C-Unidad» (eliminada) | `70a62900-b64e-4423-b7a6-eaa548cea17f` |
| Formulaciones clínicas | `4C-MED-001` con el ingrediente de prueba (500 mg, luego 650 mg); otra para `4C-MED-002` (eliminada) | `624d8121-a159-4d9c-9d5d-b690d3658591` |
| Proveedores | «4C-Distribuidora Uno S.A.C.» (con dos precios de compra), «4C-Distribuidora Dos S.A.C.», «4C-Proveedor temporal» (eliminado), «4C-Proveedor Masivo A S.A.C.» y «4C-Proveedor Masivo B S.A.C.» (carga masiva) | `09475776-cf69-460b-affd-a8156ed71ba4` (Uno), `1593b5e4-17ba-4dff-ac77-e604ba2b986b` (Dos) |
| Aseguradoras | `4C-SEG-001` y `4C-SEG-002`, `4C-SEG-TMP` (eliminada), `4C-SEG-101` y `4C-SEG-102` (carga masiva) | `55216aab-c0e5-49ab-8079-8353405390c6` (`4C-SEG-001`), `4a0c16fe-54ff-40a5-8c7e-935558d9883d` (`4C-SEG-002`) |
| Pacientes | DNI `99900001` (usado en una receta), pasaporte `4CPAS0002`, carné `4CCE0003` (eliminado), `99900101` y `4CPAS0102` (carga masiva) | `cef89078-43d8-46f6-85c3-f6c51f5ac290` (DNI `99900001`), `d33952c4-51e0-446d-b3be-75b9f21b6c76` (`4CPAS0002`) |
| Profesionales de salud | `4C-CMP-0001` (usado en una receta), `4C-CMP-0002`, `4C-CMP-TMP` (eliminado), `4C-CMP-0101` y `4C-CMP-0102` (carga masiva) | `210cc2fa-14cb-455a-a1c2-b6ba8b126931` (`4C-CMP-0001`), `88b1aff2-4f64-4d1d-b520-2f5de033f9bb` (`4C-CMP-0002`) |
| Receta de apoyo | Una receta del paciente DNI `99900001` y el profesional `4C-CMP-0001`, creada con `POST /api/v1/Recetas` para los casos «en uso» (no se documenta aquí) | `f3db974e-ce15-497d-a813-651bd5e2850b` |

Los archivos `.xlsx` de las pruebas de carga masiva los generó el autor del documento con un escritor de ZIP mínimo (una hoja con la fila de títulos y las filas de datos como texto); los títulos son los de la plantilla de cada módulo.

### 0.2 Cómo leer las fichas

Cada ficha indica el permiso, el archivo y la línea del atributo `[HttpXxx]` del controlador (`SOFIA.API/Controllers/…`, la misma línea que da `_inventario/endpoints.csv`), el comando o consulta que ejecuta y la pantalla del SPA que lo usa. «Pantalla del SPA» se toma de la [matriz de permisos][05B] y de `_inventario/rutas-spa.csv`. Los ejemplos son respuestas capturadas de la API local el 2026-10-30; se recortan las listas y las cabeceras. El token se escribe siempre `<token>`.

Los mensajes `detail` de las respuestas de error están en inglés, tal como los envía el backend. Algunos llegan con las tildes dañadas (por ejemplo `CÃ³digo Nacional`), porque el texto fuente del validador está guardado con codificación doble: es un defecto del backend (ver el apartado 8.3).

[MED]: ../01-reglas-de-negocio/01b-catalogos.md#8-medicamentos-med
[PRE]: ../01-reglas-de-negocio/01b-catalogos.md#9-presentaciones-de-venta-pre
[FOR]: ../01-reglas-de-negocio/01b-catalogos.md#10-formulaciones-clínicas-for
[PRV]: ../01-reglas-de-negocio/01b-catalogos.md#11-proveedores-prv
[ASE]: ../01-reglas-de-negocio/01b-catalogos.md#12-aseguradoras-ase
[PAC]: ../01-reglas-de-negocio/01b-catalogos.md#14-pacientes-pac
[PRO]: ../01-reglas-de-negocio/01b-catalogos.md#15-profesionales-de-salud-pro
[04A0]: 04a-convenciones-y-administracion.md#0-convenciones-comunes
[05B]: ../05-seguridad/05b-matriz-de-permisos.md
[20]: ../20-carga-masiva-y-exportacion.md

---

## 1. `Medicamentos`

Catálogo de productos de la empresa: código nacional, nombre comercial, laboratorio, unidad base, condición de venta y precio de venta base. Archivo: `sofia-apis/src/SOFIA.API/Controllers/MedicamentosController.cs`. Ruta base: `/api/v1/Medicamentos`. Reglas: [RN-MED-001 a RN-MED-026][MED]. Permisos: [05b, apartado 3.13][05B]. Pantallas: `/medicamentos`, `/medicamentos/nueva`, `/medicamentos/:id` y `/medicamentos/carga-masiva`. El listado `GET /Medicamentos` lo usan además como autocompletado las pantallas de formulaciones, jerarquías, lotes, punto de venta, presentaciones, proveedores y stock por sucursal (05b, apartado 3.13).

**Forma del medicamento** (`MedicamentoDto`, `SOFIA.Application/Medicamentos/MedicamentoDto.cs`):

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | GUID | Identificador. |
| `codigoNacional` | texto | Único por empresa entre los no eliminados (RN-MED-010). |
| `nombreComercial` | texto | Nombre comercial. |
| `laboratorioId`, `laboratorioNombre` | GUID, texto | Laboratorio. |
| `unidadBaseId`, `unidadBaseNombre` | GUID, texto | Unidad base (descripción de la unidad de medida). |
| `condicionVenta` | entero | `0` a `3` (RN-MED-007). |
| `precioVentaBase` | decimal o `null` | `null` solo en medicamentos anteriores a ese campo o cargados por carga masiva (RN-MED-009, RN-MED-021). |
| `stockTotal` | decimal o `null` | Solo si se pidió (`incluirStock`) y en el detalle (RN-MED-015, RN-MED-018). |
| `stockPorSucursal` | arreglo o `null` | Solo en el detalle: `[ { sucursalId, nombre, cantidad } ]` (RN-MED-018). |

#### `GET /api/v1/Medicamentos/condiciones-venta`

Devuelve la lista fija de textos de la condición de venta, en el orden del enumerado (el índice es el valor numérico). No consulta la base de datos.

| | |
|---|---|
| **Permiso** | `Medicamentos:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/MedicamentosController.cs:24` → `Medicamento.CondicionesValidas` (sin comando) |
| **Pantalla del SPA** | `/medicamentos/:id` (carga inicial) |

**Parámetros:** ninguno.

**Respuesta correcta:** `200 OK`, arreglo de cuatro textos (RN-MED-019).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `Medicamentos:Leer` | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/Medicamentos/condiciones-venta
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
["Venta Libre (OTC)", "Receta Simple", "Receta Retenida", "Estupefaciente"]
```

#### `GET /api/v1/Medicamentos`

Lista los medicamentos de la empresa con filtros y paginación. Ordena por nombre comercial. Excluye los medicamentos cuyo laboratorio o unidad base estén eliminados (RN-MED-014).

| | |
|---|---|
| **Permiso** | `Medicamentos:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/MedicamentosController.cs:28` → `GetMedicamentosQuery` |
| **Pantalla del SPA** | `/medicamentos` (botón «Buscar») |

**Parámetros de consulta:**

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `codigoNacional` | texto | No | Contiene este texto, sin distinguir mayúsculas. |
| `nombreComercial` | texto | No | Contiene este texto. |
| `laboratorioNombre` | texto | No | El nombre del laboratorio contiene este texto. |
| `unidadBaseNombre` | texto | No | La descripción de la unidad base contiene este texto. |
| `condicionVenta` | entero | No | `0` a `3`, coincidencia exacta. |
| `incluirStock` | booleano | No | Si es `true`, llena `stockTotal` con la suma de la cantidad física de los lotes en todas las sucursales (RN-MED-015). Por defecto `false`: `stockTotal` viene `null`. |
| `soloStockVendibleEnMiSucursal` | booleano | No | Solo con `incluirStock`: cuenta únicamente el stock vendible de la sucursal del usuario (RN-MED-015, RN-MED-017). |
| `ordenarPorMasVendidos` | booleano | No | Ordena por unidades vendidas en la sucursal del usuario en los últimos 30 días (RN-MED-016). Las dos opciones de sucursal exigen una sucursal en la sesión (RN-MED-017). |
| `pageNumber` | entero | No | Página, por defecto 1. |
| `pageSize` | entero | No | Tamaño de página, por defecto 10. |

**Respuesta correcta:** `200 OK`: `{ items: [ MedicamentoDto ], pageNumber, totalPages, totalCount, hasPreviousPage, hasNextPage }`. En el listado `stockPorSucursal` siempre viene `null`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `Medicamentos:Leer` | [apartado 0][04A0] |
| 400 | `Auth.Sucursal` | `ordenarPorMasVendidos` o `soloStockVendibleEnMiSucursal` (con `incluirStock`) sin sucursal en la sesión (`CurrentUserExtensions.cs:10-12`) | RN-MED-017. Ejemplo no verificado: la cuenta de la referencia sí tiene sucursal (con `ordenarPorMasVendidos=true` respondió 200) |

**Ejemplo real:**

```http
GET /api/v1/Medicamentos?codigoNacional=4C-MED-001&incluirStock=true
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "items": [
    {
      "id": "10420632-a172-4ee9-be87-937270292dd9",
      "codigoNacional": "4C-MED-001",
      "nombreComercial": "4C-Paracetamol 500 mg",
      "laboratorioId": "f1aae6d0-c52b-4b40-b7e7-e9581071f621",
      "laboratorioNombre": "4C-Laboratorio Referencia",
      "unidadBaseId": "f44c1220-a874-44a9-9751-d6b7b14abad0",
      "unidadBaseNombre": "4C-Unidad",
      "condicionVenta": 0,
      "precioVentaBase": 1.5,
      "stockTotal": 0,
      "stockPorSucursal": null
    }
  ],
  "pageNumber": 1,
  "totalPages": 1,
  "totalCount": 1,
  "hasPreviousPage": false,
  "hasNextPage": false
}
```

Sin `incluirStock` (`GET /api/v1/Medicamentos?nombreComercial=4C-&pageNumber=1&pageSize=2`) la misma forma llega con `"stockTotal": null`, `totalCount` 3 y `totalPages` 2.

#### `GET /api/v1/Medicamentos/{id}`

Devuelve un medicamento con su stock total y su stock por sucursal.

| | |
|---|---|
| **Permiso** | `Medicamentos:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/MedicamentosController.cs:36` → `GetMedicamentoByIdQuery` |
| **Pantalla del SPA** | `/medicamentos/:id` (carga inicial) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `200 OK`, `MedicamentoDto` con `stockTotal` (suma de la cantidad física de sus lotes) y `stockPorSucursal` (RN-MED-018).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `Medicamento.NotFound` | No existe, está eliminado o es de otra empresa | RN-MED-012 |
| 401 / 403 | — | Sin token o sin permiso | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/Medicamentos/2dfbfe8c-fb18-4eca-a37a-206cc8587559
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "id": "2dfbfe8c-fb18-4eca-a37a-206cc8587559",
  "codigoNacional": "4C-MED-002",
  "nombreComercial": "4C-Ibuprofeno 400 mg",
  "laboratorioId": "f1aae6d0-c52b-4b40-b7e7-e9581071f621",
  "laboratorioNombre": "4C-Laboratorio Referencia",
  "unidadBaseId": "f44c1220-a874-44a9-9751-d6b7b14abad0",
  "unidadBaseNombre": "4C-Unidad",
  "condicionVenta": 1,
  "precioVentaBase": 2.4,
  "stockTotal": 0,
  "stockPorSucursal": []
}
```

Error real (`GET /api/v1/Medicamentos/00000000-0000-0000-0000-000000000099`):

```json
HTTP/1.1 404 Not Found
{
  "status": 404,
  "detail": "Medicamento with ID 00000000-0000-0000-0000-000000000099 was not found.",
  "code": "Medicamento.NotFound"
}
```

#### `POST /api/v1/Medicamentos`

Crea un medicamento.

| | |
|---|---|
| **Permiso** | `Medicamentos:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/MedicamentosController.cs:44` → `CreateMedicamentoCommand` |
| **Pantalla del SPA** | `/medicamentos/nueva` (botón «Guardar») |

**Cuerpo** (`application/json`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `codigoNacional` | texto | Sí | No vacío, máximo 50 caracteres; único por empresa entre los no eliminados ([RN-MED-002][MED], [RN-MED-003][MED], [RN-MED-010][MED]). |
| `nombreComercial` | texto | Sí | No vacío, máximo 150 ([RN-MED-004][MED], [RN-MED-003][MED]). |
| `laboratorioId` | GUID | Sí | Debe existir en la empresa ([RN-MED-005][MED]). |
| `unidadBaseId` | GUID | Sí | Debe existir en la empresa ([RN-MED-006][MED]). |
| `condicionVenta` | entero | Sí | Uno de `0` a `3` ([RN-MED-007][MED]). |
| `precioVentaBase` | decimal | Sí | No negativo; si se omite, el comando lo toma como `0` ([RN-MED-008][MED], [RN-MED-009][MED]). |

**Respuesta correcta:** `201 Created`, cuerpo = GUID del medicamento (texto JSON) y cabecera `Location` hacia `GET /api/v1/Medicamentos/{id}`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `CreateMedicamentoCommand.CodigoNacional` | Código vacío o de más de 50 caracteres | RN-MED-002, RN-MED-003 |
| 400 | `CreateMedicamentoCommand.NombreComercial` | Nombre vacío o de más de 150 | RN-MED-004, RN-MED-003 |
| 400 | `CreateMedicamentoCommand.LaboratorioId` / `.UnidadBaseId` | GUID vacío | RN-MED-005, RN-MED-006 |
| 400 | `CreateMedicamentoCommand.CondicionVenta` | Valor fuera de `0`–`3` (el `detail` une también el error de precio negativo) | RN-MED-007, RN-MED-008 |
| 400 | `CreateMedicamentoCommand.PrecioVentaBase` | Precio negativo | RN-MED-008 |
| 409 | `Medicamento.CodigoNacional.Duplicado` | Otro medicamento no eliminado usa el código | RN-MED-010 |
| 404 | `Laboratorio.NotFound` | El laboratorio no existe en la empresa | RN-MED-005 |
| 404 | `UnidadMedida.NotFound` | La unidad base no existe en la empresa | RN-MED-006 |

**Ejemplo real:**

```http
POST /api/v1/Medicamentos
Authorization: Bearer <token>
Content-Type: application/json

{
  "codigoNacional": "4C-MED-001",
  "nombreComercial": "4C-Paracetamol 500 mg",
  "laboratorioId": "f1aae6d0-c52b-4b40-b7e7-e9581071f621",
  "unidadBaseId": "f44c1220-a874-44a9-9751-d6b7b14abad0",
  "condicionVenta": 0,
  "precioVentaBase": 1.5
}
```

```json
HTTP/1.1 201 Created
"10420632-a172-4ee9-be87-937270292dd9"
```

Errores reales. Código repetido (mismo cuerpo con otro nombre):

```json
HTTP/1.1 409 Conflict
{
  "status": 409,
  "detail": "Another product already uses this national code.",
  "code": "Medicamento.CodigoNacional.Duplicado"
}
```

Código vacío (`"codigoNacional": ""`):

```json
HTTP/1.1 400 Bad Request
{
  "status": 400,
  "detail": "CÃ³digo Nacional is required.",
  "code": "CreateMedicamentoCommand.CodigoNacional"
}
```

Condición `9` y precio `-1`: `400` con `code` `CreateMedicamentoCommand.CondicionVenta` y `detail` `Invalid CondiciÃ³n de Venta.; Precio de Venta Base cannot be negative.`. Laboratorio inexistente (`00000000-0000-0000-0000-000000000099`): `404`, `Laboratorio.NotFound`, `The specified laboratory does not exist.`. Unidad base inexistente: `404`, `UnidadMedida.NotFound`, `The specified unit of measure does not exist.`.

#### `PUT /api/v1/Medicamentos/{id}`

Actualiza todos los campos de un medicamento (el identificador viene de la ruta).

| | |
|---|---|
| **Permiso** | `Medicamentos:Actualizar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/MedicamentosController.cs:54` → `UpdateMedicamentoCommand` |
| **Pantalla del SPA** | `/medicamentos/:id` (botón «Guardar» en modo edición) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Cuerpo:** los mismos seis campos y reglas que en `POST` (`codigoNacional`, `nombreComercial`, `laboratorioId`, `unidadBaseId`, `condicionVenta`, `precioVentaBase`). Un `id` en el cuerpo se ignora (RN-MED-011). El laboratorio y la unidad solo se comprueban si cambian ([RN-MED-005][MED], [RN-MED-006][MED]). El código se compara excluyendo al propio medicamento ([RN-MED-010][MED]).

**Respuesta correcta:** `204 No Content`, sin cuerpo.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `UpdateMedicamentoCommand.<Propiedad>` | Mismas validaciones que al crear | RN-MED-002 a RN-MED-008 |
| 404 | `Medicamento.NotFound` | No existe o está eliminado | RN-MED-012 |
| 409 | `Medicamento.CodigoNacional.Duplicado` | Otro medicamento usa el código | RN-MED-010 |
| 404 | `Laboratorio.NotFound` / `UnidadMedida.NotFound` | El laboratorio o la unidad cambiados no existen | RN-MED-005, RN-MED-006 |

**Ejemplo real:**

```http
PUT /api/v1/Medicamentos/2dfbfe8c-fb18-4eca-a37a-206cc8587559
Authorization: Bearer <token>
Content-Type: application/json

{
  "codigoNacional": "4C-MED-002",
  "nombreComercial": "4C-Ibuprofeno 400 mg (editado)",
  "laboratorioId": "f1aae6d0-c52b-4b40-b7e7-e9581071f621",
  "unidadBaseId": "f44c1220-a874-44a9-9751-d6b7b14abad0",
  "condicionVenta": 2,
  "precioVentaBase": 2.8
}
```

```http
HTTP/1.1 204 No Content
```

Error real (con `"codigoNacional": "4C-MED-001"`, que ya usa otro medicamento):

```json
HTTP/1.1 409 Conflict
{
  "status": 409,
  "detail": "Another product already uses this national code.",
  "code": "Medicamento.CodigoNacional.Duplicado"
}
```

Con un identificador inexistente responde `404` `Medicamento.NotFound`, igual que el `GET`.

#### `DELETE /api/v1/Medicamentos/{id}`

Elimina (borrado lógico) un medicamento. Se bloquea si algo lo referencia (RN-MED-013).

| | |
|---|---|
| **Permiso** | `Medicamentos:Eliminar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/MedicamentosController.cs:68` → `DeleteMedicamentoCommand` |
| **Pantalla del SPA** | `/medicamentos/:id` (botón «Eliminar»; ante un 409 muestra `medicamentos.detail.delete-in-use`) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `204 No Content`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `Medicamento.NotFound` | No existe o ya está eliminado | RN-MED-012 |
| 409 | `Medicamento.InUse` | Lo referencia algún registro no eliminado de: lotes de inventario, presentaciones de venta, jerarquías de unidades, formulaciones clínicas, historial de precios de proveedor, servicios de agenda, servicios clínicos de inmunización u órdenes magistrales (`DeleteMedicamentoCommandHandler.cs:34-42`) | RN-MED-013 |

**Ejemplo real.** Medicamento sin dependencias (`4C-MED-TMP`):

```http
DELETE /api/v1/Medicamentos/27ee8396-20f8-4c3a-a406-a725ef991845
Authorization: Bearer <token>
```

```http
HTTP/1.1 204 No Content
```

Después, `GET` y un segundo `DELETE` sobre el mismo identificador responden `404` `Medicamento.NotFound`.

Caso real de bloqueo: `4C-MED-001` tiene una presentación de venta «4C-Caja x 10», una jerarquía, una formulación clínica y dos precios de proveedor.

```http
DELETE /api/v1/Medicamentos/10420632-a172-4ee9-be87-937270292dd9
Authorization: Bearer <token>
```

```json
HTTP/1.1 409 Conflict
{
  "status": 409,
  "detail": "Cannot delete a medication referenced by batches, presentations, unit hierarchies, clinical formulations, supplier prices, appointments, immunizations or compounding orders.",
  "code": "Medicamento.InUse"
}
```

#### `POST /api/v1/Medicamentos/exportar`

Genera un Excel con los medicamentos que cumplen los filtros. El cuerpo lleva los títulos de columna ya traducidos por el SPA.

| | |
|---|---|
| **Permiso** | `Medicamentos:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/MedicamentosController.cs:76` → `GetMedicamentosQuery` (con `PageSize` = 10 000) |
| **Pantalla del SPA** | `/medicamentos` (botón «Exportar») |

**Cuerpo** (`application/json`, `MedicamentoExportRequest`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `headers` | arreglo de texto | Sí | Hasta 50 títulos de hasta 100 caracteres (`ExportHeadersAttribute`). Si falta o excede, `400` con `errors.Headers` (ver el ejemplo). |
| `codigoNacional`, `nombreComercial`, `laboratorioNombre`, `unidadBaseNombre` | texto | No | Mismos filtros «contiene» del listado. |
| `condicionVenta` | entero | No | `0` a `3`. |

**Respuesta correcta:** `200 OK`, `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `Content-Disposition: attachment; filename=medicamentos.xlsx`. Hoja «Data»: fila de títulos (los de `headers`) y una fila por medicamento con cinco columnas: código nacional, nombre comercial, laboratorio, unidad base y el texto de la condición de venta (RN-MED-020). Orden: nombre comercial.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | (validación de modelo, `errors.Headers`) | `headers` ausente, con más de 50 elementos o con elementos de más de 100 caracteres | RN-MED-020 |
| 400 | `Export.TooManyRows` | El filtro devuelve más de 10 000 filas | [20, apartado 2.8][20]. Ejemplo no verificado: exigiría más de 10 000 medicamentos |

**Ejemplo real:**

```http
POST /api/v1/Medicamentos/exportar
Authorization: Bearer <token>
Content-Type: application/json

{
  "headers": ["Código nacional", "Nombre comercial", "Laboratorio", "Unidad base", "Condición de venta"],
  "nombreComercial": "4C-"
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=medicamentos.xlsx; filename*=UTF-8''medicamentos.xlsx

<archivo .xlsx de 6693 bytes>
```

Contenido leído del archivo (hoja «Data»): `Código nacional | Nombre comercial | Laboratorio | Unidad base | Condición de venta`, seguido de `4C-MED-002 | 4C-Ibuprofeno 400 mg (editado) | 4C-Laboratorio Referencia | 4C-Unidad | Receta Retenida` y `4C-MED-001 | 4C-Paracetamol 500 mg | 4C-Laboratorio Referencia | 4C-Unidad | Venta Libre (OTC)`.

Error real (cuerpo sin `headers`):

```json
HTTP/1.1 400 Bad Request
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": { "Headers": ["The Headers field is required.", "Export.Headers.Invalid"] },
  "traceId": "00-…"
}
```

#### `GET /api/v1/Medicamentos/plantilla`

Descarga la plantilla Excel de carga masiva (solo la fila de títulos).

| | |
|---|---|
| **Permiso** | `Medicamentos:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/MedicamentosController.cs:112` → `ExcelTemplateGenerator.GenerateTemplate` (sin comando) |
| **Pantalla del SPA** | `/medicamentos/carga-masiva` (botón «Descargar plantilla») |

**Parámetros:** ninguno.

**Respuesta correcta:** `200 OK`, `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `Content-Disposition: attachment; filename=plantilla-medicamentos.xlsx`. Hoja «Plantilla» con cinco títulos: `CodigoNacional`, `NombreComercial`, `Laboratorio`, `UnidadBase`, `CondicionVenta` (RN-MED-021).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `Medicamentos:Crear` | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/Medicamentos/plantilla
Authorization: Bearer <token>
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=plantilla-medicamentos.xlsx; filename*=UTF-8''plantilla-medicamentos.xlsx

<archivo .xlsx de 6508 bytes>
```

#### `POST /api/v1/Medicamentos/previsualizar`

Valida un Excel fila por fila y devuelve el resultado sin guardar nada (paso 2 del flujo de [20, apartado 2.3][20]).

| | |
|---|---|
| **Permiso** | `Medicamentos:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/MedicamentosController.cs:120` → `PreviewImportMedicamentosQuery` |
| **Pantalla del SPA** | `/medicamentos/carga-masiva` (al elegir o arrastrar el archivo) |

**Cuerpo** (`multipart/form-data`): campo `file`, archivo `.xlsx` de hasta 5 MB. La extensión se comprueba (RN-MED-026 y [20, apartado 2.2][20]).

**Reglas por fila** (códigos de fila en [20, apartado 2.5][20]):

| Campo | Errores de fila | Regla |
|---|---|---|
| `codigoNacional` | `required`, `max-length` (50), `duplicate` (ya existe en la empresa), `duplicate-in-file` (repetido en el archivo; todas las repeticiones quedan con error) | RN-MED-002, RN-MED-003, RN-MED-024 |
| `nombreComercial` | `required`, `max-length` (150) | RN-MED-004, RN-MED-025 |
| `laboratorio` | `required`, `not-found` (el nombre de compañía no existe), `ambiguous` (hay más de uno con ese nombre) | RN-MED-022 |
| `unidadBase` | `required`, `not-found`, `ambiguous` (se compara con la descripción de la unidad) | RN-MED-022 |
| `condicionVenta` | `required`, `invalid-option` (debe ser uno de los cuatro textos de `condiciones-venta`, sin distinguir mayúsculas) | RN-MED-023 |

**Respuesta correcta:** `200 OK`: `{ rows: [ { rowNumber, data, errors: [ { code, field, params } ], isValid } ], totalCount, validCount, errorCount }`. `rowNumber` es el número de fila del Excel (la 2 es la primera de datos).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | (validación de modelo, `errors.file`) | No se adjuntó archivo | RN-MED-026 |
| 400 | `CargaMasiva.Archivo.Invalido` | La extensión no es `.xlsx` o el contenido no es un libro `.xlsx` válido | [20, apartado 2.6][20] |
| 400 | `CargaMasiva.Archivo.ExcedeDescompresion` / `CargaMasiva.Filas.Excedidas` | El archivo se expande más de lo permitido o tiene más de 5 000 filas | [20, apartado 2.2][20]. Ejemplo no verificado: exige un archivo de más de 5 000 filas |
| 413 | `Request.TooLarge` | El cuerpo supera 5 MB | `GlobalExceptionHandler.cs:50`. Ejemplo no verificado: exige un archivo de más de 5 MB |

**Ejemplo real** (archivo de prueba con 6 filas):

```http
POST /api/v1/Medicamentos/previsualizar
Authorization: Bearer <token>
Content-Type: multipart/form-data; boundary=----4c

------4c
Content-Disposition: form-data; name="file"; filename="medicamentos.xlsx"
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet

<contenido del archivo>
------4c--
```

El archivo tiene los títulos de la plantilla y estas filas: dos válidas (`4C-MED-101`, `4C-MED-102`), una con un código que ya existe (`4C-MED-001`), una con el laboratorio «4C-Laboratorio Fantasma», una con la condición «Libre» y una sin nombre comercial.

```json
HTTP/1.1 200 OK
{
  "rows": [
    {
      "rowNumber": 2,
      "data": { "codigoNacional": "4C-MED-101", "nombreComercial": "4C-Amoxicilina 500 mg", "laboratorio": "4C-Laboratorio Referencia", "unidadBase": "4C-Unidad", "condicionVenta": "Receta Simple" },
      "errors": [],
      "isValid": true
    },
    {
      "rowNumber": 4,
      "data": { "codigoNacional": "4C-MED-001", "nombreComercial": "4C-Codigo repetido en la base", "laboratorio": "4C-Laboratorio Referencia", "unidadBase": "4C-Unidad", "condicionVenta": "Venta Libre (OTC)" },
      "errors": [ { "code": "duplicate", "field": "codigoNacional", "params": { "value": "4C-MED-001" } } ],
      "isValid": false
    },
    {
      "rowNumber": 5,
      "data": { "codigoNacional": "4C-MED-103", "laboratorio": "4C-Laboratorio Fantasma", "…": "…" },
      "errors": [ { "code": "not-found", "field": "laboratorio", "params": { "value": "4C-Laboratorio Fantasma" } } ],
      "isValid": false
    },
    {
      "rowNumber": 6,
      "data": { "codigoNacional": "4C-MED-104", "condicionVenta": "Libre", "…": "…" },
      "errors": [ { "code": "invalid-option", "field": "condicionVenta", "params": { "value": "Libre" } } ],
      "isValid": false
    },
    {
      "rowNumber": 7,
      "data": { "codigoNacional": "4C-MED-105", "nombreComercial": null, "…": "…" },
      "errors": [ { "code": "required", "field": "nombreComercial", "params": null } ],
      "isValid": false
    }
  ],
  "totalCount": 6,
  "validCount": 2,
  "errorCount": 4
}
```

(La fila 3, válida, se omite del ejemplo.) Errores reales de archivo:

```json
HTTP/1.1 400 Bad Request
{
  "status": 400,
  "detail": "The file is not a valid .xlsx workbook.",
  "code": "CargaMasiva.Archivo.Invalido",
  "traceId": "0HNPMVVCIH2GP:00000003"
}
```

(Un archivo de texto enviado con el nombre `datos.xlsx`; el mismo error devuelve un `.xlsx` válido enviado con el nombre `datos.csv`.) Sin el campo `file`:

```json
HTTP/1.1 400 Bad Request
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": { "file": ["The file field is required."] },
  "traceId": "00-…"
}
```

#### `POST /api/v1/Medicamentos/carga-masiva`

Guarda las filas que el usuario confirmó tras la vista previa (paso 3).

| | |
|---|---|
| **Permiso** | `Medicamentos:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/MedicamentosController.cs:137` → `CargaMasivaMedicamentosCommand` |
| **Pantalla del SPA** | `/medicamentos/carga-masiva` (botón «Guardar válidos (N)») |

**Cuerpo** (`application/json`, máximo 5 MB): arreglo de 1 a 5 000 filas.

| Campo de cada fila | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `codigoNacional` | texto | Sí | Máximo 50 ([RN-MED-025][MED]) |
| `nombreComercial` | texto | Sí | Máximo 150 |
| `laboratorio` | texto | Sí | Nombre de compañía, máximo 150; debe existir y ser único ([RN-MED-022][MED]) |
| `unidadBase` | texto | Sí | Descripción de la unidad, máximo 50; debe existir y ser única |
| `condicionVenta` | texto | Sí | Uno de los cuatro textos de `condiciones-venta`, máximo 50 ([RN-MED-023][MED]) |

El servidor **omite sin avisar** las filas que no puedan guardarse: laboratorio o unidad inexistente o ambigua, condición inválida, código ya tomado en la base o por una fila anterior del archivo (RN-MED-024). El precio de venta base queda `null` (la plantilla no tiene esa columna, RN-MED-021).

**Respuesta correcta:** `200 OK`, `{ "savedCount": <filas guardadas> }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `CargaMasivaMedicamentosCommand.Rows` | Arreglo vacío o con más de 5 000 filas | RN-MED-025 |
| 400 | `CargaMasivaMedicamentosCommand.Rows[i].<Campo>` | Un campo obligatorio vacío o sobre su longitud máxima en la fila `i` (desde 0) | RN-MED-025 |
| 413 | `Request.TooLarge` | El cuerpo supera 5 MB | Ejemplo no verificado: exige más de 5 MB |

**Ejemplo real** (cuatro filas: las dos válidas de la vista previa, la del código repetido y la del laboratorio inexistente):

```http
POST /api/v1/Medicamentos/carga-masiva
Authorization: Bearer <token>
Content-Type: application/json

[
  { "codigoNacional": "4C-MED-101", "nombreComercial": "4C-Amoxicilina 500 mg", "laboratorio": "4C-Laboratorio Referencia", "unidadBase": "4C-Unidad", "condicionVenta": "Receta Simple" },
  { "codigoNacional": "4C-MED-102", "nombreComercial": "4C-Loratadina 10 mg", "laboratorio": "4C-Laboratorio Referencia", "unidadBase": "4C-Unidad", "condicionVenta": "Venta Libre (OTC)" },
  { "codigoNacional": "4C-MED-001", "nombreComercial": "4C-Codigo repetido en la base", "laboratorio": "4C-Laboratorio Referencia", "unidadBase": "4C-Unidad", "condicionVenta": "Venta Libre (OTC)" },
  { "codigoNacional": "4C-MED-103", "nombreComercial": "4C-Laboratorio inexistente", "laboratorio": "4C-Laboratorio Fantasma", "unidadBase": "4C-Unidad", "condicionVenta": "Receta Simple" }
]
```

```json
HTTP/1.1 200 OK
{ "savedCount": 2 }
```

Las dos filas omitidas no producen error. Un `GET /api/v1/Medicamentos?codigoNacional=4C-MED-10` posterior devolvió solo `4C-MED-101` y `4C-MED-102`, ambos con `"precioVentaBase": null`.

Errores reales: con un arreglo vacío `[]`:

```json
HTTP/1.1 400 Bad Request
{
  "status": 400,
  "detail": "'Rows' must not be empty.",
  "code": "CargaMasivaMedicamentosCommand.Rows"
}
```

Con una segunda fila de `codigoNacional` vacío: `400`, `code` `CargaMasivaMedicamentosCommand.Rows[1].CodigoNacional`, `detail` `'Codigo Nacional' must not be empty.`.

---

## 2. `PresentacionesVenta`

Presentaciones en las que se vende un medicamento (por ejemplo «Caja x 10»), con su precio propio y su equivalencia en unidades base. La equivalencia y la descripción las calcula el sistema a partir de la unidad de venta y de las jerarquías del producto. Archivo: `sofia-apis/src/SOFIA.API/Controllers/PresentacionesVentaController.cs`. Ruta base: `/api/v1/PresentacionesVenta`. Reglas: [RN-PRE-001 a RN-PRE-015][PRE]. Permisos: [05b, apartado 3.16][05B]. Pantallas: `/presentaciones-venta`, `/presentaciones-venta/nueva` y `/presentaciones-venta/:id` (no tiene carga masiva). `GET /PresentacionesVenta` lo usa también el modal de presentaciones de `/medicamentos/:id`.

**Forma de la presentación** (`PresentacionVentaDto`, `SOFIA.Application/PresentacionesVenta/PresentacionVentaDto.cs`):

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | GUID | Identificador. |
| `productoId`, `productoNombre` | GUID, texto | Medicamento y su nombre comercial. |
| `unidadVentaId` | GUID | Unidad de medida en que se vende. |
| `descripcion` | texto | Copia de la descripción de la unidad de venta (RN-PRE-008). |
| `cantidadUnidadesBase` | decimal | Cuántas unidades base equivale una unidad de venta; la calcula el sistema (RN-PRE-007). |
| `precioVenta` | decimal | Precio propio de la presentación (RN-PRE-009). |

#### `GET /api/v1/PresentacionesVenta`

Lista las presentaciones con filtros y paginación. Ordena por nombre del producto y luego por descripción; excluye las de productos eliminados (RN-PRE-014).

| | |
|---|---|
| **Permiso** | `PresentacionesVenta:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PresentacionesVentaController.cs:16` → `GetPresentacionesVentaQuery` |
| **Pantalla del SPA** | `/presentaciones-venta` (botón «Buscar») |

**Parámetros de consulta:**

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `productoId` | GUID | No | Solo las del medicamento indicado. |
| `productoNombre` | texto | No | El nombre comercial del producto contiene este texto. |
| `descripcion` | texto | No | La descripción contiene este texto. |
| `pageNumber` | entero | No | Página, por defecto 1. |
| `pageSize` | entero | No | Tamaño de página, por defecto 10. |

**Respuesta correcta:** `200 OK`: `{ items: [ PresentacionVentaDto ], pageNumber, totalPages, totalCount, hasPreviousPage, hasNextPage }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `PresentacionesVenta:Leer` | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/PresentacionesVenta?productoNombre=4C-&pageNumber=1&pageSize=2
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "items": [
    {
      "id": "70a62900-b64e-4423-b7a6-eaa548cea17f",
      "productoId": "10420632-a172-4ee9-be87-937270292dd9",
      "productoNombre": "4C-Paracetamol 500 mg",
      "unidadVentaId": "d9596eca-dae5-40db-8a6b-959223731d9f",
      "descripcion": "4C-Caja x 10",
      "cantidadUnidadesBase": 10,
      "precioVenta": 12.5
    },
    {
      "id": "582f8794-a97f-450b-aade-e69f836741a7",
      "productoId": "10420632-a172-4ee9-be87-937270292dd9",
      "productoNombre": "4C-Paracetamol 500 mg",
      "unidadVentaId": "f44c1220-a874-44a9-9751-d6b7b14abad0",
      "descripcion": "4C-Unidad",
      "cantidadUnidadesBase": 1,
      "precioVenta": 1.5
    }
  ],
  "pageNumber": 1,
  "totalPages": 1,
  "totalCount": 2,
  "hasPreviousPage": false,
  "hasNextPage": false
}
```

#### `GET /api/v1/PresentacionesVenta/{id}`

Devuelve una presentación.

| | |
|---|---|
| **Permiso** | `PresentacionesVenta:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PresentacionesVentaController.cs:24` → `GetPresentacionVentaByIdQuery` |
| **Pantalla del SPA** | `/presentaciones-venta/:id` (carga inicial) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `200 OK`, `PresentacionVentaDto`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `PresentacionVenta.NotFound` | No existe, está eliminada o es de otra empresa | RN-PRE-011 |
| 401 / 403 | — | Sin token o sin permiso | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/PresentacionesVenta/70a62900-b64e-4423-b7a6-eaa548cea17f
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "id": "70a62900-b64e-4423-b7a6-eaa548cea17f",
  "productoId": "10420632-a172-4ee9-be87-937270292dd9",
  "productoNombre": "4C-Paracetamol 500 mg",
  "unidadVentaId": "d9596eca-dae5-40db-8a6b-959223731d9f",
  "descripcion": "4C-Caja x 10",
  "cantidadUnidadesBase": 10,
  "precioVenta": 12.5
}
```

Error real (`GET /api/v1/PresentacionesVenta/00000000-0000-0000-0000-000000000099`):

```json
HTTP/1.1 404 Not Found
{
  "status": 404,
  "detail": "Presentacion de Venta with ID 00000000-0000-0000-0000-000000000099 was not found.",
  "code": "PresentacionVenta.NotFound"
}
```

#### `POST /api/v1/PresentacionesVenta`

Crea una presentación de venta para un medicamento. La unidad de venta debe poder convertirse a la unidad base del medicamento mediante sus jerarquías de unidades (si es la misma unidad base, la cantidad es 1 y no hace falta jerarquía).

| | |
|---|---|
| **Permiso** | `PresentacionesVenta:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PresentacionesVentaController.cs:32` → `CreatePresentacionVentaCommand` |
| **Pantalla del SPA** | `/presentaciones-venta/nueva` (botón «Guardar») |

**Cuerpo** (`application/json`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `productoId` | GUID | Sí | Debe existir en la empresa y no estar eliminado ([RN-PRE-002][PRE]). No se puede cambiar después ([RN-PRE-003][PRE]). |
| `unidadVentaId` | GUID | Sí | Debe existir en la empresa; un producto tiene a lo sumo una presentación por unidad ([RN-PRE-004][PRE], [RN-PRE-005][PRE]). |
| `precioVenta` | decimal | Sí | No negativo ([RN-PRE-009][PRE]). |

El sistema calcula `cantidadUnidadesBase` ([RN-PRE-007][PRE], [RN-PRE-006][PRE]) y `descripcion` ([RN-PRE-008][PRE]); no se envían.

**Respuesta correcta:** `201 Created`, cuerpo = GUID de la presentación y `Location` hacia `GET /api/v1/PresentacionesVenta/{id}`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `CreatePresentacionVentaCommand.ProductoId` / `.UnidadVentaId` / `.PrecioVenta` | GUID vacío o precio negativo (el `detail` une todas las fallas) | RN-PRE-002, RN-PRE-004, RN-PRE-009 |
| 404 | `PresentacionVenta.Producto` | El medicamento no existe o está eliminado | RN-PRE-002 |
| 409 | `PresentacionVenta.UnidadVenta.Duplicada` | El producto ya tiene una presentación para esa unidad | RN-PRE-005 |
| 404 | `PresentacionVenta.UnidadVenta` | La unidad de venta no existe | RN-PRE-004 |
| 400 | `PresentacionVenta.Jerarquia.NoResuelta` | No hay conversión registrada entre la unidad de venta y la unidad base | RN-PRE-006 |

**Ejemplo real.** Primero, sin jerarquía registrada (medicamento con unidad base «4C-Unidad», venta en «4C-Caja x 10»):

```http
POST /api/v1/PresentacionesVenta
Authorization: Bearer <token>
Content-Type: application/json

{
  "productoId": "10420632-a172-4ee9-be87-937270292dd9",
  "unidadVentaId": "d9596eca-dae5-40db-8a6b-959223731d9f",
  "precioVenta": 12.5
}
```

```json
HTTP/1.1 400 Bad Request
{
  "status": 400,
  "detail": "There is no unit conversion registered between the chosen sale unit and this product's base unit. Register the Jerarquía de Unidades first.",
  "code": "PresentacionVenta.Jerarquia.NoResuelta"
}
```

Tras registrar con `POST /api/v1/JerarquiasUoM` la jerarquía «1 Caja = 10 Unidad» para ese medicamento, la misma petición responde:

```json
HTTP/1.1 201 Created
"70a62900-b64e-4423-b7a6-eaa548cea17f"
```

Otros errores reales con el mismo cuerpo: repetir la petición devuelve `409` `PresentacionVenta.UnidadVenta.Duplicada` (`This product already has a sale presentation for that unit.`); con `productoId` vacío y `precioVenta` `-1`, `400` `CreatePresentacionVentaCommand.ProductoId` y `'Producto Id' must not be empty.; 'Precio Venta' must be greater than or equal to '0'.`; con un producto inexistente, `404` `PresentacionVenta.Producto`; con una unidad inexistente, `404` `PresentacionVenta.UnidadVenta`. Una presentación con la propia unidad base (`unidadVentaId` = `f44c1220-…`) se crea con `cantidadUnidadesBase` 1 sin necesitar jerarquía.

#### `PUT /api/v1/PresentacionesVenta/{id}`

Cambia la unidad de venta y el precio de una presentación. El producto no se puede cambiar (RN-PRE-003).

| | |
|---|---|
| **Permiso** | `PresentacionesVenta:Actualizar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PresentacionesVentaController.cs:42` → `UpdatePresentacionVentaCommand` |
| **Pantalla del SPA** | `/presentaciones-venta/:id` (botón «Guardar» en modo edición) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Cuerpo:** `unidadVentaId` (GUID, obligatorio) y `precioVenta` (decimal, no negativo, obligatorio). Un `id` en el cuerpo se ignora (RN-PRE-010). El sistema recalcula `cantidadUnidadesBase` y `descripcion` con la unidad elegida (RN-PRE-007, RN-PRE-008).

**Respuesta correcta:** `204 No Content`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `UpdatePresentacionVentaCommand.UnidadVentaId` / `.PrecioVenta` | GUID vacío o precio negativo | RN-PRE-004, RN-PRE-009 |
| 404 | `PresentacionVenta.NotFound` | La presentación no existe | RN-PRE-011 |
| 409 | `PresentacionVenta.UnidadVenta.Duplicada` | Otra presentación del mismo producto ya usa esa unidad | RN-PRE-005 |
| 404 | `PresentacionVenta.Producto` / `PresentacionVenta.UnidadVenta` | El producto de la presentación o la unidad nueva no existen | RN-PRE-002, RN-PRE-004 |
| 400 | `PresentacionVenta.Jerarquia.NoResuelta` | Sin conversión a la unidad base | RN-PRE-006 |

**Ejemplo real:**

```http
PUT /api/v1/PresentacionesVenta/70a62900-b64e-4423-b7a6-eaa548cea17f
Authorization: Bearer <token>
Content-Type: application/json

{
  "unidadVentaId": "d9596eca-dae5-40db-8a6b-959223731d9f",
  "precioVenta": 13.2
}
```

```http
HTTP/1.1 204 No Content
```

Un `GET` posterior devuelve `"precioVenta": 13.2`. Errores reales: pasar a la unidad base mientras otra presentación del producto ya la usa, `409` `PresentacionVenta.UnidadVenta.Duplicada`; con `precioVenta` `-5`, `400` `UpdatePresentacionVentaCommand.PrecioVenta` (`'Precio Venta' must be greater than or equal to '0'.`); con un identificador inexistente, `404` `PresentacionVenta.NotFound`.

#### `DELETE /api/v1/PresentacionesVenta/{id}`

Elimina (borrado lógico) una presentación. Se bloquea si figura en algún detalle de venta (RN-PRE-012).

| | |
|---|---|
| **Permiso** | `PresentacionesVenta:Eliminar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PresentacionesVentaController.cs:52` → `DeletePresentacionVentaCommand` |
| **Pantalla del SPA** | `/presentaciones-venta/:id` (botón «Eliminar») |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `204 No Content` (RN-PRE-013).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `PresentacionVenta.NotFound` | No existe o ya está eliminada | RN-PRE-011 |
| 409 | `PresentacionVenta.InUse` | Algún detalle de venta no eliminado la usa (`DeletePresentacionVentaCommandHandler.cs:20-27`) | RN-PRE-012 |

**Ejemplo real** (presentación «4C-Unidad» creada para borrarla):

```http
DELETE /api/v1/PresentacionesVenta/582f8794-a97f-450b-aade-e69f836741a7
Authorization: Bearer <token>
```

```http
HTTP/1.1 204 No Content
```

Un segundo `DELETE` responde `404` `PresentacionVenta.NotFound` (`Presentacion de Venta with ID 582f8794-… was not found.`). Ejemplo no verificado del `409` `PresentacionVenta.InUse` (`Cannot delete a sale presentation that has sales registered.`): exige una venta completada con esa presentación, que necesita lote con stock y caja abierta; el código está en `DeletePresentacionVentaCommandHandler.cs:20-27`.

#### `POST /api/v1/PresentacionesVenta/exportar`

Genera un Excel con las presentaciones que cumplen los filtros.

| | |
|---|---|
| **Permiso** | `PresentacionesVenta:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PresentacionesVentaController.cs:60` → `GetPresentacionesVentaQuery` (con `PageSize` = 10 000) |
| **Pantalla del SPA** | `/presentaciones-venta` (botón «Exportar») |

**Cuerpo** (`application/json`, `PresentacionVentaExportRequest`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `headers` | arreglo de texto | Sí | Hasta 50 títulos de hasta 100 caracteres (`ExportHeadersAttribute`); si no, `400` con `errors.Headers`. |
| `productoNombre`, `descripcion` | texto | No | Mismos filtros «contiene» del listado (no hay filtro por `productoId` en la exportación). |

**Respuesta correcta:** `200 OK`, `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `Content-Disposition: attachment; filename=presentaciones-venta.xlsx`. Hoja «Data»: títulos de `headers` y, por presentación, cuatro columnas: producto, descripción, cantidad de unidades base y precio de venta.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | (validación de modelo, `errors.Headers`) | `headers` ausente o fuera de límites | — |
| 400 | `Export.TooManyRows` | Más de 10 000 filas | [20, apartado 2.8][20]. Ejemplo no verificado: exige más de 10 000 presentaciones |

**Ejemplo real:**

```http
POST /api/v1/PresentacionesVenta/exportar
Authorization: Bearer <token>
Content-Type: application/json

{
  "headers": ["Producto", "Descripción", "Cantidad de unidades base", "Precio de venta"],
  "productoNombre": "4C-"
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=presentaciones-venta.xlsx; filename*=UTF-8''presentaciones-venta.xlsx

<archivo .xlsx de 6604 bytes>
```

Contenido leído (hoja «Data»): `4C-Paracetamol 500 mg | 4C-Caja x 10 | 10 | 13.2` y `4C-Paracetamol 500 mg | 4C-Unidad | 1 | 1.5`.

---

## 3. `FormulacionesClinicas`

Composición activa de un medicamento: cada fila une un medicamento con un ingrediente activo, su concentración, la unidad de medida y el código TE Orange. Archivo: `sofia-apis/src/SOFIA.API/Controllers/FormulacionesClinicasController.cs`. Ruta base: `/api/v1/FormulacionesClinicas`. Reglas: [RN-FOR-001 a RN-FOR-013][FOR]. Permisos: [05b, apartado 3.7][05B]. Pantallas: `/formulaciones-clinicas`, `/formulaciones-clinicas/nueva` y `/formulaciones-clinicas/:id` (no tiene carga masiva). `GET /FormulacionesClinicas/medicamento/{productoId}` es **Solo API (sin pantalla)**.

**Forma de la formulación** (`FormulacionClinicaDto`, `SOFIA.Application/FormulacionesClinicas/FormulacionClinicaDto.cs`):

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | GUID | Identificador. |
| `productoId`, `productoNombre` | GUID, texto | Medicamento y su nombre comercial. |
| `ingredienteId`, `ingredienteNombre` | GUID, texto | Ingrediente activo y su denominación DCI. |
| `concentracionDosis` | decimal | Concentración, mayor que cero (4 decimales, RN-FOR-006). |
| `unidadMedidaId`, `unidadMedidaNombre` | GUID, texto | Unidad de la concentración y su descripción. |
| `codigoTeOrange` | texto o `null` | Código TE Orange, hasta 5 caracteres (RN-FOR-008). |

#### `GET /api/v1/FormulacionesClinicas`

Lista las formulaciones no eliminadas con filtros y paginación. Ordena por nombre del medicamento (RN-FOR-012).

| | |
|---|---|
| **Permiso** | `FormulacionesClinicas:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/FormulacionesClinicasController.cs:17` → `GetFormulacionesClinicasWithPaginationQuery` |
| **Pantalla del SPA** | `/formulaciones-clinicas` (botón «Buscar») |

**Parámetros de consulta:**

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `productoNombre` | texto | No | El nombre comercial del medicamento contiene este texto. |
| `ingredienteNombre` | texto | No | La denominación DCI del ingrediente contiene este texto. |
| `pageNumber` | entero | No | Página, por defecto 1. |
| `pageSize` | entero | No | Tamaño de página, por defecto 10. |

**Respuesta correcta:** `200 OK`: `{ items: [ FormulacionClinicaDto ], pageNumber, totalPages, totalCount, hasPreviousPage, hasNextPage }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `FormulacionesClinicas:Leer` | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/FormulacionesClinicas?productoNombre=4C-&ingredienteNombre=Paracetamol&pageNumber=1&pageSize=5
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "items": [
    {
      "id": "cfbbca2d-548a-45db-aed6-d6cd72024fd0",
      "productoId": "2dfbfe8c-fb18-4eca-a37a-206cc8587559",
      "productoNombre": "4C-Ibuprofeno 400 mg (editado)",
      "ingredienteId": "d1dca746-6702-4934-a2fe-9d33ef104649",
      "ingredienteNombre": "4C-Paracetamol de prueba",
      "concentracionDosis": 400,
      "unidadMedidaId": "30019644-fcdb-4159-948a-9a5e80da32c2",
      "unidadMedidaNombre": "4C-Miligramo",
      "codigoTeOrange": null
    },
    {
      "id": "624d8121-a159-4d9c-9d5d-b690d3658591",
      "productoNombre": "4C-Paracetamol 500 mg",
      "ingredienteNombre": "4C-Paracetamol de prueba",
      "concentracionDosis": 500,
      "unidadMedidaNombre": "4C-Miligramo",
      "codigoTeOrange": "AB",
      "…": "…"
    }
  ],
  "pageNumber": 1,
  "totalPages": 1,
  "totalCount": 2,
  "hasPreviousPage": false,
  "hasNextPage": false
}
```

(La segunda fila se abrevia con `…`; en la respuesta real trae los mismos campos que la primera.)

#### `GET /api/v1/FormulacionesClinicas/medicamento/{productoId}`

Devuelve, sin paginar, todas las formulaciones no eliminadas de un medicamento.

| | |
|---|---|
| **Permiso** | `FormulacionesClinicas:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/FormulacionesClinicasController.cs:24` → `GetFormulacionesByMedicamentoQuery` |
| **Pantalla del SPA** | Solo API (sin pantalla) |

**Parámetros de ruta:** `productoId` (GUID, obligatorio).

**Respuesta correcta:** `200 OK`, arreglo de `FormulacionClinicaDto` (vacío si el medicamento no tiene formulaciones o no existe; no valida su existencia, RN-FOR-013).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin permiso | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/FormulacionesClinicas/medicamento/10420632-a172-4ee9-be87-937270292dd9
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
[
  {
    "id": "624d8121-a159-4d9c-9d5d-b690d3658591",
    "productoId": "10420632-a172-4ee9-be87-937270292dd9",
    "productoNombre": "4C-Paracetamol 500 mg",
    "ingredienteId": "d1dca746-6702-4934-a2fe-9d33ef104649",
    "ingredienteNombre": "4C-Paracetamol de prueba",
    "concentracionDosis": 500,
    "unidadMedidaId": "30019644-fcdb-4159-948a-9a5e80da32c2",
    "unidadMedidaNombre": "4C-Miligramo",
    "codigoTeOrange": "AB"
  }
]
```

Con el identificador inexistente `00000000-0000-0000-0000-000000000099` responde `200 OK` y `[]`.

#### `GET /api/v1/FormulacionesClinicas/{id}`

Devuelve una formulación.

| | |
|---|---|
| **Permiso** | `FormulacionesClinicas:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/FormulacionesClinicasController.cs:32` → `GetFormulacionByIdQuery` |
| **Pantalla del SPA** | `/formulaciones-clinicas/:id` (carga inicial) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `200 OK`, `FormulacionClinicaDto`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `Formulacion.NotFound` | No existe o está eliminada | RN-FOR-010 |
| 401 / 403 | — | Sin token o sin permiso | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/FormulacionesClinicas/624d8121-a159-4d9c-9d5d-b690d3658591
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "id": "624d8121-a159-4d9c-9d5d-b690d3658591",
  "productoId": "10420632-a172-4ee9-be87-937270292dd9",
  "productoNombre": "4C-Paracetamol 500 mg",
  "ingredienteId": "d1dca746-6702-4934-a2fe-9d33ef104649",
  "ingredienteNombre": "4C-Paracetamol de prueba",
  "concentracionDosis": 500,
  "unidadMedidaId": "30019644-fcdb-4159-948a-9a5e80da32c2",
  "unidadMedidaNombre": "4C-Miligramo",
  "codigoTeOrange": "AB"
}
```

Error real (identificador `00000000-0000-0000-0000-000000000099`):

```json
HTTP/1.1 404 Not Found
{
  "status": 404,
  "detail": "FormulaciÃ³n with ID 00000000-0000-0000-0000-000000000099 not found.",
  "code": "Formulacion.NotFound"
}
```

#### `POST /api/v1/FormulacionesClinicas`

Agrega un ingrediente activo a un medicamento.

| | |
|---|---|
| **Permiso** | `FormulacionesClinicas:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/FormulacionesClinicasController.cs:40` → `CreateFormulacionClinicaCommand` |
| **Pantalla del SPA** | `/formulaciones-clinicas/nueva` (botón «Guardar») |

**Cuerpo** (`application/json`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `productoId` | GUID | Sí | Medicamento existente y no eliminado ([RN-FOR-002][FOR]). No se puede cambiar después ([RN-FOR-003][FOR]). |
| `ingredienteId` | GUID | Sí | Ingrediente activo existente y no eliminado; un medicamento lista cada ingrediente una sola vez ([RN-FOR-004][FOR], [RN-FOR-005][FOR]). |
| `concentracionDosis` | decimal | Sí | Mayor que cero ([RN-FOR-006][FOR]). |
| `unidadMedidaId` | GUID | Sí | Unidad existente y no eliminada ([RN-FOR-007][FOR]). |
| `codigoTeOrange` | texto | No | Hasta 5 caracteres ([RN-FOR-008][FOR]). |

**Respuesta correcta:** `201 Created`, cuerpo = GUID de la formulación y `Location` hacia `GET /api/v1/FormulacionesClinicas/{id}`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `CreateFormulacionClinicaCommand.<Propiedad>` | GUID vacío, concentración no mayor que cero o código TE Orange de más de 5 caracteres | RN-FOR-002, RN-FOR-004, RN-FOR-006 a RN-FOR-008 |
| 404 | `Medicamento.NotFound` | El medicamento no existe | RN-FOR-002 |
| 404 | `IngredienteActivo.NotFound` | El ingrediente no existe | RN-FOR-004 |
| 409 | `Formulacion.Ingrediente.Duplicado` | El medicamento ya lista ese ingrediente | RN-FOR-005 |
| 404 | `UnidadMedida.NotFound` | La unidad no existe | RN-FOR-007 |

**Ejemplo real:**

```http
POST /api/v1/FormulacionesClinicas
Authorization: Bearer <token>
Content-Type: application/json

{
  "productoId": "10420632-a172-4ee9-be87-937270292dd9",
  "ingredienteId": "d1dca746-6702-4934-a2fe-9d33ef104649",
  "concentracionDosis": 500,
  "unidadMedidaId": "30019644-fcdb-4159-948a-9a5e80da32c2",
  "codigoTeOrange": "AB"
}
```

```json
HTTP/1.1 201 Created
"624d8121-a159-4d9c-9d5d-b690d3658591"
```

Errores reales: repetir el medicamento y el ingrediente con otra concentración:

```json
HTTP/1.1 409 Conflict
{
  "status": 409,
  "detail": "This product already lists that active ingredient.",
  "code": "Formulacion.Ingrediente.Duplicado"
}
```

Con `concentracionDosis` `0` y `codigoTeOrange` `ABCDEF`:

```json
HTTP/1.1 400 Bad Request
{
  "status": 400,
  "detail": "'Concentracion Dosis' must be greater than '0'.; The length of 'Codigo Te Orange' must be 5 characters or fewer. You entered 6 characters.",
  "code": "CreateFormulacionClinicaCommand.ConcentracionDosis"
}
```

Con un medicamento, ingrediente o unidad inexistentes responde `404` con `Medicamento.NotFound`, `IngredienteActivo.NotFound` o `UnidadMedida.NotFound` (por ejemplo, `Medicamento with ID 00000000-… not found.`).

#### `PUT /api/v1/FormulacionesClinicas/{id}`

Cambia el ingrediente, la concentración, la unidad o el código TE Orange de una formulación. El medicamento no se puede cambiar (RN-FOR-003).

| | |
|---|---|
| **Permiso** | `FormulacionesClinicas:Actualizar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/FormulacionesClinicasController.cs:50` → `UpdateFormulacionClinicaCommand` |
| **Pantalla del SPA** | `/formulaciones-clinicas/:id` (botón «Guardar» en modo edición) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Cuerpo:** `ingredienteId`, `concentracionDosis`, `unidadMedidaId` (obligatorios) y `codigoTeOrange` (opcional), con las mismas reglas que al crear. El ingrediente y la unidad solo se comprueban si cambian (RN-FOR-004, RN-FOR-007). Un `id` en el cuerpo se ignora (RN-FOR-009).

**Respuesta correcta:** `204 No Content`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `UpdateFormulacionClinicaCommand.<Propiedad>` | Mismas validaciones que al crear | RN-FOR-006 a RN-FOR-008 |
| 404 | `Formulacion.NotFound` | No existe o está eliminada | RN-FOR-010 |
| 404 | `IngredienteActivo.NotFound` / `UnidadMedida.NotFound` | El ingrediente o la unidad cambiados no existen | RN-FOR-004, RN-FOR-007 |
| 409 | `Formulacion.Ingrediente.Duplicado` | Otro registro del mismo medicamento ya usa el ingrediente nuevo | RN-FOR-005 |

**Ejemplo real:**

```http
PUT /api/v1/FormulacionesClinicas/624d8121-a159-4d9c-9d5d-b690d3658591
Authorization: Bearer <token>
Content-Type: application/json

{
  "ingredienteId": "d1dca746-6702-4934-a2fe-9d33ef104649",
  "concentracionDosis": 650,
  "unidadMedidaId": "30019644-fcdb-4159-948a-9a5e80da32c2",
  "codigoTeOrange": "AB"
}
```

```http
HTTP/1.1 204 No Content
```

Un `GET` posterior devuelve `"concentracionDosis": 650`. Errores reales: `concentracionDosis` `-1` → `400` `UpdateFormulacionClinicaCommand.ConcentracionDosis` (`'Concentracion Dosis' must be greater than '0'.`); identificador inexistente → `404` `Formulacion.NotFound`; ingrediente inexistente → `404` `IngredienteActivo.NotFound`.

#### `DELETE /api/v1/FormulacionesClinicas/{id}`

Elimina (borrado lógico) una formulación. No valida dependencias (RN-FOR-011).

| | |
|---|---|
| **Permiso** | `FormulacionesClinicas:Eliminar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/FormulacionesClinicasController.cs:64` → `DeleteFormulacionClinicaCommand` |
| **Pantalla del SPA** | `/formulaciones-clinicas/:id` (botón «Eliminar») |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `204 No Content`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `Formulacion.NotFound` | No existe o ya está eliminada | RN-FOR-010 |

**Ejemplo real** (formulación de `4C-MED-002`):

```http
DELETE /api/v1/FormulacionesClinicas/cfbbca2d-548a-45db-aed6-d6cd72024fd0
Authorization: Bearer <token>
```

```http
HTTP/1.1 204 No Content
```

Un segundo `DELETE` responde `404`, `Formulacion.NotFound`, `detail` `FormulaciÃ³n with ID cfbbca2d-… not found.`. Mientras exista una formulación, el ingrediente activo no se puede eliminar: `DELETE /api/v1/IngredientesActivos/{id}` respondió `409` `IngredienteActivo.InUse` (se documenta en la parte B).

#### `POST /api/v1/FormulacionesClinicas/exportar`

Genera un Excel con las formulaciones que cumplen los filtros.

| | |
|---|---|
| **Permiso** | `FormulacionesClinicas:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/FormulacionesClinicasController.cs:72` → `GetFormulacionesClinicasWithPaginationQuery` (con `PageSize` = 10 000) |
| **Pantalla del SPA** | `/formulaciones-clinicas` (botón «Exportar») |

**Cuerpo** (`application/json`, `FormulacionClinicaExportRequest`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `headers` | arreglo de texto | Sí | Hasta 50 títulos de hasta 100 caracteres; si no, `400` con `errors.Headers`. |
| `productoNombre`, `ingredienteNombre` | texto | No | Mismos filtros «contiene» del listado. |

**Respuesta correcta:** `200 OK`, `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `Content-Disposition: attachment; filename=formulaciones-clinicas.xlsx`. Hoja «Data»: títulos de `headers` y, por formulación, cinco columnas: medicamento, ingrediente, concentración, unidad de medida y código TE Orange (vacío si no tiene).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | (validación de modelo, `errors.Headers`) | `headers` ausente o fuera de límites | — |
| 400 | `Export.TooManyRows` | Más de 10 000 filas | [20, apartado 2.8][20]. Ejemplo no verificado: exige más de 10 000 formulaciones |

**Ejemplo real:**

```http
POST /api/v1/FormulacionesClinicas/exportar
Authorization: Bearer <token>
Content-Type: application/json

{
  "headers": ["Medicamento", "Ingrediente activo", "Concentración", "Unidad", "TE Orange"],
  "productoNombre": "4C-"
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=formulaciones-clinicas.xlsx; filename*=UTF-8''formulaciones-clinicas.xlsx

<archivo .xlsx de 6660 bytes>
```

Contenido leído (hoja «Data»): `4C-Ibuprofeno 400 mg (editado) | 4C-Paracetamol de prueba | 400 | 4C-Miligramo | (vacío)` y `4C-Paracetamol 500 mg | 4C-Paracetamol de prueba | 650 | 4C-Miligramo | AB`.

---

## 4. `Proveedores`

Proveedores y distribuidores de la empresa, con su registro de precios de compra por medicamento. Archivo: `sofia-apis/src/SOFIA.API/Controllers/ProveedoresController.cs`. Ruta base: `/api/v1/Proveedores`. Reglas: [RN-PRV-001 a RN-PRV-028][PRV]. Permisos: [05b, apartado 3.18][05B]; además de `Leer`, `Crear`, `Actualizar` y `Eliminar`, el registro de precios usa la acción propia `RegistrarPrecioProveedor`. Pantallas: `/proveedores`, `/proveedores/nueva`, `/proveedores/:id` (con la tarjeta «Precios de compra» y el diálogo «Registrar precio») y `/proveedores/carga-masiva`.

**Forma del proveedor** (`ProveedorDto`, `SOFIA.Application/Proveedores/Queries/GetProveedores/GetProveedoresQuery.cs:9`):

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | GUID | Identificador. |
| `razonSocial` | texto | Razón social, única por empresa entre los no eliminados (RN-PRV-008). |
| `taxId` | texto | Identificador tributario («RUC / Tax ID»), único por empresa (RN-PRV-007). No se valida su formato (RN-PRV-003). |
| `terminosFinancieros` | texto o `null` | Hasta 100 caracteres. |
| `calificacionEsg` | decimal o `null` | De 0 a 100. |
| `tasaCumplimiento` | decimal | De 0 a 100 (RN-PRV-006). |

**Forma del precio de compra** (`PrecioProveedorDto`, `SOFIA.Application/Proveedores/Queries/GetPreciosProveedor/PrecioProveedorDto.cs`): `id`, `productoId`, `productoNombre`, `costoPorUnidadBase` (costo neto, 4 decimales), `fechaInicioVigencia`, `fechaFinVigencia` (`null` mientras esté abierto) y `vigente` (`true` si hoy está entre ambas fechas).

#### `POST /api/v1/Proveedores`

Crea un proveedor.

| | |
|---|---|
| **Permiso** | `Proveedores:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProveedoresController.cs:21` → `CreateProveedorCommand` |
| **Pantalla del SPA** | `/proveedores/nueva` (botón «Guardar») |

**Cuerpo** (`application/json`). El validador del comando está vacío («TODO»); las reglas las aplica la entidad `ProveedorDistribuidor` (`SOFIA.Domain/Entities/ProveedorDistribuidor.cs:15-65`).

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `razonSocial` | texto | Sí | No vacío, máximo 200; único entre los no eliminados ([RN-PRV-002][PRV], [RN-PRV-008][PRV]). |
| `taxId` | texto | Sí | No vacío, máximo 50; único entre los no eliminados ([RN-PRV-003][PRV], [RN-PRV-007][PRV]). |
| `terminosFinancieros` | texto | No | Máximo 100 ([RN-PRV-004][PRV]). |
| `calificacionEsg` | decimal | No | De 0 a 100 ([RN-PRV-005][PRV]). |
| `tasaCumplimiento` | decimal | No | De 0 a 100; si se omite vale 100 ([RN-PRV-006][PRV]). |

**Respuesta correcta:** `200 OK` (no 201, RN-PRV-011), cuerpo = GUID del proveedor (texto JSON), sin `Location`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `ProveedorDistribuidor.RazonSocial` / `.TaxId` / `.TerminosFinancieros` / `.CalificacionEsg` / `.TasaCumplimiento` | Un campo vacío, demasiado largo o fuera de rango (se informa la primera falla) | RN-PRV-002 a RN-PRV-006 |
| 409 | `Proveedor.TaxId.Duplicado` | Otro proveedor no eliminado usa el identificador tributario (se comprueba antes que la razón social) | RN-PRV-007 |
| 409 | `Proveedor.RazonSocial.Duplicado` | Otro proveedor no eliminado usa la razón social | RN-PRV-008 |

**Ejemplo real:**

```http
POST /api/v1/Proveedores
Authorization: Bearer <token>
Content-Type: application/json

{
  "razonSocial": "4C-Distribuidora Uno S.A.C.",
  "taxId": "4C-20100000001",
  "terminosFinancieros": "30 dias",
  "calificacionEsg": 80,
  "tasaCumplimiento": 95
}
```

```json
HTTP/1.1 200 OK
"09475776-cf69-460b-affd-a8156ed71ba4"
```

Sin `tasaCumplimiento` el proveedor queda con 100 (el segundo proveedor de prueba, `4C-Distribuidora Dos S.A.C.`, se creó solo con `razonSocial` y `taxId`). Errores reales: mismo `taxId` con otra razón social → `409` `Proveedor.TaxId.Duplicado` (`Another supplier already uses this tax id.`); misma razón social con otro `taxId` → `409` `Proveedor.RazonSocial.Duplicado` (`Another supplier already uses this business name.`); razón social vacía:

```json
HTTP/1.1 400 Bad Request
{
  "status": 400,
  "detail": "Razón Social is required.",
  "code": "ProveedorDistribuidor.RazonSocial"
}
```

Con `tasaCumplimiento` `150`: `400`, `ProveedorDistribuidor.TasaCumplimiento`, `Compliance Rate must be between 0 and 100.`.

#### `POST /api/v1/Proveedores/precios`

Registra el precio de compra de un medicamento para un proveedor. Calcula el costo neto con el descuento y cierra, en el mismo instante, el precio abierto anterior del mismo proveedor y producto (RN-PRV-020).

| | |
|---|---|
| **Permiso** | `Proveedores:RegistrarPrecioProveedor` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProveedoresController.cs:29` → `RegistrarPrecioProveedorCommand` |
| **Pantalla del SPA** | `/proveedores/:id` (diálogo abierto con el botón «Registrar precio»; solo en modo vista) |

**Cuerpo** (`application/json`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `proveedorId` | GUID | Sí | Debe existir ([RN-PRV-018][PRV], [RN-PRV-019][PRV]). Se comprueba primero. |
| `medicamentoId` | GUID | Sí | Debe existir ([RN-PRV-018][PRV], [RN-PRV-019][PRV]). |
| `precioCompra` | decimal | Sí | Mayor que cero y como máximo 9 999 999,9999 ([RN-PRV-018][PRV]). |
| `descuentoPorcentaje` | decimal | Sí | De 0 a 100 ([RN-PRV-018][PRV]). |

El costo neto es `precioCompra × (1 − descuentoPorcentaje / 100)`, redondeado a 4 decimales (alejándose del cero). El registro nuevo empieza en la hora UTC actual, sin fecha de fin, con plazo de entrega 0 y cantidad mínima 1 (`RegistrarPrecioProveedorCommandHandler.cs:25-28`; reglas de la entidad en [RN-PRV-021 a RN-PRV-024][PRV]).

**Respuesta correcta:** `200 OK`, cuerpo = GUID del registro de precio (texto JSON).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `RegistrarPrecioProveedorCommand.<Propiedad>` | GUID vacío, precio fuera de rango o descuento fuera de 0 a 100 (el `detail` une las fallas) | RN-PRV-018 |
| 404 | `Proveedor.NotFound` | El proveedor no existe | RN-PRV-019 |
| 404 | `Medicamento.NotFound` | El medicamento no existe | RN-PRV-019 |

**Ejemplo real.** Primer precio (10,00 con 5 % de descuento):

```http
POST /api/v1/Proveedores/precios
Authorization: Bearer <token>
Content-Type: application/json

{
  "proveedorId": "09475776-cf69-460b-affd-a8156ed71ba4",
  "medicamentoId": "10420632-a172-4ee9-be87-937270292dd9",
  "precioCompra": 10,
  "descuentoPorcentaje": 5
}
```

```json
HTTP/1.1 200 OK
"9f8b948c-fdc8-47ee-9aff-9475db4c524e"
```

Un segundo registro (`precioCompra` `11.5`, `descuentoPorcentaje` `0`) devolvió `"0059c00e-99af-4974-9d93-e3e37217658f"` y cerró el primero (ver `GET /Proveedores/{id}/precios`). Errores reales: con `precioCompra` `0` y `descuentoPorcentaje` `150`:

```json
HTTP/1.1 400 Bad Request
{
  "status": 400,
  "detail": "'Precio Compra' must be greater than '0'.; 'Descuento Porcentaje' must be between 0 and 100. You entered 150.",
  "code": "RegistrarPrecioProveedorCommand.PrecioCompra"
}
```

Con un proveedor inexistente: `404`, `Proveedor.NotFound`, `Proveedor with ID 00000000-0000-0000-0000-000000000099 was not found.`; con un medicamento inexistente: `404`, `Medicamento.NotFound`.

#### `GET /api/v1/Proveedores`

Lista los proveedores con filtros y paginación. Ordena por razón social y luego por identificador (RN-PRV-012).

| | |
|---|---|
| **Permiso** | `Proveedores:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProveedoresController.cs:37` → `GetProveedoresQuery` |
| **Pantalla del SPA** | `/proveedores` (botón «Buscar») |

**Parámetros de consulta:**

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `razonSocial` | texto | No | Contiene este texto. |
| `taxId` | texto | No | Contiene este texto. |
| `tasaCumplimientoDesde` | decimal | No | Tasa mayor o igual. |
| `tasaCumplimientoHasta` | decimal | No | Tasa menor o igual. |
| `pageNumber` | entero | No | Página, por defecto 1. |
| `pageSize` | entero | No | Tamaño de página, por defecto 10. |

**Respuesta correcta:** `200 OK`: `{ items: [ ProveedorDto ], pageNumber, totalPages, totalCount, hasPreviousPage, hasNextPage }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `Proveedores:Leer` | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/Proveedores?razonSocial=4C-&tasaCumplimientoDesde=90&pageNumber=1&pageSize=2
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "items": [
    { "id": "1593b5e4-17ba-4dff-ac77-e604ba2b986b", "razonSocial": "4C-Distribuidora Dos S.A.C.", "taxId": "4C-20100000002", "terminosFinancieros": null, "calificacionEsg": null, "tasaCumplimiento": 100 },
    { "id": "09475776-cf69-460b-affd-a8156ed71ba4", "razonSocial": "4C-Distribuidora Uno S.A.C.", "taxId": "4C-20100000001", "terminosFinancieros": "30 dias", "calificacionEsg": 80, "tasaCumplimiento": 95 }
  ],
  "pageNumber": 1,
  "totalPages": 2,
  "totalCount": 3,
  "hasPreviousPage": false,
  "hasNextPage": true
}
```

(En ese momento había tres proveedores `4C-` con tasa mayor o igual a 90, entre ellos el temporal.)

#### `GET /api/v1/Proveedores/{id}`

Devuelve un proveedor.

| | |
|---|---|
| **Permiso** | `Proveedores:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProveedoresController.cs:44` → `GetProveedorByIdQuery` |
| **Pantalla del SPA** | `/proveedores/:id` (carga inicial) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `200 OK`, `ProveedorDto`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `Proveedor.NotFound` | No existe o está eliminado | RN-PRV-009 |
| 401 / 403 | — | Sin token o sin permiso | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/Proveedores/09475776-cf69-460b-affd-a8156ed71ba4
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "id": "09475776-cf69-460b-affd-a8156ed71ba4",
  "razonSocial": "4C-Distribuidora Uno S.A.C.",
  "taxId": "4C-20100000001",
  "terminosFinancieros": "30 dias",
  "calificacionEsg": 80,
  "tasaCumplimiento": 95
}
```

Error real (`GET /api/v1/Proveedores/00000000-0000-0000-0000-000000000099`): `404`, `Proveedor.NotFound`, `Proveedor with ID 00000000-0000-0000-0000-000000000099 was not found.`.

#### `GET /api/v1/Proveedores/{id}/precios`

Devuelve el historial de precios de compra del proveedor (vigentes y cerrados), del más reciente al más antiguo (RN-PRV-025).

| | |
|---|---|
| **Permiso** | `Proveedores:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProveedoresController.cs:52` → `GetPreciosProveedorQuery` |
| **Pantalla del SPA** | `/proveedores/:id` (tarjeta «Precios de compra», solo en modo vista) |

**Parámetros:**

| Nombre | En | Tipo | Obligatorio | Descripción |
|---|---|---|---|---|
| `id` | ruta | GUID | Sí | Proveedor. |
| `pageNumber` | consulta | entero | No | Página, por defecto 1. |
| `pageSize` | consulta | entero | No | Tamaño de página, por defecto 10. |

**Respuesta correcta:** `200 OK`: `{ items: [ PrecioProveedorDto ], pageNumber, totalPages, totalCount, hasPreviousPage, hasNextPage }`. Orden: `fechaInicioVigencia` descendente.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `Proveedor.NotFound` | El proveedor no existe o está eliminado | RN-PRV-025 |
| 401 / 403 | — | Sin token o sin `Proveedores:Leer` | [apartado 0][04A0] |

**Ejemplo real** (tras registrar los dos precios):

```http
GET /api/v1/Proveedores/09475776-cf69-460b-affd-a8156ed71ba4/precios?pageNumber=1&pageSize=10
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "items": [
    {
      "id": "0059c00e-99af-4974-9d93-e3e37217658f",
      "productoId": "10420632-a172-4ee9-be87-937270292dd9",
      "productoNombre": "4C-Paracetamol 500 mg",
      "costoPorUnidadBase": 11.5,
      "fechaInicioVigencia": "2026-10-31T03:57:47.147Z",
      "fechaFinVigencia": null,
      "vigente": true
    },
    {
      "id": "9f8b948c-fdc8-47ee-9aff-9475db4c524e",
      "productoId": "10420632-a172-4ee9-be87-937270292dd9",
      "productoNombre": "4C-Paracetamol 500 mg",
      "costoPorUnidadBase": 9.5,
      "fechaInicioVigencia": "2026-10-31T03:57:43.987Z",
      "fechaFinVigencia": "2026-10-31T03:57:47.147Z",
      "vigente": false
    }
  ],
  "pageNumber": 1,
  "totalPages": 1,
  "totalCount": 2,
  "hasPreviousPage": false,
  "hasNextPage": false
}
```

El primer precio (10 con 5 % de descuento) quedó en `9.5` y se cerró en el mismo instante en que empezó el segundo. Con un proveedor inexistente responde `404` `Proveedor.NotFound`.

#### `PUT /api/v1/Proveedores/{id}`

Actualiza todos los campos de un proveedor.

| | |
|---|---|
| **Permiso** | `Proveedores:Actualizar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProveedoresController.cs:60` → `UpdateProveedorCommand` |
| **Pantalla del SPA** | `/proveedores/:id` (botón «Guardar» en modo edición) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Cuerpo:** `razonSocial`, `taxId`, `terminosFinancieros`, `calificacionEsg` y `tasaCumplimiento`, con las reglas de `POST`. A diferencia de `POST`, **`tasaCumplimiento` no tiene valor por defecto**: si se omite, el comando recibe `0` y ese es el valor que se guarda (`UpdateProveedorCommand.cs:7`, tipo `decimal` sin valor por defecto; comprobado con la API: un `PUT` sin el campo dejó `tasaCumplimiento` en `0`, y luego se restauró a `88`). La unicidad se comprueba excluyendo al propio proveedor ([RN-PRV-007][PRV], [RN-PRV-008][PRV]).

**Respuesta correcta:** `204 No Content`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `Proveedor.NotFound` | No existe o está eliminado | RN-PRV-009 |
| 409 | `Proveedor.TaxId.Duplicado` / `Proveedor.RazonSocial.Duplicado` | Otro proveedor ya usa el valor | RN-PRV-007, RN-PRV-008 |
| 400 | `ProveedorDistribuidor.<Propiedad>` | Campo vacío, demasiado largo o fuera de rango | RN-PRV-002 a RN-PRV-006 |

**Ejemplo real:**

```http
PUT /api/v1/Proveedores/1593b5e4-17ba-4dff-ac77-e604ba2b986b
Authorization: Bearer <token>
Content-Type: application/json

{
  "razonSocial": "4C-Distribuidora Dos S.A.C.",
  "taxId": "4C-20100000002",
  "terminosFinancieros": "Contado",
  "calificacionEsg": 60.5,
  "tasaCumplimiento": 88
}
```

```http
HTTP/1.1 204 No Content
```

Un `GET` posterior devuelve `terminosFinancieros` `Contado`, `calificacionEsg` `60.5` y `tasaCumplimiento` `88`. Errores reales: usar el `taxId` del proveedor Uno → `409` `Proveedor.TaxId.Duplicado`; `calificacionEsg` `120` → `400` `ProveedorDistribuidor.CalificacionEsg` (`ESG Rating must be between 0 and 100.`); identificador inexistente → `404` `Proveedor.NotFound`.

#### `DELETE /api/v1/Proveedores/{id}`

Elimina (borrado lógico) un proveedor. Se bloquea si tiene precios de compra registrados (RN-PRV-010).

| | |
|---|---|
| **Permiso** | `Proveedores:Eliminar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProveedoresController.cs:69` → `DeleteProveedorCommand` |
| **Pantalla del SPA** | `/proveedores/:id` (botón «Eliminar»; ante un 409 muestra `proveedores.detail.delete-in-use`) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `200 OK` con el GUID eliminado como cuerpo (no 204).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `NotFound` | No existe o ya está eliminado (código genérico, `detail` «Record not found.») | RN-PRV-009 |
| 409 | `Proveedor.InUse` | Tiene algún registro no eliminado en el historial de precios (`DeleteProveedorCommandHandler.cs:24-31`) | RN-PRV-010 |

**Ejemplo real.** Proveedor sin precios (`4C-Proveedor temporal`):

```http
DELETE /api/v1/Proveedores/927a1cfa-3830-4b15-9e1c-9dc991db7acd
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
"927a1cfa-3830-4b15-9e1c-9dc991db7acd"
```

Un `GET` posterior responde `404` `Proveedor.NotFound`; un segundo `DELETE`:

```json
HTTP/1.1 404 Not Found
{
  "status": 404,
  "detail": "Record not found.",
  "code": "NotFound"
}
```

Caso real de bloqueo (`4C-Distribuidora Uno S.A.C.`, con dos precios):

```json
HTTP/1.1 409 Conflict
{
  "status": 409,
  "detail": "Cannot delete a supplier referenced by purchase prices.",
  "code": "Proveedor.InUse"
}
```

#### `POST /api/v1/Proveedores/exportar`

Genera un Excel con los proveedores que cumplen los filtros.

| | |
|---|---|
| **Permiso** | `Proveedores:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProveedoresController.cs:78` → `GetProveedoresQuery` (con `PageSize` = 10 000) |
| **Pantalla del SPA** | `/proveedores` (botón «Exportar») |

**Cuerpo** (`application/json`, `ProveedorExportRequest`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `headers` | arreglo de texto | Sí | Hasta 50 títulos de hasta 100 caracteres; si no, `400` con `errors.Headers`. |
| `razonSocial`, `taxId` | texto | No | Filtros «contiene». |
| `tasaCumplimientoDesde`, `tasaCumplimientoHasta` | decimal | No | Rango de tasa. |

**Respuesta correcta:** `200 OK`, `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `Content-Disposition: attachment; filename=proveedores.xlsx`. Hoja «Data»: títulos de `headers` y tres columnas por proveedor: razón social, identificador tributario y tasa de cumplimiento (no incluye términos financieros ni calificación ESG).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | (validación de modelo, `errors.Headers`) | `headers` ausente o fuera de límites | — |
| 400 | `Export.TooManyRows` | Más de 10 000 filas | [20, apartado 2.8][20]. Ejemplo no verificado: exige más de 10 000 proveedores |

**Ejemplo real:**

```http
POST /api/v1/Proveedores/exportar
Authorization: Bearer <token>
Content-Type: application/json

{
  "headers": ["Razón social", "RUC / Tax ID", "Tasa de cumplimiento"],
  "razonSocial": "4C-"
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=proveedores.xlsx; filename*=UTF-8''proveedores.xlsx

<archivo .xlsx de 6575 bytes>
```

Contenido leído (hoja «Data»): `4C-Distribuidora Dos S.A.C. | 4C-20100000002 | 88` y `4C-Distribuidora Uno S.A.C. | 4C-20100000001 | 95`.

#### `GET /api/v1/Proveedores/plantilla`

Descarga la plantilla Excel de carga masiva.

| | |
|---|---|
| **Permiso** | `Proveedores:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProveedoresController.cs:110` → `ExcelTemplateGenerator.GenerateTemplate` (sin comando) |
| **Pantalla del SPA** | `/proveedores/carga-masiva` (botón «Descargar plantilla») |

**Parámetros:** ninguno.

**Respuesta correcta:** `200 OK`, `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `Content-Disposition: attachment; filename=plantilla-proveedores.xlsx`. Hoja «Plantilla» con cinco títulos: `RazonSocial`, `TaxId`, `TerminosFinancieros`, `CalificacionEsg`, `TasaCumplimiento` (RN-PRV-013).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `Proveedores:Crear` | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/Proveedores/plantilla
Authorization: Bearer <token>
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=plantilla-proveedores.xlsx; filename*=UTF-8''plantilla-proveedores.xlsx

<archivo .xlsx de 6500 bytes>
```

#### `POST /api/v1/Proveedores/previsualizar`

Valida un Excel fila por fila sin guardar nada (paso 2 de [20, apartado 2.3][20]).

| | |
|---|---|
| **Permiso** | `Proveedores:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProveedoresController.cs:119` → `PreviewImportProveedoresQuery` |
| **Pantalla del SPA** | `/proveedores/carga-masiva` (al elegir o arrastrar el archivo) |

**Cuerpo** (`multipart/form-data`): campo `file`, `.xlsx` de hasta 5 MB (RN-PRV-017).

**Reglas por fila** (códigos de fila en [20, apartado 2.5][20]):

| Campo | Errores de fila | Regla |
|---|---|---|
| `razonSocial` | `required`, `max-length` (200). **No** detecta razones sociales repetidas (RN-PRV-015). | RN-PRV-002, RN-PRV-015 |
| `taxId` | `required`, `max-length` (50), `duplicate` (ya existe), `duplicate-in-file` | RN-PRV-003, RN-PRV-007, RN-PRV-014 |
| `terminosFinancieros` | `max-length` (100) | RN-PRV-004 |
| `calificacionEsg` | `invalid-decimal`, `invalid-range` (0 a 100); opcional | RN-PRV-005 |
| `tasaCumplimiento` | `required`, `invalid-decimal`, `invalid-range` (0 a 100) | RN-PRV-006 |

Los números se leen con cultura invariante (punto decimal). No verificado en vivo cómo se interpreta una coma decimal (en `4B` se documentó un caso equivalente en jerarquías).

**Respuesta correcta:** `200 OK`: `{ rows: [ { rowNumber, data, errors, isValid } ], totalCount, validCount, errorCount }`. En `data` los números llegan como texto.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | (validación de modelo, `errors.file`) | No se adjuntó archivo | RN-PRV-017 |
| 400 | `CargaMasiva.Archivo.Invalido` | Extensión distinta de `.xlsx` o contenido inválido | [20, apartado 2.6][20] |
| 400 | `CargaMasiva.Archivo.ExcedeDescompresion` / `CargaMasiva.Filas.Excedidas` | Archivo que se expande de más o con más de 5 000 filas | Ejemplo no verificado: exige archivos de ese tamaño |
| 413 | `Request.TooLarge` | Más de 5 MB | Ejemplo no verificado: exige más de 5 MB |

**Ejemplo real** (archivo de cinco filas: dos válidas, una con razón social ya existente, una con `taxId` ya existente y una con tasa `150`):

```http
POST /api/v1/Proveedores/previsualizar
Authorization: Bearer <token>
Content-Type: multipart/form-data; boundary=----4c

------4c
Content-Disposition: form-data; name="file"; filename="proveedores.xlsx"
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet

<contenido del archivo>
------4c--
```

```json
HTTP/1.1 200 OK
{
  "rows": [
    {
      "rowNumber": 2,
      "data": { "razonSocial": "4C-Proveedor Masivo A S.A.C.", "taxId": "4C-20100000101", "terminosFinancieros": "45 dias", "calificacionEsg": "70", "tasaCumplimiento": "90" },
      "errors": [],
      "isValid": true
    },
    {
      "rowNumber": 4,
      "data": { "razonSocial": "4C-Distribuidora Uno S.A.C.", "taxId": "4C-20100000103", "terminosFinancieros": null, "calificacionEsg": null, "tasaCumplimiento": "100" },
      "errors": [],
      "isValid": true
    },
    {
      "rowNumber": 5,
      "data": { "razonSocial": "4C-Taxid repetido S.A.C.", "taxId": "4C-20100000001", "terminosFinancieros": null, "calificacionEsg": null, "tasaCumplimiento": "100" },
      "errors": [ { "code": "duplicate", "field": "taxId", "params": { "value": "4C-20100000001" } } ],
      "isValid": false
    },
    {
      "rowNumber": 6,
      "data": { "razonSocial": "4C-Tasa fuera de rango S.A.C.", "taxId": "4C-20100000105", "terminosFinancieros": null, "calificacionEsg": null, "tasaCumplimiento": "150" },
      "errors": [ { "code": "invalid-range", "field": "tasaCumplimiento", "params": { "min": 0, "max": 100 } } ],
      "isValid": false
    }
  ],
  "totalCount": 5,
  "validCount": 3,
  "errorCount": 2
}
```

(La fila 3, válida, se omite.) La fila 4 muestra la limitación de RN-PRV-015: la razón social repetida se ve «válida» en la vista previa y se omite al guardar.

#### `POST /api/v1/Proveedores/carga-masiva`

Guarda las filas confirmadas tras la vista previa (paso 3).

| | |
|---|---|
| **Permiso** | `Proveedores:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProveedoresController.cs:137` → `CargaMasivaProveedoresCommand` |
| **Pantalla del SPA** | `/proveedores/carga-masiva` (botón «Guardar válidos (N)») |

**Cuerpo** (`application/json`, máximo 5 MB): arreglo de 1 a 5 000 filas.

| Campo de cada fila | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `razonSocial` | texto | Sí | No vacío, máximo 200 ([RN-PRV-016][PRV]) |
| `taxId` | texto | Sí | No vacío, máximo 50 |
| `terminosFinancieros` | texto | No | Máximo 100 |
| `calificacionEsg` | decimal | No | De 0 a 100 (lo valida la entidad) |
| `tasaCumplimiento` | decimal | Sí en la práctica | De 0 a 100; si se omite vale `0` (el campo del registro no admite nulo ni tiene valor por defecto) |

El servidor omite sin avisar las filas que la entidad rechaza o cuyo identificador tributario o razón social ya existen en la empresa o en una fila anterior (RN-PRV-016).

**Respuesta correcta:** `200 OK`, `{ "savedCount": <filas guardadas> }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `CargaMasivaProveedoresCommand.Rows` | Arreglo vacío o con más de 5 000 filas | RN-PRV-016 |
| 400 | `CargaMasivaProveedoresCommand.Rows[i].<Campo>` | `razonSocial`, `taxId` vacíos o de más largo que el máximo, o `terminosFinancieros` de más de 100, en la fila `i` | RN-PRV-016. Ejemplo propio no verificado; el mismo comportamiento se comprobó en `Medicamentos` |
| 413 | `Request.TooLarge` | Más de 5 MB | Ejemplo no verificado |

**Ejemplo real** (las cinco filas de la vista previa):

```http
POST /api/v1/Proveedores/carga-masiva
Authorization: Bearer <token>
Content-Type: application/json

[
  { "razonSocial": "4C-Proveedor Masivo A S.A.C.", "taxId": "4C-20100000101", "terminosFinancieros": "45 dias", "calificacionEsg": 70, "tasaCumplimiento": 90 },
  { "razonSocial": "4C-Proveedor Masivo B S.A.C.", "taxId": "4C-20100000102", "tasaCumplimiento": 100 },
  { "razonSocial": "4C-Distribuidora Uno S.A.C.", "taxId": "4C-20100000103", "tasaCumplimiento": 100 },
  { "razonSocial": "4C-Taxid repetido S.A.C.", "taxId": "4C-20100000001", "tasaCumplimiento": 100 },
  { "razonSocial": "4C-Tasa fuera de rango S.A.C.", "taxId": "4C-20100000105", "tasaCumplimiento": 150 }
]
```

```json
HTTP/1.1 200 OK
{ "savedCount": 2 }
```

Se guardaron las dos primeras; la razón social repetida, el identificador repetido y la tasa fuera de rango se omitieron sin error.

---

## 5. `Seguros`

Aseguradoras médicas (el módulo de la API se llama `Seguros` y la pantalla `Aseguradoras`). Archivo: `sofia-apis/src/SOFIA.API/Controllers/SegurosController.cs`. Ruta base: `/api/v1/Seguros`. Reglas: [RN-ASE-001 a RN-ASE-012][ASE]. Permisos: [05b, apartado 3.21][05B]. Pantallas: `/aseguradoras`, `/aseguradoras/nueva`, `/aseguradoras/:id` y `/aseguradoras/carga-masiva`. `GET /Seguros` lo usa también el punto de venta (`/pos/nueva`).

Este controlador se comporta distinto de los demás en varios puntos, todos comprobados con llamadas reales: no valida duplicados en el handler (los rechaza el índice único de la base de datos con `Db.Duplicado`), eliminar un registro inexistente responde 400, la exportación no limita los títulos de columna, y la vista previa no comprueba la extensión del archivo.

**Forma de la aseguradora** (`AseguradoraDto`, `SOFIA.Application/Seguros/Queries/GetAseguradoras/GetAseguradorasQuery.cs:9`):

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | GUID | Identificador. |
| `nombreComercial` | texto | Hasta 150 caracteres (RN-ASE-002). |
| `codigoIdentificadorNacional` | texto | Hasta 50 caracteres; único por empresa entre las no eliminadas (RN-ASE-003, RN-ASE-004). |

#### `POST /api/v1/Seguros`

Crea una aseguradora.

| | |
|---|---|
| **Permiso** | `Seguros:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/SegurosController.cs:19` → `CreateAseguradoraCommand` |
| **Pantalla del SPA** | `/aseguradoras/nueva` (botón «Guardar») |

**Cuerpo** (`application/json`). El validador del comando está vacío («TODO»); las reglas las aplica la entidad `AseguradoraMedica` (`SOFIA.Domain/Entities/AseguradoraMedica.cs:17-47`).

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `nombreComercial` | texto | Sí | No vacío, máximo 150 ([RN-ASE-002][ASE]). |
| `codigoIdentificadorNacional` | texto | Sí | No vacío, máximo 50; único entre las no eliminadas ([RN-ASE-003][ASE], [RN-ASE-004][ASE]). |

**Respuesta correcta:** `200 OK` (no 201, RN-ASE-007), cuerpo = GUID de la aseguradora (texto JSON), sin `Location`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `AseguradoraMedica.NombreComercial` / `AseguradoraMedica.CodigoIdentificadorNacional` | Campo vacío o demasiado largo | RN-ASE-002, RN-ASE-003 |
| 409 | `Db.Duplicado` | El código ya lo usa otra aseguradora no eliminada (lo detecta el índice único `UX_Aseguradoras_Medicas_Tenant_Codigo_Identificador_Nacional`, no el handler) | RN-ASE-004 |

**Ejemplo real:**

```http
POST /api/v1/Seguros
Authorization: Bearer <token>
Content-Type: application/json

{
  "nombreComercial": "4C-Seguro Salud Uno",
  "codigoIdentificadorNacional": "4C-SEG-001"
}
```

```json
HTTP/1.1 200 OK
"55216aab-c0e5-49ab-8079-8353405390c6"
```

Errores reales. Código repetido con otro nombre (nótese que el cuerpo es `application/json`, no `application/problem+json`, y trae `traceId`):

```json
HTTP/1.1 409 Conflict
{
  "status": 409,
  "detail": "A record with the same unique value already exists.",
  "code": "Db.Duplicado",
  "traceId": "0HNPMVVCIH2GC:00000004"
}
```

Nombre vacío: `400`, `AseguradoraMedica.NombreComercial`, `Nombre comercial is required.`. Código de más de 50 caracteres: `400`, `AseguradoraMedica.CodigoIdentificadorNacional`, `Código identificador nacional must not exceed 50 characters.`.

#### `GET /api/v1/Seguros`

Lista las aseguradoras con filtros y paginación. No define un orden explícito (RN-ASE-008).

| | |
|---|---|
| **Permiso** | `Seguros:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/SegurosController.cs:27` → `GetAseguradorasQuery` |
| **Pantalla del SPA** | `/aseguradoras` (botón «Buscar») |

**Parámetros de consulta:**

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `nombreComercial` | texto | No | Contiene este texto. |
| `codigoIdentificadorNacional` | texto | No | Contiene este texto. |
| `pageNumber` | entero | No | Página, por defecto 1. |
| `pageSize` | entero | No | Tamaño de página, por defecto 10. |

**Respuesta correcta:** `200 OK`: `{ items: [ AseguradoraDto ], pageNumber, totalPages, totalCount, hasPreviousPage, hasNextPage }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `Seguros:Leer` | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/Seguros?nombreComercial=4C-&pageNumber=1&pageSize=2
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "items": [
    { "id": "55216aab-c0e5-49ab-8079-8353405390c6", "nombreComercial": "4C-Seguro Salud Uno", "codigoIdentificadorNacional": "4C-SEG-001" },
    { "id": "4a0c16fe-54ff-40a5-8c7e-935558d9883d", "nombreComercial": "4C-Seguro Salud Dos", "codigoIdentificadorNacional": "4C-SEG-002" }
  ],
  "pageNumber": 1,
  "totalPages": 2,
  "totalCount": 3,
  "hasPreviousPage": false,
  "hasNextPage": true
}
```

#### `GET /api/v1/Seguros/{id}`

Devuelve una aseguradora.

| | |
|---|---|
| **Permiso** | `Seguros:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/SegurosController.cs:34` → `GetAseguradoraByIdQuery` |
| **Pantalla del SPA** | `/aseguradoras/:id` (carga inicial) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `200 OK`, `AseguradoraDto`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `Aseguradora.NotFound` | No existe o está eliminada | RN-ASE-005 |
| 401 / 403 | — | Sin token o sin permiso | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/Seguros/55216aab-c0e5-49ab-8079-8353405390c6
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "id": "55216aab-c0e5-49ab-8079-8353405390c6",
  "nombreComercial": "4C-Seguro Salud Uno",
  "codigoIdentificadorNacional": "4C-SEG-001"
}
```

Error real (`GET /api/v1/Seguros/00000000-0000-0000-0000-000000000099`): `404`, `Aseguradora.NotFound`, `Aseguradora with ID 00000000-0000-0000-0000-000000000099 was not found.`.

#### `PUT /api/v1/Seguros/{id}`

Actualiza el nombre y el código de una aseguradora.

| | |
|---|---|
| **Permiso** | `Seguros:Actualizar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/SegurosController.cs:42` → `UpdateAseguradoraCommand` |
| **Pantalla del SPA** | `/aseguradoras/:id` (botón «Guardar» en modo edición) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Cuerpo:** `nombreComercial` y `codigoIdentificadorNacional`, con las mismas reglas que en `POST`. Un `id` en el cuerpo se ignora.

**Respuesta correcta:** `204 No Content`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `Aseguradora.NotFound` | No existe o está eliminada | RN-ASE-005 |
| 400 | `AseguradoraMedica.NombreComercial` / `AseguradoraMedica.CodigoIdentificadorNacional` | Campo vacío o demasiado largo | RN-ASE-002, RN-ASE-003 |
| 409 | `Db.Duplicado` | El código ya lo usa otra aseguradora | RN-ASE-004 |

**Ejemplo real:**

```http
PUT /api/v1/Seguros/4a0c16fe-54ff-40a5-8c7e-935558d9883d
Authorization: Bearer <token>
Content-Type: application/json

{
  "nombreComercial": "4C-Seguro Salud Dos (editado)",
  "codigoIdentificadorNacional": "4C-SEG-002"
}
```

```http
HTTP/1.1 204 No Content
```

Errores reales: con el código `4C-SEG-001` (de otra aseguradora) → `409` `Db.Duplicado`; con `codigoIdentificadorNacional` vacío → `400` `AseguradoraMedica.CodigoIdentificadorNacional` (`Código identificador nacional is required.`); identificador inexistente → `404` `Aseguradora.NotFound`.

#### `DELETE /api/v1/Seguros/{id}`

Elimina (borrado lógico) una aseguradora. No valida dependencias, por ejemplo reclamos de seguro asociados (RN-ASE-006).

| | |
|---|---|
| **Permiso** | `Seguros:Eliminar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/SegurosController.cs:51` → `DeleteAseguradoraCommand` |
| **Pantalla del SPA** | `/aseguradoras/:id` (botón «Eliminar») |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `200 OK` con el GUID eliminado como cuerpo (no 204).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `NotFound` | No existe o ya está eliminada. El handler no indica el código HTTP y el valor por defecto de un fallo es 400, no 404 (`DeleteAseguradoraCommandHandler.cs:21`) | RN-ASE-005 |

**Ejemplo real** (aseguradora temporal `4C-SEG-TMP`):

```http
DELETE /api/v1/Seguros/21683631-921a-4e89-b222-b45eacb569f8
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
"21683631-921a-4e89-b222-b45eacb569f8"
```

Un `GET` posterior responde `404` `Aseguradora.NotFound`. Un segundo `DELETE`:

```json
HTTP/1.1 400 Bad Request
{
  "status": 400,
  "detail": "Record not found.",
  "code": "NotFound"
}
```

#### `POST /api/v1/Seguros/exportar`

Genera un Excel con las aseguradoras que cumplen los filtros.

| | |
|---|---|
| **Permiso** | `Seguros:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/SegurosController.cs:60` → `GetAseguradorasQuery` (con `PageSize` = 10 000) |
| **Pantalla del SPA** | `/aseguradoras` (botón «Exportar») |

**Cuerpo** (`application/json`, `AseguradoraExportRequest`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `headers` | arreglo de texto | Sí | **Sin límite de cantidad ni de longitud**: a diferencia de los demás controladores, este `record` no lleva `[ExportHeaders]` (`SegurosController.cs:122`). Solo es obligatorio (si falta, `400` con `errors.Headers`). |
| `nombreComercial`, `codigoIdentificadorNacional` | texto | No | Filtros «contiene». |

**Respuesta correcta:** `200 OK`, `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `Content-Disposition: attachment; filename=aseguradoras.xlsx`. Hoja «Data»: títulos de `headers` y dos columnas por aseguradora: nombre comercial y código identificador nacional.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | (validación de modelo, `errors.Headers`: `The Headers field is required.`) | `headers` ausente | — |
| 400 | `Export.TooManyRows` | Más de 10 000 filas | [20, apartado 2.8][20]. Ejemplo no verificado: exige más de 10 000 aseguradoras |

**Ejemplo real:**

```http
POST /api/v1/Seguros/exportar
Authorization: Bearer <token>
Content-Type: application/json

{
  "headers": ["Nombre comercial", "Código identificador nacional"],
  "nombreComercial": "4C-"
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=aseguradoras.xlsx; filename*=UTF-8''aseguradoras.xlsx

<archivo .xlsx de 6534 bytes>
```

Contenido leído (hoja «Data»): `4C-Seguro Salud Uno | 4C-SEG-001` y `4C-Seguro Salud Dos (editado) | 4C-SEG-002`. Con 51 títulos de columna este endpoint respondió `200` (6 969 bytes); el mismo envío a `ProfesionalesSalud` respondió `400` con `errors.Headers: ["Export.Headers.Invalid"]`.

#### `GET /api/v1/Seguros/plantilla`

Descarga la plantilla Excel de carga masiva.

| | |
|---|---|
| **Permiso** | `Seguros:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/SegurosController.cs:89` → `ExcelTemplateGenerator.GenerateTemplate` (sin comando) |
| **Pantalla del SPA** | `/aseguradoras/carga-masiva` (botón «Descargar plantilla») |

**Parámetros:** ninguno.

**Respuesta correcta:** `200 OK`, `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `Content-Disposition: attachment; filename=plantilla-seguros.xlsx`. Hoja «Plantilla» con dos títulos: `NombreComercial`, `CodigoIdentificadorNacional` (RN-ASE-009).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `Seguros:Crear` | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/Seguros/plantilla
Authorization: Bearer <token>
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=plantilla-seguros.xlsx; filename*=UTF-8''plantilla-seguros.xlsx

<archivo .xlsx de 6423 bytes>
```

#### `POST /api/v1/Seguros/previsualizar`

Valida un Excel fila por fila sin guardar nada (paso 2 de [20, apartado 2.3][20]).

| | |
|---|---|
| **Permiso** | `Seguros:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/SegurosController.cs:98` → `PreviewImportSegurosQuery` |
| **Pantalla del SPA** | `/aseguradoras/carga-masiva` (al elegir o arrastrar el archivo) |

**Cuerpo** (`multipart/form-data`): campo `file`, archivo `.xlsx`. Diferencias con los demás controladores de carga: el método **no** declara `[RequestSizeLimit]` ni `[RequestFormLimits]` (no aplica el tope de 5 MB propio) y llama al lector sin el nombre del archivo, por lo que **no se comprueba la extensión**: lo único que se exige es que el contenido sea un libro `.xlsx` (RN-ASE-012).

**Reglas por fila** (códigos de fila en [20, apartado 2.5][20]):

| Campo | Errores de fila | Regla |
|---|---|---|
| `nombreComercial` | `required`, `max-length` (se valida contra 150, pero el parámetro `max` informado es 200: defecto, ver el apartado 8.3) | RN-ASE-002 |
| `codigoIdentificadorNacional` | `required`, `max-length` (50), `duplicate` (ya existe en la empresa), `duplicate-in-file` | RN-ASE-003, RN-ASE-010 |

**Respuesta correcta:** `200 OK`: `{ rows: [ { rowNumber, data, errors, isValid } ], totalCount, validCount, errorCount }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | (validación de modelo, `errors.file`) | No se adjuntó archivo | RN-ASE-012 |
| 400 | `CargaMasiva.Archivo.Invalido` | El contenido no es un `.xlsx` válido (la extensión no cuenta) | RN-ASE-012. Ejemplo no verificado en este controlador; el error se comprobó en `Medicamentos` |
| 400 | `CargaMasiva.Archivo.ExcedeDescompresion` / `CargaMasiva.Filas.Excedidas` | Archivo que se expande de más o con más de 5 000 filas | Ejemplo no verificado |

**Ejemplo real** (cuatro filas: dos válidas, una con un código ya existente y una sin nombre):

```http
POST /api/v1/Seguros/previsualizar
Authorization: Bearer <token>
Content-Type: multipart/form-data; boundary=----4c

------4c
Content-Disposition: form-data; name="file"; filename="seguros.xlsx"
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet

<contenido del archivo>
------4c--
```

```json
HTTP/1.1 200 OK
{
  "rows": [
    {
      "rowNumber": 2,
      "data": { "nombreComercial": "4C-Seguro Masivo A", "codigoIdentificadorNacional": "4C-SEG-101" },
      "errors": [],
      "isValid": true
    },
    {
      "rowNumber": 4,
      "data": { "nombreComercial": "4C-Codigo repetido", "codigoIdentificadorNacional": "4C-SEG-001" },
      "errors": [ { "code": "duplicate", "field": "codigoIdentificadorNacional", "params": { "value": "4C-SEG-001" } } ],
      "isValid": false
    },
    {
      "rowNumber": 5,
      "data": { "nombreComercial": null, "codigoIdentificadorNacional": "4C-SEG-104" },
      "errors": [ { "code": "required", "field": "nombreComercial", "params": null } ],
      "isValid": false
    }
  ],
  "totalCount": 4,
  "validCount": 2,
  "errorCount": 2
}
```

(La fila 3, válida, se omite.) El mismo archivo enviado con el nombre `datos.csv` devolvió la misma vista previa con `200`.

#### `POST /api/v1/Seguros/carga-masiva`

Guarda las filas confirmadas tras la vista previa (paso 3).

| | |
|---|---|
| **Permiso** | `Seguros:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/SegurosController.cs:114` → `CargaMasivaSegurosCommand` |
| **Pantalla del SPA** | `/aseguradoras/carga-masiva` (botón «Guardar válidos (N)») |

**Cuerpo** (`application/json`): arreglo de filas `{ nombreComercial, codigoIdentificadorNacional }`. A diferencia de los demás módulos, **este comando no tiene validador**: no exige entre 1 y 5 000 filas, no aplica un tope de 5 MB propio y no omite duplicados (RN-ASE-011). Solo las filas que no cumplen los guards de la entidad (campos vacíos o demasiado largos) se omiten sin error.

**Respuesta correcta:** `200 OK`, `{ "savedCount": <filas guardadas> }`; con un arreglo vacío responde `{ "savedCount": 0 }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 409 | `Db.Duplicado` | Alguna fila repite un código ya existente o repetido dentro del arreglo: falla todo el guardado y **no se guarda ninguna fila** | RN-ASE-011 |

**Ejemplo real.** Con una fila válida y otra que repite `4C-SEG-001`:

```http
POST /api/v1/Seguros/carga-masiva
Authorization: Bearer <token>
Content-Type: application/json

[
  { "nombreComercial": "4C-Seguro Masivo A", "codigoIdentificadorNacional": "4C-SEG-101" },
  { "nombreComercial": "4C-Codigo repetido", "codigoIdentificadorNacional": "4C-SEG-001" }
]
```

```json
HTTP/1.1 409 Conflict
{
  "status": 409,
  "detail": "A record with the same unique value already exists.",
  "code": "Db.Duplicado",
  "traceId": "0HNPMVVCIH2GQ:0000000A"
}
```

Con tres filas (`4C-SEG-101`, `4C-SEG-102` y una sin nombre comercial) la respuesta fue:

```json
HTTP/1.1 200 OK
{ "savedCount": 2 }
```

Que la fila `4C-SEG-101` se guardara en esta segunda llamada, sin repetirse, confirma que la primera no guardó nada.

---

## 6. `Pacientes`

Pacientes o clientes de la empresa, identificados por un documento de identidad (catálogo 06 de SUNAT). Archivo: `sofia-apis/src/SOFIA.API/Controllers/PacientesController.cs`. Ruta base: `/api/v1/Pacientes`. Reglas: [RN-PAC-001 a RN-PAC-017][PAC]. Permisos: [05b, apartado 3.15][05B]. Pantallas: `/pacientes`, `/pacientes/nueva`, `/pacientes/:id` y `/pacientes/carga-masiva`. `GET /Pacientes` lo usan además como autocompletado el punto de venta, el listado de ventas y las pantallas de recetas médicas (05b, apartado 3.15).

**Forma del paciente** (`PacienteDto`, `SOFIA.Application/Pacientes/Queries/GetPacientes/GetPacientesQuery.cs:9`):

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | GUID | Identificador. |
| `tipoDocumento` | texto o `null` | Código del catálogo 06: `"1"` DNI, `"4"` carné de extranjería, `"6"` RUC, `"7"` pasaporte. `null` solo en pacientes anteriores a ese campo (RN-PAC-002). |
| `docIdentidadGub` | texto | Número de documento, hasta 50 caracteres; único por empresa entre los no eliminados, sin importar el tipo (RN-PAC-008). |
| `nombreApellidos` | texto | Hasta 200 caracteres. |
| `fechaNacimiento` | fecha | `yyyy-MM-dd`; no puede ser futura. |
| `contactoPrimario` | texto o `null` | Hasta 100 caracteres. |

**Formato del número según el tipo** (`TipoDocumentoIdentidad.cs:50-60`, RN-PAC-004): DNI, exactamente 8 dígitos; RUC, exactamente 11 dígitos; carné de extranjería, de 1 a 12 letras o dígitos; pasaporte, de 1 a 20 letras o dígitos.

#### `POST /api/v1/Pacientes`

Crea un paciente.

| | |
|---|---|
| **Permiso** | `Pacientes:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PacientesController.cs:20` → `CreatePacienteCommand` |
| **Pantalla del SPA** | `/pacientes/nueva` (botón «Guardar») |

**Cuerpo** (`application/json`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `tipoDocumento` | texto | Sí | `"1"`, `"4"`, `"6"` o `"7"` ([RN-PAC-002][PAC]). |
| `docIdentidadGub` | texto | Sí | No vacío, máximo 50, con el formato de su tipo; único ([RN-PAC-003][PAC], [RN-PAC-004][PAC], [RN-PAC-008][PAC]). |
| `nombreApellidos` | texto | Sí | No vacío, máximo 200 ([RN-PAC-005][PAC]). |
| `fechaNacimiento` | fecha `yyyy-MM-dd` | Sí | No puede ser futura; la valida la entidad ([RN-PAC-006][PAC]). |
| `contactoPrimario` | texto | No | Máximo 100 ([RN-PAC-007][PAC]). |

**Respuesta correcta:** `201 Created`, cuerpo = GUID del paciente y `Location` hacia `GET /api/v1/Pacientes/{id}`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `CreatePacienteCommand.TipoDocumento` | Tipo vacío o fuera de `1`/`4`/`6`/`7` (el `detail` une las demás fallas del validador) | RN-PAC-002 |
| 400 | `CreatePacienteCommand.DocIdentidadGub` | Número vacío, de más de 50 caracteres o que no cumple el formato de su tipo | RN-PAC-003, RN-PAC-004 |
| 400 | `CreatePacienteCommand.NombreApellidos` / `.ContactoPrimario` | Nombre vacío o de más de 200; contacto de más de 100 | RN-PAC-005, RN-PAC-007 |
| 400 | `PacienteCliente.FechaNacimiento` | Fecha futura (o no enviada) | RN-PAC-006 |
| 409 | `Paciente.DocIdentidadGub.Duplicado` | Otro paciente no eliminado usa el número de documento | RN-PAC-008 |

**Ejemplo real:**

```http
POST /api/v1/Pacientes
Authorization: Bearer <token>
Content-Type: application/json

{
  "tipoDocumento": "1",
  "docIdentidadGub": "99900001",
  "nombreApellidos": "4C-Paciente Uno Ficticio",
  "fechaNacimiento": "1985-03-14",
  "contactoPrimario": "999000001"
}
```

```json
HTTP/1.1 201 Created
"cef89078-43d8-46f6-85c3-f6c51f5ac290"
```

Errores reales. Mismo documento con otro nombre:

```json
HTTP/1.1 409 Conflict
{
  "status": 409,
  "detail": "Another patient already uses this identity document.",
  "code": "Paciente.DocIdentidadGub.Duplicado"
}
```

DNI de 4 dígitos (`"docIdentidadGub": "1234"`):

```json
HTTP/1.1 400 Bad Request
{
  "status": 400,
  "detail": "DocIdentidadGub does not match the format of its document type.",
  "code": "CreatePacienteCommand.DocIdentidadGub"
}
```

Tipo `"9"`, documento y nombre vacíos: `400`, `CreatePacienteCommand.TipoDocumento`, `detail` `TipoDocumento must be one of: 1 (DNI), 4 (CE), 6 (RUC), 7 (Pasaporte).; 'Doc Identidad Gub' must not be empty.; 'Nombre Apellidos' must not be empty.`. Fecha `2999-01-01`: `400`, `PacienteCliente.FechaNacimiento`, `Fecha de Nacimiento cannot be in the future.`.

#### `GET /api/v1/Pacientes`

Lista los pacientes con filtros y paginación. Ordena por nombre y luego por identificador (RN-PAC-011).

| | |
|---|---|
| **Permiso** | `Pacientes:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PacientesController.cs:30` → `GetPacientesQuery` |
| **Pantalla del SPA** | `/pacientes` (botón «Buscar») |

**Parámetros de consulta:**

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `tipoDocumento` | texto | No | Coincidencia exacta con el código (`1`, `4`, `6`, `7`). |
| `docIdentidadGub` | texto | No | Contiene este texto. |
| `nombreApellidos` | texto | No | Contiene este texto. |
| `fechaNacimientoDesde` | fecha `yyyy-MM-dd` | No | Fecha de nacimiento mayor o igual. |
| `fechaNacimientoHasta` | fecha `yyyy-MM-dd` | No | Fecha de nacimiento menor o igual. |
| `pageNumber` | entero | No | Página, por defecto 1. |
| `pageSize` | entero | No | Tamaño de página, por defecto 10. |

**Respuesta correcta:** `200 OK`: `{ items: [ PacienteDto ], pageNumber, totalPages, totalCount, hasPreviousPage, hasNextPage }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `Pacientes:Leer` | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/Pacientes?nombreApellidos=4C-&fechaNacimientoDesde=1989-01-01&fechaNacimientoHasta=2001-12-31
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "items": [
    {
      "id": "d33952c4-51e0-446d-b3be-75b9f21b6c76",
      "tipoDocumento": "7",
      "docIdentidadGub": "4CPAS0002",
      "nombreApellidos": "4C-Paciente Dos Ficticio",
      "fechaNacimiento": "1990-11-02",
      "contactoPrimario": null
    },
    {
      "id": "b233c990-7e14-4125-b196-1116490c7c87",
      "tipoDocumento": "4",
      "docIdentidadGub": "4CCE0003",
      "nombreApellidos": "4C-Paciente temporal",
      "fechaNacimiento": "2000-01-31",
      "contactoPrimario": null
    }
  ],
  "pageNumber": 1,
  "totalPages": 1,
  "totalCount": 2,
  "hasPreviousPage": false,
  "hasNextPage": false
}
```

#### `GET /api/v1/Pacientes/{id}`

Devuelve un paciente.

| | |
|---|---|
| **Permiso** | `Pacientes:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PacientesController.cs:37` → `GetPacienteByIdQuery` |
| **Pantalla del SPA** | `/pacientes/:id` (carga inicial) |

**Parámetros de ruta:** `id` (GUID, obligatorio). A diferencia de los demás controladores de esta parte, la ruta de `Pacientes` (`{id}`) no declara la restricción `:guid`: un texto que no es GUID llega al método y falla el enlace del modelo con `400` (`errors.id: ["The value 'no-es-guid' is not valid."]`, comprobado), mientras que en `Medicamentos` el mismo valor no coincide con la ruta y responde `404` sin cuerpo (comprobado). Lo mismo vale para `PUT` y `DELETE` de `Pacientes`.

**Respuesta correcta:** `200 OK`, `PacienteDto`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `Paciente.NotFound` | No existe o está eliminado | RN-PAC-009 |
| 400 | (validación de modelo, `errors.id`) | El `id` no es un GUID | — |
| 401 / 403 | — | Sin token o sin permiso | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/Pacientes/cef89078-43d8-46f6-85c3-f6c51f5ac290
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "id": "cef89078-43d8-46f6-85c3-f6c51f5ac290",
  "tipoDocumento": "1",
  "docIdentidadGub": "99900001",
  "nombreApellidos": "4C-Paciente Uno Ficticio",
  "fechaNacimiento": "1985-03-14",
  "contactoPrimario": "999000001"
}
```

Error real (`GET /api/v1/Pacientes/00000000-0000-0000-0000-000000000099`): `404`, `Paciente.NotFound`, `Paciente with ID 00000000-0000-0000-0000-000000000099 was not found.`.

#### `PUT /api/v1/Pacientes/{id}`

Actualiza todos los datos de un paciente.

| | |
|---|---|
| **Permiso** | `Pacientes:Actualizar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PacientesController.cs:45` → `UpdatePacienteCommand` |
| **Pantalla del SPA** | `/pacientes/:id` (botón «Guardar» en modo edición) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Cuerpo:** los mismos cinco campos y reglas que en `POST`. El tipo de documento es obligatorio también al editar, de modo que un paciente anterior sin tipo (`null`) debe recibir uno para poder guardarse (RN-PAC-002). La unicidad excluye al propio paciente (RN-PAC-008).

**Respuesta correcta:** `204 No Content`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `UpdatePacienteCommand.<Propiedad>` | Mismas validaciones que al crear | RN-PAC-002 a RN-PAC-007 |
| 404 | `Paciente.NotFound` | No existe o está eliminado | RN-PAC-009 |
| 409 | `Paciente.DocIdentidadGub.Duplicado` | Otro paciente usa el documento | RN-PAC-008 |

**Ejemplo real:**

```http
PUT /api/v1/Pacientes/d33952c4-51e0-446d-b3be-75b9f21b6c76
Authorization: Bearer <token>
Content-Type: application/json

{
  "tipoDocumento": "7",
  "docIdentidadGub": "4CPAS0002",
  "nombreApellidos": "4C-Paciente Dos Ficticio (editado)",
  "fechaNacimiento": "1990-11-02",
  "contactoPrimario": "paciente2@ejemplo.invalid"
}
```

```http
HTTP/1.1 204 No Content
```

Errores reales: con el documento `99900001` de otro paciente → `409` `Paciente.DocIdentidadGub.Duplicado`; con `"tipoDocumento": null` → `400`, `UpdatePacienteCommand.TipoDocumento`, `'Tipo Documento' must not be empty.; TipoDocumento must be one of: 1 (DNI), 4 (CE), 6 (RUC), 7 (Pasaporte).`; identificador inexistente → `404` `Paciente.NotFound`.

#### `DELETE /api/v1/Pacientes/{id}`

Elimina (borrado lógico) un paciente. Se bloquea si alguna receta, venta, servicio de agenda o servicio clínico de inmunización lo referencia (RN-PAC-010).

| | |
|---|---|
| **Permiso** | `Pacientes:Eliminar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PacientesController.cs:54` → `DeletePacienteCommand` |
| **Pantalla del SPA** | `/pacientes/:id` (botón «Eliminar»; ante un 409 muestra `pacientes.detail.delete-in-use`) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `200 OK` con el GUID eliminado como cuerpo (no 204).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `NotFound` | No existe o ya está eliminado (código genérico, `detail` «Record not found.») | RN-PAC-009 |
| 409 | `Paciente.InUse` | Lo referencia una receta, una venta, un servicio de agenda o un servicio clínico de inmunización, no eliminados (`DeletePacienteCommandHandler.cs:37-41`) | RN-PAC-010 |

**Ejemplo real.** Paciente sin dependencias (`4CCE0003`):

```http
DELETE /api/v1/Pacientes/b233c990-7e14-4125-b196-1116490c7c87
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
"b233c990-7e14-4125-b196-1116490c7c87"
```

Un `GET` posterior responde `404` `Paciente.NotFound` y un segundo `DELETE` responde `404` con `code` `NotFound` y `detail` `Record not found.`.

Caso real de bloqueo: al paciente con DNI `99900001` se le creó una receta con `POST /api/v1/Recetas` (profesional `4C-CMP-0001`, fecha de expedición `2026-10-29`; ese endpoint se documenta en la parte D).

```http
DELETE /api/v1/Pacientes/cef89078-43d8-46f6-85c3-f6c51f5ac290
Authorization: Bearer <token>
```

```json
HTTP/1.1 409 Conflict
{
  "status": 409,
  "detail": "Cannot delete a patient referenced by prescriptions, sales, appointments or immunizations.",
  "code": "Paciente.InUse"
}
```

#### `POST /api/v1/Pacientes/exportar`

Genera un Excel con los pacientes que cumplen los filtros.

| | |
|---|---|
| **Permiso** | `Pacientes:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PacientesController.cs:63` → `GetPacientesQuery` (con `PageSize` = 10 000) |
| **Pantalla del SPA** | `/pacientes` (botón «Exportar») |

**Cuerpo** (`application/json`, `PacienteExportRequest`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `headers` | arreglo de texto | Sí | Hasta 50 títulos de hasta 100 caracteres; si no, `400` con `errors.Headers`. |
| `dateFormat` | texto | Sí | Uno de `dd/MM/yyyy`, `MM/dd/yyyy`, `dd/MM/yyyy HH:mm`, `MM/dd/yyyy HH:mm` (`ExportDateFormatAttribute`); da formato a la fecha de nacimiento. |
| `tipoDocumento`, `docIdentidadGub`, `nombreApellidos` | texto | No | Mismos filtros del listado. |
| `fechaNacimientoDesde`, `fechaNacimientoHasta` | fecha `yyyy-MM-dd` | No | Rango de fecha de nacimiento. |

**Respuesta correcta:** `200 OK`, `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `Content-Disposition: attachment; filename=pacientes.xlsx`. Hoja «Data»: títulos de `headers` y cinco columnas por paciente: documento, tipo (abreviatura `DNI`, `CE`, `RUC` o `PASAPORTE`), nombre y apellidos, fecha de nacimiento con el formato pedido y contacto.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | (validación de modelo, `errors.DateFormat`: `Export.DateFormat.Invalid`) | `dateFormat` ausente o distinto de los cuatro permitidos | — |
| 400 | (validación de modelo, `errors.Headers`) | `headers` ausente o fuera de límites | — |
| 400 | `Export.TooManyRows` | Más de 10 000 filas | [20, apartado 2.8][20]. Ejemplo no verificado: exige más de 10 000 pacientes |

**Ejemplo real:**

```http
POST /api/v1/Pacientes/exportar
Authorization: Bearer <token>
Content-Type: application/json

{
  "headers": ["Documento", "Tipo", "Nombre y apellidos", "Fecha de nacimiento", "Contacto"],
  "dateFormat": "dd/MM/yyyy",
  "nombreApellidos": "4C-"
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=pacientes.xlsx; filename*=UTF-8''pacientes.xlsx

<archivo .xlsx de 6676 bytes>
```

Contenido leído (hoja «Data»): `4CPAS0002 | PASAPORTE | 4C-Paciente Dos Ficticio (editado) | 02/11/1990 | paciente2@ejemplo.invalid` y `99900001 | DNI | 4C-Paciente Uno Ficticio | 14/03/1985 | 999000001`. Error real (con `"dateFormat": "yyyy"`):

```json
HTTP/1.1 400 Bad Request
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": { "DateFormat": ["Export.DateFormat.Invalid"] },
  "traceId": "00-…"
}
```

#### `GET /api/v1/Pacientes/plantilla`

Descarga la plantilla Excel de carga masiva.

| | |
|---|---|
| **Permiso** | `Pacientes:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PacientesController.cs:98` → `ExcelTemplateGenerator.GenerateTemplate` (sin comando) |
| **Pantalla del SPA** | `/pacientes/carga-masiva` (botón «Descargar plantilla») |

**Parámetros:** ninguno.

**Respuesta correcta:** `200 OK`, `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `Content-Disposition: attachment; filename=plantilla-pacientes.xlsx`. Hoja «Plantilla» con cinco títulos: `TipoDocumento`, `DocIdentidadGub`, `NombreApellidos`, `FechaNacimiento`, `ContactoPrimario` (RN-PAC-012).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `Pacientes:Crear` | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/Pacientes/plantilla
Authorization: Bearer <token>
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=plantilla-pacientes.xlsx; filename*=UTF-8''plantilla-pacientes.xlsx

<archivo .xlsx de 6492 bytes>
```

#### `POST /api/v1/Pacientes/previsualizar`

Valida un Excel fila por fila sin guardar nada (paso 2 de [20, apartado 2.3][20]).

| | |
|---|---|
| **Permiso** | `Pacientes:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PacientesController.cs:107` → `PreviewImportPacientesQuery` |
| **Pantalla del SPA** | `/pacientes/carga-masiva` (al elegir o arrastrar el archivo) |

**Cuerpo** (`multipart/form-data`): campo `file`, `.xlsx` de hasta 5 MB (RN-PAC-017).

**Reglas por fila** (códigos de fila en [20, apartado 2.5][20]):

| Campo | Errores de fila | Regla |
|---|---|---|
| `tipoDocumento` | `required`, `invalid-option`. Se acepta el código (`1`, `4`, `6`, `7`) o la abreviatura `DNI`, `CE`, `RUC`, `PASAPORTE`, sin distinguir mayúsculas | RN-PAC-002, RN-PAC-013 |
| `docIdentidadGub` | `required`, `invalid-document-number` (no cumple el formato de su tipo), `max-length` (50), `duplicate`, `duplicate-in-file` | RN-PAC-003, RN-PAC-004, RN-PAC-008, RN-PAC-015 |
| `nombreApellidos` | `required`, `max-length` (200) | RN-PAC-005 |
| `fechaNacimiento` | `required`, `invalid-date` (debe ser exactamente `dd/MM/yyyy`), `future-date` | RN-PAC-006, RN-PAC-014 |
| `contactoPrimario` | `max-length` (100) | RN-PAC-007 |

**Respuesta correcta:** `200 OK`: `{ rows: [ { rowNumber, data, errors, isValid } ], totalCount, validCount, errorCount }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | (validación de modelo, `errors.file`) | No se adjuntó archivo | RN-PAC-017 |
| 400 | `CargaMasiva.Archivo.Invalido` | Extensión distinta de `.xlsx` o contenido inválido | [20, apartado 2.6][20]. Ejemplo no verificado en este controlador; el error se comprobó en `Medicamentos` |
| 400 | `CargaMasiva.Archivo.ExcedeDescompresion` / `CargaMasiva.Filas.Excedidas` | Archivo que se expande de más o con más de 5 000 filas | Ejemplo no verificado |
| 413 | `Request.TooLarge` | Más de 5 MB | Ejemplo no verificado |

**Ejemplo real** (archivo de siete filas: dos válidas y una de cada error comprobado):

```http
POST /api/v1/Pacientes/previsualizar
Authorization: Bearer <token>
Content-Type: multipart/form-data; boundary=----4c

------4c
Content-Disposition: form-data; name="file"; filename="pacientes.xlsx"
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet

<contenido del archivo>
------4c--
```

```json
HTTP/1.1 200 OK
{
  "rows": [
    {
      "rowNumber": 2,
      "data": { "tipoDocumento": "DNI", "docIdentidadGub": "99900101", "nombreApellidos": "4C-Paciente Masivo A", "fechaNacimiento": "15/06/1980", "contactoPrimario": "999000101" },
      "errors": [],
      "isValid": true
    },
    {
      "rowNumber": 4,
      "data": { "tipoDocumento": "DNI", "docIdentidadGub": "99900001", "nombreApellidos": "4C-Documento repetido", "fechaNacimiento": "01/01/1990", "contactoPrimario": null },
      "errors": [ { "code": "duplicate", "field": "docIdentidadGub", "params": { "value": "99900001" } } ],
      "isValid": false
    },
    {
      "rowNumber": 5,
      "data": { "tipoDocumento": "DNI", "docIdentidadGub": "1234", "…": "…" },
      "errors": [ { "code": "invalid-document-number", "field": "docIdentidadGub", "params": null } ],
      "isValid": false
    },
    {
      "rowNumber": 6,
      "data": { "tipoDocumento": "LIBRETA", "docIdentidadGub": "99900105", "…": "…" },
      "errors": [ { "code": "invalid-option", "field": "tipoDocumento", "params": { "value": "LIBRETA" } } ],
      "isValid": false
    },
    {
      "rowNumber": 7,
      "data": { "docIdentidadGub": "99900106", "fechaNacimiento": "1990-01-01", "…": "…" },
      "errors": [ { "code": "invalid-date", "field": "fechaNacimiento", "params": null } ],
      "isValid": false
    },
    {
      "rowNumber": 8,
      "data": { "docIdentidadGub": "99900107", "fechaNacimiento": "01/01/2999", "…": "…" },
      "errors": [ { "code": "future-date", "field": "fechaNacimiento", "params": null } ],
      "isValid": false
    }
  ],
  "totalCount": 7,
  "validCount": 2,
  "errorCount": 5
}
```

(La fila 3, válida —pasaporte escrito como `PASAPORTE`—, se omite.)

#### `POST /api/v1/Pacientes/carga-masiva`

Guarda las filas confirmadas tras la vista previa (paso 3).

| | |
|---|---|
| **Permiso** | `Pacientes:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/PacientesController.cs:125` → `CargaMasivaPacientesCommand` |
| **Pantalla del SPA** | `/pacientes/carga-masiva` (botón «Guardar válidos (N)») |

**Cuerpo** (`application/json`, máximo 5 MB): arreglo de 1 a 5 000 filas.

| Campo de cada fila | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `tipoDocumento` | texto | Sí | Máximo 20; código o abreviatura ([RN-PAC-013][PAC], [RN-PAC-016][PAC]) |
| `docIdentidadGub` | texto | Sí | Máximo 50 |
| `nombreApellidos` | texto | Sí | Máximo 200 |
| `fechaNacimiento` | texto | Sí | Máximo 10 caracteres; exactamente `dd/MM/yyyy` ([RN-PAC-014][PAC]) |
| `contactoPrimario` | texto | No | Máximo 100 |

El servidor omite sin avisar las filas con fecha mal escrita, las que la entidad rechaza (tipo desconocido, formato de documento, fecha futura) y las de un documento ya existente o repetido en el arreglo (RN-PAC-015).

**Respuesta correcta:** `200 OK`, `{ "savedCount": <filas guardadas> }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `CargaMasivaPacientesCommand.Rows` | Arreglo vacío o con más de 5 000 filas | RN-PAC-016. Ejemplo propio no verificado; se comprobó en `Medicamentos` |
| 400 | `CargaMasivaPacientesCommand.Rows[i].<Campo>` | Un campo obligatorio vacío o sobre su longitud máxima en la fila `i` | RN-PAC-016 |
| 413 | `Request.TooLarge` | Más de 5 MB | Ejemplo no verificado |

**Ejemplo real** (cuatro filas: dos válidas, una con documento repetido y una con fecha `1990-01-01`):

```http
POST /api/v1/Pacientes/carga-masiva
Authorization: Bearer <token>
Content-Type: application/json

[
  { "tipoDocumento": "DNI", "docIdentidadGub": "99900101", "nombreApellidos": "4C-Paciente Masivo A", "fechaNacimiento": "15/06/1980", "contactoPrimario": "999000101" },
  { "tipoDocumento": "PASAPORTE", "docIdentidadGub": "4CPAS0102", "nombreApellidos": "4C-Paciente Masivo B", "fechaNacimiento": "02/02/1975" },
  { "tipoDocumento": "DNI", "docIdentidadGub": "99900001", "nombreApellidos": "4C-Documento repetido", "fechaNacimiento": "01/01/1990" },
  { "tipoDocumento": "DNI", "docIdentidadGub": "99900106", "nombreApellidos": "4C-Fecha mal escrita", "fechaNacimiento": "1990-01-01" }
]
```

```json
HTTP/1.1 200 OK
{ "savedCount": 2 }
```

Error real (una fila con `"tipoDocumento": ""`):

```json
HTTP/1.1 400 Bad Request
{
  "status": 400,
  "detail": "'Tipo Documento' must not be empty.",
  "code": "CargaMasivaPacientesCommand.Rows[0].TipoDocumento"
}
```

---

## 7. `ProfesionalesSalud`

Médicos y otros prescriptores que firman las recetas. Archivo: `sofia-apis/src/SOFIA.API/Controllers/ProfesionalesSaludController.cs`. Ruta base: `/api/v1/profesionalessalud` (en minúsculas en el atributo `[Route]`; el enrutamiento no distingue mayúsculas). Reglas: [RN-PRO-001 a RN-PRO-012][PRO]. Permisos: [05b, apartado 3.17][05B]. Pantallas: `/profesionales-salud`, `/profesionales-salud/nueva`, `/profesionales-salud/:id` y `/profesionales-salud/carga-masiva`. `GET /profesionalessalud` lo usan además las pantallas de recetas médicas (05b, apartado 3.17).

**Forma del profesional** (`ProfesionalSaludDto`, `SOFIA.Application/Profesionales/Queries/GetProfesionalesSalud/GetProfesionalesSaludQuery.cs:9`):

| Campo | Tipo | Descripción |
|---|---|---|
| `id` | GUID | Identificador. |
| `numeroRegistro` | texto | Hasta 50 caracteres; único por empresa entre los no eliminados (RN-PRO-002, RN-PRO-005). |
| `nombrePrescriptor` | texto | Hasta 150 caracteres (RN-PRO-003). |
| `direccionClinica` | texto o `null` | Hasta 255 caracteres (RN-PRO-004). |

#### `GET /api/v1/profesionalessalud`

Lista los profesionales con filtros y paginación. Ordena por nombre y luego por identificador (RN-PRO-008).

| | |
|---|---|
| **Permiso** | `ProfesionalesSalud:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProfesionalesSaludController.cs:19` → `GetProfesionalesSaludQuery` |
| **Pantalla del SPA** | `/profesionales-salud` (botón «Buscar») |

**Parámetros de consulta:**

| Nombre | Tipo | Obligatorio | Descripción |
|---|---|---|---|
| `numeroRegistro` | texto | No | Contiene este texto. |
| `nombrePrescriptor` | texto | No | Contiene este texto. |
| `pageNumber` | entero | No | Página, por defecto 1. |
| `pageSize` | entero | No | Tamaño de página, por defecto 10. |

**Respuesta correcta:** `200 OK`: `{ items: [ ProfesionalSaludDto ], pageNumber, totalPages, totalCount, hasPreviousPage, hasNextPage }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `ProfesionalesSalud:Leer` | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/profesionalessalud?nombrePrescriptor=4C-&pageNumber=1&pageSize=2
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "items": [
    { "id": "210cc2fa-14cb-455a-a1c2-b6ba8b126931", "numeroRegistro": "4C-CMP-0001", "nombrePrescriptor": "4C-Dr. Prueba Uno", "direccionClinica": "Av. Ficticia 123, Lima" },
    { "id": "88b1aff2-4f64-4d1d-b520-2f5de033f9bb", "numeroRegistro": "4C-CMP-0002", "nombrePrescriptor": "4C-Dra. Prueba Dos", "direccionClinica": null }
  ],
  "pageNumber": 1,
  "totalPages": 2,
  "totalCount": 3,
  "hasPreviousPage": false,
  "hasNextPage": true
}
```

#### `GET /api/v1/profesionalessalud/{id}`

Devuelve un profesional.

| | |
|---|---|
| **Permiso** | `ProfesionalesSalud:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProfesionalesSaludController.cs:27` → `GetProfesionalSaludByIdQuery` |
| **Pantalla del SPA** | `/profesionales-salud/:id` (carga inicial) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `200 OK`, `ProfesionalSaludDto`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `ProfesionalSalud.NotFound` | No existe o está eliminado | RN-PRO-006 |
| 401 / 403 | — | Sin token o sin permiso | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/profesionalessalud/210cc2fa-14cb-455a-a1c2-b6ba8b126931
Authorization: Bearer <token>
```

```json
HTTP/1.1 200 OK
{
  "id": "210cc2fa-14cb-455a-a1c2-b6ba8b126931",
  "numeroRegistro": "4C-CMP-0001",
  "nombrePrescriptor": "4C-Dr. Prueba Uno",
  "direccionClinica": "Av. Ficticia 123, Lima"
}
```

Error real (`GET /api/v1/profesionalessalud/00000000-0000-0000-0000-000000000099`): `404`, `ProfesionalSalud.NotFound`, `ProfesionalSalud with ID 00000000-0000-0000-0000-000000000099 was not found.`.

#### `POST /api/v1/profesionalessalud`

Crea un profesional de salud.

| | |
|---|---|
| **Permiso** | `ProfesionalesSalud:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProfesionalesSaludController.cs:35` → `CreateProfesionalSaludCommand` |
| **Pantalla del SPA** | `/profesionales-salud/nueva` (botón «Guardar») |

**Cuerpo** (`application/json`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `numeroRegistro` | texto | Sí | No vacío, máximo 50; único entre los no eliminados ([RN-PRO-002][PRO], [RN-PRO-005][PRO]). |
| `nombrePrescriptor` | texto | Sí | No vacío, máximo 150 ([RN-PRO-003][PRO]). |
| `direccionClinica` | texto | No | Máximo 255 ([RN-PRO-004][PRO]). |

**Respuesta correcta:** `201 Created`, cuerpo = GUID del profesional y `Location` hacia `GET /api/v1/profesionalessalud/{id}`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `CreateProfesionalSaludCommand.NumeroRegistro` / `.NombrePrescriptor` / `.DireccionClinica` | Campo vacío o más largo que el máximo | RN-PRO-002 a RN-PRO-004 |
| 409 | `ProfesionalSalud.NumeroRegistro.Duplicado` | Otro profesional no eliminado usa el número de registro | RN-PRO-005 |

**Ejemplo real:**

```http
POST /api/v1/profesionalessalud
Authorization: Bearer <token>
Content-Type: application/json

{
  "numeroRegistro": "4C-CMP-0001",
  "nombrePrescriptor": "4C-Dr. Prueba Uno",
  "direccionClinica": "Av. Ficticia 123, Lima"
}
```

```json
HTTP/1.1 201 Created
"210cc2fa-14cb-455a-a1c2-b6ba8b126931"
```

Errores reales. Número de registro repetido:

```json
HTTP/1.1 409 Conflict
{
  "status": 409,
  "detail": "Another health professional already uses this registration number.",
  "code": "ProfesionalSalud.NumeroRegistro.Duplicado"
}
```

Número de registro vacío:

```json
HTTP/1.1 400 Bad Request
{
  "status": 400,
  "detail": "NÃºmero de Registro is required.",
  "code": "CreateProfesionalSaludCommand.NumeroRegistro"
}
```

#### `PUT /api/v1/profesionalessalud/{id}`

Actualiza los datos de un profesional.

| | |
|---|---|
| **Permiso** | `ProfesionalesSalud:Actualizar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProfesionalesSaludController.cs:45` → `UpdateProfesionalSaludCommand` |
| **Pantalla del SPA** | `/profesionales-salud/:id` (botón «Guardar» en modo edición) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Cuerpo:** `numeroRegistro`, `nombrePrescriptor` y `direccionClinica`, con las reglas de `POST`. Este comando **no tiene validador**: las validaciones las hace la entidad `ProfesionalSalud` (`SOFIA.Domain/Entities/ProfesionalSalud.cs:51-86`), por eso el `code` de las fallas es `ProfesionalSalud.<Propiedad>` y no `UpdateProfesionalSaludCommand.<Propiedad>`. La unicidad excluye al propio profesional (RN-PRO-005).

**Respuesta correcta:** `204 No Content`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `ProfesionalSalud.NotFound` | No existe o está eliminado | RN-PRO-006 |
| 409 | `ProfesionalSalud.NumeroRegistro.Duplicado` | Otro profesional usa el número | RN-PRO-005 |
| 400 | `ProfesionalSalud.NumeroRegistro` / `.NombrePrescriptor` / `.DireccionClinica` | Campo vacío o más largo que el máximo | RN-PRO-002 a RN-PRO-004 |

**Ejemplo real:**

```http
PUT /api/v1/profesionalessalud/88b1aff2-4f64-4d1d-b520-2f5de033f9bb
Authorization: Bearer <token>
Content-Type: application/json

{
  "numeroRegistro": "4C-CMP-0002",
  "nombrePrescriptor": "4C-Dra. Prueba Dos (editada)",
  "direccionClinica": "Jr. Imaginario 45, Arequipa"
}
```

```http
HTTP/1.1 204 No Content
```

Errores reales: con el número `4C-CMP-0001` de otro profesional → `409` `ProfesionalSalud.NumeroRegistro.Duplicado`; con `"numeroRegistro": ""`:

```json
HTTP/1.1 400 Bad Request
{
  "status": 400,
  "detail": "Número de Registro is required.",
  "code": "ProfesionalSalud.NumeroRegistro"
}
```

Con un identificador inexistente → `404` `ProfesionalSalud.NotFound`.

#### `DELETE /api/v1/profesionalessalud/{id}`

Elimina (borrado lógico) un profesional. Se bloquea si alguna receta no eliminada lo tiene como médico (RN-PRO-007).

| | |
|---|---|
| **Permiso** | `ProfesionalesSalud:Eliminar` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProfesionalesSaludController.cs:54` → `DeleteProfesionalSaludCommand` |
| **Pantalla del SPA** | `/profesionales-salud/:id` (botón «Eliminar»; ante un 409 muestra `profesionalesSalud.detail.delete-in-use`) |

**Parámetros de ruta:** `id` (GUID, obligatorio).

**Respuesta correcta:** `204 No Content`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 404 | `ProfesionalSalud.NotFound` | No existe o ya está eliminado | RN-PRO-006 |
| 409 | `ProfesionalSalud.InUse` | Alguna receta no eliminada lo tiene como médico (`DeleteProfesionalSaludCommandHandler.cs:20-27`) | RN-PRO-007 |

**Ejemplo real.** Profesional sin recetas (`4C-CMP-TMP`):

```http
DELETE /api/v1/profesionalessalud/4f6b2c5d-530c-4cdb-bff4-657f0085b48a
Authorization: Bearer <token>
```

```http
HTTP/1.1 204 No Content
```

Después, `GET` y un segundo `DELETE` responden `404` `ProfesionalSalud.NotFound`. Caso real de bloqueo: `4C-CMP-0001` es el médico de la receta creada para el paciente de prueba.

```json
HTTP/1.1 409 Conflict
{
  "status": 409,
  "detail": "Cannot delete a health professional referenced by prescriptions.",
  "code": "ProfesionalSalud.InUse"
}
```

#### `GET /api/v1/profesionalessalud/plantilla`

Descarga la plantilla Excel de carga masiva.

| | |
|---|---|
| **Permiso** | `ProfesionalesSalud:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProfesionalesSaludController.cs:62` → `ExcelTemplateGenerator.GenerateTemplate` (sin comando) |
| **Pantalla del SPA** | `/profesionales-salud/carga-masiva` (botón «Descargar plantilla») |

**Parámetros:** ninguno.

**Respuesta correcta:** `200 OK`, `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `Content-Disposition: attachment; filename=plantilla-profesionalessalud.xlsx`. Hoja «Plantilla» con tres títulos: `NumeroRegistro`, `NombrePrescriptor`, `DireccionClinica` (RN-PRO-009).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 401 / 403 | — | Sin token o sin `ProfesionalesSalud:Crear` | [apartado 0][04A0] |

**Ejemplo real:**

```http
GET /api/v1/profesionalessalud/plantilla
Authorization: Bearer <token>
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=plantilla-profesionalessalud.xlsx; filename*=UTF-8''plantilla-profesionalessalud.xlsx

<archivo .xlsx de 6447 bytes>
```

#### `POST /api/v1/profesionalessalud/previsualizar`

Valida un Excel fila por fila sin guardar nada (paso 2 de [20, apartado 2.3][20]).

| | |
|---|---|
| **Permiso** | `ProfesionalesSalud:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProfesionalesSaludController.cs:71` → `PreviewImportProfesionalesSaludQuery` |
| **Pantalla del SPA** | `/profesionales-salud/carga-masiva` (al elegir o arrastrar el archivo) |

**Cuerpo** (`multipart/form-data`): campo `file`, `.xlsx` de hasta 5 MB (RN-PRO-012).

**Reglas por fila** (códigos de fila en [20, apartado 2.5][20]):

| Campo | Errores de fila | Regla |
|---|---|---|
| `numeroRegistro` | `required`, `max-length` (50), `duplicate`, `duplicate-in-file` | RN-PRO-002, RN-PRO-005, RN-PRO-010 |
| `nombrePrescriptor` | `required`, `max-length` (150) | RN-PRO-003 |
| `direccionClinica` | `max-length` (255) | RN-PRO-004 |

**Respuesta correcta:** `200 OK`: `{ rows: [ { rowNumber, data, errors, isValid } ], totalCount, validCount, errorCount }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | (validación de modelo, `errors.file`) | No se adjuntó archivo | RN-PRO-012 |
| 400 | `CargaMasiva.Archivo.Invalido` | Extensión distinta de `.xlsx` o contenido inválido | [20, apartado 2.6][20]. Ejemplo no verificado en este controlador; el error se comprobó en `Medicamentos` |
| 400 | `CargaMasiva.Archivo.ExcedeDescompresion` / `CargaMasiva.Filas.Excedidas` | Archivo que se expande de más o con más de 5 000 filas | Ejemplo no verificado |
| 413 | `Request.TooLarge` | Más de 5 MB | Ejemplo no verificado |

**Ejemplo real** (cuatro filas: dos válidas, una con número repetido y una sin nombre):

```http
POST /api/v1/profesionalessalud/previsualizar
Authorization: Bearer <token>
Content-Type: multipart/form-data; boundary=----4c

------4c
Content-Disposition: form-data; name="file"; filename="profesionales.xlsx"
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet

<contenido del archivo>
------4c--
```

```json
HTTP/1.1 200 OK
{
  "rows": [
    {
      "rowNumber": 2,
      "data": { "numeroRegistro": "4C-CMP-0101", "nombrePrescriptor": "4C-Dr. Masivo A", "direccionClinica": "Calle Falsa 1, Lima" },
      "errors": [],
      "isValid": true
    },
    {
      "rowNumber": 4,
      "data": { "numeroRegistro": "4C-CMP-0001", "nombrePrescriptor": "4C-Registro repetido", "direccionClinica": null },
      "errors": [ { "code": "duplicate", "field": "numeroRegistro", "params": { "value": "4C-CMP-0001" } } ],
      "isValid": false
    },
    {
      "rowNumber": 5,
      "data": { "numeroRegistro": "4C-CMP-0104", "nombrePrescriptor": null, "direccionClinica": null },
      "errors": [ { "code": "required", "field": "nombrePrescriptor", "params": null } ],
      "isValid": false
    }
  ],
  "totalCount": 4,
  "validCount": 2,
  "errorCount": 2
}
```

(La fila 3, válida, se omite.)

#### `POST /api/v1/profesionalessalud/carga-masiva`

Guarda las filas confirmadas tras la vista previa (paso 3).

| | |
|---|---|
| **Permiso** | `ProfesionalesSalud:Crear` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProfesionalesSaludController.cs:89` → `CargaMasivaProfesionalesSaludCommand` |
| **Pantalla del SPA** | `/profesionales-salud/carga-masiva` (botón «Guardar válidos (N)») |

**Cuerpo** (`application/json`, máximo 5 MB): arreglo de 1 a 5 000 filas.

| Campo de cada fila | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `numeroRegistro` | texto | Sí | No vacío, máximo 50 ([RN-PRO-011][PRO]) |
| `nombrePrescriptor` | texto | Sí | No vacío, máximo 150 |
| `direccionClinica` | texto | No | Máximo 255 |

El servidor omite sin avisar las filas que la entidad rechaza y las que repiten un número de registro existente o ya visto en el arreglo (RN-PRO-010, RN-PRO-011).

**Respuesta correcta:** `200 OK`, `{ "savedCount": <filas guardadas> }`.

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | `CargaMasivaProfesionalesSaludCommand.Rows` | Arreglo vacío o con más de 5 000 filas | RN-PRO-011. Ejemplo propio no verificado; se comprobó en `Medicamentos` |
| 400 | `CargaMasivaProfesionalesSaludCommand.Rows[i].<Campo>` | Un campo obligatorio vacío o sobre su longitud máxima en la fila `i` | RN-PRO-011 |
| 413 | `Request.TooLarge` | Más de 5 MB | Ejemplo no verificado |

**Ejemplo real** (las dos filas válidas y la del número repetido):

```http
POST /api/v1/profesionalessalud/carga-masiva
Authorization: Bearer <token>
Content-Type: application/json

[
  { "numeroRegistro": "4C-CMP-0101", "nombrePrescriptor": "4C-Dr. Masivo A", "direccionClinica": "Calle Falsa 1, Lima" },
  { "numeroRegistro": "4C-CMP-0102", "nombrePrescriptor": "4C-Dra. Masivo B" },
  { "numeroRegistro": "4C-CMP-0001", "nombrePrescriptor": "4C-Registro repetido" }
]
```

```json
HTTP/1.1 200 OK
{ "savedCount": 2 }
```

#### `POST /api/v1/profesionalessalud/exportar`

Genera un Excel con los profesionales que cumplen los filtros.

| | |
|---|---|
| **Permiso** | `ProfesionalesSalud:Leer` |
| **Límite de peticiones** | solo el global |
| **Código** | `SOFIA.API/Controllers/ProfesionalesSaludController.cs:98` → `GetProfesionalesSaludQuery` (con `PageSize` = 10 000) |
| **Pantalla del SPA** | `/profesionales-salud` (botón «Exportar») |

**Cuerpo** (`application/json`, `ProfesionalSaludExportRequest`):

| Campo | Tipo | Obligatorio | Reglas |
|---|---|---|---|
| `headers` | arreglo de texto | Sí | Hasta 50 títulos de hasta 100 caracteres; si no, `400` con `errors.Headers: ["Export.Headers.Invalid"]`. |
| `numeroRegistro`, `nombrePrescriptor` | texto | No | Filtros «contiene». |

**Respuesta correcta:** `200 OK`, `Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet`, `Content-Disposition: attachment; filename=profesionalessalud.xlsx`. Hoja «Data»: títulos de `headers` y tres columnas por profesional: número de registro, nombre del prescriptor y dirección de la clínica (vacía si no tiene).

**Errores posibles:**

| HTTP | `code` | Cuándo | Regla |
|---|---|---|---|
| 400 | (validación de modelo, `errors.Headers`) | `headers` ausente, con más de 50 elementos o con elementos de más de 100 caracteres | — |
| 400 | `Export.TooManyRows` | Más de 10 000 filas | [20, apartado 2.8][20]. Ejemplo no verificado: exige más de 10 000 profesionales |

**Ejemplo real:**

```http
POST /api/v1/profesionalessalud/exportar
Authorization: Bearer <token>
Content-Type: application/json

{
  "headers": ["Número de registro", "Nombre del prescriptor", "Dirección de la clínica"],
  "nombrePrescriptor": "4C-"
}
```

```http
HTTP/1.1 200 OK
Content-Type: application/vnd.openxmlformats-officedocument.spreadsheetml.sheet
Content-Disposition: attachment; filename=profesionalessalud.xlsx; filename*=UTF-8''profesionalessalud.xlsx

<archivo .xlsx de 6625 bytes>
```

Contenido leído (hoja «Data»): `4C-CMP-0001 | 4C-Dr. Prueba Uno | Av. Ficticia 123, Lima` y `4C-CMP-0002 | 4C-Dra. Prueba Dos (editada) | Jr. Imaginario 45, Arequipa`. Error real (51 títulos de columna):

```json
HTTP/1.1 400 Bad Request
{
  "type": "https://tools.ietf.org/html/rfc9110#section-15.5.1",
  "title": "One or more validation errors occurred.",
  "status": 400,
  "errors": { "Headers": ["Export.Headers.Invalid"] },
  "traceId": "00-…"
}
```

---

## 8. Pendientes y «No verificado»

### 8.1 No verificado

Todo lo que sigue se documentó desde el código y no se pudo provocar en la API local por las razones indicadas. Cada punto aparece también como «Ejemplo no verificado» en su ficha.

| Qué | Por qué no se verificó |
|---|---|
| `400` `Export.TooManyRows` en las 7 exportaciones | Exige más de 10 000 filas por controlador. |
| `413` `Request.TooLarge`, `400` `CargaMasiva.Filas.Excedidas` y `CargaMasiva.Archivo.ExcedeDescompresion` en las vistas previas y las cargas | Exigen archivos de más de 5 MB o de más de 5 000 filas. Los errores `CargaMasiva.Archivo.Invalido` y `errors.file` sí se provocaron, pero solo en `Medicamentos` (el mecanismo es el mismo lector compartido, `ExcelReaderService.cs`). |
| `409` `PresentacionVenta.InUse` | Exige una venta completada con la presentación (lote con stock y caja abierta). |
| `400` `Auth.Sucursal` en `GET /Medicamentos` | La cuenta de la referencia sí tiene sucursal en la sesión; no se probó con una cuenta sin sucursal. |
| Errores `CargaMasiva{Modulo}Command.Rows` y `…Rows[i].<Campo>` en `Proveedores`, `Pacientes` (solo `Rows[0].TipoDocumento` sí se provocó), `ProfesionalesSalud` | Mismo comportamiento comprobado en `Medicamentos`; la ficha lo indica. |
| Respuestas `401` y `403` | La única cuenta disponible es el administrador de la empresa de referencia, con todos los permisos; se remite a [04a, apartado 0][04A0]. |
| `PUT` y `DELETE` de `Pacientes` con un `id` que no es GUID | Se comprobó el `GET`; los otros dos tienen la misma definición de ruta pero no se llamaron. |
| Lectura de decimales con coma en la vista previa de `Proveedores` | No se probó. |
| `stockPorSucursal` con datos en `GET /Medicamentos/{id}` | Los medicamentos de prueba no tienen lotes; solo se vio el arreglo vacío. |

### 8.2 En curso (sin commitear)

Ninguno. Al empezar y al terminar el trabajo, `git status --short` estaba vacío en `sofia-apis` (commit `34089ec`) y en `sofia-web-spa` (commit `9ad15f0`). Entre la línea base `43f5529` y `34089ec` no cambió ningún archivo de los siete controladores, de sus carpetas de `SOFIA.Application`, de `SOFIA.Domain` ni de `Common` (`git diff --stat` sobre esas rutas, vacío).

### 8.3 Hallazgos y defectos del código

| # | Hallazgo | Dónde |
|---|---|---|
| 1 | Los textos de los validadores de `Medicamentos` (crear y editar) y de `CreateProfesionalSaludCommandValidator`, y los mensajes «no encontrada» de las formulaciones clínicas, llegan con las tildes dañadas (`CÃ³digo Nacional is required.`, `NÃºmero de Registro is required.`, `FormulaciÃ³n with ID … not found.`): los archivos fuente están guardados con codificación doble. El SPA traduce por `code`, no por `detail`. | `CreateMedicamentoCommandValidator.cs`, `UpdateMedicamentoCommandValidator.cs`, `CreateProfesionalSaludCommandValidator.cs`, `*FormulacionClinica*Handler.cs` |
| 2 | `Seguros` es el controlador más distinto: el handler de crear y el de editar no comprueban duplicados (responde `409` `Db.Duplicado`, con `Content-Type: application/json` en vez de `application/problem+json` y con `traceId`); `DELETE` de un inexistente responde **400** `NotFound`; el `record` de la exportación no lleva `[ExportHeaders]` (acepta cualquier cantidad y longitud de títulos); `previsualizar` y `carga-masiva` no llevan `[RequestSizeLimit]`, la vista previa no comprueba la extensión y `carga-masiva` no tiene validador (acepta `[]` y responde `savedCount` 0). | `SegurosController.cs:60-122`, `DeleteAseguradoraCommandHandler.cs:21`, `CargaMasivaSegurosCommand*.cs` |
| 3 | La vista previa de `Seguros` informa `max: 200` para `nombreComercial` pero valida contra 150. | `PreviewImportSegurosQueryHandler.cs:41` |
| 4 | `PUT /Proveedores/{id}` sin `tasaCumplimiento` deja la tasa en 0 (el comando no tiene valor por defecto; el de creación sí, 100). Igual en la fila de la carga masiva. | `UpdateProveedorCommand.cs:7`, `ProveedorImportRow` |
| 5 | Los validadores de `CreateProveedorCommand` y `CreateAseguradoraCommand` están vacíos («TODO»); `UpdateProfesionalSaludCommand` no tiene validador. Las reglas las aplican las entidades, y por eso el `code` de esas fallas es `{Entidad}.{Propiedad}` en vez de `{Comando}.{Propiedad}`. | `CreateProveedorCommandValidator.cs`, `CreateAseguradoraCommandValidator.cs` |
| 6 | Las respuestas de crear y eliminar no son uniformes: `Proveedores` y `Seguros` crean con 200 (los demás con 201); `Proveedores`, `Seguros` y `Pacientes` eliminan con 200 y el GUID (los demás con 204). Eliminar un inexistente en esos tres usa el código genérico `NotFound`. | Tabla del apartado 0 |
| 7 | La carga masiva de `Medicamentos` no incluye precio de venta base; los medicamentos cargados así quedan con `precioVentaBase` `null`, mientras que la API de creación lo exige. | `CargaMasivaMedicamentosCommandHandler.cs:49-54` |
| 8 | La ruta `{id}` de `Pacientes` no lleva la restricción `:guid` (los demás controladores sí): un `id` inválido da `400` en vez de `404`. | `PacientesController.cs:37,45,54` |
| 9 | La vista previa de `Proveedores` no detecta razones sociales repetidas (documentado como RN-PRV-015). | `PreviewImportProveedoresQueryHandler.cs` |
| 10 | `05b-matriz-de-permisos.md`, apartado 3.18, dice «Endpoints: 10» para `Proveedores`, pero su tabla tiene 11 filas y `endpoints.csv` también 11. Además, 05b cita la línea del método del controlador (por ejemplo `ProveedoresController.cs:22`) y `endpoints.csv` la del atributo `[HttpXxx]` (`:21`); este documento usa la del CSV. | `05-seguridad/05b-matriz-de-permisos.md:510-512` |

### 8.4 Datos que quedaron en la empresa de prueba

Los listados del apartado 0.1 quedaron en «Farmacia API Referencia S.A.C.»; los registros marcados como eliminados se eliminaron para documentar el borrado y el resto no se tocó. Para el caso «en uso» de `Medicamentos` quedaron la presentación, la formulación, la jerarquía y los dos precios de proveedor del medicamento `4C-MED-001`; para `Pacientes` y `ProfesionalesSalud`, la receta de apoyo. No se modificó ningún registro de otra parte (`4A-`, `4B-`, `4D-`, `4E-`).

## 9. Tabla de cobertura

Los 61 endpoints de `_inventario/endpoints.csv` para los siete controladores tienen su ficha en este documento y los 61 llevan un ejemplo real (la comprobación se hizo con un script que lee `endpoints.csv` y los encabezados `####` de este archivo). «Ejemplo real» incluye el caso de error que corresponde; los casos que no se pudieron provocar están en el apartado 8.1.

| Controlador | Método y ruta | Ficha | Ejemplo |
|---|---|---|---|
| `Medicamentos` | `GET /api/v1/Medicamentos/condiciones-venta` | 1 | real |
| `Medicamentos` | `GET /api/v1/Medicamentos` | 1 | real |
| `Medicamentos` | `GET /api/v1/Medicamentos/{id}` | 1 | real |
| `Medicamentos` | `POST /api/v1/Medicamentos` | 1 | real |
| `Medicamentos` | `PUT /api/v1/Medicamentos/{id}` | 1 | real |
| `Medicamentos` | `DELETE /api/v1/Medicamentos/{id}` | 1 | real (204 y 409 `Medicamento.InUse`) |
| `Medicamentos` | `POST /api/v1/Medicamentos/exportar` | 1 | real |
| `Medicamentos` | `GET /api/v1/Medicamentos/plantilla` | 1 | real |
| `Medicamentos` | `POST /api/v1/Medicamentos/previsualizar` | 1 | real |
| `Medicamentos` | `POST /api/v1/Medicamentos/carga-masiva` | 1 | real |
| `PresentacionesVenta` | `GET /api/v1/PresentacionesVenta` | 2 | real |
| `PresentacionesVenta` | `GET /api/v1/PresentacionesVenta/{id}` | 2 | real |
| `PresentacionesVenta` | `POST /api/v1/PresentacionesVenta` | 2 | real |
| `PresentacionesVenta` | `PUT /api/v1/PresentacionesVenta/{id}` | 2 | real |
| `PresentacionesVenta` | `DELETE /api/v1/PresentacionesVenta/{id}` | 2 | real (204; el 409 `PresentacionVenta.InUse` no verificado) |
| `PresentacionesVenta` | `POST /api/v1/PresentacionesVenta/exportar` | 2 | real |
| `FormulacionesClinicas` | `GET /api/v1/FormulacionesClinicas` | 3 | real |
| `FormulacionesClinicas` | `GET /api/v1/FormulacionesClinicas/medicamento/{productoId}` | 3 | real |
| `FormulacionesClinicas` | `GET /api/v1/FormulacionesClinicas/{id}` | 3 | real |
| `FormulacionesClinicas` | `POST /api/v1/FormulacionesClinicas` | 3 | real |
| `FormulacionesClinicas` | `PUT /api/v1/FormulacionesClinicas/{id}` | 3 | real |
| `FormulacionesClinicas` | `DELETE /api/v1/FormulacionesClinicas/{id}` | 3 | real |
| `FormulacionesClinicas` | `POST /api/v1/FormulacionesClinicas/exportar` | 3 | real |
| `Proveedores` | `POST /api/v1/Proveedores` | 4 | real |
| `Proveedores` | `POST /api/v1/Proveedores/precios` | 4 | real |
| `Proveedores` | `GET /api/v1/Proveedores` | 4 | real |
| `Proveedores` | `GET /api/v1/Proveedores/{id}` | 4 | real |
| `Proveedores` | `GET /api/v1/Proveedores/{id}/precios` | 4 | real |
| `Proveedores` | `PUT /api/v1/Proveedores/{id}` | 4 | real |
| `Proveedores` | `DELETE /api/v1/Proveedores/{id}` | 4 | real (200 y 409 `Proveedor.InUse`) |
| `Proveedores` | `POST /api/v1/Proveedores/exportar` | 4 | real |
| `Proveedores` | `GET /api/v1/Proveedores/plantilla` | 4 | real |
| `Proveedores` | `POST /api/v1/Proveedores/previsualizar` | 4 | real |
| `Proveedores` | `POST /api/v1/Proveedores/carga-masiva` | 4 | real |
| `Seguros` | `POST /api/v1/Seguros` | 5 | real |
| `Seguros` | `GET /api/v1/Seguros` | 5 | real |
| `Seguros` | `GET /api/v1/Seguros/{id}` | 5 | real |
| `Seguros` | `PUT /api/v1/Seguros/{id}` | 5 | real |
| `Seguros` | `DELETE /api/v1/Seguros/{id}` | 5 | real |
| `Seguros` | `POST /api/v1/Seguros/exportar` | 5 | real |
| `Seguros` | `GET /api/v1/Seguros/plantilla` | 5 | real |
| `Seguros` | `POST /api/v1/Seguros/previsualizar` | 5 | real |
| `Seguros` | `POST /api/v1/Seguros/carga-masiva` | 5 | real |
| `Pacientes` | `POST /api/v1/Pacientes` | 6 | real |
| `Pacientes` | `GET /api/v1/Pacientes` | 6 | real |
| `Pacientes` | `GET /api/v1/Pacientes/{id}` | 6 | real |
| `Pacientes` | `PUT /api/v1/Pacientes/{id}` | 6 | real |
| `Pacientes` | `DELETE /api/v1/Pacientes/{id}` | 6 | real (200 y 409 `Paciente.InUse`, con una receta) |
| `Pacientes` | `POST /api/v1/Pacientes/exportar` | 6 | real |
| `Pacientes` | `GET /api/v1/Pacientes/plantilla` | 6 | real |
| `Pacientes` | `POST /api/v1/Pacientes/previsualizar` | 6 | real |
| `Pacientes` | `POST /api/v1/Pacientes/carga-masiva` | 6 | real |
| `ProfesionalesSalud` | `GET /api/v1/profesionalessalud` | 7 | real |
| `ProfesionalesSalud` | `GET /api/v1/profesionalessalud/{id}` | 7 | real |
| `ProfesionalesSalud` | `POST /api/v1/profesionalessalud` | 7 | real |
| `ProfesionalesSalud` | `PUT /api/v1/profesionalessalud/{id}` | 7 | real |
| `ProfesionalesSalud` | `DELETE /api/v1/profesionalessalud/{id}` | 7 | real (204 y 409 `ProfesionalSalud.InUse`, con una receta) |
| `ProfesionalesSalud` | `GET /api/v1/profesionalessalud/plantilla` | 7 | real |
| `ProfesionalesSalud` | `POST /api/v1/profesionalessalud/previsualizar` | 7 | real |
| `ProfesionalesSalud` | `POST /api/v1/profesionalessalud/carga-masiva` | 7 | real |
| `ProfesionalesSalud` | `POST /api/v1/profesionalessalud/exportar` | 7 | real |

## 10. Fuentes

Código de `sofia-apis` (commit `34089ec`):

- Controladores: `src/SOFIA.API/Controllers/MedicamentosController.cs`, `PresentacionesVentaController.cs`, `FormulacionesClinicasController.cs`, `ProveedoresController.cs`, `SegurosController.cs`, `PacientesController.cs`, `ProfesionalesSaludController.cs`.
- Infraestructura de la API: `src/SOFIA.API/Extensions/ResultExtensions.cs`, `ExportExtensions.cs`, `RateLimitingExtensions.cs`; `src/SOFIA.API/Infrastructure/GlobalExceptionHandler.cs`, `ExportHeadersAttribute.cs`, `ExportDateFormatAttribute.cs`.
- Aplicación: `src/SOFIA.Application/Medicamentos/**`, `PresentacionesVenta/**`, `FormulacionesClinicas/**`, `Proveedores/**`, `Seguros/**`, `Pacientes/**`, `Profesionales/**`; `Common/Behaviors/ValidationBehavior.cs`; `Common/Models/PaginatedList.cs`, `PaginationLimits.cs`; `Common/Excel/*.cs`; `Common/Extensions/CurrentUserExtensions.cs`; `JerarquiasUoM/Common/JerarquiaConversionResolver.cs`.
- Dominio: `src/SOFIA.Domain/Entities/Medicamento.cs`, `PresentacionVenta.cs`, `FormulacionClinica.cs`, `ProveedorDistribuidor.cs`, `HistorialPrecioProveedor.cs`, `AseguradoraMedica.cs`, `PacienteCliente.cs`, `ProfesionalSalud.cs`; `Enums/CondicionVenta.cs`, `Enums/TipoDocumentoIdentidad.cs`; `Common/Result.cs`.
- Infraestructura: `src/SOFIA.Infrastructure/Excel/ExcelReaderService.cs`, `ExcelTemplateGenerator.cs`.
- Base de datos: `src/SOFIA.DbUp/Scripts/0020_SoftDeleteAware_Uniques_And_Checks.sql` (índices únicos), `0010_PresentacionesVenta.sql`.

Documentos y inventarios: `_inventario/endpoints.csv`, `_inventario/rutas-spa.csv`, `01-reglas-de-negocio/01b-catalogos.md` (v1.1), `05-seguridad/05b-matriz-de-permisos.md`, `20-carga-masiva-y-exportacion.md`, [04a-convenciones-y-administracion.md](04a-convenciones-y-administracion.md) y [04b-catalogos-basicos.md](04b-catalogos-basicos.md) (formato de las fichas).

Llamadas reales a `https://localhost:7300` con la empresa de prueba «Farmacia API Referencia S.A.C.», el 2026-10-30, a razón de una petición cada 3 segundos; los archivos `.xlsx` de las cargas masivas se generaron con un escritor de ZIP mínimo y se descartan.

