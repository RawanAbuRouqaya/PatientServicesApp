# PatientServices

Clean Architecture .NET 8 API for patient operations against **Vida 3** (classic) and **Vida 4 / Vida Plus** (REST), with buffered integration logging and FusionCache.

## Structure

```
PatientServices/
├── src/
│   ├── PatientServices.Domain/
│   ├── PatientServices.Application/
│   ├── PatientServices.Infrastructure/
│   └── PatientServices.API/
├── scripts/CreateApiLogTables.sql
└── PatientServices.slnx
```

## Vida call pattern (from eHealthServices)

| Mode | When | How |
|------|------|-----|
| Vida Plus / Vida 4 | `UseVidaPlus = true` | REST `POST /csi-ie-mobile/api/v1/insurance/validate` with `{ projectId, patientId, setupId }` (same as `jsonparserWithoutSLAToken`) |
| Classic Vida 3 | `UseVidaPlus = false` | HTTP POST to configured eligibility path using `VidaUrl` (same fields as `RequestNphiesEligibility`) |
| Visits | always Vida 4 | `GET /csi-ie-mobile/api/v1/patients/visit` with client-id / client-secret headers |

Request JSON is buffered before the call; response JSON is buffered after (same idea as `IntegerationLogs_Insert` / `IntegerationLogs_Update`). Every hour, logs are **SqlBulkCopy**'d into `ApiRequestLogs` / `ApiResponseLogs`.

## Prerequisites

- .NET 8 SDK
- SQL Server (run `scripts/CreateApiLogTables.sql`)
- Optional Redis for L2 cache

## Configure

Edit `src/PatientServices.API/appsettings.json`:

- `ConnectionStrings:Database`
- `Vida4:ClientId` / `Vida4:ClientSecret` (prefer user-secrets)
- `Vida3:BaseUrl` and paths when classic Vida endpoints are known
- `Redis:ConnectionString` if using distributed cache

```bash
dotnet user-secrets set "Vida4:ClientId" "eservices" --project src/PatientServices.API
dotnet user-secrets set "Vida4:ClientSecret" "<secret>" --project src/PatientServices.API
```

## Build & run

```bash
dotnet build
dotnet run --project src/PatientServices.API
```

Swagger: `/swagger`  
Health: `GET /api/health`

## API endpoints

| Method | Route | Description |
|--------|-------|-------------|
| GET | `/api/patients/visits` | Vida 4 visits |
| GET | `/api/patients/{patientId}?projectId=` | Vida 3 get patient |
| POST | `/api/patients/search` | Vida 3 search |
| POST | `/api/patients/eligibility/nphies` | NPHIES eligibility (Vida 3 or Plus) |
