# Vértice HIS · Documento conceptual

Versión 1 · 2026-10-27

## 1. Qué es

Vértice HIS es un sistema de información hospitalaria **FHIR-first** para
IPS y hospitales colombianos: un solo registro del paciente que cubre la
atención ambulatoria, urgencias, hospitalización, cirugía, farmacia, apoyo
diagnóstico y facturación, y que **nace interoperable** (HL7 FHIR R4 como
modelo de intercambio, HL7 v2 hacia equipos heredados, Resumen Digital de
Atención —RDA— hacia la IHCE del Ministerio de Salud) y **con IA en el
flujo** (borrador de notas, resúmenes, codificación, alertas, revisión
pre-factura y un servidor MCP para que agentes autorizados consulten la
historia con los mismos permisos que un humano).

**Rasgo distintivo frente al referente:** *ningún dato clínico queda
encerrado*: todo lo que se documenta en Vértice existe desde el primer
segundo como recurso FHIR versionado, consultable por API estándar, por el
RDA y por la IA, con traza de quién lo escribió (humano o IA) y quién lo
aprobó. TrakCare interopera *hacia afuera*; Vértice *es* el formato.

**Segundo rasgo: Lean Healthcare nativo.** Como cada acto clínico deja su
hora en FHIR, Vértice reconstruye sin digitación extra el recorrido real
de cada paciente, mide esperas frente a estándares y sostiene ciclos de
mejora continua (kaizen) con datos del propio sistema. Detalle en
[09-lean-healthcare.md](09-lean-healthcare.md).

## 2. El ciclo que ordena la plataforma

1. **Identificar** al paciente (índice maestro, cobertura/EPS, consentimientos).
2. **Admitir / agendar** el encuentro (cita, triage de urgencias o ingreso hospitalario).
3. **Atender y documentar** (historia, diagnósticos, órdenes) — con la IA como copiloto.
4. **Ejecutar** las órdenes (farmacia, laboratorio, imágenes, procedimientos, enfermería).
5. **Cerrar el encuentro** (egreso, epicrisis) y **enviar el RDA** a la IHCE.
6. **Facturar** (FEV + RIPS JSON), radicar y **gestionar glosas**.
7. **Medir y mejorar** (indicadores de calidad, de glosa y de flujo Lean) → ciclos kaizen que ajustan estándares, agendas, protocolos y reglas de la IA… y el ciclo vuelve a empezar con el siguiente encuentro del paciente.

Objeto central: **el encuentro (`Encounter`)**. De su estado dependen las
órdenes permitidas, la cama ocupada, los cargos facturables, el RDA que se
genera y el cierre de la factura.

## 3. Roles

| Rol | Responsabilidad |
|---|---|
| Médico / profesional clínico | Documenta, diagnostica, ordena, firma notas y aprueba borradores de la IA. |
| Enfermería | Triage, signos vitales, plan de cuidados, administración de medicamentos (eMAR) al pie de la cama. |
| Admisiones | Identifica pacientes, verifica derechos/cobertura, agenda citas, admite y egresa. |
| Facturación y auditoría de cuentas | Consolida cargos, genera FEV y RIPS, radica y responde glosas. |
| Farmacia | Valida prescripciones, dispensa, controla inventario y medicamentos de control. |
| Laboratorio / Imágenes | Recibe órdenes, publica resultados e informes. |
| Dirección / calidad | Consulta indicadores, aprueba protocolos y reglas de alerta. |
| Administrador TI / interoperabilidad | Configura sedes, catálogos, conectores, apps SMART y agentes IA autorizados. |
| Paciente (portal) | Consulta su historia, citas, resultados y autoriza el acceso de terceros. |
| Sistema / motor de reglas | Procesos automáticos versionados (vencimientos, RDA, alertas). |
| Agente IA | Redacta, resume, sugiere y detecta; nunca firma, ordena ni factura sin un humano. |

Una persona puede tener roles distintos por **sede y servicio** (p. ej.
médico en urgencias de la sede norte y auditor en la sede sur): los
permisos salen de la membresía `(persona, sede, servicio, rol)`.

## 4. Copiar / adaptar / descartar frente al referente

| Del referente | Decisión | Cómo queda aquí |
|---|---|---|
| Registro único del paciente para todos los ámbitos (TrakCare) | Copiar | Un `Patient` y una línea de tiempo clínica compartida por ambulatorio, urgencias, hospitalización y cirugía. |
| Un código, una interfaz, una plataforma (TrakCare) | Copiar | Un frontend Angular y un monorepo Go con servicios por dominio sobre una sola base Postgres. |
| HL7 v2 + FHIR + SMART on FHIR (TrakCare) | Copiar y profundizar | FHIR R4 como formato **canónico** (no solo fachada); HL7 v2 vía conector local; SMART on FHIR para apps de terceros; CDS Hooks para alertas. |
| Asistente de búsqueda sobre la historia (TrakCare Assistant) | Adaptar | IA nativa con servidor MCP, borradores con `Provenance` y aprobación humana obligatoria. |
| Base de datos propietaria (IRIS) | Descartar | Postgres gestionado (Supabase) con `jsonb` para recursos FHIR. |
| Ciclo regulatorio colombiano: RIPS, FEV, glosas, aseguradoras (Hosvital) | Copiar | Módulo de facturación con RIPS JSON (Res. 2275 de 2023) y FEV validada; tablero de glosas. |
| Aplicación de medicamentos al pie de la cama (Hosvital) | Copiar | eMAR en tableta con lectura de código de barras/QR de paciente y medicamento. |
| ERP integral: nómina, contabilidad, compras (SAP, Hosvital) | Posponer / integrar | No se construye: conectores con el ERP del cliente (SAP, Seven, Siigo) por API/eventos. |
| Implantación de meses y licencia institucional (TrakCare, SAP) | Descartar | SaaS por edición con implantación de precio fijo. |
| Flujos de evolución, órdenes y epicrisis que el personal conoce (Servinte) | Adaptar | Mismos conceptos y nombres; migración y convivencia durante el piloto. |
| Segmento IPS mediana con implantación ligera (PANACEA) | Adaptar | Edición *Esencial*. |

## 5. Módulos

### M0 · Cimientos (FND)
Identidad, organización y gobierno de datos.
- Inicio de sesión con MFA; sesiones con expiración por inactividad (estaciones compartidas).
- Organización → sedes → servicios → ubicaciones (camas, consultorios).
- Membresías por sede/servicio y rol; acceso de emergencia ("romper el vidrio") con justificación.
- Catálogos maestros: CIE-10 (y CIE-11 preparado), CUPS, CUM/IUM, LOINC, códigos DIVIPOLA, EPS.
- Bitácora inmutable de toda lectura y escritura de datos clínicos (`AuditEvent`).

### M1 · Índice maestro de pacientes (PAC)
Saber quién es el paciente, una sola vez.
- Registro y búsqueda por documento, nombre fonético y fecha de nacimiento.
- Detección y fusión de duplicados con aprobación humana.
- Coberturas (`Coverage`): régimen, EPS, plan; verificación de derechos.
- Consentimientos (`Consent`): tratamiento de datos, procedimientos, compartir con terceros.
- Contactos, acudientes y datos sociodemográficos de la canasta mínima (Res. 866 de 2021).

### M2 · Admisiones y agenda (ADM)
Abrir y cerrar encuentros.
- Agendas por profesional/servicio, citas, lista de espera, confirmaciones.
- Admisión ambulatoria, de urgencias y hospitalaria (`Encounter`).
- Autorizaciones de servicios ante la EPS cuando aplique.
- Egreso administrativo y traslado entre servicios o sedes.

### M3 · Historia clínica electrónica (HCE)
Documentar la atención.
- Plantillas de nota por especialidad (anamnesis, examen físico, análisis, plan) como `Composition`.
- Problemas/diagnósticos (`Condition`), alergias (`AllergyIntolerance`), antecedentes, signos vitales (`Observation`).
- Firma electrónica de notas; nota aclaratoria (nunca edición de lo firmado).
- Línea de tiempo clínica del paciente y resumen de un vistazo.
- Epicrisis y resumen de egreso.

### M4 · Órdenes clínicas (ORD)
Pedir lo que el paciente necesita.
- Prescripción (`MedicationRequest`) con validación de dosis, alergias e interacciones.
- Órdenes de laboratorio, imágenes, procedimientos e interconsultas (`ServiceRequest`).
- Conjuntos de órdenes (order sets) por protocolo.
- Estados y seguimiento de cada orden hasta su resultado.

### M5 · Urgencias (URG)
- Triage (5 niveles, Res. 5596 de 2015) con tiempos de atención.
- Tablero de urgencias en tiempo real (espera, observación, reanimación).
- Reingresos a 72 h y alertas de deterioro.

### M6 · Hospitalización y enfermería (HOS)
- Mapa de camas (`Location`) con estados y aislamientos.
- Ronda médica y plan de cuidados de enfermería.
- eMAR: administración de medicamentos con doble verificación por código.
- Balance de líquidos, escalas (Braden, Morse, Glasgow) y entregas de turno.

### M7 · Cirugía (CIR)
- Programación de quirófanos y lista de verificación de cirugía segura.
- Registro anestésico y descripción quirúrgica.
- Consumo de insumos ligado a cargos.

### M8 · Farmacia (FAR)
- Validación farmacéutica de prescripciones.
- Dispensación por dosis unitaria (`MedicationDispense`), devoluciones.
- Inventario por bodega, lotes, vencimientos y medicamentos de control especial.

### M9 · Apoyo diagnóstico (APO)
- Integración con LIS y RIS/PACS por HL7 v2 (ORM/ORU) y FHIR (`DiagnosticReport`, `ImagingStudy`).
- Publicación de resultados con valores críticos notificados.
- Visor de imágenes externo (DICOMweb) enlazado desde la historia.

### M10 · Facturación, RIPS y glosas (FAC)
- Cargos automáticos desde la atención (`ChargeItem`) con tarifario por contrato/EPS.
- Factura electrónica de venta (FEV) vía proveedor tecnológico autorizado por la DIAN.
- RIPS en JSON y validación previa (Res. 2275 de 2023).
- Radicación, glosas, respuestas y conciliación con la EPS (`Claim` / `ClaimResponse`).

### M11 · Interoperabilidad (INT)
- Servidor FHIR R4 (lectura/escritura, búsqueda, `_history`, `$everything`).
- Generación y envío del RDA (paciente, consulta externa, urgencias, hospitalización) a la IHCE.
- Conector local HL7 v2 (MLLP) para equipos e integraciones heredadas.
- Registro de apps SMART on FHIR, suscripciones (`Subscription`) y CDS Hooks.

### M12 · IA clínica nativa (IAC)
- Escriba ambiental: audio de la consulta → borrador de nota estructurada.
- Resumen del paciente y borrador de epicrisis desde su historia FHIR.
- Sugerencia de códigos CIE-10/CUPS y revisión pre-factura de riesgo de glosa.
- Alertas clínicas (interacciones, sepsis, deterioro) vía CDS Hooks.
- Servidor MCP con herramientas FHIR acotadas por permisos SMART.
- Registro de cada salida IA como `Provenance` con modelo, versión, solicitante y aprobador.

### M13 · Portal del paciente (POR)
- Citas, resultados, resúmenes y RDA propios.
- Autorizar o revocar el acceso de apps de terceros (SMART).
- Mensajería con el servicio y encuestas de experiencia.

### M14 · Analítica y calidad (ANA)
- Indicadores de calidad (Res. 256 de 2016), oportunidad, ocupación, glosas.
- Exportación de datos desidentificados para investigación (con aprobación del comité).
- Tableros por sede y servicio.

### M15 · Red multisede (RED)
- Índice maestro compartido entre sedes/hospitales.
- Referencia y contrarreferencia entre instituciones.
- Analítica poblacional de la red.

### M16 · Flujo y mejora continua · Lean Healthcare (FLU)
Capa transversal: no tiene pantallas de registro propias; se alimenta de
los actos clínicos de los demás módulos.
- Hitos de flujo derivados automáticamente de FHIR y del outbox (sin digitación extra).
- Estándares de tiempo versionados por servicio y tablero visual *andon* con semáforo.
- Alta planificada desde el ingreso y recambio de cama por *pull* (aseo y traslado).
- Mapa de flujo de valor automático y ciclos kaizen con A3 (línea base → resultado).
- IA de flujo: cuellos de botella, causas probables y predicción de demanda (con `Provenance`).
- Regla central: se mide el proceso, no a la persona.

## 6. Modelo de negocio

Desde D5: SaaS por **edición** (*Esencial*, *Hospital*, *Red*) cobrado a la
IPS por consultorio o cama activa, más implantación de precio fijo y
add-ons (IA por uso, conectores premium, analítica de red). La variable que
alinea ingresos es **atención facturada sin glosa**. El MVP se opera como
piloto institucional sin pasarela de pago; la facturación del propio SaaS
llega después de E6.

| Edición | Módulos | Cliente objetivo |
|---|---|---|
| Esencial | M0-M4, M10, M11, M12 (escriba y resumen), M13, M16 (hitos y estándares) | IPS ambulatorias y de baja/mediana complejidad |
| Hospital | Esencial + M5-M9, M12 completo, M14, M16 completo | Hospitales de mediana/alta complejidad (piloto HUV) |
| Red | Hospital + M15 (incluye comparación de flujo entre instituciones) | Redes públicas, secretarías de salud, grupos privados |

Add-on de servicios: acompañamiento Lean en la implantación (formación,
rondas al sitio de trabajo, ciclos kaizen) con línea base y resultado
medidos por Vértice.

## 7. Modelo de datos

**Principio:** los datos clínicos se guardan como **recursos FHIR R4
versionados** en Postgres (`jsonb`) — la fuente de verdad clínica — y los
datos operativos (camas, agendas, cargos, glosas, membresías) en tablas
relacionales en español que referencian el `id` FHIR. Así hay **una sola
verdad clínica** y el RDA, la API y la IA leen exactamente lo que se firmó.

| Entidad (tabla) | Descripción | Atributos clave | Relaciones |
|---|---|---|---|
| perfiles | Persona usuaria (1:1 con la identidad de Supabase Auth) | nombre, documento, registro profesional (ReTHUS), firma | membresias, `Practitioner` |
| organizaciones | Institución cliente (IPS / red) | NIT, código de habilitación (REPS), edición contratada | sedes |
| sedes | Sede habilitada | código REPS de sede, DIVIPOLA | servicios, ubicaciones |
| servicios | Servicio habilitado (urgencias, UCI…) | código de servicio REPS, complejidad | membresias, agendas |
| ubicaciones | Cama, consultorio, quirófano | tipo, estado (libre/ocupada/aseo/bloqueada), aislamiento | encuentros |
| membresias | Rol de una persona en sede/servicio | persona, sede, servicio, rol, vigencia | perfiles |
| recursos_fhir | Almacén FHIR versionado | tipo, id, versión, recurso `jsonb`, paciente_id, vigente, autor, `provenance_id` | todas las entidades clínicas |
| indices_busqueda_fhir | Parámetros de búsqueda extraídos | tipo, parámetro, valor normalizado, recurso_id | recursos_fhir |
| pacientes_mpi | Proyección operativa del `Patient` | documento, nombres fonéticos, enlace de fusión | recursos_fhir (`Patient`) |
| encuentros | Proyección operativa del `Encounter` | clase (AMB/EMER/IMP), estado, sede, servicio, ubicación, fechas | pacientes_mpi, cargos, rda_envios |
| ordenes | Proyección de `ServiceRequest`/`MedicationRequest` | tipo, estado, prioridad, destino | encuentros, resultados |
| administraciones_med | Registro eMAR | orden, dosis, hora programada/real, verificador | ordenes |
| inventario_lotes | Stock de farmacia por bodega | CUM, lote, vencimiento, cantidad | dispensaciones |
| tarifarios / contratos | Contratos con EPS y tarifas | EPS, modalidad (evento, PGP, cápita), vigencia | cargos |
| cargos | Ítems facturables (`ChargeItem`) | encuentro, CUPS/CUM, cantidad, valor, estado | facturas |
| facturas | FEV | número, CUFE, estado DIAN, total | cargos, rips_envios, glosas |
| rips_envios | RIPS JSON | factura, payload, resultado de validación, CUV | facturas |
| glosas | Objeción de la EPS | factura, código de glosa, valor, estado, respuesta | facturas |
| rda_envios | Envíos del RDA a la IHCE | encuentro, tipo de RDA, bundle FHIR, estado, acuse | encuentros |
| sugerencias_ia | Salidas de la IA pendientes de revisión | tipo, recurso borrador, modelo, prompt_hash, estado (propuesta/aceptada/editada/rechazada), revisor | recursos_fhir, `Provenance` |
| apps_smart | Apps y agentes autorizados | client_id, scopes, tipo (app/agente MCP), estado | bitacora |
| mensajes_hl7v2 | Bandeja de entrada/salida HL7 v2 | tipo (ADT/ORM/ORU), estado, reintentos, conector | ordenes, recursos_fhir |
| eventos_salida | Outbox transaccional | tipo de evento, recurso, estado de entrega | suscripciones, rda_envios |
| bitacora | Auditoría (`AuditEvent`) | actor (humano/sistema/IA), acción, entidad, antes/después, motivo, IP | — (solo inserción) |
| hitos_flujo | Proyección Lean de cada hito de una atención | encuentro, paciente (seudónimo), hito, ocurrido_en, sede, servicio, recurso FHIR de origen | encuentros, recursos_fhir |
| estandares_flujo | Metas de tiempo por servicio | indicador, servicio, meta, umbrales ámbar/rojo, versión, vigencia | servicios |
| mejoras_kaizen | Ciclo de mejora A3 | problema, indicador, línea base, causas, contramedidas, responsable, resultado, estado | estandares_flujo, `Provenance` |

Reglas de modelo:
- Lo firmado no se edita: nueva versión + nota aclaratoria; `recursos_fhir` conserva todas las versiones.
- Valores calculados (edad, IMC, escalas, totales) se calculan y muestran su fórmula.
- Estados con `check` y transiciones validadas en el dominio Go.
- Acceso a datos **solo** por el backend Go (la clave pública de Supabase no se usa en el navegador para datos clínicos); RLS activado igual en todas las tablas como defensa en profundidad.

## 8. Arquitectura

Variante de stack (detalle, estructura y despliegue en
[07-variante-stack.md](07-variante-stack.md); interoperabilidad e IA en
[08-interoperabilidad-e-ia.md](08-interoperabilidad-e-ia.md)).

| Capa | Solución |
|---|---|
| Frontend | **Angular** (standalone components, signals, RxJS) + Tailwind + spartan/ui, estático en **Vercel**, `/api/*` reescrito al proyecto de API |
| Backend | **Go 1.24+**, monorepo DDD con servicios por dominio desplegados como **contenedores en Vercel Services** (patrón `n4na_digital`) |
| Datos | **Supabase Postgres** (pooler de sesión), `jsonb` para FHIR; Supabase Auth como proveedor de identidad (OIDC + MFA); Supabase Storage para adjuntos y audio |
| Autorización | Servicio `auth` en Go: servidor OAuth 2.0 / **SMART on FHIR** (scopes `patient/*.read`, `user/*.write`…) sobre la identidad de Supabase |
| Interoperabilidad | Servicio `fhir` (R4), servicio `interop` (RDA, suscripciones, CDS Hooks) y **Vértice Edge**: binario Go instalado en la red del hospital para HL7 v2/MLLP y DICOM |
| Automatización | Vercel Cron → endpoints de los servicios (reintentos de RDA/RIPS, vencimientos); outbox transaccional en Postgres |
| IA | Servicio `ai` en Go: **Vercel AI Gateway** (cero retención de datos) hacia modelos Claude; servidor **MCP** sobre FHIR; siempre con aprobación humana |
| Facturación electrónica | Proveedor tecnológico autorizado por la DIAN vía API (no se custodia el certificado de firma en el código) |
| Repositorio | GitHub; preview por rama, producción desde `main` con aprobación |

## 9. Fases

| Fase | Alcance | Módulos | Duración |
|---|---|---|---|
| E0 | Cimientos: identidad, sedes, roles, bitácora, almacén FHIR, despliegue | FND, INT (base) | 3 semanas |
| E1 | Ciclo ambulatorio mínimo + RDA (MVP *Esencial*) + captura de hitos de flujo | PAC, ADM, HCE, ORD, INT, FLU | 6 semanas |
| E2 | IA clínica nativa en la consulta | IAC | 4 semanas |
| E3 | Facturación, RIPS y glosas | FAC | 5 semanas |
| E4 | Urgencias, hospitalización y farmacia (edición *Hospital*) + tablero *andon* y alta planificada | URG, HOS, FAR, FLU | 7 semanas |
| E5 | Apoyo diagnóstico, cirugía e integraciones HL7 v2 | APO, CIR, INT, FLU | 6 semanas |
| E6 | Portal del paciente, analítica, ecosistema SMART y mejora continua (mapa de flujo, kaizen, IA de flujo) | POR, ANA, INT, FLU, IAC | 6 semanas |
| E7 | Red multisede (edición *Red*) | RED | 4 semanas |

Por qué este orden: la interoperabilidad no es una etapa, es el
**cimiento** (el almacén FHIR existe desde E0 y el RDA cierra E1). La IA va
en E2, justo después de que existe documentación clínica real sobre la cual
operar y antes de crecer en módulos, porque es la prioridad diferencial.
La facturación (mueve dinero) llega cuando la documentación es confiable,
porque los RIPS y la FEV se derivan de ella. Hospitalización y cirugía
reutilizan todo lo anterior. La red llega cuando una institución opera
estable. Lean no es una etapa: la captura de hitos nace en E1 para tener
línea base desde el primer encuentro del piloto, y las herramientas de
mejora llegan cuando hay datos suficientes (E4-E6).

## 10. Cumplimiento

- **Datos sensibles de salud** (Ley 1581 de 2012, Decreto 1377 de 2013): consentimiento informado, finalidad, aviso de privacidad, registro de bases de datos ante la SIC, encargados del tratamiento (Supabase, Vercel, proveedor de IA) con contrato y ubicación de datos declarados.
- **Historia clínica** (Resolución 1995 de 1999; Ley 2015 de 2020): integridad, inalterabilidad, custodia, conservación, acceso solo de personal autorizado y del paciente.
- **Interoperabilidad** (Resolución 866 de 2021 — canasta mínima de datos; Resolución 1888 de 2025 — RDA obligatorio; ajustes de 2026 para RDA de dispensación): generar y enviar RDA dentro de los plazos.
- **Facturación** (Resolución 2275 de 2023 y normas DIAN): FEV del sector salud con RIPS JSON validado.
- **Habilitación** (Resolución 3100 de 2019): sedes y servicios ligados a su código REPS.
- **IA**: la IA no diagnostica ni prescribe por sí misma; toda sugerencia requiere aprobación de un profesional identificado, queda en `Provenance` y en bitácora; proveedor con cero retención y sin entrenamiento con los datos.
- **Residencia de datos**: a definir por cliente (algunas entidades públicas exigen datos en Colombia); la arquitectura permite mover Postgres sin cambiar el código.

*Normas citadas a la fecha de consulta (2026-10-27); confirmar textos y
plazos vigentes con el área jurídica antes de la salida a producción.*

## 11. Resultado esperado

Un hospital o IPS opera en Vértice el ciclo completo identificar → atender
→ ejecutar → cerrar → facturar → medir, con cada atención disponible como
FHIR, enviada como RDA en menos de 24 horas, facturada con RIPS validado y
con la IA ahorrando tiempo de documentación sin firmar nada por nadie.
