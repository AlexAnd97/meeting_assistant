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
**Val: OpenAI Whisper**
Stöder svenska och engelska med hög noggrannhet.
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

## Förbud

- Ingen lokal filbearbetning på klienten — all AI-bearbetning sker server-side.
- Mötesdata får inte användas för AI-träning — detta måste verifieras i API-avtal.
- Ingen chunking-logik för sammanfattning i v1 — hela transkriptet skickas i en request.
- SQLite är inte tillåtet, inte ens för lokal utveckling mot delad databas.
- Ingen realtidsbearbetning (streaming av transkript) i v1.
