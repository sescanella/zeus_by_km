# Changelog

Todos los cambios notables en este proyecto serán documentados en este archivo.

El formato está basado en [Keep a Changelog](https://keepachangelog.com/en/1.0.0/),
y este proyecto sigue [Semantic Versioning](https://semver.org/spec/v2.0.0.html).

## [Unreleased]

### Added
### Changed
### Fixed
### Security

## [0.1.0] - 2025-08-25

### Added

#### Funcionalidades principales
- **Gestión de Notas de Venta (NV)**: Creación, edición y seguimiento completo de notas de venta con estados automáticos
- **Gestión de Spools**: Sistema completo de spools vinculados a NV con control de estados jerárquicos
- **Gestión de Uniones**: Manejo de uniones por spool con numeración automática y asignación de soldadores
- **Sistema de roles granular**: Implementación de roles Admin, Organizador, QAQC, Armador, y Soldador con permisos específicos
- **Bandeja de tareas unificada**: Vista centralizada de pendientes por rol (`vw_responsables_pendientes`)
- **Transiciones automáticas de estado**: Flujo jerárquico NV → Spool → Unión con estados calculados

#### Interfaz de usuario
- **Dashboard principal**: Vista general del sistema con métricas y accesos rápidos
- **Rutas específicas por dominio**:
  - `/nv` - Gestión de Notas de Venta
  - `/spool` - Gestión de Spools
  - `/uniones` - Gestión de Uniones
  - `/usuarios` - Administración de usuarios
  - `/mis-tareas` - Bandeja personal de tareas
  - `/mis-tareas/responsables` - Asignación de responsables
- **Componentes UI reutilizables**:
  - `RoleFilteredSelect` - Selector de usuarios por rol activo
  - `UploadPlano` - Gestión de archivos con Supabase Storage
- **Diseño responsivo**: Implementación completa con Tailwind CSS

#### Base de datos y seguridad
- **Esquema completo PostgreSQL**: 6 tablas principales con triggers y validaciones
  - `nv` - Notas de Venta
  - `spool` - Spools con datos técnicos
  - `uniones` - Uniones con soldadores RA/RE
  - `usuarios` - Usuarios extendidos de Supabase Auth
  - `rol` - Catálogo de roles del sistema
  - `usuario_rol` - Relaciones many-to-many usuarios-roles
- **Row Level Security (RLS)**: Políticas granulares por tabla y rol
- **Funciones de seguridad**: Validación de roles activos y usuarios autenticados
- **Triggers de validación**: Integridad referencial y reglas de negocio automáticas
- **Campos set-once**: Protección contra modificaciones no autorizadas en campos `*_ft`
- **Auditoría completa**: Tracking de cambios con `created_at/by`, `updated_at/by` en UTC

#### Jobs asíncronos (Workers)
- **Sistema pg-boss**: Queue robusto basado en PostgreSQL
- **Job `crear_uniones`**: Generación automática de N uniones al crear spool
- **Job `state_transitions`**: Procesamiento de transiciones de estado jerárquicas
- **Dead Letter Queue**: Manejo de jobs fallidos con reintentos configurables
- **Monitoreo y logs**: Sistema completo de observabilidad para workers

#### Storage y archivos
- **Supabase Storage**: Bucket privado 'planos' para documentos técnicos
- **URLs firmadas**: Sistema seguro de preview temporal de archivos
- **Validaciones**: Control de tipos de archivo, tamaño máximo y extensiones permitidas
- **Organización jerárquica**: Estructura de carpetas por NV y Spool

#### Arquitectura técnica
- **Next.js 14**: App Router con server actions y streaming
- **TypeScript strict**: Tipado completo en todo el stack
- **Domain-Driven Design**: Organización por dominios de negocio
- **Monorepo structure**: Packages compartidos entre web y workers
- **Zod schemas**: Validación de datos client-side y server-side

### Changed

#### Configuración inicial
- **Variables de entorno**: Separación clara entre configuración web y workers
- **Scripts de desarrollo**: Comandos unificados para web, workers y base de datos
- **Linting y formatting**: ESLint + Prettier configurados para consistencia
- **Testing setup**: Jest + Testing Library + Playwright E2E

### Security

#### Implementaciones de seguridad
- **Autenticación Supabase**: Integración completa con Supabase Auth
- **Validación de usuarios activos**: Triggers que garantizan usuarios válidos en `created_by`
- **Protección de roles**: Validación automática de roles activos en asignaciones
- **CORS configurado**: Políticas restrictivas para dominios autorizados
- **Service Role limitado**: Acceso controlado solo para workers asíncronos
- **Políticas RLS exhaustivas**: Control granular de acceso por rol a nivel de fila
- **Prevención de inyección**: Uso de queries parametrizadas y validación Zod
- **Headers de seguridad**: CSP y headers de seguridad configurados en Next.js

### Technical Debt

#### Limitaciones conocidas
- **Tests de integración**: Cobertura parcial, pendiente expansión completa
- **Documentación API**: Especificación OpenAPI pendiente
- **Performance optimization**: Índices de BD optimizados pero sin análisis de carga real
- **Error handling**: Manejo básico implementado, refinamiento pendiente
- **Internacionalización**: Solo español, estructura preparada para i18n futuro

---

## Versionado

- **Major** (X.0.0): Cambios breaking en API o base de datos que requieren migración
- **Minor** (0.X.0): Nuevas funcionalidades backwards-compatible
- **Patch** (0.0.X): Bug fixes y mejoras menores

## Enlaces

- [Repositorio](https://github.com/kronos-mining/zeus_by_km)
- [Documentación técnica](docs/)
- [Guía de contribución](CONTRIBUTING.md)
- [Política de seguridad](SECURITY.md)