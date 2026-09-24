# Análisis del Mocking Dinámico con Microcks

## 1. Operaciones expuestas por la API Petstore

El contrato `petstore-with-examples.yaml` expone 2 operaciones:

| Método | Path | Dispatcher | Samples |
|---|---|---|---|
| GET | `/pet/findByStatus` | `URI_PARAMS` (filtra por el query param `status`) | 3 (`available`, `pending`, `sold`) |
| GET | `/pet/{petId}` | `SCRIPT` (Groovy, decide según el `petId` de la URL) | 2 (`pet_1`, `pet_2`) |

## 2. URL base generada por Microcks

Microcks genera automáticamente el mock siguiendo el patrón `http://{host}/rest/{title}/{version}/{path}`. Para esta API, con el `port-forward` activo en el puerto 9090:
