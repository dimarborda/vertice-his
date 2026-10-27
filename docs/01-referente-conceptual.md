# InterSystems TrakCare · Análisis conceptual del referente

Proyecto: Vértice HIS · Consulta: 2026-10-27

> Análisis externo elaborado a partir de información pública. No es
> documentación oficial de InterSystems TrakCare ni de los demás
> referentes citados. Las cifras son las declaradas por cada plataforma a
> la fecha de consulta. Lo que no pudo verificarse en fuentes públicas se
> marca como *declarado por el dueño del proyecto; no verificado en fuentes
> públicas*.

## 1. Ficha

| Atributo | Descripción |
|---|---|
| Categoría | Sistema de información hospitalaria unificado (HIS / EHR) con gestión administrativa, clínica, departamental y de ciclo de ingresos. |
| Origen y trayectoria | Producto de InterSystems (EE. UU.), empresa de plataformas de datos para salud. Desde 2019 corre sobre **InterSystems IRIS for Health**, lo que le añadió FHIR a los estándares que ya soportaba (HL7 v2, SDA). |
| Cobertura y escala declarada | Presencia internacional (Europa, Asia-Pacífico, Oriente Medio, Latinoamérica). Declara reconocimiento *Best in KLAS 2025 – Global Software* para TrakCare EHR. |
| Canales de acceso | Aplicación web de escritorio, interfaz adaptada a móviles para los flujos clínicos (versión anunciada por el fabricante), API de interoperabilidad (HL7 v2, FHIR, SMART on FHIR) y un *innovation toolkit* para acceder a los datos como FHIR. |

## 2. Familias de producto

El fabricante organiza la oferta alrededor de **un único registro del
paciente** compartido por todos los ámbitos de atención. Familias que se
infieren del material público:

1. **Administración del paciente (PAS/ADT)**: registro, citas, admisiones, camas, egresos.
2. **Clínica (EMR)**: documentación, órdenes, resultados, plan de cuidados, prescripción.
3. **Departamentales**: urgencias, quirófanos, farmacia, laboratorio, imágenes.
4. **Ciclo de ingresos (RCM)**: cargos, facturación, aseguradoras.
5. **Extensiones e innovación**: herramientas de datos FHIR, apps SMART, asistente de búsqueda sobre la historia (TrakCare Assistant).

Las familias reproducen el recorrido del paciente: identificarse →
admitirse → ser atendido → recibir servicios de apoyo → ser facturado.

## 3. Problema que resuelve

| Obstáculo | Manifestación concreta |
|---|---|
| Sistemas departamentales aislados | Laboratorio, farmacia y facturación con bases distintas; el médico no ve el resultado y el facturador no ve la atención. |
| Integraciones punto a punto | Cada sistema nuevo exige N interfaces a la medida; costo de mantenimiento creciente. |
| Registro fragmentado del paciente | Duplicados, historias parciales por sede o por ámbito (urgencias vs. hospitalización). |
| Pérdida de ingresos | Cargos no capturados porque la atención y la facturación viven en sistemas distintos. |
| Intercambio regional/nacional | Las redes de salud exigen compartir datos con estándares (HL7, FHIR) que los sistemas heredados no hablan. |

## 4. Propuesta de valor

> Un solo sistema —una interfaz, un código, una plataforma de datos— que
> mantiene un registro único del paciente en todos los ámbitos y lo
> comparte con el exterior mediante estándares.

Decisiones de diseño que la sostienen:
- **Unificación** de lo clínico, lo administrativo y lo financiero sobre el mismo modelo de datos.
- **Plataforma de interoperabilidad propia** (IRIS for Health) debajo del HIS: los estándares no son un añadido.
- **Extensibilidad** por FHIR y SMART on FHIR para que terceros construyan sobre los datos sin tocar el núcleo.

## 5. Modelo de negocio

### 5.1 Lados del mercado

| Lado | Qué aporta | Qué recibe |
|---|---|---|
| Institución de salud / red / gobierno | Licencia, implantación, mantenimiento | HIS unificado, soporte, actualizaciones |
| Socios implantadores | Servicios de implantación y localización | Contratos de servicios |
| Desarrolladores de terceros | Apps SMART / integraciones | Acceso estandarizado a los datos |

### 5.2 Estructura de ingresos

No hay precios públicos. El modelo observable es **licencia institucional o
de red + implantación + mantenimiento/soporte anual**, frecuentemente vía
contratación pública o proyectos de varios años. Alternativa de hospedaje
gestionado por el fabricante (declarada en su material comercial).

### 5.3 Ingresos complementarios

Plataforma de datos (IRIS for Health / HealthShare) para intercambio
regional, extensiones, formación y servicios profesionales.

### 5.4 Lógica económica

El ingreso crece con el **alcance del despliegue** (instituciones, sedes,
módulos) y con la dependencia de la plataforma de datos propietaria:
mientras más ámbitos se unifican, mayor es el costo de cambio.

## 6. Actores y roles

| Actor | Definición | Capacidades |
|---|---|---|
| Admisionista | Personal de registro y admisión | Registrar paciente, agendar, admitir, trasladar, egresar |
| Médico | Profesional que diagnostica y ordena | Documentar, ordenar, prescribir, firmar, consultar resultados |
| Enfermería | Cuidado directo | Signos vitales, plan de cuidados, administración de medicamentos |
| Farmacéutico | Validación y dispensación | Validar, dispensar, controlar inventario |
| Personal de laboratorio / imágenes | Servicios de apoyo | Recibir órdenes, publicar resultados e informes |
| Facturador | Ciclo de ingresos | Revisar cargos, facturar, gestionar aseguradoras |
| Directivo | Gestión | Consultar indicadores y reportes |
| Paciente | Usuario del portal | Consultar citas, resultados y datos propios |
| Integrador / app externa | Sistema tercero | Leer/escribir por HL7 v2, FHIR o SMART |
| Sistema | Procesos automáticos | Alertas, mensajería, reglas |

## 7. Dominios funcionales

| Dominio | Pregunta que responde | Objeto central |
|---|---|---|
| Identidad del paciente | ¿Quién es y qué cobertura tiene? | Paciente |
| Gestión de episodios | ¿Dónde está y en qué atención? | Episodio / encuentro |
| Documentación clínica | ¿Qué le pasa y qué se decidió? | Nota / diagnóstico |
| Órdenes y resultados | ¿Qué se pidió y qué salió? | Orden |
| Medicación | ¿Qué recibe, cuándo y quién lo dio? | Prescripción / administración |
| Recursos | ¿Qué cama, quirófano o agenda está libre? | Ubicación / agenda |
| Ciclo de ingresos | ¿Qué se cobra y a quién? | Cargo / factura |
| Interoperabilidad | ¿Qué se comparte, con quién y en qué formato? | Mensaje / recurso FHIR |

## 8. Objetos de dominio

| Objeto | Descripción | Atributos clave |
|---|---|---|
| Paciente | Registro único | identificadores, demografía, contactos, alertas |
| Episodio / encuentro | Una atención (ambulatoria, urgencias, hospitalaria) | tipo, estado, ubicación, profesional, fechas |
| Cita | Reserva en una agenda | recurso, horario, estado |
| Orden | Petición clínica | tipo, prioridad, estado, solicitante |
| Prescripción | Orden de medicamento | fármaco, dosis, vía, frecuencia |
| Administración | Registro de dosis aplicada | hora, dosis, responsable |
| Resultado / informe | Respuesta del laboratorio o imágenes | valores, rangos, estado, criticidad |
| Nota clínica | Documento firmado | plantilla, autor, firma, versión |
| Diagnóstico / problema | Condición codificada | código, estado clínico |
| Cama / ubicación | Recurso físico | estado, servicio |
| Cargo | Ítem facturable | código, cantidad, valor |
| Factura / reclamación | Cobro al pagador | pagador, total, estado |

Relaciones estructurales:
- Un paciente tiene N episodios; un episodio pertenece a un paciente.
- Las órdenes, notas, diagnósticos y cargos cuelgan del **episodio**: su estado (abierto, cerrado) gobierna qué se puede ordenar y qué se puede facturar.
- Una orden genera 0-N resultados y 0-N administraciones; cada uno puede generar cargos.
- Un episodio hospitalario ocupa una cama en cada momento (historial de traslados).

## 9. Ciclo de vida

Ciclo del episodio (objeto principal):

| Etapa | Qué ocurre | Dominio |
|---|---|---|
| Preadmisión / cita | Se agenda o se anuncia la llegada | Recursos |
| Admisión | Se abre el episodio y se asigna ubicación | Gestión de episodios |
| Atención | Notas, diagnósticos, órdenes | Documentación / órdenes |
| Ejecución | Resultados, administraciones, procedimientos | Órdenes / medicación |
| Traslado | Cambio de servicio o cama | Gestión de episodios |
| Egreso | Cierre clínico y resumen | Documentación |
| Facturación | Consolidación de cargos y cobro | Ciclo de ingresos |
| Intercambio | Mensajes/recursos hacia terceros en cada hito | Interoperabilidad |

## 10. Capa de IA

El fabricante anuncia **TrakCare Assistant**, búsqueda en lenguaje natural
para recuperar datos del paciente dentro del EMR, y la plataforma IRIS
incluye capacidades de analítica y búsqueda vectorial. El material público
no describe borradores de documentación con trazabilidad de aprobación ni
un protocolo abierto para agentes externos (p. ej. MCP).

## 11. Seguridad, confianza y cumplimiento

Control de acceso por rol y ubicación, auditoría de accesos, soporte de
estándares (HL7, FHIR, SMART on FHIR con OAuth 2.0) y despliegues
adaptados a la regulación de cada país por socios locales. El cumplimiento
colombiano específico (RIPS, FEV, RDA) no aparece en el material público.

## 12. Diferenciadores y limitaciones

| Diferenciador | Fundamento |
|---|---|
| Registro único en todos los ámbitos | Una sola interfaz, código y plataforma de datos (declarado). |
| Interoperabilidad nativa | Construido sobre IRIS for Health: HL7 v2, SDA, FHIR, SMART on FHIR. |
| Reconocimiento del mercado | Best in KLAS 2025 Global Software (declarado). |
| Extensibilidad | Datos accesibles como FHIR mediante su *innovation toolkit*. |

| Limitación | Implicación (oportunidad para nosotros) |
|---|---|
| Base de datos propietaria (IRIS) | Costo de cambio alto → Vértice usa Postgres abierto y FHIR como formato canónico. |
| Licencia institucional e implantación larga | Barrera para IPS medianas → SaaS por edición con implantación de precio fijo. |
| Sin normativa colombiana nativa | RIPS, FEV, glosas y RDA dependen de socios → Vértice los trae en el núcleo. |
| IA centrada en búsqueda | Oportunidad: IA que redacta con `Provenance` y aprobación humana, y servidor MCP abierto. |
| Precio y dependencia de un proveedor extranjero | Oportunidad de un producto nacional con soporte local. |

## 12b. Matriz comparativa de referentes

| Atributo | InterSystems TrakCare | SAP for Healthcare (IS-H) | Hosvital HIS (Digital Ware) | Servinte Clinical Suite | PANACEA (CNT Sistemas de Información) |
|---|---|---|---|---|---|
| Origen | EE. UU. (InterSystems) | Alemania (SAP) | Colombia (Digital Ware; inversión de H.I.G. Capital en 2019) | Colombia (Servinte, filial de Carvajal; acuerdo de adquisición por Osigu) | Colombia (CNT Sistemas de Información, según su propio perfil) |
| Segmento | Hospitales grandes, redes y sistemas nacionales | Clínicas y hospitales grandes con ERP SAP. Uso en la Fundación Valle del Lili: *declarado por el dueño del proyecto; no verificado en fuentes públicas* | IPS y hospitales medianos y grandes en Colombia, con presencia en Ecuador y Chile | Hospitales colombianos; uso en el HUV: *declarado por el dueño del proyecto; no verificado en fuentes públicas* | IPS medianas: *declarado por el dueño del proyecto; no verificado en fuentes públicas* |
| Alcance funcional | Administrativo, clínico, departamental y ciclo de ingresos unificados | Administración de pacientes y facturación (IS-H) integrada con todo el ERP (finanzas, logística, nómina); lo clínico vía soluciones asociadas | HIS + ERP de salud: administración de medicamentos al pie de la cama, procesos administrativos y financieros, integración con aseguradoras | Historia clínica electrónica y ERP para el sector salud | Software para el sector salud (detalle de módulos no disponible públicamente) |
| Interoperabilidad (HL7 v2 / FHIR / SMART) | Sí / Sí / Sí (declarado) | HL7 v2 vía integración SAP; FHIR/SMART no nativo en IS-H | Interoperabilidad con sistemas heredados (laboratorio, patología) e integración en línea con aseguradoras; FHIR no documentado públicamente | No documentado públicamente | No documentado públicamente |
| IA | TrakCare Assistant (búsqueda en lenguaje natural) | IA de la plataforma SAP (no específica de IS-H) | No documentada públicamente | No documentada públicamente | No documentada públicamente |
| Modelo comercial | Licencia institucional + implantación + mantenimiento | Licencia + implantación por integrador + mantenimiento (el más costoso del grupo) | Licencia/suscripción por institución + soporte | Licencia/suscripción + soporte | Licencia + soporte (supuesto) |
| Arquitectura | Plataforma única sobre IRIS for Health (propietaria) | SAP ECC; sin sucesor S/4HANA propio para IS-H | Suite empresarial integrada (HIS + ERP) | Suite clínica + ERP | No documentada |
| Limitaciones | Costo, base propietaria, sin normativa colombiana nativa | Mantenimiento estándar hasta 2027, extendido hasta 2030 y **sin sucesor de SAP**: obliga a migrar | Interoperabilidad estándar (FHIR) no evidente; integraciones a la medida | Incertidumbre de hoja de ruta tras el cambio de dueño; interoperabilidad no documentada | Información pública escasa |
| Qué toma Vértice | Registro único, FHIR/SMART nativo, asistente sobre la historia | Datos maestros compartidos y analítica; la **ventana de reemplazo** de IS-H | Ciclo regulatorio colombiano (RIPS, FEV, glosas), eMAR al pie de la cama, integración con EPS | Flujos clínicos que el personal ya conoce; migración y convivencia en el piloto | Posicionamiento ligero para IPS medianas → edición *Esencial* |

## 13. Lecturas aplicables a Vértice HIS

1. **El objeto central es el encuentro (`Encounter`).** Como el episodio en TrakCare, su estado gobierna qué se puede ordenar, qué cama se ocupa, qué cargos se facturan y qué RDA se genera. El modelo de datos se diseña alrededor de él.
2. **FHIR debe ser el formato canónico, no una fachada.** TrakCare agregó FHIR sobre una plataforma anterior; Vértice guarda lo clínico como recursos FHIR R4 versionados desde E0, de modo que API, RDA e IA leen lo mismo que se firmó.
3. **HL7 v2 sigue siendo obligatorio hacia equipos y sistemas heredados**, pero MLLP es TCP y no cabe en serverless: se resuelve con un conector local (Vértice Edge) en la red del hospital que habla MLLP y reenvía por HTTPS.
4. **La normativa colombiana es el foso que TrakCare no tiene.** RIPS JSON, FEV, glosas, RDA (Res. 1888 de 2025) y canasta mínima (Res. 866 de 2021) van en el núcleo, como lo hace Hosvital, no en un proyecto de localización.
5. **La IA debe redactar con trazabilidad, no solo buscar.** Cada salida de IA se guarda como `Provenance` con modelo, solicitante y aprobador; un servidor MCP expone FHIR a agentes con los mismos permisos SMART que un humano.
6. **Unificar sin encerrar.** La unificación de TrakCare crea costo de cambio por su base propietaria; Vértice unifica sobre Postgres abierto y exporta todo por FHIR (`$everything`, `_history`).
7. **Hay una ventana de mercado:** IS-H termina mantenimiento estándar en 2027 y extendido en 2030 sin sucesor propio de SAP, y Servinte cambió de dueño. Las instituciones que hoy usan esos sistemas buscarán reemplazo; la migración y convivencia deben estar diseñadas desde el piloto.
8. **El ERP no se construye, se integra.** SAP y Hosvital muestran el techo funcional; Vértice se conecta al ERP del cliente por eventos en lugar de duplicar contabilidad y nómina.

## 14. Fuentes

Consultadas el 2026-10-27.

- InterSystems — TrakCare: https://www.intersystems.com/trakcare
- InterSystems — TrakCare, sistema unificado: https://intersystems.com/trakcare/unified-healthcare-information-system
- InterSystems — TrakCare extensiones: https://www.intersystems.com/uk/trakcare/extensions
- InterSystems — Nueva versión con interfaz móvil: https://intersystems.com/sg/news/new-release-of-intersystems-trakcare-offers-mobile-enabled-user-interface-to-all-clinical-workflows
- La República — Digital Ware instaló Hosvital HIS en Clínica Neuro Cardiovascular: https://www.larepublica.co/empresas/digital-ware-instalo-el-sistema-hosvital-his-en-clinica-neuro-cardiovascular-3603193
- Portafolio — Digital Ware, apuesta colombiana en ERP: https://www.portafolio.co/negocios/empresas/digital-ware-apuesta-colombiana-sistemas-erp-88524
- Consultorsalud — Servinte será adquirido por Osigu: https://consultorsalud.com/servinte-proveedor-hce-osigu-adq/
- SlideShare — CNT Panacea: https://es.slideshare.net/cntpanacea
- Swisscom — SAP IS-H y soporte hasta 2030: https://www.swisscom.ch/fr/about/news/2023/11/07-sap.html
- E3 Magazine — Mantenimiento extendido de IS-H: https://e3mag.com/en/extended-maintenance-without-surcharge/
- MinSalud — Resumen Digital de Atención en Salud: https://www.minsalud.gov.co/Comunicaciones/noticias/2026/Paginas/resumen-digital-de-atencion-en-salud-transformara-el-acceso-a-la-informacion-clinica-en-colombia.aspx
- Consultorsalud — Colombia, Resumen Digital de Atención en Salud: https://consultorsalud.com/colombia-resumen-digital-de-atencion-en-salud/
- Cerlatam — Resolución 1799 de 2026: https://www.cerlatam.com/normatividad/ministerio-de-salud-y-proteccion-social-resolucion-no-001799-de-2026-5-de-agosto-de-2026/
