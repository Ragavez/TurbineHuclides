# Arquitectura y Diagramas de Componentes — Sistema Turbine

## Vista 1: Arquitectura General en Capas e Integraciones Externas

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

cloud "Sistemas Externos" {
  [Sistema Cliente\n(Brain)] as Brain
  [Pasarela de Pagos\n(Stripe)] as Stripe
  [Fuente Bancaria\n(Revolut / Extractos)] as Banco
  [Proveedor Envío\n(Correo / WhatsApp)] as ProveedorEnvio
}

' --- NIVEL 1: CAPA DE PRESENTACIÓN Y ENTRADA ---
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

' --- NIVEL 3: CAPA DE PERSISTENCIA Y DATOS ---
package "3. Capa de Persistencia y Datos" {
  [Repositorio de Datos] as Repo
  database "Base de Datos\nRelacional" as BD
  folder "Almacenamiento Documental\n(PDFs Inmutables / ZIPs)" as Storage
}

' --- CONEXIONES EXTERNAS HACIA CAPA 1 ---
Usuarios -down-> Web : HTTPS / TLS
Banco -down-> Web : Carga Extractos CSV
Brain -down-> ApiExt : API REST (HTTPS / Token)
Stripe -down-> ApiExt : Webhook Eventos (HTTPS / JSON)

' --- CAPA 1 -> CAPA 2 ---
Web -(0- ModSeguridad : ISeguridad
Web -(0- ModComercial : IComercial
Web -(0- ModFacturacion : IFacturacion
Web -(0- ModAuto : IInformes

ApiExt -(0- ModComercial : IConsultaComercial
ApiExt -(0- ModFacturacion : IRecepcionEventosYDeuda

' --- CAPA 2 -> CAPA 3 (ACCESO EXCLUSIVO POR REPOSITORIO) ---
ModSeguridad -(0- Repo : IDatabase
ModComercial -(0- Repo : IDatabase
ModFacturacion -(0- Repo : IDatabase
ModAuto -(0- Repo : IDatabase

ModFacturacion -(0- Repo : IAlmacenamientoDoc
ModAuto -(0- Repo : IAlmacenamientoDoc

' --- PERSISTENCIA FÍSICA Y SALIDA EXTERNA ---
Repo -down-> BD : SQL / Transacciones ACID
Repo -down-> Storage : Escritura / Lectura Ficheros
ModAuto -up-> ProveedorEnvio : SMTP / API Mensajería

@enduml
```

## Vista 2: Componentes de Seguridad y Gestión Comercial

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

## Vista 3: Componentes de Facturación, Cobros y Automatización

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

cloud "Sistemas Externos" {
  [Pasarela de Pagos\n(Stripe)] as Stripe
  [Fuente Bancaria\n(Revolut / Extractos)] as Banco
  [Proveedor Envío\n(Correo / WhatsApp)] as ProveedorEnvio
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
database "Base de Datos\nRelacional" as BD
folder "Almacenamiento Documental\n(PDFs Inmutables / ZIPs)" as Storage

' --- ENTRADAS EXTERNAS A CAPA 1 ---
Stripe -down-> ApiExt : Webhook Eventos (JSON)
Banco -down-> Web : Carga Extractos CSV

' --- PRESENTACIÓN / API -> NEGOCIO ---
Web -(0- Fact : IFacturacion
Web -(0- Pagos : IPagosYExtractos
Web -(0- Bandeja : IBandeja
Web -(0- Export : IInformes

ApiExt -(0- Pagos : IWebhookPagos
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

' --- INTERACCIÓN DEL MOTOR AUTOMÁTICO ---
Motor -(0- Susc : IRevisionAnual
Motor -(0- Fact : ILoteMensual
Motor -(0- Notif : IRecordatorios
Motor -(0- Pagos : IConciliacionAuto

' --- EXPORTACIÓN CONTABLE ---
Export -(0- Fact : IDatosContables
Export -(0- Audit : IAuditoria

' --- PERSISTENCIA EXCLUSIVA POR REPOSITORIO ---
Fact -(0- Repo : IDatabase / IAlmacenamientoDoc
Pagos -(0- Repo : IDatabase
Bandeja -(0- Repo : IDatabase
Notif -(0- Repo : IDatabase
Export -(0- Repo : IDatabase / IAlmacenamientoDoc

Repo -down-> BD : SQL / ACID
Repo -down-> Storage : Guarda PDF / ZIP

' --- SALIDA EXTERNA ---
Notif -up-> ProveedorEnvio : SMTP / API Mensajería

@enduml
```