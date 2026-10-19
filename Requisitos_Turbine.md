# REQUISITOS - TURBINEH

## 1. Requisitos de Negocio (RN)
* **RN1** - **Relación Comercial Separada:** Mantener para cada cliente una suscripción independiente de las facturas (plan, tarifa, responsable, condiciones), sin presuponer permanencia contractual.
* **RN2** - **Control de Decisiones Económicas:** Cambios de tarifa o excepciones que superen umbrales configurados exigen aprobación previa auditada.
* **RN3** - **Histórico Económico Consistente:** Modificaciones posteriores en tarifas o plantillas no alteran facturas ya emitidas ni impiden justificar saldos pasados.
* **RN4** - **Automatización Verificable:** Los procesos masivos continúan ante fallos individuales, permiten reintentos seguros y visibilizan errores a los responsables.
* **RN5** - **Crecimiento Futuro:** Preparar el modelo para múltiples sociedades, monedas y nuevos canales sin exigirlos totalmente en la entrega inicial.

## 2. Requisitos de Usuario (RU)
* **RU1** - **Bandeja de Trabajo Diario:** Vista operativa centralizada con vencimientos, impagos, conciliaciones pendientes, entregas fallidas y aprobaciones.
* **RU2** - **Revisión Previa de Activación:** Muestra el prorrateo inicial y valida datos fiscales antes de emitir o activar cualquier suscripción.
* **RU3** - **Gestión de Excepciones:** Permite gestionar disputas, pagos dudosos y reintentos sin perder la visión clara de la deuda real.
* **RU4** - **Desglose Transparente de Cifras:** Explicación navegable desde los agregados del dashboard hasta los documentos individuales que componen cada número.
* **RU5** - **Agilidad Operativa:** Vistas guardadas, acciones masivas controladas y borradores automáticos para tareas de alta frecuencia.

## 3. Requisitos del sistema

## 3.1 Requisitos Funcionales (RF)

### Acceso, Usuarios y Clientes
* **FR1** - **Ciclo de Vida de Usuarios:** Invitación de usuarios sin autorregistro, recuperación por enlace caducable y desactivación manteniendo autoría histórica.
* **FR2** - **Control de Sesión:** Caducidad por inactividad, invalidación al desactivar cuenta y bloqueo temporal por reintentos fallidos.
* **FR3** - **Autorización Efectiva:** Validación de permisos en cada consulta/descarga; la URL no otorga acceso a recursos no autorizados.
* **FR4** - **Asignaciones y Suplencias:** Transferencia de clientes y tareas entre responsables con soporte de delegaciones temporales programadas.
* **FR5** - **Múltiples Contactos por Cliente:** Roles de comercial, facturación e impagos. Conservación de destinatarios históricos en cada envío.
* **FR6** - **Datos Fiscales e Internacionales:** Distinción entre nombre comercial y razón social; almacenamiento de identificadores internacionales y monedas.


### Suscripciones y Facturación
* **FR7** - **Validación y Duplicados:** Detección preventiva de NIF duplicados. Bloqueo de facturación si faltan campos obligatorios. Evitar fusiones automáticas.
* **FR8** - **Importación CSV Controlada:** Previsualización, mapeo de columnas, reporte de errores por fila y registro de auditoría del proceso.
* **FR9** - **Estados del Cliente:** Gestor del ciclo de vida (pendiente, activo, pausado, baja solicitada, inactivo) con consecuencias explícitas.
* **FR10** - **Ficha de Suscripción:** Registro independiente del plan, tarifas acordadas, condiciones de pago, fechas y documentos firmados.
* **FR11** - **Tabla de Tarifas Configurable:** Administración de tramos por facturación anual. Los cambios aplican a revisiones futuras, nunca a facturas pasadas.
* **FR12** - **Campaña de Revisión Anual:** Verificación asistida de rango de facturación, propuesta de tarifa y aplicación previa aprobación humana.
* **FR13** - **Excepciones Comerciales:** Registro y flujo de aprobación de descuentos extraordinarios, meses gratis y precios pactados.
* **FR14** - **Altas, Pausas y Bajas:** Cálculo e inspección de prorrateos. Pausa de recurrencia con fecha de revisión. Mantenimiento del recobro tras baja.
* **FR15** - **Generación de Facturas:** Facturas puntuales de múltiples líneas y recurrentes automatizadas desde la suscripción. Conservación de importe y moneda.
* **FR16** - **Vista Previa e Inspección:** Verificación gráfica y validación de reglas de anomalía (ej. importes desproporcionados) antes de emitir.
* **FR17** - **Numeración Consistente:** Series configurables y asignación de número único y correlativo en el momento exacto de emisión concurrente.

### Ciclo de Vida y Cobros
* **FR18** - **Ciclo de Vida de Facturas:** Estados: borrador, pendiente de aprobación, emitida, enviada, vencida, pagada, disputa, rectificada y anulada.
* **FR19** - **Flujos de Aprobación:** Requerida en cambios manuales, descuentos altos o importes fuera de rango. Las recurrentes estándar salen automáticamente.
* **FR20** - **Bandeja de Aprobaciones:** Gestión de solicitudes con separación de funciones entre solicitante y aprobador para operaciones sensibles.
* **FR21** - **Generación Mensual Idempotente:** Ejecución segura el día 1; tolerante a fallos, reintentable y con logs detallados por cliente.
* **FR22** - **Inmutabilidad del PDF:** Preservación estricta del documento emitido. Los cambios visuales futuros de la empresa no alteran PDFs pasados.
* **FR23** - **Cadencia de Recordatorios:** Ajuste de avisos según la condición general o pactada. Suspensión temporal en facturas bajo disputa.
* **FR24** - **Integración de Eventos Stripe:** Recepción y procesamiento simulado de cobros con protección ante eventos duplicados.
* **FR25** - **Conciliación de Movimientos:** Cruce por referencia e importe. Conciliación automática solo ante certeza absoluta; asignación manual en dudas.

### Comunicaciones e Integración
* **FR26** - **Discrepancias de Importe:** Gestión de pagos parciales o excesivos registrando el saldo pendiente sin marcar la factura como totalmente pagada.
* **FR27** - **Cancelación e Incobrables:** Proceso administrativo regulado para declarar deudas incobrables sin falsear cobros reales.
* **FR28** - **Entrega y Reintentos:** Registro del canal, destinatario, intento y rebote. Reenvío seguro sin duplicar número de factura.
* **FR29** - **Plantillas Multidioma:** Plantillas en español e inglés con variables de sustitución. Consentimiento previo grabado para uso de WhatsApp.
* **FR30** - **Notificaciones Internas:** Alertas a responsables sobre vencimientos, fallos y revisiones. Opciones de marcado como leída y atendida.
* **FR31** - **Incidencias Administrativas:** Registro de tickets internos vinculados a clientes/facturas con asignación de responsable y prioridad.
* **FR32** - **Dashboard Operativo:** Consola con acceso directo a resolver vencimientos, dudas de conciliación e impagos.
* **FR33** - **Dashboard Financiero e Informes:** Métricas de facturado, cobrado, pendiente y previsión recurrente con exportación a CSV/PDF.
* **FR34** - **API Controlada:** Interfaz documentada para consultar deuda, validar bajas y recibir solicitudes externas con autenticación propia.
* **FR35** - **Supervisión de Procesos:** Monitor de salud del sistema que muestra colas, errores de integración y estado de backups.

### Otros
* **FR36** - **Alerta de Desactivación de Contacto Único:** Validación que impide o advierte al desactivar el único contacto de facturación de una empresa activa para evitar fallos el día 1.
* **FR37** - **Entorno de Pruebas y Previsualización de Reglas:** Herramienta para probar plantillas y previsualizar el calendario de avisos antes de activar automatizaciones masivas.
* **FR38** - **Auditoría de Exportaciones Masivas:** Registro obligatorio de metadatos (usuario, fecha, hora, filtros) en el historial de auditoría al descargar grandes volúmenes de datos.
* **FR39** - **Inmunidad de Botones por Doble Clic:** Protección UI/UX mediante deshabilitación inmediata y claves de idempotencia en formularios de emisión y pago.
* **FR40** - **Notificaciones para la Dirección:** Posibilidad de configurar un Daily Digest o resumen consolidado diario por correo para superadministradores, evitando el colapso por alertas individuales instantáneas.

## 3.2 Requisitos No Funcionales (NFR)
* **NFR1** - **Integridad e Idempotencia:** Garantía de no duplicación de facturas o cobros ante concurrencia, doble clic o caídas de red.
* **NFR2** - **Precisión Monetaria Exacta:** Uso de aritmética decimal fija sin imprecisiones flotantes. Cuadre perfecto entre DB, PDF y Dashboard.
* **NFR3** - **Seguridad y Privacidad:** Encriptación de secretos, protección de URLs con tokens de vida corta y mensajes de error opacos al exterior.
* **NFR4** - **Auditoría Inmutable:** Registro permanente de cambios económicos, decisiones y exportaciones sin opción de reescritura de logs.
* **NFR5** - **Resiliencia y Copias de Seguridad:** Respaldos automáticos comprobados y capacidad de recuperación automática de tareas tras caídas.
* **NFR6** - **Rendimiento de Consultas:** Búsquedas fluidas sobre miles de registros. Procesamiento asíncrono para reportes pesados.
* **NFR7** - **Accesibilidad y Responsive:** Interfaz adaptada a escritorio, tablet y móvil (Android/iOS) utilizable mediante teclado e inspección clara.
* **NFR8** - **Despliegue Reproducible:** Separación total de código y configuración. Entornos de desarrollo, pruebas y producción homologados.
