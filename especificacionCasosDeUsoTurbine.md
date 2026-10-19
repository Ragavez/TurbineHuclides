
## 1. Actores

| Actor | Tipo | Responsabilidad |
| :--- | :--- | :--- |
| **Usuario Operativo** | Primario, humano | Trabaja sobre los clientes que tiene asignados: fichas, suscripciones, facturas puntuales, seguimiento de cobros, incidencias. No cambia tarifas globales ni series ni ve el dashboard global. |
| **Administrador** | Primario, humano (hereda del Operativo) | Gestiona operación, asignaciones, aprobaciones según umbral, importaciones, rectificativas, cierre de periodos y exportaciones. |
| **Superadministrador** | Primario, humano (hereda del Administrador) | Configuración profunda: tarifas, series, umbrales, plantillas, usuarios, auditoría e incobrables. No necesita aprobación de nadie, pero sus acciones sensibles exigen motivo y quedan auditadas. |
| **Motor de Automatización** | Primario, sistema | Temporizador: lote mensual del día 1, recordatorios, resumen diario y conciliación programada. |
| **Cliente/Contacto** | Secundario, humano | Destinatario de facturas y avisos; no accede a la aplicación. |
| **Pasarela de Pagos (Stripe simulada)** | Secundario, sistema | Envía eventos de cobro con posibles duplicados. |
| **Fuente bancaria (Revolut / banco/extracto)** | Secundario, sistema o manual | Aporta movimientos para conciliar; diseñada para añadir fuentes. |
| **Sistema Cliente (Brain / API)** | Secundario, sistema | Consulta deuda y notifica solicitudes de baja mediante API autenticada. |
| **Proveedor de correo / WhatsApp (simulado)** | Secundario, sistema | Entrega y reporta el resultado de los envíos. |

La gestoría no es actor: recibe un paquete exportado (CU-26) y no tiene usuario en esta versión. Herencia de roles: Superadministrador hereda de Administrador, que hereda de Operativo. Las copias de seguridad y su restauración son un requisito transversal sin caso de uso propio; el Motor las ejecuta como parte de la operación.

---

## 2. Casos de uso 

### Clientes y contactos

#### CU-01 Gestionar cliente y contactos

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Usuario Operativo |
| **Requisitos cubiertos** | RF1, RF2, RF3, RF4, RF5, RF7, RF8, RF9, RF10, RF11, RF51, RU1, RU5, RU6, RNF1 |
| **Interesados y objetivos** | **Operativo:** alta rápida sin duplicados. **Empresa:** datos fiscales válidos. **Cliente:** recibir avisos en el contacto adecuado. |
| **Precondiciones** | Usuario autenticado con permiso sobre el cliente. |
| **Disparador** | El Operativo selecciona «Nuevo cliente» o edita uno existente. |
| **Garantía mínima** | No se guarda ningún registro parcial; el formulario conserva un borrador. |
| **Garantía de éxito** | Cliente guardado con datos fiscales y comerciales, contactos con rol, responsable y estado inicial; evento en la línea temporal y en la auditoría. |
| **Escenario principal** | 1. El Operativo solicita un nuevo cliente.<br>2. El sistema muestra el formulario con valores por defecto (país, moneda, vencimiento).<br>3. El Operativo introduce nombre comercial, razón social, identificador fiscal, domicilio, país, moneda e idioma.<br>4. El sistema valida los datos y busca duplicados (CU-03).<br>5. El Operativo añade contactos con rol (facturación, comercial, impagos), idioma y canales consentidos.<br>6. El Operativo confirma (botón protegido contra doble clic).<br>7. El sistema guarda, asigna responsable y estado, y registra el evento. |
| **Extensiones** | **4a.** Posible duplicado (resultado de CU-03): el Operativo cancela, reactiva la ficha histórica o confirma explícitamente. Nunca se fusiona automáticamente (la fusión es una ampliación: solo Administrador, controlada y auditada).<br>**3a.** Faltan datos imprescindibles para facturar: se guarda como «Pendiente de activación» listando qué falta.<br>**3b.** Cierre accidental del formulario: se recupera el borrador.<br>**5a.** Persona ya asociada a otra empresa: se reutiliza y se añade la nueva relación.<br>**5b.** Desactivar un contacto: queda inactivo, no se borra; los envíos pasados conservan sus destinatarios.<br>**5c.** Es el único contacto de facturación de un cliente activo: el sistema avisa de que el día 1 no habrá destinatario y pide confirmar o asignar otro.<br>**5d.** Canal WhatsApp sin consentimiento: no se activa.<br>**6a.** Doble clic: «ya se está procesando».<br>**7a.** Vuelve un cliente dado de baja: se reactiva la ficha histórica solo si el identificador coincide y se confirma; queda en el historial.<br>**7b.** Eliminar un cliente: borrado lógico; se conservan histórico y facturas según obligación legal. |
| **Relaciones** | `<<include>>` CU-03. Los cambios de estado se disparan desde CU-06 y CU-07. |

---

#### CU-02 Importar clientes desde CSV

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Administrador |
| **Requisitos cubiertos** | RF5, RF6, RNF1, RNF13, RNF11, RF69 |
| **Interesados y objetivos** | **Administrador:** migrar sin corromper datos. **Empresa:** trazabilidad de la carga. |
| **Precondiciones** | Usuario con permiso de importación; archivo CSV disponible. |
| **Disparador** | El Administrador selecciona «Importar CSV». |
| **Garantía mínima** | Si se cancela o falla, no se crean clientes parciales. |
| **Garantía de éxito** | Clientes válidos creados o actualizados y operación registrada con resumen. |
| **Escenario principal** | 1. El Administrador sube el archivo.<br>2. El sistema muestra una previsualización y pide mapear columnas.<br>3. El sistema valida cada fila (CU-03) y muestra errores y posibles duplicados.<br>4. El Administrador elige: corregir el fichero o importar solo filas válidas.<br>5. El sistema ejecuta la importación en segundo plano con estado visible.<br>6. El sistema registra la operación con creados, actualizados, rechazados y motivos. |
| **Extensiones** | **3a.** NIF duplicado en el fichero o en la base: se avisa y se decide fila a fila.<br>**3b.** Columna obligatoria mal mapeada: se bloquea la confirmación.<br>**2a.** Codificación incorrecta: se advierte y se propone UTF-8.<br>**5a.** Doble envío de la importación: se rechaza por idempotencia. |
| **Relaciones** | `<<include>>` CU-03 (validación y duplicados por fila). No incluye CU-01: la importación no reproduce el formulario interactivo, comparte solo las reglas. |

---

#### CU-03 Validar cliente y detectar duplicados

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Subfunción |
| **Actor principal** | Sistema (invocado por CU-01 y CU-02) |
| **Requisitos cubiertos** | Heredados de CU-01 y CU-02 (RF5, RF6, RNF13) |
| **Interesados y objetivos** | **Empresa:** calidad de datos. **Operativo:** errores comprensibles. |
| **Precondiciones** | Datos de cliente recibidos desde el formulario o desde una fila de CSV. |
| **Disparador** | Invocación desde CU-01 o CU-02. |
| **Garantía mínima** | No se guarda nada; solo se devuelve el resultado de la validación. |
| **Garantía de éxito** | Resultado con errores, avisos y posibles coincidencias. |
| **Escenario principal** | 1. El sistema normaliza espacios y formato de los campos.<br>2. Valida emails y el formato del identificador fiscal según el país.<br>3. Distingue lo necesario para facturar de lo meramente conveniente.<br>4. Busca duplicados razonables por identificador fiscal y nombre.<br>5. Devuelve errores, avisos y coincidencias. |
| **Extensiones** | **2a.** País sin regla de validación: se acepta identificador libre con aviso.<br>**3a.** Falta un dato solo conveniente (p. ej. teléfono): aviso no bloqueante.<br>**4a.** Hay duplicados: se muestran; nunca se fusiona automáticamente. |
| **Relaciones** | Incluido por CU-01 y CU-02. |

---

### Suscripciones, tarifas y aprobaciones

#### CU-04 Crear suscripción

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Usuario Operativo |
| **Requisitos cubiertos** | RF12, RF13, RF14, RF17, RF19, RN2, RU5 |
| **Interesados y objetivos** | **Operativo:** alta correcta a la primera. **Empresa:** tarifa acordada inmutable. **Cliente:** factura coherente. |
| **Precondiciones** | Cliente con datos fiscales suficientes. |
| **Disparador** | El Operativo elige «Crear suscripción» en la ficha. |
| **Garantía mínima** | Sin suscripción parcial si falla. |
| **Garantía de éxito** | Suscripción creada con importe fijado, fecha de inicio y facturación programada. |
| **Escenario principal** | 1. El Operativo solicita crear la suscripción.<br>2. El sistema propone plan y tarifa según el tramo del cliente.<br>3. El Operativo ajusta conceptos, condiciones de pago, responsable y fecha de inicio.<br>4. Si el inicio no es día 1, el sistema calcula el prorrateo (CU-05).<br>5. El sistema muestra la vista previa de la primera factura y de las siguientes.<br>6. El Operativo confirma y el sistema programa la recurrencia. |
| **Extensiones** | **3a.** Precio o descuento fuera de tarifa: se solicita aprobación (CU-09) y la suscripción queda pendiente.<br>**3b.** Fecha de inicio futura: el cliente queda «pendiente de activación», sin facturas ni recordatorios.<br>**4a.** Acuerdo de empezar al mes siguiente sin prorrateo: excepción comercial explícita y aprobable.<br>**3c.** Documento firmado externo: se adjunta a la ficha comercial.<br>**1a.** Cliente sin datos fiscales: se bloquea con explicación. |
| **Relaciones** | `<<include>>` CU-05 (cuando aplique). Extendido por CU-09. |

---

#### CU-05 Calcular prorrateo

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Subfunción |
| **Actor principal** | Sistema (invocado por CU-04, CU-06 y CU-07) |
| **Requisitos cubiertos** | RF14, RNF2, RNF19 |
| **Interesados y objetivos** | **Empresa:** importe correcto y explicable. |
| **Precondiciones** | Fecha de alta, reactivación o baja distinta del día 1 o del último día del mes. |
| **Disparador** | Invocación desde CU-04, CU-06 o CU-07. |
| **Garantía mínima** | No se devuelve importe si el cálculo falla. |
| **Garantía de éxito** | Importe prorrateado con redondeo documentado. |
| **Escenario principal** | 1. El sistema obtiene los días naturales reales del mes.<br>2. Calcula los días de servicio del tramo (inclusive).<br>3. Divide la cuota entre los días del mes y multiplica por los días de servicio.<br>4. Aplica la regla de redondeo a dos decimales y devuelve el desglose. |
| **Extensiones** | **2a.** Alta el último día del mes: 1 día de servicio.<br>**3a.** Mes de 28, 29, 30 o 31 días: se usa siempre el real, nunca 30 fijo.<br>**4a.** Excepción «sin prorrateo»: devuelve cero y marca la excepción.<br>**4b.** Prorrateo en la baja (CU-07): suposición registrada, el cliente solo habló del alta; se aplica la misma regla y se confirma en la siguiente revisión. |
| **Relaciones** | Incluido por CU-04, CU-06 y CU-07. |

---

#### CU-06 Pausar y reactivar suscripción

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Usuario Operativo |
| **Requisitos cubiertos** | RF15, RF7, RF9, RF52, RN5 |
| **Interesados y objetivos** | **Operativo:** reflejar acuerdos temporales. **Cliente:** no recibir cuotas durante la pausa. |
| **Precondiciones** | Suscripción activa. |
| **Disparador** | El Operativo selecciona «Pausar suscripción». |
| **Garantía mínima** | La suscripción sigue activa si falla. |
| **Garantía de éxito** | Pausa registrada; sin mensualidades en el periodo; deuda anterior intacta. |
| **Escenario principal** | 1. El Operativo indica motivo, fecha de inicio y reactivación prevista.<br>2. El sistema muestra su efecto sobre las próximas facturas.<br>3. El Operativo confirma y el sistema marca la suscripción como pausada.<br>4. Llegada la fecha prevista, el sistema notifica al responsable.<br>5. El Operativo reactiva (con prorrateo si corresponde, CU-05) o prorroga la pausa. |
| **Extensiones** | **4a.** La política configurada exige confirmación: el sistema no reactiva por sí solo.<br>**1a.** Pausa sin fecha de fin: queda abierta con aviso periódico.<br>**3a.** Hay facturas antiguas pendientes: siguen su flujo de cobro (CU-16). |
| **Relaciones** | `<<include>>` CU-05 al reactivar a mitad de mes. |

---

#### CU-07 Registrar baja comercial

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Usuario Operativo |
| **Requisitos cubiertos** | RF16, RF7, RF42, RN5, RF73 |
| **Interesados y objetivos** | **Empresa:** cerrar la recurrencia sin perder el cobro. **Cliente:** no ser facturado tras la baja. |
| **Precondiciones** | Suscripción activa o pausada. |
| **Disparador** | Solicitud de baja manual o tarea creada desde la API (CU-33). |
| **Garantía mínima** | Sin cambios si falla. |
| **Garantía de éxito** | Recurrencia cerrada en la fecha efectiva; el recobro de la deuda anterior sigue vivo si existe. |
| **Escenario principal** | 1. El Operativo registra la solicitud con fecha efectiva.<br>2. El sistema consulta si hay facturas pendientes.<br>3. Sin deuda: cierra la recurrencia en la fecha efectiva.<br>4. El sistema pasa el cliente a «inactivo» (solo significa que no se le factura más) y registra el evento. |
| **Extensiones** | **2a.** Con deuda: se detienen las nuevas cuotas desde la fecha efectiva y la ficha muestra «con deuda pendiente», una situación derivada de las facturas, no del estado; CU-16 mantiene el recobro.<br>**1a.** Solicitud llegada por API: se crea una tarea; el Operativo la ejecuta y el sistema externo nunca cierra la recurrencia.<br>**1b.** Fecha efectiva futura: el cliente queda en «baja solicitada» hasta esa fecha.<br>**3a.** Mes de baja parcial: se calcula con la regla de CU-05 (suposición registrada). |
| **Relaciones** | `<<include>>` CU-05. Origen posible: tarea creada por CU-33 (disparador, sin relación formal). Relacionado con CU-16. |

---

#### CU-08 Ejecutar revisión anual de rango

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Administrador |
| **Requisitos cubiertos** | RF18, RF21, RF52, RF9, RN3, RN4 |
| **Interesados y objetivos** | **Administrador:** revisar sin errores masivos. **Empresa:** tarifa acorde a la facturación real. |
| **Precondiciones** | Existen tramos vigentes; llega la fecha de campaña. |
| **Disparador** | Inicio de campaña (automático o manual). |
| **Garantía mínima** | Ningún precio cambia sin aprobación. |
| **Garantía de éxito** | Nuevas tarifas aprobadas con fecha de efecto y registro anterior/nuevo. |
| **Escenario principal** | 1. El sistema lista los clientes a revisar.<br>2. El Administrador introduce o importa la facturación verificada con fuente y fecha.<br>3. El sistema propone el tramo correspondiente.<br>4. El Administrador envía la propuesta a aprobación (CU-09).<br>5. Aprobada, el sistema fija el nuevo precio con fecha de efecto. |
| **Extensiones** | **2a.** Dato no disponible: la revisión queda pendiente y genera tarea.<br>**5a.** Rechazada: se mantiene la tarifa.<br>**4a.** Dato de enero erróneo descubierto en junio: se corrige con traza; el sistema calcula el impacto pero no emite facturas retroactivas; la regularización es una acción explícita y aprobada.<br>**3a.** Cambio de varios clientes a la vez: acción masiva con recuento y confirmación. |
| **Relaciones** | `<<include>>` CU-09 (aprobación siempre obligatoria para este tipo, aunque el importe esté bajo el umbral). |

---

#### CU-09 Gestionar excepción comercial y aprobación

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Usuario Operativo (solicitante) y Administrador (aprobador); el Superadministrador aprueba lo que supera el umbral del Administrador |
| **Requisitos cubiertos** | RF19, RF21, RF22, RF23, RF52, RF69, RN4 |
| **Interesados y objetivos** | **Operativo:** cerrar un acuerdo. **Aprobador:** decidir con contexto. **Empresa:** control de descuentos. Se mantiene como un solo caso: solicitar y resolver son un único objetivo de negocio con dos roles. |
| **Precondiciones** | Cliente con suscripción o factura; el aprobador no es el solicitante cuando la regla lo exige. |
| **Disparador** | El Operativo propone un descuento, mes gratis o precio pactado, o llega una solicitud desde otro caso de uso. |
| **Garantía mínima** | Sin cambio de condiciones hasta la aprobación. |
| **Garantía de éxito** | Decisión registrada con quién, cuándo y comentario; efecto aplicado o restituida la condición estándar. |
| **Escenario principal** | 1. El Operativo indica tipo, importe, motivo y vigencia.<br>2. El sistema determina el nivel de aprobación según el umbral configurado.<br>3. El sistema crea la solicitud pendiente y explica por qué está pendiente.<br>4. El aprobador abre la bandeja y revisa cliente, importe anterior y nuevo, motivo y vigencia.<br>5. El aprobador aprueba o rechaza con comentario.<br>6. El sistema aplica el efecto, notifica al solicitante y audita. |
| **Extensiones** | **2a.** Por debajo del umbral y con permiso: se aplica sin aprobación y se audita (salvo cambios de tarifa por revisión anual, que siempre se aprueban).<br>**2b.** El solicitante es Superadministrador: aplica la decisión sin aprobación de otro, con motivo obligatorio y auditoría (decisión D-1).<br>**4a.** Importe sobre el umbral del Administrador: solo aprueba el Superadministrador.<br>**5a.** Intento de autoaprobación de un Administrador en importes sensibles: se bloquea.<br>**4b.** Aprobador ausente: otro Administrador reasigna o actúa el suplente, con registro.<br>**5b.** Rechazo: se restituye la condición estándar.<br>**6a.** Vigencia terminada: el descuento caduca y el siguiente lote ya no lo aplica. |
| **Relaciones** | Incluido por CU-08. `<<extend>>` de CU-04, CU-11 y CU-13 cuando hay importes manuales o sobre umbral. |

---

### Facturación y documentos

#### CU-10 Generar facturación mensual recurrente

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Resumen/proceso batch |
| **Actor principal** | Motor de Automatización |
| **Requisitos cubiertos** | RF24, RF25, RF28, RF41, RF74, RN6, RNF1, RNF9, RNF11 |
| **Interesados y objetivos** | **Empresa:** facturas correctas el día 1. **Operativo:** conocer las excepciones. **Cliente:** recibir su factura. |
| **Precondiciones** | Día 1 a las 00:00; suscripciones activas. |
| **Disparador** | Temporizador mensual. |
| **Garantía mínima** | Reejecutar el lote nunca duplica facturas ni números. |
| **Garantía de éxito** | Facturas emitidas y enviadas, resultado registrado y excepciones asignadas a su responsable. |
| **Escenario principal** | 1. El motor abre un registro de ejecución.<br>2. Recupera las suscripciones facturables del periodo.<br>3. Para cada una calcula importes, impuestos y descuentos vigentes.<br>4. Emite la factura (CU-12).<br>5. Envía la factura (CU-15).<br>6. Cierra la ejecución con recuento de procesadas, creadas y fallidas. |
| **Extensiones** | **3a.** Falta contacto de facturación: la factura se emite igualmente (decisión D-3), el envío queda como fallo de entrega con incidencia (CU-21) al responsable y el lote continúa.<br>**3b.** Cliente pausado: no se factura.<br>**3c.** Tarifa nueva aprobada ayer: se aplica la nueva.<br>**3d.** Baja efectiva hoy: no se genera la mensualidad.<br>**3e.** Descuento caducado anoche: no se aplica.<br>**3f.** Condiciones a 30 días: el vencimiento pactado prevalece.<br>**4a.** Error en un cliente: se registra y continúa con los demás.<br>**1a.** Caída del servidor: al reiniciar detecta lo no generado y reanuda sin duplicar.<br>**5a.** Fallo de envío: el envío falla pero la factura existe (CU-15). |
| **Relaciones** | `<<include>>` CU-12 y CU-15. |

---

#### CU-11 Emitir factura puntual

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Usuario Operativo |
| **Requisitos cubiertos** | RF26, RF20, RF29, RF31, RF36, RF40, RNF1, RU5 |
| **Interesados y objetivos** | **Operativo:** facturar un servicio extra. **Empresa:** importes correctos y aprobados. |
| **Precondiciones** | Cliente activo con datos fiscales. |
| **Disparador** | El Operativo elige «Nueva factura puntual». |
| **Garantía mínima** | El borrador se guarda automáticamente; nada se emite parcialmente. |
| **Garantía de éxito** | Factura emitida, numerada, con PDF y enviada, o borrador / pendiente de aprobación. |
| **Escenario principal** | 1. El Operativo selecciona cliente y añade líneas (concepto del catálogo o libre, cantidad, precio, descuento, impuesto).<br>2. El sistema calcula totales y muestra vista previa y advertencias.<br>3. El Operativo adjunta soporte si procede.<br>4. El Operativo pulsa «Emitir» (protegido contra doble clic).<br>5. El sistema emite la factura (CU-12).<br>6. El sistema envía la factura (CU-15). |
| **Extensiones** | **4a.** Guardar borrador: se almacena sin consumir número.<br>**4b.** Importe manual extraordinario o muy alto: pasa a «pendiente de aprobación» (CU-09) explicando por qué.<br>**4c.** Borrar un borrador nunca emitido: permitido con traza si procede; una factura emitida nunca se borra.<br>**2a.** Importe fuera del patrón del cliente: advertencia no bloqueante que exige reconfirmación.<br>**5a.** Falta un dato fiscal: se bloquea indicando cuál. |
| **Relaciones** | `<<include>>` CU-12 y CU-15. Extendido por CU-09. |

---

#### CU-12 Emitir factura (validar, numerar y generar PDF)

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Subfunción |
| **Actor principal** | Sistema (invocado por CU-10, CU-11 y CU-13) |
| **Requisitos cubiertos** | RF27, RF28, RF29, RF30, RF31, RNF1, RNF2, RNF10 |
| **Interesados y objetivos** | **Empresa:** documento legal consistente. |
| **Precondiciones** | Borrador completo y aprobado si procede. |
| **Disparador** | Invocación desde otro caso de uso. |
| **Garantía mínima** | Sin número consumido ni documento congelado si falla. |
| **Garantía de éxito** | Factura emitida, numerada, congelada y con PDF conservado. |
| **Escenario principal** | 1. El sistema valida datos fiscales y aprobaciones.<br>2. Asigna el número correlativo de la serie de forma atómica.<br>3. Genera y conserva el PDF con la plantilla vigente.<br>4. Marca la factura como emitida y registra evento y auditoría. |
| **Extensiones** | **1a.** Falta dato fiscal: bloquea y lo explica.<br>**2a.** Dos emisiones simultáneas: la numeración atómica evita números repetidos.<br>**3a.** Cambio de plantilla posterior: la factura conserva su PDF original.<br>**1b.** Periodo cerrado (CU-27): se rechaza salvo procedimiento excepcional auditado.<br>**4a.** Reintento tras fallo posterior (p. ej. envío): no se emite de nuevo ni cambia el número. |
| **Relaciones** | Incluido por CU-10, CU-11 y CU-13. |

---

#### CU-13 Emitir factura rectificativa

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Administrador |
| **Requisitos cubiertos** | RF32, RF31, RF28, RF21, RF69, RN4 (separados del antiguo CU-12; confirmar con la lista de requisitos) |
| **Interesados y objetivos** | **Administrador:** corregir sin editar. **Empresa:** trazabilidad legal. **Gestoría:** tratamiento contable correcto. |
| **Precondiciones** | Existe una factura emitida errónea o parcialmente devuelta. |
| **Disparador** | El Administrador elige «Rectificar» en la factura. |
| **Garantía mínima** | La factura original nunca se modifica. |
| **Garantía de éxito** | Rectificativa emitida, vinculada a la original y con el saldo recalculado. |
| **Escenario principal** | 1. El Administrador localiza la factura y elige rectificar, indicando motivo y alcance.<br>2. El sistema propone los datos a rectificar y muestra las consecuencias.<br>3. El Administrador confirma.<br>4. El sistema emite la rectificativa en su serie (CU-12), la vincula con la original y recalcula el saldo.<br>5. El sistema envía la rectificativa (CU-15). |
| **Extensiones** | **3a.** Importe sobre umbral: requiere aprobación (CU-09).<br>**4a.** Periodo cerrado: requiere procedimiento excepcional auditado.<br>**4b.** Nunca se marca como «pagada»: ese estado solo lo produce un cobro validado.<br>**4c.** El saldo pasa a favor del cliente: se refleja en el cálculo de saldo.<br>**1a.** La deuda es irrecuperable en lugar de errónea: se deriva a CU-14.<br>**1b.** Anulación cuando legalmente proceda: se resuelve con una rectificativa total (decisión de modelado). |
| **Relaciones** | `<<include>>` CU-12 y CU-15. Extendido por CU-09. |

---

#### CU-14 Declarar deuda incobrable

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Superadministrador |
| **Requisitos cubiertos** | RF33, RF69, RN4, RN5 (separados del antiguo CU-12; confirmar con la lista de requisitos) |
| **Interesados y objetivos** | **Superadministrador:** cerrar una deuda irrecuperable con control. **Empresa:** trazabilidad. **Gestoría:** tratamiento contable correcto. |
| **Precondiciones** | Existe una factura emitida vencida y sin cobro. |
| **Disparador** | El Superadministrador elige «Declarar incobrable» en la factura. |
| **Garantía mínima** | La factura original no se modifica y el estado no pasa a «pagada». |
| **Garantía de éxito** | Saldo cancelado como incobrable, con motivo, fecha, autor y traza; recordatorios detenidos. |
| **Escenario principal** | 1. El Superadministrador localiza la factura y elige declararla incobrable, indicando motivo.<br>2. El sistema muestra las consecuencias y avisa de la irreversibilidad.<br>3. El Superadministrador confirma.<br>4. El sistema cancela el saldo con estado propio «incobrable», detiene los recordatorios (CU-16) y audita. |
| **Extensiones** | **3a.** Decisión D-1: el Superadministrador no requiere aprobación de otro; compensa con motivo obligatorio, aviso de irreversibilidad y auditoría.<br>**4a.** Periodo cerrado: requiere procedimiento excepcional auditado.<br>**4b.** Contablemente puede requerir otro documento: se avisa y se sugiere CU-13.<br>**1a.** Corresponde rectificar y no declarar incobrable: se deriva a CU-13. |
| **Relaciones** | Sin relaciones formales. Cancela el seguimiento de CU-16 sobre esa factura. |

---

### Envío, cobro y conciliación

#### CU-15 Enviar factura al cliente y gestionar fallos

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Motor de Automatización / Usuario Operativo (reenvío manual) |
| **Requisitos cubiertos** | RF41, RF42, RF3, RF51, RF53, RNF22, RN6 |
| **Interesados y objetivos** | **Cliente:** recibir su factura. **Operativo:** saber si falló. **Empresa:** no duplicar facturas al reenviar. Estados: «enviada» = el proveedor aceptó la petición; «entregada» = el proveedor lo confirma; la lectura nunca se inventa. |
| **Precondiciones** | Factura emitida. |
| **Disparador** | Fin de la emisión o reenvío manual. |
| **Garantía mínima** | Un fallo de envío nunca invalida la factura ni crea otra factura u otro número. |
| **Garantía de éxito** | Envío registrado con resultado, destinatarios y plantilla. |
| **Escenario principal** | 1. El sistema elige canal y destinatarios (contacto de facturación).<br>2. Envía mediante el proveedor (simulado) y registra intento, fecha, destinatarios y plantilla, indicando si es envío original o reintento.<br>3. Si el proveedor acepta el envío, marca la factura como «enviada».<br>4. Si el proveedor confirma la entrega, marca el envío como «entregada». |
| **Extensiones** | **3a.** Rebote o fallo: no se marca como enviada ni entregada, se crea incidencia de entrega (CU-21) y se avisa al responsable.<br>**3b.** Hay otro contacto de facturación: se reintenta con él según configuración.<br>**3c.** El Operativo corrige el email y reenvía el mismo documento; se distingue original y reintento.<br>**3d.** Proveedor caído: la factura existe y el envío queda en cola reintentable.<br>**4a.** El proveedor no informa de entrega: queda solo «enviada»; no se asume lectura.<br>**1a.** Canal WhatsApp sin consentimiento: se usa solo email.<br>**1b.** No hay destinatario: no se envía y se crea incidencia de entrega. |
| **Relaciones** | Incluido por CU-10, CU-11 y CU-13. Extendido por CU-21 en fallos. |

---

#### CU-16 Gestionar recordatorios e impagos

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Resumen/proceso batch |
| **Actor principal** | Motor de Automatización |
| **Requisitos cubiertos** | RF43, RF44, RF45, RF55, RF57, RN5 |
| **Interesados y objetivos** | **Empresa:** cobrar a tiempo. **Cliente:** no recibir avisos indebidos. |
| **Precondiciones** | Facturas emitidas pendientes de cobro. |
| **Disparador** | Temporizador diario. |
| **Garantía mínima** | Ningún aviso sale a un cliente con seguimiento pausado. |
| **Garantía de éxito** | Avisos enviados según cadencia y tareas de recobro generadas. |
| **Escenario principal** | 1. El motor calcula para cada factura su vencimiento aplicable (pactado o general).<br>2. Envía los recordatorios que correspondan (preventivo, vencimiento).<br>3. Si vence sin cobro, abre tarea en la bandeja y activa el flujo de impago periódico.<br>4. Registra los envíos. |
| **Extensiones** | **1a.** Cliente con 30 días pactados: no se envía «impago» al quinto día.<br>**2a.** Fecha en fin de semana o festivo: se traslada al siguiente día hábil.<br>**2b.** Seguimiento pausado (estado fijado por CU-17): se omite el aviso.<br>**4a.** Pago detectado (CU-20) o deuda declarada incobrable (CU-14): se cancelan avisos y tareas.<br>**3a.** Cliente dado de baja con deuda: el flujo continúa. |
| **Relaciones** | Cancelado por CU-14 y CU-20. Consulta el estado de pausa que fija CU-17 (sin relación formal: CU-17 es un objetivo propio de una persona). |

---

#### CU-17 Pausar seguimiento por disputa

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Usuario Operativo |
| **Requisitos cubiertos** | RF45, RF53, RF9 |
| **Interesados y objetivos** | **Operativo:** dar margen al cliente. **Empresa:** no perder la deuda de vista. |
| **Precondiciones** | Factura vencida o disputada. |
| **Disparador** | El Operativo marca «Disputa / seguimiento pausado». |
| **Garantía mínima** | Los avisos siguen activos si falla. |
| **Garantía de éxito** | Avisos detenidos con fecha de revisión y tarea programada. |
| **Escenario principal** | 1. El Operativo indica motivo y fecha de revisión.<br>2. El sistema suspende los avisos de esa factura.<br>3. El sistema mantiene la factura visible como pendiente en la bandeja.<br>4. Llegada la fecha crea tarea de revisión. |
| **Extensiones** | **1a.** Sin permiso de pausa: se rechaza.<br>**4a.** Fecha de revisión vencida sin acción: se escala a Administrador.<br>**2a.** Se abre una incidencia vinculada (CU-21). |
| **Relaciones** | Sin relación formal con CU-16 (modifica el estado que éste consulta). Relacionado con CU-21. |

---

#### CU-18 Procesar evento de pago de la pasarela

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Pasarela de Pagos (Stripe simulada) |
| **Requisitos cubiertos** | RF46, RF47, RF48, RNF1, RNF22, RN6 |
| **Interesados y objetivos** | **Empresa:** reflejar cobros correctos. **Cliente:** aviso cancelado al pagar. |
| **Precondiciones** | Facturas pendientes; integración configurada. |
| **Disparador** | Llega un evento o se consulta un pago. |
| **Garantía mínima** | Un evento nunca se contabiliza dos veces. |
| **Garantía de éxito** | Cobro conciliado o enviado a revisión. |
| **Escenario principal** | 1. El sistema recibe el evento.<br>2. Comprueba que su identificador no se haya procesado.<br>3. Registra el movimiento y lo envía a conciliación (CU-20).<br>4. Registra el resultado. |
| **Extensiones** | **2a.** Evento duplicado: se descarta sin duplicar saldos.<br>**3a.** Sin referencia clara: queda pendiente de conciliación manual.<br>**1a.** Integración caída: se reintenta la consulta y se alerta. |
| **Relaciones** | `<<include>>` CU-20. |

---

#### CU-19 Importar movimientos bancarios

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Administrador / Usuario Operativo. Secundario: Fuente bancaria (Revolut/banco/extracto) |
| **Requisitos cubiertos** | RF47, RF48 (de los antiguos CU-16 y CU-17; confirmar con la lista de requisitos) |
| **Interesados y objetivos** | **Operativo:** cargar cobros sin teclearlos. **Empresa:** añadir nuevas fuentes sin cambiar el modelo. |
| **Precondiciones** | Permiso de conciliación; fichero o fuente bancaria disponible. |
| **Disparador** | El usuario sube un extracto o el sistema consulta una fuente configurada. |
| **Garantía mínima** | Si falla, no se crean movimientos parciales. |
| **Garantía de éxito** | Movimientos registrados con importe, fecha, moneda, concepto y referencia, y enviados a conciliación; operación registrada con resumen. |
| **Escenario principal** | 1. El usuario sube el fichero (o el sistema consulta la fuente).<br>2. El sistema muestra una previsualización y pide mapear columnas.<br>3. El sistema valida cada fila.<br>4. Registra los movimientos nuevos y descarta los ya cargados.<br>5. Cada movimiento pasa a conciliación (CU-20).<br>6. Registra la operación con importados, descartados, rechazados y motivos. |
| **Extensiones** | **4a.** Movimiento ya importado (misma referencia e importe): no se duplica.<br>**2a.** Codificación incorrecta: se advierte y se propone UTF-8.<br>**3a.** Fila sin importe o fecha: se rechaza con motivo.<br>**1a.** Fuente nueva: se incorpora como adaptador sin cambiar el modelo.<br>**5a.** Importación larga: se ejecuta en segundo plano con estado visible. |
| **Relaciones** | `<<include>>` CU-20. |

---

#### CU-20 Conciliar movimiento de pago

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Usuario Operativo (y Administrador para revertir) |
| **Requisitos cubiertos** | RF47, RF48, RF9, RF69, RNF10, RNF22 |
| **Interesados y objetivos** | **Operativo:** asociar cobros sin error. **Empresa:** saber qué movimiento justificó cada pago. Cobro validado = coincidencia unívoca de importe exacto, moneda y referencia, o conciliación manual de una persona. |
| **Precondiciones** | Movimiento recibido (Stripe, banco o extracto). |
| **Disparador** | Movimiento nuevo (CU-18 o CU-19) o revisión de la bandeja de pendientes. |
| **Garantía mínima** | Ninguna factura se marca pagada sin cobro validado; el movimiento original nunca se borra. |
| **Garantía de éxito** | Factura cobrada con el movimiento que la justifica. |
| **Escenario principal** | 1. El sistema compara importe, fecha, moneda, concepto y referencia con las facturas pendientes.<br>2. Con coincidencia unívoca de importe exacto concilia o propone según política.<br>3. Con cobro validado marca la factura como cobrada y cancela avisos y tareas.<br>4. Registra qué regla o usuario concilió. |
| **Extensiones** | **1a.** Dos facturas del mismo importe: no se concilia automáticamente; pasa a revisión manual.<br>**2a.** Importe menor: el movimiento queda asociado, la factura no se marca pagada y queda con «diferencia pendiente».<br>**2b.** Importe mayor: se señala el exceso para revisión.<br>**1b.** Fuente nueva (Revolut, banco): se incorpora sin cambiar el modelo.<br>**2c.** El Operativo concilia a mano eligiendo la factura.<br>**4a.** Reversión: el Administrador deshace la conciliación con motivo; la factura vuelve a pendiente, se reactivan los avisos y se registra quién la hizo, quién la revirtió y por qué; el movimiento original se conserva.<br>**4b.** Periodo cerrado: reversión solo por procedimiento excepcional. |
| **Relaciones** | Incluido por CU-18 y CU-19. |

---

### Incidencias y trabajo diario

#### CU-21 Gestionar incidencia de facturación

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Usuario Operativo |
| **Requisitos cubiertos** | RF53, RF54, RF45, RF52 |
| **Interesados y objetivos** | **Operativo:** resolver problemas con responsable claro. **Empresa:** no tratarlo como ticketing de cliente. |
| **Precondiciones** | Cliente existente. |
| **Disparador** | El Operativo abre una incidencia (o la crea el sistema). |
| **Garantía mínima** | La incidencia no se pierde si falla. |
| **Garantía de éxito** | Incidencia resuelta con resolución registrada. |
| **Escenario principal** | 1. El Operativo crea la incidencia (tipo, cliente, factura opcional).<br>2. Asigna responsable y prioridad.<br>3. Añade comentarios, menciones y adjuntos.<br>4. Resuelve y registra la resolución. |
| **Extensiones** | **2a.** Afecta al cobro: se pausa el seguimiento (CU-17).<br>**3a.** Nota restringida: solo visible para administración.<br>**3b.** Mención a otro usuario: se genera notificación.<br>**1a.** Creada por el sistema tras un fallo de envío (CU-15): llega al responsable del cliente. |
| **Relaciones** | `<<extend>>` CU-15 en fallos de envío. Relacionado con CU-17. |

---

#### CU-22 Trabajar la bandeja diaria y las notificaciones

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Usuario Operativo |
| **Requisitos cubiertos** | RF57, RF11, RF52, RF55, RF56, RF61, RU1, RU2, RU8 |
| **Interesados y objetivos** | **Operativo:** resolver su día con pocos clics. **Superadministrador:** no recibir cientos de correos. Estados distintos: leída, atendida y resuelta. |
| **Precondiciones** | Usuario autenticado con clientes asignados. |
| **Disparador** | Inicio de sesión, apertura de la bandeja o llegada de una notificación. |
| **Garantía mínima** | No se muestran elementos fuera de su ámbito. |
| **Garantía de éxito** | Tareas resueltas y notificaciones leídas o atendidas. |
| **Escenario principal** | 1. El sistema muestra vencimientos próximos, vencidas, pagos por conciliar, fallos de entrega, aprobaciones, revisiones y tareas asignadas.<br>2. El usuario filtra y selecciona un elemento.<br>3. El sistema abre el elemento y la acción relevante.<br>4. El usuario resuelve y vuelve a la bandeja.<br>5. El usuario marca las notificaciones como leídas y atiende las que requieren acción. |
| **Extensiones** | **1a.** Cliente delegado temporalmente: aparece mientras dure la delegación.<br>**3a.** Intento de acceso a un elemento ajeno por URL: el servidor rechaza.<br>**5a.** Notificación informativa (pago recibido): basta con leerla.<br>**5b.** Deuda grave: abrirla no la resuelve; sigue pendiente de atender.<br>**1b.** Categoría configurada como resumen diario: el motor la agrega y la envía en el digest.<br>**5c.** Preferencias personales (canal, resumen) permitidas; las reglas corporativas de recordatorio no se pueden desactivar. |
| **Relaciones** | Punto de entrada a CU-09, CU-17, CU-20 y CU-21 (sin relaciones formales). |

---

#### CU-23 Buscar y consultar la ficha e historial del cliente

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Usuario Operativo |
| **Requisitos cubiertos** | RF61, RF11, RF9 |
| **Interesados y objetivos** | **Operativo:** encontrar clientes y facturas en pocos segundos. **Empresa:** entender la historia de cada cliente. |
| **Precondiciones** | Usuario autenticado. |
| **Disparador** | El usuario escribe en la búsqueda global o abre un favorito. |
| **Garantía mínima** | Nunca se muestran ni se abren elementos fuera de su ámbito. |
| **Garantía de éxito** | Cliente o factura localizados y línea temporal consultada. |
| **Escenario principal** | 1. El usuario escribe parte del nombre comercial, razón social, NIF, contacto, número de factura o correo.<br>2. El sistema muestra resultados limitados a su ámbito.<br>3. El usuario filtra (ubicación, sector, estado) o usa una vista guardada.<br>4. Abre la ficha del cliente.<br>5. Consulta la línea temporal: alta, responsable, tarifa, facturas, pagos, incidencias, pausas, baja y reactivación. |
| **Extensiones** | **2a.** Sin resultados: se sugiere crear cliente (CU-01).<br>**4a.** Copia la URL de un elemento ajeno: el servidor rechaza.<br>**1a.** Favoritos y recientes: son un atajo, nunca un mecanismo de autorización. |
| **Relaciones** | Sin relaciones formales; puede llevar a CU-01, CU-04 o CU-11. |

---

### Informes, exportación y cierres

#### CU-24 Consultar dashboard y explicar KPI

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Administrador |
| **Requisitos cubiertos** | RF58, RF39, RU4, RF9, RF64 |
| **Interesados y objetivos** | **Administrador:** entender el negocio. **Empresa:** cifras reconciliables. |
| **Precondiciones** | Usuario con permiso de dashboard global. |
| **Disparador** | Abre el dashboard. |
| **Garantía mínima** | Ninguna cifra sin detalle verificable. |
| **Garantía de éxito** | Indicadores visibles y trazables hasta documentos. |
| **Escenario principal** | 1. El sistema muestra facturado, cobrado, pendiente, vencido, tiempo medio de cobro, recurrente esperado y mayores deudores.<br>2. El Administrador selecciona una cifra.<br>3. El sistema muestra los documentos que la componen. |
| **Extensiones** | **1a.** Usuario operativo: no ve el dashboard global.<br>**2a.** Distinción entre saldo y facturación: se explica en el detalle.<br>**1b.** Objetivos configurados: se muestra el progreso (ampliación).<br>**1c.** Importe en otra moneda: se distingue importe original de importe convertido para reporting (ampliación). |
| **Relaciones** | Relacionado con CU-25. |

---

#### CU-25 Exportar informes

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Administrador / Usuario Operativo |
| **Requisitos cubiertos** | RF59, RF60, RF62, RF69, RNF11, RNF13 |
| **Interesados y objetivos** | **Usuario:** obtener datos. **Empresa:** controlar exportaciones sensibles. |
| **Precondiciones** | Usuario autenticado con permiso. |
| **Disparador** | Elige «Informes». |
| **Garantía mínima** | Nunca se exporta fuera del ámbito del usuario. |
| **Garantía de éxito** | Informe generado y exportación auditada. |
| **Escenario principal** | 1. El usuario fija filtros (periodo, cliente, responsable, país, sector, estado, tipo).<br>2. El sistema genera el resultado en segundo plano, limitado al ámbito del usuario.<br>3. El usuario descarga CSV (con codificación correcta) o PDF.<br>4. El sistema audita la exportación con fecha, filtros y usuario. |
| **Extensiones** | **2a.** Informe grande: aviso al terminar.<br>**2b.** Operativo con cinco clientes: solo exporta esos cinco.<br>**4a.** Exportación masiva (p. ej. 2.000 clientes): queda destacada en la auditoría.<br>**1a.** Informe programado (ampliación): solo a destinatarios autorizados.<br>**3a.** Los informes generados se conservan un tiempo configurable, distinto de los documentos contables. |
| **Relaciones** | Relacionado con CU-24. |

---

#### CU-26 Generar el paquete para la gestoría

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Administrador |
| **Requisitos cubiertos** | RF67, RF37, RF69, RNF11, RNF13 (confirmar con la lista de requisitos) |
| **Interesados y objetivos** | **Gestoría:** recibir datos contables. **Empresa:** versiones distinguibles. La gestoría no tiene usuario en esta versión. |
| **Precondiciones** | Usuario con permiso; periodo mensual o trimestral elegido. |
| **Disparador** | Elige «Paquete gestoría». |
| **Garantía mínima** | Nunca se exporta fuera del ámbito del usuario. |
| **Garantía de éxito** | Paquete versionado generado y su generación registrada. |
| **Escenario principal** | 1. El Administrador elige el periodo y el formato.<br>2. El sistema genera en segundo plano el listado de facturas emitidas, rectificativas, cobros y PDFs en un ZIP organizado.<br>3. El Administrador descarga el paquete.<br>4. El sistema registra cuándo se generó, qué periodo cubría y la versión. |
| **Extensiones** | **2a.** Periodo no cerrado: advierte.<br>**3a.** Regeneración tras una rectificativa: nueva versión distinguible; la anterior se conserva.<br>**4a.** Exportación auditada con usuario, fecha y filtros. |
| **Relaciones** | Relacionado con CU-27. |

---

#### CU-27 Cerrar y reabrir periodo (ampliación referenciada)

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Administrador (cierra) / Superadministrador (reabre) |
| **Requisitos cubiertos** | Por confirmar en la lista de requisitos (ampliaciones RF34, RF35, RF37, RF75) |
| **Interesados y objetivos** | **Empresa:** no alterar periodos revisados. **Gestoría:** cifras estables. |
| **Precondiciones** | Periodo vencido y revisado. |
| **Disparador** | El Administrador elige «Cerrar periodo». |
| **Garantía mínima** | Si falla, el periodo sigue abierto. |
| **Garantía de éxito** | Periodo cerrado con autor y fecha, y cambios económicos ordinarios bloqueados. |
| **Escenario principal** | 1. El Administrador elige el periodo y revisa el resumen (emitidas, pendientes de aprobación, conciliaciones por revisar).<br>2. El sistema avisa de los pendientes.<br>3. El Administrador confirma el cierre.<br>4. El sistema marca el periodo como cerrado y audita. |
| **Extensiones** | **2a.** Hay borradores o aprobaciones pendientes: avisa; bloquea o no según la política.<br>**4a.** Cambios ordinarios posteriores en elementos económicos: se rechazan; solo procede el procedimiento excepcional auditado.<br>**5a.** Reabrir: solo el Superadministrador, con motivo, y queda auditado. |
| **Relaciones** | Restricción previa de CU-12, CU-13, CU-14 y CU-20 (sin relaciones include/extend). |

---

### Usuarios, configuración e integración

#### CU-28 Iniciar sesión y recuperar el acceso

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Cualquier usuario interno (Operativo, Administrador, Superadministrador) |
| **Requisitos cubiertos** | RF66, RNF3, RU3 (confirmar con la lista de requisitos) |
| **Interesados y objetivos** | **Usuario:** acceder rápido. **Empresa:** sesiones seguras y sin compartir contraseñas. |
| **Precondiciones** | Cuenta creada por invitación (no hay autorregistro). |
| **Disparador** | El usuario abre la aplicación. |
| **Garantía mínima** | Un acceso fallido no revela si el email existe. |
| **Garantía de éxito** | Sesión abierta con expiración por inactividad, o acceso recuperado. |
| **Escenario principal** | 1. El usuario introduce email corporativo y contraseña.<br>2. El sistema valida dominio autorizado, cuenta activa y credenciales.<br>3. Abre la sesión con expiración por inactividad.<br>4. Muestra la bandeja diaria (CU-22). |
| **Extensiones** | **2a.** Intentos fallidos repetidos: bloqueo temporal básico.<br>**2b.** Usuario desactivado: se rechaza.<br>**1a.** Olvida la contraseña: solicita recuperación y recibe un enlace que caduca.<br>**3a.** Sesión inactiva: se cierra sola.<br>**3b.** Un administrador desactiva al usuario: sus sesiones activas se invalidan.<br>**3c.** Doble factor (futuro): el diseño no lo impide. |
| **Relaciones** | Precondición de los demás casos; sin relaciones formales. |

---

#### CU-29 Gestionar usuarios y roles

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Superadministrador / Administrador |
| **Requisitos cubiertos** | RF65, RF67, RF68, RF63, RF69, RU3, RNF3 (del antiguo CU-22) |
| **Interesados y objetivos** | **Superadministrador:** control de accesos. **Empresa:** trazabilidad y modelo de autorización coherente, no ifs dispersos. |
| **Precondiciones** | Permiso de gestión de usuarios. |
| **Disparador** | Abre «Usuarios». |
| **Garantía mínima** | Sin autorregistro. |
| **Garantía de éxito** | Usuario activo con rol, o desactivado sin perder su rastro. |
| **Escenario principal** | 1. El Administrador invita o crea un usuario con email corporativo y rol.<br>2. El sistema envía la invitación y el usuario activa su cuenta.<br>3. El Administrador ajusta rol y permisos.<br>4. Al desactivar a un usuario, el sistema invalida sus sesiones y conserva el rastro de sus acciones.<br>5. El sistema comprueba si deja clientes sin responsable y pide reasignarlos (CU-30). |
| **Extensiones** | **1a.** Email fuera del dominio autorizado: se rechaza.<br>**1b.** Un Administrador crea usuarios pero no Superadministradores.<br>**3a.** Un Operativo no puede darse acceso a otro cliente.<br>**4a.** Se conserva el rastro de sus acciones y el histórico indica quién hizo cada cosa. |
| **Relaciones** | Relacionado con CU-30 y CU-28. |

---

#### CU-30 Asignar y delegar clientes

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Administrador |
| **Requisitos cubiertos** | RF65, RF11, RF69 (del antiguo CU-22; confirmar con la lista de requisitos) |
| **Interesados y objetivos** | **Administrador:** repartir el trabajo. **Empresa:** sin compartir contraseñas y sin clientes huérfanos. |
| **Precondiciones** | Permiso de asignación. |
| **Disparador** | El Administrador elige «Asignar» o «Delegar» sobre uno o varios clientes. |
| **Garantía mínima** | Nunca queda un cliente sin responsable. |
| **Garantía de éxito** | Asignación o delegación aplicada, tareas transferidas y cambio auditado. |
| **Escenario principal** | 1. El Administrador elige clientes y nuevo responsable (o delegado).<br>2. El sistema muestra recuento y consecuencias.<br>3. El Administrador confirma.<br>4. El sistema transfiere las tareas pendientes, conserva la autoría histórica y registra el cambio. |
| **Extensiones** | **1a.** Delegación temporal con fecha de fin: la asignación principal no se pierde y la delegación caduca sola.<br>**2a.** Reasignación masiva: recuento previo, confirmación y resultado con éxitos y errores.<br>**4a.** Aprobaciones pendientes del ausente: se reasignan o actúa el suplente, con registro (CU-09). |
| **Relaciones** | Relacionado con CU-29. |

---

#### CU-31 Configurar parámetros del sistema

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Superadministrador |
| **Requisitos cubiertos** | RF17, RF70, RF49, RF50 |
| **Interesados y objetivos** | **Superadministrador:** cambiar reglas sin tocar código. **Empresa:** configuración validada, auditada y con efecto solo hacia delante. |
| **Precondiciones** | Permiso de configuración. |
| **Disparador** | Abre «Configuración». |
| **Garantía mínima** | Un cambio inválido no se aplica. |
| **Garantía de éxito** | Parámetro cambiado con valor anterior y nuevo, autor y fecha, sin reescribir lo emitido. |
| **Escenario principal** | 1. El Superadministrador elige el área: tarifas y rangos, series, plazos y cadencia de recordatorios, umbrales de aprobación, impuestos y monedas, plantillas y logo, calendario laboral y festivos, catálogo de conceptos, alertas, canales y datos fiscales del emisor.<br>2. Modifica el valor.<br>3. El sistema valida el cambio.<br>4. El sistema muestra una previsualización o simulación (email de prueba, calendario de recordatorios de un cliente de ejemplo).<br>5. El Superadministrador confirma.<br>6. El sistema aplica el cambio hacia delante y audita. |
| **Extensiones** | **3a.** Valor inválido: se explica qué corregir.<br>**6a.** Cambio de tarifa: no reescribe facturas históricas.<br>**6b.** Cambio de plantilla o logo: las facturas emitidas conservan su PDF original.<br>**6c.** Cambio de vencimiento por defecto: afecta solo a nuevas facturas o condiciones.<br>**1a.** Plantillas y catálogo pueden delegarse a quien tenga permiso. |
| **Relaciones** | Sin relaciones formales. Sus parámetros los usan CU-04, CU-08, CU-10 y CU-16. |

---

#### CU-32 Consultar la auditoría

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Superadministrador |
| **Requisitos cubiertos** | RF69, RF74, RNF5 (confirmar con la lista de requisitos) |
| **Interesados y objetivos** | **Superadministrador:** saber quién hizo qué. **Empresa:** nadie reescribe el pasado. |
| **Precondiciones** | Permiso de auditoría. |
| **Disparador** | Abre «Auditoría». |
| **Garantía mínima** | La auditoría no es editable por nadie. |
| **Garantía de éxito** | Acciones localizadas con quién, qué, cuándo y valores anterior y nuevo. |
| **Escenario principal** | 1. El Superadministrador abre la auditoría.<br>2. Filtra por usuario, cliente, factura, tipo de acción y periodo.<br>3. Consulta el detalle de cada acción.<br>4. Exporta el resultado si lo necesita; la exportación queda auditada. |
| **Extensiones** | **2a.** Exportaciones masivas: aparecen destacadas.<br>**3a.** Purga de datos personales por normativa: se anonimiza sin destruir la trazabilidad (requisito, fuera de este caso).<br>**1a.** Usuario sin permiso: se rechaza. |
| **Relaciones** | Sin relaciones formales. |

---

#### CU-33 Atender solicitudes externas por API

| Campo | Detalle |
| :--- | :--- |
| **Nivel** | Objetivo de usuario |
| **Actor principal** | Sistema Cliente (Brain / API) |
| **Requisitos cubiertos** | RF72, RF73, RNF3, RNF22 |
| **Interesados y objetivos** | **Sistema externo:** consultar sin saltarse reglas. |
| **Precondiciones** | Petición HTTPS con token válido. |
| **Disparador** | Llegada de una petición. |
| **Garantía mínima** | Nunca se modifica la recurrencia directamente. |
| **Garantía de éxito** | Respuesta estructurada o tarea creada. |
| **Escenario principal** | 1. El sistema valida token, origen y límite de tasa.<br>2. Procesos la consulta (deuda, cliente o factura).<br>3. Responde con el detalle mínimo autorizado y registra la llamada. |
| **Extensiones** | **1a.** Token inválido: 401.<br>**1b.** Límite excedido: 429.<br>**2a.** Solicitud de baja: se crea una tarea para el Operativo (CU-07).<br>**2b.** Datos fuera de permiso: se omiten. |
| **Relaciones** | Origen posible de la tarea que ejecuta CU-07 (sin relación formal: otro actor y otra sesión). |

---
