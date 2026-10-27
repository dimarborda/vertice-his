# Vértice HIS · Interoperabilidad HL7 e IA nativa

Versión 1 · 2026-10-27 · Prioridad 1 y 2 del dueño del proyecto

> Este documento amplía M11 (INT) y M12 (IAC) de
> [04-conceptual.md](04-conceptual.md). Es la referencia técnica que el
> agente constructor debe leer antes de las etapas E0, E1, E2, E5 y E6.
> Normas y guías citadas a 2026-10-27: confirmar versiones de los anexos
> técnicos de MinSalud antes de implementar.

## 1. Principios

1. **FHIR es el formato canónico, no una fachada.** Lo clínico se guarda como recurso FHIR R4 versionado; las pantallas, el RDA, la API pública y la IA leen el mismo recurso.
2. **Interoperar es un caso de uso, no una integración a la medida.** Cada intercambio tiene estándar, perfil, dirección, idempotencia y bitácora.
3. **Los heredados entran por el borde.** HL7 v2, DICOM y archivos planos llegan por Vértice Edge y se convierten a FHIR antes de tocar el núcleo.
4. **La IA es un actor con identidad, permisos y traza.** Usa las mismas APIs y los mismos scopes que una app SMART; propone, nunca firma.

## 2. Estándares y para qué se usan

| Estándar | Uso en Vértice | Dirección | Etapa |
|---|---|---|---|
| **HL7 FHIR R4** (REST, `Bundle`, `_history`, búsqueda) | Modelo canónico y API pública `/fhir/r4` | Entrada/salida | E0 |
| **RDA — Resumen Digital de Atención** (Res. 1888 de 2025, canasta mínima Res. 866 de 2021) | Bundle tipo documento por encuentro hacia la IHCE de MinSalud: paciente, consulta externa, urgencias, hospitalización (y dispensación cuando aplique) | Salida | E1 (E4 hospitalización/urgencias) |
| **International Patient Summary (IPS)** | Base del resumen del paciente y del portal; exportable a otros países | Salida | E6 |
| **SMART on FHIR v2** (App Launch + Backend Services) | Autorización de apps de terceros, portal del paciente y agentes IA | Entrada | E2 (agentes) · E6 (apps) |
| **CDS Hooks 2.0** | Alertas y sugerencias en `patient-view`, `order-select`, `order-sign`, `encounter-discharge` | Salida (Vértice llama a servicios) | E2 (internos) · E6 (externos) |
| **FHIR Subscriptions** (R4 *backport*, por tópico) | Notificar resultados, egresos, cambios de cama a sistemas suscritos | Salida | E5 |
| **FHIR Bulk Data** (`$export`) | Analítica, investigación desidentificada, migraciones | Salida | E6 |
| **HL7 v2.5.1** sobre MLLP | ADT (A01, A02, A03, A04, A08, A40), OML/ORM (órdenes), ORU^R01 (resultados), SIU^S12 (citas), DFT^P03 (cargos) con LIS, RIS, monitores y sistemas heredados (incluido Servinte durante la convivencia) | Entrada/salida vía Edge | E5 (ADT desde E1 si el piloto lo exige) |
| **DICOMweb** (WADO-RS, QIDO-RS) | Enlace a estudios en el PACS del hospital desde `ImagingStudy` | Consulta | E5 |
| **IHE PIXm / PDQm** | Consulta e identificación cruzada de pacientes entre sedes/redes | Entrada/salida | E7 |
| **IHE ATNA → `AuditEvent`** | Bitácora de accesos interoperable | Interna/exportable | E0 |
| **FEV + RIPS JSON** (Res. 2275 de 2023, DIAN) | Factura electrónica del sector salud y su soporte de servicios | Salida vía proveedor tecnológico | E3 |

### Terminologías

| Sistema | Uso |
|---|---|
| CIE-10 (CIE-11 preparado) | Diagnósticos (`Condition.code`), RIPS |
| CUPS | Procedimientos y servicios (`ServiceRequest`, `Procedure`, `ChargeItem`) |
| CUM / IUM | Medicamentos (`Medication`) |
| LOINC | Pruebas y resultados de laboratorio (`Observation.code`) |
| UCUM | Unidades |
| DIVIPOLA, códigos REPS, EPS | Ubicación, habilitación y aseguradoras |
| SNOMED CT | Opcional, sujeto a licencia; mapas a CIE-10 si se adopta (decisión pendiente) |

Servicio de terminología en `pkg/terminologia`: `CodeSystem`/`ValueSet`/
`ConceptMap` cargados desde `db/seeds`, operaciones `$lookup`,
`$validate-code` y `$translate`.

## 3. Mapa de módulos → recursos FHIR

| Módulo | Recursos FHIR principales |
|---|---|
| FND | `Organization`, `Location`, `Practitioner`, `PractitionerRole`, `HealthcareService`, `AuditEvent` |
| PAC | `Patient`, `RelatedPerson`, `Coverage`, `Consent`, `Person` (enlace de duplicados) |
| ADM | `Schedule`, `Slot`, `Appointment`, `Encounter`, `EpisodeOfCare` |
| HCE | `Composition`, `Condition`, `AllergyIntolerance`, `Observation`, `FamilyMemberHistory`, `DocumentReference`, `ClinicalImpression` |
| ORD | `ServiceRequest`, `MedicationRequest`, `RequestGroup`/`PlanDefinition` (order sets) |
| URG | `Encounter` (clase EMER), `Observation` (triage), `Flag` |
| HOS | `Location` (camas), `CarePlan`, `MedicationAdministration`, `Observation` (escalas) |
| CIR | `Procedure`, `Appointment` (quirófano), `Observation`, `Device` |
| FAR | `MedicationDispense`, `Medication`, `SupplyDelivery` |
| APO | `DiagnosticReport`, `Observation`, `Specimen`, `ImagingStudy` |
| FAC | `ChargeItem`, `Account`, `Claim`, `ClaimResponse`, `Invoice` |
| INT | `Bundle`, `Subscription`, `MessageHeader`, `CapabilityStatement` |
| IAC | `Provenance`, `Device` (el modelo como dispositivo/agente), `DocumentReference` (audio y transcripción) |
| POR | `Patient`, `Consent` (autorizaciones de apps), `Communication`, `QuestionnaireResponse` |
| ANA | `Measure`, `MeasureReport`, `Group` |
| RED | `Patient` (enlaces PIX), `ServiceRequest` (referencia), `Task` |

## 4. Flujos clave

### 4.1 Escritura clínica (toda nota, orden o resultado)

```
UI Angular ──► servicio de dominio (clinico/apoyo) ──► pkg/fhirstore
                                                        ├─ valida perfil (StructureDefinition)
                                                        ├─ escribe recursos_fhir (versión n+1) + índices
                                                        ├─ escribe proyección operativa (encuentros, ordenes…)
                                                        ├─ AuditEvent + Provenance (humano o IA + aprobador)
                                                        └─ eventos_salida (outbox) ── misma transacción
outbox ──► interop: Subscriptions · CDS Hooks posteriores · RDA · Edge (HL7 v2 salida)
```

### 4.2 RDA hacia la IHCE

1. El encuentro pasa a `finished` (egreso / cierre de consulta) → evento `encuentro.cerrado`.
2. `interop` arma un `Bundle` tipo `document`: `Composition` del tipo de RDA + `Patient`, `Encounter`, `Condition`, `MedicationRequest`, `Procedure`, `AllergyIntolerance`, `Observation` de la canasta mínima.
3. Valida contra el perfil del anexo técnico vigente; si falla, el encuentro queda con "RDA bloqueado" y la lista de campos faltantes visible para el profesional.
4. Envía con clave de idempotencia (`encuentro_id` + tipo + versión); guarda acuse en `rda_envios`.
5. Reintentos con espera exponencial (Cron cada 10 min); alerta a TI si supera 24 h (indicador D1-a).

### 4.3 Conector Vértice Edge (HL7 v2)

```
Equipo/LIS ──MLLP──► edge: ACK inmediato, cola local (SQLite/archivo WAL)
                     └─► traduce ORU^R01 → DiagnosticReport + Observation (FHIR)
                     └─► POST /edge/mensajes (mTLS, idempotente por MSH-10)
interop ──► fhir (escritura) ──► notificación al médico si hay valor crítico
Salida: ADT/ORM generados desde el outbox ──► edge ──MLLP──► LIS/RIS
```

El conector se registra como dispositivo con certificado propio, se
actualiza solo y reporta salud al panel "Conectores Edge".

## 5. IA nativa

### 5.1 Arquitectura

| Pieza | Decisión |
|---|---|
| Servicio | `ai` (Go). Ningún otro servicio llama a modelos directamente. |
| Proveedor | **Vercel AI Gateway** con *zero data retention* y sin entrenamiento con datos del cliente; modelos Claude (p. ej. `anthropic/claude-sonnet-5-5` para flujo diario, `anthropic/claude-opus-5-5` para resúmenes complejos, `anthropic/claude-haiku-5-5` para clasificación y codificación). Cambiar de modelo es configuración, no código. |
| Transcripción | Modelo de voz a texto vía el mismo gateway o proveedor con contrato equivalente; audio en Supabase Storage con borrado automático a los N días (política a definir). |
| Contexto clínico | Se arma desde FHIR del paciente (`$everything` filtrado por ventana y relevancia), no desde una copia paralela. Protocolos y guías institucionales indexados con `pgvector` en Postgres para recuperación. |
| Identidad | Cada capacidad IA es un `Device` FHIR y un cliente SMART con scopes mínimos. |
| Traza | Toda salida → `sugerencias_ia` (estado) + `Provenance` (agente = modelo/versión, `onBehalfOf` = profesional, firmante = quien aprueba) + `AuditEvent`. |

### 5.2 Capacidades

| Capacidad | Disparador | Entrada | Salida | Quién aprueba | Etapa |
|---|---|---|---|---|---|
| Escriba ambiental | Profesional inicia grabación con consentimiento verbal registrado | Audio de la consulta + contexto del paciente | Borrador de `Composition` (anamnesis, examen, análisis, plan) | Profesional que firma la nota | E2 |
| Resumen del paciente | Abrir paciente / `patient-view` | Historia FHIR reciente | Tarjeta "de un vistazo" con citas a los recursos fuente | Solo lectura (no se guarda como nota) | E2 |
| Borrador de epicrisis | Egreso | Encuentro completo | Borrador de `Composition` de egreso | Médico tratante | E2 / E4 |
| Codificación sugerida | Al firmar nota / al facturar | Nota + órdenes | CIE-10 y CUPS sugeridos con evidencia textual | Profesional / auditor de cuentas | E2 / E3 |
| Revisión pre-factura | Antes de radicar | Cuenta + nota + contrato + historial de glosas | Riesgo de glosa con causas y correcciones propuestas | Auditor de cuentas | E3 |
| Alertas clínicas (CDS Hooks) | `order-select`, `order-sign` | Orden + alergias + función renal + interacciones | Tarjetas de alerta con fuente | Prescriptor (acepta o justifica) | E2 / E4 |
| Entrega de turno | Fin de turno | Eventos del turno por paciente | Resumen SBAR por paciente | Enfermería saliente | E4 |
| Asistente conversacional | Pregunta en el panel IA | Pregunta + historia del paciente | Respuesta con citas a recursos FHIR | Solo lectura | E2 |
| Agentes externos vía MCP | App/agente autorizado por TI | Herramientas MCP con scopes SMART | Lecturas; escrituras solo como sugerencias | Profesional o TI según herramienta | E2 / E6 |

### 5.3 Servidor MCP (`/mcp`, Streamable HTTP, OAuth SMART)

| Herramienta | Scope mínimo | Efecto |
|---|---|---|
| `buscar_paciente` | `user/Patient.rs` | Busca por documento o nombre; devuelve ids y datos mínimos |
| `resumen_paciente` | `user/*.rs` (contexto de paciente) | Resumen IPS del paciente |
| `buscar_recursos` | `user/<Tipo>.rs` | Búsqueda FHIR acotada por tipo y fechas |
| `resultados_laboratorio` | `user/Observation.rs` | Resultados con tendencia |
| `proponer_nota` | `user/Composition.c` + rol IA | Crea **sugerencia** (no recurso firmado) en la bandeja del profesional |
| `proponer_codigos` | `user/Condition.c` + rol IA | Crea sugerencia de codificación |
| `estado_rda` | `system/Bundle.rs` | Consulta envíos RDA (solo TI) |

Reglas: sin herramientas de borrado; toda llamada queda en bitácora con
el cliente, el usuario delegante y el paciente; romper el vidrio no está
disponible para agentes; límites de tasa y presupuesto por cliente.

### 5.4 Salvaguardas

- **Humano en el circuito**: nada generado por IA pasa a `final`/firmado, se ordena, se dispensa ni se factura sin aceptación explícita de un profesional identificado.
- **Citas obligatorias**: resúmenes y respuestas enlazan los recursos FHIR de los que salieron; afirmación sin fuente se marca como tal.
- **Contenido como datos**: el texto de notas, documentos externos y mensajes se trata como dato, nunca como instrucción para el modelo (defensa contra inyección de instrucciones).
- **Desidentificación** para evaluaciones, ajustes de *prompts* y analítica; datos sintéticos (Synthea) en preview/staging.
- **Evaluaciones continuas**: conjunto de casos sintéticos por capacidad (precisión de codificación, omisiones en resúmenes, alertas falsas), métricas de aceptación/edición/rechazo por capacidad en el tablero de calidad.
- **Interruptor por capacidad y por sede**: TI puede apagar cualquier capacidad sin desplegar.
- **Transparencia al paciente**: el portal indica qué notas tuvieron asistencia de IA.

## 6. Interoperabilidad frente a los referentes

| Capacidad | TrakCare | SAP IS-H | Hosvital | Servinte | PANACEA | **Vértice** |
|---|---|---|---|---|---|---|
| FHIR R4 | Sí (sobre IRIS for Health) | Vía integración externa | No documentado públicamente | No documentado públicamente | No documentado públicamente | **Canónico** |
| SMART on FHIR | Sí | No documentado | No documentado | No documentado | No documentado | Sí (E2/E6) |
| HL7 v2 | Sí | Sí (vía middleware) | Interfaces a la medida | Interfaces a la medida | No documentado | Sí, vía Edge |
| RDA Colombia | Requiere localización | Requiere localización | Esperable (proveedor nacional) | Esperable (proveedor nacional) | Esperable | **Nativo** |
| RIPS JSON / FEV | No nativo | Localización | Sí (ciclo nacional) | Sí (ciclo nacional) | Esperable | Sí (E3) |
| IA en el flujo | Asistente de búsqueda | Vía plataforma SAP | No documentado | No documentado | No documentado | **Escriba, resumen, codificación, CDS, MCP** |

*Fuentes y nivel de verificación en
[01-referente-conceptual.md](01-referente-conceptual.md). "No documentado"
significa que no se encontró en información pública a 2026-10-27, no que
la capacidad no exista.*
