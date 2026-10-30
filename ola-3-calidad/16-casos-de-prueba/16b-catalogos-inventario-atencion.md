# Casos de prueba — Parte B: catálogos, inventario y atención

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-30 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de documentos validados y comprobación puntual en el código; casos no ejecutados |

## 1. Convenciones

Este documento reúne los casos de prueba funcionales de la **parte B**: los catálogos maestros (módulo `CAT`: unidades de medida, jerarquías de unidades, laboratorios, ingredientes activos, medicamentos, presentaciones de venta, formulaciones clínicas, proveedores con sus precios de compra, aseguradoras y catálogo DIGEMID), el inventario y la atención (módulo `INV`: lotes, stock por sucursal, pacientes, profesionales de salud, recetas médicas y escáner de recetas con IA). Cada caso sale de un escenario de las [historias de usuario 12b][hu], de los flujos del [documento de casos de uso 13b][cu] y de las reglas de [01b][rn-b] y [01c1][rn-c]; las reglas transversales `GEN` e `IMP` están en [01a][rn-a].

**Estado de ejecución.** Todos los casos están en «No ejecutado»: se escribieron desde los documentos fuente (historias, casos de uso, reglas, inventarios de pantallas [19c][p19c], [19d][p19d] y [19e][p19e], [carga masiva y exportación][p20] y el [capítulo 4 del manual][m4]) y con comprobaciones puntuales en el código. El archivo `capitulo-05-productos-y-proveedores.md` del manual está a medias y sin validar: **no se usó como fuente**.

**Tipos**

| Tipo | Qué comprueba |
|---|---|
| Positivo | El flujo principal con datos válidos |
| Negativo | Una validación o regla de negocio que debe rechazar la operación |
| Permisos | Qué se ve o qué responde la API cuando falta un permiso o no hay sesión |
| Límite | Valores en el borde (longitud máxima, mínimo, un carácter de más) |
| Concurrencia | Dos acciones simultáneas sobre el mismo dato |
| Integración | Intervención de un sistema externo (en esta parte, Gemini y Presidio en el escáner de recetas) |

**Prioridades**

| Prioridad | Criterio |
|---|---|
| Alta | Stock (por lote y por sucursal), ajustes, medicamentos controlados, recetas y escáner con IA, protección de datos del paciente, bloqueos al eliminar un maestro en uso, validaciones de la carga masiva que protegen la integridad de los maestros, precios de compra, y aislamiento entre empresas o sucursales |
| Media | Resto del flujo principal de cada pantalla: altas, ediciones, búsquedas, exportaciones y cargas masivas |
| Baja | Presentación, ordenación, textos de ayuda, paginación y navegación |

**Cómo leer un caso**

- Cada caso tiene un ID `TC-CAT-nnn` o `TC-INV-nnn`, una tabla con **Tipo**, **Prioridad** y **Cubre**, y los bloques **Precondiciones**, **Datos**, **Pasos** y **Resultado esperado**.
- **Cubre** cita el escenario de la historia (`HU-XXX-nnn` con el título del escenario entre comillas), el caso de uso (`CU-XXX-nn`: «flujo básico», alternativo `An` o «excepción paso N» según las tablas de cada caso de uso) y las reglas `RN-…`, enlazadas a su documento. La numeración de los casos de uso sigue a la de las historias: `CU-CAT-07` corresponde a `HU-CAT-007`.
- Los textos entre « » son los que ve el usuario, copiados de los documentos fuente (que los tomaron de los `es.json`). «Aviso genérico» es el mensaje genérico de guardado, búsqueda o eliminación de esa pantalla, con su texto.
- Un «Esquema del escenario» con tabla de ejemplos se cubre con **un solo caso** que lleva la misma tabla de datos y resultados; el tester recorre todas las filas.
- Los casos «Solo API» llevan la petición (método, ruta, cuerpo) y la respuesta esperada (HTTP y `code`). El token se escribe `<token>`. Las rutas llevan el prefijo `/api/v1/`.
- **Hallazgo:** cuando el sistema se comporta distinto de lo que el negocio esperaría, el caso tiene como resultado esperado el comportamiento **actual** y lo marca con la «Brecha conocida» de la historia o el número de la tabla de pendientes de [12b sección 19.2][hu]. No se inventa el comportamiento deseado.
- **No verificado:** el comportamiento no pudo respaldarse en la aplicación ni en el código; el caso dice el motivo.
- **Comportamientos comunes.** Las historias 1.2 y 1.3 de 12b describen lo que comparten todas las pantallas de búsqueda, detalle y carga masiva. Se cubren una sola vez en los casos `TC-CAT-001` a `TC-CAT-033` de unidades de medida (módulo de referencia) y cada módulo posterior solo agrega sus excepciones; los escenarios comunes se listan en la tabla de cobertura contra esos casos.

## 2. Entorno y datos de prueba

### 2.1 Entorno

Para levantar el entorno local (servicios Docker con SQL Server, Presidio y el servidor de correo de pruebas Mailpit; scripts DbUp; API y SPA) sigue la [guía del desarrollador, sección 3][g08] y el [documento de despliegue y operación][g10]; este documento no los repite. El escáner de recetas necesita además la clave del servicio Gemini configurada en el entorno (consulta la guía del desarrollador); sin ella, los casos de tipo Integración quedan «No ejecutable».

Mientras no se indique otra cosa, los casos se ejecutan en el SPA con la sesión de la persona indicada en las precondiciones. Para los casos «Solo API» se usa un cliente HTTP (Postman, Insomnia o `curl`) contra la API local, con `Authorization: Bearer <token>`.

Los permisos se escriben `Módulo:Acción`. Para entrar a una pantalla se necesita siempre `Leer` del módulo; las rutas `…/nueva` y `…/carga-masiva` exigen también `Crear`, salvo `/stock-por-sucursal/nueva` (`Inventarios:Actualizar`) y `/recetas-medicas/escanear` (`Recetas:Analizar`). En la pantalla de permisos de un rol, `Inventarios` se rotula «Inventario», `Seguros` «Aseguradoras» y `Recetas` «Recetas Médicas».

### 2.2 Empresas, usuarios y datos maestros

Todos los datos son ficticios y evidentes (dominio de correo `sofia-manual.test`). Nunca se escriben contraseñas reales: donde un caso dice «una contraseña válida de 8 caracteres o más», el tester usa una de su elección y la anota fuera del documento.

| Dato | Valor | Para qué |
|---|---|---|
| Empresa A | «Botica Manual Demo S.A.C.», RUC `20538291746`, sede «Sede Principal» | Empresa principal de todos los casos |
| Empresa B | «Farmacia Prueba Dos S.A.C.», RUC `20123456789`, administrador `luis.dos` | Casos de aislamiento entre empresas (un código igual en dos empresas, un identificador de otra empresa) |
| Administradora de A | «Ana Demo Prueba», usuario `ana.manual`, rol `Admin` | Sesión por defecto de los casos |
| Sucursales de A | «Sede Principal» y «Sucursal Demo Norte» | Stock por sucursal, ajustes y casos de sucursal no permitida |
| Cajero de A | «Carlos Cajero Demo», usuario `carlos.cajero`, sucursal base «Sede Principal», rol «Cajero» con los 19 permisos recomendados del manual (Ventas, Sesiones de Caja, Devoluciones, Inventario Leer, DIGEMID Leer, Medicamentos Leer, Pacientes Leer, Aseguradoras Leer, Recetas Médicas Leer, Analizar con IA y Crear, Profesionales de Salud Leer, Empleados Leer) | Casos de permisos. Entre los catálogos del capítulo 4 solo tiene `DIGEMID:Leer` |
| Almacenero de A | Un empleado con usuario `alma.cenero` de la sucursal «Sucursal Demo Norte» y un rol «Almacenero» con `Inventarios` Leer, Crear, Actualizar y Eliminar y `Medicamentos:Leer` | Casos de stock con una sola sucursal permitida (las sucursales permitidas son la base, las asignadas a la cuenta y las de sus roles; `Admin` accede a todas) |
| Unidades de medida | Las nueve del manual ([capítulo 4, sección 5.1][m4]): `UND` «Unidad», `CAJ` «Caja», `BLI` «Blíster», `FRA` «Frasco», `TAB` «Tableta», `MG` «Miligramo», `ML` «Mililitro», `GR` «Gramo» y `KG` «Kilogramo»; y una `PRU` «Unidad de prueba» sin uso para los casos de eliminar | Todos los catálogos que dependen de unidades |
| Laboratorios | «Laboratorio Demo Andino» (código `DEMO-AND`) y «Laboratorio Demo Pacífico» (sin código) | Medicamentos |
| Ingredientes activos | «Paracetamol» `N02BE01`, «Amoxicilina» `J01CA04`, «Clonazepam» `N03AE01` y «Tramadol» `N02AX02` (códigos ATC del manual, sección 7.1); más «Ibuprofeno» `M01AE01` y «Omeprazol» `A02BC01` que se agregan por carga masiva | Medicamentos y formulaciones |
| Medicamentos | `DEMO-MED-0001` a `DEMO-MED-0004` (uno por ingrediente del manual); el que el tester marque con una condición de venta controlada se usa en los casos de controlados | Presentaciones, lotes, stock, recetas |
| Jerarquía de ejemplo | 1 Caja = 10 Blísteres y 1 Blíster = 10 Unidades de `DEMO-MED-0001` | Presentaciones, ventas por unidad mayor |
| Lotes | Lotes con vencimiento futuro (se registran desde la pantalla); uno vencido (la pantalla no deja registrar una caducidad pasada: se prepara por base de datos o esperando a que venza uno que caduca hoy) y uno retenido en cuarentena (módulo sin pantalla: se prepara por API o base de datos) | Stock |
| Registros que se preparan por base de datos | Un medicamento cuyo laboratorio o unidad está eliminado, un paciente sin tipo de documento, aseguradoras con reclamos, lotes referenciados por módulos sin pantalla | Casos marcados «se prepara por base de datos»; si no se puede preparar, el caso queda «No ejecutable» |
| Roles de prueba | Roles con solo `Leer` de un módulo, con `Leer` y `Actualizar`, sin `Eliminar`, y sin `Analizar` (Recetas Médicas), creados desde la pantalla de roles del [capítulo 3 del manual][m3] | Casos de permisos |
| Proveedores | «Distribuidora Demo Uno S.A.C.» y «Distribuidora Demo Dos S.A.C.» con RUC ficticio de 11 dígitos | Precios de compra |
| Pacientes y profesionales | «Paciente Demo Uno» con DNI ficticio de 8 dígitos y «Dra. Médica Demo» con colegiatura ficticia | Recetas |
| Excel de carga masiva | Plantillas descargadas desde la propia pantalla de cada carga masiva y archivos de prueba que arma el tester con las filas indicadas en cada caso | Casos de carga masiva |

Antes de cada ronda conviene tener a mano: una unidad, un laboratorio y un ingrediente **sin uso** para los casos de eliminar y otro **con uso** (asociado a un medicamento) para los casos de bloqueo; y dos navegadores o perfiles distintos para los casos de concurrencia.

## 3. Suite de humo

Treinta casos de prioridad Alta que se corren primero, en este orden. Si uno falla, no tiene sentido seguir con el módulo que prueba.

| # | Caso | Qué comprueba |
|---|---|---|
| 1 | `TC-CAT-022` | Una unidad en uso no se puede eliminar |
| 2 | `TC-CAT-026` | La carga masiva rechaza archivos con formato, tamaño o filas fuera de límite |
| 3 | `TC-CAT-027` | La vista previa de la carga masiva marca vacíos, existentes, repetidos y largos |
| 4 | `TC-CAT-040` | Una conversión de unidades que contradice a las existentes se rechaza |
| 5 | `TC-CAT-074` | Un laboratorio con medicamentos no se puede eliminar |
| 6 | `TC-CAT-095` | Se registran medicamentos controlados (Receta Retenida y Estupefaciente) |
| 7 | `TC-CAT-111` | Un medicamento con lotes, presentaciones o jerarquías no se puede eliminar |
| 8 | `TC-CAT-120` | Solo cuenta como stock vendible el lote vigente, con cantidad y sin retención |
| 9 | `TC-CAT-123` | Se registra una presentación de venta con su equivalencia calculada |
| 10 | `TC-CAT-161` | Un proveedor con precios de compra no se puede eliminar |
| 11 | `TC-CAT-166` | Se registra un precio de compra y se guarda el costo neto |
| 12 | `TC-INV-004` | No se registra un lote con caducidad vencida |
| 13 | `TC-INV-021` | Un lote con stock (incluso 0) o ventas no se puede eliminar |
| 14 | `TC-INV-025` | Se ingresa el stock de un lote en una sucursal |
| 15 | `TC-INV-026` | Un segundo ingreso suma y deja una sola fila por lote y sucursal |
| 16 | `TC-INV-028` | Solo se ofrecen las sucursales del usuario al ingresar stock |
| 17 | `TC-INV-029` | La cantidad de stock no admite negativos |
| 18 | `TC-INV-030` | Ingresar stock en una sucursal no permitida responde 403 |
| 19 | `TC-INV-037` | El ajuste fija la cantidad y registra el evento `STOCK_AJUSTAR` |
| 20 | `TC-INV-041` | Un ajuste concurrente no pisa el cambio de otra operación |
| 21 | `TC-INV-044` | La consulta de stock solo devuelve las sucursales del usuario |
| 22 | `TC-INV-047` | Un lote vencido aparece en el stock pero no es vendible |
| 23 | `TC-INV-065` | Un paciente con recetas o ventas no se puede eliminar |
| 24 | `TC-INV-077` | Se registra una receta médica |
| 25 | `TC-INV-082` | El número de receta (necesario para controlados) respeta sus límites |
| 26 | `TC-INV-095` | Una receta usada en una venta (también anulada) no se puede eliminar |
| 27 | `TC-INV-101` | Sin el permiso de análisis no hay escáner de recetas |
| 28 | `TC-INV-110` | El escáner muestra el stock de la sucursal del usuario |
| 29 | `TC-INV-111` | El escáner propone equivalentes terapéuticos |
| 30 | `TC-INV-116` | Si Presidio falla, el análisis se aborta y el texto del paciente no llega a la IA |

## 4. Unidades de medida

Este módulo es el de referencia de los comportamientos comunes de búsqueda, detalle y carga masiva ([12b secciones 1.2 y 1.3][hu]): sus casos se escriben completos y los de los demás catálogos solo agregan sus excepciones.

### HU-CAT-001 — Registrar y editar una unidad de medida

#### TC-CAT-001 — Registrar una unidad de medida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-001 «Registrar una unidad»; CU-CAT-01 flujo básico; [RN-UDM-001][rn-b], [RN-UDM-002 a 006][rn-b] |

**Precondiciones:** sesión de Ana (`Admin`); no existe la unidad `UND`.
**Datos:** Código `UND`, Descripción `Unidad`.
**Pasos:**
1. Entra a «Unidades de Medida» y haz clic en «Nuevo».
2. Escribe el código y la descripción y haz clic en «Guardar».

**Resultado esperado:** aviso «Unidad de medida guardada correctamente»; vuelves a la búsqueda con los filtros vacíos y sin tabla (o con la tabla y los filtros que tenías si ya habías buscado). Al buscar sin filtros, `UND` «Unidad» aparece en la tabla.

#### TC-CAT-002 — Sin el permiso Crear no hay botón ni ruta «nueva»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | HU-CAT-001 «Sin el permiso no hay botón»; 12b §1.2 «Sin permiso de ruta»; CU-CAT-01 flujo básico paso 1; [RN-UDM-001][rn-b] |

**Precondiciones:** un rol de prueba con solo `UnidadesMedida:Leer`; sesión de un usuario con ese rol.
**Pasos:**
1. Abre «Unidades de Medida».
2. Escribe a mano la dirección `/unidades-medida/nueva` y luego `/unidades-medida/carga-masiva`.

**Resultado esperado:** no ves los botones «Nuevo» ni «Carga Masiva». Al escribir cada dirección ves «No tienes permisos para acceder a esta pantalla.» y vuelves al Inicio. **No verificado en la aplicación:** la redirección al Inicio (el administrador tiene todos los permisos y el navegador integrado impidió probarla con el cajero; 12b sección 1.2).

#### TC-CAT-003 — Campos obligatorios vacíos al guardar una unidad

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-001 «Campos obligatorios vacíos»; CU-CAT-01 excepción paso 2; [RN-UDM-002][rn-b], [RN-UDM-004][rn-b] |

**Precondiciones:** formulario «Nuevo» abierto y vacío.
**Pasos:** 1. Haz clic en «Guardar».
**Resultado esperado:** debajo de «Código» y de «Descripción» ves «Este campo es obligatorio»; no se envía nada al servidor (la red no muestra ninguna petición `POST`).

#### TC-CAT-004 — Longitudes máximas de código y descripción

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-001 «Longitudes máximas»; CU-CAT-01 excepción paso 4; [RN-UDM-003][rn-b], [RN-UDM-005][rn-b] |

**Precondiciones:** formulario «Nuevo» abierto.
**Datos:** en «Código», 16 letras `ABCDEFGHIJKLMNOP`; en «Descripción», 51 letras seguidas.
**Pasos:**
1. Escribe las 16 letras en «Código».
2. Escribe las 51 letras en «Descripción».
3. Borra y escribe exactamente 10 letras en «Código» y 50 en «Descripción»; guarda.

**Resultado esperado:** «Código» se queda en 10 caracteres y no muestra ningún mensaje; «Descripción» no deja pasar de 50. Con 10 y 50 caracteres la unidad se guarda con «Unidad de medida guardada correctamente».

#### TC-CAT-005 — Código de unidad repetido en la misma empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-001 «Código repetido en mi empresa»; CU-CAT-01 excepción paso 5; [RN-UDM-006][rn-b] |

**Precondiciones:** existe la unidad `UND`.
**Datos:** intento 1: Código `UND`; intento 2: Código `und`.
**Pasos:** 1. Haz clic en «Nuevo», escribe el código y una descripción y guarda. 2. Repite con el segundo valor.

**Resultado esperado:** en ambos intentos ves el aviso rojo «Ya existe una unidad de medida con ese código.» y te quedas en el formulario. **No verificado:** el rechazo de `und` depende de la intercalación de la base (el manual lo comprobó, pero los scripts no la definen).

#### TC-CAT-006 — El código de una unidad eliminada queda libre

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-001 «El código de una unidad eliminada queda libre»; CU-CAT-01 flujo básico; [RN-UDM-007][rn-b] |

**Precondiciones:** existe `PRU`; la eliminas siguiendo `TC-CAT-021`.
**Datos:** Código `PRU`, Descripción `Unidad de prueba`.
**Pasos:** 1. Registra una unidad nueva con el código `PRU`.

**Resultado esperado:** se guarda con «Unidad de medida guardada correctamente».

#### TC-CAT-007 — Editar una unidad de medida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-001 «Editar una unidad»; CU-CAT-01 A1; [RN-UDM-001][rn-b], [RN-UDM-005][rn-b] |

**Precondiciones:** existe `UND`; tienes `UnidadesMedida:Actualizar`.
**Pasos:** 1. Abre `UND` desde la tabla. 2. Haz clic en «Editar», cambia la descripción a `Unidad suelta` y haz clic en «Guardar».

**Resultado esperado:** aviso «Unidad de medida guardada correctamente»; te quedas en el detalle, en modo ver y con los datos recargados (la descripción nueva).

#### TC-CAT-008 — Cambiar el código a uno que ya usa otra unidad

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-001 «Cambiar el código a uno que ya usa otra unidad»; CU-CAT-01 excepción paso 5; [RN-UDM-006][rn-b] |

**Precondiciones:** existen `UND` y `PRU`.
**Pasos:** 1. Abre `PRU`, haz clic en «Editar», cambia el código a `UND` y guarda.

**Resultado esperado:** ves «Ya existe una unidad de medida con ese código.»; la unidad `PRU` no cambió.

#### TC-CAT-009 — Guardar o abrir una unidad que otra persona eliminó

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-CAT-001 «La unidad ya no existe»; CU-CAT-01 A1 (excepción `UnidadMedida.NotFound`); [RN-UDM-009][rn-b] |

**Precondiciones:** dos sesiones de `Admin`; existe `PRU`.
**Pasos:**
1. En la sesión 1 abre `PRU` y haz clic en «Editar».
2. En la sesión 2 elimina `PRU`.
3. En la sesión 1 cambia la descripción y guarda.
4. En la sesión 2 vuelve a la búsqueda de la sesión 1 y abre el detalle de `PRU` desde un enlace guardado (`/unidades-medida/<id>`).

**Resultado esperado:** al guardar ves «Error al guardar la unidad de medida» (la API responde 404 `UnidadMedida.NotFound`); al abrir el detalle ves «Error al cargar la unidad de medida».

#### TC-CAT-010 — Solo API: el identificador de la URL manda al editar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-CAT-001 «Identificador de la unidad al editar (solo API)»; CU-CAT-01 A1 (excepción `UpdateUnidadMedidaCommand.Id`); [RN-UDM-008][rn-b] |

**Precondiciones:** token de `Admin`; existen las unidades `UND` (id `<id-UND>`) y `PRU` (id `<id-PRU>`).
**Datos:** `PUT /api/v1/UnidadesMedida/<id-PRU>` con cuerpo `{ "id": "<id-UND>", "codigo": "PRU", "descripcion": "Unidad de prueba 2" }`; luego `PUT /api/v1/UnidadesMedida/00000000-0000-0000-0000-000000000000` con un cuerpo válido.
**Pasos:** 1. Envía la primera petición y consulta ambas unidades. 2. Envía la segunda.

**Resultado esperado:** la primera modifica `PRU` (la API toma el identificador de la URL y descarta el del cuerpo) y `UND` no cambia; la segunda responde 400 con el código `UpdateUnidadMedidaCommand.Id`. **No verificado:** el nombre exacto de las propiedades del cuerpo (se deduce de la pantalla).

#### TC-CAT-011 — Valores de solo espacios en un campo obligatorio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-001 12b §1.2 «Valores de solo espacios»; CU-CAT-01 excepción paso 4; [RN-UDM-002][rn-b], [RN-UDM-004][rn-b] |

**Precondiciones:** formulario «Nuevo» abierto.
**Datos:** Código `   ` (3 espacios), Descripción `   `.
**Pasos:** 1. Escribe los espacios en ambos campos. 2. Haz clic en «Guardar».

**Resultado esperado:** **Hallazgo (brecha 1 de HU-CAT-001, pendiente 1 de 12b 19.2):** el formulario lo acepta porque los campos no están vacíos; el servidor lo rechaza y ves el aviso genérico «Error al guardar la unidad de medida».

### HU-CAT-002 — Buscar, consultar y exportar unidades de medida

#### TC-CAT-012 — Menú, pantalla y filtros de unidades de medida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-002 «Ver el menú y la pantalla»; 12b §1.2 «Los resultados aparecen solo después de buscar»; CU-CAT-02 flujo básico paso 1; [RN-UDM-001][rn-b] |

**Precondiciones:** rol con `UnidadesMedida:Leer`.
**Pasos:** 1. Mira el menú. 2. Abre «Unidades de Medida».

**Resultado esperado:** «Unidades de Medida» aparece en el menú, grupo CATÁLOGOS; ves los filtros «Código» y «Descripción», ambos con lupa, la tarjeta de filtros con «Ingresa los criterios de búsqueda» y ninguna tabla. Con Enter dentro de un filtro de texto también se busca.

#### TC-CAT-013 — Buscar sin filtros y ordenación por descripción

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-002 «Buscar sin filtros»; CU-CAT-02 flujo básico; [RN-UDM-012][rn-b] |

**Precondiciones:** existen exactamente las nueve unidades del manual (Blíster, Caja, Frasco, Gramo, Kilogramo, Miligramo, Mililitro, Tableta y Unidad).
**Pasos:** 1. Haz clic en «Buscar» sin escribir nada.

**Resultado esperado:** «Resultados (9)» con las unidades ordenadas por descripción: Blíster, Caja, Frasco, Gramo, Kilogramo, Miligramo, Mililitro, Tableta y Unidad. Columnas «Código» (enlace al detalle) y «Descripción».

#### TC-CAT-014 — Filtrar unidades por código, por descripción y por ambos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-002 «Filtrar por código o por descripción»; CU-CAT-02 flujo básico paso 3; [RN-UDM-012][rn-b] |

**Precondiciones:** existen `MG`, `ML` y `PRU`.
**Datos:** (a) «Código» `m`; (b) «Descripción» `UNIDAD`; (c) «Código» `u` y «Descripción» `prueba`.
**Pasos:** 1. Busca con cada combinación.

**Resultado esperado:** (a) `MG` y `ML`; (b) las unidades cuya descripción contiene «unidad», sin distinguir mayúsculas; (c) solo la unidad que cumple las dos condiciones (`PRU`).

#### TC-CAT-015 — Búsqueda de unidades sin coincidencias

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-002 «Sin coincidencias»; 12b §1.2 «Sin coincidencias»; CU-CAT-02 excepción paso 3 (sin coincidencias) |

**Pasos:** 1. Busca el código `zzz`.

**Resultado esperado:** «Resultados (0)», la fila vacía «Resultados» y el paginador «Sin resultados».

#### TC-CAT-016 — Exportar unidades a Excel con los filtros escritos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-002 «Exportar a Excel»; CU-CAT-02 A1; [RN-UDM-013][rn-b], [RN-IMP-008][rn-a] |

**Precondiciones:** busca con «Código» `m`.
**Pasos:** 1. Haz clic en «Exportar». 2. Abre el archivo.

**Resultado esperado:** se descarga `unidades-medida.xlsx` con una hoja «Data» y las columnas «Código» y «Descripción», solo con las filas que cumplen el filtro escrito en ese momento. **No verificado en la aplicación:** el contenido del archivo (el navegador integrado no muestra descargas; solo se comprobó el 200).

#### TC-CAT-017 — Exportación con más de 10000 filas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | HU-CAT-002 «Exportación demasiado grande»; CU-CAT-02 A1 (excepción `Export.TooManyRows`); [RN-IMP-009][rn-a] |

**Precondiciones:** más de 10000 unidades en la empresa (se logra solo con carga masiva repetida; si no es práctico, queda «No ejecutable»).
**Pasos:** 1. Haz clic en «Exportar» sin filtros.

**Resultado esperado:** la API responde 400 `Export.TooManyRows`, no se genera el archivo y ves «Error al buscar unidades de medida».

#### TC-CAT-018 — Falla la búsqueda de unidades

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-CAT-002 «Falla la búsqueda»; CU-CAT-02 excepción paso 3 (falla la búsqueda) |

**Precondiciones:** entorno propio del tester, con la API detenida (nunca en un entorno compartido).
**Pasos:** 1. Haz clic en «Buscar».

**Resultado esperado:** ves «Error al buscar unidades de medida».

#### TC-CAT-019 — Limpiar, volver al detalle y paginar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | 12b §1.2 «Limpiar y volver conservan o descartan los filtros»; 12b §1.2 «Paginación»; CU-CAT-02 A2 |

**Precondiciones:** más de 10 unidades (por ejemplo, las nueve del manual más `PRU` y dos más).
**Pasos:**
1. Busca con «Descripción» `a` y abre un detalle; vuelve con la flecha de la esquina superior izquierda.
2. Haz clic en «Limpiar».
3. Busca sin filtros; cambia a 25 filas por página, ve a la última página y luego haz clic en «Buscar» de nuevo.

**Resultado esperado:** al volver, los filtros están recordados y la búsqueda se repite sola; «Limpiar» vacía los filtros, oculta la tabla y olvida el estado guardado. El paginador ofrece 10, 25 o 50 filas (10 por defecto) con botones de primera, anterior, siguiente y última; «Buscar» no vuelve a la página 1: pide la página en la que estás.

#### TC-CAT-020 — Modo ver, editar y crear: botones por permiso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | 12b §1.2 «Modo ver, editar y crear en el detalle»; HU-CAT-003 «Sin permiso no hay botón»; CU-CAT-03 flujo básico paso 1; [RN-UDM-001][rn-b] |

**Precondiciones:** tres roles de prueba sobre `UnidadesMedida`: (R1) solo `Leer`; (R2) `Leer` y `Actualizar`; (R3) `Leer`, `Actualizar` y `Eliminar`.
**Pasos:** 1. Con cada rol abre `UND` y fíjate en los botones. 2. Con R2 y R3 haz clic en «Editar».

**Resultado esperado:** R1 ve los campos deshabilitados y ningún botón; R2 ve «Editar» y, tras hacer clic, «Cancelar» y «Guardar» sin «Eliminar»; R3 ve «Cancelar», «Guardar» y «Eliminar». «Cancelar» en un registro nuevo vuelve a la búsqueda; en uno existente descarta lo escrito y vuelve al modo ver.

### HU-CAT-003 — Eliminar una unidad de medida

#### TC-CAT-021 — Eliminar una unidad sin uso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-003 «Eliminar una unidad sin uso»; «Cancelar no borra nada»; 12b §1.2 «Eliminar pide confirmación»; CU-CAT-03 flujo básico; [RN-UDM-009][rn-b], [RN-UDM-011][rn-b] |

**Precondiciones:** existe `PRU` sin uso; tienes `UnidadesMedida:Eliminar`.
**Pasos:** 1. Abre `PRU`, haz clic en «Editar» y luego en «Eliminar». 2. Mira el cuadro. 3. Haz clic en «Cancelar». 4. Vuelve a «Eliminar» y haz clic en «Confirmar».

**Resultado esperado:** se abre «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar esta unidad de medida?» y los botones «Cancelar» y «Confirmar». «Cancelar» cierra el cuadro y la unidad sigue existiendo (CU-CAT-03 A1). Al confirmar ves «Unidad de medida eliminada correctamente», vuelves a la búsqueda con tus filtros y `PRU` desaparece de todas las listas; su código queda libre (`TC-CAT-006`).

#### TC-CAT-022 — Bloqueo al eliminar una unidad en uso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-003 «La unidad está en uso»; CU-CAT-03 excepción paso 4 (`UnidadMedida.InUse`); [RN-UDM-010][rn-b] |

**Precondiciones:** tres unidades: `UND` como unidad base de `DEMO-MED-0001`; `CAJ` como unidad mayor de una jerarquía; `MG` como unidad de dosis de una formulación clínica.
**Pasos:** 1. Abre cada una, entra en modo editar y elimina. 2. Confirma.

**Resultado esperado:** en los tres casos la API responde 409 `UnidadMedida.InUse` y ves «No se puede eliminar la unidad de medida porque está asociada a medicamentos o jerarquías de unidades.»; la unidad sigue existiendo. **Hallazgo (brecha 1 y 2):** el texto no menciona las formulaciones clínicas y la pantalla lo muestra ante cualquier 409. **No verificado en la aplicación:** el bloqueo por uso (el manual lo dejó pendiente).

#### TC-CAT-023 — Hallazgo: una unidad usada por una presentación de venta se puede eliminar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-003 «La unidad está en uso» (brecha 3); CU-CAT-03 excepción paso 4; [RN-UDM-010][rn-b] |

**Precondiciones:** una unidad `BLI` usada **solo** como unidad de venta de una presentación (no es unidad base de un medicamento, ni parte de jerarquía, ni dosis de una formulación).
**Pasos:** 1. Elimina `BLI`.

**Resultado esperado:** **Hallazgo (pendiente 7 de 12b 19.2):** la eliminación se completa con «Unidad de medida eliminada correctamente» porque el handler no consulta las presentaciones de venta y el borrado lógico no dispara la clave foránea. No es el comportamiento deseado por el negocio; es el actual.

#### TC-CAT-024 — Eliminar una unidad que otra persona ya eliminó, y errores distintos de 409

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-CAT-003 «La unidad ya no existe»; 12b §1.2 «La pantalla decide el aviso de eliminación solo por el estado HTTP»; CU-CAT-03 excepción paso 4 (`UnidadMedida.NotFound`) |

**Precondiciones:** dos sesiones de `Admin`; existe `PRU` sin uso.
**Pasos:** 1. En la sesión 1 abre `PRU`, «Editar», «Eliminar». 2. En la sesión 2 elimina `PRU`. 3. En la sesión 1 haz clic en «Confirmar».

**Resultado esperado:** la API responde 404 `UnidadMedida.NotFound` y ves «Error al eliminar la unidad de medida» (el aviso genérico: cualquier estado distinto de 409 lo produce); sigues en el detalle.

### HU-CAT-004 — Cargar unidades de medida desde un Excel

#### TC-CAT-025 — Descargar la plantilla y abrir la carga masiva

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | 12b §1.3 «CM-1 Descargar la plantilla»; HU-CAT-004 «Columnas de la plantilla»; CU-CAT-04 flujo básico paso 1; [RN-UDM-014][rn-b], [RN-IMP-001][rn-a] |

**Precondiciones:** `UnidadesMedida:Crear` y `Leer`.
**Pasos:** 1. Abre `/unidades-medida` y haz clic en «Carga Masiva». 2. Mira la pantalla. 3. Haz clic en «Descargar plantilla». 4. Abre el archivo.

**Resultado esperado:** título «Carga Masiva», zona «Arrastra tu archivo Excel aquí o haz clic para seleccionar» y debajo «Solo archivos .xlsx (máx. 5 MB y 5000 filas)». Se guarda `plantilla-unidades-medida.xlsx` con una sola hoja «Plantilla» y solo la fila de títulos (negrita, texto blanco sobre azul): «Codigo» (obligatoria, máximo 10 caracteres) y «Descripcion» (obligatoria, máximo 50), sin tilde. Si la descarga falla ves «Error al descargar la plantilla. Intenta de nuevo.» **No verificado en la aplicación:** el contenido de la plantilla.

#### TC-CAT-026 — Rechazo del archivo por formato, tamaño o número de filas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | 12b §1.3 «CM-2 Rechazo del archivo»; CU-CAT-04 excepción pasos 1 a 4 (excepciones comunes); [RN-IMP-002][rn-a], [RN-IMP-003][rn-a], [RN-IMP-004][rn-a], [RN-IMP-005][rn-a] |

**Precondiciones:** pantalla «Carga Masiva» de unidades abierta.
**Datos:** (a) un archivo `datos.csv`; (b) un archivo de texto renombrado a `falso.xlsx`; (c) un `.xlsx` válido de más de 5 MB; (d) un `.xlsx` válido con 5001 filas con datos; (e) un `.xlsx` que se descomprime por encima de los límites.
**Pasos:** 1. Sube cada archivo.

**Resultado esperado:** (a) y (b): «El archivo no es un Excel .xlsx válido. Descarga la plantilla y vuelve a intentarlo.»; (c) «El archivo supera el tamaño máximo de 5 MB.»; (d) «El archivo supera el máximo de 5000 filas. Divídelo en varios archivos.»; (e) «El archivo excede los límites de contenido permitidos. Revisa que sea una plantilla válida.» Si falla el procesamiento por otra causa: «Error al procesar el archivo». **No verificado en la aplicación:** los errores de tamaño y de más de 5000 filas, y el caso (e) (manual cap. 4, sección 12).

#### TC-CAT-027 — Vista previa: totales y reglas por fila

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | 12b §1.3 «CM-3 Vista previa sin guardar nada»; 12b §1.3 «CM-4 Una fila repetida en el archivo marca todas sus apariciones»; HU-CAT-004 «Reglas por fila en la vista previa»; CU-CAT-04 excepción paso 2; [RN-UDM-002 a 006][rn-b], [RN-UDM-015][rn-b] |

**Precondiciones:** existe `UND`.
**Datos:** archivo de 7 filas (Codigo / Descripcion): (1) vacío / `Sin código`; (2) `UND` / `Repetida`; (3) `LT` / `Litro`; (4) `LT` / `Litro 2`; (5) `ABCDEFGHIJKLMN` (14 letras) / 60 letras; (6) `GR` / `Gramo`; (7) `KG` / `Kilogramo`.
**Pasos:** 1. Sube el archivo. 2. Revisa los contadores y la columna sin título de mensajes. 3. Antes de guardar, ve a la búsqueda de unidades y comprueba que no se creó nada.

**Resultado esperado:** «7 filas totales», «2 filas válidas» y «5 filas con error». Fila 1: «El campo 'codigo' es obligatorio»; fila 2: «El valor 'UND' del campo 'codigo' ya existe en el sistema»; filas 3 y 4 (las dos, también la primera): «El valor 'LT' del campo 'codigo' está duplicado en el archivo»; fila 5: dos mensajes, «'codigo' no puede superar los 10 caracteres» y «'descripcion' no puede superar los 50 caracteres»; filas 6 y 7 con la marca verde en la columna «Estado». El nombre del campo se ve como lo usa el sistema por dentro (`codigo`), no con la etiqueta de la pantalla. Todavía no se guardó nada.

#### TC-CAT-028 — Lectura del Excel: mayúsculas, columnas extra, valores recortados y filas vacías

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | 12b §1.3 «CM-3 Vista previa sin guardar nada»; CU-CAT-04 flujo básico paso 2; [RN-IMP-013][rn-a] |

**Datos:** archivo con las columnas `CODIGO`, `descripcion` y una columna extra `Notas`; una fila con `  GR  ` / `  Gramo  `; una fila totalmente vacía entre dos filas con datos; otro archivo sin la columna `Descripcion`.
**Pasos:** 1. Sube el primer archivo. 2. Sube el segundo.

**Resultado esperado:** el primer archivo se lee sin distinguir mayúsculas, ignora la columna extra, recorta los valores y no cuenta la fila vacía. En el segundo, la columna ausente se lee vacía y cada fila muestra «El campo 'descripcion' es obligatorio». **No verificado:** el número exacto que cuenta como «filas totales» con filas vacías (se deduce de la regla «las filas totalmente vacías se ignoran»).

#### TC-CAT-029 — Guardar solo las filas válidas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | 12b §1.3 «CM-5 Guardar solo las filas válidas»; HU-CAT-004 «Guardar las filas válidas»; CU-CAT-04 flujo básico pasos 3 y 4; [RN-UDM-016][rn-b], [RN-IMP-006][rn-a] |

**Datos:** archivo con las filas válidas `GR` / `Gramo` y `KG` / `Kilogramo` y una fila inválida.
**Pasos:** 1. Sube el archivo. 2. Haz clic en «Guardar válidos (2)». 3. Busca las unidades.

**Resultado esperado:** solo se envían las filas válidas; ves «Se guardaron 2 registros correctamente», vuelves a la búsqueda de unidades y `GR` y `KG` existen; la fila inválida no. Con un archivo sin ninguna fila válida el botón «Guardar válidos» está deshabilitado; si se intentara guardar, el aviso sería «No hay filas válidas para guardar».

#### TC-CAT-030 — Descripciones repetidas son válidas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-004 «La descripción puede repetirse»; CU-CAT-04 flujo básico paso 2; [RN-UDM-006][rn-b] |

**Datos:** dos filas con códigos `A1` y `A2` y la misma descripción `Pieza`.
**Pasos:** 1. Sube el archivo.

**Resultado esperado:** ambas filas son válidas (la marca verde en «Estado»).

#### TC-CAT-031 — Hallazgo: al guardar se omiten sin error las repetidas que llegan al servidor

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-004 «Al guardar se omiten las repetidas que llegaran hasta el servidor»; CU-CAT-04 flujo básico paso 3; [RN-UDM-015][rn-b], [RN-UDM-016][rn-b] |

**Precondiciones:** token de `Admin`; existe `UND`.
**Datos:** `POST /api/v1/UnidadesMedida/carga-masiva` con tres filas: `NUE` / `Nueva`, `NUE` / `Nueva 2`, `UND` / `Repetida`.
**Pasos:** 1. Envía la petición. 2. Consulta `GET /api/v1/UnidadesMedida`.

**Resultado esperado:** **Hallazgo (pendiente 3 de 12b 19.2):** el servidor guarda la primera fila `NUE` y omite las otras dos sin error; la respuesta informa solo cuántas guardó (1). En la pantalla el aviso «Se guardaron N registros correctamente» puede decir menos que las «filas válidas» de la vista previa e incluso «Se guardaron 0 registros correctamente». Compara siempre el número guardado con el de filas válidas. **No verificado:** el nombre de la propiedad de la respuesta.

#### TC-CAT-032 — Solo API: lista de filas fuera de rango y vista previa sin archivo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-004 «Lista de filas fuera de rango (solo API)»; HU-CAT-004 «Vista previa sin archivo (solo API)»; CU-CAT-04 excepción paso 3 (lista vacía o de más de 5000); [RN-UDM-016][rn-b], [RN-UDM-017][rn-b] |

**Precondiciones:** token con `UnidadesMedida:Crear`.
**Datos:** (a) `POST /api/v1/UnidadesMedida/carga-masiva` con lista vacía; (b) lo mismo con 5001 filas; (c) `POST /api/v1/UnidadesMedida/previsualizar` sin archivo; (d) lo mismo con un archivo vacío.
**Pasos:** 1. Envía cada petición.

**Resultado esperado:** (a) y (b): 400 `CargaMasivaUnidadesMedidaCommand.Rows` (la pantalla lo muestra como «El archivo supera el máximo de 5000 filas. Divídelo en varios archivos.»); (c) y (d): 400 con el texto «Debe adjuntar un archivo Excel.» La pantalla nunca provoca (c) ni (d): siempre envía un archivo.

#### TC-CAT-033 — Si el servidor rechaza filas al guardar, no se guarda ninguna

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | 12b §1.3 «CM-5 Guardar solo las filas válidas»; CU-CAT-04 excepción paso 3 (fila rechazada por el validador); [RN-UDM-016][rn-b] |

**Precondiciones:** token con `UnidadesMedida:Crear`.
**Datos:** `POST /api/v1/UnidadesMedida/carga-masiva` con dos filas: una válida y otra con `descripcion` de 60 caracteres.
**Pasos:** 1. Envía la petición. 2. Busca las unidades.

**Resultado esperado:** 400 con el código `CargaMasivaUnidadesMedidaCommand.Rows[1].Descripcion`; no se guarda ninguna de las dos filas. En la pantalla, el aviso es «Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa.»; si falla por otra causa, «Error al guardar los registros».

## 5. Jerarquías de unidades

Una jerarquía es una conversión de un producto («1 unidad mayor = N unidades menores»). Los casos usan el medicamento «Paracetamol Demo 500 mg» (`DEMO-MED-0001`) con unidad base `UND`. Los mensajes de esta pantalla vienen de 12b y de [19c sección 2.2][p19c]; el manual no la recorrió.

### HU-CAT-005 — Registrar y editar la conversión entre dos unidades de un producto

#### TC-CAT-034 — Registrar una conversión Caja a Blíster

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-005 «Registrar una conversión»; CU-CAT-05 flujo básico; [RN-JER-001 a 007][rn-b] |

**Precondiciones:** `JerarquiasUoM:Crear`, `Medicamentos:Leer` y `UnidadesMedida:Leer`; existen el medicamento y las unidades `CAJ` y `BLI`; el producto no tiene conversiones.
**Pasos:**
1. Abre «Jerarquías de Unidades» y haz clic en «Nuevo».
2. Escribe `Paracetamol` en «Producto» y elige «Paracetamol Demo 500 mg» de la lista.
3. Elige «Caja» en «Unidad Mayor» y «Blíster» en «Unidad Menor».
4. Escribe `10` en «Multiplicador» («Cuántas unidades menores equivalen a una unidad mayor») y haz clic en «Guardar».

**Resultado esperado:** aviso «Jerarquía guardada correctamente» y vuelves a `/jerarquias`; al buscar, el producto aparece con «Caja», «Blíster» y multiplicador 10.

#### TC-CAT-035 — Hay que elegir una opción de la lista

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-005 «Hay que elegir las opciones de la lista»; CU-CAT-05 excepción paso 4 (no se eligió una opción) |

**Pasos:**
1. En «Nuevo», escribe `Parace` en «Producto» sin elegir ninguna opción; escribe `Caj` en «Unidad Mayor» sin elegir; completa lo demás.
2. Haz clic en «Guardar».
3. Escribe un texto que no coincida con nada (`zzzz`) en «Producto».

**Resultado esperado:** debajo de cada campo ves «Este campo es obligatorio» y no se envía nada. Con `zzzz` la lista muestra una opción deshabilitada «Sin resultados».

#### TC-CAT-036 — Unidad mayor igual a la menor

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-005 «Unidad mayor igual a la menor»; CU-CAT-05 excepción paso 4 (misma unidad); [RN-JER-004][rn-b] |

**Pasos:** 1. Elige «Caja» en «Unidad Mayor» y en «Unidad Menor». 2. Haz clic en «Guardar».

**Resultado esperado:** debajo del formulario ves «La unidad mayor y la unidad menor no pueden ser la misma» y «Guardar» no envía nada.

#### TC-CAT-037 — Multiplicador vacío, cero o menor que 0,0001

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-005 «Multiplicador inválido»; CU-CAT-05 excepción paso 4 (multiplicador); [RN-JER-005][rn-b] |

**Datos:**

| Multiplicador | Resultado |
|---|---|
| (vacío) | «El multiplicador debe ser mayor que cero», no se envía nada |
| `0` | «El multiplicador debe ser mayor que cero», no se envía nada |
| `0,00009` | «El multiplicador debe ser mayor que cero», no se envía nada |
| `0,0001` | Se acepta (es el mínimo) |

**Pasos:** 1. Con las unidades «Caja» y «Blíster» de un producto sin conversión, prueba cada fila y guarda.

**Resultado esperado:** el de la tabla. El valor se guarda con 4 decimales. **No verificado:** si un valor con más decimales (por ejemplo `2,123456`) se redondea o se trunca al guardar.

#### TC-CAT-038 — Conversión repetida para el mismo producto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-005 «Conversión repetida»; CU-CAT-05 excepción paso 6 (conversión repetida); [RN-JER-008][rn-b] |

**Precondiciones:** el producto ya tiene «Caja a Blíster = 10».
**Pasos:** 1. Registra otra conversión con «Caja», «Blíster» y multiplicador 12.

**Resultado esperado:** la API responde 409 `JerarquiaUoM.Duplicada` y ves «Este producto ya tiene una conversión entre esas unidades.»

#### TC-CAT-039 — Conversión coherente con las existentes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-005 «La conversión es coherente con las existentes»; CU-CAT-05 A2; [RN-JER-009][rn-b], [RN-JER-010][rn-b] |

**Precondiciones:** el producto tiene «Caja a Blíster = 10» y «Blíster a Unidad = 10».
**Pasos:** 1. Registra «Caja a Unidad = 100».

**Resultado esperado:** se acepta («Jerarquía guardada correctamente»), porque coincide con la equivalencia calculada (1 caja = 10 blísteres = 100 unidades).

#### TC-CAT-040 — Conversión que contradice a las existentes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-005 «La conversión contradice a las existentes»; CU-CAT-05 excepción paso 7; [RN-JER-009][rn-b] |

**Precondiciones:** el producto tiene «Caja a Blíster = 10» y «Blíster a Unidad = 10».
**Pasos:** 1. Registra «Caja a Unidad = 50».

**Resultado esperado:** la API responde 409 `JerarquiaUoM.Multiplicador.Inconsistente` (compara ambos valores redondeados a 4 decimales) y ves «Error al guardar la jerarquía», sin el valor calculado. **Hallazgo (brecha de HU-CAT-005):** el código no tiene traducción.

#### TC-CAT-041 — Producto o unidad eliminados mientras se registra

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-CAT-005 «Producto o unidad que ya no existen»; CU-CAT-05 excepción paso 5; [RN-JER-006][rn-b], [RN-JER-007][rn-b] |

**Precondiciones:** dos sesiones de `Admin`; una unidad `PRU` sin uso.
**Pasos:** 1. En la sesión 1 llena el formulario con `PRU` como unidad menor sin guardar. 2. En la sesión 2 elimina `PRU`. 3. En la sesión 1 haz clic en «Guardar».

**Resultado esperado:** la API responde 404 `UnidadMedida.NotFound` (o `Medicamento.NotFound` si lo eliminado es el producto) y ves «Error al guardar la jerarquía».

#### TC-CAT-042 — Editar una conversión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-005 «Editar una conversión»; CU-CAT-05 A1; [RN-JER-001][rn-b], [RN-JER-012][rn-b] |

**Precondiciones:** `JerarquiasUoM:Actualizar`; el producto solo tiene «Caja a Blíster = 10».
**Pasos:**
1. Abre la jerarquía, haz clic en «Editar», cambia el multiplicador a 12 y guarda.
2. Registra «Caja a Unidad = 5» (no hay camino entre esas unidades, se acepta); ábrela, haz clic en «Editar», cambia la «Unidad Menor» a «Blíster» y guarda.

**Resultado esperado:** paso 1: «Jerarquía guardada correctamente» y, a diferencia de otros catálogos, vuelves a `/jerarquias` en vez de quedarte en el detalle. Paso 2: la edición vuelve a comprobar que no se duplique, así que ves «Este producto ya tiene una conversión entre esas unidades.» **No verificado:** el paso 2 se deduce de «vuelve a comprobar que no se duplique ni se contradiga» (12b); no hay un caso de uso que lo recorra.

#### TC-CAT-043 — La jerarquía ya no existe al guardar o abrir

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-CAT-005 «La jerarquía ya no existe»; CU-CAT-05 A1 (excepción `JerarquiaUoM.NotFound`); [RN-JER-013][rn-b] |

**Precondiciones:** dos sesiones de `Admin`; existe una jerarquía.
**Pasos:** 1. Sesión 1: abre la jerarquía y entra en «Editar». 2. Sesión 2: elimínala. 3. Sesión 1: guarda; luego abre el enlace guardado del detalle.

**Resultado esperado:** al guardar, 404 `JerarquiaUoM.NotFound` y «Error al guardar la jerarquía»; al abrir, «Error al cargar la jerarquía».

#### TC-CAT-044 — Solo API: identificador al editar una jerarquía

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-CAT-005 «Identificador al editar (solo API)»; CU-CAT-05 A1 (excepción `UpdateJerarquiaUoMCommand.Id`); [RN-JER-012][rn-b] |

**Datos:** `PUT /api/v1/JerarquiasUoM/00000000-0000-0000-0000-000000000000` con un cuerpo válido; luego `PUT` con el id de una jerarquía real y otro id distinto en el cuerpo.
**Resultado esperado:** la primera responde 400 `UpdateJerarquiaUoMCommand.Id`; la segunda modifica la jerarquía de la URL y descarta el id del cuerpo.

#### TC-CAT-045 — Cálculo de equivalencias entre unidades

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-005 «Cálculo de equivalencias»; CU-CAT-05 flujo básico paso 7; [RN-JER-010][rn-b] |

**Precondiciones:** producto con unidad base `UND`; se registran en este orden: «Caja a Blíster = 10», «Blíster a Unidad = 10».
**Datos:** luego: (a) «Caja a Unidad = 100»; (b) «Unidad a Caja = 0,01»; (c) «Unidad a Caja = 0,02»; (d) una conversión entre dos unidades sin camino con las demás (`FRA` a `ML` = 5).
**Pasos:** 1. Registra (a), (b), (c) y (d) en ese orden, anotando cada resultado.

**Resultado esperado:** (a) se acepta (de mayor a menor se multiplica: 10 × 10 = 100). (b) y (c) dependen de que el cálculo en sentido inverso divida: **No verificado en la aplicación** cuál se acepta; según RN-JER-010 (de menor a mayor se divide) el valor coherente es 0,01, y 0,02 debería rechazarse con 409 `JerarquiaUoM.Multiplicador.Inconsistente`. (d) la primera conversión sin camino conocido se acepta (no hay equivalencia que contradecir).

### HU-CAT-006 — Buscar, consultar y exportar jerarquías de unidades

#### TC-CAT-046 — Filtros disponibles y sugerencias en jerarquías

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-006 «Filtros disponibles»; CU-CAT-06 flujo básico paso 1; [RN-JER-001][rn-b] |

**Pasos:** 1. Abre `/jerarquias`. 2. Escribe `Para` en «Producto» y espera. 3. Escribe `Ca` en «Unidad Mayor».

**Resultado esperado:** ves «Producto» («Buscar producto...»), «Unidad Mayor» y «Unidad Menor» («Buscar unidad de medida...»), con lupa, y las sugerencias aparecen al escribir (espera de 300 ms, hasta 20). Ves un deslizador «Multiplicador: 0 — 1000» con paso 1.

#### TC-CAT-047 — Los filtros de texto buscan por nombre

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-006 «Los filtros de texto buscan por nombre»; CU-CAT-06 flujo básico paso 2; [RN-JER-015][rn-b] |

**Pasos:** 1. Escribe `paracetamol` (en minúsculas) en «Producto» sin elegir sugerencia y haz clic en «Buscar».

**Resultado esperado:** se listan las conversiones cuyo producto contiene el texto, sin distinguir mayúsculas.

#### TC-CAT-048 — Filtrar por multiplicador (valores en el borde)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-006 «Filtrar por multiplicador»; CU-CAT-06 flujo básico paso 3; [RN-JER-015][rn-b] |

**Precondiciones:** conversiones con multiplicador 4, 5, 20 y 21.
**Pasos:** 1. Mueve el deslizador a 5 — 20 y haz clic en «Buscar». 2. Déjalo en 0 — 1000 y busca.

**Resultado esperado:** con 5 — 20 ves solo las de multiplicador 5 y 20 (ambos inclusive). Solo se envía el límite que moviste: el mínimo si es mayor que 0 y el máximo si es menor que 1000; con 0 — 1000 ves todas. Con paso 1, el deslizador no permite decimales ni valores sobre 1000.

#### TC-CAT-049 — Tabla de resultados de jerarquías y exclusión de eliminados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-006 «Tabla de resultados»; «Sin resultados»; CU-CAT-06 flujo básico paso 4; [RN-JER-015][rn-b] |

**Precondiciones:** jerarquías de dos productos; una jerarquía cuya unidad fue eliminada después.
**Pasos:** 1. Busca sin filtros. 2. Busca con un filtro sin coincidencias.

**Resultado esperado:** columnas «Producto» (enlace al detalle), «Unidad Mayor», «Unidad Menor» y «Multiplicador» (a la derecha), ordenadas por nombre comercial. No aparecen las conversiones cuyo producto o unidades fueron eliminados. Sin coincidencias la tabla muestra «Sin resultados».

#### TC-CAT-050 — Exportar jerarquías

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-006 «Exportar»; CU-CAT-06 A1; [RN-JER-016][rn-b], [RN-IMP-008][rn-a], [RN-IMP-009][rn-a] |

**Pasos:** 1. Busca con «Producto» `Paracetamol` y multiplicador 5 — 20. 2. Haz clic en «Exportar».

**Resultado esperado:** se descarga `jerarquias-unidades.xlsx` con las columnas «Producto», «Unidad Mayor», «Unidad Menor» y «Multiplicador», aplicando los filtros de nombre y de multiplicador escritos. Si falla la búsqueda o la exportación (o hay más de 10000 filas, `Export.TooManyRows`), ves «Error al buscar jerarquías de unidades». **No verificado en la aplicación:** el contenido del archivo.

### HU-CAT-007 — Eliminar una conversión de unidades

#### TC-CAT-051 — Eliminar una conversión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-007 «Eliminar una conversión»; CU-CAT-07 flujo básico y A1; [RN-JER-013][rn-b] |

**Precondiciones:** `JerarquiasUoM:Eliminar`; una conversión sin presentaciones.
**Pasos:** 1. Edita la jerarquía y haz clic en «Eliminar»; cancela y repite; 2. confirma.

**Resultado esperado:** «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar esta jerarquía?»; «Cancelar» no borra; al confirmar ves «Jerarquía eliminada correctamente» y vuelves a `/jerarquias`.

#### TC-CAT-052 — Hallazgo: la API no valida si una presentación usa la conversión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-007 «La API no valida dependencias»; CU-CAT-07 flujo básico paso 4; [RN-JER-014][rn-b], [RN-PRE-007][rn-b] |

**Precondiciones:** una presentación de venta de «Caja» creada con la conversión «Caja a Blíster = 10».
**Pasos:** 1. Elimina esa conversión. 2. Abre la presentación.

**Resultado esperado:** **Hallazgo (pendiente 7 de 12b 19.2):** la conversión se elimina igualmente (204) y la presentación conserva el valor ya guardado. **No verificado:** si la presentación se recalcula (por el código leído no se recalcula).

#### TC-CAT-053 — Aviso de «en uso» ante un 409 y jerarquía inexistente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-CAT-007 «Aviso de "en uso" ante un conflicto»; «La jerarquía ya no existe»; CU-CAT-07 excepción paso 4 |

**Precondiciones:** dos sesiones; una jerarquía.
**Pasos:** 1. Sesión 1: entra en «Editar» y «Eliminar». 2. Sesión 2: elimina la jerarquía. 3. Sesión 1: confirma.

**Resultado esperado:** 404 `JerarquiaUoM.NotFound` y «Error al eliminar la jerarquía». El aviso «No se puede eliminar la jerarquía porque está en uso.» aparece ante cualquier 409, pero el backend nunca lo devuelve al eliminar una jerarquía: **No verificado en la aplicación** (no se puede provocar; viene de la brecha de HU-CAT-007).

### HU-CAT-008 — Cargar jerarquías de unidades desde un Excel

#### TC-CAT-054 — Columnas de la plantilla y límites por fila

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-008 «Columnas de la plantilla»; 12b §1.3 «CM-1 Descargar la plantilla»; CU-CAT-08 flujo básico paso 1; [RN-JER-017][rn-b], [RN-JER-020][rn-b] |

**Pasos:** 1. Descarga la plantilla. 2. Sube un archivo con una fila de `Producto` de 151 caracteres, `UnidadMayor` de 51 y `Multiplicador` de 31 caracteres.

**Resultado esperado:** `plantilla-jerarquias.xlsx` con las columnas «Producto», «UnidadMayor», «UnidadMenor» y «Multiplicador», todas obligatorias. Los límites por fila son Producto 150, UnidadMayor y UnidadMenor 50, Multiplicador 30; la fila larga queda con error de longitud (**No verificado:** el texto exacto del mensaje; en unidades es «'codigo' no puede superar los 10 caracteres»).

#### TC-CAT-055 — Producto y unidades se identifican por nombre

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-008 «El producto y las unidades se identifican por nombre»; CU-CAT-08 excepción paso 2 (`not-found`, `ambiguous`); [RN-JER-018][rn-b] |

**Precondiciones:** existen dos medicamentos con el mismo nombre comercial «Paracetamol Demo 500 mg» (o dos unidades con la misma descripción «Caja»).
**Datos:** fila 1: `Paracetamol Demo 500 mg` / `caja` / `BLÍSTER` / `10` (con el nombre único); fila 2: producto `Inexistente Demo`; fila 3: nombre duplicado en `UnidadMayor`.
**Pasos:** 1. Sube el archivo.

**Resultado esperado:** la fila 1 es válida (el producto se busca por nombre comercial y las unidades por descripción, no por código, sin distinguir mayúsculas). Fila 2: «El valor 'Inexistente Demo' del campo 'producto' no existe en el sistema». Fila 3: «El valor '<valor>' del campo 'unidadMayor' coincide con varios registros del sistema».

#### TC-CAT-056 — Reglas por fila de la vista previa de jerarquías

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-008 «Reglas por fila en la vista previa»; 12b §1.3 «CM-3 Vista previa sin guardar nada»; CU-CAT-08 excepción paso 2; [RN-JER-002 a 005][rn-b], [RN-JER-019][rn-b] |

**Datos:** filas: (1) campo vacío (`Producto` sin valor); (2) `UnidadMayor` y `UnidadMenor` iguales; (3) `Multiplicador` `abc`; (4) `Multiplicador` `0`; (5) `Multiplicador` `-3`; (6) `Multiplicador` `1.5`.
**Pasos:** 1. Sube el archivo.

**Resultado esperado:** (1) «El campo 'producto' es obligatorio»; (2) «'unidadMenor' no puede ser igual a 'unidadMayor'»; (3) «'multiplicador' debe ser un número decimal válido»; (4) y (5) «'multiplicador' debe ser mayor que cero»; (6) válida (el multiplicador se lee con punto como separador decimal).

#### TC-CAT-057 — Multiplicador con coma decimal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | HU-CAT-008 «Reglas por fila en la vista previa» (multiplicador con coma); CU-CAT-08 excepción paso 2; [RN-JER-019][rn-b] |

**Datos:** una fila con `Multiplicador` `1,5` (texto).
**Resultado esperado:** **No verificado:** con cultura invariante .NET podría tomar la coma como separador de miles (`1,5` se leería como 15), según el aviso de HU-CAT-008. El tester anota qué ve la vista previa y qué guarda.

#### TC-CAT-058 — Hallazgo: la vista previa no anticipa duplicados ni contradicciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-008 «La vista previa no anticipa duplicados ni contradicciones»; CU-CAT-08 excepción paso 4 (fila duplicada o inconsistente); [RN-JER-008][rn-b], [RN-JER-009][rn-b], [RN-JER-021][rn-b] |

**Precondiciones:** el producto tiene «Caja a Blíster = 10».
**Datos:** archivo con (1) `Caja` / `Blíster` / `12` (repite una conversión existente); (2) «Blíster a Unidad = 10» y (3) «Caja a Unidad = 50» (contradice lo calculable por (2) y la existente).
**Pasos:** 1. Sube el archivo y mira la vista previa. 2. Haz clic en «Guardar válidos (3)». 3. Busca las jerarquías del producto.

**Resultado esperado:** **Hallazgo (pendiente 3 de 12b 19.2):** la vista previa dice «3 filas válidas»; el guardado procesa en orden, guarda (2) y descarta sin error (1) y (3); el aviso es «Se guardaron 1 registros correctamente» (menos que las filas válidas).

#### TC-CAT-059 — Carga de jerarquías: guardar válidas y filas fuera de rango

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-008 «Reglas por fila en la vista previa»; 12b §1.3 «CM-5 Guardar solo las filas válidas»; CU-CAT-08 excepciones pasos 3 y 4; [RN-JER-016][rn-b], [RN-JER-022][rn-b] |

**Datos:** `POST /api/v1/JerarquiasUoM/carga-masiva` con lista vacía y con 5001 filas; `POST /api/v1/JerarquiasUoM/previsualizar` sin archivo; carga con una fila cuyo `Producto` tiene 151 caracteres.
**Resultado esperado:** lista vacía o de más de 5000: 400 `CargaMasivaJerarquiasUoMCommand.Rows`; vista previa sin archivo: 400 «Debe adjuntar un archivo Excel.»; fila larga: 400 `CargaMasivaJerarquiasUoMCommand.Rows[0].Producto`.

### HU-CAT-009 — Conocer en qué unidades se puede vender un producto

#### TC-CAT-060 — Solo API: unidades vendibles de un producto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-009 «Unidades vendibles de un producto»; CU-CAT-09 flujo básico; [RN-JER-010][rn-b], [RN-JER-011][rn-b] |

**Precondiciones:** token con `JerarquiasUoM:Leer`; el producto tiene unidad base «Unidad» y las conversiones «Caja a Blíster = 10» y «Blíster a Unidad = 10».
**Datos:** `GET /api/v1/JerarquiasUoM/unidades-vendibles?productoId=<id-producto>`.
**Resultado esperado:** 200 con «Caja» (`cantidadUnidadesBase` 100) y «Blíster» (10), en ese orden (de mayor a menor), sin la unidad base.

#### TC-CAT-061 — Solo API: unidad sin camino, producto sin jerarquías, producto inexistente y sin permiso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-009 «Unidades sin camino hasta la base»; «Producto sin jerarquías»; «El producto no existe»; CU-CAT-09 A1 y excepciones pasos 1 y 2; [RN-JER-011][rn-b] |

**Precondiciones:** (a) un producto con la conversión «Frasco a Mililitro = 5» pero con unidad base «Unidad» (sin camino); (b) un producto sin conversiones; (c) un id inexistente o de la empresa B; (d) un token sin `JerarquiasUoM:Leer`.
**Resultado esperado:** (a) la lista no incluye `FRA` ni `ML`; (b) lista vacía (y en la pantalla de presentaciones, «Este producto no tiene una Jerarquía de Unidades registrada. Cárgala primero para poder definir presentaciones de venta.»); (c) 404 `Medicamento.NotFound`; (d) 403.

## 6. Laboratorios

### HU-CAT-010 — Registrar y editar un laboratorio

#### TC-CAT-062 — Registrar un laboratorio con código

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-010 «Registrar un laboratorio con código»; CU-CAT-10 flujo básico; [RN-LAB-001 a 004][rn-b] |

**Precondiciones:** `Laboratorios:Crear`.
**Datos:** Nombre de Compañía `Laboratorio Demo Andino`; Código Identificador `DEMO-AND`.
**Pasos:** 1. En «Laboratorios» haz clic en «Nuevo». 2. Escribe los datos y haz clic en «Guardar».

**Resultado esperado:** aviso «Laboratorio guardado correctamente» y vuelves a la búsqueda.

#### TC-CAT-063 — El código identificador es opcional y puede haber varios sin código

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-010 «El código identificador es opcional»; CU-CAT-10 A2; [RN-LAB-004][rn-b], [RN-LAB-005][rn-b] |

**Datos:** (a) `Laboratorio Demo Pacífico` sin código; (b) `Laboratorio Sin Código Dos` con el código escrito como 3 espacios.
**Pasos:** 1. Registra (a). 2. Registra (b). 3. Busca sin filtros.

**Resultado esperado:** ambos se guardan con «Laboratorio guardado correctamente»; en la tabla los dos aparecen con «—» como código (un código vacío o de solo espacios se guarda como «sin código» y no choca con otros).

#### TC-CAT-064 — Nombre obligatorio y longitudes de un laboratorio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-010 «Nombre obligatorio y longitudes»; CU-CAT-10 excepciones pasos 2 y 4; [RN-LAB-002][rn-b], [RN-LAB-003][rn-b], [RN-LAB-004][rn-b] |

**Datos:** nombre vacío; luego un nombre de 151 letras y un código de 51 caracteres.
**Pasos:** 1. Guarda con el nombre vacío. 2. Escribe los valores largos.

**Resultado esperado:** «Este campo es obligatorio» debajo de «Nombre de Compañía» y no se envía nada; el nombre no deja pasar de 150 caracteres y el código no deja pasar de 50, sin mensaje. Con solo espacios en el nombre: **Hallazgo (brecha de HU-CAT-010):** ves «Error al guardar el laboratorio».

#### TC-CAT-065 — Nombre de laboratorio repetido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-010 «Nombre repetido»; CU-CAT-10 excepción paso 5; [RN-LAB-006][rn-b], [RN-LAB-007][rn-b] |

**Precondiciones:** existe «Laboratorio Demo Andino» con código `DEMO-AND`.
**Datos:** nombre `laboratorio demo andino` con código `DEMO-AND`.
**Pasos:** 1. Intenta registrarlo.

**Resultado esperado:** «Ya existe un laboratorio con ese nombre de compañía.» (la comprobación de nombre se evalúa antes que la de código, así que no ves el aviso del código). **No verificado:** que se rechace con otra capitalización depende de la intercalación (el manual lo comprobó).

#### TC-CAT-066 — Código identificador repetido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-010 «Código repetido»; CU-CAT-10 excepción paso 5; [RN-LAB-007][rn-b] |

**Precondiciones:** `DEMO-AND` lo usa «Laboratorio Demo Andino»; existe un laboratorio sin código.
**Pasos:** 1. Registra «Laboratorio Otro» con el código `DEMO-AND`. 2. Registra «Laboratorio Otro» sin código.

**Resultado esperado:** el primero muestra «Ya existe un laboratorio con ese código identificador.»; el segundo se guarda (los laboratorios sin código no entran en la comprobación).

#### TC-CAT-067 — Editar un laboratorio y quitarle el código

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-010 «Editar un laboratorio y quitarle el código»; CU-CAT-10 A1; [RN-LAB-005][rn-b] |

**Precondiciones:** laboratorio con código `PRUEBA-01`; `Laboratorios:Actualizar`.
**Pasos:** 1. Abre el laboratorio, haz clic en «Editar», borra el código, cambia el nombre y guarda.

**Resultado esperado:** «Laboratorio guardado correctamente»; te quedas en el detalle con el código vacío.

#### TC-CAT-068 — Nombre y código de un laboratorio eliminado quedan libres

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-010 «Nombre y código de un laboratorio eliminado quedan libres»; CU-CAT-10 flujo básico; [RN-LAB-011][rn-b] |

**Precondiciones:** laboratorio «Laboratorio Prueba Borrar» con código `BORRAR-01`, sin medicamentos.
**Pasos:** 1. Elimínalo (`TC-CAT-073`). 2. Registra otro con el mismo nombre y código.

**Resultado esperado:** se guarda sin conflicto.

#### TC-CAT-069 — El laboratorio ya no existe al guardar o abrir

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-CAT-010 «El laboratorio ya no existe»; CU-CAT-10 A1 (excepción `Laboratorio.NotFound`); [RN-LAB-009][rn-b] |

**Pasos:** 1. Sesión 1: abre un laboratorio y entra en «Editar». 2. Sesión 2: elimínalo. 3. Sesión 1: guarda y luego abre su enlace.

**Resultado esperado:** 404 `Laboratorio.NotFound`; al guardar «Error al guardar el laboratorio» y al abrir «Error al cargar el laboratorio».

#### TC-CAT-070 — Solo API: identificador al editar un laboratorio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-CAT-010 «Identificador al editar (solo API)»; CU-CAT-10 A1 (excepción `UpdateLaboratorioCommand.Id`); [RN-LAB-008][rn-b] |

**Datos:** `PUT /api/v1/Laboratorios/00000000-0000-0000-0000-000000000000` con cuerpo válido; luego `PUT` con un id real en la URL y otro distinto en el cuerpo.
**Resultado esperado:** 400 `UpdateLaboratorioCommand.Id`; la segunda modifica el laboratorio de la URL.

### HU-CAT-011 — Buscar, consultar y exportar laboratorios

#### TC-CAT-071 — Buscar y filtrar laboratorios

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-011 «Buscar sin filtros»; «Filtrar»; CU-CAT-11 flujo básico; [RN-LAB-012][rn-b] |

**Precondiciones:** existen solo «Laboratorio Demo Andino» (`DEMO-AND`) y «Laboratorio Demo Pacífico» (sin código).
**Pasos:** 1. Busca sin filtros. 2. «Nombre de Compañía» `demo`. 3. «Código Identificador» `and`. 4. «Código Identificador» `zzz`.

**Resultado esperado:** (1) «Resultados (2)» con los dos, el segundo con «—»; columnas «Nombre de Compañía» (enlace al detalle) y «Código Identificador», ordenada por nombre. (2) los dos. (3) solo «Laboratorio Demo Andino». (4) «Resultados (0)», la fila «Resultados» y el paginador «Sin resultados».

#### TC-CAT-072 — Exportar laboratorios

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-011 «Exportar»; CU-CAT-11 A1; [RN-LAB-012][rn-b], [RN-IMP-008][rn-a], [RN-IMP-009][rn-a] |

**Pasos:** 1. Busca con «Nombre de Compañía» `demo`. 2. Haz clic en «Exportar».

**Resultado esperado:** se descarga `laboratorios.xlsx` con «Nombre de Compañía» y «Código Identificador» (vacío si no tiene), con los filtros escritos. Si falla la búsqueda o la exportación (o más de 10000 filas, `Export.TooManyRows`): «Error al buscar laboratorios». **No verificado en la aplicación:** el contenido del archivo.

### HU-CAT-012 — Eliminar un laboratorio

#### TC-CAT-073 — Eliminar un laboratorio sin medicamentos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-012 «Eliminar un laboratorio sin medicamentos»; CU-CAT-12 flujo básico y A1; [RN-LAB-009][rn-b], [RN-LAB-011][rn-b] |

**Precondiciones:** «Laboratorio Prueba Borrar» sin medicamentos; `Laboratorios:Eliminar`.
**Pasos:** 1. Entra en «Editar» y haz clic en «Eliminar». 2. Cancela. 3. Repite y confirma.

**Resultado esperado:** «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este laboratorio?»; «Cancelar» no borra; al confirmar ves «Laboratorio eliminado correctamente» y vuelves a la búsqueda.

#### TC-CAT-074 — Bloqueo al eliminar un laboratorio con medicamentos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-012 «Laboratorio con medicamentos»; CU-CAT-12 excepción paso 4 (`Laboratorio.InUse`); [RN-LAB-010][rn-b] |

**Precondiciones:** `DEMO-MED-0001` pertenece a «Laboratorio Demo Andino».
**Pasos:** 1. Abre el laboratorio, «Editar», «Eliminar», «Confirmar». 2. Elimina el medicamento y repite.

**Resultado esperado:** 409 `Laboratorio.InUse` y «No se puede eliminar el laboratorio porque tiene medicamentos asociados.»; el laboratorio sigue existiendo. Tras eliminar el medicamento se puede eliminar. **Hallazgo:** la pantalla muestra ese texto ante cualquier 409. **No verificado en la aplicación:** el bloqueo (el manual lo dejó pendiente).

#### TC-CAT-075 — Otro fallo al eliminar un laboratorio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Baja | HU-CAT-012 «Otro fallo al eliminar»; CU-CAT-12 excepción paso 4 (`Laboratorio.NotFound`) |

**Pasos:** 1. Sesión 1: «Editar» y «Eliminar». 2. Sesión 2: elimina el laboratorio. 3. Sesión 1: «Confirmar».

**Resultado esperado:** 404 `Laboratorio.NotFound` y «Error al eliminar el laboratorio».

### HU-CAT-013 — Cargar laboratorios desde un Excel

#### TC-CAT-076 — Columnas de la plantilla de laboratorios

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-013 «Columnas de la plantilla»; CU-CAT-13 flujo básico paso 1; [RN-LAB-013][rn-b] |

**Pasos:** 1. Abre `/laboratorios/carga-masiva` y descarga la plantilla.

**Resultado esperado:** `plantilla-laboratorios.xlsx` con «NombreCompania» (obligatoria, máximo 150 caracteres) y «CodigoIdentificador» (opcional, máximo 50). Los escenarios CM-1 a CM-5 se prueban como en `TC-CAT-025` a `TC-CAT-033`.

#### TC-CAT-077 — Reglas por fila de la vista previa de laboratorios

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-013 «Reglas por fila en la vista previa»; CU-CAT-13 excepción paso 2; [RN-LAB-002][rn-b], [RN-LAB-003][rn-b], [RN-LAB-006][rn-b], [RN-LAB-014][rn-b] |

**Precondiciones:** existe «Laboratorio Demo Andino».
**Datos:** filas: (1) `Laboratorio Demo Andino`; (2) nombre vacío; (3) nombre de 160 letras; (4) y (5) `Laboratorio Repetido` dos veces; (6) `Laboratorio Nuevo Demo`.
**Pasos:** 1. Sube el archivo.

**Resultado esperado:** (1) «El valor 'Laboratorio Demo Andino' del campo 'nombreCompania' ya existe en el sistema»; (2) «El campo 'nombreCompania' es obligatorio»; (3) «'nombreCompania' no puede superar los 150 caracteres»; (4) y (5) las dos filas marcadas como duplicadas en el archivo; (6) válida.

#### TC-CAT-078 — Hallazgo: un código identificador ya usado pasa la vista previa y se omite al guardar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-013 «Un código identificador ya usado pasa la vista previa y se omite al guardar»; CU-CAT-13 excepción paso 3 (código ya usado); [RN-LAB-007][rn-b], [RN-LAB-015][rn-b], [RN-LAB-016][rn-b] |

**Precondiciones:** `DEMO-AND` ya lo usa «Laboratorio Demo Andino».
**Datos:** una sola fila: `Laboratorio Demo Código Repetido` con código `DEMO-AND`.
**Pasos:** 1. Sube el archivo. 2. Haz clic en «Guardar válidos (1)». 3. Busca el laboratorio.

**Resultado esperado:** **Hallazgo (pendiente 3 de 12b 19.2):** la vista previa la muestra válida; al guardar el servidor omite la fila y ves «Se guardaron 0 registros correctamente» y vuelves a la búsqueda; el laboratorio nuevo no existe.

#### TC-CAT-079 — Solo API: carga de laboratorios con filas fuera de rango o inválidas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-013 «Reglas por fila en la vista previa»; CU-CAT-13 excepción paso 3 (validador y lista de filas); [RN-LAB-016][rn-b], [RN-LAB-017][rn-b] |

**Datos:** `POST /api/v1/Laboratorios/carga-masiva` con lista vacía; con 5001 filas; con una fila de nombre de 151 caracteres. `POST /api/v1/Laboratorios/previsualizar` sin archivo.
**Resultado esperado:** 400 `CargaMasivaLaboratoriosCommand.Rows` para la lista vacía y la de 5001; 400 `CargaMasivaLaboratoriosCommand.Rows[0].NombreCompania` para la fila larga; 400 «Debe adjuntar un archivo Excel.» para la vista previa sin archivo.

## 7. Ingredientes activos

### HU-CAT-014 — Registrar y editar un ingrediente activo

#### TC-CAT-080 — Registrar un ingrediente activo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-014 «Registrar un ingrediente»; CU-CAT-14 flujo básico; [RN-ING-001 a 005][rn-b] |

**Datos:** Denominación DCI `Paracetamol`; Código ATC `N02BE01`.
**Pasos:** 1. En «Ingredientes Activos» haz clic en «Nuevo». 2. Escribe los datos y guarda.

**Resultado esperado:** «Ingrediente activo guardado correctamente» y vuelves a la búsqueda.

#### TC-CAT-081 — Campos obligatorios y longitudes de un ingrediente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-014 «Campos obligatorios y longitudes»; CU-CAT-14 excepciones pasos 2 y 4; [RN-ING-002 a 005][rn-b] |

**Pasos:** 1. Guarda con los dos campos vacíos. 2. Escribe 256 letras en la DCI y 16 en el ATC. 3. Prueba solo espacios.

**Resultado esperado:** «Este campo es obligatorio» bajo cada campo y no se envía nada; la DCI no pasa de 255 y el ATC de 15, sin mensaje. Con solo espacios: **Hallazgo:** «Error al guardar el ingrediente activo».

#### TC-CAT-082 — La DCI no puede repetirse

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-014 «La DCI no puede repetirse»; CU-CAT-14 excepción paso 5; [RN-ING-006][rn-b] |

**Precondiciones:** existe «Paracetamol».
**Pasos:** 1. Registra `paracetamol` con ATC `ZZ01`.

**Resultado esperado:** «Ya existe un ingrediente activo con esa denominación DCI.»

#### TC-CAT-083 — El código ATC no se valida ni es único

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-014 «El código ATC no se valida ni es único»; CU-CAT-14 flujo básico; [RN-ING-005][rn-b], [RN-ING-006][rn-b] |

**Datos:** (a) «Ingrediente Libre Uno» con ATC `XXX`; (b) «Ingrediente Libre Dos» con ATC `N02BE01` (ya usado).
**Resultado esperado:** ambos se guardan con «Ingrediente activo guardado correctamente».

#### TC-CAT-084 — Editar un ingrediente activo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-014 «Editar un ingrediente»; CU-CAT-14 A1; [RN-ING-007][rn-b] |

**Pasos:** 1. Abre «Ingrediente Prueba Borrar», «Editar», cambia el ATC a `XXX` y guarda.

**Resultado esperado:** «Ingrediente activo guardado correctamente» y te quedas en el detalle.

#### TC-CAT-085 — El ingrediente ya no existe y el id al editar (solo API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Baja | HU-CAT-014 «El ingrediente ya no existe»; «Identificador al editar (solo API)»; CU-CAT-14 A1; [RN-ING-007][rn-b], [RN-ING-008][rn-b] |

**Pasos:** 1. Sesión 1: edita un ingrediente. 2. Sesión 2: elimínalo. 3. Sesión 1: guarda y abre su enlace. 4. API: `PUT /api/v1/IngredientesActivos/00000000-0000-0000-0000-000000000000` con cuerpo válido.

**Resultado esperado:** 404 `IngredienteActivo.NotFound`: «Error al guardar el ingrediente activo» y «Error al cargar el ingrediente activo». El paso 4: 400 `UpdateIngredienteActivoCommand.Id`.

### HU-CAT-015 — Buscar, consultar y exportar ingredientes activos

#### TC-CAT-086 — Buscar ingredientes con filtros combinados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-015 «Buscar con filtros combinados»; CU-CAT-15 flujo básico; [RN-ING-010][rn-b] |

**Precondiciones:** existen Paracetamol (`N02BE01`), Amoxicilina (`J01CA04`), Clonazepam (`N03AE01`) y Tramadol (`N02AX02`).
**Pasos:** 1. «Denominación DCI» `CIL`. 2. «Código ATC» `n02`. 3. `para` y `n02` juntos.

**Resultado esperado:** (1) «Amoxicilina»; (2) los ingredientes cuyo código contiene «N02» (Paracetamol y Tramadol); (3) solo «Paracetamol». La tabla tiene «Denominación DCI» (enlace al detalle) y «Código ATC», ordenada por denominación.

#### TC-CAT-087 — Sin coincidencias y exportar ingredientes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-015 «Sin coincidencias»; «Exportar»; CU-CAT-15 A1; [RN-ING-010][rn-b], [RN-IMP-008][rn-a] |

**Pasos:** 1. Busca `zzz`. 2. Busca sin filtros y haz clic en «Exportar».

**Resultado esperado:** «Resultados (0)», fila «Resultados» y paginador «Sin resultados». Se descarga `ingredientes-activos.xlsx` con «Denominación DCI» y «Código ATC». Si falla: «Error al buscar ingredientes activos». **No verificado en la aplicación:** el contenido del archivo.

### HU-CAT-016 — Eliminar un ingrediente activo

#### TC-CAT-088 — Eliminar un ingrediente sin formulaciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-016 «Eliminar un ingrediente sin formulaciones»; CU-CAT-16 flujo básico y A1; [RN-ING-009][rn-b] |

**Pasos:** 1. Edita «Ingrediente Prueba Borrar», «Eliminar», «Confirmar».

**Resultado esperado:** «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este ingrediente activo?»; al confirmar «Ingrediente activo eliminado correctamente»; la API responde 204.

#### TC-CAT-089 — Bloqueo al eliminar un ingrediente incluido en formulaciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-016 «Ingrediente incluido en formulaciones clínicas»; CU-CAT-16 excepción paso 4 (`IngredienteActivo.InUse`); [RN-ING-009][rn-b] |

**Precondiciones:** una formulación clínica no eliminada usa «Paracetamol».
**Pasos:** 1. Elimina «Paracetamol». 2. Elimina la formulación y repite.

**Resultado esperado:** 409 `IngredienteActivo.InUse` y «No puedes eliminar este ingrediente activo porque está incluido en formulaciones clínicas.» (bloqueo duro). Sin formulaciones, se elimina. **No verificado en la aplicación:** el bloqueo (pendiente en el manual).

#### TC-CAT-090 — Otro fallo al eliminar un ingrediente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Baja | HU-CAT-016 «Otro fallo al eliminar»; CU-CAT-16 excepción paso 4 (`IngredienteActivo.NotFound`); [RN-ING-008][rn-b] |

**Pasos:** 1. Sesión 1: «Editar», «Eliminar». 2. Sesión 2: elimina el ingrediente. 3. Sesión 1: «Confirmar».

**Resultado esperado:** 404 `IngredienteActivo.NotFound` y «Error al eliminar el ingrediente activo».

### HU-CAT-017 — Cargar ingredientes activos desde un Excel

#### TC-CAT-091 — Columnas y reglas por fila de ingredientes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-017 «Columnas de la plantilla»; «Reglas por fila en la vista previa»; CU-CAT-17 excepción paso 2 (`required`, `max-length`); [RN-ING-002 a 005][rn-b], [RN-ING-011][rn-b] |

**Datos:** plantilla `plantilla-ingredientes-activos.xlsx` («DenominacionDci» obligatoria máximo 255 y «CodigoAtc» obligatoria máximo 15); filas: DCI vacía; ATC vacío; DCI de 256 letras; ATC de 16 caracteres.
**Resultado esperado:** «El campo 'denominacionDci' es obligatorio»; «El campo 'codigoAtc' es obligatorio»; «'denominacionDci' no puede superar los 255 caracteres»; «'codigoAtc' no puede superar los 15 caracteres».

#### TC-CAT-092 — Hallazgo: la vista previa controla el código ATC, no la DCI

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-017 «La vista previa controla el código ATC, no la DCI»; CU-CAT-17 excepción paso 2 (`duplicate`, `duplicate-in-file`); [RN-ING-006][rn-b], [RN-ING-012][rn-b] |

**Precondiciones:** «Paracetamol» usa `N02BE01`.
**Datos:** filas: «Diazepam» / `N02BE01`; «Salbutamol» / `R03AC02` y «Terbutalina» / `R03AC02`.
**Resultado esperado:** **Hallazgo (pendiente 4 de 12b 19.2):** la primera: «El valor 'N02BE01' del campo 'codigoAtc' ya existe en el sistema»; las dos siguientes: «El valor 'R03AC02' del campo 'codigoAtc' está duplicado en el archivo». Crear esos ingredientes a mano sí permitiría repetir el ATC (`TC-CAT-083`).

#### TC-CAT-093 — Hallazgo: una DCI ya registrada pasa la vista previa y se omite al guardar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-017 «Una DCI ya registrada pasa la vista previa y se omite al guardar»; CU-CAT-17 excepción paso 3 (DCI ya existente); [RN-ING-013][rn-b], [RN-ING-014][rn-b], [RN-ING-015][rn-b] |

**Precondiciones:** existe «Paracetamol».
**Datos:** tres filas: «Paracetamol» / `ZZ99`; «Ibuprofeno» / `M01AE01`; «Omeprazol» / `A02BC01`.
**Pasos:** 1. Sube el archivo. 2. Haz clic en «Guardar válidos (3)».

**Resultado esperado:** **Hallazgo:** las tres aparecen válidas; el servidor omite «Paracetamol» (y una repetida del archivo, conservando la primera); ves «Se guardaron 2 registros correctamente» y vuelves a la búsqueda. Solo API: lista vacía o de más de 5000 → 400 `CargaMasivaIngredientesActivosCommand.Rows`; fila inválida → `CargaMasivaIngredientesActivosCommand.Rows[i].DenominacionDci` o `.CodigoAtc`; vista previa sin archivo → «Debe adjuntar un archivo Excel.»

## 8. Medicamentos

El medicamento es el artículo que la botica vende: código nacional, nombre comercial, laboratorio, unidad base, condición de venta y precio de venta base. Los medicamentos controlados son los de condición «Receta Retenida» y «Estupefaciente». Recorrido del manual: `DEMO-MED-0001` «Paracetamol Demo 500 mg» (Venta Libre (OTC)), `DEMO-MED-0002` Amoxicilina (Receta Simple), `DEMO-MED-0003` Clonazepam (Receta Retenida) y `DEMO-MED-0004` Tramadol (Estupefaciente), según las búsquedas de 12b (HU-CAT-019).

### HU-CAT-018 — Registrar y editar un medicamento

#### TC-CAT-094 — Registrar un medicamento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-018 «Registrar un medicamento»; CU-CAT-18 flujo básico; [RN-MED-001 a 008][rn-b], [RN-MED-010][rn-b] |

**Precondiciones:** `Medicamentos:Crear`, `Laboratorios:Leer`, `UnidadesMedida:Leer`; existen «Laboratorio Demo Andino» y la unidad `UND`.
**Datos:** Código Nacional `DEMO-MED-0001`; Nombre Comercial `Paracetamol Demo 500 mg`; Laboratorio: escribe `Andino` y elige «Laboratorio Demo Andino»; Unidad Base: escribe `Unidad` y elige «UND Unidad»; Condición de Venta «Venta Libre (OTC)»; Precio de Venta Base `0.50` (prefijo «S/»).
**Pasos:** 1. «Nuevo». 2. Completa los campos. 3. «Guardar».

**Resultado esperado:** aviso «Medicamento guardado correctamente» y vuelves a la búsqueda con los filtros vacíos y sin tabla.

#### TC-CAT-095 — Registrar medicamentos controlados (Receta Retenida y Estupefaciente)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-018 «Registrar un medicamento»; «La lista de condiciones de venta viene de la API»; CU-CAT-18 flujo básico y excepción paso 2; [RN-MED-007][rn-b], [RN-MED-019][rn-b] |

**Datos:** `DEMO-MED-0003` «Clonazepam Demo 0.5 mg» con «Receta Retenida»; `DEMO-MED-0004` «Tramadol Demo 50 mg» con «Estupefaciente»; ambos con laboratorio, unidad base y precio válidos.
**Pasos:** 1. Abre el formulario y despliega «Condición de Venta». 2. Registra los dos medicamentos. 3. Busca por «Condición de Venta» «Receta Retenida» y luego «Estupefaciente».

**Resultado esperado:** «Condición de Venta» ofrece, en este orden, «Venta Libre (OTC)», «Receta Simple», «Receta Retenida» y «Estupefaciente». Cada medicamento se guarda con «Medicamento guardado correctamente» y aparece solo con su condición. Si la lista no carga ves «Error al cargar las condiciones de venta» (**No verificado en la aplicación**: no se puede provocar sin fallar la API). Cómo se exige la receta al vender estos productos pertenece a la parte C y a `TC-INV-082` y `TC-INV-088` de recetas.

#### TC-CAT-096 — Campos obligatorios vacíos de un medicamento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-018 «Campos obligatorios vacíos»; «Hay que elegir el laboratorio y la unidad de la lista»; CU-CAT-18 excepción paso 4; [RN-MED-002][rn-b], [RN-MED-004][rn-b], [RN-MED-005][rn-b], [RN-MED-006][rn-b] |

**Pasos:** 1. Con el formulario vacío haz clic en «Guardar». 2. Escribe `zzz` en «Laboratorio» sin elegir opción y guarda.

**Resultado esperado:** «Este campo es obligatorio» bajo Código Nacional, Nombre Comercial, Laboratorio, Unidad Base y Condición de Venta; bajo «Precio de Venta Base» ves «El precio no puede ser negativo» (también con el campo vacío: **Hallazgo, brecha 2**, no hay texto de «obligatorio» para el precio). No se envía nada. Con `zzz` la lista muestra «Sin resultados» y debajo del campo «Este campo es obligatorio».

#### TC-CAT-097 — Precio de venta base: negativo, cero y decimales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-018 «Precio negativo»; CU-CAT-18 excepción paso 4; [RN-MED-008][rn-b] |

**Datos:** `-1`, `0`, `0.50`, `0.555`.
**Pasos:** 1. Guarda un medicamento con cada precio.

**Resultado esperado:** `-1`: «El precio no puede ser negativo» y no se envía nada; `0` es válido y se guarda con 2 decimales; `0.50` válido. **No verificado:** si `0.555` se redondea o trunca (la columna es `DECIMAL(10,2)`).

#### TC-CAT-098 — Longitudes máximas de código y nombre del medicamento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | HU-CAT-018 «Longitudes máximas»; CU-CAT-18 excepción paso 5; [RN-MED-003][rn-b] |

**Datos:** 51 caracteres en «Código Nacional»; 151 en «Nombre Comercial».
**Resultado esperado:** el código no deja pasar de 50 caracteres y el nombre de 150, sin mensaje. Con solo espacios: **Hallazgo (brecha 1):** «Error al guardar el medicamento».

#### TC-CAT-099 — Código nacional repetido y nombre comercial repetido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-018 «Código nacional repetido»; «El nombre comercial puede repetirse»; CU-CAT-18 excepción paso 7; [RN-MED-010][rn-b] |

**Precondiciones:** existe `DEMO-MED-0001` «Paracetamol Demo 500 mg».
**Pasos:** 1. Registra otro medicamento con el código `demo-med-0001`. 2. Registra otro con el mismo nombre y código `DEMO-MED-0009`.

**Resultado esperado:** (1) «Ya existe un medicamento con ese código nacional.» y te quedas en el formulario; (2) se guarda. **No verificado:** el rechazo con otra capitalización depende de la intercalación.

#### TC-CAT-100 — Laboratorio o unidad eliminados mientras se registra el medicamento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-CAT-018 «Laboratorio o unidad que ya no existen»; CU-CAT-18 excepción paso 6; [RN-MED-005][rn-b], [RN-MED-006][rn-b] |

**Pasos:** 1. Sesión 1: llena el formulario con un laboratorio de prueba y no guardes. 2. Sesión 2: elimina ese laboratorio (o la unidad). 3. Sesión 1: guarda.

**Resultado esperado:** 404 `Laboratorio.NotFound` o `UnidadMedida.NotFound` y «Error al guardar el medicamento».

#### TC-CAT-101 — Editar un medicamento y medicamento antiguo sin precio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-018 «Editar un medicamento»; «Medicamento antiguo sin precio»; CU-CAT-18 A1 y A2; [RN-MED-005][rn-b], [RN-MED-009][rn-b], [RN-MED-011][rn-b] |

**Precondiciones:** `Medicamentos:Actualizar`; un medicamento cargado por carga masiva (sin precio) y `DEMO-MED-0001`.
**Pasos:** 1. Abre «Paracetamol Demo 500 mg», «Editar», cambia el precio a `0.60` y guarda. 2. Abre el medicamento sin precio, «Editar» y guarda sin tocar el precio. 3. Informa el precio y guarda.

**Resultado esperado:** (1) «Medicamento guardado correctamente», te quedas en el detalle y los datos se recargan. (2) El campo «Precio de Venta Base» aparece vacío y no deja guardar hasta informarlo («El precio no puede ser negativo»). (3) Se guarda.

#### TC-CAT-102 — El medicamento ya no existe y el id al editar (solo API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Baja | HU-CAT-018 «El medicamento ya no existe»; «Identificador al editar (solo API)»; CU-CAT-18 A1; [RN-MED-011][rn-b], [RN-MED-012][rn-b] |

**Pasos:** 1. Sesión 1 edita un medicamento; sesión 2 lo elimina; sesión 1 guarda y luego abre su enlace. 2. API: `PUT /api/v1/Medicamentos/00000000-0000-0000-0000-000000000000` con cuerpo válido.

**Resultado esperado:** 404 `Medicamento.NotFound`: «Error al guardar el medicamento» y «Error al cargar el medicamento». El segundo: 400 `UpdateMedicamentoCommand.Id`.

### HU-CAT-019 — Buscar y exportar medicamentos

#### TC-CAT-103 — Filtros disponibles y búsqueda sin filtros de medicamentos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-019 «Filtros disponibles»; «Buscar sin filtros»; CU-CAT-19 flujo básico; [RN-MED-014][rn-b], [RN-MED-019][rn-b] |

**Precondiciones:** existen los cuatro medicamentos del manual.
**Pasos:** 1. Abre `/medicamentos`. 2. «Buscar».

**Resultado esperado:** filtros «Código Nacional», «Nombre Comercial», «Laboratorio» («Buscar laboratorio...»), «Unidad Base» («Buscar unidad de medida...») y «Condición de Venta» («— Todos —», «Venta Libre (OTC)», «Receta Simple», «Receta Retenida», «Estupefaciente»). «Resultados (4)» ordenados por nombre comercial: Amoxicilina, Clonazepam, Paracetamol y Tramadol; columnas «Código Nacional» (enlace al detalle), «Nombre Comercial», «Laboratorio», «Unidad Base» y «Condición de Venta»; sin precio ni stock.

#### TC-CAT-104 — Filtros combinados de medicamentos y condición de venta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-019 «Filtros que se combinan»; CU-CAT-19 flujo básico paso 3; [RN-MED-014][rn-b] |

**Precondiciones:** «Laboratorio Demo Andino» tiene Clonazepam y Paracetamol; «Laboratorio Demo Pacífico» tiene Amoxicilina y Tramadol.
**Datos y resultado esperado:**

| Filtro | Resultado |
|---|---|
| «Laboratorio» `andino` (sin elegir sugerencia) | Clonazepam y Paracetamol |
| «Código Nacional» `0001` | Solo Paracetamol |
| «Nombre Comercial» `DEMO 500` | Amoxicilina y Paracetamol |
| «Condición de Venta» «Receta Retenida» | Solo Clonazepam |
| «Laboratorio» `pac` y «Condición de Venta» «Estupefaciente» | Solo Tramadol |

**Pasos:** 1. Aplica cada combinación y busca.

**Resultado esperado:** el de la tabla (los nombres comerciales de los medicamentos de prueba son los que dicen los datos de cada fila).

#### TC-CAT-105 — Sin coincidencias y medicamentos con laboratorio o unidad eliminados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-019 «Sin coincidencias»; «Quedan fuera los medicamentos cuyo laboratorio o unidad fueron eliminados»; CU-CAT-19 excepción paso 3; [RN-MED-014][rn-b] |

**Precondiciones:** un medicamento vivo cuyo laboratorio (o unidad base) está marcado como eliminado; solo se logra preparando los datos por base de datos (ver el resultado esperado).
**Pasos:** 1. Busca el código `zzz`. 2. Busca sin filtros.

**Resultado esperado:** (1) «Resultados (0)» y la fila «Sin resultados». (2) El medicamento cuyo laboratorio o unidad base fue eliminado no aparece en la lista. **No verificado:** cómo se llega a ese estado desde la aplicación, porque eliminar un laboratorio o una unidad en uso por un medicamento está bloqueado (`TC-CAT-022`, `TC-CAT-074`); el caso solo se puede ejecutar preparando los datos por base de datos.

#### TC-CAT-106 — Exportar medicamentos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-019 «Exportar»; «Falla la carga de condiciones de venta»; CU-CAT-19 A1 y excepción paso 1; [RN-MED-020][rn-b], [RN-IMP-008][rn-a], [RN-IMP-009][rn-a] |

**Pasos:** 1. Busca con «Condición de Venta» «Receta Simple». 2. «Exportar».

**Resultado esperado:** se descarga `medicamentos.xlsx` con la hoja «Data» y las columnas «Código Nacional», «Nombre Comercial», «Laboratorio», «Unidad Base» y «Condición de Venta» (en texto, por ejemplo «Receta Simple»), con los filtros escritos y sin precio ni stock. Si falla la búsqueda o la exportación: «Error al buscar medicamentos». Si la lista de condiciones no cargó, la búsqueda no avisa: el select queda solo con «— Todos —» y la columna de condición mostraría el número. **No verificado en la aplicación:** el contenido del archivo y este último caso.

### HU-CAT-020 — Consultar el detalle de un medicamento y su stock por sucursal

#### TC-CAT-107 — Ver el stock total y por sucursal en el detalle

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-020 «Ver el stock total y por sucursal»; CU-CAT-20 flujo básico; [RN-MED-018][rn-b], [RN-STK-017][rn-c] |

**Precondiciones:** `DEMO-MED-0001` con un lote en «Sede Principal» (cantidad 30) y otro en «Sucursal Demo Norte» (cantidad 12), creados con `TC-INV-025` y `TC-INV-026`.
**Pasos:** 1. Abre el medicamento en modo ver.

**Resultado esperado:** tarjeta «Stock por Sucursal» con «Stock Total: 42» y una tabla «Sucursal» y «Stock» (a la derecha) con 30 y 12. **No verificado:** si el cálculo excluye lotes vencidos o sucursales no permitidas del usuario (la regla solo dice «suma de la cantidad física»).

#### TC-CAT-108 — Medicamento sin stock y tarjeta oculta al crear o editar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-020 «Medicamento sin stock»; «La tarjeta no aparece al crear ni al editar»; CU-CAT-20 A1 y A2; [RN-MED-018][rn-b] |

**Pasos:** 1. Abre un medicamento sin lotes con existencias. 2. Haz clic en «Editar». 3. Abre «Nuevo».

**Resultado esperado:** (1) «Stock Total: 0» y «Sin stock registrado». (2) y (3) la tarjeta «Stock por Sucursal» no se muestra.

#### TC-CAT-109 — Abrir un medicamento que no existe

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-CAT-020 «El medicamento no existe»; CU-CAT-20 excepción paso 2; [RN-MED-012][rn-b] |

**Pasos:** 1. Escribe `/medicamentos/00000000-0000-0000-0000-000000000001`.

**Resultado esperado:** 404 `Medicamento.NotFound` y «Error al cargar el medicamento» con la ficha vacía y deshabilitada (con `Actualizar` sigue visible «Editar»).

### HU-CAT-021 — Eliminar un medicamento

#### TC-CAT-110 — Eliminar un medicamento sin dependencias

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-021 «Eliminar un medicamento sin dependencias»; CU-CAT-21 flujo básico y A1; [RN-MED-012][rn-b], [RN-MED-013][rn-b] |

**Precondiciones:** un medicamento «Medicamento Prueba Borrar» sin registros asociados; `Medicamentos:Eliminar`.
**Pasos:** 1. «Editar», «Eliminar»; cancela; repite y «Confirmar».

**Resultado esperado:** «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este medicamento?»; al confirmar «Medicamento eliminado correctamente» y vuelves a `/medicamentos`; la API responde 204.

#### TC-CAT-111 — Bloqueo al eliminar un medicamento con registros asociados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-021 «Medicamento con registros asociados»; CU-CAT-21 excepción paso 4 (`Medicamento.InUse`); [RN-MED-013][rn-b] |

**Precondiciones:** ocho medicamentos de prueba, cada uno con una sola dependencia no eliminada.
**Datos:**

| Medicamento | Dependencia |
|---|---|
| M1 | un lote |
| M2 | una presentación de venta |
| M3 | una jerarquía de unidades |
| M4 | una formulación clínica |
| M5 | un precio de proveedor |
| M6 | un servicio de agenda (sin pantalla) |
| M7 | una inmunización (sin pantalla) |
| M8 | una orden magistral (sin pantalla) |

**Pasos:** 1. Intenta eliminar cada uno.

**Resultado esperado:** para M1 a M5, la API responde 409 `Medicamento.InUse` y ves «No puedes eliminar este medicamento porque tiene lotes, presentaciones u otros registros asociados.»; el medicamento sigue existiendo. M6 a M8 se preparan por API o base de datos: **No verificado en la aplicación** (el manual dejó el bloqueo fuera de su alcance y esos módulos no tienen pantalla). **Hallazgo:** el texto no dice cuál de las ocho causas aplica y se muestra ante cualquier 409.

#### TC-CAT-112 — Hallazgo: las ventas no bloquean la eliminación de un medicamento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-021 «Las ventas no bloquean la eliminación»; CU-CAT-21 flujo básico paso 4; [RN-MED-013][rn-b] |

**Precondiciones:** un medicamento que aparece solo en ventas ya registradas (sin lotes, presentaciones ni otras dependencias).
**Pasos:** 1. Elimínalo.

**Resultado esperado:** **Hallazgo:** la API no considera las ventas una dependencia y lo elimina («Medicamento eliminado correctamente»). **No verificado en la aplicación.**

#### TC-CAT-113 — Otro fallo al eliminar un medicamento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Baja | HU-CAT-021 «Otro fallo al eliminar»; CU-CAT-21 excepción paso 4 (`Medicamento.NotFound`); [RN-MED-012][rn-b] |

**Pasos:** 1. Sesión 1: «Editar», «Eliminar». 2. Sesión 2: elimina el medicamento. 3. Sesión 1: «Confirmar».

**Resultado esperado:** 404 `Medicamento.NotFound` y «Error al eliminar el medicamento».

### HU-CAT-022 — Cargar medicamentos desde un Excel

#### TC-CAT-114 — Columnas de la plantilla de medicamentos, sin precio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-022 «Columnas de la plantilla»; 12b §1.3 «CM-1 Descargar la plantilla»; CU-CAT-22 flujo básico paso 1; [RN-MED-021][rn-b], [RN-MED-025][rn-b] |

**Pasos:** 1. Descarga la plantilla desde `/medicamentos/carga-masiva`.

**Resultado esperado:** `plantilla-medicamentos.xlsx` con «CodigoNacional» (50 caracteres), «NombreComercial» (150), «Laboratorio» (150), «UnidadBase» (50) y «CondicionVenta» (50), todas obligatorias, y **sin columna de precio**. Los escenarios CM-1 a CM-5 se prueban como en `TC-CAT-025` a `TC-CAT-033`.

#### TC-CAT-115 — Laboratorio, unidad y condición se escriben como texto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-022 «El laboratorio, la unidad y la condición se escriben como texto»; CU-CAT-22 excepción paso 2 (`not-found`, `ambiguous`, `invalid-option`); [RN-MED-022][rn-b], [RN-MED-023][rn-b] |

**Precondiciones:** dos unidades con la misma descripción (para el caso de coincidencias múltiples).
**Datos:** fila 1: `DEMO-MED-0010` / `Medicamento Texto Uno` / `laboratorio demo andino` / `unidad` / `receta simple`; fila 2: laboratorio `Laboratorio Inexistente`; fila 3: unidad con descripción duplicada; fila 4: condición `Controlado`.
**Pasos:** 1. Sube el archivo.

**Resultado esperado:** fila 1 válida (laboratorio por nombre de compañía y unidad por descripción, sin distinguir mayúsculas; la condición coincide con «Receta Simple» sin distinguir mayúsculas); fila 2: «El valor 'Laboratorio Inexistente' del campo 'laboratorio' no existe en el sistema»; fila 3: «El valor '<valor>' del campo 'unidadBase' coincide con varios registros del sistema»; fila 4: «'Controlado' no es un valor válido para 'condicionVenta'». Las condiciones válidas son «Venta Libre (OTC)», «Receta Simple», «Receta Retenida» y «Estupefaciente».

#### TC-CAT-116 — Código nacional repetido o existente en la carga

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-022 «Código nacional repetido o existente»; CU-CAT-22 excepción paso 2 (`duplicate`, `duplicate-in-file`); [RN-MED-010][rn-b], [RN-MED-024][rn-b] |

**Precondiciones:** existe `DEMO-MED-0001`.
**Datos:** fila A `DEMO-MED-0001`; filas B y C con `DEMO-MED-0020` (mayúsculas distintas en C); fila D válida `DEMO-MED-0021`.
**Pasos:** 1. Sube el archivo. 2. Guarda las válidas.

**Resultado esperado:** A: «El valor 'DEMO-MED-0001' del campo 'codigoNacional' ya existe en el sistema»; B y C: «…está duplicado en el archivo» (las dos filas); D válida. Al guardar solo se guarda D (de las repetidas del archivo, si llegaran al servidor, solo la primera).

#### TC-CAT-117 — Hallazgo: los medicamentos cargados en masa quedan sin precio de venta base

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-022 «Los medicamentos cargados quedan sin precio»; CU-CAT-22 flujo básico paso 3; [RN-MED-021][rn-b] |

**Pasos:** 1. Carga una fila válida `DEMO-MED-0021`. 2. Búscala y ábrela; haz clic en «Editar».

**Resultado esperado:** **Hallazgo (pendiente 6 de 12b 19.2):** el medicamento nace con el precio de venta base vacío; el campo aparece vacío y hay que informarlo para guardar. Para venderlo hay que asignarle precio o una presentación con precio (parte C, RN-VEN-019).

#### TC-CAT-118 — Solo API: carga de medicamentos con filas inválidas o fuera de rango

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-022 «Columnas de la plantilla»; CU-CAT-22 excepción paso 3 (validador y lista de filas); [RN-MED-025][rn-b], [RN-MED-026][rn-b] |

**Datos:** `POST /api/v1/Medicamentos/carga-masiva` con lista vacía; con 5001 filas; con una fila de `CodigoNacional` de 51 caracteres. `POST /api/v1/Medicamentos/previsualizar` sin archivo.
**Resultado esperado:** 400 `CargaMasivaMedicamentosCommand.Rows`; 400 `CargaMasivaMedicamentosCommand.Rows[0].CodigoNacional`; 400 «Debe adjuntar un archivo Excel.»

### HU-CAT-023 — Ofrecer medicamentos al punto de venta con stock y los más vendidos

#### TC-CAT-119 — Solo API: incluir el stock en el listado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-023 «Incluir el stock en el listado»; CU-CAT-23 flujo básico; [RN-MED-015][rn-b], [RN-STK-017][rn-c] |

**Precondiciones:** `DEMO-MED-0001` con lotes en dos sucursales (30 y 12); token de `Admin`.
**Datos:** `GET /api/v1/Medicamentos?IncluirStock=true`.
**Resultado esperado:** 200; el stock total de `DEMO-MED-0001` es 42, la suma de la cantidad física de sus lotes en todas las sucursales de la empresa. **No verificado en la aplicación:** el nombre exacto de la propiedad del stock en la respuesta.

#### TC-CAT-120 — Solo API: solo stock vendible en mi sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-023 «Solo stock vendible en mi sucursal»; CU-CAT-23 flujo básico; [RN-MED-015][rn-b], [RN-STK-017][rn-c] |

**Precondiciones:** token de un usuario de «Sede Principal». Medicamentos: V1 con un lote vigente (cantidad 10) en la sucursal; V2 solo con un lote vencido; V3 solo con un lote con retención activa en cuarentena; V4 con cantidad 0; V5 con stock solo en «Sucursal Demo Norte».
**Datos:** `GET /api/v1/Medicamentos?IncluirStock=true&SoloStockVendibleEnMiSucursal=true`.
**Resultado esperado:** solo cuenta el stock vendible de la sucursal del usuario (cantidad mayor que 0, lote no vencido y sin retención activa): V1 aparece con 10; V2 a V5 con 0 (o fuera de lo vendible). **No verificado:** si los medicamentos sin stock vendible se excluyen de la lista o quedan con stock 0; el documento fuente solo dice que no cuentan.

#### TC-CAT-121 — Solo API: más vendidos primero

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-023 «Más vendidos primero»; CU-CAT-23 flujo básico; [RN-MED-016][rn-b] |

**Precondiciones:** en la sucursal del usuario, en los últimos 30 días: A vendió 5 unidades, B vendió 9, C vendió 9 (venta `Completada`) y D tiene una venta anulada de 20 unidades; E nunca se vendió.
**Datos:** `GET /api/v1/Medicamentos?OrdenarPorMasVendidos=true`.
**Resultado esperado:** orden: B y C (empate: por nombre comercial), A, y después los no vendidos (D y E, por nombre comercial); la venta anulada no cuenta.

#### TC-CAT-122 — Solo API: sin sucursal en la sesión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-023 «Sin sucursal en la sesión»; CU-CAT-23 excepción paso 1; [RN-MED-017][rn-b] |

**Precondiciones:** un token sin sucursal en la sesión. **No verificado:** cómo obtener ese token en el entorno local (todos los usuarios tienen sucursal base); si no es posible, queda «No ejecutable».
**Datos:** `GET /api/v1/Medicamentos?OrdenarPorMasVendidos=true`.
**Resultado esperado:** 400 `Auth.Sucursal`.

## 9. Presentaciones de venta

Una presentación define, para un producto, la unidad en que se vende (por ejemplo «Caja»), cuántas unidades base contiene y su precio propio. No hay carga masiva. Los casos parten del producto «Paracetamol Demo 500 mg» con unidad base «Unidad» y las conversiones «Caja a Blíster = 10» y «Blíster a Unidad = 10». El manual no recorrió esta pantalla; los mensajes salen de 12b y de [19d sección 2][p19d].

### HU-CAT-024 — Registrar y editar una presentación de venta

#### TC-CAT-123 — Registrar una presentación de venta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-024 «Registrar una presentación»; CU-CAT-24 flujo básico; [RN-PRE-001 a 009][rn-b], [RN-PRE-015][rn-b], [RN-JER-011][rn-b] |

**Precondiciones:** `PresentacionesVenta:Crear`, `Medicamentos:Leer` y `JerarquiasUoM:Leer`; el producto tiene las dos conversiones.
**Datos:** Unidad de Venta «Caja (100 Unidad)»; Precio de Venta `40.00`.
**Pasos:**
1. Abre `/presentaciones-venta`, haz clic en «Nuevo» y elige el producto de la lista «Buscar producto...».
2. Despliega «Unidad de Venta».
3. Elige «Caja (100 Unidad)», escribe el precio y haz clic en «Guardar».

**Resultado esperado:** al elegir el producto aparecen «Unidad de Venta» y «Precio de Venta»; «Unidad de Venta» ofrece «Caja (100 Unidad)» y «Blíster (10 Unidad)», sin la unidad base. Al guardar ves «Presentación de venta guardada correctamente» y vuelves a `/presentaciones-venta`.

#### TC-CAT-124 — La equivalencia y la descripción las calcula el servidor

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-024 «La equivalencia y la descripción las calcula el servidor»; CU-CAT-24 flujo básico paso 7; [RN-PRE-007][rn-b], [RN-PRE-008][rn-b] |

**Precondiciones:** la presentación «Caja» de `TC-CAT-123`.
**Pasos:** 1. Mira la petición `POST /api/v1/PresentacionesVenta` en la pestaña de red. 2. Busca la presentación en `/presentaciones-venta`.

**Resultado esperado:** el cuerpo lleva solo producto, unidad y precio (el SPA no envía la equivalencia). El servidor guarda la equivalencia 100 (con 4 decimales) y la descripción es la de la unidad elegida («Caja»); la columna «Equivalencia» muestra 100.

#### TC-CAT-125 — Producto sin jerarquía al registrar una presentación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-024 «Producto sin jerarquía»; CU-CAT-24 A2; [RN-PRE-015][rn-b] |

**Precondiciones:** un producto sin conversiones; dos sesiones: una con `JerarquiasUoM:Crear` y otra sin él.
**Pasos:** 1. «Nuevo» y elige ese producto. 2. Con `JerarquiasUoM:Crear`, haz clic en «Ir a Jerarquías de Unidades». 3. Con el rol sin ese permiso, repite el paso 1.

**Resultado esperado:** ves «Este producto no tiene una Jerarquía de Unidades registrada. Cárgala primero para poder definir presentaciones de venta.»; los campos de unidad y precio no se dibujan; el botón «Ir a Jerarquías de Unidades» abre `/jerarquias/nueva` y solo existe con el permiso. **Hallazgo (brecha 3):** si pulsas «Guardar», no se envía nada y no hay error visible (por lectura del código; no se ejecutó).

#### TC-CAT-126 — Solo API: conversión que no se puede resolver

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-024 «La conversión no se puede resolver (solo API)»; CU-CAT-24 excepción paso 7; [RN-PRE-006][rn-b] |

**Precondiciones:** token de `Admin`; un producto con unidad base «Unidad» y una unidad `FRA` sin camino hasta la base.
**Datos:** `POST /api/v1/PresentacionesVenta` con ese producto, la unidad `FRA` y precio 10.
**Resultado esperado:** 400 `PresentacionVenta.Jerarquia.NoResuelta`. La pantalla lo previene (solo ofrece unidades vendibles); si llegara, ves «Error al guardar la presentación de venta». **Hallazgo (brecha 4):** pedir la propia unidad base como unidad de venta no se rechaza (el cálculo devuelve 1); la pantalla no ofrece ese caso.

#### TC-CAT-127 — Producto y unidad obligatorios; precio negativo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-024 «Producto y unidad obligatorios»; «Precio negativo»; CU-CAT-24 excepción paso 4; [RN-PRE-002][rn-b], [RN-PRE-004][rn-b], [RN-PRE-009][rn-b] |

**Pasos:** 1. Haz clic en «Guardar» sin elegir producto. 2. Elige producto, deja la unidad sin elegir y guarda. 3. Elige «Caja (100 Unidad)» y escribe `-5` en «Precio de Venta»; guarda. 4. Deja el precio vacío.

**Resultado esperado:** (1) «Este campo es obligatorio» debajo del producto; (2) «Debes seleccionar una unidad de venta»; (3) y (4) «El precio de venta no puede ser negativo» (también con el campo vacío); no se envía nada. El precio es propio de la presentación: no se calcula desde el precio base del medicamento.

#### TC-CAT-128 — Una sola presentación por unidad y producto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-024 «Una sola presentación por unidad y producto»; CU-CAT-24 excepción paso 6; [RN-PRE-005][rn-b] |

**Precondiciones:** el producto ya tiene una presentación «Caja».
**Pasos:** 1. Intenta crear otra con «Caja (100 Unidad)».

**Resultado esperado:** 409 `PresentacionVenta.UnidadVenta.Duplicada` y «Ya existe una presentación de venta para esa unidad en este producto.» (la pantalla muestra ese texto ante cualquier 409: brecha 1).

#### TC-CAT-129 — El producto no se puede cambiar al editar; editar la presentación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-024 «El producto no se puede cambiar»; «Editar una presentación»; CU-CAT-24 A1; [RN-PRE-003][rn-b], [RN-PRE-005][rn-b], [RN-PRE-007][rn-b] |

**Precondiciones:** `PresentacionesVenta:Actualizar`; la presentación «Caja» a S/ 40.00.
**Pasos:** 1. Ábrela y haz clic en «Editar». 2. Cambia el precio a `45.00` y guarda.

**Resultado esperado:** el producto queda deshabilitado con la ayuda «El producto de una presentación no se puede modificar luego de creada»; se pueden cambiar la unidad de venta y el precio. Al guardar ves «Presentación de venta guardada correctamente» y vuelves a `/presentaciones-venta`; la API vuelve a calcular la equivalencia y a comprobar que no se duplique la unidad.

#### TC-CAT-130 — Producto o unidad eliminados; la presentación ya no existe

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-CAT-024 «Producto o unidad que ya no existen»; «La presentación ya no existe»; CU-CAT-24 excepción paso 5 y A1; [RN-PRE-002][rn-b], [RN-PRE-004][rn-b], [RN-PRE-011][rn-b] |

**Pasos:** 1. Sesión 1: llena el formulario con una unidad de prueba; sesión 2 la elimina; sesión 1 guarda. 2. Sesión 1: edita una presentación; sesión 2 la elimina; sesión 1 guarda y luego abre su enlace.

**Resultado esperado:** (1) 404 `PresentacionVenta.Producto` o `PresentacionVenta.UnidadVenta` y «Error al guardar la presentación de venta». (2) 404 `PresentacionVenta.NotFound`: «Error al guardar la presentación de venta» y «Error al cargar la presentación de venta».

#### TC-CAT-131 — La equivalencia guardada no se recalcula si cambian las jerarquías

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-024 «La equivalencia guardada no se recalcula»; CU-CAT-24 flujo básico paso 7; [RN-PRE-007][rn-b] |

**Precondiciones:** la presentación «Caja» con equivalencia 100.
**Pasos:** 1. Edita la conversión «Caja a Blíster» y cámbiala a 12 (la coherencia con «Blíster a Unidad» y «Caja a Unidad» debe permitirlo: si no, elimina la conversión). 2. Mira la presentación en `/presentaciones-venta`. 3. Edítala y guarda sin cambios.

**Resultado esperado:** tras el paso 1 la presentación conserva la equivalencia 100; tras guardarla de nuevo (paso 3) la API la recalcula con las jerarquías vigentes. **No verificado:** lo que la API calcula si las jerarquías cambiadas hacen imposible la conversión.

#### TC-CAT-132 — Solo API: identificador al editar una presentación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-CAT-024 «Identificador al editar (solo API)»; CU-CAT-24 A1 (excepción `UpdatePresentacionVentaCommand.Id`); [RN-PRE-010][rn-b] |

**Datos:** `PUT /api/v1/PresentacionesVenta/00000000-0000-0000-0000-000000000000` con cuerpo válido.
**Resultado esperado:** 400 `UpdatePresentacionVentaCommand.Id`.

### HU-CAT-025 — Administrar las presentaciones desde la ficha de un medicamento

#### TC-CAT-133 — Abrir el diálogo y estados vacíos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-025 «Abrir el diálogo»; «Producto sin presentaciones»; «Producto sin jerarquía»; «Falla la carga»; CU-CAT-25 flujo básico, A4 y A5; [RN-PRE-001][rn-b], [RN-PRE-015][rn-b] |

**Precondiciones:** `JerarquiasUoM:Leer` y `PresentacionesVenta:Leer`; tres medicamentos: M1 con unidades vendibles y sin presentaciones, M2 sin jerarquías, M3 con presentaciones.
**Pasos:** 1. Abre cada medicamento en modo ver y haz clic en «Presentaciones de Venta».

**Resultado esperado:** se abre el diálogo «Presentaciones de Venta» con el nombre comercial debajo del título y el botón «Cerrar». M1: «Este producto todavía no tiene presentaciones de venta.» M2: «Este producto no tiene una Jerarquía de Unidades registrada. Cárgala primero para poder definir presentaciones de venta.» y, con `JerarquiasUoM:Crear`, «Ir a Jerarquías de Unidades» (cierra el diálogo y abre `/jerarquias/nueva`). M3: la tabla de presentaciones. Si falla la consulta: «Error al cargar las presentaciones de venta» (**No verificado en la aplicación**).

#### TC-CAT-134 — Agregar una presentación desde el diálogo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-025 «Agregar una presentación»; CU-CAT-25 flujo básico; [RN-PRE-005][rn-b], [RN-PRE-009][rn-b] |

**Precondiciones:** `PresentacionesVenta:Crear`; M1 de `TC-CAT-133`.
**Pasos:** 1. Haz clic en «Agregar presentación». 2. Elige «Caja (100 Unidad)», escribe `40.00` y haz clic en «Guardar».

**Resultado esperado:** «Presentación de venta guardada correctamente»; la lista se recarga, el formulario en línea se cierra y la tabla muestra «Unidad de Venta», «Equivalencia» («100 Unidad»), «Precio de Venta» («S/ 40.00») y los botones de editar y eliminar según tus permisos.

#### TC-CAT-135 — Solo las unidades sin presentación; todas cubiertas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-025 «Solo las unidades sin presentación»; CU-CAT-25 A3; [RN-PRE-015][rn-b] |

**Precondiciones:** el producto ya tiene presentación para «Caja».
**Pasos:** 1. Abre «Agregar presentación» y despliega «Unidad de Venta». 2. Edita la presentación «Caja». 3. Crea la de «Blíster» y vuelve a mirar «Agregar presentación».

**Resultado esperado:** (1) no se ofrece «Caja»; (2) al editar sigue disponible la propia; (3) con una presentación por cada unidad, «Agregar presentación» queda deshabilitado con el aviso «Ya existe una presentación para cada unidad definida en la Jerarquía de este producto.»

#### TC-CAT-136 — Campos obligatorios, precio negativo y unidad duplicada en el diálogo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-025 «Campos obligatorios y precio negativo»; «Unidad duplicada»; CU-CAT-25 excepciones pasos 4 y 5; [RN-PRE-005][rn-b], [RN-PRE-009][rn-b] |

**Pasos:** 1. Guarda con la unidad vacía. 2. Elige unidad y escribe `-1`. 3. Con dos sesiones abiertas, crea la misma unidad en ambas y guarda la segunda.

**Resultado esperado:** (1) «Este campo es obligatorio»; (2) «El precio de venta no puede ser negativo»; (3) 409 y «Ya existe una presentación de venta para esa unidad.»; cualquier otro error: «Error al guardar la presentación de venta».

#### TC-CAT-137 — Eliminar una presentación sin ventas y con ventas desde el diálogo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-025 «Eliminar una presentación sin ventas»; «Eliminar una presentación con ventas»; CU-CAT-25 A2; [RN-PRE-012][rn-b] |

**Precondiciones:** `PresentacionesVenta:Eliminar`; una presentación sin ventas (P1) y una presentación usada en una venta no eliminada (P2, creada con una venta de la parte C).
**Pasos:** 1. Haz clic en la papelera de P1 y confirma «¿Estás seguro de que deseas eliminar esta presentación de venta?». 2. Repite con P2.

**Resultado esperado:** P1: «Presentación de venta eliminada correctamente». P2: 409 `PresentacionVenta.InUse` y «No se puede eliminar la presentación porque tiene ventas registradas.»; otro error: «Error al eliminar la presentación de venta». **No verificado en la aplicación:** el diálogo no se recorrió.

### HU-CAT-026 — Buscar y exportar presentaciones de venta

#### TC-CAT-138 — Buscar presentaciones y tabla de resultados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-026 «Buscar presentaciones»; «Producto elegido de la lista o escrito»; «Sin coincidencias»; CU-CAT-26 flujo básico; [RN-PRE-014][rn-b] |

**Precondiciones:** presentaciones de dos productos; una presentación cuyo producto fue eliminado.
**Pasos:** 1. Abre `/presentaciones-venta` y busca sin filtros. 2. Elige un producto de la lista de sugerencias y busca. 3. Escribe solo parte del nombre (`Parac`) sin elegir y busca. 4. Busca con un filtro sin coincidencias.

**Resultado esperado:** filtros «Producto» («Buscar producto...») y «Unidad de Venta». La tabla muestra «Producto» (enlace al detalle), «Unidad de Venta», «Equivalencia» (número, sin el nombre de la unidad base) y «Precio de Venta» («S/ » con dos decimales, a la derecha), ordenada por nombre del producto y luego por descripción, sin las presentaciones cuyo producto fue eliminado. (2) filtra por ese producto exacto; (3) por nombre «contiene»; (4) «Sin resultados».

#### TC-CAT-139 — Hallazgo: la exportación ignora el producto elegido de la lista

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-026 «Exportar»; CU-CAT-26 A1; [RN-PRE-014][rn-b], [RN-IMP-008][rn-a] |

**Precondiciones:** presentaciones de dos productos distintos.
**Pasos:** 1. Elige un producto de la lista y busca. 2. Haz clic en «Exportar». 3. Borra el texto del filtro «Producto» y busca de nuevo.

**Resultado esperado:** se descarga `presentaciones-venta.xlsx` con «Producto», «Unidad de Venta», «Equivalencia» y «Precio de Venta». **Hallazgo (brecha 1 de HU-CAT-026, pendiente 13):** el Excel sale sin filtrar por el producto elegido (solo por unidad de venta). **Hallazgo (brecha 2):** al borrar el texto, el producto elegido antes no se descarta y «Buscar» sigue filtrando por él. Ambos salen de leer el código; no se ejecutaron. Si falla: «Error al buscar presentaciones de venta».

### HU-CAT-027 — Eliminar una presentación de venta

#### TC-CAT-140 — Eliminar una presentación sin ventas desde su pantalla

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-027 «Eliminar una presentación sin ventas»; CU-CAT-27 flujo básico y A1; [RN-PRE-011][rn-b], [RN-PRE-013][rn-b] |

**Pasos:** 1. Abre la presentación, «Editar», «Eliminar»; cancela; repite y «Confirmar».

**Resultado esperado:** «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar esta presentación de venta?»; al confirmar «Presentación de venta eliminada correctamente» y vuelves a `/presentaciones-venta`; la API responde 204.

#### TC-CAT-141 — Bloqueo al eliminar una presentación con ventas registradas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-027 «Presentación con ventas registradas»; «Otro fallo»; CU-CAT-27 excepción paso 4; [RN-PRE-012][rn-b] |

**Precondiciones:** una presentación que figura en un detalle de venta no eliminado.
**Pasos:** 1. Elimínala y confirma. 2. En otra sesión, elimina una presentación sin ventas y desde la primera confirma su eliminación.

**Resultado esperado:** (1) 409 `PresentacionVenta.InUse` y «No se puede eliminar la presentación porque tiene ventas registradas.»; sigue existiendo. (2) 404 `PresentacionVenta.NotFound` y «Error al eliminar la presentación de venta».

## 10. Formulaciones clínicas

Una formulación indica qué ingrediente activo contiene un medicamento, en qué concentración y unidad, con un código TE Orange opcional. No hay carga masiva.

### HU-CAT-028 — Registrar y editar una formulación clínica

#### TC-CAT-142 — Registrar una formulación clínica

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-028 «Registrar una formulación»; CU-CAT-28 flujo básico; [RN-FOR-001 a 008][rn-b] |

**Precondiciones:** `FormulacionesClinicas:Crear`, `Medicamentos:Leer`, `IngredientesActivos:Leer`, `UnidadesMedida:Leer`.
**Datos:** medicamento «Paracetamol Demo 500 mg»; ingrediente «Paracetamol»; Concentración de Dosis `500`; unidad «Miligramo».
**Pasos:** 1. Abre `/formulaciones-clinicas`, «Nuevo». 2. Elige el medicamento («Buscar medicamento...»), el ingrediente («Buscar ingrediente activo...») y la unidad («Buscar unidad de medida...»); escribe 500. 3. «Guardar».

**Resultado esperado:** «Formulación clínica guardada correctamente» y vuelves a `/formulaciones-clinicas`.

#### TC-CAT-143 — Elegir de las listas y concentración inválida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-028 «Hay que elegir de las listas»; «Concentración inválida»; CU-CAT-28 excepción paso 3; [RN-FOR-002][rn-b], [RN-FOR-004][rn-b], [RN-FOR-006][rn-b], [RN-FOR-007][rn-b] |

**Datos:** (a) medicamento, ingrediente o unidad sin elegir (texto escrito sin elegir opción); (b) concentración `0`, `0,00009` y vacía; (c) concentración `0,0001`.
**Pasos:** 1. Guarda con (a). 2. Con todo elegido, guarda con cada valor de (b) y (c).

**Resultado esperado:** (a) «Este campo es obligatorio» debajo del campo que falta, nada se envía, y los autocompletados no buscan con el campo vacío; (b) «Este campo es obligatorio» (**Hallazgo, brecha 1:** el texto no describe el problema); (c) se acepta; la concentración se guarda con 4 decimales.

#### TC-CAT-144 — Código TE Orange opcional y de 5 caracteres

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Baja | HU-CAT-028 «Código TE Orange opcional»; CU-CAT-28 excepción paso 4; [RN-FOR-008][rn-b] |

**Pasos:** 1. Escribe `ABCDEFG` en «Código TE Orange». 2. Guarda una formulación sin código.

**Resultado esperado:** el campo se detiene en 5 caracteres sin mensaje; dejarlo vacío es válido y se guarda.

#### TC-CAT-145 — Un ingrediente una sola vez por medicamento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-028 «Un ingrediente una sola vez por medicamento»; CU-CAT-28 excepción paso 5; [RN-FOR-005][rn-b] |

**Precondiciones:** «Paracetamol Demo 500 mg» ya tiene el ingrediente «Paracetamol».
**Pasos:** 1. Registra otra formulación con el mismo medicamento e ingrediente.

**Resultado esperado:** 409 `Formulacion.Ingrediente.Duplicado` y «Ya existe una formulación con los datos ingresados» (**Hallazgo, brecha 2:** la pantalla muestra ese texto ante cualquier 409).

#### TC-CAT-146 — El medicamento no se puede cambiar; editar una formulación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-028 «El medicamento no se puede cambiar»; «Editar una formulación»; CU-CAT-28 A1; [RN-FOR-003][rn-b], [RN-FOR-004][rn-b], [RN-FOR-005][rn-b] |

**Precondiciones:** `FormulacionesClinicas:Actualizar`.
**Pasos:** 1. Abre una formulación y haz clic en «Editar». 2. Cambia la concentración y guarda.

**Resultado esperado:** el medicamento queda deshabilitado y se pueden cambiar ingrediente, concentración, unidad y código. Se guarda con «Formulación clínica guardada correctamente» y vuelves a `/formulaciones-clinicas`.

#### TC-CAT-147 — Medicamento, ingrediente o unidad eliminados; formulación inexistente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-CAT-028 «Medicamento, ingrediente o unidad que ya no existen»; «La formulación ya no existe»; CU-CAT-28 excepción paso 4 y A1; [RN-FOR-010][rn-b] |

**Pasos:** 1. Sesión 1 llena el formulario con un ingrediente de prueba; sesión 2 lo elimina; sesión 1 guarda. 2. Sesión 1 edita una formulación; sesión 2 la elimina; sesión 1 guarda y abre su enlace.

**Resultado esperado:** (1) 404 `IngredienteActivo.NotFound` (o `Medicamento.NotFound` / `UnidadMedida.NotFound`) y «Error al guardar la formulación clínica». (2) 404 `Formulacion.NotFound`: «Error al guardar la formulación clínica» y «Error al cargar la formulación clínica».

#### TC-CAT-148 — Solo API: identificador al editar una formulación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-CAT-028 «Identificador al editar (solo API)»; CU-CAT-28 A1 (excepción `UpdateFormulacionClinicaCommand.Id`); [RN-FOR-009][rn-b] |

**Datos:** `PUT /api/v1/FormulacionesClinicas/00000000-0000-0000-0000-000000000000` con cuerpo válido.
**Resultado esperado:** 400 `UpdateFormulacionClinicaCommand.Id`.

### HU-CAT-029 — Buscar y exportar formulaciones clínicas

#### TC-CAT-149 — Buscar y exportar formulaciones

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-029 «Buscar formulaciones»; «Sin coincidencias»; «Exportar»; CU-CAT-29 flujo básico y A1; [RN-FOR-012][rn-b], [RN-IMP-008][rn-a] |

**Pasos:** 1. Abre `/formulaciones-clinicas`; busca sin filtros. 2. Busca «Medicamento» `PARAC` e «Ingrediente Activo» `para`. 3. Busca `zzz`. 4. Haz clic en «Exportar».

**Resultado esperado:** filtros «Medicamento» e «Ingrediente Activo», ambos con lupa; tabla «Medicamento» (enlace al detalle), «Ingrediente Activo», «Concentración», «Unidad de Medida» y «Código TE Orange», ordenada por nombre del medicamento; ambos filtros «contiene» sin distinguir mayúsculas. (3) «Resultados (0)» y la fila vacía «Resultados». (4) `formulaciones-clinicas.xlsx` con esas cinco columnas; si falla, «Error al buscar formulaciones clínicas», con «Referencia: …» si la respuesta trae referencia de soporte.

### HU-CAT-030 — Eliminar una formulación clínica

#### TC-CAT-150 — Eliminar una formulación sin bloqueo por dependencias

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-030 «Eliminar una formulación»; «No hay bloqueo por dependencias»; CU-CAT-30 flujo básico y A1; [RN-FOR-010][rn-b], [RN-FOR-011][rn-b] |

**Precondiciones:** una formulación del medicamento «Paracetamol Demo 500 mg».
**Pasos:** 1. «Editar», «Eliminar»; cancela; repite y «Confirmar».

**Resultado esperado:** «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar esta formulación clínica?»; al confirmar «Formulación clínica eliminada correctamente» y vuelves a `/formulaciones-clinicas`; la API responde 204. **Hallazgo (pendiente 7):** no existe un 409 de negocio: la API no valida dependencias al eliminar una formulación.

#### TC-CAT-151 — Cualquier error al eliminar una formulación

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Baja | HU-CAT-030 «Cualquier error al eliminar»; CU-CAT-30 excepción paso 4 (`Formulacion.NotFound`) |

**Pasos:** 1. Sesión 1: «Editar», «Eliminar». 2. Sesión 2: elimina la formulación. 3. Sesión 1: «Confirmar».

**Resultado esperado:** 404 `Formulacion.NotFound` y siempre «Error al eliminar la formulación clínica» (la pantalla no distingue el motivo).

### HU-CAT-031 — Consultar las formulaciones de un medicamento (solo API)

#### TC-CAT-152 — Solo API: formulaciones de un medicamento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-031 «Formulaciones de un medicamento»; «Medicamento inexistente»; CU-CAT-31 flujo básico y A1; [RN-FOR-013][rn-b], [RN-FOR-001][rn-b] |

**Precondiciones:** token con `FormulacionesClinicas:Leer`; un medicamento con dos formulaciones no eliminadas.
**Datos:** `GET /api/v1/FormulacionesClinicas/medicamento/<id-medicamento>`; luego con un id inexistente; luego con un token sin ese permiso.
**Resultado esperado:** 200 con las dos formulaciones; 200 con lista vacía para el id inexistente (la consulta no valida la existencia del producto); 403 sin permiso.

## 11. Proveedores y precios de compra

El proveedor es a quien la botica le compra: razón social, identificador tributario («RUC / Tax ID»), términos financieros, calificación ESG y tasa de cumplimiento; guarda además el historial de precios de compra por producto. El manual no recorrió estas pantallas; los mensajes salen de 12b y de [19d sección 4][p19d].

### HU-CAT-032 — Registrar y editar un proveedor

#### TC-CAT-153 — Registrar un proveedor

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-032 «Registrar un proveedor»; «Campos opcionales»; CU-CAT-32 flujo básico; [RN-PRV-001 a 006][rn-b], [RN-PRV-011][rn-b] |

**Precondiciones:** `Proveedores:Crear`.
**Datos:** Razón Social `Distribuidora Demo S.A.C.`; «RUC / Tax ID» `20123456789`; «Calificación ESG» y «Términos Financieros» vacíos.
**Pasos:** 1. En `/proveedores` haz clic en «Nuevo». 2. Mira «Tasa de Cumplimiento». 3. Escribe los datos y «Guardar».

**Resultado esperado:** «Tasa de Cumplimiento» trae el valor inicial 100. Aviso «Proveedor guardado correctamente» y vuelves a `/proveedores`. La API responde 200 con el identificador (no 201: pendiente 17 de 12b 19.2). Los campos «Calificación ESG» y «Términos Financieros» son opcionales.

#### TC-CAT-154 — Campos obligatorios y longitudes del proveedor

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-032 «Campos obligatorios vacíos»; CU-CAT-32 excepciones pasos 3 y 4; [RN-PRV-002][rn-b], [RN-PRV-003][rn-b], [RN-PRV-004][rn-b] |

**Pasos:** 1. Deja vacíos «Razón Social», «RUC / Tax ID» y «Tasa de Cumplimiento» y guarda. 2. Escribe 201 caracteres en «Razón Social» y 31 en «RUC / Tax ID». 3. Escribe 101 caracteres en «Términos Financieros». 4. Prueba solo espacios en razón social.

**Resultado esperado:** (1) «Este campo es obligatorio» debajo de cada uno y no se envía nada. (2) «Razón Social» no deja pasar de 200 y «RUC / Tax ID» de 30 (la API admite 50: **Hallazgo, brecha 1**). (3) **Hallazgo (pendiente 9 de 12b 19.2):** el formulario permite hasta 500 caracteres pero la API admite 100; con 101 a 500 la API responde `ProveedorDistribuidor.TerminosFinancieros` (sin traducción) y ves «Error al guardar el proveedor». (4) **Hallazgo (brecha 2):** «Error al guardar el proveedor».

#### TC-CAT-155 — Tasa de cumplimiento y calificación ESG en rango

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-032 «Tasa de cumplimiento y calificación ESG en rango»; CU-CAT-32 excepción paso 3; [RN-PRV-005][rn-b], [RN-PRV-006][rn-b] |

**Datos:** tasa `-1`, `0`, `100`, `100,01`; calificación ESG `-1`, `0`, `100`, `100,01`, `55,555`.
**Pasos:** 1. Guarda con cada valor.

**Resultado esperado:** una tasa fuera de 0 a 100 muestra «Este campo es obligatorio» (el mismo texto para cualquier error del campo) y no se envía nada; una calificación ESG fuera de 0 a 100 bloquea «Guardar» sin mostrar ningún mensaje; 0 y 100 se aceptan. Tasa y calificación se guardan con 2 decimales (**No verificado:** si `55,555` se redondea o trunca).

#### TC-CAT-156 — Identificador tributario y razón social repetidos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-032 «Identificador tributario repetido»; «Razón social repetida»; «El identificador tributario no se valida como RUC»; CU-CAT-32 excepción paso 5; [RN-PRV-003][rn-b], [RN-PRV-007][rn-b], [RN-PRV-008][rn-b] |

**Precondiciones:** existe «Distribuidora Demo S.A.C.» con RUC `20123456789`.
**Pasos:** 1. Registra otro proveedor con el mismo RUC y otra razón social. 2. Registra otro con la misma razón social y otro RUC. 3. Registra uno con RUC y razón social repetidos a la vez. 4. Registra uno nuevo con el identificador de 8 letras `ABCDEFGH`.

**Resultado esperado:** (1) 409 `Proveedor.TaxId.Duplicado`: «Ya existe un proveedor con ese identificador tributario.» (2) 409 `Proveedor.RazonSocial.Duplicado`: «Ya existe un proveedor con esa razón social.» (3) se muestra el aviso del identificador (se evalúa antes que la razón social). (4) Se guarda: la API solo exige que no esté vacío y que tenga como máximo 50 caracteres (el rótulo dice «RUC» pero no se valida el formato: brecha 3).

#### TC-CAT-157 — Editar un proveedor y proveedor inexistente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-032 «Editar un proveedor»; «El proveedor ya no existe»; CU-CAT-32 A1; [RN-PRV-007][rn-b], [RN-PRV-009][rn-b] |

**Precondiciones:** `Proveedores:Actualizar`; dos sesiones.
**Pasos:** 1. Abre el proveedor, «Editar» (la tarjeta de precios se oculta), cambia la tasa y guarda sin tocar razón social ni RUC. 2. Sesión 1 edita otro proveedor; sesión 2 lo elimina; sesión 1 guarda y abre su enlace.

**Resultado esperado:** (1) «Proveedor guardado correctamente», te quedas en el detalle y no choca consigo mismo. (2) 404 `Proveedor.NotFound`: «Error al guardar el proveedor» y «Error al cargar el proveedor».

### HU-CAT-033 — Buscar y exportar proveedores

#### TC-CAT-158 — Filtros, rango de tasa y tabla de proveedores

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-033 «Filtros»; «Buscar con rango de tasa»; «Tabla»; «Sin coincidencias»; CU-CAT-33 flujo básico; [RN-PRV-012][rn-b] |

**Precondiciones:** proveedores con tasa 70, 80, 100.
**Pasos:** 1. Abre `/proveedores` y mira los filtros. 2. Mueve el deslizador a 80 — 100 y busca. 3. Busca por «Razón Social» inexistente.

**Resultado esperado:** filtros «Razón Social» y «RUC / Tax ID» (texto con lupa, «contiene») y un deslizador «Tasa de Cumplimiento: 0 — 100» con paso 1. (2) Proveedores con tasa entre 80 y 100, ambos inclusive; solo se envía el límite que moviste. Tabla: «Razón Social» (enlace al detalle), «RUC / Tax ID» y «Tasa de Cumplimiento», ordenada por razón social y luego por identificador, sin términos financieros ni calificación ESG. (3) «Resultados (0)» y la fila vacía «Resultados».

#### TC-CAT-159 — Limpiar y volver con el deslizador; exportar proveedores

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-033 «Limpiar y volver»; «Exportar»; CU-CAT-33 A1; [RN-PRV-012][rn-b], [RN-IMP-008][rn-a], [RN-IMP-009][rn-a] |

**Pasos:** 1. Mueve el deslizador a 80 — 100, busca, abre un detalle y vuelve. 2. Haz clic en «Limpiar». 3. Busca y haz clic en «Exportar».

**Resultado esperado:** al volver desde un detalle el deslizador no se restaura (los textos sí); «Limpiar» lo devuelve a 0 — 100. Se descarga el Excel de proveedores aplicando los mismos filtros; si falla ves «Error al buscar proveedores», con «Referencia: …» si la respuesta trae referencia de soporte. **No verificado en la aplicación:** el nombre y el contenido del archivo.

### HU-CAT-034 — Eliminar un proveedor

#### TC-CAT-160 — Eliminar un proveedor sin precios

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-034 «Eliminar un proveedor sin precios»; CU-CAT-34 flujo básico y A1; [RN-PRV-009][rn-b], [RN-PRV-010][rn-b] |

**Precondiciones:** un proveedor sin precios de compra; `Proveedores:Eliminar`.
**Pasos:** 1. «Editar», «Eliminar»; cancela; repite y «Confirmar».

**Resultado esperado:** «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este proveedor?»; al confirmar «Proveedor eliminado correctamente» y vuelves a `/proveedores`; la API responde 200 con el identificador eliminado.

#### TC-CAT-161 — Bloqueo al eliminar un proveedor con precios de compra

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-034 «Proveedor con precios de compra»; CU-CAT-34 excepción paso 4 (`Proveedor.InUse`); [RN-PRV-010][rn-b] |

**Precondiciones:** un proveedor con un precio registrado con `TC-CAT-166`.
**Pasos:** 1. Elimínalo y confirma.

**Resultado esperado:** 409 `Proveedor.InUse` (bloqueo duro) y «No puedes eliminar este proveedor porque tiene precios de compra registrados.» **Hallazgo (pendiente 12):** como ninguna pantalla permite borrar un precio, un proveedor al que se registró un precio nunca más se podrá eliminar; la pantalla muestra ese texto ante cualquier 409.

#### TC-CAT-162 — Otro fallo al eliminar un proveedor

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Baja | HU-CAT-034 «Otro fallo»; CU-CAT-34 excepción paso 4 (`NotFound`); [RN-PRV-009][rn-b] |

**Pasos:** 1. Sesión 1: «Editar», «Eliminar». 2. Sesión 2: elimina el proveedor. 3. Sesión 1: «Confirmar».

**Resultado esperado:** la API responde 404 con el código genérico `NotFound` y ves «Error al eliminar el proveedor».

### HU-CAT-035 — Cargar proveedores desde un Excel

#### TC-CAT-163 — Columnas de la plantilla de proveedores

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-035 «Columnas de la plantilla»; CU-CAT-35 flujo básico paso 1; [RN-PRV-013][rn-b] |

**Pasos:** 1. Descarga la plantilla en `/proveedores/carga-masiva`.

**Resultado esperado:** `plantilla-proveedores.xlsx` con «RazonSocial» (obligatoria, 200 caracteres), «TaxId» (obligatoria, 50), «TasaCumplimiento» (obligatoria), «TerminosFinancieros» (opcional, 100) y «CalificacionEsg» (opcional); los números se leen con punto como separador decimal. Los escenarios CM-1 a CM-5 se prueban como en `TC-CAT-025` a `TC-CAT-033`.

#### TC-CAT-164 — Reglas por fila y duplicados del identificador en la vista previa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-035 «Reglas por fila en la vista previa»; «Identificador tributario existente o repetido en el archivo»; CU-CAT-35 excepción paso 2; [RN-PRV-002 a 006][rn-b], [RN-PRV-014][rn-b] |

**Precondiciones:** existe el proveedor con `TaxId` `20123456789`.
**Datos:** filas: (1) razón social vacía; (2) razón social de 201 letras; (3) `TasaCumplimiento` `abc`; (4) `TasaCumplimiento` `101`; (5) `CalificacionEsg` `101`; (6) `TaxId` `20123456789`; (7) y (8) `TaxId` `20999999999` repetido; (9) fila válida.
**Pasos:** 1. Sube el archivo.

**Resultado esperado:** (1) «El campo 'razonSocial' es obligatorio»; (2) «'razonSocial' no puede superar los 200 caracteres»; (3) «'tasaCumplimiento' debe ser un número decimal válido»; (4) «'tasaCumplimiento' debe estar entre 0 y 100»; (5) «'calificacionEsg' debe estar entre 0 y 100»; (6) «El valor '20123456789' del campo 'taxId' ya existe en el sistema»; (7) y (8) «…está duplicado en el archivo» en las dos filas; (9) válida. **Hallazgo (brecha):** la columna se rotula «Tax ID» y no «RUC / Tax ID».

#### TC-CAT-165 — Hallazgo: razón social repetida pasa la vista previa y se omite al guardar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-035 «Razón social repetida pasa la vista previa y se omite al guardar»; CU-CAT-35 excepción paso 3; [RN-PRV-008][rn-b], [RN-PRV-015][rn-b], [RN-PRV-016][rn-b], [RN-PRV-017][rn-b] |

**Precondiciones:** existe «Distribuidora Demo S.A.C.».
**Datos:** fila 1: razón social `Distribuidora Demo S.A.C.` con `TaxId` `20555555555`; fila 2: `Distribuidora Nueva Demo S.A.C.` con `TaxId` `20666666666`.
**Pasos:** 1. Sube el archivo. 2. «Guardar válidos (2)».

**Resultado esperado:** **Hallazgo (pendiente 3):** la vista previa muestra ambas válidas; el servidor omite la primera sin error y ves «Se guardaron 1 registros correctamente» (menos que las filas válidas). Solo API: lista vacía o de más de 5000, `CargaMasivaProveedoresCommand.Rows` (400); fila inválida, `CargaMasivaProveedoresCommand.Rows[0].RazonSocial` (400); vista previa sin archivo, «Debe adjuntar un archivo Excel.» (400). **No verificado:** que el guardado acepte los valores numéricos tal como los devuelve la vista previa (texto).

### HU-CAT-036 — Registrar el precio de compra de un producto con un proveedor

#### TC-CAT-166 — Registrar un precio de compra y ver el costo neto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-036 «Registrar un precio de compra»; «El costo neto se guarda, no el bruto»; CU-CAT-36 flujo básico; [RN-PRV-018][rn-b], [RN-PRV-020][rn-b], [RN-PRV-027][rn-b], [RN-PRV-028][rn-b] |

**Precondiciones:** `Proveedores:RegistrarPrecioProveedor` y `Medicamentos:Leer`; proveedor «Distribuidora Demo S.A.C.» abierto en modo ver; existe «Paracetamol Demo 500 mg».
**Datos:** Producto `Paracetamol` (elige «Paracetamol Demo 500 mg»); «Precio de compra por unidad base (S/)» `12.50`; «Descuento (%)» `10`.
**Pasos:** 1. Haz clic en «Registrar precio». 2. Completa. 3. Mira «Costo neto por unidad base». 4. «Guardar».

**Resultado esperado:** se abre «Registrar precio de compra» con «El nuevo precio reemplaza al vigente de este producto con este proveedor; el anterior queda en el historial.» «Costo neto por unidad base» muestra «S/ 11.25». Al guardar: «Precio registrado correctamente», el diálogo se cierra y la tarjeta se recarga desde la página 1. El servidor guarda el costo neto = precio × (1 − descuento / 100), redondeado a 4 decimales: 11.2500 (no guarda precio bruto ni descuento) y la vigencia empieza al instante del registro (hora UTC del servidor), sin fecha de fin. La tarjeta muestra «S/ 11.2500».

#### TC-CAT-167 — El precio nuevo reemplaza al vigente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-036 «El precio nuevo reemplaza al vigente»; CU-CAT-36 flujo básico paso 6; [RN-PRV-020][rn-b] |

**Precondiciones:** el producto tiene el precio vigente de `TC-CAT-166`.
**Pasos:** 1. Registra otro precio de `14.00` con descuento `0` para el mismo producto y proveedor. 2. Mira la tarjeta «Precios de compra».

**Resultado esperado:** el precio anterior queda cerrado con el instante del registro nuevo como «Vigente hasta»; el nuevo (S/ 14.0000, «Vigente») aparece primero. El producto tiene un solo precio vigente por proveedor. **No verificado:** la concurrencia (dos registros simultáneos podrían dejar dos precios abiertos: no hay índice único, ver `TC-CAT-173`).

#### TC-CAT-168 — Descuento 100 y costo neto cero

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-036 «Descuento 100»; CU-CAT-36 flujo básico; [RN-PRV-018][rn-b], [RN-PRV-027][rn-b] |

**Pasos:** 1. Registra un precio `10.00` con descuento `100`.

**Resultado esperado:** «Costo neto por unidad base» muestra «S/ 0.00» y se acepta.

#### TC-CAT-169 — Producto elegido de la lista y validaciones del diálogo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-036 «Producto de la lista»; «Precio y descuento fuera de rango»; CU-CAT-36 excepción paso 3; [RN-PRV-018][rn-b], [RN-PRV-027][rn-b], [RN-PRV-028][rn-b] |

**Datos:**

| Entrada | Resultado |
|---|---|
| Texto en «Producto» sin elegir opción | «Selecciona un producto de la lista», no se envía nada |
| Texto sin coincidencias | la lista dice «Sin resultados» y el campo muestra «Escribe para buscar un producto» |
| Precio o descuento vacíos | «Este campo es obligatorio» |
| Precio `0` o `-1` | «Debe ser mayor que 0» |
| Precio `10000000` (mayor que 9 999 999,9999) | «El precio es demasiado alto» |
| Descuento `-1` o `101` | «Debe estar entre 0 y 100» |

**Pasos:** 1. Prueba cada fila y mira «Costo neto por unidad base».

**Resultado esperado:** el de la tabla; mientras los valores sean inválidos, «Costo neto por unidad base» muestra «—».

#### TC-CAT-170 — Proveedor o producto eliminados mientras se registra el precio

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-CAT-036 «Proveedor o producto que ya no existen»; «Otro error al registrar»; CU-CAT-36 excepción paso 5; [RN-PRV-019][rn-b] |

**Pasos:** 1. Sesión 1 abre el diálogo y completa los datos; sesión 2 elimina el producto (y en otra ronda el proveedor); sesión 1 guarda.

**Resultado esperado:** proveedor eliminado: 404 `Proveedor.NotFound` (se comprueba primero) y «El proveedor ya no existe.»; producto eliminado: 404 `Medicamento.NotFound` y «El producto seleccionado ya no existe.»; el diálogo permanece abierto. Ante otro rechazo: «Error al registrar el precio» y «Guardar» se habilita de nuevo (**No verificado en la aplicación:** no se puede provocar).

#### TC-CAT-171 — Permisos del diálogo de precios

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | HU-CAT-036 «Sin el permiso no hay botón»; «Sin permiso de lectura de medicamentos»; CU-CAT-36 flujo básico paso 1; [RN-PRV-001][rn-b], [RN-PRV-028][rn-b] |

**Precondiciones:** rol A con `Proveedores` Leer y Actualizar pero sin «Registrar precio de proveedor»; rol B con «Registrar precio de proveedor» pero sin `Medicamentos:Leer`.
**Pasos:** 1. Con A abre un proveedor. 2. Con B abre «Registrar precio» y escribe en «Producto».

**Resultado esperado:** A no ve el botón «Registrar precio». B: la búsqueda falla en silencio y no aparecen opciones. Solo API: `POST /api/v1/Proveedores/precios` sin el permiso responde 403.

#### TC-CAT-172 — Plazo de entrega y cantidad mínima fijos; reglas internas del historial

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-036 «Valores que la API no deja informar»; «Reglas internas del historial (no alcanzables desde la pantalla)»; CU-CAT-36 excepción paso 4; [RN-PRV-021 a 024][rn-b] |

**Pasos:** 1. Registra un precio. 2. Mira el diálogo y la tarjeta. 3. Solo API: `POST /api/v1/Proveedores/precios` con precio `0` o descuento `101`.

**Resultado esperado:** no hay campos para la fecha de vigencia, el plazo de entrega (queda en 0) ni la cantidad mínima de compra (queda en 1); **Hallazgo (pendiente 12):** no se pueden informar. La API rechaza precio `0` o descuento `101` con 400 `RegistrarPrecioProveedorCommand.<Propiedad>` (el validador responde antes que la entidad); las reglas de la entidad (fin no anterior al inicio, plazo no negativo, cantidad mínima al menos 1) no son alcanzables desde la pantalla. **No verificado:** el nombre exacto de la propiedad.

#### TC-CAT-173 — Dos precios registrados a la vez para el mismo proveedor y producto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | HU-CAT-036 «El precio nuevo reemplaza al vigente»; CU-CAT-36 excepción paso 6; [RN-PRV-020][rn-b] |

**Precondiciones:** dos sesiones con el diálogo abierto para el mismo proveedor y producto.
**Pasos:** 1. Completa ambos con precios distintos. 2. Guarda ambos a la vez (dos pestañas o dos clientes HTTP). 3. Revisa la tarjeta.

**Resultado esperado:** **No verificado:** la regla «un solo precio abierto» la impone el handler, sin índice único; dos registros simultáneos podrían dejar dos precios sin fecha de fin. El tester anota cuántas filas dicen «Vigente».

### HU-CAT-037 — Consultar el historial de precios de compra de un proveedor

#### TC-CAT-174 — Ver la tarjeta de precios, el orden y los precios cerrados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-037 «Ver los precios»; «Orden y precios cerrados»; CU-CAT-37 flujo básico; [RN-PRV-025][rn-b], [RN-PRV-026][rn-b] |

**Precondiciones:** el proveedor con dos precios del mismo producto (uno cerrado).
**Pasos:** 1. Abre el proveedor en modo ver.

**Resultado esperado:** tarjeta «Precios de compra» con el subtítulo «Costo por unidad base que te cobra este proveedor, con su descuento ya aplicado. Cada producto tiene un solo precio vigente y los anteriores quedan en el historial.» y la tabla «Producto», «Costo por unidad base» («S/ » con 4 decimales), «Vigente desde» y «Vigente hasta» (fechas dd/MM/yyyy). El precio sin fecha de fin dice «Vigente» en la columna «Vigente hasta»; el nuevo aparece primero (inicio descendente) y el anterior con su fecha de fin.

#### TC-CAT-175 — Paginación propia, sin precios, producto eliminado, tarjeta oculta y error

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-037 «Paginación propia»; «Sin precios»; «Producto que ya no existe»; «La tarjeta no aparece al crear ni al editar»; «Error al cargar»; CU-CAT-37 A1, A2 y excepción paso 3; [RN-PRV-025][rn-b], [RN-PRV-026][rn-b] |

**Precondiciones:** un proveedor con más de 10 precios; otro sin precios; un precio cuyo medicamento fue eliminado.
**Pasos:** 1. Abre el primero y mira el paginador; registra un precio. 2. Abre el segundo. 3. Abre el que tiene el medicamento eliminado. 4. Abre «Nuevo» y «Editar» de un proveedor. 5. Abre `/proveedores/00000000-0000-0000-0000-000000000001`.

**Resultado esperado:** (1) el paginador ofrece 5, 10 y 25 filas (10 por defecto) y, al registrar un precio, la tabla vuelve a la primera página. (2) «Este proveedor aún no tiene precios registrados.» (3) la fila aparece con el nombre del producto vacío. (4) la tarjeta no se muestra. (5) 404 `Proveedor.NotFound` y «Error al cargar los precios del proveedor».

## 12. Aseguradoras

Catálogo de aseguradoras médicas (nombre comercial y código identificador nacional). Hoy solo se registran: el bloque de seguro del punto de venta está oculto. El módulo de permisos se llama `Seguros` (rotulado «Aseguradoras») y la pantalla «Aseguradoras Médicas». La pantalla no traduce ningún error ([19d sección 5][p19d]).

### HU-CAT-038 — Registrar y editar una aseguradora

#### TC-CAT-176 — Registrar una aseguradora

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-038 «Registrar una aseguradora»; CU-CAT-38 flujo básico; [RN-ASE-001 a 003][rn-b], [RN-ASE-007][rn-b] |

**Precondiciones:** `Seguros:Crear`.
**Datos:** Nombre Comercial `Aseguradora Demo`; Código Identificador Nacional `ASEG-0001`.
**Pasos:** 1. En `/aseguradoras` haz clic en «Nuevo». 2. Escribe los datos y «Guardar».

**Resultado esperado:** «Aseguradora guardada correctamente» y vuelves a `/aseguradoras`; la API responde 200 con el identificador (no 201: pendiente 17 de 12b 19.2).

#### TC-CAT-177 — Campos obligatorios y longitudes de la aseguradora

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-038 «Campos obligatorios y longitudes»; CU-CAT-38 excepción paso 3; [RN-ASE-002][rn-b], [RN-ASE-003][rn-b] |

**Pasos:** 1. Guarda con cada campo vacío (una vez cada uno). 2. Escribe 151 caracteres en «Nombre Comercial» y 51 en «Código Identificador Nacional».

**Resultado esperado:** «Este campo es obligatorio» debajo del campo y no se envía nada. «Nombre Comercial» no deja pasar de 150 caracteres y «Código Identificador Nacional» de 50. Con solo espacios: mensaje genérico «Error al guardar la aseguradora» (brecha 3; por lectura del código).

#### TC-CAT-178 — Hallazgo: código identificador nacional repetido se ve como error genérico

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-038 «Código identificador nacional repetido»; CU-CAT-38 excepción paso 6 (`Db.Duplicado`); [RN-ASE-004][rn-b] |

**Precondiciones:** existe la aseguradora con código `ASEG-0001`.
**Pasos:** 1. Registra otra con el mismo código.

**Resultado esperado:** **Hallazgo (pendiente 2):** los handlers no comprueban el duplicado; el índice único de la base lo rechaza (409 `Db.Duplicado`) y ves «Error al guardar la aseguradora» porque la pantalla no distingue el motivo (la clave `errors.dbDuplicado` existe pero esta pantalla no la usa).

#### TC-CAT-179 — El código de una aseguradora eliminada queda libre

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-038 «El código de una aseguradora eliminada queda libre»; CU-CAT-38 flujo básico; [RN-ASE-005][rn-b] |

**Pasos:** 1. Elimina la aseguradora con código `ASEG-0001` (`TC-CAT-183`). 2. Registra otra con ese código.

**Resultado esperado:** se guarda sin conflicto.

#### TC-CAT-180 — Editar una aseguradora y errores al guardar o abrir

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-038 «Editar una aseguradora»; «Cualquier otro error al guardar»; CU-CAT-38 A1; [RN-ASE-005][rn-b] |

**Precondiciones:** `Seguros:Actualizar`; dos sesiones.
**Pasos:** 1. Abre la aseguradora, «Editar», cambia el nombre y guarda. 2. Sesión 1 edita otra; sesión 2 la elimina; sesión 1 guarda y abre su enlace.

**Resultado esperado:** (1) «Aseguradora guardada correctamente», te quedas en el detalle y los datos se recargan. (2) 404 `Aseguradora.NotFound`: «Error al guardar la aseguradora» y, al abrirla, «Error al cargar la aseguradora».

### HU-CAT-039 — Buscar y exportar aseguradoras

#### TC-CAT-181 — Buscar aseguradoras y orden no garantizado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-039 «Buscar»; «Sin orden definido»; «Sin coincidencias»; CU-CAT-39 flujo básico; [RN-ASE-008][rn-b] |

**Precondiciones:** varias aseguradoras.
**Pasos:** 1. Abre `/aseguradoras` y busca sin filtros. 2. Filtra por «Nombre Comercial» y por «Código Identificador Nacional». 3. Busca un texto sin coincidencias.

**Resultado esperado:** filtros «Nombre Comercial» y «Código Identificador Nacional» («contiene»); la tabla muestra «Nombre Comercial» (enlace al detalle) y «Código Identificador Nacional». El orden de las filas no está garantizado (el servidor no aplica un orden explícito: no se afirma un orden en este caso). Sin coincidencias: «Resultados (0)» y la fila vacía «Resultados».

#### TC-CAT-182 — Exportar aseguradoras

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-039 «Exportar»; CU-CAT-39 A1; [RN-ASE-008][rn-b], [RN-IMP-008][rn-a], [RN-IMP-009][rn-a] |

**Pasos:** 1. Busca con un filtro y haz clic en «Exportar».

**Resultado esperado:** se descarga `aseguradoras.xlsx` con «Nombre Comercial» y «Código Identificador Nacional» y los mismos filtros; si falla, «Error al buscar aseguradoras», con «Referencia: …» si la respuesta trae referencia de soporte. **No verificado en la aplicación:** el contenido del archivo.

### HU-CAT-040 — Eliminar una aseguradora

#### TC-CAT-183 — Eliminar una aseguradora

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-040 «Eliminar una aseguradora»; CU-CAT-40 flujo básico y A1; [RN-ASE-005][rn-b], [RN-ASE-006][rn-b] |

**Pasos:** 1. Abre una aseguradora, «Editar», «Eliminar»; cancela; repite y «Confirmar».

**Resultado esperado:** «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar esta aseguradora?»; al confirmar «Aseguradora eliminada correctamente» y vuelves a `/aseguradoras`; la API responde 200 con el identificador eliminado.

#### TC-CAT-184 — Hallazgo: no se validan dependencias al eliminar una aseguradora

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-040 «No se validan dependencias»; CU-CAT-40 flujo básico paso 4; [RN-ASE-006][rn-b] |

**Precondiciones:** una aseguradora con reclamos de seguro asociados (se prepara por base de datos: no hay pantalla de reclamos). **No verificado en la aplicación.**
**Pasos:** 1. Elimínala y confirma.

**Resultado esperado:** **Hallazgo (pendiente 7):** se elimina igualmente porque el handler no comprueba dependencias.

#### TC-CAT-185 — Cualquier error al eliminar una aseguradora

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Baja | HU-CAT-040 «Cualquier error al eliminar»; CU-CAT-40 excepción paso 4 (`NotFound`) |

**Pasos:** 1. Sesión 1: «Editar», «Eliminar». 2. Sesión 2: elimina la aseguradora. 3. Sesión 1: «Confirmar».

**Resultado esperado:** la API responde 400 con el código genérico `NotFound` (no 404, porque el handler no indica el código HTTP) y ves siempre «Error al eliminar la aseguradora».

### HU-CAT-041 — Cargar aseguradoras desde un Excel

#### TC-CAT-186 — Columnas de la plantilla de aseguradoras

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-041 «Columnas de la plantilla»; 12b §1.3 «CM-1 Descargar la plantilla»; CU-CAT-41 flujo básico paso 1; [RN-ASE-009][rn-b] |

**Pasos:** 1. Descarga la plantilla en `/aseguradoras/carga-masiva`.

**Resultado esperado:** el SPA guarda `plantilla-aseguradoras.xlsx` (la API la nombra `plantilla-seguros.xlsx`) con «NombreComercial» y «CodigoIdentificadorNacional», ambas obligatorias. Los escenarios CM-1 a CM-4 y el guardado de CM-5 se prueban como en `TC-CAT-025` a `TC-CAT-033`.

#### TC-CAT-187 — Reglas por fila de la vista previa de aseguradoras

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-041 «Reglas por fila en la vista previa»; «Un nombre de más de 150 caracteres»; CU-CAT-41 excepción paso 2; [RN-ASE-002][rn-b], [RN-ASE-003][rn-b], [RN-ASE-010][rn-b] |

**Precondiciones:** existe la aseguradora con código `ASEG-0001`.
**Datos:** filas: (1) nombre vacío; (2) código vacío; (3) código `ASEG-0001`; (4) y (5) código `ASEG-0099` repetido; (6) nombre de 160 caracteres.
**Pasos:** 1. Sube el archivo.

**Resultado esperado:** (1) «El campo 'nombreComercial' es obligatorio»; (2) «El campo 'codigoIdentificadorNacional' es obligatorio»; (3) «El valor 'ASEG-0001' del campo 'codigoIdentificadorNacional' ya existe en el sistema»; (4) y (5) «…está duplicado en el archivo»; (6) **Hallazgo (pendiente 2):** «'nombreComercial' no puede superar los 200 caracteres» aunque el límite real es 150.

#### TC-CAT-188 — Hallazgo: un solo duplicado hace fallar todo el guardado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-041 «Un solo duplicado hace fallar todo el guardado»; CU-CAT-41 excepción paso 3; [RN-ASE-004][rn-b], [RN-ASE-011][rn-b] |

**Precondiciones:** dos sesiones; archivo de 3 filas válidas con códigos nuevos `ASEG-0101`, `ASEG-0102`, `ASEG-0103`.
**Pasos:** 1. Sesión 1: sube el archivo (vista previa con 3 válidas). 2. Sesión 2: registra a mano la aseguradora `ASEG-0102`. 3. Sesión 1: «Guardar válidos (3)».

**Resultado esperado:** **Hallazgo:** el índice único rechaza el guardado (409 `Db.Duplicado`), no se guarda ninguna de las tres filas y ves «Error al guardar los registros».

#### TC-CAT-189 — Solo API: aseguradoras sin validador de tamaño ni de extensión

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-CAT-041 «No se valida el tamaño de la lista en el guardado (solo API)»; «La extensión del archivo (solo la pantalla la exige)»; CU-CAT-41 excepción paso 3; [RN-ASE-011][rn-b], [RN-ASE-012][rn-b] |

**Precondiciones:** token con `Seguros:Crear`.
**Datos:** (a) `POST /api/v1/Seguros/carga-masiva` con lista vacía; (b) `POST /api/v1/Seguros/previsualizar` con un archivo `.xlsx` válido renombrado `datos.dat`; (c) lo mismo con un archivo de texto renombrado `.xlsx`.
**Resultado esperado:** (a) no existe validador del comando: el guardado procede con lo que pase los guards de la entidad (con lista vacía no hay nada que guardar); **No verificado:** la respuesta exacta. (b) el servidor no comprueba la extensión y procesa el archivo; (c) lo rechaza por la firma o el contenido. La pantalla exige `.xlsx` antes de enviar.

## 13. Catálogo DIGEMID

Tabla local de productos DIGEMID de la empresa (código, nombre, concentración, forma farmacéutica, fracción, registro sanitario, titular y estado), que se llena por carga masiva y valida los nombres que la IA detecta en una receta. SOFIA **no** se conecta a DIGEMID. Los datos de los casos son ficticios («DEMO-0001» a «DEMO-0010»).

### HU-CAT-042 — Buscar, ver y exportar el catálogo DIGEMID

#### TC-CAT-190 — Una empresa nueva tiene el catálogo vacío

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-042 «Una empresa nueva tiene el catálogo vacío»; CU-CAT-42 flujo básico; [RN-DIG-001][rn-b], [RN-DIG-009][rn-b] |

**Precondiciones:** empresa recién registrada, sin cargar productos DIGEMID.
**Pasos:** 1. Abre «Catálogo DIGEMID» y haz clic en «Buscar».

**Resultado esperado:** «Resultados (0)».

#### TC-CAT-191 — Buscar el catálogo por código y por nombre

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-042 «Buscar por código y por nombre»; «El filtro por nombre distingue los acentos»; «Tabla de resultados»; CU-CAT-42 flujo básico; [RN-DIG-009][rn-b] |

**Precondiciones:** cargados cinco productos: `DEMO-0001` y `DEMO-0003` con «TABLETA», `DEMO-0002` y `DEMO-0004` con «CAPSULA» (incluye «AMOXICILINA 500 MG CAPSULA (DEMO)» sin tilde) y otro «TABLETA».
**Datos y resultados:**

| Filtro | Resultado |
|---|---|
| «Código de Producto» `demo-000` y «Nombre del Producto» `capsula` | `DEMO-0002` y `DEMO-0004` |
| «Código de Producto» `0003` | `DEMO-0003` |
| «Nombre del Producto» `tabl` | los tres productos «TABLETA» |
| «Nombre del Producto» `cápsula` (con tilde) | no encuentra el producto |
| «Nombre del Producto» `capsula` | sí lo encuentra |

**Pasos:** 1. Busca con cada combinación; la búsqueda de código no distingue mayúsculas.

**Resultado esperado:** el de la tabla. La tabla muestra «Código» (enlace al detalle), «Nombre del Producto», «Forma Farmacéutica» («—» si no tiene) y «Estado», ordenados por nombre de producto, sin concentración, registro sanitario ni titular.

#### TC-CAT-192 — Ver el detalle de un producto DIGEMID

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-042 «Ver el detalle»; CU-CAT-42 A2; [RN-DIG-007][rn-b] |

**Pasos:** 1. Haz clic en el código `DEMO-0001`. 2. Intenta modificar un campo. 3. Vuelve con la flecha.

**Resultado esperado:** una tarjeta sin botones de edición con ocho campos siempre deshabilitados (Código de Producto, Nombre del Producto, Concentración, Forma Farmacéutica, Fracción, Registro Sanitario, Titular y Estado); los datos que el producto no tiene se ven vacíos; al volver regresas a la búsqueda con tus filtros.

#### TC-CAT-193 — Producto DIGEMID que no se puede abrir

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-CAT-042 «Producto que no se puede abrir»; CU-CAT-42 excepción paso 2; [RN-DIG-007][rn-b] |

**Pasos:** 1. Abre `/digemid-catalogo/00000000-0000-0000-0000-000000000001` y también el id de un producto de la empresa B. 2. Abre a mano `/digemid-catalogo/nueva`.

**Resultado esperado:** 404 `DigemidCatalogo.NotFound` y «Error al cargar el producto DIGEMID». Con `/digemid-catalogo/nueva` ves el formulario de detalle vacío y deshabilitado con un aviso de error de carga (**Hallazgo, brecha:** al recargar directamente esa dirección el aviso mostró la clave sin traducir `digemidCatalogo.detail.load-error`; manual cap. 4, §8.2). No se puede crear nada desde ahí.

#### TC-CAT-194 — Exportar el catálogo DIGEMID

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-042 «Exportar»; CU-CAT-42 A1; [RN-DIG-009][rn-b], [RN-IMP-008][rn-a], [RN-IMP-009][rn-a] |

**Pasos:** 1. Busca con `Nombre del Producto` `tabl`. 2. Haz clic en «Exportar».

**Resultado esperado:** se descarga `digemid-catalogo.xlsx` con las columnas «Código», «Nombre del Producto», «Forma Farmacéutica» y «Estado» (sin concentración, fracción, registro sanitario ni titular). Si falla: «Error al buscar en el catálogo DIGEMID». **No verificado en la aplicación:** el contenido del archivo.

#### TC-CAT-195 — El cajero solo consulta el catálogo DIGEMID

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | HU-CAT-042 «El cajero solo consulta»; CU-CAT-42 flujo básico paso 1; [RN-DIG-001][rn-b] |

**Precondiciones:** sesión de Carlos (rol «Cajero», solo `DIGEMID:Leer`).
**Pasos:** 1. Abre el menú «Catálogo DIGEMID». 2. Mira los botones. 3. Escribe `/digemid-catalogo/carga-masiva` a mano.

**Resultado esperado:** ves los filtros, la tabla y «Exportar», pero no «Carga Masiva»; al escribir la dirección ves «No tienes permisos para acceder a esta pantalla.» y vuelves al Inicio. **No verificado en la aplicación:** la redirección (12b sección 1.2).

### HU-CAT-043 — Cargar el catálogo DIGEMID desde un Excel

#### TC-CAT-196 — Columnas de la plantilla de DIGEMID

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-CAT-043 «Columnas de la plantilla»; 12b §1.3 «CM-1 Descargar la plantilla»; CU-CAT-43 flujo básico paso 1; [RN-DIG-010][rn-b] |

**Pasos:** 1. En «Catálogo DIGEMID» haz clic en «Carga Masiva». 2. Descarga la plantilla.

**Resultado esperado:** `plantilla-digemid-catalogo.xlsx` con ocho columnas: «CodProd» (obligatoria, 20 caracteres), «NomProd» (obligatoria, 255), «Concent» (255), «FormaFarmaceutica» (150), «Fraccion» (100), «RegistroSanitario» (50), «Titular» (255) y «Estado» (obligatoria, 50). El estado es texto libre (por ejemplo «ACTIVO»), sin lista de valores permitidos. Los escenarios CM-1 a CM-5 se prueban como en `TC-CAT-025` a `TC-CAT-033`.

#### TC-CAT-197 — Cargar productos válidos (9 filas, 5 válidas)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-CAT-043 «Cargar productos válidos»; CU-CAT-43 flujo básico; [RN-DIG-001 a 006][rn-b], [RN-DIG-011][rn-b] |

**Datos:** archivo de 9 filas: 5 productos válidos (`DEMO-0001` a `DEMO-0005`), una fila sin código, una sin estado y dos repetidas.
**Pasos:** 1. Sube el archivo. 2. «Guardar válidos (5)».

**Resultado esperado:** «9 filas totales», «5 filas válidas» y «4 filas con error»; la fila sin código muestra «El campo 'codProd' es obligatorio» y la sin estado «El campo 'estado' es obligatorio». Al guardar: «Se guardaron 5 registros correctamente» y vuelves a la búsqueda del catálogo.

#### TC-CAT-198 — Códigos repetidos sin distinguir mayúsculas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-043 «Códigos repetidos sin distinguir mayúsculas»; CU-CAT-43 excepción paso 2; [RN-DIG-006][rn-b], [RN-DIG-012][rn-b] |

**Precondiciones:** existe `DEMO-0001`.
**Datos:** filas con `DEMO-0008` y `demo-0008`; una fila con `demo-0001`.
**Pasos:** 1. Sube el archivo.

**Resultado esperado:** las dos filas `DEMO-0008`/`demo-0008` (también la primera) muestran «El valor '…' del campo 'codProd' está duplicado en el archivo»; la fila `demo-0001` muestra «El valor 'demo-0001' del campo 'codProd' ya existe en el sistema».

#### TC-CAT-199 — Hallazgo: un texto demasiado largo pasa la vista previa y rechaza todo el guardado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-CAT-043 «Un texto demasiado largo pasa la vista previa y rechaza todo el guardado»; CU-CAT-43 excepción paso 3; [RN-DIG-004][rn-b], [RN-DIG-005][rn-b], [RN-DIG-013][rn-b] |

**Datos:** archivo con una fila válida y otra con `RegistroSanitario` de 60 caracteres.
**Pasos:** 1. Sube el archivo; mira la vista previa. 2. «Guardar válidos (2)». 3. Busca el catálogo.

**Resultado esperado:** **Hallazgo (pendiente 5):** la vista previa muestra ambas válidas porque no revisa longitudes; al guardar el validador del comando rechaza el envío completo, ves «Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa.» y no se guarda nada (tampoco la fila válida).

### HU-CAT-044 — Crear, editar y eliminar un producto DIGEMID (solo API)

#### TC-CAT-200 — Solo API: crear un producto DIGEMID

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-CAT-044 «Crear un producto»; CU-CAT-44 flujo básico; [RN-DIG-001][rn-b], [RN-DIG-002][rn-b] |

**Precondiciones:** token con `DIGEMID:Crear`.
**Datos:** `POST /api/v1/DIGEMID/catalogo` con `{ "codProd": "DEMO-0010", "nomProd": "IBUPROFENO 400 MG TABLETA (DEMO)", "estado": "ACTIVO" }`.
**Resultado esperado:** 201 con el identificador; el producto aparece en `/digemid-catalogo`. **No verificado:** los nombres exactos de las propiedades del cuerpo.

#### TC-CAT-201 — Solo API: código y nombre obligatorios; código repetido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-044 «Código y nombre obligatorios»; «Código repetido»; CU-CAT-44 excepciones; [RN-DIG-002][rn-b], [RN-DIG-003][rn-b], [RN-DIG-006][rn-b] |

**Datos:** (a) `POST` con `codProd` vacío o de solo espacios; (b) lo mismo con `nomProd`; (c) `POST` con `codProd` `DEMO-0010` ya existente; (d) `PUT` de otro producto cambiando su código a `DEMO-0010`; (e) `PUT` de un producto sin cambiar su código.
**Resultado esperado:** (a) 400 `DIGEMID.CodProd`; (b) 400 `DIGEMID.NomProd`; (c) y (d) 409 `DigemidCatalogo.CodProd.Duplicado`; (e) se acepta (al editar se excluye el propio producto).

#### TC-CAT-202 — Solo API: editar y eliminar un producto DIGEMID inexistente o existente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-CAT-044 «Editar o eliminar un producto inexistente»; «Eliminar»; CU-CAT-44 excepciones; [RN-DIG-007][rn-b], [RN-DIG-008][rn-b] |

**Datos:** `PUT` y `DELETE` de `/api/v1/DIGEMID/catalogo/00000000-0000-0000-0000-000000000001`; luego `DELETE` de un producto existente; luego los mismos pedidos con un token sin `DIGEMID:Actualizar` / `DIGEMID:Eliminar`.
**Resultado esperado:** 404 `DigemidCatalogo.NotFound` para el inexistente; 204 para el existente (eliminación lógica, sin validación de dependencias); 403 sin permiso. **Hallazgo (brecha):** no hay validadores de crear ni editar: la respuesta ante un texto más largo que la columna o sin estado es **No verificado**.

### Comportamientos comunes de error (aplican a todos los catálogos)

#### TC-CAT-203 — Errores de base de datos poco frecuentes (valor único repetido a la vez)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | 12b §1.2 «Errores de base de datos poco frecuentes (no verificado en la aplicación)»; CU-CAT-01 excepción paso 6 (`Db.Duplicado`); [RN-UDM-006][rn-b] |

**Precondiciones:** dos clientes HTTP con token de `Admin` (para provocar la carrera se usa la API; la pantalla de unidades usa el mapa común de errores).
**Datos:** dos `POST /api/v1/UnidadesMedida` con el mismo código nuevo `CAR` enviados al mismo tiempo; en otra ronda, una cantidad o un valor fuera del rango de la columna (por ejemplo un multiplicador de jerarquía muy grande).
**Pasos:** 1. Envía las dos peticiones a la vez y repite varias veces. 2. Haz lo mismo desde dos pestañas del SPA con el formulario «Nuevo» de unidades.

**Resultado esperado:** según el caso, una de las dos peticiones puede fallar con 409 `Db.Duplicado` y el SPA muestra «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.»; una referencia a datos que ya no existen o en uso muestra «El registro hace referencia a datos que ya no existen o todavía está en uso.»; un valor fuera de rango muestra «Uno de los valores está fuera del rango permitido.» **No verificado en la aplicación:** ninguno de los tres avisos se pudo provocar (12b 19.3, número 1); las pantallas de Aseguradoras, Presentaciones de venta, Formulaciones clínicas, Lotes y Stock por sucursal no usan este mapa y tienen su propio tratamiento (brecha común de 12b).

## 14. Lotes de inventario

Un lote identifica una producción de un medicamento (número del fabricante, fecha de fabricación y de caducidad). El lote **no** guarda cantidades: las existencias viven en el stock por sucursal (sección 15). Los permisos de lotes y de stock son los del módulo `Inventarios` (rotulado «Inventario»). No hay carga masiva de lotes. El manual no recorrió estas pantallas; los textos vienen de 12b y de [19e secciones 1 a 3][p19e]. Las fechas de los ejemplos suponen que hoy es el 30/10/2026; el tester las adapta a la fecha real de ejecución.

### HU-INV-001 — Registrar y editar un lote

#### TC-INV-001 — Registrar un lote

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-INV-001 «Registrar un lote»; CU-INV-01 flujo básico; [RN-LOT-001 a 005][rn-c], [RN-LOT-016][rn-c] |

**Precondiciones:** `Inventarios:Crear` y `Medicamentos:Leer`; existe «Paracetamol Demo 500 mg».
**Datos:** Producto `Paracetamol` → «Paracetamol Demo 500 mg»; Número de Lote `L-2026-001`; Fecha de Fabricación `01/09/2026`; Fecha de Caducidad `31/08/2028`.
**Pasos:** 1. En `/lotes` haz clic en «Nuevo». 2. Completa y «Guardar». 3. Abre `/stock-por-sucursal` con «Solo con stock disponible» marcada.

**Resultado esperado:** «Lote guardado correctamente» y vuelves a `/lotes`; la API responde 201 con el identificador. El lote nace sin existencias: no aparece en «Stock por Sucursal» hasta ingresar su stock (`TC-INV-025`).

#### TC-INV-002 — La fecha de fabricación es opcional

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-INV-001 «La fecha de fabricación es opcional»; CU-INV-01 A2; [RN-LOT-007][rn-c] |

**Pasos:** 1. Registra un lote `L-2026-002` sin fecha de fabricación. 2. Búscalo en `/lotes`.

**Resultado esperado:** se guarda y en la tabla la fecha de fabricación se ve como «—».

#### TC-INV-003 — Campos obligatorios y longitud del número de lote

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-INV-001 «Campos obligatorios»; CU-INV-01 excepciones pasos 3 y 4; [RN-LOT-002][rn-c], [RN-LOT-003][rn-c], [RN-LOT-004][rn-c], [RN-LOT-005][rn-c] |

**Pasos:** 1. Guarda con producto, número y caducidad vacíos. 2. Elige un producto, cambia luego su texto y guarda. 3. Escribe 101 caracteres en «Número de Lote». 4. Guarda con el número de solo espacios.

**Resultado esperado:** (1) «Este campo es obligatorio» debajo de cada campo faltante y no se envía nada. (2) el producto debe elegirse de la lista: si cambias su texto después de elegir, la elección se descarta. (3) el campo no deja pasar de 100 caracteres, sin mensaje. (4) **Hallazgo (brecha 4):** llega al servidor y ves «Error al guardar el lote».

#### TC-INV-004 — Caducidad vencida

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-001 «Caducidad vencida»; CU-INV-01 excepción paso 4; [RN-LOT-006][rn-c] |

**Datos:** Fecha de Caducidad `29/10/2026` (ayer).
**Pasos:** 1. Completa el formulario con esa caducidad. 2. «Guardar».

**Resultado esperado:** el selector de fecha no limita nada; el validador rechaza la fecha (400 `CreateLoteInventarioCommand.FechaCaducidad`) y ves «Error al guardar el lote» (el rechazo no tiene mensaje propio: **Hallazgo, brecha 3**).

#### TC-INV-005 — Un lote que caduca hoy es válido todo ese día

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | HU-INV-001 «Un lote que caduca hoy es válido todo ese día»; CU-INV-01 flujo básico; [RN-LOT-006][rn-c] |

**Datos:** Fecha de Caducidad = hoy.
**Pasos:** 1. Registra un lote con la caducidad de hoy. 2. Repite con la de ayer (`TC-INV-004`).

**Resultado esperado:** con la de hoy se guarda («Lote guardado correctamente»), porque un lote se vende hasta el final de su día de caducidad; con la de ayer se rechaza. Se considera vencido cuando la caducidad es menor o igual que «ahora menos un día».

#### TC-INV-006 — Fabricación futura o posterior a la caducidad

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-INV-001 «Fabricación futura o posterior a la caducidad»; CU-INV-01 excepción paso 4; [RN-LOT-007][rn-c], [RN-LOT-008][rn-c] |

**Datos:** (a) fabricación `01/12/2026` (futura) con caducidad `31/08/2028`; (b) fabricación `01/01/2028` con caducidad `01/06/2027`.
**Resultado esperado:** (a) 400 `CreateLoteInventarioCommand.FechaFabricacion`; (b) 400 `LoteInventario.FechaFabricacion`; en ambos ves «Error al guardar el lote».

#### TC-INV-007 — Número de lote repetido para el mismo producto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-001 «Número de lote repetido para el mismo producto»; CU-INV-01 excepciones pasos 5 y 6; [RN-LOT-010][rn-c] |

**Precondiciones:** el producto A tiene el lote `L-2026-001`; existe el producto B.
**Pasos:** 1. Registra otro lote `L-2026-001` para A. 2. Registra `L-2026-001` para B. 3. Con dos sesiones, registra el mismo número nuevo para A a la vez (dos clientes HTTP al mismo tiempo).

**Resultado esperado:** (1) la API responde 400 `LoteInventario.Duplicate` y ves «Error al guardar el lote» (**Hallazgo, brecha 1:** el aviso propio casi nunca aparece). (2) se guarda: el mismo número puede repetirse en productos distintos. (3) si las dos altas llegan al índice único, una responde 409 `Db.Duplicado` y ves «Ya existe un lote con ese número para el producto seleccionado». **No verificado:** (3) y la distinción de mayúsculas (depende de la intercalación).

#### TC-INV-008 — Eliminar un lote libera su número

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-001 «Eliminar un lote libera su número»; CU-INV-01 flujo básico; [RN-LOT-010][rn-c], [RN-LOT-015][rn-c] |

**Pasos:** 1. Elimina el lote `L-2026-001` de A (sin stock, `TC-INV-020`). 2. Registra otro lote `L-2026-001` para A.

**Resultado esperado:** se guarda sin conflicto.

#### TC-INV-009 — Producto eliminado mientras se registra el lote

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-INV-001 «Producto que ya no existe»; CU-INV-01 excepción paso 5; [RN-LOT-009][rn-c] |

**Pasos:** 1. Sesión 1: llena el formulario con un producto de prueba. 2. Sesión 2: elimina ese producto. 3. Sesión 1: guarda.

**Resultado esperado:** 404 `Medicamento.NotFound` y «Error al guardar el lote».

#### TC-INV-010 — Editar un lote vigente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-001 «Editar un lote vigente»; CU-INV-01 A1; [RN-LOT-010][rn-c], [RN-LOT-012][rn-c] |

**Precondiciones:** `Inventarios:Actualizar`; un lote vigente.
**Pasos:** 1. Abre el lote, «Editar». 2. Intenta cambiar el producto. 3. Cambia el número y la caducidad y guarda.

**Resultado esperado:** el producto queda deshabilitado; se pueden cambiar número, fabricación y caducidad; al guardar «Lote guardado correctamente» y vuelves a `/lotes` (no te quedas en el detalle).

#### TC-INV-011 — Hallazgo: no se puede editar un lote ya vencido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-001 «No se puede editar un lote ya vencido»; CU-INV-01 A1 (excepción `UpdateLoteInventarioCommand.FechaCaducidad`); [RN-LOT-006][rn-c] |

**Precondiciones:** un lote con caducidad anterior a ayer (se obtiene por base de datos, o esperando a que venza uno creado con la caducidad de hoy).
**Pasos:** 1. Abre el lote, «Editar», corrige solo el número de lote y guarda.

**Resultado esperado:** **Hallazgo (pendiente 10, brecha 2):** la API vuelve a exigir caducidad vigente y responde 400 `UpdateLoteInventarioCommand.FechaCaducidad`; ves «Error al guardar el lote» mientras no le pongas una caducidad vigente.

#### TC-INV-012 — Hallazgo: al editar no se revisa la fabricación contra la caducidad

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-INV-001 «Al editar no se revisa fabricación contra caducidad»; CU-INV-01 A1; [RN-LOT-008][rn-c] |

**Precondiciones:** un lote vigente con caducidad `31/08/2028`.
**Pasos:** 1. Edítalo y pon la fabricación en `01/01/2029`; guarda.

**Resultado esperado:** **Hallazgo:** se guarda, porque esa comprobación solo existe al crear (al crear se rechaza: `TC-INV-006`).

#### TC-INV-013 — El lote ya no existe al guardar o abrir

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-INV-001 «El lote ya no existe»; «Cualquier 409 al guardar se muestra como lote duplicado»; CU-INV-01 A1 (excepción `LoteInventario.NotFound`); [RN-LOT-013][rn-c] |

**Pasos:** 1. Sesión 1: edita un lote. 2. Sesión 2: elimínalo. 3. Sesión 1: guarda y luego abre su enlace.

**Resultado esperado:** 404 `LoteInventario.NotFound`: «Error al guardar el lote» y «Error al cargar el lote». Cualquier 409 al guardar (referencia inválida `Db.ReferenciaInvalida` o conflicto de concurrencia) se muestra como «Ya existe un lote con ese número para el producto seleccionado», aunque no sea la causa real (**No verificado en la aplicación:** no se puede provocar).

#### TC-INV-014 — Solo API: identificador al editar un lote

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-INV-001 «Identificador al editar (solo API)»; CU-INV-01 A1 (excepción `UpdateLoteInventarioCommand.Id`); [RN-LOT-011][rn-c] |

**Datos:** `PUT /api/v1/LotesInventario/00000000-0000-0000-0000-000000000000` con cuerpo válido.
**Resultado esperado:** 400 `UpdateLoteInventarioCommand.Id`.

### HU-INV-002 — Buscar, consultar y exportar lotes

#### TC-INV-015 — Filtros, tabla y filtro de producto por texto en lotes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-002 «Filtros disponibles»; «Buscar»; «El producto se filtra por texto»; CU-INV-02 flujo básico; [RN-LOT-017][rn-c] |

**Precondiciones:** lotes de dos productos; un lote eliminado; un lote de un medicamento eliminado.
**Pasos:** 1. Abre `/lotes` (grupo de menú «Inventario») y mira los filtros. 2. «Buscar». 3. Escribe `paracetamol` en «Producto» sin elegir sugerencia y busca. 4. Escribe `l-2026` en «Número de Lote» y busca.

**Resultado esperado:** filtros «Producto» («Buscar producto...»), «Número de Lote», «Caducidad Desde», «Caducidad Hasta», «Fabricación Desde» y «Fabricación Hasta». Tabla: «Producto» (enlace al detalle), «Número de Lote», «Fecha de Fabricación» («—» si no tiene) y «Fecha de Caducidad» (dd/MM/yyyy), ordenada por fecha de creación, la más reciente primero, sin lotes eliminados ni los de un medicamento eliminado. (3) y (4) filtran por «contiene» sin distinguir mayúsculas.

#### TC-INV-016 — Rangos de fecha inclusivos en lotes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-INV-002 «Los rangos de fecha son inclusivos»; CU-INV-02 excepción paso 3; [RN-LOT-017][rn-c] |

**Precondiciones:** lotes que caducan el 30/12/2026, el 31/12/2026 y el 01/01/2027; un lote sin fecha de fabricación.
**Pasos:** 1. «Caducidad Hasta» `31/12/2026`. 2. «Fabricación Desde» o «Fabricación Hasta» con cualquier fecha. 3. «Caducidad Desde» `01/01/2027` y «Caducidad Hasta» `01/12/2026`.

**Resultado esperado:** (1) aparecen los lotes del 30 y 31/12/2026 (se incluye el día completo) y no el del 01/01/2027. (2) excluyen los lotes sin fecha de fabricación. (3) «Resultados (0)» (si «Desde» es posterior a «Hasta» no devuelve filas).

#### TC-INV-017 — Ver el detalle de un lote y lote que no se puede abrir

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-INV-002 «Ver el detalle»; «Lote que no se puede abrir»; CU-INV-02 flujo básico y excepción paso 5; [RN-LOT-013][rn-c] |

**Pasos:** 1. Haz clic en el producto de una fila. 2. Abre `/lotes/<id de un lote de la empresa B>`.

**Resultado esperado:** (1) `/lotes/:id` con los campos deshabilitados y, con `Inventarios:Actualizar`, el botón «Editar». (2) 404 `LoteInventario.NotFound` y «Error al cargar el lote».

#### TC-INV-018 — Exportar lotes con el formato de fecha del idioma

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-002 «Exportar con el formato de fecha del idioma»; CU-INV-02 A1; [RN-LOT-019][rn-c], [RN-IMP-008][rn-a], [RN-IMP-009][rn-a], [RN-IMP-011][rn-a] |

**Pasos:** 1. Con el SPA en español, busca y «Exportar». 2. Cambia el idioma del SPA a otro idioma y repite.

**Resultado esperado:** se descarga `lotes.xlsx` con «Producto», «Número de Lote», «Fecha de Fabricación» y «Fecha de Caducidad»; las fechas salen dd/MM/yyyy en español y MM/dd/yyyy con otro idioma. Si falla: «Error al buscar lotes». **No verificado en la aplicación:** el contenido del archivo. La tabla no marca los lotes vencidos (no hay indicador: pendiente 10).

### HU-INV-003 — Eliminar un lote

#### TC-INV-019 — Permisos de lotes: botones por permiso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | HU-INV-003 «Sin permiso no hay botón»; CU-INV-03 flujo básico paso 1; [RN-LOT-001][rn-c] |

**Precondiciones:** rol de prueba con `Inventarios:Leer` y `Actualizar` pero sin `Eliminar`, y otro sin `Crear`.
**Pasos:** 1. Abre un lote y haz clic en «Editar». 2. Mira `/lotes`.

**Resultado esperado:** no ves el botón «Eliminar» aunque entres en modo editar; sin `Inventarios:Crear` no ves «Nuevo» en `/lotes`.

#### TC-INV-020 — Eliminar un lote sin stock ni ventas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-003 «Eliminar un lote sin stock ni ventas»; CU-INV-03 flujo básico y A1; [RN-LOT-013][rn-c], [RN-LOT-015][rn-c] |

**Precondiciones:** un lote sin filas de stock ni ventas; `Inventarios:Eliminar`.
**Pasos:** 1. Abre el lote, «Editar», «Eliminar»; cancela; repite y «Confirmar».

**Resultado esperado:** «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este lote?»; al confirmar «Lote eliminado correctamente» y vuelves a `/lotes`; la API responde 204, la eliminación es lógica y el número queda libre.

#### TC-INV-021 — Bloqueo al eliminar un lote con stock o ventas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-003 «Lote con stock o ventas»; CU-INV-03 excepción paso 4 (`LoteInventario.InUse`); [RN-LOT-014][rn-c] |

**Precondiciones:** (a) un lote con una fila de stock de 10 unidades; (b) un lote con una fila de stock de **0** unidades; (c) un lote vendido en una venta no eliminada (parte C).
**Pasos:** 1. Elimina cada uno y confirma.

**Resultado esperado:** en los tres casos la API responde 409 `LoteInventario.InUse` y ves «No se puede eliminar un lote con stock o ventas registradas.»; el lote sigue existiendo. La fila con cantidad 0 también bloquea. La pantalla muestra ese texto ante cualquier 409. **No verificado en la aplicación:** la pantalla no se recorrió.

#### TC-INV-022 — Hallazgo: un lote usado en transferencias, cuarentena o magistrales se puede eliminar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-003 «Lote con stock o ventas» (brecha); CU-INV-03 flujo básico paso 4; [RN-LOT-014][rn-c] |

**Precondiciones:** un lote sin stock ni ventas pero referenciado por una línea de transferencia, una retención de cuarentena, una inmunización o una orden magistral (módulos sin pantalla; se prepara por API o base de datos).
**Pasos:** 1. Elimina el lote.

**Resultado esperado:** **Hallazgo (pendiente 7):** se elimina («Lote eliminado correctamente») porque el guard solo revisa stock y ventas y el borrado lógico no dispara las claves foráneas. **No verificado en la aplicación.**

#### TC-INV-023 — Otro fallo al eliminar un lote

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Baja | HU-INV-003 «Otro fallo»; CU-INV-03 excepción paso 4 (`LoteInventario.NotFound`); [RN-LOT-013][rn-c] |

**Pasos:** 1. Sesión 1: «Editar», «Eliminar». 2. Sesión 2: elimina el lote. 3. Sesión 1: «Confirmar».

**Resultado esperado:** 404 `LoteInventario.NotFound` y «Error al eliminar el lote».

### HU-INV-004 — Consultar los lotes de un producto (solo API)

#### TC-INV-024 — Solo API: lotes de un producto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-004 «Lotes de un producto»; «Producto inexistente»; CU-INV-04 flujo básico y A1; [RN-LOT-018][rn-c], [RN-LOT-001][rn-c] |

**Precondiciones:** un producto con tres lotes de caducidades distintas; token con `Inventarios:Leer`.
**Datos:** `GET /api/v1/LotesInventario/producto/<id-producto>`; luego con un id inexistente; luego con un token sin `Inventarios:Leer`.
**Resultado esperado:** 200 con la lista paginada ordenada por caducidad descendente; 200 con lista vacía para el id inexistente (no se comprueba que el producto exista); 403 sin permiso. Si falta el producto en un lote, su nombre sale como «Unknown».

## 15. Stock por sucursal

El stock guarda cuántas unidades físicas hay de cada lote en cada sucursal. Cambia por ingreso, ajuste, venta, devolución, transferencias y cuarentena; no existe operación de eliminar una fila de stock. Las transferencias entre sucursales (HU-OPS-001 a 005) están en la [parte C][hu-c] y sus casos en el documento 16c; aquí solo se cubren el ingreso, el ajuste y la consulta. El manual no recorrió estas pantallas. Los casos usan las sucursales «Sede Principal» y «Sucursal Demo Norte», el lote `L-2026-001` de «Paracetamol Demo 500 mg» y el almacenero `alma.cenero`, cuya sucursal base es «Sucursal Demo Norte».

### HU-INV-005 — Ingresar stock de un lote en una sucursal

#### TC-INV-025 — Ingresar stock de un lote nuevo en una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-INV-005 «Ingresar stock de un lote nuevo en una sucursal»; CU-INV-05 flujo básico; [RN-STK-001 a 004][rn-c], [RN-STK-009][rn-c] |

**Precondiciones:** `Inventarios:Actualizar` y `Leer`; el lote `L-2026-001` existe y no tiene stock en «Sede Principal».
**Datos:** Sucursal «Sede Principal»; Lote `L-2026-001` («L-2026-001 — Paracetamol Demo 500 mg»); Cantidad Física `100`.
**Pasos:** 1. En `/stock-por-sucursal` haz clic en «Nuevo». 2. Escribe `Sede` en «Sucursal» y elige «Sede Principal». 3. Escribe `L-2026-001` en «Lote» y elige la opción. 4. Escribe 100 y «Guardar». 5. Busca el stock.

**Resultado esperado:** «Stock registrado correctamente» y vuelves a `/stock-por-sucursal`; se crea una fila con 100 unidades de ese lote en esa sucursal; la API responde 200 con el identificador de la fila (no 201: pendiente 17). En el detalle del medicamento, «Stock Total» sube a 100.

#### TC-INV-026 — Si el lote ya tenía stock en esa sucursal, la cantidad se suma

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-INV-005 «Si el lote ya tenía stock en esa sucursal, la cantidad se suma»; «Una sucursal tiene una sola fila por lote»; CU-INV-05 flujo básico paso 6; [RN-STK-008][rn-c], [RN-STK-010][rn-c] |

**Precondiciones:** «Sede Principal» tiene 100 unidades del lote `L-2026-001`.
**Pasos:** 1. Ingresa 50 unidades del mismo lote en la misma sucursal. 2. Busca el stock con «Número de Lote» `L-2026-001`.

**Resultado esperado:** el stock queda en 150 en una **única** fila (la base tiene un índice único por sucursal y lote entre las filas no eliminadas) y el aviso es el mismo «Stock registrado correctamente». La pantalla nunca envía el indicador de reemplazo directo, así que desde el SPA el ingreso siempre suma.

#### TC-INV-027 — Quién puede abrir la pantalla de ingreso de stock

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | HU-INV-005 «Quién puede abrir la pantalla»; CU-INV-05 flujo básico paso 1; [RN-STK-001][rn-c] |

**Precondiciones:** rol R1 con `Inventarios:Leer` y `Actualizar`; rol R2 con `Inventarios:Leer` y `Crear` (sin `Actualizar`).
**Pasos:** 1. Con R1 abre `/stock-por-sucursal` y `/stock-por-sucursal/nueva`. 2. Con R2 repite.

**Resultado esperado:** R1 ve el botón «Nuevo» y abre la ruta. R2 no ve «Nuevo» y al escribir la ruta ve «No tienes permisos para acceder a esta pantalla.» y vuelve al Inicio (**No verificado en la aplicación:** la redirección). Solo API: `POST /api/v1/LotesInventario/stock` con el token de R2 responde 403.

#### TC-INV-028 — Solo se ofrecen mis sucursales al ingresar stock

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | HU-INV-005 «Solo se ofrecen mis sucursales»; CU-INV-05 flujo básico paso 2 y excepción paso 2; [RN-STK-006][rn-c], [RN-GEN-023][rn-a] |

**Precondiciones:** `alma.cenero` (sucursal base «Sucursal Demo Norte», sin otras sucursales asignadas); Ana (`Admin`).
**Pasos:** 1. Con `alma.cenero` abre «Nuevo» en `/stock-por-sucursal` y escribe `S` en «Sucursal». 2. Repite con Ana.

**Resultado esperado:** `alma.cenero` ve solo «Sucursal Demo Norte»; Ana ve todas (incluida «Sede Principal»). Si falla la carga ves «No se pudieron cargar tus sucursales» (**No verificado en la aplicación**).

#### TC-INV-029 — Cantidad negativa, vacía, cero y decimales al ingresar stock

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | HU-INV-005 «Cantidad negativa»; CU-INV-05 excepción paso 3; [RN-STK-004][rn-c] |

**Datos:**

| Cantidad Física | Resultado |
|---|---|
| `-5` | «La cantidad no puede ser negativa», no se envía nada |
| (vacío) | «La cantidad no puede ser negativa», no se envía nada |
| `0` (valor inicial) | Válido: se guarda con «Stock registrado correctamente» |
| `1,2345` | Válido (hasta 4 decimales): queda 1,2345 |
| `1,23456` | **No verificado:** si el campo lo acepta y cómo lo guarda la base (`DECIMAL(12,4)`) |

**Pasos:** 1. Elige sucursal y lote y prueba cada fila.

**Resultado esperado:** el de la tabla. Un valor que desborde `DECIMAL(12,4)` es **No verificado**.

#### TC-INV-030 — Solo API: ingresar stock en una sucursal que no tengo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | HU-INV-005 «Sucursal fuera de mi acceso (solo API)»; CU-INV-05 excepción paso 5 (`Inventario.Sucursal.NoPermitida`); [RN-STK-006][rn-c] |

**Precondiciones:** token de `alma.cenero` (acceso solo a «Sucursal Demo Norte»).
**Datos:** `POST /api/v1/LotesInventario/stock` con `{ "sucursalId": "<id Sede Principal>", "loteId": "<id lote>", "cantidad": 10 }`.
**Resultado esperado:** 403 `Inventario.Sucursal.NoPermitida`; en la pantalla se vería «No tienes permiso para operar el stock de esta sucursal». No se crea ni modifica ninguna fila. **No verificado:** los nombres exactos de las propiedades del cuerpo.

#### TC-INV-031 — Sucursal o lote eliminados mientras se ingresa el stock

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-INV-005 «Sucursal o lote que ya no existen»; CU-INV-05 excepción paso 5; [RN-STK-005][rn-c], [RN-STK-007][rn-c] |

**Pasos:** 1. Sesión 1: llena el formulario con un lote de prueba sin guardar. 2. Sesión 2: elimina ese lote (sin stock). 3. Sesión 1: guarda. 4. Repite con una sucursal de prueba eliminada.

**Resultado esperado:** 404 `LoteInventario.NotFound` o `Sucursal.NotFound` (esta se comprueba antes que el permiso sobre la sucursal) y «Error al registrar el stock».

#### TC-INV-032 — Hallazgo: se puede ingresar stock de un lote vencido o retenido

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-005 «Se puede ingresar stock de un lote vencido o retenido»; CU-INV-05 flujo básico paso 5; [RN-STK-007][rn-c], [RN-STK-017][rn-c] |

**Precondiciones:** un lote vencido (por base de datos) y un lote retenido en cuarentena (por API de cuarentena o base de datos: **No verificado** cómo prepararlo sin pantalla).
**Pasos:** 1. Ingresa 5 unidades del lote vencido. 2. Ingresa 5 unidades del lote retenido. 3. Busca el stock con «Solo con stock disponible» marcada.

**Resultado esperado:** **Hallazgo (pendiente 11):** ambos ingresos se registran sin error («Stock registrado correctamente») porque el ingreso no comprueba vigencia ni cuarentena. Las filas aparecen en la lista, pero ese stock no es vendible (`TC-INV-047`).

#### TC-INV-033 — Dos ingresos simultáneos crean la misma fila

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | HU-INV-005 «Dos ingresos simultáneos crean la misma fila»; CU-INV-05 excepción paso 6 (`InventarioSucursal.RegistroConcurrente`); [RN-STK-010][rn-c] |

**Precondiciones:** un lote sin stock en «Sede Principal»; dos clientes HTTP con token de `Admin`.
**Datos:** dos `POST /api/v1/LotesInventario/stock` idénticos para el mismo lote y sucursal, con cantidad 10, enviados al mismo tiempo.
**Pasos:** 1. Envía las dos peticiones a la vez (por ejemplo desde dos pestañas del cliente). 2. Consulta el stock del lote.

**Resultado esperado:** o las dos se aplican en serie (queda una sola fila con 20), o la segunda responde 409 `InventarioSucursal.RegistroConcurrente` (en la pantalla: «Error al registrar el stock»; hay que reintentar). Nunca debe haber dos filas del mismo lote en la sucursal. **No verificado:** la ventana de carrera es estrecha y puede requerir varios intentos.

#### TC-INV-034 — Otra operación cambió el mismo stock durante el ingreso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | HU-INV-005 «Otra operación cambió el mismo stock»; CU-INV-05 excepción paso 6 (`Concurrency.Conflict`); [RN-STK-011][rn-c], [RN-GEN-035][rn-a] |

**Precondiciones:** la fila de stock del lote tiene 100 unidades.
**Pasos:** 1. Sesión 1 abre «Nuevo» en stock y completa el ingreso sin guardar. 2. Sesión 2 ajusta esa fila a 90 (`TC-INV-037`). 3. Sesión 1 guarda. **No verificado:** el ingreso no lee la fila antes de abrir el formulario, así que el conflicto solo ocurre si la modificación cae entre la lectura y el guardado dentro de la misma petición; si el paso 3 simplemente suma, el tester lo anota.

**Resultado esperado:** si la fila cambió entre la lectura y el guardado, 409 `Concurrency.Conflict` y «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.»; si no, el ingreso suma sobre el valor vigente (90 + cantidad).

#### TC-INV-035 — Hallazgo: guardar el ingreso sin elegir un lote no muestra nada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-INV-005 (brecha conocida 1 y 2, sin escenario propio); CU-INV-05 excepción paso 3 (guardar sin haber elegido un lote); [RN-STK-002][rn-c], [RN-STK-003][rn-c] |

**Pasos:** 1. En «Nuevo», elige solo la sucursal, escribe una cantidad y «Guardar». 2. Deja también la sucursal sin elegir y guarda.

**Resultado esperado:** **Hallazgo (pendiente 11):** por lectura del código `saveNew()` lanza una excepción de JavaScript: no se envía ninguna petición y no se ve ningún aviso; los textos «Debes seleccionar una sucursal» y «Debes seleccionar un lote» no llegan a pintarse. **No verificado en ejecución:** deducido de leer el código. Solo API: sucursal o lote vacíos → 400 `RegisterInventarioCommand.SucursalId` o `RegisterInventarioCommand.LoteId`.

#### TC-INV-036 — Solo API: reemplazo directo de la cantidad

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | CU-INV-05 A1 (reemplazo directo); [RN-STK-008][rn-c], [RN-GEN-013][rn-a] |

**Precondiciones:** la fila de stock del lote en «Sede Principal» tiene 150; token de `Admin`.
**Datos:** `POST /api/v1/LotesInventario/stock` con sucursal y lote de esa fila, `cantidad` 140 y `esAjusteDirecto` `true`.
**Resultado esperado:** la cantidad pasa a ser 140 (reemplaza, no suma) y se genera un evento de auditoría `STOCK_AJUSTAR`. El ingreso por suma (sin el indicador) no genera evento. **No verificado:** el nombre exacto del campo `esAjusteDirecto` y cómo ver el evento (la auditoría no tiene pantalla).

### HU-INV-006 — Ajustar el stock de un lote en una sucursal

#### TC-INV-037 — Ajustar la cantidad de un lote en una sucursal

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-INV-006 «Ajustar la cantidad»; CU-INV-06 flujo básico; [RN-STK-001][rn-c], [RN-STK-012][rn-c], [RN-GEN-013][rn-a] |

**Precondiciones:** `Inventarios:Actualizar`; el registro del lote `L-2026-001` en «Sede Principal» con 150 unidades.
**Pasos:** 1. Abre el registro desde la tabla («Detalle de Stock») y haz clic en «Editar». 2. Intenta editar otros campos. 3. Escribe `140` en «Cantidad Física» y «Guardar». 4. Busca el stock.

**Resultado esperado:** solo se habilita «Cantidad Física» (sucursal, producto, lote y caducidad son de solo lectura). Al guardar ves «Cantidad ajustada correctamente» y vuelves a `/stock-por-sucursal`; la cantidad física es 140 (el ajuste fija el valor, no suma) y se registra el evento de auditoría `STOCK_AJUSTAR` con la nueva cantidad (**No verificado en la aplicación:** la auditoría no tiene pantalla).

#### TC-INV-038 — Cantidad negativa o vacía al ajustar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | HU-INV-006 «Cantidad negativa»; CU-INV-06 excepción paso 2; [RN-STK-004][rn-c] |

**Pasos:** 1. Edita un registro de stock y escribe `-1`; guarda. 2. Déjalo vacío; guarda. 3. Escribe `0`; guarda.

**Resultado esperado:** (1) y (2) «La cantidad no puede ser negativa» y no se envía nada; (3) la cantidad 0 es válida y se guarda («Cantidad ajustada correctamente»); la fila sigue existiendo con cantidad 0.

#### TC-INV-039 — Sin el permiso Actualizar no hay botón Editar; cancelar el ajuste

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | HU-INV-006 «Sin el permiso no hay botón»; «Cancelar el ajuste»; CU-INV-06 A1 y A2; [RN-STK-001][rn-c] |

**Precondiciones:** rol con solo `Inventarios:Leer`; rol con `Leer` y `Actualizar`.
**Pasos:** 1. Con el primero abre el detalle de un registro. 2. Con el segundo entra en «Editar», cambia la cantidad y haz clic en «Cancelar».

**Resultado esperado:** (1) ves el detalle pero no el botón «Editar». (2) la cantidad vuelve a la leída y el detalle vuelve al modo ver. Solo API: `PUT /api/v1/StockPorSucursal/<id>/ajustar` con el token del primero responde 403.

#### TC-INV-040 — Hallazgo: sucursal fuera de mi acceso (consultar 404, ajustar 403)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | HU-INV-006 «Sucursal fuera de mi acceso»; CU-INV-06 excepciones pasos 1 y 3; [RN-STK-006][rn-c], [RN-STK-013][rn-c], [RN-STK-014][rn-c] |

**Precondiciones:** una fila de stock en «Sede Principal»; sesión de `alma.cenero` (solo «Sucursal Demo Norte»).
**Pasos:** 1. Abre `/stock-por-sucursal/<id de la fila de Sede Principal>`. 2. Solo API: `PUT /api/v1/StockPorSucursal/<id>/ajustar` con `{ "nuevaCantidad": 5 }` y el token de `alma.cenero`.

**Resultado esperado:** (1) 404 `InventarioSucursal.NotFound`, la tarjeta no se pinta y ves «Error al cargar el registro de stock». (2) **Hallazgo (pendiente 11):** 403 `Inventario.Sucursal.NoPermitida` (en la pantalla: «No tienes permiso para operar el stock de esta sucursal»); la cantidad no cambia.

#### TC-INV-041 — Otra operación cambió el stock mientras se ajustaba

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Alta | HU-INV-006 «Otra operación cambió el stock»; CU-INV-06 excepción paso 4 (`Concurrency.Conflict`); [RN-STK-011][rn-c], [RN-GEN-035][rn-a] |

**Precondiciones:** dos sesiones de `Admin`; la fila tiene 150 unidades.
**Pasos:** 1. Sesión 1: abre el registro y haz clic en «Editar». 2. Sesión 2: ajusta la misma fila a 120 y guarda (o realiza una venta del lote, parte C). 3. Sesión 1: escribe 140 y «Guardar».

**Resultado esperado:** 409 `Concurrency.Conflict` y «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.»; la fila queda con 120 (no se pisa el cambio).

#### TC-INV-042 — El registro de stock ya no existe y el id al ajustar (solo API)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-INV-006 «El registro ya no existe»; «Identificador del registro (solo API)»; CU-INV-06 excepción paso 3; [RN-STK-012][rn-c], [RN-STK-013][rn-c] |

**Pasos:** 1. Ajusta un registro de otra empresa (id de la empresa B) o inexistente. 2. API: `PUT /api/v1/StockPorSucursal/00000000-0000-0000-0000-000000000000/ajustar`.

**Resultado esperado:** (1) 404 `InventarioSucursal.NotFound` y «Error al ajustar la cantidad». (2) 400 `AdjustStockCommand.Id`. No existe botón ni endpoint para eliminar una fila de stock.

### HU-INV-007 — Consultar y exportar el stock por sucursal

#### TC-INV-043 — Filtros disponibles y «Solo con stock disponible»

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-007 «Filtros disponibles»; «Por defecto solo veo filas con stock»; CU-INV-07 flujo básico y A3; [RN-STK-015][rn-c] |

**Precondiciones:** filas de stock con cantidad 100, 10 y 0.
**Pasos:** 1. Abre `/stock-por-sucursal` y mira los filtros. 2. «Buscar». 3. Desmarca la casilla y busca. 4. «Limpiar».

**Resultado esperado:** filtros «Sucursal» («Buscar sucursal...»), «Producto» («Buscar producto...»), «Número de Lote», «Caducidad Desde», «Caducidad Hasta», «Cantidad Mínima», «Cantidad Máxima» y la casilla «Solo con stock disponible» marcada. (2) solo filas con cantidad mayor que 0. (3) también las de cantidad 0. (4) «Limpiar» vuelve a marcar la casilla.

#### TC-INV-044 — Solo veo el stock de mis sucursales

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | HU-INV-007 «Solo veo el stock de mis sucursales»; CU-INV-07 flujo básico paso 3; [RN-STK-014][rn-c], [RN-GEN-023][rn-a] |

**Precondiciones:** filas de stock en «Sede Principal» y en «Sucursal Demo Norte»; una fila cuyo lote fue eliminado y otra cuyo producto fue eliminado.
**Pasos:** 1. Con `alma.cenero` busca sin filtros y abre una fila. 2. Con Ana (`Admin`) busca sin filtros.

**Resultado esperado:** `alma.cenero` ve solo filas de «Sucursal Demo Norte» (lista y detalle); Ana ve todas. No aparecen registros cuya sucursal, lote o producto estén eliminados.

#### TC-INV-045 — Tabla de resultados del stock

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-INV-007 «Tabla de resultados»; CU-INV-07 flujo básico paso 4; [RN-STK-015][rn-c] |

**Pasos:** 1. Busca sin filtros con stock en dos sucursales y dos productos.

**Resultado esperado:** columnas «Sucursal» (enlace al detalle), «Producto», «Número de Lote», «Fecha de Caducidad» y «Cantidad Física» (a la derecha), ordenadas por sucursal, nombre del producto y caducidad ascendente.

#### TC-INV-046 — Filtros de caducidad y cantidad del stock

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-INV-007 «Filtros de caducidad y cantidad»; CU-INV-07 flujo básico paso 3; [RN-STK-015][rn-c] |

**Precondiciones:** filas con caducidad 31/12/2026 y cantidades 9, 10 y 11; otra con caducidad 01/01/2027 y cantidad 50.
**Pasos:** 1. «Caducidad Hasta» `31/12/2026` y «Cantidad Mínima» `10`. 2. «Cantidad Máxima» `10`.

**Resultado esperado:** (1) las filas que caducan hasta el 31/12/2026 (día completo incluido) con 10 o más unidades (10 y 11), sin la de cantidad 9 ni la del 01/01/2027. (2) «Cantidad Mínima» y «Máxima» son inclusivas: solo la de 10.

#### TC-INV-047 — Un lote vencido sigue apareciendo pero no es vendible

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-007 «Un lote vencido sigue apareciendo pero no es vendible»; CU-INV-07 A2; [RN-STK-017][rn-c], [RN-LOT-006][rn-c] |

**Precondiciones:** un lote vencido con 10 unidades en «Sede Principal»; un lote vigente con 10 unidades.
**Pasos:** 1. Busca el stock con «Solo con stock disponible» marcada. 2. Solo API: `GET /api/v1/Medicamentos?IncluirStock=true&SoloStockVendibleEnMiSucursal=true` con un usuario de «Sede Principal».

**Resultado esperado:** (1) el lote vencido sigue apareciendo en la lista y la tabla no tiene indicador de vencido. (2) El stock vendible del medicamento solo cuenta el lote vigente (10): cantidad mayor que 0, lote no vencido y sin retención activa en cuarentena.

#### TC-INV-048 — Exportar el stock por sucursal y errores

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-007 «Exportar»; «Errores»; CU-INV-07 A1 y excepciones; [RN-STK-016][rn-c], [RN-IMP-008][rn-a], [RN-IMP-009][rn-a], [RN-IMP-011][rn-a] |

**Pasos:** 1. Busca con «Solo con stock disponible» desmarcada y un filtro de sucursal. 2. «Exportar» en español y con otro idioma.

**Resultado esperado:** se descarga `stock-por-sucursal.xlsx` con «Sucursal», «Producto», «Número de Lote», «Fecha de Caducidad» y «Cantidad Física»; fechas dd/MM/yyyy en español y MM/dd/yyyy con otro idioma; respeta los filtros escritos, incluida la casilla. Si falla la búsqueda o la exportación: «Error al consultar el stock» (y «No se pudieron cargar tus sucursales» si falla la carga de sucursales). Con más de 10000 filas: 400 `Export.TooManyRows`. **No verificado en la aplicación:** el contenido del archivo.

### HU-INV-008 — Consultar el stock de un medicamento en todas las sucursales (solo API)

#### TC-INV-049 — Solo API: stock de un medicamento por sucursal y lote

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-INV-008 «Stock de un medicamento»; CU-INV-08 flujo básico; [RN-STK-018][rn-c], [RN-LOT-001][rn-c] |

**Precondiciones:** un medicamento con lotes en dos sucursales (lote A: 30 unidades en «Sede Principal»; lote B: 12 en «Sucursal Demo Norte»); token con `Inventarios:Leer`.
**Datos:** `GET /api/v1/LotesInventario/stock/medicamento/<id-medicamento>`.
**Resultado esperado:** 200 con el total (42) y el desglose por sucursal y por lote, ordenado por caducidad.

#### TC-INV-050 — Hallazgo: el stock del medicamento suma toda la empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | HU-INV-008 «Suma toda la empresa»; CU-INV-08 flujo básico; [RN-STK-018][rn-c], [RN-STK-014][rn-c] |

**Precondiciones:** el medicamento de `TC-INV-049` más un lote vencido con 5 unidades y una fila con cantidad 0; token de `alma.cenero` (sin acceso a «Sede Principal»).
**Datos:** `GET /api/v1/LotesInventario/stock/medicamento/<id-medicamento>`.
**Resultado esperado:** **Hallazgo:** la suma incluye todas las sucursales de la empresa aunque `alma.cenero` no tenga acceso a «Sede Principal» (no aplica el filtro de sucursales permitidas), y también cuenta lotes vencidos y filas con cantidad 0.

#### TC-INV-051 — Solo API: stock de un medicamento inexistente o sin permiso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-INV-008 «Medicamento inexistente»; CU-INV-08 excepción paso 2; [RN-STK-018][rn-c], [RN-STK-001][rn-c] |

**Datos:** `GET /api/v1/LotesInventario/stock/medicamento/00000000-0000-0000-0000-000000000001`; luego con un token sin `Inventarios:Leer`.
**Resultado esperado:** 404 `Medicamento.NotFound`; 403 sin permiso.

## 16. Pacientes

Catálogo de pacientes y clientes (tipo y número de documento, nombre y apellidos, fecha de nacimiento y contacto). Los pacientes se usan en las recetas y en las ventas. Todos los datos de los casos son ficticios. El manual no recorrió esta pantalla; los textos vienen de 12b y de [19e sección 3][p19e].

### HU-INV-009 — Registrar y editar un paciente

#### TC-INV-052 — Registrar un paciente con DNI

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-009 «Registrar un paciente con DNI»; CU-INV-09 flujo básico; [RN-PAC-001 a 007][rn-b] |

**Precondiciones:** `Pacientes:Crear`.
**Datos:** Tipo de Documento «DNI»; Documento de Identidad `12345678`; Nombre y Apellidos `Rosa Paciente Demo`; Fecha de Nacimiento `15/03/1985`; Contacto Primario `987654321`.
**Pasos:** 1. En «Pacientes» (grupo de menú «Pacientes & Atención») haz clic en «Nuevo». 2. Completa y «Guardar».

**Resultado esperado:** «Paciente guardado correctamente» y vuelves a `/pacientes`; la API responde 200 con el identificador (no 201).

#### TC-INV-053 — Tipos de documento admitidos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-INV-009 «Tipos de documento admitidos»; CU-INV-09 flujo básico paso 2; [RN-PAC-002][rn-b] |

**Pasos:** 1. Abre la lista «Tipo de Documento».

**Resultado esperado:** ves «DNI», «Carné de extranjería», «RUC» y «Pasaporte» (guardados con los códigos SUNAT 1, 4, 6 y 7).

#### TC-INV-054 — El número debe cumplir el formato de su tipo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-INV-009 «El número debe cumplir el formato de su tipo»; CU-INV-09 excepción paso 3; [RN-PAC-004][rn-b] |

**Precondiciones:** nombre, fecha de nacimiento y demás datos válidos en cada intento.
**Datos:**

| Tipo | Número | Resultado |
|---|---|---|
| DNI | `12345678` | Se guarda |
| DNI | `1234567` | «El número no corresponde al tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números).» y no se envía |
| RUC | `20123456789` | Se guarda |
| Carné de extranjería | `AB1234567890` | Se guarda (de 1 a 12 letras o números) |
| Pasaporte | `PA123456789012345678` | Se guarda (20 caracteres: de 1 a 20 letras o números) |

**Pasos:** 1. Registra un paciente por fila con documentos distintos entre sí (cambia el número de las filas guardadas cuando haga falta para que no choquen: `TC-INV-057`).

**Resultado esperado:** el de la tabla.

#### TC-INV-055 — Campos obligatorios de un paciente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-INV-009 «Campos obligatorios»; CU-INV-09 excepción paso 3; [RN-PAC-002][rn-b], [RN-PAC-003][rn-b], [RN-PAC-005][rn-b], [RN-PAC-006][rn-b] |

**Pasos:** 1. Deja vacíos el tipo, el número, el nombre y la fecha de nacimiento y «Guardar». 2. Completa todo menos «Contacto Primario» y guarda.

**Resultado esperado:** (1) «Este campo es obligatorio» debajo de cada campo y no se envía nada. (2) «Contacto Primario» es opcional: se guarda. Con un nombre de solo espacios: **Hallazgo (brecha 3):** «Error al guardar el paciente».

#### TC-INV-056 — Solo API: rechazo del servidor por tipo o formato

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-INV-009 «Rechazo del servidor por tipo o formato»; CU-INV-09 excepciones pasos 3 y 4; [RN-PAC-002][rn-b], [RN-PAC-003][rn-b], [RN-PAC-004][rn-b] |

**Precondiciones:** token con `Pacientes:Crear`.
**Datos:** `POST /api/v1/Pacientes` con (a) tipo `9`; (b) tipo `1` y número `1234567`; (c) número vacío.
**Resultado esperado:** 400 en los tres casos: (a) `CreatePacienteCommand.TipoDocumento` (en la pantalla: «Selecciona un tipo de documento válido.»); (b) formato de número; (c) `CreatePacienteCommand.DocIdentidadGub` (**Hallazgo, brecha 4:** se asocia al mensaje de formato). Los guards se evalúan en este orden: documento vacío, longitud, tipo, formato, nombre, fecha y contacto; solo se informa el primero que falla. **No verificado:** el nombre exacto de las propiedades del cuerpo.

#### TC-INV-057 — Documento de identidad repetido aunque sea de otro tipo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-INV-009 «Documento repetido»; CU-INV-09 excepción paso 5; [RN-PAC-008][rn-b] |

**Precondiciones:** existe el paciente con el documento `12345678` (DNI).
**Pasos:** 1. Registra otro paciente con el número `12345678` y tipo «Carné de extranjería». 2. Edita el paciente original sin cambiar el documento.

**Resultado esperado:** (1) 409 `Paciente.DocIdentidadGub.Duplicado` y «Ya existe un paciente con ese documento de identidad.» (2) se guarda: al editar se excluye al propio paciente.

#### TC-INV-058 — Fecha de nacimiento futura

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-INV-009 «Fecha de nacimiento futura»; CU-INV-09 excepción paso 4; [RN-PAC-006][rn-b] |

**Datos:** Fecha de Nacimiento = mañana.
**Pasos:** 1. Completa el formulario con esa fecha y «Guardar».

**Resultado esperado:** el selector no la limita; la API responde 400 `PacienteCliente.FechaNacimiento` y ves «Error al guardar el paciente» (**Hallazgo, brecha 2**).

#### TC-INV-059 — Contacto primario de más de 100 caracteres

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-INV-009 «Contacto primario de más de 100 caracteres»; CU-INV-09 excepción paso 4; [RN-PAC-007][rn-b] |

**Datos:** (a) 100 caracteres en «Contacto Primario»; (b) 120 caracteres; (c) 151 caracteres.
**Resultado esperado:** (a) se guarda; (b) la API lo rechaza y ves «Error al guardar el paciente» (**Hallazgo, pendiente 9:** el formulario admite hasta 150); (c) el campo no deja pasar de 150. El documento: el formulario admite 20 caracteres y la API 50 (brecha 1).

#### TC-INV-060 — Editar un paciente y paciente antiguo sin tipo de documento

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-009 «Editar un paciente»; «Paciente antiguo sin tipo de documento»; CU-INV-09 A1 y A2; [RN-PAC-002][rn-b], [RN-PAC-004][rn-b] |

**Precondiciones:** `Pacientes:Actualizar`; un paciente sin tipo de documento (creado antes de existir el campo: se prepara por base de datos).
**Pasos:** 1. Abre «Rosa Paciente Demo», «Editar», cambia el contacto y guarda. 2. «Editar», cambia el tipo a «RUC» sin cambiar el número. 3. Abre el paciente sin tipo y haz clic en «Editar».

**Resultado esperado:** (1) «Paciente guardado correctamente», te quedas en el detalle y los datos se recargan. (2) el número se revalida con el formato del tipo nuevo (un DNI de 8 dígitos no cumple RUC). (3) el tipo aparece vacío y debes elegirlo para poder guardar.

#### TC-INV-061 — El paciente ya no existe

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-INV-009 «El paciente ya no existe»; CU-INV-09 A1 (excepción `Paciente.NotFound`); [RN-PAC-009][rn-b] |

**Pasos:** 1. Sesión 1: edita un paciente. 2. Sesión 2: elimínalo. 3. Sesión 1: guarda y abre su enlace.

**Resultado esperado:** 404 `Paciente.NotFound`: «Error al guardar el paciente» y «Error al cargar el paciente».

### HU-INV-010 — Buscar, consultar y exportar pacientes

#### TC-INV-062 — Filtros, tabla y semántica de los filtros de pacientes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-010 «Filtros disponibles»; «Buscar»; «Filtros»; «Sin coincidencias»; CU-INV-10 flujo básico; [RN-PAC-011][rn-b] |

**Precondiciones:** pacientes de tipos DNI y RUC con distintos nombres y fechas de nacimiento.
**Pasos:** 1. Abre `/pacientes` y mira los filtros. 2. «Buscar». 3. Elige «DNI». 4. Escribe parte del documento y parte del nombre. 5. «Fecha Nacimiento Desde» y «Hasta» con las fechas exactas de nacimiento de un paciente. 6. Busca algo inexistente.

**Resultado esperado:** filtros «Tipo de Documento» («Todos», «DNI», «Carné de extranjería», «RUC», «Pasaporte»), «Documento de Identidad», «Nombre y Apellidos», «Fecha Nacimiento Desde» y «Fecha Nacimiento Hasta». Tabla «Doc. Identidad» (enlace al detalle), «Tipo Doc.» («—» si no tiene), «Nombre y Apellidos», «Fecha Nacimiento» (dd/MM/yyyy) y «Contacto Primario» («—» si no tiene), ordenada por nombre y apellidos y luego por identificador. El tipo se compara exacto; documento y nombre con «contiene»; las fechas «Desde» y «Hasta» son inclusivas (5 devuelve a ese paciente). (6) «Resultados (0)» y la fila vacía «Resultados».

#### TC-INV-063 — Exportar pacientes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-010 «Exportar»; CU-INV-10 A1; [RN-PAC-011][rn-b], [RN-IMP-008][rn-a], [RN-IMP-009][rn-a], [RN-IMP-011][rn-a] |

**Pasos:** 1. Busca y «Exportar» en español y con otro idioma.

**Resultado esperado:** se descarga `pacientes.xlsx` con «Doc. Identidad», «Tipo Doc.» (con la abreviatura «DNI», «CE», «RUC» o «PASAPORTE»), «Nombre y Apellidos», «Fecha Nacimiento» y «Contacto Primario»; fechas dd/MM/yyyy en español y MM/dd/yyyy con otro idioma. Si falla: «Error al buscar pacientes», con «Referencia: …» si la respuesta trae referencia de soporte. **No verificado en la aplicación:** el contenido del archivo.

### HU-INV-011 — Eliminar un paciente

#### TC-INV-064 — Eliminar un paciente sin registros asociados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-011 «Eliminar un paciente sin registros asociados»; CU-INV-11 flujo básico y A1; [RN-PAC-009][rn-b], [RN-PAC-010][rn-b] |

**Precondiciones:** un paciente sin recetas, ventas, citas ni inmunizaciones; `Pacientes:Eliminar`.
**Pasos:** 1. «Editar», «Eliminar»; cancela; repite y «Confirmar».

**Resultado esperado:** «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este paciente?»; al confirmar «Paciente eliminado correctamente» y vuelves a `/pacientes`; la API responde 200 con el identificador.

#### TC-INV-065 — Bloqueo al eliminar un paciente con recetas, ventas u otros registros

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-011 «Paciente con recetas, ventas u otros registros»; CU-INV-11 excepción paso 4 (`Paciente.InUse`); [RN-PAC-010][rn-b] |

**Precondiciones:** cuatro pacientes de prueba, cada uno con una sola dependencia no eliminada: P1 una receta (`TC-INV-077`); P2 una venta (parte C); P3 un servicio de agenda; P4 un servicio clínico de inmunización.
**Pasos:** 1. Intenta eliminar cada uno.

**Resultado esperado:** 409 `Paciente.InUse` (bloqueo duro; se comprueban en ese orden) y «No puedes eliminar este paciente porque tiene recetas, ventas u otros registros asociados.»; el paciente sigue existiendo. P3 y P4 se preparan por API o base de datos (módulos sin pantalla): **No verificado en la aplicación.** La pantalla muestra ese texto ante cualquier 409.

#### TC-INV-066 — Otro fallo al eliminar un paciente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Baja | HU-INV-011 «Otro fallo»; CU-INV-11 excepción paso 4 (`NotFound`); [RN-PAC-009][rn-b] |

**Pasos:** 1. Sesión 1: «Editar», «Eliminar». 2. Sesión 2: elimina el paciente. 3. Sesión 1: «Confirmar».

**Resultado esperado:** 404 con el código genérico `NotFound` y «Error al eliminar el paciente».

### HU-INV-012 — Cargar pacientes desde un Excel

#### TC-INV-067 — Columnas de la plantilla de pacientes y tipo de documento en la carga

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-012 «Columnas de la plantilla»; «El tipo de documento como código o abreviatura»; 12b §1.3 «CM-1 Descargar la plantilla»; CU-INV-12 flujo básico; [RN-PAC-012][rn-b], [RN-PAC-013][rn-b], [RN-PAC-016][rn-b] |

**Datos:** filas con `TipoDocumento` `1`, `4`, `6`, `7`, `DNI`, `ce`, `Ruc`, `PASAPORTE` y `OTRO` (cada una con documento y demás datos válidos).
**Pasos:** 1. Descarga la plantilla y súbela con esas filas.

**Resultado esperado:** `plantilla-pacientes.xlsx` con «TipoDocumento», «DocIdentidadGub», «NombreApellidos» y «FechaNacimiento» (obligatorias) y «ContactoPrimario» (opcional); límites por fila de 20, 50, 200, 10 y 100 caracteres. Las filas con código o abreviatura (sin distinguir mayúsculas) son válidas en ese campo; `OTRO` muestra «'OTRO' no es un valor válido para 'tipoDocumento'». **Hallazgo (pendiente 14):** la columna del tipo de documento se declara con la clave `fields.tipoDocumento`, que no existe en la raíz de `es.json`; el encabezado podría verse como `fields.tipoDocumento` (**No verificado en ejecución**). Los escenarios CM-1 a CM-5 se prueban como en `TC-CAT-025` a `TC-CAT-033`.

#### TC-INV-068 — Formato del número y de la fecha en la vista previa de pacientes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-012 «El número cumple el formato de su tipo»; «La fecha de nacimiento exactamente como dd/MM/yyyy»; CU-INV-12 excepción paso 3; [RN-PAC-004][rn-b], [RN-PAC-006][rn-b], [RN-PAC-014][rn-b] |

**Datos:** fila 1: tipo `DNI`, número `1234567`; fila 2: fecha `1985-03-15`; fila 3: fecha futura `01/01/2999`; fila 4: fecha `15/03/1985` válida.
**Pasos:** 1. Sube el archivo.

**Resultado esperado:** fila 1: «'docIdentidadGub' no tiene el formato de su tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números)»; fila 2: «El formato de 'fechaNacimiento' no es válido (usa DD/MM/YYYY)»; fila 3: «'fechaNacimiento' no puede ser una fecha futura»; fila 4 válida.

#### TC-INV-069 — Documento existente o repetido, campos vacíos y largos en la carga de pacientes

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-012 «Documento existente o repetido»; «Campos vacíos o demasiado largos»; CU-INV-12 excepción paso 3; [RN-PAC-003][rn-b], [RN-PAC-005][rn-b], [RN-PAC-008][rn-b], [RN-PAC-015][rn-b], [RN-PAC-017][rn-b] |

**Precondiciones:** existe el paciente con documento `12345678`.
**Datos:** filas: documento `12345678`; dos filas con `87654321` y `87654321`; documento vacío; nombre vacío; documento de 51 caracteres; nombre de 201 caracteres.
**Pasos:** 1. Sube el archivo. 2. Guarda las válidas. 3. Solo API: `POST /api/v1/Pacientes/carga-masiva` con lista vacía y con 5001 filas.

**Resultado esperado:** «El valor '12345678' del campo 'docIdentidadGub' ya existe en el sistema»; las dos de `87654321` «…está duplicado en el archivo»; «El campo 'docIdentidadGub' es obligatorio» y «El campo 'nombreApellidos' es obligatorio»; «'docIdentidadGub' no puede superar los 50 caracteres» y «'nombreApellidos' no puede superar los 200 caracteres». Si repetidas llegan al guardado, se guarda solo la primera. Lista vacía o de más de 5000: 400 `CargaMasivaPacientesCommand.Rows`.

## 17. Profesionales de salud

Catálogo de prescriptores que se eligen como médico en las recetas: número de registro, nombre del prescriptor y dirección de la clínica (no hay especialidad). El manual no recorrió esta pantalla; los textos vienen de 12b y de [19e sección 4][p19e].

### HU-INV-013 — Registrar y editar un profesional de salud

#### TC-INV-070 — Registrar un profesional de salud

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-013 «Registrar un profesional»; «Campos obligatorios»; CU-INV-13 flujo básico; [RN-PRO-001 a 004][rn-b] |

**Precondiciones:** `ProfesionalesSalud:Crear`.
**Datos:** Número de Registro `CMP-12345`; Nombre del Prescriptor `Dr. Juan Médico Demo`; Dirección de Clínica `Av. Ficticia 456, Lima`.
**Pasos:** 1. «Nuevo» en `/profesionales-salud`. 2. Guarda primero con número y nombre vacíos. 3. Completa y guarda. 4. Registra otro sin dirección.

**Resultado esperado:** (2) «Este campo es obligatorio» debajo del campo y no se envía nada. (3) «Profesional de salud guardado correctamente» y vuelves a `/profesionales-salud`; la API responde 201 con el identificador. (4) «Dirección de Clínica» es opcional: se guarda.

#### TC-INV-071 — Número de registro repetido y longitudes de un profesional

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-INV-013 «Número de registro repetido»; «Longitudes del formulario más amplias que las de la API»; CU-INV-13 excepciones pasos 4 y 5; [RN-PRO-002][rn-b], [RN-PRO-003][rn-b], [RN-PRO-004][rn-b], [RN-PRO-005][rn-b] |

**Precondiciones:** existe el profesional con número `CMP-12345`.
**Pasos:** 1. Registra otro con el mismo número. 2. Registra uno nuevo con 180 caracteres en «Nombre del Prescriptor». 3. Registra otro con 280 caracteres en «Dirección de Clínica». 4. Edita el profesional original sin cambiar el número. 5. Prueba solo espacios en el número.

**Resultado esperado:** (1) 409 `ProfesionalSalud.NumeroRegistro.Duplicado` y «Ya existe un profesional de salud con ese número de registro.» (2) y (3) **Hallazgo (pendiente 9):** el formulario admite nombre hasta 200 y dirección hasta 300, pero la API rechaza (nombre hasta 150, dirección hasta 255, número hasta 50) y ves «Error al guardar el profesional de salud». (4) se guarda (se excluye al propio profesional). (5) **Hallazgo:** «Error al guardar el profesional de salud».

#### TC-INV-072 — Editar un profesional y profesional inexistente

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-013 «Editar un profesional»; «El profesional ya no existe»; CU-INV-13 A1; [RN-PRO-005][rn-b], [RN-PRO-006][rn-b] |

**Precondiciones:** `ProfesionalesSalud:Actualizar`; dos sesiones.
**Pasos:** 1. Abre el profesional, «Editar», cambia la dirección y guarda. 2. Sesión 1 edita otro; sesión 2 lo elimina; sesión 1 guarda y abre su enlace.

**Resultado esperado:** (1) «Profesional de salud guardado correctamente», te quedas en el detalle y los datos se recargan. (2) 404 `ProfesionalSalud.NotFound`: «Error al guardar el profesional de salud» y «Error al cargar el profesional de salud».

### HU-INV-014 — Buscar y exportar profesionales de salud

#### TC-INV-073 — Buscar y exportar profesionales de salud

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-014 «Buscar»; «Sin coincidencias»; «Exportar»; CU-INV-14 flujo básico y A1; [RN-PRO-008][rn-b], [RN-IMP-008][rn-a], [RN-IMP-009][rn-a] |

**Pasos:** 1. Abre `/profesionales-salud` y «Buscar». 2. Filtra por «Número de Registro» y por «Nombre del Prescriptor». 3. Busca algo inexistente. 4. «Exportar».

**Resultado esperado:** filtros «Número de Registro» y «Nombre del Prescriptor» («contiene»); tabla «Número de Registro» (enlace al detalle), «Nombre del Prescriptor» y «Dirección de Clínica» («—» si no tiene), ordenados por nombre y luego por identificador. (3) «Resultados (0)» y la fila vacía «Resultados». (4) `profesionales-salud.xlsx` con las tres columnas y los mismos filtros; si falla: «Error al buscar profesionales de salud», con «Referencia: …» si hay referencia. **No verificado en la aplicación:** el contenido del archivo.

### HU-INV-015 — Eliminar un profesional de salud

#### TC-INV-074 — Eliminar un profesional sin recetas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-015 «Eliminar un profesional sin recetas»; CU-INV-15 flujo básico y A1; [RN-PRO-006][rn-b], [RN-PRO-007][rn-b] |

**Pasos:** 1. Abre un profesional sin recetas, «Editar», «Eliminar»; cancela; repite y «Confirmar».

**Resultado esperado:** «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este profesional de salud?»; al confirmar «Profesional de salud eliminado correctamente» y vuelves a `/profesionales-salud`; la API responde 204.

#### TC-INV-075 — Bloqueo al eliminar un profesional con recetas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-015 «Profesional con recetas»; «Otro fallo»; CU-INV-15 excepción paso 4 (`ProfesionalSalud.InUse`); [RN-PRO-007][rn-b] |

**Precondiciones:** una receta no eliminada con «Dr. Juan Médico Demo» como médico (`TC-INV-077`).
**Pasos:** 1. Elimina al profesional y confirma. 2. Elimina la receta y repite. 3. Con dos sesiones, elimina un profesional ya eliminado por la otra sesión.

**Resultado esperado:** (1) 409 `ProfesionalSalud.InUse` (bloqueo duro) y «No puedes eliminar este profesional de salud porque tiene recetas asociadas.»; sigue existiendo. (2) se elimina. (3) 404 `ProfesionalSalud.NotFound` y «Error al eliminar el profesional de salud». La pantalla muestra el texto de «recetas asociadas» ante cualquier 409 (**No verificado en la aplicación**).

### HU-INV-016 — Cargar profesionales de salud desde un Excel

#### TC-INV-076 — Carga masiva de profesionales: columnas, reglas por fila y repetidas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-016 «Columnas de la plantilla»; «Reglas por fila en la vista previa»; «Al guardar se omiten las repetidas»; 12b §1.3 «CM-1 Descargar la plantilla»; CU-INV-16 excepciones pasos 2 y 3; [RN-PRO-002 a 005][rn-b], [RN-PRO-009][rn-b], [RN-PRO-010][rn-b], [RN-PRO-011][rn-b], [RN-PRO-012][rn-b] |

**Precondiciones:** existe el profesional con número `CMP-12345`.
**Datos:** filas: (1) `CMP-12345`; (2) y (3) `CMP-99999` repetido; (4) número vacío; (5) nombre de 151 caracteres; (6) válida `CMP-77777`.
**Pasos:** 1. Descarga la plantilla y súbela con las filas. 2. «Guardar válidos». 3. Solo API: `POST /api/v1/profesionalessalud/carga-masiva` con las filas (1), (2) y (3).

**Resultado esperado:** `plantilla-profesionales-salud.xlsx` con «NumeroRegistro» (obligatoria, máximo 50), «NombrePrescriptor» (obligatoria, 150) y «DireccionClinica» (opcional, 255); la vista previa rotula la tercera columna «Dirección Clínica». (1) «El valor 'CMP-12345' del campo 'numeroRegistro' ya existe en el sistema»; (2) y (3) «…está duplicado en el archivo» en las dos; (4) «El campo 'numeroRegistro' es obligatorio»; (5) «'nombrePrescriptor' no puede superar los 150 caracteres»; (6) válida. En el paso 3 el servidor guarda solo la primera de cada número y omite las demás sin error: «Se guardaron N registros correctamente». Lista vacía o de más de 5000: 400 `CargaMasivaProfesionalesSaludCommand.Rows`.

## 18. Recetas médicas y escáner con IA

Una receta es solo su **cabecera** (paciente, médico, fecha de expedición, repeticiones máximas, número e indicaciones): no guarda los medicamentos prescritos ni tiene vigencia por fecha. El escáner con IA interpreta la foto de una receta y **no crea ni guarda nada**. Los casos del escáner son de tipo Integración: dependen de Gemini (OCR e interpretación) y de Presidio (anonimización) reales y de la clave de Gemini configurada ([integraciones][g07]); el resultado de la IA varía, así que el tester usa una imagen de receta **ficticia** (nombres, DNI y colegiatura inventados, nunca datos de un paciente real) con medicamentos conocidos y compara la estructura del resultado. El manual no recorrió estas pantallas; los textos vienen de 12b y de [19e secciones 5][p19e].

### HU-INV-017 — Registrar y editar una receta médica

#### TC-INV-077 — Registrar una receta médica

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-INV-017 «Registrar una receta»; CU-INV-17 flujo básico; [RN-REC-001 a 010][rn-c], [RN-REC-015][rn-c] |

**Precondiciones:** `Recetas:Crear`, `Pacientes:Leer` y `ProfesionalesSalud:Leer`; existen «Rosa Paciente Demo» (`12345678`) y «Dr. Juan Médico Demo» (`CMP-12345`).
**Datos:** Paciente «Rosa Paciente Demo — 12345678»; Médico «Dr. Juan Médico Demo — CMP-12345»; Fecha de Expedición hoy; Repeticiones Máximas `0`; Número de receta `  R-0001  ` (con espacios al inicio y al final).
**Pasos:** 1. En `/recetas-medicas` haz clic en «Nuevo». 2. Escribe `Rosa` en «Paciente» y elige la opción; `Juan` en «Médico» y elige la opción. 3. Completa y «Guardar». 4. Abre la receta.

**Resultado esperado:** «Receta creada correctamente» y vuelves a `/recetas-medicas`; la API responde 200 con el identificador (no 201). El número se guarda sin espacios al inicio ni al final (`R-0001`); uno vacío queda como «sin número».

#### TC-INV-078 — Paciente y médico obligatorios

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-017 «Paciente y médico obligatorios»; CU-INV-17 excepción paso 3; [RN-REC-002][rn-c], [RN-REC-003][rn-c] |

**Pasos:** 1. En «Nuevo» no elijas paciente ni médico; completa la fecha y guarda. 2. Elige paciente y escribe texto en «Médico» sin elegir opción.

**Resultado esperado:** ves «Debes seleccionar un paciente» y «Debes seleccionar un médico» y no se envía nada. Solo API: `POST /api/v1/Recetas` sin `clienteId` o sin `medicoId` responde 400 `CreateRecetaCommand.ClienteId` o `CreateRecetaCommand.MedicoId`.

#### TC-INV-079 — Fecha de expedición: obligatoria y no futura

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | HU-INV-017 «Fecha de expedición»; CU-INV-17 excepción paso 3; [RN-REC-006][rn-c], [RN-REC-007][rn-c] |

**Datos:** (a) fecha vacía; (b) fecha de mañana escrita a mano; (c) hoy; (d) una fecha de hace 5 años.
**Pasos:** 1. Guarda con cada fecha; intenta elegir mañana en el selector.

**Resultado esperado:** (a) «La fecha de expedición es requerida» y no se envía nada; (b) el selector no deja elegir una fecha posterior a hoy y escrita a mano muestra el mismo mensaje; (c) y (d) se guardan: el sistema no vence recetas por antigüedad.

#### TC-INV-080 — Solo API: fecha de expedición futura rechazada por el servidor

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-017 «Fecha futura rechazada por el servidor»; CU-INV-17 excepción paso 4; [RN-REC-007][rn-c] |

**Datos:** `POST /api/v1/Recetas` con `fechaExpedicion` del día siguiente (UTC) y paciente y médico válidos.
**Resultado esperado:** 400 `RecetaMedica.FechaExpedicion` (en la pantalla: «La fecha de expedición no puede ser posterior a hoy.»). **No verificado:** el nombre exacto de las propiedades del cuerpo.

#### TC-INV-081 — Repeticiones máximas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-INV-017 «Repeticiones máximas»; CU-INV-17 excepción pasos 3 y 4; [RN-REC-008][rn-c] |

**Datos:** repeticiones vacías, `-1`, `0` y `2`.
**Pasos:** 1. Guarda con cada valor. 2. Solo API: `POST /api/v1/Recetas` sin repeticiones y con `-1`.

**Resultado esperado:** vacío o negativo: no se envía nada y no ves ningún mensaje (**Hallazgo, brecha 2:** el campo no tiene mensaje de error); `0` y `2` se guardan. Por API, sin repeticiones valen 0; un valor negativo se rechaza (en la pantalla: «Revisa los datos ingresados de la receta.»).

#### TC-INV-082 — Número de receta: 30 caracteres, opcional y repetible

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Alta | HU-INV-017 «Número de receta»; CU-INV-17 excepción paso 3; [RN-REC-010][rn-c] |

**Datos:** número de 31 caracteres; número de 30; el mismo número `R-0001` en dos recetas distintas; número en blanco.
**Pasos:** 1. Escribe 31 caracteres. 2. Guarda con 30. 3. Registra otra receta con el mismo número. 4. Registra otra sin número.

**Resultado esperado:** el campo no deja pasar de 30 caracteres y muestra «El número de receta admite hasta 30 caracteres»; la ayuda dice «Necesario para vender receta retenida o estupefacientes.»; se guardan las recetas con 30 caracteres, con número repetido (no hay unicidad) y sin número («sin número»).

#### TC-INV-083 — Indicaciones de uso de más de 1000 caracteres

| Tipo | Prioridad | Cubre |
|---|---|---|
| Límite | Media | HU-INV-017 «Indicaciones de uso»; CU-INV-17 excepción paso 4; [RN-REC-009][rn-c] |

**Datos:** indicaciones de 1000 y de 1001 caracteres.
**Resultado esperado:** con 1000 se guarda; con 1001 la API responde 400 `CreateRecetaCommand.IndicacionesUso` y ves «Revisa los datos ingresados de la receta.»

#### TC-INV-084 — Paciente o médico eliminados mientras se registra la receta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-INV-017 «Paciente o médico que ya no existen»; CU-INV-17 excepción paso 5; [RN-REC-004][rn-c], [RN-REC-005][rn-c] |

**Pasos:** 1. Sesión 1: llena el formulario con un paciente y un médico de prueba sin guardar. 2. Sesión 2: elimina el paciente (y en otra ronda el médico). 3. Sesión 1: guarda.

**Resultado esperado:** 404 `Paciente.NotFound` o `ProfesionalSalud.NotFound`; ves «El paciente seleccionado no existe. Vuelve a seleccionarlo.» o «El médico seleccionado no existe. Vuelve a seleccionarlo.»

#### TC-INV-085 — Editar una receta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-017 «Editar una receta»; CU-INV-17 A1; [RN-REC-004][rn-c], [RN-REC-005][rn-c], [RN-REC-011][rn-c] |

**Precondiciones:** `Recetas:Actualizar`; la receta de `TC-INV-077`.
**Pasos:** 1. Ábrela, «Editar», cambia el número de receta a `R-0002` y guarda. 2. Borra el texto de «Paciente» sin elegir otra opción y guarda.

**Resultado esperado:** (1) «Receta actualizada correctamente», vuelves al modo ver con los datos recargados; paciente y médico llegan precargados como ya elegidos. (2) **No verificado en ejecución (brecha 1):** por lectura del código, cambiar el texto sin elegir otra opción no descarta la elección anterior y «Guardar» enviaría el paciente elegido antes.

#### TC-INV-086 — La receta ya no existe; otros errores al guardar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Concurrencia | Media | HU-INV-017 «La receta ya no existe»; «Otros errores al guardar»; CU-INV-17 A1 (excepción `RecetaMedica.NotFound`); [RN-REC-012][rn-c] |

**Pasos:** 1. Sesión 1: edita una receta. 2. Sesión 2: elimínala. 3. Sesión 1: guarda y luego abre su enlace.

**Resultado esperado:** 404 `RecetaMedica.NotFound` y «La receta ya no existe.»; al abrir el detalle, «Error al cargar la receta». Ante una causa sin mensaje propio: «Error al crear la receta» al crear o «Error al actualizar la receta» al editar; los errores de base de datos usan los textos comunes («Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.» y similares; **No verificado en la aplicación:** no se pueden provocar).

#### TC-INV-087 — Solo API: identificador al editar una receta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-INV-017 «Identificador al editar (solo API)»; CU-INV-17 A1 (excepción `UpdateRecetaCommand.Id`); [RN-REC-011][rn-c] |

**Datos:** `PUT /api/v1/Recetas/00000000-0000-0000-0000-000000000000` con cuerpo válido.
**Resultado esperado:** 400 `UpdateRecetaCommand.Id`.

#### TC-INV-088 — Cuántas veces se puede dispensar una receta (se aplica al vender)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-INV-017 «Cuántas veces se puede dispensar (se aplica al vender)»; CU-INV-17 A2; [RN-REC-031][rn-c] |

**Precondiciones:** caja abierta (parte C); stock vendible de `DEMO-MED-0002` (Receta Simple) y `DEMO-MED-0003` (Receta Retenida); dos recetas con «Repeticiones Máximas» `2`, ambas con número de receta; el detalle del cobro (recetas en la venta) se prueba en el documento 16c.
**Pasos:** 1. Vende `DEMO-MED-0002` con la receta de repeticiones 2 tres veces seguidas y una cuarta. 2. Vende `DEMO-MED-0003` con la otra receta dos veces. 3. Vende un producto de venta libre con cualquier receta varias veces.

**Resultado esperado:** receta simple: admite 1 + 2 = 3 dispensaciones; la cuarta se rechaza con el código `Venta.Receta.Agotada`. Receta retenida o estupefaciente: admite 1; la segunda se rechaza con el mismo código. Venta libre: sin límite. **No verificado aquí:** el texto exacto del rechazo en el punto de venta (pertenece a la parte C, documento 16c).

### HU-INV-018 — Buscar y exportar recetas médicas

#### TC-INV-089 — Filtros y botones de la búsqueda de recetas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | HU-INV-018 «Filtros disponibles»; CU-INV-18 flujo básico paso 1; [RN-REC-001][rn-c] |

**Precondiciones:** tres sesiones: `Admin`; Carlos (rol «Cajero», con Recetas Leer, Analizar con IA y Crear); un rol con solo `Recetas:Leer`.
**Pasos:** 1. Abre `/recetas-medicas` con cada una.

**Resultado esperado:** filtros «Paciente» («Buscar paciente...»), «Médico» («Buscar médico...»), «Fecha desde» y «Fecha hasta»; el botón «Escanear receta» aparece con «Recetas Médicas › Analizar con IA» y «Nuevo» con «Crear». El tercer rol no ve ninguno de los dos botones.

#### TC-INV-090 — Paciente y médico solo filtran si eliges una opción de la lista

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-INV-018 «Paciente y médico solo filtran si eliges una opción de la lista»; CU-INV-18 flujo básico paso 2; [RN-REC-016][rn-c] |

**Precondiciones:** recetas de dos pacientes.
**Pasos:** 1. Escribe `Rosa` en «Paciente» sin elegir sugerencia y «Buscar». 2. Elige «Rosa Paciente Demo — 12345678» y busca. 3. Pulsa Enter dentro del filtro de paciente.

**Resultado esperado:** (1) el filtro no se aplica (la búsqueda envía el identificador elegido, no el texto): se ven las recetas de todos los pacientes. (2) solo las de ese paciente; cada opción muestra el nombre y «— {documento}» (paciente) o «— {número de registro}» (médico). (3) Enter no ejecuta la búsqueda en estos filtros.

#### TC-INV-091 — Buscar recetas: tabla, fechas inclusivas, eliminados y sin coincidencias

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-018 «Buscar»; «Paciente o médico eliminados»; «Sin coincidencias»; CU-INV-18 flujo básico; [RN-REC-016][rn-c] |

**Precondiciones:** recetas expedidas el 28, 29 y 30/10/2026; una receta cuyo paciente fue eliminado (por base de datos si está bloqueado).
**Pasos:** 1. Elige un paciente y «Fecha desde» 29/10/2026 y busca. 2. «Fecha hasta» 29/10/2026 y «Fecha desde» 29/10/2026. 3. Busca con «Fecha desde» posterior a «Fecha hasta». 4. Busca algo inexistente.

**Resultado esperado:** la tabla muestra «Paciente» (enlace al detalle), «Médico», «Fecha de Expedición» (dd/MM/yyyy), «Repeticiones» e «Indicaciones» («—» si no tiene), ordenada por fecha de expedición, la más reciente primero; sin el número de receta. (2) devuelve la receta del 29 (inclusivo). (3) no hay validación cruzada: devuelve «Resultados (0)». Si el paciente o el médico fueron eliminados, el nombre aparece vacío. (4) «Sin resultados». **No verificado:** la interpretación de las fechas según la zona horaria.

#### TC-INV-092 — Exportar recetas médicas

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-018 «Exportar»; CU-INV-18 A1; [RN-REC-017][rn-c], [RN-IMP-008][rn-a], [RN-IMP-009][rn-a], [RN-IMP-011][rn-a] |

**Pasos:** 1. Busca y «Exportar» en español y con otro idioma.

**Resultado esperado:** se descarga `recetas-medicas.xlsx` con «Paciente», «Médico», «Fecha de Expedición», «Repeticiones» e «Indicaciones», sin el número de receta; fechas dd/MM/yyyy en español y MM/dd/yyyy con otro idioma. Si falla: «Error al buscar recetas médicas». **No verificado en la aplicación:** el contenido del archivo.

#### TC-INV-093 — Solo API: filtro de paciente con un GUID vacío y sin sugerencias por falta de permiso

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-INV-018 «Filtro de paciente con un identificador vacío (solo API)»; CU-INV-18 excepción paso 3 y paso 2; [RN-REC-016][rn-c] |

**Datos:** `GET /api/v1/Recetas?ClienteId=00000000-0000-0000-0000-000000000000`; y con un usuario con `Recetas:Leer` pero sin `Pacientes:Leer`, escribir en el filtro «Paciente».
**Resultado esperado:** 400 `GetRecetasQuery.ClienteId`; el segundo: la API de sugerencias responde 403 y no hay manejador (**No verificado en ejecución** qué ve el usuario).

### HU-INV-019 — Eliminar una receta médica

#### TC-INV-094 — Eliminar una receta que ninguna venta usa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-019 «Eliminar una receta no usada»; CU-INV-19 flujo básico y A1; [RN-REC-012][rn-c], [RN-REC-014][rn-c] |

**Pasos:** 1. Abre una receta, «Editar», haz clic en «Eliminar» al pie; cancela; repite y «Confirmar».

**Resultado esperado:** se abre «Eliminar receta» con «¿Estás seguro de que deseas eliminar esta receta médica? Esta acción no se puede deshacer.»; al confirmar «Receta eliminada correctamente» y vuelves a `/recetas-medicas`; la API responde 200 con el identificador de la receta.

#### TC-INV-095 — Bloqueo al eliminar una receta usada en una venta (incluidas anuladas)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Alta | HU-INV-019 «Receta ya usada en una venta»; CU-INV-19 excepción paso 4 (`RecetaMedica.InUse`); [RN-REC-013][rn-c] |

**Precondiciones:** receta R1 usada en una venta completada; receta R2 usada en una venta que luego se anuló (parte C).
**Pasos:** 1. Elimina R1. 2. Elimina R2.

**Resultado esperado:** en ambos casos 409 `RecetaMedica.InUse` y «No puedes eliminar esta receta porque ya se usó en ventas.»; la comprobación cuenta también las ventas anuladas (**Hallazgo, pendiente 7**). La pantalla muestra ese texto ante cualquier 409. Otras referencias (órdenes magistrales, procesamientos de IA) no se revisan: la eliminación es lógica y no dispara las claves foráneas.

#### TC-INV-096 — Otro fallo al eliminar una receta y sin permiso Eliminar

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Media | HU-INV-019 «Otro fallo»; «Sin permiso no hay botón»; CU-INV-19 excepción paso 4 (`NotFound`); [RN-REC-001][rn-c], [RN-REC-012][rn-c] |

**Precondiciones:** un rol con `Recetas:Leer` y `Actualizar` sin `Eliminar`; dos sesiones de `Admin`.
**Pasos:** 1. Con el rol sin `Eliminar` abre una receta y «Editar». 2. Sesión 1: «Editar» y «Eliminar» sobre otra receta; sesión 2 la elimina; sesión 1 «Confirmar».

**Resultado esperado:** (1) no ves «Eliminar» aunque entres en modo editar. (2) 404 con el código genérico `NotFound` y «Error al eliminar la receta».

### HU-INV-020 — Escanear una receta con IA

#### TC-INV-097 — Elegir la imagen del escáner

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Media | HU-INV-020 «Elegir la imagen»; CU-INV-20 flujo básico; [RN-REC-018][rn-c], [RN-REC-001][rn-c] |

**Precondiciones:** `Recetas:Analizar` y `Leer`; una imagen ficticia de receta `receta-demo.jpg`, un PDF `receta-demo.pdf` y dos imágenes.
**Pasos:** 1. En `/recetas-medicas` haz clic en «Escanear receta». 2. Mira la pantalla «Lector de Recetas IA». 3. Haz clic en «Galería» y elige una imagen. 4. Arrastra el PDF a la zona. 5. Arrastra dos imágenes a la vez.

**Resultado esperado:** tarjeta «Escanear receta médica» con la insignia «Análisis con IA», el subtítulo «Toma una foto de la receta o selecciona una imagen de la galería» y la zona «Captura o selecciona una imagen para comenzar» y «o arrastra una imagen aquí». Tras elegir la imagen ves su vista previa y su nombre de archivo («Tomar foto» abre la cámara trasera en un teléfono y el selector de archivos en un equipo). El PDF arrastrado se ignora sin aviso; con dos imágenes solo se usa la primera.

#### TC-INV-098 — Interpretar una receta (estado de espera)

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | HU-INV-020 «Interpretar la receta»; CU-INV-20 flujo básico; [RN-REC-018][rn-c], [RN-REC-021][rn-c] |

**Precondiciones:** Gemini y Presidio disponibles y clave de Gemini configurada; imagen ficticia con dos medicamentos legibles.
**Pasos:** 1. Elige la imagen. 2. Haz clic en «Interpretar receta». 3. Mientras espera, intenta usar «Tomar foto», «Galería» y «Limpiar».

**Resultado esperado:** ves «Interpretando...» con el anillo giratorio y los botones «Tomar foto», «Galería» y «Limpiar» quedan bloqueados. Al terminar ves los resultados (`TC-INV-105` y siguientes). **No verificado:** el tiempo de respuesta y la calidad del análisis con servicios reales (12b 19.3, número 11).

#### TC-INV-099 — Sin imagen no pasa nada; solo API sin imagen

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Baja | HU-INV-020 «Sin imagen no pasa nada»; CU-INV-20 A2 y excepción paso 4 (sin imagen); [RN-REC-018][rn-c] |

**Pasos:** 1. Abre el escáner sin elegir imagen. 2. Solo API: `POST /api/v1/Recetas/analizar` multipart sin el campo `imagen`.

**Resultado esperado:** (1) el botón «Interpretar receta» no se muestra. (2) 400 con el texto plano «Image is required.»

#### TC-INV-100 — Limpiar la imagen y los resultados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Baja | HU-INV-020 «Limpiar»; CU-INV-20 A1 |

**Precondiciones:** una imagen interpretada con resultados.
**Pasos:** 1. Haz clic en «Limpiar». 2. Interpreta de nuevo, y sin limpiar elige otro archivo.

**Resultado esperado:** (1) se quitan la imagen, la vista previa y los resultados. (2) al elegir otro archivo se limpian los resultados anteriores.

#### TC-INV-101 — Sin el permiso de análisis no hay escáner

| Tipo | Prioridad | Cubre |
|---|---|---|
| Permisos | Alta | HU-INV-020 «Sin el permiso de análisis»; CU-INV-20 excepción paso 1; [RN-REC-001][rn-c], [RN-REC-018][rn-c] |

**Precondiciones:** un rol con `Recetas:Leer` y `Crear` sin `Analizar`.
**Pasos:** 1. Abre `/recetas-medicas`. 2. Escribe `/recetas-medicas/escanear` a mano. 3. Solo API: `POST /api/v1/Recetas/analizar` con ese token.

**Resultado esperado:** no ves «Escanear receta»; al escribir la ruta ves «No tienes permisos para acceder a esta pantalla.» y vuelves al Inicio (**No verificado en la aplicación:** la redirección); por API, 403.

#### TC-INV-102 — Límite de 10 análisis por minuto y tamaño máximo de 10 MB

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Media | HU-INV-020 «Límite de peticiones y tamaño (no verificado en la aplicación)»; CU-INV-20 excepción paso 4 (429 y `Request.TooLarge`); [RN-REC-018][rn-c], [RN-GEN-028][rn-a], [RN-GEN-039][rn-a] |

**Precondiciones:** entorno propio con Gemini real o simulado; una imagen válida y otra que lleve el cuerpo por encima de 10 MB.
**Pasos:** 1. Envía 12 análisis en menos de un minuto desde la misma empresa. 2. Envía la imagen grande.

**Resultado esperado:** pasados el límite (10 por minuto, con hasta 2 en cola) la API responde 429; la imagen grande, 413 `Request.TooLarge`; en ambos casos la pantalla muestra «Error al interpretar la receta. Intenta nuevamente.» (**Hallazgo, brecha:** un único mensaje para todo fallo). **No verificado en la aplicación:** ninguno de los dos límites se probó.

#### TC-INV-103 — Hallazgo: el servidor no valida el tipo de archivo del escáner

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Media | HU-INV-020 «El servidor no valida el tipo de archivo»; CU-INV-20 flujo básico paso 4; [RN-REC-019][rn-c] |

**Datos:** `POST /api/v1/Recetas/analizar` con un archivo de texto (o PDF) en el campo `imagen`.
**Resultado esperado:** **Hallazgo:** el servidor no valida el tipo ni el contenido: lo lee entero y lo envía a Gemini declarándolo siempre como `image/jpeg`; no responde un 400 por tipo. El resultado depende de Gemini (**No verificado**). El SPA solo acepta archivos de tipo imagen (`image/*`) y no limita el tamaño.

#### TC-INV-104 — Solo API: especialidad de contexto

| Tipo | Prioridad | Cubre |
|---|---|---|
| Negativo | Media | HU-INV-020 «Especialidad de contexto (solo API)»; CU-INV-20 A3; [RN-REC-020][rn-c] |

**Datos:** `POST /api/v1/Recetas/analizar` con `especialidadContexto` de (a) `Cardiología`; (b) 51 letras; (c) `Cardio 123`; (d) omitida.
**Resultado esperado:** (a) válida (solo letras, incluidas tildes, y espacios); (b) y (c) 400 `AnalizarRecetaQuery.EspecialidadContexto`; (d) usa «Medicina General». La pantalla nunca envía este dato.

### HU-INV-021 — Interpretar el resultado del escaneo: DIGEMID, confianza, stock y equivalentes

#### TC-INV-105 — Medicamento detectado y validado por DIGEMID

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | HU-INV-021 «Medicamento detectado y validado»; CU-INV-21 flujo básico; [RN-REC-023][rn-c], [RN-REC-024][rn-c] |

**Precondiciones:** el catálogo DIGEMID tiene «PARACETAMOL 500 MG TABLETA (DEMO)»; el catálogo de la empresa tiene «Paracetamol Demo 500 mg»; imagen ficticia donde la IA detecta «Paracetamol 500 mg».
**Pasos:** 1. Interpreta la receta. 2. Mira la sección «Medicamentos de la receta (N)».

**Resultado esperado:** una tarjeta con el nombre oficial del medicamento de tu empresa que mejor coincide; «Detectado como: "Paracetamol 500 mg"» si el texto detectado difiere del nombre oficial; la insignia «DIGEMID» y la razón «Validado por DIGEMID (nombre)»; «Cantidad sugerida: N» si la IA leyó una cantidad. **No verificado en la aplicación:** el resultado con Gemini real.

#### TC-INV-106 — Cómo se valida contra DIGEMID; catálogo vacío

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | HU-INV-021 «Cómo se valida contra DIGEMID»; CU-INV-21 A1; [RN-REC-023][rn-c] |

**Precondiciones:** (a) catálogo DIGEMID con el producto; (b) catálogo DIGEMID vacío (empresa nueva).
**Pasos:** 1. Interpreta la receta de `TC-INV-105` en (a). 2. Repite en (b). 3. Usa una receta con un medicamento que no está en el catálogo DIGEMID.

**Resultado esperado:** en (a) se contrastan todas las palabras del nombre; si no hay resultado se reintenta con la primera palabra; se ordenan hasta 50 candidatos por parecido y se toman los 5 mejores. En (b) y en el paso 3 el análisis funciona pero la razón es «Detectado en receta» y no aparece la insignia «DIGEMID».

#### TC-INV-107 — Coincidencia con el catálogo de medicamentos de la empresa

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Media | HU-INV-021 «Coincidencia con el catálogo de mi empresa»; CU-INV-21 A2; [RN-REC-024][rn-c] |

**Precondiciones:** una receta con un medicamento que existe en DIGEMID pero no en tu catálogo de empresa.
**Pasos:** 1. Interpreta la receta.

**Resultado esperado:** se consideran hasta 50 medicamentos cuyo nombre contenga la primera palabra y se acepta el mejor solo si su parecido es mayor a 0,4; si no hay coincidencia el resultado no trae producto y aparece con el nombre DIGEMID o el texto detectado.

#### TC-INV-108 — Rótulo de confianza

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Media | HU-INV-021 «Rótulo de confianza»; CU-INV-21 flujo básico; [RN-REC-025][rn-c] |

**Datos:**

| Confianza final | Rótulo |
|---|---|
| 0,90 | Confianza: Alta |
| 0,85 | Confianza: Alta |
| 0,70 | Confianza: Media |
| 0,65 | Confianza: Media |
| 0,50 | Confianza: Baja |

**Pasos:** 1. Interpreta recetas con medicamentos de claridad distinta. 2. Compara el rótulo con la tabla.

**Resultado esperado:** el rótulo corresponde al valor de la tabla. **No verificado:** el tester no puede fijar la confianza que devuelve la IA ni ver el número; la tabla solo se puede comprobar si el entorno permite simular la respuesta de Gemini, y de lo contrario queda como verificación de código.

#### TC-INV-109 — Hallazgo: cómo se calcula la confianza

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Media | HU-INV-021 «Cómo se calcula la confianza»; CU-INV-21 flujo básico; [RN-REC-025][rn-c] |

**Datos:** dos medicamentos con el mismo nivel de confianza de la IA (por ejemplo 0,8): uno validado por DIGEMID y otro que no está en el catálogo.
**Resultado esperado:** la confianza final es nivel × 0,6 + parecido × 0,4, redondeada a 2 decimales, con «parecido» 0,9 si hubo coincidencia DIGEMID y 1,0 si no la hubo: 0,8 × 0,6 + 0,9 × 0,4 = 0,84 (validado) y 0,8 × 0,6 + 1,0 × 0,4 = 0,88 (no validado). **Hallazgo (pendiente 15, hallazgo 5 de 01c1):** el no encontrado en DIGEMID queda con confianza mayor que el validado. **No verificado en la aplicación:** la pantalla muestra el rótulo, no el número.

#### TC-INV-110 — Stock en mi sucursal en el resultado del escaneo

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | HU-INV-021 «Stock en mi sucursal»; CU-INV-21 flujo básico; [RN-REC-026][rn-c], [RN-STK-017][rn-c] |

**Precondiciones:** el usuario es de «Sede Principal»; medicamento P1 con un lote vigente de 40 unidades; P2 con un lote vencido de 20; P3 con un lote retenido en cuarentena de 15 y uno vigente de 10; P4 sin lotes.
**Pasos:** 1. Interpreta una receta que contenga los cuatro medicamentos. 2. Repite con una sesión sin sucursal válida (**No verificado** cómo lograrlo).

**Resultado esperado:** P1: «En stock: 40 unidades» (sin decimales). P2 y P4: «Sin stock». P3: «En stock: 25 unidades» (a diferencia del stock vendible, no se descuentan los lotes retenidos en cuarentena: el lote vigente de 10 más el retenido de 15). Si la sesión no trae una sucursal válida, el stock es 0.

#### TC-INV-111 — Equivalentes terapéuticos

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | HU-INV-021 «Equivalentes terapéuticos»; CU-INV-21 flujo básico; [RN-REC-027][rn-c] |

**Precondiciones:** el prescrito P1 y otros dos medicamentos E1 (con stock) y E2 (sin stock) con formulación clínica de la misma sustancia, concentración y unidad; P1 sin stock.
**Pasos:** 1. Interpreta la receta con P1.

**Resultado esperado:** ves «Equivalentes terapéuticos:» con una fila por cada otro medicamento de tu empresa con la misma formulación (nombre comercial, «— {laboratorio}» y su stock) y, como P1 no tiene stock pero E1 sí, «No hay stock del medicamento prescrito, pero hay equivalentes disponibles.»

#### TC-INV-112 — Sugerencias de la IA

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Media | HU-INV-021 «Sugerencias de la IA»; CU-INV-21 flujo básico; [RN-REC-028][rn-c] |

**Pasos:** 1. Interpreta una receta para la que la IA proponga complementarios o genéricos, algunos existentes en tu catálogo y otros no.

**Resultado esperado:** solo se muestran las sugerencias que coinciden con el catálogo de tu empresa, en «Sugerencias IA (N)» con la razón «Sugerido por {medicamento}» y su stock, sin insignias ni equivalentes; las demás se descartan.

#### TC-INV-113 — Límites del resultado: 20 medicamentos y 3 sugerencias

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Baja | HU-INV-021 «Límites del resultado»; CU-INV-21 flujo básico; [RN-REC-022][rn-c] |

**Precondiciones:** una imagen con más de 20 medicamentos (o una simulación de la respuesta de Gemini).
**Pasos:** 1. Interpreta la receta.

**Resultado esperado:** solo se procesan los primeros 20 medicamentos y 3 sugerencias por cada uno; el resto se descarta sin aviso. **No verificado:** requiere una imagen o respuesta simulada con esa cantidad.

#### TC-INV-114 — Sin medicamentos detectados

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Media | HU-INV-021 «Sin medicamentos detectados»; CU-INV-21 excepción paso 1; [RN-REC-021][rn-c] |

**Pasos:** 1. Interpreta una imagen sin texto (una foto de un paisaje) o borrosa.

**Resultado esperado:** ves «No se detectaron medicamentos en la imagen. Intenta con una foto más nítida.» (la IA no extrae texto o responde algo que no es el formato esperado; la API responde 200 con lista vacía).

### HU-INV-022 — Proteger los datos del paciente y entender qué guarda el análisis

#### TC-INV-115 — Las tres etapas del análisis y la anonimización

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | HU-INV-022 «Las tres etapas del análisis»; CU-INV-22 flujo básico; [RN-REC-021][rn-c] |

**Precondiciones:** Gemini y Presidio disponibles; una receta ficticia con un nombre de paciente y un DNI inventados bien visibles.
**Pasos:** 1. Interpreta la receta. 2. Observa el tráfico saliente hacia Gemini (proxy de depuración) o los registros del servicio Presidio, según la [guía del desarrollador][g08].

**Resultado esperado:** Gemini extrae el texto de la imagen (OCR); Presidio lo anonimiza; Gemini interpreta solo el texto anonimizado y devuelve los medicamentos. El nombre y el DNI de la receta no deben viajar en la segunda llamada a Gemini. **No verificado:** cómo observar el texto anonimizado en el entorno local (no hay pantalla); queda como verificación de entorno.

#### TC-INV-116 — Si la anonimización falla, el análisis se aborta

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Alta | HU-INV-022 «Si la anonimización falla, el análisis se aborta»; CU-INV-22 excepción paso 3; [RN-REC-021][rn-c], [RN-GEN-028][rn-a] |

**Precondiciones:** entorno propio del tester donde puedas detener el servicio Presidio (nunca en un entorno compartido).
**Pasos:** 1. Detén Presidio. 2. Interpreta una receta.

**Resultado esperado:** el análisis se aborta, el texto del paciente no llega a la IA, la API responde 500 con una referencia de soporte (`traceId`) y ves «Error al interpretar la receta. Intenta nuevamente.»

#### TC-INV-117 — Fallas del servicio de IA y respuesta que no es el formato esperado

| Tipo | Prioridad | Cubre |
|---|---|---|
| Integración | Media | HU-INV-022 «Fallas del servicio de IA»; «Respuesta de la IA que no es el formato esperado»; CU-INV-22 excepciones pasos 2, 4; [RN-REC-021][rn-c] |

**Precondiciones:** entorno propio: (a) sin la clave de la API de Gemini; (b) Gemini simulado que responde texto que no es el JSON esperado.
**Pasos:** 1. Interpreta una receta en (a). 2. Repite en (b).

**Resultado esperado:** (a) la API responde 500 con `traceId` y ves «Error al interpretar la receta. Intenta nuevamente.» (también ante un fallo de red o de Gemini o Presidio). (b) el resultado es una lista vacía sin error y ves «No se detectaron medicamentos en la imagen. Intenta con una foto más nítida.»

#### TC-INV-118 — El análisis no guarda nada

| Tipo | Prioridad | Cubre |
|---|---|---|
| Positivo | Alta | HU-INV-022 «El análisis no guarda nada»; «Las tablas de recetas digitalizadas no se usan»; CU-INV-22 flujo básico; [RN-REC-029][rn-c], [RN-REC-030][rn-c] |

**Precondiciones:** anota cuántas recetas hay en `/recetas-medicas` (sin filtros).
**Pasos:** 1. Interpreta una receta con resultados. 2. Busca las recetas de nuevo. 3. Mira la pantalla del escáner buscando botones para crear la receta o pasar los medicamentos a la venta.

**Resultado esperado:** **Hallazgo (pendiente 15):** no se crea ninguna receta (el número no cambia), no se registra el procesamiento ni se genera evento de auditoría, y la pantalla no tiene botones para crear la receta ni para pasar los medicamentos a la venta; para que la receta cuente en una venta debes registrarla aparte en «Recetas Médicas › Nuevo» (`TC-INV-077`). Las tablas de recetas digitalizadas y de registro de privacidad existen en el esquema pero ningún código las escribe (**No verificado en la aplicación:** se deduce del código leído en 01c1).

## 19. Pendientes y «No verificado»

### 19.1 En curso (sin commitear)

Ninguno que afecte a este documento. La cabecera usa la línea base vigente de [REGLAS-AGENTES](../../_inventario/REGLAS-AGENTES.md) (`sofia-apis` `43f5529`, `sofia-web-spa` `eeffbb6`), la misma de las historias 12b. **No verificado:** no se ejecutó `git status` ni `git log` (las instrucciones de esta parte prohíben todo comando git), de modo que no se comprobó si hay commits o cambios locales posteriores; las historias 12b dicen que no había cambios sin commitear el 2026-10-26.

### 19.2 Alcance y trazabilidad

| # | Asunto | Detalle |
|---|---|---|
| 1 | Transferencias entre sucursales | No están en las historias 12b ni en los casos de uso 13b: son HU-OPS-001 a 005 de la [parte C][hu-c] (módulo «Solo API») y sus casos están en el documento 16c. Aquí solo se cubre el stock que las transferencias modifican (ingreso, ajuste y consulta, sección 15). Un caso de integración entre transferencias y stock por lote queda a cargo de 16c |
| 2 | Conteo de escenarios | 12b trae 401 escenarios dentro de las 66 historias y 15 escenarios comunes en las secciones 1.2 y 1.3 (416 en total, el «unos 414» del encargo). Los 416 tienen caso |
| 3 | Capítulo 5 del manual | El archivo `capitulo-05-productos-y-proveedores.md` está a medias y sin validar: no se usó como fuente. Los medicamentos, jerarquías, presentaciones, formulaciones, proveedores, aseguradoras, lotes, stock, pacientes, profesionales y recetas se escribieron desde 12b, 13b, 01b, 01c1 y 19c a 19e, sin recorrido en la aplicación (12b 19.3, número 4) |
| 4 | Comprobaciones en el código | Puntuales y de solo lectura: los controladores y manejadores de ingreso y ajuste de stock (200 en vez de 201, suma o reemplazo, 403 `Inventario.Sucursal.NoPermitida`, 409 `InventarioSucursal.RegistroConcurrente`), el evento `STOCK_AJUSTAR` (`AuditEventos.cs`, `AdjustStockCommand.cs`, `RegisterInventarioCommand.cs`), el bloqueo `LoteInventario.InUse`, el texto «Image is required.» del escáner y el código `Venta.Receta.Agotada`. Coinciden con las historias; no se encontró ninguna contradicción |

### 19.3 No verificado

| # | Qué | Casos | Por qué |
|---|---|---|---|
| 1 | Contenido de los archivos exportados y de las plantillas de carga masiva | `TC-CAT-016`, `025`, `050`, `072`, `106`, `182`, `194`, `TC-INV-018`, `048`, `063`, `073`, `092` | El navegador integrado no muestra descargas; solo se comprobó el 200 (manual cap. 4, sección 12) |
| 2 | Redirección al Inicio por falta de permiso de ruta | `TC-CAT-002`, `195`, `TC-INV-027`, `101` | El administrador tiene todos los permisos; el manual no pudo probarlo con el cajero |
| 3 | Bloqueos por uso al eliminar (unidades, laboratorios, ingredientes, medicamentos, presentaciones, proveedores, pacientes, profesionales, lotes, recetas) | `TC-CAT-022`, `074`, `089`, `111`, `141`, `161`, `TC-INV-021`, `065`, `075`, `095` | El manual los dejó pendientes y la pantalla no se recorrió; vienen de 01b, 01c1 y 19c a 19e |
| 4 | Avisos de base de datos («Ya existe un registro…», «El registro hace referencia…», «fuera del rango permitido») | `TC-CAT-203` | No se pudieron provocar |
| 5 | Comparación de códigos y nombres únicos sin distinguir mayúsculas en todas las bases | `TC-CAT-005`, `065`, `099`, `TC-INV-007` | La intercalación no está definida en los scripts DbUp |
| 6 | Redondeo o truncamiento de valores con más decimales que la columna | `TC-CAT-037`, `097`, `155`, `TC-INV-029` | No se ejecutó |
| 7 | Multiplicador con coma decimal en la carga de jerarquías | `TC-CAT-057` | No se ejecutó |
| 8 | Nombres exactos de las propiedades de los cuerpos JSON en los casos «Solo API» | `TC-CAT-010`, `031`, `200`, `TC-INV-030`, `036`, `080` | Se deducen de los formularios; no se llamó a la API |
| 9 | Registros que solo se logran por base de datos (lote vencido, retenido, medicamento con laboratorio eliminado, paciente sin tipo, reclamos de aseguradora) | `TC-CAT-105`, `184`, `TC-INV-011`, `022`, `032`, `060` | No hay forma de crearlos desde las pantallas |
| 10 | Concurrencia real (dos precios abiertos, dos ingresos simultáneos, dos altas del mismo lote) | `TC-CAT-173`, `TC-INV-007`, `033`, `034` | Requiere pruebas de carrera; la ventana es estrecha |
| 11 | Escáner con Gemini y Presidio reales: calidad, límites 429 y 413, rótulo de confianza, texto anonimizado, 20 medicamentos | `TC-INV-098`, `102`, `103`, `108`, `113`, `115` | Requiere servicios externos; el resultado de la IA varía |
| 12 | Usuario sin sucursal en la sesión | `TC-CAT-122`, `TC-INV-110` | No se sabe cómo obtener un token sin sucursal en el entorno local |
| 13 | Interpretación de las fechas de búsqueda de recetas según la zona horaria; qué ve un usuario sin `Leer` de pacientes o profesionales en las sugerencias | `TC-INV-091`, `093` | Se deduce del código; no se probó |
| 14 | Stock del detalle del medicamento: si excluye lotes vencidos o sucursales no permitidas | `TC-CAT-107` | La regla solo dice «suma de la cantidad física» |
| 15 | Cálculo en sentido inverso de las equivalencias (por ejemplo «Unidad a Caja = 0,01») | `TC-CAT-045` | No se revisó el cálculo en el código |
| 16 | Recálculo de las presentaciones al cambiar una jerarquía | `TC-CAT-131` | Por lo leído no se recalcula; no se ejecutó |

### 19.4 Hallazgos que los casos fijan con el comportamiento actual

| # | Hallazgo (pendiente de 12b 19.2 o brecha de la historia) | Casos |
|---|---|---|
| 1 | Mensajes genéricos: la mayoría de los rechazos del servidor (validación, `*.NotFound`, solo espacios) se ven como el aviso genérico de guardado | `TC-CAT-011`, `040`, `098`, `154`, `177`, `TC-INV-003`, `004`, `055` |
| 2 | Aseguradoras: ningún error se traduce, el código repetido es un error genérico, la carga masiva no controla duplicados y el mensaje de longitud dice 200 en vez de 150 | `TC-CAT-178`, `187`, `188` |
| 3 | Vista previa que no anticipa lo que el guardado descarta: conversiones, código de laboratorio, DCI, razón social; el aviso puede decir «Se guardaron 0 registros» | `TC-CAT-031`, `058`, `078`, `093`, `165` |
| 4 | Ingredientes: la vista previa valida el ATC y no la DCI | `TC-CAT-092` |
| 5 | DIGEMID: la vista previa no valida longitudes y el guardado rechaza todo el archivo; no hay pantalla para crear, editar o eliminar | `TC-CAT-193`, `199`, `202` |
| 6 | Medicamentos cargados en masa quedan sin precio de venta base | `TC-CAT-117`, `101` |
| 7 | Eliminaciones sin validación de dependencias: jerarquías, formulaciones, aseguradoras, productos DIGEMID, unidad usada solo por presentaciones, lote usado en otros módulos; la receta cuenta también las ventas anuladas | `TC-CAT-023`, `052`, `150`, `184`, `202`, `TC-INV-022`, `095` |
| 8 | Las pantallas tratan cualquier 409 como el caso concreto («en uso», «duplicado») | `TC-CAT-022`, `111`, `128`, `145`, `161`, `TC-INV-013`, `021`, `065` |
| 9 | Diferencias de longitud entre formulario y API: proveedor, paciente, profesional | `TC-CAT-154`, `TC-INV-059`, `071` |
| 10 | Lotes: el aviso de duplicado casi nunca aparece, no se puede editar un lote vencido, la fabricación contra la caducidad solo se valida al crear, sin indicador de vencido | `TC-INV-007`, `011`, `012`, `018` |
| 11 | Stock: guardar sin lote no muestra nada, los textos obligatorios no se pintan, se puede ingresar stock de lote vencido o retenido, ajustar una sucursal no permitida responde 403 y consultarla 404 | `TC-INV-032`, `035`, `040` |
| 12 | Precios de proveedor: no se editan ni eliminan (el proveedor nunca se podrá eliminar), plazo y cantidad mínima fijos, sin índice único | `TC-CAT-161`, `172`, `173` |
| 13 | Presentaciones: «Exportar» ignora el producto elegido y borrar el texto no lo descarta | `TC-CAT-139` |
| 14 | Pacientes: la carga masiva declara `fields.tipoDocumento`, que no existe en la raíz de `es.json` | `TC-INV-067` |
| 15 | Recetas y escáner: la receta no tiene líneas ni vigencia; el escáner no guarda nada; un único mensaje de error; con la misma confianza un no validado queda mayor que un validado | `TC-INV-102`, `109`, `118` |
| 16 | Stock de un medicamento (solo API) suma toda la empresa, incluidos lotes vencidos y filas en 0 | `TC-INV-050` |
| 17 | El servidor del escáner no valida el tipo de archivo | `TC-INV-103` |
| 18 | Código HTTP de éxito no uniforme (200 en lugar de 201 o 204) en proveedor, aseguradora, paciente, receta e ingreso de stock | `TC-CAT-153`, `160`, `176`, `183`, `TC-INV-025`, `052`, `064`, `077`, `094` |

## 20. Tabla de cobertura

### 20.1 Resumen

| Módulo | Casos | Alta | Media | Baja |
|---|---|---|---|---|
| `TC-CAT` (catálogos, HU-CAT-001 a 044) | 203 | 51 | 115 | 37 |
| `TC-INV` (inventario y atención, HU-INV-001 a 022) | 118 | 47 | 59 | 12 |
| Total | 321 | 98 | 174 | 49 |

Por tipo: Positivo 126, Negativo 100, Límite 34, Concurrencia 30, Permisos 15 e Integración 16 (todos de `TC-INV`, escáner de recetas). Coberturas comprobadas con un script de solo lectura (`16b-verify.js` en el scratchpad) sobre este documento y las fuentes: 401 de 401 escenarios de las historias, 15 de 15 escenarios comunes, 66 de 66 historias, 66 de 66 casos de uso, 275 de 275 reglas del alcance (`UDM`, `JER`, `LAB`, `ING`, `MED`, `PRE`, `FOR`, `PRV`, `ASE`, `DIG`, `PAC`, `PRO`, `LOT`, `STK`, `REC`), cero `RN-`, `HU-`, `CU-` ni `TC-` inexistentes, bloques de código cerrados, todos los enlaces definidos y existentes y numeración de casos sin saltos.

### 20.2 Escenarios de cada historia y casos de uso

Los escenarios se numeran en el orden en que aparecen en la historia de 12b; `n:NNN` significa que el escenario n está cubierto por el caso `TC-CAT-NNN` o `TC-INV-NNN` (según el prefijo de la historia); `+` junta varios casos. Cada caso de uso (`CU-CAT-nn` o `CU-INV-nn`, mismo número que la historia) está citado por los mismos casos que su historia.

| Historia / caso de uso | Escenarios | Escenario → caso |
|---|---|---|
| HU-CAT-001 / CU-CAT-01 | 10 | 1:001 2:002 3:003 4:004 5:005 6:006 7:007 8:008 9:009 10:010 |
| HU-CAT-002 / CU-CAT-02 | 7 | 1:012 2:013 3:014 4:015 5:016 6:017 7:018 |
| HU-CAT-003 / CU-CAT-03 | 5 | 1:021 2:021 3:022+023 4:020 5:024 |
| HU-CAT-004 / CU-CAT-04 | 7 | 1:025 2:027 3:029 4:030 5:031 6:032 7:032 |
| HU-CAT-005 / CU-CAT-05 | 12 | 1:034 2:035 3:036 4:037 5:038 6:039 7:040 8:041 9:042 10:043 11:044 12:045 |
| HU-CAT-006 / CU-CAT-06 | 6 | 1:046 2:047 3:048 4:049 5:049 6:050 |
| HU-CAT-007 / CU-CAT-07 | 4 | 1:051 2:052 3:053 4:053 |
| HU-CAT-008 / CU-CAT-08 | 4 | 1:054 2:055 3:056+057+059 4:058 |
| HU-CAT-009 / CU-CAT-09 | 4 | 1:060 2:061 3:061 4:061 |
| HU-CAT-010 / CU-CAT-10 | 9 | 1:062 2:063 3:064 4:065 5:066 6:067 7:068 8:069 9:070 |
| HU-CAT-011 / CU-CAT-11 | 3 | 1:071 2:071 3:072 |
| HU-CAT-012 / CU-CAT-12 | 3 | 1:073 2:074 3:075 |
| HU-CAT-013 / CU-CAT-13 | 3 | 1:076 2:077+079 3:078 |
| HU-CAT-014 / CU-CAT-14 | 7 | 1:080 2:081 3:082 4:083 5:084 6:085 7:085 |
| HU-CAT-015 / CU-CAT-15 | 3 | 1:086 2:087 3:087 |
| HU-CAT-016 / CU-CAT-16 | 3 | 1:088 2:089 3:090 |
| HU-CAT-017 / CU-CAT-17 | 4 | 1:091 2:091 3:092 4:093 |
| HU-CAT-018 / CU-CAT-18 | 13 | 1:094+095 2:095 3:096 4:096 5:097 6:098 7:099 8:099 9:100 10:101 11:101 12:102 13:102 |
| HU-CAT-019 / CU-CAT-19 | 7 | 1:103 2:103 3:104 4:105 5:105 6:106 7:106 |
| HU-CAT-020 / CU-CAT-20 | 4 | 1:107 2:108 3:108 4:109 |
| HU-CAT-021 / CU-CAT-21 | 4 | 1:110 2:111 3:112 4:113 |
| HU-CAT-022 / CU-CAT-22 | 4 | 1:114+118 2:115 3:116 4:117 |
| HU-CAT-023 / CU-CAT-23 | 4 | 1:119 2:120 3:121 4:122 |
| HU-CAT-024 / CU-CAT-24 | 13 | 1:123 2:124 3:125 4:126 5:127 6:127 7:128 8:129 9:129 10:130 11:130 12:131 13:132 |
| HU-CAT-025 / CU-CAT-25 | 10 | 1:133 2:133 3:133 4:134 5:135 6:136 7:136 8:137 9:137 10:133 |
| HU-CAT-026 / CU-CAT-26 | 4 | 1:138 2:138 3:138 4:139 |
| HU-CAT-027 / CU-CAT-27 | 3 | 1:140 2:141 3:141 |
| HU-CAT-028 / CU-CAT-28 | 10 | 1:142 2:143 3:143 4:144 5:145 6:146 7:146 8:147 9:147 10:148 |
| HU-CAT-029 / CU-CAT-29 | 3 | 1:149 2:149 3:149 |
| HU-CAT-030 / CU-CAT-30 | 3 | 1:150 2:150 3:151 |
| HU-CAT-031 / CU-CAT-31 | 2 | 1:152 2:152 |
| HU-CAT-032 / CU-CAT-32 | 9 | 1:153 2:153 3:154 4:155 5:156 6:156 7:156 8:157 9:157 |
| HU-CAT-033 / CU-CAT-33 | 6 | 1:158 2:158 3:158 4:158 5:159 6:159 |
| HU-CAT-034 / CU-CAT-34 | 3 | 1:160 2:161 3:162 |
| HU-CAT-035 / CU-CAT-35 | 4 | 1:163 2:164 3:164 4:165 |
| HU-CAT-036 / CU-CAT-36 | 12 | 1:166 2:166 3:167+173 4:168 5:169 6:169 7:170 8:170 9:171 10:171 11:172 12:172 |
| HU-CAT-037 / CU-CAT-37 | 7 | 1:174 2:174 3:175 4:175 5:175 6:175 7:175 |
| HU-CAT-038 / CU-CAT-38 | 6 | 1:176 2:177 3:178 4:179 5:180 6:180 |
| HU-CAT-039 / CU-CAT-39 | 4 | 1:181 2:181 3:181 4:182 |
| HU-CAT-040 / CU-CAT-40 | 3 | 1:183 2:184 3:185 |
| HU-CAT-041 / CU-CAT-41 | 6 | 1:186 2:187 3:187 4:188 5:189 6:189 |
| HU-CAT-042 / CU-CAT-42 | 8 | 1:190 2:191 3:191 4:191 5:192 6:193 7:194 8:195 |
| HU-CAT-043 / CU-CAT-43 | 4 | 1:196 2:197 3:198 4:199 |
| HU-CAT-044 / CU-CAT-44 | 5 | 1:200 2:201 3:201 4:202 5:202 |
| HU-INV-001 / CU-INV-01 | 15 | 1:001 2:002 3:003 4:004 5:005 6:006 7:007 8:008 9:009 10:010 11:011 12:012 13:013 14:013 15:014 |
| HU-INV-002 / CU-INV-02 | 7 | 1:015 2:015 3:015 4:016 5:017 6:017 7:018 |
| HU-INV-003 / CU-INV-03 | 4 | 1:020 2:021+022 3:023 4:019 |
| HU-INV-004 / CU-INV-04 | 2 | 1:024 2:024 |
| HU-INV-005 / CU-INV-05 | 11 | 1:025 2:026 3:027 4:028 5:029 6:030 7:031 8:032 9:033 10:034 11:026 |
| HU-INV-006 / CU-INV-06 | 8 | 1:037 2:038 3:039 4:039 5:040 6:041 7:042 8:042 |
| HU-INV-007 / CU-INV-07 | 8 | 1:043 2:043 3:044 4:045 5:046 6:047 7:048 8:048 |
| HU-INV-008 / CU-INV-08 | 3 | 1:049 2:050 3:051 |
| HU-INV-009 / CU-INV-09 | 11 | 1:052 2:053 3:054 4:055 5:056 6:057 7:058 8:059 9:060 10:060 11:061 |
| HU-INV-010 / CU-INV-10 | 5 | 1:062 2:062 3:062 4:062 5:063 |
| HU-INV-011 / CU-INV-11 | 3 | 1:064 2:065 3:066 |
| HU-INV-012 / CU-INV-12 | 6 | 1:067 2:067 3:068 4:068 5:069 6:069 |
| HU-INV-013 / CU-INV-13 | 6 | 1:070 2:070 3:071 4:071 5:072 6:072 |
| HU-INV-014 / CU-INV-14 | 3 | 1:073 2:073 3:073 |
| HU-INV-015 / CU-INV-15 | 3 | 1:074 2:075 3:075 |
| HU-INV-016 / CU-INV-16 | 3 | 1:076 2:076 3:076 |
| HU-INV-017 / CU-INV-17 | 13 | 1:077 2:078 3:079 4:080 5:081 6:082 7:083 8:084 9:085 10:086 11:086 12:087 13:088 |
| HU-INV-018 / CU-INV-18 | 7 | 1:089 2:090 3:091 4:091 5:091 6:092 7:093 |
| HU-INV-019 / CU-INV-19 | 4 | 1:094 2:095 3:096 4:096 |
| HU-INV-020 / CU-INV-20 | 8 | 1:097 2:098 3:099 4:100 5:101 6:102 7:103 8:104 |
| HU-INV-021 / CU-INV-21 | 10 | 1:105 2:106 3:107 4:108 5:109 6:110 7:111 8:112 9:113 10:114 |
| HU-INV-022 / CU-INV-22 | 6 | 1:115 2:116 3:117 4:117 5:118 6:118 |

### 20.3 Escenarios comunes de 12b (secciones 1.2 y 1.3)

| Escenario común | Caso |
|---|---|
| Los resultados aparecen solo después de buscar | `TC-CAT-012` |
| Limpiar y volver conservan o descartan los filtros | `TC-CAT-019` |
| Paginación | `TC-CAT-019` |
| Sin coincidencias | `TC-CAT-015` |
| Modo ver, editar y crear en el detalle | `TC-CAT-020` |
| Eliminar pide confirmación | `TC-CAT-021` |
| La pantalla decide el aviso de eliminación solo por el estado HTTP | `TC-CAT-024` |
| Valores de solo espacios | `TC-CAT-011` |
| Errores de base de datos poco frecuentes | `TC-CAT-203` |
| Sin permiso de ruta | `TC-CAT-002` |
| CM-1 Descargar la plantilla | `TC-CAT-025` y, por módulo, `054`, `114`, `186`, `196`, `TC-INV-067`, `TC-INV-076` |
| CM-2 Rechazo del archivo | `TC-CAT-026` |
| CM-3 Vista previa sin guardar nada | `TC-CAT-027`, `028`, `056` |
| CM-4 Una fila repetida en el archivo marca todas sus apariciones | `TC-CAT-027` (y los casos de cada módulo con su campo único) |
| CM-5 Guardar solo las filas válidas | `TC-CAT-029`, `033`, `059` |

### 20.4 Reglas del alcance

Cada regla con comportamiento comprobable está citada en la línea «Cubre» de al menos un caso. La tabla da la regla y los primeros casos que la comprueban; los de `CAT` llevan el prefijo `TC-CAT-` y los de `INV` el prefijo `TC-INV-`. Las reglas de los prefijos `PAC`, `PRO`, `LOT`, `STK` y `REC` van con prefijo `TC-INV-`, salvo `RN-STK-017`, que se comprueba en `TC-CAT-107`, `TC-CAT-119`, `TC-CAT-120`, `TC-INV-032`, `TC-INV-047` y `TC-INV-110`. Las reglas que describen texto o presentación sin efecto separable (por ejemplo los avisos de «Brecha conocida») se cubren dentro del caso del escenario correspondiente. No hay reglas del alcance sin caso.

| Prefijo | Regla → casos |
|---|---|
| `RN-UDM` | 001:001,002,007,012 · 002:001,003,011,027 · 003:001,004,027 · 004:001,003,011,027 · 005:001,004,007,027 · 006:001,005,008,027 · 007:006 · 008:010 · 009:009,021 · 010:022,023 · 011:021 · 012:013,014 · 013:016 · 014:025 · 015:027,031 · 016:029,031,032,033 · 017:032 |
| `RN-JER` | 001:034,042,046 · 002:034,056 · 003:034,056 · 004:034,036,056 · 005:034,037,056 · 006:034,041 · 007:034,041 · 008:038,058 · 009:039,040,058 · 010:039,045,060 · 011:060,061,123 · 012:042,044 · 013:043,051 · 014:052 · 015:047,048,049 · 016:050,059 · 017:054 · 018:055 · 019:056,057 · 020:054 · 021:058 · 022:059 |
| `RN-LAB` | 001:062 · 002:062,064,077 · 003:062,064,077 · 004:062,063,064 · 005:063,067 · 006:065,077 · 007:065,066,078 · 008:070 · 009:069,073 · 010:074 · 011:068,073 · 012:071,072 · 013:076 · 014:077 · 015:078 · 016:078,079 · 017:079 |
| `RN-ING` | 001:080 · 002:080,081,091 · 003:080,081,091 · 004:080,081,091 · 005:080,081,083,091 · 006:082,083,092 · 007:084,085 · 008:085,090 · 009:088,089 · 010:086,087 · 011:091 · 012:092 · 013:093 · 014:093 · 015:093 |
| `RN-MED` | 001:094 · 002:094,096 · 003:094,098 · 004:094,096 · 005:094,096,100,101 · 006:094,096,100 · 007:094,095 · 008:094,097 · 009:101 · 010:094,099,116 · 011:101,102 · 012:102,109,110,113 · 013:110,111,112 · 014:103,104,105 · 015:119,120 · 016:121 · 017:122 · 018:107,108 · 019:095,103 · 020:106 · 021:114,117 · 022:115 · 023:115 · 024:116 · 025:114,118 · 026:118 |
| `RN-PRE` | 001:123,133 · 002:123,127,130 · 003:123,129 · 004:123,127,130 · 005:123,128,129,134 · 006:123,126 · 007:052,123,124,129 · 008:123,124 · 009:123,127,134,136 · 010:132 · 011:130,140 · 012:137,141 · 013:140 · 014:138,139 · 015:123,125,133,135 |
| `RN-FOR` | 001:142,152 · 002:142,143 · 003:142,146 · 004:142,143,146 · 005:142,145,146 · 006:142,143 · 007:142,143 · 008:142,144 · 009:148 · 010:147,150 · 011:150 · 012:149 · 013:152 |
| `RN-PRV` | 001:153,171 · 002:153,154,164 · 003:153,154,156,164 · 004:153,154,164 · 005:153,155,164 · 006:153,155,164 · 007:156,157 · 008:156,165 · 009:157,160,162 · 010:160,161 · 011:153 · 012:158,159 · 013:163 · 014:164 · 015:165 · 016:165 · 017:165 · 018:166,168,169 · 019:170 · 020:166,167,173 · 021:172 · 022:172 · 023:172 · 024:172 · 025:174,175 · 026:174,175 · 027:166,168,169 · 028:166,169,171 |
| `RN-ASE` | 001:176 · 002:176,177,187 · 003:176,177,187 · 004:178,188 · 005:179,180,183 · 006:183,184 · 007:176 · 008:181,182 · 009:186 · 010:187 · 011:188,189 · 012:189 |
| `RN-DIG` | 001:190,195,197,200 · 002:197,200,201 · 003:197,201 · 004:197,199 · 005:197,199 · 006:197,198,201 · 007:192,193,202 · 008:202 · 009:190,191,194 · 010:196 · 011:197 · 012:198 · 013:199 |
| `RN-LOT` (`TC-INV-`) | 001:001,019,024,049 · 002:001,003 · 003:001,003 · 004:001,003 · 005:001,003 · 006:004,005,011,047 · 007:002,006 · 008:006,012 · 009:009 · 010:007,008,010 · 011:014 · 012:010 · 013:013,017,020,023 · 014:021,022 · 015:008,020 · 016:001 · 017:015,016 · 018:024 · 019:018 |
| `RN-STK` (`TC-INV-`) | 001:025,027,037,039 · 002:025,035 · 003:025,035 · 004:025,029,038 · 005:031 · 006:028,030,040 · 007:031,032 · 008:026,036 · 009:025 · 010:026,033 · 011:034,041 · 012:037,042 · 013:040,042 · 014:040,044,050 · 015:043,045,046 · 016:048 · 018:049,050,051 |
| `RN-PAC` (`TC-INV-`) | 001:052 · 002:052,053,055,056 · 003:052,055,056,069 · 004:052,054,056,060 · 005:052,055,069 · 006:052,055,058,068 · 007:052,059 · 008:057,069 · 009:061,064,066 · 010:064,065 · 011:062,063 · 012:067 · 013:067 · 014:068 · 015:069 · 016:067 · 017:069 |
| `RN-PRO` (`TC-INV-`) | 001:070 · 002:070,071,076 · 003:070,071,076 · 004:070,071,076 · 005:071,072,076 · 006:072,074 · 007:074,075 · 008:073 · 009:076 · 010:076 · 011:076 · 012:076 |
| `RN-REC` (`TC-INV-`) | 001:077,089,096,097 · 002:077,078 · 003:077,078 · 004:077,084,085 · 005:077,084,085 · 006:077,079 · 007:077,079,080 · 008:077,081 · 009:077,083 · 010:077,082 · 011:085,087 · 012:086,094,096 · 013:095 · 014:094 · 015:077 · 016:090,091,093 · 017:092 · 018:097,098,099,101 · 019:103 · 020:104 · 021:098,114,115,116 · 022:113 · 023:105,106 · 024:105,107 · 025:108,109 · 026:110 · 027:111 · 028:112 · 029:118 · 030:118 · 031:088 |

Reglas transversales citadas fuera del alcance (en [01a][rn-a]): `RN-GEN-013`, `RN-GEN-023`, `RN-GEN-028`, `RN-GEN-035`, `RN-GEN-039`, `RN-IMP-001` a `RN-IMP-006`, `RN-IMP-008`, `RN-IMP-009`, `RN-IMP-011` y `RN-IMP-013`; verificadas en el script como existentes.

## 21. Fuentes

Documentos del repositorio de documentación (`sofia-docs`):

- `_inventario/REGLAS-AGENTES.md` y `_inventario/QA-AGENTES.md` (reglas comunes e instrucciones de esta parte).
- [Historias de usuario 12b][hu] (66 historias, 401 escenarios y las secciones comunes 1.2 y 1.3), [casos de uso 13b][cu] (flujos, alternativos y tablas de excepciones), [historias 12c][hu-c] (solo para ubicar las transferencias).
- Reglas de negocio [01b][rn-b] (UDM, JER, LAB, ING, MED, PRE, FOR, PRV, ASE, DIG, PAC, PRO), [01c1][rn-c] (LOT, STK, REC) y [01a][rn-a] (GEN, IMP).
- Inventarios de pantallas [19c][p19c], [19d][p19d] y [19e][p19e]; [carga masiva y exportación (20)][p20]; [integraciones (07)][g07]; [guía del desarrollador (08)][g08]; [despliegue y operación (10)][g10]; manual de usuario [capítulo 3][m3] y [capítulo 4][m4] (el capítulo 5 no se usó); [procesos de negocio][p14], [matriz de permisos][p05b] y [catálogo de códigos de error][p21] como contexto.

Código leído de forma puntual y de solo lectura (`sofia-apis/src`):

- `SOFIA.API/Controllers/UnidadesMedidaController.cs` (identificador de la URL y GUID vacío), `LotesInventarioController.cs` (ingreso de stock) y `RecetasController.cs` (texto «Image is required.»); `SOFIA.API/Extensions/ResultExtensions.cs` (respuesta 200 en los resultados con valor).
- `SOFIA.Application/Inventarios/Commands/RegisterInventario/RegisterInventarioCommandHandler.cs` y `RegisterInventarioCommand.cs`, `AdjustStock/AdjustStockCommandHandler.cs` y `AdjustStockCommand.cs`, `DeleteLote/DeleteLoteInventarioCommandHandler.cs`, `SOFIA.Application/Common/Models/AuditEventos.cs`.
- `SOFIA.Domain/Entities/RecetaMedica.cs` (código `Venta.Receta.Agotada`).

Herramientas de verificación (solo lectura, en el scratchpad de la sesión): `16b-cu.js` y `16b-verify.js`.

[hu]: ../../ola-2-funcional/12-historias-de-usuario/12b-catalogos-inventario-atencion.md
[hu-c]: ../../ola-2-funcional/12-historias-de-usuario/12c-punto-de-venta-y-caja.md
[cu]: ../../ola-2-funcional/13-casos-de-uso/13b-catalogos-inventario-atencion.md
[rn-a]: ../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md
[rn-b]: ../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md
[rn-c]: ../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md
[p19c]: ../../ola-1-fundamentos/19-inventario-de-pantallas/19c-catalogos-1.md
[p19d]: ../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md
[p19e]: ../../ola-1-fundamentos/19-inventario-de-pantallas/19e-inventario-y-atencion.md
[p20]: ../../ola-1-fundamentos/20-carga-masiva-y-exportacion.md
[m3]: ../../ola-2-funcional/manual-usuario/capitulo-03-empleados-roles-cuentas.md
[m4]: ../../ola-2-funcional/manual-usuario/capitulo-04-catalogos-basicos.md
[g07]: ../../ola-1-fundamentos/07-integraciones.md
[g08]: ../../ola-1-fundamentos/08-guia-del-desarrollador.md
[g10]: ../../ola-1-fundamentos/10-despliegue-y-operacion.md
[p05b]: ../../ola-1-fundamentos/05-seguridad/05b-matriz-de-permisos.md
[p21]: ../../ola-1-fundamentos/21-catalogo-codigos-error.md
[p14]: ../../ola-2-funcional/14-procesos-de-negocio.md
