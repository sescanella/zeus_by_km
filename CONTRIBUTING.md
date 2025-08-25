# Guía de Contribución - Zeus by KM

¡Gracias por tu interés en contribuir a **Zeus by KM**! Este documento te ayudará a configurar tu entorno de desarrollo y seguir las mejores prácticas del proyecto.

## 🚀 Configuración del entorno de desarrollo

### Prerrequisitos

Asegúrate de tener instalado:

- **Node.js 18+** y npm/yarn
- **PostgreSQL 15+** (o acceso a Supabase)
- **Git** para control de versiones
- Editor con soporte TypeScript (recomendado: VSCode)

### 1. Clonar e instalar

```bash
# Clonar el repositorio
git clone https://github.com/kronos-mining/zeus_by_km.git
cd zeus_by_km

# Instalar dependencias
npm install
```

### 2. Configurar variables de entorno

```bash
# Copiar plantillas de configuración
cp config/stages/dev.env.example .env.local
cp config/stages/dev.env.example .env.workers
```

**Variables mínimas requeridas (.env.local):**
```env
NEXT_PUBLIC_SUPABASE_URL=https://tu-proyecto.supabase.co
NEXT_PUBLIC_SUPABASE_ANON_KEY=tu_anon_key_publico
NEXT_PUBLIC_APP_URL=http://localhost:3000
NEXT_PUBLIC_TZ=America/Santiago
```

**Variables para workers (.env.workers):**
```env
SUPABASE_URL=https://tu-proyecto.supabase.co
SUPABASE_SERVICE_ROLE_KEY=tu_service_role_key_secreto
PG_BOSS_CONNECTION_STRING=postgresql://user:pass@host:5432/db
```

### 3. Configurar base de datos

```bash
# Aplicar migraciones y esquemas
npm run db:migrate

# Poblar datos iniciales (roles, usuarios de prueba)
npm run db:seed
```

### 4. Ejecutar en desarrollo

```bash
# Terminal 1: Aplicación web
npm run dev

# Terminal 2: Workers asíncronos (en otra terminal)
npm run workers:dev
```

La aplicación estará disponible en `http://localhost:3000`

## 📋 Flujo de trabajo para contribuidores

### 1. Crear una nueva rama

```bash
# Crear rama feature desde main
git checkout main
git pull origin main
git checkout -b feature/descripcion-breve

# O rama de bugfix
git checkout -b fix/descripcion-del-bug
```

### 2. Desarrollo y validación

Antes de hacer commit, ejecuta siempre:

```bash
# Verificar linting y formato
npm run lint

# Ejecutar tests unitarios
npm run test

# Verificar tipos TypeScript
npm run type-check

# Ejecutar tests de integración (opcional)
npm run test:integration
```

### 3. Commits semánticos

Usamos [Conventional Commits](https://www.conventionalcommits.org/):

```bash
# Nuevas funcionalidades
git commit -m "feat(spool): añadir validación de estados automáticos"

# Corrección de bugs
git commit -m "fix(uniones): corregir asignación de soldadores RA/RE"

# Cambios en documentación
git commit -m "docs(api): actualizar especificación de endpoints"

# Refactoring sin cambios funcionales
git commit -m "refactor(auth): simplificar validación de roles"

# Mejoras de rendimiento
git commit -m "perf(queries): optimizar consulta de tareas pendientes"

# Tests
git commit -m "test(workers): añadir tests para job crear_uniones"
```

**Scopes recomendados:** `nv`, `spool`, `uniones`, `usuarios`, `auth`, `workers`, `ui`, `db`, `docs`

### 4. Pull Request

```bash
# Subir cambios
git push origin feature/descripcion-breve
```

Crea un PR con:
- **Título descriptivo** siguiendo formato semántico
- **Descripción detallada** de los cambios
- **Screenshots** si hay cambios visuales
- **Checklist** completado (ver template)
- **Referencias** a issues relacionados

## 🏗️ Arquitectura y estándares

### Estructura del proyecto

```
apps/
├── web/                    # Next.js App Router
│   ├── app/               # Rutas y server actions
│   ├── features/          # DDD por dominio
│   ├── components/        # UI cross-domain
│   └── lib/              # Adapters y utilidades
├── workers/              # Jobs asíncronos (pg-boss)
└── packages/             # Módulos compartidos
    ├── core-domain/      # Entidades y servicios
    ├── core-contracts/   # DTOs y schemas (Zod)
    ├── infra-supabase/   # Cliente y repos
    └── ui-kit/          # Componentes reutilizables
```

### Convenciones de código

#### TypeScript/React
- **Componentes**: `PascalCase` (`SpoolDetail.tsx`)
- **Funciones**: `camelCase` (`createSpool`, `validateUser`)
- **Variables**: `camelCase` (`spoolData`, `userRoles`)
- **Tipos**: `PascalCase` (`SpoolData`, `UserRole`)
- **Constantes**: `SCREAMING_SNAKE_CASE` (`DEFAULT_PAGE_SIZE`)

#### SQL (PostgreSQL/Supabase)
- **Tablas**: `snake_case` singular (`nv`, `spool`, `uniones`)
- **Columnas**: prefijo de tabla + `snake_case` (`spool_estado`, `union_numero`)
- **Funciones**: `fn_` + descripción (`fn_require_updated_by`)
- **Triggers**: `tr_` + tabla + acción (`tr_spool_audit`)
- **Vistas**: `vw_` + propósito (`vw_responsables_pendientes`)

### Reglas específicas del proyecto

1. **Domain-Driven Design light**: Organizar por dominio (`nv/`, `spool/`, `uniones/`)
2. **RLS obligatorio**: Todas las tablas deben tener políticas de seguridad
3. **Auditoría completa**: `created_at/by`, `updated_at/by` en todas las tablas
4. **UTC en BD**: Convertir a timezone local solo en UI
5. **Set-once fields**: Campos `*_ft` no se pueden modificar una vez asignados
6. **Roles granulares**: Admin, Organizador, QAQC, Armador, Soldador

## 🧪 Testing

### Tests unitarios
```bash
# Ejecutar todos los tests
npm run test

# Tests en modo watch
npm run test:watch

# Tests con coverage
npm run test:coverage
```

### Tests de integración
```bash
# Tests de workers y jobs
npm run test:integration

# Tests de base de datos
npm run test:db
```

### Tests E2E (Playwright)
```bash
# Instalar dependencias de Playwright (primera vez)
npx playwright install

# Ejecutar tests E2E
npm run test:e2e

# Tests E2E en modo UI
npm run test:e2e:ui
```

## 🗃️ Base de datos y migraciones

### Crear nueva migración
```bash
# Generar archivo de migración
npm run db:create-migration "descripcion_cambio"

# Aplicar migraciones
npm run db:migrate

# Rollback (si está disponible)
npm run db:rollback
```

### Convenciones SQL
- **Nombres de archivos**: `YYYYMMDDTHHMMSS__{tipo}__{dominio}.sql`
- **Ejemplo**: `20250825T120000__create__spool_estados.sql`
- **Tipos**: `create`, `alter`, `drop`, `data`, `index`, `trigger`

### RLS (Row Level Security)
Todas las nuevas tablas deben:
1. Habilitar RLS: `ALTER TABLE tabla ENABLE ROW LEVEL SECURITY;`
2. Crear políticas para cada rol: Admin, Organizador, etc.
3. Documentar en `policies.sql` centralizado
4. Testear políticas con diferentes usuarios

## 📚 Documentación

### Estructura `/docs`
- **`adr/`**: Architecture Decision Records
- **`specs/`**: Especificaciones técnicas detalladas  
- **`handbooks/`**: Guías de desarrollo y estándares

### Cambios que requieren documentación
- Nuevos endpoints o rutas
- Modificaciones en base de datos (actualizar `tbl_*.txt`)
- Cambios arquitectónicos (crear ADR)
- Nuevos jobs asíncronos (actualizar `job_*.txt`)
- Componentes UI reutilizables (actualizar `cmp_*.txt`)

## 🐛 Reportar bugs

### Antes de reportar
1. Buscar en issues existentes
2. Reproducir en entorno limpio
3. Verificar versión actualizada

### Información requerida
- Descripción clara del problema
- Pasos para reproducir
- Comportamiento esperado vs actual
- Screenshots/logs si aplica
- Navegador y versión (para bugs UI)
- Variables de entorno relevantes (sin secretos)

### Template de bug report
```markdown
## Descripción
[Descripción clara del bug]

## Pasos para reproducir
1. Ir a...
2. Hacer clic en...
3. Ver error...

## Comportamiento esperado
[Qué debería pasar]

## Comportamiento actual
[Qué está pasando]

## Contexto
- Navegador: [Chrome 120, Firefox 115, etc.]
- Rol de usuario: [admin, organizador, soldador, etc.]
- URL donde ocurre: [/spool/123, /mis-tareas, etc.]

## Logs
[Incluir errores de consola si aplica]
```

## 🔄 Proceso de revisión

### Para maintainers
- **Tiempo de respuesta**: < 48 horas para feedback inicial
- **Criterios de aprobación**: Tests pasan, código sigue convenciones, documentación actualizada
- **Merge requirements**: Al menos 1 aprobación de maintainer

### Para contribuidores
- **Responder feedback** en tiempo razonable
- **Mantener PR actualizado** con main branch
- **Resolver conflictos** antes de merge
- **Squash commits** si es necesario para mantener historial limpio

## 🆘 Obtener ayuda

### Recursos disponibles
- **Issues**: Para bugs y feature requests
- **Discussions**: Para preguntas generales y ideas
- **Wiki**: Documentación extendida y ejemplos
- **ADRs** en `/docs/adr/`: Decisiones arquitectónicas importantes

### Contacto
- **Email**: dev@kronos-mining.com
- **Slack** (interno): #zeus-development

## 📝 Licencia

Al contribuir, aceptas que tu código se licencie bajo la [Licencia MIT](LICENSE) del proyecto.

---

¡Gracias por contribuir a Zeus by KM! 🚀