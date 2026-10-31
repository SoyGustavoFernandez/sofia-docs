# Capítulo 5 — Productos y proveedores: medicamentos, jerarquías de unidades, presentaciones de venta, formulaciones clínicas, proveedores y aseguradoras

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-26 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Recorrido en la aplicación local más inventario de pantallas |

## 1. Introducción

En este capítulo registras lo que tu botica vende y a quién se lo compra. Vas a:

- Crear **medicamentos** con su laboratorio, su unidad base, su **condición de venta** y su precio de venta base.
- Definir las **jerarquías de unidades** (cuántos blísteres hay en una caja y cuántas unidades en un blíster).
- Crear **presentaciones de venta** (por ejemplo «Caja» o «Blíster») con su propio precio.
- Registrar **formulaciones clínicas** (qué ingrediente activo, en qué concentración, contiene cada medicamento).
- Registrar **proveedores** y los **precios de compra** que te cobran, con su historial.
- Crear **aseguradoras** y conocer qué parte del punto de venta las usa hoy.
- Probar qué pasa al **eliminar** algo que ya está en uso y usar la **carga masiva** de medicamentos.

Es para el **administrador** (rol «Admin», que lo puede todo). La sección 11 explica qué ve un usuario con menos permisos, por ejemplo un cajero.

Todos los ejemplos usan los datos ficticios del manual: la empresa «Botica Manual Demo S.A.C.» y los laboratorios, unidades e ingredientes que creaste en el capítulo 4. Los nombres de medicamentos, códigos, precios y proveedores son inventados.

## 2. Antes de empezar

- Haz los capítulos 1 a 4. Necesitas la sesión iniciada como administrador y estos catálogos del capítulo 4: las unidades `UND`, `CAJ`, `BLI` y `MG`; los laboratorios «Laboratorio Demo Andino» y «Laboratorio Demo Pacífico»; y los ingredientes activos Paracetamol, Amoxicilina, Clonazepam y Tramadol.
- Permisos que usa este capítulo. Los nombres son los que ves en la pantalla de permisos de un rol (Administración › Roles & Permisos): primero el módulo y luego la acción.

| Tarea | Módulo | Acciones |
|---|---|---|
| Ver, buscar y exportar medicamentos | Medicamentos | `Leer` |
| Crear un medicamento y usar su carga masiva | Medicamentos | `Crear` (además de `Leer`) |
| Editar o eliminar un medicamento | Medicamentos | `Actualizar` / `Eliminar` |
| Ver jerarquías, presentaciones, formulaciones | Jerarquías de Unidades, Presentaciones de Venta, Formulaciones Clínicas | `Leer` en cada módulo |
| Crear, editar o eliminar en esas pantallas | Los mismos módulos | `Crear`, `Actualizar`, `Eliminar` |
| Ver, crear, editar y eliminar proveedores | Proveedores | `Leer`, `Crear`, `Actualizar`, `Eliminar` |
| Registrar un precio de compra | Proveedores | `Registrar precio de proveedor` |
| Ver, crear, editar y eliminar aseguradoras | Seguros (la pantalla se llama «Aseguradoras») | `Leer`, `Crear`, `Actualizar`, `Eliminar` |

- Para elegir valores en los formularios, algunas pantallas consultan otros catálogos y necesitan además `Leer` en ellos: Medicamentos usa Laboratorios y Unidades de Medida; Jerarquías usa Medicamentos y Unidades de Medida; Presentaciones usa Medicamentos y Jerarquías de Unidades; Formulaciones usa Medicamentos, Ingredientes Activos y Unidades de Medida. El administrador los tiene todos.
- En el menú lateral, las pantallas están en el grupo **CATÁLOGOS**: «Unidades de Medida», «Jerarquías de Unidades», «Laboratorios», «Ingredientes Activos», «Medicamentos», «Presentaciones de Venta», «Formulaciones Clínicas», «Proveedores» y «Aseguradoras».
- Los avisos de la aplicación aparecen abajo al centro con un botón **Cerrar** y desaparecen solos a los pocos segundos. Los textos de este capítulo están copiados tal como se vieron en pantalla.
- Las pantallas de búsqueda, detalle (ver, editar, eliminar) y carga masiva funcionan como se explicó en el capítulo 4, sección 4. Aquí solo se repite lo que cambia.

## 3. Cómo se relacionan estas piezas (en lenguaje simple)

| Pieza | Qué es | Para qué la usa SOFIA |
|---|---|---|
| **Medicamento** (también llamado «producto») | El artículo que vendes, con un **código nacional** (el que tú le pones), un **nombre comercial**, un **laboratorio**, una **unidad base**, una **condición de venta** y un **precio de venta base** | Es lo que se ve en el inventario, en el punto de venta y en las recetas |
| **Unidad base** | La unidad más pequeña en la que cuentas ese producto (por ejemplo `UND`, una unidad) | El stock, el precio base y el descuento de stock en las ventas siempre se expresan en unidades base |
| **Condición de venta** | Una de cuatro categorías que dice si el producto se puede vender libremente o necesita receta (ver sección 4.2) | El punto de venta la usa para exigir o no una receta |
| **Precio de venta base** | El precio al público de **una unidad base** | Es el precio por defecto cuando vendes por unidad base |
| **Jerarquía de unidades** | Una conversión de un producto: «1 unidad mayor = N unidades menores» (por ejemplo, 1 Caja = 10 Blíster) | De las jerarquías sale cuántas unidades base tiene cada presentación |
| **Presentación de venta** | Una forma de vender un producto (por ejemplo «Caja») con **su propio precio** y su equivalencia en unidades base | Es lo que el cajero elige para vender una caja o un blíster completo |
| **Formulación clínica** | El dato de que un medicamento contiene un ingrediente activo en cierta concentración y unidad | Es información clínica; también impide eliminar ingredientes que están en uso |
| **Proveedor** | A quien le compras. Tiene razón social, RUC o identificador tributario y una tasa de cumplimiento | Guarda los **precios de compra** que te cobra, con su historial |
| **Aseguradora** | Una empresa de seguros médicos, con nombre comercial y código identificador nacional | Hoy solo se registra: el bloque de seguro del punto de venta está oculto (sección 9) |

Fuente de las definiciones: glosario del proyecto (`06-glosario.md`: «Condición de venta», «Unidad base», «Jerarquía de unidades», «Presentación de venta», «Precio de venta base», «Precio de compra de proveedor»).

## 4. Medicamentos

**Cómo llegar:** menú lateral › CATÁLOGOS › **Medicamentos** (`/medicamentos`).

### 4.1 Qué medicamentos crear

Para los capítulos siguientes se crean cuatro medicamentos ficticios, uno por cada condición de venta. Todos tienen la unidad base `UND` (que en pantalla se llama «Unidad»).

| Código Nacional | Nombre Comercial | Laboratorio | Unidad Base | Condición de Venta | Precio de Venta Base |
|---|---|---|---|---|---|
| `DEMO-MED-0001` | Paracetamol Demo 500 mg | Laboratorio Demo Andino | Unidad | Venta Libre (OTC) | S/ 0.50 |
| `DEMO-MED-0002` | Amoxicilina Demo 500 mg | Laboratorio Demo Pacífico | Unidad | Receta Simple | S/ 1.20 |
| `DEMO-MED-0003` | Clonazepam Demo 2 mg | Laboratorio Demo Andino | Unidad | Receta Retenida | S/ 2.50 |
| `DEMO-MED-0004` | Tramadol Demo 50 mg | Laboratorio Demo Pacífico | Unidad | Estupefaciente | S/ 3.80 |

### 4.2 La condición de venta y qué implica en el punto de venta

La **condición de venta** es obligatoria y solo admite estos cuatro valores, que ves en este orden en la lista desplegable:

| Valor | Qué significa | Qué pasa en el punto de venta |
|---|---|---|
| **Venta Libre (OTC)** | Se vende sin receta | No exige receta |
| **Receta Simple** | Se vende con receta | La línea se marca «Requiere receta». Si tu empresa tiene activada la opción de exigir registro de receta, hay que elegir una receta registrada; si no, basta marcar la casilla «Receta verificada» («Confirmas que revisaste la receta física del cliente. Quedará registrado a tu nombre.») |
| **Receta Retenida** | Se vende con receta que queda retenida en la botica | Siempre exige una receta registrada que tenga **número de receta**, del mismo cliente de la venta, y cuenta como una sola dispensación |
| **Estupefaciente** | Producto controlado | Igual que la receta retenida |

Textos del punto de venta que lo confirman: «Receta retenida y estupefacientes siempre exigen una receta registrada con su número.» y «Hay productos de receta retenida o estupefacientes: selecciona al cliente y una receta registrada que tenga número de receta.»

(No verificado en la aplicación en este capítulo: el comportamiento de cada condición al cobrar se recorre en los capítulos de recetas y ventas. Aquí viene del glosario del proyecto, `06-glosario.md` «Condición de venta», y de los textos de `pos/es.json` citados. SOFIA **no registra** que la receta física haya sido retenida o archivada: solo deja la dispensación contada.)

### 4.3 La pantalla de búsqueda

Título «Medicamentos», migas «Inicio / Catálogos / Medicamentos» y tarjeta «Búsqueda de Medicamentos» con «Ingresa los criterios de búsqueda». Arriba a la derecha de la tarjeta, **Carga Masiva** y **Nuevo** (con permiso `Crear`). Filtros:

| Filtro | Qué hace |
|---|---|
| Código Nacional | Texto con lupa. Busca códigos que **contengan** lo que escribas, sin distinguir mayúsculas |
| Nombre Comercial | Texto con lupa. «Contiene», sin distinguir mayúsculas |
| Laboratorio | Texto con lupa y lista de sugerencias («Buscar laboratorio...»). Filtra por el **nombre** que contenga lo que escribas, aunque no elijas una sugerencia |
| Unidad Base | Texto con lupa y sugerencias («Buscar unidad de medida...»). Filtra por la descripción de la unidad |
| Condición de Venta | Lista desplegable: «— Todos —», «Venta Libre (OTC)», «Receta Simple», «Receta Retenida» y «Estupefaciente» |

La tabla tiene cinco columnas: **Código Nacional** (enlace al detalle), **Nombre Comercial**, **Laboratorio**, **Unidad Base** (la descripción, por ejemplo «Unidad») y **Condición de Venta**. Se ordena por nombre comercial. **No muestra** el precio ni el stock. Debajo, **Exportar** y el paginador. Encabezado de resultados: «Resultados (N)», donde N es el número de coincidencias. Si no hay coincidencias: «Resultados (0)», fila «Sin resultados».

### 4.4 Crear un medicamento

1. Haz clic en **Nuevo** (`/medicamentos/nueva`).
2. Escribe el «Código Nacional» y el «Nombre Comercial».
3. En «Laboratorio», escribe parte del nombre (por ejemplo «Andino») y **elige una opción de la lista**.
4. En «Unidad Base», escribe «Unidad» y elige `UND Unidad` de la lista (cada opción muestra el código y la descripción).
5. En «Condición de Venta», elige una de las cuatro.
6. Escribe el «Precio de Venta Base» (el campo trae el prefijo «S/»).
7. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Código Nacional | El código con el que identificas el producto (no es el código DIGEMID, aunque puedes usar el mismo) | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 50 caracteres. No puede repetirse en tu empresa, ni con otras mayúsculas: «Ya existe un medicamento con ese código nacional.» |
| Nombre Comercial | El nombre con el que lo vendes, por ejemplo «Paracetamol Demo 500 mg» | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 150 caracteres. **Puede repetirse** entre medicamentos con distinto código |
| Laboratorio | Un laboratorio del catálogo | Sí | Debes **elegir una opción de la lista**: si solo escribes el texto, al guardar sale «Este campo es obligatorio» bajo el campo. Sin coincidencias, la lista muestra «Sin resultados» |
| Unidad Base | Una unidad de medida del catálogo | Sí | Igual que el laboratorio |
| Condición de Venta | Una de las cuatro condiciones | Sí | Vacío: «Este campo es obligatorio» |
| Precio de Venta Base | El precio al público de **una** unidad base, en soles | Sí | Cualquier problema (vacío o negativo) muestra «El precio no puede ser negativo». El cero es válido |

**Probar los errores (comprobado en la aplicación):**

1. Haz clic en **Guardar** con el formulario vacío: aparece «Este campo es obligatorio» bajo cada uno de los cinco primeros campos y, bajo el precio, «El precio no puede ser negativo» (aunque el campo esté vacío). No se envía nada.
2. Escribe `-1` en el precio y un laboratorio sin elegir de la lista (por ejemplo, el texto «zzz»): al guardar siguen los dos mensajes, «Este campo es obligatorio» en «Laboratorio» y «El precio no puede ser negativo».
3. Guarda `DEMO-MED-0001` / «Paracetamol Demo 500 mg». Luego intenta crear otro medicamento con el código `demo-med-0001`: aviso rojo «Ya existe un medicamento con ese código nacional.» y te quedas en el formulario.
4. Escribe solo espacios en «Código Nacional»: aviso rojo «Error al guardar el medicamento» y te quedas en el formulario. Escribe siempre un valor real.

**Qué pasa al terminar bien:** aparece el aviso «Medicamento guardado correctamente» y el sistema te lleva a la búsqueda (con los filtros vacíos y sin tabla: haz clic en **Buscar**). Repite los pasos con los otros tres medicamentos de la tabla 4.1.

Si no ves **Nuevo**, a tu rol le falta `Medicamentos › Crear`.

Errores al guardar un medicamento:

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Ya existe un medicamento con ese código nacional.» | Otro medicamento no eliminado de tu empresa tiene ese código (comprobado) | Usa otro código o abre el existente |
| «Error al guardar el medicamento» | El servidor rechazó el guardado: código o nombre de solo espacios (comprobado), laboratorio o unidad que ya no existen, u otro problema | Revisa los datos e inténtalo de nuevo |
| «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.», «El registro hace referencia a datos que ya no existen o todavía está en uso.» o «Uno de los valores está fuera del rango permitido.» | Errores de base de datos poco frecuentes | Revisa los datos e inténtalo de nuevo (No verificado en la aplicación: no se pudieron provocar; vienen de 19d §0) |
| «Error al cargar las condiciones de venta» | La lista de condiciones no cargó | Recarga la pantalla (No verificado: no se pudo provocar) |

### 4.5 Buscar, filtrar y exportar

Comprobado con los cuatro medicamentos de la tabla 4.1:

- **Buscar** sin filtros: «Resultados (4)», ordenados por nombre: Amoxicilina, Clonazepam, Paracetamol y Tramadol.
- «Código Nacional» = `0001`: solo Paracetamol. «Nombre Comercial» = `DEMO 500`: Amoxicilina y Paracetamol. «Laboratorio» = `andino`: Clonazepam y Paracetamol. «Unidad Base» = `unid`: los cuatro. «Condición de Venta» = «Receta Retenida»: solo Clonazepam. «Laboratorio» = `pac` junto con «Condición de Venta» = «Estupefaciente»: solo Tramadol (los filtros se combinan). «Código Nacional» = `zzz`: «Resultados (0)» y la fila «Sin resultados».
- **Limpiar** vacía los filtros y oculta la tabla.
- **Exportar** genera `medicamentos.xlsx`. Comprobado: el archivo recibido por la pantalla es un libro de Excel con una hoja llamada «Data», una fila de encabezados («Código Nacional», «Nombre Comercial», «Laboratorio», «Unidad Base», «Condición de Venta») y una fila por medicamento con los textos de la tabla (la condición en texto, por ejemplo «Receta Simple»). **No incluye precio ni stock.** Respeta los filtros que tengas escritos al pulsar el botón. Si falla la búsqueda o la exportación verás «Error al buscar medicamentos».

(No verificado en la aplicación: el nombre del archivo en el disco, porque el navegador integrado no muestra las descargas; viene de `medicamentos.search.export-filename`.)

### 4.6 Ver, editar y eliminar un medicamento

1. En los resultados, haz clic en el código nacional (por ejemplo `DEMO-MED-0001`). Se abre el detalle con los seis campos deshabilitados. Arriba de la tarjeta ves el título «Medicamentos», el texto «Datos del registro» y, a la derecha, los botones **Presentaciones de Venta** (icono de etiqueta) y **Editar**. Abajo hay una tarjeta «Stock por Sucursal» con la línea «Stock Total: 0» y el texto «Sin stock registrado» (todavía no hay lotes). Cuando existan lotes, la tabla muestra una fila por sucursal con las columnas «Sucursal» y «Stock».
2. Haz clic en **Editar**: se habilitan los campos y aparecen **Cancelar**, **Guardar** y **Eliminar** (este último con el permiso `Eliminar`). Cambia el nombre y haz clic en **Cancelar**: el campo vuelve a su valor anterior y el detalle vuelve al modo ver (comprobado).
3. Haz clic en **Editar**, cambia el «Precio de Venta Base» de 0.50 a 0.55 y haz clic en **Guardar**: aparece «Medicamento guardado correctamente» y **te quedas** en el detalle, otra vez en modo ver, con el nuevo precio (comprobado). Se volvió a poner 0.50 al terminar.
4. Si escribes un precio con tres decimales (`0.505`), el sistema lo guarda **redondeado a dos decimales** (`0.51`, comprobado).
5. Los botones **Presentaciones de Venta** y **Editar** solo se ven si tu rol tiene los permisos: **Editar** pide `Medicamentos › Actualizar`; **Presentaciones de Venta** pide `Presentaciones de Venta › Leer` y `Jerarquías de Unidades › Leer`.

Si el detalle no se pudo abrir verás «Error al cargar el medicamento» (No verificado en la aplicación: no se pudo provocar).

**Eliminar.** En modo editar, **Eliminar** abre «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este medicamento?» y los botones **Cancelar** (no borra nada, comprobado) y **Confirmar**. Al confirmar, si el medicamento no tiene dependencias, aparece «Medicamento eliminado correctamente»; si las tiene, el aviso rojo explicado en la sección 10.1. Cualquier otro fallo muestra «Error al eliminar el medicamento».

La eliminación de un medicamento (y qué pasa cuando está en uso) está en la sección 10.1.

## 5. Jerarquías de unidades

**Cómo llegar:** menú lateral › CATÁLOGOS › **Jerarquías de Unidades** (`/jerarquias`).

### 5.1 Qué es una jerarquía

Un producto se cuenta en una unidad (la unidad base), pero se compra y se vende en envases más grandes. Una **jerarquía de unidades** es **una sola conversión de un producto**: «1 unidad mayor = N unidades menores». Cada registro es un par de unidades y un multiplicador. Una caja de Paracetamol de 100 unidades se describe con **dos** jerarquías encadenadas:

| Producto | Unidad Mayor | Unidad Menor | Multiplicador | Se lee |
|---|---|---|---|---|
| Paracetamol Demo 500 mg | Caja | Blíster | 10 | 1 Caja = 10 Blíster |
| Paracetamol Demo 500 mg | Blíster | Unidad | 10 | 1 Blíster = 10 Unidad |

El sistema resuelve la cadena solo: de estas dos filas deduce que 1 Caja = 100 Unidad. La pantalla **no muestra** la cadena ni calcula equivalencias; esa cuenta la hace el sistema al guardar (para impedir contradicciones) y al crear las presentaciones de venta (sección 6). También puedes registrar un salto directo (1 Caja = 30 Unidad) si el producto no tiene blísteres.

Las jerarquías que se crean en este capítulo:

| Producto | Unidad Mayor | Unidad Menor | Multiplicador | Equivale a |
|---|---|---|---|---|
| Paracetamol Demo 500 mg | Caja | Blíster | 10 | 1 Caja = 100 Unidad |
| Paracetamol Demo 500 mg | Blíster | Unidad | 10 | 1 Blíster = 10 Unidad |
| Amoxicilina Demo 500 mg | Caja | Blíster | 5 | 1 Caja = 50 Unidad |
| Amoxicilina Demo 500 mg | Blíster | Unidad | 10 | 1 Blíster = 10 Unidad |
| Clonazepam Demo 2 mg | Caja | Unidad | 30 | 1 Caja = 30 Unidad (salto directo) |

### 5.2 La pantalla de búsqueda

Título «Jerarquías de Unidades», migas «Inicio / Catálogos / Jerarquías de Unidades» y tarjeta «Búsqueda de Jerarquías». Arriba a la derecha, **Carga Masiva** y **Nuevo** (con permiso `Crear`). Filtros:

| Filtro | Qué hace |
|---|---|
| Producto | Texto con lupa y sugerencias («Buscar producto...»). Busca por el nombre comercial que **contenga** lo que escribas, aunque no elijas una sugerencia |
| Unidad Mayor | Texto con lupa y sugerencias («Buscar unidad de medida...»). Filtra por la descripción de la unidad mayor |
| Unidad Menor | Igual, para la unidad menor |
| Multiplicador | Deslizador de rango con dos tiradores; la línea dice «Multiplicador: 0 — 1000» y cambia mientras lo mueves. Va de 0 a 1000 en pasos de 1. Solo filtra el límite que moviste |

La tabla tiene cuatro columnas: **Producto** (enlace al detalle), **Unidad Mayor**, **Unidad Menor** y **Multiplicador** (a la derecha). Se ordena por producto. Fila sin resultados: «Sin resultados». Debajo, **Exportar** y el paginador.

Comprobado con las cinco jerarquías de la sección 5.1: **Buscar** sin filtros da «Resultados (5)»; «Producto» = `paracet` da las dos de Paracetamol; «Unidad Mayor» = `caja` da tres; «Unidad Menor» = `unid` da tres; «Unidad Mayor» = `blí` junto con «Unidad Menor» = `unid` da las dos de Blíster → Unidad; «Producto» = `zzz` da «Resultados (0)». Con el deslizador en 6 — 20 salen las tres jerarquías de multiplicador 10, y con 11 — 1000 sale solo la de multiplicador 30.

**Exportar** genera `jerarquias-unidades.xlsx`. Comprobado: hoja «Data» con las columnas «Producto», «Unidad Mayor», «Unidad Menor» y «Multiplicador» y una fila por jerarquía. Si falla la búsqueda o la exportación: «Error al buscar jerarquías de unidades».

### 5.3 Crear una jerarquía

1. Haz clic en **Nuevo** (`/jerarquias/nueva`).
2. En «Producto», escribe parte del nombre y **elige una opción de la lista**.
3. En «Unidad Mayor» y «Unidad Menor» escribe y elige una opción de la lista cada una.
4. Escribe el «Multiplicador» (debajo dice «Cuántas unidades menores equivalen a una unidad mayor»).
5. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Producto | El medicamento | Sí | Debes elegir una opción de la lista. Si no: «Este campo es obligatorio» (aparece al pulsar **Guardar**) |
| Unidad Mayor | La unidad más grande de la conversión (Caja) | Sí | Igual |
| Unidad Menor | La unidad más pequeña (Blíster o Unidad) | Sí | Igual. Debe ser distinta de la mayor: si eliges la misma aparece de inmediato «La unidad mayor y la unidad menor no pueden ser la misma» y **Guardar** no envía nada |
| Multiplicador | Cuántas unidades menores caben en una mayor | Sí | Vacío, cero o menor que 0,0001: «El multiplicador debe ser mayor que cero». (Que acepte decimales: No verificado en la aplicación) |

**Probar los errores (comprobado en la aplicación):**

1. **Guardar** con el formulario vacío: «Este campo es obligatorio» bajo «Producto», «Unidad Mayor» y «Unidad Menor», y «El multiplicador debe ser mayor que cero».
2. Elige Paracetamol, «Caja» en las dos unidades y multiplicador 10: aparece «La unidad mayor y la unidad menor no pueden ser la misma».
3. Cambia la unidad menor a «Blíster» y el multiplicador a `0`: «El multiplicador debe ser mayor que cero». Con el campo vacío sale el mismo mensaje.
4. Guarda Paracetamol / Caja / Blíster / 10: aparece «Jerarquía guardada correctamente» y el sistema te lleva a la búsqueda. Crea después Paracetamol / Blíster / Unidad / 10.
5. **Jerarquía duplicada.** Intenta crear otra vez Paracetamol / Caja / Blíster, aunque sea con otro multiplicador (12): aviso rojo «Este producto ya tiene una conversión entre esas unidades.» y te quedas en el formulario.
6. **Jerarquía inconsistente.** Con las dos jerarquías de Paracetamol, intenta crear Caja / Unidad / 50. Como la cadena ya dice que 1 Caja = 100 Unidad, el sistema rechaza la contradicción con el aviso genérico «Error al guardar la jerarquía». Pasa lo mismo con Blíster / Caja / 2, que contradice a Caja / Blíster / 10. El mensaje no explica el motivo: si lo ves, revisa las jerarquías del producto.

Crea el resto de la tabla de 5.1 con los mismos pasos. Si no ves **Nuevo**, te falta `Jerarquías de Unidades › Crear`.

Errores al guardar una jerarquía:

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Este producto ya tiene una conversión entre esas unidades.» | Ya existe una jerarquía de ese producto con ese par de unidades (comprobado) | Edita la existente en lugar de crear otra |
| «Error al guardar la jerarquía» | El servidor rechazó el guardado: conversión contradictoria con las otras jerarquías del producto (comprobado), producto o unidad que ya no existen, u otro problema | Revisa las demás jerarquías del producto y el multiplicador |
| Los tres avisos de base de datos del capítulo 4 (sección 5.3) | Errores poco frecuentes | Igual que allí (No verificado en la aplicación) |

### 5.4 Ver, editar y eliminar

1. Haz clic en el nombre del producto en la tabla: se abre el detalle con los cuatro campos deshabilitados y el botón **Editar** (permiso `Actualizar`).
2. **Editar** habilita los campos y muestra **Cancelar**, **Guardar** y **Eliminar** (permiso `Eliminar`). **Cancelar** descarta los cambios y vuelve al modo ver (comprobado con el multiplicador de Clonazepam: 40 escrito, 30 restaurado).
3. **Guardar** muestra «Jerarquía guardada correctamente». **A diferencia de los catálogos del capítulo 4, al editar con éxito el sistema te lleva a la búsqueda** y no te deja en el detalle (comprobado). Se cambió el multiplicador de Clonazepam de 30 a 40 y se volvió a 30.
4. **Eliminar** abre «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar esta jerarquía?» y los botones **Cancelar** (no borra nada, comprobado) y **Confirmar**. Al confirmar: «Jerarquía eliminada correctamente» y vuelves a la búsqueda. Para probarlo se creó una jerarquía de prueba (Tramadol Demo 50 mg, Frasco → Unidad, 20) y se eliminó; ya no existe.
5. Si el sistema respondiera que está en uso, el aviso sería «No se puede eliminar la jerarquía porque está en uso.»; hoy el servidor no comprueba dependencias al eliminar una jerarquía, por lo que ese caso no se pudo provocar (No verificado). Otro fallo: «Error al eliminar la jerarquía». Si el detalle no abre: «Error al cargar la jerarquía».

La carga masiva de jerarquías está en la sección 10.4.

## 6. Presentaciones de venta

**Cómo llegar:** menú lateral › CATÁLOGOS › **Presentaciones de Venta** (`/presentaciones-venta`). También puedes administrarlas desde la ficha de un medicamento, con el botón **Presentaciones de Venta** (sección 6.5).

### 6.1 Qué es una presentación y cómo se calcula su equivalencia

Una **presentación de venta** une **un medicamento** con **una unidad de venta** (una unidad distinta de su unidad base) y **un precio de venta propio**. Sirve para vender una caja o un blíster completo de un solo golpe, con un precio que no tiene por qué ser el precio base multiplicado.

- **La unidad de venta sale de las jerarquías.** Solo puedes elegir las unidades que aparecen en las jerarquías del producto (sección 5), sin contar su unidad base. Si el producto no tiene jerarquías, la pantalla no te deja crear presentaciones.
- **La equivalencia la calcula el sistema.** No la escribes. Al guardar, el sistema resuelve la cadena de jerarquías y guarda cuántas unidades base tiene la unidad de venta. En la lista de unidades la ves entre paréntesis: para Paracetamol aparecen «Caja (100 Unidad)» y «Blíster (10 Unidad)» (1 Caja = 10 Blíster × 10 Unidad). Para Amoxicilina, «Caja (50 Unidad)» y «Blíster (10 Unidad)».
- **La descripción no se escribe.** La presentación se llama como la unidad de venta («Caja», «Blíster»). Por eso en pantalla verás «Caja» y «Blíster», no «Caja x100» ni «Blíster x10»; la cantidad de unidades está en la columna «Equivalencia».
- **El precio es propio.** No se deriva del «Precio de Venta Base»: tú decides cuánto cuesta la caja (por ejemplo, S/ 45.00 en vez de 100 × S/ 0.50 = S/ 50.00).
- **Un producto tiene como máximo una presentación por unidad de venta.**
- La equivalencia se guarda al crear o editar la presentación.

Las presentaciones que se crean en este capítulo:

| Medicamento | Unidad de Venta | Equivalencia | Precio de Venta |
|---|---|---|---|
| Paracetamol Demo 500 mg | Caja | 100 | S/ 45.00 |
| Paracetamol Demo 500 mg | Blíster | 10 | S/ 4.80 |
| Amoxicilina Demo 500 mg | Caja | 50 | S/ 55.00 |
| Amoxicilina Demo 500 mg | Blíster | 10 | S/ 11.00 |
| Clonazepam Demo 2 mg | Caja | 30 | S/ 70.00 |

### 6.2 La pantalla de búsqueda

Título «Presentaciones de Venta», migas «Inicio / Catálogos / Presentaciones de Venta» y tarjeta «Búsqueda de Presentaciones de Venta». Arriba a la derecha, solo el botón **Nuevo** (con permiso `Crear`): esta pantalla **no tiene carga masiva**. Filtros:

| Filtro | Qué hace |
|---|---|
| Producto | Texto con lupa y sugerencias («Buscar producto...»). Si eliges una sugerencia filtra por ese producto exacto; si solo escribes, filtra por nombre que contenga el texto |
| Unidad de Venta | Texto con lupa. Busca presentaciones cuya unidad de venta contenga lo que escribas |

La tabla tiene cuatro columnas: **Producto** (enlace al detalle), **Unidad de Venta**, **Equivalencia** (número, a la derecha, sin nombre de unidad) y **Precio de Venta** (con «S/» y dos decimales). Se ordena por producto y luego por unidad. Fila sin resultados: «Sin resultados».

Comprobado con las cinco presentaciones: **Buscar** sin filtros da «Resultados (5)»; «Producto» = `paracet` (con o sin elegir la sugerencia) da las dos de Paracetamol; «Unidad de Venta» = `caja` da tres (una por medicamento); «Producto» = Amoxicilina elegido de la lista junto con «Unidad de Venta» = `bl` da solo la de Blíster de Amoxicilina; «Producto» = `zzz` da «Resultados (0)».

**Exportar** genera `presentaciones-venta.xlsx`. Comprobado: hoja «Data» con las columnas «Producto», «Unidad de Venta», «Equivalencia» y «Precio de Venta» y una fila por presentación (precios como números, por ejemplo 4.8). Si falla la búsqueda o la exportación: «Error al buscar presentaciones de venta».

### 6.3 Crear una presentación

1. Haz clic en **Nuevo** (`/presentaciones-venta/nueva`). Solo ves el campo «Producto».
2. Escribe parte del nombre del medicamento y **elige una opción de la lista**.
3. Aparecen «Unidad de Venta» y «Precio de Venta». Elige la unidad (cada opción dice, por ejemplo, «Caja (100 Unidad)»).
4. Escribe el «Precio de Venta» (con prefijo «S/»).
5. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Producto | El medicamento | Sí | Debes elegir una opción de la lista. Si pulsas **Guardar** sin producto: «Este campo es obligatorio». Una vez creada, el producto **no se puede cambiar** (en la ficha existente dice «El producto de una presentación no se puede modificar luego de creada») |
| Unidad de Venta | Una de las unidades de las jerarquías del producto | Sí | Sin elegir: «Debes seleccionar una unidad de venta» |
| Precio de Venta | El precio de esa unidad de venta, en soles | Sí | Vacío o negativo: «El precio de venta no puede ser negativo». El cero es válido |

**Probar los errores (comprobado en la aplicación):**

1. **Guardar** sin elegir producto: «Este campo es obligatorio» bajo «Producto».
2. Elige un producto **sin jerarquías** (Tramadol Demo 50 mg): en lugar de unidad y precio aparece «Este producto no tiene una Jerarquía de Unidades registrada. Cárgala primero para poder definir presentaciones de venta.» y el botón **Ir a Jerarquías de Unidades** (solo si tu rol tiene `Jerarquías de Unidades › Crear`), que te lleva a `/jerarquias/nueva`. Si pulsas **Guardar** en ese estado no pasa nada: no se envía nada y no aparece ningún mensaje.
3. Elige Paracetamol y pulsa **Guardar** sin unidad ni precio: «Debes seleccionar una unidad de venta» y «El precio de venta no puede ser negativo».
4. Elige «Caja (100 Unidad)» con precio `-5` o vacío: «El precio de venta no puede ser negativo».
5. Precio `45`: aviso «Presentación de venta guardada correctamente» y vuelves a la búsqueda.
6. **Presentación duplicada.** Intenta crear otra «Caja» para Paracetamol (la lista aún muestra «Caja (100 Unidad)»): aviso rojo «Ya existe una presentación de venta para esa unidad en este producto.»

Crea el resto de la tabla de 6.1. Si no ves **Nuevo**, te falta `Presentaciones de Venta › Crear`.

Errores al guardar una presentación:

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Ya existe una presentación de venta para esa unidad en este producto.» | Ese producto ya tiene presentación para esa unidad (comprobado) | Edita la existente |
| «Error al guardar la presentación de venta» | El servidor rechazó el guardado: por ejemplo, el producto o la unidad ya no existen, o las jerarquías ya no permiten convertir esa unidad | Revisa las jerarquías del producto (No verificado en la aplicación: no se pudo provocar) |

### 6.4 Ver, editar y eliminar una presentación

1. Haz clic en el nombre del producto en la tabla: el detalle muestra el producto (deshabilitado, con la ayuda «El producto de una presentación no se puede modificar luego de creada»), la unidad de venta en texto, el precio deshabilitado y el botón **Editar** (permiso `Actualizar`).
2. **Editar** habilita la unidad de venta y el precio (el producto sigue deshabilitado) y muestra **Cancelar**, **Guardar** y **Eliminar** (permiso `Eliminar`). **Cancelar** descarta lo escrito y vuelve al modo ver (comprobado: se escribió 5 y el precio siguió en 4.8).
3. Al cambiar la unidad a otra que el producto ya tiene (por ejemplo, de «Blíster» a «Caja» en Paracetamol), **Guardar** muestra «Ya existe una presentación de venta para esa unidad en este producto.» (comprobado).
4. Cambia el precio de 4.80 a 5 y haz clic en **Guardar**: aparece «Presentación de venta guardada correctamente» y **vuelves a la búsqueda** (no te quedas en el detalle), donde ves «S/ 5.00» (comprobado). Se volvió a poner 4.80.
5. **Eliminar** abre «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar esta presentación de venta?» y los botones **Cancelar** y **Confirmar**; al confirmar: «Presentación de venta eliminada correctamente» y vuelves a la búsqueda. Si la presentación ya aparece en alguna venta, el aviso es «No se puede eliminar la presentación porque tiene ventas registradas.» (No verificado en la aplicación: todavía no hay ventas). Otro fallo: «Error al eliminar la presentación de venta». Si el detalle no abre: «Error al cargar la presentación de venta».

Para recorrer la eliminación se creó y eliminó una presentación de prueba (sección 6.5); no queda ninguna.

### 6.5 Administrar las presentaciones desde la ficha del medicamento

En el detalle de un medicamento, el botón **Presentaciones de Venta** abre un cuadro del mismo nombre con el nombre del medicamento debajo del título. Solo lo ves con los permisos `Presentaciones de Venta › Leer` y `Jerarquías de Unidades › Leer`. Según el producto, muestra:

| Situación | Qué ves |
|---|---|
| El producto no tiene jerarquías | «Este producto no tiene una Jerarquía de Unidades registrada. Cárgala primero para poder definir presentaciones de venta.» y el botón **Ir a Jerarquías de Unidades** (con `Jerarquías de Unidades › Crear`), que cierra el cuadro y te lleva a crear una jerarquía (comprobado con Tramadol Demo 50 mg) |
| Tiene jerarquías pero ninguna presentación | «Este producto todavía no tiene presentaciones de venta.» y el botón **Agregar presentación** (comprobado) |
| Tiene presentaciones | Una tabla con «Unidad de Venta», «Equivalencia» (por ejemplo «10 Unidad», con el nombre de la unidad base) y «Precio de Venta», y en cada fila los iconos de **editar** (lápiz) y **eliminar** (papelera), según tus permisos. Debajo, **Agregar presentación** y, al pie, **Cerrar** (comprobado con Paracetamol) |

- **Agregar presentación** abre un formulario en el mismo cuadro con «Unidad de Venta» y «Precio de Venta», y los botones **Cancelar** y **Guardar**. La lista de unidades solo trae las que **todavía no tienen** presentación en ese producto (a diferencia de la pantalla completa, que muestra todas). Si ya tienes una presentación para cada unidad, el botón queda **deshabilitado** y al pasar el cursor por encima su ayuda dice «Ya existe una presentación para cada unidad definida en la Jerarquía de este producto.» (comprobado: Paracetamol con Caja y Blíster, y Tramadol con la única unidad que tenía). Con la unidad sin elegir o el precio vacío o negativo aparecen los mensajes de la tabla 6.3 (comprobado con el precio: «El precio de venta no puede ser negativo»).
- Al guardar: «Presentación de venta guardada correctamente» y la tabla se actualiza sin salir del cuadro. El lápiz abre el mismo formulario con los datos de esa fila (comprobado). Un duplicado muestra «Ya existe una presentación de venta para esa unidad.» y cualquier otro fallo «Error al guardar la presentación de venta» (No verificado en la aplicación: el cuadro ya no ofrece unidades repetidas, así que no se pudo provocar).
- La papelera abre «Confirmar eliminación» («¿Estás seguro de que deseas eliminar esta presentación de venta?»); **Cancelar** no borra nada y **Confirmar** muestra «Presentación de venta eliminada correctamente» (comprobado). Con ventas registradas: «No se puede eliminar la presentación porque tiene ventas registradas.» (No verificado). Fallos de carga: «Error al cargar las presentaciones de venta».
- Para probarlo se creó, con el producto Tramadol Demo 50 mg, una jerarquía de prueba (Frasco → Unidad, 20) y una presentación (Frasco, S/ 60.00); se eliminaron las dos al terminar.

## 7. Formulaciones clínicas

**Cómo llegar:** menú lateral › CATÁLOGOS › **Formulaciones Clínicas** (`/formulaciones-clinicas`).

### 7.1 Qué es una formulación clínica

Una **formulación clínica** dice qué contiene un medicamento: liga **un medicamento** con **un ingrediente activo**, una **concentración de dosis** (un número) expresada en una **unidad de medida** y, si quieres, un **código TE Orange**. Un medicamento lista cada ingrediente una sola vez; para cambiar la concentración se edita esa fila. Un medicamento con varios ingredientes tiene varias filas. Es información clínica: no cambia precios ni stock. Sí tiene un efecto práctico: **un ingrediente activo incluido en una formulación no se puede eliminar** (sección 10.2).

Las formulaciones que se crean en este capítulo:

| Medicamento | Ingrediente Activo | Concentración | Unidad de Medida | Código TE Orange |
|---|---|---|---|---|
| Paracetamol Demo 500 mg | Paracetamol | 500 | Miligramo | (vacío) |
| Amoxicilina Demo 500 mg | Amoxicilina | 500 | Miligramo | AB |
| Clonazepam Demo 2 mg | Clonazepam | 2 | Miligramo | (vacío) |
| Tramadol Demo 50 mg | Tramadol | 50 | Miligramo | (vacío) |

### 7.2 La pantalla de búsqueda

Título «Formulaciones Clínicas», migas «Inicio / Catálogos / Formulaciones Clínicas» y tarjeta «Búsqueda de Formulaciones Clínicas». Arriba a la derecha solo está **Nuevo** (con permiso `Crear`); no hay carga masiva. Filtros:

| Filtro | Qué hace |
|---|---|
| Medicamento | Texto con lupa. Busca nombres comerciales que **contengan** lo que escribas, sin distinguir mayúsculas |
| Ingrediente Activo | Texto con lupa. «Contiene», sobre la denominación DCI |

La tabla tiene cinco columnas: **Medicamento** (enlace al detalle), **Ingrediente Activo**, **Concentración**, **Unidad de Medida** (la descripción, por ejemplo «Miligramo») y **Código TE Orange** (vacío si no tiene). Se ordena por medicamento. Fila sin resultados: **«Resultados»** (no dice «Sin resultados»). Debajo, **Exportar** y el paginador.

### 7.3 Crear una formulación

1. Haz clic en **Nuevo** (`/formulaciones-clinicas/nueva`).
2. En «Medicamento» (placeholder «Buscar medicamento...»), escribe parte del nombre y **elige una opción de la lista**.
3. En «Ingrediente Activo» (placeholder «Buscar ingrediente activo...»), escribe parte de la DCI y elige una opción.
4. Escribe la «Concentración de Dosis».
5. En «Unidad de Medida», escribe y elige una opción (por ejemplo «Miligramo»).
6. Si lo tienes, escribe el «Código TE Orange».
7. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Medicamento | El medicamento al que pertenece | Sí | Debes elegir una opción de la lista (y el texto debe seguir igual al nombre elegido). Si no: «Este campo es obligatorio». No se puede cambiar después de crear la formulación |
| Ingrediente Activo | El ingrediente que contiene | Sí | Igual que el medicamento |
| Concentración de Dosis | La cantidad del ingrediente, como número (por ejemplo `500`) | Sí | Debe ser mayor que cero (hasta 4 decimales). Vacío, cero o negativo muestran el mismo texto, «Este campo es obligatorio», que en estos dos últimos casos no describe el problema |
| Unidad de Medida | La unidad en que se expresa la concentración | Sí | Debes elegir una opción de la lista. Si no: «Este campo es obligatorio» |
| Código TE Orange | Código de equivalencia terapéutica, si lo usas | No | No deja escribir más de 5 caracteres y no muestra mensaje por ello |

**Probar los errores (comprobado en la aplicación):**

1. **Guardar** con el formulario vacío: «Este campo es obligatorio» bajo cada uno de los cuatro campos obligatorios.
2. Elige medicamento, ingrediente y unidad, y escribe `0` en la concentración: **Guardar** muestra «Este campo es obligatorio» bajo la concentración y no envía nada. Con `-5` ocurre lo mismo.
3. Escribe `500`, deja el código vacío y haz clic en **Guardar**: aparece «Formulación clínica guardada correctamente» y vuelves a la búsqueda.
4. **Formulación duplicada.** Intenta crear otra vez Paracetamol Demo 500 mg con el ingrediente Paracetamol (aunque cambies la concentración a 250): aviso rojo «Ya existe una formulación con los datos ingresados» y te quedas en el formulario. Para cambiar la concentración, edita la fila existente.

Crea las otras tres formulaciones de la tabla 7.1. Si no ves **Nuevo**, te falta `Formulaciones Clínicas › Crear`.

Errores al guardar:

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Ya existe una formulación con los datos ingresados» | Ese medicamento ya tiene ese ingrediente (comprobado) | Edita la formulación existente |
| «Error al guardar la formulación clínica» | El servidor rechazó el guardado (medicamento, ingrediente o unidad que ya no existen, u otro problema) | Revisa los datos e inténtalo de nuevo (No verificado en la aplicación: no se pudo provocar) |

### 7.4 Buscar, filtrar y exportar

Comprobado con las cuatro formulaciones de la tabla 7.1:

- **Buscar** sin filtros: «Resultados (4)».
- «Medicamento» = `demo 500`: Amoxicilina y Paracetamol. «Ingrediente Activo» = `amox`: solo Amoxicilina. «Medicamento» = `clonaz` junto con «Ingrediente Activo» = `clonaz`: solo Clonazepam (los filtros se combinan). «Medicamento» = `zzz`: «Resultados (0)» y la fila vacía dice «Resultados».
- **Limpiar** vacía los filtros y oculta la tabla.
- **Exportar** genera `formulaciones-clinicas.xlsx`. Comprobado: hoja «Data» con las columnas «Medicamento», «Ingrediente Activo», «Concentración», «Unidad de Medida» y «Código TE Orange», y una fila por formulación (la columna del código queda vacía si no tiene). Si falla la búsqueda o la exportación: «Error al buscar formulaciones clínicas».

### 7.5 Ver, editar y eliminar una formulación

1. Haz clic en el nombre del medicamento en la tabla: se abre el detalle con los cinco campos deshabilitados y el botón **Editar** (permiso `Actualizar`).
2. **Editar** habilita todo **menos el medicamento**, que no se puede cambiar, y muestra **Cancelar**, **Guardar** y **Eliminar** (permiso `Eliminar`). **Cancelar** descarta lo escrito y vuelve al modo ver (comprobado: se escribió 250 y la concentración siguió en 500).
3. Cambia la concentración a `250.5` y haz clic en **Guardar**: «Formulación clínica guardada correctamente» y **vuelves a la búsqueda**, donde la fila muestra 250.5 (comprobado: acepta decimales). Se volvió a poner 500.
4. Un medicamento puede tener varios ingredientes: se creó una segunda formulación de Paracetamol Demo 500 mg con el ingrediente Ibuprofeno (100 Miligramo) y aparecieron dos filas del mismo medicamento. Esa formulación de prueba se eliminó al terminar.
5. **Eliminar** abre «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar esta formulación clínica?» y **Cancelar** / **Confirmar**. **Cancelar** no borra nada (comprobado). Al confirmar, la formulación desaparece de la lista y vuelves a la búsqueda (comprobado; el aviso verde «Formulación clínica eliminada correctamente» no alcanzó a leerse en la prueba). La eliminación no comprueba dependencias: una formulación siempre se puede eliminar. Cualquier fallo muestra «Error al eliminar la formulación clínica». Si el detalle no abre: «Error al cargar la formulación clínica».

**Un ingrediente en una formulación no se puede eliminar.** Mientras exista una formulación con «Paracetamol», el ingrediente activo «Paracetamol» del capítulo 4 queda bloqueado para eliminar. El aviso esperado es «No puedes eliminar este ingrediente activo porque está incluido en formulaciones clínicas.» (ver sección 10.2).

## 8. Proveedores y precios de compra

**Cómo llegar:** menú lateral › CATÁLOGOS › **Proveedores** (`/proveedores`).

### 8.1 Qué es un proveedor

Un **proveedor** es a quien le compras: tiene una razón social, un RUC (o identificador tributario), una **tasa de cumplimiento** (de 0 a 100, qué tan puntual y completo te entrega) y, si quieres, una calificación ESG y unos términos financieros. En su ficha guardas los **precios de compra** que te cobra por cada producto, con su historial. Se crea un proveedor ficticio:

| Razón Social | RUC / Tax ID | Tasa de Cumplimiento | Calificación ESG | Términos Financieros |
|---|---|---|---|---|
| Distribuidora Demo Salud S.A.C. | `20611122233` | 95 | 80 | Pago a 30 días |

### 8.2 La pantalla de búsqueda

Título «Proveedores», migas «Inicio / Catálogos / Proveedores» y tarjeta «Búsqueda de Proveedores». Arriba a la derecha, **Carga Masiva** y **Nuevo** (con permiso `Crear`). Filtros:

| Filtro | Qué hace |
|---|---|
| Razón Social | Texto con lupa. Busca razones sociales que **contengan** lo que escribas |
| RUC / Tax ID | Texto con lupa. «Contiene» |
| Tasa de Cumplimiento | Deslizador de rango con dos tiradores. La línea dice «Tasa de Cumplimiento: 0 — 100» y cambia al moverlo. Va de 0 a 100 en pasos de 1; solo filtra el límite que moviste |

La tabla tiene tres columnas: **Razón Social** (enlace al detalle), **RUC / Tax ID** y **Tasa de Cumplimiento**. Fila sin resultados: «Resultados» (no dice «Sin resultados»). Debajo, **Exportar** y el paginador.

Comprobado con el proveedor creado y uno de prueba (tasa 60): **Buscar** da «Resultados (2)»; «Razón Social» = `demo` da uno; «RUC / Tax ID» = `2070` da el de prueba; con el deslizador en 80 — 100 sale solo el de tasa 95; en 0 — 70 sale el de tasa 60; en 50 — 70 sale el de 60; «Razón Social» = `zzz` da «Resultados (0)». **Limpiar** vacía los textos, devuelve el deslizador a 0 — 100 y oculta la tabla.

**Exportar** genera `proveedores.xlsx`. Comprobado: hoja «Data» con las columnas «Razón Social», «RUC / Tax ID» y «Tasa de Cumplimiento» y una fila por proveedor. No incluye ESG ni términos financieros. Si falla la búsqueda o la exportación: «Error al buscar proveedores».

### 8.3 Crear un proveedor

1. Haz clic en **Nuevo** (`/proveedores/nueva`).
2. Escribe la «Razón Social» y el «RUC / Tax ID».
3. Revisa la «Tasa de Cumplimiento» (empieza en 100).
4. Si quieres, escribe la «Calificación ESG» y los «Términos Financieros».
5. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Razón Social | El nombre legal del proveedor | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 200 caracteres. No puede repetirse (tampoco con otras mayúsculas): «Ya existe un proveedor con esa razón social.» |
| RUC / Tax ID | El RUC u otro identificador tributario | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 30 caracteres. SOFIA no comprueba que sea un RUC válido. No puede repetirse: «Ya existe un proveedor con ese identificador tributario.» |
| Tasa de Cumplimiento | Un número de 0 a 100 | Sí | Empieza en 100. Vacío o fuera de rango (por ejemplo 150): «Este campo es obligatorio» |
| Calificación ESG | Un número de 0 a 100 (criterios ambientales, sociales y de gobierno) | No | Fuera de rango: **no muestra ningún mensaje**, pero **Guardar** no envía nada. Corrige el valor o bórralo |
| Términos Financieros | Condiciones de pago, texto libre | No | El campo deja escribir hasta 500 caracteres, pero el servidor solo admite **100**: con más, aparece «Error al guardar el proveedor» (comprobado con 150 caracteres) |

**Probar los errores (comprobado en la aplicación):**

1. **Guardar** con el formulario vacío: «Este campo es obligatorio» bajo «Razón Social» y «RUC / Tax ID».
2. Escribe una tasa de `150`: «Este campo es obligatorio» bajo «Tasa de Cumplimiento».
3. Con tasa 95 y «Calificación ESG» `120`, **Guardar** no hace nada y no muestra ningún mensaje.
4. Con ESG 80 y términos «Pago a 30 días»: aviso «Proveedor guardado correctamente» y vuelves a la búsqueda.
5. Intenta crear «Otra Razon Social S.A.C.» con el mismo RUC: «Ya existe un proveedor con ese identificador tributario.» Intenta crear «distribuidora demo salud s.a.c.» con otro RUC: «Ya existe un proveedor con esa razón social.»
6. Escribe 150 caracteres en «Términos Financieros»: «Error al guardar el proveedor». Escribe solo espacios en «Razón Social»: el mismo aviso.

Si no ves **Nuevo**, te falta `Proveedores › Crear`.

Errores al guardar:

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Ya existe un proveedor con ese identificador tributario.» | Otro proveedor tiene ese RUC (comprobado) | Abre el existente |
| «Ya existe un proveedor con esa razón social.» | Otro proveedor tiene esa razón social (comprobado) | Abre el existente |
| «Error al guardar el proveedor» | El servidor rechazó el guardado: términos de más de 100 caracteres o textos de solo espacios (comprobado) | Acorta el texto o escribe un valor real |
| Los tres avisos de base de datos del capítulo 4 (sección 5.3) | Errores poco frecuentes | Igual que allí (No verificado en la aplicación) |

### 8.4 Ver, editar y eliminar un proveedor

1. Haz clic en la razón social en la tabla. El detalle tiene dos tarjetas: arriba los cinco datos (deshabilitados) con el botón **Editar**, y abajo **Precios de compra** (sección 8.5).
2. **Editar** habilita los cinco campos, **oculta** la tarjeta de precios y muestra **Cancelar**, **Guardar** y **Eliminar** (permiso `Eliminar`). Al guardar: «Proveedor guardado correctamente» y **te quedas** en el detalle, otra vez en modo ver (comprobado: tasa de 60 a 65).
3. **Eliminar** abre «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este proveedor?» y **Cancelar** / **Confirmar**. Al confirmar: «Proveedor eliminado correctamente» y vuelves a la búsqueda (comprobado con un proveedor de prueba sin precios, «Proveedor Prueba Borrar S.A.C.», que ya no existe).
4. **Un proveedor con precios de compra registrados no se puede eliminar**: el aviso es «No puedes eliminar este proveedor porque tiene precios de compra registrados.» Como ninguna pantalla borra precios, un proveedor al que ya registraste un precio queda para siempre. (No verificado en la aplicación: no se intentó eliminar a «Distribuidora Demo Salud S.A.C.» para no perder los datos del manual; viene de 19d §6.) Otro fallo: «Error al eliminar el proveedor». Si el detalle no abre: «Error al cargar el proveedor».

La carga masiva de proveedores se describe en la sección 10.5.

### 8.5 Precios de compra

En el detalle (modo ver) de «Distribuidora Demo Salud S.A.C.» está la tarjeta **Precios de compra**, con el texto «Costo por unidad base que te cobra este proveedor, con su descuento ya aplicado. Cada producto tiene un solo precio vigente y los anteriores quedan en el historial.», el botón **Registrar precio** (solo con el permiso `Registrar precio de proveedor` del módulo Proveedores) y una tabla:

| Columna | Qué muestra |
|---|---|
| Producto | El nombre comercial del medicamento |
| Costo por unidad base | «S/» y el **costo neto** con cuatro decimales (ya con el descuento aplicado) |
| Vigente desde | La fecha en que se registró el precio, `dd/MM/aaaa` |
| Vigente hasta | La fecha en que fue reemplazado o, si es el actual, el texto «Vigente» |

Sin precios la tabla dice «Este proveedor aún no tiene precios registrados.». Si no se pueden cargar, el aviso es «Error al cargar los precios del proveedor» (No verificado en la aplicación: no se pudo provocar). Debajo hay un paginador propio (5, 10 o 25 filas; empieza en 10). Los precios se ordenan del más reciente al más antiguo. La tabla no tiene enlaces ni botones por fila: **no puedes editar ni borrar un precio registrado.**

**Registrar un precio:**

1. Haz clic en **Registrar precio**. Se abre el cuadro «Registrar precio de compra» con el texto «El nuevo precio reemplaza al vigente de este producto con este proveedor; el anterior queda en el historial.»
2. En «Producto» escribe parte del nombre (aparece el placeholder «Escribe para buscar un producto») y **elige una opción de la lista**.
3. Escribe el «Precio de compra por unidad base (S/)».
4. Revisa el «Descuento (%)» (empieza en 0).
5. Mira el «Costo neto por unidad base», que se calcula en pantalla mientras escribes.
6. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Producto | El medicamento que te vende este proveedor | Sí | Debes elegir una opción de la lista; texto libre no vale: «Selecciona un producto de la lista» |
| Precio de compra por unidad base (S/) | Lo que te cuesta **una unidad base** antes del descuento (hasta 4 decimales) | Sí | Vacío: «Este campo es obligatorio». Cero o negativo: «Debe ser mayor que 0». Mayor que 9 999 999,9999: «El precio es demasiado alto» |
| Descuento (%) | El descuento que te hace el proveedor, de 0 a 100 | Sí | Vacío: «Este campo es obligatorio». Fuera de rango: «Debe estar entre 0 y 100» |
| Costo neto por unidad base | No se escribe: es el resultado `precio × (1 − descuento ÷ 100)`, redondeado a 4 decimales | — | Muestra «—» mientras el precio o el descuento no sean válidos |

El campo «Precio de compra por unidad base» es por **unidad base**: si el proveedor te vende una caja de 100 unidades a S/ 30, escribe S/ 0.30 (30 ÷ 100). Según el inventario, con **Enter** no se guarda: usa el botón **Guardar** (No verificado en la aplicación).

**Qué pasa al guardar:** aparece «Precio registrado correctamente», el cuadro se cierra y la tabla se actualiza desde la primera página. El sistema toma como inicio de vigencia el momento en que guardas y no puedes elegir otra fecha.

**Lo que se registró en la prueba (comprobado):**

| Paso | Producto | Precio | Descuento | Costo neto | Resultado en la tabla |
|---|---|---|---|---|---|
| 1 | Paracetamol Demo 500 mg | S/ 0.30 | 10 % | S/ 0.27 | «S/ 0.2700», «Vigente» |
| 2 | Amoxicilina Demo 500 mg | S/ 0.80 | 5 % | S/ 0.76 | «S/ 0.7600», «Vigente» |
| 3 | Paracetamol Demo 500 mg (segundo precio) | S/ 0.35 | 0 % | S/ 0.35 | «S/ 0.3500», «Vigente»; la fila anterior de Paracetamol (S/ 0.2700) quedó con «Vigente hasta» igual a la fecha del cambio |

Es decir: **cada producto tiene un solo precio vigente con cada proveedor**. Al registrar uno nuevo, el sistema cierra el anterior en ese mismo instante y lo deja en el historial; ves las dos filas, la nueva arriba. Otros comprobados: con precio 1.2345 y descuento 33.33 el cuadro mostró «S/ 0.823» como costo neto (redondeo a 4 decimales).

**Probar los errores del cuadro (comprobado):**

1. **Guardar** con el cuadro vacío: «Selecciona un producto de la lista» y «Este campo es obligatorio» (bajo el precio). El cuadro sigue abierto.
2. Escribe `Paracetamol` en «Producto» sin elegir la opción y precio `0`, descuento `101`: «Selecciona un producto de la lista», «Debe ser mayor que 0» y «Debe estar entre 0 y 100».
3. Precio `10000000`: «El precio es demasiado alto». Precio `-1`: «Debe ser mayor que 0». Precio y descuento vacíos: «Este campo es obligatorio» en ambos.
4. **Cancelar** cierra el cuadro sin registrar nada; según el inventario también lo cierran la tecla Esc y el clic fuera (No verificado en la aplicación).

Otros avisos posibles al guardar (No verificado en la aplicación: no se pudieron provocar): «El proveedor ya no existe.», «El producto seleccionado ya no existe.», los tres avisos de base de datos del capítulo 4 y «Error al registrar el precio». Si el producto no aparece en la lista porque no tienes `Medicamentos › Leer`, la lista queda vacía sin aviso.

## 9. Aseguradoras

**Cómo llegar:** menú lateral › CATÁLOGOS › **Aseguradoras** (`/aseguradoras`). El título de la cabecera de estas pantallas es «Aseguradoras Médicas»; el menú y las migas dicen «Aseguradoras». En la pantalla de roles, el módulo de permisos se llama **Seguros**.

### 9.1 Qué es una aseguradora y para qué sirve hoy

Una **aseguradora** es una empresa de seguros médicos con la que podrías trabajar. En SOFIA se guarda con dos datos: su **nombre comercial** y su **código identificador nacional**.

**Hoy, el punto de venta no usa las aseguradoras.** El bloque «Cobertura de Seguro (opcional)» del punto de venta (donde se elegiría la aseguradora y el monto que cubre el seguro) está construido pero **oculto**: el código lo deja apagado con una constante fija (`mostrarAseguradora = false` en `nueva-venta.component.ts:215`, y el bloque va dentro de `@if (mostrarAseguradora)` en `nueva-venta.component.html:456`) y ningún rol ni configuración lo activa. Por eso, en esta versión, registrar aseguradoras no cambia nada al vender: el cajero no ve ninguna opción de seguro y todas las ventas se cobran completas al cliente. Registra tus aseguradoras si quieres tener el catálogo listo, pero no cuentes con descuentos de seguro en la venta todavía.

Se crea una aseguradora ficticia:

| Nombre Comercial | Código Identificador Nacional |
|---|---|
| Seguros Demo Vida S.A. | `ASEG-DEMO-001` |

### 9.2 La pantalla de búsqueda

Título «Aseguradoras Médicas», migas «Inicio / Catálogos / Aseguradoras» y tarjeta «Búsqueda de Aseguradoras». Arriba a la derecha, **Carga Masiva** y **Nuevo** (con permiso `Crear` de Seguros). Filtros:

| Filtro | Qué hace |
|---|---|
| Nombre Comercial | Texto con lupa. Busca nombres que **contengan** lo que escribas |
| Código Identificador Nacional | Texto con lupa. «Contiene» |

La tabla tiene dos columnas: **Nombre Comercial** (enlace al detalle) y **Código Identificador Nacional**. La lista no tiene un orden garantizado. Fila sin resultados: «Resultados». Debajo, **Exportar** y el paginador.

Comprobado con dos aseguradoras: **Buscar** da «Resultados (2)»; «Nombre Comercial» = `vida` da una; «Código Identificador Nacional» = `prueba` da la otra; «Nombre Comercial» = `zzz` da «Resultados (0)». **Exportar** genera `aseguradoras.xlsx`; comprobado: hoja «Data» con las columnas «Nombre Comercial» y «Código Identificador Nacional» y una fila por aseguradora. Si falla la búsqueda o la exportación: «Error al buscar aseguradoras».

### 9.3 Crear una aseguradora

1. Haz clic en **Nuevo** (`/aseguradoras/nueva`).
2. Escribe el «Nombre Comercial» y el «Código Identificador Nacional».
3. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Nombre Comercial | El nombre con el que se conoce a la aseguradora | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 150 caracteres. Puede repetirse |
| Código Identificador Nacional | El código oficial o interno que la identifica | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 50 caracteres. No puede repetirse (tampoco con otras mayúsculas), pero **el aviso no lo dice** (ver abajo) |

**Probar los errores (comprobado en la aplicación):**

1. **Guardar** con el formulario vacío: «Este campo es obligatorio» bajo los dos campos.
2. Guarda «Seguros Demo Vida S.A.» con el código `ASEG-DEMO-001`: aviso «Aseguradora guardada correctamente» y vuelves a la búsqueda.
3. Intenta crear otra aseguradora con el código `aseg-demo-001`: aviso rojo **«Error al guardar la aseguradora»** y te quedas en el formulario. Este mismo aviso aparece ante **cualquier** error de guardado, también el de un código repetido: no existe un «ya existe» para aseguradoras. Si lo ves, revisa primero que el código no esté ya registrado.

Si no ves **Nuevo**, te falta `Seguros › Crear`.

### 9.4 Ver, editar y eliminar

1. Haz clic en el nombre en la tabla: se abre el detalle con los dos campos deshabilitados y el botón **Editar** (permiso `Seguros › Actualizar`).
2. **Editar** habilita los campos y muestra **Cancelar**, **Guardar** y **Eliminar** (permiso `Eliminar`). **Cancelar** descarta los cambios (comprobado). **Guardar** muestra «Aseguradora guardada correctamente» y **te quedas** en el detalle, otra vez en modo ver, con los datos nuevos (comprobado).
3. **Eliminar** abre «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar esta aseguradora?» y **Cancelar** / **Confirmar**. **Cancelar** no borra nada. Al confirmar: «Aseguradora eliminada correctamente» y vuelves a la búsqueda (comprobado con una aseguradora de prueba, «Aseguradora Prueba Borrar», que ya no existe). SOFIA no comprueba dependencias al eliminar una aseguradora.
4. Cualquier fallo al eliminar muestra «Error al eliminar la aseguradora». Si el detalle no abre: «Error al cargar la aseguradora».

La carga masiva de aseguradoras está en la sección 10.6.

## 10. Eliminar algo que está en uso y carga masiva

### 10.1 Eliminar un medicamento o una unidad en uso

SOFIA no deja eliminar un registro del que dependen otros. Comprobado con los datos de este capítulo:

| Intento | Qué se hizo | Aviso rojo que apareció |
|---|---|---|
| Eliminar el medicamento «Paracetamol Demo 500 mg» | Abrir el medicamento, **Editar**, **Eliminar** y **Confirmar**. Tiene jerarquías, presentaciones, una formulación y precios de proveedor | «No puedes eliminar este medicamento porque tiene lotes, presentaciones u otros registros asociados.» Te quedas en la ficha y el medicamento no se borra |
| Eliminar la unidad `UND` (Unidad) | Igual, en Unidades de Medida. La usan los medicamentos como unidad base | «No se puede eliminar la unidad de medida porque está asociada a medicamentos o jerarquías de unidades.» |
| Eliminar la unidad `FRA` (Frasco) | Con una jerarquía de prueba de Tramadol que usaba el Frasco como unidad mayor | El mismo aviso |

Qué bloquea la eliminación de un medicamento: lotes, presentaciones de venta, jerarquías de unidades, formulaciones clínicas, precios de proveedor, servicios de agenda, servicios de inmunización y órdenes de producción magistral. El aviso es siempre el mismo, así que no te dice cuál es la causa. Para poder eliminarlo debes quitar primero esos registros (por ejemplo, eliminar sus jerarquías, presentaciones y formulaciones). Los **precios de proveedor** no se pueden quitar desde ninguna pantalla: un medicamento con precios registrados no se podrá eliminar.

En cambio, un medicamento sin ninguna dependencia sí se elimina. Se creó «Medicamento Prueba Borrar» (código `DEMO-MED-9999`) y se eliminó con **Editar**, **Eliminar** y **Confirmar**: aviso «Medicamento eliminado correctamente» y vuelves a la búsqueda; el medicamento deja de aparecer.

Resumen de lo que se creó y eliminó como prueba en cada pantalla de este capítulo (no queda ninguno):

| Pantalla | Registro de prueba | Aviso al eliminarlo |
|---|---|---|
| Medicamentos | «Medicamento Prueba Borrar» (`DEMO-MED-9999`) | «Medicamento eliminado correctamente» |
| Jerarquías de unidades | Tramadol Demo 50 mg, Frasco → Unidad, 20 | «Jerarquía eliminada correctamente» |
| Presentaciones de venta | Tramadol Demo 50 mg, Frasco, S/ 60.00 (desde el cuadro del medicamento) | «Presentación de venta eliminada correctamente» |
| Formulaciones clínicas | Paracetamol Demo 500 mg con Ibuprofeno, 100 Miligramo | La fila desapareció (el aviso no se alcanzó a leer) |
| Proveedores | «Proveedor Prueba Borrar S.A.C.» (`20700000001`) | «Proveedor eliminado correctamente» |
| Aseguradoras | «Aseguradora Prueba Borrar» (`ASEG-PRUEBA-9`) | «Aseguradora eliminada correctamente» |

### 10.2 Eliminar un ingrediente en una formulación o un laboratorio con medicamentos

Dos pruebas anunciadas en el capítulo 4 **no se pudieron recorrer en la aplicación**: eliminar un laboratorio que tiene medicamentos y eliminar un ingrediente activo incluido en una formulación clínica. El sistema de permisos del entorno de pruebas bloqueó las dos acciones (eran eliminaciones definitivas de datos del catálogo) y no se buscó ningún atajo. Los avisos esperados, tomados del inventario de pantallas (`19c-catalogos-1.md` §3.2, §4.2 y §6), son:

| Intento | Aviso esperado |
|---|---|
| Eliminar «Laboratorio Demo Andino» (tiene medicamentos) | «No se puede eliminar el laboratorio porque tiene medicamentos asociados.» |
| Eliminar el ingrediente «Paracetamol» (está en una formulación) | «No puedes eliminar este ingrediente activo porque está incluido en formulaciones clínicas.» |

(No verificado en la aplicación: motivo arriba.)

### 10.3 Carga masiva de medicamentos

La carga masiva te permite crear muchos medicamentos de una sola vez con un Excel. El flujo es el mismo de los otros catálogos (capítulo 4, sección 9): descargar la plantilla, subir el archivo, revisar la vista previa y guardar solo las filas válidas.

**Cómo llegar:** `Medicamentos` › **Carga Masiva** (`/medicamentos/carga-masiva`), con el permiso `Medicamentos › Crear`. La pantalla se titula «Carga Masiva», con las migas «Inicio / Catálogos / Medicamentos / Carga Masiva», el botón **Descargar plantilla** y una zona que dice «Arrastra tu archivo Excel aquí o haz clic para seleccionar» y «Solo archivos .xlsx (máx. 5 MB y 5000 filas)».

**La plantilla.** **Descargar plantilla** baja el archivo `plantilla-medicamentos.xlsx`. Comprobado: tiene una hoja llamada «Plantilla» con solo la fila de encabezados, estas cinco columnas en este orden y **sin filas de ejemplo**:

| Columna del Excel | Qué escribir | Obligatoria | Detalle |
|---|---|---|---|
| `CodigoNacional` | El código del medicamento | Sí | Hasta 50 caracteres. No debe existir ya entre tus medicamentos (sin distinguir mayúsculas) ni repetirse en el archivo |
| `NombreComercial` | El nombre comercial | Sí | Hasta 150 caracteres. Puede repetirse |
| `Laboratorio` | El **nombre de compañía** de un laboratorio que ya exista | Sí | Se busca por nombre, sin distinguir mayúsculas. Debe existir y ser único |
| `UnidadBase` | La **descripción** de una unidad de medida que ya exista (por ejemplo `Unidad`, no `UND`) | Sí | Se busca por descripción |
| `CondicionVenta` | Uno de estos textos: `Venta Libre (OTC)`, `Receta Simple`, `Receta Retenida` o `Estupefaciente` | Sí | No distingue mayúsculas |

**No hay columna de precio.** Los medicamentos cargados por Excel quedan **sin precio de venta base**. Abre cada uno, haz clic en **Editar**, escribe el precio y guarda; mientras no tenga precio, el formulario de edición no te deja guardar otros cambios (el campo muestra «El precio no puede ser negativo»). Según el glosario, un medicamento sin precio no se puede vender sin presentación.

**Recorrido (comprobado en la aplicación).** El navegador integrado no tiene herramienta para elegir archivos, así que se entregó a la pantalla un `.xlsx` construido para esta prueba, con estas cinco filas:

| Fila | Código | Nombre | Laboratorio | Condición | Qué se esperaba |
|---|---|---|---|---|---|
| 1 | `DEMO-MED-0005` | Ibuprofeno Demo 400 mg | Laboratorio Demo Andino | Venta Libre (OTC) | Válida |
| 2 | `DEMO-MED-0006` | Omeprazol Demo 20 mg | Laboratorio Demo Pacífico | Receta Simple | Válida |
| 3 | `DEMO-MED-0007` | Producto Con Error | Laboratorio Inexistente | Receta Simple | Error: laboratorio no existe |
| 4 | `DEMO-MED-0008` | Producto Condicion Mala | Laboratorio Demo Andino | Libre | Error: condición no válida |
| 5 | `DEMO-MED-0001` | Paracetamol Repetido | Laboratorio Demo Andino | Venta Libre (OTC) | Error: el código ya existe |

La vista previa mostró «Vista previa», «5 filas totales», «2 filas válidas» y «3 filas con error», y una tabla con las columnas «Código Nacional», «Nombre Comercial», «Laboratorio», «Unidad Base», «Condición de Venta», una columna para el error y «Estado» (un icono de visto bueno para las válidas y un icono de error para las otras). Los textos de error, tal cual:

- «El valor 'Laboratorio Inexistente' del campo 'laboratorio' no existe en el sistema»
- «'Libre' no es un valor válido para 'condicionVenta'»
- «El valor 'DEMO-MED-0001' del campo 'codigoNacional' ya existe en el sistema»

Al final aparece el botón **Guardar válidos (2)**. Al hacer clic: aviso «Se guardaron 2 registros correctamente», y te lleva a la búsqueda de medicamentos, que ahora muestra «Resultados (6)». Las filas con error **no se guardan**. Después se abrió «Ibuprofeno Demo 400 mg» y «Omeprazol Demo 20 mg» y se les puso precio (S/ 0.40 y S/ 0.90).

Otros errores que puede mostrar la vista previa (de la referencia `20-carga-masiva-y-exportacion.md` §3.6; No verificado en la aplicación): campo vacío (el campo es obligatorio), texto demasiado largo en código o nombre, laboratorio o unidad **ambiguos** (existe más de uno con ese nombre) o no encontrados, y el mismo código repetido en el archivo (todas las filas repetidas quedan con error, también la primera).

Cosas que debes saber:

- Si entre la vista previa y el guardado cambia algo (por ejemplo otro usuario crea el mismo código), el guardado **omite en silencio** esas filas.
- Los errores generales del archivo (que no sea `.xlsx`, que pese más de 5 MB o que tenga más de 5000 filas) y de guardado se explican en el capítulo 4, sección 8.5 (No verificado en la aplicación los de tamaño).

### 10.4 Carga masiva de jerarquías de unidades

**Cómo llegar:** `Jerarquías de Unidades` › **Carga Masiva** (`/jerarquias/carga-masiva`), con `Jerarquías de Unidades › Crear`. Plantilla comprobada: hoja «Plantilla» con las columnas `Producto` (nombre comercial de un medicamento existente), `UnidadMayor` y `UnidadMenor` (descripciones de unidades existentes, distintas entre sí) y `Multiplicador` (número mayor que cero, con **punto** decimal). Todas son obligatorias.

Prueba de la vista previa (comprobada; no se guardó): con cinco filas, la pantalla dijo «2 filas válidas» y «3 filas con error». Fueron válidas una jerarquía nueva de Tramadol (Caja → Unidad, 20) **y una jerarquía que ya existía** (Paracetamol, Caja → Blíster, 10). Con error: «'unidadMenor' no puede ser igual a 'unidadMayor'», «'multiplicador' debe ser mayor que cero» y «El valor 'Producto Inexistente' del campo 'producto' no existe en el sistema».

**Cuidado:** la vista previa **no detecta** jerarquías repetidas ni contradictorias. Al guardar, esas filas se omiten en silencio y el aviso cuenta solo las que sí se guardaron (referencia §3.4; No verificado en la aplicación el guardado).

### 10.5 Carga masiva de proveedores

**Cómo llegar:** `Proveedores` › **Carga Masiva** (`/proveedores/carga-masiva`), con `Proveedores › Crear`. Plantilla comprobada: hoja «Plantilla» con las columnas `RazonSocial`, `TaxId`, `TerminosFinancieros`, `CalificacionEsg` y `TasaCumplimiento`. Son obligatorias `RazonSocial`, `TaxId` y `TasaCumplimiento`; en la vista previa las etiquetas son «Razón Social», «Tax ID» (no «RUC / Tax ID»), «Términos Financieros», «Calificación ESG» y «Tasa de Cumplimiento».

Prueba de la vista previa (comprobada; no se guardó): con cinco filas dijo «2 filas válidas» y «3 filas con error». Errores vistos: «El valor '20611122233' del campo 'taxId' ya existe en el sistema», «'calificacionEsg' debe estar entre 0 y 100», «'tasaCumplimiento' debe ser un número decimal válido» y «El campo 'tasaCumplimiento' es obligatorio». Una fila con la razón social de un proveedor existente pero otro RUC apareció **como válida**: la vista previa no revisa la razón social, y al guardar esa fila se omitiría en silencio (referencia §3.9; No verificado en la aplicación el guardado).

Límites que aplica el guardado (no la vista previa): términos financieros de hasta 100 caracteres y RUC de hasta 50. Los decimales se escriben con punto.

### 10.6 Carga masiva de aseguradoras

**Cómo llegar:** `Aseguradoras` › **Carga Masiva** (`/aseguradoras/carga-masiva`), con `Seguros › Crear`. Plantilla comprobada: hoja «Plantilla» con las columnas `NombreComercial` y `CodigoIdentificadorNacional`, ambas obligatorias. Según el inventario, el archivo se baja como `plantilla-aseguradoras.xlsx` (No verificado en la aplicación: el nombre en disco).

Prueba de la vista previa (comprobada; no se guardó): con cinco filas dijo «1 filas válidas» y «4 filas con error». Errores vistos: «El valor 'ASEG-DEMO-001' del campo 'codigoIdentificadorNacional' ya existe en el sistema», «El valor 'ASEG-REP-1' del campo 'codigoIdentificadorNacional' está duplicado en el archivo» (en **las dos** filas repetidas, incluida la primera, sin distinguir mayúsculas) y «El campo 'nombreComercial' es obligatorio».

**Cuidado con esta carga:** a diferencia de las otras, el guardado de aseguradoras **no omite duplicados**. Si entre la vista previa y el guardado aparece un código repetido, el guardado entero falla con «Error al guardar los registros» y no se guarda ninguna fila (referencia §3.1; No verificado en la aplicación). El mensaje de longitud del nombre en la vista previa dice 200 caracteres aunque el límite real es 150.

## 11. Lo que ve un cajero frente a un administrador

Se entró al sistema con el cajero del manual (`carlos.cajero`, rol «Cajero»), que solo tiene `Medicamentos › Leer` y `Aseguradoras (Seguros) › Leer` entre las pantallas de este capítulo (capítulo 3, sección del rol «Cajero»).

| Qué | Administrador | Cajero (comprobado) |
|---|---|---|
| Menú CATÁLOGOS | Ve las nueve opciones | Solo ve **Medicamentos** y **Aseguradoras** (no ve Jerarquías de Unidades, Presentaciones de Venta, Formulaciones Clínicas ni Proveedores) |
| Búsqueda de medicamentos | Filtros, tabla, **Exportar**, **Carga Masiva** y **Nuevo** | Los mismos filtros y la tabla con los seis medicamentos, y **Exportar**; **no ve** **Carga Masiva** ni **Nuevo** |
| Detalle de un medicamento | **Presentaciones de Venta** y **Editar** | Ve los datos, incluido el «Precio de Venta Base», y la tarjeta «Stock por Sucursal»; **no ve** **Presentaciones de Venta** ni **Editar** (solo la flecha de volver) |
| Búsqueda y detalle de aseguradoras | Todo | Ve la lista (una fila), **Exportar** y el detalle de solo lectura; sin **Carga Masiva**, **Nuevo** ni **Editar** |
| Pantallas de jerarquías, presentaciones, formulaciones y proveedores | Acceso completo | Sin acceso. Si escribe la dirección a mano, el sistema lo devuelve al Inicio con «No tienes permisos para acceder a esta pantalla.» (No verificado en la aplicación con estas pantallas; viene del inventario 19d §0) |
| Precios de compra de proveedores | Los ve y los registra (con `Registrar precio de proveedor`) | No los ve: no tiene `Proveedores › Leer` |

El cajero usa los medicamentos que el administrador registró (con su condición de venta y su precio) y, en el punto de venta, las presentaciones de venta que hayan quedado creadas.

## 12. Diferencias con la documentación técnica

Se compararon `19d-catalogos-2.md`, `19c-catalogos-1.md` (jerarquías), `20-carga-masiva-y-exportacion.md` y `01b-catalogos.md` con lo que hace la aplicación. Coincidieron en todo lo comprobado, con estos matices y confirmaciones:

| # | Documento y apartado | Dice | La aplicación muestra |
|---|---|---|---|
| 1 | 19d §1.3 (observaciones) y 20 §3.6 | La carga masiva de medicamentos no trae el precio, mientras el formulario lo exige | Confirmado: los medicamentos cargados quedan con el precio vacío, y al editarlos el campo exige un valor antes de guardar |
| 2 | 19d §1.2 y 01b (precio) | El precio de venta base tiene dos decimales y no puede ser negativo | Confirmado, y además un precio con tres decimales (`0.505`) se guarda **redondeado a dos** (`0.51`). Ningún documento lo menciona |
| 3 | 19d §2.2 (campos de la presentación) | La descripción la copia el servidor de la unidad elegida | Confirmado: la presentación se llama «Caja» o «Blíster»; no se puede poner un nombre como «Caja x100» |
| 4 | 19d §2.2 y §2.4 | La pantalla completa lista todas las unidades vendibles; el cuadro del medicamento solo las que faltan | Confirmado: en la pantalla completa se puede elegir una unidad que ya tiene presentación y el error aparece al guardar |
| 5 | 19d §2.2 (producto sin jerarquía) | **Guardar** sale sin enviar y sin error visible | Confirmado: no pasa nada y no hay ningún mensaje |
| 6 | 19c §2.2 (consistencia de jerarquías) | Una conversión contradictoria da el aviso genérico | Confirmado: «Error al guardar la jerarquía», tanto para Caja → Unidad = 50 (contradice 10 × 10) como para Blíster → Caja |
| 7 | 19c §2.2 (editar jerarquía) | Editar con éxito navega a la búsqueda | Confirmado. También navega a la búsqueda al editar una presentación o una formulación; en cambio, medicamentos, proveedores y aseguradoras se quedan en el detalle |
| 8 | 19d §3.2 | Concentración cero o negativa muestra «Este campo es obligatorio» | Confirmado |
| 9 | 19d §4.2 | Calificación ESG fuera de rango bloquea **Guardar** sin mensaje | Confirmado |
| 10 | 19d §4.2 y 01b RN-PRV-003/004 | El formulario admite 30 caracteres de RUC y 500 de términos; la API 50 y 100 | Confirmado en los términos: con 150 caracteres sale «Error al guardar el proveedor». El RUC se cortó en 30 por el atributo del campo |
| 11 | 01b RN-PRV-018 a RN-PRV-020 (versión antes de 1.1) | El registro de precio «no guarda nada» | La aplicación **sí guarda** el precio, cierra el anterior y lo deja en el historial (como ya corrige 19d §4.3) |
| 12 | 19d §4.5 | El precio de compra se calcula con 4 decimales | Confirmado: 1.2345 con 33.33 % da «S/ 0.823» |
| 13 | 19d §5.2 | Un código de aseguradora repetido se ve como mensaje genérico | Confirmado: «Error al guardar la aseguradora» (no existe «ya existe» para aseguradoras) |
| 14 | 20 §3.4, §3.9 | La vista previa de jerarquías y de proveedores no detecta jerarquías repetidas ni razones sociales repetidas | Confirmado: ambas filas salieron como válidas en la vista previa |

## 13. Pendientes y «No verificado»

**En curso (sin commitear):** ninguno. Al empezar, los dos repositorios seguían en `sofia-apis` `43f5529` y `sofia-web-spa` `eeffbb6`, y `git status --short` no mostró cambios locales.

**No verificado en la aplicación** (cada caso, con su motivo):

| # | Qué | Motivo |
|---|---|---|
| 1 | Eliminar un laboratorio con medicamentos y un ingrediente activo incluido en una formulación | El sistema de permisos del entorno bloqueó las dos eliminaciones; no se buscó ningún atajo. Textos de 19c §3.2, §4.2 y §6 |
| 2 | Eliminar un proveedor con precios registrados («No puedes eliminar este proveedor porque tiene precios de compra registrados.») | No se intentó para no perder los datos del proveedor del manual. Texto de 19d §6 |
| 3 | Eliminar una presentación con ventas registradas, y eliminar un medicamento por cada una de sus ocho causas | Todavía no hay ventas, lotes ni servicios; el mensaje del medicamento se comprobó con jerarquías, presentaciones, formulación y precios |
| 4 | El comportamiento de cada condición de venta al cobrar | El punto de venta se recorre en capítulos posteriores. Viene del glosario y de `pos/es.json` |
| 5 | Contenido y descarga en disco de los archivos de **Exportar** y de las plantillas | El navegador integrado no muestra las descargas. Se leyó el contenido de cada archivo desde el navegador (columnas y filas) pero no el nombre final del archivo |
| 6 | El gesto de arrastrar o hacer clic para elegir el archivo en la carga masiva | No hay herramienta para elegir archivos; se entregó el `.xlsx` de prueba a la zona de carga desde el navegador |
| 7 | **Guardar válidos** de la carga masiva de jerarquías, proveedores y aseguradoras | Solo se probó la vista previa, para no crear datos de prueba que no se pueden borrar (proveedores) ni alterar el catálogo |
| 8 | Errores de archivo (más de 5 MB, más de 5000 filas, descompresión excesiva) y de guardado de la carga masiva | No se prepararon archivos de ese tipo. Ver capítulo 4, sección 12 |
| 9 | Avisos genéricos de carga o de búsqueda («Error al buscar …», «Error al cargar …»), «El proveedor ya no existe.», «El producto seleccionado ya no existe.», «Error al registrar el precio» y los tres avisos de base de datos | Son errores del servidor o de la red; no se pueden provocar desde la pantalla |
| 10 | Paginación real (más de una página) y los paginadores de 25 y 50 filas | Hay pocos registros |
| 11 | Esc y clic fuera del cuadro de precios; Enter dentro del cuadro | No se probaron; vienen de 19d §4.5 |
| 12 | Qué ve un usuario sin permiso de lectura de los catálogos auxiliares (por ejemplo, sin `Laboratorios › Leer` al crear un medicamento) | No se creó un rol de prueba para ello |
| 13 | Redirección por falta de permiso en las pantallas de este capítulo con el cajero | No se escribió la dirección a mano con la sesión del cajero (el navegador lo bloqueó en el capítulo 4) |
| 14 | Si cambiar una jerarquía recalcula las presentaciones ya guardadas | No se editó una jerarquía con presentaciones. El glosario dice que la equivalencia no se recalcula |
| 15 | Que el multiplicador de una jerarquía acepte decimales | No se probó; 19c §2.2 dice que el formulario permite teclear más de dos decimales |
| 16 | Los códigos ATC y los valores de «Código TE Orange» usados en los ejemplos | Son datos ficticios del manual; SOFIA no los valida |

**Hallazgos de la aplicación que conviene conocer** (no son errores del manual):

- Los avisos de error de **aseguradoras** no distinguen un código repetido: siempre «Error al guardar la aseguradora».
- «Error al guardar la jerarquía» no dice si el problema es una contradicción con otras jerarquías.
- Un medicamento con precios de proveedor no se puede eliminar nunca, y un proveedor con precios tampoco; ninguna pantalla borra precios.
- Los medicamentos cargados por Excel quedan sin precio y hay que editarlos uno por uno.
- El campo «Términos Financieros» del proveedor deja escribir 500 caracteres pero el servidor acepta 100.
- La calificación ESG fuera de rango y el guardado sin producto con jerarquía bloquean **Guardar** sin ningún mensaje.
- La pantalla de presentaciones permite elegir una unidad que el producto ya usa y solo avisa al guardar.
- La vista previa de la carga masiva de jerarquías y de proveedores da por válidas filas que luego se omiten en silencio al guardar.
- El mensaje de precio de un medicamento («El precio no puede ser negativo») aparece también cuando el campo está vacío.

## 14. Tabla de cobertura

Cada elemento de las fichas de `19d-catalogos-2.md` (Medicamentos §1, Presentaciones de venta §2, Formulaciones clínicas §3, Proveedores §4 con su diálogo de precio, Aseguradoras §5, y su §0 y §6 en lo que les toca) y de Jerarquías de unidades de `19c-catalogos-1.md` (§2), con el apartado de este capítulo. «V» significa recorrido en la aplicación. Todas las pantallas comparten estos elementos, que se explican una vez en el capítulo 4, sección 4: migas, subtítulo «Ingresa los criterios de búsqueda», «Resultados (N)», paginador (10, 25, 50), restauración de filtros al volver del detalle, modos ver y editar, cuadro «Confirmar eliminación» y avisos con **Cerrar**.

### 14.1 Medicamentos — búsqueda `/medicamentos` (19d §1.1)

| Elemento | Apartado |
|---|---|
| Título «Medicamentos», migas «Inicio / Catálogos / Medicamentos», tarjeta «Búsqueda de Medicamentos» | 4.3 (V) |
| Filtro «Código Nacional» (lupa, «contiene») | 4.3, 4.5 (V) |
| Filtro «Nombre Comercial» | 4.3, 4.5 (V) |
| Filtro «Laboratorio» con sugerencias («Buscar laboratorio...», «Sin resultados») | 4.3, 4.4, 4.5 (V) |
| Filtro «Unidad Base» con sugerencias («Buscar unidad de medida...») | 4.3, 4.5 (V) |
| Filtro «Condición de Venta» y sus cuatro opciones más «— Todos —» | 4.2, 4.3, 4.5 (V) |
| Tabla: «Código Nacional» (enlace), «Nombre Comercial», «Laboratorio», «Unidad Base», «Condición de Venta»; sin precio ni stock | 4.3 (V) |
| «Resultados (N)», «Sin resultados», paginador | 4.3, 4.5 (V; varias páginas: No verificado) |
| Botones **Carga Masiva**, **Nuevo**, **Limpiar**, **Buscar** | 4.3, 4.4, 4.5, 10.3 (V) |
| **Exportar** y `medicamentos.xlsx` con cinco columnas | 4.5 (V el contenido; el nombre en disco: No verificado) |
| «Error al buscar medicamentos» | 4.5 (No verificado: no se puede provocar) |
| Permiso `Medicamentos:Leer` y menú CATÁLOGOS | 2, 11 (V) |

### 14.2 Medicamentos — detalle `/medicamentos/nueva` y `/:id` (19d §1.2)

| Elemento | Apartado |
|---|---|
| Cabecera, modos crear, ver y editar, tarjeta «Datos del registro» | 4.4, 4.6 (V) |
| Campo «Código Nacional» (obligatorio, máximo 50, único) | 4.4 (V) |
| Campo «Nombre Comercial» (obligatorio, máximo 150) | 4.4 (V) |
| Campo «Laboratorio» (elegir de la lista, «Este campo es obligatorio») | 4.4 (V) |
| Campo «Unidad Base» (igual) | 4.4 (V) |
| Campo «Condición de Venta» | 4.2, 4.4 (V) |
| Campo «Precio de Venta Base» («El precio no puede ser negativo» también si está vacío; redondeo a dos decimales) | 4.4, 4.6 (V) |
| Tarjeta «Stock por Sucursal», «Stock Total», «Sin stock registrado», columnas «Sucursal» y «Stock» | 4.6 (V la parte sin stock; la tabla con filas: No verificado, no hay lotes) |
| Botones **Presentaciones de Venta**, **Editar**, **Cancelar**, **Guardar**, **Eliminar**, volver | 4.6, 6.5, 11 (V) |
| Cuadro «Confirmar eliminación» y «¿Estás seguro de que deseas eliminar este medicamento?» | 4.6, 10.1 (V) |
| «Medicamento guardado correctamente» | 4.4, 4.6 (V) |
| «Ya existe un medicamento con ese código nacional.» | 4.4 (V) |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` y sus tres textos | 4.4 (No verificado) |
| «Error al guardar el medicamento» | 4.4 (V con solo espacios) |
| «Medicamento eliminado correctamente» | 4.6, 10.1 (V) |
| «No puedes eliminar este medicamento porque tiene lotes, presentaciones u otros registros asociados.» | 10.1 (V) |
| «Error al eliminar el medicamento», «Error al cargar el medicamento», «Error al cargar las condiciones de venta» | 4.4, 4.6 (No verificado) |
| Las ocho causas que bloquean la eliminación | 10.1 (V las cuatro presentes: jerarquías, presentaciones, formulaciones y precios; las otras: No verificado) |

### 14.3 Medicamentos — carga masiva `/medicamentos/carga-masiva` (19d §1.3)

| Elemento | Apartado |
|---|---|
| Pantalla, migas, **Descargar plantilla**, zona de arrastre | 10.3 (V) |
| Plantilla con `CodigoNacional`, `NombreComercial`, `Laboratorio`, `UnidadBase`, `CondicionVenta` | 10.3 (V) |
| Vista previa: contadores, cinco columnas, errores, «Estado» | 10.3 (V) |
| Errores de fila (laboratorio inexistente, condición no válida, código existente) | 10.3 (V); resto: No verificado |
| **Guardar válidos (N)** y «Se guardaron N registros correctamente» | 10.3 (V) |
| Los medicamentos cargados quedan sin precio | 10.3 (V) |

### 14.4 Presentaciones de venta — búsqueda `/presentaciones-venta` (19d §2.1)

| Elemento | Apartado |
|---|---|
| Título, migas, tarjeta «Búsqueda de Presentaciones de Venta» | 6.2 (V) |
| Filtro «Producto» con sugerencias («Buscar producto...», producto exacto si lo eliges) | 6.2 (V) |
| Filtro «Unidad de Venta» | 6.2 (V) |
| Tabla: «Producto» (enlace), «Unidad de Venta», «Equivalencia», «Precio de Venta» («S/» con dos decimales) | 6.2 (V) |
| Botones **Nuevo**, **Limpiar**, **Buscar**, **Exportar**; sin carga masiva | 6.2 (V) |
| «Sin resultados» y paginador | 6.2 (V) |
| `presentaciones-venta.xlsx` con cuatro columnas | 6.2 (V el contenido) |
| La exportación ignora el producto elegido de la lista | No verificado en la aplicación (se exportó solo sin filtros); hallazgo de lectura de código de 19d §2.1 |
| «Error al buscar presentaciones de venta» | 6.2 (No verificado) |

### 14.5 Presentaciones de venta — detalle `/presentaciones-venta/nueva` y `/:id` (19d §2.2 y §2.3)

| Elemento | Apartado |
|---|---|
| Modos crear, ver y editar; el producto solo se elige al crear y su ayuda | 6.3, 6.4 (V) |
| Campo «Producto» (elegir de la lista, «Buscar producto...») | 6.3 (V) |
| Campo «Unidad de Venta» («Caja (100 Unidad)», «Debes seleccionar una unidad de venta») | 6.1, 6.3 (V) |
| Campo «Precio de Venta» («El precio de venta no puede ser negativo») | 6.3 (V) |
| Producto sin jerarquía: texto, botón **Ir a Jerarquías de Unidades**, **Guardar** sin efecto | 6.3 (V) |
| Botones **Editar**, **Cancelar**, **Guardar**, **Eliminar**, volver | 6.4 (V) |
| Cuadro «Confirmar eliminación» y «¿Estás seguro de que deseas eliminar esta presentación de venta?» | 6.4, 6.5 (V en el cuadro del medicamento) |
| «Presentación de venta guardada correctamente» | 6.3, 6.4 (V) |
| «Ya existe una presentación de venta para esa unidad en este producto.» | 6.3, 6.4 (V) |
| «Error al guardar la presentación de venta» | 6.3 (No verificado) |
| «Presentación de venta eliminada correctamente» | 6.5 (V) |
| «No se puede eliminar la presentación porque tiene ventas registradas.» | 6.4 (No verificado: no hay ventas) |
| «Error al eliminar la presentación de venta», «Error al cargar la presentación de venta» | 6.4 (No verificado) |
| Cálculo de la equivalencia en unidades base (origen en jerarquías, quién calcula, precio propio) | 6.1 (V las equivalencias 100, 50, 30 y 10) |

### 14.6 Presentaciones de venta — cuadro desde el medicamento (19d §2.4)

| Elemento | Apartado |
|---|---|
| Título «Presentaciones de Venta», nombre del medicamento, botón **Cerrar** | 6.5 (V) |
| Estados: sin jerarquía (con **Ir a Jerarquías de Unidades**), sin presentaciones («Este producto todavía no tiene presentaciones de venta.») y con tabla | 6.5 (V) |
| Tabla «Unidad de Venta», «Equivalencia», «Precio de Venta» y los iconos de editar y eliminar | 6.5 (V) |
| Formulario en línea («Unidad de Venta», «Precio de Venta»), **Cancelar**, **Guardar**, **Agregar presentación** (deshabilitado si ya hay una por unidad, con su ayuda) | 6.5 (V) |
| Avisos: guardada, eliminada, «Ya existe una presentación de venta para esa unidad.», «Error al cargar las presentaciones de venta», `delete-in-use` | 6.5 (V los dos primeros; los demás: No verificado) |

### 14.7 Formulaciones clínicas — búsqueda y detalle (19d §3.1 y §3.2)

| Elemento | Apartado |
|---|---|
| Título, migas, tarjeta «Búsqueda de Formulaciones Clínicas»; sin carga masiva | 7.2 (V) |
| Filtros «Medicamento» e «Ingrediente Activo» | 7.2, 7.4 (V) |
| Tabla: «Medicamento» (enlace), «Ingrediente Activo», «Concentración», «Unidad de Medida», «Código TE Orange» | 7.2 (V) |
| Botones **Nuevo**, **Limpiar**, **Buscar**, **Exportar**; fila «Resultados» | 7.2, 7.4 (V) |
| `formulaciones-clinicas.xlsx` con cinco columnas | 7.4 (V el contenido) |
| «Error al buscar formulaciones clínicas» | 7.4 (No verificado) |
| Campos «Medicamento», «Ingrediente Activo», «Concentración de Dosis», «Unidad de Medida», «Código TE Orange» (obligatoriedad, máximos y mensajes) | 7.3 (V) |
| El medicamento no se puede cambiar al editar | 7.5 (V) |
| Botones **Editar**, **Cancelar**, **Guardar**, **Eliminar** y cuadro «¿Estás seguro de que deseas eliminar esta formulación clínica?» | 7.5 (V) |
| «Formulación clínica guardada correctamente» | 7.3, 7.5 (V) |
| «Ya existe una formulación con los datos ingresados» | 7.3 (V) |
| «Error al guardar la formulación clínica», «Error al eliminar la formulación clínica», «Error al cargar la formulación clínica» | 7.3, 7.5 (No verificado) |
| «Formulación clínica eliminada correctamente» | 7.5 (V la desaparición de la fila; el aviso no se leyó) |
| Endpoint por medicamento sin pantalla | No aplica: es solo API y no tiene pantalla |

### 14.8 Proveedores — búsqueda, detalle y carga masiva (19d §4.1, §4.2 y §4.4)

| Elemento | Apartado |
|---|---|
| Título, migas, tarjeta «Búsqueda de Proveedores» | 8.2 (V) |
| Filtros «Razón Social», «RUC / Tax ID» y deslizador «Tasa de Cumplimiento» (no se restaura al volver; **Limpiar** lo devuelve a 0 — 100) | 8.2 (V el filtro; la restauración: No verificado) |
| Tabla: «Razón Social» (enlace), «RUC / Tax ID», «Tasa de Cumplimiento»; fila «Resultados» | 8.2 (V) |
| Botones **Carga Masiva**, **Nuevo**, **Limpiar**, **Buscar**, **Exportar** y `proveedores.xlsx` con tres columnas | 8.2 (V el contenido) |
| «Error al buscar proveedores» | 8.2 (No verificado) |
| Campos «Razón Social» (200), «RUC / Tax ID» (30), «Tasa de Cumplimiento» (valor inicial 100), «Calificación ESG» (sin mensaje), «Términos Financieros» (500 en pantalla, 100 en el servidor) | 8.3 (V) |
| Botones **Editar**, **Cancelar**, **Guardar**, **Eliminar**, volver; la tarjeta de precios se oculta al editar | 8.4 (V) |
| «Proveedor guardado correctamente» | 8.3, 8.4 (V) |
| «Ya existe un proveedor con ese identificador tributario.» y «Ya existe un proveedor con esa razón social.» | 8.3 (V) |
| «Error al guardar el proveedor» | 8.3 (V) |
| `Db.*` y sus tres textos | 8.3 (No verificado) |
| Cuadro «¿Estás seguro de que deseas eliminar este proveedor?», «Proveedor eliminado correctamente» | 8.4 (V) |
| «No puedes eliminar este proveedor porque tiene precios de compra registrados.» | 8.4 (No verificado, ver sección 13) |
| «Error al eliminar el proveedor», «Error al cargar el proveedor» | 8.4 (No verificado) |
| Carga masiva: plantilla (cinco columnas), etiquetas de la vista previa, errores de fila, límites | 10.5 (V la plantilla y la vista previa; el guardado: No verificado) |

### 14.9 Proveedores — sección «Precios de compra» y cuadro de precio (19d §4.3 y §4.5)

| Elemento | Apartado |
|---|---|
| Título «Precios de compra» y su subtítulo | 8.5 (V) |
| Botón **Registrar precio** (permiso `Registrar precio de proveedor`) | 8.5, 11 (V) |
| Tabla «Producto», «Costo por unidad base» (cuatro decimales), «Vigente desde», «Vigente hasta» y el texto «Vigente» | 8.5 (V) |
| Orden, paginador de 5, 10 y 25, «Este proveedor aún no tiene precios registrados.» | 8.5 (V) |
| «Error al cargar los precios del proveedor» | 8.5 (No verificado) |
| Cuadro «Registrar precio de compra» y su texto de ayuda | 8.5 (V) |
| Campo «Producto» («Escribe para buscar un producto», «Selecciona un producto de la lista», «Sin resultados») | 8.5 (V) |
| Campo «Precio de compra por unidad base (S/)» («Este campo es obligatorio», «Debe ser mayor que 0», «El precio es demasiado alto») | 8.5 (V) |
| Campo «Descuento (%)» (valor inicial 0, «Debe estar entre 0 y 100») | 8.5 (V) |
| «Costo neto por unidad base» calculado en vivo, «—» mientras no es válido | 8.5 (V) |
| Botones **Cancelar** y **Guardar** | 8.5 (V) |
| «Precio registrado correctamente» | 8.5 (V) |
| Un precio nuevo reemplaza al vigente y el anterior queda en el historial | 8.5 (V) |
| «El proveedor ya no existe.», «El producto seleccionado ya no existe.», «Error al registrar el precio», `Db.*` | 8.5 (No verificado) |

### 14.10 Aseguradoras — búsqueda, detalle y carga masiva (19d §5.1 a §5.3)

| Elemento | Apartado |
|---|---|
| Título «Aseguradoras Médicas», migas «Inicio / Catálogos / Aseguradoras», tarjeta «Búsqueda de Aseguradoras» | 9.2 (V) |
| Filtros «Nombre Comercial» y «Código Identificador Nacional» | 9.2 (V) |
| Tabla «Nombre Comercial» (enlace) y «Código Identificador Nacional»; fila «Resultados»; sin orden definido | 9.2 (V) |
| Botones **Carga Masiva**, **Nuevo**, **Limpiar**, **Buscar**, **Exportar** y `aseguradoras.xlsx` con dos columnas | 9.2 (V el contenido) |
| «Error al buscar aseguradoras» | 9.2 (No verificado) |
| Campos «Nombre Comercial» (150) y «Código Identificador Nacional» (50) | 9.3 (V) |
| Botones **Editar**, **Cancelar**, **Guardar**, **Eliminar**, volver | 9.4 (V) |
| «Aseguradora guardada correctamente» | 9.3, 9.4 (V) |
| «Error al guardar la aseguradora» (único aviso, también ante código repetido) | 9.3 (V) |
| Cuadro «¿Estás seguro de que deseas eliminar esta aseguradora?», «Aseguradora eliminada correctamente» | 9.4 (V) |
| «Error al eliminar la aseguradora», «Error al cargar la aseguradora» | 9.4 (No verificado) |
| Bloque de seguro del punto de venta oculto | 9.1 (V en el código) |
| Carga masiva: plantilla (dos columnas), vista previa y errores | 10.6 (V la plantilla y la vista previa; el guardado: No verificado) |

### 14.11 Jerarquías de unidades (19c §2.1, §2.2 y §2.3)

| Elemento | Apartado |
|---|---|
| Qué es una jerarquía y cómo se resuelve la cadena | 5.1 (V) |
| Título «Jerarquías de Unidades», migas, tarjeta «Búsqueda de Jerarquías» | 5.2 (V) |
| Filtros «Producto», «Unidad Mayor», «Unidad Menor» (con sugerencias) y deslizador «Multiplicador» (0 a 1000, paso 1) | 5.2 (V) |
| Tabla «Producto» (enlace), «Unidad Mayor», «Unidad Menor», «Multiplicador»; fila «Sin resultados» | 5.2 (V) |
| Botones **Carga Masiva**, **Nuevo**, **Limpiar**, **Buscar**, **Exportar** y `jerarquias-unidades.xlsx` con cuatro columnas | 5.2 (V el contenido) |
| «Error al buscar jerarquías de unidades» | 5.2 (No verificado) |
| Campos «Producto», «Unidad Mayor», «Unidad Menor», «Multiplicador» y su ayuda «Cuántas unidades menores equivalen a una unidad mayor» | 5.3 (V) |
| «La unidad mayor y la unidad menor no pueden ser la misma», «El multiplicador debe ser mayor que cero» | 5.3 (V) |
| Botones **Editar**, **Cancelar**, **Guardar**, **Eliminar**, volver; editar navega a la búsqueda | 5.4 (V) |
| Cuadro «¿Estás seguro de que deseas eliminar esta jerarquía?», «Jerarquía eliminada correctamente» | 5.4 (V) |
| «Jerarquía guardada correctamente» | 5.3, 5.4 (V) |
| «Este producto ya tiene una conversión entre esas unidades.» (duplicada) | 5.3 (V) |
| «Error al guardar la jerarquía» (inconsistente y otros) | 5.3 (V) |
| «No se puede eliminar la jerarquía porque está en uso.», «Error al eliminar la jerarquía», «Error al cargar la jerarquía» | 5.4 (No verificado) |
| `Db.*` y sus tres textos | 5.3 (No verificado) |
| Carga masiva: plantilla (cuatro columnas), vista previa y errores | 10.4 (V la plantilla y la vista previa; el guardado: No verificado) |

### 14.12 Comportamientos comunes (19d §0) y eliminar (19d §6)

| Elemento | Apartado |
|---|---|
| Permisos de ruta y módulos (`Medicamentos`, `PresentacionesVenta`, `FormulacionesClinicas`, `Proveedores`, `Seguros`, `JerarquiasUoM`) | 2, 11 |
| Menú CATÁLOGOS con las nueve opciones | 2, 11 (V) |
| Fila sin resultados: «Sin resultados» (medicamentos, presentaciones, jerarquías) o «Resultados» (formulaciones, proveedores, aseguradoras) | 4.3, 5.2, 6.2, 7.2, 8.2, 9.2 (V) |
| Carga masiva solo en Medicamentos, Jerarquías, Proveedores y Aseguradoras | 6.2, 7.2, 10 (V) |
| Mapa de eliminar de las cinco pantallas | 4.6, 5.4, 6.4, 7.5, 8.4, 9.4, 10.1 |
| Claves de `proveedores` sin uso (`help.*`, `form.*`, `import.*`, `notifications.*`, `deleteDialog.title`, `searchPlaceholder`, `add`) | No aplica: el script de comprobación las lista como ausentes porque ninguna pantalla las muestra (19d §9) |

## 15. Fuentes

Documentos consultados:

- `_inventario/REGLAS-AGENTES.md` y `_inventario/MANUAL-AGENTES.md`.
- `ola-1-fundamentos/19-inventario-de-pantallas/19d-catalogos-2.md` (§0 a §9) y `19c-catalogos-1.md` (§2 y §6).
- `ola-1-fundamentos/20-carga-masiva-y-exportacion.md` (§3.1, §3.4, §3.6, §3.9).
- `ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md` (reglas MED, JER, PRE, FOR, PRV y ASE, versión 1.1).
- `ola-1-fundamentos/06-glosario.md` («Condición de venta», «Unidad base», «Jerarquía de unidades», «Presentación de venta», «Precio de venta base», «Precio de compra de proveedor», «Catálogo DIGEMID»).
- `ola-1-fundamentos/19-inventario-de-pantallas/19f-punto-de-venta.md` (§3.8, bloque de aseguradora oculto) y `sofia-web-spa/public/i18n/pos/es.json` (textos de receta del punto de venta).
- `sofia-web-spa/src/app/pages/pos/nueva-venta/nueva-venta.component.ts:215` y `.html:456` (constante `mostrarAseguradora`).
- `ola-2-funcional/manual-usuario/capitulo-04-catalogos-basicos.md` (modelo de formato y datos de entrada).

Recorridos hechos en la aplicación (`http://localhost:4200`, sesión del administrador de la empresa de prueba, salvo el último):

- Medicamentos: búsqueda vacía; **Nuevo** con formulario vacío, precio negativo, laboratorio sin elegir, código repetido en minúsculas y código de solo espacios; creación de los cuatro medicamentos; filtros (código, nombre, laboratorio, unidad, condición, combinados, sin resultados); **Limpiar**; **Exportar** (contenido del archivo); detalle, **Editar**, **Cancelar**, cambio de precio (con tres decimales); intento de eliminar un medicamento en uso; creación y eliminación de uno de prueba; carga masiva (plantilla, vista previa de cinco filas, guardado de dos) y edición de precio de los dos cargados.
- Jerarquías: **Nuevo** vacío, unidades iguales, multiplicador cero y vacío, jerarquía duplicada, jerarquías inconsistentes; creación de cinco jerarquías; filtros, deslizador, **Exportar**; detalle, **Editar**, **Cancelar**, **Guardar**; creación y eliminación de una de prueba; plantilla y vista previa de carga masiva.
- Presentaciones de venta: **Nuevo** vacío, producto sin jerarquía, **Ir a Jerarquías de Unidades**, precio negativo y vacío, duplicada; creación de cinco; filtros y **Exportar**; detalle, **Editar**, **Cancelar**, duplicado al editar, cambio de precio; cuadro del medicamento (sin jerarquía, sin presentaciones, con presentaciones, agregar, editar, **Cancelar**, eliminar).
- Unidades en uso: intentos de eliminar `FRA` y `UND`.
- Formulaciones clínicas: **Nuevo** vacío, concentración cero y negativa, código TE Orange, duplicada; creación de cuatro; filtros y **Exportar**; detalle, **Editar**, **Cancelar**, cambio de concentración; creación y eliminación de una segunda formulación.
- Proveedores: **Nuevo** vacío, tasa y ESG fuera de rango, RUC y razón social repetidos, términos de 150 caracteres, razón social de solo espacios; creación de dos; filtros con el deslizador y **Exportar**; detalle, **Editar**, eliminación del proveedor de prueba; cuadro de precio (vacío, producto sin elegir, precio cero, negativo y muy alto, descuento fuera de rango, costo neto en vivo, **Cancelar**), registro de tres precios y comprobación del historial; plantilla y vista previa de carga masiva.
- Aseguradoras: **Nuevo** vacío, código repetido; creación de dos; filtros, **Exportar**; detalle, **Editar**, **Cancelar**, **Guardar**, eliminación de la de prueba; plantilla y vista previa de carga masiva.
- Cajero (`carlos.cajero`): menú, medicamentos (búsqueda y detalle) y aseguradoras (búsqueda y detalle). Al terminar se volvió a iniciar la sesión del administrador.
- Comprobación por script (`cap5-check.js`, en el directorio temporal de la sesión): lee `19d-catalogos-2.md` (§1 a §6) y `19c-catalogos-1.md` (§2), extrae los 185 textos entre comillas latinas y comprueba que cada uno aparezca en este capítulo; ausentes al final quedan 4 y ninguno es un texto de pantalla: dos ejemplos del inventario (`Caja (30 Tableta)`, `30 Tableta`), el patrón `{descripción} ({equivalencia} {unidad base})` y la frase de definición de jerarquía (que este capítulo redacta con otras palabras). También compara los textos de los `es.json` de los seis scopes (medicamentos, presentaciones de venta, formulaciones clínicas, proveedores, aseguradoras y jerarquías): solo quedan ausentes 26 claves de `proveedores` que ninguna pantalla muestra (ayudas, títulos de formulario, avisos y textos de carga masiva propios de ese scope, que la pantalla no usa; 19d §9). Comprueba además las secciones finales y que no haya voseo.
