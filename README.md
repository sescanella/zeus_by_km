# Zeus by KM

**Zeus by KM** es un sistema integral de gestión de producción industrial especializado en la planificación y ejecución de **Notas de Venta (NV)**, **spools** y sus **uniones** correspondientes. Diseñado para optimizar la asignación de responsables (armadores, soldadores RA/RE, QAQC) y el seguimiento completo del ciclo de soldadura hasta el cierre automático de proyectos.

El sistema implementa un flujo jerárquico robusto (NV → Spool → Unión) con transiciones de estado automáticas, auditoría completa y gestión de roles granular, permitiendo a las empresas industriales mantener control total sobre sus procesos de producción con trazabilidad completa.

## 🚀 Características principales

- **Gestión jerárquica completa**: NV → Spools → Uniones con estados automáticos
- **Sistema de roles granular**: Admin, Organizador, QAQC, Armador, Soldador
- **Bandeja de tareas unificada**: Vista centralizada de pendientes por rol (powered by `vw_responsables_pendientes`)
- **Transiciones automáticas**: Estados calculados según confirmaciones y validaciones
- **Auditoría completa**: Seguimiento temporal de cambios con campos set-once
- **Gestión de planos**: Upload/preview de documentos técnicos con almacenamiento seguro
- **Validaciones de integridad**: Triggers de BD que garantizan consistencia de datos
- **Jobs asíncronos**: Generación automática de uniones y transiciones de estado
- **Seguridad RLS**: Row Level Security para acceso granular por rol
- **Multi-timezone**: Almacenamiento UTC con presentación local (America/Santiago)

## 🛠️ Tecnologías utilizadas

### Stack Principal
- **Lenguaje**: TypeScript
- **Frontend**: Next.js 14 (App Router) + React + Tailwind CSS
- **Backend**: Supabase (PostgreSQL + Auth + Storage + RLS)
- **Jobs Asíncronos**: Node.js + pg-boss
- **Arquitectura**: Domain-Driven Design (DDD) light

### Base de Datos y Seguridad
- **PostgreSQL**: 15+ con extensiones citext, pg_trgm
- **Row Level Security (RLS)**: Políticas granulares por rol
- **Triggers**: Validaciones automáticas y auditoría
- **Storage**: Bucket privado para planos con URLs firmadas

### Herramientas de Desarrollo
- **Gestión de código**: ESLint + Prettier + TypeScript strict
- **Testing**: Jest + Testing Library + Playwright E2E
- **Monorepo**: Estructura modular con packages compartidos

## 📊 Estados y transiciones del sistema

El sistema maneja transiciones automáticas basadas en confirmaciones de responsables:

```
NV (Nota de Venta)
├── nv_pendiente ────────────┐
├── nv_en_proceso ───────────┤ Estados automáticos
└── nv_listo ────────────────┘ basados en spools

SPOOL
├── spool_pendiente ─────────┐ (asignar armador + todas uniones con RA/RE)
├── spool_por_armar ─────────┤ (confirmar armador)
├── spool_armado ────────────┤ (todas uniones listas)
├── spool_soldado ───────────┤ (confirmar qaqc)
└── spool_listo ─────────────┘ → genera spool_ft

UNIÓN
├── union_pendiente ─────────┐ (asignar RA y RE)
├── union_espera_armado ─────┤ (spool_armador_ft existe)
├── union_por_soldar ────────┤ (confirmar RA_ft y RE_ft)
└── union_listo ─────────────┘ → genera union_ft

Flujo: NV contiene Spools → Spools contienen Uniones
       Estados superiores dependen de inferiores completados
```

### Jobs asíncronos activos
- **`crear_uniones`**: Genera N uniones automáticamente al crear spool
- **`state_transitions`**: Procesa transiciones de estado jerárquicas

## 🔐 Matriz de permisos por rol

| Operación | Admin | Organizador | Armador | Soldador | QAQC |
|-----------|-------|-------------|---------|----------|------|
| **NV** |
| Crear NV | ✅ | ❌ | ❌ | ❌ | ❌ |
| Editar NV | ✅ | ❌ | ❌ | ❌ | ❌ |
| Ver NV | ✅ | ✅ | ✅ | ✅ | ✅ |
| **Spools** |
| Crear spool | ✅ | ❌ | ❌ | ❌ | ❌ |
| Asignar armador | ✅ | ✅ | ❌ | ❌ | ❌ |
| Asignar qaqc | ✅ | ❌ | ❌ | ❌ | ❌ |
| Confirmar armado | ✅ | ❌ | ✅* | ❌ | ❌ |
| Confirmar qaqc | ✅ | ❌ | ❌ | ❌ | ✅* |
| Subir/editar planos | ✅ | ❌ | ❌ | ❌ | ❌ |
| **Uniones** |
| Asignar soldadores | ✅ | ✅ | ❌ | ❌ | ❌ |
| Confirmar RA | ✅ | ❌ | ❌ | ✅* | ❌ |
| Confirmar RE | ✅ | ❌ | ❌ | ✅* | ❌ |
| **Usuarios** |
| Gestionar usuarios | ✅ | ❌ | ❌ | ❌ | ❌ |
| Asignar roles | ✅ | ❌ | ❌ | ❌ | ❌ |

*Solo sobre tareas asignadas al usuario específico

### Bandejas de tareas por rol
- **Admin/Organizador**: Ven tareas de asignación + propias confirmaciones
- **Armador/Soldador/QAQC**: Solo sus confirmaciones pendientes
- **Vista fuente**: `vw_responsables_pendientes` filtra por usuario/rol automáticamente

## 📦 Instalación y configuración

### Prerrequisitos
- Node.js 18+ y npm/yarn
- PostgreSQL 15+ (o cuenta de Supabase)
- Cuenta de Supabase configurada

### 1. Clonar el repositorio
```bash
git clone https://github.com/sescanella/zeus_by_km.git
cd zeus_by_km
```

### 2. Instalar dependencias
```bash
npm install
# o
yarn install
```

### 3. Configurar variables de entorno
Copia y configura los archivos de entorno:

```bash
# Para desarrollo
cp config/stages/dev.env.example .env.local

# Para workers
cp config/stages/dev.env.example .env.workers
```

### Variables requeridas (.env.local):
```env
# Supabase (Frontend)
NEXT_PUBLIC_SUPABASE_URL=https://tu-proyecto.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=tu_anon_key

# Configuración de la app
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_TZ=America/Santiago
```

### Variables requeridas (.env.workers):
```env
# Supabase (Backend/Workers)
SUPABASE_URL=https://tu-proyecto.supabase.co
SUPABASE_SERVICE_ROLE_KEY=tu_service_role_key

# pg-boss
PG_BOSS_CONNECTION_STRING=postgresql://user:pass@host:5432/db
PG_BOSS_SCHEMA=pgboss

# Workers
UNIONES_WORKER_CONCURRENCY=5
TRANSITIONS_WORKER_CONCURRENCY=3
```

### 4. Configurar base de datos
```bash
# Ejecutar migraciones y seeds
npm run db:migrate
npm run db:seed
```

### 5. Configurar Storage (Supabase)
```sql
-- Crear bucket para planos
INSERT INTO storage.buckets (id, name, public) VALUES ('planos', 'planos', false);

-- Configurar CORS (en Supabase Dashboard)
-- Añadir tu dominio local: http://localhost:3000
```

### 6. Ejecutar en desarrollo
```bash
# Terminal 1: Aplicación web
npm run dev

# Terminal 2: Workers (en otra terminal)
npm run workers:dev
```

La aplicación estará disponible en `http://localhost:3000`

## ▶️ Uso

### Acceso inicial
1. **Crear primer usuario admin** en Supabase Auth
2. **Registrar en tabla usuarios** con rol admin
3. Acceder a `/usuarios` para gestionar el equipo

### Flujo principal
1. **Crear NV** (`/nv/nuevo`) - Solo admin
2. **Crear spools** (`/nv/:id/spools/nuevo`) - Solo admin
3. **Asignar responsables** (`/mis-tareas/responsables`) - Organizador/Admin
4. **Confirmar trabajos** (`/mis-tareas`) - Armador/Soldadores/QAQC
5. **Seguimiento** en dashboards de NV/Spool

### Comandos principales
```bash
# Desarrollo
npm run dev              # Aplicación web
npm run workers:dev      # Workers asíncronos

# Base de datos
npm run db:push          # Aplicar cambios de esquema
npm run db:migrate       # Ejecutar migraciones
npm run db:seed          # Poblar datos iniciales

# Testing
npm run test             # Tests unitarios
npm run test:e2e         # Tests end-to-end

# Producción
npm run build            # Build optimizado
npm run start            # Servidor de producción
```

### Estructura de roles
- **Admin**: Control total del sistema, crear NV/spools, asignar QAQC
- **Organizador**: Asignar armadores y soldadores, supervisión
- **QAQC**: Validación final de spools completados  
- **Armador**: Confirmación de armado de spools
- **Soldador**: Confirmación de soldaduras RA/RE en uniones

## 🚀 Deploy en producción

### Opción recomendada (Supabase + Vercel + Railway)

#### 1. Base de datos y Storage (Supabase)
```bash
# 1. Crear proyecto en Supabase
# 2. Ejecutar migraciones
supabase db push

# 3. Configurar Storage bucket
# 4. Aplicar políticas RLS
supabase db reset --local
```

#### 2. Frontend (Vercel)
```bash
# 1. Conectar repo en Vercel
# 2. Configurar variables de entorno:
NEXT_PUBLIC_SUPABASE_URL=https://xxx.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=xxx
NEXT_PUBLIC_APP_URL=https://tu-app.vercel.app
NEXT_PUBLIC_TZ=America/Santiago

# 3. Deploy automático desde main branch
```

#### 3. Workers (Railway/Render)
```bash
# 1. Crear servicio en Railway
# 2. Configurar variables:
SUPABASE_URL=xxx
SUPABASE_SERVICE_ROLE_KEY=xxx
PG_BOSS_CONNECTION_STRING=postgresql://...

# 3. Comando de inicio:
npm run workers:start
```

### Alternativa completa en VPS
```bash
# Docker Compose con PostgreSQL, Redis, Next.js y Workers
docker-compose -f docker-compose.prod.yml up -d
```

### Checklist pre-producción
- [ ] Migraciones aplicadas y verificadas
- [ ] Políticas RLS configuradas y probadas
- [ ] Bucket de Storage configurado con CORS
- [ ] Variables de entorno productivas configuradas
- [ ] Jobs asíncronos funcionando (verificar pg-boss)
- [ ] Primer usuario admin creado
- [ ] Monitoreo y logs configurados

## 🏗️ Arquitectura del proyecto

```
.
├─ apps/
│  ├─ web/                                  # Next.js (App Router)
│  │  ├─ app/                               # rutas/páginas + server actions
│  │  │  ├─ (public)/
│  │  │  ├─ mis-tareas/
│  │  │  ├─ nv/
│  │  │  ├─ spool/
│  │  │  ├─ uniones/
│  │  │  ├─ usuarios/
│  │  │  └─ admin/
│  │  ├─ features/                          # DDD light por dominio
│  │  │  ├─ nv/{ui,application,domain,infrastructure}
│  │  │  ├─ spool/{ui,application,domain,infrastructure}
│  │  │  ├─ uniones/{ui,application,domain,infrastructure}
│  │  │  ├─ usuarios/{ui,application,domain,infrastructure}
│  │  │  └─ roles/{ui,application,domain,infrastructure}
│  │  ├─ components/                        # UI cross-domain (tablas, formularios)
│  │  ├─ lib/                               # adapters genéricos (auth, supabase, rls)
│  │  ├─ hooks/                             # hooks compartidos (useRole, usePaginatedQuery)
│  │  ├─ styles/                            # Tailwind globals, tokens
│  │  ├─ types/                             # Tipos/DTO UI (reexport de packages/core-contracts)
│  │  ├─ public/                            # assets estáticos
│  │  └─ test/
│  │     ├─ unit/
│  │     ├─ integration/
│  │     └─ e2e/
│  └─ workers/                              # Procesos pg-boss
│     ├─ queues/                            # nombres/opciones/DLQ por cola
│     │  ├─ state-transitions/
│     │  └─ crear-uniones/
│     ├─ jobs/                              # contratos/payloads documentados
│     │  ├─ state_transitions.job.md
│     │  └─ crear_uniones.job.md
│     ├─ handlers/                          # lógica de procesamiento por job
│     │  ├─ state-transitions.handler.ts
│     │  └─ crear-uniones.handler.ts
│     ├─ schedulers/                        # cron tasks/reenqueues
│     ├─ infra/                             # conexión pg-boss, observabilidad
│     └─ test/
│        ├─ unit/
│        └─ integration/
├─ packages/                                # módulos compartidos
│  ├─ core-domain/                          # entidades/VO/servicios de dominio (puro TS)
│  ├─ core-contracts/                       # Zod schemas, DTOs, tipos (web/workers)
│  ├─ infra-supabase/                       # cliente y repos base Supabase + helpers RLS
│  └─ ui-kit/                               # componentes UI reutilizables (RoleFilteredSelect, UploadPlano)
├─ db/                                      # PostgreSQL (Supabase) versionado
│  ├─ init/                                 # extensiones, schemas base, bootstrap
│  ├─ migrations/                           # YYYYMMDDThhmmss__{tipo}__{dominio}.sql
│  ├─ policies/                             # RLS central (policies.sql y por tabla)
│  ├─ functions/                            # fn_* (negocio/seguridad; SECURITY DEFINER)
│  ├─ views/                                # vw_* (contratos para UI/workers)
│  ├─ triggers/                             # tr_* y funciones de trigger
│  └─ seed/                                 # datos mínimos (roles, admin, catálogos)
├─ docs/                                    # documentación viva
│  ├─ adr/                                  # Architecture Decision Records
│  ├─ specs/                                # especificaciones (vistas, flujos, contratos)
│  └─ handbooks/                            # guías de contribución, estándares
├─ scripts/                                 # utilidades CLI (sin secretos)
│  ├─ dev
│  ├─ test
│  ├─ lint
│  ├─ db:push
│  ├─ db:migrate
│  ├─ db:seed
│  └─ queues:monitor
├─ config/                                  # gestión de entornos/variables
│  ├─ env.schema.ts                         # validación/mapeo (Zod)
│  ├─ env.web.ts                            # NEXT_PUBLIC_* (solo públicas)
│  ├─ env.workers.ts                        # secretos workers (service role, pg-boss)
│  └─ stages/
│     ├─ dev.env.example
│     ├─ stg.env.example
│     └─ prod.env.example
├─ .github/
│  └─ workflows/
│     ├─ ci-web.yml                         # lint, typecheck, unit web
│     ├─ ci-workers.yml                     # lint, unit/integration workers
│     ├─ db-validate.yml                    # lint SQL + orden migraciones/policies
│     └─ preview-deploy.yml                 # despliegues de preview (opcional)
├─ .tooling/                                # configuración de calidad
│  ├─ eslint/
│  ├─ prettier/
│  └─ tsconfig/
├─ .vscode/                                 # recomendaciones de workspace (opcional)
├─ README.md
└─ LICENSE
```

### Componentes clave y su ubicación

#### UI Kit (`packages/ui-kit/`)
- **`RoleFilteredSelect`**: Selector de usuarios por rol activo
  - *Usado en*: `/spool/:id` (asignar armador/qaqc), `/union/:id` (asignar RA/RE)
  - *Props*: `rolesSlug`, `context`, `excludeUsuarioIds`
  
- **`UploadPlano`**: Gestión de archivos con Supabase Storage
  - *Usado en*: `/nv/:id/spools/nuevo`, `/spool/:id`, `/nv/:id/planos`
  - *Funcionalidad*: Upload seguro, URLs firmadas, validación de tipos

#### Vistas especializadas (`db/views/`)
- **`vw_responsables_pendientes`**: Fuente de datos para bandejas de tareas
  - *Alimenta*: `/mis-tareas` y `/mis-tareas/responsables`
  - *Filtra por*: usuario, rol, tipo de tarea, estado
  - *Optimizada con*: índices parciales en tablas base

#### Jobs asíncronos (`apps/workers/`)
- **`crear_uniones.job`**: Generación automática de uniones por spool
- **`state_transitions.job`**: Transiciones automáticas de estado

## 📋 Convenciones de nombres

### Base de datos (SQL)
- **Tablas**: `snake_case` singular (`nv`, `spool`, `uniones`)
- **Columnas**: `snake_case` con prefijo de tabla (`nv_codigo`, `spool_estado`)
- **Triggers**: prefijo `tr_` + tabla + acción (`tr_spool_audit`)
- **Funciones**: prefijo `fn_` + descripción (`fn_require_updated_by`)
- **Vistas**: prefijo `vw_` + propósito (`vw_responsables_pendientes`)
- **Índices**: sufijo `_idx` (`spool_estado_activa_idx`)

### Aplicación (TypeScript)
- **Componentes**: `PascalCase` (`RoleFilteredSelect`, `UploadPlano`)
- **Funciones**: `camelCase` (`createSpool`, `assignArmador`)
- **Variables**: `camelCase` (`spoolData`, `unionEstado`)
- **Tipos**: `PascalCase` (`SpoolData`, `UnionState`)
- **Constantes**: `SCREAMING_SNAKE_CASE` (`DEFAULT_PAGE_SIZE`)

### Archivos y directorios
- **Componentes**: `PascalCase.tsx` (`SpoolDetail.tsx`)
- **Hooks**: `use` + `PascalCase` (`useSpoolData.ts`)
- **Utils**: `camelCase.ts` (`dateFormatters.ts`)
- **Features**: `kebab-case` (`nv/`, `spool/`, `uniones/`)

### Estados y campos especiales
- **Estados**: formato `{entidad}_{estado}` (`nv_pendiente`, `spool_listo`)
- **Campos finales**: sufijo `_ft` (fecha terminal) (`spool_ft`, `union_ft`)
- **Auditoría**: `created_at/by`, `updated_at/by` (siempre en UTC)
- **Vigencia**: `activa|cancelada` (para soft delete)

## 📚 Guía de documentación

### Estructura `/docs`

| Directorio | Propósito | Ejemplos |
|------------|-----------|----------|
| **`adr/`** | Architecture Decision Records | `001-supabase-rls.md`, `002-pg-boss-jobs.md` |
| **`specs/`** | Especificaciones técnicas detalladas | `vistas-y-rutas.md`, `jobs-asincronos.md` |
| **`handbooks/`** | Guías de desarrollo y estándares | `contributing.md`, `sql-style-guide.md` |

### Documentos clave disponibles
- **`mapa_vistas_y_rutas.txt`**: Especificación completa de todas las rutas UI
- **`job_*.txt`**: Documentación detallada de workers asíncronos  
- **`tbl_*.txt`**: Esquemas DDL completos de todas las tablas
- **`policies.sql.txt`**: Políticas RLS implementadas
- **`*_componente.txt`**: Especificaciones de componentes UI reutilizables

### Para contribuidores
1. **Antes de implementar**: Revisar `/docs/specs/` para entender el contexto
2. **Cambios arquitectónicos**: Crear ADR en `/docs/adr/`
3. **Nuevas features**: Actualizar documentación correspondiente
4. **Modificaciones BD**: Actualizar especificaciones `tbl_*.txt`

## 🤝 Contribución

### Reglas de contribución
1. **Fork** el repositorio y crea una rama feature
2. **Commits semánticos**: `feat:`, `fix:`, `docs:`, `refactor:`
3. **Tests**: Asegúrate de que pasen todos los tests
4. **Linting**: Ejecuta `npm run lint` antes del commit
5. **PR template**: Incluye descripción detallada de cambios

### Estándares de código
- **TypeScript strict mode** habilitado
- **ESLint + Prettier** configurados
- **Arquitectura DDD** light por dominio
- **Naming**: PascalCase para componentes, camelCase para funciones
- **SQL**: snake_case para tablas/columnas, triggers con prefijo `tr_`

### Proceso de desarrollo
```bash
# 1. Crear rama feature
git checkout -b feature/nueva-funcionalidad

# 2. Desarrollo con validación continua
npm run lint        # Verificar estilo
npm run test        # Tests unitarios
npm run type-check  # Verificación de tipos

# 3. Commit y push
git commit -m "feat: añadir nueva funcionalidad X"
git push origin feature/nueva-funcionalidad

# 4. Crear Pull Request
# Incluir: descripción, tests, screenshots si aplica
```

### Issues y bugs
- **Usar templates** proporcionados para bugs/features
- **Incluir logs** y pasos para reproducir
- **Etiquetar apropiadamente**: bug, enhancement, question
- **Asignar prioridad**: crítico, alto, medio, bajo

## 📄 Licencia

Este proyecto está licenciado bajo la **Licencia MIT**. 

```
Copyright (c) 2025 Kronos Mining Ingeniería y Revestimiento Ltda.

Permission is hereby granted, free of charge, to any person obtaining a copy
of this software and associated documentation files (the "Software"), to deal
in the Software without restriction, including without limitation the rights
to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
copies of the Software, and to permit persons to whom the Software is
furnished to do so, subject to the following conditions:

The above copyright notice and this permission notice shall be included in all
copies or substantial portions of the Software.

THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT.
```

Ver el archivo [LICENSE](LICENSE) para detalles completos.

---

## 📞 Soporte y contacto

- **Issues**: Reporta bugs o solicita features en GitHub Issues
- **Documentación**: Revisa `/docs` para especificaciones técnicas detalladas
- **Wiki**: Consulta la wiki del proyecto para guías adicionales

**Desarrollado con ❤️ por Kronos Mining Ingeniería y Revestimiento Ltda.**
