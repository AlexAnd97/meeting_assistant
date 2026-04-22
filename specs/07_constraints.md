## Tekniska begränsningar och val

### Frontend
**Val: React**
Brett ekosystem, stor community och väl testad för dataintensiva applikationer.
Enkel integration med komponentbibliotek och API-lager.

### Backend
**Val: FastAPI (Python)**
Python är native-språket för AI/ML-bibliotek vilket eliminerar brygglösningar.
Whisper och Claude SDK körs direkt i samma runtime utan overhead.
FastAPI genererar automatisk API-dokumentation (OpenAPI) vilket underlättar
frontend-integration och framtida onboarding av utvecklare.

### Databas och infrastruktur
**Val: Supabase (PostgreSQL + Storage + Auth)**
Supabase samlar databas, fillagring för mötesfiler och autentisering i ett verktyg.
PostgreSQL som underliggande motor säkerställer att PoC:n kan skalas och migreras
till självhostat Postgres utan kodändringar. SQLite utesluts eftersom det inte
är lämpligt för flertrådad access eller produktionssättning.

### Transkription
**Val: OpenAI Whisper, modell `medium`**
Stöder svenska och engelska med hög noggrannhet.
Modell `medium` används i v1 — bra balans mellan precision och kostnad/tid.
`large` ger marginellt bättre precision men är betydligt långsammare och dyrare.
`small` eller mindre utesluts på grund av otillräcklig precision för svenska.
Kan köras lokalt eller via API beroende på kostnadsprioritet.
Outputformat är tidsstämplat JSON vilket matchar systemets transkriptmodell.

### AI för sammanfattning
**Val: Claude (Anthropic) — en modell, inte flera**
Claudes kontextfönster på 200 000 tokens täcker hela transkript från långa möten
utan att chunking-logik behövs. Chunking introducerar komplexitet kring
sammansättning av delsvar och riskerar att förlora kontext över mötessegment.
En enda modell är enklare att debugga, kostnadsövervaka och byta ut.
Flera modeller i pipeline utesluts för PoC — det är en optimering för senare versioner.

### Hosting
**Val: Vercel (frontend) + Railway (backend)**
Vercel är branschstandard för React-applikationer med noll konfiguration och
automatisk deployment vid push. Railway hanterar Python-backends utan manuell
serverhantering. Supabase hanterar databasen separat. Alla tre har gratis-tiers
som täcker en PoC utan initiala infrastrukturkostnader.

---

## Miljöer

Systemet har tre miljöer: lokal utveckling, preview och produktion.
Staging som separat miljö behövs inte — Vercel skapar automatiskt en preview-deployment
per PR som fyller samma syfte.

### Lokal utveckling
- Konfiguration via `.env.local` i både frontend och backend
- Separat Supabase-projekt dedikerat för dev — delar aldrig databas med produktion
- Whisper körs lokalt för att undvika API-kostnader under utveckling
- Backend startas med `uvicorn` lokalt, frontend med `npm run dev`

### Preview (per PR)
- Vercel skapar automatiskt en publik preview-URL vid varje pull request
- Pekar mot dev-Supabase-projektet, inte produktion
- Används för granskning och manuell testning innan merge

### Produktion
- Frontend deployad via Vercel (main-branch triggar automatisk deploy)
- Backend deployad via Railway (main-branch triggar automatisk deploy)
- Eget Supabase-projekt med produktionsdata — åtskilt från dev

### Miljövariabler
Följande variabler ska finnas i respektive `.env`-fil och får aldrig committas:

```
# Backend
SUPABASE_URL=
SUPABASE_SERVICE_KEY=
ANTHROPIC_API_KEY=
OPENAI_API_KEY=          # för Whisper API (om ej lokalt)

# Frontend
VITE_SUPABASE_URL=
VITE_SUPABASE_ANON_KEY=
VITE_API_BASE_URL=
```

`.env`-filer ska alltid ligga i `.gitignore`.
Ett `.env.example` med tomma värden ska committas som referens för nya utvecklare.

---

## GDPR och juridiska krav

Systemet bearbetar personuppgifter i form av röster, namn och diskussioner från möten.
Följande krav gäller innan systemet får användas i produktion med riktiga användare.

### Data Processor Agreements (DPA)
Tecknade DPA:s krävs med samtliga underleverantörer som bearbetar personuppgifter:
- **Supabase** — lagrar mötesfiler och transkript
- **Anthropic (Claude)** — bearbetar transkriptinnehåll
- **OpenAI (Whisper)** — bearbetar ljud-/videofiler

Utan DPA är behandlingen av personuppgifter inte laglig enligt GDPR.

### AI-träningsförbud
Varken Anthropic eller OpenAI får använda mötesdata för modellträning.
Detta ska verifieras i respektive API-avtal och aktivt opt-outas om det krävs.
Kravet gäller oavsett om data är anonymiserad eller ej.

### Dataresidency
Supabase ska konfigureras med EU-region (t.ex. `eu-west-1`) för att säkerställa
att personuppgifter inte lämnar EU utan Standard Contractual Clauses (SCC).
Notera att Railway och Vercel primärt är US-baserade — backend-bearbetning
sker utanför EU vilket kräver att SCC är på plats med dessa leverantörer.

### Datalagring och retention
Möten, transkript och sammanfattningar ska inte sparas längre än nödvändigt.
En retention-policy ska definieras innan produktionslänsering — rekommendation är
att användaren själv styr livslängden via radering (FR-19).
Automatisk radering efter inaktivitet är en v2-funktion.

### Tredjepartskonsent
Mötesdeltagare vars röster bearbetas av systemet är registrerade i systemet
som personuppgiftssubjekt även om de inte själva är användare.
Det åligger den användare som laddar upp mötesfilen att säkerställa att
inspelning och AI-bearbetning är kommunicerad till samtliga deltagare.
Systemet ska visa en tydlig informationstext om detta vid uppladdning.

---

## CORS

Backend ska tillåta cross-origin requests från frontend-origin.
CORS ska konfigureras explicit i FastAPI — utan det blockerar webbläsaren alla API-anrop.

```python
# Tillåtna origins per miljö
Lokal dev:   http://localhost:5173
Preview:     https://*.vercel.app
Produktion:  https://<produktionsdomän>
```

Tillåtna metoder: `GET`, `POST`, `PATCH`, `DELETE`, `OPTIONS`.
Tillåtna headers: `Authorization`, `Content-Type`.
Wildcard origin (`*`) är inte tillåtet i produktion.

---

## Förbud

- Ingen lokal filbearbetning på klienten — all AI-bearbetning sker server-side.
- Mötesdata får inte användas för AI-träning — detta måste verifieras i API-avtal.
- Ingen chunking-logik för sammanfattning i v1 — hela transkriptet skickas i en request.
- SQLite är inte tillåtet, inte ens för lokal utveckling mot delad databas.
- Ingen realtidsbearbetning (streaming av transkript) i v1.
- CORS wildcard (`*`) är inte tillåtet i produktion.
