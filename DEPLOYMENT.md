# Deployment Guide — SonarQube CE en Docker Desktop

## Información del despliegue

| Campo | Valor |
|-------|-------|
| **Repositorio GitHub** | https://github.com/adelriob/sonarqube-docker |
| **URL de acceso** | http://localhost:9000 |
| **Credenciales iniciales** | `admin` / `admin` — **ya cambiadas el 2026-04-13** |
| **Imagen SonarQube** | `sonarqube:community` (Community Edition — v26.4.0.121862) |
| **Base de datos** | PostgreSQL 16 (Alpine) |
| **Puerto expuesto** | `9000` |
| **Runner GitHub Actions** | Self-hosted `mac-sonarqube-runner` (macOS ARM64, label `sonarqube`) |

---

## Credenciales y secretos generados

> **IMPORTANTE:** Guarda estos datos en un gestor de contraseñas seguro (1Password, Bitwarden, etc.)

| Secreto | Valor | Dónde configurar |
|---------|-------|-----------------|
| `POSTGRES_PASSWORD` | `2a47Pu4QG2oYTa6W0soxPM4K0kXuO3ejpsatDvYCvpM=` | GitHub Secret + `.env` local |

---

## Primer despliegue — Pasos

### 1. Configurar archivo `.env` local

```bash
cd ~/workload/sonarqube
cp .env.example .env
# Edita .env y agrega la contraseña generada:
echo "POSTGRES_PASSWORD=2a47Pu4QG2oYTa6W0soxPM4K0kXuO3ejpsatDvYCvpM=" > .env
```

### 2. Levantar servicios manualmente (primera vez)

```bash
docker compose up -d
```

Verifica que los contenedores estén corriendo:

```bash
docker compose ps
```

### 3. Verificar que SonarQube esté listo

```bash
# Espera ~2 minutos. Luego comprueba el status:
curl -s http://localhost:9000/api/system/status
# Debe responder: {"status":"UP",...}
```

O simplemente abre: **http://localhost:9000**

### 4. Cambiar contraseña de administrador

> **Completado el 2026-04-13.** La contraseña por defecto `admin/admin` ya fue cambiada.
> Guarda la nueva contraseña en tu gestor de contraseñas.

---

## Configurar GitHub Actions (self-hosted runner)

### Paso 1 — Crear el repositorio en GitHub

```bash
# En GitHub: crea el repo 'sonarqube-docker' en https://github.com/adelriob
# Luego desde tu Mac:
cd ~/workload/sonarqube
git remote add origin https://github.com/adelriob/sonarqube-docker.git
git push -u origin main
```

### Paso 2 — Agregar el secret `POSTGRES_PASSWORD` en GitHub

1. Ve a: https://github.com/adelriob/sonarqube-docker/settings/secrets/actions
2. Click en **"New repository secret"**
3. Name: `POSTGRES_PASSWORD`
4. Secret: `2a47Pu4QG2oYTa6W0soxPM4K0kXuO3ejpsatDvYCvpM=`
5. Click **"Add secret"**

### Paso 3 — Registrar el runner self-hosted

1. Ve a: https://github.com/adelriob/sonarqube-docker/settings/actions/runners
2. Click **"New self-hosted runner"**
3. Selecciona: **macOS** / **x64** (o ARM si tu Mac es Apple Silicon)
4. Sigue las instrucciones de instalación que muestra GitHub
5. Cuando te pida el label adicional, **agrega `sonarqube`**:

```
# Durante la configuración del runner, cuando pregunte por labels:
Enter any additional labels (ex. label-1,label-2): sonarqube
```

6. Inicia el runner:

```bash
# Desde el directorio donde instalaste el runner:
./run.sh
# Para correrlo en background como servicio:
./svc.sh install && ./svc.sh start
```

### Paso 4 — Disparar el primer deploy vía GitHub Actions

Haz cualquier push a `main` o ve a:
https://github.com/adelriob/sonarqube-docker/actions → **"Run workflow"**

---

## Volúmenes Docker (persistencia de datos)

Los datos se almacenan en volúmenes Docker nombrados. Son independientes del ciclo de vida de los contenedores.

| Volumen | Contenido |
|---------|-----------|
| `sonarqube_postgres_data` | Base de datos PostgreSQL |
| `sonarqube_data` | Proyectos, análisis, configuración |
| `sonarqube_extensions` | Plugins instalados |
| `sonarqube_logs` | Logs de SonarQube |

Ver la ruta física de un volumen:

```bash
docker volume inspect sonarqube_data
```

---

## Comandos útiles

```bash
# Levantar servicios
docker compose up -d

# Detener servicios (datos se conservan)
docker compose down

# Ver logs en tiempo real
docker compose logs -f sonarqube

# Ver estado de contenedores
docker compose ps

# Reiniciar solo SonarQube
docker compose restart sonarqube

# Actualizar a la última versión de SonarQube
docker compose pull sonarqube
docker compose up -d sonarqube
```

---

## Backup de datos

```bash
# Backup del volumen PostgreSQL
docker run --rm \
  -v sonarqube_postgres_data:/data \
  -v $(pwd)/backups:/backup \
  alpine tar czf /backup/postgres_$(date +%Y%m%d_%H%M%S).tar.gz -C /data .

# Backup del volumen de datos SonarQube
docker run --rm \
  -v sonarqube_data:/data \
  -v $(pwd)/backups:/backup \
  alpine tar czf /backup/sonarqube_data_$(date +%Y%m%d_%H%M%S).tar.gz -C /data .
```

---

## Troubleshooting

### SonarQube no inicia (error Elasticsearch)

En Linux se requiere aumentar `vm.max_map_count`. **En macOS con Docker Desktop esto NO aplica** — Docker Desktop maneja esta configuración internamente.

Si ves el error en Linux:
```bash
sudo sysctl -w vm.max_map_count=524288
echo "vm.max_map_count=524288" | sudo tee -a /etc/sysctl.conf
```

### Puerto 9000 ocupado

Edita `docker-compose.yml` y cambia `"9000:9000"` por `"9001:9000"`, luego `docker compose up -d`.

### Verificar conectividad DB → SonarQube

```bash
docker compose logs sonarqube-db --tail=20
docker compose logs sonarqube --tail=50
```

---

## Actualización de SonarQube

```bash
# Actualizar imagen
docker compose pull sonarqube

# Reiniciar con nueva versión (datos persistentes)
docker compose up -d sonarqube

# Verificar versión actual
curl -s http://localhost:9000/api/server/version
```

> SonarQube aplica migraciones de base de datos automáticamente al arrancar con una nueva versión.
