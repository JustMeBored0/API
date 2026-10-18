# Zdroje API

API zveřejňuje čtyři zdroje a autentizační endpointy. Parametry, hlavičky, těla, příklady a důvody stavových kódů jsou u každého endpointu v [API Reference](/api-reference). Všechny cesty mají prefix `/api/v1`.

## Přehled a vazby

| Zdroj | Popis |
| --- | --- |
| **Users** | uživatelé (studenti, učitelé, administrátoři) |
| **Courses** | kurzy; každý má jednoho učitele (1:N) |
| **Lessons** | lekce; každá patří jednomu kurzu (1:N) |
| **Enrollments** | zápis uživatele do kurzu (**M:N** mezi Users a Courses) |

```
User ──< Enrollment >── Course ──< Lesson
```

- Student může být zapsán v mnoha kurzech a kurz má mnoho studentů.
- Vazba M:N je samostatný zdroj **Enrollment** s atributy `status` (`active`, `completed`, `dropped`), `grade` a `enrolledAt`.
- Dvojice (`userId`, `courseId`) je jedinečná, druhý zápis proto vrací `409 Conflict`.
- Smazání kurzu smaže i jeho lekce a zápisy (kaskádově).

## CRUD přehled

| Zdroj | Create | Read | Update | Delete |
| --- | --- | --- | --- | --- |
| Users | `POST /users` | `GET /users`, `GET /users/{id}` | `PUT /users/{id}` | `DELETE /users/{id}` |
| Courses | `POST /courses` | `GET /courses`, `GET /courses/{id}` | `PUT /courses/{id}` | `DELETE /courses/{id}` |
| Lessons | `POST /courses/{id}/lessons` | `GET /courses/{id}/lessons`, `GET /lessons/{id}` | `PATCH /lessons/{id}` | `DELETE /lessons/{id}` |
| Enrollments | `POST /courses/{id}/enroll` | `GET /courses/{id}/students`, `GET /users/{id}/courses`, `GET /enrollments/{id}` | `PATCH /enrollments/{id}` | `DELETE /courses/{id}/enroll` |

## Endpointy

| Zdroj | Metoda | Cesta | Popis | Kódy |
| --- | --- | --- | --- | --- |
| Auth | POST | `/auth/login` | získání tokenů | 200, 400, 401, 429 |
| | POST | `/auth/refresh` | obnovení tokenů | 200, 400, 401 |
| | POST | `/auth/logout` | zneplatnění refresh tokenu | 204, 401 |
| Courses | GET | `/courses` | seznam (filtrování, stránkování) | 200, 400 |
| | POST | `/courses` | vytvoření kurzu | 201, 400, 401, 403 |
| | GET | `/courses/{id}` | detail kurzu | 200, 404 |
| | PUT | `/courses/{id}` | úplná aktualizace | 200, 400, 401, 403, 404 |
| | DELETE | `/courses/{id}` | smazání kurzu | 204, 401, 403, 404 |
| Lessons | GET | `/courses/{id}/lessons` | lekce kurzu | 200, 404 |
| | POST | `/courses/{id}/lessons` | přidání lekce | 201, 400, 401, 403, 404 |
| | GET | `/lessons/{id}` | detail lekce | 200, 404 |
| | PATCH | `/lessons/{id}` | částečná úprava | 200, 400, 401, 403, 404 |
| | DELETE | `/lessons/{id}` | odstranění lekce | 204, 401, 403, 404 |
| Enrollments | POST | `/courses/{id}/enroll` | zápis do kurzu | 201, 401, 403, 404, 409 |
| | DELETE | `/courses/{id}/enroll` | zrušení zápisu | 204, 401, 404 |
| | GET | `/courses/{id}/students` | studenti kurzu | 200, 401, 403, 404 |
| | GET | `/users/{id}/courses` | kurzy studenta | 200, 401, 403, 404 |
| | GET | `/enrollments/{id}` | detail zápisu | 200, 401, 403, 404 |
| | PATCH | `/enrollments/{id}` | změna stavu nebo známky | 200, 400, 401, 403, 404 |
| Users | GET | `/users` | seznam (jen admin) | 200, 401, 403 |
| | POST | `/users` | registrace | 201, 400, 409 |
| | GET | `/users/{id}` | detail profilu | 200, 401, 403, 404 |
| | PUT | `/users/{id}` | úprava profilu | 200, 400, 401, 403, 404 |
| | DELETE | `/users/{id}` | deaktivace účtu | 204, 401, 403, 404 |

## Význam stavových kódů

| Kód | Kdy nastane |
| --- | --- |
| `200 OK` | požadavek proběhl, odpověď má tělo |
| `201 Created` | zdroj vznikl, hlavička `Location` obsahuje jeho URL |
| `204 No Content` | akce proběhla bez těla odpovědi (smazání) |
| `400 Bad Request` | nevalidní JSON nebo chybné pole |
| `401 Unauthorized` | chybí, vypršel nebo je neplatný token, nebo špatné přihlašovací údaje |
| `403 Forbidden` | uživatel je přihlášen, ale role nebo vlastnictví akci nedovolí |
| `404 Not Found` | zdroj s daným `id` neexistuje |
| `409 Conflict` | konflikt s daty (již zapsán, duplicitní e-mail) |
| `429 Too Many Requests` | překročen limit požadavků |
