## Icke-funktionella krav

---

### Prestanda

#### NFR-01: Uppladdningstid
Filer upp till 500 MB ska laddas upp utan timeout.
Frontend ska visa progress under hela uppladdningen (FR-06).

#### NFR-02: Transkriptionstid
Transkription via Whisper ska slutföras inom 2 minuter per 30 minuters möte.

#### NFR-03: Sammanfattningstid
Sammanfattning via Claude ska genereras inom 60 sekunder efter att transkriptet är klart.

#### NFR-04: API-svarstid
Alla endpoints utom bearbetningsendpoints ska svara inom 500 ms under normala förhållanden.

---

### Säkerhet

#### NFR-05: Autentisering på alla endpoints
Alla API-endpoints utom eventuella health-checks ska kräva giltig Supabase JWT.
Backend ska validera token på varje request — frontend-skydd räcker inte.

#### NFR-06: Row Level Security (RLS)
Supabase RLS ska vara aktiverat på tabellerna `meetings`, `transcripts` och `summaries`.
Policys ska säkerställa att en användare endast kan läsa och skriva egna rader.
RLS är inte aktivt som default i Supabase och måste explicit konfigureras.

#### NFR-07: Privat Storage-bucket
Supabase Storage-bucketen för mötesfiler ska vara privat.
Filer ska aldrig vara tillgängliga via publik URL.
Nedladdning av filer ska ske via signerade temporära URL:er med kort livslängd (max 60 sekunder).

#### NFR-08: Filvalidering
Backend ska validera att uppladdade filer faktiskt är av tillåtet format
genom att inspektera filinnehållet (magic bytes), inte bara filändelsen.
Filer som inte valideras ska avvisas med HTTP 422.

#### NFR-09: Rate limiting
Upload-endpointen (`POST /meetings`) ska begränsas till max 10 uppladdningar
per användare och timme för att förhindra missbruk och okontrollerade API-kostnader.

#### NFR-10: Secrets i miljövariabler
API-nycklar och databasuppgifter får aldrig hårdkodas i källkoden.
Alla secrets hanteras via miljövariabler enligt specifikationen i `07_constraints.md`.

#### NFR-11: HTTPS
All kommunikation mellan frontend, backend och externa API:er ska ske över HTTPS/TLS.
HTTP-anrop ska inte tillåtas i produktion.

---

### Dataskydd och GDPR

#### NFR-12: Kryptering i vila
All data i Supabase (databas och storage) ska vara krypterad i vila.
Supabase krypterar som default — detta ska verifieras och inte avaktiveras.

#### NFR-13: Kryptering i transit
All data i transit ska krypteras via TLS 1.2 eller högre.

#### NFR-14: Informationstext vid uppladdning
Vid uppladdning av mötesfil ska användaren se en tydlig informationstext om att:
- Mötesfilen och transkriptet bearbetas av externa AI-tjänster (Whisper, Claude)
- Det åligger användaren att säkerställa att mötesdeltagare är informerade

#### NFR-15: Fullständig radering
När ett möte raderas (FR-19) ska följande tas bort utan undantag:
- Raden i tabellen `meetings`
- Tillhörande rader i `transcripts` och `summaries`
- Filen i Supabase Storage

Kaskadradering ska konfigureras i databasen via `ON DELETE CASCADE`.

---

### Tillgänglighet

#### NFR-16: Skärmläsarkompatibilitet
Sammanfattningar och transkript ska renderas som semantisk HTML
och vara kompatibla med skärmläsare.

#### NFR-17: Responsiv layout
Gränssnittet ska fungera på skärmar från 375px (mobil) till 1440px (desktop).

---

### Skalbarhet

#### NFR-18: Bakgrundsbearbetning
Bearbetningspipelinen (transkription + sammanfattning) ska köras som bakgrundsuppgift
och inte blockera API-responsen. FastAPI `BackgroundTasks` används i v1.
