# Política de Seguridad - Zeus by KM

## 🔒 Versiones soportadas

Actualmente proporcionamos actualizaciones de seguridad para las siguientes versiones de Zeus by KM:

| Versión | Soportada          |
| ------- | ------------------ |
| 0.1.x   | :white_check_mark: |
| < 0.1   | :x:                |

**Nota**: Al ser un proyecto en fase inicial, solo soportamos la versión más reciente. Una vez que el proyecto alcance estabilidad (v1.0.0), mantendremos soporte LTS para versiones anteriores.

## 🚨 Reportar vulnerabilidades

### ¿Qué constituye una vulnerabilidad de seguridad?

Consideramos vulnerabilidades de seguridad a:

- **Autenticación y autorización**: Bypass de autenticación, escalación de privilegios, acceso no autorizado a recursos
- **Row Level Security (RLS)**: Acceso a datos fuera del scope autorizado por políticas RLS
- **Inyección**: SQL injection, XSS, command injection en workers o endpoints
- **Almacenamiento**: Acceso no autorizado a Storage (bucket 'planos'), exposición de URLs firmadas
- **Workers asíncronos**: Manipulación de colas de jobs, ejecución de código malicioso
- **Variables de entorno**: Exposición de secretos, service roles, o credenciales
- **CORS y CSP**: Configuraciones que permitan ataques cross-origin
- **Exposición de información**: Logs con datos sensibles, endpoints que revelan más información de la necesaria

### 🔐 Proceso de reporte seguro

**⚠️ NO reportes vulnerabilidades de seguridad a través de issues públicos de GitHub.**

#### Opción 1: Email seguro (recomendado)
Envía un email detallado a: **security@kronos-mining.com**

- Asunto: `[SECURITY] Zeus by KM - [Descripción breve]`
- Incluye: Descripción técnica, pasos para reproducir, impacto potencial
- Si tienes una clave PGP, úsala para cifrar información sensible

#### Opción 2: GitHub Security Advisories
Utiliza [GitHub Security Advisories](https://github.com/kronos-mining/zeus_by_km/security/advisories) para reportes privados.

### 📋 Información requerida

Para facilitar la investigación, incluye la siguiente información en tu reporte:

#### Información básica
- **Descripción clara** de la vulnerabilidad
- **Clasificación** (autenticación, autorización, inyección, etc.)
- **Componente afectado** (Web UI, Workers, Base de datos, Storage)
- **Severidad estimada** (Crítica, Alta, Media, Baja)

#### Detalles técnicos
- **Pasos detallados** para reproducir
- **Proof of Concept** (PoC) si es posible
- **URLs, endpoints o rutas** específicas afectadas
- **Roles de usuario** involucrados
- **Versión** del sistema afectada
- **Entorno** (desarrollo, staging, producción)

#### Contexto adicional
- **Precondiciones** necesarias
- **Impacto potencial** (confidencialidad, integridad, disponibilidad)
- **Posibles vectores de ataque**
- **Mitigaciones temporales** si las conoces

### 📧 Plantilla de reporte

```
Asunto: [SECURITY] Zeus by KM - [Título descriptivo]

=== INFORMACIÓN GENERAL ===
Componente afectado: [Web/Workers/BD/Storage]
Severidad estimada: [Crítica/Alta/Media/Baja]
Versión: [0.1.x o commit hash]
Entorno: [dev/staging/prod]

=== DESCRIPCIÓN ===
[Descripción técnica clara de la vulnerabilidad]

=== REPRODUCCIÓN ===
1. [Paso 1]
2. [Paso 2]
3. [Resultado observado]

=== IMPACTO ===
[Descripción del impacto potencial]

=== PROOF OF CONCEPT ===
[Código, screenshots, logs - SIN DATOS SENSIBLES REALES]

=== INFORMACIÓN ADICIONAL ===
[Cualquier contexto relevante]
```

## ⏱️ Proceso de respuesta

### Timeline esperado

| Fase | Tiempo objetivo | Descripción |
|------|----------------|-------------|
| **Confirmación inicial** | 24 horas | Confirmación de recepción del reporte |
| **Evaluación preliminar** | 72 horas | Clasificación de severidad y validación inicial |
| **Investigación completa** | 7-14 días | Análisis técnico completo y desarrollo de fix |
| **Resolución** | 14-30 días | Deploy de la corrección y validación |
| **Divulgación** | 30-90 días | Divulgación pública coordinada (si aplica) |

### Proceso interno

1. **Triage**: Evaluación inicial y asignación de severidad
2. **Investigación**: Análisis técnico y confirmación de impacto
3. **Desarrollo**: Creación de patch o mitigación
4. **Testing**: Validación de la corrección en entorno seguro
5. **Deploy**: Aplicación de la corrección en producción
6. **Seguimiento**: Verificación de efectividad y monitoreo

## 🏆 Reconocimientos

### Hall of Fame
Reconocemos públicamente a los investigadores responsables que reporten vulnerabilidades válidas:

- [Tu nombre aquí] - [Descripción de la vulnerabilidad] - [Fecha]

### Criterios para reconocimiento
- Reporte responsable siguiendo este proceso
- Vulnerabilidad confirmada con impacto real
- No divulgación pública antes de la corrección
- Cooperación durante el proceso de investigación

## 🛡️ Áreas de seguridad específicas

### Supabase y RLS
- **Políticas RLS**: Todas las tablas implementan Row Level Security
- **Funciones de seguridad**: `auth.user_has_role()`, `auth.user_is_active()`
- **Service Role**: Usado únicamente en workers para operaciones privilegiadas
- **Anon Key**: Expuesto públicamente, validación en backend

### Autenticación y autorización
- **Supabase Auth**: Manejo de sesiones y tokens JWT
- **Roles granulares**: Admin, Organizador, QAQC, Armador, Soldador
- **Validación de roles**: En triggers, políticas RLS y lógica de aplicación
- **Usuarios activos**: Validación obligatoria en operaciones críticas

### Storage y archivos
- **Bucket privado**: 'planos' requiere autenticación
- **URLs firmadas**: Para preview temporal de archivos
- **Validación de tipos**: Restricciones en extensiones y MIME types
- **Límites de tamaño**: Configurables por tipo de archivo

### Workers y jobs asíncronos
- **pg-boss**: Queue seguro con PostgreSQL
- **Aislamiento**: Workers ejecutan con service role limitado
- **Validación de payloads**: Schemas Zod para todos los jobs
- **Dead letter queue**: Para manejo de jobs fallidos

### Base de datos
- **Triggers de validación**: Integridad referencial y de negocio
- **Campos set-once**: Protección contra modificaciones no autorizadas
- **Auditoría**: Tracking completo de cambios con usuario y timestamp
- **Timezone UTC**: Almacenamiento consistente, conversión en UI

## ⚠️ Limitaciones conocidas

### Fuera del scope de seguridad
- **Vulnerabilidades en dependencias de terceros**: Reportar directamente al proyecto upstream
- **Ingeniería social**: Ataques dirigidos a usuarios finales
- **Ataques de fuerza bruta**: Mitigados por Supabase Auth (rate limiting)
- **DDoS**: Responsabilidad de la infraestructura de hosting

### Configuraciones específicas
- **CORS**: Configurado para dominios específicos en Supabase
- **CSP**: Headers de seguridad en Next.js config
- **Environment variables**: Separación clara entre públicas y secretas

## 📞 Contacto de seguridad

- **Email principal**: security@kronos-mining.com
- **Email alternativo**: dev@kronos-mining.com (si el principal no responde)
- **PGP Key**: [Key ID cuando esté disponible]
- **Telegram** (emergencias): @kronos_security

## 📚 Recursos adicionales

### Documentación de seguridad
- [Políticas RLS detalladas](db/policies/policies.sql)
- [Guía de configuración segura](docs/specs/security-setup.md)
- [ADRs de seguridad](docs/adr/) - Decisiones arquitectónicas relacionadas

### Referencias externas
- [Supabase Security](https://supabase.com/docs/guides/platform/security)
- [PostgreSQL RLS Documentation](https://www.postgresql.org/docs/current/ddl-rowsecurity.html)
- [Next.js Security](https://nextjs.org/docs/advanced-features/security-headers)

---

**Nota**: Esta política de seguridad está sujeta a actualizaciones. Los cambios significativos serán comunicados a través de los canales oficiales del proyecto.

**Última actualización**: 25 de agosto de 2025