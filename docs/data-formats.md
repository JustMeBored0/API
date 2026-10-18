# Formáty dat

## Formát a kódování

- Těla požadavků i odpovědí jsou **JSON** v kódování **UTF-8** (kromě `204 No Content`).
- Názvy polí jsou v `camelCase`.
- Klient posílá `Content-Type: application/json` u požadavků s tělem (`POST`, `PUT`, `PATCH`) a `Accept: application/json`.

## Hlavičky požadavku

| Hlavička | Povinná | Popis |
| --- | --- | --- |
| `Authorization` | u chráněných endpointů | `Bearer <access token>`, viz [Autentizace](/authentication) |
| `Content-Type` | u požadavků s tělem | vždy `application/json` |
| `Accept` | ne | `application/json` (výchozí) |
| `X-Request-Id` | ne | vlastní id požadavku, API ho vrátí beze změny |

## Hlavičky odpovědi

| Hlavička | Popis |
| --- | --- |
| `Content-Type` | `application/json; charset=utf-8` |
| `X-Request-Id` | id požadavku pro dohledání v logu |
| `X-API-Version` | konkrétní vydání API, např. `1.0.0` |
| `Location` | URL nového zdroje u `201 Created` |
| `X-RateLimit-Limit`, `X-RateLimit-Remaining` | povolený a zbývající počet požadavků v okně |
| `Retry-After` | sekundy do dalšího pokusu u `429` |

## Typy hodnot

| Typ | Formát | Příklad |
| --- | --- | --- |
| Identifikátor | UUID v4 | `"3f2b8c1e-5a6d-4c9a-8d2e-1b7e9f0a4c11"` |
| Datum a čas | ISO 8601, UTC | `"2026-10-18T08:30:00Z"` |
| Výčet | malá písmena | `"active"` |
| Chybějící hodnota | `null` nebo vynecháno | `"description": null` |

## Stránkování a filtrování

| Parametr | Výchozí | Popis |
| --- | --- | --- |
| `page` | `1` | číslo stránky od 1 |
| `limit` | `20` | počet položek na stránku (1–100) |
| `q` | – | fulltextový filtr podle názvu a popisu |

```json
{
  "data": [
    { "id": "3f2b8c1e-5a6d-4c9a-8d2e-1b7e9f0a4c11", "title": "Úvod do REST" }
  ],
  "page": 1,
  "limit": 20,
  "total": 42
}
```

Nestránkované seznamy (např. lekce kurzu) vrací pole přímo.

## Chyby

Každá chyba má stavový kód HTTP a tělo ve stejném formátu:

```json
{
  "code": "VALIDATION_ERROR",
  "message": "Pole title je povinné.",
  "details": [{ "field": "title", "issue": "required" }]
}
```

- `code` je stabilní strojově čitelný identifikátor,
- `message` je text pro člověka a může se měnit,
- `details` je volitelný seznam chyb polí.

## Omezení

| Omezení | Hodnota |
| --- | --- |
| Velikost těla požadavku | max. 1 MB |
| Limit požadavků | 100 za minutu na token, přihlášení 10 za minutu na IP |
| Nevalidní JSON | `400 Bad Request` |
| Nepodporovaný `Content-Type` | `415 Unsupported Media Type` |
| Překročený limit | `429 Too Many Requests` |

## Příklad

```http
POST /api/v1/courses HTTP/1.1
Authorization: Bearer eyJhbGciOi...
Content-Type: application/json

{ "title": "Úvod do REST", "description": "Základy návrhu API" }
```

```http
HTTP/1.1 201 Created
Location: /api/v1/courses/3f2b8c1e-5a6d-4c9a-8d2e-1b7e9f0a4c11

{
  "id": "3f2b8c1e-5a6d-4c9a-8d2e-1b7e9f0a4c11",
  "title": "Úvod do REST",
  "description": "Základy návrhu API",
  "teacherId": "a1b2c3d4-0000-4000-8000-000000000001",
  "createdAt": "2026-10-18T08:30:00Z"
}
```
