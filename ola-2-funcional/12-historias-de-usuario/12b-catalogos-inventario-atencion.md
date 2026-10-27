# Historias de usuario — Parte B: catálogos, inventario y atención

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-26 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de los documentos validados (reglas de negocio 01b v1.1 y 01c1, inventario de pantallas 19c, 19d y 19e, carga masiva 20, matriz de permisos 05b, glosario, procesos 3 y 7, manual de usuario capítulos 4 y 5) y comprobación puntual en el código y en los `es.json`. No se ejecutó la aplicación ni se llamó a la API: los mensajes exactos salen de los documentos fuente, que los comprobaron en la aplicación local o en el código. |

## 1. Cómo leer este documento

Este documento reúne las historias de usuario de la **parte B**: los catálogos maestros (unidades de medida, jerarquías de unidades, laboratorios, ingredientes activos, medicamentos, presentaciones de venta, formulaciones clínicas, proveedores con sus precios, aseguradoras y catálogo DIGEMID), el inventario (lotes y stock por sucursal) y la atención (pacientes, profesionales de salud, recetas médicas y el escáner de recetas con IA). La parte A cubre acceso y administración (`HU-ADM-…`, [12a](./12a-acceso-y-administracion.md)) y la parte C punto de venta, caja, devoluciones y dashboard.

**Convenciones**

- Los IDs son `HU-CAT-nnn` para catálogos e `HU-INV-nnn` para inventario y atención. Cada historia es una capacidad de negocio, no un botón: «registrar y editar» van juntos porque comparten pantalla y reglas.
- Las reglas `RN-…` son las del [catálogo de reglas, parte B (01b, versión 1.1)](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), las de [inventario y recetas (01c1)](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) y, para las transversales `GEN` e `IMP`, las de la [parte A (01a)](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md). En cada historia, las reglas de un mismo prefijo se enlazan juntas.
- Los textos entre « » son los que ve el usuario, copiados de los `es.json` a través de los documentos fuente. Si el sistema no tiene un mensaje propio para un caso, el escenario dice cuál es el mensaje genérico que se ve.
- **Estado.** «Implementada»: hay pantalla (o endpoint, si la historia es de API) y funciona como describen los escenarios. «Parcial»: falta algo, y la historia lo dice. «Solo API»: existe el endpoint y ninguna pantalla lo usa.
- **Brecha conocida.** Cuando el sistema hoy se comporta distinto de lo que el negocio esperaría (por ejemplo, un mensaje genérico donde debería haber uno específico), el escenario describe el comportamiento **actual** y una nota «Brecha conocida» cita la fuente. Ningún escenario da por cumplido algo que el sistema no hace.
- **No verificado en la aplicación.** Algunos escenarios dependen de casos que ni el manual ni los inventarios pudieron provocar (por ejemplo, los avisos de base de datos). Se marcan así; la fuente es el código leído en el catálogo de reglas.
- Los permisos se escriben `Módulo:Acción` como los exige el endpoint. Para entrar a una pantalla siempre se necesita además `Leer` del módulo; las rutas `…/nueva` y `…/carga-masiva` exigen también `Crear`, salvo `/stock-por-sucursal/nueva` (exige `Inventarios:Actualizar`) y `/recetas-medicas/escanear` (exige `Recetas:Analizar`). En la pantalla de permisos de un rol, el módulo `Inventarios` se rotula «Inventario», `Seguros` «Aseguradoras» y `Recetas` «Recetas Médicas».
- Los datos de los ejemplos son los ficticios del manual: la empresa «Botica Manual Demo S.A.C.», la administradora «Ana Demo Prueba», el cajero «Carlos Cajero Demo» y los datos de los capítulos 4 y 5 (unidades `UND`, `CAJ`, `BLI`; laboratorios «Laboratorio Demo Andino» y «Laboratorio Demo Pacífico»; medicamentos `DEMO-MED-0001` a `DEMO-MED-0004`).

### 1.1 Actores

| Actor | Quién es |
|---|---|
| Administrador | Usuario con el rol llamado `Admin`: pasa cualquier comprobación de permiso. Lo que se dice de «administrador» vale para cualquier usuario cuyo rol tenga el permiso indicado en la historia. |
| Almacenero | Usuario con permisos de `Inventarios` (lotes y stock). |
| Comprador | Usuario con `Proveedores` y, para registrar costos, `Proveedores:RegistrarPrecioProveedor`. |
| Cajero o personal de atención | Usuario con permisos de `Pacientes`, `ProfesionalesSalud` y `Recetas`; el cajero del manual solo tiene `DIGEMID:Leer` entre los catálogos del capítulo 4. |
| Integrador / equipo técnico | Quien consume la API directamente (historias «Solo API»). |

### 1.2 Comportamientos comunes de las pantallas de búsqueda y de detalle

Para no repetirlos en cada historia, estos comportamientos valen para todas las pantallas de esta parte (fuentes: [19c §0](../../ola-1-fundamentos/19-inventario-de-pantallas/19c-catalogos-1.md), [19d §0](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md), [19e §0](../../ola-1-fundamentos/19-inventario-de-pantallas/19e-inventario-y-atencion.md), capítulo 4 sección 4 del manual). Cada historia solo nombra las excepciones.

```gherkin
Escenario: Los resultados aparecen solo después de buscar
  Dado que abro una pantalla de búsqueda
  Entonces veo la tarjeta de filtros con "Ingresa los criterios de búsqueda" y no veo ninguna tabla
  Cuando hago clic en "Buscar"
  Entonces veo "Resultados (N)" con la tabla, cuya primera columna es un enlace al detalle
  Y con Enter dentro de un filtro de texto también se busca, salvo en los filtros de Paciente y Médico de Recetas Médicas

Escenario: Limpiar y volver conservan o descartan los filtros
  Dado que busqué con filtros y abrí un detalle
  Cuando vuelvo con la flecha de la esquina superior izquierda
  Entonces la búsqueda recuerda mis filtros y repite la búsqueda sola
  Y cuando hago clic en "Limpiar" se vacían los filtros, se oculta la tabla y se olvida el estado guardado

Escenario: Paginación
  Dado que hay resultados
  Entonces el paginador ofrece 10, 25 o 50 filas por página (10 por defecto) con botones de primera, anterior, siguiente y última página
  Y "Buscar" no vuelve a la página 1: pide la página en la que estás con los nuevos filtros

Escenario: Sin coincidencias
  Dado que ningún registro cumple los filtros
  Entonces veo "Resultados (0)" y el paginador dice "Sin resultados"
  Y la fila vacía dice "Resultados" en unidades, laboratorios, ingredientes, DIGEMID, formulaciones, proveedores, aseguradoras, pacientes y profesionales, y "Sin resultados" en jerarquías, medicamentos, presentaciones, lotes, stock y recetas

Escenario: Modo ver, editar y crear en el detalle
  Dado que abro un registro existente
  Entonces veo los campos deshabilitados y, con el permiso "Actualizar", el botón "Editar"
  Y al hacer clic en "Editar" aparecen "Cancelar", "Guardar" y, con el permiso "Eliminar", "Eliminar"
  Y "Cancelar" en crear vuelve a la búsqueda; en editar descarta lo escrito y vuelve al modo ver

Escenario: Eliminar pide confirmación
  Dado que estoy editando un registro existente con el permiso "Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces se abre "Confirmar eliminación" con el mensaje de esa pantalla y los botones "Cancelar" y "Confirmar"
  Y "Cancelar" cierra el cuadro sin borrar nada

Escenario: La pantalla decide el aviso de eliminación solo por el estado HTTP
  Dado que confirmo la eliminación
  Cuando la API responde 409 (cualquiera sea su código)
  Entonces veo el aviso "en uso" de esa pantalla
  Pero con cualquier otro error (404, 403, 500, sin conexión) veo el aviso genérico "Error al eliminar …" de esa pantalla

Escenario: Valores de solo espacios
  Dado que escribo solo espacios en un campo obligatorio de texto
  Cuando hago clic en "Guardar"
  Entonces la pantalla lo acepta porque el campo no está vacío
  Pero el servidor lo rechaza y veo el aviso genérico de guardado de esa pantalla

Escenario: Errores de base de datos poco frecuentes (no verificado en la aplicación)
  Dado que dos personas guardan el mismo valor único a la vez, o un valor llega fuera de rango
  Entonces, en las pantallas que usan el mapa común de errores, veo según el caso
    "Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo."
    "El registro hace referencia a datos que ya no existen o todavía está en uso."
    "Uno de los valores está fuera del rango permitido."

Escenario: Sin permiso de ruta
  Dado que mi rol no tiene "Leer" (o "Crear" en "nueva" y "carga-masiva") del módulo
  Cuando escribo la dirección de la pantalla a mano
  Entonces veo "No tienes permisos para acceder a esta pantalla." y vuelvo al Inicio
```

> **Brecha conocida (común).** Los avisos de las pantallas de Aseguradoras, Presentaciones de venta, Formulaciones clínicas, Lotes y Stock por sucursal no usan el mapa común de errores de base de datos; cada una tiene su propio tratamiento, descrito en su historia ([19d §0](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md), [19e §0](../../ola-1-fundamentos/19-inventario-de-pantallas/19e-inventario-y-atencion.md)). Los códigos de error de validación (`{Request}.{Propiedad}`) y los `*.NotFound` no tienen texto propio en la mayoría de las pantallas: el usuario ve el aviso genérico de guardado. **No verificado en la aplicación:** la redirección al Inicio por falta de permiso de ruta (el administrador tiene todos los permisos y el navegador integrado impidió probarlo con el cajero; viene de 19c §0, manual cap. 4 §10 y §12).

### 1.3 Escenarios comunes de la carga masiva

Doce pantallas de esta parte tienen carga masiva (unidades, jerarquías, laboratorios, ingredientes, medicamentos, proveedores, aseguradoras, catálogo DIGEMID, pacientes y profesionales de salud). Todas usan el mismo componente y los mismos límites ([documento 20](../../ola-1-fundamentos/20-carga-masiva-y-exportacion.md), reglas [RN-IMP-001 a RN-IMP-006, RN-IMP-012 a RN-IMP-016](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) y [01b §3.1](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md#31-comportamiento-común-de-la-carga-masiva-en-estos-módulos)). Cada historia de carga masiva dice «cumple los escenarios CM-1 a CM-5» y detalla solo lo propio (columnas, reglas por fila, mensajes por fila).

```gherkin
Escenario: CM-1 Descargar la plantilla
  Dado que tengo "Crear" del módulo y estoy en "Carga Masiva" (título "Carga Masiva", zona "Arrastra tu archivo Excel aquí o haz clic para seleccionar" y debajo "Solo archivos .xlsx (máx. 5 MB y 5000 filas)")
  Cuando hago clic en "Descargar plantilla"
  Entonces se guarda "plantilla-<módulo>.xlsx" con una sola hoja "Plantilla" que solo tiene la fila de títulos de columna (negrita, texto blanco sobre azul)
  Y si falla veo "Error al descargar la plantilla. Intenta de nuevo."

Escenario: CM-2 Rechazo del archivo
  Dado que elijo un archivo
  Cuando no termina en ".xlsx", no es un libro de Excel real, pesa más de 5 MB, tiene más de 5000 filas con datos o se descomprime por encima de los límites
  Entonces veo, según el caso,
    "El archivo no es un Excel .xlsx válido. Descarga la plantilla y vuelve a intentarlo."
    "El archivo supera el tamaño máximo de 5 MB."
    "El archivo supera el máximo de 5000 filas. Divídelo en varios archivos."
    "El archivo excede los límites de contenido permitidos. Revisa que sea una plantilla válida."
  Y si falla el procesamiento por otra causa veo "Error al procesar el archivo"

Escenario: CM-3 Vista previa sin guardar nada
  Dado que subí un archivo válido
  Entonces veo "N filas totales", "N filas válidas" y, si hay errores, "N filas con error"
  Y una tabla con una fila por registro, una columna sin título con los mensajes de error de la fila y la columna "Estado" (marca verde si es válida)
  Y no se guardó nada todavía
  Y las columnas del Excel se leen sin distinguir mayúsculas, las columnas extra se ignoran, una columna ausente se lee vacía, los valores se recortan y las filas totalmente vacías se ignoran

Escenario: CM-4 Una fila repetida en el archivo marca todas sus apariciones
  Dado que el mismo valor único aparece en dos filas del archivo (sin distinguir mayúsculas)
  Entonces ambas filas, también la primera, muestran "El valor '<valor>' del campo '<campo>' está duplicado en el archivo"
  Y un valor que ya existe en mi empresa muestra "El valor '<valor>' del campo '<campo>' ya existe en el sistema"
  Y el nombre del campo se ve como lo usa el sistema por dentro (por ejemplo "codigo"), no con la etiqueta de la pantalla

Escenario: CM-5 Guardar solo las filas válidas
  Dado que la vista previa tiene filas válidas
  Cuando hago clic en "Guardar válidos (N)"
  Entonces solo se envían las filas válidas y veo "Se guardaron N registros correctamente"
  Y vuelvo a la búsqueda del catálogo
  Pero si ninguna fila es válida el botón está deshabilitado y, si se intentara, veo "No hay filas válidas para guardar"
  Y si el servidor rechaza filas al guardar veo "Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa." y no se guarda ninguna
  Y si falla el guardado por otra causa veo "Error al guardar los registros"
```

> **Brecha conocida (común).** Cada carga vuelve a validar y las filas que chocan con un valor único existente se **omiten sin error**: el aviso de éxito dice cuántas se guardaron, que puede ser menos que las «filas válidas» de la vista previa, incluso «Se guardaron 0 registros correctamente» (comprobado en el manual con laboratorios). Siempre compara el número guardado con el de filas válidas. Una lista vacía muestra el mismo texto de «demasiadas filas» ([01b §3.1](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md#31-comportamiento-común-de-la-carga-masiva-en-estos-módulos), [RN-IMP-006](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md)).

## 2. Índice

| Sección | Historias |
|---|---|
| 3. Unidades de medida | HU-CAT-001 a HU-CAT-004 |
| 4. Jerarquías de unidades | HU-CAT-005 a HU-CAT-009 |
| 5. Laboratorios | HU-CAT-010 a HU-CAT-013 |
| 6. Ingredientes activos | HU-CAT-014 a HU-CAT-017 |
| 7. Medicamentos | HU-CAT-018 a HU-CAT-023 |
| 8. Presentaciones de venta | HU-CAT-024 a HU-CAT-027 |
| 9. Formulaciones clínicas | HU-CAT-028 a HU-CAT-031 |
| 10. Proveedores y precios de compra | HU-CAT-032 a HU-CAT-037 |
| 11. Aseguradoras | HU-CAT-038 a HU-CAT-041 |
| 12. Catálogo DIGEMID | HU-CAT-042 a HU-CAT-044 |
| 13. Lotes de inventario | HU-INV-001 a HU-INV-004 |
| 14. Stock por sucursal | HU-INV-005 a HU-INV-008 |
| 15. Pacientes | HU-INV-009 a HU-INV-012 |
| 16. Profesionales de salud | HU-INV-013 a HU-INV-016 |
| 17. Recetas médicas y escáner con IA | HU-INV-017 a HU-INV-022 |
| 18 a 21. Trazabilidad, pendientes, cobertura y fuentes | — |

El flujo de punta a punta de catálogo e inventario está en el [proceso 3](../14-procesos-de-negocio.md) y el de recetas con IA en el [proceso 7](../14-procesos-de-negocio.md) del documento de procesos de negocio; estas historias son su desglose por capacidad.

---

## 3. Unidades de medida

### HU-CAT-001 — Registrar y editar una unidad de medida

**Como** administrador **quiero** registrar las unidades en que cuento y venda mis productos (unidad, caja, blíster, miligramo…) y corregirlas cuando haga falta **para** que medicamentos, jerarquías, presentaciones y formulaciones usen siempre las mismas unidades.

| | |
|---|---|
| **Módulo** | Unidades de medida |
| **Pantalla** | `/unidades-medida/nueva` y `/unidades-medida/:id` — «Unidades de Medida» |
| **Endpoint(s)** | `POST /api/v1/UnidadesMedida`, `PUT /api/v1/UnidadesMedida/{id}`, `GET /api/v1/UnidadesMedida/{id}` |
| **Permiso** | `UnidadesMedida:Crear` (registrar), `UnidadesMedida:Actualizar` (editar), `UnidadesMedida:Leer` (abrir el detalle) |
| **Reglas** | [RN-UDM-001, RN-UDM-002, RN-UDM-003, RN-UDM-004, RN-UDM-005, RN-UDM-006, RN-UDM-007, RN-UDM-008, RN-UDM-009](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Registrar una unidad
  Dado que tengo el permiso "Unidades de Medida › Crear" y estoy en "/unidades-medida"
  Cuando hago clic en "Nuevo"
  Y escribo Código "UND" y Descripción "Unidad"
  Y hago clic en "Guardar"
  Entonces veo el aviso "Unidad de medida guardada correctamente"
  Y vuelvo a la búsqueda de unidades, con los filtros vacíos y sin tabla (o con la tabla y los filtros que tenía si ya había buscado)

Escenario: Sin el permiso no hay botón
  Dado que mi rol no tiene "Unidades de Medida › Crear"
  Entonces no veo "Nuevo" ni "Carga Masiva"
  Y si escribo "/unidades-medida/nueva" veo "No tienes permisos para acceder a esta pantalla." y vuelvo al Inicio

Escenario: Campos obligatorios vacíos
  Dado que el formulario está vacío
  Cuando hago clic en "Guardar"
  Entonces debajo de "Código" y de "Descripción" veo "Este campo es obligatorio" y no se envía nada

Escenario: Longitudes máximas
  Dado que escribo 16 letras en "Código"
  Entonces el campo se queda en 10 caracteres y no muestra ningún mensaje
  Y "Descripción" no deja pasar de 50 caracteres

Escenario: Código repetido en mi empresa
  Dado que ya existe la unidad "UND"
  Cuando intento registrar otra unidad con código "UND" (o "und")
  Entonces veo el aviso rojo "Ya existe una unidad de medida con ese código."
  Y me quedo en el formulario

Escenario: El código de una unidad eliminada queda libre
  Dado que eliminé la unidad con código "PRU"
  Cuando registro una unidad nueva con código "PRU"
  Entonces se guarda con "Unidad de medida guardada correctamente"

Escenario: Editar una unidad
  Dado que abrí la unidad "UND" y tengo "Unidades de Medida › Actualizar"
  Cuando hago clic en "Editar", cambio la descripción y hago clic en "Guardar"
  Entonces veo "Unidad de medida guardada correctamente"
  Y me quedo en el detalle, en modo ver y con los datos recargados

Escenario: Cambiar el código a uno que ya usa otra unidad
  Dado que estoy editando la unidad "PRU"
  Cuando cambio su código a "UND" y hago clic en "Guardar"
  Entonces veo "Ya existe una unidad de medida con ese código."

Escenario: La unidad ya no existe
  Dado que la unidad fue eliminada o es de otra empresa
  Cuando intento guardarla o abrirla
  Entonces la API responde 404 "UnidadMedida.NotFound"
  Y al guardar veo "Error al guardar la unidad de medida"
  Y al abrir el detalle veo "Error al cargar la unidad de medida"

Escenario: Identificador de la unidad al editar (solo API)
  Dado que llamo a "PUT /api/v1/UnidadesMedida/{id}"
  Entonces la API toma siempre el identificador de la URL y el del cuerpo se descarta
  Y un identificador vacío se rechaza con "UpdateUnidadMedidaCommand.Id" (la pantalla nunca lo envía vacío)
```

> **Brecha conocida.** (1) Un código o descripción de solo espacios pasa el formulario y el servidor lo rechaza con un código sin traducción: el usuario ve «Error al guardar la unidad de medida» (comprobado en el manual, sección 5.3). (2) La comparación de códigos al crear y editar usa igualdad de SQL: el manual comprobó que `und` se rechaza si existe `UND`, pero la intercalación de la base no está definida en los scripts, así que el comportamiento ante otras mayúsculas es **No verificado** en el código ([01b nota del módulo UDM](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md#4-unidades-de-medida-udm)).

### HU-CAT-002 — Buscar, consultar y exportar unidades de medida

**Como** administrador **quiero** buscar mis unidades por código o descripción y exportar la lista **para** revisar qué unidades existen antes de crear medicamentos.

| | |
|---|---|
| **Módulo** | Unidades de medida |
| **Pantalla** | `/unidades-medida` — «Unidades de Medida» (migas «Inicio / Catálogos / Unidades de Medida», tarjeta «Búsqueda de Unidades de Medida») |
| **Endpoint(s)** | `GET /api/v1/UnidadesMedida`, `POST /api/v1/UnidadesMedida/exportar` |
| **Permiso** | `UnidadesMedida:Leer` |
| **Reglas** | [RN-UDM-001, RN-UDM-012, RN-UDM-013](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009, RN-IMP-017](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Ver el menú y la pantalla
  Dado que mi rol tiene "Unidades de Medida › Leer"
  Entonces veo "Unidades de Medida" en el menú, grupo CATÁLOGOS
  Y al abrirla veo los filtros "Código" y "Descripción", ambos con lupa

Escenario: Buscar sin filtros
  Dado que existen nueve unidades
  Cuando hago clic en "Buscar" sin escribir nada
  Entonces veo "Resultados (9)" ordenadas por descripción: Blíster, Caja, Frasco, Gramo, Kilogramo, Miligramo, Mililitro, Tableta y Unidad
  Y la tabla tiene las columnas "Código" (enlace al detalle) y "Descripción"

Escenario: Filtrar por código o por descripción
  Dado que existen las unidades "MG" y "ML"
  Cuando escribo "m" en "Código" y hago clic en "Buscar"
  Entonces veo "MG" y "ML"
  Y "UNIDAD" en "Descripción" trae las unidades cuya descripción contiene "unidad", sin distinguir mayúsculas
  Y "Código" "u" junto con "Descripción" "prueba" trae solo la unidad que cumple las dos condiciones

Escenario: Sin coincidencias
  Cuando busco el código "zzz"
  Entonces veo "Resultados (0)", la fila vacía "Resultados" y el paginador "Sin resultados"

Escenario: Exportar a Excel
  Dado que busqué
  Cuando hago clic en "Exportar"
  Entonces se descarga "unidades-medida.xlsx" con una hoja "Data" y las columnas "Código" y "Descripción"
  Y el archivo respeta los filtros que tengo escritos en ese momento

Escenario: Exportación demasiado grande
  Dado que los filtros dejan más de 10000 filas
  Cuando hago clic en "Exportar"
  Entonces la API responde 400 "Export.TooManyRows" y no genera el archivo
  Y veo "Error al buscar unidades de medida"

Escenario: Falla la búsqueda
  Dado que el servidor falla
  Entonces veo "Error al buscar unidades de medida"
```

> **Nota.** Los filtros comparan «contiene» sin distinguir mayúsculas. El inventario no tiene filtros numéricos ni de fecha porque el catálogo solo tiene código y descripción. El contenido del archivo exportado y su descarga son **No verificado en la aplicación** (el navegador integrado no muestra descargas); sí se comprobó que el servidor responde 200 y que no aparece aviso de error (manual cap. 4, §5.4).

### HU-CAT-003 — Eliminar una unidad de medida

**Como** administrador **quiero** eliminar una unidad que ya no uso, sin romper productos ni conversiones que la usan **para** mantener el catálogo limpio.

| | |
|---|---|
| **Módulo** | Unidades de medida |
| **Pantalla** | `/unidades-medida/:id` (botón «Eliminar» en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/UnidadesMedida/{id}` |
| **Permiso** | `UnidadesMedida:Eliminar` |
| **Reglas** | [RN-UDM-001, RN-UDM-007, RN-UDM-009, RN-UDM-010, RN-UDM-011](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-003](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (el aviso de «en uso» no menciona las formulaciones clínicas) |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar una unidad sin uso
  Dado que abrí la unidad "PRU", hice clic en "Editar" y tengo "Unidades de Medida › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces se abre "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar esta unidad de medida?"
  Y al hacer clic en "Confirmar" veo "Unidad de medida eliminada correctamente"
  Y vuelvo a la búsqueda, que repite mis filtros
  Y la eliminación es lógica: la unidad desaparece de todas las listas y su código queda libre

Escenario: Cancelar no borra nada
  Dado que se abrió "Confirmar eliminación"
  Cuando hago clic en "Cancelar"
  Entonces el cuadro se cierra y la unidad sigue existiendo

Escenario: La unidad está en uso
  Dado que la unidad "UND" es la unidad base de un medicamento, la unidad mayor o menor de una jerarquía, o la unidad de dosis de una formulación clínica
  Cuando confirmo la eliminación
  Entonces la API responde 409 "UnidadMedida.InUse"
  Y veo "No se puede eliminar la unidad de medida porque está asociada a medicamentos o jerarquías de unidades."

Escenario: Sin permiso no hay botón
  Dado que mi rol no tiene "Unidades de Medida › Eliminar"
  Entonces no veo el botón "Eliminar" aunque entre en modo editar

Escenario: La unidad ya no existe
  Dado que otra persona la eliminó antes
  Cuando confirmo la eliminación
  Entonces la API responde 404 "UnidadMedida.NotFound"
  Y veo "Error al eliminar la unidad de medida"
```

> **Brecha conocida.** (1) El texto de «en uso» menciona medicamentos y jerarquías pero no las formulaciones clínicas, que también bloquean ([19c §1.2](../../ola-1-fundamentos/19-inventario-de-pantallas/19c-catalogos-1.md)). (2) La pantalla muestra ese texto ante **cualquier** 409, también uno de concurrencia. (3) El handler no consulta las presentaciones de venta que usan la unidad como unidad de venta: como el borrado es lógico, la clave foránea no lo impide y la unidad se puede eliminar aunque una presentación la use ([01b nota del módulo UDM](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md#4-unidades-de-medida-udm)). (4) **No verificado en la aplicación:** el bloqueo por uso (el manual lo dejó pendiente porque no existían medicamentos; viene de 19c §1.2 y §6).

### HU-CAT-004 — Cargar unidades de medida desde un Excel

**Como** administrador **quiero** cargar muchas unidades de una sola vez desde una hoja de Excel **para** no crearlas una por una.

| | |
|---|---|
| **Módulo** | Unidades de medida (carga masiva) |
| **Pantalla** | `/unidades-medida/carga-masiva` — «Carga Masiva» |
| **Endpoint(s)** | `GET /api/v1/UnidadesMedida/plantilla`, `POST /api/v1/UnidadesMedida/previsualizar`, `POST /api/v1/UnidadesMedida/carga-masiva` |
| **Permiso** | `UnidadesMedida:Crear` (además de `Leer`) |
| **Reglas** | [RN-UDM-002, RN-UDM-003, RN-UDM-004, RN-UDM-005, RN-UDM-006, RN-UDM-014, RN-UDM-015, RN-UDM-016, RN-UDM-017](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-IMP-001 a RN-IMP-006, RN-IMP-013 a RN-IMP-016](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

Cumple los escenarios CM-1 a CM-5 de la [sección 1.3](#13-escenarios-comunes-de-la-carga-masiva). Además:

```gherkin
Escenario: Columnas de la plantilla
  Dado que descargué "plantilla-unidades-medida.xlsx"
  Entonces tiene las columnas "Codigo" (obligatoria, máximo 10 caracteres) y "Descripcion" (obligatoria, máximo 50), escritas sin tilde

Escenario: Reglas por fila en la vista previa
  Dado que subí un archivo de 7 filas
  Entonces una fila con "Codigo" vacío muestra "El campo 'codigo' es obligatorio"
  Y una fila con código "UND" que ya existe muestra "El valor 'UND' del campo 'codigo' ya existe en el sistema"
  Y dos filas con código "LT" muestran las dos "El valor 'LT' del campo 'codigo' está duplicado en el archivo"
  Y una fila con 14 letras de código y 60 de descripción muestra dos mensajes: "'codigo' no puede superar los 10 caracteres" y "'descripcion' no puede superar los 50 caracteres"
  Y la vista previa dice "7 filas totales", "2 filas válidas" y "5 filas con error"

Escenario: Guardar las filas válidas
  Dado que las filas "GR / Gramo" y "KG / Kilogramo" son válidas
  Cuando hago clic en "Guardar válidos (2)"
  Entonces veo "Se guardaron 2 registros correctamente" y vuelvo a la búsqueda de unidades

Escenario: La descripción puede repetirse
  Dado que dos filas tienen distinto código y la misma descripción
  Entonces ambas son válidas

Escenario: Al guardar se omiten las repetidas que llegaran hasta el servidor
  Dado que, aun así, llegan al guardado dos filas con el mismo código o un código ya existente
  Entonces el servidor guarda la primera y omite las siguientes sin error
  Y la respuesta informa solo cuántas guardó

Escenario: Lista de filas fuera de rango (solo API)
  Dado que llamo a "POST /api/v1/UnidadesMedida/carga-masiva" con 0 o más de 5000 filas
  Entonces la API responde 400 "CargaMasivaUnidadesMedidaCommand.Rows"
  Y la pantalla lo muestra como "El archivo supera el máximo de 5000 filas. Divídelo en varios archivos."

Escenario: Vista previa sin archivo (solo API)
  Dado que llamo a "POST /api/v1/UnidadesMedida/previsualizar" sin archivo o con un archivo vacío
  Entonces la API responde 400 con el texto "Debe adjuntar un archivo Excel."
  Y la pantalla nunca lo provoca: siempre envía un archivo
```

> **Nota.** Comprobado en el manual (§9.2) con las 7 filas del ejemplo; el contenido de la plantilla y los errores de tamaño o de más de 5000 filas son **No verificado en la aplicación** (manual cap. 4, §12).

---

## 4. Jerarquías de unidades

Una jerarquía es **una conversión de un producto**: «1 unidad mayor = N unidades menores» (por ejemplo, 1 Caja = 10 Blísteres). El conjunto de conversiones de un producto forma un grafo que el sistema recorre para saber cuántas unidades base contiene cualquier unidad.

### HU-CAT-005 — Registrar y editar la conversión entre dos unidades de un producto

**Como** administrador **quiero** definir cuántas unidades menores entran en una mayor para un producto (por ejemplo 1 caja = 10 blísteres) **para** poder crear presentaciones de venta con su equivalencia correcta.

| | |
|---|---|
| **Módulo** | Jerarquías de unidades |
| **Pantalla** | `/jerarquias/nueva` y `/jerarquias/:id` — «Jerarquías de Unidades» |
| **Endpoint(s)** | `POST /api/v1/JerarquiasUoM`, `PUT /api/v1/JerarquiasUoM/{id}`, `GET /api/v1/JerarquiasUoM/{id}` |
| **Permiso** | `JerarquiasUoM:Crear` (registrar), `JerarquiasUoM:Actualizar` (editar), `JerarquiasUoM:Leer`; para elegir producto y unidades, además `Medicamentos:Leer` y `UnidadesMedida:Leer` |
| **Reglas** | [RN-JER-001, RN-JER-002, RN-JER-003, RN-JER-004, RN-JER-005, RN-JER-006, RN-JER-007, RN-JER-008, RN-JER-009, RN-JER-010, RN-JER-012, RN-JER-013](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (varios rechazos del servidor llegan como mensaje genérico) |

**Criterios de aceptación**

```gherkin
Escenario: Registrar una conversión
  Dado que tengo el permiso "Jerarquías de Unidades › Crear" y existe el medicamento "Paracetamol Demo 500 mg"
  Cuando hago clic en "Nuevo"
  Y escribo "Paracetamol" en "Producto" y elijo "Paracetamol Demo 500 mg" de la lista
  Y elijo "Caja" en "Unidad Mayor" y "Blíster" en "Unidad Menor"
  Y escribo 10 en "Multiplicador" ("Cuántas unidades menores equivalen a una unidad mayor")
  Y hago clic en "Guardar"
  Entonces veo "Jerarquía guardada correctamente" y vuelvo a "/jerarquias"

Escenario: Hay que elegir las opciones de la lista
  Dado que escribí un texto en "Producto" o en una unidad sin elegir ninguna opción
  Cuando hago clic en "Guardar"
  Entonces debajo del campo veo "Este campo es obligatorio" y no se envía nada
  Y si no hay coincidencias la lista muestra una opción deshabilitada "Sin resultados"

Escenario: Unidad mayor igual a la menor
  Dado que elegí "Caja" en las dos unidades
  Entonces debajo del formulario veo "La unidad mayor y la unidad menor no pueden ser la misma"
  Y "Guardar" no envía nada

Escenario: Multiplicador inválido
  Dado que dejo el multiplicador vacío, en 0 o por debajo de 0,0001
  Cuando hago clic en "Guardar"
  Entonces veo "El multiplicador debe ser mayor que cero" y no se envía nada
  Y el valor se guarda con 4 decimales

Escenario: Conversión repetida
  Dado que el producto ya tiene la conversión "Caja" a "Blíster"
  Cuando registro otra con las mismas unidades y el mismo sentido
  Entonces la API responde 409 "JerarquiaUoM.Duplicada"
  Y veo "Este producto ya tiene una conversión entre esas unidades."

Escenario: La conversión es coherente con las existentes
  Dado que el producto tiene "Caja a Blíster = 10" y "Blíster a Unidad = 10"
  Cuando registro "Caja a Unidad = 100"
  Entonces se acepta, porque coincide con la equivalencia calculada (1 caja = 10 blísteres = 100 unidades)

Escenario: La conversión contradice a las existentes
  Dado que el producto tiene "Caja a Blíster = 10" y "Blíster a Unidad = 10"
  Cuando registro "Caja a Unidad = 50"
  Entonces la API responde 409 "JerarquiaUoM.Multiplicador.Inconsistente" (compara ambos valores redondeados a 4 decimales)
  Y veo "Error al guardar la jerarquía" sin el valor calculado

Escenario: Producto o unidad que ya no existen
  Dado que otra persona eliminó el producto o una de las unidades
  Cuando hago clic en "Guardar"
  Entonces la API responde 404 "Medicamento.NotFound" o "UnidadMedida.NotFound"
  Y veo "Error al guardar la jerarquía"

Escenario: Editar una conversión
  Dado que abrí una jerarquía y tengo "Jerarquías de Unidades › Actualizar"
  Cuando hago clic en "Editar", cambio el multiplicador y hago clic en "Guardar"
  Entonces veo "Jerarquía guardada correctamente"
  Y a diferencia de otros catálogos, vuelvo a "/jerarquias" en vez de quedarme en el detalle
  Y al editar la API solo revalida el producto o la unidad que cambió, y vuelve a comprobar que no se duplique ni se contradiga

Escenario: La jerarquía ya no existe
  Dado que fue eliminada
  Cuando intento guardarla o abrirla
  Entonces la API responde 404 "JerarquiaUoM.NotFound"
  Y al guardar veo "Error al guardar la jerarquía" y al abrir "Error al cargar la jerarquía"

Escenario: Identificador al editar (solo API)
  Dado que llamo a "PUT /api/v1/JerarquiasUoM/{id}"
  Entonces el identificador se toma de la URL y uno vacío se rechaza con "UpdateJerarquiaUoMCommand.Id"

Escenario: Cálculo de equivalencias
  Dado que un producto tiene varias conversiones
  Entonces de mayor a menor se multiplica y de menor a mayor se divide
  Y el sistema recorre las conversiones por anchura y devuelve el factor del primer camino que conecta las dos unidades, 1 si son la misma unidad y "sin resultado" si no hay camino
```

> **Brecha conocida.** Los códigos `JerarquiaUoM.Multiplicador.Inconsistente`, `Medicamento.NotFound` y `UnidadMedida.NotFound` no tienen traducción: el usuario ve «Error al guardar la jerarquía» ([19c §2.2](../../ola-1-fundamentos/19-inventario-de-pantallas/19c-catalogos-1.md)). El código del validador de unidades iguales termina en punto (`CreateJerarquiaUoMCommand.`) porque la regla se declara sobre el objeto completo. **No verificado en la aplicación:** el manual no recorrió esta pantalla; los mensajes salen de 19c §2.2 y 01b.

### HU-CAT-006 — Buscar, consultar y exportar jerarquías de unidades

**Como** administrador **quiero** buscar las conversiones por producto, unidades y multiplicador **para** revisar cómo está configurado cada producto.

| | |
|---|---|
| **Módulo** | Jerarquías de unidades |
| **Pantalla** | `/jerarquias` — «Jerarquías de Unidades» (tarjeta «Búsqueda de Jerarquías») |
| **Endpoint(s)** | `GET /api/v1/JerarquiasUoM`, `POST /api/v1/JerarquiasUoM/exportar` |
| **Permiso** | `JerarquiasUoM:Leer` (y `Medicamentos:Leer`, `UnidadesMedida:Leer` para las sugerencias de los filtros) |
| **Reglas** | [RN-JER-001, RN-JER-015, RN-JER-016](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Filtros disponibles
  Dado que abro "/jerarquias"
  Entonces veo "Producto" ("Buscar producto..."), "Unidad Mayor" y "Unidad Menor" ("Buscar unidad de medida..."), todos con lupa y sugerencias que aparecen al escribir (espera de 300 ms, hasta 20)
  Y un deslizador "Multiplicador: 0 — 1000" con paso 1

Escenario: Los filtros de texto buscan por nombre
  Dado que escribo "Paracetamol" en "Producto" sin elegir una sugerencia
  Cuando hago clic en "Buscar"
  Entonces el filtro vale igual: busca nombres que contengan el texto, sin distinguir mayúsculas

Escenario: Filtrar por multiplicador
  Dado que muevo el deslizador a 5 — 20
  Cuando hago clic en "Buscar"
  Entonces veo solo las conversiones con multiplicador entre 5 y 20, ambos inclusive
  Y solo se envía el límite que moví: el mínimo si es mayor que 0, el máximo si es menor que 1000

Escenario: Tabla de resultados
  Dado que hay resultados
  Entonces veo "Producto" (enlace al detalle), "Unidad Mayor", "Unidad Menor" y "Multiplicador" (a la derecha), ordenadas por nombre comercial del producto
  Y no aparecen las conversiones cuyo producto o cuyas unidades fueron eliminados

Escenario: Sin resultados
  Cuando ningún registro cumple los filtros
  Entonces la tabla muestra "Sin resultados"

Escenario: Exportar
  Dado que busqué
  Cuando hago clic en "Exportar"
  Entonces se descarga "jerarquias-unidades.xlsx" con las columnas "Producto", "Unidad Mayor", "Unidad Menor" y "Multiplicador", aplicando los filtros de nombre y de multiplicador que tengo escritos
  Y si falla la búsqueda o la exportación veo "Error al buscar jerarquías de unidades"
```

> **Nota.** El deslizador usa paso 1: no permite filtrar por multiplicadores con decimales ni por valores mayores a 1000 (con el fin en 1000 el límite superior no se envía). Si falta `Medicamentos:Leer` o `UnidadesMedida:Leer` las sugerencias fallan sin aviso (qué ve el usuario exactamente: **No verificado**, [19c §7](../../ola-1-fundamentos/19-inventario-de-pantallas/19c-catalogos-1.md)).

### HU-CAT-007 — Eliminar una conversión de unidades

**Como** administrador **quiero** eliminar una conversión errónea **para** poder volver a definirla.

| | |
|---|---|
| **Módulo** | Jerarquías de unidades |
| **Pantalla** | `/jerarquias/:id` (botón «Eliminar» en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/JerarquiasUoM/{id}` |
| **Permiso** | `JerarquiasUoM:Eliminar` |
| **Reglas** | [RN-JER-001, RN-JER-013, RN-JER-014](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-003](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (sin validación de dependencias) |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar una conversión
  Dado que estoy editando una jerarquía y tengo "Jerarquías de Unidades › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces se abre "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar esta jerarquía?"
  Y al hacer clic en "Confirmar" veo "Jerarquía eliminada correctamente" y vuelvo a "/jerarquias"

Escenario: La API no valida dependencias
  Dado que una presentación de venta usa la equivalencia de esta conversión
  Cuando confirmo la eliminación
  Entonces la conversión se elimina igualmente (la API responde 204) y la presentación conserva el valor ya guardado

Escenario: Aviso de "en uso" ante un conflicto
  Dado que la API responde 409 por cualquier causa (por ejemplo, un conflicto de concurrencia)
  Entonces veo "No se puede eliminar la jerarquía porque está en uso."

Escenario: La jerarquía ya no existe
  Dado que fue eliminada antes
  Cuando confirmo
  Entonces la API responde 404 "JerarquiaUoM.NotFound"
  Y veo "Error al eliminar la jerarquía"
```

> **Brecha conocida.** El mensaje «en uso» está preparado en la pantalla, pero el backend nunca devuelve 409 de negocio al eliminar una jerarquía ([RN-JER-014](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md)). **No verificado:** si modificar o eliminar una jerarquía recalcula las presentaciones ya guardadas ([19d §7](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md), No verificado 4); por el código leído, la equivalencia de una presentación se guarda al crearla o editarla y no se recalcula ([RN-PRE-007](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md)).

### HU-CAT-008 — Cargar jerarquías de unidades desde un Excel

**Como** administrador **quiero** cargar muchas conversiones de una sola vez **para** configurar rápido los productos de mi catálogo.

| | |
|---|---|
| **Módulo** | Jerarquías de unidades (carga masiva) |
| **Pantalla** | `/jerarquias/carga-masiva` — «Carga Masiva» |
| **Endpoint(s)** | `GET /api/v1/JerarquiasUoM/plantilla`, `POST /api/v1/JerarquiasUoM/previsualizar`, `POST /api/v1/JerarquiasUoM/carga-masiva` |
| **Permiso** | `JerarquiasUoM:Crear` (además de `Leer`) |
| **Reglas** | [RN-JER-002, RN-JER-003, RN-JER-004, RN-JER-005, RN-JER-008, RN-JER-009, RN-JER-017, RN-JER-018, RN-JER-019, RN-JER-020, RN-JER-021, RN-JER-022](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-IMP-015](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (la vista previa no anticipa duplicados ni contradicciones) |

**Criterios de aceptación**

Cumple los escenarios CM-1 a CM-3 y CM-5 de la [sección 1.3](#13-escenarios-comunes-de-la-carga-masiva). Además:

```gherkin
Escenario: Columnas de la plantilla
  Dado que descargué "plantilla-jerarquias.xlsx"
  Entonces tiene las columnas "Producto", "UnidadMayor", "UnidadMenor" y "Multiplicador", todas obligatorias
  Y los límites por fila son: Producto 150 caracteres, UnidadMayor y UnidadMenor 50, Multiplicador 30

Escenario: El producto y las unidades se identifican por nombre
  Dado que escribo "Paracetamol Demo 500 mg" en "Producto" y "Caja" y "Blíster" en las unidades
  Entonces el producto se busca por su nombre comercial y las unidades por su descripción (no por código), sin distinguir mayúsculas
  Y un valor que no existe muestra "El valor '<valor>' del campo 'producto' no existe en el sistema"
  Y un nombre que coincide con varios registros muestra "El valor '<valor>' del campo 'unidadMayor' coincide con varios registros del sistema"

Escenario: Reglas por fila en la vista previa
  Dado que subí un archivo
  Entonces un campo vacío muestra "El campo '<campo>' es obligatorio"
  Y unidades iguales muestran "'unidadMenor' no puede ser igual a 'unidadMayor'"
  Y un multiplicador no numérico muestra "'multiplicador' debe ser un número decimal válido"
  Y un multiplicador cero o negativo muestra "'multiplicador' debe ser mayor que cero"
  Y el multiplicador se lee con punto como separador decimal

Escenario: La vista previa no anticipa duplicados ni contradicciones
  Dado que una fila repite una conversión existente o contradice un multiplicador ya calculable
  Entonces la vista previa la muestra como válida
  Pero al guardar el servidor procesa las filas en orden, tiene en cuenta las aceptadas antes en el mismo archivo y descarta sin error la que repite o contradice
  Y veo "Se guardaron N registros correctamente" con N menor que las filas válidas
```

> **Brecha conocida.** La vista previa de jerarquías no revisa duplicados ni consistencia de multiplicadores, ni contra la base ni dentro del archivo ([RN-JER-021](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md)). **No verificado:** con cultura invariante .NET podría tomar la coma como separador de miles (`1,5` se leería como 15); conviene probarlo. El manual no recorrió esta carga.

### HU-CAT-009 — Conocer en qué unidades se puede vender un producto

**Como** administrador (y como pantalla de presentaciones de venta) **quiero** saber qué unidades distintas de la base permiten vender un producto y cuántas unidades base contiene cada una **para** ofrecer solo presentaciones que se pueden convertir a la unidad base.

| | |
|---|---|
| **Módulo** | Jerarquías de unidades |
| **Pantalla** | Sin pantalla propia: alimenta el campo «Unidad de Venta» de `/presentaciones-venta/nueva` y del diálogo «Presentaciones de Venta» |
| **Endpoint(s)** | `GET /api/v1/JerarquiasUoM/unidades-vendibles?productoId=<guid>` |
| **Permiso** | `JerarquiasUoM:Leer` |
| **Reglas** | [RN-JER-010, RN-JER-011](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (consumida por Presentaciones de venta; ninguna pantalla de jerarquías la llama) |

**Criterios de aceptación**

```gherkin
Escenario: Unidades vendibles de un producto
  Dado que el producto "Paracetamol Demo 500 mg" tiene unidad base "Unidad" y las conversiones "Caja a Blíster = 10" y "Blíster a Unidad = 10"
  Cuando se consultan sus unidades vendibles
  Entonces la respuesta lista "Caja" con 100 unidades base y "Blíster" con 10
  Y no incluye la unidad base, porque vender por la unidad base no necesita presentación
  Y se ordena de mayor a menor cantidad de unidades base

Escenario: Unidades sin camino hasta la base
  Dado que una unidad aparece en una conversión pero no se puede convertir hasta la unidad base
  Entonces no figura entre las unidades vendibles

Escenario: Producto sin jerarquías
  Dado que el producto no tiene ninguna conversión
  Entonces la lista está vacía
  Y la pantalla de presentaciones muestra "Este producto no tiene una Jerarquía de Unidades registrada. Cárgala primero para poder definir presentaciones de venta."

Escenario: El producto no existe
  Dado que el producto fue eliminado o es de otra empresa
  Entonces la API responde 404 "Medicamento.NotFound"
```

---

## 5. Laboratorios

### HU-CAT-010 — Registrar y editar un laboratorio

**Como** administrador **quiero** registrar los laboratorios fabricantes o titulares **para** asignarlos a mis medicamentos.

| | |
|---|---|
| **Módulo** | Laboratorios |
| **Pantalla** | `/laboratorios/nueva` y `/laboratorios/:id` — «Laboratorios» |
| **Endpoint(s)** | `POST /api/v1/Laboratorios`, `PUT /api/v1/Laboratorios/{id}`, `GET /api/v1/Laboratorios/{id}` |
| **Permiso** | `Laboratorios:Crear` (registrar), `Laboratorios:Actualizar` (editar), `Laboratorios:Leer` |
| **Reglas** | [RN-LAB-001, RN-LAB-002, RN-LAB-003, RN-LAB-004, RN-LAB-005, RN-LAB-006, RN-LAB-007, RN-LAB-008, RN-LAB-009, RN-LAB-011](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Registrar un laboratorio con código
  Dado que tengo "Laboratorios › Crear" y hago clic en "Nuevo"
  Cuando escribo Nombre de Compañía "Laboratorio Demo Andino" y Código Identificador "DEMO-AND"
  Y hago clic en "Guardar"
  Entonces veo "Laboratorio guardado correctamente" y vuelvo a la búsqueda

Escenario: El código identificador es opcional
  Dado que escribo Nombre de Compañía "Laboratorio Demo Pacífico" y dejo el código vacío
  Cuando hago clic en "Guardar"
  Entonces se guarda con "Laboratorio guardado correctamente"
  Y el laboratorio queda "sin código" (se ve "—" en la tabla) y puedo tener varios así

Escenario: Nombre obligatorio y longitudes
  Dado que dejo el nombre vacío
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" debajo de "Nombre de Compañía" y no se envía nada
  Y el nombre no deja pasar de 150 caracteres y el código no deja pasar de 50, sin mensaje

Escenario: Nombre repetido
  Dado que existe "Laboratorio Demo Andino"
  Cuando intento registrar "laboratorio demo andino"
  Entonces veo "Ya existe un laboratorio con ese nombre de compañía."
  Y la comprobación de nombre se evalúa antes que la de código

Escenario: Código repetido
  Dado que "DEMO-AND" ya lo usa otro laboratorio
  Cuando intento registrar "Laboratorio Otro" con el código "DEMO-AND"
  Entonces veo "Ya existe un laboratorio con ese código identificador."
  Y los laboratorios sin código no entran en la comprobación

Escenario: Editar un laboratorio y quitarle el código
  Dado que abrí un laboratorio con código "PRUEBA-01" y tengo "Laboratorios › Actualizar"
  Cuando hago clic en "Editar", borro el código, cambio el nombre y hago clic en "Guardar"
  Entonces veo "Laboratorio guardado correctamente", me quedo en el detalle y el código aparece vacío
  Y un código vacío o de solo espacios se guarda como "sin código"

Escenario: Nombre y código de un laboratorio eliminado quedan libres
  Dado que eliminé un laboratorio
  Cuando registro otro con el mismo nombre y el mismo código
  Entonces se guarda sin conflicto

Escenario: El laboratorio ya no existe
  Dado que fue eliminado
  Cuando intento guardarlo o abrirlo
  Entonces la API responde 404 "Laboratorio.NotFound"
  Y al guardar veo "Error al guardar el laboratorio" y al abrir "Error al cargar el laboratorio"

Escenario: Identificador al editar (solo API)
  Dado que llamo a "PUT /api/v1/Laboratorios/{id}"
  Entonces el identificador se toma de la URL y uno vacío se rechaza con "UpdateLaboratorioCommand.Id"
```

> **Brecha conocida.** Un nombre de solo espacios muestra «Error al guardar el laboratorio» (comprobado en el manual, §6.3). La comparación de nombre y código al crear y editar usa igualdad de SQL; el manual comprobó que el nombre en minúsculas se rechaza, pero la intercalación de la base no está definida en los scripts (**No verificado** en el código, [01b nota LAB](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md#6-laboratorios-lab)).

### HU-CAT-011 — Buscar, consultar y exportar laboratorios

**Como** administrador **quiero** buscar laboratorios por nombre o código y exportar la lista **para** ubicar rápido un laboratorio.

| | |
|---|---|
| **Módulo** | Laboratorios |
| **Pantalla** | `/laboratorios` — «Laboratorios» (tarjeta «Búsqueda de Laboratorios») |
| **Endpoint(s)** | `GET /api/v1/Laboratorios`, `POST /api/v1/Laboratorios/exportar` |
| **Permiso** | `Laboratorios:Leer` |
| **Reglas** | [RN-LAB-001, RN-LAB-012](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Buscar sin filtros
  Dado que existen dos laboratorios
  Cuando hago clic en "Buscar"
  Entonces veo "Resultados (2)": "Laboratorio Demo Andino" (DEMO-AND) y "Laboratorio Demo Pacífico" ("—")
  Y la tabla tiene "Nombre de Compañía" (enlace al detalle) y "Código Identificador", ordenada por nombre

Escenario: Filtrar
  Cuando escribo "demo" en "Nombre de Compañía"
  Entonces veo los dos
  Y "and" en "Código Identificador" trae solo "Laboratorio Demo Andino"
  Y "zzz" muestra "Resultados (0)", la fila "Resultados" y el paginador "Sin resultados"

Escenario: Exportar
  Dado que busqué
  Cuando hago clic en "Exportar"
  Entonces se descarga "laboratorios.xlsx" con las columnas "Nombre de Compañía" y "Código Identificador" (vacío si no tiene), con los filtros escritos
  Y si falla la búsqueda o la exportación veo "Error al buscar laboratorios"
```

### HU-CAT-012 — Eliminar un laboratorio

**Como** administrador **quiero** eliminar un laboratorio que ya no uso, sin dejar medicamentos huérfanos **para** mantener coherente el catálogo.

| | |
|---|---|
| **Módulo** | Laboratorios |
| **Pantalla** | `/laboratorios/:id` (botón «Eliminar» en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/Laboratorios/{id}` |
| **Permiso** | `Laboratorios:Eliminar` |
| **Reglas** | [RN-LAB-001, RN-LAB-009, RN-LAB-010, RN-LAB-011](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar un laboratorio sin medicamentos
  Dado que estoy editando "Laboratorio Prueba Borrar" y tengo "Laboratorios › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces veo "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar este laboratorio?"
  Y al hacer clic en "Confirmar" veo "Laboratorio eliminado correctamente" y vuelvo a la búsqueda

Escenario: Laboratorio con medicamentos
  Dado que algún medicamento no eliminado pertenece al laboratorio
  Cuando confirmo la eliminación
  Entonces la API responde 409 "Laboratorio.InUse"
  Y veo "No se puede eliminar el laboratorio porque tiene medicamentos asociados."

Escenario: Otro fallo al eliminar
  Dado que el laboratorio ya no existe (404 "Laboratorio.NotFound") o el servidor falla
  Entonces veo "Error al eliminar el laboratorio"
```

> **Brecha conocida.** La pantalla muestra «tiene medicamentos asociados» ante cualquier 409. **No verificado en la aplicación:** el bloqueo con medicamentos (el manual lo dejó pendiente por no existir aún medicamentos; viene de 19c §3.2).

### HU-CAT-013 — Cargar laboratorios desde un Excel

**Como** administrador **quiero** cargar varios laboratorios de una vez **para** no crearlos uno por uno.

| | |
|---|---|
| **Módulo** | Laboratorios (carga masiva) |
| **Pantalla** | `/laboratorios/carga-masiva` — «Carga Masiva» |
| **Endpoint(s)** | `GET /api/v1/Laboratorios/plantilla`, `POST /api/v1/Laboratorios/previsualizar`, `POST /api/v1/Laboratorios/carga-masiva` |
| **Permiso** | `Laboratorios:Crear` (además de `Leer`) |
| **Reglas** | [RN-LAB-002, RN-LAB-003, RN-LAB-004, RN-LAB-006, RN-LAB-007, RN-LAB-013, RN-LAB-014, RN-LAB-015, RN-LAB-016, RN-LAB-017](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (la vista previa no revisa el código repetido) |

**Criterios de aceptación**

Cumple los escenarios CM-1 a CM-5 de la [sección 1.3](#13-escenarios-comunes-de-la-carga-masiva). Además:

```gherkin
Escenario: Columnas de la plantilla
  Dado que descargué "plantilla-laboratorios.xlsx"
  Entonces tiene "NombreCompania" (obligatoria, máximo 150 caracteres) y "CodigoIdentificador" (opcional, máximo 50)

Escenario: Reglas por fila en la vista previa
  Entonces un nombre que ya existe muestra "El valor 'Laboratorio Demo Andino' del campo 'nombreCompania' ya existe en el sistema"
  Y un nombre vacío muestra "El campo 'nombreCompania' es obligatorio"
  Y un nombre de 160 letras muestra "'nombreCompania' no puede superar los 150 caracteres"
  Y un nombre repetido dentro del archivo se marca en todas sus filas

Escenario: Un código identificador ya usado pasa la vista previa y se omite al guardar
  Dado que la fila "Laboratorio Demo Código Repetido" con código "DEMO-AND" parece válida porque la vista previa no revisa el código
  Cuando hago clic en "Guardar válidos (1)"
  Entonces el servidor omite la fila porque "DEMO-AND" ya lo usa otro laboratorio
  Y veo "Se guardaron 0 registros correctamente" y vuelvo a la búsqueda de laboratorios
```

> **Brecha conocida.** Mensaje de éxito con cero registros guardados: comprobado en el manual (§9.3 y §11, diferencia 2) y descrito en [RN-LAB-015](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md).

---

## 6. Ingredientes activos

### HU-CAT-014 — Registrar y editar un ingrediente activo

**Como** administrador **quiero** registrar los principios activos con su denominación DCI y su código ATC **para** vincularlos a los medicamentos mediante formulaciones clínicas.

| | |
|---|---|
| **Módulo** | Ingredientes activos |
| **Pantalla** | `/ingredientes-activos/nueva` y `/ingredientes-activos/:id` — «Ingredientes Activos» |
| **Endpoint(s)** | `POST /api/v1/IngredientesActivos`, `PUT /api/v1/IngredientesActivos/{id}`, `GET /api/v1/IngredientesActivos/{id}` |
| **Permiso** | `IngredientesActivos:Crear`, `IngredientesActivos:Actualizar`, `IngredientesActivos:Leer` |
| **Reglas** | [RN-ING-001, RN-ING-002, RN-ING-003, RN-ING-004, RN-ING-005, RN-ING-006, RN-ING-007, RN-ING-008](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Registrar un ingrediente
  Dado que tengo "Ingredientes Activos › Crear" y hago clic en "Nuevo"
  Cuando escribo Denominación DCI "Paracetamol" y Código ATC "N02BE01"
  Y hago clic en "Guardar"
  Entonces veo "Ingrediente activo guardado correctamente" y vuelvo a la búsqueda

Escenario: Campos obligatorios y longitudes
  Dado que dejo vacíos los dos campos
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" bajo cada campo y no se envía nada
  Y "Denominación DCI" no deja pasar de 255 caracteres y "Código ATC" de 15, sin mensaje

Escenario: La DCI no puede repetirse
  Dado que existe "Paracetamol"
  Cuando intento registrar "paracetamol" con otro código ATC
  Entonces veo "Ya existe un ingrediente activo con esa denominación DCI."

Escenario: El código ATC no se valida ni es único
  Dado que escribo el código "XXX" o un código que ya usa otro ingrediente
  Cuando hago clic en "Guardar"
  Entonces se guarda con "Ingrediente activo guardado correctamente", porque cualquier texto de hasta 15 caracteres es aceptado

Escenario: Editar un ingrediente
  Dado que abrí "Ingrediente Prueba Borrar" y tengo "Ingredientes Activos › Actualizar"
  Cuando hago clic en "Editar", cambio el código ATC a "XXX" y hago clic en "Guardar"
  Entonces veo "Ingrediente activo guardado correctamente" y me quedo en el detalle

Escenario: El ingrediente ya no existe
  Dado que fue eliminado
  Cuando intento guardarlo o abrirlo
  Entonces la API responde 404 "IngredienteActivo.NotFound"
  Y al guardar veo "Error al guardar el ingrediente activo" y al abrir "Error al cargar el ingrediente activo"

Escenario: Identificador al editar (solo API)
  Dado que llamo a "PUT /api/v1/IngredientesActivos/{id}"
  Entonces el identificador se toma de la URL y uno vacío se rechaza con "UpdateIngredienteActivoCommand.Id"
```

> **Brecha conocida.** Un valor de solo espacios muestra «Error al guardar el ingrediente activo» (comprobado en el manual, §7.3). Los códigos ATC del manual se escribieron de conocimiento del autor y no se contrastaron con una fuente oficial (manual cap. 4, §12 punto 8).

### HU-CAT-015 — Buscar, consultar y exportar ingredientes activos

**Como** administrador **quiero** buscar ingredientes por DCI o por código ATC **para** encontrar el principio activo que necesito.

| | |
|---|---|
| **Módulo** | Ingredientes activos |
| **Pantalla** | `/ingredientes-activos` — «Ingredientes Activos» (tarjeta «Búsqueda de Ingredientes Activos») |
| **Endpoint(s)** | `GET /api/v1/IngredientesActivos`, `POST /api/v1/IngredientesActivos/exportar` |
| **Permiso** | `IngredientesActivos:Leer` |
| **Reglas** | [RN-ING-001, RN-ING-010](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Buscar con filtros combinados
  Dado que existen Paracetamol (N02BE01), Amoxicilina (J01CA04), Clonazepam (N03AE01) y Tramadol (N02AX02)
  Cuando escribo "CIL" en "Denominación DCI" y busco
  Entonces veo "Amoxicilina"
  Y "n02" en "Código ATC" trae los ingredientes cuyo código contiene "N02"
  Y "para" junto con "n02" trae solo "Paracetamol"
  Y la tabla tiene "Denominación DCI" (enlace al detalle) y "Código ATC", ordenada por denominación

Escenario: Sin coincidencias
  Cuando ningún ingrediente cumple
  Entonces veo "Resultados (0)", la fila "Resultados" y el paginador "Sin resultados"

Escenario: Exportar
  Cuando hago clic en "Exportar"
  Entonces se descarga "ingredientes-activos.xlsx" con las columnas "Denominación DCI" y "Código ATC"
  Y si falla la búsqueda o la exportación veo "Error al buscar ingredientes activos"
```

### HU-CAT-016 — Eliminar un ingrediente activo

**Como** administrador **quiero** eliminar un ingrediente que no uso **para** limpiar el catálogo sin dejar formulaciones apuntando a él.

| | |
|---|---|
| **Módulo** | Ingredientes activos |
| **Pantalla** | `/ingredientes-activos/:id` (botón «Eliminar» en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/IngredientesActivos/{id}` |
| **Permiso** | `IngredientesActivos:Eliminar` |
| **Reglas** | [RN-ING-001, RN-ING-008, RN-ING-009](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar un ingrediente sin formulaciones
  Dado que estoy editando "Ingrediente Prueba Borrar" y tengo "Ingredientes Activos › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces veo "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar este ingrediente activo?"
  Y al hacer clic en "Confirmar" veo "Ingrediente activo eliminado correctamente" y vuelvo a la búsqueda
  Y la API responde 204

Escenario: Ingrediente incluido en formulaciones clínicas
  Dado que alguna formulación clínica no eliminada usa el ingrediente
  Cuando confirmo la eliminación
  Entonces la API responde 409 "IngredienteActivo.InUse" (bloqueo duro)
  Y veo "No puedes eliminar este ingrediente activo porque está incluido en formulaciones clínicas."

Escenario: Otro fallo al eliminar
  Dado que el ingrediente ya no existe (404 "IngredienteActivo.NotFound") o el servidor falla
  Entonces veo "Error al eliminar el ingrediente activo"
```

> **Brecha conocida.** La pantalla muestra el texto de «formulaciones clínicas» ante cualquier 409. **No verificado en la aplicación:** el bloqueo con formulaciones (pendiente en el manual por no existir aún formulaciones; viene de 19c §4.2).

### HU-CAT-017 — Cargar ingredientes activos desde un Excel

**Como** administrador **quiero** cargar varios ingredientes de una vez **para** completar rápido el catálogo.

| | |
|---|---|
| **Módulo** | Ingredientes activos (carga masiva) |
| **Pantalla** | `/ingredientes-activos/carga-masiva` — «Carga Masiva» |
| **Endpoint(s)** | `GET /api/v1/IngredientesActivos/plantilla`, `POST /api/v1/IngredientesActivos/previsualizar`, `POST /api/v1/IngredientesActivos/carga-masiva` |
| **Permiso** | `IngredientesActivos:Crear` (además de `Leer`) |
| **Reglas** | [RN-ING-002, RN-ING-003, RN-ING-004, RN-ING-005, RN-ING-006, RN-ING-011, RN-ING-012, RN-ING-013, RN-ING-014, RN-ING-015](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (la vista previa valida una clave distinta de la que valida el guardado) |

**Criterios de aceptación**

Cumple los escenarios CM-1 a CM-3 y CM-5 de la [sección 1.3](#13-escenarios-comunes-de-la-carga-masiva). Además:

```gherkin
Escenario: Columnas de la plantilla
  Dado que descargué "plantilla-ingredientes-activos.xlsx"
  Entonces tiene "DenominacionDci" (obligatoria, máximo 255) y "CodigoAtc" (obligatoria, máximo 15)

Escenario: Reglas por fila en la vista previa
  Entonces un campo vacío muestra "El campo 'denominacionDci' es obligatorio" o "El campo 'codigoAtc' es obligatorio"
  Y un texto demasiado largo muestra "'denominacionDci' no puede superar los 255 caracteres" o "'codigoAtc' no puede superar los 15 caracteres"

Escenario: La vista previa controla el código ATC, no la DCI
  Dado que el Paracetamol ya usa "N02BE01" y subo una fila "Diazepam" con "N02BE01"
  Entonces la fila muestra "El valor 'N02BE01' del campo 'codigoAtc' ya existe en el sistema"
  Y dos filas con el mismo ATC "R03AC02" muestran "El valor 'R03AC02' del campo 'codigoAtc' está duplicado en el archivo"
  Pero crear un ingrediente a mano sí permite repetir el ATC

Escenario: Una DCI ya registrada pasa la vista previa y se omite al guardar
  Dado que subo una fila "Paracetamol" con el ATC "ZZ99"
  Entonces la fila aparece válida
  Cuando hago clic en "Guardar válidos (3)" con otras dos filas válidas
  Entonces el servidor omite la fila porque la DCI ya existe (y también la repetida en el archivo, conservando la primera)
  Y veo "Se guardaron 2 registros correctamente" y vuelvo a la búsqueda
```

> **Brecha conocida.** Contradicción del código: crear, editar, el índice único y el guardado de la carga tratan la **DCI** como clave única; la vista previa valida la unicidad del **ATC**. Un archivo con dos ingredientes distintos que comparten ATC queda bloqueado en la vista previa aunque la API individual los aceptaría; y uno con una DCI ya registrada pasa la vista previa y se omite al guardar ([RN-ING-012, RN-ING-013](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md); comprobado en el manual, §9.4 y §11, diferencia 1).

---

## 7. Medicamentos

El medicamento (también llamado «producto») es el artículo que la botica vende: código nacional, nombre comercial, laboratorio, unidad base, condición de venta y precio de venta base de una unidad base.

### HU-CAT-018 — Registrar y editar un medicamento

**Como** administrador **quiero** registrar los productos que vendo, con su laboratorio, su unidad base, su condición de venta y su precio **para** tenerlos disponibles en el inventario, las recetas y el punto de venta.

| | |
|---|---|
| **Módulo** | Medicamentos |
| **Pantalla** | `/medicamentos/nueva` y `/medicamentos/:id` — «Medicamentos» |
| **Endpoint(s)** | `POST /api/v1/Medicamentos`, `PUT /api/v1/Medicamentos/{id}`, `GET /api/v1/Medicamentos/{id}`, `GET /api/v1/Medicamentos/condiciones-venta` |
| **Permiso** | `Medicamentos:Crear` (registrar), `Medicamentos:Actualizar` (editar), `Medicamentos:Leer`; para elegir valores, además `Laboratorios:Leer` y `UnidadesMedida:Leer` |
| **Reglas** | [RN-MED-001, RN-MED-002, RN-MED-003, RN-MED-004, RN-MED-005, RN-MED-006, RN-MED-007, RN-MED-008, RN-MED-009, RN-MED-010, RN-MED-011, RN-MED-012, RN-MED-019](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Registrar un medicamento
  Dado que tengo "Medicamentos › Crear" y hago clic en "Nuevo"
  Cuando escribo Código Nacional "DEMO-MED-0001" y Nombre Comercial "Paracetamol Demo 500 mg"
  Y escribo "Andino" en "Laboratorio" y elijo "Laboratorio Demo Andino" de la lista
  Y escribo "Unidad" en "Unidad Base" y elijo "UND Unidad" de la lista
  Y elijo "Venta Libre (OTC)" en "Condición de Venta"
  Y escribo 0.50 en "Precio de Venta Base" (prefijo "S/")
  Y hago clic en "Guardar"
  Entonces veo "Medicamento guardado correctamente"
  Y vuelvo a la búsqueda, con los filtros vacíos y sin tabla

Escenario: La lista de condiciones de venta viene de la API
  Dado que abro el formulario
  Entonces "Condición de Venta" ofrece, en este orden, "Venta Libre (OTC)", "Receta Simple", "Receta Retenida" y "Estupefaciente"
  Y si la lista no carga veo "Error al cargar las condiciones de venta"

Escenario: Campos obligatorios vacíos
  Dado que el formulario está vacío
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" bajo Código Nacional, Nombre Comercial, Laboratorio, Unidad Base y Condición de Venta
  Y bajo "Precio de Venta Base" veo "El precio no puede ser negativo", también cuando el campo está vacío
  Y no se envía nada

Escenario: Hay que elegir el laboratorio y la unidad de la lista
  Dado que escribí "zzz" en "Laboratorio" sin elegir ninguna opción
  Cuando hago clic en "Guardar"
  Entonces debajo del campo veo "Este campo es obligatorio"
  Y si no hay coincidencias la lista muestra "Sin resultados"

Escenario: Precio negativo
  Dado que escribo -1 en "Precio de Venta Base"
  Cuando hago clic en "Guardar"
  Entonces veo "El precio no puede ser negativo" y no se envía nada
  Y el precio 0 es válido y se guarda con 2 decimales

Escenario: Longitudes máximas
  Dado que escribo en "Código Nacional" y "Nombre Comercial"
  Entonces el primero no deja pasar de 50 caracteres y el segundo de 150, sin mensaje

Escenario: Código nacional repetido
  Dado que existe "DEMO-MED-0001"
  Cuando intento registrar otro medicamento con el código "demo-med-0001"
  Entonces veo "Ya existe un medicamento con ese código nacional."
  Y me quedo en el formulario

Escenario: El nombre comercial puede repetirse
  Dado que existe "Paracetamol Demo 500 mg"
  Cuando registro otro medicamento con el mismo nombre y distinto código nacional
  Entonces se guarda

Escenario: Laboratorio o unidad que ya no existen
  Dado que otra persona eliminó el laboratorio o la unidad que elegí
  Cuando hago clic en "Guardar"
  Entonces la API responde 404 "Laboratorio.NotFound" o "UnidadMedida.NotFound"
  Y veo "Error al guardar el medicamento"

Escenario: Editar un medicamento
  Dado que abrí "Paracetamol Demo 500 mg" y tengo "Medicamentos › Actualizar"
  Cuando hago clic en "Editar", cambio el precio y hago clic en "Guardar"
  Entonces veo "Medicamento guardado correctamente", me quedo en el detalle y los datos se recargan
  Y si no cambié el laboratorio ni la unidad, la API no vuelve a comprobar su existencia

Escenario: Medicamento antiguo sin precio
  Dado que el medicamento se creó antes de existir el precio o por carga masiva y su precio está vacío
  Cuando hago clic en "Editar"
  Entonces el campo "Precio de Venta Base" aparece vacío y debo informarlo para poder guardar

Escenario: El medicamento ya no existe
  Dado que fue eliminado
  Cuando intento guardarlo o abrirlo
  Entonces la API responde 404 "Medicamento.NotFound"
  Y al guardar veo "Error al guardar el medicamento" y al abrir "Error al cargar el medicamento"

Escenario: Identificador al editar (solo API)
  Dado que llamo a "PUT /api/v1/Medicamentos/{id}"
  Entonces el identificador se toma de la URL y uno vacío se rechaza con "UpdateMedicamentoCommand.Id"
```

> **Brecha conocida.** (1) Código o nombre de solo espacios muestran «Error al guardar el medicamento» (comprobado en el manual, §4.4). (2) El mensaje de precio aparece también si el campo está vacío: no hay texto de «obligatorio» para ese campo. (3) **No verificado:** qué valor toma el precio si una petición directa a la API lo omite, y la respuesta cuando la condición de venta no se puede convertir al enum (falla antes del validador) ([01b §16.1](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md)). (4) **No verificado:** si elegir una sugerencia con el teclado (flechas y Enter) la registra; si no, al guardar la elección se descartaría ([19d §7](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md), No verificado 2).

**Qué implica cada condición de venta en el punto de venta.** La condición es obligatoria y decide si el producto se vende libremente (`Venta Libre (OTC)`), con receta (`Receta Simple`) o con receta registrada y con número (`Receta Retenida` y `Estupefaciente`); el recorrido de cada una al cobrar pertenece a la parte C y a la historia [HU-INV-017](#hu-inv-017--registrar-y-editar-una-receta-médica).

### HU-CAT-019 — Buscar y exportar medicamentos

**Como** administrador **quiero** buscar medicamentos por código, nombre, laboratorio, unidad base y condición de venta **para** encontrarlos y revisar el catálogo.

| | |
|---|---|
| **Módulo** | Medicamentos |
| **Pantalla** | `/medicamentos` — «Medicamentos» (tarjeta «Búsqueda de Medicamentos») |
| **Endpoint(s)** | `GET /api/v1/Medicamentos`, `POST /api/v1/Medicamentos/exportar`, `GET /api/v1/Medicamentos/condiciones-venta`, `GET /api/v1/Laboratorios` y `GET /api/v1/UnidadesMedida` (sugerencias) |
| **Permiso** | `Medicamentos:Leer` (y `Laboratorios:Leer`, `UnidadesMedida:Leer` para las sugerencias) |
| **Reglas** | [RN-MED-001, RN-MED-014, RN-MED-019, RN-MED-020](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Filtros disponibles
  Dado que abro "/medicamentos"
  Entonces veo "Código Nacional", "Nombre Comercial", "Laboratorio" ("Buscar laboratorio..."), "Unidad Base" ("Buscar unidad de medida...") y "Condición de Venta" ("— Todos —", "Venta Libre (OTC)", "Receta Simple", "Receta Retenida", "Estupefaciente")

Escenario: Buscar sin filtros
  Dado que existen los cuatro medicamentos del manual
  Cuando hago clic en "Buscar"
  Entonces veo "Resultados (4)" ordenados por nombre comercial: Amoxicilina, Clonazepam, Paracetamol y Tramadol
  Y la tabla tiene "Código Nacional" (enlace al detalle), "Nombre Comercial", "Laboratorio", "Unidad Base" y "Condición de Venta"
  Y no muestra precio ni stock

Escenario: Filtros que se combinan
  Dado que escribo "andino" en "Laboratorio" (aunque no elija una sugerencia)
  Entonces veo solo Clonazepam y Paracetamol, porque el laboratorio se filtra por nombre "contiene"
  Y "Código Nacional" "0001" trae solo Paracetamol; "Nombre Comercial" "DEMO 500" trae Amoxicilina y Paracetamol
  Y "Condición de Venta" "Receta Retenida" trae solo Clonazepam
  Y "Laboratorio" "pac" junto con "Estupefaciente" trae solo Tramadol

Escenario: Sin coincidencias
  Cuando busco el código "zzz"
  Entonces veo "Resultados (0)" y la fila "Sin resultados"

Escenario: Quedan fuera los medicamentos cuyo laboratorio o unidad fueron eliminados
  Dado que el laboratorio o la unidad base de un medicamento fueron eliminados
  Entonces ese medicamento no aparece en la lista

Escenario: Exportar
  Dado que busqué
  Cuando hago clic en "Exportar"
  Entonces se descarga "medicamentos.xlsx" con una hoja "Data" y las columnas "Código Nacional", "Nombre Comercial", "Laboratorio", "Unidad Base" y "Condición de Venta" (en texto, por ejemplo "Receta Simple")
  Y el archivo respeta los filtros escritos y no incluye precio ni stock
  Y si falla la búsqueda o la exportación veo "Error al buscar medicamentos"

Escenario: Falla la carga de condiciones de venta
  Dado que la lista de condiciones no cargó
  Entonces la búsqueda no avisa: el select queda solo con "— Todos —" y la columna de condición mostraría el número
```

### HU-CAT-020 — Consultar el detalle de un medicamento y su stock por sucursal

**Como** administrador o almacenero **quiero** ver todos los datos de un medicamento y cuánto stock tiene en cada sucursal **para** decidir si hay que reponer.

| | |
|---|---|
| **Módulo** | Medicamentos |
| **Pantalla** | `/medicamentos/:id` — tarjeta «Stock por Sucursal» |
| **Endpoint(s)** | `GET /api/v1/Medicamentos/{id}` |
| **Permiso** | `Medicamentos:Leer` |
| **Reglas** | [RN-MED-012, RN-MED-018](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Ver el stock total y por sucursal
  Dado que abrí un medicamento con lotes en dos sucursales
  Entonces, en modo ver, veo la tarjeta "Stock por Sucursal" con "Stock Total: N"
  Y una tabla con "Sucursal" y "Stock" (a la derecha) con la cantidad física sumada de sus lotes en cada sucursal

Escenario: Medicamento sin stock
  Dado que el medicamento no tiene lotes con existencias
  Entonces la tarjeta dice "Stock Total: 0" y "Sin stock registrado"

Escenario: La tarjeta no aparece al crear ni al editar
  Dado que estoy creando o editando
  Entonces la tarjeta "Stock por Sucursal" no se muestra

Escenario: El medicamento no existe
  Dado que abro "/medicamentos/<id inexistente>"
  Entonces la API responde 404 "Medicamento.NotFound"
  Y veo "Error al cargar el medicamento" con la ficha vacía y deshabilitada
```

> **Nota.** El stock del detalle lo calcula el servidor como la suma de la cantidad física de los lotes en cada sucursal ([RN-MED-018](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md)); la regla no describe ningún filtro por vencimiento ni por sucursales permitidas del usuario, y esa diferencia con «stock vendible» ([RN-STK-017](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md)) **no se verificó** en el código de la consulta. Si la carga del registro falla, la ficha queda vacía y, con `Actualizar`, sigue visible «Editar» ([19d §1.2](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md)).

### HU-CAT-021 — Eliminar un medicamento

**Como** administrador **quiero** eliminar un medicamento que ya no vendo, sin dañar lotes, presentaciones u otros registros que lo usan **para** mantener la integridad del inventario.

| | |
|---|---|
| **Módulo** | Medicamentos |
| **Pantalla** | `/medicamentos/:id` (botón «Eliminar» en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/Medicamentos/{id}` |
| **Permiso** | `Medicamentos:Eliminar` |
| **Reglas** | [RN-MED-001, RN-MED-012, RN-MED-013](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar un medicamento sin dependencias
  Dado que estoy editando un medicamento sin registros asociados y tengo "Medicamentos › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces veo "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar este medicamento?"
  Y al hacer clic en "Confirmar" veo "Medicamento eliminado correctamente" y vuelvo a "/medicamentos"
  Y la API responde 204

Escenario: Medicamento con registros asociados
  Dado que lo referencian lotes, presentaciones de venta, jerarquías de unidades, formulaciones clínicas, precios de proveedor, servicios de agenda, inmunizaciones u órdenes magistrales no eliminados
  Cuando confirmo la eliminación
  Entonces la API responde 409 "Medicamento.InUse" (se comprueban en ese orden)
  Y veo "No puedes eliminar este medicamento porque tiene lotes, presentaciones u otros registros asociados."

Escenario: Las ventas no bloquean la eliminación
  Dado que el medicamento aparece solo en ventas ya registradas
  Entonces la API no lo considera una dependencia y lo elimina

Escenario: Otro fallo al eliminar
  Dado que el medicamento ya no existe (404 "Medicamento.NotFound") o el servidor falla
  Entonces veo "Error al eliminar el medicamento"
```

> **Brecha conocida.** El texto «lotes, presentaciones u otros registros asociados» no dice cuál de las ocho causas aplica, y se muestra ante cualquier 409 ([19d §6](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md)). Que las ventas no se comprueben es un comportamiento del código ([RN-MED-013](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md)); **No verificado en la aplicación:** el bloqueo (el manual lo dejó fuera de su alcance).

### HU-CAT-022 — Cargar medicamentos desde un Excel

**Como** administrador **quiero** cargar muchos medicamentos de una vez **para** poblar el catálogo al empezar.

| | |
|---|---|
| **Módulo** | Medicamentos (carga masiva) |
| **Pantalla** | `/medicamentos/carga-masiva` — «Carga Masiva» |
| **Endpoint(s)** | `GET /api/v1/Medicamentos/plantilla`, `POST /api/v1/Medicamentos/previsualizar`, `POST /api/v1/Medicamentos/carga-masiva` |
| **Permiso** | `Medicamentos:Crear` (además de `Leer`) |
| **Reglas** | [RN-MED-002, RN-MED-003, RN-MED-004, RN-MED-007, RN-MED-010, RN-MED-021, RN-MED-022, RN-MED-023, RN-MED-024, RN-MED-025, RN-MED-026](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-IMP-015](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Parcial (la plantilla no tiene precio: los medicamentos cargados quedan sin precio de venta base) |

**Criterios de aceptación**

Cumple los escenarios CM-1 a CM-3 y CM-5 de la [sección 1.3](#13-escenarios-comunes-de-la-carga-masiva). Además:

```gherkin
Escenario: Columnas de la plantilla
  Dado que descargué "plantilla-medicamentos.xlsx"
  Entonces tiene "CodigoNacional" (50 caracteres), "NombreComercial" (150), "Laboratorio" (150), "UnidadBase" (50) y "CondicionVenta" (50), todas obligatorias
  Y no tiene columna de precio

Escenario: El laboratorio, la unidad y la condición se escriben como texto
  Dado que escribo "Laboratorio Demo Andino" en "Laboratorio", "Unidad" en "UnidadBase" y "Receta Simple" en "CondicionVenta"
  Entonces el laboratorio se busca por su nombre de compañía y la unidad por su descripción, sin distinguir mayúsculas
  Y la condición debe coincidir (sin distinguir mayúsculas) con "Venta Libre (OTC)", "Receta Simple", "Receta Retenida" o "Estupefaciente"
  Y un laboratorio o unidad inexistente muestra "El valor '<valor>' del campo 'laboratorio' no existe en el sistema"
  Y un nombre que coincide con varios registros muestra "El valor '<valor>' del campo 'unidadBase' coincide con varios registros del sistema"
  Y una condición no válida muestra "'<valor>' no es un valor válido para 'condicionVenta'"

Escenario: Código nacional repetido o existente
  Dado que el código ya existe, o aparece en dos filas del archivo
  Entonces la vista previa muestra "El valor '<valor>' del campo 'codigoNacional' ya existe en el sistema" o "…está duplicado en el archivo"
  Y al guardar, de las repetidas se guarda solo la primera

Escenario: Los medicamentos cargados quedan sin precio
  Dado que guardé filas válidas
  Entonces los medicamentos nacen con precio de venta base vacío
  Y para venderlos hay que asignarles precio (al editar) o una presentación con precio
```

> **Brecha conocida.** El formulario exige el precio pero la carga masiva no lo trae ([RN-MED-021](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md)); el [proceso 3](../14-procesos-de-negocio.md) advierte que sin precio no se pueden vender ([RN-VEN-019](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), parte C). El comentario de la entidad que afirma que los registros nuevos siempre llevan precio queda contradicho por esta carga. El manual no recorrió esta carga.

### HU-CAT-023 — Ofrecer medicamentos al punto de venta con stock y los más vendidos

**Como** cajero **quiero** que la lista de medicamentos del punto de venta muestre primero los más vendidos de mi sucursal y solo con stock vendible **para** cobrar más rápido.

| | |
|---|---|
| **Módulo** | Medicamentos (opciones del listado usadas por el punto de venta) |
| **Pantalla** | Sin pantalla en el catálogo: lo consume el punto de venta (parte C) |
| **Endpoint(s)** | `GET /api/v1/Medicamentos` con `IncluirStock`, `SoloStockVendibleEnMiSucursal` y `OrdenarPorMasVendidos` |
| **Permiso** | `Medicamentos:Leer` |
| **Reglas** | [RN-MED-015, RN-MED-016, RN-MED-017](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-STK-017](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) |
| **Estado** | Solo API en este catálogo (la pantalla de medicamentos no usa estas opciones; las usa el punto de venta, historias de la parte C) |

**Criterios de aceptación**

```gherkin
Escenario: Incluir el stock en el listado
  Dado que se pide el listado con "IncluirStock"
  Entonces el stock total de cada medicamento es la suma de la cantidad física de sus lotes en todas las sucursales de la empresa

Escenario: Solo stock vendible en mi sucursal
  Dado que se pide "SoloStockVendibleEnMiSucursal"
  Entonces solo cuenta el stock vendible de la sucursal del usuario: cantidad mayor que 0, lote no vencido y sin retención activa en cuarentena

Escenario: Más vendidos primero
  Dado que se pide "OrdenarPorMasVendidos"
  Entonces se ordena por unidades vendidas en la sucursal del usuario durante los últimos 30 días, contando solo ventas en estado "Completada"
  Y los no vendidos van después, y a igualdad se ordena por nombre comercial

Escenario: Sin sucursal en la sesión
  Dado que el usuario no tiene una sucursal en su sesión
  Cuando pide cualquiera de las opciones que dependen de la sucursal
  Entonces la API responde 400 "Auth.Sucursal"
```

> **Nota.** La pantalla de búsqueda de medicamentos nunca pide `IncluirStock` (la tabla no muestra stock). La definición de «stock vendible» está en [RN-STK-017](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md); el consumo en la venta es de la parte C. **No verificado en la aplicación:** estas opciones no se recorrieron en el manual.

---

## 8. Presentaciones de venta

Una presentación de venta define, para un producto, una unidad en la que se vende (por ejemplo «Caja»), cuántas unidades base contiene y su precio propio. La equivalencia no se escribe a mano: el servidor la calcula con las jerarquías del producto. No hay carga masiva para este módulo.

### HU-CAT-024 — Registrar y editar una presentación de venta

**Como** administrador **quiero** definir en qué unidades se vende un producto (caja, blíster…) y a qué precio **para** que el cajero venda una caja o un blíster completo sin calcular unidades.

| | |
|---|---|
| **Módulo** | Presentaciones de venta |
| **Pantalla** | `/presentaciones-venta/nueva` y `/presentaciones-venta/:id` — «Presentaciones de Venta» |
| **Endpoint(s)** | `POST /api/v1/PresentacionesVenta`, `PUT /api/v1/PresentacionesVenta/{id}`, `GET /api/v1/PresentacionesVenta/{id}`, `GET /api/v1/JerarquiasUoM/unidades-vendibles` |
| **Permiso** | `PresentacionesVenta:Crear`, `PresentacionesVenta:Actualizar`, `PresentacionesVenta:Leer`; además `Medicamentos:Leer` y `JerarquiasUoM:Leer` (y `JerarquiasUoM:Crear` para el atajo a jerarquías) |
| **Reglas** | [RN-PRE-001, RN-PRE-002, RN-PRE-003, RN-PRE-004, RN-PRE-005, RN-PRE-006, RN-PRE-007, RN-PRE-008, RN-PRE-009, RN-PRE-010, RN-PRE-011, RN-PRE-015](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-JER-011](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Registrar una presentación
  Dado que el producto "Paracetamol Demo 500 mg" tiene unidad base "Unidad" y las conversiones "Caja a Blíster = 10" y "Blíster a Unidad = 10"
  Y tengo "Presentaciones de Venta › Crear"
  Cuando hago clic en "Nuevo" y elijo el producto de la lista "Buscar producto..."
  Entonces aparecen "Unidad de Venta" y "Precio de Venta"
  Y "Unidad de Venta" ofrece "Caja (100 Unidad)" y "Blíster (10 Unidad)", sin la unidad base
  Cuando elijo "Caja (100 Unidad)" y escribo el precio 40.00
  Y hago clic en "Guardar"
  Entonces veo "Presentación de venta guardada correctamente" y vuelvo a "/presentaciones-venta"

Escenario: La equivalencia y la descripción las calcula el servidor
  Dado que guardé la presentación "Caja"
  Entonces el servidor guarda la equivalencia (100 unidades base, con 4 decimales) calculada con las jerarquías del producto
  Y la descripción de la presentación es la descripción de la unidad elegida
  Y el SPA no envía la equivalencia: el cuerpo lleva solo producto, unidad y precio

Escenario: Producto sin jerarquía
  Dado que elijo un producto que no tiene conversiones
  Entonces veo "Este producto no tiene una Jerarquía de Unidades registrada. Cárgala primero para poder definir presentaciones de venta."
  Y, con "Jerarquías de Unidades › Crear", un botón "Ir a Jerarquías de Unidades" que abre "/jerarquias/nueva"
  Y los campos de unidad y precio no se dibujan

Escenario: La conversión no se puede resolver (solo API)
  Dado que llamo a la API con una unidad que no se puede convertir a la unidad base del producto
  Entonces la API responde 400 "PresentacionVenta.Jerarquia.NoResuelta"
  Y la pantalla lo previene porque solo ofrece unidades vendibles; si llegara, veo "Error al guardar la presentación de venta"

Escenario: Producto y unidad obligatorios
  Dado que no elegí producto
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" debajo del producto
  Y sin unidad veo "Debes seleccionar una unidad de venta"

Escenario: Precio negativo
  Dado que escribo -5 en "Precio de Venta"
  Cuando hago clic en "Guardar"
  Entonces veo "El precio de venta no puede ser negativo" (también si el campo está vacío) y no se envía nada
  Y el precio es propio de la presentación: no se calcula desde el precio base del medicamento

Escenario: Una sola presentación por unidad y producto
  Dado que el producto ya tiene una presentación "Caja"
  Cuando intento crear otra con la misma unidad
  Entonces la API responde 409 "PresentacionVenta.UnidadVenta.Duplicada"
  Y veo "Ya existe una presentación de venta para esa unidad en este producto."

Escenario: El producto no se puede cambiar
  Dado que abrí una presentación existente y hago clic en "Editar"
  Entonces el producto queda deshabilitado con la ayuda "El producto de una presentación no se puede modificar luego de creada"
  Y puedo cambiar la unidad de venta y el precio

Escenario: Editar una presentación
  Dado que tengo "Presentaciones de Venta › Actualizar"
  Cuando hago clic en "Editar", cambio el precio y hago clic en "Guardar"
  Entonces veo "Presentación de venta guardada correctamente" y vuelvo a "/presentaciones-venta"
  Y la API vuelve a calcular la equivalencia y a comprobar que no se duplique la unidad

Escenario: Producto o unidad que ya no existen
  Dado que el producto o la unidad fueron eliminados
  Cuando hago clic en "Guardar"
  Entonces la API responde 404 "PresentacionVenta.Producto" o "PresentacionVenta.UnidadVenta"
  Y veo "Error al guardar la presentación de venta"

Escenario: La presentación ya no existe
  Dado que fue eliminada
  Cuando intento guardarla o abrirla
  Entonces la API responde 404 "PresentacionVenta.NotFound"
  Y al guardar veo "Error al guardar la presentación de venta" y al abrir "Error al cargar la presentación de venta"

Escenario: La equivalencia guardada no se recalcula
  Dado que luego se modifican las jerarquías del producto
  Entonces la presentación conserva la equivalencia que guardó al crearse o editarse

Escenario: Identificador al editar (solo API)
  Dado que llamo a "PUT /api/v1/PresentacionesVenta/{id}"
  Entonces el identificador se toma de la URL y uno vacío se rechaza con "UpdatePresentacionVentaCommand.Id"
```

> **Brecha conocida.** (1) La pantalla muestra «Ya existe una presentación de venta para esa unidad en este producto.» ante **cualquier** 409, no solo ante la unidad duplicada ([19d §2.2](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md)). (2) No usa el mapa común: los códigos `Db.*` caen en «Error al guardar la presentación de venta». (3) Si eliges un producto sin jerarquía y pulsas «Guardar», no se envía nada y no hay error visible (por lectura del código; no se ejecutó). (4) La API no impide crear una presentación cuya unidad de venta sea la propia unidad base (el cálculo devuelve 1); la pantalla no ofrece ese caso ([01b nota PRE](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md#9-presentaciones-de-venta-pre)). (5) **No verificado en la aplicación:** el manual no recorrió esta pantalla.

### HU-CAT-025 — Administrar las presentaciones desde la ficha de un medicamento

**Como** administrador **quiero** agregar, editar y eliminar las presentaciones de un medicamento sin salir de su ficha **para** completar un producto de una sola vez.

| | |
|---|---|
| **Módulo** | Presentaciones de venta (diálogo) |
| **Pantalla** | Diálogo «Presentaciones de Venta» que abre el botón del mismo nombre en `/medicamentos/:id` (modo ver) |
| **Endpoint(s)** | `GET /api/v1/JerarquiasUoM/unidades-vendibles`, `GET /api/v1/PresentacionesVenta?productoId=…&pageNumber=1&pageSize=100`, `POST /api/v1/PresentacionesVenta`, `PUT /api/v1/PresentacionesVenta/{id}`, `DELETE /api/v1/PresentacionesVenta/{id}` |
| **Permiso** | Botón: `JerarquiasUoM:Leer` y `PresentacionesVenta:Leer`; «Agregar presentación»: `PresentacionesVenta:Crear`; lápiz: `PresentacionesVenta:Actualizar`; papelera: `PresentacionesVenta:Eliminar` |
| **Reglas** | [RN-PRE-001, RN-PRE-005, RN-PRE-006, RN-PRE-009, RN-PRE-012, RN-PRE-015](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Abrir el diálogo
  Dado que estoy en el detalle de un medicamento en modo ver y tengo los permisos de lectura
  Cuando hago clic en "Presentaciones de Venta"
  Entonces se abre el diálogo "Presentaciones de Venta" con el nombre comercial debajo del título y el botón "Cerrar"

Escenario: Producto sin presentaciones
  Dado que el producto tiene unidades vendibles pero ninguna presentación
  Entonces veo "Este producto todavía no tiene presentaciones de venta."

Escenario: Producto sin jerarquía
  Dado que el producto no tiene unidades vendibles
  Entonces veo "Este producto no tiene una Jerarquía de Unidades registrada. Cárgala primero para poder definir presentaciones de venta."
  Y con "Jerarquías de Unidades › Crear" veo "Ir a Jerarquías de Unidades", que cierra el diálogo y abre "/jerarquias/nueva"

Escenario: Agregar una presentación
  Dado que tengo "Presentaciones de Venta › Crear"
  Cuando hago clic en "Agregar presentación", elijo "Caja (100 Unidad)", escribo el precio 40.00 y hago clic en "Guardar"
  Entonces veo "Presentación de venta guardada correctamente", la lista se recarga y el formulario en línea se cierra
  Y la tabla muestra "Unidad de Venta", "Equivalencia" (por ejemplo "100 Unidad"), "Precio de Venta" ("S/ 40.00") y los botones de editar y eliminar según mis permisos

Escenario: Solo las unidades sin presentación
  Dado que el producto ya tiene presentación para "Caja"
  Cuando abro "Agregar presentación"
  Entonces "Unidad de Venta" no ofrece "Caja" (al editar sigue disponible la propia)
  Y si ya existe una presentación para cada unidad, "Agregar presentación" queda deshabilitado con el aviso "Ya existe una presentación para cada unidad definida en la Jerarquía de este producto."

Escenario: Campos obligatorios y precio negativo
  Dado que dejo la unidad vacía o escribo un precio negativo
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" o "El precio de venta no puede ser negativo"

Escenario: Unidad duplicada
  Dado que la API responde 409
  Entonces veo "Ya existe una presentación de venta para esa unidad."
  Y cualquier otro error muestra "Error al guardar la presentación de venta"

Escenario: Eliminar una presentación sin ventas
  Dado que tengo "Presentaciones de Venta › Eliminar"
  Cuando hago clic en la papelera y confirmo "¿Estás seguro de que deseas eliminar esta presentación de venta?"
  Entonces veo "Presentación de venta eliminada correctamente"

Escenario: Eliminar una presentación con ventas
  Dado que la presentación figura en un detalle de venta no eliminado
  Cuando confirmo
  Entonces la API responde 409 "PresentacionVenta.InUse"
  Y veo "No se puede eliminar la presentación porque tiene ventas registradas."
  Y otro error muestra "Error al eliminar la presentación de venta"

Escenario: Falla la carga
  Dado que falla la consulta de unidades vendibles o de presentaciones
  Entonces veo "Error al cargar las presentaciones de venta"
```

> **Nota.** El diálogo carga solo la primera página de 100 presentaciones. No comprueba permisos: lo hace el botón que lo abre. **No verificado en la aplicación:** el manual no recorrió el diálogo ([19d §2.4](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md)).

### HU-CAT-026 — Buscar y exportar presentaciones de venta

**Como** administrador **quiero** buscar presentaciones por producto y por unidad de venta **para** revisar precios y equivalencias.

| | |
|---|---|
| **Módulo** | Presentaciones de venta |
| **Pantalla** | `/presentaciones-venta` — «Presentaciones de Venta» (tarjeta «Búsqueda de Presentaciones de Venta») |
| **Endpoint(s)** | `GET /api/v1/PresentacionesVenta`, `POST /api/v1/PresentacionesVenta/exportar` |
| **Permiso** | `PresentacionesVenta:Leer` (y `Medicamentos:Leer` para las sugerencias de producto) |
| **Reglas** | [RN-PRE-001, RN-PRE-014](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (con un defecto en la exportación) |

**Criterios de aceptación**

```gherkin
Escenario: Buscar presentaciones
  Dado que abro "/presentaciones-venta"
  Entonces veo los filtros "Producto" ("Buscar producto...") y "Unidad de Venta"
  Cuando hago clic en "Buscar"
  Entonces la tabla muestra "Producto" (enlace al detalle), "Unidad de Venta", "Equivalencia" (número, sin el nombre de la unidad base) y "Precio de Venta" ("S/ " con dos decimales, a la derecha)
  Y se ordena por nombre del producto y luego por descripción, sin las presentaciones cuyo producto fue eliminado

Escenario: Producto elegido de la lista o escrito
  Dado que elijo un producto de la lista de sugerencias
  Entonces la búsqueda filtra por ese producto exacto
  Pero si solo escribo el texto, filtra por nombre "contiene"

Escenario: Sin coincidencias
  Cuando ninguna presentación cumple
  Entonces la tabla muestra "Sin resultados"

Escenario: Exportar
  Dado que busqué
  Cuando hago clic en "Exportar"
  Entonces se descarga "presentaciones-venta.xlsx" con "Producto", "Unidad de Venta", "Equivalencia" y "Precio de Venta"
  Y si falla veo "Error al buscar presentaciones de venta"
```

> **Brecha conocida.** (1) La exportación ignora el producto elegido en la lista: el Excel sale sin filtrar por producto (solo por unidad de venta) ([19d §2.1](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md)). (2) Si borras el texto del filtro, el producto elegido antes no se descarta y «Buscar» sigue filtrando por él. Ambos salen de leer el código; no se ejecutaron.

### HU-CAT-027 — Eliminar una presentación de venta

**Como** administrador **quiero** eliminar una presentación que ya no ofrezco, salvo que ya se haya vendido **para** conservar el historial de ventas.

| | |
|---|---|
| **Módulo** | Presentaciones de venta |
| **Pantalla** | `/presentaciones-venta/:id` (botón «Eliminar» en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/PresentacionesVenta/{id}` |
| **Permiso** | `PresentacionesVenta:Eliminar` |
| **Reglas** | [RN-PRE-001, RN-PRE-011, RN-PRE-012, RN-PRE-013](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar una presentación sin ventas
  Dado que estoy editando una presentación y tengo "Presentaciones de Venta › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces veo "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar esta presentación de venta?"
  Y al hacer clic en "Confirmar" veo "Presentación de venta eliminada correctamente" y vuelvo a "/presentaciones-venta"
  Y la API responde 204

Escenario: Presentación con ventas registradas
  Dado que figura en algún detalle de venta no eliminado
  Cuando confirmo
  Entonces la API responde 409 "PresentacionVenta.InUse"
  Y veo "No se puede eliminar la presentación porque tiene ventas registradas."

Escenario: Otro fallo
  Dado que la presentación ya no existe (404 "PresentacionVenta.NotFound") o el servidor falla
  Entonces veo "Error al eliminar la presentación de venta"
```

---

## 9. Formulaciones clínicas

Una formulación clínica indica qué ingrediente activo contiene un medicamento, en qué concentración y en qué unidad, con un código TE Orange opcional. Un medicamento con varios ingredientes tiene varias formulaciones. No hay carga masiva.

### HU-CAT-028 — Registrar y editar una formulación clínica

**Como** administrador **quiero** registrar qué ingrediente activo y en qué concentración contiene cada medicamento **para** que el escáner de recetas pueda proponer equivalentes terapéuticos.

| | |
|---|---|
| **Módulo** | Formulaciones clínicas |
| **Pantalla** | `/formulaciones-clinicas/nueva` y `/formulaciones-clinicas/:id` — «Formulaciones Clínicas» |
| **Endpoint(s)** | `POST /api/v1/FormulacionesClinicas`, `PUT /api/v1/FormulacionesClinicas/{id}`, `GET /api/v1/FormulacionesClinicas/{id}` |
| **Permiso** | `FormulacionesClinicas:Crear`, `FormulacionesClinicas:Actualizar`, `FormulacionesClinicas:Leer`; además `Medicamentos:Leer`, `IngredientesActivos:Leer` y `UnidadesMedida:Leer` para elegir valores |
| **Reglas** | [RN-FOR-001, RN-FOR-002, RN-FOR-003, RN-FOR-004, RN-FOR-005, RN-FOR-006, RN-FOR-007, RN-FOR-008, RN-FOR-009, RN-FOR-010](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Registrar una formulación
  Dado que tengo "Formulaciones Clínicas › Crear" y hago clic en "Nuevo"
  Cuando elijo el medicamento "Paracetamol Demo 500 mg" ("Buscar medicamento...")
  Y elijo el ingrediente "Paracetamol" ("Buscar ingrediente activo...")
  Y escribo Concentración de Dosis 500 y elijo la unidad "Miligramo" ("Buscar unidad de medida...")
  Y hago clic en "Guardar"
  Entonces veo "Formulación clínica guardada correctamente" y vuelvo a "/formulaciones-clinicas"

Escenario: Hay que elegir de las listas
  Dado que dejé medicamento, ingrediente o unidad sin elegir
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" debajo del campo que falta y no se envía nada
  Y los autocompletados de esta pantalla no buscan con el campo vacío

Escenario: Concentración inválida
  Dado que escribo 0, un valor menor que 0,0001 o dejo el campo vacío
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" y no se envía nada
  Y la concentración se guarda con 4 decimales

Escenario: Código TE Orange opcional
  Dado que escribo un código de más de 5 caracteres
  Entonces el campo se detiene en 5 caracteres sin mensaje
  Y dejarlo vacío es válido

Escenario: Un ingrediente una sola vez por medicamento
  Dado que "Paracetamol Demo 500 mg" ya tiene el ingrediente "Paracetamol"
  Cuando registro otra formulación con el mismo medicamento e ingrediente
  Entonces la API responde 409 "Formulacion.Ingrediente.Duplicado"
  Y veo "Ya existe una formulación con los datos ingresados"

Escenario: El medicamento no se puede cambiar
  Dado que abrí una formulación y hago clic en "Editar"
  Entonces el medicamento queda deshabilitado y puedo cambiar ingrediente, concentración, unidad y código

Escenario: Editar una formulación
  Dado que tengo "Formulaciones Clínicas › Actualizar"
  Cuando hago clic en "Editar", cambio la concentración y hago clic en "Guardar"
  Entonces veo "Formulación clínica guardada correctamente" y vuelvo a "/formulaciones-clinicas"
  Y la API solo comprueba la existencia del ingrediente o de la unidad si cambiaron

Escenario: Medicamento, ingrediente o unidad que ya no existen
  Dado que alguno fue eliminado
  Cuando hago clic en "Guardar"
  Entonces la API responde 404 "Medicamento.NotFound", "IngredienteActivo.NotFound" o "UnidadMedida.NotFound"
  Y veo "Error al guardar la formulación clínica"

Escenario: La formulación ya no existe
  Dado que fue eliminada
  Cuando intento guardarla o abrirla
  Entonces la API responde 404 "Formulacion.NotFound"
  Y al guardar veo "Error al guardar la formulación clínica" y al abrir "Error al cargar la formulación clínica"

Escenario: Identificador al editar (solo API)
  Dado que llamo a "PUT /api/v1/FormulacionesClinicas/{id}"
  Entonces el identificador se toma de la URL y uno vacío se rechaza con "UpdateFormulacionClinicaCommand.Id"
```

> **Brecha conocida.** (1) Un cero o negativo en la concentración muestra «Este campo es obligatorio», que no describe el problema. (2) La pantalla muestra «Ya existe una formulación con los datos ingresados» ante **cualquier** 409 y no usa el mapa común de errores. (3) Si el texto de un autocompletado cambia tras elegir, la elección se descarta y el campo marca «Este campo es obligatorio». (4) **No verificado en la aplicación:** el manual no recorrió esta pantalla; la elección con teclado en los autocompletados tampoco ([19d §7](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md), No verificado 2).

### HU-CAT-029 — Buscar y exportar formulaciones clínicas

**Como** administrador **quiero** buscar formulaciones por medicamento o por ingrediente **para** revisar qué contiene cada producto.

| | |
|---|---|
| **Módulo** | Formulaciones clínicas |
| **Pantalla** | `/formulaciones-clinicas` — «Formulaciones Clínicas» (tarjeta «Búsqueda de Formulaciones Clínicas») |
| **Endpoint(s)** | `GET /api/v1/FormulacionesClinicas`, `POST /api/v1/FormulacionesClinicas/exportar` |
| **Permiso** | `FormulacionesClinicas:Leer` |
| **Reglas** | [RN-FOR-001, RN-FOR-012](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Buscar formulaciones
  Dado que abro "/formulaciones-clinicas"
  Entonces veo los filtros "Medicamento" e "Ingrediente Activo", ambos con lupa
  Cuando hago clic en "Buscar"
  Entonces la tabla muestra "Medicamento" (enlace al detalle), "Ingrediente Activo", "Concentración", "Unidad de Medida" y "Código TE Orange", ordenada por nombre del medicamento
  Y ambos filtros son "contiene", sin distinguir mayúsculas

Escenario: Sin coincidencias
  Cuando ninguna formulación cumple
  Entonces veo "Resultados (0)" y la fila vacía dice "Resultados"

Escenario: Exportar
  Cuando hago clic en "Exportar"
  Entonces se descarga "formulaciones-clinicas.xlsx" con "Medicamento", "Ingrediente Activo", "Concentración", "Unidad de Medida" y "Código TE Orange"
  Y si falla veo "Error al buscar formulaciones clínicas", con "Referencia: …" si la respuesta trae referencia de soporte
```

### HU-CAT-030 — Eliminar una formulación clínica

**Como** administrador **quiero** eliminar una formulación errónea **para** corregir el catálogo.

| | |
|---|---|
| **Módulo** | Formulaciones clínicas |
| **Pantalla** | `/formulaciones-clinicas/:id` (botón «Eliminar» en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/FormulacionesClinicas/{id}` |
| **Permiso** | `FormulacionesClinicas:Eliminar` |
| **Reglas** | [RN-FOR-001, RN-FOR-010, RN-FOR-011](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (sin validación de dependencias) |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar una formulación
  Dado que estoy editando una formulación y tengo "Formulaciones Clínicas › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces veo "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar esta formulación clínica?"
  Y al hacer clic en "Confirmar" veo "Formulación clínica eliminada correctamente" y vuelvo a "/formulaciones-clinicas"
  Y la API responde 204

Escenario: No hay bloqueo por dependencias
  Dado que la API no valida dependencias al eliminar una formulación
  Entonces no existe un 409 de negocio para este caso

Escenario: Cualquier error al eliminar
  Dado que la formulación ya no existe (404 "Formulacion.NotFound") o el servidor falla
  Entonces veo siempre "Error al eliminar la formulación clínica", porque la pantalla no distingue el motivo
```

### HU-CAT-031 — Consultar las formulaciones de un medicamento (solo API)

**Como** integrador **quiero** obtener todas las formulaciones no eliminadas de un medicamento **para** mostrar sus ingredientes en otra aplicación.

| | |
|---|---|
| **Módulo** | Formulaciones clínicas |
| **Pantalla** | Ninguna: ninguna pantalla llama a este endpoint |
| **Endpoint(s)** | `GET /api/v1/FormulacionesClinicas/medicamento/{productoId}` |
| **Permiso** | `FormulacionesClinicas:Leer` |
| **Reglas** | [RN-FOR-013](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Formulaciones de un medicamento
  Dado que el medicamento tiene dos formulaciones no eliminadas
  Cuando llamo al endpoint con su identificador
  Entonces recibo las dos formulaciones

Escenario: Medicamento inexistente
  Dado que el identificador no corresponde a ningún medicamento
  Cuando llamo al endpoint
  Entonces recibo una lista vacía (200), porque la consulta no valida la existencia del producto
```

---

## 10. Proveedores y precios de compra

El proveedor es a quien la botica le compra: razón social, identificador tributario («RUC / Tax ID»), términos financieros, calificación ESG y tasa de cumplimiento. Cada proveedor guarda además los precios de compra que cobra por producto, con su historial.

### HU-CAT-032 — Registrar y editar un proveedor

**Como** comprador **quiero** registrar a mis proveedores y mantener sus datos **para** asociarles los precios de compra de mis productos.

| | |
|---|---|
| **Módulo** | Proveedores |
| **Pantalla** | `/proveedores/nueva` y `/proveedores/:id` — «Proveedores» |
| **Endpoint(s)** | `POST /api/v1/Proveedores`, `PUT /api/v1/Proveedores/{id}`, `GET /api/v1/Proveedores/{id}` |
| **Permiso** | `Proveedores:Crear`, `Proveedores:Actualizar`, `Proveedores:Leer` |
| **Reglas** | [RN-PRV-001, RN-PRV-002, RN-PRV-003, RN-PRV-004, RN-PRV-005, RN-PRV-006, RN-PRV-007, RN-PRV-008, RN-PRV-009, RN-PRV-011](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (el formulario y la API difieren en algunos límites) |

**Criterios de aceptación**

```gherkin
Escenario: Registrar un proveedor
  Dado que tengo "Proveedores › Crear" y hago clic en "Nuevo"
  Entonces el campo "Tasa de Cumplimiento" trae el valor inicial 100
  Cuando escribo Razón Social "Distribuidora Demo S.A.C." y "RUC / Tax ID" "20123456789"
  Y hago clic en "Guardar"
  Entonces veo "Proveedor guardado correctamente" y vuelvo a "/proveedores"
  Y la API responde 200 con el identificador (no 201)

Escenario: Campos opcionales
  Dado que dejo vacíos "Calificación ESG" y "Términos Financieros"
  Cuando hago clic en "Guardar"
  Entonces se guarda; ambos son opcionales

Escenario: Campos obligatorios vacíos
  Dado que dejo vacíos "Razón Social", "RUC / Tax ID" o "Tasa de Cumplimiento"
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" debajo de cada uno y no se envía nada
  Y "Razón Social" no deja pasar de 200 caracteres y "RUC / Tax ID" de 30

Escenario: Tasa de cumplimiento y calificación ESG en rango
  Dado que escribo una tasa fuera de 0 a 100
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" (el mismo texto para cualquier error del campo) y no se envía nada
  Y una calificación ESG fuera de 0 a 100 bloquea "Guardar" sin mostrar ningún mensaje
  Y la calificación y la tasa se guardan con 2 decimales

Escenario: Identificador tributario repetido
  Dado que existe un proveedor con "RUC / Tax ID" "20123456789"
  Cuando intento registrar otro con el mismo identificador
  Entonces la API responde 409 "Proveedor.TaxId.Duplicado"
  Y veo "Ya existe un proveedor con ese identificador tributario."
  Y esta comprobación se evalúa antes que la de razón social

Escenario: Razón social repetida
  Dado que existe "Distribuidora Demo S.A.C."
  Cuando intento registrar otro proveedor con la misma razón social y distinto identificador
  Entonces la API responde 409 "Proveedor.RazonSocial.Duplicado"
  Y veo "Ya existe un proveedor con esa razón social."

Escenario: El identificador tributario no se valida como RUC
  Dado que escribo un identificador de 8 letras
  Cuando hago clic en "Guardar"
  Entonces se guarda: la API solo exige que no esté vacío y que tenga como máximo 50 caracteres

Escenario: Editar un proveedor
  Dado que abrí un proveedor y tengo "Proveedores › Actualizar"
  Cuando hago clic en "Editar", cambio la tasa de cumplimiento y hago clic en "Guardar"
  Entonces veo "Proveedor guardado correctamente", me quedo en el detalle y la tarjeta de precios se oculta mientras edito
  Y al excluir al propio proveedor no choca consigo mismo

Escenario: El proveedor ya no existe
  Dado que fue eliminado
  Cuando intento guardarlo o abrirlo
  Entonces la API responde 404 "Proveedor.NotFound"
  Y al guardar veo "Error al guardar el proveedor" y al abrir "Error al cargar el proveedor"
```

> **Brecha conocida.** (1) El formulario limita el «RUC / Tax ID» a 30 caracteres (la API admite 50) y permite 500 en los términos financieros (la API admite 100): un texto de 101 a 500 caracteres pasa el formulario y la API lo rechaza con `ProveedorDistribuidor.TerminosFinancieros`, sin traducción, y el usuario ve «Error al guardar el proveedor». (2) Una razón social o RUC de solo espacios muestra «Error al guardar el proveedor». (3) El rótulo dice «RUC» pero no se valida el formato de RUC. (4) `CreateProveedorCommandValidator` está vacío y no hay validador para editar: las reglas las aplica la entidad ([01b §11](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md#11-proveedores-prv)). (5) **No verificado en la aplicación:** el manual no recorrió esta pantalla. Eliminar uno inexistente devuelve el código genérico `NotFound` (ver HU-CAT-034).

### HU-CAT-033 — Buscar y exportar proveedores

**Como** comprador **quiero** buscar proveedores por razón social, identificador y tasa de cumplimiento **para** elegir a quién comprar.

| | |
|---|---|
| **Módulo** | Proveedores |
| **Pantalla** | `/proveedores` — «Proveedores» (tarjeta «Búsqueda de Proveedores») |
| **Endpoint(s)** | `GET /api/v1/Proveedores`, `POST /api/v1/Proveedores/exportar` |
| **Permiso** | `Proveedores:Leer` |
| **Reglas** | [RN-PRV-001, RN-PRV-012](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Filtros
  Dado que abro "/proveedores"
  Entonces veo "Razón Social" y "RUC / Tax ID" (texto con lupa, "contiene") y un deslizador "Tasa de Cumplimiento: 0 — 100" con paso 1

Escenario: Buscar con rango de tasa
  Dado que muevo el deslizador a 80 — 100
  Cuando hago clic en "Buscar"
  Entonces veo los proveedores con tasa entre 80 y 100, ambos inclusive
  Y solo se envía el límite que moví

Escenario: Tabla
  Dado que hay resultados
  Entonces veo "Razón Social" (enlace al detalle), "RUC / Tax ID" y "Tasa de Cumplimiento", ordenados por razón social y luego por identificador
  Y la tabla no muestra términos financieros ni calificación ESG

Escenario: Sin coincidencias
  Cuando ningún proveedor cumple
  Entonces veo "Resultados (0)" y la fila vacía dice "Resultados"

Escenario: Limpiar y volver
  Dado que hice clic en "Limpiar"
  Entonces el deslizador vuelve a 0 — 100
  Pero al volver desde un detalle el deslizador no se restaura (los textos sí)

Escenario: Exportar
  Cuando hago clic en "Exportar"
  Entonces se descarga el Excel de proveedores aplicando los mismos filtros
  Y si falla veo "Error al buscar proveedores", con "Referencia: …" si la respuesta trae referencia de soporte
```

### HU-CAT-034 — Eliminar un proveedor

**Como** comprador **quiero** eliminar un proveedor que ya no uso **para** mantener limpia la lista, sin perder el historial de costos.

| | |
|---|---|
| **Módulo** | Proveedores |
| **Pantalla** | `/proveedores/:id` (botón «Eliminar» en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/Proveedores/{id}` |
| **Permiso** | `Proveedores:Eliminar` |
| **Reglas** | [RN-PRV-001, RN-PRV-009, RN-PRV-010](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (un proveedor con precios registrados no se puede eliminar nunca) |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar un proveedor sin precios
  Dado que estoy editando un proveedor sin precios de compra y tengo "Proveedores › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces veo "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar este proveedor?"
  Y al hacer clic en "Confirmar" veo "Proveedor eliminado correctamente" y vuelvo a "/proveedores"
  Y la API responde 200 con el identificador eliminado

Escenario: Proveedor con precios de compra
  Dado que el proveedor tiene algún registro no eliminado en el historial de precios
  Cuando confirmo la eliminación
  Entonces la API responde 409 "Proveedor.InUse" (bloqueo duro)
  Y veo "No puedes eliminar este proveedor porque tiene precios de compra registrados."

Escenario: Otro fallo
  Dado que el proveedor no existe
  Cuando confirmo
  Entonces la API responde 404 con el código genérico "NotFound"
  Y veo "Error al eliminar el proveedor"
```

> **Brecha conocida.** Como ninguna pantalla permite borrar un precio de compra, **un proveedor al que se registró un precio nunca más se podrá eliminar** ([19d §6](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md)). La pantalla muestra el texto de «precios de compra» ante cualquier 409.

### HU-CAT-035 — Cargar proveedores desde un Excel

**Como** comprador **quiero** cargar muchos proveedores de una vez **para** no registrarlos uno por uno.

| | |
|---|---|
| **Módulo** | Proveedores (carga masiva) |
| **Pantalla** | `/proveedores/carga-masiva` — «Carga Masiva» |
| **Endpoint(s)** | `GET /api/v1/Proveedores/plantilla`, `POST /api/v1/Proveedores/previsualizar`, `POST /api/v1/Proveedores/carga-masiva` |
| **Permiso** | `Proveedores:Crear` (además de `Leer`) |
| **Reglas** | [RN-PRV-002, RN-PRV-003, RN-PRV-004, RN-PRV-005, RN-PRV-006, RN-PRV-007, RN-PRV-008, RN-PRV-013, RN-PRV-014, RN-PRV-015, RN-PRV-016, RN-PRV-017](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (la vista previa no revisa la razón social repetida) |

**Criterios de aceptación**

Cumple los escenarios CM-1 a CM-3 y CM-5 de la [sección 1.3](#13-escenarios-comunes-de-la-carga-masiva). Además:

```gherkin
Escenario: Columnas de la plantilla
  Dado que descargué "plantilla-proveedores.xlsx"
  Entonces tiene "RazonSocial" (obligatoria, 200 caracteres), "TaxId" (obligatoria, 50), "TasaCumplimiento" (obligatoria), "TerminosFinancieros" (opcional, 100) y "CalificacionEsg" (opcional)
  Y los números se leen con punto como separador decimal

Escenario: Reglas por fila en la vista previa
  Entonces un campo obligatorio vacío muestra "El campo 'razonSocial' es obligatorio" (y lo mismo para 'taxId' y 'tasaCumplimiento')
  Y un texto demasiado largo muestra "'razonSocial' no puede superar los 200 caracteres"
  Y un número inválido muestra "'tasaCumplimiento' debe ser un número decimal válido"
  Y un valor fuera de 0 a 100 muestra "'calificacionEsg' debe estar entre 0 y 100" o "'tasaCumplimiento' debe estar entre 0 y 100"

Escenario: Identificador tributario existente o repetido en el archivo
  Entonces la vista previa muestra "El valor '<valor>' del campo 'taxId' ya existe en el sistema" o "…está duplicado en el archivo" (en todas las filas repetidas)

Escenario: Razón social repetida pasa la vista previa y se omite al guardar
  Dado que una fila repite una razón social ya existente
  Entonces la vista previa la muestra válida
  Pero al guardar el servidor la omite sin error (y de las repetidas del archivo guarda solo la primera)
  Y veo "Se guardaron N registros correctamente" con N menor que las filas válidas
```

> **Brecha conocida.** Las etiquetas de la vista previa usan la raíz de textos: la columna se rotula «Tax ID» y no «RUC / Tax ID». **No verificado:** que el guardado acepte los valores numéricos tal como los devuelve la vista previa (texto) al convertirlos a `decimal` ([01b §16.1](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), No verificado 7). El manual no recorrió esta carga.

### HU-CAT-036 — Registrar el precio de compra de un producto con un proveedor

**Como** comprador **quiero** registrar el precio que me cobra un proveedor por un producto, con su descuento **para** conocer el costo neto por unidad base.

| | |
|---|---|
| **Módulo** | Proveedores (precios de compra) |
| **Pantalla** | Diálogo «Registrar precio de compra» que abre el botón «Registrar precio» de la tarjeta «Precios de compra» en `/proveedores/:id` |
| **Endpoint(s)** | `POST /api/v1/Proveedores/precios`; `GET /api/v1/Medicamentos` (búsqueda del producto) |
| **Permiso** | `Proveedores:RegistrarPrecioProveedor` (rotulado «Registrar precio de proveedor»); para buscar el producto, `Medicamentos:Leer` |
| **Reglas** | [RN-PRV-001, RN-PRV-018, RN-PRV-019, RN-PRV-020, RN-PRV-021, RN-PRV-022, RN-PRV-023, RN-PRV-024, RN-PRV-027, RN-PRV-028](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Parcial (el precio no se puede editar ni eliminar; el plazo de entrega y la cantidad mínima no se pueden informar) |

**Criterios de aceptación**

```gherkin
Escenario: Registrar un precio de compra
  Dado que tengo "Proveedores › Registrar precio de proveedor" y estoy en el detalle de un proveedor en modo ver
  Cuando hago clic en "Registrar precio"
  Entonces se abre "Registrar precio de compra" con el texto "El nuevo precio reemplaza al vigente de este producto con este proveedor; el anterior queda en el historial."
  Cuando escribo "Paracetamol" en "Producto" y elijo "Paracetamol Demo 500 mg" de la lista
  Y escribo 12.50 en "Precio de compra por unidad base (S/)" y 10 en "Descuento (%)"
  Entonces "Costo neto por unidad base" muestra "S/ 11.25"
  Cuando hago clic en "Guardar"
  Entonces veo "Precio registrado correctamente", el diálogo se cierra y la tarjeta se recarga desde la página 1

Escenario: El costo neto se guarda, no el bruto
  Dado que registré precio 12.50 con descuento 10
  Entonces el servidor guarda el costo neto = precio × (1 − descuento / 100), redondeado a 4 decimales (hacia afuera en el punto medio): 11.2500
  Y no se guardan el precio bruto ni el descuento
  Y la vigencia empieza en el instante del registro (hora UTC del servidor), sin fecha de fin

Escenario: El precio nuevo reemplaza al vigente
  Dado que el producto ya tenía un precio vigente con este proveedor
  Cuando registro uno nuevo
  Entonces el precio anterior queda cerrado con ese mismo instante como fecha de fin
  Y el producto tiene un solo precio vigente por proveedor, con todo guardado en una sola operación

Escenario: Descuento 100
  Dado que escribo 100 en "Descuento (%)"
  Entonces el costo neto es 0 y se acepta

Escenario: Producto de la lista
  Dado que escribí un texto en "Producto" sin elegir una opción
  Cuando hago clic en "Guardar"
  Entonces veo "Selecciona un producto de la lista" y no se envía nada
  Y si no hay coincidencias la lista dice "Sin resultados" y el campo muestra "Escribe para buscar un producto"

Escenario: Precio y descuento fuera de rango
  Dado que dejo el precio o el descuento vacíos
  Entonces veo "Este campo es obligatorio"
  Y un precio de 0 o negativo muestra "Debe ser mayor que 0"
  Y un precio mayor que 9 999 999,9999 muestra "El precio es demasiado alto"
  Y un descuento fuera de 0 a 100 muestra "Debe estar entre 0 y 100"
  Y mientras sean inválidos, "Costo neto por unidad base" muestra "—"

Escenario: Proveedor o producto que ya no existen
  Dado que el proveedor o el producto fueron eliminados
  Cuando hago clic en "Guardar"
  Entonces la API responde 404 "Proveedor.NotFound" (se comprueba primero) o "Medicamento.NotFound"
  Y veo "El proveedor ya no existe." o "El producto seleccionado ya no existe."
  Y el diálogo permanece abierto

Escenario: Otro error al registrar
  Dado que el servidor rechaza el registro por otra causa
  Entonces veo "Error al registrar el precio" y el botón "Guardar" se habilita de nuevo

Escenario: Sin el permiso no hay botón
  Dado que mi rol no tiene "Registrar precio de proveedor"
  Entonces no veo el botón "Registrar precio"

Escenario: Sin permiso de lectura de medicamentos
  Dado que mi rol no tiene "Medicamentos › Leer"
  Cuando escribo en "Producto"
  Entonces la búsqueda falla en silencio y no aparecen opciones

Escenario: Valores que la API no deja informar
  Dado que registro un precio
  Entonces el plazo de entrega queda en 0 y la cantidad mínima de compra en 1
  Y no se puede elegir la fecha de vigencia

Escenario: Reglas internas del historial (no alcanzables desde la pantalla)
  Dado que el registro valida con la entidad del historial de precios
  Entonces exige proveedor y producto, costo no negativo, inicio de vigencia, fin no anterior al inicio, plazo de entrega no negativo y cantidad mínima de al menos 1
  Y el validador del registro responde antes con las mismas condiciones, de modo que esos rechazos no llegan al usuario
```

> **Pendiente.** Ningún comando invoca `HistorialPrecioProveedor.Update` ni existe borrado de precios: un precio registrado no se puede corregir ni eliminar. La regla «un solo precio abierto por proveedor y producto» la impone el handler; no hay índice único, y dos registros simultáneos podrían dejar dos precios abiertos (**No verificado**, no se probó concurrencia; [01b §16.1](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), No verificado 9). El módulo de ventas lee este historial para el costo de cada venta (parte C). **No verificado en la aplicación:** el manual no recorrió el diálogo; los mensajes salen de 19d §4.5.

### HU-CAT-037 — Consultar el historial de precios de compra de un proveedor

**Como** comprador **quiero** ver los precios de compra de un proveedor, vigentes y anteriores **para** comparar costos.

| | |
|---|---|
| **Módulo** | Proveedores (precios de compra) |
| **Pantalla** | `/proveedores/:id` — tarjeta «Precios de compra» (solo en modo ver) |
| **Endpoint(s)** | `GET /api/v1/Proveedores/{id}/precios?pageNumber=&pageSize=` |
| **Permiso** | `Proveedores:Leer` |
| **Reglas** | [RN-PRV-025, RN-PRV-026](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Ver los precios
  Dado que abrí un proveedor existente en modo ver
  Entonces veo la tarjeta "Precios de compra" con el subtítulo "Costo por unidad base que te cobra este proveedor, con su descuento ya aplicado. Cada producto tiene un solo precio vigente y los anteriores quedan en el historial."
  Y una tabla con "Producto", "Costo por unidad base" ("S/ " con 4 decimales, por ejemplo "S/ 11.2500"), "Vigente desde" y "Vigente hasta" (fechas dd/MM/yyyy)
  Y si el precio no tiene fecha de fin, "Vigente hasta" dice "Vigente"

Escenario: Orden y precios cerrados
  Dado que reemplacé un precio por otro
  Entonces la tabla muestra el nuevo primero (inicio de vigencia descendente) y el anterior con su fecha de fin
  Y la API incluye todos los registros, vigentes y cerrados

Escenario: Paginación propia
  Dado que hay más de 10 precios
  Entonces el paginador ofrece 5, 10 y 25 filas (10 por defecto)
  Y al registrar un precio la tabla vuelve a la primera página

Escenario: Sin precios
  Dado que el proveedor no tiene precios
  Entonces veo "Este proveedor aún no tiene precios registrados."

Escenario: Producto que ya no existe
  Dado que el medicamento de un precio fue eliminado
  Entonces la fila aparece con el nombre del producto vacío

Escenario: La tarjeta no aparece al crear ni al editar
  Dado que estoy creando o editando el proveedor
  Entonces la tarjeta "Precios de compra" no se muestra

Escenario: Error al cargar
  Dado que falla la lectura o el proveedor no existe (404 "Proveedor.NotFound")
  Entonces veo "Error al cargar los precios del proveedor"
```

> **Nota.** La API devuelve el indicador `vigente` (inicio anterior o igual a ahora y fin nulo o posterior), pero la pantalla no lo usa: el texto «Vigente» depende solo de que la fecha de fin sea nula ([19d §4.3](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md)). La tabla no tiene acciones por fila. **No verificado:** el efecto de la zona horaria en las fechas mostradas.

---

## 11. Aseguradoras

Catálogo de aseguradoras médicas: nombre comercial y código identificador nacional. Hoy solo se registran: el bloque de seguro del punto de venta está oculto (manual cap. 5, §3). El módulo de permisos se llama `Seguros` y la pantalla «Aseguradoras».

### HU-CAT-038 — Registrar y editar una aseguradora

**Como** administrador **quiero** registrar las aseguradoras médicas con las que trabajo **para** tenerlas en el catálogo.

| | |
|---|---|
| **Módulo** | Aseguradoras (permisos `Seguros`) |
| **Pantalla** | `/aseguradoras/nueva` y `/aseguradoras/:id` — «Aseguradoras Médicas» |
| **Endpoint(s)** | `POST /api/v1/Seguros`, `PUT /api/v1/Seguros/{id}`, `GET /api/v1/Seguros/{id}` |
| **Permiso** | `Seguros:Crear`, `Seguros:Actualizar`, `Seguros:Leer` |
| **Reglas** | [RN-ASE-001, RN-ASE-002, RN-ASE-003, RN-ASE-004, RN-ASE-005, RN-ASE-007](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Parcial (la pantalla no traduce ningún error: todo fallo se ve como mensaje genérico, incluido el código repetido) |

**Criterios de aceptación**

```gherkin
Escenario: Registrar una aseguradora
  Dado que tengo "Seguros › Crear" (rotulado "Aseguradoras") y hago clic en "Nuevo"
  Cuando escribo Nombre Comercial "Aseguradora Demo" y Código Identificador Nacional "ASEG-0001"
  Y hago clic en "Guardar"
  Entonces veo "Aseguradora guardada correctamente" y vuelvo a "/aseguradoras"
  Y la API responde 200 con el identificador (no 201)

Escenario: Campos obligatorios y longitudes
  Dado que dejo vacío cualquiera de los dos campos
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" debajo del campo y no se envía nada
  Y "Nombre Comercial" no deja pasar de 150 caracteres y "Código Identificador Nacional" de 50

Escenario: Código identificador nacional repetido
  Dado que ya existe una aseguradora con el código "ASEG-0001"
  Cuando intento registrar otra con el mismo código
  Entonces los handlers no comprueban el duplicado y el índice único de la base lo rechaza (409 "Db.Duplicado")
  Y veo "Error al guardar la aseguradora", porque la pantalla no distingue el motivo

Escenario: El código de una aseguradora eliminada queda libre
  Dado que eliminé la aseguradora con código "ASEG-0001"
  Cuando registro otra con ese código
  Entonces se guarda sin conflicto

Escenario: Editar una aseguradora
  Dado que abrí una aseguradora y tengo "Seguros › Actualizar"
  Cuando hago clic en "Editar", cambio el nombre y hago clic en "Guardar"
  Entonces veo "Aseguradora guardada correctamente", me quedo en el detalle y los datos se recargan

Escenario: Cualquier otro error al guardar
  Dado que el servidor falla o la aseguradora no existe (404 "Aseguradora.NotFound")
  Entonces veo "Error al guardar la aseguradora"
  Y al abrir una inexistente veo "Error al cargar la aseguradora"
```

> **Brecha conocida.** (1) `CreateAseguradoraCommandValidator` está vacío y no hay validador para editar: las reglas las aplica la entidad. (2) Un código repetido se ve como mensaje genérico; existe la clave `errors.dbDuplicado` pero esta pantalla no la usa ([RN-ASE-004](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [19d §5.2](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md)). (3) Un valor de solo espacios también muestra el mensaje genérico (por lectura del código). (4) **No verificado en la aplicación:** el manual no recorrió esta pantalla.

### HU-CAT-039 — Buscar y exportar aseguradoras

**Como** administrador **quiero** buscar aseguradoras por nombre o código **para** ubicarlas.

| | |
|---|---|
| **Módulo** | Aseguradoras |
| **Pantalla** | `/aseguradoras` — «Aseguradoras Médicas» (tarjeta «Búsqueda de Aseguradoras») |
| **Endpoint(s)** | `GET /api/v1/Seguros`, `POST /api/v1/Seguros/exportar` |
| **Permiso** | `Seguros:Leer` |
| **Reglas** | [RN-ASE-001, RN-ASE-008](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (el orden de las filas no está garantizado) |

**Criterios de aceptación**

```gherkin
Escenario: Buscar
  Dado que abro "/aseguradoras"
  Entonces veo los filtros "Nombre Comercial" y "Código Identificador Nacional" ("contiene")
  Cuando hago clic en "Buscar"
  Entonces la tabla muestra "Nombre Comercial" (enlace al detalle) y "Código Identificador Nacional"

Escenario: Sin orden definido
  Dado que hay varias aseguradoras
  Entonces el servidor no aplica un orden explícito, así que el orden de las filas no está garantizado

Escenario: Sin coincidencias
  Cuando ninguna aseguradora cumple
  Entonces veo "Resultados (0)" y la fila vacía dice "Resultados"

Escenario: Exportar
  Cuando hago clic en "Exportar"
  Entonces se descarga "aseguradoras.xlsx" con "Nombre Comercial" y "Código Identificador Nacional", con los mismos filtros
  Y si falla veo "Error al buscar aseguradoras", con "Referencia: …" si la respuesta trae referencia de soporte
```

### HU-CAT-040 — Eliminar una aseguradora

**Como** administrador **quiero** eliminar una aseguradora que ya no uso **para** mantener el catálogo al día.

| | |
|---|---|
| **Módulo** | Aseguradoras |
| **Pantalla** | `/aseguradoras/:id` (botón «Eliminar» en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/Seguros/{id}` |
| **Permiso** | `Seguros:Eliminar` |
| **Reglas** | [RN-ASE-001, RN-ASE-005, RN-ASE-006](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (sin validación de dependencias) |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar una aseguradora
  Dado que estoy editando una aseguradora y tengo "Seguros › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces veo "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar esta aseguradora?"
  Y al hacer clic en "Confirmar" veo "Aseguradora eliminada correctamente" y vuelvo a "/aseguradoras"
  Y la API responde 200 con el identificador eliminado

Escenario: No se validan dependencias
  Dado que la aseguradora tiene reclamos de seguro asociados
  Cuando confirmo la eliminación
  Entonces se elimina igualmente, porque el handler no comprueba dependencias

Escenario: Cualquier error al eliminar
  Dado que la aseguradora no existe o el servidor falla
  Entonces la API responde 400 con el código genérico "NotFound" (no 404, porque el handler no indica el código HTTP) o un error de servidor
  Y veo siempre "Error al eliminar la aseguradora"
```

### HU-CAT-041 — Cargar aseguradoras desde un Excel

**Como** administrador **quiero** cargar varias aseguradoras de una vez **para** completar el catálogo.

| | |
|---|---|
| **Módulo** | Aseguradoras (carga masiva) |
| **Pantalla** | `/aseguradoras/carga-masiva` — «Carga Masiva» |
| **Endpoint(s)** | `GET /api/v1/Seguros/plantilla`, `POST /api/v1/Seguros/previsualizar`, `POST /api/v1/Seguros/carga-masiva` |
| **Permiso** | `Seguros:Crear` (además de `Leer`) |
| **Reglas** | [RN-ASE-002, RN-ASE-003, RN-ASE-004, RN-ASE-009, RN-ASE-010, RN-ASE-011, RN-ASE-012](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Parcial (el guardado no controla duplicados: una sola fila repetida hace fallar todo el envío) |

**Criterios de aceptación**

Cumple los escenarios CM-1 a CM-4 de la [sección 1.3](#13-escenarios-comunes-de-la-carga-masiva) y de CM-5 el guardado de las filas válidas. Además:

```gherkin
Escenario: Columnas de la plantilla
  Dado que descargué la plantilla
  Entonces tiene "NombreComercial" y "CodigoIdentificadorNacional", ambas obligatorias
  Y el SPA la guarda como "plantilla-aseguradoras.xlsx" aunque la API la nombra "plantilla-seguros.xlsx"

Escenario: Reglas por fila en la vista previa
  Entonces un campo vacío muestra "El campo 'nombreComercial' es obligatorio" o "El campo 'codigoIdentificadorNacional' es obligatorio"
  Y un código existente muestra "El valor '<valor>' del campo 'codigoIdentificadorNacional' ya existe en el sistema"
  Y un código repetido en el archivo muestra "…está duplicado en el archivo"

Escenario: Un nombre de más de 150 caracteres
  Dado que una fila tiene un nombre comercial de 160 caracteres
  Entonces la vista previa muestra "'nombreComercial' no puede superar los 200 caracteres" aunque el límite real es 150

Escenario: Un solo duplicado hace fallar todo el guardado
  Dado que, entre la vista previa y el guardado, una fila pasó a repetir un código existente o el de otra fila
  Cuando hago clic en "Guardar válidos (N)"
  Entonces el índice único rechaza el guardado (409 "Db.Duplicado") y no se guarda ninguna fila
  Y veo "Error al guardar los registros"

Escenario: No se valida el tamaño de la lista en el guardado (solo API)
  Dado que llamo a "POST /api/v1/Seguros/carga-masiva" con una lista vacía o de más de 5000 filas
  Entonces no existe validador del comando y el guardado procede con lo que pase los guards de la entidad

Escenario: La extensión del archivo (solo la pantalla la exige)
  Dado que llamo a "POST /api/v1/Seguros/previsualizar" con un archivo que no termina en ".xlsx"
  Entonces el servidor no comprueba la extensión (sí la firma y el contenido)
  Pero la pantalla exige ".xlsx" antes de enviar
```

> **Brecha conocida.** La carga de aseguradoras no tiene validador ni control de duplicados ([RN-ASE-011](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md)); desde la pantalla el fallo solo ocurre si el dato cambió entre la vista previa y el guardado, porque la vista previa ya filtra los duplicados. El mensaje de longitud de la vista previa dice 200 en lugar de 150 ([hallazgo 14 de 01b](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md#163-pendientes-y-hallazgos-del-código)). El manual no recorrió esta carga.

---

## 12. Catálogo DIGEMID

Tabla local de productos DIGEMID de la empresa (código, nombre, concentración, forma farmacéutica, fracción, registro sanitario, titular y estado), que se llena con una carga masiva y se usa para validar los nombres de medicamentos que la IA detecta en una receta. SOFIA **no** se conecta a DIGEMID. Las operaciones de cuarentena y actas del mismo controlador no se tratan aquí.

### HU-CAT-042 — Buscar, ver y exportar el catálogo DIGEMID

**Como** cajero o administrador **quiero** buscar un producto en el catálogo DIGEMID y ver sus datos **para** confirmar su registro sanitario y su titular.

| | |
|---|---|
| **Módulo** | Catálogo DIGEMID |
| **Pantalla** | `/digemid-catalogo` y `/digemid-catalogo/:id` — «Catálogo DIGEMID» (grupo de menú DIGEMID; tarjeta «Búsqueda en Catálogo DIGEMID») |
| **Endpoint(s)** | `GET /api/v1/DIGEMID/catalogo`, `GET /api/v1/DIGEMID/catalogo/{id}`, `POST /api/v1/DIGEMID/catalogo/exportar` |
| **Permiso** | `DIGEMID:Leer` |
| **Reglas** | [RN-DIG-001, RN-DIG-007, RN-DIG-009](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (solo lectura) |

**Criterios de aceptación**

```gherkin
Escenario: Una empresa nueva tiene el catálogo vacío
  Dado que registré mi empresa y aún no cargué productos
  Cuando hago clic en "Buscar" sin filtros
  Entonces veo "Resultados (0)"

Escenario: Buscar por código y por nombre
  Dado que cargué cinco productos, entre ellos "DEMO-0001" y los "TABLETA"
  Cuando escribo "demo-000" en "Código de Producto" y "capsula" en "Nombre del Producto" y busco
  Entonces veo "DEMO-0002" y "DEMO-0004"
  Y "Código de Producto" "0003" trae "DEMO-0003"; "Nombre del Producto" "tabl" trae los tres productos "TABLETA"
  Y la búsqueda de código no distingue mayúsculas

Escenario: El filtro por nombre distingue los acentos
  Dado que el catálogo tiene "AMOXICILINA 500 MG CAPSULA (DEMO)" sin tilde
  Cuando escribo "cápsula" (con tilde) en "Nombre del Producto"
  Entonces no encuentra el producto
  Y "capsula" sí lo encuentra

Escenario: Tabla de resultados
  Dado que hay resultados
  Entonces veo "Código" (enlace al detalle), "Nombre del Producto", "Forma Farmacéutica" ("—" si no tiene) y "Estado", ordenados por nombre de producto
  Y no veo concentración, registro sanitario ni titular

Escenario: Ver el detalle
  Dado que hago clic en el código "DEMO-0001"
  Entonces se abre una tarjeta sin botones de edición con ocho campos siempre deshabilitados: Código de Producto, Nombre del Producto, Concentración, Forma Farmacéutica, Fracción, Registro Sanitario, Titular y Estado
  Y los datos que el producto no tiene se ven como campo vacío
  Y con la flecha de volver regreso a la búsqueda con mis filtros

Escenario: Producto que no se puede abrir
  Dado que el producto fue eliminado o es de otra empresa
  Cuando abro su detalle
  Entonces la API responde 404 "DigemidCatalogo.NotFound"
  Y veo "Error al cargar el producto DIGEMID"

Escenario: Exportar
  Cuando hago clic en "Exportar"
  Entonces se descarga "digemid-catalogo.xlsx" con las columnas "Código", "Nombre del Producto", "Forma Farmacéutica" y "Estado"
  Y no incluye concentración, fracción, registro sanitario ni titular
  Y si falla la búsqueda o la exportación veo "Error al buscar en el catálogo DIGEMID"

Escenario: El cajero solo consulta
  Dado que mi rol (por ejemplo "Cajero") tiene solo "DIGEMID › Leer"
  Entonces veo el menú "Catálogo DIGEMID" con los filtros, la tabla y "Exportar"
  Pero no veo "Carga Masiva"
```

> **Brecha conocida.** Al abrir a mano `/digemid-catalogo/nueva` la pantalla muestra el formulario de detalle vacío y deshabilitado y un aviso de error de carga; al recargar la página directamente en esa dirección el aviso mostró la clave sin traducir `digemidCatalogo.detail.load-error` (manual cap. 4, §8.2, §11 punto 7). No se puede crear nada desde ahí. **No verificado en la aplicación:** el contenido del archivo exportado; el catálogo sembrado por script no aparece en una empresa recién registrada (comprobado: «Resultados (0)»), por lo que la nota de 01b sobre si los productos sembrados quedan asociados a una empresa se confirma solo para empresas nuevas.

### HU-CAT-043 — Cargar el catálogo DIGEMID desde un Excel

**Como** administrador **quiero** cargar los productos DIGEMID desde un Excel **para** que SOFIA pueda marcar como «validados por DIGEMID» los medicamentos detectados en una receta.

| | |
|---|---|
| **Módulo** | Catálogo DIGEMID (carga masiva) |
| **Pantalla** | `/digemid-catalogo/carga-masiva` — «Carga Masiva» (botón «Carga Masiva» de la búsqueda) |
| **Endpoint(s)** | `GET /api/v1/DIGEMID/catalogo/plantilla`, `POST /api/v1/DIGEMID/catalogo/previsualizar`, `POST /api/v1/DIGEMID/catalogo/carga-masiva` |
| **Permiso** | `DIGEMID:Crear` (además de `Leer`) |
| **Reglas** | [RN-DIG-001, RN-DIG-002, RN-DIG-003, RN-DIG-004, RN-DIG-005, RN-DIG-006, RN-DIG-010, RN-DIG-011, RN-DIG-012, RN-DIG-013](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (la vista previa no revisa longitudes; el guardado rechaza todo el archivo si una fila es demasiado larga) |

**Criterios de aceptación**

Cumple los escenarios CM-1 a CM-5 de la [sección 1.3](#13-escenarios-comunes-de-la-carga-masiva). Además:

```gherkin
Escenario: Columnas de la plantilla
  Dado que descargué "plantilla-digemid-catalogo.xlsx"
  Entonces tiene ocho columnas: "CodProd" (obligatoria, 20 caracteres), "NomProd" (obligatoria, 255), "Concent" (255), "FormaFarmaceutica" (150), "Fraccion" (100), "RegistroSanitario" (50), "Titular" (255) y "Estado" (obligatoria, 50)
  Y el estado es texto libre (por ejemplo "ACTIVO"), sin lista de valores permitidos

Escenario: Cargar productos válidos
  Dado que subí un archivo de 9 filas con 5 productos válidos y 4 con problemas
  Entonces veo "9 filas totales", "5 filas válidas" y "4 filas con error"
  Y una fila sin código muestra "El campo 'codProd' es obligatorio" y una sin estado "El campo 'estado' es obligatorio"
  Cuando hago clic en "Guardar válidos (5)"
  Entonces veo "Se guardaron 5 registros correctamente" y vuelvo a la búsqueda del catálogo

Escenario: Códigos repetidos sin distinguir mayúsculas
  Dado que dos filas tienen los códigos "DEMO-0008" y "demo-0008"
  Entonces ambas, también la primera, muestran "El valor '…' del campo 'codProd' está duplicado en el archivo"
  Y una fila con un código que ya existe muestra "El valor 'demo-0001' del campo 'codProd' ya existe en el sistema"

Escenario: Un texto demasiado largo pasa la vista previa y rechaza todo el guardado
  Dado que una fila tiene un registro sanitario de 60 caracteres
  Entonces la vista previa la muestra válida, porque no revisa longitudes
  Cuando hago clic en "Guardar válidos (1)"
  Entonces el validador del comando rechaza el envío completo
  Y veo "Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa." y no se guarda nada
```

> **Brecha conocida.** Comprobado en el manual (§8.5 y §11, diferencia 3): la vista previa no valida longitudes y el guardado rechaza todo el archivo si una fila supera el límite. Revisa los límites antes de subir (en especial `RegistroSanitario`, 50). No hay un botón para agregar un producto uno por uno: la carga masiva es la única forma desde la pantalla.

### HU-CAT-044 — Crear, editar y eliminar un producto DIGEMID (solo API)

**Como** integrador **quiero** crear, corregir y eliminar un producto del catálogo DIGEMID por API **para** mantenerlo sin pasar por un Excel.

| | |
|---|---|
| **Módulo** | Catálogo DIGEMID |
| **Pantalla** | Ninguna: no existe `/digemid-catalogo/nueva` y el detalle es de solo lectura |
| **Endpoint(s)** | `POST /api/v1/DIGEMID/catalogo`, `PUT /api/v1/DIGEMID/catalogo/{id}`, `DELETE /api/v1/DIGEMID/catalogo/{id}` |
| **Permiso** | `DIGEMID:Crear`, `DIGEMID:Actualizar`, `DIGEMID:Eliminar` |
| **Reglas** | [RN-DIG-001, RN-DIG-002, RN-DIG-003, RN-DIG-004, RN-DIG-005, RN-DIG-006, RN-DIG-007, RN-DIG-008](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Crear un producto
  Dado que llamo a "POST /api/v1/DIGEMID/catalogo" con código "DEMO-0010" y nombre "IBUPROFENO 400 MG TABLETA (DEMO)"
  Entonces la API responde 201 con el identificador

Escenario: Código y nombre obligatorios
  Dado que envío el código o el nombre vacíos o de solo espacios
  Entonces la API responde 400 con "DIGEMID.CodProd" o "DIGEMID.NomProd"

Escenario: Código repetido
  Dado que ya existe un producto no eliminado con ese código en mi empresa
  Entonces la API responde 409 "DigemidCatalogo.CodProd.Duplicado"
  Y al editar se excluye el propio producto

Escenario: Editar o eliminar un producto inexistente
  Dado que el producto no existe o está eliminado
  Cuando llamo a "PUT" o "DELETE"
  Entonces la API responde 404 "DigemidCatalogo.NotFound"

Escenario: Eliminar
  Dado que el producto existe
  Cuando llamo a "DELETE /api/v1/DIGEMID/catalogo/{id}"
  Entonces la API responde 204, la eliminación es lógica y no valida dependencias
```

> **Brecha conocida.** No hay validadores de crear ni editar: la longitud de los textos y la obligatoriedad del estado solo las impone la columna de la base de datos, y la respuesta ante un texto más largo que la columna o sin estado es **No verificado** (el manejador global solo traduce los errores SQL 2601, 2627 y 547; [RN-DIG-004, RN-DIG-005](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md)). El manual indica no usar estas operaciones.

---

## 13. Lotes de inventario

Un lote identifica una producción de un medicamento (número del fabricante, fechas de fabricación y caducidad). El lote **no** guarda cantidades: las existencias viven en el stock por sucursal. No hay carga masiva de lotes.

### HU-INV-001 — Registrar y editar un lote

**Como** almacenero **quiero** registrar el lote que recibo de un medicamento, con su número, su fecha de fabricación y su caducidad **para** poder luego ingresar su stock por sucursal.

| | |
|---|---|
| **Módulo** | Lotes de inventario (permisos `Inventarios`, rotulado «Inventario») |
| **Pantalla** | `/lotes/nueva` y `/lotes/:id` — «Lotes» |
| **Endpoint(s)** | `POST /api/v1/LotesInventario`, `PUT /api/v1/LotesInventario/{id}`, `GET /api/v1/LotesInventario/{id}` |
| **Permiso** | `Inventarios:Crear` (registrar), `Inventarios:Actualizar` (editar), `Inventarios:Leer`; para elegir el producto, además `Medicamentos:Leer` |
| **Reglas** | [RN-LOT-001, RN-LOT-002, RN-LOT-003, RN-LOT-004, RN-LOT-005, RN-LOT-006, RN-LOT-007, RN-LOT-008, RN-LOT-009, RN-LOT-010, RN-LOT-011, RN-LOT-012, RN-LOT-013, RN-LOT-016](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) |
| **Estado** | Implementada (la mayoría de los rechazos del servidor se ven como mensaje genérico) |

**Criterios de aceptación**

```gherkin
Escenario: Registrar un lote
  Dado que tengo "Inventario › Crear" y hago clic en "Nuevo" en "/lotes"
  Cuando escribo "Paracetamol" en "Producto" y elijo "Paracetamol Demo 500 mg" de la lista
  Y escribo Número de Lote "L-2026-001"
  Y elijo Fecha de Fabricación 01/09/2026 y Fecha de Caducidad 31/08/2028
  Y hago clic en "Guardar"
  Entonces veo "Lote guardado correctamente" y vuelvo a "/lotes"
  Y el lote nace sin existencias: el stock se registra aparte en "Stock por Sucursal"
  Y la API responde 201 con el identificador

Escenario: La fecha de fabricación es opcional
  Dado que dejo vacía la "Fecha de Fabricación"
  Cuando hago clic en "Guardar"
  Entonces se guarda y la fecha se ve como "—" en la tabla

Escenario: Campos obligatorios
  Dado que dejo vacíos el producto, el número de lote o la caducidad
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" debajo del campo que falta y no se envía nada
  Y el producto debe elegirse de la lista; si cambio su texto después de elegir, la elección se descarta
  Y "Número de Lote" no deja pasar de 100 caracteres, sin mensaje

Escenario: Caducidad vencida
  Dado que elijo una caducidad anterior a hoy
  Cuando hago clic en "Guardar"
  Entonces el validador rechaza la fecha (400 "CreateLoteInventarioCommand.FechaCaducidad")
  Y veo "Error al guardar el lote", porque el selector de fecha no limita nada y el rechazo no tiene mensaje propio

Escenario: Un lote que caduca hoy es válido todo ese día
  Dado que elijo como caducidad la fecha de hoy
  Cuando hago clic en "Guardar"
  Entonces se guarda, porque un lote se vende hasta el final de su día de caducidad
  Y se considera vencido cuando su caducidad es menor o igual que "ahora menos un día"

Escenario: Fabricación futura o posterior a la caducidad
  Dado que elijo una fecha de fabricación futura
  O una fabricación posterior a la caducidad al crear
  Cuando hago clic en "Guardar"
  Entonces la API responde 400 ("CreateLoteInventarioCommand.FechaFabricacion" o "LoteInventario.FechaFabricacion")
  Y veo "Error al guardar el lote"

Escenario: Número de lote repetido para el mismo producto
  Dado que el producto ya tiene un lote "L-2026-001"
  Cuando intento registrar otro lote con el mismo número para ese producto
  Entonces la API responde 400 "LoteInventario.Duplicate"
  Y veo "Error al guardar el lote"
  Pero si dos altas simultáneas llegan al índice único (409 "Db.Duplicado") veo "Ya existe un lote con ese número para el producto seleccionado"
  Y el mismo número puede repetirse en productos distintos

Escenario: Eliminar un lote libera su número
  Dado que eliminé el lote "L-2026-001" de un producto
  Cuando registro otro lote con ese número para el mismo producto
  Entonces se guarda sin conflicto

Escenario: Producto que ya no existe
  Dado que el producto fue eliminado
  Cuando hago clic en "Guardar"
  Entonces la API responde 404 "Medicamento.NotFound"
  Y veo "Error al guardar el lote"

Escenario: Editar un lote vigente
  Dado que abrí un lote y tengo "Inventario › Actualizar"
  Cuando hago clic en "Editar"
  Entonces el producto queda deshabilitado (no se puede cambiar) y puedo cambiar número, fabricación y caducidad
  Y al hacer clic en "Guardar" veo "Lote guardado correctamente" y vuelvo a "/lotes" (no me quedo en el detalle)

Escenario: No se puede editar un lote ya vencido
  Dado que la caducidad del lote ya venció
  Cuando intento guardar cualquier cambio, aunque sea corregir el número de lote
  Entonces la API vuelve a exigir caducidad vigente y responde 400 "UpdateLoteInventarioCommand.FechaCaducidad"
  Y veo "Error al guardar el lote" mientras no le ponga una caducidad vigente

Escenario: Al editar no se revisa fabricación contra caducidad
  Dado que cambio la fabricación a una fecha posterior a la caducidad (ambas válidas)
  Cuando hago clic en "Guardar"
  Entonces se guarda, porque esa comprobación solo existe al crear

Escenario: El lote ya no existe
  Dado que fue eliminado
  Cuando intento guardarlo o abrirlo
  Entonces la API responde 404 "LoteInventario.NotFound"
  Y al guardar veo "Error al guardar el lote" y al abrir "Error al cargar el lote"

Escenario: Cualquier 409 al guardar se muestra como lote duplicado
  Dado que la API responde 409 por una referencia inválida ("Db.ReferenciaInvalida") o un conflicto de concurrencia
  Entonces veo "Ya existe un lote con ese número para el producto seleccionado", aunque no sea la causa real

Escenario: Identificador al editar (solo API)
  Dado que llamo a "PUT /api/v1/LotesInventario/{id}"
  Entonces el identificador se toma de la URL y uno vacío se rechaza con "UpdateLoteInventarioCommand.Id"
```

> **Brecha conocida.** (1) El aviso «Ya existe un lote con ese número…» casi nunca aparece: el duplicado detectado por el handler responde 400 y la pantalla solo lo muestra ante un 409 ([01c1 hallazgo 1](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md#63-hallazgos-del-código)). (2) La fabricación posterior a la caducidad solo se rechaza al crear, y editar un lote vencido siempre falla (hallazgo 3). (3) Los selectores de fecha no limitan la caducidad ni la fabricación. (4) Un número de lote de solo espacios llega al servidor y se ve «Error al guardar el lote». (5) **No verificado:** si la comparación del número distingue mayúsculas (intercalación de la base) y la elección con teclado en el autocompletado ([01c1 §6.1](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md)). Los comandos de lote no generan evento de auditoría. El manual no recorrió esta pantalla.

### HU-INV-002 — Buscar, consultar y exportar lotes

**Como** almacenero **quiero** buscar lotes por producto, número y fechas **para** ver qué lotes tengo y cuándo caducan.

| | |
|---|---|
| **Módulo** | Lotes de inventario |
| **Pantalla** | `/lotes` — «Lotes» (grupo de menú «Inventario»; tarjeta «Búsqueda de Lotes») y `/lotes/:id` |
| **Endpoint(s)** | `GET /api/v1/LotesInventario`, `GET /api/v1/LotesInventario/{id}`, `POST /api/v1/LotesInventario/exportar` |
| **Permiso** | `Inventarios:Leer` (y `Medicamentos:Leer` para las sugerencias de producto) |
| **Reglas** | [RN-LOT-001, RN-LOT-013, RN-LOT-017, RN-LOT-019](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009, RN-IMP-011](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (sin indicador de lote vencido) |

**Criterios de aceptación**

```gherkin
Escenario: Filtros disponibles
  Dado que abro "/lotes"
  Entonces veo "Producto" ("Buscar producto..."), "Número de Lote", "Caducidad Desde", "Caducidad Hasta", "Fabricación Desde" y "Fabricación Hasta"

Escenario: Buscar
  Cuando hago clic en "Buscar"
  Entonces la tabla muestra "Producto" (enlace al detalle), "Número de Lote", "Fecha de Fabricación" ("—" si no tiene) y "Fecha de Caducidad" (dd/MM/yyyy)
  Y se ordena por fecha de creación, la más reciente primero
  Y no aparecen los lotes eliminados ni los de un medicamento eliminado

Escenario: El producto se filtra por texto
  Dado que escribo "paracetamol" en "Producto" (aunque no elija una sugerencia)
  Entonces la búsqueda usa el texto: nombres que contengan "paracetamol", sin distinguir mayúsculas
  Y el número de lote también es "contiene" sin distinguir mayúsculas

Escenario: Los rangos de fecha son inclusivos
  Dado que elijo "Caducidad Hasta" 31/12/2026
  Entonces los lotes que caducan el 31/12/2026 aparecen (se incluye el día completo)
  Y "Fabricación Desde" o "Fabricación Hasta" excluyen los lotes sin fecha de fabricación
  Y si "Desde" es posterior a "Hasta" la búsqueda no devuelve filas

Escenario: Ver el detalle
  Dado que hago clic en el producto de una fila
  Entonces se abre "/lotes/:id" con los campos deshabilitados y, con "Inventario › Actualizar", el botón "Editar"

Escenario: Lote que no se puede abrir
  Dado que el lote fue eliminado o es de otra empresa
  Entonces veo "Error al cargar el lote"

Escenario: Exportar con el formato de fecha del idioma
  Dado que busqué
  Cuando hago clic en "Exportar"
  Entonces se descarga "lotes.xlsx" con "Producto", "Número de Lote", "Fecha de Fabricación" y "Fecha de Caducidad"
  Y las fechas salen como dd/MM/yyyy con el idioma español y MM/dd/yyyy con otro idioma
  Y si falla la búsqueda o la exportación veo "Error al buscar lotes"
```

> **Nota.** La tabla no marca los lotes vencidos ni por vencer (no hay colores ni iconos de vencimiento). La API acepta además un parámetro libre `SearchTerm` que la pantalla no envía. Sin `Medicamentos:Leer`, las sugerencias de producto fallan sin aviso (**No verificado** qué ve el usuario).

### HU-INV-003 — Eliminar un lote

**Como** almacenero **quiero** eliminar un lote registrado por error **para** que no estorbe, siempre que no tenga stock ni ventas.

| | |
|---|---|
| **Módulo** | Lotes de inventario |
| **Pantalla** | `/lotes/:id` (botón «Eliminar» en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/LotesInventario/{id}` |
| **Permiso** | `Inventarios:Eliminar` |
| **Reglas** | [RN-LOT-001, RN-LOT-013, RN-LOT-014, RN-LOT-015](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-GEN-003](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar un lote sin stock ni ventas
  Dado que estoy editando un lote y tengo "Inventario › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces veo "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar este lote?"
  Y al hacer clic en "Confirmar" veo "Lote eliminado correctamente" y vuelvo a "/lotes"
  Y la API responde 204, la eliminación es lógica y el número de lote queda libre para ese producto

Escenario: Lote con stock o ventas
  Dado que el lote tiene una fila de stock en alguna sucursal (aunque la cantidad sea 0) o una línea de venta no eliminadas
  Cuando confirmo la eliminación
  Entonces la API responde 409 "LoteInventario.InUse"
  Y veo "No se puede eliminar un lote con stock o ventas registradas."

Escenario: Otro fallo
  Dado que el lote ya no existe (404 "LoteInventario.NotFound") o el servidor falla
  Entonces veo "Error al eliminar el lote"

Escenario: Sin permiso no hay botón
  Dado que mi rol no tiene "Inventario › Eliminar"
  Entonces no veo el botón "Eliminar"
```

> **Brecha conocida.** El guard solo revisa stock y ventas: un lote usado en transferencias, cuarentena, inmunizaciones u órdenes magistrales se puede eliminar, porque el borrado lógico no dispara las claves foráneas ([RN-LOT-014](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md)). La pantalla muestra el texto de «stock o ventas» ante cualquier 409.

### HU-INV-004 — Consultar los lotes de un producto (solo API)

**Como** integrador **quiero** listar los lotes de un producto **para** mostrarlos en otra aplicación.

| | |
|---|---|
| **Módulo** | Lotes de inventario |
| **Pantalla** | Ninguna: el SPA no llama a este endpoint |
| **Endpoint(s)** | `GET /api/v1/LotesInventario/producto/{productoId}` |
| **Permiso** | `Inventarios:Leer` |
| **Reglas** | [RN-LOT-018](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Lotes de un producto
  Dado que un producto tiene tres lotes
  Cuando llamo al endpoint con su identificador
  Entonces recibo una lista paginada ordenada por caducidad descendente

Escenario: Producto inexistente
  Dado que el identificador no corresponde a ningún producto
  Entonces recibo una lista vacía, porque no se comprueba que el producto exista
  Y si falta el producto en un lote, su nombre sale como "Unknown"
```

---

## 14. Stock por sucursal

El stock guarda cuántas unidades físicas hay de cada lote en cada sucursal. Cambia por ingreso, ajuste, venta, devolución, transferencias y cuarentena; no existe operación de eliminar una fila de stock.

### HU-INV-005 — Ingresar stock de un lote en una sucursal

**Como** almacenero **quiero** registrar cuántas unidades de un lote entran a una sucursal **para** que esas existencias queden disponibles.

| | |
|---|---|
| **Módulo** | Stock por sucursal (permisos `Inventarios`) |
| **Pantalla** | `/stock-por-sucursal/nueva` — «Stock por Sucursal» (ingreso de stock) |
| **Endpoint(s)** | `POST /api/v1/LotesInventario/stock`; `GET /api/v1/Sucursales/permitidas`; `GET /api/v1/LotesInventario` (búsqueda del lote) |
| **Permiso** | `Inventarios:Actualizar` (no se exige `Crear`) y `Inventarios:Leer` |
| **Reglas** | [RN-STK-001, RN-STK-002, RN-STK-003, RN-STK-004, RN-STK-005, RN-STK-006, RN-STK-007, RN-STK-008, RN-STK-009, RN-STK-010, RN-STK-011](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-GEN-023, RN-GEN-035](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Parcial (guardar sin elegir lote no muestra ningún mensaje; los textos de sucursal y lote obligatorios no se pintan) |

**Criterios de aceptación**

```gherkin
Escenario: Ingresar stock de un lote nuevo en una sucursal
  Dado que tengo "Inventario › Actualizar" y hago clic en "Nuevo" en "/stock-por-sucursal"
  Cuando escribo "Sede" en "Sucursal" y elijo "Sede Principal" de la lista
  Y escribo "L-2026-001" en "Lote" y elijo "L-2026-001 — Paracetamol Demo 500 mg"
  Y escribo 100 en "Cantidad Física"
  Y hago clic en "Guardar"
  Entonces veo "Stock registrado correctamente" y vuelvo a "/stock-por-sucursal"
  Y si no había stock de ese lote en esa sucursal, se crea una fila con 100 unidades
  Y la API responde 200 con el identificador de la fila

Escenario: Si el lote ya tenía stock en esa sucursal, la cantidad se suma
  Dado que "Sede Principal" ya tiene 100 unidades del lote "L-2026-001"
  Cuando ingreso 50 unidades del mismo lote en la misma sucursal
  Entonces el stock queda en 150 y el aviso es el mismo "Stock registrado correctamente"
  Y la pantalla nunca envía el indicador de reemplazo directo, así que desde el SPA el ingreso siempre suma

Escenario: Quién puede abrir la pantalla
  Dado que mi rol tiene "Inventario › Actualizar"
  Entonces veo el botón "Nuevo" en "/stock-por-sucursal" y puedo abrir "/stock-por-sucursal/nueva"
  Pero si solo tengo "Inventario › Crear" no veo "Nuevo" y la ruta me devuelve al Inicio con "No tienes permisos para acceder a esta pantalla."

Escenario: Solo se ofrecen mis sucursales
  Dado que escribo en "Sucursal"
  Entonces la lista contiene solo mi sucursal base, las asignadas a mi cuenta y las de mis roles (el rol "Admin" ve todas)
  Y si falla la carga veo "No se pudieron cargar tus sucursales"

Escenario: Cantidad negativa
  Dado que escribo -5 en "Cantidad Física"
  Cuando hago clic en "Guardar"
  Entonces veo "La cantidad no puede ser negativa" y no se envía nada
  Y el mismo mensaje aparece si el campo queda vacío; el valor inicial es 0 y 0 es válido
  Y la cantidad admite hasta 4 decimales

Escenario: Sucursal fuera de mi acceso (solo API)
  Dado que llamo a "POST /api/v1/LotesInventario/stock" con una sucursal que no tengo
  Entonces la API responde 403 "Inventario.Sucursal.NoPermitida"
  Y la pantalla lo muestra como "No tienes permiso para operar el stock de esta sucursal"

Escenario: Sucursal o lote que ya no existen
  Dado que la sucursal o el lote fueron eliminados
  Cuando hago clic en "Guardar"
  Entonces la API responde 404 "Sucursal.NotFound" (se comprueba antes que el permiso sobre la sucursal) o "LoteInventario.NotFound"
  Y veo "Error al registrar el stock"

Escenario: Se puede ingresar stock de un lote vencido o retenido
  Dado que el lote ya venció o está retenido en cuarentena
  Cuando ingreso su stock
  Entonces se registra sin error, porque el ingreso no comprueba vigencia ni cuarentena

Escenario: Dos ingresos simultáneos crean la misma fila
  Dado que otra persona ingresa el mismo lote en la misma sucursal al mismo tiempo
  Cuando hago clic en "Guardar"
  Entonces la API responde 409 "InventarioSucursal.RegistroConcurrente" para el segundo
  Y veo "Error al registrar el stock" y debo reintentar

Escenario: Otra operación cambió el mismo stock
  Dado que una venta o un ajuste modificó la fila entre la lectura y el guardado
  Entonces la API responde 409 "Concurrency.Conflict"
  Y veo "Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo."

Escenario: Una sucursal tiene una sola fila por lote
  Dado que la base tiene un índice único por sucursal y lote entre las filas no eliminadas
  Entonces nunca hay dos filas de stock del mismo lote en una sucursal
```

> **Brecha conocida.** (1) Si pulsas «Guardar» sin haber elegido un lote, por lectura del código `saveNew()` lanza una excepción de JavaScript: no se envía ninguna petición y no se ve ningún aviso. (2) Los textos «Debes seleccionar una sucursal» y «Debes seleccionar un lote» están declarados pero, por lectura del código, no llegan a pintarse porque los controles no tienen validadores. Ambos son **No verificado en ejecución** ([19e §2.2 y §7](../../ola-1-fundamentos/19-inventario-de-pantallas/19e-inventario-y-atencion.md), [01c1 hallazgo 2](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md#63-hallazgos-del-código)). (3) El mapa de errores es propio de la pantalla: los códigos `Db.*` y los demás caen en «Error al registrar el stock». (4) El ingreso por suma no genera evento de auditoría; el reemplazo directo (solo API) genera `STOCK_AJUSTAR`. (5) No hay orden de compra ni recepción de proveedor: el stock se ingresa a mano ([proceso 3](../14-procesos-de-negocio.md)). El manual no recorrió esta pantalla.

### HU-INV-006 — Ajustar el stock de un lote en una sucursal

**Como** almacenero **quiero** corregir la cantidad física de un lote en una sucursal cuando el conteo real difiere **para** que el sistema refleje lo que hay en el estante.

| | |
|---|---|
| **Módulo** | Stock por sucursal |
| **Pantalla** | `/stock-por-sucursal/:id` — «Detalle de Stock» |
| **Endpoint(s)** | `GET /api/v1/StockPorSucursal/{id}`, `PUT /api/v1/StockPorSucursal/{id}/ajustar` |
| **Permiso** | `Inventarios:Leer` (ver), `Inventarios:Actualizar` (ajustar) |
| **Reglas** | [RN-STK-001, RN-STK-004, RN-STK-006, RN-STK-011, RN-STK-012, RN-STK-013](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-GEN-013, RN-GEN-035](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Ajustar la cantidad
  Dado que abrí el registro de stock del lote "L-2026-001" en "Sede Principal" con 150 unidades y tengo "Inventario › Actualizar"
  Cuando hago clic en "Editar"
  Entonces solo se habilita "Cantidad Física"; sucursal, producto, lote y caducidad son de solo lectura
  Cuando escribo 140 y hago clic en "Guardar"
  Entonces veo "Cantidad ajustada correctamente" y vuelvo a "/stock-por-sucursal" (no me quedo en el detalle)
  Y la cantidad física pasa a ser 140: el ajuste fija el valor, no suma
  Y se registra el evento de auditoría "STOCK_AJUSTAR" con la nueva cantidad

Escenario: Cantidad negativa
  Dado que escribo -1 o dejo vacío "Cantidad Física"
  Cuando hago clic en "Guardar"
  Entonces veo "La cantidad no puede ser negativa" y no se envía nada
  Y la cantidad 0 es válida

Escenario: Sin el permiso no hay botón
  Dado que mi rol solo tiene "Inventario › Leer"
  Entonces veo el detalle pero no el botón "Editar"

Escenario: Cancelar el ajuste
  Dado que estoy editando la cantidad
  Cuando hago clic en "Cancelar"
  Entonces la cantidad vuelve a la leída y el detalle vuelve al modo ver

Escenario: Sucursal fuera de mi acceso
  Dado que el registro pertenece a una sucursal a la que no tengo acceso
  Cuando abro el detalle
  Entonces la API responde 404 "InventarioSucursal.NotFound", la tarjeta no se pinta y veo "Error al cargar el registro de stock"
  Pero si intento ajustarlo (por API) la respuesta es 403 "Inventario.Sucursal.NoPermitida" y veo "No tienes permiso para operar el stock de esta sucursal"

Escenario: Otra operación cambió el stock
  Dado que una venta modificó la fila mientras yo editaba
  Cuando hago clic en "Guardar"
  Entonces la API responde 409 "Concurrency.Conflict"
  Y veo "Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo."

Escenario: El registro ya no existe
  Dado que el registro fue eliminado o es de otra empresa
  Cuando intento ajustarlo
  Entonces la API responde 404 "InventarioSucursal.NotFound"
  Y veo "Error al ajustar la cantidad"

Escenario: Identificador del registro (solo API)
  Dado que llamo a "PUT /api/v1/StockPorSucursal/{id}/ajustar"
  Entonces el controlador toma el identificador de la URL y uno vacío se rechaza con "AdjustStockCommand.Id"
```

> **Brecha conocida.** La API responde 403 al ajustar una sucursal no permitida pero 404 al consultar el mismo registro ([01c1 hallazgo 8](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md#63-hallazgos-del-código)). Un valor que desborde `DECIMAL(12,4)` es **No verificado**. No existe botón ni endpoint para eliminar una fila de stock. El manual no recorrió esta pantalla.

### HU-INV-007 — Consultar y exportar el stock por sucursal

**Como** almacenero **quiero** consultar cuánto stock hay de cada lote en mis sucursales **para** decidir qué reponer.

| | |
|---|---|
| **Módulo** | Stock por sucursal |
| **Pantalla** | `/stock-por-sucursal` — «Stock por Sucursal» (grupo de menú «Inventario»; tarjeta «Consulta de Stock por Sucursal») |
| **Endpoint(s)** | `GET /api/v1/StockPorSucursal`, `POST /api/v1/StockPorSucursal/exportar`, `GET /api/v1/Sucursales/permitidas` |
| **Permiso** | `Inventarios:Leer` (y `Medicamentos:Leer` para las sugerencias de producto) |
| **Reglas** | [RN-STK-001, RN-STK-014, RN-STK-015, RN-STK-016, RN-STK-017](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-GEN-009, RN-GEN-023](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009, RN-IMP-011](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Filtros disponibles
  Dado que abro "/stock-por-sucursal"
  Entonces veo "Sucursal" ("Buscar sucursal..."), "Producto" ("Buscar producto..."), "Número de Lote", "Caducidad Desde", "Caducidad Hasta", "Cantidad Mínima", "Cantidad Máxima" y la casilla "Solo con stock disponible" marcada

Escenario: Por defecto solo veo filas con stock
  Dado que la casilla "Solo con stock disponible" está marcada
  Cuando hago clic en "Buscar"
  Entonces veo solo filas con cantidad mayor que 0
  Y si desmarco la casilla veo también las filas con cantidad 0
  Y "Limpiar" vuelve a marcar la casilla

Escenario: Solo veo el stock de mis sucursales
  Dado que mi rol no es "Admin"
  Entonces la lista y el detalle solo devuelven stock de mi sucursal base, las asignadas a mi cuenta y las de mis roles
  Y el rol "Admin" ve todas
  Y no aparecen registros cuya sucursal, lote o producto estén eliminados

Escenario: Tabla de resultados
  Dado que hay resultados
  Entonces veo "Sucursal" (enlace al detalle), "Producto", "Número de Lote", "Fecha de Caducidad" y "Cantidad Física" (a la derecha)
  Y se ordena por sucursal, nombre del producto y caducidad ascendente

Escenario: Filtros de caducidad y cantidad
  Dado que elijo "Caducidad Hasta" 31/12/2026 y "Cantidad Mínima" 10
  Entonces veo los lotes que caducan hasta el 31/12/2026 (día completo incluido) con 10 o más unidades

Escenario: Un lote vencido sigue apareciendo pero no es vendible
  Dado que un lote ya venció y tiene stock
  Entonces sigue apareciendo en la lista con "Solo con stock disponible" marcada
  Y la tabla no tiene indicador de vencido
  Y no es "stock vendible": para vender cuenta solo cantidad mayor que 0, lote no vencido y sin retención activa en cuarentena

Escenario: Exportar
  Dado que busqué
  Cuando hago clic en "Exportar"
  Entonces se descarga "stock-por-sucursal.xlsx" con "Sucursal", "Producto", "Número de Lote", "Fecha de Caducidad" y "Cantidad Física"
  Y las fechas salen dd/MM/yyyy con el idioma español y MM/dd/yyyy con otro idioma
  Y respeta los filtros escritos, incluida la casilla "Solo con stock disponible"

Escenario: Errores
  Dado que falla la búsqueda o la exportación
  Entonces veo "Error al consultar el stock"
  Y si falla la carga de sucursales al escribir en "Sucursal" veo "No se pudieron cargar tus sucursales"
```

> **Nota.** Desde el dashboard, los indicadores de lotes vencidos y por vencer abren esta pantalla con la caducidad ya filtrada (historias de la parte C). **No verificado en la aplicación:** el manual no recorrió esta pantalla.

### HU-INV-008 — Consultar el stock de un medicamento en todas las sucursales (solo API)

**Como** integrador **quiero** el total y el desglose del stock de un medicamento por sucursal y lote **para** mostrarlo en otra aplicación.

| | |
|---|---|
| **Módulo** | Stock por sucursal |
| **Pantalla** | Ninguna: el SPA no llama a este endpoint |
| **Endpoint(s)** | `GET /api/v1/LotesInventario/stock/medicamento/{medicamentoId}` |
| **Permiso** | `Inventarios:Leer` |
| **Reglas** | [RN-STK-018](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) |
| **Estado** | Solo API |

**Criterios de aceptación**

```gherkin
Escenario: Stock de un medicamento
  Dado que el medicamento tiene lotes en dos sucursales
  Cuando llamo al endpoint con su identificador
  Entonces recibo el total y el desglose por sucursal y por lote, ordenado por caducidad

Escenario: Suma toda la empresa
  Dado que soy un usuario que no tiene acceso a una de las sucursales
  Entonces la suma incluye igualmente todas las sucursales de la empresa
  Y también cuenta lotes vencidos y filas con cantidad 0

Escenario: Medicamento inexistente
  Dado que el identificador no corresponde a ningún medicamento
  Entonces la API responde 404 "Medicamento.NotFound"
```

---

## 15. Pacientes

Catálogo de pacientes y clientes: tipo y número de documento, nombre y apellidos, fecha de nacimiento y contacto. Los pacientes se usan en las recetas y en las ventas.

### HU-INV-009 — Registrar y editar un paciente

**Como** personal de atención **quiero** registrar a un paciente o cliente con su documento de identidad **para** asociarlo a recetas y ventas.

| | |
|---|---|
| **Módulo** | Pacientes |
| **Pantalla** | `/pacientes/nueva` y `/pacientes/:id` — «Pacientes» (grupo de menú «Pacientes & Atención») |
| **Endpoint(s)** | `POST /api/v1/Pacientes`, `PUT /api/v1/Pacientes/{id}`, `GET /api/v1/Pacientes/{id}` |
| **Permiso** | `Pacientes:Crear`, `Pacientes:Actualizar`, `Pacientes:Leer` |
| **Reglas** | [RN-PAC-001, RN-PAC-002, RN-PAC-003, RN-PAC-004, RN-PAC-005, RN-PAC-006, RN-PAC-007, RN-PAC-008, RN-PAC-009](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (el formulario y la API difieren en algunos límites) |

**Criterios de aceptación**

```gherkin
Escenario: Registrar un paciente con DNI
  Dado que tengo "Pacientes › Crear" y hago clic en "Nuevo"
  Cuando elijo Tipo de Documento "DNI" y escribo Documento de Identidad "12345678"
  Y escribo Nombre y Apellidos "Rosa Paciente Demo", elijo Fecha de Nacimiento 15/03/1985 y escribo Contacto Primario "987654321"
  Y hago clic en "Guardar"
  Entonces veo "Paciente guardado correctamente" y vuelvo a "/pacientes"
  Y la API responde 200 con el identificador (no 201)

Escenario: Tipos de documento admitidos
  Dado que abro la lista "Tipo de Documento"
  Entonces veo "DNI", "Carné de extranjería", "RUC" y "Pasaporte", guardados con los códigos SUNAT 1, 4, 6 y 7

Esquema del escenario: El número debe cumplir el formato de su tipo
  Dado que elijo <tipo> y escribo <número>
  Cuando hago clic en "Guardar"
  Entonces <resultado>

  Ejemplos:
    | tipo                 | número                       | resultado                                                                                                                                              |
    | DNI                  | 12345678                     | se guarda                                                                                                                                              |
    | DNI                  | 1234567                      | veo "El número no corresponde al tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números)." y no se envía |
    | RUC                  | 20123456789                  | se guarda                                                                                                                                              |
    | Carné de extranjería | AB1234567890                 | se guarda (de 1 a 12 letras o números)                                                                                                                 |
    | Pasaporte            | PA123456789012345678         | se guarda (20 caracteres: de 1 a 20 letras o números)                                                                                                  |

Escenario: Campos obligatorios
  Dado que dejo vacíos el tipo, el número, el nombre o la fecha de nacimiento
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" debajo de cada campo y no se envía nada
  Y "Contacto Primario" es opcional

Escenario: Rechazo del servidor por tipo o formato
  Dado que la API recibe un tipo fuera de 1, 4, 6, 7 o un número que no cumple el formato
  Entonces responde 400 y la pantalla muestra "Selecciona un tipo de documento válido." o el mensaje de formato del número
  Y los guards se evalúan en este orden: documento vacío, longitud, tipo, formato, nombre, fecha y contacto; solo se informa el primero que falla

Escenario: Documento repetido
  Dado que existe un paciente con el documento "12345678"
  Cuando intento registrar otro con el mismo número, aunque sea de otro tipo de documento
  Entonces la API responde 409 "Paciente.DocIdentidadGub.Duplicado"
  Y veo "Ya existe un paciente con ese documento de identidad."
  Y al editar se excluye al propio paciente

Escenario: Fecha de nacimiento futura
  Dado que elijo una fecha de nacimiento posterior a hoy (el selector no la limita)
  Cuando hago clic en "Guardar"
  Entonces la API responde 400 "PacienteCliente.FechaNacimiento"
  Y veo "Error al guardar el paciente"

Escenario: Contacto primario de más de 100 caracteres
  Dado que escribo 120 caracteres en "Contacto Primario" (el formulario admite hasta 150)
  Cuando hago clic en "Guardar"
  Entonces la API lo rechaza y veo "Error al guardar el paciente"

Escenario: Editar un paciente
  Dado que abrí un paciente y tengo "Pacientes › Actualizar"
  Cuando hago clic en "Editar", cambio el contacto y hago clic en "Guardar"
  Entonces veo "Paciente guardado correctamente", me quedo en el detalle y los datos se recargan
  Y al cambiar el tipo de documento el número se revalida

Escenario: Paciente antiguo sin tipo de documento
  Dado que el paciente se registró antes de existir el tipo de documento
  Cuando hago clic en "Editar"
  Entonces el tipo aparece vacío y debo elegirlo para poder guardar

Escenario: El paciente ya no existe
  Dado que fue eliminado
  Cuando intento guardarlo o abrirlo
  Entonces la API responde 404 "Paciente.NotFound"
  Y al guardar veo "Error al guardar el paciente" y al abrir "Error al cargar el paciente"
```

> **Brecha conocida.** (1) El formulario limita el documento a 20 caracteres (la API admite 50) y el contacto a 150 (la API 100); de 101 a 150 caracteres de contacto el usuario ve el genérico. (2) La fecha de nacimiento futura no se valida en el formulario; el servidor la rechaza con un código sin traducción. (3) Un nombre de solo espacios muestra «Error al guardar el paciente». (4) Cuando el número viene vacío o excede la longitud, el código de error del validador se asocia al mensaje de formato ([RN-PAC-003](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md)). **No verificado en la aplicación:** el manual no recorrió esta pantalla; los mensajes salen de 19e §3.2 y 01b.

### HU-INV-010 — Buscar, consultar y exportar pacientes

**Como** personal de atención **quiero** buscar pacientes por documento, nombre y fecha de nacimiento **para** encontrar a la persona que atiendo.

| | |
|---|---|
| **Módulo** | Pacientes |
| **Pantalla** | `/pacientes` — «Pacientes» (tarjeta «Búsqueda de Pacientes») |
| **Endpoint(s)** | `GET /api/v1/Pacientes`, `POST /api/v1/Pacientes/exportar` |
| **Permiso** | `Pacientes:Leer` |
| **Reglas** | [RN-PAC-001, RN-PAC-011](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009, RN-IMP-011](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Filtros disponibles
  Dado que abro "/pacientes"
  Entonces veo "Tipo de Documento" ("Todos", "DNI", "Carné de extranjería", "RUC", "Pasaporte"), "Documento de Identidad", "Nombre y Apellidos", "Fecha Nacimiento Desde" y "Fecha Nacimiento Hasta"

Escenario: Buscar
  Cuando hago clic en "Buscar"
  Entonces la tabla muestra "Doc. Identidad" (enlace al detalle), "Tipo Doc." ("—" si no tiene), "Nombre y Apellidos", "Fecha Nacimiento" (dd/MM/yyyy) y "Contacto Primario" ("—" si no tiene)
  Y se ordena por nombre y apellidos y luego por identificador

Escenario: Filtros
  Dado que elijo el tipo "DNI"
  Entonces el tipo se compara exacto
  Y el documento y el nombre se comparan con "contiene"
  Y las fechas de nacimiento "Desde" y "Hasta" son inclusivas

Escenario: Sin coincidencias
  Cuando ningún paciente cumple
  Entonces veo "Resultados (0)" y la fila vacía dice "Resultados"

Escenario: Exportar
  Cuando hago clic en "Exportar"
  Entonces se descarga "pacientes.xlsx" con "Doc. Identidad", "Tipo Doc." (con la abreviatura "DNI", "CE", "RUC" o "PASAPORTE"), "Nombre y Apellidos", "Fecha Nacimiento" y "Contacto Primario"
  Y las fechas salen dd/MM/yyyy con el idioma español y MM/dd/yyyy con otro idioma
  Y si falla veo "Error al buscar pacientes", con "Referencia: …" si la respuesta trae referencia de soporte
```

### HU-INV-011 — Eliminar un paciente

**Como** personal de atención **quiero** eliminar un paciente registrado por error **para** no tener duplicados, sin perder el historial de recetas y ventas.

| | |
|---|---|
| **Módulo** | Pacientes |
| **Pantalla** | `/pacientes/:id` (botón «Eliminar» en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/Pacientes/{id}` |
| **Permiso** | `Pacientes:Eliminar` |
| **Reglas** | [RN-PAC-001, RN-PAC-009, RN-PAC-010](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar un paciente sin registros asociados
  Dado que estoy editando un paciente sin recetas, ventas, citas ni inmunizaciones y tengo "Pacientes › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces veo "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar este paciente?"
  Y al hacer clic en "Confirmar" veo "Paciente eliminado correctamente" y vuelvo a "/pacientes"
  Y la API responde 200 con el identificador

Escenario: Paciente con recetas, ventas u otros registros
  Dado que lo referencian recetas, ventas, servicios de agenda o servicios clínicos de inmunización no eliminados
  Cuando confirmo la eliminación
  Entonces la API responde 409 "Paciente.InUse" (bloqueo duro; se comprueban en ese orden)
  Y veo "No puedes eliminar este paciente porque tiene recetas, ventas u otros registros asociados."

Escenario: Otro fallo
  Dado que el paciente no existe
  Cuando confirmo
  Entonces la API responde 404 con el código genérico "NotFound"
  Y veo "Error al eliminar el paciente"
```

> **Brecha conocida.** La pantalla muestra el texto de «en uso» ante cualquier 409. **No verificado en la aplicación:** el bloqueo (el manual no recorrió esta pantalla; viene de 19e §6).

### HU-INV-012 — Cargar pacientes desde un Excel

**Como** personal de atención **quiero** cargar muchos pacientes de una vez **para** migrar mi cartera de clientes.

| | |
|---|---|
| **Módulo** | Pacientes (carga masiva) |
| **Pantalla** | `/pacientes/carga-masiva` — «Carga Masiva» |
| **Endpoint(s)** | `GET /api/v1/Pacientes/plantilla`, `POST /api/v1/Pacientes/previsualizar`, `POST /api/v1/Pacientes/carga-masiva` |
| **Permiso** | `Pacientes:Crear` (además de `Leer`) |
| **Reglas** | [RN-PAC-002, RN-PAC-003, RN-PAC-004, RN-PAC-005, RN-PAC-006, RN-PAC-007, RN-PAC-008, RN-PAC-012, RN-PAC-013, RN-PAC-014, RN-PAC-015, RN-PAC-016, RN-PAC-017](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (la primera columna de la vista previa podría mostrar una clave sin traducir) |

**Criterios de aceptación**

Cumple los escenarios CM-1 a CM-5 de la [sección 1.3](#13-escenarios-comunes-de-la-carga-masiva). Además:

```gherkin
Escenario: Columnas de la plantilla
  Dado que descargué "plantilla-pacientes.xlsx"
  Entonces tiene "TipoDocumento", "DocIdentidadGub", "NombreApellidos" y "FechaNacimiento" (obligatorias) y "ContactoPrimario" (opcional)
  Y los límites por fila son 20, 50, 200, 10 y 100 caracteres

Escenario: El tipo de documento como código o abreviatura
  Dado que escribo en "TipoDocumento" 1, 4, 6, 7, "DNI", "CE", "RUC" o "PASAPORTE" (sin distinguir mayúsculas)
  Entonces la fila es válida en ese campo
  Y cualquier otro texto muestra "'<valor>' no es un valor válido para 'tipoDocumento'"

Escenario: El número cumple el formato de su tipo
  Dado que una fila tiene tipo "DNI" y número de 7 dígitos
  Entonces la vista previa muestra "'docIdentidadGub' no tiene el formato de su tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números)"

Escenario: La fecha de nacimiento exactamente como dd/MM/yyyy
  Dado que una fila tiene la fecha "1985-03-15"
  Entonces muestra "El formato de 'fechaNacimiento' no es válido (usa DD/MM/YYYY)"
  Y una fecha futura muestra "'fechaNacimiento' no puede ser una fecha futura"

Escenario: Documento existente o repetido
  Dado que el documento ya existe en mi empresa o aparece en dos filas del archivo (sin distinguir mayúsculas)
  Entonces la vista previa muestra "El valor '<valor>' del campo 'docIdentidadGub' ya existe en el sistema" o "…está duplicado en el archivo"
  Y al guardar, de las repetidas se guarda solo la primera

Escenario: Campos vacíos o demasiado largos
  Entonces la vista previa muestra "El campo '<campo>' es obligatorio" o "'<campo>' no puede superar los N caracteres" (N = 50 para el documento y 200 para el nombre)
```

> **Brecha conocida.** La columna del tipo de documento se declara con la clave `fields.tipoDocumento`, que no existe en la raíz de `es.json` (verificado: la raíz no la define): por lectura del código, el encabezado de esa columna mostraría el texto crudo `fields.tipoDocumento` (**No verificado en ejecución**). La columna del documento se rotula «Doc. Identidad» aquí y «Documento de Identidad» en el formulario ([19e §3.3](../../ola-1-fundamentos/19-inventario-de-pantallas/19e-inventario-y-atencion.md)).

---

## 16. Profesionales de salud

Catálogo de prescriptores que se eligen como médico en las recetas. Solo tiene tres datos: número de registro (el identificador del profesional), nombre del prescriptor y dirección de la clínica. No existen especialidad ni tipo de profesional.

### HU-INV-013 — Registrar y editar un profesional de salud

**Como** personal de atención **quiero** registrar a los médicos que prescriben **para** elegirlos como médico en las recetas.

| | |
|---|---|
| **Módulo** | Profesionales de salud |
| **Pantalla** | `/profesionales-salud/nueva` y `/profesionales-salud/:id` — «Profesionales de Salud» (grupo de menú «Pacientes & Atención») |
| **Endpoint(s)** | `POST /api/v1/profesionalessalud`, `PUT /api/v1/profesionalessalud/{id}`, `GET /api/v1/profesionalessalud/{id}` |
| **Permiso** | `ProfesionalesSalud:Crear`, `ProfesionalesSalud:Actualizar`, `ProfesionalesSalud:Leer` |
| **Reglas** | [RN-PRO-001, RN-PRO-002, RN-PRO-003, RN-PRO-004, RN-PRO-005, RN-PRO-006](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada (el formulario y la API difieren en algunos límites) |

**Criterios de aceptación**

```gherkin
Escenario: Registrar un profesional
  Dado que tengo "Profesionales de Salud › Crear" y hago clic en "Nuevo"
  Cuando escribo Número de Registro "CMP-12345", Nombre del Prescriptor "Dr. Juan Médico Demo" y Dirección de Clínica "Av. Ficticia 456, Lima"
  Y hago clic en "Guardar"
  Entonces veo "Profesional de salud guardado correctamente" y vuelvo a "/profesionales-salud"
  Y la API responde 201 con el identificador

Escenario: Campos obligatorios
  Dado que dejo vacíos el número de registro o el nombre del prescriptor
  Cuando hago clic en "Guardar"
  Entonces veo "Este campo es obligatorio" debajo del campo y no se envía nada
  Y "Dirección de Clínica" es opcional

Escenario: Número de registro repetido
  Dado que existe un profesional con el número "CMP-12345"
  Cuando intento registrar otro con el mismo número
  Entonces la API responde 409 "ProfesionalSalud.NumeroRegistro.Duplicado"
  Y veo "Ya existe un profesional de salud con ese número de registro."
  Y al editar se excluye al propio profesional

Escenario: Longitudes del formulario más amplias que las de la API
  Dado que escribo 180 caracteres en "Nombre del Prescriptor" (el formulario admite hasta 200) o 280 en "Dirección de Clínica" (hasta 300)
  Cuando hago clic en "Guardar"
  Entonces la API lo rechaza (nombre hasta 150, dirección hasta 255, número hasta 50)
  Y veo "Error al guardar el profesional de salud"

Escenario: Editar un profesional
  Dado que abrí un profesional y tengo "Profesionales de Salud › Actualizar"
  Cuando hago clic en "Editar", cambio la dirección y hago clic en "Guardar"
  Entonces veo "Profesional de salud guardado correctamente", me quedo en el detalle y los datos se recargan
  Y al editar la validación la hace la entidad (no existe validador del comando de edición)

Escenario: El profesional ya no existe
  Dado que fue eliminado
  Cuando intento guardarlo o abrirlo
  Entonces la API responde 404 "ProfesionalSalud.NotFound"
  Y al guardar veo "Error al guardar el profesional de salud" y al abrir "Error al cargar el profesional de salud"
```

> **Brecha conocida.** Un número o nombre de solo espacios muestra «Error al guardar el profesional de salud». **No verificado en la aplicación:** el manual no recorrió esta pantalla; los mensajes salen de 19e §4.2 y 01b.

### HU-INV-014 — Buscar y exportar profesionales de salud

**Como** personal de atención **quiero** buscar profesionales por número de registro o nombre **para** ubicar al médico de una receta.

| | |
|---|---|
| **Módulo** | Profesionales de salud |
| **Pantalla** | `/profesionales-salud` — «Profesionales de Salud» (tarjeta «Búsqueda de Profesionales de Salud») |
| **Endpoint(s)** | `GET /api/v1/profesionalessalud`, `POST /api/v1/profesionalessalud/exportar` |
| **Permiso** | `ProfesionalesSalud:Leer` |
| **Reglas** | [RN-PRO-001, RN-PRO-008](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), [RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Buscar
  Dado que abro "/profesionales-salud"
  Entonces veo los filtros "Número de Registro" y "Nombre del Prescriptor" ("contiene")
  Cuando hago clic en "Buscar"
  Entonces la tabla muestra "Número de Registro" (enlace al detalle), "Nombre del Prescriptor" y "Dirección de Clínica" ("—" si no tiene), ordenados por nombre del prescriptor y luego por identificador

Escenario: Sin coincidencias
  Cuando ningún profesional cumple
  Entonces veo "Resultados (0)" y la fila vacía dice "Resultados"

Escenario: Exportar
  Cuando hago clic en "Exportar"
  Entonces se descarga "profesionales-salud.xlsx" con "Número de Registro", "Nombre del Prescriptor" y "Dirección de Clínica", con los mismos filtros
  Y si falla veo "Error al buscar profesionales de salud", con "Referencia: …" si la respuesta trae referencia de soporte
```

### HU-INV-015 — Eliminar un profesional de salud

**Como** personal de atención **quiero** eliminar un profesional registrado por error **para** no tener duplicados, sin romper las recetas que ya lo usan.

| | |
|---|---|
| **Módulo** | Profesionales de salud |
| **Pantalla** | `/profesionales-salud/:id` (botón «Eliminar» en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/profesionalessalud/{id}` |
| **Permiso** | `ProfesionalesSalud:Eliminar` |
| **Reglas** | [RN-PRO-001, RN-PRO-006, RN-PRO-007](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar un profesional sin recetas
  Dado que estoy editando un profesional sin recetas y tengo "Profesionales de Salud › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces veo "Confirmar eliminación" con "¿Estás seguro de que deseas eliminar este profesional de salud?"
  Y al hacer clic en "Confirmar" veo "Profesional de salud eliminado correctamente" y vuelvo a "/profesionales-salud"
  Y la API responde 204

Escenario: Profesional con recetas
  Dado que alguna receta no eliminada lo tiene como médico
  Cuando confirmo la eliminación
  Entonces la API responde 409 "ProfesionalSalud.InUse" (bloqueo duro)
  Y veo "No puedes eliminar este profesional de salud porque tiene recetas asociadas."

Escenario: Otro fallo
  Dado que el profesional ya no existe (404 "ProfesionalSalud.NotFound") o el servidor falla
  Entonces veo "Error al eliminar el profesional de salud"
```

> **Brecha conocida.** La pantalla muestra el texto de «recetas asociadas» ante cualquier 409. **No verificado en la aplicación:** el bloqueo.

### HU-INV-016 — Cargar profesionales de salud desde un Excel

**Como** personal de atención **quiero** cargar muchos profesionales de una vez **para** completar el catálogo de prescriptores.

| | |
|---|---|
| **Módulo** | Profesionales de salud (carga masiva) |
| **Pantalla** | `/profesionales-salud/carga-masiva` — «Carga Masiva» |
| **Endpoint(s)** | `GET /api/v1/profesionalessalud/plantilla`, `POST /api/v1/profesionalessalud/previsualizar`, `POST /api/v1/profesionalessalud/carga-masiva` |
| **Permiso** | `ProfesionalesSalud:Crear` (además de `Leer`) |
| **Reglas** | [RN-PRO-002, RN-PRO-003, RN-PRO-004, RN-PRO-005, RN-PRO-009, RN-PRO-010, RN-PRO-011, RN-PRO-012](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

Cumple los escenarios CM-1 a CM-5 de la [sección 1.3](#13-escenarios-comunes-de-la-carga-masiva). Además:

```gherkin
Escenario: Columnas de la plantilla
  Dado que descargué la plantilla ("plantilla-profesionales-salud.xlsx")
  Entonces tiene "NumeroRegistro" (obligatoria, máximo 50 caracteres), "NombrePrescriptor" (obligatoria, 150) y "DireccionClinica" (opcional, 255)
  Y la vista previa rotula la tercera columna "Dirección Clínica"

Escenario: Reglas por fila en la vista previa
  Entonces un campo vacío muestra "El campo 'numeroRegistro' es obligatorio" (o 'nombrePrescriptor')
  Y un texto demasiado largo muestra "'nombrePrescriptor' no puede superar los 150 caracteres"
  Y un número de registro que ya existe muestra "El valor '<valor>' del campo 'numeroRegistro' ya existe en el sistema"
  Y un número repetido en el archivo marca todas sus filas con "…está duplicado en el archivo"

Escenario: Al guardar se omiten las repetidas
  Dado que, aun así, llegan al guardado números repetidos o ya existentes
  Entonces el servidor guarda solo la primera de cada número y omite las demás sin error
  Y veo "Se guardaron N registros correctamente"
```

> **Nota.** El manual no recorrió esta carga.

---

## 17. Recetas médicas y escáner con IA

Una receta es solo su **cabecera**: paciente, médico, fecha de expedición, repeticiones máximas, número de receta e indicaciones de uso. No guarda los medicamentos prescritos ni tiene vigencia por fecha; la relación con los productos se da al venderlos (parte C). El escáner con IA interpreta la foto de una receta pero no crea ni guarda nada.

### HU-INV-017 — Registrar y editar una receta médica

**Como** cajero o personal de atención **quiero** registrar la cabecera de una receta (paciente, médico, fecha, repeticiones y número) **para** poder asociarla a una venta de productos con receta.

| | |
|---|---|
| **Módulo** | Recetas médicas |
| **Pantalla** | `/recetas-medicas/nueva` y `/recetas-medicas/:id` — «Recetas Médicas» (grupo de menú «Pacientes & Atención»); también el enlace «registrar receta» del punto de venta, que abre `/recetas-medicas/nueva` en otra pestaña |
| **Endpoint(s)** | `POST /api/v1/Recetas`, `PUT /api/v1/Recetas/{id}`, `GET /api/v1/Recetas/{id}` |
| **Permiso** | `Recetas:Crear`, `Recetas:Actualizar`, `Recetas:Leer`; para elegir paciente y médico, además `Pacientes:Leer` y `ProfesionalesSalud:Leer` |
| **Reglas** | [RN-REC-001, RN-REC-002, RN-REC-003, RN-REC-004, RN-REC-005, RN-REC-006, RN-REC-007, RN-REC-008, RN-REC-009, RN-REC-010, RN-REC-011, RN-REC-012, RN-REC-015, RN-REC-031](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) |
| **Estado** | Implementada (la receta no tiene líneas de medicamentos ni vigencia por fecha) |

**Criterios de aceptación**

```gherkin
Escenario: Registrar una receta
  Dado que tengo "Recetas Médicas › Crear" y hago clic en "Nuevo"
  Cuando escribo "Rosa" en "Paciente" y elijo "Rosa Paciente Demo — 12345678" de la lista
  Y escribo "Juan" en "Médico" y elijo "Dr. Juan Médico Demo — CMP-12345"
  Y elijo Fecha de Expedición de hoy, Repeticiones Máximas 0 y Número de receta "R-0001"
  Y hago clic en "Guardar"
  Entonces veo "Receta creada correctamente" y vuelvo a "/recetas-medicas"
  Y la API responde 200 con el identificador (no 201)
  Y el número se guarda sin espacios al inicio ni al final, y uno vacío queda como "sin número"

Escenario: Paciente y médico obligatorios
  Dado que no elegí paciente o médico
  Cuando hago clic en "Guardar"
  Entonces veo "Debes seleccionar un paciente" o "Debes seleccionar un médico" y no se envía nada

Escenario: Fecha de expedición
  Dado que dejo vacía la "Fecha de Expedición"
  Entonces veo "La fecha de expedición es requerida" y no se envía nada
  Y el selector no deja elegir una fecha posterior a hoy; si escribo una a mano veo el mismo mensaje
  Y el sistema no vence recetas por antigüedad: cualquier fecha pasada es válida

Escenario: Fecha futura rechazada por el servidor
  Dado que la API recibe una fecha posterior a la fecha UTC de hoy
  Entonces responde 400 "RecetaMedica.FechaExpedicion"
  Y veo "La fecha de expedición no puede ser posterior a hoy."

Escenario: Repeticiones máximas
  Dado que dejo vacío o negativo el campo "Repeticiones Máximas"
  Cuando hago clic en "Guardar"
  Entonces no se envía nada y no veo ningún mensaje
  Y si no se informan al crear por API valen 0, y un valor negativo se rechaza con "Revisa los datos ingresados de la receta."

Escenario: Número de receta
  Dado que escribo un número de receta
  Entonces el campo no deja pasar de 30 caracteres y muestra "El número de receta admite hasta 30 caracteres"
  Y la ayuda dice "Necesario para vender receta retenida o estupefacientes."
  Y el número es opcional y no tiene unicidad: el mismo número puede repetirse en recetas distintas

Escenario: Indicaciones de uso
  Dado que escribo indicaciones de más de 1000 caracteres
  Cuando hago clic en "Guardar"
  Entonces la API responde 400 "CreateRecetaCommand.IndicacionesUso"
  Y veo "Revisa los datos ingresados de la receta."

Escenario: Paciente o médico que ya no existen
  Dado que el paciente o el médico fueron eliminados
  Cuando hago clic en "Guardar"
  Entonces la API responde 404 "Paciente.NotFound" o "ProfesionalSalud.NotFound"
  Y veo "El paciente seleccionado no existe. Vuelve a seleccionarlo." o "El médico seleccionado no existe. Vuelve a seleccionarlo."

Escenario: Editar una receta
  Dado que abrí una receta y tengo "Recetas Médicas › Actualizar"
  Cuando hago clic en "Editar", cambio el número de receta y hago clic en "Guardar"
  Entonces veo "Receta actualizada correctamente", vuelvo al modo ver con los datos recargados
  Y el paciente y el médico llegan precargados como ya elegidos
  Y la API solo comprueba la existencia del paciente o del médico si cambiaron

Escenario: La receta ya no existe
  Dado que fue eliminada
  Cuando intento guardarla
  Entonces la API responde 404 "RecetaMedica.NotFound" y veo "La receta ya no existe."
  Y al abrir su detalle veo "Error al cargar la receta"

Escenario: Otros errores al guardar
  Dado que falla el guardado por una causa sin mensaje propio
  Entonces veo "Error al crear la receta" al crear o "Error al actualizar la receta" al editar
  Y los errores de base de datos usan los textos comunes ("Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo." y similares)

Escenario: Identificador al editar (solo API)
  Dado que llamo a "PUT /api/v1/Recetas/{id}"
  Entonces el identificador se toma de la URL y uno vacío se rechaza con "UpdateRecetaCommand.Id"

Escenario: Cuántas veces se puede dispensar (se aplica al vender)
  Dado que registré la receta con "Repeticiones Máximas" N
  Entonces al vender, una receta de venta libre no tiene límite, una de receta simple admite 1 más N dispensaciones, y una de receta retenida o estupefaciente admite 1
  Y si ya se alcanzó el máximo, la venta se rechaza (código "Venta.Receta.Agotada", historias de la parte C)
```

> **Brecha conocida.** (1) En el detalle, a diferencia de la búsqueda, cambiar el texto de paciente o médico sin elegir otra opción no descarta la elección anterior: «Guardar» enviaría el paciente o médico elegido antes (por lectura del código; **No verificado en ejecución**). (2) Las repeticiones máximas no tienen `mat-error`. (3) Los comandos de receta no generan evento de auditoría. (4) La condición de venta del producto y los requisitos de receta al cobrar (receta registrada, número de receta, verificación manual) pertenecen a la parte C. El manual no recorrió esta pantalla; los mensajes salen de 19e §5.2.

### HU-INV-018 — Buscar y exportar recetas médicas

**Como** personal de atención **quiero** buscar recetas por paciente, médico y fechas **para** encontrar la receta que debo adjuntar a una venta.

| | |
|---|---|
| **Módulo** | Recetas médicas |
| **Pantalla** | `/recetas-medicas` — «Recetas Médicas» (tarjeta «Búsqueda de Recetas Médicas») |
| **Endpoint(s)** | `GET /api/v1/Recetas`, `POST /api/v1/Recetas/exportar` |
| **Permiso** | `Recetas:Leer` (y `Pacientes:Leer`, `ProfesionalesSalud:Leer` para las sugerencias) |
| **Reglas** | [RN-REC-001, RN-REC-016, RN-REC-017](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-GEN-008, RN-GEN-009](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-008, RN-IMP-009, RN-IMP-011](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Filtros disponibles
  Dado que abro "/recetas-medicas"
  Entonces veo "Paciente" ("Buscar paciente..."), "Médico" ("Buscar médico..."), "Fecha desde" y "Fecha hasta"
  Y veo los botones "Escanear receta" (con "Recetas Médicas › Analizar con IA") y "Nuevo" (con "Crear")

Escenario: Paciente y médico solo filtran si eliges una opción de la lista
  Dado que escribo "Rosa" en "Paciente" sin elegir una sugerencia
  Cuando hago clic en "Buscar"
  Entonces el filtro de paciente no se aplica, porque la búsqueda envía el identificador elegido y no el texto
  Y cada opción de la lista muestra el nombre y "— {documento}" (paciente) o "— {número de registro}" (médico)

Escenario: Buscar
  Dado que elegí un paciente de la lista y una fecha desde
  Cuando hago clic en "Buscar"
  Entonces la tabla muestra "Paciente" (enlace al detalle), "Médico", "Fecha de Expedición" (dd/MM/yyyy), "Repeticiones" e "Indicaciones" ("—" si no tiene)
  Y se ordena por fecha de expedición, la más reciente primero
  Y las fechas "desde" y "hasta" son inclusivas y no hay validación cruzada entre ellas
  Y el número de receta no se muestra en la tabla

Escenario: Paciente o médico eliminados
  Dado que el paciente o el médico de una receta fueron eliminados
  Entonces el nombre aparece vacío en la fila

Escenario: Sin coincidencias
  Cuando ninguna receta cumple
  Entonces la tabla muestra "Sin resultados"

Escenario: Exportar
  Cuando hago clic en "Exportar"
  Entonces se descarga "recetas-medicas.xlsx" con "Paciente", "Médico", "Fecha de Expedición", "Repeticiones" e "Indicaciones"
  Y no incluye el número de receta
  Y las fechas salen dd/MM/yyyy con el idioma español y MM/dd/yyyy con otro idioma
  Y si falla la búsqueda o la exportación veo "Error al buscar recetas médicas"

Escenario: Filtro de paciente con un identificador vacío (solo API)
  Dado que llamo a "GET /api/v1/Recetas" con un identificador de paciente que es un GUID vacío
  Entonces la API responde 400 "GetRecetasQuery.ClienteId"
```

> **Nota.** Enter no ejecuta la búsqueda en los filtros de paciente y médico. Un usuario con `Recetas:Leer` pero sin `Pacientes:Leer` o `ProfesionalesSalud:Leer` no recibe sugerencias (la API responde 403 y no hay manejador; **No verificado en ejecución**). **No verificado:** cómo interpreta el servidor las fechas enviadas como instante si el navegador está al este de UTC (en Perú, UTC-5, el día coincide).

### HU-INV-019 — Eliminar una receta médica

**Como** personal de atención **quiero** eliminar una receta registrada por error **para** no dejar datos incorrectos, salvo que ya se haya usado en una venta.

| | |
|---|---|
| **Módulo** | Recetas médicas |
| **Pantalla** | `/recetas-medicas/:id` (botón «Eliminar» al pie, solo en modo editar) |
| **Endpoint(s)** | `DELETE /api/v1/Recetas/{id}` |
| **Permiso** | `Recetas:Eliminar` |
| **Reglas** | [RN-REC-001, RN-REC-012, RN-REC-013, RN-REC-014](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-GEN-003](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Eliminar una receta no usada
  Dado que estoy editando una receta que ninguna venta usa y tengo "Recetas Médicas › Eliminar"
  Cuando hago clic en "Eliminar"
  Entonces se abre "Eliminar receta" con "¿Estás seguro de que deseas eliminar esta receta médica? Esta acción no se puede deshacer."
  Y al hacer clic en "Confirmar" veo "Receta eliminada correctamente" y vuelvo a "/recetas-medicas"
  Y la API responde 200 con el identificador de la receta

Escenario: Receta ya usada en una venta
  Dado que la receta figura en una línea de venta no eliminada
  Cuando confirmo la eliminación
  Entonces la API responde 409 "RecetaMedica.InUse"
  Y veo "No puedes eliminar esta receta porque ya se usó en ventas."
  Y la comprobación cuenta también las ventas anuladas

Escenario: Otro fallo
  Dado que la receta no existe
  Cuando confirmo
  Entonces la API responde 404 con el código genérico "NotFound"
  Y veo "Error al eliminar la receta"

Escenario: Sin permiso no hay botón
  Dado que mi rol no tiene "Recetas Médicas › Eliminar"
  Entonces no veo "Eliminar" aunque entre en modo editar
```

> **Brecha conocida.** No se revisan otras referencias (órdenes magistrales, procesamientos de IA): la eliminación es lógica y no dispara las claves foráneas. La pantalla muestra «ya se usó en ventas» ante cualquier 409.

### HU-INV-020 — Escanear una receta con IA

**Como** cajero **quiero** subir o fotografiar una receta y obtener la lista de medicamentos que contiene **para** saber rápido qué pide el cliente y si lo tengo.

| | |
|---|---|
| **Módulo** | Recetas médicas (análisis con IA) |
| **Pantalla** | `/recetas-medicas/escanear` — «Lector de Recetas IA» (botón «Escanear receta» de `/recetas-medicas`) |
| **Endpoint(s)** | `POST /api/v1/Recetas/analizar` (multipart, campo `imagen`) |
| **Permiso** | `Recetas:Analizar` (rotulado «Analizar con IA»); la ruta exige además `Recetas:Leer`; política de límite `AiPolicy` |
| **Reglas** | [RN-REC-001, RN-REC-018, RN-REC-019, RN-REC-020](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-GEN-028, RN-GEN-039](../../ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| **Estado** | Implementada (solo muestra el resultado; ver HU-INV-022) |

**Criterios de aceptación**

```gherkin
Escenario: Elegir la imagen
  Dado que tengo "Recetas Médicas › Analizar con IA" y estoy en "Lector de Recetas IA"
  Entonces veo la tarjeta "Escanear receta médica" con la insignia "Análisis con IA", el subtítulo "Toma una foto de la receta o selecciona una imagen de la galería" y la zona "Captura o selecciona una imagen para comenzar" y "o arrastra una imagen aquí"
  Cuando hago clic en "Tomar foto" (abre la cámara trasera en un teléfono y el selector de archivos en un equipo), en "Galería" o arrastro un archivo a la zona
  Entonces veo la vista previa de la imagen y su nombre de archivo
  Y solo se usa el primer archivo, y un archivo arrastrado que no sea de tipo imagen se ignora sin aviso

Escenario: Interpretar la receta
  Dado que elegí una imagen
  Cuando hago clic en "Interpretar receta"
  Entonces veo "Interpretando..." con el anillo giratorio y los botones "Tomar foto", "Galería" y "Limpiar" quedan bloqueados
  Y al terminar veo los resultados (HU-INV-021)

Escenario: Sin imagen no pasa nada
  Dado que no elegí ninguna imagen
  Entonces el botón "Interpretar receta" no se muestra
  Y si la API recibe la petición sin imagen responde 400 con el texto "Image is required."

Escenario: Limpiar
  Dado que hay una imagen elegida
  Cuando hago clic en "Limpiar"
  Entonces se quitan la imagen, la vista previa y los resultados
  Y al elegir otro archivo se limpian los resultados anteriores

Escenario: Sin el permiso de análisis
  Dado que mi rol no tiene "Recetas Médicas › Analizar con IA"
  Entonces no veo el botón "Escanear receta"
  Y si escribo "/recetas-medicas/escanear" veo "No tienes permisos para acceder a esta pantalla." y vuelvo al Inicio

Escenario: Límite de peticiones y tamaño (no verificado en la aplicación)
  Dado que mi empresa ya hizo 10 análisis en un minuto (con hasta 2 en cola)
  O que la imagen hace que el cuerpo supere 10 MB
  Cuando hago clic en "Interpretar receta"
  Entonces la API responde 429, o 413 "Request.TooLarge"
  Y veo "Error al interpretar la receta. Intenta nuevamente."

Escenario: El servidor no valida el tipo de archivo
  Dado que subo un archivo que no es una imagen real
  Entonces el servidor no valida su tipo ni su contenido: lo lee entero y lo envía a Gemini declarándolo siempre como "image/jpeg"
  Y el SPA solo acepta archivos de tipo imagen ("image/*")

Escenario: Especialidad de contexto (solo API)
  Dado que llamo a "POST /api/v1/Recetas/analizar" con "especialidadContexto"
  Entonces debe tener como máximo 50 caracteres y solo letras (incluidas tildes) y espacios, y si no se informa se usa "Medicina General"
  Y un valor no válido responde 400 "AnalizarRecetaQuery.EspecialidadContexto"
  Y la pantalla nunca envía este dato
```

> **Brecha conocida.** La pantalla muestra «Error al interpretar la receta. Intenta nuevamente.» ante **cualquier** fallo (límite de peticiones, archivo grande o inválido, fallo del servicio de IA o sin permiso): el usuario no sabe si debe esperar, cambiar la imagen o reintentar ([19e §5.3](../../ola-1-fundamentos/19-inventario-de-pantallas/19e-inventario-y-atencion.md)). El SPA no limita el tamaño del archivo. **No verificado en la aplicación:** el manual no recorrió el escáner (comportamiento real de Gemini y Presidio, formato de imagen admitido). El mismo servicio lo usa el punto de venta (parte C).

### HU-INV-021 — Interpretar el resultado del escaneo: DIGEMID, confianza, stock y equivalentes

**Como** cajero **quiero** ver cada medicamento detectado con su validación, su confianza, el stock de mi sucursal y sus equivalentes **para** ofrecer una alternativa si no tengo el prescrito.

| | |
|---|---|
| **Módulo** | Recetas médicas (análisis con IA) |
| **Pantalla** | `/recetas-medicas/escanear` — secciones «Medicamentos de la receta» y «Sugerencias IA» |
| **Endpoint(s)** | `POST /api/v1/Recetas/analizar` |
| **Permiso** | `Recetas:Analizar` |
| **Reglas** | [RN-REC-022, RN-REC-023, RN-REC-024, RN-REC-025, RN-REC-026, RN-REC-027, RN-REC-028](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-STK-017](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) |
| **Estado** | Implementada |

**Criterios de aceptación**

```gherkin
Escenario: Medicamento detectado y validado
  Dado que la IA detectó "Paracetamol 500 mg" y el catálogo DIGEMID tiene "PARACETAMOL 500 MG TABLETA (DEMO)"
  Entonces veo en "Medicamentos de la receta (N)" una tarjeta con el nombre oficial del medicamento de mi empresa que mejor coincide
  Y "Detectado como: "Paracetamol 500 mg"" si el texto detectado difiere del nombre oficial
  Y la insignia "DIGEMID" y la razón "Validado por DIGEMID (nombre)"
  Y "Cantidad sugerida: N" si la IA leyó una cantidad

Escenario: Cómo se valida contra DIGEMID
  Dado que se busca el nombre del medicamento en el catálogo DIGEMID local de mi empresa
  Entonces deben estar contenidas todas las palabras; si no hay resultado se reintenta con la primera palabra
  Y se ordenan hasta 50 candidatos por parecido de texto y se toman los 5 mejores
  Y si no hay ninguno, la razón es "Detectado en receta" y no aparece la insignia "DIGEMID"

Escenario: Coincidencia con el catálogo de mi empresa
  Dado que se busca el producto con el nombre y la concentración del catálogo DIGEMID (o con el texto detectado si no hubo coincidencia)
  Entonces se consideran hasta 50 medicamentos cuyo nombre contenga la primera palabra y se acepta el mejor solo si su parecido es mayor a 0,4
  Y si no hay coincidencia, el resultado no trae producto: aparece con el nombre DIGEMID o el texto detectado

Esquema del escenario: Rótulo de confianza
  Dado que la confianza final del medicamento es <confianza>
  Entonces la insignia dice <rótulo>

  Ejemplos:
    | confianza | rótulo              |
    | 0,90      | Confianza: Alta     |
    | 0,85      | Confianza: Alta     |
    | 0,70      | Confianza: Media    |
    | 0,65      | Confianza: Media    |
    | 0,50      | Confianza: Baja     |

Escenario: Cómo se calcula la confianza
  Dado que la IA dio un nivel de confianza
  Entonces la confianza final es nivel × 0,6 + parecido × 0,4, redondeada a 2 decimales
  Y "parecido" vale 0,9 si hubo coincidencia DIGEMID y 1,0 si no la hubo

Escenario: Stock en mi sucursal
  Dado que el producto tiene lotes no vencidos en mi sucursal
  Entonces veo "En stock: N unidades" (sin decimales)
  Pero si no hay existencias veo "Sin stock"
  Y si mi sesión no trae una sucursal válida el stock es 0
  Y a diferencia del stock vendible, no se descuentan los lotes retenidos en cuarentena

Escenario: Equivalentes terapéuticos
  Dado que el producto tiene formulación clínica con ingrediente, concentración y unidad
  Entonces veo "Equivalentes terapéuticos:" con una fila por cada otro medicamento de mi empresa con la misma formulación (nombre comercial, "— {laboratorio}" y su stock)
  Y si el prescrito no tiene stock pero algún equivalente sí, veo "No hay stock del medicamento prescrito, pero hay equivalentes disponibles."

Escenario: Sugerencias de la IA
  Dado que la IA propone medicamentos complementarios o genéricos
  Entonces cada sugerencia se busca en el catálogo de mi empresa y las que no coinciden se descartan
  Y las que coinciden aparecen en "Sugerencias IA (N)" con la razón "Sugerido por {medicamento}", su stock y sin insignias ni equivalentes

Escenario: Límites del resultado
  Dado que la IA devuelve más de 20 medicamentos o más de 3 sugerencias por medicamento
  Entonces solo se procesan los primeros 20 medicamentos y 3 sugerencias por cada uno
  Y el resto se descarta sin aviso

Escenario: Sin medicamentos detectados
  Dado que la IA no extrae texto o responde algo que no es el formato esperado
  Entonces veo "No se detectaron medicamentos en la imagen. Intenta con una foto más nítida."
```

> **Brecha conocida.** Con la misma confianza de la IA, un medicamento no encontrado en DIGEMID queda con confianza final **mayor** que uno validado, porque su «parecido» es 1,0 frente a 0,9 ([01c1 hallazgo 5](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md#63-hallazgos-del-código)). El texto «DIGEMID» de la insignia está escrito en el código y no en los archivos de textos. **No verificado en la aplicación:** calidad del resultado con Gemini y Presidio reales; los equivalentes no se muestran en el punto de venta (parte C). Un catálogo DIGEMID vacío no impide analizar, pero ningún medicamento saldrá validado (manual cap. 4, §8.1).

### HU-INV-022 — Proteger los datos del paciente y entender qué guarda el análisis

**Como** administrador **quiero** saber que el texto de la receta se anonimiza antes de llegar a la IA y que el análisis no guarda nada **para** cumplir con la privacidad de mis pacientes y registrar la receta yo mismo cuando haga falta.

| | |
|---|---|
| **Módulo** | Recetas médicas (análisis con IA) |
| **Pantalla** | `/recetas-medicas/escanear` |
| **Endpoint(s)** | `POST /api/v1/Recetas/analizar` |
| **Permiso** | `Recetas:Analizar` |
| **Reglas** | [RN-REC-021, RN-REC-029, RN-REC-030](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) |
| **Estado** | Parcial (el resultado no se guarda ni pasa al punto de venta desde esta pantalla; las tablas de recetas digitalizadas son solo esquema) |

**Criterios de aceptación**

```gherkin
Escenario: Las tres etapas del análisis
  Dado que interpreté una receta
  Entonces Gemini extrae el texto de la imagen (OCR) y, si no extrae nada, el resultado es una lista vacía
  Y Presidio anonimiza el texto
  Y Gemini interpreta solo el texto anonimizado y devuelve los medicamentos

Escenario: Si la anonimización falla, el análisis se aborta
  Dado que Presidio falla o devuelve vacío
  Entonces el análisis se aborta y el texto del paciente no llega a la IA
  Y veo "Error al interpretar la receta. Intenta nuevamente."

Escenario: Fallas del servicio de IA
  Dado que hay un fallo de red, falta la clave de la API de Gemini, o falla Gemini o Presidio
  Entonces la API responde 500 con una referencia de soporte ("traceId")
  Y veo "Error al interpretar la receta. Intenta nuevamente."

Escenario: Respuesta de la IA que no es el formato esperado
  Dado que Gemini responde algo que no es el JSON esperado
  Entonces el resultado es una lista vacía sin error
  Y veo "No se detectaron medicamentos en la imagen. Intenta con una foto más nítida."

Escenario: El análisis no guarda nada
  Dado que terminé de interpretar una receta
  Entonces no se crea ninguna receta, no se registra el procesamiento ni se genera evento de auditoría
  Y la pantalla no tiene botones para crear la receta ni para pasar los medicamentos a la venta
  Y para que la receta cuente en una venta debo registrarla aparte en "Recetas Médicas › Nuevo" (HU-INV-017)

Escenario: Las tablas de recetas digitalizadas no se usan
  Dado que las tablas de recetas digitalizadas por IA y de registro de privacidad existen en el esquema
  Entonces ningún código las escribe
  Y sus reglas de entidad (receta y ruta de imagen obligatorias, ruta de hasta 500 caracteres, confianza de 0 a 100) no se pueden alcanzar desde ninguna pantalla ni endpoint
```

> **Pendiente.** Persistir la imagen y el resultado, crear la receta desde el escaneo y enlazarla a la venta no existen hoy ([RN-REC-029, RN-REC-030](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md)). La imagen viaja a un servicio externo (Gemini); **No verificado** si algún registro de infraestructura la conserva ([proceso 7](../14-procesos-de-negocio.md), postcondiciones). El detalle técnico de la integración está en [07-integraciones](../../ola-1-fundamentos/07-integraciones.md).

---

## 18. Tabla de trazabilidad

Cada historia con sus reglas, pantallas, endpoints y la sección del manual de usuario donde se recorre. Todas las rutas de API llevan el prefijo `/api/v1/`. «Cap. 5 (sin escribir)» significa que el [capítulo 5](../manual-usuario/capitulo-05-productos-y-proveedores.md) lo anuncia en su introducción pero a la fecha su archivo solo desarrolla la sección de medicamentos (hasta el §4.5); «—» significa que el manual aún no tiene capítulo para ese módulo (los capítulos 1 a 5 son los publicados).

| Historia | Reglas | Pantallas | Endpoints | Capítulo del manual |
|---|---|---|---|---|
| HU-CAT-001 | RN-UDM-001 a 009 | `/unidades-medida/nueva`, `/unidades-medida/:id` | POST, PUT, GET `UnidadesMedida` | Cap. 4 §5.3 y §5.5 |
| HU-CAT-002 | RN-UDM-001, RN-UDM-012, RN-UDM-013 | `/unidades-medida` | GET, POST exportar `UnidadesMedida` | Cap. 4 §5.2 y §5.4 |
| HU-CAT-003 | RN-UDM-001, RN-UDM-007, RN-UDM-009 a 011 | `/unidades-medida/:id` | DELETE `UnidadesMedida/{id}` | Cap. 4 §5.6 |
| HU-CAT-004 | RN-UDM-002 a 006, RN-UDM-014 a 017 | `/unidades-medida/carga-masiva` | GET plantilla, POST previsualizar, POST carga-masiva `UnidadesMedida` | Cap. 4 §9.2 |
| HU-CAT-005 | RN-JER-001 a 010, RN-JER-012, RN-JER-013 | `/jerarquias/nueva`, `/jerarquias/:id` | POST, PUT, GET `JerarquiasUoM` | Cap. 5 (sin escribir) |
| HU-CAT-006 | RN-JER-001, RN-JER-015, RN-JER-016 | `/jerarquias` | GET, POST exportar `JerarquiasUoM` | Cap. 5 (sin escribir) |
| HU-CAT-007 | RN-JER-001, RN-JER-013, RN-JER-014, RN-PRE-007 | `/jerarquias/:id` | DELETE `JerarquiasUoM/{id}` | Cap. 5 (sin escribir) |
| HU-CAT-008 | RN-JER-002 a 005, RN-JER-008, RN-JER-009, RN-JER-017 a 022 | `/jerarquias/carga-masiva` | GET plantilla, POST previsualizar, POST carga-masiva `JerarquiasUoM` | Cap. 5 (sin escribir) |
| HU-CAT-009 | RN-JER-010, RN-JER-011 | Sin pantalla propia (alimenta `/presentaciones-venta/nueva` y el diálogo de presentaciones) | GET `JerarquiasUoM/unidades-vendibles` | Cap. 5 (sin escribir) |
| HU-CAT-010 | RN-LAB-001 a 009, RN-LAB-011 | `/laboratorios/nueva`, `/laboratorios/:id` | POST, PUT, GET `Laboratorios` | Cap. 4 §6.3 y §6.5 |
| HU-CAT-011 | RN-LAB-001, RN-LAB-012 | `/laboratorios` | GET, POST exportar `Laboratorios` | Cap. 4 §6.2 y §6.4 |
| HU-CAT-012 | RN-LAB-001, RN-LAB-009 a 011 | `/laboratorios/:id` | DELETE `Laboratorios/{id}` | Cap. 4 §6.5 |
| HU-CAT-013 | RN-LAB-002 a 004, RN-LAB-006, RN-LAB-007, RN-LAB-013 a 017 | `/laboratorios/carga-masiva` | GET plantilla, POST previsualizar, POST carga-masiva `Laboratorios` | Cap. 4 §9.3 |
| HU-CAT-014 | RN-ING-001 a 008 | `/ingredientes-activos/nueva`, `/ingredientes-activos/:id` | POST, PUT, GET `IngredientesActivos` | Cap. 4 §7.3 y §7.5 |
| HU-CAT-015 | RN-ING-001, RN-ING-010 | `/ingredientes-activos` | GET, POST exportar `IngredientesActivos` | Cap. 4 §7.2 y §7.4 |
| HU-CAT-016 | RN-ING-001, RN-ING-008, RN-ING-009 | `/ingredientes-activos/:id` | DELETE `IngredientesActivos/{id}` | Cap. 4 §7.5 |
| HU-CAT-017 | RN-ING-002 a 006, RN-ING-011 a 015 | `/ingredientes-activos/carga-masiva` | GET plantilla, POST previsualizar, POST carga-masiva `IngredientesActivos` | Cap. 4 §9.4 |
| HU-CAT-018 | RN-MED-001 a 012, RN-MED-019 | `/medicamentos/nueva`, `/medicamentos/:id` | POST, PUT, GET `Medicamentos`; GET `Medicamentos/condiciones-venta` | Cap. 5 §4.2 y §4.4 |
| HU-CAT-019 | RN-MED-001, RN-MED-014, RN-MED-019, RN-MED-020 | `/medicamentos` | GET, POST exportar `Medicamentos`; GET `Medicamentos/condiciones-venta` | Cap. 5 §4.3 y §4.5 |
| HU-CAT-020 | RN-MED-012, RN-MED-018, RN-STK-017 | `/medicamentos/:id` (tarjeta de stock) | GET `Medicamentos/{id}` | Cap. 5 (sin escribir) |
| HU-CAT-021 | RN-MED-001, RN-MED-012, RN-MED-013 | `/medicamentos/:id` | DELETE `Medicamentos/{id}` | Cap. 5 (sin escribir) |
| HU-CAT-022 | RN-MED-002 a 004, RN-MED-007, RN-MED-010, RN-MED-021 a 026 | `/medicamentos/carga-masiva` | GET plantilla, POST previsualizar, POST carga-masiva `Medicamentos` | Cap. 5 (sin escribir) |
| HU-CAT-023 | RN-MED-015 a 017, RN-STK-017 | Sin pantalla en el catálogo (punto de venta, parte C) | GET `Medicamentos` con `IncluirStock`, `SoloStockVendibleEnMiSucursal`, `OrdenarPorMasVendidos` | — |
| HU-CAT-024 | RN-PRE-001 a 011, RN-PRE-015, RN-JER-011 | `/presentaciones-venta/nueva`, `/presentaciones-venta/:id` | POST, PUT, GET `PresentacionesVenta`; GET `JerarquiasUoM/unidades-vendibles` | Cap. 5 (sin escribir) |
| HU-CAT-025 | RN-PRE-001, RN-PRE-005, RN-PRE-006, RN-PRE-009, RN-PRE-012, RN-PRE-015 | Diálogo «Presentaciones de Venta» en `/medicamentos/:id` | GET `JerarquiasUoM/unidades-vendibles`; GET, POST, PUT, DELETE `PresentacionesVenta` | Cap. 5 (sin escribir) |
| HU-CAT-026 | RN-PRE-001, RN-PRE-014 | `/presentaciones-venta` | GET, POST exportar `PresentacionesVenta` | Cap. 5 (sin escribir) |
| HU-CAT-027 | RN-PRE-001, RN-PRE-011 a 013 | `/presentaciones-venta/:id` | DELETE `PresentacionesVenta/{id}` | Cap. 5 (sin escribir) |
| HU-CAT-028 | RN-FOR-001 a 010 | `/formulaciones-clinicas/nueva`, `/formulaciones-clinicas/:id` | POST, PUT, GET `FormulacionesClinicas` | Cap. 5 (sin escribir) |
| HU-CAT-029 | RN-FOR-001, RN-FOR-012 | `/formulaciones-clinicas` | GET, POST exportar `FormulacionesClinicas` | Cap. 5 (sin escribir) |
| HU-CAT-030 | RN-FOR-001, RN-FOR-010, RN-FOR-011 | `/formulaciones-clinicas/:id` | DELETE `FormulacionesClinicas/{id}` | Cap. 5 (sin escribir) |
| HU-CAT-031 | RN-FOR-013 | Ninguna (solo API) | GET `FormulacionesClinicas/medicamento/{productoId}` | — |
| HU-CAT-032 | RN-PRV-001 a 009, RN-PRV-011 | `/proveedores/nueva`, `/proveedores/:id` | POST, PUT, GET `Proveedores` | Cap. 5 (sin escribir) |
| HU-CAT-033 | RN-PRV-001, RN-PRV-012 | `/proveedores` | GET, POST exportar `Proveedores` | Cap. 5 (sin escribir) |
| HU-CAT-034 | RN-PRV-001, RN-PRV-009, RN-PRV-010 | `/proveedores/:id` | DELETE `Proveedores/{id}` | Cap. 5 (sin escribir) |
| HU-CAT-035 | RN-PRV-002 a 008, RN-PRV-013 a 017 | `/proveedores/carga-masiva` | GET plantilla, POST previsualizar, POST carga-masiva `Proveedores` | Cap. 5 (sin escribir) |
| HU-CAT-036 | RN-PRV-001, RN-PRV-018 a 024, RN-PRV-027, RN-PRV-028 | Diálogo «Registrar precio de compra» en `/proveedores/:id` | POST `Proveedores/precios`; GET `Medicamentos` | Cap. 5 (sin escribir) |
| HU-CAT-037 | RN-PRV-025, RN-PRV-026 | `/proveedores/:id` (tarjeta «Precios de compra») | GET `Proveedores/{id}/precios` | Cap. 5 (sin escribir) |
| HU-CAT-038 | RN-ASE-001 a 005, RN-ASE-007 | `/aseguradoras/nueva`, `/aseguradoras/:id` | POST, PUT, GET `Seguros` | Cap. 5 (sin escribir) |
| HU-CAT-039 | RN-ASE-001, RN-ASE-008 | `/aseguradoras` | GET, POST exportar `Seguros` | Cap. 5 (sin escribir) |
| HU-CAT-040 | RN-ASE-001, RN-ASE-005, RN-ASE-006 | `/aseguradoras/:id` | DELETE `Seguros/{id}` | Cap. 5 (sin escribir) |
| HU-CAT-041 | RN-ASE-002 a 004, RN-ASE-009 a 012 | `/aseguradoras/carga-masiva` | GET plantilla, POST previsualizar, POST carga-masiva `Seguros` | Cap. 5 (sin escribir) |
| HU-CAT-042 | RN-DIG-001, RN-DIG-007, RN-DIG-009 | `/digemid-catalogo`, `/digemid-catalogo/:id` | GET `DIGEMID/catalogo`, GET `DIGEMID/catalogo/{id}`, POST `DIGEMID/catalogo/exportar` | Cap. 4 §8.3 y §8.4 |
| HU-CAT-043 | RN-DIG-001 a 006, RN-DIG-010 a 013 | `/digemid-catalogo/carga-masiva` | GET plantilla, POST previsualizar, POST carga-masiva `DIGEMID/catalogo` | Cap. 4 §8.5 |
| HU-CAT-044 | RN-DIG-001 a 008 | Ninguna (solo API) | POST, PUT, DELETE `DIGEMID/catalogo` | Cap. 4 §8.2 (indica no usarlas) |
| HU-INV-001 | RN-LOT-001 a 013, RN-LOT-016 | `/lotes/nueva`, `/lotes/:id` | POST, PUT, GET `LotesInventario` | — |
| HU-INV-002 | RN-LOT-001, RN-LOT-013, RN-LOT-017, RN-LOT-019 | `/lotes`, `/lotes/:id` | GET, GET `{id}`, POST exportar `LotesInventario` | — |
| HU-INV-003 | RN-LOT-001, RN-LOT-013 a 015 | `/lotes/:id` | DELETE `LotesInventario/{id}` | — |
| HU-INV-004 | RN-LOT-018 | Ninguna (solo API) | GET `LotesInventario/producto/{productoId}` | — |
| HU-INV-005 | RN-STK-001 a 011 | `/stock-por-sucursal/nueva` | POST `LotesInventario/stock`; GET `Sucursales/permitidas` | — |
| HU-INV-006 | RN-STK-001, RN-STK-004, RN-STK-006, RN-STK-011 a 013 | `/stock-por-sucursal/:id` | GET `StockPorSucursal/{id}`, PUT `StockPorSucursal/{id}/ajustar` | — |
| HU-INV-007 | RN-STK-001, RN-STK-014 a 017 | `/stock-por-sucursal` | GET, POST exportar `StockPorSucursal`; GET `Sucursales/permitidas` | — |
| HU-INV-008 | RN-STK-018 | Ninguna (solo API) | GET `LotesInventario/stock/medicamento/{medicamentoId}` | — |
| HU-INV-009 | RN-PAC-001 a 009 | `/pacientes/nueva`, `/pacientes/:id` | POST, PUT, GET `Pacientes` | — |
| HU-INV-010 | RN-PAC-001, RN-PAC-011 | `/pacientes` | GET, POST exportar `Pacientes` | — |
| HU-INV-011 | RN-PAC-001, RN-PAC-009, RN-PAC-010 | `/pacientes/:id` | DELETE `Pacientes/{id}` | — |
| HU-INV-012 | RN-PAC-002 a 008, RN-PAC-012 a 017 | `/pacientes/carga-masiva` | GET plantilla, POST previsualizar, POST carga-masiva `Pacientes` | — |
| HU-INV-013 | RN-PRO-001 a 006 | `/profesionales-salud/nueva`, `/profesionales-salud/:id` | POST, PUT, GET `profesionalessalud` | — |
| HU-INV-014 | RN-PRO-001, RN-PRO-008 | `/profesionales-salud` | GET, POST exportar `profesionalessalud` | — |
| HU-INV-015 | RN-PRO-001, RN-PRO-006, RN-PRO-007 | `/profesionales-salud/:id` | DELETE `profesionalessalud/{id}` | — |
| HU-INV-016 | RN-PRO-002 a 005, RN-PRO-009 a 012 | `/profesionales-salud/carga-masiva` | GET plantilla, POST previsualizar, POST carga-masiva `profesionalessalud` | — |
| HU-INV-017 | RN-REC-001 a 012, RN-REC-015, RN-REC-031 | `/recetas-medicas/nueva`, `/recetas-medicas/:id` | POST, PUT, GET `Recetas` | — |
| HU-INV-018 | RN-REC-001, RN-REC-016, RN-REC-017 | `/recetas-medicas` | GET, POST exportar `Recetas` | — |
| HU-INV-019 | RN-REC-001, RN-REC-012 a 014 | `/recetas-medicas/:id` | DELETE `Recetas/{id}` | — |
| HU-INV-020 | RN-REC-001, RN-REC-018 a 020 | `/recetas-medicas/escanear` | POST `Recetas/analizar` | — |
| HU-INV-021 | RN-REC-022 a 028, RN-STK-017 | `/recetas-medicas/escanear` | POST `Recetas/analizar` | — |
| HU-INV-022 | RN-REC-021, RN-REC-029, RN-REC-030 | `/recetas-medicas/escanear` | POST `Recetas/analizar` | — |

---

## 19. Pendientes y «No verificado»

### 19.1 En curso (sin commitear)

Ninguno. `git status --short` no muestra cambios locales en `sofia-apis` (`43f5529`) ni en `sofia-web-spa` (`eeffbb6`) al cerrar este documento.

### 19.2 Pendientes de producto y brechas del sistema que las historias describen

| # | Hallazgo | Historias |
|---|---|---|
| 1 | Mensajes genéricos: la mayoría de los rechazos del servidor (validación, `*.NotFound`, texto de solo espacios) se ven como el aviso genérico de guardado de cada pantalla. | Casi todas las de registrar y editar |
| 2 | Aseguradoras: ningún error se traduce; un código repetido se ve como «Error al guardar la aseguradora»; la carga masiva no controla duplicados y un solo repetido hace fallar todo el guardado; el mensaje de longitud de la vista previa dice 200 en vez de 150. | HU-CAT-038, HU-CAT-041 |
| 3 | Vista previa que no anticipa lo que el guardado descarta sin avisar: conversiones repetidas o contradictorias, código de laboratorio, DCI de ingrediente y razón social de proveedor; el aviso de éxito puede decir «Se guardaron 0 registros». | HU-CAT-008, HU-CAT-013, HU-CAT-017, HU-CAT-035 |
| 4 | Ingredientes: la vista previa valida el código ATC (que a mano sí puede repetirse) y no la DCI (la clave real). | HU-CAT-017 |
| 5 | DIGEMID: la vista previa no valida longitudes y el guardado rechaza todo el archivo; no hay pantalla para crear, editar o eliminar un producto. | HU-CAT-043, HU-CAT-044 |
| 6 | Medicamentos cargados en masa quedan sin precio de venta base. | HU-CAT-022 |
| 7 | Eliminaciones sin validación de dependencias: jerarquías, formulaciones, aseguradoras y productos DIGEMID. La baja de una unidad no revisa presentaciones de venta; un lote usado en transferencias, cuarentena, inmunizaciones u órdenes magistrales se puede eliminar; una receta cuenta también las ventas anuladas. | HU-CAT-003, HU-CAT-007, HU-CAT-030, HU-CAT-040, HU-CAT-044, HU-INV-003, HU-INV-019 |
| 8 | Las pantallas tratan cualquier HTTP 409 al eliminar (o al guardar, en lotes, presentaciones y formulaciones) como el caso concreto («en uso», «duplicado»); un conflicto de concurrencia o de referencia mostraría ese texto. | HU-CAT-003, HU-CAT-012, HU-CAT-016, HU-CAT-021, HU-CAT-024, HU-CAT-027, HU-CAT-028, HU-INV-001, HU-INV-003 |
| 9 | Diferencias de longitud entre formulario y API: proveedor (30/50 y 500/100), paciente (20/50 y 150/100), profesional (200/150 y 300/255). | HU-CAT-032, HU-INV-009, HU-INV-013 |
| 10 | Lotes: el aviso de lote duplicado casi nunca aparece; no se puede editar un lote vencido; fabricación contra caducidad solo se valida al crear; sin indicador de lote vencido en la tabla. | HU-INV-001, HU-INV-002 |
| 11 | Stock: guardar el ingreso sin elegir lote no muestra nada; los textos de sucursal y lote obligatorios no se pintan; se puede ingresar stock de un lote vencido o retenido; ajustar una sucursal no permitida responde 403 y consultarla 404. | HU-INV-005, HU-INV-006 |
| 12 | Precios de proveedor: un precio registrado no se puede editar ni eliminar (y por eso el proveedor no se puede eliminar); no se puede informar plazo de entrega ni cantidad mínima; la regla de un solo precio abierto no tiene índice único. | HU-CAT-034, HU-CAT-036 |
| 13 | Presentaciones: «Exportar» ignora el producto elegido en la lista y borrar el texto no descarta el producto elegido. | HU-CAT-026 |
| 14 | Pacientes: la carga masiva declara la clave `fields.tipoDocumento`, que no existe en la raíz de `es.json` (verificado). | HU-INV-012 |
| 15 | Recetas: la receta no tiene líneas de medicamentos ni vigencia; el escáner no guarda nada, no crea la receta ni pasa al punto de venta; un único mensaje de error para todo fallo; las tablas de recetas digitalizadas son solo esquema; con la misma confianza de la IA un medicamento no validado por DIGEMID queda con confianza mayor que uno validado. | HU-INV-017, HU-INV-020, HU-INV-021, HU-INV-022 |
| 16 | Solo API (sin pantalla): unidades vendibles de un producto (la usa Presentaciones), formulaciones por medicamento, lotes por producto, stock por medicamento, crear/editar/eliminar un producto DIGEMID, especialidad de contexto del análisis y las opciones de listado de medicamentos del punto de venta. | HU-CAT-009, HU-CAT-023, HU-CAT-031, HU-CAT-044, HU-INV-004, HU-INV-008, HU-INV-020 |
| 17 | Código HTTP de éxito no uniforme: crear proveedor, aseguradora, paciente, receta e ingresar stock responden 200 (el resto 201); eliminar proveedor, aseguradora, paciente y receta responden 200 con el identificador (el resto 204). | HU-CAT-032, HU-CAT-034, HU-CAT-038, HU-CAT-040, HU-INV-005, HU-INV-009, HU-INV-011, HU-INV-017, HU-INV-019 |

### 19.3 No verificado

| # | Qué | Por qué |
|---|---|---|
| 1 | Los avisos de base de datos («Ya existe un registro con ese mismo valor…», «El registro hace referencia a datos que ya no existen…», «Uno de los valores está fuera del rango permitido.»). | No se pudieron provocar; vienen de 19c §0 y del mapa común de errores. |
| 2 | El bloqueo por uso al eliminar unidades, laboratorios e ingredientes, y todo el eliminar de medicamentos, presentaciones, proveedores, pacientes, profesionales, lotes y recetas. | El capítulo 4 lo dejó pendiente por falta de datos y el capítulo 5 no cubre esas secciones; vienen de 01b, 01c1 y 19c a 19e. |
| 3 | El contenido de los archivos de **Exportar** y de las plantillas de carga masiva. | El navegador integrado no muestra descargas; solo se comprobó que el servidor responde 200 (manual cap. 4 §12). |
| 4 | Las pantallas de jerarquías, presentaciones, formulaciones, proveedores (incluido el diálogo de precios), aseguradoras, lotes, stock, pacientes, profesionales, recetas y escáner se documentan desde 01b, 01c1 y 19c a 19e; ninguna se recorrió en la aplicación para este documento. | El manual publicado (capítulos 4 y 5) solo recorrió unidades, laboratorios, ingredientes, DIGEMID y la sección de medicamentos hasta el §4.5. |
| 5 | Si la comparación de códigos y nombres únicos al crear y editar distingue mayúsculas en todas las bases (el manual lo comprobó en unidades, laboratorios, ingredientes y medicamentos). | La intercalación no está definida en los scripts DbUp ([01b §16.1](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md)). |
| 6 | Cómo trata la carga masiva un número con coma (`1,5`). | No se ejecutó ([01b §16.1](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), No verificado 2). |
| 7 | Respuesta de la API ante una condición de venta inválida, un precio omitido o un texto más largo que la columna en DIGEMID. | Ocurre antes del validador o en el error de SQL Server; no se llamó a la API. |
| 8 | La selección con teclado (flechas y Enter) en los autocompletados, y lo que ve un usuario sin el permiso `Leer` del módulo consultado por un autocompletado. | Depende de Angular Material y las suscripciones no tienen manejador de error ([19d §7](../../ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md), [19e §7](../../ola-1-fundamentos/19-inventario-de-pantallas/19e-inventario-y-atencion.md)). |
| 9 | Stock: la excepción al guardar sin lote y los textos de sucursal y lote obligatorios que no se pintan. | Se deducen de leer `saveNew()` y el código de Angular Material. |
| 10 | Recetas: que el paciente o el médico anteriores sigan enviándose si cambias el texto sin elegir otra opción; la interpretación de las fechas de búsqueda según la zona horaria. | Se deduce de leer el código; no se probó. |
| 11 | Escáner de recetas: la calidad del análisis con Gemini y Presidio reales, el formato de imagen admitido y el límite de 10 peticiones por minuto (HTTP 429). | Requiere servicios externos; el encargo es de solo lectura. |
| 12 | Concurrencia: si dos registros simultáneos de precio del mismo proveedor y producto pueden dejar dos precios abiertos. | No se ejecutó una prueba de concurrencia ([01b §16.1](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md), No verificado 9). |
| 13 | Una cantidad de stock que desborde `DECIMAL(12,4)`. | El manejador global solo traduce los errores SQL 2601, 2627 y 547. |
| 14 | Si modificar o eliminar una jerarquía recalcula las presentaciones ya guardadas. | Por lo leído no se recalcula ([RN-PRE-007](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md)); no se revisaron los handlers de jerarquías con ese fin. |
| 15 | El stock del detalle del medicamento (si excluye lotes vencidos o sucursales no permitidas) y las opciones de listado para el punto de venta. | La regla solo dice «suma de la cantidad física de sus lotes»; no se revisó el código de la consulta. |
| 16 | La redirección al Inicio con «No tienes permisos para acceder a esta pantalla.» al escribir a mano la dirección de una pantalla sin permiso. | El navegador integrado impidió probar la navegación directa con el cajero (manual cap. 4 §10 y §12); viene de 19c §0. |

---

## 20. Tabla de cobertura

### 20.1 Reglas de negocio de mi alcance

Alcance: las reglas `UDM`, `JER`, `LAB`, `ING`, `MED`, `PRE`, `FOR`, `PRV`, `ASE`, `DIG`, `PAC` y `PRO` de [01b-catalogos.md](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) (versión 1.1) y `LOT`, `STK` y `REC` de [01c1-inventario-y-recetas.md](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md): **275 reglas**. Una verificación por script de lectura confirmó que (1) todos los IDs `RN-…` citados en este documento existen en 01a, 01b, 01c1 o 01c2 (`RN-VEN-019` está en 01c2); (2) ninguna de las 275 reglas queda sin historia; y (3) cada regla aparece en la fila «Reglas» de al menos una historia. Las reglas que terminan en `-001` (permisos) se cubren en cada historia de su módulo; las reglas `GEN` e `IMP` que se citan pertenecen a la parte A y no se cuentan aquí. Los rangos «a» se leen como todos los números entre los dos extremos, ambos incluidos.

| Regla | Historias |
|---|---|
| RN-UDM-001 | HU-CAT-001, HU-CAT-002, HU-CAT-003 |
| RN-UDM-002 a 006 | HU-CAT-001, HU-CAT-004 |
| RN-UDM-007 | HU-CAT-001, HU-CAT-003 |
| RN-UDM-008 | HU-CAT-001 |
| RN-UDM-009 | HU-CAT-001, HU-CAT-003 |
| RN-UDM-010, RN-UDM-011 | HU-CAT-003 |
| RN-UDM-012, RN-UDM-013 | HU-CAT-002 |
| RN-UDM-014 a 017 | HU-CAT-004 |
| RN-JER-001 | HU-CAT-005, HU-CAT-006, HU-CAT-007 |
| RN-JER-002 a 005 | HU-CAT-005, HU-CAT-008 |
| RN-JER-006, RN-JER-007 | HU-CAT-005 |
| RN-JER-008, RN-JER-009 | HU-CAT-005, HU-CAT-008 |
| RN-JER-010 | HU-CAT-005, HU-CAT-009 |
| RN-JER-011 | HU-CAT-009, HU-CAT-024 |
| RN-JER-012 | HU-CAT-005 |
| RN-JER-013 | HU-CAT-005, HU-CAT-007 |
| RN-JER-014 | HU-CAT-007 |
| RN-JER-015, RN-JER-016 | HU-CAT-006 |
| RN-JER-017 a 022 | HU-CAT-008 |
| RN-LAB-001 | HU-CAT-010, HU-CAT-011, HU-CAT-012 |
| RN-LAB-002 a 004 | HU-CAT-010, HU-CAT-013 |
| RN-LAB-005 | HU-CAT-010 |
| RN-LAB-006, RN-LAB-007 | HU-CAT-010, HU-CAT-013 |
| RN-LAB-008 | HU-CAT-010 |
| RN-LAB-009 | HU-CAT-010, HU-CAT-012 |
| RN-LAB-010 | HU-CAT-012 |
| RN-LAB-011 | HU-CAT-010, HU-CAT-012 |
| RN-LAB-012 | HU-CAT-011 |
| RN-LAB-013 a 017 | HU-CAT-013 |
| RN-ING-001 | HU-CAT-014, HU-CAT-015, HU-CAT-016 |
| RN-ING-002 a 006 | HU-CAT-014, HU-CAT-017 |
| RN-ING-007 | HU-CAT-014 |
| RN-ING-008 | HU-CAT-014, HU-CAT-016 |
| RN-ING-009 | HU-CAT-016 |
| RN-ING-010 | HU-CAT-015 |
| RN-ING-011 a 015 | HU-CAT-017 |
| RN-MED-001 | HU-CAT-018, HU-CAT-019, HU-CAT-021 |
| RN-MED-002 a 004 | HU-CAT-018, HU-CAT-022 |
| RN-MED-005, RN-MED-006 | HU-CAT-018 |
| RN-MED-007 | HU-CAT-018, HU-CAT-022 |
| RN-MED-008, RN-MED-009 | HU-CAT-018 |
| RN-MED-010 | HU-CAT-018, HU-CAT-022 |
| RN-MED-011 | HU-CAT-018 |
| RN-MED-012 | HU-CAT-018, HU-CAT-020, HU-CAT-021 |
| RN-MED-013 | HU-CAT-021 |
| RN-MED-014 | HU-CAT-019 |
| RN-MED-015 a 017 | HU-CAT-023 |
| RN-MED-018 | HU-CAT-020 |
| RN-MED-019 | HU-CAT-018, HU-CAT-019 |
| RN-MED-020 | HU-CAT-019 |
| RN-MED-021 a 026 | HU-CAT-022 |
| RN-PRE-001 | HU-CAT-024, HU-CAT-025, HU-CAT-026, HU-CAT-027 |
| RN-PRE-002 a 004 | HU-CAT-024 |
| RN-PRE-005, RN-PRE-006 | HU-CAT-024, HU-CAT-025 |
| RN-PRE-007 | HU-CAT-007, HU-CAT-024 |
| RN-PRE-008 | HU-CAT-024 |
| RN-PRE-009 | HU-CAT-024, HU-CAT-025 |
| RN-PRE-010 | HU-CAT-024 |
| RN-PRE-011 | HU-CAT-024, HU-CAT-027 |
| RN-PRE-012 | HU-CAT-025, HU-CAT-027 |
| RN-PRE-013 | HU-CAT-027 |
| RN-PRE-014 | HU-CAT-026 |
| RN-PRE-015 | HU-CAT-024, HU-CAT-025 |
| RN-FOR-001 | HU-CAT-028, HU-CAT-029, HU-CAT-030 |
| RN-FOR-002 a 009 | HU-CAT-028 |
| RN-FOR-010 | HU-CAT-028, HU-CAT-030 |
| RN-FOR-011 | HU-CAT-030 |
| RN-FOR-012 | HU-CAT-029 |
| RN-FOR-013 | HU-CAT-031 |
| RN-PRV-001 | HU-CAT-032, HU-CAT-033, HU-CAT-034, HU-CAT-036 |
| RN-PRV-002 a 008 | HU-CAT-032, HU-CAT-035 |
| RN-PRV-009 | HU-CAT-032, HU-CAT-034 |
| RN-PRV-010 | HU-CAT-034 |
| RN-PRV-011 | HU-CAT-032 |
| RN-PRV-012 | HU-CAT-033 |
| RN-PRV-013 a 017 | HU-CAT-035 |
| RN-PRV-018 a 024 | HU-CAT-036 |
| RN-PRV-025, RN-PRV-026 | HU-CAT-037 |
| RN-PRV-027, RN-PRV-028 | HU-CAT-036 |
| RN-ASE-001 | HU-CAT-038, HU-CAT-039, HU-CAT-040 |
| RN-ASE-002 a 004 | HU-CAT-038, HU-CAT-041 |
| RN-ASE-005 | HU-CAT-038, HU-CAT-040 |
| RN-ASE-006 | HU-CAT-040 |
| RN-ASE-007 | HU-CAT-038 |
| RN-ASE-008 | HU-CAT-039 |
| RN-ASE-009 a 012 | HU-CAT-041 |
| RN-DIG-001 | HU-CAT-042, HU-CAT-043, HU-CAT-044 |
| RN-DIG-002 a 006 | HU-CAT-043, HU-CAT-044 |
| RN-DIG-007 | HU-CAT-042, HU-CAT-044 |
| RN-DIG-008 | HU-CAT-044 |
| RN-DIG-009 | HU-CAT-042 |
| RN-DIG-010 a 013 | HU-CAT-043 |
| RN-PAC-001 | HU-INV-009, HU-INV-010, HU-INV-011 |
| RN-PAC-002 a 008 | HU-INV-009, HU-INV-012 |
| RN-PAC-009 | HU-INV-009, HU-INV-011 |
| RN-PAC-010 | HU-INV-011 |
| RN-PAC-011 | HU-INV-010 |
| RN-PAC-012 a 017 | HU-INV-012 |
| RN-PRO-001 | HU-INV-013, HU-INV-014, HU-INV-015 |
| RN-PRO-002 a 005 | HU-INV-013, HU-INV-016 |
| RN-PRO-006 | HU-INV-013, HU-INV-015 |
| RN-PRO-007 | HU-INV-015 |
| RN-PRO-008 | HU-INV-014 |
| RN-PRO-009 a 012 | HU-INV-016 |
| RN-LOT-001 | HU-INV-001, HU-INV-002, HU-INV-003 |
| RN-LOT-002 a 012 | HU-INV-001 |
| RN-LOT-013 | HU-INV-001, HU-INV-002, HU-INV-003 |
| RN-LOT-014, RN-LOT-015 | HU-INV-003 |
| RN-LOT-016 | HU-INV-001 |
| RN-LOT-017 | HU-INV-002 |
| RN-LOT-018 | HU-INV-004 |
| RN-LOT-019 | HU-INV-002 |
| RN-STK-001 | HU-INV-005, HU-INV-006, HU-INV-007 |
| RN-STK-002, RN-STK-003 | HU-INV-005 |
| RN-STK-004 | HU-INV-005, HU-INV-006 |
| RN-STK-005 | HU-INV-005 |
| RN-STK-006 | HU-INV-005, HU-INV-006 |
| RN-STK-007 a 010 | HU-INV-005 |
| RN-STK-011 | HU-INV-005, HU-INV-006 |
| RN-STK-012, RN-STK-013 | HU-INV-006 |
| RN-STK-014 a 016 | HU-INV-007 |
| RN-STK-017 | HU-CAT-020, HU-CAT-023, HU-INV-007, HU-INV-021 |
| RN-STK-018 | HU-INV-008 |
| RN-REC-001 | HU-INV-017, HU-INV-018, HU-INV-019, HU-INV-020 |
| RN-REC-002 a 011 | HU-INV-017 |
| RN-REC-012 | HU-INV-017, HU-INV-019 |
| RN-REC-013, RN-REC-014 | HU-INV-019 |
| RN-REC-015 | HU-INV-017 |
| RN-REC-016, RN-REC-017 | HU-INV-018 |
| RN-REC-018 a 020 | HU-INV-020 |
| RN-REC-021 | HU-INV-022 |
| RN-REC-022 a 028 | HU-INV-021 |
| RN-REC-029, RN-REC-030 | HU-INV-022 |
| RN-REC-031 | HU-INV-017 (la regla se aplica al vender; se describe como escenario y se remite a la parte C) |

### 20.2 Rutas del SPA de mi alcance

55 rutas enrutadas de [`rutas-spa.csv`](../../_inventario/rutas-spa.csv) bajo los módulos de esta parte (aseguradoras, DIGEMID, formulaciones, ingredientes, jerarquías, laboratorios, lotes, medicamentos, pacientes, presentaciones, profesionales, proveedores, recetas, stock y unidades). Ninguna queda sin historia.

| Ruta del SPA | Historias |
|---|---|
| `/unidades-medida` | HU-CAT-002 |
| `/unidades-medida/nueva` | HU-CAT-001 |
| `/unidades-medida/carga-masiva` | HU-CAT-004 |
| `/unidades-medida/:id` | HU-CAT-001, HU-CAT-003 |
| `/jerarquias` | HU-CAT-006 |
| `/jerarquias/nueva` | HU-CAT-005 |
| `/jerarquias/carga-masiva` | HU-CAT-008 |
| `/jerarquias/:id` | HU-CAT-005, HU-CAT-007 |
| `/laboratorios` | HU-CAT-011 |
| `/laboratorios/nueva` | HU-CAT-010 |
| `/laboratorios/carga-masiva` | HU-CAT-013 |
| `/laboratorios/:id` | HU-CAT-010, HU-CAT-012 |
| `/ingredientes-activos` | HU-CAT-015 |
| `/ingredientes-activos/nueva` | HU-CAT-014 |
| `/ingredientes-activos/carga-masiva` | HU-CAT-017 |
| `/ingredientes-activos/:id` | HU-CAT-014, HU-CAT-016 |
| `/medicamentos` | HU-CAT-019 |
| `/medicamentos/nueva` | HU-CAT-018 |
| `/medicamentos/carga-masiva` | HU-CAT-022 |
| `/medicamentos/:id` | HU-CAT-018, HU-CAT-020, HU-CAT-021, HU-CAT-025 |
| `/presentaciones-venta` | HU-CAT-026 |
| `/presentaciones-venta/nueva` | HU-CAT-024 |
| `/presentaciones-venta/:id` | HU-CAT-024, HU-CAT-027 |
| `/formulaciones-clinicas` | HU-CAT-029 |
| `/formulaciones-clinicas/nueva` | HU-CAT-028 |
| `/formulaciones-clinicas/:id` | HU-CAT-028, HU-CAT-030 |
| `/proveedores` | HU-CAT-033 |
| `/proveedores/nueva` | HU-CAT-032 |
| `/proveedores/carga-masiva` | HU-CAT-035 |
| `/proveedores/:id` | HU-CAT-032, HU-CAT-034, HU-CAT-036, HU-CAT-037 |
| `/aseguradoras` | HU-CAT-039 |
| `/aseguradoras/nueva` | HU-CAT-038 |
| `/aseguradoras/carga-masiva` | HU-CAT-041 |
| `/aseguradoras/:id` | HU-CAT-038, HU-CAT-040 |
| `/digemid-catalogo` | HU-CAT-042 |
| `/digemid-catalogo/carga-masiva` | HU-CAT-043 |
| `/digemid-catalogo/:id` | HU-CAT-042 |
| `/lotes` | HU-INV-002 |
| `/lotes/nueva` | HU-INV-001 |
| `/lotes/:id` | HU-INV-001, HU-INV-002, HU-INV-003 |
| `/stock-por-sucursal` | HU-INV-007 |
| `/stock-por-sucursal/nueva` | HU-INV-005 |
| `/stock-por-sucursal/:id` | HU-INV-006 |
| `/pacientes` | HU-INV-010 |
| `/pacientes/nueva` | HU-INV-009 |
| `/pacientes/carga-masiva` | HU-INV-012 |
| `/pacientes/:id` | HU-INV-009, HU-INV-011 |
| `/profesionales-salud` | HU-INV-014 |
| `/profesionales-salud/nueva` | HU-INV-013 |
| `/profesionales-salud/carga-masiva` | HU-INV-016 |
| `/profesionales-salud/:id` | HU-INV-013, HU-INV-015 |
| `/recetas-medicas` | HU-INV-018 |
| `/recetas-medicas/escanear` | HU-INV-020, HU-INV-021, HU-INV-022 |
| `/recetas-medicas/nueva` | HU-INV-017 |
| `/recetas-medicas/:id` | HU-INV-017, HU-INV-019 |

### 20.3 Endpoints de mi alcance

Los 127 endpoints de [`endpoints.csv`](../../_inventario/endpoints.csv) de los controladores `UnidadesMedida`, `JerarquiasUoM`, `Laboratorios`, `IngredientesActivos`, `Medicamentos`, `PresentacionesVenta`, `FormulacionesClinicas`, `Proveedores`, `Seguros`, `LotesInventario`, `StockPorSucursal`, `Pacientes`, `ProfesionalesSalud` y `Recetas`, más los de `catalogo` del controlador `DIGEMID` (las operaciones de cuarentena y actas no se tratan aquí), figuran en la fila «Endpoint(s)» de al menos una historia (comprobado por script). Los endpoints `GET /api/v1/Sucursales/permitidas` y `GET /api/v1/Laboratorios`/`UnidadesMedida`/`Medicamentos` que alimentan las listas de elección se citan como apoyo de las historias que los usan.

---

## 21. Fuentes

Documentos consultados (rutas relativas a `sofia-docs`):

- `_inventario/REGLAS-AGENTES.md`, `_inventario/README.md`, `_inventario/rutas-spa.csv`, `_inventario/endpoints.csv`.
- `ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md` (versión 1.1; reglas `UDM`, `JER`, `LAB`, `ING`, `MED`, `PRE`, `FOR`, `PRV`, `ASE`, `DIG`, `PAC`, `PRO`; §2, §3, §3.1 y §16).
- `ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md` (reglas `LOT`, `STK`, `REC`; §2, §6).
- `ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md` (reglas `GEN` e `IMP` citadas) y `01c2-ventas-y-comprobantes.md` (`RN-VEN-019`).
- `ola-1-fundamentos/19-inventario-de-pantallas/19c-catalogos-1.md`, `19d-catalogos-2.md` y `19e-inventario-y-atencion.md` (fichas de pantalla, mensajes, mapas de eliminar y «No verificado»).
- `ola-1-fundamentos/20-carga-masiva-y-exportacion.md`, `ola-1-fundamentos/05-seguridad/05b-matriz-de-permisos.md` (rótulos de módulos y permisos) y `ola-1-fundamentos/06-glosario.md`.
- `ola-2-funcional/14-procesos-de-negocio.md` (proceso 3, catálogo e inventario, y proceso 7, recetas con IA).
- `ola-2-funcional/manual-usuario/capitulo-04-catalogos-basicos.md` y `capitulo-05-productos-y-proveedores.md` (recorridos reales en la aplicación local, mensajes comprobados).
- `ola-2-funcional/12-historias-de-usuario/12a-acceso-y-administracion.md` (formato de las historias).

Comprobaciones puntuales en el código (solo lectura): `sofia-apis` y `sofia-web-spa` en `43f5529` y `eeffbb6` (`git log -1` y `git status --short`, sin cambios locales; `git diff --stat` sobre lotes, stock, recetas, `Application/Inventarios` y `Application/Recetas` entre los commits de 01c1 y los actuales, sin cambios) y `sofia-web-spa/public/i18n/es.json` (la clave `fields.tipoDocumento` no existe en la raíz).
