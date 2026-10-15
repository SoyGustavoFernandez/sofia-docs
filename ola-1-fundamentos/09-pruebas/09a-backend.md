# Estrategia y catálogo de pruebas automatizadas — Parte A: backend

| | |
|---|---|
| **Sistema** | SOFIA — Sistema Optimizado Farmacéutico con Inteligencia Artificial |
| **Versión del documento** | 1.0 |
| **Fecha** | 2026-10-15 |
| **Backend** | `sofia-apis`, rama `main`, commit `43f5529` |
| **Frontend** | `sofia-web-spa`, rama `main`, commit `eeffbb6` |
| **Método** | Lectura de código (los tres proyectos de prueba, sus `.csproj`, `.husky`, `.github`) y de los inventarios `sofia-docs/_inventario`; los cruces de cobertura se calcularon con un script de solo lectura. No se ejecutó ninguna prueba ni compilación. `git status --short` sin cambios en `sofia-apis` al 2026-10-15 |

Este documento describe qué pruebas automatizadas tiene el backend, cómo están hechas y qué queda sin cubrir. Los comandos para correrlas y los requisitos de Docker y secretos están en la [guía del desarrollador, sección 4.1](../08-guia-del-desarrollador.md#41-backend); aquí solo se agrega lo que esa guía no dice. Para la arquitectura de capas ver [02-arquitectura.md](../02-arquitectura.md). La parte hermana del SPA es [09b-spa.md](09b-spa.md).

## Contenido

1. Estrategia observable
2. Pruebas de arquitectura
3. Catálogo de pruebas unitarias (por módulo)
4. Catálogo de pruebas de integración
5. Cobertura por módulo
6. Huecos
7. Cómo se corren y dónde
8. Pendientes y «No verificado»
9. Tabla de cobertura
10. Fuentes

Convención de conteo: «pruebas» = métodos con `[Fact]` o `[Theory]` (así los cuenta `pruebas-backend.csv`). Un `[Theory]` con varios `[InlineData]` ejecuta más casos que métodos; en `SOFIA.UnitTests` hay 357 líneas `InlineData` repartidas en 48 archivos que contienen `[Theory]`. Rutas abreviadas: `UT/` = `sofia-apis/src/SOFIA.UnitTests/`, `IT/` = `sofia-apis/src/SOFIA.IntegrationTests/`, `AT/` = `sofia-apis/src/SOFIA.ArchitectureTests/`.

---

## 1. Estrategia observable

### 1.1 Cifras

| Proyecto | Archivos de prueba | Pruebas (`[Fact]`/`[Theory]`) | Archivos de apoyo (sin pruebas) |
|---|---|---|---|
| `SOFIA.UnitTests` | 160 | 1 079 | 3: `UT/Security/CuentaFactory.cs`, `UT/Security/EmpresaFactory.cs`, `UT/Ventas/Common/EmisorMocks.cs` |
| `SOFIA.IntegrationTests` | 32 | 172 | 7 en `IT/Infrastructure/` (más `xunit.runner.json`) |
| `SOFIA.ArchitectureTests` | 1 | 3 | — |
| **Total** | **193** | **1 254** | 10 |

Fuente: `sofia-docs/_inventario/pruebas-backend.csv` (193 filas de datos; el `README.md` del inventario dice «186 archivos», cifra desfasada: el CSV tiene 193) y recuento de archivos `.cs` fuera de `bin`/`obj` (203 = 193 + 10 de apoyo). Ninguna prueba está marcada con `Skip`: la búsqueda de `Skip =` en `UnitTests` e `IntegrationTests` no devolvió resultados.

### 1.2 Herramientas y versiones

Leídas de los `.csproj` (todos `net10.0`, `Nullable` habilitado; `UT`/`IT` con `ImplicitUsings`).

| Herramienta | Versión | Proyecto | Fuente |
|---|---|---|---|
| xUnit | 2.9.3 | los tres | `UT/SOFIA.UnitTests.csproj:16`, `IT/SOFIA.IntegrationTests.csproj:19`, `AT/SOFIA.ArchitectureTests.csproj:20` |
| xunit.runner.visualstudio | 3.1.4 | los tres | misma línea siguiente |
| Microsoft.NET.Test.Sdk | 17.14.1 | los tres | `UT/…csproj:13`, `IT/…csproj:17`, `AT/…csproj:18` |
| coverlet.collector | 6.0.4 | los tres | `UT/…csproj:11`, `IT/…csproj:13`, `AT/…csproj:17` |
| FluentAssertions | 8.10.0 | Unit e Integration | `UT/…csproj:12`, `IT/…csproj:15` (en Integration además hay `Using Include="FluentAssertions"` global, `IT/…csproj:25`) |
| Moq | 4.20.72 | solo Unit | `UT/…csproj:15` |
| MockQueryable.Moq | 10.0.8 | solo Unit | `UT/…csproj:14` |
| Microsoft.AspNetCore.Mvc.Testing | 10.0.7 | solo Integration | `IT/…csproj:16` |
| Testcontainers.MsSql | 4.15.0 | solo Integration | `IT/…csproj:18` |
| Microsoft.Extensions.Configuration.UserSecrets | 10.0.7 | solo Integration | `IT/…csproj:14` |
| NetArchTest.eNhancedEdition | 1.4.5 | solo Architecture | `AT/…csproj:19` |

Particularidades: `SOFIA.ArchitectureTests` compila con `TreatWarningsAsErrors=true` y `EnforceCodeStyleInBuild=true` (`AT/…csproj:7-9`); `SOFIA.UnitTests` e `IntegrationTests` no. `SonarAnalyzer.CSharp` no se aplica a ningún proyecto cuyo nombre termina en `Tests` (`sofia-apis/src/Directory.Build.props:7-8`). `SOFIA.Domain` expone sus tipos `internal` a `SOFIA.UnitTests` (`sofia-apis/src/SOFIA.Domain/SOFIA.Domain.csproj:12-13`), por eso las pruebas pueden usar `SetId` y otros miembros internos.

### 1.3 Organización y nombres

- **Carpetas que espejan `SOFIA.Application`**: `UT/<Modulo>/Commands/<NombreCommand>/` y `UT/<Modulo>/Queries/<NombreQuery>/`, con una clase por archivo: `<Command>HandlerTests.cs` y `<Command>ValidatorTests.cs`. Los módulos de `UT/` son los mismos nombres de carpeta de `SOFIA.Application` (por ejemplo `Ventas`, `POS`, `Security`, `Inventarios`), más `Common` para lo transversal.
- **Carpeta `Domain/`** dentro de cada módulo (`UT/<Modulo>/Domain/<Entidad>Tests.cs`) para pruebas de entidades de `SOFIA.Domain`; el módulo se nombra por la carpeta de Application, no por la entidad (por ejemplo `UT/POS/Domain/POSSesionCajaTests.cs` prueba `PosSesionCaja`; `UT/SeriesFiscales/Domain/` prueba `SunatSerieFiscal` y `SunatComprobanteEmitido`).
- **Carpeta `Common/`** dentro de un módulo para servicios auxiliares de Application (`UT/Ventas/Common/VentaDetalleFactoryTests.cs`, `UT/Devoluciones/Common/…`, `UT/Dashboard/Common/…`).
- **Nombres de método**, en inglés: `Handle_Should<Resultado>_When<Condición>` (la mayoría de los handlers), `Validate_Should(Pass|Fail)_When<Condición>` (validadores), `Create_Should…`/`Update_Should…`/`<Metodo>_Should…` (entidades). Hay desviaciones: validadores de Ventas, Recetas y Magistrales usan `Should_Have_Error_When_…`/`Should_Not_Have_Error_When_…` y otros `ValidCommand_ShouldNotHaveAnyValidationErrors`; los de Seguridad y Roles usan `Handle_<Situación>_<Resultado>` (`UT/Security/Commands/Roles/…`). No hay una convención única.
- **Integration** organizado por área funcional, no por módulo de Application (`IT/Api`, `IT/Security`, `IT/Persistence`, `IT/Maestros`, …) y los nombres de clase terminan en `IntegrationTests` (varios archivos terminan solo en `Tests`).
- Los comentarios del código de prueba están en inglés y los `Arrange/Act/Assert` solo se rotulan en las pruebas de arquitectura.

### 1.4 Patrones de las pruebas unitarias

1. **Handlers de Application con `Mock<IApplicationDbContext>`**: 96 archivos de prueba crean `Mock<IApplicationDbContext>` y le asignan `DbSet` falsos con `BuildMockDbSet()` de MockQueryable (`new List<Venta> { _venta }.BuildMockDbSet().Object`, ejemplo `UT/Ventas/Commands/AnularVenta/AnularVentaCommandHandlerTests.cs:43-49`). Las colecciones se declaran como campos `List<T>` para afirmar sobre ellas después del `Handle`. No hay base de datos en memoria ni SQLite: la búsqueda de `UseInMemory`/`Sqlite` en `UT/` no encontró resultados.
2. **Usuario actual simulado** con `Mock<ICurrentUser>` (`IsAuthenticated`, `SucursalId`, `Id`, …), ver `AnularVentaCommandHandlerTests.cs:30-32`.
3. **Qué se afirma**: `Result.IsSuccess`/`IsFailure`, el código y tipo de `Error` (NotFound, Conflict, Forbidden, Validation…), el estado mutado de la entidad y que `SaveChangesAsync` se llame una vez (`Times.Once`) o nunca (`Times.Never`) en los caminos de rechazo.
4. **Entidades de dominio**: se prueban sin dobles, llamando a las fábricas (`Venta.Create(...)`, `LoteInventario.Create(...)`) y verificando el `Result` y el estado.
5. **Validadores FluentValidation**: se instancian directamente; 7 archivos usan `FluentValidation.TestHelper` y 10 usan `TestValidate`/`ShouldHaveValidationError`; el resto llama `Validate(...)` y revisa `Errors`.
6. **Constructores de datos compartidos** (únicos tres archivos de apoyo): `CuentaFactory.WithBaseBranch` arma una cuenta con empleado y sucursal base cargados vía reflexión sobre campos de respaldo (`UT/Security/CuentaFactory.cs`); `EmpresaFactory.WithId` crea una empresa de prueba con un `Id` dado (`UT/Security/EmpresaFactory.cs`); `EmisorMocks.Setup` configura empresa y sucursal emisora del comprobante, con la bandera `ExigirRegistroReceta` fijada por reflexión (`UT/Ventas/Common/EmisorMocks.cs`). Fuera de eso cada clase de prueba arma sus datos en su constructor.
7. **Reflexión sobre campos de respaldo** para poblar propiedades de navegación sin setter público (por ejemplo `<Sucursal_Base>k__BackingField` en `CuentaFactory.cs`); es frágil ante renombres de las propiedades.
8. **Convención de no probar query handlers**: ver 1.7; se cumple casi sin excepción.

### 1.5 Patrones de las pruebas de integración

- **Fábrica de la aplicación**: `SofiaWebAppFactory : WebApplicationFactory<Program>, IAsyncLifetime` (`IT/Infrastructure/SofiaWebAppFactory.cs:54`). Levanta un contenedor `mcr.microsoft.com/mssql/server:2022-latest` con Testcontainers (`:58-60`), arranca la API en entorno `Development` (`:96`), fuerza `Email:Provider=None` y `App:FrontendBaseUrl=https://app.sofia.test` (`:99-100`), reemplaza el `DbContext` por uno apuntado al contenedor (`:105-112`), sustituye `ICurrentUser` por `TestCurrentUser` (`:115-117`) y `IEmailSender` por `CapturingEmailSender` (`:119-120`).
- **Esquema con `EnsureCreated`**: la base de pruebas se crea a partir del modelo de EF Core con `db.Database.EnsureCreated()` (`SofiaWebAppFactory.cs:125`), **no** con los scripts DbUp. Consecuencia: un cambio de esquema hecho solo en un script SQL no se verá en estas pruebas, y los índices, restricciones o columnas que solo existan en los scripts no están probados aquí. Entorno de la API durante las pruebas: `Development` (`:96`).
- **Base de datos por clase de prueba**: `BaseIntegrationTest` implementa `IClassFixture<SofiaWebAppFactory>` (`IT/Infrastructure/BaseIntegrationTest.cs:11`), así que xUnit crea **una fábrica —y por tanto un contenedor SQL Server— por clase** y la comparte entre las pruebas de esa clase. Cada prueba obtiene un `IServiceScope` propio con `ISender` (MediatR) y `ApplicationDbContext` (`:28-31`). Los datos de una clase no se limpian entre sus pruebas: el código crea los datos con identificadores únicos (`Guid.NewGuid()`) y varias clases aíslan cada prueba en una empresa propia (`CurrentUser.Empresa = Guid.NewGuid()` en el constructor, o un `RegistrarEmpresa` nuevo por prueba), ver los comentarios «Every test runs in its own company» en `IT/Ventas/AnularVentaCajaIntegrationTests.cs`, `IT/Maestros/DeleteMaestroInUseIntegrationTests.cs` y `IT/Dashboard/DashboardVentasIntegrationTests.cs`.
- **Inquilino controlado**: `TestCurrentUser` expone `Empresa`, `Empleado`, `Sucursal` y `Admin` modificables; `BaseIntegrationTest` los restablece al construir cada prueba (`BaseIntegrationTest.cs:22-26`) con la empresa fija `6f1b2c3d-0000-4000-8000-000000000001` (`IT/Infrastructure/TestCurrentUser.cs:43`).
- **Ejecución secuencial**: `parallelizeTestCollections` es `false` (`IT/xunit.runner.json:3`).
- **Dos estilos**: (a) pruebas que llaman a handlers por MediatR con la base real (`Sender.Send(...)`), (b) pruebas HTTP de extremo a extremo con `Factory.CreateClient()` contra los controladores reales (`IT/Api`, `IT/Security`).
- **Constructores de datos**: `RegistroEmpresaBuilder.Nuevo` (RUC aleatorio único y correo de administrador, `IT/Infrastructure/RegistroEmpresaBuilder.cs`), `PosCatalogSeeder` (sucursal, empleado, medicamento, lote, stock y ventas, `IT/Infrastructure/PosCatalogSeeder.cs`), `DashboardTestClock.HoyLima()` (inicio del día en Lima, UTC-5, `IT/Infrastructure/DashboardTestClock.cs`).
- **Seis archivos de integración no usan la fábrica ni la base**: `Email/EmailInfrastructureTests.cs`, `Excel/ExcelReaderServiceTests.cs`, `Api/GlobalExceptionHandlerTests.cs`, `Recetas/GeminiRecetaAnalyzerTests.cs`, `Security/JwtOptionsTests.cs`, `Security/MemoryPermissionCacheTests.cs` (no heredan `BaseIntegrationTest`; verificado con `grep`). Son pruebas de infraestructura sin contenedor.

### 1.6 Qué se mockea y qué no

| Se mockea (Moq) | No se mockea |
|---|---|
| `IApplicationDbContext` y sus `DbSet` (MockQueryable) en todas las pruebas de handlers | Entidades y objetos de valor de `SOFIA.Domain` (se construyen reales) |
| `ICurrentUser`, servicios de seguridad (hash de contraseña, tokens, caché de permisos, correo) en las pruebas de Seguridad | El comportamiento del pipeline MediatR en unitarias (los behaviors se prueban por separado: `AuditBehaviorTests`) |
| En integración: `ICurrentUser` (reemplazo por `TestCurrentUser`) y `IEmailSender` (`CapturingEmailSender`) | En integración: SQL Server real, EF Core, MediatR, validación y controladores reales |

El analizador de recetas con IA (Gemini) y el servicio de anonimización (Presidio) no se ejercen contra servicios externos: `IT/Recetas/GeminiRecetaAnalyzerTests.cs` solo tiene 2 pruebas (ver 4).

### 1.7 Convención «no probar query handlers»

Confirmada en el código con una excepción: de los 90 queries de `commands-queries.csv` solo `PreviewImportMedicamentosQueryHandler` tiene prueba de handler (`UT/Medicamentos/Queries/PreviewImportMedicamentos/PreviewImportMedicamentosQueryHandlerTests.cs`, 11 pruebas); los demás archivos bajo `Queries/` son pruebas de validadores (`DashboardHoyValidatorTests`, `AnalizarRecetaQueryValidatorTests`). Algunos queries sí se ejercitan indirectamente en integración (`IT/Dashboard/*`, `IT/Medicamentos/GetMedicamentosPosCatalogIntegrationTests.cs`, `IT/Pacientes/GetPacientesIntegrationTests.cs`, `IT/Ventas/PosLookupsIntegrationTests.cs`); el detalle por query está en la sección 5.

---

## 2. Pruebas de arquitectura

Un solo archivo, `AT/ArchitectureTests.cs`, con 3 pruebas `[Fact]` (`NetArchTest.Rules` 1.4.5). Alcance: ensamblados cargados por nombre (`Assembly.Load("SOFIA.Domain")`, etc.; el proyecto de pruebas referencia los cuatro, `AT/…csproj:29-32`).

| # | Prueba | Regla que verifica | Alcance | Fuente |
|---|---|---|---|---|
| 1 | `Domain_Should_Not_Have_Dependency_On_Other_Projects` | Ningún tipo del ensamblado `SOFIA.Domain` depende de `SOFIA.Application`, `SOFIA.Infrastructure` ni `SOFIA.API` | Todos los tipos de `SOFIA.Domain` | `AT/ArchitectureTests.cs:16-36` |
| 2 | `Application_Should_Only_Depend_On_Domain` | Ningún tipo de `SOFIA.Application` depende de `SOFIA.Infrastructure` ni `SOFIA.API` | Todos los tipos de `SOFIA.Application`. El nombre dice «solo Domain» pero la regla prohíbe únicamente Infrastructure y API; no impide dependencias de otros ensamblados (por ejemplo `SOFIA.SharedKernel` o paquetes) | `AT/ArchitectureTests.cs:38-57` |
| 3 | `Controller_Actions_Should_Declare_Authorization` | Toda acción pública de instancia con atributo de verbo HTTP (`HttpMethodAttribute`) en un `ControllerBase` concreto declara `IAuthorizeData` (`[Authorize]`, `[HasPermission]`) o `IAllowAnonymous` en el método o en la clase (incluye atributos heredados) | Todos los controladores del ensamblado `SOFIA.API`; solo métodos declarados en la clase (`DeclaredOnly`) | `AT/ArchitectureTests.cs:59-77` |

Lo que **no** verifica: que `Infrastructure` no dependa de `API`, nombres de tipos o espacios de nombres, que cada command tenga validador, ni que el permiso declarado sea el correcto (la tabla permiso × endpoint está en `sofia-docs/_inventario/endpoints.csv`).

---
## 3. Catálogo de pruebas unitarias (por módulo)

Una sección por carpeta de `UT/`. Las rutas de la columna «Archivo» son relativas a la carpeta del módulo (`UT/<Módulo>/`). «Pruebas» es el número de `[Fact]`/`[Theory]` del inventario. Todos los handlers se prueban con `Mock<IApplicationDbContext>` y `DbSet` de MockQueryable salvo que se indique otra cosa (ver 1.4). Los códigos de error (`NotFound`, `Conflict`, `Forbidden`, `Validation`) son los tipos de `Error` de `SOFIA.Domain`.

Resumen por módulo:

| Módulo | Archivos | Pruebas |
|---|---|---|
| Auditoria | 1 | 1 |
| Common | 6 | 38 |
| DIGEMID | 7 | 23 |
| Dashboard | 4 | 21 |
| Delivery | 2 | 10 |
| Devoluciones | 4 | 28 |
| Empleados | 6 | 61 |
| Empresas | 3 | 57 |
| FormulacionesClinicas | 6 | 40 |
| IngredientesActivos | 4 | 11 |
| Inventarios | 8 | 57 |
| JerarquiasUoM | 4 | 26 |
| Laboratorios | 5 | 17 |
| Magistrales | 4 | 26 |
| Medicamentos | 8 | 76 |
| POS | 4 | 39 |
| Pacientes | 7 | 29 |
| PresentacionesVenta | 6 | 34 |
| Profesionales | 4 | 11 |
| Proveedores | 7 | 30 |
| Recetas | 7 | 31 |
| Security | 20 | 103 |
| SeriesFiscales | 7 | 44 |
| Servicios | 3 | 18 |
| Sucursales | 5 | 27 |
| Transferencias | 2 | 29 |
| UnidadesMedida | 4 | 12 |
| Ventas | 12 | 180 |
| **Total** | **160** | **1 079** |

(Los totales por módulo se calcularon sumando las filas del CSV con un script de lectura; ver 5.1.)

### 3.1 Auditoria

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/AnonimizarDatosCommandHandlerTests.cs` | `AnonimizarDatosCommandHandler` | 1 | El handler devuelve «no implementado» en vez de un éxito falso (1 caso). |

### 3.2 Common (transversal)

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Behaviors/AuditBehaviorTests.cs` | `AuditBehavior<,>` (pipeline MediatR) | 12 | Escribe exactamente un evento si el command auditable tiene éxito (1); no escribe si falla, si falla la validación, si el command no es auditable o si no hay empleado autenticado (4); usa el Id devuelto como registro afectado en commands de creación y el permiso en la revocación (2); `RegisterInventario` solo audita el ajuste directo de stock (2); el detalle no contiene usuario ni contraseña (registro de cuenta) ni RUC o nombre (actualización de empresa) (2); las constantes de auditoría caben en sus columnas (1). |
| `Excel/CargaMasivaCommandValidatorsTests.cs` | Validadores de carga masiva (límite de filas y longitud de campos) | 5 | Pasa con filas válidas y con el tope exacto de filas (2); falla al superar el límite, sin filas o con un campo sobre el largo máximo (3). |
| `Excel/ImportNameLookupTests.cs` | `ImportNameLookup` | 3 | Resuelve nombres sin distinguir mayúsculas cuando son únicos; falla si el nombre está repetido o es desconocido. |
| `Extensions/QueryableExtensionsTests.cs` | `QueryableExtensions.WhereDateRange` | 5 | Sin límites devuelve todo; incluye ambos límites; trata límites sin zona como UTC; convierte límites locales a UTC sin depender de la zona del servidor; envía los límites como parámetros y no como constantes en línea. |
| `Models/PaginatedListTests.cs` | `PaginatedList<T>` | 5 | Tope del tamaño de página; página inválida cae en la primera; tamaño inválido usa el mínimo; desplazamiento que desbordaría devuelve página vacía; el constructor normaliza la paginación. |
| `Services/SucursalAccessTests.cs` | Servicio de sucursales permitidas (`SucursalAccess`) | 8 | Admin devuelve «sin restricción» (`null`); sin asignaciones solo la sucursal base; incluye asignaciones de la cuenta y de sus roles; `CanAccessAsync` usa la unión; ignora sucursales de rol fuera del inquilino; usuario no autenticado devuelve vacío; consulta una sola vez por petición. |

### 3.3 DIGEMID

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/AislarLoteCuarentena/AislarLoteCuarentenaCommandHandlerTests.cs` | `AislarLoteCuarentenaCommandHandler` | 4 | Registra sesión, sucursal, empleado y hora al aislar un lote (1); NotFound si el lote o la línea de devolución no son de la empresa (2); error si el usuario no tiene sucursal activa (1). |
| `Commands/AislarLoteCuarentena/AislarLoteCuarentenaCommandValidatorTests.cs` | `AislarLoteCuarentenaCommandValidator` | 7 | Comando válido; falla con `LoteId` vacío, cantidad no positiva, motivo demasiado largo y estado de resolución vacío o desconocido; acepta estados conocidos. |
| `Commands/CargaMasivaDigemid/CargaMasivaDigemidCommandHandlerTests.cs` | `CargaMasivaDigemidCommandHandler` | 2 | Omite filas cuya clave existe en la base o antes en el archivo; guarda la fila si la clave solo pertenece a una fila eliminada lógicamente. |
| `Commands/CreateDigemidProducto/CreateDigemidProductoCommandHandlerTests.cs` | `CreateDigemidProductoCommandHandler` | 2 | Conflict sin guardar si `CodProd` ya existe; crea si solo existe en una fila eliminada. |
| `Commands/GenerarActaDestruccion/GenerarActaDestruccionCommandHandlerTests.cs` | `GenerarActaDestruccionCommandHandler` | 2 | Crea el acta si el regente es de la empresa; NotFound si no lo es. |
| `Commands/UpdateDigemidProducto/UpdateDigemidProductoCommandHandlerTests.cs` | `UpdateDigemidProductoCommandHandler` | 3 | Guarda si conserva su propio `CodProd`; Conflict si pertenece a otra fila; guarda si solo pertenece a una eliminada. |
| `Domain/DigemidInventarioCuarentenaTests.cs` | `DigemidInventarioCuarentena` (entidad) | 3 | `Create` y `Update` rechazan un estado de resolución desconocido; `Create` acepta uno conocido. |

### 3.4 Dashboard

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Common/DashboardDiasTests.cs` | `DashboardDias` | 7 | Una ventana por día de más antiguo a hoy; ventanas UTC semiabiertas que comparten frontera sin solaparse; el día local de Lima (UTC-5) empieza en su instante UTC; ventana de un solo día; `EsHoyValido` acepta dentro de la tolerancia, rechaza fuera de ella y rechaza si no se envió. |
| `Common/DashboardScopeTests.cs` | `DashboardScope` | 4 | Orden de los ids de sucursal; `MultiSucursal` verdadero con más de una sucursal y falso con una; vacío sin sucursales. |
| `Common/PagosPorMetodoCalculatorTests.cs` | `PagosPorMetodoCalculator` | 7 | Los cuatro métodos en cero sin ventas; efectivo neto de vuelto; el vuelto se descuenta solo del efectivo en pagos mixtos; vuelto calculado contra el monto a pagar cuando el seguro cubre parte; suma por método entre ventas manteniendo el vuelto de cada una; ignora pagos de ventas no contadas; no baja de cero sin pagos. |
| `Queries/DashboardHoyValidatorTests.cs` | Validador del parámetro `Hoy` del dashboard | 3 | Acepta el inicio del día local en cualquier zona; falla si `Hoy` está a días de distancia o no se envió. |

### 3.5 Delivery

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/ProgramarDelivery/ProgramarDeliveryCommandHandlerTests.cs` | `ProgramarDeliveryCommandHandler` | 6 | Agrega y guarda con venta válida sin despacho (1); el despacho nace en `Preparando` (1); NotFound si la venta es desconocida o ajena (1); Forbidden si la sucursal de la venta no está permitida (1); Conflict si la venta está anulada o ya tiene despacho (2). |
| `Domain/DespachoDeliveryTests.cs` | `DespachoDelivery` (entidad) | 4 | Cambio de estado cuando la transición es permitida; rechazo conservando el estado cuando no lo es; permite editar otros campos si el estado no cambia; `Create` empieza en `Preparando`. |

### 3.6 Devoluciones

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/ProcesarDevolucion/ProcesarDevolucionCommandHandlerTests.cs` | `ProcesarDevolucionCommandHandler` | 14 | Rechazos, una prueba cada uno: sesión sin empleado, venta inexistente, venta de otra sucursal (Forbidden), el llamador sin sesión de caja abierta, venta anulada, venta sin comprobante emitido, sin serie activa de nota de crédito (falla en vez de omitir la NC), detalle inexistente, cantidad mayor a la vendida, devoluciones previas más la solicitada sobre lo vendido (10); la misma línea repetida en el pedido se limita a su suma (1); devolución parcial válida acredita solo las líneas devueltas y reembolsa por la sesión (1); devolver las últimas unidades marca la venta `Devuelta` (1); si el seguro cubrió parte, se reembolsa solo la parte del cliente (1). |
| `Commands/ProcesarDevolucion/ProcesarDevolucionCommandValidatorTests.cs` | `ProcesarDevolucionCommandValidator` | 8 | Comando válido; `ComprobanteOrigenId` vacío; motivo SUNAT o sustento inválidos; detalles inválidos o lista vacía; destino y método de reembolso fuera del enum; método de reembolso omitido toma `Efectivo`. |
| `Common/CantidadesDevueltasExtensionsTests.cs` | `GetCantidadesDevueltasAsync` (extensión) | 3 | Suma por línea cuando se devolvió varias veces; ignora líneas de otra venta; ignora devoluciones o líneas eliminadas lógicamente. |
| `Domain/DevolucionCabeceraTests.cs` | `DevolucionCabecera` (entidad) | 3 | `RegistrarReembolso` guarda sesión, monto y método; falla con sesión vacía y con monto negativo. |

### 3.7 Empleados

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CreateEmpleado/CreateEmpleadoCommandHandlerTests.cs` | `CreateEmpleadoCommandHandler` | 8 | Agrega y guarda (1); sin apellido materno si viene en blanco (1); NotFound si la sucursal no es del inquilino (1); falla de dominio sin guardar (1); Conflict por licencia repetida y por correo repetido sin distinguir mayúsculas (2); crea si licencia o correo solo pertenecen a un empleado eliminado, con el correo normalizado (2). |
| `Commands/CreateEmpleado/CreateEmpleadoCommandValidatorTests.cs` | `CreateEmpleadoCommandValidator` | 11 | Comando válido; licencia nula; correo vacío o válido pasa, mal formado o demasiado largo falla; sucursal vacía; nombres vacíos; apellido paterno demasiado largo; apellido materno en blanco pasa y demasiado largo falla; licencia demasiado larga. |
| `Commands/DeleteEmpleado/DeleteEmpleadoCommandHandlerTests.cs` | `DeleteEmpleadoCommandHandler` | 3 | NotFound si no existe; Conflict si es gerente de una sucursal; elimina y guarda si no gerencia ninguna. |
| `Commands/UpdateEmpleado/UpdateEmpleadoCommandHandlerTests.cs` | `UpdateEmpleadoCommandHandler` | 13 | NotFound; actualiza y guarda; vacía el apellido materno; falla de dominio sin guardar; NotFound si la nueva sucursal no es del inquilino; Forbidden si el empleado cambia su propia sucursal; permite su propia edición si la sucursal no cambia; invalida el sello de seguridad solo cuando cambia la sucursal (2); Conflict por licencia o correo de otro empleado (2); guarda si conserva su propia licencia o correo normalizado (2). |
| `Commands/UpdateEmpleado/UpdateEmpleadoCommandValidatorTests.cs` | `UpdateEmpleadoCommandValidator` | 9 | Comando válido; correo vacío/válido; correo mal formado o largo; `Id` vacío; sucursal vacía; nombres vacíos; apellido materno en blanco y largo; licencia larga. |
| `Domain/EmpleadoTests.cs` | `Empleado` (entidad) | 17 | `Create`: éxito con campos obligatorios, sin licencia, sin apellido materno (en blanco) o con espacios recortados; falla sin sucursal, sin nombres, sin apellido paterno; correo recortado y en minúsculas, vacío si viene en blanco, falla si está mal formado o excede el máximo (11). `Update`: vacía el apellido materno, recorta, muta campos válidos, no muta si es inválido, reemplaza y limpia el correo, conserva el correo si el nuevo es inválido (6). |

### 3.8 Empresas

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/RegistrarEmpresa/RegistrarEmpresaCommandHandlerTests.cs` | `RegistrarEmpresaCommandHandler` | 17 | Conflict por RUC o usuario repetidos (2); devuelve JWT y emite refresh token (2); guarda las cuatro entidades (empresa, sucursal, empleado, cuenta) (1); crea el rol Admin propio de la empresa y no reutiliza uno existente (2); valores por defecto de campos opcionales y uso de los suministrados (2); licencia vacía/recortada (2); dirección recortada en la sede principal (1); RUC guardado y falla sin guardar si el RUC viene vacío (2); empleado admin con nombres y correo normalizado y sin apellido materno si no se da (2); no guarda si falla la validación (1). |
| `Commands/RegistrarEmpresa/RegistrarEmpresaCommandValidatorTests.cs` | `RegistrarEmpresaCommandValidator` | 18 | Solo obligatorios pasa; falla con nombre de empresa vacío, usuario o contraseña cortos o vacíos, RUC vacío o inválido (pasa con RUC válido), nombres/apellido paterno vacíos o de más de 75 caracteres, apellido materno omitido o válido y de más de 75, correo vacío, inválido o largo, dirección de la sede vacía o de más de 255 caracteres (pasa en el máximo). |
| `Domain/EmpresaTests.cs` | `Empresa` (entidad) | 22 | `Create` con solo nombre o con RUC; falla con nombre vacío o RUC inválido; nace en `TrialActivo`, vence a 30 días, `TenantId` igual a su propio id (7). Vigencia: `EstaVigente` y `EstaVigenteEn` según estado, periodo, suspendida, cancelada, trial vencido y eliminada lógicamente (7). `ActivarSuscripcion` y `ExtenderTrial` (4). `Update` válido, con RUC nulo y falla con nombre vacío (3). Por defecto no exige registro de receta (1). |

### 3.9 FormulacionesClinicas

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CreateFormulacionClinica/CreateFormulacionClinicaCommandHandlerTests.cs` | `CreateFormulacionClinicaCommandHandler` | 7 | Agrega y guarda; NotFound si falta medicamento, ingrediente o unidad de medida (3); falla de dominio sin guardar; Conflict si el producto ya lista el ingrediente; crea si el mismo ingrediente solo está en una formulación eliminada. |
| `Commands/CreateFormulacionClinica/CreateFormulacionClinicaCommandValidatorTests.cs` | `CreateFormulacionClinicaCommandValidator` | 7 | Válido; falla con producto, ingrediente o unidad vacíos, concentración/dosis no positiva, código TE/Orange demasiado largo; pasa con el código nulo. |
| `Commands/DeleteFormulacionClinica/DeleteFormulacionClinicaCommandHandlerTests.cs` | `DeleteFormulacionClinicaCommandHandler` | 2 | NotFound si no existe; elimina y guarda. |
| `Commands/UpdateFormulacionClinica/UpdateFormulacionClinicaCommandHandlerTests.cs` | `UpdateFormulacionClinicaCommandHandler` | 8 | NotFound; actualiza; NotFound si el nuevo ingrediente o la nueva unidad no existen; no verifica existencia si no cambiaron; falla de dominio; Conflict si el nuevo ingrediente ya está listado; guarda si solo coincide con una eliminada. |
| `Commands/UpdateFormulacionClinica/UpdateFormulacionClinicaCommandValidatorTests.cs` | `UpdateFormulacionClinicaCommandValidator` | 6 | Válido; falla con `Id`, ingrediente o unidad vacíos, concentración no positiva y código largo. |
| `Domain/FormulacionClinicaTests.cs` | `FormulacionClinica` (entidad) | 10 | `Create` válido y con código nulo; falla con producto, ingrediente, unidad vacíos o concentración no positiva (6); `Update` muta campos y falla sin mutar con ingrediente vacío, concentración no positiva o unidad vacía (4). |

### 3.10 IngredientesActivos

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CargaMasivaIngredientesActivos/CargaMasivaIngredientesActivosCommandHandlerTests.cs` | `CargaMasivaIngredientesActivosCommandHandler` | 2 | Omite filas con clave (DCI) existente en base o repetida en el archivo; guarda si la clave solo es de una fila eliminada. |
| `Commands/CreateIngredienteActivo/CreateIngredienteActivoCommandHandlerTests.cs` | `CreateIngredienteActivoCommandHandler` | 2 | Conflict sin guardar si el DCI está tomado; crea si solo pertenece a una fila eliminada. |
| `Commands/DeleteIngredienteActivo/DeleteIngredienteActivoCommandHandlerTests.cs` | `DeleteIngredienteActivoCommandHandler` | 4 | NotFound; Conflict si lo referencia una formulación; elimina si solo lo referencian formulaciones eliminadas; elimina sin dependientes. |
| `Commands/UpdateIngredienteActivo/UpdateIngredienteActivoCommandHandlerTests.cs` | `UpdateIngredienteActivoCommandHandler` | 3 | Guarda con su propio DCI; Conflict si es de otra fila; guarda si solo es de una eliminada. |

### 3.11 Inventarios

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/AdjustStock/AdjustStockCommandHandlerTests.cs` | `AdjustStockCommandHandler` | 4 | NotFound si el registro no existe; error de validación con cantidad negativa; actualiza y guarda; Forbidden si la sucursal no está permitida. |
| `Commands/AdjustStock/AdjustStockCommandValidatorTests.cs` | `AdjustStockCommandValidator` | 4 | Válido; cantidad cero pasa; `Id` vacío y cantidad negativa fallan. |
| `Commands/CreateLote/CreateLoteInventarioValidatorTests.cs` | Validador de creación de lote | 9 | Válido; fabricación nula; producto vacío; número de lote vacío o largo; caducidad pasada falla, el día de caducidad no terminado pasa y el ya terminado falla; fabricación futura falla. |
| `Commands/DeleteLote/DeleteLoteInventarioCommandHandlerTests.cs` | `DeleteLoteInventarioCommandHandler` | 4 | NotFound; Conflict si el lote tiene stock; Conflict si tiene ventas; elimina sin stock ni ventas. |
| `Commands/RegisterInventario/RegisterInventarioCommandHandlerTests.cs` | `RegisterInventarioCommandHandler` | 4 | Crea la fila de stock si la sucursal no tiene; Conflict si una petición concurrente la creó antes; Forbidden si la sucursal no está permitida; NotFound si la sucursal no es del inquilino. |
| `Commands/UpdateLote/UpdateLoteInventarioValidatorTests.cs` | Validador de actualización de lote | 8 | Mismas reglas que la creación aplicadas a la actualización (`Id` vacío, número de lote, caducidad por día, fabricación futura). |
| `Domain/InventarioSucursalTests.cs` | `InventarioSucursal` (entidad) | 4 | `Create` falla con cantidad negativa; `AdjustStock` fija la nueva cantidad, permite cero y falla con negativa. |
| `Domain/LoteInventarioTests.cs` | `LoteInventario` (entidad) | 20 | `Create` válido, sin fabricación, falla con producto vacío, número vacío o de más de 100 caracteres, caducidad no futura, día de caducidad ya terminado, fabricación futura (9). `Update` (4). `EstaVencido` en los bordes del día de caducidad (inicio, último segundo, medianoche siguiente, con desplazamiento guardado) (6) y coherencia con `UltimaCaducidadVencida` (1). |

### 3.12 JerarquiasUoM

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CargaMasivaJerarquiasUoM/CargaMasivaJerarquiasUoMCommandHandlerTests.cs` | `CargaMasivaJerarquiasUoMCommandHandler` | 5 | Guarda una cadena consistente; omite filas que contradicen una fila anterior del archivo o una conversión existente; omite la arista repetida; omite nombres ambiguos en lugar de lanzar. |
| `Commands/CreateJerarquiaUoM/CreateJerarquiaUoMCommandHandlerTests.cs` | `CreateJerarquiaUoMCommandHandler` | 7 | NotFound si producto o unidad son desconocidos/ajenos (2); crea si ninguna conversión conecta las unidades; crea arista redundante coherente; Conflict si contradice la cadena resuelta; ignora conversiones de otros productos; Conflict si la misma conversión ya existe. |
| `Commands/UpdateJerarquiaUoM/UpdateJerarquiaUoMCommandHandlerTests.cs` | `UpdateJerarquiaUoMCommandHandler` | 8 | NotFound (3); actualiza sin conflictos; no choca consigo misma al cambiar su multiplicador; Conflict si contradice otra cadena o si otra fila ya tiene la misma conversión (2); guarda conservando su propia conversión. |
| `Common/JerarquiaConversionResolverTests.cs` | `JerarquiaConversionResolver` | 6 | Arista directa; multiplica a lo largo de varios saltos; divide de menor a mayor; devuelve 1 para la misma unidad; `null` si nada conecta; sigue el camino más corto con arista redundante coherente. |

### 3.13 Laboratorios

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CargaMasivaLaboratorios/CargaMasivaLaboratoriosCommandHandlerTests.cs` | `CargaMasivaLaboratoriosCommandHandler` | 3 | Omite filas cuyo nombre o código existe; conserva solo la primera si el archivo repite una clave; guarda si la clave es de uno eliminado. |
| `Commands/CreateLaboratorio/CreateLaboratorioCommandHandlerTests.cs` | `CreateLaboratorioCommandHandler` | 5 | Agrega y guarda; Conflict por nombre o código tomados (2); crea si otro laboratorio también carece de código; crea si las claves son de uno eliminado. |
| `Commands/DeleteLaboratorio/DeleteLaboratorioCommandHandlerTests.cs` | `DeleteLaboratorioCommandHandler` | 3 | NotFound; Conflict si tiene medicamentos; elimina si no los tiene. |
| `Commands/UpdateLaboratorio/UpdateLaboratorioCommandHandlerTests.cs` | `UpdateLaboratorioCommandHandler` | 4 | Guarda con su nombre y código; Conflict por nombre o código de otro; guarda si el código es de uno eliminado. |
| `Domain/LaboratorioTests.cs` | `Laboratorio` (entidad) | 2 | `Create` y `Update` guardan código nulo si viene en blanco. |

### 3.14 Magistrales

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CompletarOrdenMagistral/CompletarOrdenMagistralCommandHandlerTests.cs` | `CompletarOrdenMagistralCommandHandler` | 6 | Camino feliz: completa la orden y crea inventario (1); fallos: orden inexistente, ya completada, cantidad producida inválida, orden no en proceso (4); Forbidden si es de otra sucursal (1). |
| `Commands/CompletarOrdenMagistral/CompletarOrdenMagistralCommandValidatorTests.cs` | `CompletarOrdenMagistralCommandValidator` | 5 | Válido; falla con orden vacía, número de lote vacío o caducidad pasada; pasa si el día de caducidad no terminó. |
| `Commands/IniciarOrdenMagistral/IniciarOrdenMagistralCommandHandlerTests.cs` | `IniciarOrdenMagistralCommandHandler` | 10 | `LoteVencido` si el día de caducidad del insumo ya terminó, si el lote no está cargado o si un insumo posterior está vencido sin guardar nada (3); crea la orden si el lote vence hoy y con químico y sucursal de la sesión (2); NotFound si el stock es de otra sucursal, el inventario no existe, el producto o la receta no son de la empresa (4); falla con stock insuficiente (1). |
| `Commands/IniciarOrdenMagistral/IniciarOrdenMagistralCommandValidatorTests.cs` | `IniciarOrdenMagistralCommandValidator` | 5 | Válido; producto resultante vacío; sin consumos; cantidad producida ausente o no positiva; cantidad consumida negativa. |

### 3.15 Medicamentos

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CargaMasivaMedicamentos/CargaMasivaMedicamentosCommandHandlerTests.cs` | `CargaMasivaMedicamentosCommandHandler` | 8 | Guarda todas las filas válidas; resuelve catálogos sin distinguir mayúsculas; omite fila con laboratorio o unidad desconocidos o condición de venta inválida (3); omite solo las filas ambiguas cuando la descripción de unidad se repite; guarda solo las válidas en una mezcla; omite códigos existentes en base o antes en el archivo. |
| `Commands/CreateMedicamento/CreateMedicamentoCommandHandlerTests.cs` | `CreateMedicamentoCommandHandler` | 6 | NotFound si laboratorio o unidad base son desconocidos/ajenos (2); agrega y guarda; falla de dominio sin guardar; Conflict por código nacional tomado; crea si solo es de uno eliminado. |
| `Commands/CreateMedicamento/CreateMedicamentoCommandValidatorTests.cs` | `CreateMedicamentoCommandValidator` | 9 | Válido; código nacional vacío o largo; nombre comercial vacío o largo; laboratorio o unidad base vacíos; condición de venta fuera del enum; precio de venta base negativo. |
| `Commands/DeleteMedicamento/DeleteMedicamentoCommandHandlerTests.cs` | `DeleteMedicamentoCommandHandler` | 11 | NotFound; Conflict si lo referencia un lote, presentación, jerarquía, formulación, precio de proveedor, cita, inmunización u orden magistral (8); elimina si todos los dependientes están eliminados lógicamente; elimina sin dependientes. |
| `Commands/UpdateMedicamento/UpdateMedicamentoCommandHandlerTests.cs` | `UpdateMedicamentoCommandHandler` | 8 | NotFound (laboratorio, unidad, medicamento); omite búsquedas si las referencias no cambiaron; actualiza y guarda; falla de dominio; Conflict por código nacional de otro; guarda conservando el propio. |
| `Commands/UpdateMedicamento/UpdateMedicamentoCommandValidatorTests.cs` | `UpdateMedicamentoCommandValidator` | 8 | Válido; `Id`, código, laboratorio, unidad vacíos; nombre largo; condición fuera del enum; precio negativo. |
| `Domain/MedicamentoTests.cs` | `Medicamento` (entidad) | 15 | `Create` válido (1) y falla por código nacional ausente o demasiado largo, nombre comercial ausente o demasiado largo, laboratorio o unidad vacíos (6); `Update` muta o no muta si es inválido (2); `CondicionesValidas` coincide con las descripciones del enum (1); precio de venta base: se guarda, queda nulo, rechaza negativo en `Create` y `Update`, y cambia en `Update` (5). |
| `Queries/PreviewImportMedicamentos/PreviewImportMedicamentosQueryHandlerTests.cs` | `PreviewImportMedicamentosQueryHandler` | 11 | Única prueba de handler de consulta. Vista previa de importación: sin errores con fila válida; marca requerido (código, nombre), longitud máxima (código, nombre), duplicado en base y duplicado en archivo, no encontrado (laboratorio, unidad), opción inválida (condición de venta) y calcula los totales. |

### 3.16 POS (caja)

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/AperturarCaja/AperturarCajaCommandHandlerTests.cs` | `AperturarCajaCommandHandler` | 8 | Toma cajero, sucursal y hora de la sesión (1); Conflict si el cajero ya tiene caja abierta o si una petición concurrente abrió una antes (2); relanza si el guardado falla por otra razón (1); abre si solo otro cajero tiene caja abierta (1); error sin sucursal activa (1); error con monto de apertura negativo y éxito con cero (2). |
| `Commands/AperturarCaja/AperturarCajaCommandValidatorTests.cs` | `AperturarCajaCommandValidator` | 2 | Pasa con monto no negativo; falla con negativo. |
| `Commands/CerrarCaja/CerrarCajaCommandHandlerTests.cs` | `CerrarCajaCommandHandler` | 17 | NotFound; Forbidden si quien cierra no la abrió y no es admin; el admin puede cerrar la de otro cajero (3). El efectivo esperado se calcula en el servidor y se registra la diferencia (1); descuenta la cobertura del seguro al calcular el vuelto (1); toma el vuelto del efectivo en pagos mixtos (1); solo el fondo de apertura sin ventas (1); resta solo los reembolsos en efectivo de esta sesión, con o sin ventas (2); tratamiento de ventas anuladas: no cuenta la venta anulada desde la misma sesión que la cobró ni la anulación heredada sin sesión, resta el neto en efectivo si la venta de otra sesión se anuló aquí, sigue contando si se cobró aquí y se anuló desde otra sesión, no mueve efectivo si no se pagó en efectivo, resta solo la parte neta en efectivo con Yape + efectivo y solo el copago en efectivo con seguro (7); combinación de cobradas, anulaciones de otras sesiones y reembolsos (1). |
| `Domain/POSSesionCajaTests.cs` | `PosSesionCaja` (entidad) | 12 | `Create` válido, con apertura cero, falla con sucursal o empleado vacíos y apertura negativa (5); `Cerrar`: estado `Cuadrada` si el arqueo es cero y `Cerrada` si no, calcula la diferencia, falla si no está abierta, si el cierre es anterior a la apertura, o con monto declarado o calculado negativo (7). |

### 3.17 Pacientes

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CargaMasivaPacientes/CargaMasivaPacientesCommandHandlerTests.cs` | `CargaMasivaPacientesCommandHandler` | 2 | Omite claves existentes o repetidas en el archivo; guarda si la clave es de uno eliminado. |
| `Commands/CreatePaciente/CreatePacienteCommandHandlerTests.cs` | `CreatePacienteCommandHandler` | 4 | Conflict si el documento está tomado; crea si es de uno eliminado; guarda el tipo de documento; falla sin guardar si el número no corresponde al tipo. |
| `Commands/CreatePaciente/CreatePacienteCommandValidatorTests.cs` | `CreatePacienteCommandValidator` | 3 | Pasa si el número corresponde al tipo; falla con tipo ausente o desconocido y con número que no corresponde. |
| `Commands/DeletePaciente/DeletePacienteCommandHandlerTests.cs` | `DeletePacienteCommandHandler` | 7 | NotFound; Conflict si lo referencia una receta, venta, cita o inmunización (4); elimina si todos los dependientes están eliminados; elimina sin dependientes. |
| `Commands/UpdatePaciente/UpdatePacienteCommandHandlerTests.cs` | `UpdatePacienteCommandHandler` | 5 | Guarda con su propio documento; Conflict si es de otra fila; guarda si es de uno eliminado; cambia el tipo de documento si el nuevo número corresponde; falla sin tipo. |
| `Commands/UpdatePaciente/UpdatePacienteCommandValidatorTests.cs` | `UpdatePacienteCommandValidator` | 3 | Igual que el de creación. |
| `Domain/PacienteClienteTests.cs` | `PacienteCliente` (entidad) | 5 | `Create` con número acorde al tipo; falla con tipo ausente/desconocido o número que no corresponde; `Update` exige el tipo y lo guarda; `TipoDocumentoIdentidad.Parse` acepta códigos y nombres cortos. |

### 3.18 PresentacionesVenta

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CreatePresentacionVenta/CreatePresentacionVentaCommandHandlerTests.cs` | `CreatePresentacionVentaCommandHandler` | 5 | Crea; resuelve cadenas de varios saltos al vender por unidad intermedia; NotFound si no existe el producto; Conflict si ya existe presentación para esa unidad; error de validación si ninguna jerarquía conecta la unidad con la base. |
| `Commands/CreatePresentacionVenta/CreatePresentacionVentaCommandValidatorTests.cs` | `CreatePresentacionVentaCommandValidator` | 4 | Válido; falla con producto o unidad vacíos y precio negativo. |
| `Commands/DeletePresentacionVenta/DeletePresentacionVentaCommandHandlerTests.cs` | `DeletePresentacionVentaCommandHandler` | 3 | NotFound; Conflict si tiene ventas; elimina sin ventas. |
| `Commands/UpdatePresentacionVenta/UpdatePresentacionVentaCommandHandlerTests.cs` | `UpdatePresentacionVentaCommandHandler` | 5 | NotFound; actualiza; recalcula la cantidad al cambiar de unidad; Conflict si otra presentación usa la unidad; error si la unidad no tiene camino a la base. |
| `Commands/UpdatePresentacionVenta/UpdatePresentacionVentaCommandValidatorTests.cs` | `UpdatePresentacionVentaCommandValidator` | 4 | Válido; falla con `Id` o unidad vacíos y precio negativo. |
| `Domain/PresentacionVentaTests.cs` | `PresentacionVenta` (entidad) | 13 | `Create` válido y falla por producto, unidad, descripción vacía o larga, cantidad en unidades base cero o negativa, precio negativo (precio cero pasa) (8); `Update` cambia campos y falla por unidad, descripción, cantidad o precio inválidos (5). |

### 3.19 Profesionales

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CargaMasivaProfesionalesSalud/CargaMasivaProfesionalesSaludCommandHandlerTests.cs` | `CargaMasivaProfesionalesSaludCommandHandler` | 2 | Omite claves existentes o repetidas; guarda si es de una fila eliminada. |
| `Commands/CreateProfesionalSalud/CreateProfesionalSaludCommandHandlerTests.cs` | `CreateProfesionalSaludCommandHandler` | 2 | Conflict si el registro está tomado; crea si es de uno eliminado. |
| `Commands/DeleteProfesionalSalud/DeleteProfesionalSaludCommandHandlerTests.cs` | `DeleteProfesionalSaludCommandHandler` | 4 | NotFound; Conflict si lo referencia una receta; elimina si solo lo referencian recetas eliminadas; elimina sin dependientes. |
| `Commands/UpdateProfesionalSalud/UpdateProfesionalSaludCommandHandlerTests.cs` | `UpdateProfesionalSaludCommandHandler` | 3 | Guarda con su propio registro; Conflict si es de otra fila; guarda si es de una eliminada. |

### 3.20 Proveedores

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CargaMasivaProveedores/CargaMasivaProveedoresCommandHandlerTests.cs` | `CargaMasivaProveedoresCommandHandler` | 2 | Omite filas cuyo identificador tributario o razón social existe en base o antes en el archivo; guarda si las claves son de uno eliminado. |
| `Commands/CreateProveedor/CreateProveedorCommandHandlerTests.cs` | `CreateProveedorCommandHandler` | 3 | Conflict por identificador tributario o razón social tomados (2); crea si las claves son de uno eliminado. |
| `Commands/DeleteProveedor/DeleteProveedorCommandHandlerTests.cs` | `DeleteProveedorCommandHandler` | 4 | NotFound; Conflict si tiene precios; elimina si solo tiene precios eliminados; elimina sin dependientes. |
| `Commands/RegistrarPrecioProveedor/RegistrarPrecioProveedorCommandHandlerTests.cs` | `RegistrarPrecioProveedorCommandHandler` | 7 | NotFound si falta proveedor o medicamento (2); agrega un precio abierto con el descuento aplicado (1); redondea el costo neto a 4 decimales (1); cierra el precio abierto anterior del mismo proveedor y producto (1); no toca precios de otros proveedores ni los ya cerrados (1); no toca el precio abierto de otro producto (1). |
| `Commands/RegistrarPrecioProveedor/RegistrarPrecioProveedorCommandValidatorTests.cs` | `RegistrarPrecioProveedorCommandValidator` | 7 | Válido; descuento en los límites; precio máximo; precio no positivo; precio sobre el máximo; descuento fuera de rango; ids vacíos. |
| `Commands/UpdateProveedor/UpdateProveedorCommandHandlerTests.cs` | `UpdateProveedorCommandHandler` | 3 | Guarda con sus claves; Conflict si el identificador tributario es de otro; guarda si la razón social es de uno eliminado. |
| `Domain/HistorialPrecioProveedorTests.cs` | `HistorialPrecioProveedor` (entidad) | 4 | `Create` deja el precio abierto sin fecha de fin; `CerrarVigencia` fija la fecha de fin si es posterior al inicio y acepta el mismo instante; falla conservando el precio abierto si el fin es anterior. |

### 3.21 Recetas

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CreateReceta/CreateRecetaCommandHandlerTests.cs` | `CreateRecetaCommandHandler` | 4 | Agrega y guarda si paciente y médico son del inquilino; NotFound sin guardar si paciente o médico son desconocidos/ajenos (2); guarda el número de receta recortado. |
| `Commands/CreateReceta/CreateRecetaCommandValidatorTests.cs` | `CreateRecetaCommandValidator` | 3 | `NumeroReceta` falla si excede el máximo, pasa en el máximo y pasa si falta. |
| `Commands/DeleteReceta/DeleteRecetaCommandHandlerTests.cs` | `DeleteRecetaCommandHandler` | 3 | NotFound; Conflict si la receta se usa en ventas; elimina si no se usa. |
| `Commands/UpdateReceta/UpdateRecetaCommandHandlerTests.cs` | `UpdateRecetaCommandHandler` | 4 | Guarda si las referencias no cambian; NotFound sin cambiar si el nuevo paciente o médico son desconocidos/ajenos (2); guarda el número recortado. |
| `Commands/UpdateReceta/UpdateRecetaCommandValidatorTests.cs` | `UpdateRecetaCommandValidator` | 3 | Igual que el de creación. |
| `Domain/RecetaMedicaTests.cs` | `RecetaMedica` (entidad) | 11 | `PuedeDispensar`: éxito con producto OTC, con receta controlada sin usar, con receta simple con repeticiones; falla con receta controlada ya dispensada y con receta simple agotada (5). Número de receta: queda vacío si no se da, se recorta y vacío pasa a nulo, acepta el máximo tras recortar, falla si excede (4); `Update` reemplaza (recortado o limpio) y falla conservando el número si excede (2). |
| `Queries/AnalizarReceta/AnalizarRecetaQueryValidatorTests.cs` | Validador de `AnalizarRecetaQuery` | 3 | Especialidad vacía o texto simple pasa; falla con caracteres que no son letras; falla si excede el largo máximo. |

### 3.22 Security

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/ChangePassword/ChangePasswordCommandHandlerTests.cs` | `ChangePasswordCommandHandler` | 3 | Actualiza la contraseña y revoca todas las sesiones si la actual es correcta; falla sin cambios si es incorrecta; falla si la cuenta está inactiva. |
| `Commands/ChangePassword/ChangePasswordCommandValidatorTests.cs` | `ChangePasswordCommandValidator` | 5 | Pasa con contraseña nueva fuerte y distinta; falla con `CuentaId` vacío, contraseña actual vacía, nueva débil o igual a la actual. |
| `Commands/Cuentas/CuentaAdminProtectionTests.cs` | `UpdateCuentaCommandHandler` y `DeleteCuentaCommandHandler` | 5 | Un no administrador no puede desactivar ni eliminar una cuenta administradora (Forbidden, código `Cuenta.AdminProtegida`, estado 403) (2); un administrador sí puede (2); un no administrador puede desactivar una cuenta común (1). |
| `Commands/ForgotPassword/ForgotPasswordCommandHandlerTests.cs` | `ForgotPasswordCommandHandler` | 8 | Guarda solo el hash del token; emite un token distinto cada vez; envía al empleado un enlace con fragmento y el token en claro; no emite token pero responde igual si la cuenta está inactiva, el empleado no tiene correo, está eliminado o la empresa no está vigente (4); responde éxito genérico aunque el correo no se pueda enviar. |
| `Commands/Login/LoginCommandHandlerTests.cs` | `LoginCommandHandler` | 8 | Credenciales válidas devuelven tokens; cuenta bloqueada devuelve el mismo error que contraseña errónea; sucursal base de otro inquilino igual error y sin tokens; usuario desconocido igual verifica una contraseña (anti enumeración por tiempo); empresa no vigente con contraseña válida devuelve Forbidden sin tokens, con contraseña errónea devuelve credenciales genéricas; empresa eliminada o inexistente devuelve Forbidden (2). |
| `Commands/Logout/LogoutCommandHandlerTests.cs` | `LogoutCommandHandler` | 8 | Revoca la sesión de la cookie con el access token vencido; revoca la del principal sin cookie; éxito sin cambios sin principal ni cookie, con cookie desconocida o vencida (3); revoca la sesión rotada si un refresh concurrente la rotó; reintenta y revoca el token emitido entretanto; se rinde tras tres conflictos de concurrencia. |
| `Commands/RefreshToken/RefreshTokenCommandHandlerTests.cs` | `RefreshTokenCommandHandler` | 10 | Rota un token activo; Unauthorized sin emitir token si la sucursal base es de otro inquilino, la empresa no está vigente (sin revocar), está eliminada, el token venció o es desconocido (5); revoca todas las sesiones si un token rotado se reutiliza pasado el periodo de gracia y conserva las demás dentro de la gracia (2); Unauthorized ante rotación concurrente y ante conflicto al revocar la sesión reutilizada (2). |
| `Commands/Register/RegisterAccountCommandHandlerTests.cs` | `RegisterAccountCommandHandler` | 5 | NotFound si el empleado no existe; Conflict si ya tiene cuenta o el usuario está tomado; crea la cuenta si todo pasa; no guarda si falla alguna comprobación. |
| `Commands/ResetPassword/ResetPasswordCommandHandlerTests.cs` | `ResetPasswordCommandHandler` | 9 | Revoca los refresh tokens al restablecer; registra el evento de auditoría para el dueño; no lo registra con token inválido; falla si se presenta el hash almacenado como token, si el token se reutiliza o venció; conserva los refresh tokens si el token es inválido; usuario desconocido devuelve el mismo error que un token inválido y aun así calcula el hash. |
| `Commands/Roles/AssignPermission/AssignPermissionToRolCommandHandlerTests.cs` | `AssignPermissionToRolCommandHandler` | 6 | Un no administrador no puede dar permisos a su propio rol (Forbidden) pero sí a otro; un administrador sí; permiso nuevo o restaurado invalida la caché de permisos del rol (2); un permiso duplicado no la invalida. |
| `Commands/Roles/AssignRol/AssignRolToUserCommandHandlerTests.cs` | `AssignRolToUserCommandHandler` | 4 | Asigna un rol a otra cuenta y rota el sello de seguridad; la autoasignación es Forbidden; un no administrador no puede asignar el rol Admin, un administrador sí. |
| `Commands/Roles/CargaMasivaRoles/CargaMasivaRolesCommandHandlerTests.cs` | `CargaMasivaRolesCommandHandler` | 2 | Omite nombres existentes o repetidos en el archivo; guarda si el nombre es de un rol eliminado. |
| `Commands/Roles/DeleteRol/DeleteRolCommandHandlerTests.cs` | `DeleteRolCommandHandler` | 2 | Un rol sin uso invalida la caché de permisos; un rol desconocido devuelve NotFound sin invalidar. |
| `Commands/Roles/RemoveRol/RemoveRolFromUserCommandHandlerTests.cs` | `RemoveRolFromUserCommandHandler` | 5 | Quita un rol común y rota el sello de seguridad; la autoeliminación es Forbidden; un no administrador no puede quitar el rol Admin, un administrador sí; quitar un rol no asignado tiene éxito sin cambios. |
| `Commands/Roles/RevokePermission/RevokePermissionFromRolCommandHandlerTests.cs` | `RevokePermissionFromRolCommandHandler` | 2 | Un permiso existente se elimina lógicamente e invalida la caché; uno desconocido devuelve NotFound sin invalidar. |
| `Commands/Roles/SetSucursales/SetRolSucursalesCommandHandlerTests.cs` | `SetRolSucursalesCommandHandler` | 3 | Reemplaza las asignaciones si todas las sucursales son del inquilino; NotFound si una está fuera del inquilino; una lista vacía limpia las asignaciones. |
| `Domain/CuentaPasswordTests.cs` | `Cuenta` (entidad, contraseña y bloqueo) | 9 | `UpdatePassword` rota el sello y limpia el cambio forzado; `GenerateRecoveryToken` guarda solo el hash; `ResetPassword` funciona una vez si el hash coincide y falla si difiere, si no hubo token o si venció (4); `RegisterFailedAttempt` bloquea tras cinco fallos, reinicia el conteo con el bloqueo vencido y vuelve a bloquear tras otros cinco (3). |
| `Domain/CuentaTenantTests.cs` | `Cuenta.SucursalBasePerteneceAlTenant` | 4 | Verdadero para la misma empresa; falso para sucursal de otra empresa, cuenta sin inquilino o empleado no cargado. |
| `Domain/RefreshTokenTests.cs` | `RefreshToken.IsReuseAttempt` | 3 | Falso con token activo y con revocado dentro de la gracia; verdadero con revocado antes de la gracia. |
| `PasswordRecoveryEmailTests.cs` | `PasswordRecoveryEmail` | 2 | Rellena ambas partes (texto y HTML) desde las plantillas embebidas; codifica en HTML el nombre del empleado en la parte HTML. |

### 3.23 SeriesFiscales

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CreateSerieFiscal/CreateSerieFiscalCommandHandlerTests.cs` | `CreateSerieFiscalCommandHandler` | 8 | Crea la serie sin conflictos; NotFound si la sucursal no es del inquilino; Conflict si el prefijo está tomado para el tipo; permite el mismo prefijo para otro tipo; Conflict si la sucursal ya tiene una serie activa del tipo; permite una inactiva si ya hay una activa; mapea la carrera sobre el índice único a Conflict; relanza si el guardado falla por otra razón. |
| `Commands/CreateSerieFiscal/CreateSerieFiscalCommandValidatorTests.cs` | `CreateSerieFiscalCommandValidator` | 8 | Válido; sucursal vacía; prefijo de boleta inválido; prefijo que corresponde al tipo pasa y el que no corresponde falla; tipo no configurable; correlativo negativo; estado desconocido. |
| `Commands/DeleteSerieFiscal/DeleteSerieFiscalCommandHandlerTests.cs` | `DeleteSerieFiscalCommandHandler` | 3 | NotFound; Conflict si la serie emitió documentos; elimina si no se usó. |
| `Commands/UpdateSerieFiscal/UpdateSerieFiscalCommandHandlerTests.cs` | `UpdateSerieFiscalCommandHandler` | 7 | NotFound; actualiza sin conflictos; Conflict si cambia el correlativo o el prefijo tras emitir documentos (2); permite cambiar el estado tras emitir; Conflict al activar si otra serie está activa; Conflict si el nuevo prefijo está tomado para el tipo. |
| `Commands/UpdateSerieFiscal/UpdateSerieFiscalCommandValidatorTests.cs` | `UpdateSerieFiscalCommandValidator` | 5 | Válido; `Id` vacío; prefijo de factura que empieza con B; correlativo negativo; estado desconocido. |
| `Domain/SunatComprobanteEmitidoTests.cs` | `SunatComprobanteEmitido` (entidad) | 4 | `Create` deja el estado «no enviado» sin artefactos SUNAT; `Anular` marca el documento anulado y falla si ya está anulado; el estado cabe en la columna `varchar(20)`. |
| `Domain/SunatSerieFiscalTests.cs` | `SunatSerieFiscal` (entidad) | 9 | `EsPrefijoValido` aplica las reglas SUNAT; `Create` válido y falla por tipo no configurable, prefijo que no corresponde, correlativo negativo, estado desconocido; `Update` cambia el estado; `CambiaNumeracion` es falso si solo difiere el estado y verdadero si difieren prefijo, correlativo, tipo o sucursal. |

### 3.24 Servicios (agenda e inmunizaciones)

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/AgendarServicio/AgendarServicioCommandHandlerTests.cs` | `AgendarServicioCommandHandler` | 4 | Agenda si las referencias son de la empresa; NotFound si el paciente, el producto o la venta no son de la empresa (3). |
| `Commands/RegistrarInmunizacion/RegistrarInmunizacionCommandHandlerTests.cs` | `RegistrarInmunizacionCommandHandler` | 12 | `LoteVencido` si el día de caducidad terminó antes de la dosis o si la fecha de dosis es posterior al día de caducidad de un lote vigente (2); registra si el lote vence el día de la dosis o si la dosis se dio antes de vencer (2); lee la fecha de dosis sin desplazamiento como UTC y conserva el instante si es local (2); registra si las referencias son de la empresa (1); NotFound si el paciente, el profesional, el producto, el lote de otro producto o la venta no corresponden (5). |
| `Commands/RegistrarInmunizacion/RegistrarInmunizacionCommandValidatorTests.cs` | `RegistrarInmunizacionCommandValidator` | 2 | Válido; falla si falta la fecha de administración física. |

### 3.25 Sucursales

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CargaMasivaSucursales/CargaMasivaSucursalesCommandHandlerTests.cs` | `CargaMasivaSucursalesCommandHandler` | 2 | Omite claves existentes o repetidas; guarda si es de una eliminada. |
| `Commands/CreateSucursal/CreateSucursalCommandHandlerTests.cs` | `CreateSucursalCommandHandler` | 4 | NotFound si el gerente es desconocido/ajeno; crea si existe en el inquilino; Conflict por licencia tomada; crea si la licencia es de una fila eliminada. |
| `Commands/DeleteSucursal/DeleteSucursalCommandHandlerTests.cs` | `DeleteSucursalCommandHandler` | 13 | NotFound; Conflict si la referencia un empleado, una cuenta asignada, un rol asignado, inventario, sesión de caja, serie fiscal, venta, transferencia (como origen o destino), cuarentena u orden magistral (10); elimina si todos los dependientes están eliminados lógicamente; elimina sin dependientes. |
| `Commands/UpdateSucursal/UpdateSucursalCommandHandlerTests.cs` | `UpdateSucursalCommandHandler` | 5 | Guarda conservando su licencia; NotFound sin cambiar si el nuevo gerente es desconocido/ajeno; guarda si existe en el inquilino; Conflict por licencia de otra fila; guarda si es de una eliminada. |
| `Domain/SucursalTests.cs` | `Sucursal` (entidad) | 3 | `CreateSedePrincipal` deja la licencia vacía con datos válidos y falla con nombre o dirección en blanco; `Update` exige licencia si la sede principal se creó sin ella. |

### 3.26 Transferencias

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CreateTransferencia/CreateTransferenciaCommandHandlerTests.cs` | `CreateTransferenciaCommandHandler` | 6 | Unauthorized sin usuario autenticado; error de validación si origen y destino son iguales o el stock es insuficiente (2); NotFound si la sucursal destino no existe o el lote no está en la sucursal origen (2); crea la transferencia si el pedido es válido. |
| `Domain/TransferenciaTests.cs` | `Transferencia` (entidad) | 23 | `Create` válido y falla con origen o destino vacíos, origen igual a destino, emisor vacío o sin detalles (6); `Aprobar` solo desde `Iniciada` (2); `Despachar` desde `Iniciada` o `Aprobada` y falla `EnTransito` (3); `Recibir` falla si no está en tránsito, receptor vacío, lote ajeno, cantidad recibida negativa o mayor que la enviada (5), termina `Completada` si todo coincide y `RecibidaParcial` si es menos (2); `Cancelar` desde `Iniciada`/`Aprobada` y falla desde `Completada`, `RecibidaParcial` o ya `Cancelada` (5). |

### 3.27 UnidadesMedida

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/CargaMasivaUnidadesMedida/CargaMasivaUnidadesMedidaCommandHandlerTests.cs` | `CargaMasivaUnidadesMedidaCommandHandler` | 2 | Omite claves existentes o repetidas; guarda si es de una eliminada. |
| `Commands/CreateUnidadMedida/CreateUnidadMedidaCommandHandlerTests.cs` | `CreateUnidadMedidaCommandHandler` | 2 | Conflict si el código está tomado; crea si es de una eliminada. |
| `Commands/DeleteUnidadMedida/DeleteUnidadMedidaCommandHandlerTests.cs` | `DeleteUnidadMedidaCommandHandler` | 5 | NotFound; Conflict si la referencia un medicamento, una jerarquía o una formulación clínica (3); elimina sin referencias. |
| `Commands/UpdateUnidadMedida/UpdateUnidadMedidaCommandHandlerTests.cs` | `UpdateUnidadMedidaCommandHandler` | 3 | Guarda con su código; Conflict si es de otra fila; guarda si es de una eliminada. |

### 3.28 Ventas

| Archivo | Clase bajo prueba | Pruebas | Qué verifica |
|---|---|---|---|
| `Commands/ActualizarVentaPendiente/ActualizarVentaPendienteCommandHandlerTests.cs` | `ActualizarVentaPendienteCommandHandler` | 10 | NotFound sin cambiar si el nuevo cliente es desconocido/ajeno; asigna el cliente del inquilino; error si la venta no existe o no está pendiente (2); libera el stock anterior y reserva el nuevo al reemplazar líneas; error si el nuevo lote tiene stock insuficiente; `RecetaRequerida` si una línea nueva exige receta; no cuenta el uso de su propia receta al reemplazar líneas; empresa flexible estaciona sin confirmación una línea de receta simple sin receta; empresa estricta devuelve `RecetaRequerida`. |
| `Commands/ActualizarVentaPendiente/ActualizarVentaPendienteCommandValidatorTests.cs` | `ActualizarVentaPendienteCommandValidator` | 3 | Falla con `VentaId` vacío o sin detalles; pasa con comando válido. |
| `Commands/AnularVenta/AnularVentaCommandHandlerTests.cs` | `AnularVentaCommandHandler` | 11 | Anula y restaura el stock si no tiene devoluciones; guarda la sesión de caja abierta del llamador si la venta estaba cobrada; rechaza sin cambiar nada si estaba cobrada y el llamador no tiene sesión abierta, si la única sesión abierta es de otro empleado, o si la propia está cerrada o en otra sucursal (3); anula una venta estacionada sin sesión; informa «ya anulada» aunque el llamador no tenga sesión; rechaza si la venta tiene devoluciones; Forbidden si la venta es de otra sucursal; anula la boleta de la venta pero no otros documentos; deja intacta la boleta si la anulación se rechaza. |
| `Commands/CompletarVenta/CompletarVentaCommandHandlerTests.cs` | `CompletarVentaCommandHandler` | 21 | Cliente nuevo desconocido (NotFound sin guardar), venta inexistente, de otra sucursal o no pendiente (4); completa la venta pendiente y emite comprobante si el pago cubre el total; rechaza sin consumir correlativo si la sucursal no tiene serie de boleta activa; emite con la serie activa de la sucursal; pago insuficiente; pago no efectivo sin referencia (5). Seguro: NotFound si la aseguradora no existe, ignora cobertura sin aseguradora, error si la cobertura excede el total, crea el reclamo si la aseguradora es válida (4). Caja: cobra en la sesión abierta del llamador si la venta se estacionó en otra, error sin sesión abierta o si la única abierta es de otro cajero o sucursal (3). Recetas: `VerificacionRequerida` sin guardar si una venta flexible de receta simple se cobra sin confirmación, guarda el empleado que cobra y no quien estacionó, `RecetaRequerida` si la línea exige receta registrada, `NumeroRequerido` si la receta controlada guardada no tiene número, ignora la confirmación si ninguna línea la necesita (5). |
| `Commands/CompletarVenta/CompletarVentaCommandValidatorTests.cs` | `CompletarVentaCommandValidator` | 8 | Falla con `VentaId` vacío, sin pagos, pago no efectivo sin referencia, monto cubierto por seguro negativo o cobertura sin aseguradora; pasa con referencia, con aseguradora y con comando válido. |
| `Commands/CreateVenta/CreateVentaCommandHandlerTests.cs` | `CreateVentaCommandHandler` | 33 | Usuario no autenticado; cliente desconocido/ajeno (NotFound sin guardar); sesión de caja nula, inexistente o cerrada, de otro cajero o de otra sucursal (5); lote en cuarentena, lote inexistente en la sucursal, stock insuficiente (3); crea la venta con éxito y con presentación; rechaza la venta pagada sin consumir correlativo si no hay serie de boleta; crea venta pendiente sin comprobante si no hay pagos; pagos insuficientes; presentación inexistente o de otro producto; pago no efectivo sin referencia; vuelto sin pago en efectivo; cobra el precio del catálogo en unidades base; `SinPrecio` si el producto no tiene precio; toma una foto del costo vigente del proveedor al crear el detalle; seguro: aseguradora inexistente, cobertura ignorada sin aseguradora, cobertura mayor al total, reclamo creado (4); recetas: `RecetaRequerida` con producto controlado sin receta, `VerificacionRequerida` sin guardar en empresa flexible sin confirmación, guarda quién verificó y cuándo con confirmación, estaciona sin guardar verificación si no hay pagos, `RecetaRequerida` en empresa estricta con receta simple, ignora la confirmación si ninguna línea la necesita (6). |
| `Commands/CreateVenta/CreateVentaCommandValidatorTests.cs` | `CreateVentaCommandValidator` | 13 | Falla sin detalles, con `LoteId` vacío, cantidad cero o negativa, sin pagos (salvo estado `Pendiente`, que pasa), monto pagado no positivo, pago no efectivo sin referencia (el efectivo sin referencia pasa; con referencia pasa), monto cubierto por seguro negativo y cobertura sin aseguradora (pasa con aseguradora); pasa con comando válido. |
| `Common/VentaComprobanteGeneratorTests.cs` | `VentaComprobanteGenerator` | 9 | `DesglosarIgv` extrae el IGV de un total que lo incluye; `EmitirBoletaAsync` falla sin consumir correlativo si la sucursal no tiene serie de boleta activa o la empresa no tiene RUC (2); emite con la serie activa y separa el IGV del total; registra la boleta como «no enviada» sin datos SUNAT simulados; devuelve los datos del emisor; factura al cliente de la venta con su tipo y número de documento; factura al cliente anónimo sin cliente; factura a un cliente antiguo sin tipo de documento como «Varios». |
| `Common/VentaDetalleFactoryTests.cs` | `VentaDetalleFactory` | 31 | Caducidad: `Vencido` si el día de caducidad ya terminó, éxito si vence hoy (2). Recetas: éxito con OTC sin receta, no exige verificación si solo hay líneas OTC, `Requerida` si el producto con receta no la tiene, exige verificación en empresa flexible con receta simple sin receta, no la exige con receta registrada vinculada, `OtroPaciente` con receta ajena o sin cliente o con receta ajena en línea OTC, `Agotada` con receta agotada, `NumeroRequerido` con producto controlado sin número, éxito con número, éxito con receta controlada sin usar, `NoEncontrada` si la receta no existe (14). Dispensaciones: cuenta una dispensación si varias líneas comparten receta, `Agotada` si la controlada ya se dispensó, ignora ventas anuladas, ignora la venta pendiente que se reemplaza, ignora usos previos solo OTC, permite repeticiones restantes y `Agotada` si se usaron todas (7). `ExigeRegistroRecetaAsync` devuelve la bandera de la empresa y es estricto si no se resuelve (2). `RequiereVerificacionRecetaAsync` sobre líneas guardadas: exige verificación en empresa flexible, `Requerida`, `NumeroRequerido`, no exige si la receta cubre las líneas, `NoEncontrada`, lote no encontrado si no se resuelve el producto (6). |
| `Common/VentaSeguroProcessorTests.cs` | `VentaSeguroProcessor` | 5 | `Process` omite el reclamo sin aseguradora, falla si no se puede crear, agrega el reclamo con cobertura válida; `ResolveCoberturaAsync` devuelve cero sin aseguradora y NotFound si no existe. |
| `Domain/VentaPagoTests.cs` | `VentaPago` (entidad) | 5 | `Create` falla con pago no efectivo sin referencia y pasa en efectivo sin referencia o no efectivo con ella; `Update` falla al cambiar a no efectivo sin referencia; `RequiereReferencia` es verdadero solo para métodos no efectivo. |
| `Domain/VentaTests.cs` | `Venta` (entidad) | 31 | Estado inicial: `Completada` por defecto, `Pendiente` para venta estacionada, empieza sin verificación de receta (3). `RegistrarPagos`: completa la pendiente si el pago cubre el total, falla si está anulada, si la suma es insuficiente, si la cobertura del seguro está fuera de límites, solo exige el copago si la cobertura está dentro del total, falla si el vuelto excede el efectivo recibido y permite vuelto cubierto (7). `ActualizarDetalles` reemplaza y recalcula el total si está pendiente, falla si no está pendiente o la lista nueva está vacía (3). `AsignarSesion` mueve la venta pendiente, falla si no está pendiente o la sesión es vacía (3). `Anular`: falla si está devuelta, guarda la sesión que devuelve el efectivo si estaba cobrada, no necesita sesión si estaba estacionada, falla sin guardar sesión si ya está anulada o el motivo está vacío (5). `RegistrarDevolucion`: devuelve el monto acreditado y sigue `Completada` si es parcial, toca la venta en devolución parcial para provocar conflicto de `RowVersion` en devoluciones concurrentes, no la toca si se rechaza, pasa a `Devuelta` si todas las líneas se devolvieron, falla si lo previo más lo pedido excede lo vendido, si la línea no pertenece, si no está `Completada` o ya está `Devuelta` (8). `RegistrarVerificacionReceta` guarda quién verificó y cuándo, y falla sin guardar si el empleado es vacío (2). |

---

## 4. Catálogo de pruebas de integración

Rutas relativas a `IT/`. «Base real» = SQL Server 2022 en contenedor con esquema creado por `EnsureCreated` (1.5). «HTTP» = `Factory.CreateClient` contra los controladores reales; «MediatR» = `Sender.Send(...)` directo al pipeline real (validación, transacción, auditoría). Muchas clases aíslan cada prueba en una empresa nueva (`CurrentUser.Empresa = Guid.NewGuid()` o `RegistrarEmpresa` con `RegistroEmpresaBuilder`) para que los datos de una prueba no cuenten en otra.

Resumen: 32 archivos, 172 pruebas; 26 archivos usan contenedor SQL Server y 6 no (1.5).

### 4.1 Api (HTTP y manejo de errores)

| Archivo | Tipo | Pruebas | Flujo y datos | Qué verifica |
|---|---|---|---|---|
| `Api/ExportRequestValidationIntegrationTests.cs` | HTTP, base real | 5 | Registra una empresa nueva por prueba (`RegistrarEmpresa`), usa su token en `POST /api/v1/pacientes/exportar`, `/laboratorios/exportar` y `/empresas/exportar` | Devuelve el libro Excel con formato de fecha permitido; 400 con formato de fecha no permitido; 400 si los encabezados exceden los límites; 400 si las etiquetas Sí/No son demasiado largas; `ExportDateFormatAttribute` solo acepta formatos conocidos. |
| `Api/GlobalExceptionHandlerTests.cs` | Unidad del manejador (sin contenedor) | 4 | Invoca `TryHandleAsync` sobre un `HttpContext` simulado | Conflict con código estable si el `RowVersion` está obsoleto; 400 con código si se rechaza la carga Excel; 413 con código si el cuerpo excede el límite; 500 ante excepción inesperada. |
| `Api/LoggingConfigurationTests.cs` | Configuración, usa la fábrica | 2 | Resuelve `ILoggerFactory` de la aplicación | Las categorías de EF Core (comandos SQL), de hosting y del `HttpClient` no registran nivel Información (para que el texto SQL y las cadenas de consulta con DNI de pacientes no lleguen a los logs) pero sí Advertencia; la categoría `SOFIA.Application` sí registra Información. |
| `Api/NotFoundResponseIntegrationTests.cs` | HTTP, base real | 1 | Empresa nueva; `GET /api/v1/recetas/{id}` con un id inexistente | Responde 404. |
| `Api/RegistrarEmpresaValidationIntegrationTests.cs` | HTTP, base real | 2 | `POST /api/v1/empresas/registrar` anónimo | 400 con código de dirección si falta `DireccionSede`; guarda la dirección enviada en la sede principal. |
| `Api/SecurityHeadersIntegrationTests.cs` | HTTP, base real | 4 | Usa rutas de error de prueba (`/api/v1/test-errors/missing`, `/boom`), una ruta inexistente y `/health` | Las respuestas 404, 401 y 500 llevan los encabezados de seguridad y `no-store`; `/health` expone solo estado y marca de tiempo. |

### 4.2 Auditoria, concurrencia y persistencia

| Archivo | Tipo | Pruebas | Flujo y datos | Qué verifica |
|---|---|---|---|---|
| `Auditoria/AuditTrailIntegrationTests.cs` | MediatR, base real | 3 | `CerrarCajaCommand` sobre una sesión de caja sembrada | Un cierre exitoso confirma exactamente una fila de auditoría con el cambio; un cierre fallido no escribe auditoría; si la fila de auditoría no se puede escribir (empleado administrador inexistente viola la FK) el cambio se revierte y se lanza `DbUpdateException`. |
| `Concurrency/ConcurrencyIntegrationTests.cs` | `DbContext` real en dos ámbitos | 4 | Siembra sucursal, empleado, inventario (5 unidades) y ventas; dos contextos leen la misma fila | `DbUpdateConcurrencyException` si dos peticiones venden las últimas 5 unidades desde la misma lectura; si dos anulaciones leen la venta `Completada`; si dos devoluciones parciales se limitaron contra la misma lectura; y el índice único `UX_POS_Sesiones_Caja_Empleado_Abierta` rechaza una segunda caja abierta del mismo cajero. |
| `Persistence/DatabaseConstraintIntegrationTests.cs` | MediatR, `DbContext` y manejador de excepciones, base real | 9 | Pacientes, laboratorios y empleados reales; excepciones SQL reales | Documento repetido solo de un paciente eliminado se acepta y de uno vigente da Conflict; dos laboratorios sin código no chocan; el índice único filtrado rechaza la segunda fila viva (si se salta la comprobación de la aplicación); dos empleados vivos con igual correo chocan y los que no tienen correo no; el manejador responde Conflict sin registrar la clave ni el documento en el log para índice único, restricción única y clave foránea, y 400 para una restricción CHECK. |
| `Persistence/TransactionRollbackIntegrationTests.cs` | MediatR, base real | 1 | Registra un command de prueba (`FailAfterWriteCommand`) que guarda un laboratorio y falla | Al lanzar tras `SaveChanges`, `TransactionBehavior` revierte: no queda el laboratorio. |

### 4.3 Seguridad

| Archivo | Tipo | Pruebas | Flujo y datos | Qué verifica |
|---|---|---|---|---|
| `Security/AuthIntegrationTests.cs` | MediatR y HTTP, base real | 14 | Registra empresas de prueba; `LoginCommand`, `RegisterAccountCommand`, `LogoutCommand`, `RefreshTokenCommand`, `ResetPasswordCommand`, `GET /api/v1/auth/me`, `GET /api/v1/laboratorios/plantilla`, `POST /api/v1/auth/change-password`; correos capturados por `CapturingEmailSender` | Inicio de sesión devuelve JWT; contraseña incorrecta devuelve no autorizado; usuario repetido devuelve Conflict; cerrar sesión y restablecer contraseña revocan el refresh token emitido en el registro anónimo; cerrar sesión con solo la cookie; reutilizar un token rotado revoca la sesión; empresa suspendida bloquea inicio de sesión, refresh y tokens de acceso vigentes; empresa activa acepta el token; el cambio forzado de contraseña solo permite `change-password` hasta completarse; flujo completo de recuperación (olvido, enlace por correo con token en el fragmento, restablecer, iniciar con la nueva); sin correo no se envía nada; el administrador registrado recibe el correo; el refresh conserva el nombre de empresa en el nuevo token. |
| `Security/CsrfHeaderIntegrationTests.cs` | HTTP, base real | 4 | `POST /api/v1/auth/refresh` y `/auth/logout`, y un preflight `OPTIONS` | 403 con código CSRF si falta el encabezado en endpoints de cookie (2 rutas); con encabezado llega a la acción y el cierre de sesión funciona; el preflight permite el encabezado solo para orígenes configurados. |
| `Security/JwtOptionsTests.cs` | Unidad de opciones (sin contenedor) | 3 | `JwtOptions.EnsureValid` | Válida con clave de 32 bytes, emisor y audiencia; lanza si la clave falta o mide menos de 32 bytes; lanza si emisor o audiencia están vacíos. |
| `Security/MemoryPermissionCacheTests.cs` | Unidad de la caché (sin contenedor) | 3 | `MemoryPermissionCache` | Devuelve lo guardado ignorando mayúsculas del rol; otra empresa con el mismo nombre de rol no comparte la caché; `Invalidate` quita solo la empresa y el rol indicados. |
| `Security/RateLimitingIntegrationTests.cs` | HTTP, base real | 6 | `POST /api/v1/auth/login` y `/auth/refresh` con IP simuladas | 429 si una misma IP excede el límite de autenticación; otra IP no se limita; el refresh no se limita al agotar la cuota de login; la partición por empresa aísla empresas con la misma IP, comparte la cuota dentro de una misma empresa y cae a la IP si el usuario no tiene empresa. |
| `Security/TenantIsolationIntegrationTests.cs` | MediatR y `DbContext`, base real | 9 | Dos empresas; filas con y sin inquilino | `SaveChanges` asigna el inquilino actual; el filtro oculta filas de otro inquilino y las filas sin inquilino para cualquier empresa y contexto anónimo; el índice único de documento permite el mismo documento en inquilinos distintos; dos registros de empresa crean cada uno su rol Admin; el inicio de sesión funciona con cuenta de un inquilino en contexto anónimo; login y refresh ignoran roles enlazados de otro inquilino o sin inquilino; la caché de permisos no se comparte con un rol del mismo nombre en otra empresa; `UpdateEmpleado` rechaza una sucursal base de otro inquilino. |

### 4.4 Maestros y catálogos

| Archivo | Tipo | Pruebas | Flujo y datos | Qué verifica |
|---|---|---|---|---|
| `Maestros/DeleteMaestroInUseIntegrationTests.cs` | MediatR, base real | 14 | Una empresa por prueba; siembra el maestro y su dependiente (lote, empleado, cuenta, rol, receta, venta, formulación, precio de proveedor) | Para medicamento, sucursal (empleado, cuenta o rol asignado), paciente (receta o venta), profesional (receta), ingrediente activo (formulación) y proveedor (precios): el borrado se rechaza si hay dependientes y la consulta de uso se traduce a SQL real, y el registro sigue visible; sin dependientes se elimina lógicamente. |
| `Medicamentos/CreateMedicamentoIntegrationTests.cs` | MediatR, base real | 3 | Laboratorio y unidad sembrados | Persiste el medicamento; error de validación si el código nacional está vacío; NotFound sin persistir nada si el laboratorio no existe. |
| `Medicamentos/GetMedicamentosPosCatalogIntegrationTests.cs` | MediatR (query), base real | 7 | `PosCatalogSeeder`: sucursales, medicamentos, lotes, stock y ventas | Catálogo del POS: alfabético sin ventas; los más vendidos primero y luego el resto alfabético; paginación del catálogo ordenado; ignora ventas que no cuentan (otra sucursal, antiguas o no completadas); omite un más vendido eliminado sin acortar la página; cuenta solo el stock vendible de la propia sucursal con la marca POS; falla si se ordena por ventas sin sucursal en el token. |
| `FormulacionesClinicas/CreateFormulacionClinicaIntegrationTests.cs` | MediatR, base real | 3 | Medicamento, ingrediente y unidad sembrados | Persiste la formulación; NotFound si la unidad no existe; la búsqueda por nombre de ingrediente devuelve los medicamentos que lo contienen. |
| `Empleados/EmpleadoSinApellidoMaternoIntegrationTests.cs` | MediatR (command y queries), base real | 3 | Empleados con y sin apellido materno | Guarda nulo y se lee un nombre limpio con el materno en blanco; el filtro por apellido materno no falla si hay empleados sin él; cuentas y sucursales muestran el nombre sin hueco. |
| `Laboratorios/LaboratoriosExportIntegrationTests.cs` | MediatR (query de exportación), base real | 2 | Filas sembradas por encima y por debajo del tope | Rechaza con «demasiadas filas» si el resultado excede el límite de exportación; devuelve el libro si cabe. |
| `Pacientes/GetPacientesIntegrationTests.cs` | MediatR (query), base real | 1 | Varios pacientes con el mismo nombre | La paginación alfabética no repite ni salta filas. |
| `Proveedores/PreciosProveedorIntegrationTests.cs` | MediatR, base real | 7 | Empresa propia; proveedor y medicamento; precios; una venta | Registrar guarda el costo neto y la consulta lo devuelve como vigente; registrar cierra el precio anterior del mismo producto y lo conserva en el historial; no toca precios de otro proveedor; la consulta pagina del más nuevo al más antiguo con orden estable; NotFound si el proveedor es de otra empresa; NotFound sin guardar si proveedor o producto son de otra empresa; `CreateVenta` toma como costo el del precio vigente del proveedor e ignora el reemplazado. |

### 4.5 Ventas, caja y dashboard

| Archivo | Tipo | Pruebas | Flujo y datos | Qué verifica |
|---|---|---|---|---|
| `Ventas/AnularVentaCajaIntegrationTests.cs` | MediatR, base real | 6 | Empresa propia; ventas cobradas en una sesión de caja y sesiones de caja de otros cajeros | `AnularVenta` se rechaza sin cambiar nada si la venta estaba cobrada y el llamador no tiene caja abierta; saca el efectivo de la caja del llamador si la sesión que cobró ya está cerrada; deja la caja como antes de la venta si se anula desde la misma sesión que cobró; una venta estacionada no necesita sesión; el arqueo cuenta cada anulación en la caja de la que salió el efectivo; la FK `Sesion_Anulacion_ID` hacia `POS_Sesiones_Caja` rechaza una sesión inexistente. |
| `Ventas/PosLookupsIntegrationTests.cs` | MediatR (queries), base real | 9 | Productos, lotes, stock y presentaciones sembrados | `GetLotesDisponibles`: solo lotes vendibles de la propia sucursal ordenados por vencimiento, incluye la condición de venta, busca por nombre cuando no hay id y falla sin producto o sin sucursal en el token; `GetConfiguracion`: flexible por defecto y estricta cuando se activa la bandera de la empresa en la base, estricta si la sucursal no tiene empresa, falla sin sucursal en el token; `GetPresentaciones` devuelve solo las del producto. |
| `Dashboard/DashboardCajaDevolucionesIntegrationTests.cs` | MediatR (queries), base real | 8 | Empresa propia; sesiones de caja abiertas, ventas y devoluciones parciales y totales | Tarjeta de caja: mi sesión con el efectivo esperado del arqueo; el administrador sin empleado ve las abiertas de todas las sucursales; un no administrador sin sesión ve solo las de su sucursal; sin sucursal ni empleado devuelve vacío. Devoluciones: cuenta las del día de la propia sucursal manteniendo las ventas brutas, las de todas las sucursales para el administrador, ceros sin datos, y falla la validación si `Hoy` no está cerca del día actual. |
| `Dashboard/DashboardInventarioIntegrationTests.cs` | MediatR (query), base real | 6 | Empresa propia; lotes con stock que caducan alrededor de cada frontera | Reparte los lotes por día de caducidad; cuenta bajo mínimo y agotados sobre el stock vendible de una sucursal; cuenta cada par sucursal-producto para el administrador; ceros sin datos o sin sucursal; falla la validación si `Hoy` no es válido. |
| `Dashboard/DashboardVentasIntegrationTests.cs` | MediatR (query), base real | 8 | Empresa propia; ventas cobradas, anuladas y estacionadas con pagos mixtos, alrededor de la medianoche de Lima | Solo cuenta las ventas cobradas, netas del vuelto; ubica cada venta en su día local (medianoche y después de las 19 h de Lima); desglosa la empresa por sucursal para el administrador; un no administrador ve solo su sucursal y sus propias ventas; top 5 de productos por unidades base en los últimos 7 días; ceros y sin cifras personales sin datos ni empleado; ceros sin sucursal en alcance; falla la validación si `Hoy` no es válido. |

### 4.6 Infraestructura sin contenedor

| Archivo | Tipo | Pruebas | Qué verifica |
|---|---|---|---|
| `Email/EmailInfrastructureTests.cs` | Unidad de infraestructura | 11 | `EmailOptions.EnsureValid`: proveedor deshabilitado solo si se permite, configuración SMTP completa, configuración Azure Communication completa, lanza si el proveedor elegido está incompleto o es desconocido; `AppOptions.EnsureValid` acepta la URL del SPA y lanza si falta, es relativa o no es https fuera de desarrollo; el enlace de restablecimiento pone token y usuario en el fragmento; `SmtpEmailSender` arma el mensaje MIME (remitente, destinatario, asunto, ambos cuerpos), sigue SSL y puerto en las opciones de socket y devuelve fallo en vez de lanzar si el servidor es inalcanzable; `AzureCommunicationEmailSender` arma el mensaje con las mismas partes. |
| `Excel/ExcelReaderServiceTests.cs` | Unidad del lector Excel | 8 | `ReadRows` devuelve las filas de un libro válido; acepta el tope exacto de filas; rechaza el exceso de filas, una extensión que no es `.xlsx`, contenido que no es un paquete ZIP, un ZIP que no es libro, una entrada con razón de compresión excesiva y un tamaño total descomprimido sobre el límite. |
| `Recetas/GeminiRecetaAnalyzerTests.cs` | Unidad con `HttpMessageHandler` simulado | 2 | `GeminiRecetaAnalyzer` usa el modelo OCR para el OCR y el de razonamiento para el análisis; si el de razonamiento no está definido (nulo o vacío) usa el de OCR. No llama a Gemini ni a Presidio reales. |

---

## 5. Cobertura por módulo

### 5.1 Método

Script de solo lectura (`09a-cov.js`, en el scratchpad de la sesión; no se versiona) que cruza los inventarios con el texto de las pruebas. No mide cobertura de líneas (no se ejecutó `coverlet`): mide **si existe una prueba que nombra la clase**.

1. **Universo**: `sofia-docs/_inventario/commands-queries.csv` (113 commands y 90 queries), `validadores.csv` (83 validadores) y `entidades.csv` (50 entidades). El módulo de un command, query o validador es la carpeta de primer nivel bajo `SOFIA.Application` (de la ruta del CSV). Las entidades no tienen módulo en el inventario; se asignaron a mano por afinidad (por ejemplo `VentaPago` a Ventas, `RefreshToken` a Security, `SistemaOutboxEvento` a «Transversal»).
2. **Command o query «con prueba»**: la clase del handler (resuelta leyendo en `SOFIA.Application` la clase que implementa `…Handler<TRequest…>`; los 203 se resolvieron) aparece por nombre en algún archivo de `SOFIA.UnitTests`.
3. **Validador «con prueba»**: la clase del validador (nombre real de la clase leído del archivo, que a veces difiere del archivo: `CreateLoteInventarioValidator` vive en `CreateLoteInventarioCommandValidator.cs`) aparece por nombre en algún archivo de `SOFIA.UnitTests`. Los validadores que corren dentro del pipeline en una prueba de integración (por ejemplo el de `RegistrarEmpresa`) no cuentan aquí si ninguna prueba unitaria los instancia.
4. **Entidad «con prueba»**: existe un archivo en una carpeta `Domain/` de `SOFIA.UnitTests` cuyo nombre empieza por el de la entidad (sin distinguir mayúsculas). Es una medida estricta: entidades hijas como `DetalleVenta` se ejercitan dentro de pruebas de su agregado (`VentaTests`) pero no cuentan como «con prueba dedicada».
5. **Integración**: columna «Integración (archivos / invocados)»: número de archivos de `SOFIA.IntegrationTests` que mencionan por nombre algún command o query del módulo, y cuántos commands + queries distintos del módulo se mencionan. Una mención en un archivo que solo siembra datos con ese command cuenta igual (por ejemplo `CreateVentaCommand` en `PreciosProveedorIntegrationTests.cs`).
6. Los totales por columna se obtienen sumando las filas; la última columna es el número de pruebas unitarias del módulo según el CSV (la carpeta de pruebas coincide con la de Application salvo `Common`, que no tiene módulo de Application, y `Seguros`/`Transversal`, que no tienen carpeta de pruebas).

### 5.2 Tabla

Formato `con prueba / total`.

| Módulo | Commands | Validadores | Entidades (prueba de dominio) | Handlers de consulta | Integración (archivos / invocados: commands+queries) | Pruebas unitarias del módulo |
|---|---|---|---|---|---|---|
| Auditoria | 1/1 | 0/1 | 0/0 | 0/0 | 0 / 0+0 | 1 |
| Dashboard | 0/0 | 3/3 | 0/0 | 0/5 | 4 / 0+5 | 21 |
| Delivery | 1/4 | 0/2 | 1/1 | 0/2 | 0 / 0+0 | 10 |
| Devoluciones | 1/1 | 1/2 | 1/2 | 0/2 | 0 / 0+0 | 28 |
| DIGEMID | 5/6 | 2/4 | 1/4 | 0/5 | 0 / 0+0 | 23 |
| Empleados | 3/3 | 2/3 | 1/1 | 0/2 | 2 / 2+2 | 61 |
| Empresas | 1/2 | 1/2 | 1/1 | 0/2 | 3 / 1+0 | 57 |
| FormulacionesClinicas | 3/3 | 2/2 | 1/1 | 0/3 | 1 / 1+1 | 40 |
| IngredientesActivos | 4/4 | 1/3 | 0/1 | 0/3 | 1 / 1+0 | 11 |
| Inventarios | 3/5 | 3/4 | 2/2 | 0/6 | 0 / 0+0 | 57 |
| JerarquiasUoM | 3/4 | 1/3 | 0/1 | 0/4 | 0 / 0+0 | 26 |
| Laboratorios | 4/4 | 1/3 | 1/1 | 0/3 | 1 / 1+0 | 17 |
| Magistrales | 2/2 | 2/3 | 0/2 | 0/2 | 0 / 0+0 | 26 |
| Medicamentos | 4/4 | 3/3 | 1/1 | 1/3 | 3 / 2+1 | 76 |
| Pacientes | 4/4 | 3/3 | 1/1 | 0/3 | 3 / 2+1 | 29 |
| POS | 2/2 | 1/3 | 1/1 | 0/2 | 2 / 1+0 | 39 |
| PresentacionesVenta | 3/3 | 2/2 | 1/1 | 0/2 | 0 / 0+0 | 34 |
| Profesionales | 4/4 | 1/2 | 0/1 | 0/3 | 1 / 1+0 | 11 |
| Proveedores | 5/5 | 2/3 | 1/2 | 0/4 | 2 / 2+1 | 30 |
| Recetas | 3/3 | 3/4 | 1/3 | 0/3 | 0 / 0+0 | 31 |
| Security | 16/20 | 2/10 | 2/8 | 0/8 | 3 / 6+2 | 103 |
| Seguros | 0/4 | 0/1 | 0/1 | 0/3 | 0 / 0+0 | — |
| SeriesFiscales | 3/3 | 2/2 | 2/2 | 0/2 | 0 / 0+0 | 44 |
| Servicios | 2/6 | 1/3 | 0/2 | 0/4 | 0 / 0+0 | 18 |
| Sucursales | 4/4 | 1/3 | 1/1 | 0/4 | 2 / 1+2 | 27 |
| Transferencias | 1/4 | 0/2 | 1/2 | 0/2 | 0 / 0+0 | 29 |
| Transversal (entidades de infraestructura) | 0/0 | 0/0 | 0/2 | 0/0 | 0 / 0+0 | — |
| UnidadesMedida | 4/4 | 1/3 | 0/1 | 0/3 | 0 / 0+0 | 12 |
| Ventas | 4/4 | 4/4 | 2/4 | 0/5 | 3 / 2+3 | 180 |
| `Common` (sin módulo de Application) | — | — | — | — | — | 38 |
| **Total** | **90/113** | **45/83** | **23/50** | **1/90** | — | **1 079** |

Lectura rápida: los commands están bien cubiertos por pruebas de handler (80 %); los validadores un poco más de la mitad (54 %); las entidades con prueba de dominio dedicada, menos de la mitad (46 %, aunque varias de las restantes son líneas hijas que se prueban dentro de su agregado); los handlers de consulta casi nada, por convención (1.7). Catorce módulos no tienen ninguna prueba de integración que nombre sus commands o queries: Auditoria, Delivery, Devoluciones, DIGEMID, Inventarios, JerarquiasUoM, Magistrales, PresentacionesVenta, Recetas, Seguros, SeriesFiscales, Servicios, Transferencias y UnidadesMedida (ver 6).

Notas sobre la tabla:

- El módulo Ventas figura con «4/4 commands» (`ActualizarVentaPendiente`, `AnularVenta`, `CompletarVenta`, `CreateVenta`); el módulo POS con 2/2 (`AperturarCaja`, `CerrarCaja`). `ProcesarDevolucion` es el único command de Devoluciones.
- «Security» agrupa autenticación, cuentas y roles; sus 8 entidades incluyen `Rol`, `PermisoRol`, `RolSucursal`, `CuentaRol`, `CuentaSucursal`, `AuditoriaEventoSeguridad`, `Cuenta` y `RefreshToken`.
- La columna de integración cuenta archivos que nombran la clase; no garantiza que el flujo del módulo se pruebe de punta a punta. Un ejemplo es Ventas, donde `CreateVentaCommand` solo se usa para sembrar una venta en un archivo de proveedores (4.4).

### 5.3 Consultas ejercidas fuera de las pruebas unitarias

De las 90 queries, 1 tiene prueba de handler unitaria (`PreviewImportMedicamentosQuery`) y 18 se ejercitan en integración: `GetDashboardCajaQuery`, `GetDashboardCuarentenaQuery`, `GetDashboardDevolucionesQuery`, `GetDashboardInventarioQuery`, `GetDashboardVentasQuery`, `GetEmpleadoByIdQuery`, `GetEmpleadosWithPaginationQuery`, `GetFormulacionesClinicasWithPaginationQuery`, `GetMedicamentosQuery`, `GetPacientesQuery`, `GetPreciosProveedorQuery`, `GetCuentaByIdQuery`, `GetCuentasQuery`, `GetSucursalByIdQuery`, `GetSucursalesWithPaginationQuery`, `GetConfiguracionVentaQuery`, `GetLotesDisponiblesVentaQuery`, `GetPresentacionesProductoVentaQuery`. Las otras **71** queries no tienen ninguna prueba que las nombre (ni de handler ni de integración); además, 6 de las 18 (las de empleados, cuentas y sucursales) aparecen en una sola prueba que verifica el nombre sin hueco de un empleado sin apellido materno.

---

## 6. Huecos

Todo lo que sigue sale del cruce del apartado 5 y de las pruebas catalogadas en 3 y 4. «Sin prueba» significa «ninguna prueba nombra la clase»; puede haber cobertura indirecta que el método no ve (por ejemplo, una entidad ejercitada dentro de otra). Los IDs `RN-…` son los de [01-reglas-de-negocio](../01-reglas-de-negocio/).

### 6.1 Commands sin prueba de handler (23 de 113)

| Módulo | Commands |
|---|---|
| DIGEMID | `DeleteDigemidProductoCommand` |
| Delivery | `DeleteDespachoDeliveryCommand`, `UpdateDeliveryEstadoCommand`, `UpdateDespachoDeliveryCommand` (la transición de estado sí se prueba en la entidad `DespachoDelivery`) |
| Empresas | `UpdateEmpresaCommand` |
| Inventarios | `CreateLoteInventarioCommand`, `UpdateLoteInventarioCommand` (sus validadores sí tienen prueba) |
| JerarquiasUoM | `DeleteJerarquiaUoMCommand` |
| Security | `AssignSucursalToCuentaCommand`, `RemoveSucursalFromCuentaCommand`, `CreateRolCommand`, `UpdateRolCommand` |
| Seguros | `CargaMasivaSegurosCommand`, `CreateAseguradoraCommand`, `DeleteAseguradoraCommand`, `UpdateAseguradoraCommand` |
| Servicios | `DeleteInmunizacionCommand`, `DeleteServicioCommand`, `UpdateInmunizacionCommand`, `UpdateServicioCommand` |
| Transferencias | `CancelarTransferenciaCommand`, `DespacharTransferenciaCommand`, `RecibirTransferenciaCommand` (la máquina de estados sí se prueba en la entidad `Transferencia`, 23 pruebas, pero no el handler: stock, permisos de sucursal, receptor) |

### 6.2 Validadores sin prueba (38 de 83)

| Módulo | Validadores |
|---|---|
| Auditoria | `AnonimizarDatosCommandValidator` |
| DIGEMID | `GenerarActaDestruccionCommandValidator`, `GetCuarentenaQueryValidator` |
| Delivery | `ProgramarDeliveryCommandValidator`, `UpdateDeliveryEstadoCommandValidator` |
| Devoluciones | `GetDevolucionesQueryValidator` |
| Empleados | `GetEmpleadosWithPaginationQueryValidator` |
| Empresas | `UpdateEmpresaCommandValidator` |
| IngredientesActivos | `CreateIngredienteActivoCommandValidator`, `UpdateIngredienteActivoCommandValidator` |
| Inventarios | `RegisterInventarioValidator` |
| JerarquiasUoM | `CreateJerarquiaUoMCommandValidator`, `UpdateJerarquiaUoMCommandValidator` |
| Laboratorios | `CreateLaboratorioCommandValidator`, `UpdateLaboratorioCommandValidator` |
| Magistrales | `GetOrdenesQueryValidator` |
| POS | `CerrarCajaCommandValidator`, `GetSesionesQueryValidator` |
| Profesionales | `CreateProfesionalSaludCommandValidator` |
| Proveedores | `CreateProveedorCommandValidator` |
| Recetas | `GetRecetasQueryValidator` |
| Security | `UpdateCuentaCommandValidator`, `ForgotPasswordCommandValidator`, `LoginCommandValidator`, `RefreshTokenCommandValidator`, `RegisterAccountCommandValidator`, `ResetPasswordCommandValidator`, `CreateRolCommandValidator`, `UpdateRolCommandValidator` |
| Seguros | `CreateAseguradoraCommandValidator` |
| Servicios | `AgendarServicioCommandValidator`, `GetInmunizacionesQueryValidator` |
| Sucursales | `CreateSucursalCommandValidator`, `UpdateSucursalCommandValidator` |
| Transferencias | `CreateTransferenciaCommandValidator`, `RecibirTransferenciaCommandValidator` |
| UnidadesMedida | `CreateUnidadMedidaCommandValidator`, `UpdateUnidadMedidaCommandValidator` |

Varios de estos validadores sí corren en el pipeline de una prueba de integración (por ejemplo `RegistrarEmpresaValidationIntegrationTests`), pero ninguna prueba instancia el validador y recorre sus reglas.

### 6.3 Entidades sin prueba de dominio dedicada (27 de 50)

`AseguradoraMedica`, `AuditoriaEventoSeguridad`, `CuentaRol`, `CuentaSucursal`, `DetalleTransferencia`, `DetalleVenta`, `DevolucionDetalle`, `DigemidActaDestruccion`, `DigemidActaDetalle`, `DigemidCatalogoProducto`, `IngredienteActivo`, `JerarquiaUoM`, `MagistralConsumoInsumo`, `MagistralOrdenProduccion`, `PermisoRol`, `ProfesionalSalud`, `ProveedorDistribuidor`, `RecetaDigitalizadaIA`, `RegistroPrivacidadPresidio`, `Rol`, `RolSucursal`, `ServicioAgenda`, `ServicioClinicoInmunizacion`, `SistemaNotificacionInterna`, `SistemaOutboxEvento`, `UnidadMedida`, `VentaReclamoSeguro`.

De ellas, el script no encontró ninguna mención en pruebas unitarias ni de integración de `CuentaRol`, `CuentaSucursal`, `DigemidActaDetalle`, `RecetaDigitalizadaIA`, `RegistroPrivacidadPresidio` ni `SistemaNotificacionInterna`. `MagistralOrdenProduccion` y `MagistralConsumoInsumo` aparecen en las pruebas de los handlers de Magistrales (3.14) pero no tienen prueba propia.

### 6.4 Handlers de consulta

71 de las 90 queries no tienen ninguna prueba (5.3). Entre las que importan al negocio: `GetVentasQuery` y `GetVentaByIdQuery` (listado y detalle de ventas, RN-VEN-070 a RN-VEN-074), `GetSesionesQuery` y `GetSesionByIdQuery` (RN-CAJ-020 a RN-CAJ-023), `GetDevolucionesQuery` y `GetDevolucionByIdQuery` (RN-DEV-030, RN-DEV-031), `GetStockPorSucursalQuery`, `GetTransferenciasQuery`, `GetCuarentenaQuery`, `GetProfileQuery`, `GetPermissionsByRolQuery`, `AnalizarRecetaQuery` (solo se prueba su validador y la selección de modelo) y todas las `PreviewImport…Query` salvo la de medicamentos. Las exportaciones a Excel (RN-VEN-078, RN-CAJ-024) tampoco tienen prueba salvo las de pacientes, laboratorios y empresas en `IT/Api/ExportRequestValidationIntegrationTests.cs` y `IT/Laboratorios/LaboratoriosExportIntegrationTests.cs`.

### 6.5 Flujos de negocio críticos sin prueba de integración

Estado por flujo. «Sin integración» significa que ningún archivo de `SOFIA.IntegrationTests` nombra el command (5.2), es decir, que no se prueba contra la base real ni por HTTP; la anulación de venta es el único flujo de esta tabla que sí está cubierto de punta a punta:

| Flujo | Lo que sí hay | Hueco |
|---|---|---|
| Apertura de caja (`AperturarCajaCommand`) | 8 + 2 pruebas unitarias con mocks (3.16); el índice único `UX_POS_Sesiones_Caja_Empleado_Abierta` se prueba en `Concurrency` (4.2) | Ninguna prueba de integración pasa por el handler ni por el endpoint. |
| Cierre y arqueo de caja (`CerrarCajaCommand`) | 17 pruebas unitarias del cálculo (3.16); dos pruebas de integración cierran una caja (`AuditTrail`, 4.2) y una cuenta anulaciones entre sesiones (`AnularVentaCaja`, 4.5) | No hay prueba de integración del arqueo con ventas, pagos mixtos, seguro y devoluciones reales (RN-CAJ-011 a RN-CAJ-014) ni de dos cierres simultáneos de la misma caja (punto abierto 3 de la sección de pendientes de [01c3](../01-reglas-de-negocio/01c3-caja-devoluciones-dashboard.md)); no hay prueba que nombre las ventas en espera al cerrar (RN-CAJ-017) ni que el arqueo es una foto (RN-CAJ-018). |
| Crear una venta (`CreateVentaCommand`) | 33 + 13 pruebas unitarias (3.28) | Solo se usa como siembra en `PreciosProveedorIntegrationTests` (costo histórico, RN-VEN-022). No hay prueba de integración del descuento de stock con base real (RN-VEN-014, RN-VEN-015), de cuarentena (RN-VEN-012), de la emisión de la boleta ni de la unicidad del correlativo (RN-CMP-004, RN-CMP-009). |
| Cobrar una venta en espera (`CompletarVentaCommand`) | 21 + 8 pruebas unitarias | Sin integración (RN-VEN-056). |
| Actualizar venta en espera (`ActualizarVentaPendienteCommand`) | 10 + 3 pruebas unitarias | Sin integración (RN-VEN-053 a RN-VEN-055). |
| Anulación de venta | 11 pruebas unitarias + 6 de integración + 2 de concurrencia | Cubierta (RN-VEN-062 a RN-VEN-069, RN-CMP-012); no se prueba el rechazo por HTTP ni el permiso de anular. |
| Devolución (`ProcesarDevolucionCommand`) | 14 + 8 pruebas unitarias del handler y el validador, 3 de las extensiones y 3 de la cabecera; concurrencia de devoluciones parciales a nivel de entidad (4.2) | **Sin prueba de integración del command ni del endpoint**: nota de crédito real contra la serie, reintegro de stock en base real, transaccionalidad todo o nada (RN-DEV-020 a RN-DEV-025, RN-DEV-028). Las dos consultas de devoluciones (RN-DEV-030, RN-DEV-031) no tienen prueba. |
| Recetas: exigencia al vender | Matriz de exigencia en `VentaDetalleFactoryTests` (31) y en los handlers de venta; configuración flexible o estricta en `PosLookupsIntegrationTests` | La política `ExigirRegistroReceta` (RN-VEN-028, RN-VEN-029) no se prueba con una venta real de punta a punta. |
| Recetas: análisis con IA | `GeminiRecetaAnalyzerTests` (2: selección de modelo) y validador de especialidad (3) | Sin prueba: formato de imagen (RN-REC-019), fallo del proveedor (RN-REC-021), límites del resultado (RN-REC-022), validación contra DIGEMID (RN-REC-023), coincidencia con el catálogo (RN-REC-024), stock por sucursal (RN-REC-026), equivalentes y sugerencias (RN-REC-027, RN-REC-028). Presidio (anonimización) solo como doble. |
| CRUD de recetas | Handlers unitarios (Create/Update/Delete) y un 404 por HTTP | Sin integración. |
| Transferencias | Entidad (23) y handler de creación (6) | Sin pruebas de despachar, recibir y cancelar a nivel de handler ni de integración; el stock no se mueve en ninguna prueba de integración. |
| Inventario (lotes y stock) | Entidades y validadores; `AdjustStock`, `DeleteLote`, `RegisterInventario` por handler; concurrencia de stock | Sin handlers de `CreateLote` y `UpdateLote`; sin integración de ajuste de stock. |
| DIGEMID y cuarentena | `AislarLoteCuarentena` y actas por handler (mocks) | Sin integración; `GetCuarentenaQuery` y su validador sin prueba. |
| Magistrales | Handlers de iniciar y completar por mocks | Sin integración; entidades sin prueba propia. |
| Servicios e inmunizaciones | Agendar (4) y registrar inmunización (12 + 2) | Sin `Update`/`Delete` de servicio ni de inmunización; sin integración. |
| Seguros (aseguradoras) | Solo el procesamiento de la cobertura dentro de una venta (`VentaSeguroProcessorTests`, 3.28) | El módulo Seguros (maestro, carga masiva, 4 commands, 3 queries) no tiene ninguna prueba. |
| Series fiscales | Handlers y entidades por mocks | Ninguna prueba de integración del consumo concurrente del correlativo ni del índice `UX_SUNAT_Comprobantes_Emitidos_Serie_Correlativo` (RN-CMP-009, RN-SER-013 numeración atómica). |

### 6.6 Seguridad: reglas sin prueba o con prueba parcial

| Regla | Estado |
|---|---|
| Autorización por permiso en los endpoints (RN-GEN-015, RN-GEN-020, RN-GEN-021) | La prueba de arquitectura solo verifica que cada acción declare `[HasPermission]`, `[Authorize]` o `[AllowAnonymous]` (2); no hay prueba que llame a un endpoint con un usuario sin el permiso y espere 403 (los únicos 403 que se prueban por HTTP son el de CSRF, `CsrfHeaderIntegrationTests`, y el del cambio forzado de contraseña o empresa suspendida, `AuthIntegrationTests`). Solo `PermissionHandler` se prueba por la caché entre empresas (4.3). |
| Límites de peticiones (RN-GEN-025 a RN-GEN-029) | `RateLimitingIntegrationTests` cubre solo el límite de credenciales (`/api/v1/auth/login` y `/auth/refresh`) y la partición por empresa. No hay prueba del límite general, del de registro de empresas ni del de endpoints de IA. |
| Creación y edición de roles (`CreateRol`, `UpdateRol`) y asignación de sucursales a cuentas (RN-CTA-019) | Sin handler ni validador probados. Sí están: asignar permisos, asignar y quitar rol a usuario (RN-ROL-010, RN-CTA-015, RN-CTA-016), sucursales del rol, borrar rol y revocar permiso. |
| Validadores de autenticación (`Login`, `RefreshToken`, `RegisterAccount`, `ForgotPassword`, `ResetPassword`) | Sin prueba propia; el comportamiento sí se prueba a nivel de handler. |
| Actualizar empresa (`UpdateEmpresaCommand`) | Sin prueba de handler ni de validador; solo se verifica que el detalle de su auditoría no contiene RUC ni nombre (`AuditBehaviorTests`). |
| Cubiertas | Bloqueo por intentos (RN-AUT-007), tiempo de respuesta uniforme (RN-AUT-004), rotación y detección de robo de sesión (RN-AUT-016, RN-AUT-018), cierre de sesión idempotente (RN-AUT-020), recuperación de contraseña (RN-AUT-035 a RN-AUT-039), empresa vigente y cambio forzado de clave (RN-GEN-018, RN-GEN-019), aislamiento por empresa (RN-GEN-001, RN-GEN-004), transaccionalidad (RN-GEN-012), auditoría (RN-GEN-013), cabeceras de seguridad (RN-GEN-030), configuración mínima del token (RN-GEN-033), errores globales (RN-GEN-035 a RN-GEN-040) y cuentas de administrador protegidas (RN-CTA-010). |

### 6.7 Huecos de la propia estrategia

- **Esquema de integración distinto del real**: las pruebas usan `EnsureCreated` (1.5). Los scripts DbUp (fuente de verdad del esquema) no se ejercitan en ninguna prueba; un índice o restricción que exista solo en un script no se prueba.
- **Integración fuera de CI**: ver 7.
- **Suites que dependen de reflexión** sobre campos de respaldo (`CuentaFactory`, `EmisorMocks`) se romperían silenciosamente con un renombre.
- **Sin pruebas de rendimiento, carga ni de interfaz** del lado del backend; la seguridad estática depende de SonarAnalyzer y OWASP Dependency Check en CI (7).

---

## 7. Cómo se corren y dónde

Los comandos, los requisitos de Docker y la clave del contenedor de pruebas están en la [guía del desarrollador, sección 4.1](../08-guia-del-desarrollador.md#41-backend); no se repiten. Resumen:

| Proyecto | Comando | Necesita |
|---|---|---|
| `SOFIA.UnitTests` | `dotnet test src/SOFIA.UnitTests/SOFIA.UnitTests.csproj` | Nada externo |
| `SOFIA.ArchitectureTests` | `dotnet test src/SOFIA.ArchitectureTests/SOFIA.ArchitectureTests.csproj` | Nada externo |
| `SOFIA.IntegrationTests` | `dotnet test src/SOFIA.IntegrationTests/SOFIA.IntegrationTests.csproj` | Docker, contraseña del contenedor (`SOFIA_TEST_DB_PASSWORD` o user-secret `Testing:ContainerDbPassword`) y la configuración JWT de la API, según la guía |

Lo que la guía no dice y se observa en el código: (a) las pruebas de integración que no heredan de `BaseIntegrationTest` (6, ver 1.5) no necesitan Docker, pero el proyecto en conjunto sí lo exige para las otras 26; (b) cada clase de integración arranca su propio contenedor SQL Server (`IClassFixture`), así que el tiempo crece con el número de clases (26 contenedores); (c) las colecciones no corren en paralelo (`xunit.runner.json`).

**Dónde se ejecutan automáticamente**

| Punto | Qué ejecuta | Fuente |
|---|---|---|
| Hook `pre-commit` (Husky.Net) | `detect-secrets`, `dotnet format` sobre los archivos en staging, `git add` de los formateados y **solo las pruebas de arquitectura** (`dotnet test` en `Release` con `--no-restore`) | `sofia-apis/.husky/task-runner.json:35-46`, `sofia-apis/.husky/pre-commit` |
| Workflow `Security Scan` (job `build-analysis`) | `dotnet build` en `Release` de la solución y, en este orden, las pruebas **unitarias** (con resultado `.trx`) y las **de arquitectura**. Se dispara en `push` y `pull_request` a `master`/`main` y los lunes a las 08:00 UTC | `sofia-apis/.github/workflows/security.yml:3-10,20-54` |
| Integración | **No se ejecuta en ningún hook ni en CI**: el workflow no tiene paso para `SOFIA.IntegrationTests`. Solo se corren a mano | `security.yml` (ningún paso la menciona); `task-runner.json` |

Los otros jobs del workflow (OWASP Dependency Check y gitleaks) no ejecutan pruebas. El SPA no tiene CI (ver [09b-spa.md](09b-spa.md)).

---

## 8. Pendientes y «No verificado»

**No verificado**

1. **No se ejecutó ninguna prueba ni compilación** (prohibido en este encargo). No se sabe si hoy las 1 254 pruebas pasan; el documento describe lo que el código dice que verifican.
2. Las cifras de pruebas son **métodos con `[Fact]`/`[Theory]`**, no casos: las 357 filas `[InlineData]` ejecutan más casos de los que cuentan.
3. La cobertura del apartado 5 es **por nombre de clase**, no de líneas ni de ramas; no se ejecutó `coverlet`. Una clase cubierta por nombre puede tener ramas sin probar, y una clase «sin prueba» puede ejercitarse indirectamente.
4. Los recuentos de subgrupos entre paréntesis de 3 (por ejemplo «(5)») se obtuvieron leyendo los nombres de los métodos, no ejecutando; los totales por archivo sí coinciden con el CSV.
5. El contenido interno de cada prueba de integración se resumió desde sus nombres, comentarios y cabeceras; no se leyó línea por línea el cuerpo de cada una (se leyeron completas la infraestructura, `ConcurrencyIntegrationTests`, `GeminiRecetaAnalyzerTests`, `LoggingConfigurationTests`, `TransactionRollbackIntegrationTests` y las cabeceras de los demás).
6. No se comprobó si el contenedor de CI (`ubuntu-latest`) podría ejecutar las pruebas de integración ni si faltaría configuración (JWT, contraseña).
7. Las reglas de negocio citadas en 6 se tomaron de los títulos de los documentos [01-reglas-de-negocio](../01-reglas-de-negocio/) (validados por otro agente) y de la lista de IDs vista; no se contrastó cada regla contra el código de la prueba. Los IDs RN-SER se tomaron de la lista de títulos de las reglas de series fiscales (RN-SER-013).

**En curso (sin commitear):** ninguno. `git status --short` en `sofia-apis` no mostró cambios al 2026-10-15; el commit sigue siendo `43f5529`.

**Pendientes**

- Evaluar añadir pruebas de integración a CI (Docker en el runner) o al menos un hook previo al push.
- Cerrar los huecos de 6 empezando por los flujos que mueven dinero o stock sin integración (devolución, cobro de venta pendiente, cierre de caja, transferencias).
- Reconciliar con [08-guia-del-desarrollador.md](../08-guia-del-desarrollador.md): sus citas a `SofiaWebAppFactory.cs` líneas 19-25 y 33-90 corresponden a una versión anterior del archivo (ahora la fábrica está en las líneas 54-127); el contenido sigue siendo correcto. Y el `README.md` de `_inventario` dice «186 archivos» de prueba mientras el CSV regenerado tiene 193.

---

## 9. Tabla de cobertura

### 9.1 Archivos de `pruebas-backend.csv`

Los 193 archivos del inventario, con el apartado donde están documentados (el apartado 3.N es la sección del módulo; el 4.N, el área de integración).

| Archivo (`pruebas-backend.csv`) | Pruebas | Apartado |
|---|---|---|
| `SOFIA.UnitTests/Auditoria/Commands/AnonimizarDatosCommandHandlerTests.cs` | 1 | 3.1 Auditoria |
| `SOFIA.UnitTests/Common/Behaviors/AuditBehaviorTests.cs` | 12 | 3.2 Common |
| `SOFIA.UnitTests/Common/Excel/CargaMasivaCommandValidatorsTests.cs` | 5 | 3.2 Common |
| `SOFIA.UnitTests/Common/Excel/ImportNameLookupTests.cs` | 3 | 3.2 Common |
| `SOFIA.UnitTests/Common/Extensions/QueryableExtensionsTests.cs` | 5 | 3.2 Common |
| `SOFIA.UnitTests/Common/Models/PaginatedListTests.cs` | 5 | 3.2 Common |
| `SOFIA.UnitTests/Common/Services/SucursalAccessTests.cs` | 8 | 3.2 Common |
| `SOFIA.UnitTests/DIGEMID/Commands/AislarLoteCuarentena/AislarLoteCuarentenaCommandHandlerTests.cs` | 4 | 3.3 DIGEMID |
| `SOFIA.UnitTests/DIGEMID/Commands/AislarLoteCuarentena/AislarLoteCuarentenaCommandValidatorTests.cs` | 7 | 3.3 DIGEMID |
| `SOFIA.UnitTests/DIGEMID/Commands/CargaMasivaDigemid/CargaMasivaDigemidCommandHandlerTests.cs` | 2 | 3.3 DIGEMID |
| `SOFIA.UnitTests/DIGEMID/Commands/CreateDigemidProducto/CreateDigemidProductoCommandHandlerTests.cs` | 2 | 3.3 DIGEMID |
| `SOFIA.UnitTests/DIGEMID/Commands/GenerarActaDestruccion/GenerarActaDestruccionCommandHandlerTests.cs` | 2 | 3.3 DIGEMID |
| `SOFIA.UnitTests/DIGEMID/Commands/UpdateDigemidProducto/UpdateDigemidProductoCommandHandlerTests.cs` | 3 | 3.3 DIGEMID |
| `SOFIA.UnitTests/DIGEMID/Domain/DigemidInventarioCuarentenaTests.cs` | 3 | 3.3 DIGEMID |
| `SOFIA.UnitTests/Dashboard/Common/DashboardDiasTests.cs` | 7 | 3.4 Dashboard |
| `SOFIA.UnitTests/Dashboard/Common/DashboardScopeTests.cs` | 4 | 3.4 Dashboard |
| `SOFIA.UnitTests/Dashboard/Common/PagosPorMetodoCalculatorTests.cs` | 7 | 3.4 Dashboard |
| `SOFIA.UnitTests/Dashboard/Queries/DashboardHoyValidatorTests.cs` | 3 | 3.4 Dashboard |
| `SOFIA.UnitTests/Delivery/Commands/ProgramarDelivery/ProgramarDeliveryCommandHandlerTests.cs` | 6 | 3.5 Delivery |
| `SOFIA.UnitTests/Delivery/Domain/DespachoDeliveryTests.cs` | 4 | 3.5 Delivery |
| `SOFIA.UnitTests/Devoluciones/Commands/ProcesarDevolucion/ProcesarDevolucionCommandHandlerTests.cs` | 14 | 3.6 Devoluciones |
| `SOFIA.UnitTests/Devoluciones/Commands/ProcesarDevolucion/ProcesarDevolucionCommandValidatorTests.cs` | 8 | 3.6 Devoluciones |
| `SOFIA.UnitTests/Devoluciones/Common/CantidadesDevueltasExtensionsTests.cs` | 3 | 3.6 Devoluciones |
| `SOFIA.UnitTests/Devoluciones/Domain/DevolucionCabeceraTests.cs` | 3 | 3.6 Devoluciones |
| `SOFIA.UnitTests/Empleados/Commands/CreateEmpleado/CreateEmpleadoCommandHandlerTests.cs` | 8 | 3.7 Empleados |
| `SOFIA.UnitTests/Empleados/Commands/CreateEmpleado/CreateEmpleadoCommandValidatorTests.cs` | 11 | 3.7 Empleados |
| `SOFIA.UnitTests/Empleados/Commands/DeleteEmpleado/DeleteEmpleadoCommandHandlerTests.cs` | 3 | 3.7 Empleados |
| `SOFIA.UnitTests/Empleados/Commands/UpdateEmpleado/UpdateEmpleadoCommandHandlerTests.cs` | 13 | 3.7 Empleados |
| `SOFIA.UnitTests/Empleados/Commands/UpdateEmpleado/UpdateEmpleadoCommandValidatorTests.cs` | 9 | 3.7 Empleados |
| `SOFIA.UnitTests/Empleados/Domain/EmpleadoTests.cs` | 17 | 3.7 Empleados |
| `SOFIA.UnitTests/Empresas/Commands/RegistrarEmpresa/RegistrarEmpresaCommandHandlerTests.cs` | 17 | 3.8 Empresas |
| `SOFIA.UnitTests/Empresas/Commands/RegistrarEmpresa/RegistrarEmpresaCommandValidatorTests.cs` | 18 | 3.8 Empresas |
| `SOFIA.UnitTests/Empresas/Domain/EmpresaTests.cs` | 22 | 3.8 Empresas |
| `SOFIA.UnitTests/FormulacionesClinicas/Commands/CreateFormulacionClinica/CreateFormulacionClinicaCommandHandlerTests.cs` | 7 | 3.9 FormulacionesClinicas |
| `SOFIA.UnitTests/FormulacionesClinicas/Commands/CreateFormulacionClinica/CreateFormulacionClinicaCommandValidatorTests.cs` | 7 | 3.9 FormulacionesClinicas |
| `SOFIA.UnitTests/FormulacionesClinicas/Commands/DeleteFormulacionClinica/DeleteFormulacionClinicaCommandHandlerTests.cs` | 2 | 3.9 FormulacionesClinicas |
| `SOFIA.UnitTests/FormulacionesClinicas/Commands/UpdateFormulacionClinica/UpdateFormulacionClinicaCommandHandlerTests.cs` | 8 | 3.9 FormulacionesClinicas |
| `SOFIA.UnitTests/FormulacionesClinicas/Commands/UpdateFormulacionClinica/UpdateFormulacionClinicaCommandValidatorTests.cs` | 6 | 3.9 FormulacionesClinicas |
| `SOFIA.UnitTests/FormulacionesClinicas/Domain/FormulacionClinicaTests.cs` | 10 | 3.9 FormulacionesClinicas |
| `SOFIA.UnitTests/IngredientesActivos/Commands/CargaMasivaIngredientesActivos/CargaMasivaIngredientesActivosCommandHandlerTests.cs` | 2 | 3.10 IngredientesActivos |
| `SOFIA.UnitTests/IngredientesActivos/Commands/CreateIngredienteActivo/CreateIngredienteActivoCommandHandlerTests.cs` | 2 | 3.10 IngredientesActivos |
| `SOFIA.UnitTests/IngredientesActivos/Commands/DeleteIngredienteActivo/DeleteIngredienteActivoCommandHandlerTests.cs` | 4 | 3.10 IngredientesActivos |
| `SOFIA.UnitTests/IngredientesActivos/Commands/UpdateIngredienteActivo/UpdateIngredienteActivoCommandHandlerTests.cs` | 3 | 3.10 IngredientesActivos |
| `SOFIA.UnitTests/Inventarios/Commands/AdjustStock/AdjustStockCommandHandlerTests.cs` | 4 | 3.11 Inventarios |
| `SOFIA.UnitTests/Inventarios/Commands/AdjustStock/AdjustStockCommandValidatorTests.cs` | 4 | 3.11 Inventarios |
| `SOFIA.UnitTests/Inventarios/Commands/CreateLote/CreateLoteInventarioValidatorTests.cs` | 9 | 3.11 Inventarios |
| `SOFIA.UnitTests/Inventarios/Commands/DeleteLote/DeleteLoteInventarioCommandHandlerTests.cs` | 4 | 3.11 Inventarios |
| `SOFIA.UnitTests/Inventarios/Commands/RegisterInventario/RegisterInventarioCommandHandlerTests.cs` | 4 | 3.11 Inventarios |
| `SOFIA.UnitTests/Inventarios/Commands/UpdateLote/UpdateLoteInventarioValidatorTests.cs` | 8 | 3.11 Inventarios |
| `SOFIA.UnitTests/Inventarios/Domain/InventarioSucursalTests.cs` | 4 | 3.11 Inventarios |
| `SOFIA.UnitTests/Inventarios/Domain/LoteInventarioTests.cs` | 20 | 3.11 Inventarios |
| `SOFIA.UnitTests/JerarquiasUoM/Commands/CargaMasivaJerarquiasUoM/CargaMasivaJerarquiasUoMCommandHandlerTests.cs` | 5 | 3.12 JerarquiasUoM |
| `SOFIA.UnitTests/JerarquiasUoM/Commands/CreateJerarquiaUoM/CreateJerarquiaUoMCommandHandlerTests.cs` | 7 | 3.12 JerarquiasUoM |
| `SOFIA.UnitTests/JerarquiasUoM/Commands/UpdateJerarquiaUoM/UpdateJerarquiaUoMCommandHandlerTests.cs` | 8 | 3.12 JerarquiasUoM |
| `SOFIA.UnitTests/JerarquiasUoM/Common/JerarquiaConversionResolverTests.cs` | 6 | 3.12 JerarquiasUoM |
| `SOFIA.UnitTests/Laboratorios/Commands/CargaMasivaLaboratorios/CargaMasivaLaboratoriosCommandHandlerTests.cs` | 3 | 3.13 Laboratorios |
| `SOFIA.UnitTests/Laboratorios/Commands/CreateLaboratorio/CreateLaboratorioCommandHandlerTests.cs` | 5 | 3.13 Laboratorios |
| `SOFIA.UnitTests/Laboratorios/Commands/DeleteLaboratorio/DeleteLaboratorioCommandHandlerTests.cs` | 3 | 3.13 Laboratorios |
| `SOFIA.UnitTests/Laboratorios/Commands/UpdateLaboratorio/UpdateLaboratorioCommandHandlerTests.cs` | 4 | 3.13 Laboratorios |
| `SOFIA.UnitTests/Laboratorios/Domain/LaboratorioTests.cs` | 2 | 3.13 Laboratorios |
| `SOFIA.UnitTests/Magistrales/Commands/CompletarOrdenMagistral/CompletarOrdenMagistralCommandHandlerTests.cs` | 6 | 3.14 Magistrales |
| `SOFIA.UnitTests/Magistrales/Commands/CompletarOrdenMagistral/CompletarOrdenMagistralCommandValidatorTests.cs` | 5 | 3.14 Magistrales |
| `SOFIA.UnitTests/Magistrales/Commands/IniciarOrdenMagistral/IniciarOrdenMagistralCommandHandlerTests.cs` | 10 | 3.14 Magistrales |
| `SOFIA.UnitTests/Magistrales/Commands/IniciarOrdenMagistral/IniciarOrdenMagistralCommandValidatorTests.cs` | 5 | 3.14 Magistrales |
| `SOFIA.UnitTests/Medicamentos/Commands/CargaMasivaMedicamentos/CargaMasivaMedicamentosCommandHandlerTests.cs` | 8 | 3.15 Medicamentos |
| `SOFIA.UnitTests/Medicamentos/Commands/CreateMedicamento/CreateMedicamentoCommandHandlerTests.cs` | 6 | 3.15 Medicamentos |
| `SOFIA.UnitTests/Medicamentos/Commands/CreateMedicamento/CreateMedicamentoCommandValidatorTests.cs` | 9 | 3.15 Medicamentos |
| `SOFIA.UnitTests/Medicamentos/Commands/DeleteMedicamento/DeleteMedicamentoCommandHandlerTests.cs` | 11 | 3.15 Medicamentos |
| `SOFIA.UnitTests/Medicamentos/Commands/UpdateMedicamento/UpdateMedicamentoCommandHandlerTests.cs` | 8 | 3.15 Medicamentos |
| `SOFIA.UnitTests/Medicamentos/Commands/UpdateMedicamento/UpdateMedicamentoCommandValidatorTests.cs` | 8 | 3.15 Medicamentos |
| `SOFIA.UnitTests/Medicamentos/Domain/MedicamentoTests.cs` | 15 | 3.15 Medicamentos |
| `SOFIA.UnitTests/Medicamentos/Queries/PreviewImportMedicamentos/PreviewImportMedicamentosQueryHandlerTests.cs` | 11 | 3.15 Medicamentos |
| `SOFIA.UnitTests/POS/Commands/AperturarCaja/AperturarCajaCommandHandlerTests.cs` | 8 | 3.16 POS |
| `SOFIA.UnitTests/POS/Commands/AperturarCaja/AperturarCajaCommandValidatorTests.cs` | 2 | 3.16 POS |
| `SOFIA.UnitTests/POS/Commands/CerrarCaja/CerrarCajaCommandHandlerTests.cs` | 17 | 3.16 POS |
| `SOFIA.UnitTests/POS/Domain/POSSesionCajaTests.cs` | 12 | 3.16 POS |
| `SOFIA.UnitTests/Pacientes/Commands/CargaMasivaPacientes/CargaMasivaPacientesCommandHandlerTests.cs` | 2 | 3.17 Pacientes |
| `SOFIA.UnitTests/Pacientes/Commands/CreatePaciente/CreatePacienteCommandHandlerTests.cs` | 4 | 3.17 Pacientes |
| `SOFIA.UnitTests/Pacientes/Commands/CreatePaciente/CreatePacienteCommandValidatorTests.cs` | 3 | 3.17 Pacientes |
| `SOFIA.UnitTests/Pacientes/Commands/DeletePaciente/DeletePacienteCommandHandlerTests.cs` | 7 | 3.17 Pacientes |
| `SOFIA.UnitTests/Pacientes/Commands/UpdatePaciente/UpdatePacienteCommandHandlerTests.cs` | 5 | 3.17 Pacientes |
| `SOFIA.UnitTests/Pacientes/Commands/UpdatePaciente/UpdatePacienteCommandValidatorTests.cs` | 3 | 3.17 Pacientes |
| `SOFIA.UnitTests/Pacientes/Domain/PacienteClienteTests.cs` | 5 | 3.17 Pacientes |
| `SOFIA.UnitTests/PresentacionesVenta/Commands/CreatePresentacionVenta/CreatePresentacionVentaCommandHandlerTests.cs` | 5 | 3.18 PresentacionesVenta |
| `SOFIA.UnitTests/PresentacionesVenta/Commands/CreatePresentacionVenta/CreatePresentacionVentaCommandValidatorTests.cs` | 4 | 3.18 PresentacionesVenta |
| `SOFIA.UnitTests/PresentacionesVenta/Commands/DeletePresentacionVenta/DeletePresentacionVentaCommandHandlerTests.cs` | 3 | 3.18 PresentacionesVenta |
| `SOFIA.UnitTests/PresentacionesVenta/Commands/UpdatePresentacionVenta/UpdatePresentacionVentaCommandHandlerTests.cs` | 5 | 3.18 PresentacionesVenta |
| `SOFIA.UnitTests/PresentacionesVenta/Commands/UpdatePresentacionVenta/UpdatePresentacionVentaCommandValidatorTests.cs` | 4 | 3.18 PresentacionesVenta |
| `SOFIA.UnitTests/PresentacionesVenta/Domain/PresentacionVentaTests.cs` | 13 | 3.18 PresentacionesVenta |
| `SOFIA.UnitTests/Profesionales/Commands/CargaMasivaProfesionalesSalud/CargaMasivaProfesionalesSaludCommandHandlerTests.cs` | 2 | 3.19 Profesionales |
| `SOFIA.UnitTests/Profesionales/Commands/CreateProfesionalSalud/CreateProfesionalSaludCommandHandlerTests.cs` | 2 | 3.19 Profesionales |
| `SOFIA.UnitTests/Profesionales/Commands/DeleteProfesionalSalud/DeleteProfesionalSaludCommandHandlerTests.cs` | 4 | 3.19 Profesionales |
| `SOFIA.UnitTests/Profesionales/Commands/UpdateProfesionalSalud/UpdateProfesionalSaludCommandHandlerTests.cs` | 3 | 3.19 Profesionales |
| `SOFIA.UnitTests/Proveedores/Commands/CargaMasivaProveedores/CargaMasivaProveedoresCommandHandlerTests.cs` | 2 | 3.20 Proveedores |
| `SOFIA.UnitTests/Proveedores/Commands/CreateProveedor/CreateProveedorCommandHandlerTests.cs` | 3 | 3.20 Proveedores |
| `SOFIA.UnitTests/Proveedores/Commands/DeleteProveedor/DeleteProveedorCommandHandlerTests.cs` | 4 | 3.20 Proveedores |
| `SOFIA.UnitTests/Proveedores/Commands/RegistrarPrecioProveedor/RegistrarPrecioProveedorCommandHandlerTests.cs` | 7 | 3.20 Proveedores |
| `SOFIA.UnitTests/Proveedores/Commands/RegistrarPrecioProveedor/RegistrarPrecioProveedorCommandValidatorTests.cs` | 7 | 3.20 Proveedores |
| `SOFIA.UnitTests/Proveedores/Commands/UpdateProveedor/UpdateProveedorCommandHandlerTests.cs` | 3 | 3.20 Proveedores |
| `SOFIA.UnitTests/Proveedores/Domain/HistorialPrecioProveedorTests.cs` | 4 | 3.20 Proveedores |
| `SOFIA.UnitTests/Recetas/Commands/CreateReceta/CreateRecetaCommandHandlerTests.cs` | 4 | 3.21 Recetas |
| `SOFIA.UnitTests/Recetas/Commands/CreateReceta/CreateRecetaCommandValidatorTests.cs` | 3 | 3.21 Recetas |
| `SOFIA.UnitTests/Recetas/Commands/DeleteReceta/DeleteRecetaCommandHandlerTests.cs` | 3 | 3.21 Recetas |
| `SOFIA.UnitTests/Recetas/Commands/UpdateReceta/UpdateRecetaCommandHandlerTests.cs` | 4 | 3.21 Recetas |
| `SOFIA.UnitTests/Recetas/Commands/UpdateReceta/UpdateRecetaCommandValidatorTests.cs` | 3 | 3.21 Recetas |
| `SOFIA.UnitTests/Recetas/Domain/RecetaMedicaTests.cs` | 11 | 3.21 Recetas |
| `SOFIA.UnitTests/Recetas/Queries/AnalizarReceta/AnalizarRecetaQueryValidatorTests.cs` | 3 | 3.21 Recetas |
| `SOFIA.UnitTests/Security/Commands/ChangePassword/ChangePasswordCommandHandlerTests.cs` | 3 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/ChangePassword/ChangePasswordCommandValidatorTests.cs` | 5 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/Cuentas/CuentaAdminProtectionTests.cs` | 5 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/ForgotPassword/ForgotPasswordCommandHandlerTests.cs` | 8 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/Login/LoginCommandHandlerTests.cs` | 8 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/Logout/LogoutCommandHandlerTests.cs` | 8 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/RefreshToken/RefreshTokenCommandHandlerTests.cs` | 10 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/Register/RegisterAccountCommandHandlerTests.cs` | 5 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/ResetPassword/ResetPasswordCommandHandlerTests.cs` | 9 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/Roles/AssignPermission/AssignPermissionToRolCommandHandlerTests.cs` | 6 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/Roles/AssignRol/AssignRolToUserCommandHandlerTests.cs` | 4 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/Roles/CargaMasivaRoles/CargaMasivaRolesCommandHandlerTests.cs` | 2 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/Roles/DeleteRol/DeleteRolCommandHandlerTests.cs` | 2 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/Roles/RemoveRol/RemoveRolFromUserCommandHandlerTests.cs` | 5 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/Roles/RevokePermission/RevokePermissionFromRolCommandHandlerTests.cs` | 2 | 3.22 Security |
| `SOFIA.UnitTests/Security/Commands/Roles/SetSucursales/SetRolSucursalesCommandHandlerTests.cs` | 3 | 3.22 Security |
| `SOFIA.UnitTests/Security/Domain/CuentaPasswordTests.cs` | 9 | 3.22 Security |
| `SOFIA.UnitTests/Security/Domain/CuentaTenantTests.cs` | 4 | 3.22 Security |
| `SOFIA.UnitTests/Security/Domain/RefreshTokenTests.cs` | 3 | 3.22 Security |
| `SOFIA.UnitTests/Security/PasswordRecoveryEmailTests.cs` | 2 | 3.22 Security |
| `SOFIA.UnitTests/SeriesFiscales/Commands/CreateSerieFiscal/CreateSerieFiscalCommandHandlerTests.cs` | 8 | 3.23 SeriesFiscales |
| `SOFIA.UnitTests/SeriesFiscales/Commands/CreateSerieFiscal/CreateSerieFiscalCommandValidatorTests.cs` | 8 | 3.23 SeriesFiscales |
| `SOFIA.UnitTests/SeriesFiscales/Commands/DeleteSerieFiscal/DeleteSerieFiscalCommandHandlerTests.cs` | 3 | 3.23 SeriesFiscales |
| `SOFIA.UnitTests/SeriesFiscales/Commands/UpdateSerieFiscal/UpdateSerieFiscalCommandHandlerTests.cs` | 7 | 3.23 SeriesFiscales |
| `SOFIA.UnitTests/SeriesFiscales/Commands/UpdateSerieFiscal/UpdateSerieFiscalCommandValidatorTests.cs` | 5 | 3.23 SeriesFiscales |
| `SOFIA.UnitTests/SeriesFiscales/Domain/SunatComprobanteEmitidoTests.cs` | 4 | 3.23 SeriesFiscales |
| `SOFIA.UnitTests/SeriesFiscales/Domain/SunatSerieFiscalTests.cs` | 9 | 3.23 SeriesFiscales |
| `SOFIA.UnitTests/Servicios/Commands/AgendarServicio/AgendarServicioCommandHandlerTests.cs` | 4 | 3.24 Servicios |
| `SOFIA.UnitTests/Servicios/Commands/RegistrarInmunizacion/RegistrarInmunizacionCommandHandlerTests.cs` | 12 | 3.24 Servicios |
| `SOFIA.UnitTests/Servicios/Commands/RegistrarInmunizacion/RegistrarInmunizacionCommandValidatorTests.cs` | 2 | 3.24 Servicios |
| `SOFIA.UnitTests/Sucursales/Commands/CargaMasivaSucursales/CargaMasivaSucursalesCommandHandlerTests.cs` | 2 | 3.25 Sucursales |
| `SOFIA.UnitTests/Sucursales/Commands/CreateSucursal/CreateSucursalCommandHandlerTests.cs` | 4 | 3.25 Sucursales |
| `SOFIA.UnitTests/Sucursales/Commands/DeleteSucursal/DeleteSucursalCommandHandlerTests.cs` | 13 | 3.25 Sucursales |
| `SOFIA.UnitTests/Sucursales/Commands/UpdateSucursal/UpdateSucursalCommandHandlerTests.cs` | 5 | 3.25 Sucursales |
| `SOFIA.UnitTests/Sucursales/Domain/SucursalTests.cs` | 3 | 3.25 Sucursales |
| `SOFIA.UnitTests/Transferencias/Commands/CreateTransferencia/CreateTransferenciaCommandHandlerTests.cs` | 6 | 3.26 Transferencias |
| `SOFIA.UnitTests/Transferencias/Domain/TransferenciaTests.cs` | 23 | 3.26 Transferencias |
| `SOFIA.UnitTests/UnidadesMedida/Commands/CargaMasivaUnidadesMedida/CargaMasivaUnidadesMedidaCommandHandlerTests.cs` | 2 | 3.27 UnidadesMedida |
| `SOFIA.UnitTests/UnidadesMedida/Commands/CreateUnidadMedida/CreateUnidadMedidaCommandHandlerTests.cs` | 2 | 3.27 UnidadesMedida |
| `SOFIA.UnitTests/UnidadesMedida/Commands/DeleteUnidadMedida/DeleteUnidadMedidaCommandHandlerTests.cs` | 5 | 3.27 UnidadesMedida |
| `SOFIA.UnitTests/UnidadesMedida/Commands/UpdateUnidadMedida/UpdateUnidadMedidaCommandHandlerTests.cs` | 3 | 3.27 UnidadesMedida |
| `SOFIA.UnitTests/Ventas/Commands/ActualizarVentaPendiente/ActualizarVentaPendienteCommandHandlerTests.cs` | 10 | 3.28 Ventas |
| `SOFIA.UnitTests/Ventas/Commands/ActualizarVentaPendiente/ActualizarVentaPendienteCommandValidatorTests.cs` | 3 | 3.28 Ventas |
| `SOFIA.UnitTests/Ventas/Commands/AnularVenta/AnularVentaCommandHandlerTests.cs` | 11 | 3.28 Ventas |
| `SOFIA.UnitTests/Ventas/Commands/CompletarVenta/CompletarVentaCommandHandlerTests.cs` | 21 | 3.28 Ventas |
| `SOFIA.UnitTests/Ventas/Commands/CompletarVenta/CompletarVentaCommandValidatorTests.cs` | 8 | 3.28 Ventas |
| `SOFIA.UnitTests/Ventas/Commands/CreateVenta/CreateVentaCommandHandlerTests.cs` | 33 | 3.28 Ventas |
| `SOFIA.UnitTests/Ventas/Commands/CreateVenta/CreateVentaCommandValidatorTests.cs` | 13 | 3.28 Ventas |
| `SOFIA.UnitTests/Ventas/Common/VentaComprobanteGeneratorTests.cs` | 9 | 3.28 Ventas |
| `SOFIA.UnitTests/Ventas/Common/VentaDetalleFactoryTests.cs` | 31 | 3.28 Ventas |
| `SOFIA.UnitTests/Ventas/Common/VentaSeguroProcessorTests.cs` | 5 | 3.28 Ventas |
| `SOFIA.UnitTests/Ventas/Domain/VentaPagoTests.cs` | 5 | 3.28 Ventas |
| `SOFIA.UnitTests/Ventas/Domain/VentaTests.cs` | 31 | 3.28 Ventas |
| `SOFIA.IntegrationTests/Api/ExportRequestValidationIntegrationTests.cs` | 5 | 4.1 |
| `SOFIA.IntegrationTests/Api/GlobalExceptionHandlerTests.cs` | 4 | 4.1 |
| `SOFIA.IntegrationTests/Api/LoggingConfigurationTests.cs` | 2 | 4.1 |
| `SOFIA.IntegrationTests/Api/NotFoundResponseIntegrationTests.cs` | 1 | 4.1 |
| `SOFIA.IntegrationTests/Api/RegistrarEmpresaValidationIntegrationTests.cs` | 2 | 4.1 |
| `SOFIA.IntegrationTests/Api/SecurityHeadersIntegrationTests.cs` | 4 | 4.1 |
| `SOFIA.IntegrationTests/Auditoria/AuditTrailIntegrationTests.cs` | 3 | 4.2 |
| `SOFIA.IntegrationTests/Concurrency/ConcurrencyIntegrationTests.cs` | 4 | 4.2 |
| `SOFIA.IntegrationTests/Dashboard/DashboardCajaDevolucionesIntegrationTests.cs` | 8 | 4.5 |
| `SOFIA.IntegrationTests/Dashboard/DashboardInventarioIntegrationTests.cs` | 6 | 4.5 |
| `SOFIA.IntegrationTests/Dashboard/DashboardVentasIntegrationTests.cs` | 8 | 4.5 |
| `SOFIA.IntegrationTests/Email/EmailInfrastructureTests.cs` | 11 | 4.6 |
| `SOFIA.IntegrationTests/Empleados/EmpleadoSinApellidoMaternoIntegrationTests.cs` | 3 | 4.4 |
| `SOFIA.IntegrationTests/Excel/ExcelReaderServiceTests.cs` | 8 | 4.6 |
| `SOFIA.IntegrationTests/FormulacionesClinicas/CreateFormulacionClinicaIntegrationTests.cs` | 3 | 4.4 |
| `SOFIA.IntegrationTests/Laboratorios/LaboratoriosExportIntegrationTests.cs` | 2 | 4.4 |
| `SOFIA.IntegrationTests/Maestros/DeleteMaestroInUseIntegrationTests.cs` | 14 | 4.4 |
| `SOFIA.IntegrationTests/Medicamentos/CreateMedicamentoIntegrationTests.cs` | 3 | 4.4 |
| `SOFIA.IntegrationTests/Medicamentos/GetMedicamentosPosCatalogIntegrationTests.cs` | 7 | 4.4 |
| `SOFIA.IntegrationTests/Pacientes/GetPacientesIntegrationTests.cs` | 1 | 4.4 |
| `SOFIA.IntegrationTests/Persistence/DatabaseConstraintIntegrationTests.cs` | 9 | 4.2 |
| `SOFIA.IntegrationTests/Persistence/TransactionRollbackIntegrationTests.cs` | 1 | 4.2 |
| `SOFIA.IntegrationTests/Proveedores/PreciosProveedorIntegrationTests.cs` | 7 | 4.4 |
| `SOFIA.IntegrationTests/Recetas/GeminiRecetaAnalyzerTests.cs` | 2 | 4.6 |
| `SOFIA.IntegrationTests/Security/AuthIntegrationTests.cs` | 14 | 4.3 |
| `SOFIA.IntegrationTests/Security/CsrfHeaderIntegrationTests.cs` | 4 | 4.3 |
| `SOFIA.IntegrationTests/Security/JwtOptionsTests.cs` | 3 | 4.3 |
| `SOFIA.IntegrationTests/Security/MemoryPermissionCacheTests.cs` | 3 | 4.3 |
| `SOFIA.IntegrationTests/Security/RateLimitingIntegrationTests.cs` | 6 | 4.3 |
| `SOFIA.IntegrationTests/Security/TenantIsolationIntegrationTests.cs` | 9 | 4.3 |
| `SOFIA.IntegrationTests/Ventas/AnularVentaCajaIntegrationTests.cs` | 6 | 4.5 |
| `SOFIA.IntegrationTests/Ventas/PosLookupsIntegrationTests.cs` | 9 | 4.5 |
| `SOFIA.ArchitectureTests/ArchitectureTests.cs` | 3 | 2 |
| **Total** | **1254** (193 archivos) | |

### 9.2 Otros elementos del encargo y del inventario

| Elemento | Apartado |
|---|---|
| Archivos de apoyo de `SOFIA.UnitTests` (`CuentaFactory.cs`, `EmpresaFactory.cs`, `EmisorMocks.cs`) | 1.4 (punto 6) |
| Archivos de apoyo de `SOFIA.IntegrationTests` (`BaseIntegrationTest`, `SofiaWebAppFactory`, `TestCurrentUser`, `CapturingEmailSender`, `DashboardTestClock`, `PosCatalogSeeder`, `RegistroEmpresaBuilder`, `xunit.runner.json`) | 1.5 |
| Herramientas y versiones de los tres `.csproj` | 1.2 |
| Convenciones de nombres y organización | 1.3 |
| Convención de no probar query handlers | 1.7 y 5.3 |
| `commands-queries.csv` (203: 113 commands, 90 queries) | 5.2, 5.3, 6.1, 6.4 |
| `validadores.csv` (83) | 5.2, 6.2 |
| `entidades.csv` (50) | 5.2, 6.3 |
| Hooks y CI (`.husky/task-runner.json`, `.github/workflows/security.yml`) | 7 |
| Reglas de negocio cruzadas (RN-CAJ, RN-DEV, RN-VEN, RN-CMP, RN-REC, RN-AUT, RN-GEN, RN-CTA, RN-ROL, RN-SER) | 6.4, 6.5, 6.6 |
| `enums.csv`, `endpoints.csv`, `permisos.csv`, `codigos-error*.csv`, `scripts-dbup.csv`, `tablas.csv` | No aplican a este documento (no son pruebas); `endpoints.csv` se menciona en 2 como referencia de la prueba de autorización. |

---

## 10. Fuentes

Código de `sofia-apis` (commit `43f5529`), todo bajo `sofia-apis/`:

- `src/SOFIA.UnitTests/SOFIA.UnitTests.csproj`, `src/SOFIA.IntegrationTests/SOFIA.IntegrationTests.csproj`, `src/SOFIA.ArchitectureTests/SOFIA.ArchitectureTests.csproj`, `src/Directory.Build.props`, `src/SOFIA.Domain/SOFIA.Domain.csproj`, `SOFIA.Solution.slnx`.
- `src/SOFIA.ArchitectureTests/ArchitectureTests.cs`.
- `src/SOFIA.IntegrationTests/Infrastructure/*.cs` (7 archivos) y `src/SOFIA.IntegrationTests/xunit.runner.json`.
- Los 160 archivos de `src/SOFIA.UnitTests/` y los 32 de `src/SOFIA.IntegrationTests/` listados en 9.1 (nombres de métodos, cabeceras, constructores y, por muestreo, cuerpos: `AnularVentaCommandHandlerTests.cs`, `CuentaAdminProtectionTests.cs`, `ConcurrencyIntegrationTests.cs`, `GeminiRecetaAnalyzerTests.cs`, `LoggingConfigurationTests.cs`, `TransactionRollbackIntegrationTests.cs`, `ExportRequestValidationIntegrationTests.cs`, `DashboardVentasIntegrationTests.cs`).
- Clases de `src/SOFIA.Application` (handlers y validadores, solo para resolver nombres) y `src/SOFIA.Application/DependencyInjection.cs`.
- `.husky/pre-commit`, `.husky/commit-msg`, `.husky/task-runner.json`, `.github/workflows/security.yml`.

Inventarios y documentos (carpeta `sofia-docs`):

- `_inventario/pruebas-backend.csv`, `commands-queries.csv`, `validadores.csv`, `entidades.csv`, `README.md`, `REGLAS-AGENTES.md`.
- `ola-1-fundamentos/08-guia-del-desarrollador.md` (secciones 1, 4.1, 5, 6) y `02-arquitectura.md` (enlace).
- `ola-1-fundamentos/01-reglas-de-negocio/01a-administracion-y-seguridad.md`, `01b-catalogos.md`, `01c1-inventario-y-recetas.md`, `01c2-ventas-y-comprobantes.md`, `01c3-caja-devoluciones-dashboard.md` (IDs y títulos de reglas).
- `ola-1-fundamentos/09-pruebas/09b-spa.md` (formato de referencia).
- Script de cruce `09a-cov.js` (scratchpad de la sesión, solo lectura).
