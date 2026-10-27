# Vértice HIS · Casos de uso por etapas

Versión 1 · 2026-10-27 · Fuente: [04-conceptual.md](04-conceptual.md) y [00-decisiones.md](00-decisiones.md)

## Cómo leer este documento

- ID `UC-MOD-nn`; los IDs no se reutilizan (un caso descartado deja su número vacante).
- Prioridad: 🟥 alta (sin ella la etapa no cierra) · 🟨 media · 🟩 baja.
- Puntos Fibonacci: 1, 2, 3, 5, 8, 13. Nada supera 13.
- Estado: ⬜ pendiente · 🔄 en construcción · ✅ terminado y verificado.
- Cada etapa indica la **edición** que habilita (*Esencial*, *Hospital*, *Red*).
- Un mismo prefijo puede aparecer en varias etapas (p. ej. `INT`, `IAC`, `ADM`): la etapa la define el caso, no el módulo.

## Prefijos

| Prefijo | Módulo | Etapa |
|---|---|---|
| FND | M0 · Cimientos: identidad, sedes, roles, catálogos, auditoría | E0 |
| PAC | M1 · Índice maestro de pacientes | E1 |
| ADM | M2 · Admisiones y agenda | E1, E4 |
| HCE | M3 · Historia clínica electrónica | E1, E4 |
| ORD | M4 · Órdenes clínicas | E1 |
| URG | M5 · Urgencias | E4 |
| HOS | M6 · Hospitalización y enfermería | E4 |
| CIR | M7 · Cirugía | E5 |
| FAR | M8 · Farmacia | E4 |
| APO | M9 · Apoyo diagnóstico | E5 |
| FAC | M10 · Facturación, RIPS y glosas | E3 |
| INT | M11 · Interoperabilidad (FHIR, RDA, HL7 v2, SMART, CDS Hooks) | E0, E1, E4, E5, E6 |
| IAC | M12 · IA clínica nativa | E2, E3, E4, E6 |
| POR | M13 · Portal del paciente | E6 |
| ANA | M14 · Analítica y calidad | E6 |
| RED | M15 · Red multisede | E7 |

## Actores

| Actor | Qué hace | Qué NO puede hacer |
|---|---|---|
| Médico / profesional clínico | Documenta, diagnostica, ordena, firma notas y aprueba o rechaza borradores de la IA | Ver pacientes fuera de su sede/servicio sin relación de atención (salvo "romper el vidrio"); editar una nota firmada; facturar; cambiar catálogos |
| Enfermería | Triage, signos vitales, plan de cuidados, administración de medicamentos (eMAR), entrega de turno | Prescribir; firmar notas médicas; administrar sin verificar paciente y medicamento por código |
| Admisiones | Registra y busca pacientes, coberturas, citas, admisiones, egresos administrativos | Leer notas clínicas, diagnósticos o resultados (solo datos administrativos); fusionar pacientes sin aprobación |
| Facturación y auditoría de cuentas | Contratos, cargos, FEV, RIPS, radicación, glosas | Modificar documentación clínica; borrar cargos o facturas (solo notas crédito/débito y ajustes) |
| Farmacia | Valida prescripciones, dispensa, controla inventario y control especial | Cambiar una prescripción sin devolverla al prescriptor; dispensar sin validación |
| Laboratorio / Imágenes | Recibe órdenes, publica resultados e informes | Ver la historia completa más allá de lo necesario para la orden; modificar un resultado publicado sin versión corregida |
| Dirección / calidad | Consulta indicadores, aprueba protocolos, reglas de alerta y exportaciones | Ver datos identificados de pacientes en tableros agregados; editar registros clínicos |
| Administrador TI / interoperabilidad | Configura organización, sedes, membresías, catálogos, conectores, apps SMART y agentes IA | Leer contenido clínico sin rol asistencial; borrar o alterar la bitácora; otorgarse a sí mismo roles clínicos sin un segundo administrador |
| Paciente (portal) | Consulta su historia, citas, resultados y RDA; autoriza o revoca apps de terceros | Ver datos de otros pacientes; modificar su historia clínica (solo solicitar corrección) |
| Sistema / motor de reglas | Ejecuta procesos automáticos versionados: outbox, reintentos, vencimientos, cargos automáticos | Actuar sin regla vigente; actuar sin dejar traza en bitácora |
| Agente IA | Redacta borradores, resume, sugiere códigos, detecta riesgos y responde consultas sobre la historia | Firmar, ordenar, dispensar, facturar o enviar nada sin aprobación humana; leer fuera de los scopes concedidos; escribir directamente en `recursos_fhir` como versión vigente |
| App SMART externa | Lee/escribe recursos FHIR según los scopes que autorizó el profesional o el paciente | Usar scopes no concedidos; seguir accediendo tras la revocación o expiración del token |
| IHCE / MinSalud | Recibe el RDA y devuelve acuse o rechazo | Modificar datos de Vértice (solo responde) |
| EPS / aseguradora | Verifica derechos, recibe facturas y RIPS, emite glosas | Acceder a la historia clínica fuera de los soportes radicados |
| Conector Vértice Edge | Traduce HL7 v2 (MLLP) y DICOM en la red del hospital y los relaya por HTTPS | Guardar datos clínicos de forma persistente fuera de su cola temporal cifrada; aceptar conexiones de equipos no registrados |

## Mapa de etapas

| Etapa | Propósito | Módulos | Edición | Duración | Criterio de salida |
|---|---|---|---|---|---|
| E0 | Cimientos | FND, INT | Todas | 3 semanas | Una persona entra con MFA, pertenece a una sede/servicio con un rol, toda lectura y escritura clínica queda en bitácora y existe un almacén FHIR versionado; la app está publicada en Vercel |
| E1 | Ciclo ambulatorio mínimo + RDA (MVP) | PAC, ADM, HCE, ORD, INT | Esencial | 6 semanas | Un paciente real se registra, se agenda, se atiende, se le ordena, se cierra el encuentro y su RDA llega a la IHCE con acuse |
| E2 | IA clínica nativa | IAC | Esencial / Hospital | 4 semanas | Un médico atiende con escriba ambiental, aprueba el borrador y queda `Provenance`; un agente vía MCP consulta solo lo autorizado |
| E3 | Facturación, RIPS y glosas | FAC, IAC | Esencial | 5 semanas | Una atención de E1 se factura con FEV aceptada por la DIAN y RIPS validado, se radica y una glosa se responde dentro de Vértice |
| E4 | Urgencias, hospitalización y farmacia | URG, ADM, HOS, HCE, FAR, INT, IAC | Hospital | 7 semanas | Un paciente pasa triage → observación → hospitalización con cama, eMAR y dispensación por dosis unitaria, egresa con epicrisis y su RDA de hospitalización se envía |
| E5 | Apoyo diagnóstico, cirugía e integraciones HL7 v2 | APO, CIR, INT | Hospital | 6 semanas | Una orden de laboratorio viaja al LIS por HL7 v2 vía Vértice Edge y el resultado vuelve a la historia; una cirugía se programa, se verifica y se documenta |
| E6 | Portal del paciente, analítica y ecosistema SMART | POR, ANA, INT, IAC | Esencial / Hospital | 5 semanas | Una app SMART de terceros se lanza desde la historia con scopes limitados, una alerta CDS Hooks aparece al ordenar, el paciente ve su RDA en el portal y dirección ve indicadores de calidad |
| E7 | Red multisede | RED | Red | 4 semanas | Dos instituciones comparten índice maestro y un paciente referido de una a otra llega con su historia visible bajo consentimiento |

## E0 · Cimientos

**Edición:** todas (base común).

**Criterio de salida:** una persona entra con MFA a la URL pública de Vercel, pertenece a una sede y un servicio con un rol, puede crear y leer un recurso FHIR versionado, y toda lectura y escritura de ese recurso aparece en la bitácora con actor, fecha y motivo.

| ID | Caso de uso | Actor | Prioridad | Puntos | Estado |
|---|---|---|---|---|---|
| UC-FND-01 | Iniciar sesión con MFA | Usuario | 🟥 | 3 | ⬜ |
| UC-FND-02 | Configurar organización, sedes y servicios con código REPS | Administrador TI | 🟥 | 5 | ⬜ |
| UC-FND-03 | Configurar ubicaciones (camas, consultorios, quirófanos) | Administrador TI | 🟨 | 3 | ⬜ |
| UC-FND-04 | Asignar membresías por sede, servicio y rol | Administrador TI | 🟥 | 5 | ⬜ |
| UC-FND-05 | Bloquear la sesión por inactividad en estación compartida | Sistema | 🟥 | 2 | ⬜ |
| UC-FND-06 | Acceder en emergencia ("romper el vidrio") | Profesional clínico | 🟥 | 5 | ⬜ |
| UC-FND-07 | Revisar accesos de emergencia | Dirección / calidad | 🟨 | 3 | ⬜ |
| UC-FND-08 | Cargar y versionar catálogos CIE-10, CUPS, CUM, LOINC, DIVIPOLA y EPS | Administrador TI | 🟥 | 5 | ⬜ |
| UC-FND-09 | Registrar en bitácora toda lectura y escritura clínica (`AuditEvent`) | Sistema | 🟥 | 5 | ⬜ |
| UC-FND-10 | Consultar la bitácora con filtros | Dirección / calidad | 🟨 | 3 | ⬜ |
| UC-FND-11 | Completar perfil profesional (ReTHUS y firma) | Profesional clínico | 🟨 | 3 | ⬜ |
| UC-INT-01 | Guardar y versionar un recurso FHIR R4 | Sistema | 🟥 | 8 | ⬜ |
| UC-INT-02 | Buscar recursos FHIR por parámetros básicos | Sistema | 🟥 | 5 | ⬜ |
| UC-INT-03 | Desplegar frontend Angular y servicios Go en Vercel con preview por rama | Administrador TI | 🟥 | 5 | ⬜ |
| UC-INT-04 | Publicar eventos con outbox transaccional | Sistema | 🟨 | 5 | ⬜ |

### UC-FND-01 · Iniciar sesión con MFA
- **Actor:** Usuario (cualquier rol interno).
- **Precondición:** el administrador creó la cuenta o envió la invitación.
- **Disparador:** la persona abre Vértice.
- **Flujo principal:** 1) Ingresa correo y contraseña. 2) Confirma el segundo factor (TOTP). 3) Elige sede/servicio si tiene varias membresías. 4) Entra a su inicio por rol.
- **Alternativas:** segundo factor no configurado → se obliga a configurarlo antes de entrar; cinco intentos fallidos → bloqueo temporal y aviso al administrador.
- **Reglas de negocio:** sin MFA no hay acceso a datos clínicos; el token emitido por el servicio `auth` lleva sede, servicio y rol activos.
- **Criterio de aceptación:** una persona invitada entra desde la URL pública con MFA y ve su nombre y su sede activa; sin el segundo factor no entra.

### UC-FND-02 · Configurar organización, sedes y servicios con código REPS
- **Actor:** Administrador TI.
- **Precondición:** organización creada al contratar la edición.
- **Disparador:** implantación o apertura de una sede/servicio.
- **Flujo principal:** 1) Registra NIT y código de habilitación. 2) Crea sedes con código REPS y DIVIPOLA. 3) Crea servicios habilitados con complejidad. 4) Guarda.
- **Alternativas:** código REPS repetido → se rechaza; servicio no habilitado en la edición contratada → se avisa.
- **Reglas de negocio:** las sedes y servicios se publican también como `Organization`/`Location`/`HealthcareService` FHIR; los cambios quedan en bitácora.
- **Criterio de aceptación:** la sede y sus servicios aparecen en la configuración y como recursos FHIR consultables.

### UC-FND-04 · Asignar membresías por sede, servicio y rol
- **Actor:** Administrador TI.
- **Precondición:** la persona tiene cuenta y la sede/servicio existe.
- **Disparador:** ingreso, traslado o cambio de responsabilidades.
- **Flujo principal:** 1) Busca a la persona. 2) Asigna sede, servicio, rol y vigencia. 3) Confirma con motivo. 4) Queda en bitácora.
- **Alternativas:** asignarse a sí mismo un rol clínico → exige aprobación de un segundo administrador; quitar el último administrador → se impide.
- **Reglas de negocio:** los permisos salen de la membresía `(persona, sede, servicio, rol)`, no de la persona; membresía vencida no da acceso.
- **Criterio de aceptación:** el cambio aparece en la bitácora y los permisos cambian en el siguiente request sin reiniciar sesión.

### UC-FND-05 · Bloquear la sesión por inactividad en estación compartida
- **Actor:** Sistema.
- **Precondición:** sesión abierta en una estación marcada como compartida.
- **Disparador:** inactividad mayor al umbral configurado (por defecto 5 minutos).
- **Flujo principal:** 1) Detecta inactividad. 2) Guarda borradores localmente cifrados. 3) Oculta la pantalla clínica. 4) Pide reautenticación.
- **Alternativas:** otra persona se autentica → se cierra la sesión anterior y sus borradores quedan solo para su dueño.
- **Reglas de negocio:** ningún dato clínico permanece visible tras el bloqueo.
- **Criterio de aceptación:** tras el umbral, la pantalla se bloquea y el borrador se recupera al volver a entrar la misma persona.

### UC-FND-06 · Acceder en emergencia ("romper el vidrio")
- **Actor:** Profesional clínico.
- **Precondición:** el paciente no está en su sede/servicio ni tiene relación de atención con él.
- **Disparador:** necesita ver la historia por una urgencia.
- **Flujo principal:** 1) Busca al paciente. 2) Elige "acceso de emergencia". 3) Escribe la justificación y confirma. 4) Obtiene acceso temporal (por defecto 4 horas).
- **Alternativas:** sin justificación → no se concede.
- **Reglas de negocio:** cada acceso de emergencia queda en bitácora con justificación y entra a la cola de revisión de calidad (UC-FND-07).
- **Criterio de aceptación:** el acceso funciona, expira solo y aparece en la cola de revisión con la justificación escrita.

### UC-FND-08 · Cargar y versionar catálogos CIE-10, CUPS, CUM, LOINC, DIVIPOLA y EPS
- **Actor:** Administrador TI.
- **Precondición:** archivo oficial del catálogo disponible.
- **Disparador:** implantación o actualización normativa.
- **Flujo principal:** 1) Sube el archivo. 2) El sistema valida formato y diferencias contra la versión vigente. 3) Muestra altas, bajas y cambios. 4) El administrador publica la nueva versión con fecha de vigencia.
- **Alternativas:** formato inválido → se rechaza con el detalle de las filas.
- **Reglas de negocio:** las versiones anteriores se conservan; un código dado de baja no se ofrece para nuevos registros pero sigue legible en los históricos; los catálogos se exponen como `CodeSystem`/`ValueSet`.
- **Criterio de aceptación:** un médico encuentra un código CIE-10 de la versión publicada y un código dado de baja ya no aparece en el buscador.

### UC-FND-09 · Registrar en bitácora toda lectura y escritura clínica (`AuditEvent`)
- **Actor:** Sistema.
- **Precondición:** cualquier operación sobre datos clínicos o de configuración sensible.
- **Disparador:** lectura, creación, actualización, exportación o envío.
- **Flujo principal:** 1) El middleware Go registra actor (humano, sistema o IA), acción, recurso, paciente, sede, IP y motivo. 2) Inserta en `bitacora`. 3) Publica el `AuditEvent` FHIR.
- **Alternativas:** falla la escritura de bitácora → la operación se rechaza (no hay operación sin traza).
- **Reglas de negocio:** la bitácora solo admite inserción; nadie, ni el administrador, puede modificarla o borrarla.
- **Criterio de aceptación:** al abrir la historia de un paciente aparece una entrada de lectura con el nombre de quien la abrió; un intento de borrar la entrada falla.

### UC-INT-01 · Guardar y versionar un recurso FHIR R4
- **Actor:** Sistema.
- **Precondición:** servicio `fhir` desplegado y base Postgres conectada.
- **Disparador:** un módulo crea o actualiza un recurso clínico.
- **Flujo principal:** 1) Valida el recurso contra el perfil R4 (y perfiles colombianos cuando existan). 2) Asigna `id` y `meta.versionId`. 3) Guarda la versión en `recursos_fhir` y extrae índices de búsqueda. 4) Registra bitácora y evento de salida.
- **Alternativas:** recurso inválido → `OperationOutcome` con el detalle; conflicto de versión (`If-Match`) → 412.
- **Reglas de negocio:** nunca se sobrescribe una versión; `_history` devuelve todas; FHIR es la verdad clínica.
- **Criterio de aceptación:** crear y actualizar un `Patient` deja dos versiones consultables en `_history` y la bitácora muestra ambas escrituras.

### UC-INT-02 · Buscar recursos FHIR por parámetros básicos
- **Actor:** Sistema.
- **Precondición:** recursos con índices extraídos.
- **Disparador:** un módulo, una app o la IA buscan recursos.
- **Flujo principal:** 1) Recibe la búsqueda (`patient`, `identifier`, `code`, `date`, `status`, `_lastUpdated`). 2) Aplica permisos del token. 3) Devuelve un `Bundle` paginado.
- **Alternativas:** parámetro no soportado → se ignora y se informa en `OperationOutcome`.
- **Reglas de negocio:** el filtrado por permisos ocurre antes de paginar; cada búsqueda queda en bitácora.
- **Criterio de aceptación:** buscar las `Condition` de un paciente devuelve solo las suyas y solo si quien busca tiene acceso.

### UC-INT-03 · Desplegar frontend Angular y servicios Go en Vercel con preview por rama
- **Actor:** Administrador TI.
- **Precondición:** cuentas de GitHub, Vercel y Supabase creadas por el dueño.
- **Disparador:** inicio del proyecto.
- **Flujo principal:** 1) Repositorio en GitHub. 2) Proyecto web (Angular estático) y proyecto API (Vercel Services con contenedores Go). 3) Variables de entorno en Vercel. 4) Push a una rama → URL de preview; `main` → producción con aprobación.
- **Alternativas:** falla el build → no se publica y se muestra el error.
- **Reglas de negocio:** ningún secreto en el repositorio; `/api/*` del web reescribe al proyecto API.
- **Criterio de aceptación:** el dueño abre la URL de Vercel, ve la pantalla de inicio de sesión y `/api/health` responde desde el contenedor Go.

## E1 · Ciclo ambulatorio mínimo + RDA

**Edición:** Esencial (MVP).

**Criterio de salida:** en la URL de preview, un paciente real del piloto se registra con su EPS, recibe una cita, es atendido con nota firmada, diagnóstico y órdenes, se cierra el encuentro y su RDA de consulta externa llega a la IHCE con acuse visible en el monitor de envíos, sin hojas de cálculo paralelas.

| ID | Caso de uso | Actor | Prioridad | Puntos | Estado |
|---|---|---|---|---|---|
| UC-PAC-01 | Registrar paciente con la canasta mínima de datos | Admisiones | 🟥 | 5 | ⬜ |
| UC-PAC-02 | Buscar paciente por documento, nombre fonético y fecha de nacimiento | Admisiones | 🟥 | 3 | ⬜ |
| UC-PAC-03 | Registrar cobertura (EPS, régimen, plan) y verificar derechos | Admisiones | 🟥 | 5 | ⬜ |
| UC-PAC-04 | Registrar consentimiento de tratamiento de datos | Admisiones | 🟥 | 3 | ⬜ |
| UC-PAC-05 | Detectar y fusionar pacientes duplicados con aprobación | Admisiones | 🟨 | 8 | ⬜ |
| UC-PAC-06 | Registrar acudientes y contactos | Admisiones | 🟩 | 2 | ⬜ |
| UC-ADM-01 | Configurar agendas por profesional y servicio | Admisiones | 🟥 | 5 | ⬜ |
| UC-ADM-02 | Asignar una cita | Admisiones | 🟥 | 3 | ⬜ |
| UC-ADM-03 | Reprogramar, cancelar y gestionar lista de espera | Admisiones | 🟨 | 3 | ⬜ |
| UC-ADM-04 | Admitir al paciente ambulatorio (abrir el encuentro) | Admisiones | 🟥 | 3 | ⬜ |
| UC-ADM-05 | Enviar recordatorios y confirmar citas | Sistema | 🟩 | 3 | ⬜ |
| UC-ADM-06 | Registrar la autorización de servicios de la EPS | Admisiones | 🟨 | 5 | ⬜ |
| UC-ADM-07 | Cerrar el encuentro ambulatorio | Profesional clínico | 🟥 | 2 | ⬜ |
| UC-HCE-01 | Documentar la nota de consulta con plantilla | Profesional clínico | 🟥 | 8 | ⬜ |
| UC-HCE-02 | Registrar diagnósticos CIE-10 | Profesional clínico | 🟥 | 3 | ⬜ |
| UC-HCE-03 | Registrar alergias y antecedentes | Profesional clínico | 🟥 | 3 | ⬜ |
| UC-HCE-04 | Registrar signos vitales con valores calculados | Enfermería | 🟥 | 3 | ⬜ |
| UC-HCE-05 | Firmar electrónicamente la nota | Profesional clínico | 🟥 | 5 | ⬜ |
| UC-HCE-06 | Agregar una nota aclaratoria a una nota firmada | Profesional clínico | 🟥 | 3 | ⬜ |
| UC-HCE-07 | Consultar la línea de tiempo clínica del paciente | Profesional clínico | 🟥 | 5 | ⬜ |
| UC-HCE-08 | Configurar plantillas de nota por especialidad | Administrador TI | 🟨 | 5 | ⬜ |
| UC-HCE-09 | Exportar la historia en PDF para el paciente | Profesional clínico | 🟨 | 3 | ⬜ |
| UC-ORD-01 | Prescribir medicamentos con validación de alergias y dosis | Profesional clínico | 🟥 | 8 | ⬜ |
| UC-ORD-02 | Ordenar laboratorio, imágenes y procedimientos (CUPS) | Profesional clínico | 🟥 | 5 | ⬜ |
| UC-ORD-03 | Solicitar una interconsulta | Profesional clínico | 🟨 | 3 | ⬜ |
| UC-ORD-04 | Usar conjuntos de órdenes por protocolo | Profesional clínico | 🟩 | 5 | ⬜ |
| UC-ORD-05 | Seguir el estado de cada orden | Profesional clínico | 🟨 | 3 | ⬜ |
| UC-INT-05 | Generar el RDA de paciente y de consulta externa | Sistema | 🟥 | 8 | ⬜ |
| UC-INT-06 | Enviar el RDA a la IHCE con reintentos e idempotencia | Sistema | 🟥 | 5 | ⬜ |
| UC-INT-07 | Monitorear los envíos de RDA | Administrador TI | 🟨 | 3 | ⬜ |

### UC-PAC-01 · Registrar paciente con la canasta mínima de datos
- **Actor:** Admisiones.
- **Precondición:** la búsqueda (UC-PAC-02) no encontró al paciente.
- **Disparador:** llega un paciente nuevo.
- **Flujo principal:** 1) Captura tipo y número de documento, nombres, sexo, fecha de nacimiento, residencia (DIVIPOLA), pertenencia étnica y demás datos de la canasta mínima. 2) El sistema busca posibles duplicados. 3) Guarda como `Patient` FHIR y proyección `pacientes_mpi`.
- **Alternativas:** posible duplicado → muestra candidatos y pide confirmar que es otra persona; paciente sin documento (NN, urgencia) → identificador temporal que luego se concilia.
- **Reglas de negocio:** tipo y número de documento únicos por paciente; la edad se calcula, no se digita.
- **Criterio de aceptación:** el paciente aparece en la búsqueda y su `Patient` es consultable por FHIR con los campos de la canasta mínima.

### UC-PAC-02 · Buscar paciente por documento, nombre fonético y fecha de nacimiento
- **Actor:** Admisiones (y cualquier rol con acceso).
- **Precondición:** sesión activa.
- **Disparador:** inicio de cualquier atención o búsqueda desde la paleta de comandos.
- **Flujo principal:** 1) Escribe documento o nombre. 2) El sistema busca exacto y fonético. 3) Muestra resultados con documento, edad y última atención. 4) Selecciona.
- **Alternativas:** sin resultados → ofrece registrar (UC-PAC-01).
- **Reglas de negocio:** admisiones ve solo datos administrativos; cada apertura queda en bitácora.
- **Criterio de aceptación:** buscar "Jhon Peres" encuentra a "John Pérez" y la selección queda en bitácora.

### UC-PAC-03 · Registrar cobertura (EPS, régimen, plan) y verificar derechos
- **Actor:** Admisiones.
- **Precondición:** paciente registrado.
- **Disparador:** admisión o cambio de EPS.
- **Flujo principal:** 1) Selecciona EPS y régimen del catálogo. 2) Consulta derechos (servicio de la EPS o fuente oficial disponible). 3) Guarda `Coverage` con vigencia y resultado de la verificación.
- **Alternativas:** servicio de verificación caído → se registra la cobertura como "no verificada" con motivo y se reintenta.
- **Reglas de negocio:** solo una cobertura principal vigente por régimen; la verificación guarda fecha y fuente.
- **Criterio de aceptación:** la cobertura aparece en el banner del paciente con su estado de verificación.

### UC-PAC-04 · Registrar consentimiento de tratamiento de datos
- **Actor:** Admisiones.
- **Precondición:** paciente registrado.
- **Disparador:** primera atención o cambio de autorización.
- **Flujo principal:** 1) Muestra el aviso de privacidad. 2) El paciente o acudiente acepta (firma en tableta o casilla con testigo). 3) Se guarda `Consent` con alcance y fecha.
- **Alternativas:** paciente rechaza compartir con terceros → se marca y se respeta en apps y red, sin impedir la atención.
- **Reglas de negocio:** el consentimiento es versionado; revocarlo no borra la historia (obligación legal de custodia).
- **Criterio de aceptación:** el `Consent` es consultable y su alcance se refleja en el banner del paciente.

### UC-ADM-01 · Configurar agendas por profesional y servicio
- **Actor:** Admisiones (coordinación).
- **Precondición:** profesional con membresía en el servicio.
- **Disparador:** apertura de agenda mensual.
- **Flujo principal:** 1) Define días, horas, duración de cupo y tipo de consulta. 2) Publica la agenda (`Schedule`/`Slot`).
- **Alternativas:** cruce con otra agenda del mismo profesional → se impide.
- **Reglas de negocio:** bloquear una agenda con citas exige reprogramarlas primero.
- **Criterio de aceptación:** los cupos aparecen disponibles para asignar.

### UC-ADM-02 · Asignar una cita
- **Actor:** Admisiones.
- **Precondición:** paciente con cobertura y agenda publicada.
- **Disparador:** solicitud del paciente.
- **Flujo principal:** 1) Busca cupo por servicio/profesional/fecha. 2) Asigna. 3) Se crea `Appointment` y se notifica al paciente.
- **Alternativas:** sin cupos → lista de espera (UC-ADM-03).
- **Reglas de negocio:** se registra la fecha deseada por el paciente para medir oportunidad.
- **Criterio de aceptación:** la cita aparece en la agenda del profesional y el paciente recibe la notificación.

### UC-ADM-04 · Admitir al paciente ambulatorio (abrir el encuentro)
- **Actor:** Admisiones.
- **Precondición:** cita del día o atención prioritaria.
- **Disparador:** el paciente llega.
- **Flujo principal:** 1) Confirma identidad y cobertura. 2) Abre el `Encounter` clase AMB en estado "en curso". 3) El paciente aparece en la lista del profesional.
- **Alternativas:** cobertura no vigente → se admite como particular o se deja pendiente con motivo.
- **Reglas de negocio:** un encuentro abierto es requisito para documentar y ordenar.
- **Criterio de aceptación:** el paciente aparece en la lista de trabajo del médico al instante.

### UC-ADM-07 · Cerrar el encuentro ambulatorio
- **Actor:** Profesional clínico.
- **Precondición:** nota firmada y diagnóstico principal registrado.
- **Disparador:** termina la atención.
- **Flujo principal:** 1) Verifica pendientes (nota, diagnóstico, órdenes). 2) Cierra el `Encounter`. 3) Se dispara la generación del RDA.
- **Alternativas:** falta diagnóstico principal o nota sin firmar → no se permite cerrar y se indica qué falta.
- **Reglas de negocio:** un encuentro cerrado solo admite notas aclaratorias.
- **Criterio de aceptación:** al cerrar, el encuentro cambia de estado y aparece un envío de RDA en el monitor.

### UC-HCE-01 · Documentar la nota de consulta con plantilla
- **Actor:** Profesional clínico.
- **Precondición:** encuentro abierto.
- **Disparador:** inicia la consulta.
- **Flujo principal:** 1) Elige plantilla de su especialidad. 2) Diligencia motivo, enfermedad actual, revisión por sistemas, examen físico, análisis y plan. 3) Guarda borrador automáticamente. 4) Queda como `Composition` en borrador.
- **Alternativas:** red inestable → borrador local cifrado que se sincroniza al volver la conexión.
- **Reglas de negocio:** un borrador solo lo ve su autor; nada es historia oficial hasta firmar.
- **Criterio de aceptación:** al recargar la página el borrador sigue ahí y otro profesional no lo ve.

### UC-HCE-02 · Registrar diagnósticos CIE-10
- **Actor:** Profesional clínico.
- **Precondición:** encuentro abierto.
- **Disparador:** análisis de la consulta.
- **Flujo principal:** 1) Busca el código por texto o código. 2) Marca principal/relacionado y tipo (impresión, confirmado). 3) Se guarda `Condition`.
- **Alternativas:** código no vigente → no se ofrece.
- **Reglas de negocio:** un solo diagnóstico principal por encuentro.
- **Criterio de aceptación:** el diagnóstico aparece en el resumen del paciente y en la línea de tiempo.

### UC-HCE-03 · Registrar alergias y antecedentes
- **Actor:** Profesional clínico.
- **Precondición:** paciente seleccionado.
- **Disparador:** anamnesis.
- **Flujo principal:** 1) Registra alergia (sustancia, reacción, severidad) o antecedente. 2) Se guarda `AllergyIntolerance` o `Condition`/`FamilyMemberHistory`. 3) El banner se actualiza.
- **Alternativas:** "niega alergias" se registra explícitamente (no es lo mismo que vacío).
- **Reglas de negocio:** las alergias alimentan la validación de prescripciones (UC-ORD-01).
- **Criterio de aceptación:** la alergia aparece en el banner del paciente en todas las vistas.

### UC-HCE-04 · Registrar signos vitales con valores calculados
- **Actor:** Enfermería.
- **Precondición:** encuentro abierto.
- **Disparador:** preconsulta o control.
- **Flujo principal:** 1) Registra TA, FC, FR, T°, SatO2, peso y talla. 2) El sistema calcula IMC y muestra la fórmula. 3) Se guardan `Observation` con LOINC.
- **Alternativas:** valor fuera de rango fisiológico → pide confirmación.
- **Reglas de negocio:** los valores calculados nunca se digitan.
- **Criterio de aceptación:** el IMC aparece calculado con su fórmula visible y los signos se grafican en la línea de tiempo.

### UC-HCE-05 · Firmar electrónicamente la nota
- **Actor:** Profesional clínico.
- **Precondición:** nota en borrador con secciones obligatorias completas.
- **Disparador:** termina de documentar.
- **Flujo principal:** 1) Revisa vista previa. 2) Firma reautenticando (PIN o segundo factor). 3) La `Composition` pasa a "final" con hash, fecha y firmante.
- **Alternativas:** secciones obligatorias vacías → se indica cuáles.
- **Reglas de negocio:** lo firmado no se edita; queda sellado con fecha, persona y registro profesional.
- **Criterio de aceptación:** tras firmar, el botón editar desaparece y solo aparece "nota aclaratoria".

### UC-HCE-06 · Agregar una nota aclaratoria a una nota firmada
- **Actor:** Profesional clínico.
- **Precondición:** nota firmada.
- **Disparador:** detecta un error u omisión.
- **Flujo principal:** 1) Abre la nota. 2) Escribe la aclaración con motivo. 3) Firma. 4) Queda enlazada a la original (`relatesTo`).
- **Alternativas:** ninguna: la nota original nunca se modifica.
- **Reglas de negocio:** la original y la aclaratoria se muestran juntas.
- **Criterio de aceptación:** la nota original conserva su texto y su hash y muestra la aclaración enlazada.

### UC-HCE-07 · Consultar la línea de tiempo clínica del paciente
- **Actor:** Profesional clínico.
- **Precondición:** acceso al paciente.
- **Disparador:** abre la historia.
- **Flujo principal:** 1) Ve encuentros, notas, diagnósticos, órdenes, resultados y signos en orden cronológico. 2) Filtra por tipo, servicio o fecha. 3) Abre el detalle.
- **Alternativas:** paciente sin historia previa → estado vacío con acción "nueva nota".
- **Reglas de negocio:** se muestra solo lo que el rol puede ver; la apertura queda en bitácora.
- **Criterio de aceptación:** la línea de tiempo muestra la consulta recién firmada en primer lugar.

### UC-ORD-01 · Prescribir medicamentos con validación de alergias y dosis
- **Actor:** Profesional clínico.
- **Precondición:** encuentro abierto.
- **Disparador:** plan terapéutico.
- **Flujo principal:** 1) Busca el medicamento (CUM/principio activo). 2) Define dosis, vía, frecuencia y duración. 3) El sistema valida alergias, dosis máxima y duplicidad. 4) Se guarda `MedicationRequest` y se imprime/envía la fórmula.
- **Alternativas:** alerta de alergia o dosis → exige justificación para continuar; medicamento no POS/PBS → marca para trámite.
- **Reglas de negocio:** toda alerta ignorada queda con justificación y en bitácora.
- **Criterio de aceptación:** prescribir un medicamento al que el paciente es alérgico muestra la alerta y no deja continuar sin justificación.

### UC-ORD-02 · Ordenar laboratorio, imágenes y procedimientos (CUPS)
- **Actor:** Profesional clínico.
- **Precondición:** encuentro abierto.
- **Disparador:** plan diagnóstico.
- **Flujo principal:** 1) Busca CUPS. 2) Define prioridad e indicación clínica. 3) Se guarda `ServiceRequest` con destino.
- **Alternativas:** orden duplicada reciente → se avisa.
- **Reglas de negocio:** toda orden exige diagnóstico asociado.
- **Criterio de aceptación:** la orden aparece con estado "solicitada" en el seguimiento y como `ServiceRequest` FHIR.

### UC-INT-05 · Generar el RDA de paciente y de consulta externa
- **Actor:** Sistema.
- **Precondición:** encuentro ambulatorio cerrado.
- **Disparador:** evento de cierre del encuentro.
- **Flujo principal:** 1) Arma el `Bundle` FHIR con los recursos de la canasta mínima según la guía técnica vigente del RDA. 2) Valida contra los perfiles publicados por MinSalud. 3) Registra el envío pendiente en `rda_envios`.
- **Alternativas:** falta un dato obligatorio → el envío queda "incompleto" y se notifica a quien cerró el encuentro.
- **Reglas de negocio:** el RDA se arma desde la verdad clínica FHIR firmada, nunca desde texto libre recopiado.
- **Criterio de aceptación:** el bundle generado pasa la validación de perfiles y se ve en el monitor de envíos.

### UC-INT-06 · Enviar el RDA a la IHCE con reintentos e idempotencia
- **Actor:** Sistema.
- **Precondición:** RDA generado y válido.
- **Disparador:** outbox o cron de reintentos.
- **Flujo principal:** 1) Envía al servicio de la IHCE con credenciales de la institución. 2) Guarda acuse. 3) Marca "aceptado".
- **Alternativas:** error transitorio → reintento con espera exponencial; rechazo → "rechazado" con causa y alerta.
- **Reglas de negocio:** cada envío lleva una clave de idempotencia; reintentar nunca duplica un RDA.
- **Criterio de aceptación:** el RDA de la consulta del piloto aparece "aceptado" con acuse; forzar un reintento no genera un segundo registro.

## E2 · IA clínica nativa

**Edición:** Esencial (escriba y resumen) / Hospital (todas las funciones).

**Criterio de salida:** un médico del piloto atiende una consulta con el escriba ambiental, edita y firma el borrador, y en el detalle de la nota se ve el `Provenance` con modelo, versión y aprobador; un agente IA registrado consulta por MCP los datos de un paciente autorizado y recibe un rechazo al pedir uno no autorizado.

| ID | Caso de uso | Actor | Prioridad | Puntos | Estado |
|---|---|---|---|---|---|
| UC-IAC-01 | Capturar el audio de la consulta con consentimiento del paciente | Profesional clínico | 🟥 | 5 | ⬜ |
| UC-IAC-02 | Generar el borrador de nota estructurada desde la transcripción | Agente IA | 🟥 | 8 | ⬜ |
| UC-IAC-03 | Revisar sugerencias de la IA en la bandeja de aprobación | Profesional clínico | 🟥 | 5 | ⬜ |
| UC-IAC-04 | Registrar `Provenance` de cada salida de la IA | Sistema | 🟥 | 3 | ⬜ |
| UC-IAC-05 | Generar el resumen del paciente | Agente IA | 🟥 | 5 | ⬜ |
| UC-IAC-06 | Sugerir códigos CIE-10 y CUPS | Agente IA | 🟨 | 5 | ⬜ |
| UC-IAC-07 | Exponer el servidor MCP con herramientas FHIR acotadas por scopes | Sistema | 🟥 | 8 | ⬜ |
| UC-IAC-08 | Autorizar un agente IA y sus scopes | Administrador TI | 🟥 | 3 | ⬜ |
| UC-IAC-09 | Preguntar a la historia con respuestas citadas | Profesional clínico | 🟨 | 8 | ⬜ |
| UC-IAC-10 | Configurar políticas de IA por sede | Administrador TI | 🟨 | 3 | ⬜ |
| UC-IAC-11 | Medir la tasa de aceptación y edición de sugerencias | Dirección / calidad | 🟩 | 3 | ⬜ |
| UC-IAC-12 | Seudonimizar datos antes de enviarlos al modelo cuando la función no requiere identidad | Sistema | 🟨 | 5 | ⬜ |

### UC-IAC-01 · Capturar el audio de la consulta con consentimiento del paciente
- **Actor:** Profesional clínico.
- **Precondición:** encuentro abierto; función habilitada en la sede.
- **Disparador:** el médico pulsa "escriba".
- **Flujo principal:** 1) El sistema pide registrar el consentimiento verbal del paciente. 2) Graba en el navegador. 3) Sube el audio cifrado a Storage con retención corta. 4) Inicia transcripción.
- **Alternativas:** paciente no consiente → no se graba y la consulta sigue manual.
- **Reglas de negocio:** el audio se borra tras la retención configurada (por defecto 7 días) y nunca se usa para entrenar modelos.
- **Criterio de aceptación:** sin consentimiento registrado el botón de grabar no se activa; con consentimiento, el audio aparece asociado al encuentro.

### UC-IAC-02 · Generar el borrador de nota estructurada desde la transcripción
- **Actor:** Agente IA.
- **Precondición:** transcripción disponible.
- **Disparador:** fin de la grabación.
- **Flujo principal:** 1) El servicio `ai` envía transcripción y contexto mínimo al modelo vía AI Gateway. 2) Recibe nota en las secciones de la plantilla. 3) La guarda en `sugerencias_ia` como "propuesta". 4) Aparece en la bandeja.
- **Alternativas:** el modelo no responde → aviso y la nota sigue manual; contenido fuera de formato → se descarta.
- **Reglas de negocio:** la IA nunca escribe la versión vigente de la nota: solo propone.
- **Criterio de aceptación:** el borrador aparece en la bandeja en menos de 60 segundos y no figura como nota oficial hasta que el médico lo firma.

### UC-IAC-03 · Revisar sugerencias de la IA en la bandeja de aprobación
- **Actor:** Profesional clínico.
- **Precondición:** hay sugerencias "propuesta".
- **Disparador:** abre la bandeja o el panel del encuentro.
- **Flujo principal:** 1) Ve la sugerencia junto a su fuente (transcripción o recursos citados). 2) Acepta, edita o rechaza. 3) Lo aceptado pasa al borrador humano y sigue el flujo de firma.
- **Alternativas:** rechaza con motivo opcional para medir calidad.
- **Reglas de negocio:** se guarda el estado (aceptada/editada/rechazada) y el revisor; lo editado conserva diferencia con lo propuesto.
- **Criterio de aceptación:** cada sugerencia termina en uno de los tres estados con nombre del revisor visible.

### UC-IAC-04 · Registrar `Provenance` de cada salida de la IA
- **Actor:** Sistema.
- **Precondición:** una sugerencia se acepta o edita.
- **Disparador:** aprobación humana.
- **Flujo principal:** 1) Crea `Provenance` con agente IA (modelo y versión), solicitante, aprobador, hash del prompt y recurso destino. 2) Lo enlaza al recurso firmado.
- **Alternativas:** ninguna: sin `Provenance` no se acepta la sugerencia.
- **Reglas de negocio:** toda salida de IA es trazable y queda en bitácora como actor "IA".
- **Criterio de aceptación:** el detalle de la nota firmada muestra "redactado con IA (modelo, versión) · aprobado por (médico)".

### UC-IAC-05 · Generar el resumen del paciente
- **Actor:** Agente IA.
- **Precondición:** paciente con historia y acceso del solicitante.
- **Disparador:** el médico abre la historia y pide "resumen".
- **Flujo principal:** 1) Lee los recursos FHIR permitidos. 2) Genera resumen con problemas activos, medicamentos, alergias y últimos resultados. 3) Cada afirmación enlaza al recurso fuente.
- **Alternativas:** historia vacía → indica que no hay datos.
- **Reglas de negocio:** el resumen es efímero (no se guarda como historia) salvo que el médico lo incorpore a una nota.
- **Criterio de aceptación:** cada línea del resumen abre el recurso que la respalda.

### UC-IAC-07 · Exponer el servidor MCP con herramientas FHIR acotadas por scopes
- **Actor:** Sistema.
- **Precondición:** servidor FHIR y autorización SMART operativos.
- **Disparador:** un agente registrado se conecta.
- **Flujo principal:** 1) El agente se autentica con OAuth (client credentials o en nombre de un usuario). 2) Recibe herramientas (`buscar_paciente`, `leer_recursos`, `resumen`, `proponer_borrador`) según sus scopes. 3) Cada llamada pasa por el mismo control de permisos y bitácora que un humano.
- **Alternativas:** llamada fuera de scope → error de autorización registrado.
- **Reglas de negocio:** por defecto solo lectura; escribir solo crea "propuestas"; nunca firma ni ordena.
- **Criterio de aceptación:** un agente con scope de un paciente lee sus datos y recibe rechazo con otro paciente; ambas llamadas aparecen en bitácora.

### UC-IAC-08 · Autorizar un agente IA y sus scopes
- **Actor:** Administrador TI.
- **Precondición:** solicitud aprobada por dirección.
- **Disparador:** alta de una integración IA.
- **Flujo principal:** 1) Registra el agente en `apps_smart` tipo "agente MCP". 2) Asigna scopes y vigencia. 3) Entrega credenciales por canal seguro.
- **Alternativas:** revocación → tokens invalidados de inmediato.
- **Reglas de negocio:** la autorización exige aprobación de dirección registrada.
- **Criterio de aceptación:** tras revocar, la siguiente llamada del agente falla.

## E3 · Facturación, RIPS y glosas

**Edición:** Esencial.

**Criterio de salida:** una atención del piloto genera sus cargos automáticamente, se audita, se emite la FEV aceptada por la DIAN a través del proveedor tecnológico, el RIPS JSON pasa la validación, se radica ante la EPS y una glosa recibida se responde con soportes dentro de Vértice.

| ID | Caso de uso | Actor | Prioridad | Puntos | Estado |
|---|---|---|---|---|---|
| UC-FAC-01 | Configurar contratos y tarifarios con cada EPS | Facturación | 🟥 | 5 | ⬜ |
| UC-FAC-02 | Generar cargos automáticos desde la atención | Sistema | 🟥 | 8 | ⬜ |
| UC-FAC-03 | Auditar la cuenta antes de facturar | Facturación | 🟥 | 5 | ⬜ |
| UC-FAC-04 | Emitir la FEV vía proveedor tecnológico | Facturación | 🟥 | 8 | ⬜ |
| UC-FAC-05 | Generar y validar el RIPS JSON | Sistema | 🟥 | 8 | ⬜ |
| UC-FAC-06 | Radicar la factura ante la EPS | Facturación | 🟥 | 3 | ⬜ |
| UC-FAC-07 | Registrar una glosa | Facturación | 🟥 | 3 | ⬜ |
| UC-FAC-08 | Responder una glosa con soportes | Facturación | 🟥 | 5 | ⬜ |
| UC-FAC-09 | Conciliar y cerrar glosas | Facturación | 🟨 | 5 | ⬜ |
| UC-FAC-10 | Emitir notas crédito y débito | Facturación | 🟨 | 5 | ⬜ |
| UC-FAC-11 | Consultar el tablero de cartera y glosas | Dirección / calidad | 🟨 | 3 | ⬜ |
| UC-IAC-13 | Revisar con IA el riesgo de glosa antes de facturar | Agente IA | 🟨 | 5 | ⬜ |

### UC-FAC-01 · Configurar contratos y tarifarios con cada EPS
- **Actor:** Facturación.
- **Precondición:** EPS en catálogo.
- **Disparador:** firma o renovación de contrato.
- **Flujo principal:** 1) Registra modalidad (evento, PGP, cápita), vigencia y tarifario base. 2) Ajusta valores por CUPS/CUM. 3) Publica.
- **Alternativas:** tarifas superpuestas en vigencia → se impide.
- **Reglas de negocio:** los contratos son versionados; una factura usa la versión vigente a la fecha de atención.
- **Criterio de aceptación:** un CUPS del contrato muestra su valor pactado al generar cargos.

### UC-FAC-02 · Generar cargos automáticos desde la atención
- **Actor:** Sistema.
- **Precondición:** contrato vigente para la cobertura del encuentro.
- **Disparador:** eventos de atención (consulta firmada, procedimiento realizado, dispensación).
- **Flujo principal:** 1) Recibe el evento del outbox. 2) Crea `ChargeItem` con código, cantidad y valor del contrato. 3) Lo asocia al encuentro.
- **Alternativas:** sin contrato → cargo "sin tarifa" para revisión manual.
- **Reglas de negocio:** los cargos se calculan, no se digitan; un evento procesado dos veces no duplica cargos.
- **Criterio de aceptación:** al firmar la consulta aparece su cargo en la cuenta con el valor del contrato.

### UC-FAC-03 · Auditar la cuenta antes de facturar
- **Actor:** Facturación.
- **Precondición:** encuentro cerrado con cargos.
- **Disparador:** cuenta lista para facturar.
- **Flujo principal:** 1) Revisa cargos contra soportes clínicos (nota, orden, autorización). 2) Corrige con ajustes. 3) Aprueba la cuenta.
- **Alternativas:** soporte faltante → devuelve al servicio con motivo.
- **Reglas de negocio:** nunca se borra un cargo: se anula con ajuste y motivo.
- **Criterio de aceptación:** la cuenta aprobada muestra quién la auditó y cada ajuste con motivo.

### UC-FAC-04 · Emitir la FEV vía proveedor tecnológico
- **Actor:** Facturación.
- **Precondición:** cuenta auditada.
- **Disparador:** facturar.
- **Flujo principal:** 1) Arma la factura con el contrato y el usuario del servicio. 2) La envía al proveedor tecnológico autorizado. 3) Recibe CUFE y estado DIAN. 4) Guarda en `facturas`.
- **Alternativas:** rechazo DIAN → estado "rechazada" con causa y corrección.
- **Reglas de negocio:** envío idempotente; Vértice no custodia el certificado de firma en el código.
- **Criterio de aceptación:** la factura del piloto muestra CUFE y estado "aceptada".

### UC-FAC-05 · Generar y validar el RIPS JSON
- **Actor:** Sistema.
- **Precondición:** FEV emitida.
- **Disparador:** factura aceptada.
- **Flujo principal:** 1) Genera el RIPS JSON desde los recursos FHIR de la atención y la factura. 2) Lo envía al mecanismo de validación vigente. 3) Guarda el resultado (CUV o errores).
- **Alternativas:** errores de validación → lista por campo con enlace al dato de origen.
- **Reglas de negocio:** el RIPS se deriva de la historia y de la factura; no se diligencia a mano; reenvío idempotente.
- **Criterio de aceptación:** el RIPS de la factura del piloto aparece "validado" con su código.

### UC-FAC-06 · Radicar la factura ante la EPS
- **Actor:** Facturación.
- **Precondición:** FEV aceptada y RIPS validado.
- **Disparador:** cierre de corte.
- **Flujo principal:** 1) Selecciona facturas. 2) Radica por el canal de la EPS. 3) Registra fecha y número de radicado.
- **Alternativas:** canal de la EPS caído → radicación manual con soporte cargado.
- **Reglas de negocio:** la fecha de radicación inicia los plazos de glosa.
- **Criterio de aceptación:** la factura muestra número y fecha de radicado.

### UC-FAC-07 · Registrar una glosa
- **Actor:** Facturación.
- **Precondición:** factura radicada.
- **Disparador:** la EPS notifica una glosa.
- **Flujo principal:** 1) Registra código de glosa, valor y detalle. 2) Se calcula el plazo de respuesta. 3) Se asigna responsable.
- **Alternativas:** glosa sobre factura inexistente → se rechaza.
- **Reglas de negocio:** el plazo se calcula desde la fecha de notificación.
- **Criterio de aceptación:** la glosa aparece en el tablero con plazo y responsable.

### UC-FAC-08 · Responder una glosa con soportes
- **Actor:** Facturación.
- **Precondición:** glosa abierta.
- **Disparador:** preparación de respuesta.
- **Flujo principal:** 1) Adjunta soportes generados desde la historia (nota, orden, autorización). 2) Acepta, acepta parcialmente o rechaza con argumento. 3) Envía y registra.
- **Alternativas:** soporte no disponible → solicita al servicio.
- **Reglas de negocio:** los soportes se generan desde la verdad clínica; las aceptaciones generan nota crédito.
- **Criterio de aceptación:** la glosa queda "respondida" con soportes adjuntos y fecha.

## E4 · Urgencias, hospitalización y farmacia

**Edición:** Hospital.

**Criterio de salida:** un paciente de prueba pasa triage, observación y hospitalización con cama asignada en el mapa, recibe medicamentos con eMAR verificado por código y dispensados por dosis unitaria, egresa con epicrisis firmada y su RDA de urgencias y de hospitalización aparece "aceptado".

| ID | Caso de uso | Actor | Prioridad | Puntos | Estado |
|---|---|---|---|---|---|
| UC-URG-01 | Clasificar al paciente en triage | Enfermería | 🟥 | 5 | ⬜ |
| UC-URG-02 | Ver el tablero de urgencias en tiempo real | Profesional clínico | 🟥 | 5 | ⬜ |
| UC-URG-03 | Pasar a observación y decidir destino | Profesional clínico | 🟨 | 3 | ⬜ |
| UC-URG-04 | Alertar reingresos a 72 horas | Sistema | 🟩 | 2 | ⬜ |
| UC-ADM-08 | Admitir a hospitalización y asignar cama | Admisiones | 🟥 | 5 | ⬜ |
| UC-ADM-09 | Trasladar entre servicios | Admisiones | 🟨 | 3 | ⬜ |
| UC-ADM-10 | Dar egreso administrativo | Admisiones | 🟥 | 3 | ⬜ |
| UC-HOS-01 | Consultar y operar el mapa de camas | Enfermería | 🟥 | 5 | ⬜ |
| UC-HOS-02 | Hacer la ronda médica y la evolución diaria | Profesional clínico | 🟥 | 5 | ⬜ |
| UC-HOS-03 | Registrar el plan de cuidados de enfermería | Enfermería | 🟨 | 5 | ⬜ |
| UC-HOS-04 | Administrar medicamentos con eMAR y doble verificación | Enfermería | 🟥 | 8 | ⬜ |
| UC-HOS-05 | Registrar balance de líquidos y escalas | Enfermería | 🟨 | 5 | ⬜ |
| UC-HOS-06 | Entregar el turno | Enfermería | 🟨 | 3 | ⬜ |
| UC-HCE-10 | Elaborar y firmar la epicrisis | Profesional clínico | 🟥 | 5 | ⬜ |
| UC-IAC-14 | Proponer el borrador de epicrisis con IA | Agente IA | 🟨 | 5 | ⬜ |
| UC-FAR-01 | Validar prescripciones (farmacia) | Farmacia | 🟥 | 5 | ⬜ |
| UC-FAR-02 | Dispensar por dosis unitaria | Farmacia | 🟥 | 5 | ⬜ |
| UC-FAR-03 | Registrar devoluciones | Farmacia | 🟨 | 3 | ⬜ |
| UC-FAR-04 | Controlar inventario por lote y vencimiento | Farmacia | 🟥 | 5 | ⬜ |
| UC-FAR-05 | Controlar medicamentos de control especial | Farmacia | 🟨 | 5 | ⬜ |
| UC-INT-08 | Generar y enviar el RDA de urgencias y hospitalización | Sistema | 🟥 | 5 | ⬜ |

### UC-URG-01 · Clasificar al paciente en triage
- **Actor:** Enfermería.
- **Precondición:** paciente identificado (o NN temporal).
- **Disparador:** llegada a urgencias.
- **Flujo principal:** 1) Registra motivo, signos vitales y discriminadores. 2) Asigna nivel I-V. 3) Se abre `Encounter` EMER y el paciente entra al tablero con su tiempo objetivo.
- **Alternativas:** nivel I → alerta inmediata al equipo de reanimación.
- **Reglas de negocio:** el triage registra hora exacta; reclasificar exige motivo.
- **Criterio de aceptación:** el paciente aparece en el tablero con nivel y cronómetro de espera.

### UC-URG-02 · Ver el tablero de urgencias en tiempo real
- **Actor:** Profesional clínico.
- **Precondición:** membresía en urgencias.
- **Disparador:** turno.
- **Flujo principal:** 1) Ve pacientes por zona, nivel, tiempo de espera, responsable y pendientes. 2) Toma un paciente. 3) El tablero se actualiza para todos.
- **Alternativas:** tiempo objetivo vencido → resaltado.
- **Reglas de negocio:** actualización en vivo sin recargar.
- **Criterio de aceptación:** tomar un paciente en un equipo lo muestra asignado en otro en menos de 3 segundos.

### UC-ADM-08 · Admitir a hospitalización y asignar cama
- **Actor:** Admisiones.
- **Precondición:** orden de hospitalización.
- **Disparador:** decisión médica.
- **Flujo principal:** 1) Selecciona cama libre compatible (servicio, sexo, aislamiento). 2) Abre `Encounter` IMP. 3) La cama pasa a "ocupada".
- **Alternativas:** sin cama → lista de espera de cama con prioridad.
- **Reglas de negocio:** una cama, un paciente; el encuentro de urgencias se enlaza como origen.
- **Criterio de aceptación:** la cama aparece ocupada con el nombre del paciente en el mapa.

### UC-ADM-10 · Dar egreso administrativo
- **Actor:** Admisiones.
- **Precondición:** egreso médico y epicrisis firmada.
- **Disparador:** salida del paciente.
- **Flujo principal:** 1) Verifica pendientes (epicrisis, cuenta). 2) Cierra el encuentro. 3) La cama pasa a "aseo".
- **Alternativas:** sin epicrisis → se impide.
- **Reglas de negocio:** el cierre dispara RDA de hospitalización y cierre de la cuenta.
- **Criterio de aceptación:** la cama cambia a "aseo" y aparece el RDA en el monitor.

### UC-HOS-01 · Consultar y operar el mapa de camas
- **Actor:** Enfermería.
- **Precondición:** ubicaciones configuradas.
- **Disparador:** turno o movimiento de paciente.
- **Flujo principal:** 1) Ve camas por servicio con estado y aislamiento. 2) Cambia estados (aseo → libre, bloqueada). 3) Abre el paciente desde la cama.
- **Alternativas:** bloquear cama ocupada → se impide.
- **Reglas de negocio:** cada cambio de estado registra actor y motivo.
- **Criterio de aceptación:** el porcentaje de ocupación del servicio cambia al liberar una cama.

### UC-HOS-02 · Hacer la ronda médica y la evolución diaria
- **Actor:** Profesional clínico.
- **Precondición:** paciente hospitalizado.
- **Disparador:** ronda.
- **Flujo principal:** 1) Abre la lista de sus pacientes en tableta. 2) Revisa signos, resultados y pendientes. 3) Escribe y firma la evolución. 4) Ajusta órdenes.
- **Alternativas:** sin conexión momentánea → borrador local.
- **Reglas de negocio:** la evolución diaria es obligatoria; su ausencia genera alerta a las 24 h.
- **Criterio de aceptación:** la evolución firmada aparece en la línea de tiempo del día.

### UC-HOS-04 · Administrar medicamentos con eMAR y doble verificación
- **Actor:** Enfermería.
- **Precondición:** prescripción validada y dispensada.
- **Disparador:** hora programada.
- **Flujo principal:** 1) Escanea la manilla del paciente. 2) Escanea el medicamento. 3) El sistema verifica paciente, medicamento, dosis, vía y hora. 4) Registra la administración (`MedicationAdministration`).
- **Alternativas:** no coincide → bloqueo con motivo; medicamento de alto riesgo → exige segundo verificador; dosis no administrada → motivo obligatorio.
- **Reglas de negocio:** no se registra administración sin las dos lecturas.
- **Criterio de aceptación:** escanear un medicamento distinto al prescrito bloquea el registro.

### UC-HCE-10 · Elaborar y firmar la epicrisis
- **Actor:** Profesional clínico.
- **Precondición:** decisión de egreso.
- **Disparador:** egreso médico.
- **Flujo principal:** 1) Abre la plantilla precargada con diagnósticos, procedimientos y medicamentos. 2) Completa evolución y plan de egreso. 3) Firma.
- **Alternativas:** puede partir del borrador IA (UC-IAC-14).
- **Reglas de negocio:** sin epicrisis firmada no hay egreso administrativo.
- **Criterio de aceptación:** la epicrisis firmada se puede entregar en PDF y aparece en el RDA.

### UC-FAR-01 · Validar prescripciones (farmacia)
- **Actor:** Farmacia.
- **Precondición:** `MedicationRequest` hospitalaria activa.
- **Disparador:** nueva prescripción.
- **Flujo principal:** 1) Revisa dosis, interacciones y estabilidad. 2) Valida o devuelve al prescriptor con observación.
- **Alternativas:** devuelta → el médico recibe notificación.
- **Reglas de negocio:** farmacia no modifica la prescripción.
- **Criterio de aceptación:** una prescripción devuelta aparece al médico con la observación.

### UC-FAR-02 · Dispensar por dosis unitaria
- **Actor:** Farmacia.
- **Precondición:** prescripción validada.
- **Disparador:** corte de dispensación.
- **Flujo principal:** 1) Genera el listado por paciente y hora. 2) Dispensa por lote. 3) Registra `MedicationDispense` y descarga inventario.
- **Alternativas:** sin stock → alternativa o pendiente.
- **Reglas de negocio:** la dispensación genera cargo automáticamente.
- **Criterio de aceptación:** el inventario del lote disminuye y el cargo aparece en la cuenta.

### UC-FAR-04 · Controlar inventario por lote y vencimiento
- **Actor:** Farmacia.
- **Precondición:** bodegas configuradas.
- **Disparador:** ingresos, traslados y consultas.
- **Flujo principal:** 1) Registra ingresos con lote y vencimiento. 2) Traslada entre bodegas. 3) Consulta próximos a vencer.
- **Alternativas:** lote vencido → bloqueado para dispensar.
- **Reglas de negocio:** se dispensa primero lo que vence primero.
- **Criterio de aceptación:** un lote vencido no aparece al dispensar.

### UC-INT-08 · Generar y enviar el RDA de urgencias y hospitalización
- **Actor:** Sistema.
- **Precondición:** encuentro EMER o IMP cerrado.
- **Disparador:** cierre del encuentro.
- **Flujo principal:** 1) Arma el bundle del tipo correspondiente. 2) Valida perfiles. 3) Envía con idempotencia y guarda acuse.
- **Alternativas:** igual que UC-INT-06.
- **Reglas de negocio:** reutiliza el motor de RDA de E1.
- **Criterio de aceptación:** ambos RDA del paciente de prueba aparecen "aceptados".

## E5 · Apoyo diagnóstico, cirugía e integraciones HL7 v2

**Edición:** Hospital.

**Criterio de salida:** con Vértice Edge instalado en la red del hospital, una orden de laboratorio viaja al LIS como ORM, el resultado vuelve como ORU y aparece en la historia como `DiagnosticReport` con alerta de valor crítico; una cirugía se programa, pasa la lista de verificación y queda documentada.

| ID | Caso de uso | Actor | Prioridad | Puntos | Estado |
|---|---|---|---|---|---|
| UC-INT-09 | Instalar y registrar Vértice Edge en la red del hospital | Administrador TI | 🟥 | 8 | ⬜ |
| UC-INT-10 | Intercambiar HL7 v2 (ADT, ORM, ORU) por MLLP vía Edge | Conector Vértice Edge | 🟥 | 8 | ⬜ |
| UC-INT-11 | Gestionar la bandeja de mensajes HL7 v2 y reprocesar | Administrador TI | 🟨 | 5 | ⬜ |
| UC-APO-01 | Enviar la orden de laboratorio al LIS | Sistema | 🟥 | 5 | ⬜ |
| UC-APO-02 | Recibir resultados de laboratorio como `DiagnosticReport` | Sistema | 🟥 | 5 | ⬜ |
| UC-APO-03 | Notificar valores críticos | Sistema | 🟥 | 3 | ⬜ |
| UC-APO-04 | Integrar RIS/PACS y abrir el estudio vía DICOMweb | Profesional clínico | 🟨 | 8 | ⬜ |
| UC-APO-05 | Publicar el informe de imágenes | Laboratorio / Imágenes | 🟨 | 3 | ⬜ |
| UC-CIR-01 | Programar el quirófano | Admisiones | 🟥 | 5 | ⬜ |
| UC-CIR-02 | Aplicar la lista de verificación de cirugía segura | Enfermería | 🟥 | 3 | ⬜ |
| UC-CIR-03 | Registrar el acto anestésico | Profesional clínico | 🟨 | 5 | ⬜ |
| UC-CIR-04 | Documentar la descripción quirúrgica | Profesional clínico | 🟥 | 3 | ⬜ |
| UC-CIR-05 | Registrar consumo de insumos ligado a cargos | Enfermería | 🟨 | 5 | ⬜ |

### UC-INT-09 · Instalar y registrar Vértice Edge en la red del hospital
- **Actor:** Administrador TI.
- **Precondición:** servidor o VM en la red del hospital con salida HTTPS.
- **Disparador:** integración con equipos heredados.
- **Flujo principal:** 1) Instala el binario Go. 2) Lo registra con un código de un solo uso (certificado mTLS). 3) Declara los equipos permitidos (IP/puerto/aplicación HL7). 4) Ve su estado "en línea" en Vértice.
- **Alternativas:** pierde conexión → encola cifrado y reintenta; certificado revocado → deja de relayar.
- **Reglas de negocio:** el Edge no guarda datos clínicos persistentes fuera de su cola temporal cifrada.
- **Criterio de aceptación:** el Edge aparece en línea y un equipo no declarado no puede enviarle mensajes.

### UC-INT-10 · Intercambiar HL7 v2 (ADT, ORM, ORU) por MLLP vía Edge
- **Actor:** Conector Vértice Edge.
- **Precondición:** Edge registrado y equipo declarado.
- **Disparador:** mensaje entrante o evento saliente.
- **Flujo principal:** 1) Recibe/envía por MLLP con ACK. 2) Relaya por HTTPS al servicio `interop`. 3) Se traduce a/desde FHIR. 4) Se guarda en `mensajes_hl7v2`.
- **Alternativas:** mensaje inválido → NACK con error y registro en bandeja.
- **Reglas de negocio:** procesamiento idempotente por ID de control del mensaje.
- **Criterio de aceptación:** un ADT de prueba crea o actualiza el `Patient` y un mensaje repetido no duplica nada.

### UC-APO-01 · Enviar la orden de laboratorio al LIS
- **Actor:** Sistema.
- **Precondición:** `ServiceRequest` de laboratorio y LIS integrado.
- **Disparador:** orden firmada.
- **Flujo principal:** 1) Traduce a ORM (o FHIR si el LIS lo soporta). 2) Envía vía Edge. 3) La orden pasa a "enviada".
- **Alternativas:** LIS no responde → reintento y alerta.
- **Reglas de negocio:** el estado de la orden refleja el ACK.
- **Criterio de aceptación:** la orden aparece en el LIS y su estado cambia a "enviada" en Vértice.

### UC-APO-02 · Recibir resultados de laboratorio como `DiagnosticReport`
- **Actor:** Sistema.
- **Precondición:** orden enviada.
- **Disparador:** ORU recibido.
- **Flujo principal:** 1) Traduce a `DiagnosticReport` + `Observation` con LOINC. 2) Enlaza a la orden. 3) Notifica al solicitante.
- **Alternativas:** resultado sin orden conocida → bandeja de conciliación.
- **Reglas de negocio:** resultado corregido = nueva versión, nunca sobrescritura.
- **Criterio de aceptación:** el resultado aparece en la línea de tiempo enlazado a su orden.

### UC-APO-03 · Notificar valores críticos
- **Actor:** Sistema.
- **Precondición:** resultado con bandera crítica.
- **Disparador:** recepción del resultado.
- **Flujo principal:** 1) Notifica al médico tratante y a enfermería del servicio. 2) Exige acuse de lectura.
- **Alternativas:** sin acuse en 15 minutos → escala al jefe de servicio.
- **Reglas de negocio:** la cadena de notificación y acuse queda en bitácora.
- **Criterio de aceptación:** un resultado crítico de prueba genera alerta y escala si nadie la confirma.

### UC-CIR-01 · Programar el quirófano
- **Actor:** Admisiones (programación quirúrgica).
- **Precondición:** orden de procedimiento quirúrgico.
- **Disparador:** solicitud de programación.
- **Flujo principal:** 1) Selecciona quirófano, fecha, equipo y tiempo estimado. 2) Confirma. 3) Aparece en el tablero quirúrgico.
- **Alternativas:** cruce de quirófano o cirujano → se impide.
- **Reglas de negocio:** cancelaciones con motivo para indicadores.
- **Criterio de aceptación:** la cirugía aparece en el tablero del día.

### UC-CIR-02 · Aplicar la lista de verificación de cirugía segura
- **Actor:** Enfermería.
- **Precondición:** paciente en sala.
- **Disparador:** antes de inducción, antes de incisión y antes de salida.
- **Flujo principal:** 1) Diligencia cada momento. 2) Registra participantes. 3) Firma.
- **Alternativas:** ítem no cumplido → no avanza al siguiente momento sin justificación.
- **Reglas de negocio:** sin lista completa no se cierra la descripción quirúrgica.
- **Criterio de aceptación:** la lista queda con los tres momentos firmados y hora.

### UC-CIR-04 · Documentar la descripción quirúrgica
- **Actor:** Profesional clínico.
- **Precondición:** lista de verificación completa.
- **Disparador:** fin de la cirugía.
- **Flujo principal:** 1) Registra hallazgos, procedimiento (CUPS), complicaciones. 2) Firma. 3) Se crea `Procedure`.
- **Alternativas:** puede partir de dictado con el escriba (E2).
- **Reglas de negocio:** el procedimiento firmado genera cargos.
- **Criterio de aceptación:** el procedimiento aparece en la historia y su cargo en la cuenta.

## E6 · Portal del paciente, analítica y ecosistema SMART

**Edición:** Esencial (portal) / Hospital (SMART, CDS Hooks, analítica).

**Criterio de salida:** una app SMART de terceros se lanza desde la historia con scopes limitados; al prescribir aparece una tarjeta CDS Hooks; un paciente entra al portal, ve su RDA y revoca una app; dirección ve el tablero de indicadores de calidad.

| ID | Caso de uso | Actor | Prioridad | Puntos | Estado |
|---|---|---|---|---|---|
| UC-INT-12 | Registrar y lanzar una app SMART on FHIR desde la historia | Administrador TI | 🟥 | 8 | ⬜ |
| UC-INT-13 | Integrar servicios CDS Hooks y mostrar tarjetas | Sistema | 🟥 | 8 | ⬜ |
| UC-INT-14 | Gestionar suscripciones FHIR | Administrador TI | 🟨 | 5 | ⬜ |
| UC-INT-15 | Publicar el CapabilityStatement y pruebas de conformidad | Administrador TI | 🟨 | 3 | ⬜ |
| UC-IAC-15 | Alertar deterioro clínico y riesgo de sepsis con IA vía CDS Hooks | Agente IA | 🟨 | 8 | ⬜ |
| UC-POR-01 | Entrar al portal y ver la propia historia y RDA | Paciente | 🟥 | 5 | ⬜ |
| UC-POR-02 | Gestionar citas desde el portal | Paciente | 🟨 | 3 | ⬜ |
| UC-POR-03 | Consultar resultados liberados | Paciente | 🟨 | 3 | ⬜ |
| UC-POR-04 | Autorizar y revocar apps de terceros | Paciente | 🟥 | 5 | ⬜ |
| UC-POR-05 | Responder encuestas de experiencia | Paciente | 🟩 | 2 | ⬜ |
| UC-ANA-01 | Consultar el tablero de indicadores de calidad | Dirección / calidad | 🟥 | 8 | ⬜ |
| UC-ANA-02 | Consultar ocupación y oportunidad | Dirección / calidad | 🟨 | 5 | ⬜ |
| UC-ANA-03 | Exportar datos desidentificados para investigación | Dirección / calidad | 🟨 | 5 | ⬜ |

### UC-INT-12 · Registrar y lanzar una app SMART on FHIR desde la historia
- **Actor:** Administrador TI (registro) y profesional clínico (lanzamiento).
- **Precondición:** app aprobada por dirección.
- **Disparador:** alta de la app.
- **Flujo principal:** 1) Registra client_id, URLs de lanzamiento y redirección y scopes. 2) El profesional la lanza desde el paciente (EHR launch). 3) La app recibe token con contexto de paciente.
- **Alternativas:** scope pedido no autorizado → se recorta.
- **Reglas de negocio:** tokens de corta duración; cada acceso de la app queda en bitácora como actor "app".
- **Criterio de aceptación:** la app de prueba lee los datos del paciente en contexto y no puede leer otro paciente.

### UC-INT-13 · Integrar servicios CDS Hooks y mostrar tarjetas
- **Actor:** Sistema.
- **Precondición:** servicio CDS registrado.
- **Disparador:** hooks `patient-view` y `order-sign`.
- **Flujo principal:** 1) Envía el contexto al servicio. 2) Recibe tarjetas (info, advertencia, crítica). 3) Las muestra sin bloquear el flujo, salvo críticas.
- **Alternativas:** servicio lento (> 2 s) → se omite y se registra.
- **Reglas de negocio:** aceptar o ignorar una tarjeta queda registrado con motivo en las críticas.
- **Criterio de aceptación:** al firmar una orden de prueba aparece la tarjeta del servicio CDS.

### UC-POR-01 · Entrar al portal y ver la propia historia y RDA
- **Actor:** Paciente.
- **Precondición:** identidad verificada (enrolamiento presencial o validación de identidad).
- **Disparador:** el paciente abre el portal.
- **Flujo principal:** 1) Inicia sesión con MFA. 2) Ve resúmenes, epicrisis y RDA propios. 3) Descarga PDF.
- **Alternativas:** menor de edad → acceso del acudiente registrado.
- **Reglas de negocio:** el paciente ve lo liberado por la institución; cada consulta queda en bitácora.
- **Criterio de aceptación:** el paciente ve su RDA del piloto y no puede acceder a otro paciente cambiando la URL.

### UC-POR-04 · Autorizar y revocar apps de terceros
- **Actor:** Paciente.
- **Precondición:** sesión en el portal.
- **Disparador:** una app solicita acceso (standalone launch) o el paciente revisa permisos.
- **Flujo principal:** 1) Ve la app, los datos solicitados y la vigencia. 2) Autoriza o revoca. 3) Se actualiza el `Consent`.
- **Alternativas:** revocar → tokens invalidados de inmediato.
- **Reglas de negocio:** la autorización del paciente nunca amplía lo que la institución permite a la app.
- **Criterio de aceptación:** tras revocar, la app recibe error en su siguiente llamada.

### UC-ANA-01 · Consultar el tablero de indicadores de calidad
- **Actor:** Dirección / calidad.
- **Precondición:** datos de al menos un periodo.
- **Disparador:** comité o corte mensual.
- **Flujo principal:** 1) Ve indicadores (oportunidad, reingresos, eventos adversos, RDA a tiempo, glosas) por sede y servicio. 2) Abre la fórmula y el numerador/denominador. 3) Exporta.
- **Alternativas:** periodo sin datos → estado vacío explicado.
- **Reglas de negocio:** tableros agregados sin datos identificados; cada indicador muestra su fórmula.
- **Criterio de aceptación:** el indicador "% RDA enviado en < 24 h" coincide con el monitor de envíos.

## E7 · Red multisede

**Edición:** Red.

**Criterio de salida:** dos instituciones de prueba comparten índice maestro; un paciente referido desde la primera llega a la segunda con su historia visible bajo su consentimiento y la contrarreferencia vuelve al origen.

| ID | Caso de uso | Actor | Prioridad | Puntos | Estado |
|---|---|---|---|---|---|
| UC-RED-01 | Incorporar una institución a la red con índice maestro compartido | Administrador TI | 🟥 | 8 | ⬜ |
| UC-RED-02 | Consultar la historia de la red bajo consentimiento | Profesional clínico | 🟥 | 5 | ⬜ |
| UC-RED-03 | Referir y contrarreferir pacientes entre instituciones | Profesional clínico | 🟥 | 5 | ⬜ |
| UC-RED-04 | Consultar analítica poblacional de la red | Dirección / calidad | 🟨 | 5 | ⬜ |
| UC-RED-05 | Ver la disponibilidad de camas de la red | Admisiones | 🟨 | 3 | ⬜ |

### UC-RED-01 · Incorporar una institución a la red con índice maestro compartido
- **Actor:** Administrador TI (de la red).
- **Precondición:** contrato de la edición *Red* y acuerdo de intercambio entre instituciones.
- **Disparador:** alta de institución.
- **Flujo principal:** 1) Registra la institución como organización miembro. 2) Concilia pacientes contra el MPI de la red (enlaces, no fusiones). 3) Activa el intercambio.
- **Alternativas:** coincidencias dudosas → cola de revisión humana.
- **Reglas de negocio:** cada institución sigue siendo custodio de su historia; la red enlaza, no copia sin necesidad.
- **Criterio de aceptación:** un paciente atendido en ambas aparece enlazado con un identificador de red.

### UC-RED-02 · Consultar la historia de la red bajo consentimiento
- **Actor:** Profesional clínico.
- **Precondición:** paciente enlazado y `Consent` que permite compartir en la red.
- **Disparador:** atención del paciente.
- **Flujo principal:** 1) Ve en la línea de tiempo los encuentros de otras instituciones marcados por origen. 2) Abre el detalle.
- **Alternativas:** sin consentimiento → solo ve que existe información en otra institución (y puede romper el vidrio en urgencia).
- **Reglas de negocio:** cada acceso entre instituciones queda en la bitácora de ambas.
- **Criterio de aceptación:** el profesional ve el encuentro de la otra institución y la bitácora de esa institución registra su acceso.

### UC-RED-03 · Referir y contrarreferir pacientes entre instituciones
- **Actor:** Profesional clínico.
- **Precondición:** instituciones en la red.
- **Disparador:** necesidad de mayor complejidad o retorno.
- **Flujo principal:** 1) Crea la referencia (`ServiceRequest` con resumen). 2) La institución receptora acepta o rechaza. 3) Al terminar, envía contrarreferencia.
- **Alternativas:** rechazo → motivo y nueva búsqueda de destino.
- **Reglas de negocio:** tiempos de respuesta medidos.
- **Criterio de aceptación:** la referencia aceptada y la contrarreferencia aparecen en la historia del paciente en ambas instituciones.

## Reglas globales

| Código | Regla |
|---|---|
| RG-01 | Lo firmado no se edita: se corrige con nota aclaratoria o nueva versión, conservando la original. |
| RG-02 | La IA propone; un humano identificado aprueba todo lo que se vuelve historia, orden, dispensación, factura o envío. Cada salida aceptada lleva `Provenance`. |
| RG-03 | Todo acceso (lectura o escritura) a datos clínicos queda en la bitácora con actor (humano, sistema, IA o app), fecha, motivo y origen. Sin bitácora no hay operación. |
| RG-04 | "Romper el vidrio" exige justificación escrita, expira solo y entra a revisión obligatoria de calidad. |
| RG-05 | FHIR R4 es la verdad clínica: RDA, RIPS, soportes de glosa, IA y apps leen de los recursos firmados, nunca de copias ni texto recopiado. |
| RG-06 | Los envíos externos (RDA, RIPS, FEV, HL7 v2) son idempotentes: reintentar nunca duplica. |
| RG-07 | Todo cambio de estado registra actor, fecha y motivo. |
| RG-08 | Valores calculados (edad, IMC, escalas, cargos, totales, indicadores) se calculan y muestran su fórmula; nunca se digitan. |
| RG-09 | Las operaciones financieras se corrigen con ajustes y notas, nunca borrando. |
| RG-10 | Los permisos salen de la membresía (sede, servicio, rol) y del consentimiento del paciente; lo más restrictivo gana. |
| RG-11 | Datos de pacientes nunca se envían a un modelo de IA sin cero retención contractual; cuando la función no requiere identidad, se seudonimiza. |

## Criterios globales de aceptación

- Cada caso se demuestra en la URL de preview de Vercel con datos de prueba identificables (nunca datos reales fuera del piloto autorizado).
- Ninguna pantalla clínica es accesible sin MFA; ningún dato de un paciente es accesible cambiando IDs en la URL o la API.
- Todo recurso clínico creado es consultable por la API FHIR y aparece en la bitácora.
- Contraste AA, operación completa por teclado y estados vacío/cargando/error en cada pantalla nueva.
- `go test ./...` y el build de Angular pasan antes de cada push.

## Eventos y notificaciones

| Evento | Destinatario | Canal |
|---|---|---|
| Invitación o cambio de rol | Usuario afectado | Aplicación y correo |
| Acceso de emergencia | Dirección / calidad | Bandeja de revisión |
| Cita asignada, reprogramada o recordatorio | Paciente | Correo / SMS / portal |
| Sugerencia de IA lista para revisión | Profesional que la pidió | Aplicación |
| RDA rechazado o incompleto | Profesional que cerró el encuentro y administrador TI | Aplicación |
| Prescripción devuelta por farmacia | Prescriptor | Aplicación |
| Resultado disponible | Solicitante | Aplicación |
| Valor crítico | Médico tratante, enfermería, jefe de servicio (escalamiento) | Aplicación con acuse obligatorio |
| FEV o RIPS rechazados | Facturación | Aplicación y correo |
| Glosa recibida o por vencer | Responsable de la glosa | Aplicación y correo |
| Vértice Edge fuera de línea | Administrador TI | Correo |
| App o agente revocado | Administrador TI | Aplicación |
| Referencia recibida | Institución receptora | Aplicación |

## Decisiones pendientes del usuario

- ~~Ratificar D1-D5 y las ediciones *Esencial / Hospital / Red*~~ — ratificadas por el dueño el 2026-10-27.
- Residencia de datos: región de Supabase y si algún cliente público exige datos en Colombia.
- Proveedor tecnológico de facturación electrónica (FEV) a integrar.
- Proveedor de IA y modelos (vía Vercel AI Gateway con cero retención): modelos Claude por defecto; validar contrato de tratamiento de datos.
- Alcance del piloto en el HUV y estrategia de convivencia/migración con Servinte (qué módulos primero, qué datos históricos se migran).
- Terminología: adopción de SNOMED CT (licencia) además de CIE-10, CUPS, CUM y LOINC.
- Precios por edición, por consultorio/cama e implantación.
- Credenciales y ambiente de pruebas de la IHCE para el envío del RDA.
