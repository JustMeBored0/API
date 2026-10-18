# Verzování API

API je verzované, aby se mohlo vyvíjet bez rozbití existujících klientů.

## Jak se verze určuje

Hlavní verze je součástí cesty každého endpointu:

```
https://api.example.com/api/v1/courses
https://api.example.com/api/v2/courses
```

- Mění se jen **hlavní** verze (`v1`, `v2`, ...). Jiný způsob výběru verze neexistuje.
- Konkrétní vydání (např. `1.3.0`) se řídí [sémantickým verzováním](https://semver.org/lang/cs/) a vrací se v hlavičce `X-API-Version`.
- Více hlavních verzí může běžet současně.

## Kompatibilní změny

Nezvyšují hlavní verzi:

- nový endpoint (`GET /api/v1/courses/{id}/reviews`),
- nové nepovinné pole nebo parametr,
- nová hodnota výčtu v odpovědi,
- změna textu v `message`.

Klienti proto **musí ignorovat neznámá pole** v odpovědích.

## Zásadní změny (breaking changes)

Vyžadují novou hlavní verzi:

- odebrání nebo přejmenování endpointu, pole či parametru,
- změna typu pole (např. `id` z řetězce na číslo),
- nové povinné pole nebo parametr v požadavku,
- změna významu stavového kódu nebo pravidel oprávnění.

## Životní cyklus verze

| Fáze | Popis |
| --- | --- |
| **Current** | Doporučená verze, dostává nové funkce. |
| **Supported** | Jen opravy chyb a bezpečnostní záplaty. |
| **Deprecated** | Odpovědi obsahují hlavičky `Deprecation` a `Sunset`. |
| **Retired** | Endpointy vrací `410 Gone`. |

Předchozí verze zůstane po vydání nové **podporovaná nejméně 12 měsíců**. Zrušení se oznamuje nejméně 6 měsíců předem v changelogu a e-mailem a hlavičkami v každé odpovědi:

```http
Deprecation: true
Sunset: Wed, 31 Dec 2027 23:59:59 GMT
Link: <https://api.example.com/docs/migration-v2>; rel="deprecation"
```

K nové hlavní verzi vznikne migrační průvodce s příklady před/po. Vyřazená verze odpoví:

```json
{
  "code": "VERSION_RETIRED",
  "message": "API verze v1 byla vyřazena. Použijte v2."
}
```
