
## Parte 1: Gestión de Usuarios y Asignación de Clientes (CU-29 / CU-30 / CU-22)

```plantuml
@startuml
autonumber
title Parte 1: Gestión de Usuarios y Asignación de Clientes (CU-29 / CU-30)

actor "Admin" as Admin
actor "Usuario" as Operativo
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

== Fase 1: Creación de Cuenta de Usuario (CU-29) ==

Admin -> Sistema : invitarUsuario(nombre, correoCorporativo, permisos, tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Admin")
activate BD
BD --> Sistema : sesionValida
deactivate BD

Sistema -> BD : crearUsuario(nombre, correoCorporativo, permisos, activo=true)
activate BD
create participant ":Usuario" as UserObj
BD --> UserObj : crear()
BD --> Sistema : idUsuario
deactivate BD

Sistema --> Admin : usuarioCreado(idUsuario)
deactivate Sistema

== Fase 2: Asignación y Delegación de Clientes (CU-30) ==

Admin -> Sistema : asignarClientes(listaIdCliente, idUsuario, tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Admin")
activate BD
BD --> Sistema : sesionValida
deactivate BD

Sistema -> BD : obtenerClientes(listaIdCliente)
activate BD
BD --> Sistema : listaClientes
deactivate BD

loop [Para cada Cliente en listaClientes]
    Sistema -> BD : actualizarResponsableCliente(idCliente, idUsuario)
    activate BD
    BD --> Sistema : asignacionOk
    deactivate BD
end

Sistema -> BD : transferirBandejaTrabajo(listaIdCliente, idUsuario)
activate BD
BD --> Sistema : bandejaActualizada
deactivate BD

Sistema --> Admin : asignacionCompletada()
deactivate Sistema

== Fase 3: Consulta de Bandeja Diaria (CU-22) ==

Operativo -> Sistema : consultarBandejaTrabajo(tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida(idUsuario)
deactivate BD

Sistema -> BD : obtenerBandejaTrabajo(idUsuario)
activate BD
BD --> Sistema : bandejaTrabajo
deactivate BD

Sistema --> Operativo : mostrarBandejaTrabajo(bandejaTrabajo)
deactivate Sistema
@enduml
```

---

## Parte 2: Alta de Cliente y Detección de Duplicados (CU-01 / CU-03)

```plantuml
@startuml
autonumber
title Parte 2: Alta de Cliente y Detección de Duplicados (CU-01 / CU-03)

actor "Usuario" as Operativo
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

Operativo -> Sistema : solicitarNuevoCliente(nombreComercial, razonSocial, identificadorFiscal, direccionFiscal, pais, contactos, moneda, tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida(idUsuario)
deactivate BD

Sistema -> BD : buscarDuplicadosCliente(identificadorFiscal, razonSocial)
activate BD
BD --> Sistema : listaCoincidencias
deactivate BD

opt [<<extend>> Duplicados detectados por NIF o Razón Social (CU-01 ext 4a)]
    Sistema --> Operativo : advertenciaDuplicado(listaCoincidencias)
    Operativo -> Sistema : confirmarCreacionExplicita(tokenSesion)
end

Sistema -> BD : crearCliente(nombreComercial, razonSocial, identificadorFiscal, direccionFiscal, pais, contactos, moneda, estado="ACTIVO", idUsuario)
activate BD
create participant ":Cliente" as ClienteObj
BD --> ClienteObj : crear()
BD --> Sistema : idCliente
deactivate BD

Sistema --> Operativo : clienteCreadoExitosamente(idCliente)
deactivate Sistema
@enduml
```

---

## Parte 3: Creación de Suscripción y Aprobación por Umbrales (CU-04 / CU-05 / CU-09)

```plantuml
@startuml
autonumber
title Parte 3: Crear Suscripción y Aprobación por Umbral (CU-04 / CU-05 / CU-09)


actor "Usuario" as Operativo
actor "Admin" as Admin
actor "SuperAdmin" as SuperAdmin
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

Operativo -> Sistema : crearSuscripcion(idCliente, plan, tarifaMensual, moneda, condiciones, plazoPagoDias, fechaInicio, tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida
deactivate BD

alt [<<extend>> Tarifa o descuento fuera de catálogo (CU-09)]
    Sistema -> BD : evaluarUmbralAprobacion(tarifaMensual, condiciones)
    activate BD
    BD --> Sistema : nivelAprobador
    deactivate BD

    Sistema -> BD : crearSolicitud(tipo, motivo, valorPropuesto, fechaSolicitud=now(), estado="PENDIENTE", idCliente)
    activate BD
    create participant ":Solicitud" as SolicitudObj
    BD --> SolicitudObj : crear()
    BD --> Sistema : idSolicitud
    deactivate BD

    Sistema --> Operativo : suscripcionPendienteAprobacion(idSolicitud)
    deactivate Sistema

    alt [nivelAprobador == "Admin"]
        Admin -> Sistema : revisarSolicitudes(tokenSesion)
        activate Sistema
        Sistema -> BD : validarSesionYPermisos(tokenSesion, "Admin")
        activate BD
        BD --> Sistema : sesionValida(idAdmin)
        deactivate BD
        Sistema -> BD : obtenerSolicitudesPendientes("Admin")
        activate BD
        BD --> Sistema : listaSolicitudes
        deactivate BD
        Sistema --> Admin : mostrarSolicitudes(listaSolicitudes)
        deactivate Sistema

        alt [Admin Aprueba la excepción]
            Admin -> Sistema : resolverSolicitud(idSolicitud, estado="APROBADA", comentarioResolucion, tokenSesion)
            activate Sistema
            Sistema -> BD : actualizarSolicitud(idSolicitud, "APROBADA", fechaResolucion=now(), comentarioResolucion, idAdmin)
            activate BD
            BD --> Sistema : ok
            deactivate BD

            Sistema -> BD : crearSuscripcion(idCliente, plan, tarifaMensual, moneda, condiciones, plazoPagoDias, fechaInicio, estado="ACTIVA")
            activate BD
            create participant ":Suscripcion" as SuscObj1
            BD --> SuscObj1 : crear()
            BD --> Sistema : idSuscripcion
            deactivate BD
            Sistema --> Admin : confirmacionActivacion(idSuscripcion)
            deactivate Sistema

        else [Admin Rechaza la excepción]
            Admin -> Sistema : resolverSolicitud(idSolicitud, estado="RECHAZADA", comentarioResolucion, tokenSesion)
            activate Sistema
            Sistema -> BD : actualizarSolicitud(idSolicitud, "RECHAZADA", fechaResolucion=now(), comentarioResolucion, idAdmin)
            activate BD
            BD --> Sistema : ok
            deactivate BD
            Sistema --> Admin : confirmacionRechazo()
            deactivate Sistema
        end

    else [nivelAprobador == "SuperAdmin"]
        SuperAdmin -> Sistema : resolverSolicitudSensible(idSolicitud, estado, comentarioResolucion, tokenSesion)
        activate Sistema
        Sistema -> BD : validarSesionYPermisos(tokenSesion, "SuperAdmin")
        activate BD
        BD --> Sistema : sesionValida(idSuperAdmin)
        deactivate BD

        alt [SuperAdmin Aprueba la excepción]
            Sistema -> BD : actualizarSolicitudYAuditoria(idSolicitud, "APROBADA", fechaResolucion=now(), comentarioResolucion, idSuperAdmin)
            activate BD
            BD --> Sistema : ok
            deactivate BD
            Sistema -> BD : crearSuscripcion(idCliente, plan, tarifaMensual, moneda, condiciones, plazoPagoDias, fechaInicio, estado="ACTIVA")
            activate BD
            create participant ":Suscripcion" as SuscObj2
            BD --> SuscObj2 : crear()
            BD --> Sistema : idSuscripcion
            deactivate BD
            Sistema --> SuperAdmin : confirmacionActivacionAuditada(idSuscripcion)
        else [SuperAdmin Rechaza la excepción]
            Sistema -> BD : actualizarSolicitudYAuditoria(idSolicitud, "RECHAZADA", fechaResolucion=now(), comentarioResolucion, idSuperAdmin)
            activate BD
            BD --> Sistema : ok
            deactivate BD
            Sistema --> SuperAdmin : confirmacionRechazoAuditado()
        end
        deactivate Sistema
    end

else [Tarifa estándar de catálogo (CU-04 include CU-05)]
    Sistema -> Sistema : calcularProrrateoInicial(fechaInicio, tarifaMensual)
    Sistema -> BD : crearSuscripcion(idCliente, plan, tarifaMensual, moneda, condiciones, plazoPagoDias, fechaInicio, estado="ACTIVA")
    activate BD
    create participant ":Suscripcion" as SuscObj3
    BD --> SuscObj3 : crear()
    BD --> Sistema : idSuscripcion
    deactivate BD
    Sistema --> Operativo : suscripcionCreada(idSuscripcion)
    deactivate Sistema
end
@enduml
```

---

## Parte 4: Facturación Mensual Recurrente, Incidencias y Bandeja (CU-10 / CU-12 / CU-15 / CU-21 / CU-22)

```plantuml
@startuml
autonumber
title Parte 4: Lote Recurrente, Avisos e Incidencias (CU-10 / CU-12 / CU-15 / CU-21 / CU-22)

actor "Motor de Automatización" as Motor
actor "Usuario" as Operativo
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

== Fase 1: Lote Mensual y Apertura de Aviso por Fallo (CU-10 include CU-12 / CU-15) ==

Motor -> Sistema : ejecutarLoteMensual(fechaGeneracion)
activate Sistema

Sistema -> BD : obtenerSuscripcionesActivas()
activate BD
BD --> Sistema : listaSuscripciones
deactivate BD

loop [Para cada Suscripcion activa]
    Sistema -> BD : obtenerCliente(idCliente)
    activate BD
    BD --> Sistema : datosCliente
    deactivate BD

    Sistema -> BD : obtenerSiguienteNumeroSerie(serie)
    activate BD
    BD --> Sistema : numero
    deactivate BD

    Sistema -> BD : crearFactura(serie, numero, tipoEmision="ORDINARIA", fechaCreacion=now(), fechaEmision, fechaVencimiento, lineas, baseImponible, totalImpuestos, importeTotal, moneda, estadoEmision="EMITIDA", estadoCobro="PENDIENTE", archivoPDF, claveEmision)
    activate BD
    create participant ":Factura" as FacturaObj
    BD --> FacturaObj : crear()
    BD --> Sistema : idFactura
    deactivate BD

    alt [Envío correcto a contactos de facturación]
        Sistema -> BD : actualizarEstadoEmisionFactura(idFactura, estadoEmision="ENVIADA")
        activate BD
        BD --> Sistema : ok
        deactivate BD
    else [<<extend>> Fallo de envío o sin contacto (CU-21)]
        Sistema -> BD : crearAviso(tipo="INCIDENCIA_ENTREGA", informacion="Fallo envio factura", fechaProgramada=now(), estado="PENDIENTE", idCliente)
        activate BD
        create participant ":Avisos" as AvisoIncObj
        BD --> AvisoIncObj : crear()
        BD --> Sistema : idAviso
        deactivate BD
    end
end

Sistema -> BD : guardarResumenEjecucionLote()
activate BD
BD --> Sistema : resumenGuardado
deactivate BD

Sistema --> Motor : loteCompletado()
deactivate Sistema

== Fase 2: Atender y Resolver la Incidencia en Bandeja Diaria (CU-22 / CU-21 / CU-15) ==

Operativo -> Sistema : consultarBandejaTrabajo(tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida(idUsuario)
deactivate BD

Sistema -> BD : obtenerBandejaTrabajo(idUsuario)
activate BD
BD --> Sistema : bandejaTrabajo
deactivate BD

Sistema --> Operativo : mostrarBandejaTrabajo(bandejaTrabajo)
deactivate Sistema

Operativo -> Sistema : resolverAviso(idAviso, nuevoContacto, tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida
deactivate BD

Sistema -> BD : actualizarAviso(idAviso, estado="RESUELTO")
activate BD
BD --> Sistema : ok
deactivate BD

Sistema -> BD : reenviarFactura(idFactura, nuevoContacto)
activate BD
BD --> Sistema : envioCorrecto
deactivate BD

Sistema -> BD : actualizarEstadoEmisionFactura(idFactura, estadoEmision="ENVIADA")
activate BD
BD --> Sistema : facturaActualizada
deactivate BD

Sistema --> Operativo : incidenciaResueltaYFacturaEnviada()
deactivate Sistema
@enduml
```

---

## Parte 5: Procesamiento de Pago Pasarela y Conciliación (CU-18 / CU-20 / CU-22)

```plantuml
@startuml
autonumber
title Parte 5: Pago Pasarela, Notificaciones y Conciliación Manual (CU-18 / CU-20 / CU-22)

actor "Pasarela de Pagos (Stripe)" as Stripe
actor "Usuario" as Operativo
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

== Fase 1: Webhook de Stripe y Procesamiento Automático (CU-18 include CU-20) ==

Stripe -> Sistema : procesarEventoPago(importe, moneda, fechaRecepcion, origen="STRIPE", idOperacionOrigen, referencia)
activate Sistema

Sistema -> BD : comprobarIdempotencia(idOperacionOrigen)
activate BD
BD --> Sistema : noProcesado
deactivate BD

Sistema -> BD : crearPago(importe, moneda, fechaRecepcion, origen, idOperacionOrigen, referencia, estadoRevision="PENDIENTE")
activate BD
create participant ":Pagos" as PagosObj
BD --> PagosObj : crear()
BD --> Sistema : idPago
deactivate BD

Sistema -> BD : buscarFacturaPorReferenciaEImporte(referencia, importe)
activate BD
BD --> Sistema : resultadoBusqueda
deactivate BD

alt [Coincidencia exacta de importe y referencia (CU-20)]
    Sistema -> BD : actualizarEstadoCobroFactura(idFactura, estadoCobro="PAGADA")
    activate BD
    BD --> Sistema : ok
    deactivate BD

    Sistema -> BD : actualizarEstadoRevisionPago(idPago, estadoRevision="CONCILIADO")
    activate BD
    BD --> Sistema : ok
    deactivate BD

    Sistema -> BD : crearAviso(tipo="PAGO_RECIBIDO", informacion="Pago conciliado automaticamente", fechaProgramada=now(), estado="PENDIENTE", idCliente)
    activate BD
    create participant ":Avisos" as AvisoPagoObj
    BD --> AvisoPagoObj : crear()
    BD --> Sistema : idAviso
    deactivate BD

    Sistema --> Stripe : webhookHTTP200OK()

else [<<extend>> Discrepancia en importe o referencia dudosa]
    Sistema -> BD : actualizarEstadoRevisionPago(idPago, estadoRevision="REVISION_MANUAL")
    activate BD
    BD --> Sistema : ok
    deactivate BD

    Sistema -> BD : crearAviso(tipo="REVISION_PAGO", informacion="Requiere conciliacion manual", fechaProgramada=now(), estado="PENDIENTE", idCliente)
    activate BD
    create participant ":Avisos" as AvisoRevObj
    BD --> AvisoRevObj : crear()
    BD --> Sistema : idAviso
    deactivate BD

    Sistema --> Stripe : webhookHTTP200OK()
end
deactivate Sistema

== Fase 2: Interacción en Bandeja de Trabajo del Operativo (CU-22 / CU-20) ==

Operativo -> Sistema : consultarBandejaTrabajo(tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida(idUsuario)
deactivate BD

Sistema -> BD : obtenerBandejaTrabajo(idUsuario)
activate BD
BD --> Sistema : bandejaTrabajo
deactivate BD

Sistema --> Operativo : mostrarBandejaTrabajo(bandejaTrabajo)
deactivate Sistema

alt [Notificación informativa de pago recibido (CU-22 ext 5a)]
    Operativo -> Sistema : marcarAvisoLeido(idAviso, tokenSesion)
    activate Sistema
    Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
    activate BD
    BD --> Sistema : sesionValida
    deactivate BD
    Sistema -> BD : actualizarAviso(idAviso, estado="LEIDO")
    activate BD
    BD --> Sistema : ok
    deactivate BD
    Sistema --> Operativo : avisoArchivado()
    deactivate Sistema

else [<<extend>> Conciliación manual por discrepancia (CU-20 ext 2c)]
    Operativo -> Sistema : conciliarManualmente(idPago, idFactura, idAviso, tokenSesion)
    activate Sistema

    Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
    activate BD
    BD --> Sistema : sesionValida
    deactivate BD

    Sistema -> BD : vincularPagoFactura(idPago, idFactura)
    activate BD
    BD --> Sistema : ok
    deactivate BD

    Sistema -> BD : actualizarEstadoCobroFactura(idFactura, estadoCobro="PAGADA")
    activate BD
    BD --> Sistema : ok
    deactivate BD

    Sistema -> BD : actualizarEstadoRevisionPago(idPago, estadoRevision="CONCILIADO")
    activate BD
    BD --> Sistema : ok
    deactivate BD

    Sistema -> BD : actualizarAviso(idAviso, estado="RESUELTO")
    activate BD
    BD --> Sistema : ok
    deactivate BD

    Sistema --> Operativo : conciliacionCompletadaExitosamente()
    deactivate Sistema
end
@enduml
```

---

## Parte 6A: Importación Masiva de Clientes desde CSV (CU-02 / CU-03)

```plantuml
@startuml
autonumber
title Parte 6A: Importación Masiva de Clientes desde CSV (CU-02 include CU-03)

actor "Usuario" as Operativo
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

Operativo -> Sistema : importarClientesCSV(archivoCSV, tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida(idUsuario)
deactivate BD

Sistema -> Sistema : parsearYValidarFormatoCSV(archivoCSV)

loop [Para cada fila de Cliente en archivoCSV]
    Sistema -> BD : buscarDuplicadosCliente(identificadorFiscal, razonSocial)
    activate BD
    BD --> Sistema : resultadoBusqueda
    deactivate BD

    alt [Fila válida y sin duplicados]
        Sistema -> BD : crearCliente(nombreComercial, razonSocial, identificadorFiscal, direccionFiscal, pais, contactos, moneda, estado="ACTIVO", idUsuario)
        activate BD
        create participant ":Cliente" as ClienteCSV
        BD --> ClienteCSV : crear()
        BD --> Sistema : idCliente
        deactivate BD
    else [<<extend>> Duplicado detectado o error de formato]
        Sistema -> Sistema : registrarFilaRechazadaEnInforme(fila, motivo)
    end
end

Sistema --> Operativo : resumenImportacionCSV(creados, rechazados)
deactivate Sistema
@enduml
```

---

## Parte 6B: Importación de Extractos Bancarios y Conciliación (CU-19 / CU-20)

```plantuml
@startuml
autonumber
title Parte 6B: Importación de Movimientos Bancarios (CU-19 include CU-20)

actor "Usuario" as Operativo
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

Operativo -> Sistema : importarExtractoBancario(archivoExtracto, origen="BANCO_REVOLUT", tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida
deactivate BD

loop [Para cada movimiento en archivoExtracto]
    Sistema -> BD : comprobarIdempotencia(idOperacionOrigen)
    activate BD
    BD --> Sistema : noProcesado
    deactivate BD

    Sistema -> BD : crearPago(importe, moneda, fechaRecepcion, origen, idOperacionOrigen, referencia, estadoRevision="PENDIENTE")
    activate BD
    create participant ":Pagos" as PagoBanco
    BD --> PagoBanco : crear()
    BD --> Sistema : idPago
    deactivate BD

    Sistema -> BD : buscarFacturaPorReferenciaEImporte(referencia, importe)
    activate BD
    BD --> Sistema : coincidencia
    deactivate BD

    alt [Coincidencia exacta de importe y referencia]
        Sistema -> BD : actualizarEstadoCobroFactura(idFactura, estadoCobro="PAGADA")
        activate BD
        BD --> Sistema : ok
        deactivate BD
        Sistema -> BD : actualizarEstadoRevisionPago(idPago, estadoRevision="CONCILIADO")
        activate BD
        BD --> Sistema : ok
        deactivate BD
    else [<<extend>> Discrepancia o referencia dudosa]
        Sistema -> BD : actualizarEstadoRevisionPago(idPago, estadoRevision="REVISION_MANUAL")
        activate BD
        BD --> Sistema : ok
        deactivate BD
        Sistema -> BD : crearAviso(tipo="REVISION_PAGO", informacion="Movimiento bancario requiere conciliacion", fechaProgramada=now(), estado="PENDIENTE", idCliente)
        activate BD
        create participant ":Avisos" as AvisoBanco
        BD --> AvisoBanco : crear()
        BD --> Sistema : idAviso
        deactivate BD
    end
end

Sistema --> Operativo : resumenConciliacionBancaria()
deactivate Sistema
@enduml
```

---

## Parte 7A: Pausar, Reactivar y Baja Comercial de Suscripción (CU-06 / CU-07)

```plantuml
@startuml
autonumber
title Parte 7A: Pausa, Reactivación y Baja Comercial (CU-06 / CU-07)

actor "Usuario" as Operativo
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

== Fase 1: Pausar y Reactivar Suscripción (CU-06) ==

Operativo -> Sistema : cambiarEstadoSuscripcion(idSuscripcion, accion, fechaRevision, motivo, tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida
deactivate BD

alt [accion == "PAUSAR"]
    Sistema -> BD : actualizarSuscripcion(idSuscripcion, estado="PAUSADA", fechaFin=fechaRevision, condiciones=motivo)
    activate BD
    BD --> Sistema : ok
    deactivate BD
    Sistema -> BD : crearAviso(tipo="REVISION_SUSCRIPCION", informacion="Revisar reactivacion", fechaProgramada=fechaRevision, estado="PENDIENTE", idCliente)
    activate BD
    create participant ":Avisos" as AvisoPausa
    BD --> AvisoPausa : crear()
    BD --> Sistema : idAviso
    deactivate BD
else [accion == "REACTIVAR"]
    Sistema -> BD : actualizarSuscripcion(idSuscripcion, estado="ACTIVA", fechaFin=null)
    activate BD
    BD --> Sistema : ok
    deactivate BD
end

Sistema --> Operativo : estadoSuscripcionActualizado()
deactivate Sistema

== Fase 2: Registrar Baja Comercial (CU-07) ==

Operativo -> Sistema : registrarBajaComercial(idSuscripcion, fechaBajaEfectiva, motivo, idAvisoBajaOpcional, tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida
deactivate BD

opt [<<extend>> Proviene de solicitud externa recibida por API]
    Sistema -> BD : actualizarAviso(idAvisoBajaOpcional, estado="RESUELTO")
    activate BD
    BD --> Sistema : ok
    deactivate BD
end

Sistema -> BD : actualizarSuscripcion(idSuscripcion, estado="BAJA_SOLICITADA", fechaBajaEfectiva=fechaBajaEfectiva)
activate BD
BD --> Sistema : ok
deactivate BD

Sistema -> BD : verificarFacturasImpagadas(idCliente)
activate BD
BD --> Sistema : facturasPendientes
deactivate BD

alt [Existen facturas pendientes de cobro]
    Sistema --> Operativo : bajaRegistradaConAvisoDeuda(facturasPendientes)
else [Sin deuda pendiente]
    Sistema --> Operativo : bajaRegistradaCorrectamente()
end
deactivate Sistema
@enduml
```

---

## Parte 7B: Revisión Anual Automática de Rango (CU-08 / CU-09)

```plantuml
@startuml
autonumber
title Parte 7B: Revisión Anual Automática de Rango (CU-08 extend CU-09)

actor "Motor de Automatización" as Motor
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

Motor -> Sistema : ejecutarRevisionAnual(fechaActual)
activate Sistema

Sistema -> BD : obtenerSuscripcionesParaRevisionAnual(fechaActual)
activate BD
BD --> Sistema : listaSuscripciones
deactivate BD

loop [Para cada Suscripcion a revisar]
    Sistema -> Sistema : calcularNuevoRangoYTarifa(suscripcion)

    alt [Cambio de tarifa estándar dentro de catálogo]
        Sistema -> BD : actualizarTarifaSuscripcion(idSuscripcion, nuevaTarifaMensual)
        activate BD
        BD --> Sistema : ok
        deactivate BD
    else [<<extend>> Supera umbral de excepción comercial (CU-09)]
        Sistema -> BD : crearSolicitud(tipo="TARIFA_ESPECIAL", motivo="Revision anual fuera de umbral", valorPropuesto=nuevaTarifaMensual, fechaSolicitud=now(), estado="PENDIENTE", idCliente)
        activate BD
        create participant ":Solicitud" as SolAnual
        BD --> SolAnual : crear()
        BD --> Sistema : idSolicitud
        deactivate BD
    end
end

Sistema --> Motor : revisionAnualCompletada()
deactivate Sistema
@enduml
```

---

## Parte 8A: Facturación Puntual, Rectificativas y Deuda Incobrable (CU-11 / CU-13 / CU-14)

```plantuml
@startuml
autonumber
title Parte 8A: Factura Puntual, Rectificativa y Deuda Incobrable (CU-11 / CU-13 / CU-14)

actor "Usuario" as Operativo
actor "Admin" as Admin
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

== Fase 1: Emitir Factura Puntual o Rectificativa (CU-11 / CU-13 include CU-12 / CU-15) ==

Operativo -> Sistema : emitirFacturaManual(idCliente, tipoEmision, lineas, facturaOrigenOpcional, motivo, tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida
deactivate BD

Sistema -> BD : obtenerSiguienteNumeroSerie(serie)
activate BD
BD --> Sistema : numero
deactivate BD

alt [tipoEmision == "PUNTUAL" (CU-11)]
    Sistema -> BD : crearFactura(serie, numero, tipoEmision="PUNTUAL", fechaCreacion=now(), fechaEmision=now(), lineas, baseImponible, totalImpuestos, importeTotal, estadoEmision="EMITIDA", estadoCobro="PENDIENTE", archivoPDF, claveEmision)
    activate BD
    create participant ":Factura" as FactPuntual
    BD --> FactPuntual : crear()
    BD --> Sistema : idFactura
    deactivate BD
else [tipoEmision == "RECTIFICATIVA" (CU-13)]
    Sistema -> BD : crearFactura(serie, numero, tipoEmision="RECTIFICATIVA", fechaCreacion=now(), fechaEmision=now(), lineas, baseImponible, totalImpuestos, importeTotal, estadoEmision="EMITIDA", estadoCobro="PAGADA", archivoPDF, claveEmision)
    activate BD
    create participant ":Factura" as FactRect
    BD --> FactRect : crear()
    BD --> Sistema : idFactura
    deactivate BD
end

Sistema -> BD : enviarFacturaCliente(idFactura, idCliente)
activate BD
BD --> Sistema : envioOk
deactivate BD

Sistema -> BD : actualizarEstadoEmisionFactura(idFactura, estadoEmision="ENVIADA")
activate BD
BD --> Sistema : ok
deactivate BD

Sistema --> Operativo : facturaEmitidaYEnviada(idFactura)
deactivate Sistema

== Fase 2: Declarar Deuda Incobrable (CU-14) ==

Admin -> Sistema : declararDeudaIncobrable(idFactura, motivoJustificado, tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Admin")
activate BD
BD --> Sistema : sesionValida(idAdmin)
deactivate BD

Sistema -> BD : actualizarEstadoCobroYAuditoria(idFactura, estadoCobro="INCOBRABLE", motivoJustificado, idAdmin)
activate BD
BD --> Sistema : ok
deactivate BD

Sistema --> Admin : deudaMarcadaIncobrable()
deactivate Sistema
@enduml
```

---

## Parte 8B: Gestión de Disputas y Lote Diario de Recordatorios (CU-16 / CU-17)

```plantuml
@startuml
autonumber
title Parte 8B: Disputas y Lote Diario de Recordatorios (CU-16 extend CU-17)

actor "Usuario" as Operativo
actor "Motor de Automatización" as Motor
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

== Fase 1: Pausar o Reanudar Seguimiento por Disputa (CU-17) ==

Operativo -> Sistema : gestionarDisputaFactura(idFactura, pausar, motivoDisputa, tokenSesion)
activate Sistema

Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida
deactivate BD

alt [pausar == true]
    Sistema -> BD : actualizarEstadoCobroFactura(idFactura, estadoCobro="DISPUTA")
    activate BD
    BD --> Sistema : ok
    deactivate BD
else [Reanudar seguimiento tras resolver disputa]
    Sistema -> BD : actualizarEstadoCobroFactura(idFactura, estadoCobro="PENDIENTE")
    activate BD
    BD --> Sistema : ok
    deactivate BD
end

Sistema --> Operativo : estadoDisputaActualizado()
deactivate Sistema

== Fase 2: Lote Diario de Recordatorios e Impagos (CU-16 extend CU-17) ==

Motor -> Sistema : ejecutarLoteRecordatorios(fechaActual)
activate Sistema

Sistema -> BD : obtenerFacturasVencidasOProximas(fechaActual)
activate BD
BD --> Sistema : listaFacturas
deactivate BD

loop [Para cada Factura pendiente]
    alt [estadoCobro == "DISPUTA" (<<extend>> Pausada por CU-17)]
        Sistema -> Sistema : omitirEnvioRecordatorio(idFactura)
    else [estadoCobro == "PENDIENTE"]
        Sistema -> BD : enviarRecordatorioCobro(idFactura, idCliente)
        activate BD
        BD --> Sistema : envioOk
        deactivate BD
        Sistema -> BD : crearAviso(tipo="RECORDATORIO_IMPAGO", informacion="Recordatorio enviado", fechaProgramada=now(), estado="LEIDO", idCliente)
        activate BD
        create participant ":Avisos" as AvisoRec
        BD --> AvisoRec : crear()
        BD --> Sistema : idAviso
        deactivate BD
    end
end

Sistema --> Motor : loteRecordatoriosFinalizado()
deactivate Sistema
@enduml
```

---

## Parte 9A: Inicio de Sesión, Consultas e Informes (CU-23 / CU-24 / CU-25 / CU-26 / CU-28)

```plantuml
@startuml
autonumber
title Parte 9A: Acceso, Consultas, Dashboard e Informes (CU-23 / CU-24 / CU-25 / CU-26 / CU-28)

actor "Usuario" as Operativo
actor "Admin" as Admin
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

== Fase 1: Inicio de Sesión y Recuperación de Acceso (CU-28) ==

Operativo -> Sistema : iniciarSesion(correoCorporativo, contrasena)
activate Sistema
Sistema -> BD : verificarCredenciales(correoCorporativo, hashContrasena)
activate BD
BD --> Sistema : resultadoAutenticacion
deactivate BD

alt [Credenciales válidas y usuario activo]
    Sistema --> Operativo : tokenSesionValido()
else [<<extend>> Solicitud de recuperación de acceso]
    Operativo -> Sistema : solicitarRecuperacionAcceso(correoCorporativo)
    Sistema -> BD : generarTokenRecuperacion(correoCorporativo)
    activate BD
    BD --> Sistema : enlaceEnviado
    deactivate BD
    Sistema --> Operativo : correoRecuperacionEnviado()
end
deactivate Sistema

== Fase 2: Ficha de Cliente y Dashboard KPI (CU-23 / CU-24) ==

Operativo -> Sistema : consultarFichaYDashboard(idClienteOpcional, filtros, tokenSesion)
activate Sistema
Sistema -> BD : validarSesionYPermisos(tokenSesion, "Usuario")
activate BD
BD --> Sistema : sesionValida
deactivate BD

alt [Consulta ficha e historial de cliente (CU-23)]
    Sistema -> BD : obtenerFichaHistorialCliente(idClienteOpcional)
    activate BD
    BD --> Sistema : datosCliente
    deactivate BD
    Sistema --> Operativo : mostrarFichaCliente(datosCliente)
else [Consulta dashboard y desglose KPI (CU-24)]
    Sistema -> BD : calcularMetricasYDesglose(filtros)
    activate BD
    BD --> Sistema : kpisTrazables
    deactivate BD
    Sistema --> Operativo : mostrarDashboard(kpisTrazables)
end
deactivate Sistema

== Fase 3: Exportación de Informes y Paquete Gestoría (CU-25 / CU-26) ==

Admin -> Sistema : exportarInformesYPaqueteGestoria(tipoExportacion, periodo, tokenSesion)
activate Sistema
Sistema -> BD : validarSesionYPermisos(tokenSesion, "Admin")
activate BD
BD --> Sistema : sesionValida(idAdmin)
deactivate BD

alt [tipoExportacion == "INFORME_CSV_PDF" (CU-25)]
    Sistema -> BD : generarInformeYRegistrarAuditoria(periodo, idAdmin)
    activate BD
    BD --> Sistema : archivoInforme
    deactivate BD
    Sistema --> Admin : descargarInforme(archivoInforme)
else [tipoExportacion == "PAQUETE_GESTORIA_ZIP" (CU-26)]
    Sistema -> BD : empaquetarFacturasPagosYResumenZIP(periodo, idAdmin)
    activate BD
    BD --> Sistema : paqueteZIPVersionado
    deactivate BD
    Sistema --> Admin : descargarPaqueteGestoria(paqueteZIPVersionado)
end
deactivate Sistema
@enduml
```

---

## Parte 9B: Administración del Sistema y API Externa (CU-27 / CU-31 / CU-32 / CU-33)

```plantuml
@startuml
autonumber
title Parte 9B: Administración del Sistema y API Externa (CU-27 / CU-31 / CU-32 / CU-33)

actor "SuperAdmin" as SuperAdmin
actor "Sistema Cliente (Brain)" as Brain
participant "«interfaz»\nSistema" as Sistema
database "Base de Datos" as BD

== Fase 1: Administración, Periodos y Auditoría (CU-27 / CU-31 / CU-32) ==

SuperAdmin -> Sistema : ejecutarOperacionAdministrativa(accion, parametros, motivoObligatorio, tokenSesion)
activate Sistema
Sistema -> BD : validarSesionYPermisos(tokenSesion, "SuperAdmin")
activate BD
BD --> Sistema : sesionValida(idSuperAdmin)
deactivate BD

alt [accion == "CERRAR_O_REABRIR_PERIODO" (CU-27)]
    Sistema -> BD : actualizarEstadoPeriodoYAuditoria(parametros, motivoObligatorio, idSuperAdmin)
    activate BD
    BD --> Sistema : periodoActualizado
    deactivate BD
else [accion == "CONFIGURAR_PARAMETROS" (CU-31)]
    Sistema -> BD : guardarParametrosSistemaYAuditoria(parametros, motivoObligatorio, idSuperAdmin)
    activate BD
    BD --> Sistema : parametrosGuardados
    deactivate BD
else [accion == "CONSULTAR_AUDITORIA" (CU-32)]
    Sistema -> BD : buscarRegistrosAuditoria(parametros)
    activate BD
    BD --> Sistema : registrosInmutables
    deactivate BD
end

Sistema --> SuperAdmin : operacionAdministrativaCompletada()
deactivate Sistema

== Fase 2: Atender Solicitudes Externas por API (CU-33) ==

Brain -> Sistema : solicitudAPIExterna(endpoint, payload, tokenAPI)
activate Sistema
Sistema -> BD : validarTokenAPIYPermisos(tokenAPI, endpoint)
activate BD
BD --> Sistema : accesoAutorizado
deactivate BD

alt [endpoint == "CONSULTAR_ESTADO_CLIENTE_O_DEUDA"]
    Sistema -> BD : obtenerEstadoClienteYFacturas(idCliente)
    activate BD
    BD --> Sistema : estadoYDeuda
    deactivate BD
    Sistema --> Brain : respuestaJSON200(estadoYDeuda)
else [endpoint == "SOLICITAR_BAJA_EXTERNA"]
    Sistema -> BD : crearAviso(tipo="SOLICITUD_BAJA_API", informacion=payload, fechaProgramada=now(), estado="PENDIENTE", idCliente)
    activate BD
    create participant ":Avisos" as AvisoAPI
    BD --> AvisoAPI : crear()
    BD --> Sistema : idAviso
    deactivate BD
    Sistema --> Brain : respuestaJSON202Aceptado(idAviso)
end
deactivate Sistema
@enduml
```
