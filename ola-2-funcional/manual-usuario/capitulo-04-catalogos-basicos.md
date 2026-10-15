# Capítulo 4 — Catálogos básicos: unidades de medida, laboratorios, ingredientes activos y catálogo DIGEMID

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-15 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Recorrido en la aplicación local más inventario de pantallas |

## 1. Introducción

En este capítulo aprendes a llenar los cuatro catálogos que SOFIA necesita antes de que puedas registrar medicamentos (capítulo 5). Vas a:

- Crear las **unidades de medida** que usarás para contar y vender tus productos (unidad, caja, blíster, frasco, miligramo…).
- Registrar los **laboratorios** (fabricantes o titulares) a los que pertenecerán tus medicamentos.
- Registrar los **ingredientes activos** (el principio activo de cada medicamento, con su DCI y su código ATC).
- Conocer el **catálogo DIGEMID**, que SOFIA usa para revisar los nombres de los medicamentos que detecta en una receta, y cargarlo desde un Excel.
- Buscar, exportar a Excel, editar y eliminar registros de esos catálogos, y usar la **carga masiva** para crear muchos de una sola vez.

Es para el **administrador** (rol «Admin», que lo puede todo). La sección 10 explica qué ve un usuario con menos permisos, por ejemplo un cajero.

Todos los ejemplos usan los datos ficticios del manual: la empresa «Botica Manual Demo S.A.C.» y la administradora «Ana Demo Prueba». Los laboratorios, los códigos de producto y los registros sanitarios que verás en los ejemplos son inventados.

Las jerarquías de unidades (cuántos blísteres entran en una caja, por ejemplo) **no** están en este capítulo: necesitan medicamentos y se explican en el capítulo 5.

## 2. Antes de empezar

- Haz los capítulos 1 a 3: debes tener tu empresa registrada y la sesión iniciada como administrador.
- Permisos que usa este capítulo. Los nombres son los que ves en la pantalla de permisos de un rol (Administración › Roles & Permisos): primero el módulo y luego la acción.

| Tarea | Módulo | Acciones |
|---|---|---|
| Ver, buscar y exportar unidades de medida | Unidades de Medida | `Leer` |
| Crear una unidad y usar su carga masiva | Unidades de Medida | `Crear` (además de `Leer`) |
| Editar una unidad | Unidades de Medida | `Actualizar` |
| Eliminar una unidad | Unidades de Medida | `Eliminar` |
| Lo mismo para laboratorios | Laboratorios | `Leer`, `Crear`, `Actualizar`, `Eliminar` |
| Lo mismo para ingredientes activos | Ingredientes Activos | `Leer`, `Crear`, `Actualizar`, `Eliminar` |
| Ver, buscar y exportar el catálogo DIGEMID | DIGEMID | `Leer` |
| Cargar productos al catálogo DIGEMID desde Excel | DIGEMID | `Crear` (además de `Leer`) |

- En el menú lateral, «Unidades de Medida», «Jerarquías de Unidades», «Laboratorios» e «Ingredientes Activos» están en el grupo **CATÁLOGOS**. «Catálogo DIGEMID» está en su propio grupo, **DIGEMID**.
- Los avisos de la aplicación (verdes de éxito o rojos de error) aparecen abajo al centro con un botón **Cerrar** y desaparecen solos a los pocos segundos. Los textos de este capítulo están copiados tal como se vieron en pantalla.

## 3. Qué es cada catálogo (en lenguaje simple)

| Catálogo | Qué es | Para qué lo usa SOFIA |
|---|---|---|
| **Unidad de medida** | Una forma de contar o medir: unidad, caja, blíster, frasco, miligramo, mililitro. Tiene un **código** corto (por ejemplo `UND`) y una **descripción** (por ejemplo «Unidad») | Cada medicamento elige una **unidad base**: la unidad más pequeña en la que cuentas ese producto. El stock, el precio base y el descuento de stock en las ventas siempre se expresan en unidades base. Las jerarquías (capítulo 5) dicen cuántas unidades menores entran en una mayor, y las presentaciones de venta (caja, blíster) usan estas unidades |
| **Laboratorio** | La empresa fabricante o titular de un producto farmacéutico. Tiene un **nombre de compañía** y, si quieres, un **código identificador** (puede ser un código interno o un registro) | Todo medicamento exige un laboratorio |
| **Ingrediente activo** | La sustancia que produce el efecto del medicamento (por ejemplo, el paracetamol). Se identifica con su **DCI** y su **código ATC** | Se asocia a los medicamentos mediante las formulaciones clínicas (capítulo 5) |
| **DCI** (denominación común internacional) | El nombre único, reconocido en todo el mundo, de un principio activo; también se conoce como nombre genérico. Ejemplos: «Paracetamol», «Amoxicilina» | Es el **nombre del ingrediente activo** en SOFIA. No puede repetirse en tu empresa |
| **Código ATC** | Código del sistema ATC de la OMS (clasificación anatómica, terapéutica y química), que ordena cada principio activo en cinco niveles. Un ejemplo con el formato real: `N02BE01` | Es un dato obligatorio del ingrediente. SOFIA **no valida** su estructura: acepta cualquier texto de hasta 15 caracteres, y dos ingredientes pueden tener el mismo código |
| **Catálogo DIGEMID** | La relación de productos farmacéuticos registrados ante DIGEMID (la autoridad peruana de medicamentos), con su código, nombre, concentración, forma farmacéutica, registro sanitario, titular y estado | Es una tabla **local** de consulta: SOFIA **no** se conecta a DIGEMID. Cuando analizas una receta con IA, SOFIA busca en este catálogo el nombre de cada medicamento detectado y, si lo encuentra, lo marca como «validado por DIGEMID» |

Fuente de las definiciones: glosario del proyecto (`06-glosario.md`: «Unidad base», «Unidad de medida», «Laboratorio», «DCI», «Código ATC», «Catálogo DIGEMID») y regla RN-REC-023 para el uso del catálogo en el análisis de recetas.

## 4. Lo que tienen en común las cuatro pantallas

Las cuatro pantallas de búsqueda se parecen mucho entre sí y a las de los capítulos 2 y 3. Aquí se explica una sola vez; en cada sección solo encontrarás lo propio.

**Pantalla de búsqueda.** Arriba, el título de la pantalla y a la derecha las migas («Inicio / Catálogos / Unidades de Medida», por ejemplo; en DIGEMID: «Inicio / DIGEMID / Catálogo DIGEMID»). Debajo hay una tarjeta «Búsqueda de …» con el texto «Ingresa los criterios de búsqueda». En la esquina superior derecha de la tarjeta están los botones **Carga Masiva** (icono de subida) y **Nuevo** (signo más); solo los ves con el permiso `Crear` del módulo, y el catálogo DIGEMID **no** tiene **Nuevo**. Debajo hay dos filtros de texto, cada uno con una lupa a la derecha, y al pie los botones **Limpiar** y **Buscar**.

- Los filtros buscan por «contiene» y no distinguen mayúsculas de minúsculas. Con **Enter** dentro de un filtro también se busca. Los dos filtros se combinan (deben cumplirse los dos).
- Los resultados **no** aparecen al abrir la pantalla: haz clic en **Buscar**. Bajo la tarjeta aparece «Resultados (N)» con la tabla; la primera columna es un enlace que abre el detalle.
- Si no hay coincidencias se ve «Resultados (0)», la fila vacía dice «Resultados» y el paginador dice «Sin resultados».
- **Limpiar** vacía los filtros y oculta la tabla.
- Bajo la tabla, a la izquierda, está el botón **Exportar** (icono de descarga) y, a la derecha, el paginador: «Items por página» (10, 25 o 50) y «Página 1 de 1» con los botones de primera, anterior, siguiente y última página. Con pocos registros siempre hay una sola página.
- Si abres un detalle y vuelves con la flecha de la esquina superior izquierda, la búsqueda recuerda tus filtros y repite la búsqueda sola.

**Pantalla de detalle (unidades, laboratorios e ingredientes).** La misma pantalla sirve para crear, ver y editar:

- **Nuevo** abre el formulario vacío y habilitado, con los botones **Cancelar** y **Guardar**. Al guardar bien, aparece el aviso verde y vuelves a la búsqueda **sin tabla** (haz clic en **Buscar**) o, si habías buscado antes, con la tabla y los filtros que tenías. **Cancelar** vuelve a la búsqueda sin guardar.
- Al abrir un registro existente ves una tarjeta con el título de la pantalla, el texto «Datos del registro» y, a la derecha, el botón **Editar** (con el permiso `Actualizar`). Los campos aparecen deshabilitados.
- **Editar** habilita los campos y muestra **Cancelar**, **Guardar** y **Eliminar** (este con el permiso `Eliminar`). Al guardar bien, aparece el aviso verde y **te quedas** en el detalle, otra vez en modo ver, con los datos recargados. **Cancelar** descarta lo que escribiste y vuelve al modo ver.
- **Eliminar** abre un cuadro «Confirmar eliminación» con el mensaje de la pantalla y los botones **Cancelar** (no borra nada) y **Confirmar**. Al confirmar, aparece el aviso verde y vuelves a la búsqueda. La eliminación es lógica: el registro desaparece de todas las listas y su código o nombre queda libre para volver a usarlo.
- Los campos de texto no dejan escribir más caracteres que su máximo, y no muestran ningún mensaje por ello (se comprobó tecleando 16 caracteres en un campo de 10: quedaron 10).

**Un detalle que debes conocer.** Si escribes solo espacios en un campo obligatorio, la pantalla lo acepta, pero el servidor lo rechaza y verás el aviso genérico de guardado de esa pantalla (por ejemplo «Error al guardar la unidad de medida»). Escribe siempre un valor real.

## 5. Unidades de medida

**Cómo llegar:** menú lateral › CATÁLOGOS › **Unidades de Medida** (`/unidades-medida`).

### 5.1 Qué unidades crear

Para los capítulos siguientes se crean estas unidades. Las siete primeras se crean una por una en esta sección; «Gramo» y «Kilogramo» se crean con la carga masiva (sección 9.2).

| Código | Descripción | Para qué se usará |
|---|---|---|
| `UND` | Unidad | Unidad base de la mayoría de los productos |
| `CAJ` | Caja | Presentación de venta y unidad mayor de las jerarquías |
| `BLI` | Blíster | Presentación de venta y unidad intermedia |
| `FRA` | Frasco | Unidad base de jarabes y gotas |
| `TAB` | Tableta | Unidad base de tabletas y comprimidos |
| `MG` | Miligramo | Dosis en las formulaciones clínicas |
| `ML` | Mililitro | Volumen de jarabes y soluciones |
| `GR` | Gramo | Carga masiva |
| `KG` | Kilogramo | Carga masiva |

### 5.2 La pantalla de búsqueda

Título «Unidades de Medida», migas «Inicio / Catálogos / Unidades de Medida» y tarjeta «Búsqueda de Unidades de Medida». Filtros:

| Filtro | Qué hace |
|---|---|
| Código | Texto con lupa. Busca códigos que **contengan** lo que escribas (`m` encuentra `MG` y `ML`) |
| Descripción | Texto con lupa. Busca descripciones que contengan lo que escribas, sin importar mayúsculas (`UNIDAD` encuentra «Unidad» y «Unidad de prueba») |

La tabla tiene dos columnas, **Código** (enlace al detalle) y **Descripción**, y se ordena por descripción. No tiene filtros numéricos ni de fecha.

### 5.3 Crear una unidad

1. Haz clic en **Nuevo**. Se abre el formulario (`/unidades-medida/nueva`) con los campos «Código» y «Descripción».
2. Escribe el código y la descripción.
3. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Código | Una abreviatura corta, por ejemplo `UND` | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 10 caracteres. No puede repetirse en tu empresa: «Ya existe una unidad de medida con ese código.» (también con distinta capitalización: `und` fue rechazado al existir `UND`) |
| Descripción | El nombre completo, por ejemplo «Unidad» | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 50 caracteres |

**Probar los errores (comprobado en la aplicación):**

1. Haz clic en **Guardar** con el formulario vacío: debajo de «Código» y de «Descripción» aparece «Este campo es obligatorio» y no se envía nada.
2. Escribe 16 letras en «Código»: el campo se queda en 10.
3. Escribe solo espacios en «Código» y una descripción cualquiera: aparece el aviso rojo «Error al guardar la unidad de medida» y te quedas en el formulario.
4. Guarda `UND` / «Unidad». Luego intenta crear otra unidad con código `UND`: aviso rojo «Ya existe una unidad de medida con ese código.» y te quedas en el formulario.

**Qué pasa al terminar bien:** aparece el aviso «Unidad de medida guardada correctamente» y el sistema te lleva a la búsqueda de unidades. Repite los pasos con las demás unidades de la tabla 5.1.

Si no ves **Nuevo**, a tu rol le falta el permiso `Crear` del módulo «Unidades de Medida».

Errores al guardar una unidad:

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Ya existe una unidad de medida con ese código.» | Otra unidad de tu empresa ya usa ese código (comprobado) | Elige otro código |
| «Error al guardar la unidad de medida» | El servidor rechazó el guardado; lo viste con un código de solo espacios (comprobado). También sale ante cualquier otro problema | Revisa que los dos campos tengan texto real e inténtalo de nuevo |
| «Ya existe un registro con ese mismo valor. Revisa los datos e inténtalo de nuevo.», «El registro hace referencia a datos que ya no existen o todavía está en uso.» o «Uno de los valores está fuera del rango permitido.» | Errores de base de datos poco frecuentes (por ejemplo, dos personas creando el mismo código a la vez) | Revisa los datos e inténtalo de nuevo. (No verificado en la aplicación: no se pudieron provocar; vienen del inventario 19c §0) |

### 5.4 Buscar, filtrar y exportar

1. En la búsqueda, haz clic en **Buscar** sin filtros: ves todas las unidades. Con las nueve unidades de la tabla 5.1 se ve «Resultados (9)»: Blíster, Caja, Frasco, Gramo, Kilogramo, Miligramo, Mililitro, Tableta y Unidad (ordenadas por descripción).
2. Prueba los filtros (comprobado): «Código» = `m` trae `MG` y `ML`; «Descripción» = `UNIDAD` trae las unidades cuya descripción contiene «unidad»; «Código» = `u` junto con «Descripción» = `prueba` trae solo la unidad de prueba; «Código» = `zzz` muestra «Resultados (0)», la fila «Resultados» y el paginador «Sin resultados».
3. Haz clic en **Limpiar** para vaciar los filtros y ocultar la tabla.
4. Haz clic en **Buscar** otra vez y luego en **Exportar**. El sistema genera un archivo `unidades-medida.xlsx` con los resultados de los filtros que tengas escritos en ese momento.

El archivo trae una hoja «Data» con las columnas «Código» y «Descripción». Comprobado en la aplicación: al hacer clic en **Exportar** el servidor respondió correctamente (200) y la pantalla no mostró ningún aviso. (No verificado en la aplicación: el contenido del archivo ni su descarga, porque el navegador integrado no muestra los archivos descargados; las columnas y el nombre vienen del inventario 19c §1.1 y del documento 20 §4.21.) Si falla la búsqueda o la exportación verás «Error al buscar unidades de medida».

### 5.5 Ver y editar una unidad

1. En los resultados, haz clic en el código (por ejemplo `PRU`, la unidad de prueba de la sección 5.6). Se abre el detalle con los dos campos deshabilitados y el botón **Editar**.
2. Haz clic en **Editar**, cambia la descripción y haz clic en **Cancelar**: la descripción vuelve a su valor anterior y el detalle vuelve al modo ver.
3. Haz clic en **Editar**, cambia la descripción y haz clic en **Guardar**: aparece «Unidad de medida guardada correctamente» y te quedas en el detalle, con la nueva descripción.
4. Si cambias el código a uno que ya usa otra unidad, al guardar ves «Ya existe una unidad de medida con ese código.» (comprobado cambiando `PRU` por `UND`). Haz clic en **Cancelar** para descartar.

Si no ves **Editar**, te falta el permiso `Actualizar` del módulo «Unidades de Medida». Si el detalle no se pudo abrir verás «Error al cargar la unidad de medida» (No verificado en la aplicación: no se pudo provocar).

### 5.6 Eliminar una unidad

Para probarlo se creó una unidad de prueba, `PRU` / «Unidad de prueba», que se eliminó al final (no queda en la empresa de prueba).

1. Abre la unidad, haz clic en **Editar** y luego en **Eliminar**.
2. Se abre el cuadro «Confirmar eliminación» con el mensaje «¿Estás seguro de que deseas eliminar esta unidad de medida?» y los botones **Cancelar** y **Confirmar**. **Cancelar** cierra el cuadro y no borra nada (comprobado).
3. Haz clic en **Confirmar**: aparece «Unidad de medida eliminada correctamente» y vuelves a la búsqueda, que recuerda tus filtros y repite la búsqueda.

El código de una unidad eliminada queda libre: se comprobó volviendo a crear `PRU` después de eliminarla.

**Cuando la unidad está en uso.** SOFIA no deja eliminar una unidad que aún usen medicamentos (como unidad base), jerarquías (como unidad mayor o menor) o formulaciones clínicas. El aviso rojo es: «No se puede eliminar la unidad de medida porque está asociada a medicamentos o jerarquías de unidades.» (el texto no menciona las formulaciones clínicas, aunque también bloquean). Cualquier otro fallo al eliminar muestra «Error al eliminar la unidad de medida». (No verificado en la aplicación: todavía no existe ningún medicamento ni jerarquía que use una unidad, así que no se pudo provocar; viene del inventario 19c §1.2 y §6. **Se vuelve a probar en el capítulo 5**, cuando existan medicamentos y jerarquías.)

Si no ves **Eliminar**, te falta el permiso `Eliminar` del módulo «Unidades de Medida».

### 5.7 Carga masiva

La carga masiva de unidades se explica en la sección 9.2.

## 6. Laboratorios

**Cómo llegar:** menú lateral › CATÁLOGOS › **Laboratorios** (`/laboratorios`).

### 6.1 Qué laboratorios crear

Se crean dos laboratorios ficticios, que usarán los medicamentos del capítulo 5:

| Nombre de Compañía | Código Identificador |
|---|---|
| Laboratorio Demo Andino | `DEMO-AND` |
| Laboratorio Demo Pacífico | (sin código) |

### 6.2 La pantalla de búsqueda

Título «Laboratorios», migas «Inicio / Catálogos / Laboratorios», tarjeta «Búsqueda de Laboratorios». Filtros:

| Filtro | Qué hace |
|---|---|
| Nombre de Compañía | Texto con lupa. Busca nombres que contengan lo que escribas |
| Código Identificador | Texto con lupa. Busca códigos que contengan lo que escribas |

La tabla tiene dos columnas: **Nombre de Compañía** (enlace al detalle) y **Código Identificador** (muestra «—» cuando el laboratorio no tiene código). Se ordena por nombre.

### 6.3 Crear un laboratorio

1. Haz clic en **Nuevo** (`/laboratorios/nueva`).
2. Escribe el nombre y, si quieres, el código.
3. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Nombre de Compañía | El nombre del fabricante o titular | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 150 caracteres. No puede repetirse en tu empresa (sin importar mayúsculas): «Ya existe un laboratorio con ese nombre de compañía.» |
| Código Identificador | Un código interno o de registro, si lo tienes | No | No deja escribir más de 50 caracteres y no muestra mensaje por ello. Si lo escribes, no puede repetirse en otro laboratorio: «Ya existe un laboratorio con ese código identificador.» Si lo dejas vacío (o lo borras), el laboratorio queda «sin código» y puedes tener varios así |

**Probar los errores (comprobado en la aplicación):**

1. Haz clic en **Guardar** con el formulario vacío: «Este campo es obligatorio» bajo «Nombre de Compañía» y no se envía nada.
2. Escribe solo espacios en el nombre: aviso rojo «Error al guardar el laboratorio».
3. Guarda «Laboratorio Demo Andino» con código `DEMO-AND`: aviso «Laboratorio guardado correctamente» y vuelves a la búsqueda.
4. Intenta crear «laboratorio demo andino» (en minúsculas): aviso rojo «Ya existe un laboratorio con ese nombre de compañía.»
5. Intenta crear «Laboratorio Otro» con el código `DEMO-AND`: aviso rojo «Ya existe un laboratorio con ese código identificador.»
6. Guarda «Laboratorio Demo Pacífico» sin código: «Laboratorio guardado correctamente».

Si no ves **Nuevo**, te falta el permiso `Crear` del módulo «Laboratorios».

Errores al guardar:

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Ya existe un laboratorio con ese nombre de compañía.» | El nombre ya está registrado (comprobado) | Usa otro nombre o abre el existente |
| «Ya existe un laboratorio con ese código identificador.» | El código ya lo usa otro laboratorio (comprobado) | Usa otro código o déjalo vacío |
| «Error al guardar el laboratorio» | Nombre de solo espacios (comprobado) u otro problema del servidor | Revisa los datos e inténtalo de nuevo |
| Los tres avisos de base de datos de la sección 5.3 | Errores poco frecuentes | Igual que allí (No verificado en la aplicación) |

### 6.4 Buscar y exportar

Comprobado con los laboratorios de la empresa de prueba:

- **Buscar** sin filtros: «Resultados (2)», «Laboratorio Demo Andino» (`DEMO-AND`) y «Laboratorio Demo Pacífico» (`—`).
- «Nombre de Compañía» = `demo`: los dos. «Código Identificador» = `and`: solo el Andino. «Código Identificador» = `zzz`: «Resultados (0)», la fila «Resultados» y el paginador «Sin resultados».
- **Exportar** genera `laboratorios.xlsx` con una hoja «Data» y las columnas «Nombre de Compañía» y «Código Identificador» (vacío si no tiene). Comprobado: el servidor respondió 200 y no hubo aviso de error. (No verificado en la aplicación: el contenido del archivo ni su descarga; viene de 19c §3.1 y del documento 20 §4.8.) Si falla la búsqueda o la exportación: «Error al buscar laboratorios».

### 6.5 Ver, editar y eliminar un laboratorio

Se creó un laboratorio de prueba, «Laboratorio Prueba Borrar» con código `PRUEBA-01`, para estas pruebas. Ya no existe.

1. Haz clic en el nombre en la tabla: se abre el detalle con los dos campos deshabilitados y el botón **Editar**.
2. **Editar**: se habilitan los campos y aparecen **Cancelar**, **Guardar** y **Eliminar**. En la prueba se borró el código y se cambió el nombre a «Laboratorio Prueba Borrar (editado)». Al guardar: «Laboratorio guardado correctamente», te quedas en el detalle y el código aparece vacío (el laboratorio quedó sin código).
3. Con el nombre vacío, **Guardar** muestra «Este campo es obligatorio». **Cancelar** descarta los cambios.
4. **Eliminar** abre «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este laboratorio?» y los botones **Cancelar** y **Confirmar**. Al confirmar: «Laboratorio eliminado correctamente» y vuelves a la búsqueda.
5. **Cuando el laboratorio está en uso:** si algún medicamento no eliminado lo usa, el aviso rojo es «No se puede eliminar el laboratorio porque tiene medicamentos asociados.». Otro fallo: «Error al eliminar el laboratorio». (No verificado en la aplicación: todavía no hay medicamentos; se vuelve a probar en el capítulo 5.)

Los permisos para editar y eliminar son `Actualizar` y `Eliminar` del módulo «Laboratorios». Si el detalle no se puede abrir: «Error al cargar el laboratorio» (No verificado).

La carga masiva de laboratorios se explica en la sección 9.3.

## 7. Ingredientes activos

**Cómo llegar:** menú lateral › CATÁLOGOS › **Ingredientes Activos** (`/ingredientes-activos`).

### 7.1 Qué ingredientes crear

Se crean cuatro ingredientes, uno por cada medicamento del capítulo 5. Los códigos ATC son los del sistema de clasificación de la OMS; escríbelos tal cual o, si no estás seguro de alguno, confirma el código en la lista oficial antes de usarlo en tu botica (SOFIA no lo comprueba).

| Denominación DCI | Código ATC |
|---|---|
| Paracetamol | `N02BE01` |
| Amoxicilina | `J01CA04` |
| Clonazepam | `N03AE01` |
| Tramadol | `N02AX02` |

Con la carga masiva (sección 9.4) se agregan además «Ibuprofeno» (`M01AE01`) y «Omeprazol» (`A02BC01`).

### 7.2 La pantalla de búsqueda

Título «Ingredientes Activos», migas «Inicio / Catálogos / Ingredientes Activos», tarjeta «Búsqueda de Ingredientes Activos». Filtros:

| Filtro | Qué hace |
|---|---|
| Denominación DCI | Texto con lupa. Busca nombres que contengan lo que escribas |
| Código ATC | Texto con lupa. Busca códigos que contengan lo que escribas |

La tabla tiene dos columnas: **Denominación DCI** (enlace al detalle) y **Código ATC**. Se ordena por denominación.

### 7.3 Crear un ingrediente activo

1. Haz clic en **Nuevo** (`/ingredientes-activos/nueva`).
2. Escribe la denominación DCI y el código ATC.
3. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Denominación DCI | El nombre genérico del principio activo | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 255 caracteres. No puede repetirse en tu empresa (sin importar mayúsculas): «Ya existe un ingrediente activo con esa denominación DCI.» |
| Código ATC | El código de la clasificación ATC | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 15 caracteres. SOFIA no revisa su estructura (cualquier texto sirve, se comprobó guardando `XXX`) y **sí se puede repetir** entre ingredientes (se comprobó con dos ingredientes con `N02BE01`) |

**Probar los errores (comprobado en la aplicación):**

1. **Guardar** con el formulario vacío: «Este campo es obligatorio» bajo cada campo, sin envío.
2. Solo espacios en los dos campos: aviso rojo «Error al guardar el ingrediente activo».
3. Guarda «Paracetamol» / `N02BE01`: «Ingrediente activo guardado correctamente» y vuelves a la búsqueda.
4. Intenta crear «paracetamol» (minúsculas) con otro código: «Ya existe un ingrediente activo con esa denominación DCI.»
5. Guarda «Amoxicilina», «Clonazepam» y «Tramadol» con los códigos de la tabla 7.1.

Si no ves **Nuevo**, te falta el permiso `Crear` del módulo «Ingredientes Activos».

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «Ya existe un ingrediente activo con esa denominación DCI.» | La DCI ya está registrada (comprobado) | Abre el ingrediente existente |
| «Error al guardar el ingrediente activo» | Valor de solo espacios (comprobado) u otro problema | Revisa los datos e inténtalo de nuevo |
| Los tres avisos de base de datos de la sección 5.3 | Errores poco frecuentes | Igual que allí (No verificado en la aplicación) |

### 7.4 Buscar y exportar

Comprobado con los ingredientes de la empresa de prueba: «Denominación DCI» = `CIL` trae «Amoxicilina»; «Código ATC» = `n02` trae los ingredientes cuyo código contiene «N02» (Paracetamol y Tramadol, más el de prueba); `para` con `n02` trae solo «Paracetamol». **Exportar** genera `ingredientes-activos.xlsx` con las columnas «Denominación DCI» y «Código ATC» (el servidor respondió 200 y no hubo aviso de error; No verificado en la aplicación: el contenido del archivo; viene de 19c §4.1 y del documento 20 §4.6). Fallo de búsqueda o exportación: «Error al buscar ingredientes activos».

### 7.5 Ver, editar y eliminar

Se creó «Ingrediente Prueba Borrar» (`N02BE01`) para probar. Al abrirlo y pulsar **Editar** se cambió el código a `XXX` y se guardó («Ingrediente activo guardado correctamente», sin validar la estructura). **Eliminar** abre «Confirmar eliminación» con «¿Estás seguro de que deseas eliminar este ingrediente activo?»; al confirmar aparece «Ingrediente activo eliminado correctamente» y vuelves a la búsqueda.

**Cuando el ingrediente está en uso:** SOFIA no deja eliminar un ingrediente incluido en alguna formulación clínica (capítulo 5); el aviso rojo es «No puedes eliminar este ingrediente activo porque está incluido en formulaciones clínicas.». Otro fallo: «Error al eliminar el ingrediente activo». (No verificado en la aplicación: todavía no hay formulaciones clínicas; se vuelve a probar en el capítulo 5.) Si el detalle no se puede abrir: «Error al cargar el ingrediente activo» (No verificado).

Permisos: `Actualizar` para **Editar** y `Eliminar` para **Eliminar**, ambos del módulo «Ingredientes Activos».

## 8. Catálogo DIGEMID

**Cómo llegar:** menú lateral › DIGEMID › **Catálogo DIGEMID** (`/digemid-catalogo`). El grupo **DIGEMID** solo tiene esta opción.

### 8.1 Qué es y para qué lo usa SOFIA

DIGEMID es la Dirección General de Medicamentos, Insumos y Drogas del Ministerio de Salud. Su relación de productos farmacéuticos registrados es la base de este catálogo. En SOFIA el catálogo es una **tabla local de tu empresa**: la llenas tú cargando un Excel, y SOFIA **no** se conecta a DIGEMID para actualizarla ni consultarla.

Se usa para el **análisis de recetas con IA** (capítulo de recetas): por cada medicamento que la IA detecta en la foto de una receta, SOFIA busca su nombre en este catálogo; si encuentra coincidencias, marca el medicamento como «validado por DIGEMID» (con la razón «Validado por DIGEMID (nombre)»); si no, queda como «Detectado en receta». Un catálogo vacío no impide analizar recetas, pero ningún medicamento saldrá validado. (Fuente: regla RN-REC-023; el análisis de recetas se recorre en un capítulo posterior.)

Una empresa recién registrada tiene el catálogo **vacío**: comprobado, la búsqueda de la empresa de prueba mostró «Resultados (0)» antes de cargar nada.

### 8.2 Qué puedes hacer y qué no en esta pantalla

| Acción | ¿Se puede? |
|---|---|
| Buscar por código y por nombre del producto | Sí (sección 8.3) |
| Abrir un producto y ver todos sus datos | Sí, solo lectura (sección 8.4) |
| Exportar la lista a Excel | Sí (sección 8.3) |
| Cargar productos desde un Excel (carga masiva) | Sí, con el permiso `Crear` (sección 8.5) |
| Crear un producto uno por uno | **No.** No existe el botón **Nuevo** |
| Editar un producto | **No.** El detalle no tiene el botón **Editar** y los campos siempre están deshabilitados |
| Eliminar un producto | **No.** No existe el botón **Eliminar** |

Si necesitas agregar o corregir productos, hazlo con la carga masiva. (La API tiene operaciones para crear, editar y eliminar un producto, pero ninguna pantalla las usa; no las uses desde esta guía.)

Se comprobó escribiendo la dirección `/digemid-catalogo/nueva` a mano: la pantalla se abre con el formulario de detalle vacío y deshabilitado y muestra un aviso de error de carga; no se puede crear nada.

### 8.3 Buscar y exportar

Título «Catálogo DIGEMID», migas «Inicio / DIGEMID / Catálogo DIGEMID», tarjeta «Búsqueda en Catálogo DIGEMID». Arriba a la derecha de la tarjeta solo hay un botón: **Carga Masiva** (con el permiso `Crear`). Filtros:

| Filtro | Qué hace |
|---|---|
| Código de Producto | Texto con lupa. Busca códigos que contengan lo que escribas, sin importar mayúsculas (`demo-000` encuentra `DEMO-0001`) |
| Nombre del Producto | Texto con lupa. Busca nombres que contengan lo que escribas, sin importar mayúsculas. **Sí distingue los acentos**: `cápsula` (con tilde) no encontró «CAPSULA» (sin tilde) y `capsula` sí. Escribe el nombre tal como está en el catálogo |

Tabla de resultados, ordenada por nombre de producto:

| Columna | Qué muestra |
|---|---|
| Código | El código del producto; es un enlace al detalle |
| Nombre del Producto | El nombre |
| Forma Farmacéutica | La forma («Tableta», «Cápsula»…) o «—» si no tiene |
| Estado | El estado tal como se cargó (texto libre) |

La tabla no muestra concentración, registro sanitario ni titular; los ves en el detalle. Comprobado con los productos de la sección 8.5: «Código de Producto» = `0003` trae «DEMO-0003»; «Nombre del Producto» = `tabl` trae los tres productos «TABLETA»; `demo-000` junto con `capsula` trae «DEMO-0002» y «DEMO-0004». Sin resultados: «Resultados (0)» y «Sin resultados» en el paginador.

**Exportar** genera `digemid-catalogo.xlsx`, con una hoja «Data» y cuatro columnas: «Código», «Nombre del Producto», «Forma Farmacéutica» y «Estado» (las mismas de la tabla; **no** incluye concentración, fracción, registro sanitario ni titular). Comprobado: el servidor respondió 200 y no hubo aviso de error. (No verificado en la aplicación: el contenido del archivo; viene de 19c §5.1 y del documento 20 §4.2.) Fallo de búsqueda o exportación: «Error al buscar en el catálogo DIGEMID».

### 8.4 Ver un producto

1. En los resultados, haz clic en el código (por ejemplo `DEMO-0001`).
2. Se abre el detalle: flecha de volver, título «Catálogo DIGEMID» y migas, y una tarjeta **sin** botones de edición. Los ocho campos aparecen siempre deshabilitados:

| Campo | Ejemplo (producto `DEMO-0001`) |
|---|---|
| Código de Producto | DEMO-0001 |
| Nombre del Producto | PARACETAMOL 500 MG TABLETA (DEMO) |
| Concentración | 500 mg |
| Forma Farmacéutica | Tableta |
| Fracción | 10 |
| Registro Sanitario | RS-DEMO-0001 |
| Titular | Laboratorio Demo Andino |
| Estado | ACTIVO |

Los datos que el producto no tiene se ven como campo vacío. Con la flecha de volver regresas a la búsqueda, que recuerda tus filtros. Si el producto no se puede abrir verás «Error al cargar el producto DIGEMID» (se vio la clave sin traducir, `digemidCatalogo.detail.load-error`, al recargar la página directamente en `/digemid-catalogo/nueva`; ver la sección 11).

### 8.5 Cargar productos desde un Excel (carga masiva)

Es la **única** forma de agregar productos al catálogo. Aquí se recorre el flujo completo; los otros tres catálogos usan exactamente la misma pantalla (sección 9).

**Cómo llegar:** **Catálogo DIGEMID** › **Carga Masiva** (`/digemid-catalogo/carga-masiva`). Necesitas el permiso `Crear` del módulo «DIGEMID». Sin ese permiso no ves el botón y, si escribes la dirección a mano, el sistema te devuelve al Inicio con «No tienes permisos para acceder a esta pantalla.» (No verificado en la aplicación: el administrador tiene todos los permisos.)

**La pantalla.** Flecha de volver, título «Carga Masiva» y migas «Inicio / DIGEMID / Catálogo DIGEMID / Carga Masiva». Una tarjeta con el botón **Descargar plantilla** (arriba a la derecha) y una zona punteada con el texto «Arrastra tu archivo Excel aquí o haz clic para seleccionar» y debajo «Solo archivos .xlsx (máx. 5 MB y 5000 filas)».

**Pasos:**

1. Haz clic en **Descargar plantilla**. Se guarda `plantilla-digemid-catalogo.xlsx` (comprobado que el servidor responde 200 y la pantalla no muestra error), con una sola hoja «Plantilla», una fila de títulos azul y ninguna fila de ejemplo ni listas desplegables. Las columnas, escritas **exactamente así**, son:

| Columna en el Excel | Qué escribir | Obligatoria | Límite |
|---|---|---|---|
| `CodProd` | Código del producto | Sí | 20 caracteres |
| `NomProd` | Nombre del producto | Sí | 255 |
| `Concent` | Concentración | No | 255 |
| `FormaFarmaceutica` | Forma farmacéutica | No | 150 |
| `Fraccion` | Fracción | No | 100 |
| `RegistroSanitario` | Registro sanitario | No | 50 |
| `Titular` | Titular del registro | No | 255 |
| `Estado` | Estado (texto libre, por ejemplo `ACTIVO`) | Sí | 50 |

2. Llena el Excel: una fila por producto, desde la fila 2. Guárdalo como `.xlsx`. El orden de las columnas no importa, las columnas extra se ignoran y los títulos deben estar escritos tal cual (sin tildes ni cambios), o esa columna se lee vacía.
3. Elige el archivo: arrástralo a la zona punteada o haz clic en ella. La pantalla comprueba en tu navegador que el nombre termine en `.xlsx` y que pese menos de 5 MB.
4. Revisa la **Vista previa**. Aparece una tarjeta con tres contadores («N filas totales», «N filas válidas» y «N filas con error», este último solo si hay errores) y una tabla con una fila por producto. Las columnas son: «Cód. Producto», «Nombre del Producto», «Concentración», «Forma Farmacéutica», «Fracción», «Registro Sanitario», «Titular» y «Estado» (el estado del producto), una columna **sin título** con los mensajes de error de la fila y una segunda columna «Estado» (la validez de la fila: una marca verde si es válida o un icono de error si no). Aún no se guardó nada.
5. Si hay filas con error, corrígelas en tu Excel y vuelve a subirlo: la vista previa nueva reemplaza a la anterior. No puedes corregir filas dentro de la pantalla.
6. Haz clic en **Guardar válidos (N)**, bajo la tabla. Solo se guardan las filas válidas. El botón está deshabilitado si no hay ninguna.
7. **Resultado:** aparece el aviso «Se guardaron N registros correctamente» y el sistema te lleva a la búsqueda del catálogo.

**Lo que se hizo en la aplicación.** Se subió un archivo de prueba con 9 filas (se construyó dentro del navegador, ver «No verificado»): cinco productos válidos con ingredientes de la sección 7 y cuatro filas con problemas.

| Código | Nombre del Producto | Resultado en la vista previa |
|---|---|---|
| DEMO-0001 | PARACETAMOL 500 MG TABLETA (DEMO) | Válida |
| DEMO-0002 | AMOXICILINA 500 MG CAPSULA (DEMO) | Válida |
| DEMO-0003 | CLONAZEPAM 2 MG TABLETA (DEMO) | Válida |
| DEMO-0004 | TRAMADOL 50 MG CAPSULA (DEMO) | Válida |
| DEMO-0005 | IBUPROFENO 400 MG TABLETA (DEMO) | Válida |
| (vacío) | PRODUCTO SIN CODIGO (DEMO) | «El campo 'codProd' es obligatorio» |
| DEMO-0007 | PRODUCTO SIN ESTADO (DEMO) | «El campo 'estado' es obligatorio» |
| DEMO-0008 | PRODUCTO REPETIDO UNO (DEMO) | «El valor 'DEMO-0008' del campo 'codProd' está duplicado en el archivo» |
| demo-0008 | PRODUCTO REPETIDO DOS (DEMO) | «El valor 'demo-0008' del campo 'codProd' está duplicado en el archivo» |

La vista previa mostró «9 filas totales», «5 filas válidas» y «4 filas con error», y el botón **Guardar válidos (5)**. Al hacer clic apareció «Se guardaron 5 registros correctamente» y la pantalla volvió a la búsqueda; con **Buscar** se ven los cinco productos (ordenados por nombre: Amoxicilina, Clonazepam, Ibuprofeno, Paracetamol, Tramadol). Fíjate en que **las dos filas repetidas quedaron con error, incluida la primera**, y que `DEMO-0008` y `demo-0008` cuentan como el mismo código (no distingue mayúsculas).

Para completar la prueba se subió después un segundo archivo con otros dos casos:

| Fila | Resultado en la vista previa |
|---|---|
| `demo-0001` (código que ya existe en el catálogo) | «El valor 'demo-0001' del campo 'codProd' ya existe en el sistema» |
| `DEMO-0009` con un registro sanitario de 60 caracteres | **Válida** (la vista previa no revisa las longitudes) |

Al hacer clic en **Guardar válidos (1)** apareció el aviso rojo «Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa.», te quedaste en la pantalla y **no se guardó nada**. Es un comportamiento importante: en el catálogo DIGEMID, **un texto demasiado largo pasa la vista previa y hace fallar el guardado completo**. Antes de subir, revisa los límites de la tabla del paso 1 (en especial `RegistroSanitario`, 50 caracteres).

**Errores del archivo y del guardado** (los de esta tabla se repiten en los cuatro catálogos):

| Mensaje | Qué significa | Qué hacer |
|---|---|---|
| «El archivo no es un Excel .xlsx válido. Descarga la plantilla y vuelve a intentarlo.» | El archivo no termina en `.xlsx` o está dañado. Comprobado subiendo un archivo `notas.txt` | Usa un `.xlsx` creado desde la plantilla |
| «El archivo supera el tamaño máximo de 5 MB.» | Pesa más de 5 MB | Divide el archivo |
| «El archivo supera el máximo de 5000 filas. Divídelo en varios archivos.» | Más de 5000 filas con datos | Divídelo |
| «El archivo excede los límites de contenido permitidos. Revisa que sea una plantilla válida.» | Archivo sospechoso (demasiado comprimido) | Vuelve a crearlo desde la plantilla |
| «Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa.» | El servidor rechazó filas al guardar (comprobado con el texto demasiado largo) | Corrige el Excel y súbelo de nuevo |
| «Error al procesar el archivo» / «Error al guardar los registros» | Falla genérica en la vista previa / al guardar | Inténtalo de nuevo |
| «Error al descargar la plantilla. Intenta de nuevo.» | No se pudo bajar la plantilla | Inténtalo de nuevo |
| «No hay filas válidas para guardar» | Se intentó guardar sin filas válidas (el botón lo impide) | — |

Mensajes por fila de la vista previa que puedes ver en cualquier catálogo (el nombre del campo sale como lo usa el sistema por dentro, por ejemplo `codProd`, no con la etiqueta de la pantalla): «El campo '…' es obligatorio», «'…' no puede superar los N caracteres», «El valor '…' del campo '…' ya existe en el sistema» y «El valor '…' del campo '…' está duplicado en el archivo».

(No verificado en la aplicación: el contenido de la plantilla ni de los archivos exportados, porque el navegador integrado no muestra descargas; los errores de tamaño, de más de 5000 filas, de contenido y de servidor, porque no se prepararon archivos de ese tipo; y el gesto de arrastrar o hacer clic para elegir el archivo, porque el navegador integrado no tiene herramienta para elegir archivos del disco: se entregó el archivo de prueba a la pantalla desde el propio navegador.)

## 9. Cargar unidades, laboratorios e ingredientes desde un Excel

Las tres cargas masivas funcionan igual que la del catálogo DIGEMID (sección 8.5): **Descargar plantilla**, elegir el archivo, revisar la vista previa y **Guardar válidos (N)**. Cambian las columnas, las reglas de cada fila y la pantalla a la que vuelves. Para cada una se recorrió la aplicación con un archivo de prueba.

### 9.1 Antes de cargar

- Entra por el botón **Carga Masiva** de la pantalla de búsqueda (necesitas `Crear` del módulo). La pantalla tiene el mismo aspecto que en la sección 8.5, con las migas del catálogo (por ejemplo «Inicio / Catálogos / Unidades de Medida / Carga Masiva»).
- La plantilla se guarda como `plantilla-unidades-medida.xlsx`, `plantilla-laboratorios.xlsx` o `plantilla-ingredientes-activos.xlsx`. Trae solo la fila de títulos.
- Una fila **repetida en el archivo** se marca con error **en todas sus apariciones**, también la primera.
- Los límites son los de la sección 8.5: 5 MB y 5000 filas.

### 9.2 Unidades de medida

**Columnas de la plantilla:**

| Columna en el Excel | Qué escribir | Obligatoria | Límite |
|---|---|---|---|
| `Codigo` | Código de la unidad (sin tilde) | Sí | 10 caracteres |
| `Descripcion` | Descripción (sin tilde) | Sí | 50 caracteres |

La vista previa muestra las columnas «Código» y «Descripción», la columna de errores y «Estado». Las reglas por fila: código y descripción obligatorios, máximos de 10 y 50 caracteres, código que ya existe en tu empresa, y código repetido en el archivo (todas las filas repetidas quedan con error, sin distinguir mayúsculas). La descripción **sí** puede repetirse.

**Lo que se comprobó.** Se subió un archivo con 7 filas:

| Código | Descripción | Resultado |
|---|---|---|
| GR | Gramo | Válida |
| KG | Kilogramo | Válida |
| (vacío) | Fila sin código | «El campo 'codigo' es obligatorio» |
| UND | Unidad repetida | «El valor 'UND' del campo 'codigo' ya existe en el sistema» |
| LT | Litro | «El valor 'LT' del campo 'codigo' está duplicado en el archivo» |
| LT | Litro dos | «El valor 'LT' del campo 'codigo' está duplicado en el archivo» |
| CODIGOMUYLARGO | (60 letras) | «'codigo' no puede superar los 10 caracteres» y «'descripcion' no puede superar los 50 caracteres» (dos mensajes en la misma fila) |

La vista previa dijo «7 filas totales», «2 filas válidas» y «5 filas con error». **Guardar válidos (2)** mostró «Se guardaron 2 registros correctamente» y llevó a la búsqueda de unidades. Se crearon `GR` / «Gramo» y `KG` / «Kilogramo».

### 9.3 Laboratorios

**Columnas de la plantilla:**

| Columna en el Excel | Qué escribir | Obligatoria | Límite |
|---|---|---|---|
| `NombreCompania` | Nombre de la compañía (sin tilde) | Sí | 150 caracteres |
| `CodigoIdentificador` | Código identificador | No | 50 caracteres |

La vista previa muestra «Nombre de Compañía», «Código Identificador», la columna de errores y «Estado». Reglas por fila: nombre obligatorio, máximos de 150 y 50 caracteres, nombre que ya existe y nombre repetido en el archivo.

**Lo que se comprobó.** Se subió un archivo de 4 filas:

| Nombre de Compañía | Código Identificador | Resultado |
|---|---|---|
| Laboratorio Demo Andino | OTRO-COD | «El valor 'Laboratorio Demo Andino' del campo 'nombreCompania' ya existe en el sistema» |
| Laboratorio Demo Código Repetido | DEMO-AND | **Válida** |
| (vacío) | X1 | «El campo 'nombreCompania' es obligatorio» |
| (160 letras) | (vacío) | «'nombreCompania' no puede superar los 150 caracteres» |

Al hacer clic en **Guardar válidos (1)** apareció «Se guardaron 0 registros correctamente» y la pantalla volvió a la búsqueda de laboratorios. **Cuidado:** la vista previa no revisa si el **código identificador** ya existe, así que esa fila parecía válida; al guardar, SOFIA la omitió en silencio porque `DEMO-AND` ya lo usa el «Laboratorio Demo Andino», y el aviso de éxito dijo 0. Lee siempre el número de registros guardados y compáralo con las filas válidas. No se creó ningún laboratorio con esta prueba.

### 9.4 Ingredientes activos

**Columnas de la plantilla:**

| Columna en el Excel | Qué escribir | Obligatoria | Límite |
|---|---|---|---|
| `DenominacionDci` | Denominación DCI | Sí | 255 caracteres |
| `CodigoAtc` | Código ATC | Sí | 15 caracteres |

La vista previa muestra «Denominación DCI», «Código ATC», la columna de errores y «Estado». Reglas por fila: ambos campos obligatorios, máximos de 255 y 15 caracteres, y **código ATC** que ya existe o que se repite en el archivo.

**Lo que se comprobó.** Se subió un archivo de 8 filas:

| Denominación DCI | Código ATC | Resultado |
|---|---|---|
| Ibuprofeno | M01AE01 | Válida |
| Omeprazol | A02BC01 | Válida |
| (vacío) | A01 | «El campo 'denominacionDci' es obligatorio» |
| Metformina | (vacío) | «El campo 'codigoAtc' es obligatorio» |
| Diazepam | N02BE01 | «El valor 'N02BE01' del campo 'codigoAtc' ya existe en el sistema» (el Paracetamol ya usa ese código) |
| Paracetamol | ZZ99 | **Válida** |
| Salbutamol | R03AC02 | «El valor 'R03AC02' del campo 'codigoAtc' está duplicado en el archivo» |
| Salbutamol sulfato | R03AC02 | Igual que la anterior |

La vista previa dijo «8 filas totales», «3 filas válidas» y «5 filas con error». **Guardar válidos (3)** mostró «Se guardaron 2 registros correctamente» y llevó a la búsqueda de ingredientes: se crearon «Ibuprofeno» y «Omeprazol», y la fila «Paracetamol» se **omitió en silencio** porque esa DCI ya existe.

Dos cosas que debes saber de esta carga, porque la pantalla de un ingrediente se comporta distinto:

- La vista previa revisa si el **código ATC** se repite, pero crear un ingrediente a mano **sí permite** repetir el ATC. Un archivo con dos ingredientes que comparten ATC queda bloqueado en la vista previa aunque a mano sí los podrías crear.
- La vista previa **no** revisa si la **denominación DCI** ya existe (que es lo que realmente no se puede repetir). Esas filas parecen válidas y se omiten al guardar; compara siempre el número guardado con el de filas válidas.

## 10. Lo que ve un cajero frente a un administrador

Comprobado iniciando sesión con el cajero de prueba del capítulo 3 (`carlos.cajero`, rol «Cajero», que solo tiene `DIGEMID › Leer` entre los permisos de este capítulo):

| Pantalla o elemento | Administrador (rol «Admin») | Cajero (rol «Cajero» del capítulo 3) |
|---|---|---|
| Menú «Unidades de Medida», «Laboratorios» e «Ingredientes Activos» (grupo CATÁLOGOS) | Los ve | No los ve: su menú es Dashboard, Empleados, Medicamentos, Aseguradoras, Lotes, Stock por Sucursal, Sesiones de Caja, Ventas, Pacientes, Profesionales de Salud, Recetas Médicas y Catálogo DIGEMID |
| Menú «Catálogo DIGEMID» (grupo DIGEMID) | Lo ve | Lo ve (tiene `DIGEMID › Leer`) |
| Búsqueda del catálogo DIGEMID | Filtros, tabla y **Exportar** | Igual: filtros, tabla y **Exportar** (comprobado: ve los 5 productos de la sección 8.5) |
| Botón **Carga Masiva** del catálogo DIGEMID | Lo ve | **No lo ve** (necesita `DIGEMID › Crear`) |
| Botones **Nuevo**, **Editar**, **Eliminar** en las pantallas de unidades, laboratorios e ingredientes | Los ve según el permiso (`Crear`, `Actualizar`, `Eliminar` del módulo) | Solo con el permiso correspondiente del módulo |
| Escribir a mano la dirección de una pantalla sin permiso | — | El sistema lo devuelve al Inicio con «No tienes permisos para acceder a esta pantalla.» |

(No verificado en la aplicación: la redirección del último renglón con los catálogos de este capítulo; el navegador impidió probar la navegación directa con la sesión del cajero. Viene del inventario 19c §0. Tampoco se abrió el detalle de un producto DIGEMID con el cajero.)

El cajero no necesita estos catálogos para vender: usa los medicamentos que el administrador registró con ellos (capítulo 5). Las listas de unidades, laboratorios e ingredientes también aparecen dentro de las pantallas de Medicamentos y Formulaciones clínicas.

## 11. Diferencias con la documentación técnica

El inventario `19c-catalogos-1.md` y la referencia `20-carga-masiva-y-exportacion.md` coincidieron con la aplicación en todo lo comprobado. Se anotan estos matices y confirmaciones:

| # | Documento y apartado | Dice | La aplicación muestra |
|---|---|---|---|
| 1 | 20 §3.3 y 01b RN-ING-012, RN-ING-013 | La vista previa de ingredientes revisa el ATC repetido y no la DCI repetida; al guardar se omiten las filas con DCI existente | Confirmado: `Diazepam` con un ATC existente salió con error; `Paracetamol` con otro ATC salió válida y al guardar se omitió: «Se guardaron 2 registros» de 3 filas válidas |
| 2 | 20 §3.5 y 01b RN-LAB-015 | La vista previa no revisa el código identificador repetido | Confirmado: la fila salió válida y el guardado informó «Se guardaron 0 registros correctamente» (un mensaje de éxito con cero registros) |
| 3 | 20 §3.2 y 01b RN-DIG-005 | La vista previa no valida longitudes y el guardado sí | Confirmado: un registro sanitario de 60 caracteres salió válido y el guardado completo fue rechazado con «Algunas filas tienen datos inválidos. Vuelve a subir el archivo y revisa la vista previa.» |
| 4 | 20 §3.12 y 3.2 | Las filas repetidas en el archivo quedan con error | Confirmado, incluida la **primera** aparición (`LT` x2 en unidades; `DEMO-0008` y `demo-0008` en DIGEMID) |
| 5 | 06-glosario («Catálogo DIGEMID») y 01b nota de RN-DIG | El catálogo se siembra con el script `0004_Seed_Digemid_Catalogo.sql`; 01b dice «No verificado» si esos productos son visibles para cada empresa | En la empresa de prueba, recién registrada, la búsqueda del catálogo mostró «Resultados (0)»: no se ve ningún producto sembrado |
| 6 | 19c §5.1 y 01b RN-DIG-009 | Los filtros buscan «contiene» | Además, el filtro «Nombre del Producto» **distingue los acentos** (`cápsula` no encuentra «CAPSULA»), aunque no distingue mayúsculas. Ningún documento lo menciona |
| 7 | 19c §5 (abrir `/digemid-catalogo/nueva`) | «Error al cargar el producto DIGEMID» con el formulario vacío | Formulario vacío y deshabilitado; y, al recargar la página directamente en esa dirección, el aviso rojo mostró la clave sin traducir `digemidCatalogo.detail.load-error` (el aviso salió antes de que cargaran los textos del catálogo) |
| 8 | 19c §0 (crear con éxito navega a la búsqueda) | Navega a la búsqueda | Coincide; con los filtros vacíos y sin tabla, como en Sucursales (capítulo 2) |

## 12. Pendientes y «No verificado»

En curso (sin commitear): ninguno. Los dos repositorios estaban sin cambios locales al empezar (`sofia-apis` `43f5529`, `sofia-web-spa` `eeffbb6`).

Pendiente para el capítulo 5: probar eliminar una **unidad** en uso (medicamento o jerarquía), un **laboratorio** con medicamentos y un **ingrediente** incluido en una formulación clínica.

No verificado en la aplicación (cada caso, con su motivo):

| # | Qué | Motivo |
|---|---|---|
| 1 | Contenido y descarga de los archivos de **Exportar** (unidades, laboratorios, ingredientes y catálogo DIGEMID) y de las cuatro plantillas | El navegador integrado no muestra los archivos descargados. Se comprobó que la petición se envía, que el servidor responde 200 y que no hay avisos de error. Las columnas vienen de 19c y del documento 20 §4.2, §4.6, §4.8 y §4.21 |
| 2 | El gesto de arrastrar o hacer clic para elegir el archivo en la carga masiva | El navegador integrado no tiene herramienta para elegir un archivo. Se entregó a la pantalla, desde el propio navegador, un `.xlsx` de prueba construido para este manual (y un archivo de texto para probar el rechazo) |
| 3 | Errores de archivo de la carga masiva: más de 5 MB, más de 5000 filas, descompresión excesiva, errores del servidor al procesar o al guardar, la espera «Procesando archivo...» | No se prepararon archivos de ese tipo; vienen de 19a §6.5 y §8.9 y del documento 20 §2.6 |
| 4 | Eliminar con el registro en uso (unidad, laboratorio, ingrediente) | Todavía no hay medicamentos, jerarquías ni formulaciones. Textos de 19c §1.2, §3.2, §4.2 y §6. Se probará en el capítulo 5 |
| 5 | Avisos genéricos de error: «Error al buscar …», «Error al cargar …», «Error al eliminar …», «Error al guardar …» (salvo el de solo espacios), los tres avisos de base de datos | Son errores del servidor o de la red; no se pueden provocar desde la pantalla |
| 6 | Que **Cancelar** en el formulario de crear vuelve a la búsqueda sin guardar (laboratorios, ingredientes y unidades); que cada campo corta la escritura en su máximo (solo se tecleó el código de unidad, que se cortó en 10; los otros máximos —50, 150, 255, 15— se leyeron del atributo del campo) | Viene de 19c §1.2, §3.2 y §4.2 |
| 7 | Paginación real (más de una página) y el paginador con 25 o 50 filas | Con 9 unidades, 2 laboratorios, 6 ingredientes y 5 productos hay una sola página |
| 8 | Los códigos ATC de los ingredientes (`N02BE01`, `J01CA04`, `N03AE01`, `N02AX02`, `M01AE01`, `A02BC01`) | Se escribieron de conocimiento del autor; SOFIA no los valida y no se contrastaron con una fuente oficial en esta sesión. Confírmalos antes de usarlos en producción |
| 9 | Qué significa el dato «Fracción» del catálogo DIGEMID (en el ejemplo se usó `10`) | El glosario lo define como la fracción de la presentación en que puede venderse el producto, sin fuente oficial; SOFIA no lo usa para vender |
| 10 | Redirección por falta de permiso en estas pantallas, y el detalle y la exportación del catálogo DIGEMID con el cajero | El navegador impidió probar la navegación directa con la sesión del cajero. Viene de 19c §0 |
| 11 | Productos de catálogo DIGEMID con la IA de recetas | Se recorre en el capítulo de recetas |

Hallazgos de la aplicación que conviene conocer (no son errores del manual):

- Un mensaje de éxito con cero registros: la carga masiva de laboratorios con una sola fila «válida» que repetía un código existente terminó con «Se guardaron 0 registros correctamente».
- La carga masiva de ingredientes valida el ATC repetido (que a mano sí se permite) y no la DCI repetida (que es la clave real).
- La vista previa del catálogo DIGEMID no revisa longitudes y el guardado rechaza todo el archivo si una fila es demasiado larga.
- La búsqueda por nombre del catálogo DIGEMID distingue los acentos.
- La clave sin traducir en el aviso al recargar `/digemid-catalogo/nueva`.
- El texto del aviso de «unidad en uso» no menciona las formulaciones clínicas, que también bloquean (19c §1.2).
- La pantalla acepta valores de solo espacios y el servidor los rechaza con el aviso genérico de guardado.

## 13. Tabla de cobertura

Cada elemento de las fichas de `19c-catalogos-1.md` de Unidades de medida (§1), Laboratorios (§3), Ingredientes activos (§4) y Catálogo DIGEMID (§5), y de su §0 y §6 en lo que les toca, con el apartado de este capítulo. «V» significa recorrido en la aplicación.

### 13.1 Unidades de medida — búsqueda `/unidades-medida` (19c §1.1)

| Elemento | Apartado |
|---|---|
| Título «Unidades de Medida», migas «Inicio / Catálogos / Unidades de Medida» (sin botón volver), tarjeta «Búsqueda de Unidades de Medida» y subtítulo | 4, 5.2 (V) |
| Filtro «Código» (lupa, Enter, «contiene») | 5.2, 5.4 (V) |
| Filtro «Descripción» (lupa, Enter, «contiene») | 5.2, 5.4 (V) |
| Botones **Carga Masiva** y **Nuevo** (permiso `UnidadesMedida:Crear`) | 4, 5.3, 9.2 (V) |
| Botones **Limpiar** y **Buscar** | 5.4 (V) |
| Tabla: «Código» (enlace) y «Descripción» | 5.2 (V) |
| «Resultados (N)», fila «Resultados», «Sin resultados», paginador 10/25/50 | 4, 5.4 (V; varias páginas: No verificado) |
| Botón **Exportar** y archivo `unidades-medida.xlsx` con 2 columnas | 5.4 (petición V; contenido del archivo: No verificado) |
| «Error al buscar unidades de medida» | 5.4 (No verificado: no se puede provocar) |
| Restauración de filtros al volver del detalle | 4, 5.6 (V) |
| Permiso de ruta `UnidadesMedida:Leer` y menú CATÁLOGOS | 2, 10 |

### 13.2 Unidades de medida — detalle `/unidades-medida/nueva` y `/:id` (19c §1.2)

| Elemento | Apartado |
|---|---|
| Cabecera (título, migas, flecha de volver) y modos crear, ver y editar | 4, 5.3, 5.5 (V) |
| Tarjeta «Unidades de Medida» + «Datos del registro» + **Editar** (permiso `UnidadesMedida:Actualizar`) | 4, 5.5 (V) |
| Campo «Código» (obligatorio, máximo 10, único) | 5.3 (V) |
| Campo «Descripción» (obligatorio, máximo 50) | 5.3 (V) |
| Botones **Cancelar** y **Guardar** | 5.3, 5.5 (V) |
| Botón **Eliminar** (permiso `UnidadesMedida:Eliminar`, solo en modo editar de una unidad existente) | 5.6 (V) |
| Cuadro «Confirmar eliminación» («¿Estás seguro de que deseas eliminar esta unidad de medida?», **Cancelar**, **Confirmar**) | 5.6 (V) |
| «Unidad de medida guardada correctamente» | 5.3, 5.5 (V) |
| «Ya existe una unidad de medida con ese código.» | 5.3, 5.5 (V) |
| Códigos `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` y sus tres textos | 5.3 (No verificado) |
| «Error al guardar la unidad de medida» | 5.3 (V con solo espacios) |
| «Unidad de medida eliminada correctamente» | 5.6 (V) |
| «No se puede eliminar la unidad de medida porque está asociada a medicamentos o jerarquías de unidades.» | 5.6 (No verificado: sin usos todavía; capítulo 5) |
| «Error al eliminar la unidad de medida» | 5.6 (No verificado) |
| «Error al cargar la unidad de medida» | 5.5 (No verificado) |
| Spinner mientras carga y valores de solo espacios | 4, 5.3 (V la parte de espacios) |

### 13.3 Unidades de medida — carga masiva `/unidades-medida/carga-masiva` (19c §1.3)

| Elemento | Apartado |
|---|---|
| Título «Carga Masiva», migas, flecha de volver, **Descargar plantilla**, zona de arrastre | 9.1, 8.5 (V) |
| Plantilla `plantilla-unidades-medida.xlsx`, columnas `Codigo` y `Descripcion` | 9.2 (petición V; archivo: No verificado) |
| Vista previa: contadores, columnas «Código» y «Descripción», columna de errores, «Estado» | 9.2 (V) |
| Errores de fila `required`, `max-length`, `duplicate`, `duplicate-in-file` | 9.2 (V) |
| **Guardar válidos (N)** y «Se guardaron N registros correctamente» | 9.2 (V) |
| Errores de archivo y de guardado | 8.5 (V el de extensión; el resto No verificado) |

### 13.4 Laboratorios — búsqueda `/laboratorios` (19c §3.1)

| Elemento | Apartado |
|---|---|
| Título «Laboratorios», migas, tarjeta «Búsqueda de Laboratorios» y subtítulo | 4, 6.2 (V) |
| Filtros «Nombre de Compañía» y «Código Identificador» | 6.2, 6.4 (V) |
| Botones **Carga Masiva**, **Nuevo**, **Limpiar**, **Buscar** | 4, 6.3, 9.3 (V) |
| Tabla: «Nombre de Compañía» (enlace) y «Código Identificador» («—» si no tiene) | 6.2, 6.4 (V) |
| «Resultados (N)», «Resultados», «Sin resultados», paginador | 4, 6.4 (V; varias páginas: No verificado) |
| **Exportar** y `laboratorios.xlsx` con 2 columnas | 6.4 (petición V; archivo: No verificado) |
| «Error al buscar laboratorios» | 6.4 (No verificado) |
| Permiso `Laboratorios:Leer` | 2 |

### 13.5 Laboratorios — detalle `/laboratorios/nueva` y `/:id` (19c §3.2)

| Elemento | Apartado |
|---|---|
| Cabecera, modos crear, ver y editar, tarjeta «Datos del registro», **Editar** (`Laboratorios:Actualizar`) | 4, 6.3, 6.5 (V) |
| Campo «Nombre de Compañía» (obligatorio, máximo 150, único) | 6.3 (V) |
| Campo «Código Identificador» (opcional, máximo 50, único si se informa; vacío = sin código) | 6.3, 6.5 (V) |
| **Cancelar**, **Guardar**, **Eliminar** (`Laboratorios:Eliminar`) y cuadro «Confirmar eliminación» («¿Estás seguro de que deseas eliminar este laboratorio?») | 6.5 (V) |
| «Laboratorio guardado correctamente» | 6.3, 6.5 (V) |
| «Ya existe un laboratorio con ese nombre de compañía.» y «Ya existe un laboratorio con ese código identificador.» | 6.3 (V) |
| Códigos `Db.*` | 6.3 (No verificado) |
| «Error al guardar el laboratorio» | 6.3 (V con solo espacios) |
| «Laboratorio eliminado correctamente» | 6.5 (V) |
| «No se puede eliminar el laboratorio porque tiene medicamentos asociados.» | 6.5 (No verificado; capítulo 5) |
| «Error al eliminar el laboratorio» y «Error al cargar el laboratorio» | 6.5 (No verificado) |

### 13.6 Laboratorios — carga masiva `/laboratorios/carga-masiva` (19c §3.3)

| Elemento | Apartado |
|---|---|
| Pantalla, plantilla `plantilla-laboratorios.xlsx`, columnas `NombreCompania` y `CodigoIdentificador` (no obligatoria) | 9.1, 9.3 (V la petición) |
| Vista previa: «Nombre de Compañía», «Código Identificador», errores, «Estado» | 9.3 (V) |
| Errores `required`, `max-length`, `duplicate` | 9.3 (V); `duplicate-in-file` (misma regla que 9.2, V en unidades) |
| Código identificador repetido pasa la vista previa y se omite al guardar | 9.3 (V) |
| **Guardar válidos (N)** y «Se guardaron N registros correctamente» | 9.3 (V) |

### 13.7 Ingredientes activos — búsqueda `/ingredientes-activos` (19c §4.1)

| Elemento | Apartado |
|---|---|
| Título «Ingredientes Activos», migas, tarjeta «Búsqueda de Ingredientes Activos» y subtítulo | 4, 7.2 (V) |
| Filtros «Denominación DCI» y «Código ATC» | 7.2, 7.4 (V) |
| Botones **Carga Masiva**, **Nuevo**, **Limpiar**, **Buscar** | 4, 7.3, 9.4 (V) |
| Tabla: «Denominación DCI» (enlace) y «Código ATC» | 7.2, 7.4 (V) |
| «Resultados (N)», «Resultados», «Sin resultados», paginador | 4, 7.4 (V; varias páginas: No verificado) |
| **Exportar** y `ingredientes-activos.xlsx` con 2 columnas | 7.4 (petición V; archivo: No verificado) |
| «Error al buscar ingredientes activos» | 7.4 (No verificado) |
| Permiso `IngredientesActivos:Leer` | 2 |

### 13.8 Ingredientes activos — detalle `/ingredientes-activos/nueva` y `/:id` (19c §4.2)

| Elemento | Apartado |
|---|---|
| Cabecera, modos crear, ver y editar, **Editar** (`IngredientesActivos:Actualizar`) | 4, 7.3, 7.5 (V) |
| Campo «Denominación DCI» (obligatorio, máximo 255, único) | 7.3 (V) |
| Campo «Código ATC» (obligatorio, máximo 15, sin validar estructura, repetible) | 7.3, 7.5 (V) |
| **Cancelar**, **Guardar**, **Eliminar** (`IngredientesActivos:Eliminar`) y cuadro «Confirmar eliminación» («¿Estás seguro de que deseas eliminar este ingrediente activo?») | 7.5 (V) |
| «Ingrediente activo guardado correctamente» | 7.3, 7.5 (V) |
| «Ya existe un ingrediente activo con esa denominación DCI.» | 7.3 (V) |
| Códigos `Db.*` | 7.3 (No verificado) |
| «Error al guardar el ingrediente activo» | 7.3 (V con solo espacios) |
| «Ingrediente activo eliminado correctamente» | 7.5 (V) |
| «No puedes eliminar este ingrediente activo porque está incluido en formulaciones clínicas.» | 7.5 (No verificado; capítulo 5) |
| «Error al eliminar el ingrediente activo» y «Error al cargar el ingrediente activo» | 7.5 (No verificado) |

### 13.9 Ingredientes activos — carga masiva `/ingredientes-activos/carga-masiva` (19c §4.3)

| Elemento | Apartado |
|---|---|
| Pantalla, plantilla `plantilla-ingredientes-activos.xlsx`, columnas `DenominacionDci` y `CodigoAtc` (obligatorias) | 9.1, 9.4 (V la petición) |
| Vista previa: «Denominación DCI», «Código ATC», errores, «Estado» | 9.4 (V) |
| Errores `required`, `duplicate` y `duplicate-in-file` sobre el código ATC | 9.4 (V) |
| `max-length` | 9.4 (misma regla que 9.2, V en unidades) |
| DCI repetida pasa la vista previa y se omite al guardar | 9.4 (V) |
| **Guardar válidos (N)** y «Se guardaron N registros correctamente» | 9.4 (V) |

### 13.10 Catálogo DIGEMID — búsqueda `/digemid-catalogo` (19c §5.1)

| Elemento | Apartado |
|---|---|
| Título «Catálogo DIGEMID», migas «Inicio / DIGEMID / Catálogo DIGEMID», tarjeta «Búsqueda en Catálogo DIGEMID» y subtítulo | 8.3 (V) |
| Filtros «Código de Producto» y «Nombre del Producto» | 8.3 (V) |
| Botón **Carga Masiva** (`DIGEMID:Crear`), único del encabezado; no hay **Nuevo** | 4, 8.2, 8.3 (V) |
| Botones **Limpiar** y **Buscar** | 8.3 (V) |
| Tabla: «Código» (enlace), «Nombre del Producto», «Forma Farmacéutica» («—» si no tiene), «Estado» | 8.3 (V) |
| «Resultados (N)», «Resultados», «Sin resultados», paginador | 8.3 (V; varias páginas: No verificado) |
| **Exportar** y `digemid-catalogo.xlsx` con 4 columnas | 8.3 (petición V; archivo: No verificado) |
| «Error al buscar en el catálogo DIGEMID» | 8.3 (No verificado) |
| Qué se puede y qué no (crear, editar y eliminar no existen en pantalla) | 8.2 (V) |
| Permiso `DIGEMID:Leer` y menú DIGEMID | 2, 10 (V con el cajero) |

### 13.11 Catálogo DIGEMID — detalle `/digemid-catalogo/:id` (19c §5.2)

| Elemento | Apartado |
|---|---|
| Cabecera (título, migas, flecha de volver), tarjeta sin encabezado ni botones | 8.4 (V) |
| Ocho campos de solo lectura: Código de Producto, Nombre del Producto, Concentración, Forma Farmacéutica, Fracción, Registro Sanitario, Titular, Estado | 8.4 (V) |
| Valores nulos se ven vacíos | 8.4 (V) |
| «Error al cargar el producto DIGEMID» | 8.4, 11 (V parcial: se vio la clave sin traducir) |
| Ruta `/digemid-catalogo/nueva` | 8.2 (V) |

### 13.12 Catálogo DIGEMID — carga masiva `/digemid-catalogo/carga-masiva` (19c §5.3)

| Elemento | Apartado |
|---|---|
| Pantalla, migas, **Descargar plantilla**, zona de arrastre | 8.5 (V) |
| Plantilla `plantilla-digemid-catalogo.xlsx` y sus 8 columnas | 8.5 (petición V; archivo: No verificado) |
| Columnas de la vista previa (8 + errores + «Estado»), con las dos columnas «Estado» | 8.5 (V) |
| Errores `required`, `duplicate`, `duplicate-in-file` | 8.5 (V) |
| Vista previa sin comprobar longitudes; el guardado rechaza todo | 8.5 (V) |
| **Guardar válidos (N)** y «Se guardaron N registros correctamente» | 8.5 (V) |
| Errores de archivo y de guardado (tabla) | 8.5 (V el de extensión y el de filas inválidas; el resto No verificado) |

### 13.13 Comportamientos comunes (19c §0) y eliminar (19c §6)

| Elemento | Apartado |
|---|---|
| Permiso de ruta por módulo (`UnidadesMedida`, `Laboratorios`, `IngredientesActivos`, `DIGEMID`) y acción `Leer`/`Crear` | 2, 10 |
| Menú: grupo CATÁLOGOS (tres pantallas de este capítulo) y grupo DIGEMID | 2, 10 (V) |
| Directiva de permiso (botones según `Crear`, `Actualizar`, `Eliminar`) | 2, 4, 10 |
| Avisos (botón **Cerrar**, duración) | 2 |
| Estado de la búsqueda, **Buscar** conserva la página | 4 (restauración V; conservar la página: No verificado, solo hay una página) |
| Modo ver / editar y diferencias entre crear y editar al guardar | 4, 5.3, 5.5 (V) |
| «Este campo es obligatorio» y máximos por `maxlength` | 4, 5.3, 6.3, 7.3 (V) |
| Diálogo de eliminar (400 px, **Cancelar**/**Confirmar**) | 4, 5.6 (V) |
| Mapa de errores de guardado (`Db.*`, mensaje genérico) | 5.3 (No verificado) |
| Exportación con encabezados traducidos y nombre fijo | 5.4, 6.4, 7.4, 8.3 |
| Carga masiva común (plantilla, vista previa, guardado) | 8.5, 9 |
| Mapa de eliminar (unidades, laboratorios, ingredientes; DIGEMID sin eliminar) | 5.6, 6.5, 7.5, 8.2 |
| Jerarquías de unidades (19c §2) | No aplica: es el capítulo 5 |

## 14. Fuentes

Documentos consultados:

- `docs/_inventario/REGLAS-AGENTES.md` y `docs/_inventario/MANUAL-AGENTES.md`.
- `docs/ola-1-fundamentos/19-inventario-de-pantallas/19c-catalogos-1.md` (§0, §1, §3, §4, §5, §6 y §8).
- `docs/ola-1-fundamentos/20-carga-masiva-y-exportacion.md` (§2, §3.2, §3.3, §3.5, §3.12, §4.2, §4.6, §4.8, §4.21).
- `docs/ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md` (reglas UDM, LAB, ING, DIG, versión 1.1) y `01c1-inventario-y-recetas.md` (RN-REC-023).
- `docs/ola-1-fundamentos/06-glosario.md` («Unidad base», «Unidad de medida», «Laboratorio», «DCI», «Código ATC», «Catálogo DIGEMID»).
- `docs/ola-2-funcional/manual-usuario/capitulo-02-empresa-sucursales-series.md` (modelo de formato).

Recorridos hechos en la aplicación (`http://localhost:4200`, sesión del administrador de la empresa de prueba, salvo el último):

- Unidades de medida: búsqueda vacía; **Nuevo** con formulario vacío, código de 16 letras, código de solo espacios, `UND` duplicado (también en minúsculas); creación de `UND`, `CAJ`, `BLI`, `FRA`, `TAB`, `MG`, `ML` y `PRU`; filtros por código, descripción, ambos y sin resultados; **Limpiar**; **Exportar**; detalle, **Editar**, **Cancelar**, **Guardar**, código repetido al editar; **Eliminar** con **Cancelar** y **Confirmar**; volver a crear `PRU` tras eliminarla; creación y eliminación de una unidad temporal para ver el estado de la búsqueda; carga masiva (plantilla, vista previa de 7 filas, guardado de 2).
- Laboratorios: **Nuevo** vacío, solo espacios, nombre repetido en minúsculas, código repetido; creación de «Laboratorio Demo Andino» y «Laboratorio Demo Pacífico» y de uno de prueba; filtros, **Exportar**; detalle, edición (borrar el código, cambiar el nombre), nombre vacío, **Cancelar**; **Eliminar**; carga masiva (plantilla, vista previa de 4 filas, guardado con resultado 0).
- Ingredientes activos: **Nuevo** vacío, solo espacios, DCI repetida en minúsculas; creación de Paracetamol, Amoxicilina, Clonazepam, Tramadol y uno de prueba con ATC repetido; filtros (DCI, ATC, ambos); **Exportar**; edición del ATC a `XXX`; **Eliminar**; carga masiva (vista previa de 8 filas, guardado de 3 válidas con 2 guardadas).
- Catálogo DIGEMID: búsqueda vacía de la empresa nueva; carga masiva (archivo de 9 filas, guardado de 5; archivo de 2 filas con código existente y texto largo, guardado rechazado; archivo de texto rechazado); filtros por código, nombre y ambos (incluida la prueba de acentos); **Exportar**; detalle de `DEMO-0001`; dirección `/digemid-catalogo/nueva`.
- Cajero (`carlos.cajero`): menú, búsqueda del catálogo DIGEMID, ausencia de **Carga Masiva** y presencia de **Exportar**. Después se volvió a iniciar sesión como administrador.
