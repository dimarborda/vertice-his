# Vértice HIS · Decisiones de diseño

Fecha: 2026-10-27 · Stack: **variante Go + Angular + Vercel** (ver [07-variante-stack.md](07-variante-stack.md))

> Cinco decisiones que cierran el diseño. El dueño del proyecto pidió el
> 2026-10-27 *"diséñalo todo"* y delegó las decisiones. En la revisión del
> mismo día **ratificó las cinco decisiones (D1-D5) tal cual**, incluida la
> forma de copiar cada componente de los referentes (D2).

> **Variantes frente a la skill.** La skill `diseno-express-de-apps` fija
> Next.js + Supabase + Vercel. Este proyecto aplica dos variantes:
> 1. **Variante de stack**: backend en **Go** (servicios en contenedor sobre
>    Vercel Services) y frontend en **Angular** (estático en Vercel), con
>    Postgres gestionado en Supabase, igual que el proyecto `n4na_digital`.
> 2. **Variantes de edición del producto**: un mismo núcleo con tres
>    ediciones (*Esencial*, *Hospital*, *Red*), ver D1 y D5.

## D1 · Qué quiero construir

**Enunciado:** Vértice HIS es un sistema de información hospitalaria para
que hospitales e IPS colombianas puedan **atender, documentar, facturar e
intercambiar la información clínica del paciente en un solo registro
interoperable (HL7 FHIR R4) con asistencia de IA nativa**, sin depender de
suites cerradas, costosas e integradas por interfaces punto a punto.

| Campo | Valor |
|---|---|
| Usuarios (roles) | 1) Profesional clínico (médico, enfermería, terapeuta) · 2) Admisiones y facturación · 3) Servicios de apoyo (farmacia, laboratorio, imágenes) · 4) Dirección / calidad · 5) Administrador TI e interoperabilidad. El paciente y los sistemas externos son actores, no usuarios internos. |
| Quién paga | La institución prestadora (IPS pública o privada). En la edición *Red*, la entidad territorial o la red de hospitales. |
| Qué hacen hoy sin la plataforma | Usan un HIS heredado (Servinte en el HUV; Hosvital, PANACEA u otros en IPS medianas; SAP en clínicas grandes) con interoperabilidad limitada, integraciones por archivo plano o HL7 v2 artesanal, RIPS y glosas gestionados en Excel, y cero IA en el flujo clínico. |
| Resultado medible | (a) % de atenciones con **RDA enviado a la IHCE** en < 24 h (meta ≥ 95 %). (b) Glosas por inconsistencia clínico-administrativa (meta −30 % el primer año). (c) Tiempo de documentación por consulta (meta −25 % con IA de transcripción y resumen). |
| Alcance inicial | **Producto para vender** a IPS, con piloto en un hospital universitario (escenario de referencia: Hospital Universitario del Valle, HUV). |
| Variantes de edición | **Esencial** (IPS baja/mediana complejidad: ambulatorio, urgencias, facturación, RDA) · **Hospital** (alta complejidad: hospitalización, cirugía, UCI, farmacia, LIS/RIS) · **Red** (varias sedes/hospitales, índice maestro de pacientes compartido, analítica poblacional). |

**Por qué:** el mercado colombiano ya obliga a interoperar (Ley 2015 de
2020, Resolución 866 de 2021, Resolución 1888 de 2025 del RDA) y los
referentes nacionales nacieron antes de FHIR; el único referente con FHIR
nativo (TrakCare) es caro y extranjero. El hueco es un HIS **FHIR-first,
nacional y con IA en el flujo**, vendible por ediciones.

**Ampliación (2026-10-27, aprobada por el dueño):** se incorpora **Lean
Healthcare nativo** como capa transversal (módulo M16 · FLU). Vértice
deriva de FHIR los hitos de flujo de cada atención, mide esperas frente a
estándares, muestra tableros *andon* y sostiene ciclos kaizen, midiendo
el proceso y no a la persona. Se añade un resultado medible (d): espera de
puerta a médico en urgencias y altas antes del mediodía en el piloto.
Detalle en [09-lean-healthcare.md](09-lean-healthcare.md).
**Estado:** aprobada — ratificada por el dueño el 2026-10-27

## D2 · Basado en qué (referente funcional)

Mecánica del problema: **flujo de trabajo clínico-administrativo alrededor
de un objeto central (el encuentro / episodio de atención)**, con
intercambio estandarizado hacia terceros (EPS, MinSalud, laboratorios).

| Referente | Rol | Qué tomamos | Qué NO tomamos |
|---|---|---|---|
| **InterSystems TrakCare** | Principal | Registro único del paciente para todos los ámbitos; un solo código y una sola interfaz para lo clínico, lo administrativo y el ciclo de ingresos; interoperabilidad nativa HL7 v2 / FHIR / SMART on FHIR; asistente de búsqueda sobre la historia. | Dependencia de una base de datos propietaria (IRIS); modelo de licencia por institución con implantación de meses; enfoque genérico sin normativa colombiana. |
| **Hosvital HIS** (Digital Ware) | Secundario | Cobertura del ciclo colombiano completo: admisión → atención → facturación → RIPS → glosas; ERP de salud integrado; aplicación de medicamentos al pie de la cama; integración en línea con aseguradoras. | Arquitectura monolítica cliente-servidor; interoperabilidad por interfaces a la medida. |
| **SAP for Healthcare (IS-H)** | Secundario | La idea de que **todo** (clínico, logística, finanzas, nómina) comparte datos maestros; analítica de gestión. | Costo y complejidad; IS-H con mantenimiento estándar hasta 2027 y extendido hasta 2030 sin sucesor de SAP: es un mercado que buscará reemplazo. |
| **Servinte Clinical Suite** | Secundario (escenario real) | Lo que el HUV ya opera: flujos de historia clínica que el personal conoce (evolución, órdenes, epicrisis); lo usamos para **migración y convivencia**. | Su interfaz y su acoplamiento interno. |
| **PANACEA** (CNT Sistemas de Información) | Secundario | Posicionamiento para IPS medianas, implantación ligera y precio accesible → inspira la edición *Esencial*. | — (información pública escasa; no se toman funciones concretas). |

**Por qué:** TrakCare aporta el **modelo de datos y la interoperabilidad**
(prioridad del dueño); Hosvital aporta el **ciclo regulatorio colombiano**
que TrakCare no trae; SAP muestra el techo funcional y una ventana de
mercado (fin de IS-H); Servinte es la realidad del piloto; PANACEA el
segmento de entrada.
**Estado:** aprobada — ratificada por el dueño el 2026-10-27 (incluida la tabla de qué se toma y qué no de cada referente)

## D3 · Qué diseño visual quiero aplicar

| Eje | Decisión |
|---|---|
| Densidad | **Compacta** en estaciones clínicas y facturación (mucho dato por pantalla); **normal** en el portal del paciente. |
| Tono | Sobrio, clínico, confiable. Usted en el portal del paciente; frases cortas e imperativas en el personal. |
| Plataforma principal | **Escritorio** (estaciones de enfermería, consultorios, facturación) + **tableta** para ronda y aplicación de medicamentos al pie de la cama. |
| Modo | Claro por defecto; **oscuro** disponible (turnos nocturnos, salas de imágenes). |
| Identidad existente | Ninguna: nombre "Vértice" en texto con tipografía propia. |
| Accesibilidad / conectividad | Contraste AA, operación completa por teclado (atajos), tolerancia a red hospitalaria inestable (guardado de borradores locales), lenguaje clínico estándar (CIE-10/CIE-11, CUPS, CUM). |

**Por qué:** un HIS se usa 8-12 horas por turno; la densidad y el teclado
ahorran clics, y la ronda exige tableta.
**Estado:** aprobada — ratificada por el dueño el 2026-10-27

## D4 · Basado en qué (referente visual)

**Referente visual:** **IBM Carbon Design System** (densidad de datos
empresariales, tablas y estados) + **Linear** (paleta de comandos,
velocidad por teclado). Implementación con **Tailwind CSS + spartan/ui**
(equivalente de shadcn/ui para Angular) sobre Angular CDK.

Rasgos que tomamos:
1. Barra lateral colapsable por rol + **banner del paciente** fijo (identidad, alergias, alertas) en toda vista clínica.
2. Paleta de comandos `⌘K / Ctrl+K` para buscar paciente, abrir orden o saltar a módulo.
3. Tablas densas con filtros guardados, columnas configurables y acciones en lote.
4. Chips de estado con color + ícono + texto (nunca solo color).
5. Línea de tiempo clínica del paciente como vista principal del registro.

**Por qué:** Carbon resuelve la densidad de datos con accesibilidad
probada; Linear resuelve la velocidad. Ninguno es de salud, así que no hay
riesgo de confusión de marca con los referentes funcionales.
**Estado:** aprobada — ratificada por el dueño el 2026-10-27

## D5 · Qué modelo de negocio puedo obtener

| Campo | Valor |
|---|---|
| Modelo del referente | TrakCare y SAP: licencia institucional + implantación + mantenimiento anual (alto costo inicial). Hosvital/Servinte/PANACEA: licencia o suscripción por institución + soporte, con módulos adicionales. |
| Modelo adaptado | **SaaS por edición y por cama/consultorio activo**, con implantación de precio fijo. Add-ons: IA clínica (por uso), conectores premium (LIS/RIS/PACS), analítica de red. |
| Quién paga, cuánto y cuándo | La IPS paga mensual. Tentativo: *Esencial* desde 2 USD/consulta-equivalente o tarifa plana por consultorio; *Hospital* por cama activa/mes; *Red* contrato institucional anual. Implantación fija por edición. (Cifras a validar con el mercado.) |
| Variable que alinea nuestro ingreso con el del cliente | **Atenciones facturadas sin glosa** y RDA enviados: si Vértice reduce glosas, el cliente cobra más y justifica el crecimiento por camas/sedes. |
| Entra al MVP | Licenciamiento interno del piloto (sin pasarela); medición de uso por edición. |
| Se pospone | Facturación del SaaS, cobro por uso de IA, marketplace de conectores. |
| Riesgos (regulación, fraude, impuestos, datos) | Datos de salud = **datos sensibles** (Ley 1581 de 2012); custodia de historia clínica (Resolución 1995 de 1999 y Ley 2015 de 2020); residencia de datos exigida por algunas entidades públicas; contratación pública (Ley 80) en la edición *Red*; responsabilidad sobre sugerencias de IA (siempre con aprobación humana). |

**Por qué:** el modelo SaaS por edición rompe la barrera de entrada de
TrakCare/SAP y compite con los nacionales por interoperabilidad e IA, no
por precio.
**Estado:** aprobada — ratificada por el dueño el 2026-10-27
