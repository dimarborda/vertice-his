# InterSystems TrakCare · Catálogo de casos de uso e integraciones

Proyecto: Vértice HIS · Consulta: 2026-10-27

> Inventario funcional del referente, elaborado como análisis externo a
> partir de información pública de un HIS unificado tipo TrakCare. No es
> documentación oficial de InterSystems. Código `CU-MM-NN`: MM = módulo,
> NN = consecutivo.

## 1. Actores

| Sigla | Actor | Alcance |
|---|---|---|
| SIS | Sistema | Procesos automáticos de la plataforma (reglas, mensajería, alertas) |
| ADM | Admisionista | Registro, citas, admisión, camas, egreso administrativo |
| MED | Médico | Documentación, diagnóstico, órdenes, prescripción, firma |
| ENF | Enfermería | Triage, signos vitales, cuidados, administración de medicamentos |
| FAR | Farmacéutico | Validación, dispensación, inventario |
| LAB | Laboratorio | Muestras y resultados |
| IMG | Imágenes | Estudios e informes radiológicos |
| QUI | Coordinación quirúrgica | Programación de quirófanos y registro quirúrgico |
| FAC | Facturador | Cargos, facturas, pagadores |
| DIR | Directivo / calidad | Indicadores y reportes |
| PAC | Paciente | Portal |
| TIC | Administrador TI / integración | Configuración, interfaces, apps |
| EXT | Sistema externo / app SMART | Consumidor o emisor de datos estándar |

## 2. Módulos

| Módulo | Denominación | Casos |
|---|---|---|
| M01 | Administración del paciente (PAS/ADT) | 9 |
| M02 | Agenda y citas | 7 |
| M03 | Historia clínica electrónica | 9 |
| M04 | Órdenes y prescripción | 7 |
| M05 | Urgencias | 6 |
| M06 | Hospitalización y enfermería | 8 |
| M07 | Cirugía | 6 |
| M08 | Farmacia | 6 |
| M09 | Laboratorio | 5 |
| M10 | Imágenes | 5 |
| M11 | Facturación y ciclo de ingresos | 7 |
| M12 | Interoperabilidad | 8 |
| M13 | Portal del paciente | 5 |
| M14 | Analítica y reportes | 4 |
| M15 | IA y asistente | 4 |
| | **Total** | **96** |

## 3. Catálogo

### M01 · Administración del paciente (PAS/ADT)

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-01-01 | Buscar un paciente por identificador, nombre o fecha de nacimiento | ADM | Paciente encontrado o confirmado como inexistente |
| CU-01-02 | Registrar un paciente nuevo con datos demográficos | ADM | Registro único del paciente creado |
| CU-01-03 | Registrar la cobertura o aseguradora del paciente | ADM | Cobertura vigente asociada al paciente |
| CU-01-04 | Detectar posibles duplicados al registrar | SIS | Candidatos a duplicado presentados antes de guardar |
| CU-01-05 | Fusionar dos registros duplicados | ADM | Un solo registro con la historia de ambos y traza de la fusión |
| CU-01-06 | Admitir a un paciente a hospitalización | ADM | Episodio hospitalario abierto con cama asignada |
| CU-01-07 | Trasladar a un paciente entre servicios o camas | ADM | Ubicación actualizada con historial de traslados |
| CU-01-08 | Registrar el egreso administrativo | ADM | Episodio cerrado y cama liberada |
| CU-01-09 | Registrar alertas administrativas del paciente | ADM | Alerta visible en el banner del paciente |

### M02 · Agenda y citas

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-02-01 | Configurar la agenda de un profesional o recurso | TIC | Agenda con horarios y tipos de cita disponibles |
| CU-02-02 | Asignar una cita | ADM | Cita reservada en estado programada |
| CU-02-03 | Reprogramar o cancelar una cita | ADM | Cita movida o cancelada con motivo |
| CU-02-04 | Gestionar la lista de espera | ADM | Paciente priorizado en lista y citado al liberarse un cupo |
| CU-02-05 | Registrar la llegada del paciente a la cita | ADM | Cita en estado llegado y episodio ambulatorio abierto |
| CU-02-06 | Enviar recordatorios de cita | SIS | Recordatorio enviado y registrado |
| CU-02-07 | Registrar la inasistencia | SIS | Cita marcada como no asistida |

### M03 · Historia clínica electrónica

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-03-01 | Consultar la línea de tiempo clínica del paciente | MED | Episodios, notas, resultados y medicación visibles en orden cronológico |
| CU-03-02 | Documentar una nota clínica con plantilla | MED | Nota guardada como borrador |
| CU-03-03 | Firmar una nota clínica | MED | Nota firmada e inalterable |
| CU-03-04 | Agregar una nota aclaratoria a una nota firmada | MED | Adenda vinculada sin modificar el original |
| CU-03-05 | Registrar diagnósticos codificados | MED | Problema activo con código estándar |
| CU-03-06 | Registrar alergias e intolerancias | MED | Alergia activa que dispara alertas en prescripción |
| CU-03-07 | Registrar signos vitales | ENF | Observación con fecha, valor y responsable |
| CU-03-08 | Solicitar una interconsulta | MED | Interconsulta pendiente en la bandeja del especialista |
| CU-03-09 | Elaborar el resumen de egreso | MED | Resumen firmado y disponible para el paciente y terceros |

### M04 · Órdenes y prescripción

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-04-01 | Prescribir un medicamento | MED | Prescripción activa enviada a farmacia |
| CU-04-02 | Validar dosis, alergias e interacciones al prescribir | SIS | Alerta mostrada o prescripción aceptada sin alertas |
| CU-04-03 | Ordenar exámenes de laboratorio | MED | Orden de laboratorio pendiente de muestra |
| CU-04-04 | Ordenar estudios de imágenes | MED | Orden de imágenes pendiente de programación |
| CU-04-05 | Aplicar un conjunto de órdenes por protocolo | MED | Varias órdenes creadas en un solo paso |
| CU-04-06 | Suspender o modificar una orden | MED | Orden suspendida o nueva versión con motivo |
| CU-04-07 | Seguir el estado de una orden hasta su resultado | MED | Estado actual de la orden visible |

### M05 · Urgencias

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-05-01 | Registrar la llegada a urgencias | ADM | Episodio de urgencias abierto |
| CU-05-02 | Clasificar al paciente en triage | ENF | Nivel de triage asignado con hora |
| CU-05-03 | Ver el tablero de urgencias | MED | Pacientes por estado, nivel y tiempo de espera |
| CU-05-04 | Asignar un médico tratante | MED | Paciente asociado a un responsable |
| CU-05-05 | Decidir el destino (alta, observación, hospitalización) | MED | Destino registrado y flujo siguiente iniciado |
| CU-05-06 | Alertar tiempos de espera excedidos | SIS | Alerta visible para el coordinador |

### M06 · Hospitalización y enfermería

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-06-01 | Consultar el mapa de camas | ADM | Estado de cada cama visible |
| CU-06-02 | Cambiar el estado de una cama (aseo, bloqueo) | ENF | Cama con estado actualizado |
| CU-06-03 | Registrar la ronda médica | MED | Evolución diaria firmada |
| CU-06-04 | Definir el plan de cuidados de enfermería | ENF | Plan con intervenciones programadas |
| CU-06-05 | Administrar un medicamento con verificación | ENF | Administración registrada con hora y responsable |
| CU-06-06 | Registrar balance de líquidos | ENF | Balance calculado por turno |
| CU-06-07 | Aplicar escalas de valoración | ENF | Puntaje calculado y riesgo marcado |
| CU-06-08 | Hacer la entrega de turno | ENF | Resumen de turno registrado |

### M07 · Cirugía

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-07-01 | Programar un procedimiento quirúrgico | QUI | Quirófano, equipo y horario reservados |
| CU-07-02 | Completar la lista de verificación de cirugía segura | ENF | Lista completa por fases |
| CU-07-03 | Registrar el acto anestésico | MED | Registro anestésico firmado |
| CU-07-04 | Registrar la descripción quirúrgica | MED | Descripción firmada |
| CU-07-05 | Registrar insumos consumidos | ENF | Consumo asociado a cargos |
| CU-07-06 | Consultar el tablero de quirófanos | QUI | Estado de cada sala en tiempo real |

### M08 · Farmacia

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-08-01 | Validar una prescripción | FAR | Prescripción validada o devuelta con observación |
| CU-08-02 | Dispensar en dosis unitaria | FAR | Dispensación registrada y stock descontado |
| CU-08-03 | Registrar una devolución | FAR | Stock reintegrado y cargo anulado |
| CU-08-04 | Controlar lotes y vencimientos | FAR | Lotes próximos a vencer señalados |
| CU-08-05 | Controlar medicamentos de control especial | FAR | Movimiento registrado con doble firma |
| CU-08-06 | Reabastecer una bodega | FAR | Pedido generado y recibido |

### M09 · Laboratorio

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-09-01 | Recibir una orden de laboratorio | LAB | Orden en la lista de trabajo |
| CU-09-02 | Registrar la toma de muestra | ENF | Muestra etiquetada y con hora |
| CU-09-03 | Publicar resultados | LAB | Resultado visible en la historia |
| CU-09-04 | Notificar un valor crítico | SIS | Notificación entregada y confirmada por el médico |
| CU-09-05 | Corregir un resultado publicado | LAB | Nueva versión con motivo y aviso al solicitante |

### M10 · Imágenes

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-10-01 | Programar un estudio | IMG | Estudio agendado |
| CU-10-02 | Registrar la realización del estudio | IMG | Estudio realizado y enlazado al PACS |
| CU-10-03 | Publicar el informe radiológico | IMG | Informe firmado en la historia |
| CU-10-04 | Abrir las imágenes desde la historia | MED | Visor abierto en el estudio correcto |
| CU-10-05 | Notificar un hallazgo crítico | IMG | Notificación entregada y confirmada |

### M11 · Facturación y ciclo de ingresos

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-11-01 | Capturar cargos automáticamente desde la atención | SIS | Cargos generados por cada servicio prestado |
| CU-11-02 | Revisar la cuenta de un episodio | FAC | Cuenta consolidada sin cargos faltantes |
| CU-11-03 | Aplicar tarifas según contrato del pagador | SIS | Valores calculados por contrato |
| CU-11-04 | Emitir la factura al pagador | FAC | Factura emitida |
| CU-11-05 | Registrar una objeción del pagador | FAC | Objeción registrada con valor y motivo |
| CU-11-06 | Responder una objeción | FAC | Respuesta enviada y estado actualizado |
| CU-11-07 | Registrar pagos y conciliar | FAC | Cartera actualizada |

### M12 · Interoperabilidad

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-12-01 | Enviar eventos ADT a sistemas externos | SIS | Mensaje ADT entregado y confirmado |
| CU-12-02 | Recibir resultados por HL7 v2 | EXT | Resultado incorporado a la historia |
| CU-12-03 | Exponer datos del paciente por FHIR | EXT | Recursos FHIR devueltos según permisos |
| CU-12-04 | Lanzar una app SMART desde la historia | MED | App abierta con contexto de paciente y usuario |
| CU-12-05 | Registrar una app SMART y sus permisos | TIC | App autorizada con scopes definidos |
| CU-12-06 | Monitorear la cola de mensajes y reintentos | TIC | Mensajes fallidos visibles y reprocesables |
| CU-12-07 | Mapear terminologías locales a estándares | TIC | Código local asociado a código estándar |
| CU-12-08 | Compartir un resumen con una red regional | SIS | Documento disponible en el repositorio regional |

### M13 · Portal del paciente

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-13-01 | Consultar citas propias | PAC | Citas futuras y pasadas visibles |
| CU-13-02 | Solicitar o cancelar una cita | PAC | Solicitud registrada |
| CU-13-03 | Consultar resultados liberados | PAC | Resultados visibles |
| CU-13-04 | Descargar un resumen de atención | PAC | Documento descargado |
| CU-13-05 | Actualizar datos de contacto | PAC | Datos actualizados con traza |

### M14 · Analítica y reportes

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-14-01 | Consultar ocupación y estancia | DIR | Indicadores por servicio |
| CU-14-02 | Consultar oportunidad de atención | DIR | Tiempos por servicio y periodo |
| CU-14-03 | Generar reportes regulatorios | DIR | Reporte exportado |
| CU-14-04 | Auditar accesos a una historia | DIR | Lista de accesos con usuario, fecha y motivo |

### M15 · IA y asistente

| Código | Caso de uso | Actor | Resultado esperado |
|---|---|---|---|
| CU-15-01 | Buscar en la historia en lenguaje natural | MED | Datos relevantes del paciente presentados |
| CU-15-02 | Resumir la historia de un paciente | MED | Resumen presentado para lectura |
| CU-15-03 | Consultar datos por API para analítica avanzada | EXT | Datos FHIR entregados al entorno analítico |
| CU-15-04 | Detectar pacientes en riesgo con reglas o modelos | SIS | Alerta generada en la lista de trabajo |

## 4. Integraciones

| Categoría | Plataformas | Para qué |
|---|---|---|
| Laboratorio (LIS) | Sistemas LIS del mercado | Órdenes y resultados por HL7 v2 ORM/ORU |
| Imágenes (RIS/PACS) | RIS y PACS (DICOM / DICOMweb) | Programación, estudios, informes, visor |
| Equipos médicos | Monitores, bombas, analizadores | Signos vitales y datos de dispositivos |
| Aseguradoras / pagadores | Portales y APIs de aseguradoras | Elegibilidad, autorizaciones, reclamaciones |
| Redes regionales / nacionales | Repositorios de historia compartida (en Colombia: IHCE / RDA) | Intercambio de resúmenes |
| ERP | SAP, ERP locales | Contabilidad, compras, nómina |
| Apps de terceros | Apps SMART on FHIR | Funciones especializadas sobre los datos |
| Mensajería | SMS, correo | Recordatorios y notificaciones |

### Eventos y mensajes

| Estándar | Mensaje / evento | Uso |
|---|---|---|
| HL7 v2 | ADT (A01 admisión, A02 traslado, A03 egreso, A04 registro, A08 actualización, A40 fusión) | Sincronizar pacientes y episodios con sistemas externos |
| HL7 v2 | ORM / OML | Enviar órdenes a LIS/RIS |
| HL7 v2 | ORU | Recibir resultados e informes |
| HL7 v2 | SIU | Sincronizar citas |
| HL7 v2 | DFT | Enviar cargos a facturación externa |
| FHIR R4 | Subscriptions (sobre `Encounter`, `ServiceRequest`, `DiagnosticReport`…) | Notificar cambios a apps y servicios suscritos |
| SMART on FHIR | Lanzamiento EHR / standalone | Abrir apps con contexto |

## 5. Frentes de complejidad para una implementación análoga

| Frente | Complejidad | Recomendación |
|---|---|---|
| HL7 v2 sobre MLLP (TCP persistente) | Alta | MLLP no está disponible en funciones serverless: instalar un conector local (Vértice Edge, binario Go) en la red del hospital que hable MLLP y reenvíe por HTTPS con cola y reintentos. |
| Motor de búsqueda FHIR sobre `jsonb` | Alta | Extraer parámetros de búsqueda a una tabla de índices al escribir; empezar por los parámetros que usan el RDA y la interfaz; no prometer búsqueda FHIR completa en E0. |
| Terminologías (CIE-10/11, CUPS, CUM, LOINC, SNOMED CT) | Media | Catálogos versionados con mapeos locales; licencias de SNOMED CT a verificar; servicio de terminología propio simple (`$lookup`, `$validate-code`). |
| RDA hacia la IHCE (Res. 1888 de 2025) | Media-alta | Generar el bundle FHIR desde el `Encounter` cerrado; outbox con reintentos y acuses; seguir el anexo técnico vigente y sus cambios de 2026. |
| RIPS JSON y FEV (Res. 2275 de 2023) | Alta | Proveedor tecnológico autorizado por la DIAN para la FEV; validación previa de RIPS; derivar ambos de los cargos, nunca digitarlos. |
| Firma electrónica de notas | Media | Firma con identidad fuerte (MFA) y sello de integridad (hash) por versión; evaluar firma digital certificada donde se exija. |
| eMAR fuera de línea | Media-alta | Tableta con cola local cifrada y sincronización idempotente; la verificación por código funciona sin red y se concilia después. |
| Unificación de duplicados del paciente | Media | Algoritmo de coincidencia probabilística con aprobación humana y fusión reversible con traza. |
| Rendimiento con alto volumen clínico | Media | Proyecciones relacionales para listas de trabajo; `jsonb` solo para el recurso completo. |
| Seguridad y auditoría de accesos | Alta | Bitácora de lectura y escritura; acceso de emergencia con justificación; SMART scopes por app y agente. |
