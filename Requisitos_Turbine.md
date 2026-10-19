# Requisitos - Turbineh

## 1. Requisitos de Negocio (RN)

* **RN1. Independencia Operativa:** Herramienta interna 100% independiente del producto Brain, accesible via web sin requerir integración en la interfaz de Brain.
* **RN2. Separación Relación Comercial vs. Factura:** Mantener de forma independiente la ficha/suscripción comercial (tarifas, condiciones de pago, responsable) de las facturas emitidas.
* **RN3. Modelo Recurrente Mensual y Revisión Anual:** Tarifa recurrente con emisión el día 1, revisable anualmente según la facturación del cliente sin alterar el histórico emitido.
* **RN4. Gobierno Económico y Trazabilidad:** Excepciones comerciales y cambios de tarifa requieren aprobación. Las facturas emitidas son inmutables (rectificativas).
* **RN5. Desconexión de Baja Comercial y Recobro:** La baja comercial frena la emisión de nuevas cuotas pero no paraliza ni destruye las tareas de cobro de deudas previas.

## 2. Requisitos de Usuario y Usabilidad (RU)

* **RU1. Eficiencia y Mínimos Clics:** Formularios y flujos optimizados para completar altas, consultas y emisión en menos de un minuto.
* **RU2. Bandeja de Trabajo Diario:** Panel centralizado donde cada empleado visualiza y resuelve sus tareas asignadas (vencimientos, impagos, aprobaciones, revisiones).
* **RU3. Control por Roles y Ámbitos:** Accesos diferenciados según rol (Superadmin, Admin, Operador) restringiendo la vista de clientes/facturas a lo asignado.
* **RU4. Explicabilidad del Dashboard:** Capacidad de clicar en cualquier KPI financiero (facturado, cobrado, pendiente) para desglosar los documentos que lo forman.

## 3. Requisitos del sistema

## 3.1. Requisitos Funcionales (RF)

### Gestión de Clientes, Contactos y Privacidad
* **RF1. Ficha Comercial y Fiscal:** Registrar nombre comercial, razón social, NIF/identificador fiscal, domicilio, país, moneda y estado comercial.
* **RF2. Múltiples Contactos y Destinatarios Fijos:** Registrar contactos por función (facturación, comercial, impagos) congelando los destinatarios en el histórico de cada envío. Alerta al borrar el único contacto activo.
* **RF3. Validación y Alerta de Duplicados:** Comprobar datos obligatorios antes de emitir y alertar de posibles duplicados por NIF sin fusionar fichas automáticamente.
* **RF4. Importación CSV de Clientes:** Previsualización, mapeo de columnas, reporte de errores por fila e importación masiva controlada.
* **RF5. Ciclo de Estados del Cliente:** Gestionar estados (Pendiente activación, Activo, Pausado, Baja) aplicando sus reglas de facturación correspondientes.
* **RF6. Cumplimiento RGPD y Anonimización:** Atender solicitudes de borrado/anonimización de contactos operativos sin invalidar la integridad de las facturas históricas emitidas.

### Suscripciones, Tarifas y Aprobaciones
* **RF7. Gestión de Suscripciones y Prorrateo:** Configurar plan, cuota, fecha de inicio y cálculo automático de prorrateo inicial hasta fin de mes.
* **RF8. Tabla de Tarifas Configurable:** Administración de tramos de precios por facturación anual con vigencia para revisiones futuras.
* **RF9. Campaña de Revisión Anual:** Identificación de clientes a revisar, registro de facturación verificada y propuesta de nueva cuota con aprobación.
* **RF10. Excepciones Comerciales y Descuentos:** Registro de descuentos o meses gratuitos con motivo y periodo de validez sujeto a aprobaciones.
* **RF11. Bandeja de Aprobaciones:** Solicitar, aprobar o rechazar cambios de importe, descuentos extraordinarios o vencimientos fuera de norma.

### Gestión de Facturación, Series y Documentos Adjuntos
* **RF12. Facturas Recurrentes Automáticas:** Generación masiva e independiente cada día 1 a partir de las suscripciones activas.
* **RF13. Facturas Puntuales Multilinea:** Creación de facturas con múltiples conceptos, unidades, precios, descuentos e impuestos.
* **RF14. Ciclo de Vida de Factura:** Control de estados: Borrador → Pendiente Aprobación → Emitida → Enviada → Vencida → Cobrada / Disputa / Rectificada.
* **RF15. Series y Numeración Correlativa:** Asignación segura e ininterrumpida de número dentro de su serie al emitir.
* **RF16. Facturas Rectificativas:** Generación de documentos rectificativos vinculados a la factura original para ajustes legales/económicos.
* **RF17. Vista Previa, PDF Corporativo y Adjuntos:** Vista previa de datos obligatorios, PDF fiel e incorporación de documentos adjuntos (pedidos, presupuestos firmados).

### Seguimiento de Cobros, Conciliación y Comunicaciones
* **RF18. Cadencia Configurable de Avisos:** Envíos al emitir, recordatorio preventivo (día 3) y vencimiento (día 5 o pactado), más flujo periódico de impago.
* **RF19. Pausa de Recordatorios (Disputa):** Congelación temporal de avisos automáticos indicando motivo y fecha de revisión.
* **RF20. Integración Simulada de Pagos (Stripe/Banco):** Recepción y consulta de cobros mediante webhook/evento simulado con control de duplicados.
* **RF21. Conciliación y Discrepancias de Importe:** Asociación por referencia/importe. Pagos parciales o excedentes quedan señalados para revisión manual.
* **RF22. Gestión de Plantillas Multilingüe y Previsualización:** Plantillas (email/WhatsApp simulado con consentimiento) en español/inglés con simulador/entorno de prueba previo al envío.
* **RF23. Incidencias Operativas y Digest del Administrador:** Registro de rebotes de email o cobros dudosos y envío opcional de resumen diario por email para administradores.

### Informes, API e Integración
* **RF24. Dashboard e Informes Exportables:** Indicadores de facturación, cobros, morosidad y previsión recurrente con exportación a CSV/PDF.
* **RF25. API de Consultas y Bajas:** Endpoint para consulta externa de deudas antes de procesar bajas comerciales en otros sistemas.
* **RF26. Exportación para Gestoría:** Paquete mensual/trimestral formateado con facturas emitidas, rectificativas y cobros para asesoría.
* **RF27. Entorno de Sandbox y Datos de Demo:** Capacidad de cargar y ejecutar datos sintéticos/demo separados de producción para pruebas sin riesgo.

## 3.2. Requisitos No Funcionales (RNF)

* **RNF1. Idempotencia y Prevención del Doble Clic:** Bloqueo de botones e identificadores únicos en operaciones monetarias para evitar cobros o emisiones duplicadas.
* **RNF2. Precisión Decimal Strict:** Cálculos con decimales exactos evitando errores de redondeo o coma flotante entre pantalla, PDF e informes.
* **RNF3. Seguridad de Accesos e Inactividad:** Expiración de sesión por inactividad, cifrado de datos y restricción de login al dominio de correo corporativo.
* **RNF4. Auditoría Imborrable de Acciones y Exportaciones:** Registro imborrable de cambios de datos fiscales, decisiones económicas, aprobaciones y descargas masivas de datos sensibles.
* **RNF5. Copias de Seguridad y Recuperación:** Respaldos automáticos de base de datos y documentos emitidos con procedimiento probado de restauración.
* **RNF6. Rendimiento y Procesos en Segundo Plano:** Búsquedas fluidas con miles de clientes y ejecución en segundo plano para tareas pesadas (generación masiva).
* **RNF7. Diseño Responsive Multiplataforma:** Interfaz web adaptada a PC, tablet y móvil con usabilidad mediante teclado y zoom.
* **RNF8. Supervisión de Automatizaciones:** Panel de salud para monitorizar la última ejecución de tareas programadas, errores de envío y colas.
* **RNF9. Política de Conservación y Purga Temporal:** Retención legal de documentos fiscales emitidos frente a la purga periódica configurable de informes temporales exportados.
