# Autentizace a oprávnění

API používá **JWT access tokeny**, **refresh tokeny**, role (RBAC) a kontrolu vlastnictví zdrojů.

- **Autentizace** určuje, *kdo jsi*. Selže-li, API vrací `401`.
- **Autorizace** určuje, *co smíš*. Selže-li, API vrací `403`.

## Přihlášení

1. Klient pošle e-mail a heslo na `POST /api/v1/auth/login`.
2. API ověří heslo proti hashi (Argon2id nebo bcrypt, heslo se neukládá v čitelné podobě).
3. API vrátí **access token** (platnost 15 minut) a **refresh token** (7 dní).
4. Klient posílá access token v každém chráněném požadavku:

```http
Authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCJ9...
```

```json
{
  "accessToken": "eyJhbGciOiJSUzI1NiIs...",
  "refreshToken": "d7f3a9...",
  "tokenType": "Bearer",
  "expiresIn": 900
}
```

## Obsah JWT

Token je podepsaný (RS256), ale ne šifrovaný, proto neobsahuje citlivé údaje.

| Tvrzení | Význam |
| --- | --- |
| `sub` | id uživatele |
| `role` | `student`, `teacher` nebo `admin` |
| `iat`, `exp` | čas vydání a expirace |
| `jti` | jedinečné id tokenu |

## Obnovení a odhlášení

- Po vypršení access tokenu (`401`) klient zavolá `POST /api/v1/auth/refresh` s refresh tokenem a dostane nový pár tokenů.
- Používá se **rotace**: starý refresh token se po použití zneplatní. Pokud se použije znovu, API ho považuje za ukradený a zneplatní celou řadu tokenů uživatele.
- `POST /api/v1/auth/logout` zneplatní refresh token. Access token platí nejvýše 15 minut.

## Role

| Role | Popis |
| --- | --- |
| `student` | Zapisuje se do kurzů, čte kurzy a lekce, spravuje svůj profil. |
| `teacher` | Vytváří a spravuje **své** kurzy a lekce, vidí jejich studenty. |
| `admin` | Plný přístup. |

Nový uživatel je `student`. Roli mění jen `admin`.

## Matice oprávnění

✅ povoleno, 🔒 jen vlastník (nebo `admin`), ❌ zakázáno, 🌐 bez přihlášení.

| Endpoint | student | teacher | admin |
| --- | --- | --- | --- |
| `POST /auth/login`, `/auth/refresh` | 🌐 | 🌐 | 🌐 |
| `POST /auth/logout` | ✅ | ✅ | ✅ |
| `GET /courses`, `/courses/{id}`, `/courses/{id}/lessons`, `/lessons/{id}` | 🌐 | 🌐 | 🌐 |
| `POST /courses` | ❌ | ✅ | ✅ |
| `PUT`, `DELETE /courses/{id}` | ❌ | 🔒 | ✅ |
| `POST /courses/{id}/lessons`, `PATCH`, `DELETE /lessons/{id}` | ❌ | 🔒 | ✅ |
| `POST`, `DELETE /courses/{id}/enroll` | ✅ (sám sebe) | ❌ | ✅ |
| `GET /courses/{id}/students` | ❌ | 🔒 | ✅ |
| `GET /users/{id}/courses` | 🔒 | 🔒 | ✅ |
| `GET /enrollments/{id}` | 🔒 | 🔒 | ✅ |
| `PATCH /enrollments/{id}` | ❌ | 🔒 | ✅ |
| `GET /users` | ❌ | ❌ | ✅ |
| `POST /users` | 🌐 | 🌐 | 🌐 |
| `GET`, `PUT`, `DELETE /users/{id}` | 🔒 | 🔒 | ✅ |

„Vlastník" je u kurzu a lekce učitel, který kurz vytvořil. U uživatele a zápisu je to daný uživatel, u zápisu navíc učitel kurzu.

## Chybové stavy

| Situace | Kód |
| --- | --- |
| Chybí `Authorization`, token je neplatný nebo vypršel | `401` |
| Špatný e-mail nebo heslo (zpráva neříká, co z toho) | `401` |
| Role nemá právo nebo uživatel není vlastníkem | `403` |
| Příliš mnoho pokusů o přihlášení | `429` |

Odpověď `401` obsahuje hlavičku `WWW-Authenticate: Bearer`.

## Bezpečnostní doporučení

- Pouze **HTTPS**, tokeny jen v hlavičce, nikdy v URL.
- Přihlášení je omezeno na 10 pokusů za minutu na IP.
- Refresh token ukládejte bezpečně (ne do `localStorage`).
- Možné rozšíření: **OAuth 2.0** (Authorization Code s PKCE) pro školní účty.
