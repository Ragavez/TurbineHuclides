## Página 1

ESPECIFICACIÓN TÉCNICA & AUDITORÍA DE NEGOCIO
Análisis de Requisitos: Sistema Turbine
Evaluación Integral, Lista Consolidada de Requisitos (FR1-FR66, NFR1-NFR21) y Ajustes Directivos
66
Requisitos Funcionales
21
Requisitos No Funcionales
7
Decisiones Críticas Abiertas
1. Resumen Ejecutivo y Visión General
El presente documento constituye el análisis consolidado de requisitos para la plataforma interna de
facturación y cobros de Turbine. El objetivo prioritario es sustituir los procesos manuales por una
herramienta automatizada, determinista y auditable capaz de gestionar miles de clientes y suscripciones
sin convertirse en un ERP sobredimensionado.
Principio Director del Cliente: La aplicación debe operar de forma independiente al producto principal (Brain).
La generación mensual, las alertas y la conciliación de cobros deben funcionar sin fallos silenciosos,
garantizando que el trabajo operativo diario se realice con ﬂuidez, alta precisión contable e integridad de datos.
2. Correcciones Obligatorias sobre la Lista Inicial
Requisito Original Deﬁciencia / Ambigüedad Detectada Ajuste / Corrección Deﬁnitiva
RN2 (Precio ﬁjo) Asumía rigidez total de tarifas anuales. Admitir excepciones aprobadas: descuentos
temporales, meses gratuitos y precios pactados con
vigencia.
RN3 / FR9
(Recurrencia)
Incluía frecuencias semanales y
anuales.
Fijar la recurrencia ordinaria como mensual (día 1).
Otras frecuencias quedan pospuestas como
evolución posterior.
FR7 / FR8
(Bajas/Impago)
Bloqueaba la baja comercial si existía
deuda.
Distinguir baja comercial de recobro: la baja corta
nuevas cuotas, pero el ﬂujo de recobro de la deuda
anterior continúa.
FR12
(Modiﬁcaciones)
Permitía modiﬁcar facturas del
trimestre.
Una factura emitida nunca se edita directamente; se
rectiﬁca o anula trazablemente según normativa de
gestoría.
RU2 / FR2 (Rol
"Lector")
Denominación confusa para un usuario
activo.
Renombrar a "Usuario Operativo", con permisos
explícitos de edición limitados a sus clientes
asignados.
NFR6 (Retención) "Conservar el máximo tiempo legal". Sustituir por política explícita de anonimización
operativa en bajas, conservando los documentos
ﬁscales emitidos.
Turbine — Sistema Interno de Facturación Página 1 de 6

## Página 2

3. Requisitos de Negocio y Usuario Consolidados
A continuación se detallan los requisitos de nivel superior que rigen el comportamiento económico y las
operaciones de la plantilla administrativa.
Requisitos de Negocio (RN)
RN6 F — Relación Comercial Separada: Mantener para cada cliente una suscripción independiente de
las facturas (plan, tarifa, responsable, condiciones), sin presuponer permanencia contractual.
RN7 F — Control de Decisiones Económicas: Cambios de tarifa o excepciones que superen umbrales
conﬁgurados exigen aprobación previa auditada.
RN8 F — Histórico Económico Consistente: Modiﬁcaciones posteriores en tarifas o plantillas no
alteran facturas ya emitidas ni impiden justiﬁcar saldos pasados.
RN9 F — Automatización Veriﬁcable: Los procesos masivos continúan ante fallos individuales,
permiten reintentos seguros y visibilizan errores a los responsables.
RN10 NF — Crecimiento Futuro: Preparar el modelo para múltiples sociedades, monedas y nuevos
canales sin exigirlos totalmente en la entrega inicial.
Requisitos de Usuario (RU)
RU5 F — Bandeja de Trabajo Diario: Vista operativa centralizada con vencimientos, impagos,
conciliaciones pendientes, entregas fallidas y aprobaciones.
RU6 F — Revisión Previa de Activación: Muestra el prorrateo inicial y valida datos ﬁscales antes de
emitir o activar cualquier suscripción.
RU7 F — Gestión de Excepciones: Permite gestionar disputas, pagos dudosos y reintentos sin perder
la visión clara de la deuda real.
RU8 F — Desglose Transparente de Cifras: Explicación navegable desde los agregados del dashboard
hasta los documentos individuales que componen cada número.
RU9 NF — Agilidad Operativa: Vistas guardadas, acciones masivas controladas y borradores
automáticos para tareas de alta frecuencia.
4. Requisitos Funcionales: Acceso, Usuarios y Clientes
FR21 F — Ciclo de Vida de Usuarios: Invitación de usuarios sin autorregistro, recuperación por enlace
caducable y desactivación manteniendo autoría histórica.
FR22 F — Control de Sesión: Caducidad por inactividad, invalidación al desactivar cuenta y bloqueo
temporal por reintentos fallidos.
FR23 F — Autorización Efectiva: Validación de permisos en cada consulta/descarga; la URL no otorga
acceso a recursos no autorizados.
FR24 F — Asignaciones y Suplencias: Transferencia de clientes y tareas entre responsables con
soporte de delegaciones temporales programadas.
FR25 F — Múltiples Contactos por Cliente: Roles de comercial, facturación e impagos. Conservación
de destinatarios históricos en cada envío.
FR26 F — Datos Fiscales e Internacionales: Distinción entre nombre comercial y razón social;
almacenamiento de identiﬁcadores internacionales y monedas.
Turbine — Análisis de Requisitos Página 2 de 6

## Página 3

5. Requisitos Funcionales: Suscripciones y Facturación
FR27 F — Validación y Duplicados: Detección preventiva de NIF duplicados. Bloqueo de facturación si
faltan campos obligatorios. Evitar fusiones automáticas.
FR28 F — Importación CSV Controlada: Previsualización, mapeo de columnas, reporte de errores por
ﬁla y registro de auditoría del proceso.
FR29 F — Estados del Cliente: Gestor del ciclo de vida (pendiente, activo, pausado, baja solicitada,
inactivo) con consecuencias explícitas.
FR31 F — Ficha de Suscripción: Registro independiente del plan, tarifas acordadas, condiciones de
pago, fechas y documentos ﬁrmados.
FR32 F — Tabla de Tarifas Conﬁgurable: Administración de tramos por facturación anual. Los
cambios aplican a revisiones futuras, nunca a facturas pasadas.
FR33 F — Campaña de Revisión Anual: Veriﬁcación asistida de rango de facturación, propuesta de
tarifa y aplicación previa aprobación humana.
FR34 F — Excepciones Comerciales: Registro y ﬂujo de aprobación de descuentos extraordinarios,
meses gratis y precios pactados.
FR35 F — Altas, Pausas y Bajas: Cálculo e inspección de prorrateos. Pausa de recurrencia con fecha
de revisión. Mantenimiento del recobro tras baja.
FR36 F — Generación de Facturas: Facturas puntuales de múltiples líneas y recurrentes
automatizadas desde la suscripción. Conservación de importe y moneda.
FR37 F — Vista Previa e Inspección: Veriﬁcación gráﬁca y validación de reglas de anomalía (ej.
importes desproporcionados) antes de emitir.
FR38 F — Numeración Consistente: Series conﬁgurables y asignación de número único e correlativo
en el momento exacto de emisión concurrente.
6. Requisitos Funcionales: Ciclo de Vida y Cobros
FR39 F — Ciclo de Vida de Facturas: Estados: borrador, pendiente de aprobación, emitida, enviada,
vencida, pagada, disputa, rectiﬁcada y anulada.
FR40 F — Flujos de Aprobación: Requerida en cambios manuales, descuentos altos o importes fuera
de rango. Las recurrentes estándar salen automáticamente.
FR41 F — Bandeja de Aprobaciones: Gestión de solicitudes con separación de funciones entre
solicitante y aprobador para operaciones sensibles.
FR42 F — Generación Mensual Idempotente: Ejecución segura el día 1; tolerante a fallos, reintentable
y con logs detallados por cliente.
FR43 F — Inmutabilidad del PDF: Preservación estricta del documento emitido. Los cambios visuales
futuros de la empresa no alteran PDFs pasados.
FR47 F — Cadencia de Recordatorios: Ajuste de avisos según la condición general o pactada.
Suspensión temporal en facturas bajo disputa.
FR48 F — Integración de Eventos Stripe: Recepción y procesamiento simulado de cobros con
protección ante eventos duplicados.
FR49 F — Conciliación de Movimientos: Cruce por referencia e importe. Conciliación automática solo
ante certeza absoluta; asignación manual en dudas.
Turbine — Análisis de Requisitos Página 3 de 6

## Página 4

7. Requisitos Funcionales: Comunicaciones e Integración
FR50 F — Discrepancias de Importe: Gestión de pagos parciales o excesivos registrando el saldo
pendiente sin marcar la factura como totalmente pagada.
FR51 F — Cancelación e Incobrables: Proceso administrativo regulado para declarar deudas
incobrables sin falsear cobros reales.
FR52 F — Entrega y Reintentos: Registro del canal, destinatario, intento y rebote. Reenvío seguro sin
duplicar número de factura.
FR53 F — Plantillas Multidioma: Plantillas en español e inglés con variables de sustitución.
Consentimiento previo grabado para uso de WhatsApp.
FR54 F — Notiﬁcaciones Internas: Alertas a responsables sobre vencimientos, fallos y revisiones.
Opciones de marcado como leída y atendida.
FR55 F — Incidencias Administrativas: Registro de tickets internos vinculados a clientes/facturas con
asignación de responsable y prioridad.
FR57 F — Dashboard Operativo: Consola con acceso directo a resolver vencimientos, dudas de
conciliación e impagos.
FR58 F — Dashboard Financiero e Informes: Métricas de facturado, cobrado, pendiente y previsión
recurrente con exportación a CSV/PDF.
FR60 F — API Controlada: Interfaz documentada para consultar deuda, validar bajas y recibir
solicitudes externas con autenticación propia.
FR62 F — Supervisión de Procesos: Monitor de salud del sistema que muestra colas, errores de
integración y estado de backups.
8. Incorporaciones Fundamentales Extraídas de la Entrevista
Tras la revisión detallada de las conversaciones con el cliente, se identiﬁcan e integran formalmente los
siguientes requisitos indispensables no recogidos en la lista preliminar:
FR67 F — Alerta de Desactivación de Contacto Único: Validación que impide o advierte al desactivar
el único contacto de facturación de una empresa activa para evitar fallos el día 1.
FR68 F — Entorno de Pruebas y Previsualización de Reglas: Herramienta para probar plantillas y
previsualizar el calendario de avisos antes de activar automatizaciones masivas.
FR69 F — Auditoría de Exportaciones Masivas: Registro obligatorio de metadatos (usuario, fecha,
hora, ﬁltros) en el historial de auditoría al descargar grandes volúmenes de datos.
FR70 F — Inmunidad de Botones por Doble Clic: Protección UI/UX mediante deshabilitación
inmediata y claves de idempotencia en formularios de emisión y pago.
Nota sobre Notiﬁcaciones para la Dirección: Se incorpora como FR71 la posibilidad de conﬁgurar un Daily
Digest o resumen consolidado diario por correo para superadministradores, evitando el colapso por alertas
individuales instantáneas.
Turbine — Análisis de Requisitos Página 4 de 6

## Página 5

9. Requisitos No Funcionales Consolidados (NFR)
NFR11 F — Integridad e Idempotencia: Garantía de no duplicación de facturas o cobros ante
concurrencia, doble clic o caídas de red.
NFR12 F — Precisión Monetaria Exacta: Uso de aritmética decimal ﬁja sin imprecisiones ﬂotantes.
Cuadre perfecto entre DB, PDF y Dashboard.
NFR13 F — Seguridad y Privacidad: Encriptación de secretos, protección de URLs con tokens de vida
corta y mensajes de error opacos al exterior.
NFR14 F — Auditoría Inmutable: Registro permanente de cambios económicos, decisiones y
exportaciones sin opción de reescritura de logs.
NFR15 F — Resiliencia y Copias de Seguridad: Respaldos automáticos comprobados y capacidad de
recuperación automática de tareas tras caídas.
NFR16 F — Rendimiento de Consultas: Búsquedas ﬂuidas sobre miles de registros. Procesamiento
asíncrono para reportes pesados.
NFR17 F — Accesibilidad y Responsive: Interfaz adaptada a escritorio, tablet y móvil (Android/iOS)
utilizable mediante teclado e inspección clara.
NFR18 F — Despliegue Reproducible: Separación total de código y conﬁguración. Entornos de
desarrollo, pruebas y producción homologados.
10. Elementos Excluidos Explícitamente (Fuera de Alcance)
Aclaraciones Directas del Cliente: Para evitar sobrecostes y desviaciones, los siguientes aspectos quedan
excluidos formalmente de la primera versión o simpliﬁcados de manera estricta.
Característica Excluida Justiﬁcación y Límite del Sistema
Pagos Fraccionados Comerciales No existe esa modalidad de venta. Solo se contemplan diferencias de importe en la
conciliación.
Motor de IA / Lógica No
Determinista
El núcleo de facturación debe ser 100% determinista y guiado por reglas de
negocio convencionales.
Integración con Calendarios
Oﬁciales
Se usará un gestor de festivos local/nacional simple; se descartan llamadas a APIs
ministeriales.
Acceso Directo para Gestoría No tendrán perﬁl de usuario. El sistema generará paquetes de exportación en
ZIP/CSV para su envío.
Verifactu e Integración Fiscal
Real
Queda fuera del alcance obligatorio de esta entrega. La asesoría externa validará
las reglas.
Turbine — Análisis de Requisitos Página 5 de 6

## Página 6

11. Decisiones de Negocio Pendientes de Conﬁrmación
Previo al cierre de la fase de diseño de software, deben acordarse formalmente los siguientes puntos
ambiguos con el cliente y gestoría:
# Punto de Decisión Opciones o Propuestas de Análisis
1 Fórmula exacta de prorrateo ¿Se dividen los días exactos del mes comercial o se toma base ﬁja de
30 días? Deﬁnir alta en el último día del mes.
2 Efecto de la baja comercial Conﬁrmar si la baja solicitada actúa de forma inmediata o si se aplica
siempre a ﬁn del periodo mensual en curso.
3 Umbrales exactos de aprobación Deﬁnir la cifra en euros o porcentaje de descuento a partir de la cual se
requiere la ﬁrma del Superadministrador.
4 Gestión contable de impagados Validar con gestoría el plazo y procedimiento para emitir rectiﬁcativas
y declarar créditos incobrables.
5 Desplazamiento por festivos Acordar si los avisos de cobro que caen en domingo pasan al lunes o
se emiten independientemente del día.
6 Tolerancia en conciliación Fijar el margen de céntimos aceptable para conciliaciones
automáticas por diferencias de tipo de cambio o comisiones.
7 Alcance de la API inicial Determinar si la API de la V1 solo responderá consultas de deuda o si
permitirá crear solicitudes de baja.
12. Indicaciones para el Modelado de Casos de Uso (UML)
No dibujar un Caso de Uso por cada validación: Reglas como la comprobación del NIF o el cálculo del
impuesto deben modelarse como reglas de negocio o lógica interna del sistema.
Atención a Flujos Alternativos: Focalizar el diagrama de casos de uso en las aprobaciones de
excepciones, la baja con deuda pendiente, los reintentos de cobro y la conciliación dudosa.
Actores Principales: Superadministrador, Administrador, Usuario Operativo y Sistema Externo (Stripe
simulado / API Brain).
Conclusión del Análisis: La especiﬁcación presentada equilibra la potencia operativa con la simplicidad de uso.
Con la aprobación de los puntos pendientes, el equipo está en condiciones de iniciar el diseño arquitectónico y
del modelo de dominio.
Turbine — Análisis de Requisitos Página 6 de 6