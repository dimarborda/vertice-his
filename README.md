# Vértice HIS

**Sistema de información hospitalaria colombiano, interoperable por diseño (HL7 FHIR R4) y con inteligencia artificial integrada en el trabajo clínico.**

> Estado: **diseño completo y ratificado, sin código** · Fecha: 27 de octubre de 2026 ·
> Método: skill [`diseno-express-de-apps`](../hourus-skills/diseno-express-de-apps/SKILL.md) (Hourus Skills) con variantes ·
> Stack: Go + Angular + Vercel (mismo patrón que `n4na_digital`)

---

## En una frase

Vértice HIS permite a hospitales e IPS **atender, documentar, facturar e
intercambiar la información del paciente en un solo registro
interoperable**, con IA que redacta, resume y alerta, sin depender de
suites cerradas, caras e integradas a la medida.

## El problema

| Hoy | Consecuencia |
|---|---|
| Los HIS nacionales más usados (Hosvital, Servinte, PANACEA) nacieron antes de FHIR. | Cada integración (laboratorio, imágenes, EPS, MinSalud) es un desarrollo a la medida. |
| El referente con interoperabilidad nativa (TrakCare de InterSystems) y el más completo (SAP; según el equipo, el que usa la Fundación Valle del Lili) son caros, extranjeros y lentos de implantar. | Solo las instituciones grandes pueden pagarlos. SAP IS-H tiene soporte estándar hasta 2027 y extendido hasta 2030, sin sucesor de SAP: quienes lo usan buscarán reemplazo. |
| La norma ya obliga a interoperar: Ley 2015 de 2020, Res. 866 de 2021 y **Res. 1888 de 2025 (Resumen Digital de Atención, RDA)**. | Las IPS tienen que enviar el RDA a la plataforma del Ministerio con sistemas que no fueron hechos para eso. |
| RIPS, factura electrónica y glosas se concilian a mano. | Pérdida de ingresos por glosas evitables. |
| La documentación clínica consume buena parte de la consulta, y no hay IA en el flujo. | Menos tiempo con el paciente y notas incompletas que después se glosan. |

## La propuesta

1. **FHIR como formato nativo, no como adaptador.** Todo lo clínico existe desde el primer momento como recurso HL7 FHIR R4 versionado. El RDA, la API pública, las apps de terceros y la IA leen lo mismo que se firmó.
2. **El ciclo colombiano completo.** Identificar → admitir → atender → ejecutar → cerrar y enviar el RDA → facturar (FEV + RIPS JSON) → gestionar glosas → medir.
3. **IA integrada y gobernada.** Escriba ambiental que pasa la consulta a nota, resumen del paciente, borrador de epicrisis, codificación CIE-10/CUPS, revisión antes de facturar, alertas CDS Hooks y un servidor **MCP** para que agentes autorizados consulten la historia con los mismos permisos que una persona. **La IA propone y un profesional aprueba**, y cada sugerencia queda registrada con su procedencia (`Provenance`).
4. **Equipos heredados conectados sin abrir el hospital a internet.** El conector *Vértice Edge*, instalado en la red del hospital, traduce HL7 v2 a FHIR y lo envía de forma segura.
5. **Tres ediciones sobre el mismo núcleo**:

| Edición | Para quién | Incluye |
|---|---|---|
| **Esencial** | IPS ambulatorias y de mediana complejidad | Pacientes, agenda, historia clínica, órdenes, facturación/RIPS, RDA, IA de documentación, portal |
| **Hospital** | Hospitales de alta complejidad (piloto: HUV) | Esencial + urgencias, hospitalización, eMAR, cirugía, farmacia, laboratorio e imágenes, analítica |
| **Red** | Redes públicas y grupos hospitalarios | Hospital + índice maestro compartido, referencia y contrarreferencia, analítica poblacional |

## Referentes: qué tomamos de cada uno

| Referente | Lo que es | Lo que toma Vértice |
|---|---|---|
| **InterSystems TrakCare** *(principal)* | HIS unificado con HL7 v2, FHIR y SMART on FHIR | Registro único del paciente, una sola plataforma, interoperabilidad nativa, asistente de IA |
| **Hosvital HIS** (Digital Ware) | HIS + ERP de salud muy usado en Colombia | Ciclo regulatorio nacional: RIPS, factura, glosas, medicamentos al pie de la cama |
| **SAP for Healthcare (IS-H)** | La suite más completa y más cara | Datos maestros compartidos y analítica de gestión; ventana de mercado por el fin de IS-H |
| **Servinte Clinical Suite** | El HIS que usa hoy el HUV | Flujos que el personal ya conoce; plan de migración y convivencia en el piloto |
| **PANACEA** (CNT) | HIS para IPS medianas | Segmento de entrada → edición *Esencial* |

Se copian conceptos, flujos y modelos de negocio. **No** se copian marcas,
textos, interfaces ni código.

## Cómo se construiría

| Etapa | Qué entrega | Duración |
|---|---|---|
| E0 Cimientos | Acceso con MFA, sedes y roles, bitácora, almacén FHIR, app publicada | 3 sem |
| E1 Ciclo ambulatorio (MVP) | Paciente → cita → consulta → órdenes → **RDA enviado** | 6 sem |
| E2 IA clínica | Escriba, resumen, codificación, servidor MCP, bandeja de aprobación | 4 sem |
| E3 Facturación | Cargos, FEV, RIPS JSON, glosas y revisión con IA antes de radicar | 5 sem |
| E4 Hospital | Urgencias, camas, eMAR, farmacia | 7 sem |
| E5 Apoyo y cirugía | Laboratorio, imágenes, quirófanos, conector HL7 v2 | 6 sem |
| E6 Ecosistema | Portal del paciente, apps SMART, CDS Hooks, analítica | 5 sem |
| E7 Red | Varias sedes o instituciones | 4 sem |

Unas **40 semanas** en total: **121 casos de uso y 572 puntos**, validados con el script de la skill. La edición *Esencial* se puede pilotar al
cerrar E3 (~18 semanas).

**Indicadores de éxito:** al menos el 95 % de las atenciones con RDA
enviado en menos de 24 h · 30 % menos glosas por inconsistencias en el
primer año · 25 % menos tiempo de documentación por consulta.

## Arquitectura en breve

- **Frontend:** Angular + Tailwind + spartan/ui, publicado en Vercel.
- **Backend:** Go, con siete servicios por dominio (`auth`, `fhir`, `clinico`, `apoyo`, `facturacion`, `interop`, `ai`) como contenedores en Vercel Services.
- **Datos:** Postgres en Supabase con los recursos FHIR en `jsonb`, Supabase Auth (MFA) y Storage.
- **IA:** Vercel AI Gateway sin retención de datos hacia modelos Claude.
- **En el hospital:** conector *Vértice Edge* (Go) para HL7 v2/MLLP y DICOM.

## Cumplimiento

Datos sensibles de salud (Ley 1581 de 2012), historia clínica (Res. 1995
de 1999, Ley 2015 de 2020), interoperabilidad y RDA (Res. 866 de 2021,
Res. 1888 de 2025), facturación electrónica y RIPS (Res. 2275 de 2023),
habilitación (Res. 3100 de 2019). La IA no diagnostica ni prescribe sola.
*Las normas se citan a la fecha de consulta; el área jurídica debe
validarlas antes de producción.*

## Lo que falta decidir

Las cinco decisiones de diseño (D1-D5) quedaron **ratificadas el 27 de octubre de 2026**.

1. Residencia de los datos: nube regional o datos en Colombia.
2. Proveedor de factura electrónica y proveedor y modelos de IA.
3. Alcance del piloto en el HUV y cómo convive con Servinte.
4. Si se adopta SNOMED CT (requiere licencia).
5. Precios por edición.

## Documentos del diseño

| # | Documento | Qué contiene |
|---|---|---|
| 00 | [Decisiones](docs/00-decisiones.md) | Qué se construye, referentes, diseño visual, modelo de negocio y variantes |
| 01 | [Referente conceptual](docs/01-referente-conceptual.md) | Análisis de TrakCare y matriz comparativa de los cinco referentes |
| 02 | [Casos de uso del referente](docs/02-referente-casos-de-uso.md) | Catálogo funcional, integraciones y frentes de complejidad |
| 03 | [Diseño visual](docs/03-diseno-visual.md) | Tokens, navegación por rol e inventario de pantallas |
| 04 | [Documento conceptual](docs/04-conceptual.md) | Ciclo, roles, 16 módulos, modelo de datos, arquitectura, fases y cumplimiento |
| 05 | [Casos de uso por etapas](docs/05-casos-de-uso.md) | Backlog E0–E7 con prioridades, puntos y criterios de salida |
| 07 | [Variante de stack](docs/07-variante-stack.md) | Go + Angular + Vercel Services: servicios, repositorio, `vercel.json`, límites |
| 08 | [Interoperabilidad e IA](docs/08-interoperabilidad-e-ia.md) | FHIR, RDA, HL7 v2, SMART, CDS Hooks, servidor MCP y salvaguardas de IA |
| — | [CLAUDE.md](CLAUDE.md) | Reglas para el agente que construirá el proyecto |

*El documento 06 (lecciones aprendidas) se crea con la primera auditoría
de construcción.*
