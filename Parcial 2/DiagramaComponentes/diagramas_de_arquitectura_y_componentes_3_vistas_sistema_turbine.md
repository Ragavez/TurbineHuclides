# Arquitectura y Diagramas de Componentes — Sistema Turbine (3 Vistas)

Este documento presenta el diseño arquitectónico del sistema **Turbine** siguiendo el enfoque de **Vistas Múltiples / Modelo C4** y el patrón de **Arquitectura en Capas (*n-tier*)** combinada con procesamiento orientado a eventos, garantizando bajo acoplamiento, alta cohesión y cumplimiento estricto de las restricciones de acceso a datos.

---

## Vista 1: Arquitectura General en Capas e Integraciones Externas (Nivel Macro)

Muestra la topología global del sistema dividida en **3 capas** (Presentación, Lógica de Negocio y Persistencia), los conectores externos (HTTPS, API REST, Webhooks, SMTP) y la restricción arquitectónica de que ninguna capa superior accede directamente a la base de datos, pasando obligatoriamente por el `Repositorio de Datos` (`IDatabase`).

```plantuml
@startuml Vista1_Arquitectura_General_Turbine
title Vista 1: Arquitectura General en Capas e Integraciones - Sistema Turbine

top to bottom direction
skinparam backgroundColor #FFFFFF
skinparam defaultFontName SansSerif
skinparam defaultFontSize 13
skinparam nodesep 70
skinparam ranksep 80
skinparam shadowing false

skinparam package {
  BackgroundColor #F8FAFC
  BorderColor #475569
  FontColor #0F172A
  Padding 15
}
skinparam component {
  BackgroundColor #E2E8F0
  BorderColor #334155
  Padding 12
}
skinparam interface {
  BackgroundColor #FEF08A
  BorderColor #CA8A04
}

' --- NIVEL 0: ACTORES Y SISTEMAS EXTERNOS ---
actor "Usuarios Internos\n(Operativo, Admin, SuperAdmin)" as Usuarios

together {
  cloud "Sistemas Externos" {
    [Sistema Cliente\n(Brain)] as Brain
    [Pasarela de Pagos\n(Stripe)] as Stripe
    [Fuente Bancaria\n(Revolut / Extractos)] as Banco
    [Proveedor Envío\n(Correo / WhatsApp)] as ProveedorEnvio
  }
}

' --- NIVEL 1: CAPA DE PRESENTACIÓN ---
package "1. Capa de Presentación y Entrada" {
  [Interfaz Web Responsiva] as Web
  [API REST Externa] as ApiExt
}

' --- NIVEL 2: CAPA DE LÓGICA DE NEGOCIO ---
package "2. Capa de Lógica de Negocio (Núcleo Turbine)" {
  [Módulo Seguridad\ny Administración] as ModSeguridad
  [Módulo Gestión\nComercial] as ModComercial
  [Módulo Facturación\ny Cobros] as ModFacturacion
  [Módulo Automatización\ny Salida] as ModAuto
}

' --- NIVEL 3: CAPA DE PERSISTENCIA ---
package "3. Capa de Persistencia y Datos" {
  [Repositorio de Datos] as Repo
  database "Base de Datos\nRelacional" as BD
  folder "Almacenamiento Documental\n(PDFs Inmutables / ZIPs)" as Storage
}

' --- CONEXIONES EXTERNAS ---
Usuarios -down-> Web : HTTPS / TLS
Brain -down-> ApiExt : API REST (HTTPS / Token)
Stripe -down-> ModFacturacion : Webhook Eventos (JSON)
Banco -down-> ModFacturacion : Extractos CSV / API

' --- PRESENTACIÓN -> NEGOCIO ---
Web -(0- ModSeguridad : ISeguridad
Web -(0- ModComercial : IComercial
Web -(0- ModFacturacion : IFacturacion
Web -(0- ModAuto : IInformes

ApiExt -(0- ModComercial : IConsultaComercial
ApiExt -(0- ModFacturacion : IConsultaDeuda

' --- NEGOCIO -> PERSISTENCIA Y SALIDA ---
ModSeguridad -(0- Repo : IDatabase
ModComercial -(0- Repo : IDatabase
ModFacturacion -(0- Repo : IDatabase
ModAuto -(0- Repo : IDatabase

Repo -down-> BD : SQL / Transacciones ACID
ModFacturacion -down-> Storage : Escritura PDF Inmutable
ModAuto -down-> Storage : Empaquetado ZIP
ModAuto -up-> ProveedorEnvio : SMTP / API Mensajería

@enduml
```

### Elementos y decisiones clave de la Vista 1:
* **Arquitectura en Capas (*n-tier*):** Separa la interfaz de usuario (`Interfaz Web` y `API REST`), las reglas de negocio (`Núcleo Turbine`) y la persistencia (`Repositorio de Datos`, `Base de Datos` y `Almacenamiento Documental`).
* **Orientación a Eventos:** Los cobros de `Stripe` entran de forma asíncrona mediante *Webhooks* hacia el módulo de facturación y cobros.
* **Restricción de seguridad y consistencia:** La presentación nunca toca la base de datos ni el almacenamiento documental directamente; todo se canaliza mediante interfaces explícitas (*ball-and-socket* `-(0-`).

---

## Vista 2: Componentes de Seguridad y Gestión Comercial (Detalle de Interfaces)

Desglosa los paquetes internos de **Seguridad y Administración** y **Gestión Comercial**, detallando las interfaces proporcionadas (`0`) y requeridas (`(`) para el control de sesiones, gestión de usuarios, auditoría inmutable, cartera de clientes, suscripciones y aprobaciones por umbral.

```plantuml
@startuml Vista2_Componentes_Comercial_Seguridad
title Vista 2: Componentes de Seguridad y Gestión Comercial (Detalle de Interfaces)

top to bottom direction
skinparam backgroundColor #FFFFFF
skinparam defaultFontName SansSerif
skinparam defaultFontSize 13
skinparam nodesep 65
skinparam ranksep 75
skinparam shadowing false

skinparam package {
  BackgroundColor #F8FAFC
  BorderColor #475569
  FontColor #0F172A
  Padding 15
}
skinparam component {
  BackgroundColor #E2E8F0
  BorderColor #334155
  Padding 12
}
skinparam interface {
  BackgroundColor #FEF08A
  BorderColor #CA8A04
}

[Interfaz Web Responsiva] as Web
[API REST Externa] as ApiExt

package "Paquete Seguridad y Administración" {
  [Control de Acceso\ny Sesión] as Auth
  [Gestión de Usuarios\ny Roles] as Users
  [Auditoría y\nConfiguración] as Audit
}

package "Paquete Gestión Comercial" {
  [Gestión de Clientes\ny Contactos] as Clientes
  [Gestión de Suscripciones\ny Tarifas] as Susc
  [Flujo de\nAprobaciones] as Aprob
}

[Repositorio de Datos] as Repo

' --- PRESENTACIÓN / API -> NEGOCIO ---
Web -(0- Auth : IAuth
Web -(0- Users : IUsuarios
Web -(0- Audit : IConfigAuditoria
Web -(0- Clientes : IClientes
Web -(0- Susc : ISuscripciones
Web -(0- Aprob : IAprobaciones

ApiExt -(0- Clientes : IConsultaCliente
ApiExt -(0- Susc : IConsultaSuscripcion

' --- DEPENDENCIAS INTERNAS ---
Users -(0- Audit : IAuditoria
Clientes -(0- Audit : IAuditoria
Susc -(0- Clientes : IClientes
Susc -(0- Aprob : IAprobaciones
Aprob -(0- Audit : IAuditoria

' --- PERSISTENCIA ---
Auth -(0- Repo : IDatabase
Users -(0- Repo : IDatabase
Audit -(0- Repo : IDatabase
Clientes -(0- Repo : IDatabase
Susc -(0- Repo : IDatabase
Aprob -(0- Repo : IDatabase

@enduml
```

### Responsabilidades de los componentes (Vista 2):
* **Control de Acceso y Sesión (`IAuth`):** Valida el inicio de sesión corporativo sin autorregistro, controla tokens, expiración por inactividad y ámbito de acceso por cartera (CU-28).
* **Gestión de Usuarios y Roles (`IUsuarios`):** Gestiona la jerarquía (`Operativo` $\rightarrow$ `Admin` $\rightarrow$ `SuperAdmin`) y la delegación/reasignación de clientes sin dejar fichas huérfanas (CU-29, CU-30).
* **Auditoría y Configuración (`IConfigAuditoria` / `IAuditoria`):** Registra de forma inmutable acciones críticas y cambios económicos con motivo obligatorio (CU-32), y permite al `SuperAdmin` parametrizar tarifas, umbrales y festivos (CU-31).
* **Gestión de Clientes y Contactos (`IClientes` / `IConsultaCliente`):** Ejecuta altas manuales y por CSV (CU-01, CU-02), detecta duplicados por NIF/razón social sin fusiones automáticas (CU-03) y expone consultas controladas a la API externa (CU-33).
* **Gestión de Suscripciones y Tarifas (`ISuscripciones` / `IConsultaSuscripcion`):** Administra planes, prorrateos por días naturales reales (CU-04, CU-05), pausas con revisión programada (CU-06) y bajas comerciales (CU-07).
* **Flujo de Aprobaciones (`IAprobaciones`):** Evalúa los umbrales configurados ante descuentos o tarifas especiales y gestiona su resolución con separación de funciones (CU-09).

---

## Vista 3: Componentes de Facturación, Cobros y Automatización (Detalle de Interfaces)

Detalla el núcleo operativo de **Facturación y Cobros** y **Automatización y Salida**, mostrando la coordinación entre emisión de facturas, conciliación de pagos, bandeja diaria de incidencias, el motor batch temporizado y la exportación documental.

```plantuml
@startuml Vista3_Componentes_Facturacion_Automatizacion
title Vista 3: Componentes de Facturación, Cobros y Automatización (Detalle de Interfaces)

top to bottom direction
skinparam backgroundColor #FFFFFF
skinparam defaultFontName SansSerif
skinparam defaultFontSize 13
skinparam nodesep 65
skinparam ranksep 75
skinparam shadowing false

skinparam package {
  BackgroundColor #F8FAFC
  BorderColor #475569
  FontColor #0F172A
  Padding 15
}
skinparam component {
  BackgroundColor #E2E8F0
  BorderColor #334155
  Padding 12
}
skinparam interface {
  BackgroundColor #FEF08A
  BorderColor #CA8A04
}

[Interfaz Web Responsiva] as Web
[API REST Externa] as ApiExt
[Gestión de Suscripciones\ny Tarifas] as Susc
[Auditoría y\nConfiguración] as Audit

package "Paquete Facturación y Cobros" {
  [Motor de Facturación\ny PDF] as Fact
  [Conciliación\ny Pagos] as Pagos
  [Incidencias y\nBandeja Diaria] as Bandeja
}

package "Paquete Automatización y Salida" {
  [Motor Batch\n(Temporizador)] as Motor
  [Gestor de Notificaciones\ny Avisos] as Notif
  [Exportación e Informes\n(Gestoría)] as Export
}

[Repositorio de Datos] as Repo
folder "Almacenamiento Documental\n(PDFs Inmutables / ZIPs)" as Storage

' --- PRESENTACIÓN / API -> NEGOCIO ---
Web -(0- Fact : IFacturacion
Web -(0- Pagos : IPagos
Web -(0- Bandeja : IBandeja
Web -(0- Export : IInformes

ApiExt -(0- Fact : IConsultaDeuda
ApiExt -(0- Bandeja : ISolicitudBaja

' --- DEPENDENCIAS INTERNAS ---
Fact -(0- Susc : ISuscripciones
Fact -(0- Notif : INotificaciones
Fact -(0- Bandeja : IIncidencias
Fact -(0- Audit : IAuditoria

Pagos -(0- Fact : IFacturacion
Pagos -(0- Bandeja : ITareasConciliacion
Pagos -(0- Notif : INotificaciones

' --- INTERACCIÓN DEL MOTOR AUTOMÁTICO (CU-08, CU-10, CU-16, CU-20) ---
Motor -(0- Susc : IRevisionAnual
Motor -(0- Fact : ILoteMensual
Motor -(0- Notif : IRecordatorios
Motor -(0- Pagos : IConciliacionAuto

' --- EXPORTACIÓN CONTABLE ---
Export -(0- Fact : IDatosContables
Export -(0- Audit : IAuditoria

' --- PERSISTENCIA Y DOCUMENTOS ---
Fact -(0- Repo : IDatabase
Pagos -(0- Repo : IDatabase
Bandeja -(0- Repo : IDatabase
Notif -(0- Repo : IDatabase
Export -(0- Repo : IDatabase

Fact -down-> Storage : Guarda PDF Inmutable
Export -down-> Storage : Genera ZIP Gestoría

@enduml
```

### Responsabilidades de los componentes (Vista 3):
* **Motor de Facturación y PDF (`IFacturacion` / `ILoteMensual` / `IConsultaDeuda`):** Emite facturas ordinarias, puntuales y rectificativas (CU-10, CU-11, CU-13), asigna numeración correlativa atómica (CU-12), congela el PDF inmutable en almacenamiento documental y gestiona cierres de periodo y deuda incobrable con auditoría (CU-14, CU-27).
* **Conciliación y Pagos (`IPagos` / `IConciliacionAuto`):** Procesa eventos idempotentes de Stripe (CU-18), importa extractos bancarios (CU-19) y concilia automáticamente solo ante coincidencia exacta, derivando discrepancias a revisión manual (CU-20).
* **Incidencias y Bandeja Diaria (`IBandeja` / `IIncidencias` / `ITareasConciliacion` / `ISolicitudBaja`):** Centraliza en una vista única fallos de entrega (CU-21), discrepancias de cobro, disputas pausadas (CU-17) y solicitudes de baja entrantes por API (CU-33 $\rightarrow$ CU-07), gestionando su ciclo de vida (`PENDIENTE` $\rightarrow$ `LEIDO` $\rightarrow$ `RESUELTO`, CU-22).
* **Motor Batch (`Temporizador`):** Orquesta de forma determinista y reintentable los procesos automáticos: revisión anual de tarifas (`IRevisionAnual`, CU-08), lote de facturación del día 1 (`ILoteMensual`, CU-10), cadencia diaria de recordatorios (`IRecordatorios`, CU-16) y conciliación automática (`IConciliacionAuto`).
* **Gestor de Notificaciones y Avisos (`INotificaciones` / `IRecordatorios`):** Renderiza plantillas multidioma (ES/EN), envía facturas y avisos por correo/WhatsApp y registra su estado real de entrega (CU-15, CU-16).
* **Exportación e Informes (`IInformes`):** Alimenta el dashboard financiero con KPIs trazables (CU-24), genera exportaciones auditadas en CSV/PDF (CU-25) y empaqueta en ZIP versionado la documentación contable para la gestoría externa (CU-26).