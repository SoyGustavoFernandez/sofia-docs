# Capítulo 2 — Configurar tu empresa: datos de la empresa, sucursales y series fiscales

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-14 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Recorrido en la aplicación local más inventario de pantallas |

## 1. Introducción

En este capítulo aprendes a dejar lista la configuración básica de tu botica o farmacia antes de vender. Vas a:

- Revisar y editar los datos de tu empresa (nombre y RUC).
- Completar la sede principal, que el registro creó con el número de licencia vacío.
- Crear más sucursales, buscarlas, exportarlas a Excel y eliminarlas.
- Conocer la carga masiva de sucursales desde un archivo Excel.
- Entender qué son las series fiscales y crear las que necesita el punto de venta: una serie de **Boleta** y una de **Nota de crédito** por sucursal.
- Editar, desactivar y eliminar una serie.

Es para el **administrador** (el rol «Admin», que lo puede todo). Un usuario con otro rol, por ejemplo un cajero, solo ve estas pantallas si el administrador le dio permiso; la sección 12 explica qué cambia.

Todos los ejemplos usan los datos ficticios del manual: la empresa «Botica Manual Demo S.A.C.» (RUC `20538291746`) y la administradora «Ana Demo Prueba». Las direcciones y los números de licencia de los ejemplos son inventados.

## 2. Antes de empezar

- Haz el capítulo 1: debes tener tu empresa registrada y la sesión iniciada como administrador.
- Permisos que usa este capítulo. Los nombres son los que ves en la pantalla de permisos de un rol (Administración › Roles & Permisos): primero el módulo y luego la acción.

| Tarea | Módulo | Acciones |
|---|---|---|
| Ver y buscar tu empresa, exportar | Empresa | `Leer` |
| Editar el nombre y el RUC de tu empresa | Empresa | `Actualizar` |
| Ver, buscar y exportar sucursales | Sucursales | `Leer` |
| Crear una sucursal y usar la carga masiva | Sucursales | `Crear` (además de `Leer`) |
| Editar una sucursal | Sucursales | `Actualizar` |
| Eliminar una sucursal | Sucursales | `Eliminar` |
| Ver, buscar y exportar series | Series Fiscales | `Leer` |
| Crear una serie | Series Fiscales | `Crear` (además de `Leer`) |
| Editar o desactivar una serie | Series Fiscales | `Actualizar` |
| Eliminar una serie | Series Fiscales | `Eliminar` |

- El menú «Empresa» y «Sucursales» están en el grupo **ADMINISTRACIÓN** del menú lateral; «Series Fiscales» está en el grupo **SUNAT** (capítulo 1, sección 4.1).
- Los avisos de la aplicación (verdes de éxito o rojos de error) aparecen abajo al centro de la pantalla con un botón **Cerrar** y desaparecen solos a los pocos segundos. Los mensajes de este capítulo están copiados tal como se vieron en pantalla.

## 3. Revisar y editar los datos de tu empresa

**Cómo llegar:** menú lateral › ADMINISTRACIÓN › **Empresa** (`/empresas`).

### 3.1 Buscar tu empresa

La pantalla se llama «Empresas». Arriba, a la derecha, están las migas «Inicio / Administración / Empresa». Debajo hay una tarjeta «Búsqueda de Empresas» con el texto «Ingresa los criterios de búsqueda» y estos filtros:

| Filtro | Qué hace |
|---|---|
| Nombre de la Empresa | Texto con una lupa a la derecha. Busca las empresas cuyo nombre contenga lo que escribas (sin importar mayúsculas: «manual» encuentra «Botica Manual Demo S.A.C.»). Con **Enter** también busca |
| Estado | Lista con las opciones «Todos», «Trial Activo», «Activo», «Suspendido» y «Cancelado». Empieza en blanco, que es lo mismo que «Todos» |
| Vencimiento desde | Fecha. Solo se elige con el calendario (el campo no se puede escribir); se ve como `20/10/2026` |
| Vencimiento hasta | Igual que el anterior |

Abajo a la derecha de la tarjeta están **Limpiar** y **Buscar**.

Pasos:

1. Escribe lo que quieras en los filtros, o déjalos en blanco para ver todo.
2. Haz clic en **Buscar** (o presiona **Enter** dentro del campo del nombre). La respuesta puede tardar un par de segundos.
3. Bajo la tarjeta aparece «Resultados (N)» con la tabla.

Los resultados no aparecen al abrir la pantalla: hay que hacer clic en **Buscar**. Como tu usuario solo ve su propia empresa, el resultado es siempre una sola fila: «Botica Manual Demo S.A.C.».

La tabla tiene estas columnas:

| Columna | Qué muestra |
|---|---|
| Empresa | El nombre. Es un enlace: haz clic para abrir el detalle |
| RUC | Los 11 dígitos |
| Estado | «Trial Activo», «Activo», «Suspendido» o «Cancelado» |
| Vigente | Un icono: una marca verde de aprobado si la empresa está vigente (puede entrar y operar) o una equis si no lo está. No lleva texto |

Bajo la tabla, a la izquierda, está el botón **Exportar** (con un icono de descarga) y, a la derecha, el paginador: «Items por página» (10, 25 o 50) y «Página 1 de 1», con los botones de primera, anterior, siguiente y última página.

Comprobado en la aplicación:

- Con el estado «Trial Activo» aparece tu empresa. Con «Activo» no aparece: se ve «Resultados (0)», la fila vacía dice solamente «Resultados» y el paginador dice «Sin resultados». Tu empresa está en «Trial Activo» porque es una prueba gratuita de 1 mes.
- Con las dos fechas en `20/10/2026` tampoco aparece (la empresa vence el 13/11/2026), y con «Vencimiento desde» en `20/10/2026` sí aparece.
- **Limpiar** vacía los cuatro filtros y oculta la tabla.

**Exportar:** descarga un archivo `empresas.xlsx` con una hoja «Data» y las columnas «Empresa», «RUC», «Estado», «Vigente» (con «Sí» o «No»), «Inicio Trial», «Vencimiento» (ambas como `dd/MM/yyyy`) y «Sucursales» (la cantidad). Solo aparece después de buscar, y exporta con los filtros que tengas escritos en ese momento. Comprobado en la aplicación: al hacer clic en **Exportar** el servidor respondió correctamente y la pantalla no mostró ningún aviso de error. (No verificado en la aplicación: el contenido del archivo ni su descarga, porque el navegador integrado no muestra los archivos descargados; las columnas y el nombre del archivo vienen del inventario 19b §1.1 y del documento 20 §4.4. Según el documento 20, la columna «Estado» del archivo trae el nombre interno sin traducir, por ejemplo `TrialActivo`.)

Si falla la búsqueda o la exportación: «Error al buscar empresas».

### 3.2 Ver el detalle de tu empresa

1. En los resultados, haz clic en el nombre de la empresa.
2. Se abre el detalle (`/empresas/…`). Arriba ves el botón de volver (una flecha a la izquierda), el título «Empresas» y las migas «Inicio / Administración / Empresa». Debajo hay una tarjeta con el título «Empresas», el texto «Datos del registro» y, a la derecha, el botón **Editar**.

La tarjeta tiene seis campos, todos deshabilitados (solo lectura) mientras no pulses **Editar**:

| Campo | Qué muestra | ¿Se puede cambiar? |
|---|---|---|
| Estado | «Trial Activo» | No, nunca. Lo maneja SOFIA |
| Sucursales | El número de sucursales de tu empresa que no fueron eliminadas (al registrarte, `1`; después de crear «Sucursal Demo Norte» y eliminar las sucursales de prueba, `2`) | No, nunca. Cambia solo al crear o eliminar sucursales |
| Inicio Trial | La fecha en que registraste la empresa (`14/10/2026`) | No, nunca |
| Vencimiento | La fecha en que termina la prueba (`13/11/2026`, 30 días después) | No, nunca |
| Nombre de la Empresa | «Botica Manual Demo S.A.C.» | Sí, con **Editar** |
| RUC (11 dígitos, opcional) | `20538291746` | Sí, con **Editar** |

Solo se pueden editar el nombre y el RUC. No existe pantalla para cambiar el estado de la suscripción, las fechas ni el número de sucursales, ni para crear o eliminar empresas (la empresa se crea una sola vez con el registro del capítulo 1). Tampoco hay en esta pantalla opciones de configuración de la empresa, como la exigencia de registrar la receta en las ventas.

Si no ves el botón **Editar**, a tu rol le falta el permiso `Actualizar` del módulo «Empresa».

### 3.3 Editar el nombre o el RUC

1. En el detalle, haz clic en **Editar**. El nombre y el RUC se habilitan, y abajo a la derecha aparecen **Cancelar** y **Guardar**. Los otros cuatro campos siguen deshabilitados.
2. Cambia lo que necesites.
3. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Nombre de la Empresa | La razón social | Sí | Vacío: «Este campo es obligatorio» (aparece debajo del campo al salir de él o al hacer clic en **Guardar**, y no se envía nada). El campo no deja escribir más de 150 caracteres (se comprobó: de 262 caracteres tecleados quedaron 150) |
| RUC (11 dígitos, opcional) | Exactamente 11 dígitos, por ejemplo `20538291746` | No (pero ver la advertencia) | El campo deja escribir hasta 20 caracteres y la pantalla **no** marca ningún error por el formato. Es el servidor el que lo rechaza si no son 11 dígitos, y entonces ves el aviso «Error al actualizar la empresa» sin más explicación |

**Qué pasa al terminar bien:** aparece el aviso «Empresa actualizada correctamente», la pantalla vuelve al modo ver (campos deshabilitados, botón **Editar**) y te quedas en el detalle, con los datos recargados. No te lleva a la búsqueda.

**Cancelar** descarta lo que escribiste, restaura el nombre y el RUC que tenía la empresa y vuelve al modo ver.

Errores que puedes ver (todos comprobados en la aplicación):

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Este campo es obligatorio» (bajo el nombre) | Dejaste el nombre vacío | Escribe el nombre |
| «Error al actualizar la empresa» al guardar un RUC de 10 dígitos | El servidor exige exactamente 11 dígitos numéricos | Corrige el RUC |
| «Error al actualizar la empresa» al guardar un RUC con una letra (11 caracteres, uno de ellos `X`) | Igual: solo se aceptan dígitos | Escribe solo números |
| «Error al actualizar la empresa» al guardar el RUC de otra empresa ya registrada en SOFIA | El RUC es único en todo el sistema. La pantalla no distingue este caso de los demás | Revisa que sea tu RUC. Si es el tuyo y otra empresa lo usa, comunícate con soporte |
| «Error al actualizar la empresa» al guardar con el RUC vacío | El servidor no acepta un RUC vacío; el RUC no cambia | Vuelve a escribir los 11 dígitos o haz clic en **Cancelar** |

**Importante: no dejes tu empresa sin RUC.** El punto de venta no emite comprobantes si la empresa no tiene RUC: al cobrar, el sistema rechaza la venta con el aviso «Tu empresa no tiene RUC registrado, por lo que no puede emitir comprobantes. Complétalo en Administración > Empresa para poder cobrar.» (viene de la regla RN-CMP-002 y del texto `pos.messages.empresaSinRuc`; el aviso se ve en el capítulo de ventas). En esta versión, desde la pantalla no pudiste quitar el RUC: al guardarlo vacío el servidor lo rechazó y la empresa conservó su RUC `20538291746`. Si por cualquier medio la empresa quedara sin RUC, vuelve a esta pantalla y escríbelo.

Las pruebas con RUC incorrecto se hicieron con la empresa de prueba y no cambiaron ningún dato: al terminar, el nombre y el RUC seguían siendo los originales, y se guardó una vez sin cambios para ver el aviso de éxito.

## 4. Completar la sede principal

Al registrar tu empresa, SOFIA creó por ti una sucursal llamada «Sede Principal» con la dirección que escribiste en el registro y el «Nº de Licencia» **vacío**. La licencia es obligatoria al editar una sucursal, así que el primer paso es completarla.

**Cómo llegar:** menú lateral › ADMINISTRACIÓN › **Sucursales** (`/sucursales`).

### 4.1 La pantalla de sucursales

La pantalla se llama «Sucursales», con las migas «Inicio / Administración / Sucursales». Tiene una tarjeta «Búsqueda de Sucursales» («Ingresa los criterios de búsqueda»). En la esquina superior derecha de la tarjeta hay dos botones, **Carga Masiva** (con un icono de subida) y **Nuevo** (con un signo más); solo los ves si tu rol tiene el permiso `Crear` del módulo «Sucursales». Debajo, tres filtros de texto (cada uno con una lupa), y al pie **Limpiar** y **Buscar**.

### 4.2 Abrir la sede principal

1. Haz clic en **Buscar** (sin escribir nada). Ves «Resultados (1)» con una fila:

| Nombre | Nº de Licencia | Dirección Física | Gerente |
|---|---|---|---|
| Sede Principal | (en blanco) | Av. Ficticia 123, Miraflores, Lima | — |

   La celda de la licencia aparece vacía; el gerente muestra «—» cuando no hay.
2. Haz clic en «Sede Principal».
3. Se abre el detalle (`/sucursales/…`). Arriba: botón de volver, el título «Sucursales» y las migas. En la tarjeta, el título «Sucursales», «Datos del registro» y el botón **Editar** (con el permiso `Actualizar`). Los campos, deshabilitados: «Nombre», «Nº de Licencia», «Dirección Física» y «Gerente».

### 4.3 Editar y guardar con una licencia

1. Haz clic en **Editar**. Los cuatro campos se habilitan y abajo a la derecha aparecen **Cancelar**, **Guardar** y **Eliminar** (este último solo con el permiso `Eliminar`).
2. Prueba guardar sin licencia: haz clic en **Guardar**. Debajo de «Nº de Licencia» aparece «Este campo es obligatorio» y no se guarda nada.
3. Escribe una licencia en «Nº de Licencia». En el ejemplo: `LIC-DEMO-0001` (ficticia).
4. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Nombre | El nombre de la sucursal. Lo puedes cambiar (por ejemplo, de «Sede Principal» a otro) | Sí | Vacío: «Este campo es obligatorio». El campo no deja escribir más de 100 caracteres |
| Nº de Licencia | El número de licencia de funcionamiento de esa sucursal | Sí | Vacío: «Este campo es obligatorio». Máximo 50 caracteres. No puede repetirse en otra sucursal de tu empresa (sin importar mayúsculas): «Ya existe una sucursal con ese número de licencia.» |
| Dirección Física | Calle, número y distrito. Es la dirección que sale en el encabezado de las boletas | Sí | Vacío: «Este campo es obligatorio». Máximo 255 caracteres |
| Gerente | Un empleado de tu empresa que dirige la sucursal | No | Escribe al menos 2 letras del nombre: tras una pausa de medio segundo la lista muestra los empleados que coinciden (en el ejemplo, escribir «Ana» muestra «Ana Demo Prueba»). Solo puedes elegir uno de la lista: si dejas escrito un texto que no corresponde a ningún empleado y sales del campo, el campo se vacía. Cuando la sucursal ya tiene gerente y lo borras, queda sin gerente |

**Qué pasa al terminar:** aparece el aviso «Sucursal guardada correctamente» y la pantalla vuelve al modo ver, en el mismo detalle. El campo «Nº de Licencia» ya muestra `LIC-DEMO-0001`. Si vuelves a la búsqueda, la tabla muestra la licencia.

**Cancelar**, en el modo editar de una sucursal que ya existe, descarta los cambios y vuelve al modo ver.

Si no ves **Editar**, te falta el permiso `Actualizar` del módulo «Sucursales». Para buscar gerente se necesita además el permiso `Leer` del módulo «Empleados».

Errores que puedes ver al guardar una sucursal existente:

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Este campo es obligatorio» | Un campo obligatorio está vacío | Complétalo |
| «Ya existe una sucursal con ese número de licencia.» | Otra sucursal de tu empresa ya usa esa licencia | Escribe la licencia correcta |
| «Error al guardar la sucursal» | Cualquier otro problema del servidor | Revisa los datos e inténtalo de nuevo |
| «Error al cargar la sucursal» | No se pudo abrir el detalle | Vuelve a la búsqueda e inténtalo de nuevo |

(No verificado en la aplicación: «Error al guardar la sucursal» y «Error al cargar la sucursal» no se pudieron provocar; vienen del inventario 19b §2.2. Tampoco se asignó un gerente a la sede principal para no dejar una relación que otros capítulos no esperan.)

## 5. Crear una segunda sucursal

Crea la sucursal «Sucursal Demo Norte», que usarán los capítulos de cuentas, inventario y dashboard.

**Cómo llegar:** **Sucursales** › botón **Nuevo** (`/sucursales/nueva`). Necesitas el permiso `Crear` del módulo «Sucursales».

La pantalla se ve igual que el detalle, pero ya está en modo editar y sin el botón **Eliminar**: título «Sucursales», migas, flecha de volver y tres campos de texto («Nombre», «Nº de Licencia» y «Dirección Física»), el buscador «Gerente» y los botones **Cancelar** y **Guardar**. Los campos y sus reglas son los de la sección 4.3.

### 5.1 Probar los errores antes de guardar

1. Haz clic en **Guardar** con el formulario vacío. Los tres campos muestran «Este campo es obligatorio» y no se envía nada.
2. Escribe más de 100 caracteres en «Nombre» y más de 50 en «Nº de Licencia»: el campo se detiene en 100 y en 50 caracteres.
3. Escribe el nombre «Sucursal Demo Norte», la licencia `LIC-DEMO-0001` (la de la sede principal) y la dirección «Av. Ficticia Norte 456, Los Olivos, Lima», y haz clic en **Guardar**: aparece el aviso rojo «Ya existe una sucursal con ese número de licencia.» y te quedas en la pantalla con lo que escribiste.

### 5.2 Guardar

1. Cambia la licencia por una que no esté usada. En el ejemplo: `LIC-DEMO-0002`.
2. Deja «Gerente» en blanco (es opcional).
3. Haz clic en **Guardar**.

**Qué pasa al terminar:** aparece el aviso «Sucursal guardada correctamente» y el sistema te lleva a la búsqueda de sucursales (`/sucursales`) con los filtros vacíos y **sin** tabla: haz clic en **Buscar** para ver la lista. **Cancelar** en esta pantalla te devuelve a la búsqueda sin guardar nada.

Quedaron estas sucursales en la empresa de prueba:

| Nombre | Nº de Licencia | Dirección Física |
|---|---|---|
| Sede Principal | `LIC-DEMO-0001` | Av. Ficticia 123, Miraflores, Lima |
| Sucursal Demo Norte | `LIC-DEMO-0002` | Av. Ficticia Norte 456, Los Olivos, Lima |

## 6. Buscar sucursales y exportarlas a Excel

**Cómo llegar:** **Sucursales** (`/sucursales`).

### 6.1 Filtros

| Filtro | Qué hace |
|---|---|
| Nombre | Texto con lupa. Busca nombres que contengan lo que escribas |
| Nº de Licencia | Texto con lupa. Busca licencias que contengan lo que escribas, sin importar mayúsculas |
| Dirección Física | Texto con lupa. Busca direcciones que contengan lo que escribas, sin importar mayúsculas |

Los tres filtros se combinan (deben cumplirse todos). Con **Enter** dentro de cualquiera de los tres también se busca. Los filtros no tienen validaciones.

Comprobado en la aplicación:

- **Buscar** sin filtros trae todas las sucursales, ordenadas por nombre: primero «Sede Principal» y luego «Sucursal Demo Norte».
- La licencia `demo-0002` (en minúsculas) trae solo «Sucursal Demo Norte». Si además escribes «miraflores» en «Dirección Física», no trae ninguna: se ve «Resultados (0)», la tabla dice «Sin resultados» y el paginador también.
- **Limpiar** vacía los tres filtros y oculta la tabla.

### 6.2 Tabla de resultados

| Columna | Qué muestra |
|---|---|
| Nombre | El nombre; es un enlace al detalle |
| Nº de Licencia | La licencia (vacía si aún no la completaste) |
| Dirección Física | La dirección |
| Gerente | El nombre del gerente o «—» si no tiene |

Encima de la tabla se lee «Resultados (N)». Debajo hay un paginador con «Items por página» (opciones 10, 25 y 50; empieza en 10), «Página 1 de 1» y los botones de primera, anterior, siguiente y última página. Con tan pocas sucursales siempre hay una sola página: cambiar a 25 muestra «Items por página 25 · Página 1 de 1». Cuando tengas más sucursales que el tamaño de página, usa esos botones para recorrerlas.

Dos detalles del buscador (según el inventario; no se comprobaron porque no hay suficientes sucursales para verlo):

- Al cambiar de filtro y pulsar **Buscar**, la búsqueda conserva la página en la que estabas en lugar de volver a la página 1. Si no ves resultados que esperabas, ve a la primera página.
- Al abrir un detalle y volver con la flecha, la pantalla recuerda tus filtros y la página (solo mientras no recargues el navegador).

### 6.3 Exportar a Excel

1. Haz clic en **Buscar** (el botón **Exportar** solo aparece cuando la tabla ya se mostró).
2. Haz clic en **Exportar** (abajo a la izquierda de la tabla, con un icono de descarga).

El sistema genera un archivo `sucursales.xlsx` con los resultados de los filtros que tengas escritos en ese momento. Trae una hoja «Data» con las columnas «Nombre», «Dirección Física», «Nº de Licencia» y «Gerente» (vacío si no hay), y exporta hasta 10 000 filas.

Comprobado en la aplicación: al hacer clic en **Exportar**, la pantalla envió la petición al servidor y no mostró ningún aviso de error (el navegador integrado no registró el código de respuesta de esta petición; en Empresa y en Series fiscales sí respondió 200). (No verificado en la aplicación: el contenido del archivo ni su descarga, porque el navegador integrado no muestra los archivos descargados; las columnas y el nombre del archivo vienen del inventario 19b §2.1 y del documento 20 §4.20.)

Si algo falla verás el aviso «Error al buscar sucursales» (el mismo de la búsqueda). Si hay más de 10 000 filas la exportación se rechaza y la pantalla muestra ese mismo aviso genérico: aplica filtros para reducir los resultados.

## 7. Eliminar una sucursal

Eliminar una sucursal la oculta de todas las listas (es una eliminación lógica). Para probarlo, crea una sucursal de prueba solo para eliminarla.

### 7.1 Crear la sucursal de prueba

Sigue la sección 5 con estos datos: nombre «Sucursal Prueba Eliminar», licencia `LIC-DEMO-0003` y dirección «Calle Ficticia 789, Surco, Lima». Al guardar ves «Sucursal guardada correctamente» y vuelves a la búsqueda. Haz clic en **Buscar**: ahora hay tres sucursales y la nueva sale al final (orden por nombre).

### 7.2 Eliminarla

**Cómo llegar:** **Sucursales** › **Buscar** › clic en el nombre › **Editar**. El botón **Eliminar** solo existe en el modo editar de una sucursal que ya existe y solo con el permiso `Eliminar` del módulo «Sucursales». Está abajo a la derecha, a la derecha de **Guardar**.

1. En el detalle de «Sucursal Prueba Eliminar», haz clic en **Editar**.
2. Haz clic en **Eliminar**.
3. Se abre un cuadro con el título «Confirmar eliminación», el mensaje «¿Estás seguro de que deseas eliminar esta sucursal?» y los botones **Cancelar** y **Confirmar**.
4. **Cancelar** cierra el cuadro y no borra nada (comprobado: sigues en el modo editar). Haz clic en **Confirmar**.

**Qué pasa al terminar:** aparece el aviso «Sucursal eliminada correctamente» y el sistema te lleva a la búsqueda de sucursales. Al volver a buscar, la sucursal ya no aparece.

### 7.3 Cuando la sucursal está en uso

SOFIA no deja eliminar una sucursal que todavía tiene registros asociados. Se comprueban empleados con esa sucursal como base, cuentas de usuario, roles, stock, sesiones de caja, series fiscales, ventas, transferencias, cuarentena DIGEMID y órdenes magistrales. La «Sede Principal» tiene empleados y cuentas, así que al intentar eliminarla (Editar › **Eliminar** › **Confirmar**) aparece el aviso rojo:

> «No puedes eliminar esta sucursal porque tiene empleados, usuarios, roles, stock, ventas u otros registros asociados.»

(Comprobado en la aplicación con la «Sede Principal».) La sucursal no se borra y te quedas en el modo editar. Para poder eliminarla tendrías que quitarle primero lo que la usa: pasar los empleados y cuentas a otra sucursal, quitar el rol de la sucursal, eliminar las series que tenga, etc. El aviso no dice cuál de esas cosas es la que la bloquea.

Cualquier otro fallo al eliminar muestra «Error al eliminar la sucursal» (No verificado en la aplicación: no se pudo provocar; viene de 19b §2.2).

## 8. Cargar sucursales desde un Excel (carga masiva)

Sirve para crear muchas sucursales de una vez en lugar de una por una. Es útil si tienes una cadena con varios locales.

**Cómo llegar:** **Sucursales** › botón **Carga Masiva** (`/sucursales/carga-masiva`). Necesitas el permiso `Crear` del módulo «Sucursales». Sin ese permiso no ves el botón y, si escribes la dirección a mano, el sistema te devuelve al Inicio con el aviso «No tienes permisos para acceder a esta pantalla.» (No verificado en la aplicación: el administrador tiene todos los permisos; viene de 19b §0.)

### 8.1 La pantalla

Arriba, la flecha de volver, el título «Carga Masiva» y las migas «Inicio / Administración / Sucursales / Carga Masiva». Debajo, una tarjeta con:

- El botón **Descargar plantilla** (con un icono de descarga), arriba a la derecha.
- Una zona punteada con un icono de archivo y el texto «Arrastra tu archivo Excel aquí o haz clic para seleccionar», y debajo «Solo archivos .xlsx (máx. 5 MB y 5000 filas)».

### 8.2 Flujo completo

1. **Descarga la plantilla.** Haz clic en **Descargar plantilla**. El archivo se llama `plantilla-sucursales.xlsx` y trae una sola hoja llamada «Plantilla» con una fila de títulos (azul, en negrita) y ninguna fila de ejemplo ni listas desplegables. Las tres columnas, escritas exactamente así y sin tildes, son:

| Columna en el Excel | Qué escribir | Obligatoria | Límite |
|---|---|---|---|
| `Nombre` | El nombre de la sucursal | Sí | 100 caracteres |
| `DireccionFisica` | La dirección | Sí | 255 caracteres |
| `NumeroLicencia` | El número de licencia | Sí | 50 caracteres |

   No hay columna para el gerente: las sucursales de la carga masiva quedan sin gerente (puedes asignarlo después con **Editar**). Si cambias el nombre de una columna (por ejemplo, con tilde) no se reconoce y esa columna se lee como vacía. El orden de las columnas no importa y las columnas extra se ignoran.
2. **Llena el Excel.** Una sucursal por fila, desde la fila 2. Guárdalo como `.xlsx`.
3. **Elige el archivo.** Arrástralo a la zona punteada o haz clic en ella. La pantalla comprueba primero en tu navegador que el nombre termine en `.xlsx` y que pese menos de 5 MB.
4. **Espera la vista previa.** Mientras el sistema lee el archivo ves «Procesando archivo...» (No verificado en la aplicación: con archivos pequeños la espera es muy breve y no se alcanzó a ver; viene de 19a §6.5). No se guarda nada todavía.
5. **Revisa la vista previa.** Aparece una tarjeta «Vista previa» con los contadores «N filas totales», «N filas válidas» y «N filas con error» (este último solo si hay errores) y una tabla con las columnas «Nombre», «Dirección Física», «Nº de Licencia», una columna sin título con los errores de cada fila y la columna «Estado»: una marca verde si la fila es válida o un icono de error si no lo es. La tabla no tiene paginación: se ven todas las filas.
6. **Corrige y vuelve a subir.** Si hay filas con error, corrígelas en tu Excel y súbelo otra vez; una vista previa nueva reemplaza a la anterior. No puedes corregir las filas dentro de la pantalla.
7. **Guarda.** Haz clic en **Guardar válidos (N)**, bajo la tabla. Solo se guardan las filas válidas. El botón está deshabilitado si no hay ninguna fila válida.
8. **Resultado.** Aparece el aviso «Se guardaron N registros correctamente» y el sistema te lleva a la búsqueda de sucursales.

### 8.3 Errores por fila en la vista previa

Para cada fila con error, la columna de errores lista un mensaje por cada problema. Comprobado en la aplicación con un archivo de prueba de cuatro filas:

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «El campo 'nombre' es obligatorio» | La fila tiene la celda del nombre vacía (lo mismo vale para la dirección y la licencia: 'direccionFisica' y 'numeroLicencia') | Completa la celda |
| «El valor 'LIC-DEMO-0001' del campo 'numeroLicencia' ya existe en el sistema» | Ya tienes una sucursal con esa licencia | Cambia la licencia o quita la fila |
| «El valor 'LIC-CARGA-0001' del campo 'numeroLicencia' está duplicado en el archivo» | La misma licencia aparece en más de una fila del archivo. **Se marcan todas las filas repetidas, incluida la primera** | Deja una sola fila con esa licencia o cámbiales la licencia |
| «'…' no puede superar los N caracteres» (con el campo y el límite) | El texto pasa de 100, 255 o 50 caracteres | Acórtalo |

Los nombres de campo salen tal como los usa el sistema por dentro (`nombre`, `direccionFisica`, `numeroLicencia`), no con la etiqueta de la pantalla. (El mensaje «'…' no puede superar los N caracteres» no se provocó; viene del inventario 19a §8.9.)

### 8.4 Errores del archivo

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «El archivo no es un Excel .xlsx válido. Descarga la plantilla y vuelve a intentarlo.» | El archivo no termina en `.xlsx` o está dañado | Usa un `.xlsx` creado desde la plantilla |
| «El archivo supera el tamaño máximo de 5 MB.» | Pesa más de 5 MB | Divide el archivo |
| «El archivo supera el máximo de 5000 filas. Divídelo en varios archivos.» | Tiene más de 5000 filas con datos | Divídelo |
| «El archivo excede los límites de contenido permitidos. Revisa que sea una plantilla válida.» | Archivo sospechoso (demasiado comprimido) | Vuelve a crearlo desde la plantilla |
| «Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa.» | El servidor rechazó filas al guardar | Sube el archivo de nuevo y mira la vista previa |
| «Error al procesar el archivo» / «Error al guardar los registros» | Falla genérica en la vista previa / al guardar | Inténtalo de nuevo |
| «Error al descargar la plantilla. Intenta de nuevo.» | No se pudo bajar la plantilla | Inténtalo de nuevo |
| «No hay filas válidas para guardar» | Se intentó guardar sin filas válidas (el botón lo impide) | — |

### 8.5 Qué se comprobó y qué no

- **Plantilla:** al hacer clic en **Descargar plantilla** el servidor respondió y la pantalla no mostró errores. (No verificado en la aplicación: el contenido del archivo descargado, porque el navegador integrado no muestra descargas; la descripción de la hoja viene del documento 20 §2.1 y §3.11.)
- **Subir un archivo:** el navegador integrado no tiene herramienta para elegir un archivo del disco. Para poder comprobar el resto del flujo se entregó a la pantalla, desde el propio navegador, un archivo `.xlsx` de prueba construido para este manual (y un archivo de texto para probar el rechazo). Con eso se comprobaron: el rechazo del archivo que no es `.xlsx` («El archivo no es un Excel .xlsx válido…»), la vista previa con sus contadores, los errores «obligatorio», «ya existe en el sistema» y «duplicado en el archivo», que una vista previa nueva reemplaza a la anterior y el botón **Guardar válidos (1)**, que mostró «Se guardaron 1 registros correctamente» y llevó a la búsqueda de sucursales. La sucursal creada así («Sucursal Carga Uno») se eliminó después desde la pantalla; no quedó en la empresa de prueba. Como no se usó el selector de archivos del sistema operativo, el gesto de «arrastrar o hacer clic» **no verificado en la aplicación** en sí mismo.
- Los demás errores de archivo de la tabla 8.4 (tamaño, más de 5000 filas, descompresión, errores del servidor) vienen del inventario 19a §6.5 y §8.9 y del documento 20 §2.6: **No verificado en la aplicación**.
- Las filas que SOFIA omite sin avisar al guardar: si entre la vista previa y el guardado otra persona crea una sucursal con la misma licencia, esa fila se omite en silencio y el aviso dice cuántas se guardaron realmente (documento 20 §3.11; no verificado en la aplicación).

## 9. Series fiscales: qué son y para qué las usa SOFIA

### 9.1 Qué es una serie

Una **serie fiscal** es la numeración de un tipo de comprobante en una sucursal. Es como un talonario: tiene un **prefijo** de 4 caracteres (por ejemplo `B001`) y un contador, el **Correlativo Actual**, que guarda el **último número emitido**. Cada comprobante nuevo usa el número siguiente: con el correlativo en `0`, el primer comprobante lleva el número 1 y se imprime con 8 dígitos, por ejemplo `B001-00000001` (regla RN-CMP-004; el comprobante impreso se ve en el capítulo de ventas).

Cada serie pertenece a **una sucursal** y a **un tipo de comprobante**. Los tipos que puedes elegir en pantalla son cuatro:

| Tipo | Con qué letra debe empezar el prefijo | Para qué lo usa SOFIA hoy |
|---|---|---|
| Boleta | `B` | Cada venta cobrada emite una boleta |
| Factura | `F` | Ninguna pantalla emite facturas todavía; puedes crear la serie, pero hoy no se usa |
| Nota de Crédito | `B` o `F` (la letra del comprobante que corrige) | Cada devolución emite una nota de crédito |
| Proforma | Cualquiera | Ninguna pantalla la usa todavía |

Dentro del sistema existen también los tipos «Ticket» y «Nota de Débito», pero no se pueden crear ni filtrar desde la pantalla.

### 9.2 Lo que debes tener claro

- **El punto de venta solo emite boletas y notas de crédito.** Aunque el cliente tenga RUC, la venta genera una **boleta**, nunca una factura. Por eso cada sucursal que venda necesita una serie de **Boleta activa**, y cada sucursal que acepte devoluciones necesita una serie de **Nota de Crédito activa**.
- **Los comprobantes no se envían a SUNAT.** SOFIA solo los registra en tu sistema; salen con el estado «Pendiente de envío a SUNAT». No existe todavía ninguna pantalla ni proceso de envío (por eso el grupo **SUNAT** del menú solo tiene «Series Fiscales»).
- **Una sola serie activa por sucursal y tipo.** Puedes tener todas las series inactivas que quieras, pero solo una «Activa» de cada tipo en cada sucursal.
- **El prefijo no se repite entre sucursales.** En tu empresa no puede haber dos series del mismo tipo con el mismo prefijo, aunque sean de sucursales distintas. Por eso la sucursal 1 usa `B001` y la 2 usa `B002`.
- **Sin serie activa no hay venta ni devolución.** Si la sucursal no tiene serie de Boleta activa, el cobro se rechaza con el aviso «Esta sucursal no tiene una serie de boletas activa. Configúrala en SUNAT > Series Fiscales para poder cobrar.» Si no tiene serie de Nota de Crédito activa, la devolución se rechaza con «La sucursal no tiene una serie de nota de crédito activa. Configúrala en Series Fiscales.» (No verificado en la aplicación en este capítulo: estos avisos aparecen en el punto de venta y vienen de la regla RN-CMP-003 y del texto `pos.devolucion.errors.serieNcNoConfigurada`; se verán en los capítulos de ventas y devoluciones.)

### 9.3 Qué series crear

Para dejar lista la empresa de prueba se crean estas cuatro series (todas «Activa», con correlativo inicial `0`):

| Sucursal | Tipo de Comprobante | Prefijo de Serie |
|---|---|---|
| Sede Principal | Boleta | `B001` |
| Sede Principal | Nota de Crédito | `BC01` |
| Sucursal Demo Norte | Boleta | `B002` |
| Sucursal Demo Norte | Nota de Crédito | `BC02` |

Los prefijos de las notas de crédito empiezan con `B` porque corrigen boletas. Dos series de tipos distintos pueden compartir el prefijo, pero aquí se usan prefijos distintos para no confundirlas.

Si tu botica ya usaba una numeración antes de SOFIA, escribe en «Correlativo Actual» el último número que emitiste (por ejemplo `250` si tu última boleta fue la 250) para que SOFIA continúe desde ahí.

## 10. Crear una serie fiscal

**Cómo llegar:** menú lateral › SUNAT › **Series Fiscales** (`/series-fiscales`) › botón **Nuevo**. Necesitas el permiso `Crear` del módulo «Series Fiscales» (y `Leer`). Si no ves **Nuevo**, te falta ese permiso.

### 10.1 Cómo es la pantalla

Arriba, la flecha de volver, el título «Series Fiscales» y las migas «Inicio / SUNAT / Series Fiscales». El formulario está ya en modo editar y tiene, en este orden, cinco campos y los botones **Cancelar** y **Guardar** (no hay **Eliminar** al crear).

| Campo | Qué escribir o elegir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Sucursal | Elige la sucursal en la lista. Aparecen todas las sucursales de tu empresa | Sí | Vacío: «Este campo es obligatorio» |
| Tipo de Comprobante | Elige «Boleta», «Factura», «Nota de Crédito» o «Proforma». Empieza en «Boleta» | Sí | Siempre tiene un valor |
| Prefijo de Serie | 4 caracteres: letras mayúsculas y números. Debajo se lee la ayuda «4 caracteres en mayúsculas. Boleta empieza con B, Factura con F, Nota de Crédito con B o F.». Lo que escribas en minúsculas se convierte solo a mayúsculas, y el campo no deja escribir más de 4 caracteres | Sí | Vacío: «Este campo es obligatorio». Menos de 4 caracteres, con un símbolo (por ejemplo `B-01`) o con una letra que no corresponde al tipo (por ejemplo `F001` en una Boleta): «El prefijo debe tener 4 caracteres en mayúsculas y empezar con la letra que exige el tipo de comprobante.» |
| Correlativo Actual | Un número entero. Empieza en `0`. Debajo se lee «Último número emitido; el siguiente comprobante usará este valor + 1.» | Sí | Vacío: «Este campo es obligatorio». Negativo (`-3`) o con decimales (`1.5`): «El correlativo debe ser un número entero mayor o igual a 0.» |
| Estado | «Activa» o «Inactiva». Empieza en «Activa» | Sí | Siempre tiene un valor |

Todos los mensajes de campo aparecen debajo del campo, al salir de él o al hacer clic en **Guardar** con el formulario incompleto (en ese caso no se envía nada). El mensaje del prefijo se vuelve a evaluar al cambiar el tipo: un prefijo `BC01` es válido para «Nota de Crédito» y «Boleta», pero no para «Factura».

### 10.2 Probar los errores antes de guardar

Comprobado en la aplicación, con la sucursal «Sede Principal»:

1. Haz clic en **Guardar** con el formulario tal como se abre: aparece «Este campo es obligatorio» bajo «Sucursal» y bajo «Prefijo de Serie».
2. Elige la sucursal y escribe `b00` en «Prefijo de Serie»: el campo lo muestra como `B00` y, al salir, aparece el mensaje «El prefijo debe tener 4 caracteres en mayúsculas y empezar con la letra que exige el tipo de comprobante.»
3. Escribe `f001` con el tipo «Boleta»: se ve `F001` y aparece el mismo mensaje.
4. Escribe `B-01`: el mismo mensaje.
5. Escribe `-3` o `1.5` en «Correlativo Actual»: «El correlativo debe ser un número entero mayor o igual a 0.» Deja el campo vacío: «Este campo es obligatorio».

### 10.3 Guardar

1. Elige la sucursal «Sede Principal» y el tipo «Boleta».
2. Escribe el prefijo `B001`.
3. Deja el correlativo en `0` y el estado en «Activa».
4. Haz clic en **Guardar**.

**Qué pasa al terminar:** aparece el aviso «Serie fiscal guardada correctamente» y el sistema te lleva a la búsqueda de series (`/series-fiscales`). Repite los pasos para las otras tres series de la sección 9.3 (**Nuevo** otra vez).

### 10.4 Errores del servidor al guardar

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Ya existe otra serie de este tipo de comprobante con ese prefijo.» | Probado: se intentó crear una Boleta `B001` para «Sucursal Demo Norte» cuando `B001` ya existía en la «Sede Principal». El prefijo no se puede repetir en la empresa para el mismo tipo | Elige otro prefijo (en el ejemplo, `B002`) |
| «La sucursal ya tiene una serie activa de este tipo. Desactívala antes de activar otra.» | Probado: se intentó crear una segunda Boleta «Activa» (`B009`) para la «Sede Principal». Solo se permite una serie activa por sucursal y tipo | Desactiva la anterior (sección 11.3) o guarda la nueva como «Inactiva» |
| «Error al guardar la serie fiscal» | Cualquier otro problema | Revisa los datos e inténtalo de nuevo |
| «La serie fiscal no existe.» / «La sucursal seleccionada no existe.» | Alguien eliminó la serie o la sucursal mientras trabajabas | Vuelve a la búsqueda y revisa |
| «No se pudieron cargar las sucursales» | No se pudo llenar la lista de sucursales | Recarga la pantalla |

(No verificado en la aplicación: «Error al guardar la serie fiscal», «La serie fiscal no existe.», «La sucursal seleccionada no existe.» y «No se pudieron cargar las sucursales» no se pudieron provocar; vienen del inventario 19b §6.1 y §6.2.)

## 11. Buscar, editar, desactivar y eliminar una serie

### 11.1 Buscar series y exportar

**Cómo llegar:** menú lateral › SUNAT › **Series Fiscales**. Tarjeta «Búsqueda de Series Fiscales» con el botón **Nuevo** arriba a la derecha y estos filtros:

| Filtro | Qué hace |
|---|---|
| Sucursal | Lista con «Todas» y las sucursales de tu empresa |
| Prefijo de Serie | Texto con lupa, máximo 4 caracteres. Busca prefijos que **contengan** lo que escribas, sin importar mayúsculas (`c0` encuentra `BC01` y `BC02`). Con **Enter** busca |
| Tipo de Comprobante | «Todos», «Boleta», «Factura», «Nota de Crédito» o «Proforma» |
| Estado | «Todos», «Activa» o «Inactiva» |

Los filtros se combinan. **Limpiar** los vacía y oculta la tabla. Los resultados aparecen al hacer clic en **Buscar**. La tabla, ordenada por prefijo, tiene estas columnas:

| Columna | Qué muestra |
|---|---|
| Serie | El prefijo; es un enlace al detalle |
| Tipo de Comprobante | «Boleta», «Factura», «Nota de Crédito» o «Proforma» |
| Sucursal | El nombre de la sucursal |
| Correlativo Actual | El último número emitido |
| Estado | «Activa» o «Inactiva» |

Con las series de la sección 9.3 la búsqueda sin filtros muestra:

| Serie | Tipo de Comprobante | Sucursal | Correlativo Actual | Estado |
|---|---|---|---|---|
| B001 | Boleta | Sede Principal | 0 | Activa |
| B002 | Boleta | Sucursal Demo Norte | 0 | Activa |
| BC01 | Nota de Crédito | Sede Principal | 0 | Activa |
| BC02 | Nota de Crédito | Sucursal Demo Norte | 0 | Activa |

Comprobado: el filtro «Tipo de Comprobante» = «Nota de Crédito» junto con «Sucursal» = «Sucursal Demo Norte» trae solo `BC02`. Sin series, la búsqueda muestra «Resultados (0)» y «Sin resultados». Bajo la tabla están **Exportar** y el paginador (10, 25 o 50 por página). Cuando vuelves a la búsqueda después de abrir o guardar una serie, la pantalla recuerda tus filtros y repite la búsqueda sola (mientras no recargues el navegador).

**Exportar** genera `series-fiscales.xlsx` con las columnas «Serie», «Tipo de Comprobante», «Sucursal», «Correlativo Actual», «Estado» y «Fecha de Creación» (con formato `dd/MM/yyyy` en español; la fecha va en hora UTC, sin convertir a hora de Lima). Comprobado: al hacer clic en **Exportar** el servidor respondió correctamente y no hubo aviso de error. (No verificado en la aplicación: el contenido del archivo; viene del documento 20 §4.17.) Si falla: «Error al exportar las series fiscales»; si falla la búsqueda: «Error al buscar series fiscales».

### 11.2 Ver y editar una serie

1. En la búsqueda, haz clic en el prefijo de la serie. Se abre el detalle, con el título «Series Fiscales», el texto «Datos del registro» y el botón **Editar** (con el permiso `Actualizar`). Los cinco campos aparecen deshabilitados.
2. Haz clic en **Editar**. Abajo a la derecha aparecen **Cancelar**, **Guardar** y **Eliminar** (este con el permiso `Eliminar`). Los cinco campos y sus reglas son los de la sección 10.1.
3. Cambia lo que necesites y haz clic en **Guardar**.

Comprobado con una serie de prueba (una Proforma `PF01` de la «Sede Principal»): se cambió el prefijo a `PF02` y el correlativo a `5`; al guardar apareció «Serie fiscal guardada correctamente», el sistema volvió a la búsqueda (a diferencia de Sucursales, que se queda en el detalle) y la tabla mostró `PF02`, `Proforma`, `Sede Principal`, `5`, `Activa`.

**Cuando la serie ya emitió comprobantes** (No verificado en la aplicación: la empresa de prueba todavía no emitió comprobantes; viene del inventario 19b §6.2 y de la regla RN-SER-010). Si alguna venta o devolución ya usó la serie, al pulsar **Editar** aparece sobre el formulario el aviso «Esta serie ya emitió comprobantes: solo puedes cambiar su estado.» y solo queda habilitado el campo **Estado**. «Sucursal», «Tipo de Comprobante», «Prefijo de Serie» y «Correlativo Actual» se bloquean porque cambiarlos alteraría la numeración que ya emitiste. Si de todos modos el servidor recibiera un cambio de esos datos, rechazaría el guardado con «La serie ya emitió comprobantes: solo puedes cambiar su estado.»

### 11.3 Desactivar una serie

Desactivar una serie la deja de usar sin borrarla: es lo que debes hacer cuando una serie ya emitió comprobantes y no quieres seguir usándola, o antes de activar otra del mismo tipo en la sucursal.

1. Abre la serie y haz clic en **Editar**.
2. En «Estado», elige «Inactiva».
3. Haz clic en **Guardar**.

Comprobado con `PF02`: apareció «Serie fiscal guardada correctamente» y la tabla mostró el estado «Inactiva». Mientras no haya otra serie activa del mismo tipo en la sucursal, el punto de venta no podrá emitir ese tipo de comprobante en ella (en el caso de la Boleta, no se podrá cobrar; ver la sección 9.2). Para volver a usarla, repite los pasos eligiendo «Activa».

### 11.4 Eliminar una serie

1. Abre la serie, haz clic en **Editar** y luego en **Eliminar** (abajo a la derecha, después de **Guardar**).
2. Se abre un cuadro con el título «Confirmar eliminación», el mensaje «¿Estás seguro de que deseas eliminar esta serie fiscal?» y los botones **Cancelar** y **Confirmar**.
3. Haz clic en **Confirmar**.

**Qué pasa al terminar:** aparece el aviso «Serie fiscal eliminada correctamente» y vuelves a la búsqueda, donde la serie ya no aparece (comprobado con `PF02`: «Resultados (0)»).

**No puedes eliminar una serie que ya emitió comprobantes.** El servidor lo rechaza y verías el aviso «No se puede eliminar una serie que ya emitió comprobantes. Puedes desactivarla.» (No verificado en la aplicación: no hay una serie con comprobantes; viene de 19b §6.2.) En ese caso, desactívala (sección 11.3). Cualquier otro fallo al eliminar muestra «Error al eliminar la serie fiscal»; si no se puede abrir la serie, «Error al cargar la serie fiscal» (ambos no verificados en la aplicación).

Antes de eliminar una sucursal (sección 7), recuerda que las series fiscales que tenga cuentan como registros asociados y la bloquean: elimina primero las series que no hayan emitido comprobantes.

## 12. Lo que ve un cajero frente a un administrador

| Pantalla o elemento | Administrador (rol «Admin») | Cajero (u otro rol con permisos limitados) |
|---|---|---|
| Menú «Empresa», «Sucursales» y «Series Fiscales» | Los ve | Solo si su rol tiene el permiso `Leer` del módulo «Empresa», «Sucursales» o «Series Fiscales». El grupo **SUNAT** desaparece si no tiene `Leer` en «Series Fiscales» |
| Escribir la dirección de una pantalla sin permiso | — | El sistema lo devuelve al Inicio con «No tienes permisos para acceder a esta pantalla.» |
| Botón **Editar** del detalle | Lo ve | Solo con el permiso `Actualizar` del módulo |
| Botones **Nuevo** y **Carga Masiva** (sucursales), **Nuevo** (series) | Los ve | Solo con el permiso `Crear` del módulo. La carga masiva exige además `Leer` |
| Botón **Eliminar** (sucursal y serie) | Lo ve en el modo editar | Solo con el permiso `Eliminar` del módulo |
| Botón **Exportar** | Lo ve | Lo ve cualquiera que pueda abrir la búsqueda (la pantalla no lo condiciona a un permiso propio; el servidor exige `Leer`) |
| Buscar gerente al editar una sucursal | Funciona | Necesita además el permiso `Leer` del módulo «Empleados» |

Un cajero normalmente no tiene permisos sobre estas pantallas: la configuración de la empresa la hace el administrador. (No verificado en la aplicación en este capítulo: todavía no existe ningún cajero en la empresa de prueba; la columna del cajero viene del inventario 19b §0 y de los permisos de la sección 2; se verá en vivo en el capítulo 3.)

## 13. Diferencias con la documentación técnica

El inventario `19b-administracion.md` (versión 1.1) coincidió con lo que hace la aplicación en todo lo comprobado. Solo se anotan estos matices:

| # | Documento y apartado | Dice | La aplicación muestra |
|---|---|---|---|
| 1 | 19b §2.2 (guardar una sucursal nueva) | Tras crear, navega a `/sucursales` | Coincide. Se anota además que la búsqueda queda con los filtros vacíos y sin tabla (hay que hacer clic en **Buscar**) cuando antes no habías buscado en esa sesión |
| 2 | 20 §3.11 y 19b §2.3 (vista previa: `duplicate-in-file`) | La licencia «repetida en el archivo» se marca con `duplicate-in-file` | Se marcan **todas** las filas que repiten la licencia, también la primera (probado con dos filas con `LIC-CARGA-0001`: las dos salieron con error y «0 filas válidas»). El documento no precisa si la primera ocurrencia queda como válida |
| 3 | 19b §6.2 (Editar) | Éxito: aviso y navega a `/series-fiscales` | Coincide. Contrasta con Sucursales y Empresa, que tras editar se quedan en el detalle en modo ver |
| 4 | 19b §1.1 (fila vacía) | La fila vacía de la tabla de Empresa dice «Resultados» (no «Sin resultados») | Coincide: la tabla de Empresa dice «Resultados» y el paginador «Sin resultados»; en Sucursales y Series la fila vacía dice «Sin resultados» |
| 5 | 19b §1.2 (rótulo del RUC) | El campo se rotula «RUC (11 dígitos, opcional)» pero no se valida en pantalla | Coincide, y se comprobó el efecto: el servidor rechaza un RUC de 10 dígitos, con letras, repetido o vacío con el mismo aviso genérico «Error al actualizar la empresa»; por eso, aunque el rótulo diga «opcional», desde la pantalla no se puede dejar la empresa sin RUC |

## 14. Pendientes y «No verificado»

En curso (sin commitear): ninguno. Los dos repositorios estaban sin cambios locales al empezar (`sofia-apis` `43f5529`, `sofia-web-spa` `eeffbb6`).

No verificado en la aplicación (cada caso, con su motivo):

| # | Qué | Motivo |
|---|---|---|
| 1 | Contenido y descarga de los archivos de **Exportar** (empresas, sucursales y series) y de la plantilla de carga masiva | El navegador integrado no muestra los archivos descargados. Se comprobó que la petición se envía y que la pantalla no muestra errores (en Empresa y Series el servidor respondió 200). Las columnas vienen del documento 20 §2.1, §3.11, §4.4, §4.17 y §4.20 |
| 2 | El gesto de arrastrar o hacer clic para elegir el archivo en la carga masiva | El navegador integrado no tiene herramienta para elegir un archivo. El flujo se comprobó entregando a la pantalla, desde el propio navegador, un archivo `.xlsx` de prueba y uno de texto; la sucursal creada se eliminó después |
| 3 | Errores de archivo de la carga masiva: tamaño de más de 5 MB, más de 5000 filas, descompresión excesiva, errores del servidor al procesar o al guardar, `'…' no puede superar los N caracteres` | No se prepararon archivos de ese tipo; vienen de 19a §6.5 y §8.9 y del documento 20 §2.6 |
| 4 | Filas que el guardado de la carga masiva omite sin avisar (licencia creada por otra persona entre la vista previa y el guardado) | Requiere dos usuarios a la vez; viene del documento 20 §3.11 |
| 5 | Avisos «Error al buscar empresas», «Error al buscar sucursales», «Error al exportar las series fiscales», «Error al buscar series fiscales», «Error al cargar la empresa», «Error al cargar la sucursal», «Error al cargar la serie fiscal», «Error al guardar la sucursal», «Error al guardar la serie fiscal», «Error al eliminar la sucursal», «Error al eliminar la serie fiscal», «La serie fiscal no existe.», «La sucursal seleccionada no existe.», «No se pudieron cargar las sucursales» | Son errores genéricos del servidor o de red; no se pueden provocar desde la pantalla. Vienen de 19b §1, §2 y §6 |
| 6 | Paginación real (más de una página), el botón de volver a la página 1 y que la búsqueda de Sucursales y Series conserva la página al cambiar de filtro | Con 2 sucursales y 4 series solo hay una página. Viene de 19b §0, §2.1 y §6.1. Sí se comprobó que el selector pasa de 10 a 25 y que, al volver de un detalle, la búsqueda de series recuerda los filtros |
| 7 | Series que ya emitieron comprobantes: aviso «Esta serie ya emitió comprobantes: solo puedes cambiar su estado.», campos bloqueados, «La serie ya emitió comprobantes: solo puedes cambiar su estado.» y «No se puede eliminar una serie que ya emitió comprobantes. Puedes desactivarla.» | La empresa de prueba todavía no emitió comprobantes (se verán después de los capítulos de ventas y devoluciones). Viene de 19b §6.2 y RN-SER-010 y RN-SER-011 |
| 8 | Avisos del punto de venta cuando falta el RUC o la serie («Tu empresa no tiene RUC registrado…», «Esta sucursal no tiene una serie de boletas activa…», «La sucursal no tiene una serie de nota de crédito activa…») | Se ven al cobrar o devolver (capítulos siguientes). Vienen de RN-CMP-002, RN-CMP-003 y los textos `pos.messages.empresaSinRuc`, `pos.messages.serieBoletaNoConfigurada` y `pos.devolucion.errors.serieNcNoConfigurada` |
| 9 | La vista de un cajero (menú reducido, ausencia de botones, redirección «No tienes permisos para acceder a esta pantalla.») y qué ve quien no tiene `Empleados:Leer` al buscar gerente | El administrador tiene todos los permisos y aún no existe un cajero; se verá en el capítulo 3 |
| 10 | Asignar un gerente a una sucursal (elegir un empleado y guardar) | Se comprobó la búsqueda de empleados y el vaciado del campo, pero no se guardó un gerente para no dejar esa relación en las sucursales del manual. Viene de 19b §2.2 |
| 11 | Que el servidor acepte quitar el RUC (enviarlo nulo) | La pantalla no puede enviar un RUC nulo: un RUC vacío se rechazó. Viene de 19b §1.2 y RN-EMP-004 |
| 12 | Cuántos días de prueba quedan y qué ocurre al vencer la suscripción | Es de SOFIA y no hay pantalla para cambiarlo (RN-EMP-015); el estado de la empresa de prueba no se modificó |

Hallazgos de la aplicación que conviene conocer (no son errores del manual):

- El rótulo «RUC (11 dígitos, opcional)» de la pantalla de Empresa no coincide con el comportamiento: el RUC no se puede dejar vacío (el servidor lo rechaza) y el servidor exige 11 dígitos, pero la pantalla no lo valida y solo muestra «Error al actualizar la empresa».
- El campo «Nombre de la Empresa» del detalle se corta a los 150 caracteres, mientras que el registro del capítulo 1 admite 200.
- La carga masiva de sucursales marca como duplicadas **todas** las filas que repiten una licencia, no solo las repetidas a partir de la segunda.

## 15. Tabla de cobertura

Cada elemento de las fichas de `19b-administracion.md` de Empresa (§1), Sucursales (§2) y Series fiscales (§6), con el apartado del capítulo donde quedó.

### 15.1 Empresa — búsqueda `/empresas` (19b §1.1)

| Elemento | Apartado |
|---|---|
| Título «Empresas» y migas «Inicio / Administración / Empresa» (sin botón volver) | 3.1 |
| Tarjeta «Búsqueda de Empresas» y subtítulo «Ingresa los criterios de búsqueda» | 3.1 |
| Filtro «Nombre de la Empresa» (lupa, Enter, «contiene») | 3.1 (verificado) |
| Filtro «Estado» (Todos, Trial Activo, Activo, Suspendido, Cancelado; empieza en blanco) | 3.1 (verificado) |
| Filtros «Vencimiento desde» y «Vencimiento hasta» (calendario, campos de solo lectura) | 3.1 (verificado) |
| Botones **Limpiar** y **Buscar** | 3.1 (verificado) |
| Los resultados no aparecen hasta buscar | 3.1 (verificado) |
| Tabla: Empresa (enlace), RUC, Estado, Vigente (icono) | 3.1 (verificado) |
| «Resultados (N)», fila vacía «Resultados» y «Sin resultados» del paginador | 3.1 (verificado) |
| Paginador: «Items por página» 10/25/50, «Página 1 de 1», primera/anterior/siguiente/última | 3.1 (verificado; más de una página: No verificado, solo hay una empresa) |
| Botón **Exportar**, archivo `empresas.xlsx` y sus 7 columnas | 3.1 (el contenido del archivo: No verificado en la aplicación) |
| No hay **Nuevo** ni **Carga Masiva** | 3.2 |
| Mensaje «Error al buscar empresas» (búsqueda y exportación) | 3.1 (No verificado en la aplicación: no se puede provocar) |
| Permiso `Empresas:Leer` | 2 |

### 15.2 Empresa — detalle `/empresas/:id` (19b §1.2)

| Elemento | Apartado |
|---|---|
| Título, migas, flecha de volver y tarjeta «Empresas» con «Datos del registro» | 3.2 |
| Campos de solo lectura: Estado, Sucursales, Inicio Trial, Vencimiento | 3.2 (verificado) |
| Campo «Nombre de la Empresa» (obligatorio, máximo 150) | 3.3 (verificado) |
| Campo «RUC (11 dígitos, opcional)» (máximo 20, sin validación de formato en pantalla) | 3.3 (verificado) |
| Botón **Editar** (permiso `Empresas:Actualizar`) | 3.2, 3.3 |
| Botones **Cancelar** y **Guardar** | 3.3 (verificado) |
| «Empresa actualizada correctamente» | 3.3 (verificado) |
| «Error al actualizar la empresa» (RUC de 10 dígitos, con letra, repetido, vacío) | 3.3 (verificado) |
| «Este campo es obligatorio» | 3.3 (verificado) |
| «Error al cargar la empresa» | 3.3 (No verificado en la aplicación: no se puede provocar) |
| Estados de suscripción que se muestran (Trial Activo, Activo, Suspendido, Cancelado) | 3.1, 3.2 |
| No existe **Eliminar**, ni crear empresa, ni opciones de configuración | 3.2 |
| El indicador «Vigente» solo está en la tabla de búsqueda | 3.1, 3.2 |

### 15.3 Sucursales — búsqueda `/sucursales` (19b §2.1)

| Elemento | Apartado |
|---|---|
| Título «Sucursales», migas, tarjeta «Búsqueda de Sucursales» y subtítulo | 4.1 |
| Botones **Carga Masiva** y **Nuevo** (permiso `Sucursales:Crear`) | 4.1, 5, 8 |
| Filtros «Nombre», «Nº de Licencia» y «Dirección Física» (lupa, Enter) | 6.1 (verificado) |
| **Limpiar** y **Buscar** | 6.1 (verificado) |
| Tabla: Nombre (enlace), Nº de Licencia, Dirección Física, Gerente («—») | 4.2, 6.2 (verificado) |
| «Resultados (N)», «Sin resultados» y paginador (10/25/50) | 6.1, 6.2 (verificado; páginas múltiples: No verificado) |
| Botón **Exportar** y archivo `sucursales.xlsx` con 4 columnas | 6.3 (el contenido del archivo: No verificado en la aplicación) |
| «Error al buscar sucursales» | 6.3 (No verificado en la aplicación) |
| La búsqueda conserva la página; se restauran filtros al volver | 6.2 (No verificado en la aplicación en Sucursales; sí en Series, 11.1) |
| Permiso `Sucursales:Leer` | 2 |

### 15.4 Sucursales — detalle `/sucursales/nueva` y `/sucursales/:id` (19b §2.2)

| Elemento | Apartado |
|---|---|
| Cabecera (título, migas, flecha de volver) y modos crear, ver y editar | 4.2, 4.3, 5 |
| Botón **Editar** (permiso `Sucursales:Actualizar`) | 4.2, 4.3 |
| Campo «Nombre» (obligatorio, máximo 100) | 4.3, 5.1 (verificado) |
| Campo «Nº de Licencia» (obligatorio, máximo 50, único) | 4.3, 5.1 (verificado) |
| Campo «Dirección Física» (obligatorio, máximo 255) | 4.3 (máximo 255: No verificado, se verificaron 100 y 50) |
| Campo «Gerente» (buscador de empleados, mínimo 2 letras, se vacía si no coincide) | 4.3 (verificado la búsqueda y el vaciado; asignar un gerente: No verificado) |
| Botones **Cancelar** y **Guardar** | 4.3, 5 (verificado) |
| Botón **Eliminar** (permiso `Sucursales:Eliminar`, solo en modo editar de una sucursal existente) | 7.2 (verificado) |
| Cuadro «Confirmar eliminación» («¿Estás seguro de que deseas eliminar esta sucursal?», **Cancelar**, **Confirmar**) | 7.2 (verificado) |
| «Sucursal guardada correctamente» | 4.3, 5.2 (verificado) |
| «Ya existe una sucursal con ese número de licencia.» | 4.3, 5.1 (verificado) |
| Códigos `Db.*` («Ya existe un registro con ese mismo valor…», etc.) | No aplica: no se pudieron provocar; caen en «Error al guardar la sucursal» (4.3, No verificado en la aplicación) |
| «Error al guardar la sucursal» | 4.3 (No verificado en la aplicación) |
| «Sucursal eliminada correctamente» | 7.2 (verificado) |
| «No puedes eliminar esta sucursal porque tiene empleados, usuarios, roles, stock, ventas u otros registros asociados.» | 7.3 (verificado) |
| «Error al eliminar la sucursal» | 7.3 (No verificado en la aplicación) |
| «Error al cargar la sucursal» | 4.3 (No verificado en la aplicación) |
| La sede principal nace con la licencia vacía y se completa al editar | 4 (verificado) |

### 15.5 Sucursales — carga masiva `/sucursales/carga-masiva` (19b §2.3, 19a §6.5-6.6, doc 20 §3.11)

| Elemento | Apartado |
|---|---|
| Título «Carga Masiva», migas y flecha de volver | 8.1 |
| Botón **Descargar plantilla**, archivo `plantilla-sucursales.xlsx` | 8.2 (el contenido: No verificado en la aplicación) |
| Zona «Arrastra tu archivo Excel aquí o haz clic para seleccionar» y «Solo archivos .xlsx (máx. 5 MB y 5000 filas)» | 8.1 (verificado) |
| Columnas de la plantilla `Nombre`, `DireccionFisica`, `NumeroLicencia` (todas obligatorias) y sin gerente | 8.2 |
| «Procesando archivo...» | 8.2 (No verificado en la aplicación: fue demasiado breve para verla) |
| «Vista previa», «N filas totales», «N filas válidas», «N filas con error» y columnas «Nombre», «Dirección Física», «Nº de Licencia», «Estado» | 8.2, 8.5 (verificado con un archivo de prueba) |
| Errores por fila `required`, `duplicate`, `duplicate-in-file` | 8.3 (verificado) |
| Error por fila `max-length` | 8.3 (No verificado en la aplicación) |
| **Guardar válidos (N)** y «Se guardaron N registros correctamente» | 8.2, 8.5 (verificado con un archivo de prueba) |
| «El archivo no es un Excel .xlsx válido. Descarga la plantilla y vuelve a intentarlo.» | 8.4 (verificado con un archivo de texto) |
| Demás errores de archivo y de guardado (5 MB, 5000 filas, descompresión, filas inválidas, genéricos, plantilla, «No hay filas válidas para guardar») | 8.4 (No verificado en la aplicación) |
| Permiso `Sucursales:Crear` | 2, 8 |

### 15.6 Series fiscales — búsqueda `/series-fiscales` (19b §6.1)

| Elemento | Apartado |
|---|---|
| Título «Series Fiscales», migas «Inicio / SUNAT / Series Fiscales», tarjeta «Búsqueda de Series Fiscales» | 11.1 |
| Botón **Nuevo** (permiso `SeriesFiscales:Crear`) | 10, 11.1 |
| Filtro «Sucursal» («Todas» y las sucursales) | 11.1 (verificado) |
| Filtro «Prefijo de Serie» (máximo 4, mayúsculas, «contiene», Enter) | 11.1 (verificado) |
| Filtro «Tipo de Comprobante» (Todos, Boleta, Factura, Nota de Crédito, Proforma) | 11.1 (verificado) |
| Filtro «Estado» (Todos, Activa, Inactiva) | 11.1 (verificado) |
| **Limpiar** y **Buscar** | 11.1 (verificado) |
| Tabla: Serie (enlace), Tipo de Comprobante, Sucursal, Correlativo Actual, Estado | 11.1 (verificado) |
| «Resultados (N)», «Sin resultados», paginador | 11.1 (verificado; páginas múltiples: No verificado) |
| Botón **Exportar**, `series-fiscales.xlsx` y 6 columnas | 11.1 (el contenido: No verificado en la aplicación) |
| «Error al buscar series fiscales», «Error al exportar las series fiscales», «No se pudieron cargar las sucursales» | 11.1, 10.4 (No verificado en la aplicación) |
| Tipos y prefijos permitidos; tipos Ticket y Nota de Débito no configurables | 9.1 |
| Permiso `SeriesFiscales:Leer` | 2 |

### 15.7 Series fiscales — detalle `/series-fiscales/nueva` y `/series-fiscales/:id` (19b §6.2)

| Elemento | Apartado |
|---|---|
| Cabecera y modos crear, ver y editar | 10.1, 11.2 |
| Campo «Sucursal» (obligatorio) | 10.1, 10.2 (verificado) |
| Campo «Tipo de Comprobante» (empieza en Boleta) | 10.1 (verificado) |
| Campo «Prefijo de Serie» (4 caracteres, mayúsculas automáticas, ayuda, error de formato y de tipo) | 10.1, 10.2 (verificado) |
| Campo «Correlativo Actual» (entero ≥ 0, ayuda) | 10.1, 10.2 (verificado) |
| Campo «Estado» (empieza en Activa) | 10.1, 11.3 (verificado) |
| Botón **Editar** (permiso `SeriesFiscales:Actualizar`) | 11.2 |
| Botones **Cancelar**, **Guardar** y **Eliminar** (permiso `SeriesFiscales:Eliminar`) | 10.1, 11.2, 11.4 (verificado) |
| Cuadro «Confirmar eliminación» («¿Estás seguro de que deseas eliminar esta serie fiscal?») | 11.4 (verificado) |
| «Serie fiscal guardada correctamente» | 10.3, 11.2, 11.3 (verificado) |
| «Ya existe otra serie de este tipo de comprobante con ese prefijo.» | 10.4 (verificado) |
| «La sucursal ya tiene una serie activa de este tipo. Desactívala antes de activar otra.» | 10.4 (verificado) |
| «El prefijo debe tener 4 caracteres en mayúsculas y empezar con la letra que exige el tipo de comprobante.» | 10.1, 10.2 (verificado) |
| «El correlativo debe ser un número entero mayor o igual a 0.» | 10.1, 10.2 (verificado) |
| «Este campo es obligatorio» | 10.1, 10.2 (verificado) |
| «Serie fiscal eliminada correctamente» | 11.4 (verificado) |
| Aviso «Esta serie ya emitió comprobantes: solo puedes cambiar su estado.» y campos bloqueados | 11.2 (No verificado en la aplicación) |
| «La serie ya emitió comprobantes: solo puedes cambiar su estado.» | 11.2 (No verificado en la aplicación) |
| «No se puede eliminar una serie que ya emitió comprobantes. Puedes desactivarla.» | 11.4 (No verificado en la aplicación) |
| «La serie fiscal no existe.», «La sucursal seleccionada no existe.», «Error al guardar la serie fiscal», «Error al eliminar la serie fiscal», «Error al cargar la serie fiscal» | 10.4, 11.4 (No verificado en la aplicación) |
| Códigos `Db.*` de guardado | No aplica: el formulario de series no los traduce; caen en «Error al guardar la serie fiscal» |

## 16. Fuentes

Documentos consultados:

- `docs/_inventario/REGLAS-AGENTES.md` y `docs/_inventario/MANUAL-AGENTES.md`.
- `docs/ola-1-fundamentos/19-inventario-de-pantallas/19b-administracion.md` (§0, §1, §2 y §6; versión 1.1).
- `docs/ola-1-fundamentos/19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md` (§6.1, §6.3, §6.5, §6.6, §8.3, §8.4, §8.9).
- `docs/ola-1-fundamentos/20-carga-masiva-y-exportacion.md` (§2, §3.11, §4.4, §4.17, §4.20).
- `docs/ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md` (EMP, SUC y SER) y `01c2-ventas-y-comprobantes.md` (CMP).
- `sofia-web-spa/public/i18n/roles/es.json` (nombres de módulos y acciones en la pantalla de permisos) y `sofia-web-spa/public/i18n/pos/es.json` (avisos de falta de RUC y de serie).

Recorridos hechos en la aplicación local (`http://localhost:4200`, API `https://localhost:7300`), con el administrador de la empresa de prueba:

1. Empresa: búsqueda sin filtros, con «Trial Activo», con «Activo» (0 resultados), por nombre parcial y con rangos de fecha; **Limpiar**; abrir el detalle; **Editar**, nombre vacío, nombre de más de 150 caracteres, RUC de 10 dígitos, RUC con una letra, RUC de otra empresa, RUC vacío, **Cancelar** y un guardado correcto sin cambios; **Exportar**.
2. Sucursales: búsqueda sin filtros y por licencia y dirección; **Limpiar**; paginador (cambio a 25); **Exportar**; abrir la «Sede Principal», **Editar**, guardar sin licencia, buscador de gerente, guardar con licencia.
3. Crear sucursal: formulario vacío, textos de más de 100 y 50 caracteres, licencia repetida, creación de «Sucursal Demo Norte», creación y eliminación de «Sucursal Prueba Eliminar» (con **Cancelar** del cuadro de confirmación y con **Confirmar**), intento de eliminar la «Sede Principal».
4. Carga masiva: pantalla, **Descargar plantilla**, rechazo de un archivo de texto, vista previa con un archivo `.xlsx` de prueba (dos versiones), **Guardar válidos** y eliminación de la sucursal creada («Sucursal Carga Uno»).
5. Series fiscales: búsqueda vacía (0 resultados), formulario vacío, prefijos `b00`, `f001` y `B-01`, correlativos `-3`, `1.5` y vacío, cambio de tipo con el prefijo escrito, creación de `B001`, intento con prefijo repetido, creación de `B002`, intento con segunda boleta activa, creación de `BC01` y `BC02`, creación de `PF01` (Proforma de prueba); búsqueda con filtros de sucursal, tipo y prefijo; **Limpiar**; **Exportar**; edición de `PF01` a `PF02` con correlativo 5; desactivación; eliminación con confirmación.

### Datos que quedan en la empresa de prueba

| Qué | Detalle |
|---|---|
| Sucursales | «Sede Principal», licencia `LIC-DEMO-0001`, dirección «Av. Ficticia 123, Miraflores, Lima»; «Sucursal Demo Norte», licencia `LIC-DEMO-0002`, dirección «Av. Ficticia Norte 456, Los Olivos, Lima». Las dos sin gerente |
| Series fiscales | `B001` Boleta y `BC01` Nota de Crédito de la «Sede Principal»; `B002` Boleta y `BC02` Nota de Crédito de la «Sucursal Demo Norte». Las cuatro «Activa», correlativo 0 |
| Empresa | Sin cambios: «Botica Manual Demo S.A.C.», RUC `20538291746`, «Trial Activo» |
| Eliminadas durante las pruebas | «Sucursal Prueba Eliminar» (`LIC-DEMO-0003`), «Sucursal Carga Uno» (`LIC-CARGA-0001`) y la serie `PF02` (Proforma de prueba) |
