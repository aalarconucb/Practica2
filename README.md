# Tarea 2 – DevSecOps CI/CD Pipeline (Practica2)

Este repositorio implementa un pipeline CI/CD con enfoque DevSecOps sobre el repositorio base **Practica2** (microservicios Node/Express + frontend React), ejecutado en **GitHub Actions**.

## Ubicación del workflow
- `.github/workflows/devsecops.yml`

## Objetivo
Automatizar calidad, pruebas y controles de seguridad desde el código hasta el artefacto final (imágenes Docker), aplicando *quality gates* que bloquean la entrega ante hallazgos relevantes.

## Flujo del pipeline (alto nivel)
1. **Install (reproducible):** `npm ci`
2. **Quality:** ESLint (frontend)
3. **Tests:** Jest (`npm test`)
4. **SAST:** Semgrep (CLI)
5. **SCA:** `npm audit` (dependencias de producción)
6. **Build:** `docker compose build`
7. **Versioning:** tag por commit SHA (`:${GITHUB_SHA}`)
8. **Container scan:** Trivy (HIGH/CRITICAL, exit-code=1)
9. **Smoke test:** `health` + `login` + endpoint protegido
10. **Cleanup:** `docker compose down --volumes`

## Herramientas y por qué están en el pipeline
- **ESLint:** control de calidad y consistencia del frontend (gate).
- **Jest:** verificación automática del comportamiento (gate).
- **Semgrep:** SAST para detectar patrones inseguros (gate).
- **npm audit:** SCA para CVEs en dependencias de producción (gate).
- **Docker Compose:** build y ejecución efímera para pruebas.
- **Trivy:** escaneo de imágenes Docker (gate HIGH/CRITICAL).

## Notas sobre SCA
- Se corrigió configuración de dependencias para evitar que herramientas de desarrollo queden como dependencias de producción (p. ej., Jest en `dependencies`).
- Se aplicó remediación con `npm audit fix` y actualización de `package-lock.json` cuando correspondió.

## Notas sobre Trivy (alcance)
- El gate de Trivy se mantiene activo para **HIGH/CRITICAL**.
- Para evitar bloqueos por herramientas globales de la imagen base no utilizadas por el runtime de la app (npm/yarn/corepack), se ajustó el alcance con `skip-dirs` en la action.


## Cómo ejecutar localmente (opcional)
```bash
# preparar envs
cp backend/users-service/.env.example backend/users-service/.env
cp backend/academic-service/.env.example backend/academic-service/.env
cp backend/api-gateway/.env.example backend/api-gateway/.env
cp frontend/.env.example frontend/.env

# levantar stack
cd backend
docker compose up -d --build

# health
curl http://localhost:3000/health
```
