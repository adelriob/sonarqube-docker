# sonarqube-docker

SonarQube Community Edition desplegado en Docker Desktop con persistencia de datos y automatización vía GitHub Actions.

## Stack

- **SonarQube CE** — `sonarqube:community`
- **PostgreSQL 16** — base de datos persistente
- **Docker Compose** — orquestación local
- **GitHub Actions** — CI/CD con runner self-hosted

## Acceso rápido

| Recurso | URL |
|---------|-----|
| SonarQube UI | http://localhost:9000 |
| API status | http://localhost:9000/api/system/status |

Credenciales iniciales: `admin` / `admin` (forzado a cambiar en primer login)

## Inicio rápido

```bash
# 1. Configurar variables de entorno
cp .env.example .env
# Edita .env con la contraseña (ver DEPLOYMENT.md)

# 2. Levantar servicios
docker compose up -d

# 3. Verificar
docker compose ps
```

Para instrucciones completas de despliegue, configuración de GitHub Actions y backups, ver **[DEPLOYMENT.md](DEPLOYMENT.md)**.
