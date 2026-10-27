# Vértice HIS · Variante de stack: Go + Angular + Vercel

Versión 1 · 2026-10-27

> La skill `diseno-express-de-apps` fija Next.js + Supabase + Vercel. Este
> documento registra la **variante** que usa Vértice, calcada del patrón ya
> probado en el proyecto `n4na_digital` (servicios Go en contenedor sobre
> **Vercel Services**, panel Angular estático en Vercel, Postgres en
> Supabase). Todo lo demás del método (documentos, etapas, auditorías) se
> mantiene. Donde `docs/04` o `CLAUDE.md` dicen "stack", manda este archivo.

## 1. Por qué la variante

| Necesidad del HIS | Por qué Go + Angular la cubre mejor que Next.js solo |
|---|---|
| Dominio grande y regulado (16 módulos) con reglas de negocio densas | Go con diseño DDD por servicio: tipos estrictos, binarios pequeños, concurrencia para colas de mensajería clínica. |
| Interoperabilidad (FHIR, HL7 v2, RDA) y procesos de larga vida | Servicios Go dedicados (`fhir`, `interop`) y un **binario Edge** que corre dentro de la red del hospital: el mismo lenguaje en la nube y on-premise. |
| Aplicación de escritorio densa, de uso diario de 8-12 horas | Angular: formularios reactivos tipados, signals, CDK (tablas virtuales, atajos, accesibilidad), estructura opinada para equipos grandes. |
| Equipo y conocimiento existente | Mismo stack que `n4na_digital`: scripts, Dockerfiles y reglas de despliegue reutilizables. |

## 2. Vista general

```
                    ┌────────────────────────── Vercel ───────────────────────────┐
 Navegador /        │  Proyecto vertice-web (Angular estático)                    │
 tableta  ────────► │    /api/*  ──rewrite──►  Proyecto vertice-api               │
                    │                           (Vercel Services, contenedores Go)│
 Apps SMART,        │    auth · fhir · clinico · apoyo · facturacion · interop · ai│
 agentes MCP ─────► │                           │            │           │        │
                    └───────────────────────────┼────────────┼───────────┼────────┘
                                                ▼            ▼           ▼
                                   Supabase Postgres   Supabase     Vercel AI Gateway
                                   (pooler de sesión)  Storage/Auth  → modelos Claude
                                                ▲
   ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ red del hospital ─ ─ ─ ┼ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─ ─
   LIS · RIS/PACS · monitores ──HL7 v2/MLLP──► Vértice Edge (binario Go) ──HTTPS──► interop
   IHCE MinSalud ◄──FHIR (RDA)── interop        Proveedor FEV/DIAN ◄──API── facturacion
```

## 3. Servicios Go

| Servicio | Prefijos de API | Módulos | Responsabilidad |
|---|---|---|---|
| `auth` | `/auth`, `/oauth2`, `/.well-known/smart-configuration` | FND | Sesión sobre Supabase Auth (OIDC + MFA), membresías por sede/servicio, **servidor OAuth 2.0 / SMART on FHIR** (scopes, *launch context*), romper el vidrio. |
| `fhir` | `/fhir/r4` | INT | Servidor FHIR R4: CRUD, búsqueda, `_history`, `$everything`, `Bundle` transaccional, validación contra perfiles colombianos. Escribe en `recursos_fhir` + índices de búsqueda. |
| `clinico` | `/pacientes`, `/agenda`, `/encuentros`, `/notas`, `/ordenes`, `/urgencias`, `/camas`, `/cirugia` | PAC, ADM, HCE, ORD, URG, HOS, CIR | Flujos de trabajo clínicos. Persiste lo clínico **a través del paquete de dominio FHIR compartido** (no duplica la verdad) y lo operativo en tablas propias. SSE para tableros en tiempo real. |
| `apoyo` | `/farmacia`, `/laboratorio`, `/imagenes` | FAR, APO | Validación farmacéutica, dispensación, inventario, resultados e informes. |
| `facturacion` | `/cuentas`, `/facturas`, `/rips`, `/glosas` | FAC | Cargos, tarifarios, FEV vía proveedor tecnológico, RIPS JSON, glosas. |
| `interop` | `/interop`, `/cds-services`, `/edge` | INT | RDA → IHCE, suscripciones FHIR, CDS Hooks, relé del conector Edge, outbox de eventos. |
| `ai` | `/ai`, `/mcp` | IAC | Escriba, resúmenes, codificación, revisión pre-factura, **servidor MCP** (Streamable HTTP) sobre FHIR con permisos SMART; registra `Provenance`. |
| `edge` *(fuera de Vercel)* | — | INT | Binario Go instalado en la red del hospital: escucha **MLLP** (HL7 v2), traduce a FHIR/JSON y lo envía por HTTPS a `interop`; cola local persistente si cae internet. |

Por qué `edge` existe: las funciones de Vercel atienden HTTP(S)/WebSockets,
no aceptan conexiones TCP crudas entrantes como MLLP, y los equipos de
laboratorio/imágenes viven en redes privadas. El conector local resuelve
ambas cosas sin abrir puertos del hospital a internet (conexión saliente
con mTLS).

## 4. Estructura del repositorio

```
vertice-his/
├── CLAUDE.md · AGENTS.md · README.md
├── docs/                       # este método (00-08, 06 lecciones, auditoria/)
├── web/                        # Angular (vertice-web)
│   ├── src/app/
│   │   ├── core/               # auth, interceptores, contexto de sede, atajos
│   │   ├── shared/ui/          # spartan/ui personalizado con tokens de 03
│   │   ├── features/           # un folder por módulo: pacientes, hce, ordenes…
│   │   └── fhir/               # tipos FHIR R4 generados + cliente
│   ├── src/styles.css · tailwind.config.js · angular.json
├── backend/                    # módulo Go único (go.mod)
│   ├── pkg/                    # compartido: http, middleware, auditoria, fhirstore,
│   │                           # terminologia, outbox, smartscopes, provenance
│   ├── services/
│   │   ├── auth/ · fhir/ · clinico/ · apoyo/ · facturacion/ · interop/ · ai/
│   │   │   └── cmd/server · internal/{domain,application,infrastructure,interfaces}
│   ├── edge/                   # conector on-premise (cmd/edge)
│   └── Dockerfile.<servicio>.vercel
├── db/
│   ├── migrations/             # SQL versionado (única forma de cambiar el esquema)
│   └── seeds/                  # catálogos CIE-10, CUPS, CUM, DIVIPOLA (datos públicos)
├── infra/
│   ├── vercel/web/vercel.json
│   ├── vercel/api/vercel.json
│   └── docker/docker-compose.yml   # integración local con las mismas imágenes
└── .github/workflows/          # build, go test, ng test, pruebas de RLS, Dependabot
```

## 5. Configuración de Vercel (patrón `n4na_digital`)

`infra/vercel/api/vercel.json` — un proyecto con varios servicios en contenedor:

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "services": {
    "auth":        { "runtime": "container", "root": "auth",        "entrypoint": "Dockerfile.vercel" },
    "fhir":        { "runtime": "container", "root": "fhir",        "entrypoint": "Dockerfile.vercel" },
    "clinico":     { "runtime": "container", "root": "clinico",     "entrypoint": "Dockerfile.vercel" },
    "apoyo":       { "runtime": "container", "root": "apoyo",       "entrypoint": "Dockerfile.vercel" },
    "facturacion": { "runtime": "container", "root": "facturacion", "entrypoint": "Dockerfile.vercel" },
    "interop":     { "runtime": "container", "root": "interop",     "entrypoint": "Dockerfile.vercel" },
    "ai":          { "runtime": "container", "root": "ai",          "entrypoint": "Dockerfile.vercel" }
  },
  "rewrites": [
    { "source": "/auth/(.*)",        "destination": { "service": "auth" } },
    { "source": "/oauth2/(.*)",      "destination": { "service": "auth" } },
    { "source": "/.well-known/(.*)", "destination": { "service": "auth" } },
    { "source": "/fhir/(.*)",        "destination": { "service": "fhir" } },
    { "source": "/(pacientes|agenda|encuentros|notas|ordenes|urgencias|camas|cirugia)(.*)", "destination": { "service": "clinico" } },
    { "source": "/(farmacia|laboratorio|imagenes)(.*)", "destination": { "service": "apoyo" } },
    { "source": "/(cuentas|facturas|rips|glosas)(.*)",  "destination": { "service": "facturacion" } },
    { "source": "/(interop|cds-services|edge)(.*)",     "destination": { "service": "interop" } },
    { "source": "/(ai|mcp)(.*)",     "destination": { "service": "ai" } }
  ],
  "crons": [
    { "path": "/interop/tareas/reintentar-rda",   "schedule": "*/10 * * * *" },
    { "path": "/rips/tareas/reintentar",          "schedule": "*/15 * * * *" },
    { "path": "/glosas/tareas/vencimientos",      "schedule": "0 6 * * *" }
  ]
}
```

`infra/vercel/web/vercel.json` — Angular estático con proxy a la API:

```json
{
  "$schema": "https://openapi.vercel.sh/vercel.json",
  "framework": null,
  "outputDirectory": ".",
  "rewrites": [
    { "source": "/api/:path*", "destination": "__VERTICE_API_ORIGIN__/:path*" },
    { "source": "/(.*)", "destination": "/index.html" }
  ],
  "headers": [
    { "source": "/(.*)", "headers": [
      { "key": "Content-Security-Policy", "value": "default-src 'self'; frame-ancestors 'self'; connect-src 'self'" },
      { "key": "X-Content-Type-Options", "value": "nosniff" },
      { "key": "Referrer-Policy", "value": "strict-origin-when-cross-origin" },
      { "key": "Permissions-Policy", "value": "camera=(self), microphone=(self), geolocation=()" }
    ]}
  ]
}
```

*(El micrófono se permite en el propio origen para el escriba ambiental y
la cámara para leer códigos en eMAR.)* El script de despliegue reemplaza
`__VERTICE_API_ORIGIN__` por la URL del proyecto de API de cada ambiente,
igual que en `n4na_digital`. Si se prefiere, la misma configuración puede
escribirse en `vercel.ts` (`@vercel/config`).

Dockerfile por servicio (copiado como `<servicio>/Dockerfile.vercel` por el script de despliegue):

```dockerfile
FROM golang:1.24-alpine AS build
WORKDIR /src
COPY go.mod go.sum ./
RUN go mod download
COPY pkg ./pkg
COPY services ./services
RUN CGO_ENABLED=0 GOOS=linux go build -trimpath -ldflags="-s -w" -o /out/app ./services/fhir/cmd/server

FROM gcr.io/distroless/static-debian12
COPY --from=build /out/app /app
EXPOSE 80
ENTRYPOINT ["/app"]
```

## 6. Límites de la plataforma que condicionan el diseño

| Límite | Decisión |
|---|---|
| Cuerpo de petición/respuesta de 4.5 MB en funciones | Audio del escriba, PDF e imágenes suben **directo a Supabase Storage** con URL firmada; el servicio recibe solo la referencia. `Bundle` FHIR grandes se paginan. |
| Duración por invocación (300 s por defecto) | Transcripciones y resúmenes largos se procesan como trabajos asíncronos (tabla de trabajos + outbox + Cron); SSE con reconexión para tableros. |
| Sin TCP entrante (MLLP) ni acceso a redes privadas | Conector **Vértice Edge** on-premise con conexión saliente mTLS. |
| Conexiones a Postgres desde instancias efímeras | Pooler de sesión de Supabase, pool pequeño por instancia, consultas con *timeout*. |
| Datos sensibles | Región de Vercel y de Supabase lo más cercana posible (p. ej. `gru1` / `sa-east-1`); residencia en Colombia, si se exige, se resuelve moviendo Postgres sin tocar el código. |

## 7. Datos y seguridad

- **Migraciones solo SQL** en `db/migrations/` (nada de `AutoMigrate` ni *seeding* desde Go; regla heredada de `n4na_digital`).
- RLS activado en todas las tablas en la misma migración que las crea, aunque el acceso sea solo desde Go (defensa en profundidad). El rol de base de datos del backend no es `postgres`/`service_role` sino uno con privilegios mínimos por servicio.
- La clave pública de Supabase no da acceso a ninguna tabla clínica; el navegador solo habla con `/api`.
- Secretos en variables de entorno de Vercel por ambiente (`vercel env`), nunca en el repo ni en `docs/`; `.env.example` sin valores.
- `bitacora` y `recursos_fhir` sin `update`/`delete` para el rol de aplicación.
- Cifrado en tránsito (TLS) y en reposo (Supabase); campos especialmente sensibles (VIH, salud mental, violencia) con etiqueta de seguridad FHIR (`meta.security`) y política de acceso reforzada.

## 8. Ambientes y despliegue

| Ambiente | Para qué | Regla |
|---|---|---|
| `preview` | Cada rama `etapa-eN` | Despliegue automático por push; datos sintéticos (Synthea) únicamente. |
| `staging` | Validación de criterios de salida con el piloto | Solo commits en `main`; datos sintéticos o anonimizados. |
| `prod` | Operación real | Solo con autorización explícita del dueño; protocolo pre/post despliegue (informe de cambios, confirmación, commit `deploy: prod AAAA-MM-DD — resumen`). |

Local: `docker compose up --build` levanta los 7 servicios con las mismas
imágenes que Vercel y un proxy en `http://localhost:8080/api`; Angular con
`ng serve --configuration local`.

## 9. Equivalencias con la guía de construcción de la skill

| Guía original (Next.js + Supabase) | En esta variante |
|---|---|
| `create-next-app` + shadcn | `ng new web --style=css --routing --ssr=false` + Tailwind + spartan/ui |
| Server Actions / Route Handlers | Servicios Go (`internal/interfaces/http`) |
| `@supabase/ssr` en el navegador | Sin cliente Supabase de datos en el navegador; Supabase Auth solo para iniciar sesión (PKCE) y el backend valida el JWT |
| `supabase db push` | `db/migrations/*.sql` aplicadas con el script de BD tras autorización |
| `supabase gen types` | Tipos FHIR R4 generados para Go y TypeScript desde las definiciones oficiales |
| AI SDK | Cliente HTTP de Go hacia Vercel AI Gateway (API compatible) o SDK oficial de Anthropic para Go |
| `npm run build` antes de push | `go vet ./... && go test ./...` + `ng build` + `ng test` |
