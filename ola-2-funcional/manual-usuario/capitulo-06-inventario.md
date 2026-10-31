# Capítulo 6 — Inventario: lotes y stock por sucursal

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-30 |
| **Backend** | `sofia-apis`, rama `main`, commit `34089ec` (más nuevo que la línea base `43f5529`; `git status --short` sin cambios locales al empezar) |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `9ad15f0` (más nuevo que la línea base `eeffbb6`; `git status --short` sin cambios locales al empezar) |
| **Método** | Recorrido en la aplicación local más inventario de pantallas |

## 1. Introducción

En este capítulo registras **qué mercadería tienes y dónde está**. Vas a:

- Crear **lotes**: la tanda de fabricación de un medicamento, con su número, su fecha de fabricación y su fecha de caducidad.
- **Buscar, ver, editar y eliminar** lotes.
- **Ingresar stock**: decir cuántas unidades de un lote hay en una sucursal.
- **Consultar el stock por sucursal**, filtrarlo y exportarlo.
- **Ajustar** una cantidad cuando el conteo físico no coincide con el sistema.
- Ver qué pasa cuando intentas eliminar un lote que ya tiene stock.
- Ver qué pantallas de inventario usa un cajero.

Es para el **administrador** (rol «Admin», que lo puede todo). La sección 13 explica qué ve el cajero.

Al terminar, la empresa de prueba «Botica Manual Demo S.A.C.» tendrá **lotes con stock** de los medicamentos `DEMO-MED-0001` a `DEMO-MED-0006`, listos para los capítulos de recetas, caja y venta, y devoluciones. Los nombres de lote y las cantidades son inventados.

## 2. Antes de empezar

- Haz los capítulos 1 a 5. Necesitas la sesión iniciada como administrador (`ana.manual`), las dos sucursales («Sede Principal» y «Sucursal Demo Norte») y los medicamentos del capítulo 5.
- Permisos que usa este capítulo. Los nombres son los que ves en la pantalla de permisos de un rol (Administración › Roles & Permisos): primero el módulo y luego la acción. **Lotes y stock comparten el mismo módulo, «Inventario».**

| Tarea | Módulo | Acción |
|---|---|---|
| Ver, buscar y exportar lotes; ver, buscar y exportar stock | Inventario | `Leer` |
| Crear un lote | Inventario | `Crear` (además de `Leer`) |
| Editar un lote | Inventario | `Actualizar` |
| Eliminar un lote | Inventario | `Eliminar` |
| **Ingresar** stock y **ajustar** stock | Inventario | `Actualizar` (**no** hace falta `Crear`) |

- Para elegir el producto de un lote (o buscarlo por nombre), la pantalla consulta los medicamentos y necesita además `Medicamentos › Leer`. El administrador y el cajero del manual lo tienen.
- En el menú lateral, las dos pantallas están en el grupo **INVENTARIO**: «Lotes» y «Stock por Sucursal». Si tu rol no tiene `Inventario › Leer`, el grupo no aparece.
- Los avisos de la aplicación aparecen abajo al centro con un botón **Cerrar** y desaparecen solos a los pocos segundos. Los textos de este capítulo están copiados tal como se vieron en pantalla.
- Las fechas se muestran siempre como día/mes/año (`dd/MM/aaaa`). En los selectores de fecha puedes escribir la fecha a mano en ese mismo formato (por ejemplo `31/12/2027`) o usar el icono de calendario de la derecha del campo.
- Los campos obligatorios de los formularios llevan un asterisco junto a la etiqueta (por ejemplo «Número de Lote*» y «Fecha de Caducidad*»). El campo «Producto» del lote es obligatorio aunque se vea sin asterisco.

## 3. Cómo se relacionan estas piezas (en lenguaje simple)

| Pieza | Qué es | Para qué la usa SOFIA |
|---|---|---|
| **Lote** | Una tanda de un medicamento, con el **número de lote del fabricante**, una **fecha de fabricación** (opcional) y una **fecha de caducidad** | Saber qué se vence y cuándo. El lote **no guarda cantidades** |
| **Stock por sucursal** (también «registro de stock») | La **cantidad física** de **un lote** en **una sucursal**. Hay a lo sumo un registro por lote y sucursal | Es lo que se descuenta al vender y lo que muestra el medicamento en su tarjeta «Stock por Sucursal» |
| **Ingreso de stock** | Una operación que **suma** unidades a un lote en una sucursal (o crea el registro si no existía) | Registrar una compra o una entrada de mercadería |
| **Ajuste de stock** | Una operación que **reemplaza** la cantidad de un registro por el valor nuevo | Corregir la cantidad cuando el conteo físico no coincide |
| **Transferencia entre sucursales** | Mover stock de una sucursal a otra | **Solo API (sin pantalla)**: no existe en la aplicación. Ver la sección 12 |

Idea central: **primero se crea el lote, después se le da stock**. Un lote recién creado tiene cero unidades en todas las sucursales y no aparece en «Stock por Sucursal» hasta que ingresas stock (comprobado).

Un lote se puede vender **hasta el final de su día de caducidad**: se considera vencido cuando su fecha de caducidad es anterior a hoy. Para que una unidad se considere **vendible** debe cumplir tres cosas: cantidad mayor que cero, lote no vencido y lote sin retención activa en cuarentena (regla `RN-STK-017` de `01c1-inventario-y-recetas.md`; el punto de venta se recorre en el capítulo 8).

Fuente de las reglas: `01c1-inventario-y-recetas.md` (`RN-LOT-001` a `RN-LOT-019`, `RN-STK-001` a `RN-STK-018`).

## 4. Los datos que se crean en este capítulo

Para los capítulos siguientes se crean **siete lotes**, uno por medicamento y uno más del Paracetamol con **vencimiento cercano**. Todos los números de lote son inventados.

| Medicamento | Número de lote | Fabricación | Caducidad | Para qué sirve |
|---|---|---|---|---|
| `DEMO-MED-0001` Paracetamol Demo 500 mg | `LOT-PARA-2027` | 01/06/2026 | 31/12/2027 | Lote normal, con stock en las dos sucursales |
| `DEMO-MED-0001` Paracetamol Demo 500 mg | `LOT-PARA-CERCA` | 01/01/2026 | 15/11/2026 | **Vencimiento cercano** (16 días después de la fecha del recorrido, 30/10/2026) |
| `DEMO-MED-0002` Amoxicilina Demo 500 mg | `LOT-AMOX-2027` | 01/07/2026 | 30/06/2027 | Receta simple |
| `DEMO-MED-0003` Clonazepam Demo 2 mg | `LOT-CLON-2027` | 01/08/2026 | 31/12/2027 | Controlado (receta retenida) |
| `DEMO-MED-0004` Tramadol Demo 50 mg | `LOT-TRAM-2027` | 01/08/2026 | 31/12/2027 | Estupefaciente |
| `DEMO-MED-0005` Ibuprofeno Demo 400 mg | `LOT-IBU-2027` | 01/08/2026 | 31/12/2027 | Venta libre |
| `DEMO-MED-0006` Omeprazol Demo 20 mg | `LOT-OME-2027` | 01/08/2026 | 31/12/2027 | Receta simple |

Y este stock (cantidad física por sucursal) al terminar el capítulo:

| Número de lote | Sede Principal | Sucursal Demo Norte |
|---|---|---|
| `LOT-PARA-2027` | 200 | 30 |
| `LOT-PARA-CERCA` | 15 | — |
| `LOT-AMOX-2027` | 100 | 20 |
| `LOT-CLON-2027` | 60 | — |
| `LOT-TRAM-2027` | 60 | — |
| `LOT-IBU-2027` | 100 | — |
| `LOT-OME-2027` | 100 | — |

La «Sede Principal» es la sucursal base del cajero `carlos.cajero`; por eso concentra casi todo el stock. Las cantidades son unidades base (`UND`), porque todos los medicamentos del manual usan la unidad base `UND` (capítulo 5).

## 5. Crear un lote

**Cómo llegar:** menú lateral › INVENTARIO › **Lotes** (`/lotes`).

La pantalla se titula «Lotes», con las migas «Inicio / Inventario / Lotes» y una tarjeta «Búsqueda de Lotes» («Ingresa los criterios de búsqueda»). Arriba a la derecha de la tarjeta está el botón **Nuevo** (con el permiso `Inventario › Crear`). Los filtros se explican en la sección 6. Al entrar por primera vez no hay tabla: aparece al hacer clic en **Buscar**. Con la empresa recién empezada, **Buscar** muestra «Resultados (0)», la cabecera de la tabla y la fila «Sin resultados».

### 5.1 Pasos

1. Haz clic en **Nuevo** (`/lotes/nueva`). Se abre un formulario con el título «Lotes», una flecha para volver y cuatro campos: «Producto», «Número de Lote», «Fecha de Fabricación» y «Fecha de Caducidad».
2. En «Producto», escribe parte del nombre comercial (por ejemplo «Paracetamol») y **elige una opción de la lista**. La lista tarda un instante en llenarse: mientras la consulta está en curso puede mostrar «Sin resultados»; espera un segundo y aparece el nombre («Paracetamol Demo 500 mg»).
3. Escribe el «Número de Lote» (el que trae el envase del fabricante).
4. Si lo conoces, escribe o elige la «Fecha de Fabricación».
5. Escribe o elige la «Fecha de Caducidad».
6. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Producto | El medicamento al que pertenece el lote | Sí | Debes **elegir una opción de la lista**. Si solo escribes el texto y haces clic en **Guardar**, sale «Este campo es obligatorio» bajo el campo y no se envía nada. Sin coincidencias, la lista muestra «Sin resultados». **No se puede cambiar después** (sección 7) |
| Número de Lote | El número de lote del fabricante, por ejemplo `LOT-PARA-2027` | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 100 caracteres. No puede repetirse **para el mismo producto**, ni con otras mayúsculas (ver errores abajo). Sí puede repetirse entre productos distintos |
| Fecha de Fabricación | El día en que se fabricó el lote | No | No puede ser una fecha futura (ver errores abajo) |
| Fecha de Caducidad | El último día en que el lote se puede vender | Sí | Vacía: «Este campo es obligatorio». No puede ser una fecha ya vencida (ver errores abajo) |

Los selectores de fecha **no te impiden** elegir una fecha vencida o futura: el rechazo llega recién al guardar y con un aviso genérico.

### 5.2 Qué pasa al terminar bien

Aparece el aviso «Lote guardado correctamente» y el sistema te lleva a la búsqueda de lotes. Si ya habías buscado antes en esta sesión, la pantalla repite esa búsqueda sola y verás la tabla con el lote nuevo; si no, verás solo los filtros (haz clic en **Buscar**).

Crea ahora los siete lotes de la tabla de la sección 4. El lote nace **sin existencias**: el stock se ingresa aparte (sección 8).

Si no ves **Nuevo**, a tu rol le falta `Inventario › Crear`.

### 5.3 Errores que puedes ver (todos comprobados en la aplicación)

1. **Guardar** con el formulario vacío: «Este campo es obligatorio» bajo «Producto», «Número de Lote» y «Fecha de Caducidad». «Fecha de Fabricación» no marca error. No se envía nada.
2. Texto escrito en «Producto» sin elegir de la lista, con los demás campos bien: «Este campo es obligatorio» bajo «Producto».
3. Fecha de caducidad **de ayer** (`29/10/2026`, con el recorrido hecho el 30/10/2026): aviso rojo **«Error al guardar el lote»** y te quedas en el formulario. Una caducidad **de hoy** (`30/10/2026`) sí se aceptó.
4. Fecha de fabricación **futura** (`01/01/2027`): el mismo aviso «Error al guardar el lote».
5. Número de lote ya existente para el mismo producto (`LOT-PARA-2027`, y también `lot-para-2027` en minúsculas): el mismo aviso «Error al guardar el lote». **No** aparece un mensaje de «ya existe». La respuesta del servidor sí dice el motivo (`A batch with number '…' already exists for this product.`), pero la pantalla no lo muestra.
6. Número de lote formado solo por espacios: el mismo aviso «Error al guardar el lote».

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Este campo es obligatorio» | Falta el producto (elegido de la lista), el número de lote o la caducidad | Completa el campo; en «Producto», elige una opción de la lista |
| «Error al guardar el lote» | El servidor rechazó el guardado. Causas posibles: caducidad vencida, fabricación futura, número de lote repetido para ese producto, número formado solo por espacios, u otro problema | Revisa en este orden: que la caducidad sea hoy o futura, que la fabricación no sea futura, que el número no exista ya para ese producto (búscalo en la lista), y que no sean solo espacios |
| «Ya existe un lote con ese número para el producto seleccionado» | Dos personas guardaron el mismo número a la vez | Busca el lote en la lista. (No verificado en la aplicación: no se pudo provocar; ver sección 15) |

Otras reglas del servidor que se comprobaron por lectura del inventario y no se pudieron provocar: la fabricación no puede ser posterior a la caducidad al crear (`RN-LOT-008`).
## 6. Buscar, filtrar y exportar lotes

**Cómo llegar:** menú lateral › INVENTARIO › **Lotes**.

### 6.1 Los filtros

| Filtro | Qué hace |
|---|---|
| Producto | Texto con lupa y lista de sugerencias («Buscar producto...»). Filtra por el **texto** que escribas: nombres comerciales que lo **contengan**, sin distinguir mayúsculas, aunque no elijas una sugerencia |
| Número de Lote | Texto con lupa. «Contiene», sin distinguir mayúsculas |
| Caducidad Desde / Caducidad Hasta | Fechas. Lotes cuya caducidad cae en el rango; las dos fechas **están incluidas** |
| Fabricación Desde / Fabricación Hasta | Fechas, con el mismo criterio. Los lotes **sin fecha de fabricación quedan fuera** cuando usas cualquiera de estos dos filtros |

Los seis filtros se combinan. **Limpiar** vacía todos los campos y oculta la tabla. No hay validación entre «Desde» y «Hasta»: si «Desde» es posterior a «Hasta», la búsqueda devuelve «Resultados (0)».

### 6.2 La tabla de resultados

Encabezado «Resultados (N)». Columnas: **Producto** (enlace al detalle), **Número de Lote**, **Fecha de Fabricación** (se muestra «—» si el lote no la tiene) y **Fecha de Caducidad**. Los lotes salen del más reciente al más antiguo según su fecha de creación. Debajo hay el botón **Exportar** y el paginador («Items por página», 10 por defecto, «Página 1 de 1»). **La tabla no marca los lotes vencidos ni los que están por vencer**: para eso usa los filtros de caducidad (sección 9.4).

### 6.3 Comprobado con los siete lotes del capítulo

- **Buscar** sin filtros: «Resultados (7)». Fila por lote con los cuatro datos de la tabla de la sección 4.
- «Producto» = `ibu`: los lotes de Ibuprofeno. «Número de Lote» = `para`: los tres lotes cuyo número contiene «para» (`LOT-PARA-2027` de Paracetamol, `LOT-PARA-CERCA` y el `LOT-PARA-2027` de prueba del Omeprazol, que se eliminó en la sección 11).
- «Caducidad Hasta» = `15/11/2026`: `LOT-PARA-CERCA` (el día 15/11 está incluido) y el lote de prueba que vencía el 30/10/2026. «Caducidad Desde» = `01/01/2027`: los lotes que vencen desde ese día.
- «Fabricación Desde» = `01/07/2026`: solo los lotes con fabricación desde ese día; el lote de prueba sin fecha de fabricación **no** aparece. «Fabricación Hasta» = `01/06/2026`: los dos lotes de Paracetamol.
- «Producto» = `zzz`, o «Caducidad Desde» = `01/01/2028` con «Caducidad Hasta» = `01/01/2027`: «Resultados (0)» y la fila «Sin resultados».
- **Limpiar**: los campos quedan vacíos y desaparece la tabla.

Si falla la búsqueda verás «Error al buscar lotes» (No verificado en la aplicación: no se pudo provocar). Si vuelves de un detalle, la pantalla recuerda los filtros y repite la búsqueda sola.

### 6.4 Exportar

**Exportar** genera el archivo `lotes.xlsx` (nombre según el inventario; no se pudo ver en el disco). Comprobado: es un libro de Excel con una hoja llamada «Data», una fila de encabezados («Producto», «Número de Lote», «Fecha de Fabricación», «Fecha de Caducidad») y una fila por lote, con las fechas como `dd/MM/aaaa`. Respeta los filtros que tengas escritos en el momento de hacer clic. Si falla, el aviso es el mismo «Error al buscar lotes».

**No existe carga masiva de lotes**: cada lote se crea uno por uno.

## 7. Ver y editar un lote

1. En los resultados, haz clic en el nombre del producto del lote. Se abre el detalle con los cuatro campos deshabilitados. Arriba de la tarjeta ves el título «Lotes», el texto «Datos del registro» y, a la derecha, el botón **Editar** (con el permiso `Inventario › Actualizar`).
2. Haz clic en **Editar**. Se habilitan «Número de Lote», «Fecha de Fabricación» y «Fecha de Caducidad»; el **«Producto» sigue deshabilitado** (un lote no cambia de producto; si te equivocaste, elimina el lote y créalo de nuevo). Aparecen **Cancelar**, **Guardar** y **Eliminar** (este último con el permiso `Inventario › Eliminar`).
3. **Cancelar** descarta los cambios y vuelve al modo ver con los datos anteriores (comprobado).
4. Cambia el número de lote o las fechas y haz clic en **Guardar**: aparece «Lote guardado correctamente» y, a diferencia de otros catálogos, **vuelves a la búsqueda** (no te quedas en el detalle), con los filtros que tenías puestos (comprobado: se cambió un lote de prueba de `LOT-PRUEBA-BORRAR` a `LOT-PRUEBA-EDITADO` y se le agregó la fecha de fabricación 01/10/2026).

Errores al editar: poner a un lote el número de **otro lote del mismo producto** da el aviso rojo «Error al guardar el lote» y te quedas en el formulario (comprobado). Las demás reglas son las de la sección 5.3. Si el detalle no abre, verás «Error al cargar el lote» (No verificado en la aplicación: no se pudo provocar).

(No verificado en la aplicación: según el inventario, un lote que ya venció no se puede guardar al editar, ni siquiera para corregir su número, hasta ponerle una caducidad vigente. No se pudo comprobar porque SOFIA no deja crear lotes vencidos y el único lote que vencía el día del recorrido seguía vigente.)

## 8. Ingresar stock

Un ingreso dice «de este lote hay N unidades más en esta sucursal». **Suma** a lo que ya había; no reemplaza.

**Cómo llegar:** menú lateral › INVENTARIO › **Stock por Sucursal** (`/stock-por-sucursal`) › **Nuevo** (`/stock-por-sucursal/nueva`). El botón **Nuevo** de esta pantalla pide `Inventario › Actualizar` (no `Crear`).

### 8.1 Pasos

1. En «Stock por Sucursal», haz clic en **Nuevo**. Se abre un formulario con el título «Stock por Sucursal», una flecha para volver y tres campos: «Sucursal», «Lote» y «Cantidad Física» (este último empieza en 0).
2. En «Sucursal», escribe parte del nombre (por ejemplo «Sede») y **elige una opción de la lista**. La lista ofrece solo las sucursales a las que tu usuario tiene acceso.
3. En «Lote», escribe el **número de lote** (por ejemplo `LOT-PARA`) y **elige una opción de la lista**. Cada opción muestra el número de lote y, debajo, el nombre del producto; si dos productos tienen un lote con el mismo número, aparecen las dos opciones y debes elegir la del producto correcto. Al elegir, el campo muestra «número — producto», por ejemplo «LOT-PARA-2027 — Paracetamol Demo 500 mg».
4. Escribe la «Cantidad Física» (las unidades que ingresan).
5. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Sucursal | La sucursal donde está la mercadería | Sí | Debes elegir una opción de la lista (placeholder «Buscar sucursal...»; sin coincidencias, «Sin resultados») |
| Lote | El lote que ingresa | Sí | Debes elegir una opción de la lista (placeholder «Buscar por número de lote...»). La lista muestra cualquier lote de tu empresa, **también uno ya vencido**: SOFIA no comprueba la vigencia al ingresar |
| Cantidad Física | Número de unidades base | Sí | Vacía o negativa: «La cantidad no puede ser negativa». El 0 es válido. Los decimales se aceptan al ajustar (sección 10); al ingresar, No verificado en la aplicación |

### 8.2 Qué pasa al terminar bien

Aparece el aviso «Stock registrado correctamente» y el sistema te lleva a la búsqueda de stock. Si ya había stock de ese lote en esa sucursal, **se suma**: comprobado, se ingresaron 100 unidades de `LOT-PARA-2027` en la «Sede Principal» y, al repetir el ingreso con otras 100, la cantidad quedó en **200**. El aviso es el mismo cuando el registro se crea y cuando se suma.

Haz estos ingresos (el primero se hizo dos veces, para ver la suma):

| Sucursal | Lote | Cantidad a ingresar | Queda |
|---|---|---|---|
| Sede Principal | `LOT-PARA-2027` | 100, y otra vez 100 | 200 |
| Sede Principal | `LOT-AMOX-2027` | 100 | 100 |
| Sede Principal | `LOT-CLON-2027` | 60 | 60 |
| Sede Principal | `LOT-TRAM-2027` | 60 | 60 |
| Sede Principal | `LOT-PARA-CERCA` | 20 (luego se ajusta a 15, sección 10) | 15 |
| Sede Principal | `LOT-IBU-2027` | 100 | 100 |
| Sede Principal | `LOT-OME-2027` | 100 | 100 |
| Sucursal Demo Norte | `LOT-PARA-2027` | 30 | 30 |
| Sucursal Demo Norte | `LOT-AMOX-2027` | 20 | 20 |

Si no ves **Nuevo**, a tu rol le falta `Inventario › Actualizar`. Si escribes la dirección a mano sin ese permiso, el sistema te devuelve al Inicio con «No tienes permisos para acceder a esta pantalla.» (No verificado en la aplicación con este formulario; viene de 19e §0).

### 8.3 Errores que puedes ver

| Qué hiciste | Qué pasa (comprobado) | Qué hacer |
|---|---|---|
| **Guardar** sin elegir ninguna sucursal ni lote | **No pasa nada**: no sale ningún aviso ni mensaje bajo los campos y te quedas en el formulario | Elige la sucursal y el lote de la lista y vuelve a guardar |
| Elegiste un lote y después cambiaste o borraste el texto del campo «Lote» | Al guardar tampoco pasa nada, y la elección se descarta | Vuelve a escribir el número de lote y elige otra vez una opción de la lista |
| Cantidad negativa o vacía | «La cantidad no puede ser negativa» bajo el campo, sin enviar nada | Escribe 0 o una cantidad positiva |

Los textos «Debes seleccionar una sucursal» y «Debes seleccionar un lote» existen en el sistema pero **no se ven nunca**: lo comprobado es que el formulario se queda mudo.

Otros avisos posibles al guardar (No verificado en la aplicación: no se pudieron provocar; vienen de 19e §2.2): «No tienes permiso para operar el stock de esta sucursal» (la sucursal no es una de las tuyas), «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» (alguien tocó el mismo registro a la vez) y «Error al registrar el stock» (cualquier otro fallo).

## 9. Consultar el stock por sucursal

**Cómo llegar:** menú lateral › INVENTARIO › **Stock por Sucursal**.

### 9.1 La pantalla de búsqueda

Título «Stock por Sucursal», migas «Inicio / Inventario / Stock por Sucursal» y tarjeta «Consulta de Stock por Sucursal» («Ingresa los criterios de búsqueda»). Arriba a la derecha de la tarjeta, **Nuevo** (con `Inventario › Actualizar`). El administrador ve el stock de **todas** las sucursales; los demás usuarios solo el de las sucursales a las que tienen acceso (sección 13).

| Filtro | Qué hace |
|---|---|
| Sucursal | Texto con lupa y sugerencias («Buscar sucursal...»). Filtra por nombres que **contengan** lo que escribas, sin distinguir mayúsculas |
| Producto | Texto con lupa y sugerencias («Buscar producto...»). «Contiene» |
| Número de Lote | Texto con lupa. «Contiene» |
| Caducidad Desde / Caducidad Hasta | Fechas, ambas incluidas |
| Cantidad Mínima / Cantidad Máxima | Números (no deslizadores). Dejan filas con cantidad entre los dos valores, ambos incluidos |
| Solo con stock disponible | Casilla, **marcada al abrir y después de Limpiar**. Marcada, la lista oculta los registros con cantidad 0 |

La tabla tiene cinco columnas: **Sucursal** (enlace al detalle), **Producto**, **Número de Lote**, **Fecha de Caducidad** y **Cantidad Física**. Sale ordenada por sucursal, luego producto y luego caducidad. Debajo, **Exportar** y el paginador. Fila sin resultados: «Sin resultados». La tabla **no marca** los lotes vencidos.

### 9.2 Comprobado con el stock del capítulo

- **Buscar** sin tocar nada: «Resultados (9)» (las nueve filas de la tabla de la sección 8.2, con la cantidad final).
- «Sucursal» = `norte`: solo las dos filas de «Sucursal Demo Norte». «Producto» = `paracetamol`: tres filas (los dos lotes de la Sede Principal y el de la Sucursal Demo Norte). «Número de Lote» = `clon`: una fila.
- «Caducidad Hasta» = `31/12/2026`: solo `LOT-PARA-CERCA`. «Caducidad Desde» = `01/01/2027`: las ocho filas restantes.
- «Cantidad Mínima» = `100`: cuatro filas (con 100 o más). «Cantidad Máxima» = `30`: tres filas (15, 20 y 30). Mínima 50 y máxima 100 juntas: cinco filas.
- **Solo con stock disponible**: con la casilla marcada, un registro con cantidad 0 **no aparece**; al **desmarcarla** y hacer clic en **Buscar**, vuelve a aparecer (se comprobó dejando un registro en 0 y luego restaurándolo).
- Un filtro sin coincidencias (por ejemplo, el cajero escribiendo «norte» en «Sucursal», sección 13): «Resultados (0)» y «Sin resultados».
- **Limpiar** vacía los filtros, vuelve a marcar la casilla y oculta la tabla.

Si falla la búsqueda verás «Error al consultar el stock» (No verificado en la aplicación: no se pudo provocar). Si falla la carga de tus sucursales al escribir en «Sucursal» verás «No se pudieron cargar tus sucursales» (No verificado: no se pudo provocar). La pantalla recuerda los filtros al volver de un detalle.

### 9.3 Exportar

**Exportar** genera `stock-por-sucursal.xlsx` (nombre según el inventario; no se vio en el disco). Comprobado con «Sucursal» = `norte`: libro de Excel con la hoja «Data», los encabezados «Sucursal», «Producto», «Número de Lote», «Fecha de Caducidad» y «Cantidad Física», y solo las dos filas de esa sucursal, con la cantidad como número. Respeta los filtros escritos al hacer clic, incluida la casilla «Solo con stock disponible». No existe carga masiva de stock.

### 9.4 Ver lotes vencidos o por vencer desde el Dashboard

El **Dashboard** tiene una zona «Inventario» con cuatro tarjetas de lotes: «Lotes vencidos con stock», «Lotes que vencen en 30 días o menos», «Lotes que vencen en 31 a 60 días» y «Lotes que vencen en 61 a 90 días», cada una con un número y el enlace **Ver lotes**. Con el stock de este capítulo, la tarjeta de 30 días o menos mostró **1** (el lote `LOT-PARA-CERCA`) y las demás 0. **Ver lotes** abre «Stock por Sucursal» con los filtros ya puestos: para esa tarjeta, «Caducidad Desde» = `30/10/2026` y «Caducidad Hasta» = `29/11/2026`, la casilla marcada y la tabla con la fila de `LOT-PARA-CERCA` (cantidad 15). Es la forma más rápida de revisar qué se te vence.

### 9.5 Ver el stock desde un medicamento

El detalle de un medicamento (capítulo 5, sección 4.6) tiene la tarjeta «Stock por Sucursal» con «Stock Total» y una fila por sucursal. Comprobado con `DEMO-MED-0001` (Paracetamol): «Stock Total: 245», «Sede Principal 215» (200 + 15 de los dos lotes) y «Sucursal Demo Norte 30».

## 10. Ajustar una cantidad

Un ajuste dice «la cantidad correcta de este registro es N». **Reemplaza** la cantidad anterior; no suma. Úsalo cuando el conteo físico no coincide con el sistema.

**Cómo llegar:** menú lateral › INVENTARIO › **Stock por Sucursal** › **Buscar** › clic en el nombre de la sucursal de la fila (`/stock-por-sucursal/:id`).

### 10.1 Pasos

1. En los resultados, haz clic en el nombre de la sucursal del registro que quieres corregir (por ejemplo, la fila de `LOT-PARA-CERCA` en la «Sede Principal», que tiene 20).
2. Se abre el «Detalle de Stock» con el texto «Datos del registro» y cinco campos deshabilitados: «Sucursal», «Producto», «Número de Lote», «Fecha de Caducidad» y «Cantidad Física». A la derecha ves el botón **Editar** (con `Inventario › Actualizar`).
3. Haz clic en **Editar**. Solo se habilita «Cantidad Física»; aparecen **Cancelar** y **Guardar**. No hay botón **Eliminar**: un registro de stock no se borra, se deja en 0.
4. Escribe la cantidad correcta (para el ejemplo, `15`) y haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Cantidad Física | La cantidad real que hay | Sí | Vacía o negativa: «La cantidad no puede ser negativa» y no se envía nada. El 0 es válido. Se aceptan decimales (comprobado con `2.5`) |

### 10.2 Qué pasa al terminar bien

Aparece «Cantidad ajustada correctamente» y el sistema te lleva a la búsqueda de stock (no te quedas en el detalle), donde la fila muestra la cantidad nueva. **Cancelar** descarta lo escrito y deja la cantidad anterior (comprobado).

Comprobado además:

- **Dejar un registro en 0.** Se ajustó el stock de `LOT-AMOX-2027` en la Sucursal Demo Norte a 0: con la casilla «Solo con stock disponible» marcada el registro **desaparece** de la lista, y con la casilla desmarcada aparece con cantidad 0. Después se volvió a ajustar a 20 para dejar el dato como en la sección 4. Según el inventario (`RN-LOT-014`), un registro en 0 **sigue impidiendo eliminar el lote** (sección 11; No verificado en la aplicación con un lote en 0).
- **Decimales.** Se ajustó una cantidad a `2.5` y quedó guardada como 2,5.

Si no ves **Editar**, a tu rol le falta `Inventario › Actualizar`. Otros avisos posibles (No verificado en la aplicación: no se pudieron provocar; vienen de 19e §2.3): «Error al ajustar la cantidad», «No tienes permiso para operar el stock de esta sucursal», «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» y, si el detalle no abre, «Error al cargar el registro de stock».

## 11. Eliminar un lote

SOFIA no deja eliminar un lote del que dependen otros registros. Se comprobó con los dos casos.

### 11.1 Un lote de prueba sin stock

Se creó el lote de prueba `LOT-PRUEBA-BORRAR` (Ibuprofeno Demo 400 mg, sin fecha de fabricación, caducidad 30/10/2026, es decir, el día del recorrido) y se editó a `LOT-PRUEBA-EDITADO` (sección 7). Para eliminarlo:

1. Abre el lote desde la búsqueda y haz clic en **Editar**.
2. Haz clic en **Eliminar** (permiso `Inventario › Eliminar`). Se abre «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este lote?» y los botones **Cancelar** y **Confirmar**.
3. **Cancelar** cierra el cuadro y no borra nada (comprobado).
4. **Confirmar**: aparece «Lote eliminado correctamente» y vuelves a la búsqueda, donde el lote ya no aparece.

También se eliminó el segundo lote de prueba (un `LOT-PARA-2027` del Omeprazol Demo 20 mg, creado solo para comprobar que **el mismo número de lote sí se puede usar en otro producto**): mismo aviso «Lote eliminado correctamente».

### 11.2 Un lote con stock

Se intentó eliminar el lote `LOT-PARA-2027` del Paracetamol, que ya tiene stock en dos sucursales: aviso rojo **«No se puede eliminar un lote con stock o ventas registradas.»** Te quedas en la ficha y el lote no se borra. Para que un lote se pueda eliminar **no debe tener ningún registro de stock en ninguna sucursal** (aunque la cantidad sea 0) **ni líneas de venta** (la parte «aunque la cantidad sea 0» viene de `RN-LOT-014` y no se comprobó en la aplicación). Como la pantalla no tiene forma de borrar un registro de stock, un lote al que ya le ingresaste stock **no se podrá eliminar desde la aplicación**: si el lote quedó mal, ajusta su stock a 0 y deja de usarlo.

Cualquier otro fallo al eliminar muestra «Error al eliminar el lote» (No verificado en la aplicación: no se pudo provocar). Eliminar un lote **libera su número**: puedes volver a usarlo en ese producto (No verificado en la aplicación: no se recreó el número; viene de `RN-LOT-010` y `RN-LOT-015`).

Tras la sección 11.1, quedan los siete lotes de la sección 4.

## 12. Transferencias entre sucursales

Mover stock de una sucursal a otra (transferencias) **es solo API (sin pantalla)**: la aplicación no tiene ninguna pantalla ni botón para hacerlo, por lo que este manual no lo documenta. Para pasar mercadería entre sucursales hoy debes ajustar el stock de cada sucursal (sección 10) o ingresarlo (sección 8). Las reglas del módulo están en `01d-modulos-solo-api.md` (`RN-TRA`), documento de referencia para el equipo técnico.

## 13. Lo que ve un cajero frente a un administrador

Se entró al sistema con el cajero del manual (`carlos.cajero`, rol «Cajero»), que solo tiene `Inventario › Leer` y `Medicamentos › Leer` entre los permisos de este capítulo (capítulo 3, sección del rol «Cajero»). Su sucursal base es la «Sede Principal» y no tiene sucursales adicionales.

| Qué | Administrador | Cajero (comprobado) |
|---|---|---|
| Menú INVENTARIO | «Lotes» y «Stock por Sucursal» | Las dos opciones |
| Búsqueda de lotes | Filtros, tabla, **Nuevo** y **Exportar** | Los mismos filtros y la misma tabla (siete lotes) y **Exportar**; **no ve Nuevo** |
| Detalle de un lote | **Editar** y, en modo editar, **Guardar** y **Eliminar** | Ve los cuatro datos deshabilitados; **no ve Editar** (solo la flecha de volver) |
| Búsqueda de stock | Stock de **todas** las sucursales (nueve filas) y **Nuevo** | Solo stock de la «Sede Principal» (siete filas, con las mismas cantidades); **no ve Nuevo**; sí **Exportar** |
| Filtro «Sucursal» del stock | Ofrece las dos sucursales | Solo ofrece «Sede Principal». Escribir «norte» da «Sin resultados» en la lista de sugerencias y «Resultados (0)» al buscar |
| Detalle de un registro de stock | **Editar** y ajuste | «Detalle de Stock» con los cinco datos deshabilitados; **no ve Editar** |
| Ingresar y ajustar stock; crear, editar y eliminar lotes | Sí | No: le faltan `Inventario › Actualizar`, `Crear` y `Eliminar` |

El cajero consulta lotes y stock de **su** sucursal para saber qué hay antes de vender; no puede cambiar nada. (No verificado en la aplicación: la redirección con «No tienes permisos para acceder a esta pantalla.» al escribir `/lotes/nueva` o `/stock-por-sucursal/nueva` a mano con la sesión del cajero. No se probó porque el navegador integrado bloqueó esa navegación en el capítulo 4; viene de 19e §0.)

## 14. Diferencias con la documentación técnica

Se compararon `19e-inventario-y-atencion.md` (§0, §1, §2, §6) y `01c1-inventario-y-recetas.md` (`RN-LOT`, `RN-STK`) con lo que hace la aplicación. Coincidieron en todo lo comprobado, con estas confirmaciones y matices:

| # | Documento y apartado | Dice | La aplicación muestra |
|---|---|---|---|
| 1 | 19e §2.2 y §7 (hallazgo 1) | **Guardar** sin lote elegido lanza un error en la consola y no hace nada visible | Confirmado. Sin aviso, sin petición y sin mensaje bajo los campos. La consola del navegador muestra `TypeError: Cannot read properties of null (reading 'numeroLoteMfr')` en `StockDetailComponent.loteDisplay` |
| 2 | 19e §2.2 nota 1 y §7 (hallazgo 2) | Los textos «Debes seleccionar una sucursal» y «Debes seleccionar un lote» no se pintan nunca | Confirmado: con el formulario vacío y al cambiar el texto del lote, ningún mensaje bajo los campos |
| 3 | 19e §1.2 (mapa de respuestas) y §7 (hallazgo 3) | Un número de lote duplicado devuelve 400 y se ve el aviso genérico, no el de duplicado | Confirmado: «Error al guardar el lote». La respuesta trae el código `LoteInventario.Duplicate` (400), y el duplicado se detecta también con otras mayúsculas. El aviso «Ya existe un lote con ese número para el producto seleccionado» no se vio |
| 4 | 19e §1.2 (Observaciones) | Caducidad vencida y fabricación futura se rechazan con el aviso genérico | Confirmado: caducidad de ayer y fabricación futura dan «Error al guardar el lote»; una caducidad de hoy se acepta (coincide con `RN-LOT-006`) |
| 5 | 19e §1.2 (campos) | El producto solo se elige al crear; en editar queda deshabilitado | Confirmado |
| 6 | 19e §1.2 (Comportamientos particulares) | Al editar con éxito navega a la búsqueda | Confirmado, y vuelve con los filtros que tenías puestos |
| 7 | 19e §2.3 | Ajustar fija la cantidad y navega a la búsqueda | Confirmado. Además se comprobó que se aceptan decimales (`2.5`) y que 0 es válido (19e §7 hallazgo 12 solo hablaba del ingreso) |
| 8 | 01c1 `RN-STK-008` | El ingreso suma a la cantidad existente | Confirmado: 100 + 100 = 200 |
| 9 | 19e §1.1 (autocompletar) | Sin coincidencias muestra una opción deshabilitada «Sin resultados» | Confirmado, y además aparece **mientras la consulta está en curso** (un segundo o dos): se ve «Sin resultados» y luego llega el nombre. No es un error |
| 10 | 19e §1.2 (campos) | El campo «Producto» es obligatorio | Confirmado el mensaje; en pantalla las etiquetas «Número de Lote*» y «Fecha de Caducidad*» llevan asterisco y «Producto» no |
| 11 | 19e §6 (Lotes) y 01c1 `RN-LOT-014` | No se puede eliminar un lote con stock (cualquier cantidad) o ventas | Confirmado el caso con stock: «No se puede eliminar un lote con stock o ventas registradas.». El caso con ventas: No verificado (todavía no hay ventas) |
| 12 | 19e §2.1 (dashboard) | El dashboard abre «Stock por Sucursal» con la caducidad ya filtrada | Confirmado: la tarjeta de 30 días abre la pantalla con «Caducidad Desde» = hoy y «Caducidad Hasta» = hoy + 29 días |

## 15. Pendientes y «No verificado»

**En curso (sin commitear):** ninguno. Al empezar, `sofia-apis` estaba en `34089ec` y `sofia-web-spa` en `9ad15f0` (ambos más nuevos que la línea base `43f5529` y `eeffbb6`), y `git status --short` no mostró cambios locales. No se comparó el código de lotes y stock entre las dos líneas base; lo que afirma este capítulo sobre pantallas está comprobado en la aplicación corriendo sobre esos commits.

**No verificado en la aplicación** (cada caso, con su motivo):

| # | Qué | Motivo |
|---|---|---|
| 1 | El aviso «Ya existe un lote con ese número para el producto seleccionado» (dos altas simultáneas del mismo número) | No se pudo provocar una carrera entre dos usuarios |
| 2 | Que la fabricación posterior a la caducidad se rechace al crear (`RN-LOT-008`) | Con una caducidad vigente y una fabricación no futura, la combinación no se puede formar en pantalla |
| 3 | Editar un lote ya vencido (el inventario dice que no se puede guardar) | SOFIA no deja crear lotes vencidos y el lote de prueba que vencía el día del recorrido seguía vigente |
| 4 | Eliminar un lote con ventas registradas | Todavía no hay ventas; se comprobó solo el bloqueo por stock |
| 5 | Que eliminar un lote libere su número de lote | No se volvió a crear un número eliminado; viene de `RN-LOT-010` y `RN-LOT-015` |
| 6 | Los avisos «Error al buscar lotes», «Error al cargar el lote», «Error al eliminar el lote», «Error al consultar el stock», «No se pudieron cargar tus sucursales», «Error al registrar el stock», «Error al ajustar la cantidad», «Error al cargar el registro de stock» | Son errores de servidor o de red que no se pudieron provocar |
| 7 | «No tienes permiso para operar el stock de esta sucursal» y «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» al ingresar y ajustar | Se necesitaría un usuario sin acceso a una sucursal con permiso `Actualizar`, o dos usuarios a la vez; no se creó un rol de prueba (no se cambian permisos de roles) |
| 8 | Una cantidad decimal al **ingresar** stock (al ajustar sí se comprobó) | No se probó en el formulario de ingreso |
| 9 | Elegir una sugerencia con el teclado (flechas y Enter) en los autocompletar | Se eligió siempre con clic sobre la opción |
| 10 | El nombre del archivo descargado (`lotes.xlsx`, `stock-por-sucursal.xlsx`) | El navegador integrado no muestra las descargas; el contenido (hoja «Data», encabezados y filas) se leyó interceptando el archivo que genera la pantalla; el nombre en disco viene de las claves `lotes.search.export-filename` y `stockPorSucursal.search.export-filename` |
| 11 | Paginación con más de 10 filas, paginadores de 25 y 50, y «Buscar» sin volver a la página 1 | Hay solo siete lotes y nueve registros de stock |
| 12 | Redirección del cajero al escribir a mano `/lotes/nueva` o `/stock-por-sucursal/nueva` | El navegador integrado bloqueó esa navegación con la sesión del cajero en el capítulo 4 |
| 13 | El selector de calendario (ventana emergente) al elegir fechas | Se escribieron las fechas a mano en el campo; no se abrió el calendario |
| 14 | Qué pasa si falta `Medicamentos › Leer` al escribir en «Producto» | No se creó un rol de prueba |
| 15 | Eliminar un lote con ventas o transferencias, cuarentena, inmunizaciones y órdenes magistrales | No existen datos de esos módulos (algunos no tienen pantalla) |
| 16 | Eliminar un lote cuyo stock quedó en 0 (si queda bloqueado o no) | No se probó para no dejar un lote de prueba en 0 sin poder borrarlo; viene de `RN-LOT-014` |

**Hallazgos de la aplicación que conviene conocer** (no son errores del manual):

- **Guardar** un ingreso de stock sin elegir lote o sucursal no muestra nada: ni aviso ni mensaje (error de la pantalla, sección 8.3).
- El aviso de lote duplicado, de caducidad vencida y de fabricación futura es siempre el mismo, «Error al guardar el lote»; la pantalla no dice cuál fue la causa.
- Un lote al que se le ingresó stock no se puede eliminar desde la aplicación: ninguna pantalla borra registros de stock y, según el inventario, ni dejándolo en 0 se libera (no verificado con un lote en 0).
- Al editar un lote vuelves a la búsqueda (no al modo ver), distinto de los catálogos del capítulo 5.
- La tabla de lotes y la de stock **no marcan** los lotes vencidos; solo el Dashboard cuenta los vencidos y los que están por vencer.
- Transferencias entre sucursales: sin pantalla.

**Datos de prueba que deja este capítulo:** siete lotes, nueve registros de stock (secciones 4 y 8.2). El lote `LOT-PARA-CERCA` vence el 15/11/2026: **no lo uses en capítulos posteriores si necesitas un lote vigente**.

## 16. Tabla de cobertura

Cada elemento de las fichas de `19e-inventario-y-atencion.md` (§1 Lotes, §2 Stock por sucursal, y de §0 y §6 lo que les toca) y las reglas `RN-LOT` y `RN-STK` de `01c1`. Convención: (V) = comprobado en la aplicación.

### 16.1 Lotes — búsqueda `/lotes` (19e §1.1)

| Elemento | Apartado |
|---|---|
| Título «Lotes», migas «Inicio / Inventario / Lotes», tarjeta «Búsqueda de Lotes» y «Ingresa los criterios de búsqueda» | 5 (V) |
| Filtro «Producto» (lupa, «Buscar producto...», «Sin resultados», filtra por el texto) | 5.1, 6.1, 6.3 (V) |
| Filtro «Número de Lote» («contiene») | 6.1, 6.3 (V) |
| Filtros «Caducidad Desde» y «Caducidad Hasta» (inclusivos) | 6.1, 6.3 (V) |
| Filtros «Fabricación Desde» y «Fabricación Hasta» (excluyen lotes sin fabricación) | 6.1, 6.3 (V) |
| «Desde» posterior a «Hasta» sin validación | 6.1, 6.3 (V) |
| Tabla: «Producto» (enlace), «Número de Lote», «Fecha de Fabricación» («—» si no hay), «Fecha de Caducidad» | 6.2 (V) |
| Orden por creación descendente, «Resultados (N)», «Sin resultados», paginador | 6.2, 6.3 (V; varias páginas: No verificado) |
| Sin indicador de lote vencido ni por vencer | 6.2, 9.4 (V) |
| Botones **Nuevo**, **Limpiar**, **Buscar**, **Exportar** | 5, 6.3, 6.4 (V) |
| Exportación: cuatro encabezados, `lotes.xlsx`, respeta filtros; sin carga masiva | 6.4 (V el contenido; el nombre en disco: No verificado) |
| «Error al buscar lotes» (búsqueda y exportación) | 6.3, 6.4 (No verificado: no se pudo provocar) |
| Restauración de filtros al volver | 5.2, 6.3, 7 (V) |
| Permiso `Inventario › Leer` y menú INVENTARIO | 2, 13 (V) |

### 16.2 Lotes — detalle `/lotes/nueva` y `/lotes/:id` (19e §1.2)

| Elemento | Apartado |
|---|---|
| Modos crear, ver y editar; «Datos del registro» | 5.1, 7 (V) |
| Campo «Producto» (elegir de la lista, deshabilitado al editar) | 5.1, 7 (V) |
| Campo «Número de Lote» (obligatorio, máximo 100) | 5.1, 5.3 (V) |
| Campo «Fecha de Fabricación» (opcional, no futura) | 5.1, 5.3 (V) |
| Campo «Fecha de Caducidad» (obligatoria, no vencida) | 5.1, 5.3 (V) |
| Botones **Editar**, **Cancelar**, **Guardar**, **Eliminar**, volver | 5, 7, 11 (V) |
| Cuadro «Confirmar eliminación» y «¿Estás seguro de que deseas eliminar este lote?» | 11.1 (V) |
| «Lote guardado correctamente» | 5.2, 7 (V) |
| «Ya existe un lote con ese número para el producto seleccionado» | 5.3 (No verificado: dos altas simultáneas) |
| «Error al guardar el lote» (duplicado, vencida, fabricación futura, solo espacios, número ajeno al editar) | 5.3, 7 (V) |
| «Lote eliminado correctamente» | 11.1 (V) |
| «No se puede eliminar un lote con stock o ventas registradas.» | 11.2 (V el caso con stock; con ventas: No verificado) |
| «Error al eliminar el lote», «Error al cargar el lote» | 7, 11.2 (No verificado) |
| Mapa de respuestas de la API (400 duplicado, 400 validador, 400 entidad, 404, 409) | 5.3, 14 (V el 400 por duplicado y por validador; los demás: No verificado) |
| Editar un lote vencido | 7 (No verificado) |

### 16.3 Stock por sucursal — búsqueda `/stock-por-sucursal` (19e §2.1)

| Elemento | Apartado |
|---|---|
| Título, migas «Inicio / Inventario / Stock por Sucursal», tarjeta «Consulta de Stock por Sucursal» | 9.1 (V) |
| Filtros «Sucursal» y «Producto» (sugerencias, «contiene») | 9.1, 9.2 (V) |
| Filtro «Número de Lote» | 9.1, 9.2 (V) |
| Filtros «Caducidad Desde» y «Caducidad Hasta» | 9.1, 9.2 (V) |
| Filtros «Cantidad Mínima» y «Cantidad Máxima» (numéricos, inclusivos) | 9.1, 9.2 (V) |
| Casilla «Solo con stock disponible» (marcada al abrir y tras **Limpiar**) | 9.1, 9.2, 10.2 (V) |
| Tabla: «Sucursal» (enlace), «Producto», «Número de Lote», «Fecha de Caducidad», «Cantidad Física» | 9.1 (V) |
| Orden, «Resultados (N)», «Sin resultados», paginador | 9.1, 9.2 (V; varias páginas: No verificado) |
| Botones **Nuevo** (con `Actualizar`), **Limpiar**, **Buscar**, **Exportar** | 8, 9.1, 9.3 (V) |
| Exportación: cinco encabezados, `stock-por-sucursal.xlsx`, respeta filtros; sin carga masiva | 9.3 (V el contenido; el nombre en disco: No verificado) |
| «Error al consultar el stock», «No se pudieron cargar tus sucursales» | 9.2 (No verificado) |
| Solo sucursales permitidas (el administrador ve todas) | 9.1, 13 (V) |
| Enlace desde el Dashboard con la caducidad filtrada | 9.4 (V) |

### 16.4 Stock por sucursal — ingreso `/stock-por-sucursal/nueva` (19e §2.2)

| Elemento | Apartado |
|---|---|
| Cabecera, migas, flecha de volver; permiso `Leer` + `Actualizar` | 8, 2 (V el botón; la redirección por falta de permiso: No verificado) |
| Campo «Sucursal» (elegir de la lista, «Buscar sucursal...») | 8.1 (V) |
| Campo «Lote» («Buscar por número de lote...», opción con producto debajo, «número — producto») | 8.1 (V) |
| Campo «Cantidad Física» (empieza en 0; «La cantidad no puede ser negativa» también vacío) | 8.1, 8.3 (V) |
| «Debes seleccionar una sucursal» y «Debes seleccionar un lote» (no se ven) | 8.3, 14 (V) |
| **Guardar** sin lote elegido: nada visible | 8.3, 14 (V) |
| Botones **Cancelar**, **Guardar**, volver | 8 (V) |
| «Stock registrado correctamente» y suma al existente | 8.2 (V) |
| «No tienes permiso para operar el stock de esta sucursal», «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.», «Error al registrar el stock» | 8.3 (No verificado) |
| El ingreso admite lotes vencidos | 8.1 (No verificado en la aplicación: viene de `RN-STK-007`) |

### 16.5 Stock por sucursal — detalle y ajuste `/stock-por-sucursal/:id` (19e §2.3)

| Elemento | Apartado |
|---|---|
| «Detalle de Stock», «Datos del registro», cinco campos | 10.1 (V) |
| Botones **Editar**, **Cancelar**, **Guardar**, volver; no hay **Eliminar** | 10.1 (V) |
| Solo «Cantidad Física» editable; «La cantidad no puede ser negativa» | 10.1 (V) |
| «Cantidad ajustada correctamente» y reemplazo de la cantidad | 10.2 (V) |
| «Error al ajustar la cantidad», «Error al cargar el registro de stock», «No tienes permiso para operar el stock de esta sucursal», «Otra operación modificó estos datos al mismo tiempo. Actualiza la pantalla e inténtalo de nuevo.» | 10.2 (No verificado) |

### 16.6 Comportamientos comunes y reglas

| Elemento | Apartado |
|---|---|
| 19e §0: permisos de ruta, menú, estado de búsqueda, resultados, fechas `dd/MM/aaaa`, autocompletar, modo ver/editar, tras guardar, mensaje obligatorio, diálogo de eliminar, exportación | 2, 5 a 11 (V; el teclado en autocompletar y «Buscar» sin volver a la página 1: No verificado) |
| 19e §6 (Lotes y Stock): mapa de eliminar; Stock sin eliminar | 11, 10.1 (V) |
| `RN-LOT-001` a `RN-LOT-007`, `RN-LOT-009`, `RN-LOT-010`, `RN-LOT-012`, `RN-LOT-016`, `RN-LOT-017`, `RN-LOT-019` | 2, 3, 5 a 7 (V las que se pueden provocar) |
| `RN-LOT-008` (fabricación posterior a la caducidad) | 5.3 (No verificado) |
| `RN-LOT-011`, `RN-LOT-013` (identificador y «no encontrado») | No aplica: el usuario no escribe el identificador |
| `RN-LOT-014`, `RN-LOT-015` (eliminar) | 11 (V con stock; con 0, con ventas y liberación del número: No verificado) |
| `RN-LOT-018`, `RN-STK-018` (consultas por producto y por medicamento) | 3, 12: Solo API (sin pantalla); No aplica a este manual |
| `RN-STK-001` a `RN-STK-004`, `RN-STK-008`, `RN-STK-009`, `RN-STK-012`, `RN-STK-014` a `RN-STK-016` | 2, 8, 9, 10 (V) |
| `RN-STK-005` a `RN-STK-007`, `RN-STK-010`, `RN-STK-011`, `RN-STK-013` | 8.3, 10.2 (No verificado: no se pudieron provocar; ver sección 15) |
| `RN-STK-017` (stock vendible) | 3 (definición; el punto de venta se recorre en el capítulo 8) |
| Transferencias entre sucursales | 12 (Solo API, sin pantalla) |

## 17. Fuentes

**Documentos consultados**

- [19e — inventario y atención](../../ola-1-fundamentos/19-inventario-de-pantallas/19e-inventario-y-atencion.md) (§0, §1, §2, §6, §7).
- [01c1 — inventario y recetas](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) (`RN-LOT-001` a `RN-LOT-019`, `RN-STK-001` a `RN-STK-018`).
- [01d — módulos solo API](../../ola-1-fundamentos/01-reglas-de-negocio/01d-modulos-solo-api.md) (Transferencias, `RN-TRA`), solo para la remisión de la sección 12.
- [Capítulo 3](capitulo-03-empleados-roles-cuentas.md), [capítulo 4](capitulo-04-catalogos-basicos.md) y [capítulo 5](capitulo-05-productos-y-proveedores.md) del manual (datos de prueba y rol «Cajero»).
- Textos de la interfaz: `sofia-web-spa/public/i18n/lotes/es.json`, `public/i18n/stock-por-sucursal/es.json`, `public/i18n/roles/es.json` (nombre del módulo «Inventario») y `public/i18n/es.json`.
- Código del menú y del enlace del Dashboard: `sofia-web-spa/src/app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts`, `sofia-web-spa/src/app/pages/dashboard/dashboard-links.ts`.

**Recorridos hechos en la aplicación** (SPA `http://localhost:4200`, API `https://localhost:7300`; administrador `ana.manual` y cajero `carlos.cajero`)

1. Lotes › Buscar sin lotes (cero resultados) y entrada a **Nuevo**.
2. Lotes › Nuevo: **Guardar** vacío; producto escrito sin elegir; caducidad de ayer; fabricación futura; número duplicado (mayúsculas iguales y distintas); número solo de espacios; longitud máxima del campo.
3. Lotes › Nuevo: creación de los siete lotes de la sección 4, de un lote de prueba con caducidad de hoy y sin fabricación y de un lote de prueba con número repetido en otro producto.
4. Lotes › Buscar: filtros por producto, número, caducidad y fabricación; sin coincidencias; «Desde» posterior a «Hasta»; **Limpiar**.
5. Lotes › Detalle: ver, **Editar**, **Cancelar**, número duplicado al editar, guardado correcto con fecha de fabricación nueva.
6. Lotes › Exportar (lectura del archivo generado).
7. Lotes › Eliminar: cuadro de confirmación con **Cancelar**; eliminación correcta de dos lotes de prueba; intento de eliminar un lote con stock.
8. Stock por Sucursal › Buscar sin stock (cero resultados).
9. Stock por Sucursal › Nuevo: **Guardar** sin elegir nada y con el texto del lote cambiado (consola del navegador); cantidad negativa y vacía; nueve ingresos de stock, uno repetido para comprobar la suma.
10. Stock por Sucursal › Buscar: sin filtros y con filtros de sucursal, producto, lote, caducidad y cantidad; casilla «Solo con stock disponible»; **Limpiar**; **Exportar** (lectura del archivo generado).
11. Stock por Sucursal › Detalle: ver, **Editar**, cantidad negativa, **Cancelar**, ajuste a 15, a 2.5, a 0 y a 20.
12. Dashboard › tarjetas «Inventario» › **Ver lotes** (30 días o menos).
13. Medicamentos › detalle de `DEMO-MED-0001` › tarjeta «Stock por Sucursal».
14. Cajero `carlos.cajero`: menú, Lotes (búsqueda, detalle, **Exportar**), Stock por Sucursal (búsqueda, filtro «Sucursal», detalle) y cierre de sesión; después se volvió a iniciar sesión como administrador.
