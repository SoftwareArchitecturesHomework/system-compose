<center>

# Szoftverarchitektúrák házi feladat dokumentáció<br>Céges munkaidő napló

</center>

<br>

<img src="../core-app/public/logo-light.png" alt="Logo" style="display: block;
    margin: 0 auto;
    max-width: 100%;" />

<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>
<br>

| Név              | Neptun |
| ---------------- | ------ |
| Farkas Fanni     | EU7XYP |
| Bertók Dániel    | H01HRM |
| Dremák Gergely   | KSHSLY |
| Forrás Máté Ákos | ONQ3KU |

- [Szoftverarchitektúrák házi feladat dokumentációCéges munkaidő napló](#szoftverarchitektúrák-házi-feladat-dokumentációcéges-munkaidő-napló)
  - [Rendszer célja és környezete](#rendszer-célja-és-környezete)
    - [Feladatkiírás](#feladatkiírás)
    - [Funkciók](#funkciók)
    - [Rendszer környezete](#rendszer-környezete)
  - [Architektúra](#architektúra)
    - [Architektúra áttekintése](#architektúra-áttekintése)
    - [Architektúrális döntések és indoklásuk](#architektúrális-döntések-és-indoklásuk)
      - [Microservice architektúra](#microservice-architektúra)
      - [REST API kommunikáció](#rest-api-kommunikáció)
      - [NUXT full-stack keretrendszer](#nuxt-full-stack-keretrendszer)
      - [PostgreSQL adatbázis](#postgresql-adatbázis)
      - [JWT alapú autentikáció](#jwt-alapú-autentikáció)
  - [Rétegek leírása](#rétegek-leírása)
    - [Adatbázis réteg (Database Layer)](#adatbázis-réteg-database-layer)
    - [Repository réteg (Data Access Layer)](#repository-réteg-data-access-layer)
    - [Service réteg (Business Logic Layer)](#service-réteg-business-logic-layer)
    - [API réteg (Route Handlers)](#api-réteg-route-handlers)
    - [Authentication réteg](#authentication-réteg)
    - [Prezentációs réteg (Frontend)](#prezentációs-réteg-frontend)
    - [Rétegek közötti kommunikáció](#rétegek-közötti-kommunikáció)
  - [Microsevice-ek](#microsevice-ek)
    - [Comms Szolgáltatás – (Elixir/Phoenix)](#comms-szolgáltatás--elixirphoenix)
      - [1. Cél és Hatókör](#1-cél-és-hatókör)
      - [2. Fő Összetevők](#2-fő-összetevők)
        - [E‑mail](#email)
        - [Discord](#discord)
      - [3. Adat és Sablonkezelés](#3-adat-és-sablonkezelés)
      - [4. Folyamatok](#4-folyamatok)
        - [E‑mail Küldés](#email-küldés)
        - [Discord Interaction](#discord-interaction)
        - [Discord Command Telepítés](#discord-command-telepítés)
      - [5. Hibakezelés](#5-hibakezelés)
      - [6. Biztonság](#6-biztonság)
      - [7. Egyszerű Metrikák / Logok (Aktuális Állapot)](#7-egyszerű-metrikák--logok-aktuális-állapot)
      - [8. Tesztek](#8-tesztek)
    - [Reporting Service (Python)](#reporting-service-python)
  - [Telepítési leírás](#telepítési-leírás)
    - [Development](#development)
      - [Futtatás fejlesztői módban:](#futtatás-fejlesztői-módban)
        - [Core Application](#core-application)
        - [Communications Service](#communications-service)
        - [Reporting Service](#reporting-service)
      - [Adatbázis](#adatbázis)
    - [Production](#production)
      - [Előkészületek :](#előkészületek-)
      - [PostgreSQL adatbázis:](#postgresql-adatbázis-1)
      - [Core/Migrator/Reporting szolgáltatások:](#coremigratorreporting-szolgáltatások)
      - [Migrator:](#migrator)
      - [Core Application (Nuxt):](#core-application-nuxt)
      - [Communications Service (Elixir/Phoenix):](#communications-service-elixirphoenix)
      - [Reporting Service (Python):](#reporting-service-python-1)
    - [Deployment](#deployment)
      - [Futtatás lokálisan buildelt image-ekkel:](#futtatás-lokálisan-buildelt-image-ekkel)
      - [Futtatás Docker Hubos image-ekkel:](#futtatás-docker-hubos-image-ekkel)

## Rendszer célja és környezete

### Feladatkiírás

A projekt célja egy olyan munkaidő napló webes alkalmazás létrehozása, amely megkönnyíti a projektek menedzselését, és azokról statisztikákat állít elő egy vállalat dolgozói számára.
A rendszerben felvehetőek projektek, amelyeknek van egy kezdési, és lehet egy tervezett befejezési dátuma. Ezek a projektek megtekinthetőek egy naptár nézetben, illetve az alkalmazottak minden eseményükhöz (pl. megbeszélés, feladatok) rögzíthetik, hogy melyik projekttel foglalkoztak. Emellett a dolgozók a naptárba felvehetik a tervezett szabadnapjaikat, amelyeket a közvetlen felettesüknek kell jóváhagynia.
A felettesek megnézhetik a beosztottjaik jóváhagyásra váró szabadnapjait, illetve amikor azok felvételre kerülnek a naptárba, arról E-Mail értesítést is kapnak. Opcionális célunk, hogy E-Mail mellett más értesítési formák is megvalósításra kerüljenek, elsősorban Discord értesítés formájában.

### Funkciók

- **Projektek kezelése**
  - Új projekt létrehozása (név, kezdési dátum, tervezett befejezési dátum)
  - Projektek megtekintése (lista és részletes nézet)
  - Résztvevők hozzáadása a projekthez
  - Résztvevők eltávolítása a projektből
  - Projekt lezárása (tényleges befejezési dátum rögzítése)
- **Feladatok kezelése**
  - Feladat létrehozása projekthez rendelten
    - Discord integrációval: slash commandok segítségével is
  - Feladat részleteinek megtekintése
  - Feladat hozzárendelése dolgozóhoz
    - Discord integrációval: slash commandok segítségével is
  - Feladat hozzárendelés módosítása vagy törlése
  - Feladat státuszának változtatása (befejezett/folyamatban)
  - Feladat törlése (csak létrehozó által)
- **Munkaidő nyilvántartás**
  - Munkaidő bejegyzések rögzítése feladatokhoz
  - Rögzített adatok: dátum, óraszám, megjegyzés
  - Munkaidő bejegyzések megtekintése
  - Saját munkaidő bejegyzések törlése
- **Megbeszélések**
  - Megbeszélés létrehozása
  - Résztvevők hozzáadása a megbeszéléshez
  - Kezdési és befejezési időpont megadása
- **Szabadságok kezelése**
  - Szabadság igénylés létrehozása
  - Szabadság jóváhagyása vagy elutasítása (menedzser által)
  - Jóváhagyásra váró szabadságok listázása
- **Adminisztrációs felület (csak menedzsereknek)**
  - Dolgozók munkaidő adminisztrációjának megtekintése
  - Szűrés év és hónap alapján
  - Adminisztrált órák és kötelező órák összehasonlítása
  - Szabadságok figyelembevétele a kötelező órákból
  - Státusz jelzés (elegendő/hiányos/nincs bejegyzés)
- **Naptár nézet**
  - Meetingek, szabadságok és események megtekintése a naptárban
  - Új meeting hozzáadása és módosítása
  - Új szabadság hozzáadása és módosítása
  - Új esemény hozzáadása és módosítása
- **Felhasználói rendszer**
  - Bejelentkezés Google OAuth-al
  - Bejelentkezés email és jelszóval
  - Szerepkör alapú jogosultságkezelés (Menedzser, Alkalmazott)
- **Értesítések**
  - Email értesítések szabadság igénylésekről
- **Riportok**
  - HTML és PDF kiterjesztésű riportok megtekintése, letöltése

### Rendszer környezete

A rendszer egy modern webes alkalmazás, amely három fő komponensből áll. A központi alkalmazás egy Nuxt alapú full-stack web alkalmazás, amely a felhasználói felületet (Vue komponensek), az üzleti logikát és az API végpontokat egyaránt tartalmazza. Az alkalmazás PostgreSQL relációs adatbázist használ az adatok tárolására, amely Prisma ORM segítségével kerül elérésre.

A hitelesítés NextAuth (Auth.js) könyvtárral van megvalósítva, amely támogatja mind a Google és Discord OAuth alapú bejelentkezést, mind az email-jelszó páros hitelesítést. A munkamenetek JWT tokenekkel vannak kezelve, biztosítva a biztonságos felhasználói azonosítást. A rendszer szerepkör alapú hozzáférés-vezérlést (RBAC) implementál, két szerepkörrel: Menedzser és Alkalmazott.

Az értesítési szolgáltatásokat egy Elixir nyelven írt microservice kezeli, amely felelős az email és Discord értesítések kiküldéséért. Emellett egy Python alapú microservice gondoskodik a riportok és statisztikák generálásáról. A három komponens REST API-kon keresztül kommunikál egymással.

A rendszer Docker compose-al futtatható, megkönnyítve a fejlesztést és a telepítést. A frontend modern, reszponzív felületet biztosít Nuxt UI komponenskönyvtár és TailwindCSS segítségével, támogatva a világos és sötét témákat is.

## Architektúra

### Architektúra áttekintése

A rendszer microservice alapú architektúrát követ, amely három fő komponensből áll:

1. **Core Application** (Nuxt) - Központi webes alkalmazás
2. **Communications Service** (Elixir/Phoenix) - Értesítési microservice
3. **Reporting Service** (Python) - Riportgeneráló microservice

```mermaid
graph LR

User["Users / Frontend Clients"]


CoreAPI["Core Application (Nuxt.js)"]


subgraph Micro["Microservices"]
    Comms["Communications Service"]
    Reporting["Reporting Service"]
end

subgraph DB["Database"]
    Postgres["PostgreSQL"]
end

subgraph Outside["Outside context"]
    Email["SMTP Email Service"]
    Discord["Discord"]
end

User --> CoreAPI
CoreAPI --> Comms
Comms --> CoreAPI
Comms --> Email
Comms --> Discord
Discord --> Comms
User --> Reporting
CoreAPI --> Postgres
Reporting --> Postgres

```

### Architektúrális döntések és indoklásuk

#### Microservice architektúra

A rendszer microservice alapú felépítést követ, amely az alábbi előnyöket nyújtja:

**Szeparált felelősségek**: Minden microservice egy konkrét üzleti funkciót lát el (értesítések, riportok), amely egyszerűsíti a karbantartást és fejlesztést.

**Technológiai függetlenség**: Minden szolgáltatás a feladatköréhez jól passzoló technológiával készült:

- **Elixir/Phoenix** az értesítési szolgáltatáshoz - kiváló konkurencia kezelés, hibatűrés
- **Python** a riportgeneráláshoz - gazdag adatelemző és statisztikai könyvtárak (pandas, matplotlib)
- **Nuxt** a core alkalmazáshoz - teljes stack fejlesztés egyetlen keretrendszerben

**Skálázhatóság**: A komponensek egymástól függetlenül skálázhatók. Például az értesítési szolgáltatás külön skálázható nagy terhelés esetén.

**Hibatűrés**: Egy microservice hibája nem okozza a teljes rendszer leállását. Ha a riportgeneráló szolgáltatás nem érhető el, a core funkciók továbbra is működnek.

#### REST API kommunikáció

A microserviceek közötti kommunikációhoz REST API-kat választottunk:

**Egyszerűség**: A REST jól ismert, széles körben támogatott protokoll, amely megkönnyíti az integrációt.

**Stateless kommunikáció**: Minden kérés független, ami egyszerűsíti a rendszer állapotkezelését.

**Platform függetlenség**: Bármilyen HTTP kliens képes kommunikálni a szolgáltatásokkal.

#### NUXT full-stack keretrendszer

A core alkalmazáshoz a Nuxt.js-t választottuk:

**Unified codebase**: Frontend és backend azonos nyelvben (TypeScript/JavaScript), csökkentve a kontextusváltást.

**Server-Side Rendering (SSR)**: Jobb SEO és gyorsabb kezdeti oldalbetöltés.

**File-based routing**: Az API végpontok és oldalak könyvtárstruktúrán alapuló routing-ja egyszerűsíti a fejlesztést.

**Auto-imports**: Automatikus importálás csökkenti a boilerplate kódot.

**Type safety**: TypeScript támogatás az egész stacken keresztül.

#### PostgreSQL adatbázis

Relációs adatbázist választottunk az alábbi okok miatt:

**ACID tulajdonságok**: Tranzakciós biztonság kritikus a munkaidő-nyilvántartásnál.

**Komplex kapcsolatok**: A rendszerben számos entitás áll kapcsolatban (felhasználók, projektek, feladatok, munkaidő bejegyzések).

**Strukturált adatok**: Az adatok sémája jól definiált és stabil.

**Prisma ORM integráció**: Type-safe adatbázis hozzáférés, automatikus migrációk.

#### JWT alapú autentikáció

A munkamenet kezeléshez JWT tokeneket használunk:

**Microservice kompatibilitás**: A tokenek könnyen validálhatók a különböző szolgáltatásokban.

**CredentialsProvider támogatás**: Lehetővé teszi mind az OAuth, mind az email-jelszó alapú bejelentkezést.

**Biztonságos**: A tokenek aláírva tartalmazzák a felhasználói információkat, határozott élettartammal.

## Rétegek leírása

A Core Application többrétegű architektúrát követ, amely egyértelműen elkülöníti a különböző felelősségeket. Az alkalmazás rétegei alulról felfelé:

### Adatbázis réteg (Database Layer)

Ez a legalsó réteg, amely a PostgreSQL adatbázist tartalmazza. A Prisma ORM biztosítja az adatbázis séma kezelését és a migrációkat. Az adatbázis tárolja az összes perzisztens adatot: felhasználókat, projekteket, feladatokat, munkaidő bejegyzéseket és egyéb entitásokat.

**Prisma séma** (`/prisma/schema.prisma`):

- Típusbiztos adatmodell definíciók
- Kapcsolatok és constraint-ek definiálása
- Automatikus migráció generálás
- Automatikus seedelés biztosítása fejlesztéshez

**Adatmodell (Entity-Relationship Diagram)**:

```mermaid
erDiagram
  USER {
    int id PK
    string email
    string password
    string name
    string image
    datetime emailVerified
    int managerId FK
    string role
  }

  ACCOUNT {
    int id PK
    int userId FK
    string type
    string provider
    string providerAccountId
    string refresh_token
    string access_token
    int expires_at
    string token_type
    string scope
    string id_token
    string session_state
  }

  SESSION {
    int id PK
    string sessionToken
    int userId FK
    datetime expires
  }

  VERIFICATIONTOKEN {
    string identifier
    string token PK
    datetime expires
  }

  PROJECT {
    int id PK
    string name
    datetime startDate
    datetime endDate
    datetime plannedEndDate
    int ownerId FK
  }

  USERPROJECT {
    int userId FK
    int projectId FK
  }

  TASK {
    int id PK
    string type
    int creatorId FK
    datetime createdAt
    string name
    datetime startDate
    datetime endDate
    string description
    int projectId FK
    int assigneeId FK
    datetime dueDate
    bool isDone
    bool isApproved
  }

  MEETINGPARTICIPANT {
    int id PK
    int meetingId FK
    int userId FK
  }

  TIMEENTRY {
    int id PK
    int taskId FK
    int userId FK
    datetime date
    float hours
    string note
  }

  %% Self-relation: manager ↔ employees
  USER ||--o{ USER : "manages (managerId)"

  %% User auth
  USER ||--o{ ACCOUNT : "has accounts"
  USER ||--o{ SESSION : "has sessions"

  %% Projects
  USER ||--o{ PROJECT : "owns"
  USER ||--o{ USERPROJECT : "assigned to projects"
  PROJECT ||--o{ USERPROJECT : "has users"

  %% Tasks
  USER ||--o{ TASK : "creates (creator)"
  USER ||--o{ TASK : "assigned (assignee)"
  PROJECT ||--o{ TASK : "has tasks"

  %% Meetings (Task with type = MEETING)
  TASK ||--o{ MEETINGPARTICIPANT : "has participants"
  USER ||--o{ MEETINGPARTICIPANT : "attends"

  %% Time entries
  TASK ||--o{ TIMEENTRY : "has time entries"
  USER ||--o{ TIMEENTRY : "logs time"
```

**Főbb entitások**:

- **User**: Felhasználók, hierarchikus manager-employee kapcsolattal, szerepkör támogatással (EMPLOYEE, MANAGER role-ok)
- **Account**: OAuth provider fiókok (Google, Discord)
- **Session**: Felhasználói munkamenetek (JWT esetén nem használt)
- **Project**: Projektek kezdési és befejezési dátumokkal
- **UserProject**: Kapcsoló tábla felhasználók és projektek között (many-to-many)
- **Task**: Többcélú entitás - feladatok, megbeszélések, szabadságok, egyéni feladatok
- **TimeEntry**: Munkaidő bejegyzések feladatokhoz rendelve
- **MeetingParticipant**: Kapcsolótábla egy megbeszélés és résztvevői között

### Repository réteg (Data Access Layer)

A repository réteg biztosítja az adathozzáférési logikát. Minden entitáshoz tartozik egy repository, amely elrejti a Prisma specifikus implementációt.

**Felelősségek**:

- CRUD műveletek implementálása
- Adatbázis lekérdezések összeállítása
- Kapcsolódó entitások betöltése (includes, relations)
- Adatbázis hibák kezelése

**Főbb repository-k**:

- `userRepository.ts` - Felhasználók kezelése
- `projectRepository.ts` - Projektek kezelése
- `taskRepository.ts` - Feladatok kezelése
- `timeEntryRepository.ts` - Munkaidő bejegyzések kezelése
- `userProjectRepository.ts` - Felhasználó-projekt kapcsolatok

### Service réteg (Business Logic Layer)

**Jelenlegi implementáció**:

A Core Application esetében a service réteg **jelenleg nincs külön kiválasztva**, mivel az alkalmazás főként CRUD műveleteket végez, ahol az üzleti logika minimális. A legtöbb endpoint esetében a logika elsősorban **validációból** áll (pl. autentikáció ellenőrzés, jogosultság ellenőrzés, input validálás), amely közvetlenül az API rétegben van implementálva.

**Komplex üzleti logika elhelyezése**:

A rendszer architektúrájában a jelentős üzleti logika a **microservice-ekben** található:

- **Communications Service** (Elixir): Értesítési szabályok, email template generálás, Discord integráció logika
- **Reporting Service** (Python): Statisztikai számítások, riportgenerálás, adatelemzés

A Core Application felelőssége ezért elsősorban:

- Adatok perzisztálása és lekérése
- Felhasználói hitelesítés és jogosultságkezelés
- Alapvető input validáció
- microserviceek koordinálása REST API hívásokon keresztül

**Jövőbeli fejlesztés**: Amennyiben a Core Application-ben több komplex üzleti logika jelenne meg (pl. összetett munkaidő számítások, automatikus projekt értékelések), akkor érdemes lenne a service réteget kiemelni az API handler-ekből a jobb szeparáció és tesztelhetőség érdekében.

### API réteg (Route Handlers)

Az API réteg (`/server/api/`) tartalmazza a HTTP végpontokat. Nuxt.js fájlrendszer alapú routing-ok használ, ahol a fájlstruktúra határozza meg az endpoint-okat.

**Felelősségek**:

- HTTP kérések fogadása
- Autentikáció és autorizáció ellenőrzése
- Request validálás
- Repositoryk hívása
- Response formálása
- HTTP státuszkódok kezelése

### Authentication réteg

Az autentikáció NextAuth (Auth.js) könyvtárral van implementálva (`/server/api/auth/[...].ts`).
Minden endpoint-ot csak bejelentkezett felhasználók érnek el. Az adott felhasználó jogosultsága validálva van az egyes kéréseknél. PL.: Task-ot csak a létrehozója törölhet

**Támogatott provider-ek**:

- Google OAuth 2.0
- Discord OAuth
- Email-jelszó (Credentials Provider)

**JWT munkamenet**:

- Stateless session kezelés
- Token tartalmazza: user id, email, name, role, image
- Biztonságos titkosítás a NEXTAUTH_SECRET-tel

### Prezentációs réteg (Frontend)

A frontend Vue.js komponensekből áll, Nuxt.js keretrendszerben (`/app/`). A komponensek reszponzívak, támogatják a sötét és világos témákat, ARIA attribútumokkal vannak ellátva a jobb hozzáférhetőség érdekében és TailwindCSS-el vannak dizájnolva.

A Nuxt a front-end is az apihoz hasonlóan fájlrendszer alapú routingot használ, így az oldalak könnyen megtalálhatók a könyvtárstruktúrában és a jobb SEO és teljesítmény érdekében server-side renderinget (SSR) alkalmaz front-end hydration-nel a dinamikus interakciókhoz.

**Komponensek** (`/app/components/`):

- `ProjectCard.vue` - Projekt megjelenítés
- `TimeAdministrationCard.vue` - Munkaidő adminisztráció lista elem
- `AddTimeEntryModal.vue` - Munkaidő rögzítés modal
- `TaskAssigneeModal.vue` - Feladat hozzárendelés modal
- És további komponensek...

**Oldalak** (`/app/pages/`):

- `login/` - Bejelentkezés
  <center>

  ![login](image-5.png)

  </center>

- `projects/` - Projekt menedzsment oldalak
  <center>

  ![project lis](image.png)

  ![project details](image-1.png)

  </center>

- `tasks/` - Feladat kezelés oldalak
  <center>

  ![task details](image-2.png)

  </center>

- `administration/` - Adminisztrációs felület
  <center>

  ![administration](image-3.png)

  </center>

- `approvals/` - Jóváhagyások
  <center>

  ![approvals](image-4.png)

  </center>

- `calendar/` - naptár nézet
  <center>

  ![calendar](image-6.png)

  </center>
  - új esemény létrehozása
    <center>

  ![create_new_event](image-7.png)

    </center>
  - új meeting létrehozása
    <center>

  ![create_new_meeting](image-8.png)

  ![create_new_meeting](image-9.png)

  ![create_new_meeting](image-10.png)

    </center>
  - új szabadság létrehozása
    <center>

  ![create_vacation](image-11.png)

    </center>
  - a módosító felületeken ugyanezek jönnek be, az adatok betöltésével:
    <center>

  ![update_meeting](image-12.png)

    </center>

**Composable-ök** (`/app/composables/`):

- `useUser.ts` - A felhasználó adatainak session-ből kiolvasását könnyíti.

**UI könyvtár**: Nuxt UI komponensek TailwindCSS-el

- Sötét/világos téma támogatás
- Reszponzív és szép design
- Sok előre elkészített komponens

**Naptár könyvtár**: FullCalendar - robusztus és rugalmas naptárkönyvtár

- Többféle nézet: Támogatja a fő naptárnézeteket, mint a hónap, hét (időráccsal vagy anélkül), nap és a különböző lista nézetek
- Interaktivitás: Támogatja az eseményekkel való interakciót (pl. drag and drop, kattintás).

### Rétegek közötti kommunikáció

```mermaid
graph TD
    UI[Prezentációs réteg<br/>Vue komponensek]
    API[API réteg<br/>Route handlers]
    Auth[Authentication réteg<br/>NextAuth]
    Repo[Repository réteg<br/>Data Access]
    DB[(PostgreSQL<br/>Database)]

    UI -->|HTTP Request| API
    API -->|Authorize| Auth
    API -->|Query| Repo
    Repo -->|Prisma ORM| DB
    Auth --> |Prisma ORM| DB
```

**Elvek**:

- **Separation of Concerns**: Minden réteg egyetlen felelősséggel rendelkezik
- **Dependency Direction**: Fentről lefelé (UI → API → Service → Repository → DB)
- **Abstraction**: Minden réteg elrejti az implementációs részleteket
- **Testability**: Rétegek külön-külön unit tesztelhetők

## Microsevice-ek

A rendszer két különálló microserviceszel egészül ki, amelyek a Core Application-től függetlenül futnak és specifikus üzleti funkciókat látnak el.

### Comms Szolgáltatás – (Elixir/Phoenix)

#### 1. Cél és Hatókör

- E‑mailek küldése üzleti események (feladat kiosztás, projekt státusz, szabadság kérés stb.) alapján.
- Discord slash command / interaction kérések fogadása, validálása, aláírás ellenőrzése és válasz generálása.

#### 2. Fő Összetevők

##### E‑mail

- Swoosh (`Comms.Mailer`) a küldéshez; konfiguráció `:comms` OTP alkalmazás alatt (pl. `smtp_from_email`).
- Sablonok: `priv/templates/email/*.html.eex` – klasszikus `.eex`. Minden sablon a várt assign kommenttel indul (pl. `# assigner: %{name: String, email: String}`).
- Builder: `Comms.Notifications` modul állítja össze a `Swoosh.Email` struktúrát (from, to, subject, body). A body renderelést Phoenix sablonmotor végzi a megfelelő path alapján.

##### Discord

Mivel Elixirhez nem találtam jól karbantartott Discord könyvtárat, a beépített Req HTTP klienst használtam a Discord API hívásokhoz és a hivatalos JS könyvtár forráskódját tanulmányoztam a számunkra szükséges funkciók implementálásához.

- Router pipeline `:discord` és `CommsWeb.Plugs.VerifyDiscordSignature` plug a Discord aláírás ellenőrzéséhez (fejlécek: `X-Signature-Ed25519`, `X-Signature-Timestamp`).
- Végpontok:
  - `POST /discord/interactions` – slash command / component interactions.

#### 3. Adat és Sablonkezelés

- Sablonváltozók: közvetlenül a `render` híváskor kerülnek átadásra assign mapként; a sablonok végeznek interpolációt.
- Discord válaszok: általában JSON payload a Discord API elvárásai szerint (type, data). A validáció a signature ellenőrzés és a parancs típus alapján történik.

#### 4. Folyamatok

##### E‑mail Küldés

1. HTTP POST érkezik egy dedikált értesítési végpontra (pl. `/notifications/vacation-request`).
2. Controller / action validálja a minimális mezőket (címzett, JWT, kontextus adatok).
3. `Comms.Notifications` összeállítja a `Swoosh.Email` struktúrát: `from_email()` + dinamikus `to`.
4. Sablon render: `Phoenix.View.render("email/vacation_request.html", assigns)` – generált HTML a levél törzséhez.
5. `Comms.Mailer.deliver(email)` → küldés. Hibánál visszatér `{:error, reason}`.

##### Discord Interaction

1. Discord küld egy HTTP POST-ot az `interactions` végpontra, fejlécekben időbélyeg + aláírás.
2. `VerifyDiscordSignature` plug hitelesíti (Ed25519).
3. Controller dekódolja a JSON-t; slash command név alapján routing / dispatch.
4. Válasz JSON: ack / ephemeral üzenet / follow‑up trigger.
   - Az ephemeral a discord kontextusában azt jelenti, hogy csak a releváns user látja az adott üzenetet

##### Discord Command Telepítés

`mix discord.install_commands` futtatás:

- Ellenőrzi szükséges env változókat.
- Küldi az upsert kérést a Discord API-hoz Req segítségével.
  - A Discordnak meg kell mondani, hogy milyen parancsokat szeretnénk regisztrálni, a teljes parancs definíciót JSON formátumban kell elküldeni a megfelelő végpontra, beleértve a parancs nevét, leírását, és a paramétereket.
- Siker / hiba logolás, hibánál.
  - A dockerizált környezetben a task az rpc parancson keresztül futtatható; illetve magától is megpróbál lefutni a docker container indításkor (ha a környezeti változók rendelkezésre állnak).

```sh
# Manuális futtatás docker konténerben
docker exec <container> /app/bin/comms rpc 'Comms.Discord.Registrar.install_global_commands()'
```

#### 5. Hibakezelés

- E‑mail: Swoosh visszatérési érték alapján döntés. Nincs automatikus retry vagy backoff – hiba esetén a hívó fél (felsőbb szolgáltatás) kezeli az újrapróbálkozást.
- Discord: Signature hiba → 401 / 403; ismeretlen parancs → 400 egyszerű üzenettel; parse hiba → 422 JSON.
- Mix task: env hiány esetén `Mix.raise`; API hiba esetén status + body megjelenítés.

#### 6. Biztonság

- Discord aláírás ellenőrzés kötelező minden interakciónál. A Discord az endpoint regisztráció során küld több néhol direkt hibás PING kérést is, ezeket is ellenőrizni kell és a megfelelő hibakóddal válaszolni különben nem lesz sikeres a regisztráció.
- E‑mail végpontok: asszimetrikus JWT alapú hitelesítés (RS256). A token tartalmazza a kezdeményező felhasználó azonosítóját és jogosultságait.
- Sablonokban nincs futtatható kód injektálás: csak interpoláció a kapott assign értékekkel.

#### 7. Egyszerű Metrikák / Logok (Aktuális Állapot)

- Strukturált log: siker / hiba e‑mail küldés, Discord parancs neve, státusz.

#### 8. Tesztek

- Unit:
  - `Comms.Notifications` – sablon renderelés különböző assign-ekkel, e‑mail struktúra ellenőrzése.
  - Discord signature plug test: hamis Discord signature → 403.
- Integrációs:
  - Health controller: egyszerű GET → 200 OK.
  - Discord interactions controller: ismert parancsok → várt válasz struktúra.

### Reporting Service (Python)

A riportgeneráló szolgáltatás Python nyelven készült, adatelemzésre és statisztikák előállítására optimalizálva.
A feladata, hogy a Postgre adatbázisunkon elhelyezett aggregált view-k alapján jelentést készítsen az adott menedzserek projektjeiről, beosztottjairól.  
HTML-ben és PDF-ben generálja ki a jelentéseket, ezeket gRPC és REST végpontokon keresztül lehet lekérdezni.
A jelentések tartalmaznak szöveges adatokat, diagramokat, illetve táblázatokat, ezekre pár példa:

<center>

![report_diagrams](image-13.png)
![report_tables](image-14.png)

</center>

**Felelősségek**:

- Projekt előrehaladás riportok
- Dolgozói teljesítmény összesítők
- Exportálás különböző formátumokba
- Adatvizualizáció (grafikonok, diagramok)

**Működés**  
A szolgáltatás válaszidejének növelésének érdekében nem minden kérés alkalmával kerül végrehajtásra a szükséges adatok kinyerését szolgáló SQL lekérdezés, hanem előre definiált view-kból kérdez le az alkalmazás.
A view-k előre átgondolva, az ábrázolhatóságot figyelembe véve lettek megtervezve. Minden esetben a legfelső szinten a felhasználó id-ja szerint van csoportosítás, hiszen minden jelentés egy adott menedzser számára készül el.
A lekérdezett adatokból utána diagramokat készít a szoftver, ezeket png-ként elmenti egy temporális könyvtárba, hogy majd a riport generálásakor bele lehessen ágyazni őket.
A jelentés alapját egy .html kiterjesztésű template jelenti. Ebben a templateben a riportonként nem változó statikus szövegek mellett a későbbiekben kitöltendő változók, illetve css szabályok szerepelnek az igényes megjelenítést szolgálva. A sablon szövegekbe, táblázatokba beszúrásra kerülnek az adatbázisból kinyert adatok, illetve a base64 enkódolású képek (ezzel elérve, hogy beágyazottan, a ténylegesn png nélkül hordozható legyen a html fájl tartalma.). Ezen a ponton a jelentés készen is van, itt vagy megáll a folyamat, mivel a html végpont lett meghívva, vagy pedig pdf generálódik a html-ből. A végeredmény egy 3 oldalas pdf dokumentum, melyben az alábbi statisztikák szerepelnek:

- Menedzser alkalmazottjai által ledolgozott munkaórák hónapkonként
- Menedzser alkalmazottjainak egyes projektekhez elkönyvelt munkaórái
- Átlagos projekt időtartam
- Projektek becsült és tényleges hossza közti eltérés
- Projektekhez könyvelt munkaórák száma hónaponként
- Alkalmazottak teljes munkaideje projektenkként

**REST API végpontok**:

- `GET reports/manager/{manager_id}/pdf` - Adott id-val rendelkező menedzser riportját legenerálja, pdf-be kirendereli, majd application/pdf formátumban visszatér vele.
- `GET reports/manager/{manager_id}/html` - Adott id-val rendelkező menedzser riportjának a generálása html-ben, visszatérés vele.

**gRPC végpontok**

- `rpc GetManagerHTML` - Adott id-val rendelkező menedzser riportjának a generálása html-ben, visszatérés vele, egy string formájában.
- `rpc GetManager` - Adott id-val rendelkező menedzser riportjának a generálása pdf-ben, visszatérés vele base64-elt pdf formájában.
- `rpc GetAllReportsOfManager` - Előző két pont eredményeit egyben adja vissza
- `rpc GetAllManagerReports` - Az összes menedzser, összes jelentését visszaadja, egyéb metaadatokkal (pl.: a jelentésekben szereplő képek külön) együtt.

## Telepítési leírás

### Development

Az itt leírt eszközök természetesen dockerizált fejlesztési környezetben is használhatók.

- Javasolt:
  - Docker
  - Docker Compose
  - `ngrok` vagy hasonló eszköz helyi szerverek publikus eléréséhez (Discord interactions endpoint teszteléséhez)
  - `@antfu/ni` - node fejlesztői környezet menedzseléséhez
- Szükséges:
  - Node.js (ajánlott verzió: 22.x vagy újabb)
    - alterannatívaként használható Deno vagy Bun
  - NPM (Node Package Manager, a Node.js telepítéssel együtt települ)
  - PNPM csomagkezelő
    - Telepítés: `npm install -g pnpm`
  - Erlang OTP (ajánlott verzió: 26.x vagy újabb)
  - Elixir (ajánlott verzió: 1.17.x vagy újabb)
  - Mix (Elixir build tool, az Elixir telepítéssel együtt települ)
  - Python (ajánlott verzió: 3.10 vagy újabb)
  - pip (Python csomagkezelő, a Python telepítéssel együtt települ)
  - Virtualenv (Python virtuális környezet menedzseléséhez)
    - Telepítés: `pip install virtualenv`
  - Visual Studio Code / Zed / nvim / bármilyen kedvenc kódszerkesztő ami támogatja a következő vagy hasonló bővítményeket:
    - Prettier: egységes kódformázáshoz
    - ESLint: kódminőség ellenőrzéshez
    - Tailwind IntelliSense: Tailwind osztályok automatikus kiegészítéséhez
    - Prisma
    - Vue (Official)
    - Python
    - ElixirLS

A core-app és a comms fejlesztéshez beolvas `.env` fájlokat. Bár a comms csak akkor, ha a dev-server.sh scriptet használod az indításhoz.

Minden projektben a saját dependenciáit a megfelelő csomagkezelővel kell telepíteni:

- Core Application (Nuxt.js): `pnpm install`
- Communications Service (Elixir/Phoenix): `mix deps.get`
- Reporting Service (Python): (előbb érdemes egy venv-et csinálni, lásd lejebb) `pip install -r requirements.txt`

Projekt struktúra:

```sh
Gyökér
  ├── app                     # Core Application (Nuxt.js)
  ├── comms                   # Communications Service (Elixir/Phoenix)
  ├── reporting               # Reporting Service (Python)
  └── docker-compose.yml      # Docker Compose (kitöltendő)
```

<center>

**<< Minden ami a production előkészítéséhez szükséges >>**

</center>

#### Futtatás fejlesztői módban:

##### Core Application

```bash
cd app
pnpm run dev
```

##### Communications Service

```bash
cd comms
./dev-server.sh
```

##### Reporting Service

```bash
cd reporting
# Virtuális környezet létrehozása és aktiválása
virtualenv venv
source venv/bin/activate  # Linux/MacOS
venv\Scripts\activate     # Windows
# Függőségek telepítése
pip install -r requirements.txt
# Fejlesztői szerver indítása
python app.py
```

#### Adatbázis

A prisma automatikusan elvégzi a migrációkat és seedeli az adatbázist fejlesztői módban a `pnpm run dev` parancs futtatásakor. De, a prisma sutdio is használható az adatbázis böngészéséhez és manuális módosításához:

```sh
cd app
pnpm run db:studio
```

Érdemes lehet a gyökérben és a core-app mappában található `package.json` fájlokban definiált futtatható parancsokat is megnézni. Illetve a dockerfile-okat, hogy hogyan épülnek a projektek production környezethez.

### Production

Környezeti változók a docker-compose-hoz és fejlesztéshez:

#### Előkészületek :

- RSA kulcspár generálása JWT aláíráshoz (RS256) a gyökér mappában futtassuk:

```bash
pnpm run rsa:key-gen
pnpm run rsa:pub-key-gen
```

Vagy, ha nincs pnpm:

```bash
ssh-keygen -t rsa -b 4096 -m PEM -f private.key
ssh-keygen -f private.key -e -m PKCS8 > public.key
```

- Google OAuth
  1. Google Cloud Console-ban hozz létre egy új projektet.
  2. Engedélyezd az "OAuth 2.0 Client IDs" szolgáltatást.
  3. Hozz létre egy új OAuth 2.0 kliensazonosítót webalkalmazás típusban.
  4. Állítsd be a visszahívási URL-eket (pl. `http://localhost:3000/api/auth/callback/google`).
  - [https://developers.google.com/identity/protocols/oauth2/](https://developers.google.com/identity/protocols/oauth2/)]
  - [https://console.developers.google.com/](https://console.developers.google.com/)
- Discord OAuth
  1. Lépj be a Discord Developer Portal-ba.
  2. Hozz létre egy új alkalmazást
  3. Menj az "OAuth2" fülre, és állítsd be a redirect URI-ket (pl. `http://localhost:3000/api/auth/callback/discord`).
  - [https://discord.com/developers/docs/topics/oauth2](https://discord.com/developers/docs/topics/oauth2)
  - [https://discord.com/developers/applications](https://discord.com/developers/applications)
- Discord Bot / Interactions
  1. A Discord Developer Portal-ban menj az alkalmazásod "Bot" fülére.
  2. Klikkelj a Reset Token gombra, és jegyezd fel a token értékét.
  3. Menj a "General Information" fülre, és jegyezd fel az "Application ID"-t és a "Public Key"-t.
  4. Majd, ha fut a `comms` szolgáltatás PUBLIKUS URL-en (fejlesztéshez: `ngrok http 4000`), állítsd be az Interactions endpointot (pl. `https://your-public-domain.com/api/discord/interactions`)
  5. Engedélyezd a szükséges jogosultságokat a bot számára az "Installation" fülön:
     - scopes: applications.commands, bot
     - permissions: Attach Files, Create Events, Embed Links, Manage Events, Send Messages
  6. Állítsd be, hogy a bot csak Guild (=szerver) installációval legyen elérhető.
  7. Telepítsd a botot a szerveredre az "Installation" fül alatt látható URL segítségével, ami valahogy így néz ki: `https://discord.com/oauth2/authorize?client_id=YOUR_CLIENT_ID`

#### PostgreSQL adatbázis:

- `POSTGRES_USER` - adatbázis felhasználónév
- `POSTGRES_PASSWORD` - adatbázis jelszó
- `POSTGRES_DB` - adatbázis név
- `POSTGRES_PORT` - adatbázis port (alapértelmezett: 5432)

#### Core/Migrator/Reporting szolgáltatások:

- `DATABASE_URL` - PostgreSQL adatbázis connection string (pl.: `postgresql://user:password@host:port/dbname?schema=workplanner`) a fentebbi értékek felhasználásával.

#### Migrator:

- `SEED_DB` - ha `true`, akkor a migráció után seedeli az adatbázist, alapértelmezett: `false`, elsősorban fejlesztéshez meg demozáshoz használatos.

#### Core Application (Nuxt):

- `NUXT_AUTH_SECRET`: Az Auth.js használja aláíráshoz és titkosításhoz.
- `NUXT_AUTH_URL`: Az Auth.js callback URL-je. `http://localhost:3000/api/auth`
- `NUXT_AUTH_ORIGIN`: A Core Application alap URL-je kéne, hogy legyen, de valamiért csak a `http://localhost:3000/api/auth` működik.
- `NUXT_COMMS_API_URL`: A communications microservice URL-je (pl. http://comms:4000)
- `NUXT_PUBLIC_REPORT_SERVICE_URL`: A reporting microservice PUBLIKUS URL-je (pl. http://localhost:8080)
- `NUXT_PUBLIC_REPORT_SERVICE_API_KEY`: A reporting microservice API kulcsa. Tetszőleges string, csak egyeznie kell a microservice-ben beállított értékkel. (pl. `eb791ef310f839e817cc0aac46f5883adea878e57f60472fba9d670ef74b2187`)
- `NUXT_JWT_PRIVATE_KEY`: RSA private key JWT aláíráshoz (RS256) a comms szolgáltatás felé küldött kérésekhez. A feljebb generált RSA private key tartalma. Ha a docker-composeban van ez megadva, akkor inline JSON string formátumban kell megadni ("-----BEGIN....\n...\n-----END...") vagy többsoros formátumban idézőjelek nélkül.
- `NUXT_GOOGLE_CLIENT_ID`: A Google OAuth kliens azonosító.
- `NUXT_GOOGLE_CLIENT_SECRET`: A Google OAuth kliens titkos kulcsa.
- `NUXT_DISCORD_CLIENT_ID`: A Discord OAuth kliens azonosító.
- `NUXT_DISCORD_CLIENT_SECRET`: A Discord OAuth kliens titkos kulcsa.

#### Communications Service (Elixir/Phoenix):

- `SECRET_KEY_BASE` - Phoenix alkalmazás titkos kulcsa (generálható: `mix phx.gen.secret`)
- `PHX_HOST` - Phoenix alkalmazás host neve (pl. `localhost` vagy `worktime-comms.yourdomain.com`)
- `DNS_CLUSTER_QUERY` - (opcionális) ha clustering-et használsz pl. Kubernetes-ben, akkor a DNS név, amin keresztül a node-ok megtalálják egymást (pl. `worktime-comms-headless.default.svc.cluster.local`)
- `JWT_PUBLIC_KEY` - A fenti privát kulcshoz tartozó publikus kulcs, JWT ellenőrzéshez (RS256) a core alkalmazás felől érkező kérésekhez. Ezt idézőjelek nélkül, többsoros formátumban kell megadni (yaml-ban pl. `|` vagy `>` operátorral).
- `SMTP_SERVER` - SMTP szerver címe e‑mailek küldéséhez.
- `SMTP_PORT` - SMTP szerver portja (pl. 587 TLS-hez vagy 465 SSL-hez).
- `SMTP_USERNAME` - SMTP szerver felhasználónév.
- `SMTP_PASSWORD` - SMTP szerver jelszó.
- `SMTP_SSL` - ha `true`, akkor SSL kapcsolatot használ az SMTP szerverrel.
- `SMTP_TLS_VERIFY` - ha `true`, akkor TLS kapcsolat esetén ellenőrzi a szerver tanúsítványát.
- `CORE_SERVICE_HTTP` - A core alkalmazás URL-je (pl. http://core-app:3000) bizonyos discord querykhez szükséges adatok lekéréséhez.
- `CORE_SERVICE_PUBLIC_HTTP` - A core alkalmazás PUBLIKUS URL-je (pl. http://localhost:3000) email linkek generálásához.
- `DISCORD_APP_ID` - A Discord alkalmazás azonosítója.
- `DISCORD_PUBLIC_KEY` - A Discord alkalmazás publikus kulcsa az interakciók aláírás ellenőrzéséhez.
- `DISCORD_BOT_TOKEN` - A Discord bot tokenje az API hívásokhoz.

#### Reporting Service (Python):

- `API_KEY` - A reporting szolgáltatás API kulcsa. Tetszőleges string, csak egyeznie kell a core alkalmazásban beállított értékkel. (pl. `eb791ef310f839e817cc0aac46f5883adea878e57f60472fba9d670ef74b2187`)

### Deployment

A deployment Docker Compose segítségével történik. A `docker-compose.yml` fájl jó kiindulási alapot nyújt, de a környezeti változókat ki kell tölteni a fentiek alapján.

#### Futtatás lokálisan buildelt image-ekkel:

```bash
docker-compose up -d --build
```

#### Futtatás Docker Hubos image-ekkel:

A compose fájl alapvetően buildelésre van beállítva, de az image sorok kikommentelésével és a build sorok eltávolításával használhatók a Docker Hub-on tárolt előre buildelt image-ek.

```bash
docker-compose up
```

**Fontos megjegyzés**: Ha szeretnéd a discord parancsokat használni, akkor a comms szolgáltatásnak elérhetőnek kell lennie egy publikus URL-en keresztül (pl. ngrok vagy valamilyen reverse proxy segítségével).
