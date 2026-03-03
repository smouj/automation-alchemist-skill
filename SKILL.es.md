name: automation-alchemist
version: 1.2.0
description: Transforma flujos de trabajo manuales y repetitivos en soluciones automatizadas elegantes con control de versiones, monitoreo y capacidades de rollback
author: SMOUJBOT
tags: [automatización, flujos de trabajo, eficiencia, scripts]
dependencies:
  - bash: ">=4.0"
  - python3: ">=3.8"
  - cron: "any"
  - systemd: "optional"
  - git: ">=2.0"
  - jq: "optional"
  - curl: "any"
environment:
  AUTOMATION_DIR: "~/.openclaw/automations"
  AUTOMATION_LOG_DIR: "/var/log/openclaw/automations"
  AUTOMATION_CONFIG_DIR: "~/.openclaw/config/automations"
  DEFAULT_SCHEDULE: "@daily"
  BACKUP_RETENTION_DAYS: 30
required_skills: [file-ops, task-runner, system-monitor]
```

# Habilidad de Automatización Alquimista

## Propósito

Transforma flujos de trabajo manuales y repetitivos en soluciones automatizadas robustas, monitoreadas y con control de versiones. Esta habilidad elimina los errores humanos en tareas operativas, garantiza consistencia y proporciona auditorías completas.

### Casos de Uso Reales

1. **Automatización de Mantenimiento de Bases de Datos**: Convierte scripts manuales de limpieza `psql` en tareas programadas con alertas de error y rollback.
2. **Rotación de Logs + Análisis**: Automatiza la configuración de `logrotate` más análisis post-rotación y detección de anomalías.
3. **Orquestación de Respaldos**: Crea scripts de respaldo automatizados con verificación de checksum, políticas de retención y sincronización a la nube.
4. **Generación de Reportes**: Transforma la extracción manual de datos y generación de PDFs en reportes programados y distribuidos por correo.
5. **Agregador de Revisiones de Salud**: Construye monitoreo centralizado de salud del sistema que agrega múltiples revisiones de servicios.
6. **Ensamblaje de Pipeline de Despliegue**: Crea automatización de pasos CI/CD para builds Docker, migraciones y pruebas de humo.
7. **Automatización de Cumplimiento**: Genera y recopila logs de auditoría de seguridad, aplica endurecimiento de configuración y produce reportes de cumplimiento.

## Alcance

### Comandos Proporcionados

```
automation-alchemist create <name> [--template <template>] [--schedule <cron>] [--notify <email>]
automation-alchemist list [--format <json|table>] [--status <active|inactive>]
automation-alchemist test <name> [--dry-run] [--verbose] [--env-file <path>]
automation-alchemist deploy <name> [--force] [--user <user>] [--group <group>]
automation-alchemist disable <name>
automation-alchemist enable <name>
automation-alchemist logs <name> [--tail <n>] [--since <time>] [--level <INFO|DEBUG|ERROR>]
automation-alchemist status <name> [--detail]
automation-alchemist update <name> [--script <path>] [--schedule <cron>] [--notify <email>]
automation-alchemist rollback <name> [--to-version <version>] [--commit <hash>]
automation-alchemizer diff <name> [--from <version>] [--to <version>]
automation-alchemist monitor <name> [--duration <seconds>] [--interval <seconds>]
automation-alchemist export <name> [--format <tar|zip>] [--output <path>]
automation-alchemist import <path> [--name <new-name>]
automation-alchemist validate <name> [--check-security] [--check-performance]
```

## Proceso de Trabajo Detallado

### Paso 1: Descubrimiento y Análisis
```bash
# Identify manual workflows to automate
cat > /tmp/manual-workflow.sh << 'EOF'
#!/bin/bash
# Example manual process to be automated
today=$(date +%Y-%m-%d)
echo \"Backing up database for $today...\"
pg_dump mydb > /backups/manual_$today.sql
echo \"Compressing...\"
gzip /backups/manual_$today.sql
echo \"Uploading to S3...\"
aws s3 cp /backups/manual_$today.sql.gz s3://mybucket/backups/
echo \"Cleaning old backups (>30 days)...\"
find /backups -name \"*.gz\" -mtime +30 -delete
EOF
```

### Paso 2: Crear Automatización
```bash
# Create from analysis
automation-alchemist create database-backup \
  --template backup \
  --schedule \"0 2 * * *\" \
  --notify \"admin@example.com\"
```

### Paso 3: Configurar Entorno
Cree `~/.openclaw/config/automations/database-backup.env`:
```bash
export PGPASSWORD=\"super_secret_password\"
export AWS_ACCESS_KEY_ID=\"AKIAIOSFODNN7EXAMPLE\"
export AWS_SECRET_ACCESS_KEY=\"wJalrXUtnFEMI/K7MDENG/bPxRfiCYEXAMPLEKEY\"
export BACKUP_DIR=\"/var/backups/postgres\"
export S3_BUCKET=\"mybucket/backups\"
export RETENTION_DAYS=30
```

### Paso 4: Probar Automatización
```bash
automation-alchemist test database-backup --dry-run --verbose
automation-alchemist test database-backup --env-file ~/.secrets/db-backup.env
```

### Paso 5: Desplegar
```bash
automation-alchemist deploy database-backup --user postgres --group backup
# Verification
automation-alchemist status database-backup
ls -la ~/.openclaw/automations/database-backup/
crontab -l | grep database-backup
```

### Paso 6: Configurar Monitoreo
```bash
# Add monitoring
automation-alchemist monitor database-backup \
  --duration 3600 \
  --interval 60
# Check logs
automation-alchemist logs database-backup --tail 50 --since \"2 hours ago\"
```

## Reglas de Oro

1. **Nunca almacenes secretos en scripts de automatización** - siempre usa archivos de entorno con permisos 600
2. **Siempre incluye manejo de errores** - cada script automatizado debe salir con códigos apropiados y registrar errores
3. **Implementa idempotencia** - los scripts deben ser seguros para ejecutarse múltiples veces sin efectos secundarios
4. **Controla versiones todo** - cada automatización obtiene un commit git con mensajes semánticos
5. **Siempre verifica prerequisitos** - valida dependencias y permisos antes de la ejecución
6. **Incluye pasos de verificación** - comprobaciones posteriores a la condición para asegurar que la automatización tuvo éxito
7. **Configura alertas** - los códigos de salida distintos de cero deben activar notificaciones
8. **Prueba en aislamiento** - usa el flag `--dry-run` antes de cualquier despliegue en producción
9. **Respeta los límites del sistema** - incluye chequeos de recursos (espacio en disco, memoria, CPU)
10. **Documenta el rollback** - cada automatización debe tener un procedimiento de rollback probado

## Ejemplos

### Ejemplo 1: Automatización de Limpieza de Logs Simple
```bash
# Input
automation-alchemist create log-cleanup \
  --template cleanup \
  --schedule \"0 3 * * 0\" \
  --notify \"ops@example.com\"

# Generated structure:
~/.openclaw/automations/log-cleanup/
├── script.sh              # Main automation script
├── config.yaml           # Configuration
├── precheck.sh           # Prerequisite validation
├── rollback.sh           # Rollback procedure
├── test/                # Test suite
│   ├── unit/
│   └── integration/
├── .env.example         # Environment template
└── README.md           # Documentation

# The generated script includes:
# - Input validation
# - Error trapping
# - Logging to $AUTOMATION_LOG_DIR
# - Metrics collection
# - Alert on failure
# - Exit code reporting

# Test execution
automation-alchemist test log-cleanup --dry-run
# Output:
# [DRY-RUN] Would execute: /home/user/.openclaw/automations/log-cleanup/script.sh
# [DRY-RUN] Environment: /home/user/.openclaw/config/automations/log-cleanup.env
# [DRY-RUN] Schedule: 0 3 * * 0
# [DRY-RUN] Notification: ops@example.com
# [DRY-RUN] Validation: ✓ precheck passed
# [DRY-RUN] Simulation: ✓ would delete 142 old log files (45.2 MB)
# [DRY-RUN] Test completed successfully

# Deploy
automation-alchemist deploy log-cleanup
# Deploying automation 'log-cleanup'...
# ✓ Script installed to /usr/local/bin/log-cleanup-automation
# ✓ Cron job added: 0 3 * * 0 /usr/local/bin/log-cleanup-automation
# ✓ Environment configured
# ✓ Permissions set (700)
# Deployment successful

# Verify
automation-alchemist status log-cleanup
# Name: log-cleanup
# Status: active
# Schedule: 0 3 * * 0
# Last run: 2024-01-15 03:00:12 UTC
# Next run: 2024-01-22 03:00:00 UTC
# Exit code last: 0
# Log location: /var/log/openclaw/automations/log-cleanup.log
```

### Ejemplo 2: Respaldo de Base de Datos con Retención
```bash
# User prompt: \"Create an automation that backs up PostgreSQL daily, keeps 7 days, uploads to S3, and notifies on failure\"

# Create with interactive wizard
automation-alchemist create pg-backup --interactive

# Wizard prompts:
# ? Template: backup
# ? Schedule: 0 1 * * * (daily at 1 AM)
# ? Backup retention: 7 days
# ? Notification email: dbadmin@example.com
# ? Upload destination: S3
# ? S3 bucket: mycompany-db-backups
# ? Compression: gzip (default)
# ? Encryption: yes (uses GPG)

# Generated script includes:
# - pg_dump with custom format
# - Checksum generation (SHA256)
# - S3 sync with delete flag
# - GPG encryption if configured
# - Retention enforcement
# - Post-backup verification (restore test in temp)
# - Email notification with summary

# Test with realistic data volume
automation-alchemist test pg-backup --env-file ~/.secrets/pg-prod.env --size 5GB

# Output:
# [TEST] Starting pg-backup automation...
# [TEST] Connecting to database: production-db.example.com:5432
# [TEST] Dumping database 'customers' (5.2 GB)...
# [TEST] ✓ Dump completed in 4m 32s
# [TEST] Compressing with gzip...
# [TEST] ✓ Compressed to 1.1 GB
# [TEST] Encrypting with GPG...
# [TEST] ✓ Encrypted (1.1 GB)
# [TEST] Uploading to S3...
# [TEST] ✓ Upload completed (1.1 GB in 2m 18s)
# [TEST] Verifying checksum...
# [TEST] ✓ Checksum matches
# [TEST] Retention cleanup: removed 8 old backups (>7 days)
# [TEST] ✓ All tests passed (9m 12s)

# Deploy
automation-alchemist deploy pg-backup

# Monitor first few runs
automation-alchemist logs pg-backup --tail 20 --since \"3 days ago\"
# [2024-01-15 01:00:00] Starting backup...
# [2024-01-15 01:04:32] Dump completed: 5.2 GB
# [2024-01-15 01:05:15] Compressed: 1.1 GB
# [2024-01-15 01:07:33] Upload complete: s3://mybucket/backups/pg-backup-20240115.sql.gz.gpg
# [2024-01-15 01:07:45] Verification passed
# [2024-01-15 01:08:02] Cleanup: removed 8 old files
# [2024-01-15 01:08:03] ✓ Backup successful (8m 03s)
```

### Ejemplo 3: Agregador de Revisiones de Salud
```bash
# User prompt: \"Create automation that checks 5 services, aggregates results, and sends daily digest\"

automation-alchemist create health-aggregator \
  --template monitor \
  --schedule \"0 8 * * *\" \
  --notify \"team@example.com\"

# The generated script:
#!/bin/bash
# Auto-generated by automation-alchemist v1.2.0
set -euo pipefail

# Load configuration
source \"${AUTOMATION_CONFIG_DIR}/health-aggregator/config.env\"

# Services to check
SERVICES=(
  \"api.rpgclaw.dev:3000|/health\"
  \"db.rpgclaw.dev:5432||pg_isready\"
  \"redis.rpgclaw.dev:6379||redis-cli ping\"
  \"nginx.rpgclaw.dev:80|/nginx_status\"
  \"flickclaw.dev:3010|/api/health\"
)

RESULTS_FILE=\"/tmp/health-check-$(date +%s).json\"
ALERT_THRESHOLD=2

check_service() {
  local service=$1
  local host_port=$2
  local endpoint=$3
  local check_cmd=$4
  
  if [[ -n \"$endpoint\" ]]; then
    # HTTP check
    local response=$(curl -s -o /dev/null -w \"%{http_code}\" \"http://${host_port}${endpoint}\")
    if [[ \"$response\" == \"200\" ]]; then
      echo \"✓ $service\"
      return 0
    else
      echo \"✗ $service (HTTP $response)\"
      return 1
    fi
  elif [[ -n \"$check_cmd\" ]]; then
    # Custom command check
    if eval \"$check_cmd\" &>/dev/null; then
      echo \"✓ $service\"
      return 0
    else
      echo \"✗ $service (command failed)\"
      return 1
    fi
  fi
}

# Execute checks
FAILED=0
for svc in \"${SERVICES[@]}\"; do
  IFS='|' read -r name hostport endpoint cmd <<< \"$svc\"
  if ! check_service \"$name\" \"$hostport\" \"$endpoint\" \"$cmd\"; then
    ((FAILED++))
  fi
done >&2

# Generate report
cat > \"$RESULTS_FILE\" <<JSON
{
  \"timestamp\": \"$(date -u +%Y-%m-%dT%H:%M:%SZ)\",
  \"total\": ${#SERVICES[@]},
  \"passed\": $((${#SERVICES[@]} - FAILED)),
  \"failed\": $FAILED,
  \"status\": \"$([ $FAILED -eq 0 ] && echo 'healthy' || echo 'degraded')\"
}
JSON

# Send notification if threshold exceeded
if [[ $FAILED -ge $ALERT_THRESHOLD ]]; then
  mail -s \"🚨 Health Check Failed: $FAILED services down\" \"$NOTIFICATION_EMAIL\" < \"$RESULTS_FILE\"
fi

exit $FAILED
```

# Test and monitor
automation-alchemist test health-aggregator --dry-run
automation-alchemist deploy health-aggregator
automation-alchemist status health-aggregator
automation-alchemist logs health-aggregator --since \"1 day ago\"
```

## Comandos de Rollback

### Rollback Inmediato a la Versión Anterior
```bash
# List versions
automation-alchemist diff pg-backup --from v1.0 --to v1.1

# Rollback to specific version
automation-alchemist rollback pg-backup --to-version v1.0
# OR by commit hash
automation-alchemist rollback pg-backup --commit abc123def

# Output:
# Rolling back 'pg-backup' to version v1.0 (commit abc123)...
# ✓ Previous script backed up to ~/.openclaw/automations/archive/pg-backup-v1.1-20240115.sh
# ✓ Restored script from v1.0
# ✓ Cron job updated
# ✓ Deactivated v1.1 deployment
# Rollback successful. Current version: v1.0
```

### Deshabilitación de Emergencia (Mantener Estado)
```bash
# Disable automation without removing
automation-alchemist disable pg-backup
# Output:
# Disabling 'pg-backup'...
# ✓ Cron job removed (but saved in ~/.openclaw/automations/archive/pg-backup.cron)
# ✓ Systemd service stopped (if applicable)
# ✓ Automation state saved as 'inactive'
# Use 'automation-alchemist enable pg-backup' to re-enable

# Re-enable later
automation-alchemist enable pg-backup
```

### Rollback Completo con Inspección de Estado
```bash
# See what changed between versions
automation-alchemist diff pg-backup --from v1.0 --to v1.1
# Output:
# Differences between v1.0 and v1.1:
# 
# Script changes (script.sh):
#   - RETENTION_DAYS=30
#   + RETENTION_DAYS=15
#   - aws s3 cp
#   + aws s3 sync --delete
# 
# Config changes (config.yaml):
#   - schedule: \"0 2 * * *\"
#   + schedule: \"0 3 * * *\"
# 
# Metrics: 23 lines added, 8 removed, 2 modified

# Rollback with verification
automation-alchemist rollback pg-backup --to-version v1.0 --verify
# After rollback:
automation-alchemist test pg-backup --dry-run --verbose
# Confirms old version works before live deployment
```

## Dependencias y Requisitos

### Paquetes del Sistema
```bash
# Ubuntu/Debian
apt-get install -y bash python3 cron git jq curl mailutils

# RHEL/CentOS
yum install -y bash python3 cronie git jq curl mailx

# macOS
brew install bash python git jq curl
```

### Dependencias de Python
`~/.openclaw/automations/requirements.txt`:
```
croniter>=1.4.0    # Cron parsing
pytz>=2023.3       # Timezone handling
boto3>=1.28.0      # AWS integration (optional)
python-dotenv>=1.0.0  # Environment loading
psutil>=5.9.0      # System metrics
```

### Permisos
```bash
# Automated setup
mkdir -p ~/.openclaw/automations
mkdir -p ~/.openclaw/config/automations
mkdir -p /var/log/openclaw/automations
chmod 700 ~/.openclaw/automations
chmod 600 ~/.openclaw/config/automations/*.env
```

## Pasos de Verificación

### Verificación Pre-Despliegue
```bash
# 1. Syntax check
bash -n ~/.openclaw/automations/<name>/script.sh

# 2. Dry-run with environment validation
automation-alchemist test <name> --dry-run --verbose

# 3. Check for secrets in plaintext
grep -r \"password\|secret\|token\" ~/.openclaw/automations/<name>/script.sh || echo \"✓ No hardcoded secrets\"

# 4. Verify idempotency (run twice)
automation-alchemist test <name> --dry-run && automation-alchemist test <name> --dry-run

# 5. Validate cron schedule
crontab -l | grep <name> | awk '{print $1,$2,$3,$4,$5}' | croniter -c \"true\"
```

### Verificación Post-Despliegue
```bash
# 1. Confirm cron job
crontab -l | grep <name>

# 2. Check permissions
ls -la ~/.openclaw/automations/<name>/

# 3. Test manual execution
~/.openclaw/automations/<name>/script.sh

# 4. Verify logs directory
touch /var/log/openclaw/automations/<name>.log
ls -la /var/log/openclaw/automations/

# 5. Check systemd service (if used)
systemctl status openclaw-automation-<name>.service
```

## Solución de Problemas

### Problema: La automatización no se ejecuta a la hora programada
```
Diagnosis:
1. Check cron: crontab -l | grep <name>
2. Check cron logs: grep CRON /var/log/syslog
3. Verify script permissions: ls -la ~/.openclaw/automations/<name>/script.sh
4. Test manually: ~/.openclaw/automations/<name>/script.sh

Fix:
chmod 700 ~/.openclaw/automations/<name>/script.sh
# Ensure environment file exists and has 600 permissions
chmod 600 ~/.openclaw/config/automations/<name>.env
```

### Problema: Variables de entorno no cargadas
```
Diagnosis:
automation-alchemist logs <name> --since \"1 hour ago\" | grep \"variable not found\"

Fix:
1. Verify env file exists:
   ls -la ~/.openclaw/config/automations/<name>.env
2. Check script loads it:
   grep \"source.*env\" ~/.openclaw/automations/<name>/script.sh
3. Test sourcing manually:
   source ~/.openclaw/config/automations/<name>.env && env | grep <VAR>
```

### Problema: Rollback falla con \"version not found\"
```
Diagnosis:
automation-alchemist diff <name> --list-versions

Fix:
# List available versions
git --git-dir=~/.openclaw/automations/.git log --oneline -- <name>/script.sh

# Use commit hash instead
automation-alchemist rollback <name> --commit <hash>
```

### Problema: Alto uso de recursos durante la ejecución
```
Diagnosis:
automation-alchemist monitor <name> --duration 300 --interval 5

Fix:
Add resource limits to script:
  
  # In script.sh, before main logic
  ulimit -v 2097152  # 2GB virtual memory
  timeout 3600 /path/to/heavy/command
  
  # Or use cpulimit
  cpulimit -l 50 -- /path/to/command
```

### Problema: Notificaciones por correo electrónico no se envían
```
Diagnosis:
1. Check mail logs: tail -f /var/log/mail.log
2. Test mail command: echo \"test\" | mail -s \"test\" user@example.com
3. Verify NOTIFICATION_EMAIL in config

Fix:
# Ensure mailutils/postfix installed
apt-get install -y mailutils

# Configure /etc/ssmtp/ssmtp.conf or /etc/mail.rc
# Test with:
automation-alchemist test <name> --trigger-notification
```

## Consideraciones de Seguridad

1. **Principio de Mínimo Privilegio**: Ejecuta automatizaciones bajo usuarios dedicados no privilegiados
2. **Gestión de Secretos**: Usa `~/.openclaw/config/automations/<name>.env` con permisos 600; nunca incluyas secretos en commits
3. **Validación de Entrada**: Sanitiza todos los parámetros en los scripts para prevenir inyección
4. **Aislamiento de Red**: Restringe el acceso de red de las automatizaciones con reglas de firewall
5. **Auditoría**: Todas las ejecuciones se registran con marcas de tiempo, usuario y códigos de salida
6. **Límite de Tasa**: Añade intervalos de sleep entre operaciones repetitivas
7. **Fijación de Dependencias**: Registra las versiones exactas de paquetes utilizados
8. **Firma de Código**: Firma opcional de scripts de automatización con GPG

## Optimización de Rendimiento

1. **Ejecución Paralela**: Para tareas independientes, usa GNU parallel o jobs en segundo plano
2. **Almacenamiento en Caché**: Caché de operaciones costosas (ej. búsquedas DNS, llamadas API)
3. **Carga Perezosa**: Carga solo los módulos requeridos
4. **Presupuesto de Recursos**: Establece ulimits y timeouts
5. **Procesamiento Incremental**: Procesa datos en fragmentos para evitar agotamiento de memoria
6. **Salida Temprana**: Fallar rápido en errores irrecuperables
7. **Agrupación de Conexiones**: Reutiliza conexiones para bases de datos/APIs
8. **Compresión**: Usa niveles de compresión apropiados (equilibrio velocidad vs tamaño)
```