# Vértice HIS · Lean Healthcare nativo

Versión 1 · 2026-10-27 · Amplía [00-decisiones.md](00-decisiones.md) (D1),
[04-conceptual.md](04-conceptual.md) (módulo M16) y
[05-casos-de-uso.md](05-casos-de-uso.md) (casos `FLU` e `IAC-16/17`).

## 1. La idea en una frase

Lean Healthcare mejora **cómo trabaja el hospital**; Vértice es la
herramienta con la que trabaja. La propuesta es que el HIS **mida el flujo
del paciente por sí solo y sostenga la mejora continua**, en lugar de
depender de cronómetros, planillas y consultores externos.

Lean no es software: sin liderazgo, rondas al sitio de trabajo (*gemba*) y
cultura de mejora, ningún tablero cambia nada. Vértice **habilita y
sostiene** el método; no lo reemplaza.

## 2. Por qué encaja con la arquitectura

Todo lo clínico vive como recurso FHIR versionado y cada escritura pasa por
el outbox (`eventos_salida`). Cada cambio de estado de un `Encounter`,
`ServiceRequest`, `Task`, `MedicationAdministration` o `Location` ya trae su
hora y su actor. De ahí se deriva, sin digitar nada, la **cadena de hitos**
de cada atención:

```
llegada → triage → primer contacto médico → órdenes → resultados
       → decisión → solicitud de cama → cama asignada → ingreso
       → alta médica → egreso → cama libre → cama lista
```

Con esa cadena se calcula el **mapa de flujo de valor** (*value stream
map*) real y continuo: tiempos de valor agregado frente a tiempos de espera
en cada paso. Ningún referente analizado lo trae de forma nativa
(hipótesis a verificar antes de usarla comercialmente).

## 3. Los ocho desperdicios y la respuesta de Vértice

| Desperdicio | Cómo se ve en el hospital | Respuesta de Vértice | Módulos |
|---|---|---|---|
| Defectos | Errores de medicación, identificación o codificación | Validaciones que impiden el error (*poka-yoke*): dosis, alergias, paciente correcto por código, RIPS validado al registrar | ORD, HOS, FAR, FAC |
| Sobreproducción | Exámenes que no cambian la conducta | Alerta de exámenes duplicados o recientes vía CDS Hooks al ordenar | ORD, INT |
| Esperas | Paciente esperando médico, resultado, cama o traslado | Hitos medidos y alertas *andon* cuando se rompe el estándar de tiempo | FLU, URG, HOS, APO |
| Talento no aprovechado | Enfermería y médicos llenando papeles | Escriba ambiental, resúmenes y entrega de turno con IA | IAC |
| Transporte (de información) | Volver a digitar entre sistemas | Un solo registro FHIR; HL7 v2 vía Edge; RDA desde lo firmado | INT |
| Inventario | Sobrestock o faltantes en farmacia; camas "ocupadas" vacías | Reposición por consumo real; estados de cama en tiempo real | FAR, HOS |
| Movimientos | Navegar muchas pantallas; buscar al paciente | Atajos por teclado, línea de tiempo única, órdenes por protocolo | HCE, ORD |
| Reprocesos | Glosas, devoluciones, notas incompletas | Revisión pre-factura con IA y RIPS validado en línea | FAC, IAC |

## 4. Herramientas Lean que se vuelven funciones

| Herramienta | Qué hace en Vértice | Caso de uso |
|---|---|---|
| Mapa de flujo de valor | Se dibuja solo desde los hitos, por servicio y periodo | UC-FLU-07 |
| Gestión visual / *andon* | Tablero en tiempo real con semáforo por estándar de tiempo | UC-FLU-03 |
| Trabajo estandarizado | Estándares de tiempo versionados por servicio; órdenes por protocolo | UC-FLU-02, UC-ORD-04 |
| *Pull* (tirar) | El alta se planifica desde el ingreso; la solicitud de cama dispara aseo y traslado | UC-FLU-04, UC-FLU-05 |
| *Jidoka* | El flujo se detiene ante una condición insegura (alergia, paciente equivocado) | UC-ORD-01, UC-HOS-04 |
| *Heijunka* (nivelar) | Predicción de llegadas a urgencias y altas del día siguiente | UC-IAC-17 |
| Kaizen + A3 | Ciclo de mejora (planear, hacer, verificar, actuar) con línea base y resultado medidos por el sistema | UC-FLU-08 |
| *Gemba* asistido | La IA señala dónde están los cuellos de botella; la ronda al sitio de trabajo valida | UC-IAC-16 |

## 5. Indicadores de flujo (catálogo inicial)

| Indicador | Fórmula | Fuente |
|---|---|---|
| Puerta a triage | triage − llegada | `Encounter` EMER |
| Puerta a médico | primer contacto médico − llegada | `Encounter` + primera nota |
| Estancia en urgencias | egreso de urgencias − llegada | `Encounter` EMER |
| Tiempo de respuesta de laboratorio | resultado final − orden firmada | `ServiceRequest` + `DiagnosticReport` |
| Solicitud a cama asignada | cama asignada − solicitud de cama | `Task` + `Location` |
| Recambio de cama | cama lista − cama libre | estados de `ubicaciones` |
| Altas antes del mediodía | % egresos con hora < 12:00 | `Encounter` IMP |
| Estancia hospitalaria frente a la esperada | días reales − días esperados por diagnóstico | `Encounter` + fecha estimada de alta |
| Puntualidad del primer caso quirúrgico | % primeros casos que inician a la hora | `Procedure` + agenda de quirófano |
| Recambio de quirófano | inicio del siguiente caso − salida del anterior | `Procedure` |
| Tiempo de documentación | tiempo con la nota abierta por consulta | eventos del editor |
| Glosas por inconsistencia | glosas de ese tipo ÷ facturas radicadas | `glosas` |

Cada indicador muestra su fórmula, numerador y denominador (RG-08).

## 6. Reglas de diseño

1. **Se mide el proceso, no a la persona.** Los tableros agregan por
   servicio, turno y sede; nunca rankean individuos. Cada profesional puede
   ver sus propios tiempos, y nadie más los ve identificados (RG-12).
2. **Cero digitación extra.** Un hito que exige un clic adicional solo para
   medir no se diseña; se deriva de un acto clínico que ya ocurre.
3. **Los hitos son una proyección operativa**, no historia clínica: se
   reconstruyen desde `recursos_fhir` y el outbox, son idempotentes y
   referencian el `id` FHIR que los originó.
4. **Estándares versionados.** Cambiar una meta de tiempo crea una versión
   nueva; los tableros históricos se leen con la versión vigente en su
   fecha.
5. **La IA propone hipótesis, el equipo decide.** Las causas y mejoras que
   sugiere la IA llevan `Provenance` y entran al A3 solo si una persona las
   acepta.
6. **Datos desidentificados** para análisis de flujo y para la IA cuando no
   se requiere la identidad (RG-11).

## 7. Modelo de datos (adiciones)

| Tabla | Descripción | Atributos clave |
|---|---|---|
| hitos_flujo | Proyección de cada hito de una atención | encuentro, paciente (seudónimo), hito, ocurrido_en, sede, servicio, recurso FHIR de origen |
| estandares_flujo | Metas de tiempo por servicio | indicador, servicio, meta, umbral ámbar/rojo, versión, vigencia |
| mejoras_kaizen | Ciclo A3 | problema, indicador, línea base, causas, contramedidas, responsable, fechas, resultado, estado |

Las tres con RLS activado en la misma migración. Se pueblan desde el
outbox en el servicio `clinico` (proyección) y se analizan en el servicio
`ai` (cuellos de botella y predicción). No hace falta un servicio nuevo.

## 8. Ediciones y etapas

| Edición | Qué incluye de Lean |
|---|---|
| Esencial | Hitos ambulatorios (puerta a consulta, oportunidad de cita, tiempo de documentación) y estándares |
| Hospital | Todo: tablero *andon*, alta planificada, recambio de cama y quirófano, mapa de flujo de valor, kaizen e IA de flujo |
| Red | Hospital + comparación de flujo entre instituciones (*benchmarking*) desidentificado |

| Etapa | Casos | Por qué ahí |
|---|---|---|
| E1 | UC-FLU-01, UC-FLU-02 | La captura de hitos nace con el primer encuentro: sin línea base no hay antes/después en el piloto |
| E4 | UC-FLU-03, UC-FLU-04, UC-FLU-05 | Urgencias y camas son donde más se espera |
| E5 | UC-FLU-06 | Quirófanos |
| E6 | UC-FLU-07, UC-FLU-08, UC-IAC-16, UC-IAC-17 | Con meses de hitos ya hay datos para mapa, kaizen e IA |

## 9. Valor para el negocio y el piloto

- **Retorno medible:** el piloto en el HUV se diseña con línea base (primeras
  semanas de E1 y E4) y comparación posterior, para mostrar menos espera,
  menos estancia, más giro de cama y menos glosas con datos del propio
  sistema.
- **Ingresos por servicios:** acompañamiento Lean (formación, rondas,
  ciclos kaizen) como add-on de implantación; aumenta la permanencia del
  cliente.
- **Meta propuesta (a validar con el piloto):** −20 % en puerta a médico de
  urgencias y +15 % de altas antes del mediodía a los 6 meses de E4.

## 10. Riesgos

| Riesgo | Mitigación |
|---|---|
| El personal percibe vigilancia y rechaza el sistema | Regla 1 (proceso, no persona); comunicarlo desde la implantación; comité con representación clínica |
| Tableros sin método (se mira, no se mejora) | Kaizen dentro del producto con responsable y fecha; acompañamiento Lean en la implantación |
| Hitos incompletos por flujos fuera del sistema (papel, Servinte en convivencia) | Indicar cobertura del dato en cada indicador; no publicar indicadores con cobertura < 80 % |
| Alcance que infla el MVP | En E1 solo captura de hitos y estándares (8 puntos); lo demás en E4-E6 |
