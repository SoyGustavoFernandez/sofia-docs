# Capítulo 7 — Pacientes, profesionales de salud y recetas

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-31 |
| **Backend** | `sofia-apis`, rama `main`, commit `34089ec` (más nuevo que la línea base `43f5529`; `git status --short` sin cambios locales al empezar) |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `9ad15f0` (más nuevo que la línea base `eeffbb6`; `git status --short` sin cambios locales al empezar) |
| **Método** | Recorrido en la aplicación local más inventario de pantallas |

## 1. Introducción

En este capítulo registras **a quién le vendes y quién le recetó**. Vas a:

- Registrar **pacientes** (las personas a las que atiendes), con su documento de identidad.
- Registrar **profesionales de salud** (los médicos que firman las recetas).
- Registrar **recetas médicas**: quién es el paciente, quién es el médico, cuándo se expidió la receta y cuántas veces más se puede repetir.
- **Buscar, ver, editar y eliminar** cada uno de ellos, **exportarlos** a Excel y, en pacientes y profesionales, **cargarlos de golpe** desde un Excel.
- Usar el **Lector de Recetas IA** (escáner), que lee la foto de una receta y te dice qué medicamentos reconoció y si tienes stock.
- Ver qué cambia para un cajero.

Es para el **administrador** (rol «Admin»). La sección 13 explica qué ve el cajero.

Al terminar, la empresa de prueba «Botica Manual Demo S.A.C.» tendrá **pacientes, un profesional de salud y tres recetas** (una de receta simple, una de receta retenida y una de estupefaciente) listas para los capítulos de caja y venta y de devoluciones y anulación. Todos los nombres, documentos y números son inventados.

**Dos avisos importantes antes de empezar**

- En SOFIA una receta registrada es **solo la cabecera**: paciente, médico, fecha de expedición, repeticiones máximas, número de receta e indicaciones. **No tiene líneas de medicamentos ni cantidades, y tampoco tiene fecha de vigencia** (comprobado en la pantalla, sección 10). El medicamento que cubre la receta lo escribes en «Indicaciones de Uso» como texto libre; el sistema no lo valida contra el producto que vendes.
- El Lector de Recetas IA **solo muestra un resultado en pantalla**: no crea la receta ni la guarda (sección 12).

## 2. Antes de empezar

- Haz los capítulos 1 a 6. Necesitas la sesión iniciada como administrador (`ana.manual`) y, para vender más adelante, los medicamentos y lotes de los capítulos 5 y 6.
- Permisos que usa este capítulo. Los nombres son los que ves en la pantalla de permisos de un rol (Administración › Roles & Permisos): primero el módulo y luego la acción.

| Tarea | Módulo | Acción |
|---|---|---|
| Ver, buscar y exportar pacientes | Pacientes | `Leer` |
| Registrar un paciente y cargar pacientes desde Excel | Pacientes | `Crear` (además de `Leer`) |
| Editar un paciente | Pacientes | `Actualizar` |
| Eliminar un paciente | Pacientes | `Eliminar` |
| Ver, buscar y exportar profesionales de salud | Profesionales de Salud | `Leer` |
| Registrar profesionales y cargarlos desde Excel | Profesionales de Salud | `Crear` (además de `Leer`) |
| Editar un profesional de salud | Profesionales de Salud | `Actualizar` |
| Eliminar un profesional de salud | Profesionales de Salud | `Eliminar` |
| Ver, buscar y exportar recetas | Recetas Médicas | `Leer` |
| Registrar una receta | Recetas Médicas | `Crear` (además de `Leer`) |
| Editar una receta | Recetas Médicas | `Actualizar` |
| Eliminar una receta | Recetas Médicas | `Eliminar` |
| Usar el Lector de Recetas IA (botón **Escanear receta**) | Recetas Médicas | `Analizar con IA` (**no** basta con `Crear`) |

- Para elegir al paciente o al médico en una receta, la pantalla consulta las listas de pacientes y de profesionales de salud. Para que esas listas se llenen necesitas además `Pacientes › Leer` y `Profesionales de Salud › Leer`. El administrador y el cajero del manual los tienen. Qué ocurre si falta uno: No verificado en la aplicación (no se probó quitar permisos a un rol).
- En el menú lateral, las tres pantallas están en el grupo **PACIENTES & ATENCIÓN**: «Pacientes», «Profesionales de Salud» y «Recetas Médicas». Una opción solo aparece si tu rol tiene `Leer` de ese módulo; si no tienes ninguna, el grupo completo no aparece.
- Si escribes la dirección de una pantalla a la que no tienes permiso, la aplicación te lleva al **Dashboard** y muestra el aviso «No tienes permisos para acceder a esta pantalla.» (comprobado con el cajero, sección 13).
- Los avisos de la aplicación aparecen abajo al centro con un botón **Cerrar** y desaparecen solos a los pocos segundos. Los textos de este capítulo están copiados tal como se vieron en pantalla.
- Las fechas se muestran siempre como día/mes/año (`dd/MM/aaaa`). En los selectores de fecha puedes escribir la fecha a mano en ese formato (por ejemplo `15/03/1985`) o usar el icono de calendario.
- Los campos obligatorios llevan un asterisco junto a la etiqueta. **Excepción:** en la receta, «Paciente» y «Médico» son obligatorios aunque no lleven asterisco.

## 3. Cómo se relacionan estas piezas (en lenguaje simple)

| Pieza | Qué es | Para qué la usa SOFIA |
|---|---|---|
| **Paciente** | Una persona a la que atiendes, identificada por su documento (DNI, carné de extranjería, RUC o pasaporte) | Saber a quién pertenece una receta y a quién le vendes |
| **Profesional de salud** | El médico (o prescriptor) que firma la receta. SOFIA guarda solo tres datos: **número de registro**, **nombre del prescriptor** y **dirección de la clínica** | Dejar constancia de quién recetó. No hay campos de especialidad ni de colegiatura aparte: el «Número de Registro» es su identificador |
| **Receta médica** | La **cabecera** de la receta: paciente, médico, fecha de expedición, repeticiones máximas, número de receta e indicaciones | Es lo que eliges en la venta cuando el medicamento exige receta. El sistema **no vence recetas por fecha**: no existe un campo de vigencia |
| **Repeticiones máximas** | Las veces **adicionales** que se puede volver a dispensar la receta | En una receta simple se puede dispensar 1 vez más las repeticiones; en una receta retenida o de estupefaciente, 1 sola vez. La cuenta se controla **al vender** (capítulo 8), no al registrar la receta (regla `RN-REC-031`; No verificado en la aplicación en este capítulo, porque aún no hay ventas) |
| **Número de receta** | El número que trae la receta de papel (hasta 30 caracteres) | Es **obligatorio al vender** receta retenida o estupefacientes; en las demás es opcional. Se puede repetir entre recetas distintas |

Cómo se exige la receta al vender (lo verás en el capítulo 8):

- **Venta libre (OTC):** nunca pide receta.
- **Receta retenida y estupefaciente:** siempre piden una receta registrada y con número.
- **Receta simple:** depende de una política de la empresa llamada «exigir registro de receta». Si está **encendida**, también pide receta registrada. Si está **apagada** (así queda al registrar una empresa), el cajero puede vender sin receta registrada si marca «Receta verificada» al cobrar, y la venta queda a su nombre. **Esa política no se cambia desde ninguna pantalla** (solo en la base de datos), por eso este capítulo no la modifica.

Fuente de las reglas: `01b-catalogos.md` (`RN-PAC-001` a `RN-PAC-017`, `RN-PRO-001` a `RN-PRO-012`), `01c1-inventario-y-recetas.md` (`RN-REC-001` a `RN-REC-031`), `01c2-ventas-y-comprobantes.md` (`RN-VEN-028`, `RN-VEN-029`, `RN-VEN-034`) y `06-glosario.md` («Receta verificada»).

## 4. Los datos que se crean en este capítulo

Para los capítulos 8 (caja y venta) y 9 (devoluciones y anulación) se crean **pacientes, profesionales de salud y tres recetas**. Todos los nombres y documentos son inventados.

**Pacientes** (los dos primeros se crean uno a uno; los otros dos, por carga masiva)

| Documento | Tipo | Nombre y Apellidos | Fecha de nacimiento | Contacto Primario |
|---|---|---|---|---|
| `70000001` | DNI | Luis Paciente Demo | 15/03/1985 | Tel. 999000112 |
| `70000002` | DNI | María Paciente Demo | 20/07/1990 | `maria.paciente@sofia-manual.test` |
| `70000003` | DNI | Pedro Carga Demo | 10/10/1975 | Tel. 999000333 |
| `CE0000004` | Carné de extranjería | Ana Extranjera Demo | 11/11/1982 | (sin contacto) |

**Profesionales de salud**

| Número de Registro | Nombre del Prescriptor | Dirección de Clínica |
|---|---|---|
| `CMP-DEMO-0001` | Dr. Raúl Médico Demo | Clínica Demo Salud, Av. Ficticia 100, Miraflores, Lima |
| `CMP-DEMO-0002` | Dra. Elena Carga Demo (por carga masiva) | Policlínico Demo Norte, Los Olivos |

**Recetas** (las tres con fecha de expedición 30/10/2026 y médico Dr. Raúl Médico Demo)

| Número de receta | Paciente | Repeticiones | Para qué medicamento (escrito en «Indicaciones de Uso») | Condición de venta del producto |
|---|---|---|---|---|
| `RS-DEMO-0001` | Luis Paciente Demo | 2 | Amoxicilina Demo 500 mg (`DEMO-MED-0002`) | Receta Simple |
| `RR-DEMO-0001` | María Paciente Demo | 0 | Clonazepam Demo 2 mg (`DEMO-MED-0003`) | Receta Retenida |
| `RE-DEMO-0001` | Luis Paciente Demo | 0 | Tramadol Demo 50 mg (`DEMO-MED-0004`) | Estupefaciente |

Como la receta **no guarda el medicamento ni la cantidad**, no hay una cantidad "autorizada" que configurar: lo que limita las ventas son las repeticiones (la receta simple `RS-DEMO-0001` se puede dispensar 3 veces; las otras dos, 1 vez). Registros de prueba que se crearon y se eliminaron durante el recorrido: un paciente (`PASBORRAR1`), un profesional (`CMP-BORRAR-001`) y una receta (`RX-BORRAR-001`).

## 5. Registrar un paciente

**Cómo llegar:** menú lateral › PACIENTES & ATENCIÓN › **Pacientes** (`/pacientes`).

La pantalla se titula «Pacientes», con las migas «Inicio / Pacientes & Atención / Pacientes» y una tarjeta «Búsqueda de Pacientes» («Ingresa los criterios de búsqueda»). Arriba a la derecha de la tarjeta están los botones **Carga Masiva** y **Nuevo** (los dos con el permiso `Pacientes › Crear`). Los filtros están en la sección 6. Al entrar no hay tabla: aparece al hacer clic en **Buscar**. Con la empresa recién empezada, **Buscar** muestra «Resultados (0)», la cabecera de la tabla y una fila que dice «Resultados» (en esta pantalla la fila vacía no dice «Sin resultados»; esa frase aparece más abajo, junto al paginador).

### 5.1 Pasos

1. Haz clic en **Nuevo** (`/pacientes/nueva`). Se abre un formulario con una flecha para volver y cinco campos.
2. En «Tipo de Documento», elige una opción: «DNI», «Carné de extranjería», «RUC» o «Pasaporte».
3. Escribe el «Documento de Identidad» con el formato que corresponde al tipo.
4. Escribe el «Nombre y Apellidos».
5. Escribe o elige la «Fecha de Nacimiento».
6. Si quieres, escribe un «Contacto Primario» (teléfono o correo).
7. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Tipo de Documento | «DNI», «Carné de extranjería», «RUC» o «Pasaporte» | Sí | Vacío: «Este campo es obligatorio» |
| Documento de Identidad | El número, según el tipo (tabla de abajo) | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 20 caracteres. Con formato equivocado: «El número no corresponde al tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números).» Si cambias el tipo después de escribir el número, la pantalla lo vuelve a revisar al instante |
| Nombre y Apellidos | El nombre completo | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 200 caracteres |
| Fecha de Nacimiento | El día de nacimiento, en formato `dd/MM/aaaa` | Sí | Vacía: «Este campo es obligatorio». El selector **no te impide** una fecha futura; el rechazo llega al guardar (sección 5.3) |
| Contacto Primario | Un teléfono, correo u otro dato de contacto | No | La pantalla deja escribir hasta 150 caracteres, pero **más de 100 se rechazan al guardar** (sección 5.3) |

Formato del número según el tipo (comprobado en la pantalla):

| Tipo | Formato | Ejemplo válido | Ejemplo rechazado |
|---|---|---|---|
| DNI | Exactamente 8 dígitos | `70000001` | `1234`, `123` |
| Carné de extranjería | De 1 a 12 letras o números | `CE0000004` | `AB-12` (lleva un guion) |
| RUC | Exactamente 11 dígitos | `20123456789` | — |
| Pasaporte | De 1 a 20 letras o números | `ABC12345678`, `PASBORRAR1` | — |

### 5.2 Qué pasa al terminar bien

Aparece el aviso «Paciente guardado correctamente» y el sistema te lleva a la búsqueda de pacientes. Si ya habías buscado antes en esta sesión, la pantalla repite esa búsqueda sola y verás la tabla con el paciente nuevo; si no, verás solo los filtros.

Crea ahora los dos primeros pacientes de la sección 4 (Luis y María). Los otros dos entran por carga masiva (sección 7).

### 5.3 Errores que puedes ver (todos comprobados en la aplicación)

| Qué ves | Qué significa | Qué hacer |
|---|---|---|
| «Este campo es obligatorio» bajo cada campo, al hacer clic en **Guardar** con el formulario vacío | Falta un dato obligatorio. No se envía nada | Completa los campos marcados |
| «El número no corresponde al tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números).» bajo «Documento de Identidad» | El número no cumple el formato del tipo elegido | Corrige el número o cambia el tipo |
| «Ya existe un paciente con ese documento de identidad.» | Ya hay un paciente (no eliminado) con ese número de documento. **Se compara solo el número**: con el mismo número no vale cambiar el tipo (se probó con `70000001` como pasaporte) | Busca al paciente existente y úsalo |
| «Error al guardar el paciente» con una fecha de nacimiento futura (se probó `01/01/2030`) | La fecha de nacimiento no puede ser posterior a hoy | Corrige la fecha |
| «Error al guardar el paciente» con un contacto de 101 caracteres | El contacto admite como máximo 100 caracteres, aunque el campo deje escribir 150 | Acorta el contacto a 100 caracteres o menos |

El aviso «Error al guardar el paciente» es genérico: no dice cuál es el problema. Un nombre o un documento formados solo por espacios también deberían rechazarse según la documentación técnica (`RN-PAC-003` y `RN-PAC-005`); No verificado en la aplicación.

Reglas de fondo: `RN-PAC-002` a `RN-PAC-008` en `01b-catalogos.md`.

## 6. Buscar, ver y editar pacientes

**Cómo llegar:** menú lateral › PACIENTES & ATENCIÓN › **Pacientes**.

### 6.1 Los filtros

| Filtro | Qué escribir o elegir | Cómo busca |
|---|---|---|
| Tipo de Documento | «Todos» (valor inicial), «DNI», «Carné de extranjería», «RUC» o «Pasaporte» | Solo los pacientes de ese tipo |
| Documento de Identidad | Parte del número (lleva una lupa) | «Contiene»: `0002` encuentra `70000002`. La tecla Enter también busca |
| Nombre y Apellidos | Parte del nombre (lleva una lupa) | «Contiene», **sin ignorar los acentos**: «María» encuentra a María Paciente Demo, pero «maria» no encuentra nada |
| Fecha Nacimiento Desde | Una fecha | Nacidos ese día o después |
| Fecha Nacimiento Hasta | Una fecha | Nacidos ese día o antes |

Los botones son **Limpiar** (vacía los filtros, oculta la tabla y borra la búsqueda guardada) y **Buscar**. Si pones «Desde» posterior a «Hasta» (se probó 01/01/2000 y 01/01/1990), no hay aviso: la tabla sale con «Resultados (0)».

### 6.2 La tabla de resultados

Encabezado «Resultados (N)». Columnas: «Doc. Identidad» (es un enlace al detalle), «Tipo Doc.» (por ejemplo «DNI» o «Carné de extranjería»), «Nombre y Apellidos», «Fecha Nacimiento» (`dd/MM/aaaa`) y «Contacto Primario». Va ordenada por nombre y apellidos. Debajo hay un paginador («Items por página», 10 por defecto, «Página 1 de 1») y, a la izquierda, el botón **Exportar**. Sin coincidencias la tabla muestra una fila «Resultados» y el paginador dice «Sin resultados».

Comprobado con los pacientes del capítulo: sin filtros salen todos; «Pasaporte» deja solo al paciente de ese tipo; «0002» deja solo a María; «Desde» 01/01/1988 deja a los nacidos después; «Hasta» 31/12/1989 deja a Luis; escribir un documento completo y pulsar Enter lo encuentra. Al volver desde el detalle, la pantalla recuerda tus filtros y repite la búsqueda.

Si un paciente no tiene contacto, la celda de «Contacto Primario» sale vacía cuando lo registraste a mano y con un guion largo («—») cuando entró por carga masiva.

### 6.3 Ver un paciente

Haz clic en el documento de una fila. Se abre la ficha del paciente (`/pacientes/<id>`) con el título «Pacientes», el encabezado «Datos del registro» y los campos en solo lectura. Arriba a la derecha está el botón **Editar** (con el permiso `Pacientes › Actualizar`).

### 6.4 Editar un paciente

1. Abre la ficha y haz clic en **Editar**. Los campos se habilitan y abajo aparecen **Cancelar**, **Guardar** y **Eliminar**.
2. Cambia lo que necesites.
3. Haz clic en **Guardar**.

Aparece «Paciente guardado correctamente» y **te quedas en la ficha**, otra vez en modo solo lectura, con los datos nuevos (se probó cambiando el contacto de Luis a «Tel. 999000112»). **Cancelar** descarta los cambios y vuelve al modo solo lectura. Las reglas y los mensajes son los de la sección 5; además, si cambias el documento por uno que ya tiene otro paciente, sale «Ya existe un paciente con ese documento de identidad.» y la ficha sigue en modo editar.

## 7. Exportar y cargar pacientes desde Excel

### 7.1 Exportar

Haz una búsqueda y, bajo la tabla, haz clic en **Exportar**. Se descarga un Excel con **cinco columnas** («Doc. Identidad», «Tipo Doc.», «Nombre y Apellidos», «Fecha Nacimiento», «Contacto Primario») y una fila por paciente, con las fechas en `dd/MM/aaaa`. Se comprobó el contenido con los pacientes del capítulo. Usa los filtros que tengas escritos en ese momento. El nombre del archivo en tu disco (`pacientes.xlsx` según el inventario de pantallas): No verificado en la aplicación, porque el navegador integrado no muestra el archivo guardado. Si falla, el aviso es «Error al buscar pacientes» (No verificado: no se pudo provocar).

### 7.2 Cargar muchos pacientes (Carga Masiva)

**Cómo llegar:** **Pacientes** › **Carga Masiva** (`/pacientes/carga-masiva`). Pide `Pacientes › Crear`.

La pantalla se titula «Carga Masiva» (migas «Inicio / Pacientes & Atención / Pacientes / Carga Masiva»), con el botón **Descargar plantilla** y una zona que dice «Arrastra tu archivo Excel aquí o haz clic para seleccionar» y «Solo archivos .xlsx (máx. 5 MB y 5000 filas)».

1. Haz clic en **Descargar plantilla**. El Excel trae una sola fila con cinco encabezados: `TipoDocumento`, `DocIdentidadGub`, `NombreApellidos`, `FechaNacimiento` y `ContactoPrimario`.
2. Llena una fila por paciente. El tipo se escribe como código (`1`, `4`, `6`, `7`) o como abreviatura: `DNI`, `CE`, `RUC` o `PASAPORTE` (`RN-PAC-013`; en la prueba se usaron `DNI` y `CE`). La fecha va como `dd/MM/aaaa`. Los cuatro primeros datos son obligatorios.
3. Entrega el archivo en la zona de carga.
4. La pantalla muestra una **Vista previa**: «N filas totales», «N filas válidas» y «N filas con error», y una tabla con las columnas de la plantilla, una columna de errores y una columna «Estado» (un visto para las válidas, un signo de alerta para las que tienen error).
5. Haz clic en **Guardar válidos (N)**. Solo se guardan las filas válidas.

Se probó con cuatro filas: dos correctas, una con DNI de 3 dígitos y una con un documento que ya existía. La vista previa dijo «4 filas totales», «2 filas válidas», «2 filas con error», y mostró estos mensajes en la fila con error:

| Mensaje | Qué significa |
|---|---|
| «'docIdentidadGub' no tiene el formato de su tipo de documento (DNI: 8 dígitos, RUC: 11 dígitos, CE: hasta 12 y pasaporte: hasta 20 letras o números)» | El número no cumple el formato de su tipo. El nombre del campo sale en su forma técnica |
| «El valor '70000001' del campo 'docIdentidadGub' ya existe en el sistema» | Ese documento ya está registrado |

Al guardar aparece «Se guardaron 2 registros correctamente» y vuelves a la búsqueda de pacientes, que repite la búsqueda y muestra los nuevos (Pedro Carga Demo y Ana Extranjera Demo).

Detalle que verás en la vista previa: **la primera columna se rotula con el texto crudo `fields.tipoDocumento`** en lugar de «Tipo de Documento» (error de la pantalla, sección 14).

No se pudieron probar: un archivo de más de 5 MB o de más de 5000 filas, un archivo vacío, un documento repetido dentro del mismo archivo y una fecha escrita con otro formato (No verificado en la aplicación; las reglas están en `RN-PAC-014` a `RN-PAC-017`). La entrega del archivo se hizo desde el navegador integrado, que no puede abrir el selector de archivos; lo que viene después (vista previa y guardado) es el flujo normal.

## 8. Eliminar un paciente

El botón **Eliminar** (con el permiso `Pacientes › Eliminar`) solo aparece **en modo editar** de un paciente que ya existe, abajo, a la derecha de **Guardar**.

### 8.1 Un paciente de prueba sin recetas

1. Crea un paciente de prueba (por ejemplo, tipo «Pasaporte», documento `PASBORRAR1`, nombre «Paciente Prueba Borrar»).
2. Ábrelo, haz clic en **Editar** y luego en **Eliminar**.
3. Aparece el cuadro «Confirmar eliminación» con el texto «¿Estás seguro de que deseas eliminar este paciente?» y los botones **Cancelar** y **Confirmar**. **Cancelar** cierra el cuadro y no hace nada.
4. Haz clic en **Confirmar**.

Aparece «Paciente eliminado correctamente» y vuelves a la búsqueda, donde el paciente ya no está.

### 8.2 Un paciente que tiene recetas

Se probó con Luis Paciente Demo, que tiene dos recetas. Al confirmar, aparece «No puedes eliminar este paciente porque tiene recetas, ventas u otros registros asociados.» El paciente **no se elimina** y la ficha sigue en modo editar (haz clic en **Cancelar**). La API bloquea también si el paciente tiene ventas, citas de servicios o inmunizaciones (`RN-PAC-010`; solo se comprobó el caso de las recetas).

Si el aviso es «Error al eliminar el paciente», el problema fue otro (por ejemplo, ya no existe): No verificado en la aplicación.

## 9. Profesionales de salud

**Cómo llegar:** menú lateral › PACIENTES & ATENCIÓN › **Profesionales de Salud** (`/profesionales-salud`).

La pantalla se titula «Profesionales de Salud», con las migas «Inicio / Pacientes & Atención / Profesionales de Salud» y una tarjeta «Búsqueda de Profesionales de Salud» («Ingresa los criterios de búsqueda»). Arriba a la derecha están **Carga Masiva** y **Nuevo** (permiso `Profesionales de Salud › Crear`). Funciona igual que Pacientes: la tabla aparece al hacer clic en **Buscar**.

### 9.1 Registrar un profesional de salud

1. Haz clic en **Nuevo** (`/profesionales-salud/nueva`). Se abre un formulario con tres campos.
2. Escribe el «Número de Registro».
3. Escribe el «Nombre del Prescriptor».
4. Si quieres, escribe la «Dirección de Clínica».
5. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Número de Registro | El número de colegiatura o registro del médico (por ejemplo `CMP-DEMO-0001`) | Sí | Vacío: «Este campo es obligatorio». No deja escribir más de 50 caracteres. No tiene formato fijo. **No puede repetirse** entre profesionales de la empresa |
| Nombre del Prescriptor | El nombre del médico | Sí | Vacío: «Este campo es obligatorio». La pantalla deja escribir hasta 200 caracteres, pero **más de 150 se rechazan al guardar** (se probó con 151) |
| Dirección de Clínica | La dirección del consultorio o la clínica | No | La pantalla deja escribir hasta 300 caracteres; la documentación técnica dice que más de 255 se rechazan al guardar (No verificado en la aplicación) |

Al terminar aparece «Profesional de salud guardado correctamente» y vuelves a la búsqueda (si ya habías buscado, la tabla se repite con el nuevo registro). Crea ahora `CMP-DEMO-0001` «Dr. Raúl Médico Demo» con su dirección.

Errores comprobados:

| Qué ves | Qué significa | Qué hacer |
|---|---|---|
| «Este campo es obligatorio» bajo «Número de Registro» y «Nombre del Prescriptor», al hacer clic en **Guardar** vacío | Falta un dato obligatorio | Complétalos |
| «Ya existe un profesional de salud con ese número de registro.» | Ese número ya lo tiene otro profesional (se probó con `CMP-DEMO-0001`) | Busca al profesional existente o usa otro número |
| «Error al guardar el profesional de salud» con un nombre de 151 caracteres | El nombre admite como máximo 150 caracteres | Acórtalo |

### 9.2 Buscar, ver y editar

**Filtros:** «Número de Registro» y «Nombre del Prescriptor» (los dos con lupa y búsqueda «contiene»; Enter busca). Botones **Limpiar** y **Buscar**. Se comprobó que «DEMO» encuentra `CMP-DEMO-0001`, que «raúl» encuentra al Dr. Raúl y que «Raul» (sin tilde) no encuentra nada.

**Tabla:** «Resultados (N)» con las columnas «Número de Registro» (enlace al detalle), «Nombre del Prescriptor» y «Dirección de Clínica» («—» si no tiene). Va ordenada por nombre del prescriptor. Sin coincidencias, la fila dice «Resultados» y el paginador «Sin resultados».

**Ver:** haz clic en el número de registro. La ficha muestra «Datos del registro», los tres campos en solo lectura y el botón **Editar** (permiso `Profesionales de Salud › Actualizar`).

**Editar:** haz clic en **Editar** (aparecen **Cancelar**, **Guardar** y **Eliminar**), cambia los datos y haz clic en **Guardar**. Aparece «Profesional de salud guardado correctamente» y te quedas en la ficha, en solo lectura (se probó ampliando la dirección del Dr. Raúl a «Clínica Demo Salud, Av. Ficticia 100, Miraflores, Lima»).

### 9.3 Exportar

Bajo la tabla, **Exportar** descarga un Excel con tres columnas («Número de Registro», «Nombre del Prescriptor», «Dirección de Clínica»); si no hay dirección la celda queda vacía. Se comprobó el contenido. Nombre del archivo en disco (`profesionales-salud.xlsx` según el inventario): No verificado en la aplicación.

### 9.4 Cargar profesionales desde Excel

**Cómo llegar:** **Profesionales de Salud** › **Carga Masiva** (`/profesionales-salud/carga-masiva`; permiso `Profesionales de Salud › Crear`). Es la misma pantalla de la sección 7.2. La plantilla trae tres encabezados: `NumeroRegistro`, `NombrePrescriptor` y `DireccionClinica` (los dos primeros obligatorios).

Se probó con tres filas: una correcta (`CMP-DEMO-0002`, «Dra. Elena Carga Demo»), una con un número que ya existía y una sin número. La vista previa dijo «3 filas totales», «1 filas válidas» y «2 filas con error», con las columnas «Número de Registro», «Nombre del Prescriptor» y «Dirección Clínica» (sin la palabra «de»), y estos mensajes:

| Mensaje | Qué significa |
|---|---|
| «El valor 'CMP-DEMO-0001' del campo 'numeroRegistro' ya existe en el sistema» | Ese número ya está registrado |
| «El campo 'numeroRegistro' es obligatorio» | La fila no trae número de registro |

**Guardar válidos (1)** muestra «Se guardaron 1 registros correctamente» y te lleva a la búsqueda. Lo que no se pudo probar es lo mismo de la sección 7.2.

### 9.5 Eliminar un profesional de salud

El botón **Eliminar** (permiso `Profesionales de Salud › Eliminar`) aparece solo en modo editar. El cuadro dice «Confirmar eliminación» y «¿Estás seguro de que deseas eliminar este profesional de salud?».

- **Profesional de prueba sin recetas** (`CMP-BORRAR-001`): al confirmar aparece «Profesional de salud eliminado correctamente» y vuelves a la búsqueda.
- **Profesional con recetas** (el Dr. Raúl): aparece «No puedes eliminar este profesional de salud porque tiene recetas asociadas.» y el profesional no se elimina (la ficha sigue en modo editar; usa **Cancelar**).
- «Error al eliminar el profesional de salud»: otro problema, por ejemplo que ya no exista (No verificado en la aplicación).

## 10. Registrar una receta

**Cómo llegar:** menú lateral › PACIENTES & ATENCIÓN › **Recetas Médicas** (`/recetas-medicas`).

La pantalla se titula «Recetas Médicas», con las migas «Inicio / Pacientes & Atención / Recetas Médicas» y una tarjeta «Búsqueda de Recetas Médicas» («Ingresa los criterios de búsqueda»). Arriba a la derecha de la tarjeta están **Escanear receta** (con el permiso `Recetas Médicas › Analizar con IA`, sección 12) y **Nuevo** (con `Recetas Médicas › Crear`). Los filtros están en la sección 11.

Antes de registrar una receta, el paciente y el médico **ya deben existir** (secciones 5 y 9). Un cajero con `Recetas Médicas › Crear` puede registrar la receta, pero no puede registrar al paciente (no tiene `Pacientes › Crear`): pídele al administrador que lo cree antes.

### 10.1 Pasos

1. Haz clic en **Nuevo** (`/recetas-medicas/nueva`). Se abre un formulario con una flecha para volver y seis campos: «Paciente», «Médico», «Fecha de Expedición», «Repeticiones Máximas», «Número de receta» e «Indicaciones de Uso». **No hay un lugar para agregar medicamentos ni cantidades.**
2. En «Paciente» (placeholder «Buscar paciente...»), escribe parte del nombre y **elige una opción de la lista**. Cada opción muestra el nombre y el documento, por ejemplo «Luis Paciente Demo — 70000001». La lista tarda un instante en llenarse.
3. En «Médico» (placeholder «Buscar médico...»), haz lo mismo. Cada opción muestra el nombre y el número de registro, por ejemplo «Dr. Raúl Médico Demo — CMP-DEMO-0001».
4. Escribe o elige la «Fecha de Expedición» (la que trae la receta de papel).
5. Deja o cambia las «Repeticiones Máximas» (empieza en 0).
6. Escribe el «Número de receta» si la receta de papel lo trae.
7. Escribe en «Indicaciones de Uso» qué medicamento y cómo se toma.
8. Haz clic en **Guardar**.

| Campo | Qué escribir | Obligatorio | Reglas y mensaje si te equivocas |
|---|---|---|---|
| Paciente | El paciente de la receta | Sí | Debes **elegir una opción de la lista**; escribir el nombre sin elegir no basta. Sin elegir: «Debes seleccionar un paciente». Sin coincidencias, la lista muestra la opción «Sin resultados» |
| Médico | El profesional de salud que firmó la receta | Sí | Igual: «Debes seleccionar un médico» |
| Fecha de Expedición | El día en que se expidió la receta | Sí | Vacía: «La fecha de expedición es requerida». **No puede ser posterior a hoy**: el selector no deja elegir días futuros y, si escribes a mano una fecha futura (se probó `01/12/2026`), sale el mismo mensaje «La fecha de expedición es requerida». Se aceptó la fecha de hoy. No hay límite hacia atrás: el sistema no vence recetas |
| Repeticiones Máximas | Un número entero desde 0 | Sí | Empieza en 0. Si lo dejas vacío o escribes un número negativo (se probó `-1`), **no se guarda y no aparece ningún mensaje** |
| Número de receta | El número de la receta de papel | No (sí al vender receta retenida o estupefaciente) | No deja escribir más de 30 caracteres (por eso el mensaje «El número de receta admite hasta 30 caracteres» del inventario de pantallas no llega a verse). Bajo el campo dice «Necesario para vender receta retenida o estupefacientes.» **Se puede repetir** en otra receta (se probó con `RS-DEMO-0001`) |
| Indicaciones de Uso | Texto libre (en el manual, el medicamento y la dosis) | No | La pantalla no limita el largo; la documentación técnica dice que más de 1000 caracteres se rechazan al guardar con «Revisa los datos ingresados de la receta.» (No verificado en la aplicación) |

### 10.2 Qué pasa al terminar bien

Aparece «Receta creada correctamente» y el sistema te lleva a la búsqueda de recetas. Si ya habías buscado antes en esta sesión, verás la tabla con la receta nueva.

Crea ahora las tres recetas de la sección 4. Usa la fecha 30/10/2026, el médico Dr. Raúl Médico Demo y estos datos:

| Paciente | Repeticiones Máximas | Número de receta | Indicaciones de Uso |
|---|---|---|---|
| Luis Paciente Demo | 2 | `RS-DEMO-0001` | Amoxicilina Demo 500 mg: 1 cápsula cada 8 horas por 7 días (receta de prueba) |
| María Paciente Demo | 0 | `RR-DEMO-0001` | Clonazepam Demo 2 mg: 1 tableta cada noche por 30 días (receta retenida de prueba) |
| Luis Paciente Demo | 0 | `RE-DEMO-0001` | Tramadol Demo 50 mg: 1 cápsula cada 12 horas por 5 días (receta de estupefaciente de prueba) |

### 10.3 Errores que puedes ver

| Qué ves | Qué significa | Qué hacer |
|---|---|---|
| «Debes seleccionar un paciente», «Debes seleccionar un médico» y «La fecha de expedición es requerida» al hacer clic en **Guardar** vacío (comprobado) | Faltan datos obligatorios. No se envía nada | Complétalos |
| **Guardar** no hace nada y no muestra aviso (comprobado con repeticiones `-1`) | Las repeticiones están vacías o son negativas | Escribe 0 o un número mayor |
| «El paciente seleccionado no existe. Vuelve a seleccionarlo.» / «El médico seleccionado no existe. Vuelve a seleccionarlo.» | Alguien eliminó al paciente o al médico mientras llenabas la receta | Elígelo de nuevo (No verificado en la aplicación: no se pudo provocar) |
| «La fecha de expedición no puede ser posterior a hoy.» | El servidor rechazó una fecha futura (cerca de la medianoche puede discrepar del navegador) | Corrige la fecha (No verificado en la aplicación: la pantalla ya lo impide) |
| «Revisa los datos ingresados de la receta.» | Repeticiones, número o indicaciones fuera de rango en el servidor | Revisa esos campos (No verificado en la aplicación) |
| «Error al crear la receta» | Cualquier otro fallo al crear | Vuelve a intentarlo; si sigue, avisa a soporte (No verificado en la aplicación) |

## 11. Buscar, ver, editar, eliminar y exportar recetas

### 11.1 Los filtros

| Filtro | Qué escribir o elegir | Cómo busca |
|---|---|---|
| Paciente | Parte del nombre y **elige una opción de la lista** (placeholder «Buscar paciente...») | Solo las recetas de ese paciente. **Si no eliges una opción no filtra**, y si después cambias el texto, la elección se descarta (comprobado) |
| Médico | Igual, con profesionales (placeholder «Buscar médico...») | Solo las recetas de ese médico |
| Fecha desde | Una fecha | Expedidas ese día o después |
| Fecha hasta | Una fecha | Expedidas ese día o antes |

Botones **Limpiar** y **Buscar**. En estos filtros la tecla Enter no busca (según el inventario de pantallas; No verificado en la aplicación). Comprobado con las recetas del capítulo: elegir a María deja 1 receta; «Fecha desde» 31/10/2026 deja «Resultados (0)» con la fila «Sin resultados»; «Fecha desde» y «Fecha hasta» ambas 30/10/2026 dejan las tres recetas (los dos extremos cuentan).

### 11.2 La tabla de resultados

Encabezado «Resultados (N)». Columnas: «Paciente» (enlace al detalle), «Médico», «Fecha de Expedición» (`dd/MM/aaaa`), «Repeticiones» e «Indicaciones» («—» si no hay). Va ordenada por fecha de expedición, la más reciente primero; entre recetas del mismo día el orden no es fijo. **El número de receta no sale en la tabla ni en el Excel**: se ve en el detalle.

### 11.3 Ver y editar una receta

1. Haz clic en el nombre del paciente de una fila. Se abre «Detalle de Receta» (`/recetas-medicas/<id>`) con «Datos del registro», los seis campos en solo lectura y el botón **Editar** (permiso `Recetas Médicas › Actualizar`). Un cajero ve la ficha sin ese botón.
2. Haz clic en **Editar**. Los campos se habilitan, con el paciente y el médico ya elegidos (no hay que volver a elegirlos). Abajo aparecen **Eliminar** (a la izquierda), **Cancelar** y **Guardar** (a la derecha).
3. Cambia lo que necesites y haz clic en **Guardar**.

Aparece «Receta actualizada correctamente» y te quedas en la ficha, en solo lectura, con los datos nuevos (se probó subiendo las repeticiones de una receta de prueba de 1 a 3 y cambiando sus indicaciones). **Cancelar** vuelve al modo solo lectura sin guardar. Los mensajes de error son los de la sección 10.3, salvo el último: al editar es «Error al actualizar la receta».

### 11.4 Eliminar una receta

El botón **Eliminar** (permiso `Recetas Médicas › Eliminar`) aparece solo en modo editar, abajo a la izquierda.

1. Haz clic en **Editar** y luego en **Eliminar**.
2. Aparece el cuadro «Eliminar receta» con el texto «¿Estás seguro de que deseas eliminar esta receta médica? Esta acción no se puede deshacer.» y los botones **Cancelar** y **Confirmar**. **Cancelar** no hace nada.
3. Haz clic en **Confirmar**.

Aparece «Receta eliminada correctamente» y vuelves a la búsqueda (comprobado con una receta de prueba, `RX-BORRAR-001`, que se creó, se editó y se eliminó).

Si la receta ya se usó en una venta (incluso en una venta anulada), no se puede eliminar y aparece «No puedes eliminar esta receta porque ya se usó en ventas.» No verificado en la aplicación en este capítulo, porque todavía no hay ventas; se vuelve a mencionar en los capítulos 8 y 9. Otro fallo: «Error al eliminar la receta».

### 11.5 Exportar

Bajo la tabla, **Exportar** descarga un Excel con cinco columnas («Paciente», «Médico», «Fecha de Expedición», «Repeticiones», «Indicaciones») y las fechas en `dd/MM/aaaa`. **No incluye el número de receta.** Se comprobó el contenido con las tres recetas. Usa los filtros que tengas en ese momento. Nombre del archivo en disco (`recetas-medicas.xlsx` según el inventario): No verificado en la aplicación. Si falla: «Error al buscar recetas médicas» (No verificado: no se pudo provocar). No existe carga masiva de recetas.

## 12. El Lector de Recetas IA (escáner)

**Cómo llegar:** Recetas Médicas › **Escanear receta** (`/recetas-medicas/escanear`). Pide `Recetas Médicas › Analizar con IA`. Si tu rol no lo tiene, el botón no aparece y, si escribes la dirección, la aplicación te lleva al Dashboard con el aviso «No tienes permisos para acceder a esta pantalla.» (No verificado en la aplicación para este permiso: el administrador y el cajero del manual sí lo tienen; el aviso se comprobó con otra pantalla, sección 13.)

**Qué hace:** subes una foto o imagen de una receta, la inteligencia artificial extrae el texto, lo compara con el catálogo DIGEMID y con tus medicamentos, y te muestra qué medicamentos reconoció, la cantidad que leyó y cuánto stock tienes. **Solo muestra el resultado en pantalla: no crea la receta, no guarda nada y no te lleva al punto de venta.** Para vender, registra la receta a mano (sección 10). La imagen se envía a un servicio externo de IA, y cada análisis consume parte del cupo de ese servicio.

### 12.1 La pantalla

Título «Lector de Recetas IA» (migas «Inicio / Pacientes & Atención / Recetas Médicas» y una flecha para volver). Una tarjeta «Escanear receta médica», con la insignia «ANÁLISIS CON IA» y el subtítulo «Toma una foto de la receta o selecciona una imagen de la galería». La zona central dice «Captura o selecciona una imagen para comenzar» y «o arrastra una imagen aquí». Debajo hay dos botones: **Tomar foto** (en un teléfono abre la cámara trasera; en un computador abre el selector de archivos) y **Galería**.

### 12.2 Pasos

1. Haz clic en **Galería** (o en **Tomar foto**) y elige la imagen de la receta, o arrástrala a la zona central.
2. Se muestra la vista previa de la imagen y, debajo, su nombre de archivo. Aparecen los botones **Limpiar** (quita la imagen y los resultados) e **Interpretar receta**.
3. Haz clic en **Interpretar receta**. Aparece «Interpretando...» con «Análisis con IA»; los botones se bloquean. En la prueba tardó entre 30 y 60 segundos.
4. Cuando termina, debajo de la tarjeta aparecen los resultados. Puedes elegir otra imagen y volver a interpretar.

La prueba del manual se hizo con **una imagen inventada** (generada para la prueba): un texto que dice «RECETA MEDICA - DATOS FICTICIOS (PRUEBA)», un paciente y un médico ficticios y dos medicamentos, «Amoxicilina 500 mg capsulas, cantidad 21» y «Paracetamol 500 mg tabletas, cantidad 10». Se hizo **un solo análisis real**.

### 12.3 Qué muestran los resultados (comprobado con esa imagen)

**Medicamentos de la receta (2)**: una tarjeta por medicamento detectado.

| Dato | Amoxicilina | Paracetamol |
|---|---|---|
| Nombre oficial (tu medicamento que mejor coincide) | Amoxicilina Demo 500 mg | Paracetamol Demo 500 mg |
| «Detectado como: "…"» (lo que la IA leyó) | «Detectado como: "Amoxicilina"» | «Detectado como: "Paracetamol"» |
| Insignia «DIGEMID» | Sí (el nombre se validó contra el catálogo DIGEMID) | Sí |
| «Cantidad sugerida: N» | 21 | 10 |
| «Confianza: …» | Alta | Alta |
| Stock en tu sucursal | «En stock: 100 unidades» | «En stock: 215 unidades» |

La confianza se rotula «Alta» (0,85 o más), «Media» (0,65 o más) o «Baja» (menos). El stock es el de **tu sucursal** (la de tu sesión) y cuenta solo lotes no vencidos: en la prueba, la Sede Principal tenía 100 de Amoxicilina y 215 de Paracetamol (200 del lote normal más 15 del lote que vence pronto), igual que en el capítulo 6.

**Sugerencias IA (2)**: medicamentos complementarios o genéricos que propone la IA y que existen en tu catálogo. Cada tarjeta muestra el nombre, «Sugerido por Amoxicilina» (o «Sugerido por Paracetamol») y el stock. En esta prueba las sugerencias repitieron los mismos dos medicamentos de la receta.

### 12.4 Lo que no se pudo comprobar en la aplicación

Para no gastar el cupo de IA solo se hizo un análisis. Lo siguiente está tomado del inventario de pantallas (`19e` §5.3) y las reglas `RN-REC-018` a `RN-REC-029`, y es **No verificado en la aplicación**:

| Situación | Qué muestra la pantalla según el inventario |
|---|---|
| Cualquier error: demasiadas peticiones (la IA admite 10 análisis por minuto por empresa), archivo demasiado grande (el servidor rechaza más de 10 MB), archivo inválido, falla del servicio de IA o sin permiso | Un solo aviso: «Error al interpretar la receta. Intenta nuevamente.» La pantalla no distingue el motivo. Si es por demasiadas peticiones, espera un minuto; si es el archivo, prueba con otra imagen más liviana |
| La IA no detecta medicamentos | Tarjeta «No se detectaron medicamentos en la imagen. Intenta con una foto más nítida.» |
| Un medicamento sin stock | «Sin stock» en su tarjeta |
| Medicamento sin stock pero con equivalentes (mismo ingrediente activo, concentración y unidad) con stock | Lista «Equivalentes terapéuticos:» con su laboratorio y stock, y el aviso «No hay stock del medicamento prescrito, pero hay equivalentes disponibles.» |
| Confianza «Media» o «Baja» | Se rotula así con otro color |
| Máximos | Se procesan como mucho 20 medicamentos y 3 sugerencias por cada uno |
| Arrastrar y soltar un archivo que no es imagen | Se ignora sin avisar |
| Tomar foto con la cámara de un teléfono | No se probó (el navegador integrado no tiene cámara) |

## 13. Lo que ve un cajero frente a un administrador

Comprobado con `carlos.cajero` (rol «Cajero» del capítulo 3), que tiene `Pacientes › Leer`, `Profesionales de Salud › Leer` y `Recetas Médicas › Leer`, `Crear` y `Analizar con IA`.

| Pantalla | Administrador (`ana.manual`) | Cajero (`carlos.cajero`) |
|---|---|---|
| Menú | Grupo PACIENTES & ATENCIÓN con «Pacientes», «Profesionales de Salud» y «Recetas Médicas» | Las mismas tres opciones |
| Pacientes (búsqueda) | **Carga Masiva**, **Nuevo**, **Limpiar**, **Buscar** y **Exportar** | Solo **Limpiar**, **Buscar** y **Exportar**. No ve **Carga Masiva** ni **Nuevo**: no puede registrar pacientes |
| Pacientes (ficha) | **Editar** y, en modo editar, **Eliminar** | Solo ve los datos: sin **Editar** (y por tanto sin **Eliminar**) |
| Profesionales de Salud (búsqueda) | **Carga Masiva**, **Nuevo**, **Limpiar**, **Buscar**, **Exportar** | Solo **Limpiar** y **Buscar** (y **Exportar** tras buscar, por tener `Leer`; solo se comprobó el botón en Pacientes y Recetas Médicas) |
| Profesionales de Salud (ficha) | **Editar** y **Eliminar** | Solo lectura |
| Recetas Médicas (búsqueda) | **Escanear receta**, **Nuevo**, **Limpiar**, **Buscar**, **Exportar** | Los mismos cinco botones, porque el rol Cajero tiene `Crear` y `Analizar con IA` |
| Recetas Médicas (ficha) | **Editar** y **Eliminar** | Solo lectura: sin **Editar** ni **Eliminar** |
| Registrar una receta | Sí | Sí. El formulario abre con **Cancelar** y **Guardar**, y el campo «Paciente» ofrece los pacientes («Luis Paciente Demo — 70000001»), porque tiene `Pacientes › Leer` (no se guardó ninguna receta con el cajero) |

Si el cajero escribe en el navegador `/pacientes/nueva` o `/pacientes/carga-masiva`, la aplicación lo lleva al **Dashboard** y muestra «No tienes permisos para acceder a esta pantalla.» (comprobado).

Consecuencia práctica: el cajero **puede registrar la receta** de un cliente que llega a la botica, pero **no puede crear al paciente**. Si el paciente es nuevo, el administrador debe registrarlo antes. Si quieres que el cajero también registre pacientes, agrega `Pacientes › Crear` a su rol (capítulo 3); este capítulo no cambia permisos.

Después de la prueba se cerró la sesión del cajero y se volvió a iniciar sesión como administrador.

## 14. Diferencias con la documentación técnica

Lo que la aplicación hace distinto de lo que dicen los documentos, o lo que el inventario de pantallas dejaba como «No verificado» y ahora se comprobó:

| # | Documento y apartado | Qué dice | Qué hace la aplicación |
|---|---|---|---|
| 1 | `19e` §3.1 (tabla de resultados de Pacientes) | «Contacto Primario»: «—» si es nulo | Solo sale «—» cuando el paciente entró por carga masiva sin contacto. Un paciente registrado a mano sin contacto muestra la celda **vacía** (se probó con `PASBORRAR1`) |
| 2 | `19e` §3.3 y §7 (hallazgo 7) | La primera columna de la vista previa de la carga masiva de pacientes mostraría el texto crudo `fields.tipoDocumento` (No verificado) | **Confirmado**: la vista previa muestra `fields.tipoDocumento` en lugar de «Tipo de Documento» |
| 3 | `19e` §3.3 | Los mensajes de la vista previa llevan `'{{field}}'` | Confirmado: el campo sale con su nombre técnico, por ejemplo `'docIdentidadGub'` y `'numeroRegistro'`, no con la etiqueta de la pantalla |
| 4 | `19e` §3.2 (observaciones) | Fecha futura y contacto de 101 a 150 caracteres darían «Error al guardar el paciente» (No verificado) | **Confirmado** con `01/01/2030` y con 101 caracteres |
| 5 | `19e` §4.2 (observaciones) | Un nombre de prescriptor de 151 a 200 caracteres daría «Error al guardar el profesional de salud» (No verificado) | **Confirmado** con 151 caracteres |
| 6 | `19e` §5.2 | «Repeticiones Máximas» vacío o negativo no guarda y no muestra mensaje | **Confirmado** con `-1` |
| 7 | `19e` §5.2 y §6 | Eliminar paciente, profesional o receta con dependencias da el mensaje «en uso» | **Confirmado** para paciente y profesional con recetas (commit `27597cd`). Para la receta usada en una venta: No verificado (no hay ventas) |
| 8 | `19e` §5.3 | Los resultados del escáner traen «Medicamentos de la receta» y «Sugerencias IA» | Confirmado. En la prueba, las «Sugerencias IA» repitieron los mismos dos medicamentos de la receta, y el botón **Interpretar receta** sigue visible después de mostrar los resultados |
| 9 | `19e` §3.3 y §4.3 | Mensaje de éxito de la carga masiva | Sale «Se guardaron 1 registros correctamente» incluso con un solo registro (plural fijo) |

Otro punto: puede que esperes encontrar «líneas» de medicamentos en la receta o recetas «vigentes». **No existen** en la aplicación ni en el modelo: la receta solo tiene cabecera y el sistema no vence recetas (`RN-REC-007`, `19e` §5). No es una diferencia entre el documento y la aplicación, sino una limitación del producto.

## 15. Pendientes y «No verificado»

**Pendientes del producto** (no son errores de este manual):

- Recetas: no hay líneas de medicamentos, cantidades ni vigencia; el escáner no crea la receta ni pasa al punto de venta; un único aviso de error para todos los fallos del análisis.
- Carga masiva de pacientes: la primera columna de la vista previa sale con el texto técnico `fields.tipoDocumento`, y los mensajes usan nombres técnicos de campo.
- Pacientes y profesionales: los límites del formulario son mayores que los del servidor (contacto 150 contra 100; nombre del prescriptor 200 contra 150; dirección de clínica 300 contra 255) y el aviso es genérico.
- La política «exigir registro de receta» no se puede cambiar desde la aplicación.

**No verificado en la aplicación** (motivo entre paréntesis):

1. Escáner: errores por demasiadas peticiones, archivo de más de 10 MB, archivo inválido, falla del servicio de IA (para no gastar el cupo: solo se hizo un análisis).
2. Escáner: «No se detectaron medicamentos…», «Sin stock», «Equivalentes terapéuticos:», «No hay stock del medicamento prescrito, pero hay equivalentes disponibles.», confianza «Media» y «Baja», arrastrar y soltar, **Tomar foto** con cámara.
3. Nombre y documento de un paciente formados solo por espacios; dirección de clínica de más de 255 caracteres; indicaciones de más de 1000 caracteres.
4. Carga masiva (pacientes y profesionales): archivo de más de 5 MB o de más de 5000 filas, archivo vacío, repetidos dentro del mismo archivo, fecha con otro formato, tipo escrito como código numérico o como `RUC`/`PASAPORTE`.
5. Nombre de los archivos descargados (`pacientes.xlsx`, `profesionales-salud.xlsx`, `recetas-medicas.xlsx`, `plantilla-pacientes.xlsx`, `plantilla-profesionales-salud.xlsx`) y avisos de error de búsqueda y exportación.
6. Más de 10 resultados (paginación, «25» y «50» filas por página) y que **Buscar** no vuelve a la página 1.
7. Elegir paciente o médico con el teclado (flechas y Enter) en el formulario de la receta, y que Enter busque en los filtros de la receta.
8. Qué ocurre con un rol sin `Pacientes › Leer` o `Profesionales de Salud › Leer` al escribir en los campos de la receta; ocultar el grupo del menú cuando no hay ninguna opción visible; el botón **Escanear receta** para un rol sin `Analizar con IA`.
9. Eliminar una receta usada en una venta, un paciente con ventas, citas o inmunizaciones (aún no hay ventas); capítulos 8 y 9.
10. Que el paciente o el médico anteriores sigan enviándose en el formulario de la receta si cambias el texto sin elegir otra opción (`19e` §5.2); en la búsqueda sí se comprobó que la elección se descarta.
11. Los avisos «El paciente seleccionado no existe…», «El médico seleccionado no existe…», «La fecha de expedición no puede ser posterior a hoy.», «Revisa los datos ingresados de la receta.», «Error al crear la receta», «Error al actualizar la receta», «Error al cargar el paciente / el profesional de salud / la receta» (no se pudieron provocar).
12. Control de dispensaciones por repeticiones (`RN-REC-031`) y la casilla «Receta verificada»: se recorren en el capítulo 8.

**En curso (sin commitear):** ninguno (`git status --short` sin cambios en ambos repos al empezar y al cerrar).

## 16. Tabla de cobertura

(V) = comprobado en la aplicación. «No verificado» = ver sección 15.

### 16.1 Pacientes — búsqueda `/pacientes` (19e §3.1)

| Elemento | Apartado |
|---|---|
| Título «Pacientes», migas, tarjeta «Búsqueda de Pacientes», subtítulo | 5 (V) |
| Filtros «Tipo de Documento», «Documento de Identidad», «Nombre y Apellidos», «Fecha Nacimiento Desde», «Fecha Nacimiento Hasta» | 6.1 (V) |
| Búsqueda «contiene», sensible a acentos; Enter en el documento | 6.1 (V) |
| Sin validación cruzada entre fechas | 6.1 (V) |
| Tabla: «Doc. Identidad» (enlace), «Tipo Doc.», «Nombre y Apellidos», «Fecha Nacimiento», «Contacto Primario» | 6.2 (V) |
| Orden por nombre, «Resultados (N)», fila vacía «Resultados», paginador | 6.2 (V; más de 10 filas y tamaños 25 y 50: No verificado) |
| Botones **Carga Masiva**, **Nuevo**, **Limpiar**, **Buscar**, **Exportar** | 5, 6, 7 (V) |
| Exportación: cinco encabezados, fechas `dd/MM/aaaa` | 7.1 (V el contenido; el nombre en disco: No verificado) |
| «Error al buscar pacientes» (búsqueda y exportación) | 7.1 (No verificado: no se pudo provocar) |
| Restauración de filtros al volver | 6.2 (V) |
| Permiso `Pacientes › Leer` y menú | 2, 13 (V) |

### 16.2 Pacientes — detalle `/pacientes/nueva` y `/pacientes/:id` (19e §3.2)

| Elemento | Apartado |
|---|---|
| Modos crear, ver y editar; «Datos del registro» | 5.1, 6.3, 6.4 (V) |
| Campos «Tipo de Documento», «Documento de Identidad» (máx. 20), «Nombre y Apellidos» (máx. 200), «Fecha de Nacimiento», «Contacto Primario» (máx. 150) | 5.1 (V) |
| «Este campo es obligatorio» y el mensaje de formato por tipo; revalidación al cambiar el tipo | 5.1, 5.3 (V) |
| Formatos DNI, CE, RUC, pasaporte | 5.1 (V DNI, CE, RUC y pasaporte con un ejemplo cada uno) |
| Botones **Editar**, **Cancelar**, **Guardar**, **Eliminar**, volver | 5, 6.4, 8 (V) |
| «Paciente guardado correctamente» y qué pasa después | 5.2, 6.4 (V) |
| «Ya existe un paciente con ese documento de identidad.» (crear y editar) | 5.3, 6.4 (V) |
| «El número no corresponde al tipo de documento…» (mensaje de formato) | 5.1 (V). Su versión del servidor (códigos de comando): No aplica, la pantalla lo detecta antes |
| «Selecciona un tipo de documento válido.» | No aplica: el tipo es un desplegable obligatorio que la pantalla controla antes de enviar |
| `Db.Duplicado`, `Db.ReferenciaInvalida`, `Db.ValorInvalido` | No aplica a este recorrido: no se pudieron provocar (No verificado) |
| «Error al guardar el paciente» (fecha futura, contacto largo) | 5.3 (V) |
| Cuadro «Confirmar eliminación» y «¿Estás seguro de que deseas eliminar este paciente?» | 8.1 (V) |
| «Paciente eliminado correctamente» | 8.1 (V) |
| «No puedes eliminar este paciente porque tiene recetas, ventas u otros registros asociados.» | 8.2 (V con recetas; ventas, citas e inmunizaciones: No verificado) |
| «Error al eliminar el paciente», «Error al cargar el paciente» | 8.2 (No verificado) |
| Observaciones de `19e` (contacto 100, fecha futura, solo espacios) | 5.3 (V contacto y fecha; solo espacios: No verificado) |

### 16.3 Pacientes — carga masiva `/pacientes/carga-masiva` (19e §3.3)

| Elemento | Apartado |
|---|---|
| Título «Carga Masiva», migas, **Descargar plantilla** y zona de carga | 7.2 (V) |
| Plantilla de cinco columnas | 7.2 (V) |
| Vista previa («filas totales», «filas válidas», «filas con error», «Estado») | 7.2 (V) |
| Columnas de la vista previa y encabezado crudo `fields.tipoDocumento` | 7.2, 14 (V) |
| Mensajes de formato y de documento ya existente | 7.2 (V) |
| **Guardar válidos (N)** y «Se guardaron N registros correctamente» | 7.2 (V) |
| Límites de 5 MB y 5000 filas, archivo vacío, repetidos en el archivo | 7.2 (No verificado) |

### 16.4 Profesionales de salud — búsqueda, detalle y carga masiva (19e §4.1 a §4.3)

| Elemento | Apartado |
|---|---|
| Título, migas, tarjeta «Búsqueda de Profesionales de Salud» | 9 (V) |
| Filtros «Número de Registro» y «Nombre del Prescriptor» («contiene», acentos) | 9.2 (V) |
| Tabla: «Número de Registro» (enlace), «Nombre del Prescriptor», «Dirección de Clínica» («—»); orden por nombre; fila vacía «Resultados» | 9.2 (V) |
| Botones **Carga Masiva**, **Nuevo**, **Limpiar**, **Buscar**, **Exportar** | 9 a 9.4 (V) |
| Exportación: tres encabezados | 9.3 (V el contenido; nombre en disco: No verificado) |
| «Error al buscar profesionales de salud» | 9.3 (No verificado) |
| Campos «Número de Registro» (50), «Nombre del Prescriptor» (200), «Dirección de Clínica» (300) | 9.1 (V; el límite de 255 de la dirección: No verificado) |
| «Profesional de salud guardado correctamente» (crear y editar) | 9.1, 9.2 (V) |
| «Ya existe un profesional de salud con ese número de registro.» | 9.1 (V) |
| «Error al guardar el profesional de salud» | 9.1 (V con nombre de 151 caracteres) |
| Modos ver y editar; **Editar**, **Cancelar**, **Guardar**, **Eliminar** | 9.2, 9.5 (V) |
| Cuadro «Confirmar eliminación» y «¿Estás seguro de que deseas eliminar este profesional de salud?» | 9.5 (V) |
| «Profesional de salud eliminado correctamente» | 9.5 (V) |
| «No puedes eliminar este profesional de salud porque tiene recetas asociadas.» | 9.5 (V) |
| «Error al eliminar el profesional de salud», «Error al cargar el profesional de salud» | 9.5 (No verificado) |
| Carga masiva: plantilla de tres columnas, vista previa, «Dirección Clínica», mensajes, **Guardar válidos (1)** | 9.4 (V) |
| Datos del profesional (solo tres campos; sin especialidad ni tipo) | 3, 9.1 (V) |

### 16.5 Recetas médicas — búsqueda, detalle y escáner (19e §5.1 a §5.3)

| Elemento | Apartado |
|---|---|
| Título, migas, tarjeta «Búsqueda de Recetas Médicas» | 10 (V) |
| Filtros «Paciente» y «Médico» (elegir de la lista), «Fecha desde», «Fecha hasta» (inclusivas) | 11.1 (V) |
| Tabla: «Paciente» (enlace), «Médico», «Fecha de Expedición», «Repeticiones», «Indicaciones» («—»); sin número de receta; orden por fecha | 11.2 (V) |
| «Resultados (N)», fila «Sin resultados», paginador | 11.1, 11.2 (V; más de 10 filas: No verificado) |
| Botones **Escanear receta**, **Nuevo**, **Limpiar**, **Buscar**, **Exportar** | 10, 11, 12 (V) |
| Exportación: cinco encabezados, sin número de receta | 11.5 (V el contenido; nombre en disco: No verificado) |
| «Error al buscar recetas médicas» | 11.5 (No verificado) |
| Campos «Paciente», «Médico», «Fecha de Expedición», «Repeticiones Máximas», «Número de receta», «Indicaciones de Uso» | 10.1 (V) |
| Mensajes «Debes seleccionar un paciente / un médico», «La fecha de expedición es requerida» | 10.1, 10.3 (V) |
| Pista «Necesario para vender receta retenida o estupefacientes.» | 10.1 (V) |
| «Receta creada correctamente», «Receta actualizada correctamente» | 10.2, 11.3 (V) |
| Detalle «Detalle de Receta», **Editar**, **Eliminar**, **Cancelar**, **Guardar** | 11.3, 11.4 (V) |
| Cuadro «Eliminar receta» y «¿Estás seguro de que deseas eliminar esta receta médica? Esta acción no se puede deshacer.» | 11.4 (V) |
| «Receta eliminada correctamente» | 11.4 (V) |
| «No puedes eliminar esta receta porque ya se usó en ventas.» | 11.4 (No verificado: no hay ventas) |
| «Error al eliminar la receta», «Error al crear la receta», «Error al actualizar la receta», «Error al cargar la receta» | 10.3, 11.4 (No verificado) |
| Códigos de error mapeados (`Paciente.NotFound`, `ProfesionalSalud.NotFound`, `RecetaMedica.NotFound`, `RecetaMedica.FechaExpedicion`, «Revisa los datos ingresados de la receta.») | 10.3 (No verificado: no se pudieron provocar) |
| Escáner: pantalla, tarjeta, botones **Tomar foto**, **Galería**, **Limpiar**, **Interpretar receta**, vista previa, «Interpretando...» | 12.1, 12.2 (V; **Tomar foto** con cámara y arrastrar y soltar: No verificado) |
| Escáner: «Medicamentos de la receta (N)», «Detectado como», «DIGEMID», «Cantidad sugerida», «Confianza», «En stock: N unidades» | 12.3 (V con un análisis; «Media» y «Baja»: No verificado) |
| Escáner: «Sugerencias IA (N)» y «Sugerido por …» | 12.3 (V) |
| Escáner: «Sin stock», «Equivalentes terapéuticos:», «No hay stock del medicamento prescrito…», «No se detectaron medicamentos…», «Error al interpretar la receta. Intenta nuevamente.» | 12.4 (No verificado en la aplicación: se hizo un solo análisis) |
| Escáner: límite de 10 análisis por minuto, error 413 por más de 10 MB, imagen inválida | 12.4 (No verificado en la aplicación) |
| Modelo de la receta (solo cabecera, sin líneas ni vigencia) | Introducción, 3, 10.1 (V) |

### 16.6 Comportamientos comunes y reglas

| Elemento | Apartado |
|---|---|
| 19e §0: permisos de ruta, menú, directiva de permisos, avisos, fechas `dd/MM/aaaa`, modo ver/editar, qué pasa tras guardar | 2, 5 a 13 (V) |
| 19e §0: «Buscar» no vuelve a la página 1; autocompletar con el teclado | 15 (No verificado) |
| 19e §6: mapa de eliminar (pacientes, profesionales, recetas) | 8, 9.5, 11.4 (V pacientes y profesionales con recetas; receta en venta: No verificado). Lotes y Stock: capítulo 6 |
| `RN-PAC-001` a `RN-PAC-008`, `RN-PAC-010`, `RN-PAC-011` | 2, 5 a 8 (V las que se pueden provocar; `RN-PAC-003` y `RN-PAC-005` con solo espacios: No verificado) |
| `RN-PAC-009` (inexistente) | No aplica: el usuario no escribe identificadores |
| `RN-PAC-012` a `RN-PAC-017` (carga masiva) | 7.2 (V `RN-PAC-012`, `RN-PAC-013`, `RN-PAC-015` solo en parte y `RN-PAC-017`; `RN-PAC-014`, `RN-PAC-016`: No verificado) |
| `RN-PRO-001` a `RN-PRO-005`, `RN-PRO-007`, `RN-PRO-008` | 2, 9 (V; `RN-PRO-004` el límite de 255: No verificado) |
| `RN-PRO-006` (inexistente) | No aplica: el usuario no escribe identificadores |
| `RN-PRO-009` a `RN-PRO-012` (carga masiva) | 9.4 (V `RN-PRO-009`; `RN-PRO-010` a `RN-PRO-012`: No verificado en lo no mostrado) |
| `RN-REC-001` a `RN-REC-003`, `RN-REC-006` a `RN-REC-010`, `RN-REC-014`, `RN-REC-015`, `RN-REC-016`, `RN-REC-017` | 2, 10, 11 (V las que se pueden provocar; `RN-REC-009` el límite de 1000: No verificado) |
| `RN-REC-004`, `RN-REC-005`, `RN-REC-011`, `RN-REC-012` | No aplica: dependen de eliminar a otro usuario en paralelo o del identificador interno |
| `RN-REC-013` (receta usada en una venta) | 11.4 (No verificado: no hay ventas) |
| `RN-REC-018` a `RN-REC-029` (análisis con IA) | 12 (V con un análisis: `RN-REC-018`, `RN-REC-021` en general, `RN-REC-023`, `RN-REC-024`, `RN-REC-025`, `RN-REC-026`, `RN-REC-028` y `RN-REC-029`; los límites, `RN-REC-019`, `RN-REC-020`, `RN-REC-022`, `RN-REC-027`: No verificado) |
| `RN-REC-030` (tabla sin uso) | No aplica: no tiene pantalla |
| `RN-REC-031` (dispensaciones) y política «exigir registro de receta» | 3 (explicación); la comprobación en la venta es del capítulo 8 |

## 17. Fuentes

**Documentos consultados**

- [19e — inventario y atención](../../ola-1-fundamentos/19-inventario-de-pantallas/19e-inventario-y-atencion.md) (§0, §3, §4, §5, §6, §7).
- [01b — catálogos](../../ola-1-fundamentos/01-reglas-de-negocio/01b-catalogos.md) (`RN-PAC-001` a `RN-PAC-017`, `RN-PRO-001` a `RN-PRO-012`).
- [01c1 — inventario y recetas](../../ola-1-fundamentos/01-reglas-de-negocio/01c1-inventario-y-recetas.md) (`RN-REC-001` a `RN-REC-031`).
- [01c2 — ventas y comprobantes](../../ola-1-fundamentos/01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) (`RN-VEN-028`, `RN-VEN-029`, `RN-VEN-034`) y [glosario](../../ola-1-fundamentos/06-glosario.md) («Receta verificada»), solo para explicar la política de receta.
- [Capítulo 3](capitulo-03-empleados-roles-cuentas.md), [capítulo 5](capitulo-05-productos-y-proveedores.md) y [capítulo 6](capitulo-06-inventario.md) del manual (rol «Cajero», medicamentos y lotes).
- Textos de la interfaz: `sofia-web-spa/public/i18n/es.json`, `public/i18n/pacientes/es.json`, `public/i18n/profesionalesSalud/es.json`, `public/i18n/recetas-medicas/es.json` y `public/i18n/roles/es.json` (nombres de módulos y acciones).

**Recorridos hechos en la aplicación** (SPA `http://localhost:4200`, API `https://localhost:7300`; administrador `ana.manual` y cajero `carlos.cajero`)

1. Pacientes › Buscar sin datos; **Nuevo**: **Guardar** vacío, formato por tipo (DNI, pasaporte, RUC, carné de extranjería), cambio de tipo, fecha futura, contacto de 101 caracteres, documento repetido con otro tipo; creación de Luis y María y de un paciente de prueba.
2. Pacientes › Buscar: por tipo, por documento, por nombre (con y sin acento), por fechas (incluida «Desde» posterior a «Hasta»), Enter, **Limpiar**.
3. Pacientes › Ficha: ver, **Editar**, **Cancelar**, documento repetido al editar, guardado; **Eliminar** (con **Cancelar** y **Confirmar**) un paciente de prueba; **Eliminar** un paciente con recetas.
4. Pacientes › **Exportar** (lectura del archivo); **Carga Masiva**: plantilla, vista previa con 4 filas y guardado de 2.
5. Profesionales de Salud › Nuevo: **Guardar** vacío, nombre de 151 caracteres, número repetido; creación del Dr. Raúl y de uno de prueba; Buscar por número y por nombre; ficha, **Editar**, guardado; **Exportar**; Carga Masiva (plantilla, vista previa de 3 filas y guardado de 1); **Eliminar** el de prueba y el que tiene recetas.
6. Recetas Médicas › Buscar sin datos; Nuevo: **Guardar** vacío, fecha futura, repeticiones `-1`, listas de pacientes y médicos; creación de las tres recetas, de una de prueba para editar y eliminar y de una con número repetido y sin indicaciones (eliminada).
7. Recetas Médicas › Buscar por paciente (con y sin elegir de la lista), por fechas; ficha, **Editar**, **Cancelar**, guardado; **Eliminar** (con **Cancelar** y **Confirmar**); **Exportar** (lectura del archivo).
8. Recetas Médicas › **Escanear receta**: vista previa de una imagen inventada, **Limpiar**, nueva carga y **un análisis real** con **Interpretar receta**.
9. Cajero `carlos.cajero`: menú; Pacientes, Profesionales de Salud y Recetas Médicas (búsqueda y ficha); formulario de receta nueva con la lista de pacientes; entrada al escáner (sin analizar); direcciones `/pacientes/nueva` y `/pacientes/carga-masiva`; cierre de sesión y nuevo inicio de sesión como administrador.

