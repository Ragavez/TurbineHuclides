@startuml Turbine_clases_con_atributos
' Vista del boceto acordado: atributos y relaciones, sin notas ni leyenda.
' Los tipos de datos compuestos y enumeraciones se definen al final.
top to bottom direction
skinparam backgroundColor #FFFFFF
skinparam defaultFontName SansSerif
skinparam defaultFontSize 14
skinparam classAttributeIconSize 0
skinparam shadowing false
skinparam roundcorner 6
skinparam minClassWidth 170
skinparam nodesep 65
skinparam ranksep 75
skinparam class {
  BackgroundColor #F7FAFC
  HeaderBackgroundColor #DFEAF2
  BorderColor #40566F
  ArrowColor #526D82
  FontColor #183B56
}
hide circle
hide empty methods

interface Sistema <<interfaz>>

class Usuario {
  -idUsuario : UUID {id}
  -nombre : String
  -correoCorporativo : String
  -hashContrasena : String
  -permisos : String [0..*]
  -activo : Boolean
  -/bandejaTrabajo : ItemBandeja [0..*]
}

class Admin
class SuperAdmin

class Cliente {
  -idCliente : UUID {id}
  -cuentaBancaria : String [0..1]
  -nombreComercial : String
  -razonSocial : String
  -identificadorFiscal : String
  -direccionFiscal : String
  -pais : String
  -contactos : Contacto [0..*]
  -moneda : String
  -estado : EstadoCliente
}

class Suscripcion {
  -idSuscripcion : UUID {id}
  -plan : String
  -tarifaMensual : Decimal
  -moneda : String
  -condiciones : String
  -plazoPagoDias : Integer
  -fechaInicio : Date
  -fechaFin : Date [0..1]
  -fechaBajaEfectiva : Date [0..1]
  -estado : EstadoSuscripcion
  -documentosFirmados : String [0..*]
}

class Factura {
  -idFactura : UUID {id}
  -serie : String [0..1]
  -numero : Integer [0..1]
  -tipoEmision : TipoEmision
  -fechaCreacion : DateTime
  -fechaEmision : Date [0..1]
  -fechaVencimiento : Date [0..1]
  -periodoDesde : Date [0..1]
  -periodoHasta : Date [0..1]
  -datosEmisor : DatosFiscales [0..1]
  -datosDestinatario : DatosFiscales [0..1]
  -lineas : LineaFactura [0..*]
  -baseImponible : Decimal
  -totalImpuestos : Decimal
  -importeTotal : Decimal
  -moneda : String
  -estadoEmision : EstadoEmision
  -estadoCobro : EstadoCobro
  -archivoPDF : String [0..1]
  -claveEmision : String
}

class Pagos {
  -idPago : UUID {id}
  -importe : Decimal
  -moneda : String
  -fechaRecepcion : DateTime
  -origen : OrigenPago
  -idOperacionOrigen : String
  -referencia : String [0..1]
  -estadoRevision : EstadoPago
}

class Solicitud {
  -idSolicitud : UUID {id}
  -tipo : TipoSolicitud
  -motivo : String
  -valorPropuesto : String [0..1]
  -fechaSolicitud : DateTime
  -estado : EstadoSolicitud
  -fechaResolucion : DateTime [0..1]
  -comentarioResolucion : String [0..1]
}

class Avisos {
  -idAviso : UUID {id}
  -tipo : TipoAviso
  -informacion : String
  -fechaProgramada : DateTime
  -estado : EstadoAviso
}

' Realizacion de la interfaz e herencia del boceto original.
Sistema <|.. Usuario
Sistema <|.. Suscripcion
Sistema <|.. Cliente
Sistema <|.. Factura
Sistema <|.. Avisos
Usuario <|-- Admin
Admin <|-- SuperAdmin

' Se mantiene el 1:1 Cliente-Suscripcion del boceto acordado.

Suscripcion "1" -- "1" Cliente : suscripcion del cliente
Cliente "1" -- "0..*" Factura : destinatario
Suscripcion "0..1" -- "0..*" Factura : origina
Factura "0..1" -- "0..*" Pagos : pagos asociados
Cliente "0..1" -- "0..*" Pagos : cliente asociado
Cliente "1" -- "0..*" Solicitud : solicitante
Usuario "0..1" -- "0..*" Solicitud : aprobador
Cliente "1" -- "0..*" Avisos : destinatario
