# Glosario de negocio y del dominio farmacéutico peruano en SOFIA

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-13 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de código (entidades, enums, handlers, scripts DbUp, componentes Angular y archivos `es.json`), contraste con los documentos validados de `ola-1-fundamentos` y 19 búsquedas web en fuentes oficiales peruanas (SUNAT, DIGEMID, gob.pe, El Peruano). No se ejecutó la aplicación ni se llamó a la API. `git status --short` estaba vacío en ambos repositorios al empezar: no hay trabajo sin commitear que afecte a este documento. |

## 1. Cómo leer este glosario

Este documento fija **un término preferido** para cada concepto: el que usa la interfaz en español. Cuando el código, la base de datos o la API usan otra palabra, aparece en la línea *También aparece como*. Lo leerán personas de negocio, quienes escriban el manual de usuario y desarrolladores nuevos.

**Cómo está armada cada entrada**

- **Definición.** Qué significa el término en el mundo farmacéutico o tributario peruano. Termina con una de estas tres marcas:
  - `Fuente oficial: <URL>`: la definición se apoya en una página oficial peruana encontrada por búsqueda web. Lo que la fuente no confirma se dice aparte.
  - `Definición general, no verificada en fuente oficial.`: la escribí sin poder confirmarla en una fuente oficial.
  - `Término propio de SOFIA (no normativo).`: es un concepto del sistema, no de una norma.
- **En SOFIA.** Qué significa exactamente en el sistema, con `archivo:línea`.
- **Dónde aparece.** Pantalla, etiqueta exacta, tabla, enum, endpoint y reglas de negocio relacionadas. Cada `RN-...` enlaza al documento donde está definida.

**Abreviaturas de ruta** (todas son rutas reales):

| Abreviatura | Ruta completa |
|---|---|
| `DOM/` | `sofia-apis/src/SOFIA.Domain/Entities/` |
| `ENUM/` | `sofia-apis/src/SOFIA.Domain/Enums/` |
| `APP/` | `sofia-apis/src/SOFIA.Application/` |
| `CFG/` | `sofia-apis/src/SOFIA.Infrastructure/Persistence/Configurations/` |
| `API/` | `sofia-apis/src/SOFIA.API/Controllers/` |
| `DBUP/` | `sofia-apis/src/SOFIA.DbUp/Scripts/` |
| `SPA/` | `sofia-web-spa/src/app/` |
| `I18N/` | `sofia-web-spa/public/i18n/` (las claves se escriben como `archivo:línea` y, entre paréntesis, la clave dentro del archivo) |

**Documentos a los que enlazan las reglas `RN-...`**

| Prefijos | Documento |
|---|---|
| `GEN`, `AUT`, `EMP`, `EPL`, `ROL`, `CTA`, `SER`, `SUC`, `IMP` | [01a Administración y seguridad](01-reglas-de-negocio/01a-administracion-y-seguridad.md) |
| `ASE`, `DIG`, `FOR`, `ING`, `JER`, `LAB`, `MED`, `PAC`, `PRE`, `PRO`, `PRV`, `UDM` | [01b Catálogos](01-reglas-de-negocio/01b-catalogos.md) |
| `LOT`, `STK`, `REC` | [01c1 Inventario y recetas](01-reglas-de-negocio/01c1-inventario-y-recetas.md) |
| `VEN`, `CMP` | [01c2 Ventas y comprobantes](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) |
| `CAJ`, `DEV`, `DAS` | [01c3 Caja, devoluciones y dashboard](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) |
| `TRA`, `DEL`, `MAG`, `SRV`, `CUA`, `AUD` | [01d Módulos solo API](01-reglas-de-negocio/01d-modulos-solo-api.md) |

Otros documentos a los que remite este glosario: [matriz de permisos](05-seguridad/05b-matriz-de-permisos.md), [integraciones](07-integraciones.md), [modelo de datos: maestros y seguridad](03-modelo-de-datos/03a-maestros-y-seguridad.md) y los inventarios de pantallas [19a](19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md), [19b](19-inventario-de-pantallas/19b-administracion.md) y [19c](19-inventario-de-pantallas/19c-catalogos-1.md).

**Estados de implementación que usa este glosario.** «Solo API (sin pantalla)»: existe el endpoint pero ninguna ruta del SPA lo muestra. «Parcial»: existe una parte. «Solo en el enum»: el valor existe en el código pero ninguna función lo usa. Los módulos sin pantalla enrutada son Transferencias, Delivery, Magistrales, Servicios e inmunizaciones, Cuarentena y actas DIGEMID, listado y detalle de Devoluciones y Auditoría (comprobado contra `docs/_inventario/rutas-spa.csv`). El envío de comprobantes a SUNAT no existe.

**Aviso sobre los documentos de reglas.** Algunos documentos de reglas se escribieron sobre commits anteriores. Cuando el código actual difiere, manda el código y se avisa en la entrada (por ejemplo, el registro de precios de proveedor: ver «Precio de compra»).

## 2. Términos que el sistema nombra de más de una forma

| Concepto | La interfaz (es.json) dice | El código dice | La base de datos dice | Término preferido en la documentación |
|---|---|---|---|---|
| Aseguradora / seguro | «Aseguradoras» en el menú, «Aseguradoras Médicas» como título de pantalla (`I18N/aseguradoras/es.json:2`), «Cobertura de Seguro (opcional)» en el POS | `AseguradoraMedica`, módulo de permisos `Seguros`, `VentaReclamoSeguro` | `Aseguradoras_Medicas`, `Ventas_Reclamos_Seguro` | **Aseguradora** (la entidad) y **cobertura de seguro** (lo que se aplica a una venta). El módulo de permisos se llama `Seguros` pero se rotula «Aseguradoras» |
| Medicamento / producto | «Medicamentos» (menú y pantalla); «Producto» en lotes, jerarquías y presentaciones; «Medicamento / Insumo» en una columna del POS | `Medicamento`, `ProductoId`, `Producto` | `Medicamentos` (clave `Producto_ID`) | **Medicamento** para el maestro; **producto** solo cuando la pantalla lo rotula así. Son la misma fila de `Medicamentos` |
| Paciente / cliente | «Pacientes» (menú); «Cliente» en el POS y en la tabla de ventas; «Datos del Cliente» | `PacienteCliente`, `ClienteId`, `Cliente` | `Pacientes_Clientes` (clave `Cliente_ID`) | **Paciente** para el maestro; **cliente** para quien compra en una venta. Es la misma entidad |
| Sesión de caja / caja | «Sesiones de Caja» (menú); «Caja Abierta», «Aperturar Caja», «Cerrar Caja» en el POS | `PosSesionCaja`, `EstadoSesion`, módulo de permisos `POS` | `POS_Sesiones_Caja` | **Sesión de caja** para el registro; **caja** en frases como «abrir la caja». Un cajero solo puede tener una abierta |
| Venta en espera / pendiente | «Ventas en espera» (dashboard), «Cobrar Venta en Espera», «Guardar y Atender Siguiente»; «Pendiente» como etiqueta del estado | `EstadoVenta.Pendiente` | `Ventas_Cabecera.Estado = 'Pendiente'` | **Venta en espera** (el concepto); el valor guardado es `Pendiente` |
| Jerarquía / UoM | «Jerarquías de Unidades» (menú vigente); «Jerarquías UoM» solo en un archivo de textos sin pantalla | `JerarquiaUoM`, módulo `JerarquiasUoM` | `Jerarquia_UoM` | **Jerarquía de unidades** |
| Unidad de medida / unidad base / unidad de venta | «Unidades de Medida»; «Unidad Base» (medicamento); «Unidad de Venta» (presentación) | `UnidadMedida`, `UnidadBaseId`, `UnidadVentaId` | `Unidades_Medida` | **Unidad de medida** (catálogo); **unidad base** y **unidad de venta** son los dos usos que le dan el medicamento y la presentación |
| Sucursal / sede | «Sucursales»; «Sede» en «Sucursal de Destino (Sede Proveedora)» y «Dirección de la sede principal» | `Sucursal`, `SucursalId` | `Sucursales` | **Sucursal**; **sede principal** solo para la primera sucursal creada al registrar la empresa |
| Lote / lote de inventario | «Lotes»; «Número de Lote»; «Lote de Inventario» en un archivo sin pantalla | `LoteInventario`, `NumeroLoteMfr` | `Lotes_Inventario` | **Lote** |
| Stock / cantidad física / existencias | «Stock por Sucursal», «Cantidad Física» | `InventarioSucursal`, `CantidadFisica`, `LotesEnSucursal` | `Inventario_Sucursal` | **Stock** (una fila por lote y sucursal); **cantidad física** es el número guardado en esa fila |
| Caducidad / vencimiento | «Fecha de Caducidad», «Caducidad Desde»; en el dashboard «Lotes vencidos», «Lotes que vencen» | `FechaCaducidad`, `EstaVencido` | `Fecha_Caducidad` | **Fecha de caducidad** (el dato) y **lote vencido** (el estado) |
| Comprobante / boleta / venta | «Comprobante», «Nº Comprobante», «Código de Venta» | `SunatComprobanteEmitido`, `TransaccionId` | `SUNAT_Comprobantes_Emitidos`, `Ventas_Cabecera.Transaccion_ID` | **Venta** es la operación; **comprobante** es el documento que la acredita (hoy solo boleta) |
| Serie / prefijo de serie | «Series Fiscales»; «Prefijo de Serie» (campo); «Serie» (columna) | `SunatSerieFiscal`, `PrefijoSerie` | `SUNAT_Series_Fiscales.Prefijo_Serie` | **Serie** (el registro) y **prefijo** (sus 4 caracteres) |
| Correlativo | «Correlativo Actual» | `CorrelativoActual`, `NumeroCorrelativo` | `Correlativo_Actual`, `Numero_Correlativo` | **Correlativo** |
| Devolución / reembolso / nota de crédito | «Devolver», «Procesar devolución», «Reembolso estimado», «Devoluciones / Notas de crédito» | `DevolucionCabecera`, `MontoReembolsado`, `NotaCredito` | `Devoluciones_Cabecera` | **Devolución** (la operación), **reembolso** (el dinero que sale de la caja), **nota de crédito** (el comprobante) |
| Cuarentena / retenido / aislamiento | «Lotes en cuarentena», «Retenidos: no se pueden vender» | `DigemidInventarioCuarentena`, `EstadoResolucion = Retenido`, `AislarLoteCuarentena` | `DIGEMID_Inventario_Cuarentena` | **Cuarentena** (el registro) y **retenido** (el estado que bloquea la venta) |
| Regente / químico farmacéutico / director técnico | No aparece en ningún texto de la interfaz | `RegenteResponsableId`, `QuimicoPreparadorId` | `Regente_Responsable_ID`, `Quimico_Preparador_ID` | **Regente** solo en el acta de destrucción; ver la entrada «Regente» |
| Licencia (de sucursal) / licencia profesional (de empleado) | «Nº de Licencia» (sucursal); «Licencia Profesional» (empleado) | `Numero_Licencia`, `Licencia_Prof` | `Numero_Licencia`, `Licencia_Prof` | **Número de licencia** (sucursal) y **licencia profesional** (empleado): son datos distintos |
| Médico / prescriptor / profesional de salud | «Profesionales de Salud» (menú), «Médico» (receta), «Nombre del Prescriptor» | `ProfesionalSalud`, `MedicoId`, `NombrePrescriptor` | `Profesionales_Salud` | **Profesional de salud** para el maestro; **médico** es el rótulo del campo en la receta |
| Rol / rol Admin / administrador | «Roles & Permisos»; «Usuario Administrador» al registrar la empresa; «rol Admin» en mensajes | `Rol`, `Rol.AdminRoleName = "Admin"` | `Seguridad_Roles` | **Rol**; el rol de nombre `Admin` es **rol Admin** |
| Empresa / tenant / compañía | «Empresa», «Empresas» | `Empresa`, `TenantId` | `Empresas`, columna `TenantId` | **Empresa** (el cliente de SOFIA); **tenant** solo al hablar del aislamiento técnico |
| Estado de cita / estado de producción | «Estado de Cita» (sin pantalla) | `EstadoCita`, `EstadoProduccion` | `Estado_Cita`, `Estado_Produccion` | Sin término preferido: son textos libres de módulos sin pantalla |

## 3. SUNAT y comprobantes

Mapa de la familia: SOFIA emite hoy **solo boletas** (al cobrar una venta) y **notas de crédito** (al procesar una devolución). Factura, ticket, nota de débito y proforma existen como valores del enum `TipoComprobante`, pero ninguna función los emite. Nada se envía a SUNAT: los comprobantes quedan guardados con estado `NoEnviado`.

### Base imponible (operación gravada)
*También aparece como:* «Subtotal (Op. Gravada)», `MontoGravadoIgv`, `Monto_Gravado_IGV`, `Gravado`.

**Definición.** Es el importe sobre el que se calcula el IGV en una operación gravada. En la venta de bienes, la base es el valor de venta. Fuente oficial: https://orientacion.sunat.gob.pe/3053-concepto-tasa-y-operaciones-gravadas-igv-empresas y https://orientacion.sunat.gob.pe/3109-05-calculo-del-impuesto (confirmado por el resumen de la búsqueda; no se abrió el texto completo).

**En SOFIA.** Los precios de SOFIA incluyen IGV. La base imponible de una boleta o nota de crédito es `total / 1,18` redondeada a 2 decimales (hacia fuera del cero) y el IGV es el resto, de modo que ambos suman siempre el total (`APP/Ventas/Common/VentaComprobanteGenerator.cs:93-98`). El monto exonerado se guarda siempre en 0 (`VentaComprobanteGenerator.cs:53`): SOFIA no maneja operaciones exoneradas ni inafectas.

**Dónde aparece.** POS, resumen de venta y comprobante: «Subtotal (Op. Gravada):» (`I18N/pos/es.json:200`, `checkout.subtotalLabel`). Tabla `SUNAT_Comprobantes_Emitidos`, columnas de monto gravado, exonerado, IGV y total (ver [modelo de datos](03-modelo-de-datos/03a-maestros-y-seguridad.md) para el esquema general). Reglas: [RN-CMP-006](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CMP-008](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Boleta
*También aparece como:* boleta de venta, `TipoComprobante.Boleta`, descripción `"03"`.

**Definición.** Comprobante de pago que se emite a consumidores finales; no permite ejercer crédito fiscal ni sustentar gasto o costo tributario. Fuente oficial: https://cpe.sunat.gob.pe/tipos_de_comprobantes/boleta

**En SOFIA.** Es el único comprobante que emite el POS. Al cobrar, el sistema busca la serie `Boleta` activa de la sucursal y, si falta, rechaza el cobro (`APP/Ventas/Common/VentaComprobanteGenerator.cs:23-28`). El número visible es `{prefijo}-{correlativo con 8 dígitos}`, por ejemplo `B001-00000042` (`VentaComprobanteGenerator.cs:68`). Código `"03"` en `ENUM/TipoComprobante.cs:9`. No hay ninguna regla que pida datos del cliente por monto: sin cliente se emite a nombre de «CLIENTE EVENTUAL» (ver «Cliente eventual»).

**Dónde aparece.** POS, diálogo «Venta Procesada Exitosamente»: «Nº Comprobante:» (`I18N/pos/es.json:241`, `comprobante.boletaNumero`). Pantalla Series Fiscales: tipo «Boleta» (`I18N/series-fiscales/es.json:50`). Tabla `SUNAT_Series_Fiscales`, columna `Tipo_Comprobante` = `'03'`. Reglas: [RN-CMP-001](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CMP-003](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CMP-004](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CMP-005](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-059](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Carné de extranjería
*También aparece como:* CE, código `"4"` del catálogo 06 de SUNAT.

**Definición.** Documento de identidad de extranjeros residentes; es el código 4 del catálogo de tipos de documento de identidad de SUNAT. Fuente oficial: https://www.sunat.gob.pe/legislacion/superin/2021/anexo-112-2021.pdf (tabla de tipos de documento de identidad).

**En SOFIA.** Valor `CarneExtranjeria = "4"` (`ENUM/TipoDocumentoIdentidad.cs:10`). El número admite de 1 a 12 letras o números (`TipoDocumentoIdentidad.cs:56-57`). En cargas masivas se acepta el código o la abreviatura `CE` (`TipoDocumentoIdentidad.cs:19-23`).

**Dónde aparece.** Selector «Tipo de Documento» de Pacientes; texto «Carné de extranjería» (`I18N/es.json:272`, `tipos-documento.4`). Columna `Pacientes_Clientes.Tipo_Doc_Identidad` (CHECK que acepta `'1','4','6','7'`, `DBUP/0023_Pacientes_Tipo_Documento.sql:7`). Reglas: [RN-PAC-002](01-reglas-de-negocio/01b-catalogos.md), [RN-PAC-004](01-reglas-de-negocio/01b-catalogos.md), [RN-PAC-013](01-reglas-de-negocio/01b-catalogos.md).

### Cliente eventual
*También aparece como:* «Cliente Eventual (Público General)», «CLIENTE EVENTUAL», «Sin documento», tipo de documento `"-"` («Varios»).

**Definición.** Cliente que compra sin identificarse. Término propio de SOFIA (no normativo).

**En SOFIA.** Cuando la venta no tiene paciente (o el paciente tiene un documento de más de 20 caracteres), la boleta se emite con tipo de documento `"-"`, número `00000000` y nombre `CLIENTE EVENTUAL` (`APP/Ventas/Common/VentaComprobanteGenerator.cs:11-12`, `:82-91`). El valor `"-"` no está entre los tipos que un paciente puede tener (`ENUM/TipoDocumentoIdentidad.cs:8` frente a `:14`).

**Dónde aparece.** POS, «Datos del Cliente»: «Cliente Eventual (Público General)» (`I18N/pos/es.json:168`, `checkout.clienteEventual`). Texto «Sin documento» (`I18N/es.json:275`, clave `tipos-documento` con la llave `-`). Reglas: [RN-VEN-025](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CMP-005](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Comprobante de pago
*También aparece como:* comprobante, comprobante emitido, `SunatComprobanteEmitido`, `SUNAT_Comprobantes_Emitidos`.

**Definición.** Documento que sustenta una transferencia de bienes o una prestación de servicios. SUNAT distingue, entre otros, factura, boleta de venta, ticket o cinta de máquina registradora, nota de crédito y nota de débito. Fuente oficial: https://cpe.sunat.gob.pe/informacion_general/tipos_comprobantes_pago

**En SOFIA.** Cada fila de `SUNAT_Comprobantes_Emitidos` guarda venta, serie, correlativo, fecha de emisión, tipo y número de documento del cliente, razón social del cliente, los cuatro montos y el estado (`DOM/SunatComprobanteEmitido.cs:10-25`). Los campos de firma, XML, constancia y URL de verificación quedan vacíos porque no hay envío (`VentaComprobanteGenerator.cs:49-58`). El emisor no se guarda: se lee de la empresa y la sucursal al emitir (ver «Emisor»).

**Dónde aparece.** POS: diálogo de comprobante y detalle de venta (sección «Comprobante», `I18N/pos/es.json:83`, `ventasDetail.comprobanteTitle`). Reglas: [RN-CMP-001](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) a [RN-CMP-015](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md). Integración: [07-integraciones §5](07-integraciones.md).

### Correlativo
*También aparece como:* «Correlativo Actual», `CorrelativoActual`, `NumeroCorrelativo`.

**Definición.** Número que, junto con la serie, identifica a un comprobante. En los comprobantes electrónicos la serie tiene 4 posiciones alfanuméricas y el correlativo hasta 8 posiciones, empezando en 1. Fuente oficial: https://cpe.sunat.gob.pe/informacion_general/tipos_comprobantes_pago

**En SOFIA.** `CorrelativoActual` es el **último número emitido** de la serie; el siguiente comprobante usa ese valor más 1 (`I18N/series-fiscales/es.json:20`, `correlativo-hint`). El incremento se hace con una sola instrucción atómica (`IncrementarCorrelativoSunatAsync`, usada en `VentaComprobanteGenerator.cs:44`). Se muestra con 8 dígitos (`:D8`). Una serie que ya emitió comprobantes no admite cambiar su correlativo (solo su estado).

**Dónde aparece.** Series Fiscales: campo «Correlativo Actual» (`I18N/series-fiscales/es.json:36`, `fields.correlativoActual`). Columna `SUNAT_Series_Fiscales.Correlativo_Actual`. Reglas: [RN-SER-005](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-SER-010](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-SER-013](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-CMP-004](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CMP-009](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### DNI
*También aparece como:* Documento Nacional de Identidad, código `"1"` del catálogo 06 de SUNAT.

**Definición.** Documento de identidad de los peruanos; es el código 1 del catálogo de tipos de documento de identidad de SUNAT, con 8 dígitos. Fuente oficial: https://www.sunat.gob.pe/legislacion/superin/2021/anexo-112-2021.pdf

**En SOFIA.** Valor `Dni = "1"` (`ENUM/TipoDocumentoIdentidad.cs:9`); el número debe ser exactamente 8 dígitos (`:50`). Abreviatura `DNI` en cargas masivas y exportaciones.

**Dónde aparece.** Pacientes: «Tipo de Documento» y «Documento de Identidad»; texto «DNI» (`I18N/es.json:271`). Mensaje de formato: `I18N/pacientes/es.json:89` (`detail.documento-formato`). Reglas: [RN-PAC-002](01-reglas-de-negocio/01b-catalogos.md) a [RN-PAC-004](01-reglas-de-negocio/01b-catalogos.md).

### Emisor
*También aparece como:* razón social del emisor, RUC del emisor, `EmisorComprobante`.

**Definición.** Contribuyente que emite el comprobante de pago. Definición general, no verificada en fuente oficial.

**En SOFIA.** No es una tabla: se arma al emitir con el nombre de la empresa, su RUC y la dirección física de la sucursal que cobra (`APP/Ventas/Common/VentaComprobanteGenerator.cs:31-39`). Si la empresa no tiene RUC, no se puede emitir ni cobrar (`:18-21`, error `Venta.Empresa.SinRuc`).

**Dónde aparece.** POS: «RUC:» en el comprobante (`I18N/pos/es.json:245`, `comprobante.rucLabel`); mensaje «Tu empresa no tiene RUC registrado…» (`:355`, `messages.empresaSinRuc`). Registro de empresa: «Dirección de la sede principal», con la ayuda «Aparecerá en el encabezado de tus boletas» (`I18N/empresa/es.json:12-14`). Reglas: [RN-CMP-002](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CMP-011](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Estado SUNAT del comprobante
*También aparece como:* «Estado SUNAT:», `EstadoComprobanteSunat`, `Estado_Aceptacion`, «estado de aceptación».

**Definición.** Situación del comprobante electrónico frente a SUNAT (por ejemplo, aceptado o rechazado tras su validación). Definición general, no verificada en fuente oficial.

**En SOFIA.** Es un texto de hasta 20 caracteres (`ENUM/EstadoComprobanteSunat.cs:3-10`). Valores y cómo los muestra la interfaz (`I18N/pos/es.json:357-363`, `estadoComprobante.*`):

| Valor guardado | Texto en pantalla | Quién lo asigna |
|---|---|---|
| `NoEnviado` | «Pendiente de envío a SUNAT» | Toda boleta y nota de crédito al nacer (`VentaComprobanteGenerator.cs:55`; `ProcesarDevolucionCommandHandler.cs:206`) |
| `Pendiente` | «Pendiente de envío a SUNAT» | Solo el DEFAULT del script (`DBUP/0001_Initial_Schema.sql:573`); el código no lo asigna |
| `Aceptado` | «Aceptado por SUNAT» | Nadie: reservado para la integración con un OSE o PSE (comentario en `ENUM/EstadoComprobanteSunat.cs:3`) |
| `Rechazado` | «Rechazado por SUNAT» | Nadie: reservado |
| `Anulado` | «Anulado» | La anulación de la venta (`DOM/SunatComprobanteEmitido.cs:157-162`) |

**Dónde aparece.** POS: «Estado SUNAT:» en el diálogo y en el comprobante impreso (`I18N/pos/es.json:242`, `comprobante.estadoSunat`); subtítulo «Comprobante registrado. Pendiente de envío a SUNAT.» (`:240`). Reglas: [RN-CMP-007](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CMP-012](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md). Enum: `EstadoComprobanteSunat`.

### Factura
*También aparece como:* `TipoComprobante.Factura`, descripción `"01"`.

**Definición.** Comprobante de pago que se emite solo a quien tiene RUC y que sirve para sustentar costo o gasto y crédito fiscal. Su serie empieza con la letra F. Fuente oficial: https://cpe.sunat.gob.pe/informacion_general/tipos_comprobantes_pago

**En SOFIA.** Parcial. Se puede **configurar una serie** de factura (prefijo que empieza con F, `DOM/SunatSerieFiscal.cs:41`) y las devoluciones aceptan una factura como documento de origen (`APP/Devoluciones/Commands/ProcesarDevolucion/ProcesarDevolucionCommandHandler.cs:123`), pero **ninguna función emite facturas**: el POS solo llama a `EmitirBoletaAsync`.

**Dónde aparece.** Series Fiscales: tipo «Factura» (`I18N/series-fiscales/es.json:51`). Sugerencia: «Boleta empieza con B, Factura con F, Nota de Crédito con B o F.» (`:19`, `detail.prefijo-hint`). Reglas: [RN-SER-003](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-SER-004](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### IGV
*También aparece como:* «IGV (18%):», `MontoTotalIgv`, `Monto_Total_IGV`.

**Definición.** Impuesto General a las Ventas. La orientación de SUNAT consultada presenta una tasa total de 18%, compuesta por el IGV y el Impuesto de Promoción Municipal. Fuente oficial: https://orientacion.sunat.gob.pe/3053-concepto-tasa-y-operaciones-gravadas-igv-empresas (el desglose lo informa el resumen de la búsqueda; no se abrió la página completa).

**En SOFIA.** La tasa **no es configurable**: el código divide entre `1,18` (`APP/Ventas/Common/VentaComprobanteGenerator.cs:96`) y la interfaz tiene el rótulo fijo «IGV (18%):». Los precios del catálogo ya incluyen el impuesto. SOFIA muestra y guarda un solo importe de IGV por comprobante.

**Dónde aparece.** POS, resumen y comprobante: «IGV (18%):» (`I18N/pos/es.json:201`, `checkout.igvLabel`). Tabla `SUNAT_Comprobantes_Emitidos`, columna del IGV total. Reglas: [RN-CMP-006](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-DEV-021](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) (nota de crédito).

### Motivo de la nota de crédito
*También aparece como:* «Motivo SUNAT», `MotivoSunatCatalogo`, `Motivo_Sunat_Catalogo`, catálogo 09 de SUNAT.

**Definición.** Código del catálogo 09 de SUNAT que indica por qué se emite una nota de crédito electrónica. En ese catálogo, 06 es devolución total y 07 devolución por ítem. Fuente oficial: https://www.sunat.gob.pe/legislacion/superin/2020/anexo3-193-2020.pdf (la búsqueda confirmó los códigos 06 y 07 pero no su redacción exacta).

**En SOFIA.** El backend exige solo que el código tenga exactamente 2 caracteres (`DOM/DevolucionCabecera.cs:46-52`). El SPA ofrece dos: `06` «Devolución total» y `07` «Devolución por ítem» (`SPA/pages/pos/dialogs/devolver-venta-dialog/devolver-venta-dialog.component.ts:18-19`); por defecto propone `07` y cambia a `06` cuando se devuelve todo (`:104`). El sustento descriptivo es obligatorio y admite hasta 255 caracteres.

**Dónde aparece.** POS, diálogo «Procesar devolución»: «Motivo SUNAT» y las opciones (`I18N/pos/es.json:119-123`, `devolucion.motivoLabel`, `devolucion.motivo`); «Sustento» (`:125`). Tabla `Devoluciones_Cabecera`. Reglas: [RN-DEV-014](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-DEV-015](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Nota de crédito
*También aparece como:* `TipoComprobante.NotaCredito`, descripción `"07"`.

**Definición.** Comprobante que modifica uno o más comprobantes emitidos, reduciendo o anulando lo facturado; puede referirse a facturas o boletas. Fuente oficial: https://cpe.sunat.gob.pe/informacion_general/tipos_comprobantes_pago

**En SOFIA.** Se emite al **procesar una devolución**. Acredita la boleta (o factura) original de la venta, por el importe de las líneas devueltas, con el mismo cliente y a través de la serie `NotaCredito` activa de la sucursal (`APP/Devoluciones/Commands/ProcesarDevolucion/ProcesarDevolucionCommandHandler.cs:117-138`, `:197-216`). Nace con estado `NoEnviado`. Sin serie activa de nota de crédito, la devolución se rechaza (error `Devolucion.SerieNotaCredito.NoConfigurada`). El prefijo de la serie puede empezar con B o con F (`DOM/SunatSerieFiscal.cs:43`).

**Dónde aparece.** POS: tras devolver, «Devolución procesada y nota de crédito emitida.» (`I18N/pos/es.json:130`, `devolucion.success`); sección «Devoluciones / Notas de crédito» en el detalle de venta (`:132`). Series Fiscales: tipo «Nota de Crédito» (`I18N/series-fiscales/es.json:52`). Reglas: [RN-DEV-020](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-DEV-021](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-DEV-022](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-DEV-037](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CMP-015](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Nota de débito
*También aparece como:* `TipoComprobante.NotaDebito`, descripción `"08"`.

**Definición.** Comprobante que se emite para recuperar gastos o costos en que incurrió el vendedor después de emitir el comprobante original. Fuente oficial: https://cpe.sunat.gob.pe/tipos_de_comprobantes/nota_de_debito

**En SOFIA.** **Solo en el enum.** El valor existe (`ENUM/TipoComprobante.cs:11`) y la base lo admite (`DBUP/0001_Initial_Schema.sql:555`), pero no es un tipo configurable (`DOM/SunatSerieFiscal.cs:13-14`), no figura en los textos de Series Fiscales y nada lo emite. El único uso es aceptar cualquier prefijo en `EsPrefijoValido` (`:44`).

**Dónde aparece.** Solo en el código y en el modelo del SPA (`SPA/pages/series-fiscales/models/serie-fiscal.model.ts:14`). Sin texto de interfaz.

### OSE y PSE (envío electrónico a SUNAT)
*También aparece como:* OSE, PSE, «integración con SUNAT», «Pendiente de envío a SUNAT».

**Definición.** El Operador de Servicios Electrónicos (OSE) valida los comprobantes que le envía el emisor y remite la información validada a SUNAT; el Proveedor de Servicios Electrónicos (PSE) presta al emisor la plataforma para emitir comprobantes en su nombre, firmando con su propio certificado. Fuente oficial: https://www.gob.pe/26777-operador-de-servicios-electronicos-ose y https://www.gob.pe/26399-proveedores-de-servicios-electronicos-pse

**En SOFIA.** **No existe.** SOFIA no firma, no genera XML, no envía y no recibe constancias: hash, rutas de XML y CDR y URL de verificación quedan vacíos «hasta que se integre un OSE/PSE» (`APP/Ventas/Common/VentaComprobanteGenerator.cs:49`), y `Aceptado` y `Rechazado` están reservados (`ENUM/EstadoComprobanteSunat.cs:3`). Anular una venta solo cambia el estado local; no se genera comunicación de baja.

**Dónde aparece.** Solo como aviso: «Comprobante registrado. Pendiente de envío a SUNAT.» (`I18N/pos/es.json:240`). Integración: [07-integraciones §5](07-integraciones.md). Reglas: [RN-CMP-007](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CMP-012](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Proforma
*También aparece como:* `TipoComprobante.Proforma`, descripción `"PR"`.

**Definición.** Documento comercial previo a la venta que informa precios y condiciones. No es un comprobante de pago de SUNAT. Definición general, no verificada en fuente oficial.

**En SOFIA.** Parcial: se puede **configurar una serie** de proforma (`DOM/SunatSerieFiscal.cs:14`; el CHECK de la base admite `'PR'`, `DBUP/0018_SeriesFiscales_UniqueNumbering.sql:7`), pero **ninguna función emite proformas**.

**Dónde aparece.** Series Fiscales: tipo «Proforma» (`I18N/series-fiscales/es.json:53`). Reglas: [RN-SER-003](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### RUC
*También aparece como:* Registro Único de Contribuyentes, «RUC (11 dígitos, opcional)», «RUC / Tax ID», tipo de documento `"6"`, `Ruc` (objeto de valor).

**Definición.** Número de 11 dígitos con el que SUNAT identifica a cada contribuyente. Empieza con 10 (persona natural con DNI), 15 (persona natural extranjera) o 20 (persona jurídica). Fuente oficial: https://orientacion.sunat.gob.pe/01-inscripcion-al-ruc-empresas

**En SOFIA.** Tres usos: (1) el **RUC de la empresa** (`DOM/Empresa.cs:19`), obligatorio al registrar (11 dígitos), único en todo el sistema y necesario para emitir comprobantes; solo se comprueba que sean 11 dígitos, no el dígito verificador; (2) el **tipo de documento `"6"`** de un paciente, con exactamente 11 dígitos (`ENUM/TipoDocumentoIdentidad.cs:11`, `:53`); (3) el **identificador tributario del proveedor** (`TaxId`, rotulado «RUC / Tax ID», hasta 50 caracteres sin formato obligatorio).

**Dónde aparece.** Registro de empresa y edición: «RUC» en el registro (`I18N/empresa/es.json:11`) y «RUC (11 dígitos, opcional)» en la edición (`:64`, `:114`); error «Ya existe una empresa registrada con este RUC» (`:88`). Proveedores: «RUC / Tax ID» (`I18N/proveedores/es.json:7`). Reglas: [RN-EMP-003](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-EMP-005](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-PRV-003](01-reglas-de-negocio/01b-catalogos.md), [RN-PAC-004](01-reglas-de-negocio/01b-catalogos.md).

### Serie (prefijo)
*También aparece como:* «Series Fiscales», «Prefijo de Serie», `SunatSerieFiscal`, `PrefijoSerie`.

**Definición.** Parte alfanumérica de 4 posiciones de la numeración de un comprobante electrónico; la primera letra depende del tipo (B para boletas, F para facturas). Fuente oficial: https://cpe.sunat.gob.pe/informacion_general/tipos_comprobantes_pago y https://www.sunat.gob.pe/legislacion/superin/2022/anexo-123-2022.pdf

**En SOFIA.** Una serie pertenece a una **sucursal** y a un **tipo de comprobante** (solo Boleta, Factura, Nota de Crédito o Proforma). El prefijo tiene 4 letras mayúsculas o números y la letra inicial que exige el tipo (`DOM/SunatSerieFiscal.cs:31-47`). Estado `Activa` o `Inactiva`; una sucursal puede tener una sola serie activa por tipo. Una serie que ya emitió comprobantes no se puede eliminar ni cambiar de numeración: solo cambia de estado.

**Dónde aparece.** Menú «SUNAT» > «Series Fiscales» (`I18N/es.json:27`, `nav.seriesFiscales`). Campos: «Sucursal», «Tipo de Comprobante», «Prefijo de Serie», «Correlativo Actual», «Estado» (`I18N/series-fiscales/es.json:33-38`); estados «Activa» e «Inactiva» (`:55-58`). Inventario de pantallas: [19b](19-inventario-de-pantallas/19b-administracion.md). Reglas: [RN-SER-001](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-SER-014](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Ticket
*También aparece como:* `TipoComprobante.Ticket`, descripción `"00"`.

**Definición.** Comprobante de pago impreso por una máquina registradora, con numeración correlativa autogenerada por la máquina. Fuente oficial: https://cpe.sunat.gob.pe/informacion_general/tipos_comprobantes_pago

**En SOFIA.** **Solo en el enum** (`ENUM/TipoComprobante.cs:7`); no es configurable ni se emite. No se debe confundir con el recibo impreso del POS (ver «Comprobante impreso»), que es la representación impresa de la boleta.

**Dónde aparece.** Solo en el código y en la restricción de la base de datos (`DBUP/0001_Initial_Schema.sql:555`). Sin texto de interfaz.

### Tipo de comprobante
*También aparece como:* `TipoComprobante`, `Tipo_Comprobante`.

**Definición.** Clase de comprobante de pago (factura, boleta, etc.). Fuente oficial: https://cpe.sunat.gob.pe/informacion_general/tipos_comprobantes_pago

**En SOFIA.** Enum con seis valores; se guarda el código de 2 caracteres de la descripción (`CFG/SunatSerieFiscalConfiguration.cs:27`), y la API lo entrega como número:

| Valor del enum | Código guardado | Texto en la interfaz | Estado |
|---|---|---|---|
| `Ticket` | `00` | Sin texto | Solo en el enum |
| `Factura` | `01` | «Factura» | Serie configurable; no se emite |
| `Boleta` | `03` | «Boleta» | Se emite en cada cobro |
| `NotaCredito` | `07` | «Nota de Crédito» | Se emite en cada devolución |
| `NotaDebito` | `08` | Sin texto | Solo en el enum |
| `Proforma` | `PR` | «Proforma» | Serie configurable; no se emite |

(`ENUM/TipoComprobante.cs:7-12`; textos en `I18N/series-fiscales/es.json:49-54`; tipos configurables en `DOM/SunatSerieFiscal.cs:13-14`.)

**Dónde aparece.** Series Fiscales: filtro y campo «Tipo de Comprobante» (`I18N/series-fiscales/es.json:34`). Reglas: [RN-SER-003](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Tipo de documento de identidad
*También aparece como:* «Tipo de Documento», «Tipo Doc.», `TipoDocumentoIdentidad`, `Tipo_Doc_Identidad`, catálogo 06 de SUNAT.

**Definición.** Clase de documento con el que se identifica una persona: el catálogo 06 de SUNAT asigna 1 al DNI, 4 al carné de extranjería, 6 al RUC y 7 al pasaporte (y otros códigos que SOFIA no usa). Fuente oficial: https://www.sunat.gob.pe/legislacion/superin/2021/anexo-112-2021.pdf

**En SOFIA.** Se guarda como texto con el código de SUNAT (`ENUM/TipoDocumentoIdentidad.cs:5-14`):

| Código | Texto en la interfaz | Abreviatura (cargas masivas) | Formato del número | Puede tenerlo un paciente |
|---|---|---|---|---|
| `1` | «DNI» | `DNI` | 8 dígitos | Sí |
| `4` | «Carné de extranjería» | `CE` | 1 a 12 letras o números | Sí |
| `6` | «RUC» | `RUC` | 11 dígitos | Sí |
| `7` | «Pasaporte» | `PASAPORTE` | 1 a 20 letras o números | Sí |
| `-` | «Sin documento» | — | — | No: lo usa solo el comprobante del cliente eventual |

(`I18N/es.json:270-276`, `tipos-documento`.) Los pacientes anteriores a la columna pueden tener el tipo vacío y se facturan como cliente eventual con su propio número (`VentaComprobanteGenerator.cs:81-91`).

**Dónde aparece.** Pacientes: «Tipo de Documento» (`I18N/pacientes/es.json:93`). Tabla `Pacientes_Clientes` y `SUNAT_Comprobantes_Emitidos.Tipo_Doc_Identidad_Cliente`. Reglas: [RN-PAC-002](01-reglas-de-negocio/01b-catalogos.md) a [RN-PAC-004](01-reglas-de-negocio/01b-catalogos.md), [RN-PAC-013](01-reglas-de-negocio/01b-catalogos.md). Enum: `TipoDocumentoIdentidad`.

## 4. DIGEMID y farmacia

Mapa de la familia: SOFIA modela el **catálogo DIGEMID** como una tabla local de consulta (no hay conexión con DIGEMID), la **condición de venta** de cada medicamento (que decide si hace falta receta) y el circuito **receta → dispensación**. Cuarentena, actas de destrucción, fórmulas magistrales e inmunizaciones existen solo en la API.

### Acta de destrucción
*También aparece como:* acta DIGEMID, `DigemidActaDestruccion`, `DIGEMID_Actas_Destruccion`, «Generar acta de destrucción» (permiso).

**Definición.** Documento que deja constancia de la destrucción de productos farmacéuticos. Para una farmacia o botica no se encontró en fuente oficial el formato ni la norma vigente que fija sus requisitos. Definición general, no verificada en fuente oficial.

**En SOFIA.** **Solo API (sin pantalla).** Una acta guarda número de resolución interna (único por empresa, hasta 50 caracteres), empresa de residuos biocontaminados, documento de manifiesto de transporte (opcional), fecha de ejecución, regente responsable y ruta del PDF firmado (`DOM/DigemidActaDestruccion.cs:11-16`). El regente debe ser un empleado de la empresa (`APP/DIGEMID/Commands/GenerarActaDestruccion/GenerarActaDestruccionCommandHandler.cs:13`). Generar el acta **no cambia nada más**: no marca los registros de cuarentena como `Destruido` ni descuenta stock ([RN-CUA-017](01-reglas-de-negocio/01d-modulos-solo-api.md)).

**Dónde aparece.** Sin texto de interfaz; permiso «Generar acta de destrucción» (`I18N/roles/es.json:100`, `permisos.accion.GenerarActaDestruccion`). Tablas `DIGEMID_Actas_Destruccion` y `DIGEMID_Actas_Detalle`. Reglas: [RN-CUA-013](01-reglas-de-negocio/01d-modulos-solo-api.md) a [RN-CUA-019](01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-CUA-021](01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-CUA-022](01-reglas-de-negocio/01d-modulos-solo-api.md).

### Catálogo DIGEMID
*También aparece como:* «Catálogo DIGEMID», `DigemidCatalogoProducto`, `DIGEMID_Catalogo_Productos`.

**Definición.** Relación de productos farmacéuticos registrados ante DIGEMID. DIGEMID publica la relación de medicamentos registrados con su condición de venta. Fuente oficial: https://www.gob.pe/institucion/minsa/noticias/962205-farmacias-y-boticas-deben-exigir-receta-antes-de-vender-medicamentos-que-requieren-prescripcion-medica (confirma que la Autoridad publica esa relación; el contenido exacto del catálogo de SOFIA no sale de allí).

**En SOFIA.** Es una **tabla local**, sembrada por el script `DBUP/0004_Seed_Digemid_Catalogo.sql` y ampliable por carga masiva; SOFIA no consulta a DIGEMID. Cada fila tiene código de producto (`CodProd`, hasta 20 caracteres), nombre (`NomProd`, hasta 255), concentración, forma farmacéutica, fracción, registro sanitario, titular y estado (texto libre) (`DOM/DigemidCatalogoProducto.cs:7-14`; longitudes en [RN-DIG-005](01-reglas-de-negocio/01b-catalogos.md)). La pantalla consulta y exporta; el alta, edición y baja existen en la API pero ninguna pantalla los llama. Se usa para **validar con la IA** el nombre de un medicamento detectado en una receta ([RN-REC-023](01-reglas-de-negocio/01c1-inventario-y-recetas.md)).

**Dónde aparece.** Menú «DIGEMID» > «Catálogo DIGEMID» (`I18N/es.json:16`, `nav.catalogoDigemid`). Campos: «Código de Producto», «Nombre del Producto», «Concentración», «Forma Farmacéutica», «Fracción», «Registro Sanitario», «Titular», «Estado» (`I18N/digemidCatalogo/es.json:20-27`). Inventario: [19c](19-inventario-de-pantallas/19c-catalogos-1.md). Reglas: [RN-DIG-001](01-reglas-de-negocio/01b-catalogos.md) a [RN-DIG-013](01-reglas-de-negocio/01b-catalogos.md).

### Código ATC
*También aparece como:* «Código ATC», `CodigoAtc`, `Codigo_ATC`, clasificación Anatómica Terapéutica Química.

**Definición.** Código del sistema ATC de la OMS, que clasifica cada principio activo en cinco niveles (grupo anatómico, subgrupo terapéutico, farmacológico, químico y sustancia); el Ministerio de Salud adoptó ese sistema. Fuente oficial: http://www.hnhu.gob.pe/cuerpo/epidemiologia/ASIS%202011%20(F)/contenidos/normastecnicas/02ProductosFarmaceuticos.pdf (estándar de dato en salud del Minsa, alojado en el sitio de un hospital; confirmado por el resumen de la búsqueda).

**En SOFIA.** Es un atributo del **ingrediente activo**: obligatorio, hasta 15 caracteres, **sin validación de estructura** (cualquier texto es válido) ([RN-ING-004](01-reglas-de-negocio/01b-catalogos.md), [RN-ING-005](01-reglas-de-negocio/01b-catalogos.md)). No es único: la clave única del ingrediente es la denominación DCI.

**Dónde aparece.** Ingredientes Activos: «Código ATC» (`I18N/ingredientesActivos/es.json:23`, `fields.codigoAtc`). Tabla `Ingredientes_Activos`. Reglas: [RN-ING-004](01-reglas-de-negocio/01b-catalogos.md), [RN-ING-005](01-reglas-de-negocio/01b-catalogos.md), [RN-ING-012](01-reglas-de-negocio/01b-catalogos.md).

### Colegiatura
*También aparece como:* número de colegiatura, colegiado.

**Definición.** Número de inscripción de un profesional en su colegio profesional. En la normativa de establecimientos farmacéuticos, el nombre y el número de colegiatura del químico farmacéutico que asume la dirección técnica se declaran para obtener la autorización sanitaria. Fuente oficial: https://busquedas.elperuano.pe/normaslegales/decreto-supremo-que-modifica-el-reglamento-de-establecimient-decreto-supremo-n-004-2021-sa-1918578-1

**En SOFIA.** **No se usa en SOFIA.** Evidencia: la búsqueda de `colegiatura` y `colegiado` (sin distinguir mayúsculas) no devuelve resultados en `sofia-web-spa/src`, en `sofia-web-spa/public/i18n`, en `sofia-apis/src` ni en los scripts DbUp. Los datos más cercanos son la **licencia profesional** del empleado (ver «Licencia profesional») y el **número de registro** del profesional de salud (ver «Profesional de salud»); ninguno se declara como colegiatura.

**Dónde aparece.** No aparece.

### Concentración
*También aparece como:* «Concentración» (catálogo DIGEMID), «Concentración de Dosis» (formulación), `Concent`, `ConcentracionDosis`, `ConcentracionDetectada`.

**Definición.** Cantidad de principio activo contenida en un medicamento. Definición general, no verificada en fuente oficial.

**En SOFIA.** Tiene tres formas distintas: (1) en el **catálogo DIGEMID**, un texto libre de hasta 255 caracteres (`DOM/DigemidCatalogoProducto.cs:9`); (2) en la **formulación clínica**, un número decimal mayor que cero con 4 decimales, acompañado de una unidad de medida ([RN-FOR-006](01-reglas-de-negocio/01b-catalogos.md)); (3) en el **análisis de receta con IA**, un texto detectado en la imagen que ayuda a emparejar el medicamento ([RN-REC-024](01-reglas-de-negocio/01c1-inventario-y-recetas.md)). La tabla de medicamentos no tiene concentración.

**Dónde aparece.** «Concentración» (`I18N/digemidCatalogo/es.json:22`); «Concentración de Dosis» (`I18N/formulacionesClinicas/es.json:21`). Reglas: [RN-FOR-006](01-reglas-de-negocio/01b-catalogos.md), [RN-REC-024](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Condición de venta
*También aparece como:* «Condición de Venta», `CondicionVenta`, `Condicion_Venta`.

**Definición.** Condición bajo la cual se autoriza la venta de un producto farmacéutico, definida en su registro sanitario: de venta libre o con receta médica, y dentro de esta última con receta simple, retenida o especial. DIGEMID aplica reglas distintas según la categoría. Fuente oficial: https://www.digemid.minsa.gob.pe/webDigemid/registro-sanitario/productos-farmaceuticos/venta-sin-receta/ y https://www.gob.pe/institucion/minsa/noticias/962205-farmacias-y-boticas-deben-exigir-receta-antes-de-vender-medicamentos-que-requieren-prescripcion-medica (confirmado por el resumen de la búsqueda).

**En SOFIA.** Cada medicamento tiene **una** condición, obligatoria. Son cuatro valores; la base los guarda como el texto de la descripción y el CHECK `CHK_Condicion_Venta` los exige (`DBUP/0001_Initial_Schema.sql:78`, `:86`):

| Valor del enum | Texto que muestra la interfaz y valor guardado | Efecto en la venta |
|---|---|---|
| `VentaLibreOTC` | «Venta Libre (OTC)» | No exige receta |
| `RecetaSimple` | «Receta Simple» | Exige receta registrada si la empresa lo exige; si no, basta marcar «Receta verificada» al cobrar |
| `RecetaRetenida` | «Receta Retenida» | Exige receta registrada con su número; una sola dispensación |
| `Estupefaciente` | «Estupefaciente» | Igual que la retenida |

(`ENUM/CondicionVenta.cs:7-10`.) La lista de textos la entrega la API (`GET medicamentos/condiciones-venta`, `API/MedicamentosController.cs:24-25`, construida en `DOM/Medicamento.cs:27-31`) y el SPA la usa para el selector; el SPA conoce los valores por número (`SPA/pages/pos/models/venta.model.ts:16-21`).

**Dónde aparece.** Medicamentos: filtro y campo «Condición de Venta» (`I18N/medicamentos/es.json:34`, `fields.condicionVenta`); columna de la tabla; POS: etiqueta «Requiere receta» y avisos de receta (`I18N/pos/es.json:189-192`). Reglas: [RN-MED-007](01-reglas-de-negocio/01b-catalogos.md), [RN-MED-019](01-reglas-de-negocio/01b-catalogos.md), [RN-VEN-028](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-029](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Cuarentena
*También aparece como:* «Lotes en cuarentena», «Retenidos: no se pueden vender», `DigemidInventarioCuarentena`, `DIGEMID_Inventario_Cuarentena`, «Aislar lote en cuarentena» (permiso).

**Definición.** Situación de un lote de producto farmacéutico que se separa y se inmoviliza hasta decidir su destino. El manual de buenas prácticas de almacenamiento vigente (R.M. 132-2015/MINSA) trata las áreas de cuarentena, pero no se pudo confirmar su texto. Definición general, no verificada en fuente oficial. Referencia: https://www.digemid.minsa.gob.pe/webDigemid/normas-legales/2015/resolucion-ministerial-n-132-2015-minsa/

**En SOFIA.** **Solo API (sin pantalla)** para aislar. Un registro de cuarentena apunta a un lote y una sucursal, con cantidad aislada, motivo, fecha, empleado que lo registra y estado de resolución (`DOM/DigemidInventarioCuarentena.cs:10-17`). Un lote con un registro en estado `Retenido` **no se puede vender en ninguna sucursal** (`APP/Ventas/Common/VentaDetalleFactory.cs:168`) y deja de contar como stock vendible (`APP/Inventarios/Common/InventarioVendibleExtensions.cs`). Aislar no descuenta stock. No existe ninguna operación para liberar, destruir o devolver un lote retenido ([RN-CUA-009](01-reglas-de-negocio/01d-modulos-solo-api.md)). Devolver mercadería con destino «Cuarentena DIGEMID» solo anota el destino: no crea el registro de cuarentena ([RN-DEV-024](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CUA-011](01-reglas-de-negocio/01d-modulos-solo-api.md)).

**Dónde aparece.** Dashboard, sección Inventario: «Lotes en cuarentena» y «Retenidos: no se pueden vender» (`I18N/dashboard/es.json:72-73`, `inventario.quarantine`, `quarantineHint`). Permiso «Aislar lote en cuarentena» (`I18N/roles/es.json:92`). Reglas: [RN-CUA-001](01-reglas-de-negocio/01d-modulos-solo-api.md) a [RN-CUA-012](01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-CUA-020](01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-CUA-023](01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-DAS-022](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### DCI (denominación común internacional)
*También aparece como:* «Denominación DCI», `DenominacionDci`, nombre genérico, principio activo.

**Definición.** Nombre único, reconocido en todo el mundo y de propiedad pública, que identifica a un principio activo; se conoce también como nombre genérico. Fuente oficial: https://www.digemid.minsa.gob.pe/webDigemid/preguntas-frecuentes/ (confirmado por el resumen de la búsqueda; la obligación de prescribir por DCI figura en la normativa que cita DIGEMID, cuyo detalle no se verificó).

**En SOFIA.** Es el nombre del **ingrediente activo**: obligatorio, hasta 255 caracteres, **único por empresa** entre los no eliminados ([RN-ING-002](01-reglas-de-negocio/01b-catalogos.md), [RN-ING-003](01-reglas-de-negocio/01b-catalogos.md), [RN-ING-006](01-reglas-de-negocio/01b-catalogos.md)). SOFIA no obliga a recetar por DCI ni sustituye marcas por genéricos.

**Dónde aparece.** Ingredientes Activos: «Denominación DCI» (`I18N/ingredientesActivos/es.json:22`); mensaje «Ya existe un ingrediente activo con esa denominación DCI.» (`:13`, `detail.duplicate-dci`). Reglas: [RN-ING-002](01-reglas-de-negocio/01b-catalogos.md), [RN-ING-006](01-reglas-de-negocio/01b-catalogos.md).

### DIGEMID
*También aparece como:* Dirección General de Medicamentos, Insumos y Drogas, módulo de permisos `DIGEMID`.

**Definición.** Es la máxima autoridad nacional encargada de regular, controlar y vigilar los productos farmacéuticos, dispositivos médicos y productos sanitarios que se comercializan en el Perú; es un órgano de línea del Ministerio de Salud. Fuente oficial: https://www.gob.pe/21087-ministerio-de-salud-direccion-general-de-medicamentos-insumos-y-drogas-digemid

**En SOFIA.** Aparece de cuatro formas: (1) el **catálogo DIGEMID** local; (2) el módulo de permisos `DIGEMID` (que también protege cuarentena y actas); (3) el destino de devolución «Cuarentena DIGEMID»; (4) el texto «Reportes DIGEMID automatizados» en la pantalla de acceso (`I18N/auth/es.json:24`, `features.digemid`). **No hay reportes DIGEMID automatizados ni integración con DIGEMID** (comprobado en [07-integraciones §6](07-integraciones.md)): ese texto es de marketing.

**Dónde aparece.** Grupo de menú «DIGEMID» (`I18N/es.json:33`, `breadcrumbs.digemid`). Módulo «DIGEMID» en los permisos (`I18N/roles/es.json:61`). Matriz: [05b](05-seguridad/05b-matriz-de-permisos.md).

### Dispensación
*También aparece como:* dispensar, «dispensada el máximo de veces permitido», `PuedeDispensar`, `Venta.Receta.Agotada`.

**Definición.** Acto profesional de entregar al paciente el medicamento con la información necesaria para su uso. La venta de productos con receta está reservada a farmacias, boticas y servicios de farmacia. Fuente oficial: https://www.gob.pe/institucion/minsa/noticias/962205-farmacias-y-boticas-deben-exigir-receta-antes-de-vender-medicamentos-que-requieren-prescripcion-medica (confirma la obligación de exigir receta; no define «dispensación» textualmente).

**En SOFIA.** Una **venta cuenta como una sola dispensación de la receta, sin importar cuántas líneas tenga**, y solo si tiene productos que no son de venta libre (`APP/Ventas/Common/VentaDetalleFactory.cs:275-297`). Las ventas anuladas devuelven su dispensación; las ventas en espera la reservan; las devoluciones no la liberan ([RN-DEV-026](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md)). El máximo lo fija la condición de venta del producto (`DOM/RecetaMedica.cs:130-141`): receta simple, `1 + repeticiones máximas`; receta retenida y estupefaciente, 1; venta libre, sin límite. Al superarlo el cobro se rechaza con el mensaje «La receta médica ya fue dispensada el máximo de veces permitido.» (`I18N/pos/es.json:351`, `messages.recetaAgotada`).

**Dónde aparece.** Recetas Médicas: «Repeticiones Máximas» (`I18N/recetas-medicas/es.json:33`). Reglas: [RN-REC-031](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-VEN-033](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-DEV-026](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Estado de resolución de cuarentena
*También aparece como:* `EstadoResolucionCuarentena`, `Estado_Resolucion`.

**Definición.** Situación en que se encuentra un lote en cuarentena: retenido a la espera de decisión o ya liberado, destruido o devuelto. Definición general, no verificada en fuente oficial.

**En SOFIA.** Son cuatro valores de texto; el conjunto cerrado lo valida la entidad y **no hay CHECK en la base** (`ENUM/EstadoResolucionCuarentena.cs:3-13`; `DOM/DigemidInventarioCuarentena.cs:69`; columna con DEFAULT `'Retenido'` en `DBUP/0001_Initial_Schema.sql:640`):

| Valor | Significado en SOFIA | ¿Quién lo asigna? |
|---|---|---|
| `Retenido` | El lote no se puede vender | El aislamiento (por defecto) |
| `Liberado` | Previsto: el lote vuelve a venderse | Nadie: no hay operación ([RN-CUA-009](01-reglas-de-negocio/01d-modulos-solo-api.md)) |
| `Destruido` | Previsto: el lote se destruyó | Nadie |
| `Devuelto` | Previsto: el lote volvió al proveedor | Nadie |

No hay textos de interfaz para estos valores.

**Dónde aparece.** Solo en el código y en la columna `DIGEMID_Inventario_Cuarentena.Estado_Resolucion`. Reglas: [RN-CUA-005](01-reglas-de-negocio/01d-modulos-solo-api.md) a [RN-CUA-009](01-reglas-de-negocio/01d-modulos-solo-api.md).

### Estupefaciente
*También aparece como:* `CondicionVenta.Estupefaciente`, «Estupefaciente» (texto guardado).

**Definición.** Sustancia sujeta a control especial; su manejo se rige en el Perú por el reglamento de estupefacientes, psicotrópicos y otras sustancias sujetas a fiscalización sanitaria, que contempla recetas especiales o recetas comunes retenidas. Fuente oficial: https://cdn.www.gob.pe/uploads/document/file/283858/255646_DS023-2001.pdf20190110-18386-1k7n0nb.pdf (D.S. 023-2001-SA según el resultado de la búsqueda; su vigencia actual no se comprobó).

**En SOFIA.** Una de las cuatro condiciones de venta. Se trata como «controlado», igual que la receta retenida: **siempre exige una receta registrada con número de receta**, un cliente que sea su paciente y **una sola dispensación**, con la política de la empresa en cualquier valor (`APP/Ventas/Common/VentaDetalleFactory.cs:145-162`; `DOM/RecetaMedica.cs:137`). SOFIA no emite ni valida el formato de receta especial.

**Dónde aparece.** Medicamentos: «Condición de Venta» = «Estupefaciente». POS: «Receta retenida y estupefacientes siempre exigen una receta registrada con su número.» (`I18N/pos/es.json:173`, `checkout.recetaHint`). Reglas: [RN-VEN-029](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-032](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Fecha de caducidad
*También aparece como:* «Fecha de Caducidad», vencimiento, `FechaCaducidad`, `Fecha_Caducidad`, lote vencido, `EstaVencido`.

**Definición.** Fecha límite de uso del producto, registrada en el empaque por el fabricante. DIGEMID recomienda verificar el registro sanitario, la fecha de vencimiento y el estado de conservación antes de comprar. Fuente oficial: https://www.digemid.minsa.gob.pe/webDigemid/registro-sanitario/productos-farmaceuticos/

**En SOFIA.** Atributo obligatorio del **lote**. El lote **se vende hasta el final de su día de caducidad** y está vencido desde el día siguiente (`DOM/LoteInventario.cs:93-99`). Al crear un lote no se admite una caducidad ya vencida. Un lote vencido no se puede vender, se cuenta en el dashboard y se excluye del stock vendible. Término preferido: **caducidad** para el dato, **lote vencido** para el estado.

**Dónde aparece.** Lotes: «Fecha de Caducidad», «Caducidad Desde», «Caducidad Hasta» (`I18N/lotes/es.json:23`, `:24-25`). Dashboard: «Lotes vencidos con stock» y «Lotes que vencen en 30 días o menos» (`I18N/dashboard/es.json:64-65`). POS: «Uno de los lotes está vencido y no se puede vender. Retíralo del carrito.» (`I18N/pos/es.json:354`). Reglas: [RN-LOT-005](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-LOT-006](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-VEN-013](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-DAS-018](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Forma farmacéutica
*También aparece como:* «Forma Farmacéutica», `FormaFarmaceutica`.

**Definición.** Presentación física de un medicamento (tableta, jarabe, ampolla, etc.). DIGEMID otorga el registro sanitario según el nombre del producto, su forma farmacéutica, la cantidad de principio activo, el fabricante y el país. Fuente oficial: https://www.digemid.minsa.gob.pe/webDigemid/registro-sanitario/productos-farmaceuticos/

**En SOFIA.** Solo existe como texto libre (hasta 150 caracteres) en el **catálogo DIGEMID** (`DOM/DigemidCatalogoProducto.cs:10`). Los medicamentos y las presentaciones de venta no tienen forma farmacéutica.

**Dónde aparece.** Catálogo DIGEMID: «Forma Farmacéutica» en el filtro, la tabla y el detalle (`I18N/digemidCatalogo/es.json:23`, `:32`). Reglas: [RN-DIG-005](01-reglas-de-negocio/01b-catalogos.md).

### Formulación clínica
*También aparece como:* «Formulaciones Clínicas», `FormulacionClinica`, `Formulacion_Clinica`, «Código TE Orange».

**Definición.** Relación entre un medicamento y cada uno de sus ingredientes activos, con la concentración de la dosis. Término propio de SOFIA (no normativo).

**En SOFIA.** Une un **medicamento**, un **ingrediente activo**, una **concentración de dosis** (mayor que cero) y una **unidad de medida**. Un medicamento lista cada ingrediente una sola vez. El código TE Orange es opcional (hasta 5 caracteres). Dos medicamentos con el mismo ingrediente, concentración y unidad son **equivalentes terapéuticos** ([RN-REC-027](01-reglas-de-negocio/01c1-inventario-y-recetas.md)). El medicamento de una formulación no se puede cambiar una vez creada.

**Dónde aparece.** Menú «Catálogos» > «Formulaciones Clínicas» (`I18N/es.json:20`). Campos: «Medicamento», «Ingrediente Activo», «Concentración de Dosis», «Unidad de Medida», «Código TE Orange» (`I18N/formulacionesClinicas/es.json:18-23`). Reglas: [RN-FOR-001](01-reglas-de-negocio/01b-catalogos.md) a [RN-FOR-013](01-reglas-de-negocio/01b-catalogos.md).

### Fórmula magistral (orden magistral)
*También aparece como:* «Órdenes Magistrales», `MagistralOrdenProduccion`, `Magistrales_Ordenes_Produccion`, «Iniciar orden magistral», «Completar orden magistral».

**Definición.** Preparado farmacéutico destinado a un paciente individualizado, elaborado por el químico farmacéutico o bajo su dirección según una prescripción facultativa, y dispensado en una farmacia o botica. Fuente oficial: https://busquedas.elperuano.pe/normaslegales/aprueban-norma-tecnica-de-salud-para-la-elaboracion-de-prepa-resolucion-ministerial-n-538-2016minsa-1410178-1 (Norma Técnica de Salud 122-MINSA/DIGEMID, R.M. 538-2016/MINSA; la definición se leyó en el resumen de la búsqueda).

**En SOFIA.** **Solo API (sin pantalla).** Una orden se **inicia** consumiendo insumos de stock (queda `Iniciada`) y se **completa** creando un lote terminado del producto resultante con su stock (queda `Completada`) (`DOM/MagistralOrdenProduccion.cs:9-10`, `:21`; [RN-MAG-009](01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-MAG-016](01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-MAG-017](01-reglas-de-negocio/01d-modulos-solo-api.md)). La sucursal y el químico preparador salen del token; no se verifica que el empleado sea químico farmacéutico. SOFIA no impide mantener stock de una preparación.

**Dónde aparece.** Sin texto de interfaz. Permisos «Iniciar orden magistral» y «Completar orden magistral» (`I18N/roles/es.json:102`, `:99`); módulo «Órdenes Magistrales» (`:69`). Reglas: [RN-MAG-001](01-reglas-de-negocio/01d-modulos-solo-api.md) a [RN-MAG-023](01-reglas-de-negocio/01d-modulos-solo-api.md).

### Fracción
*También aparece como:* «Fracción», `Fraccion`.

**Definición.** Dato del catálogo DIGEMID que indica la fracción de la presentación en que puede venderse el producto. Definición general, no verificada en fuente oficial.

**En SOFIA.** Texto libre opcional del **catálogo DIGEMID** (`DOM/DigemidCatalogoProducto.cs:11`). No se usa para vender ni para calcular unidades: las equivalencias de venta se manejan con la jerarquía de unidades.

**Dónde aparece.** Catálogo DIGEMID, detalle: «Fracción» (`I18N/digemidCatalogo/es.json:24`).

### Ingrediente activo
*También aparece como:* «Ingredientes Activos», `IngredienteActivo`, `Ingredientes_Activos`, principio activo.

**Definición.** Sustancia responsable del efecto farmacológico de un medicamento; se identifica por su DCI y se clasifica con el código ATC. Definición general, no verificada en fuente oficial.

**En SOFIA.** Maestro con **denominación DCI** (única) y **código ATC**. Se usa en las formulaciones clínicas; no se puede eliminar un ingrediente incluido en formulaciones (`I18N/ingredientesActivos/es.json:16`, `detail.delete-in-use`).

**Dónde aparece.** Menú «Catálogos» > «Ingredientes Activos» (`I18N/es.json:11`). Inventario: [19c](19-inventario-de-pantallas/19c-catalogos-1.md). Reglas: [RN-ING-001](01-reglas-de-negocio/01b-catalogos.md) a [RN-ING-015](01-reglas-de-negocio/01b-catalogos.md).

### Inmunización
*También aparece como:* `ServicioClinicoInmunizacion`, `Servicios_Clinicos_Inmunizacion`, «Registrar inmunización» (permiso), `Fecha_Entrega_VIS`.

**Definición.** Registro de la administración de una vacuna a un paciente. Definición general, no verificada en fuente oficial.

**En SOFIA.** **Solo API (sin pantalla).** Guarda paciente, profesional que administra (un empleado), producto, lote, vía de administración, sitio anatómico, volumen de la dosis, fecha de administración, fecha de entrega de la hoja informativa (opcional) y modalidad de registro. El lote debe ser del producto y no estar vencido. **No consume stock** ([RN-SRV-012](01-reglas-de-negocio/01d-modulos-solo-api.md)).

**Dónde aparece.** Sin texto de interfaz. Permiso «Registrar inmunización» (`I18N/roles/es.json:105`). Reglas: [RN-SRV-007](01-reglas-de-negocio/01d-modulos-solo-api.md) a [RN-SRV-015](01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-SRV-020](01-reglas-de-negocio/01d-modulos-solo-api.md).

### Laboratorio
*También aparece como:* «Laboratorios», «Nombre de Compañía», «Código Identificador», `Laboratorio`.

**Definición.** Empresa fabricante o titular de un producto farmacéutico. Definición general, no verificada en fuente oficial.

**En SOFIA.** Maestro con **nombre de compañía** (obligatorio y único, hasta 150 caracteres) y **código identificador** (opcional y único si se informa, hasta 50). Todo medicamento exige un laboratorio; no se puede eliminar un laboratorio con medicamentos (`I18N/laboratorios/es.json:75`, `detail.delete-in-use`). La ayuda del campo dice «ej. Registro sanitario o código interno» (`:41`), pero el sistema no valida que sea un registro sanitario.

**Dónde aparece.** Menú «Catálogos» > «Laboratorios» (`I18N/es.json:10`). Reglas: [RN-LAB-001](01-reglas-de-negocio/01b-catalogos.md) a [RN-LAB-017](01-reglas-de-negocio/01b-catalogos.md).

### Lote
*También aparece como:* «Lotes», «Número de Lote», «Número de Lote (Fabricante)», `LoteInventario`, `NumeroLoteMfr`.

**Definición.** Cantidad de un producto fabricada en un mismo ciclo, identificada por el número de lote impreso en el empaque por el fabricante. Definición general, no verificada en fuente oficial.

**En SOFIA.** Un lote pertenece a un **medicamento** y tiene número de lote (hasta 100 caracteres, único por medicamento), fecha de caducidad y fecha de fabricación opcional (`DOM/LoteInventario.cs:9-12`). **El lote nace sin existencias**: las unidades se registran por sucursal en el stock (ver «Stock por sucursal»). La venta se hace por lote. No se puede eliminar un lote con stock o ventas.

**Dónde aparece.** Menú «Inventario» > «Lotes» (`I18N/es.json:22`). Campos: «Número de Lote», «Fecha de Fabricación», «Fecha de Caducidad» (`I18N/lotes/es.json:21-23`). Reglas: [RN-LOT-001](01-reglas-de-negocio/01c1-inventario-y-recetas.md) a [RN-LOT-019](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Modalidad de registro
*También aparece como:* `ModalidadRegistro`, `Modalidad_Registro`, `Corriente`, `Historica`.

**Definición.** Indica si una inmunización se registra en el momento en que se administra o se carga después desde antecedentes. Definición general, no verificada en fuente oficial.

**En SOFIA.** Enum de dos valores, obligatorio al registrar una inmunización; la base lo guarda como texto y lo exige con un CHECK (`ENUM/ModalidadRegistro.cs:5-9`; `DBUP/0001_Initial_Schema.sql:404`):

| Valor | Texto guardado |
|---|---|
| `Corriente` | `Corriente` |
| `Historica` | `Historica` |

No tiene texto de interfaz (módulo sin pantalla).

**Dónde aparece.** Solo API. Reglas: [RN-SRV-011](01-reglas-de-negocio/01d-modulos-solo-api.md).

### Número de receta
*También aparece como:* «Número de receta», `NumeroReceta`, `Numero_Receta`, «N.° {{numero}}».

**Definición.** Identificador impreso de la receta médica. Definición general, no verificada en fuente oficial.

**En SOFIA.** Dato **opcional** del maestro de recetas (hasta 30 caracteres; un valor vacío se guarda como sin número), pero **obligatorio para vender receta retenida o estupefaciente**: sin él, el cobro se rechaza (`DOM/RecetaMedica.cs:8`, `:145-146`; `APP/Ventas/Common/VentaDetalleFactory.cs:157-159`, `:271`).

**Dónde aparece.** Recetas Médicas: «Número de receta» con la ayuda «Necesario para vender receta retenida o estupefacientes.» (`I18N/recetas-medicas/es.json:34`, `:91`). POS: «N.° {{numero}}» (`I18N/pos/es.json:174`). Reglas: [RN-REC-010](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-VEN-032](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Paciente
*También aparece como:* «Pacientes», cliente, `PacienteCliente`, `Pacientes_Clientes`.

**Definición.** Persona que recibe atención o compra medicamentos. Definición general, no verificada en fuente oficial.

**En SOFIA.** Maestro con tipo y número de documento, nombre y apellidos (un solo campo), fecha de nacimiento (obligatoria, no futura) y contacto primario (opcional). El documento es único por empresa. En el POS se le llama **cliente**. La fecha de nacimiento aparece en la ayuda como «utilizada para el cálculo automático de dosis», pero SOFIA no calcula dosis.

**Dónde aparece.** Menú «Pacientes & Atención» > «Pacientes» (`I18N/es.json:14`). Campos: «Tipo de Documento», «Documento de Identidad», «Nombre y Apellidos», «Fecha de Nacimiento», «Contacto Primario» (`I18N/pacientes/es.json:93-97`). Reglas: [RN-PAC-001](01-reglas-de-negocio/01b-catalogos.md) a [RN-PAC-017](01-reglas-de-negocio/01b-catalogos.md).

### Profesional de salud
*También aparece como:* «Profesionales de Salud», prescriptor, médico, `ProfesionalSalud`, «Nombre del Prescriptor», «Número de Registro».

**Definición.** Profesional que prescribe medicamentos. Definición general, no verificada en fuente oficial.

**En SOFIA.** Maestro con **número de registro** (obligatorio, único por empresa, hasta 50 caracteres), **nombre del prescriptor** (hasta 150) y dirección de clínica (opcional) (`DOM/ProfesionalSalud.cs:9-11`). Es el «Médico» de la receta. No guarda especialidad ni colegiatura. No es un empleado: el profesional que **administra** una inmunización sí es un empleado (ver «Inmunización»).

**Dónde aparece.** Menú «Pacientes & Atención» > «Profesionales de Salud» (`I18N/es.json:15`). Campos: `I18N/profesionalesSalud/es.json:22-24`. Receta: «Médico» (`I18N/recetas-medicas/es.json:31`). Reglas: [RN-PRO-001](01-reglas-de-negocio/01b-catalogos.md) a [RN-PRO-012](01-reglas-de-negocio/01b-catalogos.md).

### Receta médica
*También aparece como:* «Recetas Médicas», «Receta médica», `RecetaMedica`, `Recetas_Medicas`.

**Definición.** Documento por el cual un profesional prescriptor indica un medicamento a un paciente. Las farmacias y boticas deben exigirla para vender los medicamentos que la requieren. No se encontró en fuente oficial una definición textual del documento. Fuente oficial (obligación de exigirla): https://www.gob.pe/institucion/minsa/noticias/962205-farmacias-y-boticas-deben-exigir-receta-antes-de-vender-medicamentos-que-requieren-prescripcion-medica

**En SOFIA.** Registro de **paciente**, **médico** (profesional de salud), **fecha de expedición** (no futura), **repeticiones máximas** (≥ 0, por defecto 0), **indicaciones de uso** (hasta 1 000 caracteres) y **número de receta** opcional (`DOM/RecetaMedica.cs:12-19`). **No guarda los medicamentos recetados**: la receta registrada se vincula a las líneas de una venta. No se puede eliminar una receta ya usada en una venta. Hay también un flujo de **lectura con IA** (ver «Análisis de receta»).

**Dónde aparece.** Menú «Pacientes & Atención» > «Recetas Médicas» (`I18N/es.json:26`). Campos: «Paciente», «Médico», «Fecha de Expedición», «Repeticiones Máximas», «Número de receta», «Indicaciones de Uso» (`I18N/recetas-medicas/es.json:30-35`). POS: «Receta médica», «Sin receta», «Registrar receta» (`I18N/pos/es.json:171-175`). Reglas: [RN-REC-001](01-reglas-de-negocio/01c1-inventario-y-recetas.md) a [RN-REC-017](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-REC-031](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Receta retenida
*También aparece como:* `CondicionVenta.RecetaRetenida`, «Receta Retenida».

**Definición.** Receta que el establecimiento retiene y archiva tras atenderla. Según DIGEMID, las recetas de medicamentos de ciertas listas de control se retienen en el establecimiento después de ser atendidas y deben archivarse. Fuente oficial: https://cdn.www.gob.pe/uploads/document/file/283858/255646_DS023-2001.pdf20190110-18386-1k7n0nb.pdf (D.S. 023-2001-SA; el detalle de qué listas la exigen no se verificó).

**En SOFIA.** Condición de venta cuya regla es: **receta registrada con número de receta**, del mismo cliente de la venta, y **una sola dispensación** (`DOM/RecetaMedica.cs:136`). SOFIA **no registra que la receta física haya sido retenida ni archivada**: lo único que deja es la dispensación contada y, si el cajero lo marcó, la verificación de la receta.

**Dónde aparece.** Medicamentos: «Condición de Venta» = «Receta Retenida». POS: «Hay productos de receta retenida o estupefacientes: selecciona al cliente y una receta registrada que tenga número de receta.» (`I18N/pos/es.json:191`). Reglas: [RN-VEN-029](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-032](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-033](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-REC-031](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Receta simple
*También aparece como:* `CondicionVenta.RecetaSimple`, «Receta Simple».

**Definición.** Receta común que habilita la venta de un medicamento de venta con receta, sin las exigencias de retención de los controlados. Definición general, no verificada en fuente oficial.

**En SOFIA.** Su exigencia depende de la **política de la empresa** `ExigirRegistroReceta` (apagada al registrar la empresa; solo se cambia en la base de datos): con la política **encendida**, la venta exige receta registrada; con ella **apagada**, el cajero puede vender sin receta registrada si marca «Receta verificada» (`APP/Ventas/Common/VentaDetalleFactory.cs:145-155`; `DOM/Empresa.cs:24-25`). El máximo de dispensaciones es `1 + repeticiones máximas` (`DOM/RecetaMedica.cs:135`).

**Dónde aparece.** Medicamentos: «Condición de Venta» = «Receta Simple». POS: «Hay productos que requieren una receta registrada: selecciona al cliente y su receta antes de cobrar.» (`I18N/pos/es.json:190`). Reglas: [RN-VEN-028](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-029](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-EMP-016](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Receta verificada
*También aparece como:* «Receta verificada», «Verificación de receta», `EmpleadoVerificaRecetaId`, `FechaVerificacionReceta`.

**Definición.** Confirmación del cajero de que revisó la receta física del cliente. Término propio de SOFIA (no normativo).

**En SOFIA.** Casilla del POS que solo se exige al **cobrar** cuando hay líneas de receta simple sin receta registrada en una empresa con la política apagada. Se guarda quién cobró y cuándo (`DOM/Venta.cs:22-23`, `:205-217`). **El número de la receta de papel no se guarda en ningún lado.**

**Dónde aparece.** POS: casilla «Receta verificada» con la ayuda «Confirmas que revisaste la receta física del cliente. Quedará registrado a tu nombre.» (`I18N/pos/es.json:193-194`); detalle de venta: «Receta verificada por {{nombre}} el {{fecha}}» (`:82`). Reglas: [RN-VEN-034](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-035](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Regente
*También aparece como:* químico farmacéutico, director técnico, «regente responsable», `RegenteResponsableId`, `Regente_Responsable_ID`, `QuimicoPreparadorId`.

**Definición.** En la normativa vigente de establecimientos farmacéuticos, el profesional químico farmacéutico que asume la dirección técnica se denomina **Director Técnico**; «regente» es el término del reglamento anterior (D.S. 021-2001-SA, derogado). El establecimiento no puede funcionar sin director técnico. Fuente oficial: https://www.gob.pe/38160-obtener-autorizacion-sanitaria-para-farmacias?child=57900 y https://busquedas.elperuano.pe/normaslegales/decreto-supremo-que-modifica-el-reglamento-de-establecimient-decreto-supremo-n-004-2021-sa-1918578-1 (la vigencia y el texto consolidado no se verificaron).

**En SOFIA.** No hay un cargo ni una entidad de regente. El «regente responsable» de un acta de destrucción es **cualquier empleado de la empresa** elegido por su identificador, y el «químico preparador» de una orden magistral es **el empleado que inicia la orden**. SOFIA no verifica que sean químicos farmacéuticos. Existe un rol de base llamado `Farmaceutico` descrito como «Químico Farmacéutico - Responsable Clínico y Regencia» (`DBUP/0002_SeedBaseRoles.sql:20`), pero ese rol se siembra sin permisos y con `TenantId` nulo, por lo que las consultas normales no lo ven (ver [03a §10.1](03-modelo-de-datos/03a-maestros-y-seguridad.md)).

**Dónde aparece.** Sin texto de interfaz. Reglas: [RN-CUA-014](01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-MAG-002](01-reglas-de-negocio/01d-modulos-solo-api.md).

### Registro sanitario
*También aparece como:* «Registro Sanitario», `RegistroSanitario`.

**Definición.** Documento oficial que emite DIGEMID para autorizar la fabricación, importación y comercialización de un producto farmacéutico, tras evaluar su eficacia, seguridad y calidad. Fuente oficial: https://www.digemid.minsa.gob.pe/webDigemid/registro-sanitario/productos-farmaceuticos/

**En SOFIA.** Solo existe como texto opcional del **catálogo DIGEMID** (`DOM/DigemidCatalogoProducto.cs:12`). Los medicamentos de la empresa no tienen registro sanitario, y SOFIA no comprueba que un medicamento esté registrado.

**Dónde aparece.** Catálogo DIGEMID: «Registro Sanitario» (`I18N/digemidCatalogo/es.json:25`).

### Repeticiones (máximas)
*También aparece como:* «Repeticiones Máximas», `RepeticionesMax`, `Repeticiones_Max`.

**Definición.** Número de veces adicionales que una receta puede volver a dispensarse tras la primera entrega. Definición general, no verificada en fuente oficial.

**En SOFIA.** Número entero ≥ 0 de la receta (por defecto 0). **Solo cuenta para la receta simple** (`dispensaciones permitidas = 1 + repeticiones`); para receta retenida y estupefaciente se permite una sola dispensación aunque la receta tenga repeticiones (`DOM/RecetaMedica.cs:134-137`).

**Dónde aparece.** Recetas Médicas: «Repeticiones Máximas» (campo) y «Repeticiones» (columna) (`I18N/recetas-medicas/es.json:33`, `:41`). Reglas: [RN-REC-008](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-REC-031](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Venta libre (OTC)
*También aparece como:* `CondicionVenta.VentaLibreOTC`, «Venta Libre (OTC)».

**Definición.** Medicamento que puede venderse sin receta médica. DIGEMID mantiene una página de orientación sobre la venta sin receta. Fuente oficial: https://www.digemid.minsa.gob.pe/webDigemid/registro-sanitario/productos-farmaceuticos/venta-sin-receta/

**En SOFIA.** Valor cero del enum. **Nunca exige receta** y no limita las dispensaciones. Si la venta incluye una receta registrada, esta debe ser del mismo cliente aunque el producto sea libre (`APP/Ventas/Common/VentaDetalleFactory.cs:266-269`).

**Dónde aparece.** Medicamentos: «Venta Libre (OTC)». POS: el filtro de catálogo «Todos» y la etiqueta «Requiere receta» distinguen los productos con receta (`I18N/pos/es.json:216`, `:189`). Reglas: [RN-VEN-029](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

## 5. Caja y punto de venta

Mapa de la familia: el cajero **abre una sesión de caja** con un monto de apertura, **cobra ventas** con uno o varios métodos de pago (cada cobro emite una boleta), puede dejar ventas **en espera**, **anular** una venta o **devolver** parte de ella (con nota de crédito y reembolso) y al final **cierra la caja contando el efectivo**: el sistema calcula el efectivo esperado, compara y deja la sesión `Cuadrada` o `Cerrada`.

### Anulación
*También aparece como:* «Anular Venta», «Motivo de Anulación», `Anular`, `EstadoVenta.Anulada`, `Sesion_Anulacion_ID`.

**Definición.** Dejar sin efecto una venta registrada. Definición general, no verificada en fuente oficial.

**En SOFIA.** Se anula una venta `Pendiente` (en espera) o `Completada`; una venta con devoluciones **ya no puede anularse** (`DOM/Venta.cs:89-111`). El motivo es obligatorio. Anular devuelve el stock a cada lote, deja el comprobante en estado `Anulado` y libera la dispensación de la receta. Los pagos y el reclamo de seguro **quedan como estaban**. Para anular una venta ya cobrada hace falta **tener la propia caja abierta en la sucursal**: la anulación se registra en esa sesión y el efectivo neto de la venta sale de ella en el arqueo (`APP/POS/Common/ArqueoCajaCalculator.cs:10-21`).

**Dónde aparece.** Detalle de venta: botón «Anular Venta» y diálogo con «Motivo de Anulación» (`I18N/pos/es.json:95-98`); avisos `anularConDevoluciones` y `anularSinSesionAbierta` (`:102-103`). Permiso «Anular» del módulo «Ventas» (`I18N/roles/es.json`, `permisos.accion.Anular`). Reglas: [RN-VEN-062](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) a [RN-VEN-069](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CMP-012](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CAJ-012](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Apertura de caja
*También aparece como:* «Aperturar Caja», «Aperturar», «Aperturar Caja de Turno», `AperturarCaja`.

**Definición.** Inicio del turno de un cajero con un fondo en efectivo. Definición general, no verificada en fuente oficial.

**En SOFIA.** Crea una **sesión de caja** en estado `Abierta`. La sucursal, el cajero y la hora **no se eligen**: salen del token y del servidor. Un empleado **no puede tener dos cajas abiertas a la vez, en ninguna sucursal** (índice único filtrado `UX_POS_Sesiones_Caja_Empleado_Abierta`).

**Dónde aparece.** Sesiones de Caja: «Nueva Sesión de Caja» con «La caja se abrirá a tu nombre, en tu sucursal activa y con la hora actual.» (`I18N/sesiones-caja/es.json:13`, `:16`); POS: «Aperturar Caja» y «Debes aperturar la caja del turno antes de procesar una venta.» (`I18N/pos/es.json:5`, `:205`). Mensaje de duplicado: «Ya tienes una caja abierta. Ciérrala antes de abrir otra.» (`I18N/sesiones-caja/es.json:18`). Reglas: [RN-CAJ-001](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) a [RN-CAJ-005](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CAJ-026](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Arqueo
*También aparece como:* «Cierre y Arqueo de Caja», «Diferencia Arqueo (S/)», «Reporte de Cierre de Caja», `DiferenciaArqueo`, `ArqueoCajaCalculator`.

**Definición.** Recuento del dinero de la caja para compararlo con lo que debería haber según el sistema. Definición general, no verificada en fuente oficial.

**En SOFIA.** Se hace **una sola vez, al cerrar**, y queda guardado como una foto: efectivo esperado (calculado por el sistema), monto declarado (contado por el cajero) y su diferencia (`DOM/PosSesionCaja.cs:121-150`). No se recalcula después.

**Dónde aparece.** Sesiones de Caja, detalle: «Monto Cierre Declarado (S/)», «Monto Cierre Calculado (S/)» y «Diferencia Arqueo (S/)» (`I18N/sesiones-caja/es.json:33-35`). Reglas: [RN-CAJ-011](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) a [RN-CAJ-019](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Cajero
*También aparece como:* «Empleado / Cajero», «Cajero Principal» (valor de ejemplo), «Empleado».

**Definición.** Empleado que opera una caja y cobra ventas. Término propio de SOFIA (no normativo).

**En SOFIA.** No hay un rol «Cajero» que el código reconozca (el script `DBUP/0002_SeedBaseRoles.sql:29-33` siembra uno con ese nombre, sin permisos y con `TenantId` nulo, que las pantallas no ven): el cajero de una sesión o de una venta es el **empleado del token** (`Empleado_ID`). Solo el cajero que abrió la caja o un usuario con el rol Admin puede cerrarla (`APP/POS/Commands/CerrarCaja/CerrarCajaCommandHandler.cs:28-32`).

**Dónde aparece.** Sesiones de Caja: «Cajero» (`I18N/sesiones-caja/es.json:29`); POS: «Empleado / Cajero» (`I18N/pos/es.json:37`, `table.colEmpleado`).

### Cierre de caja
*También aparece como:* «Cerrar Caja», «Cerrar Sesión», «Confirmar Cierre», `CerrarCaja`.

**Definición.** Fin del turno de un cajero con el recuento del efectivo. Definición general, no verificada en fuente oficial.

**En SOFIA.** El cajero declara el efectivo contado; el sistema calcula el esperado, guarda la diferencia y pasa la sesión a `Cuadrada` (diferencia exactamente 0) o `Cerrada` (cualquier otra diferencia, aunque sea de un centavo) (`DOM/PosSesionCaja.cs:146-147`). **Un efectivo esperado negativo impide cerrar**. No existe reapertura. El cierre no revisa las ventas en espera.

**Dónde aparece.** Sesiones de Caja: «Cerrar Sesión» (`I18N/sesiones-caja/es.json:21`, `:55`), «Confirmar Cierre» (`:56`); error «Solo el cajero que abrió la caja o un administrador puede cerrarla.» (`:25`). Permiso «Cerrar caja» (`I18N/roles/es.json`, `permisos.accion.CerrarCaja`). Reglas: [RN-CAJ-007](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) a [RN-CAJ-010](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CAJ-015](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) a [RN-CAJ-019](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Código de venta
*También aparece como:* «Código de Venta», `{prefijo}-{correlativo}`.

**Definición.** Identificador legible de una venta. Término propio de SOFIA (no normativo).

**En SOFIA.** **No es una columna**: se calcula al consultar. Si la venta tiene boleta, es el número del comprobante (`B001-00000042`); si es una venta en espera, son los primeros 8 caracteres del identificador de la venta en mayúsculas. Al cobrar una venta en espera el código cambia al número de la boleta ([RN-VEN-059](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md)).

**Dónde aparece.** Ventas: columna «Código de Venta» (`I18N/pos/es.json:68`, `ventasTable.colCodigo`). Reglas: [RN-VEN-059](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Cobertura de aseguradora
*También aparece como:* «Cobertura de Seguro (opcional)», «Monto Cubierto por Seguro (S/)», `MontoCubiertoSeguro`, `AseguradoraId`.

**Definición.** Parte del total de una venta que paga una aseguradora; el paciente paga el resto (copago). Definición general, no verificada en fuente oficial.

**En SOFIA.** **Solo API (la pantalla la tiene apagada).** La petición de venta puede traer una aseguradora y un monto cubierto entre 0 y el total; los pagos solo deben cubrir `total − cobertura`; el comprobante se emite por el total completo. La sección de seguro del terminal POS existe pero está **oculta por una constante en `false`** (`SPA/pages/pos/nueva-venta/nueva-venta.component.ts:215`, usada en `nueva-venta.component.html:456`). El arqueo descuenta la cobertura al calcular el vuelto y el reembolso de una devolución excluye la parte cubierta.

**Dónde aparece.** Textos del POS (hoy ocultos): «Cobertura de Seguro (opcional)», «Aseguradora», «Monto Cubierto por Seguro (S/)» (`I18N/pos/es.json:176-179`). Reglas: [RN-VEN-047](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-DEV-017](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CAJ-013](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Comprobante impreso
*También aparece como:* «Imprimir Comprobante», «Reimprimir Comprobante», representación impresa, recibo.

**Definición.** SUNAT admite entregar el comprobante electrónico mediante una representación impresa. Fuente oficial: https://cpe.sunat.gob.pe/tipos_de_comprobantes/boleta (confirmado por el resumen de la búsqueda).

**En SOFIA.** El POS imprime la **boleta** desde el navegador, sin impresora fiscal ni formato SUNAT: muestra los datos del emisor (razón social, RUC y dirección de la sucursal), el cliente, las líneas, subtotal, IGV, total, el vuelto cuando existe, el estado SUNAT y el cajero. El detalle de una venta anulada no ofrece reimprimir.

**Dónde aparece.** Diálogo «Venta Procesada Exitosamente»: «Imprimir Comprobante» (`I18N/pos/es.json:13`); detalle de venta: «Reimprimir Comprobante» (`:20`). Reglas: [RN-CMP-013](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CMP-014](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Destino de la devolución
*También aparece como:* «Destino», `DestinoDevolucion`, `Destino_Fisico_Logico`.

**Definición.** Qué se hace con la mercadería devuelta: volver a venderla o aislarla. Término propio de SOFIA (no normativo).

**En SOFIA.** Enum de dos valores por línea devuelta; se guarda el texto del enum y la base lo exige con un CHECK (`ENUM/DestinoDevolucion.cs:7-8`; `DBUP/0001_Initial_Schema.sql:624`):

| Valor guardado | Texto en la interfaz | Efecto |
|---|---|---|
| `Reingreso_Venta` | «Reingreso a venta» | La cantidad vuelve al stock del lote en la sucursal (`APP/Devoluciones/Commands/ProcesarDevolucion/ProcesarDevolucionCommandHandler.cs:175-193`) |
| `Cuarentena_DIGEMID` | «Cuarentena DIGEMID» | **Solo se anota el destino**: no se repone stock y no se crea registro de cuarentena ([RN-DEV-024](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CUA-011](01-reglas-de-negocio/01d-modulos-solo-api.md)) |

**Dónde aparece.** Diálogo «Procesar devolución», columna «Destino» (`I18N/pos/es.json:114-117`, `devolucion.destino`). Reglas: [RN-DEV-023](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-DEV-024](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Despacho (delivery)
*También aparece como:* «Despachos Delivery», `DespachoDelivery`, `Despachos_Delivery`, «Programar delivery» (permiso).

**Definición.** Entrega a domicilio de una venta. Definición general, no verificada en fuente oficial.

**En SOFIA.** **Solo API (sin pantalla).** Una venta tiene como máximo un despacho vivo, con una plataforma de servicio (hasta 50 caracteres) y un estado. No tiene costo ni se refleja en el comprobante. Estados del enum `EstadoDespacho` (`ENUM/EstadoDespacho.cs:7-10`), sin textos de interfaz:

| Valor | Significado | Transiciones permitidas ([RN-DEL-010](01-reglas-de-negocio/01d-modulos-solo-api.md)) |
|---|---|---|
| `Preparando` | Estado inicial al programar | A `En_Camino` o `Devuelto` |
| `En_Camino` | Salió del local | A `Entregado` o `Devuelto` |
| `Entregado` | Recibido por el cliente | Final |
| `Devuelto` | Regresó al local | Final |

**Dónde aparece.** Permiso «Programar delivery» y módulo «Despachos Delivery» (`I18N/roles/es.json`, `permisos.accion.ProgramarDelivery`, `permisos.modulo.Delivery`). Reglas: [RN-DEL-001](01-reglas-de-negocio/01d-modulos-solo-api.md) a [RN-DEL-018](01-reglas-de-negocio/01d-modulos-solo-api.md).

### Devolución
*También aparece como:* «Devolver», «Procesar devolución», «Devoluciones / Notas de crédito», `DevolucionCabecera`, `Devoluciones_Cabecera`.

**Definición.** Entrega de vuelta de mercadería ya vendida, con la nota de crédito que reduce el comprobante. Ver «Nota de crédito». Definición general, no verificada en fuente oficial.

**En SOFIA.** Se procesa **desde el detalle de la venta** (botón «Devolver»). Condiciones: venta `Completada` de la **propia sucursal**, **caja propia abierta** y una **boleta o factura** emitida que acreditar, más una serie de nota de crédito activa. Se indica la cantidad por línea, el destino, el motivo SUNAT, el sustento y el método de reembolso. Una devolución **parcial** deja la venta `Completada`; cuando todo queda devuelto, la venta pasa a `Devuelta` (`DOM/Venta.cs:148-152`). Devolver **no libera** la dispensación de la receta ([RN-DEV-026](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md)). Las pantallas de listado y detalle de devoluciones no existen aunque la API las expone.

**Dónde aparece.** Detalle de venta: «Devolver» (`I18N/pos/es.json:106`), «Procesar devolución» (`:107`), «Devolver todo lo pendiente» (`:109`), columnas «Vendido», «Devuelto», «Devolvible», «A devolver», «Destino» (`:110-114`). Reglas: [RN-DEV-001](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) a [RN-DEV-037](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Diferencia de arqueo (faltante y sobrante)
*También aparece como:* «Diferencia Arqueo (S/)», «Faltante», «Sobrante», `DiferenciaArqueo`, `Diferencia_Arqueo`.

**Definición.** Resultado de comparar el efectivo contado con el efectivo esperado. Definición general, no verificada en fuente oficial.

**En SOFIA.** `Diferencia = monto declarado − efectivo esperado` (`DOM/PosSesionCaja.cs:146`). **Negativa = faltante** (se declaró menos de lo esperado); **positiva = sobrante**; cero = cuadre exacto. La pantalla de sesiones muestra solo el número con dos decimales. Los textos «Faltante», «Sobrante» y «Cuadre Exacto» existen en `pos/es.json` pero **ninguna pantalla los usa** hoy.

**Dónde aparece.** Sesiones de Caja, detalle: «Diferencia Arqueo (S/)» (`I18N/sesiones-caja/es.json:35`). Textos sin uso: «Faltante: -S/ {{monto}}» y «Sobrante: +S/ {{monto}}» (`I18N/pos/es.json:264-265`, `dialogs.faltanteMonto`, `dialogs.sobranteMonto`). Reglas: [RN-CAJ-015](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Efectivo esperado
*También aparece como:* «Monto Cierre Calculado (S/)», «Efectivo Esperado en Cajón», `MontoCierreCalculado`, `Monto_Cierre_Calculado`.

**Definición.** Efectivo que debería haber en la caja al cerrar, según las operaciones registradas. Término propio de SOFIA (no normativo).

**En SOFIA.** `Efectivo esperado = monto de apertura + efectivo neto de las ventas cobradas en la sesión − efectivo neto de ventas de otras cajas anuladas desde esta sesión − reembolsos en efectivo de las devoluciones procesadas desde esta sesión`. El efectivo neto de una venta es lo pagado en efectivo menos el vuelto. No cuentan las ventas en espera (`APP/POS/Common/ArqueoCajaCalculator.cs:10-54`). Se guarda con el cierre.

**Dónde aparece.** Sesiones de Caja: «Monto Cierre Calculado (S/)» (`I18N/sesiones-caja/es.json:34`). Dashboard, tarjeta Caja: «Efectivo esperado» (`I18N/dashboard/es.json`, `caja.expected`). Reglas: [RN-CAJ-011](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) a [RN-CAJ-014](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-DAS-015](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Estado de la sesión de caja
*También aparece como:* `EstadoSesion`, `Estado_Sesion`, «Caja Abierta», «Caja Cerrada», «Caja Cuadrada», sesión cuadrada frente a cerrada.

**Definición.** Situación de una sesión de caja. Término propio de SOFIA (no normativo).

**En SOFIA.** Tres valores de texto, con CHECK `CHK_Estado_Sesion` (`ENUM/EstadoSesion.cs:7-9`; `DBUP/0001_Initial_Schema.sql:332`):

| Valor | Texto en Sesiones de Caja | Significado |
|---|---|---|
| `Abierta` | «Abierta» | Turno en curso; el cajero puede cobrar |
| `Cerrada` | «Cerrada» | Cerrada con **diferencia** distinta de cero (faltante o sobrante) |
| `Cuadrada` | «Cuadrada» | Cerrada con **diferencia exactamente cero** |

`Cerrada` y `Cuadrada` son estados finales (`DOM/PosSesionCaja.cs:147`). Textos antiguos «Caja Abierta», «Caja Cerrada» y «Caja Cuadrada» existen en `pos/es.json:25-27` pero las pantallas vigentes usan los de Sesiones de Caja.

**Dónde aparece.** Filtro y columna «Estado» de Sesiones de Caja (`I18N/sesiones-caja/es.json:48-51`, `estado.*`). Reglas: [RN-CAJ-010](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CAJ-015](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Estado de la venta
*También aparece como:* `EstadoVenta`, `Estado`.

**Definición.** Situación de una venta. Término propio de SOFIA (no normativo).

**En SOFIA.** Cuatro valores de texto con CHECK `CHK_Venta_Estado` (`ENUM/EstadoVenta.cs:5-9`; `DBUP/0001_Initial_Schema.sql:358`):

| Valor guardado | Texto en la interfaz | Cuándo ocurre |
|---|---|---|
| `Completada` | «Completada» | Venta cobrada (con pagos y comprobante) |
| `Anulada` | «Anulada» | Se anuló una venta en espera o cobrada |
| `Devuelta` | «Devuelta» | Se devolvió **todo** (las devoluciones parciales dejan `Completada`) |
| `Pendiente` | «Pendiente» | Venta en espera: sin pagos ni comprobante |

(`I18N/pos/es.json:49-54`, `estado.*`.) Una venta `Anulada` o `Devuelta` no vuelve a otro estado.

**Dónde aparece.** Ventas: filtro «Estado» y columna «Estado» (`I18N/pos/es.json:60`, `:73`). Reglas: [RN-VEN-049](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Método de pago
*También aparece como:* `MetodoPago`, `Metodo_Pago`, «Método de Pago», forma de pago.

**Definición.** Medio con el que el cliente paga una venta. Definición general, no verificada en fuente oficial.

**En SOFIA.** Cuatro valores. La base guarda el texto de la descripción del enum (`ENUM/MetodoPago.cs:7-10`; conversión en `CFG/VentaPagoConfiguration.cs:26`); la API usa el nombre del enum:

| Nombre en la API | Valor guardado | Texto en el POS | Texto en el dashboard | ¿Pide número de operación? |
|---|---|---|---|---|
| `Efectivo` | `EFECTIVO` | «Efectivo» | «Efectivo» | No |
| `YapePlin` | `YAPE_PLIN` | «Yape / Plin» | «Yape / Plin» | Sí |
| `Tarjeta` | `TARJETA` | «Tarjeta Débito/Crédito» | «Tarjeta» | Sí |
| `Transferencia` | `TRANSFERENCIA` | «Transferencia» | «Transferencia» | Sí |

(`I18N/pos/es.json:181-184`; `I18N/dashboard/es.json`, `metodo.*`.) Los mismos cuatro sirven para el **método de reembolso** de una devolución. SOFIA no integra pasarelas de pago: el método es solo un registro.

**Dónde aparece.** POS, «Método de Pago» (`I18N/pos/es.json:180`); detalle de venta, «Método de Pago» (`:91`); dashboard, «Cobrado por método de pago». Reglas: [RN-VEN-040](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-042](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-DEV-018](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-DAS-010](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Monto de apertura
*También aparece como:* «Monto Apertura (S/)», «Monto de Apertura (S/)», «Monto Inicial (S/)», `MontoAperturaEfectivo`.

**Definición.** Fondo en efectivo con el que arranca la caja. Término propio de SOFIA (no normativo).

**En SOFIA.** Número mayor o igual a 0 (el cero es válido) (`DOM/PosSesionCaja.cs:47`). Es la base del efectivo esperado.

**Dónde aparece.** Sesiones de Caja: «Monto Apertura (S/)» (`I18N/sesiones-caja/es.json:32`); POS, diálogo de apertura: «Monto de Apertura (S/)» (`I18N/pos/es.json:209`). Reglas: [RN-CAJ-003](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Monto declarado
*También aparece como:* «Monto Cierre Declarado (S/)», «Monto Declarado en Soles (S/)», `MontoCierreDeclarado`.

**Definición.** Efectivo que el cajero cuenta y declara al cerrar la caja. Término propio de SOFIA (no normativo).

**En SOFIA.** Decimal mayor o igual a 0, obligatorio para cerrar. Se compara con el efectivo esperado.

**Dónde aparece.** Sesiones de Caja: «Monto Cierre Declarado (S/)» (`I18N/sesiones-caja/es.json:33`) y el aviso «Ingresa el monto declarado al cierre» (`:24`). Reglas: [RN-CAJ-009](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Número de operación
*También aparece como:* «Nro Operación», «N° operación», «Referencia de Operación», `ReferenciaOperacion`, `Referencia_Operacion`.

**Definición.** Código que identifica una operación electrónica (transferencia, billetera, tarjeta) para poder rastrearla. Definición general, no verificada en fuente oficial.

**En SOFIA.** **Obligatorio para todo pago que no sea en efectivo**; hasta 100 caracteres; no es único (`DOM/VentaPago.cs:89`; `VentaPago.cs:30`). El POS deshabilita el cobro mientras falte.

**Dónde aparece.** POS: «Nro Operación» (`I18N/pos/es.json:186`) y «Ingresa el N° de operación para poder cobrar.» (`:188`); detalle de venta: «Referencia de Operación» (`:93`). Reglas: [RN-VEN-042](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Pago dividido
*También aparece como:* «+ Dividir en varios métodos de pago», «Volver a pago simple», «Agregar Método de Pago».

**Definición.** Pagar una misma venta con más de un método. Término propio de SOFIA (no normativo).

**En SOFIA.** La API acepta cualquier cantidad de pagos. El POS permite dividir y, en modo dividido, no deja repetir un método (fusiona los montos). La suma de pagos debe cubrir el monto a pagar; el vuelto solo puede salir de un pago en efectivo.

**Dónde aparece.** POS: «+ Dividir en varios métodos de pago» (`I18N/pos/es.json:222`), «Volver a pago simple» (`:223`), «Agregar Método de Pago» (`:185`). Reglas: [RN-VEN-043](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-044](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-045](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Punto de venta (POS)
*También aparece como:* «Terminal de Punto de Venta (POS)», «Punto de Venta» (grupo de menú), módulo de permisos `POS`.

**Definición.** Terminal desde el que se registran y cobran ventas. Término propio de SOFIA (no normativo).

**En SOFIA.** El grupo de menú «Punto de Venta» tiene dos opciones: «Sesiones de Caja» y «Ventas». El terminal de venta es la ruta `/pos/nueva` (botón de nueva venta). El permiso de módulo `POS` se rotula «Sesiones de Caja» y el de ventas se llama `Ventas`.

**Dónde aparece.** Menú (`I18N/es.json:56`, `breadcrumbs.puntoVenta`); título del terminal «Terminal de Punto de Venta (POS)» (`I18N/pos/es.json:3`). Inventario de pantallas: [19a](19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md).

### Reclamo de seguro
*También aparece como:* `VentaReclamoSeguro`, `Ventas_Reclamos_Seguro`, `EstadoReclamo`, `CodigoAutorizacion`, copago.

**Definición.** Registro de lo que una aseguradora cubre de una venta. Definición general, no verificada en fuente oficial.

**En SOFIA.** **Solo API.** Cuando una venta trae aseguradora y monto cubierto, el sistema crea **un** reclamo ligado a la **primera línea** de la venta, con estado `Aprobado` (texto fijo), el monto cubierto, el **copago** del paciente (`total − cobertura`) y un código de autorización `AUTH_` más 8 caracteres hexadecimales generados al azar (`APP/Ventas/Common/VentaSeguroProcessor.cs:18-24`). No hay integración con la aseguradora, ni pantalla de reclamos, ni otros estados.

**Dónde aparece.** Sin texto de interfaz. Reglas: [RN-VEN-047](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-048](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Reembolso
*También aparece como:* «Método de reembolso», «Reembolso estimado», «Reembolsado (S/)», `MontoReembolsado`, `MetodoReembolso`.

**Definición.** Dinero que se devuelve al cliente por una devolución. Definición general, no verificada en fuente oficial.

**En SOFIA.** El monto es la parte del cliente del crédito: `monto devuelto × (total de la venta − cobertura de seguro) / total de la venta`, redondeado a 2 decimales (`APP/Devoluciones/Commands/ProcesarDevolucion/ProcesarDevolucionCommandHandler.cs:141-155`). El sistema **solo lo registra, no mueve dinero**; si el método es efectivo, resta del efectivo esperado de la caja de quien procesa la devolución.

**Dónde aparece.** Diálogo de devolución: «Método de reembolso» y «Reembolso estimado» (`I18N/pos/es.json:124`, `:127`); detalle de venta: «Reembolsado (S/)» (`:135`). Reglas: [RN-DEV-017](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-DEV-018](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-DEV-019](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-CAJ-014](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Sesión de caja
*También aparece como:* «Sesiones de Caja», caja, «Nº Sesión», `PosSesionCaja`, `POS_Sesiones_Caja`.

**Definición.** Periodo de trabajo de un cajero con su caja, entre la apertura y el cierre. Término propio de SOFIA (no normativo).

**En SOFIA.** Una fila por turno con sucursal, empleado, hora de apertura y de cierre, monto de apertura, efectivo esperado, monto declarado, diferencia y estado. Cada venta cobrada queda ligada a la sesión con que se cobró. Un empleado solo puede tener **una sesión abierta** a la vez.

**Dónde aparece.** Menú «Punto de Venta» > «Sesiones de Caja» (`I18N/es.json:24`). Campos: «Sucursal», «Cajero», «Fecha / Hora Apertura», «Fecha / Hora Cierre», «Monto Apertura (S/)», «Monto Cierre Declarado (S/)», «Monto Cierre Calculado (S/)», «Diferencia Arqueo (S/)», «Estado» (`I18N/sesiones-caja/es.json:28-36`). Reglas: [RN-CAJ-001](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) a [RN-CAJ-027](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Venta
*También aparece como:* «Ventas», `Venta`, `Ventas_Cabecera`, transacción (`Transaccion_ID`).

**Definición.** Operación de entrega de productos a cambio de un pago. Definición general, no verificada en fuente oficial.

**En SOFIA.** Cabecera con sucursal, empleado (cajero), cliente opcional, sesión de caja, fecha, total bruto, estado y motivo de anulación; líneas por **lote** con cantidad y precio fijado; y pagos. El historial de la pantalla Ventas muestra solo las ventas de la **sucursal del usuario**. No hay descuentos. La moneda es el sol.

**Dónde aparece.** Menú «Punto de Venta» > «Ventas» (`I18N/es.json:25`); columnas «Código de Venta», «Fecha», «Cliente», «Empleado», «Total (S/)», «Estado», «Ítems» (`I18N/pos/es.json:68-74`). Reglas: [RN-VEN-001](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) a [RN-VEN-083](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) (ver el catálogo completo).

### Venta completada
*También aparece como:* `EstadoVenta.Completada`, «Completada», «Cobrar Venta», «Venta Procesada Exitosamente».

**Definición.** Venta cobrada. Término propio de SOFIA (no normativo).

**En SOFIA.** Una venta pasa a `Completada` cuando se registran los pagos que cubren el monto a pagar (`DOM/Venta.cs:262`). En ese momento se emite la boleta, se descuenta stock (si no estaba reservado en espera) y se cuenta en el arqueo. Es el único estado desde el que se puede devolver.

**Dónde aparece.** POS: «Procesar Venta» (`I18N/pos/es.json:11`), «Venta Procesada Exitosamente» (`:239`), «Venta cobrada correctamente.» (`:336`). Reglas: [RN-VEN-046](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-050](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Venta en espera (pendiente)
*También aparece como:* «Ventas en espera», «Cobrar Venta en Espera», «Actualizar Venta en Espera», «Guardar y Atender Siguiente», `EstadoVenta.Pendiente`, venta pendiente.

**Definición.** Venta que se guarda sin cobrar para atender a otro cliente y retomarla después. Término propio de SOFIA (no normativo).

**En SOFIA.** Se guarda con estado `Pendiente`: **reserva el stock** y cuenta como dispensación de la receta, pero **no tiene pagos ni comprobante** y su código es provisional. No caduca ni se libera sola: solo se cobra o se anula. Puede retomarla cualquier usuario de la sucursal con su propia caja abierta. El dashboard cuenta las ventas en espera y su importe «Por cobrar».

**Dónde aparece.** POS: «Guardar y Atender Siguiente» (`I18N/pos/es.json:17`), «Venta guardada en espera. Retómala desde Ventas cuando el cliente vuelva.» (`:335`), «Cobrar Venta en Espera» (`:268`). Dashboard: «Ventas en espera» y «Por cobrar» (`I18N/dashboard/es.json:25-26`). Reglas: [RN-VEN-052](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md) a [RN-VEN-056](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CAJ-017](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-DAS-009](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Vuelto
*También aparece como:* «Vuelto a Entregar:», «Vuelto Entregado:», «Vuelto:».

**Definición.** Dinero que se devuelve al cliente cuando entrega más efectivo que el monto a pagar. Definición general, no verificada en fuente oficial.

**En SOFIA.** `Vuelto = suma de pagos − monto a pagar`. Solo puede haber vuelto si hay un pago en efectivo, y no puede superar el efectivo entregado (`DOM/Venta.cs:244-254`). **No se guarda en ninguna columna**: el pago en efectivo se registra por lo que entregó el cliente y el vuelto se descuenta al calcular el arqueo y los totales por método de pago.

**Dónde aparece.** POS: «Vuelto a Entregar:» (`I18N/pos/es.json:199`), «Vuelto Entregado:» (`:204`), «Vuelto:» (`:221`). Reglas: [RN-VEN-044](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-CAJ-013](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

## 6. Inventario y catálogo

Mapa de la familia: un **medicamento** pertenece a un **laboratorio**, tiene una **unidad base** y una **condición de venta**. Sus **lotes** (con caducidad) se reparten en **stock por sucursal**. La **jerarquía de unidades** dice cuántas unidades menores hay en una mayor y las **presentaciones de venta** son las formas en que se vende (caja, blíster) con su propio precio.

### Ajuste de stock
*También aparece como:* «Ajustar cantidad», `AdjustStock`, `EsAjusteDirecto`, evento `STOCK_AJUSTAR`.

**Definición.** Corrección de la cantidad registrada de un producto para que coincida con el recuento físico. Definición general, no verificada en fuente oficial.

**En SOFIA.** Un ajuste **fija** la cantidad física al valor indicado (no suma), no admite negativos y deja un evento de auditoría `STOCK_AJUSTAR`. Solo se puede ajustar el stock de sucursales a las que el usuario tiene acceso. La API también admite un «ajuste directo» al ingresar stock; la pantalla nunca lo envía, así que desde el SPA un ingreso siempre suma.

**Dónde aparece.** Stock por Sucursal, detalle: «Ajustar cantidad» (`I18N/stock-por-sucursal/es.json:10`); «Cantidad ajustada correctamente» (`:12`). Reglas: [RN-STK-004](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-STK-006](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-STK-008](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-STK-012](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Aseguradora
*También aparece como:* «Aseguradoras», «Aseguradoras Médicas», seguro, `AseguradoraMedica`, `Aseguradoras_Medicas`, módulo `Seguros`.

**Definición.** Empresa que cubre total o parcialmente el costo de los medicamentos de sus asegurados. Definición general, no verificada en fuente oficial.

**En SOFIA.** Maestro con **nombre comercial** (obligatorio, hasta 150 caracteres) y **código identificador nacional** (obligatorio, único por empresa). Se usa en la cobertura de seguro de una venta (hoy oculta en el POS). El permiso de módulo se llama `Seguros`.

**Dónde aparece.** Menú «Catálogos» > «Aseguradoras» (`I18N/es.json:13`); título «Aseguradoras Médicas» (`I18N/aseguradoras/es.json:2`); campos «Nombre Comercial» y «Código Identificador Nacional» (`:20-21`). Reglas: [RN-ASE-001](01-reglas-de-negocio/01b-catalogos.md) a [RN-ASE-012](01-reglas-de-negocio/01b-catalogos.md).

### Código nacional
*También aparece como:* «Código Nacional», `CodigoNacional`, `Codigo_Nacional`.

**Definición.** Código con el que se identifica un medicamento en el catálogo. Definición general, no verificada en fuente oficial.

**En SOFIA.** Dato obligatorio del **medicamento**: hasta 50 caracteres, **único por empresa** entre los no eliminados (`DOM/Medicamento.cs:24`). Lo asigna el usuario; SOFIA no lo valida contra el catálogo DIGEMID (cuyo código se llama «Código de Producto»).

**Dónde aparece.** Medicamentos: «Código Nacional» (`I18N/medicamentos/es.json:30`). Mensaje de duplicado: «Ya existe un medicamento con ese código nacional.» (`:13`). Reglas: [RN-MED-002](01-reglas-de-negocio/01b-catalogos.md), [RN-MED-003](01-reglas-de-negocio/01b-catalogos.md), [RN-MED-010](01-reglas-de-negocio/01b-catalogos.md).

### Equivalente terapéutico
*También aparece como:* «Equivalentes terapéuticos», «Sugerencias IA».

**Definición.** Medicamento con el mismo principio activo, concentración y forma de dosis que otro. Definición general, no verificada en fuente oficial.

**En SOFIA.** Dos medicamentos de la empresa son equivalentes si comparten **formulación clínica**: mismo ingrediente activo, misma concentración y misma unidad de medida. Se muestran solo en el análisis de receta con IA, con su laboratorio y su stock.

**Dónde aparece.** Lector de Recetas IA: «Equivalentes terapéuticos» y «No hay stock del medicamento prescrito, pero hay equivalentes disponibles.» (`I18N/recetas-medicas/es.json:73-74`). Reglas: [RN-REC-027](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Estado de la transferencia (estado logístico)
*También aparece como:* «Estado Logístico», `EstadoLogistico`, `Estado_Logistico`.

**Definición.** Situación de una transferencia de stock entre sucursales. Término propio de SOFIA (no normativo).

**En SOFIA.** **Solo API (sin pantalla).** Seis valores de texto con CHECK `CHK_Estado_Logistico` (`ENUM/EstadoLogistico.cs:5-10`; `DBUP/0001_Initial_Schema.sql:201`). Los textos de interfaz existen en un archivo de traducciones que ninguna pantalla usa:

| Valor | Texto en `transferencias/es.json` | Cómo se llega |
|---|---|---|
| `Iniciada` | «Iniciada» | Al crear la transferencia |
| `Aprobada` | «Aprobada» | **No se alcanza por la API**: el método existe en la entidad pero ningún endpoint lo llama ([RN-TRA-011](01-reglas-de-negocio/01d-modulos-solo-api.md)) |
| `En_Transito` | «En Tránsito» | Al despachar (descuenta el stock del origen) |
| `Recibida_Parcial` | «Recibida Parcial» | Al recibir con alguna cantidad menor a la enviada |
| `Completada` | «Recibida Completa» | Al recibir todo lo enviado |
| `Cancelada` | «Cancelada» | Al cancelar desde `Iniciada`, `Aprobada` o `En_Transito` |

(`DOM/Transferencia.cs:67`, `:87`, `:98`, `:146`; textos en `I18N/transferencias/es.json:22-27`, `status.*`.) Recibida Parcial y Cancelada son finales: la diferencia de una recepción parcial **se pierde** (no vuelve al origen); cancelar una transferencia `En_Transito` devuelve el stock al origen.

**Dónde aparece.** Solo API. Reglas: [RN-TRA-010](01-reglas-de-negocio/01d-modulos-solo-api.md) a [RN-TRA-024](01-reglas-de-negocio/01d-modulos-solo-api.md).

### Historial de precios (de proveedor)
*También aparece como:* `HistorialPrecioProveedor`, `Historial_Precios_Prov`, «Vigente desde», «Vigente hasta».

**Definición.** Registro cronológico de lo que cada proveedor ha cobrado por un producto. Término propio de SOFIA (no normativo).

**En SOFIA.** Cada fila une un proveedor, un producto, un **costo por unidad base**, una vigencia desde/hasta, un plazo de entrega y una cantidad mínima de compra (`DOM/HistorialPrecioProveedor.cs:9-15`). Un producto tiene **un solo precio vigente por proveedor**: al registrar uno nuevo, el abierto se cierra en ese mismo instante (`APP/Proveedores/Commands/RegistrarPrecioProveedor/RegistrarPrecioProveedorCommandHandler.cs:34-46`). El plazo de entrega se guarda en 0 y la cantidad mínima en 1 (la pantalla no los pide). El costo vigente se toma como costo histórico de las líneas de venta ([RN-VEN-022](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md)).

**Dónde aparece.** Proveedores, detalle, sección «Precios de compra» con columnas «Producto», «Costo por unidad base», «Vigente desde», «Vigente hasta» y la marca «Vigente» (`I18N/proveedores/es.json:104-114`). Reglas: [RN-PRV-018](01-reglas-de-negocio/01b-catalogos.md) a [RN-PRV-024](01-reglas-de-negocio/01b-catalogos.md). Esquema: [03a §7.2](03-modelo-de-datos/03a-maestros-y-seguridad.md).

### Ingreso de stock
*También aparece como:* «Nuevo Registro de Stock», «Stock registrado correctamente», `RegisterInventario`.

**Definición.** Alta de unidades de un lote en una sucursal. Término propio de SOFIA (no normativo).

**En SOFIA.** Se elige sucursal y lote y se indica la cantidad (≥ 0). Si ya hay stock de ese lote en esa sucursal, **se suma** a la existente; si no, se crea la fila. Crear un lote **no** crea stock. Solo se puede ingresar a sucursales accesibles para el usuario.

**Dónde aparece.** Stock por Sucursal > «Nuevo Registro de Stock» (`I18N/stock-por-sucursal/es.json:17`); campos «Sucursal», «Lote», «Cantidad Física» (`:22`, `:32`, `:28`). Reglas: [RN-STK-002](01-reglas-de-negocio/01c1-inventario-y-recetas.md) a [RN-STK-010](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Jerarquía de unidades
*También aparece como:* «Jerarquías de Unidades», «Jerarquía UoM», UoM, `JerarquiaUoM`, `Jerarquia_UoM`.

**Definición.** Regla que dice cuántas unidades menores contiene una unidad mayor de un producto (por ejemplo, una caja de 30 tabletas). Término propio de SOFIA (no normativo).

**En SOFIA.** Une un **producto**, una **unidad mayor**, una **unidad menor** y un **multiplicador**. Las dos unidades deben ser distintas; si ya existen conversiones que determinan la equivalencia, la nueva debe coincidir con la calculada. De las jerarquías salen las **unidades vendibles** de un producto y la equivalencia de cada presentación ([RN-JER-011](01-reglas-de-negocio/01b-catalogos.md)).

**Dónde aparece.** Menú «Catálogos» > «Jerarquías de Unidades» (`I18N/es.json:9`). Campos «Producto», «Unidad Mayor», «Unidad Menor», «Multiplicador» (`I18N/jerarquias/es.json:23-26`). Reglas: [RN-JER-001](01-reglas-de-negocio/01b-catalogos.md) a [RN-JER-022](01-reglas-de-negocio/01b-catalogos.md).

### Medicamento (producto)
*También aparece como:* «Medicamentos», «Producto», «Medicamento / Insumo», `Medicamento`, `Medicamentos`, `ProductoId`.

**Definición.** Producto farmacéutico que la empresa vende. Los productos farmacéuticos se identifican en el país con su registro sanitario ante DIGEMID. Fuente oficial: https://www.digemid.minsa.gob.pe/webDigemid/registro-sanitario/productos-farmaceuticos/

**En SOFIA.** Maestro con **código nacional**, **nombre comercial** (hasta 150 caracteres), **laboratorio**, **unidad base**, **condición de venta** y **precio de venta base** (no negativo, 2 decimales; puede estar vacío en registros antiguos o de carga masiva, y entonces no se puede vender sin presentación) (`DOM/Medicamento.cs:11-18`). Un medicamento no guarda su registro sanitario, forma farmacéutica ni concentración (esos datos están en el catálogo DIGEMID y en la formulación clínica). **Medicamento** y **producto** son la misma fila: la documentación usa «medicamento» para el maestro y «producto» solo cuando el rótulo de la pantalla lo dice.

**Dónde aparece.** Menú «Catálogos» > «Medicamentos» (`I18N/es.json:18`). Campos: «Código Nacional», «Nombre Comercial», «Laboratorio», «Unidad Base», «Condición de Venta», «Precio de Venta Base», «Stock Total» (`I18N/medicamentos/es.json:30-36`). Reglas: [RN-MED-001](01-reglas-de-negocio/01b-catalogos.md) a [RN-MED-026](01-reglas-de-negocio/01b-catalogos.md).

### Multiplicador
*También aparece como:* «Multiplicador», `Multiplicador`.

**Definición.** Cantidad de unidades menores contenidas en una unidad mayor. Término propio de SOFIA (no normativo).

**En SOFIA.** Decimal mayor que cero con 4 decimales (`DBUP/0020_SoftDeleteAware_Uniques_And_Checks.sql:123-124`, CHECK `> 0`). Ejemplo de la ayuda: «Si una caja tiene 30 tabletas, el multiplicador es 30».

**Dónde aparece.** Jerarquías de Unidades: «Multiplicador» y la ayuda «Cuántas unidades menores equivalen a una unidad mayor» (`I18N/jerarquias/es.json:18`, `:26`). Reglas: [RN-JER-005](01-reglas-de-negocio/01b-catalogos.md), [RN-JER-009](01-reglas-de-negocio/01b-catalogos.md).

### Precio de compra
*También aparece como:* «Precios de compra», «Registrar precio», «Costo por unidad base», `CostoPorUnidadBase`, costo neto.

**Definición.** Lo que el proveedor cobra a la farmacia por una unidad de producto. Definición general, no verificada en fuente oficial.

**En SOFIA.** Se registra por proveedor y producto: precio de compra por unidad base (mayor que 0) y un descuento de 0 a 100 %; el **costo neto** es `precio × (1 − descuento / 100)` redondeado a 4 decimales (`APP/Proveedores/Commands/RegistrarPrecioProveedor/RegistrarPrecioProveedorCommandHandler.cs:26`) y es el valor que se guarda. **Aviso:** los documentos `01b` ([RN-PRV-020](01-reglas-de-negocio/01b-catalogos.md)) y `03a` anteriores dicen que el registro «no guarda nada»; el código actual (commit `43f5529`) sí guarda y cierra el precio anterior. Manda el código.

**Dónde aparece.** Proveedores > «Registrar precio»: «Precio de compra por unidad base (S/)», «Descuento (%)» y «Costo neto por unidad base» (`I18N/proveedores/es.json:106`, `:122-124`). Permiso «Registrar precio de proveedor» (`I18N/roles/es.json`, `permisos.accion.RegistrarPrecioProveedor`). Reglas: [RN-PRV-018](01-reglas-de-negocio/01b-catalogos.md), [RN-PRV-019](01-reglas-de-negocio/01b-catalogos.md).

### Precio de venta base
*También aparece como:* «Precio de Venta Base», `PrecioVentaBase`, `Precio_Venta_Base`.

**Definición.** Precio de venta al público de una unidad base del medicamento. Término propio de SOFIA (no normativo).

**En SOFIA.** Precio **por unidad base**, con IGV incluido (ver «IGV»). Cuando la línea de venta no usa presentación, el precio unitario es este; el precio se congela en la línea de venta. Una presentación de venta tiene precio propio, no derivado de este.

**Dónde aparece.** Medicamentos: «Precio de Venta Base» (`I18N/medicamentos/es.json:35`) y el error «El precio no puede ser negativo» (`:27`). POS: «Este producto no tiene precio de venta configurado en el catálogo. Pide a un administrador que lo registre.» (`I18N/pos/es.json:334`). Reglas: [RN-MED-008](01-reglas-de-negocio/01b-catalogos.md), [RN-MED-009](01-reglas-de-negocio/01b-catalogos.md), [RN-VEN-019](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Presentación de venta
*También aparece como:* «Presentaciones de Venta», «Unidad de Venta», «Equivalencia», `PresentacionVenta`, `Presentaciones_Venta`.

**Definición.** Forma en que se vende un producto (caja, blíster, unidad) con su equivalencia en unidades base y su precio. Término propio de SOFIA (no normativo).

**En SOFIA.** Une un **producto**, una **unidad de venta**, una **equivalencia** en unidades base (la calcula el sistema desde las jerarquías y no se recalcula si estas cambian) y un **precio de venta** propio (≥ 0). Un producto tiene como máximo una presentación por unidad de venta. Para crear una, el producto debe tener una jerarquía de unidades. Al vender, la cantidad se ingresa en presentaciones y el stock se descuenta en unidades base. No se puede eliminar una presentación con ventas.

**Dónde aparece.** Menú «Catálogos» > «Presentaciones de Venta» (`I18N/es.json:19`). Campos «Producto», «Unidad de Venta», «Precio de Venta» y columna «Equivalencia» (`I18N/presentaciones-venta/es.json:45-48`, `:53`). Reglas: [RN-PRE-001](01-reglas-de-negocio/01b-catalogos.md) a [RN-PRE-015](01-reglas-de-negocio/01b-catalogos.md), [RN-VEN-017](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-018](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Próximo a vencer
*También aparece como:* «Lotes que vencen en 30 días o menos», «31 a 60 días», «61 a 90 días».

**Definición.** Lote cuya fecha de caducidad se acerca. Término propio de SOFIA (no normativo).

**En SOFIA.** No hay una alerta propia: el **dashboard** cuenta las filas de stock con cantidad > 0 en tres tramos de caducidad desde hoy: hasta 30 días, de 31 a 60 y de 61 a 90 (`APP/Dashboard/Queries/GetDashboardInventario/GetDashboardInventarioQueryHandler.cs:26-41`). Un lote que vence hoy aún no está vencido. Los lotes ya vencidos con stock se cuentan aparte.

**Dónde aparece.** Dashboard, sección Inventario: «Lotes que vencen en 30 días o menos», «Lotes que vencen en 31 a 60 días» y «Lotes que vencen en 61 a 90 días» (`I18N/dashboard/es.json:65-67`). Reglas: [RN-DAS-018](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [RN-DAS-019](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Proveedor
*También aparece como:* «Proveedores», distribuidor, `ProveedorDistribuidor`, `Proveedores_Dist`.

**Definición.** Empresa que abastece de productos a la farmacia. Definición general, no verificada en fuente oficial.

**En SOFIA.** Maestro con **razón social** (única por empresa, hasta 200 caracteres), **RUC / Tax ID** (único), términos financieros (opcional), calificación ESG (0 a 100, opcional) y **tasa de cumplimiento** (0 a 100, por defecto 100) (`DOM/ProveedorDistribuidor.cs:9-13`). Sus precios de compra forman su historial. No se puede eliminar un proveedor que tiene precios registrados. SOFIA no registra órdenes de compra.

**Dónde aparece.** Menú «Catálogos» > «Proveedores» (`I18N/es.json:12`). Campos: `I18N/proveedores/es.json:22-26`. Reglas: [RN-PRV-001](01-reglas-de-negocio/01b-catalogos.md) a [RN-PRV-017](01-reglas-de-negocio/01b-catalogos.md).

### Stock bajo
*También aparece como:* «Productos con stock bajo», «Stock bajo» (etiqueta del POS), `UmbralStockBajo`.

**Definición.** Producto cuyas existencias están por debajo de un mínimo definido. Definición general, no verificada en fuente oficial.

**En SOFIA.** El umbral es **fijo: 10 unidades base vendibles o menos, por sucursal y producto** (`APP/Dashboard/Queries/GetDashboardInventario/GetDashboardInventarioQueryHandler.cs:14`); no es configurable por producto. La pantalla Stock por Sucursal filtra por «Cantidad Mínima» y «Cantidad Máxima», que son filtros de consulta y no un punto de reposición.

**Dónde aparece.** Dashboard: «Productos con stock bajo» con la ayuda «{{umbral}} unidades o menos, por sucursal» (`I18N/dashboard/es.json:69-70`); POS, catálogo: «Stock bajo» (`I18N/pos/es.json:218`). Reglas: [RN-DAS-021](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Stock por sucursal
*También aparece como:* «Stock por Sucursal», «Cantidad Física», `InventarioSucursal`, `Inventario_Sucursal`, `LotesEnSucursal`, existencias.

**Definición.** Cantidad de cada lote disponible en cada sucursal. Término propio de SOFIA (no normativo).

**En SOFIA.** **Una fila por lote y sucursal**; guarda la **cantidad física** en unidades base, nunca negativa (CHECK `CHK_Inventario_Positivo`, `DBUP/0001_Initial_Schema.sql:180`), con una versión de fila contra cambios simultáneos. El listado muestra solo las sucursales accesibles del usuario y, por defecto, solo filas con stock. Una venta, una transferencia, un ajuste o una orden magistral cambian esa cantidad.

**Dónde aparece.** Menú «Inventario» > «Stock por Sucursal» (`I18N/es.json:23`). Filtros y columnas: «Sucursal», «Producto», «Número de Lote», «Fecha de Caducidad», «Cantidad Física», «Solo con stock disponible» (`I18N/stock-por-sucursal/es.json:22-31`). Reglas: [RN-STK-001](01-reglas-de-negocio/01c1-inventario-y-recetas.md) a [RN-STK-018](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Stock vendible
*También aparece como:* stock disponible, «Stock Disp:», «Disponible», «Agotado».

**Definición.** Unidades que realmente se pueden vender. Término propio de SOFIA (no normativo).

**En SOFIA.** Es el stock de la sucursal con cantidad > 0, de un **lote no vencido** y que **no esté retenido en cuarentena** (`APP/Inventarios/Common/InventarioVendibleExtensions.cs:17-24`). Lo usan el POS, el dashboard y la lista de medicamentos. El análisis de recetas con IA usa una definición más simple (solo lotes no vencidos).

**Dónde aparece.** POS, catálogo: «Stock Disp:», «Disponible», «Agotado» (`I18N/pos/es.json:151`, `:217`, `:219`). Reglas: [RN-STK-017](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-REC-026](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Transferencia entre sucursales
*También aparece como:* «Transferencias», «Solicitar Transferencia entre Sedes», `Transferencia`, `Transferencias_Cab`.

**Definición.** Traslado de stock de un lote de una sucursal a otra de la misma empresa. Término propio de SOFIA (no normativo).

**En SOFIA.** **Solo API (sin pantalla).** La sucursal de origen y el emisor salen del token; el destino debe ser distinto. Pasos: crear (`Iniciada`), **despachar** (el personal del origen; descuenta el stock del origen y pasa a `En_Transito`), **recibir** (el personal del destino; solo entra lo recibido) y **cancelar** (el personal del origen). Una recepción parcial exige indicar el motivo (merma, rotura, faltante) en la pantalla prevista. Estados: ver «Estado de la transferencia». Los textos «Solicitar Transferencia entre Sedes» del POS existen en `pos/es.json`, pero ningún componente del SPA los usa (búsqueda de `transferDialog` sin resultados en `sofia-web-spa/src`).

**Dónde aparece.** Textos del POS: «Solicitar Transferencia entre Sedes», «Sucursal de Origen (Sede Proveedora)» (`I18N/pos/es.json:227-228`). Permisos del módulo «Transferencias» (`I18N/roles/es.json`, `permisos.modulo.Transferencias`). Reglas: [RN-TRA-001](01-reglas-de-negocio/01d-modulos-solo-api.md) a [RN-TRA-030](01-reglas-de-negocio/01d-modulos-solo-api.md).

### Unidad base
*También aparece como:* «Unidad Base», `UnidadBaseId`, `Unidad_Base_ID`.

**Definición.** Unidad más pequeña en la que se cuenta un producto. Término propio de SOFIA (no normativo).

**En SOFIA.** Unidad de medida elegida por cada medicamento. **El stock, el precio base y el descuento de stock de las ventas se expresan siempre en unidades base**; las presentaciones y las jerarquías convierten a ella.

**Dónde aparece.** Medicamentos: «Unidad Base» (`I18N/medicamentos/es.json:33`). Reglas: [RN-MED-006](01-reglas-de-negocio/01b-catalogos.md), [RN-VEN-015](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [RN-VEN-017](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md).

### Unidad de medida
*También aparece como:* «Unidades de Medida», UoM, `UnidadMedida`, `Unidades_Medida`.

**Definición.** Catálogo de unidades en que se expresan cantidades (tableta, caja, mililitro). Definición general, no verificada en fuente oficial.

**En SOFIA.** Maestro con **código** (abreviatura única por empresa, hasta 10 caracteres; ejemplos de la ayuda: KG, G, UN, LT) y **descripción** (hasta 50) (`DOM/UnidadMedida.cs:9-10`). Se usa como unidad base de medicamentos, unidades de jerarquías, unidad de venta de presentaciones y unidad de dosis de formulaciones; no se puede eliminar si está en uso.

**Dónde aparece.** Menú «Catálogos» > «Unidades de Medida» (`I18N/es.json:8`). Campos «Código» y «Descripción» (`I18N/unidadesMedida/es.json:19-20`). Reglas: [RN-UDM-001](01-reglas-de-negocio/01b-catalogos.md) a [RN-UDM-017](01-reglas-de-negocio/01b-catalogos.md).

## 7. Organización, seguridad y plataforma

Mapa de la familia: una **empresa** (cliente de SOFIA, aislado de las demás) tiene **sucursales**; sus **empleados** tienen una **sucursal base**; una **cuenta de usuario** liga un empleado a un usuario y contraseña; los **roles** reúnen **permisos** (módulo y acción) y pueden limitar **sucursales**. El rol `Admin` lo puede todo.

### Auditoría
*También aparece como:* «Auditoría de Seguridad», «Auditoría del Sistema», `AuditoriaEventoSeguridad`, `Auditoria_Eventos_Seguridad`, `IAuditableCommand`.

**Definición.** Registro de quién hizo qué y cuándo en el sistema. Término propio de SOFIA (no normativo).

**En SOFIA.** Cuando un comando auditable termina con éxito, el sistema guarda un evento con el empleado que actúa, la tabla afectada, el id del registro, el tipo de acción (por ejemplo `CAJA_CERRAR`, `CUENTA_ELIMINAR`, `STOCK_AJUSTAR`), un detalle breve sin datos personales y la IP (`APP/Common/Behaviors/AuditBehavior.cs:10-30`). Los eventos son de solo agregar. **No existe ninguna consulta, endpoint ni pantalla para leerlos**, y la operación «Anonimizar datos» responde 501 sin tocar datos ([RN-AUD-003](01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-AUD-005](01-reglas-de-negocio/01d-modulos-solo-api.md)). No se auditan sucursales, empleados, inicios de sesión ni cargas masivas de roles por este mecanismo.

**Dónde aparece.** No hay pantalla. Hay textos huérfanos en `auditoria/es.json` («Auditoría del Sistema») y el módulo de permisos «Auditoría de Seguridad» (`I18N/roles/es.json:58`, `permisos.modulo.Auditoria`) con la acción «Anonimizar datos». Reglas: [RN-AUD-001](01-reglas-de-negocio/01d-modulos-solo-api.md) a [RN-AUD-010](01-reglas-de-negocio/01d-modulos-solo-api.md), [RN-GEN-013](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Bloqueo de cuenta
*También aparece como:* «Bloqueado», «Bloqueado Hasta», «Intentos Fallidos», «Resetear Intentos Fallidos», `BloqueadoHasta`, `IntentosFallidos`.

**Definición.** Impedimento temporal de iniciar sesión tras varios intentos fallidos. Término propio de SOFIA (no normativo).

**En SOFIA.** Al llegar a **5 intentos fallidos** de contraseña, la cuenta queda bloqueada **15 minutos** (`DOM/Cuenta.cs:86-88`). Un inicio correcto pone los intentos en 0 y quita el bloqueo. Un administrador puede resetear los intentos desde la cuenta.

**Dónde aparece.** Cuentas de Usuario: «Bloqueado», «Intentos Fallidos», «Bloqueado Hasta» y la acción «Resetear Intentos Fallidos» (`I18N/cuentas/es.json:25-28`, `:34`). Reglas: [RN-AUT-005](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-AUT-007](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-CTA-011](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Borrado lógico
*También aparece como:* eliminar, `IsDeleted`, `DeletedAt`, `DeletedBy`, soft delete.

**Definición.** Marcar un registro como eliminado sin borrarlo físicamente. Término propio de SOFIA (no normativo).

**En SOFIA.** Casi todas las entidades heredan de `BaseEntity`: «Eliminar» marca `IsDeleted` y las consultas dejan de verlo (filtro global). El nombre o código de un registro eliminado **queda libre** para volver a usarse, porque las claves únicas ignoran las filas eliminadas. Varios maestros bloquean la eliminación si tienen dependencias (por ejemplo, un laboratorio con medicamentos o una serie con comprobantes). Mensajes como «No puedes eliminar este paciente porque tiene recetas, ventas u otros registros asociados.» vienen de esa validación.

**Dónde aparece.** Botón «Eliminar» de cada detalle en modo edición y su diálogo «Confirmar eliminación». Reglas: [RN-GEN-002](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-003](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-UDM-007](01-reglas-de-negocio/01b-catalogos.md), [RN-LAB-011](01-reglas-de-negocio/01b-catalogos.md).

### Cambio de contraseña obligatorio
*También aparece como:* «Requiere Cambio de Clave», «Forzar Cambio de Contraseña», `RequiereCambioClave`, `Requiere_Cambio_Clave`.

**Definición.** Obligación de cambiar la contraseña en el siguiente ingreso. Término propio de SOFIA (no normativo).

**En SOFIA.** Un administrador puede encender el indicador de una cuenta (solo se puede encender; se apaga cuando el usuario cambia o restablece su clave). Mientras esté encendido, la API rechaza cualquier petición salvo el cambio de clave ([RN-GEN-019](01-reglas-de-negocio/01a-administracion-y-seguridad.md)) y el SPA lleva al usuario a la pantalla de cambio con el aviso «Tu contraseña fue asignada por un administrador. Debes cambiarla antes de continuar.» Al cambiarla se cierran todas las sesiones. La contraseña nueva debe tener 8 caracteres o más y ser distinta de la actual.

**Dónde aparece.** Pantalla «Cambiar contraseña» (`I18N/cambiar-clave/es.json:4`, `forced`); Cuentas de Usuario: «Forzar Cambio de Contraseña» (`I18N/cuentas/es.json:33`). Reglas: [RN-AUT-024](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-AUT-030](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-CTA-011](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-019](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Carga masiva
*También aparece como:* «Carga Masiva», «Subir archivo», `carga-masiva`, `previsualizar`.

**Definición.** Alta de muchos registros a la vez desde un archivo Excel. Término propio de SOFIA (no normativo).

**En SOFIA.** Flujo de tres pasos: **descargar la plantilla**, **subir el archivo** (solo `.xlsx`; máx. 5 MB y 5 000 filas; se lee la primera hoja) y **revisar la vista previa**, que marca cada fila como «Válido» o «Error» antes de **guardar solo las válidas**. Al guardar, cada fila se valida otra vez. Está disponible solo en los **maestros** (12 pantallas): Aseguradoras, Catálogo DIGEMID, Ingredientes Activos, Jerarquías de Unidades, Laboratorios, Medicamentos, Pacientes, Profesionales de Salud, Proveedores, Roles, Sucursales y Unidades de Medida (`docs/_inventario/rutas-spa.csv`, rutas `.../carga-masiva`). Exige el permiso `Crear` del módulo.

**Dónde aparece.** Botón «Carga Masiva» en la pantalla de búsqueda y pantalla «Carga Masiva» con «Descargar plantilla», «Subir archivo», «Vista previa» y «Guardar válidos ({{count}})» (`I18N/es.json:218-232`, `carga-masiva.*`). Reglas: [RN-IMP-001](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-IMP-007](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-012](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-IMP-016](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Cuenta de usuario
*También aparece como:* «Cuentas de Usuario», «Nombre de Usuario», «Cuenta Activa», `Cuenta`, `Seguridad_Cuentas`.

**Definición.** Credencial (usuario y contraseña) con la que un empleado entra al sistema. Término propio de SOFIA (no normativo).

**En SOFIA.** Un empleado tiene **a lo sumo una cuenta**. El **nombre de usuario es único en todo el sistema** (no solo en la empresa), hasta 50 caracteres. La contraseña se guarda con hash BCrypt y debe tener 8 caracteres o más. Solo se puede cambiar el estado activo, forzar el cambio de clave y resetear los intentos; no se pueden cambiar el usuario ni el empleado. Una cuenta no puede modificar sus propios roles, y solo un Admin puede tocar la cuenta de otro Admin. Eliminarla es un borrado lógico.

**Dónde aparece.** Menú «Administración» > «Cuentas de Usuario» (`I18N/es.json:17`). Campos: «Nombre de Usuario», «Empleado», «Cuenta Activa», «Roles Asignados», «Sucursales Asignadas» (`I18N/cuentas/es.json:22-32`). Inventario: [19b](19-inventario-de-pantallas/19b-administracion.md). Reglas: [RN-CTA-001](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-CTA-020](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Dashboard
*También aparece como:* «Dashboard», pantalla de inicio, «Hola, {{nombre}}», «Bienvenido a SOFIA».

**Definición.** Pantalla de inicio con indicadores del día. Término propio de SOFIA (no normativo).

**En SOFIA.** Seis bloques: Ventas de hoy, Caja, Inventario, Ventas de los últimos 7 días, Productos más vendidos en 7 días y Devoluciones de hoy. Cada sección se pide y falla por separado y se muestra solo si el usuario tiene el permiso de lectura correspondiente. Un administrador ve el total de la empresa con desglose por sucursal; los demás ven el total de sus sucursales accesibles y sus propias ventas. Los importes de venta son **brutos**: las devoluciones se muestran aparte.

**Dónde aparece.** Menú «Principal» > «Dashboard» (`I18N/es.json:3`); saludo y secciones en `I18N/dashboard/es.json` (por ejemplo «Ventas de hoy», línea 17). Inventario: [19a](19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md). Reglas: [RN-DAS-001](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md) a [RN-DAS-024](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md).

### Empleado
*También aparece como:* «Empleados», `Empleado`, `Empleados`.

**Definición.** Persona que trabaja en la empresa. Término propio de SOFIA (no normativo).

**En SOFIA.** Maestro con **nombres**, **apellido paterno**, **apellido materno** (opcional), **licencia profesional** (opcional, única por empresa), **correo electrónico** (opcional, único por empresa; sirve para enviar el enlace de recuperación de contraseña) y **sucursal base** (obligatoria). El nombre completo se arma uniendo las partes que existan. No se puede eliminar un empleado que gerencia una sucursal. Los empleados registrados como **cajero**, **regente** o **químico preparador** son estos mismos registros.

**Dónde aparece.** Menú «Administración» > «Empleados» (`I18N/es.json:6`). Campos: `I18N/empleados/es.json:22-28`. Reglas: [RN-EPL-001](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-EPL-018](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Empresa (tenant)
*También aparece como:* «Empresa», «Empresas», compañía, `Empresa`, `TenantId`, multiempresa.

**Definición.** Cliente de SOFIA: un negocio farmacéutico con su propio RUC, sucursales y datos. Término propio de SOFIA (no normativo).

**En SOFIA.** SOFIA es multiempresa: casi todas las tablas llevan `TenantId` (el id de la empresa) y toda consulta se filtra por la empresa del usuario, de modo que **una empresa nunca ve los datos de otra** ([RN-GEN-001](01-reglas-de-negocio/01a-administracion-y-seguridad.md)). La empresa tiene nombre (hasta 200 caracteres), RUC, estado de suscripción, fechas de prueba y vencimiento y la política «exigir registro de receta» (solo editable en la base de datos). Se puede **registrar** una empresa sin sesión; los usuarios editan solo la propia.

**Dónde aparece.** Menú «Administración» > «Empresa» (`I18N/es.json:4`); registro público «Registrar Empresa» (`I18N/empresa/es.json:6`). Modelo: [03a §3.1](03-modelo-de-datos/03a-maestros-y-seguridad.md). Reglas: [RN-EMP-001](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-EMP-018](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-001](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-004](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Estados de suscripción
*También aparece como:* «Trial Activo», «Activo», «Suspendido», «Cancelado», «Vigente», `EstadoEmpresa`, `TrialActivo`.

**Definición.** Situación del contrato de una empresa con SOFIA. Término propio de SOFIA (no normativo).

**En SOFIA.** Enum numérico de la entidad (`DOM/Empresa.cs:6-12`):

| Valor | Número | Texto en la interfaz | ¿Está vigente? |
|---|---|---|---|
| `TrialActivo` | 0 | «Trial Activo» | Sí, mientras el vencimiento sea futuro |
| `Activo` | 1 | «Activo» | Sí |
| `Suspendido` | 2 | «Suspendido» | No |
| `Cancelado` | 3 | «Cancelado» | No |

Una empresa está **vigente** si no está eliminada y está `Activo`, o `TrialActivo` con vencimiento futuro (`DOM/Empresa.cs:30-33`). Se comprueba al iniciar sesión, al renovar y en **cada petición**: si no está vigente, nadie de la empresa puede usar el sistema. **Pendiente:** las operaciones de activar, suspender, cancelar y extender la prueba existen en la entidad pero ningún endpoint ni pantalla las invoca; hoy el estado solo se cambia en la base de datos ([RN-EMP-015](01-reglas-de-negocio/01a-administracion-y-seguridad.md)).

**Dónde aparece.** Empresa: filtro y columna «Estado» y «Vigente» (`I18N/empresa/es.json:42-50`, `estado.*`). Mensaje de login: «La suscripción de tu empresa no está activa o ha vencido…» (`I18N/auth/es.json:15`). Reglas: [RN-EMP-012](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-EMP-015](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-AUT-009](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Exportación
*También aparece como:* «Exportar», `exportar`, Excel.

**Definición.** Descarga de los resultados de una búsqueda como archivo Excel. Término propio de SOFIA (no normativo).

**En SOFIA.** El botón «Exportar» de cada pantalla de búsqueda genera un `.xlsx` con **los mismos filtros** de la consulta. Los títulos de columna y las etiquetas «Sí»/«No» los manda el SPA ya traducidos y las fechas salen como `dd/MM/yyyy` (español) u otro formato permitido. Un archivo no puede superar **10 000 filas**; si la consulta filtrada tiene más, no se genera.

**Dónde aparece.** Botón «Exportar» (`I18N/es.json:77`, `actions.export`); nombres de archivo como `ventas.xlsx` (`I18N/pos/es.json:65`). Reglas: [RN-IMP-008](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-IMP-011](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-IMP-017](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Licencia profesional
*También aparece como:* «Licencia Profesional», `Licencia_Prof`, `LicenciaProf`.

**Definición.** Número de licencia profesional del empleado. Para el rol de químico farmacéutico, la normativa habla del número de colegiatura; ver «Colegiatura». Definición general, no verificada en fuente oficial.

**En SOFIA.** Texto opcional del **empleado** (hasta 50 caracteres), único por empresa entre empleados no eliminados (`DOM/Empleado.cs:21`). SOFIA no valida su formato ni lo relaciona con ninguna función clínica.

**Dónde aparece.** Empleados: «Licencia Profesional» (`I18N/empleados/es.json:25`); error «Ya existe un empleado con esa licencia profesional.» (`:12`). Reglas: [RN-EPL-007](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-EPL-008](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Nivel de jerarquía
*También aparece como:* «Nivel de Jerarquía», «Nivel Jerárquico», `NivelJerarquia`.

**Definición.** Número que indica la autoridad relativa de un rol. Término propio de SOFIA (no normativo).

**En SOFIA.** Entero ≥ 0 por rol (0 si no se indica); un número mayor indica más autoridad (el rol `Admin` base tiene 100). Es una etiqueta de orden: el código **no lo usa** para decidir permisos; los permisos salen solo de la tabla de permisos del rol (la búsqueda de `NivelJerarquia` en `sofia-apis/src` solo encuentra el alta, la edición, la carga masiva y la exportación de roles; ningún uso en la autorización).

**Dónde aparece.** Roles: «Nivel de Jerarquía», «Nivel desde», «Nivel hasta» (`I18N/roles/es.json:24-26`). Reglas: [RN-ROL-004](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-ROL-019](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Número de licencia (de la sucursal)
*También aparece como:* «Nº de Licencia», `Numero_Licencia`.

**Definición.** Identificador de la autorización de funcionamiento del establecimiento. Para obtener la autorización sanitaria de una farmacia o botica se exige un trámite ante la autoridad de salud. Fuente oficial: https://www.gob.pe/38160-obtener-autorizacion-sanitaria-para-farmacias?child=57900 (trámite confirmado por el resumen de la búsqueda; SOFIA no dice si el número que se guarda corresponde a esa autorización).

**En SOFIA.** Texto obligatorio de la **sucursal** (hasta 50 caracteres), único por empresa. La **sede principal** creada al registrar la empresa puede nacer sin licencia; pasa a ser obligatoria la primera vez que se edita la sucursal (`DOM/Sucursal.cs:56-57`).

**Dónde aparece.** Sucursales: «Nº de Licencia» (`I18N/sucursales/es.json:22`); error «Ya existe una sucursal con ese número de licencia.» (`:10`). Reglas: [RN-SUC-004](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-SUC-005](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Periodo de prueba
*También aparece como:* «1 mes de prueba gratis», «Inicio Trial», `FechaInicioTrial`, `DiasTrial`, trial.

**Definición.** Tiempo inicial gratuito para probar SOFIA. Término propio de SOFIA (no normativo).

**En SOFIA.** Una empresa nueva nace en `TrialActivo` con vencimiento **30 días** después del registro (`DOM/Empresa.cs:37`, `:64`). Al vencer, la empresa deja de estar vigente. No hay pantalla para activar la suscripción.

**Dónde aparece.** Registro: «Crea tu empresa y obtén 1 mes de prueba gratis» (`I18N/empresa/es.json:7`). Empresa: «Inicio Trial» (`:116`). Reglas: [RN-EMP-012](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Permiso (módulo y acción)
*También aparece como:* «Permisos», «Módulo del Sistema», «Acción», `HasPermission`, `PermisoRol`, `Seguridad_Permisos_Rol`.

**Definición.** Autorización para hacer una acción en un módulo. Término propio de SOFIA (no normativo).

**En SOFIA.** Un permiso es un par **módulo + acción** (por ejemplo `POS:CerrarCaja`). El catálogo son exactamente los pares declarados en los endpoints: **105 pares en 27 módulos** (comprobado en `docs/_inventario/permisos.csv`). Acciones genéricas: Leer, Crear, Actualizar, Eliminar; hay acciones propias como «Anular», «Aperturar caja», «Cerrar caja», «Procesar devolución», «Analizar con IA», «Asignar roles» y «Gestionar permisos». Un permiso se asigna a un rol; revocarlo es borrado lógico. Los permisos de un rol se guardan en memoria 5 minutos, pero asignar o revocar un permiso invalida esa memoria de inmediato; los usuarios conectados ven su menú actualizado al recargar la página. El SPA oculta los botones sin permiso, pero la API es la que lo hace cumplir.

**Dónde aparece.** Roles, pestaña «Permisos» (`I18N/roles/es.json:38`) con los nombres de módulo (`:57-85`) y de acción (`:86-107`). Matriz completa: [05b](05-seguridad/05b-matriz-de-permisos.md). Reglas: [RN-ROL-010](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-ROL-016](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-020](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-022](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Plantilla
*También aparece como:* «Descargar plantilla», `plantilla-{{entity}}.xlsx`.

**Definición.** Archivo Excel vacío con las columnas esperadas para una carga masiva. Término propio de SOFIA (no normativo).

**En SOFIA.** Libro con una hoja «Plantilla» que solo contiene la cabecera de columnas del maestro. Se descarga desde la pantalla de búsqueda del maestro.

**Dónde aparece.** «Descargar plantilla» (`I18N/es.json:218`). Reglas: [RN-IMP-016](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Rol
*También aparece como:* «Roles & Permisos», «Roles», `Rol`, `Seguridad_Roles`.

**Definición.** Conjunto de permisos que se asigna a las cuentas. Término propio de SOFIA (no normativo).

**En SOFIA.** Tiene **nombre** (único por empresa, hasta 50 caracteres, **no se puede cambiar** una vez creado), descripción (opcional), **nivel de jerarquía** y permisos. Un rol puede además limitar **sucursales**. No se puede eliminar un rol con cuentas vinculadas. Quien no es Admin no puede ampliar los permisos de un rol que él mismo tiene asignado (no autopromoción).

**Dónde aparece.** Menú «Administración» > «Roles & Permisos» (`I18N/es.json:7`). Pestañas «Menú», «Permisos» y «Sucursales» (`I18N/roles/es.json:37-39`). Reglas: [RN-ROL-001](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-ROL-023](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Rol Admin
*También aparece como:* `Admin`, `Rol.AdminRoleName`, «Administrador de la empresa», «Usuario Administrador».

**Definición.** Rol con acceso total dentro de su empresa. Término propio de SOFIA (no normativo).

**En SOFIA.** Se crea **por empresa** al registrarla (descripción «Administrador de la empresa», `APP/Empresas/Commands/RegistrarEmpresa/RegistrarEmpresaCommandHandler.cs:116`) y se asigna a la cuenta del primer usuario. Un token con el rol llamado `Admin` (sin distinguir mayúsculas) **pasa cualquier comprobación de permiso** sin consultar la tabla ([RN-GEN-021](01-reglas-de-negocio/01a-administracion-y-seguridad.md)) y accede a **todas las sucursales** de su empresa. Solo un Admin puede asignar o quitar el rol Admin, y nadie puede cambiar sus propios roles.

**Dónde aparece.** Cuentas de Usuario: «Solo un administrador puede asignar o quitar el rol Admin» (`I18N/cuentas/es.json:59`). Registro: «Usuario Administrador» (`I18N/empresa/es.json:9`). Reglas: [RN-GEN-021](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-023](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-CTA-010](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-CTA-015](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-CTA-016](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Sede principal
*También aparece como:* «Dirección de la sede principal», «Sede Principal», `CreateSedePrincipal`.

**Definición.** Primera sucursal de una empresa nueva. Término propio de SOFIA (no normativo).

**En SOFIA.** Se crea al registrar la empresa. El **nombre** es opcional («Sede Principal» por defecto); la **dirección es obligatoria** (hasta 255 caracteres) y aparece en el encabezado de las boletas; el **número de licencia** es opcional al registrar y se vuelve obligatorio la primera vez que se edita (`APP/Empresas/Commands/RegistrarEmpresa/RegistrarEmpresaCommandValidator.cs:49-51`; `RegistrarEmpresaCommandHandler.cs:85-91`). Los documentos de reglas más antiguos ([RN-EMP-010](01-reglas-de-negocio/01a-administracion-y-seguridad.md)) la daban por opcional; el código actual manda.

**Dónde aparece.** Registro de empresa: «Dirección de la sede principal» y «Aparecerá en el encabezado de tus boletas» (`I18N/empresa/es.json:12-14`); error «Ingresa la dirección de la sede principal (máximo 255 caracteres)» (`:90`). Reglas: [RN-EMP-010](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-EMP-011](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Sucursal
*También aparece como:* «Sucursales», sede, local, `Sucursal`, `Sucursales`.

**Definición.** Local de la empresa donde se vende y se guarda stock. Término propio de SOFIA (no normativo).

**En SOFIA.** Maestro con **nombre** (hasta 100 caracteres), **dirección física** (hasta 255), **número de licencia** (único por empresa) y **gerente** (un empleado, opcional). Casi todo se separa por sucursal: caja, stock, series fiscales, ventas. No se puede eliminar una sucursal con empleados, usuarios, roles, stock, ventas u otros registros asociados.

**Dónde aparece.** Menú «Administración» > «Sucursales» (`I18N/es.json:5`). Campos: «Nombre», «Dirección Física», «Nº de Licencia», «Gerente» (`I18N/sucursales/es.json:20-23`). Reglas: [RN-SUC-001](01-reglas-de-negocio/01a-administracion-y-seguridad.md) a [RN-SUC-013](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Sucursal base
*También aparece como:* «Sucursal Base», `Sucursal_Base_ID`, `Sucursal_Base`.

**Definición.** Sucursal a la que pertenece un empleado. Término propio de SOFIA (no normativo).

**En SOFIA.** Obligatoria en cada empleado. **Define la sucursal de la sesión**: el token de un usuario lleva como `sucursalId` la sucursal base de su empleado, y con ella se abre una caja, se registra una venta o se procesa una devolución. Nadie puede cambiar su propia sucursal base; al cambiar la de otro empleado se invalidan sus sesiones.

**Dónde aparece.** Empleados: «Sucursal Base» (`I18N/empleados/es.json:28`, `fields.sucursal`). Reglas: [RN-EPL-002](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-EPL-011](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-EPL-012](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-GEN-023](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Sucursales permitidas
*También aparece como:* «Sucursales Asignadas», pestaña «Sucursales» de un rol, `CuentaSucursal`, `RolSucursal`, sucursales accesibles.

**Definición.** Sucursales en las que un usuario puede ver y operar. Término propio de SOFIA (no normativo).

**En SOFIA.** Un Admin accede a todas las sucursales de su empresa. Cualquier otro usuario accede a: su **sucursal base**, las **asignadas a su cuenta** y las **asignadas a sus roles**, cruzadas con las sucursales no eliminadas de la empresa. Los selectores de sucursal solo muestran estas.

**Dónde aparece.** Cuentas de Usuario: «Sucursales Asignadas» («Sucursales adicionales donde esta cuenta puede operar, además de la sucursal base de su empleado») (`I18N/cuentas/es.json:47-48`); Roles, pestaña «Sucursales» (`I18N/roles/es.json:39`). Reglas: [RN-GEN-023](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-SUC-010](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-CTA-019](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-ROL-017](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [RN-ROL-018](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

## 8. Inteligencia artificial

Mapa de la familia: el único uso de IA en SOFIA es **leer una receta desde una foto**. El orden es: la imagen va a **Gemini** (que hace el **OCR**), el texto sale a **Presidio** (que lo **anonimiza**), el texto anonimizado vuelve a Gemini (que interpreta los medicamentos) y SOFIA los cruza con el **catálogo DIGEMID**, el catálogo de medicamentos y el stock de la sucursal. **No se guarda nada**: ni la imagen, ni el texto, ni el resultado. Detalle técnico: [07-integraciones §2 y §3](07-integraciones.md).

### Análisis de receta (escaneo)
*También aparece como:* «Lector de Recetas IA», «Escanear receta», «Escanear receta médica», «Interpretar receta», «Análisis con IA», `AnalizarReceta`, `Recetas:Analizar`.

**Definición.** Lectura automática de una receta fotografiada para sugerir los medicamentos que contiene. Término propio de SOFIA (no normativo).

**En SOFIA.** Se sube o fotografía una imagen y se pulsa «Interpretar receta»; el sistema devuelve hasta **20 medicamentos** y, por cada uno, hasta **3 sugerencias**; el resto se descarta sin aviso. Cada medicamento muestra si fue «Detectado como» otro nombre, su confianza, la cantidad sugerida, el stock en la sucursal y los equivalentes terapéuticos. Exige el permiso `Recetas:Analizar` y está limitado a **10 peticiones por minuto por empresa**. En el POS aparece como una pestaña «Receta» (visible solo con ese permiso) que agrega los medicamentos al carrito. El resultado **no reemplaza** a la receta registrada: para vender receta retenida o estupefaciente sigue haciendo falta una receta del maestro con número.

**Dónde aparece.** Recetas Médicas > «Escanear receta» (`I18N/recetas-medicas/es.json:7`), pantalla «Lector de Recetas IA» (`:56`) con «Tomar foto», «Galería», «Interpretar receta» (`:58-62`); POS: pestaña «Receta» y «{{count}} medicamento(s) detectado(s) en la receta» (`I18N/pos/es.json:214-215`). Reglas: [RN-REC-018](01-reglas-de-negocio/01c1-inventario-y-recetas.md) a [RN-REC-029](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-GEN-028](01-reglas-de-negocio/01a-administracion-y-seguridad.md).

### Anonimización
*También aparece como:* `AnonimizarDatos`, «Anonimizar datos» (permiso), `IPrivacyService`, «PII».

**Definición.** Reemplazar los datos personales de un texto por etiquetas genéricas. En el Perú los datos de salud y de identidad son datos personales; la norma de protección aplicable no se consultó. Definición general, no verificada en fuente oficial.

**En SOFIA.** Dos cosas distintas: (1) el **filtro de Presidio** que se aplica al texto de la receta antes de enviarlo a la segunda llamada de Gemini; es **cerrado ante fallos**: si Presidio falla o devuelve vacío, el análisis se aborta (`SOFIA.Infrastructure/Services/PresidioPrivacyService.cs:20-41`); (2) la operación «Anonimizar datos» de Auditoría, que **no está implementada** y responde 501 ([RN-AUD-003](01-reglas-de-negocio/01d-modulos-solo-api.md)). **Advertencia:** la imagen original se envía a Gemini para el OCR **antes** de anonimizar; la anonimización protege solo el texto de la segunda llamada ([07-integraciones §2.3](07-integraciones.md)).

**Dónde aparece.** Sin texto propio en la interfaz. Permiso «Anonimizar datos» (`I18N/roles/es.json:94`, `permisos.accion.AnonimizarDatos`). Reglas: [RN-REC-021](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-AUD-001](01-reglas-de-negocio/01d-modulos-solo-api.md) a [RN-AUD-005](01-reglas-de-negocio/01d-modulos-solo-api.md).

### Confianza (puntaje)
*También aparece como:* «Confianza», «Alta», «Media», «Baja», `NivelConfianza`, `ConfianzaFinal`, puntaje.

**Definición.** Medida de qué tan seguro está el sistema de que un medicamento detectado es correcto. Término propio de SOFIA (no normativo).

**En SOFIA.** Número entre 0 y 1 calculado así: `confianza = NivelConfianza de la IA × 0,6 + parecido × 0,4`, redondeado a 2 decimales, donde `parecido` vale **0,9** si el nombre se encontró en el catálogo DIGEMID y **1,0** si no (`APP/Recetas/Queries/AnalizarReceta/AnalizarRecetaQueryHandler.cs:55`, `:63`, `:66`). Las sugerencias llevan confianza fija **0,70** (`:121`). La pantalla la rotula «Alta» desde 0,85, «Media» desde 0,65 y «Baja» por debajo (`SPA/pages/recetas-medicas/recetas-scanner/recetas-scanner.component.ts:125-130`). Un medicamento sin coincidencia DIGEMID puntúa **más alto** en el factor de parecido que uno con coincidencia: es una característica del cálculo vigente, no una medida de calidad de la coincidencia.

**Dónde aparece.** Lector de Recetas IA: «Confianza», «Alta», «Media», «Baja» (`I18N/recetas-medicas/es.json:66`, `:77-79`). Reglas: [RN-REC-025](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Gemini
*También aparece como:* Google Gemini, `GeminiRecetaAnalyzer`, `GeminiApi`.

**Definición.** Modelo de IA de Google que SOFIA usa por HTTP. Definición general, no verificada en fuente oficial.

**En SOFIA.** Se llama **dos veces** por análisis: la primera con la **imagen completa** para extraer el texto (OCR) y la segunda con el texto ya anonimizado para identificar los medicamentos (temperatura 0,2, respuesta en JSON). Los modelos, la URL base y la clave salen de la configuración (`GeminiApi:BaseUrl`, `OcrModel`, `ReasoningModel`, `ApiKey`; solo se documentan los nombres). Si Gemini devuelve algo que no es el JSON esperado, el resultado es una lista vacía sin error; cualquier otro fallo sale como error 500 con referencia de soporte.

**Dónde aparece.** Sin texto de interfaz; solo el error genérico «Error al interpretar la receta. Intenta nuevamente.» (`I18N/recetas-medicas/es.json:76`, `scanner.error`). Detalle: [07-integraciones §2](07-integraciones.md). Reglas: [RN-REC-021](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### OCR
*También aparece como:* reconocimiento óptico de caracteres, `ExtractTextFromImageAsync`, «primera llamada».

**Definición.** Tecnología que convierte el texto de una imagen en texto digital. Definición general, no verificada en fuente oficial.

**En SOFIA.** No hay un motor de OCR propio: la lectura de la imagen la hace Gemini con un prompt que le pide transcribir sin interpretar. Si no extrae texto, el análisis termina con una lista vacía y no se llama a Presidio. El tipo de archivo no se valida en el servidor (solo el navegador filtra imágenes).

**Dónde aparece.** Sin texto de interfaz. Mensaje de lista vacía: «No se detectaron medicamentos en la imagen. Intenta con una foto más nítida.» (`I18N/recetas-medicas/es.json:75`). Reglas: [RN-REC-019](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-REC-021](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Presidio
*También aparece como:* Microsoft Presidio, `PresidioPrivacyService`, `Registro_Privacidad_Presidio`, microservicio de anonimización.

**Definición.** Herramienta de código abierto de Microsoft para detectar y anonimizar datos personales; SOFIA la usa como un microservicio interno. Definición general, no verificada en fuente oficial.

**En SOFIA.** Servicio aparte (Python) llamado en `POST /api/anonimizar`. Detecta DNI de 8 dígitos, RUC de 11 dígitos que empiezan por 10, 15, 17 o 20, celulares peruanos de 9 dígitos y las entidades predefinidas de Presidio para español, con una lista de términos permitidos (nombres de medicamentos y vocabulario farmacéutico) que no se marcan. Devuelve solo el texto anonimizado, no las entidades detectadas. La tabla `Registro_Privacidad_Presidio` existe pero **nunca se escribe** (Pendiente). El detalle exacto de los reemplazos no se verificó por ejecución.

**Dónde aparece.** Sin texto de interfaz. Detalle: [07-integraciones §3](07-integraciones.md); tabla en [03a §4.9](03-modelo-de-datos/03a-maestros-y-seguridad.md). Reglas: [RN-REC-021](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [RN-REC-030](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

### Sugerencia de la IA
*También aparece como:* «Sugerencias IA», `Sugerencias`, tipo «Recomendacion».

**Definición.** Medicamento alternativo propuesto por la IA para un medicamento de la receta. Término propio de SOFIA (no normativo).

**En SOFIA.** Por cada medicamento, la IA puede proponer hasta 3 alternativas; solo se muestran las que se encuentran en el catálogo de la empresa, con la razón «Sugerido por {medicamento}», sin validación DIGEMID, sin equivalentes y con confianza fija 0,70.

**Dónde aparece.** Lector de Recetas IA: sección «Sugerencias IA» (`I18N/recetas-medicas/es.json:69`, `scanner.section-suggestions`). Reglas: [RN-REC-028](01-reglas-de-negocio/01c1-inventario-y-recetas.md).

## 9. Índice alfabético

Cada línea enlaza a la entrada del término. Las líneas con «ver» son variantes o sinónimos que no tienen entrada propia. Hay 133 entradas.

- Abierta (sesión): ver [Estado de la sesión de caja](#estado-de-la-sesión-de-caja)
- [Acta de destrucción](#acta-de-destrucción)
- [Ajuste de stock](#ajuste-de-stock)
- Análisis de receta: ver [Análisis de receta (escaneo)](#análisis-de-receta-escaneo)
- [Análisis de receta (escaneo)](#análisis-de-receta-escaneo)
- [Anonimización](#anonimización)
- [Anulación](#anulación)
- Anular venta: ver [Anulación](#anulación)
- [Apertura de caja](#apertura-de-caja)
- [Arqueo](#arqueo)
- [Aseguradora](#aseguradora)
- [Auditoría](#auditoría)
- Base imponible: ver [Base imponible (operación gravada)](#base-imponible-operación-gravada)
- [Base imponible (operación gravada)](#base-imponible-operación-gravada)
- [Bloqueo de cuenta](#bloqueo-de-cuenta)
- [Boleta](#boleta)
- [Borrado lógico](#borrado-lógico)
- Caja: ver [Sesión de caja](#sesión-de-caja)
- [Cajero](#cajero)
- Cajero (empleado): ver [Cajero](#cajero)
- [Cambio de contraseña obligatorio](#cambio-de-contraseña-obligatorio)
- [Carga masiva](#carga-masiva)
- [Carné de extranjería](#carné-de-extranjería)
- Carné de extranjería (documento): ver [Carné de extranjería](#carné-de-extranjería)
- [Catálogo DIGEMID](#catálogo-digemid)
- Cerrada (sesión): ver [Estado de la sesión de caja](#estado-de-la-sesión-de-caja)
- Cierre: ver [Cierre de caja](#cierre-de-caja)
- [Cierre de caja](#cierre-de-caja)
- Cliente: ver [Paciente](#paciente)
- [Cliente eventual](#cliente-eventual)
- [Cobertura de aseguradora](#cobertura-de-aseguradora)
- [Código ATC](#código-atc)
- [Código de venta](#código-de-venta)
- [Código nacional](#código-nacional)
- [Colegiatura](#colegiatura)
- Colegiatura (número de colegiatura): ver [Colegiatura](#colegiatura)
- Comprobante: ver [Comprobante de pago](#comprobante-de-pago)
- [Comprobante de pago](#comprobante-de-pago)
- [Comprobante impreso](#comprobante-impreso)
- [Concentración](#concentración)
- [Condición de venta](#condición-de-venta)
- Confianza: ver [Confianza (puntaje)](#confianza-puntaje)
- [Confianza (puntaje)](#confianza-puntaje)
- [Correlativo](#correlativo)
- Cuadrada (sesión): ver [Estado de la sesión de caja](#estado-de-la-sesión-de-caja)
- [Cuarentena](#cuarentena)
- [Cuenta de usuario](#cuenta-de-usuario)
- [Dashboard](#dashboard)
- [DCI (denominación común internacional)](#dci-denominación-común-internacional)
- Denominación común internacional: ver [DCI (denominación común internacional)](#dci-denominación-común-internacional)
- Denominación DCI: ver [DCI (denominación común internacional)](#dci-denominación-común-internacional)
- [Despacho (delivery)](#despacho-delivery)
- [Destino de la devolución](#destino-de-la-devolución)
- [Devolución](#devolución)
- [Diferencia de arqueo (faltante y sobrante)](#diferencia-de-arqueo-faltante-y-sobrante)
- [DIGEMID](#digemid)
- Director técnico: ver [Regente](#regente)
- [Dispensación](#dispensación)
- [DNI](#dni)
- [Efectivo esperado](#efectivo-esperado)
- [Emisor](#emisor)
- [Empleado](#empleado)
- Empresa: ver [Empresa (tenant)](#empresa-tenant)
- [Empresa (tenant)](#empresa-tenant)
- Envío electrónico a SUNAT: ver [OSE y PSE (envío electrónico a SUNAT)](#ose-y-pse-envío-electrónico-a-sunat)
- [Equivalente terapéutico](#equivalente-terapéutico)
- Escaneo de receta: ver [Análisis de receta (escaneo)](#análisis-de-receta-escaneo)
- Estado de la sesión: ver [Estado de la sesión de caja](#estado-de-la-sesión-de-caja)
- [Estado de la sesión de caja](#estado-de-la-sesión-de-caja)
- [Estado de la transferencia (estado logístico)](#estado-de-la-transferencia-estado-logístico)
- [Estado de la venta](#estado-de-la-venta)
- [Estado de resolución de cuarentena](#estado-de-resolución-de-cuarentena)
- Estado de suscripción: ver [Estados de suscripción](#estados-de-suscripción)
- Estado logístico: ver [Estado de la transferencia (estado logístico)](#estado-de-la-transferencia-estado-logístico)
- [Estado SUNAT del comprobante](#estado-sunat-del-comprobante)
- [Estados de suscripción](#estados-de-suscripción)
- [Estupefaciente](#estupefaciente)
- Estupefacientes: ver [Estupefaciente](#estupefaciente)
- Existencias: ver [Stock por sucursal](#stock-por-sucursal)
- [Exportación](#exportación)
- [Factura](#factura)
- Faltante: ver [Diferencia de arqueo (faltante y sobrante)](#diferencia-de-arqueo-faltante-y-sobrante)
- [Fecha de caducidad](#fecha-de-caducidad)
- [Forma farmacéutica](#forma-farmacéutica)
- Fórmula magistral: ver [Fórmula magistral (orden magistral)](#fórmula-magistral-orden-magistral)
- [Fórmula magistral (orden magistral)](#fórmula-magistral-orden-magistral)
- [Formulación clínica](#formulación-clínica)
- [Fracción](#fracción)
- [Gemini](#gemini)
- Hash, XML y CDR (envío a SUNAT): ver [OSE y PSE (envío electrónico a SUNAT)](#ose-y-pse-envío-electrónico-a-sunat)
- Historial de precios: ver [Historial de precios (de proveedor)](#historial-de-precios-de-proveedor)
- [Historial de precios (de proveedor)](#historial-de-precios-de-proveedor)
- [IGV](#igv)
- [Ingrediente activo](#ingrediente-activo)
- [Ingreso de stock](#ingreso-de-stock)
- [Inmunización](#inmunización)
- [Jerarquía de unidades](#jerarquía-de-unidades)
- Jerarquía UoM: ver [Jerarquía de unidades](#jerarquía-de-unidades)
- [Laboratorio](#laboratorio)
- Liberado, Destruido, Devuelto (cuarentena): ver [Estado de resolución de cuarentena](#estado-de-resolución-de-cuarentena)
- [Licencia profesional](#licencia-profesional)
- [Lote](#lote)
- Lote vencido: ver [Fecha de caducidad](#fecha-de-caducidad)
- Medicamento: ver [Medicamento (producto)](#medicamento-producto)
- [Medicamento (producto)](#medicamento-producto)
- [Método de pago](#método-de-pago)
- Métodos de pago: ver [Método de pago](#método-de-pago)
- [Modalidad de registro](#modalidad-de-registro)
- [Monto de apertura](#monto-de-apertura)
- [Monto declarado](#monto-declarado)
- [Motivo de la nota de crédito](#motivo-de-la-nota-de-crédito)
- [Multiplicador](#multiplicador)
- [Nivel de jerarquía](#nivel-de-jerarquía)
- [Nota de crédito](#nota-de-crédito)
- [Nota de débito](#nota-de-débito)
- Número de licencia: ver [Número de licencia (de la sucursal)](#número-de-licencia-de-la-sucursal)
- [Número de licencia (de la sucursal)](#número-de-licencia-de-la-sucursal)
- [Número de operación](#número-de-operación)
- [Número de receta](#número-de-receta)
- [OCR](#ocr)
- Operación gravada: ver [Base imponible (operación gravada)](#base-imponible-operación-gravada)
- Orden magistral: ver [Fórmula magistral (orden magistral)](#fórmula-magistral-orden-magistral)
- OSE: ver [OSE y PSE (envío electrónico a SUNAT)](#ose-y-pse-envío-electrónico-a-sunat)
- [OSE y PSE (envío electrónico a SUNAT)](#ose-y-pse-envío-electrónico-a-sunat)
- OTC: ver [Venta libre (OTC)](#venta-libre-otc)
- [Paciente](#paciente)
- [Pago dividido](#pago-dividido)
- Pasaporte: ver [Tipo de documento de identidad](#tipo-de-documento-de-identidad)
- [Periodo de prueba](#periodo-de-prueba)
- Permiso: ver [Permiso (módulo y acción)](#permiso-módulo-y-acción)
- [Permiso (módulo y acción)](#permiso-módulo-y-acción)
- [Plantilla](#plantilla)
- [Precio de compra](#precio-de-compra)
- [Precio de venta base](#precio-de-venta-base)
- Prefijo de serie: ver [Serie (prefijo)](#serie-prefijo)
- [Presentación de venta](#presentación-de-venta)
- [Presidio](#presidio)
- Producto: ver [Medicamento (producto)](#medicamento-producto)
- [Profesional de salud](#profesional-de-salud)
- [Proforma](#proforma)
- [Proveedor](#proveedor)
- [Próximo a vencer](#próximo-a-vencer)
- PSE: ver [OSE y PSE (envío electrónico a SUNAT)](#ose-y-pse-envío-electrónico-a-sunat)
- Puntaje: ver [Confianza (puntaje)](#confianza-puntaje)
- [Punto de venta (POS)](#punto-de-venta-pos)
- Químico farmacéutico: ver [Regente](#regente)
- [Receta médica](#receta-médica)
- [Receta retenida](#receta-retenida)
- [Receta simple](#receta-simple)
- [Receta verificada](#receta-verificada)
- [Reclamo de seguro](#reclamo-de-seguro)
- [Reembolso](#reembolso)
- [Regente](#regente)
- [Registro sanitario](#registro-sanitario)
- Repeticiones: ver [Repeticiones (máximas)](#repeticiones-máximas)
- [Repeticiones (máximas)](#repeticiones-máximas)
- Reporte de cierre de caja: ver [Arqueo](#arqueo)
- Retenido (cuarentena): ver [Estado de resolución de cuarentena](#estado-de-resolución-de-cuarentena)
- [Rol](#rol)
- [Rol Admin](#rol-admin)
- [RUC](#ruc)
- Sede: ver [Sucursal](#sucursal)
- [Sede principal](#sede-principal)
- Seguro: ver [Aseguradora](#aseguradora)
- Serie: ver [Serie (prefijo)](#serie-prefijo)
- [Serie (prefijo)](#serie-prefijo)
- Sesión cuadrada frente a sesión cerrada: ver [Estado de la sesión de caja](#estado-de-la-sesión-de-caja)
- [Sesión de caja](#sesión-de-caja)
- Sobrante: ver [Diferencia de arqueo (faltante y sobrante)](#diferencia-de-arqueo-faltante-y-sobrante)
- Stock: ver [Stock por sucursal](#stock-por-sucursal)
- [Stock bajo](#stock-bajo)
- [Stock por sucursal](#stock-por-sucursal)
- [Stock vendible](#stock-vendible)
- [Sucursal](#sucursal)
- [Sucursal base](#sucursal-base)
- [Sucursales permitidas](#sucursales-permitidas)
- [Sugerencia de la IA](#sugerencia-de-la-ia)
- Tenant: ver [Empresa (tenant)](#empresa-tenant)
- [Ticket](#ticket)
- [Tipo de comprobante](#tipo-de-comprobante)
- [Tipo de documento de identidad](#tipo-de-documento-de-identidad)
- Transferencia (estados): ver [Estado de la transferencia (estado logístico)](#estado-de-la-transferencia-estado-logístico)
- [Transferencia entre sucursales](#transferencia-entre-sucursales)
- Trial: ver [Periodo de prueba](#periodo-de-prueba)
- [Unidad base](#unidad-base)
- [Unidad de medida](#unidad-de-medida)
- Unidad mayor: ver [Jerarquía de unidades](#jerarquía-de-unidades)
- Unidad menor: ver [Jerarquía de unidades](#jerarquía-de-unidades)
- UoM: ver [Unidad de medida](#unidad-de-medida)
- Vencimiento (fecha): ver [Fecha de caducidad](#fecha-de-caducidad)
- [Venta](#venta)
- [Venta completada](#venta-completada)
- Venta en espera: ver [Venta en espera (pendiente)](#venta-en-espera-pendiente)
- [Venta en espera (pendiente)](#venta-en-espera-pendiente)
- Venta libre: ver [Venta libre (OTC)](#venta-libre-otc)
- [Venta libre (OTC)](#venta-libre-otc)
- Venta pendiente: ver [Venta en espera (pendiente)](#venta-en-espera-pendiente)
- [Vuelto](#vuelto)
- Yape / Plin: ver [Método de pago](#método-de-pago)

## 10. Pendientes y «No verificado»

### 10.1 Cifras de este glosario

| Marca de la definición | Entradas |
|---|---|
| `Fuente oficial: <URL>` (confirmada por búsqueda web; varias solo en parte, y así se dice en la entrada) | 36 |
| `Definición general, no verificada en fuente oficial.` | 45 |
| `Término propio de SOFIA (no normativo).` | 52 |
| Total de entradas | 133 |

Se hicieron 19 búsquedas web (el máximo permitido era 25) y un intento de abrir una página de SUNAT que respondió 403.

### 10.2 «No verificado»

1. **Las fuentes oficiales se confirmaron con el resumen de cada búsqueda, no abriendo cada documento.** El único intento de abrir una página (`cpe.sunat.gob.pe/informacion_general/tipos_comprobantes_pago`) devolvió HTTP 403. Los datos que salen de un resumen llevan la nota «confirmado por el resumen de la búsqueda».
2. **IGV.** El desglose «IGV más Impuesto de Promoción Municipal, total 18%» sale del resumen de la búsqueda sobre la orientación de SUNAT. El resumen advertía que algunos textos legales aún muestran otras tasas. El código usa 1,18 y la interfaz «IGV (18%)», fijos.
3. **Motivo de la nota de crédito (06 y 07).** La búsqueda confirmó que 06 es devolución total y 07 devolución por ítem en el catálogo 09, pero no devolvió la redacción exacta.
4. **Catálogo 06 de SUNAT.** Se citó el anexo 112-2021; el resumen advirtió que no se pudo confirmar la versión vigente más reciente.
5. **Receta retenida y estupefaciente.** Se cita el D.S. 023-2001-SA tal como lo devolvió la búsqueda; no se verificó su vigencia actual. El resultado también mencionó el D.S. 020-2024-SA, que adecúa los reglamentos de la Ley 29459; no se leyó.
6. **Regente y director técnico.** Se afirma, según los resultados de búsqueda, que la normativa vigente habla de «Director Técnico» y que «regente» era del reglamento anterior (D.S. 021-2001-SA, derogado); no se leyó el texto consolidado del D.S. 014-2011-SA.
7. **Código ATC.** La fuente oficial hallada es un estándar de dato en salud del Minsa de 2011 alojado en el sitio de un hospital.
8. **Sin fuente oficial hallada:** cuarentena (solo se identificó el manual de buenas prácticas de almacenamiento, R.M. 132-2015/MINSA, sin su texto), acta de destrucción, definición textual de receta médica y de dispensación, inmunización, Yape y Plin, Gemini, OCR, Presidio y anonimización (no se consultó la norma peruana de datos personales).
9. **Presidio.** La lista exacta de entidades predefinidas para español y el formato exacto de los reemplazos no se verificaron por ejecución ([07-integraciones §3.4](07-integraciones.md)).
10. **Moneda de la venta.** El glosario dice que se vende en soles porque todos los montos de la interfaz son «S/»; no se buscó una regla que lo imponga.

### 10.3 Pendientes (existe el nombre, falta la función)

- Envío de comprobantes a SUNAT (OSE o PSE), factura, ticket, nota de débito y proforma: solo el enum o la serie.
- Pantallas de Transferencias, Delivery, Magistrales, Servicios e inmunizaciones, Cuarentena y actas, listado de Devoluciones y Auditoría: solo API.
- Operaciones de cuarentena para liberar, destruir o devolver un lote (los estados existen, nadie los asigna).
- Cobertura de seguro en el POS: apagada por una constante.
- Activar, suspender, cancelar o extender la prueba de una empresa: solo en la base de datos.
- Estado `Aprobada` de la transferencia: ningún endpoint lo alcanza.
- Tabla `Registro_Privacidad_Presidio` y `Recetas_Digitalizadas_IA`: nunca se escriben.

### 10.4 Hallazgos para el orquestador

1. **Documentos anteriores desactualizados frente al código.** RN-PRV-020 del [01b](01-reglas-de-negocio/01b-catalogos.md) y el modelo [03a](03-modelo-de-datos/03a-maestros-y-seguridad.md) dicen que registrar un precio de proveedor «no guarda nada»; el handler actual guarda el precio y cierra el anterior (`APP/Proveedores/Commands/RegistrarPrecioProveedor/RegistrarPrecioProveedorCommandHandler.cs:26-46`). RN-EMP-010 del [01a](01-reglas-de-negocio/01a-administracion-y-seguridad.md) da la dirección de la sede principal por opcional; el validador actual la exige (`RegistrarEmpresaCommandValidator.cs:49-51`). Varias reglas de borrado de maestros del 01b dicen «no valida dependencias» y el commit `27597cd` las agregó.
2. **`enums.csv` no incluye `EstadoEmpresa`**, que se declara dentro de `DOM/Empresa.cs:6-12` y no en la carpeta `Enums`. Se documentó en «Estados de suscripción». Tampoco incluye enums del SPA (`TipoComprobante` numérico, `CondicionVenta` numérico).
3. **Textos de interfaz sin pantalla:** los archivos `uom`, `servicios`, `transferencias`, `inventarios`, `seguridad`, `auditoria` y `login` (en `public/i18n`) no los usa ningún componente (búsqueda sin resultados en `sofia-web-spa/src`), y dentro de `pos/es.json` no se usan `dialogs.*`, `reportDialog.faltanteText`, `table.faltanteText` y `transferDialog.*`.
4. **Texto de marketing:** «Reportes DIGEMID automatizados» (`I18N/auth/es.json:24`) no corresponde a ninguna función.
5. **Etiquetas distintas para el mismo valor:** `EstadoLogistico.Completada` se rotula «Recibida Completa»; el método `Tarjeta` se rotula «Tarjeta Débito/Crédito» en el POS y «Tarjeta» en el dashboard; el estado `Pendiente` de la venta se llama «venta en espera» en el resto de la interfaz.
6. **Estado de aceptación del comprobante:** el script deja `Pendiente` por defecto (`DBUP/0001_Initial_Schema.sql:573`) y el código siempre escribe `NoEnviado`.
7. **Confianza del análisis de receta:** el factor de parecido da 1,0 al medicamento sin coincidencia DIGEMID y 0,9 al que la tiene (`AnalizarRecetaQueryHandler.cs:54`, `:63`), de modo que lo no validado puntúa algo más alto.
8. **Roles base sembrados (`0002`)** tienen `TenantId` nulo y sin permisos: ningún rol «Cajero», «Farmaceutico», etc. es visible para una empresa.
9. **«En curso (sin commitear)»:** ninguno. `git status --short` estaba vacío en ambos repositorios (`43f5529` y `eeffbb6`).

## 11. Tabla de cobertura

### 11.1 Enums de `docs/_inventario/enums.csv`

| Enum | Valores | Entrada(s) donde quedó |
|---|---|---|
| `CondicionVenta` | `VentaLibreOTC`, `RecetaSimple`, `RecetaRetenida`, `Estupefaciente` (4) | [Condición de venta](#condición-de-venta), [Venta libre (OTC)](#venta-libre-otc), [Receta simple](#receta-simple), [Receta retenida](#receta-retenida), [Estupefaciente](#estupefaciente) |
| `DestinoDevolucion` | `Reingreso_Venta`, `Cuarentena_DIGEMID` (2) | [Destino de la devolución](#destino-de-la-devolución) |
| `EstadoComprobanteSunat` | `NoEnviado`, `Aceptado`, `Rechazado`, `Anulado` (4; más `Pendiente` solo en el script) | [Estado SUNAT del comprobante](#estado-sunat-del-comprobante) |
| `EstadoDespacho` | `Preparando`, `En_Camino`, `Entregado`, `Devuelto` (4) | [Despacho (delivery)](#despacho-delivery) |
| `EstadoLogistico` | `Iniciada`, `Aprobada`, `En_Transito`, `Recibida_Parcial`, `Completada`, `Cancelada` (6) | [Estado de la transferencia (estado logístico)](#estado-de-la-transferencia-estado-logístico), [Transferencia entre sucursales](#transferencia-entre-sucursales) |
| `EstadoResolucionCuarentena` | `Retenido`, `Liberado`, `Destruido`, `Devuelto` (4) | [Estado de resolución de cuarentena](#estado-de-resolución-de-cuarentena), [Cuarentena](#cuarentena) |
| `EstadoSesion` | `Abierta`, `Cerrada`, `Cuadrada` (3) | [Estado de la sesión de caja](#estado-de-la-sesión-de-caja) |
| `EstadoVenta` | `Completada`, `Anulada`, `Devuelta`, `Pendiente` (4) | [Estado de la venta](#estado-de-la-venta), [Venta completada](#venta-completada), [Venta en espera (pendiente)](#venta-en-espera-pendiente), [Anulación](#anulación) |
| `MetodoPago` | `Efectivo`, `YapePlin`, `Tarjeta`, `Transferencia` (4) | [Método de pago](#método-de-pago) |
| `ModalidadRegistro` | `Corriente`, `Historica` (2) | [Modalidad de registro](#modalidad-de-registro), [Inmunización](#inmunización) |
| `TipoComprobante` | `Ticket`, `Factura`, `Boleta`, `NotaCredito`, `NotaDebito`, `Proforma` (6) | [Tipo de comprobante](#tipo-de-comprobante), [Ticket](#ticket), [Factura](#factura), [Boleta](#boleta), [Nota de crédito](#nota-de-crédito), [Nota de débito](#nota-de-débito), [Proforma](#proforma) |
| `TipoDocumentoIdentidad` | `Dni`, `CarneExtranjeria`, `Ruc`, `Pasaporte` y `Varios` (5 códigos) | [Tipo de documento de identidad](#tipo-de-documento-de-identidad), [DNI](#dni), [Carné de extranjería](#carné-de-extranjería), [RUC](#ruc), [Cliente eventual](#cliente-eventual) |
| `EstadoEmpresa` (no figura en `enums.csv`; está en `DOM/Empresa.cs:6-12`) | `TrialActivo`, `Activo`, `Suspendido`, `Cancelado` (4) | [Estados de suscripción](#estados-de-suscripción), [Periodo de prueba](#periodo-de-prueba) |

Doce de doce enums de `enums.csv` cubiertos, más `EstadoEmpresa`.

### 11.2 Términos de la lista del encargo

| Término del encargo | Entrada o resultado |
|---|---|
| RUC | [RUC](#ruc) |
| Comprobante de pago | [Comprobante de pago](#comprobante-de-pago) |
| Boleta | [Boleta](#boleta) |
| Factura | [Factura](#factura) |
| Nota de crédito | [Nota de crédito](#nota-de-crédito) |
| Nota de débito | [Nota de débito](#nota-de-débito) |
| Ticket | [Ticket](#ticket) |
| Proforma | [Proforma](#proforma) |
| Serie (prefijo) | [Serie (prefijo)](#serie-prefijo) |
| Correlativo | [Correlativo](#correlativo) |
| IGV | [IGV](#igv) |
| Base imponible u operación gravada | [Base imponible (operación gravada)](#base-imponible-operación-gravada) |
| Emisor | [Emisor](#emisor) |
| Tipos de documento de identidad (DNI, carné de extranjería, pasaporte, RUC) | [Tipo de documento de identidad](#tipo-de-documento-de-identidad), [DNI](#dni), [Carné de extranjería](#carné-de-extranjería), [RUC](#ruc) |
| Motivo de la nota de crédito | [Motivo de la nota de crédito](#motivo-de-la-nota-de-crédito) |
| Estado SUNAT del comprobante | [Estado SUNAT del comprobante](#estado-sunat-del-comprobante) |
| OSE/PSE y envío electrónico | [OSE y PSE (envío electrónico a SUNAT)](#ose-y-pse-envío-electrónico-a-sunat) (no existe en SOFIA) |
| DIGEMID | [DIGEMID](#digemid) |
| Catálogo DIGEMID | [Catálogo DIGEMID](#catálogo-digemid) |
| Registro sanitario | [Registro sanitario](#registro-sanitario) |
| Condición de venta y sus valores | [Condición de venta](#condición-de-venta), [Venta libre (OTC)](#venta-libre-otc), [Receta simple](#receta-simple), [Receta retenida](#receta-retenida), [Estupefaciente](#estupefaciente) |
| Receta médica | [Receta médica](#receta-médica) |
| Receta simple | [Receta simple](#receta-simple) |
| Receta retenida | [Receta retenida](#receta-retenida) |
| Estupefaciente | [Estupefaciente](#estupefaciente) |
| Dispensación | [Dispensación](#dispensación) |
| Repeticiones | [Repeticiones (máximas)](#repeticiones-máximas) |
| Receta verificada | [Receta verificada](#receta-verificada) |
| DCI | [DCI (denominación común internacional)](#dci-denominación-común-internacional) |
| Código ATC | [Código ATC](#código-atc) |
| Ingrediente activo | [Ingrediente activo](#ingrediente-activo) |
| Concentración | [Concentración](#concentración) |
| Formulación clínica | [Formulación clínica](#formulación-clínica) |
| Forma farmacéutica | [Forma farmacéutica](#forma-farmacéutica) |
| Laboratorio | [Laboratorio](#laboratorio) |
| Lote | [Lote](#lote) |
| Fecha de caducidad o vencimiento | [Fecha de caducidad](#fecha-de-caducidad) |
| Cuarentena y sus estados | [Cuarentena](#cuarentena), [Estado de resolución de cuarentena](#estado-de-resolución-de-cuarentena) |
| Acta de destrucción | [Acta de destrucción](#acta-de-destrucción) |
| Químico farmacéutico o regente | [Regente](#regente) |
| Fórmula u orden magistral | [Fórmula magistral (orden magistral)](#fórmula-magistral-orden-magistral) |
| Inmunización | [Inmunización](#inmunización) |
| Profesional de salud | [Profesional de salud](#profesional-de-salud) |
| Colegiatura | [Colegiatura](#colegiatura) (**No se usa en SOFIA**: la búsqueda de `colegiatura` y `colegiado` no encuentra nada en el SPA, los textos, la API ni los scripts) |
| Sesión de caja | [Sesión de caja](#sesión-de-caja) |
| Apertura y monto de apertura | [Apertura de caja](#apertura-de-caja), [Monto de apertura](#monto-de-apertura) |
| Cierre | [Cierre de caja](#cierre-de-caja) |
| Arqueo | [Arqueo](#arqueo) |
| Efectivo esperado | [Efectivo esperado](#efectivo-esperado) |
| Monto declarado | [Monto declarado](#monto-declarado) |
| Diferencia (faltante, sobrante) | [Diferencia de arqueo (faltante y sobrante)](#diferencia-de-arqueo-faltante-y-sobrante) |
| Sesión cuadrada frente a cerrada | [Estado de la sesión de caja](#estado-de-la-sesión-de-caja) |
| Venta | [Venta](#venta) |
| Venta en espera (pendiente) | [Venta en espera (pendiente)](#venta-en-espera-pendiente) |
| Venta completada | [Venta completada](#venta-completada) |
| Anulación | [Anulación](#anulación) |
| Devolución | [Devolución](#devolución) |
| Reembolso | [Reembolso](#reembolso) |
| Destino de la devolución | [Destino de la devolución](#destino-de-la-devolución) |
| Vuelto | [Vuelto](#vuelto) |
| Métodos de pago | [Método de pago](#método-de-pago) |
| Número de operación | [Número de operación](#número-de-operación) |
| Pago dividido | [Pago dividido](#pago-dividido) |
| Cobertura de aseguradora | [Cobertura de aseguradora](#cobertura-de-aseguradora) |
| Reclamo de seguro | [Reclamo de seguro](#reclamo-de-seguro) |
| Comprobante impreso | [Comprobante impreso](#comprobante-impreso) |
| Medicamento o producto | [Medicamento (producto)](#medicamento-producto) |
| Código nacional | [Código nacional](#código-nacional) |
| Unidad de medida | [Unidad de medida](#unidad-de-medida) |
| Unidad base | [Unidad base](#unidad-base) |
| Jerarquía de unidades | [Jerarquía de unidades](#jerarquía-de-unidades) |
| Multiplicador | [Multiplicador](#multiplicador) |
| Presentación de venta | [Presentación de venta](#presentación-de-venta) |
| Precio de venta base | [Precio de venta base](#precio-de-venta-base) |
| Stock por sucursal | [Stock por sucursal](#stock-por-sucursal) |
| Ingreso y ajuste de stock | [Ingreso de stock](#ingreso-de-stock), [Ajuste de stock](#ajuste-de-stock) |
| Stock bajo | [Stock bajo](#stock-bajo) |
| Próximo a vencer | [Próximo a vencer](#próximo-a-vencer) |
| Transferencia entre sucursales y sus estados | [Transferencia entre sucursales](#transferencia-entre-sucursales), [Estado de la transferencia (estado logístico)](#estado-de-la-transferencia-estado-logístico) |
| Proveedor | [Proveedor](#proveedor) |
| Precio de compra | [Precio de compra](#precio-de-compra) |
| Historial de precios | [Historial de precios (de proveedor)](#historial-de-precios-de-proveedor) |
| Empresa (tenant) | [Empresa (tenant)](#empresa-tenant) |
| Periodo de prueba | [Periodo de prueba](#periodo-de-prueba) |
| Estados de suscripción | [Estados de suscripción](#estados-de-suscripción) |
| Sede principal | [Sede principal](#sede-principal) |
| Sucursal | [Sucursal](#sucursal) |
| Sucursal base | [Sucursal base](#sucursal-base) |
| Sucursales permitidas | [Sucursales permitidas](#sucursales-permitidas) |
| Empleado | [Empleado](#empleado) |
| Cuenta de usuario | [Cuenta de usuario](#cuenta-de-usuario) |
| Rol | [Rol](#rol) |
| Rol Admin | [Rol Admin](#rol-admin) |
| Permiso (módulo y acción) | [Permiso (módulo y acción)](#permiso-módulo-y-acción) |
| Bloqueo de cuenta | [Bloqueo de cuenta](#bloqueo-de-cuenta) |
| Cambio de contraseña obligatorio | [Cambio de contraseña obligatorio](#cambio-de-contraseña-obligatorio) |
| Borrado lógico | [Borrado lógico](#borrado-lógico) |
| Auditoría | [Auditoría](#auditoría) |
| Carga masiva | [Carga masiva](#carga-masiva) |
| Plantilla | [Plantilla](#plantilla) |
| Exportación | [Exportación](#exportación) |
| Dashboard | [Dashboard](#dashboard) |
| Análisis o escaneo de receta | [Análisis de receta (escaneo)](#análisis-de-receta-escaneo) |
| OCR | [OCR](#ocr) |
| Anonimización | [Anonimización](#anonimización) |
| Gemini | [Gemini](#gemini) |
| Presidio | [Presidio](#presidio) |
| Confianza o puntaje | [Confianza (puntaje)](#confianza-puntaje) |

Términos de la lista: 108 filas, todas con entrada; uno solo («Colegiatura») no se usa en SOFIA.

### 11.3 Opciones del menú lateral (`SPA/matdash/layouts/full/vertical/sidebar/sidebar-data.ts:3-180`)

| Grupo | Opción (`es.json`) | Entrada |
|---|---|---|
| Principal | Dashboard | [Dashboard](#dashboard) |
| Administración | Empresa | [Empresa (tenant)](#empresa-tenant) |
| Administración | Sucursales | [Sucursal](#sucursal) |
| Administración | Empleados | [Empleado](#empleado) |
| Administración | Roles & Permisos | [Rol](#rol), [Permiso (módulo y acción)](#permiso-módulo-y-acción) |
| Administración | Cuentas de Usuario | [Cuenta de usuario](#cuenta-de-usuario) |
| Catálogos | Unidades de Medida | [Unidad de medida](#unidad-de-medida) |
| Catálogos | Jerarquías de Unidades | [Jerarquía de unidades](#jerarquía-de-unidades) |
| Catálogos | Laboratorios | [Laboratorio](#laboratorio) |
| Catálogos | Ingredientes Activos | [Ingrediente activo](#ingrediente-activo) |
| Catálogos | Medicamentos | [Medicamento (producto)](#medicamento-producto) |
| Catálogos | Presentaciones de Venta | [Presentación de venta](#presentación-de-venta) |
| Catálogos | Formulaciones Clínicas | [Formulación clínica](#formulación-clínica) |
| Catálogos | Proveedores | [Proveedor](#proveedor) |
| Catálogos | Aseguradoras | [Aseguradora](#aseguradora) |
| Inventario | Lotes | [Lote](#lote) |
| Inventario | Stock por Sucursal | [Stock por sucursal](#stock-por-sucursal) |
| Punto de Venta | Sesiones de Caja | [Sesión de caja](#sesión-de-caja) |
| Punto de Venta | Ventas | [Venta](#venta) |
| Pacientes & Atención | Pacientes | [Paciente](#paciente) |
| Pacientes & Atención | Profesionales de Salud | [Profesional de salud](#profesional-de-salud) |
| Pacientes & Atención | Recetas Médicas | [Receta médica](#receta-médica) |
| DIGEMID | Catálogo DIGEMID | [Catálogo DIGEMID](#catálogo-digemid) |
| SUNAT | Series Fiscales | [Serie (prefijo)](#serie-prefijo) |

24 de 24 opciones cubiertas.

### 11.4 Resumen de cobertura

| Elemento del inventario | Total | Cubiertos |
|---|---|---|
| Enums de `enums.csv` | 12 | 12 (más `EstadoEmpresa`) |
| Términos de la lista del encargo | 108 filas | 108 (1 marcado «No se usa en SOFIA») |
| Opciones del menú lateral | 24 | 24 |
| Secciones pedidas | 6 | 6 |
| Entradas del glosario | 133 | 133 |

## 12. Fuentes

### 12.1 Código y scripts de SOFIA

- Enums: `sofia-apis/src/SOFIA.Domain/Enums/*.cs` (12 archivos) y `docs/_inventario/enums.csv`.
- Entidades: `sofia-apis/src/SOFIA.Domain/Entities/` (`Empresa`, `Sucursal`, `Empleado`, `Cuenta`, `Rol`, `Medicamento`, `LoteInventario`, `InventarioSucursal`, `RecetaMedica`, `ProfesionalSalud`, `PosSesionCaja`, `Venta`, `VentaPago`, `VentaReclamoSeguro`, `DevolucionCabecera`, `SunatComprobanteEmitido`, `SunatSerieFiscal`, `DigemidCatalogoProducto`, `DigemidInventarioCuarentena`, `DigemidActaDestruccion`, `MagistralOrdenProduccion`, `DespachoDelivery`, `ServicioAgenda`, `ServicioClinicoInmunizacion`, `HistorialPrecioProveedor`, `ProveedorDistribuidor`, `Transferencia`, `UnidadMedida`, `JerarquiaUoM`, `PresentacionVenta`).
- Aplicación: `APP/Ventas/Common/VentaComprobanteGenerator.cs`, `VentaDetalleFactory.cs`, `VentaSeguroProcessor.cs`; `APP/Devoluciones/Commands/ProcesarDevolucion/ProcesarDevolucionCommandHandler.cs`; `APP/POS/Common/ArqueoCajaCalculator.cs`; `APP/POS/Commands/CerrarCaja/CerrarCajaCommandHandler.cs`; `APP/Dashboard/Queries/GetDashboardInventario/GetDashboardInventarioQueryHandler.cs`; `APP/Inventarios/Common/InventarioVendibleExtensions.cs`; `APP/Proveedores/Commands/RegistrarPrecioProveedor/RegistrarPrecioProveedorCommandHandler.cs`; `APP/Recetas/Queries/AnalizarReceta/AnalizarRecetaQueryHandler.cs`; `APP/Empresas/Commands/RegistrarEmpresa/*`; `APP/DIGEMID/Commands/GenerarActaDestruccion/*`; `APP/Common/Behaviors/AuditBehavior.cs`.
- Infraestructura y API: `sofia-apis/src/SOFIA.Infrastructure/Services/GeminiRecetaAnalyzer.cs`, `PresidioPrivacyService.cs`; `CFG/*Configuration.cs` citadas; `API/MedicamentosController.cs`.
- Scripts DbUp (`sofia-apis/src/SOFIA.DbUp/Scripts`): `0001_Initial_Schema.sql`, `0002_SeedBaseRoles.sql`, `0018_SeriesFiscales_UniqueNumbering.sql`, `0020_SoftDeleteAware_Uniques_And_Checks.sql`, `0023_Pacientes_Tipo_Documento.sql`.
- SPA: `sofia-web-spa/src/app/matdash/layouts/full/vertical/sidebar/sidebar-data.ts`; `SPA/pages/pos/models/venta.model.ts`; `SPA/pages/pos/nueva-venta/*`; `SPA/pages/pos/dialogs/devolver-venta-dialog/*`; `SPA/pages/pos/dialogs/comprobante-dialog/*`; `SPA/pages/series-fiscales/models/serie-fiscal.model.ts`; `SPA/pages/recetas-medicas/recetas-scanner/*`.
- Textos de interfaz (`sofia-web-spa/public/i18n`): `es.json` y los `es.json` de `pos`, `sesiones-caja`, `series-fiscales`, `recetas-medicas`, `lotes`, `stock-por-sucursal`, `medicamentos`, `empresa`, `auth`, `cambiar-clave`, `cuentas`, `empleados`, `sucursales`, `roles`, `dashboard`, `transferencias`, `pacientes`, `proveedores`, `aseguradoras`, `profesionalesSalud`, `digemidCatalogo`, `formulacionesClinicas`, `ingredientesActivos`, `laboratorios`, `unidadesMedida`, `presentaciones-venta`, `jerarquias`.
- Inventarios: `docs/_inventario/enums.csv`, `rutas-spa.csv`, `permisos.csv`.

### 12.2 Documentos del proyecto usados como pista (las afirmaciones se verificaron en el código)

[01a](01-reglas-de-negocio/01a-administracion-y-seguridad.md), [01b](01-reglas-de-negocio/01b-catalogos.md), [01c1](01-reglas-de-negocio/01c1-inventario-y-recetas.md), [01c2](01-reglas-de-negocio/01c2-ventas-y-comprobantes.md), [01c3](01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md), [01d](01-reglas-de-negocio/01d-modulos-solo-api.md), [03a](03-modelo-de-datos/03a-maestros-y-seguridad.md), [05b](05-seguridad/05b-matriz-de-permisos.md), [07](07-integraciones.md), [19a](19-inventario-de-pantallas/19a-acceso-layout-y-compartidos.md), [19b](19-inventario-de-pantallas/19b-administracion.md), [19c](19-inventario-de-pantallas/19c-catalogos-1.md).

### 12.3 Fuentes oficiales peruanas consultadas (búsqueda web, 2026-10-13)

- SUNAT, tipos de comprobante de pago (factura, ticket, notas, serie y correlativo): https://cpe.sunat.gob.pe/informacion_general/tipos_comprobantes_pago
- SUNAT, boleta de venta electrónica: https://cpe.sunat.gob.pe/tipos_de_comprobantes/boleta
- SUNAT, nota de débito electrónica: https://cpe.sunat.gob.pe/tipos_de_comprobantes/nota_de_debito
- SUNAT, catálogo 09 de tipos de nota de crédito: https://www.sunat.gob.pe/legislacion/superin/2020/anexo3-193-2020.pdf
- SUNAT, catálogo 06 de tipos de documento de identidad: https://www.sunat.gob.pe/legislacion/superin/2021/anexo-112-2021.pdf
- SUNAT, factura electrónica (serie con letra F, hasta 8 dígitos de correlativo): https://www.sunat.gob.pe/legislacion/superin/2022/anexo-123-2022.pdf
- SUNAT, concepto, tasa y operaciones gravadas del IGV: https://orientacion.sunat.gob.pe/3053-concepto-tasa-y-operaciones-gravadas-igv-empresas
- SUNAT, cálculo del IGV: https://orientacion.sunat.gob.pe/3109-05-calculo-del-impuesto
- SUNAT, inscripción al RUC: https://orientacion.sunat.gob.pe/01-inscripcion-al-ruc-empresas
- gob.pe, Operador de Servicios Electrónicos: https://www.gob.pe/26777-operador-de-servicios-electronicos-ose
- gob.pe, Proveedores de Servicios Electrónicos: https://www.gob.pe/26399-proveedores-de-servicios-electronicos-pse
- gob.pe, DIGEMID: https://www.gob.pe/21087-ministerio-de-salud-direccion-general-de-medicamentos-insumos-y-drogas-digemid
- DIGEMID, registro sanitario de productos farmacéuticos: https://www.digemid.minsa.gob.pe/webDigemid/registro-sanitario/productos-farmaceuticos/
- DIGEMID, venta sin receta: https://www.digemid.minsa.gob.pe/webDigemid/registro-sanitario/productos-farmaceuticos/venta-sin-receta/
- DIGEMID, preguntas frecuentes (DCI): https://www.digemid.minsa.gob.pe/webDigemid/preguntas-frecuentes/
- Minsa, farmacias y boticas deben exigir receta: https://www.gob.pe/institucion/minsa/noticias/962205-farmacias-y-boticas-deben-exigir-receta-antes-de-vender-medicamentos-que-requieren-prescripcion-medica
- Reglamento de estupefacientes, psicotrópicos y otras sustancias (D.S. 023-2001-SA): https://cdn.www.gob.pe/uploads/document/file/283858/255646_DS023-2001.pdf20190110-18386-1k7n0nb.pdf
- Minsa, estándar de dato en salud sobre productos farmacéuticos y ATC: http://www.hnhu.gob.pe/cuerpo/epidemiologia/ASIS%202011%20(F)/contenidos/normastecnicas/02ProductosFarmaceuticos.pdf
- El Peruano, Norma Técnica de Salud para preparados farmacéuticos (R.M. 538-2016/MINSA): https://busquedas.elperuano.pe/normaslegales/aprueban-norma-tecnica-de-salud-para-la-elaboracion-de-prepa-resolucion-ministerial-n-538-2016minsa-1410178-1
- El Peruano, D.S. 004-2021-SA (reglamento de establecimientos farmacéuticos): https://busquedas.elperuano.pe/normaslegales/decreto-supremo-que-modifica-el-reglamento-de-establecimient-decreto-supremo-n-004-2021-sa-1918578-1
- gob.pe, autorización sanitaria para farmacias: https://www.gob.pe/38160-obtener-autorizacion-sanitaria-para-farmacias?child=57900
- DIGEMID, R.M. 132-2015/MINSA (buenas prácticas de almacenamiento): https://www.digemid.minsa.gob.pe/webDigemid/normas-legales/2015/resolucion-ministerial-n-132-2015-minsa/
