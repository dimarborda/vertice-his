# Vértice HIS

Sistema de información hospitalaria FHIR-first con IA nativa, diseñado con
el método de diseño express (skill `diseno-express-de-apps` de
`hourus-skills`) en su **variante Go + Angular + Vercel**. Hoy el proyecto
está en **fase de diseño**: no hay código todavía. Explica cada cambio en
una frase sin jerga.

## Fuente de verdad

Los documentos de `docs/` mandan sobre cualquier suposición:

| Archivo | Contenido |
|---|---|
| `docs/00-decisiones.md` | Qué se construye, referentes, diseño, modelo de negocio y variantes |
| `docs/01-referente-conceptual.md` | Análisis de TrakCare + matriz de los cinco referentes |
| `docs/02-referente-casos-de-uso.md` | Catálogo de casos del referente |
| `docs/03-diseno-visual.md` | Tokens, navegación e inventario de pantallas (Angular + spartan/ui) |
| `docs/04-conceptual.md` | Concepto, ciclo, módulos, modelo de datos, fases, cumplimiento |
| `docs/05-casos-de-uso.md` | Backlog por etapas con estado ⬜/🔄/✅ |
| `docs/06-lecciones-aprendidas.md` | Reglas aprendidas de las auditorías (se crea con la primera) |
| `docs/07-variante-stack.md` | Arquitectura Go + Angular + Vercel Services (manda sobre "stack") |
| `docs/08-interoperabilidad-e-ia.md` | FHIR, HL7 v2, RDA, SMART, CDS Hooks, IA y servidor MCP |
| `docs/09-lean-healthcare.md` | Capa Lean (M16 · FLU): hitos de flujo, andon, kaizen, reglas |
| `docs/auditoria/` | Informes de auditoría por fecha |

Al terminar un caso de uso, actualiza su estado en `docs/05-casos-de-uso.md`.

## Stack (variante, ver `docs/07-variante-stack.md`)

- Frontend: Angular (standalone, signals) + Tailwind + spartan/ui en `web/`, estático en Vercel.
- Backend: Go 1.24+, módulo único en `backend/`, servicios DDD (`auth`, `fhir`, `clinico`, `apoyo`, `facturacion`, `interop`, `ai`) como contenedores en Vercel Services; conector on-premise `backend/edge`.
- Datos: Supabase Postgres (pooler de sesión), recursos FHIR R4 en `jsonb`; Supabase Auth para identidad; Storage para adjuntos y audio.

## Reglas

- FHIR R4 es la verdad clínica: nada clínico se guarda fuera de `recursos_fhir`.
- Lo firmado no se edita (nueva versión + nota aclaratoria). `bitacora` y `recursos_fhir` sin `update`/`delete`.
- La IA propone; un profesional identificado aprueba. Toda salida IA genera `Provenance`.
- Lean mide el proceso, no a la persona; ningún hito de flujo exige digitación extra (RG-12).
- Migraciones solo SQL en `db/migrations/`; RLS activado en todas las tablas en la misma migración.
- Nunca secretos en el código ni en `docs/`; nunca datos reales de pacientes fuera de producción (usar Synthea).
- `go vet ./... && go test ./...` y `ng build` sin errores antes de cada push.
- Ramas `etapa-eN`; producción solo con aprobación explícita del dueño y protocolo pre/post despliegue.
- Ninguna operación sobre bases de datos o despliegues sin autorización escrita (acción, ambiente, riesgo, reversibilidad).
- Tablas en español, `snake_case`, plural.
- Antes de cada etapa, lee `docs/06-lecciones-aprendidas.md` (si existe). Al cerrar cada etapa, auditoría con la skill `auditoria-express-de-apps` (adaptando sus chequeos de Next.js/Supabase a Go/Angular).
