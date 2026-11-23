@startuml

entity Usuario{
*id : UUID <<PK>>
*nombre : VARCHAR
*correo : VARCHAR
*contraseña : VARCHAR
*permisos/tipo : VARCHAR
}

entity Avisos{
*id : UUID <<PK>>
*información : VARCHAR
*destinatario : UUID
*fecha : DATE
}

entity Suscripción{
*id : UUID <<PK>>
*plan : VARCHAR
*tarifa : VARCHAR
*condiciones : VARCHAR
*fecha inicio : DATE
*fecha fin : DATE
*estado : VARCHAR
}

entity Factura{
*id : UUID <<PK>>
*serie : VARCHAR
*numero : VARCHAR
*fecha emisión : DATE
*fecha vencimiento : DATE
*destinatario : UUID <<FK>>
*importe : DOUBLE
*estado : VARCHAR
}

entity Líneas_Factura{
*id : UUID <<PK>>
*concepto : VARCHAR
*cantidad : DOUBLE
*precio : DOUBLE
*% descuento : DOUBLE
*% impuesto : DOUBLE
*importe total : DOUBLE
}

entity Solicitud{
*id : UUID <<PK>>
*mensaje : VARCHAR
*fecha : DATE
*estado : VARCHAR
*fecha resolución : DATE
}

entity Cliente{
*id : UUID <<PK>>
*cuenta : VARCHAR
*nombre : VARCHAR
*contacto : VARCHAR
*moneda : VARCHAR
*dir.fiscal : VARCHAR
*razón social : VARCHAR
*id fiscal : VARCHAR
*país : VARCHAR
*contacto : UUID <<FK>>
}

entity Contacto{
*id : UUID <<PK>>
*datos contacto : VARCHAR
*rol : VARCHAR
*nombre : VARCHAR
*idioma : VARCHAR
}

entity Pagos{
*id : UUID <<PK>>
*importe : DECIMAL
*moneda : VARCHAR
*fecha recepción : TIMESTAMP
*origen : VARCHAR
*id operación origen : VARCHAR
referencia : VARCHAR
*estado revisión : VARCHAR
idCliente : UUID <<FK>>
idFactura : UUID <<FK>>
}

Usuario ||--o{ Solicitud : aprueba
Usuario ||--o{ Cliente : registra
Usuario }|--|| Trabajador : [hereda]
Usuario }|--|| Admin : [hereda]
Usuario }|--|| Superadmin : [hereda]
Usuario ||--|| Sistema : controla
Sistema ||--o{ Avisos : envía 
Cliente ||--o{ Solicitud : solicita
Cliente ||--o{ Contacto : contiene
Cliente ||--o{ Factura : paga
Cliente ||--|| Suscripción : escoge
Factura }o--|| Suscripción : incluye
Factura ||--|| Líneas_Factura : incluye
Avisos ||--o{ Cliente : avisa

Cliente |o--o{ Pagos : realiza
Factura |o--o{ Pagos : recibe

@enduml
