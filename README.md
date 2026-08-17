# Sesión 7: Ecosistema de APIs — Descubrimiento, Contratos y Mocking Dinámico

En esta sesión construimos un **Developer Control Plane** local que integra dos herramientas clave del ecosistema de gestión de APIs:

| Herramienta | Rol en la PoC |
|---|---|
| **Backstage** | Portal del desarrollador — catálogo de servicios con documentación interactiva |
| **Microcks** | Motor de mocking dinámico — simula APIs a partir de contratos OpenAPI/AsyncAPI alojado en Kubernetes |

El objetivo es habilitar el **desarrollo asíncrono** entre equipos: un equipo puede consumir y probar una API antes de que el servicio real esté implementado.

---

## Requisitos Previos

Antes de comenzar, asegúrate de tener instalado:

```bash
# Verificar Docker
docker version

# Verificar kind
kind version          # >= 0.20.0

# Verificar kubectl
kubectl version --client

# Verificar helm
helm version          # >= 3.x

# Verificar Node.js y yarn (para Backstage)
node --version        # >= 18.x
yarn --version        # >= 1.22.x
```

---

## Guías de la PoC

Sigue las guías en orden:

1. **[Fase 0 — Setup del entorno](00-setup/README.md)**
2. **[Fase 1 — Microcks: Mocking Dinámico](01-microcks/README.md)**
3. **[Fase 2 — Backstage: Portal del Desarrollador](02-backstage/README.md)**
