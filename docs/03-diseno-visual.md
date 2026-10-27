# Vértice HIS · Diseño visual

Fecha: 2026-10-27 · Base técnica (variante Angular): **Tailwind CSS + spartan/ui + Angular CDK**

> Se toman patrones de interfaz del referente visual (IBM Carbon Design
> System y Linear), nunca su marca, logo, textos ni imágenes. spartan/ui
> cumple en Angular el papel que shadcn/ui cumple en React: componentes
> accesibles copiados al proyecto y personalizados con tokens propios.

## 1. Referente visual y rasgos tomados

| Rasgo del referente | Cómo lo aplicamos |
|---|---|
| Tablas de datos densas con filtros, orden y acciones en lote (Carbon) | Listas de trabajo (pacientes en urgencias, órdenes pendientes, cargos, glosas) con filtros guardados por usuario y columnas configurables. |
| Jerarquía tipográfica de "productividad" (Carbon) | Texto base 14 px en vistas clínicas, 16 px en portal; números clave en tamaño grande y tabular. |
| Notificaciones en línea y *toasts* con severidad (Carbon) | Alertas clínicas en el banner del paciente; *toasts* solo para confirmaciones, nunca para alertas críticas. |
| Paleta de comandos `⌘K / Ctrl+K` (Linear) | Buscar paciente por documento o nombre, abrir órdenes, saltar a módulos, ejecutar acciones ("nueva nota", "pedir hemograma"). |
| Atajos de teclado visibles y navegación sin ratón (Linear) | Atajos por acción (`N` nueva nota, `O` nueva orden, `F` firmar) mostrados en menús y tooltips. |
| Barra lateral colapsable con secciones (ambos) | Navegación por módulo según rol; colapsa a íconos en tableta. |

## 2. Tokens propios

| Token | Valor |
|---|---|
| Color primario | `#0F6E6E` (verde azulado clínico "Vértice") · hover `#0B5858` |
| Acento | `#4B5FD6` (índigo, para IA: todo lo propuesto por la IA lleva este color y el ícono ✦) |
| Neutros | escala **slate** de Tailwind |
| Éxito / alerta / error / crítico | `#1E8E3E` / `#B26A00` / `#C62828` / `#8E0000` con fondo de alto contraste para valores críticos |
| Triage (5 niveles) | I `#B71C1C` · II `#E65100` · III `#F9A825` · IV `#2E7D32` · V `#1565C0` — siempre con número y texto |
| Tipografía | **IBM Plex Sans** (interfaz) + **IBM Plex Mono** (códigos CIE-10/CUPS, valores de laboratorio) — Google Fonts, licencia OFL |
| Radio | `0.375rem` (sobrio, poco redondeado) |
| Densidad | Compacta (filas de 32 px) en personal clínico; normal (44 px) en portal y tableta |
| Modo oscuro | Sí (turnos nocturnos, salas de imágenes); conmutado por usuario |
| Foco | Anillo de 2 px `#4B5FD6` siempre visible |

Variables CSS en `src/styles.css` (`--primary`, `--accent`, `--radius`,
`--ai`…) consumidas por Tailwind (`tailwind.config.js`) y spartan/ui.

## 3. Navegación

- **Barra superior**: sede y servicio activos (conmutables), paleta de comandos, bandeja de tareas (firmas pendientes, sugerencias IA por revisar, resultados críticos), usuario.
- **Barra lateral por rol**:
  - Clínico: Mis pacientes · Agenda · Urgencias · Hospitalización · Cirugía · Bandeja IA.
  - Enfermería: Mapa de camas · eMAR · Plan de cuidados · Entrega de turno.
  - Admisiones: Pacientes · Agenda · Admisiones · Egresos.
  - Facturación: Cuentas abiertas · Facturas · RIPS · Glosas.
  - Apoyo: Farmacia · Laboratorio · Imágenes.
  - Dirección: Indicadores · Calidad.
  - TI: Organización · Catálogos · Conectores · Apps SMART y agentes IA · Bitácora.
- **Contexto de paciente**: al abrir un paciente aparece el **banner fijo** (nombre, documento, edad calculada, EPS, alergias, alertas, ubicación, encuentro activo) y pestañas: Resumen · Línea de tiempo · Notas · Órdenes · Resultados · Medicamentos · Documentos · Facturación.
- **Tableta (ronda / eMAR)**: pestañas inferiores (Pacientes · eMAR · Signos · Escanear).
- **Portal del paciente**: navegación superior simple, móvil primero.

## 4. Inventario de pantallas

| Pantalla | Rol | Arquetipo | Módulo (casos de uso) | Etapa |
|---|---|---|---|---|
| Inicio de sesión + MFA | Todos | Formulario centrado | FND | E0 |
| Selección de sede/servicio | Personal | Lista de tarjetas | FND | E0 |
| Organización, sedes, servicios, ubicaciones | TI | Árbol + detalle | FND | E0 |
| Miembros y roles | TI | Tabla con acciones | FND | E0 |
| Bitácora de auditoría | TI, Calidad | Tabla densa con filtros | FND | E0 |
| Romper el vidrio (acceso de emergencia) | Clínico | Diálogo con justificación | FND | E0 |
| Búsqueda / registro de paciente | Admisiones | Búsqueda + formulario por pasos | PAC | E1 |
| Fusión de duplicados | Admisiones, TI | Comparación lado a lado | PAC | E1 |
| Agenda | Admisiones, Clínico | Calendario por profesional | ADM | E1 |
| Admisión ambulatoria | Admisiones | Formulario corto | ADM | E1 |
| Resumen del paciente | Clínico | Tablero de tarjetas | HCE | E1 |
| Línea de tiempo clínica | Clínico | Línea de tiempo filtrable | HCE | E1 |
| Editor de nota clínica | Clínico | Formulario largo por secciones + panel IA lateral | HCE, IAC | E1-E2 |
| Prescripción y órdenes | Clínico | Formulario con buscador de catálogo y alertas en línea | ORD | E1 |
| Estado de envíos RDA | TI, Calidad | Tabla con estados y reintento | INT | E1 |
| Bandeja de sugerencias IA | Clínico | Lista + comparación borrador/editado | IAC | E2 |
| Panel de escriba ambiental | Clínico | Grabación + transcripción + borrador | IAC | E2 |
| Agentes IA y servidor MCP | TI | Tabla con scopes y consumo | IAC | E2 |
| Cuentas abiertas y cargos | Facturación | Tabla densa con acciones en lote | FAC | E3 |
| Factura + RIPS (validación) | Facturación | Detalle con lista de errores | FAC | E3 |
| Tablero de glosas | Facturación | Kanban por estado | FAC | E3 |
| Tablero de urgencias | Clínico, Enfermería | Tablero en tiempo real | URG | E4 |
| Triage | Enfermería | Formulario guiado | URG | E4 |
| Mapa de camas | Enfermería, Admisiones | Cuadrícula por piso con estados | HOS | E4 |
| eMAR (tableta) | Enfermería | Lista horaria + escáner | HOS | E4 |
| Entrega de turno | Enfermería | Lista por paciente con resumen IA | HOS, IAC | E4 |
| Validación farmacéutica y dispensación | Farmacia | Cola de trabajo | FAR | E4 |
| Inventario de farmacia | Farmacia | Tabla por bodega y lote | FAR | E4 |
| Resultados de laboratorio | Clínico | Tabla con tendencias y valores críticos | APO | E5 |
| Informes de imágenes | Clínico | Detalle con enlace al visor DICOMweb | APO | E5 |
| Programación de quirófanos | Cirugía | Calendario por sala | CIR | E5 |
| Lista de verificación de cirugía segura | Cirugía | Lista por fases | CIR | E5 |
| Conectores Edge y mensajes HL7 v2 | TI | Tabla de mensajes con reproceso | INT | E5 |
| Portal: inicio, citas, resultados, accesos | Paciente | Tarjetas móviles | POR | E6 |
| Registro de apps SMART | TI | Tabla + formulario | INT | E6 |
| Tableros de indicadores | Dirección | Tablero de KPIs | ANA | E6 |
| Referencia y contrarreferencia | Clínico | Bandeja de entrada/salida | RED | E7 |

## 5. Tono de los textos

- **Personal**: imperativo breve y vocabulario clínico estándar. Botones: "Firmar nota", "Pedir", "Administrar", "Radicar". Errores: qué pasó + cómo resolverlo ("La dosis supera el máximo diario (4 g). Ajuste la dosis o justifique.").
- **IA**: siempre identificada ("✦ Borrador de IA — revise antes de firmar"); nunca en primera persona humana; muestra de qué datos salió ("Basado en 3 notas y 12 resultados de los últimos 30 días").
- **Portal del paciente**: usted, lenguaje claro sin siglas ("Su resultado de glucosa está listo").
- Estados vacíos con la acción siguiente ("No hay órdenes pendientes. Pulse O para crear una.").

## 6. Estados

- **Cargando**: esqueletos con la forma de la tabla/tarjeta; nunca bloquear el banner del paciente.
- **Sin conexión**: banda superior ámbar "Sin conexión — sus cambios se guardan como borrador local"; eMAR sigue operando con cola local y sincroniza al reconectar.
- **Error**: mensaje en línea junto al campo; error de servidor con identificador de traza para soporte.
- **Éxito**: confirmación discreta; las acciones irreversibles (firmar, administrar, radicar) muestran resumen y piden confirmación.
- **Crítico**: valores críticos y alertas de alergia con fondo rojo oscuro, ícono y texto, y exigen acuse.
- **Contenido IA**: borde y etiqueta en color `--ai`, con botones "Aceptar", "Editar" y "Descartar"; nada IA se guarda como firmado sin aceptación explícita.
